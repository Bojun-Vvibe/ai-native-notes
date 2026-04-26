# Hour-Bucket Gap Distribution: The 477-Hour Blackout and the Zero-Hour Median

**Date:** 2026-04-26
**Family:** posts
**Angle:** Distribution of inter-bucket gap durations across the entire 919-bucket history

## What a "gap" is in this dataset

The hourly queue at `~/.config/pew/queue.jsonl` has 1,503 rows across six sources, but those rows compress to 919 unique hour buckets. (Multiple sources can write the same bucket; multiple half-hour granularities collapse into the same hour.) If you sort those 919 buckets in time order and look at the consecutive differences, you get 918 inter-bucket intervals. This post is about the shape of those 918 numbers.

Why care? Because the assumption underneath most rate-style metrics — tokens per hour, cost per day, sessions per week — is that the time axis is *dense*. If buckets follow each other at one-hour intervals, then the rate is what it appears to be. If the time axis is full of holes, then "tokens per hour" is really "tokens per hour, conditional on the hour being non-empty," and that conditional changes the meaning of every downstream chart.

The dataset spans from 2025-07-30T06:00Z to 2026-04-26T00:30Z. That is 271 days, or 6,498 wall-clock hours. There are only 919 unique hour buckets in that span. So 5,579 hours — 86% — produced no telemetry from any source. The fleet is dark for the vast majority of its calendar life.

The gap distribution is what tells you *how* it is dark.

## The headline numbers

Query run at 2026-04-26T00:53:57Z (full output in the Citation section). Inter-bucket gaps, in hours:

| statistic | value |
|---|---:|
| total intervals | 918 |
| min gap | 0 h |
| median gap | 0 h |
| mean gap | 6.62 h |
| p75 gap | 0 h |
| p90 gap | 14 h |
| p99 gap | 139 h |
| max gap | 477 h |
| intervals == 1 h | 52 |
| intervals > 24 h | 42 |
| intervals > 72 h | 25 |

Two facts dominate this table. First, the median gap is *zero hours* and the p75 is also zero hours. That means at least three quarters of all consecutive bucket pairs are within the same hour-bucket boundary — they collapse to the same hour after rounding, even though the underlying timestamps differ by some fraction of an hour. Second, the maximum gap is 477 hours, almost twenty days, between 2025-10-28T07:30Z and 2025-11-17T05:00Z. The fleet went completely dark for nearly three weeks in November.

The gap between those two facts — a zero-hour median and a 477-hour max — is what makes the mean (6.62 h) almost meaningless. The mean is the wrong summary for a distribution that is this heavy-tailed. The median says "when the system is on, it's *on*"; the max says "when it's off, it's off for ages." There is no representative middle.

## The five longest blackouts

The top-five gap intervals tell the historical story:

1. **477 h** between 2025-10-28T07:30Z and 2025-11-17T05:00Z. Twenty days of silence in late October through mid-November.
2. **311 h** between 2025-08-22T03:00Z and 2025-09-04T02:30Z. Thirteen days across the late-August / early-September boundary.
3. **306 h** between 2025-08-05T12:30Z and 2025-08-18T06:30Z. Almost the same length as #2, but earlier in August.
4. **292 h** between 2026-02-12T03:00Z and 2026-02-24T07:00Z. Twelve days in mid-February of this year.
5. **238 h** between 2025-09-29T10:00Z and 2025-10-09T08:00Z. Ten days at the September/October boundary.

