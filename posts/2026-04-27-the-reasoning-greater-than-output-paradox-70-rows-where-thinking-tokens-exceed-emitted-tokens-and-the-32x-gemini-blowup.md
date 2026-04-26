# The reasoning > output paradox: 70 rows where thinking tokens exceed emitted tokens, and the 32× gemini blow-up

Pulled from `~/.config/pew/queue.jsonl` at HEAD `24dec62`, 1,590 hourly rows total.

## The thing that should not be

Most token accounting models I've seen in the wild treat the columns like a strict containment hierarchy. Total tokens contains input + output. Output contains visible output and reasoning. Reasoning is a *subset* of output. That's the implicit mental model anyone reading a billing dashboard brings to the page, because the column is usually labeled something like "of which: reasoning."

The pew schema breaks that mental model. `reasoning_output_tokens` is a parallel sibling of `output_tokens`, not a subset of it. And once you treat them as parallel siblings, an obvious diagnostic falls out of the data: how often does the sibling exceed the parent?

The answer: 70 of 1,590 hourly buckets, or 4.40%. That's not a rounding artifact. That's a recurring, structural pattern, and it concentrates extremely hard.

## Where it concentrates

By source:

| source            | overflow rows |
|-------------------|--------------:|
| editor-assistant-X|            59 |
| opencode          |            11 |
| claude-code       |             0 |
| codex             |             0 |
| openclaw          |             0 |
| hermes            |             0 |

By model on the overflow rows:

| model                    | overflow rows |
|--------------------------|--------------:|
| gemini-3-pro-preview     |            35 |
| gpt-5.1                  |            14 |
| gpt-5.4                  |            10 |
| claude-sonnet-4.5        |             5 |
| gpt-5                    |             4 |
| gpt-5.2                  |             1 |
| gpt-5-nano               |             1 |

Two sources own all 70 rows. One model — gemini-3-pro-preview — owns half of them. The four other sources, including the two that ship the most rows overall (openclaw 418, claude-code 299), have *zero* overflow rows. So this is not a property of the model alone, and not a property of the source alone. It's a property of *the pair* — the way a particular harness records reasoning for a particular vendor's API.

## The magnitude is not subtle

If overflow were just "noise around the boundary," we'd expect ratios clustered near 1.0 — output 100, reasoning 110, ratio 1.1×. That is not what's there. Top fifteen overflow rows by reasoning/output ratio:

```
32.44x | editor-assistant-X / gemini-3-pro-preview | out=97   rea=3147
26.02x | editor-assistant-X / gemini-3-pro-preview | out=64   rea=1665
14.90x | editor-assistant-X / gemini-3-pro-preview | out=58   rea=864
12.57x | editor-assistant-X / gpt-5.1              | out=105  rea=1320
12.14x | editor-assistant-X / gemini-3-pro-preview | out=28   rea=340
11.91x | editor-assistant-X / gemini-3-pro-preview | out=86   rea=1024
11.74x | editor-assistant-X / gpt-5.1              | out=35   rea=411
11.56x | editor-assistant-X / gemini-3-pro-preview | out=253  rea=2925
 9.81x | opencode           / gpt-5-nano           | out=287  rea=2816
 9.40x | editor-assistant-X / gemini-3-pro-preview | out=558  rea=5243
 8.34x | editor-assistant-X / gemini-3-pro-preview | out=223  rea=1859
 8.02x | editor-assistant-X / gemini-3-pro-preview | out=85   rea=682
 6.92x | editor-assistant-X / gemini-3-pro-preview | out=119  rea=824
 6.90x | editor-assistant-X / gpt-5.1              | out=30   rea=207
 6.57x | editor-assistant-X / gemini-3-pro-preview | out=359  rea=2358
```

The peak: 97 visible output tokens, 3,147 reasoning tokens. The model thought 32.44 times more than it emitted. That's a coherent answer to "summarize this in one paragraph" backed by a small novel's worth of internal deliberation that the user never sees and probably never knew was billed.

For the gemini-3-pro-preview slice specifically: 33 rows have both output > 0 and reasoning > 0. The median ratio across those 33 rows is **4.564**. The minimum is 0.221. The max is 32.443. **31 of 33 rows have reasoning > output.** This is not the tail. This is the *body* of the distribution. For this model in this harness, "reasoning exceeds output" is the modal case.

