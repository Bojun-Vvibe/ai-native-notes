# Bucket Streak Length: The Marathon-vs-Sprinter Model Axis

When you stare at a token-spend leaderboard, models look like they fall on a one-dimensional ladder: who burned the most tokens this week. That ladder hides a second axis that, once you start looking for it, turns out to be at least as important for capacity planning, model selection, and even debugging: **how long can a given model stay continuously hot?** Not "how much did it consume" but "how unbroken was its presence in your fleet's hour-by-hour timeline."

The pew-insights `bucket-streak-length` lens makes this axis legible. It defines an *active bucket* as any UTC half-hour window in which a model recorded at least one token, and a *streak* as a maximal run of consecutive active buckets where each step is exactly one bucket-width apart. The headline number per model is `longestStreak`: the largest such run ever observed. The companion numbers are `streaks` (how many distinct runs there were), `mean-streak` (average run length), and `activeBuckets` (the raw count of hot half-hours).

## What the data actually shows

A run on my own queue right now (`bucket-streak-length --min-buckets 5`) returns eleven models across 1,260 active buckets and roughly 8.53 billion tokens. Sorted by longest streak descending:

| model | active-buckets | streaks | longest | mean-streak | tokens |
|---|---:|---:|---:|---:|---:|
| gpt-5.4 | 384 | 19 | **331** | 20.21 | 2.49B |
| claude-opus-4.7 | 284 | 41 | 74 | 6.93 | 4.81B |
| unknown | 56 | 4 | 49 | 14.00 | 35.6M |
| claude-opus-4.6.1m | 167 | 44 | 15 | 3.80 | 1.11B |
| gpt-5 | 170 | 62 | 15 | 2.74 | 0.85M |
| gemini-3-pro-preview | 37 | 21 | 7 | 1.76 | 0.15M |
| claude-haiku-4.5 | 30 | 23 | 5 | 1.30 | 70.7M |
| claude-sonnet-4.5 | 37 | 22 | 5 | 1.68 | 0.11M |
| claude-sonnet-4 | 26 | 17 | 4 | 1.53 | 53K |
| gpt-5.1 | 53 | 38 | 4 | 1.39 | 0.11M |
| claude-sonnet-4.6 | 9 | 6 | 3 | 1.50 | 12.6M |

The leaderboard tells one story. The streak axis tells a different one.

## The marathon: gpt-5.4 at 331 buckets

`gpt-5.4` shows a longest streak of **331 consecutive 30-minute buckets**. That is 165.5 hours — almost exactly seven calendar days — of unbroken half-hourly presence, from `2026-04-18T13:00Z` to `2026-04-25T10:00Z`. Across 384 total active buckets, it amassed only 19 streaks total, with a *mean* streak of 20.21. In other words: when this model is on, it stays on for a long time. The ratio `longest / activeBuckets ≈ 0.86` is unusually high — most of its lifetime is concentrated inside a single, monolithic activity span.

This is the signature of a **dispatcher-pinned workhorse**: something is keeping the model warm at sub-hourly cadence essentially every half hour for a week straight. Not human typing — a person can't generate that envelope. Cron-like dispatchers, idle pings, or a long-running daemon that draws from this model on a fixed schedule will produce exactly this shape.

## The sprinter: claude-opus-4.7 at 74

`claude-opus-4.7` burned **the most absolute tokens** in the table — 4.81B vs gpt-5.4's 2.49B — and yet its longest streak is only 74 buckets (37 hours). It racks up 41 distinct streaks across 284 active buckets, with a mean streak of just 6.93. Ratio of longest to total: ~0.26.

The story this shape tells is the opposite of the marathon: high *intensity* during active windows, but the model does not own the half-hour grid. Something else (gpt-5.4, presumably, or human idle time) sits in the buckets between Opus's bursts. Opus is **bursty and dominant when active, but episodic across the timeline**.

If I only ranked by tokens, I'd have concluded "Opus 4.7 is the workhorse." Streak length corrects that to "Opus 4.7 is the heavy hitter; gpt-5.4 is the workhorse keeping the lights on between Opus bursts." Those are different operational facts and they imply different capacity decisions.

## Why mean-streak is the underrated column

Most readers' eyes will jump to `longest`. But `mean-streak` is the column that actually classifies a model's *normal* behavior, because it is robust to a single freak run.

- `gpt-5.4`: longest 331, mean 20.21 → even the typical streak is 10+ hours. Sustained.
- `claude-opus-4.7`: longest 74, mean 6.93 → typical streak ~3.5 hours. Bursty.
- `claude-opus-4.6.1m`: longest 15, mean 3.80 → typical streak under 2 hours. Pure spike behavior.
- `gpt-5.1`, `claude-sonnet-4`, `gemini-3-pro-preview`: mean ≤ 1.76 → essentially "one bucket on, gone" probes. These are evaluation runs or one-shot experiments, not deployed traffic.

