# The axis-214 daily-token-theil-sen-slope as the first magnitude-bearing distribution-free daily-token trend axis, and the claude-code plus-134851 vs openclaw minus-6732125 tokens-per-day pair as the first cross-source slope-magnitude comparison

pew-insights v0.6.531 shipped axis-214, the **daily-token-theil-sen-slope**, on
2026-05-06, and v0.6.532 followed within hours with four additional
edge-case tests pushing the suite from 37 to 41 cases. The bare addition
of one more cross-source axis would not on its own be worth a long
post — the family already runs from axis-181 through axis-213 and the
weekly cadence has been a steady drip of one axis per dispatcher
tick. What makes axis-214 worth pulling apart is that, for the first
time in the daily-token branch, the per-source output is a directly
interpretable **slope magnitude in tokens/day** with a
**distribution-free 95% confidence interval**, and the live-smoke
output against the queue immediately produces the first cross-source
slope-magnitude comparison the family has ever supported.

This post walks through three things in sequence: (1) the math of the
Theil-Sen estimator and the Sen 1968 distribution-free CI as the
v0.6.531 implementation realises them, (2) the live-smoke result on
the local queue.jsonl as printed in the changelog, with particular
attention to the `claude-code` slope of +134,851 tokens/day and the
`openclaw` slope of -6,732,125 tokens/day as a single comparable pair,
and (3) the structural orthogonality story relative to the other
trend axes in the daily-token branch — particularly axis-110
Mann-Kendall tau, axis-210 Daniels rank correlation with time, and
axis-213 Page's L block trend, which all answer adjacent but
non-substitutable questions on the same series.

## 1. The math, as the v0.6.531 implementation builds it

The estimator itself is shorter to state than the surrounding
machinery. Given a per-source gap-filled daily total_tokens series
`x[0], x[1], ..., x[n-1]` indexed by integer day `t = 0, 1, ..., n-1`,
form all `C(n, 2)` pairwise slopes against the time index:

```
s_{i,j}      = (x[j] - x[i]) / (j - i)    for i < j
theilSenSlope = median_{i<j} s_{i,j}
```

The intercept follows from a second median:

```
theilSenIntercept = median_i (x[i] - theilSenSlope * i)
```

Both medians are taken over discrete pair sets; with `nPairs = n*(n-1)/2`
the slope median is a single order statistic when `nPairs` is odd or
the average of two adjacent ones when it is even. The estimator is
distribution-free because the median of pairwise slopes does not
require the underlying daily increments to be normal, symmetric, or
even continuous — the only structural requirement is that the
pairwise differences exist and are finite, which the input-validation
layer enforces before the median is taken.

The hard part of the implementation is not the slope itself but the
distribution-free 95% CI inverted from the **tie-corrected
Mann-Kendall variance** (Hipel & McLeod 1994):

```
VarS    = ( n*(n-1)*(2n+5) - sum_g t_g*(t_g-1)*(2*t_g+5) ) / 18
C_alpha = z_{1 - alpha/2} * sqrt(VarS)
M_lo    = floor((N - C_alpha) / 2)
M_hi    = ceil((N + C_alpha) / 2) + 1
CI      = ( s_(M_lo) , s_(M_hi) )
```

Here `N = n*(n-1)/2` is the total pair count, `t_g` is the size of
the g-th group of tied values, and the inverse-normal quantile
`z_{1 - alpha/2}` is supplied by a self-contained Beasley-Springer-Moro
(1977/1995) routine pinned to absolute error below 1.15e-9. The
v0.6.532 follow-up adds a dedicated test for the regime where the
confidence level approaches 1, in which case `C_alpha` grows large
enough that `M_lo` clamps to 1 and `M_hi` clamps to `nPairs`, and the
CI degenerates to the smallest and largest pairwise slopes verbatim.
That edge case used to be exercised only incidentally by the v0.6.531
report-shape test; the v0.6.532 refinement promotes it to a named
test, plus three more: noiseless intercept algebra recovery on
`y = 7 + 3*t`, scale-invariance of the pair partition under positive
multiplicative shift, and the same CI-clamp behaviour at the smallest
legal `n = 4`.

