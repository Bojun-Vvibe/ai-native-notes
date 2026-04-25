---
date: 2026-04-25
title: "Prompt→Output Correlation Per Model: Why The Slope Is Always Tiny And The R Is Sometimes Missing"
tags: [pew-insights, regression, model-behavior, telemetry, output-shape]
---

A correlation coefficient between input tokens and output tokens, computed per model across all the active hour-buckets in a fleet, is one of those metrics that sounds like it should be obvious and turns out to be the opposite. The naive expectation is that bigger prompts produce bigger completions: more context, more to say, more tokens streamed back. The actual numbers do not agree, and the disagreement is structured enough that it tells you something useful about how each model class is actually being driven.

Here is the live snapshot I am going to dissect, from `pew-insights prompt-output-correlation` captured at 2026-04-25T14:35:08.500Z over 899 active buckets, 8,615,338,606 total tokens, 15 model groups (12 shown), with a global Pearson r of 0.577 and a global OLS slope of 0.006:

```
model                 tokens         in             out         buckets  r      slope  intercept  degen
--------------------  -------------  -------------  ----------  -------  -----  -----  ---------  -----
claude-opus-4.7       4,879,894,184  1,360,993,892  25,160,859  293      0.540  0.005  61854      no
gpt-5.4               2,505,535,347  1,309,502,899  6,879,514   393      0.818  0.005  2249       no
claude-opus-4.6.1m    1,108,978,665  606,191,092    3,450,625   167      0.493  0.004  4338       no
claude-haiku-4.5      70,717,678     66,264,788     133,160     30       0.495  0.001  1871       no
unknown               35,575,800     18,262,497     410,432     56       0.737  0.021  638        no
claude-sonnet-4.6     12,601,545     9,943,726      73,444      9        0.938  0.011  -4469      no
gpt-5                 850,661        0              842,008     170      —      —      —          yes
claude-opus-4.6       350,840        343,761        5,787       4        0.603  0.012  397        no
gemini-3-pro-preview  154,496        0              43,665      37       —      —      —          yes
gpt-5.1               111,623        0              81,808      53       —      —      —          yes
claude-sonnet-4.5     105,382        0              93,835      37       —      —      —          yes
claude-sonnet-4       53,062         0              53,062      26       —      —      —          yes
```

Six rows have a real correlation. Five rows are degenerate. One row (`unknown`) is doing something visibly different from the rest. The degenerates and the outlier are where the post is.

## The slope is microscopic by design

Look at the slope column for the four highest-token rows. claude-opus-4.7: 0.005. gpt-5.4: 0.005. claude-opus-4.6.1m: 0.004. claude-haiku-4.5: 0.001. The top three workhorses agree to within a thousandth that for every additional input token the bucket sees, the model emits roughly five thousandths of an output token. That is a 200:1 input-to-output expansion ratio sitting underneath six different per-bucket sample sizes ranging from 30 to 393.

This is not a coincidence and it is not a property of the regression. It is a property of how agentic chat is actually shaped. Each turn loads a context — the system prompt, the conversation history, the tool result blobs, the file reads — and emits a small reply: a short message, a tool call, or a brief plan. The expansion factor is set by the product, not by the prompt. A 4-million-token bucket of input pulls roughly 20,000 tokens of output back, and a 400,000-token bucket pulls roughly 2,000. The slope is tiny because the model's job is to compress, not to expand.

The intercept column reinforces this. claude-opus-4.7 sits at intercept 61,854 with mean output 85,873, which means once you subtract the slope-times-mean-input contribution (≈24,000 tokens), the bucket-level baseline output is 62k. That baseline is almost certainly the floor of "this fleet ran during this hour at all" — system messages, tool routing, scaffolding chatter — which appears whether the input is 1M tokens or 10M tokens. A bucket with no work at all would not be in the dataset, so the intercept is reading the cost of just being awake.

## Why the r-values fan out

The slopes are nearly identical across the workhorses but the correlations are not. claude-opus-4.7 has r = 0.540. gpt-5.4 has r = 0.818. claude-opus-4.6.1m has r = 0.493. This is the interesting part.

When the slope is fixed by product behavior, the only thing that can move r is the bucket-level scatter around that slope. r is going to be high when each bucket's (in, out) pair sits close to the regression line, and low when buckets diverge from the line in unpredictable ways.

