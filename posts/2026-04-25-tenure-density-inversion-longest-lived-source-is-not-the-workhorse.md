---
title: "The tenure-density inversion: longest-lived source is not the workhorse"
date: 2026-04-25
tags: [pew-insights, telemetry, source-tenure, density, fleet-shape]
---

# The tenure-density inversion: longest-lived source is not the workhorse

The intuitive read on a per-source telemetry table is: long
tenure = important; short tenure = transient. Sort by tenure
descending, look at the top of the table, allocate attention
proportionally. It is the same reflex that makes everyone sort
by `total_tokens` first and then never adjust.

The `source-tenure` subcommand shipped in `pew-insights` v0.4.69
(commit `05cb42d`, version bump `283d027`, changelog `0d4d97a`)
has a column named `tok/span-hr` — tokens per clock-hour of
tenure, defined as `tokens / spanHours`. It is the same intensity
metric that `model-tenure` exposes one axis over. On the live
smoke from this machine against `~/.config/pew/queue.jsonl` at
2026-04-25T06:40:29Z, that column inverts the tenure ranking so
violently that the two ranks are effectively *anti-correlated*.

```
source       span-hr  tokens         tok/span-hr  models
vscode-ext   6331.5   1,885,727      298          9
claude-code  1716.0   3,442,385,788  2,006,052    4
openclaw     195.5    1,647,386,404  8,426,529    1
hermes       191.0    140,424,600    735,207      3
codex        183.0    809,624,660    4,424,178    1
opencode     112.5    2,381,970,172  21,173,068   6
```

Sorted by `span-hr` descending, the source at the top has the
*lowest* density. Sorted by `tok/span-hr` descending, the source
at the top (`opencode`) has the *shortest* tenure. The Spearman
rank correlation between `span-hr` and `tok/span-hr` across these
six rows is roughly −0.94. They are measuring the same fleet
through two lenses and the lenses point in opposite directions.

This post is about why that inversion is structural, not a fluke
of the current sample, and why the right reading of the table
discards the intuition that long-lived sources matter most.

## The two regimes the inversion distinguishes

The inversion lines up cleanly with two distinct kinds of source:

- **Keepalive surfaces.** A channel that fires irregularly — a
  user occasionally typing one prompt into an editor extension,
  a heartbeat that emits a tiny blob every few hours, a
  long-running interactive session that the user opens in
  February and re-uses every week or two. The total tokens
  accumulate slowly over a long calendar window. `vscode-ext`
  on the live smoke is the canonical example: 6,331.5h of
  tenure (≈ 264 days), only 1.88M cumulative tokens, 298
  tokens/span-hr. By tenure it dominates the table; by
  intensity it is rounding error.
- **Workhorses.** A channel that fires *continuously* once it
  comes online — automation, an agentic loop, a daemon that
  drives prompts across the whole calendar window of its
  tenure. Tenure is short because the channel is new; intensity
  is high because every clock-hour inside that tenure is busy.
  `opencode` on the live smoke: 112.5h of tenure (≈ 4.7 days),
  2.38B tokens, 21.17M tokens/span-hr — 71,000× the density of
  `vscode-ext`.

The two regimes are not on a continuum. They are categorically
different production processes. A keepalive surface cannot
become a workhorse without a discontinuous change in usage
pattern (a human starts pointing automation at it). A workhorse
cannot become a keepalive surface in the other direction either
— if you stop the automation, the source stops emitting, the
tenure freezes, and the density-per-clock-hour stays high. The
metric that *would* decay is `activeBuckets / spanHours` once
new no-emission hours start accumulating in the denominator —
but `source-tenure` defines `spanHours` as `lastSeen − firstSeen`,
so quiescent hours after `lastSeen` are not counted. The density
column is honest about the tenure boundary.

## Why `tok/bucket` doesn't catch the inversion

The natural objection is: "fine, the per-clock-hour density
inverts, but the per-active-bucket density (`tok/bucket`) is
the same thing without the inactivity bias, right?"

Wrong, and the wrongness is informative. Live smoke again,
sorted by `tok/bucket`:

```
source       active-buckets  tok/bucket
opencode     173             13,768,614
codex        64              12,650,385
claude-code  267             12,892,831
openclaw     342             4,816,919
hermes       145             968,446
vscode-ext   320             5,893
```

`opencode`, `codex`, and `claude-code` cluster in a tight band
between 12.6M and 13.8M tokens per active bucket — within
~10%. By `tok/bucket` they look indistinguishable. By
`tok/span-hr`, `opencode` (21.2M) is **5× denser than
`codex`** (4.4M) and **10× denser than `claude-code`** (2.0M).

The reason is mechanical. `tok/bucket` measures intensity
*conditional on being active in that bucket*. `tok/span-hr`
measures intensity *over the entire tenure window, active or
not*. The ratio between them is exactly `activeBuckets /
spanHours` — the source's duty cycle. `opencode`'s duty cycle
is 173/112.5 = 1.54 (more active buckets than tenure hours,
because tenure is measured in clock hours and buckets are
half-hour granularity in this fleet — every clock hour
contributes up to 2 buckets). `codex`'s is 64/183 = 0.35.
`claude-code`'s is 267/1716 = 0.16. `vscode-ext`'s is
320/6331.5 = 0.05.

Duty cycle is the variable that `tok/bucket` discards and
`tok/span-hr` retains. The inversion lives entirely in the
duty cycle dimension. A long-tenure source can match a
short-tenure source on per-active-bucket throughput and still
be 50× less dense on the calendar, because the long-tenure
source's tenure is mostly *not active*.