Notice how the bottom of the table (`gpt-5`, `gpt-5.1`, `claude-sonnet-4.5`, `claude-sonnet-4`) all show 50–170 active buckets but mean streaks under 3. They are everywhere on the calendar — but never anywhere for long. That is the fingerprint of a *probe model*: pinged often enough to stay in the ledger, but never made primary.

## The geometry: streaks vs activeBuckets

A useful mental model is to plot each row as a point in the (`activeBuckets`, `longestStreak / activeBuckets`) plane.

- **Top right** (high active, ratio → 1): the marathon quadrant. gpt-5.4 sits here.
- **Top middle** (high active, ratio ~0.25): the heavy-burst quadrant. claude-opus-4.7.
- **Bottom right** (high active, ratio → 0): the relentless-prober quadrant. gpt-5 (170 buckets, longest 15, ratio ~0.09) lives here. It is everywhere, never sustained.
- **Bottom left** (low active, low ratio): noise. The 9-bucket / longest-3 row for `claude-sonnet-4.6` is here — too little data to classify.

This 2×2 is doing real work. It separates "this model carries a continuous workload" from "this model gets paged in for spikes" from "this model gets evaluated occasionally" — three categories that the raw token leaderboard cannot distinguish.

## What `streaks` count tells you about the dispatcher

`streaks` is the count of distinct runs. High streaks + low mean-streak = the dispatcher is *touching* this model often but *committing* to it rarely.

`gpt-5` shows the most extreme version: 62 streaks across 170 active buckets means roughly one streak every 2.7 buckets — i.e., the typical run lasts barely longer than the single bucket that started it. Compare to gpt-5.4: 19 streaks across 384 buckets, ratio ~20. When gpt-5.4 starts, it commits.

This becomes a debugging lever. If you see a model whose `streaks` count balloons week over week while `mean-streak` collapses, something has changed in the dispatcher: a router is now flapping back to this model on tiny one-off requests instead of keeping it pinned. That is a behavior regression even if the absolute token count looks flat.

## What to do with this signal

1. **Capacity reservations should track longest streak, not total tokens.** A model that runs for 165 unbroken hours at p50 throughput needs a different rate-limit posture than a model that bursts for 37 hours and goes quiet. Reserved-throughput contracts that get sized off weekly totals will over-pay for the bursty model and under-pay for the marathon one.

2. **Use mean-streak as a regression alarm.** If `mean-streak` for your primary model drops below half of its trailing 14-day median, something has fragmented: tool errors, retry loops, or a router change. The token total can stay constant while the *shape* changes catastrophically.

3. **Streaks count is a router-flap indicator.** A sudden 2× jump in `streaks` for a single model — without a corresponding jump in `activeBuckets` — means the model is being toggled in and out faster than before. That is almost always a router or fallback misconfiguration, not real demand.

4. **The `unknown` row is a tagging bug, not real traffic.** 56 active buckets with a 49-bucket longest streak and only 35.6M tokens is one continuous low-volume probe whose model field never got populated. Streak shape exposes telemetry gaps the same way it exposes traffic shapes.

5. **Probe models should be culled.** Anything with mean-streak < 2 and activeBuckets < 50 is contributing nothing operational; it is just polluting the dashboard. Either promote it to a real evaluation slot or stop sending it traffic.

## The orthogonality claim

The headline finding from this single snapshot — claude-opus-4.7 leads on tokens but gpt-5.4 leads on streak length, by a factor of 4.5 in `longest` and 2.9 in `mean-streak` — generalizes. **Token volume and temporal continuity are orthogonal.** A model can be the biggest spender without being the most continuous; it can be the most continuous without being the biggest spender. Treating them as the same axis is one of the easiest ways to misjudge what your fleet is actually doing.

The bucket-streak lens is cheap to compute (one pass over the queue, group by `(model, hourBucket)`, walk the sorted bucket list per model counting consecutive runs). It runs in seconds on hundreds of millions of rows. There is no reason not to put it on the same dashboard as the spend leaderboard. The two together give you a 2×2 that answers a question the spend column alone cannot: not just "who is expensive" but "who is *load-bearing*."

That distinction — expensive vs load-bearing — is exactly the one that matters when a model gets deprecated, throttled, or repriced. The expensive model is a budget problem you can plan around. The load-bearing model is the one whose absence breaks the dispatcher.
