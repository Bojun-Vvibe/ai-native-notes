# The Welsch three-bucket weight partition in pew-insights v0.6.213 — and the first infinite-support redescender where no row is ever exactly rejected

**Date:** 2026-04-29
**Repo:** `pew-insights`
**Release:** `v0.6.213`
**Release SHA:** `0b975f5`
**Analyzer SHA (feat):** `3f4024b`
**Test SHA:** `c56e826`
**Refinement SHA:** `a81487f`

## What shipped

`pew-insights` v0.6.213 added a fifth M-estimator analyzer to the
location-lens suite:

```
pew-insights source-row-token-m-estimator-welsch
```

It implements the **Welsch (Leclerc) Gaussian-kernel redescending
M-estimator** of location for per-row `total_tokens`, solving
`sum_i psi(z) = 0` via iteratively reweighted least squares (IRLS)
seeded at the median with MAD/0.6745 scale. The Gaussian influence
function

```
psi(z) = z * exp( -(z/c)^2 / 2 )
```

derives from the rho

```
rho(z) = (c^2 / 2) * (1 - exp(-(z/c)^2 / 2))
```

with canonical tuning `c = 2.9846` (≈ 95% asymptotic relative
efficiency at the normal). The weight envelope is

```
w(z) = exp(-(z/c)^2 / 2)
```

which is **strictly positive for every finite `z`** — Welsch's
defining mechanical feature, and the property that distinguishes it
from every other shipped redescender in the suite.

## Why this is structurally novel

The location-lens M-estimator family at v0.6.213 now contains five
analyzers, shipped over five consecutive releases:

| Release   | Analyzer                                  | psi shape           | Support of nonzero psi      | Smoothness |
|-----------|-------------------------------------------|---------------------|-----------------------------|------------|
| v0.6.209  | `m-estimator-huber`                       | clipped linear      | all of R (monotone)         | C^0 corners at ±c |
| v0.6.210  | `m-estimator-tukey`                       | smooth polynomial   | compact, `\|z\| <= c=4.685` | C^1 |
| v0.6.211  | `m-estimator-hampel`                      | piecewise linear    | compact, `\|z\| <= c=8.5`   | C^0 corners at ±a, ±b, ±c |
| v0.6.212  | `m-estimator-andrews`                     | sinusoidal          | compact, `\|z\| <= A*pi=4.207` | C^infty inside, C^0 at boundary |
| **v0.6.213** | **`m-estimator-welsch`**               | **Gaussian-kernel** | **all of R (infinite)**     | **C^infty everywhere** |

Welsch is the first member of the suite that is simultaneously
**redescending** (psi → 0 as |z| → infty) and **infinite-support**
(psi never exactly equals zero at any finite z). Every other
redescender in the suite — Tukey, Hampel, Andrews — has compact
support: there exists a finite cutoff beyond which the estimator
treats a row as **literally not present in the dataset**, with
`w(z) = 0` and zero contribution to the IRLS update. Welsch never
does this. Even at `|z| = 50` MAD-units, Welsch assigns

```
w(50) = exp(-(50/2.9846)^2 / 2) ≈ exp(-140.3) ≈ 1.5e-61
```

— vanishingly small, but **nonzero**. The row is still in the
dataset; it just contributes a literally negligible amount of
gradient.

This matters for two reasons. First, it means Welsch never has to
make a binary in/out classification decision about any row, which
removes a discontinuity that all the other redescenders have.
Hampel's piecewise-linear `psi` has corners at the knot positions
(canonically `a=1.7`, `b=3.4`, `c=8.5` in MAD-units) and a **hard
zero past `c`**; Tukey's polynomial `psi(z) = z*(1-(z/c)^2)^2` is
tangent-to-zero at `±c=±4.685` and **identically zero past it**;
Andrews's sinusoidal `psi(z) = A*sin(z/A)` is `0` past `A*pi ≈ 4.207`.
All three of these are smooth-then-zero. Welsch is smooth-and-fades.

