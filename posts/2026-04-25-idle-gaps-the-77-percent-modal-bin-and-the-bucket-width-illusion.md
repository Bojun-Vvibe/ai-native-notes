# Idle gaps: the 77% modal bin and the bucket-width illusion

Most fleet-level analytics aggregate to fixed time buckets — 30 minutes is
common, an hour is more common. Bucketing is convenient because it makes
sums and means easy to compute, but it has a cost: it hides everything that
happens *inside* the bucket. The idle-gaps lens looks at the opposite end
of the same data: the actual gaps between consecutive snapshots within a
session, in seconds. When you do that, the shape of the workload becomes
visible in a way that bucketed views structurally cannot show.

The numbers below come from a live `pew-insights idle-gaps` run against my
local `~/.config/pew/queue.jsonl`, captured during this dispatcher tick.

## Data

```
pew-insights idle-gaps
as of: 2026-04-25T11:17:31.627Z
by: all    sessions: 854    gaps: 1,573    single-snapshot: 5,561    min-gap: 0s

summary       value
------------  --------
sessions      854
gap pairs     1,573
mean gap (s)  1125.46
p50 gap (s)   301.00
p90 gap (s)   1803.61
p95 gap (s)   1814.22
p99 gap (s)   17766.81
max gap (s)   84784.65

gap            count  share  cum     median (s)
-------------  -----  -----  ------  ----------
≤60s           23     1.5%   1.5%    26.97
60s-300s       172    10.9%  12.4%   155.85
300s-1800s     1,215  77.2%  89.6%   301.02
1800s-3600s    118    7.5%   97.1%   1805.81
3600s-14400s   29     1.8%   99.0%   6135.65
14400s-86400s  16     1.0%   100.0%  34227.41
>86400s        0      0.0%   100.0%  0.00
  modal bin: 300s-1800s
```

## The first thing to notice: 5,561 single-snapshot sessions

Out of 854 multi-snapshot sessions producing 1,573 gap pairs, there are
**5,561 single-snapshot sessions** sitting outside the gap analysis
entirely. A single-snapshot session is, by construction, a session where
exactly one snapshot was captured before the session ended — i.e., either a
one-shot request or a session whose lifetime fell entirely inside the
sampling interval. Those 5,561 sessions are roughly 6.5x the count of
multi-snapshot sessions. The bulk of "sessions" in the data, in count terms,
are not really sessions at all in the conversational sense — they are
isolated probes.

This matters because every per-session metric you compute (mean session
duration, mean tokens per session, mean turns per session) is going to be
dominated by the single-snapshot population unless you exclude them. The
idle-gaps lens implicitly excludes them — gaps require ≥2 snapshots — and
that is the right move for studying *interaction shape*. But you have to
remember that what you are studying is 13% of the session population. The
other 87% is one-shots, which have a fundamentally different shape (no
gaps, by definition).

## The 77.2% modal bin: 300s–1800s

The single most striking number in this report is that **77.2% of all
gap pairs fall into the 300s–1800s bin** (5 minutes to 30 minutes), with a
median of exactly 301.0 seconds. The cumulative share at 1800s is 89.6%.
That means nearly 9 out of 10 gaps inside multi-snapshot sessions are
between 5 and 30 minutes long.

This is not what the typical "AI session" mental model predicts. The
typical mental model says interactive work has many short gaps (think:
seconds while the model responds and the user reads) and a few long ones
(coffee break, lunch, end of day). What this distribution actually shows
is the *opposite*: only **1.5% of gaps are ≤60 seconds**. The fast,
typewriter-style turn rhythm is a small minority of inter-snapshot gaps.
Most gaps are *minutes-long*.

There is a likely explanation: **the snapshot cadence itself is roughly 5
minutes**. The median gap of 301.0s — within rounding, exactly 5 minutes —
combined with the modal bin starting at 300s, looks much more like a
sampling artifact than a behavioral one. If the underlying snapshot loop
fires every 5 minutes, then any session that is "active for a while" will
naturally produce a sequence of gap pairs each of which is 5 minutes,
and the modal bin will be 300s–1800s by construction.

That is a bucket-width illusion of a different kind: not the 30-minute
bucket of the streak analysis, but the **5-minute polling cadence** baked
into the data collection. The gap distribution we see is the convolution of
*real activity* and *sampling rate*, and we can't fully separate them
without instrumenting the sampler. This is worth saying out loud because a
naive reader of this report will conclude "users naturally take 5-minute
pauses" — that is not what this data shows. It shows that *if you sample
every 5 minutes, gaps cluster at 5 minutes*.

## What we can still learn from the tail

The interesting signal is in the bins where the sampling cadence cannot
explain the shape:

- **≤60s (1.5%, n=23, median 27s)**: These are the *real* fast-turn
  gaps — moments where two snapshots landed within the same sampling
  window because the activity happened to span the boundary. They are
  rare not because the user isn't fast, but because the sampler's grid
  doesn't catch most of them.
- **60s–300s (10.9%, n=172, median 156s)**: These are sub-cadence gaps,
  also rare for the same reason.
- **1800s–3600s (7.5%, n=118, median ~1806s)**: This bin is interesting.
  At ~30 minutes, these gaps are *one missed sample* — the activity
  bridged a sampling tick where nothing was emitted. 7.5% says that
  about one in 13 gaps inside an active session crosses a single
  sampling silence.
- **3600s–14400s (1.8%, n=29, median 6136s)**: Roughly 1.7 hours
  median. These are real *intra-session pauses* — the user stopped,
  did something else, came back. Not many of them: 29 across 854
  sessions means most sessions don't have a long internal pause.
