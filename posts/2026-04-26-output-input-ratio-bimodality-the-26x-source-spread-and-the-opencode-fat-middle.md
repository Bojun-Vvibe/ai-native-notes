# Output:input ratio bimodality — the 26× source spread and the opencode fat middle

The output-to-input ratio is one of the most under-read numbers in any AI usage telemetry. People look at total tokens, they look at cost, they look at cache hit rate, and then they stop. But the ratio of how many tokens come out of a model versus how many tokens were fed in tells you something the other metrics can't: it tells you what *shape* of work that model is being asked to do. A ratio under 0.01 means the model is being treated as a discriminator — it reads a lot, says little, makes a decision or a small edit. A ratio above 0.5 means the model is being treated as a generator — it reads a small prompt and produces a large piece of text. A ratio in the middle, say 0.05 to 0.5, means the model is doing what most people *think* AI assistants do all day: read some context, reason a bit, write a moderately sized response.

The interesting question isn't what the average ratio looks like. The interesting question is whether that ratio is bimodal — whether each source pins itself to one mode or the other, or whether it spreads itself across the spectrum. If the ratio is bimodal per source, that's evidence that each source has a *narrow* job. If the ratio is unimodal and spread across all bins, that's evidence the source is being used as a general-purpose tool. Both are valid; they just have very different ergonomic implications, and they predict very different optimisation strategies.

## The query

Captured at `2026-04-26T03:39:33Z` against `~/.config/pew/queue.jsonl` (per-bucket token rollups across all sources, 1188 valid rows after dropping zero-input rows).

```
jq -r 'select(.input_tokens>0) | "\(.source)\t\(.output_tokens / .input_tokens)"' \
  ~/.config/pew/queue.jsonl
```

I then bucketed the per-source output by ratio band and counted how many rows fell in each.

## What came back

```
claude-code     n=299  mean=0.0096  min=0.0000  max=0.1133  lo<.01=235  mid=64   hi>.5=0
openclaw        n=384  mean=0.0047  min=0.0002  max=0.0343  lo<.01=326  mid=58   hi>.5=0
opencode        n=279  mean=0.1294  min=0.0006  max=0.9933  lo<.01=12   mid=261  hi>.5=6
editor-asst     n=6    mean=0.0168  min=0.0066  max=0.0372  lo<.01=1    mid=5    hi>.5=0
codex           n=64   mean=0.0056  min=0.0003  max=0.0291  lo<.01=60   mid=4    hi>.5=0
hermes          n=156  mean=0.0796  min=0.0006  max=0.4190  lo<.01=25   mid=131  hi>.5=0
```

Mean ratio across sources spans from `0.0047` (openclaw) to `0.1294` (opencode). That's a **27.5× spread** in mean output:input ratio across sources that all nominally do "the same thing" — they take a human prompt, they read some files, they write a response. By any naive read of what an AI coding assistant does, you'd expect these numbers to be within a factor of two or three of each other. They are not. They differ by an order of magnitude and a half.

The aggregate histogram across all 1188 rows looks like this:

```
<0.001       131  (11.0%)
0.001-0.01   528  (44.4%)
0.01-0.05    261  (22.0%)
0.05-0.1      70  ( 5.9%)
0.1-0.5      192  (16.2%)
0.5-1          6  ( 0.5%)
```

That's not bimodal in aggregate. That's a long left tail with a secondary bump at `0.1-0.5`. **55.4% of all hour-buckets have a ratio under 0.01** — meaning more than half the time, across all sources combined, every hundred input tokens produce fewer than one output token. That's an extremely lopsided shape. It tells you that the *dominant mode* of how these tools are being used is read-heavy: cache the world, reason over it, emit very little.

But the per-source breakdown reveals the real structure. Three of the six sources — `openclaw`, `codex`, `claude-code` — have **>78% of their rows in the `<0.01` band**. They are pure read-heavy tools. `openclaw` is the most extreme: 326 of 384 rows (85%) sit in `<0.01`, and the maximum ratio it ever achieves is `0.0343`. The model never, in 384 hour-buckets, produces output that is even 4% of its input. That's a tool whose entire purpose, at the token level, is to take huge contexts and produce tiny verdicts.

`opencode` is the structural opposite. Only 12 of 279 rows (4.3%) sit in `<0.01`. The vast majority — **261 of 279, or 93.5% — sit in the middle band** (`0.01` to `0.5`). Six rows (2.2%) are above `0.5`, including a maximum of `0.9933`, which is a row where the model produced almost as many output tokens as it consumed in input. That's a *generation-dominant* signature. Opencode is being used as a writer, not a reader.

`hermes` is interesting because it splits the difference. 25 of 156 rows (16%) are in the read-heavy band; 131 of 156 (84%) are in the middle; the maximum is `0.4190`. Hermes never crosses 0.5, but it also rarely drops below 0.01. It's a *moderate* tool — it reads, it reasons, it writes, but in proportions that don't pin to either extreme. If you had to guess which source was being used for the broadest variety of tasks, hermes would be a defensible answer just from this number alone.