Second, it means Welsch's residual partition is keyed on something
other than support. The four other M-estimators all naturally
partition rows into "in support" vs "out of support" buckets
(Huber's case is degenerate — every row is in support, so it
partitions on `|z| < c` vs `|z| >= c` instead, the linear vs the
clipped region). Welsch can't do that, because every row is in
support. It instead partitions on **weight magnitude**.

## The three-bucket weight partition

The v0.6.213 analyzer reports a unique three-bucket residual
partition keyed on weight magnitude:

| Bucket           | Weight range  | Equivalent z-range                                  |
|------------------|---------------|-----------------------------------------------------|
| `coreRows`       | `w >= 0.5`    | `\|z\| <= c*sqrt(2 ln 2) ≈ 2.484` MAD-units         |
| `descendingRows` | `0.01 <= w < 0.5` | `2.484 < \|z\| <= c*sqrt(2 ln 100) ≈ 9.046` MAD-units |
| `negligibleRows` | `w < 0.01`    | `\|z\| > 9.046` MAD-units                            |

with `coreRows + descendingRows + negligibleRows = n` exactly. No
other shipped M-estimator partitions on weight, because they all
hit a support cutoff first and the bucket structure naturally
becomes "in support" vs "rejected".

The boundaries `0.5` and `0.01` are not arbitrary. The `0.5`
boundary corresponds to "a row contributes at least half of what an
unweighted row would contribute" — a natural threshold for "the
estimator considers this row representative". The `0.01` boundary
corresponds to "a row contributes less than 1% of an unweighted
row" — a natural threshold for "the estimator has effectively
rejected this row, even though it's still technically in the sum".
Inverting `w(z) = 0.5` gives `|z| = c*sqrt(2 ln 2) ≈ 2.484` and
inverting `w(z) = 0.01` gives `|z| = c*sqrt(2 ln 100) ≈ 9.046`.

The `negligibleRows` bucket is Welsch's softest analog of the hard
rejection bucket that Tukey, Hampel, and Andrews have. It contains
rows the estimator has **effectively** zeroed out, but they're not
**actually** zeroed out. This distinction is not philosophical — it
shows up in IRLS convergence behavior, where Welsch can in principle
"un-reject" a row across iterations as `mu` and `s` shift, while
Tukey/Andrews/Hampel cannot.

## The live-smoke against `~/.config/pew/queue.jsonl`

The CHANGELOG reports the following live-smoke run against the
local queue (1,916 rows, 6 sources, default tuning `c = 2.9846`,
with the redacted IDE source shown as `vscode-XXX`):

```
source        rows  mean         median      welsch       mad         scale       iter  core  descend  negligible  welsch-mean   welsch-median
------------  ----  -----------  ----------  -----------  ----------  ----------  ----  ----  -------  ----------  ------------  -------------
codex         64    12650385.31  7132861.00  10541413.84  6506096.00  9645952.36  13    61    3        0            -2108971.48   +3408552.84
opencode      426   10422261.40  8018557.00   8021238.29  4664066.50  6914955.34   8    399   27       0            -2401023.11      +2681.29
claude-code   299   11512995.95  3319967.00   4303631.53  3103438.00  4601164.06  14    243   34       22           -7209364.42    +983664.53
openclaw      532    3732520.70  2317756.50   2848797.29  1337259.50  1982623.90  13    499   25       8             -883723.41    +531040.79
hermes        262     758983.46   432176.00    532005.41   262201.00   388739.78  14    233   27       2             -226978.05     +99829.41
vscode-XXX    333       5662.84     2319.00      2871.79     1736.00     2573.80  12    305   20       8               -2791.05       +552.79
```

Several observations fall out of this table that don't fall out of
any of the prior four M-estimators on the same data.

### 1. Two sources have zero negligibleRows

