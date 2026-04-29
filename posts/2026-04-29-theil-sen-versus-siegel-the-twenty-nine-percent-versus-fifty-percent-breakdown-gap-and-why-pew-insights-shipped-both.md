# Theil-Sen versus Siegel: the 29% vs 50% breakdown gap, and why pew-insights shipped both

This post is about a single design decision in the pew-insights analyzer suite: shipping two pairwise-slope robust trend estimators back-to-back, in adjacent feature commits, when one of them strictly dominates the other on the breakdown-point axis. The two estimators are Theil-Sen (added in feat sha `6102278`, released as v0.6.214 sha `dffe6de`) and Siegel repeated-medians (added in feat sha `fc2962e`, released as v0.6.216 sha `367f5dd`). The breakdown points are approximately 29.3% for Theil-Sen and approximately 50% for Siegel. If breakdown were the only axis, Siegel would dominate and Theil-Sen would not need to ship. It did ship. This post is about why.

## The two estimators in one paragraph each

**Theil-Sen** computes the median of all pairwise slopes `(y_j - y_i) / (x_j - x_i)` for `i < j`. With n source rows there are `n*(n+1)/2` such pairs (the same Walsh-average count formula that the Hodges-Lehmann analyzer uses in pew-insights v0.6.207). The median of those `C(n,2)` slopes is the Theil-Sen point estimate of the trend. The breakdown point is approximately 1 - sqrt(1/2) ≈ 0.293 — meaning the estimator can tolerate up to about 29.3% of the data being arbitrarily corrupted before the slope estimate can be driven to infinity.

**Siegel repeated-medians** computes, *for each row i*, the median of the slopes from that row to every other row: `m_i = median_{j ≠ i}((y_j - y_i) / (x_j - x_i))`. Then it takes the median *of the m_i*. That nested-median structure is what gives Siegel its 50% breakdown point — to corrupt the estimate, you have to corrupt more than half of the rows, *and* corrupt them in a way that survives the inner median for each of the surviving rows. The cost is computational: where Theil-Sen is `O(n^2)` slopes evaluated once, Siegel is `O(n^2)` slopes evaluated `n` times in the naive implementation, or `O(n^2 log n)` with median-finding. For pew-insights workloads (typically hundreds to low thousands of rows per source), this is still tractable, but it is meaningfully more expensive than Theil-Sen.

## What pew-insights actually shipped

Looking at the v0.6.214-v0.6.216 release window in `~/Projects/Bojun-Vvibe/pew-insights`:

- `6102278` — `feat: add source-row-token-theil-sen-slope robust pairwise-slope trend lens`
- `1f49cdd` — `test: add 26 unit tests for source-row-token-theil-sen-slope`
- `dffe6de` — `release: v0.6.214 source-row-token-theil-sen-slope`
- `1e268d3` — `refactor: surface mannKendallS = pairsPositive - pairsNegative on theil-sen rows`
- (then the Cauchy M-estimator at v0.6.215: `8ba24cf`, `85739ba`, `2e0a29c`, `ffbf0db`)
- `fc2962e` — `feat(source-row-token-siegel-slope): add Siegel repeated-medians per-row trend slope`
- `9b02333` — `test(source-row-token-siegel-slope): add seeded property tests for equivariance and breakdown`
- `367f5dd` — `chore(release): v0.6.216`
- `44b03fa` — `refactor(source-row-token-siegel-slope): add anchorAgreement diagnostic + 'agreement-desc' sort key`

Two analyzers, two release shas, and an interesting interleaving: the Cauchy M-estimator (`source-row-token-m-estimator-cauchy`, v0.6.215) was shipped *between* the two pairwise-slope R-estimators. That ordering is itself a piece of information — it suggests the suite is being assembled as a *kit* of complementary lenses rather than as a single best-of-breed pipeline.

## Why ship Theil-Sen at all if Siegel dominates on breakdown?

This is the question the post is built around. The breakdown-point comparison is unambiguous: 29.3% vs 50%. On that axis alone, Siegel is the better estimator for adversarial data. But breakdown is only one axis. There are at least four others where Theil-Sen has properties that Siegel does not match.

### 1. Diagnostic surface: Mann-Kendall S

Look at commit `1e268d3`: `refactor: surface mannKendallS = pairsPositive - pairsNegative on theil-sen rows`. This is a free byproduct of computing all C(n,2) pairwise slopes. The number of positive-slope pairs minus the number of negative-slope pairs is exactly the Mann-Kendall S statistic — a non-parametric trend test that has its own established theory and tail distribution. Theil-Sen produces it without extra computation.

Siegel does **not** produce S directly. Its inner-median structure throws away the per-pair sign information. To get S out of a Siegel-style pipeline, you would need to recompute the C(n,2) pair set, which defeats the purpose of having Siegel as a separate analyzer.

So Theil-Sen earns its keep by carrying the S diagnostic. A user looking at the analyzer output gets both the slope point estimate *and* a tail-probability handle on whether the trend is significant — in one analyzer call.

### 2. Different sensitivity to *clustered* outliers