The "editor-assistant" source has only 6 rows, which is too few to draw bimodality conclusions from. But it's worth noting that 5 of those 6 rows (83%) fall in the middle band, with a max of `0.0372`. Even with a tiny sample, the source pins to a narrow, low-but-not-zero ratio band.

`codex` has 64 rows and looks like a slightly less extreme version of `openclaw`: 60 of 64 (94%) are read-heavy, max ratio `0.0291`.

## The bimodality reading

If you stare at those numbers long enough, a clear pattern emerges: bimodality exists, but it's **across sources, not within them**. Each source is broadly unimodal in its own signature — `openclaw` always sits in `<0.01`, `opencode` always sits in the middle, hermes always sits in the moderate band. But the population of sources, taken together, splits cleanly into two camps:

1. **Read-dominant sources**: claude-code, openclaw, codex, editor-assistant. Mean ratios under 0.02. Output is always under 5% of input.
2. **Generate-or-mix sources**: opencode, hermes. Mean ratios above 0.07. Output regularly hits 10-50% of input, occasionally crosses 50%.

There is no source in the middle. There is no source whose mean ratio is, say, 0.03 or 0.04. The closest is the editor-assistant at 0.0168, but even that is closer to the read-dominant camp than to the middle.

This is the bimodality. It's a property of the *fleet*, not of any individual tool. The fleet bifurcates into "tools that compress" and "tools that expand." Understanding which camp a tool sits in tells you almost everything about its cost shape, its caching behaviour, and the kind of work it's most useful for.

## Why this matters for cost

Output tokens cost roughly 4-5× more than input tokens on most current API price sheets, and cached input tokens cost about a tenth of fresh input. So the dollar impact of the ratio is not linear in the ratio itself. A read-dominant source like openclaw, which runs at 0.005 ratio, spends most of its dollars on input — but if that input is heavily cached (and prior posts in this corpus have shown it usually is), the dollar bill stays small. A generation-dominant source like opencode, running at 0.13 ratio, has the *opposite* dollar shape: a much larger fraction of total spend is on output tokens, which can't be cached.

This means the same total token volume produces wildly different bills depending on which side of the bimodality your tool sits on. A million tokens through openclaw at 0.005 ratio is roughly 995k input + 5k output. A million tokens through opencode at 0.13 ratio is roughly 885k input + 115k output. At canonical pricing (input $3/Mtok, output $15/Mtok, cache discount 90% on input), with 80% cache hit on input:

- openclaw: 995k × (0.2 × $3 + 0.8 × $0.30) / 1M + 5k × $15 / 1M = $0.477 + $0.075 = **$0.55**
- opencode: 885k × (0.2 × $3 + 0.8 × $0.30) / 1M + 115k × $15 / 1M = $0.425 + $1.725 = **$2.15**

Same million tokens, **3.9× the cost**, just from the ratio difference. And opencode does in fact sustain ~80% cache hit on input from prior measurements, so this isn't a worst-case fabrication — it's roughly what the bill actually does.

## Why this matters for caching

A read-dominant source benefits massively from prompt caching. Its entire game is "load the same context many times, ask different questions of it." If the cache works, the per-call cost collapses to near zero on the input side. If the cache doesn't work — say, because the source jitters its prompt prefix, or because it doesn't pin a system message — the per-call cost balloons and the source becomes the most expensive thing in the fleet.

A generation-dominant source benefits much less from caching. Its dominant cost is output, and you cannot cache output. The lever for opencode-style sources is not caching; it's *brevity*. If you can teach a generation-dominant source to write tighter responses without sacrificing quality, you save real money. If you teach a read-dominant source the same thing, you save nothing because the output bill was negligible to begin with.

This is why "we should make the model less verbose" is great advice for some sources and irrelevant for others. The bimodality tells you which is which. Apply the wrong optimisation to the wrong source and you waste effort.

## The opencode fat middle

The most distinctive shape in the breakdown is opencode's middle-band concentration. **93.5% of opencode rows sit in the `0.01-0.5` band.** That is a remarkably narrow distribution for what is nominally a general-purpose coding agent. It does not have a long left tail of "read-heavy" buckets, which would be expected if the agent were doing a lot of pure exploration. It does not have a meaningful right tail of "generation-dominant" buckets either, except for the six outliers above 0.5. It just sits in the middle.

The interpretation: **opencode is structurally a balanced tool**. It reads, it writes, in proportions that don't drift much from hour to hour. The 0.13 mean and the 93.5% middle-band concentration together imply a tool that has settled into a stable behavioural regime. It doesn't oscillate between modes the way you might naively expect a coding agent to. It picks a ratio and holds it.

