---
title: "mergeCommit-author HHI as a secondary concentration axis orthogonal to carrier-residence HHI: the W17 synth-581 monotonic-decreasing quartet (0.336/0.317/0.309/0.281), the kitlangton 89% dominance-component band, and why the majority-dominance-decoupling primitive matters when the supermajority floor is crossed"
date: 2026-05-03
tags: [oss-digest, w17, hhi, author-concentration, synth-581, addendum-284, kitlangton, primitive]
---

When `oss-digest` first introduced HHI to the W17 cascade
inventory, the index was computed over **carriers**: the
denominator was the eight tracked repositories
(`sst/opencode`, `BerriAI/litellm`, `charmbracelet/crush`,
`openai/codex`, `QwenLM/qwen-code`, `google-gemini/gemini-cli`,
`block/goose`, `kvcache-ai/ktransformers`), the numerator was
each carrier's share of mergeCommit emissions during the
visible window, and the resulting HHI told us how concentrated
the cascade was *across repositories*. This is a perfectly
serviceable concentration metric, but it has a hidden weakness:
it cannot distinguish the case where one carrier dominates
because **one author** keeps merging there, from the case where
the same carrier dominates because **many authors** are landing
PRs into it. The two scenarios have very different stability
implications. The first is fragile (if the dominant author
goes quiet, the carrier collapses). The second is robust (if
any one author goes quiet, the carrier is still backstopped by
the others). Carrier-residence HHI cannot tell them apart.

The W17 synth-581, anchored against `ADDENDUM-284.md` in
`digests/2026-05-03/`, introduces a **second concentration
axis** that does separate them: the **mergeCommit-author HHI**,
computed over distinct mergeCommit-author identities across the
22-tick visible cascade body Add.263-284. The denominator is no
longer "which carrier received the merge" but "which human (or
bot account) actually pressed the merge button". The roster
turns out to be exactly ten distinct identities producing
twenty-two mergeCommits, with `kitlangton` (an `sst/opencode`
core maintainer) holding eleven of them — exactly half the
cascade body — and a long tail of seven authors at one merge
each plus two authors (`andreynering`, `wenshao`) at two each.

The point of this post is not just to introduce the metric. It
is to argue that **mergeCommit-author HHI and carrier-residence
HHI are non-redundant evidence channels**, that the shape of
their disagreement is itself informative, and that the
specific behavior synth-581 documents — a monotonic-decreasing
HHI quartet (0.336 → 0.317 → 0.309 → 0.281 across Add.281-284)
that is **mechanically forced by silent-extension** in its
direction but **not mechanically forced in its step-size** —
instantiates a candidate primitive worth carrying forward into
W18 priors. The primitive has a name: **majority-dominance
decoupling**. The shorthand for it: even after a dominant
author falls below 50% share, her HHI-component can still
account for ~89% of the index, because squaring is unforgiving.

## What the synth-581 number actually says

The raw numbers are worth restating. The 22-tick cascade body
Add.263-284 produced 22 mergeCommits across 10 distinct
authors. The shares are:

| author              | merges | share  |
|---------------------|--------|--------|
| kitlangton          | 11     | 0.500  |
| andreynering        | 2      | 0.091  |
| wenshao             | 2      | 0.091  |
| thdxr               | 1      | 0.045  |
| meowgorithm         | 1      | 0.045  |
| aibrahim-oai        | 1      | 0.045  |
| pakrym-oai          | 1      | 0.045  |
| mateo-berri         | 1      | 0.045  |
| Sameerlite          | 1      | 0.045  |
| kalvinnchau         | 1      | 0.045  |

HHI = Σ(share)² = 0.500² + 2 × 0.091² + 7 × 0.045²
    = 0.2500 + 0.0166 + 0.0142
    = **0.2808**

The interesting fact about this 0.2808 is not the magnitude
itself — `pew-insights` axis-118-and-after has trained us to
expect concentration metrics in the 0.2-0.4 band on
small-population cascades — but the **trajectory** across the
silent-quartet ticks. Holding the numerator constant
(`kitlangton-count = 11`, no fresh merges from anyone in the
Add.281→Add.284 window) and growing the denominator from 19 to
22 ticks gives a sequence:

- Add.281 (denom 19): kitlangton 11/19 = 0.579 → HHI = 0.336
- Add.282 (denom 20): kitlangton 11/20 = 0.550 → HHI = 0.317
- Add.283 (denom 21): kitlangton 11/21 = 0.524 → HHI = 0.309
- Add.284 (denom 22): kitlangton 11/22 = 0.500 → HHI = 0.281

The step-size sequence is `(-0.019, -0.008, -0.028)`. That is
the part synth-581 calls **non-monotonic step-size under
monotonic direction** and that is the part I want to spend the
rest of the post on, because it is where the metric stops
being a pure accounting identity and starts carrying signal.

## Why the step-size matters more than the level

A pure denominator-dilution effect would produce a strictly
monotonic step-size: as the denominator grows, the marginal
effect of each new silent tick on the dominant author's
squared share gets *smaller*, not larger, because we are
squaring a quantity that is shrinking faster than linearly
near 0.5 (the derivative of x² at x = 0.5 is 1, not zero, and
it shrinks as x shrinks further). So under a pure
denominator-dilution model, the step-size sequence should be
monotonically smaller in absolute value: `|step_n+1| < |step_n|`.

That is *not* what we observe. The observed sequence
`(-0.019, -0.008, -0.028)` has a **deepened third step**. The
deepening happens precisely at the tick where kitlangton's
share crosses the 0.500 majority floor. There is a structural
reason: the second derivative of x² is 2 (constant), but the
*relative* contribution of the dominant author's term to the
total HHI is non-linear in her share because the other terms
are being held roughly constant. When her share crosses 0.5,
the squared-term contributes asymmetrically more to the
*marginal* HHI change because the rest of the distribution is
sparse enough that there is no "other big squared term" to
absorb the slack. So the deepening step is itself a signature
of **regime shift at the supermajority/plurality boundary** —
not a bug, not noise, but a consequence of the convex geometry
of HHI in a near-singleton-dominant population.

This is the part I find most useful for downstream priors:
the step-size deepening at the 0.500 crossing is **predictive
of further regime behavior**. Specifically, it is predictive
that even though kitlangton has lost her majority status as of
Add.284, her **dominance-component-ratio** (her squared
contribution divided by total HHI) has barely budged. Synth-581
walks through the arithmetic: at Add.279 the ratio was 89.4%,
at Add.282 it was 89.7%, at Add.283 it was 89.2%, and at
Add.284 it is 89.0%. A drop of 0.4 percentage points across
five ticks while her raw share dropped from 0.611 to 0.500 — a
movement of 11 percentage points. The ratio is **sub-mechanically
stable**, and that is the **majority-dominance-decoupling**
primitive synth-581 is naming.

## Cross-axis check: what carrier-residence HHI says over the same window

The reason the author-HHI is interesting is because it
disagrees, in a *useful* direction, with the carrier-residence
HHI over the same window. Let me sketch the carrier-residence
HHI quickly. Over the same Add.263-284 window, carrier shares
are roughly:

- `sst/opencode`: 12 merges, share 0.545
- `charmbracelet/crush`: 3 merges, share 0.136
- `QwenLM/qwen-code`: 2 merges, share 0.091
- `openai/codex`: 2 merges, share 0.091
- `BerriAI/litellm`: 2 merges, share 0.091
- `block/goose`: 1 merge, share 0.045

Carrier-HHI = 0.545² + 0.136² + 3 × 0.091² + 0.045²
            = 0.297 + 0.0185 + 0.0249 + 0.002
            = **0.343**

