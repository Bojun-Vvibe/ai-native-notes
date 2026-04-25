# The 1M+ prompt tier: 47.7% of rows, and the gpt-vs-claude shape divergence

## Lede

Almost half of all priced inference rows in the trailing window are
sending more than a million input tokens per request. Forty-seven point
seven percent. That's not a tail — that's the modal bin. And when you
break the same distribution down per-model, the two top providers
disagree about whether the 1M+ tier is even normal: one provider has
68% of its rows there, the other has 27%. Same workload, same operator,
same week. The shape divergence is the story.

## The data

Snapshot from `pew-insights prompt-size`, run against
`~/.config/pew/queue.jsonl` at `2026-04-25T18:29:53Z`, 1,147 priced
rows, total input volume 3,383,318,008 tokens (3.38 billion), mean
2,949,711 tokens per row, max 55,738,577:

```
overall input_tokens distribution
bucket     rows  share
---------  ----  -----
0–4k       1     0.1%
4k–32k     61    5.3%
32k–128k   104   9.1%
128k–200k  49    4.3%
200k–500k  189   16.5%
500k–1M    196   17.1%
1M+        547   47.7%

per-model prompt-size summary (sorted by row count desc)
model               rows  mean       p95         max         0–4k  4k–32k  32k–128k  128k–200k  200k–500k  500k–1M  1M+
------------------  ----  ---------  ----------  ----------  ----  ------  --------  ---------  ---------  -------  ---
gpt-5.4             442   2,980,372  9,637,773   29,634,948  ·     2       9         6          33         90       302
claude-opus-4.7     421   3,241,946  22,660,856  55,738,577  1     46      73        23         90         75       113
claude-opus-4.6.1m  182   3,330,720  14,674,276  28,001,329  ·     11      14        12         17         21       107
unknown             56    326,116    527,137     1,401,547   ·     2       2         6          42         3        1
claude-haiku-4.5    31    2,137,574  6,805,993   7,800,557   ·     ·       1         1          4          4        21
claude-sonnet-4.6   9     1,104,858  2,974,411   2,974,411   ·     ·       ·         ·          3          3        3
claude-opus-4.6     4     85,940     139,952     139,952     ·     ·       3         1          ·          ·        ·
gpt-5-nano          1     37,807     37,807      37,807      ·     ·       1         ·          ·          ·        ·
gpt-5.2             1     90,545     90,545      90,545      ·     ·       1         ·          ·          ·        ·
```

Dropped: 327 zero-input rows (replays, no-op queue items, or
empty-prompt artefacts), 0 below the at-least floor.

## What it means

### The 1M+ bucket is not the tail

If you read the overall distribution top-to-bottom, the natural
instinct is to treat the 1M+ row as a tail — a big-number outlier
that probably reflects a few pathological prompts. That instinct is
wrong here. 547 of 1,147 rows are 1M+. That's 47.7% — the largest
single bucket and almost twice the size of the next bucket
(`500k–1M` at 17.1%). The mode of the distribution is the 1M+ tier.
Half the workload, by request count, is operating at or above the
million-token mark.

That has architectural consequences. A workload whose modal
prompt-size is 1M+ is not a "chat" workload and it is not an
"autocomplete" workload. It is a *long-context-as-a-feature*
workload — prompts that are themselves multi-megabyte assemblies,
probably involving full-file context, long histories, or large
retrieval payloads. Pricing, latency, and cache behaviour all
behave differently in that regime. A reasoning model that processes
2.9M tokens per request on average is doing something fundamentally
different from one processing 30k tokens per request, even when the
output token counts are similar.

The total volume number — 3.38 billion input tokens across 1,147
rows in roughly a week — is the same point in absolute terms. That
is not a humanly authored stream of prompts. It cannot be. The
mean of 2.95M tokens per row is on the order of a 7,000-page novel
of context per request. The only way to reach those numbers is by
machine-assembled context: tool results, retrieval bundles,
multi-file pulls, conversation transcripts pasted forward. The
system is not really sending "prompts" — it is sending working
sets.

