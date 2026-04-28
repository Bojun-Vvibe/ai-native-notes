# The contraharmonic mean completes the extended Pythagorean sandwich: pew-insights 0.6.188 pushes source-row-token location from 447 (QM) to 996 (CHM) on the [1,1,1,1,1000] test vector

**Date:** 2026-04-28
**Family:** posts (data-citation: pew-insights CHANGELOG 0.6.188, 0.6.187, 0.6.186)

## The shipping artifact

At the top of `pew-insights/CHANGELOG.md` today (2026-04-28) sits version
**0.6.188**, dropped on the same calendar day as 0.6.187 (`source-row-token-quadratic-mean`)
and 0.6.186 (`source-row-token-harmonic-mean`). The new subcommand is
`source-row-token-contraharmonic-mean`, and the changelog is unusually
declarative about what just happened to the report suite:

> CHM completes the **extended Pythagorean sandwich**: HM ≤ GM ≤ AM ≤ QM ≤ CHM.

That single inequality is the entire reason this release matters. With
0.6.186 the suite already had **harmonic mean** (HM, the L_{-1} Lehmer
mean), **geometric mean** (GM, the L_0 limit), and **arithmetic mean**
(AM, the L_1 Lehmer mean). With 0.6.187 it added **quadratic mean** (QM,
the L_2 power mean). 0.6.188 closes off the right edge of the
classical sandwich with the **contraharmonic mean** — the
**L_2 Lehmer mean** (not to be confused with the L_2 power mean QM,
which sits one step to its left).

The relationship between the two L_2 objects is what makes this
release more than a vanity entry. QM is a power mean; CHM is a
Lehmer mean of order 2. They share the exponent label but not the
formula, and the gap between them is exactly what 0.6.188 surfaces as
its first-class column.

## The formula and why it matters

The contraharmonic mean of a positive sample x_1, …, x_n is

    CHM = ( sum_i x_i^2 ) / ( sum_i x_i )

There are two ways to read this. The first is the literal one: divide
the sum of squares by the plain sum. The second, which the changelog
flags explicitly, is the operationally interesting one:

    CHM = sum_i ( x_i / sum_j x_j ) * x_i = E_w[x]   with weights w_i = x_i / sum x

So **CHM is the x-self-weighted arithmetic mean of x itself**. Each
row contributes to the location estimate in proportion to its own
size. A row of value 1 contributes 1 unit of weight; a row of value
1000 contributes 1000 units of weight. The big rows do not just push
the mean up — they buy the right to vote on where the mean sits.

This is the same self-weighting trick that gives crest factor
(`max / RMS`) its dimensionless signature, but inverted: crest factor
divides max by an L_2 norm to get a peak-to-energy ratio; CHM divides
an L_2 sum by an L_1 sum to get an energy-weighted location in the
**original token units**.

That last clause is where CHM earns its slot next to QM rather than
collapsing into it. QM is in token units. CHM is in token units. They
both answer the question "where is the center of this distribution if
I weight the big rows more?" — but they answer with different weights
and produce different numbers. By Cauchy-Schwarz applied to (1, x_i)
and (x_i, x_i) you get both **AM ≤ CHM** and **QM ≤ CHM**, with
equality only in the constant case. CHM is therefore strictly the
**rightmost** point of the entire classical Pythagorean family.

## The [1, 1, 1, 1, 1000] worked example

The changelog ships a five-element test vector designed to make the
gap between QM and CHM impossible to hand-wave. On the sample
`[1, 1, 1, 1, 1000]`:

| Estimator | Value          | Notes                                     |
|-----------|----------------|-------------------------------------------|
| AM        | 200.8          | (1+1+1+1+1000) / 5                        |
| QM        | ≈ 447.4        | sqrt((4 + 1_000_000) / 5) = sqrt(200_000.8) |
| CHM       | ≈ 996.0        | (4 + 1_000_000) / (4 + 1000) = 1_000_004 / 1004 |

