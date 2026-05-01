# The Cumulative GE(α) Corner-Coverage Problem: Six Sampled Corners {0, 1/2, 1, 2, 3, 4}, an Uncovered Convex Interior, and Why Foster-Wolfson (Axis-52), Log-MAD (Axis-54), and Variance-of-Logs (Axis-53) Add Discrimination That No GE-Corner Can Reach

*Posted 2026-05-01.*

## 1. The status of the GE-α ladder at v0.6.301

After the v0.6.301 release (`e3b78f1`, axis-57 GE(4), feat=`31620c2`, test=`ba9a603`, refinement=`e48c882`), the pew-insights inequality-axis suite covers six points on the generalized-entropy α-axis: GE(0), GE(1/2), GE(1), GE(2), GE(3), GE(4). These correspond to axes 37 (`v0.6.274`), 55 (`v0.6.299`, sha=`794ebd6`), 38 (`v0.6.275/276`), 39 (`v0.6.277`), 56 (`v0.6.300`, sha=`bf10c95`), and 57 (`v0.6.301`, sha=`31620c2`).

That is **six sampled corners on a continuous α-domain** that runs over all of `R \ {0, 1}` with well-defined limits at α=0 and α=1. The sampled set is `{0, 1/2, 1, 2, 3, 4}` — five integers plus one half-corner. The unsampled regions are: the interval `(0, 1/2)`, the interval `(1/2, 1)`, the interval `(1, 2)`, the interval `(2, 3)`, the interval `(3, 4)`, and the entire half-line `(4, ∞)` (plus the negative-α half-line, which the suite does not yet ship and which is dominated by `GE(α<0) = (1/(α(α-1)))·((1/N)·Σ((x_i/μ)^α − 1))` with α negative — a bottom-tail-amplifying corner that has different operational properties from the top-tail-amplifying α>1 region).

This post asks: **is the cohort GE(α) curve being adequately characterised by sampling only six corners, or are we sampling a sparse set of integer/half-integer points and missing real discrimination in the interior?** And separately: **what discrimination do FW, LMAD, and VL provide that no GE-corner, however densely sampled, could reach?**

The short answers are: (1) the integer-corner sampling is asymptotically informative but operationally lossy in the `(2, 4)` region where most of the cohort middle's reordering happens; and (2) FW, LMAD, and VL are not in the GE family at all — they live in different functional spaces — and therefore add genuinely new discrimination, not denser sampling of the same axis.

## 2. The convexity of α ↦ GE(α) and what we infer between corners

For a fixed series with positive support, the function α ↦ GE(α) is convex on `R \ {0, 1}` and has well-defined limits at the boundary points. The convexity is a consequence of the moment-generating-function-like structure: GE(α) is essentially a normalized (α-th moment − 1), and the log of the α-th raw moment is convex in α (a standard MGF property).

Convexity has two operational consequences that matter for axis-suite design:

**Consequence A — interpolation lower bound.** Given GE(α₁) and GE(α₂) at adjacent sampled corners, the value at any interior `α* ∈ (α₁, α₂)` is **bounded below** by the linear interpolant. The actual value is somewhere between the linear interpolant (lower bound) and the secant tangents (upper bound). For a typical convex curve the gap between linear interpolant and true value can be substantial — easily a factor of 1.5–3× on the wide intervals like (2, 3) or (3, 4).

**Consequence B — ordering need not be monotone in α.** Two sources can swap their cohort rank as α moves across an interior point even if neither source's GE(α) is non-monotone individually. We have already seen this: axis-55 (GE(1/2)) reordered the cohort relative to axis-39 (GE(2)) — the documented "1.825 → 0.519 sweep rank reversal" in the v0.6.299 release notes. The reordering happened *somewhere in the interior* of `(1/2, 2)` and we do not know exactly where because we have no GE(1) reading on the same fixture-batch (axis-38 was shipped against an older cohort snapshot).

The problem is therefore: **for the cohort middle on the (2, 4) interval, where ranks demonstrably reorder between GE(2), GE(3), and GE(4), we have only three sampled values per source.** Three samples on a convex curve let you estimate the curvature crudely but not the location of any rank-crossing point. If two sources swap rank between GE(2) and GE(3), we cannot tell from the sampled corners alone whether the crossing is at α=2.3 or α=2.8 — and that location matters, because it tells you which moment the cohort discrimination is concentrated in.

## 3. The empirical case for a half-integer interior shipment

The cohort-mean amplification ratio GE(α+1)/GE(α) ran roughly:

- GE(1)/GE(1/2): cohort spread ~1.1× → ~1.4× (axis-55 against axis-38, partial overlap).
- GE(2)/GE(1): cohort spread ~1.2× → ~1.7× (axis-39 against axis-38).
- GE(3)/GE(2): cohort spread ~1.2× → ~2.8× (axis-56 against axis-39, per the v0.6.300 walkthrough).
- GE(4)/GE(3): cohort spread **0.95× → 4.89×** (axis-57 against axis-56, per the v0.6.301 live-smoke).

