---
title: "Where the Output Mass Lives: Why Completion-Tail Economics Beat Per-Call Pricing Intuitions"
date: 2026-04-25
tags: [cost, telemetry, completions, agent-cli, economics]
---

# Where the Output Mass Lives: Why Completion-Tail Economics Beat Per-Call Pricing Intuitions

Almost every time I've seen someone reason about the cost of an agent CLI fleet, they reason in per-call units. "The model costs $X per million input tokens, $Y per million output tokens, our average call is Z tokens, therefore each call costs roughly K dollars, and we make about N calls per day, so the daily bill is ~K·N." The arithmetic is fine; the model is wrong. Not because the unit prices are wrong, but because *the average call is a fiction*. The distribution is so heavily skewed that the mean is being held up by a small number of monsters, and the practical economics live in the tail, not at the centre.

Yesterday's release of the local insights tool I run added an `output-size` subcommand (v0.4.40) and then a `--by source` re-aggregation flag (v0.4.41). Looking at the histograms across 1,367 rows from my own queue log clarified something I'd known abstractly for a while: the long-tail buckets dominate the cost story, and reasoning about average completions actively misleads.

This post lays out the numbers, the geometry, and three operational rules I've started enforcing as a consequence.

## The histogram that did it

Run against my local queue log, the unfiltered overall completion histogram looks like this:

```
bucket   rows  share
-------  ----  -----
0–256    70    5.1%
256–1k   189   13.8%
1k–4k    342   25.0%
4k–8k    194   14.2%
8k–16k   179   13.1%
16k–64k  258   18.9%
64k+     135   9.9%
```

The mean output is 24,769 tokens. The max is 416,890. The 8k+ buckets together — meaning rows whose generated completion was at least 8,000 tokens — are 41.9% of the row count. That is already higher than most people's intuition. But row count understates the cost story, because cost is roughly proportional to *tokens generated*, and the 64k+ bucket alone, despite being only 9.9% of rows, contributes a wildly disproportionate share of the total 33,858,725 generated tokens.

A quick back-of-envelope: if we assign each bucket the geometric mean of its endpoints (treating `64k+` as anchored around its observed max of 416k, and using a conservative 100k for that bucket as a representative value), the 64k+ bucket alone is responsible for somewhere on the order of 13.5M of the 33.8M total tokens — roughly 40% of the generated mass from 9.9% of the rows. The 16k–64k bucket adds another sizable slice. Together, the two longest buckets (28.8% of rows) account for the *majority* of all generated tokens.

This is the shape that breaks per-call cost intuitions. The "average call" is not a useful unit when 28.8% of calls account for the majority of the bill. The right unit is the bucket-wise contribution.

## Why this matters more than it should

A lot of agent-CLI cost optimisation effort gets spent on the wrong end of the distribution. People iterate on system prompt length, on tool description verbosity, on input-side caching strategies, on conversation compaction. All of those interventions touch the *input* side, and most of them touch every call uniformly. They feel productive because the change set is broad and the dashboard moves a little.

But if your output mass is concentrated in 28.8% of rows that each generate 8k+ tokens, and your typical CLI cost rates put output tokens at 4-5x the price of input tokens, then *uniformly trimming all input contexts by 20%* might shave a few percent off the bill, while *capping the worst 10% of completions at 8k* would shave double-digit percent off. The two interventions look similar on a project plan. They are not similar in impact.

This isn't a hypothetical. The recent insights tooling makes the asymmetry visible: the `--by source` view reveals that one producer (`opencode`, 218 rows) has a p95 completion of 282,231 tokens, while another producer (`vscode-ext`, 321 rows — *more* rows) has a p95 of 14,290. Both routing to the same backend model. Same per-token unit price. Wildly different cost contribution per call. The 218 long-tail rows from `opencode` contribute more cost than the 321 short rows from `vscode-ext`, even though the latter is the row-count leader.

If you were prioritising optimisation work by row count, you'd touch `vscode-ext` first. That would be wrong. If you were prioritising by mean output, you'd touch the producer with the highest mean — which here is `opencode` (mean 58,144) — but the *gap* between mean (58,144) and p95 (282,231) is itself information: this producer has a heavy tail relative to its mean. The intervention that pays off most is not "lower the mean," it's "kill the tail."

## The three rules I now enforce

After staring at these histograms for a few days, I have rewritten my own internal heuristics for cost work. Three rules:

### Rule 1: Optimise the tail, not the mean

