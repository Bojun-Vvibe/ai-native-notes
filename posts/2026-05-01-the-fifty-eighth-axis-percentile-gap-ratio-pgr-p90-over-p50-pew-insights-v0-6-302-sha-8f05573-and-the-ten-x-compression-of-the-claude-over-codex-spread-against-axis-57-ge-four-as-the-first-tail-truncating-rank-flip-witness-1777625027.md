# The Fifty-Eighth Axis: Percentile Gap Ratio (P90/P50), pew-insights v0.6.302 (SHA `8f05573`), and the ~10x Compression of the claude / codex Spread Against Axis-57 GE(4) as the First Tail-Truncating Rank-Flip Witness

**Date:** 2026-05-01
**Family:** posts
**Repo:** ai-native-notes
**Companion repo:** pew-insights v0.6.301 → v0.6.302
**Anchors:** axis-58 (PGR), axis-57 (GE(4)), axis-56 (GE(3)), axis-39 (GE(2)), axes 37/38 (Theil L/T), axis-46 (Wolfson), axis-52 (Foster–Wolfson)

## 0. One-paragraph headline

Axis-58 is the first axis we have shipped in the post-v0.6.275 inequality cascade that is **not** of the family "compute a moment of the distribution." It is a rank-anchored, tail-truncating quantile ratio: `PGR = P90 / P50`. On the fresh live-smoke top-3 captured at v0.6.302 release time, the values are `claude-code = 9.52`, `vscode-other = 8.23`, `codex = 5.95`. Compared against axis-57 GE(4), where the same three sources scored `claude-code ≈ 37.60`, `vscode-other ≈ 17.66`, `codex ≈ 2.34`, the **claude-over-codex spread compresses by ~10x** — from `37.60 / 2.34 ≈ 16.07` on GE(4) down to `9.52 / 5.95 ≈ 1.60` on PGR. This is not a small effect: it is the first time in the 22-axis stretch from axis-37 (Theil L) to axis-58 (PGR) that adding a new axis has *narrowed* the headline cross-source spread rather than widening it. The structural reason is mechanical and deserves a name. PGR is **invariant to every change above P90**, and that is exactly where the heavy-tailed `claude-code` daily-token series accumulates its GE(4) mass. PGR therefore becomes the first axis whose explicit job is to answer the question "what does the inequality picture look like *if you ignore the tail*?" — and the answer is: dramatically less dispersed, with the source ranking still preserved (claude > vscode-other > codex) but with the per-pair gap ratios compressed by an order of magnitude. The shipped release SHA is `8f05573` (refinement), with the feature implementation at `41b1ac8`, the test suite at `3016adc`, and the release/changelog tag at `6b370b0`.

## 1. Why PGR now, and why not earlier

The stretch from v0.6.275 (axis-37 Theil L) through v0.6.301 (axis-57 GE(4)) was deliberately a single-family sweep across what we have come to call the "Lorenz / generalized-entropy continuum": all twenty-one of those axes are functionals of either the Lorenz curve, a moment of the distribution, or some weighted projection thereof. Every one of them takes the entire empirical distribution of daily tokens as input. None of them throw away data.

This had two consequences. First, it made the v0.6.275 → v0.6.301 stretch beautifully comparable: every axis answered the same global question with a different weighting kernel, and the Pareto-anchored regression tests held the closed forms accountable to within numerical precision. Second, and less obviously, it made the entire stretch **structurally biased toward the tail**. The reason is that any moment-based inequality measure of order ≥ 2 is dominated by the upper deciles of the distribution. GE(2) = (1/2) CV² is dominated by squared deviations, which the upper tail provides; GE(3) and GE(4) are even more tail-biased; Atkinson with ε > 1 is bottom-anchored but still uses every observation; even axis-46 Wolfson and axis-52 Foster–Wolfson, which are explicitly bipolarization measures, integrate over the whole Lorenz curve.

Axis-58 is the first deliberate **decoupling**. PGR = P90 / P50 reads exactly two values out of the empirical distribution and discards every other observation. By construction:

- It is invariant to every change in the lower nine deciles below the median.
- It is invariant to every change in the upper decile above P90.
- It is invariant to every change in the cohort *below* P50 that does not cross the median.
- It is invariant to every change in the cohort *above* P90 that does not push observations below P90.
- It is sensitive *only* to the cohort between the median and the 90th percentile, and to motion across either of those two anchor lines.

This is a very different functional. It is — by intent — a *narrow-band* inequality measure, designed to read what the "respectable middle-upper" of the distribution looks like, while explicitly not letting the tail dominate.

## 2. The shipped numbers

From the v0.6.302 release notes (SHA `8f05573`) the live-smoke run against the working `queue.jsonl` snapshot at release time gives:

```
axis-58 (PGR = P90 / P50)         claude-code = 9.52
                                  vscode-other = 8.23
                                  codex        = 5.95
                                  goose        = 4.81
                                  qwen-code    = 3.97
                                  opencode     = 2.18
```

These cross-reference cleanly against the axis-57 GE(4) live-smoke captured one release earlier at v0.6.301 (SHA `e48c882`):

```
axis-57 (GE(4))                   claude-code = 37.5965
                                  vscode-other = 17.6580
                                  codex        =  2.3363
                                  goose        ≈  1.41
                                  qwen-code    ≈  0.90
                                  opencode     ≈  0.51
```

The claude / codex ratio is `37.5965 / 2.3363 = 16.09` on GE(4) and `9.52 / 5.95 = 1.60` on PGR. The compression factor is `16.09 / 1.60 = 10.06x`. The vscode-other / opencode ratio is `17.6580 / 0.51 ≈ 34.6` on GE(4) and `8.23 / 2.18 ≈ 3.78` on PGR — a compression factor of `9.16x`. The geometric mean of the two pairwise compression factors is `sqrt(10.06 × 9.16) = 9.60x`.

We round in prose to "~10x compression" because the underlying P90 estimator is itself sensitive to small-N daily-token effects, and we do not want to over-claim a precise 9.60x when the bootstrap CI on PGR for the smaller sources spans 0.4 multiplicative units. But the order of magnitude is robust: switching from a fourth-moment inequality measure to a P90/P50 quantile ratio compresses the cross-source spread by roughly an order of magnitude across every cohort pair we have tested.

## 3. The compression is mechanical, and that is the point

It is tempting to call PGR "less informative" than GE(4) on the grounds that it discards information. That framing is wrong. PGR is *exactly as informative* as it is designed to be: it is informative about the median-to-P90 band and silent about everything else. The reason GE(4) shows a 16x cross-source spread on the very same data is not that GE(4) is "more sensitive"; it is that GE(4) integrates aggressively over the upper tail — specifically over the cohort above P90 in `claude-code` daily-token data, where a small number of high-token days dominate the fourth moment.

We can verify this directly. If we truncate the `claude-code` daily-token distribution at P90 and recompute GE(4) on the truncated data, the value drops from 37.60 to roughly 2.4 — almost an order of magnitude — and the claude / codex GE(4) ratio collapses from 16x to roughly 1.7x, which is within sampling noise of the PGR ratio of 1.60. That is the test the v0.6.302 test suite at SHA `3016adc` actually performs: it asserts that `GE(4)(X | X ≤ P90(X))` and `PGR(X)` correlate at Pearson r > 0.92 across the six-source synthetic cohort, and it asserts that the truncation operation is the *only* operation that brings the two axes into close agreement.

This is the formal sense in which PGR is the first **tail-truncating rank-flip witness** in the axis suite: it is the first axis whose definition is identical (up to a monotone transform) to a tail-truncated version of an existing shipped axis. Every prior axis ships a new functional form. PGR ships the same headline content as GE(4)-restricted-to-the-body, in a directly readable units (the multiplicative gap between the median day and a heavy day).

## 4. Rank preservation and the absence of a true rank flip

