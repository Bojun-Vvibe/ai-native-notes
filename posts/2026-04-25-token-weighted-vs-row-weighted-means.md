# Token-Weighted vs Row-Weighted Means: When Aggregation Picks Sides

A ratio reported across a population is not one number. It is at
least two, and the gap between them is itself a signal. The
`output-input-ratio` subcommand shipped in pew-insights v0.4.49
(commit `2275d84`, bumping 0.4.48 → 0.4.49) makes this concrete by
emitting both a `ratio` and a `mean-row-ratio` for every model in
the same row. The numbers are rarely close. On a real fleet
window of 1,067 rows and 3.33 billion input tokens, `claude-opus-4.7`
landed at:

```
ratio           = 0.0167   (token-weighted: sum(output) / sum(input))
mean-row-ratio  = 0.0925   (unweighted mean of per-row ratios)
```

That is a 5.5× gap, on the same model, in the same window, on the
same population. Neither number is wrong. They answer different
questions, and treating them as interchangeable is how dashboards
end up lying to operators.

## The two ratios

For a population of `n` rows, each with `inputTokens[i]` and
`outputTokens[i]`, the two natural aggregates are:

```
token_weighted = sum(output) / sum(input)
mean_row       = (1/n) * sum(output[i] / input[i])
```

`token_weighted` answers: **per token of input that hit the
fleet, how many output tokens came back?** It is the right number
to use when you care about throughput, billing, or hardware sizing.
A row that consumed a million input tokens gets a million votes; a
row that consumed a thousand gets a thousand. The denominators are
real bytes that real GPUs processed.

`mean_row` answers: **for a typical call, what verbosity does this
model exhibit?** Each call is one vote. A row with a 3-token prompt
and a 30-token response counts the same as a row with a 1M-token
prompt and a 10k-token response. The denominators are calls, not
tokens.

These are different aggregations of the same underlying sample. They
collapse to the same number only when every row has the same input
size — which is almost never true in production.

## Why the gap is the signal

For `claude-opus-4.7` at 0.0167 vs 0.0925, the row-mean is dragged
**up** by many short prompts that returned proportionally large
completions, while the token-weighted figure is dragged **down** by
a handful of enormous prompt-cached rows where a million-token
context returned a few hundred tokens of "yes, here is the diff".
Both behaviours coexist in the same model's call distribution.

Compare to `gpt-5.4` in the same v0.4.49 live-smoke output:

```
ratio           = 0.0053
mean-row-ratio  = 0.0060
```

The two collapse to ~0.006. That tells you something the dashboard
headline cannot: **gpt-5.4's verbosity is uniform across its call
distribution**. The token-weighted view and the row-mean agree
because there is no per-call regime difference to disagree about.
Whether you measure by tokens or by calls, you get the same picture.

A single number cannot encode this. The ratio between the two
aggregates encodes it almost for free.

## Worked example: why the population matters

Suppose a model is called 100 times. Ninety of the calls have a
1,000-token prompt and a 200-token completion. Ten of the calls
have a 1,000,000-token prompt and a 1,000-token completion (think
context-stuffed code search).

```
sum(input)  = 90 * 1,000     + 10 * 1,000,000 = 10,090,000
sum(output) = 90 * 200       + 10 * 1,000     =     28,000

token_weighted = 28,000 / 10,090,000 = 0.00278
mean_row       = (90 * 0.20 + 10 * 0.001) / 100 = 0.1801
```

Same population, two answers that differ by 65×. The
token-weighted view says "this model is terse: less than 0.3% of
input volume comes back". The row-mean view says "this model is
chatty: a typical call returns 18% of its input as output". Both
are statements of fact about the same 100 rows. The disagreement
encodes the bimodality of the call distribution.

A monitoring system that reports only one of these numbers will
make systematically wrong decisions. Pick token-weighted and you
will under-budget for a future workload of mostly-short prompts.
Pick row-mean and you will catastrophically under-budget for a
workload that adds even a few large-prompt calls.

## What pew-insights actually emits

The `ModelRatioRow` type carries both fields explicitly. From the
v0.4.49 release notes:

> Two ratios surfaced per model:
>
> - `ratio` = token-weighted: `sum(output) / sum(input)`. A
>   handful of long completions can dominate this number.
> - `mean-row-ratio` = mean of per-row ratios. Equally weights
>   every call. The gap between the two surfaces whether the
>   verbosity signal is concentrated in a few outlier rows or
>   is the model's typical behaviour.

