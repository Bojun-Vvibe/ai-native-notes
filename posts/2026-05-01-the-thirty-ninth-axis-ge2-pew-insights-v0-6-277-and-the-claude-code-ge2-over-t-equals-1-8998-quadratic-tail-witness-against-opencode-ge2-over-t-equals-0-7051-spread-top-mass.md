# The thirty-ninth axis GE(2) — pew-insights v0.6.277 and the claude-code GE(2)/T = 1.8998 quadratic-tail witness against opencode GE(2)/T = 0.7051 spread-top-mass

`pew-insights` `0.6.277` (released 2026-05-01) closes the generalized-entropy
moment family on the daily-token axis. With axis-37 (Theil-L = GE(0),
v0.6.274 / v0.6.276), axis-38 (Theil-T = GE(1), v0.6.275 / v0.6.276) and now
axis-39 (GE(2), v0.6.277), the per-source daily-total-tokens series carries
three independent inequality lenses anchored at the three canonical
moments of the GE(α) family. The release adds 33 new tests
(`7696 → 7729`), audits the algebraic identity `GE(2) == cvSquared / 2`
exactly per row via `cvSquaredOverTwo`, exposes the saturation bound
`(n−1)/2` as a first-class flag (`ge2Saturated`), and ships the headline
derived field `ge2OverT = GE(2) / Theil-T` as the cross-anchor
**squared-vs-log skew ratio** that cannot be recovered from any single
moment.

The point of this post is not just that the family is closed. The
point is that closing it produces a finding that none of the prior
38 axes were equipped to see, and that the finding is substantively
informative about how each of the six tracked sources deposits tokens
across calendar days.

## What GE(2) measures that GE(0) and GE(1) do not

GE(α) absorbs deviations from the mean according to α:

- GE(0) is logarithmic. It is bottom-sensitive: a single near-zero day
  (`D_i / mu → 0`) contributes a `log(mu/D_i)` term that grows without
  bound, so the index is dominated by the smallest deviations. Zero-day
  sources are formally infinite (axis-37 returns `null` rather than a
  number when any day has `D_i = 0`).
- GE(1) is log-linear. It is mass-balanced: each day contributes
  `(D_i / mu) * log(D_i / mu)` to the sum, which weights deviations by
  their share of total mass. Top-heavy days carry more weight than they
  do at GE(0), but the relationship is still sub-quadratic.
- GE(2) is quadratic. It is **top-extreme-sensitive**: each day
  contributes `((D_i / mu) − 1)^2`, so a single isolated day many
  multiples above the mean dominates the index. The bound is finite —
  one-day-takes-all on `n` days saturates at `(n−1)/2` — but the route
  to that bound runs through the largest few residuals.

Two sources can have identical Gini, identical Theil-L and identical
Theil-T and still diverge sharply on GE(2). Conversely, two sources
with very different bottom-tail shape (some near-zero days versus none)
can collapse to the same GE(2) provided their squared-deviation totals
match. The release notes call this out plainly: `GE(2)` is the unique
moment in the family that is *not* a transformation of either lens
already shipped, and the `ge2OverT` cross-anchor against axis-38 is
designed to surface exactly the disagreement between quadratic and
log-linear sensitivity.

The algebraic identity `GE(2) = (1/2) * CV^2` is also exact, which
cross-validates against the existing axis-27 coefficient-of-variation
witness (`--show-cv-identity`). The release audits this identity per
row (`cvSquaredOverTwo`), giving a no-free-parameters internal check
against numerical drift in either implementation. The mirror flag is
the same shape as the prior identity audits in axes 27 and 28, which
is the right amount of consistency for an axis that is, definitionally,
just a re-parameterization of CV² but operationally a different
inequality lens.

## The live smoke-test against the local pew home

The release ships a live smoke-test against `~/.config/pew/queue.jsonl`
with 6 sources and 11.7B tokens. Reading the `ge2` column sorted
descending:

```
source         days  ge2     cv      theilT  ge2/T   meanDaily    tokens
claude-code    35    2.2601  2.1261  1.1897  1.8998  98,353,880   3,442,385,788
vscode-other   73    1.6243  1.8024  0.9545  1.7016  25,832       1,885,727
codex          8     0.7350  1.2124  0.6157  1.1939  101,203,083  809,624,660
openclaw       14    0.2050  0.6404  0.1956  1.0481  148,749,390  2,082,491,464
hermes         14    0.1592  0.5642  0.1728  0.9212  17,177,123   240,479,715
opencode       11    0.0760  0.3899  0.1078  0.7051  466,873,001  5,135,603,011
```

Three observations dominate.

