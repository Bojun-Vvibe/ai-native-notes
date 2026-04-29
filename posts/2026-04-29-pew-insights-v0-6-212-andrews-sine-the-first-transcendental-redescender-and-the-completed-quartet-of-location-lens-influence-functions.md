# pew-insights v0.6.212: Andrews sine, the first transcendental redescender, and the completed quartet of location-lens influence functions

`pew-insights v0.6.212` shipped today — `2026-04-29` — adding
`source-row-token-m-estimator-andrews`, the **fourth** robust
M-estimator in the location-lens suite and the **first** built on a
transcendental influence function. With Andrews now alongside Huber
(`v0.6.209`), Tukey biweight (`v0.6.210`), and Hampel (`v0.6.211`),
the suite has a complete four-way contrast covering the canonical
shapes a robust statistician reaches for: monotone-bounded,
smooth-redescending-polynomial, piecewise-linear-redescending, and
smooth-redescending-transcendental. This post walks through why
Andrews is mechanically distinct from the three already in the suite,
what its **3.1 % – 18.1 % rejection-rate range** across the live-smoke
fleet tells us about per-source tail behavior, and why the choice of
the canonical tuning constant `A = 1.339` matters more than it might
look.

## The four-shape taxonomy of robust location

A robust M-estimator of location solves
`sum_i psi((x_i - mu) / s) = 0` for `mu`, given a scale `s` (here
`MAD/0.6745`) and an influence function `psi`. The shape of `psi`
determines everything about the estimator's behavior on contaminated
data. Four shapes recur in the textbooks, and they map cleanly onto
what `pew-insights` now ships:

1. **Monotone-bounded** — `psi` rises linearly to a clip threshold
   then stays flat at `+- c` forever. The mean has unbounded `psi`
   (the identity); Huber's `psi` is identity on `|z| <= c` and `+- c`
   beyond. Tail influence is **bounded but never zero**. This is
   `v0.6.209`, with `c = 1.345`.

2. **Smooth-redescending-polynomial** — `psi` rises, peaks, then
   smoothly descends to exactly zero at a finite rejection point
   `+- c`. Tukey biweight uses the cubic `z * (1 - (z/c)^2)^2`,
   tangent to zero at `+- c`. Tail influence is **eventually zero, on
   a polynomial schedule**. This is `v0.6.210`, with `c = 4.685`.

3. **Piecewise-linear-redescending** — `psi` is a sequence of
   straight-line segments with corners at `+- a`, `+- b`, `+- c`, an
   inner *plateau* on `(a, b]` where `psi` equals the constant
   `+- a`, and a linear ramp down to zero on `(b, c]`. Hampel
   `(a, b, c) = (1.7, 3.4, 8.5)` is the canonical choice. Tail
   influence is **eventually zero, with corners**. This is `v0.6.211`.

4. **Smooth-redescending-transcendental** — `psi` is the
   *sinusoidal* `A * sin(z / A)` on `|z| <= A * pi`, zero outside.
   Tail influence is **eventually zero, on a half-cosine arch**, and
   `psi` is **C-infinity smooth on its support**. This is `v0.6.212`,
   with `A = 1.339`.

Before today, three of the four cells in this taxonomy were filled.
Today the fourth — the only one whose `psi` is not a polynomial — got
filled, and the choice was deliberate: Andrews is the canonical
transcendental redescender in the M-estimator literature precisely
because **no polynomial of any finite degree can mimic the
half-cosine arch shape** that Andrews uses to taper rejection.

## Why "transcendental" is not just an adjective

It is tempting to dismiss the polynomial-vs-transcendental distinction
as cosmetic — both Tukey and Andrews redescend smoothly to zero,
both have a single peak in `psi`, both reach zero at a finite
threshold. But there is a real, computable difference, and it shows
up at the **derivative**.

- Tukey's `psi(z) = z * (1 - (z/c)^2)^2` has derivative
  `psi'(z) = (1 - (z/c)^2) * (1 - 5*(z/c)^2)`. This goes to zero at
  `z = c/sqrt(5) ≈ 0.447 c` (the location of the peak in `|psi|`)
  and then turns negative; `psi` reaches its peak at
  `|z| ≈ 0.447 c ≈ 2.094` for `c = 4.685`, then descends as a
  quartic-times-something polynomial back to zero at `|z| = c`.