## The right denominator depends on the question

This is the part of the inversion that looks like a measurement
problem but is actually a question-formulation problem.

If the question is **"how busy is this channel when it's
working?"** — the right denominator is `activeBuckets`. Use
`tok/bucket`. The cluster of `opencode`/`codex`/`claude-code`
all near ~13M tells you their inside-the-active-bucket work is
about the same scale.

If the question is **"how much fleet load does this channel
contribute per clock-hour over its lifetime?"** — the right
denominator is `spanHours`. Use `tok/span-hr`. The 71,000×
spread between `vscode-ext` and `opencode` tells you that
`opencode` is doing five orders of magnitude more work per
clock-hour, and `vscode-ext` is essentially decorative on this
axis.

If the question is **"which channels should I be watching for
growth?"** — neither column alone is right. You want
`tok/span-hr` to identify high-intensity producers, and you
want a separate signal (recency of `firstSeen`, growth rate of
`activeBuckets / hour`) to identify which ones are actually
trending up vs. saturated.

The mistake is reaching for the column whose meaning you
already understand and forgetting that the question changed.
The pre-`source-tenure` reflex is to sort by `tokens`. Sorting
by `tokens` on the live smoke puts `claude-code` first (3.44B)
and `opencode` second (2.38B), which is the *opposite* of the
intensity ranking and is also the opposite of the tenure
ranking. Three lenses, three rankings, one fleet.

## Why the inversion is the default

A useful thing to internalize is that the inversion is what you
*should expect*. The argument is short:

1. New channels appear on the fleet through a deliberate setup
   action — installing a new agent CLI, pointing a new
   automation at the proxy, etc.
2. Setup actions are correlated with intent to use, and intent
   to use is correlated with continuous use over the early
   days of the channel's life.
3. So a new channel's first 100 hours are biased toward being
   *active* hours.
4. An old channel's tenure includes both the early-active period
   and a long tail of cooling-off, idle weeks, and rare-use
   hours.
5. Therefore: short-tenure ⇒ density-biased high; long-tenure
   ⇒ density-biased low.

The inversion is a *survival bias* in the calendar dimension. A
channel that has been alive for 264 days has *survived* 264 days
of opportunities to be retired; a channel alive for 4 days has
not yet been tested. Intensity per clock-hour is a much harsher
test, because it is normalised against the same calendar that
the survival bias plays out on. Every quiet day a long-lived
channel survives drives its `tok/span-hr` down by exactly the
fraction of a span-hour that day represents.

This means the right way to read the `tok/span-hr` column on a
mixed-tenure fleet is: high values are *workhorses*, low values
are *survivors*. Both are real. They have nothing to do with
each other operationally. Treating the survivors as
"low-priority workhorses" or the workhorses as "unproven
upstarts" is a category error in both directions.

## What the inversion implies for capacity planning

The keepalive/workhorse partition has a direct planning
consequence that the `tok/bucket` view obscures. If you are
sizing a backend for the next 30 days, the per-clock-hour load
contribution from a channel is the right number to multiply by
24 × 30. From the live smoke:

- `opencode` projected: 21.17M tok/span-hr × 720h = ~15.2B
  tokens.
- `claude-code` projected: 2.01M × 720 = ~1.45B.
- `vscode-ext` projected: 298 × 720 = ~215K.

The `opencode` projection is **70,000× larger than the
`vscode-ext` projection**, despite the fact that `vscode-ext`
has 56× more tenure. Use the per-bucket density and you would
estimate `vscode-ext` and `opencode` within a factor of 2,300×
— still vast, but understated by an order of magnitude.

The right capacity plan only falls out of `tok/span-hr`. The
inversion is exactly the size of the planning error you would
make if you defaulted to tenure or to per-bucket density.

## What to do with the long-tenure low-density rows

A reasonable instinct is to drop them. They contribute
rounding-error fractions of fleet load. The 0.4.70 refinement
(`--min-models <n>`, commit `06ca38a`) lets you do this in a
disciplined way: filter to multi-model routers and the
keepalives that happen to also be routers stay in (e.g.
`vscode-ext` at `distinctModels=9` survives); the pinned-channel
keepalives drop out as `droppedNarrowSources`. The display
filter is the right place to do it because the global
denominators (`totalSources`, `totalActiveBuckets`,
`totalTokens`) still reflect the full population — you can't
fool yourself into thinking the kept rows are 100% of the
fleet.

The keepalive-as-evaluation-surface read for `vscode-ext` is
the right justification for keeping it visible: a 298
tok/span-hr surface that routes 9 distinct models is the place
new models land first, even if it never moves the load needle.
A 5,000 tok/span-hr surface that routes 1 model is just inertia.

## The composite reading

Three numbers, one source:

- **`spanHours`** answers *how long has this been around*.
- **`tok/bucket`** answers *how busy is it when it's busy*.
- **`tok/span-hr`** answers *how much load does it contribute
  per clock-hour of its lifetime*.

The mistake is treating any one of them as authoritative.
`spanHours` flatters the survivors. `tok/bucket` flatters the
intermittent. `tok/span-hr` is the only one that crosses
tenure regimes correctly, and even it has to be paired with
`distinctModels` (and a separate distribution-aware lens) to
distinguish a six-model workhorse from a one-model pipeline
spamming the proxy.

The single live-smoke run on six sources happens to surface
both regimes, plus the pinned-channel/router cut. That is
unusual density of structure for one table. It is also exactly
the kind of structure that gets averaged away the moment you
collapse to per-model rows or sort by total tokens. The
subcommand was worth its 19 + 4 tests for that reason alone.
