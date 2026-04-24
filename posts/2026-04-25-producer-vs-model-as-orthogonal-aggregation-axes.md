---
title: "Producer vs Model: Two Aggregation Axes That Look Redundant Until They Aren't"
date: 2026-04-25
tags: [observability, telemetry, aggregation, agent-cli]
---

# Producer vs Model: Two Aggregation Axes That Look Redundant Until They Aren't

If you have been instrumenting an agent CLI fleet for any length of time, you have probably aggregated token usage by model. It feels like the obvious cut: `claude-opus-4.7` cost X dollars this week, `gpt-5.4` cost Y, `gemini-2.5-pro` cost Z. Sum the rows, group by `model`, render a table, ship the dashboard. Done.

The trouble is that "model" is only one of two axes that matter, and the other axis — call it the **producer axis** — collapses entirely under model-only aggregation. The producer is the CLI, harness, agent, or shell that *generated* the call: `claude-code`, `codex`, `opencode`, `vscode-ext`, `hermes`, `openclaw`, and whatever else you have running. The same model can be reached through several producers; the same producer can route to several models. The two axes are genuinely orthogonal, and any one-axis view will hide things the other axis would surface immediately.

I learned this concretely yesterday when the local insights tool I run against my queue log shipped a `--by source` flag for its `output-size` subcommand. The model view said one thing. The source view said another. Both views were summarising the *same 1,367 rows*, the same 33,858,725 generated tokens, the same window. Neither view was wrong. They just answered different questions, and the question I had been asking — "where should I invest in completion-length optimisation?" — could only be answered by the source view.

This post is about why that distinction matters, what it looks like in real numbers, and how to design your telemetry pipeline so the orthogonality is preserved instead of accidentally laundered out.

## The setup

Concrete numbers help. The tool in question is a small TypeScript CLI that reads my local queue log (a JSONL append-only file, one row per request) and renders histograms. As of release 0.4.41 (shipped 2026-04-25), the `output-size` subcommand accepts `--by <model|source>`. Default stays at `model`. The row population is identical between the two views — same 1,367 rows after the standard drops (12 zero-output rows, 0 bad timestamps, 0 below the at-least floor). Mean output is 24,769 tokens, max is 416,890, the 8k+ buckets together cover 41.9% of rows.

The model view tells me the long-tail generation is concentrated in `claude-opus-4.7`. Useful, but actionable only if I want to switch models — which I don't, because the model is doing exactly what I asked it to do. The question I actually want to answer is upstream: *which client* is generating the long completions? That's a producer question, and the model view literally cannot answer it.

Run the source view and the picture changes:

```
source          rows  mean    p95      max      8k–16k  16k–64k  64k+
--------------  ----  ------  -------  -------  ------  -------  ----
openclaw        324   14,392  58,172   116,291  40      74       6
vscode-ext  321   3,537   14,290   37,381   25      11       ·
claude-code     299   40,565  188,818  416,890  52      56       61
opencode        218   58,144  282,231  333,890  22      66       55
hermes          141   8,590   25,827   38,184   31      30       ·
codex           64    31,954  100,274  163,782  9       21       13
```

Look at `vscode-ext` and `opencode`. Both can route to the same backend model. By row count, `vscode-ext` is the leader — 321 rows, more than any other producer. By mean output size, it is the *smallest* — 3,537 tokens, ~16x less than `opencode`'s 58,144. Its p95 is 14,290; `opencode`'s p95 is 282,231. Twenty times the tail length.

The model view collapsed those two together. They both touch `claude-opus-4.7` heavily, so in a `group by model` aggregation they were summed into one bucket and the distinction vanished. Only when you re-aggregate by the producer string does the asymmetry come back.

## Why this is not just a curiosity

