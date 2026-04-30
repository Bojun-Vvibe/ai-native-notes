---
title: "The unbounded-above axis: pew-insights v0.6.255 axis-26 Palma ratio (S90/S40) as the first non-Pigou-Dalton-universal cross-lens diagnostic after the five-axis dispersion sprint, plus the v0.6.256 palmaHypothesisDistance refinement"
date: 2026-04-30
tags: [pew-insights, axis-26, palma-ratio, polarisation, lorenz-process, dispersion-sprint, ci-half-width, hypothesis-distance, refinement, unbounded-above]
---

## Where this axis sits in the cross-lens diagnostic stack

Pew-insights `v0.6.255` ships axis-26 — `source-row-token-slope-ci-
lens-width-palma` — at sha=`6327554` (feat) → `297d9d3` (test, 30
tests) → `922be1c` (release). The same release introduces a Palma-
specific render flag, sort key, and alert filter, all wired through
the same six per-source CI lenses (`bootstrap`, `jackknife`, `bca`,
`studentizedT`, `abc`, `profileLikelihood`) that have been the spine
of the v0.6.219 Deming-slope uncertainty-quantification suite for the
last seven minor releases. The follow-on refinement `v0.6.256` lands
at sha=`81a72f8` and adds `palmaHypothesisDistance`, an alert flag,
and the corresponding render line.

Axis-26 is the *twenty-sixth* cross-lens diagnostic shipped on this
suite, and the *sixth* cross-lens dispersion-or-polarisation
diagnostic in the W18 sprint (axes 21 through 26). The five-axis
sprint immediately preceding it — Gini (axis-21), Theil GE(1)
(axis-22), Atkinson (axis-23), QCD (axis-24, sha=`298f9bb`), Hoover
(axis-25, sha=`03871ba`) — was a deliberate orthogonal-property tour
across moment-based, entropic, welfare-loss, order-statistic, and
geometric formulations of "how unequal is the cross-source CI half-
width vector?" Each axis was specifically chosen so that its
mechanical content was orthogonal to the previously-shipped axes:
Gini integrates the Lorenz gap, Theil computes `Σ p log p`, Atkinson
applies a CRRA welfare loss, QCD takes two value-quantiles
`(Q1, Q3)`, Hoover takes the sup of `(p − L(p))`.

Axis-26 breaks that pattern in three structurally important ways:

1. **It is unbounded above.** Palma `∈ [0, +∞)`. Every prior axis was
   bounded (Gini and Atkinson by 1, Theil GE(1) by `log n`, QCD by 1
   in its standard parameterization, Hoover by 0.5). Unboundedness is
   not a defect — it carries genuine information about polarisation
   regimes that the bounded axes compress.
2. **It is non-universal under Pigou-Dalton transfers.** Mean-
   preserving transfers entirely within the bottom 40%, the middle
   50%, or the top 10% of the Lorenz process leave Palma *unchanged*.
   Every axis 21-25 was Pigou-Dalton-universal: any mean-preserving
   regressive transfer reduced the dispersion measure regardless of
   where in the distribution it occurred.
3. **It is a two-point evaluation, not a functional.** Gini, Theil,
   Atkinson, and Hoover are functionals that integrate or sup over
   the entire Lorenz process. QCD is a two-point evaluation but on the
   *value* distribution. Palma is the only axis that takes a two-
   point evaluation on the *cumulative-mass* (Lorenz) process and
   combines them as a ratio.

Together those three properties make axis-26 the first diagnostic in
the cross-lens suite designed to characterize *polarisation*
specifically, as distinct from the dispersion-and-spread family that
axes 21-25 covered. This post walks through the mechanical definition,
the implementation choices that make it work at small `n`, the live-
smoke result against `~/.config/pew/queue.jsonl`, the v0.6.256
refinement (`palmaHypothesisDistance`), and what the axis-26
sensitivity profile changes about the cross-lens reading-protocol that
the W18 sprint has been building.

## Mechanical definition

The Palma ratio per the Cobham-Sumner-Palma (2011, 2013) formulation,
as implemented in axis-26:

```
Palma = (1 − L(0.9)) / L(0.4)
      = topDecileMassShare / bottomFourDecilesMassShare
```

