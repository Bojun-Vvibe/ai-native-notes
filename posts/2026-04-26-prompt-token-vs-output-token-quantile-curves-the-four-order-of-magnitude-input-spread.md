# Prompt-token vs output-token quantile curves: the four-order-of-magnitude input spread, the 100x output range, and the per-source curve divergence

If you slice the queue ledger purely by token volume — ignoring source, ignoring time, ignoring model — the global story is already striking. Across **1,523 half-hour buckets** of token activity captured in the local queue ledger as of 2026-04-26, the input and output token quantiles look like this:

```
INPUT   n=1523  mean=2,240,548   min=0  max=55,738,577
        p10=0      p25=29,067   p50=504,072   p75=1,982,159
        p90=5,224,501   p95=11,837,368   p99=26,895,846

OUTPUT  n=1523  mean=26,369   min=0  max=416,890
        p10=445    p25=1,436    p50=5,616    p75=22,309
        p90=79,051   p95=131,309   p99=253,374
```

Two observations jump out before any decomposition. First, the input distribution spans more than four orders of magnitude between p10 (which is literally 0) and p99 (26.9 million). Second, the input mean of 2.24 million is roughly four and a half times the input median of 504 thousand — a heavy right-skew signature. The output distribution is a touch better behaved (mean 26,369 vs. median 5,616 is "only" a 4.7× ratio), but it too spans almost three orders of magnitude between p25 and p99.

The natural follow-up question is whether input and output move together. If a bucket has 5× the median input, does it have 5× the median output? Or are the curves decoupled? The answer turns out to be: they are decoupled in source-specific ways, and the global ratio of output-to-input tokens is a poor summary of any individual source's behavior.

This post walks the input-token quantile curve and the output-token quantile curve, source by source, and shows that the curves are not just shifted; they have different shapes. One source has a literally degenerate input curve. One has an output curve a hundred times higher than the global. One has an output-to-input ratio that bumps up to 99.3 percent on a single bucket.

## The global quantile curves

The global curves climb at very different rates. Here are the input and output values at matching percentiles, with the multiplier from the previous percentile shown alongside:

```
percentile   INPUT (×prev)        OUTPUT (×prev)
p10                  0  (—)                445  (—)
p25             29,067  (∞)              1,436  (3.23×)
p50            504,072  (17.3×)          5,616  (3.91×)
p75          1,982,159  (3.93×)         22,309  (3.97×)
p90          5,224,501  (2.64×)         79,051  (3.54×)
p95         11,837,368  (2.27×)        131,309  (1.66×)
p99         26,895,846  (2.27×)        253,374  (1.93×)
max         55,738,577  (2.07×)        416,890  (1.65×)
```

A few things stand out:

- **The input curve has a discontinuity at p10–p25.** From 0 to 29,067 is mathematically an infinite multiplier; in plain English, at least 10 percent of all rows have literally zero input tokens, while the 25th percentile already has nearly 30k. There is a population of rows that simply never carries input.
- **The input curve climbs much faster than the output curve through the lower half.** From p25 to p50, input multiplies by 17.3× while output multiplies by 3.9×. From p50 to p75, input multiplies by 3.9× and output by 4.0×. So in the lower half the two curves decouple sharply; in the middle they happen to track; in the upper tail they decouple again, with input multiplying by 2.3× from p75 to p90 while output multiplies by 3.5×.
- **The maxima are not at a fixed ratio.** The global max input (55.7M) divided by the global max output (416,890) gives a ratio of 134-to-1. The medians divided give 504,072 / 5,616 = 89.8-to-1. The means give 2,240,548 / 26,369 = 85.0-to-1. So at the high end, the system gets less efficient at producing output per input. (Or, equivalently, the largest-input buckets are not the largest-output buckets.)

A single global ratio of "output is roughly 1 percent of input" hides the entire structure. To see what is actually generating these curves, decompose by source.

## The input quantile curves, by source

```
source            n     p50          p90          p99            max         mean
claude-code      299   2,110,759   20,066,952   36,693,559   55,738,577    6,135,832
codex             64   3,151,085   15,925,961   28,982,569   29,634,948    6,418,456
openclaw         388   1,569,966    4,780,952   14,281,181   23,487,070    2,374,931
opencode         282     385,628    1,401,547    4,883,366   11,956,114      672,744
hermes           157     101,355      994,701    1,904,784    3,124,801      351,540
ide-assistant-A  333           0            0       50,729      172,138        1,745
```

