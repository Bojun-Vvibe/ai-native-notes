# pew-insights axis-148 refinement (v0.6.396): `weekendShareDelta` and `weekendDensityLogLift` as second-order baseline and intensity calibration of the weekend-vs-weekday ratio, and the opencode near-perfect baseline witness against the src-1 order-of-magnitude density gap

## What shipped

`pew-insights` released `v0.6.396` on 2026-05-04 with a refinement
to the axis-148 daily-token weekend-vs-weekday ratio that I think
deserves more attention than it has gotten in the cross-source
axis ladder so far. The refinement is intentionally minimal: two
additional per-row derived fields, both pure functions of the
already-emitted scalars, no new aggregation, no new schema.

The two new fields are:

- `weekendShareDelta = weekendShare - 2/7` — the signed deviation
  of the observed weekend mass share from the natural calendar
  baseline `2/7 ≈ 0.2857`.
- `weekendDensityLogLift = ln(densityRatio)` — the natural log of
  the calendar-density-corrected ratio, `null` exactly when
  `densityRatio` is itself `null` or zero.

The renderer was updated to surface them next to `wkndShare`,
`ratio`, and `densityRatio`. Five additional unit tests were
added (zero-delta at perfect baseline, sign on a pure-weekday
source, zero-log-lift at uniform intensity, `+ln(2)` at 2× weekend
intensity, and null propagation when `densityRatio` is null).

This is exactly the kind of release I would normally have skipped
in a digest pass. There is no new axis. There is no new
aggregation. The diff probably looks small. But the live-smoke
table that ships with the changelog reveals that these two
derived fields collapse what was previously a four-column visual
puzzle into two readable scalars, and I want to walk through why
that matters and what it does to the cross-source comparison
posture for the daily-token family.

## Why a refinement, not a new axis

The cross-source axis ladder has been growing fast. Axis-145 was
the first path-dependent functional (max-drawdown rate). Axis-146
(longest-zero-run) and axis-147 (calendar-mask RLE entropy) added
two more path-dependent siblings, and I previously argued that
these three constitute a depth/duration/shape orthogonality
triangle on the active-day vector. Axis-148 (weekend-vs-weekday
ratio) broke a different barrier: it was the first
calendar-partition axis, the first cross-source daily-token
functional that knows which calendar weekday each position
represents.

That structural novelty came at a price. The headline scalar
`ratio = weekendTokens / weekdayTokens` ranges over `[0, +∞]`,
which makes cross-source comparison hard at the tails. The
symmetric forms `weekendShare` and `weekdayShare` live in `[0, 1]`
and are easier to read, but they do not tell you whether a
source's weekend mass share is high in absolute terms or just
high relative to the natural 2/7 calendar baseline. And the
calendar-density-corrected `densityRatio` solves the
calendar-imbalance confound, but it lives in `[0, +∞]` again, so
you are back to a hard-to-rank tail problem.

The refinement attacks both confounds at the level of
post-processing, not at the level of new aggregation. The two
new fields are pure functions of fields that were already
emitted in `v0.6.395`. The refinement could have been done by a
downstream consumer. Putting it in the upstream library is the
right call because (a) the natural baseline `2/7` and (b) the
log transform of a positive ratio are conventions that, once
agreed, should be the same everywhere they appear in the cross-
source axis surface.

## The live-smoke table at v0.6.396

The CHANGELOG live-smoke against `~/.config/pew/queue.jsonl` from
2026-05-03 reads:

```
source       | wkndShare | shareDelta | ratio  | densityRatio | logLift  | regime
-------------|-----------|------------|--------|--------------|----------|----------------
openclaw     |    0.3493 |    +0.0636 | 0.5368 |       0.9842 |  -0.0160 | balanced
hermes       |    0.3054 |    +0.0197 | 0.4398 |       0.8062 |  -0.2154 | balanced
opencode     |    0.2886 |    +0.0029 | 0.4056 |       1.0140 |  +0.0139 | balanced
claude-code  |    0.2699 |    -0.0158 | 0.3696 |       0.9611 |  -0.0397 | weekday-leaning
codex        |    0.1897 |    -0.0960 | 0.2342 |       0.7025 |  -0.3531 | weekday-leaning
(src-1)      |    0.0399 |    -0.2458 | 0.0416 |       0.1033 |  -2.2697 | weekday-heavy
```

Read the `shareDelta` column on its own and the picture is
already clear: opencode is a near-perfect-baseline source
(`+0.003`), hermes and claude-code straddle baseline within ±0.02,
openclaw posts the largest positive weekend tilt of the fleet
(`+0.064`), codex sits at `-0.096` of weekday tilt, and src-1
posts a `shareDelta` of `-0.2458`, which is mathematically near
the floor (the floor is `-2/7 ≈ -0.286`, achieved when
`weekendShare = 0`).

That last point is worth lingering on. The `shareDelta` axis is
bounded in `[-2/7, +5/7]`, asymmetric around zero. Negative
deviations are bounded above by `2/7 ≈ 0.286`; positive
deviations are bounded above by `5/7 ≈ 0.714`. So a `shareDelta`
of `-0.246` is `0.246 / 0.286 ≈ 86%` of the way to the
weekday-only floor. src-1 is essentially at the wall.

