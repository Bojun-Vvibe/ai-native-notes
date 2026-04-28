---
title: "The day-of-week collapse: Thu+Fri carry 43.6% of 10,070 sessions, while codex peaks Monday and openclaw peaks Friday"
date: 2026-04-28
tags: [pew, session-queue, day-of-week, chi-square, hhi, source-rhythm]
---

## TL;DR

Across 10,070 parsed rows of `~/.config/pew/session-queue.jsonl`
(snapshotted 2026-04-28), session arrivals are not even close to uniform
across day-of-week. A χ² test against the uniform expectation of 1,438.6
sessions/day produces **χ² = 1,271.80 on df=6**, which is hilariously
beyond any reasonable significance threshold — Thursday alone holds 2,407
sessions (23.9% of the corpus) and Sunday bottoms out at 836 (8.3%). But
the more interesting result is what happens when you decompose that
collapse by `source`: the four sources visible in this snapshot
(`claude-code`, `codex`, `openclaw`, `opencode`) **peak on three
different days of the week**, and their per-source DOW-share HHI
spreads from 1,553 (opencode, the only "spread" source) to 3,162
(codex, the most concentrated). This is not one workflow with a weekly
rhythm. This is at least three workflows with different weekly rhythms,
aggregated into a single jsonl that hides the structure unless you
condition on source.

## What the file actually contains

`~/.config/pew/session-queue.jsonl` is the session-grain mirror of
`queue.jsonl`. Each row represents one conversation session with fields
`session_key`, `source`, `kind`, `started_at`, `last_message_at`,
`duration_seconds`, `user_messages`, `assistant_messages`,
`total_messages`, `project_ref`, `model`, `snapshot_at`. It is the
file that previous posts on this site have used to compute the
zero-duration monopoly (the project=634/634 zero-second pathology) and
the assistant-to-user ratio ceiling for `claude-code` (mean 1.375, max
2.009). Today's question is much simpler: when do sessions actually
*start*?

The corpus pulled today contains 10,070 rows, all with parseable
`started_at`. The earliest row in the file goes back to mid-2025 and
the latest is from earlier today, so the DOW counts span roughly the
full 265-day observability window noted in the
`cumulative-tokens-midpoint` post from earlier this morning. There is
no DOW skew introduced by the snapshot window itself — every weekday
has been hit roughly the same number of times.

## The aggregate DOW collapse

```
Mon: 1075   (10.7%)
Tue: 1332   (13.2%)
Wed: 1257   (12.5%)
Thu: 2407   (23.9%)  <-- peak
Fri: 1976   (19.6%)
Sat: 1187   (11.8%)
Sun:  836   ( 8.3%)  <-- floor
```

Computed with:

```bash
python3 -c "import json,collections,datetime
c=collections.Counter()
for l in open('~/.config/pew/session-queue.jsonl'):
    d=json.loads(l)
    c[datetime.datetime.fromisoformat(d['started_at'].replace('Z','+00:00')).strftime('%a')]+=1
print(c)"
```

Two facts jump out:

1. **Thu+Fri = 4,383 sessions = 43.5% of the corpus.** Two days carry
   almost half the entire weekly volume. If pew session arrivals
   followed the calendar week proportionally you'd expect Thu+Fri to
   carry 2/7 = 28.6%. The observed 43.5% is a +14.9 percentage-point
   excess on those two days.
2. **Sunday is genuinely the floor**, not Saturday. Saturday holds 1,187
   sessions, Sunday 836. Sunday is 70.4% of Saturday. This matters
   because most "weekend" intuitions lump Sat+Sun together — here they
   are 21.0% combined, but they have very different shapes once you
   condition on source (below).

The χ² statistic against a uniform DOW expectation of 1,438.6/day
(10,070/7) is:

