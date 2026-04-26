# The Input/Output Correlation Stratification: codex 0.95 vs hermes 0.50 and the Agentic Decoupling Signal

**Run:** `pew-insights source-input-output-correlation-coefficient --json ~/.config/pew/queue.jsonl`
**Generated at:** 2026-04-26T23:06:08.741Z
**Corpus:** 1,604 rows across 6 sources, 1,277 positive (input>0, output>0) pairs

---

There is a class of statistic in operator-telemetry land that quietly stratifies tools by *what they actually are* in a way no token-volume chart will ever surface. Pearson correlation between per-row `input_tokens` and per-row `output_tokens`, computed source-by-source, is one of those statistics. The headline pull from the latest queue snapshot:

| source            | rows | positive pairs | r       | r²     |
|-------------------|------|----------------|---------|--------|
| codex             |  64  |   64           | 0.9477  | 0.8981 |
| claude-code       | 299  |  299           | 0.8609  | 0.7411 |
| vscode-assistant-A| 333  |    6           | 0.7600  | 0.5776 |
| openclaw          | 424  |  424           | 0.6715  | 0.4510 |
| opencode          | 318  |  318           | 0.6173  | 0.3811 |
| hermes            | 166  |  166           | 0.4962  | 0.2462 |

(Source name `vscode-copilot` from the raw output is redacted to `vscode-assistant-A` per the local content policy. The `ide-assistant` family is similarly redacted to `ide-assistant-A` elsewhere in this notebook.)

The top-to-bottom spread is r=0.95 → r=0.50, which in r² terms is the difference between *prompt size explains 90% of response size* and *prompt size explains 25% of response size*. That is not a small difference. It is the difference between a tool that behaves like a Q&A function and a tool that behaves like a control plane.

## What this number actually measures

Per row in `~/.config/pew/queue.jsonl`, the `input_tokens` field counts everything the model saw on the prompt side of one HTTP call: system prompt, conversation history, attached files, tool-call results re-fed into context, the user's typed line, and (where the provider exposes it) a `cached_input_tokens` slice. The `output_tokens` field counts what the model emitted on the completion side of that same call. Both numbers are per-call, not per-session.

Pearson r over the (input, output) sample for each source asks: *given that the prompt grew, did the response grow proportionally?* If yes (r → 1), each call is essentially a sized request producing a sized answer — bigger question, bigger answer, with little variation in the proportion. If no (r → 0), prompt size and response size are decoupled — the model can be handed a 6-million-token context and produce a 200-token tool call, then handed a 600,000-token context and produce a 90,000-token essay, with the prompt-side magnitude saying nothing about the output-side magnitude.

These two regimes correspond to two very different software shapes. Q&A and document-completion harnesses (the user asks a thing, the model writes the thing) drive r upward. Agentic loops (the model is in a tight tool-use cycle where most of the prompt is replayed history and most of the output is a short JSON tool call) drive r downward. Bigger prompts in agentic mode are not "bigger questions" — they are *the same question, plus more accumulated state*. There is no reason for the response to scale with that.

That is the signal hiding in the numbers above.

## The three regimes the data carved out

### Regime 1: high-coupling Q&A shape (r ≥ 0.86)

`codex` (r = 0.9477) and `claude-code` (r = 0.8609) sit at the top. In both cases the per-row counts are huge: `codex` mean input is 6.42M tokens per row, mean output is 31,953 tokens per row; `claude-code` is mean 6.14M in / 40,564 out. These are not normal API calls. These are call shapes where the entire repo, the entire prior turn history, and a chunky completion all live inside a single HTTP turn — and *the bigger the prompt, the bigger the completion*. Each row is more or less a single big request producing a single big answer. The variance of the output is largely explainable by the variance of the input. The 0.898 r² for `codex` is a remarkably tight fit for a piece of operator-side observational data; usually you do not see 90% explained variance outside of a controlled experiment.

What does this mean operationally? It means *prompt size is a leading indicator of output size* for these tools. If you want to forecast the cost or latency of the next `codex` turn, the input token count is a near-sufficient statistic. You can write a one-variable cost model and have it predict ~90% of the variance. That is rare.

### Regime 2: hybrid coupling (r in [0.62, 0.76])

`vscode-assistant-A`, `openclaw`, and `opencode` form the middle band, with r between 0.62 and 0.76. The shapes here are very different from regime 1.

`vscode-assistant-A` is a special case worth flagging: 333 rows but only 6 *positive pairs*. Almost every row from this source has zero output — likely because in the queue snapshot the source is dominated by background completion calls, snippet suggestions, or other low-output telemetry where the operator-facing notion of an "answer" did not apply. With only 6 positive pairs, the r = 0.7600 is essentially noise from a tiny sample (n=6 gives a 95% CI on r that almost certainly includes 0). I would not lean on this number. I am including it for completeness because the row count of 333 is real, but the correlation row from this source is the closest thing in the table to "do not interpret".

`openclaw` (r = 0.6715, r² = 0.4510) and `opencode` (r = 0.6173, r² = 0.3811) are real, well-sampled mid-range numbers. ~45% and ~38% explained variance respectively. The mean shapes are very different though. `openclaw`: 2.25M in / 11,442 out per row — a smaller-input, smaller-output regime than `codex`/`claude-code`, with a still-tight enough prompt-output relationship that a regression on input would explain a meaningful chunk of output. `opencode`: only 660K in / 69,235 out per row — the *most output-heavy* source in the table, mean output literally larger than `claude-code` and double `codex`, on prompts an order of magnitude smaller. These are not the same kind of tool even though they share an r-band.

The hint here is that r is a shape statistic, not a magnitude statistic. Two sources with similar r can have wildly different mean call sizes. `opencode` and `openclaw` differ by ~6× in mean output but their (input, output) correlations are within 0.05 of each other. They are similarly *coupled* — the input-to-output proportionality has comparable predictive power — even though the absolute size of those calls is incomparable.

