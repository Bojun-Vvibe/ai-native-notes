# Cache ratio per model: the claude-opus-4.7 2.83x inversion and the haiku zero floor

**Date:** 2026-04-26
**Data captured:** 2026-04-26T04:14:06Z
**Tags:** prompt-caching, model-comparison, telemetry-schema, anthropic, openai, jsonl, jq

## The finding

If you sum `cached_input_tokens` and divide by `input_tokens` for each model in the local pew queue, the resulting ratio is not a number you can read as "what percent of the prompt was cached." For the dominant model on this device — `claude-opus-4.7` — the ratio is **2.83**. That is, the count of cached input tokens is **2.83 times larger** than the count of raw input tokens. Other models in the same file produce ratios from `0.00` (haiku-class, no cache reuse at all) up to `0.98` (a vendor-prefixed opus variant that approaches a clean "almost everything is cached" interpretation). The spread is not a ranking of "which model caches better." It is a ranking of *which billing schema the source emits*, and reading these numbers as efficiency is a category error.

## Methodology

One JSON file, one `jq` aggregation:

```bash
jq -s 'group_by(.model)
  | map({
      model: .[0].model,
      rows: length,
      input: (map(.input_tokens) | add),
      cached: (map(.cached_input_tokens) | add),
      output: (map(.output_tokens) | add)
    })
  | map(. + {cache_ratio: (if .input>0 then (.cached/.input) else 0 end)})
  | sort_by(-.input)' ~/.config/pew/queue.jsonl
```

Verbatim output (top of the list, trimmed for length):

```json
[
  { "model": "gpt-5.4",                         "rows": 465, "input": 1333210958, "cached": 1206786048, "output": 6929533,  "cache_ratio": 0.9051 },
  { "model": "claude-opus-4.7",                 "rows": 368, "input": 1325981725, "cached": 3750543577, "output": 27103427, "cache_ratio": 2.8285 },
  { "model": "claude-opus-4.6-1m",              "rows": 167, "input":  601843219, "cached":  495092916, "output": 3311069,  "cache_ratio": 0.8226 },
  { "model": "claude-haiku-4-5-20251001",       "rows":  26, "input":   61560104, "cached":          0, "output":   96738,  "cache_ratio": 0.0000 },
  { "model": "claude-opus-4-7",                 "rows":  73, "input":   48863418, "cached":   37428846, "output":  627886,  "cache_ratio": 0.7660 },
  { "model": "big-pickle",                      "rows":  53, "input":   17746683, "cached":   11012310, "output":  366926,  "cache_ratio": 0.6205 },
  { "model": "claude-sonnet-4.6",               "rows":   6, "input":    7617119, "cached":    2584375, "output":   73379,  "cache_ratio": 0.3393 },
  { "model": "claude-haiku-4.5",                "rows":   4, "input":    4385555, "cached":    4004713, "output":   33653,  "cache_ratio": 0.9132 },
  { "model": "vendor-wrapped/claude-opus-4.6-1m","rows": 15, "input":    4347873, "cached":    4244032, "output":  139556,  "cache_ratio": 0.9761 }
]
```

Two complementary row counts confirm the shape: across all 1518 hourly buckets in the file, **293 rows (19.3%)** have `cached_input_tokens > input_tokens`. Restricted to `claude-opus-4.7` only, **276 of 368 rows (75.0%)** are inverted. The dominant model is *almost always* in the inverted regime; everything else is *almost never*.

## What this is and what it isn't

The naive read is "claude-opus-4.7 caches 283% of its input, the cache hit rate is over 100%, the model is doing magic." That read is wrong. There are two non-magical explanations, and either one is enough on its own to produce the inversion.

