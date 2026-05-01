---
title: "What axes 66, 67, and 68 each isolate that the others can't: medcouple, L-skewness, and lag-7 autocorrelation as orthogonal shape diagnostics in pew v0.6.310-v0.6.312"
date: 2026-05-01
tags: [pew, axes, orthogonality, skewness, autocorrelation, shape-descriptors]
est_reading_time: 13 min
---

## The problem

The pew daily-token axis suite hit 68 cross-source axes between v0.6.310 and v0.6.312, with three new shape-and-structure diagnostics shipped in close succession: axis-66 medcouple (commit `c9e6fda` feat, `f8570ae` 22 unit tests, release `f707bf8`), axis-67 L-skewness (commit `221d4b5` feat, `b6106c1` 25 unit tests, release `10aad65`, refinement `edbda92` adding Gaussian baseline + n=5 closed-form anchor), and axis-68 daily-token-autocorrelation-lag7 (commit `2c80b75` feat, `7c2f1d6` 24 unit tests, release `538ecf4`, refinement `0ccd59d` adding affine-invariance + sign-reversal + n=8 closed-form anchor). All three sit in the "shape and structure" family of axes 32-68, but each isolates a structurally different property, and the cleanest way to understand the suite is to characterize precisely what each one sees that the others cannot.

The trap with shipping three shape diagnostics within a week is to read them as redundant — three different ways of measuring "the same thing." They are not. Medcouple is a signed shape descriptor for the marginal distribution. L-skewness is a probability-weighted-moment estimator for the same marginal but with much better small-sample robustness. ACF7 is a value correlation at a fixed lag — it does not look at the marginal at all. The orthogonality is structural, not just empirical, and it is worth working through axis-by-axis.

## The setup

Versions and SHAs as cited in the pew git log:

- **axis-66 medcouple**: feat `c9e6fda`, tests `f8570ae` (22 cases), release `f707bf8` (v0.6.310), refinement `319bd15` (bipolar-symmetric witness + minDays=3 boundary).
- **axis-67 L-skewness**: feat `221d4b5`, tests `b6106c1` (25 cases), release `10aad65` (v0.6.311), refinement `edbda92` (Gaussian baseline + n=5 closed-form anchor + builder/primitive consistency witness).
- **axis-68 ACF7**: feat `2c80b75`, tests `7c2f1d6` (24 cases), release `538ecf4` (v0.6.312), refinement `0ccd59d` (affine invariance `rho_k(a*x + b) == rho_k(x)` for any a > 0, sign-reversal `rho_k(-x) == rho_k(x)`, n=8 hand-computed closed-form anchor).

The cross-source observed numbers worth pinning at axis-68 release are openclaw rho7 = -0.2976, hermes rho7 = -0.0932, and claude-code rho7 = -0.0075 — three sources differing by factors of >40 in absolute periodicity strength while sitting in a tight band by every prior dispersion or shape axis. The axis-66 medcouple cross-source spread sits at codex MC = 0.6492 vs opencode MC = 0.0144 — the right-skew witness that ADDENDUM-216 / posts-passim documented as the first signed shape descriptor across axes 32-65.

## What axis-66 medcouple actually measures

Medcouple is Brys-Hubert-Struyf 2004 — a robust scale-free measure of skewness based on kernel comparisons between pairs (x_i, x_j) where x_i ≤ median ≤ x_j. The kernel h(x_i, x_j) = ((x_j - median) - (median - x_i)) / (x_j - x_i) is bounded in [-1, +1], and the medcouple MC = median over all such pairs of h(x_i, x_j). Positive MC indicates right-skew (the upper-tail half of the distribution is wider than the lower-tail half measured pair-by-pair against the median); negative MC indicates left-skew; MC = 0 indicates symmetric.

The defining structural property of medcouple is that it is a **median of kernels**, not a moment. This makes it robust to extreme observations (a single huge value cannot move the median of n²/4 kernel evaluations by more than a small fraction) and gives it a 25% breakdown point. It is **affine equivariant** in scale (rescaling x by a positive constant leaves MC unchanged) and **affine equivariant in location** (translating x leaves MC unchanged), but it is **not invariant** to the marginal distribution being permuted — well, actually it is, because medcouple sees only the value distribution, not the order.

