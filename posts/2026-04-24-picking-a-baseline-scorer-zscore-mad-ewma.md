# Picking a baseline scorer for agent metrics: z-score vs MAD vs EWMA

When you start putting cron jobs in front of an LLM agent — daily token-usage rollups, per-tool error-rate snapshots, latency histograms — the first interesting question is not _which metrics to track_. It's _what counts as anomalous_.

The default answer most people reach for is "send me a Slack ping if the value is more than 2 standard deviations from the rolling mean." That answer is wrong about 60% of the time, in my own data. Not catastrophically wrong — it just produces a steady drip of false positives on heavy-tailed metrics, and silently misses real regressions on metrics with structural zeros. After spending a few days writing a small rolling-window scorer module to sit behind a `pew anomalies` subcommand and paging through the failures, I have a stronger opinion than I started with.

This post is the lessons-learned: three scorers (z-score, MAD, EWMA), the data shapes each one is good at, the data shapes each one fails on, and a decision rubric I now actually use instead of guessing.

## The toy setup

To make this concrete, every example below comes from a tiny test harness — a 21-test unittest suite I wrote against three scorer implementations in `metric-baseline-rolling-window`. The scorers are all stdlib-only Python, no numpy, no pandas, no scipy. About 200 lines total. The point is not the implementation; the point is what each scorer does when you feed it the kind of metric a coding agent actually produces.

The three scorers, in one paragraph each:

- **Z-score over a rolling window**: take the last N observations, compute mean and stdev, return `(x - mean) / stdev`. Threshold at `|score| > 2.5` or wherever your tolerance lives. Symmetric, parametric, assumes the underlying distribution is roughly Gaussian.
- **MAD (Median Absolute Deviation)**: take the last N observations, compute the median, then compute the median of `|x_i - median|`. Multiply by 1.4826 to get a robust stdev estimator. Score is `(x - median) / (1.4826 * MAD)`. Robust to outliers in the window. Thresholds tend to live around `|score| > 3.5`.
- **EWMA (Exponentially Weighted Moving Average)**: weight recent observations more than older ones via a smoothing factor `alpha`. Track both the EWMA mean and EWMA variance. Score is `(x - ewma_mean) / sqrt(ewma_var)`. Adapts to slow drift without flagging it as anomalous.

All three give you a single scalar score per observation. None of them tell you _why_ the value is weird. That's deliberate — the scorer is a triage filter, not a root-cause tool.

## The metrics that broke each scorer

I ran each scorer over six sample metric streams. Three are clean (the scorer should mostly stay quiet); three have real anomalies seeded in (the scorer should flag them). The streams:

1. **`daily_tokens_used`** — a smooth-ish counter, mean ~3.2M/day, stdev ~600k. Real anomaly seeded: a 14M-token day on day 18 (a runaway agent loop).
2. **`per_call_latency_ms`** — heavy-tailed. P50 ~800ms, P99 ~9000ms. Real anomaly seeded: a sustained shift to P50 ~3500ms starting day 12 (a regional model deployment regression).
3. **`tool_error_count`** — a count metric with structural zeros. Most days are exactly 0 errors. A handful of days have 1–2 errors. Real anomaly seeded: 47 errors on day 22 (a tool schema change).
4. **`pr_reviews_per_day`** — bimodal. Weekdays ~6, weekends ~0. No anomalies — should stay quiet on the weekly cycle.
5. **`cache_hit_ratio`** — bounded in [0, 1], typically 0.78–0.92. Slow drift downward over the window from 0.88 to 0.81 (a model swap that changed the prompt distribution). Should stay quiet — drift, not anomaly.
6. **`agent_session_minutes`** — long-tailed but stable. P50 ~12, P95 ~85. Real anomaly seeded: a 4-hour session on day 25 (I forgot to close a tmux pane).

Now the failure modes.

### Z-score on heavy tails: the false-positive treadmill

Z-score on `per_call_latency_ms` produced 11 alerts in a 30-day window, of which exactly **one** corresponded to the real seeded regression. The other 10 were ordinary tail observations — calls that took 12s instead of 9s, scoring 2.7σ above the rolling mean. These are not anomalies. They are what the distribution does on a normal Tuesday.

The mechanism is straightforward: z-score's denominator is the stdev of the window, and stdev is itself blown up by the tail. So you end up with a scorer that says "this point is 2.7σ above the mean" while quietly telling you the mean and stdev were estimated on a sample where 5% of points were already 3σ outliers. The math is consistent. The interpretation — "anomaly" — is not.

Z-score also missed the seeded regression for the first six days, because the regression was a sustained shift, not a single spike. Once the new high-latency days were inside the rolling window, they pulled the mean up and the stdev wider, and the regression stopped looking like a regression. By day 18 the score was back to ~1.6σ — well below threshold. The scorer had absorbed the regression into its baseline.

This is the textbook failure mode of a parametric scorer on heavy-tailed metrics: it normalises drift _into_ the baseline, and over-flags ordinary tail observations as outside it.

