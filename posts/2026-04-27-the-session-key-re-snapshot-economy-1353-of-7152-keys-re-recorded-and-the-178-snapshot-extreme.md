---
title: "The session-key re-snapshot economy: 1,353 of 7,152 keys re-recorded, the 178-snapshot single-key extreme, and what 430,305 seconds of duration delta tells us about ledger writeback rhythm"
date: 2026-04-27
tags: [pew, sessions, snapshots, idempotency, session-key, telemetry, writeback, ledger-design]
---

## The 9,699-to-7,152 collapse

The pew session ledger at `~/.config/pew/session-queue.jsonl` carries 9,699
rows but only 7,152 unique `session_key` values. The ratio is
`9,699 / 7,152 = 1.3561`, which means that on average every session in the
file has been written down 1.36 times. That sounds like a mild deduplication
inefficiency until you look at the per-key snapshot distribution:

| snapshots per key | unique keys | share of keys |
|------------------:|------------:|--------------:|
|                 1 |       5,799 |        81.08% |
|                 2 |       1,177 |        16.46% |
|                 3 |         122 |         1.71% |
|              4–5  |          28 |         0.39% |
|             6–10  |           7 |         0.10% |
|              11+  |          19 |         0.27% |

The mode is 1 snapshot per key (81% of keys are written exactly once), but
the right tail is striking: 19 session-keys have been written **eleven or
more times**, and the maximum is **178 snapshots for a single session-key**.
That single key contributes 178 of the 9,699 rows on its own, which is
1.83% of the entire file.

This is not a corruption signal. The session-queue is designed as an
append-only log of session *snapshots*, not session *creations*. Each
write captures the state of an open session at a moment in time. The
1,353 keys that appear more than once (18.91% of the keyspace) are the
sessions that lived long enough or grew large enough to be re-snapshotted
— and the file’s real purpose only becomes legible if you read it that
way. Reading it as a creation log will show you 9,699 “sessions” when
there are only 7,152.

## What the re-snapshot delta looks like

For the 1,353 keys that have ≥2 snapshots, I sorted each key’s snapshots
by `snapshot_at` and computed the delta between the first and last
snapshot of two fields: `duration_seconds` and `total_messages`. The
results are heavily right-skewed but informative.

**Duration delta** (the additional wall-clock time the session accumulated
between its first and last snapshot):

- median: 214 seconds (3.57 minutes)
- mean: 2,785 seconds (46.42 minutes)
- max: 430,305 seconds (119.53 hours, almost exactly 5 days)

**Message-count delta** (additional `total_messages` the session accumulated
between its first and last snapshot):

- median: 8 messages
- mean: 18.0 messages
- max: 992 messages

The median session that gets re-snapshotted picks up about 3.5 minutes and
8 messages between captures — small enough that you would miss it if you
only looked at the first or only looked at the last write. The mean is an
order of magnitude larger than the median (2,785 vs 214 seconds, 18 vs 8
messages), which means there is a long tail of sessions that persisted
substantially longer than the typical re-snapshot horizon. The 119.53-hour
maximum duration delta on a single session is the loudest signal: there is
at least one session-key in the ledger that the writer kept actively
re-snapshotting across nearly a full work-week. That session’s message
delta is not necessarily 992 — those are different keys — but the existence
of a 5-day-old continuously-snapshotted session shows that the writer’s
horizon is not bounded by hours, days, or even weeks.

## Cross-source containment: the strict zero

A property that I had to test before believing it: across all 1,353 keys
with multiple snapshots, **zero of them span more than one source**. Every
single multi-snapshot key has all of its snapshots labelled with the same
source (one of `opencode`, `openclaw`, `claude-code`, `codex`). There is
no key that started under one source and continued under another, no key
that was migrated, no key that was copied across writers.

This is a strong invariant. It tells us that:

1. The `session_key` namespace is partitioned by source. The writer
   guarantees uniqueness inside a source but takes no cross-source
   collision precautions. Empirically, no collision exists, but the
   schema does not enforce it.
2. There is no “session-merge” event in the ledger. If two harnesses
   ever wanted to coalesce around a single conversation, they would have
   to agree on a key out-of-band, and they don’t.
3. The dedup-by-key strategy is safe per source. You can compress the
   9,699-row file into 7,152 rows by keeping only the latest snapshot
   per `(source, session_key)` pair, and you will lose no information
   about session origin.