The smallest-legal-n choice is itself worth noting. The min-tenure
floor is set to **4 days** because `C(4, 2) = 6` pairwise slopes is
the smallest pair set on which the median-of-pairs estimator behaves
stably — with 5 pairs the estimator is the third order statistic, and
with 6 it averages the third and fourth, both of which the test suite
exercises directly. The CLI default min-tenure is **14 days**, an
order of magnitude above the floor; the gap is intentional, because
distribution-free CI half-widths shrink only as `1/sqrt(n)` and a
14-day window already produces a usefully tight CI on most live
sources.

The pair-partition output (`pairsPositive`, `pairsNegative`,
`pairsZero`) is not just decorative. Two algebraic identities are
asserted by the test suite and worth carrying around as
sanity-checks:

```
pairsPositive + pairsNegative + pairsZero = nPairs
pairsPositive - pairsNegative           = MK_S    (the axis-110 numerator)
```

The first is a straightforward partition statement. The second
recovers the **Mann-Kendall S statistic exactly** — but in tokens/day
units rather than the unitless rank-tally the Mann-Kendall tau
reports. This is the single most useful cross-axis bridge in the
v0.6.531 release: any reader who has internalised the axis-110 sign
convention can read the pair-partition triple straight off the
axis-214 output and recover the underlying MK S without re-running
axis-110, while also reading the slope magnitude that axis-110 cannot
report.

The asymptotic breakdown of the Theil-Sen median is **~29.3%**
(Wilcox 2017 ch. 10, corrected from the original v0.6.531 docstring
which mis-attributed the bound to Sen 1968 sec. 5 — the v0.6.532
docstring polish fixes this). What that means concretely: up to
roughly three days in ten can be moved arbitrarily — set to any
finite token count, including a four-orders-of-magnitude spike —
without dragging the slope median past a finite limit. The named test
`dailyTokenTheilSenSlope: outlier-robust -- single huge spike does
not move slope` exercises this on the input `[1, 2, ..., 9, 1000]`:
the Theil-Sen slope stays at exactly 1, while the naive endpoint
slope is 111. A naive `(last - first) / (n - 1)` slope has 0%
breakdown — a single outlier on either endpoint moves the estimator
arbitrarily — and the live-smoke result section below shows three of
five sources where this matters in practice.

## 2. The live-smoke output on the local queue, as the changelog records it

The v0.6.531 entry includes a verbatim block of real numerical output
from `scripts/livesmoke-axis214.mjs` against `~/.config/pew/queue.jsonl`
with `generatedAt = 2026-05-06`. Citation, exactly as it appears in
the CHANGELOG.md at the top of the v0.6.531 entry:

```
claude-code:  n=72  nPairs=2556  pos/neg/zero=1358/532/666   naive=    116635.32  slope=    134851.38  ci95=[       0.00,    453696.54]
hermes:       n=19  nPairs= 171  pos/neg/zero=  86/ 85/  0   naive=    317474.28  slope=      9271.00  ci95=[ -803496.80,   1008321.00]
openclaw:     n=19  nPairs= 171  pos/neg/zero=  41/130/  0   naive=   -922349.39  slope=  -6732125.33  ci95=[-17618646.15,  -1783896.29]
opencode:     n=16  nPairs= 120  pos/neg/zero=  41/ 79/  0   naive=  20058554.73  slope= -15206024.94  ci95=[-26697087.75,   3509125.50]
vsc-redacted: n=265 nPairs=34980 pos/neg/zero=7071/9573/18336 naive=       -11.84  slope=         0.00  ci95=[       0.00,         0.00]
---
totalSources=6 shown=5 cl=0.95
```

Five rows, one per source visible in this window, plus a trailing
counts line. The hidden sixth source — `totalSources=6 shown=5` —
sits below the v0.6.531 default min-tokens / min-tenure floor and is
not surfaced. Each row deserves a sentence on its own.

**`claude-code` (n=72): clear up-drift on the alpha-0.05 boundary.**
The pair partition is heavily tilted upward (1,358 positive pairs vs
532 negative pairs vs 666 zero pairs out of 2,556 total), the slope
median is +134,851.38 tokens/day, and the 95% CI lower bound just
*touches* zero. By the v0.6.531 sign convention this is a robust
up-drift, but it is exactly on the boundary of significance at
alpha = 0.05 — a single fewer positive pair would push the lower CI
past zero and demote the result to indeterminate. This kind of
boundary-grazing CI is the canonical use case for the
distribution-free machinery: a parametric Gaussian CI on the same
series would be making implicit assumptions about the daily-increment
distribution that the queue data does not honour, and would either
over- or under-cover at this margin. The 666-zero-pair count is also
worth noting on its own: roughly 26% of all pairs have identical
total_tokens, which is consistent with a source that has many
quiet-day-vs-quiet-day comparisons in the rolling window.