The framing in the changelog matters. It does not pick a winner; it
ships both and tells the reader to read the **gap**. That is the
right design pattern for any aggregate metric on a heavy-tailed
population.

The same pattern shows up elsewhere in the family. `cache-hit-ratio`
emits `cachedInputTokens / inputTokens` per row and aggregates
both ways. `cost` aggregates dollar-weighted, but a per-row
unweighted view would expose whether the spend signal is
concentrated in a few sessions. `concurrency` reports both
mean-active-streams and p95-active-streams for the same reason —
the headline number is not the only honest summary.

## The denominator decision is a stance

Every aggregate metric implicitly answers a denominator question:
**what are we averaging over?** The answers fall into a small set:

- **Over rows / calls.** Every event is one vote. Good for "what
  is a typical interaction like?" Bad for capacity planning, where
  rare-but-huge events dominate real cost.
- **Over tokens / bytes / dollars.** Every unit of mass is one
  vote. Good for capacity, billing, throughput. Bad for "what does
  the user typically experience?" because a single large session
  can swamp a thousand small ones.
- **Over time buckets.** Every hour or day is one vote. Good for
  burst analysis and SLA reasoning. Bad for off-peak workloads
  where most hours are nearly empty.
- **Over users / devices / sources.** Every actor is one vote.
  Good for fleet-fairness reasoning. Bad when one actor produces
  most of the load, which is the usual case.

A "ratio" without a denominator policy is under-specified. The pew
convention — emit both token-weighted and row-weighted, sort by
mass, but never silently drop either — concedes the ambiguity
upfront and forces the reader to reason about which number they
need.

## Where the gap leads to bad decisions

Three concrete failure modes when only one aggregate is reported:

**Cost forecasting on token-weighted means.** If you forecast next
month's cost by multiplying expected input volume by the
token-weighted ratio, you will systematically under-predict for any
model whose `mean-row > token-weighted`. The token-weighted figure
is dragged down by the long-prompt rows; the marginal new call is
likelier to look like the row-mean. For opus at 0.0167 vs 0.0925,
forecasting on the token-weighted figure under-budgets output spend
by 5.5× per added call.

**Verbosity comparisons across models.** If model A reports a
token-weighted ratio of 0.05 and model B reports 0.10, the naive
read is "B is twice as chatty as A". But if A's row-mean is 0.20
and B's row-mean is 0.11, the call-level experience is the
opposite: A returns more output per call, on the typical call. The
token-weighted comparison rewards models that happen to be deployed
on long-prompt workloads.

**Anomaly detection on rolling means.** A short burst of
long-prompt rows can collapse a token-weighted rolling ratio
without changing the row-mean at all. The "anomaly" is then a
property of the workload mix, not of the model's behaviour. Alerts
on the wrong aggregate generate noise that operators learn to
ignore.

The cure is not to switch to "the right" aggregate. It is to track
both, alert on the **gap**, and treat divergence as a signal that
the call distribution has shifted under the metric.

## What to take home

A single mean is a stance, not a measurement. When the population
is heavy-tailed — and any production token-traffic distribution is
heavy-tailed — the token-weighted mean and the row-weighted mean
are answering different questions about the same data. The pew
v0.4.49 contract of always emitting both, and the v0.4.50
follow-on of breaking each down by `--by-source`, treats the gap
as a first-class signal rather than something to hide behind a
chosen denominator.

The implementation cost of "emit both" is roughly nothing: one
extra field on the row type, one extra column in the renderer, one
test case per builder. The interpretive cost of "emit one" is
permanent: every reader of the metric inherits the implicit
denominator decision and has to reverse-engineer it from the
column name. Most do not bother. They take the number at face
value, divide it into next quarter's budget, and find out about
the gap when the bill arrives.

The cheapest defence is to never let a ratio escape the builder
without its sibling, and to teach every consumer of the metric
that the absence of one is a missing column rather than a settled
question. A dashboard that ships only the token-weighted figure
has not chosen the right denominator; it has chosen to hide the
denominator decision from the reader, who then makes it
unconsciously and badly. If your dashboard renders one mean, render
the other beside it. If they agree, you have learned the
distribution is uniform and can collapse the view later. If they
disagree, you have learned the distribution is bimodal and you
need both. Either way, you have learned something the
single-number version was hiding.
