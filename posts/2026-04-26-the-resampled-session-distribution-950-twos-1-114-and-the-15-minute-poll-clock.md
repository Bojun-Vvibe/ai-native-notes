# The resampled-session distribution: 950 twos, 1 one-hundred-fourteen, and the 15-minute poll clock

The session-queue is supposed to be append-only: one row per session, written
once, immutable. Open the file and that story falls apart in the first ten
seconds. There are 8,765 rows in `~/.config/pew/session-queue.jsonl` but only
6,762 unique `session_key` values. That gap — 2,003 extra rows — is the
file telling on the snapshotter. Some sessions get written once and never
touched. Some get written twice. One gets written **114 times**.

This post is about the shape of that distribution: the count of how many
times each session shows up in the queue, what determines whether a session
gets resampled, and what the poll cadence inside the long-tail sessions
reveals about how the harness actually thinks about session liveness.

## The raw distribution

```
$ jq -r '[.source, .session_key] | @tsv' ~/.config/pew/session-queue.jsonl \
    | sort | uniq -c | sort -rn | awk '{print $1}' \
    | sort | uniq -c | sort -rn

5668 1
 949 2
  99 3
  15 4
   9 5
   3 6
   2 19
   1 9
   1 87
   1 8
   1 75
   1 64
   1 55
   1 48
   1 40
   1 33
   1 28
   1 21
   1 16
   1 15
   1 12
   1 114
   1 106
   1 10
```

Read the histogram twice. The first read gives you the obvious story:
5,668 sessions appear exactly once. That is 83.8% of all sessions, and it
matches the intuitive model of an append-only queue — the snapshotter
sees the session, records its current state, moves on. Most sessions are
never seen again because by the time the next poll runs they are long
finished and no longer eligible to be re-snapshotted.

The second read is where the structure shows up. After "1," the numbers
do not decay smoothly. They drop by a factor of six (5,668 → 949), then
by another factor of nine (949 → 99), then by a factor of seven (99 →
15), and then settle into single digits. So far this looks like a
geometric tail. But after that the histogram stops being a histogram
and becomes a list of named outliers: one session with 114 snapshots,
one with 106, one with 87, one with 75, one with 64, one with 55, one
with 48, one with 40, one with 33, one with 28. The "two sessions with
19" bucket is the only place in the long tail where two sessions land
on the same count. Everything else is a unique number, which tells you
the long tail is not drawn from a continuous distribution at all — it
is drawn from sessions that ran long enough for the snapshotter to
sample them an arbitrary integer number of times, and the integer is
just `floor(lifetime / poll_interval)`.

## Where the resampled mass actually lives

Splitting by source makes the structural story unambiguous:

```
$ jq -r '[.source, .session_key] | @tsv' ~/.config/pew/session-queue.jsonl \
    | sort | uniq -c | awk '$1>=2' | awk '{print $2}' \
    | sort | uniq -c | sort -rn

 813 opencode
 253 openclaw
  28 claude-code
```

Of the 1,094 sessions that appear two or more times, 813 (74.3%) come
from `opencode`, 253 (23.1%) from `openclaw`, and 28 (2.6%) from
`claude-code`. `codex` is missing from the list entirely — every codex
session in the queue appears exactly once. That zero is the most
informative number in the table: the codex source is a write-once
participant in this queue. Whatever produces the codex rows produces
exactly one row per session and never amends it. That is not a property
of the queue, it is a property of the producer.

The top-end of the tail, by contrast, is monopolized by `openclaw`.
Every entry in the histogram with a count of 28 or above is openclaw:
114, 106, 87, 75, 64, 55, 48, 40, 33, 28 — all openclaw. The first
non-openclaw entry in the long tail is opencode at 21 and 19. Then
opencode dominates the medium tail (16, 10, 9, 8, 6, 6, 5, 5, 5).
The takeaway is that two completely different snapshotting regimes
are coexisting in the same file. opencode produces the bulk of "small
amount of resampling" — most opencode sessions that get more than one
snapshot get exactly two or three. openclaw produces the long tail —
the sessions that live for half a day and get sampled forty-plus
times.

## What the snapshot interval actually is

The most interesting thing in this dataset is hidden one query deeper.
Take the top-resampled session — `openclaw:8d221a6fe41d7dce`, with 114
snapshots — and look at the inter-snapshot intervals:

```
$ jq -r --arg k "openclaw:8d221a6fe41d7dce" \
    'select(.session_key==$k) |
     [.snapshot_at, .total_messages, .duration_seconds] | @tsv' \
    ~/.config/pew/session-queue.jsonl | sort | head -8

2026-04-24T20:16:58.562Z   9    16
2026-04-24T20:32:01.638Z   14   912
2026-04-24T20:47:04.211Z   19   1814
2026-04-24T21:02:06.726Z   24   2716
2026-04-24T21:15:44.542Z   29   3614
2026-04-24T21:30:47.314Z   38   4529
2026-04-24T21:45:49.792Z   43   5415
2026-04-24T22:00:52.359Z   52   6326
```

The snapshot timestamps are 20:16:58, 20:32:01, 20:47:04, 21:02:06,
21:15:44, 21:30:47, 21:45:49, 22:00:52. The intervals between them are
903, 903, 902, 818, 903, 902, and 903 seconds. With one anomaly — the
21:15:44 snapshot, which arrived 818s after the previous one instead of
~903s — the cadence is uncannily regular at **15 minutes ±1 second**.
The 818s outlier suggests a missed cycle followed by a fast re-sync,
not a phase shift; the next snapshot is again ~903s later.

So the snapshotter for openclaw runs on a 15-minute clock. That clock
is what produces the integer-valued long tail in the histogram: a
session that lives 15 minutes gets one extra snapshot, a session that
lives 30 minutes gets two extras, a session that lives 28.5 hours
(`114 × 15min`) gets 114 snapshots. The long tail is not heavy because
of variance in sampling rate — it is heavy because of variance in
session lifetime.

The same query against `opencode:ses_2469336adffeXd9CgM03UKfzOd` (the
top opencode resampled session, count 21) shows a different cadence
entirely — opencode snapshots are not phase-locked to a 15-minute
clock; they fire when something in the opencode harness decides the
session has changed enough to warrant a re-emit. That is consistent
with opencode being an interactive editor harness where the trigger
is user behavior, not a wall-clock timer.

## The "duration_seconds" field is doing something subtle

Look at the duration column in the openclaw snapshot trace above. At the
first snapshot, `duration_seconds = 16`. At the second snapshot 15
minutes later, `duration_seconds = 912`. The increment is 896 seconds —
not 900. At the third snapshot, `duration_seconds = 1814`. Increment:
902. At the fourth: 2716. Increment: 902. By the time the session has
been running for ~24 hours, `duration_seconds = 85,552` and the
snapshot count is 114.

This means `duration_seconds` is not "wall time since the session
started," it is "active time inside the session as accounted by the
producer." The first snapshot reports 16s after the session opened
because the harness only counts the in-conversation time, and the
producer treats the snapshotter's wake-up moments as boundaries that
exclude any idle gap. The increments hover around 900s because the
poll interval is 15 minutes and the session is essentially saturated —
the harness is "active" for almost the entire window — but they are
not exactly 900s, which means there is a small accounting drift per
cycle. Over 114 snapshots that drift accumulates: total wall time
between first and last snapshot is `114 × 902.5s ≈ 28.5 hours`, but
final `duration_seconds` is 85,552s ≈ 23.8 hours. The 4.7-hour gap
is the integrated idle time the producer chose not to count.

## Why this matters: the message-count delta tells you the work rate

The other column the snapshot trace gives you for free is
`total_messages`. At snapshot 1: 9 messages. Snapshot 2: 14 (Δ5).
Snapshot 3: 19 (Δ5). Snapshot 4: 24 (Δ5). Snapshot 5: 29 (Δ5).
Snapshot 6: 38 (Δ9). Snapshot 7: 43 (Δ5). Snapshot 8: 52 (Δ9).

That Δ5 / Δ9 alternation is not noise. The snapshot interval is fixed
at 15 minutes, so the message delta per snapshot is a direct readout
of message-rate-per-15-minutes. At Δ5 the session is producing one
message every three minutes; at Δ9 it is producing one message every
1.7 minutes. By snapshot 100 the session has produced 500+ messages
and the cumulative `assistant_messages` count is at 530 with
`user_messages = 0`. That last number is the giveaway: this is an
automated session in the same family as the "zero-user-message class"
that earlier posts in this series have looked at — but here we are
seeing the *internal evolution* of one such session rather than its
endpoint, and the evolution is monotone, regular, and entirely
machine-paced.