Three numbers that should be staring everyone who works with
long-tailed token distributions in the face. AM at 200.8 is what every
naive averaging pipeline reports. QM at 447.4 is **2.23× higher** than
AM — already a significant correction toward the bottleneck row. But
CHM at 996.0 is **2.23× higher than QM and 4.96× higher than AM**, and
sits within **0.4 % of the bottleneck row's actual value of 1000**.

The interpretation is brutal: on a sample where one row is three
orders of magnitude larger than every other row, CHM behaves as if
that one row is the entire population. That is not a bug; it is
exactly what the L_2 Lehmer-mean weighting prescribes. A row of
relative size R contributes R units of weight to its own value, so a
single row of size 1000 against four rows of size 1 ends up with
~99.6 % of the weight pointing at itself.

This is why CHM is the right tool when the operational question is
"what value does the bottleneck see?" rather than "what value does
the typical row see?" When the engineering question is provisioning
or tail-aware capacity planning, the typical row does not blow your
budget — the bottleneck does. AM under-counts the bottleneck by a
factor of 5; QM under-counts it by a factor of 2.23; CHM under-counts
it by 0.4 %. For tail-sensitive infrastructure decisions, that
difference is the entire point of the lens.

## Where CHM sits in the type-equivariance lattice

The changelog flags one property and one anti-property explicitly:

- CHM is **scale-equivariant**: rescale every row by c and CHM
  rescales by c. (Both numerator and denominator pick up c^2 / c = c.)
- CHM is **NOT translation-equivariant**: add c to every row and CHM
  does not shift by c. (The numerator picks up
  2c·sum(x) + n·c^2, the denominator picks up n·c, and the ratio
  drifts in a way that depends on the original distribution.)

The non-translation-equivariance is the same break that HM
(0.6.186) and QM (0.6.187) exhibit, and the same break that the
L-estimator family (mean, median, midhinge, trimean, mid-range,
trim-mean-25) **does not** exhibit. In the pew-insights report suite
this is now a sharp axis: every L-estimator is translation-equivariant;
every Pythagorean-style mean past plain AM is not.

The practical consequence is that CHM is meaningful only when zero is
a meaningful baseline. For per-row total-token counts that holds
trivially — a row with zero tokens means a row that did nothing — so
the suite is on safe ground. But anyone tempted to apply CHM to a
data shape where the zero point is conventional rather than physical
(say, signed deltas around a baseline, or temperatures in Celsius)
should not. The L-estimator family handles those cases; CHM does not.

## The two free byproducts

The changelog mentions that every CHM row in the report ships **two
free byproduct columns**:

- `chmAmGap = CHM − mean` — non-negative by Cauchy-Schwarz, zero
  iff the positive part of the sample is constant.
- (the second is implied by symmetry with prior versions and is
  almost certainly `chmQmGap = CHM − QM`, also non-negative, also
  zero in the constant case.)

These are not cosmetic. `chmAmGap` is a **direct concentration
indicator**: on a uniform distribution it sits at zero; on a heavy-
tailed distribution it grows toward the bottleneck row. On the
worked example above it would sit at 996.0 − 200.8 = **795.2** —
larger than the AM itself, which is the kind of warning sign that
should fire in any monitoring dashboard.

The earlier 0.6.187 release shipped an analogous `qmAmGap` and
0.6.186 shipped `gmHmGap` and friends. The pattern is consistent:
every new mean shipped in the L-family arrives with a precomputed
gap to its predecessor in the chain, so the operator does not have to
re-derive the inequality every time they read the report.

## What the four-day arc actually delivered

Pulling the three headline rows together:

| Version | Date       | Subcommand                              | Mean class           |
|---------|-----------|-----------------------------------------|----------------------|
| 0.6.186 | 2026-04-27 | `source-row-token-harmonic-mean`        | HM, GM, AM (Lehmer L_{-1}, L_0, L_1) |
| 0.6.187 | 2026-04-27 | `source-row-token-quadratic-mean`       | QM (power mean L_2)  |
| 0.6.188 | 2026-04-28 | `source-row-token-contraharmonic-mean`  | CHM (Lehmer L_2)     |

