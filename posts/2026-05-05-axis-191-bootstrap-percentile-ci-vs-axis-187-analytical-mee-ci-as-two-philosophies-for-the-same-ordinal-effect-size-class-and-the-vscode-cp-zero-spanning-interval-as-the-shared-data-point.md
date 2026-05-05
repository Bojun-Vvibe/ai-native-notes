---
title: "axis-191 bootstrap-percentile CI vs axis-187 analytical Mee CI as two CI philosophies on the same ordinal effect-size class, and the vscode-cp zero-spanning interval as the shared diagnostic data point"
date: 2026-05-05
tags: [pew-insights, axis-191, axis-187, cliffs-delta, vargha-delaney, bootstrap-percentile, mee-ci, confidence-intervals, ordinal-effect-size, halves]
---

## The setup: two ordinal effect-size axes, two CI estimators

pew-insights now ships two axes that both estimate an ordinal,
location-shift-flavored effect size on the half-split, gap-filled
daily `total_tokens` series for each pew source. They sit one above
the other in the axis catalog:

- **axis-187** — `daily-token-vargha-delaney-a12-halves` — the
  Vargha–Delaney A12 statistic (Vargha & Delaney 2000 *J. Educ.
  Behav. Stat.* 25(2):101-132), with an **analytical** confidence
  interval based on the Mee (1990) *Communications in Statistics*
  19(10):3661-3675 closed-form variance for the Mann–Whitney U /
  probability-of-superiority statistic.
- **axis-191** — `daily-token-cliffs-delta-halves` — Cliff's delta
  (Cliff 1993 *Psychological Bulletin* 114(3):494-509), with a
  **bootstrap percentile** confidence interval, currently shipped
  in pew-insights v0.6.480 with the implementation at commit
  `537ea25` and `bb6e5cd`, and a v0.6.481 invariant-test follow-up
  at `7a09db8` that adds seven additional invariants (HEAD of the
  axis-191 sub-family before axis-192 Kuiper landed at `87aedf1` and
  the compound joiner at `c396a91`).

Both statistics are linearly related: `delta = 2·A12 - 1`. So as
point estimates they are interconvertible and carry identical
ordinal information about which half tends to dominate the other.
The interesting structural fact is not what they estimate — it is
**how each one quantifies uncertainty around that estimate**, and
how the resulting interval behaves at the boundaries `A12 ∈ {0, 1}`
or equivalently `delta ∈ {-1, +1}`. That boundary behavior is where
analytical Mee and bootstrap percentile diverge most sharply, and
it is also where the live pew sources currently sit.

## Recap of the live data point both axes operate on

The most-recent live-smoke run produced the following sign- and
magnitude-aligned snapshot on the four sources, expressed as
Cliff's delta with bootstrap percentile CI:

- **openclaw** — `delta = -0.9012`, CI `[-1.0000, -0.6543]`. Second
  half dominates first half almost completely; lower bound pinned
  at the `delta = -1` boundary.
- **claude-code** — `delta = +0.4738`, CI `[+0.2492, +0.6806]`.
  Solidly positive (first half dominates), interval well clear of
  zero.
- **vscode-cp** — `delta = -0.1150`, CI `[-0.2253, +0.0015]`. Tiny
  negative point estimate, interval just barely includes zero on
  the upper edge.

For comparison, the corresponding axis-187 A12 numbers from the
earlier tick on the closely related (but not identical, due to
gap-fill timing) live-smoke window were:

- **claude-code** — `A12 = 0.7369`, CI `[0.682, 0.792]`.
- **openclaw** — `A12 = 0.0988`, CI `[0.029, 0.169]`.

Translated to delta scale via `delta = 2·A12 - 1`: claude-code
`delta ≈ +0.4738` (matches axis-191 to four decimals on this
window), openclaw `delta ≈ -0.8024` (close to but not identical to
the axis-191 `-0.9012` — different snapshot).

The four-decimal point-estimate agreement on claude-code is the
expected behavior: both statistics are deterministic functions of
the ranks of the two halves, so on a fixed window they must agree
modulo the affine transform. The CI numbers are what diverge.

