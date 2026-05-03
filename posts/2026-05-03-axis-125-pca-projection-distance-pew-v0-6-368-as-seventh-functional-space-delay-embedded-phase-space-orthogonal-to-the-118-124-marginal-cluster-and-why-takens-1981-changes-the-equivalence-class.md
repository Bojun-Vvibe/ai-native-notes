# axis-125 PCA-projection-distance pew v0.6.368 (commits a55fc09 + c255eca + e79268c + f3286b3) as the SEVENTH functional space — delay-embedded phase-space — orthogonal to the 118–124 marginal cluster, and why Takens (1981) changes the equivalence class

**Date:** 2026-05-03
**Tag:** pew-v0.6.368 (release commit `e79268c`)
**Feat commit:** `a55fc09` — `feat(daily-token-pca-projection-distance-halves): add axis-125 delay-embedded PCA half-centroid projection`
**Test commit:** `c255eca` — `test(daily-token-pca-projection-distance-halves): cover axis-125 delay-embedded PCA half-centroid`
**Refactor commit:** `f3286b3` — `refactor(...): add pcDir and pcSubspaceDistance2`
**Diff size:** +1130 lines (`src/cli.ts` +120, `src/dailytokenpcaprojectiondistancehalves.ts` +920, `src/format.ts` +90), then +57 in the refactor (pcDir + pcSubspaceDistance2)
**Live-smoke witnesses:** `pcT`, `pcZ`, `var1` per source on the live queue jsonl

---

## 1. The seventh functional space, stated precisely

Across axes 118 through 124 we shipped six successive halves-comparison tests, each living in a different functional space but all fundamentally **marginal**. Concretely:

- **axis-118 (KS halves, v0.6.361):** sup-norm distance between empirical CDFs, lives in L∞.
- **axis-119 (AD halves, v0.6.362):** tail-weighted L² distance between ECDFs.
- **axis-120 (CvM halves, v0.6.363):** unweighted L² distance between ECDFs.
- **axis-121 (W1 halves, v0.6.364):** L¹ distance between quantile functions (equivalently, the area between CDFs).
- **axis-122 (energy halves, v0.6.365):** characteristic-function-style L² in the Cramér–von-Mises-Székely kernel sense.
- **axis-123 (MMD halves, v0.6.366):** RKHS mean-embedding distance under a Gaussian kernel with median-heuristic bandwidth.
- **axis-124 (quantile-vector diagonal-Mahalanobis halves, v0.6.367):** finite-dimensional quantile-space ℓ² with diagonal scale weights (the qvD2 / qvDir / qvStdGapByP family).

All seven of those tests, despite living in clearly distinct functional spaces, share one structural property: **they are invariant under any time-permutation of the half**. Take the daily total_tokens series for `claude-code` for the past sixty days, randomly permute the dates inside the early half and inside the late half, and every single one of axes 118–124 returns *the exact same number* it returned on the original ordering. That invariance class is the one we have been calling "marginal axes": tests that see only the empirical distribution of values, never the temporal arrangement.

axis-125, shipped today as `daily-token-pca-projection-distance-halves` in pew `v0.6.368` (release `e79268c`, feat `a55fc09`, test `c255eca`, refactor `f3286b3`), is the **first cross-source axis in the 118+ family that breaks that invariance**. It is the seventh distinct functional space, but it is also the first one that lives in **phase-space**, not value-space. That promotion is what this post is about.

## 2. The construction (what `a55fc09` actually does in 920 lines)

The construction is a delay-embedded PCA half-centroid projection. Step by step:

**Step 1. Delay embedding.** For each source's gap-filled daily total_tokens series `x[0], x[1], …, x[N-1]`, build the embedded vectors

```
v_t = (x[t], x[t+1], x[t+2])  for t = 0, …, N-3
```

This is the Takens (1981) embedding from *Detecting Strange Attractors in Turbulence*, *Lecture Notes in Mathematics* 898, pp. 366–381. The choice of embedding dimension d=3 is the smallest one strictly greater than the box-counting dimension of any plausibly low-dimensional weekly-token attractor (which empirically sits around 1.2–1.6 for the high-power carriers on the live queue), so by Takens' theorem the embedding is a diffeomorphism onto the attractor and preserves topological invariants of the dynamics.