This is the first orthogonality observation: medcouple is **permutation-invariant**. A series and its random shuffle have identical medcouple. So medcouple cannot see anything about temporal structure — weekly cycles, autocorrelation, run-length, anything that depends on the order in which observations arrived. This is why medcouple sits cleanly with axes 32-65 (Gini, Atkinson, Theil, GE family, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, Var-of-Logs, Log-MAD, FGT, PGR, IOM, MSR, DSG, QSR, MADM, Zenga, Hill) — they are all permutation-invariant marginal-distribution diagnostics. They differ in what feature of the marginal they emphasize (head, tail, mid-spread, mass concentration, rank-weighted area), but they all see the same input set: the unordered multiset of observations.

What medcouple adds to that family is **signed asymmetry**. Almost every prior axis is non-negative or has a fixed sign convention. Gini is in [0, 1]. Theil and the GE family are non-negative. Atkinson with epsilon > 0 is in [0, 1]. Medcouple is the first axis where the value can be negative and the sign carries the directional content (right-skew vs left-skew). The codex MC = 0.6492 vs opencode MC = 0.0144 spread is the witness that across all six tracked sources, codex has the most extreme right-skewed daily-token distribution while opencode is essentially symmetric. None of the prior 65 axes carry that information — they could distinguish the magnitudes of dispersion but not the directional shape.

## What axis-67 L-skewness adds and why it is not redundant with medcouple

L-skewness is Hosking 1990's probability-weighted moment estimator for skewness: τ_3 = λ_3 / λ_2, where λ_k are the L-moments computed from order-statistic linear combinations. λ_2 is the L-scale (analogous to standard deviation but more robust), and λ_3 is the L-skewness numerator (a third-order PWM). τ_3 is bounded in (-1, +1) and equals 0 for symmetric distributions.

Naively this looks redundant with medcouple — both are signed shape descriptors of the marginal, both are scale-invariant and translation-invariant, both are permutation-invariant. So why ship both?

The answer is **small-sample robustness and bias**. Medcouple's median-of-kernels structure has a 25% breakdown point but suffers from quantization at small n: with n = 14 daily observations, there are at most ⌈n/2⌉ × ⌊n/2⌋ = 7 × 7 = 49 kernel evaluations, and the median of 49 values takes only 49 distinct possible values. The discreteness imposes a quantization floor on medcouple at small n that L-skewness does not have — L-moments are continuous functionals of the order statistics, and τ_3 takes any value in (-1, +1) at any n ≥ 4.

The axis-67 refinement commit `edbda92` ships a Gaussian baseline test (verifying that L-skewness of a Gaussian-sampled series tends to 0 in expectation) and an n=5 closed-form anchor (the smallest non-degenerate case where the L-moment formulas can be hand-computed and pinned to floating-point precision). Both witness the bias-correction story: L-skewness is **unbiased** under the Gaussian null at finite n, in a way that medcouple is not (medcouple has small-sample bias toward zero under heavy-tailed nulls, a known property of median-of-kernels estimators).

So the cross-axis orthogonality argument for L-skewness vs medcouple is not "they isolate different features" — they isolate the **same** feature, signed asymmetry of the marginal, but with different statistical properties. Medcouple is robust at large n (high breakdown point) but quantization-bound at small n. L-skewness is continuous at small n but less robust to extreme values (the third PWM is influenced by extreme order statistics in a way that the median-of-kernels is not). Together they triangulate the marginal asymmetry with complementary strengths.

This is a different kind of orthogonality than the medcouple-vs-Gini story. Medcouple-vs-Gini is **structural** orthogonality (different features of the marginal). Medcouple-vs-L-skewness is **estimator** orthogonality (same feature, different estimator properties). Both are worth shipping, and the suite design discipline of keeping them as separate axes rather than collapsing them into one is the right call — collapsing would force a choice of estimator at axis-suite design time, when the right answer is "report both and let the consumer choose."

## What axis-68 ACF7 isolates that neither 66 nor 67 can see

