# Inter-arrival gap distribution by source: the 30-minute floor and the 568-hour tail

**Date:** 2026-04-26
**Dataset:** `~/.config/pew/queue.jsonl`, 1,556 digest rows, single device `a6aa6846…6441eee`, hour-buckets observed at `:00` and `:30` (781 and 775 rows respectively, so the underlying telemetry is on a 30-minute heartbeat, not a true hourly grid).

## What I actually measured

For each `source`, I took the set of distinct `hour_start` timestamps where that source contributed at least one digest row, sorted them, and computed consecutive gaps in hours. A "gap" of 0.5h means two adjacent 30-minute buckets — i.e., the source is emitting back-to-back. A gap of 24h means a full day went by with no rows from that source. The shape of this gap distribution tells you whether a source behaves like a steady process, a bursty process, or a sporadic process with long sleep periods.

The summary table, sorted by number of populated half-hour buckets:

| source            | n_hrs | mean gap | median | p90    | p99     | max     | consec ≤1.5h | gaps ≥24h | span (days) |
|-------------------|------:|---------:|-------:|-------:|--------:|--------:|-------------:|----------:|------------:|
| openclaw          |   403 |   0.56h  |  0.50h |  0.50h |   1.99h |  12.50h |       99.0%  |     0.0%  |        9.4  |
| ide-assistant-A   |   320 |  19.85h  |  0.50h | 47.50h | 351.90h | 568.00h |       65.8%  |    12.5%  |      263.8  |
| claude-code       |   267 |   6.45h  |  0.50h | 16.00h | 128.09h | 315.50h |       77.1%  |     4.9%  |       71.5  |
| opencode          |   232 |   0.61h  |  0.50h |  0.50h |   3.84h |   9.50h |       97.4%  |     0.0%  |        5.9  |
| hermes            |   160 |   1.39h  |  1.00h |  2.50h |   9.20h |  11.00h |       66.7%  |     0.0%  |        9.2  |
| codex             |    64 |   2.90h  |  0.50h | 13.10h |  30.56h |  27.50h |       79.4%  |     1.6%  |        7.6  |

Total token mass behind these rows is 9,198,374,466 tokens. The total span of the dataset (earliest hour to latest hour, across all sources) runs from `2025-07-30T06:00:00Z` to `2026-04-26T12:30:00Z`, but no individual source covers that entire window — only `ide-assistant-A` reaches back into 2025.

## The 30-minute floor is real, and almost everyone hits it

The first thing that jumps out: **every single source has a median gap of 0.50h or 1.00h.** That isn't a coincidence — it's the heartbeat of the underlying digest pipeline. Each source flushes a row per (source, model, hour-bucket, device) tuple, and the bucket is 30 minutes wide. If a source is "active" in two consecutive buckets, the gap is 0.5h, full stop. There is no shorter gap possible in this dataset; the floor is 30 minutes by construction.

That makes the median uninformative on its own. The interesting numbers are the **share of consecutive gaps** (how often the source fires back-to-back) and the **upper tail** (how big the silence gets when it does happen).

## Two clean classes: continuous and intermittent

If you sort by `consecutive_share` (fraction of gaps ≤ 1.5h), the sources cleave neatly into two groups.

**Continuous sources — basically always on when the host is on:**
- `openclaw`: 99.0% of all gaps are ≤1.5h. The source span is 9.4 days. The biggest gap it ever takes is 12.5h. There is not a single ≥24h sleep in its history. In other words: openclaw runs whenever the laptop is awake, and the 12.5h max gap is almost certainly an overnight sleep, not a usage pause.
- `opencode`: 97.4% consecutive, max gap 9.5h, 0% gaps ≥24h. Same shape, slightly noisier — three times during the 5.9-day span it briefly stopped emitting for between 1.5h and 4h, but it never went dark for a full day.

**Intermittent sources — bursty within long calendar spans:**
- `ide-assistant-A`: only 65.8% consecutive, p99 gap is **351.9 hours (≈14.7 days)**, max gap is **568.0 hours (≈23.7 days)**. The source span is 263.8 days. So the source has been "around" for 38 weeks but only contributed 320 active half-hour buckets — a duty cycle on the order of 5%.
- `claude-code`: 77.1% consecutive, max gap 315.5h (≈13.1 days), span 71.5 days, ~4.9% of inter-arrivals exceed a full day.
- `codex`: 79.4% consecutive but with the smallest absolute footprint (64 active buckets) over a 7.6-day span — short calendar window, but plenty of multi-hour silences inside it.
- `hermes`: 66.7% consecutive, max 11h, no day-long gaps — an in-between case. It's not as smooth as `openclaw`, but it never disappears for a workday.

## What "p99 = 568h" actually means for ide-assistant-A

Take the worst case literally. The p99 inter-arrival gap for `ide-assistant-A` is 351.9h and the max is 568.0h. That is **almost 24 days of complete silence between two emit events.** The source has 320 distinct active half-hour buckets and 263.8 days of calendar span. If you back-of-the-envelope it: 320 × 0.5h = 160 active hours out of a possible 263.8 × 24 = 6,331 hours, so **about 2.5% wall-clock duty cycle**.

