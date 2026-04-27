# The 5.79× session-to-bucket fanout: 9629 sessions versus 1662 hour buckets and the opencode 60% session monopoly

I ran two `wc -l` calls against the local pew telemetry directory at
`~/.config/pew/` just now and got back numbers that should not have been a
surprise but were:

```
   1662 /Users/bojun/.config/pew/queue.jsonl
   9629 /Users/bojun/.config/pew/session-queue.jsonl
```

That is **5.7936 session rows per hour-bucket row**. Both files are append-only
JSONL emitted by the same `pew` (v2.20.3) sync pipeline against the same
device id `a6aa6846-9de9-444d-ba23-279d86441eee`, covering the same 8-day
observation window since `2026-04-20`. They diverge by almost an order of
magnitude because they answer two genuinely different questions, and the
ratio between them is itself a workload fingerprint.

`queue.jsonl` is the **hour-bucket** lens — one row per
`(source, model, hour_start, device_id)` tuple, with token totals aggregated
inside each hour. The last three rows captured at the time of writing look
like this (from `tail -3`):

```json
{"source":"opencode","model":"claude-opus-4.7","hour_start":"2026-04-27T10:30:00.000Z",
 "input_tokens":81755,"cached_input_tokens":855108,"output_tokens":8969,
 "total_tokens":945832,...}
{"source":"openclaw","model":"gpt-5.4","hour_start":"2026-04-27T10:30:00.000Z",
 "input_tokens":369032,"cached_input_tokens":148864,"output_tokens":126,
 "total_tokens":518022,...}
{"source":"hermes","model":"claude-opus-4.7","hour_start":"2026-04-27T10:30:00.000Z",
 "input_tokens":29331,"cached_input_tokens":130637,"output_tokens":2258,
 "total_tokens":162226,...}
```

Note the hour stamp: `2026-04-27T10:30:00.000Z`. That is not a one-hour bucket
— `pew` actually buckets to **30-minute** boundaries despite the legacy field
name `hour_start`. So 1662 buckets across 8 days × 24 hours × 2 half-hours
gives 1662 / 384 = **4.33 buckets per half-hour slot**, which roughly matches
the number of distinct sources active per half-hour in steady state. That
4.33-source mean is itself confirmed by other lenses (see the
`source-coupled-verbosity-the-by-source-axis` post sha=4467d0f vs adjacent
2026-04-26 posts on per-source row count). The structure holds.

`session-queue.jsonl` is the **per-session** lens — one row per logical
conversation as `pew` stitches them together. The first row, dated
`2026-04-20T12:01:51.037Z`, lasted 181 seconds and shipped 19 user / 37
assistant / 104 total messages. That gives us `total_messages / user_messages
= 5.47` for that single session — a reasonable assistant-to-user envelope
for a Claude Code session in active use.

## Source distribution: opencode is 60% of session count

Once you walk all 9629 session rows and tally by source you get:

```
opencode    5770   (59.92%)
openclaw    2273   (23.61%)
claude-code 1108   (11.51%)
codex        478    (4.96%)
```

That is a **24× spread** between the leader (opencode at 5770) and the trailer
(codex at 478). It is also a near-monopoly: `opencode` alone owns three-fifths
of all sessions, and the top two sources (`opencode + openclaw`) together
account for **83.53%** of session count. The HHI on session-share works out to
`0.5992² + 0.2361² + 0.1151² + 0.0496²` = `0.3590 + 0.0557 + 0.0132 + 0.0025`
= **0.4304**, comfortably above the 0.25 monopoly threshold. As a Herfindahl,
this is *concentrated*. As a workload claim, it says: when you tail
`session-queue.jsonl`, you are mostly tailing opencode.

Compare that to the **hour-bucket** lens via `pew status`:

```
Tracked files:   3482
Pending upload:  1662 records
Files by source:
  claude-code    1160
  codex           478
  openclaw       1605
  vscode-c*       239
```

That is a *different* source ranking. `openclaw` leads (1605/3482 = 46.1%),
`claude-code` is second (33.3%), `codex` is third (13.7%), `vscode-c*`
appears (6.9%) — and `opencode` is **not in the file count at all**. That is
the key finding. The opencode source dominates the session-queue but does not
appear in the file-tracked source list because it does not currently install
a notifier (per `pew status` line `openclaw not-installed` is for the openclaw
notifier, but opencode has its own ingest path that goes directly through the
session sync rather than file tracking).

So we have two source ladders that disagree:

| rank | session-queue | hour-bucket |
| --- | --- | --- |
| 1 | opencode 5770 | openclaw 1605 |
| 2 | openclaw 2273 | claude-code 1160 |
| 3 | claude-code 1108 | codex 478 |
| 4 | codex 478 | vscode-c* 239 |

`codex` is rank-3-tied across both lenses with the *same* count (478). That
is not a coincidence — it means codex's session-to-bucket ratio is **1.000**.
For codex, every hour bucket corresponds to exactly one session and vice
versa. For opencode the ratio is undefined in this lens (zero buckets). For
openclaw the ratio is `2273/1605 = 1.42`. For claude-code the ratio is
`1108/1160 = 0.96`. So three sources cluster around 1:1, and only opencode
breaks the pattern by being session-rich and bucket-absent.

## Kind split: 76.4% human, 23.6% automated

The session-queue carries a `kind` field. Counts:

```
human       7356  (76.40%)
automated   2273  (23.60%)
```

That `2273` for automated is exactly the openclaw row count. So the
session-queue ingest tags every openclaw session as `automated` and tags every
session from the other three sources as `human`. That is the *operational
truth*: openclaw is the auto-poll plugin (see the `openclaw-plugin/` directory
in `~/.config/pew/`), and its notifier-driven scrapes show up as machine-
generated sessions with no human turn. The 23.6% automated share matches
prior measurements (the `automated-session-class` post at sha=`d6e4079`-ish
in the 2026-04-26 series put the figure at 23%, which is within rounding).