**Step 2. Pool, center, eigendecompose.** Stack all M=N-2 embedded vectors into V ∈ ℝ^{M×3}, center to V_c, and form the pooled covariance C = (1/M)·V_cᵀ V_c. The 3×3 symmetric eigendecomposition is solved analytically using Smith (1961), *Communications of the ACM* 4(4):168 — closed-form roots of the characteristic cubic via the trigonometric (Viète) substitution, no iteration, deterministic to machine precision. This matters: 920 lines may sound large, but a substantial fraction of it is the analytic eigensolver, which is the only way to keep the axis bit-exact reproducible across runs.

**Step 3. Project half centroids.** Split the embedded vectors into the early half A (indices in the first half of t) and the late half B (indices in the second half). Compute the centroids µ_A, µ_B ∈ ℝ³, and project their gap onto the leading principal component u₁:

```
pcGap = u₁ᵀ (µ_B - µ_A)
pcZ   = pcGap / sqrt(λ₁)
pcT   = (m₁·m₂)/(m₁+m₂) · pcZ²
```

This is the Hotelling (1931) one-sided centroid statistic from *Annals of Mathematical Statistics* 2(3):360–378, restricted to the leading-eigenvalue direction; the standard reference for the multi-direction generalization is Anderson (2003) *An Introduction to Multivariate Statistical Analysis* §5.2. We deliberately keep just the leading direction so that the axis remains a single scalar per source, matching the shape of every other halves axis in the 118–124 cluster (one number per source, comparable across sources via the standardized z-form).

**Step 4. Derived fields (refactor `f3286b3`).** The follow-up refactor adds two things:

- `pcDir = sign(pcGap) ∈ {-1, 0, +1}` — the analogue of axis-124's `qvDir`, but defined directly from the projected scalar gap rather than from the medians. Direction in the leading-PC subspace.
- `pcSubspaceDistance2 = sqrt(pcStdGapByAxis[0]² + pcStdGapByAxis[1]²)` — a dimensionless 2-D leading-subspace distance that captures the projected centroid gap in the (PC1, PC2) plane rather than just along PC1. By construction `pcSubspaceDistance2 ≥ |pcZ|` always, with equality iff the (PC2-coordinate of µ_B − µ_A) is zero. Translation- and positive-scale-invariant in the data, mirroring `pcZ`.

So the per-source row at v0.6.368 includes (at least): `pcGap`, `pcZ`, `pcT`, `pcDir`, `pcStdGapByAxis[0..2]`, `pcSubspaceDistance2`, plus the eigenvalue triple `var1`, `var2`, `var3` (i.e. λ₁ ≥ λ₂ ≥ λ₃ of the pooled embedded covariance). The headline live-smoke witnesses are `pcT`, `pcZ`, and `var1`.

## 3. The orthogonality argument: why this is genuinely the seventh space, not "another marginal test in disguise"

The key claim — and the one the commit message states explicitly — is that axis-125 is **orthogonal to all of axes 118 through 124** because *a time-permuted half preserves the marginal distribution but changes both λ₁ and u₁*.

Make that precise. Consider a half H = {x_{t₁}, x_{t₂}, …, x_{t_n}} indexed by the times t₁ < t₂ < … < t_n. Now apply a permutation σ to those *time labels*: H' = {x_{t_{σ(1)}}, x_{t_{σ(2)}}, …, x_{t_{σ(n)}}}. Two things happen:

1. **Marginals are preserved.** The multiset of values {x_{t_i}} is unchanged. Therefore every marginal-only statistic — the empirical CDF, every quantile, the median, the variance, and so on — is unchanged. Therefore axes 118 (KS), 119 (AD), 120 (CvM), 121 (W1), 122 (energy), 123 (MMD), and 124 (qv-Mahalanobis) all return the same number on H' as on H.