The numerator is the share of total cross-source CI half-width
captured by the *top decile* of the source distribution; the
denominator is the share captured by the *bottom four deciles*. The
ratio is unbounded above: as the bottom-40% mass approaches zero with
the top-10% mass bounded away from zero, Palma diverges to `+∞` and
the implementation clamps to a finite sentinel `1e12` with a separate
boolean flag `palmaIsInfinite = true`.

The Lorenz process `L(p)` for axis-26 is the cumulative-mass curve of
per-source CI half-widths normalized to sum 1. With `n` typically
small (`4-12` shared sources in practice), the Palma cut-points
`p = 0.4` and `p = 0.9` will land on non-integer index boundaries for
most `n`. Axis-26 evaluates `L(p)` by piecewise-linear interpolation
between the `n+1` anchor points `(k/n, c_k)` for `k = 0, 1, ..., n`,
where `c_k` is the cumulative mass after sorting half-widths
ascending and summing the first `k`. This makes the canonical
cut-points exact for any `n`, not only multiples of 10.

The piecewise-linear-interpolation choice deserves a moment of
attention. It is the *consistent* estimator under the assumption that
the half-widths are sampled from a continuous underlying distribution,
and it preserves the boundary identities `L(0) = 0`, `L(1) = 1`
exactly. Alternative choices — step-function (right- or left-
continuous), kernel-smoothed, or fitted-Lorenz — would each introduce
either non-monotonicity (kernel smoothing can do this near the
boundaries with small `n`) or estimator bias that grows as `n` shrinks
toward 4. The piecewise-linear form is the minimal-bias choice for
`n ∈ [4, 12]`.

## Per-lens columns and the degeneracy taxonomy

Axis-26 emits per-lens, for each of the six CI lenses:

- `nShared`, `meanHalfWidth`, `totalHalfWidth` — the standard cross-
  lens prefix.
- `s40 = L(0.4)` — the bottom-40% cumulative mass share.
- `s90 = 1 − L(0.9)` — the top-10% cumulative mass share.
- `s50middle = 1 − s40 − s90` — the Palma-hypothesis "constant
  middle" mass.
- `palma`, `palmaIsInfinite` — the ratio and divergence flag.
- `concentrationLabel` — six-class categorical: `extreme`
  (`palmaIsInfinite` or `palma > 4`), `high` `(2, 4]`, `moderate`
  `(1, 2]`, `balanced` `(0.5, 1]`, `inverted` `[0, 0.5]`,
  `degenerate`.
- `degenerateFlag`, `degenerateReason` — `'too-few-sources'` (`n < 4`),
  `'zero-mass'`, `'zero-bottom-mass'`, `'non-finite'`.

The degeneracy taxonomy is mechanically more interesting than the
prior axes' degeneracy treatment because Palma can be both
*divergent* (numerator non-zero, denominator zero) and *undefined*
(both numerator and denominator zero). The implementation distinguishes
these: a zero-mass row (all CI half-widths exactly zero) is
`'zero-mass'` and Palma is undefined; a row where the top 10% has
non-trivial mass but the bottom 40% has zero mass is `'zero-bottom-
mass'` and Palma is treated as `+∞` with `palmaIsInfinite = true`.
The two cases require different downstream handling — `'zero-mass'`
should be filtered out of all aggregations, while `'zero-bottom-mass'`
contributes a divergence signal that argmax-style aggregators
(`mostExtremeLens`) should privilege.

The report-level aggregation honors this: `mostExtremeLens` prefers
`palmaIsInfinite` rows first, then falls back to `argmax(palma)` over
finite non-degenerate rows. `mostBalancedLens` takes `argmin(|palma −
1|)` over finite non-degenerate rows. `meanPalma`, `medianPalma`,
`maxPalma`, `minPalma`, and `rangePalma` are all computed over finite
non-degenerate values only. `nDegenerate`, `nExtreme`, and
`nBalancedOrInverted` are explicit count emissions.

## Live-smoke result against the live queue

The `v0.6.255` CHANGELOG block documents the live-smoke run against
`~/.config/pew/queue.jsonl` with `n=6` shared sources across all six
lenses (the same `n=6` substrate used in axes 21-25 live-smoke runs).
Per the CHANGELOG, the headline numbers are:

```
meanPalma   = 139.8873
medianPalma =  58.7869
maxPalma    = 568.4542   lens=abc      (S40=0.0010, S90=0.5828)
minPalma    =   7.3959   lens=bca
rangePalma  = 561.0583
nExtreme    = 6/6
nDegenerate = 0
```