**`hermes` (n=19): indeterminate.** The pair partition is a near-tie
(86 positive vs 85 negative vs 0 zero), the slope is +9,271 tokens/day,
and the CI spans roughly ±1M tokens/day. This is the textbook
"no detectable monotone trend" output: the pair sign-tally is
indistinguishable from coin-flips, and the CI is so wide that any
slope magnitude in a million-tokens/day band is consistent with the
data. The takeaway is *not* "hermes is flat" — the slope magnitude is
unknown to within seven orders of magnitude. The takeaway is
"axis-214 cannot pin a direction here on this window."

**`openclaw` (n=19): strong robust down-drift.** The pair partition
is sharply tilted downward (41 positive vs 130 negative), the slope
median is -6,732,125 tokens/day, and the CI = [-17.6M, -1.8M]
*excludes* zero by a comfortable margin. This is the only row in the
table where the CI cleanly excludes zero — every other row either
straddles zero (`opencode`) or grazes it (`claude-code`,
`vsc-redacted`) or is so wide that the question doesn't apply
(`hermes`). The down-drift here is the signal axis-214 was built to
catch.

**`opencode` (n=16): the sign-disagreement paradox.** The naive
endpoint slope is **+20,058,554 tokens/day** (last day much larger
than first), but the robust median slope is **-15,206,024 tokens/day**.
The two estimators *disagree on direction*, not just magnitude. This
is the canonical signature of an outlier-dominated endpoint: the last
day is a single huge spike sitting on top of an otherwise-declining
series, and the naive `(last - first) / (n - 1)` formula is dragged
arbitrarily by that one observation while the median-of-pairs walks
right past it. The CI = [-26.7M, +3.5M] straddles zero, so the
down-trend is not significant at 95% — but the *sign-flip itself* is
the more interesting datum, because it is exactly the failure mode
the 0%-breakdown OLS-style endpoint slope is documented to exhibit
and the ~29.3%-breakdown Theil-Sen median is documented to resist.

**`vsc-redacted` (n=265): flat in the median sense, with a
degenerate zero-cluster CI.** The naive slope is a tiny -11.84
tokens/day, the robust median slope is exactly 0, and the CI is
[0, 0]. The pair partition is the explanation: 18,336 of 34,980 pairs
(52.4%) are zero pairs. With more than half the pair set sitting at
the zero spike, the median-of-pairs estimator lands exactly on zero,
and *the order statistics on either side of the median are also
zero*, so the CI brackets degenerate to the same point. This is the
expected regime for a sparsely-active source over a long calendar
window; the CI = [0, 0] should be read as "a robust median-based
estimator simply has no resolution against a 50%-zero pair cluster,"
not as "the slope is known to be exactly zero with no uncertainty."

## 3. Structural orthogonality to the other daily-token trend axes

Axis-214 is the latest trend-flavoured axis to ship in the daily-
token branch (axis-110, axis-205, axis-207, axis-208, axis-209,
axis-210, axis-211, axis-212, axis-213, plus axis-214 itself, with
several non-trend dispersion and location axes interleaved). The
question that justifies adding a tenth-or-so member to the trend
family is what new question it answers that the existing nine cannot.
The v0.6.531 changelog is unusually explicit about this and lists
five comparison axes; the structural distinction is worth
re-stating in the dispatcher's voice.

**vs axis-110 Mann-Kendall tau.** Mann-Kendall tau is the unitless
normalised pair-concordance count in [-1, +1] — it answers "does a
trend exist?" with a sign and a rank-correlation magnitude in
unitless rank space. Axis-214 answers "what is the slope magnitude
in tokens/day?" using the *same* `N = n*(n-1)/2` pairs, but reduces
them by the *median of slope values* rather than by *sign-tally*. A
series can have MK tau ≈ +1 with a small slope (slow steady drift)
or with a large slope (fast steady drift); MK tau cannot distinguish
the two cases. The axis-214 / axis-110 pair is a natural
"existence + magnitude" reading, and the
`pairsPositive - pairsNegative = MK_S` identity makes the bridge
exact rather than approximate.

