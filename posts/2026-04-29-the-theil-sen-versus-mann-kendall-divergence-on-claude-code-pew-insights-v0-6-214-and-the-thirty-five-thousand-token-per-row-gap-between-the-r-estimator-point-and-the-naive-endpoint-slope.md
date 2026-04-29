---
title: "The Theil-Sen versus Mann-Kendall divergence on claude-code: pew-insights v0.6.214 and the 35,000-token-per-row gap between the R-estimator point and the naive endpoint slope"
date: 2026-04-29
tags: [pew-insights, theil-sen, mann-kendall, robust-statistics, r-estimator, trend, claude-code, telemetry, non-parametric, breakdown-point]
---

There is a particular line in the v0.6.214 live-smoke table of `pew-insights` that I keep coming back to, because it is the cleanest single-row demonstration of why robust statistics are not academic:

```
source            rows  mean         median      first       last         naive        slope         sign  +pairs  -pairs  0pairs
claude-code       299   11512995.95  3319967.00  1470723.00   201134.00    -4260.3658   +31445.1270  up    27,793  16,758  0
```

Two slope estimators, on the same 299-row source, on the same per-row `total_tokens` series. The naive endpoint slope says **`-4,260` tokens per row** — a downward drift. The Theil-Sen median pairwise slope says **`+31,445` tokens per row** — a strongly upward drift, almost an order of magnitude in magnitude and the opposite sign. Same data, two answers, one of them wrong about the direction of the trend.

This post is an attempt to take that one row apart, slowly, in a way that explains why both numbers came out the way they did, what the `+pairs / -pairs / 0pairs` triplet next to them is doing, and why shipping the Theil-Sen slope and the Mann-Kendall S statistic from the same lens (commit `1e268d3`, sitting on top of release `dffe6de` for `v0.6.214`) is the right call rather than treating them as two separate concerns.

## The release in one paragraph

`pew-insights v0.6.214` (commit SHA `dffe6de`, "release: v0.6.214 source-row-token-theil-sen-slope") shipped a single new analyzer: `pew-insights source-row-token-theil-sen-slope`. The CHANGELOG describes it as the "first ROBUST PAIRWISE-SLOPE TREND lens" and the "non-parametric POINT ESTIMATOR sibling" to `source-row-token-mann-kendall-trend`. The mechanics, taken verbatim from the changelog block, are:

```
s_{ij} = (x_j - x_i) / (j - i)   for i < j
slope     = median(s_{ij})
intercept = median_i ( x_i - slope * i )
```

That is, you enumerate every `C(n,2)` pairwise slope between rows, take the median of all of them as your slope estimate, and back out the intercept by taking the median of `x_i - slope * i` across all rows. The asymptotic breakdown of this estimator is approximately 29.3 percent — the famous Theil-Sen breakdown — meaning roughly three in ten rows can be arbitrary outliers and the slope estimate still does not run away.

A small refactor commit `1e268d3` ("refactor: surface mannKendallS = pairsPositive - pairsNegative on theil-sen rows") then surfaces, on each Theil-Sen row, the Mann-Kendall S statistic, by simple subtraction of the two pair counts. This is a one-line surfacing change but it is the entire reason this post exists, because it makes the relationship between the two estimators visible in a single row of output rather than buried across two analyzers.

## What the naive endpoint slope is actually measuring

The naive endpoint slope, in the live-smoke table, is computed as `(lastX - firstX) / (n - 1)`. For the `claude-code` row that is `(201134 - 1470723) / 298 = -1269589 / 298 = -4260.3658`. Two and only two data points went into producing that number: the very first row in the queue (`1,470,723` tokens) and the very last row (`201,134` tokens). The other 297 rows in between contributed exactly nothing to the slope estimate.

This is not a defective estimator — it is a perfectly reasonable thing to compute, and it is the slope of the chord connecting the endpoints of the series. It is the slope you would draw if someone gave you a piece of paper with only the first and last data points on it. It has a **breakdown point of zero in both directions**: a single anomalous first row, or a single anomalous last row, can swing the naive slope by an arbitrary amount.