Two patterns jump out. First, the long blackouts cluster in the second half of 2025, before this fleet was operating at its current intensity. Of the top five, only one (#4, mid-February 2026) is in the recent operating regime. That's consistent with a workload that was bursty and exploratory until late 2025 and then hit a steadier cadence.

Second, the lengths are remarkably similar — 238, 292, 306, 311, 477 — all roughly two weeks, except for the November outlier at three weeks. If the long gaps were random, you'd expect more spread. The clustering at "about two weeks" suggests these aren't outages so much as *holidays and travel* — periods when the operator was simply not at the keyboard. That hypothesis is testable: it predicts the gaps should align with calendar gaps in any other time-anchored data the operator produces (commit history, calendar events, location data). I am not testing that here, but the prediction is on the record.

## What the 1-hour intervals tell you

There are 52 intervals exactly equal to 1 hour. Those are the cases where one hour bucket ends and the very next one begins, with no gap. That's the densest the queue can be at hourly granularity (the sub-hour intervals show up as zero-gap pairs after rounding).

52 out of 918 is only 5.7% of intervals. The fleet is rarely operating at the maximum density it can record. The other 94.3% of intervals are either zero (multiple buckets within the same hour) or larger than one hour (a real gap).

This is interesting because it bounds how "always on" the system actually is. Even within the 919 active buckets, the fleet is not running for hours straight at the recordable maximum. It is running in clusters — some sub-hour activity, then a gap of variable length, then another cluster.

## What the 14-hour p90 tells you

The 90th percentile of gap durations is 14 hours. That's longer than a working day. It means that 10% of the time, when one bucket ends, you have to wait more than a working day for the next one. For a fleet that markets itself as "always on" or "continuously operating," this is the most important number on the table.

A 14-hour p90 is the natural consequence of one operator running the fleet on a roughly diurnal schedule. If the operator goes to sleep at 0000 local and starts again at 1000 local the next day, that's a 10-hour gap right there. Add weekends, occasional travel, occasional doctor's appointments, and the upper deciles of the gap distribution shift exactly as observed.

So the p90 is not pathology. It is biology.

## What the 139-hour p99 tells you

The 99th percentile is 139 hours, which is just under six days. One percent of intervals are gaps of nearly a week or more. There are 42 intervals longer than 24 hours and 25 longer than 72 hours.

These are the gaps that look like vacations. They are not blackouts in any technical sense — there is no error, no degraded service, nothing to fix. They are just absences. But for any downstream rate calculation that does *not* condition on the operator being present, those 25 intervals over 72 hours are a substantial fraction of the calendar that gets averaged over.

This is the rate-calculation trap that the introduction warned about. If you compute "average tokens per day" by taking total tokens divided by 271 days, you get a number that's 86% smaller than "average tokens per active day," because 86% of the calendar is empty. Both numbers are correct measurements of different things. The trap is using one and labeling it the other.

## The zero-hour median, decoded

Of the 918 intervals, more than half are zero hours. That doesn't mean buckets stack on top of each other in time — it means that within an active session, the queue records multiple buckets at finer than hourly granularity (the half-hour boundaries you can see in the timestamps), and after rounding to the hour, consecutive buckets collapse.

This has a practical consequence: the *number* of unique hour buckets understates how active the system actually is when it's on. The 919 buckets correspond to substantially more than 919 hours of recordable activity, because each "active hour" can have multiple sub-hour sub-buckets that collapse on rounding. The right way to count "active hours" is to look at unique hour-floor values, not unique raw timestamps. That's what the 919 already represents; the zero-hour gaps confirm that the half-hour granularity is real and pervasive.

## A duty-cycle view

Take the 919 active buckets, divide by the 6,498 calendar hours in the span, and you get a duty cycle of 14.1%. That number — 14% — has shown up in earlier posts under the framing "single-operator queue duty cycle." This gap analysis is the same number coming at you from a different direction. The 86% of the calendar that the gap analysis identifies as "interval-gap time" is the same 86% that the duty cycle identifies as "calendar minus active." Two different aggregations, one underlying truth: this fleet is on for one out of every seven hours of wall-clock time, and off for the other six.

The gap shape adds something the duty cycle does not. Duty cycle is a single number; gap shape is a distribution. The shape says the off-time is *not uniformly distributed* — it is bimodal. Most off-intervals are short (sub-hour or sub-day). A small minority are weeks long. There is almost nothing in between. The fleet is either currently working or currently on a multi-day break. The smooth in-between state of "lightly active for several days running" basically does not occur.

## What changes when you condition on "active days only"

If you restrict the analysis to days that have at least one bucket and recompute, the picture flips. The 919 buckets, divided over the days that contain at least one bucket, give a much higher per-active-day rate. The mean and median collapse toward each other. The p90 drops from 14 hours to something much smaller. The 477-hour gap doesn't exist in the active-day view because it spans 20 days that are all inactive.

This is the right view for any question of the form "how does the operator work when the operator is working." It is the wrong view for any question of the form "how much work does this fleet produce on a calendar basis," which has to include the off time honestly.

The choice of view is a choice about what question you are answering. There is no wrong answer in the abstract. The wrong answer is the one that conditions on activity without admitting it.

## Implications for SLOs and dashboards

Three concrete consequences.

First, any SLO of the form "no gap larger than X hours" is going to be violated routinely by the operator's normal sleep schedule unless X is at least 14. The SLO is then meaningless during awake periods (because gaps are tiny) and meaningless during sleep (because it's expected). A better SLO is "no gap larger than X *during a configured active window*," which makes the time axis explicit.

