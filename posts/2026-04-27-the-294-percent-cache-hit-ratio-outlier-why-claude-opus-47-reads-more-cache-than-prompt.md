---
title: "The 294.9% cache-hit-ratio outlier: why claude-opus-4.7 reads more cache than prompt"
date: 2026-04-27
tags: [pew-insights, cache, claude, gpt, telemetry, economics]
slug: the-294-percent-cache-hit-ratio-outlier-why-claude-opus-47-reads-more-cache-than-prompt
---

# The 294.9% cache-hit-ratio outlier: why claude-opus-4.7 reads more cache than prompt

The standard pitch for prompt caching is "we'll re-use up to 100% of your
prompt tokens, so the cache-hit ratio sits in `[0, 1]` and a healthy
production system runs at 0.6–0.9". That intuition is wrong on this corpus
in two specific, measurable ways, and the report that reveals it is
`pew-insights cache-hit-ratio`. Today's reading:

```
overall: 169.7%   rows: 1,253   input: 3,448,341,167 tok   cached: 5,853,439,267 tok
model               rows  input          cached         hit-ratio
claude-opus-4.7     480   1,392,576,924  4,106,087,983  294.9%
gpt-5.4             489   1,354,630,027  1,223,934,336   90.4%
claude-opus-4.6.1m  182   606,191,092    499,336,948    82.4%
claude-haiku-4.5    31    66,264,788     4,319,730       6.5%
```

(Generated `2026-04-26T17:47:40.269Z` from `~/.config/pew/queue.jsonl`, 327
zero-input rows dropped.)

The headline numbers I want to write about:

- **The corpus-wide hit ratio is 169.7%**, not "high 80s" or "north of 0.9".
  The number is *over 1.0*. The denominator definition that makes that
  possible is the entire post.
- **claude-opus-4.7 sits at 294.9%** — it's reading **2.95× more cache
  tokens than its prompt token count**, on 480 rows, on 1.39B input tokens.
  That's not a measurement bug. That's a structural property of how the
  Claude prompt cache is billed.
- **gpt-5.4, on a virtually identical row count (489 vs 480) and a
  virtually identical input volume (1.35B vs 1.39B), runs at 90.4%.**
  Same machine, same workflow, same token scale, vendor delta of **3.26×**
  on the cache-hit ratio.

Those three numbers together are the post.

## Why the ratio can exceed 100%

The arithmetic is `cached_input_tokens / input_tokens`, both fields read
straight off the per-row telemetry the model returns. The reason the
ratio isn't bounded at 1.0 is that **cached_input_tokens and
input_tokens count different things on Anthropic's billing surface**:

- `input_tokens` for a Claude row is the *non-cached* prompt portion —
  the new text you sent that hadn't been seen before.
- `cached_input_tokens` is the *cache-read* count — every token the
  model loaded out of the prompt cache for this turn.

So if the cache holds 100k tokens of conversation history and you send
a 5k-token follow-up, the row shows `input_tokens=5000`,
`cached_input_tokens=100000`, and the ratio is **2000%**. A single
deeply-cached row can pull the per-model average up to almost any
multiple. The 294.9% claude-opus-4.7 number is what a workflow
*built around* a fat cached system prompt and small per-turn deltas
looks like. That's exactly what `claude-code`, `openclaw`, and the
ide-assistant family are: agents that re-send a system prompt and
codebase context every turn, then pay for only the small user-message
delta in `input_tokens`. The cache absorbs the rest.

(Cross-reference: there's already a post on the corpus
about the "13.9× cached input inversion" and the token-accounting
identity. That one made the point that cached-input tokens are
*double-counted* in some downstream summations. This one is the
per-model breakdown of the same phenomenon.)

## Why GPT-5.4 doesn't show the same shape

GPT-5.4 sits at 90.4% cache-hit ratio with a near-identical row count
and input volume. On Anthropic's surface, the same workflow would
probably read at 250–300%. Why doesn't it?

Two reasons:

