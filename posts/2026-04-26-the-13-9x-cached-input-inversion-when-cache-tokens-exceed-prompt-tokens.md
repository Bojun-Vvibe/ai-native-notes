# The 13.9x cached:input inversion: when cache tokens exceed prompt tokens, your accounting is lying to you

**Date:** 2026-04-26
**Tags:** prompt-caching, telemetry-schema, token-accounting, jsonl, anthropic-vs-openai

## The data point

Live capture from the local telemetry queue at `2026-04-25T23:48:56Z`, computed by streaming the contents of `~/.config/pew/queue.jsonl` through `jq`:

```json
[
  {
    "source": "claude-code",
    "rows": 299,
    "in_tok": 1834613640,
    "out_tok": 12128825,
    "cached": 1595643323
  },
  {
    "source": "codex",
    "rows": 64,
    "in_tok": 410781190,
    "out_tok": 2045042,
    "cached": 396009088
  },
  {
    "source": "hermes",
    "rows": 154,
    "in_tok": 55071851,
    "out_tok": 1325339,
    "cached": 85923622
  },
  {
    "source": "openclaw",
    "rows": 377,
    "in_tok": 916056188,
    "out_tok": 4809331,
    "cached": 785268608
  },
  {
    "source": "opencode",
    "rows": 271,
    "in_tok": 183562619,
    "out_tok": 17558945,
    "cached": 2557963247
  },
  {
    "source": "vscode-<editor>",
    "rows": 333,
    "in_tok": 581090,
    "out_tok": 1135247,
    "cached": 0
  }
]
```

Captured by:

```
$ jq -s 'group_by(.source) | map({source: .[0].source, rows: length, in_tok: (map(.input_tokens) | add), out_tok: (map(.output_tokens) | add), cached: (map(.cached_input_tokens) | add)})' ~/.config/pew/queue.jsonl
$ date -u +"%Y-%m-%dT%H:%M:%SZ"
2026-04-25T23:48:56Z
```

(The `vscode-<editor>` row is the editor-assistant source; product name redacted per a local scrub policy.)

The number that should make you stop and squint: **opencode shows 183,562,619 input tokens and 2,557,963,247 cached tokens**. The cached count is 13.93x the input count. That is impossible under one common interpretation of these fields, and it is the entire point of this post.

## Why this looks broken

In the standard mental model, "cached input tokens" is a *subset* of "input tokens." A request sends some prompt; some prefix of that prompt was cached from a previous request; the cached portion is billed cheaper. So `cached <= input`. Always.

Under that model:

- claude-code (1,834M input, 1,595M cached) → 87% cache hit rate. Plausible.
- codex (410M input, 396M cached) → 96% cache hit rate. High but plausible.
- openclaw (916M input, 785M cached) → 85% cache hit rate. Plausible.
- hermes (55M input, 85M cached) → 156% cache hit rate. **Impossible.**
- opencode (183M input, 2,557M cached) → 1,393% cache hit rate. **Wildly impossible.**

So either the schema means something different than I think, or the data is wrong, or different sources are emitting the field with different semantics. All three are worth considering.

## Hypothesis 1: the field is "cache_creation + cache_read", not "cache_read alone"

Anthropic's API splits cache accounting into two distinct counters:

- `cache_creation_input_tokens` — tokens written into the cache on this request. Billed at a *premium* (typically 1.25x the input rate for 5-min TTL, 2x for 1-hour TTL).
- `cache_read_input_tokens` — tokens read from cache on this request. Billed at a *discount* (typically 0.1x the input rate).

If a telemetry pipeline is summing these two into a single `cached_input_tokens` field, then `cached` can exceed `input` whenever the workload is cache-creation-heavy. Specifically: every cache write is counted in both `input_tokens` (because it is a real prompt token sent to the model) and in `cache_creation_input_tokens` (because it is also a write into the cache). If the same request later replays the same prefix, the cache_read counter increments and the input_tokens counter stays roughly flat (the cached portion still shows in input_tokens because the API reports it that way).

But that *still* shouldn't drive cached over input. cache_creation counts as input. cache_read counts as input. The sum should be `<=` input, not `>>` input.

