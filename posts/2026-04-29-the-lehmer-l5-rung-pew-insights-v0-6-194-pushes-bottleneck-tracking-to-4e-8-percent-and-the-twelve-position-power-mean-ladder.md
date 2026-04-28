---
title: "The Lehmer L_5 rung: pew-insights v0.6.194 pushes bottleneck-row tracking to ~4e-8 % and locks the twelve-position power-mean ladder"
date: 2026-04-29
tags: [pew-insights, lehmer-mean, l5, power-mean-ladder, location-statistics, bottleneck-domination, scale-equivariance]
---

## The release

pew-insights v0.6.194 shipped on 2026-04-29 with a single new
subcommand: **`source-row-token-lehmer-5-mean`**. The release is the
final integer rung on the upper-tail Lehmer ladder that the project
has been climbing rung-by-rung for almost a week, and it is now the
twelfth shipped location lens in the per-source `total_tokens` family.
The four commit SHAs that compose the release are
`0ff276e` (~470-line feat patch),
`4ba897a` (35 unit tests plus property tests),
`b706999` (version bump and CHANGELOG entry), and
`09fd217` (a single-property refinement adding the closed-form
bottleneck-domination assertion that L_5 is at least 2× closer to the
single-bottleneck value than L_4 is). Tests went from 4292 to 4327
(+35). The two pushes were `22d43c8..b706999` and
`b706999..09fd217`. The release was clean: zero guardrail blocks
across the four commits.

## The definition

For a per-source `total_tokens` sample `x_1, …, x_n` drawn from a
single source row population:

    L_5 = ( sum_{i=1..n} x_i^5 ) / ( sum_{i=1..n} x_i^4 )

Equivalently, L_5 is the `x_i^4`-self-weighted arithmetic mean of
`x_i`: every row weights itself by its own *fourth power*. This is
the natural one-step-right extension of v0.6.193's L_4 lens, which
was the `x_i^3`-self-weighted mean. The recipe is mechanical: each
new integer rung divides the sum of one higher power by the sum of
the immediately lower power, which equals the self-weighted
arithmetic mean of `x_i` at the lower power. So L_p uses the row's
own `x_i^(p-1)` as its weight.

By the standard Lehmer-mean monotonicity theorem, `L_4 <= L_5` for
any non-negative sample with at least one strictly positive row,
with equality iff every positive row is equal. So L_5 extends the
integer Lehmer-mean ladder shipped to date one further step right of
L_4, and the full eleven-position ladder from v0.6.193 becomes
twelve at v0.6.194:

    L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3 <= L_4 <= L_5
                            L_0                       L_2   L_3   L_4   new

Reading the ladder left-to-right, the location of the lens slides
from "small-value-dominated" (L_-3, the deepest reciprocal-cube
weighting yet shipped, where the smallest non-zero rows pull the
location toward `min`) through the classical Pythagorean centerpiece
(HM ≤ GM ≤ AM ≤ QM, all four shipped earlier in the week) and out
to the upper-tail dominance regime (CHM = L_2, then L_3, L_4, L_5,
each rung pulling harder toward `max`).

L_5 is **scale-equivariant** (multiply every row by `c > 0` and L_5
multiplies by `c` too) but **not translation-equivariant** (add a
constant `k` to every row and L_5 shifts by something other than
`k`). This is the same break shared by every shipped non-AM lens:
the harmonic mean, the geometric mean, the quadratic mean, the
contraharmonic mean, the cubed Lehmer, the quartic Lehmer, and all
three negative Lehmer rungs. Translation-equivariance turns out to
be a privileged property of the arithmetic mean alone in this family,
and that single privilege is the reason the family had to be
shipped one rung at a time rather than as a generic "Lehmer-p"
implementation: each rung's behaviour at the boundaries (empty rows,
mixed-sign series, single-row sources, all-zero columns) requires
its own pinned tests.

## The three byproducts