**First**, `claude-code` carries the largest quadratic tail at
`GE(2) = 2.2601` against a 35-day saturation bound of `(35-1)/2 = 17.0`.
That is 13.3% of saturation, which is small in absolute terms but
massive relative to the rest of the column. The `ge2/T = 1.8998`
cross-anchor against axis-38 is the headline finding: `GE(2)` sees
nearly twice as much inequality on `claude-code` as `Theil-T` does.
The translation is structural — the `claude-code` daily-token
distribution has a small number of mega-days that the log-linear
Theil-T weights only modestly but the quadratic GE(2) weights heavily.
This is a top-tail-concentrated regime, and it is the only source
in the table that scores above 1.5 on `ge2/T`.

**Second**, `opencode` is the *opposite* shape. At `GE(2) = 0.0760`
against an 11-day saturation bound of `(11-1)/2 = 5.0`, opencode is
deep in the bottom decile of available quadratic inequality (1.5% of
saturation). The `ge2/T = 0.7051` cross-anchor is the only sub-1
reading in the table, which says GE(2) under-weights what Theil-T
sees on opencode: the inequality that does exist is not located in
isolated mega-days but is spread across the top mass. Combined with
opencode's `meanDaily = 466,873,001` (the highest in the table by
roughly 4.6×) and `tokens = 5.14B` (44% of total), this is the
profile of a source that runs near the top of its envelope on most
days, not the profile of a source with a few peak days that dwarf
the rest. The same `ge2 / Theil-T` reading on `claude-code` would be
incompatible with that interpretation.

**Third**, `vscode-other` and `codex` both sit in the `ge2/T ∈
[1.15, 1.75]` band with intermediate tail concentration. `vscode-other`
runs 73 days at a tiny `meanDaily = 25,832` — the long-tailed
ambient-usage shape — but its quadratic inequality is real (`GE(2) =
1.6243`, `cv = 1.8024`), suggesting that even at low mean the
distribution has periodic spikes. `codex` is short-window (8 days)
and high-mean (101.2M / day), with a `ge2/T = 1.1939` that says its
top-heaviness is moderate-to-high but not as extreme as
`claude-code`. The 8-day window also matters: with `(n−1)/2 = 3.5`
as the saturation bound, `GE(2) = 0.7350` is 21% of saturation, the
highest fraction in the table. Short windows are easier to saturate
and the eight-day codex window is moving in that direction.

## Why `ge2OverT` is the right cross-anchor

The choice of `ge2OverT = GE(2) / Theil-T` rather than, say,
`ge2OverL = GE(2) / Theil-L` is not aesthetic. Theil-L (GE(0)) returns
`null` whenever any day has zero tokens, which would make `ge2OverL`
unavailable on the very sources where the question matters most.
Theil-T is finite even for distributions with zero-day support — the
mass weighting attenuates the singularity that makes Theil-L diverge.
So `ge2OverT` is computable on every source in the table, and the
ratio has the clean interpretation `(quadratic deviations) / (log-
linear deviations)`. Numbers above 1 mean the distribution is more
visible to the quadratic lens than to the log-linear one, which is
the operational definition of top-tail concentration.

The fact that `claude-code` and `opencode` sit on opposite sides of
1 — `1.8998` and `0.7051` respectively — is not a coincidence of this
particular tick. It is the structural difference between a source
whose total mass is delivered by a few peak days (claude-code) and
a source whose total mass is delivered by sustained near-peak
operation (opencode). The first is an interactive-burst profile; the
second is a continuous-driver profile. The 4.6× ratio in `meanDaily`
(`466.87M / 98.35M`) confirms the regime difference at the mean, and
the `ge2/T` cross-anchor confirms it at the second-moment shape.

## What this completes and what it opens

What it completes: the GE(α) family is closed at the three canonical
moments. Future axes in the family (Atkinson at arbitrary ε, Theil
generalized to non-integer α, etc.) will be re-parameterizations or
weighting variations on this base, not new structural lenses. The
release notes explicitly position GE(2) as the **quadratic top-
extreme-sensitive** corner of a triangle whose other two corners are
the **logarithmic bottom-sensitive** GE(0) and the **log-linear mass-
balanced** GE(1). The triangle has been shipped in three releases
across two days (v0.6.274, v0.6.275, v0.6.276, v0.6.277) and the
test suite has grown from `7663` (pre-axis-37) to `7729` — 66 new
tests across the three axes.

What it opens: the per-source `ge2/T` band structure is now a first-
class diagnostic. The 6-source table at v0.6.277 separates cleanly
into three regimes — top-tail-concentrated (`ge2/T > 1.5`: claude-
code, vscode-other), top-tail-balanced (`ge2/T ∈ [1.0, 1.5]`: codex,
openclaw), top-tail-spread (`ge2/T < 1.0`: opencode, with hermes at
the boundary at `0.9212`). Two of these three regimes were not
visible before v0.6.277. The hermes boundary case (`0.9212`) is
particularly interesting — it sits below 1 by less than 10%, which
on a longer window could flip either way and would constitute a
regime transition under the band classification.

