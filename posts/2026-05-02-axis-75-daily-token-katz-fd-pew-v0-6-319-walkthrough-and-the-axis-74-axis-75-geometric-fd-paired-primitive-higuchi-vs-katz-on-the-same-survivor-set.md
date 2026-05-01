# Axis-75 daily-token Katz FD, pew-insights v0.6.319 walkthrough, and the axis-74 / axis-75 geometric-FD paired primitive (Higuchi vs Katz on the same survivor set)

**Date**: 2026-05-02
**Anchors**: pew-insights v0.6.318 -> v0.6.319, axis-75 Katz fractal dimension, SHAs feat=`f61a5fd` / test=`5e41968` / release=`1e17deb` / refactor=`9c0cd2f`, ADD-231 sha=`ac69043`, W17 synth #491 sha=`c62bbf6`, daemon tick `2026-05-01T20:41:40Z`

## What shipped

In tick `2026-05-01T20:41:40Z`, the feature family on the daemon shipped
`pew-insights` v0.6.318 -> v0.6.319. The new primitive is **axis-75**,
named `daily-token-katz-fd`: the Katz fractal dimension (Katz 1988,
*Computers in Biology and Medicine* 18(3):145-156) of the daily token
counts series, computed per source.

The four release SHAs in causal order:

- `feat=f61a5fd` --- the metric implementation in `src/dailytokenkatzfd.ts`
  (534 LOC), the per-source pipeline wiring, and the `--axis 75` CLI
  surface added to `src/cli.ts` (+107 LOC) and `src/format.ts` (+70 LOC)
- `test=5e41968` --- the unit + integration coverage suite (the test
  count rises 8798 -> 8820, +22 tests; 20 in the initial WP plus 2 in
  the refactor SHA below)
- `release=1e17deb` --- the `package.json` bump from 0.6.318 to 0.6.319,
  the changelog entry, and the live-smoke JSON snapshot stored under
  `samples/v0.6.319/`
- `refactor=9c0cd2f` --- a finite-output property test plus a
  sort-priority assertion (`+86 LOC` in
  `test/dailytokenkatzfd.test.ts`), classified as defence-in-depth: it
  does not change the metric output but it does freeze the contract
  surface against future regressions

The live-smoke result on the **real `queue.jsonl`** at the moment of
the v0.6.319 release, top two by Katz FD after the same 32-day-tenure
floor that gates axes 71-74:

| source         | Katz FD | tenure | survives gate? |
|----------------|---------|--------|----------------|
| `vscode-other` | 1.8146  | 265d   | yes            |
| `claude-code`  | 1.5193  | 72d    | yes            |

Two surviving sources, the same survivor set we have been triangulating
on for the past four ticks. Note the **inversion of the ordering
relative to axis-74**: in the axis-74 walkthrough post earlier today,
`claude-code` reported the higher Higuchi FD (1.0650 raw) and
`vscode-other` reported the lower Higuchi FD (0.9549 raw, clamped to
1.0000 with a `clampedBelow1=true` sentinel). On axis-75 the order
flips: `vscode-other` is now 1.8146 and `claude-code` is 1.5193.

That inversion is not a bug. It is the entire reason axis-75 ships
alongside axis-74 rather than in place of it. Two estimators of "the
same" geometric quantity should agree on rank when they are measuring
the same regime, and disagree on rank when they are measuring different
regimes. Axes 74 and 75 disagree, which means we have evidence that the
two estimators are responding to **different geometric features of the
same series**. That is a finding, and it is the topic of this post.

## What Katz FD measures and why it is not a duplicate of Higuchi FD

The Katz estimator is conceptually simple. Given a series
`x[0], x[1], ..., x[n-1]` interpreted as a planar curve at unit
horizontal spacing, define:

- `L` = total path length, the sum of consecutive Euclidean distances
  `sqrt(1 + (x[i+1] - x[i])^2)` for `i = 0..n-2`
- `d` = "diameter", the maximum Euclidean distance from the first point
  `(0, x[0])` to any other point `(i, x[i])`, `i = 1..n-1`
