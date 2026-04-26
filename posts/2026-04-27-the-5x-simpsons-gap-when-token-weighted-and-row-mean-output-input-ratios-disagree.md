---
title: "The 5x Simpson's gap: when token-weighted and row-mean output/input ratios disagree"
date: 2026-04-27
tags: [pew-insights, simpsons-paradox, output-input-ratio, claude, gpt, statistics]
slug: the-5x-simpsons-gap-when-token-weighted-and-row-mean-output-input-ratios-disagree
---

# The 5x Simpson's gap: when token-weighted and row-mean output/input ratios disagree

Every per-model ratio report in `pew-insights` exposes two columns that
look superficially similar and are in fact computing very different
quantities: the *token-weighted* aggregate ratio (one big sum over the
numerator divided by one big sum over the denominator) and the
*mean-row* ratio (an unweighted average of per-row ratios). For most
sources on most days these two numbers track within 10–20% of each
other, and the column is decorative. For `claude-opus-4.7` on today's
snapshot, they differ by **5.06×**, and that gap is by itself a
diagnostic worth half a page of analysis.

The reading from `pew-insights output-input-ratio` against
`~/.config/pew/queue.jsonl`, timestamp `2026-04-26T19:32:35.468Z`:

```
overall: 0.0121     rows: 1,260     input: 3,452,589,148    output: 41,841,715

model               rows  input          output      ratio   mean-row-ratio
------------------  ----  -------------  ----------  ------  --------------
claude-opus-4.7     484   1,394,099,005  30,830,256  0.0221  0.1119
gpt-5.4             492   1,357,355,927   6,936,034  0.0051  0.0053
claude-opus-4.6.1m  182     606,191,092   3,450,625  0.0057  0.0108
claude-haiku-4.5     31      66,264,788     133,160  0.0020  0.0028
unknown              56      18,262,497     410,432  0.0225  0.0342
claude-sonnet-4.6     9       9,943,726      73,444  0.0074  0.0060
```

The `ratio` column is `sum(output) / sum(input)` over the model's rows.
The `mean-row-ratio` column is `mean_i(output_i / input_i)`. These are
the two natural ways to summarise a per-row ratio metric, and which one
you should believe depends entirely on what you want the summary to
mean.

For `claude-opus-4.7`: token-weighted ratio is 0.0221 (output is 2.21%
of input), but the row-mean ratio is 0.1119 (the average row produces
output equal to 11.19% of its input). Same model, same window, same
484 rows, same definition of output and input. The two numbers
disagree by a factor of 5.06.

For `gpt-5.4`: token-weighted 0.0051, row-mean 0.0053. Disagreement is
4%.

For `claude-opus-4.6.1m`: token-weighted 0.0057, row-mean 0.0108. Gap
1.89×.

For `claude-haiku-4.5`: token-weighted 0.0020, row-mean 0.0028. Gap
1.40×.

For `unknown`: 0.0225 vs 0.0342. Gap 1.52×.

For `claude-sonnet-4.6`: 0.0074 vs 0.0060. Gap **inverted** — row-mean
*lower* than token-weighted. Ratio 0.81×.

This is a textbook Simpson's-paradox-shape result. Three of the six
models have row-mean above token-weighted. One has them below. Two have
them roughly equal. The biggest disagreement, by a wide margin, is on
the model that drove the most output token mass on the corpus
(`claude-opus-4.7`, 30.83M output tokens, 73.7% of the corpus's total
output volume).

## What the gap is telling us

Token-weighted ratio is what you should use for *cost reasoning*. If
you want to know "for every dollar I spend on input, how much output
does this model produce in aggregate?", you want token-weighted. It is
a hard, accountable number: it ties directly to what the API meter
shows.

Row-mean ratio is what you should use for *behavioural reasoning*. If
you want to know "how chatty is this model on a typical request?", you
want row-mean. It is the right summary if every row represents one
unit of operator intent — one request from one human, one tool call,
one whatever — and you want to know how the model behaves *per
request*.

The two answers diverge when the row-size distribution is heavily
skewed and the per-row ratio correlates with row size. Specifically:
when big rows have *low* per-row output/input ratios (long prompts
that produce short answers) and small rows have *high* per-row ratios
(short prompts that produce relatively chatty answers), the
token-weighted ratio gets pulled toward the low value and the row-mean
ratio gets pulled toward the high value, and the gap appears.

For `claude-opus-4.7`, the prompt-size distribution from
`pew-insights prompt-size` (same snapshot, `2026-04-26T19:32:35.527Z`)
shows the shape clearly:

```
claude-opus-4.7
rows  mean       p95         max
484   2,880,370  21,723,200  55,738,577

bucket counts:
0–4k       1
4k–32k    48
32k–128k  84
128k–200k 24
200k–500k 110
500k–1M   103
1M+       114
```

114 rows above 1M input tokens. 48 rows in the 4k–32k band. The mean
is 2.88M but the median is somewhere in the low hundreds of thousands.
This is precisely the distribution shape that drives the Simpson's
gap: a long head of small-prompt rows where the answer can plausibly
be 10% of the prompt by mass, and a long tail of giant-prompt rows
where the answer can almost never exceed 1–2% of the prompt by mass
because the model is fundamentally limited in how much output it can
generate on a single turn.