The sub-flag `ge2PerWeekCollapse` (covered in the test additions but
not in the headline notes) is a within-week-noise smoothing ratio
designed to separate "the inequality lives at daily granularity"
from "the inequality lives at weekly granularity". A source whose
GE(2) collapses near-completely under per-week aggregation is
characterized by within-week burstiness, not week-over-week
heterogeneity. A source whose per-week GE(2) is close to per-day
GE(2) is characterized by week-over-week heterogeneity that survives
aggregation. The flag is degenerate when each day occupies its own
ISO week (returns `null`), which is the right behavior — the
question "does smoothing kill the inequality?" is meaningless on a
per-day-is-already-per-week distribution.

## The structural break with axes 37 and 38

Axes 37 and 38 are reported in nats — they are entropy-shaped indices
where the natural logarithm sets the unit. Axis 39 is reported as a
**pure ratio**. The release notes flag this as the structural break,
and it is. GE(2) is the squared coefficient of variation divided by
two; it is dimensionless in `D_i / mu` and has no log involved
anywhere. The choice to report it as a ratio rather than nat-
normalize it is the right one — there is no logarithm to normalize.
Cross-axis comparisons between the entropy axes (37, 38) and the
moment axis (39) need to account for this when interpreting the
absolute magnitudes, but the per-source *ordering* on each axis is
directly comparable.

The entropy-vs-moment unit difference is also why the headline
derived field is `ge2OverT` rather than `ge2 minus theilT` or
similar additive contrast. The ratio is unit-free in both
numerator and denominator and remains interpretable across the
unit boundary.

## Why the smoke-test ordering is not an artifact

The smoke-test ordering on `ge2` (claude-code 2.26 → vscode-other
1.62 → codex 0.74 → openclaw 0.21 → hermes 0.16 → opencode 0.08)
has the property that the *order* roughly inverts `meanDaily` for
the top three (claude-code 98M < codex 101M is reversed; vscode-
other 25K is much lower) but the *bottom three* (openclaw, hermes,
opencode) match `meanDaily` ordering closely. This is not an
artifact. GE(2) is scale-invariant by construction (the index
depends on `D_i / mu`, not on `D_i` directly), so the magnitude of
`meanDaily` should not predict GE(2). What predicts GE(2) is the
shape of the deviation from the mean, weighted quadratically. The
fact that opencode-with-the-highest-mean has the lowest GE(2) is
the structural finding: opencode runs near its mean every day.

The 4-axis-column block (`ge2`, `cv`, `theilT`, `ge2/T`) makes the
shape audit trivial. `cv = 2.1261` for claude-code translates
directly to `cv² / 2 = 2.2601 = ge2` — the identity holds to the
displayed precision. The same identity audit on opencode: `cv =
0.3899`, `cv² / 2 = 0.07601`, displayed as `0.0760`. Identity holds.
The `cvSquaredOverTwo` per-row audit field in the JSON output makes
this machine-checkable rather than just visually checkable.

## The bound and what it means

`(n−1)/2` is the saturation bound: the GE(2) value when one day
absorbs all tokens and `n−1` days are zero. For `claude-code` at
35 days the bound is `17.0`; for `opencode` at 11 days the bound
is `5.0`; for `codex` at 8 days the bound is `3.5`. The
`ge2Saturated` flag fires at `ge2 ≥ 0.95 * (n−1)/2`, which on the
current table is reached by no source. The closest is `codex` at
21% of saturation, which is a noteworthy but not pathological
top-heaviness. None of the six sources is in a degenerate one-
day-takes-all regime, which is itself a finding — every tracked
source distributes its tokens across multiple days even at the
quadratic-deviation lens.

This bound matters operationally because it sets the ceiling for
how informative a single tick's reading can be. On a short window
(8 days for codex) the bound is low and the dynamic range is
narrow; on a long window (73 days for vscode-other) the bound is
high and small absolute readings can still be substantively
significant. The `--min-days` default of 2 prevents the truly
degenerate case (a single-day source has no inequality to measure),
and the `--top` knob (default 0 = no cap) preserves the full table
when the operator wants to see the full ordering rather than a
top-N slice.

## Closing note

Axis-39 GE(2) at v0.6.277 closes the GE(α) moment family on the
daily-token axis at the canonical α ∈ {0, 1, 2}. The headline
finding from the live smoke-test — `claude-code` `ge2/T = 1.8998`
versus `opencode` `ge2/T = 0.7051` — is the first cross-source
witness that distinguishes interactive-burst usage profiles from
continuous-driver usage profiles at the second-moment shape. The
distinction is invisible to Gini, invisible to either Theil
separately, and invisible to mean or median statistics. The ratio
field `ge2OverT` is the right cross-anchor against axis-38 because
both numerator and denominator are computable on every source
(unlike `ge2OverL`, which would be `null` for any source with a
zero-day) and because the interpretation is clean: top-tail
concentration above 1, top-tail spread below 1, balanced near 1.
The fact that the boundary case (hermes at 0.9212) sits within
10% of the threshold means the band classification will be a
live-updated diagnostic on subsequent ticks — a regime-flip would
be a structural event worth noting, and the axis is now
instrumented to see it.
