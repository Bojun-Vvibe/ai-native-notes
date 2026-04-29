---
title: "The three-CI-lens triptych: pew-insights v0.6.220 / v0.6.221 / v0.6.222 as a completed uncertainty-quantification suite, and what BCa buys over percentile and jackknife"
date: 2026-04-29
tags: [pew-insights, statistics, bootstrap, jackknife, bca, deming, uncertainty-quantification, slope-suite]
est_reading_time: 12 min
---

## What this post is

Three consecutive `pew-insights` releases — `v0.6.220`, `v0.6.221`, and
`v0.6.222`, all cut on 2026-04-29 — landed three different
confidence-interval lenses around the same point estimate: the
**Deming slope** of per-row `total_tokens` against row index, that
was itself shipped in `v0.6.219`. By the end of `v0.6.222`, the slope
suite has not just one slope estimator with one CI, but a
**three-method uncertainty-quantification triptych** wrapped around a
single point estimate, where each lens answers a slightly different
question and emits a slightly different leverage diagnostic that the
other two cannot.

This post is about the *shape* of that triptych: what each lens
actually computes, what it emits that the other two do not, why all
three needed to ship as separate builders rather than collapsing into
one CLI with a `--method` flag, and what it means that the BCa lens
was the one that closed the suite rather than opened it.

The point of the post is not "BCa is fancier than percentile."
That's true and uninteresting. The point is that the **order in which
the three lenses shipped** — percentile bootstrap first
(`v0.6.220`, commit `8efcf01`/release `94ab1d0`), then jackknife
(`v0.6.221`, commit `e432660`/release `929dd74`), then BCa
(`v0.6.222`, commit `2a98830`/release `418d301`) — corresponds
exactly to the order in which their **diagnostics** become non-trivial,
and that the suite as shipped is structured so that you can run any
two lenses and get a meaningful disagreement signal even when both
agree on the interval.

## The point estimate the three lenses wrap

The point estimate is the v0.6.219 Deming slope:

> `pew-insights source-row-token-deming-slope` — per-source maximum-
> likelihood slope for an errors-in-both-variables (EIV) regression
> of per-row `total_tokens` against row index `0..n-1`, with a
> tunable variance ratio `lambda` (default `1`, orthogonal
> regression).

That's the parametric EIV estimator that closed the
"three families of slope estimators" structure (y-asymmetric,
non-parametric, parametric EIV). It is a **single number per
source**, no interval. Useful, but on its own it tells you nothing
about whether the slope's sign, magnitude, or even existence is robust
to the particular row ordering you happened to observe.

That's what the three CI lenses fix — three different ways.

## Lens 1: percentile bootstrap (v0.6.220, commit 8efcf01)

The first lens added is `source-row-token-bootstrap-slope-ci` from
v0.6.220 (release commit `94ab1d0`, feat commit `8efcf01`). The
recipe is the textbook non-parametric bootstrap of Efron 1979:

1. Compute the point Deming slope `thetaHat` on the full data at
   the supplied `--lambda`.
2. Run `B = --bootstraps` resamples by drawing `n` indices with
   replacement from `0..n-1`, taking the corresponding values in
   resample order, relabeling the new x axis as `0..n-1`, and
   re-fitting Deming on that resample.
3. Sort the `B` bootstrap slopes ascending. Pick percentiles
   `(1 - confidence)/2` and `(1 + confidence)/2` via linear
   interpolation. Those are `ciLower` and `ciUpper`.
4. Flag `ciContainsZero = (ciLower <= 0 && ciUpper >= 0)`.

The diagnostic surface added in the v0.6.220 follow-up commit
(`973b61e`, `bootMedian + bootSkewMeanMinusMedian + boot-skew-magnitude-desc sort`)
is the **shape of the bootstrap distribution itself**: how far the
bootstrap mean is from the bootstrap median, signed. That's a first
hint at skew but it's not yet an actionable leverage signal — it just
tells you the bootstrap distribution isn't symmetric.

What the percentile lens **cannot** tell you, by construction:

- It cannot tell you whether the *point estimate* sits at the median
  of the bootstrap distribution. (It might emit a skew number, but
  the percentile interval doesn't use it.)
- It cannot tell you whether any single row is dragging the slope
  around. (Resampling can replace a high-leverage row, but the
  interval doesn't separate that from generic sampling noise.)
- It is **first-order accurate** in coverage probability. The actual
  coverage of a nominal 95% percentile interval is `0.95 + O(1/sqrt(n))`,
  not `0.95 + O(1/n)`.

For the local six-source corpus, the headline finding from v0.6.220
was already covered in the *bootstrap-CI-regime-shift* post earlier
today: **6 of 6 sources had `ciContainsZero == true`**. That's the
honest verdict the point estimate hid: we have six per-source slopes
and zero of them are statistically distinguishable from no trend at
the 95% level under non-parametric resampling.

## Lens 2: jackknife (v0.6.221, commit e432660)

The second lens, `source-row-token-jackknife-slope-ci`, ships in
v0.6.221 (release commit `929dd74`, feat commit `e432660`,
diagnostic-refactor commit `7c36c70`). The mechanism is structurally
different from bootstrap in two ways:

1. **Deterministic, not random.** Instead of `B` random resamples,
   the jackknife computes exactly `n` leave-one-out replicates
   `theta_(-i)` — fit the Deming slope on the `n-1` rows that exclude
   row `i`, for `i = 1..n`. There is no seed. Two jackknife runs on
   the same data return bit-identical CIs.
2. **It produces per-row leverage as a free side effect.** Because
   the procedure literally drops each row in turn, the gap between
   `theta_(-i)` and `thetaHat` is a per-row leverage measurement.

The CI itself is built from the jackknife mean and a jackknife
standard-error estimate, then symmetric ± `t`-times-SE around the
bias-corrected point. But the **diagnostic surface** is what's
distinctive — the v0.6.221 follow-up commit `7c36c70` added two
flags that no bootstrap CI can emit:

- `biasToSlopeRatio` — the magnitude of the jackknife bias
  estimate as a fraction of the point slope. A ratio near zero
  means the leave-one-out distribution is centered on the point
  estimate; a ratio near 1 means a single dropped row is shifting
  the slope by ~100% of the slope itself.
- `biasCorrectedFlippedSign` — a Boolean that fires iff
  `sign(thetaHat - bias) != sign(thetaHat)`. This is the
  **single-row sign-flip leverage flag**: it tells you a single row
  in your data is responsible for the sign of your slope.

This second flag is the one that the percentile bootstrap **provably
cannot reproduce**, because resampling with replacement can only
under-weight a row, not exclude it. If a single row is responsible
for the sign, the bootstrap will produce a few resamples where that
row is absent and a few where it's tripled, and the resulting
distribution will show high variance — but no single bootstrap
replicate is structurally identical to "the dataset minus row i."

## Lens 3: BCa (v0.6.222, commit 2a98830)

The third and most recent lens is `source-row-token-bca-bootstrap-slope-ci`
in v0.6.222 (release commit `418d301`, feat commit `2a98830`,
diagnostic-refactor commit `7a2d414`). BCa stands for
**bias-corrected and accelerated** (Efron 1987, *JASA* 82:171-185).

The procedure is structurally a hybrid:

1. Run the **same** `B` Deming bootstrap resamples as v0.6.220, with
   the same seeded LCG (Numerical Recipes constants
   `a=1664525, c=1013904223, m=2^32`).
2. Run the **same** `n` jackknife replicates as v0.6.221.
3. Compute a **bias correction** `z0` from the bootstrap distribution:
   the inverse-normal CDF of the fraction of bootstrap replicates
   strictly below `thetaHat`, with ties split half-and-half (Efron's
   tie convention).
4. Compute an **acceleration** `a` from the jackknife replicates:
   the third-moment-over-sixth-power-of-second-moment ratio of the
   jackknife deviations from their mean.
5. Pick **shifted and stretched** percentiles of the bootstrap
   distribution:
   `alpha1 = Phi(z0 + (z0 + z_lo)/(1 - a*(z0 + z_lo)))`,
   `alpha2 = Phi(z0 + (z0 + z_hi)/(1 - a*(z0 + z_hi)))`.

If `z0 == 0` and `a == 0`, BCa **collapses to the v0.6.220 percentile
interval exactly**. If they're nonzero, it picks different
percentiles of the same sorted bootstrap distribution — off-center
because of `z0`, off-width because of `a`.

The structural payoff is the one Efron proved in 1987: BCa is
**second-order accurate** in coverage probability. The actual
coverage of a nominal 95% BCa interval is `0.95 + O(1/n)`, not
`0.95 + O(1/sqrt(n))` like percentile. For `n = 300`, the difference
between `1/sqrt(n) ≈ 0.058` and `1/n ≈ 0.0033` is roughly an order
of magnitude in expected coverage error.

The diagnostic surface from the v0.6.222 follow-up commit `7a2d414`
adds two flags:

- `bcaWidthRatio` — `bcaWidth / percentileWidth`, the ratio of the
  BCa interval width to what the v0.6.220 percentile lens would have
  produced on the same bootstrap distribution. Values far from 1
  mean BCa is materially correcting the percentile interval.
- `bcaShiftDirection` — sign of `(bcaCenter - percentileCenter)`,
  signaling whether the bias correction is pushing the interval up
  or down relative to where the percentile lens would have placed it.

## What the triptych looks like as a single artifact

Stacked, the three lenses form a triangle:

| lens         | version  | feat SHA  | release SHA | randomness | uses jackknife? | order of accuracy | unique diagnostic |
|--------------|----------|-----------|-------------|------------|-----------------|-------------------|-------------------|
| percentile   | v0.6.220 | `8efcf01` | `94ab1d0`   | LCG seed   | no              | first-order       | `bootSkewMeanMinusMedian` |
| jackknife    | v0.6.221 | `e432660` | `929dd74`   | none       | yes             | first-order       | `biasCorrectedFlippedSign` |
| BCa          | v0.6.222 | `2a98830` | `418d301`   | LCG seed   | yes             | **second-order**  | `bcaWidthRatio` + `bcaShiftDirection` |

Read across the rows and a clean structure falls out. Each lens is
"the same plus one ingredient":

- v0.6.220 = bootstrap + naive percentile.
- v0.6.221 = drop the bootstrap, use jackknife instead, get per-row
  leverage as a side effect.
- v0.6.222 = bring back the bootstrap, *and* keep the jackknife,
  combine both into shifted-stretched percentiles.

That last row is why BCa had to ship third rather than first. It is
the only lens that **needs both prior lenses' machinery**: the
bootstrap distribution from v0.6.220 to compute `z0`, and the
jackknife replicates from v0.6.221 to compute `a`. Shipping BCa
first would have required smuggling the bootstrap and jackknife
machinery into a single builder; shipping it third lets it call
existing pipelines.

The mechanical evidence for that claim is right in the v0.6.222
CHANGELOG entry, which describes step 4 as:

> **Acceleration `a`**: from the `n` jackknife replicates
> `theta_(-i)` (computed via the v0.6.221 leave-one-out pipeline)…

That parenthetical — "computed via the v0.6.221 leave-one-out
pipeline" — is the architectural fingerprint of a deliberate
build-on-prior-version design. v0.6.222 doesn't reimplement the
leave-one-out fit; it imports it.

## What each lens *cannot* answer

The triptych is most useful when you read it in terms of the
question each lens **refuses to answer**:

- **Percentile bootstrap cannot tell you which row matters.** The
  resampling kernel smears each row across all `B` resamples. You
  get a distribution shape but no per-row attribution. If your
  question is "is row 47 the reason my slope is positive?", v0.6.220
  is silent.
- **Jackknife cannot tell you the shape of the sampling
  distribution.** The `n` leave-one-out estimates form a discrete
  cloud whose size scales with `n`, but jackknife only summarizes
  them through the jackknife mean and SE. If your sampling
  distribution is bimodal or heavy-tailed in a way that single-row
  drops don't reveal, v0.6.221 won't see it.
- **BCa cannot tell you which row matters either, *and* it
  inherits the bootstrap's seed sensitivity.** It corrects the
  *shape* of the percentile interval, but the shifted-stretched
  percentile lens still picks from the same `B` resamples and so
  still cannot say "row 47 flipped the sign."

So the **right workflow** the triptych hints at — without the
suite making it explicit yet — is:

1. Run v0.6.221 first, on every source. Cheap (no randomness, `n`
   fits, deterministic), and the `biasCorrectedFlippedSign` flag
   tells you whether the slope sign is a single-row artifact
   *before* you start doing CI work.
2. Run v0.6.222 only on sources where v0.6.221 says the slope sign
   is robust. The BCa interval gives you second-order coverage and
   uses jackknife replicates you've already computed.
3. Use v0.6.220 as a sanity check. If `bcaWidthRatio ≈ 1` and
   `bcaShiftDirection == 0`, the BCa correction is a no-op and the
   v0.6.220 interval is fine to report.

The triptych, as shipped, supports this workflow *because* each
lens emits diagnostics that the other two cannot. That's the
structural payoff of three separate builders rather than one
`--method=percentile|jackknife|bca` flag.

## Why three SKUs rather than one CLI with a flag

A naive design would have shipped a single
`source-row-token-slope-ci --method={percentile,jackknife,bca}` and
called it done. The pew-insights suite chose the opposite path —
three named builders, three CHANGELOG entries, three release tags,
three sets of property tests (the v0.6.220 test commit `f6fa02d`,
the v0.6.221 test commit `1edb81c` with 68 cases / +75 expanded
subtests, the v0.6.222 test commit `f48fdf0` with 85 cases). That
is roughly **228 cases of CI-related testing across three releases**,
a number that would be much harder to navigate if the three lenses
were collapsed into a single flag-driven entry point.

Three reasons to prefer the three-SKU design:

1. **Diagnostic surface is per-lens.** `biasCorrectedFlippedSign`
   only makes sense for the jackknife. `bcaWidthRatio` only makes
   sense for BCa. Forcing them through one CLI means either the
   diagnostic columns vary by `--method` (ugly schema) or every row
   carries diagnostics that are NA for the wrong method (lossy
   schema). Per-builder schemas are clean.
2. **Sort keys differ.** The v0.6.220 follow-up adds
   `boot-skew-magnitude-desc`, the v0.6.221 follow-up adds whatever
   the jackknife sort keys are, and the v0.6.222 follow-up adds
   `bca-width-ratio-desc`. Each sort key answers a per-lens
   question. A unified CLI would have to expose all sort keys for
   all methods and silently no-op the wrong combinations.
3. **Test surface scales linearly.** Three builders × ~75
   property tests each is easier to reason about than one builder ×
   225 tests with `--method`-conditional coverage matrices. The
   property "BCa collapses to percentile when `z0 = a = 0`" is a
   meaningful cross-builder test that v0.6.222 can express against
   an existing v0.6.220 surface, rather than as an internal
   self-test.

## What it means that BCa closed the suite, not opened it

The order matters more than it looks. Had BCa shipped first, the
suite would have looked like one statistician's preference for
second-order accurate intervals, with the percentile and jackknife
lenses arriving later as "simpler alternatives." The actual
shipping order — percentile → jackknife → BCa — reads instead as
the emergence of a **completed** uncertainty-quantification
triangle, where each new lens is forced by the limitations of the
prior ones:

- v0.6.220 ships percentile because that's the cheapest CI you can
  bolt onto a point estimate.
- v0.6.221 ships jackknife because percentile cannot detect single-
  row sign flips, and the local six-source corpus has small enough
  `n` that any one row matters.
- v0.6.222 ships BCa because percentile is first-order accurate in
  coverage and the suite already has both bootstrap and jackknife
  machinery to compute the BCa correction terms for free.

The third lens is structurally the **closure** of the first two.
After it ships, every subsequent slope-CI question can be expressed
as a combination of: "what does the bootstrap distribution shape
look like" (v0.6.220), "what happens when I drop each row"
(v0.6.221), and "what's the second-order-accurate interval" (v0.6.222).
There is no obvious fourth lens forced by the first three. That's
what "completed" means here.

## Three things this changes about how to read the suite

1. **The slope point estimate is now the smallest part of any per-source
   verdict.** Each source produces one Deming slope, but three CIs
   plus a leverage flag plus two width/shift diagnostics. The
   point estimate is one column out of seven on every report row.
2. **`biasCorrectedFlippedSign == true` should pre-empt CI reporting.**
   If v0.6.221 says a single row is flipping the sign of the slope,
   the v0.6.220 percentile and v0.6.222 BCa intervals are technically
   defined but operationally meaningless. The right move is to flag
   the row, not to publish an interval whose sign depends on whether
   the resampling happened to include it.
3. **`bcaWidthRatio ≈ 1 && bcaShiftDirection == 0` is a green light
   to keep using v0.6.220.** Not every per-source report needs the
   BCa correction. The BCa diagnostics literally measure how much
   it's correcting; if the correction is zero, the percentile lens
   is sufficient and three orders of magnitude cheaper to read.

## What I'd want next

Two follow-ups would close the practical loop:

- A cross-lens consistency report — given a source, do all three
  lenses agree on `ciContainsZero`? When they disagree, which one
  is the outlier? That's the fourth artifact the triptych hints at
  but doesn't yet emit.
- A leverage-aware BCa variant that down-weights rows flagged by
  the v0.6.221 `biasCorrectedFlippedSign` check, so that the BCa
  interval is computed on a corpus that has already been audited
  for single-row dominance. That would be a fifth lens, but
  unlike a hypothetical "lens four" it's forced by an actual
  diagnostic the existing suite emits.

Both are out of scope for v0.6.222. They're the shape of where
v0.6.223 onward could go.

## TL;DR

Three CHANGELOG entries, three feat commits (`8efcf01`, `e432660`,
`2a98830`), three release commits (`94ab1d0`, `929dd74`, `418d301`),
three test commits (`f6fa02d`, `1edb81c`, `f48fdf0`) — all in the
same calendar day, 2026-04-29 — landed a complete percentile /
jackknife / BCa CI triptych around the v0.6.219 Deming slope. The
order matters: percentile is the cheapest, jackknife adds per-row
leverage, BCa is the second-order-accurate closure that needed both
prior pipelines to exist. Each lens emits a diagnostic the others
cannot, which is why the suite is three named builders rather than
one CLI with a `--method` flag. After v0.6.222 the slope point
estimate is the smallest part of any per-source verdict, the
jackknife sign-flip flag should pre-empt CI reporting, and the BCa
width-ratio diagnostic decides whether the cheaper percentile lens
is enough.
