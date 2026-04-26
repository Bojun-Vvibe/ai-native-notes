---
title: "Weekday-share HHI: the Saturday-Claude and the Monday-GPT axis on 9.35B tokens"
date: 2026-04-27
tags: [pew-insights, weekday, hhi, claude, gpt, telemetry]
slug: weekday-share-hhi-the-saturday-claude-and-the-monday-gpt-axis
---

# Weekday-share HHI: the Saturday-Claude and the Monday-GPT axis on 9.35B tokens

Most of the diurnal analysis I've done on this corpus has lived at hour-of-day
resolution: the 33-second peak, the 15-minute poll clock, the cron fingerprint
in the gap distribution. Hour-of-day is where the obvious shapes are — the
typing rhythm, the launchd schedule, the lunch-hour dip. But there's a coarser
lens that I've been ignoring, and when I finally ran it today the answer was
not at all the boring "everyone works Mon–Fri, dies on weekends" plot I was
expecting. The weekday distribution per model is **load-bearingly different
between providers**, on the same machine, in the same workflow, on the same
set of running days, and the difference isn't a small wobble — it's a peak
that flips by three full days of the week.

The data point: from `pew-insights weekday-share` over
`~/.config/pew/queue.jsonl` (1,580 hour-buckets, 9,345,559,228 total tokens,
as of `2026-04-26T17:47:37.193Z`):

```
model              tokens         peak-day    peak-share  active  hhi
claude-opus-4.7    5,529,209,025  Sat         22.7%       7/7     0.164
gpt-5.4            2,586,441,128  Mon         33.2%       7/7     0.191
claude-opus-4.6.1m 1,108,978,665  Wed         50.5%       5/7     0.324
claude-haiku-4.5      70,717,678  Wed         46.0%       5/7     0.340
```

Global peak across all models: **Mon 21.8%**. Global HHI across the seven
weekdays: **0.154** (uniform reference is 1/7 ≈ 0.143). So at the aggregate
level, the corpus is *almost weekday-uniform*. That global flatness is
hiding the per-model story underneath.

## What the numbers actually say

Look at the top two models — they cover **8.12 billion of the 9.35 billion
tokens**, i.e. 86.8% of all output. They're the workhorses. And they peak
on opposite halves of the week:

- `claude-opus-4.7`: peak **Saturday at 22.7%**, Friday is the *trough* at
  7.0%, weekend share (Sat+Sun) = 35.3%, weekday-five share = 64.7%.
- `gpt-5.4`: peak **Monday at 33.2%**, Tuesday is the trough at 7.7%,
  weekend share (Sat+Sun) = 28.4%, weekday-five share = 71.6%.

If you just stared at "weekend share" you'd say they're similar (35% vs 28%).
But the *shape* is wildly different. Claude opus-4.7 has a smooth bowl with
a single Saturday peak and a Friday minimum — the classic "I work weekends on
side projects, I crash Friday night" curve. Gpt-5.4 has a Monday spike that
is **4.3× its Tuesday floor**, then a slow recovery through Friday, then a
second smaller bump Sunday (17.1%). That's a different distribution entirely.

The HHI numbers quantify it. Both models touch all 7 weekdays
(`active 7/7`), so the concentration isn't a coverage artifact. Yet:

- claude-opus-4.7 HHI = **0.164** (close to the 0.143 uniform floor)
- gpt-5.4 HHI = **0.191** (16% more concentrated than claude)

That 16% relative gap is on the same operator, same week-rhythm, same set
of projects. The difference is *which days the operator routes to which
model*. I don't actually know that I'm doing this consciously. I'd have
told you, before I ran the report, that I use whatever model the harness
defaulted to that morning. The data says something tighter: I reach for
Claude on the weekend, and I reach for the GPT family Monday morning.

## The 50.5%-Wednesday model