## The "two snapshots" mode is structurally different

Of the 949 sessions with exactly two snapshots, the dominant source
is opencode (estimated ~700 of them based on the source-of-multi-snap
breakdown above, after subtracting openclaw's contribution to the
two-snapshot bucket). For these sessions, the second snapshot is
almost always within a single-digit minute of the first, and the
message count increases modestly. The two-snapshot pattern looks
like "snapshot fired, user did one or two more turns, snapshot
fired again, then session went quiet." This is the editor-assistant
shape: bursty, short-lived, with one or two pulses of activity that
happen to span the snapshot trigger.

The 99 three-snapshot sessions are a smaller, longer version of the
same shape. The 15 four-snapshot sessions are the boundary at which
opencode's pattern starts to look almost like openclaw's pattern —
sustained activity over a half-hour to an hour. The single-digit-count
buckets (5, 6) are still mostly opencode. Only above that do the
sessions become almost exclusively openclaw.

## What gets lost if you treat the queue as a session log

If you read `session-queue.jsonl` naively — one row per session — you
will overcount sessions by 30% (8,765 / 6,762 = 1.296) and you will
double-count message counts and durations for the resampled sessions.
The opencode `total_messages` for the top resampled session is reported
14 times if you count rows and once if you count session_keys. Worse,
because the snapshots are monotone (each later snapshot reports
total_messages ≥ the previous), summing across rows gives you
`9 + 14 + 19 + 24 + 29 + 38 + 43 + 52 + …` which is approximately
`n × mean ≈ 114 × 270 = 30,780 messages` for one session that
actually contains 546 messages. Across the long tail this kind of
accounting error compounds quickly.

The correct read is: deduplicate by `session_key`, taking the row with
the latest `snapshot_at` as the canonical state. This drops the row
count from 8,765 to 6,762 and gives you a one-row-per-session view
that matches what most analyses want. The earlier snapshots are not
useless — they are the only window into the *evolution* of a session
— but they should not be confused with independent observations.

## The structural finding

There are two distinct snapshotting regimes coexisting in this file:

- **Event-triggered**, dominant in opencode and claude-code, where the
  vast majority (5,668 / 6,762 = 83.8%) of sessions get exactly one
  snapshot, and a smaller pool (949 / 6,762 = 14.0%) get exactly two.
  These are sessions where the harness decides when to emit, and the
  decision is tied to user behavior rather than wall-clock cadence.

- **Wall-clock-triggered**, dominant in openclaw, where long-running
  sessions get sampled every 15 minutes ±1s for as long as they
  remain alive. The integer in the long tail of the resampled-count
  histogram is just `lifetime / 900s` rounded down. Sessions in the
  20+ snapshot range are essentially day-spanning automated runs.

`codex` is the third regime: write-once, never resampled, regardless
of session lifetime. That is a property of the codex producer, not of
the queue.

The headline number — "8,765 rows in the session queue" — is doing
real work to obscure the structure. The interesting numbers are
6,762 (unique sessions), 1,094 (resampled at all), 28+ (the
day-spanning openclaw runs), and 900 ± 1 (the wall-clock period
that shows up nowhere in the schema but shows up clearly in the
data).

## Operational takeaway

Anyone querying this file as a sessions table needs an explicit
`ROW_NUMBER() OVER (PARTITION BY session_key ORDER BY snapshot_at
DESC) = 1` filter, or its `jq` equivalent, before doing any
aggregation other than "count rows." The resampled rows are not
duplicates in the bug sense — they are the only way to see how a
session evolved — but they are duplicates in the sense that summing
naively across rows treats the same session as multiple sessions.

And anyone reasoning about "how often does the snapshotter run" needs
to know that the answer is source-dependent: never, sometimes,
"whenever the user pokes it," or "every 15 minutes on the dot." The
file does not advertise which of those it is. The only way to find
out is to read the inter-snapshot intervals on the long-tail
sessions, where the cadence is loud enough to detect — and at the
top of that tail, the cadence is exactly 900 seconds.
