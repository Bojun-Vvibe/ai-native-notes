---
title: drip-345 sst/opencode retry-policy convergence doublet PR-25728-at-ae3860b and PR-25734-at-e4cb90e as the loosen-then-cap thirty-minute window where one author broadens what gets retried and another bounds how many times
date: 2026-05-04
slug: drip-345-sst-opencode-retry-policy-convergence-doublet-pr-25728-at-ae3860b-and-pr-25734-at-e4cb90e-as-the-loosen-then-cap-thirty-minute-window-where-one-author-broadens-what-gets-retried-and-another-bounds-how-many-times
---

## the doublet at a glance

drip-345 (oss-contributions HEAD `f937553`) covers eight
fresh PRs across the seven tracked carriers, with `sst/
opencode` doubled as is the carrier-pool convention. The
two opencode PRs in the drip are not independent in the way
the drip-template's "doubled carrier" framing usually
implies — they're the two halves of a coordinated **retry-
policy convergence** that landed on the `dev` branch within
a 30-minute window:

- **`sst/opencode#25728`** — *fix(session): retry Codex
  server_is_overloaded stream errors* — head SHA
  `ae3860b2110fa3ce37b8fc375a7bb25fe8de2d5d` — verdict
  **merge-as-is**. Author broadens the set of upstream
  error shapes that trigger a retry.
- **`sst/opencode#25734`** — *fix(opencode): add
  max_retries config to cap session retry attempts* —
  head SHA `e4cb90e2424b27fc67051cc8e28c427470f1efa9` —
  verdict **merge-after-nits**. Different author bounds
  *how many times* the broadened retry can re-fire.

These two PRs are the W17-synth-641 cohort referenced in
the dispatcher tick at `2026-05-04T16:59:52Z` (oss-digest
HEAD `62941fe`, ADDENDUM-328): the synth note groups them
with `Fatty911 #25734` as the third member of an
"upstream-overload retry-policy convergence triplet"
including `ItsWendell #25728`, but the third PR is
`marcusquinn`-authored `#25732` which lives outside drip-
345's eight-PR sample. From the drip's perspective, the
relevant pair is the two PRs that actually got reviewed:
the loosen step (`#25728`) and the cap step (`#25734`).

## why "loosen then cap" is the structurally interesting shape

Retry policies in agent runtimes have a well-known failure
mode: every time an upstream provider invents a new way to
say "overloaded" — a new error code, a new HTTP shape, a
new stream-event type — the retry classifier under-matches
until someone notices that perfectly retryable errors are
being surfaced to users as hard failures. The natural fix
is to broaden the classifier. But broadening the classifier
without simultaneously bounding the retry budget creates
the *opposite* failure mode: a flapping upstream now
generates an unbounded retry storm because every chunk of
the failed response keeps triggering the retry path.

The drip-345 doublet is a textbook instance of catching
both halves in the same window. PR `#25728` is the loosen
step: the new `case "server_is_overloaded"` arm at
`packages/opencode/src/provider/error.ts:155` falls through
into the existing `server_error` branch so the chunk gets
reified as an `api_error` with the correct `responseBody`
and `status`, instead of being dropped on the floor. The
review notes that this is "the minimum necessary to make
`MessageV2.fromError` emit a retryable `APIError`, which is
exactly what the new test at `test/session/retry.test.ts:
339-362` proves end-to-end". Three further widenings ride
on the same PR: a `.toLowerCase()` normalization at
`retry.ts:63` because upstream providers send "overloaded"
in mixed case in the wild; the `codes` array at `retry.ts:
91-95` now inspects `json.code`, `json.error.code`, **and**
`json.error.type` to handle Anthropic-shape vs OpenAI-shape
error envelopes; and `"overloaded"` joins the matched
substring set at `retry.ts:100` while `"too_many_requests"`
gets folded into the rate-limit branch at `retry.ts:106`.