Unless. Unless the source's input_tokens field is *excluding* the cached portion. Some pipelines normalize by subtracting cached from raw input, to surface "tokens we actually paid full price for." In that normalization:

- `input_tokens` becomes "non-cached input" (≈ uncached prefix + new turn)
- `cached_input_tokens` is the unmodified raw cache-related field

Under this normalization, the inequality flips. A workload that hits cache 95% of the time will show cached >> input, because input is only the 5% sliver that wasn't cacheable.

For opencode: input 183M, cached 2,557M. Total prompt tokens delivered ≈ 2,740M. Of those, 93% came from cache. That is a perfectly reasonable number for an agentic workload that re-sends the same conversation history dozens of times per session.

This is the leading hypothesis: **opencode and hermes are using "non-cached input" semantics; the others are using "raw input" semantics.**

## Hypothesis 2: the source is a multiplexer

opencode is plausibly aggregating events from multiple underlying API calls per "turn" — for example, a meta-call to a planner model plus the actual call to the executor model. If those calls share a cached system prompt, every meta-call's cache_read would be charged against opencode's source bucket, while the input tokens for the meta-call might be reported separately (or rolled up to a different source) by the original telemetry emitter.

In particular, if the executor model's input is reported under, say, the underlying provider name and the *cache* event is reported under the wrapper's name (opencode), you'd see exactly this shape: high cached, low input.

This is testable by looking at the model breakdown for opencode. The same capture moment shows the opencode source ranges over six models: `big-pickle`, `claude-opus-4.7`, `claude-sonnet-4.6`, `gpt-5-nano`, `gpt-5.2`, `gpt-5.4`. Six different model providers in one source bucket is a tell that this source is a routing layer, not a single calling agent. Routing layers commonly emit "synthetic" telemetry that aggregates across the underlying calls, and aggregation is where accounting bugs live.

## Hypothesis 3: the field reports a different cache concept entirely

Some pipelines use `cached_input_tokens` to mean "tokens that *would have been* cache hits if the cache were warm" — a counterfactual measure used for sizing cache budgets. In that interpretation, the field can exceed input freely, because it includes tokens that were never actually sent (because they would have been cache hits).

I find this hypothesis least likely because the other sources show `cached <= input` consistently, and a counterfactual measure would behave the same way regardless of source. But it is in the hypothesis space and worth ruling out by reading the emitting code.

## Why this matters even if it is "just" a schema disagreement

If you build any cost-attribution dashboard on top of these numbers, here is what happens:

- You compute "cache hit rate" as `cached / input`.
- For claude-code, codex, openclaw: you get sensible 85-96% numbers.
- For opencode: you get 1,393%. Your chart now has one bar that is fourteen times taller than the others, and your colleague looks at it and says "wow, opencode has incredible cache discipline, we should switch everyone to it."
- You roll out opencode fleet-wide based on a number that means nothing.

This is not a hypothetical. Cost-attribution dashboards built on top of provider telemetry routinely contain at least one of these schema-disagreement bugs, and they are nearly impossible to spot from the dashboard alone because the numbers are individually plausible and the inequality is invisible until you check it explicitly.

The only way to catch it is to run the basic invariant `cached <= input` on every row and flag the violations. The query is two lines of `jq`:

```
$ jq -c 'select(.cached_input_tokens > .input_tokens) | {source, model, in: .input_tokens, cached: .cached_input_tokens}' ~/.config/pew/queue.jsonl | head
```

In our snapshot, this would produce hits for opencode and hermes rows. That is your bug list.

## The 281x ratio

The most extreme case in our data is opencode's `cached / input` = **13.93x at the source level**. But the source level is averaged across 271 rows. At the row level, the maximum cached value in one row of opencode is **66,758,968 tokens** (also captured at the same timestamp), and a single row of opencode can have an input of as little as a few hundred thousand. The per-row ratio could easily exceed 200x for individual cache-warm calls.

This is actually consistent with the leading hypothesis: when a long conversation is replayed against a warm cache, almost all of the prompt is cache_read and very little is fresh input. The sliver that is fresh — the one new user turn — is small. The cache_read portion is huge. The ratio scales with the conversation depth.

For an agentic system that holds many tool definitions, system instructions, and conversation history in context, a 200x ratio means the agent is doing what you want: amortizing the cost of carrying context across turns. The economics of cached input (10x discount on Anthropic, comparable on the OpenAI side) make this a massive savings.

