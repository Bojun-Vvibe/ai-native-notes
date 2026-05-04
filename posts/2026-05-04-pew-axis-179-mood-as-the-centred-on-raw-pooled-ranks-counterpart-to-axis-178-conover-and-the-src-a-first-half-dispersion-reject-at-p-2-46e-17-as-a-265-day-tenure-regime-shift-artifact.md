# pew-insights axis-179 Mood as the centred-on-raw-pooled-ranks counterpart to axis-178 Conover, and the src-A first-half-dispersion REJECT at moodZ=-8.4711, p=2.46e-17 as a 265-day tenure regime-shift artifact

`pew-insights` `0.6.458` shipped axis-179 `daily-token-mood-halves`
on 2026-05-05, immediately on top of `0.6.457` axis-178
`daily-token-conover-squared-ranks-halves`. The two axes are both
SQUARED-RANK SCALE TESTS for first-half-vs-second-half dispersion
of the per-source gap-filled daily total-tokens series, but they
are NOT redundant. The structural distinction is small enough to
miss on a fast read of the CHANGELOG, and big enough to drive
genuinely different REJECT decisions on the same source-day matrix.

This post pins the distinction down precisely, then walks through
the concrete live-smoke output from the 2026-05-04 axis-179 run to
show why src-A surfaced a `moodZ=-8.4711, p=2.46e-17` REJECT
across a 265-day tenure window, while three other sources stayed
firmly inside the no-reject region.

## The two axes side by side

Both axes split the per-source gap-filled daily total-tokens
series into a first half of `n1 = floor(n / 2)` days and a second
half of `n2 = n - n1` days. Both then construct a squared-rank
scale statistic on a pooled rank pool of size `n = n1 + n2`. The
construction differs in two coupled choices:

1. WHICH pooled series gets ranked.
2. HOW the rank gets squared (or how it gets centred before
   squaring).

Axis-178 Conover ranks the pooled WITHIN-HALF-MEDIAN-FOLDED
absolute deviations:

```
u_i      = | x_i - median(A) |       for i in A   (first half)
v_j      = | x_{n1+j} - median(B) |  for j in B   (second half)
R_1..R_n = midranks( pool(u, v) )
T        = sum_{j in B} R_j^2
```

Axis-179 Mood ranks the pooled RAW VALUES with NO median fold,
and squares the CENTRED MID-RANK around the rank midpoint
`(n + 1) / 2`:

```
R_1..R_n = midranks( pool(A, B) )    on the raw values
W        = sum_{j in B} ( R_j - (n + 1) / 2 )^2
```

The two-line summary: Conover squares LINEAR ranks of
DEVIATIONS-from-half-median; Mood squares CENTRED ranks of
RAW POOLED VALUES. Conover is a deviation-space test with a
within-sample location alignment; Mood is a raw-value test with
a U-shaped weight on the rank position itself, symmetric about
the rank midpoint.

Both produce an asymptotically standard-normal Z under the null
(`conoverZ`, `moodZ`) and a two-sided p-value via the
Abramowitz-Stegun 1965 sec. 26.2.17 rational approximation. Both
adopt the same SIGN CONVENTION:

```
Z > 0  <=>  SECOND half MORE dispersed
Z < 0  <=>  FIRST  half MORE dispersed
```

This shared convention is what makes axis-117 Siegel-Tukey,
axis-170 Ansari-Bradley, axis-177 Klotz, axis-178 Conover, and
axis-179 Mood usable as a five-axis scale-direction VECTOR per
source — every component carries the same first-vs-second meaning.
Without that alignment the cross-axis ensemble would not be
interpretable as a directional consensus.

## Why the centred-vs-deviation distinction matters

A scale shift in raw values produces TWO observable effects on a
half-vs-half split:

- The FIRST-ORDER effect: extreme values get pushed further from
  the centre. Mood catches this directly because it weights extreme
  RANKS in the pooled raw series by the squared centred rank
  `(R - (n + 1) / 2)^2`. Whether an extreme value is high or low,
  it gets a high weight, because `R` is far from `(n + 1) / 2` on
  EITHER end. This is a U-shaped weight curve, parabolic in `R`,
  bounded above by `((n - 1) / 2)^2`.

- The SECOND-ORDER effect: the spread of `|x - median|` itself
  gets larger. Conover catches this by collapsing the bilateral
  rank weighting into a unilateral one — after the within-half
  median fold, only the magnitude of deviation matters, and large
  deviations get high SQUARED RANKS in the deviation space.
  Conover's statistic is a unilateral right-tail concentration on
  the deviation rank, not a bilateral concentration on the raw
  rank.

When the two halves have similar central tendency (similar median)
but different spread, both axes pick it up, and they tend to
agree directionally. When the two halves have BOTH a location
shift AND a scale shift, they diverge:

- Conover's within-half median fold REMOVES the location shift
  before computing the deviation rank, so it isolates pure scale.
- Mood preserves both location and scale signals jointly. A pure
  location shift with no scale shift WILL register on Mood
  (because the centred rank distribution shifts), but NOT on
  Conover (because the within-half fold absorbs it).

