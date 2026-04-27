---
title: "The cached > input paradox: 402 rows, 1518% opencode ratio, and the Anthropic token accounting trap"
date: 2026-04-27
tags: [data, tokens, pew, queue-jsonl, cache, anthropic, accounting]
---

# The cached > input paradox: 402 rows, 1518% opencode ratio, and the Anthropic token accounting trap

There is a 1,703-row JSONL ledger sitting in `~/.config/pew/queue.jsonl` with a schema so simple it looks impossible to misread:

```json
{
  "source": "opencode",
  "model": "claude-opus-4.7",
  "hour_start": "2026-04-23T17:30:00.000Z",
  "input_tokens": 45383,
  "cached_input_tokens": 3132549,
  "output_tokens": 12410,
  "reasoning_output_tokens": 0,
  "total_tokens": 3190342
}
```

Five token counters, one timestamp, one model, one source. Anyone seeing this for the first time will write the same line of code:

```python
cache_hit_ratio = cached_input_tokens / input_tokens
```

And then they will look at the result and quietly go back to work, because the number they get back is **246.39%** for the claude family, **89.97%** for the gpt-5 family, **1518.98%** for the opencode source, and an arithmetic-average **17.22** for opencode's row distribution. The metric that is supposed to be bounded between zero and one returns sixty-nine *times* one. Across the full 1,703-row ledger, **402 rows (23.61%)** have `cached_input_tokens > input_tokens` — almost a quarter of the data violates the canonical denominator assumption. This post is about why that happens, what it means, and why the "cache hit ratio" intuition imported from CDN and database literature is actively misleading when applied to LLM token billing.

## The five extreme rows

Sort the 402-row anomaly subset by `cached / input` ratio descending and the top five all come from the same harness, the same model, and a four-day window:

```
opencode  claude-opus-4.7  2026-04-23T17:30  cached/input=69.02x  in=45,383     cached=3,132,549
opencode  claude-opus-4.7  2026-04-22T08:30  cached/input=66.02x  in=106,375    cached=7,023,164
hermes    opus             2026-04-17T06:30  cached/input=65.87x  in=30,632     cached=2,017,731
opencode  claude-opus-4.7  2026-04-20T15:30  cached/input=61.18x  in=110,207    cached=6,742,235
opencode  claude-opus-4.7  2026-04-24T00:00  cached/input=60.39x  in=8,348      cached=504,124
```

Three million cached tokens against forty-five thousand input tokens. If you were charging per request based on the conventional definition — input is what the model sees, cached is the subset of input that hit the prefix cache — these numbers would be incoherent. The cache subset cannot be larger than the set it indexes. Either the schema is lying or the words mean something else.

## What the words actually mean (in Anthropic-land)

Anthropic's billing schema, which the pew digest faithfully copies row-for-row, defines the input-side counters as **disjoint buckets, not as a subset relationship**:

- `input_tokens` = uncached fresh input tokens (the prompt fragments the cache could not match)
- `cached_input_tokens` = cache-read input tokens (prefix bytes served from the cache without re-encoding)
- The total prompt the model actually saw is **`input_tokens + cached_input_tokens`**, not `input_tokens` alone.

OpenAI's schema before the Responses API consolidation, and most third-party dashboards built against the chat-completions schema, used the **subset** convention: `input_tokens` was the full prompt and `cached_input_tokens` was the discount-eligible portion of it. In that schema the ratio is bounded by 1.0 and "cache hit ratio" works as a metaphor.

Pew's row schema has no field that flags which convention each row is using. The `source` and `model` columns implicitly carry that information. The ledger is a collision of two billing dialects on the same five column names, and the cached-greater-than-input rows are the diagnostic exhaust of that collision.

## The family-level breakdown

Group the 1,703 rows by family — claude (843 rows), gpt-5 (765), other (57), gemini (37), gpt-4 (1) — and run the bad ratio anyway:

```
family    rows     input         cached         cache%   output
claude    843   2,104,411,221  5,184,970,881  246.39%   39,228,556
gpt5      765   1,439,469,729  1,295,048,192   89.97%    7,961,373
other      57      18,581,626     17,217,888   92.66%      413,201
gemini     37               0              0    0.00%       43,665
gpt4        1               0              0    0.00%           72
```

Two clean signals. Claude rows live in the disjoint-bucket convention: **246%** is the average ratio of cache-read to fresh-input tokens across 843 rows of usage, which translates to "for every uncached token the model saw, roughly 2.46 cached tokens came along with it." That is a **71%** cache-hit rate if you compute it the right way: `cached / (cached + input) = 5.18B / (5.18B + 2.10B) = 71.16%`. Phrased as a ratio bounded by 1.0, the claude family is reusing a little less than three quarters of every prompt.

The gpt-5 rows live in the subset convention: **89.97%** says that of the input tokens the model saw, just under 90% were cache-eligible. There is no recomputation needed. The 89.97% is the cache hit rate.