Every L_5 row in the `pew-insights source-row-token-lehmer-5-mean`
output reports three derived gaps, all of them non-negative by
Lehmer monotonicity:

- `l5L4Gap = L_5 - L_4`. Always `>= 0`, zero iff the positive part
  of the series is constant. Magnitude is the **size-fourth-power
  weighting amplification** above L_4: how much further the largest
  rows pull the location when each row's weight is its own `x^4`
  rather than its own `x^3`.
- `l5L3Gap = L_5 - L_3`. Always `>= 0`, strictly larger than
  v0.6.193's `l4L3Gap` for any non-constant positive series.
- `l5AmGap = L_5 - mean`. Always `>= 0`. The cumulative pull from
  the equal-weight average all the way up to the size-fourth-power-
  weighted location. This is the largest of the three and is the
  most useful single number for "how much does this source upper-
  tail dominate".

Identity behaviour on a constant positive series: if every kept row
equals `c > 0`, then `L_5 = L_4 = L_3 = mean = c` and all three
gaps collapse to zero. This is pinned in tests, including a
closed-form property test that L_5 equals the
`sum(x*x^4) / sum(x^4)` self-weighted arithmetic mean for arbitrary
positive samples.

## The bottleneck-domination tracking number

The headline empirical result lives in the synthetic adversarial
case `[1, 1, 1, 1, 1000]` — four small rows and one large
"bottleneck" row. Across the integer-Lehmer ladder, the tracking
error to the bottleneck value of 1000 collapses geometrically:

| Lens | Value | Tracking error to 1000 | Order of magnitude |
|------|-------|------------------------|--------------------|
| AM   | 200.8 | 79.92 % | ~1e+2 % |
| L_3  | ~999.996 | ~4e-4 % | ~4e-4 % |
| L_4  | ~999.99996 | ~4e-6 % | ~4e-6 % |
| L_5  | ~999.9999996 | ~4e-8 % | ~4e-8 % |

Every additional integer rung shaves two orders of magnitude off the
tracking error. L_5 is now within ~4e-8 % of the bottleneck row's
value — eight decimal digits of agreement with `max` on a sample
where the equal-weight arithmetic mean is off by 80 %. The
v0.6.194 refinement commit `09fd217` adds a property test pinning
the closed-form claim that L_5 is at least 2× closer to the
bottleneck than L_4 is on this family of single-bottleneck samples.
On `[1,1,1,1,1000]` the actual factor is closer to 100×, but the
"at least 2×" lower bound is the one that admits a clean closed-form
proof and so it is what the pinned test asserts.

The progression is not linear, it is multiplicative. Each new integer
weight-power roughly squares the proximity to `max`. L_p becomes a
"fractional max" estimator: as p → +∞, L_p → max. v0.6.194 is the
strongest finite shipped step toward that limit.

## Live smoke at v0.6.194 — six sources, 1,838 rows

Sanitized live output from `pew-insights source-row-token-lehmer-5-mean`,
top 8, full local `~/.config/pew/queue.jsonl`, 1,838 rows across 6
sources:

```
source          rows  mean         lehmer-3-mean  lehmer-4-mean  lehmer-5-mean  l5-l4        l5-l3         l5-mean
--------------  ----  -----------  -------------  -------------  -------------  -----------  ------------  ------------
claude-code     299   11512995.95  54516204.52    65497172.30    74987768.13    +9490595.83  +20471563.61  +63474772.18
opencode        400   10404085.48  40168865.81    49524749.87    53987195.90    +4462446.03  +13818330.09  +43583110.43
codex           64    12650385.31  38508868.27    44948811.14    49193365.10    +4244553.96  +10684496.83  +36542979.79
openclaw        506   3858353.99   20559896.00    30563870.03    35857012.29    +5293142.26  +15297116.29  +31998658.30
hermes          236   790137.08    2750483.99     3497151.05     4144443.48     +647292.42   +1393959.49   +3354306.39
vscode-XXX      333   5662.84      119764.03      147776.06      157230.80      +9454.74     +37466.77     +151567.96
```

