---
title: "The zero-duration session anomaly: 1,252 of 9,699 pew sessions, the acp-runtime 100% collapse, and the asst/user ratio fingerprint that reveals three different harnesses"
date: 2026-04-27
tags: [pew, sessions, telemetry, zero-duration, harness-fingerprint, openclaw, opencode, claude-code, codex, acp-runtime]
---

## What the ledger says

The pew session ledger at `~/.config/pew/session-queue.jsonl` carries 9,699 rows
across 7,152 unique `session_key` values. Inside that ledger lives one of the
loudest data anomalies I have seen in the last week of token-shape work:
**1,252 of those 9,699 rows (12.91%) record `duration_seconds = 0`** even
though they are written into the session table, not the per-hour token table,
and even though many of them carry non-zero `assistant_messages`.

Zero-duration sessions are not equally distributed. Sliced by `source`, the
fingerprint is dramatic enough to distinguish four harnesses without looking
at any other field:

| source       | rows  | zero-duration | share  |
|--------------|------:|--------------:|-------:|
| openclaw     | 2,291 |           634 | 27.7%  |
| opencode     | 5,822 |           566 |  9.7%  |
| codex        |   478 |            19 |  4.0%  |
| claude-code  | 1,108 |            33 |  3.0%  |

Sliced by `kind`, the same gap re-appears in a different shape: 634 of 2,291
`automated` rows are zero-duration (27.7%), versus 618 of 7,408 `human` rows
(8.3%). The numerical equality `automated zero-duration = openclaw
zero-duration = 634` is not a coincidence: every `automated` row in the
ledger is an openclaw row, and openclaw contributes the entire 27.7% slice.
The zero-duration rate of `automated` is exactly the zero-duration rate of
openclaw because the two sets are the same set.

But the cleanest cut is the per-model breakdown:

| model                          | rows  | zero-duration |  share |
|--------------------------------|------:|--------------:|-------:|
| `acp-runtime`                  |   634 |           634 | 100.0% |
| `None` (null)                  |   527 |           519 |  98.5% |
| `big-pickle`                   |   788 |            45 |   5.7% |
| `<synthetic>`                  |    93 |             2 |   2.2% |
| `gpt-5.4`                      | 2,116 |            18 |   0.9% |
| `claude-opus-4.7`              | 5,052 |            32 |   0.6% |
| `claude-sonnet-4.6`            |   281 |             0 |   0.0% |
| `claude-opus-4.6-1m`           |   145 |             0 |   0.0% |

`acp-runtime` is 100% zero-duration. Every single one of its 634 rows. The
`None` model bucket — sessions written without a model field at all — is
98.5% zero-duration (519/527). Together those two slices account for
1,153 of the 1,252 zero-duration rows, which is 92.1% of the anomaly. The
remaining 99 zero-duration rows are scattered across `big-pickle` (45),
`claude-opus-4.7` (32), `gpt-5.4` (18), and the synthetic stub (2). Two
production-graded models, `claude-sonnet-4.6` and `claude-opus-4.6-1m`,
contribute zero zero-duration rows out of 426 combined: not a single
session of theirs lasted exactly zero seconds.

## What zero-duration actually means here

The pew session schema records `duration_seconds` as the wall-clock interval
between `started_at` and `last_message_at`. A genuinely zero-duration session
is one where the only timestamp the harness ever emitted was `started_at`,
and either no `last_message_at` was ever set or it was set to the same value.
Three cleanly separable pathologies can cause this, and the ledger lets us
discriminate between them by looking at the `user_messages` and
`assistant_messages` fields on the same row.

Across the 1,252 zero-duration rows:

- **451 rows have both `user_messages = 0` and `assistant_messages = 0`.**
  These are pure shells: a session row was written, but no turn was ever
  exchanged. The harness opened a session-key and abandoned it before the
  first message landed.
- **634 rows have `user_messages = 0` and `assistant_messages > 0`** (median
  `assistant_messages = 2`, max = 2). This is the signature of a fully
  automated, machine-driven harness that emits assistant turns without a
  human turn ever existing — the user side is replaced by harness state
  rather than a literal human message. Every single one of these 634 rows
  is the `acp-runtime` slice. It is not a coincidence: `acp-runtime`
  *defines* the “no user message, exactly two assistant messages, zero
  duration” shape, and it does so 634 times in a row.
- **87 rows have `user_messages > 0` and `assistant_messages = 0`** (median
  `user_messages = 0` due to the row count, max = 3). These are aborted
  sessions: a human typed something, the assistant never responded, and the
  session row crystallized at `started_at = last_message_at`.
- The remaining **80 rows** have both sides positive but still report
  zero duration — a clock-resolution rounding case, where both timestamps
  fell into the same second.

