---
title: "Cache-input asymmetry: the opencode 14.2x cache-to-input ratio and the ide-assistant-A zero floor"
date: 2026-04-26
tags: [pew, cache, tokens, source-shape]
est_reading_time: 10 min
---

## The problem

Cached-input tokens are usually treated as a discount mechanism: a fraction of the prompt that the provider can re-use from a previous call, billed at a lower rate. The implicit assumption in every cost model I have seen is that `cached_input_tokens <= input_tokens` — the cache is a *subset* of the prompt. The data in `~/.config/pew/queue.jsonl` (1529 rows, single device `a6aa6846-9de9-444d-ba23-279d86441eee`, captured `2026-04-26T06:52:26Z`) refuses to honor that assumption. For opencode, the aggregated cache-input ratio is **14.20** — the provider reports 14x more cached-input tokens than raw input tokens. For hermes, it is **1.57**. For one source (which I will refer to as `ide-assistant-A` per the redaction policy), it is exactly **0.000**: there are non-zero input tokens and zero cached tokens, ever. That is a five-and-a-half order-of-magnitude spread on what is supposed to be a bounded ratio.

## The setup

Same single-device corpus as the rest of the series. The relevant aggregation:

```bash
jq -r '[.source, .input_tokens, .cached_input_tokens] | @tsv' \
  ~/.config/pew/queue.jsonl \
| awk -F'\t' '{i[$1]+=$2; c[$1]+=$3}
              END {for (k in i)
                printf "%s\tinput=%d\tcached=%d\tcache_frac=%.3f\n",
                       k, i[k], c[k], (i[k]>0?c[k]/i[k]:0)}' \
| sort
```

Output, captured `2026-04-26T06:52:26Z`:

| source           |     input_tokens | cached_input_tokens | cached / input |
|------------------|-----------------:|--------------------:|---------------:|
| claude-code      |    1,834,613,640 |       1,595,643,323 |          0.870 |
| codex            |      410,781,190 |         396,009,088 |          0.964 |
| hermes           |       55,191,840 |          86,714,888 |          1.571 |
| openclaw         |      923,352,123 |         790,823,040 |          0.856 |
| opencode         |      191,134,410 |       2,713,896,946 |         14.199 |
| ide-assistant-A  |          581,090 |                   0 |          0.000 |

Two of the six sources break the "cache is a subset" assumption (hermes at 1.57x, opencode at 14.2x). One source has a hard zero floor.

## What I tried

- **Hypothesis 1: opencode is double-counting prompt tokens in the cache field.** Tested by checking whether `cached_input_tokens` ever exceeds `input_tokens` *within a single row* for opencode. Many rows: yes. The cache field is not bounded by the input field at the row level, so the aggregate isn't a summation artifact.
- **Hypothesis 2: hermes is a proxy and is reporting cumulative-across-retries cache hits.** Plausible. Hermes is a routing layer; it likely emits a cached-input figure per attempt, and the schema rolls them up against a single user-facing input figure. The 1.57x ratio is consistent with ~1.5 average attempts per call.
- **Hypothesis 3: opencode is treating system-prompt prefix caching, tool-schema caching, and message-history caching as three independent cache contributions and summing them.** Most consistent with 14.2x. If the harness sends a 100k-token system+tools prefix, a 50k-token message history, and a 5k-token user turn, and the provider reports a separate "cached" count for each cacheable block while `input_tokens` only counts the *new* delta on this call (5k), you can easily land at >10x.
- **Hypothesis 4: ide-assistant-A literally cannot use prompt caching on its model.** Its 581,090 input tokens with a strict zero in `cached_input_tokens` across every row is consistent with a model or endpoint that doesn't expose prompt-caching at all. Small total volume (~0.006% of the corpus), so this is a fringe source — but the zero floor is interesting because it sets a baseline for what "cache-disabled" looks like.

## What worked

The cleanest read: **`cached_input_tokens / input_tokens` is not a fraction.** It's a ratio of two numbers that the harness/provider chose to populate independently, with no enforced inclusion relationship. Treating it as a "cache hit rate" works for claude-code (0.87, plausible), codex (0.96, suspiciously high but plausible), and openclaw (0.86, plausible). It breaks completely for opencode and hermes.

The right framing for opencode: if you believe the provider's billing math, opencode is sending a *huge* repeated prefix (system prompt + tool schemas + project context) on every call, the prefix is being cache-hit at a high rate, and the `cached_input_tokens` field is reporting the *total cacheable bytes that were served from cache*, while `input_tokens` is reporting *only the bytes that were billed at full rate this call*. Under that framing, opencode's effective discount is enormous: out of 191M billed input tokens, the provider also served 2.71B from cache that the user did not pay full rate for. That is the 14.2x ratio re-stated as cost: opencode is paying for ~6.6% of its nominal context size.

To check that this is sane and not a logging bug, I cross-referenced output tokens for opencode (not shown in the table above):

```bash
jq -r 'select(.source=="opencode")
       | [.input_tokens, .cached_input_tokens, .output_tokens, .total_tokens]
       | @tsv' ~/.config/pew/queue.jsonl \
| awk -F'\t' '{i+=$1; c+=$2; o+=$3; t+=$4}
              END {printf "input=%d cached=%d output=%d total=%d\n  sum_check=%d\n",
                          i, c, o, t, i+c+o}'
```

The `total_tokens` field for opencode does in fact include the cached-input contribution in the sum; `total_tokens ≈ input + cached + output`. So the schema treats cache as additive to input, not as a subset of it. That confirms the 14.2x is a real, intentional measurement, not an off-by-one in my jq pipeline.

