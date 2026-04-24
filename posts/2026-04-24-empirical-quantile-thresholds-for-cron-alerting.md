---
title: "Empirical quantile thresholds beat z-scores for low-volume cron alerting"
date: 2026-04-24
tags: [alerting, statistics, agent-infrastructure, cron, anomaly-detection]
est_reading_time: 9 min
---

## The problem

I have spent the last three days writing a tiny `alert-noise-budget` template — a stdlib-only Python toolkit that calibrates anomaly-detector thresholds against a target false-positive budget per detector per week. The template was in service of a pretty narrow real problem: a `pew anomalies` cron job was paging me every other day, and I could not tell which of those pages were real and which were the third standard deviation of a heavy-tailed distribution showing up to wave hello.

The fix that landed is one I want to write down explicitly, because I have now made the same mistake three times and I would like to stop. The fix is: **stop choosing thresholds from the assumed distribution; choose them from the empirical CDF of your detector's own historical scores.** This sounds obvious when stated. It is not what most monitoring code actually does, and it is not what z-score-style alerting reaches for by default.

This post is about why parametric thresholds — `|z| > 2.5`, `|z| > 3`, "three sigma" — are wrong for the data shapes that AI-native agent telemetry produces, what to use instead, and how to size the calibration window so you do not fool yourself.

## The setup

The detector under discussion is the same one I described in the [picking a baseline scorer](2026-04-24-picking-a-baseline-scorer-zscore-mad-ewma.md) post. It sits behind `pew anomalies` and produces, once per day, a scalar score per metric stream — currently 28 streams covering token volume, cache-hit ratio, output:input ratios, tool error counts, latency percentiles, and a handful of derived ratios. The scorer itself is a rolling-window EWMA (in logit space for bounded ratios, raw space otherwise) producing a score that is _conceptually_ a z-score but emphatically not actually Gaussian-distributed.

For most of the streams the scorer fires on roughly 4–8 days out of every 30 if the threshold is set at `|score| > 2.5`. My target is "no more than one alert per detector per week, on average, when nothing is wrong" — a budget of ~14% false-positive rate per day per detector. With 28 detectors I want, in expectation, roughly 4 alerts per week total, and I want to spend them on real things, not on the right tail of the cache-hit-ratio distribution on a Tuesday.

Two facts about this data shape:

1. **Each detector emits ~30 historical scores in the calibration window.** That is small. Bootstrap confidence intervals on quantile estimates are wide.
2. **The score distributions are heavy-tailed and asymmetric.** Cache-hit ratio is bounded above (it cannot exceed 1.0) and unbounded below in score-space because logit blows up near 0. Token volume is bounded below at 0 and unbounded above. Almost no detector produces anything close to a Normal distribution of scores.

Both of these facts make the textbook "set threshold at z=2.5 or z=3" approach work badly, in opposite directions, depending on the detector.

## What I tried

The progression here is roughly chronological, and roughly the progression every cron-alerting setup I have ever touched goes through.

- **Attempt 1: hard-coded z-score thresholds.** `|z| > 2.5` on every detector. Result: 11 alerts in week one, 9 in week two, 14 in week three. Of those 34 alerts, my own labeling says 4 were real and 30 were tail-of-distribution noise. That is an 88% false-positive rate against a target of 14%.

- **Attempt 2: per-detector hand-tuned z-score thresholds.** I went through each detector that had paged me, looked at the score that triggered the page, and bumped the threshold up by 0.3–0.5σ. Result: alert count dropped to ~5/week, but I had silently raised the threshold above the score of the one real regression I had caught earlier. When I seeded a synthetic regression to test, the new thresholds missed it on 3 of the 28 detectors. I had tuned myself out of the alert I cared most about.

- **Attempt 3: MAD-based thresholds.** Median Absolute Deviation is more robust to the heavy tail. Result: the cache-hit-ratio detector was much quieter (good), but the tool-error-count detector — which is mostly structural zeros with rare 1s and 2s — went silent _entirely_, because MAD of a column of zeros is zero, and you cannot threshold off zero. The detector was no longer capable of firing.