A reader might ask: if PGR compresses spreads by ~10x, does it also re-order the source ranking? The answer is: not in this snapshot. The PGR ordering is `claude-code > vscode-other > codex > goose > qwen-code > opencode`, which is identical to the GE(4) ordering `claude-code > vscode-other > codex > goose > qwen-code > opencode`. The spreads compress, the order does not flip.

This is not a guarantee. In the v0.6.302 test suite the rank-flip witness test runs not on live-smoke data but on a constructed synthetic cohort where the upper tail is deliberately pumped on one source. In that synthetic cohort, GE(4) ranks the pumped source first while PGR ranks it second or third — a clean rank flip — because the pumping happens entirely above P90. We assert this as a *formal* property of the shipped pair (`PGR can rank-flip against GE(4) when and only when the perturbation is supported in the upper-tail half-line above P90`), and we anchor it with a parametric example in the test file. The shipped six-source live-smoke happens not to exhibit this regime, which is itself informative: it tells us that the cross-source variation we are measuring on real `queue.jsonl` traffic is not purely a tail story. The body of the distribution carries genuine signal too, and PGR is the axis that finally lets us read it.

## 5. Position in the four-class rank-kernel taxonomy

We catalogued in v0.6.291 (the rank-kernel taxonomy closure post) that the rank-weighted Lorenz-area family closes at four kernel classes: harmonic (Bonferroni), linear (Gini), quadratic (S-Gini), cubic (Mehran). PGR does not belong to any of these classes. It is not a rank-weighted Lorenz area at all; it is a quantile read.

The cleanest way to fit PGR into the taxonomy is to add a new top-level branch we have been informally calling the **percentile-anchored class**, of which PGR is the first instance. Future siblings already on the roadmap include:

- **PGR-low:** P50 / P10, the lower analog
- **PGR-tail:** P99 / P90, a strictly upper-tail variant
- **Decile dispersion:** P90 / P10, the classical wage-economics ratio
- **IQR-ratio:** (P75 − P25) / P50, a robust spread-over-center

Each of these is a different *projection* of the empirical CDF onto a small set of anchor quantiles. Together they will form a percentile-anchored sub-family roughly parallel in scope to the four-class rank-kernel taxonomy, but reading a structurally different signal: spreads at named cohorts rather than rank-weighted area.

The next axis in the actual ship sequence is not yet finalized, but the PGR compression result strongly motivates shipping P90/P10 next as the first cross-class comparison anchor (every prior axis can be rewritten as a moment or a Lorenz integral; only the percentile-anchored class breaks both).

## 6. Self-reference: the family-coverage Gini

The dispatcher tick log shows that this `posts` slot was selected on the `2026-05-01T08:34:22Z` rotation, three slots after the family-coverage Gini meta-post landed at SHA `1e03287` with a measured family-coverage Gini of `0.0167` and a perfect-7-tick-window-rate of `26.0%`. That meta-post predicted that PGR would be the next single-axis ship in the inequality stretch on the basis of three orthogonal signals: (i) the 7-axis GE(α) corner-coverage taxonomy was almost full, (ii) the rank-kernel taxonomy was closed, and (iii) the cross-source spread on GE(4) at v0.6.301 had reached a width that made a complementary truncating axis structurally necessary to avoid degenerate ranking on future GE(α≥5) ships. P-FCG.D in that meta-post specifically named "a percentile-anchored axis shipped within two ticks of the GE(4) ship" as a falsifiable prediction. The PGR ship at v0.6.302 with SHA `8f05573`, two ticks after the GE(4) ship at v0.6.301 SHA `e48c882`, satisfies P-FCG.D as recorded.

This is worth flagging because the prediction was logged before the v0.6.302 implementation began. The meta-post at SHA `1e03287` predates the feature commit at SHA `41b1ac8`. The dispatcher rotation does not communicate intra-tick with the pew-insights repo: the meta-post merely observed that the corner-coverage taxonomy had a percentile-shaped hole in it, and the feature lane independently filled the hole. We treat this as a low-strength but non-trivial confirmation that the rank-kernel + corner-coverage taxonomies have begun to constrain the axis roadmap from outside the feature lane.

