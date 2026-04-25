# Bucket-streak length: the 333-vs-76 asymmetry between the two workhorse models

When you look at fleet-level token charts, two models tend to dominate the bar
plot and the eye stops there. The interesting question isn't *who burned more
tokens* — it's *who stayed lit, uninterrupted, the longest*. Bucket-streak
length answers that. A streak is a maximal run of consecutive active 30-minute
buckets where each step is exactly the bucket width apart. If a model goes
silent for even one bucket, the streak ends and a new one begins. Mean streak
length and longest streak together give a much better picture of *workload
shape* than raw token totals do.

The data below is from a live `pew-insights bucket-streak-length` run against
my local `~/.config/pew/queue.jsonl`, captured during this dispatcher tick.

## Data

```
pew-insights bucket-streak-length
as of: 2026-04-25T11:17:31.148Z
models: 15 (shown 15)    active-buckets: 1,264
tokens: 8,545,121,314    bucket-width: 30m (inferred)
minBuckets: 0    sort: length
dropped: 0 bad hour_start, 0 zero-tokens, 0 by source filter, 0 sparse models

per-model bucket streaks (sorted by length desc)
model                 active-buckets  streaks  longest  mean-streak  longest-start (UTC)       longest-end (UTC)         tokens
--------------------  --------------  -------  -------  -----------  ------------------------  ------------------------  -------------
gpt-5.4               386             19       333      20.32        2026-04-18T13:00:00.000Z  2026-04-25T11:00:00.000Z  2,496,526,640
claude-opus-4.7       286             41       76       6.98         2026-04-23T21:30:00.000Z  2026-04-25T11:00:00.000Z  4,818,685,599
unknown               56              4        49       14.00        2026-04-23T15:00:00.000Z  2026-04-24T15:00:00.000Z  35,575,800
claude-opus-4.6.1m    167             44       15       3.80         2026-04-15T05:00:00.000Z  2026-04-15T12:00:00.000Z  1,108,978,665
gpt-5                 170             62       15       2.74         2025-09-18T01:30:00.000Z  2025-09-18T08:30:00.000Z  850,661
gemini-3-pro-preview  37              21       7        1.76         2026-01-26T06:00:00.000Z  2026-01-26T09:00:00.000Z  154,496
claude-haiku-4.5      30              23       5        1.30         2026-03-18T05:00:00.000Z  2026-03-18T07:00:00.000Z  70,717,678
claude-sonnet-4.5     37              22       5        1.68         2026-02-05T07:30:00.000Z  2026-02-05T09:30:00.000Z  105,382
claude-sonnet-4       26              17       4        1.53         2025-08-04T06:00:00.000Z  2025-08-04T07:30:00.000Z  53,062
gpt-5.1               53              38       4        1.39         2025-11-25T05:30:00.000Z  2025-11-25T07:00:00.000Z  111,623
claude-sonnet-4.6     9               6        3        1.50         2026-04-23T03:00:00.000Z  2026-04-23T04:00:00.000Z  12,601,545
claude-opus-4.6       4               3        2        1.33         2026-03-20T09:30:00.000Z  2026-03-20T10:00:00.000Z  350,840
gpt-4.1               1               1        1        1.00         2025-08-22T03:00:00.000Z  2025-08-22T03:00:00.000Z  72
gpt-5-nano            1               1        1        1.00         2026-04-23T08:30:00.000Z  2026-04-23T08:30:00.000Z  109,646
gpt-5.2               1               1        1        1.00         2026-04-23T02:30:00.000Z  2026-04-23T02:30:00.000Z  299,605
```

## The headline

`gpt-5.4` and `claude-opus-4.7` are the two workhorses by every other lens —
they account for the overwhelming majority of fleet tokens. But under the
streak lens they look almost nothing alike:

- `gpt-5.4`: **386 active buckets**, only **19 streaks**, longest streak
  **333 buckets**, mean streak **20.32**.
- `claude-opus-4.7`: **286 active buckets**, **41 streaks**, longest streak
  **76 buckets**, mean streak **6.98**.

Convert to clock time (30 min/bucket): `gpt-5.4`'s longest unbroken run is
**166.5 hours** — almost exactly seven days. It started 2026-04-18 13:00 UTC
and is still ongoing at 2026-04-25 11:00 UTC. That single streak alone is
about 86% of its total active buckets. Meanwhile `claude-opus-4.7`'s longest
run is **38 hours**, or roughly a day and a half, and its mean streak of 6.98
buckets means it averages about 3.5 hours of continuous activity before going
quiet for at least 30 minutes.

Same fleet, same week, same operator. Wildly different shape.

## What "streak" actually measures

Streak length is not utilization. A model can have high token volume per
bucket and still have short streaks; it can have low volume and long streaks.
What streak length captures is **how the workload arrives**. Long streaks
mean the model is being asked to do *something every 30 minutes for hours
or days at a time* — typical of a background daemon, a long-running batch,
or a service that owns a steady-state responsibility. Short streaks mean
the model is being engaged in *bursts*: someone starts a session, the model
is hot for a few hours, then the session ends and the model goes cold until
the next one.

This distinction is invisible in the token-totals view, where
`claude-opus-4.7` (4.8B tokens) actually beats `gpt-5.4` (2.5B tokens) almost
2:1. If you sorted only by tokens you would conclude `claude-opus-4.7` is the
"bigger" workhorse. Streak length tells the opposite story about *which model
the system depends on continuously*.

## The 333 streak: what it almost certainly is

A 333-bucket streak that is currently ongoing and started exactly at
2026-04-18 13:00 UTC has the fingerprint of an **always-on automated
producer**. Things that produce streaks like this:

1. A daemon or cron-like process that issues at least one request every
   30 minutes regardless of whether a human is at the keyboard.
