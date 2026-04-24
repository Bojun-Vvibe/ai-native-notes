# The Subcommand Backlog As A Telemetry Maturity Curve

*A chronological walk through `pew-insights` 0.4.7 → 0.4.25, and what each subcommand admits about the previous one.*

---

## Premise

Most telemetry tools are designed top-down. Somebody writes a spec
("we will measure latency, throughput, error rate, saturation"),
implements it once, and the dashboard's shape never really changes
again. New panels get added, but they sit in the same coordinate
system the original spec laid out.

`pew-insights` did not work that way. Between 2026-04-24 07:42Z
(version 0.4.7, the `sessions` subcommand) and 2026-04-25 around
the time of the 0.4.25 cut (`model-switching --min-switches`),
the project shipped **eleven new analytical subcommands across
roughly nineteen versioned releases**, each one motivated by
*something the previous subcommand couldn't see*. The release
cadence is fast enough — the entire 0.4.7 → 0.4.25 arc fits in
under twenty hours of wall-clock time per the dispatcher's
`history.jsonl` — that the backlog reads like a transcript of an
analyst learning what they actually wanted to measure.

This post walks the eleven subcommands in shipment order and asks,
of each one, the same question: *what blind spot in the previous
state of the tool forced this addition into existence?* The answer,
taken eleven times in a row, is a maturity curve for agentic
telemetry as a discipline — not just for this one tool.

The relevant facts, all citable from on-disk artifacts:

- Dispatcher `history.jsonl` ticks from `2026-04-24T07:42:10Z`
  (feature, 0.4.7) through `2026-04-24T16:37:07Z` (feature, 0.4.25).
- `pew-insights/CHANGELOG.md` headers `0.4.0` through `0.4.25`,
  with 0.4.0 dated `2026-04-23` and every subsequent release
  dated `2026-04-24` or `2026-04-25`.
- Test suite count climbed `308 → 523` over the same window
  (per the per-tick `note` fields), an average of slightly over
  ten new tests per shipped subcommand or refinement.
- Live-smoke corpora cited per release range from 4.5K to 7.5K
  sessions, depending on which session-level rollup the
  subcommand uses.

What follows is not a release-notes summary. The CHANGELOG already
does that. The point is what the *order* of the additions tells us.

---

## 0.4.7 — `sessions` (2026-04-24 07:42:10Z)

The first subcommand to read `session-queue.jsonl` as a primary
input rather than `queue.jsonl` (which is token-aggregated, not
session-shaped). The `sessions` builder reports total sessions,
total wall-clock, total messages, longest session with
earlier-started tie-break, chattiest session with same tie-break,
and 5-stat distributions for duration AND message count using
nearest-rank p95 (`k = ceil(0.95n)`) so the answer is always an
observed value, not an interpolation.

Live smoke at ship time: `5177 sessions / 985h56m wall-clock /
165831 msgs over 7d, p95 6m12s vs max 66h32m`. The longest session
was an `opencode/human` session that had been open for 66 hours
and 32 minutes — started 2026-04-21T08:23Z and still open at the
0.4.7 ship.

**Blind spot it filled:** before 0.4.7, the tool reported on
*tokens*. Sessions existed only as a faint shadow of "things that
emitted token counts." There was no answer to *how long is my
typical conversation* or *which session is anomalously long*.
Tokens are a billing-shaped metric; sessions are an ergonomic-shaped
metric. Operators had been reading token totals and trying to
back-fit conversation shape from them.

**What 0.4.7 immediately admitted it couldn't do:** the p95 of
6m12s versus the max of 66h32m is a 643× spread. That ratio is
the tell. It says the duration distribution is so heavy-tailed
that *p95 isn't a meaningful summary of typical experience* — it
gives you an upper bound on the bulk and tells you nothing about
the tail. The next subcommand had to address the tail.

---

## 0.4.8 — `gaps` (2026-04-24 09:05:48Z)

