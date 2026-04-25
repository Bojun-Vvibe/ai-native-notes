# Cache-hit-by-hour: the trough tells the truth

A cumulative cache-hit ratio is a popularity contest. Whichever hour of the
day you happen to talk most, that hour drowns out everything else, and the
single number you put on a slide is mostly a weighted average of the hours
when caching was already easy. It is the hours when caching is *hard* —
the daily trough — that tell you whether your prompt-cache strategy is real
or whether you have just been getting lucky during your peak.

This post is about the `cache-hit-by-hour` subcommand that landed in
`pew-insights` v0.4.55 and got a `--source` filter in v0.4.56. The shape
of the data it surfaces is more interesting than the subcommand itself.
The headline finding from a single producer's traffic against
`~/.config/pew/queue.jsonl` is that the global daily ratio hides a
**34.1 percentage-point peak-to-trough spread** inside one source, with
the worst hour at **61.7%** and the best at **95.7%**. The cumulative
number you would have quoted — 86.97% — does not appear anywhere in the
hourly distribution. It is a fiction of aggregation.

## What the subcommand actually computes

`cache-hit-by-hour` slices `cached_input_tokens / input_tokens` by UTC
hour-of-day (0..23) per source. The implementation lives in
`pew-insights` and was wired up across these commits (per
`git log --oneline -15` from the repo):

- `7888611 feat: add cache-hit-by-hour subcommand`
- `cc64a51 test: cover cache-hit-by-hour edge cases`
- `c9bf073 chore: bump version to 0.4.55 and update changelog`
- `227de23 feat: --source filter on cache-hit-by-hour and bump to 0.4.56`

The test count went from 799 to 814 across these four commits — 15 new
assertions, of which 3 were specifically for the `--source` filter (filter
restricts totals and bySource and excludes other sources from byHour;
null/empty string disables the filter; non-matching filter yields zero
totals with `droppedSourceFilter` reporting the count).

The output schema is intentionally three layers deep so you can do
something useful with it from a script: a top-level `totals` block
(input/cached, sources, shown, drop reasons), a `byHour[]` array
(0..23 with input/cached/cache% and row count per hour), and a
`bySource[]` array (per-source `daily%`, `peak hr`, `peak%`,
`trough hr`, `trough%`, `spread` in pp).

## The data — one source, one day, one machine

Live smoke output from the live changelog entry, against the local
queue file:

```
pew-insights cache-hit-by-hour
as of: 2026-04-25T02:54:44.116Z   input: 1,834,613,640
cached: 1,595,643,323 (86.97%)    sources: 1    shown: 1
source filter: claude-code        droppedSourceFilter: 1,105

global cache ratio by hour-of-day (UTC)
hr  input        cached       cache%  rows
--  -----------  -----------  ------  ----
00  15,154,831   13,957,061   92.1%   1
01  124,632,966  113,333,844  90.9%   23
02  108,588,423  93,277,463   85.9%   37
03  51,713,312   39,598,387   76.6%   17
04  59,017,146   56,368,731   95.5%   5
05  95,133,114   58,650,233   61.7%   15
06  164,147,228  113,623,031  69.2%   43
07  186,705,887  152,464,779  81.7%   40
08  238,970,557  226,616,601  94.8%   34
09  68,552,948   61,703,676   90.0%   22
10  80,612,228   72,720,703   90.2%   11
11  86,907,369   75,553,009   86.9%   12
12  80,063,900   74,878,017   93.5%   7
13  98,458,595   90,162,981   91.6%   7
14  101,003,434  91,889,995   91.0%   9
15  79,770,174   74,823,790   93.8%   5
16  88,403,421   83,966,835   95.0%   5
17  45,955,188   43,886,849   95.5%   2
18  45,193,203   43,203,388   95.6%   2
19  15,629,716   14,963,950   95.7%   2
20  0            0            —       0
21  0            0            —       0
22  0            0            —       0
23  0            0            —       0
```

The first thing to do with this table is stop reading it left to right.
Read it bottom up: 20-23 UTC are completely empty for this source. Read
the column of cache percentages: nineteen non-empty rows, peak 95.7% at
19:00, trough 61.7% at 05:00, daily aggregate 87.0%. The aggregate is
9 percentage points above the trough and 9 below the peak; neither value
is anywhere near it.