The two numbers are not comparable. Putting them in the same column of the same spreadsheet creates the headline-grabbing "claude has a 246% cache rate, gpt-5 has 90%, claude wins" reading, which is wrong twice over: once because the ratios are dimensionally different, and a second time because under the disjoint convention claude's *true* cache hit rate (71%) is *worse* than gpt-5's (90%).

## The per-source breakdown is louder

Group by `source` instead of by family and the dialect mismatch lights up cleanly:

```
source           rows   over_ratio_count   raw_cache%
openclaw          461            0           85.38%
opencode          355          292         1518.98%
ide-assistant-v   333            0            0.00%
claude-code       299            0           86.97%
hermes            191          110          176.86%
codex              64            0           96.40%
```

Two columns matter. `over_ratio_count` is the number of rows in that source where `cached > input`. `raw_cache%` is `sum(cached) / sum(input) * 100` aggregated across all rows in the source.

Read the table column by column:

- **ide-assistant-v, codex, openclaw, claude-code: zero over-ratio rows.** These four sources never emit a row where cached > input. They are either using the subset convention (ide-assistant-v, codex) or aggregating in a way that hides the disjointness (openclaw, claude-code). Their raw_cache% values — 0%, 96%, 85%, 87% — are interpretable as cache hit rates.
- **opencode: 292 of 355 rows (82.25%) have cached > input.** Almost every opencode row violates the subset convention. The aggregate ratio of **1518.98%** translates to "across the full opencode dataset, opencode emitted 15.19 cached tokens for every fresh input token." Re-anchored: the true opencode cache hit rate is `1518.98 / (1518.98 + 100) = 93.83%`. That is the highest cache hit rate in the entire dataset. Opencode is the most cache-efficient harness here, by a clear margin — but the naïve ratio reads "1500% rate" and looks like a bug.
- **hermes: 110 of 191 rows (57.59%) over-ratio.** Hermes is partially in the disjoint convention. Its raw 176.86% translates to a 63.88% true hit rate. Roughly a third of its rows behave one way and the rest behave the other way, which suggests hermes is multiplexing across multiple upstreams that do not normalize their token accounting before forwarding.

## The opencode row distribution is bimodal

Bucket all 355 opencode rows by ratio:

```
ratio == 0:           3 rows  (no cache reads at all)
ratio < 1:           60 rows  (subset-convention rows or cold-cache hours)
1 ≤ ratio < 5:       14 rows  (transition zone)
5 ≤ ratio < 20:     147 rows  (typical disjoint-convention hot-cache hours)
ratio ≥ 20:         131 rows  (highly-cached agent loops)
```

The 60 subset-style rows and the 278 disjoint-style rows live in the same source field. Same harness, same schema, same author of the upstream client. The bimodal distribution means **the convention is not even consistent within opencode** — likely because opencode is multiplexing across multiple LiteLLM-style proxies, some of which normalize the bucket convention upstream and some of which forward Anthropic's raw counters unchanged. The median opencode ratio of **17.22** sits firmly in the disjoint regime; the mean of **16.99** confirms there is no long tail dragging the median up. The p90 is **30.40** and the absolute max is **69.02** — the 69.02 row is the same `2026-04-23T17:30` opus-4.7 row that headlines the anomaly subset.

## The downstream metrics this corrupts

Once you accept that `cached_input_tokens / input_tokens` is not a hit rate, you have to audit every downstream metric that was secretly using it as one:

1. **Cost-per-token estimates.** If a dashboard multiplies `input_tokens * full_rate + cached_input_tokens * cached_rate`, claude-family rows are double-counting the prompt and gpt-5 rows are not. The dollar number is wrong by exactly the cached fraction.
2. **Throughput metrics.** "Tokens processed per hour" computed as `sum(input_tokens)` understates claude throughput by a factor of `1 + cache_ratio` — for the claude family that is 3.46×. The real prompt volume claude is processing per hour is 3.46× the naïve number.
3. **Cache effectiveness comparisons.** Any chart that puts claude and gpt-5 cache rates on the same y-axis is comparing apples to fruit-shaped objects. You cannot say "model X has better caching than model Y" without first normalizing both sides to the same convention.
4. **Per-session cache attribution.** The session-queue ledger does not carry cached/input columns, so the cache attribution has to be hour-rolled-up from the queue ledger — but if the convention varies hour-by-hour within a single source (as the opencode bimodality shows), the per-session cache estimate inherits that variance.

## The 64-row "cache off" subset

There are 64 rows in the entire 1,703-row ledger where `input_tokens > 0` and `cached_input_tokens == 0`. These are the rows where caching was either disabled, broken, or genuinely cold (first prompt of a session, no prefix to match). Their distribution by source:

- **codex: many of the 64 rows** — codex, like the rest of the OpenAI line, often runs without prompt caching enabled by default for short prompts.
- **gemini and claude-haiku-4-5-20251001 sub-rows** — gemini reports zero cached because the pew schema does not have a column for gemini's "implicit cache" billing line, so gemini cache is invisible in this dataset by construction.

The 64 number is the "no cache present" floor. Everything above it is caching something. The more interesting upper bound is that **3 opencode rows** had `cached == 0` despite being on the heavily-cached `claude-opus-4.7` model: those are the cold-start hours, where a fresh session began at the top of the hour and the prefix cache had not yet been primed.

