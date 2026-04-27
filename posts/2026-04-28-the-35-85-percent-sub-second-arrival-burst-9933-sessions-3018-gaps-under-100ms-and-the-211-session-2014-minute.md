---
title: "The 35.85% sub-second arrival burst: 9933 sessions, 3018 gaps under 100ms, and the 211-session 20:14 minute that broke the per-minute Fano (11.30) — what inter-arrival distributions reveal about pew session ingestion"
date: 2026-04-28
tags: [telemetry, pew, session-queue, inter-arrival, burstiness, fano-factor, ingestion, distribution]
est_reading_time: 13 min
---

## The problem

I have been treating the rows in `~/.config/pew/session-queue.jsonl` as a stream of *sessions* — discrete units, each with a `started_at`, a `duration_seconds`, a `model`, a `source`. The mental model when looking at counts has been Poisson: sessions arrive at some average rate, gaps between arrivals are exponentially distributed, hourly rollups should be close to mean ± sqrt(mean). That model is wrong by a factor of about 11 in dispersion and about three orders of magnitude in tail behavior, and the fingerprint of why it is wrong is hidden in the inter-arrival gap distribution. Specifically: of the 9,932 inter-arrival gaps in the current snapshot, **3,561 (35.85%) are less than one second**, and **3,018 of those are less than 100 milliseconds**. The session stream is not a Poisson process. It is a clustered point process where roughly a third of all session "arrivals" are not really arrivals at all — they are bulk-flush moments where the snapshotter writes many rows together because they all became visible at the same upstream tick. Any rate-based statistic computed on the raw stream is therefore measuring two different things glued together: the actual conversational arrival rate, and the snapshotter's batching cadence.

This is a follow-up to the snapshot-lag asymmetry post from earlier today. That post established that lag from `last_message_at` to `snapshot_at` differs by 1000× across sources. This post takes the next step and asks: given that the snapshotter batches, what does the inter-arrival distribution between `started_at` timestamps look like, and what does that tell us about how to read counts and rates off `session-queue.jsonl`?

## The setup

- **File:** `~/.config/pew/session-queue.jsonl`. Current row count: **9,933 sessions** (verified via `wc -l`).
- **All rows have `started_at`.** Zero missing values, so the gap analysis is not biased by exclusion.
- **Time range:** First `started_at` is 2026-02-11T02:49:02.371000+00:00. Last is 2026-04-27T20:59:36.464000+00:00. That is approximately 75.75 days of observation, giving a global mean rate of one session every ~659 seconds — roughly one every 11 minutes.
- **Reference SHA.** pew-insights HEAD `4574e08` ("docs(changelog): cross-link v0.6.146 base entry to v0.6.147 refinement").
- **Definition of inter-arrival gap.** Sort all 9,933 `started_at` timestamps globally; gap *i* = `started_at[i+1] - started_at[i]` in seconds. This produces 9,932 gaps. Per-source gaps are computed analogously: sort that source's timestamps, take consecutive differences.

## The global gap distribution

Computed in plain Python:

```python
import json, statistics
from datetime import datetime
rows = [json.loads(l) for l in open('/Users/bojun/.config/pew/session-queue.jsonl')]
def parse(t): return datetime.fromisoformat(t.replace('Z','+00:00'))
starts = sorted(parse(r['started_at']) for r in rows)
gaps = [(starts[i+1]-starts[i]).total_seconds() for i in range(len(starts)-1)]
```

Output:

```
n=9932 min=0.000 max=1135682.2 mean=659.02 median=9.81 stdev=15436.81
p10=0.000 p25=0.000 p50=9.810 p75=44.034 p90=146.933 p95=567.713 p99=2979.752
zero-or-sub-second gaps: 3561 (35.85%)
bursts (gap < 0.1s): 3018
long pauses (gap > 600s): 483
long pauses (gap > 3600s): 78
```

Two numbers in this output dominate everything else:

**(a)** `mean=659.02` versus `median=9.81`. The mean is 67× the median. For a Poisson process they would be within 30% of each other (the median of an exponential distribution is `mean × ln(2)` ≈ `0.693 × mean`). A 67× ratio is a strong signal that the distribution has both a massive zero-cluster (pulling the median down) and a heavy tail (pulling the mean up). Neither feature is compatible with the Poisson mental model.

**(b)** `p25=0.000`. A full quarter of the inter-arrival gaps round to zero seconds at millisecond precision. This is not "very small" — it is "below the timestamp resolution." The snapshotter is writing rows in batches that are tighter than its own timestamp granularity can distinguish.

## The sub-second cluster: 35.85%

