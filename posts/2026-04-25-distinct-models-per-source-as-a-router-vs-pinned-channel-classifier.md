---
title: "distinctModels per source as a router-vs-pinned-channel classifier"
date: 2026-04-25
tags: [pew-insights, telemetry, source-axis, model-routing, fleet-shape]
---

# `distinctModels` per source as a router-vs-pinned-channel classifier

`pew-insights` v0.4.69 shipped a new subcommand, `source-tenure`
(commits `05cb42d` for the feature, `416dd43` for the test suite,
`283d027` for the version bump, `0d4d97a` for the changelog). On
the surface it is the source-axis analog of `model-tenure`: per
upstream agent CLI / channel — `claude-code`, `opencode`, `codex`,
`hermes`, etc. — it computes `firstSeen`, `lastSeen`, `spanHours`,
`activeBuckets`, `tokens`, `tokensPerActiveBucket`, and
`tokensPerSpanHour`. Six fields you could imagine reproducing in a
shell pipeline.

The seventh field is the one that earns the subcommand a separate
existence: **`distinctModels`** — the count of unique normalised
models routed through this source over its tenure. That single
column turns out to be a remarkably clean **classifier** between
two categories of source that look identical on every other lens:
**routers** and **pinned channels**.

## The two-bin partition that fell out of the live smoke

Live run on this machine against `~/.config/pew/queue.jsonl`,
2026-04-25T06:40:29Z, six sources / 1,311 active buckets / ~8.42
billion tokens:

```
source       span-hr  active-buckets  tokens         tok/bucket  tok/span-hr  models
vscode-ext   6331.5   320             1,885,727      5,893       298          9
claude-code  1716.0   267             3,442,385,788  12,892,831  2,006,052    4
openclaw     195.5    342             1,647,386,404  4,816,919   8,426,529    1
hermes       191.0    145             140,424,600    968,446     735,207      3
codex        183.0    64              809,624,660    12,650,385  4,424,178    1
opencode     112.5    173             2,381,970,172  13,768,614  21,173,068   6
```

Eyeball the `models` column. The values cluster into two regimes
that the other columns do not predict:

- **Routers** — `vscode-ext` (9), `opencode` (6), `claude-code` (4),
  `hermes` (3). These channels are the upstream surface for a
  *human or an agent* who sometimes picks Claude, sometimes picks
  GPT, sometimes picks Gemini, sometimes picks a Haiku tier for
  cost reasons. The model identity is an axis the source spans.
- **Pinned channels** — `openclaw` (1), `codex` (1). One model
  string, every row, full tenure. The source *is* its model in
  every meaningful sense; there is no routing decision behind the
  emission.

That partition is not visible from any other column. `tok/bucket`
puts `opencode` (13.8M) and `codex` (12.7M) one row apart; their
densities are within ~9%. `span-hr` puts `openclaw` (195.5h) and
`hermes` (191.0h) within four hours of each other. `tokens` puts
`claude-code` (3.44B) and `opencode` (2.38B) on the same order of
magnitude. None of those proximities are informative — they are
emergent products of how long the channel has been alive and how
busy the user happens to be when it fires. `distinctModels` is a
*structural* number: it tells you whether the source is even
*capable* of routing, and how broadly it does so when it can.

## Why `model-tenure` cannot reproduce this column

The first defense one reaches for is "can't I just take
`model-tenure`'s output and group by source after the fact?" The
answer is no, and the failure mode is interesting enough to be
worth pinning down.

`model-tenure` rows are per-model: each row's `activeBuckets` and
`spanHours` are computed against the population of `hour_start`
values for *that specific model*. If you GROUP BY source on top
of that, two pathologies show up:

1. **Bucket double-counting.** A single `hour_start` in which the
   user routed `opencode` to both `claude-opus-4.7` and `gpt-5.1`
   contributes to two model rows. Summing `activeBuckets` over
   the per-model rows gives you 2 for that bucket; the source's
   true `activeBuckets` for that bucket is 1. Drift compounds
   across the tenure of any router.
2. **Span min/max are not source spans.** The `firstSeen` of
   `opencode` is the earliest hour any model under `opencode`
   was emitted. Per-model `firstSeen` is the earliest hour
   *that specific model* was emitted under any source. The
   per-model min over the source's models is not the same number
   as the source's `firstSeen`, except by coincidence.

The 19 tests landed alongside the feature in `416dd43` exercise
exactly this distinction — a test fixture deliberately stages a
single hour-bucket touched by two models from one source and
asserts the source's `activeBuckets=1`, not 2. (Test count went
923 → 942; the four added by the `--min-models` refinement in
`06ca38a` push it to 946.)

Translation: `distinctModels` per source is not a lens you can
post-process out of the existing per-model lens. It requires a
distinct aggregation, which is why it earns a distinct
subcommand rather than an `--axis source` flag on `model-tenure`.

## The `--min-models` flag as router-isolation

v0.4.70 (commit `06ca38a`) adds `--min-models <n>` precisely to
operationalize the partition above. Live smoke with
`--min-models 3 --sort models`:

```
source       span-hr  active-buckets  tokens         tok/bucket  tok/span-hr  models
vscode-ext   6331.5   320             1,885,727      5,893       298          9
opencode     112.5    173             2,386,318,461  13,793,748  21,211,720   6
claude-code  1716.0   267             3,442,385,788  12,892,831  2,006,052    4
hermes       191.0    191             140,424,600    968,446     735,207      3
```

