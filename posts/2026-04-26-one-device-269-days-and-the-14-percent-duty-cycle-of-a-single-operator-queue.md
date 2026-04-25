---
date: 2026-04-26
tags: [pew-insights, device-tenure, telemetry, single-device]
source: pew-insights device-tenure @ 2026-04-25T19:06:44.792Z
---

# One device, 6469 hours, 8.75 B tokens: what single-device tenure tells you that nothing else can

Most operator-analytics writing is about distribution. How many models do
you use, how many sources, how many providers, how is the mass split, how
high is the tail. Distribution lenses are useful exactly because they
multiplex — they take a long flat queue and turn it into a small
comparable table.

But there is one axis where I am, by construction, a population of one:
the device axis. I have one machine I work from. My `pew` queue records
a `device_id` UUID per row, and when I run the `device-tenure` lens at
snapshot `2026-04-25T19:06:44.792Z`, the result is exactly one row. Total
devices: 1. Devices shown: 1. Recently active: 1/1.

That single row is more interesting than it looks.

## The snapshot

```
pew-insights device-tenure
as of: 2026-04-25T19:06:44.792Z    devices: 1 (shown 1)    active-buckets: 908
tokens: 8,745,663,181    minBuckets: 0    sort: span    recentThreshold: 24h
recentlyActive: 1/1
dropped: 0 bad hour_start, 0 zero-tokens, 0 by source filter,
0 by model filter, 0 sparse devices, 0 below top cap
```

And the per-device line:

```
device a6aa6846-9de9-444d-ba23-279d86441eee
first-seen 2025-07-30T06:00:00.000Z
last-seen  2026-04-25T19:00:00.000Z
span-hr    6469.0
active-buckets 908
tokens     8,745,663,181
tok/bucket 9,631,788
tok/span-hr 1,351,934
sources    6
models     15
longest-gap-hr 477.5
hr-since-last  0.1
recent     yes
```

Eight numbers. Each one has an operator interpretation. Let me walk them.

## 6469 span-hours is 269 days, and that is the upper bound on this analysis

The span — first-seen to last-seen on this `device_id` — is **6469 clock
hours**, which is **269.54 days**. That number is a hard upper bound on
every other metric in the row. Tokens-per-span-hour, density, gap stats,
all of them are normalised against 6469. If the device UUID had been
rotated for any reason — OS reinstall, telemetry reset, a fresh
`~/.config/pew` directory — the span would be shorter and every per-hour
ratio would inflate, even though the underlying behaviour was unchanged.

This is the first reason single-device tenure matters: **it is the
denominator for everything else**. The fact that this UUID has survived
269 days of uninterrupted use means I can trust the ratios that lean on
it. Specifically, I can trust **1,351,934 tokens per span-hour** as a
lifetime average burn rate, and I can trust **9,631,788 tokens per active
bucket** as a per-active-hour intensity. Both of those are real
operator-relevant numbers — the first sets a monthly cost ceiling, the
second sets per-session capacity expectations.

## 908 active buckets in 6469 span-hours is a 14.0% duty cycle

This is the key shape number. Out of 6469 hours where the device existed
in the queue, only 908 were active in any sense — meaning at least one
positive-token row landed in that hour-bucket. That is a duty cycle of
**908 / 6469 = 14.04%**.

Operator translation: **across 269 days of calendar life, the AI workload
was active during about 1 hour in 7**. That sounds low, and it is. But
it is bounded by two structural facts:

1. The bucket granularity is one hour. An hour with 60 seconds of
   activity counts the same as an hour with 60 minutes of activity.
2. Sleep, weekends, meetings, and the entire human-attention budget are
   all subtracted from the numerator implicitly.

A 14% duty cycle is what a single-operator workload looks like when you
strip out everything except the hours where some token actually moved.
For comparison, a continuously-running scheduled batch job would sit
near 100%. A pure on-demand human-in-the-loop assistant would sit
somewhere between 5% and 25%, depending on how many time zones the
operator straddles. 14% says: I am one human, in roughly one time
zone, doing actual work.

## 9.6 M tokens per active bucket is the per-hour intensity

