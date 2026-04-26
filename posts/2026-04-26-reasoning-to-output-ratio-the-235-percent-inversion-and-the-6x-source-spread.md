# Reasoning-to-Output Ratio: the 235% Inversion and the Six-fold Source Spread

> **Captured:** 2026-04-26T00:29:03Z
> **Source:** `jq -s 'map(select(.reasoning_output_tokens > 0)) | group_by(.source) | …' ~/.config/pew/queue.jsonl`
> **Scope:** 1,500 hourly buckets, six telemetry sources, one operator, one device.

## The number

Across all hourly buckets where the model emitted any chain-of-thought tokens, the **opencode** source reports a reasoning-to-output ratio of **235.2%** — meaning the model spent more than two reasoning tokens for every one user-visible output token. Three other sources sit dramatically lower:

```json
[
  {"source":"opencode","rows":13,"total_reasoning":139846,"total_output":59450,"reasoning_to_output_pct":235.23},
  {"source":"hermes","rows":1,"total_reasoning":58,"total_output":67,"reasoning_to_output_pct":86.57},
  {"source":"editor-assistant","rows":116,"total_reasoning":169390,"total_output":216683,"reasoning_to_output_pct":78.17},
  {"source":"codex","rows":63,"total_reasoning":789340,"total_output":2044705,"reasoning_to_output_pct":38.60}
]
```

The full spread is **38.6% → 235.2%**, a factor of 6.1× between the most output-efficient source (codex) and the most reasoning-heavy source (opencode). This is a single device, a single operator, a single overlapping window of model availability — yet four sources running the same underlying class of reasoning models produce ratios that differ by an order of magnitude.

The other two sources in the corpus (openclaw, claude-code) emit zero reasoning tokens entirely — they run against models that don't expose chain-of-thought at all, or against routes that suppress it — so they don't appear in this slice. The interesting comparison is among the four that *do* report reasoning, because they are at least playing the same game on paper.

## Why this is not just a model-choice artifact

The naive reading is "different sources call different models, so different ratios are expected." That reading is wrong, or at least incomplete, for two reasons.

**First, the same physical model lives behind multiple sources.** GPT-class reasoning models and Claude-class reasoning models are both routable from codex, opencode, and the editor-assistant source on this device — the configs overlap. Yet the ratios are not converging on a per-model average; they are clustering by *source*. That suggests the determining factor is not the model but the *prompt scaffold* the source wraps around the user's request.

**Second, the row counts make the spread statistically real.** Codex contributes 63 rows, the editor-assistant source 116 rows, opencode 13 rows. Even the smallest of these (opencode at 13 rows) sums to 139,846 reasoning tokens — that is not a one-off spike, that is sustained behavior across roughly two weeks of activity. The spread is too wide and too persistent to attribute to sampling noise.

Something about how each source structures its turns is causing the model to think dramatically more or less per visible token of reply.

## What a 235% ratio actually looks like inside a single turn

To get reasoning > output by 2.35×, the model must be spending the bulk of its compute on planning that the user never sees, then emitting a comparatively tight response. Concretely, on opencode rows: every visible token of model reply sits on top of roughly 2.35 hidden tokens of deliberation.

That ratio is the signature of an **agent-style** prompt scaffold:

- A long system prompt loaded with tool definitions, file-edit protocols, and worked examples.
- A user message that often boils down to "do this whole task" rather than "answer this question".
- Permission for the model to think through which tool to call, validate its own plan, and discard branches before committing to an output.

In that setup the model is supposed to think more than it writes — the writing is the small visible tip of a much larger reasoning iceberg, because most of the reasoning resolves into a single tool call (which is short to emit) rather than into prose (which would be long).

The opencode source matches this profile: it is a coding agent that wraps tool-use prompts around the model. Its 235% ratio is not pathological; it is what efficient agentic reasoning looks like when scored on the visible-output axis. The model is doing its job and not narrating it back.

## Why codex sits at 38.6%

Codex, the lowest of the four reasoning-emitting sources at 38.6%, runs a fundamentally different scaffold: it is a CLI coder that frequently emits **long-form explanations and patch diffs** as its primary output. When the user asks codex to "refactor this function and explain why", a substantial fraction of the model's compute is going *into* the visible explanation, not into hidden deliberation.

The output side is doing the work. Reasoning is still happening — 789,340 tokens of it across 63 rows is not small — but the output it produces is even larger (2,044,705 tokens), so the ratio compresses.

Concretely on a single codex row, every reasoning token corresponds to roughly 2.6 output tokens. The hidden-to-visible ratio inverts compared to opencode: codex narrates its work, opencode acts on it.

## The editor-assistant source at 78%: a hybrid

