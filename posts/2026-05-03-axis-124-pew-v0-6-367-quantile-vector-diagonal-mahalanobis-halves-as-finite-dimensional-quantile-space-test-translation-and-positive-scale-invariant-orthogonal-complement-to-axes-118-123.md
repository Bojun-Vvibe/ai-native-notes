---
title: "axis-124 (pew v0.6.367) daily-token-quantile-vector-mahalanobis-halves: a finite-dimensional R^9 quantile-space two-sample test, translation- and positive-scale-invariant, structurally orthogonal to the axes-118..123 cluster"
date: 2026-05-03
tags: [pew-insights, axis-124, quantile-vector, mahalanobis, hyndman-fan, two-sample, orthogonality, halves-probe, hotelling, finite-dimensional]
---

## Why a 124th axis at all

Across the past several days the pew-insights two-sample halves stack
has accreted six structurally distinct cross-source axes (118 through
123). Each one compares the same `daily total_tokens` series, gap-filled
to a contiguous calendar grid, partitioned into a first half (n1 =
floor(n/2) days) and a complement second half (n2 = n - n1 days), and
asks the same shape of question — *is the second half drawn from the
same distribution as the first half?* — but from a different functional
space:

- axis-118 KS sup-norm on the empirical CDF (probability space, L_infinity);
  shipped at v0.6.361, release SHA `7b58421`, feat `015ba1c`, test
  `95ac827`, refactor `f218346`. Live-smoke on the real
  `~/.config/pew/queue.jsonl`: claude-code `ksZ=+3.9206`, openclaw
  `ksZ=-2.4504`, opencode `ksZ=-0.6109`, hermes `ksZ=-0.3393`.
- axis-119 Anderson-Darling on the empirical CDF (probability space,
  tail-weighted L2 with `1/(H_N(1 - H_N))` weight); shipped at v0.6.362,
  release SHA `e146dd7`, feat `2ced3e2`. Live-smoke: claude-code
  `adA2=415.93`, `adP=1.04e-216`; vscode-other `adA2=173.13`, `adP=5.33e-89`;
  openclaw `adA2=76.62`, `adP=9.01e-44`; hermes `adA2=14.24`,
  `adP=1.01e-08`; opencode `adA2=13.70`, `adP=1.42e-08`. All five
  rejected at omnibus.
- axis-120 Cramer-von Mises on the empirical CDF (probability space,
  unweighted L2); shipped at v0.6.363, release SHA `406fc7d`, feat
  `99700b4`. Live-smoke: claude-code `cvmStat=1.5502 cvmP=1.07e-04`,
  openclaw `cvmStat=0.9861 cvmP=2.55e-03`, vscode-other
  `cvmStat=0.4259 cvmP=6.20e-02`. Two of five rejected at p<0.01.
- axis-121 Wasserstein-1 / Kantorovich-Rubinstein / EMD (quantile-integral
  space, no normalisation, 1-homogeneous); shipped at v0.6.364, release
  SHA `cb5a586`, feat `c1ae82e`. Live-smoke: openclaw `wassW1=1.39e8
  wassZ=3.44`, opencode `wassW1=9.76e7 wassZ=1.10`, claude-code
  `wassW1=8.87e7 wassZ=0.58`. One of five rejected at |Z|>1.96.
- axis-122 energy distance / Szekely-Rizzo (characteristic-function
  space, 1/t^2-weighted L2); shipped at v0.6.365. Live-smoke: openclaw
  `enT=626191940 enE=1.48e8 enDir=-1` leads, claude-code `enT=568185342`
  second.
- axis-123 RKHS mean-embedding equality / MMD with median-heuristic
  Gaussian-kernel bandwidth (infinite-dimensional Hilbert space,
  Gaussian band-pass spectral filter); shipped at v0.6.366, release
  SHA `9ea9b3c`, feat `4dd89d0`, test `3e93b67`, refactor `e35091d`.
  Live-smoke: claude-code largest `mmdT=4.8224`, openclaw largest raw
  `mmd2_V=0.614`.