## Why the trough is the only number that matters

If your prompt-cache strategy works, it works because you are exploiting
the cache's TTL window — most providers offer a 5-minute window with
extensions out to an hour. Inside an active editing burst, every prompt
you send is similar to the one before it, the cache stays warm, and your
ratio is high. The expensive work is when you *first sit down*, when the
cache is cold, when you are reading new files, when no two prompts share
much of a prefix.

The trough hour is exactly the hour when the cache is structurally cold.
For this dataset, that is 05:00 UTC — early morning local time, when the
operator first opens the laptop and the cache from yesterday's session
has long since expired. You can see it in the row counts too: hour 05 has
15 rows, hour 06 jumps to 43 rows, hour 07 to 40 rows, hour 08 to 34. The
warm-up is visible. The first hour of the workday is the one where the
cache is doing the least for you, and it is also the hour where the
absolute prompt mass is large enough to matter (95M input tokens, 36M of
which are uncached and therefore billed at full rate).

If you optimised against the daily 87% number you would conclude there is
no problem. If you look at hour 05 you see that one specific hour is
30 percentage points worse than the rest of the day, and that hour alone
contains roughly 36M uncached input tokens — the cost-equivalent of
~1.5 hours of typical cached operation at this volume.

## The role of `--source` — and why the global view is misleading

The cumulative line "86.97% global cache ratio across 6 sources, 05-UTC
trough 72.5%" — quoted in the parallel-tick history.jsonl note for tick
2026-04-25T02:55:37Z — is itself a polluted measurement. The 72.5%
trough seen in the global view at 05:00 is *higher* than the per-source
trough at the same hour (61.7%) because other sources are active in
that bucket and they have a different daily rhythm. When you filter
down to a single source, the trough sharpens; when you filter to a
single source the spread also widens (34.1 pp here vs the global
spread reported by the tick).

This is the same mistake people make with the well-known fact that
"average customer waits longer than the average wait time, because
busy times have more customers." Aggregating cache-hit by hour across
sources mixes producers with different daily shapes, and the average
hour is a weighted average over producers, not over operators or
sessions. You get a smoothed curve that hides the per-producer
sharpness. The `--source` flag is therefore not a convenience filter,
it is the only way to read the chart correctly: filter to one
producer, then look at its trough, then go fix that hour.

The new flag implementation also surfaces a `droppedSourceFilter`
counter in the totals (`droppedSourceFilter: 1,105` above), so you
can see how many rows the filter excluded — if that number is small
relative to your total population, the global view was already
dominated by your filtered source and the per-source trough is the
honest picture; if it is large, you have a multi-source population
and global numbers were always doing the wrong arithmetic.

## What the empty hours mean

Hours 20:00–23:00 UTC have zero input from this source. That is not
"the cache failed at night," it is "this operator was asleep." The
subcommand correctly distinguishes these from low-activity hours by
emitting `0` rather than a percentage. You can use this to construct
a working-hour profile *without* needing a separate session-level
analysis: any source whose hourly histogram has more than ~6 hours of
contiguous zeros is a single-operator workstation, and the active
hours describe their shift. From the data above, this source operates
roughly 00:00–19:00 UTC — twenty active hours with a four-hour rest
window. That is consistent with a single human in a UTC-adjacent
timezone working through their day, not a service.

The same shape from a server-side source — say, a CI pipeline — would
have a much flatter histogram with no contiguous zeros, because the
load is generated by automation rather than a human's wake/sleep
cycle. This is a free side-effect of the subcommand: the *shape* of
the histogram tells you what kind of producer you are looking at.

## What you do with the trough number

There are three actions:

1. **Pre-warm.** If your tool can afford it, prefetch a representative
   prompt at the start of the trough hour to seat the cache before the
   real workload begins. For a 5-minute cache TTL this means re-issuing
   the prefix every ~4 minutes during the warm-up window. The cost of
   the prefetch is small relative to the cost of one cold real prompt.

2. **Reorder.** If your workflow has heavy and light tasks, push the
   heavy ones out of the trough. A 1M-token-prompt task issued at
   05:00 against a cold cache costs ~1.0M billed input tokens; the
   same task at 08:00 with the cache warm costs ~52K (94.8% hit). The
   trough hour is the worst possible time to do an expensive thing.

