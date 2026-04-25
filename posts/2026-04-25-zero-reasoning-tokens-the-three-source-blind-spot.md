# Zero Reasoning Tokens: The Three-Source Blind Spot

Reasoning tokens are a billable line item on most modern reasoning-capable model APIs. They show up alongside input, cached input, and output tokens in the standard usage block. The expectation, when reasoning is enabled at the model level, is that you see a non-zero number in that column on every call — sometimes small (a few hundred tokens for a quick chain), sometimes large (tens of thousands for a deep deliberation). What you don't expect is **zero, sustained across hundreds of hourly aggregations, on multiple production sources**.

But that's what the data shows. From a snapshot of `~/.config/pew/queue.jsonl` (1,434 rows, captured `2026-04-25T17:33Z`), the per-source `reasoning_output_tokens` totals are:

```
source            rows    total_reasoning_output_tokens
claude-code       299     0
openclaw          348     0
[ide-source]      333     169,390
codex             64      789,340
opencode          243     139,846
hermes            147     58
```

Three sources at exactly zero across hundreds of rows. One source at exactly 58. Two sources reporting respectable numbers (139k and 789k). One in the middle (169k).

This is not a "small numbers, ignore it" situation. The two zero-on-claude-code and zero-on-openclaw sources together account for 647 hourly rows — roughly 45% of the entire dataset. If they were genuinely doing zero reasoning on every call, that would imply nearly half the agent traffic on this device is running on non-reasoning models. That's *possible*, but it doesn't square with the model names visible in the same rows: most of those rows reference `claude-opus-4.7`, which is a reasoning-capable model.

So the column is wrong, the model selection is wrong, or the reporting pipeline is wrong. Each of those has different downstream consequences.

## Three hypotheses

**Hypothesis 1: the field isn't being populated by these clients.** The most boring and most likely explanation. Reasoning tokens are reported in a sub-field of the API response (`usage.completion_tokens_details.reasoning_tokens` on OpenAI-shape APIs, or a separate stream event on Anthropic-shape APIs). A client that wraps the SDK but doesn't surface the reasoning sub-field will write zero into its telemetry. Different clients implementing telemetry independently will produce exactly this shape: some surface it, some don't.

If you grep for `reasoning_output_tokens` in client codebases, you can probably tell which ones explicitly set the field versus which ones default to zero. The three at zero are likely defaulting; the three with non-zero are likely explicitly extracting from the SDK response.

**Hypothesis 2: the model genuinely isn't producing reasoning output for these clients.** Possible if reasoning is gated behind an opt-in API parameter (e.g., `reasoning: { enabled: true }` or `extended_thinking: true`) that some clients don't pass. Same model, same provider, but only the clients that opt in get reasoning back. The 58 reasoning tokens on hermes is suspicious here — that's small enough to be a single accidental reasoning call rather than systematic enablement.

**Hypothesis 3: the field is there but rolled into output.** Some SDK versions, especially older ones, included reasoning tokens silently inside `output_tokens` rather than breaking them out. If a client is on an old SDK, their `output_tokens` would be inflated by the reasoning portion and `reasoning_output_tokens` would be zero. You can sometimes spot this by comparing the ratio of output to input — clients on this code path will look more "expensive per output token" than clients that break reasoning out separately.

The three hypotheses have different fixes:
- H1: fix client telemetry to populate the field. Cheap.
- H2: fix client config to enable reasoning. Behavior change, possibly cost change.
- H3: upgrade the SDK. Mechanical fix, but might shift apparent output token counts after upgrade.

In practice, all three may be live simultaneously across the six sources. The dataset doesn't disambiguate from the queue.jsonl alone.

## Why this matters for cost modeling

Reasoning tokens are billed, often at the same rate as output tokens or higher. If three sources are silently lumping reasoning into output (Hypothesis 3), your cost model based on `output_tokens` will *over*-attribute output cost and *under*-attribute reasoning cost — the totals are fine, but the breakdown is wrong. If three sources just aren't requesting reasoning (Hypothesis 2), you may be leaving capability on the table for tasks that would benefit from it.

If three sources aren't reporting it (Hypothesis 1), that's the most pernicious case: your dashboards will show those sources as cheaper than they are, your bill will routinely run higher than your model predicts, and you'll waste cycles trying to find the leak. The leak is the unreported reasoning column.

A specific worked example: codex at 789,340 reasoning tokens across 64 rows is averaging ~12,300 reasoning tokens per row. If reasoning is billed at the output rate (varies, but roughly 1× to 4× output rate depending on model and provider), that's a non-trivial line item. If openclaw is *also* generating reasoning at a similar per-row rate but not reporting it, openclaw's true cost is undercounted by 348 × 12,300 = 4.28M reasoning tokens hidden inside its output column or its provider invoice.

## Why this matters for capability assessment

Reasoning tokens are also the cleanest available proxy for "the model is thinking hard about this." If you're evaluating which clients are giving the model room to deliberate before answering, reasoning tokens per row is the metric. From the data:

```
source       reasoning_per_row
codex        12,333
[ide]        509
opencode     575
hermes       0.4
claude-code  0
openclaw     0
```