## The "ratio" replaced by the right denominator

If you keep one formula from this post, keep this one:

```python
true_cache_hit_rate = cached_input_tokens / (input_tokens + cached_input_tokens)
```

Run it across all 1,703 rows:

- claude family: `5,184,970,881 / (2,104,411,221 + 5,184,970,881)` = **71.13%**
- gpt-5 family: `1,295,048,192 / (1,439,469,729 + 1,295,048,192)` = **47.36%**
- other: `17,217,888 / (18,581,626 + 17,217,888)` = **48.10%**

This is the apples-to-apples comparison. Claude reuses 71% of every prompt. GPT-5 reuses 47%. The 24-point gap is real and large. It says: in this workload, on this set of harnesses, with this set of repository contexts, claude's prefix caching is materially more effective. That is a finding. The naïve "246% vs 90%" framing was hiding it under a unit error.

## What this implies for the digest's other counters

If `cached_input_tokens` is in the disjoint convention for claude rows, then `total_tokens` should equal `input_tokens + cached_input_tokens + output_tokens + reasoning_output_tokens`. Spot-check the headline claude row:

```
input=22,374,827  cached=20,780,746  output=177,825  reasoning=0
sum = 22,374,827 + 20,780,746 + 177,825 + 0 = 43,333,398
total_tokens field: 43,333,398
```

Exact match. The total field is consistent with the disjoint convention. So when the queue ledger says `total_tokens: 43,333,398` it really does mean "the model saw 22.4M fresh input + 20.8M cached input + emitted 177K output." The naïve reading "the model processed 22.4M input tokens this hour" is off by **2×**. The naïve reading "the model processed 43.3M tokens this hour" is correct.

For gpt-5 rows the same arithmetic gives:

```
input=1,439,469,729 (already includes cached)  cached=1,295,048,192 (subset)
output=7,961,373
naïve sum: 1,439,469,729 + 1,295,048,192 + 7,961,373 = 2,742,479,294
```

Whereas the actual total emitted across all 765 gpt-5 rows is the sum of their per-row `total_tokens`, which under the subset convention should equal `input + output` (no double counting). If you sum the gpt-5 `total_tokens` field and get something close to **1.45B**, the convention is subset. If you get something close to **2.74B**, the convention is disjoint and the family-level distinction in the previous section is wrong. The fact that the family analysis worked at all suggests the convention is consistent within source columns, even if not across them.

## Operational consequences

The pew digest CLI emits aggregate cache numbers built on top of this ledger. Any downstream alert that fires on "cache rate dropped below 50%" needs to know *which convention* the source is in before comparing. Three concrete consequences:

1. **Don't put claude and gpt-5 cache rates on the same dashboard tile.** Use two tiles, label them with the convention, or normalize both to `cached / (cached + input)` before plotting.
2. **The opencode source needs a normalization pass.** The bimodality between subset rows (60) and disjoint rows (278) means any single aggregate is averaging across a unit mismatch. Fix it by rewriting the upstream LiteLLM-style middleware to coerce all incoming token counters into one convention before persisting.
3. **The "402 rows where cached > input" alarm should fire as a data-quality check, not a billing anomaly.** Right now those rows look broken to anyone reading the schema. They are not broken. They are correctly labeled in a convention the column name does not document.

## The single number

If you only carry one number out of this post:

> **23.61% of all rows in `~/.config/pew/queue.jsonl` have `cached_input_tokens > input_tokens`. Every one of those rows is an Anthropic-style disjoint-bucket emission. The maximum ratio is 69.02× on a single opencode hour. The "cache hit ratio" as conventionally written cannot be computed from this column pair without first asking which convention the row is in.**

The schema looks like one schema. It is two schemas wearing the same column names. The 402-row anomaly subset is the dataset telling you so, in the only language a JSONL file knows how to speak: by emitting numbers the column header forbids.

## A note on sample size

These numbers are from a single user's 1,703-row queue ledger snapshot taken on 2026-04-27. The claude-family aggregate is 843 rows; the gpt-5 aggregate is 765. The opencode aggregate is 355 rows of which 292 are over-ratio. The conclusions about convention dialect generalize from the schema design — they would hold on any ledger built from the same upstream sources — but the specific cache-rate percentages are workload-specific. A ledger drawn from a workload with no prompt-caching benefit (e.g., one-shot completions, no conversation reuse) would show 0% on every row regardless of convention. The fact that this workload shows 71% true-rate on claude says something about *this user's* prompt reuse pattern, not about claude's caching engine in the abstract.

The unit error is not workload-specific. The unit error is in the column header. It will be in any pew ledger drawn from the same set of sources, on any machine, for any user. Fixing it is a one-line normalization pass at ingest. Recognizing it is a one-row glance at the data:

```
SELECT source, COUNT(*) FROM queue WHERE cached_input_tokens > input_tokens GROUP BY source;
```

If that query returns any rows at all, you are in the disjoint convention and the ratio is not a ratio.
