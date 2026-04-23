---
title: "Tool-calling parallelism: when 'just do them in parallel' is wrong"
date: 2026-04-23
tags: [agents, tool-calling, parallelism, performance, latency]
est_reading_time: 14 min
---

## TL;DR

Modern frontier models can emit multiple tool calls in a single assistant turn, and most agent runtimes will execute them concurrently if you let them. The performance win is real — three independent file reads in parallel is roughly one round-trip instead of three — but the failure modes that come with parallelism are different from the ones serial loops have. This post is about when parallel tool calls help, when they hurt, and how to design tool surfaces and agent loops so the model gets the speed-up without producing data races, double-writes, or context that no longer makes causal sense. The short version: parallelism is safe for **independent reads**, dangerous for **anything that writes**, and requires explicit design — not just an `enabled: true` flag — for the messy middle.

## The problem

The first time I turned on parallel tool calls in an agent loop, latency dropped meaningfully and I was happy for about a week. Then I noticed two new failure modes. First, an agent doing a multi-file refactor started occasionally producing diffs where two parallel `edit_file` calls had each read the file before either wrote, and one write silently clobbered the other. Second, an agent doing investigation work would issue six search calls in parallel, get the results back in nondeterministic order, and write a summary that referred to results by position ("the second result shows...") that no longer matched what was actually second. Neither was a bug in the model; both were consequences of letting the loop be parallel without thinking about what parallelism means for stateful tools and order-sensitive reasoning. The fix is not "turn parallelism off." It is to design for it.

## The setup

- An agent loop that supports a single assistant turn emitting multiple `tool_call` blocks (the standard shape across recent OpenAI, Anthropic, and Gemini APIs).
- A tool registry where each tool declares whether it is read-only, idempotent, or stateful.
- A scheduler in the loop that decides, given the model's emitted batch of tool calls, what to actually run concurrently.

The key insight: the model proposes a batch, and the scheduler decides. The model is not responsible for safety; the loop is.

## The latency math

Before talking about safety, it is worth being honest about how big the win is. For N independent tool calls, each with provider round-trip time `r` and execution time `e`, the serial cost is `N * (r + e)` and the parallel cost is roughly `r + max(e_i)` (one round-trip, then bounded by the slowest call). For typical values — `r` around 50–200ms for a local MCP server, `e` from a few ms (file read) to seconds (test run) — the win is dominated by `r` for small fast calls and by the slowest call for big ones.

A back-of-the-envelope from a real session of mine, doing a "find all callsites then read the relevant files" workflow:

- Serial: 11 tool calls × ~120ms median round-trip = ~1.3s, plus per-call execution.
- Parallel (in two batches because the second batch depended on the first): ~240ms in round-trips, plus per-call execution.

That is a ~5x reduction in round-trip overhead, which is the main thing parallel tool calls actually buy you. The model's "thinking time" is unchanged, and for any single tool that takes seconds to run, parallelism mostly hides other tools' latency behind it.

The takeaway: parallelism is most valuable when you have **many small independent calls**. For one call that takes a long time, parallelism does nothing useful.

## Classification: which tools are safe to parallelize

The simplest mental model: classify each tool as one of three kinds.

1. **Pure read.** No side effects. Result depends only on inputs and external state. Safe to call in any order, any number of times. Examples: `read_file`, `grep`, `list_dir`, most `search_*` tools.
2. **Idempotent write.** Has side effects, but calling it twice with the same arguments has the same effect as calling it once. Safe to parallelize against *different* arguments; unsafe to parallelize against the *same* arguments. Examples: `set_label`, `upsert_record`, `create_with_idempotency_key`.
3. **Stateful.** Effect depends on order or on prior state. Not safe to parallelize without explicit coordination. Examples: `edit_file`, `append_to_log`, `run_migration`, anything that reads-then-writes.

The tool registry should require every tool to declare its class. A reasonable shape:

```ts
type ToolDescriptor = {
  name: string
  schema: JsonSchema
  kind: "read" | "idempotent_write" | "stateful"
  // Optional: a function that returns a "resource key" derived from args.
  // Tools with the same key cannot run in parallel with each other.
  resourceKey?: (args: any) => string
}
```

The `resourceKey` is the concept that makes the messy middle tractable. Two `edit_file` calls against different paths can run in parallel; two against the same path cannot. The resource key encodes that:

```ts
const editFile: ToolDescriptor = {
  name: "edit_file",
  schema: editFileSchema,
  kind: "stateful",
  resourceKey: (args) => `file:${path.resolve(args.path)}`,
}
```

## The scheduler

Given a batch of N tool calls from the model, the scheduler builds a dependency graph and runs the maximum independent set in parallel. Pseudocode:

```ts
async function schedule(calls: ToolCall[], registry: ToolRegistry): Promise<ToolResult[]> {
  // 1. Group by resource key. Calls sharing a key run serially in
  //    submission order; calls with no key are unconstrained.
  const groups = new Map<string, ToolCall[]>()
  const free: ToolCall[] = []
  for (const c of calls) {
    const desc = registry.get(c.name)!
    const key = desc.resourceKey?.(c.args)
    if (!key) free.push(c)
    else (groups.get(key) ?? groups.set(key, []).get(key)!).push(c)
  }

  // 2. Run all "free" calls fully in parallel.
  const freeP = Promise.all(free.map(c => runOne(c)))

  // 3. For each resource group, run sequentially.
  const groupP = Array.from(groups.values()).map(async (group) => {
    const out: ToolResult[] = []
    for (const c of group) out.push(await runOne(c))
    return out
  })

  // 4. Reassemble in original order.
  const results = new Map<string, ToolResult>()
  for (const r of await freeP)             results.set(r.id, r)
  for (const grp of await Promise.all(groupP))
    for (const r of grp)                    results.set(r.id, r)
  return calls.map(c => results.get(c.id)!)
}
```

A few important details:

- **The model's order is preserved in the response.** Even if call #5 finishes before call #2, the response sent back to the model lists them in submission order. This avoids the "second result" problem from the intro.
- **Stateful calls within one resource serialize.** Two edits to the same file run in the order the model proposed them. The model's own intent about ordering is respected.
- **Stateful calls across different resources parallelize freely.** Editing two different files at once is fine.
- **A simple per-call concurrency cap** (not shown) prevents 50 parallel calls from saturating the host. I usually cap at 8.

## When the model proposes an unsafe batch

Sometimes the model emits a batch the scheduler cannot safely parallelize even with grouping — for example, two writes to the same record where the second depends on the first having succeeded. Three options:

1. **Reject the batch and ask the model to retry.** Send back a structured error explaining which calls conflict. Works, but burns a turn.
2. **Run them serially anyway.** This is what most loops do today. Safe but loses the win.
3. **Refactor the tool surface so the model cannot propose the unsafe batch.** This is the long-term answer. If the model keeps proposing two-step writes that should be atomic, the tool itself should expose an atomic version.

A real example. An agent kept proposing `read_record` followed in the same batch by `update_record` with the read value. That is a check-then-set with no atomicity. The fix was to add a `compare_and_set` tool that took both the expected value and the new value. The model picked it up immediately and the unsafe pattern stopped.

The general principle: **if you find yourself rejecting a batch shape often, fix the tools, not the model.**

## Tool design rules for a parallel world

A few rules that make tools play well with a parallel scheduler:

1. **Every write tool must be idempotent or have a `resourceKey`.** No exceptions. A write tool with no idempotency story will eventually be called twice and produce inconsistent state.
2. **Read tools should not have hidden side effects.** A "read" tool that also updates a "last accessed" timestamp is, for scheduling purposes, a write tool. Mark it as such or remove the side effect.
3. **Tool results should be self-describing.** The model receiving results out of model-proposed-order is fine if each result repeats enough of its input to be unambiguous. Prefer `{ "for_path": "x.ts", "lines": 42 }` to bare `42`.
4. **Long-running tools should expose progress, not block.** A 30-second `run_tests` call holds a slot in the parallel pool. If you can return a `task_id` immediately and have a separate `poll_task` tool, the slot frees and other work proceeds.
5. **Avoid tools that take other tools' outputs as opaque blobs.** If `tool_a`'s output goes straight into `tool_b`'s input via the model copy-pasting, you cannot reorder them and the model will sometimes get it wrong. Refactor into a single composite tool when this pattern recurs.

