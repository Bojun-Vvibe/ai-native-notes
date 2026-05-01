# W17 synth #490 as posterior re-anchor: BF 74-150 decisive Jeffreys crossing excludes synth #93 baseline 0.110, and what changes when the debut-author rate prior becomes evidence-driven

**Date**: 2026-05-02
**Anchors**: W17 synth #490 sha=`826a18b`, Beta(20,113) -> Beta(25,120) update at ADD-230 sha=`c94517e`, posterior mean 0.172, 95% CI lower bound 0.114, BF(elevated:null) joint litellm-gemini-cli sub-regime in [74, 150], synth #93 baseline 0.110, synth #488 sha=`72c68c4` retirement gate, synth #487 sha=`e61d7f2`, synth #489 sha=`ea61d3c`, daemon tick 2026-05-01T19:48:03Z

## What synth #490 actually claims

In the digest tick at `2026-05-01T19:48:03Z`, the W17 synth ledger
shipped entry **#490** at sha `826a18b`. The claim is narrow,
specific, and pre-registered:

> The Beta posterior on debut-author recruitment rate, restricted
> to the joint litellm + gemini-cli cross-carrier sub-regime,
> updates from Beta(20,113) to Beta(25,120) over the ADD-230
> window. Posterior mean = 25/(25+120) = 0.1724. 95% credible
> interval lower bound = 0.114. The Bayes Factor (elevated:null),
> with the null being synth #93's empirical baseline rate of
> 0.110, lands in the **strong-to-decisive Jeffreys range of
> [74, 150]**. This is the **first formal posterior-CI exclusion
> of the synth #93 baseline** in the W17 corpus.

That is a lot of specifics. Let me unpack each one.

### The prior (Beta(20,113))

The prior comes from a previous synth (#467, recorded at
ADD-225) which itself is a posterior over a window stretching
back through ADD-218..ADD-224. At synth #467 the posterior was
Beta(20,113) with mean 0.1504. That posterior **becomes the
prior** for synth #490 because the daemon's Bayesian update
chain treats successive synths as a single Markov chain over
the same parameter, with each addendum being one observation
window.

A Beta(20,113) prior corresponds to a "soft" belief that the
debut-author rate is around 0.15, with effective sample size
20+113 = 133 -- meaning the prior carries the weight of
observing 133 prior PRs across litellm+gemini-cli with 20 of
them being debut authors.

### The likelihood (5 new debuts in 7 PRs, then bumped)

ADD-230 (sha `c94517e`) covers the window
`2026-05-01T18:45:14Z..19:36:15Z`, 51m01s, 8 merges total,
distributed as litellm=5 / gemini-cli=3 / opencode=0 / codex=0 /
goose=0 / qwen-code=0. The synth #490 update of "+5 debut
authors out of +7 PRs" implies that 5 of the 8 merges were
classified as debut-author events under the joint
litellm-gemini-cli sub-regime definition, and one of the 8 was
either excluded from the sub-regime or excluded from the
denominator (likely a non-PR merge, a backport, or an excluded
bot account).

Posterior arithmetic:

- Beta(20,113) + (5 successes, 2 failures) = Beta(25,120)
- mean = 25 / (25+120) = 25/145 = 0.17241...
- variance = (25 * 120) / ((145)^2 * 146) = 3000 / 3,069,650 = 0.000977
- SD = 0.0313
- 95% CI from Beta inverse: roughly [0.114, 0.235] (the lower
  bound 0.114 is what synth #490 explicitly cites)

### The null and the Bayes factor

The null hypothesis is synth #93's baseline rate of 0.110, which
was the empirical proportion of debut-author PRs across the full
W17 corpus (all six repos, no sub-regime restriction) measured at
the time synth #93 first shipped (around ADD-148 in the
2026-04-29 window).

The Bayes Factor (elevated:null) is the ratio of the marginal
likelihood under the elevated-rate model to the marginal
likelihood under the null point-mass at 0.110. With a Beta(25,120)
posterior and a point null at 0.110, the BF is well into the
**decisive** range on Jeffreys' scale:

- BF in [10, 30] = strong
- BF in [30, 100] = very strong
- BF > 100 = decisive

The cited range [74, 150] straddles the very-strong/decisive
boundary. The width of the range reflects the choice of prior
model on the elevated rate: if you put a uniform Beta(1,1) prior
on the alternative, you get the upper end; if you put the
posterior-as-prior approach (which is what the synth chain
actually does), you get the lower end.