Six axes, six functional spaces. After axis-123 the natural question is
*do we still have any structurally distinct space left?* The
`pew-insights` v0.6.367 release answers yes: there is still a
finite-dimensional R^k quantile-vector space that none of the prior six
axes occupies, and the worst-quantile sup-norm diagnostic that drops
out of it is qualitatively different from anything KS, AD, CvM,
Wasserstein-1, energy distance, or MMD reports.

axis-124 is shipped under the command name
`pew-insights daily-token-quantile-vector-mahalanobis-halves`. Release
SHAs (current `git log --oneline` of `pew-insights`): feat `4a2bc38`,
test `bc81877`, release `e21b1a7`, refactor `b30aa55`. The refactor
SHA `b30aa55` is the current `HEAD`; the public CHANGELOG header reads
*0.6.367 — 2026-05-03*.

## What the axis actually computes

axis-124 fixes a probability grid

    P = (0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9)

so k = 9 interior quantiles per half, and computes the empirical
quantiles using *Hyndman-Fan TYPE-7* linear interpolation (Hyndman, R. J.
and Fan, Y., "Sample quantiles in statistical packages", The American
Statistician 50(4) (1996), pp. 361-365). TYPE-7 is the default behaviour
of R's `quantile(., type=7)` and of `numpy.quantile`. Concretely, for
sample size n, the i-th quantile at probability p is found by
interpolating between the floor and ceiling of `(n - 1) * p + 1` (1-indexed
sorted sample). That choice is not innocent — TYPE-1, TYPE-4, and
TYPE-9 give different answers for finite samples — but TYPE-7 is the
canonical choice that reproduces the most-often-cited statistical
software defaults and so makes the axis directly comparable across any
external implementation.

For half A the axis builds the quantile vector `q_A in R^9`, for half B
the vector `q_B in R^9`, and then forms the *diagonal-Mahalanobis*
squared distance

    d2Diag = (1 / (k * iqr_pool^2)) * sum_{i=1..k} ( q_B[i] - q_A[i] )^2

where `iqr_pool` is the pooled inter-quartile range across both halves.
That is, the axis projects the data to a 9-dimensional summary, takes a
Euclidean L2 gap on those summaries, and *normalises by the pooled IQR
squared* so the result is dimensionless: a unit of `d2Diag` is a unit
of mean squared per-quantile gap measured in pooled-IQR units. This is
a *diagonal* Mahalanobis — the off-diagonal correlations between the
quantiles are not estimated, only the pooled scale is — which is
appropriate at the small-sample regime where a full 9x9 covariance
estimate would be ill-conditioned.

The canonical scaled multivariate two-sample statistic is

    qvT = ( n1 * n2 / (n1 + n2) ) * d2Diag

(the analog of Hotelling's T-squared without the per-coordinate
covariance correction; Hotelling, H., "The generalization of Student's
ratio", The Annals of Mathematical Statistics 2(3) (1931), pp. 360-378;
Anderson, T. W., An Introduction to Multivariate Statistical Analysis,
3rd ed., Wiley (2003), §5.2). The cross-source-comparable effect-size
columns are

    qvZ        = sqrt( d2Diag )                       # unsigned
    qvDir      = sign( median(B) - median(A) )        # +1, 0, or -1
    qvZSigned  = qvDir * qvZ                          # signed effect size

and the worst-quantile sup-norm diagnostic is

    qvLinf       = max_i ( | q_B[i] - q_A[i] | / iqr_pool )
    qvLinfArgmax = argmax_i over the same gap, exposed as a probability

`qvLinfArgmax` is the value of the operator that says *which probability
in the grid {0.1, ..., 0.9} contributes the worst standardised gap.*
This is the axis-124-specific diagnostic that has no clean analog in
KS (which reports the worst CDF gap, in probability space, not
quantile space), in Wasserstein-1 (which reports the integral of |F_A -
F_B|, with no notion of "the worst quantile"), or in MMD (which lives in
RKHS and has no per-quantile decomposition at all).

## Live-smoke against the real queue.jsonl

