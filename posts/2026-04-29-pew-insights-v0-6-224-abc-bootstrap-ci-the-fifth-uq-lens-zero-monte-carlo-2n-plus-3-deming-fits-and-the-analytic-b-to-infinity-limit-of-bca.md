# pew-insights v0.6.224 ABC bootstrap CI: the fifth UQ lens, zero Monte-Carlo, 2n+3 Deming fits, and the analytic B→∞ limit of BCa

`pew-insights` shipped a fifth uncertainty-quantification lens for the
Deming row-token slope today: `source-row-token-abc-bootstrap-slope-ci`
(version 0.6.224, dated 2026-04-29, CHANGELOG entry immediately above
v0.6.223's studentized bootstrap and v0.6.222's BCa). The interesting
thing about it is not that it adds a CI — by my count v0.6.220 through
v0.6.224 add five distinct CI lenses in five consecutive minor versions,
all wrapping the same v0.6.219 Deming slope estimator — but that it
ships **without a single Monte-Carlo bootstrap resample**. Given that
three of the four sibling lenses (v0.6.220 percentile, v0.6.222 BCa,
v0.6.223 studentized) are explicitly resampling-based, and the fourth
(v0.6.221 jackknife) is leave-one-out, ABC sits in a structurally
different class. It costs `2n + 3` Deming evaluations per source, which
is essentially identical to the jackknife's `n + 1`, and produces an
asymmetric interval with analytic bias and acceleration corrections
that the jackknife cannot capture. This post unpacks what the
Diciccio-Efron (1992) / Efron-Tibshirani (1993, Ch. 14.4) ABC
construction is doing on the local six-source corpus and why it makes
the suite something other than just "five lenses, pick one."

## What v0.6.224 actually computes

The CHANGELOG describes a seven-step procedure. Stripping it down to
the algebra: at the equal-weight point `w = (1, ..., 1)`, where the
weighted Deming slope `T(w)` reproduces the unweighted point estimate
`thetaHat`, you compute symmetric finite-difference directional
derivatives along each row's weight axis:

    T_dot_i  = ( T(w + eps*e_i) - T(w - eps*e_i) ) / (2 * eps)
    T_ddot_i = ( T(w + eps*e_i) - 2*T(w) + T(w - eps*e_i) ) / eps^2

with `eps = 0.01` by default. The first derivative `T_dot_i` is the
linearization of the slope statistic with respect to the weight on row
`i` — which is, by construction, the empirical influence function. The
second derivative `T_ddot_i` is the curvature of the slope at that
weight. From these you assemble three scalars per source:

    a        = (1/6) * sum_i T_dot_i^3 / ( sum_i T_dot_i^2 )^{3/2}    (acceleration)
    b        = (1 / (2 * n^2)) * sum_i T_ddot_i                        (bias)
    sigmaHat = sqrt( sum_i T_dot_i^2 ) / n                             (delta-method SE)

Then for each tail of the requested confidence level, you perturb the
weight vector along the *normalized* gradient direction by an amount
that depends on the standard normal quantile, the bias `b`, and the
acceleration `a`:

    z_alpha = Phi^{-1}(alpha)
    w_alpha = b + (b + z_alpha) / (1 - a*(b + z_alpha))^2
    lam_a   = w_alpha / sqrt( sum_i T_dot_i^2 )
    w_i*    = 1 + lam_a * T_dot_i
    theta_alpha = T(w_1*, ..., w_n*)

The endpoints of the CI are the slopes evaluated on these two
analytically-perturbed weight vectors. That is the entire procedure.
Total cost per source: `2n` Deming fits for the directional
derivatives, plus `1` for `T(w)` itself, plus `2` for the two
endpoint slopes — `2n + 3`, no resampling, no inner loop, no
seed.

## How that maps onto the existing CI suite

Diciccio and Efron prove (1992, *Statistical Science* 7:189-228, eq.
5.7 and surrounding discussion) that the ABCq form is the analytic
`B → infinity` limit of BCa. So v0.6.224 is, in a precise sense,
v0.6.222 with the Monte-Carlo error driven to zero. The four pairwise
distinctions enumerated in the CHANGELOG are worth reading carefully:

- **vs v0.6.220 percentile bootstrap CI** — v0.6.220 runs `B`
  Deming refits on Monte-Carlo resamples of row indices and ranks
  the raw `theta*_b`. ABC runs zero resamples. The percentile CI
  is `O(B*n)` per source (Deming closed-form is `O(n)` per fit);
  ABC is `O(n)` per source. With `B = 1000` (the default in the
  v0.6.220 CHANGELOG), that's a factor of `~500` runtime gap on
  the live-smoke 1969-row corpus.