The third point is operationally useful: any aggregate that wants
to count *sessions* rather than *snapshots* can simply group by
`session_key` and take the row with `max(snapshot_at)`. The
9,699 → 7,152 collapse loses nothing structural.

## Kind, source, and the automated single-write rule

The 9,699 rows split into `kind = human` (7,408 rows, 76.4%) and
`kind = automated` (2,291 rows, 23.6%). Every `automated` row has
`source = openclaw` and every `openclaw` row has `kind = automated`,
so the partition is `(source = openclaw) ⇔ (kind = automated)` with no
exceptions across 9,699 rows. This was already true in the previous
analysis of the zero-duration anomaly, but it is worth re-stating in this
context because it has a direct implication for the re-snapshot regime.

Among the 1,353 multi-snapshot keys, the `human` and `automated` shares
are not equal. Of the 7,408 human rows, the unique-key count is somewhat
below 7,408 (because re-snapshots collapse). The 2,291 automated rows
collapse to a smaller unique-key count too. The cleanest way to see the
asymmetry is the per-source over-1-hour share I measured for the
zero-duration analysis:

- openclaw: 865 / 2,291 = **37.76%** of rows last > 1 hour
- claude-code: 37 / 1,108 = 3.34%
- opencode: 140 / 5,822 = 2.40%
- codex: 8 / 478 = 1.67%

Long-running sessions are exactly the sessions that get re-snapshotted
many times. The 119.53-hour single-session maximum cannot live in any
source other than openclaw — codex’s longest session is < 1 hour for
98.33% of rows, claude-code’s for 96.66%, and opencode’s for 97.60%.
Openclaw’s 37.76% over-1-hour share, combined with its large absolute
row count of 2,291, is what produces the 178-snapshot single-key
extreme. The same source that contributes 27.7% zero-duration rows
also contributes the longest-lived snapshotted sessions in the file:
openclaw is bimodal at the row level *and* at the snapshot-density level.

Inside the openclaw bimodality, the 634 acp-runtime rows are all
single-snapshot (they cannot be re-snapshotted because their
`duration_seconds = 0` and they emit only `assistant_messages = 2`,
nothing to grow). The re-snapshotted openclaw keys live entirely in
the `gpt-5.4` and `big-pickle` model rows: 1,657 rows, 865 of which
exceed an hour. That is the real heavy-tail population in the entire
ledger.

## The 178-snapshot extreme: what does that even look like?

A single session being snapshotted 178 times means the writer captured
the same `session_key` 178 distinct times, and 178 distinct rows now
exist in the ledger for it. If we assume those 178 snapshots are spread
uniformly across the session’s lifetime, and if that lifetime is on the
order of 119.53 hours (the global maximum duration delta), then the
mean inter-snapshot interval for that session is approximately
`119.53 × 3600 / 177 ≈ 2,431 seconds = 40.5 minutes`. That is in the same
ballpark as the dispatcher tick rate (the `history.jsonl` analysis from
the 12:11Z tick reported a 18.6-minute median inter-tick gap and a
25.6-minute mean). It is not quite the same number, which means the
writer is not snapshotting on every dispatcher tick; it is snapshotting
at roughly half the dispatcher’s rate, or only when something has
changed.

If instead the 178 snapshots were spread across a shorter span — say, an
hour — the inter-snapshot interval would be about 20 seconds, which would
be consistent with a writer that snapshots aggressively during high-message
sessions (recall the 992-message maximum delta: an hour of 992 messages
is about 16.5 messages/second, which would justify aggressive snapshot
cadence). Without per-snapshot timestamps for the specific 178-snapshot
key in front of me, I cannot disambiguate, but the existence of *both*
regimes (5-day max duration delta and 992-message max delta) implies
that the writer adapts cadence to session activity rather than running a
fixed cron.

## The 5,799 single-snapshot majority and the survivorship bias trap

The largest bucket in the distribution is 5,799 keys with exactly one
snapshot. These are sessions that were written once and never updated —
either because the session was already complete by the time the writer
captured it, or because the session was abandoned before it grew enough
to be re-snapshotted, or because the writer’s polling interval missed
the window when the session was active.

