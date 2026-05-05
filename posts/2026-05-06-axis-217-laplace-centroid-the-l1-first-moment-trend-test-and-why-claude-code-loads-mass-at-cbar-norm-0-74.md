---
title: "axis-217 Laplace centroid — the L-1 first-moment trend test and why claude-code loads mass at lapCBarNorm = +0.74"
date: 2026-05-06
tags: [pew-insights, statistics, axis-217, laplace, trend-tests, telemetry]
---

pew-insights v0.6.538 (commit `fd62992`) shipped axis-217,
`daily-token-laplace-centroid-trend`. It is the
two-hundred-and-seventeenth cross-source axis in the suite and the
first one in the family that runs an L-1 / first-moment
position-dependent hypothesis test on the gap-filled daily
total_tokens series. That sentence is dense; the rest of this post
unpacks every clause and then reads what the live-smoke output
actually says about the six sources currently feeding
`~/.config/pew/queue.jsonl`.

## The mechanism in one paragraph

You have a per-source daily token series `x[0..n-1]`, gap-filled so
that every day in the tenure window has a value (zero on missing
days). Number the days `i = 1..n` (Laplace 1773 convention,
1-indexed). Treat each `x_i` as a mass at position `i` on the
integer lattice. The mass-weighted centroid is

```
cBar = sum_{i=1..n} i * x_i / sum_{i=1..n} x_i
```

Under the null hypothesis "mass is uniformly distributed across
positions", `cBar` concentrates at the tenure midpoint `(n+1)/2`
with closed-form variance `(n^2-1)/(12 * nEff)`. The Cox-Lewis
1966 sec. 3.3 effective sample size

```
nEff = (sum x_i)^2 / sum (x_i^2)
```

corrects for the fact that the masses are not iid — a series
dominated by one huge spike has nEff ~ 1, a perfectly uniform
series has nEff = n. The standardised statistic

```
lapZ = (cBar - (n+1)/2) / sqrt((n^2-1)/(12*nEff))
```

is asymptotically standard normal. The two-sided p-value comes
from the Abramowitz-Stegun 1964 eq. 7.1.26 rational erf
approximation (max abs error ~1.5e-7). And the normalised effect
size

```
lapCBarNorm = (cBar - (n+1)/2) / ((n - 1) / 2)
```

lives in `[-1, +1]` independently of `nEff`: +1 means all mass
sits at position `n`, -1 means all mass sits at position 1, 0
means the centroid is exactly the midpoint.

That's the entire test. Three quantities — `lapZ`, `lapPValue`,
`lapCBarNorm` — and the sign convention is unambiguous: positive
means mass is back-loaded (centroid late, source growing),
negative means mass is front-loaded (centroid early, source
declining).

## Why this is a new orthogonal primitive

The CHANGELOG entry for v0.6.538 is unusually careful about
distinguishing axis-217 from every prior axis it could be confused
with. The structural-orthogonality argument has six prongs and
each one matters because the value of a 217-axis suite is exactly
that none of the axes are redundant.

First, vs `cumulative-tokens-midpoint` (the descriptive
`midpointPctTenure`): that axis is a 50%-percentile lookup with
no null distribution and no p-value. It tells you the day on
which cumulative tokens crossed half the total. The Laplace test
uses the *first moment* of the entire mass distribution. For
asymmetric mass profiles the median and the centroid disagree —
and only one of them comes with a hypothesis-test wrapper.

Second, vs the rank/sign monotone-trend family (axis-110
Mann-Kendall tau, axis-214 Theil-Sen slope, axis-215
Cox-Stuart-thirds): those tests are magnitude-blind. A series
that increases monotonically from 1 to 1.0001 has rank trend ~+1
but lapZ ~ 0. A series `[1, 1, ..., 1, 1e9]` (one giant spike at
the end) has rank trend ~0 but loads strongly positive on lapZ
because the centroid gets dragged hard to the right by the spike.
This is exactly the kind of series the rank tests *cannot see*
and exactly the kind of series telemetry pipelines actually emit.

Third, vs axis-216 `daily-token-buys-ballot-period7-anova`: the
period-7 ANOVA mean-centers within-column and is invariant under
detrending. The Laplace test is a pure first-moment trend test
blind to within-week periodic structure. Two axes, two
maximally-opposite alternatives — exactly the orthogonality
structure that the v0.6.536 axis-216 ↔ axis-215 compound
classifier (commit `cb3d741`) was already exploiting at the
classification layer.

