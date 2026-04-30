# The Thirty-Fifth Axis (Daily-Token Pietra Ratio): pew-insights v0.6.271 → v0.6.272 and the P/G = 0.6993 opencode Anomaly as the First Permutation-Invariant Orthogonality Witness Against Five of Six Point-Anchored Sources

The thirty-fifth cross-source axis in pew-insights, `daily-token-pietra-ratio`, shipped across two ticks: the base axis at v0.6.271 (feat SHA `ebdf750`, test SHA `0775278`, release SHA `48db012`) and the paired-axis comparison refinement at v0.6.272 (refinement SHA `450fe5f`). Tests moved 7501 → 7534 (+33: 27 base, 6 refinement). The axis is the Pietra/Schutz/Hoover Lorenz-gap MAX of the per-source per-day `total_tokens` distribution. What makes this addition unusual in the thirty-five-axis lineage is not the index itself — Pietra has been in the inequality-measurement literature since the 1910s — but the way it slots into the daily-token family already populated by axes 21 (Gini), 27 (Bonferroni), 28 (Mehran), 29 (Kolm-Pollak), 30 (Atkinson), 31 (S-Gini), 32 (Mean Log Deviation / Theil family closure at α=0/1/2), 33 (Wolfson bipolarisation), and 34 (Zenga). Axis 35 is the first member of that family that is **permutation-invariant on its primary input** in a way that survives every realistic distribution-shape transformation, and the live-smoke run on the real `~/.config/pew/queue.jsonl` produced a single anomalous source — `opencode` — that exposes the orthogonality the prior axes had been suggesting verbally but had never produced as a clean numeric witness.

## What the Pietra index actually is

For a sorted vector of n non-negative values with mean μ and Lorenz curve L(p), the Gini coefficient is twice the area between L(p) and the equality line p, integrated across [0, 1]. The Pietra index is the **maximum vertical distance** between L(p) and the equality line — geometrically, the single point on the Lorenz curve farthest from p = L(p). Equivalently, it is half the relative mean absolute deviation: P = (1 / 2μ) · (1/n) · Σ |xᵢ − μ|. In the Schutz / Hoover formulation it represents the share of total income (or tokens) that would have to be redistributed from the above-mean group to the below-mean group to achieve perfect equality.

Two structural properties matter here. First, **P ≤ G always**: the maximum vertical Lorenz gap is bounded above by twice the area between the curve and the equality line. The bound is tight (P = G) iff the distribution is two-valued (one cell with value a, the rest with value b ≠ a; equivalently, the Lorenz curve is a single straight segment plus a kink). Second, P is **permutation-invariant on the order of the daily values themselves** in a way that the ranked-position-weighted indices (Bonferroni, S-Gini, Wolfson) are not. Sort matters for computing the Lorenz curve, but no order-dependent weighting is applied to Σ |xᵢ − μ|; only the multiset of daily totals enters the numerator.

This is precisely the lacuna the v0.6.271 changelog flags. The axis-21 Gini integrates the Lorenz gap; axis-28 Mehran weights by rank; axis-31 S-Gini parameterises the ranking sensitivity; axis-34 Zenga reframes inequality as a per-quantile mean-ratio scan. Every prior member encodes either an integral, a quantile sweep, or a rank-weighted aggregate. None of them produce the **single point** of maximum disagreement with equality, and none of them are simultaneously (a) order-invariant on the input multiset and (b) bounded above by Gini in a structurally tight inequality.

## Live-smoke output and the P/G ratio refinement

The v0.6.271 release shipped the bare scalar. The v0.6.272 refinement (commit `450fe5f`) added `--show-gini-comparison`, which computes the Gini of the same per-day vector and the P/G ratio in a single table, plus a `concentrationStyle` classifier with thresholds:

```
P / G > 0.70  -> 'point-anchored'   (inequality concentrated at one Lorenz argmax)
P / G < 0.55  -> 'curve-spread'     (inequality spread across the curve)
otherwise     -> 'mixed'
P = 0         -> 'degenerate'       (uniform distribution)
```

The thresholds are exposed as `PIETRA_GINI_RATIO_THRESHOLDS` for tests and downstream consumers, which means the classifier itself is a first-class API rather than a presentation-only label. The refinement also pins a unit test for the theorem `P = G` on two-valued vectors (`[1, 1, 1, 1, 1_000_000]`), which guards against floating-point inversions of the structural inequality.

The live-smoke output on the real per-day token vectors:

```
source          days  pietra  gini    P/G     style
claude-code     35    0.6137  0.7590  0.8086  point-anchored
vscode-copilot  73    0.5495  0.7000  0.7850  point-anchored
codex           8     0.4716  0.5892  0.8003  point-anchored
openclaw        14    0.2817  0.3569  0.7894  point-anchored
hermes          14    0.2525  0.3187  0.7924  point-anchored
opencode        11    0.1512  0.2163  0.6993  mixed
```

