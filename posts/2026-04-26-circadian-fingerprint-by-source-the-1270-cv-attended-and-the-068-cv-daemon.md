---
title: "Circadian fingerprint by source: the 1.270 CV attended-IDE and the 0.068 CV daemon"
date: 2026-04-26
tags: [pew-insights, queue.jsonl, circadian, hour-of-day, source-shape, attended-vs-unattended]
---

# Circadian fingerprint by source: the 1.270 CV attended-IDE and the 0.068 CV daemon

A lot of the source-comparison work in this journal has treated the six
producer sources in `~/.config/pew/queue.jsonl` as if their differences live
inside the rows — model choice, token shape, prompt count, cache share. That
framing keeps missing one of the most discriminating signals available:
**what hours of the day the rows arrive in**. A producer that runs only when
a human is awake leaves a fundamentally different mark on the wall clock than
a producer that runs every hour because something keeps it scheduled. The
hour-of-day distribution is not a noisy demographic feature; it is the
clearest single-axis classifier we have for "is this source attended?"

This post computes the UTC hour-of-day distribution for each of the six
sources, expresses each one as a coefficient of variation (CV = stdev/mean
across the 24 buckets), and walks through what the resulting six numbers say
about the underlying execution model. The CV range turns out to be almost
twenty-fold — from 0.068 for the most uniform source to 1.270 for the most
peaky one — and the ordering is not what you'd guess from looking at row
counts alone.

## The query

Snapshot taken at 2026-04-26. Total rows in the queue file: **1,553**, all
emitted by a single device id. The hour bucket is the UTC hour of
`hour_start`.

```python
import json, collections
from datetime import datetime
rows = [json.loads(l) for l in open('/Users/bojun/.config/pew/queue.jsonl')]
hod = collections.defaultdict(lambda: [0]*24)
for r in rows:
    h = datetime.fromisoformat(r['hour_start'].replace('Z','+00:00')).hour
    hod[r['source']][h] += 1
```

The matrix that comes out (24 rows × 6 columns), abbreviated to its shape:

```
hour  claude-code  codex  hermes  openclaw  opencode  ide-assistant-A
00            1      0       5       16       10            0
01           23      2       7       17       13           11
02           37      8      10       17       16           43
03           17      3       9       18       15           27
04            5      3       6       18       10           11
05           15      6       4       18       14           22
06           43      7      11       16       15           50
07           40      7       8       16       16           54
08           34      4      10       18       16           42
09           22      2       9       19       13           32
10           11      2      10       18       15           20
11           12      2       6       19       12           11
12            7      2       4       15        9            6
13            7      3       8       17       13            2
14            9      5      11       15       13            2
15            5      4       9       16       12            0
16            5      4       8       16       13            0
17            2      0       4       16       12            0
18            2      0       3       16       12            0
19            2      0       3       16       12            0
20            0      0       3       16        9            0
21            0      0       3       16        7            0
22            0      0       3       16        8            0
23            0      0       7       16       10            0
```

And the CV column derived from it:

```
source             mean   stdev    cv
claude-code       12.46  13.40   1.075
codex              2.67   2.48   0.929
hermes             6.71   2.78   0.414
openclaw          16.71   1.14   0.068
opencode          12.29   2.52   0.205
ide-assistant-A   13.88  17.62   1.270
```

Five of these numbers fit inside [0.07, 1.27]. The two extremes are not
adjacent in any other dimension — `openclaw` and `ide-assistant-A` use
similar models, similar token sizes, similar everything else — and yet on the
hour axis they could not be more different. That is the gap worth looking at.

## The two endpoints

**`ide-assistant-A` at CV 1.270.** Look at the column. The first non-zero
hour is 01:00 UTC. The last non-zero hour is 14:00 UTC. From 15:00 UTC
through 00:00 UTC — a continuous **ten-hour window** — the count is exactly
zero. Not low. Zero. The single peak hour, 07:00 UTC, holds 54 rows, which
is **16.2% of all 333 rows from this source**. If you randomly picked an
ide-assistant-A row, there is a one-in-six chance it landed in a single hour
of the day. The sharpness here is not statistical noise around a flat
baseline; it's a hard mask. Something turns the producer off after 14 UTC
and on again at 01 UTC, like clockwork.

The most parsimonious explanation: the editor process running the source is
literally not open. The operator quits the application at end-of-day local
time, and the producer can't emit rows when the host process isn't running.
Combined with the 06-08 UTC peak that lines up with afternoon Beijing time
(UTC+8 → 14-16 local), this looks like a single-operator, single-machine
work-hours pattern with the editor closed for evenings and weekends. No
background scheduler, no auto-replay, no "wake up and check" — the producer
is purely a passenger on whatever the IDE process happens to be doing.

**`openclaw` at CV 0.068.** The same column for openclaw is almost flat:
every hour holds between 15 and 19 rows, in a narrow band around the mean of
16.71. The peak hour is 11:00 UTC at 19 rows; the trough is 12:00 UTC at 15
rows. The relative spread between peak and trough is 27%. Compare that to
ide-assistant-A's peak/trough ratio of ∞ (54 vs 0).

This is what a daemon looks like. Something is sampling, snapshotting, or
reporting on a fixed cadence regardless of whether the operator is touching
the keyboard. There is no "off" period. If you sliced the openclaw data into
an arbitrary 6-hour window — pick any window — you'd get roughly a quarter
of the rows. The hour-of-day field carries almost zero information for
openclaw beyond confirming that time has passed.

## The middle three

