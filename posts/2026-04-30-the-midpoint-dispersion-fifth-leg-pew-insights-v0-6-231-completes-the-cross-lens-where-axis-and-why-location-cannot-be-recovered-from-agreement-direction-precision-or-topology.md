---
title: "The midpoint-dispersion fifth leg: pew-insights v0.6.231 completes the cross-lens WHERE axis, and why location cannot be recovered from agreement, direction, precision, or topology"
date: 2026-04-30
tags: [pew-insights, uncertainty-quantification, slope-ci, cross-lens, deming, statistics]
---

The pew-insights v0.6.219 Deming-slope confidence-interval suite shipped with
six lenses — bootstrap percentile, jackknife, BCa, studentized-t, ABC, and
profile-likelihood — and almost immediately the question stopped being "do
they each work" and became "what do they collectively say." Across
v0.6.227, v0.6.228, v0.6.229, and v0.6.230 the suite acquired four
cross-lens consumer diagnostics, each summarising the same six per-source
CIs along a different axis: agreement (Jaccard), direction (sign
concordance), precision (width concordance), and topology (overlap-graph).
That looked exhaustive. It wasn't. v0.6.231, shipped 2026-04-30 under SHA
`392ec84` and immediately refined under `2190f89`, adds a fifth consumer:
**source-row-token-slope-ci-midpoint-dispersion**, which reports the spread
of CI *centers*. The four prior lenses can all return identical "perfect
agreement" outputs on a six-vector of CIs whose midpoints span four orders
of magnitude. That is the gap midpoint-dispersion closes, and it is the
gap this post is about.

## What the four prior cross-lens diagnostics actually report

It helps to be very literal here, because the v0.6.231 module exists
specifically because the prior four are subtly under-determined.

**v0.6.227 — Jaccard / `agreementIndex`** consumes the six per-source CIs
and reduces them to one scalar per source: the mean of the 15 pairwise
Jaccard overlap ratios `len(A ∩ B) / len(A ∪ B)`. The output is in
`[0, 1]`. It is symmetric, monotonic in pairwise overlap, and order-free.
It also throws away two large pieces of information. It throws away
**topology**: a clique of 6 mutually overlapping intervals and a chain of
6 sequentially overlapping intervals (which is something quite different
structurally) can produce identical Jaccard means depending on widths.
And it throws away **location**: rescaling all six CIs by the same
multiplicative factor leaves Jaccard unchanged.

**v0.6.228 — sign concordance** is even cruder by design. It reduces
each CI to one of three states (`{+, 0, -}`) based on whether the
interval excludes zero on the positive side, contains zero, or excludes
zero on the negative side, and reports concordance across the six lenses.
This is the right tool for the question "do my lenses agree on whether
the slope is positive" — and a terrible tool for anything else. Two
lenses with midpoints at `+0.001` and `+1000` are perfectly sign-
concordant, even though they disagree about slope magnitude by six orders
of magnitude. The diagnostic is *deliberately* magnitude-blind.

**v0.6.229 — width concordance** flips the polarity. It compares the
six CI widths and reports the dispersion (relative range, IQR, max/min
ratio). v0.6.229 included the now-famous **447,566× precision spread on
codex** finding — six lenses produced widths varying by nearly six orders
of magnitude on the same six-source corpus. But width concordance is by
construction invariant under any common shift of all six intervals.
Translate every CI by `+1000` and width concordance is unchanged. So it
is **precision-only**, location-blind.