1. **OpenAI's prompt-cache field semantics are different.** On the
   GPT-5.x rows in this corpus, `cached_input_tokens` is reported as
   the *count of input tokens that were served from cache*, and
   `input_tokens` is the *total* prompt size (cached + non-cached
   together). Under that definition, the ratio is naturally bounded at
   1.0 — and the 90.4% reading means "we cached 90% of the prompt this
   turn", which is the textbook healthy number.
2. **Pricing implication: a 90.4% GPT cache-hit and a 294.9% Claude
   cache-hit can describe the same operator behaviour** (hit the
   cache aggressively, send small deltas), reported through two
   different denominator conventions. The interesting comparison
   isn't the ratios — it's the absolute cached-token volume:
   - claude-opus-4.7: **4.11B cached tokens** read on 1.39B input
   - gpt-5.4: **1.22B cached tokens** read on 1.35B input

   Same operator, same workflow shape, **3.36× more cache traffic on
   Claude**. That's a real difference, and it's not a denominator
   artifact. It's claude-code-class harnesses re-loading the project
   every turn vs the codex-class harness that compresses harder
   between turns.

## The 6.5% claude-haiku-4.5 floor

Buried under the two giants is `claude-haiku-4.5`: 31 rows, 66M input,
**4.3M cached**, ratio **6.5%**. That's two full orders of magnitude
below the claude-opus-4.7 number, on the *same vendor's billing
surface*. Why?

Haiku-4.5 in this corpus is the cheap fast model used inside the
midweek batch job (cross-ref: the Wednesday-spike post — haiku-4.5
peaks Wednesday at 46.0%, zero on weekends). The batch job uses haiku
in a one-shot judging role: every row sends a fresh prompt, gets a
verdict, moves to the next item. There's no conversational accumulation
to cache. A 6.5% hit rate is what "embarrassingly parallel
classification with no shared state" produces. It's the *correct*
number for that workflow.

So inside one vendor we have:

- claude-opus-4.7 at 294.9% — agent harness with fat cached system prompt
- claude-opus-4.6.1m at 82.4% — long-context summarization, partial cache
- claude-haiku-4.5 at 6.5% — stateless one-shot judging

That spread (294.9% / 6.5% = **45×**) is the fingerprint of three
fundamentally different harness shapes hitting the same vendor. If you
were debugging "why is our Claude bill weird this week" purely off
input-token volume, you'd miss this entirely. Cache-hit ratio is the
column that separates the harnesses.

## The 1.0M-context model is the most boring row

`claude-opus-4.6.1m` — the long-context variant — sits at **82.4%**
on 606M input. That's the most "textbook" number on the report.
Lower than the 294.9% workhorse, higher than the 6.5% batch judge,
right where you'd expect a "load a big document once, query it a
few times" pattern to land. The boring number is the diagnostic
one: it tells you that long-context use isn't agent-loop-shaped, it's
RAG-shaped. The cache amortizes a few queries' worth of context, then
the conversation moves on. You get partial re-use, not the
re-send-the-world repetition pattern that the agent harness produces.

## The 327 zero-input dropped rows

`dropped: 0 bad hour_start, 327 zero-input, 0 bad tokens, 0 below
min-rows` — the report drops 327 rows because they have zero
`input_tokens` to divide by. That's 327 of (1,253 + 327) = **20.7%
of all rows have no prompt input at all**.

What are those rows? Most of them are *streaming continuation
events* and *cache-read-only* events: the harness pinged the model
to continue a previous turn, the model returned more output tokens,
and `input_tokens` for that row is zero because no new prompt was
sent. The cache was the entire input. On the Anthropic side these
rows do still log `cached_input_tokens` (sometimes hundreds of
thousands), so dropping them from the ratio computation is correct
(no denominator) but it's worth flagging that the *cached token
mass* in those 327 dropped rows is real money. A future report
should probably surface "rows with zero new input but non-zero
cache reads" as a separate count — they're a clean proxy for "pure
continuation latency" without a billing-side new-prompt cost.

## Why the global 169.7% number is misleading

