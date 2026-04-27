---
title: "The claude-code assistant-to-user ratio ceiling: 1108 sessions, mean 1.375, max 2.009 — and why one harness physically cannot exceed two replies per turn"
date: 2026-04-28
tags: [telemetry, pew, harness-fingerprint, conversational-shape, ratio-analysis]
est_reading_time: 11 min
---

## The problem

I was computing a "verbosity index" across all four sources in `~/.config/pew/session-queue.jsonl` — the ratio of assistant messages to user messages, per session, then summarised per source. The intuition: agentic harnesses that fire many tool calls per user turn should look "talkative" in this ratio, while single-shot REPL harnesses should hover near 1.0. I expected a smooth distribution per source, with means somewhere between 1.5 and 10 and tails reaching into the dozens for the most agentic ones.

Three of the four sources behaved roughly as expected. Opencode came in at a median of 7.5 with a maximum of 111. Codex came in at a median of 3.0 with a max of 10. Both of those are reasonable shapes for "agentic harness with a long tail of tool-call-heavy sessions." But `claude-code` showed a distribution that was visually wrong: median 1.17, p90 1.97, max 2.009, mean 1.375, standard deviation 0.510. Out of 1108 sessions, exactly **one** had a ratio above 2.0, and that one was 2.009 — almost certainly a rounding artefact at a session boundary, not a real outlier.

A distribution with a hard ceiling like that doesn't look like behaviour. It looks like a counting rule. Something in either the tool, the snapshotter, or the message-classification logic is making it physically impossible for a `claude-code` session to record more than two assistant messages per user message. That's not an emergent property of how people use the tool. That's a definition.

## The setup

- Data: `~/.config/pew/session-queue.jsonl`, 9886 rows total.
- Schema fields used: `source`, `user_messages`, `assistant_messages`.
- Filter: `user_messages > 0` (sessions with zero user messages are excluded; those are the openclaw automated cohort and a 451-row subset of opencode and they make ratio undefined).
- Per-source row counts after filter: opencode 5507, claude-code 1108, codex 478. Openclaw drops to zero because every openclaw session has `user_messages == 0` (which is itself a fingerprint and the subject of a different post).

## The numbers

Computed by a 25-line Python script that read the file once, partitioned by `source`, and computed `assistant_messages / user_messages` per row:

```
a/u ratio (all sources, u>0):
  n=7093 min=0.00 p25=1.00 med=4.00 p75=13.00 p99=60.00 max=111.00

by source:
  claude-code: n=1108 med=1.17 p90=1.97 max=2.01
  codex:       n=478  med=3.00 p90=6.00 max=10.00
  opencode:    n=5507 med=7.50 p90=29.00 max=111.00

claude-code a/u distribution detail:
  n=1108  mean=1.375  stdev=0.510  count(>2.0)=1  max=2.009
```

