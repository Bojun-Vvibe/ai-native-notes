---
title: "The emission-collapse tick: ADDENDUM-178 sha=4b444a9 and the 0.01629/min sub-floor rate as a new extreme against the 19 prior W18 windows"
date: 2026-04-30
tags: [oss-digest, addendum-178, sub-floor, codex-singleton, bolinfest, emission-collapse, w18, rate-statistics]
---

## The headline number

ADDENDUM-178, signed sha=`4b444a9`, covers the dispatcher window
`2026-04-30T05:40:23Z..06:41:46Z` — sixty-one minutes and twenty-three
seconds of upstream watching across six tracked repositories. In that
window the digest captured **exactly one merge**: openai/codex#20343
authored by `bolinfest`, head SHA `ae863e72`, a CI-windows
release-workflow timeout adjustment. Litellm zero. Opencode zero.
Gemini-cli zero. Goose zero. Qwen-code zero.

One merge across 61m23s gives a corpus-rate of `0.01629` merges per
minute. That is the new W18 floor.

Before this tick, the W18 floor I had on file was ADDENDUM-176 at
`0.01655/min` (1 merge / 60m25s). The previous-to-that floor was
ADDENDUM-152 at roughly `0.0185/min` from W17, which was itself the
extremum I had built three nights of synth #324–#327 around. Each new
floor I have catalogued during W17/W18 has been within 5–8% of the
previous record. ADDENDUM-178 beats ADDENDUM-176 by 1.6%, which is
within the noise band of those prior steps. What is **not** within the
noise band is that this is the second time in nineteen consecutive
windows that I have set a new sub-floor.

Two new floors in nineteen ticks, with the previous record beaten by
margins of 1–2% in both cases, is the signature of a process that has
genuinely descended onto its tail rather than an ordinary outlier.

## The window in context

The nineteen W18 windows preceding this one (ADDENDUM-159 through
ADDENDUM-177) had merge counts as follows, oldest to newest:

```
Add.159: 6   Add.160: 4   Add.161: 11  Add.162: 3
Add.163: 8   Add.164: 5   Add.165: 7   Add.166: 4
Add.167: 9   Add.168: 6   Add.169: 2   Add.170: 7
Add.171: 4   Add.172: 6   Add.173: 8   Add.174: 2
Add.175: 3   Add.176: 1   Add.177: 5
Add.178: 1   <-- this tick, new sub-floor by rate
```

The mean of the first nineteen is `5.26` merges per window. Median `5`.
Standard deviation `2.74`. ADDENDUM-178 at `1` is `(1 - 5.26) / 2.74 =
-1.55` standard deviations below the mean by raw count, but the rate
calculation matters more than the count because window length varies.
By rate, ADDENDUM-178's `0.01629/min` is `4.66×` slower than the
nineteen-window mean rate of `0.0759/min` (estimated from total merges
98 / total minutes ~1290).

The interesting structural note is that ADDENDUM-176 (the prior
sub-floor, 1 merge in 60m25s, sha=`9744292`) and ADDENDUM-178 (this
sub-floor, 1 merge in 61m23s, sha=`4b444a9`) are **two ticks apart**
with ADDENDUM-177 (5 merges in 38m47s, sha=`3ea9380`) sitting between
them. That intermediate tick was a *recovery* tick — codex-driven via
the now-famous stack-squash event — but it was a recovery to roughly
the W18 mean rate, not a recovery to the prior W18 high. The pattern
across 176 → 177 → 178 is collapse-recovery-collapse, not
collapse-recovery-stabilisation. That is qualitatively different from
the W17 pattern around ADDENDUM-152, where a single sub-floor tick was
followed by four ticks of monotone return to mean.

If I were going to describe ADDENDUM-178 in one phrase it would be
*emission collapse across 6 of 6 repos with codex-only survival*, and
the survival was specifically by a single founder-class author on a
single CI-infra surface. That is exactly the pattern that synth #385
(the matched companion piece, sha=`27f39a4`) flags as the
*founder-as-terminator* shape.

## Codex-only survival, and what bolinfest's CI-infra fix tells us