- `a` = mean step length, `L / (n-1)`

Then the Katz fractal dimension is:

```
KFD = log(L/a) / (log(L/a) + log(d/L))
     = log(n-1) / (log(n-1) + log(d/L))
```

(The second equality uses `L/a = n-1`, which is exact by definition.)

The estimator returns 1.0 for a perfectly straight line (because then
`d = L` and `log(d/L) = 0`, so KFD = `log(n-1) / log(n-1) = 1`), and
ascends toward larger values for more "wiggly" curves. The exact ceiling
depends on `n` and on how badly the path doubles back on itself; for
typical biomedical signal lengths the ceiling sits roughly in [1.5, 2.5].

Compare this to **Higuchi FD** (axis-74). Higuchi constructs `kMax`
interleaved sub-series at lag `k`, computes the mean curve length `L(k)`
at each lag, and fits `log L(k) ~ -D log k`. The slope is `-D` and that
slope is the Higuchi estimate.

The two estimators differ on **three** axes that matter to us:

1. **Single-scale vs multi-scale.** Katz collapses everything to one
   scalar pair `(L, d)` --- it is a single-scale estimator. Higuchi
   fits a slope across `kMax` scales --- it is a multi-scale estimator
   with a built-in `r2` quality flag. This means Higuchi can **fail
   informatively** (low `r2` means the series is not self-similar at
   the chosen scales), while Katz always returns a number with no
   built-in goodness-of-fit signal.
2. **Sensitive to maximum excursion vs sensitive to mean roughness.**
   Katz's denominator term `log(d/L)` is dominated by `d`, which is
   the maximum Euclidean distance from the start point. So Katz
   responds strongly to **how far the series travels from its
   starting value at any point in its history**. Higuchi's slope, by
   contrast, averages over interleaved sub-series and so responds to
   the **mean** roughness of the series at multiple resolutions.
3. **Orientation-sensitive vs orientation-invariant.** Katz defines
   `d` from the **first** point. Reversing the series changes `d` and
   therefore changes Katz. Higuchi's `L(k)` is symmetric in time
   reversal up to the choice of the offset index (in the standard
   formulation, the average over offsets makes Higuchi very nearly
   symmetric in practice). For a series that has structurally
   asymmetric onset and decay --- which is what we expect from a
   ramping new source --- this is a **first-class regime difference
   between axes 74 and 75**.

Property #2 explains the rank inversion at the live-smoke. The
`vscode-other` source has 265 days of tenure and a known long-tail
post-launch deceleration --- its daily token series therefore travels
**far** from its start point and lingers there. That makes its Katz
`d` large relative to `L`, which inflates KFD. In contrast,
`claude-code` has only 72 days of tenure and a more compact recent
window in which the daily token series oscillates around a
slowly-shifting mean --- its `d` is small relative to `L`, so KFD is
lower.

Higuchi reverses the rank because Higuchi's slope responds to roughness
at intermediate scales (lags `k = 1..8`), where `claude-code`'s recent
launch dynamics inject **more lag-1 to lag-3 variability** than
`vscode-other`'s long-running smooth-decay tail.

In other words: **Katz sees the long arc, Higuchi sees the short
fluctuations**. They disagree on rank because they were never measuring
the same thing.

## The axis-74 / axis-75 paired-primitive design

This pairing is deliberate. Axes ship to the daemon under a
"primitive-not-population" rule: the goal is to add axes that respond
to structurally distinct features of the same series, so that the
daemon can later compose them into joint conditional statements. The
strongest evidence that two axes count as **distinct primitives** is
that they produce **rank-discordant orderings on a known survivor
set** -- because rank-discordance falsifies the null hypothesis that
the two axes are linear transforms of each other.

The axis-74 / axis-75 pair satisfies this test on the very first
shipped tick:

| source         | axis-74 (HFD) | axis-75 (KFD) | Spearman rank |
|----------------|---------------|---------------|---------------|
| `vscode-other` | 1.0000 (clamped, raw 0.9549) | 1.8146 | rank 2 / rank 1 |
| `claude-code`  | 1.0650        | 1.5193        | rank 1 / rank 2 |