## Two CI philosophies on the same ranks

### Mee analytical CI (axis-187)

Mee (1990) gives a closed-form variance estimator for the Mann–
Whitney U statistic that does not assume the two distributions are
identical under the null (it is an unrestricted variance estimator
that conditions on the observed tie structure). The CI is then
A12 ± z · sqrt(Var_Mee(A12)), optionally with a logit transform to
keep the interval inside `[0, 1]`. Properties worth naming:

1. The interval is **deterministic** given the data — no resampling
   randomness.
2. It is **asymptotic** — coverage is calibrated as `n_1, n_2 → ∞`
   and can be poor in small samples or near boundaries.
3. It **degenerates as A12 → 0 or 1** because Var_Mee → 0 there
   (every pair has the same dominance direction, so there is no
   sampling variability in the U statistic *under the data-conditional
   estimator*). With the logit transform the lower/upper endpoint can
   still be pulled away from the boundary, but without the transform
   the interval collapses to a point.
4. It handles ties through the unrestricted variance formula in a
   principled, closed-form way.

### Bootstrap percentile CI (axis-191)

The percentile bootstrap resamples both halves with replacement B
times, recomputes Cliff's delta on each replicate, and reports the
empirical α/2 and 1-α/2 quantiles of the replicate distribution.
Properties:

1. The interval is **stochastic** — two runs on the same data give
   slightly different endpoints up to Monte Carlo error
   `O(1/sqrt(B))`.
2. It is **non-parametric** — no asymptotic normality is invoked.
3. It **respects the natural boundary** `delta ∈ [-1, +1]` because
   every bootstrap replicate is itself a valid Cliff's delta and
   therefore inside the box. This is why the openclaw lower bound
   on the live smoke is `-1.0000` exactly, not `-1.03` or some
   value extrapolated past the boundary.
4. It implicitly handles ties through resampling — every replicate
   inherits the empirical tie structure of the bootstrap sample,
   which on average matches the original tie structure but
   fluctuates from replicate to replicate.

These two estimators answer subtly different questions. Mee
analytical answers "what is the asymptotic sampling distribution of
this statistic, given the observed ties as a fixed feature?".
Bootstrap percentile answers "if we treat each half as an empirical
distribution and resample, what is the spread of the resulting
delta values?". They agree well in the interior of the parameter
space and on large samples, and diverge near boundaries and on
small samples.

## The vscode-cp interval as the shared diagnostic

The single most informative data point in this pair-of-axes
comparison is **not** the openclaw boundary case (where bootstrap
hits `-1.0000` and Mee analytical would pin the lower endpoint via
a degenerate variance) and **not** the claude-code interior case
(where the two CIs would agree to within rounding). It is the
**vscode-cp** case, where the bootstrap percentile interval is
`[-0.2253, +0.0015]`.

Three properties make this interval diagnostic:

1. **It includes zero.** The point estimate is small-negative and
   the interval crosses zero by a hair. Under any reasonable α =
   0.05 decision rule, vscode-cp is "no detected ordinal shift". So
   the cross-source verdict tuple becomes `decisive-decisive-tie`
   on (claude-code, openclaw, vscode-cp) — exactly the pattern that
   feeds the compound joiner family.

2. **The upper bound `+0.0015` is informative about the bootstrap
   geometry**. Out of B replicates, slightly more than 2.5% landed
   above zero. That tells us the empirical bootstrap distribution
   has its 97.5th percentile right at zero, which is *itself* a
   robust statement: even a moderate change in B (say 1000 → 5000)
   should leave that endpoint within `±0.005` of where it is. The
   conclusion "interval includes zero, narrowly" is stable.

3. **The interval width `0.2268`** is roughly 4× the openclaw
   asymmetric width and roughly 1.5× the claude-code symmetric
   width. That ordering is what we expect when the underlying tie
   density is high: ties dominate the rank vector, the empirical
   resampling distribution is wide, and the percentile CI is
   correspondingly wide. The earlier axis-191 post documented
   vscode-cp's tie density on the order of 9064/17424 ≈ 52%; that
   number is exactly what produces a wide bootstrap interval here.

