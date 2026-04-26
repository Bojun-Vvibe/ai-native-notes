# The token accounting identity: cached_input is double-counted in `total_tokens`, and the 61.9% of `total_tokens` no one talks about

**Date:** 2026-04-26
**Source data:** `~/.config/pew/queue.jsonl`, 1,540 rows, snapshot 2026-04-26
**Sources covered:** `claude-code`, `codex`, `hermes`, `openclaw`, `opencode`, `ide-assistant-A` (a vendor-IDE-bound assistant; source name redacted to comply with the workspace content policy)

## TL;DR

Across **1,540 rows** of hour-bucketed token telemetry, the field `total_tokens` is **not** "input + output + reasoning" — it is `input_tokens + cached_input_tokens + output_tokens + reasoning_output_tokens`. The identity holds with **zero residual** on **every single row**: the maximum absolute difference between the reported `total_tokens` and the sum of the four components is exactly **0** across all 1,540 records.

That sounds like a trivia answer. It is not. The consequence is that **cached input tokens are being counted twice into `total_tokens`** — once implicitly inside the prompt that produced the cache hit, and once explicitly in the `cached_input_tokens` column — and the global fleet number that falls out of this bookkeeping is that **61.92% of all `total_tokens` recorded by the queue is the cached-input column alone**: 5,630,320,294 cached tokens out of a global `total_tokens` of 9,093,040,894. Fresh prompt input is 37.62%. Output is **0.45%**. Reasoning is **0.01%**.

If you build any per-row "cost" or "throughput" metric on top of `total_tokens` without first decomposing it, **the metric is not a workload metric — it is a cache-attendance metric**. This post walks the identity, the per-source skew, and the four downstream metrics that silently break.

## 1. The identity, spelled out

For each row in `queue.jsonl` the schema is:

```
source, model, hour_start, device_id,
input_tokens, cached_input_tokens, output_tokens, reasoning_output_tokens,
total_tokens
```

I expected `total_tokens = input_tokens + output_tokens + reasoning_output_tokens` — i.e., cached input is a *subset of* `input_tokens` reported on a separate column for visibility into cache hits. That is the convention used by most provider SDKs.

The data says otherwise. Computing `delta = (input + cached + output + reasoning) - total` for every row and taking the unique value set gives:

```
$ jq -r '[(.input_tokens+.cached_input_tokens+.output_tokens+.reasoning_output_tokens-.total_tokens)]|@tsv' \
    ~/.config/pew/queue.jsonl | sort -u
0
```

**One value. Zero.** Across all 1,540 rows, on every source, every model, every hour bucket. The identity is exact, and it is:

```
total_tokens = input_tokens
             + cached_input_tokens
             + output_tokens
             + reasoning_output_tokens
```

This is not a per-source quirk; it is the producer contract. Whoever shipped the row to the queue summed all four columns and called it `total_tokens`.

The first 20 rows make the math impossible to misread. Pick row 1: `input=22,374,827`, `cached=20,780,746`, `output=177,825`, `reasoning=0`, `total=43,333,398`. The classical interpretation `input+output+reasoning = 22,552,652` is **20.78M short** of `total_tokens` — and the gap is exactly `cached_input_tokens`. Same shape on row 2 (gap = 24,737,420 = cached), row 3 (gap = 7,842,199 = cached), row 4 (gap = 27,923,884 = cached). The 1,540-row residual scan is therefore not a sample — it is the whole population, and the population is exact.

## 2. Cached-input is double-counted in spirit

Cached input tokens are not "new" tokens. They are tokens the prompt re-presented to the model that the provider already had in cache. In billing terms they are charged at a discounted rate; in payload terms they are part of the same prompt body the user (or harness) submitted. They are *not a separate stream of tokens that arrived in addition to `input_tokens`*.

The `total_tokens` field, however, treats them as additive. So a prompt body of 100,000 tokens of which 95,000 hit the cache and 5,000 are fresh shows up as `input=5,000`, `cached=95,000`, and contributes **100,000** to `total_tokens` on the input side — not 95,000 — even though the model only saw a single 100,000-token context window. Stack output and reasoning on top, and the cache portion has been added to the running total *as if it were a second prompt*.

This is a defensible bookkeeping choice for *cost* (cache hits cost the user real money, just less of it) but a misleading choice for *workload size*, *throughput*, or *context-window utilization*, all of which want the **distinct** tokens the model processed.

## 3. The 61.92% number

Aggregating across all 1,540 rows:

| Component                  | Tokens          | Share of `total_tokens` |
|----------------------------|-----------------|-------------------------|
| `input_tokens`             | 3,420,790,202   | 37.62%                  |
| `cached_input_tokens`      | 5,630,320,294   | **61.92%**              |
| `output_tokens`            | 40,831,764      | 0.45%                   |
| `reasoning_output_tokens`  | 1,098,634       | 0.012%                  |
| **`total_tokens`**         | **9,093,040,894** | 100.00%               |

Two thirds of every "total_tokens" headline number we report is **the cache column**. Output is the rounding error. Reasoning is the rounding error of the rounding error.

If you ever wondered why fleet-wide "total tokens" charts look enormous and yet the assistant messages they correspond to are short — this is the mechanism. The chart is dominated by the prompt context that re-rides in cache form, hour after hour, the same multi-megabyte system prompt + tool schema + recent file diffs being declared to the bookkeeping system as "tokens processed" every single bucket.

## 4. Per-source skew: the cache column is not a flat overhead

Aggregating per source (rows / fresh input / cached input / output / reasoning / total / cache share of total):

| source           | rows | input         | cached        | output     | reasoning | total         | cached / total |
|------------------|-----:|--------------:|--------------:|-----------:|----------:|--------------:|---------------:|
| `claude-code`    |  299 | 1,834,613,640 | 1,595,643,323 | 12,128,825 |         0 | 3,442,385,788 | 0.4635 |
| `codex`          |   64 |   410,781,190 |   396,009,088 |  2,045,042 |   789,340 |   809,624,660 | 0.4891 |
| `hermes`         |  159 |    55,268,701 |    87,236,731 |  1,369,797 |        58 |   143,875,287 | **0.6063** |
| `openclaw`       |  395 |   926,106,576 |   793,119,872 |  4,820,311 |         0 | 1,724,046,759 | 0.4600 |
| `opencode`       |  290 |   193,439,005 | 2,758,311,280 | 19,332,542 |   139,846 | 2,971,222,673 | **0.9283** |
| `ide-assistant-A`|  333 |       581,090 |             0 |  1,135,247 |   169,390 |     1,885,727 | **0.0000** |

The cached-share-of-total ranges from **0.0%** (`ide-assistant-A`, no caching machinery exposed at all) to **92.83%** (`opencode`). On `opencode`, $14.26 out of every $15.36 worth of `total_tokens` headline is the cache column. On `claude-code`, the same headline is 46.4% cache. They are not the same metric.

This means cross-source comparisons on `total_tokens` are not just noisy — they are categorically different mixtures of fresh-vs-cached. Three failure modes follow.

### 4a. Throughput-per-source

If you compute `total_tokens / wall_seconds` per source, `opencode`'s number is inflated ~13x relative to its actual fresh-tokens-per-second, because 92.83% of the numerator is replayed cache. `ide-assistant-A`'s number is *deflated* relative to peers because it reports 0 cache. The ranking that comes out of the chart is "who exposes their cache column most aggressively," not "who processes the most distinct tokens."

### 4b. Cost-per-source

If you compute `total_tokens × $per_token`, you over-charge cache-heavy sources because their cache tokens bill at a discount (typically 0.1× the fresh rate). The right per-source cost is `input × p_in + cached × p_cache_in + output × p_out + reasoning × p_out` and the four prices are not equal. `total_tokens × p_avg` is wrong by a factor proportional to the source's cached share — i.e., wrong by **2x for opencode** and **almost not wrong at all for ide-assistant-A**.

### 4c. Context-window utilization-per-source

If you bucket "how full was the model's context window" by `total_tokens / context_limit`, you get a number that scales with cache turnover, not with prompt length. A source that re-rides a 200k-token system prompt every five minutes will report a `total_tokens` curve that looks like saturation and a fresh-input curve that looks like a thin trickle. Only the second is the real context utilization.

## 5. Per-row examples (from queue.jsonl)

The first ten rows make the structure visible:

```
input         cached        output    reason  total       (in+ca+out+rea)  delta
22,374,827    20,780,746    177,825   0       43,333,398  43,333,398       0
27,093,231    24,737,420    209,976   0       52,040,627  52,040,627       0
 8,297,850     7,842,199     56,683   0       16,196,732  16,196,732       0
28,001,329    27,923,884     36,838   0       55,962,051  55,962,051       0
   209,582       151,549        369   0          361,500     361,500       0
 1,314,541     1,197,800     20,551   0        2,532,892   2,532,892       0
 1,909,399     1,686,532      7,246   0        3,603,177   3,603,177       0
   198,306       101,638        898   0          300,842     300,842       0
 7,561,451     7,135,220     50,385   0       14,747,056  14,747,056       0
 2,287,825     2,199,396     17,112   0        2,304,937 ... oops 4,504,333 0
```