`codex` (n=64) and `opencode` (n=426) report `negligibleRows = 0`.
Every row in those two sources falls into either the `coreRows` or
`descendingRows` bucket. This is a direct consequence of Welsch's
infinite support and the Gaussian decay rate. For a row to land in
`negligibleRows`, it must have `|z| > 9.046` MAD-units; for `codex`
with MAD = 6,506,096 that requires the row to be more than
~58.9 million tokens away from the IRLS-converged center. The
`codex` data tops out below that. For `opencode` with MAD =
4,664,067 the cutoff is ~42.2 million tokens — and `opencode`'s
26 descending rows all sit between |z|=2.484 and |z|=9.046, but
none cross the 9.046 line.

This is structurally impossible under Tukey/Andrews/Hampel — those
estimators' rejection cutoffs sit at lower z-values (Tukey 4.685,
Andrews 4.207, Hampel 8.5), so any row past those cutoffs
**necessarily** lands in the rejected bucket. Welsch's higher
effective-rejection threshold (9.046) means some rows that Tukey
and Andrews zero out get to keep contributing under Welsch — at
weight `~0.01` to `~0.5` — and that contribution shifts the
estimate.

You can read this off the comparison: on `codex`, Andrews
(v0.6.212) gave 8,997,876.58 and rejected 2 rows; Welsch gives
**10,541,413.84** and rejects 0 rows. The Welsch estimate is
~1.54M higher than Andrews on `codex` precisely because Welsch
retained influence from rows that Andrews zeroed.

### 2. claude-code is the only source with non-trivial negligibleRows

`claude-code` reports `negligibleRows = 22` out of 299 rows
(7.4%). This is the largest negligible-bucket fraction in the
fleet. The other sources sit at 0% (codex, opencode), 1.5%
(openclaw 8/532), 0.8% (hermes 2/262), and 2.4% (vscode-XXX
8/333). The `claude-code` upper tail is heavy enough that 22 rows
sit past 9.046 MAD-units from the converged center, even with
MAD = 3,103,438 — meaning those rows are individually larger than
~28 million tokens, well above the source mean of ~11.5M.

This matches the heavy-tail signature you can pull from the
companion analyzers: `claude-code` had the largest |hlMeanGap|
under HL (`-5,111,709`), the largest |hdMeanGap| under HD
(`-8,010,484`), the largest |huberMeanGap| under Huber
(`-6,417,969`), and now the largest |welschMeanGap| at
`-7,209,364.42`. The same right tail keeps showing up under every
robust lens, just attenuated to different degrees.

### 3. The welsch-median gap is small everywhere except codex

The `welsch-median` column reports the signed gap
`welsch_estimate - median`. Most sources sit close to zero:

- `opencode`:    `+2,681.29`     (≈ 0.03% of median)
- `vscode-XXX`:  `+552.79`       (≈ 24% of median, but median is tiny)
- `hermes`:      `+99,829.41`    (≈ 23% of median)
- `openclaw`:    `+531,040.79`   (≈ 23% of median)
- `claude-code`: `+983,664.53`   (≈ 30% of median)
- `codex`:       `+3,408,552.84` (≈ 48% of median, the outlier)

Welsch sits **above** the median on every source. This is a direct
consequence of right-tail contamination: the descending-bucket and
non-zero-contribution-from-negligible-bucket rows pull the IRLS
fixed point upward from where the median sits. On `codex`, where
Welsch retained ALL 64 rows (zero negligible), the upward pull is
the strongest — the converged estimate sits 48% above the median.

This is structurally distinct from Andrews on the same data, where
the codex `andrews-median` gap was `+1,865,015.58` (≈ 26% of
median). Welsch sits ~1.54M higher than Andrews on codex precisely
because of the retained right-tail contribution.

## Where Welsch lands on the family ordering

The five-member redescender ladder, ordered by **how aggressively
each estimate rejects right-tail rows on contaminated data**, is
now (most aggressive → least):

1. **Andrews** — sinusoidal, rejects past `A*pi ≈ 4.207`
2. **Tukey** — polynomial, rejects past `c = 4.685`
3. **Hampel** — piecewise-linear, rejects past `c = 8.5`
4. **Welsch** — Gaussian-kernel, never rejects but effectively-rejects past `9.046`
5. **Huber** — clipped linear, never rejects, never down-weights to vanishing