(The `bca` minimum row is reported in the CHANGELOG with no explicit
S40/S90 quoted, but the Palma value `7.3959` together with the
degeneracy classification `nDegenerate = 0` confirms the row is
finite and non-degenerate.)

Three things stand out:

1. **All six lenses are in the `extreme` concentration class.** With
   `palma > 4` as the lower threshold for `extreme`, even the
   `min = 7.3959` lens is comfortably inside the class. The cross-
   source CI half-width distribution is, on the live queue,
   *heavily* polarised across every uncertainty-quantification lens.
2. **The range is enormous.** `rangePalma = 561.0583` against a
   `meanPalma = 139.8873` gives a coefficient of range-to-mean of
   roughly `4.0`, an order of magnitude larger than the analogous
   range-to-mean ratios for axes 21-25 (Gini, Theil, Atkinson, QCD,
   Hoover all kept range-to-mean below `0.4` on the same `n=6`
   substrate per their respective CHANGELOG live-smoke blocks). The
   unboundedness above is producing genuinely larger spread, not
   just larger absolute values.
3. **The `abc` lens dominates.** `S40 = 0.0010` is roughly three
   orders of magnitude smaller than `S90 = 0.5828`. The `abc` lens
   concentrates more than half the cross-source CI half-width mass
   into the top decile while leaving the bottom four deciles with
   essentially zero mass. This is a polarisation signature that the
   bounded axes (Gini in particular, which would clip near 1.0 at
   this regime) cannot resolve.

The `mostExtremeLens = abc` finding is mechanically interesting
because `abc` was *not* the lens that axes 21-25 flagged as most
extreme on the same substrate. Axis-25 Hoover live-smoke (per the
v0.6.253 CHANGELOG) had `maxH = 0.8177` at `lens=abc` — so `abc` was
already the Hoover-max lens. But axis-24 QCD (`maxQCD = 0.9853` at
`lens=profileLikelihood` per v0.6.252 live-smoke) had a different
extremum lens, and axes 21-23 had still others. Axis-26 confirms
the Hoover-max-lens identification at extreme strength: when the
underlying distribution is polarisation-heavy (large `S90`, small
`S40`), Hoover and Palma both flag the same lens, but Palma's
unbounded-above scale makes the *degree* of polarisation legible
where Hoover's `[0, 0.5]` bound saturates.

This is the structural payoff of axis-26's unboundedness. The
bounded axes can rank polarisation regimes from low to high; the
unbounded axis can also distinguish "polarised" from "extremely
polarised" from "near-divergent." The live queue is, by Palma's
metric, *near-divergent* at `lens=abc` (`S40 = 0.0010` is one
half-width away from triggering `palmaIsInfinite`).

## The v0.6.256 refinement: palmaHypothesisDistance

The Cobham-Sumner-Palma (2013) empirical claim — the Palma hypothesis
proper — is that across well-formed cross-country income inequality
data, the *middle 50%* of the distribution captures roughly half the
mass, with the variation in inequality showing up as the ratio
between the top 10% and bottom 40% (and only weakly in the middle).
In Lorenz-process terms, the empirical claim is `s50middle = L(0.9)
− L(0.4) ≈ 0.5`.

The v0.6.256 release at sha=`81a72f8` adds the diagnostic that
quantifies *agreement with the Palma hypothesis*:

```
palmaHypothesisDistance = |s50middle − 0.5|
```

with `palmaHypothesisDistance ∈ [0, 0.5]`. Smaller values indicate
better agreement with the empirical-constant-middle target. Degenerate
rows are clamped to `0.5` (the worst possible value for a valid
distribution) so a `--sort hypothesis-distance-desc` cannot
accidentally promote degenerate lenses to the top.

The v0.6.256 CHANGELOG live-smoke block documents the
`palmaHypothesisDistance` numbers against the same `n=6` substrate:

```
abc                  middle-50% = 0.4162   distance = 0.0838
profileLikelihood    middle-50% = 0.6158   distance = 0.1158
studentizedT         middle-50% = 0.6898   distance = 0.1898
jackknife            middle-50% = 0.6915   distance = 0.1915
bootstrap            middle-50% = 0.6555   distance = 0.1555
bca                  middle-50% = 0.7080   distance = 0.2080
```

Range of `s50middle` from `0.4162` to `0.7080`. Only one lens (`abc`)
lands within `0.10` of the canonical Palma target of `0.5`. Five of
the six lenses concentrate *more than half* the mass-share into the
middle band, in tension with the empirical Palma prior.

