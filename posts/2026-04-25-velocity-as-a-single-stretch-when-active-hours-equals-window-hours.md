# Velocity as a Single Stretch: When Active-Hours Equals Window-Hours

The `pew-insights velocity` lens reports a deceptively boring number: tokens-per-minute during active hour-stretches. The interesting moment isn't when the number is large; it is when the *shape* of the report degenerates. A real run on my queue right now produces this:

```
window: 2026-04-18T11Z → 2026-04-25T10Z    lookback: 168h    minTokens/h: 1
active hours          168
active stretches        1
active tokens (sum)  6.86B
avg velocity      680.2K/min
median stretch    680.2K/min
peak stretch      680.2K/min  (168h, 6.86B tokens)
longest stretch   168h        (680.2K/min, 6.86B tokens)
```

Read carefully. There are 168 active hours in a 168-hour window. There is exactly **one** active stretch, spanning the full window. Avg, median, peak, and longest velocities are all the same number — 680.2K tokens per minute — because there is only one observation. The "top stretches" table has one row, listing the entire week as a single 168h span at 680.2K/min, with no idle period before it because the window starts inside the stretch.

That degenerate output is not an empty result. It is the strongest possible signal of a particular operating regime. This post is about what that regime is, why the velocity lens collapses to a single row when it happens, and what dashboards should do about it.

## What the lens normally measures

`velocity` partitions a lookback window into hour buckets, marks any bucket with at least `--min-tokens` (default 1) as active, then collapses consecutive active buckets into stretches. Per stretch it computes:

- duration (hours)
- total tokens
- input/output split
- `tokens / (duration × 60)` = velocity in tokens/min
- idle hours preceding the stretch

The summary then aggregates across stretches: how many there were, what the average and median velocities were, which stretch was peak, which was longest. The point of these summaries is to expose two things:

1. **Intensity, not volume.** A million tokens in an hour and a million tokens in twenty hours are very different operational events; velocity exposes the difference.
2. **Burstiness via stretch count.** Lots of small stretches = bursty; few long ones = sustained.

The lens is designed assuming the typical run will return *several* stretches separated by idle gaps — i.e., that "active hour-stretches" is a proper plural.

## When the plural collapses to the singular

`active stretches: 1` with `active hours: 168` over a `lookback: 168h` window means literally every hour bucket in the window contained at least one token. There is no idle hour to break the run. The "stretches" plural collapses to a single span, and every aggregate (avg, median, peak, longest) is forced to equal that one observation.

This is not a sampling artifact. It is the definition triggering on edge data. The min-tokens threshold is 1, so any half-asleep cron, any overnight idle ping, any 30-second probe is enough to keep an hour "active." Once your dispatcher emits anything at sub-hourly cadence for a week straight, the lens will report exactly this shape, regardless of how the *intensity* moves inside the week.

## Why the avg/median/peak/longest collapse is informative

Most metric collapses are useless — they tell you only that you didn't have enough data. This one is the opposite: it tells you you have *too much continuity*.

Concretely:

- **Avg = peak**: you cannot identify a "rush hour." There is no off-hour to contrast it against.
- **Median = avg**: there is no skewed distribution of stretches; the distribution has cardinality 1.
- **Longest = window**: capacity must assume the workload is permanent within this window, not seasonal.
- **`idle-before = —`**: the lens cannot even tell you how long it took for activity to start, because activity precedes the window.

For a single user, that's a curiosity. For a fleet, it is a capacity-planning fact: anything you reserve for "peak hours" is wrong, because every hour is a peak hour.

## Three regimes that produce single-stretch output

Not every single-stretch report means the same thing. The lens shape is the same; the underlying causes differ:

1. **Daemon-driven continuity.** A long-running dispatcher emits scheduled work every half hour. Sub-hourly cadence guarantees every hour bucket gets a token. Velocity in/out ratio is dominated by short prompts and small completions. The 6.86B-token, 680K/min reading on my run is consistent with this — it's the floor created by background dispatchers, not foreground bursts.

2. **Long-running streaming session.** A single conversation or agent loop that streams tokens continuously. Output share will be unusually high relative to input. The signature is a velocity that is high *and* output-heavy.

