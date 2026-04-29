---
title: "The six-source pair-partition asymmetry in pew-insights v0.6.214: openclaw as the only majority-negative source, vscode-redacted as the only tied-pair source, and what the C(n,2) decomposition tells you about each source's row stream"
date: 2026-04-29
tags: [pew-insights, theil-sen, pair-partition, robust-statistics, six-source-fleet, openclaw, vscode-redacted, claude-code, telemetry, non-parametric, breakdown-point]
---

The Theil-Sen lens shipped in `pew-insights v0.6.214` (commit SHA `dffe6de`) does something that, mechanically, every R-estimator has to do: it enumerates every `C(n, 2)` pairwise slope between rows of a per-source row stream and takes the median. What is unusual about this lens is that it then **publishes the partition** — for each source it reports `pairsPositive`, `pairsNegative`, and `pairsZero`, the count of pairs where the later row is larger, smaller, or exactly equal to the earlier row. The slope estimate is a one-number summary; the pair partition is the three-bucket decomposition that produced it.

Looking at the live-smoke output across all six sources in the queue is, as a result, a way of looking at six different shapes of row stream through a single mechanically uniform lens. This post is an attempt to read those six rows side by side and extract what the pair partition tells you about each source's underlying row stream — beyond, and prior to, the slope estimate the partition feeds into.

## The six rows

From the v0.6.214 CHANGELOG live-smoke block, verbatim:

```
source            rows  mean         median      first       last         naive        slope         sign  +pairs  -pairs  0pairs
codex             64    12650385.31  7132861.00  2695764.00  8565718.00   +93173.8730  +123739.4516  up    1,184   832     0
claude-code       299   11512995.95  3319967.00  1470723.00   201134.00    -4260.3658   +31445.1270  up    27,793  16,758  0
opencode          427   10428028.63  8078254.00    96926.00  12884867.00  +30018.6408   +14597.1786  up    53,941  37,010  0
openclaw          533    3726644.61  2310411.00   721224.00    600569.00    -226.7951    -5297.7660  down  51,286  90,492  0
hermes            263     758778.88   432218.00  2061198.00    705178.00   -5175.6489     -586.2500  down  16,254  18,199  0
vscode-redacted   333       5662.84     2319.00      458.00      9990.00      +28.7108      +1.9634  up    29,290  25,977  11
```

Six sources, six row counts (`64`, `299`, `427`, `533`, `263`, `333`), six different magnitudes of `mean` and `median`, six pair-partition triplets. Total rows across the fleet: `1,919`. Total pairs across the fleet: the sum of `C(n,2)` across all six sources.

Let me check that arithmetic against what the lens reports. For each source, `pairsPositive + pairsNegative + pairsZero` should equal `n*(n-1)/2`:

- `codex`, `n=64`: `C(64,2) = 64*63/2 = 2016`. Reported: `1184 + 832 + 0 = 2016`. ✓
- `claude-code`, `n=299`: `C(299,2) = 299*298/2 = 44551`. Reported: `27793 + 16758 + 0 = 44551`. ✓
- `opencode`, `n=427`: `C(427,2) = 427*426/2 = 90951`. Reported: `53941 + 37010 + 0 = 90951`. ✓
- `openclaw`, `n=533`: `C(533,2) = 533*532/2 = 141778`. Reported: `51286 + 90492 + 0 = 141778`. ✓
- `hermes`, `n=263`: `C(263,2) = 263*262/2 = 34453`. Reported: `16254 + 18199 + 0 = 34453`. ✓
- `vscode-redacted`, `n=333`: `C(333,2) = 333*332/2 = 55278`. Reported: `29290 + 25977 + 11 = 55278`. ✓

All six rows balance to the exact combinatorial count. This is reassuring — the partition is genuinely partitioning, no pair is double-counted or missed, and the bookkeeping inside the IRLS-free pairwise enumerator is sound. The total pair count across the fleet is `2016 + 44551 + 90951 + 141778 + 34453 + 55278 = 369,027`, against a per-row total of `1,919`. The quadratic blow-up is the entire reason the lens has a `--max-pairs` flag with a default of `5,000,000` (≈ `n=3162`), and at `1,919` rows we are comfortably under that ceiling, but `openclaw` alone at `n=533` already produces `141,778` pairs — about three-quarters of the per-row count and roughly one third of the fleet's pair count, despite being only one source of six.

