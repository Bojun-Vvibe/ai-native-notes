---
title: "Partial tool output streaming: the uncommitted-bytes problem"
date: 2026-04-25
tags: [agents, tools, streaming, reliability]
---

## The shape of the problem

When a tool call in an agent loop returns a long blob — a `grep -r` over a
monorepo, a `kubectl logs --tail=10000`, a 200KB JSON body from an HTTP
endpoint — the runtime has a choice it almost never thinks about
explicitly:

1. Buffer the entire tool stdout to completion, then hand a single
   `tool_result` block to the model.
2. Stream the bytes to the model as they arrive, in incremental chunks,
   so the model can begin reasoning before the tool has finished.

Almost every shipped agent runtime today picks (1). It's the safe
default: tool results are "atomic", the model sees a clean
request/response shape, and the schema (`{type: "tool_result",
tool_use_id, content}`) maps cleanly onto OpenAI / Anthropic /
Bedrock message conventions.

But (1) has a real cost, and as long-running tool calls become normal
the cost is becoming visible. This post is about that cost, the
half-baked attempts to fix it, and the genuinely hard semantic problem
that nobody's solved: **what does it mean to commit to bytes the tool
hasn't finished producing?**

## Why this matters now: data point

`pew-insights` `output-size --by source` (v0.4.41, shipped
2026-04-25 per `CHANGELOG.md`) shows that 28.8% of completions across
1367 sampled rows now exceed 8k tokens, with `opencode` source
hitting p95 = 282k completion tokens vs `vscode-ext` p95 = 14k —
a 20x output-size gap (history.jsonl tick `2026-04-24T21:18:53Z`).

That's the *model output* side. Tool *input* to the model is the
mirror image and is unmeasured by output-size, but `prompt-size`
(v0.4.39) tells the same story: 50.7% of 1049 sampled rows shipped
1M+ input tokens, mean 3.17M, max 55.7M for `claude-opus-4.7` (tick
`2026-04-24T20:40:37Z`). A meaningful fraction of that input is tool
output the previous turn buffered in full before the model saw any
of it.

The latency picture, when you have it, is uglier than the byte
picture. If a `grep` takes 12 seconds and produces 180KB, option (1)
forces the model to wait 12 seconds staring at nothing before it can
emit a single output token. Option (2) lets the model start
reasoning at t=200ms when the first chunk lands. For interactive
agents this is the difference between "thinking…" and a typing
cursor.

## Why nobody ships option (2) cleanly

The schema doesn't support it. Look at the actual tool-result
envelope every major model API uses:

```jsonc
{
  "role": "tool",
  "tool_call_id": "call_abc123",
  "content": "<the entire result, as one string>"
}
```

There is no `partial: true` flag. There is no sequence number. There
is no "this tool is still producing, here's chunk 3 of N, expect
more". The protocol assumes tool calls are atomic
request/response. Streaming exists *for the assistant's output* — SSE
chunks of `content_delta` and `tool_use_delta` — but not for *tool
return values flowing back into the model*.

Crush v0.62.0 ran into this with their long-running shell tool
(see `crush#2679`, the streaming SQLite checkpoint PR — same family
of "we can't represent partial state cleanly" pain). The patch they
shipped wasn't true streaming; it was *checkpointing*: snapshot the
partial output at fixed byte intervals so a crashed tool can resume.
The model still doesn't see anything until the tool finishes.

opencode's series of `tool-prompt-diet` PRs (`#24202`, `#24196`,
drip-23 in `oss-contributions/INDEX.md`) attacks the *opposite* end
of the same pipe: shrink the *committed* tool result before it gets
serialized into the next turn. That's a useful optimization but it's
fundamentally treating the tool result as a fixed artifact to
post-process, not as a stream to interleave with model reasoning.

## The four fake solutions

When teams notice the latency hit and try to fix it, they reach for
one of four patterns. None of them is real streaming. All of them
have failure modes worth naming.

### Fake solution 1: "Heartbeat with progress text"

The tool emits intermediate progress lines that get captured in the
final result:

```
[12:00:01] Scanning...
[12:00:03] Found 47 matches in 12 files
[12:00:07] Found 134 matches in 38 files
[12:00:12] Done. 247 matches in 67 files.
<actual results>
```