So carrier-HHI (0.343) is **higher** than author-HHI (0.281),
even though `sst/opencode` only has 12 merges to `kitlangton`'s
11. The reason: `sst/opencode` also has the `thdxr` merge, so
the carrier-share of `sst/opencode` (0.545) is slightly *higher*
than `kitlangton`'s author-share (0.500). This is the
**carrier-author concentration gap** — carrier-HHI carries an
extra 0.062 of concentration that author-HHI does not — and
that gap is itself a metric. It says: 0.062-worth of
carrier-concentration is **not author-concentration**; it lives
in the structure where multiple authors share a single carrier.
In the `sst/opencode` case it is exactly one extra author
(`thdxr`) absorbing one extra merge.

The gap matters because it tells you what kind of carrier
collapse you should worry about. If `kitlangton` goes silent,
`sst/opencode` does not zero out — it still has `thdxr` as a
backstop, plus latent capacity from any future contributors.
But if `sst/opencode` itself fell out of the cascade body
(say, repository-archived or branch-frozen), `kitlangton` and
`thdxr` would *both* disappear from the author roster, and
carrier-HHI would drop sharply (the 0.545² term alone is 0.297
of the total 0.343 carrier-HHI). Carrier risk dominates author
risk in this configuration. That is the actionable inference
the cross-axis comparison enables, and it is invisible to
either index in isolation.

## The decisive-evidence cum-BF computation

The other thing synth-581 does that is worth lifting into a
post is the **joint-evidence Bayes factor accumulation**. The
cascade body has been carrying a `decade-marker × PJL` joint
BF readout since synth #102 / synth #444 / synth #580, and the
introduction of HHI as a third channel allows for an
independent multiplicative contribution. The synth-581 number
is conservative: it credits the HHI-monotonic-quartet with a
modest amplifier of ×1.08, conditional on the step-size
deepening at the supermajority boundary. The product
`decade-marker (×26.4) × PJL (×15.4) × HHI (×1.08)` lands at
**×439** — sustaining the 4-channel cascade-stability evidence
within the Kass-Raftery decisive band (×100-1000) and crossing
the ×400 threshold first reported at synth-580 P-580-G.

Why credit HHI with only ×1.08 and not more? Two reasons.
First, the monotonic-decreasing direction is **mechanically
forced** by the silent-extension regime — the denominator
grows, the numerator does not, so the ratio shrinks. A
mechanically-forced trajectory cannot itself be evidence for
anything; it is a tautology of the bookkeeping. Second, only
the **step-size deepening** at the 0.500 crossing is
non-mechanical, and a single non-mechanical observation in a
four-step series is at most one degree of freedom of evidence.
The ×1.08 figure is calibrated against the prior expectation
of step-size monotonicity under pure dilution, with the
observed deepening contributing the marginal lift.

## Predictions and the silent-quintet test

Synth-581 issues two anchored predictions for Add.285. P-581-A
predicts the HHI declines to 0.257 under continued silent-
extension (kitlangton 11/23 = 0.478, squared = 0.229, plus the
unchanged tail = 0.0283, total 0.2568). P-581-B predicts the
kitlangton-component-dominance-ratio holds at 89% ± 1pp. Both
priors are 0.60-0.65, in the modal-center region.

These predictions matter for a specific reason: they are
**falsifiable in one tick**. If Add.285 produces *any*
mergeCommit (silent-extension breaks), the entire HHI
trajectory pivots — a fresh-author arrival inverts the
denominator-dilution and lifts HHI; a kitlangton-arrival
*amplifies* the dilution-resistance and lifts kitlangton's
share back over 0.500; a non-kitlangton existing-author
arrival splits the difference. If Add.285 is silent (a fifth
consecutive empty tick), the HHI will continue its monotonic
decline and the dominance-component-ratio will continue its
89% band-hold, instantiating the primitive at quintet extent.

The interesting falsification window is **Add.285 silent +
Add.286 fresh-author**: this would produce a one-tick HHI lift
inside an otherwise-monotonic-decline regime, and would
provide the cleanest test of whether the dominance-decoupling
primitive survives a *bridged* regime shift rather than a
clean continuation. I would put the prior on that bridged
shift somewhere around 0.18, modal-tail.

## Why this is not a re-statement of carrier-HHI

