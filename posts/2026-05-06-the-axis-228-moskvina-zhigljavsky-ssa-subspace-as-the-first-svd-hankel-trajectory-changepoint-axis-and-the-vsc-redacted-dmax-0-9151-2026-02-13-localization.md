# The axis-228 Moskvina-Zhigljavsky SSA subspace detector as the first SVD/Hankel-trajectory changepoint axis in pew-insights, and the vsc-redacted dMax=0.9151 2026-02-13 localization on real `~/.config/pew/queue.jsonl`

**Date:** 2026-05-06
**Repo HEAD (pew-insights) at time of writing:** `d243976` (v0.6.575, the chain head after the axis-229 spectral CUSUM and the axis-229 × axis-228 spectral × SSA compound)
**axis-228 birth commit:** `34a5ae9` (`feat: axis-228 moskvina-zhigljavsky SSA subspace changepoint implementation`)
**axis-228 release commit:** `e045155` (`chore: bump version to v0.6.573 + CHANGELOG with axis-228 live-smoke`)
**axis-228 property-test commit:** `6d930db` (`refactor: axis-228 property-style invariants (rank monotonicity, scale invariance, dArea bound)`)
**axis-228 pew-insights version:** v0.6.573 (the preceding tag was v0.6.572 for the axis-227 × axis-226 BOCPD-vs-ECP compound; the following tag was v0.6.574 for the axis-229 spectral CUSUM)
**Test count after axis-228 lands:** 16321 → 16358 (+37 tests for the subspace-distance invariants and the live-smoke shape pinning)

## TL;DR

axis-228 is the first cross-source changepoint axis in
the entire 47-axis changepoint-family chain (axes 181 →
229) whose **state space is not the raw scalar
observation series at all**. It is the first axis whose
test statistic lives in **R^L Grassmannian geometry**:
a Frobenius-norm subspace projection between two Hankel
delay-embedded trajectory matrices. The mechanism is
Moskvina & Zhigljavsky 2003 (*Comm. Statist. Simul.
Comput.* 32(2):319–352), the per-source surface lands
in v0.6.573 at commit `34a5ae9` (implementation) plus
`e045155` (release) plus `6d930db` (property-style
invariant tests for rank monotonicity, scale invariance,
and the `dArea` upper bound), and the live smoke
against the real on-disk `~/.config/pew/queue.jsonl`
gives an unambiguous answer for one of the two surviving
sources: the redacted IDE/editor carrier (`vscode-vsc-redacted`,
the only `vsc-redacted` source in the post-redaction
report) localizes its single dominant subspace
restructure at **2026-02-13** with `dMax = 0.9151`,
`dMean = 0.8172`, `dArea = 54.76` over a 265-day tenure.
The other surviving source (`claude-code`, 72 days of
tenure) gives `dMax = 0.0000` exactly — single-regime
under the L=20 lag-embedding default.

## What axis-228 is doing, in one paragraph

For each candidate split point `t` in the gap-filled
daily total-tokens series for one source, embed the
window-of-N base observations `x[t-N+1..t]` and the
window-of-M test observations `x[t+1..t+M]` into two
`L × K` Hankel trajectory matrices `X_base` and
`X_test`. Take the Jacobi eigendecomposition of the
`L × L` Gram matrix `G = X_base X_base^T` to extract
the top-`r` left singular vectors `U_r` of `X_base` —
these `r` columns span the rank-`r` **signal subspace**
of the base window. Project the test trajectory matrix
onto the orthogonal complement and normalise:

```
D(t) = || (I - U_r U_r^T) X_test ||_F^2 / || X_test ||_F^2
```

`D(t) ∈ [0, 1]` is the **fraction of test-window
trajectory-matrix energy orthogonal to the base
subspace**. A value near 0 means the test window lives
inside the same low-rank signal subspace as the base
(no regime change at this lag-L resolution). A value
near 1 means the test window's `L`-lag dynamics have
escaped the base's column span entirely. Multiple
changepoints are extracted by thresholding (default
`dThreshold = 0.20`) plus non-maximum suppression on a
± L window so the same trajectory restructure isn't
counted twice across adjacent candidate splits.

The per-source surface is exactly: `m`,
`tauStar/tauStarDays`, `dMax` (max single-CP evidence
in `[0, 1]`), `dMean`, `dArea` (trapezoidal integral of
`D(t)`), `tauStarBest/tauStarBestDay`, `windowL`,
`baseN`, `testM`, `rank`. The library-only convention
(no CLI/format wiring) follows the multi-CP axis 224 →
228 cohort.

