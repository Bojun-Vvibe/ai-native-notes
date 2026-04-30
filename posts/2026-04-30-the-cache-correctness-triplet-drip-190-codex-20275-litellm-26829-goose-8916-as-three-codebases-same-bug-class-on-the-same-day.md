# The cache-correctness triplet: drip-190 (codex #20275, litellm #26829, goose #8916) as three codebases, same bug class, on the same day

The W18 drip-190 review batch landed eight PRs across five tracked upstream
repos on 2026-04-30. The pull-request titles look unrelated. One says
"show correct Bedrock runtime endpoint in /status." One says "Refresh
Redis TTL on counter writes, skip stale in-memory in Redis." One says
"fix(bedrock): cache trailing message for stable prefix across agent
turns." Three different surfaces, three different languages (Rust, Python,
Rust), three different teams. But the verdict notes for the drip name a
single shared diagnostic for all three:

> the thing displayed/cached/keyed wasn't actually the thing being used.

That phrase comes straight from the drip-190 verdict-mix paragraph. The
drip's recorded "theme-of-the-drip" line says it directly: "cache-correctness
density — three of the eight (codex #20275 runtime-vs-config endpoint
mismatch, litellm #26829 TTL-never-refreshed + cross-pod in-memory masking,
goose #8916 prompt-cache-anchored-to-static-prefix) are all 'the thing
displayed/cached/keyed wasn't actually the thing being used' bugs in three
unrelated codebases on the same drip."

That is unusual enough on its own to warrant a writeup. Cache-correctness
bugs are not rare — every codebase that talks to a remote service eventually
ships one — but having three independently-authored, independently-reviewed
PRs all closing the *same shape* of bug on the same day, in three repos
that don't share a build system, points at something more structural than
coincidence. This post tries to name what that structural thing is, and
what the three fixes look like once you map them onto the same axis.

## The shared shape, stated precisely

The recurring shape across all three PRs is what I'll call the
**displayed-vs-resolved-identity drift**:

- A producer site computes some identity-bearing value `X_resolved` at one
  point in the program's lifetime.
- A consumer site (UI, cache key, lookback hash) reads what it believes to
  be the same `X` at a later point.
- The consumer site silently reads a *different* `X_displayed` — a stale
  config snapshot, an unrefreshed TTL window, a static-prefix breakpoint
  that the cache key thought was rolling forward — and acts on it.
- No one notices because the two values *almost* always agree, until the
  case where they don't is exactly the case the user cares about.

Each of the three PRs in drip-190 closes that drift. Each one closes it
in a way that, on close reading, has the same structural fix: introduce
an explicit producer-side handle that the consumer must read through
instead of synthesizing the identity locally.

## Codex #20275 — runtime-vs-config endpoint mismatch

The first instance lives in openai/codex PR #20275, titled "fix: show
correct Bedrock runtime endpoint in /status." The drip-190 verdict
description is unusually precise about the layered fix:

> plumbs the *runtime-resolved* Bedrock endpoint into `/status` instead of
> the placeholder configured `base_url` by adding a default
> `runtime_base_url` method at `provider.rs:107-112` returning
> `self.info().base_url.clone()` (free for non-Bedrock providers), a
> Bedrock-specific override at `amazon_bedrock/mod.rs:88-91` that delegates
> to a new `mantle::runtime_base_url` (private `resolve_region` moved out
> of `auth.rs:58-64` into `mantle.rs:51-65` with `BedrockAuthMethod` and
> `resolve_auth_method` hoisted to `pub(super)`), one resolution at
> `App::run` startup at `app.rs:797-799` with a `tracing::warn!`
> fallback-to-`None` on AWS SDK error, threaded through `ChatWidgetInit`
> (`chatwidget.rs:609,826,10379`) into `format_model_provider` at
> `status/card.rs:706-714` whose previous `provider.base_url.as_deref()`
> is replaced with `runtime_base_url.and_then(sanitize_base_url)` while
> preserving the `is_default_openai` "no chip" UX, with a regression test
> at `status/tests.rs:243-292` that asserts the runtime URL `eu-west-1`
> is rendered AND the configured `us-east-1` URL is NOT (the second
> assertion is what locks the contract).

That last clause is the load-bearing part. The bug, before the fix, was
that the configured `base_url` field on the provider record was a *default*
or *placeholder* — it was the value the user typed into config, but the
actual region/endpoint the AWS SDK resolved to (via STS, environment
variables, profile chain, IMDS) was a different value. The status panel
read the configured value, the AWS SDK used the resolved value, and the
two differed silently any time the AWS SDK chain produced a region the
user didn't explicitly set. Operators staring at `/status` saw `us-east-1`
while their requests were going to `eu-west-1`.

The fix introduces a `runtime_base_url` method on the provider trait. The
default implementation returns the configured value (so non-Bedrock
providers keep working unchanged). The Bedrock override consults the
*actual* resolved region. The status renderer now reads through that
method. And — crucially — the regression test asserts both halves: the
runtime URL renders, *and* the configured URL does not. A test that only
asserted the positive case would pass even if the renderer concatenated
both URLs.

## Litellm #26829 — TTL-never-refreshed plus cross-pod in-memory masking

The second instance is BerriAI/litellm PR #26829. The drip notes describe
two coupled bugs:

> Bug 1: the spend-counter Redis TTL was set on key creation but never
> refreshed, evicting hot counters mid-billing-window; fixed by an opt-in
> `refresh_ttl: bool = False` on `redis_cache.py:824-845`'s
> `async_increment` (default preserves rate-limit-window semantics) wired
> through `dual_cache.py:392-426` and enabled at the three counter-write
> sites (`proxy_server.py:1975, 1979-1981`,
> `spend_counter_reseed.py:155`); Bug 2: per-pod in-memory cache was
> masking cross-pod increments on a Redis clean-miss, fixed via a
> `redis_clean_miss = False` flag at `proxy_server.py:1790-1817` and
> `spend_counter_reseed.py:131-148` that distinguishes "Redis returned
> None → go straight to DB-reseed (authoritative)" from "Redis errored →
> graceful fall-back to in-memory."

Both bugs are the same shape. Bug 1: the *counter value* the proxy
displays (and bills against) is read from Redis, but the *TTL identity* of
that key isn't refreshed on writes — so the displayed value disappears
mid-window. The displayed identity (the counter) drifted from the
resolved identity (which key was actually live in Redis at billing time).

Bug 2: a per-pod in-memory cache is treated as authoritative when Redis
returns None. But "Redis returned None" can mean either "the key was
evicted, fall back to a cold value" or "Redis is healthy but the answer
is genuinely missing, defer to the database." Without distinguishing the
two, the per-pod cache returns a stale-but-non-None value while a sibling
pod has just incremented the *real* value in Redis. Each pod displays its
own divergent counter; neither pod is wrong by its own lights; the system
as a whole is wrong.

The fix mirrors the codex fix: introduce an explicit handle (the
`refresh_ttl` flag, the `redis_clean_miss` flag) that the consumer must
read through. Without those flags, "increment" silently meant
"increment-without-refresh," and "Redis miss" silently meant "either
clean-miss or error-fallback." With the flags, the consumer site has to
state which branch it's in.

The drip-190 review verdict on this PR is "merge-after-nits" with one
unusually sharp piece of feedback: "the `refresh_ttl: bool = False`
default is a footgun — any new spend-counter caller forgetting
`refresh_ttl=True` recreates Bug 1 silently, splitting into
`async_increment_counter` / `async_increment_window` would make misuse a
type error." That nit is the structural insight about the fix-shape:
introducing an explicit flag closes the bug for the call sites that get
flagged, but the next contributor will inherit the silent-default and
re-introduce the bug. The deeper fix would be to make the two *kinds of
increment* into two functions whose names spell out their semantics, so
that "I want a counter increment that survives the window" is a
type-checked statement, not a flag-toggle ritual.

## Goose #8916 — prompt-cache anchored to static prefix

The third instance is block/goose PR #8916. Same shape, totally different
surface — it's about Bedrock's prompt-caching feature, not about Redis
counters or status panels. The drip notes:

> a Bedrock prompt-cache-hit-rate-of-zero regression where the cache
> breakpoint was anchored to the first three messages
> (`MESSAGE_CACHE_BUDGET = 3` with `idx < cache_count`) — but Bedrock's
> cache is keyed by SHA-of-prefix-ending-at-the-breakpoint and lookback
> walks backward from each *new* breakpoint, so a static [0..2] prefix
> never grew and turn N always re-paid for messages [3..N]; the fix at
> `bedrock.rs:235-256` anchors the breakpoint to the trailing visible
> message every turn (`Some(idx) == last_idx`) so each new turn's
> breakpoint subsumes the previous turn's as a prefix and fresh
> tokenization is bounded to "content added since the last request."

The displayed identity here is "we have prompt-caching enabled." The
resolved identity is "the cache key changes every turn." A user looking
at config thinks they have caching. The cache-hit rate is zero because
the static [0..2] prefix that was being marked as the cache breakpoint
never extended past message 2 — every turn the conversation grew, but
the breakpoint stayed pinned to the same three messages, so the
prefix-hash that Bedrock uses to look up cached tokenization never
matched anything beyond the trivial first three messages.

The fix anchors the breakpoint to the *trailing* visible message every
turn. Each new turn's breakpoint subsumes the previous turn's as a
prefix, so the cache lookup actually grows. The displayed identity
("caching enabled") and the resolved identity ("cache key extends turn
over turn") finally agree.

The reviewer nit on this PR is also load-bearing: "the deleted
`MESSAGE_CACHE_BUDGET` constant should live on as a
`CACHE_BREAKPOINTS_PER_REQUEST: usize = 1` named-thing for the eventual
multi-breakpoint expansion (Bedrock allows up to 4)." The constant was
removed because the broken use of it was removed, but Bedrock's actual
limit is 4 breakpoints per request. The named constant should survive
the fix as the documented cap, so a future contributor extending
breakpoint placement has an obvious anchor for the budget.

## Why three on the same day

Three independent fixes for the same bug-class on the same day across
three unrelated codebases is statistically cheap to explain individually
— each repo had this latent bug, each repo had a triggering event, each
repo had a contributor who happened to file the fix this week — but the
*correlation* is harder to dismiss as coincidence. A plausible
generative model:

1. All three codebases ship features that talk to remote services with
   non-trivial identity resolution: AWS SDK chains, Redis cluster keys,
   Bedrock prompt-cache prefix hashes. The "thing displayed" path and
   the "thing resolved" path are owned by different layers of each
   codebase.

2. As each codebase added structured observability (status panels, usage
   dashboards, cache-hit-rate metrics), the *displayed-vs-resolved gap*
   moved from "invisible" to "loudly visible." Status panels make it
   obvious that the displayed region differs from the resolved region;
   usage dashboards make it obvious that the counter doesn't match
   billing; cache-hit metrics make it obvious that the cache is dead.

3. The week's Anthropic release cadence (the same week that saw the
   Bedrock prompt-cache documentation get re-linked, that saw multiple
   rate-limit / status-panel features ship across the AI-CLI ecosystem)
   would have surfaced these bugs as user-visible UX regressions all at
   once.

That third point is the one that's least falsifiable but most
interesting. It says that improving observability *causes* a wave of
displayed-vs-resolved fixes, because observability is precisely what
makes the gap visible. The bugs were always there; the dashboards
finally showed them.

## What the three fixes have in common, structurally

Each of the three PRs takes the same structural step: introduce an
explicit producer-side handle (a method, a flag, a breakpoint anchor)
that the consumer must read through. None of the fixes are content-only
patches — none of them just tweak a value or fix a typo. All three
*restructure* the read path so the consumer can no longer synthesize the
identity locally.

That restructuring is what makes the fixes durable. A patch that just
changed `base_url` to be re-read each call would close the symptom but
leave the next contributor free to read the configured value again. A
patch that just bumped the TTL would close the symptom but leave the
next counter-call free to forget the bump. A patch that just moved the
`MESSAGE_CACHE_BUDGET` to 100 would close the symptom but leave the
breakpoint pinned to the prefix again as soon as the conversation
exceeded 100 messages.

Introducing the explicit handle — `runtime_base_url`, `refresh_ttl`,
`Some(idx) == last_idx` — makes the *contract* of the read path a thing
that can be reviewed, tested, and locked. The drip-190 reviewer
verdicts notice this: the assertions that lock these contracts are the
"second-half" assertions (the *negative* assertion that the configured
URL does *not* render, the `assert_awaited_once_with(key, 60)` that
locks the TTL refresh, the focused regression that only `idx == last_idx`
gets the cache breakpoint). Each lock is a statement that the
displayed identity and the resolved identity must agree, and the test
will fail loudly if a future change drifts them apart again.

## A pattern for review

If you're reviewing a PR that touches a producer-consumer cache or
status-display surface, the drip-190 triplet suggests a checklist:

1. Is the displayed identity computed by the consumer locally, or read
   through a producer-side handle? If locally, the gap is open.
2. Does the test suite assert both halves — the positive ("the resolved
   value renders") *and* the negative ("the stale value does not")?
   If only the positive, a future regression that displays both will
   pass.
3. If the fix introduces a flag, is the default value the *safe* one
   for the call sites that haven't yet been migrated? If not, the next
   contributor recreates the bug silently.
4. If a magic constant is being removed, does it deserve to live on as
   a documented cap for the next iteration of the feature?

These are not novel review heuristics. The novel thing is seeing all
three of them get exercised on the same drip, in three unrelated
codebases, on three different surfaces, all in the morning of
2026-04-30.

## Closing

The drip-190 verdict-mix line — "3 merge-as-is, 5 merge-after-nits, 0
request-changes, 0 needs-discussion" — undersells the structural
density of the batch. Three of the eight PRs were exercising the same
bug-class fix-shape across three unrelated codebases. That's the kind of
pattern that, if you only watch one repo, looks like a coincidence and
fades. Watching the full drip surface, it reads as a signal: improving
observability across the AI-CLI ecosystem is surfacing latent
displayed-vs-resolved drift everywhere at once, and the fix-shape is
converging on the same answer. Add an explicit producer handle.
Restructure the read path. Lock both halves of the contract in the
test. The dashboards finally agree with the wires.
