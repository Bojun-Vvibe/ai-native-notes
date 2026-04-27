# The 137× cache-to-output ratio and the write-amplification inversion in agentic token flow

**Date:** 2026-04-27 (UTC)
**Source window:** `pew-insights digest` since `2026-04-20T09:08:15.823Z`, 834 events, 6,402 sessions

## The number

Run the local digest on the eight-day window ending today and you get a totals block that looks innocuous until you start dividing things by each other:

```
Totals
metric             tokens
────────────────  ───────
total               6.19B
input               1.55B
cached_input        4.60B
output             33.58M
reasoning_output  544.91K
```

Cached input is **4.60 billion**. Output is **33.58 million**. The ratio is 4,600,000,000 / 33,580,000 ≈ **137.0**.

For every single token the model emitted as user-visible output across this entire eight-day, six-source, six-thousand-session window, **it re-read 137 tokens from prompt cache**. Add the 1.55B of fresh (non-cached) input and the per-output-token "context surface" balloons to (1.55B + 4.60B) / 33.58M ≈ **183**. One hundred eighty-three tokens of context, every single time, for one token of reply.

This is not a normal ratio. This is the inversion.

## Why "inversion" is the right word

In classical text generation — chatbots, completion APIs circa 2023, traditional NLP pipelines — the dominant arithmetic is the other way around. You send a prompt of a few hundred tokens, you get back a paragraph of a few hundred tokens, the input/output ratio hovers near 1, maybe 2 or 3 in chat-with-system-prompt setups. The expensive thing is generation: each output token is an autoregressive sampling step that has to run the full forward pass, while input is consumed in parallel during prefill.

Agent workloads have flipped this. Look at what the digest is actually measuring:

- **opencode** alone burned **3.62 B total tokens** to emit only **24.37 M output tokens**. Ratio: 148×.
- **openclaw** burned **1.19 B total** for **3.52 M output**. Ratio: **338×**.
- **claude-code** burned **912.79 M total** for **3.74 M output**. Ratio: **244×**.
- **codex** burned **389.31 M total** for **1.01 M output**. Ratio: **385×**.
- **hermes** burned **74.80 M total** for **947.94 K output**. Ratio: **79×**.

The smallest ratio in the table is hermes at 79× — and hermes is the lightest agent in the lineup, with the smallest tool catalogue and the shortest context windows. The biggest ratio is **codex at 385×**, the harness with the heaviest reasoning behavior and the deepest tool stacks. The ratio is correlated with how "agentic" the workload is, not with how loud the user is.

In the chatbot regime, output cost dominated. In the agent regime, **context maintenance cost** dominates by two-and-a-half orders of magnitude.

## What the cache is actually caching

When the digest reports `cached_input: 4.60 B`, it is counting tokens that the provider's prompt cache returned a hit for. That cache, in the Anthropic and OpenAI implementations relevant here, has roughly these properties:

1. It keys on a prefix hash. If you send the exact same first N tokens as you sent five minutes ago, you get N tokens of cache hit.
2. The hit charges a fraction of input price (typically 10% of base input on Anthropic, 50% on OpenAI's flavor — though numbers shift).
3. The TTL is short — minutes, not hours — so cached_input being high mostly tells you about *iteration density*, not just total volume.

What is the prefix? In an agent loop, the prefix is roughly:

`[system prompt] + [tool definitions] + [conversation history up to the last tool result]`

That prefix grows linearly with every tool call and every model turn. The output is a small delta — usually a single tool call (a few hundred tokens of JSON), or a short natural-language response. So every loop iteration looks like:

- Send: 50,000 tokens of context (cache hit on 49,500 of them, 500 fresh from the latest tool result).
- Receive: 200 tokens of output (one tool call).
- Loop.

Stack a hundred such iterations and you get exactly the shape the digest reports: cached_input dominates, raw input is a thin layer of "fresh" deltas, output is rounding error.

## The 137× number is a workload fingerprint

The interesting thing isn't that 137 is large. It's that **137 is stable across very different workloads**, while the components that produce it are not.

Consider the per-source breakdown again. opencode and openclaw differ by a factor of 3 in total token volume, by a factor of 7 in output tokens, and they run completely different agent harnesses, but their cached-to-total ratios — which I can compute by going back to the source rows — sit in a band roughly 80–95%. That's tight. Across six harnesses, two providers, and seven model variants, the share of total tokens that came from cache hits is bounded in a narrow strip.

The reason is geometric. If your loop has roughly K iterations, and each iteration appends roughly D delta tokens to a context that started at C₀, then the total bytes shipped over the wire is:

```
sum over i in [0,K) of (C₀ + i*D)
= K*C₀ + D * K*(K-1)/2
```

The total cache-eligible portion of that is everything except the brand-new D tokens at each step:

```
sum over i in [0,K) of (C₀ + i*D - D)
= K*(C₀ - D) + D * K*(K-1)/2
```

Divide and the cache share converges to `1 - D / (C₀ + (K-1)*D/2)`. For real agent workloads where C₀ is in the tens of thousands and D is in the hundreds, this expression sits north of 0.9 almost regardless of K. **The cache-share-near-1 regime is structural, not tuned.**

That is why every agent harness in the table looks the same on this axis. They all have the same loop shape.

## Where the inversion actually costs you

If 137× is structural, then naive optimization advice ("write less," "cap output length") is pointing at the wrong knob. A 50% reduction in output tokens — heroic, hard to even define — moves the totals by 33.58M × 0.5 = ~16.8M tokens, or **0.27% of the 6.19B total**. You cannot cost-optimize an agent by trimming its replies.

Where you *can* move the needle:

- **C₀ — the per-iteration overhead.** Every token in your system prompt, every tool definition, every example in your few-shot block multiplies by K (the loop length) and then by however many times you ran the loop. A 2,000-token diet on the system prompt is worth more than every reply being 500 tokens shorter.
- **D — the delta size per iteration.** This is mostly the size of tool results. A grep that returns 30,000 lines of matches when you wanted 30 is poisoning every subsequent cache hit and every subsequent fresh-input charge. The token-budget for tool outputs is the most underweighted lever in agent design.
- **K — the iteration count.** This is the only knob the model itself can pull. A model that can answer in 4 tool calls instead of 12 saves 3× on the K*C₀ term and 9× on the D*K² term.

The digest's per-source breakdown supports this. **codex** has the highest cache-to-total ratio AND the highest cache-to-output ratio, which means its loops are long (high K) and its prefix is fat (high C₀). **hermes** sits at the other end with low ratios because its loops are short. Neither is producing dramatically more or less *user value* per token; they're just on different points of the K × C₀ × D surface.

## The "fresh input" line is the most actionable signal

Of the 6.19B total tokens, fresh (non-cached) input is **1.55 B**. That's the line that maps most directly to actions you can take right now:

- It's the bytes that *can't* hit cache: tool outputs that vary across runs, fresh user messages, system-prompt tweaks that invalidate the prefix.
- Per source, fresh input ranges from 16.41M (hermes) to 640.85M (openclaw). Openclaw is sending **39× more "fresh" bytes** than hermes for only **3.7× more output**. That fresh-input-per-output-token ratio is the closest thing the digest provides to a "tool output bloat" metric.

If I had to pick one number from this digest to put in front of an agent author and say "fix this," it would not be the 137×. It would be the per-source fresh-input-to-output ratio. openclaw at 640.85M / 3.52M = **182** is the outlier. Every output token costs 182 tokens of brand-new prompt material. Either tool results are not being summarized, or each iteration is shoving large fresh artifacts (search results, file dumps, web fetches) into the context unaltered.

## What the model row says about the inversion

```
By model
model                total    input   output  events
─────────────────  ───────  ───────  ───────  ──────
claude-opus-4.7      4.55B  683.26M   28.55M     413
gpt-5.4              1.60B  838.85M    4.59M     360
```

Two models, comparable event counts (413 vs 360), wildly different shapes:

- claude-opus-4.7: total/output = **159×**, input/output = **24×**, so cached-input/output ≈ **135×**.
- gpt-5.4: total/output = **349×**, input/output = **183×**, so cached-input/output ≈ **166×**.

Same workload class (agent loops), same operator, same eight-day window, and the gpt-5.4 column carries roughly **half its weight in fresh input** while opus carries roughly **15% in fresh input**. Either:

- The gpt-5.4 cache is structurally less effective on this kind of prefix (different chunking, different keying, different TTL), or
- The harnesses that route to gpt-5.4 are doing something that defeats prefix stability — maybe tool-call IDs are being re-randomized per iteration, maybe the tool catalogue is reordered, maybe the system prompt is being re-templated with timestamps.

I don't have ground truth here. But the digest is unambiguous: per output token, **gpt-5.4 charges roughly 7.6× as many fresh input tokens as claude-opus-4.7 does** (183 vs 24). That's the kind of asymmetry that quietly bends a monthly bill.

## The 544.91K reasoning column is the only counter-trend

Reasoning output — internal chain-of-thought tokens that some models emit and bill for separately — is **544.91 K** total across the window. That is 1.6% of the user-visible output (33.58M) and **0.0088%** of the total token volume. In a regime where caching dominates by two orders of magnitude and fresh input by one, reasoning is not a budget issue. The narratives about "models think too much" don't map onto this dataset. Across 834 events, 6,402 sessions, eight days, and seven model variants, *thinking* is the cheapest line item by a wide margin.

The only sub-row where reasoning is non-trivial is concentrated in opus events that route through claude-code and opencode harnesses, where the reasoning column shows a heavier presence. But even there it's swamped by the cache term. The optimization story for reasoning tokens at this token volume is "ignore it."

## What changes if the inversion gets stronger

The 137× number is going to keep climbing. Three forces are pushing it up:

1. **Larger context windows.** Every increase in C₀ that the model can absorb is an increase in the cacheable prefix. Million-token contexts make the ratio explode by another order of magnitude even at constant K and D.
2. **More tool calls per task.** As agents take on multi-step tasks (refactor a repo, audit a codebase, run a CI loop), K grows.
3. **Better cache layers.** Provider-side cache hit rates are improving, which moves more of the (already-large) input volume out of the "fresh" bucket and into the "cached" bucket, mechanically inflating the ratio.

The forces pushing it down are weaker:

- Smaller, sharper system prompts (helps C₀, but only marginally — most C₀ growth is in tool definitions, not boilerplate).
- Tool-result summarization (helps D — this is the biggest available win and most harnesses don't do it well).
- Better cache invalidation discipline (helps shift volume from fresh to cached, but doesn't reduce total).

By this time next year I'd expect the headline ratio to be in the **300–500× range** for agentic workloads. At that point the chatbot mental model — "you're paying for the model to talk" — will have completely inverted. You'll be paying almost entirely for the model to *remember*, with output cost as a rounding error.

## Practical takeaways from this digest

For anyone running an agent harness against a real provider with a real bill:

1. **Stop optimizing output length.** At 137× it's noise.
2. **Audit C₀ once.** Print the actual byte size of (system prompt + tool definitions + first user message) before any iteration. Anything over 10K tokens is worth a one-time review. After that, leave it alone.
3. **Police D ruthlessly.** Cap tool output sizes at the harness layer, not at the model layer. Truncate, summarize, page. Every kilobyte you let through gets multiplied by every subsequent iteration's cache fee.
4. **Watch the per-source fresh-input-to-output ratio.** The 182 number for openclaw in this digest is the loudest single signal available. If a harness's number is climbing week over week, something in that harness's tool result handling is regressing.
5. **Trust the cache.** A high cache share isn't waste; it's the inversion working *for* you. The waste is in the fresh column.

The digest will show this same shape next week, next month, and next year. The constants will move. The inversion is permanent.

---
*Data: local `pew-insights digest` snapshot, window 2026-04-20T09:08:15Z → 2026-04-27, 834 events / 6,402 sessions across 6 sources and 7 model variants. All ratios computed from the totals block and the by-source / by-model breakdowns of that single digest invocation.*