Total tokens 8,745,663,181 divided by 908 active buckets gives
**9,631,788 tokens per active hour**. That is a useful per-active-hour
intensity that you cannot get from any other lens. Per-source and
per-model versions of the same number exist — `bucket-intensity` is the
proper distributional form — but the device-tenure version answers the
operator question "when I am working, how many tokens am I moving per
hour, on this machine, on average?".

9.6 M tokens per hour is a substantial number. At a notional blended
rate of $1 per million tokens (low, but order-of-magnitude), that is
about $9.60 per active hour. At $5 per million it is $48 per active
hour. The distance between those two numbers is exactly why the
`cost` lens exists separately and why blended rates are a trap. For
operator planning, the key takeaway is: every hour I am at the
keyboard with this device, on average, costs roughly an hour of senior
engineering labour in tokens. That ratio is right at the edge of
"obviously worth it" and "needs justification".

## 6 sources and 15 models is the surface area

The lens reports `sources: 6` and `models: 15`. This is the
**reachability footprint** of the device: the number of distinct
`source` tags (CLI tools / agents that wrote rows tagged to this
device) and `model` strings (LLM identities those tools talked to)
seen in the entire 269-day window.

Six sources is a small alphabet but a busy one. From cross-referencing
the `source-decay-half-life` snapshot at the same wall clock, the six
known sources include `hermes`, `opencode`, `openclaw`, `codex`,
`claude-code`, and an editor-side helper. Each one is a distinct
process that knows how to write to my queue. Each one is also a
distinct **failure surface** — a buggy writer in any of those six can
silently corrupt the queue for the entire device, because there is
only one device row to corrupt.

Fifteen models is a wider alphabet. From the
`prompt-output-correlation` snapshot at the same instant, those
fifteen include three opus variants, two gpt-5 variants, two sonnet
variants, a haiku, a gemini preview, and an `unknown` bucket. The
`unknown` bucket is itself diagnostic — it means at least one source
emitted rows without a recognisable model string, and the lens has
quarantined those rows under a synthetic label. That is correct
behaviour, and it is also a backlog item: every `unknown` row is a row
where the per-model cost attribution is wrong by construction.

## 477.5 hours is a 19.9-day longest gap

The longest-gap field reports **477.5 hours** — the longest stretch
between consecutive active hour-buckets. That is **19.9 days**. This
is a real gap; my queue genuinely had no positive-token rows tagged
to this device for nearly three weeks at some point in the 269-day
window. Without inspecting the source-decay-half-life table I would
not be able to date the gap precisely, but the snapshot's other
context narrows it: `vscode-copilot` first-seen is `2025-07-30` and
last-seen is `2026-04-20`, while the heavier sources only first appear
in mid-April 2026. The implied story is that this device existed
quietly through 2025 with one low-volume editor-side writer, then in
April 2026 multiple new writers came online almost simultaneously and
the duty cycle jumped.

The 19.9-day gap is the fingerprint of that quiet period. It is also
a useful **denial-of-anomaly** signal: if a future tick reports a
longest-gap of, say, 800 hours, that is novel and worth investigating.
If it reports 477.5 still, the system has merely preserved its known
worst quiet stretch and the rest of the timeline is contiguous enough
that nothing larger has emerged.

## hr-since-last 0.1 is the liveness check

The last field, `hr-since-last 0.1`, says the most recent active
bucket on this device is 6 minutes old at snapshot time. The snapshot
was taken at `19:06:44 UTC`, so the most recent activity was within
the `19:00:00 UTC` bucket. That field is also why `recentThreshold:
24h` and `recentlyActive: 1/1` appear in the header — the threshold
is satisfied with three orders of magnitude to spare.

This is the canary. Any tick where `hr-since-last` exceeds the
operator's working-hour expectation by a meaningful margin is a
signal that the device has stopped writing — either because the
operator is offline, or because all six writers are simultaneously
broken. The latter is rare but possible, and the metric is the
cheapest way to detect it.

## Why "one row" is methodologically valuable

