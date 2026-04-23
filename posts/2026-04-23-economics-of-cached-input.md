---
title: "The economics of cached input: where the savings actually come from"
date: 2026-04-23
tags: [llm, cost, prompt-cache, economics, infrastructure]
est_reading_time: 15 min
---

## TL;DR

Cached input tokens are priced at a discount because the provider is selling you something cheaper to produce, not because they are running a loyalty program. Understanding the actual cost structure on the provider side, and how it maps to the discount they offer you, lets you predict which workloads will benefit from caching and which will not. The short version: caching pays off when the per-call setup cost (loading model weights, processing the prefix's KV cache, scheduling the call) is large relative to the per-call marginal cost, which is true for long-prefix short-output calls and untrue for short-prefix long-output calls. This post walks through the cost model, the breakeven math, and four workloads I have measured against the math, with surprises in two of them.

The framing: cached-input pricing is a window into how the provider's serving stack actually works. If you read the discount as a sales tactic, you will misuse it. If you read it as a cost-pass-through, you will use it correctly.

## Section 1: where the cache discount comes from on the provider side

Inference for a transformer LLM has two clearly distinct phases: prefill and decode. Prefill processes the input tokens in parallel, producing a key-value cache (the "KV cache") that captures the attention state at every position in the input. Decode then generates output tokens one at a time, each step looking back at the KV cache to compute attention.

Prefill is compute-bound: the GPU is doing matrix multiplications over thousands of tokens at once and is at or near its peak floating-point throughput. Decode is memory-bandwidth bound: each step reads the entire KV cache from GPU memory to compute one new token, so total throughput is limited by how fast you can stream the cache, not by how fast you can multiply.

Both phases cost money in different ways:

- Prefill cost scales with the square of the input length (because attention is O(n^2) in sequence length on a naive implementation, or O(n^2/sparsity) with various optimizations).
- Decode cost scales linearly with the output length, with a per-token constant proportional to the KV cache size (which itself is proportional to input length).

When the provider offers you a "cached input" discount, what they are really doing is amortizing the prefill cost across multiple calls. They computed your prefix's KV cache once (the cache create), stored it for some TTL, and on subsequent calls they skip the prefill phase entirely for the cached portion. Their marginal cost for the cached prefix is now near zero (just the GPU memory for the stored KV cache), which is why they can charge you 10 percent or 50 percent of the uncached price.

This explains the discount asymmetry between providers. Anthropic charges 10 percent of fresh input for cache reads; OpenAI charges 50 percent. The plausible reading is that Anthropic's cache infrastructure has lower amortized storage cost (perhaps shorter TTL, perhaps better dedup, perhaps better hardware utilization) and they pass the savings along. The exact internals are speculation; the price ratio is the public signal.

## Section 2: the cost model from your side

To predict whether caching helps a workload, you need a per-call cost expression. The simplified model:

```
cost_per_call = P_in_fresh * tokens_in_fresh
              + P_in_create * tokens_in_to_cache    (only on the call that creates the cache)
              + P_in_read * tokens_in_cached         (on every call that reads the cache)
              + P_out * tokens_out
```

Where the prices (April 2026, list, USD per 1M tokens) for Claude Sonnet are roughly:

- `P_in_fresh = 3.00`
- `P_in_create = 3.75` (Anthropic charges a 25 percent premium for the create call)
- `P_in_read = 0.30`
- `P_out = 15.00`

For a workload of N calls all sharing a cacheable prefix of size T tokens:

```
cost_uncached = N * (P_in_fresh * T + P_out * tokens_out)
cost_cached   = P_in_create * T + (N - 1) * P_in_read * T + N * P_out * tokens_out
                                + N * P_in_fresh * tokens_in_per_call_unique
```

The savings over N calls, ignoring per-call unique input:

```
savings = N * P_in_fresh * T - P_in_create * T - (N - 1) * P_in_read * T
        = T * [N * P_in_fresh - P_in_create - (N - 1) * P_in_read]
        = T * [(N - 1) * (P_in_fresh - P_in_read) + (P_in_fresh - P_in_create)]
```

The first call pays a 25 percent premium; every subsequent call pays a 90 percent discount. Breakeven:

```
(N - 1) * (P_in_fresh - P_in_read) > P_in_create - P_in_fresh
N - 1 > (P_in_create - P_in_fresh) / (P_in_fresh - P_in_read)
N - 1 > (3.75 - 3.00) / (3.00 - 0.30)
N - 1 > 0.278
N > 1.28
```

In other words, on Anthropic Sonnet, caching pays off if the cached prefix is read more than once. Two calls is enough. This is why caching is "always on" for any agent loop with more than one turn against the same system prompt.

## Section 3: the OpenAI math is different

Run the same calculation for OpenAI `gpt-4o`:

- `P_in_fresh = 2.50`
- `P_in_create = 2.50` (no create premium)
- `P_in_read = 1.25`
- `P_out = 10.00`

```
N - 1 > (2.50 - 2.50) / (2.50 - 1.25)
N - 1 > 0
N > 1
```

Even better: any second call pays. But the discount per call is much smaller (50 percent vs 90 percent), so the magnitude of the savings is smaller. The breakeven point is sooner; the slope is gentler.

This has implications for which workloads benefit most. On Anthropic, a workload with 2 calls per session sharing 30k tokens of prefix saves 27 percent on input cost; the same workload on OpenAI saves 12.5 percent. On Anthropic, a workload with 200 calls per session sharing the same prefix saves 89 percent on input cost; on OpenAI, 49 percent. The longer the session, the more Anthropic's deeper discount matters.

## Section 4: where the math breaks down, workload one

A workload I expected caching to dominate but it did not: a tool that processes 500 short user queries per hour, each independent, each with a 5k-token system prompt that lists available tools.

The intuition was: identical 5k-token prefix, hundreds of calls, this should be a caching slam dunk. The reality: the cache TTL on Anthropic ephemeral cache is 5 minutes. At 500 queries per hour, average inter-arrival time is 7.2 seconds, well within the TTL. Should be fine.

But the queries arrive in bursts. The actual inter-arrival distribution had a long tail: median 4 seconds, p90 30 seconds, p99 280 seconds. About 8 percent of calls arrived more than 5 minutes after the previous call, so they paid the cache create cost again.

Effective cache hit rate: 92 percent on prefix tokens. Effective savings: 88 percent of theoretical max. Still a big win, but not as big as the headline math suggested. The lesson: the breakeven math assumes calls are within TTL of each other. Bursty arrival patterns invalidate that assumption.

The fix was a 5-minute keep-alive ping, a no-op call to refresh the cache. At 5k input tokens per ping, the ping cost was about 1 cent per hour, against savings of about 30 cents per hour from the fewer cache misses. Worth it.

## Section 5: where the math broke down, workload two

A workload I expected caching to fail at but it succeeded: a code-review pipeline that processes 50 PRs per day, each PR having a unique diff. The diff is the bulk of the input. The shared prefix (system prompt, repo context) was only about 8k tokens, against PR-specific content of 30 to 100k tokens.

The intuition was: the cached portion is a small fraction of total input, so the savings should be small. The reality: yes, the savings on the cached portion alone were small in percentage of input cost. But the bigger win was unexpected: latency.

Cache hits skip the prefill phase for the cached portion. With an 8k token prefix, that is meaningful prefill compute the model can skip. Time-to-first-token dropped from 1.8 seconds to 0.6 seconds, a 3x improvement. The cost savings were modest (about 7 percent of total bill); the latency improvement made the pipeline feel substantially more responsive to humans reviewing the agent's output.

The lesson: caching's economic value is not only the dollar discount. It is also the latency improvement, which has its own dollar value depending on what the latency was bottlenecking.

## Section 6: workload three, the routing case

A workload where the math said caching was a wash and that turned out to be correct: a router that fans the same input out to three different models in parallel and picks the best response. Each model is a separate provider, so the cache is per-provider and never shared.

The math: three calls, three different cache contexts, three create costs paid in full per session. Net savings vs no caching: zero on the first session, modest on subsequent sessions if any. Plus, the routing logic itself meant we did not always call all three on subsequent requests, so the cache was further fragmented.

I disabled caching on this path. The cache create premium plus the storage TTL waste was a net cost vs just paying the fresh input rate every call. The router's value was in model diversity, not in repeat patterns; caching's value is in repeat patterns. They do not combine.

## Section 7: workload four, the long-running agent

A workload where caching was the difference between economic feasibility and infeasibility: a long-running agent that runs for 200 to 500 model calls per session over the course of an hour, with a constantly-growing message history.

The math, naively: 200 calls, each with the full message history as input, at sonnet pricing, with an average input length growing from 30k tokens (turn 1) to 80k tokens (turn 200). Total input tokens per session: roughly 11 million. At fresh input pricing: 33 USD per session. Yes, per session.

With caching at 95 percent prefix hit rate (the practical achievable rate for an agent that follows the cache invalidation rules from the earlier post): the same session costs about 3.50 USD. A 9x reduction.

This is where caching stops being an optimization and becomes table stakes. Without it, my long-agent workload would have a per-session cost that would force me to throttle aggressively. With it, the cost-per-session is in the same range as a non-trivial database query, and I can let the agent run.

The lesson: for long-running agents, caching is not optional. The math says you cannot afford to run them at scale without it. Plan the prompt structure around caching from day one, not as a later optimization.

## Section 8: the per-token cost of latency

A subtle point that caching makes visible: the per-token cost of a model call has two components, the dollar cost and the latency cost. The dollar cost is what the bill says. The latency cost is what the user pays in waiting time, multiplied by whatever the user's time is worth in the application.

For a chat assistant where a human is waiting for the response, latency cost dominates dollar cost by a wide margin. A 1-second delay across 100 daily uses is 100 seconds per day per user, worth meaningful money for any user with a salary. A 0.3-second cost reduction in that same flow is, in dollar-equivalent terms, often larger than the entire input bill.

For a batch job that runs unattended, latency cost is essentially zero and dollar cost is everything.

Caching reduces both. The dollar reduction is a function of the price multiplier. The latency reduction is a function of how much prefill compute is skipped, which is approximately quadratic in cached prefix length on the prefill side, linear on the decode side. For a 50k-token prefix, the latency improvement can be several seconds; for a 1k-token prefix, milliseconds.

I now think of caching as two separate optimizations layered on the same mechanism. For batch jobs I tune the cache to maximize dollar savings (as much prefix as possible, long TTL via keep-alives, batch within the TTL). For interactive use I tune for latency (cache anything I will reuse within 30 seconds, even if the dollar savings are modest, because the latency win pays for itself).

## Section 9: the case for not caching

Three patterns where caching is the wrong choice and I have explicitly disabled it:

The first: one-shot calls with no expected reuse. A user runs a CLI command that calls the model once and exits. Caching pays for itself starting on the second call, so the first call alone never benefits. Worse, on Anthropic, the create premium means the one-shot is 25 percent more expensive cached than uncached. Disable for one-shots.

The second: workloads with high prompt diversity. If the system prompt changes every call (because it embeds per-request context that should arguably be in the user message), caching is impossible. Either restructure the prompt to expose a stable prefix, or accept that the workload does not fit the caching model and stop trying.

The third: when the cache TTL is much shorter than the inter-call interval. If your calls arrive every 10 minutes and the cache TTL is 5 minutes, you are paying the create premium every call with no offsetting reads. Either decrease the inter-call interval (batch), or extend the TTL with keep-alives, or disable.

In all three, the right move is to align the workload with the caching model or to opt out, not to enable caching and hope.

## Section 10: the reading I would do next

The provider's blog posts on caching are useful but not deep. The deeper material that helped me actually understand the cost structure:

- The vLLM and SGLang papers, which describe efficient KV cache management and the prefix-sharing optimizations that make this commercially viable.
- The "Efficient Memory Management for Large Language Model Serving with PagedAttention" paper, which is the canonical reference for how production inference servers manage KV cache memory.
- The "Splitwise" paper from Microsoft Research, which separates prefill and decode onto different hardware to optimize both phases independently. Helps explain why provider pricing distinguishes them.

The economics of cached input are not a marketing decision. They are a window into the inference stack. Reading the prices as cost-pass-through and aligning your workload accordingly is the difference between paying the bill and paying twice the bill.

## References

- Anthropic prompt caching pricing: https://www.anthropic.com/pricing
- OpenAI prompt caching docs: https://platform.openai.com/docs/guides/prompt-caching
- vLLM PagedAttention paper: https://arxiv.org/abs/2309.06180
- SGLang RadixAttention paper: https://arxiv.org/abs/2312.07104
- Splitwise paper: https://arxiv.org/abs/2311.18677