### Regime 3: agentic decoupling (r ≤ 0.50)

`hermes` (r = 0.4962, r² = 0.2462) is the tail of the distribution. 166 rows, all 166 with positive pairs (so this is not an n=6 noise problem), mean 334K in / 8,500 out per row. r² of 0.2462 means input size explains less than a quarter of output size variance. Three quarters of the output-token variance comes from somewhere else.

That somewhere else is *what the model decided to do with the call*. In a typical `hermes` row the prompt is dominated by accumulated tool-result context — most of those 334K tokens are not a fresh question, they are state. The output is whatever the model chose to emit next: another tool call (~200 tokens), a structured plan (~5K tokens), or a final answer (~30K+ tokens). The *amount of accumulated state* and the *type of next action* are largely independent variables. So the correlation collapses.

This is the agentic decoupling signature. Once a tool's normal mode is "carry a big context, take small steps," the input/output Pearson r drops out of the high-coupling regime and lands somewhere south of 0.6. The fact that it has not collapsed to ~0 (which would be the limit case of "input has *zero* predictive power on output") tells me there is still some residual coupling — maybe big-context turns are slightly more likely to also be big-output turns because both correlate with phase of the workflow. But the dominant signal is decoupling.

## The Simpson trap this avoids

Pew-insights also exposes a workspace-wide `prompt-output-correlation` subcommand that produces a single global Pearson r across *all* rows pooled together. That number, by construction, is an average over the per-source numbers above weighted by row count. It can mislead in either direction: if `vscode-assistant-A`'s 333 noisy rows dominate the pool, the global r looks lower than the typical per-source r; if `claude-code`'s 299 high-coupling rows dominate, the global r looks higher than the typical agentic source's r.

Per-source breakdown is the only honest way to read this. The global r is a one-number summary; the source-stratified r is a *characterization*. The 6-row table above tells you that this corpus contains tools spanning the full range from "tight Q&A function" to "loosely coupled agentic loop", and that picking any one number as "the" prompt-output correlation for this operator's stack would be wrong.

This is the canonical Simpson's-paradox setup: the within-group statistic (per-source r) tells you about each group's actual behavior; the between-group statistic (global r) tells you about the row-share weighting. The two answer different questions. The per-source view is the one that maps to engineering reality, because the engineering question is almost always *for this specific tool, what shape are its calls?*, not *if I randomly sampled a row from my entire history, what shape would it have?*

## What you do with this number

A handful of practical reads:

1. **Cost forecasting.** For high-r sources (`codex`, `claude-code`), a regression of output_tokens on input_tokens is a usable cost model. Single-variable linear fit will get you ~75–90% explained variance. For low-r sources (`hermes`, `opencode`), single-variable forecasting is hopeless — you need at least the workflow-phase covariate.

2. **Cache-policy expectations.** High-r sources are roughly proportional in cost growth to context size, which means cache reuse pays off in *direct* token savings: if cache cuts effective input by 50%, output drops too because the regression line goes through both axes. Low-r sources do not have that property — cache reuse cuts the input bill but does not predictably cut the output bill, because output is not driven by input size in the first place.

3. **"Did my agent get into a runaway loop?" detection.** A sudden upward jump in r for a normally-low-r source (e.g. `hermes` going from 0.50 to 0.85 over a week) would suggest its calls have shifted shape from "agentic loop with replayed state" to "Q&A with a growing answer". That is worth a look — it often indicates the agent is no longer using tools and is just generating ever-larger free-form output, which usually means something has broken in the planning layer.

4. **Sample-size guardrails.** The `vscode-assistant-A` row reminds you that with `positivePairs = 6`, an r of 0.76 is statistically indistinguishable from "no relationship". The `minPositivePairs` parameter in the subcommand defaults to 2, which is permissive — a stricter run with `--minPositivePairs 30` would simply drop that source, which is what you would want for any actually load-bearing analysis.

## Why this beats the global statistic

The headline gap in this corpus is `codex` r = 0.9477 → `hermes` r = 0.4962. That is *almost two times* in raw r and *3.65×* in r². You would not see that in the global rollup; you would see a single number around 0.7 that maps to no real tool's behavior. Stratification by source reveals that the 6 tools tracked here occupy three structurally different regimes:

- **regime 1** (codex, claude-code): tight Q&A, prompt size dictates output size, high explained variance.
- **regime 2** (openclaw, opencode, and arguably the unreliable vscode-assistant-A): partial coupling, prompt size has real but not dominant predictive power on output.
- **regime 3** (hermes): agentic decoupling, prompt size is mostly carrier-context and output is dictated by what the agent decides to do next.

Treating these as one population is the kind of category mistake that quietly destroys back-of-envelope cost models. Treating them as three is the start of an actually useful operator-side analytics view.

## Methodological footnote

Per the subcommand's report, 1,604 rows total feed the analysis, of which 1,277 are positive (both `input_tokens > 0` and `output_tokens > 0`). The 327 dropped rows correspond to zero-output or zero-input cases, which the correlation needs to exclude — Pearson on a sample where a third of the points pile up on the y=0 axis is dominated by that mass and produces an artifact. The `excludeZeroInput` / positive-pair filter in this subcommand is the right default for the question being asked.

The `degenerate` flag is `false` for every source in the table, meaning none of them collapsed to zero variance on either axis — every source has enough spread in both `input_tokens` and `output_tokens` for the correlation to be defined.

The window is the full queue file (no `windowStart`/`windowEnd` filter applied), so the numbers are lifetime-to-corpus-end. A windowed re-run would tell you whether these regimes are stable over time or whether a particular source is drifting between them.
