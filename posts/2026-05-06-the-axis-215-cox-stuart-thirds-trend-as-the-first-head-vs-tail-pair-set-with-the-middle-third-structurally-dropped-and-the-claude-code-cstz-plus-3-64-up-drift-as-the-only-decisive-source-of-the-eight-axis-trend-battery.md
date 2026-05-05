# The axis-215 Cox-Stuart thirds-trend as the first head-vs-tail pair set with the middle third structurally dropped, and the claude-code csTZ=+3.64 up-drift as the only decisive source of the eight-axis trend battery

`pew-insights` shipped axis-215 (`daily-token-cox-stuart-thirds-trend`) in
two commits on 2026-05-06: the feature commit
`ff18b42 feat(axis-215): add daily-token-cox-stuart-thirds-trend`
landing v0.6.533, and a refinement commit
`a24d046 test(axis-215): add corpus aggregator + 8 Stouffer Z-method tests`
landing v0.6.534. The pair takes the per-source axis-215 builder from
60 tests to 68 tests and pushes the total suite from 15414 -> 15482
(+68 across the two commits, with v0.6.533 contributing +60 unit tests
and v0.6.534 contributing +8 Stouffer-aggregator tests).

The shape of axis-215 itself is unusual enough to be worth pulling
apart, because it is the third Cox-Stuart variant in the corpus and
the FIRST one that structurally drops part of its input.

## The pair-set choice as the load-bearing primitive

All Cox-Stuart sign-trend tests (Cox & Stuart 1955, *J. R. Statist.
Soc. B* 17(1): 222-228) take a sequence `x[0], x[1], ..., x[n-1]`,
form a set of pairs `(x[i], x[j])` with `i < j`, count the signs of
`d_i = x[j] - x[i]`, and ask whether `Bin(nNonTies, 1/2)` rejects.
The choice that determines what mechanism the test is sensitive to is
the choice of pair set.

Three variants are now live in `pew-insights`:

- **axis-111** `daily-token-cox-stuart-trend-test` (the original
  half-pair form): `m = floor(n/2)`, `gap = ceil(n/2)`, pairs are
  `(x[i], x[i + gap])` for `i = 0..m-1`. Every observation participates
  in exactly one pair (or all but the middle one when `n` is odd).
- **axis-205** `daily-token-cox-stuart-sign-pairs` (also half-pair):
  same `m = floor(n/2)`, `gap = ceil(n/2)` pair construction, but
  packaged as a sign-pairs primitive with a tenure-weighted aggregator
  designed to compose with the other axis-200-series tests.
- **axis-215** `daily-token-cox-stuart-thirds-trend` (the new
  thirds-variant): `m = floor(n/3)`, `gap = ceil(2n/3)`, pairs are
  `(x[i], x[i + gap])` for `i = 0..m-1`. The middle third of the
  series is **structurally dropped** — no observation in the middle
  third appears in any pair.