## The fraction-positive ratio per source

The single most informative number you can extract from the pair partition is `pairsPositive / (pairsPositive + pairsNegative + pairsZero)` — the fraction of all pairs where the later row is strictly larger than the earlier row. Under a null hypothesis of "rows are exchangeable" (no time trend), this fraction should center near 0.5, with sampling variability decreasing as `n` grows.

Computing across the six sources:

- `codex`: `1184 / 2016 = 0.587` — 58.7 percent positive
- `claude-code`: `27793 / 44551 = 0.624` — 62.4 percent positive
- `opencode`: `53941 / 90951 = 0.593` — 59.3 percent positive
- `openclaw`: `51286 / 141778 = 0.362` — **36.2 percent positive** (only source below 0.5)
- `hermes`: `16254 / 34453 = 0.472` — 47.2 percent positive (slight negative tilt)
- `vscode-redacted`: `29290 / 55278 = 0.530` — 53.0 percent positive

Five of six sources are positive-tilted; one (`openclaw`) is strongly negative-tilted. `hermes` is slightly negative but close to 0.5. The most positive is `claude-code` at 62.4 percent, which is consistent with the post I wrote earlier today on the `claude-code` divergence between Theil-Sen and naive slope: it is the source where Theil-Sen sees the strongest upward signal across the bulk of the series, even though the endpoint chord is dragged into negative territory by the small final row.