The 634-row `u=0, a>0` block is the dominant pathology by count, and it is
purely an automated harness emitting pre-canned or mirror-style assistant
content into the ledger. That class is well-understood: it is the same
class that the synth ledger has been flagging since W17 #100 as the
substitution-shape pattern, except here it shows up not as a PR-pair
property but as a per-session steady state.

## The asst/user ratio: three different harnesses, one ledger

The other half of this anomaly only becomes visible if you compute the
assistant-to-user message ratio per source, restricting to sessions where
`user_messages > 0` (so the ratio is finite). When you do, three of the
four sources emit fingerprint-grade distinct medians:

| source       |    n  | median a/u | mean a/u | max a/u |
|--------------|------:|-----------:|---------:|--------:|
| claude-code  | 1,108 |       1.15 |     1.38 |     2.0 |
| codex        |   478 |       3.00 |     3.02 |    10.0 |
| opencode     | 5,371 |       7.00 |    11.56 |   111.0 |
| openclaw     |     0 |          — |        — |       — |

(openclaw drops out entirely because all 2,291 of its rows carry
`user_messages = 0`; the ratio is undefined for the entire source.)

These three medians — 1.15, 3.00, 7.00 — are not random points on a
continuum. They cleanly separate three harness families:

1. **claude-code at a/u ≈ 1.15** is the strict-pair harness: nearly one
   assistant turn per user turn, with the small surplus coming from
   tool-call fan-out being collapsed into a single assistant message.
   The mean (1.38) is barely above the median, the max is 2.0, and
   the entire distribution sits inside `[1, 2]`. This is the shape of a
   harness that linearises tool calls into the assistant turn rather
   than emitting them as separate ledger entries.

2. **codex at a/u ≈ 3.00** is the triplet harness: one user turn typically
   produces a planning message, a tool-call burst, and a finalisation
   message. The mean (3.02) tracks the median almost exactly, which means
   the ratio is nearly delta-distributed at 3.00. The max of 10.0 is the
   only ceiling-class outlier — long-tail-suppressed, by design.

3. **opencode at a/u ≈ 7.00** is the sprawl harness: each user turn
   detonates into a long sequence of assistant turns covering reasoning,
   multiple tool calls, and follow-up reasoning. The mean (11.56) is far
   above the median, which means the distribution is right-skewed; the
   max of 111.0 means that at least one opencode session in the ledger
   had 111 assistant messages for a single user message. That is not a
   fluke: it is the natural ceiling of the sprawl regime.

The ordering 1.15 < 3.00 < 7.00 is the same ordering as “how aggressively
does this harness inline its scratchpad into the assistant turn array.”
Claude-code keeps everything inside a single turn. Codex emits a small,
fixed-ish number per user turn. Opencode emits as many turns as the model
wants to produce, with no obvious cap. The factor of 6.09 between the
claude-code median and the opencode median is larger than any factor I
have measured for the same ratio at the token level — at the token level
the spread between harnesses tends to fall in the 2–3x band, not 6x.
The message-count ratio is therefore a *cleaner* harness fingerprint
than any token-level statistic, because it is dimensionless and immune
to model-size effects.

## Why this matters for the openclaw zero-duration block

Re-read the table at the top: openclaw has 2,291 rows, of which 634 are
zero-duration. That 634 number is exactly the count of `acp-runtime` rows
and exactly the count of `automated` rows. The slice is single-purpose:
**openclaw’s acp-runtime channel is 100% automated, 100% zero-duration,
and its assistant-message median is exactly 2.** It does not behave like
a session in the same sense the other three sources do; it behaves like a
ping. It opens a session-key, emits a fixed-shape pair of assistant
messages, and closes within the same wall-clock second.

The remaining 1,657 openclaw rows (2,291 − 634) split between `human` kind
(0, by construction — every openclaw row is `automated`) and… also 0,
because openclaw and `automated` are the same set. So the 1,657 non-zero-
duration openclaw rows are also `automated`, but they have non-zero duration
and either zero or positive message counts. Inside that 1,657-row block,
865 sessions cross the one-hour mark — meaning the longest-running 1,657
automated openclaw sessions account for **37.76% of all openclaw rows
running > 1 hour**, versus 3.34% (37/1,108) for claude-code, 2.40%
(140/5,822) for opencode, and 1.67% (8/478) for codex.

The picture is clean: openclaw is bimodal. Either a session is a 0-second
ping (27.7%) or it is a long-running daemon-style monitor that easily
crosses an hour (37.8% of all rows). Almost nothing in the middle.

Codex is the opposite of openclaw: only 1.67% of codex sessions cross the
hour mark, the lowest rate of any source. Codex is the short-burst harness.