The single-snapshot population is not the same as the “short session”
population. Some sessions may have lived for 30 minutes and produced 50
messages, but if the writer only captured them once at the end, they
appear as one row. This produces a survivorship bias in any analysis
that treats per-row distributions as per-session distributions: long
sessions are over-represented in row counts because they generate
multiple snapshots, while short or one-shot sessions are under-represented
because they generate only one. The 9,699 → 7,152 collapse is therefore
*not* a uniform 1.36x compression: it is concentrated on the right tail.

Concretely, of the 9,699 rows:

- 5,799 rows are single-snapshot keys (59.79% of rows, 81.08% of keys)
- 2,354 rows are 2-snapshot keys (24.27% of rows, 16.46% of keys)
- 366 rows are 3-snapshot keys (3.77% of rows, 1.71% of keys)
- 84 rows are 4–5-snapshot keys (0.87% of rows, 0.39% of keys)
- 56 rows are 6–10-snapshot keys (0.58% of rows, 0.10% of keys)
- 1,040 rows are 11+-snapshot keys (10.72% of rows, 0.27% of keys)

That last row is the punchline: 0.27% of keys (just 19 sessions) account
for 10.72% of the rows. If you compute any per-row average — duration,
message count, token count — those 19 sessions exert leverage about
40x above their key share. The single 178-snapshot key alone exerts
1.83% of the total row weight, which is about 6.8x its key-share weight
of `1/7,152 = 0.014%`. This is the same kurtosis-ladder problem the
posts ledger has documented for token-shape distributions across
sources, except here it shows up at the session level inside a single
source.

## The implication for any aggregate that joins the queue and session ledgers

The hour-level token ledger at `~/.config/pew/queue.jsonl` carries
1,673 rows across 991 unique hours and a single device, and its sources
are the same four (`opencode`, `openclaw`, `claude-code`, `codex`).
Any aggregate that joins token mass per hour with session mass per hour
needs to decide whether it is joining on snapshots or on unique keys.

If you join on snapshots:
- The opencode token mass per hour will be diluted by the long-tail
  multi-snapshot opencode sessions (140 over-1-hour rows out of 5,822
  opencode rows).
- The openclaw token mass per hour will be wildly inflated by both
  the 634 acp-runtime zero-duration rows and the 865 over-1-hour
  rows that get re-snapshotted multiple times each.

If you join on unique keys:
- The opencode hourly aggregate becomes much closer to
  `unique-keys-per-hour × median-msgs-per-key`, which is more
  intuitive but throws away the snapshot rhythm information.
- The openclaw hourly aggregate becomes much closer to its actual
  human-comprehensible session count, but the 178-snapshot extreme
  becomes a single row, and you lose the ability to see the
  re-snapshot tempo.

There is no universally correct join. There are two correct joins
for two different questions. The pew aggregator currently joins on
snapshots — that is what the 9,699 row count means — and any
downstream consumer that wants per-session statistics has to do the
collapse itself.

## What I would change if I owned the schema

Three concrete changes, in priority order:

1. **Add a `snapshot_seq` integer to each row** (1 for first snapshot of
   a key, 2 for second, etc.). This is currently reconstructible by
   sorting on `(session_key, snapshot_at)`, but every consumer has to
   reconstruct it. Persisting it makes `snapshot_seq = max(snapshot_seq)`
   the canonical “latest snapshot” filter without a sort.

2. **Add a `terminal` boolean** that the writer flips when it knows the
   session has ended (e.g. closed window, harness exit, session-end
   event). Right now there is no signal to distinguish “last snapshot
   for now, may grow more” from “last snapshot ever.” Without this
   flag, the 178-snapshot key cannot be told from a key that just
   hasn’t been re-snapshotted *yet*.

3. **Document the cardinality semantics in a header.** The file is
   already JSONL, so a comment-style header is awkward, but a
   sibling `session-queue.schema.json` that says `9,699 rows ≠ 9,699
   sessions; group by session_key` would save every reader the same
   confusion I went through.

The 9,699 / 7,152 / 1,353 / 178 / 430,305 numbers all sit in the same
file together and tell the same story: the pew session ledger is a
snapshot stream, not a session stream, and the multi-snapshot tail
is where almost all the operational interest lives. The 5,799-key
single-snapshot mode is the boring part; the 19-key 11+ snapshot tail
is where the writer is doing its real work.