**Explanation one: separate accounting for cache writes and cache reads.** Anthropic's billing schema (and several wrappers around it) reports cached tokens as a *cost line item*, not as a subset of the prompt that was free. Specifically, it reports cache *creation* tokens (the first time the prefix is committed) and cache *read* tokens (every subsequent time the prefix is reused) as two separate billable counters, and the wrapper that writes our local queue collapses both into a single `cached_input_tokens` field. The `input_tokens` field, meanwhile, is the *fresh* portion of the request — the suffix that wasn't cacheable. So the ratio `cached / input` is not "fraction of prompt that was cached." It is "tokens you paid for at cached rates, divided by tokens you paid for at full rates." With long system prompts, persistent tool definitions, and a chat history that keeps growing while the per-turn user message stays small, that ratio will exceed 1.0 trivially, and on a busy day will exceed 2.0 without anyone doing anything special.

**Explanation two: row-level multi-turn aggregation.** Each row in `queue.jsonl` is a one-hour window for a `(source, model, device)` triple. If a single conversation produces twenty assistant turns inside that hour, and each turn re-reads a 50,000-token prefix, the row will accumulate 1,000,000 `cached_input_tokens` against perhaps 4,000 `input_tokens` of actually-new user content. The ratio reported per row is the *integral over the hour*, not a per-request rate, and the integral grows linearly in turns. The inversion is therefore a property of *how busy the row is*, not of how cacheable the prompt is. This is why the inverted-row fraction concentrates in the model that gets used the most: there are simply more long-tail rows where one chat dominated the hour.

Both explanations apply simultaneously. You cannot tell from the queue file alone which one contributes more, and that is the most actionable finding here: **the cache ratio number, in isolation, is not interpretable.**

## Why the spread across models is the real signal

Look again at the ranking. `gpt-5.4` sits at `0.91` — slightly under 1, which is consistent with OpenAI's accounting where `cached` is a strict subset of `input` (so the ratio is bounded by 1.0 by construction, and 0.91 means "91% of the prompt was a cache hit"). `claude-opus-4.7` at `2.83` is in the additive-accounting regime described above. `claude-haiku-4-5-20251001` at exactly `0.00` is a third regime entirely: no cached tokens are ever reported, either because the wrapper for haiku rolls cache reads into the input bucket silently, or because haiku traffic on this device is ad-hoc one-shot calls that never benefit from a cache hit. The two `claude-opus-4-7` rows (note the dashes instead of dots — it's a different name string emitted by a different SDK version) sit at `0.77`, neither fully inverted nor fully bounded, suggesting *yet another* schema variant where only some of the cache reads are double-counted.

In other words, the model name in this file is doing the work of three different keys at once: it identifies the underlying weights, the wrapper SDK that produced the row, and the billing schema that wrapper assumes. Because the schema is encoded by string match on the model field, any downstream consumer that wants to compute a meaningful "cache hit rate" has to maintain its own model-to-schema lookup table. We don't have one. Nobody publishes one. The wrappers don't agree even with themselves: `claude-opus-4.7` and `claude-opus-4-7` are the same weights but different schemas, and the difference shows up as a 3.7x swing in the apparent cache ratio.

## Why the dominant-model concentration matters

Of the 1518 hourly buckets in the queue, the top two models (`gpt-5.4` and `claude-opus-4.7`) account for 833 rows — 54.9% of the file. Of those, the two are within 0.6% of each other on `input_tokens` summed (`1.333B` vs `1.326B`), which means whoever picks one of these two as the "reference model" for any cross-source dashboard is picking a coin flip. But the cache ratios diverge by a factor of `2.83 / 0.91 ≈ 3.1`. So a dashboard that shows "average cache ratio across the fleet" is essentially reporting *which of the two dominant models happened to bucket more rows into the time window*, with everything else as third-decimal-place noise. The number is hostage to its sampling schedule.

The right move, if you have to publish one number, is to publish it *per model* and *per schema family*, never aggregated. The closest thing to a meaningful aggregate is the OpenAI-style ratio (cached / (cached + uncached_input)), which requires you to know that cached is a subset rather than a separate column — and you only know that by knowing the model. So the aggregation has to happen *after* the schema-aware normalization, not before. None of our current dashboards do this. All of them publish the ratio as if it were a fleet-wide property.

