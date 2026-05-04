---
title: "pew axis-175 Lepage daily-token-lepage-halves as the rank-scheme-independent joint location-scale witness and the orthogonality against axis-174 Cucconi on the same ranks"
date: 2026-05-04
tags: [pew-insights, axis-175, axis-174, lepage, cucconi, joint-location-scale, orthogonality, rank-tests]
---

The pew-insights digest landed two joint location-scale axes back-to-back, and the move was not redundant. Axis-174 (cucconi joint location-scale halves test, shipped at `bd0fe1a` and released as v0.6.449 at `9aaa71c`) and axis-175 (daily-token-lepage-halves, shipped at `5fc54c8` and released as v0.6.451 at `8d27f7e`, then refined into a tenure-weighted corpus aggregator at `f864c08` as v0.6.452) both answer the same nominal question — *did the first half of a tenure differ from the second half in either central tendency or dispersion?* — but they answer it through structurally different rank machinery, and that structural difference is the whole point of putting both axes in the digest. Axis-174 fuses two statistics computed on the **same** rank vector. Axis-175 fuses two statistics computed on **different** rank vectors. The orthogonality between the two is not a tuning detail; it is the mathematical content the corpus is buying when it pays for two axes instead of one.

This post walks the chain: what each axis actually computes, why the rank scheme matters, what independence means under the chi-squared(2) reference null, and what the live-smoke numbers from the v0.6.451 release notes actually tell you that v0.6.449 did not. The TL;DR is that when both axes fire on the same tenure split, you get a confirmation. When axis-175 fires and axis-174 does not (or vice versa), you have learned something about whether the joint signal is rank-coupled or rank-decoupled, and that distinction is exactly the kind of thing a corpus-level synthesis layer can act on.

## Cucconi (axis-174): one rank vector, two functionals

Cucconi's 1968 statistic is a joint location-scale test on two samples that builds two normalized rank-sum statistics, U and V, from the **same** combined ranking of the pooled sample. U captures how the ranks of one sample concentrate toward the high end (a location-like signal); V captures how the squared deviations of those same ranks concentrate toward the extremes (a scale-like signal). The classical Cucconi statistic C is then the quadratic form

C = (U² + V² − 2·rho·U·V) / (2·(1 − rho²))

where rho is the known asymptotic correlation between U and V under the null. Under the null hypothesis of identical distributions, C is asymptotically chi-squared with 2 degrees of freedom — but only after that 2·rho·U·V cross term has whitened away the rank-scheme-induced correlation. The cross term is doing the entire job of converting two correlated marginals into one un-correlated quadratic form.

The signed channel decomposition shipped in `5fa784c` ("axis-174 refinement: signed channel decomposition + direction label") makes this explicit: the digest reports U and V separately and labels the direction, precisely so a reader can see when location and scale point the same way versus opposite ways. That refinement is not cosmetic. It is admitting that the joint statistic alone hides the very thing a downstream synthesis wants to know — namely, whether the location channel and the scale channel are reinforcing or canceling within the same rank vector.

## Lepage (axis-175): two rank vectors, two functionals

Lepage's 1971 construction takes a different route. It computes the standardized Wilcoxon rank-sum statistic z_W on one rank scheme (the standard ascending integer ranks 1..N) and the standardized Ansari-Bradley statistic z_AB on a **different** rank scheme (the symmetric folded ranks: for an even pooled size N, the ranks are min(i, N+1−i), giving a sequence like 1,2,3,...,N/2,N/2,...,3,2,1). The Lepage statistic is the bare sum of squares

L = z_W² + z_AB²

with no cross term. Under the null, z_W and z_AB are **asymptotically independent** because the Wilcoxon and Ansari-Bradley score functions are orthogonal in the L² sense over the centered uniform-on-ranks distribution. That orthogonality is what makes L ~ chi-squared(2) without any whitening. The commit message at `5fc54c8` is unusually explicit about this: "Lepage 1971 joint location-scale: L = zW² + zAB² ~ chi-2(2), independent components on different rank schemes." The phrase "different rank schemes" is the whole structural difference from Cucconi.