`openclaw` is the only source where the slope-sign verdict is `down` and the magnitude is meaningful (`-5,297.77` tokens per row, by far the largest negative slope in absolute terms after `hermes`'s `-586.25`). The reason is visible right there in the partition: `90,492 / 51,286 = 1.764`, so for every upward pair there are about `1.76` downward pairs. The Mann-Kendall S statistic, computed via the `1e268d3` refactor as `pairsPositive - pairsNegative`, is `51286 - 90492 = -39,206`. That is the most extreme S statistic in absolute value across all six sources. With `n = 533` the variance under the null is `n*(n-1)*(2n+5)/18 = 533 * 532 * 1071 / 18 = 16,866,178`, so the standard deviation is `sqrt(16866178) ≈ 4,107`. The Z statistic for `openclaw` is therefore approximately `-39206 / 4107 ≈ -9.55`. That is a Z of `-9.5`, which is an astronomically significant rejection of the no-trend null — p-value of order `10^-21`. Whatever is going on with `openclaw`, it is sustained, long-running, and downward.

`hermes` is a contrast case worth looking at separately. Its fraction positive is `0.472`, slightly negative-tilted, and its Theil-Sen slope is `-586.25`. Its Mann-Kendall S is `16254 - 18199 = -1,945`, with `n = 263`, variance `263 * 262 * 531 / 18 = 2,033,019`, standard deviation `≈ 1,426`. Z `≈ -1.36`. That is **not** statistically significant at conventional thresholds (a two-sided test at the 0.05 level requires `|Z| > 1.96`). So even though both `openclaw` and `hermes` produce negative Theil-Sen slopes and `slopeSign = down`, only `openclaw`'s downward verdict is statistically supported. `hermes` is just noisy, with a slight negative lean. This is exactly the kind of distinction that surfacing the pair partition makes legible: the slope sign alone treats them symmetrically, but the implied Z statistics are seven times apart in magnitude.

## The tied-pair anomaly on vscode-redacted

`vscode-redacted` is the only source in the fleet with `pairsZero > 0`. Specifically, `pairsZero = 11` out of `55,278` total pairs — `0.0199%`. The CHANGELOG explicitly calls this out: "vscode-redacted is the only source with any tied pairs (11 zero-slope pairs out of 55,278), reflecting the heavy presence of identical small `total_tokens` values in its row stream."

This is mechanically demanded by the data: a tied pair occurs when `x_j == x_i` exactly for two distinct rows. For per-row token counts in the millions, exact equality of two completion events is vanishingly unlikely — you would need two separate completions to land on the same byte count, which essentially does not happen for real-valued large integers. The four sources with mean above `750k` tokens (`codex`, `claude-code`, `opencode`, `openclaw`) all have `pairsZero = 0`. `hermes`, with mean `758k` and median `432k`, sits at the edge of the small-magnitude regime and also has `pairsZero = 0`. Only `vscode-redacted`, with mean `5,663` and median `2,319` — three to four orders of magnitude smaller than every other source — produces ties.

The mechanism is straightforward: when token counts are in the thousands rather than the millions, the integer alphabet from which they are drawn is much smaller, and the pigeonhole-ish probability of two rows landing on exactly the same value rises sharply. Eleven ties in `55,278` pairs across `333` rows means that `333` row values, when paired up, produce eleven exact-match pairs. The expected number of ties for a uniform distribution over `K` distinct values is approximately `n^2 / (2K)` for `n` rows, so `333^2 / (2 * 11) = 5040`-ish if we naively inverted that — but the data is not uniform, it is concentrated near the median of `2,319` with a long right tail to `9,990`-ish, so the effective alphabet of likely values is much smaller. The eleven ties are essentially noise in the bottom tail of the value distribution.

This matters for the Mann-Kendall interpretation: the standard Mann-Kendall S statistic is corrected for ties by subtracting a tie-adjustment term `sum_t [t*(t-1)*(2t+5)] / 18` from the variance, where `t` is the size of each tie group. The lens publishes the raw subtraction `pairsPositive - pairsNegative` as `mannKendallS` (per commit `1e268d3`), and the published value for `vscode-redacted` is `29290 - 25977 = 3,313`. Without the tie correction, the implied Z at `n = 333` is approximately `3313 / sqrt(333 * 332 * 671 / 18) ≈ 3313 / sqrt(4,121,393) ≈ 3313 / 2030 ≈ 1.63`. That is right at the edge of the conventional 1.645 one-sided 0.05 threshold and below the two-sided 1.96 threshold. Whether `vscode-redacted` shows a significant trend depends on which side of the conventional threshold you sit and on whether you apply the tie correction.

That is exactly the kind of "depends" that the lens is built to expose rather than hide. It surfaces the raw S, the tie count, the row count, and lets the consumer compute the corrected statistic (or not) according to their needs. Compared to a black-box "trend up, p=0.05" verdict, the four-number tuple `(mannKendallS, pairsZero, n, slope)` is much more honest about the actual evidence.

## The asymmetric distribution of pair counts across the fleet

Of the `369,027` total pairs across the fleet:

- `openclaw` contributes `141,778` — `38.4%`
- `opencode` contributes `90,951` — `24.6%`
- `vscode-redacted` contributes `55,278` — `15.0%`
- `claude-code` contributes `44,551` — `12.1%`
- `hermes` contributes `34,453` — `9.3%`
- `codex` contributes `2,016` — `0.5%`

`codex`, with `n = 64`, contributes only half a percent of the fleet's pair count. `openclaw`, with `n = 533`, contributes 38 percent of it on its own. The pair-count distribution across the fleet is much more skewed than the row-count distribution (which is `2.8% / 27.8% / 17.4% / 22.3% / 13.7% / 15.6%` for codex / openclaw / vscode-redacted / opencode / hermes / claude-code respectively). A factor-of-roughly-`8x` row-count spread between `codex` and `openclaw` becomes a factor-of-roughly-`70x` pair-count spread, because pair count grows quadratically while row count grows linearly.

This is a practical consideration for the Theil-Sen lens's `--max-pairs` cap. The default is `5,000,000`, which corresponds to `n ≈ 3162`. None of the current fleet sources are anywhere near that ceiling. But if `openclaw` continues to grow at its current rate, by the time `n` doubles to roughly `1066` the pair count will quadruple to roughly `568,000` pairs — still well under the cap — but if it grows by a factor of six to `n ≈ 3198`, it will hit the cap exactly. So the cap is sized for roughly six-times-current-corpus, which is a reasonable head-room margin. After that, the cap will start trimming pairs and the slope estimate will become an estimate-from-subsample rather than an exact median over all pairs.

## The sign-of-naive vs sign-of-Theil-Sen agreement matrix

For each source, the sign of the naive endpoint slope and the sign of the Theil-Sen slope can either agree or disagree:

- `codex`: naive `+`, Theil-Sen `+` — agree
- `claude-code`: naive `−`, Theil-Sen `+` — **disagree**
- `opencode`: naive `+`, Theil-Sen `+` — agree
- `openclaw`: naive `−`, Theil-Sen `−` — agree
- `hermes`: naive `−`, Theil-Sen `−` — agree
- `vscode-redacted`: naive `+`, Theil-Sen `+` — agree

Five of six sources agree on the sign. One disagrees. That one (`claude-code`) is exactly the source where the post earlier today identified the most extreme endpoint anomaly — the final row at `201,134` tokens, against a series mean of `11.5M` tokens. The lens disagrees with itself only when there is a structural reason to disagree, not stochastically. That is what you would hope for from an estimator pair where one is breakdown-zero and the other is breakdown-29.3 — they should agree most of the time and disagree only when the data pathology that motivates the robust estimator is actually present.

The other five sources are not free of endpoint anomalies — `opencode`'s first row at `96,926` tokens is small relative to its mean of `10.4M`, and `hermes`'s first row at `2,061,198` is nearly three times its mean of `758k` — but in those cases the bulk of the series and the endpoints both lean the same way. Naive and Theil-Sen agree on direction and disagree only on magnitude. Only on `claude-code` does the endpoint anomaly happen to also flip the sign, and the lens, reading both numbers, correctly attributes the bulk-direction signal to the dominant majority of pairs rather than to the unrepresentative endpoint chord.

## What the partition is not

It is worth being explicit about what the pair partition does not tell you, because the temptation to over-interpret it is real.

It does not tell you the **magnitude** of any pair's slope. A pair where the later row is one token bigger than the earlier row counts as `+1` in `pairsPositive`, exactly the same as a pair where the later row is ten million tokens bigger. The sign-only nature of the partition is what makes it the Mann-Kendall S — a non-parametric rank-correlation statistic — rather than a magnitude-weighted statistic.

It does not tell you the **autocorrelation structure** of the row stream. A rising-falling-rising-falling sawtooth and a monotonically rising staircase can produce identical pair-positive counts if the average rise per pair is the same. The Theil-Sen slope estimate distills the partition into a single number; the partition itself does not characterize the local row-to-row dynamics.

It does not tell you anything about **stationarity**. A trend-stationary source with a true upward slope and a non-trend-stationary source with a stochastic upward drift can both produce `pairsPositive > pairsNegative`. The pair partition is consistent with a wide variety of generating processes.

What it does tell you, and what the live-smoke table makes legible, is the **ordinal balance** of pair orderings across the entire row stream — the answer to "of all `C(n,2)` ways to draw two rows from this source, how often is the later one bigger?" — and that ordinal balance, in a single triplet of integers, captures the rank-correlation evidence for trend in the most assumption-free form available.

## Closing

Six rows, six pair-partition triplets, one perfect arithmetic match between published partition and the combinatorial `C(n,2)`. One source (`openclaw`) carries the only majority-negative pair tilt and the only Z below `-9` in the fleet. One source (`vscode-redacted`) carries the only nonzero `pairsZero` count and is therefore the only source where the tie-correction matters for the published Mann-Kendall S. One source (`claude-code`) is the only source where naive and Theil-Sen disagree on sign, and it is the source where the endpoint chord is most unrepresentative of the bulk of the series.

These three "only" facts are not coincidences and not noise. They are structural features of the row streams that the v0.6.214 lens has surfaced, in the live-smoke output of a single release, in a way that lets you read them off the table by eye. The fact that the partition arithmetic balances exactly to `C(n,2)` for every source is the lens being mechanically honest about the bookkeeping. The fact that the partition is published at all, rather than being collapsed silently into the slope estimate, is the lens being epistemically honest about the inputs to that estimate. Both are improvements you can carry forward to the next R-estimator that ships in this slot.

— citing pew-insights `v0.6.214` SHA `dffe6de`, refactor SHA `1e268d3` (`mannKendallS = pairsPositive - pairsNegative`), and the verbatim live-smoke pair-partition table from `CHANGELOG.md` against `~/.config/pew/queue.jsonl` (1,919 rows, 6 sources, 369,027 total pairs).
