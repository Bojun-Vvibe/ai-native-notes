# Phantom Input: the 98.2% Zero-Input Rows of the Editor-Assistant Source

> **Captured:** 2026-04-26T00:29:03Z
> **Tool:** `jq -s 'group_by(.source) | …' ~/.config/pew/queue.jsonl`
> **Corpus:** 1,500 hourly buckets across 6 sources (one operator, one device).

## The number that should not exist

Among the six telemetry sources currently feeding the local queue, one of them — the editor-assistant source (the IDE-bound chat surface that runs inside the editor process, not the standalone CLI) — reports **327 of its 333 rows with `input_tokens == 0`**. That is **98.2%** of all its hourly buckets, claiming the model produced output without consuming any input.

```json
{
  "source": "editor-assistant",
  "rows": 333,
  "zero_output_rows": 12,
  "zero_input_rows": 327,
  "total_input": 581090,
  "total_output": 1135247,
  "total_cached": 0
}
```

For comparison, every other source in the corpus has **zero** zero-input rows:

```json
[
  {"source":"openclaw","rows":378,"zero_input_rows":0},
  {"source":"claude-code","rows":299,"zero_input_rows":0},
  {"source":"opencode","rows":272,"zero_input_rows":0},
  {"source":"hermes","rows":154,"zero_input_rows":0},
  {"source":"codex","rows":64,"zero_input_rows":0}
]
```

Five sources at 0%. One source at 98.2%. There is no plausible reading in which the model genuinely received zero prompt tokens 327 separate hours and still emitted, on aggregate, 1,135,247 output tokens. Something in the accounting pipeline is silently dropping the prompt side of the ledger.

This is the most extreme telemetry-truthfulness anomaly in the entire 1,500-row corpus, and it has been hiding in plain sight because the *output* numbers look fine.

## What "zero input" cannot mean

Before reaching for an explanation, it is worth eliminating the things this number cannot mean.

It cannot mean the model was prompted with an empty string. Modern chat completions reject empty `messages[]` arrays and, even when they accept them, the system prompt alone is dozens to hundreds of tokens. A truly empty input would surface as a 400 error, not as a successful row with 1,280 output tokens.

It cannot mean the user typed nothing. Even the smallest editor-side gesture — "explain this", "fix the lint", a single character followed by Tab — carries an attached file context, a selection, an open-tab list, a workspace summary. The minimum realistic prompt for an inline-edit request is on the order of 2–4 KB of text, which is hundreds of tokens at minimum.

It cannot mean caching ate the input. The same row reports `cached_input_tokens: 0`. If caching had absorbed the prompt, the cached field would be non-zero and the input field would still typically reflect either the cache-write cost or the uncached delta. Both at zero is not a cache-hit signature; it is a missing field signature.

And it cannot mean the model is one of those that doesn't bill input separately. The same source, in the small minority of rows where it *does* report input, reports it correctly:

```json
{
  "source":"editor-assistant","model":"claude-opus-4.6",
  "hour_start":"2026-03-20T09:30:00.000Z",
  "input_tokens":50500,"cached_input_tokens":0,
  "output_tokens":1881,"reasoning_output_tokens":381,
  "total_tokens":52762
}
```

So the schema is capable of carrying input. The pipeline has just chosen not to in 98.2% of cases.

## What it almost certainly does mean

The most defensible reading is that the editor-assistant source has **two distinct upstream usage reporters**, and only one of them populates `input_tokens`. The shape of the data supports this:

- The 6 non-zero-input rows look like complete usage frames (input, sometimes cached, output, reasoning, all coherent with `total_tokens`).
- The 327 zero-input rows look like *truncated* usage frames where only the model's own emission has been measured. The `total_tokens` field on those rows equals `output_tokens + reasoning_output_tokens` exactly, with no input contribution.

That pattern is consistent with editor-assistant providers who bill the IDE on a "completions delivered" basis and only return the output side in their per-request usage object, leaving the prompt side either rolled up into a separate enterprise-billing endpoint or simply not exposed to the client at all. The CLI-class sources in the same corpus (codex, opencode, claude-code, openclaw, hermes) all run against APIs that return a complete `usage{}` object on every request, which is why they are at 0% phantom-input.

In other words: **the missing input is not missing from the model's perspective, only from the client's.** The provider knows what it billed; the local telemetry collector does not.

## Why this matters more than it looks

It is easy to dismiss this as "just an editor source quirk" and move on. That would be a mistake, for three reasons.

### 1. Every fleet-aggregate input statistic is biased downward

Any analysis that sums `input_tokens` across sources — input/output ratio, cost-per-token estimates, prompt-velocity-per-hour, anything — is silently undercounting the editor-assistant source's prompts by roughly 98%. If you naively compute "total fleet input tokens" from this corpus and compare sources, the editor-assistant source will look like an order of magnitude more output-efficient per input token than it actually is. It isn't. Its inputs are simply unrecorded.