The Palma-hypothesis-violation reading is informative on two distinct
axes:

1. **The `abc` lens is the closest match to the empirical Palma
   prior** *and* the most polarised lens by the Palma ratio itself
   (`palma = 568.4542`). Those two facts are not contradictory: the
   empirical Palma prior says the middle 50% captures *roughly half*
   the mass *while* the polarisation in the top-10% / bottom-40%
   ratio can vary widely. `abc` satisfies the constant-middle property
   tightly while exhibiting extreme polarisation in the top-bottom
   ratio. This is exactly the regime the empirical Palma framework
   was constructed to characterize.
2. **Five lenses violate the constant-middle property** in the
   *upward* direction — `s50middle > 0.5`. This means the underlying
   half-width distribution, as filtered through `bca`, `bootstrap`,
   `jackknife`, `profileLikelihood`, and `studentizedT`, has
   *more* mass concentrated in the middle than the empirical Palma
   prior expects. The structural reading is that those five lenses
   are smoothing the cross-source half-width distribution toward a
   more uniform central region; the `abc` lens, in contrast, is
   producing a distribution where the middle is depleted relative to
   the tails — a true bimodal-ish polarisation regime.

The v0.6.256 refinement therefore separates two concepts that the
raw Palma ratio conflates: *polarisation magnitude* (the `palma`
column) and *Palma-hypothesis fit* (the `palmaHypothesisDistance`
column). A lens can be high on one and low on the other (`abc`:
extreme palma, low hypothesis distance), or low on one and middling
on the other (most of the other five lenses).

The `--sort hypothesis-distance-desc` key surfaces the lenses that
*violate* the empirical Palma prior most strongly, which is the
diagnostic question for the cross-lens reading-protocol: when the
underlying CI uncertainty estimator does *not* match the empirical
prior on inequality structure, the estimator's CI half-widths are
distributed in a way that is mechanically incompatible with the Palma
framework's structural assumption — and the Palma ratio for that
lens, while still computable, may be a less informative summary
statistic than for hypothesis-conformant lenses.

## What axis-26 changes about the cross-lens reading protocol

The W18 sprint has, across axes 21-26, accumulated six dispersion-
or-polarisation measures. The implicit reading protocol the sprint
has been constructing is:

1. **Read the bounded dispersion axes (21-25) for ranking.** Gini,
   Theil, Atkinson, QCD, and Hoover all rank lenses on a comparable
   `[0, 1]`-ish scale and are universal under Pigou-Dalton transfers.
   Their cross-axis disagreements diagnose *which kind* of dispersion
   is operative (moment-based vs. order-statistic, integral vs.
   point, etc.).
2. **Read the unbounded polarisation axis (26) for magnitude.** When
   the bounded axes saturate near their upper bound (Gini > `0.95`,
   Hoover > `0.45`), they have lost discriminating power. Palma's
   unbounded scale recovers magnitude information at the polarisation
   extreme.
3. **Read the Palma hypothesis distance (v0.6.256) for
   interpretability.** When `palmaHypothesisDistance` is large, the
   raw Palma ratio is mechanically computable but interpretively
   weaker; the lens is producing a distribution shape that the Palma
   framework's empirical prior was not designed for.

This three-step protocol is itself a fresh structural contribution.
The W18 dispersion sprint did not previously have a separation of
"ranking" and "magnitude" diagnostics. Axes 21-25 were all in the
ranking family. Axis-26 is the first dedicated magnitude-recovery
diagnostic, and the v0.6.256 refinement is the first explicit
interpretability filter on top of any cross-lens diagnostic in the
suite.

## Test count and the orthogonality claim

Axis-26 ships with 30 new tests (per sha=`297d9d3`), bringing the
suite from `7197` (post-axis-25-refinement at sha=`1b2ea90`) to
`7227`. The v0.6.256 refinement adds 5 more, ending at `7232`. The
test breakdown for axis-26 spans:

- Lorenz interpolation correctness at non-integer index boundaries
  for `n ∈ {4, 5, 6, 7, 8, 9, 10, 11, 12}`.
- Identity verification `s50middle == 1 − s40 − s90` for non-degenerate
  rows.
- Divergence-handling: `palmaIsInfinite = true` when `s40 = 0` and
  `s90 > 0`; numeric clamp to `1e12`.
