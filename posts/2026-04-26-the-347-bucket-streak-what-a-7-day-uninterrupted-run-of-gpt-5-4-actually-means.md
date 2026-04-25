---
title: "The 347-bucket streak: what a 7-day uninterrupted run of gpt-5.4 actually means"
date: 2026-04-26
tags: [pew-insights, bucket-streak, telemetry, model-tenure, observability]
---

# The 347-bucket streak: what a 7-day uninterrupted run of gpt-5.4 actually means

A `pew-insights bucket-streak-length` snapshot taken at `2026-04-25T18:07:11.049Z` against the local queue (`~/.config/pew/queue.jsonl`, 1,292 active 30-minute buckets, 8.71 B tokens, 15 distinct models) shows one model with a streak so long it deserves its own post:

```
model                 active-buckets  streaks  longest  mean-streak  longest-start (UTC)       longest-end (UTC)
--------------------  --------------  -------  -------  -----------  ------------------------  ------------------------
gpt-5.4               400             19       347      21.05        2026-04-18T13:00:00.000Z  2026-04-25T18:00:00.000Z
claude-opus-4.7       300             41       90       7.32         2026-04-23T21:30:00.000Z  2026-04-25T18:00:00.000Z
unknown               56              4        49       14.00        2026-04-23T15:00:00.000Z  2026-04-24T15:00:00.000Z
claude-opus-4.6.1m    167             44       15       3.80         2026-04-15T05:00:00.000Z  2026-04-15T12:00:00.000Z
```

`gpt-5.4` racked up **347 consecutive 30-minute active buckets** between `2026-04-18T13:00:00.000Z` and `2026-04-25T18:00:00.000Z`. That's 173.5 hours, or 7 days 5 hours 30 minutes. Every single half-hour window in that range had at least one event with positive token mass attributed to that model.

This post is about what that number actually means, what it doesn't mean, and why the second-place streak (`claude-opus-4.7` at 90 buckets, 45 hours) is qualitatively different even though it's also a "long uninterrupted run".

## What `bucket-streak-length` measures

The definition is mechanical: a "streak" is a maximal run of consecutive active buckets where each step is exactly one bucket-width apart (here, 30 minutes, inferred from the queue). One missed bucket — one half-hour with zero attributed tokens to that model — and the streak ends. The next active bucket starts a new streak.

So 347 buckets at 30 minutes per bucket is 173.5 hours of uninterrupted half-hourly activity. There is no half-hour window in that span where `gpt-5.4` produced zero tokens for me. Sleep, meals, walks, meetings — none of those gaps exceeded 30 minutes. Or, more precisely, none of them exceeded the 30-minute window boundaries in a way that left a fully-empty bucket.

This is an important distinction. A 25-minute gap that straddles a bucket boundary still leaves both buckets non-empty. A 35-minute gap that perfectly straddles a boundary creates a single empty bucket and breaks the streak. The metric is sensitive to *bucket alignment*, not strictly to *gap length*. A model with the exact same gap distribution as another can have a wildly different longest-streak depending on where its gaps fall.

This is good and bad. Good: the metric is computationally trivial and reproducible. Bad: small phase shifts in the data can produce big shifts in the streak number. Don't read too much into a 5% difference in streak length; do read into a 4× difference like 347 vs 90.

## Why 347 is a real signal, not an artifact

The two highest streaks in the table are 347 and 90. That's a 3.86× ratio. For that to be a phase-shift artifact, you'd need `claude-opus-4.7` to have nearly the same actual gap distribution as `gpt-5.4` but with most of its gaps unluckily landing on bucket boundaries. Possible in theory; not credible at this magnitude.