If I look at the model view and decide to "optimise completion length for `claude-opus-4.7`", I will end up doing something silly: writing a generic prompt-engineering pass, or a generic completion-cap policy, or a generic post-processing trim, that lands on *every* call to that model. Most of those calls are coming from `vscode-ext` and are already short. I will be optimising rows that don't need optimising, and the actual culprit — the `opencode` tail with p95 of 282k — will continue undisturbed because the average across all producers will look "fine."

Whereas if I look at the source view, the answer is obvious within ten seconds: go change something about how `opencode` invokes the model. Maybe its system prompt is encouraging long answers. Maybe its task structure is one-shot rather than incremental. Maybe its tool result truncation policy is too generous. Maybe it is running in an `until done` loop with weak stop conditions. Whatever the cause, the *intervention point* is the producer, not the model.

This is the kind of mistake that doesn't show up as a bug. It shows up as engineering effort spent on the wrong layer, weeks later, when the dashboards still say `claude-opus-4.7 mean = 24,769` and nobody can explain why the bill keeps climbing.

## The deeper claim

Producer and model are orthogonal because they correspond to different *causal* stories about a request:

- **Model** answers "what computational substrate ran this call?" It controls the per-token cost, the context window, the rate limits, the eval performance.
- **Producer** answers "what software decided to make this call, and what shape did it ask for?" It controls the prompt structure, the tool-call patterns, the retry behaviour, the stop conditions, the system prompt, the truncation policies.

Cost, latency, and failure modes are *joint functions* of both axes. Holding one fixed and varying the other is the only way to attribute responsibility cleanly. If you only aggregate by one, you have collapsed a 2D distribution into a 1D marginal and you have lost the interaction term.

This is not unique to LLM telemetry. It is the same shape of problem as Simpson's paradox in classical statistics: aggregating across a confounding axis can flip the sign of a relationship. If `vscode-ext` overwhelmingly routes to a *cheaper* model and `opencode` overwhelmingly routes to a more expensive one, then the model view will show "expensive model has long completions" and you will conclude the model is the problem. It isn't. The producer is the problem. The model is just where the producer happened to land.

## What this implies for log schema design

The lesson here is one I wish I had internalised earlier: **every row in your request log needs to carry both axes as first-class fields, even when they look redundant.**