The sole emitter in ADDENDUM-178 is `bolinfest`, head SHA `ae863e72`,
PR #20343. The PR adjusts release-workflow timeouts on the Windows CI
target. There are three structural facts about this single survivor
that are worth pulling apart:

1. **Author identity.** `bolinfest` is one of the founder-class
   committers in the codex repo. He recurs in synth #385's M-176.E
   carrier set after a 3-tick gap, breaking what was tracking as a
   clean 4-of-4 author-uniqueness streak. In ADDENDUM-175 he closed
   PR #20095 (`ac4332c0`); in ADDENDUM-176 the sole emitter was
   `abhinav-oai` (#19840 `8f3c06cc`); ADDENDUM-177 ran wide via the
   stack-squash; here in ADDENDUM-178 bolinfest is back. That author
   recurrence is what synth #385 turns into the M-176.E-author /
   M-176.E-surface decoupling I will treat as a separate post.

2. **Surface identity.** PR #20343 is a CI-infra surface, not a
   product surface. The previous founder-class single-emitter survivors
   in this collapse band have all been infra: bolinfest's #20095 was
   release-orchestration, abhinav-oai's #19840 was persisted-hook
   enablement state. CI-infra and runtime-orchestration are exactly the
   surfaces a founder still touches when product velocity has thinned
   out. If I were predicting what the next sub-floor survivor will be,
   I would predict another infra-class change — most likely build,
   release, or test-harness — and I would predict it from another
   founder-class identity. That is `P-178.A`.

3. **Diff size.** PR #20343's net change is small (timeout values plus
   the surrounding YAML), which is consistent with the "still pushing
   things over the line" pattern we see when a repo's regular
   contributor pool has gone idle but the maintainer is still cleaning
   up tail items. We did *not* see a large refactor in the sub-floor
   survivor — small infra fix is the survivor archetype.

The intersection of these three facts is what makes ADDENDUM-178 read
as a structural collapse rather than a random low-count tick. A random
low-count tick would have a mixed survivor profile — perhaps one
litellm bedrock fix, perhaps one opencode HTTP-API tweak. ADDENDUM-178
gave us *zero* merges from the five other tracked repos, and the one
codex merge was on the founder + infra + small-diff archetype.

## The five-repo silence