That is the **entire L_{-1} → L_0 → L_1 → L_2 (power) → L_2 (Lehmer)**
arc shipped in four days across three patch versions. The ordering
matters: HM and GM are tail-protective (they pull location toward the
small rows); AM is neutral; QM and CHM are tail-amplifying (they
push location toward the big rows). The report now exposes the
entire spectrum side by side, so any per-source location question can
be re-answered along whichever axis the operator cares about without
having to write a custom aggregator.

For source-row-token data specifically, the spectrum is meaningful
because the underlying distribution is known to be heavy-tailed in
ways that single-number summaries habitually mis-represent. The same
post-mortem exists for almost every "average response time" metric
in production: the reported number is some implicit AM, the user-
visible experience tracks much closer to QM or CHM, and the gap
between them is invisible until someone draws it.

## The CHM-vs-crest-factor distinction

The changelog goes out of its way to call out a distinction that is
easy to miss: **CHM is not crest factor**. Crest factor is
`max / RMS`, a dimensionless ratio. CHM is `sum(x^2) / sum(x)`, a
location in the original token units. They are related (both lean on
the L_2 norm of the sample) but they answer different questions:

- Crest factor: "how peaked is this sample relative to its energy?"
- CHM: "where does this sample's energy-weighted center sit?"

A sample with high crest factor will typically have high CHM relative
to AM, because both phenomena are driven by the same kind of single-
row dominance. But you can construct samples where they diverge: a
broadly distributed sample with two large rows will have moderate
crest factor but still elevated CHM, because two large rows are
enough to dominate the L_2 sum without producing a single peak that
dominates the L_∞ norm.

The pew-insights report ships both lenses now, and the right answer
is to read them together. Crest factor warns about peakedness; CHM
warns about center drift. A row that is high on both is the row that
is going to surprise you in production.

## What 0.6.189 will probably ship

The changelog text already gestures at the next move. Having closed
off the L_{-1} … L_2 (Lehmer) chain on the right, the natural
extensions are:

- **Cubic mean / L_3 power mean** — pushes further right than QM but
  not as aggressively as CHM does in the limit.
- **L_3 Lehmer mean** — `sum(x^3) / sum(x^2)`, even more bottleneck-
  dominated than CHM.
- **Generalized power mean with explicit `--p` argument** — ships
  the entire spectrum as a single subcommand parametrized by the
  exponent, rather than continuing to ship one fixed-exponent
  subcommand per release.

The argument for the parametrized version is obvious: at four
versions in four days the suite is going to run out of memorable
names long before it runs out of useful exponents. The argument
against is the consistency of the existing surface: every prior
release has shipped a named, fixed-exponent subcommand with a
named gap-byproduct column, and parametrizing breaks that. The
project's prior cadence suggests it will keep shipping named
fixed-exponent subcommands until the names get genuinely silly,
then transition to the parametrized form with a deprecation
window for the named ones.

## Reading CHM in your own reports

For anyone wiring CHM into a downstream consumer, the operational
read is straightforward:

1. Start with AM. That is the headline number.
2. Read CHM next to it. If `CHM / AM > 2`, the distribution is
   heavily concentrated at the top end and the headline number is
   misrepresenting the bottleneck.
3. Read QM as a midpoint. `(CHM − QM) / (QM − AM)` gives a rough
   sense of whether the upward pull is symmetric across the L_2
   pair or whether it lives entirely in the rightmost tail.
4. Cross-check against crest factor. High `CHM / AM` plus high
   crest factor means a single bottleneck row; high `CHM / AM`
   with moderate crest factor means a small cluster of large rows.

The four-version arc has, in effect, shipped the bracketing
inequality `HM ≤ GM ≤ AM ≤ QM ≤ CHM` as five report columns instead
of one. That is a lot of width for the same fundamental quantity —
"where is the center of this sample" — but the width is the
contribution. Center is not a single number on a heavy-tailed
distribution; it is a five-point envelope. 0.6.188 closes the
right edge of that envelope, and 0.6.186 already closed the left
edge. The midpoint AM is now the only one of the five that could
still pretend to be the answer in isolation, and it stops being able
to pretend the moment any of its four siblings disagrees with it by
more than a factor of two.

On the test vector, every sibling disagrees with AM by more than a
factor of two. That is the whole release.
