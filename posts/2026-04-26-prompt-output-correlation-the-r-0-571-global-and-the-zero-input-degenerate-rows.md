---
date: 2026-04-26
tags: [pew-insights, correlation, tokens, model-shape]
source: pew-insights prompt-output-correlation @ 2026-04-25T19:06:44.237Z
---

# Prompt→output correlation: the r=0.571 global and the zero-input degenerate rows

There is a question I keep avoiding because the answer is uncomfortable: when
I send a longer prompt, do I actually get a longer answer back? Most operator
intuition says yes — bigger context, more to summarise, more to cite, more
tokens out. But intuition is not a measurement. So I ran the
`prompt-output-correlation` lens against my queue snapshot at
`2026-04-25T19:06:44.237Z`, across 908 active hour-buckets and 8,745,663,181
total tokens, and the global Pearson r came back as **0.571** with a global
slope of **0.006**. That number is the entire post.

## What the lens computes

For each model group, the lens collects every UTC hour-bucket where that
model produced any positive output. Each bucket becomes a single
`(input_tokens, output_tokens)` point. Pearson r is computed across those
points, and an OLS line `y = slope · x + intercept` is fitted on top. The
correlation is **bucket-level**, not row-level — meaning it answers "in hours
where I sent more prompt tokens to model M, did I also get more completion
tokens back?" rather than "in any single API call, does longer input mean
longer output?". Those are very different questions, and conflating them is
how you end up with bad routing heuristics.

The header line of the snapshot:

```
as of: 2026-04-25T19:06:44.237Z    groups: 15 (shown 12)    active-buckets: 908
tokens: 8,745,663,181    in: 3,387,131,538    out: 38,102,195
minBuckets: 2    minTokens: 0    sort: tokens
global r: 0.571    global slope: 0.006
dropped: 0 bad hour_start, 0 zero/non-finite tokens, 0 by source filter,
0 by model filter, 3 below min-buckets, 0 below min-tokens, 0 below top cap
```

Three model groups were dropped for having fewer than the minimum bucket
count. Zero rows were dropped for bad timestamps or non-finite tokens, which
is itself a small piece of evidence that the queue ingestion is clean — no
schema drift this week, no NaN propagation from a half-shipped writer.

The aggregate ratio is also worth holding in your head before drilling in:
input is **3.39 B tokens**, output is **38.1 M tokens**. Output is **1.13%**
of input by mass. That ratio is the macro shape of the workload — almost all
the cost is on the prompt side, not the completion side. The
correlation question is whether the small completion side **tracks** the
huge prompt side, hour by hour.

## The per-model table

The interesting rows are not at the top.

| model              | tokens         | buckets | mean-in   | mean-out | std-in    | std-out | r     | slope | intercept |
|--------------------|---------------:|--------:|----------:|---------:|----------:|--------:|------:|------:|----------:|
| claude-opus-4.7    | 4,991,968,211  | 302     | 4,523,082 | 86,180   | 8,946,844 | 85,528  | 0.536 | 0.005 | 62,998    |
| gpt-5.4            | 2,523,785,895  | 402     | 3,283,648 | 17,129   | 4,564,266 | 25,606  | 0.819 | 0.005 | 2,045     |
| claude-opus-4.6.1m | 1,108,978,665  | 167     | 3,629,887 | 20,662   | 4,995,667 | 45,586  | 0.493 | 0.004 | 4,338     |
| claude-haiku-4.5   | 70,717,678     | 30      | 2,208,826 | 4,439    | 1,999,608 | 4,698   | 0.495 | 0.001 | 1,871     |
| unknown            | 35,575,800     | 56      | 326,116   | 7,329    | 191,970   | 5,345   | 0.737 | 0.021 | 638       |
| claude-sonnet-4.6  | 12,601,545     | 9       | 1,104,858 | 8,160    | 850,619   | 10,370  | 0.938 | 0.011 | -4,469    |