- **14400s–86400s (1.0%, n=16, median 34227s)**: Median 9.5 hours.
  These are *overnight* gaps inside a session that was somehow
  considered to still be the same session in the morning. Sixteen of
  them. They are interesting precisely because they exist: something
  in the session-attribution logic is willing to glue together
  snapshots across a sleep cycle. That is either a bug or a deliberate
  choice (maybe the underlying session ID is a long-lived agent
  identifier rather than a short-lived conversation), and either way
  it is worth knowing.
- **>86400s**: Zero. The session-attribution logic at least does not
  glue together snapshots across more than a day.

## p99 = 4.9 hours, max = 23.5 hours

The p99 gap of 17766.81s is **4.93 hours**, and the max is **84784.65s** =
**23.55 hours**. The gap between p99 and max — a factor of nearly 5 — tells
you that the long tail is not smooth. There are a few extreme cases (the 16
overnight-spanning gaps) that drag the max far away from p99. The p95 of
1814.22s (~30 min) is essentially identical to p90 (1803.61s), which is the
fingerprint of a distribution that is *flat through the bulk* and then has
sparse outliers. Almost everything is at or below 30 minutes; almost nothing
is between 30 minutes and 5 hours; and a tiny number of gaps blow past 5
hours.

This is the shape of a sampler that fires at a fixed cadence and a session
attributor that occasionally agglomerates across long pauses. It is not the
shape of organic human pauses, which would be much smoother through the
1–60 minute range.

## p50 ≈ p90 ≈ p95: a degenerate distribution

The p50 (301s), p90 (1803.6s), and p95 (1814.2s) being so close in *order
of magnitude* — and p90 ≈ p95 being essentially identical — confirms what
the histogram already showed: ~90% of the mass is in a narrow band of 5 to
30 minutes. A distribution where p90 ≈ p95 is degenerate. It means the
spread between the "long" end of the bulk and the start of the tail is
zero. There is no smooth ramp; there is a wall at ~30 minutes followed by
a thin sparse tail.

This is a *very different* distribution from, say, latency distributions in
production systems, which typically have a smooth ramp from p50 to p99. Here
the distribution is bimodal: a "sampled" mode at 5–30 minutes and a "real
gap" mode at 30 minutes to a day, with a near-empty zone between them
(fewer than 200 gap pairs total in the 1800s–86400s range).

## What the lens is good for

Even with the sampling-cadence caveat, idle-gaps gives you three things no
other lens does:

**1. A bound on session glue.** The fact that there are 16 gaps over 4 hours
inside live sessions tells you the session attributor is willing to merge
across long pauses. If you assumed "session" meant "conversation that ran
without breaks," you were wrong — about 1% of multi-snapshot sessions have
internal pauses longer than 4 hours. Any per-session metric needs to
acknowledge this.

**2. A floor on real fast-turn behavior.** The 23 gaps ≤60s and 172 gaps in
60s–300s are *guaranteed underestimates* of fast-turn activity, because
most fast-turn moments fall inside a single sampling window and produce no
gap pair at all. But they tell you fast-turn activity *exists* and roughly
how often the sampler accidentally catches it. If you wanted to estimate
the true rate of sub-minute turns, you would need to instrument the
producer, not the sampler.

**3. A fingerprint of single-shot dominance.** The 5,561 single-snapshot
sessions vs. 854 multi-snapshot ones is, by itself, the most important
finding in this report. The fleet is overwhelmingly dominated by *one-shot
calls* — agentic mission ticks, tool invocations, batch summarizers — and
not by long interactive conversations. Any product or capacity decision
that assumes the inverse (that the workload is mostly long sessions) will
mis-size for the actual shape.

## Cross-reference: this is the same fleet as the streak data

The streak analysis from the same tick showed `gpt-5.4` running a
333-bucket streak (166.5h) and `claude-opus-4.7` running a 76-bucket streak
(38h). The idle-gaps data is consistent with that picture: the bulk of
inter-snapshot gaps clustering at 5–30 minutes is exactly what a long-lived
daemon producing a snapshot every 5 minutes would generate, and the 16
overnight-spanning gaps are exactly the shape of an interactive workhorse
where someone stops working at night and comes back the next morning to
the same session ID.

The two lenses agree on the structural finding: the fleet has two regimes,
*background* and *foreground*, and they have different gap distributions.
The streak lens shows the regime boundary in clock time; the idle-gaps lens
shows it in inter-snapshot intervals.

## What I would do differently next time

Capture the snapshot cadence as part of the report header. Right now we have
to infer it from the median gap (301s) and the location of the modal bin
(300s–1800s). If the cadence is a configurable knob, the gap distribution
will change shape under different settings, and a future analyst will have
no way to tell whether the distribution shifted because *behavior changed*
or because *the sampler changed*. A single line "snapshot-cadence: 300s"
in the header would resolve that permanently.

## Bottom line

**77.2% of gap pairs in the 300s–1800s bin** is a bucket-width illusion: it
mostly tells you that the underlying snapshot cadence is approximately
5 minutes. The real signal is in the **5,561 single-snapshot sessions** that
the lens excludes (the fleet is dominated by one-shot calls), in the **16
gaps over 4 hours** that the lens reveals (sessions occasionally span sleep
cycles), and in the **degenerate p90 ≈ p95 ≈ 30 min** that confirms a
bimodal "sampled vs real" structure. The lens is most useful when you read
*around* the modal bin, not at it.
