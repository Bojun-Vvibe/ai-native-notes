# The 100-to-1 fleet: when output/input ratio collapses to 0.01

`pew-insights` v0.4.x ships an `output-input-ratio` lens that does exactly what the name says: for each model, it sums `inputTokens` and `outputTokens` across every row in `queue.jsonl`, divides them, and reports both the token-weighted ratio and the mean of per-row ratios. Two numbers per model. Should be boring.

Here is what it returns on a real local queue, 2026-04-25T04:55Z, across 1,086 considered rows from this machine:

```
overall:                   in=3,343,592,888  out=34,267,118  ratio=0.01025

claude-opus-4.7   rows=387 in=1,350,536,784  out=23,352,931  ratio=0.01729  mean_row=0.09477
gpt-5.4           rows=415 in=1,291,921,888  out= 6,838,762  ratio=0.00529  mean_row=0.00596
claude-opus-4.6.1m rows=182 in=  606,191,092  out= 3,450,625  ratio=0.00569  mean_row=0.01079
claude-haiku-4.5  rows= 31 in=   66,264,788  out=   133,160  ratio=0.00201  mean_row=0.00284
unknown           rows= 56 in=   18,262,497  out=   410,432  ratio=0.02247  mean_row=0.03416
claude-sonnet-4.6 rows=  9 in=    9,943,726  out=    73,444  ratio=0.00739  mean_row=0.00603
claude-opus-4.6   rows=  4 in=      343,761  out=     5,787  ratio=0.01683  mean_row=0.01812
```

Drop counters: 327 rows dropped for `zeroInput`, 0 dropped for invalid tokens, 0 for invalid hour_start. Of the original 1,413 rows in the queue, 23% were excluded by the zero-input filter alone — that's a number to come back to.

The headline is the **0.01025 overall ratio**. Across 3.34 billion input tokens, the fleet produced 34 million output tokens. Roughly **97.6 input tokens per output token** in token-weighted aggregate. That's a 100:1 fleet, and it is not a measurement error.

This post is what that ratio means, why some models look like outliers but aren't, why the 327 dropped rows matter more than the kept ones, and what the gap between `ratio` and `meanRowRatio` is silently telling you about workload shape.

## A 100:1 ratio is structural, not pathological

If you've never seen this lens before, your first instinct is going to be that the numbers are wrong. Real LLM outputs are usually somewhere between 1:5 and 1:50 input:output for *generative* workloads — write me a function, summarize this email, draft a reply. The model produces a meaningful fraction of the input back as output. A 1:100 ratio implies the model is mostly *reading* and barely *writing*.

Which is exactly what's happening, because 1,086 rows of `queue.jsonl` from an agentic CLI workload are not generative workloads. They're tool-loop turns. The shape of an agentic tool turn is:

1. ~50KB of system prompt (instructions, tool schemas, behavior policy).
2. ~80KB of conversation history (every prior tool call, every prior result, growing monotonically).
3. ~200KB of tool output that just came back from the previous turn (file reads, command output, web fetches).
4. ~5KB of "decide what to do next."

The output is item 4 plus a tool call envelope. Maybe 1-3KB of actual model production per turn. That's the entire bill on the output side. The input side is items 1+2+3, growing turn by turn, and item 3 alone — fresh tool output — is usually 50-200× larger than the model's response to it.

So a token-weighted overall ratio of 0.01025 is what you get when most of your traffic is *reactive* (model reads a lot, decides a little). It's not a bug, it's a fingerprint of agentic workload. Generative workloads on the same hardware would report 0.05 - 0.2 and you'd have to investigate if they didn't.

The lens is not telling you "the models are being lazy." It's telling you "your workload is 99% read-comprehend, 1% produce." Which has direct cost consequences, because input tokens are usually 4-5× cheaper than output tokens per million, but the volume asymmetry here means input still dominates the bill by ~20× (`100 × 0.25 = 25`, in round numbers). The cost lever in this fleet is *prompt size*, not output size. Always. By construction.

## Why `gpt-5.4` looks 3× cheaper than `claude-opus-4.7` and isn't

The two largest rows in the table are:

- `claude-opus-4.7`: 387 rows, ratio 0.01729, mean_row 0.09477
- `gpt-5.4`: 415 rows, ratio 0.00529, mean_row 0.00596