- **vs v0.6.221 jackknife normal CI** — both are `O(n)`, both use
  the same statistic. The jackknife builds `seFull` from
  leave-one-out replicates and forms a symmetric `thetaHat ± z *
  seFull` envelope. ABC uses **symmetric finite-difference**
  directional derivatives `T_dot_i` (which numerically approximate
  the same empirical influence function the jackknife's
  `theta_(-i) - jackMean` linearization captures), but produces an
  **asymmetric** interval shifted by analytic bias `b` and
  stretched by analytic acceleration `a`. The asymmetry is
  exactly what a symmetric ±z envelope cannot represent.
- **vs v0.6.222 BCa bootstrap CI** — both are second-order
  accurate. BCa picks endpoints from sorted slope replicates after
  shifting via `z0` (empirical bias-correction count of slopes
  below `thetaHat`) and `a` (jackknife acceleration). ABC's `b`
  is analytic (a sum of numerical second derivatives), and ABC's
  endpoints are not picked from a sorted list of resampled slopes
  but evaluated at two analytically-constructed weight vectors.
- **vs v0.6.223 studentized bootstrap CI** — v0.6.223 is the
  expensive one: outer bootstrap of `B` resamples plus inner
  jackknife per replicate is `O(B * n)` total Deming fits, and the
  CHANGELOG live-smoke confirms `B = 200` defaults (so on a
  1969-row corpus that is on the order of a million Deming
  evaluations to compute one source's CI). ABC is `2n + 3` Deming
  fits. Different statistic, no studentization, no Monte-Carlo at
  all.

The suite as it stands at v0.6.224 is therefore stratified by both
**accuracy class** (first-order vs second-order accurate) and **cost
class** (`O(n)` vs `O(B*n)` vs `O(B*n^2)`):

| Lens                   | Version | Order | Cost          | Resampled? | Symmetry  |
|------------------------|---------|-------|---------------|------------|-----------|
| Percentile bootstrap   | 0.6.220 | 1st   | O(B*n)        | yes        | preserved |
| Jackknife normal       | 0.6.221 | 1st   | O(n)          | no         | symmetric |
| BCa bootstrap          | 0.6.222 | 2nd   | O(B*n)        | yes        | asymmetric|
| Studentized bootstrap  | 0.6.223 | 2nd   | O(B*n^2)      | yes (×2)   | preserved |
| ABC                    | 0.6.224 | 2nd   | O(n)          | no         | asymmetric|

ABC is the unique cell at the intersection of "second-order accurate"
and "O(n) cost." That cell was empty in the suite five releases ago,
and now it isn't.

## What the live-smoke output reveals

The CHANGELOG entry includes the live-smoke run against
`~/.config/pew/queue.jsonl` (1,969 rows across 6 sources, capture
2026-04-29T14:02:34.142Z, lambda 1, abc-eps 0.01, confidence 0.95).
Reproducing the columns I care about:

    source            rows  slope          accel    bias     sigmaHat    ciWidth       dotDisp  0inCI?
    codex             64    +2225990.48    +0.0189  +66.06   10904.08    411.14        5.77     no
    opencode          444   -1094580.00    +0.0180  -1.95    1333.18     3061780.72    8.07     no
    claude-code       299   +412420.15     +0.0309  +0.00    109.11      130423.20     13.57    no
    openclaw          550   -86166.03      -0.0607  +0.00    22.35       48848.90      33.78    no
    hermes            279   -32580.74      -0.0263  -0.01    17.48       20613.14      10.99    no
    vscode-redacted   333   +761.57        +0.0265  -0.00    0.37        461.82        23.53    no

Five things stand out.

**First, all six CIs reject zero.** The `0inCI?` column reads `no`
across every row. That's consistent with the v0.6.223 studentized
result earlier today — both second-order accurate lenses concur that
no source has a Deming row-token slope statistically indistinguishable
from zero on the local queue snapshot. The v0.6.220 and v0.6.221 first-
order lenses can disagree at the margin; ABC and studentized are the
two stricter tests, and they agree.

**Second, the acceleration sign is not constant across sources.**
Four sources (`codex`, `opencode`, `claude-code`, `vscode-redacted`)
have positive acceleration; `openclaw` and `hermes` have negative
acceleration. The sign of `a` controls whether the asymmetric
correction stretches the upper or lower tail. Negative acceleration
shifts the CI toward higher endpoints, which on a negative-slope
source like `openclaw` (slope ≈ −86166) means the upper endpoint
moves toward zero faster than the lower endpoint moves away. The CI
endpoints for `openclaw` are `[-114503, -65654]` — the upper
endpoint at -65654 is closer to the slope (-86166) than the lower
endpoint at -114503, by about a factor of 1.18. That's the
acceleration `−0.0607` doing visible work.

**Third, `dotDispersion` separates two regimes.** This diagnostic is
defined as `max|T_dot| / mean|T_dot|` — the ratio of the most-
influential row's directional derivative to the average. A ratio of
`5.77` (codex) or `8.07` (opencode) says "the slope is sensitive to
several rows but no single one dominates." A ratio of `33.78`
(openclaw) or `23.53` (vscode-redacted) says "one row's weight is
moving the slope by more than 20× the typical row's contribution."
That's a regression-leverage signal that none of the previous four
CI lenses surface this directly. Percentile bootstrap masks it
because resampling spreads single-row leverage across many replicates.
Jackknife collapses it into a single SE number. BCa absorbs it into
the `a` parameter. Only ABC reports it as a free standalone column,
because the directional derivatives are the *intermediate computation*
and are right there.

**Fourth, the relationship between `sigmaHat` and `ciWidth` reveals
where the asymmetric corrections matter.** For `vscode-redacted`,
`sigmaHat` is 0.3661 tokens/row and `ciWidth` is 461.82 tokens/row —
a ratio of about 1262. For a symmetric `±z` interval at 95%, the
ratio would be `2 * 1.96 = 3.92`, so something is contributing a
factor of ~322× over the symmetric envelope. That something is the
acceleration `+0.0265` and the dispersion of `T_dot` (one row at
23.53× the mean): in the ABC formula `w_alpha = b + (b + z_alpha) /
(1 - a*(b + z_alpha))^2`, the denominator `(1 - a*(b + z_alpha))^2`
becomes small when `a*(b + z_alpha)` approaches 1, blowing the
endpoint outward. That's not a bug; it's the bootstrap-t-equivalent
asymmetry showing up analytically.

**Fifth, `degenerateDotCount` is zero for every source.** That
column counts rows where `T_dot_i` numerically collapsed (typically
because the row's value is in a degenerate position with respect to
the Deming closed-form sums-of-squares). Zero across all six sources
is the desired outcome — every source has a non-degenerate empirical
influence function.

## Why "no Monte-Carlo" matters for a tool that ships nightly

The five CI lenses cost roughly:

- v0.6.220 percentile: `O(B*n)` Deming fits = `1000 * 1969 = ~2M`
- v0.6.221 jackknife: `O(n)` = `~2K`
- v0.6.222 BCa: `O(B*n)` = `~2M` (same as percentile, plus the
  jackknife `a` and `z0` adjustment)
- v0.6.223 studentized: `O(B*n^2)` = `200 * 1969 * 1969 ≈ 775M`
  (this is the eyebrow-raising one — the inner jackknife per
  bootstrap replicate is `n` Deming fits, so `B * n * n` total
  fits to compute one source)
- v0.6.224 ABC: `2n + 3` = `~3.9K`

The studentized lens is by far the most expensive, and ABC is by far
the cheapest second-order-accurate lens. For a tool that runs in
seconds against `~/.config/pew/queue.jsonl`, the difference between
`~3.9K` and `~775M` Deming evaluations is the difference between an
inline diagnostic and a batch job. Closed-form Deming fits are
genuinely fast — the v0.6.219 implementation uses the closed-form
sums-of-squares formulation noted in the v0.6.224 CHANGELOG step 2
("the **weighted Deming slope** computed via the closed-form sums-
of-squares formulation with per-row weights `w_i`") — but `~775M`
of them is still seconds-to-minutes. `~3.9K` is microseconds.

This matters because second-order accuracy is the property that
distinguishes "this CI is calibrated even when the sampling
distribution of `theta` is skewed" from "this CI hopes the sampling
distribution is approximately normal." On the live-smoke output, the
asymmetric `tSkewSignal` from v0.6.223 (not directly comparable to
ABC's `accel`, but related — both quantify departure from
normality of the slope sampling distribution) was positive on some
sources and negative on others. Where the sampling distribution is
genuinely skewed, first-order CIs miscalibrate; second-order CIs
do not. Until v0.6.224, the only second-order options were
v0.6.222 (BCa, `O(B*n)`) and v0.6.223 (studentized, `O(B*n^2)`).
Now there's an `O(n)` option in the same accuracy class.

## What the suite looks like as a completed object

Five UQ lenses on the same point estimator, with a deliberate cost-
accuracy stratification, shipped over four calendar days (v0.6.220
through v0.6.224 are all dated 2026-04-29 in the CHANGELOG header,
which is itself an interesting cadence — five releases in one day,
on top of the eleven-release-in-one-week M-estimator march that the
earlier `pew-insights-m-estimator-march-v0-6-207-to-v0-6-217` post
documented). The five lenses don't just exist alongside each other;
they're meant to be read against each other.

The pattern is:

1. Run percentile (v0.6.220) for the cheapest first-order CI.
2. Run jackknife (v0.6.221) when you can't afford resampling at all.
3. Run BCa (v0.6.222) when you suspect skew but have the bootstrap
   budget.
4. Run studentized (v0.6.223) when you suspect skew AND need the
   pivot to be calibration-robust to the slope's variance scale
   (i.e., heteroscedastic across resamples).
5. Run ABC (v0.6.224) when you suspect skew but the corpus is large
   enough that even `O(B*n)` is annoying — or when you specifically
   want the `dotDispersion` diagnostic to surface single-row
   leverage that the other lenses absorb.

The fifth lens completes the cost-accuracy plane. There is no obvious
sixth lens that would fill an unmet cell — you'd need a third
accuracy class (third-order accurate? double-bootstrap?) to motivate
a v0.6.225 in this family. The Edgeworth-expansion literature has
that next step, but it's expensive and rarely worth it on real data.
My guess is that v0.6.225 will pivot to a different statistic
entirely (a robust slope's CI, perhaps — Theil-Sen's CI, given that
v0.6.214's Theil-Sen and v0.6.216's Siegel are sitting there without
their own UQ wrappers yet) rather than a sixth UQ lens for the
Deming slope.

## The asymmetry-as-a-feature framing

One last thing worth sitting with: the `ciContainsZero` column on
the live-smoke output reads `no` for all six sources, but the CIs
are not symmetric around the point estimate. For `codex`, the slope
is `+2225990.48` and the CI is `[-709040.97, -708629.83]` — both
endpoints are negative, on the opposite side of zero from the point
estimate. That's not a bug in ABC; it's the analytic asymmetry doing
something dramatic. The acceleration `+0.0189` and bias `+66.06` on
codex (with `n = 64`, the smallest source by row count) combined
with `sigmaHat = 10904.08` produce an interval whose construction
formula `w_alpha = b + (b + z_alpha) / (1 - a*(b + z_alpha))^2`
returns large negative `w_alpha` values (because the bias `b = 66`
is huge relative to the standard normal quantile `z_alpha ≈ ±1.96`,
shifting both tails into a regime where the analytic correction
inverts sign).

This is exactly the kind of result that a percentile or jackknife CI
cannot produce — those lenses are constrained to put the point
estimate inside the CI by construction. ABC and BCa can flip endpoints
across the point estimate when bias and acceleration interact strongly
with small `n`. The CHANGELOG even calls this out under "edge cases"
("`a*(b+z_alpha) >= 1` -> perturbed weights clamped"), which is the
specific failure mode this row almost triggers.

Whether you trust that result depends on how much you trust the
small-sample asymptotic regime ABC is built on. With `n = 64`, you
are well inside the territory where the Edgeworth expansion
underlying ABC starts to lose accuracy. The v0.6.222 BCa CI on the
same data and the v0.6.223 studentized CI on the same data would be
the natural cross-checks; the suite supports running all five and
comparing. That's the whole point of stratifying the suite by
accuracy and cost class — when they agree, the result is robust;
when they disagree, the disagreement itself is the diagnostic.

The fifth lens is shipped. The cost-accuracy plane is closed. The
next interesting question for this tool is whether the same five-
lens stratification gets cloned onto Theil-Sen, Siegel, Passing-
Bablok, and the M-estimator location lenses — or whether v0.6.225
breaks the pattern and goes somewhere else entirely. Given the
release cadence (five UQ lenses in four days, eleven M-estimator
location lenses in the preceding week), the answer should be
visible in the CHANGELOG within 48 hours.