If your distribution is heavy-tailed (and almost every agent-CLI completion distribution I've ever measured is heavy-tailed), the mean is a misleading optimisation target. The mean moves when the tail moves; it barely moves when you trim the body.

Concretely: if you want to cut 20% of generation cost, identify the rows whose completion is in the top 10% by token count and figure out *why those rows generated so much*. Common causes I've seen:

- Loops with weak stop conditions that keep going until they hit a token cap
- Tool result echoes where the model recapitulates large tool outputs in its reply
- Multi-step plans rendered as one giant completion instead of a stream of short turns
- Verbose self-explanation enabled by default in some producer's system prompt
- Reasoning content emitted alongside the visible answer (with no truncation)

Each of those has a structural fix that is much cheaper than reducing the mean across all calls.

### Rule 2: Always look at p95, not just mean

The mean of a heavy-tailed distribution is not a sufficient summary. p95 — the 95th percentile completion size — is the bare minimum companion metric. Better: report mean, p50, p95, and max together. The shape of those four numbers tells you whether you have a tame distribution (mean ≈ p50, p95 within 3x, max within 10x) or a beast (mean >> p50, p95 >> mean, max >> p95).

The four-number summary for my fleet right now:

- mean = 24,769
- p50 ≈ 4k–8k (lives in that bucket)
- p95 = 282,231 (worst producer) / 14,290 (best producer) / varies wildly per producer
- max = 416,890

Compare those to the per-call price intuition you had before reading them. The intuition was wrong. The numbers are different by an order of magnitude.

### Rule 3: Aggregate by producer before you aggregate by model

Cost attribution by model tells you what to negotiate with your provider. Cost attribution by producer tells you what to *fix in your own software*. The latter is almost always the higher-leverage move, because providers don't lower prices in response to one customer's frowning, but you can ship a code change to your own producer this afternoon.

The `--by source` flag on the histograms makes this trivial. The implementation falls back to an `unknown` sentinel for missing producer data so the global denominators stay stable, which is a small detail but matters: you do not want to be in a situation where two different aggregation views disagree on the row count because one silently dropped null-producer rows. Stable denominators across re-aggregations are a precondition for any cost reasoning that crosses views.

## Per-call pricing intuition is also wrong about caching

A connected mistake: when people reason per-call, they tend to assume caching effects average out. They don't. Cache hit rates are themselves heavy-tailed, because some producers re-issue near-identical context across many turns (high cache hit) while others re-build fresh context on every turn (zero cache hit).

A previous insights release surfaced cache-hit ratios per model and the variance was extreme — `claude-opus-4.7` was clocking a 232% effective ratio (i.e., the cache was being read more than the inputs, because of how multi-turn re-reads interact with the cache pricing model), while another model variant was hovering near 91%. Per-call reasoning would treat both as "cached" and leave it there. Per-row-with-distribution reasoning treats them as completely different cost regimes that need different optimisation playbooks.

The pattern is the same: the per-call mental model collapses a distribution into a scalar, and the scalar lies.

## What this looks like in practice

If I were starting from scratch on cost telemetry for an agent CLI fleet today, I would build it like this:

1. **Log every row with `prompt_tokens`, `output_tokens`, `cached_tokens`, `model`, `producer`, `timestamp`.** Six fields. JSONL append-only. No fancier.
2. **Build histograms first, dashboards later.** A dashboard with a "mean output tokens" tile is worse than no dashboard, because it implies a level of summary the data does not support.
3. **Default the histogram view to per-producer, not per-model.** You can flip the axis with one flag, but the producer view is more actionable.
4. **Make the bucket ladder fixed and visible.** The 0/256/1k/4k/8k/16k/64k ladder I'm using above is not magic, but it spans the realistic range for completions and makes period-over-period comparisons honest. A floating ladder ("auto-bucket the data") makes drift comparisons impossible.
5. **Track the row share AND the token share of each bucket.** Row share tells you how often the bucket is hit. Token share tells you how much of the bill it owns. They diverge sharply in the tail, and the divergence *is* the story.
6. **Track p95 and max alongside mean.** Always. If your dashboard only has space for one number per row, pick p95 over mean. p95 is a better leading indicator of cost surprise.
7. **Re-run the histograms after every intervention.** "We added a max_output_tokens cap of 8k" should produce a visible bucket-shape change. If it doesn't, the cap isn't being applied where you think it is.

## A note on the input-side analogue

Everything above is about the output side, because that's where my own bill lives. But the same shape applies to the input side, sometimes worse. A previous release of the same insights tooling shipped a `prompt-size` histogram across the 0/4k/32k/128k/200k/500k/1M ladder. Live smoke against 1,049 rows showed 50.7% of rows shipping 1M+ token prompts, with a mean of 3.17M and a max of 55.7M for `claude-opus-4.7`. Applying an `--at-least 1000000` floor kept 532 rows and lifted the mean to 5.90M — a 1.86x hidden delta that the unfiltered mean was disguising.

Same pattern. Same lesson. The body of the distribution is not where the cost lives. The tail is.

## Closing

The per-call pricing intuition is comforting because it lets you reason about cost the way you reason about a coffee budget: I drink three a day, each one is $4, that's $12. Easy. Defensible. Predictable.

Agent CLI calls are not coffees. The distribution of work-per-call is bimodal at best and power-law at worst. A small fraction of calls do most of the generating, and within that fraction, an even smaller sub-fraction does most of the *very* long generating. Reasoning about averages launders this geometry into a single scalar and the scalar is a lie that costs you money.

The fix is not exotic. It is a fixed bucket ladder, two aggregation axes (model and producer), a four-number summary (mean, p50, p95, max), and the discipline to look at the histogram before you look at the mean. With those in place, the next intervention that meaningfully lowers your bill becomes obvious. Without them, you will spend another quarter optimising things that do not move the curve, while the actual culprit — almost certainly a single producer with a runaway p95 — sits there unaddressed because the model-mean dashboard says everything is fine.

Look at your tail. The cost lives there.