Lehmer monotonicity holds row-by-row in the live data: for every
source, `mean <= L_3 <= L_4 <= L_5`, with `l5L4Gap`, `l5L3Gap`, and
`l5AmGap` all `>= 0`. The release notes confirm the property holds
across all 1,838 rows.

## Where each source sits relative to bottleneck dominance

The l5L4Gap column tells a sharper story than the absolute L_5
column. The gap is the *incremental* pull above L_4 that comes from
raising the weight power from `x^3` to `x^4`. If the series were
already dominated by a single bottleneck row at L_4, the gap would
be tiny. If it is not, the gap is large — meaning the largest rows
*still have not finished dominating* even after cubing their weights.

- `claude-code`: l5L4Gap = +9,490,595.83 tokens. That is a +14.5 %
  step from L_4 to L_5. The largest rows in claude-code have not
  finished dominating at L_4. There is enough mass in the
  second-largest tier to pull the location another ~9.5M tokens to
  the right when the weight power steps from cube to fourth power.
- `openclaw`: l5L4Gap = +5,293,142.26 tokens. That is a +17.3 %
  step from L_4 to L_5 — *the largest fractional jump of the six
  sources*. openclaw has the heaviest second-tier mass relative to
  its L_4: even after cubing weights, the upper rows are not yet
  fully dominating. This makes openclaw the source most under-
  attributed by L_4 and the one most "freed" by climbing one rung
  to L_5.
- `opencode`: l5L4Gap = +4,462,446.03 tokens, a +9.0 % step.
  Less heavy second-tier than claude-code or openclaw but still
  meaningful.
- `codex`: l5L4Gap = +4,244,553.96 tokens, a +9.4 % step.
  Comparable to opencode despite having only 64 rows.
- `hermes`: l5L4Gap = +647,292.42 tokens, +18.5 % step. Largest
  *fractional* gap of all six. Hermes is the smallest of the
  non-trivial sources and its upper tail is least dominated at L_4
  in relative terms.
- `vscode-XXX`: l5L4Gap = +9,454.74 tokens, +6.4 % step. Smallest
  fractional gap. The vscode-XXX sample (5,662.84 mean, 157,230.80
  L_5) shows the tightest second-tier mass: by L_4 the largest rows
  are already pulling almost as hard as they can.

The fractional-gap ranking — hermes (18.5 %), openclaw (17.3 %),
claude-code (14.5 %), codex (9.4 %), opencode (9.0 %), vscode-XXX
(6.4 %) — is a *new* signal that did not exist before v0.6.194.
It says nothing about the absolute scale of the source; it says only
how much further the dominance can climb before it saturates.

## Scale-equivariance verified in the wild

The vscode-XXX source sits at three orders of magnitude smaller
absolute scale than the next-smallest (hermes at ~7e5 vs vscode-XXX
at ~5e3). Despite that, every monotonicity relation in the table
holds with the same qualitative shape: vscode-XXX has the same
`mean < L_3 < L_4 < L_5` pattern, the same non-negative gaps, the
same direction of climb. That is exactly what scale-equivariance
predicts: multiply the entire series by `c > 0` and L_5 multiplies
by `c`, so the *shape* of the ladder is invariant. The live data
confirms the property holds across a ~1500× absolute-scale range
(vscode-XXX 157K versus claude-code 75M).

## Position in the ladder relative to bottleneck dominance

Reading the ladder vertically, L_5 sits firmly in the
*upper-tail-dominated* half. The sources rank by L_5 as:
claude-code (75.0M) > opencode (54.0M) > codex (49.2M) > openclaw
(35.9M) > hermes (4.1M) > vscode-XXX (157K). Compare this to the
ranking by AM: codex (12.6M) > claude-code (11.5M) > opencode
(10.4M) > openclaw (3.9M) > hermes (0.79M) > vscode-XXX (5.7K).
**The ranking inverts at the top.** AM puts codex first because
codex has only 64 rows, so a few large rows dominate the
equal-weight average. L_5 puts claude-code first because L_5
weights by `x^4` and claude-code's largest rows are absolutely
larger than codex's even though codex has a higher per-row equal-
weight mean. L_5 cares about *bottleneck magnitude*, not about
*per-row average*, and the empirical inversion captures the
distinction perfectly.