This makes humans tailing the log feel better. The model still gets
the entire blob at the end. Latency to first model token is
unchanged. Worse, you've now polluted the tool result with
timestamp noise that bloats prompt size for zero reasoning value —
the exact thing the opencode tool-prompt-diet PRs were trying to
delete.

### Fake solution 2: "Chunk into multiple tool calls"

The agent calls `grep_chunk(offset=0, size=4096)` repeatedly until
EOF. Each call is a clean atomic request/response. The model
"streams" by issuing a sequence of small calls.

This works, sort of, until you realize:

- The model now decides chunk size, which is a terrible decision to
  delegate to a probabilistic decoder. It picks 1024 sometimes and
  65536 other times based on vibes.
- Every chunk costs another full round-trip through the API,
  including prompt re-tokenization (the model re-reads its own
  history for every call). Net latency goes up for tools where the
  whole-buffer wait was tolerable.
- You've reinvented `read(2)` at the LLM-API layer, with worse
  semantics and 1000x higher per-syscall cost.

### Fake solution 3: "Stream into a side-channel file"

Tool writes to `/tmp/tool-output-abc.jsonl`. Returns a path to the
model. The model can issue follow-up `tail` calls if it wants more.

This is closer to honest. It admits the protocol can't carry partial
state and pushes the partial state into the filesystem (or object
store, or KV cache). The model now has a handle, not the data.

The cost: the model has to *know* to ask for the file, has to *know*
when to stop asking, and the file becomes an out-of-band artifact
that has to be GC'd, secured, and replayed on session resume. You've
just built a tiny database with the model as the query planner. (See
`W17-synthesis-31-session-resume-semantics-divergence.md` in
`oss-digest/digests/_weekly/` for what happens when sessions resume
without the side-channel state.)

### Fake solution 4: "Truncate aggressively, keep last N bytes"

The "tool-result-size-limits-and-truncation-policy" post from
2026-04-24 covers this in depth. Summary: pick a byte budget, slice
the tool output to fit, hope the model doesn't need the truncated
middle. Latency-to-first-token is unchanged (still wait for tool
completion to *know what to truncate*), but at least prompt size is
bounded.

## What real streaming would actually require

Honest tool-output streaming — chunks flowing into the model's
context as the tool produces them, model reasoning interleaved with
tool production — needs four things the current API surface does
not provide:

**1. A partial-result message type.**

The schema needs a `tool_result_chunk` distinct from
`tool_result`. Each chunk carries `tool_use_id`, `seq`, `content`,
and `done: bool`. The final chunk has `done: true` and may carry an
`exit_code` or `error`.

**2. Model-side chunked attention.**

The model needs to be able to attend to the partial chunks already
delivered while *more chunks are still being committed to its
context*. This is currently impossible with prefix-caching
architectures because the prefix is, by definition, frozen. You can
prefill chunks as they arrive (each chunk extends the prefix), but
you can't generate output tokens that interleave with chunks
arriving *during generation*. Generation locks the prefix.

This is the deep one. It's not a protocol problem; it's an inference
engine architecture problem. vLLM, TGI, llama.cpp all assume the
prompt is fixed at generation start. Mid-generation prompt
extension would require rewinding the KV cache or doing speculative
decoding against a moving target, neither of which is shipped today.

**3. Tool-side cancellation propagation.**

If the model decides at chunk 3 that it has enough information and
emits its answer, the tool needs to know to stop producing. Without
this you've made things worse: the tool keeps running, burning CPU
and possibly billing, while the model has already moved on. SIGTERM
propagation through the entire chain (model runtime → agent
orchestrator → tool subprocess → child processes the tool spawned)
is famously unreliable in practice.

**4. Replay determinism.**

If you re-run the session — for debugging, for evaluation, for the
"deterministic-replay-for-debugging-agents" pattern from 2026-04-25 —
you need the tool output to chunk *the same way* on replay. That
means recording chunk boundaries as part of the trace, not just the
concatenated final bytes. `history.jsonl`-style append-only logs
become append-only-with-chunk-markers, which is non-trivial because
chunk boundaries are I/O-timing-dependent (they reflect when bytes
happened to arrive at the agent process, not any property of the
tool itself).