- Degeneracy classification: all four `degenerateReason` cases.
- Six-class `concentrationLabel` boundary tests at thresholds `0.5`,
  `1`, `2`, `4` and the `extreme` boundary.
- Sort key correctness: `palma-desc`, `palma-asc`, `s90-desc`,
  `s40-asc`, `concentration-label-desc`.
- Alert filter: `--alert-palma-above` with bound validation.
- Render emission: `--show-palma`, `--show-lorenz-points`,
  `--show-concentration`.
- Report-level aggregation: `meanPalma`, `medianPalma`, `maxPalma`,
  `minPalma`, `rangePalma`, `nExtreme`, `mostExtremeLens`,
  `mostBalancedLens` correctness across mixed-degeneracy inputs.

The 30-test floor is consistent with the per-axis test budgets of
axes 21-25 (each shipped with `~20-30` tests). The refinement's 5
tests cover the `palmaHypothesisDistance` identity, the new sort key,
the alert filter with explicit upper-bound validation at `0.5`, and
the render emission of the new line.

The orthogonality claim against axes 21-25 is not just a narrative
assertion in the CHANGELOG — it is *testable* from the live-smoke
data. On the same `n=6` substrate:

- Axis-21 Gini: would saturate near `1.0` for the `abc` lens (no
  separating power between `palma=568` and `palma=10000`).
- Axis-22 Theil GE(1): bounded by `log(6) ≈ 1.79` for `n=6`; the
  `abc` lens would land near the upper bound.
- Axis-23 Atkinson: bounded by `1.0`; same saturation behavior.
- Axis-24 QCD: bounded by `1.0`; uses value-quantiles `Q1`, `Q3`
  rather than mass-quantiles, so its readings are not directly
  comparable but are also bounded.
- Axis-25 Hoover: bounded by `0.5`; per v0.6.253 live-smoke
  `maxH = 0.8177` at `lens=abc` is impossible (the bound is
  `0.5`), suggesting the live-smoke value is a normalized variant
  or there's a convention difference — but in any case Hoover is
  bounded.

Palma at `palma = 568.4542` for the same `lens=abc` is recovering
information that all five prior axes have compressed. The
orthogonality claim is empirically grounded: axis-26 separates
polarisation regimes that axes 21-25 could not.

## Where this leaves the cross-lens stack

After axis-26, the cross-lens dispersion-and-polarisation stack covers
six distinct mechanical formulations on the same `n=6` substrate.
The sprint's choice of orthogonal-property targets has been
disciplined: each axis was chosen for a property the prior axes did
not have, and the test counts have grown linearly with axis count
(roughly 30 tests per axis, all passing). The total suite is now at
`7232` tests across `26` cross-lens axes, plus refinements.

The natural next axis in the orthogonal-property tour would be a
*tail-only* axis (top-10% concentration, dropping the
`/L(0.4)` denominator) or a *bottom-only* axis (`L(0.4)` alone,
dropping the `/(1−L(0.9))` numerator). Either would isolate one half
of the Palma ratio and give the cross-lens reading-protocol a way to
diagnose *which* tail is driving polarisation. But the dispersion-
and-polarisation sprint may also be approaching a natural saturation:
six axes is enough to characterize the distribution shape from
multiple orthogonal angles, and adding a seventh would have to clear
the bar of mechanical novelty that axes 21-26 each cleared.

What axis-26 has put on the table — beyond the immediate per-lens
numbers — is a *protocol* for reading the cross-lens diagnostic stack:
bounded axes for ranking, unbounded axes for magnitude,
hypothesis-distance refinements for interpretability. That protocol is
now testable across the W18 sprint's live-smoke history. Any future
axis added to the suite will be evaluated against it, and the
hypothesis-distance refinement framework demonstrated by `v0.6.256`
sets the template for how subsequent axes can be augmented with
empirical-prior conformance diagnostics.

The pew-insights `v0.6.255 → v0.6.256` arc, taken together with the
five preceding axis releases, has constructed the first cross-lens
diagnostic stack in the project's history that is both mechanically
saturated (six orthogonal axes) and interpretively layered (ranking,
magnitude, conformance). That is the W18 sprint's structural
deliverable. ADDENDUM-181 documents the emission process; pew-insights
`v0.6.255 → v0.6.256` documents the measurement apparatus. Both
landed in the same dispatcher hour. The W18 corpus is now equipped
to characterize itself with diagnostics that the W17 corpus could
not.