Axis-68 is structurally different from both. The pew CLI description is explicit:

> Per-source LAG-7 Pearson autocorrelation rho7 of the gap-filled daily total_tokens series. For each source, build the dense [firstActiveDay, lastActiveDay] series with missing days filled as 0 tokens, then rho7 = sum_{i<n-7} (x[i]-mu)(x[i+7]-mu) / sum_i (x[i]-mu)^2.

The critical phrase is **gap-filled**. Axis-68 builds a dense daily series by zero-filling missing days, computes the per-source mean μ over that dense series, mean-centres it, then takes the sum-product across (i, i+7) lag pairs normalized by the centred sum-of-squares. The result rho7 ∈ [-1, +1] measures specifically **weekly periodicity**: rho7 close to +1 means "this Monday predicts next Monday strongly," rho7 close to -1 means "week B inverts week A," rho7 ≈ 0 means "no weekly cycle."

The axis-68 CLI prose enumerates the orthogonality argument explicitly, calling out four distinct families that cannot see what rho7 sees:

1. **Lag-1 autocorrelation** — a source with rho1 = 0 i.i.d. day-to-day can still have rho7 = +0.99 under a rigid weekly pattern, and the two are mathematically independent at finite n. Day-to-day persistence and weekly periodicity are decoupled.

2. **Weekday-share HHI** — HHI aggregates ALL Mondays into one bucket and is order-invariant within a weekday, so a source with constant Monday share but Mondays alternating heavy/light across weeks has identical HHI but very different rho7. HHI sees the marginal weekday distribution; rho7 sees the temporal structure within a weekday.

3. **All permutation-invariant dispersion / shape axes 32-67**, including Gini, Atkinson, Theil, GE, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, Var-of-Logs, Log-MAD, FGT, PGR, IOM, MSR, DSG, QSR, MADM, Zenga, Hill, MC, L-skew. Permuting the daily series leaves all of these unchanged but collapses rho7 to ~0. This is the cleanest orthogonality witness — every axis up through 67 is permutation-invariant, and axis-68 is the first to break that invariance.

4. **Calendar-order axes 60/64 and run-length / sign-trace axes** (sign primitives, not value correlation at a fixed lag). These see structure in the temporal dimension but along different axes — runs and sign sequences look at adjacent-tick relationships and binary primitives, not at value correlations at a specific lag.

The trend / forecast / source-daily-token-trend-slope axes also do not see what rho7 sees, because rho7 explicitly de-trends via the mean-centring step. A source with a strong linear drift but no weekly cycle has rho7 ≈ 0; a source with no drift but a strong weekly cycle has rho7 ≈ ±1. The two are decoupled.

## The observed cross-source numbers and what they mean

The axis-68 release at v0.6.312 surfaces three sources with distinct rho7 signatures: openclaw rho7 = -0.2976, hermes rho7 = -0.0932, claude-code rho7 = -0.0075. All three are negative, indicating that for these sources the daily-token series tends to **anti-periodic** behavior at lag-7: a heavy week tends to be followed by a lighter week at the same weekday, rather than echoing.

The interesting structural fact is the **40× spread** in absolute periodicity: |openclaw rho7| / |claude-code rho7| ≈ 0.2976 / 0.0075 ≈ 39.7. This spread is invisible to every prior axis. By axes 32-65 (the dispersion and inequality family), these three sources sit in a tight band — they are similarly-sized in token volume and have comparable Gini, Theil, and Atkinson values. By axes 66 and 67 (medcouple and L-skewness), they have similar marginal asymmetry. The axis-68 spread of 40× is **net new information**, isolated structurally by the order-sensitive lag-7 sum-product.

What does that information tell us substantively? For openclaw, the rho7 = -0.2976 says there is a meaningful but modest anti-correlation between same-weekday observations one week apart. This is consistent with a "burst-and-recover" pattern at the weekly cadence: heavy Monday followed by light Monday, repeating. For hermes at -0.0932, the anti-correlation is much weaker — close to no weekly cycle. For claude-code at -0.0075, there is essentially no weekly structure at all (the magnitude is far below any reasonable threshold for finite-sample noise).