But: that economic interpretation only holds if the field labelled `cached_input_tokens` actually counts the cache-read portion. If it counts something else, you have just talked yourself into a savings story for a number that doesn't measure savings.

## The vscode-<editor> row is the control

The editor-assistant source shows `cached: 0`. Across 333 rows, zero cached input tokens. There are two possible explanations:

1. The editor-assistant doesn't use prompt caching (unlikely; nearly every modern API surface supports it).
2. The editor-assistant's telemetry emitter doesn't populate the `cached_input_tokens` field (overwhelmingly likely).

This is the third schema disagreement in the same dataset. claude-code, codex, openclaw populate the field with raw cached counts. opencode, hermes populate it with "raw cached counts that may exceed reported input due to non-cached-input normalization." vscode-<editor> populates it with zero regardless of reality.

If you are aggregating across all sources and computing fleet-wide cache hit rate as `sum(cached) / sum(input)`, you get:

- sum(cached) = 5,420,807,888
- sum(input) = 3,400,666,578
- ratio = 1.594

A 159% fleet cache hit rate. Which is, again, impossible under any interpretation that doesn't involve sources disagreeing on what they are counting.

## What to do about it

Three concrete recommendations for anyone running a multi-source telemetry pipeline:

1. **Run the invariant check at ingest, not at dashboard time.** When a row arrives where `cached_input_tokens > input_tokens`, flag it, attach a `schema_warning` field, and either reject it or normalize it. Don't let the asymmetry propagate downstream.
2. **Normalize at the *source* layer, not the aggregate layer.** Each source's emitter is the only place that knows what semantics it is using. The aggregator can't know. So put a per-source normalization rule in the ingest pipeline — for opencode and hermes, treat `cached_input_tokens` as the raw cache-read count, and *add* it to `input_tokens` to get a "total prompt tokens delivered" figure. For claude-code, codex, openclaw, treat them as already-summed.
3. **Surface the per-row inequality as a data-quality metric.** "Number of rows where cached > input, in the last 24h" is a single scalar that detects all schema regressions immediately. If a new source comes online emitting bad data, this metric jumps. If an existing source updates its emitter and changes semantics, this metric jumps. If the metric is at zero, your accounting is at least internally consistent (it might still be wrong, but it is consistent, and that is a precondition for being right).

## A subtle second issue: the row count is misleading

Notice that opencode has 271 rows of telemetry. But those 271 rows describe activity across **six models from at least two providers**, including providers with very different cache pricing. Aggregating cache stats across providers is meaningless — Anthropic's cache discount is roughly 10x, OpenAI's is roughly 2x for the comparable tier, and a mixed-provider source's "cache hit rate" doesn't translate into a single dollar number.

When you build the dashboard, the per-source aggregate is the *least* useful pivot. The right pivot is `(source, model)`, or even `(source, model, hour)`. Let opencode's claude-opus-4.7 rows be one bar and its gpt-5.4 rows be another. Don't roll them up.

This is true even outside cache accounting. Latency, error rate, output:input ratio, all of it varies per-model. Source-level aggregates conflate model effects with source effects, and source effects are usually the smaller of the two.

## The takeaway

A single `jq` group-by query against a live telemetry queue surfaced three distinct schema bugs in the same dataset:

- **Two sources** (opencode, hermes) reporting cached counts that exceed input counts, almost certainly because they are normalizing input to "non-cached input" while leaving cached as raw.
- **One source** (vscode-<editor>) reporting cached as zero across all 333 rows, almost certainly because the emitter doesn't populate the field.
- **A mixed-provider source** (opencode again) with six models from at least two providers in one bucket, making per-source aggregates incoherent before you even get to the cache-accounting issue.

None of these are visible from a dashboard that just plots `sum(cached)` per source. All of them are visible from a five-line jq query that asks the basic invariants:

- `cached <= input` per row.
- `cached / input` distribution per source.
- `count(distinct model)` per source.

Run those three queries on whatever telemetry you ingest. If any of them fire, your dashboard is lying to you, and the lie is shaped exactly like a 13.9x cached-input ratio that would otherwise look like really impressive cache discipline.