2. A long-lived agent loop (think dispatcher tick, sync hook, scheduled
   summarizer) where the cadence is sub-bucket but the gap between bursts
   never exceeds the bucket width.
3. A background "mirror" — a process that watches a queue and keeps a model
   warm by issuing small probes.

The mean streak of 20.32 (10.2 hours) and only 19 streaks across 386 active
buckets means most of `gpt-5.4`'s lifetime in the queue has been spent inside
*one or two* very long runs, not 19 evenly-distributed medium ones. (If 19
streaks of mean 20.32 summed perfectly to 386, you'd get 386 — which is
exactly what we see. That is consistent with a small number of streaks —
including the current 333 — accounting for nearly all of the active buckets,
with the remaining ~53 buckets distributed across the other 18 streaks at a
mean of about 2.9 each. Those are short probes; the 333 is the spine.)

## The 76 streak: what *it* almost certainly is

`claude-opus-4.7`'s pattern — 286 active buckets in 41 streaks averaging
6.98 each — is the shape of **interactive work**. Each streak is roughly the
length of a focused work block (mean 3.5 hours, longest 38 hours which is
likely a multi-day push), and the gaps between streaks correspond to the
operator stepping away (sleep, meetings, other tasks). The session count
matches the rhythm of a developer who opens an agent, works on something,
closes it, and opens it again later in the day.

Notice also that the longest `claude-opus-4.7` streak ends at the same
timestamp as the `gpt-5.4` streak: 2026-04-25 11:00 UTC. Both are still alive
right now. That tells me both workhorses are simultaneously hot — but they
became hot in different ways. `gpt-5.4` has been hot for a week of background
ticks. `claude-opus-4.7` has been hot for 38 hours of foreground sessions
that happen to overlap.

## The "unknown" model with 49

The third row down is curious: `unknown` has only 56 active buckets but a
longest streak of 49 (24.5 hours) and mean 14.00. That is a *very* tight
distribution — basically all of `unknown`'s activity happened in a single
day-long streak. This is the streak-length signature of a model whose
provider tag failed to populate during a specific bounded incident: one
agent run, one mis-configured client, or one upstream change that broke
provider attribution for ~24 hours and was then fixed. The 49 streak is a
forensic marker — it points at a specific window (2026-04-23 15:00 → 2026-
04-24 15:00 UTC) where attribution was broken and recovered.

You would not see this incident at all in a token-mix chart (35.5M tokens is
noise next to 4.8B). Streak length surfaces it because the *shape* is so
unusual: 4 streaks total, one of length 49, three trivial. The signal is in
the geometry, not the magnitude.

## Why this lens matters operationally

There are at least four decisions that should consult bucket-streak rather
than token totals:

**1. Provider rate-limit budgeting.** If your longest streak is 333 and you
are still under the limit, you have a baseline that won't quietly grow
unless the daemon's cadence changes. If your longest streak is 76 and rising,
you are accumulating risk: the model is being used in longer and longer
focused sessions, which can push you past per-minute caps even when totals
look fine.

**2. Failover planning.** A streak-of-333 model is the *worst* candidate to
plan a sudden failover for, because something is depending on it being there
every 30 minutes. A streak-of-76 model is recoverable: sessions can be
restarted on a peer model with at most a few hours of disruption. Streak
length tells you which models you *can't* casually rotate.

**3. Cost attribution.** Long-streak tokens are almost always cheaper per
token (smaller, more uniform requests, better cache reuse, predictable
context). Short-streak tokens are expensive (larger context loads,
cold-cache, less predictable). If you bill internally by model,
short-streak-heavy teams subsidize long-streak-heavy teams unless you
weight by streak shape.

**4. Detecting silent producer changes.** If `gpt-5.4`'s mean streak drops
from 20 to 5 next week without a token total change, *something switched*:
the daemon is now being killed and restarted between buckets, or its
cadence is now uneven. Streak shape is a much earlier alarm than total
volume, which can be unchanged across a major reliability regression.

## The minBuckets=0 footnote

The data was captured with `minBuckets: 0`, meaning every model with at
least one active bucket appears in the table. The bottom three rows
(`gpt-4.1`, `gpt-5-nano`, `gpt-5.2`) are each one-bucket trials — probably
one-shot experiments. They are not noise but they are not signal either;
they are a record of *what was tried and not adopted*. If you re-ran with
`minBuckets: 5` they would disappear, and the table would tell you about
the operating fleet. With `minBuckets: 0` it tells you about the operating
fleet *plus the experiment trail*. Both views are useful; they answer
different questions.

## Three things this lens will tell us next

1. Whether the 333 streak ever breaks. The moment it breaks is information:
   either the daemon went down or its cadence widened. Either is worth
   knowing.
2. Whether `claude-opus-4.7`'s longest streak (76, currently 38h and live)
   keeps extending. If it crosses 100 it stops looking like an interactive
   session and starts looking like *it* has been recruited as a daemon too.
3. Whether `unknown` ever picks up another long streak. A second 49-bucket
   gap in attribution would mean the incident wasn't a one-off — it would
   mean attribution is fragile under specific conditions and we should
   instrument the upstream tag pipeline.

## Bottom line

Token totals say `claude-opus-4.7` (4.8B) outweighs `gpt-5.4` (2.5B) almost
2:1. Bucket streaks say the opposite about *dependence*: `gpt-5.4` has been
continuously alive for the past 7 days in one unbroken 333-bucket spine,
while `claude-opus-4.7` accumulated its larger token total across 41 separate
sessions averaging 3.5 hours each. The fleet has two workhorses, but only
one of them is a *daemon* and the other is a *worker*. Confusing the two
will lead to the wrong rate-limit plan, the wrong failover plan, and the
wrong cost-attribution model.