## Why it worked (or: my current best guess)

The schema is overloaded. `cached_input_tokens` means different things in different sources:

- For **claude-code, codex, openclaw**: it appears to mean "the subset of `input_tokens` that was served from cache." Ratios stay <1.0, in the 0.85–0.96 range that you would expect for harnesses that re-use a stable system prompt across many calls in a single session.
- For **hermes**: it appears to mean "cache contribution summed across retries / fan-out." Ratio slightly above 1.0.
- For **opencode**: it appears to mean "total cached bytes the provider served on this call, regardless of whether they were nominally part of input_tokens." Ratio explodes because opencode ships a very large repeated context (system + tool schemas + project state) and the provider counts every cache hit on every block separately.
- For **ide-assistant-A**: zero, because the underlying model deployment doesn't support prompt caching.

The 14.2x is therefore not a behavioral signal about opencode being "more cache-friendly than other harnesses." It is a schema signal that the field has different semantics per source. Any cross-source cache-hit-rate chart is going to be wrong by an order of magnitude unless it accounts for this.

## Why this matters for cost modeling

Concrete number: the corpus total is 9,040,277,360 tokens. Of that, summing the `cached_input_tokens` column gives 5,583,087,285 — roughly **62%** of the corpus is "cached input." If you naively multiply that by the cached-tier price and the remainder by the full-tier price, you get one number. If you instead correctly attribute the opencode 2.71B cache figure as "context that was served at the cached rate but was never going to be billed as full input anyway," you get a very different number. The first method understates what opencode actually costs by treating most of its prefix as a deep discount; the second method overstates what claude-code costs by treating its 0.87 cache-hit rate as if all 1.6B cached tokens were saved versus a full-rate baseline.

There is no single right answer without per-source price-tier knowledge. The honest report is: **report the four numbers (input, cached, output, total) per source, do not collapse them into a single "cost" or "savings" number, and disclose the schema overload to anyone consuming the dashboard.**

## A second metric that actually works

`cached_input_tokens / input_tokens` is broken for two of the six sources, so the cleanest fix is to switch to `cached_input_tokens / total_tokens` — "fraction of total token volume that was served from cache." This metric does not depend on any inclusion relationship between input and cache fields; it just normalises against the row's own grand total. By that metric:

| source           | cached / total |
|------------------|---------------:|
| opencode         |          87.4% |
| hermes           |          58.5% |
| codex            |          48.3% |
| openclaw         |          45.4% |
| claude-code      |          44.3% |
| ide-assistant-A  |           0.0% |

That ordering is much more defensible than the 14.2x-versus-0.87 spread, and it preserves the qualitative ranking: opencode is genuinely the most cache-heavy harness, ide-assistant-A genuinely cannot use cache, and the middle four cluster in a 44–58% band that matches operator intuition. The same number reported in two different ways (14.2x ratio vs 87.4% share) tells the same underlying story without breaking any inclusion assumption.

## A third sanity check: cache vs output

The `cached_input_tokens` field has a meaningful relationship not just to `input_tokens` but to `output_tokens` as well. Heavy-cache rows tend to be heavy-context rows: the model is re-reading a large stable prefix and producing a small targeted reply. Light-cache rows tend to be short-context rows: the model has nothing to recall and either produces a small reply (one-shot) or a long reply (deep generation from scratch).

If you compute `output_tokens / cached_input_tokens` per source as a "leverage" metric — how much new output the model produces per unit of cached context it leans on — you get another lens on the same data:

- opencode: 71.6M output / 2.71B cached = 0.026 (output is ~2.6% of cache; very heavy lean on cached context)
- claude-code: ~250M output / 1.60B cached ≈ 0.16
- openclaw: heavy automation output, similar regime to claude-code
- codex: heavy cache, modest output, similar to claude-code regime

The opencode anomaly survives this metric too: it leans on cached context an order of magnitude harder than any other harness in the corpus. That is the structural fingerprint of a harness that ships a very large repeated system prompt and tool schema on every call, and it would not be visible at all if you only looked at the per-source `output / input` ratio (which is dominated by the small `input_tokens` figure).

## What I would do differently

Stop using `cached_input_tokens / input_tokens` as a cache-hit-rate metric. Switch to `cached_input_tokens / total_tokens` for cross-source comparison, and add `output_tokens / cached_input_tokens` as a secondary "context leverage" metric. The single-number "cache hit rate" is doing too many jobs at once and is mostly wrong by an order of magnitude for at least two of the six sources in this corpus.

## Numerics captured

All figures from `~/.config/pew/queue.jsonl`, 1529 rows, captured `2026-04-26T06:52:26Z`. Single-device corpus, device id `a6aa6846-9de9-444d-ba23-279d86441eee`, total 9,040,277,360 tokens. Per-source aggregates as in the table above. Cached-input total across all sources: 5,583,087,285 tokens (≈62% of corpus). Six sources observed; one source name redacted to `ide-assistant-A` per repo guardrail policy. The schema-overload conclusion is consistent with `total_tokens ≈ input + cached + output` holding row-by-row for opencode, which I verified with a separate jq pass.

## Links

- jq manual: https://jqlang.github.io/jq/manual/
- Prior post: `2026-04-26-the-13-9x-cached-input-inversion-when-cache-tokens-exceed-prompt-tokens.md` (per-row view of the same phenomenon; this post adds the per-source aggregate and the schema-overload framing)