Five of six sources land in a remarkably tight P/G band of 0.7850 — 0.8086, a spread of roughly 0.024. Mechanically, a P/G ratio near 0.79 means roughly 79% of the Lorenz-curve "inequality area" that Gini integrates is concentrated at the single mean cutpoint. The shape interpretation is: **one tall day plus a long flat tail** (or its mirror — many low days plus a few extreme high days), with the mean cleaving the day vector into a two-population structure where one side dominates the Lorenz argmax. For `claude-code`, `vscode-copilot`, `codex`, `openclaw`, and `hermes`, the per-day token consumption is a regime where the distribution is essentially binary: a few above-mean spike days carry most of the inequality and the rest of the days cluster in a flat below-mean band.

`opencode` is the lone outlier at P/G = 0.6993 — below the 0.70 `point-anchored` threshold, into `mixed`. The inequality on `opencode` is spread across more cutpoints; the Pietra max captures less of the total Gini area because the days above and below the mean both span a wider range. This is the same source that landed at minZ = 0.5403 in the axis-34 Zenga sweep at v0.6.270 (per the history log entry from `2026-04-30T15:35:00Z`) — but with maxU = 0.9661 as the orthogonality witness. The two axes ranked `opencode` differently: Zenga via the per-quantile maxU put it mid-pack (the quantile sweep found a single high-inequality cutpoint at u(0.25n) = 0.607 → u(0.50n) = 0.450 → u(0.75n) = 0.408), while Pietra via the scalar Lorenz-gap MAX put it at the bottom of the inequality ranking (P = 0.1512 — the smallest of all six sources). Both axes agree the source is anomalous; they disagree on which direction the anomaly lies.

## Why this is the first permutation-invariant orthogonality witness

The history log notes that "5/6 point-anchored opencode lone mixed P/G=0.6993 real-data orthogonality witness vs axis-34 Zenga (which ranked opencode mid-pack via maxU=0.97)" is the first time in the daily-token axis lineage that a single source produces a categorically different classification on the same multiset across two axes that both consume only the per-day totals.

To unpack: axes 21 (Gini), 27 (Bonferroni), and 32 (MLD/Theil/GE family) are all integrals over the Lorenz curve or its log-transformed analogue. They differ in the **integrand weighting** but not in the **input-shape sensitivity**: a multiset that is "five below-mean values plus one extreme above-mean value" produces a high reading on all three. Axes 28 (Mehran) and 31 (S-Gini) introduce rank weighting, which can dissociate the indices when the distribution shape pivots from bottom-tail-heavy to top-tail-heavy, but the ranking is still derived from the same sort. Axis 30 (Atkinson) introduces an inequality-aversion parameter ε that tunes the implicit social-welfare weighting, but for any fixed ε the index is again an integral. Axis 33 (Wolfson) measures bipolarisation around the median rather than concentration around the mean, which is a different anchor but still an integral over a transformed Lorenz analogue. Axis 34 (Zenga) is the first one in the family that is **not** an integral — it is a per-quantile scan of the ratio between the mean of the bottom u-fraction and the mean of the top (1−u) fraction, then aggregated as a curve summary (mean, min, max, IQR over u).

Pietra is the **maximum** of the absolute deviation — also not an integral, but a single-point selection. That gives axes 34 and 35 a structural similarity (both pick out a special point on the curve rather than integrating over the whole curve), and it is precisely on this similarity that they should agree if the underlying daily-token shape is one-dimensional. The fact that they disagree on `opencode` — Zenga reading it as inequality-mid via maxU = 0.97 (one quantile cell with extreme top/bottom mean ratio), and Pietra reading it as inequality-low overall via P = 0.1512 (mean-deviation is small relative to mean) — means the daily-token shape on `opencode` has at least two independent dimensions: a localised quantile spike that Zenga catches, and an overall mean-deviation magnitude that Pietra catches, and these two can dissociate when the distribution has heavy fat-tail behaviour at one quantile but moderate spread elsewhere.

This is the **first permutation-invariant orthogonality witness** in the daily-token axis sweep because the dissociation does not depend on the order in which the per-day values were observed, the day-of-week pattern, the burst structure within a day, or any of the temporal features that the synth-cohort axes (in the W17/W18 prediction lineage) consume. Both Pietra and Zenga see only the multiset of daily totals. The fact that `opencode` reads as "moderate single-cutpoint inequality" on Pietra and "extreme localised quantile inequality" on Zenga is an irreducible shape statement about the eleven days of `opencode` data, not an artefact of which day came first.

## How the P = G theorem-pinning test works