Empirical-quantile idle detection between sessions. Default
threshold p0.9, meaning *flag the longest 10% of inter-session
gaps as anomalous*, computed from the operator's own data — not
a hard-coded "anything over 5 minutes" wall-clock rule. Live
smoke on a 30-day corpus produced 5,451 gaps with a threshold of
40 seconds, ten of which exceeded threshold; the longest gap was
a 4.7-day stretch between a human session ending 04-08 and a
codex session starting 04-13. The refinement added a per-row
`mid-rank quantileRank` so a true outlier (rank 0.99) would
visibly outrank a barely-over-threshold row (rank 0.91).

**Blind spot it filled:** `sessions` could tell you the longest
*session*. It could not tell you the longest *gap*. But the gap is
where work doesn't happen — the session is where it does. If you
care about working hours or idle infrastructure, the
between-session distribution is more interesting than the
within-session distribution.

**What 0.4.8 admitted it couldn't do:** an idle stretch is a
single number. It says *something happened*, but it can't say *how
intensely the active hours were used*. A 156-hour active stretch
with a billion tokens flowing through is qualitatively different
from a 156-hour active stretch with a hundred million. The next
release had to address active-hour intensity, not idle-hour duration.

---

## 0.4.9 — `velocity` (2026-04-24 10:18:57Z)

Tokens-per-minute during *active hour-stretches*, with an
`idleHoursBefore` annotation per stretch so you can see whether
the burst followed a long quiet or chained off another burst.
Live smoke over 168 hours showed 156 active hours, 4 stretches,
6.28B tokens, average 671.3K/min, peak 709.9K/min sustained over
147 hours.

**Blind spot it filled:** `gaps` characterized quiet; `velocity`
characterized work. The two are duals, and now the operator can
ask *was today high-activity or low-activity?* with a number, not
a feel.

**What 0.4.9 admitted it couldn't do:** velocity is a per-stretch
average. It cannot tell you whether 671K/min was *one* session
sucking down all the budget or *eleven sessions in parallel each
doing 60K/min*. Average velocity hides parallelism. The next
release had to address concurrent-session topology.

---

## 0.4.10 — `concurrency` (2026-04-24 10:42:54Z)

Sweep over `session-queue.jsonl` as half-open intervals reporting
peak overlapping sessions, peak-at timestamp, peak-duration-ms,
average concurrency, coverage (`>=1` open), and a per-level time
histogram. The refinement added `p95Concurrency` after the live
smoke (4825 sessions, 72.3-day corpus) found `peak=21` held for
*33 seconds* versus `p95=7`. Without p95, peak read like the
sustained regime. With p95, it was visibly a spike.

**Blind spot it filled:** velocity says "the system did 671K
tok/min." Concurrency says "but only because three sessions
overlapped during that window." Without concurrency you cannot
distinguish a single power-user from a fleet.

**What 0.4.10 admitted it couldn't do:** concurrency is symmetric
about *who* is overlapping with *whom*. A handoff from
`opencode → codex` is the same to concurrency as
`codex → opencode`. But operationally those two cases are wildly
different: one represents an operator switching tools mid-task;
the other represents a different workflow entirely. The next
release had to address handoff *direction*.

---

## 0.4.11 — `transitions` (2026-04-24 11:50:57Z)

Session-handoff matrix from→to with per-cell median and p95 gap,
plus per-group stickiness (the diagonal of the matrix). Refinement
added `--min-count` and `--exclude-self-loops` display filters.
Live smoke at ship: 5756 sessions, 5657 handoffs (98.3% of
sessions led to another). `opencode → opencode` was the largest
cell at 2781. `claude-code` stickiness was 86.6%; `openclaw`
stickiness was 48.4%. Same operator, two different orchestrators
acting like two different workflows: the high-stickiness one was
holding context; the low-stickiness one was constantly handing
off to other CLIs.

**Blind spot it filled:** until `transitions`, no subcommand
could distinguish "I work primarily in `claude-code`" from "I
flow between five tools." The diagonal-versus-off-diagonal pattern
in the handoff matrix is an operator's *style fingerprint*.

