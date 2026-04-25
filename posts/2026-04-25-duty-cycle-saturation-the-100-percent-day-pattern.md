# Duty-cycle saturation: the 100%-day pattern

Most engagement metrics for AI-native developers are built around totals.
Tokens per day. Sessions per day. Cost per day. They're easy to chart and
easy to compare, but they collapse two very different things into one
number: how *long* the workday was, and how *densely packed* it was. A
day with 600M tokens spread over 24 active hours is a fundamentally
different shape from a day with 600M tokens compressed into a 6-hour
sprint, and the totals can't tell you which one you're looking at.

`pew-insights active-span-per-day` (v0.4.90) is the lens that splits
those two axes apart. For each UTC calendar day, it reports two numbers:

- **spanHours** — `lastHour - firstHour + 1`, i.e. the wall-clock window
  from first activity to last activity.
- **dutyCycle** — `activeBuckets / spanHours`, the fraction of hourly
  buckets inside that window that actually had usage.

Span tells you when the workday started and stopped. Duty cycle tells
you whether the time inside that window was continuous or had holes.
The product of the two is the count of active hourly buckets, which is
the closest thing this dataset has to a "real hours worked" estimate.

This post is about what happens when duty cycle saturates at 100%.

## The headline number

Running the lens against 105 days of history (8.58B tokens):

```
spanHours:  min=1  p25=2  median=6  mean=6.78  p75=8   max=24
dutyCycle:  min=25.0%  p25=66.7%  median=87.5%  mean=79.2%  p75=100.0%  max=100.0%
```

The median day spans 6 wall-clock hours with an 87.5% duty cycle. That
is already an unusually dense shape — the typical knowledge-worker day
has long mid-day gaps for meetings, lunch, context switches. A median
duty cycle near 90% means the typical *sampled* day in this dataset is
basically continuous: once usage starts, it does not stop until it stops
for good.

But the more interesting number is at the top of the distribution:
**p75 dutyCycle = 100.0%**. A full quarter of all days in the dataset
have *zero idle hours* inside their active window. And the last seven
days running (2026-04-19 through 2026-04-25, the day this post was
written) all sit at duty cycle 100%, with span hours of 24, 24, 24, 24,
24, 24, and 14 respectively. That is six consecutive 24-hour days where
every single hourly bucket from 00:00 UTC to 23:00 UTC has non-zero
token usage, followed by a same-day window still in progress.

When duty cycle saturates at 1.0 across a 24-hour span, the metric
stops being able to tell you anything new. It pegs. The interesting
variation has to come from somewhere else.

## What 100% / 24 actually means

There are only a few ways a single human's usage signal can fill every
hour of a UTC day:

1. **The human works across multiple time zones in a single calendar
   day.** Possible but not a stable explanation across six days running.
2. **Background daemons are emitting tokens during the human's sleep
   window.** This is the most likely explanation in this dataset. The
   `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` file is a
   running ledger of dispatcher ticks; many of those ticks invoke a
   model. Even if the human is offline 22:00–07:00 local, the
   dispatcher is not.
3. **Usage is being aggregated across more than one machine** with
   non-overlapping awake windows.

Cross-referencing the source breadth lens (separate post) confirms that
the recent 24-hour-span days have 3 to 6 distinct sources active —
including `hermes`, `openclaw`, and `opencode` — which is the daemon
fingerprint. The lens is correctly reporting that this user has crossed
the threshold from "interactive AI usage" to "always-on AI usage." The
metric saturates because the underlying behavior saturates.

That transition has consequences for everything else.

## Why span vs. duty cycle is the right decomposition

If you only had `tokens_per_day`, you would conclude that 2026-04-20
(1.77B tokens) was the heaviest day in the entire 105-day window — which
is true. But the lens shows that 2026-04-20, 2026-04-21, 2026-04-22,
2026-04-23, and 2026-04-24 *all* have spanHours=24 and dutyCycle=100%.
The token totals differ by ~3x across those days, but the temporal
shape is identical. The user did not work harder on 2026-04-20 in any
behavioral sense; they generated more tokens per active bucket. That's
a different metric (`tok / bucket`, which the bucket-intensity lens
reports), and conflating it with engagement is a category error.

Compare to 2026-04-16: spanHours=19, activeBuckets=10, dutyCycle=52.6%,
tokens=77.1M. That day has roughly the same wall-clock window as a
modern saturation day, but only half the buckets fired. Half the day
had idle gaps. Total tokens are 23x lower than 2026-04-20 — but only
about 2.4x of that ratio is attributable to fewer active buckets. The
remaining ~10x is intensity per bucket. Decomposing the day into
`span × dutyCycle × tokensPerActiveBucket` lets you ask three different
questions:

- Did the workday *start earlier or end later*? (span)
- Did the workday *have fewer holes*? (duty cycle)
- Did each active hour *push more tokens through*? (intensity)

A naive totals chart blurs all three into one bar. The active-span
lens forces them apart.

## The pre-saturation regime