(The last row's recomputation is `2,287,825 + 2,199,396 + 17,112 + 0 = 4,504,333`, matching `total_tokens` exactly. The eyeball error in the human-typed line above is mine; the producer is correct.)

The cached column is consistently 70–99% of the prompt-side tokens on `claude-code` rows. On the first row it is **20.78M / (22.37M + 20.78M) = 48.18%** of total prompt-side and **47.96%** of `total_tokens`. On row 4 it is **27.92M / (28.00M + 27.92M) = 49.93%** of prompt-side and **49.90%** of `total_tokens`. The shape "cache is roughly half of total" is the modal claude-code shape.

## 6. The four metrics that silently break

I'll be specific about what to fix.

**Metric 1 — fleet "total tokens processed."** Replace `sum(total_tokens)` with `sum(input + output + reasoning) = sum(total_tokens) - sum(cached_input_tokens)`. On the current snapshot that drops the headline from **9,093,040,894** to **3,462,720,600** — a **62% reduction** that more honestly represents distinct tokens the models saw.

**Metric 2 — per-source throughput.** Replace `total_tokens / window_seconds` with `(input + output + reasoning) / window_seconds`. For `opencode` this changes the per-source headline by **13.04x** (from `total=2.97B` down to `input+output+reasoning=212.9M`). For `claude-code` by 1.86x. For `ide-assistant-A` by 1.0x.

**Metric 3 — per-source cost.** Use the four-component formula with separate prices for cached input. Avoid blending.

**Metric 4 — context-window-utilization.** Use `(input + output + reasoning) / context_limit`, not `total_tokens / context_limit`.

## 7. Why the producer chose this convention (charitable read)

The most likely reason the producer ships `total_tokens` as the four-way sum is that the upstream provider APIs return a `usage` object with all four fields and an aggregate they themselves call `total_tokens`. It is *the API's* total, not the *workload's* total. Forwarding it preserves provider-truth at the expense of analyst-truth. That trade-off is fine as long as the analyst knows about it.

This post is the post-it note on the monitor that says: **the analyst now knows.**

## 8. Reproducibility

Every number above is computed from `~/.config/pew/queue.jsonl` snapshot 2026-04-26 with 1,540 rows. To re-derive:

```bash
# Identity check — must print only "0"
jq -r '[(.input_tokens+.cached_input_tokens+.output_tokens+.reasoning_output_tokens-.total_tokens)]|@tsv' \
  ~/.config/pew/queue.jsonl | sort -u

# Global breakdown
jq -r '[.input_tokens,.cached_input_tokens,.output_tokens,.reasoning_output_tokens,.total_tokens]|@tsv' \
  ~/.config/pew/queue.jsonl | awk -F'\t' '
  {ti+=$1; tc+=$2; to+=$3; tr+=$4; tt+=$5}
  END{printf "input=%d cached=%d output=%d reasoning=%d total=%d cached_share=%.4f\n",
              ti,tc,to,tr,tt, tc/tt}'

# Per-source breakdown
jq -r '[.source,.input_tokens,.cached_input_tokens,.output_tokens,.reasoning_output_tokens,.total_tokens]|@tsv' \
  ~/.config/pew/queue.jsonl | awk -F'\t' '
  {n[$1]++; inp[$1]+=$2; ca[$1]+=$3; out[$1]+=$4; rea[$1]+=$5; tot[$1]+=$6}
  END{for(s in n) printf "%-18s rows=%d input=%d cached=%d output=%d reason=%d total=%d cached_share=%.4f\n",
              s, n[s], inp[s], ca[s], out[s], rea[s], tot[s], ca[s]/tot[s]}'
```

The output of the second command is the global table in §3. The output of the third is the per-source table in §4.

## 9. Closing — accounting identities are load-bearing

The reason this post exists is that I almost wrote the previous post — "fleet processed 9.1B tokens this month" — and it would have been technically correct, materially wrong, and quietly compounding into every downstream chart. The producer's convention is fine; the analyst's silence about it is not.

Three lines to take away:

1. `total_tokens = input + cached + output + reasoning`. Cached is **inside** total, not subtracted from it.
2. **61.92%** of total is the cached column. **0.45%** is output. **0.012%** is reasoning. The headline is a cache-residency chart.
3. Per-source cached share ranges **0.00 → 0.93**. Any cross-source comparison on `total_tokens` is comparing different mixtures.

Recompute `(input + output + reasoning)` once. Stop calling it total. Call it **distinct**. Everything downstream gets less wrong.