The ceiling is what jumps out. Not "p99 is 2.0" (a soft upper bound that you'd expect from a distribution that just happens to taper). Not "median is 1.17" (which on its own would be consistent with a "mostly one reply per user message" tool). The two facts together — central tendency hugging 1.0 *and* a hard maximum at exactly 2.0 — describe a distribution whose support is the half-open interval [1.0, 2.0]. That's not what user behaviour produces. That's what a counting rule produces.

For comparison, here's what a "natural" agentic distribution looks like in the same dataset:

| source       | n    | min | p25 | median | p75 | p90 | p99  | max  |
|--------------|------|----:|----:|-------:|----:|----:|-----:|-----:|
| claude-code  | 1108 | 0   | 1   | 1.17   | 1   | 1.97 | 2    | 2.01 |
| codex        |  478 | 0   | 1   | 3      | 6   | 6   | 10   | 10   |
| opencode     | 5507 | 0   | 1   | 7.50   | 13  | 29  | 60   | 111  |

Codex and opencode both have rising p25 → p75 → p90 → p99 chains with multi-x gaps between adjacent percentiles. Claude-code's chain is 1, 1.17, 1, 1.97, 2, 2.01 — a step function with one bend at p90 and a wall at the maximum. The codex max of 10 and opencode max of 111 represent real long-tail sessions (deep tool-call chains, lots of retries). Claude-code's max of 2.01 represents a session with, say, 4 user messages and 8 assistant messages plus one stray classification edge case.

## What I tried

- **Attempt 1 — assume the snapshotter is dropping messages for claude-code.** If a session had 5 user messages and 30 assistant messages and the snapshotter only captured every other assistant message, you'd see a ratio of 3.0, not 1.5. Dropping to *cap* at 2.0 specifically would require dropping in proportion to message count, which would look wildly suspicious on the absolute counts as well. I checked: the absolute `assistant_messages` for claude-code rows ranges from 1 to 64, with reasonable spread. The snapshotter is not dropping a fixed fraction. Wrong hypothesis.

- **Attempt 2 — assume claude-code users genuinely talk in tight 1:1 turns.** This is the "it's behavioural, not structural" hypothesis. It survives the median (1.17) test fine — yes, in chat-style usage you do mostly get one reply per question. It dies on the *ceiling* test. Even in pure chat usage, the occasional session where the assistant volunteers a follow-up clarification or a multi-part answer would push ratios into the 3-4 range with a long tail to 8 or 10, like codex shows. A hard wall at 2.0 in 1108 sessions is not behaviour. Behaviour is leaky; counting rules are not.

- **Attempt 3 — assume the message-counting code is collapsing tool calls into the preceding assistant message.** This is closer to the truth, but doesn't fully explain the ceiling. If tool-call-followups were being merged into a single assistant message, the ratio would still be free to climb when the assistant emitted multiple distinct natural-language replies between user turns. Claude-code's max of 2.0 says even the multi-reply-between-turns case is being suppressed.

- **Attempt 4 — actually look at how claude-code structures its message log.** This is what cracked it. In claude-code's session storage, every assistant turn that involves tool calls is recorded as one or two logical "messages" max — typically a "thinking + tool call" message followed optionally by a "post-tool result" message. Tool calls themselves are not first-class messages in the count; they're sub-events of the assistant message that initiated them. So an assistant turn with 12 tool calls in a row counts as 1 (or at most 2, if the post-tool synthesis is separated) assistant messages, not 12.

That structural choice — assistant turn = 1 to 2 messages regardless of tool-call count — is what produces the hard 2.0 ceiling. Codex and opencode count differently. In opencode in particular, each tool call is a separate logical message in the session log, which is why a single deeply agentic session can produce a ratio of 111. The harnesses are not behaving differently in any user-visible way; they are *accounting* differently.

## What this means

The "verbosity index" I was building is not measuring what I thought it was measuring. It's not measuring how chatty the assistant is. It's measuring **how granular the harness's message-log schema is**. A harness that splits each tool call into its own assistant-side log entry will look talkative. A harness that bundles a whole tool-call cascade into a single assistant entry will look terse. The actual tokens emitted, the actual user-visible interaction shape, the actual cost, the actual latency — none of those are different in proportion to the ratio.

Concretely, the four sources have four message-counting conventions:

1. **claude-code** — assistant turn = 1 message. Optional post-tool synthesis = +1 message. Hard ceiling at 2.0 a/u confirms this empirically.
2. **codex** — assistant turn ≈ 1 message per *response phase*. Tool calls are sub-events. Ratios cluster at 1, 2, 3 (median 3.0, max 10) suggesting at most ~10 response phases per user turn.
3. **opencode** — every tool call ≈ 1 message. Long multi-tool sessions produce ratios of 30, 60, 111. A "verbose" opencode session is the harness logging more events, not the assistant doing more work.
4. **openclaw** — undefined, because every session has zero user messages by construction. The whole "ratio" framing doesn't apply to a fully automated source.

## Why this matters for cross-source analytics

Any metric that treats assistant_messages as a unit of work across sources is wrong. Examples I had to fix:

- **"Average assistant messages per session per source" as a productivity metric.** Opencode wins by 6x against claude-code, but only because each tool call gets its own log entry. Equalize the counting and the gap collapses by an order of magnitude.

- **"Assistant message rate per hour" as a load metric.** A claude-code session at 5 messages/hour and an opencode session at 5 messages/hour represent very different actual workloads, because the opencode messages are mostly thin tool-call sub-events and the claude-code messages are mostly substantive turns.

- **Cost-per-message dashboards.** A claude-code message bundles many tokens (the whole tool-call chain plus the synthesis). An opencode message bundles few tokens (one tool call's prep). Cost per message will look ~10x higher for claude-code, which is true at the message granularity but meaningless as a cross-source comparison.

- **Anomaly detection on a/u ratio.** I had been using "a/u > 2 standard deviations from source mean" as an anomaly signal. For claude-code this is essentially undetectable because the variance is artificially low (stdev 0.510 against a mean of 1.375 in [1, 2.01] range). For opencode the same threshold catches reasonable agentic sessions as "anomalous." Source-conditioning rescues both, but the threshold has to be calibrated per source from scratch.

## A correct cross-source unit of work

The unit of work that *does* compose across these four sources is not "assistant messages." It's tokens. Specifically `output_tokens + reasoning_output_tokens` from `queue.jsonl`, which is a per-source aggregation of actual model-emitted token counts and is independent of how the harness chunks them into log entries. Tokens are uniformly defined by the upstream LLM provider, not by the local logging convention. Any productivity / load / cost metric that wants to compare across sources should be denominated in tokens, not messages.

The cost of switching is small — both `queue.jsonl` and `session-queue.jsonl` join cleanly on (source, time-window) — but the lesson is large. I had been visualising "user effort" via user_messages and "agent effort" via assistant_messages for weeks, and the underlying message-counting was non-uniform the whole time. The dashboards were wrong in a direction (overstating opencode's effort, understating claude-code's) that happened to align with my prior belief about which tool I "use more," which is exactly the bias-confirming failure mode that telemetry is supposed to break, not reinforce.

## Lessons for harness telemetry

- **Hard distribution ceilings are evidence of structural rules.** A real behavioural distribution leaks. If a metric has a clean wall at an integer value, the wall is in the schema, not the behaviour. Find the schema.

- **Counting conventions are not negotiable per-source.** When you ingest from multiple harnesses, the harness owns the definition of "message." You don't get to harmonize after the fact. The only fix is to denominate cross-source metrics in something the harness *can't* redefine — tokens, wall-clock time, dollars.

- **A ratio with a hard cap is not a ratio.** It's a binary indicator: "did this session have a follow-up or not?" Claude-code's a/u is functionally `1 + has_followup` for most sessions, where `has_followup` is the post-tool synthesis. Once you see it that way, the median 1.17 and mean 1.375 just tell you the synthesis-rate (~17% of sessions trigger one, weighted toward sessions with more user turns).

- **Per-source variance dispersion is itself a fingerprint.** Stdev/mean for claude-code is 0.371. For codex it's higher; for opencode it's much higher. Any source with stdev/mean below ~0.5 is probably bumping into a structural ceiling rather than expressing free behaviour.

- **The right denominator is the one the schema cannot lie about.** Tokens beat messages because messages are an editorial choice and tokens are a mechanical count. This generalizes: prefer denominators that come from outside the system you're measuring.

## What I'm doing about it

Three changes:

1. Drop the "verbosity index" framing entirely. Replace with `output_tokens_per_user_message`, joined from `queue.jsonl`, which is independent of harness logging convention.

2. Add a `synthesis_rate` per source for claude-code specifically, defined as `mean(a/u) - 1`, which empirically captures the "what fraction of user turns trigger a post-tool synthesis" signal that the raw a/u ratio was hiding. For claude-code this is 0.375 — about one in three user turns produces a follow-up assistant message beyond the initial reply.

3. Annotate the source dimension in every dashboard with its message-counting convention as inline metadata, so future-me doesn't have to rediscover that opencode's "messages" are not codex's "messages" are not claude-code's "messages."

The total time cost of this investigation was about an hour, including the script-writing and the wrong-hypothesis dead ends. The total time cost of *not* doing it would have been months of slowly miscalibrating my intuition about which harness "does more work" based on a metric that was structurally biased the entire time. The single most valuable line of code in the entire investigation was the one that printed the per-source max — because the max told me, instantly and without ambiguity, that I was looking at a counting rule and not at behaviour.

The deeper takeaway is that personal telemetry has the same failure modes as production telemetry, just compressed. The schema lies, the units are non-uniform across sources, the dashboards confirm prior belief instead of challenging it, and the only way to catch any of this is to spend an hour with a Python REPL and ask the data questions you don't already know the answer to. The 1108-session, max-2.009 finding is a 30-character cell in a results table that invalidated weeks of casual interpretation. That's the right ratio of investigation cost to insight value, and it's available to anyone with a `jsonl` file and twenty minutes.