A better explanation comes from the `streaks` and `mean-streak` columns. `gpt-5.4` has 19 streaks total with a mean of 21.05 buckets — so the typical `gpt-5.4` streak is 10.5 hours long. `claude-opus-4.7` has 41 streaks with a mean of 7.32 buckets, or 3.66 hours. So `gpt-5.4` is breaking less frequently *and* running longer on average. The 347 isn't an outlier within `gpt-5.4`'s pattern — it's the upper end of a distribution that is structurally longer-tailed than `claude-opus-4.7`'s.

The `streaks-per-active-bucket` ratio makes this concrete:

- `gpt-5.4`: 19 streaks / 400 active buckets = 0.0475 break-rate per bucket
- `claude-opus-4.7`: 41 / 300 = 0.137 break-rate per bucket

`gpt-5.4` breaks roughly one-third as often per active bucket. That is a structural difference in usage pattern, not a phase-shift artifact.

## What the 347 actually represents

Three plausible mechanisms can produce a 347-bucket streak:

1. **A long-running background job** that emits something every < 30 minutes. A persistent daemon, a scheduled re-evaluation loop, anything with a `setInterval` shorter than the bucket width.
2. **Continuous human use across the full week** with a bucket width forgiving enough to cover overnight idle. 30 minutes is not actually that forgiving; a typical sleep window of 7–8 hours would create 14–16 empty buckets and break the streak instantly.
3. **A heterogeneous mix of human and automated traffic** where the automated traffic fills in the human idle windows.

For `gpt-5.4` specifically, mechanism 3 is the most likely. The total span (173.5 hours) is too long to be sustained pure human use without breaking — even accounting for a generous time-zone interpretation, you'd need to be active or have a process active in *every* half-hour for over a week. The realistic explanation is that there's at least one continuously-running background process generating `gpt-5.4` traffic on a sub-30-minute cadence, with human-generated bursts riding on top.

This matches the model's `active-buckets` count: 400 total active buckets across the model's full tenure, with a 347-bucket longest streak. That means 87% of `gpt-5.4`'s lifetime active-buckets are inside this one streak. The remaining 53 buckets are distributed across 18 other streaks — average streak length 2.94 buckets — which is the signature of sporadic earlier use, followed by a regime change starting `2026-04-18T13:00:00.000Z` where something started generating continuous traffic.

## What the 90 actually represents

`claude-opus-4.7`'s 90-bucket streak (45 hours, ending at the same `2026-04-25T18:00:00.000Z` the snapshot was taken — i.e. the streak is still ongoing) is a different beast. 41 streaks, mean 7.32 buckets (3.66 hours), longest 90 (45 hours). The shape is "frequent multi-hour sessions, occasionally one stretches into a full weekend".

The 90-bucket run starts `2026-04-23T21:30:00.000Z` — a Thursday evening. The end at `2026-04-25T18:00:00.000Z` is Saturday afternoon. That's a recognisable pattern: a multi-day deep-focus stretch crossing two nights. The fact that it didn't break across either overnight period is interesting — either the operator was up at every half-hour mark (unlikely for two consecutive nights), or there's also a background-traffic generator on this model, or the two overnight gaps happened to fall just inside bucket boundaries and the metric got lucky. Honestly, I'd bet on a combination of all three.

The crucial difference from `gpt-5.4`: the *typical* `claude-opus-4.7` streak is 7.32 buckets. The 90-bucket run is 12× the model's mean — a real outlier within its own distribution. For `gpt-5.4` the longest is 16.5× its mean (347 / 21.05), but its mean is itself already very high. So:

- `gpt-5.4`: high mean, modest outlier ratio, structurally always-on.
- `claude-opus-4.7`: lower mean, high outlier ratio, occasionally goes very long.

These are two distinct usage signatures, and the streak metric distinguishes them cleanly even though both top the leaderboard.

## What `mean-streak` and `streaks` reveal that `longest` doesn't

Most people will read this report and skip straight to the `longest` column. Don't. The trio `(streaks, longest, mean-streak)` is the actual information. Some patterns that emerge from the full table:

- **`unknown` (49 longest, mean 14, 4 streaks)**: small total, but each streak is substantial. This is the signature of a tightly-bounded process that runs for ~7 hours then stops. Probably a specific daemon I haven't tagged yet.
- **`claude-opus-4.6.1m` (15 longest, 3.80 mean, 44 streaks)**: many short streaks, modest longest. This is normal interactive use without any persistent background hook.
- **`gemini-3-pro-preview` (7 longest, 1.76 mean, 21 streaks)**: streaks rarely make it past one bucket. Pure exploratory poking.
- **`gpt-5` (15 longest, 2.74 mean, 62 streaks across 170 buckets)**: many sessions, mostly short. Note the longest run is from `2025-09-18` — old data, before this model was effectively retired in my workflow.

The single most useful derivative statistic is the **continuity ratio**: `longest / active-buckets`. It captures "how concentrated is this model's lifetime into one stretch?":

- `gpt-5.4`: 347/400 = 0.868 — almost everything is in one run
- `unknown`: 49/56 = 0.875 — same shape, smaller scale
- `claude-opus-4.7`: 90/300 = 0.300 — substantial recent run, plus a long history
- `claude-opus-4.6.1m`: 15/167 = 0.090 — diffuse, no dominant streak
- `gpt-5`: 15/170 = 0.088 — diffuse, no dominant streak

Models above 0.5 are dominated by one continuous regime. Models below 0.2 are spread across many sessions. The 0.2–0.5 band is the interesting middle: real ongoing use that isn't reduced to a single bursty event.

## Calibration: what these numbers say about my own work

A few things this single snapshot reveals about my actual workflow that I would not have surfaced otherwise:

1. **There is a persistent automated `gpt-5.4` consumer running since `2026-04-18T13:00:00.000Z`.** I should know what it is. The streak length tells me the cadence is sub-30-minute. A grep through my launchd plists and crontab is on the to-do list.

2. **`claude-opus-4.7` is the active interactive model right now.** It has the most sub-week streaks (41 streaks) and the most recent long streak (still ongoing as of snapshot time). It's the one I'm typing into in real time.

3. **Several models are inert.** `gpt-4.1`, `gpt-5-nano`, and `gpt-5.2` each have exactly one bucket of activity total across their entire tenure. These are exploratory pokes that never went anywhere. I should consider archiving or excluding them from the dashboard view.

4. **`openclaw`'s 366 buckets (from the sibling `tail-share` report)** must be split across multiple models, since the largest single-model bucket-count here is `gpt-5.4`'s 400. This is an interesting cross-report consistency check — `openclaw` is a multi-model source.

## A small operational lesson

Streak metrics are great at one specific thing: surfacing structural regime changes that get washed out in volume-based dashboards. A token-count chart for `gpt-5.4` would show the model gradually picking up volume over the last week. The streak metric snaps to attention with a single number — 347 — that immediately tells you "something started producing continuous traffic on April 18 at 13:00 UTC and hasn't stopped". That's a much more actionable observation than "tokens trending up".

The opposite is also true: streak metrics are bad at quantifying *how much* a model is doing. A 347-bucket run could be 1 token per bucket or 1 million tokens per bucket — the streak length is the same. Always pair streak views with mass views (`bucket-intensity`, `model-tenure`, `tail-share`) to get the full picture.

## Reproducing

```bash
cd ~/Projects/Bojun-Vvibe/pew-insights
node dist/cli.js bucket-streak-length
```

Useful flags:

- `--source <name>` to scope to one source.
- `--min-buckets <N>` to filter sparse models.
- `--sort length|mean|count` to re-sort the table.

The snapshot I cited was at `2026-04-25T18:07:11.049Z` with default flags. The 347 number for `gpt-5.4` will keep growing until something in the pipeline misses a half-hour window. The first time it breaks will itself be a useful event — that's the point at which whatever started on April 18 either changed cadence or stopped. I'll know within 30 minutes of it happening, which is a nice property for a one-line CLI report to have.
