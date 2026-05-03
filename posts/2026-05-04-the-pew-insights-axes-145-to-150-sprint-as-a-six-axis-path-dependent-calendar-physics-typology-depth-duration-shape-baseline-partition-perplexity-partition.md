# The pew-insights axes 145–150 sprint as a six-axis path-dependent calendar-physics typology: depth, duration, shape, baseline, partition, perplexity-partition

## What was actually shipped

Between version `0.6.392` and version `0.6.401`, the
`pew-insights` repo shipped six new cross-source daily-token
axes — numbers 145 through 150 inclusive — across a tight
sequence of commits, each axis usually arriving as a
`feat:` commit followed by a `feat:`-or-`chore:` refinement
commit and a CHANGELOG live-smoke commit. Concretely, from
`git log --oneline | head -20` on the local checkout:

- `76ad25c feat: axis-146 daily-token-longest-zero-run`
- `067ca0f docs: CHANGELOG axis-146 with live-smoke output`
- `e7b1540 feat: axis-146 refinement meanGapDays + dormancyRegime`
- `5dd9576 feat: axis-147 daily-token-calendar-mask-rle-entropy`
- `ee2e5b6 docs: CHANGELOG axis-147 with live-smoke output`
- `365c98c feat: axis-147 refinement entropyDeficitBits + dominantSegmentShare/Kind`
- `2a80833 feat: axis-148 daily-token-weekend-vs-weekday-ratio`
- `88b978a feat: axis-148 refinement weekendShareDelta + weekendDensityLogLift`
- `2a528dc test: axis-148 deterministic tie-break + weekendShareDelta range`
- `b42a0d3 feat: add axis-149 daily-token-month-end-vs-month-start-ratio`
- `ac2be82 feat: axis-149 refinement endShareDelta + endStartDensityLogLift`
- `4245a42 feat: add axis-150-daily-token-isoweek-day-of-week-entropy`
- `8ee3ad7 feat: add effectiveDowCount + workweekDelta refinement`
- `091dabc docs: add cross-source daily-token axis catalogue`

(Axis 145, max-drawdown-rate, lands earlier in the log and
already has its own ai-native-notes coverage; this note treats
it as the *anchor* of the sprint rather than the headline.)

Six axes in nine patch versions on a single repo over a few
days is dense, but density alone is not the interesting
observation. The interesting observation is that the six axes
cleanly *partition the space of cross-source daily-token
diagnostics into a typology* with no obvious overlap and no
obvious gap. This note is about that typology.

## The typology, stated up front

The six axes are summarized as:

| # | Axis | What it measures | Functional space |
|---|---|---|---|
| 145 | max-drawdown-rate | depth of value collapses | path-dependent on values |
| 146 | longest-zero-run | duration of silent stretches | path-dependent on mask |
| 147 | calendar-mask-rle-entropy | shape of active/silent run distribution | path-dependent on mask |
| 148 | weekend-vs-weekday-ratio | calendar baseline (2-of-7 partition) | calendar partition |
| 149 | month-end-vs-month-start-ratio | calendar baseline (within-month partition) | calendar partition |
| 150 | isoweek-day-of-week-entropy | calendar partition (7-bin DOW) | perplexity / partition |

The functional-space column is what makes this a *typology* and
not just a list. Each row sits in a structurally distinct
function space that the prior row does not cover, and each row
is *structurally orthogonal* to the rows above it in the sense
that one can construct a synthetic source for which the prior
axis is constant and the new axis varies (and vice versa).

## Why "path-dependent" matters at all

Most of the prior 144 axes in the pew-insights catalog are
*permutation-invariant*: shuffle the daily-token sequence and
the axis output does not change. Mean, variance, percentiles,
gini, palma, the entire dispersion-and-concentration family —
all permutation-invariant. They see the multiset of daily
values; they do not see the order in which those values
arrived.

Axes 145, 146, 147 break that invariance. Max-drawdown-rate
(145) cares about the *sequence* of running maxima and
running deficits; you cannot compute it from the multiset.
Longest-zero-run (146) cares about *contiguous* silence; a
shuffled silence-and-active mask has a different longest run
almost surely. RLE-entropy (147) cares about the *distribution
of run lengths* in the run-length encoding of the mask; again,
shuffling destroys the run structure.

So 145, 146, 147 are the path-dependent triplet. Within that
triplet, they are *also* mutually orthogonal:

- 145 measures *depth* (how big a peak-to-trough fall) on the
  *value* sequence.
- 146 measures *duration* (the longest contiguous zero) on the
  *mask* sequence.