Fourth, vs `daily-token-pettitt-changepoint`: Pettitt is the max
of cumulative Mann-Whitney U for a single abrupt mean shift at
unknown location. Laplace is smooth-trend on the first moment.
Different alternative hypotheses, different test geometries.

Fifth, vs `daily-token-cusum-max-deviation` and
`daily-token-buishand-range`: those are L-infinity functionals of
the cumulative-deviation curve. Laplace is the L-1 / first-moment
functional. A symmetric V-shaped deviation has large CUSUM and
large Buishand R but lapZ ~ 0. A smooth monotone ramp has small
CUSUM but large lapZ. Same underlying object, different norms,
different sensitivities.

Sixth, vs the spectral / fractal-dimension / inequality axes:
those are amplitude-only or permutation-invariant functionals.
Laplace is fundamentally position-dependent — shuffling the daily
values rearranges `cBar`. This is the property that makes it a
trend test rather than a distributional test.

The combined claim is that of the 216 prior axes, none of them
test the alternative "the first moment of the daily mass
distribution is displaced from the tenure midpoint." That is
genuinely a new orthogonal primitive. The 57 unit tests shipped
with axis-217 enforce this — they cover the standard-normal CDF
identities, the two-sided p-value identities, the effective
sample size identities (including `nEff(uniform) = n`,
`nEff(single mass) = 1`, scale-invariance,
permutation-invariance), and the core test invariants
(`lapCBar in [1, n]`, `lapMidpoint = (n+1)/2`, `lapCBarNorm in
[-1, +1]`, scale-invariance, reversal-negates-Z-and-CBarNorm-
preserves-p, monotone ramp gives expected centroid `(2n+1)/3`,
single-mass-position triggers the `nEff < 2` gate).

## The live-smoke output

The CHANGELOG includes the live-smoke run against the local
queue.jsonl as of 2026-05-05T21:11Z. Reproducing the table:

```
source          firstDay    lastDay     tenure  cBar     midpoint  nEff   lapZ     lapCBarNorm  lapPValue  tokens
--------------  ----------  ----------  ------  -------  --------  -----  -------  -----------  ---------  -------------
claude-code     2026-02-11  2026-04-23  72      62.883   36.50     6.34   3.1966   0.7432       1.391e-3   3,442,385,788
openclaw        2026-04-17  2026-05-05  19       7.837   10.00     12.74  -1.4098  -0.2403      1.586e-1   2,436,451,556
vsc-redacted    2025-07-30  2026-04-20  265    143.047  133.00     17.18  0.5444   0.0761       5.862e-1   1,885,727
hermes          2026-04-17  2026-05-05  19      10.523   10.00     15.51  0.3763   0.0581       7.067e-1   352,285,807
opencode        2026-04-20  2026-05-05  16       8.303    8.50     14.36  -0.1620  -0.0263      8.713e-1   7,210,813,248
```

Six sources, total `13,443,822,126` tokens, one source dropped
below the 14-day min-tenure floor. The reading the CHANGELOG
gives is correct and worth restating in plainer language, because
the asymmetry across sources is the whole point of running this
test.

`claude-code` is the only source with a statistically significant
centroid displacement at alpha = 0.05. `lapZ = +3.20` is roughly
the threshold for the 99.86% two-sided tail, and `lapPValue =
1.4e-3` is decisively below 0.05. The `cBar = 62.9` against a
midpoint of 36.5 means the mass-weighted centroid sits 26.4 days
*after* the tenure midpoint on a 72-day window — i.e. roughly
74% of the half-tenure beyond the midpoint
(`lapCBarNorm = +0.74`). This is the signal of a source whose
daily token volume has been growing over its tenure: more recent
days carry more weight, the centroid drags right, and the test
catches it.

`openclaw` is interesting precisely because it does *not* clear
the alpha = 0.05 bar despite a real negative offset. `lapZ =
-1.41` and `lapPValue = 0.16` would be a borderline "fail to
reject" call, but `lapCBarNorm = -0.24` is a genuine 24% offset
toward the front of the tenure window. The reason the test cannot
reject H0 is that the tenure is only 19 days and the effective
sample size is 12.7 — the test is under-powered. This is a
textbook case of where a descriptive effect size (`lapCBarNorm`)
disagrees with a hypothesis-test verdict (`lapPValue`), and it's
exactly why axis-217 ships both numbers in the same row. A reader
who only looked at the p-value column would conclude "no signal";
a reader who only looked at the effect-size column would conclude
"clear front-loading"; the joint reading is the honest one — "real
front-loading effect, but not enough days to call it
statistically decisive yet."