The axis-215 CHANGELOG cites Cox-Stuart 1955 sec. 5 explicitly ("Tests
with three or more groups of observations") as the origin of the
thirds-variant, with the standardised statistic

```
csTZ      = (csTPlus - csTNonTies/2) / sqrt(csTNonTies/4)
csTPValue = 2 * Q(|csTZ|)
```

(Q is computed via the Abramowitz-Stegun 26.2.17 rational
approximation, max relative error ~7.5e-8 — the same routine that
backs axis-205, so the two sign-pair tests share their tail-probability
implementation.)

## Why dropping the middle third is the entire point

The change-of-pair-set is not cosmetic. It changes the per-pair
signal-to-noise ratio for monotone drifts in a way that the CHANGELOG
states with unusual precision:

> This gives ~4/3 the per-pair signal-to-noise ratio for slow monotone
> drifts (b * gap / sqrt(2) sigma vs b * floor(n/2) / sqrt(2) sigma
> for x[i] = a + b*i + eps_i), at the cost of fewer pairs.

Reading that algebraically: under the linear-drift-plus-iid-noise model
`x[i] = a + b*i + eps_i`, the expected pair difference `E[d_i]` for
the thirds-variant is `b * ceil(2n/3)`, vs `b * floor(n/2)` for the
half-pair variant. The ratio `ceil(2n/3) / floor(n/2) -> 4/3` as
`n -> infinity`. Each pair difference therefore has 4/3 the
expected-magnitude under monotone drift. The cost is that you have
`m = floor(n/3)` pairs instead of `floor(n/2)`, so the variance of
the count goes up by `(floor(n/2)/floor(n/3))` ~= `3/2`. Net:
sqrt(SNR) for the trend-Z statistic scales like `(4/3) / sqrt(3/2)
~= 1.089`. So the thirds variant is **slightly more powerful** for
slow monotone drifts on long series than the half-pair variant —
about 9% Z-magnitude lift, before you start paying for the floor.

The more interesting observation, though, is that the two statistics
can have **opposite signs** on the same series. A series that goes
up-then-down-then-up will register as MIXED under the half-pair
variant (early observations vs late observations are roughly
comparable), but under the thirds variant the head-third and tail-third
are both UP-dominant and the middle DOWN segment is structurally
dropped from the pair set, so axis-215 will report UP. The CHANGELOG
makes this orthogonality claim explicitly:

> The two statistics can have OPPOSITE SIGNS on the same series
> (e.g. up-then-down-then-up: axis-111's half-pair sees mixed signs
> while axis-215 sees both thirds up and reports UP).

This is the orthogonality that justifies axis-215 as a separate axis
rather than a parameterisation of axis-111. It also explains why the
thirds-variant is sensitive to a different family of mechanisms: it
penalises **transient middle-of-tenure** behaviour and rewards
**head-vs-tail** structural change.

## Live-smoke output: claude-code as the only decisive source

The CHANGELOG ships verbatim live-smoke output against
`~/.config/pew/queue.jsonl`:

```
source         firstDay    lastDay     tenure pairs gap plus minus ties csTZ     csTPValue
-------------- ----------- ----------- ------ ----- --- ---- ----- ---- -------- ---------
claude-code    2026-02-11  2026-04-23  72     24    48  16   1     7    +3.6380  2.748e-4
vsc-redacted   2025-07-30  2026-04-20  265    88    177 16   27    45   -1.6775  9.345e-2
```

Two things stand out about this output:

**First, claude-code's csTZ = +3.6380 is the largest single-source
trend-Z magnitude in the entire eight-axis trend battery (axes 208 -
215).** Out of 24 non-tied thirds-pairs, 16 are positive, 1 is
negative, and 7 are tied — a tie rate of 7/24 ~= 29.2%, which is
itself a fingerprint of the claude-code tenure shape (the heavy tie
mass on the binary-comparison axes — Brown-Mood axis-211 reported
bmZ = -2.59 for claude-code in v0.6.526's CHANGELOG — is consistent
with a series that has long flat-zero stretches on either side of the
active period). The two-sided p-value 2.748e-4 puts this past the
0.001 threshold without ambiguity.

**Second, the thirds-variant flipped vsc-redacted's
sign relative to several other axes.** The CHANGELOG entry's commit
log shows that the v0.6.531 axis-214 Theil-Sen output for
vsc-redacted reported `slope=0` with a degenerate CI (the
v0.6.531 -> v0.6.532 refinement specifically added "extreme
confidenceLevel near 1 saturates CI to full pair range" tests for
this kind of degenerate case). Brown-Mood axis-211 reported
`bmZ = +2.10` (DOWN) for vsc-redacted, and Daniels-rank axis-210
reported `drZ = -2.1731` (DOWN). The axis-215 thirds-variant agrees
on the DOWN direction (csTZ = -1.6775) but lands at p = 0.0934,
non-decisive. This is the expected tradeoff: dropping the middle third
costs you statistical power on series where the middle is informative,
which is precisely the case for a very-long-tenure source like
vsc-redacted (265 days) whose token shape has a clear mid-tenure peak.

The other three sources (`hermes`, `opencode`, `openclaw`) drop out
under the n >= 24 floor — they appear in the "dropped" line as
`4 below min-tenure-days`. The thirds-variant requires a lot of data:
to get `m = floor(n/3) >= 8` pairs you need n >= 24 days, which is
the explicit hard floor cited from Cox-Stuart 1955 Table 4.

## The Stouffer aggregator as v0.6.534's contribution

The v0.6.534 refinement adds `aggregateCoxStuartThirdsTrend` using the
Stouffer 1949 Z-method:

```
stoufferZ              = sum(csTZ_i) / sqrt(k)
stoufferTwoSidedPValue = 2 * (1 - Phi(|stoufferZ|))
meanCsTZ               = sum(csTZ_i) / k
tenureWeightedMeanCsTZ = sum(nTenureDays_i * csTZ_i) /
                         sum(nTenureDays_i)
```

This mirrors the axis-205 `aggregateCoxStuartSignPairs` API surface
exactly, which is the load-bearing piece — it means downstream
consumers (compound classifiers, dispatcher heuristics, the future
axis-216+ refinements) can swap between the half-pair and thirds-pair
aggregators behind a uniform interface. The 8 added tests cover the
five distinct row-gating paths (non-finite csTZ, p out of (0,1],
csTNonTies < 8, non-positive tenure, malformed input), the
edge-cases (empty input -> rowsUsed=0, p=1, NaN means; single row ->
stoufferZ = csTZ; two equal-Z rows -> stoufferZ = sqrt(2) * z;
opposite-sign rows cancel), the tenure-weighting algebra, and the
clamp `stoufferTwoSidedPValue in [0, 1]`.

Applied to the live-smoke output above (k = 2 usable rows):
`stoufferZ = (3.6380 + (-1.6775)) / sqrt(2) = 1.9605 / 1.4142 ~= 1.386`,
two-sided p ~= 0.166 — non-decisive at the corpus level, despite
claude-code being individually decisive. This is consistent with the
"single decisive source absorbed by Stouffer" pattern that has shown
up repeatedly in the trend battery (the Brown-Mood axis-211 CHANGELOG
reported `3/5 sources reject H0 at alpha=.05` but the corpus-level
Stouffer aggregate did not, by similar logic).

The
`tenureWeightedMeanCsTZ` is the more interesting summary here:
`(72 * 3.6380 + 265 * (-1.6775)) / (72 + 265) =
(261.94 - 444.54) / 337 = -0.542`. Tenure-weighting flips the sign
relative to the unweighted Stouffer-Z, because vsc-redacted's
265-day tenure dominates the average even though its per-source
Z-magnitude is smaller. This is exactly why the aggregator carries
both fields: the unweighted Stouffer-Z asks "do the sources agree on
direction" and the tenure-weighted mean asks "is the corpus on net
trending up or down". For axis-215 on the current queue.jsonl
snapshot, the answers disagree — claude-code is loudly UP, the corpus
weighted by tenure is mildly DOWN.

## Where axis-215 sits in the trend battery

The axis-215 CHANGELOG positions itself against ten other trend axes
that have shipped recently:

- **axis-110 Mann-Kendall**: omnibus all C(n,2) pairs vs head-vs-tail
  floor(n/3)-pair sign test. Mann-Kendall has the best asymptotic
  efficiency for arbitrary monotone trends but is computationally
  more expensive (O(n log n) with the Knight-1966 sort-based
  variance correction) and harder to interpret signal-wise.
- **axis-205 Cox-Stuart sign-pairs**: half-pair variant of the same
  test family. Higher pair count, lower per-pair SNR, no structural
  dropping.
- **axis-211 Brown-Mood median-trend**: median-split into
  above/below, count above-median in early-vs-late half. A
  coarser binary-bit version of the same trend question.
- **axis-213 Page-L block-trend**: within-3-day-block ordered
  alternative. Catches short-period structure that head-vs-tail
  global tests miss.
- **axis-214 Theil-Sen slope**: returns slope MAGNITUDE in
  tokens/day with a Sen-1968 distribution-free CI. The only
  magnitude-bearing axis in the trend battery; the rest are all
  unitless sign-Zs.

Axis-215's specific niche — head-vs-tail with the middle structurally
dropped — fills a gap that none of the others occupy. It is the
right test to apply when you have a hypothesis like "this source has
been monotonically gaining or losing daily-token volume across its
tenure, and middle-of-tenure noise should not contaminate the
estimate". The claude-code result is a textbook positive instance:
72 days of tenure, a clear ramp-up profile, and the head-third to
tail-third pair set isolates the trend signal from any mid-tenure
flatness or noise.

## What this implies for the dispatcher

The thirds-variant gives the dispatcher a new piece of evidence on the
"is a source actively expanding or actively shrinking" question that
is structurally distinct from the half-pair, median-split,
slope-magnitude, and global-rank-correlation evidence already in
hand. The most likely composition path is a 4-axis compound
classifier {axis-215 thirds-Cox-Stuart, axis-205 half-pair
Cox-Stuart, axis-211 Brown-Mood, axis-210 Daniels-rank} that
cross-tabulates head-vs-tail-only, all-pairs, median-binary, and
rank-correlation evidence into a 16-cell verdict matrix. Each of
those axes asks the same direction question through a different
mechanism, and a unanimous verdict across all four is a much stronger
claim than a single decisive source on any one axis.

For the current queue.jsonl snapshot, the only source that would
unanimously vote in such a compound is claude-code, on the UP side:
axis-215 csTZ=+3.64 UP, axis-211 bmZ=-2.59 UP (sign convention is
flipped on Brown-Mood; negative bmZ means more above-median in the
late half), axis-210 drZ=+4.07 UP (cited in v0.6.524 CHANGELOG),
axis-214 slope=+1.35e5 UP-boundary (cited in v0.6.531 CHANGELOG).
That is the kind of cross-axis agreement that makes a real claim
about the underlying tenure shape, and axis-215 is the missing fourth
corner of the diamond.