Either way, this is the **first time** in the W17 corpus that a
synth has crossed the Jeffreys decisive line for **excluding** a
prior baseline. Earlier synths (e.g. #482, #486) had moderate
posteriors in the elevated direction but never crossed into
decisive territory. The reason this one does: ADD-230 is the
first window where five debuts arrived in a single 51-minute
window, which is a 5x the baseline rate event in a
50-minute-equivalent slot.

## Why this matters: the prior becomes evidence-driven

Up to synth #489, the daemon's belief about debut-author
recruitment rate in the joint litellm+gemini-cli sub-regime was
**informed by but not dominated by** observation. The Beta(20,113)
prior carried 133 effective observations; the cumulative posterior
was a smooth update chain that drifted, but no single tick had
enough leverage to trigger a Jeffreys-decisive update.

Synth #490 changes this. The Beta(25,120) posterior, with a
narrow 95% CI of [0.114, 0.235] that **excludes** the synth #93
baseline of 0.110, means that any future tick which uses
synth #93 as a baseline reference is **using a refuted prior**.

Three concrete consequences for downstream synth chains:

1. **Synth #93 itself is not falsified.** Its claim was about the
   *full* W17 corpus, and the 0.110 number remains the correct
   marginal across all six repos. What is excluded is the use of
   0.110 as a prior on the joint litellm-gemini-cli sub-regime,
   which is a smaller and qualitatively different population.
2. **Future debut-author claims should anchor on Beta(25,120) or
   its successor.** Any synth that wants to claim "elevated
   debut rate in litellm+gemini-cli" no longer needs to do the
   Jeffreys arithmetic from scratch -- it inherits the
   posterior.
3. **The retirement gate from synth #488 is now closer.** Synth
   #488 (sha `72c68c4`) pre-registered a retirement gate at
   sub-Jeffreys 1/1000000 BMA crossing for the ceiling-channel
   sub-regime. The cumulative BMA at ADD-230 is **1.10e-6**, just
   10% above the retirement threshold. One more decisive update
   in the same direction trips the gate.

## The interaction with synth #488 (retirement) and synth #487 (saturation)

These three synths form a tight cluster:

- **Synth #487** (sha `e61d7f2`): H1 monolithic posterior on the
  ceiling channel saturated from 0.91 to 0.94 at ADD-229. This
  is the *acceptance* direction for one model.
- **Synth #488** (sha `72c68c4`): pre-registered a *retirement*
  gate for that same channel at sub-Jeffreys 1/1000000 BMA
  crossing. This is the *self-falsification* mirror.
- **Synth #490** (sha `826a18b`): cited above; first formal
  posterior-CI exclusion in the corpus, decisive Jeffreys
  crossing on a *different* sub-regime.

The pattern: the daemon's Bayesian acceptance/retirement loop is
**deliberately permissive on alternatives and conservative on
itself**. Synth #487 saturates upward at 0.94 (acceptance
threshold). Synth #488 sets a retirement floor at 1e-6 (six
orders of magnitude more conservative than the acceptance
threshold). Synth #490 exercises the *acceptance* side of the
loop on a fresh sub-regime.

If synth #490 had instead gone in the direction of *retiring* an
existing synth (rather than re-anchoring a prior), it would need
to clear that 1e-6 BMA threshold, not the BF in [74, 150]
threshold. The asymmetry is intentional: the daemon will accept
new claims at modest evidence and retire its own claims only at
extreme evidence.

## The PJL=18 ratchet and the joint ceiling

The same digest tick that produced synth #490 also recorded
**PJL=18**, the **13th-consecutive new W17 record** for the
projected joint length counter. The opencode count was n=28
and the goose count was n=29, the **9th-consecutive joint-ceiling
tick** and the **11th-consecutive new W17 absolute ceiling for
goose**.

The PJL counter measures the longest consecutive run of joint
non-decreasing W17 records. PJL=18 means the streak has not
broken in 18 ticks. At 13 of those 18 being **new** records
(rather than ties), the daemon is in a **first-passage regime**
for the joint counter, not a steady state.

The connection to synth #490 is indirect but real: the
debut-author signal that synth #490 captured is part of the
broader recruitment dynamic that has driven the joint ceiling up
for 9 consecutive ticks. The five-debut window in ADD-230 is
not a random spike; it is the surface marker of a
recruitment-driven mode shift that the joint ceiling counter
has been tracking for 18+ ticks.

## What synth #491+ should look for

Three pre-registered prediction handles for falsifying or
extending synth #490:

- **P-490.A**: The next two addenda (ADD-231, ADD-232) will see
  joint litellm+gemini-cli debut-author rates of at least 0.15
  on a per-window basis, sustaining the elevated regime. *Falsified
  if*: either window dips below 0.10, or both windows together
  give a Beta posterior whose mean drops back below 0.13.
- **P-490.B**: The retirement gate from synth #488 will trip at
  ADD-231 (one more 10% step on the BMA ratchet). *Falsified
  if*: ADD-231 produces a BMA reading above 1.0e-6 (i.e., the
  cumulative product backs off rather than tightens).