If a request comes in with 1M input tokens and the model emits a 10K
output, the per-row ratio is 0.01. If the same model gets a 4K input
and emits a 400-token output, the per-row ratio is 0.10. Same model,
same chat-completeness intent, but the per-row ratio differs by 10×
purely because of input scaling. Average those, and you get 0.055. Sum
the masses and divide, and you get something close to 0.011 because
the 1M-token row dominates the denominator.

## Why this matters for `claude-opus-4.7` specifically

The 5.06× gap on `claude-opus-4.7` is the largest in the corpus by a
wide margin. The comparable model `claude-opus-4.6.1m` shows only a
1.89× gap on a similar (1M-context) ceiling. So the explanation isn't
just "this is what 1M-context Anthropic models do."

The differentiator is row volume. `claude-opus-4.7` has 484 rows and
`claude-opus-4.6.1m` has 182 rows. With ~2.7× more rows over a similar
input mass, `claude-opus-4.7`'s row-size distribution is necessarily
broader: more rows means more *small* rows (since the total input is
similar), which means a heavier head of high-per-row-ratio
observations, which inflates the row-mean while the token-weighted
remains anchored by the giant-prompt tail.

The same effect explains the much smaller gap on `gpt-5.4`. `gpt-5.4`
has 492 rows over 1.36B input tokens — almost identical row count and
input mass to `claude-opus-4.7` — but its row-size *bucket
distribution* is different:

```
gpt-5.4
4k–32k    2
32k–128k  9
128k–200k 6
200k–500k 44
500k–1M   119
1M+       312
```

312 of 492 rows (63.4%) are above 1M tokens. For `claude-opus-4.7`, only
114 of 484 rows (23.6%) are above 1M. `gpt-5.4`'s prompt distribution
is *much* more uniformly large — most rows are in the same order of
magnitude. So the per-row ratios don't have room to vary by much, and
the row-mean and token-weighted summaries converge.

Said simply: **the token-weighted vs row-mean gap is a measure of
prompt-size dispersion, expressed in output/input ratio space.** It's
not a model behaviour metric — it's a corpus shape metric.

## The inverted case: `claude-sonnet-4.6`

The one model with *inverted* ordering (row-mean 0.0060 *lower* than
token-weighted 0.0074) has only 9 rows. With n = 9, neither summary
is statistically reliable, but the inversion itself is interesting.
For row-mean to be lower than token-weighted, the *biggest* rows
have to have *higher-than-average* per-row ratios — i.e., the bigger
the prompt, the chattier the response, proportionally. That's an
unusual pattern. With only 9 rows it's almost certainly noise: a
single row with both a large input and a large output can flip the
ordering at this sample size.

I list it not because the inversion is real but because it's the
honest "be careful at low n" footnote. Anything below n ≈ 30 should
be reported with both columns and a sample-size caveat, and any gap
should be treated as artefactual until proven otherwise.

## The corpus-wide reading

The corpus-wide overall ratio at the top of the report is 0.0121
(token-weighted). That's 1.21% — the entire workload, all sources, all
models, produces about 1.2 cents of output for every dollar of input
in token units. If you instead computed the unweighted mean of per-row
ratios across all 1,260 rows, the result would be much higher, probably
in the 6–10% range, because the head of small-prompt rows would dominate
the average.

Neither number is wrong. They answer different questions:

- **Cost question**: "what fraction of my token spend goes to output?"
  → token-weighted, 1.2%.
- **Behaviour question**: "for a typical request, how much does the
  model respond relative to the prompt?" → row-mean, ~6–10%.
- **Cache-economics question**: "how much do I save by caching the
  prompt?" → token-weighted again, but you should also pull
  `cache-hit-ratio` and combine.

## Operational lessons

Four things to take away:

1. **Always read both columns.** A 1.5× gap is normal. A 5× gap means
   your row-size distribution is doing something interesting, and the
   explanation is almost always "the giant-prompt tail anchors the
   token-weighted denominator while the small-prompt head dominates
   the row-mean numerator."

2. **Pick the column that matches your question.** Cost reporting
   wants token-weighted. Per-request behaviour wants row-mean.
   Latency-percentile work wants neither — it wants a per-row
   distribution, not a summary scalar.

3. **Treat large gaps as a structural finding, not noise.** The 5.06×
   gap on `claude-opus-4.7` is a genuine fact about how the model is
   being driven on this workload: a mix of giant-prompt agentic
   sessions and smaller chat-style interactions. That mix is exactly
   what the gap is measuring.

4. **Below n ≈ 30 rows, treat both columns as anecdotal.** The
   `claude-sonnet-4.6` inversion is a sample-size artefact; do not
   build any narrative on it.

The output-input-ratio report becomes a different and more interesting
instrument once you read the two ratio columns as a pair. The
disagreement between them is where most of the story lives. On today's
snapshot, that disagreement says: `claude-opus-4.7` is being driven
across a much wider range of prompt sizes than any other model in the
corpus, and that breadth is exactly what makes its per-request
behaviour and its cost economics tell different stories.

---

*Source*: `pew-insights output-input-ratio` against
`~/.config/pew/queue.jsonl`, snapshot timestamp
`2026-04-26T19:32:35.468Z`, 1,260 rows, 3,452,589,148 input tokens,
41,841,715 output tokens. Cross-referenced with
`pew-insights prompt-size` from snapshot
`2026-04-26T19:32:35.527Z` for the per-model bucket distributions.
Same queue file, same build of `pew-insights`.