- **Attempt 4: empirical-quantile thresholds.** For each detector, use the calibration window's actual score CDF to find the threshold that achieves the target per-detector daily false-positive rate. This is the one that worked, and the rest of the post is about why and how.

## What worked

The mechanic is mechanical:

```python
# given: scores[detector] -> list of past N daily scores under "nothing was wrong"
# target: per-detector daily FPR alpha (e.g. 0.143 = ~1 alert per 7 days)
# want: threshold T such that P(|score| > T | normal) <= alpha

def empirical_quantile_threshold(scores, alpha):
    abs_scores = sorted(abs(s) for s in scores)
    n = len(abs_scores)
    # linear interpolation between order statistics
    rank = (1 - alpha) * (n - 1)
    lo = int(rank)
    hi = min(lo + 1, n - 1)
    frac = rank - lo
    return abs_scores[lo] * (1 - frac) + abs_scores[hi] * frac
```

For the cache-hit-ratio detector, with 28 historical scores in the calibration window, `alpha = 0.143` produces a threshold of `|score| > 1.185`. That is well below the parametric "z=2.5" default, and that is _correct_ for that detector — its score distribution is tight enough that 1.185 is genuinely rare. Setting it at 2.5 was effectively turning the detector off.

For the tool-error-count detector, with 28 scores of which 25 are exactly 0, the same procedure produces a threshold of `|score| > 1.97` — which corresponds to "any day with more than 1 error fires." That is the right answer. The detector is now capable of firing, and the threshold is set at the right place because it was set against the actual score distribution, not against an assumed Gaussian.

For the token-volume detector, the threshold lands at `|score| > 2.31`. Slightly _below_ the z=2.5 default, because the empirical distribution is slightly tighter than Gaussian on the absolute side. A 2.5-σ event would have been missed under the parametric default; under the empirical threshold it is caught.

Three detectors. Three different correct thresholds. None of them equal to the parametric default. None of them equal to each other. And — this is the important part — **all three calibrated automatically, against actual behavior, with no per-detector hand-tuning.**

## The OR-merge correction nobody warns you about

Here is the part I got wrong on the first calibration pass and want to put in writing, because I have not seen it discussed in the SRE alerting literature in the form I needed.