The flagship model, `claude-opus-4.7`, has the largest mass and a middling
correlation of 0.536. `gpt-5.4` is the workhorse with 402 buckets and a much
tighter coupling at 0.819. The 1m-context Opus variant drops to 0.493. And
then `claude-sonnet-4.6`, with only 9 buckets, has a near-perfect r of
0.938 — which is almost certainly a small-sample artefact and not a real
property of the model. Nine points will fit anything if you squint hard
enough.

## What 0.571 actually means in operator terms

A Pearson r of 0.571 is the kind of number that lets a manager tell two
opposite stories. Story A: "input strongly predicts output, you can budget
on prompt mass alone." Story B: "input only explains about a third of the
variance in output, you cannot budget on prompt mass alone." Both
sentences are technically defensible, and that is exactly the problem with
quoting r without an effect size.

The grown-up way to read it is `r²`. `0.571² = 0.326`. So **about 33% of
the hour-bucket-level variance in output token mass is explained by
input token mass**, globally, across my queue. The remaining 67% is
something else: tool-call density, refusal rate, retry storms,
streaming-cancellation cutoffs, the operator's own choice to run a long
think versus a short edit. None of that is captured in the prompt-size
axis.

The slope is the more useful number for budgeting. Global slope is **0.006**.
Per-model slopes cluster between 0.001 (haiku) and 0.011 (small-sample
sonnet). For the high-mass models — opus-4.7, gpt-5.4, opus-4.6.1m — the
slope is essentially identical at **0.004 to 0.005**. That is the
real finding hidden in this lens: across three different vendors and two
different context-window sizes, **the marginal output-per-input-token
ratio sits in a narrow band around 0.5%**.

If you give the model a thousand more prompt tokens this hour, you should
expect roughly five more output tokens this hour, on average. That is
remarkably stable. It also tells you that the model providers have all
converged on the same operator behaviour: I use them for summarisation,
not for generation. The 1.13% output-to-input mass ratio is consistent
with that; the 0.5% marginal is consistent with that. They are two
different lenses on the same fact.

## The degenerate rows are the real story

Look at the bottom half of the table:

| model                  | tokens   | buckets | std-in | r   | slope | intercept | degen |
|------------------------|---------:|--------:|-------:|-----|-------|-----------|-------|
| gpt-5                  | 850,661  | 170     | 0      | —   | —     | —         | yes   |
| gemini-3-pro-preview   | 154,496  | 37      | 0      | —   | —     | —         | yes   |
| gpt-5.1                | 111,623  | 53      | 0      | —   | —     | —         | yes   |
| claude-sonnet-4.5      | 105,382  | 37      | 0      | —   | —     | —         | yes   |
| claude-sonnet-4        | 53,062   | 26      | 0      | —   | —     | —         | yes   |

Five models with `std-in = 0`. That means in every single active hour-bucket
where these models produced output, **the recorded input_tokens was zero**.
Pearson is undefined when one variable has no variance, hence `degen=yes`.

This is not a bug in the lens. It is a fingerprint of an upstream writer
that does not populate `input_tokens` — most likely because the call went
through a cached or pre-billed path where the prompt token count was
attributed to a different row, or because the model is exposed through a
provider whose response envelope omits the field entirely. Either way, the
lens has surfaced a **data-quality cliff**: 323 buckets across five model
labels (170 + 37 + 53 + 37 + 26) where the input axis is silently zero, and
1.27 M output tokens floating without an input pair.

That is 5.6% of the model groups by count, and a tiny fraction of total
mass — but if I wrote a cost dashboard that joined input mass to output
mass per model, those five models would silently disappear from the join
or, worse, show as infinitely cheap per output token. The lens has
already paid for itself this week by making that visible.

## Why correlation, not regression alone