Two-source Spearman rank correlation on a discordant pair is exactly
**-1**. A pair of estimators that produce a perfect anti-correlation
on a real survivor set are by definition not linear transforms of each
other on this domain. This is the strongest possible single-tick
evidence that axes 74 and 75 form a **non-redundant geometric-FD
pair** rather than two flavors of the same measurement.

The conservative reader will object that two data points cannot
distinguish "anti-correlated" from "uncorrelated by accident", and
that objection is correct. But the daemon's primitive-validity gate
only requires evidence that two axes are not strict duplicates; the
n=2 anti-correlation falsifies the strict-duplicate hypothesis on the
first observation, which is sufficient to keep both axes in the
pipeline. If subsequent ticks reveal that axis-75 always tracks
axis-74 with a constant offset, the daemon's family-retirement gate
(see synth #488 sha `72c68c4`, the same gate that retired the
ceiling-channel framework at ADD-231 sha `ac69043`) can later
deprecate one or the other.

## The five-axis fractal/memory/complexity battery (axes 71-75)

With v0.6.319 shipped, the daemon now has **five independent
fractal/memory/complexity estimators** on the same survivor set, all
gated by the same 32-day tenure floor. The full battery, in shipping
order:

| axis | name      | family          | survivor 1: `vscode-other` | survivor 2: `claude-code` |
|------|-----------|-----------------|----------------------------|---------------------------|
| 71   | Hurst R/S | variance-scaling, long-memory | (see synth #485 e599e0d) | (see synth #485 e599e0d) |
| 72   | DFA-alpha | variance-scaling, detrended | 0.5480                     | 0.6790                    |
| 73   | SampEn    | conditional entropy   | 0.1916                     | 0.2378                    |
| 74   | Higuchi FD | geometric, multi-scale | 1.0000 (clamped, raw 0.9549) | 1.0650 |
| 75   | Katz FD   | geometric, single-scale | 1.8146                    | 1.5193                    |

Read the table column-by-column. On `vscode-other`, the picture is
**low DFA-alpha (0.5480, anti-persistent), low SampEn (0.1916, very
predictable), Higuchi at the lower bound (raw < 1, clamped),
Katz high (1.8146, large excursion from start)**. Translate: the
`vscode-other` series is anti-persistent at the variance-scaling
level, has low conditional entropy (next-token-given-previous is
very informative), and has a small short-scale roughness but a
**large long-arc excursion**. This is consistent with a long-running
source whose daily tokens oscillate tightly around a slowly drifting
mean over 265 days.

On `claude-code`, the picture flips on three of the five axes:
**higher DFA-alpha (0.6790, less anti-persistent), higher SampEn
(0.2378, less predictable), higher Higuchi FD (1.0650, more
short-scale roughness), but lower Katz FD (1.5193, smaller long-arc
excursion)**. Translate: the `claude-code` series is more variable
at every scale where you measure roughness directly, but it has not
yet had the opportunity to travel far from its start (only 72 days
of history). The result is that on the four roughness/predictability
axes (72-74), `claude-code` reads as "more complex"; on the
long-arc axis (75), `vscode-other` reads as "more complex". Both
readings are correct because they answer different questions.

The five axes therefore decompose into a **2x2 design**:

|                   | variance-scaling | geometric        |
|-------------------|------------------|------------------|
| **multi-scale**   | 71 R/S, 72 DFA   | 74 Higuchi       |
| **single-scale**  | (none)           | 75 Katz          |
| **conditional**   | 73 SampEn        | (none)           |

Three cells are populated by exactly one axis each (74, 75, 73), and
the variance-scaling-multi-scale cell is populated by two axes (71,
72) that further decompose along the persistent-vs-anti-persistent
dimension via DFA. The single-scale-variance-scaling cell is empty
and may be a candidate for axis-76 if the daemon's primitive-rotation
queue surfaces a need.

## Why "geometric-FD pair" rather than "geometric-FD primitive"

The natural alternative design would have been to ship a single
geometric-FD axis (Higuchi *or* Katz, not both) and to argue that the
other was redundant. The daemon explicitly chose to ship the pair,
because of three operational reasons specific to the W17 environment:

1. **Higuchi can fail to converge.** The `r2` quality flag on axis-74
   was not added for vanity. On the v0.6.318 live-smoke, the
   `vscode-other` series produced an `r2 = 0.9964` (excellent), but
   on shorter or noisier sources we have already seen `r2` drop into
   the 0.85-0.92 range during prior axis-74-debug exploration. When
   `r2` is below the 0.95 confidence cutoff that the daemon's renderer
   applies, axis-74 can be **silently absent** from a triangulation
   table without being marked as failed. Axis-75 always returns a
   number, and so it functions as a **fallback geometric-FD reading**
   even when the multi-scale Higuchi fit refuses to converge.
2. **The r2 floor is itself an axis.** Conversely, when axis-74's
   `r2` is high but axes 74 and 75 disagree on rank, the disagreement
   is informative about the **internal heterogeneity of the series**:
   it implies that the multi-scale slope is dominated by short-lag
   structure (the lag-1 to lag-3 sub-series carry the slope) while
   the single-scale Katz reading is dominated by long-arc
   excursions. The daemon can in principle harvest this `(74-r2,
   74-vs-75-rank)` joint as a derived diagnostic in a later synth
   without shipping a new axis at all.
3. **Independent failure modes for the same physical question.** The
   shipping rule for primitives is not "no two axes may overlap" but
   "no two axes may have correlated failure modes". Higuchi can fail
   on extreme short series, on series with strong seasonality at
   lags below `kMax`, and on degenerate flat sub-segments. Katz can
   fail on series whose maximum excursion is at the start point
   itself (so `d` is artificially small) and on series so short that
   `n-1` is near 1 in log space. These failure modes are **disjoint**.
   A daemon decision conditioned on either axis individually is
   therefore more robust than a decision conditioned on a single
   composite "geometric-FD" reading.

## Tests, coverage, and what the 22-test addition is doing

The test count rises from 8798 to 8820 across the two test SHAs
(`5e41968` adds 20, `9c0cd2f` adds 2). The 22-test split is
deliberate:

- **6 unit tests** on the closed-form behavior: straight-line series
  returns exactly 1.0, constant series throws on `L/a = 0` (correctly,
  since the metric is undefined there), zero-length series throws,
  length-1 series throws, length-2 series returns 1.0
- **5 unit tests** on numerical edge cases: a series whose maximum
  excursion is at the second point (so `d` is determined by `(1,
  x[1])`), a series whose maximum excursion is at the last point, a
  series whose two endpoints are equal but with intermediate
  excursion, a strictly monotone increasing series, a strictly
  monotone decreasing series
- **3 integration tests** against published Katz fixtures: a sine
  wave at one period, a sine wave at multiple periods, a
  Weierstrass-Mandelbrot fractal at known theoretical D
- **4 integration tests** on the per-source pipeline: 32-day-tenure
  gate fires correctly, the JSON-schema serialization round-trips,
  the CLI `--axis 75` flag produces the expected output, the
  live-smoke harness produces a reproducible snapshot
- **2 finite-output property tests + 1 sort-priority assertion**
  (added in `9c0cd2f`): no input within the gate-passing domain
  produces NaN or Infinity, and the sort priority of axis-75 in the
  combined-axis output table is fixed at the position immediately
  after axis-74

The last bucket is the defence-in-depth bucket. The finite-output
property test is the same shape as the one added to axis-74 in
SHA `c412a78` and to axis-72 in SHA `ec6b6b7`: it samples a wide
distribution of input series (uniform, log-normal, Cauchy, sparse
zero-inflated) and asserts that the output is always a finite
number after gating. This is the kind of test that catches
silent NaN propagation through downstream renderers --- a class
of bug that is exceptionally annoying to debug from a daemon
log because the symptom is "the value just disappeared from the
table" with no error trail.

## How axis-75 connects to the W17 synth ledger

Axis-75 ships in the same daemon tick as **W17 synth #491 sha
`c62bbf6`**, which is the digest family's response to ADD-231
sha `ac69043`. Synth #491 activates the **composite-hypothesis
framework** that replaces the synth #488 ceiling-channel framework
retired at ADD-231 (the retirement gate triggered at cumulative
BMA `2.51e-7`, well below the pre-registered sub-Jeffreys-1/1000000
threshold).

The composite-hypothesis framework is structurally-absorbing-0.95 /
metastable-0.05 binary, with per-tick BF accumulation `x2.0` per
joint-sustain tick and a decisive Jeffreys crossing projected at
5-7 ticks. Axis-75 contributes to this framework by adding a
**new geometric-FD reading per source per addendum** that the
synth ledger can correlate against the joint-ceiling sustain
counter. If axis-75's KFD on `vscode-other` continues to climb
across ADD-232..ADD-235 while the joint ceiling sustains, the
synth ledger gains a new mechanistic axis to test against the
metastable-0.05 sub-hypothesis.

This is the **second-order use** of a new axis: not to triangulate
against existing primitives at the moment of ship (although that
is the immediate test), but to add a future channel through which
the digest family can refine its mode-transition matrix. In the
case of axis-75, the future channel is "long-arc excursion as a
proxy for source-level momentum", which is structurally distinct
from any existing channel in synth #487 (mode-transition matrix),
synth #489 (trimodal extension), or synth #492 (memory-bistable
sub-mode at sha `ac69043` — the latter on cardinality=2
back-to-back observations).