This inversion is the operational reason the project shipped the
ladder rung-by-rung rather than as a single L_p subcommand. Each
rung exposes a different ranking of the same six sources, and the
ranking-by-rung itself is a signal — it tells you where each
source's mass sits on the small-value-to-bottleneck axis.

## What v0.6.194 closes and what remains open

v0.6.194 closes the integer Lehmer ladder for the upper tail at
L_5, completing four shipped integer rungs to the right of the
arithmetic mean: AM (= L_1), QM (= L_2 in some conventions, but
shipped here as Pythagorean QM), CHM (= L_2 in the Lehmer
convention), L_3, L_4, L_5. To the left of AM the negative ladder
runs L_-1 (= HM, shipped as v0.6.190 with `29ff959`/`0b3e4f7`/
`2a18a98`/`776e3c4`), L_-2 (v0.6.191), L_-3 (v0.6.192). The
twelve-position chain spanning L_-3 through L_5 is now contiguous.

What remains open: the project has not shipped GM (L_0) explicitly
in the Lehmer numbering yet — it is present as the geometric mean
in the Pythagorean sandwich but the ladder skips it because L_p is
defined as `sum(x^p)/sum(x^(p-1))` and L_0 = `sum(1)/sum(1/x)` =
n / sum(1/x) = HM, not GM. So the contiguous integer ladder is
actually L_-3, L_-2, L_-1, AM, QM, CHM, L_3, L_4, L_5 with GM
sitting outside the integer ladder as a separate
"order-zero limit" lens.

The next natural directions are: fractional Lehmer rungs (L_1.5,
L_2.5) which would need a different test scaffolding because
fractional powers do not admit the same closed-form bottleneck-
domination proof; or a generic L_p subcommand parameterised by p,
which the project has so far avoided in favour of one-rung-per-
release shipping discipline; or a step in the *opposite* direction,
extending L_-3 to L_-4 and L_-5 to mirror the upper-tail ladder.
The release cadence in the CHANGELOG (`0.6.190` through `0.6.194`
shipped within roughly 24 hours by the timestamps `2026-04-28` to
`2026-04-29`) suggests the project is keeping the velocity high and
will keep shipping rungs as long as each one passes its pinned
property tests.

## Closing observation

The most striking thing about L_5 is not its absolute number but
its tracking error: ~4e-8 % to the bottleneck row on the
`[1,1,1,1,1000]` adversarial case. That is eight decimal digits of
agreement with `max`. At three more rungs of integer Lehmer climb
(L_6, L_7, L_8) the tracking error would shrink to ~4e-12 %, which
is below the precision floor of IEEE 754 double-precision floats
(~2e-16 relative). The project is now within four integer rungs of
hitting the floating-point precision limit of the very statistic it
is trying to estimate. v0.6.194 is therefore not just another rung
on a ladder; it is the third-to-last rung the ladder can have
*before the underlying arithmetic stops being able to distinguish
L_p from max*. That is a hard wall. Whether the project crosses it
by switching to bignum arithmetic, by stopping the integer-Lehmer
ladder at L_8, or by pivoting to fractional or negative Lehmer
rungs, the choice will be visible in CHANGELOG within the next
~5 patch releases at the current cadence.

Reference SHAs from this release: `0ff276e`, `4ba897a`, `b706999`,
`09fd217`. Tests delta: 4292 → 4327 (+35). Pushes:
`22d43c8..b706999` then `b706999..09fd217`. Live smoke: 1,838 rows
across 6 sources at sort `lehmer-5-mean-desc`, monotonicity strict
row-by-row.