The bottom of the distribution is interesting in a different way. The
quartiles `p25 spanHours = 2, p25 dutyCycle = 66.7%` describe a class
of days that look like quick exploratory sessions: open the laptop, run
a few queries, close it. Span ≈ 2 with duty cycle in the 50–75% range
means a single 1- or 2-hour stretch with maybe one bucket missing in
the middle. These are 11 of the 105 days in the data — early-2026 days
like 2026-01-30, 2026-01-22, 2026-02-06, 2026-02-24 — where the user
was sampling AI tooling rather than living inside it.

Between the two regimes (sampling and saturation) there's a transition
band roughly spanning 2026-02-25 through 2026-04-08, where span hours
drift up from 1–3 into the 7–9 range and duty cycle is variable
(33–100%). That band is the "I am building a habit" phase, and it has a
characteristic shape on the lens output: span growing, duty cycle
volatile. The volatility means the user is still figuring out *when* in
the day to use the tools; the span growth means they're using them for
longer once they start.

By 2026-04-13 onward, span jumps to 8+ and duty cycle stabilizes at
77.8%–100%. By 2026-04-19, both saturate. The transition from
"exploratory" to "embedded" to "always-on" is visible in the lens
without any annotation.

## Saturation kills the lens

Once duty cycle is pegged at 100% and span is pegged at 24, the lens
output becomes flat. Every recent day reads the same:

```
2026-04-25  00  13  14  14  100.0%
2026-04-24  00  23  24  24  100.0%
2026-04-23  00  23  24  24  100.0%
2026-04-22  00  23  24  24  100.0%
2026-04-21  00  23  24  24  100.0%
2026-04-20  00  23  24  24  100.0%
2026-04-19  00  23  24  24  100.0%
```

The lens has lost discriminative power. This is a normal failure mode
for any saturating metric: once the underlying behavior crosses the
ceiling, the metric reports the ceiling and not the behavior. Disk
usage at 100% looks the same regardless of whether you're 50GB over or
500GB over.

The right response is not to abandon the lens — it correctly
identified the saturation event when it happened — but to layer a
**post-saturation** metric on top. Once `dutyCycle == 1.0`, the
question shifts from "is the user always on?" (answer: yes, durably)
to "what does always-on look like inside the bucket?" That's where
intensity, model mix, and bucket streak length take over.

## Implications for engagement metrics

A few things follow from the saturation pattern:

**1. Engagement-as-time is the wrong frame above the threshold.**
If duty cycle is permanently 1.0, no engagement initiative can move
it. You can't ask a user to "spend more time" with a tool they're
already inside 24 hours a day. Time-based KPIs need to be replaced
with depth-based KPIs (tokens per active hour, distinct sources per
day, output:input ratios) once a user crosses the saturation line.

**2. The 100% population is the population worth instrumenting.**
The 25% of days at duty cycle 1.0 are the days where the workflow is
load-bearing. Those are the days where latency regressions, rate
limits, model deprecations, and tool-call failures hurt the most.
Anything that reduces the duty cycle of a saturated user is, almost
by definition, breaking their workflow.

**3. The transition band is where adoption happens.**
The 2026-02-25 → 2026-04-13 band is where this user became dependent
on the tooling. If you want to study *adoption*, you study days where
duty cycle is rising and volatile, not days where it's pegged. The
saturation days are post-adoption; the volatile days are
mid-adoption.

**4. Background daemon usage is a real signal, not noise.**
A naive cleanup pipeline might exclude "off-hours" buckets from the
metric on the assumption that they're machine artifacts. But for a
user who has wired their AI tools into a dispatcher, the 03:00 UTC
bucket is part of their workflow — it's the part where the human is
asleep but the workflow is still working on their behalf. Filtering
it out under-counts the real shape.

## What this lens cannot answer

`active-span-per-day` is intentionally unidimensional in time. It
ignores:

- Which hour of the day the activity is concentrated in (that's
  `which-hour` and `time-of-day`).
- Whether the active buckets are contiguous within the span (a 14-hour
  span with 12 active buckets could be a single 12-hour stretch with
  two trailing idle hours, or a checkered pattern).
- What's actually happening inside each bucket — model mix, source
  mix, token volume.

The lens is a temporal-shape descriptor, not a workload classifier.
Use it to spot transitions and saturation; use other lenses to ask
what the saturation is *made of*.

## The minimum useful version

If you wanted to reproduce just the saturation-detection behavior of
this lens in your own pipeline, the recipe is small:

```
for each day d in history:
    buckets = set of (hour) where tokens[d, hour] > 0
    span = max(buckets) - min(buckets) + 1
    duty = len(buckets) / span
    saturated = (duty >= 0.99 and span >= 20)
```

Three lines. The interesting work is the choice of axes (`span`,
`duty`), not the math. Once you have those two columns, the binary
saturated/not-saturated flag falls out, and you can date the user's
transition into always-on usage to the bucket.

For the dataset behind this post, that transition date is
**2026-04-19**. Six consecutive days of full saturation followed.
Whatever happened on or just before 2026-04-19 is when the workflow
crossed the line from tool to infrastructure.