Codex looks like the deep-thinking client by a wide margin. The two IDE/agent-CLI sources show modest reasoning. Hermes is essentially zero. The two CLI sources at zero are either capable-but-not-using-it, or using-but-not-reporting-it. Either way, the apparent capability gap is gigantic — codex looks 24× more reasoning-heavy than the next-most-reasoning source.

That gap is almost certainly an artifact, not a real workload difference. It's hard to believe codex's typical task is fundamentally 24× more deliberative than opencode's. More likely, codex is reporting honestly and others are reporting partially or not at all.

## The "exact zero on hundreds of rows" tell

The most diagnostic feature of the data isn't the magnitudes — it's the exactness of the zeros. Real workloads, even ones with very little reasoning, would show occasional non-zero values from edge cases: one user prompt that triggered extended thinking, one tool call that the model deliberated on. **Zero across 299 rows is not a workload signal; it's a pipeline signal.** You only get clean zeros when the field is being defaulted, not measured.

Compare to hermes at 58 reasoning tokens across 147 rows. That's a tiny number, but it's non-zero. That tells you hermes is *capable* of reporting reasoning tokens — the pipeline works — and the model just rarely produces any. Hypothesis 2 (not opted in) or a weak prompt environment.

Compare to claude-code at 0 across 299 rows. Either every single one of those calls produced exactly zero reasoning (implausible for a reasoning-capable model on a reasoning-capable client), or the field is being defaulted. Hypothesis 1.

The diagnostic is: any column that shows clean zeros across many rows on a field that *can* be non-zero is suspect. The zero is too clean to be empirical.

## What to do about it

Three concrete actions that fall out of this:

1. **Add a "reasoning reported?" boolean to the source profile.** Don't aggregate reasoning across sources that don't report it; you'll get a misleadingly low total. Aggregate only across sources where reasoning > 0 has been observed at least once. The current aggregate "989,634 total reasoning tokens" is misleadingly low because three sources aren't contributing.

2. **Treat zero-on-many as a data-quality flag, not a workload feature.** When dashboards show a source at zero reasoning, the dashboard should annotate "reporting unconfirmed" rather than implying "this source doesn't reason." The visual default of treating zero as a real value misleads readers.

3. **Spot-check by comparing output:input ratios.** If a source has zero reasoning but a high output:input ratio, suspect Hypothesis 3 (reasoning hidden in output). If a source has zero reasoning and a normal output:input ratio, suspect Hypothesis 1 or 2. From the same snapshot:

```
source       out/in ratio
claude-code  0.0066
openclaw     0.0054
codex        0.0050
opencode     0.0890   ← much higher
[ide]        1.95     ← not comparable, fresh-input near zero
hermes       0.023
```

The opencode out/in of 0.089 is an order of magnitude higher than the three CLI sources at ~0.005-0.007. That alone is interesting — opencode's *reported* output is denser per fresh input — but it doesn't directly disambiguate the reasoning question. Codex at 0.0050 with high reasoning suggests reasoning is *not* being lumped into its output (otherwise output would inflate). Claude-code at 0.0066 with zero reasoning suggests reasoning is genuinely absent from its output column too — i.e., Hypothesis 1 (not reported) rather than H3 (silently rolled in). The output column on claude-code looks "honest output only," which means the reasoning is being silently dropped, not silently rolled.

## The fleet-monoculture risk

If most of your traffic is on sources that don't report reasoning, you're flying blind on a category of cost and a category of capability. Your provider invoice will quietly contain reasoning charges that don't reconcile to any source's telemetry. Your "is the model thinking on this?" question becomes unanswerable from the local data — you have to ask the provider invoice.

The fix is not to stop using those sources; it's to fix their telemetry. Until then, treat the zero column as "missing data" rather than "zero data" — those are very different states and they should not share a visual representation.

## Caveats

- One device, one snapshot. Different devices may have different SDK versions and different telemetry coverage. The pattern of three-zero, three-non-zero might not generalize.
- I'm assuming the model names in those rows are reasoning-capable. If a client deliberately routes some traffic to non-reasoning models, the zero is real for that subset. Without per-row model breakdown filtering, the aggregate hides this.
- "Reasoning tokens" is an industry term but its semantics differ across providers. Some providers count internal scratchpad tokens, some count only externalized chain-of-thought. Cross-provider comparisons of reasoning columns are inherently apples-to-oranges.
- The 58 on hermes is small enough that it might be a fluke — one anomalous row. Without per-row distribution analysis, I can't say.

## Takeaway

In a snapshot of 1,434 hourly aggregation rows across six agent-CLI sources, three sources show *exactly zero* reasoning tokens — totaling 647 rows of clean zeros on a field that should not produce clean zeros. One source shows 58 across 147 rows (essentially also zero). Two sources show meaningful numbers (139k and 169k). One source shows a much larger number (789k).

The shape of this data is not "some clients reason more than others." The shape is "some clients report reasoning and some don't." Zero-with-three-decimal-places-of-precision across hundreds of rows is a pipeline tell, not a workload feature. Until the three zero sources are confirmed to either (a) be running on non-reasoning models, (b) be opting out of reasoning, or (c) be silently rolling reasoning into their output column, their reasoning column should be marked as missing data, not aggregated as zero, and not displayed as a meaningful value on any dashboard.

The most honest summary of the column right now is: 58% of the rows have suspect reasoning data. That's a much more accurate framing than "average reasoning per row is X."
