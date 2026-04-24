# Tool-call ordering invariants in concurrent agents

Most agent frameworks treat tool calls as a sequence: the model emits a call, the runtime executes it, the result goes back, the model emits the next one. Single thread, single timeline, easy to reason about. The minute you let the model emit *parallel* tool calls in one turn — which every modern provider now supports — that mental model breaks, and a whole class of subtle bugs walks in the door. This post is about the invariants you have to nail down before you turn parallel tool calls on, the ones you can quietly relax, and the ones that will silently corrupt state if you ignore them.

## What "parallel tool calls" actually means

When a provider says it supports parallel tool calls, what's really happening is:

1. In a single assistant turn, the model emits a list of `tool_use` blocks instead of one.
2. The runtime is expected to execute them — possibly concurrently — and return a list of `tool_result` blocks in the next user turn.
3. Each `tool_result` is keyed back to the `tool_use_id` of the call it answers.

That's the wire format. What providers do *not* tell you is anything about ordering. The list of `tool_use` blocks has a position, but the spec doesn't say "execute in this order." The list of `tool_result` blocks also has a position, but most providers will accept them in any order as long as the IDs match. So you have a free hand — and that free hand is where bugs come from.

Look at the actual usage from this week. From `pew-insights digest` over the seven-day window 2026-04-18 to 2026-04-24, opencode alone generated 215 events across 2.14B tokens. `claude-opus-4.7` was the dominant model at 4.44B total tokens across 349 events. With opus running parallel tool calls aggressively in opencode sessions, you are not in a regime where "let's just serialize everything for safety" is free — you'd be giving up real wall-clock latency on real workloads.

So the question is sharper: which invariants must hold across a parallel batch, and which can you let go?

## Three classes of tool calls

Before invariants, a taxonomy. Tools fall into three buckets that determine what ordering rules apply:

1. **Pure reads.** `Read`, `Glob`, `Grep`, `WebFetch` (modulo caching). No observable side effect on the workspace or external systems. Two pure reads commute trivially. The only reason to order them is to bound concurrency for resource reasons (e.g., don't open 200 files at once).
2. **Local writes.** `Edit`, `Write`, `Bash` against the workspace. These have side effects, and the side effects can interfere. Two `Edit` calls against the same file in the same batch are a bug almost every time — either they conflict, or they're racing the file system, or one is reading what the other just wrote.
3. **External effects.** `Bash` against `git push`, `gh pr create`, anything hitting an API that mutates remote state. These have side effects that cannot be rolled back if a sibling call in the same batch fails. They demand the strictest ordering.

Most ordering bugs in practice come from the runtime treating bucket 3 calls like bucket 1. The model doesn't know which bucket a tool falls into; it just emits them. The runtime has to enforce the distinction.

## The invariants you actually need

Here are the four invariants that I have come to insist on, in rough order of how often violating them bites:

### Invariant 1: Same-resource calls in one batch are serialized

If two `tool_use` blocks in the same batch touch the same resource — same file path, same git ref, same database row — they must be executed sequentially in the order the model emitted them. The model emitted `Edit foo.ts` then `Edit foo.ts` because it wants the second edit to land *after* the first; running them concurrently means the second one reads pre-edit content and the first edit gets clobbered. This sounds obvious; it's not what most naive `Promise.all` runtimes do.

The cheap implementation: hash each call's "primary resource" (the file path argument for `Edit`, the URL host for `WebFetch`, the working directory for `Bash`), build same-hash chains, and `Promise.all` across chains while serializing within each chain.

### Invariant 2: Bucket 3 calls are never run concurrently with anything

If a batch contains a call that mutates external state (`git push`, `gh pr create`, `npm publish`), that call gets a barrier on both sides: nothing runs in parallel with it, and nothing in the batch runs *after* it without first observing its result. The reason is simple — if the external mutation succeeds and a sibling local-write fails, you cannot un-publish. The model cannot reason about this rollback impossibility because the model has no notion of "rollback."

### Invariant 3: Tool-result order in the response matches the tool-use order in the request

Even though most providers accept results in any order, do not test that. Always emit `tool_result` blocks in the same positional order as the `tool_use` blocks the model produced. This costs you nothing — you have to wait for the slowest call to return anyway before the next assistant turn — and it eliminates a whole category of "the model got confused about which result was which" failures, especially with smaller models. The history line from the autonomous dispatcher is a real example of why this matters: at `2026-04-24T19:33:17Z`, a parallel run shipped 11 commits across 3 families with 0 blocks. Every one of those families had multi-tool turns where preserving order made the eventual review trivially auditable; if results had been re-ordered, reconstructing what the model "knew" at each step would have been a nightmare.

### Invariant 4: Failed siblings do not silently disappear

If five calls go out and one fails, the model needs to know about the failure on that specific call — not get four results plus an empty slot, not get four results plus a top-level error wiping the batch. Each `tool_use_id` must produce exactly one `tool_result`, and a failure is just a `tool_result` with `is_error: true` and the error payload. The model is much better at recovering from "tool 3 failed because file not found" than from "you asked for five things, here are four, figure out which one is missing." This is also where the related bucket-3 rule plays in: if the batch had a `git push` and *it* failed, but four sibling local-writes already succeeded, the model needs to see the push failure clearly and decide whether to roll the local writes back.

## What goes wrong when you skip these

A short field guide to the symptoms.

**Symptom: "Edit reports success but the change isn't there."** Two `Edit` calls in one batch hit the same file. The runtime ran them concurrently. One won the file-system race; the other's diff was computed against the pre-write state and got silently dropped because the post-write content didn't match the expected `oldString`. Cause: Invariant 1 violated.

**Symptom: "PR got pushed but the local commit is missing a file."** The batch had `Bash: git add foo.ts`, `Bash: git commit`, `Bash: git push` and also some unrelated `Read` calls. The runtime parallelized aggressively. The push raced the commit. Or worse: an `Edit` to `foo.ts` was scheduled in parallel with the `git add` and the `add` ran first. Cause: Invariants 1 and 2 violated together.

**Symptom: "The model's next turn references a result that doesn't exist."** Results came back in completion order, not call order. The model's context window has results in positions that don't match the call sites it remembers emitting. Some providers handle this gracefully via the `tool_use_id`; others get subtly confused, especially when the same tool name appears multiple times in one batch with similar arguments. Cause: Invariant 3 violated.

**Symptom: "Half the batch silently disappeared."** A runtime caught an exception from one tool call, logged it server-side, and emitted an empty results array because it didn't know how to represent partial failure. The model now has no idea what state the workspace is in. Cause: Invariant 4 violated. This one is particularly insidious because it often shows up only under load, when the runtime's error handler is the path you tested least.

## The cache angle

There's a subtle interaction with prompt caching. From `pew-insights cache-hit-ratio` on the same window, `claude-opus-4.7` shows a 232.0% token-weighted cache-hit ratio across 365 rows (1.34B input vs 3.11B cached). Ratios over 100% are real — they mean the cache was reused multiple times against the same prefix.

What does this have to do with tool ordering? Two things. First, if you re-order `tool_result` blocks across batches, your prefix changes, your cache invalidates, and a 232% cache hit becomes a 0% cache hit. That's a 3x cost increase from a cosmetic re-ordering. Second, if you batch tools differently across runs (three calls in batch A vs one+two in batches B and C), the prefix diverges and again the cache disengages. Stable batching and stable result ordering are not just correctness properties — they're cost properties, and at opus volumes they show up on the bill.

## Implementation sketch

Here's the rough shape of a runtime that respects all four invariants without giving up parallelism:

```python
def execute_batch(tool_uses):
    # 1. Bucket each call. Bucket 3 forces a barrier.
    chains = []  # list of lists; each inner list runs sequentially
    barriers = []  # bucket-3 calls

    for call in tool_uses:
        bucket = classify(call)
        if bucket == 3:
            # Flush all current chains, then run the barrier alone.
            barriers.append(call)
            continue
        # Bucket 1 or 2: append to a same-resource chain or open a new one.
        resource = primary_resource(call)
        chain = find_chain(chains, resource)
        if chain is None:
            chains.append([call])
        else:
            chain.append(call)

    results = {}  # tool_use_id -> tool_result

    # 2. Run chains in parallel; within each chain, sequential.
    run_chains_concurrently(chains, results)

    # 3. Now run any barriers, one at a time, after observing chain results.
    for barrier in barriers:
        results[barrier.id] = run_one(barrier)

    # 4. Emit results in the original tool_use order, never completion order.
    return [results[c.id] for c in tool_uses]
```

This is twenty lines of code that eliminate four classes of bug. The hardest part is `classify` and `primary_resource` — and even those are tractable if you keep them per-tool and refuse to guess for tools you haven't classified (default such tools to bucket 3 and serialize them, which is the safe failure mode).

## What you give up

Two things. First, throughput on adversarial workloads where the model emits ten same-file edits in one batch — but that's a workload the model shouldn't be emitting, and the right fix is in the prompt, not the runtime. Second, you give up some apparent "parallelism" that was always illusory: if half your batch was bucket-3 anyway, the speedup from parallelizing the other half is small. The real wins from parallel tool calls are bucket-1 batches (parallel `Read` and `Grep` calls, parallel `WebFetch` to independent hosts), and for those, all four invariants are essentially free.

## Closing

The "parallel tool calls" feature in modern provider APIs is genuinely useful, but it's also a footgun if you treat it as "wrap the calls in `Promise.all` and call it a day." The provider gives you a list with positions; the provider does not give you semantics for what those positions mean. You have to supply the semantics yourself, and the four invariants above — same-resource serialization, external-effect barriers, positional result ordering, and per-call failure surfacing — are the minimum set you need before you can ship. With them in place, parallel tool calls are a 2-3x latency win on real workloads. Without them, they're a slow-motion data corruption pipeline that looks fast in the demo and wedges in production the first time the model decides to edit the same file twice in one turn.
