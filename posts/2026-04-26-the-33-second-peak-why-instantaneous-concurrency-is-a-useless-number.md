# The 33-second peak: why instantaneous concurrency is a useless number

Every system that tracks concurrency reports a peak number, and almost every
team treats that number as load-bearing. "We hit 21 concurrent sessions on
April 22nd" sounds like the kind of fact you would put on a capacity-planning
slide. It is not. The number is real, but the operational interpretation
that gets attached to it — that the system needs to be sized for 21 — is
almost always wrong, because the duration of that peak is missing.

The `pew-insights concurrency` report from 2026-04-25 makes this concrete.
Across 4,834 sessions over 5.7 days, peak concurrency was 21. The peak was
first seen at 2026-04-22T10:33:39.381Z. And the total time the system spent
*at* that peak was 33 seconds. Not 33 minutes. Thirty-three seconds, across
a 5.7-day window. That is 0.0067% of wall time. Average concurrency over
the same window was 8.26. The 95th percentile was 14.

If you sized the system for 21, you would be sizing for an event that
occupies six and a half thousandths of a percent of the operational
calendar. If you sized it for 8.26, you would underprovision for the
~5% of time you sit above 14. The right number is not the peak and not
the mean. The right number depends on what failure mode you are
defending against — and the report makes the failure-mode question
unavoidable in a way that a single peak number does not.

## What the actual numbers say

```
peak concurrency           21
peak first seen            2026-04-22T10:33:39.381Z
peak total time            33s
avg concurrency            8.26
p95 concurrency            14
coverage (>=1 open)        100.0%
```

Coverage at 100% means there was never a moment in the 5.7-day window
when zero sessions were open. The system is always doing something. The
mean is 8.26, the p95 is 14, and the peak is 21 — but the peak is held
for 33 seconds total. Not in one stretch; this is *cumulative* time at
the maximum level, which means there were several brief spikes that all
hit 21 and immediately dropped back. The histogram of time-at-each-level
is not in the summary, but the relationship between mean (8.26) and p95
(14) tells you the bulk of the distribution is in single-digits with a
fat tail.

The interesting structural detail is what was open at the peak. Of the
21 sessions, the report names 10:

```
openclaw:0c02…  automated  started 2026-04-21T20:01:58Z  ended 2026-04-22T19:48:55Z
openclaw:1c78…  automated  started 2026-04-21T20:28:20Z  ended 2026-04-22T19:58:28Z
openclaw:281c…  automated  started 2026-04-22T06:54:05Z  ended 2026-04-23T19:08:15Z
openclaw:3dd5…  automated  started 2026-04-22T06:54:05Z  ended 2026-04-23T02:40:09Z
openclaw:99ad…  automated  started 2026-04-17T02:54:38Z  ended 2026-04-23T07:16:31Z
openclaw:e4e9…  automated  started 2026-04-22T10:30:35Z  ended 2026-04-22T10:35:10Z
opencode:ses_…    human    started 2026-04-22T10:33:39Z  ended 2026-04-22T10:35:40Z
opencode:ses_…    human    started 2026-04-22T10:33:39Z  ended 2026-04-22T10:35:40Z
opencode:ses_…    human    started 2026-04-22T10:33:39Z  ended 2026-04-22T10:35:40Z
opencode:ses_…    human    started 2026-04-22T10:33:38Z  ended 2026-04-22T10:35:39Z
```

The peak is not 21 *concurrent users hammering the system*. It is 6
long-lived automated sessions plus 4 (named) human opencode sessions
that all opened within the same wall-clock second (10:33:38–10:33:39)
and all closed within the same wall-clock second (~10:35:39–10:35:40).
Those four sessions held the system for almost exactly 121 seconds. The
peak event has a clear shape: a fan-out of human-driven opencode
sessions opening simultaneously into a steady-state population of
automated openclaw work, holding for two minutes, and then collapsing
together.

The 33 seconds at peak are the moments inside that two-minute window
when *all* of the simultaneous opens overlapped with *all* of the
existing automated sessions. The fan-out is not gradual; the collapse
is not gradual. This is what concurrency peaks look like when the
underlying workload is bursty: a single decision creates four sessions
in the same second, and the peak is the brief intersection of that
decision with the ambient long-running work.

## What this rules out for capacity planning

The 21 number cannot be used as a capacity floor for several reasons,
each of which is independently sufficient:

**The peak is duration-zero.** Sizing for 21 means provisioning
resources that, on this workload, would sit unused for 99.9933% of the
time. If the cost of an extra concurrency slot is non-trivial — and it
almost always is, because each slot needs memory, file handles, network
capacity, possibly a worker process — then the peak number is the
wrong target.

**The peak is dominated by automated sessions.** Six of the ten named
overlapping sessions are openclaw automated work that has been running
for hours or days. Those sessions are not sensitive to per-request
latency in the way human-driven sessions are. They can absorb queueing.
A capacity model that treats all 21 sessions as equally latency-sensitive
is over-provisioning for a workload class that does not need it.

**The peak is fan-out, not load.** The four human opencode sessions
that opened in the same second are almost certainly a single user
opening four parallel agents — a deliberate, controlled fan-out. The
correct capacity model for fan-out is "support the fan-out factor" not
"support the peak concurrency." If the fan-out factor is 4, then
ensuring 4 parallel slots are always available is the right design;
ensuring 21 slots are always available is the wrong design.