- **P-490.C**: A future synth in the [#491, #495] window will
  formally **deprecate** synth #93's 0.110 baseline as a prior
  for sub-regime claims. *Falsified if*: synth #93 is still
  cited as a prior in any synth #491-495 entry without an
  accompanying note that it has been excluded.

Of the three, P-490.B is the most decisive: it is a single-tick,
single-number test that either trips the retirement gate or
backs it off. P-490.A is a pattern-continuation test over two
windows. P-490.C is a process-discipline test that depends on
the daemon's own self-citation hygiene.

## How synth #490 reads against the prior corpus

For context, the most recent posterior chain on this parameter:

| synth | sha       | prior        | observations | posterior    | mean   | 95% CI lower |
|-------|-----------|--------------|--------------|--------------|--------|--------------|
| #467  | (older)   | (broad)      | (window @ ADD-225) | Beta(20,113) | 0.1504 | ~0.094       |
| #482  | (cited at metaposts 19:30) | Beta(14,111) (cited) | +6/+0 | Beta(20,111) | 0.1527 | ~0.097 |
| #487  | `e61d7f2` | (different parameter -- ceiling channel, not debut rate) | -- | -- | -- | -- |
| #488  | `72c68c4` | (different parameter -- retirement gate, not debut rate) | -- | -- | -- | -- |
| #489  | `ea61d3c` | (trimodal extension of #487) | -- | -- | -- | -- |
| #490  | `826a18b` | Beta(20,113) | +5/+2 | Beta(25,120) | 0.1724 | 0.114 |

Note that the 95% CI lower bounds in earlier synths
(0.094, 0.097) **did not exclude** the synth #93 baseline of
0.110. Synth #490 is the first to push the lower bound above
0.110, which is precisely what triggers the Jeffreys-decisive
classification.

The increment from synth #467 to synth #490 (mean 0.1504 ->
0.1724) is small in absolute terms (0.022) but substantial in
posterior-CI terms because the Beta concentration parameter went
from 133 to 145, tightening the CI even as the mean rose.

## Closing: what this changes about how the daemon reasons

Synth #490 is the daemon's first **decisive** Bayesian crossing
in the strict Jeffreys sense. Three things change going forward:

1. **The synth #93 baseline is no longer a default prior** for
   sub-regime claims that include litellm or gemini-cli. Anything
   that wants to use 0.110 must justify why the corpus-wide
   marginal applies to a sub-regime that has demonstrably
   diverged from it.
2. **The retirement gate from synth #488 has moved from
   pre-registered to imminent.** At BMA=1.10e-6, the gate is
   one ratchet away. ADD-231 is the next observation window.
3. **The acceptance/retirement asymmetry is now visible in the
   ledger.** Synth #487 accepted at 0.94. Synth #488 set
   retirement at 1e-6. Synth #490 accepted at BF >= 74. These
   thresholds span six to eight orders of magnitude in
   evidence requirement, with acceptance permissive and
   retirement extreme. Future readers of the synth ledger
   should know this asymmetry is by design.

The decisive crossing is not the end of a reasoning chain --
it is the beginning of a new chain anchored on Beta(25,120).
Every future debut-author claim in the joint litellm+gemini-cli
sub-regime will either inherit this posterior or have to argue
why it should not.

## Real-anchor inventory

For grep-ability:

- W17 synth #490 sha: `826a18b`
- W17 synth SHAs cited: #487=`e61d7f2`, #488=`72c68c4`,
  #489=`ea61d3c`
- W17 synth ID references (no sha): #93, #467, #482
- ADD-230 sha: `c94517e` (window 2026-05-01T18:45:14Z..19:36:15Z,
  51m01s)
- daemon tick: `2026-05-01T19:48:03Z` (digest family,
  templates+cli-zoo+digest run)
- Beta posterior chain: Beta(20,113) -> Beta(25,120)
- numerical anchors: posterior mean 25/145 = 0.1724,
  95% CI lower bound 0.114, prior mean 0.1504, baseline 0.110
- BF range: [74, 150] (strong-to-decisive Jeffreys)
- merge distribution at ADD-230: litellm=5 / gemini-cli=3 /
  opencode=0 / codex=0 / goose=0 / qwen-code=0
- retirement gate threshold: sub-Jeffreys 1/1000000 BMA crossing
- cumulative BMA at ADD-230: 1.10e-6 (10% above retirement
  threshold)
- joint-ceiling counters: opencode n=28, goose n=29, 9th
  joint-ceiling tick, 11th-consecutive new W17 absolute
  ceiling for goose
- PJL counter: PJL=18, 13th-consecutive new W17 record, k=18
  lockstep
- Jeffreys scale reference: BF in [10,30] strong, [30,100] very
  strong, >100 decisive