### The gpt-vs-claude shape divergence

This is the part that surprised me. The two top-volume models look
almost identical at the top-line:

- `gpt-5.4`: 442 rows, mean 2.98M, p95 9.64M, max 29.63M.
- `claude-opus-4.7`: 421 rows, mean 3.24M, p95 22.66M, max 55.74M.

The means are within 9%. The medians (not shown, but implied by the
bucket distribution) are not far off either. The two providers look
like they are getting the same kind of work.

But the shape underneath is *not* the same. Look at the per-bucket
breakdown:

| bucket    | gpt-5.4 share | opus-4.7 share |
|-----------|--------------:|---------------:|
| 0–4k      |          0.0% |           0.2% |
| 4k–32k    |          0.5% |          10.9% |
| 32k–128k  |          2.0% |          17.3% |
| 128k–200k |          1.4% |           5.5% |
| 200k–500k |          7.5% |          21.4% |
| 500k–1M   |         20.4% |          17.8% |
| 1M+       |         68.3% |          26.8% |

The two providers disagree, sharply, about where most of their work
sits. `gpt-5.4` puts 68.3% of its rows in the 1M+ tier. `claude-opus-4.7`
puts 26.8%. The opus distribution is much more spread out — 35.4% of
opus rows are in the 4k–500k range, vs only 11.4% for gpt-5.4. Opus is
handling a wider variety of request sizes; gpt-5.4 is handling almost
exclusively large ones.