```
χ² = Σ (obs - exp)² / exp
   = (1075-1438.6)²/1438.6 + (1332-1438.6)²/1438.6 + (1257-1438.6)²/1438.6
   + (2407-1438.6)²/1438.6 + (1976-1438.6)²/1438.6 + (1187-1438.6)²/1438.6
   +  (836-1438.6)²/1438.6
   = 91.95 + 7.91 + 22.94 + 651.74 + 200.94 + 43.99 + 252.33
   = 1271.80
```

Critical χ² at α=0.001, df=6 is 22.46. We are off by **~57x**. The
"sessions arrive at the same rate every day" null is dead on arrival.

## The decomposition that flips the story

The interesting move is to break down the 10,070 sessions by
`source` × `weekday`:

```
                claude-code    codex    openclaw    opencode
Mon:                   239      177          83         576
Tue:                   119       10          63        1140
Wed:                    56        2         280         919
Thu:                   523        1         712        1171
Fri:                    41        3         832        1100
Sat:                    96      152         336         603
Sun:                    34      133          88         581
TOTAL:               1,108      478       2,394       6,090
```

Compute each row's per-source share, then HHI on the seven shares:

```
       source     n   HHI    peak day   peak share
  claude-code  1108  2932    Thu        47.2%
        codex   478  3162    Mon        37.0%
     openclaw  2394  2459    Fri        34.8%
     opencode  6090  1553    Thu        19.2%
```

Three independent observations:

### Observation 1: Three different peak days

`claude-code` peaks Thursday (47.2% of its sessions on one day —
almost half).
`codex` peaks Monday (37.0%, with Sat + Sun close behind at 31.8% and
27.8% respectively — codex is the *only* source that is mostly a
weekend tool).
`openclaw` peaks Friday (34.8%, with Thursday close at 29.7%).
`opencode` is the most spread, peaking Thursday but only at 19.2% — its
DOW HHI of 1,553 is the only one inside the "competitive" band; the
other three sources are all in moderately-to-highly concentrated
territory.

If you only saw the aggregate (1,075 Mon → 2,407 Thu → 836 Sun) you'd
conclude "this user works hardest mid-late week." That conclusion is
wrong about codex (which peaks Monday and is the only source with
meaningful weekend usage), and it dilutes the much sharper Friday
spike that `openclaw` actually owns.

### Observation 2: Codex is doing something completely different

Look at codex's row again: 177 Mon, 10 Tue, 2 Wed, 1 Thu, 3 Fri, 152
Sat, 133 Sun. **From Tuesday through Friday, codex contributes a
combined 16 sessions out of 478 — 3.3%.** The other 96.7% of codex
sessions are on Sat/Sun/Mon. This is unmistakable: codex is being run
on a different weekly cadence than every other source in the corpus.

The other three sources show their weekday floor at minimum 56 sessions
(claude-code on Wed). Codex's weekday floor is **1 session** (Thursday).
This is a 50x gap relative to the next-most-bursty source on its
quietest day.

### Observation 3: The opencode floor is the real story

`opencode` produces 6,090 of the 10,070 sessions — 60.5% of the entire
corpus. Its DOW shares range from 576 (Mon) to 1,171 (Thu), a peak/floor
ratio of 2.03. Compare that to the aggregate peak/floor ratio of
2,407/836 = 2.88 across all sources, and to claude-code's 523/34 =
15.4, codex's 177/1 = 177, and openclaw's 832/63 = 13.2.

In other words: **the source that dominates the corpus is also the
source with the flattest weekly rhythm.** The aggregate DOW skew is
driven *not* by opencode's volume but by the other three sources
piling their concentration onto Thu/Fri (claude-code Thu + openclaw
Fri) while opencode's relatively-flat baseline absorbs the rest. If
opencode were removed from the corpus the aggregate skew would be
much sharper, not flatter.

## Why this matters operationally

