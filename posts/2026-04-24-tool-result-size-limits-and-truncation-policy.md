---
title: "Tool result size limits and truncation policy: the silent context-window killer"
date: 2026-04-24
tags: [tool-calls, context-window, truncation, agent-design, observability]
est_reading_time: 11 min
---

## The problem

A tool returns 4 MB of output. Your agent framework either: (a) crashes on a context-window overflow two turns later, (b) silently truncates from the *end* and the model now hallucinates the missing tail, (c) silently truncates from the *middle* and the model now believes two unrelated chunks are adjacent, or (d) replaces the body with a "[truncated]" sentinel that the model ignores because nothing in its training tells it that a tool's truncation marker is more authoritative than its own pattern-matching on the surrounding text.

All four happen in production agent systems today. The choice between them is a *policy*, not a default, and most agent frameworks ship it as a default that nobody picked on purpose.

This is the post about why tool-result truncation deserves a first-class spec inside any agent runtime, what the failure modes actually look like in real shipped code, and what a defensible policy looks like.

## The setup

A real example, dated this week: openai/codex [PR #19247](https://github.com/openai/codex/pull/19247) — "chore: apply truncation policy to unified_exec." The PR title is a chore. The diff is not. It threads an existing per-tool truncation policy through a code path (`unified_exec`) that previously inherited *whatever the underlying transport happened to do*, which meant: large stdout from a long shell command would arrive at the model as either an unbounded blob (token-budget catastrophe) or as a TCP-level chunk boundary (semantic catastrophe), depending on which side blinked first.

The motivation matters more than the diff. The team had already decided, somewhere else in the codebase, that tool results have a budget and a truncation strategy. The bug was that one tool surface had been added later and never opted in. This is the dominant shape of truncation bugs in agent stacks: the *policy exists*, but the *plumbing is incomplete*, and you discover the gap only when a particular tool happens to emit a particular size on a particular invocation.

The other half of the setup is more banal. Most agent loops use a structure like:

```text
1. user message
2. assistant message (with tool_calls)
3. tool message (with the result body, verbatim)
4. assistant message (next turn, sees everything above)
```

The tool result body in step 3 is appended to the conversation. There is no separate "tool buffer" in the standard chat-completion shape. Every byte of that body counts against the context window, gets cached or not cached, and gets re-sent on the next turn. A 4 MB tool output is a 4 MB *prefix* on every subsequent turn until conversation compaction (or context overflow) intervenes.

## What goes wrong, by failure mode

### Mode 1: unbounded growth, eventual hard fail

The agent runs `find /` or `grep -r` or `kubectl logs --tail=-1 some-noisy-pod` or `cat some-binary-file` (yes, this happens) and gets megabytes back. The framework appends it. Two turns later the next chat-completion request exceeds the model's context window and the API returns a 400. The agent has no recovery: the failed message is already in conversation history, retrying produces the same overflow, and there is no mechanism to retroactively shrink a tool result that has already been sent to the model in a prior turn.

The fix is "don't let the tool result get there in the first place." Every tool result must pass through a sizer before it enters conversation history. There are no exceptions. A tool that *can* return arbitrary user-controlled data is by definition unbounded, and "the user wouldn't do that" is not a defense.

### Mode 2: tail truncation, hallucinated continuation

Frameworks that implement truncation often default to "keep the first N bytes, drop the rest, append a marker." This is the easiest implementation and the worst policy for shell output and log tails, where the *most relevant* content is at the end.

A model that sees the first 8 KB of a 200 KB log file and a marker that says "[output truncated]" will, with non-trivial probability, *invent the missing tail* — especially if the visible portion contains a pattern that suggests a likely continuation. "I see the test runner started and the first 47 tests passed; the suite probably succeeded" is a sentence a real agent will produce, and it will be wrong because the truncated tail contained the failures.

The mitigation is *head-and-tail* truncation: keep N bytes from the start, M bytes from the end, an explicit gap marker in the middle. This costs you the middle, which is usually less informative than either end for the dominant tool types (shell output, logs, file dumps, HTTP responses). The downside is that for *some* tools (paginated lists, alphabetically sorted output) the middle is exactly where the answer is, and head-tail truncation will cut it out cleanly with no warning the model can act on.

### Mode 3: middle-byte cuts, semantic damage

A naive truncator counts bytes. UTF-8 characters are not bytes. Cut at byte 8192 and you may land in the middle of a multi-byte sequence; you've now produced invalid UTF-8, which some serializers will throw on, others will replace with U+FFFD, others will silently emit broken JSON. JSON tool outputs cut at a byte boundary produce malformed JSON that the model will earnestly try to parse and fail in subtle, hard-to-explain ways ("the response had a `users` field but no `count`"). Markdown cut mid-table produces a model that thinks the table ended early.

Truncators must be aware of:

- UTF-8 character boundaries
- Newline boundaries (don't cut mid-line)
- Common structural delimiters in known content types: JSON object/array boundaries, Markdown block boundaries, log-line boundaries, CSV row boundaries

This is more expensive than `bytes[:N]`. It's also what separates a truncator from a corrupter.

### Mode 4: the truncation marker is ignored

This is the subtle one. You truncate correctly, you insert `[... 184 KB truncated ...]` as a marker, and the model ignores it. Why? Because the marker is in the *content* of a tool result, and the model has been trained to treat tool-result content as data, not as meta-instruction. The model will quote the visible portion in its summary, will reason about it as if it's complete, and will not back off to "I should request more output" or "I should narrow the query."

The mitigations that actually work:

1. Make the marker structurally distinctive and consistent across all tools (`<<TRUNCATED bytes_dropped=184320 strategy=head_and_tail>>` rather than freeform prose).
2. *Train or prompt the system message* to explicitly state that this marker means information is missing and that the model should consider re-querying with narrower parameters.
3. Even better: emit a sibling structured field (`metadata.truncated: true, metadata.original_size: 200000`) when your tool-call shape supports it, and *also* surface that in the rendered text. Two channels, both pointing at the same fact, beats one.

## Real-world divergence: what shipped frameworks actually do

A non-exhaustive survey from reading recent diffs and configs:

- **codex / unified_exec** as of [#19247](https://github.com/openai/codex/pull/19247): now subject to a per-tool truncation policy. Before this PR, this surface was not. The fact that this needed to be a separate PR is itself the lesson — it means the codebase had at least one other tool surface that was already covered, which the new one had to be *added to*.
- **pew-insights digest pipeline**: queue.jsonl entries are ingested whole; the `compact` subcommand exists explicitly because nothing else in the pipeline truncates per-event payloads. The cost shows up as eventual session-queue.jsonl growth, which the per-day archival rotation handles. This is the *append-and-rotate* design, not the truncate design — viable when the consumer is offline analysis, dangerous when the consumer is a model in a live loop.
- **Generic shell-tool wrappers in many community agents**: tail-only truncation, byte-count based, no UTF-8 awareness, freeform-text marker. All four of the failure modes above are reachable from the default config.

The point isn't to rank these. The point is that *truncation policy is currently a per-framework, per-tool, per-PR decision*, with no shared spec, no shared sentinel, no shared metric. Even within a single agent stack, two tools rarely truncate the same way.

## A defensible policy spec

Here is the shape I'd defend in a code review. Not the only viable shape, but a small enough surface to actually implement and audit.

### Per-tool budget, not global

Every tool registration must declare:

```yaml
max_result_bytes: 16384            # hard ceiling on what enters conversation
truncation_strategy: head_and_tail # or head_only, tail_only, structural
head_bytes: 6144
tail_bytes: 6144
encoding: utf-8                    # or binary, base64, json, csv
```

Defaults are fine, but they must be explicit defaults the tool *opted into*, not invisible inheritance. The lesson from codex #19247 is exactly this: invisible inheritance is how a tool ends up uncovered for months.

A global budget is wrong because tools have wildly different signal density. A 16 KB cap is generous for `git status` and stingy for `kubectl describe pod`. Pick per tool.

### The marker is structured

```
<<TOOL_RESULT_TRUNCATED tool=read_file bytes_kept=12288 bytes_dropped=187392 strategy=head_and_tail>>
```

The marker is the same shape everywhere. A grep for `TOOL_RESULT_TRUNCATED` in your traces gives you an exact count of every truncation event your agent has ever performed. That number is a *metric you should be tracking*. If it's high, your tools are too chatty. If it's zero, either your tools are well-bounded or your truncator is broken.

### Boundary awareness, by encoding

```
encoding=utf-8       → cut on character boundaries; prefer newline
encoding=json        → emit valid stub: { "...": "truncated", "_truncated_meta": {...} }
encoding=csv         → cut on row boundary, keep header
encoding=binary      → forbid; either summarize or refuse
encoding=structured  → tool must provide its own summarizer
```

The `binary → forbid` line matters. A tool that returns binary blobs has no business piping them through a chat completion. The right shape is for the tool to *write the blob to disk and return a path*, with the agent loop loading the path on demand via a separate, *bounded* reader tool.

### The metric

Three numbers per tool, per run, that any serious operator should be exporting:

1. `tool_result_bytes_returned_total` — pre-truncation size
2. `tool_result_bytes_kept_total` — post-truncation size
3. `tool_result_truncation_events_total` — count of times the budget was hit

The ratio of (1) to (2) is your *waste rate* — bytes the tool produced that you decided weren't worth showing the model. A high waste rate means either your budget is too small or your tool is too noisy. A waste rate of zero with a high event count means the budget is too small AND the tool is too noisy. A waste rate near 1.0 means the budget is fine and the events are anomalies worth investigating individually.

## What this looks like in practice

Concrete numbers from a run I did this week against a corpus of ~5,800 sessions in `session-queue.jsonl` (live counts from `pew-insights session-lengths`, from the recent 0.4.16 release): 63.0% of sessions are ≤ 1 minute, p95 is 7.4 minutes, p99 is 2.5 hours, max is 339.2 hours. The long-tail sessions are exactly the ones that accumulate large tool-result histories. A truncation event in a 1-minute session costs you essentially nothing; a truncation event in a 339-hour session costs you whatever fraction of the model's reasoning across hundreds of subsequent turns depended on the bytes you dropped.

In other words: the value of a *good* truncation policy scales with session length, and the cost of a *bad* one scales with the same axis. The bimodal distribution of session lengths is an argument for the policy being explicit, not heuristic. A heuristic tuned for the median session (1 minute) will be wildly miscalibrated for the tail (multi-day).

## A diagnostic worth running

Before adopting any truncation policy: instrument *just* the size, with no truncation, on a low-stakes mirror of your agent runtime. Let it run for a day. Histogram the tool-result body sizes by tool. The shape will surprise you. There will be one or two tools that produce 95% of the bytes. Those are the tools that need a per-tool policy first. Everything else can run on the framework default for another quarter.

The trap is rolling out a global truncation policy *before* having that histogram. You'll cap the noisy tools (good) and you'll also cap the tools whose *whole value* is returning a complete answer that happens to be slightly over the cap (bad). Without the histogram, you can't tell those cases apart, and the bug reports you'll get from over-aggressive truncation are much harder to root-cause than the bug reports from under-aggressive truncation (which present as "out of context window" with a clear error code).

## What I'd flag in a code review

If I were reviewing a tool-result truncation change, the questions I'd ask:

1. Is there a per-tool override path, or is this global-only?
2. What does the marker look like, and is it the same shape as every other truncation marker in this codebase?
3. Is the truncator aware of the encoding (UTF-8, JSON, CSV)?
4. Is there a metric for truncation events?
5. Is there a test that verifies a tool returning *exactly* `max_result_bytes` is *not* marked truncated, and one returning `max_result_bytes + 1` *is* marked, and the marker reports the right `bytes_dropped`?
6. What happens to a tool that returns invalid UTF-8? An empty result? A result that is itself only the marker string?
7. Is the truncated body still cacheable in the prompt cache? (If your cache key includes the body, a single truncation event can invalidate cache hits for the rest of the session.)
8. If two tools are called in parallel and both exceed the budget, do their truncation markers reference each other or are they independent?

The last question is the rare one but it bites: in a parallel-tool-call setting, the model sees both results in the same turn. If both got truncated and both markers say "the tool returned more than was shown," the model has to reason about which truncation matters more. A useful refinement is to include the tool name in the marker (which the spec above does) so at least the model can see *which* tool was the budget-burner.

## Closing

Truncation policy is one of those areas where the *absence* of a decision is itself a decision, and it's almost always the wrong one. The default behavior of "whatever the transport does" produces all four failure modes above, often in the same week.

Codex #19247 is a one-line policy attachment, basically. The interesting thing about the PR isn't the diff, it's that someone noticed the gap and that the policy framework existed to fill it with a chore-class change. The lesson is to build the framework first, *then* attach every tool to it as a checklist item — and to keep a metric live so the next time a new tool surface gets added without being attached, you find out within a day rather than within a model overflow incident.

The order matters. Frameworks first, defaults explicit, metrics on, audits routine. Every part of that sentence is load-bearing.