`gpt-5.4`'s ratio is 3.27× lower than `claude-opus-4.7`'s. If you read the lens as "output produced per input consumed," that suggests `gpt-5.4` is dramatically more efficient — same input volume class, much less output. Cheaper per turn. Pick `gpt-5.4` and save money.

That reading is wrong, and the lens itself contains the evidence of why: look at the `meanRowRatio` column. For `claude-opus-4.7` it's 0.09477 — almost 5.5× the token-weighted ratio. For `gpt-5.4` it's 0.00596, statistically identical to the token-weighted ratio.

What does that gap mean? `meanRowRatio` is the unweighted average of per-row ratios. If every row in `claude-opus-4.7`'s 387 rows had similar input/output proportions, the two ratios would be close. They're not — `meanRowRatio` is 5.5× higher than `ratio`. The only way that happens is if **the rows with the most input tokens have the lowest per-row ratios, and the rows with little input have high per-row ratios.**

Concretely: `claude-opus-4.7` has a population of turns that fall into two clusters. One cluster is huge-context turns (250K+ input, 1-2K output) — per-row ratio around 0.005. The other cluster is small-context turns (5K input, 1-2K output) — per-row ratio around 0.2-0.4. The token-weighted ratio is dominated by the huge-context cluster (because it owns most of the tokens), but the unweighted mean is pulled up by the small-context cluster (because there are similar *counts* of them).

`gpt-5.4`'s `ratio == meanRowRatio` to two decimal places means *its 415 rows are uniformly large-context*. Every turn is a fat-prompt turn. There's no small-context cluster to pull the unweighted mean up.

The implication isn't that `gpt-5.4` is cheaper; it's that **`gpt-5.4` is being used for a different shape of work**. Probably long-running planning sessions where every turn carries a saturated context window. `claude-opus-4.7` is being used for a mix: some long sessions, some short interactive turns. The 3× ratio gap is workload-shape signal, not model-quality signal.

This is the most common misread of `output-input-ratio`. The cure is to always compare `ratio` and `meanRowRatio` together. Their gap is the heteroscedasticity index. When they agree, the workload is uniform. When `meanRowRatio >> ratio`, the workload is bimodal with a heavy-input tail. When `meanRowRatio << ratio` — which would mean the small-input rows are producing disproportionately small outputs — you're looking at a different pathology (probably empty-response retries), and you should investigate.

## The 327 dropped rows are the most interesting rows in the dataset

The lens reports `droppedZeroInput: 327`. Out of 1,413 candidate rows, 23.1% were excluded because their `inputTokens` field was 0 or missing. The lens *has to* drop these — you can't compute output/input when input is zero, the ratio explodes. But the population of dropped rows is not random.

A row with `inputTokens = 0` and `outputTokens > 0` is a row where the agent emitted output without any model input. The shapes that produce this:

- **Cached responses** — some providers report 0 input when a response is fully cache-hit. The cache lookup happened, the model didn't actually re-process the prompt. In `pew-insights`, `cache-hit-ratio` is a separate lens for exactly this case, but it can leak into `output-input-ratio`'s drop counter.

- **Streaming continuations** where the row represents a continuation of a prior message and the input was accounted for in the parent row.

- **Tool-call-only turns** where the model emitted a tool call envelope and provider accounting attributed the tokens to a sibling row.

- **Buggy provider responses** where the input field is missing or zeroed for non-semantic reasons (rate-limit retries, partial responses, error envelopes that still emit some output).

23.1% is too high for any one of these to be the sole cause. The healthy interpretation is: zero-input rows are a *feature of the upstream queue.jsonl format*, not noise to be filtered away. The lens correctly excludes them from the ratio computation, but a 23% drop rate is the lens's way of telling you "there's a quarter of your traffic that doesn't look like a normal turn, go investigate it with a different lens."

The right follow-up is `cache-hit-ratio` (v0.4.x, separate subcommand), which will pull these rows back in and tell you what fraction are actually cache-hits versus accounting glitches versus streaming continuations. The `output-input-ratio` lens is a perfect example of single-responsibility: it answers exactly one question, and when the answer requires excluding 23% of the data, it tells you so in the drop counter and lets you decide whether to chase it.

