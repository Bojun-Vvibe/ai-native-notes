# The Monster-Prompt Index: Input-Token HHI and the 71% Top-3 Row Share on `ide-assistant-A`

*Generated from `pew-insights source-input-token-top-row-share --json` at 2026-04-26T21:27:39Z. Window: full corpus. Total input tokens across 6 sources: 3,458,942,980. Default `topK = 3`, default `minRows = 3`.*

If you are paying for inference at the prompt-token grain, the *shape* of your input mass matters as much as its sum. A million input tokens spread evenly across a thousand modest prompts is a different cost geometry than a million input tokens delivered as three monster requests of 333,000 tokens each. The first profile rewards prompt caching, batching, and connection reuse. The second profile burns into a long-context provisioned tier and creates retry blast radius if any one of the three monsters fails — a single failed call costs a third of the total spend.

The `source-input-token-top-row-share` lens in `pew-insights` measures exactly this. It computes, per source, the share of total `input_tokens` mass concentrated in the K largest individual queue rows (default `K=3`), plus the Herfindahl-Hirschman Index (HHI) of per-row input mass for that source. HHI here is `Σ (rowInputTokens / totalInputTokens)²` summed over all the source's rows: 0 if input is uniformly spread across infinitely many equal rows, 1 if a single row carries the entire mass.

This metric is orthogonal to almost everything else in the doctrine — it is per-row, per-source, on the **input** column specifically, and it answers one specific question: how much of a tool's lifetime input volume is dominated by a small number of monster prompts?

## What the corpus actually shows

Pulled at `2026-04-26T21:27:39.159Z`:

| Source | rowsConsidered | inputSum (tokens) | top1Share | top3Share | HHI |
|---|---:|---:|---:|---:|---:|
| `ide-assistant-A` | 6 | 581,090 | 29.62% | 71.36% | 0.2047 |
| `codex` | 64 | 410,781,190 | 7.21% | 19.88% | 0.0351 |
| `hermes` | 165 | 55,482,039 | 5.63% | 12.59% | 0.0191 |
| `opencode` | 314 | 208,326,141 | 5.74% | 10.66% | 0.0108 |
| `claude-code` | 299 | 1,834,613,640 | 3.04% | 7.69% | 0.0105 |
| `openclaw` | 420 | 949,158,880 | 2.47% | 6.69% | 0.0056 |

The corpus has 3,458,942,980 total input tokens. Zero rows were dropped for invalid hour-start, zero for source filter, zero for all-zero input. The metric is computed on every positive-input row in the queue. There is no smoothing, no winsorisation, no minimum-magnitude floor — every row votes.

There are five readings worth pulling out.

## Reading 1: the small-N source has the highest concentration, and it is not noise

`ide-assistant-A` reports `rowsConsidered = 6`. That is small, and small-N concentration metrics deserve scrutiny — with only 6 rows, a single monster row of average size will produce a top1Share of ~16.7% just by being one of six. So 29.62% top1 and 71.36% top3 are *above* the uniform baseline, but only modestly: a perfectly uniform 6-row source would have top3Share of 50%. The fact that this source's top3Share is 71.36% means its top three rows together carry 1.43× the share they would carry under a uniform distribution. Not extreme, but not nothing.

The HHI of 0.2047 is the more telling number. For 6 uniform rows, HHI would be `6 × (1/6)² = 1/6 ≈ 0.1667`. The observed 0.2047 says the source is concentrated *above* that floor, but not dramatically so. The HHI you'd see if a single row carried half the mass and the other 5 split the rest evenly would be `0.5² + 5 × 0.1² = 0.25 + 0.05 = 0.30`. So `ide-assistant-A` sits between "uniform 6" (0.167) and "one row carrying half" (0.30).

Of note: the same JSON also reports `rowsZeroInput = 327` for this source. That is, 327 of the source's 333 total queue rows had `input_tokens = 0`, and the lens by construction excludes them. So the 6 considered rows are the *long tail* of an otherwise empty-input distribution — every one of those 6 is presumably a real interactive prompt, while the 327 zero-input rows are most likely keep-alive pings, autocomplete probes, or telemetry beacons. Reading top1Share = 29.62% on 6 surviving rows out of 333 is therefore not a noise warning — it is a *signal* that this source's actual prompt traffic is sparse and clustered, with three prompts (out of 6 real ones) dominating 71% of the mass.

## Reading 2: `codex` is the heavy-tail heavy source

Among the four heavy-traffic sources (`opencode`, `openclaw`, `claude-code`, `codex`), `codex` is the standout: 64 rows, top1Share 7.21%, top3Share 19.88%, HHI 0.0351. These are 2× to 4× the corresponding numbers for the other three.

Why? The `codex` mass total is 410.78M input tokens spread over only 64 rows. Mean input per row is ~6.42M. The top-1 row is 29.63M tokens, which is ~4.6× the mean. The top-3 sum is 81.66M, which is ~3.2× what 3 mean-sized rows would contribute. So `codex` has both a lower row count *and* a heavier-tailed individual-row distribution.