The third row is the one that made me stop and re-read.
`claude-opus-4.6.1m` — the long-context Claude — has **50.5% of its
1.1B tokens on a single weekday (Wednesday)**, and **zero tokens
Saturday and Sunday**. Active 5/7 days, HHI = 0.324, more than double
the uniform reference. `claude-haiku-4.5` has the same shape: 46.0%
Wednesday peak, 0.0% on both weekend days, HHI = 0.340.

Two different models, two different vendors of long-context vs fast-cheap,
and they both peak on Wednesday and both vanish on the weekend. That is
not a model preference. That's a *workflow signature*. There's a job —
or a cluster of jobs — that runs midweek, that uses a 1m-context Claude
and a haiku-class fallback, that I do not run on weekends. Looking at
the calendar that's almost certainly the long-form review/summarization
batch I do mid-sprint: long context goes into the 1m model, fast-and-cheap
goes into haiku-4.5 for the per-item judging step. The HHI of 0.324 / 0.340
is just the report telling me "this is a scheduled batch with a sharp peak,
not an interactive thing".

The contrast between the workhorse pair (HHI ~0.16–0.19, full coverage)
and the batch pair (HHI ~0.32–0.34, 5/7 coverage) is the cleanest
visual of "interactive vs scheduled" I've seen in this corpus, and it
falls out of one column.

## The 100%-on-one-day rows aren't noise — they're floors

The bottom of the table has three rows with HHI = 1.000:

```
gpt-5.2     299,605 tok  Thu 100.0%  1/7  hhi 1.000
gpt-5-nano  109,646 tok  Thu 100.0%  1/7  hhi 1.000
gpt-4.1          72 tok  Fri 100.0%  1/7  hhi 1.000
```

These are models that fired on exactly one weekday across the entire
corpus window. The natural read is "tiny — ignore". The more honest read
is that they're A/B probes. `gpt-5.2` at ~300k tokens and `gpt-5-nano`
at ~110k tokens both hitting Thursday only is what an evaluation run looks
like — pick a day, run the comparison, log the data, never come back. The
72-token gpt-4.1 row is the limit case of that: a single tiny ping to make
sure the model name still resolved, then nothing. None of these are
operational; they're experiments. Filtering them out via a min-tokens
threshold would lose the signal that **the corpus contains experiments at
all**, which is something I want to keep visible.

## What's *not* there

A thing I expected to see and didn't:

**No model has a Friday peak in the top-volume tier.** Friday is the
trough for claude-opus-4.7 (7.0%, the only single-digit weekday across
the top two) and a midfield day for gpt-5.4 (13.6%). The "I'll wrap up
the week with a long Claude session" pattern doesn't exist. What does
exist is the opposite: I close out the week mostly on GPT, and pick up
Claude on Saturday morning. The Saturday-Claude axis is real and it's
22.7% — peak day for the largest single model in the corpus.

A thing I expected to see and *did*:

**The Tuesday dip.** gpt-5.4 has Tuesday at 7.7%, a tie for its
absolute weekday low. claude-opus-4.7 has Tuesday at 17.4%, well below
its 19.8% Monday. Tuesday looks meeting-heavy in the calendar, and the
token volume agrees — I touch the keys less on Tuesday. Both models
agree on that, even though they disagree on every other peak.

## Why this is orthogonal to everything else

The angles already covered in this corpus that I checked for overlap:

- **hour-of-day**: heatmap, peak-hour-share, hour-of-day-source-mix-entropy.
  All sub-day. Weekday-share is one level coarser and shows shapes those
  miss (Saturday-peak vs Monday-peak).
- **source-rank-churn / source-debut-recency / source-active-day-streak**:
  per-source calendar dynamics, but on the source dimension, not the
  *model* dimension. The Claude-vs-GPT split on weekdays cuts across
  sources — claude-opus-4.7 is used by claude-code AND openclaw AND
  ide-assistant-A, but the Saturday peak holds across all three.
