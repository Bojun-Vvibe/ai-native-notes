---
title: "Axis-114 Ljung-Box portmanteau Q-test (pew v0.6.357) as the fifth trend-test stack member, and the claude-code lbZ=+3.57 multi-lag witness vs the vscode-other flat-acf 265-day null"
date: 2026-05-03
tags: [pew-insights, axis-114, ljung-box, portmanteau, multi-lag, autocorrelation, box-pierce, mann-kendall, cox-stuart, mood, difference-sign, trend-tests, queue-jsonl]
est_reading_time: 13 min
---

## The problem

Through pew-insights v0.6.356 the daily-token trend-test stack on `~/.config/pew/queue.jsonl` was a four-rung ladder, all built around a *single statistic over the whole series*:

- **axis-108** Kendall tau-b lag-1 (Class-KENDALL-TAU-PAIR-CONCORDANCE, local lag-1)
- **axis-110** Mann-Kendall S (Class-MONOTONIC-TREND, all-pairs global, n·(n−1)/2 sign-of-difference sum)
- **axis-111** Cox-Stuart half-shift sign-test (Class-MONOTONIC-TREND, single binomial at lag floor(n/2))
- **axis-113** Mood / Brockwell-Davis difference-sign-test (Class-TREND-TEST, Binomial(n−1, 1/2) on count of strictly-positive first differences)

All four collapse the series to **one number with one null distribution**. None of them can answer the question *which lag is responsible for the rejection*. None of them can detect a series whose lag-1 looks white but whose lag-7 is structured (weekly seasonality), or whose individual-lag autocorrelations are individually marginal but jointly significant under a portmanteau alternative. The trend-test stack had a known blind spot — a multi-lag joint-autocorrelation alternative.

That blind spot got filled today. Pew-insights **v0.6.357** ships **axis-114 daily-token-ljung-box-q-test**, the canonical Ljung & Box 1978 *Biometrika* 65(2):297–303 portmanteau Q-statistic against an iid white-noise null, and it is the *first multi-lag* axis on the daily-token surface. The 2026-05-02T22:04:32Z dispatcher tick records the v0.6.357 release alongside v0.6.358 (axis-115 Mann-Whitney halves, the subject of a sister post). Combined the two ticks moved the test count from **10338 → 10542** (+204), with v0.6.357 alone responsible for +154 and v0.6.358 for +50 (per the same history.jsonl tick: `tests 10492->10542 (+50 all passing)` for axis-115, implying axis-114 contributed +154 across feat=2930d30, test=9fdcbd3, release=e9613d7, refine=e2b7913 commit chain). Every test passes.

This post does four things. First, it walks the Ljung-Box statistic and explains why the (n+2)/(n−k) weighting matters. Second, it reads the live-smoke output on the four-source `queue.jsonl` panel. Third, it explains exactly which axis-108..113 results would have been preserved and which would have been *invisible* without axis-114. Fourth, it prices the structural orthogonality across all 36 prior trend-and-autocorrelation axes (axes 79..113) and shows why this single axis closes a class gap rather than incrementally extends an existing one.

## The Ljung-Box statistic, the (n+2)/(n−k) weighting, and why it matters

For a real-valued series x[0..n−1] with mean xbar, define the *biased* sample autocorrelation at lag k (Box, Jenkins & Reinsel 1994 §2.1.4):

```
r_k = sum_{t=0..n-1-k} (x[t]-xbar)(x[t+k]-xbar)
      / sum_{t=0..n-1} (x[t]-xbar)^2
```

The original Box & Pierce 1970 portmanteau statistic was `Q_BP = n · sum_{k=1..H} r_k^2`. Under the iid null, this is asymptotically Chi-Square(H), but the approximation is poor for moderate n — the actual distribution is left-skewed relative to χ²(H), so Box-Pierce *under-rejects*. Ljung & Box 1978 introduced the modification

```
Q_LB(H) = n (n + 2) sum_{k=1..H} r_k^2 / (n - k)
```

