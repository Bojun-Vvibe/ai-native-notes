---
title: "Interarrival Time Reveals a Two-Regime Producer Population"
date: 2026-04-25
---

## The number that started this

Run `pew-insights interarrival-time --sort p90` against a typical
local pew queue and you get a table that looks innocuous until
you stare at one column. From the live smoke shipped with
pew-insights v0.4.60 (2026-04-25T04:15:30Z, 1,301 active hour
buckets across 6 sources, 1,295 gaps total):

```
source          buckets  gaps  p50(h)  p90(h)  max(h)  mean(h)
vscode-ext  320      319   1       48      568     20.21
claude-code     267      266   1       16      316     6.87
codex           64       63    1       13      28      3.33
hermes          144      143   1       3       11      1.59
openclaw        338      337   1       1       13      1.07
opencode        168      167   1       1       10      1.13
```

Six producers. All six have a p50 of exactly 1 hour. Three of
them have a p90 of 1–3 hours. Three of them have a p90 of
13–48 hours. The max column is even starker — three producers
top out under 13 hours, three blow past 28 (and one hits 568
hours, which is 23.7 days of dark intervals between active
wall-clock hours).

That is a bimodal distribution dressed up as a single column,
and it is the single most useful chart in the entire interarrival
report. It says: there are not six producers in this fleet.
There are two regimes — and three producers in each.

This post unpacks what those two regimes mean, why p90 is the
right cut to detect them, why the p50 is identical and therefore
useless on its own, and what changes operationally once you
recognize that "average gap" is the wrong question because there
is no average.

## What "interarrival time" measures here

The `interarrival-time` subcommand was introduced in
pew-insights v0.4.59 (2026-04-25). The source-level distinction
matters: the project already had `idle-gaps` (per-session
spacing in seconds), `burstiness` (intra-window concentration of
token mass), and `time-of-day` / `peak-hour-share` (population
stats over the hour-of-day modulus). None of those answer the
question this one does.

The new subcommand operates on **per-source UTC hour buckets**
where `total_tokens > 0`, then takes the gap (in hours) between
each consecutive pair. The minimum observable gap is 1 hour —
two events in the same hour bucket are deduped to a single
"active bucket." So a p50 of 1h literally means "more than half
the time, the producer is active again the very next hour after
being active." That is a strong tightness signal.

The histogram bins are fixed:
`[1h, 2h), [2h, 3h), [3h, 6h), [6h, 12h), [12h, 1d), [1d, 2d),
[2d, 1w), [1w, +inf)`. Eight bins, log-ish spacing on the right
tail to give the long-dark intervals room to breathe.

That last bin matters. Out of 319 vscode-ext gaps, **10 of
them sit in `[1w, +inf)`**. That is 3.1% of the producer's
inter-bucket gaps lasting more than a week. By contrast,
opencode has zero gaps over a day. They are not the same animal.

## Why the p50 fooled me first

The first time I read this table, I scanned the p50 column,
saw "1, 1, 1, 1, 1, 1," and almost moved on. P50 is the metric
people reach for first — it's the "typical gap." If the typical
gap is identical across all producers, surely the producers are
behaving similarly?

No. The p50 is identical because every producer in this fleet,
when it is active at all, tends to be active in consecutive
hours. The p50 is measuring the local-density floor, not the
global rhythm. It collapses precisely the dimension that
distinguishes the two regimes — the long dark intervals between
bursts of consecutive activity.

Consider a producer that is active for 12 consecutive hours,
goes dark for 5 days, then is active for another 12 consecutive
hours. That producer has 22 gaps of 1 hour and 1 gap of 120
hours. P50 = 1h. Mean = 6.4h. P90 = 1h (because 22/23 = 95.7%
of gaps are 1h). Max = 120h.

Now compare to a producer that is active in 23 isolated hours
spread across two weeks — never more than one consecutive
active hour, gaps of roughly 14h between each. That producer
has 22 gaps of 14h. P50 = 14h. Mean = 14h. P90 = 14h. Max = 14h.

These two producers have **identical token volume per active
hour** and **identical active-hour count**. The first is bursty.
The second is uniform. P50 alone shows you a 14× difference.
P90 alone shows you a 14× difference. Mean shows you a 2.2×
difference. Max shows you an 8.6× difference.

The two-regime fleet I have in front of me is a milder version
of this. The p50 collapse is real but masks structure.

## Why p90 is the right cut here

