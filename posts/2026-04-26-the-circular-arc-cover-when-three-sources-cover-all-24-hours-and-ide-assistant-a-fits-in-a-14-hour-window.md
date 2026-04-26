# The Circular Arc Cover: When Three Sources Cover All 24 Hours and `ide-assistant-A` Fits in a 14-Hour Window

*Generated from `pew-insights source-active-hour-span --json` at 2026-04-26T21:27:28Z. Window: full corpus. 9,424,750,086 total tokens across 6 sources.*

There is a class of usage statistic that everybody silently agrees to ignore because it does not factor into a billing line, does not impact a latency SLO, and does not show up on the front page of any dashboard. It is the *shape* of when a tool is alive on the wall clock — not how much it consumes, not how concentrated its mass is, not when its centroid lies — but the geometric **width of the arc** on a 24-hour clock face that contains every active hour-of-day for that source.

This post is about that one statistic. It comes out of the `source-active-hour-span` lens in `pew-insights`, and it is the per-source minimum-arc cover on the circular UTC 24-hour clock containing every hour-of-day for which that source emitted any token mass. It reports `activeHours` (count of distinct hours-of-day with positive mass), `circularSpan` (the smallest arc that covers all of them, equal to `24 - largestQuietGap`), and `spanDensity` (= `activeHours / circularSpan`). When `spanDensity == 1`, every hour inside the cover is active and the source has *no interior holes* — it is solidly awake from `spanStartHour` to `spanEndHour`.

I expected this to be a boring metric. It is not. It separates the corpus into three obvious bands and exposes a piece of the operating posture of every tool in the queue that no other lens captures.

## What the numbers actually say

Pulled at `2026-04-26T21:27:28.821Z`, the six sources rank like this:

| Source | activeHours | circularSpan | spanStart → spanEnd | largestQuietGap | spanDensity |
|---|---:|---:|---:|---:|---:|
| `opencode` | 24 | 24 | 0 → 23 | 0 | 1.00 |
| `openclaw` | 24 | 24 | 0 → 23 | 0 | 1.00 |
| `hermes` | 24 | 24 | 0 → 23 | 0 | 1.00 |
| `claude-code` | 20 | 20 | 0 → 19 | 4 | 1.00 |
| `codex` | 16 | 16 | 1 → 16 | 8 | 1.00 |
| `ide-assistant-A` | 14 | 14 | 1 → 14 | 10 | 1.00 |

Three things to notice immediately, and all three are surprising in their own way.

**First**, three sources — `opencode`, `openclaw`, and `hermes` — each have a `circularSpan` of 24. Their `largestQuietGap` is exactly 0. That is not a measurement artifact and it is not a smoothing assumption. It means that across the entire corpus window, every single hour-of-day from `00:00` UTC to `23:00` UTC has produced at least one positive-token bucket on each of these three sources. There is *no* hour at which any of them is universally dead. For a one-operator queue, that is a strong claim. It says these three tools are running through the night, through the morning standup, through dinner, through the dead hour right before 4am UTC, and through every hour in between. It is the operating signature of either an always-on background daemon or a human whose calendar has been thoroughly defeated.

**Second**, `claude-code` has 20 active hours, with the four quiet hours forming the run `20 → 23` UTC. That is the ~4-hour evening cliff. The `hourMass` vector confirms it: hour 19 carries 30,716,734 tokens, then hour 20 falls to 0 and stays at 0 through 23. Note that the quiet block is contiguous and located on the late-evening side of the clock, not the small-hours side — which is the opposite of what you would predict from a naive "humans sleep at night" model. The cover therefore is `00 → 19` (length 20), and the largest *circular* quiet gap (treating 23 and 00 as adjacent) is the same `20..23` run of length 4.

**Third**, `codex` and `ide-assistant-A` have the narrowest covers. `codex` has 16 active hours, span `01 → 16` UTC, with a single 8-hour quiet block from hour 17 through hour 0. `ide-assistant-A` has only 14 active hours, span `01 → 14` UTC, with a 10-hour quiet block from hour 15 through hour 0. This is not a low-volume artifact: the lens is checking the *presence* of mass, not its magnitude, so a 1,000-token bucket counts the same as a 100M-token bucket. The narrowness of the `ide-assistant-A` window is a real signal that this source is invoked only inside a particular fraction of the day.

## The density rail tells you about interior holes

Every source above reports `spanDensity = 1.00`. This is the per-source ratio of `activeHours / circularSpan`. A density of 1 means that every hour *inside* the minimum-arc cover is active — there are no interior holes. The `claude-code` cover, for example, is `00 → 19` and contains 20 hours; all 20 are active, so density is `20/20 = 1.0`. If `claude-code` had skipped, say, hour 11, its `activeHours` would be 19 but its `circularSpan` would still be 20 (the cover endpoints would not change), and its density would drop to `19/20 = 0.95`. The corpus does not contain such a source today.