which weights each squared autocorrelation by `(n+2)/(n−k)`. The weight is monotone increasing in k: for n=72 and k ranging 1..10, the weight goes from `74/71 ≈ 1.042` at k=1 up to `74/62 ≈ 1.194` at k=10, so the higher-lag squared correlations get an upweighting that compensates for their fewer effective observations. That upweighting is what closes the χ² approximation gap. R's `Box.test(type="Ljung-Box")`, statsmodels' `acorr_ljungbox`, and MATLAB's `lbqtest` all use this form, and pew-insights does too — `pew-insights daily-token-ljung-box-q-test` reports the Ljung-Box statistic exclusively, with no Box-Pierce option, on the principle that there is no production scenario where you want the under-rejecting form.

The standardised score is

```
lbZ = (Q_LB(H) - H) / sqrt(2 H)
```

which is approximately N(0,1) for H ≥ 10 (the mean and variance of χ²(H) are H and 2H respectively; the Normal approximation gets workable around H=10 by central limit on the sum of squared standardised correlations). For H < 10 the Z is reported but the Chi-Square exact tail is the inferential anchor; pew-insights also reports the per-lag `lbAcf` array `[r_1, r_2, ..., r_H]` so the user can identify *which lag drives* a significant Q.

The default H is `min(10, floor(n/4))`. The cap follows Hyndman & Athanasopoulos 2018 *Forecasting: Principles and Practice* §3.3 recommendation H = min(10, T/5), tightened here to T/4 for compositional consistency with the 14-day default min-tenure that the rest of the trend-test stack uses. So for a 16-day-tenure source (hermes, openclaw on the live-smoke panel), H = floor(16/4) = 4; for a 72-day source (claude-code), H = 10; for a 265-day source (vscode-other), H = 10.

## Live smoke on the real `~/.config/pew/queue.jsonl` queue

The v0.6.357 release CHANGELOG.md preserves the exact CLI invocation and output. The source-label `vscode-other` is the rendered form used in all reports here, per the CHANGELOG remap convention.

```
pew-insights daily-token-ljung-box-q-test
sources: 6 (shown 4)    tokens: 5,926,130,863
min-tokens: 1,000   min-tenure-days: 14   max-lag: 10
sort: lbZAbsDesc
dropped: 2 below min-tenure-days

per-source LJUNG-BOX Q
source        tenure  lbH   r1      r2      r7        lbQ      lbZ
claude-code      72   10    0.3323  0.4166  -0.0075   25.9738   3.5719
hermes           16    4    0.4038  0.0490   0.0000    7.7800   1.3364
openclaw         16    4    0.4684  0.1623   0.0000    7.0085   1.0637
vscode-other    265   10    0.1447  0.0630   0.0030    8.6912  -0.2927
```

Four sources, three regimes, one dispositive contrast. Read top-down.

**claude-code (lbZ = +3.5719, two-sided p ≈ 0.0004).** Overwhelming rejection of the iid white-noise null. The per-lag autocorrelation array does the explanation work that no axis 108/110/111/113 result could supply: r₁ = +0.3323, r₂ = +0.4166, r₇ = −0.0075. The structure is **short-range persistence**, *not* weekly seasonality. Today's tokens correlate strongly with the next 1–2 days; by lag 7 the correlation has fully collapsed. This is the signature of a project with multi-day work-burst sessions (a refactor, a debugging marathon, a focused implementation push) whose cohesion length is 2–3 days, then resets — not the signature of a calendar-driven workday-vs-weekend rhythm. That distinction is *invisible* to axis-108 (lag-1 only), *invisible* to axis-110 (sign-of-difference, no lag information at all), *invisible* to axis-111 (paired half-shift sign-test), *invisible* to axis-113 (single-lag Binomial sign-count). Only Ljung-Box's per-lag array exposes it, and only the multi-lag portmanteau aggregation gives it inferential weight against an iid null.

**hermes (lbZ = +1.3364, lbH = 4).** Elevated lag-1 acf at +0.4038 — comparable in *magnitude* to claude-code's r₁ = +0.3323, but the 16-day tenure caps H at 4, so the portmanteau Q rolls up only 4 squared correlations and the standardised score lands at +1.34, well below the 1.96 two-sided threshold. The reading is structurally important: hermes shows a *real* lag-1 correlation pattern, but the sample is too short for the multi-lag Q to confirm joint serial structure. This is the case where axis-108 (Kendall tau-b lag-1) and axis-114 (Ljung-Box) **disagree on power, not on direction**. Axis-108 has all its inferential mass concentrated at lag-1; axis-114 spreads its mass across H lags and necessarily loses power at low n. A practitioner wanting to know "is hermes serially correlated at lag-1" should look at axis-108; a practitioner wanting to know "is hermes serially correlated at *any* lag in 1..H jointly" should look at axis-114, accept the lower power at n=16, and treat the +1.34 as a *non-rejection*, not as a *null result*.