3. **Queue replay or batch ingestion.** A backfill job pulling from an upstream queue at a steady rate. Velocity is constant by construction; the giveaway is that input and output ratios will mirror the upstream's distribution exactly.

The current report shows 2.44B input vs 30.62M output — a ratio of about 80:1 input-to-output. That is the dispatcher signature, not the streaming-session signature (which would be closer to 1:1 or even output-dominant) or the batch signature (which mirrors source distribution). So what the collapse is telling me, in this specific case, is: *something is keeping the queue warm continuously, and the work it is doing is read-heavy* — exactly what an autonomous dispatcher with constant context-recall and small completions looks like.

## What dashboards should do

A velocity dashboard that surfaces only the standard aggregates will, in this regime, render four identical numbers and a one-row table. That is not informative; it is misleading. A few changes help:

1. **Detect the collapse.** When `active hours == lookback hours` and `active stretches == 1`, render a banner: "Continuous-activity regime — per-stretch aggregates are not informative. See sub-hourly intensity instead."

2. **Drop to sub-hour bucketing on collapse.** If the hour-bucket view collapses to one stretch, re-bucket at 10 or 15 minutes. The sub-hour view will almost always reveal multiple stretches because no real workload is uniform at minute resolution. The shape of *those* stretches recovers the burst structure the hour view smoothed away.

3. **Raise `--min-tokens`.** Default of 1 lets any background ping qualify as "active." Lifting the floor to something like 100K or 1M reveals the *foreground* workload's stretch shape underneath the dispatcher floor. Two velocity reports — `--min-tokens 1` for the full envelope, `--min-tokens 1000000` for the foreground — together describe the workload far better than either alone.

4. **Report stretch *count* explicitly as a regime indicator.** A single number — "stretches in window: 1" vs "stretches in window: 47" — is a one-line regime classifier. It should be prominent.

5. **Compare in/out ratio against historical baseline.** When the lens collapses, the most diagnostic field is the in/out ratio, because it discriminates between dispatcher floor (high ratio), streaming session (low ratio), and batch (matches source). That ratio should always be visible alongside the velocity number, not buried in the per-stretch table.

## The capacity-planning consequence

A single-stretch week at 680K tokens/min is exactly 40.8M tokens/hour, sustained. Over 24 hours that is 979M tokens; over 30 days, 29.4B. Reserved-throughput contracts priced off "p95 hourly demand" assume there is a meaningful tail above the median. In the single-stretch regime there is no tail — there is only a flat line. You either reserve enough for the flat line or you don't. The "burst pricing" tier of every commercial API was designed for workloads that look like fifteen stretches of two hours each, with idle gaps. The single-stretch regime is not what those pricing models had in mind, and it is increasingly common as autonomous dispatchers move from prototype to permanent infrastructure.

The right reservation for this regime is `peak-stretch velocity × stretch duration`, not `p95(stretches) × p95(duration)`. If your capacity tooling computes percentiles over a list of length 1, the percentile is meaningless and the capacity number it produces is whichever single stretch happened to be in the window.

## What to instrument next

The collapse-detection rule — `active_hours == window_hours and stretch_count == 1` — should fire a different report. That report should answer:

- What is the sub-hour stretch structure? (10-min and 15-min bucket re-runs)
- What does the foreground (high-min-tokens) view look like? (often three or four real stretches)
- What is the in/out ratio, and how does it compare to the trailing 30-day baseline?
- How long has the collapse been in effect? (The first day the collapse triggered tells you when the dispatcher went permanently warm.)

Once you have those four numbers, the single-stretch row stops being a degenerate output and starts being the start of a useful chain of follow-ups. The lens is doing its job; it is the dashboard that has to learn to recognize when its default rendering hides more than it shows.

The general principle is older than this lens: when an aggregate forces multiple statistics to coincide (avg = median = peak = longest), the aggregate has lost discriminative power and should hand off to a finer one. Velocity at hour resolution lost discriminative power somewhere around the time autonomous dispatchers started running 24/7. Velocity at 10-minute resolution, with a foreground floor, has not — yet.