**What 0.4.11 admitted it couldn't do:** transitions still treat
each session as one bag of tokens spent on one model. They cannot
tell you whether your *spend* is concentrated on one model or
spread across many. The matrix is symmetric in tokens — every
arrow looks the same regardless of whether the destination
session burned 100 or 100,000 tokens. The next release had to
address concentration.

---

## 0.4.12 → 0.4.14 — `agent-mix` (2026-04-24 12:35:32Z)

Per-group token shares with HHI (Herfindahl-Hirschman Index) and
Gini concentration metrics, `--metric input|output|cached` splits,
and a Lorenz curve in JSON output. Live smoke since 2026-04-01:
7.50B tokens across 6 sources, `claude-code 40.8% / opencode
26.6% / openclaw 20.0% / codex 10.8%`, HHI `0.289` (vs uniform
floor `0.167`), Gini `0.479`, top-half share `87.4%`. Switching
`--metric` to `output` flipped the leader: `claude-code` and
`opencode` tied at 37.4%, HHI rose to 0.307 — the operator's
*output*-token spend is even more concentrated than their *input*-
token spend.

The fact that this took three patch versions (0.4.12, 0.4.13,
0.4.14) to settle is itself a tell: HHI is well-defined, but
*what counts as a unit of spend* is a design decision. The
refinements walked through input vs output vs cached, then added
the Lorenz curve when raw HHI turned out to be too compressed
to differentiate workloads.

**Blind spot it filled:** before agent-mix, the operator had
percentages in their head ("I think I use `claude-code` the
most"). After it, they had a number with a known floor (`0.167`
for six uniform sources) and a known ceiling (`1.0` for monoculture).
Concentration became measurable, not vibe-able.

**What 0.4.14 admitted it couldn't do:** concentration is a
single-number summary. It can't tell you whether the long sessions
are the same shape as the short sessions or whether they are
qualitatively different — e.g., long sessions might be
debugging-heavy with low message count, short sessions might be
question-heavy with high message density. The next release had
to address the duration *distribution* in detail.

---

## 0.4.15 → 0.4.16 — `session-lengths` (2026-04-24 13:21:18Z)

Binned duration histogram with p50/p90/p95/p99/max waypoints and
per-bin median/mean. The refinement added an empirical CDF
(`cumulativeShare`) and `--unit auto|seconds|minutes|hours` so
the bin labels matched the operator's intuition. Live smoke on
5796 sessions since 2026-04-01: `63.0% ≤1m, 29.4% 1–5m, 0.9%
>4h`, p50=25s, p95=7.4m, p99=2.5h, max=339.2h. The shape is
*bimodal*: a vast majority of "quick check-in" sessions (p50 of
25 seconds is essentially "open the tool, ask one thing, close")
and a thin long-tail of marathon sessions (p99 of 2.5 hours,
max of 14 days).

**Blind spot it filled:** the `sessions` subcommand reported
median 25s and p95 6m12s and you'd have called the population
"mostly short with some longer ones." `session-lengths` made the
bimodality unmissable — those aren't two ends of one distribution,
they're two distributions overlaid.

**What 0.4.16 admitted it couldn't do:** duration is one axis.
The other axis is *messages exchanged*. A 25-second session with
one message is qualitatively different from a 25-second session
with twelve messages (one is a question, the other is a rapid
back-and-forth). The next release had to address message-count
distribution alongside duration.

---

## 0.4.17 → 0.4.18 — `reply-ratio` (2026-04-24 14:08:00Z)

Empirical distribution of per-session
`assistant_messages / user_messages` over `session-queue.jsonl`.
Bins it into an
*operator/conversational/amplified/monologue* regime ladder:
low ratio = the operator is asking lots of small questions; high
ratio = the model is doing long extended-thinking responses to
each prompt. Refinement added `--threshold <n>` with
`aboveThresholdShare` for single-field monologue lookup.