- Andrews's `psi(z) = A * sin(z / A)` has derivative
  `psi'(z) = cos(z / A)`. This goes to zero at `z = A * pi/2` —
  exactly the location where `sin(z/A)` reaches its maximum of `1` —
  and then turns negative; `psi` peaks at `|z| = A * pi/2 ≈ 2.103`
  for `A = 1.339`, then descends as a half-cosine arch back to zero
  at `|z| = A * pi ≈ 4.207`.

The peaks are at almost identical MAD-positions (Tukey at 2.094,
Andrews at 2.103), but the **rejection thresholds differ**: Tukey
rejects at `c = 4.685`, Andrews at `A * pi ≈ 4.207`. Andrews is
**more aggressive per MAD-unit** on the tail, by a ratio of
`4.685 / 4.207 ≈ 1.114`. On a row 4 MAD-units out, Tukey still
assigns a small but nonzero `psi`; Andrews has already begun the
sinusoidal descent toward zero and is much closer to fully rejecting
the row.

The other property that matters: Andrews's `psi` is **infinitely
differentiable** on its support — every derivative is another
sinusoid. Tukey's `psi` is C-infinity globally too, but its
**fourth derivative changes sign** at well-defined polynomial roots,
which means certain higher-moment expansions of the IRLS recursion
behave qualitatively differently. For diagnostic plotting, the
practical difference is: Andrews's weight curve `psi(z) / z =
A * sin(z/A) / z = sinc(z/A)` is the **sinc function** scaled by a
factor — a shape every signal-processing engineer already has a
mental picture of. Tukey's weight curve is a polynomial bump.

## Live-smoke against `~/.config/pew/queue.jsonl`

The `v0.6.212` CHANGELOG ships a six-source live-smoke run against
the standing queue file — **1,913 rows across 6 sources**, default
tuning `A = 1.339`. The full table:

```
source        rows  mean         median      andrews     mad         scale       iter  core  descend  rejected  andrews-mean   andrews-median
codex         64    12650385.31  7132861.00  8997876.58  6506096.00  9645952.36  16    55    7        2          -3652508.73   +1865015.58
opencode      425   10412409.48  7958860.00  7357641.66  4652994.00  6898539.23  12    386   16       23         -3054767.82    -601218.34
claude-code   299   11512995.95  3319967.00  3371623.92  3103438.00  4601164.06  12    223   22       54         -8141372.03      +51656.92
openclaw      531    3738273.68  2325102.00  2575519.65  1334954.00  1979205.76  15    458   48       25         -1162754.03    +250417.65
hermes        261     760488.25   432218.00   435386.11   264040.00   391466.29  14    211   29       21          -325102.14      +3168.11
vscode-XXX    333       5662.84     2319.00     2496.29     1736.00     2573.80  13    282   24       27            -3166.55       +177.29
```

Three things in this table deserve unpacking.

**Convergence.** All six sources converged in **12 to 16 IRLS
iterations**, with the heaviest tail (`codex`) requiring the most
iterations (16). For comparison, monotone Huber on the same data
typically converges in 5-8 iterations because its `psi` does not
redescend and so its weight surface has no local extrema. Andrews
needs more iterations because the weight surface
`w(z) = A * sin(z/A) / z` has a peak at `z = 0`, descends through
`w = 0` at `z = A * pi`, and so the IRLS sequence has to navigate a
non-monotone weight landscape. 16 iterations is still cheap — a few
hundred microseconds per source on the live-smoke sizes — and the
fact that the fixed-point converges at all on every source confirms
that the IRLS implementation is well-conditioned for this fleet.

**Rejection-rate range: 3.1 % to 18.1 %.** Per-source rejection
rates land at:

- `codex`: 2/64 = **3.1 %** rejected
- `opencode`: 23/425 = **5.4 %** rejected
- `openclaw`: 25/531 = **4.7 %** rejected
- `hermes`: 21/261 = **8.0 %** rejected
- `vscode-XXX`: 27/333 = **8.1 %** rejected
- `claude-code`: 54/299 = **18.1 %** rejected

The 6x spread between `codex` (3.1 %) and `claude-code` (18.1 %) is
the most striking thing in the table, and it does not match what a
naive look at the medians would suggest. `codex` has the **largest
median** of any source (`7,132,861` tokens) and the **second-largest
mean** (`12,650,385`); `claude-code` has a much smaller median
(`3,319,967`) but the **largest mean** (`11,512,995`). The reason
`claude-code` has the highest rejection rate is precisely the
`mean - median` gap: claude-code's mean is **3.47x its median**,
implying a heavy upper tail of rows that Andrews recognizes as
extreme (`|z| > A*pi`) and zeros out. `codex`'s mean is `1.77x` its
median — a heavy tail too, but proportionally less extreme, and on
only 64 rows. The rejection-rate ranking is therefore a direct
read-out of **which source's per-row token distribution has the
heaviest right tail relative to its body**, not a read-out of which
source uses the most tokens.

**The `andrews - mean` gap is always negative; the `andrews - median`
gap straddles zero.** This is the structural signature of a
redescending M-estimator on heavy-tailed data:

- `andrews - mean`: codex `-3.65 M`, opencode `-3.05 M`, claude-code
  `-8.14 M`, openclaw `-1.16 M`, hermes `-325 K`, vscode-XXX `-3.2
  K`. **Always negative**, and the largest magnitude (`-8.14 M` on
  claude-code) is on the source with the highest rejection rate.
  This is mechanical: the mean is being pulled up by exactly the
  rows Andrews zeros out, so removing those rows necessarily moves
  `mu` down.

- `andrews - median`: codex `+1.87 M`, opencode `-601 K`,
  claude-code `+52 K`, openclaw `+250 K`, hermes `+3.2 K`,
  vscode-XXX `+177`. **Straddles zero**, with the magnitudes much
  smaller (max `+1.87 M` on codex, which is a `+26 %` move on a
  median of `7.13 M`; min `-601 K` on opencode, `-7.5 %`). Andrews
  lives near the median for these distributions, never far. This is
  the hallmark of a high-breakdown estimator: it tracks the median
  closely on the body, and only diverges from it when the body is
  itself asymmetric enough to pull the IRLS fixed point off the
  median.

The codex `+1.87 M` is the most interesting outlier inside this
pattern. With only 64 rows, `codex`'s body distribution has more
sample variance than the larger sources, and Andrews's IRLS finds a
fixed point that pulls `mu` significantly above the median. On 425
rows (opencode) or 531 rows (openclaw), the body is dense enough
that the fixed point lands much closer to the median.

## Andrews vs Hampel: a side-by-side on identical data

The `v0.6.211` Hampel changelog ran the same six-source live-smoke;
comparing the two estimators on identical input is the cleanest way
to see the polynomial-vs-transcendental contrast operationally.

Hampel uses knots `(a, b, c) = (1.7, 3.4, 8.5)`. Its rejection
threshold is **`c = 8.5` MAD-units**. Andrews's rejection threshold
is **`A * pi ≈ 4.207` MAD-units**. So per MAD-unit, Andrews is
**roughly twice as aggressive** about declaring rows rejected.
On `claude-code`, Hampel rejected fewer rows than Andrews because
Hampel's `c = 8.5` lets a long stretch of moderately extreme rows
through with linearly decaying weight; Andrews's `A * pi = 4.207`
zeroes them out entirely. The two estimators land on **similar
`mu` values** (both near the median), but the **residual partition
is very different**: Hampel keeps more rows alive at low weight;
Andrews kills them.

The practical consequence: if you want a robust location estimate
that is **maximally insensitive to a heavy upper tail** (e.g., the
claude-code distribution), Andrews is the right pick from the suite.
If you want a robust location estimate that **uses information from
moderate outliers but downweights them sharply**, Hampel is the
right pick. Tukey sits in between, with a polynomial taper that is
gentler than Andrews's sinusoid but more aggressive than Hampel's
piecewise-linear ramp.

## The three-bucket residual partition

Andrews's per-source report includes a unique three-bucket residual
partition:

- `coreRows` — `|z| <= A * pi/2`, the rising half of the sine arch,
  with weight in `[2/pi, 1]`
- `descendingRows` — `A * pi/2 < |z| <= A * pi`, the falling half,
  with weight in `(0, 2/pi)`
- `rejectedRows` — `|z| > A * pi`, weight exactly zero

with `core + descend + reject = n` exactly. The fleet partition:

- codex: `55 / 7 / 2` (86 % core, 11 % descending, 3 % rejected)
- opencode: `386 / 16 / 23` (91 % / 4 % / 5 %)
- claude-code: `223 / 22 / 54` (75 % / 7 % / 18 %)
- openclaw: `458 / 48 / 25` (86 % / 9 % / 5 %)
- hermes: `211 / 29 / 21` (81 % / 11 % / 8 %)
- vscode-XXX: `282 / 24 / 27` (85 % / 7 % / 8 %)

The `core` percentage tells you what fraction of the data lives in
the **fully-trusted** region of the sine arch, where weight is at
least `2/pi ≈ 0.637`. The `descending` fraction is rows in the
**partially-trusted** region — they contribute to `mu` but at
diminishing weight. The `rejected` fraction is rows the estimator
treats as if they did not exist.

The most interesting cross-source contrast is again `claude-code`:
only 75 % of its rows are in the core, the lowest of any source, and
18 % are rejected outright. Compare with `opencode` (91 % core, 5 %
rejected): two sources of comparable size (425 and 299 rows) with
very different residual partitions. The opencode body fits the
Andrews `psi` shape comfortably; the claude-code body has so many
extreme rows that almost a fifth of the data ends up at zero
weight.

## Why `A = 1.339` (and not `A = 1.5` or `A = 2`)

The canonical Andrews tuning `A = 1.339` is chosen to give
**~95 % asymptotic relative efficiency at the normal distribution**.
Equivalently, if the data really were normal, using Andrews instead
of the mean would cost you only about 5 % of the precision you would
get from the optimal estimator. This is the same efficiency target
Huber's `c = 1.345` and Tukey's `c = 4.685` are tuned to hit,
which makes the four estimators **directly comparable on a fair
footing**: each is the canonical 95 %-efficient version of its
shape family.

If you raise `A`, you push the rejection threshold further out
(`A = 2` gives `A * pi ≈ 6.28` MAD-units) and Andrews becomes more
like Tukey or Hampel — gentler about rejection, closer to the mean.
If you lower `A` to `1.0`, the rejection threshold drops to
`pi ≈ 3.14` MAD-units and the estimator becomes more like a trimmed
median, throwing away anything beyond about 3 MAD-units. The flag
`--tuning <f>` in `pew-insights source-row-token-m-estimator-andrews`
exposes this knob; the default `1.339` is the textbook choice and
the right starting point for fleet-level diagnostics.

## What `v0.6.212` adds to the suite contract

With Andrews shipped, the location-lens suite now exposes a complete
**four-shape contract** for robust location: pick your `psi` shape
family by what you believe about your data's tail. The four sub-
commands now read like a menu:

- `m-estimator-huber` — bounded but nonzero tail influence
- `m-estimator-tukey` — polynomial taper to zero
- `m-estimator-hampel` — piecewise-linear taper with a plateau and
  corners
- `m-estimator-andrews` — sinusoidal taper, smoothest of the
  redescenders

Every one of them ships with the same fleet diagnostic columns
(`rows`, `mean`, `median`, `<estimator>`, `mad`, `scale`, `iter`,
plus partition columns specific to each shape) and the same
`<estimator>-mean` and `<estimator>-median` gap columns, so a fleet
operator can run all four and read off the `mu` agreement (a
confidence-building check: if all four redescenders land on
similar `mu` for a given source, the body is genuinely well-defined
and the disagreement is purely about how to handle the tail).

The next axis to fill is not in the *shape* of `psi` — that
taxonomy is essentially saturated by the four already shipped — but
in the *scale* `s` used inside `psi(z) = psi((x - mu) / s)`. Today
the suite uses `s = MAD / 0.6745` for every estimator, the standard
robust scale. A future `v0.6.213` could switch to **Qn** or **Sn**
(Rousseeuw-Croux scale estimators) and re-run the four `psi`
families; this would let an operator separate "the location
estimate moved because the `psi` shape changed" from "the location
estimate moved because the `s` estimate changed". Both moves are
real, both are diagnostically useful, and the suite contract is now
clean enough to support that orthogonal axis without disturbing the
shape contract.

For now, `v0.6.212` is the version that **completes the canonical
quartet** — Huber, Tukey, Hampel, Andrews — and gives the
location-lens suite a footprint identical to what you would expect
in any robust-statistics textbook chapter on M-estimators. The
1,913-row live-smoke shows it works: every source converges, the
rejection rates make physical sense, and the `andrews - mean` /
`andrews - median` gap pattern matches the textbook signature of a
high-breakdown redescender on heavy-tailed real-world fleet data.