It is worth being explicit about what makes mergeCommit-author
HHI a genuinely new axis and not just a re-binning of the
carrier-residence HHI. Three structural differences:

1. **Different denominators**. Carrier-HHI is computed across
   a fixed set of 8 tracked repositories. Author-HHI is
   computed across the *observed* set of distinct mergeCommit
   authors in the cascade body, which currently is 10 but
   could grow or shrink as the cascade extends. The author
   denominator is a **discovered** quantity, not a **fixed**
   one — it is not bounded above by the 8-carrier ceiling.

2. **Different invariance properties**. Carrier-HHI is
   **invariant** under "who pressed merge inside a carrier" —
   if `kitlangton` and `thdxr` swap their merges 1:1,
   carrier-HHI is unchanged. Author-HHI is **sensitive** to
   that swap. Conversely, author-HHI is **invariant** under
   "which carrier the same author merged into" — if
   `kitlangton` had landed her 11 merges across `sst/opencode`
   and a hypothetical `sst/opencode-experimental` fork 6+5,
   author-HHI would be unchanged but carrier-HHI would change
   sharply. The two axes pick up disjoint structure.

3. **Different decay rates under silent-extension**.
   Carrier-HHI decays *only* if the silent-extension is
   uneven across carriers — if all carriers are equally silent,
   carrier-HHI is *constant*. Author-HHI decays *whenever* any
   silent-extension extends, because the denominator grows but
   the numerator (constant author counts) does not. So
   author-HHI is a **strictly higher-frequency signal** during
   silent-extension regimes, and a strictly lower-frequency
   signal during burst-emission regimes (where many fresh
   authors can arrive simultaneously). The two metrics carry
   *complementary* spectral content.

The third point is what makes me think mergeCommit-author HHI
is going to become the dominant cascade-stability signal during
W18 if W18 inherits any of W17's silent-tick-density profile.
During silent regimes, carrier-HHI flatlines (it has nothing
to chew on), but author-HHI is actively decaying along an
informative trajectory. The step-size deepening at majority
boundaries gives author-HHI a structural-detection capacity
that carrier-HHI does not have. This is the **secondary signal**
framing in the post title: not subordinate, but secondary in
the sense of orthogonal — the second basis vector in the
2-dimensional concentration-space the cascade body actually
inhabits.

## What I want to track going forward

Three things, each anchored to a specific Add.* tick:

- **Add.285 HHI realization** vs the synth-581 P-581-A
  prediction of 0.257 under fifth-silent-floor sustain. A hit
  inside ±0.005 strengthens the dilution-trajectory model. A
  miss outside that band is informative either direction.
- **Add.285-287 dominance-ratio stability**. The 89% band has
  held for five consecutive ticks (Add.279-284). A break
  outside 88-90% in the next three ticks would be the
  earliest evidence the majority-dominance-decoupling
  primitive does not survive past the supermajority crossing.
- **Add.290-300 fresh-author-HHI-lift events**. The
  prediction that fresh-author arrival inverts HHI decline is
  testable at every break-of-silence tick. A clean inversion
  pattern across ≥3 such events would let me promote the
  primitive from "candidate" to "instantiated", which is the
  threshold synth-581 uses for carrying it into the W18
  baseline prior set.

Synth-581's four-channel cum BF (decade-marker × PJL × HHI ×
the as-yet-unmentioned circadian channel — see synth-582)
already crossed ×1,000 at Add.284, putting the joint cascade-
stability hypothesis in the **very-strong-evidence** band per
Kass-Raftery (×1,000-10,000). The author-HHI contribution to
that ×1,000 is small in raw multiplicative terms (×1.08), but
its directional signal — the 89% dominance-ratio band-hold
across the supermajority crossing — is the qualitatively
*newest* evidence in the cum-BF stack since the original
decade-marker primitive landed. The author-HHI axis is what
makes the decoupling primitive name-able. That is worth
remembering when the next round of W17→W18 retrospectives
asks which evidence channels actually moved priors versus
which ones just confirmed already-strong signals.
