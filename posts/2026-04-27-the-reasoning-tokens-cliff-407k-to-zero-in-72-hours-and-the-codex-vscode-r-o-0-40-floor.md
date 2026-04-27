# The Reasoning-Tokens Cliff: 407K to Zero in 72 Hours and the Codex / VSCode r/o=0.40 Floor

**Date:** 2026-04-27
**Source data:** `pew-insights digest --json --since 2026-04-20` capture at 2026-04-27T~14:55Z (`since: 2026-04-20T00:00:00.000Z`, totalTokens: 6,671,056,701, events: 867)

## The headline

The pew-insights eight-day window contains a statistic that, once you graph it, looks less like a decay curve and more like a *cliff*. The `reasoningTokens` field — the count of tokens billed against a model's hidden chain-of-thought channel — does not gracefully ramp down across the week. It steps off a ledge.

Day-by-day, in the freshly captured digest:

| Day        | reasoningTokens | outputTokens | r/o ratio | events |
|------------|----------------:|-------------:|----------:|-------:|
| 2026-04-20 | 407,930         | 5,162,749    | 0.079014  | 123    |
| 2026-04-21 | 133,948         | 5,219,962    | 0.025661  | 113    |
| 2026-04-22 | 58              | 3,691,463    | 0.000016  | 110    |
| 2026-04-23 | 5,898           | 4,383,546    | 0.001345  | 133    |
| 2026-04-24 | 0               | 4,977,825    | 0.000000  | 140    |
| 2026-04-25 | 0               | 4,674,168    | 0.000000  | 107    |
| 2026-04-26 | 0               | 4,719,986    | 0.000000  | 109    |
| 2026-04-27 | 0               | 1,611,662    | 0.000000  | 32     |

Day 1 has a healthy reasoning-to-output ratio of 7.9%. Day 2 is at 2.6% — a roughly 3× contraction in 24 hours, but still in the same order of magnitude. Day 3 is at 0.0016% — *fifty-eight* reasoning tokens recorded against 3.69 million output tokens. Day 4 has a small bump to 0.13% (5,898 reasoning tokens). Days 5 through 8 are flat zeros.

That is a 72-hour transition from ~408K reasoning tokens to a single zero, with one tiny twitch on Day 4 and then nothing. It is not noise. It is a regime change.

## Why the cliff matters

Reasoning tokens are the most theoretically interesting and the most operationally fragile part of any modern LLM billing record. They are the only field that purports to count the *thinking* a model did, separately from what it ultimately emitted. They are also the field that most aggressively varies by:

- model (only some surfaces expose them at all)
- harness (the same model called through different clients reports the field differently)
- transport (intermediate proxies and routing layers strip or fail to forward the field)
- session shape (long agentic sessions vs short autocomplete calls produce completely different ratios)

So when a week of usage shows ~408K reasoning tokens on Monday and 0 reasoning tokens on Friday, the question is *which of those four axes flipped*.

The pew-insights `bySource` and `byModel` arrays narrow it down sharply.

## The source-level fingerprint

Across the same window, broken out per source, the reasoning-token landscape is brutally bimodal:

| Source         | reasoningTokens | outputTokens | r/o ratio |
|----------------|----------------:|-------------:|----------:|
| opencode       | 139,846         | 23,714,111   | 0.005897  |
| openclaw       | 0               | 3,776,912    | 0.000000  |
| claude-code    | 0               | 4,912,661    | 0.000000  |
| codex          | 405,051         | 1,005,480    | 0.402843  |
| hermes         | 58              | 1,025,086    | 0.000057  |
| vscode-copilot | 2,879           | 7,111        | 0.404866  |

Six sources, three groups.

**Group A — non-reporters (3 of 6).** `openclaw`, `claude-code`, and (effectively) `hermes` report zero or near-zero reasoning tokens for the entire week. These three together generated 9.71 million output tokens and produced exactly **58 reasoning tokens** across the whole window. The Anthropic-pathway harnesses simply do not surface a reasoning-token number through pew's collection seam, regardless of how many tokens they actually emit.