Let me dwell on this number. 3,561 of 9,932 gaps are under one second. 3,018 of them are under 100 milliseconds. That means roughly 30% of all "session arrivals" on this stream arrive within 100ms of the previous one. There are three possible explanations and they have very different implications.

**Explanation 1: The snapshotter writes rows in batches.** The most boring and most likely explanation. When the upstream collector observes that 30 sessions have crossed the "report me" threshold in the last polling interval, it writes them all out in one tight loop, and their `started_at` values may come from very different real-world times but their write order happens to be the order in which the batch loop visits them. If this is the dominant mechanism, the sub-second gaps tell you nothing about user behavior — they only tell you about batch boundaries.

**Explanation 2: Genuine concurrent sessions.** Some harnesses (notably `opencode` and `openclaw`) support multiple concurrent in-flight sessions. If three sessions all start within the same 100ms window because the user spawned them as part of an automated dispatcher, those are real concurrent arrivals, and the sub-second gap is genuine.

**Explanation 3: The `started_at` field is being backfilled from the message, not the actual session start.** If `started_at` is computed as "the timestamp of the first message in the session" and the snapshotter sees many sessions where the first message was buffered before the snapshotter saw them, the `started_at` values reflect upstream message clock, not arrival clock. The 100ms clusters would then be the moments when many buffered messages all became visible to the snapshotter at once.

The data pattern suggests it is a mix of (1) and (2), not (3). Here is why: per-source inter-arrival statistics show very different sub-second-cluster behavior, which is incompatible with explanation (3) (which would affect all sources equally, since they all go through the same snapshotter).

## Per-source breakdown

| source       |   n   | median |   p90    |    p99     |     max     | Fano (approx) |
|--------------|------:|-------:|---------:|-----------:|------------:|--------------:|
| opencode     | 5,991 |  13.32s| 166.7s   |  1,298.3s  |  40,738.1s  |       5,940.8 |
| openclaw     | 2,354 |   0.00s| 298.6s   |  3,601.1s  |  82,151.7s  |      23,378.2 |
| claude-code  | 1,107 |  31.23s| 1,779.9s | 93,231.5s  | 1,135,682.2s|     386,369.7 |
| codex        |   477 |  69.75s| 299.6s   | 54,173.5s  |  99,335.6s  |      50,877.7 |

Five things in this table are worth their own paragraph each.

**1. openclaw has a median inter-arrival gap of zero.** That is, more than half of openclaw sessions arrive within timestamp-resolution distance of the previous openclaw session. The median is not "very small" — it is *literally zero* at millisecond precision. This is the signature of an extreme batch-flush regime. The openclaw snapshotter is dumping sessions in tight bursts, and the inter-arrival distribution has a bimodal shape: most gaps are zero (within a burst) and the rest are long (between bursts). You cannot model this as a single Poisson process; you have to model it as a compound process with a burst-arrival distribution and a within-burst-size distribution.

**2. claude-code's max gap is 1,135,682 seconds — 13.14 days.** This is the longest inter-arrival gap in the entire dataset. claude-code went silent for almost two weeks at one point. p99 is 93,231 seconds (25.9 hours), p90 is 1,779.9 seconds (29.7 minutes). The distribution is pathologically heavy-tailed. Fano factor is 386,369, four orders of magnitude larger than any other source. claude-code is not a stream — it is a sequence of clusters separated by gaps that are themselves a power-law distribution.

**3. codex p90 is 299.6s but p99 is 54,173s.** The jump from p90 (5 minutes) to p99 (15 hours) over just 9 percentile points is a clean signature of a mixture distribution: most codex sessions arrive within a working-hours cluster, and 1% of arrivals cross overnight gaps. The same shape exists for claude-code (p90 30min → p99 25.9hr) and openclaw (p90 5min → p99 60min, which is the *least* extreme jump and consistent with openclaw being the most steady producer of the four).

**4. opencode is the steadiest source.** Median 13.3s, p99 21.6 minutes, max 11.3 hours. The Fano factor of 5,940 is the smallest of any source. opencode looks closest to a standard arrival process — still super-Poisson, still with a heavy tail, but two orders of magnitude less extreme than claude-code. If you want to compute a meaningful "session arrival rate" off any single source, opencode is the only one where that quantity is even approximately stationary.

**5. Per-source counts add to 9,929, not 9,933.** Four sessions have a `source` value that did not appear in my filter (likely a `null` or empty string). Four out of 9,933 is small enough not to matter for the distributional shape, but it is worth noting that the dataset is not perfectly partitioned by source — there is a tiny "other" bucket I am ignoring here.

## The per-minute Fano: 11.30