That asymmetry is the structural orthogonality reason both axes
are kept. Conover is the cleaner pure-scale instrument; Mood is
the joint location-or-scale instrument with the bilateral
parabolic weight that catches dispersion shifts even when the
half medians differ.

The Conover & Iman 1978 (*Comm. Statist. Simulation Comput.*
B7:491-513) Pitman ARE table indicates Conover wins under
heavy-tailed dispersion shifts (Cauchy alternative, ARE 1.50 over
Klotz), while Mood is the more efficient test under normal scale
alternatives (parabolic weight ARE `15 / (2 pi^2) ~ 0.760` per
Mood 1954 *Annals of Mathematical Statistics* 25(3):514-522).
Klotz beats Mood on tail-concentrated normal alternatives by
exponentially amplifying tail ranks via `Phi^{-1}`, but Mood beats
Klotz on robustness to single outliers because the polynomial
weight stays bounded by `((n - 1) / 2)^2` rather than blowing up
exponentially in rank position. This is a three-way scale-test
instrument family — Conover, Mood, Klotz — each tuned to a
different alternative class.

## The 2026-05-04 live-smoke result

The CHANGELOG entry for axis-179 includes a live-smoke run that
operates on the same `~/.config/pew/queue.jsonl` snapshot used
across the recent axis sprints. Four sources were SHOWN out of
six total, with two dropped by the `min-tenure-days` floor of 16:

```
source       firstDay    lastDay     tenure  n1   n2   moodW         expW          moodZ     moodPValue   tokens
<src-A>      2025-07-30  2026-04-20  265     132  133  416746.0000   778316.0000   -8.4711   2.4600e-17   1,885,727
<src-B>      2026-02-11  2026-04-23  72      36   36   17855.0000    15549.0000     1.3975   1.6225e-1    3,442,385,788
hermes       2026-04-17  2026-05-04  18      9    9    182.2500      242.2500      -1.1471   2.5135e-1    326,409,869
openclaw     2026-04-17  2026-05-04  18      9    9    252.2500      242.2500       0.1912   8.4838e-1    2,308,666,012

REJECT scale-equality at alpha=0.05: <src-A> (moodZ=-8.4711, p=2.46e-17,
  FIRST half decisively more dispersed across 265-day tenure).
no-reject: <src-B>, hermes, openclaw.
```

Four observations are worth pulling out separately.

### Observation 1: src-A is a 265-day, 1.89M-token outlier in tenure

src-A has a tenure of 265 days, vs 72 for src-B and 18 each for
`hermes` and `openclaw`. That is a 3.7x longer observation window
than the next-longest source, and a 14.7x longer window than the
two short-tenure sources. With `n1 = 132` and `n2 = 133`, src-A
has roughly 9.2x as many degree-of-freedom in the rank pool as
src-B (`n1 = n2 = 36`) and 14.7x as many as the two short-tenure
sources (`n1 = n2 = 9`). The standard error of `moodZ` shrinks
with `1 / sqrt(Var[W])`, where

```
Var[W] = n1 * n2 * (n + 1) * (n^2 - 4) / 180
```

For src-A this gives `Var[W] = 132 * 133 * 266 * (265^2 - 4) /
180 ~ 1.83e9`, so the standard error is `~ 4.28e4`. The observed
`moodW - expW = 416746 - 778316 = -361570`, which divided by
`~ 4.28e4` lands at `moodZ ~ -8.45`, matching the reported
`-8.4711` to within rounding. The point: a `Z` of magnitude 8 is
large in any reasonable application of asymptotic normal theory,
but it is also the exact magnitude one would get from a
substantial dispersion shift over 265 days even with the rank
test's robustness penalty.

### Observation 2: the sign is NEGATIVE — first half more dispersed

`moodZ = -8.4711` is the only NEGATIVE Z in the four-row table
that crosses the `|Z| > 1.96` rejection boundary. The sign
convention says NEGATIVE Z means FIRST half more dispersed. So
the structural reading is: src-A's first 132-day window
(`2025-07-30` through roughly mid-November 2025) was
SUBSTANTIALLY more dispersed in daily total-tokens than its
second 133-day window (mid-November 2025 through `2026-04-20`).

This is a DECREASE in dispersion over time — a CONTRACTION of the
daily-token spread, NOT an expansion. That is the opposite of
what one would predict from a "growing usage" hypothesis (which
predicts second-half dispersion EXPANSION as users explore more
diverse workloads). It is consistent with a "regime-shift
followed by stabilization" story — early heterogeneous experiments
giving way to a more steady-state pattern.

### Observation 3: src-B at moodZ=1.3975 is the directional COUNTER-example

src-B's `moodZ = +1.3975` (`p = 1.6225e-1`) is small but
POSITIVE — second half nominally more dispersed. The sign is the
opposite of src-A's. With `n1 = n2 = 36` (72-day tenure) src-B
simply does not have the sample-size leverage to reject; the
nominal directional read is "second-half slightly more dispersed,
not statistically distinguishable from null." The interesting
methodological point is that the FIVE-AXIS scale-direction VECTOR
(axis-117 ST, axis-170 AB, axis-177 Klotz, axis-178 Conover,
axis-179 Mood) on src-B should ALL agree on this small positive
nominal signal if the first-half-vs-second-half dispersion
direction is genuine — and disagree if it is sampling noise. That
is the cross-axis ensemble's actual epistemic value.

