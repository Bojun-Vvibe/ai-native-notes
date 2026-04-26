# Output/Input Ratio Bimodality: The 23× Harness Gap and the 0.99 Tail

## The single number that distinguishes a tool-use harness from an inference loop

If you take every row in `~/.config/pew/queue.jsonl` (1,560 rows, schema `{source, model, hour_start, device_id, input_tokens, cached_input_tokens, output_tokens, reasoning_output_tokens, total_tokens}`) and compute one number per row — `output_tokens / input_tokens` — you get a distribution that looks nothing like the symmetric, single-peaked thing your stats intuition expects. It is sharply **bimodal**, the two modes are roughly **two orders of magnitude apart**, and which mode a row lands in is almost entirely determined by which CLI emitted it.

The pew-insights tooling at `pew-insights@9f39374` (`docs: changelog for v0.6.50 with refinement live-smoke output`) is now mature enough that we can lean on its refinement framework, but for this analysis a single pass over the queue file is enough. This post is about what that single pass reveals.

## The shape

Bucket `output / input` across all 1,233 rows where `input_tokens > 0`:

```
[0.000, 0.001):  145 rows  — 11.8%
[0.001, 0.005):  310 rows  — 25.1%   ← lower mode
[0.005, 0.010):  224 rows  — 18.2%
[0.010, 0.020):  171 rows  — 13.9%
[0.020, 0.050):   91 rows  —  7.4%
[0.050, 0.100):   71 rows  —  5.8%
[0.100, 0.200):  141 rows  — 11.4%   ← upper mode
[0.200, 0.500):   74 rows  —  6.0%
[0.500, 1.000):    6 rows  —  0.5%   ← extreme tail
[1.000, ∞):        0 rows
```

Two clear humps. The valley sits between 0.02 and 0.10, where only ~13% of rows live. To the left, output is ≤2% of input — i.e. the model is reading thirty to a thousand times more than it writes. To the right, output is 10-50% of input — the model is roughly writing a paragraph for every paragraph it reads.

There is no row where output exceeds input. The hard ceiling at 1.0 is real: across 1,233 rows the maximum observed ratio is **0.9933** (one row on `2026-04-24T00:00:00Z` from `opencode` running `claude-opus-4.7`, with 8,348 input tokens and 8,292 output tokens). Even the heaviest "writer" never out-writes its own context.

## The mode split is per-source, not per-model

Group by `source`, compute the percentile spread of the ratio, and the bimodality stops being a property of the population and becomes a property of which CLI emitted the row:

```
source         n     p10        p50        p90        mean
openclaw      404    0.0004     0.0021     0.0123     0.0045
codex          64    0.0012     0.0049     0.0083     0.0056
claude-code   299    0.0013     0.0064     0.0176     0.0096
hermes        162    0.0040     0.0375     0.2296     0.0833
opencode      298    0.0312     0.1262     0.2506     0.1323
ide-assistant-A 6  ~0.014                              0.0168
```

`openclaw` and `claude-code` together — the two largest sources — anchor the lower mode at p50 ≈ 0.002–0.006. `opencode` alone anchors the upper mode at p50 ≈ 0.126. The ratio of medians is **0.1262 / 0.0064 ≈ 19.7×** between `opencode` and `claude-code`, and **0.1262 / 0.0021 ≈ 60×** between `opencode` and `openclaw`. If you compare means, `opencode` (0.1323) vs `openclaw` (0.0045) is **29.4×**; vs `codex` (0.0056) is **23.6×**.

This is not a model effect. Every source above is at some point running on the same `claude-opus-4.7` weights:

```
opencode / claude-opus-4.7:   n=228  mean=0.1657  p50=0.1511  p90=0.2778  p99=0.6191
opencode / big-pickle:        n= 53  mean=~0.020  p50=0.0191  p90=0.0310
opencode / gpt-5.4:           n= 11  mean=~0.030  p50=0.0265  p90=0.0712
opencode / claude-sonnet-4.6: n=  4
```

Two things to notice in those four rows. First, even within `opencode` the model matters: `claude-opus-4.7` rows have p50 0.151, but `big-pickle` and `gpt-5.4` rows on the same harness sit closer to 0.02-0.03 — much closer to the lower mode. Second, the distinction between modes is not "Claude vs GPT" or "thinking vs non-thinking"; it is **harness-driven**. Take `claude-opus-4.7` itself and run it through different harnesses:

- `opencode / claude-opus-4.7` — p50 0.1511
- `claude-code / *` (which is mostly Opus-family) — p50 0.0064
- `openclaw / *` (Opus + GPT mix) — p50 0.0021

Same model, three different p50s spanning 70× — and that's the median, not the tail. The distribution is shaped by what the harness *does with the model's output*, not what the model is.

## Why the lower mode is so low

The lower mode is dominated by rows where input is enormous and output is a rounding error. Sorted by ratio ascending, excluding zero-output rows, the bottom 10 look like this:

```
ratio       source       model                     input        output
0.000020    claude-code  claude-sonnet-4-6           990,186     20
0.000020    claude-code  claude-haiku-4-5-20251001   441,414      9
0.000022    claude-code  claude-haiku-4-5-20251001 6,346,255    141
0.000024    claude-code  claude-haiku-4-5-20251001 1,905,492     45
0.000032    claude-code  claude-sonnet-4-6           632,360     20
0.000036    claude-code  claude-sonnet-4-6           704,061     25
0.000039    claude-code  claude-haiku-4-5-20251001 1,470,666     57
0.000041    claude-code  claude-haiku-4-5-20251001   172,269      7
0.000049    claude-code  claude-haiku-4-5-20251001   905,286     44
0.000167    openclaw     gpt-5.4                   1,378,127    230
```

These are not chat turns. A 6.3-million-token input that produces 141 output tokens is not "the user asked a question and the model answered." It is a tool-use loop where the model is being shown a giant context window — repository files, terminal scrollback, prior tool outputs — and is responding with a single short JSON-encoded tool call: "open this file", "search for that string", "yes". The output is structurally bounded because *the harness only needs the next move*, not prose.

`claude-code`'s use of Haiku is especially revealing: every Haiku row in the bottom 10 has a 7-141 token output. Haiku is being used as a planner-or-classifier: "given this 1-6 MB of context, emit one structured decision." The model is doing all of its work in attention; almost none of it leaves through the output channel.

## Why the upper mode is so high

The upper mode is `opencode / claude-opus-4.7` doing prose generation. The top 10 rows by ratio:

```
ratio    source     model              input    output    hour
0.9933   opencode   claude-opus-4.7    8,348    8,292     2026-04-24T00:00:00Z
0.6454   opencode   claude-opus-4.7    42,248   27,267    2026-04-23T18:30:00Z
0.6191   opencode   claude-opus-4.7    48,384   29,956    2026-04-23T13:00:00Z
0.5728   opencode   claude-opus-4.7    23,226   13,303    2026-04-23T21:30:00Z
0.5569   opencode   claude-opus-4.7    26,263   14,626    2026-04-24T01:00:00Z
0.5533   opencode   claude-opus-4.7    31,642   17,508    2026-04-23T16:30:00Z
0.4747   opencode   claude-opus-4.7    45,383   21,542    2026-04-23T17:30:00Z
0.4492   opencode   claude-opus-4.7    38,057   17,094    2026-04-23T19:30:00Z
0.4492   opencode   claude-opus-4.7   640,292  287,588    2026-04-23T14:30:00Z
0.4348   opencode   claude-opus-4.7    27,750   12,067    2026-04-23T20:00:00Z
```

Every single one is `opencode / claude-opus-4.7`, every single one is in a 12-hour window on 2026-04-23, and the inputs are *small* by claude-code standards — tens of thousands of tokens, not millions. The 0.99 row in particular is striking: 8,348 in, 8,292 out. That is essentially a model writing a document of the same length as its prompt. Expansion-task signature.

The 640,292 / 287,588 row is the interesting one because it has both a large input *and* a 0.45 ratio. That session was probably "here is a large body of text, rewrite it" or "here is a long log, summarize at length" — the only pattern that justifies hundreds of thousands of output tokens.

## What the bimodality means in token mass

The interesting part is not just row counts; it's where the actual GPU-burn lives. Bucketing token totals by ratio:

```
ratio range        rows    input tokens       output tokens   row share
[0.000, 0.005)     455     1,407,692,155        3,613,542     36.9%
[0.005, 0.020)    395     1,799,142,901       14,679,623     32.0%
[0.020, 0.100)    162       133,217,285        6,529,269     13.1%
[0.100, 0.500)    215        94,747,741       15,641,084     17.4%
[0.500, 2.000)      6           180,111          110,952      0.5%
```

The two modes have radically different token-mass profiles. The lower mode (rows with ratio < 0.02, which is two-thirds of all rows) carries **3.21 billion input tokens and only 18.3 million output tokens** — an effective ratio of 0.0057 across the bucket. The upper mode (rows with ratio ≥ 0.1, ~17.9% of rows) carries only **94.9 million input tokens but 15.6 million output tokens** — an effective ratio of 0.165. So the lower mode contains 97% of all input-token traffic but only 35% of output-token traffic. The upper mode is barely a footnote in input volume, but it accounts for about 41% of all output token mass observed in this dataset.

Translated into work: the cost of running this stack is overwhelmingly an **input cost**, dominated by a small number of CLIs (`claude-code`, `openclaw`) that ship enormous contexts to get back tiny structured outputs. The expensive *output* token mass — the prose, the document writing — is a side activity that lives in `opencode` sessions on `claude-opus-4.7` only.

## How to use this in practice

Three concrete uses for the ratio:

**1. Anomaly detection via ratio drift.** A row from `claude-code` with ratio 0.15 is wildly off-distribution: p90 for `claude-code` is 0.0176. Either the harness mode changed (suddenly emitting prose instead of tool calls) or someone is using `claude-code` interactively to write a document in chat, which is not its dominant pattern. Either is interesting and probably worth surfacing.

**2. Source classification from a single row.** Given an unknown row, the ratio is a strong source prior. ratio < 0.005 → lower-mode CLI (overwhelmingly `claude-code`/`openclaw`/`codex`). ratio > 0.1 → upper-mode CLI (almost certainly `opencode`/`hermes`). The valley between 0.02 and 0.05 is where the false positives live, but it is also the rarest band in the data.

**3. Caching strategy follows from mode.** Lower-mode rows are sending the same massive context repeatedly with small deltas, which is exactly the workload prompt caching is designed for. (Indeed `claude-code` shows median cached/input of 0.883 — most input is cached.) Upper-mode rows are reading less and writing more; cache amortization is bounded above by input mass, so cache strategy matters far less for them. If you are deciding whether to invest in a cache layer, look at the ratio distribution first — if your traffic is upper-mode-heavy, the ROI is small.

## The 0.99 row

I keep coming back to it. 8,348 input tokens, 8,292 output tokens, on a single row, on `claude-opus-4.7`, in `opencode`, at midnight UTC on April 24. That is somebody handing the model a document and asking it to rewrite the document. The model wrote one for one. There is no cache value here, no harness loop here, no tool use. It is pure transformation work. Six rows in 1,560 cross the 0.5 threshold, and all six are `opencode / claude-opus-4.7` in the same 12-hour window. Pull on that thread and you can probably find the user task: a long-form rewrite session that hit Opus eight or nine times in succession. Looking at the timestamps clusters them: 13:00, 14:30, 16:30, 17:30, 18:30, 19:30, 20:00, 21:30, 00:00, 01:00 — a continuous editing run from afternoon into the small hours.

The lower mode tells you what it costs to *think with code*. The upper mode tells you what it costs to *write*. The valley between them is the boundary between agent and assistant.

## Caveats

Only rows with `input_tokens > 0` are in the ratio histogram (1,233 of 1,560). The 327 rows with zero input are a separate cohort already covered elsewhere; they would all have undefined or infinite ratio.

`cached_input_tokens` is not in the denominator. If it were, the `opencode` and `hermes` numbers would shift somewhat (their cached/input is high), but the directional gap to the lower-mode CLIs would persist because their cached fractions are even higher.

The IDE-side completion source is rendered here as `ide-assistant-A` per the redaction convention. Its sample size (n=6) is too small to anchor either mode, but the few rows that exist (mean ≈ 0.017) place it in the valley, consistent with an interactive tool-use harness with occasional inline-completion bursts.

The ratio is a *single-row* statistic; rolling it up to per-session would require the resampled-session schema, which is a separate discussion. But the per-row picture is already sharp enough to act on: two modes, 60× apart at the median, and the source label tells you which one a row belongs to with very high confidence.