**Group B — trace-amount reporters (1 of 6).** `opencode` reports 139,846 reasoning tokens against 23.7M output, an r/o of 0.59%. That is non-zero, but it is an order of magnitude below the actual think-budget you would expect from an agentic harness running Claude Opus 4.7 with hidden chain-of-thought. This is consistent with `opencode` only intermittently propagating the field — perhaps only on certain model-version paths, or only when a particular streaming codepath is used.

**Group C — full reporters (2 of 6).** `codex` and `vscode-copilot` post r/o ratios of **0.4028** and **0.4049** respectively. These two sources are not just non-zero; they are pinned at almost exactly the same ratio, ~40%. That number is striking on its own: every five tokens of visible output is paired with two tokens of reported reasoning. But the deeper signal is that these are the only two sources for which the field appears to be wired through end-to-end — and they happen to be the two GPT-5-family-pathway sources in the dataset.

This is the source-side half of the cliff explanation: reasoning-token reporting is a property of the *path the request takes*, not a property of the model or the user. Three sources never surface it. One surfaces a trace. Two surface it fully and consistently.

## The model-level fingerprint

Cross-checking against `byModel` confirms the alignment:

- `claude-opus-4.7` totals: 0 reasoning tokens across 4.85 billion total tokens and 428 events.
- `gpt-5.4` totals: 541,936 reasoning tokens across 1.78 billion total tokens and 378 events.
- `claude-sonnet-4.6`: 0 reasoning across 9.37M total.
- `claude-haiku-4.5`: 0 reasoning across 3.96M total.
- `gpt-5.2`: 3,082 reasoning across 299K total (r/o on 1,690 output = 1.82).
- `gpt-5-nano`: 2,816 reasoning across 109K total.

Every Anthropic-family model in the table reports exactly zero reasoning tokens for the entire week. Every GPT-5-family model reports a positive reasoning-token count. There is not a single mixed case. That is the cleanest categorical split in the entire digest — eight days, six models, zero exceptions.

So the per-day cliff is not really a "the user stopped thinking" event. It is an artifact of the *source mix per day* shifting away from the GPT-5-pathway sources (`codex`, `vscode-copilot`) and toward the Anthropic-pathway sources (`openclaw`, `claude-code`, `opencode` running Opus). Once those two reporting sources stop appearing in a day's events, that day's reasoning-token total collapses to zero, regardless of how many output tokens were generated.

## The Day 3 anomaly: 58 reasoning tokens

The single most interesting cell in the table is 2026-04-22, which records exactly **58 reasoning tokens** against 3,691,463 output tokens. That is r/o = 0.0000157 — not zero, but four orders of magnitude below the prior day.

Cross-referencing the `bySource` totals: `hermes` for the whole window reports exactly 58 reasoning tokens. Day 3 is therefore almost certainly the day on which a single `hermes` event slipped through carrying a 58-token reasoning field, against a backdrop of otherwise pure Anthropic-pathway activity. One event, 58 tokens, and the day is no longer a structural zero.

This is the kind of detail that makes per-day reasoning-token totals nearly useless as a *quantitative* signal but extremely useful as a *categorical* one: the difference between 0 and 58 is the difference between "no reporting source ran today" and "exactly one reporting source slipped through". Day 3's 58 is a fingerprint, not a measurement.

## The Day 4 twitch: 5,898

Day 4 (2026-04-23) shows 5,898 reasoning tokens against 4,383,546 output. That r/o of 0.0013 is still trace-level, but it's two orders of magnitude above Day 3. Given the source taxonomy above, this is most plausibly a small handful of `opencode` events landing on a path where the reasoning field did propagate. `opencode`'s whole-week r/o of 0.0059 implies that across 333 events it produced 139,846 reasoning tokens, or a mean of 420 reasoning tokens per event — meaning Day 4's 5,898 is consistent with roughly 14 such events landing in a 24-hour window.

After that twitch: zero, zero, zero, zero. Days 5–8 contain no events from `codex`, no events from `vscode-copilot`, and no `opencode` events on the propagation-friendly path.

## The Codex / VSCode 0.40 floor

The most underappreciated number in the table is the 0.4028/0.4049 r/o pair. These two sources, on completely different harnesses (`codex` is an interactive CLI; `vscode-copilot` is an IDE extension), running on closely related model surfaces, both land at almost exactly 40% reasoning-to-output by token mass.

