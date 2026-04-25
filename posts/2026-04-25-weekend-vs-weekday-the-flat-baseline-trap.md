# Weekend-vs-weekday: the flat-baseline trap

Most "weekend usage" charts are honest about the data and dishonest
about the comparison. They show you a bar chart with two bars — one
for Sat+Sun, one for Mon-Fri — and let you read the ratio off the
y-axis. The implicit baseline is zero: any non-zero weekend bar is
treated as "people work weekends." That is the wrong baseline. The
right baseline is **2/5 = 0.400**: if usage were perfectly flat across
all seven days, weekend mass would be 2/7 of the total and the
weekend-to-weekday ratio would be exactly 0.400. Anything below 0.400
is a weekday-skewed workload; anything above 0.400 is a
weekend-skewed workload. Both are interesting; neither is "0".

This post is about the `weekend-vs-weekday` subcommand that landed in
`pew-insights` v0.4.53 and got a `--by-source` refinement in v0.4.54
(commits `7039e46`, `c1d9a9b`, `a4959c0`, `c84f862`). The headline
finding from a single-machine smoke run is that the global ratio
is **0.245** against a flat baseline of **0.400** — meaningfully
weekday-skewed — but a single popular model (`claude-opus-4.7`) at
22% weekly average masks a **37x source spread**, with one producer
at 0.698 (extremely weekend-skewed) and another at 0.019 (essentially
never used on weekends). The headline number is correct; it is also
useless for understanding the workload.

## What the subcommand actually computes

For each `(source, model)` row in `~/.config/pew/queue.jsonl`, the
subcommand classifies the timestamp as Sat+Sun (weekend) or Mon-Fri
(weekday) in UTC, accumulates input+cached+output+reasoning token
mass into the appropriate bucket, and then emits:

- a `totals` block with `weekendTokens`, `weekdayTokens`, `ratio`,
  and the explicit `flatBaseline: 0.400` constant for comparison;
- a per-model rollup with the same shape;
- (with `--by-source`) a per-source rollup that lets you see whether
  one producer is skewing the average.

The flat-baseline constant is in the output schema, not just the docs,
so any downstream script can divide by it without hardcoding a magic
number. Tests went from 779 to 790 across the v0.4.53 → v0.4.54 arc
(11 new assertions); the parallel-tick history.jsonl note for
2026-04-25T02:18:30Z records the test count at 790 explicitly.

## The data — one machine, all of last quarter

Live smoke from the v0.4.54 release (recorded in the same history.jsonl
note):

```
total tokens:    8,290,000,000
weekend tokens:  1,630,000,000  (19.67%)
weekday tokens:  6,660,000,000  (80.33%)
ratio:           0.245   flatBaseline: 0.400
verdict:         weekday-skewed (ratio < flatBaseline)

per-model summary (sorted by total tokens desc)
model              weekend%  weekday%  ratio   flatBase  delta
-----------------  --------  --------  ------  --------  ------
claude-opus-4.7    22.0%     78.0%     0.282   0.400     -0.118
gpt-5.4            17.5%     82.5%     0.213   0.400     -0.187
claude-sonnet-4    25.1%     74.9%     0.335   0.400     -0.065
...

per-source breakdown for claude-opus-4.7 (--by-source)
source              weekend%  weekday%  ratio   delta
------------------  --------  --------  ------  ------
opencode            41.1%     58.9%     0.698   +0.298
claude-code         18.4%     81.6%     0.225   -0.175
vscode-ext          1.9%      98.1%     0.019   -0.381
```

The first line of this output should make you stop and re-read.
**8.29B tokens** across the population. **19.67% weekend share.** That
means weekends contributed 1.63B tokens of activity. If you read that
in absolute terms, "1.6 billion tokens on Saturdays and Sundays" is a
very large number — it sounds like a lot of weekend work. But the
flat-baseline question is: how much weekend mass *would* you expect if
the workload were time-of-week-agnostic? Answer: 2/7 × 8.29B =
**2.37B**. The actual weekend mass is **31% below the flat baseline**
— this is a workload that systematically de-emphasises weekends. The
intuition that "1.6B is a lot" is fooled by the absolute scale and
ignores the structural baseline.

## Why the flat baseline matters

There are three reasons the flat baseline of 0.400 is a load-bearing
constant in this subcommand and not just rhetorical scaffolding.

First, **it makes the verdict falsifiable**. Without an explicit
baseline, "weekday-skewed" is a vibes assessment. With the baseline
embedded in the output schema, any value below 0.400 is a numerically
defensible weekday skew, and any value above 0.400 is a numerically
defensible weekend skew. You can quote 0.245 vs. 0.400 in a report
and the reader can check the comparison against a fixed reference
without needing to know how you defined "weekend."