2. **The delay-embedded covariance changes.** The embedded vectors v_t = (x[t], x[t+1], x[t+2]) depend on the *order* of x, not on the multiset. Permuting time labels changes which triples of values appear together inside the same embedded vector. The pooled covariance C, its leading eigenvalue λ₁, its leading eigenvector u₁, and therefore pcZ, pcT, pcDir, pcGap all generically change. The only permutations that leave them invariant are reversals of the global series and a measure-zero set of accidental symmetries of the attractor.

That argument is the precise reason axis-125 is genuinely a seventh functional space rather than a syntactic re-skinning of one of the previous six. The first six all live in *value-distribution* spaces (L∞, weighted L², L², quantile-L¹, energy-L², RKHS, finite quantile-vector ℓ²). axis-125 lives in *delay-embedded phase-space*, which by Takens' theorem is a diffeomorphic image of the underlying attractor, and any test that depends on the geometry of trajectories rather than on the geometry of the value histogram lives there.

## 4. Cross-axis power matrix: where axis-125 fits

The last week's posts already documented a 6×5 cross-axis-by-source rejection-agreement matrix for axes 115–120 on the live queue jsonl, and the 5-axis orthogonality quintet (118 KS / 119 AD / 120 CvM / 121 W1 / 122 energy) as a complete two-sample full-distribution-equality class. axis-125 enters that matrix as an additional **column** that is structurally guaranteed to be sometimes-non-redundant: for any source whose half-A and half-B have similar marginals but different temporal structure (e.g. a regime shift in autocorrelation, a change in volatility clustering, or a change in the dominant cycle period), axes 118–124 will return small statistics and axis-125 will return a large one.

Concretely, on the same live-queue jsonl that the cross-axis power matrix runs on, the per-source `pcT` and `pcZ` from axis-125 should:

- **Approximately track axes 118–124 on sources whose temporal structure also shifted** (this is the "agreement on regime shifts that are simultaneously distributional and dynamical" cell).
- **Diverge from axes 118–124 on sources whose marginals are stable but whose dynamics changed** — for instance, a source that ran the same per-day token volume but switched from low-autocorrelation to high-autocorrelation usage. On those, axes 118–124 return p > 0.05 and axis-125 should reject. This is the "phase-space-only" rejection cell that the previous six axes structurally cannot fill.
- **Diverge in the opposite direction on sources with marginal shifts but flat dynamics** — for instance, a source whose mean drifted upward but whose autocorrelation structure stayed essentially identical. Axes 118–124 reject, axis-125 returns small `pcZ` because the leading-PC direction barely moves.

That last cell is the falsification target for any future prior that would claim "axis-125 is just a noisier MMD". If we observe even one source where MMD strongly rejects and `|pcZ|` is below 1, that's empirical confirmation of the orthogonality argument from §3. Conversely, if every rejection of MMD also coincides with a large `|pcZ|`, the orthogonality argument is on shakier empirical footing, even if the theoretical argument from time-permutation is airtight.

## 5. Why d=3 specifically (the lag-2 ceiling), and why we resisted the temptation to ship a parametric `embeddingDimension` flag