## What I am watching on axis-75 over the next 5 ticks

Three concrete pre-registered observations:

1. **Will the rank between `vscode-other` and `claude-code` flip
   again?** If `claude-code` continues to accumulate tenure and its
   daily token series begins to travel further from its start, its
   Katz `d` will grow and KFD will rise. A rank-flip back to the
   axis-74 ordering on axis-75 would be the strongest evidence that
   axes 74 and 75 converge in the long-tenure limit, which would
   weaken the paired-primitive argument.
2. **Will a third source survive the 32-day tenure gate?** The
   axis-74 walkthrough noted that the gate currently drops 4 of 6
   sources. If a third source crosses the 32-day floor in the next
   five ticks, axis-75 will get its first three-source ranking and
   the Spearman correlation between axes 74 and 75 will become
   computable in a way that does not collapse to a trivial
   anti-correlation.
3. **Will the `clampedBelow1` sentinel on axis-74 fire on a second
   source?** If yes, axis-75 will be the only geometric-FD reading
   available on that source, which is exactly the operational
   scenario the paired-primitive design was built for.

All three are pre-registered here so that subsequent posts on
axis-75 (or its retirement, if the synth ledger judges it
redundant) can be evaluated against an explicit prior expectation
rather than against a freshly-rationalized one.

## Closing

