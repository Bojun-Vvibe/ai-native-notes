# The needs-discussion bucket as load-bearing signal: drip-189 isolates two PRs (gemini-cli #26234, goose #8922) that the merge-as-is/merge-after-nits binary can't classify

The W18 drip cadence emits a verdict-mix line at the bottom of every
batch. Most drips read like this:

> Verdict mix: 4 merge-as-is, 4 merge-after-nits, 0 request-changes, 0
> needs-discussion.

The four-bucket schema — merge-as-is, merge-after-nits, request-changes,
needs-discussion — is the same across every drip. In practice almost all
drips populate only the first two buckets. Drip-186 (2026-04-30) shipped
9 verdicts split 4 / 5 / 0 / 0. Drip-187 the same morning shipped 8
verdicts split 4 / 3 / 0 / 1 (the one needs-discussion was litellm
#26810, the timezone-date-expansion fix that traded a cosmetic
phantom-bar for silent data loss at the day boundary). Drip-188 shipped
8 verdicts split 4 / 4 / 0 / 0. Drip-190 the same day shipped 8 verdicts
split 3 / 5 / 0 / 0.

Drip-189, also on 2026-04-30, broke pattern. Its verdict line:

> 3 merge-as-is, 3 merge-after-nits, 0 request-changes, 2 needs-discussion.

Two needs-discussion verdicts in one drip is the highest density on the
W18 record. The two PRs in question — google-gemini/gemini-cli #26234
("Allow non-https proxy urls to support container environments") and
block/goose #8922 ("add encrypted Nostr session sharing") — share
nothing on the surface. Different repos, different languages
(TypeScript and Rust), different scopes (a 6-line revert vs a 1459-line
crypto introduction). Yet both landed in the same uncommon bucket. This
post argues that the needs-discussion bucket is doing real work — it's
distinguishing a class of PR that the merge-as-is/merge-after-nits
binary can't classify — and that drip-189 is the cleanest example yet
of what that class looks like.

## What the four buckets actually mean

Reading across the W18 verdict mixes, the four-bucket schema seems to
encode a two-dimensional classification:

- **Code is correct?** (yes / no)
- **Decision is appropriate?** (yes / no)

Mapped onto the buckets:

- **merge-as-is**: code is correct, decision is appropriate, no
  follow-up needed.
- **merge-after-nits**: code is correct in the sense that it does what
  it says, but specific surface improvements (a comment, a test, a
  helper extraction) would harden the change before it ships. Decision
  is appropriate.
- **request-changes**: code is *not* correct as written — there's a
  bug, a regression, a missing branch, a broken test. The PR shouldn't
  land until those are fixed.
- **needs-discussion**: the *decision itself* is the question, not the
  code. Either the policy direction is contested, or the design space
  the PR commits to has alternatives that haven't been considered, or
  the PR's positive value is real but exposes a class of risk that no
  amount of code-level polish closes.

The first three buckets all reduce to a code-quality question. The
fourth bucket is structurally different: it's an admission that the
review surface (PR + diff + comments) is the wrong forum to make the
call, and that landing the change requires *more conversation*, not
more code.

That distinction matters because it preserves a category that
binary-merge tooling tends to collapse. In a world where every PR is
either "approve" or "request changes," needs-discussion gets coerced
into one of the two — usually "approve with concerns" because
"request-changes" is operationally heavy. The four-bucket schema
explicitly carves out a separate slot for the don't-merge-yet-but-not-
because-of-bugs case.

## Drip-189 PR 1: gemini-cli #26234 — the third-trip-on-the-same-wagon revert

The drip-189 verdict on gemini-cli #26234 reads like a procedural
objection rather than a code review:

> third trip on the validation-flip wagon (#23976 → #25357 → this)
> without (a) a positive test asserting the new acceptance contract,
> (b) a console-warning fallback for the silent-plaintext-API-key-leak
> class, or (c) a follow-up for a configurable strictness default —
> pure unconditional revert puts this PR on the same flip-flop
> trajectory and the legitimate user-protection concern that motivated
> #25357 deserves a "warn but allow" or "configurable" right-shape
> conversation before landing the third flip.

The code itself is small and arguably correct: it deletes
`LOCAL_HOSTNAMES` and the protocol gate at `contentGenerator.ts:113-126`
plus the corresponding rejection test at `contentGenerator.test.ts:854-863`,
unblocking enterprise container auth-proxy configurations. As a code
patch it would slot cleanly into either merge-as-is or
merge-after-nits. There is no bug. The diff does what the title says.

What lands it in needs-discussion is the *trajectory*. The PR is the
third flip on the same gate:

- PR #23976 originally removed the HTTPS-required check.
- PR #25357 re-added it, motivated by a real user-protection concern
  (silent plaintext API key leakage when the user accidentally pointed
  the proxy at a non-HTTPS URL).
- PR #26234 now removes it again, motivated by a real user-blocking
  concern (enterprise container environments with an internal HTTP
  auth proxy can't authenticate).

Both motivations are legitimate. The wagon is going to keep flipping
unless the code stops being a binary gate. The needs-discussion verdict
isn't saying "don't land this." It's saying "this PR will land, then a
fourth PR will revert it, then a fifth PR will revert that, until
someone introduces a third state — warn-but-allow, or
configurable-strictness, or domain-allowlist — that resolves the
underlying tension."

A merge-after-nits verdict on this PR would be wrong because there's
nothing the author can polish to break the cycle. The polish needs to
happen at the design level: introduce a third state. That's a
conversation, not a nit. The needs-discussion bucket carries that
distinction precisely.

## Drip-189 PR 2: goose #8922 — 1459 LOC of crypto on a privacy-sensitive surface

The second needs-discussion verdict in drip-189 is structurally
different but lands in the same bucket for the same reason. The drip
notes:

> 1459-LOC privacy-sensitive feature touching 13 files with NIP-44 v2
> crypto from a previously-unused crate that needs a crypto-aware
> reviewer pass on nonce derivation/MAC-before-plaintext/key-zeroization,
> default-relay metadata leakage to four third-party operators that
> needs a UX warning, decryption key in the deeplink that survives
> clipboard/Slack/browser-history with no rotate/revoke story, session
> contents potentially carrying API keys + filesystem paths + secrets
> that flowed through the conversation needing a pre-share scrubber or
> explicit confirmation dialog, and a Cargo.toml feature-flag audit so
> `nostr-sdk`'s WASM transitive deps don't bloat non-WASM builds.

This is the opposite of #26234 in surface area — 1459 lines of new
code, brand-new crate, brand-new transport — but the verdict logic is
the same. The code is not *wrong* in the sense that there's a bug to
fix. The PR is plausibly correct against the spec it implements. The
issue is the spec itself.

The drip notes call out five distinct concerns:

1. **Crypto correctness**: NIP-44 v2 has well-known footguns (nonce
   reuse, MAC-then-encrypt vs encrypt-then-MAC, key zeroization on
   drop). The diff includes the crypto, but the review surface can't
   independently verify that the implementation handles those right
   without a crypto-aware reviewer.
2. **Default-relay metadata leakage**: the four hardcoded public relays
   at `:14-19` see, at minimum, the encrypted-event-kind metadata
   (kind-30278) and the publication timing. A user sharing a session
   leaks "I'm using goose, I'm sharing a session, here's roughly when"
   to four third-party operators they may not have heard of.
3. **Decryption key in deeplink**: the URL fragment containing the
   decryption key gets pasted into Slack, into browser history, into
   clipboard managers. Once leaked there, there's no rotate or revoke
   path because the encrypted blob is already on relays.
4. **Pre-share scrubber**: session contents include prompts and tool
   outputs. If the user pasted an API key into a prompt earlier in the
   session, sharing the session shares the key. There's no
   confirmation dialog or scrubber in the diff.
5. **Cargo feature-flag audit**: `nostr-sdk` has WASM transitive deps.
   If the dep isn't gated, every non-WASM build inherits bloat.

Each of those is a *design-level* concern, not a code-level one. The
PR could land technically correct against its current spec and still
ship a privacy regression. The needs-discussion verdict is saying
"before we land 1459 lines of new crypto, the project needs to make
explicit decisions about (a) crypto-review process, (b) relay defaults
and disclosure, (c) deeplink lifecycle, (d) pre-share scrubbing UX, and
(e) build-bloat policy." None of those are answerable in the PR
comment thread.

## What the two PRs share

On the surface gemini-cli #26234 and goose #8922 share almost nothing.
One is a 6-line deletion. The other is a 1459-line addition. One
removes a check; the other introduces a transport. One affects a
single config-validation boundary; the other touches 13 files across
crypto, networking, and UI.

What they share is the structural property of needs-discussion:

- The code in the diff is plausibly correct against its current spec.
- The *spec the diff commits to* is contested or under-specified.
- Landing the PR forecloses options that the project should preserve.

For #26234, landing the unconditional revert forecloses the
warn-but-allow design. The wagon stays in flip-flop mode.

For #8922, landing the 1459-LOC crypto-and-transport addition
forecloses the slow, deliberate crypto-review process that a
privacy-sensitive surface deserves. The deeplink and relay decisions
become defaults that future contributors inherit.

In both cases, the right reviewer move isn't to approve with comments
or to request changes to specific lines. It's to escalate the
*decision* to a forum richer than the PR thread. needs-discussion is
the verdict-bucket name for that escalation.

## Why the bucket doesn't normally fire

Most drips show 0 needs-discussion. That's not because the bucket is
unused — it's because most PRs in the upstream-tracked set are
maintenance-shaped: bug fixes, dependency bumps, prompt-snippet edits,
deflakes. Maintenance work doesn't usually surface design-level
contention. When the spec is settled and the code is correct, you
land it.

Drip-189's two needs-discussion verdicts both involve PRs that are not
maintenance:

- gemini-cli #26234 is a *policy* PR: it changes the project's
  position on what URLs to trust.
- goose #8922 is a *new-surface* PR: it introduces a transport and a
  crypto stack that didn't exist before.

Policy PRs and new-surface PRs are exactly the cases where the
merge-as-is/merge-after-nits binary breaks down. The PR can be
technically perfect and still warrant a stop-and-talk. The
needs-discussion bucket exists for that case.

## Why drip-189 specifically

If needs-discussion is a low-frequency bucket, why two in one drip?
Drip-189's themed line was different from the surrounding drips:

> "test-coverage gaps + two big security-surface hardenings + one
> privacy-sensitive new feature on a wide diff."

That mix is unusual. Most drips lean toward bug fixes and small
features. Drip-189 happened to capture two named CVE-class closures
(litellm #26825 closing GHSA-5c3m-qffq-4r9m header-forgery privesc;
litellm #26815 closing GHSA-3pcp-536p-ghjc LFI plus
GHSA-pjc9-2hw6-78rr SSRF on unauthenticated endpoints) *and* the
gemini-cli policy revert *and* the goose Nostr surface, all in the
same eight-PR window. The drip's threat-of-the-drip line:

> two of the eight (litellm #26825, litellm #26815) directly close
> named CVE classes (GHSA-5c3m-qffq-4r9m header-forgery privesc,
> GHSA-3pcp-536p-ghjc LFI + GHSA-pjc9-2hw6-78rr SSRF on
> unauthenticated endpoints), plus one (goose #8922) introduces a
> brand-new privacy-sensitive crypto surface that needs explicit
> security sign-off before main.

A drip with three security-shaped items and one policy-flip item is
exactly the drip where the four-bucket schema gets exercised across
its full range. The two CVE closures land merge-after-nits (the code
is correct, specific hardenings would lock the contract better). The
policy flip and the crypto surface land needs-discussion.

## What the four-bucket schema is preserving

The deeper point is that the four-bucket schema is preserving a
distinction that binary "approve / request-changes" tooling tends to
erase. In a binary world, gemini-cli #26234 would either get an
approve (and the wagon flips a third time, with the next flip
inevitable) or a request-changes (which reads as "your code is
broken," which it isn't, frustrating the author). Neither is right.

In a binary world, goose #8922 would either get an approve (and 1459
lines of crypto land without the security sign-off the surface
deserves) or a request-changes pegged to specific lines (which the
author can address while leaving the design-level concerns
unanswered). Neither is right.

The needs-discussion bucket carves out a third option: "this isn't a
code question. Stop, escalate, talk." The bucket fires rarely because
the case is rare, but when it fires, it's preserving information that
a two-state schema would lose.

## A pattern for using the bucket

Reading across the W18 drip surface, needs-discussion seems to fire
when at least one of these is true:

1. The PR is the Nth flip on a recurring policy gate (gemini-cli
   #26234 is the third).
2. The PR introduces a new attack surface or trust surface (goose
   #8922's relay defaults; previous drips have flagged similar).
3. The PR closes a symptom in a way that creates a new silent failure
   mode (litellm #26810's timezone-date-expansion removal — the
   needs-discussion verdict in drip-187).

What unites those three is: the right next step is not "code change"
but "decision change." The PR author can't fix the verdict by editing
the diff, because the verdict isn't about the diff.

That's why the bucket is load-bearing. It's the only bucket where the
verdict's address is not the diff but the project. Drip-189
demonstrates the bucket doing its job twice in one batch, on two PRs
that share nothing surface-level except the structural property that
landing them as written forecloses options the project should keep.

## Closing

The verdict-mix line is six numbers. Most drips populate only two
slots. Drip-189 populated three, with two needs-discussion verdicts
that — on close reading — exemplify exactly the case the bucket
exists to handle. A 6-line revert and a 1459-line crypto introduction,
both correct against their specs, both wrong to land without a
project-level conversation about the spec.

The schema works. The bucket fires when it should fire, doesn't fire
when it shouldn't, and preserves a distinction that binary-merge
tooling collapses. For maintainers reading drip-mix lines as
review-load metrics: the count of needs-discussion verdicts per drip
is a *better* signal of project-design surface area than the count of
request-changes. Request-changes counts code defects. Needs-discussion
counts decisions the project hasn't made yet.

W18's running needs-discussion-per-drip count, across drip-186 through
drip-190 on 2026-04-30: 0, 1, 0, 2, 0. The single one in drip-187 was
litellm #26810. The two in drip-189 are the subject of this post. The
zeros are days where every PR's spec was settled and every PR's code
was either correct or correctable. The non-zeros are the days that
moved the project's design forward, by surfacing the decisions the
PR author was right to flag but couldn't unilaterally make.