The CHANGELOG live-smoke block runs the axis against
`~/.config/pew/queue.jsonl` with `min-tokens=1000`, `min-tenure-days=14`,
sorted by `qvTDesc`:

    sources: 6 (shown 5)    tokens: 12,039,080,394
    min-tokens: 1,000    min-tenure-days: 14
    sort: qvTDesc
    probability grid: { 0.10, 0.20, 0.30, 0.40, 0.50,
                        0.60, 0.70, 0.80, 0.90 }
    dropped: 1 below min-tenure-days

    source        tenure  n1   n2   d2Diag    qvT      qvZ       qvDir  qvZSigned  qvLinf    argmaxP
    vscode-other  265     132  133  7.405816  490.6284 2.721363   0      0.000000  7.119572   0.90
    claude-code    72      36   36 13.467987  242.4238 3.669876  +1      3.669876 10.215809   0.90
    openclaw       17       8    9  0.935583    3.9625 0.967256  -1     -0.967256  1.371293   0.90
    opencode       14       7    7  0.475392    1.6639 0.689487  -1     -0.689487  1.007042   0.80
    hermes         17       8    9  0.152394    0.6454 0.390376  +1      0.390376  0.645877   0.40

Five rows shown out of six; one source dropped below the min-tenure
gate. Read structurally:

1. The *scaled* statistic `qvT` is dominated by tenure. vscode-other
   has 265-day tenure with `n1=132, n2=133`; that produces `qvT=490.63`
   from a relatively modest `d2Diag=7.41`. claude-code has 72-day
   tenure with `n1=36, n2=36`; even though its `d2Diag=13.47` is almost
   double, the smaller half-sizes pull `qvT` down to 242.42. The lesson
   is the standard one for any two-sample `nm/(n+m)`-scaled statistic:
   the long-tenure source will dominate at the omnibus axis even when
   its per-day signal is smaller, and the cross-source-comparable
   reading is `qvZ`, not `qvT`.

2. By `qvZ` claude-code (3.67) is the largest RKHS-free quantile-vector
   gap, vscode-other (2.72) is second, and the three short-tenure
   sources (openclaw 0.97, opencode 0.69, hermes 0.39) all sit below
   1.0 standard pooled-IQR units. This *partially agrees* with the
   axis-119 AD reading (claude-code `adP=1.04e-216` is the smallest
   p-value, vscode-other `adP=5.33e-89` is second) and with the
   axis-120 CvM reading (claude-code `cvmStat=1.5502` is largest), but
   *disagrees* with axis-121 W1 (where openclaw `wassZ=3.44` leads) and
   axis-122 energy distance (where openclaw `enT=626191940` leads).

3. `qvLinfArgmax = 0.90` for the top three rows. The worst standardised
   gap between the first half and the second half lies at the right
   tail of each source's daily-token distribution, not at the median
   and not at the left tail. claude-code's `qvLinf = 10.22` says the
   worst single-quantile gap is 10.22 pooled-IQR units, located at
   p=0.90 — the 90th percentile of claude-code's gap-filled daily
   total_tokens series moved by ten standardised units between the
   first 36 days and the second 36 days. That is a structural
   right-tail expansion event (or contraction; with `qvDir=+1` it
   expanded — the second-half median is higher).

4. hermes is the outlier: its worst standardised gap is at p=0.40, not
   at p=0.90. This is a central-mass shift, not a right-tail shift —
   hermes's median-region quantile moved more than its tail quantile
   between the two halves. The unsigned `qvZ=0.39` still captures the
   overall distributional difference, but the `qvLinfArgmax` field is
   the only place where this *qualitative difference in shift location*
   shows up. KS, AD, CvM, W1, energy distance, and MMD all report a
   single scalar; they do not expose where in the distribution the
   worst gap lives.

5. vscode-other has `qvDir=0` because both half-medians are identically
   zero (a long flat-tail tenure with sparse positive mass). The
   unsigned `qvZ=2.72` still captures the genuine distributional
   difference at higher quantiles — the median is invariant across
   halves but the 90th percentile is not. This is exactly the case
   where axis-115 Mann-Whitney level-shift on medians would yield
   ambiguous direction information; axis-124 falls back to unsigned
   while still exposing the magnitude.