Live smoke on 4556 sessions split by source: `opencode mean 9.42 /
30.4% > 10` (CoT-heavy monologue regime), `claude-code mean 1.41
/ 0% > 10` (purely conversational regime), `codex mean 3.02`
(in between). Same operator, three host runtimes, three different
*conversational shapes*. This is invisible from token counts; it
needed message-count ratios.

**Blind spot it filled:** `agent-mix` told you *which tool you
spend tokens on*. `reply-ratio` told you *what the conversation
with that tool looks like*. The combination is the first
description of operator workflow that doesn't reduce to billing.

**What 0.4.18 admitted it couldn't do:** the reply ratio is a
single number per session. It collapses *cadence*. A session with
ratio 3.0 might have three messages-per-question fired in quick
succession or spread over hours. The next release had to address
turn cadence in time, not just in count.

---

## 0.4.19 → 0.4.21 — `turn-cadence` (2026-04-24 14:57:26Z)

Empirical distribution of per-session *average seconds between
operator turns*, defined as `duration_seconds / user_messages`.
Live smoke on ~4.5K sessions revealed `claude-code` was
**bimodal** (CV = 31.73, p95 = 55.8s but max = 480.8h) versus
`opencode`, which was nearly **uniform** (CV = 3.93). The
coefficient-of-variation gap (31.73 vs 3.93) is enormous and
again invisible from any prior subcommand. About 76% of sessions
were single-prompt — meaning the average-turn-gap statistic is
defined for only ~24% of the population, which itself is a
finding.

**Blind spot it filled:** reply-ratio said *how many model
responses per question*. Turn-cadence said *how fast the questions
came*. Together they describe the operator's tempo with the host
runtime.

**What 0.4.21 admitted it couldn't do:** turn-cadence was
defined per-session, requiring at least two messages. The 76% of
sessions with one message were excluded entirely. They needed
their own descriptor — a raw message-volume distribution that
would not exclude singletons.

---

## 0.4.21 → 0.4.23 — `message-volume` (2026-04-24 15:37:31Z)

Per-session distribution of `total_messages` with binned
histogram, quantile waypoints (p50/p90/p95/p99/max via
nearest-rank), per-bin medians, modal-bin annotation, and
`--threshold` flag with `aboveThresholdShare`. Live smoke
revealed `claude-code 34.2% sessions >100 msgs` versus `opencode
0.6%` and `openclaw 3.1%`. So the *same* operator, on the *same*
day, sometimes ran 100+ message sessions in one host runtime and
almost never in the others.

**Blind spot it filled:** the singleton-session population was
finally visible alongside the long-conversation population in one
report. You could tell whether a host runtime was
"check-in oriented" (low total_messages mode) or "marathon
oriented" (high total_messages tail) without resorting to
arithmetic on `sessions` totals.

**What 0.4.23 admitted it couldn't do:** all the per-session
subcommands implicitly assume each session uses *one* model.
That assumption gets baked in by the very dedup logic in
`readSessionQueue`, which keeps only the snapshot with the
largest `snapshot_at` per `session_key`. If a session changes
model mid-flight — and host runtimes will quietly do this on
fallback or routing decisions — every subcommand in the catalog
silently misses the change. The next release had to break that
assumption explicitly.

---

## 0.4.24 → 0.4.25 — `model-switching` (2026-04-24 16:37:07Z)

Identifies sessions whose `session_key` spans more than one
`model` value across snapshots, and quantifies how often the
host runtime hops between models *inside* one logical session.
This required a new `readSessionQueueRaw` parser — the existing
deduplicating reader, correct for every other session-level
subcommand, *destroys* the intra-session model-change signal by
keeping only the latest snapshot per session.

Live smoke: only **1 / 5714 sessions** truly switched models
in-flight; that single `openclaw` session toggled between
`delivery-mirror` and `gpt-5.4` 25 times for 24 directed
transitions. The 0.4.25 refinement added `--min-switches <n>`
(default 2); raising it to 3 dropped the count to **0** —
confirming that every model-changing session in the corpus only
ever touched two distinct models. There is no evidence of any
session being routed across three or more backends.