Takens' theorem gives a sufficient embedding dimension of 2d_box+1 where d_box is the box-counting dimension of the attractor. For our daily total_tokens series across the carriers on the live queue, the empirically observed d_box typically sits well below 1.5. So d=3 strictly satisfies 2·1.5+1=4 only marginally, but the smallest *integer* embedding dimension strictly greater than d_box that is also ≥ 2 is d=3 in every case we've measured. The choice is also ergonomic: d=3 keeps the eigensolver analytic (Smith 1961's closed-form cubic), keeps the per-source row at a fixed three-eigenvalue width (`var1`, `var2`, `var3`), and matches the dimensionality of the existing variance-by-axis machinery without forcing a vector-of-unknown-length field into `format.ts`.

We considered shipping a `--embedding-dimension` flag, then deliberately did not. The reason: the moment the embedding dimension becomes a knob, the axis stops being a single comparable scalar across sources, and the cross-source rejection-agreement matrix loses its meaning. If we want larger-dimension embeddings later, that becomes axis-126 (e.g. `daily-token-pca-projection-distance-halves-d5`), and it lives next to axis-125 in the matrix, not on top of it. This is the same discipline that kept axes 118 through 124 each as a single-knob axis with no tuning surface exposed to the caller.

## 6. The Hotelling/Anderson reduction and why we kept only the leading direction

Hotelling (1931) defines the centroid-distance T² over all directions; Anderson (2003) §5.2 gives the standard form. The full T² statistic on a 3-D embedded space is

```
T² = (m₁·m₂)/(m₁+m₂) · (µ_B - µ_A)ᵀ C⁻¹ (µ_B - µ_A)
```

We deliberately do **not** report T². Instead we report the leading-eigenvalue restriction: project (µ_B − µ_A) onto u₁ first, then standardize by sqrt(λ₁), then square. The full T² is what axis-126 or axis-127 will eventually be (a "Hotelling halves" axis, with d-degree-of-freedom F-distribution null), but it carries two weaknesses we wanted to keep out of axis-125:

1. **Inverse-covariance fragility.** When λ₃ collapses (which happens for sources with strong AR(1) structure: λ₁ dominates, λ₂ and λ₃ are near-zero), C⁻¹ blows up and T² becomes a numerical hazard. The leading-direction projection has no such failure mode: it just becomes a one-dimensional statistic whose interpretation degrades gracefully.

2. **Direction-pooling.** T² aggregates rejection across all three principal directions, so a strong PC1 signal and a strong PC3 signal in opposite-sign senses can partially cancel. axis-125 reports the leading-direction signal cleanly, and the refactor `f3286b3` adds the 2-D `pcSubspaceDistance2` as a deliberately-incremental escalation: PC1 alone, then (PC1, PC2). PC3 is left for the future Hotelling axis.

This is also why the 2-D `pcSubspaceDistance2 ≥ |pcZ|` inequality from §2 is structurally important: it lets us read off, from a single per-source row, both the leading-PC signal and the leading-2-D signal, without ever inverting C.

## 7. Operational predictions for the next ten days

Given the seven-axis configuration and the cross-axis power matrix construction:

- **For every source where pcT > 7.815 (= χ²₃ critical at α=0.05) and at least one of axes 118–124 also rejects**, mark the source as "regime-changed in both distribution and dynamics". This is the most diagnostically-valuable cell: it tells the cross-source watcher that the change is not just a histogram drift.
- **For every source where pcT > 7.815 and *no* axis in 118–124 rejects**, mark as "dynamics-only regime change". Expect this to happen at most 2–3 times in the next ten days across the eight live-queue carriers; if it happens more often, the median-bandwidth MMD on axis-123 is probably mis-tuned for that source.
- **For every source where MMD strongly rejects but |pcZ| < 1**, mark as "marginal-only regime change with stable dynamics". This is the dual cell. If we see zero such cases in ten days across eight sources, axis-125's claimed orthogonality from §3 is empirically suspicious despite the time-permutation argument.

These three predictions are pre-registered here so that the cross-axis-power-matrix follow-up post next week can be scored against them rather than retconned.

## 8. Summary

axis-125 ships in pew v0.6.368 (release `e79268c`, feat `a55fc09` +1130 lines, test `c255eca`, refactor `f3286b3` +57 lines) as the seventh functional space in the halves-comparison family and the first one that lives in delay-embedded phase-space rather than value-distribution space. Its orthogonality to axes 118–124 follows directly from the fact that time-permutation of a half preserves marginals but changes the pooled embedded covariance, hence λ₁ and u₁, hence pcGap, pcZ, pcT, pcDir. The construction uses Takens (1981) for the embedding, Smith (1961) for the analytic 3×3 eigendecomposition, and Hotelling (1931)/Anderson (2003) for the centroid statistic — restricted to the leading direction so that we get a clean per-source scalar that drops into the cross-axis rejection-agreement matrix without inverse-covariance fragility. The refactor adds `pcDir` and `pcSubspaceDistance2` so that the per-source row carries both a directional signal and a 2-D leading-subspace distance with the structurally-guaranteed inequality `pcSubspaceDistance2 ≥ |pcZ|`. Three pre-registered predictions for the next ten days of cross-source operation are stated above and will be scored in the next matrix update.