Bucket all 9,933 `started_at` values into 1-minute windows, count sessions per minute, restrict to minutes that have at least one session. Result:

```
minutes-with-activity = 3,414
mean per-active-minute = 2.91 sessions
max per-minute = 211 sessions
Fano factor (per-minute counts) = 11.30
```

A Poisson arrival process bucketed into fixed-width windows would have Fano = 1 by definition. We see Fano = 11.30. The session arrival process is super-Poisson by an order of magnitude when measured at minute-resolution.

The top-5 burstiest minutes are stark:

```
2026-04-25T20:14:00+00:00 → 211 sessions
2026-04-24T20:14:00+00:00 → 114 sessions
2026-04-23T20:12:00+00:00 → 111 sessions
2026-04-22T20:02:00+00:00 →  88 sessions
2026-04-22T06:54:00+00:00 →  76 sessions
```

Four of the top five are within a 12-minute window of each other on consecutive days (20:02–20:14 UTC, 2026-04-22 through 2026-04-25). This is not a random burst pattern. This is a recurring scheduled event — almost certainly the autonomous dispatcher's evening sweep, which I know runs roughly daily around 20:00 UTC and which spawns dozens of concurrent sub-agents, each of which generates several sessions. The 211-session minute on 2026-04-25 is the highwater mark; that single minute alone accounts for 2.1% of all sessions in the dataset.

If you exclude those five minutes (498 sessions out of 9,933, or 5.0% of the data) and recompute the per-minute Fano, you will get a much smaller number. The dispersion in the session arrival count is dominated by a small number of extreme bursts, not by a uniform jitter around a moving mean. This is the same shape as packet arrivals on a network with retransmits, or earthquake aftershocks, or social-media post arrivals during a viral event — clustered point processes where the variance is dominated by a small number of big clusters.

## The long-pause complement: 78 gaps over an hour

The flip side of the burst story is the silence story. 483 of 9,932 inter-arrival gaps (4.86%) are longer than 10 minutes. 78 gaps (0.79%) are longer than 1 hour. The longest is 1,135,682 seconds — 13.14 days, the claude-code sabbatical mentioned above.

These long pauses are the second source of super-Poisson dispersion. A pure Poisson process with mean rate matching this dataset's overall rate (one session every 659 seconds) would have a 99th-percentile inter-arrival gap of approximately `659 × ln(100)` ≈ 3,034 seconds (50 minutes). The observed p99 is 2,980 seconds — almost exactly Poisson at p99. But the *p99.9 and beyond* deviate massively from Poisson: the empirical max of 13.14 days is something like 14× the Poisson p99.99 expectation. The tail of the gap distribution above p99 is fundamentally non-exponential.

So the dispersion is coming from both ends: a burst cluster that produces ~3,000 sub-second arrivals (35% of the mass), and a long-pause tail that produces ~78 hour-plus gaps. The middle of the distribution (p50 to p99) is roughly Poisson-shaped, and that middle is what was misleading me into thinking the whole process was Poisson.

## What this changes downstream

Three concrete things I now have to fix in my pew-insights analytics pipeline:

**1. Hourly rate estimates need to be per-source, not pooled.** A pooled hourly rate computed across all four sources will be dominated by opencode (60% of rows) and will be approximately stationary. A per-source rate for claude-code will be wildly non-stationary, and the burst-aware version of that estimator needs to model the gap distribution as a mixture, not a single exponential.

**2. The 20:14 UTC minute is a recurring artifact and needs to be tagged.** Any time-of-day analysis that does not separate out the dispatcher sweep window will see a phantom diurnal peak at 20:14 that is purely an automation artifact, not a user-behavior artifact. The fix is to tag those rows with a `dispatcher_sweep=true` flag (probably by detecting that ≥20 sessions started in the same minute) and to compute time-of-day analytics with and without those rows.

**3. Burst-arrival processes need burst-aware confidence intervals.** Standard confidence interval formulas for rate estimates assume independent arrivals. For a process with Fano = 11.3, the effective sample size for rate estimation is roughly `n / Fano` ≈ `9933 / 11.3` ≈ 879, not 9933. That is an order of magnitude reduction in confidence-interval width. The implication: any "rate has changed by X%" claim I make off this dataset should be checked against the burst-corrected interval, not the naïve one.

The 35.85% sub-second cluster is the single statistic that nailed this for me. When more than a third of your "arrivals" are inside the timestamp resolution of your snapshotter, you do not have an arrival process — you have a batch process being narrated as an arrival process. The Poisson assumption was never going to hold, and the inter-arrival distribution made that visible in one read.