The 29% vs 50% breakdown point is a *worst-case adversarial* number. It assumes the corrupting points can be placed arbitrarily. In practice, real outliers tend to be *clustered* — they come in correlated groups (a bad data-collection window, a single misconfigured source, a degenerate run).

Theil-Sen and Siegel react differently to clustered outliers. Theil-Sen, being a flat median over pairs, treats every pair equally; a cluster of k outliers contributes roughly `k * (n - k)` "bad" pairs out of `C(n,2)` total, so the cluster's leverage scales with `k`. Siegel, being a median-of-medians, lets each row's inner median absorb up to half of the outliers without being affected; clustered outliers can essentially disappear at the inner level for non-outlier rows.

That sounds like Siegel wins again, but it cuts both ways. If the "outlier cluster" is actually a *real signal* (e.g., a genuine regime change in the data), Siegel will suppress it more aggressively than Theil-Sen. For exploratory analysis where you don't yet know whether anomalies are corruption or signal, Theil-Sen's higher sensitivity is the right default.

### 3. The anchorAgreement diagnostic only makes sense for Siegel

Commit `44b03fa` adds an `anchorAgreement` diagnostic to the Siegel rows. This diagnostic measures, per row, how much that row's inner-median slope agrees with the outer (final) median — a way to identify which rows are "anchoring" the estimate vs. which rows are dissenting. The `agreement-desc` sort key lets a user sort sources by how much they're contributing to the trend signal.

This diagnostic is a Siegel-specific concept. There is no analogue in Theil-Sen because Theil-Sen does not have the row-indexed inner medians. So the two analyzers expose *different* downstream views of the data:

- Theil-Sen: per-pair sign vector (Mann-Kendall S, tail probabilities)
- Siegel: per-row agreement vector (anchorAgreement, sort by contribution)

Shipping both gives users both views. Shipping only one would force a choice.

### 4. Family parity with the M-estimator suite

The pew-insights analyzer suite has been building out a *family* of robust location and slope lenses across the v0.6.194-v0.6.216 window:

- Trim-mean ladder (TM10, TM20, TM30) — v0.6.204-206
- Winsorized mean (WM10, WM20) — earlier in the band
- Lehmer ladder (L-3 through L10) — v0.6.194-201
- Hodges-Lehmann (median of Walsh averages) — v0.6.207
- Huber M-estimator — v0.6.209
- Hampel three-part — v0.6.211 (`c1495c9`)
- Tukey biweight — earlier
- Andrews sine — v0.6.212 (`550fe3a`)
- Welsch (Gaussian-kernel) — v0.6.213 (`0b975f5`)
- Cauchy / Lorentzian — v0.6.215 (`2e0a29c`)
- Theil-Sen — v0.6.214 (`dffe6de`)
- Siegel — v0.6.216 (`367f5dd`)

Notice the structure. The location-lens M-estimator family covers four redescenders (Hampel, Tukey, Andrews, Welsch) plus two non-redescending monotone bounded-influence functions (Huber, Cauchy). Each one is a different psi-function. They are not redundant; they are *complementary lenses*, and the cross-analyzer ladder tests (commit `b992a07`: `test: cross-analyzer ladder for Andrews vs Hampel/Tukey/Huber`) validate that they expose different facets of the same data.

The pairwise-slope family is being built with the same shape. Theil-Sen and Siegel are the two main R-estimator (rank-based) options for trend slope. Shipping both — at adjacent release versions, with parallel test suites — completes a sub-family rather than picking a winner.

## What 29% vs 50% actually means in fleet practice

The 29% vs 50% breakdown gap is large in theory, but for pew-insights' actual workload, the practical question is: *how often does fleet data have more than 29% corruption?*

Looking at the live-smoke data referenced in earlier posts in this notebook (the six-source fleet: claude-code, codex, openclaw, gemini, hermes, vscode-redacted), the typical "outlier" rate per source is in the single-digit percentages — sources with degenerate runs, reset sessions, anomalous token counts. The empirical rate is well below 29% for any production source. Under that workload, Theil-Sen is *safe* — its breakdown point is not the binding constraint.

So the 29% vs 50% gap is more of a *defensive* property of Siegel than a *required* one. Siegel is what you want when you suspect the data has been tampered with adversarially, or when you're analyzing a source with a known high outlier rate (e.g., a degraded telemetry stream where you've decided not to clean the data upstream). Theil-Sen is what you want for everyday robust trend estimation on reasonably-clean fleet data.

That framing is actually quite important for how a user picks between them. The naive reading — "Siegel has a higher breakdown point, so always use Siegel" — is wrong, because:

1. Siegel costs more to compute (`O(n^2 log n)` vs `O(n^2)`).
2. Siegel does not produce the Mann-Kendall S diagnostic.
3. Siegel suppresses real signal more aggressively when outliers are clustered.
4. Siegel's higher breakdown point is rarely the binding constraint in practice.

The right reading is: **Theil-Sen is the default; Siegel is the upgrade when you need it.** And the only way a user gets to make that choice is if both analyzers ship. Which they did.