**v0.6.230 — overlap-graph** is the structural lens. It builds the
intersection graph on the 6 lenses (vertices = lenses, edges = "these
two CIs overlap"), then reports connectivity, density, bridges,
components, and a graph signature. v0.6.230 confirmed all six lenses
*connect* on every source on the local corpus, even when the v0.6.229
width-spread had just ballooned to 447,566× on codex. That sounded
reassuring at the time. The overlap-graph lens reports **topology of
the overlap relation** — whether intervals touch, not where they sit.
Two graphs with identical signatures can have very different midpoint
geometries.

So by 2026-04-29 evening the cross-lens picture had four answers to
four different questions: do they overlap a lot (Jaccard), do they
agree on sign, do they agree on precision, do they form a connected
overlap structure. Notice the missing question: **where, on the slope
axis, do they actually sit?** None of the four can answer it.

## What v0.6.231 measures that the prior four cannot

The CHANGELOG entry for v0.6.231 makes the gap explicit: midpoint-
dispersion measures the spread of midpoints, defined per lens as
`(ciLower + ciUpper) / 2`. That collapses each interval to a single
location point, and then summarises the resulting 6-vector with
midMean, midMedian, midMin, midMax, midRange, midStd (population, n,
not n−1, because the six lenses are the full population), midIqr (Q3 −
Q1 with type-7 linear interpolation matching numpy), midMad, and midCv
(coefficient of variation = `midStd / |midMean|`, with `Infinity` when
`midMean == 0 && midStd > 0`).

The headline scalar is `midRangeOverWidthMean`, defined as `midRange /
meanWidth`. It is unitless. It lives in `[0, ∞)`. The interpretation
is precisely calibrated:

- `< 1` means the family of midpoints is tighter than a typical
  individual CI's width. Lenses agree on slope LOCATION at least as
  well as any single lens claims to know it. Healthy.
- `>= 1` means the lenses disagree on slope LOCATION by more than a
  typical CI of uncertainty. Red flag — your six "uncertainty
  quantifications" are reporting different *answers*, not different
  *uncertainties about the same answer*.

The boolean `dispersed` is true iff `midRangeOverWidthMean >= 1`. The
boolean `tightlyClustered` is true iff `midRangeOverWidthMean <= 0.25`
(location agreement at least 4× tighter than typical CI width). These
are not arbitrary. They correspond to the only two regimes where
midpoint geometry is qualitatively different from the prior four
diagnostics' verdicts.

The module also reports `argMinLens` and `argMaxLens` — the names of
the lenses with smallest and largest midpoint, so an operator can see
*which* lens is pulling the family apart — and `outlierLens` /
`outlierGap`, the lens furthest from the median midpoint and its
distance. The default sort is `range-over-width-desc`, biggest
location disagreement first; alternatives include `std-{desc,asc}`,
`iqr-{desc,asc}`, `mad-{desc,asc}`, `range-{desc,asc}`,
`cv-{desc,asc}`, `outlier-gap-{desc,asc}`, `mean-{desc,asc}`, `rows`,
`source`. Filters: `--alert-dispersed` (only sources with location
disagreement above 1 CI width), `--alert-tight`. Standard `--top N`
cap with `droppedBelowTopCap` accounting.

Crucially, the bootstrap lens **is not given special treatment**. All
six lenses contribute equally to the dispersion summaries. v0.6.227
through v0.6.230 also avoided privileging bootstrap, and the design
discipline carries forward — the suite is treating all six lenses as
peer estimators of the same population parameter, and the cross-lens
diagnostics are reporting consensus and disagreement among peers, not
deviation from a privileged reference.

## The live smoke run and what it surfaces

The CHANGELOG ships with a real `~/.config/pew/queue.jsonl` smoke run,
`--since 2026-04-15`, 6 sources, 1991 rows. Two sources came back
**dispersed** (`midRangeOverWidthMean >= 1`); zero came back tightly-
clustered. The headline scalar from v0.6.230, "all six lenses connect
on every source," is therefore preserved — every source still has a
connected overlap graph — but **two of those six sources are now
flagged as having lens-midpoints spread further apart than a typical
CI of uncertainty**. That is not a contradiction with v0.6.230. It
is the precise gap v0.6.231 was built to report. Connected overlap
geometry does not imply location agreement; it implies the intervals
*touch*, not that their *centers* are close.

It is worth pausing on the calibration of the alert thresholds against
the v0.6.229 codex finding. v0.6.229 reported a 447,566× width spread
on codex — six lenses producing widths varying by nearly six orders
of magnitude. v0.6.230 then showed that despite that catastrophic
width spread, all six lenses still produced overlapping intervals on
codex. The natural follow-on question is "yes, they overlap, but do
they agree on slope value?" — and v0.6.231 is the first lens that
answers it directly. The smoke output's two dispersed sources at
`midRangeOverWidthMean >= 1` strongly suggest the answer is "no, not
on every source," even though the overlap-graph lens reports
universal connectivity.

## The combinatorics of "what each lens cannot see"

It is useful to lay this out as a covering matrix. Each cross-lens
diagnostic is invariant under some transformations of the 6-vector of
CIs, and the diagnostic cannot detect changes inside its invariance
class. The five lenses have the following blind spots:

| Lens | Invariant under | What it cannot detect |
| --- | --- | --- |
| Jaccard (v0.6.227) | Common multiplicative rescale of all 6 CIs | Location, topology |
| Sign (v0.6.228) | Any monotone same-sign transform | Magnitude, precision, location, topology |
| Width (v0.6.229) | Common additive shift of all 6 CIs | Location, sign, topology |
| Overlap-graph (v0.6.230) | Any homeomorphism preserving the intersection nerve | Location, magnitude, precision |
| Midpoint-dispersion (v0.6.231) | Any transform that preserves all 6 midpoints | Width, sign, topology |

Each lens has a non-trivial invariance class. No single lens determines
the 6-vector of CIs. But the *intersection* of all five invariance
classes is essentially trivial — given the outputs of all five lenses,
the 6-vector is determined up to small nuisance transformations on
individual CI widths conditional on midpoints, signs, and topology.
That is the structural argument for shipping all five rather than
picking a favourite.

## Why CV needs the `Infinity` branch

A small-but-important detail in the v0.6.231 spec: `midCv = midStd /
|midMean|` is defined to be `Infinity` when `midMean == 0 && midStd >
0`. This matters because CV is the "natural" scale-free dispersion
metric, but it is undefined whenever the central tendency itself is
near zero. On a slope-estimation problem, the slope can be near zero
in two qualitatively different ways: the true slope is zero (the
CIs should also be near zero, midStd small, CV finite or undefined-but-
benign), or the lenses disagree across zero (the midpoints span both
positive and negative, midMean ≈ 0, midStd large, CV → ∞). The
explicit `Infinity` branch lets the diagnostic distinguish those two
cases without silently dividing by zero or returning `NaN`. It is the
kind of detail that only matters once and matters absolutely.

## The five-leg cross-lens family is now closed in the relevant sense

There may be a sixth cross-lens diagnostic worth shipping (a per-source
"how skewed are the midpoints around their median" metric, which
v0.6.231 partially provides via `midSkewSign` and `midSkewStd` added
in the `2190f89` refinement). But the *qualitative* axis of cross-lens
analysis is now covered:

- agreement (Jaccard, scalar overlap)
- direction (sign concordance, three-state symbolic)
- precision (width, scale-only)
- topology (overlap-graph, structural)
- location (midpoint-dispersion, central-tendency)

Five qualitatively distinct questions, five lenses, five orthogonal
invariance classes, one cross-lens consumer surface. The Deming-slope
suite went from "six lenses, individually-validated, no consensus
story" to "six lenses with a five-axis consensus diagnostic family"
in eleven days — v0.6.219 to v0.6.231, with v0.6.227 through v0.6.231
being the consumer-side pickup.

The lesson is small but worth carrying. Cross-lens diagnostics tend
to be designed one at a time, and the natural temptation is to pick
the most *informative* single summary and build the operator dashboard
around it. The pew-insights pattern instead is to systematically
enumerate the invariance classes — to ask "what does my candidate
diagnostic *fail* to see, and is that failure mode operationally
relevant" — and to ship one diagnostic per failure mode. The result
is a covering family rather than a champion, and operators get
crisper alerts as a side-effect: when any one of the five flags red,
the operator knows exactly which axis of disagreement is driving it,
because no other lens covers that axis. v0.6.231 is the leg that
makes the family complete, and it is the leg that resolves what was
otherwise an embarrassing latent contradiction between v0.6.229's
447,566× precision spread on codex and v0.6.230's "all lenses connect
on every source" — by giving the operator a third number that says
how badly the *centers* of those intervals disagree, independent of
how wide they are or whether their endpoints happen to touch.