Compare this to `claude-code`, which has a wider relative range despite a lower mean: its rows span from 0.0000 to 0.1133, and 22% of rows are in the middle band rather than the low band. Claude-code has more behavioural variance per bucket. Some hours it reads heavily and writes nothing. Other hours it writes more. Opencode just doesn't do that.

This stability is itself diagnostic. It suggests the opencode workflow has more uniform sessions: the agent reads roughly the same amount of context every time, and produces roughly the same amount of output every time, regardless of what the user is asking it to do. That could be because the agent has a strong default scaffolding (reads X files, writes Y lines of summary) that dominates the variance from user prompts. Or it could be because opencode users tend to use it for a narrower range of tasks. Either way, the per-bucket signature is unusually flat.

## The six high-ratio outliers

Six rows — all from opencode — sit above 0.5 ratio, with the highest at 0.9933. These are buckets where the model produced nearly as many output tokens as it consumed input tokens. In a coding context that's almost always one of two things: either a long-form generation task (writing a document, drafting a spec, generating a large boilerplate file) or a session where the model was asked to elaborate or critique without much new input context.

Six rows out of 279 is 2.2%. That's small enough to be edge cases, large enough to be a real mode rather than a single anomaly. It suggests opencode has a low-frequency but real "generation mode" alongside its dominant balanced mode. The fleet's total exposure to genuinely generation-dominant work is concentrated in this 2.2% of opencode buckets and nowhere else. No other source ever crosses 0.5 in this dataset.

## The codex contrast

`codex` is interesting precisely because it's small and read-dominant. 64 rows, 94% in the `<0.01` band, max ratio 0.029. It looks like a miniature openclaw in shape — same compress-not-expand signature — but at a much smaller volume. The mean ratio of 0.0056 is actually lower than openclaw's 0.0047 by an irrelevant margin; functionally the two are indistinguishable in shape, just different in volume.

That this shape is shared between two ostensibly different tools is a reminder that **the ratio signature is more about how the tool is being used than about what tool it is.** Codex and openclaw are doing the same kind of work at the token level: read a lot, decide a little, emit almost nothing. If the user changed their workflow — started asking codex to write long responses — the signature would change. The ratio is a behavioural fingerprint of the human-tool pairing, not a fixed property of the tool.

## What you can predict from the ratio alone

Given just the per-source mean output:input ratio, you can predict, with reasonable confidence:

- **Cost shape**: read-dominant sources are dominated by input cost (and therefore by cache effectiveness). Generation-dominant sources are dominated by output cost.
- **Optimisation lever**: read-dominant sources benefit from cache hygiene; generation-dominant sources benefit from output-length discipline.
- **Latency profile**: output tokens dominate per-call latency in most current systems. So generation-dominant sources will feel slower per call even at lower total volume.
- **Failure mode**: a read-dominant source whose ratio drifts upward is usually a sign of context degradation — the agent stopped trusting its retrievals and started writing more from scratch. A generation-dominant source whose ratio drifts downward is usually a sign the user has started giving it longer prompts (often a sign of escalating task complexity).

## What the data does not tell you

The per-bucket ratio aggregates across many calls within an hour, so it is a coarse proxy for the per-call ratio. A bucket with ratio 0.05 could be one call at 0.05, ten calls all at 0.05, or one call at 0.5 and nine calls at 0.0. The histogram cannot distinguish these. To get true per-call bimodality you need finer-grained logs.

The data also does not tell you *what task* each bucket was doing. Two buckets at the same ratio could be wildly different work: one a code review, one a refactor, one a documentation pass. The ratio constrains the *shape* of the work but does not identify it.

And finally: ratio alone does not tell you whether the work was *good*. A bucket with ratio 0.0001 (read a lot, write almost nothing) might be a great review or a complete punt. A bucket with ratio 0.5 might be a brilliant essay or a wall of plausible nonsense. Quality is orthogonal to shape.

## The takeaway

The fleet's output:input ratio is not a single number. It is a bimodal property of the *source set*, with a 27.5× spread between the most read-heavy source (openclaw, 0.0047 mean) and the most generation-heavy source (opencode, 0.1294 mean). 55.4% of all hour-buckets in the corpus sit in the lowest two ratio bands (`<0.01`), which means the fleet's dominant character is read-heavy compression, not generation. But that dominance is driven entirely by the read-dominant camp; the generation-dominant camp (opencode, hermes) operates in a completely different regime, with 93.5% of opencode buckets concentrated in the moderate middle band and a measurable but small generation-dominant tail.

The actionable read is: stop applying uniform optimisation logic across all sources. Cache-tuning matters for openclaw, claude-code, codex; output-length discipline matters for opencode and hermes. Treat them as different tools because, at the ratio level, they *are* different tools — even when the underlying model and the surface workflow look the same.

The bimodality is real, it's stable across 1188 buckets, and it's the single best one-number summary of what each source is actually doing all day.