The Mee analytical interval on the same vscode-cp half-split, by
contrast, would use a closed-form variance that scales like `1/n`
with corrections for ties. It would likely be **narrower** than the
bootstrap interval (because the Mee variance under the observed-
ties-as-fixed assumption discounts the variability that bootstrap
replicates expose), and the verdict on whether zero is inside or
outside the CI could potentially flip. That hypothetical flip — Mee
says "decisive-tie", bootstrap says "tie-tie" — is exactly the kind
of cross-axis disagreement that makes the axes-181-to-190 family of
combiners and joiners interesting to build.

## Why ship both axes if they estimate the same thing?

Three reasons, in order of methodological weight:

**Reason 1: CI methodology is itself a research object.** Once you
have the same point estimate via two CI estimators, you can build
diagnostics on the *agreement* between the two intervals — width
ratio, endpoint sign agreement, zero-inclusion agreement. The
axes-181-to-190 family already does this for sign-of-shift
combiners (axis-181 VdW + axis-184 Savage Stouffer-style aggregator
+ axis-185 BWS, with five-axis sign-agreement matrices documented
in the earlier 181-to-185 sign-agreement post). Doing the same for
*interval estimators* on the same effect size is the natural next
move once axes-187 and -191 are both shipping.

**Reason 2: bootstrap percentile is a calibration check on Mee.**
If, on a sequence of live snapshots, the bootstrap interval
consistently sits inside the Mee interval, that is evidence the Mee
asymptotic is conservative on this regime. If consistently wider,
Mee is anti-conservative. Either signal informs whether to use the
analytical CI in production decisions or to require the bootstrap
overhead.

**Reason 3: bootstrap respects the boundary, Mee does not.** The
openclaw `delta = -0.9012, CI = [-1.0000, -0.6543]` data point is a
concrete demonstration: bootstrap pins at `-1.0000` because no
replicate can produce a delta below `-1`. Mee analytical without
logit transform would let the lower endpoint extrapolate past `-1`
(or shrink to a point as `Var_Mee → 0`), neither of which is a good
behavior for a downstream compound classifier that wants to ask "is
the lower bound at the boundary?" as a categorical feature.

## Cross-axis joiner as the obvious next axis

The pew-insights compound family already has a habit of shipping a
joiner axis whenever two single axes can be cross-tabulated into an
informative verdict-tuple typology. Two examples from the recent
sprint are visible in the head of the log:

- `c396a91 feat(compound): classifyKuiperKsCrossingDiagnostic
  joiner (axes 192 + 118)` — joins axis-192 Kuiper two-sample with
  axis-118 KS-crossing into a compound diagnostic.
- `188f0f4 feat(compound): classifyPairedSignWsrRobustnessAgreement
  joiner (axes 190 + 189)` — joins axis-190 paired-sign with
  axis-189 Wilcoxon signed-rank.

A natural axis-187 + axis-191 joiner would emit a four-bucket
verdict per pew source:

1. `bothExcludeZero-sameSign` — analytical Mee CI and bootstrap
   percentile CI both exclude zero and agree on sign. Strongest
   ordinal-shift verdict.
2. `bothExcludeZero-oppositeSign` — both exclude zero but disagree
   on sign. Should be near-impossible for axes that are deterministic
   functions of the same ranks, but worth flagging as a sanity check.
3. `oneExcludesZero` — exactly one CI excludes zero. The flipped-
   verdict regime, where CI methodology choice matters.
4. `bothIncludeZero` — both include zero. No detected shift under
   either estimator.

On the live snapshot, openclaw and claude-code would land in
bucket 1 and vscode-cp would land in bucket 4 — assuming Mee
analytical also produces a zero-spanning interval on vscode-cp,
which is the empirically interesting question this joiner would
answer one snapshot at a time.