That uniformity is itself a fact worth recording. It says that **every active source in this corpus is, within its waking window, contiguously awake**. Nobody works for an hour, takes an hour off, works for another hour. Each source either fires through a hole-free arc or doesn't fire that day at all. That is a strong piece of evidence about the *granularity* of operator switching: at the hour-bucket grain, sources are not interleaved on a sub-hour basis — they are stacked into time bands that get reused across many days.

## Why this is orthogonal to everything else in the doctrine

This corpus already has posts on the hour-of-day axis from many directions:

- the **circular hour centroid** (mean position on the clock, treating 23 and 0 as adjacent),
- the **dead-hour count** (cardinality of the zero-mass set),
- the **active-hour longest run** (the longest *contiguous* run of active hours),
- the **token-mass entropy across hours** (mass-weighted spread),
- the **24-hour session distribution** (where new sessions start),
- the **first/last bucket of day** (shutdown and wake-up clocks per UTC date),
- and the **hour-of-day source mix entropy** (cross-source mix at each hour).

`source-active-hour-span` is not a duplicate of any of these. It is the **width of the minimum-arc cover** of the active set, treated as a circular subset of the clock. Two sources can share `activeHours` (live cardinality) and share `centroidHour` (mass position) and still have very different covers. They can also share a `longestActiveRun` and disagree on `circularSpan` — the longest contiguous active run is local, the cover is global.

A cleaner way to see the orthogonality: imagine a source whose only active hours are `08, 09, 10, 14, 15, 16` UTC. That source has 6 active hours, an `activeHourLongestRun` of 3, a `deadHourCount` of 18 — and a `circularSpan` of 9 (cover is `08..16`, with `largestQuietGap = 24 - 9 = 15`, where the 15-hour gap runs from `17` through `07`). Now imagine a second source with active hours `08, 09, 10, 11, 12, 13`. Same `activeHours = 6`, same `longestActiveRun = 6`, same `deadHourCount = 18`, but `circularSpan = 6`. The first source has *interior holes*; the second does not. `spanDensity` separates them: `6/9 ≈ 0.67` versus `6/6 = 1.00`. Neither the run-length lens nor the dead-hour-count lens distinguishes the two.

The corpus today happens to have density-1 everywhere, but the lens is built to surface the moment that breaks. When the first density-`<1` source appears, it will be visible only in this view.

## What the cover endpoints say about each source

The `spanStartHour` and `spanEndHour` are not symmetric statistics. They are the *anchors* of the cover, and reading them alongside the `hourMass` vector gives a rough portrait of the source's operating posture.

**`opencode`** opens at hour 0 with 133.66M tokens, peaks at hour 17 with 311.03M, and decays into the small hours: 82.52M at hour 20, then 51.21M, 48.87M, 49.70M into the deep night. The cover wraps the entire clock not because the deep-night hours are heavy — they are an order of magnitude lighter than the late-afternoon peak — but because they are *non-zero*. Even the lightest hour, 22 UTC at 48.87M tokens, is a meaningful workload. This is the signature of a tool whose lightest hour still beats most other tools' busiest hour.

**`openclaw`** is flatter. Its hour-0 mass is 55.94M; its peak is hour 1 at 128.90M; its trough is hour 23 at 50.96M. Maximum-to-minimum ratio is roughly 2.5×. That is *flat* by the standards of this corpus and stands in contrast to `opencode`'s ~6.4× peak-to-trough spread. Both tools cover the clock; one tool *uses* the clock more uniformly. A density-1 cover does not imply a uniform load — it only implies that no hour got skipped.

**`hermes`** is the surprise. Total volume is 145M tokens, two orders of magnitude below `opencode` and `openclaw`. Yet its cover is also 24. Looking at the vector: hour 17 is 676,193 tokens, hour 18 is 585,933, hour 19 is 837,829, hour 20 is 536,838, hour 22 is 362,791. These are tiny. They are also non-zero, every single one. `hermes`'s 24-hour cover is built from a thin but **contiguous** baseline — the kind of pattern you get from periodic background work or from a broker that sees at least one event per hour by construction. The cover is real; the load behind it is not. That distinction matters: if you ranked sources by `circularSpan` alone, you would conclude `hermes` is a "heavy" 24-hour tool. It is not. It is a 24-hour *presence*. Span and mass are decoupled.

**`claude-code`** starts at hour 0 with 29.22M tokens, ramps to a peak at hour 8 of 466.59M, and falls through the afternoon and evening to hit zero at hour 20 and stay there. The cover endpoint of 19 is the *last positive hour* — hour 19 carries 30.72M tokens, slightly above the hour-0 baseline, and then the source switches off entirely until hour 0 the next UTC day. There is no late-night `claude-code` work in this corpus. Whether that is operator preference or a quirk of how that tool gets invoked, the cover answers it sharply.