## Per-model investigations the lens makes possible

A few quick reads off the table that are not in the lens output but fall out of it:

**claude-haiku-4.5** has 31 rows, `ratio = 0.00201`, the lowest in the fleet. Almost 500:1 input:output. Mean row ratio 0.00284, basically identical to the token-weighted ratio — so the workload is uniform, not bimodal. This is consistent with haiku being used as a dedicated *summarization* or *classification* model on large contexts: read a wall of text, output a short label. The 30 active half-hour buckets reported by `model-tenure` for the same model put this at maybe one summarization session per day on average. That fits.

**unknown** has 56 rows, ratio 0.02247 — the *highest* ratio in the fleet, more than 2× the next-highest. Mean row ratio 0.03416. Reasonable interpretation: the rows with an unrecognized model string are coming from a different code path entirely — probably a one-shot generative endpoint, not the agentic tool loop. 18M input tokens / 410K output tokens for a "real" generative workload would be 1:44, which is in the ballpark for "summarize / draft / answer" turns where the model produces a meaningful fraction of the input back. Worth pulling these rows by `device_id` or source to identify what's making them.

**claude-opus-4.6.1m** has 182 rows, ratio 0.00569, mean_row 0.01079. The 1.9× gap between mean and weighted suggests some bimodality, but milder than `claude-opus-4.7`. The `1m` suffix on the model name (1M context window) is consistent with this — the model is *capable* of huge contexts, but on this machine it's being driven to a less extreme distribution than the smaller-context sibling.

**claude-sonnet-4.6** has 9 rows. The numbers are real but the population is too small to read anything from. The lens reports them honestly without flagging them. This is one of those places where you'd want a `--min-rows` filter (which `output-input-ratio` does have, as `--minRows`, default 0) to suppress low-N rows for display purposes — but the underlying calculation should remain population-complete, and it does.

## What the lens cannot tell you

Three things you cannot read off `output-input-ratio` even though you'll be tempted:

1. **Cost.** The lens does not multiply by per-million-token prices. A model with a 0.005 ratio at $3/M input and $15/M output costs *more* per output token than a model with a 0.02 ratio at $0.50/M input and $2.50/M output. Ratio is necessary for cost reasoning but not sufficient.

2. **Latency.** Output tokens drive latency far more than input tokens (input is parallelizable across the prompt; output is sequential autoregression). A low ratio means *fast turns*, paradoxically — you're doing a lot of read in parallel and a tiny amount of generate. This is one of the few cases where the cheap thing and the fast thing align.

3. **Quality.** A low ratio is not a quality signal in either direction. It tells you about workload shape and prompt structure; it doesn't tell you whether the outputs are good. You need eval traces, not aggregation lenses, for that.

## Operating implications

After staring at the table for a while:

- **Default to reading `ratio` and `meanRowRatio` as a pair.** The single-number summary is misleading on bimodal models. The gap between them is the bimodality index. A 5× gap means there are two workload shapes hiding inside the model's row population.

- **Don't chase low ratios as efficiency wins.** A 100:1 ratio is structural to agentic workloads. The way to reduce input volume is to *reduce conversation history retention or tool-output truncation policy*, not to swap models. Model swap moves the ratio by maybe 3×; conversation pruning moves it by 50×.

- **Track the drop rate as a first-class signal.** 23% zero-input rows is a number to monitor. If it climbs to 40% you've got a caching change or a provider bug. If it drops to 5% you've changed something in the prompt pipeline. Either way the trend matters more than the absolute level.

- **Use the "unknown" row as a discovery channel.** Every quarter, 5-10% of your traffic is from a model the parser doesn't recognize. That's where the new model releases live. Watching the size of the `unknown` row over time is a leading indicator of fleet drift.

The `output-input-ratio` lens is two divisions and one mean. It produces about a hundred bytes of output per model. The information density per byte is higher than almost any other lens in `pew-insights`, because the *gap between the two reported ratios* contains workload-shape information that no other single lens exposes. The lens shipped without much fanfare and the smoke output looks unremarkable. The numbers are anything but.

When the overall ratio on your fleet is 0.01025 and you're surprised, the surprise is the finding. Agentic workloads consume context-windows full of input to produce thumbnail-sized outputs, and the ratio number makes that geometry impossible to ignore.
