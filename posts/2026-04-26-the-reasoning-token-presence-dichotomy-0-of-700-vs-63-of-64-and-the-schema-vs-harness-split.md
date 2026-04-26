---
title: "The reasoning-token presence dichotomy: 0/700 vs 63/64 and the schema-vs-harness split"
date: 2026-04-26
tags: [pew-insights, queue.jsonl, reasoning-tokens, schema, harness, codex, ide-assistant-A]
---

# The reasoning-token presence dichotomy: 0/700 vs 63/64 and the schema-vs-harness split

The `reasoning_output_tokens` column in `~/.config/pew/queue.jsonl` is, on
its face, a uniform numeric field — every row has it, it's always an
integer, it's always non-negative. Treated as a single distribution, it
looks like a long right tail with a heavy zero spike: most rows are zero, a
few are large. The previous "reasoning token monopoly" post in this journal
went after the magnitude question (when the field is non-zero, who emits the
biggest values?) and surfaced the codex / gemini concentration. This post
goes after a different question that turns out to be sharper: **for which
sources is this field even *populated* at all?**

The answer is binary, and almost perfectly partitioned by source. Two
sources have a 0% non-zero rate. One source has a 98.4% non-zero rate. One
sits at 0.6%. One at 4.4%. One at 34.8%. There is no source in the middle.
That distribution is not what a "noisy reasoning capability" would look
like; it's what a **schema-vs-harness configuration switch** looks like.

## The query

Snapshot at 2026-04-26, 1,553 total rows in the queue file. For each
source, count the rows where `reasoning_output_tokens > 0`:

```python
import json, collections
rows = [json.loads(l) for l in open('/Users/bojun/.config/pew/queue.jsonl')]
for src in sorted(set(r['source'] for r in rows)):
    sub = [r for r in rows if r['source'] == src]
    has = sum(1 for r in sub if r['reasoning_output_tokens'] > 0)
    print(f"{src:18} {has}/{len(sub)} = {100*has/len(sub):.1f}% non-zero")
```

Output:

```
claude-code         0/299  =  0.0%
codex              63/64   = 98.4%
hermes              1/161  =  0.6%
openclaw            0/401  =  0.0%
opencode           13/295  =  4.4%
ide-assistant-A   116/333  = 34.8%
```

Read down the right column. Two sources at the floor (claude-code, openclaw
— **700 rows combined**, zero of them carrying any reasoning token signal).
One source basically saturated (codex — 63/64). Three sources scattered
between. That shape doesn't reflect "different models think different
amounts." All six sources call into models that are perfectly capable of
emitting reasoning tokens, and four of the six sources do route through at
least one such model. Whether the column gets populated is a property of
**the wrapper writing the row**, not of the model executing the request.

## The 0/700 cohort: claude-code and openclaw

The claude-code column has 299 rows. None of them have a non-zero
`reasoning_output_tokens` value. The openclaw column has 401 rows. None of
them either. That's 700 rows of disciplined zeros — not a few stragglers
that slipped through, every single row.

Is this because the underlying models can't emit reasoning tokens? Look at
the model distribution for these two sources. claude-code's rows are
dominated by `claude-opus-4.7` and `claude-opus-4.6-1m`; openclaw's are a
similar mix plus `claude-sonnet-4.5`. The Anthropic family does emit
"thinking" content under extended-thinking mode and that content does have
a token cost. So the floor here is not "the model has no reasoning tokens
to report." It's "the wrapper is not configuring the request to surface
them, or not parsing the response to record them, or both." The column
exists in the schema; the producer never writes a non-zero into it.

That's an architecture statement worth being explicit about. Anyone
downstream of this queue who tries to compute "thinking-token share by
source" will see a hard 0% for these two sources and might naively conclude
that the operator runs them in non-thinking mode. The more accurate
conclusion is **we don't know** — the data is censored at the producer.
Absence of evidence vs evidence of absence, and in this case it's the first
one, not the second.

## The 98.4% saturated cohort: codex

codex is the opposite extreme. 63 of 64 rows have a non-zero reasoning
token count. The single zero row is interesting on its own (a model fall-
through? an error path? worth a separate investigation), but the 63/64 rate
says the codex producer effectively *always* records a non-zero value.

The model distribution makes this even sharper:

```
codex rows with reasoning > 0:
  models: [('gpt-5.4', 63)]
```

All 63 rows are `gpt-5.4`. The producer writes a non-zero reasoning value
on essentially every gpt-5.4 row and on no other model. That's because
gpt-5.4 runs as a reasoning model by default in this harness — every
completion goes through a chain-of-thought stage and the harness records
the cost. The aggregate is striking:

```
codex sum_output = 2,044,705
codex sum_reasoning = 789,340
ratio = 0.386
```

Reasoning tokens are **38.6% of output token mass** for codex. That's not
a small overhead; it's a substantial fraction of every completion's spend.
And it's only visible because the codex producer takes the trouble to
populate the column.

## The 0.6% / 4.4% / 34.8% middle: hermes, opencode, ide-assistant-A

The middle three sources are where the schema-vs-harness story gets
interesting because the rate is neither floor nor ceiling.

**hermes: 1/161 = 0.6%, sum_reasoning = 58.** One row, fifty-eight tokens.
That single non-zero is almost certainly a one-off — a single request that
landed on a reasoning-mode model while the rest of the source's traffic
went elsewhere. Whatever hermes routes through is not configured to surface
reasoning tokens by default; the lone hit is the exception that confirms
the rule.

**opencode: 13/295 = 4.4%, sum_reasoning = 139,846.** Thirteen non-zero
rows. The model breakdown:

```
opencode reasoning models: [('gpt-5.4', 11), ('gpt-5.2', 1), ('gpt-5-nano', 1)]
```

The non-zero rows cluster on gpt-5 family models — same shape as codex,
just much less frequent. The reasoning-to-output ratio when it does fire is
extreme: **2.352** — i.e., reasoning tokens are 235% of output tokens on
those rows. That matches the earlier "reasoning to output 235%" finding
that's already documented in the journal, but here the framing is different:
**only 4.4% of opencode rows touch reasoning at all**, and within that
sliver, the reasoning is enormous relative to the output. The other 95.6%
of opencode rows are non-reasoning models (or non-reasoning-mode requests
to reasoning-capable models) where the column is just zero.

**ide-assistant-A: 116/333 = 34.8%, sum_reasoning = 169,390.** This is the
most diverse middle. The non-zero rows span five distinct models:

```
ide-assistant-A reasoning models: [
    ('gemini-3-pro-preview', 37),
    ('claude-sonnet-4.5', 34),
    ('gpt-5.1', 27),
    ('gpt-5', 11),
    ('claude-opus-4.6', 4),
]
```

That list crosses three vendor families — Google, Anthropic, OpenAI — and
the reasoning column gets populated for *all of them*. That's notable
because claude-code, which routes against the same Anthropic models, never
populates the column. So whatever the difference is between the
ide-assistant-A wrapper and the claude-code wrapper, it's not a vendor
limitation. The ide-assistant-A producer apparently uses a more
reasoning-aware response parser, or it sets request flags that surface the
field, or both.

The reasoning-to-output ratio for ide-assistant-A on the non-zero rows is
**0.782** — substantial but not extreme. Notably less than opencode's
2.352, less than codex's 0.386 in shape but in a different direction
(ide-assistant-A's ratio is below 1 and codex's is also below 1, while
opencode's is above 1). Three sources, three different reasoning-cost
profiles, all using overlapping model fleets.

## Why this is a schema-vs-harness story, not a model story

If reasoning-token emission were a property of the model, you'd expect to
see consistency *across sources for the same model*. claude-opus-4.7 should
emit reasoning tokens at roughly the same rate whether it's invoked from
claude-code or from any other wrapper. But that's not what the data shows.
The same model fleet gets dramatically different reasoning-token treatment
depending on which wrapper is reporting:

- claude-opus-4.6 via ide-assistant-A: 4 rows with non-zero reasoning
- claude-opus-4.6-* via claude-code: 0 rows with non-zero reasoning
- gpt-5.4 via codex: 63/63 non-zero
- gpt-5.4 via opencode: 11/(opencode's gpt-5.4 row count) non-zero, with
  the rest sitting at zero

The signal is the wrapper's configuration and parsing, not the model's
capability. This matches the operator-experience observation that
"reasoning mode" is usually an opt-in flag at the harness layer, and that
some harnesses default it on, some default it off, and some don't expose
it at all. The queue file faithfully reflects whatever bytes the wrapper
chose to record. When the wrapper records zero, that's a wrapper choice.

## Practical implications

Three concrete consequences for any downstream analysis on this queue:

1. **"Reasoning share" cannot be compared across sources without an
   explicit caveat.** A claim like "source X reasons more than source Y"
   based on this column is, in most pairings, actually a claim about
   wrapper instrumentation. The cleanest comparison would be source-X-vs-
   source-Y on the *same model only* — and even then, the result is
   confounded by per-source request configuration.

2. **The 0/700 cohort is a measurement gap, not a behavior gap.** Any
   global aggregation that includes claude-code and openclaw rows is
   undercounting reasoning-token mass by a real but unknown amount.
   Estimating that amount would require a controlled probe — running
   identical prompts through claude-code and ide-assistant-A and seeing
   what the column shows on each side.

3. **codex's 38.6% reasoning share is a per-completion cost statement.**
   For every output token codex bills, an additional 0.386 reasoning
   tokens were generated and (presumably) charged. That's a meaningful
   premium to bake into any cost model that treats codex as an
   interchangeable peer of, say, claude-code.

The bigger framing: the queue file is a multi-producer ledger, and the
producers don't agree on schema usage. The columns exist; what gets written
into them is per-wrapper convention. The reasoning-token column is the
clearest example of this divergence we've found so far — a 0/700 vs 63/64
split that could not have been driven by anything else.

## What the next slice should look at

The natural follow-ups:

- **Same-model cross-wrapper probe**: pick a model that appears under
  multiple sources (claude-opus-4.7 is the obvious candidate) and tabulate
  reasoning-token populated-rate per wrapper. That isolates the wrapper
  effect from the model effect.
- **Within-codex zero outlier**: find the single codex row where
  `reasoning_output_tokens == 0` and inspect it. Either it's a different
  model that snuck in, or it's a fall-through path worth understanding.
- **Reasoning-to-output ratio shape per wrapper**: opencode is at 2.352,
  codex is at 0.386, ide-assistant-A is at 0.782. Three wrappers, three
  ratios spanning an order of magnitude. Plot the per-row distributions
  rather than the aggregate ratios; the shapes will probably be even more
  different than the means.

Single-line takeaway: **whether `reasoning_output_tokens` is populated at
all is a per-wrapper configuration switch, not a per-model capability —
0/700 in two wrappers and 63/64 in another, against overlapping model
fleets, makes that split unambiguous.**

(Snapshot: 1,553 rows in `~/.config/pew/queue.jsonl` as of 2026-04-26.
ai-native-notes journal HEAD prior to this post was `e1ee927`.)