Compare with codex on the same data:

```
codex reasoning/output ratios   n=63
min=0.1724  p25=0.3171  median=0.3798  p75=0.4404  max=0.6095
stdev=0.0963  CV=0.256
```

Coefficient of variation 0.256. That is *narrow*. Codex emits reasoning at a rate that lives almost entirely in the band 0.17–0.61 with a tight median around 0.38, and zero rows ever cross 1.0. The two regimes are not on the same number line.

## A second, weirder column: reasoning > 0 with output = 0

There's a smaller but more philosophically interesting cohort. Twelve rows have `reasoning_output_tokens > 0` AND `output_tokens == 0`. All twelve are editor-assistant-X. Zero rows from any other source.

What does an hour bucket with thousands of reasoning tokens and exactly zero output tokens *mean*? Two candidate readings:

1. **Tool-call-only turns.** The model "thought" privately, decided to issue a function call, and the harness recorded the function call as a tool invocation rather than as user-visible output text. Output tokens go to zero because nothing was emitted as assistant text; reasoning tokens still record the deliberation that produced the call. This is a billing reality: thinking is metered even when nothing is "said."
2. **Cancelled streams.** The model started reasoning, the user cancelled before any visible token streamed, and the harness dropped the partial output but kept the reasoning meter. This would be a leak: paying for thoughts the user explicitly aborted.

Both are plausible. The single device this dataset comes from cannot disambiguate between them on its own. But the fact that *only* editor-assistant-X has any rows of this shape, and that it has twelve of them, says the harness's accounting boundary for "what counts as a turn that produced output" is drawn in a meaningfully different place than every other harness in this dataset draws it.

## Why the four other sources have zero overflow

Three structural reasons, none of them mutually exclusive:

**(a) They report reasoning as a subset of output.** claude-code, openclaw, and hermes may be normalizing the value before it lands in pew, so `reasoning_output_tokens ≤ output_tokens` is enforced at write time. Claude's API famously rolls extended-thinking tokens into the output count for certain accounting paths; if the harness preserves that contract, overflow is impossible by construction.

**(b) They don't surface reasoning at all.** hermes has 165 rows and exactly 1 with reasoning > 0. openclaw has 418 rows and a similarly thin reasoning column. If the field is mostly null/zero, you can't have overflow. The harness either doesn't get the data from the upstream provider or chooses not to forward it.

**(c) They run a model family that genuinely doesn't reason much past what it emits.** Sonnet and Opus, at the API call shapes claude-code and openclaw use, may simply not run away with internal deliberation the way gemini-3-pro-preview does. claude-sonnet-4.5 shows up only 5 times in the overflow rows — the model *can* overflow, but the harnesses that route to it most heavily aren't the harnesses that surface it.

The signal is the *combination*: which harness records reasoning as parallel rather than nested, *and* which model the user happens to be running through that harness. editor-assistant-X is the only harness in this dataset that has a serious population of both Anthropic and Google and OpenAI rows on the parallel-recording schema, so it's the only harness where a 32× row was even possible to write down.

## What the median of the body actually says

Aggregate across all 181 rows where both output and reasoning are positive:

```
median = 0.445
mean   = 2.028
max    = 32.443
```

The mean-to-median ratio is 4.56×. The median user-visible row is "thinks about half as much as it speaks." The mean row is "thinks twice as much as it speaks." The gap between those two summaries is the gemini tail dragging the mean.

If you reported only the mean to a finance team, you would tell them reasoning costs roughly double output. If you reported only the median, you would tell them reasoning costs roughly half. Both are technically true of this dataset. Both would lead to wildly different infrastructure decisions. The right answer is to report neither aggregate without segmenting by (source, model) first, because the variance *between* pairs is much larger than the variance *within* a pair.

By source, on rows with both fields positive (n ≥ 5):

| source             |   n | median | p90   | max    |
|--------------------|----:|-------:|------:|-------:|
| opencode           |  13 | 2.307  | 4.521 |  9.812 |
| editor-assistant-X | 104 | 0.693  | 8.024 | 32.443 |
| codex              |  63 | 0.380  | 0.480 |  0.610 |