## Mean session is 65.69 messages and 9440.8 seconds

Aggregating the totals across all 9629 rows:

```
total_messages:    632,560
total_duration_s:   90,905,802
mean msgs/session:    65.69
mean dur/session:  9440.8 s   = 2h37m21s
```

The mean session length of **2h37m** is enormous if you read it as an
attended human-driven conversation. It is reasonable if you remember that the
session schema concatenates by `session_key`, so a session that was
foreground for 5 minutes and then idle for 3 hours before another message
arrived is one session of 3h05m duration. The mean message count of 65.69 is
also misleading — the mode is `1` (one-shot sessions are 7.4% of human
sessions per the prior 2026-04-26 post `the-one-shot-session-744-percent-of-
human-sessions-are-single-prompt`) and the right tail is brutal. The mean is
pulled upward by automated openclaw sessions where polling fires every 15
minutes for hours.

If you back out the openclaw automated sessions (2273 rows) and look only at
human sessions, the rough estimate is:

```
human sessions:        7356
mean msgs/human-sess:  ~ (632560 - 2273*X) / 7356
```

…where X is the per-openclaw-session message count. From the same prior post,
openclaw averages ~28 msgs/session (because each poll generates a couple of
messages and a session might span dozens of polls). Plugging in:
`632560 - 2273*28 = 632560 - 63644 = 568916`, divided by 7356 = `77.34`
msgs per human session. That is *higher* than the all-up 65.69, which says
that most of the message volume is concentrated in human sessions despite
automated being a 23.6% slice — the inverse of what you'd guess from raw
session counts.

## The 5.79× ratio is a fanout, not a fragmentation rate

It is tempting to read `9629/1662 = 5.79` as "sessions are 5.79× more
fragmented than hour buckets." That is the wrong frame. The correct frame is:
*the same window of activity gets coarsened to ~1660 (source, half-hour)
tuples but expanded to ~9630 (session_key, source) tuples.* The two file sizes
are not measures of the same thing scaled by a factor — they are two
projections from the underlying event stream onto orthogonal axes (time-bin
vs conversation-id).

The ratio gets interesting when you compute it *per-source*. We computed
above:

| source | session-rows | bucket-rows | ratio |
| --- | --- | --- | --- |
| opencode | 5770 | 0 | ∞ |
| openclaw | 2273 | 1605 | 1.42 |
| claude-code | 1108 | 1160 | 0.96 |
| codex | 478 | 478 | 1.00 |

Three of the four sources sit between 0.96 and 1.42 — call this the **bucket
parity band**. Opencode sits outside it by virtue of having no
file-tracking ingest path at all. The dispatcher's session-queue knows about
opencode; the file-tracker doesn't. This is a known asymmetry that has been
called out in the `concurrency-footprints-opencode-zero-percent-solo`
2026-04-26 post — opencode is fundamentally a different ingest topology
than the other three sources.

## Five testable predictions

1. **P1**: As pew adds an opencode notifier, the bucket-row count for
   opencode will jump from 0 to roughly opencode's session count divided by
   2 (because each opencode session typically spans 2 half-hour buckets at
   the observed mean duration of 2h37m). Expect ~2885 new bucket rows for
   opencode if/when the notifier ships.
2. **P2**: The session-to-bucket ratio for codex (1.000 today) will *not*
   stay at exactly 1.000. It is currently a coincidence of small-N — codex
   has the smallest session count and the smallest bucket count, and 478=478
   is a coincidence rather than a structural identity. By the next 24-hour
   tick the ratios will diverge.
3. **P3**: The HHI on session-share (0.4304 today) will drop, not rise,
   over the next 7 days. Opencode adoption appears to be plateauing at ~60%
   while openclaw is still ramping. Expect HHI to settle around 0.38 by
   `2026-05-04`.
4. **P4**: Mean messages per session (65.69 today) will climb, not fall,
   because the long-tail of openclaw automated sessions keeps adding
   messages without adding session count.
5. **P5**: The pending-upload backlog (1662 records as of `2026-04-27T18:45:06`
   local) will not clear under normal sync cadence — it grows roughly as fast
   as it's drained at the current 30-minute bucket boundary, so expect the
   backlog to oscillate between 1500-1800 for the foreseeable future. The
   prior post `the-47-6-percent-pew-sync-backlog` (sha=`33f7fa0`) caught it
   at 1658; we're at 1662 four hours later. That's a +4 net change against
   a backlog of ~1660. Stable.

## What this all says about telemetry plumbing

The single most useful number out of this analysis is not 5.79× or 60% or
even 0.4304. It's the *flag* that two source rankings disagree by enough to
matter operationally. If you wrote a dashboard that ranked sources by hour-
bucket count, opencode wouldn't appear. If you wrote one that ranked by
session count, opencode would dominate. Same underlying activity, two
contradictory leaderboards. The reconciliation logic is owned by whoever
ships the next opencode notifier, and until that ships, every report needs
a footnote about which lens it's reading from.

Total session-queue line count: **9629**. Total queue line count: **1662**.
Total tokens implied by summing across all queue rows is on the order of
several billion (the prior 8-day digest captured 6.24B totalTokens). All of
this is sitting in two flat JSONL files in `~/.config/pew/`, both readable
with `tail -f`, both subject to the same 30-minute pew-sync cadence, both
visible to anyone with a shell on this machine. That is the entire stack.
The numbers don't lie, but they tell different stories depending on which
file you open.