## What you can actually ship today

Given the inference-engine constraint above, real streaming isn't on
the menu in 2026. But two things are:

**Eager-commit prefill.** Buffer the tool output in the agent
runtime, but send each chunk to the model API as a *new
prompt-cache-extending suffix* the moment it arrives. The model
can't reason on it until completion, but the *prompt cache* is warm
when generation starts, eliminating the prompt-tokenization round
trip. `pew-insights cache-hit-ratio` (v0.4.35, tick
`2026-04-24T19:06:59Z`) shows `claude-opus-4.7` already hitting
232% cache leverage from cache-friendly call patterns; eager
prefill compounds this. (Yes, ratios above 100% are real signal —
documented in `PEW_INTERNALS` — they reflect the same prefix being
reused across multiple turn-pairs in the same session.)

**Bounded-staleness tool views.** For tools whose output is
"queryable" (logs, search results, file contents), wrap them so the
agent gets a fast initial slice (first 4KB, latest 100 lines, top 20
matches) and a continuation handle. The model sees committed bytes
at t=200ms and decides whether the slice was sufficient. If not, it
calls `continue(handle)` — which is fake-solution-2 above, but
*scoped to one tool* and with sane chunk semantics the tool author
controls. This is the pattern `pew-insights` itself uses for its
subcommands: the histograms (`output-size`, `prompt-size`,
`time-of-day`) all return aggregates first; raw rows require an
explicit second call. It composes cleanly with prompt budgets and
doesn't pretend to be streaming.

## The semantic problem nobody's solved

Even if all four protocol/engine pieces shipped tomorrow, there's a
deeper question:

**What does it mean for a model to commit to bytes the tool hasn't
finished producing?**

If the model reads chunk 1 (the first 50 search results), starts
synthesizing an answer, and then chunk 2 arrives with a result that
contradicts chunk 1 — what's the model supposed to do? Roll back its
generated tokens? Emit a correction? Pretend chunk 2 doesn't exist?

The single-tool-call atomicity assumption isn't just an API
convenience. It's a *consistency model*. It says "the model reasons
over a snapshot, not a stream". Breaking that assumption opens the
same can of worms distributed databases have been chewing on for 40
years: read-your-writes, linearizability, causal consistency, the
whole catalog. Nobody has done the work of porting those concepts
to "what does the model see, when, and what guarantees does it have".

The closest published work I've seen is the `partial-output-checkpointer`
template (`ai-native-workflow`, catalog 54→56 per tick
`2026-04-24T18:29:07Z`) — and that's a tool-side artifact, not a
model-side semantic. We have the *plumbing* for partial state. We
don't have the *model contract* for what it means.

## The honest summary

Tool output is currently a synchronous request/response RPC bolted
onto an asynchronous, streaming model interface. The mismatch is
load-bearing: it's why the API surface looks the way it does, why
the latency hits are what they are, and why none of the four fake
solutions feels right.

Real streaming would require: (a) protocol surface for partial
results, (b) inference engines that allow mid-generation prefix
extension, (c) reliable cancellation propagation, (d) deterministic
chunk-boundary replay, and (e) a model-level consistency contract
that says what bytes the model is allowed to act on and when. We
have (d) in scattered form, partial (a) in nobody's published
protocol, and zero of (b), (c), or (e).

Until those land, the right move is to be very loud about which
fake solution you're using and what its failure mode is. Don't claim
"streaming tool output" when you mean "progress text in the result
blob". Don't claim "incremental tool calls" when you mean "the model
is paying for tokenization on every chunk". Name the constraint.
The next person to look at the system needs to know which corner of
the design space you compromised in.

The byte volumes (`prompt-size` 1M+ for half of all sampled rows,
`output-size` 282k p95 for `opencode`-sourced sessions) say this
problem is going to keep getting worse before any of the structural
fixes ship. Plan for the buffer-and-wait latency to dominate
interactive agent experience for at least another year. The honest
optimizations are eager prefill and bounded-staleness views. Call
them what they are.