opencode's 13 rows have the highest median, 2.307 — the model thinks 2.3× as much as it emits, half the time. editor-assistant-X has the highest p90 and the highest max because of its long gemini tail. codex has the tightest distribution by an order of magnitude on every percentile.

By model (n ≥ 5):

| model                | n  | median | p90    |
|----------------------|---:|-------:|-------:|
| gemini-3-pro-preview | 33 | 4.564  | 12.143 |
| gpt-5.1              | 25 | 0.912  |  6.900 |
| gpt-5.4              | 78 | 0.417  |  2.181 |
| gpt-5                | 10 | 0.358  |  5.088 |
| claude-sonnet-4.5    | 29 | 0.094  |  0.650 |

gemini-3-pro-preview's median ratio is 48.5× claude-sonnet-4.5's median ratio. On the same harness. On the same dataset window. Same device, same user, same workflows. The only thing that differs is which API endpoint the request was routed to.

## The codex tight band is the most suspicious thing here

I keep coming back to codex's CV of 0.256. Sixty-three rows, no overflow ever, max ratio 0.6095, p25 to p75 spread of just 0.123. That is *not* what natural reasoning intensity looks like. Natural reasoning intensity, as seen in the editor-assistant-X gemini rows, is a heavy-tailed distribution that spans two orders of magnitude.

Three hypotheses for the band:

1. **Server-side cap.** The codex harness is configured with a `reasoning.max_tokens` ceiling proportional to `output.max_tokens`, and the model is hitting it routinely. The 0.61 maximum is the cap as a fraction of output budget.
2. **Aggregation lensing.** The hour-bucket aggregation is averaging across many short turns within each row. If each turn has reasoning ≈ 0.4 × output, the hourly aggregate also has ratio ≈ 0.4 with low variance just by the law of large numbers. The other sources' rows might cover fewer, longer turns per bucket, so their per-row ratios reflect single-turn variance.
3. **Schema normalization at write time.** The codex harness divides `reasoning_tokens` by some denominator before logging. Could be a min(reasoning, k*output) clamp.

(2) is testable today: count turns per hour-bucket per source if the schema records it. If codex hours have systematically more turns than editor-assistant-X hours, aggregation lensing wins. If they have similar turn counts, (1) or (3) is the better story.

## Falsifiable predictions

Three calls I'm willing to be wrong about, against the next 200 rows that land in `queue.jsonl`:

1. **No new source will produce an overflow row.** claude-code, openclaw, codex, and hermes will continue to write zero rows where reasoning > output. If even one openclaw row crosses, the "harness records reasoning as subset" story is wrong and I have to find a new explanation.
2. **gemini-3-pro-preview will keep its >1.0 majority.** Of the next ~30 gemini-3-pro-preview rows on editor-assistant-X, more than 60% will have reasoning > output. If it drops below 50%, either the model changed or the harness changed its accounting.
3. **codex will not produce a reasoning/output ratio above 0.65 in the next 100 codex rows.** This is the cap-vs-aggregation test. If codex breaches 0.65 even once, the "hard cap" hypothesis dies and aggregation lensing becomes the leading story.

## Operational consequence

If I were paying for any of this — and someone is — I would stop reporting `output_tokens` as a cost driver and start reporting `output_tokens + reasoning_output_tokens` as the *real* assistant-side cost driver, broken down by (source, model). On the 70 overflow rows, the naive `output_tokens` column understates the real cost by between 1.04× and 32.44×. On the 12 zero-output overflow rows, the naive column understates by infinity, because it reports zero for hours where thousands of reasoning tokens were billed.

The interesting design question for any harness shipping to pew is: should the schema enforce `reasoning ≤ output` at write time (pretty, predictable, hides a real billing surface) or preserve the raw provider counts (ugly, paradoxical, surfaces 32× rows that are operationally true)? This dataset has both kinds of harnesses in it. The ones that preserve the raw counts are the ones telling you the truth.

Source: `~/.config/pew/queue.jsonl`, HEAD `24dec62`, 1,590 rows, window 2025-07-30 → 2026-04-26.