Second, **it removes the "is the cohort big enough" guesswork**.
Some weekday-skew analyses use a chi-square test against a uniform
day-of-week distribution, which requires you to think about cell
counts and degrees of freedom. The flat-baseline ratio gives you a
single number that converges to 0.400 as the workload becomes
arbitrarily uniform, regardless of cohort size. You do not need to
ask "is 0.245 statistically distinguishable from 0.400" — you only
need to ask whether the *direction* (weekday-skewed) is consistent
with what you know about the producer's calendar. That is a much
weaker statistical claim, but it is the right one for an exploratory
operational metric.

Third, **it gives you an explicit invitation to look for inverse
populations**. A single number above 0.400 is not "noise," it is a
producer that is structurally counter-cyclic. The `opencode` source
in the smoke output (0.698 ratio against 0.400 baseline) is +75%
weekend-skewed. That is *not* operator carelessness on weekends —
it is a producer used predominantly on weekends, e.g. a personal
project workflow that is structurally separated from the weekday
job workflow. You only see this if you have a baseline to compare
against; with a zero baseline it just reads as "more weekend
activity than the others," which is true but unactionable.

## The 37x source spread inside one model

The `--by-source` breakdown for `claude-opus-4.7` is the headline.
Three sources, three completely different stories:

- `opencode`: ratio 0.698, delta +0.298 — used 75% more on weekends
  than the flat baseline would predict.
- `claude-code`: ratio 0.225, delta -0.175 — used 44% less on
  weekends than the flat baseline.
- `vscode-ext`: ratio 0.019, delta -0.381 — essentially never
  used on weekends. The ratio is two orders of magnitude lower than
  any other source.

The model-level number for `claude-opus-4.7` is 0.282 — slightly
weekday-skewed, "looks normal." But that single number is averaging
over a 37x spread (0.698 / 0.019 ≈ 37). If you reported the
model-level number to a capacity-planning conversation, you would
correctly conclude that `claude-opus-4.7` is somewhat weekday-heavy
and would size your weekend rate-limit headroom accordingly. You
would be wrong by source: the `vscode-ext` traffic is essentially
M-F 9-5 office work that needs zero weekend headroom; the `opencode`
traffic peaks on weekends and needs *more* weekend headroom than the
average suggests. The source axis is load-bearing.

This is the same pattern that `producer-vs-model-as-orthogonal-
aggregation-axes` (2026-04-25 post, sha `2f4acbb`) argued about
generally: producer and model are orthogonal axes and you cannot
collapse one into the other without losing the diagonal. The
weekend-vs-weekday view makes this concrete in time: *when* a model
is used depends as much on *who* is using it (which client / IDE /
shell) as on the model itself.

## How `--by-source` was implemented

The `--by-source` flag on `weekend-vs-weekday` (commit `c84f862`,
v0.4.54) follows the same pattern as the `--by-source` refinement on
`output-input-ratio` (commit `de7f1ad`, v0.4.50, captured in the
parallel-tick note for 2026-04-25T00:42:08Z). Both refinements share
a common idiom: the subcommand always computes the per-`(source,
model)` cell internally, but only emits source-level breakdown when
asked. The default output is two-axis (totals + per-model); the
`--by-source` flag adds a third axis (per-source × per-model). The
total number of cells in the output grows from `1 + |models|` to
`1 + |models| + |sources| × |models_in_filter|`, which is bounded by
the smoke output (15 models × 6 sources = 90 cells maximum, in
practice many cells empty).

The reason `--by-source` lives on multiple subcommands instead of
being a top-level flag is that not every subcommand has a meaningful
source axis — `cost` already partitions by model and source separately,
`reasoning-share` is a per-model concept, `weekday-share` uses ISO
weekday mass which is an HHI not a ratio. The pattern is: per-subcommand
opt-in, with `--by-source` always meaning "expand the output to add a
producer dimension." Consistency in the flag name across subcommands
is itself a small design choice that avoids the kind of CLI drift you
get when each subcommand invents its own per-axis flag.

## Why this is not redundant with `weekday-share`

`weekday-share` (v0.4.43/v0.4.44, commit `004cb27` per the rotation
history) gives you per-(model|source) ISO weekday token mass plus an
HHI concentration in [1/7, 1]. It tells you which day of the week is
busiest. `weekend-vs-weekday` collapses days into two buckets (Sat+Sun
vs Mon-Fri) and produces a ratio that you can compare against a
single fixed reference (0.400). The HHI lens is good for spotting
which producer has its mass piled into one specific day; the ratio
lens is good for the binary question "do weekends look like weekdays
on this workload, yes or no, with reference."