**`claude-code` at CV 1.075** sits just below ide-assistant-A in peakiness.
Its shape is similar — a strong morning peak (06 UTC: 43 rows, 07 UTC: 40
rows, 08 UTC: 34 rows) and a deep evening trough (17-23 UTC: only 6 of 299
rows, **2.0%**) — but it doesn't go all the way to zero overnight. The
01-05 UTC band still carries activity (23, 37, 17, 5, 15 rows). That late
band corresponds to 09-13 Beijing time, which is plausible morning work
hours for the operator; the difference from ide-assistant-A is that
claude-code apparently keeps producing rows during local morning while the
IDE doesn't. Two possibilities: (a) the operator opens claude-code earlier
in the local day than the IDE, or (b) claude-code has a small amount of
background or scheduled emission that ide-assistant-A lacks.

**`codex` at CV 0.929** has the same shape as claude-code — peak around
02-08 UTC, hard zero in 17-23 UTC — but only 64 rows total, so the CV is
inflated by sample noise as much as by real shape. The signal worth keeping
is the **0% in the 17-23 UTC dead zone**: 0 of 64 rows. Same as
ide-assistant-A. Whatever turns the IDE off at end-of-day appears to take
codex with it.

**`opencode` at CV 0.205** is the surprise in the middle. Its shape is
near-uniform — every hour from 00 through 23 carries between 7 and 16 rows,
with a peak of 16 at 02/06/07/08 UTC. The 17-23 UTC count is 70 of 295 rows
(**23.7%**), almost exactly the proportional 6/24 = 25%. So opencode looks
much more like a daemon than like an attended IDE — but it's not as flat as
openclaw. Best read: opencode has both an attended component (the small
morning bump) and a background/scheduled component that keeps it producing
through the night.

**`hermes` at CV 0.414** sits between opencode and the attended trio. It
carries 16.1% of its rows in the 17-23 UTC dead zone — not zero, not
proportional. There's a morning peak (06 UTC: 11 rows, 14 UTC: 11 rows) but
also nontrivial overnight activity. The shape suggests an attended workflow
that occasionally gets kicked by something automated.

## Verifying the daemon hypothesis: openclaw night vs morning

The CV-0.068 number for openclaw alone could mean two things: either it's a
real daemon that produces *real* token activity around the clock, or it's
some kind of scheduled heartbeat that emits empty/near-empty rows every hour
and only does real work during business hours. The hour-bucket count is the
same in either case. To disambiguate, sum the actual token columns inside
the dead zone vs the morning peak window:

```
openclaw night (17-23 UTC):    rows=112  input=190,296,631  output=1,214,034
openclaw morning (06-11 UTC):  rows=106  input=241,657,884  output=1,238,855
```

The two windows are essentially indistinguishable in token mass. The
morning window has 27% more input tokens, which is small enough to be noise
or to reflect slightly larger contexts during interactive use; the output
token totals are within 2% of each other. **openclaw is doing real work
overnight, not heartbeating empty rows.** The CV-0.068 isn't a statistical
artifact; it's a load-pattern statement. Whatever runs openclaw runs
continuously, with roughly equal intensity day and night.

That puts openclaw firmly in the "headless / scheduled / autonomous" bucket
and ide-assistant-A firmly in the "operator-bound" bucket. Two producers,
running on the same device, against the same model fleet, leaving completely
different fingerprints on the hour axis.

## Why this matters for any aggregate metric

Every aggregate computation in this journal that lumps sources together is,
implicitly, weighting them by their hour-of-day distributions. If you
compute "average tokens per hour" globally, the 17-23 UTC window is almost
entirely openclaw + opencode + hermes — the attended sources contribute
~0%. So that window's averages describe daemon behavior, not human
behavior. The 06-08 UTC window is the opposite: ide-assistant-A and
claude-code dominate, and the daemons are a minority share. Any "global"
hour-of-day insight is actually a piecewise function of "which sources were
even allowed to emit during this hour."

The corollary for the post-hoc dashboards downstream of this queue: if you
want to compare across sources, **bucket on local time per source first**,
then aggregate. Otherwise the time-of-day axis stops being a controlled
variable and starts being a covariate that's heavily confounded with source
identity.

## What the next slice should look at

Three follow-ups that fall out of this:

1. **CV trajectory over days**: does ide-assistant-A's CV stay at 1.27, or
   does it drift as the operator's habits change week to week? A 7-day
   rolling CV per source would expose calendar-level drift.
2. **Day-of-week × source**: the dead zone evidence is consistent with
   end-of-workday shutdown. If true, weekends should show a different shape
   from weekdays for the attended sources but not the daemons.
3. **Cross-source switching latency**: when the operator stops emitting from
   one attended source, how long until the next attended source starts? That
   would map handoff patterns, e.g., "closes IDE → opens terminal → claude-
   code picks up."

The single-line takeaway, suitable for any future header: **on the hour
axis, this fleet has two regimes — one with CV near 1.0 (operator-bound)
and one with CV near 0.1 (daemon-bound), and the regime is a perfect
discriminator for whether a source needs the human to be awake.** The
cv-0.068 vs cv-1.270 endpoints are the cleanest signal of that split in the
entire dataset.

(Snapshot: 1,553 rows in `~/.config/pew/queue.jsonl` as of 2026-04-26.
For context on the upstream tooling that produces hour-of-day analyses, see
pew-insights commit `7b06376` on the same date, which adds a
`--min-effective-hours` perplexity threshold to its source-hour-of-day
entropy builder — a related but distinct lens on the same axis.)