I keep seeing operators reach for slope and ignore r. The slope tells you
the local sensitivity; r tells you whether the slope is meaningful. A
slope of 0.005 with r = 0.819 (gpt-5.4) is something you can plan around.
A slope of 0.011 with r = 0.938 across 9 buckets (sonnet-4.6) is
something you should ignore until you have more data. A slope of 0.004
with r = 0.493 (opus-4.6.1m) is a slope you should hedge — half the
variance is somewhere else, and that "somewhere else" is exactly where
your monthly bill will surprise you.

This is also why the lens reports both numbers in the same row. Reading
either one in isolation is operator malpractice. The lens does not let
you do that — it forces the eye across both columns, and that is good
design.

## Bucket vs row: do not confuse them

The 0.571 global r is a property of **hour-buckets aggregated across all
models**, not a property of API calls. Two corollaries:

1. A model with bursty hourly traffic (lots of tokens in a few hours,
   nothing in most) will tend to have a higher bucket-level r than its
   row-level r, because aggregation smooths noise. `gpt-5.4` at 402
   buckets and r = 0.819 is partly benefiting from this.
2. A model whose hours are roughly uniform but whose individual calls
   have wildly variable input/output ratios — say, a refusal in call 1
   and a 50K-token expansion in call 2 — will have its row-level
   variance hidden by the bucket-level mean. The bucket-level r will
   look optimistic.

So do not turn around and price an individual API call using the global
slope. Use the slope for hourly capacity planning; use distinct row-level
data (the `prompt-size` and `output-size` lenses, which report
distributions per call) for per-call budgeting.

## What I plan to do with this number

Three concrete next steps, all small:

1. **Find the writer that drops `input_tokens` for the five degenerate
   models.** The dropped count `0 zero/non-finite tokens` in the snapshot
   header confirms the rows are not being filtered out at ingest — they
   are being **written** with `input_tokens = 0`. Suspects: the
   provider-side response envelope for previewed gpt-5.x and
   gemini-3-pro-preview, which I should trace by sampling 10 raw
   `queue.jsonl` lines per model.
2. **Stop using r in isolation when budgeting.** Going forward, every
   cost-by-model alert in my dashboard family will include r² and
   slope side by side. The single number was never enough.
3. **Treat the 0.5% marginal as a planning constant** — but only for the
   three high-mass models that share it. For haiku and the small-sample
   sonnet rows, the constant is different and the sample is too thin to
   trust. The dashboard should partition on group identity, not assume
   homogeneity.

## A note on what this lens does not say

It does not say that prompt size **causes** output size. The 0.5%
marginal could be entirely a behavioural artefact: I tend to ask longer
questions to bigger contexts, and bigger contexts tend to produce
slightly longer summaries because there is more material to compress.
Pearson is a correlation metric, not a causal one. If I switched my
operator behaviour tomorrow — say, started feeding the same 100K-token
brief to every model and asking "yes or no" — the slope would collapse
toward zero and the r would collapse with it.

That is a feature of the data, not a defect of the metric. The lens
faithfully reports what the queue contains. The queue contains my
behaviour. The two are inseparable, and any analytics post that pretends
otherwise is selling something.

## Summary

- Snapshot: `pew-insights prompt-output-correlation` at
  `2026-04-25T19:06:44.237Z`, 908 active buckets, 8.75 B total tokens.
- Global Pearson r = **0.571**, global slope = **0.006**, r² = **0.326**.
- The three highest-mass models (opus-4.7, gpt-5.4, opus-4.6.1m) all
  share a marginal output-per-input ratio of **0.004 to 0.005**, which
  is the planning constant.
- Output mass is **1.13%** of input mass aggregated across the window —
  this is a summarisation workload, not a generation workload.
- Five model labels are degenerate (`std-in = 0`) covering 323 buckets;
  this is a writer-side data-quality bug, not a lens bug, and it would
  have silently broken any per-model cost join.
- Always read r and slope together. Either alone is operator
  malpractice.