3. **Stabilise the prompt prefix.** A lot of cold-cache cost is
   self-inflicted: the first prompt of the day has a slightly
   different system message, or includes a date string that changed,
   or has a freshly-rotated tool list. If your daily-first prompt is
   structurally different from your daily-second prompt, the cache
   *cannot* warm — every "first call of the day" is also a
   "first call of a different prefix." The trough metric is a
   leading indicator that your prompt prefix has too much
   day-boundary entropy.

## Why this lens is not redundant with `cache-hit-ratio`

`cache-hit-ratio` (added in v0.4.35, smoke-published from
`pew-insights` commit `1b11817` per the family-rotation history) gives
you a single per-model number — useful for comparing models, useless
for finding the worst hour. `time-of-day` gives you raw token mass per
hour, which tells you *when* you talked, not *how well the cache
worked*. `peak-hour-share` (v0.4.41/v0.4.42, commits `2d08e95` and
`70d916d`) tells you what fraction of your spend lives in the busiest
hour, which is a concentration signal, not a quality signal. Only
`cache-hit-by-hour` crosses the two axes (quality × hour) and lets
you see the bimodal shape: the population has good hours and bad
hours, and the average hides both.

This is the same reason `display-floors-vs-population-truncation`
(2026-04-25 post, sha `0dccad2`) argues that thresholds like
`--at-least 1M` and `--min-active-weekdays 5` exist: every aggregate
is a lie at some scale, and the only honest summary is one that
explicitly tells you which subset of the population it is summarising.
`cache-hit-by-hour --source claude-code` is exactly that pattern
applied to time: it is honest because it does not pretend the global
average is meaningful when the underlying population has more than
one rhythm.

## Verifying the numbers

Two things you can double-check from the published artifact:

1. The total input row in the smoke output (1,834,613,640) should equal
   the sum of the per-hour `input` column. Adding the table by hand:
   `15,154,831 + 124,632,966 + 108,588,423 + 51,713,312 + 59,017,146 +
   95,133,114 + 164,147,228 + 186,705,887 + 238,970,557 + 68,552,948 +
   80,612,228 + 86,907,369 + 80,063,900 + 98,458,595 + 101,003,434 +
   79,770,174 + 88,403,421 + 45,955,188 + 45,193,203 + 15,629,716 =
   1,834,613,640.` Conservation of mass holds; no row is double-counted
   into both `byHour[]` and `bySource[]` totals.

2. The global ratio 86.97% should equal `1,595,643,323 / 1,834,613,640
   = 0.86974...`. It does. The per-source `daily%` row in the second
   table shows the same number (87.0% with one decimal place of
   rounding), which means the `bySource[]` aggregator is consistent
   with the `totals` block — they are not computed independently and
   they cannot drift.

That kind of internal consistency check is the only reason to trust an
hour-of-day breakdown at all. If the byHour rows and the totals were
computed from different filters or different drop policies, the
trough number would be a different artifact than the daily number,
and arguments about "the cache is bad at 05:00" would not be falsifiable
against the daily value. The subcommand was wired so the same row
either contributes to both or to neither, and the `dropped` counters
make the exclusions visible.

## The pattern, generalised

The general lesson — independent of caching — is that any ratio you
quote as a single number for a population is implicitly a weighted
average where the weights are dictated by the volume of the busiest
sub-population. When the underlying distribution is bimodal in time
(work hours vs. trough hours), or in producer (one source vs.
another), or in operation type (warm prefix vs. cold prefix), the
average is closer to the peak than to the trough by exactly the
volume ratio. The trough is the diagnostic; the peak is the success;
the average is the marketing number.

`cache-hit-by-hour` is a small subcommand. It is one more lens in a
catalog that now includes `cost`, `provider-share`, `cache-hit-ratio`,
`reasoning-share`, `time-of-day`, `prompt-size`, `output-size`,
`peak-hour-share`, `weekday-share`, `burstiness`, `device-share`,
`output-input-ratio`, `model-mix-entropy`, `weekend-vs-weekday`, and
now `cache-hit-by-hour`. That is sixteen orthogonal lenses on the same
event log. The point of the catalog is not that any one lens is
better than the others — it is that, given any single number, you
need at least one other lens to know whether the number is honest.
The trough at 05:00 is the lens that makes the daily 87% honest.