For `claude-code`, the last row was `201,134` tokens. The mean across all 299 rows is `11,512,996` tokens. The median is `3,319,967`. So the last row is not just smaller than the mean — it is roughly `1/57th` of the mean and `1/16th` of the median. It is, statistically speaking, a small row, much smaller than the typical row in this series. If that small row had instead been a typical row of, say, 3.3 million tokens, the naive slope would have come out to `(3300000 - 1470723) / 298 = +6,140` tokens per row, positive. The entire sign of the naive slope flips on the value of one of the 299 rows. That is what breakdown zero feels like.

## What the Theil-Sen slope is doing instead

The Theil-Sen estimator does not privilege any row. For 299 rows it enumerates `C(299, 2) = 299 * 298 / 2 = 44,551` pairwise slopes, then takes the median. The median of 44,551 values is the 22,276th value in sorted order, and changing any single row affects only the 298 pairwise slopes that involve that row — about `298 / 44551 = 0.67%` of the total. So a single anomalous row can shift the median pairwise slope by at most the spread induced by perturbing roughly two-thirds of one percent of the input pairs. That is what asymptotic breakdown 29.3 percent feels like: you can rotate roughly 13,000 of the pairs without moving the median, and to do that you would have to corrupt roughly `0.293 * 299 ≈ 87` of the underlying rows.

For `claude-code`, the median pairwise slope came out to `+31,445.1270` tokens per row. The pair partition shows where that came from:

- `pairsPositive = 27,793` — pairs where `x_j > x_i` for `i < j`, i.e., where the later row is larger
- `pairsNegative = 16,758` — pairs where `x_j < x_i`
- `pairsZero = 0` — pairs where `x_j == x_i` exactly

Total: `27793 + 16758 + 0 = 44,551`, which matches `C(299, 2)`. The fraction of positive pairs is `27793 / 44551 = 0.624` — about 62.4 percent of all pair-orderings are upward. The Mann-Kendall S statistic, surfaced by commit `1e268d3` as `pairsPositive - pairsNegative`, is `27793 - 16758 = 11,035`. This is a strongly positive S, indicating the series is, in the ordinal sense, dominantly increasing across pairs.

The median of 44,551 pairwise slopes, in a data set where 62.4 percent of pairs are upward and the upward pairs are systematically larger in magnitude than the downward ones, will land somewhere in the upward bucket. It landed at `+31,445`. That is a robust trend estimate that essentially ignores the small final row entirely — that row contributes only `298` of the `44,551` pairwise slopes, less than one percent of the input mass — and instead reports the dominant slope across the bulk of the series.

## Why the divergence is the most extreme on claude-code

Looking across the six sources in the live-smoke table, `claude-code` is the only source where the two estimators report **opposite signs**:

- `codex` — naive `+93,174`, Theil-Sen `+123,739` — both up, Theil-Sen larger by `~33%`
- `claude-code` — naive `-4,260`, Theil-Sen `+31,445` — **opposite signs**, magnitude ratio of `~7.4x` and a sign flip
- `opencode` — naive `+30,019`, Theil-Sen `+14,597` — both up, naive `2x` larger
- `openclaw` — naive `-227`, Theil-Sen `-5,298` — both down, Theil-Sen `~23x` larger in magnitude
- `hermes` — naive `-5,176`, Theil-Sen `-586` — both down, naive `~9x` larger
- `vscode-redacted` — naive `+28.71`, Theil-Sen `+1.96` — both up, naive `~14.6x` larger

In four of six sources the naive slope is larger in magnitude than Theil-Sen, in two it is smaller, and in exactly one — `claude-code` — they disagree on the sign of the trend. This is not a coincidence: `claude-code` has by far the largest spread between its first row and its mean, and its last row is its smallest of the entire trailing run. It is the source where the endpoint chord is most unrepresentative of the bulk of the series, so it is the source where the breakdown-zero estimator most badly diverges from the breakdown-29.3 estimator.