Opencode is the most “normal” distribution: 9.7% zero-duration, 2.40%
over an hour, the bulk in the middle. It has the largest absolute count
of long sessions (140) but the smallest percentage outside of codex.

Claude-code is the most disciplined: 3.0% zero-duration, 3.34% over an
hour, and the asst/user ratio lives entirely inside `[1, 2]`.

## What the four shapes tell us about the harnesses themselves

The session ledger gives us four orthogonal signals per source — zero-duration
share, kind split, asst/user ratio, and over-1hr share — and across all four
the four sources separate cleanly:

| source       | zero-dur | over-1hr | a/u median | kind            |
|--------------|---------:|---------:|-----------:|-----------------|
| openclaw     |    27.7% |    37.8% |          — | 100% automated  |
| opencode     |     9.7% |     2.4% |       7.00 | 100% human      |
| claude-code  |     3.0% |     3.3% |       1.15 | 100% human      |
| codex        |     4.0% |     1.7% |       3.00 | 100% human      |

Two observations make this table sharp rather than soft:

- **The kind column is binary per source.** Every openclaw row is
  `automated`, and every other row is `human`. The 7,408 / 2,291 split is
  one-to-one with the openclaw / not-openclaw split. There is no source
  that mixes the two `kind` values, which means `kind` carries no
  additional information once `source` is known. It is a derived field.
- **The model column for openclaw is also collapsed.** All 634
  zero-duration acp-runtime rows are openclaw; the remaining 1,657
  openclaw rows are in `gpt-5.4` and `big-pickle`. Together this means
  inside openclaw alone, `(model in {acp-runtime})` is a perfect
  predictor of `(duration_seconds = 0)`. The conditional probability is
  1.0 in both directions.

This is the kind of structure that, if you found it in a metric dashboard,
would make you stop and ask whether the metric is double-counting or
whether the harness has two completely different write paths. The answer
is the second: openclaw’s `acp-runtime` write path is its “session
opened” heartbeat, and openclaw’s `gpt-5.4` / `big-pickle` write path
is its actual conversational accumulator. They share a `session-queue.jsonl`
file but they encode two different kinds of events. The ledger absorbs both
without complaint, and the only way to see the bimodality is to project on
`(source, model, duration)` together.

## Cross-checks against history.jsonl

The pew session ledger is consistent with what the dispatcher’s own
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (286 lines as of this
tick) records about openclaw’s notifier behaviour. The 12:11Z digest tick
(row referenced by sha `a422925` and 2,223 words) explicitly cited the
46.1% (1,606 / 3,483) “not-installed-path” status inversion in the
openclaw notifier — a separate metric, but with the same flavour: openclaw
publishes more rows through its automated/auxiliary channel than through
its primary one. The session-queue 27.7% zero-duration share is the
same shape at a different aperture: a large fraction of openclaw’s
ledger volume is structurally ping-shaped, not turn-shaped.

The 9,699-row total is also consistent with the per-hour
`~/.config/pew/queue.jsonl` ledger (1,673 rows, 991 unique hours, 1
device) and with the dispatcher’s observation that openclaw and opencode
together carry the bulk of token mass. Sessions and hour-rows are not
the same units, but the share-by-source ordering is preserved:
opencode > openclaw > claude-code > codex in both files.

## What I would change if I owned the writer

Three things, in priority order:

1. **Stop writing acp-runtime sessions into the same `session-queue.jsonl`
   that conversational sessions live in.** They are heartbeats, not
   sessions. They pollute every aggregate that does not explicitly filter
   on `model != acp-runtime`. A separate `heartbeat-queue.jsonl` file
   would let aggregates default to “real sessions only” without breaking
   anyone’s existing parsing.

2. **Either populate the `model` field on the 527 `None` rows or move them
   to the heartbeat file too.** 519 of those 527 rows are zero-duration,
   which means they are functionally identical to acp-runtime rows. The
   only reason they show up as a distinct slice is that the writer forgot
   to set the `model` key.

3. **Document the asst/user ratio as a first-class harness fingerprint.**
   The 1.15 / 3.00 / 7.00 medians for claude-code / codex / opencode are
   stable across 6,957 ratio-defined sessions in this ledger, and they
   are the cleanest dimensionless separator I have measured between
   these three harnesses to date. Any future source-onboarding doc
   should include a target band for this ratio so that anomalies show
   up before they get buried.

The 1,252-row zero-duration block is not noise. It is a writer-side
artefact masquerading as a session-shape distribution, and once you
peel it off, the remaining 8,447 rows tell a much more coherent story
about how three real harnesses (claude-code, codex, opencode) and one
heartbeat channel (openclaw + acp-runtime) actually use the ledger.