**The peak time is non-recurring within the window.** The peak was
"first seen" at 10:33:39 on April 22. The phrase "first seen" implies it
recurred, but the cumulative time at peak is only 33 seconds — meaning
the recurrences are also brief. There is no sustained operational
regime in which the system runs at 21. There are spikes that touch 21
and immediately drop.

## What p95 actually tells you

The p95 of 14 is the more honest planning number. It is the level
above which the system spends 5% of time. On a 5.7-day window, that is
about 6.8 hours total. That is a real operational constraint: for nearly
seven hours of the week, the system is at or above 14 concurrent
sessions. If a slot above 14 is not available, queueing happens, and
queueing during sustained periods is what users perceive as "the system
is slow today."

But even p95 understates the relevant question. The relevant question
is *how long the system stays above each level*, not what fraction of
time it does. A workload that spends 5% of time at level 14 in one
6.8-hour stretch is operationally different from one that spends the
same 5% in 100 short bursts. The former produces a sustained
degradation that looks like an outage. The latter produces a
shimmering of small slowdowns that no one notices.

The concurrency report does not directly expose the run-length
distribution at each level, but the 33-second total at the peak is
strong evidence that the peaks are short-burst-shaped rather than
sustained. If 33 seconds is the cumulative time at level 21, then the
cumulative time at levels 18, 19, 20 is probably in the same minute
range, which means the entire upper tail is short-burst behavior.
That changes the engineering response. Short-burst peaks are absorbed
by queueing; sustained peaks require more concurrency.

## The mean (8.26) and the integral

Mean concurrency of 8.26 over a 100%-coverage window has a precise
operational interpretation: it is the integral of session-time divided
by wall-time, which is exactly the *number of slot-hours consumed per
hour*. This is the quantity that maps directly to cost on a
provisioned-throughput model. If each concurrency slot costs $X per
hour to keep available, the system is consuming 8.26 × $X × 24 ×
5.7 = ~$1130 worth of slot-hours per dollar-of-X over the window.

This is the only concurrency number that maps cleanly to money. The
peak does not. The p95 does not. The mean does, because the mean *is*
the integral. If your provisioning model is "always have N slots
available and charge for them," the answer to "what should N be" is
some quantile of the concurrency distribution that you choose based on
your tolerance for queueing — and the *cost* of that choice is N × the
slot rate, regardless of how often you use them.

## When the peak number is the right number

There is one case where the peak number is correct: when the cost of
ever exceeding a hard concurrency limit is catastrophically higher
than the cost of provisioning above it. If hitting concurrency 22
crashes the process and loses all 22 sessions' state, then yes, you
must size for 21 — and ideally for 22 or more. This is the
"file-descriptor limit" failure mode. It is real, and for systems with
hard limits, the peak is the planning number.

Most modern systems are not in this regime. They have soft queueing
above some target, gracefully degraded throughput, and recoverable
session state. For those systems, the peak is decoration; the
distribution shape is the design input.

## The "sessions skipped" detail

The header line is worth reading carefully:

```
sessions: 4,834 considered, 643 skipped
```

643 sessions out of 5,477 total were skipped from the concurrency
calculation. That is 11.7%. The skip reasons are not in the summary,
but in tools of this shape they are typically: missing end timestamp,
zero-duration sessions, or sessions outside the window after a left-edge
clip. A 11.7% skip rate is high enough that the reported peak of 21
is an undercount — there were sessions open during the peak window
that got excluded because their metadata was incomplete.

This is the kind of detail that destroys naive capacity planning. The
"true" peak is not 21; it is "21 plus some unknown fraction of 643
skipped sessions that happened to overlap the window." If your
planning depends on the precise peak value, you have to chase down
why those 643 were skipped. If your planning depends on the
distribution shape — mean, p95, run-length — the skip rate is just
noise, and you can plan against the visible distribution with
reasonable confidence.

## What 33 seconds is really telling you

The single most important thing in the report is the 33-second peak
duration. It is telling you:

1. **The system is bursty, not sustained.** Whatever produces
   concurrent demand is event-driven, not load-driven. Capacity planning
   for bursty systems uses queueing models, not steady-state models.

2. **The peak is a fan-out signature.** Four sessions opening in the
   same wall-clock second is not load. It is a deliberate, controlled
   parallelization decision by a single source. The right capacity
   response is to support that fan-out factor *for that source*, not
   to scale the whole system.

3. **The bulk of the work is long-running automated sessions.** Those
   are the 6 openclaw entries with multi-hour durations. They contribute
   to mean concurrency continuously and to peak concurrency only
   incidentally. Capacity for them is a steady-state planning problem
   with a known integral.

4. **The dramatic-sounding peak number is a distraction.** A peak of
   21 held for 33 seconds is operationally indistinguishable from a
   peak of 14 held for 33 seconds, except in pathological hard-limit
   regimes. The p95 of 14 is the real planning constraint.

The lesson generalizes: any concurrency number reported without a
duration is a partial fact. A peak with no duration is a worst-case
that may be vanishingly rare. A mean with no run-length distribution
is a quantity that says nothing about queueing behavior. A p95 without
the burst shape is a target that may demand much more provisioning
than necessary, or much less.

If you have a concurrency report that gives you only the peak, you
should treat it the way you treat a thermometer that only reports the
maximum reading of the day: technically true, operationally useless.
The 33-second annotation in this report is what makes the peak number
a *story* rather than a *number*. Without it, you would size for 21.
With it, you can correctly conclude that the system is fine at the
provisioning level it already has, and the right work to do is on the
fan-out source — not on the total concurrency budget.