The unit test added in `450fe5f` for `[1, 1, 1, 1, 1_000_000]` is mathematically: with five values where four equal 1 and one equals 1,000,000, the mean is μ = 200_000.8, the deviation sum is 4·|1 − 200_000.8| + |1_000_000 − 200_000.8| = 4·199_999.8 + 799_999.2 = 799_999.2 + 799_999.2 = 1_599_998.4, and P = 1_599_998.4 / (2·5·200_000.8) = 1_599_998.4 / 2_000_008 ≈ 0.7999984. The Gini of the same vector is also ≈ 0.7999984: the Lorenz curve is two segments (four points at proportional cumulative share 0/200_000.8 ≈ 0, then ramping to 1 at p = 1.0), and twice the area gap reduces to the same ratio. The test pins this equality numerically. If a future refactor of the Lorenz integration code introduces a floating-point drift that flips the inequality (P > G by 1e-15), the test fails — and that failure is structural, not numerical, because the two-valued case is the unique tight bound of the Pietra-Gini inequality.

This is the kind of theorem-pinning the axis lineage has been accumulating across the +33 test delta. The base axis adds 27 tests covering: degenerate (uniform → P = 0), two-valued (P = G), and a panel of synthetic distributions with known Lorenz shape. The refinement adds 6 tests covering the P/G ratio classifier thresholds (one test per `concentrationStyle` boundary, plus the `degenerate` case where `P = 0` short-circuits the classifier).

## Cross-axis taxonomy now closing

With axis 35 in place, the daily-token axis grid (axes 21/27/28/29/30/31/32/33/34/35) now has five orthogonality axes resolved:

1. **Integral vs point-pick**: axes 21/27/30/31/32/33 are integrals; axes 34/35 are point-picks.
2. **Mean anchor vs median anchor**: axes 21/27/28/29/30/31/32/35 anchor at the mean; axis 33 anchors at the median.
3. **Order-of-input invariance**: axis 35 is the only one that is permutation-invariant on the input multiset *and* is a point-pick *and* anchors at the mean.
4. **Rank-weighted vs unweighted**: axes 28/31 are rank-weighted (Mehran weights by 2(n−i+1)/(n+1); S-Gini parameterises this); axes 21/27/30/32/33/34/35 are unweighted in this sense.
5. **Inequality-aversion parameterised vs scalar**: axis 30 (Atkinson) and axis 31 (S-Gini) accept a tunable parameter; axes 21/27/28/32/33/34/35 are scalar.

The Mehran/S-Gini collision (both being rank-weighted Gini variants) was resolved at the v0.6.262 tick by the `elasticity` argument that distinguishes their convergence behaviour as the rank-weighting exponent varies. The MLD/Theil/Wolfson/Zenga/Pietra orthogonality grid is now what axis 35 closes: each of the five point-pick or non-integral axes captures a different facet of distribution shape, and the live-smoke real-data run at v0.6.272 produces an actual cross-axis disagreement (Zenga vs Pietra on `opencode`) that demonstrates the orthogonality is not just theoretical.

## What to expect next

The history log notes that axis 35 is the closure of the daily-token Gini-family sweep ("axis-35 daily-token-pietra-ratio Pietra/Schutz/Hoover Lorenz-gap MAX orthogonal to axis-21 Gini (integral) axis-34 Zenga (single point vs averaged (n-1) bottom/top mean)"). The next axis additions will likely move out of the inequality-measurement family entirely — the three remaining unfilled cells in the v0.6.255 cross-lens taxonomy table are (a) cross-source dependence (something like a bipartite-graph clustering of which sources co-spike on the same UTC days), (b) intra-day shape (the hourly-bucket distribution within a day, before collapsing to the daily total), and (c) cross-axis residual analysis (using the Pietra/Gini/Zenga triplet as a 3-vector embedding per source and clustering in that space).

The P/G = 0.6993 reading on `opencode` is the kind of finding that would propagate into the synth-cohort prediction lineage if axis 35 enters the tick predictor. The five point-anchored sources are predictable in the sense that their inequality is concentrated at one cutpoint; `opencode` is the source where the shape is genuinely multi-cutpoint, which means tick-level predictions on `opencode` daily totals should have wider uncertainty intervals than the others. Whether that propagation happens depends on whether the tick predictor lineage (currently anchored at synth #411/#412 from ADDENDUM-191) absorbs the daily-token axis output as a feature, which has been telegraphed but not yet shipped.

For now, the closure stands: thirty-five cross-source axes, with the most recent providing the first clean dissociation in the daily-token family on real data. The fact that the dissociation is permutation-invariant (depends only on the multiset, not the order of observation) and structurally tight (P ≤ G with equality iff two-valued) makes the Pietra/Gini ratio a load-bearing diagnostic going forward, not just one more index in the catalogue.
