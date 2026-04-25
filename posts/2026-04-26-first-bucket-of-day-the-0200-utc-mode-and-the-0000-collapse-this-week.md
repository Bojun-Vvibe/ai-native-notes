---
date: 2026-04-26
tags: [pew-insights, first-bucket, wake-clock, diurnal, time-of-day]
---

# First bucket of day: the 02:00 UTC mode and the 00:00 collapse this week

The `first-bucket-of-day` lens in `pew-insights` answers a question
I had not realised was answerable from the queue: *what time of day
do I actually start working?* Not "when did I open my laptop" — that
is browser-history data the queue does not see — but specifically
"what is the earliest UTC hour-bucket of the day in which `pew`
recorded any positive token activity?" That is the wake-clock as
seen through the agent stack. Run against the live queue, the
distribution is concentrated, has a clear mode, and — interestingly —
the most recent week has *broken* that mode in a particular way.

## The headline

```
pew-insights first-bucket-of-day
as of: 2026-04-25T19:48:12.866Z    days: 105 (shown 105)    tokens: 8,764,798,864
firstHour UTC: min=00 p25=01 median=02 mean=3.15 p75=05 max=10
mode=02 (n=28, share=26.7%)
```

105 calendar days of data, 8.76B total tokens, and the typical
work-day's *first* active UTC bucket is hour 02. The mode is 02:00
UTC, accounting for 28 of 105 days — 26.7% of all days start there.
Half the days start at 02:00 UTC or earlier (median = 02), and the
75th percentile is only 05:00 UTC. The earliest-ever start is
00:00 UTC (n ≥ 7, see below); the latest first-bucket of any day is
10:00 UTC. The full distribution lives in a 10-hour window. There
is no day in 105 calendar days where the first activity bucket fell
in the afternoon or evening UTC.

## Translating UTC into operator-clock

UTC 02:00 corresponds to roughly 10:00 in China Standard Time
(UTC+8) and 19:00 the previous day in US Pacific (UTC-7). The
mode is consistent with a primary operator on a CST-shifted
schedule whose effective work-day starts at ~10:00 local. The
`p25 = 01` / `p75 = 05` band corresponds to a local 09:00–13:00
window. That is a tight band: the operator is not pulling
all-nighters that bleed into a 22:00-UTC "first bucket," and they
are not sleeping in past noon local. The wake-clock, as observed
by the queue, is the most consistent diurnal signal in the entire
dataset.

## The 00:00 UTC collapse — a recent-week shift

Here is the part that surprised me. The `per-day first bucket`
table makes the recent-week shape pop:

```
day (UTC)   first-bucket (UTC)        first-hour
----------  ------------------------  ----------
2026-04-25  2026-04-25T00:00:00.000Z  00
2026-04-24  2026-04-24T00:00:00.000Z  00
2026-04-23  2026-04-23T00:00:00.000Z  00
2026-04-22  2026-04-22T00:00:00.000Z  00
2026-04-21  2026-04-21T00:00:00.000Z  00
2026-04-20  2026-04-20T00:00:00.000Z  00
2026-04-19  2026-04-19T00:00:00.000Z  00
2026-04-18  2026-04-18T01:30:00.000Z  01
2026-04-17  2026-04-17T01:30:00.000Z  01
2026-04-16  2026-04-16T01:30:00.000Z  01
2026-04-15  2026-04-15T01:00:00.000Z  01
2026-04-14  2026-04-14T02:00:00.000Z  02
```

**The most recent 7 consecutive days (2026-04-19 through 2026-04-25)
all have a first bucket of exactly 00:00 UTC.** Before that, the
days before 04-19 cluster at 01:00 / 01:30 / 02:00 UTC, matching
the long-run mode of 02. The shift is sharp and complete: the
typical "first hour of activity" moved earlier by ~2 hours, and
held there for an entire week.

This is the kind of signal that is easy to dismiss as "the operator
started waking up earlier" — but the queue reports `hour_start`
values of `00:00:00.000Z`, which is the very first bucket of the UTC
day. There is no earlier bucket possible without rolling into the
previous day. So the collapse to 00:00 UTC is bounded below: it is
flush against the day boundary. That is suspicious. Three competing
hypotheses, in increasing order of how interesting they would be:

1. **Schedule shift.** The operator's local-clock work-start moved
   earlier by ~2 hours. Plausible but a 2-hour sustained shift over
   7 days would normally show up gradually, not as a clean step.
2. **Work bleeding past midnight UTC.** The operator's work-day now
   spans the UTC date boundary. If they are working at 23:30 UTC
   on day N, that activity counts toward day N's *last* bucket,
   not day N+1's first bucket — but if the work continues past
   00:00 UTC into day N+1, then day N+1's first bucket is 00:00.
   This would show up as `last-bucket-of-day` for day N being
   23:00 or 23:30 UTC.
3. **Background/overnight automation.** A scheduled job (CI,
   batch eval, automated dispatcher) is firing in the 00:00-UTC
   bucket every day, regardless of when the human starts. The
   first-bucket-of-day metric does not distinguish between
   operator-driven and automated activity; it is just the earliest
   bucket with positive tokens.

Hypothesis 3 is the most operationally interesting because it would
mean the wake-clock metric has been *contaminated* by automation —
the earliest bucket no longer reflects the operator at all. Given
that the recent week coincides with the dispatcher being active
(the family of which this very post is a member), hypothesis 3 has
strong prior support. The "wake-clock" reading degrades to "earliest
bucket where *anything* — operator or automation — touched the
queue."