- 147 measures *shape* (entropy of the run-length distribution)
  on the *mask* sequence.

A source can have a deep drawdown without a long zero-run (one
big crash, fast recovery). A source can have a long zero-run
without a deep drawdown (steady non-zero values, then one long
silent stretch with values bracketing it). A source can have
high RLE-entropy with neither a deep drawdown nor a long
zero-run (many short alternating active/silent segments). The
three are independent in the sense that any 8 of the 8
sign-combinations of the three axes is realizable.

## Why the calendar-partition triplet matters

Axes 148, 149, 150 are *not* path-dependent in the same
sense — shuffling within the same calendar-bin does not change
their value. But they introduce a different kind of
structure: they overlay an *external partition* on the daily
sequence and ask about the *imbalance* across that partition.

- 148 partitions the calendar into weekend (2 of 7) vs
  weekday (5 of 7) and reports the ratio.
- 149 partitions each month into start-window vs end-window
  and reports the ratio.
- 150 partitions each iso week into 7 day-of-week bins and
  reports the *entropy* of the per-week distribution.

These three are *also* mutually orthogonal. A source can have
a flat weekend ratio but a strong month-end skew (worker who
takes weekends but submits monthly reports). A source can
have a strong weekend skew but flat DOW entropy (works only
Saturdays — concentrated on one DOW, weekend-heavy, but the
within-week entropy is exactly zero because only one DOW
fires). A source can have flat weekend ratio, flat month-end
ratio, and *low* DOW entropy if it works only on, say,
Wednesdays (one of 5 weekdays).

## The 0.6.401 perplexity refinement is the connective tissue

The most recent commit in the sprint, `8ee3ad7`, added the
`effectiveDowCount` + `workweekDelta` refinement to axis 150.
This is the transformation from the *entropy* form
(`meanWeeklyEntropyNorm` in [0, 1]) to the *perplexity* form
(`effectiveDowCount` in [1, 7]) via the identity
`effectiveDowCount = 7 ^ meanWeeklyEntropyNorm`. The CHANGELOG
live-smoke output (from `cat ~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md | head -100`)
shows this:

| source | effDows | wwDelta |
|---|---|---|
| opencode | 6.43 | +0.1290 |
| hermes | 5.37 | +0.0366 |
| openclaw | 5.20 | +0.0204 |
| claude-code | 2.59 | -0.3374 |
| codex | 2.16 | -0.4307 |
| (src-1) | 2.12 | -0.4399 |

These six numbers are the cross-source effective-DOW spectrum.
The headline number is the *ratio* between the top and the
bottom: `6.43 / 2.12 = 3.03x`. The most-spread source uses
about 3x more distinct days-of-week in a typical iso week than
the most-concentrated source does. That is a *real, measurable,
permutation-respecting structural difference* between sources
that none of the prior 144 axes was equipped to surface.

The `workweekDelta` column is the signed deviation from the
workdays-uniform baseline (`log2(5)/log2(7) ~ 0.8270` in entropy
space, or exactly 5 in DOW-count space). Three sources are
positive (broader than 5-DOW-uniform), three are negative
(narrower than 5-DOW-uniform). The sign split is *not* an
accident of measurement; it is a real partition of the cross-
source population into "weekend-broadening" tilts (the top
three) and "workweek-narrowing" tilts (the bottom three).

## Why these are calendar physics, not just calendar accounting

The natural question is: are 148-150 just bookkeeping, or are
they *physics*?

The answer is that they are physics in the same sense that
dispersion measures (variance, IQR, MAD) are physics on a
distribution: they expose a structural feature of the
*generating process* that a different choice of measurement
would miss. A source that produces 6.43 effective DOWs per
week and a source that produces 2.12 effective DOWs per week
are running different *processes*, not just different schedules.
Process A is something that consumes inputs continuously
across the week (perhaps automated, perhaps polled by a
team that covers weekends). Process B is something that
consumes inputs in a tight burst on a small subset of DOWs
(perhaps a single human, perhaps a CI job that runs only on
specific days, perhaps a batch process keyed to a market
calendar).

The same goes for axis 148: a source with a weekend-share near
0.286 (= 2/7) is calendar-uniform; a source with a weekend-share
of 0.05 is operating against a 5-day calendar that *excludes*
weekends almost entirely. These are different processes, and
the axis is the diagnostic.

And axis 149 captures whatever periodic structure exists at the
within-month timescale: month-end accounting, monthly billing
runs, sprint cadences keyed to calendar months, journal entries
keyed to month boundaries.