There is a temptation, when a lens reports a single row, to call it
trivial and move on. That is the wrong instinct. The single-device
case has a property that no multi-device case has: **every other lens
in the family is implicitly conditioned on this device**. The
prompt-output correlation, the source decay, the bucket density, the
hour-of-week heatmap, the model-mix entropy — every one of them is
computed against the same 8.75 B tokens that this single row also
sums. The device-tenure row is the **integrator** of the entire
family.

Three concrete consequences:

1. **No cross-device confounding.** Because there is one device, any
   diurnal pattern in the heatmap is the operator's pattern, not a
   superposition of multiple operators with overlapping schedules.
   The hour-of-week entropy of 0.914 (from the same snapshot family)
   is genuinely the entropy of one human's schedule, not a blurred
   average.
2. **No device-axis attribution required.** Cost dashboards do not
   need to apportion mass across machines. The 8.75 B tokens belong
   to one operator's setup, full stop.
3. **Anomaly detection has a perfect baseline.** The `anomalies`
   lens, when run with sigma thresholds against trailing baselines,
   does not need to consider whether a spike is a new device coming
   online. It is always the same device. Spikes are behavioural.

That third point is the one I underweight most often. Multi-device
operator analytics has a category of false-positive spikes that
single-device analytics simply does not have, and the cost of running
one more device for the convenience of using another shell is, in
analytics terms, surprisingly high. Keeping the device count at one
is itself an analytics-quality decision.

## What the row does not tell me

It is worth being explicit about the limits.

- It does not tell me **how many distinct sessions** ran on the
  device. Sessions are a different aggregation grain — that is what
  the `sessions` and `session-lengths` lenses are for. A single
  active hour-bucket can contain one long session or six short ones.
- It does not tell me **which hours were active**. The 908 number is
  a count, not a calendar. The `heatmap` and `hour-of-week` lenses
  are the calendars.
- It does not tell me **which sources or models drove which buckets**.
  The 6 sources and 15 models are surface-area integers; the per-
  bucket attribution lives in `bucket-intensity`,
  `source-tenure`, `model-tenure`, and `bucket-handoff-frequency`.
- It does not tell me **anything about cost**. Tokens are not money.
  The `cost` lens applies a per-model rate table and the device
  identity drops out.

These are all features, not gaps. The lens is a tenure lens; tenure
is what it reports.

## What I will do with this row going forward

Three actions:

1. **Pin the device UUID `a6aa6846-9de9-444d-ba23-279d86441eee` as
   the canonical device for the dashboard family.** Any future writer
   I add must inherit this UUID, not generate a fresh one. A new
   UUID would silently double the device count and break the "one
   row" invariant that other lenses are tacitly relying on.
2. **Add a daily check that `hr-since-last` < 24 during my normal
   working week.** This is the cheapest possible liveness alert and
   it requires no new infrastructure beyond the existing snapshot
   cadence.
3. **Treat any future change in `longest-gap-hr` as a behavioural
   event worth annotating.** If the next snapshot reports 600 hours,
   I want to know which calendar week that gap straddled, because it
   means something happened to my workflow. The number is a slow-
   moving signal precisely because the span keeps growing
   underneath it; a sudden jump is meaningful.

## Summary

- Snapshot: `pew-insights device-tenure` at
  `2026-04-25T19:06:44.792Z`, one device, 908 active buckets,
  8,745,663,181 tokens.
- Span: **6469 hours / 269.5 days** — the integrator of every other
  lens in the family.
- Duty cycle: **14.04% active / 85.96% quiet**, which is the
  signature of a single human operator on a single device.
- Per-active-bucket intensity: **9,631,788 tokens** — substantial,
  and the right number to budget weekly capacity against.
- Surface area: **6 sources, 15 models**, with one
  `unknown` model bucket as a known data-quality backlog item.
- Longest gap: **477.5 hours / 19.9 days**, a real and dated quiet
  period that any future spike must clear to qualify as novel.
- Liveness: **0.1 hours since last active bucket** at snapshot
  time — the canary is green.
- Methodological dividend: keeping the device count at exactly one
  is itself an analytics-quality decision; the entire dashboard
  family inherits its lack of cross-device confounding from this
  single row.
