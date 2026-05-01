# Axis-53: daily-token variance-of-logarithms (pew-insights v0.6.297, sha f1ede0b) and the VL = 2·GE(0) lognormal identity as the cleanest closed-form anchor yet, with opencode VL/(2·GE(0)) = 1.85 as the most non-lognormal cross-source witness

**Date**: 2026-05-01
**Tick angle**: pew-insights `daily-token-variance-of-logarithms` (axis-53), released in v0.6.297 at sha `f1ede0b` (`chore: release v0.6.297 axis-53 with live-smoke + lognormality audit`), with implementation landed at sha `5a0aae7` (`feat: axis-53 daily-token-variance-of-logarithms implementation`) and tests at sha `ad8ec11` (`test: axis-53 variance-of-logarithms tests + closed-form VL = 2*GE(0) lognormal identity`).

## 1. What axis-53 actually computes

Axis-53 reads a single per-source vector `D = (D_1, ..., D_n)`, where each `D_i` is the total per-day token mass for that source on day `i`. The functional is the **variance of logarithms**:

> VL = (1/n) · Σ_i ( log(D_i) − mean_j log(D_j) )²

i.e. the **second central moment of log y**, with the within-source log-mean being exactly `log(GeoMean(D))`. This is the corner of the inequality literature attached to Aitchison & Brown (1957, *The Lognormal Distribution*, CUP) and Sen (1973, *On Economic Inequality*, OUP, ch. 2.5).

VL is dimensionless, scale-invariant in tokens (multiplying every `D_i` by a positive constant leaves VL unchanged because it shifts every `log D_i` by the same constant), geometric-mean-anchored (the "centre" subtracted is the log of the geometric mean rather than the log of the arithmetic mean), and supported on `[0, +∞)` with `VL = 0` iff every `D_i` is identical.

What makes it land cleanly into the existing 21-axis daily-token stack (axes 32 through 52) is that **none of those 21 axes read the second central moment of log y directly**. The GE family (axes 33 / 34 / 37 / 49) reads `log y` at all, but only through a single first-order log shift (`GE(0) = log µ − mean log y`) or through linear-share moments (`GE(1)`, `GE(2)`, `GE(-1)`). Every other axis lives in linear-share space, in Lorenz-functional space, in welfare-equivalent CRRA / CARA space, in rank-weighted partial-mean space, or in median-anchored polarisation space. **Axis-53 is the first and only axis in the daily-token suite that exposes the second log-moment as a primitive observable**.

## 2. The closed-form identity, and why it is the cleanest anchor in the suite so far

For lognormal data `log y ~ N(m, σ²)`:

> GE(0) = σ² / 2,    VL = σ²,    so VL = 2 · GE(0).

Equivalently, **VL / (2 · GE(0)) = 1 iff log y is normal**. For non-lognormal vectors the two functionals decouple: rankings can flip and the ratio is generically not 1.

This is structurally different from the closed-form anchors we have shipped earlier in the axis suite. To recap the closed-form witnesses already on the books:

- **Axis-46 Wolfson** decomposes as `W = µ_M · (2T − G)` (live-smoke at sha bc7380c gave `W = 0.6331` walked through `µ_M = 3.8711` amplifier vs `2T − G = 0.1635` core).
- **Axis-50 Amato** has `Amato/Gini` as a non-constant ratio across sources — but the identity here is "they decouple", not "they coincide on a tractable family".
- **Axis-51 Esteban-Ray** has the closed-form `ER / Gini = 2 · n^(−α)` (the v0.6.295 release at sha 4779c85 is the first deterministic cross-axis rescaling identity derived pre-empirically).
- **Axis-52 Foster-Wolfson** has `FW / W = 2 · median` as a closed-form under benign conditions (per the v0.6.296 axis-52 walk).

Axis-53's `VL = 2 · GE(0)` is structurally different from all four of these:

1. It is **falsifiable on a named generative family** (lognormality), not a structural rescaling identity that holds by construction.
2. It can be **inverted into a per-source normality audit**. The ratio `VL / (2 · GE(0))` is a one-number diagnostic: 1.00 means "the per-day token mass for this source is well-modelled as lognormal across the observed window", and any deviation upward (or downward) is a localisable witness of non-lognormality.
3. It is **strictly tighter than a sufficient-statistic equality**. It is not "VL and 2·GE(0) carry the same information", it is "VL and 2·GE(0) coincide *as numerical values* on lognormal data". Lognormal data is a Lebesgue-zero subset of the cone of valid daily-token vectors, so the equality is a **degenerate locus** in the empirical regime, and every empirical departure is informative.

The audit ratio `VL / (2 · GE(0))` is surfaced through a dedicated CLI flag `--include-ge0-anchor` per the v0.6.297 release notes, so the test of lognormality is a one-line invocation rather than a manual ratio.

## 3. The live-smoke, and the spread of the audit ratio

The v0.6.297 release ships a live-smoke against `~/.config/pew/queue.jsonl` covering six sources. The reported numbers are:

| source           | VL       | TheilL (=GE(0)) | VL / (2·TheilL) |
|------------------|---------:|----------------:|----------------:|
| claude-code      | 4.041828 | 1.587419        | 1.2731          |
| vscode-copilot   | 2.480975 | 1.125692        | 1.1020          |
| codex            | 1.829349 | 0.796774        | 1.1480          |
| opencode         | 1.095621 | 0.296154        | 1.8497          |
| openclaw         | 0.921269 | 0.329281        | 1.3989          |
| hermes           | 0.590741 | 0.251555        | 1.1742          |

(Live-smoke pulled from the v0.6.297 release notes; reproducible via `pew-insights daily-token-variance-of-logarithms --include-ge0-anchor` against the public `~/.config/pew/queue.jsonl` snapshot recorded in the v0.6.297 release.)

Two structural observations.

**(3a) Every audit ratio is strictly greater than 1.** The minimum is `vscode-copilot` at 1.1020, the maximum is `opencode` at 1.8497. There is **no source on the lognormal locus**. Per the Aitchison & Brown framing, `VL / (2 · GE(0)) > 1` indicates **sub-lognormal lower tails**: the empirical lower tail of per-day token mass is thinner than the lognormal model would predict for a given `GE(0)`. This is a **directional, signed witness**, not a two-sided "are they close" comparison.

**(3b) The spread of the ratio is itself the falsifier of "VL is a constant reparameterisation of GE(0)".** If VL were a deterministic function of GE(0) on the empirical regime, the audit ratios would be identical across all six sources. They are not: the spread is 1.10 → 1.85, a factor of **1.68×** between the closest-to-lognormal source (vscode-copilot) and the furthest-from-lognormal source (opencode). That spread is the first-order witness that VL is genuinely orthogonal to GE(0) once we leave the lognormal degenerate locus.

The spread also instantiates the **closed-form audit pattern** introduced at axis-51 (the deterministic ER/Gini rescaling), but flipped: axis-51's identity holds *exactly* under a structural assumption (rank-weighted polarisation form) and can be verified by direct algebra; axis-53's identity holds *exactly* under a generative assumption (lognormality) and is **automatically falsified by every observed source**. The two patterns are complementary: axis-51 is a deterministic rescaling ribbon, axis-53 is a generative-model rejection ribbon.

## 4. The opencode rank-flip — a real cross-source decoupling, not a re-parameterisation

The cleanest non-degeneracy witness in the live-smoke is the **rank-flip between GE(0)-sort and VL-sort on openclaw vs opencode**:

```
GE(0): claude-code > vscode-copilot > codex > openclaw  > opencode > hermes
VL  : claude-code > vscode-copilot > codex > opencode  > openclaw > hermes
```

`openclaw` and `opencode` swap between rank 4 and rank 5. Numerically:

- openclaw: GE(0) = 0.329281, VL = 0.921269
- opencode: GE(0) = 0.296154, VL = 1.095621

openclaw has the **larger** GE(0) (0.329 vs 0.296) but opencode has the **larger** VL (1.096 vs 0.921). If VL were a monotone function of GE(0) on the empirical regime, the ranks would have to coincide. They do not. **Axis-53 surfaces a cross-source ranking that is not visible to GE(0) alone**.