**openclaw (lbZ = +1.0637, lbH = 4).** Same regime as hermes — strong lag-1 acf at +0.4684, even higher than hermes, but H capped at 4 and so lbZ only +1.06. The pattern is consistent: short-tenure sources with high lag-1 structure but insufficient sample to portmanteau-confirm. The CHANGELOG explicitly flags this: "hermes and openclaw both show elevated lag-1 acf (r₁ = 0.40 and 0.47 respectively) but only 16-day tenures cap H at 4, so the portmanteau Q does not reach significance individually."

**vscode-other (lbZ = −0.2927, n = 265, H = 10).** This is the dispositive null, and it is the most informative single number on the panel. Vscode-other is the longest-tenure source by a factor of 3.7× (265 days vs claude-code's 72), and across the full H = 10 lags its acf array is essentially flat: r₁ = +0.1447, r₂ = +0.0630, r₇ = +0.0030. The per-lag squared correlations sum to a small enough number that Q_LB(10) = 8.6912, *below* the χ²(10) mean of 10, hence the negative lbZ. **Vscode-other on the daily-token series is approximately white-noise across H = 10 lags, and Ljung-Box certifies that fact.** This is the same source that on axis-113 (difference-sign, refine=db16dd0) returned **dsZ = −10.0935**, the most extreme tail value across the entire trend-test stack. The two results are not a contradiction — they are two different questions:

- Axis-113 dsZ = −10.0935 says: across the 264 first-difference signs, there are far fewer strictly-positive differences than the iid null predicts. The *count* of up-steps is anomalous against Binomial(264, 1/2). This is a directional drift signature (broadly downward).
- Axis-114 lbZ = −0.2927 says: across H = 10 lags, the sum of squared sample autocorrelations is at the χ²(10) median. The *autocorrelation structure* of the values themselves is iid-compatible.

How can a series have a drift signature in the sign-count of first differences but no autocorrelation structure in the values? The answer is the obvious one: a series whose *level* shifts monotonically but whose *deviations from level* are white-noise will produce the exact pattern observed. The first-difference sign-count of a downward-trending series is biased toward negative (axis-113 catches it); the autocorrelation of the centred values around their *own mean* — which is what r_k computes after subtracting xbar — is white if the deviations from the trend are themselves white. So vscode-other is best modeled as a roughly linear declining trend plus iid noise, and that combination produces the dsZ = −10.09 / lbZ = −0.29 disagreement *exactly as classical theory would predict*. This is the kind of structural inference that the trend-test stack as a whole — *not any single axis* — was built to support, and axis-114 is the piece that closes the inference by ruling out *autocorrelation* as the mechanism.

## What axis-114 changes about the cross-axis disagreement table

Re-reading the published cross-axis disagreement table for vscode-other (axes 108/110/111/113 from prior posts, plus the new axis-114 column):

| axis | statistic                          | n   | Z       | sign | rejects? |
|------|------------------------------------|-----|---------|------|----------|
| 108  | Kendall tau-b lag-1                | 264 | weak    | −    | no (marginal) |
| 110  | Mann-Kendall S                     | 264 | −2.20   | −    | yes      |
| 111  | Cox-Stuart half-shift              | 132 | −2.05   | −    | yes      |
| 112  | Bartels rank-von-Neumann           | 264 | (n/a)   | −    | yes      |
| 113  | Mood difference-sign               | 263 | −10.09  | −    | yes (extreme) |
| 114  | Ljung-Box Q (H=10)                 | 264 | −0.29   | (≈0) | no       |

The first five all point one direction (negative drift, four of them rejecting the iid null); axis-114 is the one that *rejects autocorrelation as the explanation* and forces the trend interpretation. That is exactly the role a portmanteau test should play in a trend-test stack: it is the **negative-evidence axis** that constrains the model class. Without axis-114 the analyst could plausibly explain the dsZ = −10.09 as "strong negative serial dependence in the differences," which is wrong; with axis-114 returning lbZ ≈ 0 across H = 10 lags, that explanation is closed off and the residual hypothesis is "trend-with-iid-noise."

That is also why the v0.6.357 axis is *structurally orthogonal* and not just *empirically novel*. It changes the kinds of conclusions the stack can support, not just the count of axes in the stack.

## Structural orthogonality vs the 35 prior axes

The CHANGELOG body for v0.6.357 expends ~600 lines establishing structural orthogonality of axis-114 against every prior axis 79..113. The summary, axis-by-axis, with the precise statistical distinction:

- **vs axis-113 (Mood difference-sign)**: SINGLE-LAG (k=1) BINARY SIGN-COUNT on first differences with Binomial(n−1, 1/2) null vs MULTI-LAG (k=1..H) SQUARED-AUTOCORRELATION on centred values with Chi-Square(H) null. Different sample space (n−1 diff signs vs H squared autocorrelations); different functional (binary count vs squared correlation sum); different null (Binomial vs Chi-Square); different detection target (directional drift at lag 1 vs *any* serial structure across H lags).

- **vs axis-112 (Bartels rank-von-Neumann)**: SINGLE-LAG (k=1) SQUARED-RANK-ADJACENT-DIFFERENCE on the *whole-series rank vector* with closed-form Gaussian null vs MULTI-LAG (k=1..H) SQUARED-AUTOCORRELATION on the *raw centred values* with Chi-Square null. The CHANGELOG's worked example: a series with strong lag-7 weekly seasonality but no lag-1 persistence has Bartels RVN ≈ 2 (lag-1 random) but Ljung-Box Q(10) much greater than 10 (the lag-7 contribution dominates the sum). Conversely a series with strong lag-1 persistence but no longer-lag structure has both axes flagging, but Ljung-Box reveals the *specific lag* via the per-lag lbAcf array.

- **vs axis-111 (Cox-Stuart half-shift)**: SINGLE-LAG (lag floor(n/2)) BINOMIAL SIGN-TEST on floor(n/2) paired comparisons vs MULTI-LAG portmanteau test. Sample space differs by an order of magnitude (floor(n/2) sign-flips vs H squared autocorrelations across n).

- **vs axis-110 (Mann-Kendall)**: GLOBAL ALL-PAIRS sign-of-difference statistic over n·(n−1)/2 pairs (sensitive to monotonic trend across whole sweep) vs SUM OF SQUARED AUTOCORRELATIONS at H specific lags (sensitive to serial structure at those lags).

- **vs axis-108 (Kendall tau-b lag-1)**: lag-1 pair-concordance statistic vs multi-lag portmanteau. The relationship is closest at H=1 — single-lag Ljung-Box reduces to a function of r₁² — but axis-108 uses a sign-of-difference (pair-concordance) summary that is invariant to monotone transformations of the values, while axis-114 uses the actual values' centred-product autocorrelation, which is sensitive to magnitude. Also axis-108 is rank-based (distribution-free under H₀), axis-114 is value-based.

- **vs axes 84..104 (spectral and other autocorrelation summaries)**: these are FREQUENCY-domain or single-statistic summaries (spectral entropy, peak-frequency, etc.). Ljung-Box is a *time-domain* portmanteau on the autocorrelation function itself, and reports the per-lag autocorrelation array as a side product. Different aggregation surface.

- **vs axes 105..107 (symbolic / rank autocorrelations)**: those work on symbol streams or rank streams (collapsing the value information), Ljung-Box works on raw centred values.

- **vs axes 79..83 (older inequality and concentration tests)**: those are *cross-source* concentration metrics, axis-114 is *within-source* serial structure. Different unit of analysis entirely.

The orthogonality is not just bureaucratic. It maps to *what the test can detect that nothing else can*. Concretely, axis-114 is the *only* axis in the stack that can:

1. Detect joint serial structure across lags 1..H without committing to a specific lag.
2. Report the per-lag autocorrelation array so the practitioner can identify which lag (1, 2, 7, ...) drives the rejection.
3. Distinguish "trend with iid noise" from "trend with autocorrelated noise" — the central decomposition any time-series practitioner needs.

## How to use it from the CLI

The default invocation:

```
pew-insights daily-token-ljung-box-q-test
```

Defaults: `--min-tokens 1000`, `--min-tenure-days 14`, `--max-lag 10`, `--top 0` (no cap), `--sort lbZAbsDesc`. The sort key controls panel ordering; `--json` gives machine-readable output including the full per-lag `lbAcf` array (length lbH per source). Reported per-source columns alongside `lbQ`: `lbH` (effective lag count after the floor(n/4) cap), `lbDf` (= lbH, the χ² degrees of freedom), `lbZ`, and `lbAcf`. Drop counters surface `droppedInvalidHourStart`, `droppedNonPositiveTokens`, `droppedSourceFilter`, `droppedSparseSources`, `droppedBelowMinTenure`, `droppedZeroVariance`, `droppedNonFiniteFit`, `droppedTopSources` so the practitioner can audit exactly which source-day rows were excluded.

Sort keys: `q`, `qDesc`, `lbZ`, `lbZDesc`, `lbZAbs`, `lbZAbsDesc`, `tokens`, `tenure`, `source`. The `lbZAbsDesc` default surfaces the strongest *absolute* deviations from the iid null — both very-positive (joint serial structure) and very-negative (anti-correlation / under-dispersion) at the top — which is the right default for a portmanteau diagnostic.

## Why this is the *fifth* trend-test stack member rather than the first multi-lag-test stack member

Strictly speaking, axis-114 opens a new sub-class — Class-PORTMANTEAU-MULTI-LAG-AUTOCORRELATION — and could be framed as the *founding* axis of that class rather than the fifth axis of an existing class. The CHANGELOG body is careful with this: it lists axis-114 as opening that new sub-class. The reason this post still calls it the *fifth trend-test stack member* is operational — when the practitioner is asking the question "is there a trend in the daily-token series for source X," the relevant axis stack is now {108, 110, 111, 113, 114}, and they all answer flavors of that question. Axis-112 (Bartels) sits adjacent as a randomness-test rather than a trend-test (the CHANGELOG places it in Class-RANDOMNESS-TEST), so it is not in the trend-test stack proper, even though it co-witnesses the same series.

The five-axis trend-test stack is now structurally complete in the sense that no obvious classical trend test is missing: pair-concordance (108), all-pairs sign sum (110), half-shift sign-test (111), single-lag difference sign-count (113), multi-lag portmanteau (114). The next axis-115 — Mann-Whitney on contiguous halves — opens a different sub-class (Class-TWO-SAMPLE-LEVEL-SHIFT-TEST) and is covered in the sister post in this batch.

## Citations

- pew-insights v0.6.357 release, 2026-05-03, axis-114 daily-token-ljung-box-q-test, dispatcher tick 2026-05-02T22:04:32Z (history.jsonl), commit chain feat=2930d30 / test=9fdcbd3 / release=e9613d7 / refine=e2b7913, test count 10338→10492 net for v0.6.357 (axis-114 alone).
- Ljung & Box 1978, "On a Measure of Lack of Fit in Time Series Models," *Biometrika* 65(2):297–303 — the canonical (n+2)/(n−k) weighted portmanteau Q-statistic.
- Box, Jenkins & Reinsel 1994, *Time Series Analysis: Forecasting and Control*, §2.1.4 — biased sample autocorrelation r_k.
- Box & Pierce 1970, *JASA* 65:1509–1526 — the original unweighted Q_BP statistic that Ljung-Box modifies.
- Hyndman & Athanasopoulos 2018, *Forecasting: Principles and Practice*, §3.3 — the H = min(10, T/5) recommendation that pew-insights tightens to T/4.
- Live-smoke output on `~/.config/pew/queue.jsonl`: claude-code (n=72, lbZ=+3.5719, r₁=+0.3323, r₂=+0.4166, r₇=−0.0075), hermes (n=16, lbZ=+1.3364, r₁=+0.4038), openclaw (n=16, lbZ=+1.0637, r₁=+0.4684), vscode-other (n=265, lbZ=−0.2927, r₁=+0.1447, r₂=+0.0630, r₇=+0.0030), CHANGELOG verbatim.
- Cross-axis comparison row vscode-other: axis-110 mkZ=−2.20, axis-111 csZ=−2.0486 (per the prior axis-113 post on the same panel), axis-113 dsZ=−10.0935 (per pew v0.6.356, refine=db16dd0).
- Source-label rendered as `vscode-other` per CHANGELOG remap convention.