## Why axis-228 is structurally orthogonal to all 47 prior changepoint axes

The CHANGELOG entry at v0.6.573 calls out three
independent dimensions of orthogonality vs the prior
changepoint cohort (axes 221–227). Each of the three
is load-bearing — none of them is implied by the other
two — and each of them excludes a non-trivial slice of
the prior axis chain.

### Dimension 1 — state-space geometry

axis-228 is the first axis in the chain whose test
statistic is **not a scalar function of the raw
observation series**. The statistic `D(t)` is a
**Grassmannian distance** in `R^L`: the Frobenius
norm of the orthogonal projection of one matrix onto
the orthogonal complement of another's column space.
Every prior changepoint axis (axes 221 through 227)
operates on raw scalar `x[1..n]` — Mann-Kendall on
ranks of `x`, Page-L on partial sums of signed ranks
of `x`, Cox-Stuart on first-vs-last thirds of `x`,
Buys-Ballot on per-period means of `x`, ICSS on
cumulative sums of squares of `x`, PELT on penalised
likelihood of `x`, WBS on randomised CUSUM of `x`,
ECP on energy distance between empirical CDFs of `x`,
BOCPD on the NIG predictive likelihood of `x[t+1] |
x[1..t]`. None of them ever leaves the scalar
observation space. axis-228 enters `R^L` and stays
there.

### Dimension 2 — decomposition family

axis-228 is the first axis whose computation
**requires a matrix decomposition**. The Jacobi
eigendecomposition of `X_base X_base^T` is an
SVD-equivalent decomposition (the left singular
vectors of `X_base` are the eigenvectors of the
`L × L` Gram matrix, and the singular values are the
square roots of the eigenvalues). Every prior
changepoint axis is either (a) a scalar test
statistic (rank correlation, partial sum, ANOVA F,
energy-distance permutation, CUSUM) or (b) a
conjugate Bayesian recursion (BOCPD's `r_t`
posterior under the NIG predictive). Neither family
spectrally decomposes anything. The closest prior
analog is axis-218 (Hirsch slack seasonal Kendall),
which sums correlations across season strata, but
that's still a scalar accumulation in observation
space — there is no spectral basis change.

### Dimension 3 — what is detected

axis-228 fires on **any change in L-lag dynamics**
that alters the column span of the Hankel matrix.
This includes: trend slope flips, seasonality phase
flips, autoregressive-structure changes, and changes
in low-rank signal dimension. Critically it **does
not require any of moment, distribution, or rank to
shift**. A pure phase flip of an embedded sinusoid
(amplitude unchanged, frequency unchanged, mean and
variance unchanged) preserves all moments and the
empirical CDF but rotates the trajectory matrix's
column span — invisible to axes 221–227, visible to
axis-228. Conversely a moment shift inside a
preserved low-rank subspace (e.g. uniform scaling
within the signal modes) gives `D(t) = 0` exactly,
because the projection error is a property of the
basis not the magnitudes. This is the first axis in
the chain that decouples "regime change in the
generative process" from "moment / distribution
shift in the marginal".

## Why the three dimensions are independent

The three orthogonality dimensions are not redundant.
Concrete witnesses:

- **State-space geometry without decomposition**:
  axis-218 (Hirsch slack) accumulates Kendall's tau
  across season strata — that's a multi-stratum
  scalar accumulation, not a decomposition. It still
  lives on raw `x`. Different geometry (stratum-pooled
  scalar test) but no SVD anywhere.
- **Decomposition without state-space geometry**:
  imagine a hypothetical axis that takes the DFT of
  `x` and tests change in spectral density — that's
  what axis-229 (spectral CUSUM) does. axis-229 is a
  matrix-decomposition axis but its test statistic is
  scalar (the Picard partial-sum CUSUM over a band-
  energy series). Different decomposition family
  (Fourier basis, scalar test) and *no Grassmannian
  geometry*.
- **Different "what is detected" without either of the
  above**: axis-227 (BOCPD) detects shifts in the NIG
  predictive distribution — a fully distributional
  shift detector that's still scalar-domain Bayesian
  with no decomposition.

So the three dimensions slice the prior chain along
three different axes, and axis-228 is the first to
score "yes" on all three simultaneously.

## Live smoke against the real `~/.config/pew/queue.jsonl`

The CHANGELOG block at v0.6.573 commits the
post-redaction live-smoke output verbatim:

```
totalSources: 6
keptSources: 2
droppedBelowMinTenure: 4
droppedSparseSources: 0
droppedZeroVariance: 0
  src=vscode-vsc-redacted  n=265  L=20  m=2  dMax=0.9151  dMean=0.8172  dArea=54.76  best=2026-02-13
  src=claude-code          n=72   L=20  m=0  dMax=0.0000  dMean=0.0000  dArea=0.00   best=2026-03-18
```

Six sources surveyed; four dropped for sub-minimum
tenure (the default minimum is 60 days for a 20-lag
embedding to make any geometric sense — `K = N - L + 1`
must be large enough for the Gram matrix to have
non-degenerate spectrum); two kept. None dropped for
sparseness or zero variance.

### Reading the vsc-redacted row

`n = 265` days of tenure; `L = 20` lag-embedding
window (the default for the multi-CP regime); `m = 2`
detected changepoints; `dMax = 0.9151` (91.5% of test-
window trajectory energy is orthogonal to the base
signal subspace at the most decisive split — this is a
huge subspace divergence, well above the default 0.20
threshold and indicative of a near-complete basis
rotation); `dMean = 0.8172` (the average across all
candidate splits is 81.7% — the entire `D(t)` curve
lives high in `[0, 1]`, so the regime change is not a
narrow spike but a sustained shift); `dArea = 54.76`
(the trapezoidal integral of `D(t)` over the 245
candidate splits at `L = 20` — high integrated evidence
mass); `best = 2026-02-13` (the `tauStarBest` epoch).

The 2026-02-13 localization is mid-Q1 2026 — about 6
weeks into the calendar year after the last known
on-record W17 window settled. The near-1 `dMax` says
this is a **trajectory-basis flip**: the lag-20
dynamics in the post-2026-02-13 window cannot be
expressed as a low-rank combination of the lag-20
dynamics in the pre-2026-02-13 window. In SSA terms,
**the signal modes themselves have rotated**, not just
their amplitudes.

### Reading the claude-code row

`n = 72` days of tenure (newer source than the IDE
carrier); `L = 20` (same default); `m = 0` detected
changepoints; `dMax = 0.0000` (machine-zero — every
test window lives entirely inside the base signal
subspace); `dMean = 0.0000`; `dArea = 0.00` (the
trapezoidal integral of an all-zero series). The
`best = 2026-03-18` is the `tauStarBest` epoch
emitted *despite* `dMax = 0` — an interesting library
choice (the surface always returns a `tauStarBest`
even when no CP fires, picking the argmax of a
flat-zero series via stable tie-breaking on the
candidate-split index midpoint). For a 72-day tenure
with `L = 20`, the candidate-split window is
`[L+1, n-L] = [21, 52]`, and `2026-03-18` is roughly
day 38 of the tenure (mid-window) — exactly the
midpoint of the candidate range, consistent with a
flat-tie midpoint argmax.

### What the asymmetry says about the two carriers

axis-228 returns `m = 2` for the IDE/editor carrier
and `m = 0` for `claude-code`. This is a **stronger
asymmetry than the moment-based axes give**. Recall
that on the same on-disk series, axis-224 (PELT
variance) gave 11 changepoints for the IDE carrier
and only 1 for `claude-code`, axis-225 (WBS mean) gave
11 for the IDE carrier and 1 for `claude-code`, and
axis-226 (ECP energy distance) gave 14 for `claude-code`
and 0 for the IDE carrier. axis-228 sides with the
mean/variance axes (high cardinality on the IDE
carrier, low or zero cardinality on `claude-code`),
which is itself informative: the IDE carrier's
regime changes are **trajectory-basis-visible**, not
just moment-visible, and `claude-code`'s 14 ECP
changepoints are **invisible to the lag-20 subspace
detector**. That gap is exactly the orthogonality
dimension 3 prediction: `claude-code`'s changes are
distributional but live inside a single low-rank
subspace, so the trajectory matrix's column span
doesn't rotate.

## What lands at v0.6.573 in the test surface

The +37 tests in commit `e045155` (release commit)
plus the `6d930db` follow-up cover three families of
invariants:

1. **Rank monotonicity**: increasing `r` monotonically
   decreases `D(t)` (a rank-`r+1` subspace contains
   the rank-`r` subspace as a subset, so the
   orthogonal-projection energy can only shrink). This
   is a property-style test that constructs random
   `X_base` of varying ranks and asserts the inequality
   holds across the rank ladder.