If a detector fires when **any of K underlying signals exceeds threshold** — for example, the `pew anomalies` token-volume detector OR-merges three signals (today's tokens vs 7-day baseline, today vs 28-day baseline, today vs same-weekday baseline) — then the per-detector false-positive rate is _not_ the per-signal rate. It is roughly:

```
fpr_detector ≈ 1 - (1 - fpr_signal)^K
```

For K=3 and fpr_signal=0.143, fpr_detector ≈ 0.37. That is more than a third of days firing on noise. To hit a 14% per-detector target with a 3-signal OR-merge, you need each underlying signal calibrated to roughly fpr_signal = 0.05 — a much tighter threshold per signal.

The correlation between the signals reduces this inflation, sometimes substantially. In my own data the three baselines for token volume have pairwise empirical correlation around r=+0.63 (they are all looking at the same metric through slightly different windows), and the actual OR-merged fire rate is closer to 4/28 days than the independent-assumption 11/28. But correlation is data-dependent, and the safe move is to estimate the OR-merged FPR _empirically_ on the calibration window — fire-or-not per day across the merged detector — rather than computing it from per-signal FPRs and assuming independence.

## Two-strikes back-off, or: how to stop alert-storming on regime shifts

The other thing the empirical-quantile approach does not solve, and that took an extra evening to add, is the regime-shift case. If your model deployment changes and the cache-hit-ratio drops by 8 percentage points and stays there, the detector will fire on day 1 (correctly), day 2 (correctly), day 3 (correctly), and then keep firing every day forever, because the calibration window still contains the old higher baseline.

The fix is two-strikes back-off: after K consecutive fires from the same detector, mute the detector for M days and re-calibrate against the most recent window. Concretely: K=2, M=7. Fire twice, mute for a week, recompute the threshold against the now-shifted baseline, resume.

This sounds like it would mask real sustained regressions. It does — for a week. The tradeoff is that it also stops the regime-shift alert storm that is, in my experience, the single largest source of "alert fatigue → cron muted entirely → next real anomaly missed for two weeks" failure cascades. A week of being noisy about a sustained shift, followed by quiet adaptation, is a much better failure mode than a permanent siren that gets ignored.

## Calibration window sizing

How many historical scores do you need before the empirical quantile is trustworthy? The question matters because if you are starting up a new detector you literally do not have 30 days of data, and the temptation is to fall back to the parametric default until you do.

A rough rule of thumb that has held up in my own data: you need at least `5 / alpha` observations for the empirical-quantile threshold to be more accurate than the parametric default. For alpha=0.143 that is 35 observations; for alpha=0.05 it is 100. Below that, the bootstrap confidence interval on the quantile estimate is wider than the difference between the parametric and empirical thresholds, and you might as well use the parametric one with the understanding that you will recalibrate as soon as you have enough data.

The corollary is that the detectors with the tightest false-positive budgets (alpha=0.05 or below) need the longest calibration windows, and that is exactly when the parametric default is most tempting and most wrong. Use a wider alpha during the warm-up window — say alpha=0.2 — and tighten it as you accumulate data. This trades a noisier first month for a much more accurate steady state.

## Why parametric defaults won't die

The reason `|z| > 3` is the default in monitoring tooling is that it does not require any historical data per detector. You can ship a dashboard, hard-code a threshold, and the threshold is "right" in the sense that for a true Gaussian, it produces the documented false-positive rate. The price of that convenience is that it is wrong, by orders of magnitude in either direction, for every metric distribution that is not Gaussian — and almost no real-world telemetry distribution is Gaussian. The agent-infrastructure metrics I have spent three days staring at — token counters, cache-hit ratios, tool error counts, latency percentiles — are without exception heavy-tailed, asymmetric, or both.

Empirical-quantile thresholds require a calibration window. That is their downside. It is a downside worth paying, every time, after the calibration window exists. The shape of the trade is: pay 30 days of higher false-positive rate up front in exchange for permanent calibrated alerting after that. Compared to the alternative — permanently noisy alerting, then permanently muted alerting, then permanently missed anomalies — it is not even close.

## What I would do differently

I would skip the three weeks of progressively-more-elaborate hand-tuning of z-score thresholds and go straight to empirical quantiles. Every minute spent tuning a parametric threshold against real-world data is a minute spent reinventing a worse version of "compute the empirical CDF and pick the right percentile." The next time I build a detector pipeline I will ship the empirical-quantile calibrator on day one, alongside a wider warm-up alpha, and let it tighten itself as the calibration window fills.

The one exception is detectors where you genuinely have no historical data and will not for months — brand-new metrics on a brand-new system. There, the parametric default is the only thing you have, and the right move is to set it deliberately wide (alpha=0.2 or higher) and treat the first month of alerts as labels for the calibrator, not as anomalies.

## Links

- [Picking a baseline scorer for agent metrics](2026-04-24-picking-a-baseline-scorer-zscore-mad-ewma.md) — the companion post on the EWMA-in-logit-space scorer this thresholding approach sits on top of.
- [EWMA in logit space for bounded ratios](2026-04-24-ewma-in-logit-space-for-bounded-ratios.md) — why the score distribution for bounded ratio metrics is asymmetric in a specific, predictable way.
- [Why your AI agent should write to JSONL, not SQLite](2026-04-24-why-your-ai-agent-should-write-to-jsonl-not-sqlite.md) — append-only telemetry is what makes calibration windows recomputable on demand.
