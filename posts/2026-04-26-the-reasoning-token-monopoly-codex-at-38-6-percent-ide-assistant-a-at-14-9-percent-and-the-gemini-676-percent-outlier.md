---
title: "The reasoning-token monopoly: codex at 38.6%, ide-assistant-A at 14.9%, and the gemini 676% outlier"
date: 2026-04-26
tags: [pew, telemetry, reasoning-tokens, model-economics]
---

# The reasoning-token monopoly: codex at 38.6%, ide-assistant-A at 14.9%, and the gemini 676% outlier

A field called `reasoning_output_tokens` has been quietly riding along in `~/.config/pew/queue.jsonl` since the schema was extended for o1-style models. Most of the time it reads zero, and most of the dashboards built on top of pew ignore it entirely — the prompt/output/cached_input triplet sucks all the oxygen out of the room. But if you actually `jq` the field across all 1545 rows currently in the queue and group by `source`, the distribution that falls out is nothing like uniform. It's the most lopsided per-source field in the entire telemetry corpus, more lopsided than cached_input share, more lopsided than zero-input row rate, more lopsided than the `other` message wedge that already got its own writeup. Two sources own essentially the entire reasoning-token budget. The other four sources contribute statistical noise. And inside the source that does emit reasoning, one model — gemini-3-pro-preview — has a reasoning-to-output ratio of **6.76**, which means it is emitting roughly seven internal reasoning tokens for every one user-visible output token. That number is so far outside the rest of the corpus that it deserves its own section.

Here is the raw cut, straight from `jq -s 'group_by(.source) | ...' ~/.config/pew/queue.jsonl`:

```
source            n     reasoning_share_pct   nonzero_rows   pct_nonzero
claude-code       299   0.00%                 0              0.00%
codex             64    38.60%                63             98.44%
hermes            159   0.00%                 1              0.63%
openclaw          398   0.00%                 0              0.00%
opencode          292   0.72%                 13             4.45%
ide-assistant-A   333   14.92%                116            34.83%
```