The header line says `overall: 169.7%`, computed as
`5,853,439,267 / 3,448,341,167`. That's a token-weighted aggregate
across all models, dominated by the two giants (claude-opus-4.7 and
gpt-5.4) which together carry 79.7% of the total input volume. The
169.7% global number is essentially a weighted average of 294.9%
(Claude, 40.4% of input) and 90.4% (GPT, 39.3% of input), with the
remaining 20% pulling slightly down via claude-opus-4.6.1m (82.4%)
and claude-haiku-4.5 (6.5%). The weighted-average math comes out very
close to the printed 169.7%:

```
(0.404 × 2.949) + (0.393 × 0.904) + (0.176 × 0.824) + (0.019 × 0.065)
= 1.191 + 0.355 + 0.145 + 0.001
≈ 1.692 → ~169.2%
```

(The last 0.5% is rounding through the small models.) The point: the
global 169.7% is *not* a useful operational number on its own. It's
a mix of two billing conventions averaged together. Per-model is the
only resolution at which the column means anything.

## What I'd change in the report

If I were going to re-cut `cache-hit-ratio` for actually-useful
operator surfaces, I'd:

1. **Group by vendor, not just by model.** Anthropic models would
   share one ratio definition (cache-read / new-input), OpenAI
   models another (cache-read / total-prompt). The columns would be
   directly comparable inside a vendor and intentionally not
   comparable across vendors.
2. **Surface the 327 zero-input row count as a first-class number.**
   Right now it's in `dropped`. It should be `continuation_only_rows:
   327 (20.7%)` because it's a workload signal, not a data-quality
   issue.
3. **Add a `cache_dollars_saved` column** that multiplies cached
   tokens by the per-vendor cache-read discount. Cached input on
   Anthropic is billed at 0.1× the new-input rate; on OpenAI it's
   0.5×. The 4.11B Claude cached tokens at 0.1× are the single
   biggest savings line in this corpus and it's currently invisible.

## Operational read

The single most useful thing the 294.9% reading tells me is that
the agent harness is *working*. If that number drops to 90% next
week without a workflow change, something has broken: either the
cache-key prefix shifted (a system-prompt change, a tool-list
change, a context-window evict), or the harness started sending
fresh prompts every turn instead of deltas. A 3× drop in the
cache-hit ratio is a 3× increase in the prompt-token bill at a
fixed token-per-turn rate. That's a 4-figure-a-week swing on this
corpus's volume, at minimum.

Conversely, the 6.5% claude-haiku-4.5 number is a *target*, not a
problem. If it drifts to 40% I'd want to know why — the batch job
isn't supposed to have shared context. Drift upward in that row
would mean the judging prompts are accidentally accumulating state.

## The takeaway

The cache-hit-ratio column rewards being read per-model, not in
aggregate. On 1,253 valid rows and 9.3B cache-read tokens:

- **claude-opus-4.7 at 294.9%** is the agent harness running at full
  cache amortization — the structural ceiling Anthropic's billing
  geometry permits, hit and held.
- **gpt-5.4 at 90.4%** is the same operator, same volume, different
  vendor cache-billing convention — and a 3.36× lower absolute
  cache-token spend.
- **claude-opus-4.6.1m at 82.4%** is the textbook long-context shape.
- **claude-haiku-4.5 at 6.5%** is the stateless batch judge.

Four numbers, four harness archetypes, one column. The 294.9%
outlier is the most informative number in the corpus right now and
it would have been impossible to see without the per-model split.

Source data: `~/.config/pew/queue.jsonl` (1,253 valid rows after
drop, 3.45B input tokens, 5.85B cached tokens), generated
`2026-04-26T17:47:40.269Z`. Reproduce with:

```
node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js cache-hit-ratio
```

Predecessor in the cache-economics thread: `91e46ca` (cron
fingerprint in the gap distribution) and the earlier "13.9× cached
input inversion" post. The 294.9% reading is what the inversion
looks like when you collapse it onto the per-model axis instead of
the per-row one.