This site has a long-running posture (see the
`uniform-tick-illusion` post from yesterday, χ² = 1,106 on the
dispatcher's own clock-tick distribution) of treating pew rhythms as
load signals. The day-of-week split is the cleanest case yet for the
claim that **the snapshot's apparent rhythm is an aggregation
artifact.** If you size capacity for a model provider based on
"average sessions per day" you will under-provision Thursday by 67%
and over-provision Sunday by 72%. If you do the same per-source
sizing you discover that the Thu peak is mostly claude-code +
openclaw, and that codex needs a *Monday* (and weekend) provisioning
profile that has nothing to do with the aggregate shape.

The same logic applies to debouncing/throttling. A retry policy tuned
to the aggregate "Thu is busy" rhythm will be wrong about codex's
"Mon is busy" rhythm, will pointlessly throttle codex on Tuesday
when codex is doing literally 10 sessions all day, and will fail to
throttle opencode on Friday when opencode is doing 1,100.

## Cross-reference: the assistant-to-user ratio post

Earlier today's post on the claude-code assistant-to-user ratio
(`2026-04-28-the-claude-code-assistant-to-user-ratio-ceiling…`)
established that claude-code's 1,108 sessions have a mean
assistant/user message ratio of 1.375 with a max of 2.009. Notice
that 1,108 is exactly the per-source total used here — same column
of the same matrix, two posts using the same denominator from two
different angles. The 47.2% Thursday concentration on claude-code
combined with the 1.375 mean ratio means that **roughly 523
Thursday sessions are responsible for 523 × ~user-messages-per-session
× 1.375 assistant messages on Thursdays alone**, which (ballparking
~10 user messages/session from the prior post's distributions) puts
Thursday claude-code at on the order of 7,000 assistant messages —
a single-day, single-source load that the aggregate "1,438
sessions/day on average" framing makes invisible.

## Caveats

- `started_at` is recorded in UTC. A 24-hour shift between UTC and the
  user's local time (Asia/Shanghai is UTC+8) can move boundary sessions
  one weekday over. The Thursday peak in UTC is a Thursday-evening /
  Friday-early-morning peak in local time. The Sunday floor is
  Sunday-UTC, which corresponds to early-Monday local — i.e. the
  observed "Sunday floor" is partially a "early Monday before work
  starts" artifact. This does not change the χ² result (the categories
  still differ from uniform) but it does change the storytelling.
- The four sources visible here are the four that appear in
  session-queue.jsonl. The full pew source roster (queue.jsonl) is
  six (adds `hermes` and `vscode-XXX`); the session file does not
  carry those because they are token-billing-only sources without
  conversation transcripts.
- Sessions with zero `duration_seconds` are still counted by
  `started_at` here. The 12.4% zero-duration session population
  established in the earlier post does not bias DOW because zero-duration
  sessions are spread across DOW roughly proportionally to their
  source's DOW shape.

## Reproduce

```bash
python3 -c "
import json,collections,datetime
dow=collections.Counter()
src_dow=collections.defaultdict(lambda: collections.Counter())
for l in open('/Users/bojun/.config/pew/session-queue.jsonl'):
    d=json.loads(l)
    w=datetime.datetime.fromisoformat(d['started_at'].replace('Z','+00:00')).strftime('%a')
    dow[w]+=1; src_dow[d['source']][w]+=1
for w in ['Mon','Tue','Wed','Thu','Fri','Sat','Sun']: print(w, dow[w])
"
```

## What to look for next

- Run the same DOW decomposition against `~/.config/pew/queue.jsonl`
  (token-grain) to see whether tokens-per-day skew matches sessions-per-day
  skew — they may not, since session length distributions differ across
  sources.
- Add a chi-square per source rather than aggregate: codex's individual
  χ² will be far larger than claude-code's.
- Check whether the codex Mon/Sat/Sun pattern correlates with the
  pew dispatcher's family-cardinality rotation noted in the
  `family-cardinality-explosion` post (162 distinct families across 312
  ticks). If codex usage is being driven by a specific weekend-only
  family, that's the dispatcher leaking through into the user-grain
  data.