### MAD on the same metric: quieter, slower, more honest

MAD on `per_call_latency_ms` produced 2 alerts in the same window. One was the seeded regression (caught on day 12, the day of the shift). The other was a single 47-second outlier on day 7 that turned out to be a real timeout I had forgotten about — so arguably a true positive too.

The reason MAD did better here is mechanical: the median of `|x_i - median|` is unaffected by the size of the tail. If 5% of your window is at 12s and 95% is at 800ms, the median is still 800ms and the MAD is still small. A real shift to 3500ms median moves the median, immediately, and the score is large.

MAD's weakness is the converse: when the underlying distribution _is_ approximately Gaussian, MAD is less efficient than the stdev. You need a slightly bigger effect size to cross threshold. In the `agent_session_minutes` test, MAD caught the 4-hour session at score 4.1 (above the 3.5 threshold), while z-score caught it at 5.8 (further above its 2.5 threshold). Both fired, MAD fired with less margin. On a clean unimodal distribution, z-score wins.

But coding-agent metrics are almost never clean unimodal distributions. Latency is heavy-tailed. Token counts have weekend dips. Error counts have structural zeros. The cases where z-score's efficiency advantage matters are rare. The cases where its noise dominates are common.

### Both classical scorers on `tool_error_count`: catastrophic

This was the most embarrassing failure. `tool_error_count` is mostly zeros: 22 of 30 days have 0 errors, 6 days have 1 error, 1 day has 2 errors. The seeded anomaly is 47 errors on day 22.