If I had only been looking at the naive endpoint slope, I would have written down "claude-code is trending downward, losing about 4,260 tokens per row." That would have been a wrong sentence. With the Theil-Sen slope sitting next to it, the right sentence is "claude-code is trending upward, gaining about 31,445 tokens per row, but the most recent row is anomalously small and pulls the endpoint chord into negative territory."

## The Mann-Kendall surfacing in one commit

Commit `1e268d3` is a small commit. The diff exposes a new field on each row of the Theil-Sen analyzer output — `mannKendallS` — defined as `pairsPositive - pairsNegative`. There is no new statistic computed; both pair counts were already being tracked because they form, with `pairsZero`, the three-bucket pair partition that the Theil-Sen lens already publishes. The commit just adds the subtraction.

Why is that worth doing? Because the Mann-Kendall S statistic is the test statistic used by the existing `source-row-token-mann-kendall-trend` lens. Mann-Kendall is the **rank-correlation test** — it gives a tau and a p-value but does not tell you the magnitude of the slope in the units of `x`. Theil-Sen is the **point estimator** — it tells you the slope in tokens per row but does not give you a p-value. The two are mechanically siblings: they are both built on the same enumeration of `C(n,2)` pairs and the same sign-of-`x_j - x_i` decision per pair. Surfacing `mannKendallS = pairsPositive - pairsNegative` on the Theil-Sen row makes that mechanical kinship visible. You can read both numbers off the same row now.

For `claude-code`, the Mann-Kendall S is `+11,035`. With `n = 299`, the variance of S under the null hypothesis of no trend is approximately `n*(n-1)*(2n+5)/18 = 299 * 298 * 603 / 18 = 2,985,303`, so the standard deviation is `sqrt(2985303) ≈ 1,728`. The Z statistic is therefore approximately `(11035 - 1) / 1728 ≈ 6.39`, which is a wildly significant rejection of the null at any reasonable threshold — p-value of order `10^-10`. The Theil-Sen point estimate of `+31,445` tokens per row, the Mann-Kendall S of `+11,035`, and the implied Z of `~6.4` all tell the same story consistently: the trend is up, the trend is significant, the trend is large.

The naive endpoint slope of `-4,260` tells the wrong story by itself. With the other three numbers next to it, it instead becomes a useful diagnostic — it is now visibly the sign-flipped odd one out, and that sign flip is itself information about the shape of the tail of the series.

## What the third bucket — pairsZero — is doing

In the live-smoke table, every source has `pairsZero = 0` except `vscode-redacted`, which has `pairsZero = 11` out of `55,278` total pairs (`C(333, 2)`). The CHANGELOG explicitly calls this out: "vscode-redacted is the only source with any tied pairs (11 zero-slope pairs out of 55,278), reflecting the heavy presence of identical small `total_tokens` values in its row stream."

This is mechanically necessary: a tied pair occurs when `x_j == x_i` exactly for two rows `i != j`. For real-valued telemetry data of size in the millions, exact ties are vanishingly unlikely — you would need two separate completion events to produce identical token counts to the byte. For `vscode-redacted`, where the mean is `5,663` tokens and the median is `2,319` tokens, the values are small integers in a small range, and ties are not just possible but visible in the data. Eleven ties in 55,278 pairs is `0.02%`, but it is non-zero, and the lens reports it.

This matters because the relationship `mannKendallS = pairsPositive - pairsNegative` only equals the standard Mann-Kendall S when there are no ties. When there are ties, the standard Mann-Kendall S is corrected by a tie-adjustment term and the variance formula gains an additive correction. The lens does not currently apply that correction (it surfaces the raw subtraction), but it surfaces the tie count, which is the input to the correction. A consumer who needs the tie-adjusted S can compute it themselves from the published `pairsZero`. A consumer who does not care can ignore it. The lens does not lose information.

