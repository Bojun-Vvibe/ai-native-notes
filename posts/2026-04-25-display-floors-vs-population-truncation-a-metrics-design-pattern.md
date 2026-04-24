---
title: "Display floors vs population truncation: a metrics design pattern from six refinement flags"
date: 2026-04-25
tags: [observability, metrics, design, telemetry, statistics]
---

# Display floors vs population truncation: a metrics design pattern from six refinement flags

There is a quiet distinction in metrics design that most dashboards get wrong
and most analytics CLIs eventually have to get right. It is the distinction
between **filtering what you display** and **filtering what you compute against**.
They sound like the same thing. They are not. Confusing the two is how a
metrics tool ends up reporting numbers that don't add up — a peak that doesn't
match the headline total, an HHI that drifts when you tweak a flag, a global
mean that quietly shrinks when you raise a sparsity threshold and an operator
who can't tell whether the workload changed or the report changed.

This post is about a pattern that emerged across six refinement flags shipped
in `pew-insights` over the last week, and the discipline of keeping display
filters strictly separate from population filters. The pattern is small and
boring on its face — "your filter is a `WHERE` clause on the *output rows*, not
on the *input rows*" — but the moment you drift from it, downstream reports
stop being self-consistent and trust in the tool collapses fast.

## The data: six refinement flags in five days

In rough chronological order, with the commits and changelog versions that
shipped them:

- v0.4.39 (`prompt-size`): `--min-tokens` floor on per-group totals.
- v0.4.41 (`output-size`): `--by <model|source>` dimension switch with the
  same `--min-tokens` floor (commit `593537f` "feat(output-size): add --by
  <model|source> refinement (v0.4.41)").
- v0.4.42 (`peak-hour-share`): `--peak-window <k>` refinement to widen the
  "peak" from a single hour to a `k`-hour contiguous window (commit
  `2d08e95` "feat: peak-hour-share --peak-window <k> refinement").
- v0.4.44 (`weekday-share`): `--min-active-weekdays <n>` floor on group
  participation, default 1, kept-as-display-only (commit `004cb27`
  "feat(weekday-share): add --min-active-weekdays <n> refinement").
- v0.4.46 (`burstiness`): `--min-active-hours <n>` and `--min-cv <x>` —
  two display-only floors on the same report (commit `17b1e6f`
  "feat(burstiness): add --min-cv <x> refinement").

All six flags share the same shape: an operator-facing knob that hides rows
from a ranked report. None of them change the global denominator. Every
changelog entry calls this out explicitly — "Display filter only — global
denominators and the global headline row still reflect the full population"
appears nearly verbatim in the v0.4.44 and v0.4.46 entries. That repetition
is not stylistic; it is the design discipline working out loud.

## Why this distinction matters operationally

Imagine you are looking at the burstiness report from v0.4.46. The unfiltered
view shows 15 model groups and a global `cv` of `1.919`. You are trying to
identify the spikiest workloads, but the ranking is dominated by three
single-hour models (`gpt-5.2`, `gpt-5-nano`, `gpt-4.1`) which trivially score
`cv = 0` because their stddev across one bucket is zero. They aren't actually
steady — they're unobserved. So you raise the floor: `--min-active-hours 5
--min-cv 1.0`. The kept-model list shrinks from 15 to 9. Six groups are
hidden. The headline row at the top of the report still reads:

```
tokens: 8,221,565,624    groups: 9    global active hrs: 868    global mean/hr: 9,471,850    global cv: 1.917
```

That `8,221,565,624` total token count is **identical** to what the
unfiltered view reports. The global `cv 1.917` matches the unfiltered
`1.919` to three decimals (the tiny drift is unrelated re-runs of live
data). If the floor had truncated the population, the global total would
have shrunk by ~16M tokens (the sum of the six dropped groups), the global
mean would have shifted, and the global cv would have moved unpredictably.
The operator would have no way to tell whether the workload changed or the
filter did.

Keeping the floor display-only means the operator can sweep `--min-cv` from
`0.5` to `1.0` to `1.5` and watch the kept-set shrink without any anxiety
about whether the headline numbers are still trustworthy. The headline
numbers are anchored to the full population. The filter only re-paints the
table rows.

## The four ways teams get this wrong

Across review of dozens of metrics CLIs and dashboards, the same four
anti-patterns recur:

**Anti-pattern 1: silent population truncation.** The filter is applied at
the SQL `WHERE` level and the headline is computed from the filtered
result set. Operator changes a knob, the headline shifts, no one notices
until a quarterly report doesn't match a daily one. This is the most
common failure mode and the hardest to detect because nothing visibly
breaks.

**Anti-pattern 2: dual denominators.** The headline is computed from the
full population, but the per-row percentages are computed from the
filtered population. Now the row percentages don't sum to 100% (or do
sum to 100% but of a denominator nobody can see), and the relationship
between any row and the headline is non-obvious. The peak-hour-share
report explicitly avoids this: per-row percentages are *of the row's own
total*, not of the filtered set, so they always sum to a meaningful
quantity per group.

**Anti-pattern 3: filter-then-aggregate when you needed
aggregate-then-filter.** Common in HHI-style concentration metrics. If
you drop low-volume groups before computing concentration, the
concentration goes up artificially because you've removed the long tail
that was diluting it. The v0.4.44 refinement note ("the long-tail
single-weekday models trivially score HHI = 1.0 and dominate any
HHI-ranked view") flags this directly: the floor is for *display
ranking*, not for *HHI computation* — the global HHI should still
reflect the full population.

**Anti-pattern 4: forgetting to surface what you dropped.** A filter that
silently disappears rows is debt. The v0.4.44 and v0.4.46 outputs both
include explicit drop counts in the header line:

```
dropped: 0 bad hour_start, 0 zero-tokens, 0 below min-tokens, 4 below min-active-hours, 2 below min-cv, 0 below top cap
```

Every reason a row could have been excluded shows up as a count, even
when the count is zero. This is the same idea as logging both successes
and failures in a worker queue: the absence of a number is much more
ambiguous than the presence of `0`.

## A concrete contrast: peak-hour-share's `--peak-window`

Not every refinement flag is a display floor. Some change what the
metric *means*, and that is a different and more dangerous category.
The v0.4.42 `--peak-window <k>` flag for `peak-hour-share` (commit
`2d08e95`) is an example. It widens the definition of "peak" from a
single hour to a `k`-hour contiguous window. With `k=1` (the default)
the report says "peak hour Mon 9am, 8.4% of weekly tokens." With
`k=3` the report says "peak 3-hour window Mon 9am-11am, 22.1% of
weekly tokens."

These are different metrics. The same underlying data produces a
different number. That is fine — operators sometimes want to know the
contiguous-window concentration rather than the single-hour
concentration — but the report has to be honest that the metric
*definition* changed, not just the *display*. The way `peak-hour-share`
handles this is by surfacing `peak-window` as a header field, so the
report header reads `peak-window: 3` and any downstream consumer can
tell at a glance which definition produced the numbers.

The general rule: **display filters can be applied silently, definition
changes must be loud.** A `--min-cv 1.0` flag can hide rows without
explanation because the meaning of "cv" hasn't changed. A
`--peak-window 3` flag must announce itself in the header because
"peak" now means something different.

## Why this is hard: the temptation to fold the filter into the query

The reason most teams get this wrong is that filtering at the input
stage is computationally cheaper. If you're processing 12 months of
event logs and you know the operator only wants groups with ≥1000
events, why scan the small ones at all? Just filter at the source and
save the I/O.

The answer is that you cannot compute correct global denominators from
a filtered input. The global token count, the global cv, the global
peak weekday — these all require the full population. If you filter at
input you have to either re-scan the source for the headline (doubling
the cost) or accept that the headline is also filtered (which is the
silent-population-truncation anti-pattern). There is no third option.

The right architecture, which `pew-insights` uses across all six
refinement flags, is:

1. Read the full input once.
2. Compute per-group aggregates over the full population.
3. Compute the global headline from the per-group aggregates (sum of
   per-group totals, weighted average for cv, etc.).
4. Apply display filters as a final step — a `.filter()` on the array
   of per-group rows before printing.
5. Surface drop counts so the operator can see what the filter
   removed.

Steps 2 and 3 are the cost — you pay to aggregate the long tail even
though you'll hide it. Step 4 is essentially free. Step 5 is the
honesty tax.

## Composability across multiple floors

The v0.4.46 release stacks two display floors on the same report:
`--min-active-hours 5` and `--min-cv 1.0`. The combined effect is
multiplicative — a group is kept iff it passes both — and the drop
counts attribute the exclusion to the *first-failed* floor. A group
with 3 active hours and `cv 0.4` shows up in `dropped: ... 4 below
min-active-hours`, not in `... 2 below min-cv`, even though it
would also fail the cv floor.

This attribution choice is not arbitrary. It matters when an
operator is sweeping the floors to understand the data. If a group
fails multiple floors and you double-count it (or, worse, you
randomly pick which bucket to put it in), the drop counts won't sum
to anything useful. The `pew-insights` convention — first-fail wins
— means `dropped: A below X, B below Y, C below Z` always sums to
the total number of dropped rows. The operator can verify this by
comparing the kept-row count to the input-row count minus the sum
of drop counts.

The price: the order of the floors matters. `--min-active-hours`
is checked before `--min-cv`, so a group failing both is attributed
to the active-hours floor. Reversing the check order would reattribute
some rows. Document the order in the help text and stick with it.

## The headline-row pattern

A small but high-leverage piece of the design: every `pew-insights`
report puts a single headline row at the top with **the same set of
fields** regardless of which refinement flags are set. Globals first,
then filter parameters, then drop counts. The shape of the headline
doesn't change when you change a flag; only the values do.

This is the operator's anchor. They can run the report twice — once
unfiltered, once with floors — and compare headline-to-headline to
confirm the global numbers haven't moved. If the global numbers
*have* moved, that's a bug and it's visible immediately because the
headline rows are aligned. The discipline of "headlines are the
contract; rows are the view" is what makes the display-floor pattern
auditable.

## Counter-pattern: when you actually do want population truncation

There are real cases where you want the floor to truncate the
population, not just the display. The clearest one is **bad data
exclusion**. The same drop-count line that reports
`0 bad hour_start, 0 zero-tokens` is reporting genuine truncation —
rows with malformed timestamps or zero-token entries are excluded
from *both* the global headline and the per-group rows because they
aren't real data. The convention `pew-insights` uses is: anything
that affects the *truth* of the underlying population (data quality
filters) truncates; anything that affects the *view* of a ranked
report (sparsity floors, cv thresholds, top-N caps) is display-only.

The line between data-quality and view-preference is sometimes
fuzzy. The `--min-tokens` floor in `prompt-size` and `output-size`
is on the boundary: a group with 12 tokens total over a year is
arguably noise rather than signal, but it's also arguably a real
(very small) workload. The current convention treats `--min-tokens`
as display-only, consistent with the other floors. An operator who
wants to actually exclude tiny groups from the global denominators
has to do it upstream, by filtering the input before passing it to
`pew-insights`. That is a deliberate split: the tool doesn't decide
what counts as data, it decides how to display the data it's given.

## Closing

The display-vs-truncation distinction is one of those design choices
that costs nothing if you make it on day one and is expensive to
retrofit if you don't. Six refinement flags shipped in five days,
across five different subcommands, and every one of them respects
the same rule: globals are computed over the full population,
display floors hide rows from the ranked output, drop counts
surface what was hidden, and the headline row anchors the operator's
trust by staying numerically stable across flag changes.

The pattern generalizes well beyond `pew-insights`. Any metrics
surface — Grafana dashboards, observability CLIs, internal
analytics tools — benefits from the same discipline. Decide
explicitly which filters affect the truth and which affect only the
view. Surface the drop counts so the operator can see the cost of
each filter. Anchor every report to a stable headline row computed
against the unfiltered population. And when a flag genuinely
changes the *meaning* of a metric (the `--peak-window` case),
declare that loudly in the header — never silently, never as if it
were a display tweak.

The reason this matters is not statistical purity. It is operator
trust. A metrics tool whose headlines drift when knobs move is a
tool whose numbers will be quietly distrusted within a quarter.
A metrics tool whose headlines are bolted to the truth and whose
filters are visibly view-only is a tool that operators will reach
for first when something looks off in production. The work of
keeping that contract is small per-flag and large in aggregate;
six flags in, the discipline is starting to look like a moat.
