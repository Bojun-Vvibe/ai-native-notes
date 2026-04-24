---
title: "Reasoning content as a side channel: why DeepSeek-style `reasoning_content` keeps breaking your agent"
date: 2026-04-24
tags: [reasoning-models, tool-calls, schema-drift, deepseek, openai-compatible]
est_reading_time: 12 min
---

## The problem

A reasoning-class model emits two streams per assistant turn: the visible answer (`content`) and the chain-of-thought it used to get there (`reasoning_content`, on DeepSeek-style providers; `reasoning` summary blocks on OpenAI's Responses API; `thinking` on Anthropic's extended-thinking shape). Each provider names this differently, ships it through a different transport, and disagrees about whether you should round-trip it back on the next turn.

When you stitch a multi-vendor agent loop together — model A on this turn, model B on the next, both behind a unifying proxy — the side channel becomes a contract. Every part of that contract is currently load-bearing and currently under-specified. A spectacular cluster of bugs landed in this week's open-source agent stacks because of exactly this gap, and it is worth pulling apart what each one tells us about the underlying shape of the problem.

## The setup

Three real diffs from the past seven days, all converging on the same root cause:

- **litellm [#26395](https://github.com/BerriAI/litellm/pull/26395)** — DeepSeek V4 Pro `reasoning_content` was being stripped before being passed back to the provider on subsequent turns, breaking the model's ability to continue a chain-of-thought across the assistant→tool→assistant boundary.
- **opencode [#24146](https://github.com/sst/opencode/pull/24146)** (plus the closed-cluster siblings #24157 and #24163) — DeepSeek reasoning content was being mis-rendered in the TUI and then mis-replayed when a session was resumed, which caused the model to receive a malformed assistant-message history on resume.
- **crush [#2696](https://github.com/charmbracelet/crush/pull/2696)** — the streaming parser collapsed `reasoning_content` deltas into the wrong message field when the SSE stream interleaved `content` and `reasoning_content` chunks.

Three projects, three layers of the stack (proxy, frontend, transport parser), one root cause: the side channel exists, no one agrees on its semantics, and every layer that touches it has to invent a local convention.

The cluster is not an accident. It is what happens when a feature is added to one provider's API as a backward-compatible field — `reasoning_content` is a sibling of `content` inside the same delta, opt-in for callers that know what to do with it — and then propagates through the ecosystem under the assumption that "everyone will figure it out." Everyone does figure it out. They just figure out *different things*.

## The four contract questions nobody answered

Reasoning-content support is currently failing at four specific decision points. Let me name them, because the litellm/opencode/crush diffs above each correspond to a different one.

### Q1: Is `reasoning_content` part of conversation history?

You get a turn back from the model. It contains `content="The answer is 42."` and `reasoning_content="First I considered ... then I concluded ..."`. On the *next* turn, when you build the messages array to send to the model, do you include the `reasoning_content` from the prior turn?

There are three defensible answers:

- **Yes, always.** The model emitted it for a reason; replaying it gives the next turn full chain-of-thought continuity. This is what the DeepSeek docs imply for its own model.
- **No, never.** `reasoning_content` is for the *human reader* (or for downstream observability). Sending it back wastes tokens and biases the next turn toward continuing a possibly-wrong chain.
- **Yes, but only within a single tool-call sub-loop.** Round-trip it across `assistant → tool → assistant` so the model can pick up where it left off after the tool result, but drop it at user-message boundaries.

litellm #26395 implicitly chose option (1) — the bug was that the proxy was stripping it, treating it as option (2) when callers expected option (1). Notice that the bug existed for some non-trivial time before being noticed, which means a population of users were silently running with degraded reasoning continuity and not knowing.

### Q2: Where in the message structure does it live on the round-trip?

When you send `reasoning_content` *back* to the provider on the next turn, where does it go? On DeepSeek, the assistant message structure looks like:

```json
{
  "role": "assistant",
  "content": "...",
  "reasoning_content": "..."
}
```

So the obvious answer is: same field, sibling of `content`. But other providers use:

- A separate `reasoning` block array (Responses API)
- A separate `thinking` content type inside the content array (Anthropic extended thinking)
- A header in the SSE stream that has no equivalent in the JSON-shaped history

A unifying proxy like litellm has to translate between these on the way *out* (provider-native → unified) and on the way *back* (unified → provider-native). Either translation can drop, mis-key, or mis-shape the field. #26395 was a strip-on-way-back bug. There are mirror-image bugs lurking on the way-out path that haven't been caught yet, and you'd see them as "the assistant looks like it's not thinking, even though the model bill shows reasoning tokens consumed."

### Q3: How does it interleave with `content` in a stream?

Here's where crush #2696 lived. SSE streaming chunks arrive in order, but the model can emit `reasoning_content` deltas and `content` deltas interleaved within a single turn:

```
data: {"choices":[{"delta":{"reasoning_content":"First I "}}]}
data: {"choices":[{"delta":{"reasoning_content":"considered "}}]}
data: {"choices":[{"delta":{"content":"The "}}]}
data: {"choices":[{"delta":{"reasoning_content":"the alternatives. "}}]}
data: {"choices":[{"delta":{"content":"answer "}}]}
data: {"choices":[{"delta":{"content":"is 42."}}]}
```

A naive parser that uses a single accumulator will produce `"First I considered The the alternatives. answer is 42."` and `""` instead of `"First I considered the alternatives. "` and `"The answer is 42."`. The fix is two accumulators, keyed on which field each delta touched, not one. This is conceptually trivial and operationally easy to miss because the test fixtures everyone writes have all `content` *or* all `reasoning_content` in any given chunk, never both, and the model's actual behavior of interleaving only shows up under specific reasoning-effort settings on specific prompts.

### Q4: How does it survive a session save/resume?

opencode #24146 (and the cluster around it) was a save/resume bug. When a session is serialized to disk and re-loaded, what's the canonical on-disk form of an assistant message that has `reasoning_content`? If the on-disk schema has only `content`, you've silently dropped reasoning-continuity across sessions. If the on-disk schema has both but the loader populates only `content`, you've half-dropped it. If both are populated correctly but the *renderer* shows them concatenated as one blob in the TUI, the user sees garbage and reports it as a render bug — when the underlying memory is fine and the model will resume cleanly.

The opencode cluster of three PRs to fix this in a single week is the tell. It wasn't one bug, it was a family of bugs all traceable to "the on-disk shape and the in-memory shape and the on-wire shape were three different specifications, none of them written down."

## Why this is harder than it looks

A skeptic could read the above and conclude: write down the spec, treat `reasoning_content` symmetrically with `content`, and ship a test matrix that covers all four questions. Done.

This underestimates two things.

**First, the upstream specs are themselves moving.** DeepSeek's own API has changed the rules around when `reasoning_content` is required to be round-tripped at least once during the V3.x → V4 transition. The Responses API's `reasoning` block has versioned shape changes. Anthropic's `thinking` content type changed its `signature` requirement during the extended-thinking GA. Any unifying proxy that pinned to one shape last quarter is wrong this quarter, and any agent framework that treats `reasoning_content` as a stable contract is going to ship a regression every time one of the upstream providers ships a minor revision.

This is the same shape of problem as schema drift in tool definitions across model upgrades, but with a much smaller blast radius (only reasoning-class models are affected) and a much larger semantic risk (when `content` drifts you usually notice in tests; when `reasoning_content` drifts you only notice when the model's *answer quality* degrades, which is much harder to catch in CI).

**Second, the side channel is becoming load-bearing for non-reasoning concerns.** Operators are using `reasoning_content` as:

- A debugging surface (show me what the model was thinking before it produced this wrong answer)
- A guardrail input (parse the reasoning trace for indicators of jailbreak attempts or off-policy reasoning)
- A cache key component (two turns with the same `content` but different `reasoning_content` are arguably different turns for cache purposes)
- A billing line item (reasoning tokens are billed separately on most providers)
- A latency budget (reasoning token throughput is often a different number than content token throughput)

Each of these uses *requires* the side channel to round-trip faithfully. A bug in any of the four contract questions above is no longer just "the model's chain-of-thought continuity is degraded" — it's "the cache layer is wrong, the guardrail saw less than it thought, the bill doesn't reconcile, and the latency dashboard is lying." The blast radius scales with how many other systems have started leaning on the field.

## The shape of a correct implementation

Drawing from what survived in the merged versions of #26395, #24146, and #2696:

### At the parser

Two independent accumulators per turn, one for `content` and one for `reasoning_content`. Don't share a buffer. Don't infer one from the other. Don't drop either when the other is missing in a particular delta. Treat them as orthogonal streams that *happen* to share a single SSE transport.

### At the proxy

Round-trip `reasoning_content` by default for the *immediately next* assistant turn after a tool call, when the upstream provider is the same model family. Drop it across user-message boundaries unless the caller opts in via an explicit flag. Never silently strip — if the proxy decides to drop, log it at debug level with a counter, so operators can detect a misconfiguration before the model's answer quality regresses.

The asymmetry — keep across tool boundaries, drop across user boundaries — is a specific design choice. The argument for it: tool-call → assistant is an *intra-thought* transition, and dropping reasoning continuity there is exactly the bug. User → assistant is an *inter-thought* transition, and the user has presented new context that the prior chain-of-thought is mostly stale against. Shipping both as the default with an opt-out for either gives operators a clean lever.

### At the on-disk schema

Persist `reasoning_content` as a sibling of `content`, in a *versioned* assistant-message schema. When you load a session, check the schema version and apply migrations forward, not backward. A session saved by the v1 schema (no `reasoning_content` field) loaded by the v2 schema should default to `reasoning_content=null`, and any code that reads it must handle null without crashing.

The opencode cluster looked, from the outside, like the schema migration was the *easy* part and the renderer was the *hard* part — because the renderer had been written assuming `reasoning_content` was always present, then was patched assuming it was always absent, then was patched again to handle both. A renderer that treats `reasoning_content` as optional from day one would have prevented all three PRs.

### At the renderer

Show `reasoning_content` collapsed-by-default in the TUI, with an explicit toggle to expand. Never concatenate it inline with `content`. Make it visually distinct (different color, italics, sidebar — pick one and be consistent). The reason: a user who can't tell the difference between the model's reasoning and the model's answer will *quote the reasoning at the model* on the next turn, accidentally feeding the chain-of-thought back as user input, which is a separate failure mode the model is not trained to handle gracefully.

## A test matrix that would have caught all three bugs

The bugs above were caught one at a time, by users in production. The test matrix that would have caught them in CI:

| | `content` only | `reasoning_content` only | both, separate chunks | both, interleaved chunks |
|---|---|---|---|---|
| Single turn | basic | basic | round-trip | parser stress |
| Tool-call sub-loop | round-trip | round-trip | round-trip | round-trip |
| User → assistant boundary | persistence | persistence | persistence | persistence |
| Save / resume | schema | schema | schema | schema |

Sixteen cells. Every framework I've looked at this week had at most four of them, and the four they had were all in the top-left quadrant. The bugs lived in the bottom-right.

It is worth asking why the missing tests are missing. The honest answer: the model behavior that produces the relevant input (interleaved content+reasoning streaming, reasoning-content survival across save/resume, reasoning-content as part of a tool-call sub-loop) is hard to reproduce deterministically. You can mock the SSE stream byte-for-byte, but the team writing the parser doesn't usually own a captured fixture from the upstream provider in every reasoning-effort setting. The fix is for the upstream provider to publish *canonical fixtures* — actual SSE byte streams — that downstream parsers can replay. None of the major reasoning-model providers do this today.

## What this implies for an agent operator

If you operate an agent stack that uses any reasoning-class model:

1. Pin your unifying proxy version. Test reasoning-content round-trip after every upgrade. The litellm #26395 pattern will repeat.
2. Audit every parser in your pipeline for the dual-accumulator property. The crush #2696 pattern will repeat in any parser that doesn't have it.
3. Audit every save/resume path for the `reasoning_content`-as-optional property. The opencode #24146 cluster will repeat in any persistence layer that doesn't have it.
4. Treat `reasoning_content` as a billed, observable, cached side channel — not as a debug-only string. Anything that touches `content` for billing, observability, or caching has to also touch `reasoning_content` or your numbers will silently lie.

## Closing

The shape of this whole class of bug is not "DeepSeek's API is messy." DeepSeek's API is fine. The shape is "side channels become contracts, contracts go un-specified, and every layer of every consumer reinvents the contract differently." The same shape will apply to whatever the *next* sibling-field becomes — `safety_signals`, `tool_intent_summary`, `confidence_score`, whatever the providers ship next that travels alongside `content` and is opt-in for callers who know about it.

The fix is not to ship faster patches. The fix is to write down the contract, version it, fixture it, and treat any new sibling field as a P1 schema change that requires the four-question audit before it ships to production. Three open-source agent stacks ate that bill in a single week last week. The bill compounds: each of those PRs is a fix, and each fix is also a *reminder* that the contract is still un-written. Until somebody writes it down, every reasoning-model upgrade and every new sibling-field ships with a free copy of the same cluster.