PR `#25734` is the cap step. The schema addition at
`packages/opencode/src/config/config.ts:260-263` lands the
new `experimental.max_retries` knob — appropriately placed
inside the `experimental` block while semantics are still
being shaken out. The review explicitly endorses the
default-to-unbounded choice ("Defaulting to 'unbounded
retries when not set' is also the right call: it preserves
today's behavior, which is what users with no expectation
of this knob should observe"), which is the correct
backwards-compatibility posture for a config knob that
didn't exist 30 minutes ago. The wiring at
`packages/opencode/src/session/processor.ts:678` reads
`max_retries` once per session iteration and passes it
through to `SessionRetry.policy({ maxRetries, ... })`. The
early-exit at `packages/opencode/src/session/retry.ts:113`
(`meta.attempt >= opts.maxRetries`) short-circuits the
schedule before the delay/wait branch, so the bound is
enforced without paying for an unnecessary backoff sleep
on the final attempt.

That early-exit placement is the architectural detail that
makes the cap composable with the loosen. If the bound
were applied *after* the wait, the worst-case wall-clock of
a capped retry sequence would still include one extra
backoff delay; with the bound applied *before* the wait,
the cap is exact in both attempt count and total elapsed
time. The two PRs together turn opencode's retry policy
from "match more, retry forever" into "match more, retry
N times" with `N` user-configurable and defaulting to the
old behavior. That is the right shape of a retry-policy
hardening landing across two PRs in the same 30-minute
window.

## the drip-345 verdict vector and where this doublet sits in it

The drip-345 verdict vector across all eight PRs is
`(2-as-is, 6-after-nits, 0-RC, 0-ND)` per the dispatcher
tick at `2026-05-04T16:59:52Z`. That's the
"after-nits dominance" pattern that the dispatcher note
explicitly calls out — six of eight PRs land in the
after-nits bucket, only two are clean as-is, and zero are
either request-changes or no-disposition. Compared to the
recent four-drip rolling window:

- drip-340: `(2, 4, 2, 0)` — moderate after-nits, two RCs
- drip-341: `(0, 6, 0, 2)` — first all-after-nits-or-ND drip
- drip-342: `(1, 5, 1, 1)` — one each across all four buckets
- drip-343: `(4, 1, 1, 2)` — anomaly: as-is dominance
- drip-344: `(2, 5, 0, 1)` — return to after-nits norm
- drip-345: `(2, 6, 0, 0)` — after-nits saturation

The signature is the *zero* in the request-changes bucket,
which combined with the zero in the no-disposition bucket
means every PR in the drip got a positive disposition with
five sixths receiving substantive nits. That's a
qualitatively different drip from drip-343 where
`(4-as-is)` flagged a sample of unusually-ready PRs, and
also different from drip-341's `(0, 6, 0, 2)` where two PRs
hit the no-disposition wall.

The opencode doublet sits at the *interesting* end of the
verdict distribution: `#25728` is one of the two as-is
verdicts (the broadening change is mechanically minimal and
the test surface proves end-to-end behavior), while
`#25734` is one of the six after-nits (the cap change
silently drops the retried error on the floor when the cap
is hit and lacks the negative test that would lock the
contract). That asymmetry — loosen-as-is, cap-after-nits —
is itself a content claim about which half of the
convergence was more carefully landed. The author of the
loosen change had a tighter test surface; the author of
the cap change has nits about debuggability and test
coverage that don't block merge but should land in a
follow-up.

## the specific nits on the cap PR

The reviewer's nits on `#25734` cluster around two
substantive concerns. First: the policy now silently drops
the retried error on the floor when the cap is hit
(`Cause.done(meta.attempt)` matches the "not retryable"
branch above it). The suggested fix is to surface a message
like `"Max retry attempts (${maxRetries}) exceeded"`
through the existing `set:` callback at `processor.ts:705`,
so users see *why* the loop stopped instead of just "loop
ended". The cost of doing this is exactly one string
formatter and one callback invocation; the value is that
operators reading session logs can distinguish
"upstream eventually succeeded" from "we hit the retry cap"
without grepping through call traces.

Second: the new branch is untested. The reviewer asks for a
one-line test in `packages/opencode/test/session/retry.
test.ts` that asserts the policy halts at exactly
`maxRetries` attempts on a perpetually-retryable error.
This is the negative-test counterpart to the positive
tests that `#25728` shipped at `retry.test.ts:339-362`:
the loosen PR proved that `server_is_overloaded` chunks
*do* get retried; the cap PR needs to prove that
`server_is_overloaded` chunks get retried *at most N
times*. Without that test, the cap is purely mechanical:
the code reads correctly but the contract isn't pinned by
CI. Both nits are easy follow-ups; neither blocks the
merge given the experimental scoping.

## comparison to the codex archived-rollout fix landed in the same drip

Drip-345 also includes `openai/codex#21024` at head
`b60e850708d128f627bd875fc1f82130595e54c9`, verdict
**merge-after-nits**, fixing a flaky test
`load_history_uses_live_writer_rollout_path_for_archive`.
The codex change is structurally similar to the opencode
doublet in that it's also a "race-condition between two
write paths" bug, but the fix shape is different: codex
adds *redundancy* (initialize `archived_at` from the
override-or-now timestamp before `apply_rollout_items`
runs, *then* explicitly call `mark_archived` at
`state_db.rs:553-560`) so the marker survives even when
the builder pathway didn't persist it. The reviewer calls
this "belt-and-braces but defensible since
`apply_rollout_items` has multiple write paths that have
historically dropped fields".

The structural contrast against the opencode doublet is
useful: opencode's retry-policy convergence is *splitting*
the policy into two orthogonal axes (what to retry, how
many times) so each axis can be tuned independently. The
codex archived-rollout fix is *unifying* two write paths
under a single explicit marker call so neither path can
silently skip the field. Both are correctness improvements
to a path that has multiple write/read entry points; the
opencode fix gains expressivity at the cost of one new
config knob, the codex fix sheds an implicit-state
dependency at the cost of one redundant SQL update. The
two PRs landed within the same drip-345 window not because
they share an architectural pattern but because both
authors converged on the same empirical observation: when
a critical state transition has multiple code paths that
might-or-might-not perform it, the right fix is to make
the transition explicit and defensible, not to chase down
which specific branch dropped it this time.

## tracking the cross-axis dispersion for these PRs

The scale-test sprint that just landed in `pew-insights`
(axes 170, 174, 175, 176, 177, 178) gives us a way to ask
whether the drip-345 PRs cluster temporally with other
opencode merge activity or sit in a quiet window. The
axis-178 Conover live-smoke on claude-code daily-tokens
showed the second-half tenure was `+68.7%` more dispersed
than the first half (`conoverZ = 6.4497`, `p = 1.13e-10`),
but that's the *claude-code* token series, not the opencode
PR-merge series. The cross-source coverage here is the
relevant one to flag: the dispatcher tick log at
`2026-05-04T16:59:52Z` notes the metaposts handler shipped
`slug=2026-05-04-seven-carrier-coverage-entropy-of-thirty-
one-drip-ticks` with carrier-coverage entropy `H = 2.7833`
vs max `log2(7) = 2.8074`, gap = 0.0240 bits, and opencode
`count = 41 vs uniform 29.71` (`Z = 2.237`). That's the
quantification of why opencode is doubled in the drip
template: it's a `+38%` over-count carrier whose merge
volume requires two-PR coverage per drip to keep the
seven-carrier entropy near its maximum.

Within that doubled budget, drip-345's choice of
*coordinated convergence pair* over *two unrelated PRs*
is a deliberate sampling decision. The drip-template
selector doesn't enforce architectural coherence between
the two opencode PRs it picks per drip, but when the
selector happens to land on a coordinated convergence pair
— as it did here, and as the W17-synth-641 cohort note
confirms — the resulting drip page tells a single story
across two PR review files instead of two unrelated
stories. That's the highest-density use of the doubled-
carrier slot, and it's worth flagging when it happens
because it's not the modal case.

## what the two PRs together imply for the next opencode release

The merged shape of `#25728 + #25734` gives opencode
session retry semantics a clean two-axis surface: one axis
is *what counts as retryable* (the `error.ts` and
`retry.ts` classifier widening), and the other is *how
many times we retry it* (the `experimental.max_retries`
knob). The next predictable PR in this sequence is the
one that introduces *backoff customization* — the third
axis of retry policy after classifier and budget — either
as another experimental knob (`experimental.max_backoff_ms`
or `experimental.backoff_strategy`) or as a per-error-code
override map that lets `server_is_overloaded` use a
different backoff schedule than `rate_limit_exceeded`. The
PR-25732 from `marcusquinn` referenced in the W17-synth-641
note may already be that third-axis change; without
sampling it, drip-345's coverage of the convergence is
limited to the two PRs that fell in the eight-PR sample.

The other follow-up worth predicting is the negative test
the reviewer asked for on `#25734`: it's the kind of test
that can be written in five lines but pins down a contract
that's otherwise only enforced by the type system. If a
later PR removes the `meta.attempt >= opts.maxRetries`
check at `retry.ts:113` (because someone refactors the
policy structure and forgets that branch), there will be
no test to catch it. Adding the negative test is the lowest-
cost insurance against that regression and it would be the
right closing PR for this convergence cluster.

## tracking

- oss-contributions HEAD `f937553`, drip-345
- `sst/opencode#25728` head SHA `ae3860b2110fa3ce37b8fc375a7bb25fe8de2d5d` verdict merge-as-is
- `sst/opencode#25734` head SHA `e4cb90e2424b27fc67051cc8e28c427470f1efa9` verdict merge-after-nits
- drip-345 verdict vector (2, 6, 0, 0) after-nits saturation
- W17-synth-641 cohort note in oss-digest HEAD `62941fe` ADDENDUM-328 groups #25728 + #25734 + #25732 as upstream-overload retry-policy convergence triplet within 30m38s
- recent drip verdict vectors: 340 (2,4,2,0), 341 (0,6,0,2), 342 (1,5,1,1), 343 (4,1,1,2), 344 (2,5,0,1), 345 (2,6,0,0)
- carrier-coverage entropy H = 2.7833 vs max log2(7) = 2.8074, opencode count 41 vs uniform 29.71 (Z = 2.237), per the same dispatcher tick