- **session-start time-of-day**: hour-of-day on session starts. Different
  unit (sessions, not tokens) and different time grain.

The closest neighbour is `peak-hour-share`, which I haven't expanded on
yet. The orthogonal lens is: hour-of-day tells you when the *operator*
is awake and active; weekday tells you what the operator is *trying to
do*. Saturday at 14:00 with Claude is "fix the side project". Monday at
09:30 with GPT is "process the inbox". Same operator, same machine,
different goal — and the model-by-weekday matrix is the cleanest place
that mode-switch shows up.

## Mechanics: how `weekday-share` computes the number

The subcommand pivots `queue.jsonl` rows by `(model, ISO weekday of
hour_start in UTC)`. Token totals aggregate per cell, then the row
shares are computed as cell-tokens / model-row-total. HHI is the sum of
squared shares: `Σ(share²)` over the seven weekday cells. With seven
buckets, the uniform-distribution HHI floor is `7 × (1/7)² = 1/7 ≈
0.143`; the maximum is 1.0 when one weekday has 100% of the model's
mass. Active count is the number of weekday cells with > 0 tokens.

Two pieces of nuance I want to flag:

1. **UTC weekday, not local.** I'm in a UTC-7-ish zone, so the report
   pushes some of my 17:00–23:59 local-time activity into the *next*
   UTC weekday. That probably explains a chunk of the Sunday-night /
   Monday-morning ambiguity for gpt-5.4. The Monday 33.2% peak likely
   absorbs Sunday-evening US-Pacific work.
2. **Token-weighted, not session-weighted.** A single 56M-token Claude
   session on a Saturday weighs 56,000× more than a 1k ping on a
   Tuesday. The Saturday-Claude peak is partially the long-context
   cache-tokens story bleeding through. (See the cache-hit-ratio
   companion post: `claude-opus-4.7` runs at a 294.9% cache-hit ratio,
   which is what a "load a giant corpus once and re-query it on
   Saturday" workflow looks like.)

## Operational use

What I'd actually do with this:

- **Pre-warm the cache on the right day.** If I know
  `claude-opus-4.6.1m` peaks Wednesday 50.5%, I can pre-stage the
  Tuesday-night cache build instead of letting it cold-start at 09:00
  Wednesday. That alone would shave a real chunk off the
  Wednesday-morning latency bar.
- **Schedule expensive batch jobs against off-peak weekdays.** The
  Friday 7.0% claude-opus-4.7 trough is a *resource* — that's the day
  when nothing is competing for the local cache or the rate limits.
  Running the weekly retrospective digest there is free.
- **Use HHI as an alert.** A model whose HHI suddenly jumps from 0.19
  to 0.40 between weeks is either being repurposed for a batch job or
  being abandoned outside one specific use case. Either way it's a
  signal worth noticing.

## The takeaway

On the same machine, same operator, same week, with 9.35B tokens of
evidence, the two largest models in the corpus peak on opposite ends
of the week. That isn't a vendor preference and it isn't a per-model
quirk. It's a workflow fingerprint: I default to the GPT family when
I'm in inbox-mode (Monday), and I default to Claude when I'm in
side-project-mode (Saturday). The HHI of 0.164 vs 0.191 is small in
absolute terms but it's the *direction* of the asymmetry that matters.
And the long-context Claude with HHI 0.324 and 0% weekend coverage is
sitting there as proof that there's a third mode entirely — a
midweek scheduled-batch mode — that the hour-of-day reports miss
because it's a *which-day* phenomenon, not a *which-hour* one.

Source data: `~/.config/pew/queue.jsonl` (1,580 buckets,
9,345,559,228 total tokens, generated
`2026-04-26T17:47:37.193Z`). Reproduce with:

```
node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js weekday-share
```

The previous post in the series is `f3c45dd` (commits-per-tick
bimodality — 40 legacy + 178 steady + 1 twelve-commit hapax). The
weekday lens is the next coarser ring out from that one.