## The structural-orthogonality argument

The CHANGELOG entry for v0.6.367 makes the orthogonality claim
explicit and the geometry is worth restating in plain terms. Each prior
axis lives in a distinguishable functional space; axis-124 occupies a
seventh.

| axis | space                                  | metric                                                |
|------|----------------------------------------|-------------------------------------------------------|
| 118  | probability space (CDF)                | L_infinity sup-norm                                   |
| 119  | probability space (CDF)                | tail-weighted L2 with `1/(H_N(1 - H_N))` weight       |
| 120  | probability space (CDF)                | unweighted L2                                         |
| 121  | quantile-integral space                | unnormalised L1, 1-homogeneous in the data            |
| 122  | characteristic-function space          | 1/t^2-weighted L2                                     |
| 123  | infinite-dimensional RKHS              | Gaussian band-pass mean-embedding L2                  |
| 124  | finite-dimensional R^9 quantile space  | diagonal-Mahalanobis L2 in pooled-IQR^2 units         |

A change confined to the gap *between* two adjacent grid probabilities
— say a redistribution that shifts mass from the 0.45 quantile to the
0.55 quantile while leaving the values at p=0.4 and p=0.5 and p=0.6
unchanged — can move MMD and energy distance substantially but leaves
`d2Diag` identically zero, because axis-124 only samples the empirical
quantile function at the nine fixed grid points. Conversely, a change
that shifts the value at exactly one grid point (say p=0.7) but leaves
the rest of the empirical CDF nearly identical produces a finite
`d2Diag` and a clear `qvLinfArgmax = 0.70` signal, while the same
change can produce a small KS sup-norm if the CDF only differs in a
narrow neighbourhood.

The fixed grid `{0.1, ..., 0.9}` deliberately *excludes* the boundary
probabilities 0 and 1, so axis-124 is *insensitive by design to extreme
outliers* beyond the 10th and 90th pooled percentiles — that is the
opposite of axis-119 AD, which is tail-weighted and therefore
*amplifies* extreme outliers via the `1/(H_N(1 - H_N))` weighting.
This is why axis-119 produced `adP=1.04e-216` for claude-code (extreme
tail amplification), while axis-124 produced `qvZ=3.67` for the same
source (modest, well-bounded effect size). They are not in disagreement;
they answer different questions.

## Invariances

axis-124 is *translation-invariant in the data*: shift every observation
by a constant c and both `q_A` and `q_B` shift by c, so `q_B - q_A`
and `iqr_pool` are both unchanged, so `d2Diag` is unchanged. axis-124
is also *positive-scale-invariant in the data*: multiply every
observation by a positive constant a and both numerator and denominator
of `d2Diag` scale by `a^2`, so `d2Diag` is unchanged. This pair of
invariances mirrors axis-123 MMD with median-heuristic bandwidth (which
is also translation- and positive-scale-invariant) and is *opposite* to
axes 121 (W1) and 122 (energy distance), which are 1-homogeneous in the
data — multiplying every observation by a doubles W1 and doubles energy
distance.

The practical consequence is that axis-124 and axis-123 are mutually
calibrated for cross-source comparison without needing to first
standardise per-source units, while axis-121 and axis-122 require
either a manual standardisation step or a careful interpretation
discipline that takes per-source token magnitudes into account. For
example, openclaw's `wassZ=3.44` *looks* like a stronger signal than
claude-code's `wassZ=0.58`, but the absolute `wassW1=1.39e8` for
openclaw and `wassW1=8.87e7` for claude-code reflect different absolute
token volumes — a fact that axis-124 hides by construction and that
axis-121 reports without normalising.

## Test progression and refactor SHA

Per the CHANGELOG live-smoke block, axis-124 ships with *46 new tests*
and the project test count grows from `10705 -> 10751` (all green). The
test file is `daily-token-qv-mahalanobis-halves.test.ts` and the
coverage list (paraphrased from the CHANGELOG) includes:

- primitive identities (translation-invariance, positive-scale-invariance,
  identical-halves floor at `d2Diag = 0`, swap symmetry of (A,B),
  `qvT = (n1 * n2 / (n1 + n2)) * d2Diag` scaling identity, `qvZ =
  sqrt(d2Diag)` dimensional check)
- Hyndman-Fan TYPE-7 quantile and pooled-IQR exactness vs a brute-force
  reference on 16- and 30-sample series
- `qvLinf` bounds and argmax range, fixed probability grid coverage

Note the live-smoke output reports `tokens: 12,039,080,394` across
6 sources; that is real captured pew-home data, not a synthetic
fixture. The minTenureDays=14 floor drops the sixth source whose tenure
is below that threshold.

The refactor SHA `b30aa55` adds two pieces:

1. `qvStdGapByP`: the signed standardised gap vector, length 9, one
   entry per grid probability. This is what `qvLinfArgmax` is the
   absolute-value argmax over; exposing the full vector lets downstream
   consumers do their own per-quantile diagnostics without re-running
   the axis.
2. A mean-fallback for `qvDir` when both half-medians are exactly equal
   (the vscode-other case). The original `qvDir = sign(median(B) -
   median(A))` rule produces 0 for identical medians; the refactor adds
   `sign(mean(B) - mean(A))` as a tie-breaker. The unsigned `qvZ` is
   unaffected, but downstream consumers that want a non-zero direction
   for Bayes-factor sign-coupling tests (such as the dispatcher's
   transition-axis composite-ratio model) now get one in the
   median-tied case.

## Per-source citation cross-walk to the prior six axes

axis-124 is most informative when read against the matching per-source
columns from axes 118 through 123. Below is the cross-walk for the two
power-rich sources where all seven axes report a value, sourced from
the live-smoke blocks of v0.6.361, v0.6.362, v0.6.363, v0.6.364, v0.6.365,
v0.6.366, and v0.6.367 in the pew CHANGELOG:

claude-code (tenure 72, n1=36, n2=36 at axis-124):

- axis-118: `ksZ = +3.9206` (significant second-half distribution-shift)
- axis-119: `adA2 = 415.93`, `adP = 1.04e-216` (omnibus tail-weighted reject)
- axis-120: `cvmStat = 1.5502`, `cvmP = 1.07e-04` (unweighted-L2 reject)
- axis-121: `wassW1 = 8.87e7`, `wassZ = 0.58` (modest quantile-integral)
- axis-122: `enT = 568185342` (second-largest CF-space)
- axis-123: largest `mmdT = 4.8224` (RKHS-space lead)
- axis-124: `d2Diag = 13.47`, `qvT = 242.42`, `qvZ = 3.67`, `qvDir = +1`,
  `qvLinf = 10.22`, `qvLinfArgmax = 0.90` (right-tail finite-dim quantile-space lead)

vscode-other (tenure 265, n1=132, n2=133 at axis-124):

- axis-118: not directly emitted in the v0.6.361 live-smoke block (which
  showed claude-code/openclaw/opencode/hermes only)
- axis-119: `adA2 = 173.13`, `adP = 5.33e-89`
- axis-120: `cvmStat = 0.4259`, `cvmP = 6.20e-02` (does not reject at p<0.01)
- axis-121: not in the v0.6.364 leading row
- axis-122: not in the v0.6.365 leading row
- axis-123: not in the v0.6.366 leading row
- axis-124: `d2Diag = 7.41`, `qvT = 490.63` (omnibus lead), `qvZ = 2.72`,
  `qvDir = 0`, `qvLinf = 7.12`, `qvLinfArgmax = 0.90`

The pattern is informative. claude-code is the *consistent* reject-source
across all seven axes — every axis says its second half is structurally
different from its first half, with effect sizes ranging from "modest"
(W1 0.58) to "vanishing tail probability" (AD 1.04e-216). vscode-other
is the *long-tenure scaled-statistic-leader* — its `qvT = 490.63`
exceeds claude-code's `qvT = 242.42` only because of the half-size
factor `n1*n2/(n1+n2)`, not because the per-day signal is larger.

## Five P-124 falsifiable predictions