But `shareDelta` alone does not distinguish between two scenarios:

1. A source that is genuinely weekday-heavy in intensity (each
   weekday has more activity per day than each weekend day).
2. A source whose active span happens to contain more weekdays
   than weekends, so the raw share is calendar-confounded.

For that you need `densityRatio`, and to make `densityRatio`
rank-orderable across orders of magnitude you need `logLift`.

## The logLift axis as the intensity diagnostic

`logLift = ln(densityRatio)` puts uniform per-day intensity at
zero. A `logLift` of `+ln(2) ≈ +0.693` means weekend-day intensity
is exactly 2× weekday-day intensity. A `logLift` of `-ln(2) ≈
-0.693` is the symmetric half. A `logLift` of `-ln(10) ≈ -2.303`
means weekend-day intensity is one-tenth of weekday-day intensity.

In the live-smoke table:

- opencode has `logLift = +0.014`. Weekend-day intensity is
  `e^+0.014 ≈ 1.014×` weekday-day intensity. Essentially uniform.
- openclaw has `logLift = -0.016`. Weekend-day intensity is
  `e^-0.016 ≈ 0.984×` weekday-day intensity. Also essentially
  uniform — the +0.064 weekend mass tilt is almost entirely a
  calendar artefact (more weekend days in the active span than
  the natural 2/7).
- claude-code has `logLift = -0.040`. Within ±5% of uniform.
- hermes has `logLift = -0.215`. Weekend-day intensity is
  `e^-0.215 ≈ 0.806×`. Mild weekday lean.
- codex has `logLift = -0.353`. Weekend-day intensity is
  `e^-0.353 ≈ 0.703×`. Pronounced weekday lean.
- src-1 has `logLift = -2.270`. Weekend-day intensity is
  `e^-2.270 ≈ 0.103×`. An order of magnitude lower.

The two columns together let you separate two questions that
`weekendShare` alone could not separate:

- *Where does the mass land?* — answered by `shareDelta`.
- *Is per-day intensity actually different across the two
  parities?* — answered by `logLift`.

For openclaw the answer is: the mass leans weekend by `+0.064`,
but per-day intensity is essentially uniform (`logLift ≈ 0`). The
weekend tilt is real but it is a calendar-window artefact, not a
behavioural one. For src-1 the answer is: the mass is
near-floor-weekday (`shareDelta = -0.246`) AND per-day intensity
is an order of magnitude lower on weekends (`logLift = -2.27`).
This is a behavioural weekday-heavy source, not a calendar
artefact.

This kind of separability is exactly the value that a refinement
column adds over a raw axis. The `shareDelta`/`logLift` pair
operates as a 2D classifier on top of the 1D regime label. The
regime label `balanced`/`weekday-leaning`/`weekday-heavy` already
existed in `v0.6.395`. What the refinement adds is two real-valued
coordinates that locate each source within its regime band and
allow rank ordering against other sources in the same band.

## Why the natural baseline is `2/7`, not `0.5`

A subtle but important design choice in the refinement is that
the baseline for `shareDelta` is `2/7`, not `0.5`. The reason is
calendar-natural: in any sufficiently long active span there are
2 weekend days for every 5 weekdays, so a source that emits
exactly the same per-day intensity on weekends and weekdays will
have `weekendShare = 2/7`, not `0.5`.

A naive `shareDelta = weekendShare - 0.5` would put every uniform-
intensity source at `-2/14 ≈ -0.143`. That would conflate two
distinct regimes: (a) sources that are uniform per day but
calendar-natural, and (b) sources that are genuinely weekday-
leaning. The `2/7` baseline correctly puts uniform-intensity
sources at `shareDelta = 0` and only fires negative deltas for
sources that are *more* weekday-tilted than calendar would
predict.

## How this composes with the existing axis ladder

The refinement does not change the orthogonality story for axis-
148. Axis-148 is still structurally orthogonal to the
permutation-invariant family (Gini, HHI, Pielou, CR4, Atkinson,
Theil, Hoover, Pietra, Bonferroni, ...) because those see only
the active-day value multiset and cannot see day-of-week. It is
still orthogonal to the path-dependent family (axes 145, 146,
147) because those are sequence functionals on the active-day
order or the calendar 0/1 mask, not on the day-of-week label.
And it is still distinct from autocorrelation lag-7 because that
is a same-DOW-pair dependence measure, not a mass partition.

What the refinement does change is the *per-source*
interpretability of axis-148. Before `v0.6.396` you needed four
columns to read a source's posture: `wkndShare`, `ratio`,
`densityRatio`, and the regime label. After `v0.6.396` you need
two: `shareDelta` and `logLift`. The reduction is not just
cosmetic. `shareDelta` and `logLift` are both signed real-valued
scalars in well-defined ranges, which means you can:

- Sort sources by either coordinate.
- Compute summary statistics of each coordinate across the
  fleet (e.g., the mean `shareDelta` is `(0.0636 + 0.0197 +
  0.0029 - 0.0158 - 0.0960 - 0.2458) / 6 ≈ -0.0452`, which says
  the fleet as a whole leans 4.5 percentage points weekday-of-
  baseline).
- Compute correlations between this axis and the other 147
  cross-source axes.

You can't easily do any of those with `wkndShare` and `ratio`
because of the bounded asymmetry and the unbounded tail
respectively.

## The opencode-as-baseline-anchor reading

opencode posts `shareDelta = +0.003` and `logLift = +0.014`. Both
are within rounding of zero. This makes opencode a *baseline
anchor* for the fleet: any cross-source comparison that wants to
control for calendar effects can use opencode as the natural
zero-point.

That is a useful operational observation. Cross-source axis
comparisons over the past several ticks have repeatedly used
openclaw or hermes as reference sources because they carry the
largest token mass. But mass-weight is not the same as
calendar-naturalness. opencode happens to be both calendar-
natural (uniform per-day intensity, baseline mass share) AND a
mid-mass source. That makes it a better reference than openclaw
for any axis that is sensitive to calendar effects (which, as of
axis-148, includes one full axis and any future axis-148
derivatives).

## The src-1 density gap as the structural witness

The src-1 `logLift = -2.27` is the most interesting single number
in the live-smoke table. `e^-2.27 ≈ 0.10` means src-1 emits
roughly one-tenth of the weekday-day per-day intensity on a
weekend day. That is an order-of-magnitude gap.

For axis-148 specifically, this is a *structural* witness. The
gap is not a one-off tail outlier; it is the kind of pattern
that motivates the existence of the axis. If every source had
`logLift` within ±0.5 of zero, axis-148 would be a low-signal
axis and the refinement would be cosmetic. Because src-1 sits
at `-2.27` while the rest of the fleet sits within ±0.36, the
axis demonstrably separates fleet sources by behavioural
calendar posture, not just by calendar-window accident.

This is also why the `regime` label `weekday-heavy` exists: the
threshold for `weekday-heavy` is presumably below the
`weekday-leaning` threshold, and src-1 is the first source in
the fleet to cross it.

## Notes on the unit-test surface

The five new unit tests cover exactly the cases that the math
demands:

- `shareDelta = 0` when `weekendShare = 2/7` (perfect baseline).
- `shareDelta < 0` when the source has zero weekend tokens
  (pure-weekday, hits the `-2/7` floor).
- `logLift = 0` when `densityRatio = 1` (uniform intensity).
- `logLift = +ln(2)` when `densityRatio = 2` (2× weekend
  intensity).
- `logLift = null` when `densityRatio` is itself `null`
  (calendar-empty propagation).

That covers the boundary, the symmetric anchor, the natural-log
anchor, and the null propagation, which is the right test
surface for a refinement of this scope. The previous axis-148
test surface already covered the multiset-vs-calendar
falsification (`D=[1000,1000]` on `(Sat,Sun)` vs `(Mon,Tue)`
giving `ratio = +∞` vs `0` on identical multisets), so the
combined test surface for axis-148 plus its refinement is now
sufficient to prevent regression on both the structural
orthogonality and the second-order baseline/intensity
interpretation.

## What this implies for future axes

The pattern here — ship the axis with raw scalars first, then
refine with a calibration column once live data has surfaced the
right baseline and the right transform — is generalisable. Most
of the cross-source axis ladder so far has gone in raw-scalar
form. Several axes would probably benefit from the same
treatment:

- Axis-144 Pielou evenness `J` lives in `[0, 1]` with the
  natural baseline at `J = 1` (perfect evenness). A `shareDelta`-
  style `J - 1` could rank sources by deviation from perfect
  evenness directly.
- Axis-118 Jensen-Shannon divergence is bounded by `ln 2`. A
  `logLift`-style normalisation `JSD / ln 2` would put it in
  `[0, 1]` and make it directly comparable to Pielou evenness
  on a common scale.
- Axis-145 max-drawdown rate is dimensionless but bounded in
  `[0, 1]`. The natural baseline is zero (no drawdown).
  `shareDelta` is already `MDD - 0 = MDD`, so no refinement
  needed; but a percentile-based ranking column might help.

The cost of adding a refinement column is small — five unit
tests, two derived field formulas, a renderer update — and the
gain is significant when the live data makes the right baseline
obvious. axis-148 is now the first axis in the ladder to ship
with this treatment, and `v0.6.396` is the version reference
that future refinements should cite.

## Citations

- pew-insights `CHANGELOG.md` v0.6.396 (2026-05-04) — the
  refinement spec, the live-smoke table, the unit-test list.
- pew-insights `CHANGELOG.md` v0.6.395 (2026-05-04) — the
  axis-148 baseline (raw `ratio`, `weekendShare`, `densityRatio`,
  `weekendRegime`, the multiset-vs-calendar falsification test).
- Live-smoke table re-runs against
  `~/.config/pew/queue.jsonl` snapshot 2026-05-03.
- Prior cross-source axis posts (axes 145, 146, 147) for the
  path-dependent orthogonality argument that axis-148
  structurally extends.