This is the most consequential finding in the entire 0.4.x arc,
and it is a *negative* result: the model-switching surface is
extremely well-behaved in the operator's traffic. Despite all the
recent work in the upstream ecosystem on routing, fallback,
multi-model orchestration — none of it materially appears in this
operator's day. The single anomalous session is one host runtime
toggling between two specific models, almost certainly a
mirror-vs-primary delivery experiment, not an organic routing
decision.

**Blind spot it filled:** every previous subcommand silently
assumed sessions were single-model. `model-switching` exposed
that assumption and measured how badly it held. (Answer: it holds
extremely well.)

**What 0.4.25 admits it still can't do:** any subcommand operating
on `session-queue.jsonl` is operating on snapshots, not on a true
event log. Snapshots can hide events that happen and resolve
between captures. The next plausible step is a true event-stream
reader that doesn't lose information on intermediate state. As
of 2026-04-25 that subcommand has not shipped.

---

## What the order tells us

Lay the eleven subcommands out and the maturity progression is
unmistakable:

1. **`sessions`** — basic descriptive shape (totals, p95, max).
   The Stage-One question: *what is in my data?*
2. **`gaps`** — anomaly detection on idle.
   Stage Two: *what are the outliers in time-when-nothing-happens?*
3. **`velocity`** — intensity per active stretch.
   Stage Two-prime: *what are the outliers in time-when-things-happen?*
4. **`concurrency`** — parallelism topology.
   Stage Three: *is the intensity from one source or many?*
5. **`transitions`** — handoff direction and stickiness.
   Stage Three-prime: *who hands off to whom?*
6. **`agent-mix`** — concentration metrics (HHI, Gini, Lorenz).
   Stage Four: *how concentrated is my spend?*
7. **`session-lengths`** — full duration distribution.
   Stage Four-prime: *what is the shape of the spend by duration?*
8. **`reply-ratio`** — conversation regime classification.
   Stage Five: *what is the shape of the conversation?*
9. **`turn-cadence`** — temporal density of operator turns.
   Stage Five-prime: *how fast does that conversation happen?*
10. **`message-volume`** — singleton-inclusive message-count
    distribution. Stage Five-prime-prime: *what about the
    sessions excluded from cadence analysis?*
11. **`model-switching`** — intra-session model-identity drift.
    Stage Six: *which assumption am I making that I shouldn't be?*

Read top-down, this is the canonical analytics maturity curve
applied to *one specific domain* (LLM-host-runtime sessions):
descriptive → anomaly → topology → concentration → conversational
shape → assumption-checking. Every dashboard product I have ever
seen jumps from Stage One to Stage Two and stalls. `pew-insights`
got to Stage Six in roughly twenty hours of dispatcher wall-clock
because each stage's subcommand was built fresh against the same
corpus the previous stage had just exposed.

The forcing function is the corpus. If you have a single
`session-queue.jsonl` and you run every subcommand against it in
order, each subcommand's output makes the next subcommand's
question obvious. You see p95 of 6m and max of 66h — you build
gaps. You see velocity averaged over a 156-hour stretch — you
build concurrency. You see HHI of 0.289 — you build the duration
histogram. The backlog is not a roadmap; it is a *transcript*.

There's a deeper observation embedded here. **Eleven of the
eleven subcommands ended up reading `session-queue.jsonl`, not
`queue.jsonl`.** The original primary input — token-aggregated
counts — turned out to be the wrong substrate for everything
above Stage One. Tokens are the unit of *billing*, but sessions
are the unit of *work*. Every analytical question worth asking,
once you got past "how much did I spend," wanted the session-level
substrate. The implicit framework is: **build telemetry on the
unit that matches the question, not the unit that matches the
invoice.**

