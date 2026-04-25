# Source-Coupled Verbosity: Why `--by-source` Is Not Just Another Group-By

When a model behaves differently depending on which client called
it, you have learned something about the **client**, not the
model. Most token-telemetry pipelines collapse this distinction by
default — they aggregate per model, summing across every producer
that issued the requests. The `--by-source` refinement shipped in
pew-insights v0.4.50 (commit `de7f1ad`, bumping 0.4.49 → 0.4.50)
treats it as a first-class axis. The numbers it surfaces on a real
fleet are sharp enough to settle arguments that pure per-model
views can only restate.

The headline finding from the v0.4.50 live-smoke run, on a
window of 386 rows from 2026-04-22 onward:

```
model            source       rows  input        output     ratio
---------------  -----------  ----  -----------  ---------  ------
claude-opus-4.7  opencode     119   74,865,619   8,877,453  0.1186
                 claude-code  5     6,799,703    370,680    0.0545
                 hermes       57    4,359,721    481,923    0.1105
```

Same model. Same window. Same fleet. The output-to-input ratio
is 2.18× higher under `opencode` than under `claude-code`. The
question this post is about is: **what does that gap actually
mean, and why is the per-model collapsed view structurally unable
to surface it?**

## The collapse problem

Every per-model report in the family — `cost`, `prompt-size`,
`output-size`, `cache-hit-ratio`, `output-input-ratio` — sums
across producers by default. A row in the queue carries a
`source` field (the producer CLI: `opencode`, `claude-code`,
`hermes`, etc.), but the model-keyed groupBy treats it as
irrelevant metadata. Two rows with the same model but different
sources land in the same bucket and are added together.

This is the right default. The bulk question — "how much did opus
cost this week?" — does not care which producer made the call.
But the collapsed view also makes a structural claim: that the
model's behaviour is a property of the model, not of the
producer–model interaction. That claim is empirically false, and
v0.4.50's source breakdown is the cheapest existing way to refute
it.

## The producer is part of the prompt

A "model" in the sense of "weights and decoding parameters" is
deterministic given input. A "model" in the sense of "what shows
up in the queue line under `model_name`" is the weights plus
whatever system prompt, tool schema, retrieval payload, and
formatting wrapper the producer added before the request hit the
inference endpoint.

The producer adds:

- a system prompt that instructs the model to respond in a
  specific shape (terse for tool-loop agents, expansive for chat
  REPLs);
- a tool schema that biases output toward `tool_use` blocks
  versus prose;
- conversation history compaction that controls how much the
  model "remembers" of prior turns;
- formatting and stop-sequence configuration that bounds output
  length.

All of this is invisible at the per-model aggregate. Opus called
from a tool-loop agent and opus called from an interactive REPL
are the same set of weights producing two different output
distributions because the **prompts are different**. The
collapsed view averages across both and reports a meaningless
midpoint.

## What the v0.4.50 numbers actually say

Re-read the table:

```
claude-opus-4.7  opencode     0.1186   (8.88M output / 74.87M input)
claude-opus-4.7  hermes       0.1105   (0.48M output /  4.36M input)
claude-opus-4.7  claude-code  0.0545   (0.37M output /  6.80M input)
```

The two tool-loop producers (`opencode`, `hermes`) cluster at
0.11–0.12. The interactive REPL (`claude-code`) sits at 0.054.
That is a 2× gap between two ways of calling the same weights.

The natural reading is that `claude-code`, in this fleet, is being
used for short interactive turns where the user types a question
and the model returns a focused answer. The tool-loop producers
generate longer chains of reasoning + tool-use blocks per call,
inflating output mass relative to input mass.

The changelog summary is sharper:

> opus is ~2× as chatty per input token under tool-loop producers
> vs the interactive REPL.

That summary is only sayable because the source axis exists. From
the collapsed per-model view, opus is one number (0.1131 in the
same window). That number is the weighted average of the three
sources, dominated by `opencode` because it carries 87% of the
input volume. It tells you nothing about whether the verbosity
signal is uniform or producer-driven.

## The control case

Producer-coupling is not universal. The same v0.4.50 live-smoke
output also reports `gpt-5.4`:

```
gpt-5.4   openclaw   145   373,042,260   3,005,039   0.0081
          hermes     1      15,400        67          0.0044
```

The 0.0044 row is n=1 and noise; the real signal is that
`openclaw` runs at 0.0081, the bulk per-model aggregate is 0.0081,
and there is functionally no producer separation to find. As the
changelog puts it:

> `gpt-5.4` is uniform across `openclaw` and `hermes` (~0.008),
> confirming its terse-completion profile is a model property,
> not a client-prompting artefact.