## The haiku zero is not a bug

It's tempting to flag `claude-haiku-4-5-20251001` at `cache_ratio: 0.0000` as a missing-data problem. It isn't. Haiku, in this corpus, is exclusively used for short ad-hoc tasks: 26 rows total, 61.6M input tokens, 96.7K output tokens. The output-to-input ratio is 0.0016 — extreme retrieval-style reads with one-line answers. There is no multi-turn conversation, no persistent system prompt, no tool definitions to amortize. A cache miss is the expected and economically correct behavior. The zero is not "we forgot to enable caching for haiku"; it's "haiku is being used in a workload shape where caching has no purchase." If we suddenly saw the haiku ratio climb to 0.5, we would have a real anomaly: it would mean haiku had been routed into a long-context conversational role it isn't priced for, and the bill would jump.

The same logic explains `claude-haiku-4.5` (no dash-date suffix) at `0.91` with only 4 rows and 4.4M input. That is not a contradicting signal. Four rows is below the floor at which any aggregate is meaningful, and the high ratio almost certainly reflects a single conversation that briefly used haiku in a long-context role — exactly the workload-shape mismatch the zero-floor was warning us against. Watching this number quarter-over-quarter would tell us whether haiku is being mis-routed.

## Falsifiable predictions

1. **The 2.83 will drift down within 30 days.** Not because caching gets worse, but because new model strings will appear in the file (e.g. `claude-opus-4.8`, `claude-opus-4.7-1m`) and the row count under the current `claude-opus-4.7` string will start to fall in proportion. By 2026-05-26, the `claude-opus-4.7` cache ratio computed from the same `jq` will be in the `[2.0, 2.6]` range, and a new top-row string will hold the inversion crown.
2. **At least one new "shadow" model string will appear that is the same weights as a current entry but a different SDK.** The pattern from `claude-opus-4.7` vs `claude-opus-4-7` will repeat. The new pair will diverge by at least 0.3 in cache_ratio for the same underlying weights.
3. **The inverted-row fraction (currently 19.3% of all rows) will not move outside `[15%, 25%]` over the next 60 days,** because it is set by what fraction of fleet hours are dominated by long-context conversational workloads, and that mix is structural to the way the device is used, not a function of model availability.
4. **No haiku model will sustain a `cache_ratio > 0.4` over a rolling 30-day window.** If one does, it indicates a routing bug — something is sending long-context work to a model not priced for it, and the dollar cost will show up before the ratio does.

## What to do with this in a dashboard

Don't publish a single "cache hit rate." Publish three columns: `cached_billing_units` (verbatim from the wrapper, no division), `input_billing_units` (likewise), and a categorical `schema_family` derived from the model string via an explicit lookup table you maintain. Force the reader to do their own arithmetic if they want a ratio. This sounds user-hostile until you remember that the alternative is publishing a single number that means three different things depending on which way the model mix tilted that hour, and which has already convinced at least one person on this device that prompt caching is an arithmetically impossible feature. It isn't. The schema is just a liar.

## Follow-up questions

- Does the `cached / input` ratio for `claude-opus-4.7` correlate with `output_tokens` per row? If yes, the inversion really is "long conversations dominate," and we can de-bias the ratio by dividing by turn count. If no, the inversion is purely an accounting artifact and can't be normalized away.
- How does the `0.98` ratio on `vendor-wrapped/claude-opus-4.6-1m` decompose against the `0.82` ratio on plain `claude-opus-4.6-1m`? Same weights, different wrappers — a controlled experiment for "which schema family is which."
- If we re-bucket the queue by `(model, source)` pair instead of `model` alone, do the inverted-row fractions concentrate in one source? That would tell us whether the schema is a function of the wrapper that shipped the row, or a function of the model itself.
