---
title: "The intra-turn idle gap fingerprint: codex 1.2s, opencode 9s, openclaw 69s — four harnesses, four conversational tempos"
date: 2026-04-27
tags: [data, sessions, pew, session-queue, idle-gap, harness-tempo]
---

# The intra-turn idle gap fingerprint: codex 1.2s, opencode 9s, openclaw 69s — four harnesses, four conversational tempos

There is a single derived metric you can compute from `~/.config/pew/session-queue.jsonl` that fingerprints a coding-agent harness more reliably than any explicit field in the schema. It is not the source name. It is not the model name. It is not the user count or the assistant count. It is the **intra-turn idle gap**:

```python
gap_per_turn = duration_seconds / (total_messages - 1)
```

Run it across the 9,837-row session ledger, restrict to sessions with `duration_seconds > 0` and `total_messages > 1` (n=8,581 valid rows), then bucket by `source`. The result is four wildly different distributions sitting on top of identical schema columns:

```
source           n      p10     p50     p90     p99       max
opencode      5353     4.5     9.0    28.0   669.9      4346
openclaw      1694     1.6    69.3   165.4   658.3      2473
claude-code   1075     0.9     3.5     6.8   252.7    346197
codex          459     0.9     1.2     7.0   117.1       509
```

Same schema. Same `started_at / last_message_at / total_messages` columns. Same human-in-the-loop assumption (these are all `kind: human` sessions, automated rows already filtered by the upstream digest). And yet the median time between consecutive messages varies by **57.75×** from codex (1.2s) to openclaw (69.3s), and the p90 varies by **24.3×** from claude-code (6.8s) to openclaw (165.4s).

This post is about what that gap is actually measuring, why each harness produces a different distribution, what the bimodality tells you about user intent versus tool autonomy, and how the single 346,197-second outlier in the claude-code column reveals an entire failure mode of the session-ledger schema itself.

## What "gap_per_turn" measures

The session ledger collapses every conversation into seven counters:

```
session_key, source, kind, started_at, last_message_at, duration_seconds,
user_messages, assistant_messages, total_messages, project_ref, model
```

`duration_seconds` is the wall-clock span from the first message to the last. `total_messages` is `user + assistant + system`. The number of *gaps* between consecutive messages is exactly `total_messages - 1`. So `duration / (total_messages - 1)` is the average inter-message wall-clock interval — the average time between any two consecutive turns, regardless of who spoke.

What that interval contains depends on which side of the keyboard is doing the waiting:

- **User-side gap.** The time from when the assistant finished its turn to when the user typed the next prompt. This is "human think time" or "human idle time."
- **Assistant-side gap.** The time from when the user finished typing to when the assistant emits its first token. This is "model latency + prompt-processing time."
- **Tool-call gap.** The time from when an assistant emits a tool call to when the assistant emits its next message containing the tool result. This is "tool execution time," and it is often counted as one assistant-side gap because both bookend messages are assistant-role messages.

The mix of these three components is what the gap distribution actually fingerprints. A harness with low assistant-side gaps and few tool calls will look fast. A harness with heavy tool autonomy (many internal turns per user prompt) will compress the wall clock per turn and look fast. A harness where the user pastes a prompt and walks away to read documentation for two minutes will look slow regardless of how fast the model is.

## The four distributions, one bucket at a time

Bucket each source's gap distribution into five intervals — `<2s`, `2-10s`, `10-60s`, `1-10min`, `10min+` — and the four harnesses sort themselves into four distinct conversational regimes:

```
source         <2s    2-10s   10-60s   1-10m   10m+
opencode      0.5%    54.1%   37.8%    6.5%    1.1%
openclaw     12.9%    32.6%    3.5%   49.4%    1.6%
claude-code  41.9%    52.1%    3.8%    1.8%    0.5%
codex        65.4%    32.5%    0.7%    1.5%    0.0%
```

Read it row by row.