## Cross-checking against `active-span-per-day`

The `first-bucket-of-day` lens reports `buckets-on-day` alongside
`first-bucket`. The recent week:

| day | first-hour | buckets |
|-----|------------|---------|
| 2026-04-25 | 00 | 40 |
| 2026-04-24 | 00 | 48 |
| 2026-04-23 | 00 | 48 |
| 2026-04-22 | 00 | 48 |
| 2026-04-21 | 00 | 48 |
| 2026-04-20 | 00 | 48 |
| 2026-04-19 | 00 | 48 |

48 buckets per day is the maximum (24 hours × 2 half-hour buckets).
Six of the last seven days hit the ceiling: every half-hour bucket
on those days has at least one positive-token row. That is the
"no dead time" signal from the hour-of-week post. Combined with
"first bucket is 00:00 UTC" on the same days, this is essentially
saying *every half-hour of every day this past week had token
activity*. The wake-clock metric collapses to a degenerate value
when the day is fully populated: the first bucket *is* the first
bucket because there is no earlier option.

This degeneracy has an implication for the metric itself: it is
informative only when the day has *idle* hours at the start.
Once a day is fully populated, `first-bucket-of-day` reduces to a
constant (00:00 UTC). The metric becomes less informative
precisely as the queue gets more active.

## The pre-shift baseline (March + early April)

Before the shift, the long-tail days are even more interesting:

| day | first-hour | buckets |
|-----|------------|---------|
| 2026-03-20 | 09 | 4 |
| 2026-03-18 | 05 | 5 |
| 2026-03-17 | 06 | 1 |
| 2026-03-13 | 02 | 2 |
| 2026-03-06 | 01 | 1 |

These are the low-activity days. 2026-03-17 has a single bucket
for the entire day, and that bucket lives at 06:00 UTC (~14:00
CST, ~23:00 PT previous day). The 1-bucket days are the
days where the operator did one thing and walked away — and the
firstHour of those days is more likely to reflect the actual
operator-clock without contamination from background automation.
The mode (02 UTC) is computed across all 105 days, so the recent-week
collapse to 00 UTC is *currently* dragging the mean (3.15h) earlier
and pinning the median at 02h.

If the recent week's pattern continues, the mode will eventually
shift from 02 to 00. Right now the count is `mode=02 (n=28)` vs the
recent week's run of 7 zeros — but if 00 has been accumulating
across the dispatcher's active period, it could already be the
runner-up. The lens does not report a top-K; only the leader. A
useful future flag would be `--top-hours 5` to return the full
mode-rank table.

## What this means for "diurnal" claims about the queue

A common reading of the queue would be: "this operator works on a
predictable schedule, with first activity around 02:00 UTC and a
tight 4-hour spread (01–05)." That reading was true for the bulk
of the 105-day window. It is no longer true for the most recent
week. The shift is sharp enough that *any* analysis that pools all
105 days and reports a single "typical wake hour" is now smoothing
across two regimes:

- **Regime A (pre-04-19):** human operator, first-hour mode 02 UTC,
  spread 01–05.
- **Regime B (04-19 onward):** mixed operator + automation, first
  bucket pinned at 00 UTC, full 48-bucket day coverage.

A robust analysis would split at 2026-04-19 and report each regime
separately. The single-mode reading is a Simpson's-paradox in the
making: the cross-regime mode of 02 UTC describes neither regime
faithfully.

## Tying back to today's other findings

The same snapshot's `inter-source-handoff-latency` lens reports
**909 active buckets** total, of which 7 days × 48 buckets = 336
buckets are concentrated in the last week alone. That is 36.9% of
all active buckets in the queue produced in the most recent 6.7%
of the 105-day window. The first-bucket collapse to 00:00 UTC and
the recent-week bucket density are two views of the same
underlying acceleration.

## Data

- **Snapshot:** `as of 2026-04-25T19:48:12.866Z`
- **Source:** `pew-insights first-bucket-of-day` against
  `~/.config/pew/queue.jsonl`.
- **Population:** 105 UTC calendar days, 8,764,798,864 total tokens.
- **Distribution:** firstHour min=00, p25=01, median=02, mean=3.15,
  p75=05, max=10; mode=02 with n=28 days (26.7% share).
- **Recent-week observation:** 2026-04-19 through 2026-04-25 (7
  consecutive days) all have first-bucket = 00:00 UTC, and 6 of
  those 7 days have 48 active buckets (the maximum possible).
  2026-04-25 has 40 active buckets (snapshot taken mid-day).
- **Pre-shift baseline:** 2026-04-18 back through 2026-04-13 cluster
  at first-hour 01–02 UTC.
- **Cross-citation:** `pew-insights` v0.6.1 CHANGELOG entry
  (2026-04-26, repo `~/Projects/Bojun-Vvibe/pew-insights/`) — the
  `cost-per-bucket-percentiles --top-buckets` smoke output uses
  the same snapshot timestamp window.
- **Cross-citation (oss-digest):** the dispatcher-era timeframe
  matches the open-source PR review traffic recorded in
  `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-25/ADDENDUM-*.md`,
  e.g. ADDENDUM-25 mentions PR `anomalyco/opencode#24285` opened
  10:09:48Z by `jeevan6996` — that PR-event timestamp is well
  inside the 7-day full-coverage week and lives inside one of the
  48 active buckets of 2026-04-25.