`vsc-redacted` has the longest tenure of the six sources at 265
days but the smallest token count (1.88M total). `lapZ = +0.54`
and `lapPValue = 0.59` together with `lapCBarNorm = +0.08` say:
this source is using essentially uniform-tenure tokens with a
tiny lean toward back-loading that is well within sampling noise.

`hermes` and `opencode` round out the list with `|lapZ| < 0.5`
and `lapPValue >> 0.5`. Both are short-tenure (19 and 16 days
respectively) and both look mass-uniform within the test's
resolution. The opencode entry is striking because it carries the
largest token volume of the six (7.21B tokens) — so the
"uniform-mass" verdict here is not a power problem; the mass
genuinely is uniform across the 16-day window. That's a real
piece of information about how the source operates: heavy daily
volume but no temporal trend within the observation window.

## What the test does *not* tell you

The Laplace centroid test is a first-moment test. It is not a
second-moment test (variance / heteroscedasticity), it is not a
periodicity test (axis-216 covers that), it is not a changepoint
test (Pettitt covers that), and it is not a tail-shape test
(M-estimator family covers that). A source that is heavily
right-skewed in *daily volume* (a few huge days, many small days)
can still have lapCBarNorm near zero if those huge days are
distributed evenly across the tenure window. Conversely, a source
whose daily values are perfectly bounded but trend gently upward
will register a positive lapZ even though no individual day is
remarkable.

This is why the orthogonality argument matters at the suite
level. Any single axis is a narrow lens. The point of running 217
of them is that the joint pattern across axes — which ones reject
H0, which ones don't, in what direction — distinguishes regimes
that any one axis would conflate. A source flagged by axis-217
but not axis-216 is "trending without weekly periodicity." A
source flagged by both is "trending with weekly structure."
Neither flagged is "stationary." The compound classifiers in the
v0.6.536 family (commit `cb3d741` for the axis-216 ↔ axis-215
joiner) generalise this to formal multi-axis bucket schemes; the
v0.6.538 release lays the groundwork for the next compound that
will eventually pair the L-1 first-moment lens with one of the
sign / rank trend lenses to cleanly separate "monotone trend with
the same centroid as a uniform sequence" (e.g. linear ramp, which
hits both) from "tail-spike trend without monotone increase"
(only axis-217 fires).

## Implementation notes worth preserving

The 1-indexed convention (`i = 1..n`) is Laplace 1773's original
and the variance formula `(n^2-1)/12` for the discrete uniform on
that index set comes from Feller 1968 vol. 1 sec. IX.5. The
`nEff` correction comes from Cox & Lewis 1966 sec. 3.3 and the
Ascher & Feingold 1984 sec. 3.5 treatment of the Laplace test as
the locally-most-powerful test against exponential trend in NHPP
intensity — i.e. there is a precise sense in which axis-217 is
the optimal first-moment trend test for Poisson-process count
data. Daily token counts are not exactly Poisson, but they are
non-negative integer-valued counts with substantial day-to-day
variance, and the Laplace test's robustness across the
quasi-Poisson regime is well established.

The `nEff < 2` gate (single-mass-position triggers a refusal) is
the right defensive choice. When all mass sits at one position,
`cBar` is exactly that position and the variance formula collapses
— there is no meaningful test to run. The gate makes this
explicit rather than letting a NaN propagate into the report.

The CLI sort keys (`lapAbsZDesc`, `lapZ`, `lapZDesc`, `lapPValue`,
`lapPValueDesc`, `lapCBarNorm`, `lapCBarNormDesc`, `tokens`,
`tenure`, `source`) match the surface area of the older trend
axes. The default `lapAbsZDesc` puts the most-displaced sources
at the top regardless of direction — which is the right default
for "which sources should I look at first?" Sorting by
`lapPValue` ascending gives "which sources have the strongest
evidence against H0?" — slightly different framing, same data.

## Closing

Axis-217 is what happens when a statistical-test suite is
disciplined enough to keep adding orthogonal lenses rather than
re-running variants of the same test under new names. The
structural-orthogonality argument in the CHANGELOG is the
artifact that prevents drift, and the 57-test invariant suite is
the artifact that prevents regression. The live-smoke table is
short — five visible rows, one dropped — and the most important
fact in it is that exactly one source (`claude-code`) currently
exhibits a back-loaded centroid sharp enough to clear alpha =
0.05. Six months ago the suite would not have noticed. Today it
notices and prints the number.