The 1.135M reported output tokens from this source, divided by the 581K reported input, gives a ratio of 1.95 output per input. That number is wrong by approximately 50× — the true ratio, given typical editor prompts of 2–8K tokens per turn, is more like **0.04 output per input**, i.e. classic agent-style "huge prompt, small completion" behavior.

This is not a small bias. It flips the qualitative reading of which source is the most "talkative" in the fleet.

### 2. The zero-input rows have non-zero reasoning tokens

Spot-check a single phantom-input row:

```json
{
  "source":"editor-assistant","model":"gpt-5.4",
  "hour_start":"2026-04-13T02:00:00.000Z",
  "input_tokens":0,"cached_input_tokens":0,
  "output_tokens":1280,"reasoning_output_tokens":2410,
  "total_tokens":3690
}
```

Two thousand four hundred ten reasoning tokens, allegedly produced from zero input. Reasoning tokens are a strict function of prompt complexity for current-generation models — you do not get 2,410 chain-of-thought tokens from an empty context window. This is the strongest single-row evidence that the input field is being *dropped*, not legitimately absent.

It also means: any analysis of "reasoning intensity per prompt token" for this source is meaningless. The denominator is fictional.

### 3. The pattern is undetectable from output-side checks alone

If you only look at `total_tokens`, the editor-assistant source looks healthy. 1,500 buckets, 1.7M total tokens, no obvious outliers. The bug is invisible unless you specifically partition by "rows with input_tokens == 0", which is not a natural slice — most analytics dashboards don't draw that line because no other source in the corpus has any rows in it.

This is the canonical shape of a silent telemetry bug: the data is internally consistent (`total_tokens` adds up), the per-row schema is valid (no nulls, no negatives), and the source isn't *failing* — it is just systematically lying about half the ledger. You only catch it when you cross-reference against sources that don't lie, and notice that one is at 98.2% on a bucket-flag the rest are at 0% on.

## What to do about it

Three things, in increasing order of effort.

**First, mark and exclude.** In any per-source comparison of input-side metrics — input volume, input/output ratio, cache hit rates (the editor-assistant source is also at 0/0 on cached_input, which is the second telemetry hole), reasoning-per-prompt-token — explicitly drop rows where `source == "editor-assistant" && input_tokens == 0`. That is 327 rows to ignore. The remaining 6 give you a tiny but honest sample.

**Second, switch the denominator.** When the editor-assistant source must be included in a fleet-aggregate, use `total_tokens` (which is at least internally consistent on every row) instead of `input_tokens` for normalization. You lose the input/output decomposition, but you stop propagating the bias.

**Third, fix it upstream.** The 327 phantom-input rows are not a one-off — they span from the earliest editor-assistant timestamp in the corpus (2025-09-11) through 2026-04-26, which is roughly 7.5 months of consistent under-reporting. That means the editor-side telemetry collector has *never* captured input for the bulk of its requests. Whichever ingest path produces these rows (almost certainly a hook that listens for `usage` events from a provider that doesn't include `prompt_tokens` in its streaming response) needs a second source — either an explicit token-count call before the request, or a client-side `tiktoken`-equivalent estimate stored in a separate `input_tokens_estimated` field.

The estimate route is the cheaper fix and is well-established: open-source CLI clients like `codex` and `opencode` already maintain client-side tokenizers for cost preview purposes, and could expose them as a public utility. Wiring an editor-side hook to call a shared tokenizer on the outbound prompt before each completion would close the gap at zero marginal cost per request, since the tokenization happens client-side and adds no provider round-trip.

## What this says about telemetry plumbing in general

The deeper lesson is about how AI-tool telemetry pipelines actually fail. They almost never fail by going dark — that gets noticed within a sync cycle. They fail by **dropping fields silently**, leaving the row count and total-tokens columns intact while quietly removing one of the dimensions you need to compute the ratios that actually matter.

The editor-assistant source's 98.2% phantom-input rate is the largest example in this corpus, but it is not the only one. The same source also reports `cached_input_tokens: 0` on **every single row** (333 of 333), which is implausible for an editor that has been running against cache-supporting models for months — every other source in the corpus that runs the same models reports non-trivial cached-input volume. Two telemetry holes in the same source, from the same upstream pipeline, both invisible to top-line metrics.

The actionable principle: **for every source, compute the rate at which each individual usage field is zero, and chart it across sources.** Any source with an unusually high zero-rate on a given field, relative to peer sources running comparable workloads, is reporting a hole rather than a measurement. The hole rates are the ground truth of telemetry quality, and in a multi-source fleet they are easy to compute and impossible to game.

## The bottom line

Five out of six telemetry sources report input on every row. One reports input on 1.8% of rows. That one source is not getting magic free prompts; it is shipping a truncated usage frame, and every analysis that touches its input column is wrong by approximately the same factor.

The fix is straightforward — exclude, renormalize, or estimate — but it requires noticing first, and the noticing requires partitioning by a field-presence flag that no dashboard naturally surfaces. Until then, the editor-assistant source will continue to look like the most prompt-efficient provider in the fleet, on the strength of input numbers it never measured.

Sometimes the most important number in a dataset is the count of how often another number is zero.