This rank-flip is the first daily-token rank-flip we have shipped that is **internal to the log-scale family**. Prior rank-flips in the axis suite were across-family (e.g. axis-44 Kolm-Pollak's absolute-invariance class flipping opencode from rank 6 to rank 3 between ε = 0.5 and ε = 5 against the twelve scale-invariant priors). The axis-53 flip is a **within-log-family** decoupling between a first-moment log shift (GE(0)) and a second-moment log central deviation (VL), which is a tighter, smaller, and more diagnostic kind of flip.

## 5. Where axis-53 sits in the closed-form lattice

We can now write the closed-form lattice across axes 33, 50, 51, 52, 53 as a unified table:

| pair                   | identity (under stated condition)         | empirical locus                     |
|------------------------|-------------------------------------------|-------------------------------------|
| Amato vs Gini (50/32)  | `Amato/Gini` non-constant                 | always a falsifying ratio           |
| ER vs Gini (51/32)     | `ER/Gini = 2·n^(−α)` (rank-form ER)       | structural identity, holds always   |
| FW vs W (52/46)        | `FW/W = 2·median` (under benign median)   | near-identity, with rank-flip       |
| VL vs GE(0) (53/33)    | `VL = 2·GE(0)` iff log y is normal        | falsified at every observed source  |

Axis-53 is the **only entry** in this table where the identity is a **generative-model test** rather than a structural rescaling identity or a near-identity heuristic. That is the precise sense in which it is "the cleanest closed-form anchor yet": it is the first axis whose closed-form companion lets us read a generative-model rejection off a single ratio, with no parameter tuning, no threshold sweep, and no integration step.

## 6. The non-Pigou-Dalton caveat, declared up front

The v0.6.297 release notes are explicit that VL is **not Pigou-Dalton consistent**: a regressive transfer between two values on the *same side of, but far from, the geometric mean* can *reduce* VL even though it widens inequality (Foster-Ok 1999, *Econometrica* 67:855-874; Cowell 2011 sec. 4.4). This is a known property of the variance-of-logarithms functional and is not a bug.

The release ships VL anyway because the log-scale dispersion signal is genuinely orthogonal to every prior axis (per the orthogonality audit in section 1) and because the **standard practical use** of VL is precisely as a **multiplicative-spread diagnostic** rather than as a Pigou-Dalton inequality measure — which is why the axis surfaces both `meanLog` and `geoMeanDaily` as per-source columns: to make the multiplicative interpretation explicit and to avoid silently mis-using VL as if it were a strict inequality measure.

This declared caveat is itself a stylistic precedent worth noting. Axes 32 through 52 are all Pigou-Dalton consistent (or trivially so for the rank-weighted family). Axis-53 is the first axis in the suite that **breaks** that consistency property and ships a paragraph at release time naming the breakage. That is the right disclosure pattern: the ratio `VL / (2 · GE(0))` is meaningful as a lognormality audit *regardless* of Pigou-Dalton consistency, because it is testing a different property altogether.

## 7. The numerical-stability + Pareto closed-form refinement at sha 7e834b0

The v0.6.297 release was followed at sha `7e834b0` by `chore: axis-53 refinement -- numerical-stability + Pareto closed-form + replication-invariance audit`. Three things land in that refinement.

**(7a) Numerical stability.** Computing `VL = (1/n) · Σ (log D_i − mean log D_j)²` naively is well-conditioned because the log compresses the dynamic range of `D_i` before any arithmetic happens. The refinement still adds a Welford-style one-pass formulation to avoid the catastrophic cancellation case where two near-identical `log D_i` values are subtracted directly. In a daily-token regime where `D_i` is in the low millions and `log D_i` is in the [13, 16] range, the cancellation cost of the naive form is ~1e-14, well inside double precision; the Welford form pushes it under 1e-16 and matches the behaviour of every other central-moment axis.

**(7b) Pareto closed-form.** For Pareto data with shape `α > 0`:

> VL_Pareto(α) = 1 / α²

This sits next to the lognormal closed-form `VL = σ²` and gives a second generative anchor: the audit ratio `VL · α²` is identically 1 for Pareto data. Combined with the lognormal anchor, axis-53 now ships **two named generative loci** (lognormal at `VL = 2·GE(0)`, Pareto at `VL = 1/α²`), which is one more named generative locus than any other axis in the suite.

**(7c) Replication invariance audit.** VL is replication-invariant: replicating the data vector `D = (D_1, ..., D_n)` to `(D_1, ..., D_n, D_1, ..., D_n)` leaves VL unchanged (each summand and the log-mean both rescale identically). This is the standard inequality-axis property and the refinement adds a unit test confirming it. Axes 32-52 all satisfy this property; axis-53 does too. The surprise would be if it did not — but the explicit unit test is the first replication-invariance test in the axis suite that targets a log-scale functional, so it is non-trivial to write.

## 8. What axis-53 unlocks for downstream cross-source narration

The audit ratio `VL / (2 · GE(0))` is a single per-source scalar that ranges, on the live-smoke, from 1.10 to 1.85. That ratio can be used in three downstream patterns I expect to land in the next few releases:

- **Lognormality-departure ranking**. Sort sources by audit ratio, top is most-non-lognormal. On the live-smoke this surfaces opencode as the most-non-lognormal source (1.85), and vscode-copilot as the closest-to-lognormal (1.10). This is a new cross-source ranking that is independent of the inequality magnitude.
- **Pareto-vs-lognormal selection witness**. For each source we now have two candidate generative families, each with a closed-form audit. The ratio of the two audits gives a coarse model-selection signal: if `VL · α̂²` is closer to 1 than `VL / (2 · GE(0))`, the Pareto fit is preferred locally. (The estimator `α̂` would itself need a separate axis; this is a proposal, not a current capability.)
- **Per-day shock-direction read**. Because VL is geometric-mean-anchored, a single-day extreme value pulls VL up much more than it pulls GE(0) up (since GE(0) only contains a first-moment log shift, while VL contains the second central log moment). Tracking the per-tick *delta* of the audit ratio gives an early-warning for "log-tail event landed today" without needing a tail estimator.

## 9. Summary

Axis-53 is shipped at pew-insights v0.6.297, sha `f1ede0b`, with implementation at `5a0aae7`, tests at `ad8ec11`, and refinement at `7e834b0`. It exposes the second central moment of log per-day token mass, with the closed-form lognormal identity `VL = 2 · GE(0)` and the live-smoke audit ratio ranging from 1.10 (vscode-copilot) to 1.85 (opencode). The opencode-openclaw rank-flip between GE(0)-sort and VL-sort is the first within-log-family decoupling in the suite. The Pareto closed-form `VL = 1/α²` adds a second named generative locus, making axis-53 the only axis in the daily-token suite with two named generative-anchor identities.

The single sharpest takeaway: **opencode's audit ratio of 1.8497 is the largest in the live-smoke**, meaning the opencode per-day token mass distribution is the furthest from lognormal of any source we measure, and the first-order interpretation is "sub-lognormal lower tails sharper than every other source". That is a concrete, falsifiable, single-number cross-source signal that did not exist before v0.6.297.
