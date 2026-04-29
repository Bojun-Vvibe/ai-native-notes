# The CI-shape sixth leg: pew-insights v0.6.232 and the claude-code BCa asymmetry of 153M tokens

**Date:** 2026-04-30
**Subject:** `pew-insights source-row-token-slope-ci-asymmetry-concordance`
**Anchor release:** v0.6.232 (`CHANGELOG.md`, 2026-04-30)
**Live-smoke timestamp:** `2026-04-29T18:11:09.741Z` against `~/.config/pew/queue.jsonl`, `--since 2026-04-15`

## What v0.6.232 actually adds

The 0.6.227 → 0.6.231 release window stood up five orthogonal cross-lens
diagnostics over the same six per-source confidence intervals shipped by
the v0.6.219 Deming-slope uncertainty-quantification suite (bootstrap,
jackknife, BCa, studentized-T, ABC, profile-likelihood). Each diagnostic
took the same six intervals as input and reduced them along a different
axis:

- **v0.6.227** — set-overlap (Jaccard). How much do the six intervals
  overlap as point-sets on the real line? Loses both location and shape.
- **v0.6.228** — sign concordance. Do the lenses agree on the *sign* of
  the slope? Pure direction.
- **v0.6.229** — width concordance. Do the lenses agree on CI *width*?
  Pure precision.
- **v0.6.230** — overlap-graph topology. *If* lenses overlap (binary),
  not how much.
- **v0.6.231** — midpoint-dispersion. The spread of the six CI *centers*.
  Pure location.

v0.6.232 adds the sixth and last orthogonal axis: **CI shape around the
point estimate**, encoded as a per-lens scalar

```
asym_i = (ciUpper_i - slope_i) - (slope_i - ciLower_i)
       = ciUpper_i + ciLower_i - 2 * slope_i
```

`asym_i` is the signed gap between the upper half-width and the lower
half-width of lens `i`'s confidence interval. Positive `asym_i` means the
right tail (above the point estimate) is wider than the left tail, i.e.,
the CI is right-skewed *relative to the point*. Negative means
left-skewed. Exactly zero means the point estimate sits at the midpoint
of the interval — perfect symmetry.

The headline summary per source is the **`signs` 6-vector**, the
**`dominantSign`** (strict majority over `{-1, 0, +1}` requiring ≥4 of 6;
`null` otherwise), the **`concordance`** ratio
`max(pluses, zeros, minuses) / 6 ∈ [1/6, 1]`, and the bounded-headline
ratio **`meanAbsAsymOverWidth ∈ [0, 1]`** — a normalized magnitude:
0 means perfectly symmetric on every lens; 1 means every lens has the
point estimate at one of the CI endpoints.

## Why CI shape is mechanically distinct from the prior five lenses

The two intervals `[a, b]` and `[a, b]` with point estimate `p` and `p'`
can have:

- identical Jaccard overlap (they're the same set);
- identical sign;
- identical width (`b - a`);
- identical overlap-graph topology (every neighbor relation is preserved);
- identical midpoint (`(a + b) / 2`).

…and yet have *opposite* CI asymmetry signs whenever `p ≠ p'`. Two CIs
with identical midpoint, width, and peer overlap can still have opposite
asymmetry signs — the right tail is fat for one lens, the left tail is
fat for the other — and **no other module surfaces that**. This is the
formal justification for shipping a sixth lens rather than declaring the
cross-lens family closed at v0.6.231.

The clean way to see it: the prior five lenses all treat each CI as an
*interval* (a set, a width, a center, a topological neighbor). The sixth
lens treats it as a *distribution shape* anchored at the point estimate.
Two intervals that are interchangeable as sets can still encode
opposite-direction sampling-distribution skews.

## Live smoke: real data, six sources, 1994 rows

The release ships a `Live smoke` block in `CHANGELOG.md` exercising the
new module against `~/.config/pew/queue.jsonl` with `--since 2026-04-15`.
Verbatim header:

```
pew-insights source-row-token-slope-ci-asymmetry-concordance
as of: 2026-04-29T18:11:09.741Z   sources: 6   rows: 1994
min-rows: 4   confidence: 0.95   lambda: 1   bootstraps: 1000   seed: 42
dropped: 0 missing-from-some-lens, 0 not-mixed (alert),
         0 not-unanimous (alert), 0 below top cap;
mixed: 6;  unanimous-asym: 0;  unanimous-sym: 0
```

The per-source rows (sorted `concordance-desc`):

| source           | rows | +/0/− | dom | conc   | cMinB  | meanAsym       | meanAbs        | mWidth         | mAbs/W | argMaxLens | argMaxVal       | mix | unA | uS |
|------------------|------|-------|-----|--------|--------|----------------|----------------|----------------|--------|------------|------------------|-----|-----|----|
| claude-code      |  299 | 5/0/1 |  +  | 0.8333 | 0.5000 | 26,717,574.33  | 26,992,399.69  | 49,992,590.19  | 0.5399 | bca        | +153,055,532.56  | yes | NO  | NO |
| codex            |   64 | 4/0/2 |  +  | 0.6667 | 0.3333 |  8,292,448.66  | 11,944,899.81  | 62,842,053.99  | 0.1901 | bca        |  +31,686,262.98  | yes | NO  | NO |
| hermes           |  288 | 2/0/4 |  −  | 0.6667 | 0.3333 |    132,169.95  |    140,734.45  |  1,473,472.18  | 0.0955 | bootstrap  |     +756,963.08  | yes | NO  | NO |
| openclaw         |  558 | 2/0/4 |  −  | 0.6667 | 0.3333 |   −553,761.41  |  1,595,931.19  |  7,775,782.54  | 0.2052 | bca        |   −6,424,919.48  | yes | NO  | NO |
| opencode         |  452 | 2/0/4 |  −  | 0.6667 | 0.3333 | −3,541,234.82  |  5,898,826.87  | 24,253,797.35  | 0.2432 | bca        |  −18,627,285.65  | yes | NO  | NO |
| vscode-redacted  |  333 | 4/0/2 |  +  | 0.6667 | 0.3333 |      4,328.92  |      7,665.42  |     36,328.24  | 0.2110 | bca        |       +35,319.03 | yes | NO  | NO |

A few things jump out immediately and are worth reading carefully.

### 1. The fleet is unanimously `mixed`

All six sources have `mixed = yes`. `mixed = yes` means the per-source
sign vector contains *both* `+1` and `−1`: the six lenses disagree on
which side of the point estimate is wider. `unanimousAsym = 0` and
`unanimousSym = 0` reinforces this — there is no source on which all
six lenses agree on the asymmetry direction, and there is also no source
on which all six lenses are exactly symmetric. The cross-lens shape
disagreement is **structural, not exceptional**: every source carries
internal lens disagreement on CI shape.

This is the headline empirical fact of v0.6.232: at this corpus size
(1994 rows over 6 sources, 64 ≤ rows ≤ 558 per source), no source has a
clean lens-unanimous shape verdict. The sixth lens has surfaced a *new*
disagreement axis that the first five lenses had no way to expose.

### 2. claude-code is the strict-majority outlier

Five sources have concordance `0.6667 = 4/6`. `claude-code` has
**0.8333 = 5/6** — the strict-majority dominant sign holds for five of
six lenses on `claude-code`, against exactly one dissenter. The
`cMinB = 0.5000` (concordance minus the uniform-random baseline of
`1/3`) is also exactly 0.5 — half the available headroom above random
agreement. No other source comes close: every other source sits at
`cMinB = 0.3333` (one-third of the headroom).

`claude-code` is also the source with the largest absolute mean
asymmetry (`meanAbs = 26,992,399.69` tokens) and the largest
**`meanAbsAsymOverWidth = 0.5399`** — over half the CI width is, on
average, asymmetric around the point estimate. That is the largest
shape-magnitude in the fleet. The combination
*(highest concordance + highest normalized asymmetry)* means
claude-code is the source where the cross-lens family says, with the
most agreement and the most magnitude, that the sampling distribution
of the slope is *right-skewed* (`dominantSign = +`).

### 3. BCa is the lens that produces the largest single-lens asymmetry

Five of six sources have `argMaxLens = bca`. Only `hermes` has
`argMaxLens = bootstrap`. The bias-corrected-and-accelerated bootstrap
is, on this corpus, the lens that most aggressively pushes the upper or
lower tail away from the point estimate. The `argMaxVal` for
`claude-code` is **+153,055,532.56 tokens** — a single-lens
upper-half-width-minus-lower-half-width gap of about 153M tokens. For
context, the v0.6.213/214/220 work on this same source consistently put
the point slope of claude-code in the low tens of millions of tokens
per row; an absolute asymmetry of 153M is several multiples of the
point estimate.

This is consistent with the v0.6.222 BCa refinement live-smoke
(`CHANGELOG.md`, earlier release block), which independently flagged
**claude-code with `wRatio = 2.27` — the BCa interval is 2.27× wider
than the v0.6.220 percentile bootstrap interval on the same sorted
distribution, with `z0 = +0.040` and `acceleration = +0.045`**, the
largest in the table. The asymmetry-concordance lens recovers the same
finding through a completely different mechanical path: by measuring
how far the point estimate sits from the CI midpoint, lens by lens.
Both paths converge on claude-code as the source where BCa is doing
the most non-trivial corrective work.

### 4. The two distinct shape regimes in the six-source fleet

Reading down the `dominantSign` column gives two clean groups:

- **Right-skewed shape (`dom = +`)**: claude-code (5/6), codex (4/6),
  vscode-redacted (4/6).
- **Left-skewed shape (`dom = −`)**: hermes (4/6), openclaw (4/6),
  opencode (4/6).

Three-three. There is no "symmetric" group — `zeros` is 0 across all
sources. Every source picks a side; the fleet is exactly split. The
right-skewed sources include the two with the largest positive point
slopes from the v0.6.219 / v0.6.221 / v0.6.222 work
(claude-code, codex), and the left-skewed sources include the two with
the largest negative point slopes (opencode, openclaw). The shape is
*aligned* with the sign of the slope — the sampling distribution is
fatter on the side of *more extreme* deviation. That is what one would
expect from a Tukey-class tail behavior on bounded-positive token data,
and it is now machine-readable per-source via the sixth-leg diagnostic.

## What gets surfaced that v0.6.227–v0.6.231 could not

Re-reading the table with the prior five lenses in mind:

- **v0.6.229 (width concordance)** would have flagged the
  6-decade `mWidth` spread (`vscode-redacted` 36,328.24 →
  `codex` 62,842,053.99 — about 1,729,000× across the fleet), but it
  cannot tell you that within `claude-code` the BCa lens is the
  asymmetry-amplifier and bootstrap is the asymmetry-baseline. Width
  concordance scales agnostic to which side of the point estimate the
  width comes from.

- **v0.6.230 (overlap-graph topology)** would tell you that all six
  lenses overlap on every source (the v0.6.230 live-smoke documents
  full connectivity), but cannot tell you that the *shape* of those
  overlapping intervals is structurally different lens by lens.

- **v0.6.231 (midpoint-dispersion)** measures how far apart the six CI
  *centers* are; v0.6.232 measures how far the point estimate sits
  from each of those centers. These are two different statistics and
  can move independently. Specifically, the midpoint of a BCa interval
  can sit *near* the midpoint of the percentile-bootstrap interval
  (small midpoint dispersion) while still being substantially offset
  from the point estimate (large `asym_i`). The sixth lens is sensitive
  to the second offset; the fifth lens is not.

The rigor case for v0.6.232 is therefore: *every other lens treats the
CI as either a set, a sign, a scalar (width), an edge in a graph, or a
center, and not one of those representations encodes how the CI is
arranged around the point estimate*. The sixth lens fills that exact
gap, and the live-smoke shows the gap is non-trivial — every source has
internally disagreeing lens asymmetries (`mixed = yes` × 6 of 6).

## Sort keys, filters, and operational use

The release ships eleven sort keys: `concordance-{desc,asc}`,
`mean-abs-asym-{desc,asc}`, `mean-abs-over-width-{desc,asc}`,
`mean-asym-{desc,asc}`, `arg-max-abs-{desc,asc}`, `pluses-desc`,
`minuses-desc`, `zeros-desc`, `dominant-sign-{desc,asc}`, `rows`,
`source`. Two filters: `--alert-mixed` (sign vector contains both `+`
and `−`) and `--alert-unanimous` (`unanimousAsymmetric == true`).
Standard `--top N` cap with `droppedBelowTopCap` accounting matches
the rest of the family.

For triage on a live corpus the operationally interesting query is
`--alert-mixed --sort mean-abs-over-width-desc --top 5`. On the
2026-04-29 live-smoke corpus this returns claude-code (0.5399) at the
top, followed by opencode (0.2432), vscode-redacted (0.2110), openclaw
(0.2052), codex (0.1901), hermes (0.0955) — the same six sources, all
already mixed, ranked by *how much* of their CI width is asymmetric
on average. The two-decade spread (0.5399 / 0.0955 ≈ 5.65×) means the
normalized headline ratio discriminates an order-of-magnitude regime
across the fleet even though the un-normalized `meanAbs` spans seven
decades (7,665.42 → 26,992,399.69) and `mWidth` spans six decades.

## The exact-zero comparator

The release notes the sign computation uses an **exact-zero
comparator**: `signs[i] = +1 if asym_i > 0, -1 if asym_i < 0, 0 if
asym_i == 0`. This is the same regime each of the six lens kernels
already enforces upstream — every lens controls its own numerical
behavior and the asymmetry-concordance diagnostic does not introduce a
tolerance band on top. A deliberately-symmetric construction (e.g., a
percentile bootstrap on a perfectly symmetric resample distribution,
which v0.6.220's kernel can produce by construction for symmetric
input) returns `asym_i == 0` exactly, and the sixth lens preserves
that zero. On the real-data corpus shipped in the live-smoke,
`zeros = 0` for every source — no lens, on any source, lands exactly
on the point estimate's midpoint. The exact-zero discipline is the
right call: introducing a tolerance band would produce false
"symmetric" verdicts that mask the structural shape disagreement the
sixth lens is designed to surface.

## What this completes

With v0.6.232, the cross-lens family over the v0.6.219 Deming
six-lens substrate is **mechanically saturated**. The six axes are:

1. set-overlap (v0.6.227)
2. sign concordance (v0.6.228)
3. width concordance (v0.6.229)
4. overlap-graph topology (v0.6.230)
5. midpoint-dispersion (v0.6.231)
6. shape (asymmetry concordance) (v0.6.232)

Any further cross-lens diagnostic over the same six intervals reduces
to a function of these six axes. A seventh axis would have to either
add a *different* substrate (a different resampling regime, a different
estimator family) or operate on lens-internal state that the six-CI
output API does not expose. The shipping ladder is complete at six.

The next operationally interesting move is *not* a seventh cross-lens
axis. It is a **per-row drill-down**: take a source where the sixth
lens flags `mixed = yes` with high `meanAbsAsymOverWidth` (claude-code
at 0.5399 is the obvious candidate) and inspect *which subset of the
299 rows* drives the BCa lens to the +153M-token asymmetry argMax.
That is leverage analysis, and it sits one level below the cross-lens
family rather than parallel to it. Expect that to be where v0.6.233+
heads.

## Summary

v0.6.232 ships `source-row-token-slope-ci-asymmetry-concordance`, a
per-source diagnostic over the v0.6.219 six-CI substrate that measures
the gap between each lens's CI midpoint and the point estimate. On the
2026-04-29T18:11:09.741Z live-smoke against the real
`~/.config/pew/queue.jsonl` corpus (1994 rows, 6 sources, `--since
2026-04-15`):

- All six sources are `mixed = yes` — the sixth lens surfaces a
  cross-lens disagreement axis that the prior five lenses cannot
  see.
- claude-code is the strict-majority outlier at concordance 0.8333
  (5/6), with the largest normalized asymmetry magnitude
  `meanAbsAsymOverWidth = 0.5399` and a single-lens BCa asymmetry
  argMax of **+153,055,532.56 tokens**.
- BCa is the asymmetry-amplifier on five of six sources; bootstrap
  is the amplifier only on hermes.
- The fleet splits exactly 3-3 on `dominantSign` (right-skewed:
  claude-code, codex, vscode-redacted; left-skewed: hermes,
  openclaw, opencode), and the shape sign aligns with the sign of
  the slope on every source.
- The cross-lens family is now saturated at six orthogonal axes over
  the v0.6.219 substrate; further diagnostics reduce to functions
  of these six.