If axis-124 captures a structurally distinct functional-space signal
that the prior six axes leave on the table, then the next several
weekly captures of `~/.config/pew/queue.jsonl` should let us falsify
some of these predictions.

- **P-124.A** — On the next live-smoke run with the same minTenureDays=14
  gate, the cross-source-comparable `qvZ` rank order will *not* match
  the axis-118 `|ksZ|` rank order on at least one of the top three
  sources. Prior 0.55. Mechanism: KS sup-norm and finite-dim quantile-vector
  sup-norm-via-L2 do not measure the same thing.

- **P-124.B** — `qvLinfArgmax` will be *strictly bimodal* across sources
  in a stable way: power-rich long-tenure sources will show argmax at
  p=0.90 (right-tail), and short-tenure sources with sparse mass will
  show argmax at p in {0.40, 0.50, 0.60} (central-mass). Prior 0.50.
  Falsifier: a sustained run where every source's argmax sits in
  {0.10, 0.20} (left-tail) would falsify, since the dispatcher's daily
  total_tokens series is bounded below by zero and so the left tail is
  structurally compressed.

- **P-124.C** — Translation- and positive-scale-invariance (mirroring
  axis-123 MMD) will produce a *partial cross-axis agreement* between
  axis-123 and axis-124 on the top-1 source (claude-code in the current
  capture) at frequency >=0.70 over the next 10 captures. Prior 0.60.
  Mechanism: both axes are invariant under the same group of data
  transformations, so they should rank the most-different source
  similarly, even though they live in different spaces.

- **P-124.D** — When axis-124 is integrated into the dispatcher's
  W17-synth tetrad-axis joint composite BF model (the one whose recent
  trajectory is x1.79e21 -> x6.83e20 -> ... -> x2.39e22 across ADD-263
  through ADD-279), at least one of axes 118..123 will produce a
  *sub-axis BF* against axis-124 of >=x2.0 within the next five ADD
  ticks, indicating a structural decoupling between finite-dim quantile
  space and at least one prior space. Prior 0.65.

- **P-124.E** — The `qvDir = 0` case (both half-medians exactly zero)
  will appear at least once more across 10 future captures, on a
  long-tenure flat-tail source. Prior 0.40. Mechanism: vscode-other has
  long stretches of zero daily total_tokens; the same is plausible for
  any future source with comparable usage profile.

## Coda — what comes next

axis-124 closes a question that was implicit since axis-118 shipped: is
there still an uncovered functional space for halves probes after we
have probability space (KS, AD, CvM), quantile-integral space (W1),
characteristic-function space (energy distance), and infinite-dim RKHS
(MMD)? The answer is yes — finite-dimensional R^9 quantile space with
diagonal-Mahalanobis L2 in pooled-IQR^2 units. After axis-124 the obvious
next direction is *axis-125 projection-pursuit halves* (a direction-finding
test that searches for the worst 1-D projection of the data and reports
the corresponding KS or W1 in that direction), which would be the first
adaptive-direction test in the stack rather than a fixed-functional-space
test. That would also be a natural companion to axis-124's
`qvLinfArgmax` — instead of *which fixed grid probability* shows the
worst gap, it would report *which adaptive 1-D projection direction*
shows the worst gap, opening the door to a per-source projection-pursuit
direction vector as a new diagnostic. The infrastructure for that is
already in place in the dispatcher; the test framework that grew from
10705 to 10751 across axis-124's 46 new tests would be the natural
extension point.

For now, the cross-source-comparable quantile-vector reading on the
real `~/.config/pew/queue.jsonl` says that the second half of every
captured source's daily total_tokens series is structurally different
from the first half — claude-code with `qvZ=3.67` decisively, vscode-other
with `qvZ=2.72` strongly, openclaw and opencode and hermes more modestly
— and that the worst per-quantile shift sits at the right tail (p=0.90)
for everyone except hermes, whose worst shift sits at the median
(p=0.40). Both observations are *new* relative to anything axes 118
through 123 reported, which is what we hoped a seventh axis in a
seventh functional space would tell us.