claude-opus-4.7's standard deviations are std-in = 9.05M, std-out = 86,765. Mean-in is 4.65M, mean-out is 85,873. The output coefficient of variation is 86,765 / 85,873 ≈ 1.01. The input coefficient of variation is 9.05M / 4.65M ≈ 1.95. Those are wide bucket distributions on both axes. The buckets themselves contain a heterogeneous mix of work — some are agent loops with many small turns, some are single deep thinking turns, some are tool-call-heavy. The wide bucket variance washes out the linear relationship even though the slope is locked.

gpt-5.4's std-in / mean-in is 4.6M / 3.33M ≈ 1.38, and its std-out / mean-out is 25,776 / 17,505 ≈ 1.47. Tighter on both axes. Less heterogeneity per bucket, so the linear fit is cleaner — r = 0.818. The mechanism is not that gpt-5.4 is "more linear" as a model. It is that the buckets in which gpt-5.4 was active happen to be more homogeneous in their workload character. Probably a single source dominates that model's traffic and emits a more consistent shape per hour-bucket.

claude-sonnet-4.6 has the highest r in the table at 0.938 over only 9 buckets. With nine points, you can get to r = 0.938 with very little effort if the line goes through the origin. This is a sample-size warning, not a signal. Any per-model row with buckets < 30 should be read with suspicion.

claude-haiku-4.5 has slope 0.001 — five times smaller than the workhorses. r = 0.495 is similar to opus-4.7's r, but the slope difference is structural. Haiku is being driven differently: shorter prompts, even shorter replies, and the bucket-level scatter dominates a much smaller signal. When the slope itself drops, the SNR drops with it and r becomes noisier.

## The five degenerate rows: zero input

`gpt-5`, `gemini-3-pro-preview`, `gpt-5.1`, `claude-sonnet-4.5`, `claude-sonnet-4` all show `in = 0` and `degen = yes`. The regression is undefined because std-in is zero — every bucket has the same input value (zero), and you cannot fit a line to a vertical strip of points.

Zero input but nonzero output is not a sensor failure. It is a real shape that happens when the input column for those rows is not being populated at all — either the upstream telemetry source emits cached_input + reasoning + completion fields but no fresh-input field for that model, or the model's API surface returns usage with `prompt_tokens = 0` because the entire context arrived through cache_creation and cache_read and the fresh-input slot is never touched.

That last interpretation matters. If a model's traffic is dominated by cache reads, the "fresh input" axis collapses to zero and the prompt-output correlation becomes literally undefined. The model is not less correlated with its input — it has no measurable fresh input on this axis. The right way to read those five `degen=yes` rows is "this model is running in cache-saturated mode within this fleet's traffic," not "this model's behavior is anomalous."

You can sanity-check this by looking at output volume. gpt-5 has 842,008 output tokens over 170 buckets — about 4,953 tokens per bucket. claude-sonnet-4 has 53,062 over 26 buckets — about 2,041 per bucket. These are non-trivial production volumes. The buckets are real. They simply are not contributing usable data to a fresh-input regression because the fresh-input axis is dark.

The right per-model decision here is to either (a) report degenerate rows as `—` and exclude them from any aggregate r calculation, which is what the tool does, or (b) re-run the regression with `cached_input + fresh_input` as the x-axis. (b) is more honest for fleets where caching is dominant, but it changes the meaning of slope from "additional output per fresh prompt token" to "additional output per total context byte," which is a different question. The tool's choice — show the row, mark it `degen=yes`, refuse to fit — is the safer one.

## The `unknown` row is a category error

`unknown` shows slope = 0.021 and intercept = 638. That is a slope four times larger than the workhorses and an intercept two orders of magnitude smaller. r = 0.737 over 56 buckets is a stronger signal than the top three rows.

What is `unknown`? It is whatever the model-extraction logic could not identify: some sources do not stamp a model name into their JSONL, or stamp it in a field the parser does not recognize, or use a per-request alias the catalog cannot resolve. The bucket of "unknown" is therefore a heterogeneous mix of whatever providers and models slipped through identification, and the regression is fitting that aggregate.

The high slope (0.021) suggests the unknown bucket is dominated by short-prompt, short-output workloads where each input token visibly drives output (think completion-style or autocomplete-style traffic, not chat-style with massive context). The low intercept (638) suggests there is barely any baseline scaffolding cost — these are not agent loops, they are single-shot lookups.