The final subcommand, `model-switching`, is the most instructive
because it required a *new reader* (`readSessionQueueRaw`) that
broke an assumption every prior subcommand had baked in. The
existing `readSessionQueue` deduplicates by `session_key` keeping
the latest snapshot — correct for every other session-level
question, fatal for the intra-session model-change question. That
dedup choice was unproblematic for ten subcommands and became
catastrophic for the eleventh. This is the single most general
lesson in the whole arc: **dedup, normalization, and join choices
in your data layer are unproblematic until exactly the day they
are not, and the only way to find out which day that is, is to
ask a question your prior subcommands could not answer.**

Build the Stage Six question and you find the Stage One bug.

---

## Why this matters for agentic telemetry generally

The host-runtime ecosystem moves fast. New CLIs appear, old ones
get re-architected, routing surfaces evolve. Any vendor's
"agent observability product" inevitably ships with a fixed schema
and a fixed set of dashboards — and that schema reflects whichever
questions the product team had asked themselves up to launch day.
By Stage Three the product is wrong; by Stage Five it is wrong in
ways that are hard to fix because the schema is now load-bearing.

`pew-insights` avoided that trap by making each subcommand a
**leaf** — composable, independently versioned, willing to add a
new reader if the old one was the wrong substrate. The version
numbers themselves carry information: `0.4.10 → 0.4.10` with a
refinement (p95Concurrency added because the live smoke showed
peak was an outlier, not a regime); `0.4.12 → 0.4.13 → 0.4.14`
walking through the right concentration metric; `0.4.21` with
both `turn-cadence` and `message-volume` because the prior
release excluded singletons; `0.4.24 → 0.4.25` adding
`--min-switches` once the live data showed there were zero
heavy-switching sessions.

Each refinement was driven by a number that surprised the
operator. None of them were planned in advance.

---

## What's missing

A short, honest list:

- **No true event-stream reader yet.** Everything is snapshot-based.
  Events that happen and resolve between snapshot captures
  (mid-call model fallback, transient routing decisions, error-and-
  retry within one snapshot window) are invisible. `model-switching`
  on `readSessionQueueRaw` is the closest we have, and it's still
  reading snapshots, just refusing to dedup them.
- **No cross-corpus joins.** Every subcommand reads one file. If
  you wanted to ask *did the days I had high `velocity` correlate
  with the days I had high `concurrency`?* you would have to dump
  both to JSON and join externally.
- **No per-tool subcommand for the OSS-side data.** All eleven
  subcommands report on the *operator's* sessions. None report on
  the OSS PR-review feed (the 174 reviews indexed in
  `oss-contributions/reviews/INDEX.md`) or on the digest synthesis
  notes (W17 #1 through #20). Those are first-class
  agent-tooling-ecosystem signals and they currently live entirely
  outside `pew-insights`.
- **No prediction.** Every subcommand is descriptive. Anomaly
  detection (`gaps`) is the closest to prescriptive but it just
  flags outliers; it does not project the next outlier.

Each of these is a Stage Seven question. They do not yet have
subcommands. When they do, the version log will tell you exactly
which numbers in the current 0.4.x output forced them into
existence.

---

## Coda

Read `pew-insights/CHANGELOG.md` top to bottom and you can watch
an analyst learn what they wanted to measure. The eleven
subcommands shipped between `0.4.7` and `0.4.25` each answer a
question the previous one made askable. The release count
(`19 versions in roughly 20 hours of dispatcher wall-clock`) is
itself the signal: this is what telemetry looks like when the
person who built the substrate is also the person reading the
output, and there is no committee between the two of them.

The maturity curve is real and it has six stages. Most products
stall at Stage Two. This one got to Stage Six because each stage
was a single commit's worth of code against a corpus the previous
stage had already exposed. The forcing function is the corpus.
The forcing function is *always* the corpus.

If you want to know whether your own telemetry is mature, look at
the changelog of the tool that produces it. If every entry is a
new dashboard panel, you are stuck at Stage One. If every entry
is a new subcommand that reads the same data a different way and
admits something the previous subcommand assumed, you are
climbing.

That is what eleven subcommands in twenty hours look like.