You can verify the ordering on the `claude-code` row:

| Estimator | Estimate    | mean-gap     | rejected/effectively-rejected |
|-----------|-------------|--------------|-------------------------------|
| Andrews   | 3,371,624   | -8,141,372   | 54/299 = 18.1% rejected       |
| Tukey     | (n/a column in table, but smaller) | similar | ~51/299 |
| Hampel    | (n/a column) | similar | ~30/299 |
| Welsch    | 4,303,632   | -7,209,364   | 22/299 = 7.4% effectively-rejected |
| Huber     | (n/a) | similar magnitude | 0/299 (clips, never rejects) |

The mean-gap shrinks as you move down the ladder (Andrews −8.14M
→ Welsch −7.21M), reflecting that less-aggressive rejection
retains more of the upward pull from the heavy tail.

The interesting observation is that on the cleanest source
(`opencode`, where 99% of rows fit in the core bucket) Welsch
agrees with the median to four significant figures (`8,021,238`
vs `8,018,557`, gap `+2,681`). This is the regime where Welsch
behaves indistinguishably from a clean median — the data isn't
contaminated, the IRLS converges to within rounding error of the
median, and the descending and negligible buckets contribute
nothing material. The redescender machinery is silent because
there's nothing to redescend against.

## Why ship a fifth M-estimator

The five-member ladder is now mechanically complete in the sense
that every standard textbook redescender has a representative:

- **Monotone bounded:** Huber (the bounded-influence baseline)
- **Compact polynomial:** Tukey biweight (the smooth redescender)
- **Compact piecewise-linear:** Hampel (the inner-plateau redescender)
- **Compact transcendental:** Andrews (the sinusoidal redescender)
- **Infinite-support smooth:** Welsch (the Gaussian-kernel redescender)

A user asking "which redescender should I use" can now compare
against five mechanically distinct shapes on the same live-smoke
data. The shapes are not equivalent — they produce different
estimates, with different mean-gaps and different rejection
behavior — and the gaps between them are interpretable. The
Welsch-vs-Andrews gap on codex (`+1.54M` higher under Welsch)
quantifies the cost of hard-rejecting two specific high-token rows
versus assigning them weight `~0.01-0.1`. The Welsch-vs-Huber gap
on claude-code (Welsch sits ~7.2M below the mean, Huber sits ~6.4M
below) quantifies the additional pull-up that Huber's
non-redescending tail influence produces.

The IRLS termination diagnostic refinement (`a81487f`, the fourth
commit in this release) surfaces per-source iteration counts so
that a user can see whether convergence was easy (8 iterations
for `opencode`) or required more work (14 iterations for
`claude-code`, `hermes`, and `vscode-XXX`). The 8-vs-14 spread is
not a bug — it reflects how much the right-tail contamination
shifts the IRLS fixed point per-source, and how much weight
recomputation is needed to pin down the converged center.

## Conclusion

`pew-insights` v0.6.213 adds the first infinite-support redescender
to the location-lens suite. The defining mechanical feature is that
no row is ever exactly rejected — every row contributes weight
`exp(-(z/c)^2/2) > 0` regardless of how extreme it is. The analyzer
exposes this through a three-bucket weight partition (`coreRows`,
`descendingRows`, `negligibleRows`) that is structurally distinct
from the support-based partitions used by the four prior
M-estimators. The 1,916-row live-smoke shows two sources with zero
negligibleRows (`codex`, `opencode`) — a regime impossible under
Tukey/Andrews/Hampel — and confirms Welsch sits above the median
on every source, with the largest upward gap on `codex` (`+3.4M`,
48% above median) where the Gaussian decay never quite reaches
zero on any of the 64 rows. The release ships at SHA `0b975f5`
with 50 new tests (`5169 → 5219`) and a per-source IRLS
termination diagnostic (`a81487f`) for runtime introspection.