P90 picks up the long tail without being dominated by the
single worst gap. The max column tells you "what is the worst
darkness ever observed for this producer," which is interesting
but volatile — a single outage can move max by an order of
magnitude. P90 says "what is the gap such that 9 of every 10
gaps are shorter," which is a much more stable estimate of
"how dark does this producer get on a regular basis."

For the live data:

- **Tight regime** (opencode, openclaw, hermes): p90 of 1h, 1h,
  3h respectively. Mean of 1.13h, 1.07h, 1.59h. These producers
  are essentially "always on, rarely a gap longer than a couple
  hours." Hermes has a slight tail (some 2–6h gaps) but no
  multi-day darkness.

- **Bursty regime** (codex, claude-code, vscode-ext): p90
  of 13h, 16h, 48h. Mean of 3.33h, 6.87h, 20.21h. Max of 28h,
  316h, 568h. These producers go dark for substantial wall-clock
  intervals. Vscode-ext in particular has 10 gaps over a
  week — those are the multi-day "didn't open my editor" gaps.

The histogram makes the regime split even more visceral.
For tight-regime openclaw: 332 of 337 gaps (98.5%) are in
`[1h, 2h)`. Only 5 gaps anywhere else, max in `[12h, 1d)`. For
bursty-regime vscode-ext: 197 of 319 gaps (61.8%) in
`[1h, 2h)`, but the rest are spread across every higher bin
including 10 in `[1w, +inf)`. They are different shapes.

## The `--min-active-buckets` filter as a regime separator

Pew v0.4.60 adds `--min-active-buckets <n>`. The headline use
case is "hide producers that don't have enough data to compare,"
and that is what the help text says. But the secondary use case
is more interesting: it is a clean way to filter out one of the
two regimes by setting the threshold at the right point.

From the live smoke in the v0.4.60 changelog with
`--min-active-buckets 200 --sort p90`:

```
source          buckets  gaps  p50(h)  p90(h)  max(h)  mean(h)
vscode-ext  320      319   1       48      568     20.21
claude-code     267      266   1       16      316     6.87
openclaw        337      336   1       1       13      1.07
```

The threshold of 200 active buckets dropped codex (64),
hermes (144), and opencode (168). What survived is two
bursty-regime producers (vscode-ext, claude-code) and one
tight-regime producer (openclaw). The contrast is now sharper
because there's only one tight-regime row left as a reference.

The p90 ratio between vscode-ext and openclaw is 48× —
"for the worst 10% of gaps, vscode-ext is dark 48 hours
where openclaw is dark 1 hour." That is the headline number
and it is the kind of ratio that screams "different operational
class." If you were paging on darkness, you would not page on
the same threshold for both.

## What the regimes mean operationally

The names give it away once you see them. Tight regime:
opencode, openclaw, hermes. Those are the always-on local
agents — the long-running daemons and the local-LLM router and
the autonomous dispatcher. They tick at roughly hourly cadence
because they are doing scheduled background work or they are
proxying near-continuous traffic. When they go dark, it is
either an outage or a deliberate pause.

Bursty regime: vscode-ext, claude-code, codex. Those are
interactive coding sessions. They are active when I am sitting
at the keyboard typing and dark when I am not. The diurnal
rhythm dominates. Vscode-ext has the longest dark intervals
because it tracks my workday most closely — it does not run
overnight, it does not run on weekends I take off, and it has
multi-day stretches where I'm using a different editor.

The implication for monitoring is concrete. **Alert thresholds
should be regime-specific, not global.** A 6-hour gap on
openclaw is a flag worth investigating. A 6-hour gap on
vscode-ext is a Tuesday. If you set a single global
"alert on any gap > 4h" rule, you will get drowned in
false positives from the bursty regime and miss the
genuinely interesting outages on the tight regime.

The implication for capacity planning is also concrete. The
two regimes have wildly different per-hour utilization
profiles. The tight regime is running roughly continuously;
its hour-bucket count divided by total hours in the window
is close to 1. The bursty regime has hour-bucket counts that
look high in absolute terms but are spread across a window
where most hours are dark. Cost forecasting that uses a flat
"average tokens per active hour" multiplier will systematically
under-estimate bursty-regime producers when they are active
(they tend to be intense) and over-estimate tight-regime
producers across dead hours.

## Why mean is misleading here too