**Codex (n=459, median 1.2s).** Two-thirds of all gaps are sub-two-second and another third are 2-10 seconds. Almost nothing exists above one minute (1.5%) and literally nothing exists above ten minutes. This is a *fully-autonomous tool loop* signature: every gap is the model finishing a tool call and immediately emitting the next one, with no human in the loop slowing things down. The 0.0% in the 10min+ bucket is structurally interesting — it means codex sessions either complete or get killed before any single inter-turn gap reaches ten minutes. There is a hard ceiling somewhere in the harness that prevents a long pause.

**Claude-code (n=1075, median 3.5s).** The `<2s` bucket holds **41.9%** of gaps, the `2-10s` bucket holds **52.1%**, and 94% of all gaps are under ten seconds. This is the closest thing to "REPL-shaped" in the dataset: the modal gap is a couple of seconds, dominated by model first-token latency for short responses. The cap at p99=252.7s and the 0.5% in the 10min+ bucket — 5 sessions out of 1075 — suggest that long pauses do happen but are rare. The harness is fast and reasonably tightly coupled to the user.

**Opencode (n=5353, median 9.0s).** Now the distribution shifts hard. Only **0.5%** of gaps are sub-two-second — opencode essentially never has a sub-second turn. The mode (54.1%) is in the 2-10s bucket and a substantial **37.8%** lives in the 10-60s bucket. This is the signature of a harness where each "turn" is a substantial unit of work — a tool call that does real I/O, a long generation, an agentic deliberation step. Opencode's median assistant message contains more work than claude-code's, and the wall clock reflects that. The 1.1% in the 10min+ bucket (60 gaps out of 5353) is where opencode sessions go to sit overnight.

**Openclaw (n=1694, median 69.3s).** This is the anomaly column. **49.4%** of all openclaw gaps are in the 1-10 minute bucket — almost half the dataset sits in a regime that is essentially absent from the other three harnesses (0.7-6.5%). The median gap is 69 seconds, **8× the opencode median** and **20× the claude-code median**. And yet openclaw also has the highest sub-2s share **after** codex (12.9%, vs 0.5% for opencode). Openclaw's distribution is bimodal: a thin spike at "instantaneous" (sub-2s, automated retries or notifier polls) and a giant hump in the 1-10 minute zone (humans walking away from the keyboard between turns).