Compare to `claude-code`: 299 rows, 1.83B input tokens, mean ~6.14M (close to `codex`'s mean), but top1Share only 3.04% and HHI only 0.0105. Same per-row mean, but `claude-code`'s rows cluster much more tightly around the mean — its top-1 row is only ~9× the mean instead of `codex`'s ~46×. (Top-1 of `claude-code` = 55.74M; mean = 6.14M; ratio ~9.1.) Both sources can plausibly be running the same model family. The difference is operating posture: one is a long-context single-shot tool, the other is a turn-based session tool that breaks even big jobs into a sequence of similarly-sized turns.

## Reading 3: the four-decimal HHI separation is real

HHI values across the four heavy sources are `codex` 0.0351, `hermes` 0.0191, `opencode` 0.0108, `claude-code` 0.0105, `openclaw` 0.0056. That looks like a tight band of small numbers, but it is in fact a 6× spread (0.0351 / 0.0056 = 6.27).

The rule of thumb for HHI on positive shares: if all rows were uniformly equal, HHI would be `1/N`. So `openclaw`'s 0.0056 is very close to its uniform floor of `1/420 = 0.00238`, but a factor of ~2.4 above it. `claude-code`'s 0.0105 is ~3.1× its uniform floor of `1/299 = 0.00334`. `opencode`'s 0.0108 is ~3.4× its uniform floor of `1/314 = 0.00318`. `codex`'s 0.0351 is ~22× its uniform floor of `1/64 = 0.01563`.

So measured *as a multiple of the uniform-N baseline*, the ranking flips: `codex` is by far the most concentrated, `openclaw` the least, with the other three forming a cluster at 3× to 3.4× their respective floors. The HHI/uniform-floor ratio is the calibrated read: `codex` is exceptional, the rest are approximately Pareto-with-light-tail.

## Reading 4: `top1Share` and `topKShare` are not redundant — the gap matters

The lens reports `top1Share` and `topKShare` separately. The ratio `topKShare / (K × top1Share)` says how the next-largest rows compare to the single largest. If the top K rows are all about the same size, that ratio approaches 1; if the top row dominates and the next K-1 are small, it approaches `1/K`.

For our six sources:

- `ide-assistant-A`: 0.7136 / (3 × 0.2962) = 0.803 — top 3 rows are roughly comparable to each other
- `codex`: 0.1988 / (3 × 0.0721) = 0.919 — extremely close to flat across top 3
- `hermes`: 0.1259 / (3 × 0.0563) = 0.745
- `opencode`: 0.1066 / (3 × 0.0574) = 0.619 — top 1 is meaningfully heavier than rows 2 and 3
- `claude-code`: 0.0769 / (3 × 0.0304) = 0.844
- `openclaw`: 0.0669 / (3 × 0.0247) = 0.903

The high values for `codex` (0.919) and `openclaw` (0.903) say that when you look at each tool's three biggest input rows, those three are roughly the same size. There is not one dominant monster — there are three peers. The lower value for `opencode` (0.619) says the largest row is meaningfully bigger than rows 2 and 3, which together carry less than the top.

This is a useful distinction for cost engineering. A "top-3 are peers" source is what you get when the same kind of large prompt repeats. A "top-1 dominates" source is what you get when one specific request is genuinely a different shape.

## Reading 5: the input-mass-to-row-count elasticity

If you cross the input-mass column against the row count, an interesting elasticity falls out. Sort sources by total input mass and tabulate alongside row count:

| Source | inputSum (M tokens) | rowsConsidered | mean per row (M) |
|---|---:|---:|---:|
| `claude-code` | 1,834.61 | 299 | 6.14 |
| `openclaw` | 949.16 | 420 | 2.26 |
| `codex` | 410.78 | 64 | 6.42 |
| `opencode` | 208.33 | 314 | 0.66 |
| `hermes` | 55.48 | 165 | 0.34 |
| `ide-assistant-A` | 0.58 | 6 | 0.097 |

Two sources share a per-row mean near 6.2M (`claude-code` and `codex`) but differ 4.7× in row count and 4.5× in total input mass. Two sources share row counts in the 300s (`claude-code` and `opencode`) but differ 9.3× in mean per-row input. This is the per-row variability you would never see from the aggregate `digest` view.

`openclaw` is the high-row-count, moderate-per-row source: 420 rows at 2.26M each. Its low HHI (0.0056) and low top1Share (2.47%) are consistent — many similar-sized rows, no single row dominating.

`opencode` and `hermes` are both small-per-row sources: under 1M tokens of input per row on average. These are the chat-style or polling-style sources. Their HHI (0.0108 and 0.0191) sits around the "lightly Pareto" floor.

`ide-assistant-A` is the ultra-small per-row source: under 100K mean. With 6 surviving positive-input rows, its biggest row at 172,138 tokens carries 29.62% of its (admittedly tiny) total input mass.

## Why this index is orthogonal to the rest of the doctrine

This corpus already has posts on:

- the **decile distribution of input tokens** (global rank-and-bucket on input column), which is a corpus-wide partition that does not break out per source;
- the **single-day mass concentration per source** (per-source, but on `total_tokens`, daily axis);
- the **cost-class mix per source** (categorical bucketing of rows into small/medium/large by `total_tokens`, not concentration on one column);
- the **output-tokens-per-row percentiles per source** (output column, percentile shape rather than HHI);
- the **cold/warm row ratio per source** (cache-state partition, ignores magnitude);
- the **Pareto tail share per source** (Pareto on hour-of-week buckets, not individual rows).

`source-input-token-top-row-share` is the *per-source HHI on the input column at the row grain*. None of the existing lenses produces that scalar. The closest is the global input-token decile distribution, but a global decile lens cannot tell you which source contributes the rows in the top decile — it only tells you that the top decile exists and how much mass it carries.

The orthogonality has a sharp consequence: a source can pass every existing test and still flunk this one. A source whose `output-tokens-per-row` percentiles look unremarkable, whose daily-Gini is normal, whose `cold/warm` ratio is balanced, can still be a "monster prompt" source if its lifetime input volume is dominated by a few very large requests. This lens is the one that catches that.

## Pricing and operational consequences

Three concrete reasons this index is operationally useful:

**1. Budget concentration risk.** A source with HHI of 0.2047 has 20% of its cost geometry sitting in `Σ (row_share)²`. In plain English: if you lost the ability to make the top-1 input request at parity cost — say, the long-context tier got 2× more expensive — this source's expected cost would jump roughly proportional to its `top1Share`. For `ide-assistant-A`, that is 29.62% of its input bill. For `codex`, 7.21%. For `openclaw`, 2.47%. The tail risk on a per-source basis scales with `top1Share`, not with row count.

**2. Cache invalidation blast radius.** A source whose top-3 rows are all peers (high `topKShare / (K × top1Share)` ratio) is a source that is repeatedly issuing a similarly-shaped large prompt. That is exactly the regime where prompt caching helps the most — the deltas between the three peers are likely small, and a cached prefix can absorb most of the cost. `codex` (ratio 0.919) and `openclaw` (ratio 0.903) are the prime cache-warming candidates by this metric. `opencode` (ratio 0.619) is not — its top row is a one-off shape, not a repeated peer of rows 2 and 3.

**3. Retry blast radius.** If the top-1 row of a source carries 30% of that source's input mass and it retries, the source's cost spikes hard. The source-level `top1Share` is the worst-case fraction by which a single retry can inflate the source's input bill. `ide-assistant-A` at 29.62% is the highest-risk source by this measure. `openclaw` at 2.47% is the lowest. The four-decimal HHI band hides this but `top1Share` shows it directly.

## What this lens does not see

Three honest limitations:

- It is a snapshot. Any single monster prompt that is older than the queue's compaction horizon falls off and the index updates. A source that *used to* have a heavy top-1 row and no longer does will read clean. (Counterfactual: if the queue retained 365 days, the picture would shift.)
- It is per-row at the queue grain, not per-session. A 50-turn session whose 50 turns are each 20K input tokens contributes 50 medium rows to the index, not one monster session row. The lens is intentionally row-axis only.
- It is symmetric across the input column only. A source whose **output** is dominated by a few monster generations would not appear in this lens at all. The output-axis analog already exists (`source-output-tokens-per-row-percentiles`) and is reported separately.

## The honest reading

Out of the six sources in the corpus, the input-mass concentration story is:

- **`codex`** is the heavy source with the heavy tail. 64 rows, mean 6.42M, top-1 row 4.6× the mean, top-3 essentially equal-sized peers. HHI 0.0351 — 22× the uniform-N floor.
- **`ide-assistant-A`** is the small-N source with deceptively high concentration. The top1Share of 29.62% is real but reflects only 6 surviving positive-input rows out of 333 total (the other 327 are zero-input keep-alive traffic).
- **`claude-code`**, **`opencode`**, and **`openclaw`** are the "spread it out" sources. HHI 0.0056 to 0.0108. Their input mass sits in many similar-sized rows; no single row dominates.
- **`hermes`** sits in between with HHI 0.0191 — moderate concentration, small per-row mean.

Generated at `2026-04-26T21:27:39.159Z` against the live `~/.config/pew/queue.jsonl`. Total input tokens 3,458,942,980. Six sources. Six per-row HHIs. Zero invalid-row drops, zero filter drops. The numbers are reproducible from `pew-insights source-input-token-top-row-share --json` at that moment.

The index is not a routing decision on its own. It is a slice that sits next to the per-source cache-hit ratio, the per-source output/input ratio, and the per-source decile shape. Together, those four scalars describe a source's *cost surface* with much more fidelity than any single global metric ever will. The monster-prompt index is the one of those four that catches the case where a tool's lifetime cost is dominated by a small number of very large requests — and in this corpus, today, that case is `codex`.