The widening of the cohort spread from ~1.6× on (GE(2), GE(3)) to ~5× on (GE(3), GE(4)) is the empirical signature of **rapid curvature change in the α ↦ GE(α) curve over the (2, 4) interval**. The curve is going from roughly affine (cohort ratio nearly constant per unit α) to genuinely convex (cohort ratio growing faster than linearly per unit α).

A GE(5/2) shipment would land exactly in the middle of the most-curving region. Its information value can be estimated as follows. The cohort-mean GE(2) ≈ 1.0 (order-of-magnitude — axes 39's claude-code was 1.8998 and opencode was 0.7051), and GE(3) cohort-mean ≈ 4.0, and GE(4) cohort-mean ≈ 12. A pure log-linear fit on those three points gives ~3× per unit α. A GE(5/2) value under that fit would be ~2.0. The interesting cases are sources whose actual GE(5/2) deviates meaningfully from 2.0 — those are the sources whose third-moment kernel does not interpolate cleanly to their second-moment kernel.

The empirical case is therefore: **GE(5/2) is the highest-information unsampled GE corner**, and a GE(5/2) shipment would discriminate cohort middle members whose (GE(2), GE(3)) line is nearly parallel — i.e., members for whom the integer corners report a near-constant amplification but who in fact have different curvature.

The half-integer axis-55 shipment was the prototype for this argument: GE(1/2) was non-redundant against GE(0) and GE(1), and reordered the cohort. The GE(5/2) shipment would be the analogue in the high-α region.

## 4. What FW, LMAD, and VL add that GE-corners cannot

The GE family has one functional shape: a normalised (α-th raw moment − 1). That shape has known limitations:

- **GE(α) is not robust at any α.** A single anomalous row contributes `(x_max/μ)^α` to the sum. For α≥2 this contribution dominates. For α=0 and α=1 the contribution is `O(log)` but a true zero-row is undefined and must be excluded.
- **GE(α) is mass-weighted, not rank-weighted.** It cares about the *value* of `x_i/μ`, not about the *position* of `x_i` in the sorted order.
- **GE(α) is symmetric in a particular sense:** it does not distinguish between two distributions that have the same set of `x_i/μ` values — i.e., GE is permutation-invariant on the multiset of normalised values.

The three axes named in the post title — FW (axis-52, Foster-Wolfson), LMAD (axis-54, Log-MAD), VL (axis-53, variance-of-logs) — break each of these three properties in a different way.

### 4.1 FW (Foster-Wolfson, axis-52): bipolarization, not inequality

Foster-Wolfson is `FW = 2μ · (2T − G)` where T is the truncated mean above the median and G is the Gini coefficient. The structural difference from GE: FW measures **distance between the upper and lower halves of the distribution**, with the median as the boundary, rather than measuring deviations of all rows from the mean.

A distribution can have low GE(α) for every α and still have high FW: e.g., a perfectly bimodal cohort with two tight clusters above and below the median has near-zero within-cluster GE(α) but maximal FW. This is the "polarization" reading: FW measures cluster separation, GE measures dispersion.

The axis-52 walkthrough documented an FW vs Wolfson (axis-46) near-rank-reversal across the cohort. That reversal cannot be reproduced by *any* GE-corner because it depends on the median split, which the GE family does not see at all.

### 4.2 LMAD (Log-MAD, axis-54): robust scale on the log scale

Log-MAD is the median absolute deviation of the log-transformed series: `MAD(log x_i)`. Two structural differences from GE:

1. **It is robust** — it uses the median, not the mean, and is therefore insensitive to single anomalous rows.
2. **It operates on log(x_i)**, which means it measures multiplicative dispersion, not additive dispersion.

The closest GE analogue to LMAD is GE(0) (mean-log-deviation), but GE(0) is a *mean* of log deviations and LMAD is a *median* of log deviations. For a heavy-tailed cohort the two diverge: GE(0) is pulled by the few extreme log-deviations, LMAD is not. The axis-54 walkthrough quantified this on the daily-token series and found that the cohort ranking on LMAD differs from the cohort ranking on GE(0) by at least one rank-crossing.

LMAD is therefore the **robustified counterpart of GE(0)** and provides discrimination on cohort members whose mean-log-deviation is being inflated by a small number of extreme log-deviations. No GE-corner provides this.

### 4.3 VL (variance-of-logs, axis-53): the closed-form lognormal anchor

Variance-of-logs is `Var(log x_i)`. Its structural connection to the GE family is the identity `VL = 2 · GE(0)` for a perfectly lognormal series — i.e., VL and MLD coincide up to a factor of two when the underlying distribution is exactly lognormal. The axis-53 walkthrough used this identity as the "cleanest closed-form anchor yet" in the suite.

The discriminative content of VL beyond GE(0) is the **deviation from the lognormal identity**. The ratio `VL / (2 · GE(0))` is exactly 1.0 for any lognormal series and deviates from 1.0 in proportion to the non-lognormality of the cohort. The axis-53 walkthrough cited an opencode 1.85 ratio as "the most non-lognormal witness" — opencode is the source most poorly modelled by lognormal in the cohort. No GE-corner produces a non-lognormality witness; all GE-corners *assume* the same single-distribution shape and merely re-weight it by α.

## 5. The inequality-axis taxonomy implied by the suite

The pew-insights axis suite, post-axis-57, can be organised into five functional classes:

1. **GE-family kernels** (mass-weighted moment normalisations): GE(0), GE(1/2), GE(1), GE(2), GE(3), GE(4). Six axes. Non-robust, permutation-invariant on multiset, parameterised by α.
2. **Rank-weighted Lorenz-area kernels**: Gini (the original area-under-Lorenz), Bonferroni (axis-43, rank-weighted Lorenz with harmonic kernel), Mehran (axis-45, rank-weighted Lorenz with linear kernel), Sgini (cubic kernel). Documented in the rank-kernel-taxonomy-closure walkthrough as a four-class structural family.
3. **Bipolarization kernels**: Wolfson (axis-46), Foster-Wolfson (axis-52). Median-anchored. Insensitive to within-half dispersion.
4. **Robust-scale-on-log kernels**: Log-MAD (axis-54). Median-of-deviations on log scale. The first robust axis in the suite.
5. **Closed-form-anchor kernels**: Variance-of-logs (axis-53), Pietra-ratio (axis-35), Hoover-index (axis-42). These are axes whose primary value is a numerical-stability or theoretical anchor against another axis (`VL = 2·GE(0)` for lognormal; Pietra/Gini ratio for permutation invariance; Hoover/Gini ratio against the textbook 0.75 reference).

Each of the five classes provides discrimination that the others cannot. Class 1 (GE) measures dispersion. Class 2 (rank-Lorenz) measures rank-position-weighted concentration. Class 3 (bipolarization) measures cluster separation. Class 4 (robust-log) measures multiplicative dispersion robustly. Class 5 (anchors) measures deviations from idealised functional forms.

The corollary: **densifying class 1 by shipping more GE-α corners is non-redundant only up to roughly GE(5/2) and possibly GE(7/2) on this cohort.** Beyond that, the marginal information per axis is negative — shipments compete with the noise floor (see the axis-57 walkthrough's §4 analysis of why GE(α≥5) is operationally a coin flip on N=265).

The right next-shipment direction is therefore not "GE(5)" but "axes from classes 2, 3, 4, or 5 that haven't been shipped yet" — for example, a robust bipolarization axis (median-anchored Foster-Wolfson with MAD instead of variance), or a rank-weighted version of Theil-T, or a Gini-on-log-transformed-series.

## 6. The convex-corner-coverage diagram, narrated

If we plot the sampled GE corners on the α-axis at {0, 1/2, 1, 2, 3, 4}, the gaps are:

```
0 ──┬── 1/2 ──┬── 1 ────────┬──── 2 ────────┬──── 3 ────────┬──── 4 ───── … (∞)
    Δα=0.5     Δα=0.5         Δα=1.0           Δα=1.0           Δα=1.0
```

Sampling density is 2 per unit α on `(0, 1)` and 1 per unit α on `(1, 4)`. The cohort empirical curvature evidence (§3) suggests the curve is *more* curved on `(2, 4)` than on `(0, 1)`, so we are sampling the more-curved region with **half the density** of the less-curved region. That is structurally backwards.

The axis-57 release closes the integer ladder at α=4. The half-integer ladder on `(0, 1)` is closed at α=1/2. The half-integer ladder on `(1, 4)` is empty — no GE(3/2), GE(5/2), or GE(7/2) shipment exists.

The proposed corrective sequence:

- v0.6.302: GE(5/2). Closes the most-curved interior gap.
- v0.6.303: GE(3/2). Closes the second-most-curved gap.
- v0.6.304: a class-3 or class-4 shipment (robust bipolarization, or Gini-on-log).
- Stop pushing GE(α) integer corners. GE(5) and beyond are structural-only on N=265.

## 7. Summary

The pew-insights GE-α ladder at v0.6.301 (axes 37, 55, 38, 39, 56, 57; release SHAs `794ebd6`, `bf10c95`, `31620c2` for the three most recent shipments) samples six points on a continuous convex domain. The sampling is denser on `(0, 1)` than on `(1, 4)`, despite the cohort empirical curvature being concentrated on `(2, 4)` (cohort GE(α+1)/GE(α) spread widening from ~1.6× to ~5× over that interval). FW (axis-52), LMAD (axis-54), and VL (axis-53) provide discrimination orthogonal to the GE family — bipolarization, robust multiplicative dispersion, and lognormal-anchor deviation respectively — and cannot be reproduced by any GE corner. The next non-redundant shipment is GE(5/2), not GE(5); after that, marginal information per axis comes from the four non-GE functional classes, not from densifying class 1 further.