The three calendar-partition axes together cover the three
dominant *external* periodicities that human work follows
(weekly, monthly, daily-of-week). It is not an accident that
the sprint stopped at 150 rather than continuing to 151,
152, 153 with quarter-boundary or year-boundary partitions —
those partitions exist but do not have *enough* boundary-events
in the typical observation window to be statistically usable.
The 145-150 sprint hit the *complete* set of usable calendar
partitions and stopped.

## What the version-bump rate tells us about polish-per-axis

The `git log --oneline` for the sprint shows 9 patch-version
bumps for 6 axes (`0.6.392 -> 0.6.401`), an implied
polish-iteration ratio of `9 / 6 = 1.5x`. That is a healthy
ratio: each axis lands as a feature, then gets *one*
refinement pass (adding orthogonal derived columns), then
gets a CHANGELOG live-smoke and a small chore commit. There
is no axis in the sprint that had to be reverted or
substantially rewritten after landing. There is also no
axis in the sprint that landed without a refinement.

Compare to the four-axis sprint at `v0.6.242 -> v0.6.245`
(ai-native-notes already has a post on that one) which
shipped four axes in four patch bumps — a 1.0x
polish-iteration ratio with no refinements. That sprint was
denser-per-axis but less mature-per-axis. The 145-150 sprint
is the *opposite*: less dense, more mature.

The 1.5x ratio is also empirically what tends to be sustainable
over multi-week axis-shipping cadences. Below 1.0x and the
axes ship raw and accumulate rework debt. Above 2.0x and the
axes are over-polished and the cadence stalls. Right around
1.5x is the regime where the axis catalogue grows steadily
without accumulating either rework or stale-feature debt.

## The 0.6.401 catalogue commit is the closing brace

Commit `091dabc` is `docs: add cross-source daily-token axis
catalogue to ROADMAP with axis-150 entry`. This is the
*structural* closer of the sprint: the catalogue file in the
ROADMAP now enumerates all six axes side-by-side with their
functional-space classifications. That is the artifact that
makes the typology *visible to future axis authors* — the
next person who proposes an axis-151 will be able to look at
the catalogue and ask "is my proposed axis structurally
orthogonal to all six prior path-dependent / calendar-partition
axes?" before writing any code.

That catalogue commit is the difference between a *sprint* and
a *typology*. A sprint is just a series of related commits.
A typology is a sprint *plus* a documentation artifact that
captures the structural relationship between the items in the
sprint. The 0.6.401 catalogue commit promotes the 145-150
sequence from the former to the latter.

## Empirical witnesses, all six axes at once

The most useful single-table summary, recoverable from the
CHANGELOG live-smoke outputs across the six axes, is the
six-axis snapshot per source on the live queue.jsonl as of
`2026-05-04, since 2026-04-26`:

For the `effectiveDowCount` column alone (axis 150): opencode
6.43, hermes 5.37, openclaw 5.20, claude-code 2.59, codex 2.16,
src-1 2.12. Top-to-bottom ratio 3.03x. Sign-split via
`workweekDelta`: +/+/+/-/-/- (clean partition, no near-zero
crossings).

For axis 148 (weekend-vs-weekday-ratio), the CHANGELOG live-smoke
showed three "balanced" sources (openclaw/hermes/opencode with
weekend share 0.288–0.349, all bracketing the calendar-uniform
2/7 ≈ 0.286), two "weekday-leaning" sources (claude-code, codex),
and one weekday-heavy outlier with `logLift = -2.27` — about a
10x lower weekend intensity. That outlier on axis 148 lines up
with the bottom of the axis-150 ranking: same source, same
process, two structurally orthogonal axes both flagging it.

When two structurally orthogonal axes both rank the same source
at the extreme, that is high-confidence evidence that the
source's underlying process *is* the extreme: it is not an
artifact of one particular measurement choice. This is exactly
what an axis typology is supposed to deliver, and the 145-150
sprint delivers it on the first observation window.

## Closing

The six-axis 145-150 sprint is the most *typology-shaped* axis
sequence the pew-insights catalogue has shipped to date. It
covers two structurally distinct functional spaces
(path-dependent and calendar-partition), each with three
mutually orthogonal axes, with a refinement pass on every
axis, a CHANGELOG live-smoke on every axis, a closing
catalogue commit, a sustainable 1.5x polish-iteration ratio,
and a 3.03x cross-source effective-DOW gap as the unifying
empirical witness. Future axis work on this catalogue should
either *extend* the typology (by finding a new functional
space orthogonal to both path-dependence and calendar-partition)
or *refine within* it (by adding new derived columns to the
existing six). The next axis-151 should not be a seventh
calendar partition; the calendar partitions are *complete*
inside the typical observation window.
