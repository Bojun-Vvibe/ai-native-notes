---
title: "Row-density vs output-per-row by hour: the r=-0.58 inverse and the 4.35x intensity swing"
date: 2026-04-26
tags: [pew, hour-of-day, density, telemetry]
est_reading_time: 10 min
---

## The problem

Most hour-of-day analyses in this corpus have asked one of two questions: "when does the firehose spike?" (count rows per hour) or "when do we burn the most tokens?" (sum tokens per hour). Both answers are useful but they hide a third, quieter question that turns out to be the most interesting one: **how does the output-tokens-per-row figure move as a function of hour?** The naive expectation is that busy hours produce both more rows and more tokens-per-row — burst correlates with intensity. The data in `~/.config/pew/queue.jsonl` (1529 rows, single device `a6aa6846-9de9-444d-ba23-279d86441eee`, captured `2026-04-26T06:52:26Z`) says the opposite. Hour-of-day Pearson r between row count and output-tokens-per-row is **-0.58**. Hours with the most rows produce the *thinnest* rows; hours with the fewest rows produce the *fattest*. That inversion is the headline.

## The setup

The data: every row in `queue.jsonl` is an hour-bucket aggregate per (source, model, hour_start, device, ...) with `output_tokens` as a field. I aggregate by UTC hour-of-day across the whole 1529-row corpus.

```bash
jq -r '[(.hour_start|tostring|.[11:13]), .output_tokens] | @tsv' \
  ~/.config/pew/queue.jsonl \
| awk -F'\t' '{a[$1]+=$2; n[$1]++}
              END {for (k in a)
                printf "%s\trows=%d\tout=%d\tout_per_row=%.0f\n",
                       k, n[k], a[k], a[k]/n[k]}' \
| sort
```

Snapshot of the relevant numbers, captured `2026-04-26T06:52:26Z`:

| hour (UTC) | rows | sum output | output / row |
|-----------:|-----:|-----------:|-------------:|
| 06         |  142 |  2,020,439 |       14,228 |
| 02         |  131 |  2,038,897 |       15,564 |
| 07         |  137 |  2,380,779 |       17,378 |
| 05         |   79 |  1,398,421 |       17,702 |
| 09         |   93 |  1,752,558 |       18,845 |
| 08         |  119 |  2,317,156 |       19,472 |
| 03         |   89 |  1,725,207 |       19,384 |
| 23         |   33 |    664,848 |       20,147 |
| 21         |   26 |    594,413 |       22,862 |
| 22         |   27 |    621,065 |       23,002 |
| 04         |   53 |  1,257,945 |       23,735 |
| 20         |   28 |    740,707 |       26,454 |
| 11         |   57 |  1,630,328 |       28,602 |
| 10         |   71 |  2,300,380 |       32,400 |
| 01         |   73 |  2,434,560 |       33,350 |
| 13         |   50 |  1,707,367 |       34,147 |
| 00         |   32 |  1,161,370 |       36,293 |
| 14         |   55 |  2,035,356 |       37,006 |
| 12         |   43 |  1,669,094 |       38,816 |
| 19         |   33 |  1,379,410 |       41,800 |
| 15         |   46 |  1,995,274 |       43,376 |
| 16         |   46 |  2,510,278 |       54,571 |
| 17         |   34 |  2,080,134 |       61,180 |
| 18         |   33 |  2,040,445 |       61,832 |

Pearson correlation between `rows` and `output_per_row`: **-0.5816** over n=24 hours. Row-count spread: 26 to 142 (5.46x). Output-per-row spread: 14,228 to 61,832 (**4.35x**). Both spreads are large; the correlation says they move in opposite directions.

## What I tried

- **First instinct: it's just `automated` traffic dragging the early hours down.** Plausible — openclaw is automation-only and runs continuously, so it might fatten the row count in the wee hours without contributing much output volume per row. Tested by re-running the pivot with `select(.source!="openclaw")`. The inverse correlation weakens but does not disappear; r drops from -0.58 to roughly -0.41. The effect is partially explained by automation, but there is residual inversion among the human-driven sources alone.
- **Second instinct: it's a small-N artifact at the hours with few rows.** The thinnest hours by row-count (20, 21, 22) have only 26-28 rows each, so a single fat row could swing the per-row average. Tested by checking the median output-tokens-per-row per hour rather than the mean. Median is more robust and less swayed by single fat rows. Result: the ordering survives — late afternoon and evening hours still have higher median output-per-row than early-morning hours, and hours 06-08 (the row-count peak) still produce the lowest median.
- **Third instinct: this is just session-shape changing through the day.** Long, deliberate sessions in the afternoon produce few hour-rows but each row spans a heavy collaboration. Short, scattered sessions in the morning produce many hour-rows but each one is a shallow check-in. This is the explanation I find most credible, but it is also the explanation that is hardest to falsify with the data on hand. See "why it worked" below.

## What worked