That is the right way to read the source axis: as a test for
**which observed behaviours are model properties versus
producer-coupled artefacts**. When all sources collapse to the
same ratio, the property is intrinsic. When they spread, it is at
least partly the producer.

This is information you cannot derive from running the per-model
view at higher temporal resolution, or with tighter windows, or
with more filters. The producer dimension is orthogonal to all of
those, and collapsing it is information loss that cannot be
recovered downstream.

## Display-only, denominator-stable

A subtle but important property of the v0.4.50 implementation:
turning on `--by-source` does not change the bulk numbers.
Quoting the release notes:

> Display only — global denominators (`consideredRows`,
> `totalInputTokens`, `totalOutputTokens`, `overallRatio`) and
> per-model `ratio` / `meanRowRatio` are byte-identical to the
> un-split run; only the new `bySource` map is populated.

This matters. A common anti-pattern in dashboard design is that
turning on a refinement also rebases the denominators, so the
"split" view and the "rolled-up" view report different totals for
the same window. Operators then cannot trust that drilling down is
a non-destructive operation.

Pew enforces the convention across the family: `--top`,
`--min-rows`, `--min-cv`, `--min-active-hours`, `--redact`,
`--by-source` — all of them are display-only. Toggling them is
guaranteed not to change any number that was visible before
toggling them. The split appears beneath; the rolled-up totals
above stay byte-identical. There are four explicit test cases
guarding this in v0.4.50 alone (767 total, up from 763), one of
which checks that `bySource: true` does NOT change global
denominators or per-model ratios versus the plain run.

This denominator-stability is what lets the producer breakdown be
trusted as a true refinement rather than a re-bucketing. You can
read the bulk row first, then drop into the per-source rows
beneath it knowing they are constrained to add up.

## When you actually need the source axis

Three operational situations make `--by-source` not optional:

**Investigating a verbosity regression.** A model's overall ratio
moves week-over-week. Did the model change, or did the producer
mix change? The collapsed view cannot tell you. The source
breakdown lets you check whether one producer's per-source ratio
moved (model-side change) or whether the producer-mix shifted
toward a chattier client (workload-side change). The two have
opposite remediations.

**Negotiating with a model vendor.** "Your model got more
expensive" is a much weaker claim than "your model got more
expensive *under our tool-loop producer specifically, while
remaining flat under our REPL*". The source axis is what turns the
former into the latter.

**Setting per-producer budgets.** If `opencode` runs opus at
0.1186 and `claude-code` runs opus at 0.0545, a single per-model
cost cap will systematically squeeze the wrong producer. The
source breakdown lets you set producer-specific output-token
budgets that match the producer-specific verbosity profile.

In each case the per-model view is structurally unable to give the
answer. The source axis is the cheapest fix.

## The general pattern: orthogonal aggregation axes

Token telemetry has at least four axes worth keeping orthogonal:

1. `model` — the weights identifier the inference endpoint
   recognises.
2. `source` — the producer CLI that minted the request.
3. `device_id` — the physical machine that ran the producer.
4. `hour_start` — the time bucket the row landed in.

Every report in the family picks one of these as the primary key
and collapses the other three by default. The
collapse-but-allow-refinement design lets each report stay
single-purpose while letting the operator drill across axes when
the bulk view is ambiguous. The v0.4.48 `device-share` report
keys on `device_id` and collapses model. `weekday-share` keys on
day-of-week and collapses everything else. `output-input-ratio`
keys on model — and now, with v0.4.50, optionally splits by
source.

The principle is: **the number of useful axes exceeds the number
of axes any single report can render legibly**, so each report
chooses its primary key and exposes refinement flags for the
others. The reader composes the views; the renderer does not try
to be the whole truth.

## What to take home

A per-model ratio is a producer-weighted average of per-producer
ratios. When producers behave differently — and tool-loop agents
do behave systematically differently from interactive REPLs — the
average is a number that nobody experienced. Operators see the
producer's view; vendors see the model's view; the collapsed
metric splits the difference and is wrong for both.

The v0.4.50 `--by-source` flag costs a single optional flag, a
nested map on the row type, and four test cases. It pays for
itself the first time someone asks "is opus chatty?" and you can
answer "yes from `opencode` at 0.1186, no from `claude-code` at
0.0545, and the bulk number 0.1131 is just the weighted average
of those two regimes". That is a different conversation than the
one you have when the only number on the table is 0.1131.

The cheapest defence against producer-coupled metric lies is to
keep the producer axis as a refinement on every per-model report,
default it off, and turn it on the moment a number stops making
sense. Information you collapsed in the builder cannot be
recovered in the renderer. Information you kept and chose not to
display can always be displayed later.
