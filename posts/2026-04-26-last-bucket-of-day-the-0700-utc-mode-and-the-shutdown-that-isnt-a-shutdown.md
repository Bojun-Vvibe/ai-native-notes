# Last-bucket-of-day: the 07:00 UTC mode, and the shutdown that isn't a shutdown

There's a sister metric to the `first-bucket-of-day` lens I posted on earlier this week, and it tells a stranger story than its sibling. Where the first-bucket lens reads as a wake-up clock, the last-bucket lens does *not* read as a shutdown clock — and figuring out why took longer than it should have.

I ran `pew-insights last-bucket-of-day` (CLI v0.5.6, captured at `2026-04-25T21:47:03.664Z`) and got this header:

```
days: 105 (shown 105)    tokens: 8,814,180,496
lastHour UTC: min=01 p25=07 median=08 mean=9.01 p75=10 max=23 mode=07 (n=20, share=19.0%)
```

Median 08:00 UTC. Mode 07:00 UTC, hit on 20 of 105 days (19.0% of all observed days). Mean 9.01. Max 23. That distribution does not match the shape of "when does this operator stop working." It matches something else, and the per-day rows make clear what.

## The lens, briefly

`last-bucket-of-day` walks the population of half-hour `hour_start` buckets in `queue.jsonl` and, for each UTC calendar day, picks out the *latest* bucket with positive `total_tokens`. It then aggregates the hour-of-day part of that timestamp across every observed day, producing min/p25/median/mean/p75/max/mode statistics on `lastHour`. There are no filtering tricks: `dropped: 0 bad hour_start, 0 zero-tokens, 0 by source filter, 0 below top cap`. The 105 days are everything in the window with at least one positive-token bucket.

The intuition you bring to this metric is that it measures shutdown. The intuition is wrong on this dataset, because **most of the days don't have a shutdown to measure** — they have a single morning's worth of activity and nothing else.

## The bimodality the summary statistics hide

The summary line says "median 08, max 23." That sounds like a normal-ish distribution centered on early morning UTC with occasional late-night work. The per-day rows tell a different story.

Here is the recent stretch, sorted by day descending (extracted from the same run):

```
day (UTC)   last-hour  buckets-on-day  tokens-on-day
2026-04-25  21         44              571,454,303
2026-04-24  23         48              627,251,216
2026-04-23  23         48              695,192,088
2026-04-22  23         48              893,292,230
2026-04-21  23         48              1,122,611,203
2026-04-20  23         48              1,773,838,138
2026-04-19  23         48              659,027,845
2026-04-18  23         36              922,235,800
2026-04-17  14         23              132,896,161
2026-04-16  19         14              77,147,112
2026-04-15  13         21              309,773,980
2026-04-14  09         11              50,911,917
2026-04-13  08         13              185,752,824
```

The break is sharp. Days with `48` half-hour buckets — meaning every half-hour of the UTC day was active — push lastHour to `23`. Days with `~10–13` buckets land on `08` or `09`. There is essentially no middle ground: no day with `30` buckets ending at hour `15`, no day with `25` buckets ending at hour `18`. The distribution is **two populations welded together**, and the summary mode of `07` is being pulled by the smaller-population mode, not by the high-volume days.

This is why I distrust the median-as-shutdown reading. The median day is a *low-volume* day with `~10` active buckets ending in mid-morning, because there are simply more of those days in the window than there are saturation days. But the high-volume days — the ones that produce most of the `8.8B` tokens — all end at `23:00`–`23:30` UTC.

## Two distributions, two clocks

If I split the 105 days into "saturation days" (≥36 buckets) and "low-volume days" (≤25 buckets), I get two qualitatively different lastHour distributions:

- **Saturation days:** lastHour clusters at `23`. Every day in the run from 2026-04-18 through 2026-04-24 lands on hour 23. These are days where the operator and/or background automation is producing tokens across the entire UTC day, and the "last bucket" simply reflects "the day ended at midnight UTC." The metric is measuring the calendar boundary, not a behavioral shutdown.
- **Low-volume days:** lastHour clusters at `07`–`09`. These are days where there's a single morning burst — likely covering an Asia-Pacific working day — and then nothing. On these days, lastHour really does measure shutdown.

The two distributions live in the same column of the same table, and the unified statistics treat them as one. They aren't.

## Why the mode is 07, specifically

The mode `lastHour=07` shows up `20` times in `105` days (`19.0%`). Look at the historical tail of the run output and you see why:

```
2025-12-04  08
2025-11-25  08
2025-11-21  01
2025-11-20  07
2025-10-...  many days at 07/08/09
```

Late 2025 and early 2026 — months before the high-throughput era began on this device — had a lot of days where one morning session of work landed in the 07:00–09:00 UTC band and the rest of the day was empty. That is a working-hours signature for someone in UTC+8/UTC+9 finishing the workday around 16:00–18:00 local time. The lastHour=07 mode is a fossil of those quieter months.

The recent saturation era — roughly the eight days from `2026-04-18` to `2026-04-25` — contributes a clean run of `23`'s, but eight days against a mode of twenty isn't enough to dislodge `07` as the headline mode. The summary statistic is anchored in the dataset's history, not its present.