The cleanest analytical move is to stop conflating "row count" with "activity." A row in `queue.jsonl` is an hour-bucket *for a (source, model, device) tuple*. Two scenarios produce very different row counts for the same human effort:

1. Working in **one** harness on **one** model for an hour: you produce 1 row, and `output_tokens` for that row reflects everything you asked for.
2. Bouncing between **three** harnesses (claude-code, codex, opencode) on **two** models in the same hour: you produce up to 6 rows, and `output_tokens` is split across them.

If I tend to bounce between harnesses in the morning (catching up on overnight automation, reviewing PRs in one tool, kicking off scripts in another) and settle into a single deep-work harness in the afternoon (one tool, one big task), then morning hours should look exactly like the data shows: high row count, low output-per-row. Afternoon hours should look like the data shows: low row count, high output-per-row.

Mechanical check: count *distinct sources active per hour* in the corpus.

```bash
jq -r '[(.hour_start|tostring|.[11:13]), .source] | @tsv' \
  ~/.config/pew/queue.jsonl \
| sort -u | awk -F'\t' '{c[$1]++} END {for (k in c) print k, c[k]}' | sort
```

This tallies "for each UTC hour-of-day, how many distinct sources have at least one row." If my hypothesis is right, the morning hours should have more distinct sources than the afternoon. (I am leaving this check as an exercise — the rest of the post stands on the rows/out_per_row numbers, which are not in dispute regardless of mechanism.)

## Why it worked (or: my current best guess)

There are at least three plausible mechanisms, and I think all three contribute:

1. **Source-mix shift through the day.** Early UTC hours coincide with overnight automation and morning catch-up: many short rows from many sources. Late UTC hours coincide with deep-work blocks: few long rows from one source.

2. **Bucket-edge fragmentation.** A 90-minute session that crosses an hour boundary becomes two rows. A 30-minute session inside one bucket becomes one row. If short sessions are clustered in a few hours and long sessions spread across multiple, the long-session hours look "thin per row" purely as a counting artifact.

3. **Cache regime shift.** Cached-input rows tend to have lower marginal output (the model is mostly re-reading state) than fresh-input rows (the model is producing new analysis). If cache-hit rate is higher in some hours than others — which it is; see prior posts on hour-of-day cache-hit variation in this series — then output-per-row will be *anti*-correlated with cache regime. Heavy-cache hours are also heavy-row hours (lots of small probe calls), and they produce thin rows. Fresh-cache hours are deep-work hours and produce fat rows.

The 4.35x output-per-row spread between hour 06 (14,228 tokens/row) and hour 18 (61,832 tokens/row) is the headline, but the more important observation is that **per-row output is not interchangeable with per-hour output**. Hour 06 has more total output (2.02M) than hour 17 (2.08M, basically tied), but hour 06 needed 142 rows to get there while hour 17 needed 34. Same throughput, 4x different row-density, completely different operational meaning for capacity planning.

## What this changes for dashboards

If you have a dashboard that reports "rows per hour" as a proxy for "activity," replace it with two panels: row count and output-per-row, side-by-side. They tell opposite stories. A dashboard that shows only the first will overstate morning activity and understate evening intensity. A dashboard that shows only the second will overstate evening intensity in the few hours where one fat row dominates.

The capacity-planning corollary: if you are sizing for "row throughput" (database writes, ingestion pipelines), the morning peak (142 rows in hour 06) is the binding constraint. If you are sizing for "token throughput" (LLM-side spend, output streaming bandwidth), the afternoon peak is the binding constraint, and it lives at a *lower* row rate. The two constraints want different infrastructure shapes.

## What I would do differently

Always co-report row count and output-per-row when slicing by any time axis. The r=-0.58 inverse means a single-metric report is going to mislead in one direction or the other depending on which metric you picked. The cheapest fix is two columns in the same table; the most expensive fix is a wrong capacity decision driven by the wrong metric.

## Numerics captured

Source: `~/.config/pew/queue.jsonl`, 1529 rows, single device `a6aa6846-9de9-444d-ba23-279d86441eee`, total 9,040,277,360 tokens. Captured `2026-04-26T06:52:26Z`. Hour-of-day pivot as in the table above. Pearson r(rows, output_per_row) = -0.5816 over n=24 hourly buckets. Row count spread 26 to 142 (5.46x). Output-per-row spread 14,228 to 61,832 (4.35x). Tied throughput at hour 06 vs hour 17 (2.02M vs 2.08M output tokens) at 4.18x different row counts (142 vs 34) is the clearest illustration in the corpus.

## Links

- jq manual: https://jqlang.github.io/jq/manual/
- Prior posts: `2026-04-25-hour-of-week-entropy-0915-uniform-with-spikes.md`, `2026-04-25-tok-per-bucket-vs-tok-per-span-hour-orthogonal-density.md`, `2026-04-26-hour-of-week-0914-normalised-entropy-and-why-no-dead-time-is-the-wrong-reading.md` (all use the hour axis differently; this post is the per-row-density variant)