The means converge because opus *also* has a much heavier extreme
tail. Opus's max is 55.7M (almost 2× gpt-5.4's 29.6M max). Opus's p95
is 22.7M (2.4× gpt-5.4's 9.6M p95). So the mean equilibrates not
because the distributions are similar but because they are
asymmetrically skewed in opposite ways: gpt-5.4 has a fat *body* in
the 1M+ band, opus has a fat *tail* above 22M.

That asymmetry tells you something about the routing layer's
implicit policy. The router appears to be sending:

- Mid-context jobs (4k–500k) preferentially to opus.
- Large-but-bounded jobs (500k–10M) preferentially to gpt-5.4.
- Truly enormous jobs (>20M) almost exclusively to opus.

The mid-context preference is probably a "claude is good at code-edit
shaped prompts" heuristic. The huge-context preference is probably a
context-window capacity thing — `claude-opus-4.6.1m` (the long-context
variant) sits at 182 rows with 58.8% of those in 1M+, suggesting the
operator deliberately reaches for the 1M-context family when the
prompt is over a million. The middle band going to gpt-5.4 is the part
worth investigating: is gpt-5.4 actually the right choice for a 5M
context, or is it just where the router defaults when the prompt is
"too big to be cheap, too small to need the explicit 1M endpoint"?

### Cost composition implication

Pair this with the sibling cost-concentration snapshot for the same
window: `claude-opus-4.7` accounts for 79% of priced spend on 49.5%
of priced events. The prompt-size distribution explains *why*: opus
has the much heavier extreme tail (a p95 that is 2.4× gpt-5.4's), and
extreme-tail rows dominate input token volume nonlinearly. A single
55.7M-token request is worth as many dollars as roughly 11,000
sub-5k requests at the same per-token rate.

So the cost concentration is not really a "model is more expensive"
story. It is a "model is asked to handle the giant prompts" story.
Per-token, opus is 1.76× the price of gpt-5.4. Per-event, opus
appears to be more like 4× the price of gpt-5.4. The other ~2.3×
multiplier comes from the routing layer asking opus to handle
prompts whose mean and p95 input sizes are larger. If you wanted
to move the cost needle, the lever is not "switch models" — the
lever is "send the >20M-token jobs to a cheaper model" or "shrink
the >20M-token jobs". Both are more architecturally invasive than
flipping a routing weight.

### The `unknown` model is anomalously *small*

Easy to miss in the per-model table: the `unknown` row has 56
rows, mean 326,116, max 1.40M. That is the smallest mean of any
non-trivial model in the table. While the priced models are
operating in the 2–3M-token-per-row regime, whatever `unknown`
is, it is operating an order of magnitude below that. 42 of its
56 rows sit in the 200k–500k bucket — a single bucket carrying
75% of the model's rows.

That tight clustering suggests `unknown` is one specific consumer
or one specific tool, not a heterogeneous mix of unidentified
calls. A heterogeneous mix would smear across buckets. A 75%-in-one-bucket
shape implies a single producer with a stable prompt template
that lands consistently in the 200k–500k band. Worth tracing:
this is probably one named source whose model-id failed to propagate
through the queue logger. Once identified, it'll fold into one of
the existing model buckets and the cost report's "unpriced events"
yellow alert (see sibling post) goes away.

### What this is not

This is not evidence of context-window abuse. A 1M+ prompt is not
inherently wasteful — it depends entirely on whether the model is
*using* the long context or just being handed it. A dispatch that
sends 5M tokens of context to extract a 200-token answer might be
the right call (the alternative is multi-pass retrieval, which can
be slower and more expensive end-to-end). The prompt-size
distribution alone cannot tell you whether the long contexts are
earning their keep. You'd need to cross-reference with the
output-size distribution and ideally with downstream task-success
signals.

It is also not evidence that the operator is doing something
unusual. A workload whose modal prompt is 1M+ is increasingly
ordinary in the long-context-as-a-feature category of agent tools.
What's interesting is the *shape divergence* between providers
within that workload — the 68% vs 27% split in the 1M+ tier — which
is a property of the routing layer, not the workload.

## Caveats / what would falsify

The 327 zero-input rows that got dropped are not investigated here.
If those represent a class of legitimately small requests (e.g.
tool-only no-prompt invocations), then the overall distribution is
skewed toward the large end by selection. The 1M+ share of 47.7%
should be read as "47.7% of *priced, non-zero-input* rows", not
"47.7% of all queue events".

The per-model shape comparison assumes that the routing layer is
deterministic enough that the assignment of prompt-sizes to models
is meaningful. If the router is doing on-the-fly load-balancing or
fallback, then the gpt-5.4 vs opus shape difference might be a
function of which model was *available* at request time, not which
was *chosen*. A retry-driven fallback regime would smear the shape
distinction. Worth checking against any retry-rate telemetry.

The `claude-opus-4.6.1m` row (182 events, 58.8% in 1M+) is excluded
from the head-to-head shape analysis because its presence muddies
the "opus vs gpt-5.4" framing — but if you fold its rows back into
opus, opus's overall 1M+ share rises from 26.8% to roughly 36.4%.
Still well below gpt-5.4's 68.3%, but the gap narrows. Whether to
treat the 1M-context variant as "opus" or as a separate tier is a
modeling choice; the post takes the conservative approach of
treating it as separate, which makes the shape divergence look
larger than it would under the alternative bookkeeping.

Falsification paths: (a) next week's snapshot showing the gpt-5.4
1M+ share dropping below 50% without a workload change would imply
the routing policy moved, not the workload; (b) the unknown bucket
clustering breaking apart (e.g. spreading evenly across buckets)
would imply it was always a heterogeneous bucket, not a single
hidden producer; (c) the overall 1M+ share dropping below 30%
would invalidate the "modal bin is 1M+" framing and suggest this
week was an outlier.

## Citations

- `pew-insights prompt-size` snapshot, captured `2026-04-25T18:29:53.589Z`,
  source `~/.config/pew/queue.jsonl`, 1,147 priced rows, 327 zero-input
  drops, 3,383,318,008 input tokens total.
- Tool: `pew-insights@0.5.4`, built from `~/Projects/Bojun-Vvibe/pew-insights`,
  invoked as `node dist/cli.js prompt-size`.
- Sibling post: `2026-04-26-cost-concentration-79-percent-on-one-model-and-the-63-percent-cache-rebate.md`
  — same week, dollar view of the same routing imbalance.
- Earlier related post: `2026-04-25-tok-per-bucket-vs-tok-per-span-hour-orthogonal-density.md`
  — token volume framed by time density rather than per-row size.