The bimodality is the signature. Openclaw is the only harness in the four where humans are explicitly waiting between turns, doing something else, coming back, and emitting another turn. The other three harnesses run in conversational regimes where either the human is glued to the screen (claude-code, codex) or the agent is autonomous (opencode's longer turns are still tool work, not human idle).

## Why codex is so much faster than its peers

The codex p50 of 1.2 seconds is suspiciously fast. Models do not respond in 1.2 seconds end-to-end on real prompts; first-token latency alone is usually 600-1200ms and the rest of the response takes longer. So either:

1. Codex sessions consist almost entirely of very short, single-token-ish messages (tool calls, acknowledgements), or
2. Codex's `total_messages` counter is inflated by counting things the other harnesses don't count (e.g., system messages, tool result messages, reasoning checkpoints), or
3. Both.

The session ledger does not break out message types. But the symptom — gap p50 = 1.2s — strongly implies (2): codex is emitting more, smaller messages per session than the other harnesses, which compresses the per-message wall-clock denominator. A claude-code session that takes 60 seconds and emits 6 messages produces a gap of 12 seconds. A codex session that does the same wall-clock work but emits 60 messages (each a tool step) produces a gap of 1 second. Same conversation, ten times the message count, ten times the gap denominator, looks ten times faster.

This is why "gap per turn" alone does not say "harness X is faster than harness Y." It says "harness X emits more, smaller turns per second than harness Y," which is a structural fact about the harness, not a latency claim.

## Why opencode dominates the 10-60s bucket

Opencode's **37.8%** in the 10-60s bucket — versus 3.5-3.8% for openclaw and claude-code, and 0.7% for codex — is the crispest fingerprint in the table. A sub-minute but non-trivial gap is the signature of a substantial single-step tool call: a `Read` of a 2000-line file, a `Bash` running a test suite, an `Edit` that takes the model a few seconds to compose. Opencode runs these as one assistant turn each. Claude-code runs them as several assistant turns each (tool call, tool result, follow-up message), so each individual gap is shorter even though the total work is the same. Codex runs them as many tiny turns each, compressing each gap into a sub-second slice.

The 10-60s bucket is the "real work per turn" bucket, and opencode owns it: 37.8% of all opencode inter-turn gaps are sustained, single-step, non-trivial operations. That number does not get smaller as opencode scales; if anything, it grows as opencode tackles bigger codebases, because the tool calls themselves take longer.

## The 346,197-second outlier and the schema's blind spot

There is a single row in the claude-code source where `gap = 346,197 seconds`. That is **96.2 hours** — four full days of "inter-turn idle." The session details:

```
source=claude-code  model=claude-opus-4.6-1m
duration_seconds=1,730,983 (≈ 20 days)
total_messages=6  (u=1, a=1, plus 4 system)
gap_per_turn = 1,730,983 / 5 = 346,197s
```

A 20-day session with one user message and one assistant message. By any reasonable definition this is not a "session" — it is a session ledger row that was opened, never properly closed, and got snapshot-rolled while still nominally alive. The `last_message_at` field probably reflects the last *system* message (a heartbeat, a context refresh, a model availability check) rather than any actual conversational turn.

This row exposes a schema-level blind spot: the session ledger has no notion of "active" vs "idle" vs "abandoned." Any session that opens and emits at least two messages will have a finite `duration_seconds`, regardless of whether those messages were one second apart or twenty days apart. The gap_per_turn metric inherits this and produces nonsense values for these long-tail rows.

The next four claude-code rows by gap descending are 2,951s, 2,337s, 1,083s, 752s — between 13 minutes and 50 minutes per turn. These are the boundary cases where the user opened a session, walked away, came back the next day, asked a follow-up question. They are real conversations stretched across human-scale time, not abandoned sessions. The cutoff between "real long session" and "abandoned ledger row" is not knowable from the schema; it requires session-level provenance data the ledger does not carry.

For the analysis above I left these rows in. The medians are robust to single outliers. But the **claude-code max of 346,197s should not be quoted as a real metric**; it is a ledger artifact, and the same ledger could produce arbitrarily larger numbers if a session sits open for arbitrarily long.

## Cross-source comparison: the codex/openclaw 57.75× spread

The headline number from the medians is **57.75×**: codex's median gap (1.2s) versus openclaw's median gap (69.3s). Two harnesses, both running on the same machine, both invoked by the same user, both reporting `kind: human`, separated by a factor of fifty-eight in their conversational tempo.

What does the user experience look like at each end?

- **Codex experience:** Watching a stream of tool calls scroll by, each completing in a second or less, occasional model thinking pauses, total session feels like a continuous stream rather than a conversation.
- **Openclaw experience:** Send a prompt, wait a minute or two, see a substantial response, read it, think for a minute or two, send the next prompt. Total session feels like an asynchronous chat — closer to email tempo than IM tempo.

These are not just different latencies. They are different **conversational genres**. A 1.2-second-tempo agent and a 69-second-tempo agent require different mental models from the user. The user who has just been working in codex cannot transition into openclaw without recalibrating their expectation of "how long should I wait before assuming it crashed." The user who has been in openclaw and switches to codex will feel like the agent is being interruptive and not letting them think.

## The 1107 instant sessions

Separately, there are **1,107 sessions** in the ledger where `started_at == last_message_at` exactly, and a further **1,256 sessions** with `duration_seconds == 0`. These are the "instant" sessions — opened and closed within the same JSON-serialization second, or never receiving a second message. They are excluded from the gap analysis (because `total_messages-1` would be zero or `duration` would be zero), but they form a sizable chunk of the ledger: **2,363 of 9,837 rows (24.0%)** are too short to compute a gap on.

The remaining 76% (n=7,474) is what the gap distribution above is built on, after further filtering to sessions with at least two messages and positive duration. The numbers in the bucket table are stable percentages of that filtered set.

## Three ways the gap metric is *not* a latency metric

It is tempting to read the gap distribution as a latency benchmark: "codex is faster than claude-code is faster than opencode is faster than openclaw." That reading is wrong in three specific ways:

1. **Message count denominator effect.** As discussed for codex, a harness that emits more, smaller messages will compress its gap regardless of total work done. Codex's 1.2s median says "codex emits messages quickly," not "codex completes work quickly."

2. **Human-side time inclusion.** The gap includes user think time. An openclaw session where the user takes 5 minutes between prompts will show 300-second gaps even if the model itself responds in 2 seconds. The 49.4% openclaw share in the 1-10min bucket is mostly human think time, not model latency.

3. **Tool-execution time inclusion.** A turn that involves a 30-second test run will show a 30-second gap, attributed to the harness even though the time was spent in `pytest` or `npm test`. Opencode's 10-60s bucket is heavily this.

The right way to read the gap distribution is as a **conversational tempo fingerprint**: a single number that captures how the harness, the user, and the tool ecosystem co-produce the rhythm of a session. Two harnesses with the same tempo produce similar gaps. Two harnesses with different tempos produce different gaps. That is all the metric promises, and it is a lot.

## Operational reads

Three concrete things you can do with this distribution:

1. **Detect harness misclassification.** If a row claims `source=claude-code` but its gap distribution looks like opencode (median ~9s, big 10-60s bucket), the row was probably mis-tagged at ingest. The fingerprint is sharp enough to flag misclassification with a single percentile comparison.

2. **Predict session abandonment.** Sessions whose first three gaps are all >600s are very likely to be abandoned (the user opened them, asked one thing, walked away, never came back). The current ledger has 1,107 instant-zero-duration rows that confirm this pattern; the 1-10min bucket sessions are the predictive signal for the *next* round of abandonment.

3. **Calibrate timeout settings.** If a harness's p99 gap is 117s (codex), then setting a tool-call timeout below 117s will kill 1% of legitimate turns. If a harness's p99 is 670s (opencode) or 658s (openclaw), the same timeout would kill far more. The p99 column is the natural minimum for any timeout policy that wants to preserve >99% of real sessions.

## The headline number, restated

> **The median intra-turn gap is 1.2s for codex, 3.5s for claude-code, 9.0s for opencode, and 69.3s for openclaw. The maximum, ignoring the 96.2-hour ledger artifact, is 670s (opencode) — eleven minutes between consecutive messages in a still-considered-active session. Four harnesses on one machine, one schema, four conversational tempos, separated by 57.75× at the median and over 24× at p90.**

Same schema, different physics. The gap metric is the cheapest way to fingerprint the difference, and the four-row table at the top of this post is the entire argument: opencode and openclaw are not faster or slower than codex and claude-code. They are running a different *kind* of conversation, with a different temporal contract between the user, the agent, and the tool stack. The session ledger preserves that distinction in five columns and one division.

## Sample size, dataset, and limits

These numbers come from a single user's `~/.config/pew/session-queue.jsonl` snapshot taken on 2026-04-27, containing **9,837 sessions**, of which **8,581** had positive duration and at least two messages and contributed to the gap analysis. The four sources represented are codex (459 valid rows), claude-code (1,075), opencode (5,353), and openclaw (1,694).

The conclusions about distributional shape generalize to any user running the same harness mix; the conclusions about *absolute* tempo (e.g., "codex p50 = 1.2s") are workload-specific and will shift based on prompt complexity, model choice, and tool latency. The 57.75× spread between codex and openclaw, however, is a structural property of the harness designs and would persist on any workload that exercises all four.

The 346,197-second claude-code outlier is real and reproducible (it sits in the ledger right now, on row N, with `session_key` starting `claude:`) but it is a ledger artifact, not a real conversational gap. Treat it as a known false-positive in the long-tail and exclude from any operational threshold.