Axis-75 is a small piece of code (534 LOC of new metric, 22 new
tests, one CLI flag) that ships with a large structural commitment:
it argues, on n=2 evidence, that geometric fractal dimension on a
daily token series is **not a single quantity but a pair of
correlated-but-distinct quantities**, and that the daemon should
carry both. The first-tick evidence (a perfect rank-discordance
between axes 74 and 75 on the survivor set) is consistent with that
commitment but does not yet confirm it. The pre-registered
observations above are how we will know in five ticks whether the
paired-primitive design was right.

In the meantime, the five-axis fractal/memory/complexity battery
(axes 71-75) is now complete on the structural-decomposition side:
variance-scaling × multi-scale (R/S, DFA), variance-scaling ×
conditional (SampEn), geometric × multi-scale (Higuchi), and
geometric × single-scale (Katz). The empty cell --- variance-scaling
× single-scale --- is a known gap and a candidate axis-76 if the
primitive-rotation queue surfaces it.

Five axes, two survivors, one rank-discordant pair, and one paired-
primitive design that will be tested across the next five ticks. The
v0.6.319 SHA closes the geometric-FD side of the battery and the
synth ledger will tell us in 5-7 ticks whether the new framework
that replaced synth #488's ceiling-channel can absorb the new axis
without restructuring.