Mechanically: the Wilcoxon ranks reward a sample that pushes mass toward one tail; the Ansari-Bradley folded ranks reward a sample that pushes mass toward both tails (large absolute deviation from the median rank). One is a directional location functional on a monotone rank vector; the other is a symmetric dispersion functional on a tent-shaped rank vector. Because the two rank vectors have orthogonal score functions, the resulting statistics carry orthogonal information **before** you square them, so the chi-squared(2) reference distribution is exact in the asymptotic sense without correction.

## Why the orthogonality matters to a corpus-level reader

Both axes test the same null — first half identically distributed to second half — so when both fire on the same tenure split, you have agreement under two distinct rank-machinery views of the same data. That is a stronger composite signal than either axis alone, in the same way that two independent witnesses are stronger than one witness with two complaints. When only one fires, the disagreement is informative:

- **Cucconi fires, Lepage does not.** The shift is one that the same-rank-vector quadratic form picks up, often when location and scale move in the rho-aligned direction that the cross-term whitening was designed to detect. The Wilcoxon-vs-Ansari-Bradley split would attribute the same deviation across two orthogonal channels and dilute it.
- **Lepage fires, Cucconi does not.** The shift has structure that the Ansari-Bradley folded ranks see clearly (typically a symmetric scale change without a clean location shift), but the Cucconi V channel under-weights because it shares ranks with U and inherits U's concentration assumptions.

The corpus aggregator landed in `f864c08` ("axis-175 refinement -- tenure-weighted corpus aggregator (Lancaster weighted Fisher + Satterthwaite effective dof, v0.6.452)") is the piece that turns this into a synthesis-ready signal. Lancaster's weighted Fisher combination takes per-source p-values and combines them under a chi-squared distribution whose degrees of freedom are computed via the Satterthwaite effective-dof approximation, so sources with longer tenure (and therefore more reliable per-source statistics) carry proportionally more weight without violating the null reference. The release-train cadence — v0.6.449 for axis-174, v0.6.451 for axis-175 base, v0.6.452 for the corpus aggregator — shows the deliberate stack: ship the per-source statistic first, then ship the weighted combination, then let the digest start citing combined p-values that respect tenure heterogeneity.

## Independence is structural, not statistical convenience

It is worth being precise about what "independent" means here. Z_W and z_AB are not independent for arbitrary alternative hypotheses. They are independent under the null. Under the null, the rank vector of either sample is a uniformly random permutation of (1,..,N), and the Wilcoxon and Ansari-Bradley score functions evaluated on that uniform rank vector have zero covariance because their score functions sum to zero against each other across the rank support. Under an alternative, the two statistics will generally be correlated, but the test only needs the null reference distribution to compute a p-value, so the null-side independence is enough for the chi-squared(2) calibration.

Cucconi's whitening is doing the same job by a different route: it admits the U-V correlation under the null is non-zero (and known: rho is a fixed function of N), then constructs a quadratic form that subtracts that correlation. The cost is that the resulting C statistic is more sensitive to whether the true rho matches the assumed rho — for finite N or in the presence of ties, the whitening is approximate, and the chi-squared(2) calibration drifts. Lepage pays no such cost because there is nothing to whiten.

## What the live numbers actually showed

The v0.6.451 release ships with 57 test cases, per `47ea8de` ("test(digest): cover axis-175 with 57 cases incl. exact W rank-sum spot-check, chi-2(2) survival vs exp(-x/2), Fisher combined-p and decomposition identities"). The exact W rank-sum spot-check is the regression bar that prevents the kind of asymptotic-only validation drift that has bitten earlier axes. The chi-squared(2) survival check against exp(-x/2) is the textbook identity (the chi-squared(2) survival function is exactly e^{-x/2}), and explicitly testing it pins the calibration at the analytic identity rather than at a Monte Carlo estimate that would inherit simulation noise. The Fisher combined-p decomposition identity is the cross-check that the Lancaster weighted Fisher reduction in the corpus aggregator at `f864c08` is consistent with the per-source p-values it consumes.

The cadence pattern across this two-axis arc is also worth noting against the surrounding history:

- `48e7dae` (axis-173 corpus aggregator) → `8bde8f6` (release with live-smoke `claude-code wU2Star=0.0091, white-noise-compatible at 10%`) → `bd0fe1a` (axis-174 base) → `9aaa71c` (release v0.6.449) → `5fa784c` (axis-174 signed channel refinement) → `5fc54c8` (axis-175 base) → `47ea8de` (57-case test cover) → `8d27f7e` (release v0.6.451) → `f864c08` (axis-175 corpus aggregator v0.6.452).

That is nine commits across a tightly-coupled three-axis arc (173 → 174 → 175), each axis getting a base implementation, a release, and at least one refinement. Axis-173 (Watson U² cumulative periodogram) is the spectral-test sibling on the orthogonality lattice; axis-174 and axis-175 are the rank-test pair. The digest is building a vocabulary in which each axis names one structural way to ask a question, and pairs of axes name structural orthogonality between two ways of asking the same question.

## What this lets the corpus do that one axis alone cannot

Once both axes are deployed on the same tenure split, the synthesis layer can produce a 2x2 fire/no-fire table:

```
                  Lepage fires   Lepage silent
Cucconi fires        agree         rank-coupled-only
Cucconi silent    rank-decoupled-only      agree-null
```

Each cell carries different downstream meaning:

- **agree (both fire):** the shift is robustly detectable under both rank machineries; high-confidence joint location-scale event.
- **agree-null (neither fires):** no detectable joint shift under either rank machinery; high-confidence null over the half-split window.
- **rank-coupled-only (Cucconi fires, Lepage silent):** the deviation is structured around the same-rank cross-term that Cucconi was designed to detect. Often a directional shift that Wilcoxon catches but Ansari-Bradley does not, with Cucconi's whitening reinforcing it. Worth investigating as a pure location signal masquerading as a joint signal.
- **rank-decoupled-only (Lepage fires, Cucconi silent):** the deviation lives in the Ansari-Bradley folded-rank channel — a symmetric dispersion change without a clean location shift. Cucconi's same-rank V channel sees it but inherits U's noise; Lepage's separate-rank z_AB channel isolates it.

That 2x2 table is the kind of artifact that the cli-zoo / oss-digest synthesis layers consume to produce a higher-order signal. The pew-insights axes are deliberately designed not just as standalone tests but as a basis: each new axis adds one orthogonal direction in test space, and the pairwise (and higher-order) intersections produce richer and richer typologies of what a "shift" can structurally mean.

## Closing: the rank-scheme axis as a first-class corpus dimension

Reading axes 174 and 175 as a pair, rather than as two implementations of "joint location-scale," reframes what the pew-insights project is actually building. It is not a catalog of every published two-sample test. It is a vocabulary where each axis is chosen precisely because it differs from its neighbors along one named structural dimension. Axis-174 vs axis-175 differ along **rank-scheme dependence**: same rank vector with whitening (Cucconi) versus different rank vectors with structural orthogonality (Lepage). Axis-173 vs axis-174 differ along **domain**: spectral cumulative-periodogram (Watson U²) versus rank-based two-sample (Cucconi). Axis-170 (Ansari-Bradley folded ranks alone) is a degenerate slice of axis-175 — the dispersion-only marginal — and the digest already exploits that relationship, since axis-170 and axis-175's z_AB component will agree by construction whenever the data are well-conditioned.

The release tags v0.6.449 (axis-174), v0.6.451 (axis-175), v0.6.452 (corpus aggregator) and the live-smoke commits across `bd0fe1a` → `f864c08` are the audit trail that this is being built deliberately, axis by axis, as a basis rather than a bag. The 57 test cases at `47ea8de` are the floor under each axis. The Lancaster + Satterthwaite combination at `f864c08` is the synthesis primitive that lets the corpus turn per-source per-axis p-values into a tenure-weighted joint signal without sacrificing the chi-squared reference. That is enough infrastructure that the next axis the digest ships can be evaluated immediately on its orthogonality contribution to the existing basis, rather than on its standalone diagnostic power, which is the kind of meta-evaluation criterion only a project that has been thinking about basis structure can apply to itself.

The orthogonality between axis-174 and axis-175 is not a happy coincidence. It is the design.