## Provider differences

The spec is mostly the same across providers, but the practical defaults differ:

- **OpenAI** parallel tool calls are on by default; you can disable per-request with `parallel_tool_calls: false`. A few specific scenarios (notably some structured-output modes) require disabling them.
- **Anthropic** emits parallel tool calls when the prompt encourages it; the system prompt phrasing matters more than a flag. The "tool_use" content blocks in a single assistant turn are the parallel batch.
- **Gemini** function calling supports parallel calls and has its own quirks around mixing function calls with other content blocks; check the latest docs.

The portable assumption: **a single assistant turn may contain N tool calls, and you must handle N >= 2.** Build the scheduler once; it works for all three.

## What this means for prompt design

Two prompt-level levers actually move the needle on how often the model uses parallelism:

1. **Tell it explicitly.** A line like "When you have multiple independent tool calls, emit them in the same assistant turn rather than one at a time" is worth putting in the system prompt. Models will under-use parallelism without the nudge.
2. **Show, don't just tell.** In the system prompt or examples, show a turn with two tool calls and a turn with one. The model will pattern-match on the shape.

Conversely, two anti-patterns:

- **Don't tell the model "always parallelize."** It will batch dependent calls. Say "when independent."
- **Don't list every tool's class in the system prompt.** The scheduler enforces safety; the model does not need to know which tools are stateful. It just needs to be encouraged to batch when it knows calls are independent.

## Measuring whether parallelism is helping

A simple instrumentation trick: log, per assistant turn, both `n_tool_calls` and `parallelizable_tool_calls` (the latter computed by the scheduler — how many calls actually ran without serializing on a resource key). The ratio is your "parallelism utilization."

Useful queries on this data:

- **Average parallel batch size.** If it stays at 1, the model is not batching even when it could. Tweak the prompt.
- **Serialized-on-resource rate.** If many batches contain calls that share a resource key, the model is proposing conflict-prone batches. Consider adding atomic tools.
- **Median tool round-trips per session.** This is the metric that should go down once parallelism is working. If it doesn't, parallelism is on but not being used.

```sql
-- Per day, average parallel batch size and effective parallel utilization.
SELECT
  date_trunc('day', ts) AS day,
  avg(n_tool_calls)                                            AS avg_batch,
  round(avg(parallelizable_tool_calls::DOUBLE / n_tool_calls), 2) AS util
FROM tool_batches
WHERE n_tool_calls > 0
GROUP BY 1
ORDER BY 1;
```

## What I would do differently

I shipped parallel tool execution before adding the resource-key concept and paid for it with two write-conflict bugs in the first month. Add resource keys before you turn on parallelism, not after. Specifically: classify every existing tool as read / idempotent_write / stateful, and add a `resourceKey` to every stateful one. The change is small, and it converts an entire class of bugs into "the scheduler serialized those, so they happened in order" — which is exactly what you want.

The other thing I would do earlier is build the parallelism-utilization dashboard. Without it, you have no idea whether the model is using the capability you turned on. With it, you can iterate on the system prompt with a real number to chase.

## Links

- OpenAI parallel function calling: https://platform.openai.com/docs/guides/function-calling
- Anthropic tool use docs: https://docs.claude.com/en/docs/build-with-claude/tool-use
- Gemini function calling: https://ai.google.dev/gemini-api/docs/function-calling
- "Patterns of Distributed Systems" (Martin Kleppmann reading list): https://martin.kleppmann.com/2020/12/02/dotnext-data-on-the-outside.html
- Idempotency keys at Stripe: https://stripe.com/blog/idempotency