Z-score on this stream: the rolling stdev is dominated by the seeded 47-error day once it lands in the window, so the score for the seeded day itself is "only" 4.5σ — fired, fine. But for the next 19 days, every 1-error day scores at 2.1σ–2.4σ (one error is a long way from a mean that's been pulled up to ~2.3 by the spike). False positives every other day for three weeks.

MAD is worse. The median of a window that's mostly zeros is zero. The MAD of `|x_i - 0|` for a window that's mostly zeros is also zero — and now the denominator is zero, the score is undefined (or infinity, depending on whether your code checks). Every nonzero observation is "infinitely anomalous." Every zero observation is "exactly normal." The scorer is providing no information at all.

The fix here is not parameter tuning. It's recognising that the metric has a fundamentally different generative model — most days the process emits zero, occasionally it emits a Poisson-ish small count, very occasionally it spikes — and using a scorer that understands that. In the implementation I added a "zero-aware" variant of MAD that excludes zeros from the median/MAD computation, and treats the zero-rate itself as a separate signal. The seeded anomaly fires cleanly. The 1-error days do not.

The general lesson is broader than just MAD: **a scorer is making an assumption about the data-generating process, whether you stated it or not.** Z-score assumes approximately Gaussian. MAD assumes approximately symmetric and continuous. Neither is true for count metrics with structural zeros. Pick a scorer whose assumptions match the metric, or transform the metric until they do.

### EWMA on the drift case: the only one that doesn't cry wolf

`cache_hit_ratio` drifted slowly from 0.88 to 0.81 over 30 days. This is not an anomaly. It's a real but expected change in the underlying process (a model swap changing what gets cached). I do not want to be paged for it.

Z-score: after the drift had moved through about half the window, the score for "current value vs rolling mean" hovered at 1.8σ–2.3σ, occasionally crossing 2.5σ on a noisy day. Five false alerts over the window.

MAD: similar story. The median tracks the drift slowly, and current observations score around 2.5–3.2 — sometimes over the 3.5 threshold, sometimes under. Three false alerts.

EWMA with `alpha = 0.1`: the EWMA mean moved with the drift. Scores stayed under 1.5 the entire window. Zero false alerts. When I then injected a true spike — a `cache_hit_ratio` of 0.42 on day 27 — EWMA scored it at 6.2 and fired immediately.

This is exactly what EWMA is for: when the metric has a slow non-stationary baseline that you don't want to flag, but you still want to catch sudden departures _from_ that drifting baseline. The cost is the smoothing factor `alpha` — pick it too high (close to 1) and EWMA degenerates to "last observation," with no smoothing; pick it too low (close to 0) and EWMA never adapts to real shifts and flags everything for weeks. Alpha tuning is the EWMA tax.

### EWMA's own failure mode: the cold start

EWMA is initialised with the first observation, or with a prior. For the first ~`1/alpha` observations, the scorer is essentially flying blind — its variance estimate is unstable, its mean is dominated by initial conditions. In testing, the first 10 observations under `alpha = 0.1` produced scores that varied wildly (one observation scored 18 in a window where nothing was anomalous, because the EWMA variance happened to be near zero on that step).

The mitigation is a warm-up period: don't emit scores for the first `2/alpha` observations. The implementation does this. It costs you the first 20 days of new metric streams, but it prevents the cold-start chaos from dominating early alerts.

## The decision rubric

After running these tests, the rubric I now use looks like this. It is short on purpose.

**1. What does the metric look like?**

- _Smooth-ish, roughly symmetric, no structural zeros, stable baseline_ → **z-score**.
- _Heavy-tailed (P99 / P50 > 5) but otherwise stationary_ → **MAD**.
- _Slow drift in the baseline that you don't want to alert on_ → **EWMA**.
- _Mostly-zero count metric with occasional bursts_ → **zero-aware MAD**, or skip statistical scoring entirely and use a fixed threshold (e.g. "alert if `count > 10`").
- _Bounded in `[0, 1]` (ratios, hit rates)_ → transform to logit space first, then EWMA. Untransformed ratios near 0 or 1 produce misleading variance estimates.

**2. How fast does the underlying process change?**

- _Process is genuinely stationary over weeks_ → any scorer with a 14–28 day window works.
- _Process has known weekly seasonality (weekend dips)_ → either subtract the seasonal component first, or use a scorer keyed on weekday (separate baselines for Mon/Tue/.../Sun).
- _Process drifts on a timescale of days_ → EWMA with `alpha` tuned so the EWMA half-life matches the drift timescale.
- _Process changes every time you deploy_ → reset the baseline on deploy. Don't try to make the scorer absorb deploy-induced shifts.

**3. What's the cost of a false positive vs a false negative?**

- _False positives are cheap (a Slack message you ignore)_ → use a tighter threshold, accept more noise, catch more real issues.
- _False positives are expensive (paging a human, blocking a deploy)_ → use a wider threshold, run two scorers in agreement (z-score AND MAD both fired), or add a minimum-duration requirement (anomaly must persist for N consecutive observations).

**4. Have you actually validated the scorer on this metric, on real data?**

This is the one most people skip and it's the one that matters most. None of the rules above survive contact with a metric whose distribution you assumed wrong. The fix is not theoretical: feed each candidate scorer 30 days of historical data, manually label the anomalies, count how many each scorer catches and misses. Pick the one with the best precision/recall tradeoff for that specific stream. It takes an hour per metric. It is worth the hour every time.

## What I changed in my own setup

Before this exercise, I had a single z-score scorer with a 14-day window and a `|score| > 2.5` threshold sitting behind every metric I cared about. After this exercise:

- `daily_tokens_used`, `pr_reviews_per_day` (after weekday adjustment): **z-score**, 28-day window, `|score| > 3.0`. Threshold widened because the previous version was firing weekly on Mondays.
- `per_call_latency_ms`, `agent_session_minutes`: **MAD**, 21-day window, `|score| > 3.5`. Caught 100% of the seeded regressions in testing; cut alerts on this stream from ~3/week to ~1/month.
- `tool_error_count`: zero-aware MAD with a fallback fixed threshold of 10. The statistical scorer alone is not enough on count metrics; the fixed threshold is the safety net.
- `cache_hit_ratio`, `prompt_cache_hit_ratio`: logit-transformed **EWMA**, `alpha = 0.1`, score threshold 4.0, 20-observation warm-up. Zero false alerts on the slow drift; catches step changes within one day.
- `agent_session_minutes` already covered above; the long-running tmux session on day 25 fires a separate "session_open_too_long" check that doesn't go through the statistical scorer at all — some checks are just thresholds, and pretending otherwise creates false sophistication.

The migration from one-size-fits-all z-score to per-metric scorers took about three hours. Total alert volume dropped by roughly 70%. The alerts I do get are now overwhelmingly real.

## Three things I'd want to know before I started

If I were doing this again from scratch, three pieces of advice would have saved me time.

**First: don't pick the scorer first. Pick the metric, look at 30 days of it, and only then pick the scorer.** The shape of the data tells you which scorer fits. The default of "z-score because it's familiar" wastes weeks of false-positive triage.

**Second: count metrics with structural zeros are not a special case. They are the most common case in agent metrics.** Tool errors, MCP server crashes, retries, rate-limit hits, pre-push guardrail blocks — all of these are mostly-zero processes. Plan for them up front. Don't assume your statistical scorer will handle them and discover otherwise in production.

**Third: the scorer is the cheap part. The hard part is the labelled validation set.** Building three scorers and a 21-test unittest suite took less than a day. Hand-labelling 60 days of historical data across six metrics — deciding for each "spike" whether it was a real issue or a normal tail observation — took longer. But without that labelled set, you cannot compare scorers, so you cannot pick one. Budget for the labelling. It is the actual work.

The scorer code lives in [`ai-native-workflow/templates/metric-baseline-rolling-window/`](https://github.com/Bojun-Vvibe/ai-native-workflow/tree/main/templates/metric-baseline-rolling-window). It is small, stdlib-only, deliberately unclever. The decision rubric above is more important than the implementation. You can rewrite the implementation in your own language in an afternoon. You cannot rewrite the decision rubric without first making all the mistakes I described.