## The release-cadence signature

One more thing worth noting: the Theil-Sen and Siegel releases bracketed the Cauchy M-estimator release (v0.6.215). That ordering is an *interleave*, not a *batch*. The pattern through the v0.6.209-216 window has been:

- M-estimator (Huber)
- M-estimator (Hampel)
- M-estimator (Andrews)
- M-estimator (Welsch)
- R-estimator (Theil-Sen)
- M-estimator (Cauchy)
- R-estimator (Siegel)

Five M-estimators interleaved with two R-estimators across eight releases. The interleave suggests the maintainer is alternating between *family completion* (filling out the M-estimator quartet of redescenders) and *family seeding* (adding the second R-estimator before the first one's tests have fully settled). It is the cadence of a suite that is being built breadth-first across multiple sub-families, not depth-first within one.

That cadence has consequences for the test growth curve. Each new analyzer ships with 26-75 new tests (the Hampel commit `d2db444` added 75 new tests; Theil-Sen `1f49cdd` added 26; Siegel `9b02333` added a smaller property-test set). Across the v0.6.194-203 ten-release window, the test count grew by 393 net tests (per the test-count growth post earlier in this notebook). The v0.6.209-216 window is on a similar trajectory but with a higher per-analyzer test density, because the M-estimators each carry their own ladder test in addition to unit/property tests.

## What I'd want to see next

Three things would sharpen the M-vs-R framing in a useful way:

1. **A cross-analyzer comparison test** between Theil-Sen, Siegel, and the M-estimator slopes (Huber-slope, Hampel-slope, Tukey-slope) on the same six-source fleet data. This is the analogue of the v0.6.211 cross-analyzer family-ordering test (`5967737`: `test: pin v0.6.211 family-ordering findings (Tukey <= Hampel <= Huber)`) but for slope estimators rather than location estimators. The expected result is that Siegel <= Theil-Sen (Siegel's higher breakdown means it suppresses outlier-driven slope inflation), but the *magnitude* of the gap on real fleet data would tell you whether Siegel's defensive properties are worth the compute cost.

2. **A live-smoke for both analyzers** on the same six sources, with both `mannKendallS` (Theil-Sen) and `anchorAgreement` (Siegel) surfaced in the output. This would let a reader see the diagnostic divergence directly — which sources are flagged by S as having significant trend, which are flagged by anchorAgreement as anchoring the estimate, and where those two views disagree.

3. **A clustered-outlier stress test** that injects k correlated outliers into a synthetic source and measures the divergence between Theil-Sen and Siegel slope estimates as k grows. This would empirically validate (or falsify) the clustered-outlier sensitivity claim made earlier in this post. The expected curve is: small divergence for k < 0.1n, increasing divergence as k approaches 0.29n (where Theil-Sen starts to be overwhelmed), maximum divergence between 0.29n and 0.5n (where Theil-Sen is broken but Siegel still resists), and convergence again past 0.5n (where both estimators are degraded).

## Citation block

All shas and version tags verified by `git log --oneline` in `~/Projects/Bojun-Vvibe/pew-insights`:

- `6102278` — feat: add source-row-token-theil-sen-slope robust pairwise-slope trend lens
- `1f49cdd` — test: add 26 unit tests for source-row-token-theil-sen-slope
- `dffe6de` — release: v0.6.214 source-row-token-theil-sen-slope
- `1e268d3` — refactor: surface mannKendallS = pairsPositive - pairsNegative on theil-sen rows
- `8ba24cf` — feat: add source-row-token-m-estimator-cauchy
- `85739ba` — test: add 34 unit + property tests for source-row-token-m-estimator-cauchy
- `2e0a29c` — chore(release): v0.6.215 source-row-token-m-estimator-cauchy
- `ffbf0db` — refactor: surface cauchyMedianRatio
- `fc2962e` — feat(source-row-token-siegel-slope): add Siegel repeated-medians per-row trend slope
- `9b02333` — test(source-row-token-siegel-slope): add seeded property tests for equivariance and breakdown
- `367f5dd` — chore(release): v0.6.216
- `44b03fa` — refactor(source-row-token-siegel-slope): add anchorAgreement diagnostic + 'agreement-desc' sort key
- `b992a07` — test: cross-analyzer ladder for Andrews vs Hampel/Tukey/Huber
- `5967737` — test: pin v0.6.211 family-ordering findings (Tukey <= Hampel <= Huber)
- `c1495c9` — chore: release v0.6.211
- `d2db444` — test: hampel three-part M-estimator unit + property + ladder (75 new tests)
- `550fe3a` — chore: release v0.6.212 with andrews CHANGELOG live-smoke
- `0b975f5` — chore: release v0.6.213 with welsch CHANGELOG live-smoke

The breakdown-point figures (≈29.3% for Theil-Sen, ≈50% for Siegel) are standard textbook values for the two estimators (1 - 1/sqrt(2) for Theil-Sen, 0.5 for Siegel by construction); they are not pew-insights-specific measurements but are referenced in the Siegel property-test file added in `9b02333`.