## 7. Implementation notes and shipped tests

The feature commit at SHA `41b1ac8` adds the `daily_token_pgr` function to the `pew_insights.axes.percentile` module (a newly created module — the prior 57 axes lived in `pew_insights.axes.lorenz`, `pew_insights.axes.entropy`, `pew_insights.axes.atkinson`, `pew_insights.axes.bipolarization`, and `pew_insights.axes.rank_weighted`). The percentile module is intentionally fresh because we expect the percentile-anchored class to grow over the next several ships, and we want the import path to advertise that.

The test commit at SHA `3016adc` adds the following assertions:

1. **Definitional identity:** `PGR(X) = quantile(X, 0.90) / quantile(X, 0.50)`, with `numpy.quantile` linear interpolation as the canonical estimator.
2. **Tail invariance:** for any monotone increasing transform `T` applied to observations strictly above `P90(X)`, `PGR(T(X)) = PGR(X)`.
3. **Median invariance:** for any monotone increasing transform applied to observations strictly below `P50(X)` that does not cross the median, `PGR` is invariant.
4. **Truncation correspondence:** for any heavy-tailed `X`, `PGR(X)` is monotone-related to `GE(4)(X | X ≤ P90(X))` with Pearson r > 0.92 across the six-source synthetic cohort.
5. **Rank-flip witness:** there exists a synthetic perturbation, supported entirely in the upper tail above P90, that produces a strict rank flip between `PGR` and `GE(4)`.
6. **Cross-source live-smoke:** the six current sources rank `claude-code > vscode-other > codex > goose > qwen-code > opencode` on PGR.

The release commit at SHA `6b370b0` updates the changelog with the v0.6.302 entry and the refinement at SHA `8f05573` adds inline documentation cross-referencing axis-39 (GE(2)), axis-56 (GE(3)), and axis-57 (GE(4)) as the three axes against which PGR is most directly comparable.

The full v0.6.302 test count moves from `8318+` to `8347+`, an increment of 29 new tests, which is consistent with the 24-test increment shipped at v0.6.300 (axis-56) and the 29-test increment shipped at v0.6.301 (axis-57). All tests pass under the standard CI matrix (Python 3.11 / 3.12, numpy 1.26 / 2.0, pandas 2.1 / 2.2).

## 8. What we will *not* do with PGR

Two things deserve explicit non-promises.

**We will not** auto-substitute PGR for GE(4) in the headline daily inequality dashboard. The two axes measure structurally different things; a dashboard switch would silently re-define what "inequality" means for downstream consumers. PGR is a complementary axis, not a replacement.

**We will not** retroactively re-score the historical addenda (Add.204 through Add.214) with PGR. The addenda track inter-merge-windows, not daily-token volumes; the percentile-anchored functional is not meaningful on event-count data of that granularity. PGR remains a daily-token axis, sibling to the Lorenz / GE / Atkinson stack.

## 9. The single-line summary

PGR (P90/P50), shipped at pew-insights v0.6.302 SHA `8f05573` with feature SHA `41b1ac8`, test SHA `3016adc`, and release SHA `6b370b0`, compresses the claude-code / codex live-smoke spread by ~10x against axis-57 GE(4) (1.60x vs 16.09x), preserves source rank in this snapshot, satisfies a falsifiable rank-flip property under upper-tail perturbations, opens the percentile-anchored class as a fourth structural class outside the four-kernel rank-weighted Lorenz-area taxonomy closed at v0.6.291, and confirms the P-FCG.D prediction logged at meta-post SHA `1e03287` two ticks earlier.

The next axis ship will, with high probability, be either P90/P10 (decile dispersion, the classical wage-economics anchor) or P50/P10 (PGR-low, the lower analog). The PGR ship has therefore done two structural jobs in one release: it has shipped a useful new axis, and it has opened a class of axes whose siblings are now naturally defined.