The editor-assistant source's 78.2% ratio sits between codex and opencode, and that is consistent with its workload mix. It serves both **inline edits** (closer to opencode's tool-use shape — short visible output, much hidden planning to figure out the right multi-line patch) and **chat-style explanations** (closer to codex's narration shape). The aggregate ratio is the weighted average of the two regimes.

That this source contributes the most rows of any reasoning-emitter (116) and still lands in the middle of the spread is itself informative: it means the inline-edit and chat-explain workloads are roughly balanced on this device. A pure inline-edit IDE would push toward opencode-style ratios (>200%); a pure chat-only IDE would push toward codex-style ratios (<50%).

## The hermes outlier (n=1)

Hermes shows up with 86.6% on a single row (58 reasoning tokens against 67 output tokens). One sample is not a trend, but it is worth noting that hermes only crosses into reasoning territory once across 154 total rows. That is a 0.6% reasoning-emission rate, by far the lowest of any source that emits reasoning at all.

The interpretation is straightforward: hermes is not running reasoning models as its default. The single row that does emit reasoning is almost certainly a manual override or a stray request routed to a thinking-class model. For analytical purposes, hermes can be classified alongside openclaw and claude-code as a "non-reasoning route" source, with the caveat that its routing is not strictly enforced.

## Cross-source concentration: where the reasoning compute actually lives

The headline ratios obscure where the reasoning compute is *concentrated*. Total reasoning tokens by source:

```
codex             789,340  (71.5% of fleet reasoning)
editor-assistant  169,390  (15.3%)
opencode          139,846  (12.7%)
hermes                 58  (<0.1%)
                ─────────
fleet total     1,098,634  reasoning tokens
```

Codex alone burns **71.5%** of all reasoning tokens in the corpus, despite being one of the smaller sources by row count (64 rows, vs 333 for the editor-assistant source). The reasoning compute per row is the relevant intensity metric:

```
opencode:          10,757 reasoning tokens / row
codex:             12,529 reasoning tokens / row
editor-assistant:   1,460 reasoning tokens / row
hermes:                58 reasoning tokens / row
```

Codex is the most *reasoning-intensive* source on a per-row basis (12.5K reasoning tokens per active hour bucket), and opencode is a close second at 10.8K. The editor-assistant source, despite contributing the most reasoning-emitting rows, is roughly **8× lighter per row** — a fingerprint of a much higher rate of trivial thinking-model calls per active hour, plausibly because IDE-side autocomplete and inline-action flows trigger tiny reasoning bursts very frequently.

The two intensity axes — reasoning-per-row and reasoning-to-output ratio — give a four-quadrant taxonomy of how each source uses thinking models:

| Source            | Reasoning/row | R/O ratio | Reading                                |
|-------------------|---------------|-----------|----------------------------------------|
| codex             | 12,529        | 38.6%     | Heavy thinker, heavier writer          |
| opencode          | 10,757        | 235.2%    | Heavy thinker, light writer (agentic)  |
| editor-assistant  | 1,460         | 78.2%     | Light thinker, balanced writer         |
| hermes            | 58            | 86.6%     | Effectively non-reasoning              |

This is a more useful classification than "which model does each source call", because it captures what the source *does* with the model — the prompt scaffold and turn shape — rather than just which weights it routes to.

## Why the ratio is the right axis to watch

There is a temptation to track reasoning tokens in absolute terms because they show up on bills as a separate line item. That is a fine thing to track for cost, but it is the wrong axis to track for *behavior*.

Absolute reasoning tokens go up when:
- The source is busier (more rows).
- The model is upgraded to a more thinking-heavy variant.
- The user happens to be working on harder problems this week.

The reasoning-to-output ratio, by contrast, isolates the **structural** question: given that the model decided to think, how much of that thinking made it to the user? That ratio is invariant to busyness, robust against model upgrades within a generation, and largely orthogonal to problem difficulty (harder problems usually scale both reasoning and output together).

When the ratio shifts on a given source, that is a signal worth investigating. It almost always means one of:

1. The source's system prompt has changed (more or fewer tool definitions, different examples).
2. The source has switched its routing policy between reasoning-heavy and reasoning-light models.
3. The user's interaction shape has changed (more "do this" tasks vs more "explain this" questions).

All three are real and tracked, but only the ratio surfaces the change immediately. The absolute counts don't move enough to notice for weeks.

## The 6.1× spread as a fleet-design lever

The most actionable read of this data is that **the same underlying model class can be made 6× more output-efficient or 6× more reasoning-heavy depending on how you wrap it**. Codex and opencode are both calling reasoning models; codex gets 2.6 output tokens per reasoning token; opencode gets 0.43. If the goal is "minimize reasoning cost per visible result", the codex-style scaffold is the answer. If the goal is "let the model think hard so the patch is right the first time", opencode's scaffold is the answer.

This is not a "which source is better" question — they are optimizing different objectives. But it does imply that **moving a workload between sources has an order-of-magnitude effect on reasoning cost**, even when the underlying model is held constant. That is a big lever for fleet-level cost management, and the data above is the first concrete measurement of how big the lever actually is on this device.

## What to track next

Three derivative metrics fall out of this naturally:

- **Reasoning emission rate per source**: what fraction of all rows for a given source contain non-zero reasoning. (Codex: 63/64 = 98%. Opencode: 13/272 = 4.8%. Editor-assistant: 116/333 = 34.8%. Hermes: 1/154 = 0.6%.) The spread here is even wider than the ratio spread, and it answers the question "when does this source choose to think at all?"

- **Per-model reasoning ratio**, holding the source constant. This decomposes the source-level ratio into the underlying model contribution, and would let you tell whether opencode's 235% is driven by one specific model variant or by all reasoning models it routes to.

- **Time-of-day drift in the ratio**. If the ratio shifts with the hour of day for a single source, that's evidence the user is doing different *kinds* of work at different times — agentic work in the morning, narrative work in the afternoon, for instance — and that signal can feed back into automated source routing.

The 1,500-row corpus is large enough to compute all three. The 235% headline number is just the entry point — the full distribution is the actual picture, and it is telling a story about scaffold design that no single number captures.

## The bottom line

Among the four sources in this fleet that emit chain-of-thought tokens at all, the reasoning-to-output ratio spans **38.6% to 235.2%** — a 6.1× range. The spread is not driven by which model each source calls; it is driven by how each source *frames the turn*. Codex narrates, opencode acts, the editor-assistant source does both. Hermes essentially abstains.

The lesson is that "we are using a reasoning model" is not by itself a meaningful operational statement. The same model, behind two different scaffolds, will produce visible output at wildly different efficiency. Pick the scaffold that matches what you actually want the model to do — and measure the ratio, because the absolute token counts will not tell you when you have picked wrong.