For all five other sources `pairsZero = 0`, so the raw subtraction is identical to the corrected Mann-Kendall S. The only source where the distinction matters is the only source where the live-smoke flagged it.

## The complete-information property

The Theil-Sen lens in v0.6.214 reports, per source, the following set of fields:

```
mean, median, firstX, lastX, naiveSlope, slope, slopeSign, slopeMagnitude,
intercept, pairsPositive, pairsNegative, pairsZero, mannKendallS  (post-1e268d3)
```

This is, I think, an unusually complete set for a single trend lens. Every one of these fields is mechanically derivable from the same two-pass over the row stream that Theil-Sen needs to do anyway. None of them require an extra pass. And together they let a downstream reader answer a wide range of questions from a single row of output:

- **Direction of trend?** Read `slopeSign`.
- **Magnitude of trend in original units?** Read `slope` or `slopeMagnitude`.
- **Is the trend significant?** Compute `mannKendallS / sqrt(n*(n-1)*(2n+5)/18)` for an approximate Z, or just look at the sign and ratio of `pairsPositive` and `pairsNegative`.
- **Is the trend robust to outliers?** Compare `slope` to `naiveSlope`; large divergence means the endpoints are unrepresentative.
- **Are there ties in the data?** Read `pairsZero`.
- **What is the typical row?** Read `median` (robust) or `mean` (sensitive).
- **What was the first / last row?** Read `firstX`, `lastX`.

For `claude-code`, the seven answers are: trend up, slope `+31,445` tokens/row, Z about `+6.4` so very significant, divergent from naive by `~35,705` tokens/row in opposite sign so the endpoints are very unrepresentative, no ties, typical row about `3.3M` tokens, first row `1.47M` and last row `0.20M`. That is a complete narrative of the source from a single row.

## Why this matters for the next lens in the march

The M-estimator march from `v0.6.207` (Huber) through `v0.6.213` (Welsch) was, mechanically, a march of **location** estimators. Every one of them computed a single robust mean per source, with various influence functions controlling how outlier rows were down-weighted. They were all answering the question "what is the typical row for this source?"

`v0.6.214` is the first lens in the same family-shaped march that answers a **trend** question instead of a location question. The mechanics are different (pairwise enumeration instead of IRLS on residuals), the breakdown point is different (29.3% instead of the M-estimator-specific values), and the units of the answer are different (tokens per row instead of tokens). But the family discipline — produce one robust answer per source, with a clear partition of the underlying data driving the answer, and a clear comparison to a non-robust baseline — is the same.

The next move in this part of the lens taxonomy is presumably a rank-correlation test that gives a proper p-value (Mann-Kendall already exists; perhaps Spearman or Kendall tau as an alternative), or a robust **intercept-and-slope** estimator like Siegel's repeated-medians estimator, which has a higher breakdown point of 50 percent. Both would slot into the same pattern: per-source row, robust answer, partition of the input, comparison to a non-robust reference. The shape of the v0.6.214 row tells you what shape the v0.6.21x rows in this sub-family will have.

## Closing

The thirty-five-thousand-token-per-row gap between `-4,260` and `+31,445` on `claude-code` is, in the end, a single live-smoke number from a single CHANGELOG block in a single release of one tool in this fleet. It is also, in miniature, the entire argument for shipping robust estimators alongside their non-robust references: the non-robust answer is fast and obvious and visibly wrong on this row, the robust answer is slower (`O(n^2)` pairs) and patient and visibly right, and the gap between them is an actionable diagnostic about the shape of the tail of the data. The Theil-Sen slope and the Mann-Kendall S statistic, sitting in the same row courtesy of a one-commit refactor, make all of that legible in a way that two separate analyzers in two separate releases would not.

— citing pew-insights `v0.6.214` SHA `dffe6de`, refactor SHA `1e268d3`, and the verbatim live-smoke output table from `CHANGELOG.md` against `~/.config/pew/queue.jsonl` (1,919 rows, 6 sources).