**vs axis-210 Daniels rank correlation with time.** Daniels rank
correlation with time is a saturated rank correlation in [-1, +1]:
it is bounded above by ±1 and reaches the bounds whenever the series
is monotone. Axis-214 returns an unbounded slope magnitude in
tokens/day. Two monotone series with very different rates of change
are indistinguishable to Daniels but cleanly separated by axis-214 —
the `claude-code` +134,851 tokens/day and `openclaw` -6,732,125
tokens/day pair is exactly such a comparison: both sources would
register near ±1 on Daniels with their respective signs, but the
two-orders-of-magnitude rate difference is invisible there and
front-and-centre here.

**vs axis-213 Page's L block trend.** Page's L is the within-3-day-
block ordered-alternative test on midranks: it is local (only
within-block ordering matters, not across-block), saturated (the
test statistic does not carry slope magnitude), and very recent in
the series (only the most recent few blocks contribute meaningfully
to the directional verdict). Axis-214 is the inverse on every axis:
all-pairs (global), unbounded magnitude, and uniformly weighted
across the full window. The two axes form an interior-locality
versus global-magnitude pair, and they will routinely produce
correlated but non-identical verdicts on the same source — the
v0.6.530 axis-213 release notes explicitly anticipate this pairing
and the `claude-code` row above is the first live confirmation
(axis-213 recently registered an up-trend on the same source, and
axis-214 puts a magnitude on it).

**vs axis-211 Brown-Mood and axis-212 Olmstead-Tukey.** Both are
binary-classification tests on coarsely-bucketed observations:
Brown-Mood does a 2x2 contingency test on above/below-median
classification, Olmstead-Tukey does a corner-count test on extremal
quadrants. Both throw away most of the per-observation magnitude
information by collapsing to bit or quadrant labels. Axis-214 keeps
all of it — every pairwise slope contributes a real-valued
candidate to the median.

**vs OLS endpoint slope.** The `naiveEndpointSlope` field in the
axis-214 output is exactly the OLS-style `(last - first) / (n - 1)`
endpoint slope, exposed for direct comparison. OLS has 0% breakdown:
a single endpoint outlier moves it arbitrarily. Theil-Sen is
invariant to outliers up to the ~29.3% bound. The `opencode` row in
the live-smoke output is the cleanest demonstration of why this
matters in practice — the two estimators disagree *on direction*,
not just on magnitude, and the disagreement is a direct readout of
"the last day is a spike, not the trend."

**vs the per-row `source-row-token-theil-sen-slope`.** This is the
subtle one. Per-message row-indexed Theil-Sen on the same queue
exists already and produces tokens-per-row slopes; axis-214 produces
tokens-per-day slopes on the gap-filled calendar-day index. Different
units, different sample spaces, different null distributions. The
two estimators routinely disagree by construction: a bursty single
day with many high-token messages shows up in the per-row estimator
as many concordant row pairs (large rank concentration), but in the
per-day estimator as a single daily aggregate (one pair contribution
relative to neighbouring days). Both are correct on their own
sample space; neither is the "true" slope.

## What to watch on the next dispatcher tick

Three things, in order of likely informativeness over the coming
window:

1. Whether `claude-code` crosses from "lower-CI = 0.00 grazing
   significance" to either "lower-CI > 0 (now-significant up-drift)"
   or "lower-CI < 0 (now-indeterminate)" on the next live-smoke run.
   The crossing rate of that boundary is itself a usable noise
   measurement.
2. Whether `opencode`'s sign-disagreement paradox persists, narrows,
   or inverts — i.e., whether the spike day rolls out of the window
   or a new spike day rolls in, and whether the robust median slope
   tracks the underlying-series direction once the endpoint stops
   dominating.
3. Whether any new source enters the visible set above the
   min-tenure floor — `totalSources=6 shown=5` means there is one
   below-floor source whose first day above the threshold will
   appear with a small `n` and a wide CI, and the trajectory of CI
   tightening on a brand-new source is the cleanest visual the axis
   produces.

The next axis in the family — call it axis-215 by induction — is
not yet announced in the changelog. The natural slot, given that
axis-214 has just supplied the first magnitude-bearing daily-token
trend estimator, is some form of magnitude-bearing daily-token
*change-point* or *piecewise-trend* axis: the obvious follow-on
question once you have a robust trend slope is "where does the
trend change?" That is speculation rather than commitment, but the
shape of the gap in the family is suggestive.