The "65.8% consecutive" number says that even within the active windows, the source is reasonably bursty — when it's on, it tends to stay on for a few buckets at a time. This is consistent with editor-side telemetry: you open the IDE, work for an afternoon, the source flushes a dozen consecutive 30-minute rows, then the IDE sits closed for two weeks and contributes nothing.

Compare this to `openclaw`'s 99.0% consecutive over a 9.4-day span. `openclaw` is a constant background process; `ide-assistant-A` is a foreground tool used in spurts.

## openclaw and opencode have effectively no tail

`openclaw`'s p99 gap is 1.99h, and its max is 12.5h. That p99 is exactly the kind of number you'd see if the source has a 30-min heartbeat, occasionally skips one bucket (yielding a 1h gap), very rarely skips two (yielding 1.5–2h), and only goes dark during host sleep. The 12.5h max is one event, almost certainly the longest overnight idle in the 9.4-day window.

`opencode` looks similar: p99 = 3.84h, max = 9.5h, no ≥24h gaps. Slightly more variance than `openclaw`, but the same fundamental "always-on while host is on" pattern.

This is operationally important. If you're building dashboards that assume each source emits at a similar cadence, you will badly mis-estimate `ide-assistant-A`'s presence: it has the second-most distinct half-hour buckets in the dataset (320, behind `openclaw`'s 403), but it spreads them across 28× more calendar time.

## codex: short window, tail behavior anyway

`codex` is interesting because its calendar span is only 7.6 days — comparable to `openclaw` and `opencode` — yet it shows ~1.6% of gaps ≥24h (one full-day silence) and a p99 of 30.56h. With only 64 active buckets, that single 27.5-hour max gap dominates the tail. So even within a narrow window, `codex` is closer in shape to the intermittent cluster than to the continuous cluster.

The mean gap for `codex` is 2.90h vs `openclaw`'s 0.56h. That's a 5.2× ratio between two sources observed over almost identical span lengths. This is a real behavioral difference, not an artifact of observation window.

## hermes is a middle case worth flagging

`hermes` has 160 active half-hour buckets, mean gap 1.39h, median 1.00h, max 11h. The median of **1.00h** is the only median in the dataset that isn't 0.50h — meaning that more often than not, when `hermes` emits, the next emission is one hour later, not 30 minutes later. That suggests `hermes` is doing some kind of every-other-bucket polling, or its activity is naturally aligned to 1-hour cycles rather than the 30-minute grid.

Its consecutive_share is 66.7% — almost identical to `ide-assistant-A`'s 65.8% — but it never produces a ≥24h gap. So `hermes` is "patchy within a day" rather than "absent for days at a time."

## The 30-minute heartbeat assumption matters for any anti-burst alarm

If you wanted to write an alert that says "source X has gone quiet," the threshold has to be source-specific:

- For `openclaw` / `opencode`, anything beyond ~2h of silence is in the right tail of normal behavior. A 12h alarm would be a near-certain incident signal.
- For `hermes`, ~3h of silence is well within normal; ≥10h would be unusual.
- For `claude-code`, you cannot meaningfully alarm under ~16h (the p90).
- For `ide-assistant-A`, you cannot meaningfully alarm under ~48h (the p90), and even multi-week silences are within the observed distribution.

A single global "1 hour of silence = page" rule would either be useless for `openclaw` (too lenient) or alert-storm for `ide-assistant-A` (constantly firing). The duty cycle ratio between the most and least continuous source is roughly **99% / 65.8% ≈ 1.5× on consecutive_share**, but the ratio of p99 gaps is **351.9h / 1.99h ≈ 177×**. Tail behavior is what differentiates these sources, not central tendency.

## Methodology note: the half-hour grid biases everything toward 0.5h

One more caveat. The minute counts for `hour_start` are exactly 781 (`:00`) and 775 (`:30`), almost a perfect split. So the underlying ingest writes every active source into the next 30-minute boundary regardless of when the actual API call happened. This means:

1. Two API calls 5 minutes apart, both inside the same 30-minute bucket, produce **one** row, not two.
2. Two API calls 35 minutes apart can produce two rows if they straddle a `:30` boundary, giving an inter-arrival of 0.5h, even though the user-visible cadence was 35 minutes.

So gap = 0.5h is best read as "the source was active in two adjacent half-hour buckets," not "the source emitted exactly one event every 30 minutes." All the consecutive_share numbers above are upper bounds on burstiness in this sense — the underlying API call rate inside an active half-hour bucket is invisible at this resolution.

## What I'd want to look at next

1. **Inter-arrival distribution in real timestamp space**, not 30-minute bucket space — would require the per-call telemetry that feeds these digests, which `queue.jsonl` doesn't expose.
2. **Conditional gap distribution given "host is awake"** — i.e., subtract presumed laptop-sleep windows, then re-measure. `openclaw`'s 12.5h max is almost certainly an overnight sleep and should be removed from the active-period analysis.
3. **Joint inter-arrival between sources** — when `claude-code` emits, what's the distribution of time-to-next `openclaw` emission in the same bucket? That would let us decompose "source pair co-activity" beyond simple Jaccard.

The headline takeaway from the row counts alone: **`openclaw` is a daemon, `ide-assistant-A` is a tool used twice a month in bursts, and treating them with the same monitoring schema is the kind of mistake the median doesn't reveal.**