In other words, `unknown` is leaking a different traffic class into the same regression. The right action is not to interpret its slope, but to fix the model-name extraction so that traffic gets re-attributed to the right per-model row. Until then, treat the unknown row as a tag for "telemetry coverage gap," not as a model fingerprint.

## What the global r = 0.577 actually means

Aggregate Pearson over the whole fleet is 0.577 with global slope 0.006. That looks like a reasonable mid-correlation, but it is the average of six well-correlated rows, five degenerate rows that did not contribute, and one mis-classified row. The "global" slope of 0.006 is essentially a token-weighted average of the workhorse slopes (0.005, 0.005, 0.004) pulled slightly upward by the unknown row's 0.021.

Reading the global r/slope as a fleet-level fact misses the structure. The fleet does not have one prompt-output relationship. It has at least three:

1. **Workhorse models in cache-aware agent loops** — slope ≈ 0.005, intercept ≈ tens of thousands, r in the 0.5–0.8 range depending on bucket homogeneity.
2. **Cache-saturated models with no fresh-input signal** — slope undefined, but real output volume; need a different x-axis.
3. **Misclassified short-form traffic** — high slope, low intercept, decent r, but actually a mixed bag that should be split.

A fleet-level dashboard that quotes only the global r is hiding the existence of categories 2 and 3.

## Practical reading rules

A few rules that fall out of the table:

- **Slope alone is the cleanest model-class signal.** Workhorses cluster around 0.005. Anything above 0.01 is either a different traffic class or a model with very different output-shaping. Anything at 0.001 or below is being driven in a degenerate regime (haiku here is one example).
- **r is a sample-quality signal, not a model-quality signal.** A model with r = 0.94 over 9 buckets tells you about your bucket count, not about the model. A model with r = 0.49 over 167 buckets tells you the workload is heterogeneous within those buckets.
- **`degen=yes` is a fleet shape signal.** It says "for this model, the fresh-input axis is empty in your data." That is a property of how the fleet's traffic flows through caching, not of the regression machinery.
- **Intercept floors are scaffolding cost.** The intercept is what you would emit per bucket even at zero fresh input. Big-context agents with multi-tool turns will have intercepts in the tens of thousands. Single-shot completion traffic will have intercepts near zero. The intercept is a workload-shape fingerprint hiding inside a regression coefficient.
- **The token-weighted global is an average over different regimes.** If your fleet has a bimodal mix (long-context agent traffic plus short-form completion traffic), the global r will sit somewhere in the middle and describe neither. Always read per-model first.

## The 200:1 expansion ratio is the headline

If I had to pick one number from the snapshot, it is not r. It is the slope. Every workhorse is converging on 0.005 output tokens per fresh input token. That ratio is what governs cost economics, latency budgets, and cache strategy. Cached input is roughly an order of magnitude cheaper than fresh input on most providers, but the output side is full price and capped at this 200:1 multiplier from the input side. So the dominant cost lever is *input volume × 1.0* for fresh input plus *cached volume × ~0.1* for cached input plus *output volume × ~5* for completion at typical price ratios. With slope = 0.005, output dollars are governed by fresh input volume × slope × output_unit_cost — a very small product compared to fresh-input cost itself.

That is why the rational economics for agentic workloads is "cache as much input as possible and accept that output is small." The slope tells you the output side will not blow up no matter what you do to the input side, because the model is not doing expansion. It is doing compression. The 200:1 ratio at the slope level is the empirical floor on how compressed agentic traffic actually is, and any product-level claim about output verbosity needs to be reconciled against that floor.

## Closing

`prompt-output-correlation` is not really a correlation tool. It is a per-model regression panel that lets you read four signals at once: workload homogeneity (via r), output-shaping behavior (via slope), scaffolding cost (via intercept), and fleet-shape pathologies (via the `degen=yes` rows). The Pearson r at the top of the screen is the least informative number on it. The 200:1 slope at three decimal places is the most informative number on it, and the five degenerate rows are the second most informative thing on it because they are telling you which models are running in modes the regression cannot see.

Snapshot: `pew-insights prompt-output-correlation` as-of 2026-04-25T14:35:08.500Z, 899 buckets, 8.6B tokens, global r = 0.577, slope = 0.006, 12 model rows displayed of 15 groups (3 below min-buckets dropped).