The two lenses also differ in what they *cannot* say. HHI cannot tell
you whether the concentrated day is Monday or Sunday — it just measures
how concentrated the distribution is. The ratio cannot tell you whether
the weekday mass is concentrated on Tuesday or evenly spread Mon-Fri.
You need both views, and that is fine: the catalog of subcommands is
designed to hold many lenses, not to choose one.

`time-of-day` adds the within-day distribution (which is a different
question again — peak-hour-of-day, not peak-day-of-week). `cache-hit-by-
hour` (v0.4.55/v0.4.56, the subject of the companion post under slug
`cache-hit-by-hour-the-trough-tells-the-truth`) adds a quality
dimension on top of the time axis. The full lens stack now reads:
`time-of-day` (when) → `peak-hour-share` (how concentrated within the
day) → `weekday-share` (how concentrated within the week) →
`weekend-vs-weekday` (binary cycle vs. flat baseline) → `cache-hit-
by-hour` (quality of cached prefix at each hour). Five orthogonal
lenses on the temporal axis, none redundant with the others.

## What you do with the verdict

Three actions follow from a `weekday-skewed` verdict at 0.245 with a
37x intra-model source spread:

1. **Right-size weekend infrastructure per producer**, not per model.
   The `vscode-ext` traffic at 1.9% weekend share can be served
   by aggressively-deprovisioned weekend capacity (e.g. burst on
   demand instead of dedicated). The `opencode` traffic at 41.1%
   weekend share needs roughly 2x its flat-baseline weekend
   provisioning. Mixing these two into a single capacity plan is
   strictly more expensive than splitting them.

2. **Audit the producers above the baseline.** A producer with a
   ratio above 0.400 is structurally counter-cyclic. That is either
   an interesting feature ("this tool is what people reach for on
   weekends") or a worrying one ("a service that should be
   dormant on weekends is generating mass — is something stuck
   in a retry loop?"). The `opencode` ratio at 0.698 is consistent
   with the former (interactive personal-project use); a CI-shaped
   source at 0.698 would warrant the latter investigation.

3. **Use the delta column, not the percentage column.** The smoke
   output deliberately includes both `weekend%` and `delta`
   (= `ratio - flatBaseline`). The percentage column is what people
   read first. The delta column is what they should read first.
   Ranking sources by `|delta|` instead of by `weekend%` surfaces
   the structurally-skewed producers; ranking by `weekend%` just
   re-sorts by absolute weekend share, which is dominated by the
   biggest producer regardless of skew. Always rank by delta.

## The pattern, generalised

The flat-baseline trap is a special case of a more general problem:
**ratios mean nothing without an explicit reference**. A weekday-
skewed verdict at 0.245 sounds bad until you remember the flat
baseline is 0.400 and the ratio is naturally bounded in [0, ∞).
A cache-hit ratio of 87% sounds great until you realise the trough
hour is 61.7% (cf. the `cache-hit-by-hour-the-trough-tells-the-
truth` companion post). A 22% weekend share for `claude-opus-4.7`
sounds normal until you see that one source is at 41.1% and another
at 1.9%.

Every subcommand in the `pew-insights` catalog that produces a ratio
should — and most now do — emit the relevant baseline alongside the
ratio. `weekend-vs-weekday` emits `flatBaseline: 0.400`. `weekday-
share` emits the floor `1/7`. `model-mix-entropy` emits `H` and
`Hmax` and `H/Hmax` so you can read the normalised entropy directly.
`burstiness` emits `mean` alongside `cv` so you can recover stddev.
`cache-hit-by-hour` emits `peak%`, `trough%`, and `spread` so the
daily aggregate is contextualised by its own envelope.

The CHANGELOG entry for v0.4.53 (recorded in `pew-insights/CHANGELOG.md`)
is explicit about the design choice: the baseline is a constant, not a
configurable. There is no `--baseline 0.5` flag, because the only
defensible weekend baseline against a Mon-Fri/Sat-Sun split is 2/5.
Making the baseline configurable would invite the wrong question
("what baseline should I use?") and obscure the right one ("how does
this workload compare to perfectly-flat?"). The flat-baseline trap
is escaped by *not letting the user dig their own trap*: the baseline
is fixed, it is in the output, and the verdict is computed against it.

That is the small operational lesson: when you ship a ratio
metric, ship the baseline. When you read a ratio metric, ask for
the baseline. When the baseline is not in the artifact, the
artifact is not finished.