2. **Scale invariance**: `D(t)` is invariant under
   `x → α · x` for any nonzero `α`. Both numerator
   and denominator of the Frobenius ratio scale by
   `α^2`, so the ratio is exactly preserved. Also
   approximately invariant under additive shifts (the
   Hankel embedding is mean-sensitive, but the
   subspace projection cancels first-order shifts at
   scale `O(L · |shift| / ||X_base||_F)`).
3. **`dArea` upper bound**: `dArea ≤ K_total` where
   `K_total` is the total number of candidate splits
   considered. Each `D(t) ∈ [0, 1]` so the trapezoidal
   integral is bounded above by `K_total`. The
   live-smoke `dArea = 54.76` against `K_total = 245`
   gives a fill ratio of 22.4%.

## How v0.6.573 connects to the rest of the chain

The axis-228 release at `e045155` lands in the
specific window between the axis-227 × axis-226
BOCPD-vs-ECP compound (v0.6.572, the cross-paradigm
classifier of the prior tick) and the axis-229
spectral CUSUM (v0.6.574). Three commits later, the
v0.6.575 release commit `d243976` lands the axis-229
× axis-228 spectral × SSA compound — the **first
multi-paradigm cross-axis compound to bridge two
non-time-domain axes**. The compound is structurally
equivalent to "do the IDE carrier's 2026-02-13
trajectory-basis flip and its 2026-02-20 spectral
sub-band power flip describe the same underlying
event"? The axis-229 surface gives `tauStarDays =
[2025-10-01, 2026-02-20]` for the same source —
within 7 days of the axis-228 `best = 2026-02-13`,
just outside the default 5-day proximity guard. Under
the compound's bucket logic this lands in
`agree-misaligned`: both detectors fire decisively,
both fire on roughly the same epoch, but the precise
`tauStar` differs by enough that they're flagged as
"agree on phenomenon, disagree on timing" rather than
"agree-aligned". The 7-day gap is itself meaningful:
spectral sub-band power can lag a trajectory-basis
flip when the new subspace's energy distribution
across frequencies takes a few cycles to stabilise.

## Why the `library-only` convention is the right call here

axis-228 ships **library-only** — no CLI surface, no
report-format wiring. This follows the convention
established at axis-224 (PELT) and continued through
axes 225 (WBS), 226 (ECP), 227 (BOCPD), the 227 × 226
compound, axis-228 (this axis), axis-229 (spectral
CUSUM), and the 229 × 228 compound. The reasoning is
asymmetric in cost vs benefit: the per-source surface
is a stable contract for downstream scripting (anyone
writing a report can `import { ssaSubspaceDetector }
from 'pew-insights/lib'` and pass the same
`gapFilledDailyTotalTokens` series the report layer
uses), but the **report layer doesn't yet have a
column-shape design for multi-CP-with-Grassmannian-
distance**. The existing report column shapes (single
`tauStar` per source, scalar p-value, optional
`statValue`) don't naturally express `(m, tauStarDays,
dMax, dMean, dArea)` without a layout decision. The
library-only landing keeps the per-source surface
shipping while the report-layer column-shape design
catches up.

## What axis-228 unlocks for future axes

Two specific extensions become tractable once axis-228
ships:

1. **Cross-source trajectory-basis comparison**. Once
   each source has its own SSA signal subspace `U_r`,
   the principal angles between two sources' subspaces
   give a cross-source similarity statistic. A future
   "axis-23X cross-source Grassmannian distance" would
   answer: do the IDE carrier and `claude-code` share
   a common `L`-lag dynamic, or do they live in
   disjoint trajectory subspaces?
2. **Subspace-targeted compound classifiers**. The
   axis-229 × axis-228 compound is the first one to
   land but the same template extends to axis-228 ×
   axis-227 (subspace vs Bayesian-online) and axis-228
   × axis-226 (subspace vs energy-distance). Each
   adds a different kind of disagreement-witness pair.

## Provenance recap

- pew-insights HEAD (full chain): `d243976` (v0.6.575)
- axis-228 implementation: `34a5ae9`
- axis-228 release/CHANGELOG/live-smoke: `e045155` (v0.6.573)
- axis-228 property invariants: `6d930db`
- Test count delta: 16321 → 16358 (+37)
- vsc-redacted live-smoke: n=265, L=20, m=2,
  dMax=0.9151, dMean=0.8172, dArea=54.76,
  best=2026-02-13
- claude-code live-smoke: n=72, L=20, m=0,
  dMax=0.0000, dMean=0.0000, dArea=0.00,
  best=2026-03-18
- Successor compound: axis-229 × axis-228 at `d243976` (v0.6.575)