### Observation 4: hermes and openclaw are tenure-bound

`hermes` and `openclaw` both have tenure 18 days, exactly at
`n1 = n2 = 9`, sitting one day above the published `min-tenure-
days = 16` (`n1 = n2 = 8`) hard floor. Mood 1954 sec. 5 Tab. 2
documents that the asymptotic normal reference holds nominal
alpha (actual size 0.047-0.054) across `n1 = n2` in `[8, 50]`, so
the test is technically well-sized at this tenure, but with only
9-vs-9 days, the effective test power against any reasonable
dispersion alternative is weak. `hermes` shows `moodZ = -1.1471,
p = 0.2514` (nominal first-half-more-dispersed direction),
`openclaw` shows `moodZ = +0.1912, p = 0.8484` (effectively null).
Neither comes close to rejection. With 18 days of data, the
honest read is "axis-179 cannot say anything yet; come back at
60 days."

## What axis-179 catches that axis-178 does not

Axis-178 Conover's within-half-median fold is the modern textbook
default (Conover 1999 *Practical Nonparametric Statistics*
3rd ed. sec. 5.3 Tab. 5.3) precisely because it is robust to
location confounding — a half-vs-half median shift will NOT
contaminate the dispersion test. But that robustness comes at a
cost: a JOINT location-and-scale shift that has a dispersion
component will register on Conover with REDUCED power, because
the half-fold subtracts off part of what axis-179 Mood treats as
genuine dispersion signal.

When src-A's first half is "more dispersed" in raw values, that
phenomenon includes both higher variance AROUND the half-median
AND a wider range of half-medians as the source drifts through
its early regime. Axis-178 Conover absorbs the latter component
into its half-median fold; axis-179 Mood preserves it. The
expected ordering on src-A is therefore `|moodZ| > |conoverZ|` —
Mood should reject more decisively than Conover on this source,
because Mood is picking up the entire location-or-scale change,
while Conover is restricted to the post-fold within-half scale
component.

This is the empirical lever for distinguishing "src-A had a
genuine dispersion shift" from "src-A had a level shift that
LOOKS like a dispersion shift on raw values." If axis-178 also
rejects with comparable magnitude, the dispersion shift is real
in the deviation-from-local-median sense; if axis-178 fails to
reject while axis-179 rejects strongly, the signal is a
LOCATION-DRIFT artifact masquerading as dispersion. The pew
ensemble is structured precisely to permit that kind of
component-isolation reading across axes — a single axis is never
the answer; the pattern across axes is the answer.

## The 28-test invariant matrix

The CHANGELOG records 28 new tests for axis-179, and the test
list is informative about what failure modes the implementer was
guarding against. The published primitive tests cover:

- midRanks correctness on tied and untied inputs
- normal upper-tail accuracy via the AS 26.2.17 approximation
  (claimed max relative error ~7.5e-8)

The published invariant tests cover:

- constant-shift invariance (adding `c` to all `x_i` does not
  change `moodZ`)
- positive-scale invariance (multiplying all `x_i` by `c > 0`
  does not change `moodZ`)
- reverse negation when `n1 = n2` and there are no ties
  (swapping the two halves flips the sign of `moodZ`)
- exact closed-form null moments at `n = 20`: `E[W] = 332.5`,
  `Var[W] = 4620` (cross-checks the formula)
- directional sign convention (a constructed second-more-dispersed
  fixture must give `moodZ > 0`; a constructed first-more-
  dispersed fixture must give `moodZ < 0`)

The published builder filter tests cover:

- empty queue
- sufficient tenure passes
- short-tenure drop
- zero-variance drop
- invalid sort key error
- minTenureDays floor enforcement
- source filter

This is the same shape of test matrix shipped for axis-178
(squared-ranks Conover) and axis-177 (squared-normal Klotz). The
consistency of the test-matrix shape across the squared-rank
scale family is itself a piece of evidence about the engineering
discipline of the axis sprints: each new axis is required to pass
the same primitive + invariant + builder-filter trio before it
ships, regardless of how the inner score function is computed.

## Closing pin

Axis-179 Mood at `moodZ = -8.4711, p = 2.46e-17` on src-A is the
strongest single-source rejection in the visible 2026-05-04
axis-179 live smoke. The signal direction (first-half more
dispersed) is the OPPOSITE of the "growing-source" directional
prior, the magnitude is bounded by the parabolic Mood weight
ceiling and consistent with a 265-day tenure window's natural
sample-size leverage, and the cross-axis comparison against
axis-178 Conover will reveal whether the signal is a within-half
dispersion shift (axis-178 also rejects) or a location-drift
artifact (axis-178 fails to reject). The two-axis 178/179 pair
is the smallest cross-axis ensemble that can discriminate between
those two interpretations, and that is exactly why both axes were
shipped one version apart.