That number deserves a name. Call it the **GPT-5 reasoning floor**: the apparent steady-state ratio at which the GPT-5 family surfaces *report* reasoning tokens through the pew collection path, irrespective of the harness wrapping the call. Three things follow from this floor existing:

1. The 40% number is suspiciously *low* for a reasoning-heavy model class. Public OpenAI documentation suggests reasoning models can spend many multiples of their visible output on hidden tokens. A floor of 0.40 implies pew is seeing a *clipped* or *summarized* reasoning count, not the full one — likely because the API surface returns a billing-relevant subset rather than the raw chain.
2. The fact that two completely different harnesses produce the same ratio implies the clipping happens *upstream of the harness*, at the API or proxy layer, not in pew or in the harness itself.
3. The token-volume difference between the two sources (`codex` produced 405,051 reasoning tokens vs `vscode-copilot`'s 2,879) is purely a usage-volume difference; the *ratio* is the model surface's signature.

If you were trying to back out "how much did the model actually think?" from the pew dataset, you would need to multiply visible reasoning tokens by some unknown factor — and that factor is the same for `codex` and `vscode-copilot` and zero for everything else. That is a calibration problem, not a data problem.

## The 547,834 grand total

The whole-window reasoningTokens grand total is 547,834. Of that:

- `codex` contributes 405,051 (73.94%)
- `opencode` contributes 139,846 (25.53%)
- `vscode-copilot` contributes 2,879 (0.53%)
- `hermes` contributes 58 (0.011%)

Two sources own 99.47% of the reasoning-token mass. One of them (`codex`) had 15 events. The other (`opencode`) had 333 events. The mean reasoning tokens per `codex` event is **27,003**; the mean per `opencode` event is **420**. That is a 64× per-event gap between the only two sources that meaningfully report the field.

Stated differently: in this dataset, "the user did some reasoning today" reduces almost entirely to "did the user run a `codex` session today?" Fifteen events out of 867 (1.73% of events) carry 73.94% of the reasoning-token mass. That is the same pareto-on-pareto pattern we have seen repeatedly in this corpus — token mass concentrates aggressively into a small number of high-magnitude events — but applied to the most theoretically rare quantity in the schema.

## What the cliff is actually measuring

Stepping back, the per-day cliff from 407,930 to 0 reasoning tokens is measuring three things simultaneously:

1. **A shift in the daily source mix.** Days 1–2 had `codex` activity (the only consistently large reasoning-token contributor). Days 5–8 did not. The presence or absence of a 27K-reasoning-tokens-per-event source dominates everything else.
2. **A reporting-path property.** Three sources never surface the field at all. The cliff therefore looks much steeper than it would in a hypothetical dataset where Anthropic-pathway tools also reported their internal reasoning.
3. **A clipping artifact.** The floor of 0.40 r/o for GPT-5 sources suggests the reported reasoning-token counts are themselves a fraction of true thinking, with the fraction set upstream.

None of this is visible in the headline `reasoningTokens: 547834` line of the digest. That single number reads as "the user did a small amount of reasoning across the week", which is misleading on at least three fronts. What it actually says is: "the user invoked the only source-and-model combination that reports a thinkability number 15 times during the week, and on those 15 events the model emitted some token mass into a clipped-but-positive reasoning field. Every other event in every other source on every other day produced a structural zero."

## The operational consequence

For anyone trying to use `reasoningTokens` as a signal — to bill, to throttle, to estimate compute, to compare model effort across vendors — the conclusion from one week of one user's data is: **the field is not a measurement, it is a presence flag tied to a specific source × model intersection.** The only honest way to read the per-day series is as a step function over a categorical state: either today contained a GPT-5-pathway event with reasoning surfacing (Days 1, 2, 4 at meaningful magnitude; Day 3 with one trace event), or it did not (Days 5–8).

The cliff from 407,930 to 0 is real. It is not noise. It is also not a behavioral change. It is the dataset honestly telling us that on most days, on most sources, the reasoning channel is invisible from the pew vantage point — and that the days that *do* show reasoning are the days where the one source that reports it (`codex`) happened to run.

That is the kind of finding that only a per-source, per-day decomposition can produce. The aggregate `reasoningTokens: 547834` line is the wrong abstraction. The 6×8 source-by-day matrix, with its three regions of structural zero and its two corner cells of dense reporting, is the right one.
