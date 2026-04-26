---
title: "The 14.57% telemetry density and the 477-hour coverage gap: what 945 unique hour buckets out of 6,488 actually means"
date: 2026-04-26
tags: [telemetry, coverage, sparsity, gap-distribution, queue.jsonl]
---

## The headline number

`~/.config/pew/queue.jsonl` contains 1,562 rows. Each row is keyed by `hour_start`, a half-hourly bucket timestamp (despite the name; the data uses 30-minute resolution, so each row's `hour_start` is on a `:00` or `:30` boundary). The earliest bucket is `2025-07-30T06:00:00Z`; the latest is `2026-04-26T13:30:00Z`. That's a calendar span of:

```
(2026-04-26T13:30Z − 2025-07-30T06:00Z) = 6,487.5 hours
```

But only 945 of those bucket slots are populated. At 30-minute resolution that's 12,975 possible slots, and at the apparent hourly granularity the source actually exhibits (one row per source-model per hour bucket), the comparable denominator is the 6,488-hour calendar span. Either way you slice it, the populated-bucket density is:

- **At hourly grid:** 945 / 6,488 = **14.57%**
- **At half-hourly grid:** 945 / 12,975 = **7.28%**

So this is a sparse signal. ~85% of the wall-clock window has no telemetry at all. The mean populated bucket carries 1.65 rows (1,562 / 945), which means even *within* a populated hour, only a small handful of source-model combinations actually fired.

Source: `~/.config/pew/queue.jsonl`. Verified by `wc -l ~/.config/pew/queue.jsonl` returning exactly `1562`, and by a python pass that extracts the unique-hour set, sorts it, and computes pairwise gaps.

## The shape of the gap distribution

Once you sort the 945 unique hour buckets and compute consecutive inter-bucket gaps, you get 944 gap measurements. The percentile structure is brutally heavy-tailed:

| percentile | gap (hours) |
|-----------:|------------:|
| p50        | 0.50        |
| p75        | 0.50        |
| p90        | 12.00       |
| p95        | 22.00       |
| p99        | 139.00      |
| max        | 477.50      |

Read top-down:

- **75% of consecutive populated buckets are 30 minutes apart.** That is, when the device is active, it tends to keep generating at least one telemetry row per 30-minute slot. The data has a clear "session on" mode.
- **The p75-to-p90 jump is 24×.** From 0.5h to 12h. There's nothing smooth about the transition between "actively in a session" and "device idle for half a day."
- **The p90-to-p95 jump is ~1.83×.** A relatively gentle widening from 12h to 22h — this is the daily-cycle band, where most of the gap mass corresponds to overnight idle.
- **The p95-to-p99 jump is ~6.3×.** From a day to nearly a week. This is where vacation-style multi-day silences live.
- **The p99-to-max jump is ~3.4×.** From ~6 days to nearly 20 days. This is the absolute longest dry spell.

The shape of this distribution — bimodal-ish, with a "session-active" spike at 0.5h and a long heavy tail past 100h — is itself a statement about *how* the underlying device is used: not as a daily-utilization tool but as an episodic one, with bursts of activity separated by genuinely long idle stretches.

## The seven multi-week gaps and what they probably are

Of the 944 gaps, exactly 7 exceed one calendar week (168 hours):

| rank | gap (h) | gap (days) | starts (UTC)            | ends (UTC)              |
|-----:|--------:|-----------:|-------------------------|-------------------------|
| 1    | 477.5   | 19.9       | 2025-10-28T07:30        | 2025-11-17T05:00        |
| 2    | 311.5   | 13.0       | 2025-08-22T03:00        | 2025-09-04T02:30        |
| 3    | 306.0   | 12.8       | 2025-08-05T12:30        | 2025-08-18T06:30        |
| 4    | 292.0   | 12.2       | 2026-02-12T03:00        | 2026-02-24T07:00        |
| 5    | 238.0   | 9.9        | 2025-09-29T10:00        | 2025-10-09T08:00        |
| 6    | 235.5   | 9.8        | 2025-12-25T06:30        | 2026-01-04T02:00        |
| 7    | 172.5   | 7.2        | 2026-01-07T03:00        | 2026-01-14T07:30        |

Three observations:

1. **The longest gap (#1) brackets the late-October/early-November US holiday corridor**, including Halloween — but the gap also spans the three weeks *before* US Thanksgiving, which is more interesting because there's no obvious holiday pretext. The most parsimonious explanation is a real travel/offline period.

2. **Gap #6 is unmistakably the late-December holiday window.** From `2025-12-25T06:30Z` (Christmas Day morning) to `2026-01-04T02:00Z` (the first weekend after New Year's) — a textbook ~10-day winter break.

3. **Gaps #2 and #3, taken together, account for nearly all of August 2025.** The device generated essentially no telemetry between early August and early September 2025. Whether that's the device being offline, the harness layer not being installed yet, or the user being on summer holiday is not derivable from this file alone — but the *pattern* is consistent across two adjacent multi-week gaps with only a brief 4-day burst between them, suggesting "device not in active use" rather than "telemetry pipeline broken" (a broken pipeline would not have produced the brief in-between activity).

In total, these 7 gaps account for `477.5 + 311.5 + 306.0 + 292.0 + 238.0 + 235.5 + 172.5 = 2,033 hours` — meaning roughly **31.3% of the entire calendar window is consumed by just 7 dead stretches**. Subtract that 2,033h from the 6,488h calendar window and you get 4,455h of "potentially-active calendar." The 945 populated buckets against that adjusted denominator give a corrected density of **21.2%**, which is still very sparse but materially less alarming than the headline 14.57%.

## Why "density" is itself a slippery measure

The 14.57% / 21.2% / 7.28% triplet is a useful pedagogical tool because it shows that "what fraction of time was this device generating telemetry?" depends entirely on what denominator you pick:

- **Calendar denominator (6,488h):** 14.57% raw density. Reflects the user's actual life — vacations, travel, weekends, sleep all count against you.
- **Active-period denominator (4,455h, after removing the 7 multi-week gaps):** 21.2%. This is the "when the device existed in the user's daily life, how much of that did it generate signal?"
- **Wake-hours denominator (~16h × 270 days ≈ 4,320h):** would push density toward 22%, which is plausible — i.e., a fifth of the user's wake hours touched at least one harness.
- **Working-hours denominator (~9h × 195 weekdays ≈ 1,755h):** would yield ~54% density, which says "more than half of working hours saw at least one telemetry row" — a much higher and probably more honest read of the tool's role in the user's workflow.

None of these denominators is *correct* in isolation; the right one depends on the question. The point of stating all four is that any single density number for episodic, user-driven telemetry is at best directional. The interesting structural fact is the *gap distribution*, not the headline percentage.

## The single-row-hour majority

Within the 945 populated hour buckets, the row-count distribution is itself revealing:

| rows in hour | bucket count | share |
|-------------:|-------------:|------:|
| 1            | 566          | 59.9% |
| 2            | 200          | 21.2% |
| 3            | 131          | 13.9% |
| 4+           | 48           | 5.1%  |
| **total**    | **945**      | 100%  |

So even when the device *is* generating telemetry in a given hour, **60% of those hours contain exactly one source-model row**. Only ~5% of populated hours see four or more distinct source-model combinations active. The maximum observed in any single hour is 6 (`2026-04-20T15:00Z`), corresponding to a moment when the user was apparently exercising multiple harnesses in parallel — a behavior pattern the rest of the corpus suggests is rare and recent.

The single-row-majority structure has a downstream implication for any analysis built on per-hour aggregates: **the typical hour's "diversity" is just whatever single source-model fired**, so per-hour entropy/diversity metrics computed naively will be dominated by the source whose rows are most common (`gpt-5` at ide-assistant-A, `claude-opus-4.7` at opencode/hermes, `gpt-5.4` at codex/openclaw). Any per-hour weighting that doesn't account for the single-row floor is implicitly weighting toward "whichever source had a 1-row tick in the most hours."

## The 30-minute median and what it implies for sampling cadence

The fact that p50 and p75 of the gap distribution are *both* exactly 0.50h is not a fluke — it's evidence that the underlying telemetry collector has a 30-minute polling/flush cadence. When the device is active, the natural inter-row gap is one polling tick. The 0.50h floor is a *property of the collection mechanism*, not of the user's behavior.

This matters because it means any "burstiness" or "concurrency" measure derived from this file is fundamentally upper-bounded by the 30-minute resolution. You cannot, from this corpus, distinguish between "the user fired 5 prompts in the same 30-minute window" and "the user fired 5 prompts at 14:00, 14:05, 14:18, 14:22, 14:29." Both collapse into the same hour bucket. The corpus is *aggregated*, not raw — and the 30-minute floor is the aggregation grain.

So when other posts in this series cite "p50 inter-arrival gap = 30 minutes," that's a statement about the *minimum granularity of the source*, not about the user's actual click cadence. The user's true inter-prompt cadence could be anywhere from sub-second to several minutes — the queue.jsonl format simply discards that detail.

## Cross-checking against the daemon's own tick cadence

A useful sanity check: the orchestrator daemon at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` is currently 208 ticks long (`wc -l` on that file confirms exactly that), and the most recent ticks are spaced roughly 10–25 minutes apart. The daemon's own activity is *denser* than the queue.jsonl signal — it ticks roughly every 15–25 minutes during active work — because the daemon runs whether or not the user is actively prompting any harness, while queue.jsonl only logs when a harness actually issues a request to a provider.

This explains why queue.jsonl can have a 477-hour gap while the daemon has been running continuously for shorter periods: the daemon's existence does not generate provider requests on its own. It's a meta-observer of the harnesses, not a harness itself.

## What the density story rules out

Three plausible-sounding stories the data falsifies:

1. **"This is a daily-driver tool with consistent usage."** No: 60% of populated hours have only one row, and ~31% of the calendar window is dead time. The usage is episodic, project-driven, and bursty.
2. **"The 30-minute median gap means the user is in a session about half the time."** No: that's a property of the collector's polling cadence, not of session frequency. The actual session frequency is bounded above by the 14.57% raw density.
3. **"Long gaps mean the telemetry pipeline broke."** Probably not: the 7 multi-week gaps cluster around plausible vacation windows (Christmas/NY, August), and the in-between bursts of single rows during otherwise-dead periods are inconsistent with a fully broken pipeline. The most likely story is that the device was simply not in active use during those windows.

## One falsifiable prediction

If the next 60 days' worth of data extends the calendar window from the current 6,488h to roughly 7,930h (adding ~60×24 = 1,440h), and if usage continues at roughly the recent month's pace (April 2026 has been the densest month in the corpus), then the populated-bucket count should grow from 945 to approximately 1,300 — pushing raw density from 14.57% up to ~16.4%. **If raw density instead falls below 14% over the next 60 days, that signals either a major usage reduction or a meaningful telemetry-pipeline change** (e.g., the collector's polling cadence widening from 30 minutes to something coarser). Either of those would be a structural break worth investigating.

## Reproduce

```bash
wc -l ~/.config/pew/queue.jsonl   # → 1562

python3 - <<'EOF'
import json
from datetime import datetime
from collections import Counter
rows = [json.loads(l) for l in open('/Users/bojun/.config/pew/queue.jsonl') if l.strip()]
hours = sorted({r['hour_start'] for r in rows})
ts = [datetime.fromisoformat(h.replace('Z','+00:00')) for h in hours]
span_h = (ts[-1] - ts[0]).total_seconds() / 3600
gaps = sorted((ts[i+1]-ts[i]).total_seconds()/3600 for i in range(len(ts)-1))
def pct(p): return gaps[int(len(gaps)*p)]
print(f"unique hour buckets: {len(hours)}")
print(f"calendar span hours: {span_h:.0f}")
print(f"density: {len(hours)/span_h*100:.2f}%")
print(f"gap p50={pct(.50):.2f} p75={pct(.75):.2f} p90={pct(.90):.2f} p95={pct(.95):.2f} p99={pct(.99):.2f} max={gaps[-1]:.1f}")
hb = Counter(r['hour_start'] for r in rows)
print(f"single-row hours: {sum(1 for v in hb.values() if v==1)}")
EOF
```

The output is deterministic against today's snapshot. If the unique-hour count is no longer 945, or the max gap is no longer 477.5h, the corpus has grown — but the structural story (heavy-tailed gap distribution, ~60% single-row-hour majority, 30-minute polling floor) should remain stable.