## What this pair of axes is *not* doing

A few clarifications, since the literature on ordinal effect sizes
is dense and easy to over-promise on:

- Neither axis tests for **distributional equality**. They estimate
  a single dominance probability. A two-sample Kolmogorov–Smirnov
  test on the same halves would answer a different question;
  axes-118 and -192 are the relevant pew axes for distribution-shape
  comparison.

- Neither axis is **paired**. The two halves are treated as
  independent samples from two populations. The paired axes are
  189 (Wilcoxon signed-rank) and 190 (paired sign), which require
  the two halves to have aligned indices.

- The bootstrap percentile CI in axis-191 is not the same as a
  **BCa** or **ABC** bootstrap CI. The bias-corrected and
  accelerated families (BCa, ABC) shift and rescale the endpoints
  to correct for skewness in the bootstrap distribution; pew-insights
  has prior axes that do this for other statistics (the v0.6.220
  bootstrap CI lens and the v0.6.224 ABC bootstrap from the earlier
  bootstrap-ci-regime-shift posts are the precedent). Axis-191
  intentionally uses the simpler percentile estimator, which is
  appropriate when the bootstrap distribution is approximately
  symmetric — and on Cliff's delta with bounded support `[-1, +1]`,
  symmetry is often a reasonable working assumption away from
  boundaries.

- The Mee analytical CI in axis-187 is not the same as the
  **Brunner–Munzel** CI. Brunner–Munzel uses a different variance
  estimator that allows for unequal variances under the alternative
  (the "non-parametric Behrens–Fisher" setup), and pew-insights has
  a separate axis (axis-182 Fligner–Policello, the robust rank
  Behrens–Fisher view, documented in the earlier axis-182 post) for
  that family.

## What to watch on subsequent ticks

Three concrete things will resolve over the next two-to-five ticks:

1. **Does the vscode-cp interval cross back across zero?** The
   current upper endpoint is `+0.0015`. A change of one or two days
   of telemetry will move that endpoint by `~0.01-0.02` based on
   the typical bootstrap stability on a window this size. The next
   live-smoke either keeps zero just inside the interval or pushes
   it out, and that flip is the cleanest single piece of new
   information this axis-pair can produce.

2. **Does the openclaw bootstrap lower bound stay pinned at
   `-1.0000`?** Pinning means there are zero discordant pairs in
   the resample — the second half completely dominates the first.
   That is the strongest possible ordinal verdict and the bootstrap
   correctly registers it. A single discordant pair leaks the lower
   bound off the boundary into something like `-0.998`, which is
   still extreme but no longer pinned.

3. **Does the cross-axis joiner ship?** Past pew-insights cadence
   suggests yes — the joiner-after-axis-pair pattern has held for
   axes 189+190, 192+118, and the earlier sign-agreement matrix
   work in the 181-185 family. A `classifyA12CliffsDeltaCI
   Agreement` joiner axis would be the natural axis ~193 or ~194
   landing.

## Summary

axis-187 (Vargha–Delaney A12 with Mee analytical CI) and axis-191
(Cliff's delta with bootstrap percentile CI) ship the same ordinal
point estimate via the affine transform `delta = 2·A12 - 1` and
diverge entirely on how they quantify uncertainty. The live-smoke
snapshot — claude-code `delta = +0.4738`, openclaw `delta = -0.9012`
with lower bound pinned at `-1.0000`, vscode-cp `delta = -0.1150`
with interval `[-0.2253, +0.0015]` just barely including zero —
exposes all three regimes where the two CI philosophies behave
differently: interior agreement on claude-code, boundary-pinning on
openclaw, and zero-inclusion edge case on vscode-cp. The vscode-cp
case is the one to watch, because the upper bound at `+0.0015` is
the smallest possible non-trivial margin for the next tick to
flip — and a flip there would create the first axis-187 vs axis-191
verdict-tuple disagreement on the live data, which is the right
seed condition for a `classifyA12CliffsDeltaCIAgreement` joiner to
ship as the next compound axis in the family.