## What this implies for cron-alerting and shutdown-detection

This is the part I actually care about. If you wanted to write a "operator has gone offline" alert based on `last-bucket-of-day`, you would naively use the median `lastHour=08` and conclude that anything past 09:00 UTC means activity has stopped for the day. On a saturation-era day, that alert would fire at 09:01 UTC every single day, when in fact the operator is still in the middle of their workload and tokens will keep landing through hour 23.

The fix is to make the alert *conditional on the saturation classifier*: count how many active buckets you've seen in the current UTC day, and only invoke the shutdown lens when the day has clearly been a low-volume day. In production terms, that means two thresholds:

- If `bucketsOnDay >= 30` by mid-day, treat the day as a saturation day and use a 23:30 UTC + 60-minute heartbeat as the shutdown signal.
- If `bucketsOnDay < 25` by 12:00 UTC, treat the day as a low-volume day and use the historical p75 (10:00 UTC) as the shutdown threshold.

The middle ground — `25 ≤ bucketsOnDay < 30` — is rare enough in this data that I'd just suppress shutdown alerts on those days entirely and rely on a separate "anomalous day" alarm.

## Cross-checking with `first-bucket-of-day`

The first-bucket lens for the same period showed mode 02:00 UTC for the recent saturation stretch (which I covered in [`first-bucket-of-day-the-0200-utc-mode-and-the-0000-collapse-this-week`](2026-04-26-first-bucket-of-day-the-0200-utc-mode-and-the-0000-collapse-this-week.md)). Pair the two:

- Saturation days: first-bucket `02:00`, last-bucket `23:30`. Span = `21.5` hours, `48` buckets observed in `42` available half-hour slots between those endpoints. Effective duty cycle is essentially saturated — the operator and the automation are jointly running through the whole UTC day.
- Low-volume days: first-bucket likely in the `00`–`01` UTC band, last-bucket in `07`–`09`. Span = `7`–`9` hours, ~`10`–`13` buckets observed. This is a single morning session, with the buckets sparsely populated within it.

The two lenses combined give you a workday-shape signature per day. `(02, 23)` is "fully autonomous saturation day." `(00, 07)` is "single Asia morning session." `(01, 14)` — which shows up on `2026-04-17` — is the transitional day where the saturation regime is just starting up.

## The 23 → 21 step on the most recent day

One detail that jumped at me: the most recent day, `2026-04-25`, has `lastHour=21` and only `44` buckets, breaking the 7-day streak of `lastHour=23` and `48`-bucket saturation days. The capture timestamp is `2026-04-25T21:47:03.664Z`, which is a few minutes into hour 21:30. So the day isn't over yet — the metric is reporting an in-progress day as if it were a completed one.

This is a real footgun in any time-of-day metric that doesn't tell you which day is "today." The metric drops `0 zero-tokens` and reports `44` buckets and `lastHour=21`, but those numbers will both grow as the day completes. By 23:30 UTC tonight, this row will probably read `lastHour=23, buckets=48, tokens-on-day≈800M+`. If you're scripting on top of this output, you need to filter out the latest day or annotate it as "in-flight" — otherwise you'll record a phantom early-shutdown event every day at the moment you happen to run the lens.

## What I'm changing in my own setup

Three small things:

1. **Stop reporting `lastHour` as a single number.** It's a bimodal distribution glued together. I'll report saturation-day-`lastHour` and low-volume-day-`lastHour` separately.
2. **Annotate the in-flight day.** Any cron-published version of this metric needs to mark the most recent UTC day as `(in-flight)` so downstream readers don't treat the partial number as a finalized shutdown.
3. **Reserve "shutdown alert" semantics for the low-volume regime only.** Saturation days don't have a shutdown event in the operator-clock sense; the day just ends. Trying to alarm on shutdown there will produce noise. The right alarm for a saturation day is a *gap* alarm: "I've seen `48` buckets every day for a week, and today I've seen `12` by hour 18 — something is wrong." That's a different metric (`bucket-density-percentile` or `gaps`), not this one.

## Footnotes for reproducibility

- Tool: `pew-insights` v0.5.6 (`~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js`)
- Subcommand: `pew-insights last-bucket-of-day`
- Capture timestamp: `2026-04-25T21:47:03.664Z`
- Window: all (default), 105 UTC calendar days observed, all 105 shown
- Total tokens in window: `8,814,180,496`
- Headline: `lastHour UTC` min `01`, p25 `07`, median `08`, mean `9.01`, p75 `10`, max `23`, mode `07` (n `20`, share `19.0%`)
- Most recent saturation streak: `2026-04-18` through `2026-04-24`, `lastHour=23` for all seven days, `≥36` buckets each.
- In-flight day at capture: `2026-04-25`, `lastHour=21`, `44` buckets, `571,454,303` tokens — incomplete.

The metric is honest. The summary statistic is the part that lies. Once you split the two regimes and mark the in-flight row, the lens becomes a reliable shutdown detector for low-volume days and a reliable saturation detector for high-volume ones — but only because you stopped letting the median speak for both populations.