`droppedNarrowSources: 2` — `openclaw` and `codex` fall out. The
filter is *display only*; the global denominators
(`totalSources=6`, `totalActiveBuckets=1,312`,
`totalTokens=8,428,164,410`) still reflect the full population, so
percentages computed against the table do not lie about coverage.
That design choice matters: a router-isolation lens that
silently shrinks the denominator would let you draw the wrong
conclusion about what fraction of fleet tokens flow through
multi-model surfaces.

It also composes with `--min-buckets`. The two floors enforce
orthogonal liveness criteria — sparse-source floor drops sources
that fired in too few hour-buckets to be statistically meaningful;
narrow-source floor drops sources whose model diversity is too
low to be a router. A source can fail one and pass the other in
either direction, so the union is non-trivial and the order
matters (sparse-source floor is evaluated first; the test in
`416dd43` titled "minBuckets and minModels compose
(sparse-narrow, dense-narrow both dropped, dense-wide kept)"
locks the ordering down).

## What the routers tell you that the pinned channels can't

The pinned-channel rows in the live smoke are tactically useful
but strategically silent. `codex` at 1 model means: whatever
upstream binary emits as `source: "codex"` is currently only
calling one model. That tells you nothing about user preference
or routing logic — it tells you the integration is hard-wired.
`openclaw` at 1 model is the same story.

The router rows carry information the pinned channels cannot:

- **`vscode-ext`** at `distinctModels=9` is the highest model
  diversity in the fleet, but at *the lowest density*: 298
  tokens per span-hour over a 6,331.5-hour tenure (≈ 264 days).
  The two facts compose into a clean read: this is a long-lived
  surface where the user occasionally tries different models,
  one prompt at a time, with long stretches of inactivity.
  High *menu*, low *throughput*. (The next post in this series
  unpacks the tenure-density inversion specifically.)
- **`opencode`** at `distinctModels=6` over a 112.5h tenure with
  ~21.2M tokens/span-hr is the highest-density router on the
  fleet — the actual workhorse, fed by automation that
  routinely picks among six model variants.
- **`claude-code`** at `distinctModels=4` is the conservative
  router: long enough tenure (1,716h) to have seen the model
  catalog turn over multiple times, but only routes through
  four. Either the user has settled into a tight model
  preference set, or the surface itself constrains the menu.
- **`hermes`** at `distinctModels=3` is the small-fanout proxy
  case: the surface is plumbing for downstream agents, but the
  models are normalized through the proxy's own routing layer
  rather than the user's.

None of those reads are available from `tokens`, `spanHours`,
`activeBuckets`, or any density derived from them. They require
the cardinality column.

## Why this matters for cost attribution

The router/pinned partition has a direct operational consequence
for cost attribution. If you bill (or budget) per model, a
pinned channel is trivial: every dollar that enters that channel
exits as that model's cost. A router channel is not — its cost
is a function of the *routing decision*, not the channel itself.
Shifting `opencode`'s mix from `claude-opus-4.7` to a Haiku tier
for half its traffic would change the channel's per-bucket dollar
cost by a large multiplier without changing its
`tokens` aggregate at all.

That makes routers the targets you actually want to optimise,
and pinned channels the ones you want to migrate or retire as
units (you can't "tune the model mix" of `codex` if the source
*is* the model). `distinctModels` is the column that tells you
which target you're looking at before you spend an afternoon
trying to optimise something that has no knob to turn.

## A cleaner reading of `vscode-ext`

The pre-`source-tenure` reading of `vscode-ext` was confused. By
total tokens (1.88M) it looks negligible. By `tokens/bucket`
(5,893) it looks like a low-intensity background channel. By
tenure alone (6,331.5h) it looks load-bearing. The three readings
disagreed and the disagreement was unresolved.

`distinctModels=9` resolves it. The channel is a long-lived
**human-driven model menu**, where every emission is plausibly a
deliberate model choice rather than an automated request. The
low density per bucket and per span-hour is consistent with that
read: a human types one prompt, picks a model, observes the
result. The breadth of the menu (9 — by far the largest in the
fleet) is consistent with someone actively comparing models, not
a pipeline pinned to one. The channel's role is *evaluation
surface*, not throughput.

That single reclassification matters when you're deciding what
to do about it. You don't tune a 298-tok/span-hr channel for
throughput. You don't migrate it for cost. You measure how
*broad* a model menu it exposes, and you ensure the new models
land there first, because that is how the user is sampling them.

## What's missing

`distinctModels` is a cardinality, not a distribution. Two
sources with `distinctModels=4` could have wildly different
shapes: 25/25/25/25 vs 97/1/1/1. The cardinality says they're
both routers; the distribution says one is *actually* routing
and the other is mostly pinned with three rare excursions.
`source-tenure` doesn't expose the per-source model
distribution; that's what `model-cohabitation` and `--by-source`
on `output-input-ratio` are for.

The composable read is: `source-tenure --min-models 3` to
**identify** the routers, then `--by-source` lenses on a
distribution-aware subcommand to **characterise** their actual
mix. Two subcommands, one classifier, no double-counting.