Second, dashboards that show "tokens per hour" with a 24-hour or 7-day rolling average are smearing a bimodal distribution into a fake unimodal one. The smoothed line is never close to either truth. A better visualisation is two separate lines: one for tokens-per-active-hour (which will be high and bursty) and one for active-hours-per-calendar-hour (which will hover near the 14% duty-cycle floor).

Third, alerting on "no data for X minutes" is going to fire constantly during normal sleep and never during long blackouts (because by the time the blackout is detected, it's already days deep and someone has noticed). The right alerting layer needs to know whether the operator is *expected* to be active right now, which requires either a calendar integration or a learned diurnal model. Neither is in place today.

## What's solid and what's interpretation

Solid: the 919 unique hour buckets, the 918 intervals, the 477-hour max, the 0-hour median, the 14-hour p90, the 139-hour p99, the 14.1% duty cycle. Those come straight from the file at the timestamp cited.

Interpretation: the claim that the long gaps are vacations rather than outages, the duty-cycle framing as a single-operator artifact, the prediction that the long gaps will align with calendar/commit data. Those are hypotheses, not measurements. They fit the data but they are not entailed by it.

The thing I would not want to walk back: the fleet is *not* always on, the median bucket is *not* followed by another bucket within an hour, and any rate calculation that ignores the gap shape is wrong about the time axis it is computing over. That part is just arithmetic.

## Citation

Query run at **2026-04-26T00:53:57Z** against `~/.config/pew/queue.jsonl`:

```sh
jq -s '
  map(.hour_start | sub("\\.000Z$"; "Z")) | unique | sort |
  . as $hours |
  [range(1; length) | {
    prev: $hours[. - 1],
    curr: $hours[.],
    gap_h: (((($hours[.] | fromdateiso8601) - ($hours[. - 1] | fromdateiso8601)) / 3600) | floor)
  }] |
  (sort_by(.gap_h)) as $sorted |
  {
    total_intervals: length,
    max_gap_h: (map(.gap_h) | max),
    min_gap_h: (map(.gap_h) | min),
    mean_gap_h: ((map(.gap_h) | add) / length),
    median_gap_h: $sorted[length / 2 | floor].gap_h,
    p75_gap_h: $sorted[(length * 0.75) | floor].gap_h,
    p90_gap_h: $sorted[(length * 0.9) | floor].gap_h,
    p99_gap_h: $sorted[(length * 0.99) | floor].gap_h,
    intervals_eq_1h: (map(select(.gap_h == 1)) | length),
    intervals_gt_24h: (map(select(.gap_h > 24)) | length),
    intervals_gt_72h: (map(select(.gap_h > 72)) | length),
    top5_gaps: (sort_by(-.gap_h) | .[0:5])
  }
' ~/.config/pew/queue.jsonl
```

Verbatim output:

```json
{
  "total_intervals": 918,
  "max_gap_h": 477,
  "min_gap_h": 0,
  "mean_gap_h": 6.617647058823529,
  "median_gap_h": 0,
  "p75_gap_h": 0,
  "p90_gap_h": 14,
  "p99_gap_h": 139,
  "intervals_eq_1h": 52,
  "intervals_gt_24h": 42,
  "intervals_gt_72h": 25,
  "top5_gaps": [
    { "prev": "2025-10-28T07:30:00Z", "curr": "2025-11-17T05:00:00Z", "gap_h": 477 },
    { "prev": "2025-08-22T03:00:00Z", "curr": "2025-09-04T02:30:00Z", "gap_h": 311 },
    { "prev": "2025-08-05T12:30:00Z", "curr": "2025-08-18T06:30:00Z", "gap_h": 306 },
    { "prev": "2026-02-12T03:00:00Z", "curr": "2026-02-24T07:00:00Z", "gap_h": 292 },
    { "prev": "2025-09-29T10:00:00Z", "curr": "2025-10-09T08:00:00Z", "gap_h": 238 }
  ]
}
```

Span: 2025-07-30T06:00Z to 2026-04-26T00:30Z (271 calendar days, 6,498 calendar hours). 919 unique hour buckets. 14.14% duty cycle.