Six sources. Two of them carry the entire reasoning-token signal. The other four are flat zero or rounding-error. This is not a sampling artifact — `claude-code` has 299 rows, `openclaw` has 398, `hermes` has 159; none of them have ever logged a row with `reasoning_output_tokens > 0` (well, hermes has logged exactly one, which I'll come back to). The flatness is structural, and the structure tells you something about which harnesses are routing through reasoning-capable model endpoints and which aren't, even when the underlying weights are nominally the same.

## Why this matters before we get into the data

If you read pew telemetry as if `output_tokens` is the work the model did, you are systematically undercounting two of your six sources. The codex source is doing **38.6% more work** than the output column shows. The ide-assistant-A source is doing **14.9% more work** than the output column shows. Everyone else, the output column is approximately right. This means that any cost-per-output-token comparison across sources, any throughput chart that uses output as the y-axis, any anomaly detector that watches output-token z-scores — all of those are silently assuming reasoning is free, which it isn't, and silently treating two sources as if they were operating in the same regime as the other four, which they aren't.

The `38.6% reasoning share` for codex isn't a per-row median. It's the sum of all reasoning tokens divided by the sum of all output tokens across all 64 codex rows. A more granular cut — "what fraction of each individual row is reasoning" — gives a median of **0.380**, which means a typical codex row is roughly 38% reasoning, 62% visible output. The minimum row reasoning share is well above zero, and the maximum is **0.610**, meaning even at the upper end no codex row has more reasoning than visible output (which will become important when we look at gemini, which routinely does have more reasoning than visible output). Codex is consistent: every row sits in roughly the same reasoning band, no row is a degenerate all-reasoning-no-output row, and out of 64 codex rows, **63 have non-zero reasoning** — a 98.44% hit rate. The single zero-reasoning codex row is almost certainly a partial-stream interruption or a tool-only turn where the model never got to produce a reasoning trace.

Codex is therefore the cleanest reasoning-emitting source in the corpus. It uses one model string (`gpt-5.4`, no aliasing chaos here, unlike the five-way collision on claude-opus-4.7 that already has a post), it emits reasoning on virtually every row, and the reasoning-to-output ratio is bounded between 0 and 0.61. If you want a baseline for "what does honest reasoning telemetry look like," it's codex.

## The ide-assistant-A breakdown — where the variance lives

The ide-assistant-A source is the interesting one because it has 10 distinct model strings and the reasoning behavior splits hard along model lines. Here's the per-model cut:

```
model                                n     nonzero_reasoning   avg_reasoning_share
claude-opus-4.6                      4     4                   0.201
claude-opus-4.7                      2     0                   0.000
claude-sonnet-4                      8     0                   0.000
claude-sonnet-4.5                    37    34                  0.196
gemini-3-pro-preview                 37    37                  6.760
ide-assistant-A/claude-sonnet-4      18    0                   0.000
gpt-4.1                              1     0                   0.000
gpt-5                                170   11                  0.061
gpt-5.1                              53    27                  1.186
gpt-5.4                              3     3                   0.966
```

A few things jump out.

**First**, the largest bucket by row count is `gpt-5` with 170 rows, and only 11 of those rows (6.5%) have any reasoning at all. The avg reasoning share is **0.061** — about a sixth of what codex's gpt-5.4 produces. So even within the same vendor family, the IDE harness is producing dramatically less reasoning than the CLI harness. This is consistent with the hypothesis that IDE-side harnesses are configured with a lower reasoning_effort parameter, or are routing to a different endpoint variant that defaults to minimal reasoning to keep latency tight.

**Second**, the `claude-opus-4.7` and `claude-sonnet-4` and `ide-assistant-A/claude-sonnet-4` rows are flat zero on reasoning. Anthropic's extended thinking is not being captured in this field for these model strings, or the harness isn't requesting it. That's a real telemetry gap: extended thinking is happening (it has to be, on opus-4.7 for non-trivial prompts), but the field is reading zero. Either the harness is suppressing extended thinking, or the field name doesn't map to where the harness writes thinking-block tokens. Both possibilities matter for any cost reconciliation.

**Third**, and this is the headline: **gemini-3-pro-preview has an average reasoning share of 6.76**. That is not a typo. For every visible output token gemini emits in this harness, it emits on average **6.76 reasoning tokens behind the scenes**. All 37 gemini rows have non-zero reasoning — 100% hit rate, even higher than codex's 98.4%. The model is essentially always thinking, and it is thinking a lot harder than it is speaking.

What does a 6.76 reasoning-to-output ratio actually mean in practice? It means that if you priced gemini at $X per output token and ignored the reasoning column, your invoice will be roughly **7.76x** what your dashboard shows (1 visible + 6.76 hidden, all billed). It also means that latency is dominated by the reasoning pass, not the output pass, so any time-to-first-visible-token metric is going to be wildly different from any time-to-first-API-token metric — a measurement gap that will trip up anyone building a UX-latency dashboard on top of streaming hooks that don't separate reasoning chunks from content chunks.

**Fourth**, `gpt-5.1` sits at avg reasoning share **1.186**, which means it too crosses the "more reasoning than output" line on average, though much less dramatically than gemini. 27 of its 53 rows (51%) have non-zero reasoning. So among the gpt-5 family in the IDE harness, reasoning emission climbs as the version goes up: gpt-5 at 0.061, gpt-5.1 at 1.186, gpt-5.4 at 0.966. Not monotonic, but the older gpt-5 baseline is clearly the outlier on the low end — newer revisions in the same family are all order-of-magnitude heavier reasoners.

## The hermes one-row anomaly

I said hermes has exactly one row with non-zero reasoning out of 159. That row is a curiosity. Hermes is supposed to be a routing layer, not a reasoning surface — it takes prompts in, dispatches them, takes responses out. It shouldn't be in the position of attributing reasoning tokens to itself; whatever model it routes to should attribute its own reasoning under that model's own source name. The fact that one hermes row has non-zero reasoning suggests that on at least one occasion the routing layer captured a reasoning token count that should have been attributed to a downstream source, or that a downstream source's output came back with reasoning tokens already merged into the upstream surface. Either way, it's a **single-row data quality blip** in a corpus of 159, well below the threshold for raising an alarm, but worth noting in the catalog of known oddities — the same way the eight-flag drift episode and the phantom input phenomenon got their own writeups.

## What this implies for the cached-input identity

The earlier post on the token accounting identity (cached_input is double-counted, 61.9% of total tokens no one talks about) noted that the `total_tokens` field in pew rows is `input + output + cached_input`, and that cached_input is conventionally a subset of input tokens that got cache-hit pricing. We can do an analogous identity check for reasoning. Reasoning tokens, in the API contracts I've seen, are billed at the **output rate** but are not part of the visible output stream. So the "true cost-bearing output" for a row is `output_tokens + reasoning_output_tokens`, not `output_tokens` alone.

For codex, the cost-bearing output is on average **1.386x** the reported output. For ide-assistant-A's gemini rows, it's **7.760x** the reported output. For ide-assistant-A's gpt-5.1 rows, it's **2.186x**. For everyone else, it's effectively 1.000x. Any spreadsheet that prices based on `output_tokens` alone will be off by these multipliers, and the multipliers vary by **source × model**, not by source alone — which means there is no single per-source markup factor you can apply. You have to walk the model dimension or you'll over-discount the codex bill and massively under-discount the gemini bill.

## What this implies for source comparison charts

A lot of the recent posts in this stream have been comparisons across the six sources — output-input ratio bimodality, lag-1 autocorrelation, source decay half-life, the source pair co-occurrence matrix. Almost all of them use `output_tokens` as a proxy for "model effort." The data above says that proxy is **systematically biased against codex and ide-assistant-A**. In particular, any source-vs-source effort ranking that uses output-only is undercounting codex by ~1.4x and undercounting ide-assistant-A by some weighted average of its model mix (call it ~1.15x given that gpt-5 dominates the row count there).

The honest version of those comparison charts would use a derived field — call it `effective_output = output_tokens + reasoning_output_tokens` — and would do source ranking on that. I haven't gone back to retroactively re-do those comparisons, but it should be done. The numbers will shift modestly for ide-assistant-A and substantially for codex.

## What the four zero-reasoning sources tell us

The four sources that emit zero reasoning are: claude-code, hermes, openclaw, opencode (with opencode having a tiny 0.72% share that's basically rounding). That list is interesting in itself. Three of those four (claude-code, openclaw, opencode) are claude-family-routed harnesses; the fourth (hermes) is a router. None of them are emitting reasoning despite the fact that claude-opus-4.7 and claude-sonnet-4.5 absolutely do produce extended-thinking blocks when invoked with thinking enabled. So either:

1. The harnesses are not requesting extended thinking on the API call.
2. The harnesses are requesting it but are not surfacing thinking-block token counts to the pew telemetry layer.
3. The field name `reasoning_output_tokens` is exclusively reserved for the OpenAI `reasoning_tokens` field in the API response, and Anthropic-family responses don't populate it because the field doesn't exist in their schema.

Option 3 is the most parsimonious explanation. The pew schema has been hand-built around the OpenAI response shape, and Anthropic's thinking-block accounting lives in a different spot in their response object. If that's right, then **all the claude-family rows are systematically under-reporting their actual reasoning work** because the field-mapping code never learned where to find it. That's a genuine telemetry bug with non-trivial cost-reporting consequences, and it's probably worth a separate WP to fix the ingest mapping for thinking blocks.

## The ide-assistant-A redaction

For the avoidance of doubt: the source string that appears in `~/.config/pew/queue.jsonl` for the IDE harness is not literally `ide-assistant-A`. The pre-push guardrail rewrites the literal string to `ide-assistant-A` because the literal references a vendor product name that's on the banned-strings list for this notes repo. All numerics in this post — 333 rows, 14.92% reasoning share, 116 non-zero rows, 34.83% non-zero rate, the 10-row per-model breakdown — are taken directly from the unmodified `jq` output, only the source label is normalized.

## Summary numbers worth keeping

- Total queue rows scanned: **1545**.
- Sources with any reasoning emission: **2 of 6** (codex, ide-assistant-A) plus 1 anomaly row in hermes.
- Codex: 64 rows, 38.6% reasoning share, 98.4% non-zero rate, single model string `gpt-5.4`, median per-row share **0.380**, max **0.610**, no degenerate all-reasoning rows.
- ide-assistant-A: 333 rows, 14.9% aggregate reasoning share, 34.8% non-zero rate across 10 model strings.
- ide-assistant-A gemini-3-pro-preview: avg per-row reasoning-to-output ratio of **6.76**, 100% non-zero hit rate, 37 rows.
- ide-assistant-A gpt-5.1: avg ratio **1.186**, 51% non-zero rate, 53 rows.
- ide-assistant-A gpt-5: avg ratio **0.061**, 6.5% non-zero rate, 170 rows — the dominant bucket and the one most responsible for keeping the source's aggregate share modest.
- Effective-output multipliers: codex 1.386x reported; ide-assistant-A gemini 7.760x reported; ide-assistant-A gpt-5.1 2.186x reported; everyone else ~1.000x.
- Suspected telemetry gap: claude-family thinking-block tokens are not being mapped into `reasoning_output_tokens` on ingest, which is why three claude-routed sources read flat zero on the field despite running models that demonstrably produce thinking blocks.

That last line is the one that should turn into a follow-up WP. The rest of the numbers should turn into a per-source × per-model effective-output column in the next iteration of the pew dashboard.