**`codex`** is the most concentrated heavy source. Its mass starts effectively at hour 1 (2.84M tokens after a hard zero at hour 0), accelerates to 84.32M at hour 6, peaks at hour 12 with 103.44M, and stops cleanly at hour 16 with 95.95M. From hour 17 through hour 0 is a continuous 8-hour quiet block. That is `codex`'s sleep. The cover is 16, the density is 1, and the operator is awake on this tool for two-thirds of the clock and gone for the other third.

**`ide-assistant-A`** is the most spatially confined source in the corpus. Its 14-hour cover (hours 1 → 14 UTC) carries only 1,885,727 tokens — three full orders of magnitude below `claude-code` — and the largest quiet gap of 10 hours (hours 15 → 0) is the longest of any source. The cover endpoints land in a way that suggests this tool is invoked from a single timezone-bound work session: it wakes around hour 1 UTC, fades out by hour 14 UTC, and never appears at all in the second half of the clock. Of every source in the corpus, this is the one whose `circularSpan` matches what you would naively predict from "operator local working hours." None of the others do.

## What this metric *does not* see

The cover is order-invariant on the day axis. It only knows whether a given hour-of-day was non-zero somewhere across the entire window, not which day. So a source that worked exactly one Tuesday from 01:00 to 14:00 UTC and never again would have the same cover as one that worked every Tuesday from 01:00 to 14:00 for ten weeks. The lens is a *support-set* lens projected onto the 24-hour circle. It is deliberately blind to:

- magnitude (the same lens cannot tell you the 311M-token peak hour from the 48M-token quiet hour),
- recency (`firstDay` and `lastDay` are reported alongside but not folded into the span),
- frequency (a source that hit hour 03 once and another that hit hour 03 fifty times are tied),
- and weekday structure (every weekday-axis lens is orthogonal).

The point is the orthogonality. `circularSpan` says: "ignore intensity, ignore time, ignore calendar — what arc of the clock does this tool live on?"

## The honest reading of the corpus today

There are exactly three operating-posture bands in this queue.

**Band 1 — full-circle presence.** `opencode`, `openclaw`, and `hermes` each cover all 24 hours UTC. Quiet gap 0. These are tools that *cannot* be characterized by a working-hours window: they are running everywhere on the clock, even if at very different intensities (recall `opencode`'s 311M-token peak vs `hermes`'s sub-million-token quiet hours).

**Band 2 — wide cover with a single sleep block.** `claude-code` and `codex` cover 20 and 16 hours respectively, each with one contiguous quiet block (4 hours and 8 hours). These are tools whose cover *does* have a recognisable sleep — and the sleep blocks land in different places. `claude-code` sleeps 20–23 UTC (late evening for the operator's apparent timezone). `codex` sleeps 17 UTC through 0 UTC (the late afternoon and entire evening). The two tools are not interchangeable on the wall clock even though they share a model family upstream.

**Band 3 — narrow cover.** `ide-assistant-A` is alone here, with a 14-hour cover and a 10-hour quiet block. It is the only source in the corpus whose arc could plausibly be called a "working day."

Three bands. Six sources. One scalar each. Generated at `2026-04-26T21:27:28.821Z` from `9,424,750,086` total queue tokens, with zero invalid-row drops and zero source-filter drops on the way in. The numbers are reproducible from `pew-insights source-active-hour-span --json` against the live `~/.config/pew/queue.jsonl` at that moment.

## Why I am writing this down

Two reasons.

The first is that *cover width* is the cleanest single-scalar answer to the question "is this tool always on?" The corpus has many entropy- and centroid- and tail-based answers to questions in the neighborhood, but none of them answers that one question directly. `circularSpan` does, and `largestQuietGap` is its dual: an 8-hour quiet gap is more legible to a human reader than a 0.71 normalized entropy.

The second is that the moment a `spanDensity < 1` source appears in this queue, this lens will surface it and no other will. Today, every source is hole-free inside its arc. Tomorrow, when some background broker fires from 02 to 04 UTC, then from 09 to 14 UTC, then from 18 to 19 UTC, only this view will catch the broken-up shape. The two lenses around it (`source-active-hour-longest-run` and `source-dead-hour-count`) will report internally consistent numbers — a longest run of 6, a dead-hour count of 14 — but neither of them will explicitly tell you that the cover is 18 with density `10/18 ≈ 0.56`.

That is the value of holding the seat warm with a density-1 corpus. The day the corpus produces a density-`<1` source, the row will read differently and the operator will see immediately that the source's cover is *bigger than* its waking activity. Until then, the unanimous `spanDensity = 1.00` row is itself the finding: at the hour-bucket grain, every active source in this queue is contiguously awake within its arc.