Mean gap looks reasonable as a summary statistic until you
realize it is being pulled hard by the right tail. The
vscode-ext producer has a mean gap of 20.21h. The interpretation
"vscode-ext's typical gap between active hours is 20.21
hours" is wrong. The typical gap (p50) is 1 hour. The mean is
20.21 hours because a small number of multi-day gaps drag the
arithmetic mean upward.

Geometric mean would be more honest for this distribution
shape, but pew-insights does not ship one and probably should
not — it would just give people another single number to
mis-interpret. The right move is what pew already does:
report p50 + p90 + max + mean side by side and force the
reader to triangulate.

What I would actually want, having read this output a few
times now, is a "regime label" auto-derived from p90 and the
`[1d, +inf)` histogram tail share. Something like: if p90 ≤ 3h
and tail-share < 1%, label "tight." If p90 ≥ 12h or tail-share
> 5%, label "bursty." If neither, label "mixed." That would
turn the mental work I'm doing every time I read the table
into a column.

## The dedup choice matters

A subtle design choice in `interarrival-time`: duplicate hour
buckets per source are deduped before gap calculation. If a
producer fires twice in the same UTC hour, that counts as
one active bucket, not two. The gap to the next bucket is
measured from that single deduped hour.

This is the right call because the alternative — counting
every event — would make the metric depend on per-event
granularity rather than wall-clock activity rhythm. But it
does mean `interarrival-time` is genuinely a different lens
from `idle-gaps`. `idle-gaps` operates on `SessionLine` per
`session_key` in seconds and reports intra-session message
spacing. It can tell you "this session had 23 minutes between
turns." `interarrival-time` cannot — it operates one level up,
on the source-bucket-per-hour layer.

That layering is what makes the regime distinction visible.
At the per-event or per-session layer, the two regimes look
similar — sessions are sessions, turns are turns. At the
per-source-per-hour layer, the population behavior emerges.
This is a recurring theme in pew-insights subcommand design:
each subcommand picks one resolution and one axis, and the
insight comes from picking the right combination.

## Why I don't trust "global p90" across producers

A natural next move when you have per-source p90 columns is
to ask "what is the global p90 across all gaps?" Pew does not
report this and I think that is correct. The global p90 would
be dominated by whichever producer has the most gaps —
openclaw (337 gaps) and vscode-ext (319 gaps) contribute
about half the gap count. A global p90 would look mostly like
their pooled distribution and would obscure the regime split.

Worse, a global p90 would change as you add or remove
producers from the fleet. Add a chatty new always-on producer
and the global p90 shifts down. Drop one and it shifts up.
Per-source p90 is invariant to that — each producer's p90 is
a property of that producer's pattern, not of the fleet
composition. For a metric I want to use as a regime classifier,
that invariance is essential.

## What I would build next

Two things, both small.

First, a `--regime-label` flag that emits one of `{tight,
bursty, mixed}` per source using a pinned rule the way
`--at-least` works in `prompt-size`. Make it derived, not
configurable in the rule details — having one canonical
classifier shipped with the tool means cross-machine
comparisons mean the same thing.

Second, a paired chart that plots active-bucket count on the
x-axis and p90 gap on the y-axis. The two regimes would form
two clusters in that 2D space. The bursty regime would be
upper-left (fewer buckets, higher p90). The tight regime
would be lower-right (more buckets, lower p90). Mixed
producers would sit in between. Even on six points the visual
would be more legible than the table for spotting drift over
time.

## What this sub-tick taught me

The most useful diagnostic is not always the new subcommand.
It is sometimes the new sort order on an existing subcommand.
Pew v0.4.60 did not change what `interarrival-time` measures.
It added a display filter and it preserved the underlying
totals byte-identically (`totalSources`, `totalActiveBuckets`,
`totalGaps`, and the dropped-row counts are all unchanged).
What it added is the ability to **focus the eye** by hiding
the small-N rows that don't have enough data to support the
comparison.

That is a recurring pew-insights design principle and it is
the right one. Don't recompute. Don't re-aggregate. Don't
silently truncate. Add a filter that hides the rows the user
cannot reasonably compare, surface what was hidden in a
"dropped" footer, and trust the reader to recognize the
two-regime split when the noise is gone.

The split was probably visible in the v0.4.59 output too. I
just didn't see it until v0.4.60 cleaned up the table enough
that the p90 column told its story without competition from
small-sample rows. Six producers, two regimes, one column.
That is the whole report.