Read this as six different curves. The curves do not just shift; they cross each other, they have different gradients, and one of them is degenerate.

The most striking thing is the bottom row. The IDE-side editor assistant (333 rows) has a median input of zero, a p90 of zero, and a max of 172,138. Its mean is 1,745. Out of 333 rows, **327 have an input value of exactly zero** — 98.2 percent. The remaining 6 rows account for the entire input mass attributed to this source, and even those are small relative to the other sources.

This explains where the global "p10 = 0" comes from. The IDE-side assistant alone supplies all 327 zero-input rows in the dataset (327 of 1,523 = 21.5 percent), single-handedly putting at least the bottom 21st percentile of the global input curve on the floor. Without this source, the global input p10 jumps from 0 to roughly 60,000.

Above that floor, the next ranking is hermes, the workflow proxy. Hermes shows real input tokens — median 101k — but it is an order of magnitude below the other heavyweights. Hermes is essentially a small-batch source.

opencode is the next layer up, with a median input around 386k. Then openclaw at 1.57M. Then claude-code and codex at the top, with medians of 2.11M and 3.15M respectively.

But notice what happens at p99 and at max. claude-code's p99 of 36.7M is more than double codex's p99 of 29.0M, and claude-code's max of 55.7M is essentially double codex's max of 29.6M. So even though codex has the higher *median* input (3.15M vs 2.11M), claude-code has the much heavier tail. The two curves cross between p50 and p90.

That crossover is meaningful. codex generates buckets with consistently large but bounded input — it does not appear to ever exceed about 30M input tokens in a half-hour bucket. claude-code generates buckets with somewhat lower typical input but a long tail upward, with the top ten input buckets in the entire dataset all belonging to claude-code on the claude-opus-4.7 model, ranging from 30.3M up to 55.7M, all between 2026-04-19 and 2026-04-21. The single highest input bucket — 55,738,577 input tokens, 416,890 output tokens — was at 2026-04-21T01:00:00Z on claude-opus-4.7. That is not a typical workload; it is a heavy day, and it lives entirely inside one source.

## The output quantile curves, by source

The output curves do not match the input curves' ranking:

```
source            n      p10       p50        p90       p99        max         mean
opencode         282    4,382    38,335    155,931    322,558    333,890     66,252
claude-code      299      416    10,968    144,604    289,012    416,890     40,565
codex             64      993    17,290     77,981    148,783    163,782     31,954
openclaw         388      445     3,645     48,764     66,910    116,291     12,414
hermes           157      840     6,295     20,401     29,022     38,184      8,608
ide-assistant-A  333      125     1,674      8,200     23,898     37,381      3,409
```

opencode has the highest median output by a wide margin: 38,335 tokens, more than 3× claude-code's median of 10,968 and 11× hermes's median of 6,295. It also has the highest p90 (155,931) and a max only 14 percent below the global max. Yet opencode's *input* p50 is only 386k, the third-lowest of the meaningful sources.

That is the key observation: **the source ranked third in input volume is ranked first in output volume.** opencode produces a lot of output for relatively modest input.

The flip side is openclaw, which has the third-largest input median (1.57M) but the second-lowest output median (3,645). openclaw is the inverse of opencode: lots of input flowing through, very little output coming out.

These contrasts are far easier to see in the explicit ratio of output-to-input tokens:

```
source           output/input ratio (mean over rows with input>0)
opencode         0.131
hermes           0.081
ide-assistant-A  0.017  (n=6 only, the rest have input=0)
claude-code      0.010
codex            0.006
openclaw         0.005
```

opencode emits, on average, 13.1 cents of output for every dollar of input. claude-code emits about 1 cent. openclaw emits a half cent. The factor between opencode and openclaw is 26×.

A single global "output is 1.18 percent of input" number (which is what you get if you divide global mean output 26,369 by global mean input 2,240,548) is again a fiction of the source mix. Each source is operating at a different point on the input–output efficiency curve, and the ratios span more than an order of magnitude.

## The 99.33 percent ratio outlier

Inside opencode, even the per-row ratios are wide. The maximum recorded output-to-input ratio in the entire dataset is **0.9933** — one bucket emitted 99.33 percent as much output as it consumed in input. Filtering the queue ledger for opencode rows whose output-to-input ratio exceeds 0.5 yields the following examples:

```
input    output   timestamp                  model
48,384   29,956   2026-04-23T13:00:00.000Z   claude-opus-4.7
31,642   17,508   2026-04-23T16:30:00.000Z   claude-opus-4.7
42,248   27,267   2026-04-23T18:30:00.000Z   claude-opus-4.7
23,226   13,303   2026-04-23T21:30:00.000Z   claude-opus-4.7
 8,348    8,292   2026-04-24T00:00:00.000Z   claude-opus-4.7
```

Across the run from 2026-04-23T13:00 to 2026-04-24T00:00 — about eleven hours — opencode hit ratios of 0.62, 0.55, 0.65, 0.57, and 0.99. That is fundamentally different behavior from claude-code, whose maximum ratio across all 299 of its rows is 0.1133 (just over 11 percent), or from openclaw, whose maximum ratio is 0.0343 (3.4 percent).

What does it mean for a half-hour bucket to emit nearly as much output as it takes input? It means the assistant is composing long-form output relative to short prompts. It is a pattern of "small instruction, large response," sustained across the entire bucket. The fact that it appears specifically in opencode and specifically on claude-opus-4.7 says that there is a substrate-and-model combination where this efficiency curve climbs an order of magnitude above what other substrates produce.

By contrast the hermes top-ratio rows top out at 0.42 (input 30,632, output 12,835, 2026-04-17T06:30 on opus) and 0.36 (input 54,794, output 19,483, 2026-04-24T00:30 on claude-opus-4.7). hermes can hit moderate ratios but it does not approach unity.

## Curve crossings: where the rankings flip

Listing the per-source ranks at each percentile shows just how mobile the ordering is:

```
                   p50 input rank     p50 output rank
codex              1                  3
claude-code        2                  4
openclaw           3                  5
opencode           4                  1
hermes             5                  6
ide-assistant-A    6                  6 (tie at the floor)
```

Four of the six sources move at least two ranks between the input and output medians. opencode jumps from 4th to 1st. claude-code falls from 2nd to 4th. openclaw falls from 3rd to 5th. Codex stays high on both. Hermes stays low on both.

If you did the same exercise at p99 you'd see another rearrangement. claude-code's p99 output (289,012) is below opencode's p99 output (322,558), even though claude-code's p99 input (36.7M) is far above opencode's p99 input (4.9M). Yet claude-code holds the global *max* output (416,890), beating opencode's max output (333,890). The tail behavior is yet another reordering: at the very tip of the right tail, claude-code's heavy input does eventually translate into the largest single output bucket, but only there.

## Why this matters for any aggregate metric

Three concrete consequences:

1. **Global output-to-input ratio is meaningless.** Treating "output as a percentage of input" as a system-wide efficiency number averages 13 percent (opencode) with 0.5 percent (openclaw) and gets you a number that describes neither source. Always condition on source.

2. **Input p10 is a count of editor-assistant rows.** As long as the IDE-side assistant logs zero-input rows, the bottom decile of input is a property of that source's logging, not a property of token consumption. Strip those rows out and the input curve becomes monotonically informative.

3. **The 26× ratio span is the real story.** The fact that opencode runs 26× more output-token-efficient than openclaw on the same dataset is the strongest single fact in the input-versus-output picture. A workflow that needs lots of output per input dollar should run on the opencode substrate. A workflow that needs lots of context-conditioned analysis without much output (refactor diffs, code review with terse comments) is a natural fit for openclaw or claude-code.

## Closing observation

The global input and output token quantile curves are useful as cover sheets — they tell you that input spans four orders of magnitude and output spans three, that the medians are 504k and 5.6k respectively, and that the max-input bucket (55.7M tokens, 2026-04-21T01:00) and max-output bucket (416,890 tokens, also claude-code on claude-opus-4.7) both come from the heavy claude-code source. But once you decompose by source, the curves split into six different stories: a degenerate floor (the IDE assistant), a small-batch worker (hermes), a high-leverage compressor (opencode at 13 percent output ratio), a heavy-context analyzer (claude-code), a steady mid-volume runner (codex), and a high-input low-output substrate (openclaw). These six processes do not share an efficiency curve. They live on different curves, and the global view is the volumetric average of those curves, not an underlying physical constant.