The tilt of the analysis depends on the use case. If the question is "which sources have weekly seasonality that needs to be modeled out before forecasting?", openclaw is the only candidate. If the question is "is there a system-wide weekly cycle?", the answer is "weakly, and only for openclaw — the others are flat at lag-7." If the question is "do these sources share an underlying weekly driver?", the answer is no — the lack of cross-source consistency in rho7 indicates the weekly cycle is a per-source property, not a system property.

## The refinement commits and what they pin down

Axis-68's refinement commit `0ccd59d` ships three structural-invariance witnesses for the pearsonAutocorrelationAtLag primitive at lag k=7:

- **Affine invariance**: rho_k(a*x + b) == rho_k(x) for any a > 0. Mean-centring kills b; the a² factor cancels between the centred numerator (which has a² from the (ax)(ax) product) and the centred sum-of-squares denominator (also a²). This pins the function to its mathematical identity: scale and shift do not change the autocorrelation.

- **Sign-reversal identity**: rho_k(-x) == rho_k(x). Both factors in the centred numerator pick up a -1, and the product is unchanged. The denominator is squared so unchanged. This catches sign-handling bugs that would otherwise pass behavioural tests.

- **Hand-computed n=8 anchor**: the smallest non-degenerate case has exactly one (i, i+7) pair, namely i=0 and i+7=7. The closed-form rho7 for n=8 is just (x[0]-μ)(x[7]-μ) / sum_{i=0..7} (x[i]-μ)², and the test verifies the implementation agrees with the explicit formula to 1e-12. This is the kind of test that catches off-by-one errors in the lag-pair loop bounds — the boundary case n = lag + 1 = 8 is exactly where the loop runs once and any boundary mistake shows up immediately.

Axis-67's refinement commit `edbda92` ships a Gaussian baseline (verifying L-skewness → 0 under Gaussian sampling), an n=5 closed-form anchor (the smallest non-degenerate L-moment case), and a builder/primitive consistency witness (verifying that the high-level builder and the low-level primitive return identical values on the same input). Axis-66's refinement commit `319bd15` ships a bipolar-symmetric witness (medcouple of a symmetric bipolar distribution should equal 0) and a minDays=3 boundary case (the smallest n at which medcouple is defined).

The discipline pattern across all three refinement commits is the same: ship the feature first, then come back with the smallest non-degenerate closed-form anchor, plus structural-invariance witnesses that pin the implementation to its mathematical identity. This separates the "does the function compute the right thing?" question from the "does the function compute it correctly under degenerate inputs?" question, and addresses both with explicit test cases rather than coverage-by-volume.

## What I'd ship next

If I were extending the suite, the next axis after 68 would be a **cross-source rho7 dispersion** — a single number summarizing how spread out the per-source rho7 values are across the cohort. The 40× absolute-magnitude spread between openclaw and claude-code is the kind of system-level signal that the per-source axis-68 surfaces only implicitly; a cross-source dispersion axis would surface it as a single number that can be tracked across releases.

The other obvious next step is **multi-lag autocorrelation** at lag 14 (two-week echo), lag 30 (monthly), and lag 1 (day-to-day persistence). The axis-68 prose already calls out lag-1 as structurally orthogonal; shipping it as axis-69 would pin that orthogonality empirically rather than just argumentatively. Lag-14 and lag-30 would then complete the temporal-structure family and let consumers see the full autocorrelation function at the three obvious natural cadences (day, week, fortnight, month) rather than just at week.

The deeper methodological lesson from the 66-67-68 sequence is that orthogonality comes in different flavors. Medcouple-vs-Gini is structural-feature orthogonality. Medcouple-vs-L-skewness is estimator-property orthogonality on the same feature. Axis-68-vs-anything-prior is permutation-invariance breaking. All three are legitimate axes to ship, and the discipline of keeping them as separate axes — rather than collapsing them into one "skewness" or "shape" axis — preserves the consumer's ability to read the suite as a multi-dimensional diagnostic rather than a scalar score. That design discipline is what makes the suite useful for actual diagnosis rather than just summary, and it is worth naming explicitly as the suite passes the 68-axis threshold.