For a long time my queue log only carried `model`. The `source` field was something I added late, partly because the model name often *looked* like it implied the producer (e.g., `claude-opus-4.7` "obviously" came from `claude-code`, except it absolutely didn't — `opencode` and `openclaw` and `vscode-ext` and `hermes` all hit it too). Once I had both fields, every aggregation became more honest. Until I had both fields, I was occasionally drawing wrong conclusions and not noticing.

The general principle:

1. **Log the immutable identity of the model that ran the call.** Use the canonical id, not the alias. If your routing layer maps `claude-3.7-haiku-alias` → `claude-3.7-haiku-2026-03-01`, log the resolved one. Aliases drift; resolved ids don't.
2. **Log the producer string the request arrived with.** This is usually a header (`User-Agent`, `X-Client-Id`, `X-Producer`) or a CLI flag (`--client-tag`). Capture it raw. If it's missing, log a sentinel like `unknown` rather than dropping the row, so your denominators stay stable across reaggregations.
3. **Never compute one from the other.** No "if model starts with `claude-` then producer is `claude-code`" rules. They will be wrong eventually and they will be wrong silently.
4. **Resist the urge to merge.** The temptation to fold `opencode` and `openclaw` into a single "open* family" bucket because they "look similar" is a mistake. Keep them separate at log time. Aggregate at query time. You can always merge later; you cannot un-merge.

The 0.4.41 release notes had a clean way of putting this: "the grouping dimension echoes back in the report's new `by` field; the renderer adapts the table header automatically." That phrasing matters. The aggregation axis is *named in the output*, so a reader two months later does not have to guess which view they are looking at. If you ship a tool that supports multiple aggregation axes, name the axis in the output schema. Otherwise people will export tables, lose the context, and end up with two CSVs that look the same and disagree.

## What this implies for dashboards

If you maintain a dashboard for an agent fleet, here is a small checklist:

- For every per-model panel, build a per-producer panel right next to it. Same metric, different axis. Place them adjacent, not on different tabs.
- Show the row count for each axis, not just the aggregate metric. A producer with 4 rows and a producer with 400 rows can both have similar means; the small one is noise.
- Show the row count for the *intersection* (model × producer) cells too, when feasible. A heatmap with `model` on one axis and `producer` on the other, coloured by mean output size, is worth more than two separate bar charts. The cells with low row count tell you which combinations are rare and possibly meaningless to interpret; the cells with high row count tell you where the load actually lives.
- Keep an `unknown` / `null` bucket for missing producer data. Don't suppress it. The size of that bucket tells you how much of your fleet is unattributable, which is itself a metric worth tracking. (The 0.4.41 implementation falls back to an `unknown` sentinel for empty/missing source so global denominators stay stable. Same idea.)
- Resist merging "small" producers into an `other` bucket on the dashboard itself. Let the user choose. If the dashboard pre-merges, the small but anomalous producer will hide forever.

## What about more axes?

Producer × model is the minimum viable two-axis cut. There are obviously more. A non-exhaustive list of axes I've found useful at one time or another:

- **Task type** — was this a tool-call turn, a planning turn, a summarisation turn, a one-shot completion?
- **Conversation depth** — first turn, second turn, fifth turn? Token usage tends to grow with turn count, and producers differ in how they manage context.
- **Time of day** — more relevant than you'd think; some producers run in scheduled cron windows.
- **Caller identity** — which user, which workspace, which repo. Especially in multi-tenant setups.
- **Routing tier** — which provider gateway / proxy / region the call landed on. Two calls to "the same model" can hit different inference clusters with different latency profiles.

Every additional axis you log is one more orthogonal dimension you can re-aggregate against. The cost is bytes-per-row and a tiny amount of pipeline complexity. The benefit is that future questions you haven't thought of yet can be answered without re-instrumenting. This is the cheapest insurance you can buy in a telemetry pipeline. Buy it.

## A field test you can run today

If you have a queue log of any kind — JSONL, parquet, sqlite, doesn't matter — try this:

1. Pick your highest-cost or highest-volume metric. Mean output tokens, p95 latency, total cost, retry rate, whatever.
2. Aggregate it by model. Note the top-3 entries.
3. Aggregate it by producer (or whatever the closest producer-shaped field you have is — `client_id`, `user_agent`, `service_name`, `app_name`). Note the top-3 entries.
4. Compare. Are the rankings the same?

If they are, you are either in a single-producer or single-model environment, in which case carry on; or you are in a more complex environment but you got lucky and the two axes happen to align this week. They won't align next week.

If they are different — which is the common case in a heterogeneous agent fleet — congratulations, you just discovered an attribution gap that was costing you. Now go fix the producer that earned the largest delta between its model-marginal share and its producer-marginal share. That delta is, in expectation, where your next 10x optimisation lives.

## Closing

The two-axis discipline is unglamorous. It is one extra field in your log schema, one extra column in your dashboards, one extra `--by` flag in your CLI. It does not feel like infrastructure work. It feels like bookkeeping.

But every time I have pushed back against this discipline — every time I have thought "model is enough, I can infer the rest" — I have been wrong, and I have been wrong in ways that took weeks to notice because the model-only view was internally consistent. It told a coherent story. The story was just incomplete in a way that cost real money.

The 0.4.41 by-source view took the same 1,367 rows and showed me that `opencode` had a p95 of 282,231 tokens while `vscode-ext` had a p95 of 14,290 — both routing to the same model. That is a 20x ratio. It was invisible in every previous version of the tool. It is now the single most important number on my screen this week.

If your telemetry doesn't separate producer from model, separate them. Today. The view from the new axis will not be what you expect, and that is exactly the point.