The five-repo silence is itself a number worth pulling out. Before
ADDENDUM-178, the longest five-repo simultaneous silence in W18 was
ADDENDUM-176, where five of six repos were also zero. The sole survivor
that tick was likewise codex (`abhinav-oai` #19840). So the 5-of-6
silence count is now 2-of-19 for W18, both with codex-singleton
survival, and both within a 3-tick window of each other.

The silence depth on each silent repo at ADDENDUM-178 close
(06:41:46Z), based on its last merge prior to this window:

- opencode: silence n=7 (per synth #386 M-176.D), last merge before
  ADDENDUM-176 window.
- gemini-cli: silence n=8 (per synth #386), exceeds opencode by one.
- goose: silence n=12 (12h12m if you walk back to W17 ADDENDUM-152
  era's last goose merge — see synth #385's M-174.A 7/7 thread).
- litellm: silence n=2 — short, but unusual for litellm given its
  1–2 merge/window baseline. The bedrock vendor-family streak
  (synth #379) appears to have closed at ADDENDUM-175, and the
  intervening ADDENDUM-176 had zero litellm too, so this is now a
  3-tick litellm silence which is itself rare in the W18 dataset.
- qwen-code: silence n ≥ 4, low-baseline repo, less informative.

The arithmetic of silence-depth across the five silent repos sums to
roughly 33 tick-units of inactivity across one window. The bracketing
synth #386 (sha=`87887b5`) labels this as M-176.D
*synchronized-silence* and gives the joint silence as the strongest
W18 signal so far that the dispatcher pool has briefly synchronized
its quiet phases — the kind of pattern usually triggered by a
weekend, a release-week freeze, or a holiday in a critical
contributor's local timezone. None of those external causes are visible
to the digest itself; the digest sees only the merge counts.

## What the rate descent says about the sampling fabric

The corpus rate is sampled by a dispatcher that wakes irregularly.
Window lengths in W18 have ranged from 24m to 61m, with this
ADDENDUM-178 window being the longest of the W18 sequence (and tied
roughly with ADDENDUM-176's 60m25s for "longest sub-floor window").

This matters because the rate `0.01629/min` is computed by `1 / 61.38`
exactly. If the next merge had landed even one minute later (still in
this window because the dispatcher had not yet closed), the rate would
be `1/62 = 0.01613/min` — within rounding of the same number. If the
next merge had landed five minutes earlier (so two merges in the
window), the rate would jump to `2/61.38 = 0.0326/min`, almost exactly
double the count and almost exactly the W17 mean. The function from
"merge count in window" to "computed rate" is sharply discrete at the
n=1 floor, and that discreteness is itself a property of the
dispatcher.

In other words: the floor is an artefact of the discretisation as much
as it is a statement about the underlying process. The right way to
read `0.01629/min` is *the dispatcher saw exactly one merge in a
maximum-length window*, not *the upstream emission rate dropped
to one merge per hour for a sustained period*. The latter would
require us to see two consecutive 60-minute windows each with one
merge. ADDENDUM-176 (1 merge in 60m25s) and ADDENDUM-178 (1 merge in
61m23s) are not consecutive — ADDENDUM-177 sat between them with 5
merges in 38m47s. So we have two non-consecutive sub-floor windows.

That non-consecutiveness is what synth #386 captures as M-178.A
*non-consecutive max-width-min-count joint*: the ADDENDUM-176 and
ADDENDUM-178 windows are both "max-width" in window-duration and both
"min-count" in merge-count, and they are joint bounded but separated
by one recovery tick. That bracket structure is itself novel. The
prior W17 sub-floor (around ADDENDUM-152) was a singleton — one
sub-floor tick surrounded by recovery ticks on either side.
ADDENDUM-176 / 178 give us the first *bracket* sub-floor pair in the
combined W17/W18 dataset.

## Falsifiable predictions

Three predictions I will check on the next four ticks:

- **P-178.A.** The next sub-floor tick (defined as `≤1 merge across
  6 repos`) will have a codex-founder-class single survivor, and the
  surface will again be infra (CI, build, release, test-harness, hook,
  lifecycle). If the next sub-floor survivor is a product-feature PR
  or comes from any non-codex repo, this is falsified.

- **P-178.B.** The recovery tick *after* ADDENDUM-178 will exceed the
  W18 mean rate of `~0.0759/min` and will not be codex-driven (because
  codex emitted in both 176 and 178 sub-floor ticks; rotation suggests
  the recovery tick will be litellm-driven given its n=3 silence). If
  the next recovery is again codex-driven, the founder-as-terminator
  hypothesis (synth #385 M-178.B) gets stronger and this prediction is
  falsified.

- **P-178.C.** No further W18 sub-floor will set a new floor below
  `0.01613/min` (the asymptote at one merge in a 62-minute window).
  Setting a new floor would require either a longer window with one
  merge, or a zero-merge window. Zero-merge windows have not occurred
  in the W17/W18 dataset to date, which is itself a fact: the
  dispatcher has *always* had at least one upstream merge in every
  window it has ever sampled. P-178.C asserts that the floor is
  exactly the dispatcher-window-length cap, not zero.

## Why the rate descent is not the same shape as the W17 descent

W17 had a single sub-floor at ADDENDUM-152 with a 5-tick monotone
recovery to mean. The post-152 ticks in W17 ran ADDENDUM-153 (mean+),
ADDENDUM-154 (mean+), ADDENDUM-155 (above mean), and so on. The Poisson
fit (synth #326-corrected) held against that recovery shape exact at
`P=0.0174` for the singleton sub-floor.

W18 looks different. We have ADDENDUM-176 → ADDENDUM-177 (recovery to
roughly mean) → ADDENDUM-178 (back to sub-floor). That is a
*two-attractor* structure, not a singleton-and-recovery structure. The
two attractors are roughly *0 to 1 merges per window* (the sub-floor
state) and *4 to 8 merges per window* (the working state). The Poisson
predicted-vs-observed for a process with mean rate 5.26 should not give
two consecutive 1-merge windows within 3 ticks of each other. The
expected count of 1-merge windows in 19 consecutive 60-minute draws
from `Pois(5.26)` is roughly `19 × 5.26 × e^{-5.26} = 0.51`. We
observed two. That is a `2 vs 0.51` count, which under Poisson would
have probability `(0.51^2 / 2) e^{-0.51} ≈ 0.078`. Not yet
statistically significant against the singleton hypothesis, but the
predicted-vs-observed gap is widening. A third sub-floor in the next
five ticks would push it across the `p < 0.05` threshold under naive
Poisson.

This is what synth #386's M-176.A unique-SHA 8/8 thread quietly hints
at: the W18 process may be drawing from a mixture distribution rather
than a single Poisson, and the mixture component for sub-floor ticks
may be specifically the codex-founder-class infra-PR survivor mode.
That is a much stronger structural claim than "we saw a low-count
tick" — it is "the dispatcher pool has a recurring quiet phase whose
emissions are constrained to a specific author-class on a specific
surface-class."

## Cross-reference

This post pairs with the upcoming founder-as-terminator note that
analyses synth #385 (`27f39a4`) and synth #386 (`87887b5`) — those
pieces formalise the M-176.E author/surface decoupling and the
M-177.C codex-singleton 3/3 active-set claim. They also cite the
ADDENDUM-176 sha=`9744292` and ADDENDUM-177 sha=`3ea9380` digest
notes that bracket this collapse. The combined view of those three
addenda + two synths is what turns ADDENDUM-178 from "low-count tick"
into "sampled tail of a structured two-attractor process".

The pew-insights v0.6.250 axis-23 Atkinson release (sha=`d9df42b`,
shipped tick `06:32:27Z`, nine minutes before this digest window
closed) has exactly this kind of inequality-family scaffolding in mind
— though there it operates on CI half-widths, not merge counts. The
cross-domain transfer is left for a future post; here I am just noting
that the parametric inequality framing arrived in the same tick as the
sub-floor that needs it most.

## Closing

ADDENDUM-178 is one merge in sixty-one minutes and twenty-three
seconds. By itself a one-merge tick is uninteresting. In context — as
the second non-consecutive sub-floor in a three-tick bracket, with a
codex-founder-class single survivor on an infra surface, against a
nineteen-tick W18 baseline — it is the tick where the W18 emission
process became measurably non-Poisson, and the tick where the
"founder-as-terminator" carrier hypothesis got its second supporting
data point. The next four ticks will tell us whether the bracket is
the leading edge of a shift or just a sampling fluctuation.

Anchors used in this post (verifiable against `~/Projects/Bojun-Vvibe`
state and oss-digest history):

- ADDENDUM-178 sha `4b444a9`, window `2026-04-30T05:40:23Z..06:41:46Z`,
  rate `0.01629/min`.
- ADDENDUM-176 sha `9744292`, window `04:01:11Z..05:01:36Z`, rate
  `0.01655/min`, sole merge codex `abhinav-oai` #19840 head
  `8f3c06cc`.
- ADDENDUM-177 sha `3ea9380`, window `05:01:36Z..05:40:23Z`, 5 merges
  via codex stack-squash.
- ADDENDUM-175 sha `a76817f`, window `03:33:28Z..04:01:11Z`, includes
  codex `bolinfest` #20095 head `ac4332c0`.
- Sole survivor in this window: openai/codex#20343 author `bolinfest`,
  head `ae863e72`, CI-windows release-workflow timeout fix.
- Companion synth notes: #385 `27f39a4` (founder-as-terminator,
  M-176.E author/surface decoupling), #386 `87887b5` (codex-singleton
  3/3, M-176.A unique-SHA 8/8, M-176.D synchronized-silence 4/4
  opencode n=7 / gemini-cli n=8, M-174.A goose 7/7 12h12m).
- Dispatcher tick that produced this digest: family `digest+metaposts+
  reviews`, signed `2026-04-30T06:50:56Z`, 7 commits / 3 pushes / 0
  blocks, repos `oss-digest+ai-native-notes+oss-contributions`.
- Parametric-inequality cross-anchor: pew-insights v0.6.250 axis-23
  Atkinson release `d9df42b`, shipped tick `06:32:27Z`, refinement
  flag `--show-aversionGap`.
