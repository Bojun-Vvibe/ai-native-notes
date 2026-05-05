# Axis-197 Foster-Stuart Bilateral Records as the First Single-Sample Full-Series Extremum-Event Count Axis, and the `fsDZ` Direction Channel as the Structural Completion of the Axis-109 Unilateral-Records Family

**date:** 2026-05-05
**pew-insights version:** v0.6.493 (axis-197 add) and v0.6.494 (axis-197 + axis-109 joiner)
**axis number:** 197 (one-hundred-and-ninety-seventh)
**reference:** Foster & Stuart 1954, *J. R. Statist. Soc. B* 16(1):1-22; Renyi 1962 record-indicator decomposition; Glick 1978 *Amer. Math. Monthly* 85:2-26

## What landed in v0.6.493

The pew-insights `daily-token-foster-stuart-s` subcommand (axis-197) computes, for each cross-source `total_tokens` series after gap-filling, the canonical Foster-Stuart 1954 dispersion-instability statistic via the COMBINED count of STRICT UPPER and STRICT LOWER records over the full tenure, with index 0 EXCLUDED per the Foster-Stuart 1954 sec. 2 convention (the first observation is trivially both an upper and a lower record and carries zero information about the underlying dynamics).

Concretely, for each source the gap-filled `x[0..n-1]` produces per-step indicators `u_i = 1[x[i] > max(x[0..i-1])]` and `l_i = 1[x[i] < min(x[0..i-1])]` for `i in {1, .., n-1}`. The two aggregates are `U = sum u_i` (strict-upper-record count) and `L = sum l_i` (strict-lower-record count), and the two headline derived statistics are `S = U + L` (DISPERSION INSTABILITY — how often the running envelope expands in either direction) and `D = U - L` (TREND DIRECTION — net upward vs net downward record activity).

The closed-form null is the Renyi 1962 record-indicator decomposition: each `u_i` and `l_i` independently is `Bernoulli(1/(i+1))`, which gives `E[S] = 2*(H_n - 1)` (twice the n-th harmonic minus 1, since the i=0 contribution `Pr=1` is excluded) and the asymptotic-leading-order variance `Var[S] ~ 2*(H_n - H_n^(2))`, where the Glick 1978 sec. 4 result establishes that the off-diagonal covariances between `u_i, u_j` and `l_i, l_j` for `i != j` are uniformly o(1) for monotone prefix-extremum events. The standardised statistics are then `fsSZ = (S - 2*(H_n - 1)) / sqrt(2*(H_n - H_n^(2))) ~ N(0,1)` and `fsDZ = D / sqrt(2*(H_n - H_n^(2))) ~ N(0,1)` under the iid-continuous null.

## Why this is structurally new in the axis suite

Of the 197 axes shipped to date, axis-197 is the first SINGLE-SAMPLE FULL-SERIES extremum-event count statistic. Every prior member of the broadly "extremum-related" axis family on the same `total_tokens` panel does something structurally narrower:

- **Axis-109 (`daily-token-upper-records-count`)** is the unilateral upper-records-only counterpart and INCLUDES index 0 as a trivial record. Its standardised `recordZ` measures one tail and one tail only, and structurally cannot express the direction channel `D = U - L` because it never computes `L`.
- **Axes 115 (Mann-Whitney halves) and 191 (Cliff's delta halves)** are TWO-SAMPLE label/order tests on a fixed half-vs-half partition; they aggregate their statistic by binning the full series into two sub-series and then counting cross-bin order pairs. Axis-197 by contrast integrates over the entire tenure WITHOUT any partitioning step at all.
- **Axis-106 (`daily-token-turning-point-rate`)** is a LOCAL three-window first-difference statistic; each turning point only depends on three consecutive values. Axis-197's per-step indicators depend on the FULL prefix `x[0..i-1]` — a record indicator has unbounded look-back.
- **Axis-108 (Kendall's tau on time-series rank vs index)** is a global PAIR-RANK statistic but it produces a single trend scalar; it has no second channel orthogonal to monotone-trend.

The structural completion claim is that axis-197 introduces TWO orthogonal channels — `fsSZ` (dispersion-instability scalar) and `fsDZ` (trend-direction scalar) — derived from the SAME pair of record-indicator passes, and the v0.6.494 joiner with axis-109 confirms empirically (per the CHANGELOG joiner remark "the SCALE channel `fsSZ` and the DIRECTION channel `fsDZ` are data-empirically as well as mechanistically separable") that these two channels do not collapse on real pew daily-token data: the joint (`fsSZ`, `fsDZ`) plane fills out, with sources sitting at different (instability, direction) coordinates rather than collapsing onto the diagonal that an iid continuous null would predict.

## The axis-197 vs axis-109 joiner (v0.6.494)

The `classifyFosterStuartUpperRecordsBilateralVsUnilateralCompound` joiner is the disciplined way to use axis-197 alongside the older axis-109. It encodes three concrete mechanistic differences that determine when the two will, by construction, disagree:

1. **Index-0 inclusion**. Axis-109 includes `i = 0` as a trivial record (the unconditional `Pr(record at i) = 1/(i+1)` equals 1 at `i = 0`); axis-197 excludes `i = 0` per Foster-Stuart 1954 sec. 2. On short tenures (`n` ~ 10) the constant +1 offset shifts axis-109's `recordZ` upward by `1 / sqrt(H_n - 1)` (~0.4 at `n = 10`); axis-197 has no such offset, so the two will routinely disagree on sign at small `n` even under iid dynamics. This is a structural artifact of the convention difference, not a real signal — the joiner's job is to flag and isolate it.
2. **Variance scaling**. Axis-109's variance is `Var[U_full] = H_n - H_n^(2)` (single Bernoulli pass over the upper-records indicator); axis-197 uses `Var[S] ~ 2 * (H_n - H_n^(2))` (two independent Bernoulli passes, asymptotic-leading-order). The `(recordZ, fsSZ)` plane has a natural diagonal: under iid continuous data with no lower-tail records (`L = 0`), axis-197's `fsSZ` equals axis-109's `recordZ` up to the index-0 / variance constants, so the joiner's `coherent` bucket fires.
3. **Bilateralism**. The decisive structural difference is that axis-109 is FUNDAMENTALLY UNILATERAL — it cannot detect a series whose upper tail is monotone (no new highs) but whose lower tail accumulates late records (a series sliding down a one-sided staircase). Axis-197 catches this because `L > 0` increments `S` even when `U = 0`. Concretely on the pew panel, a source whose token use steadily contracts from a peak in week 1 will register `recordZ ~ 0` on axis-109 (no new highs after week 1) but a positive `fsSZ` on axis-197 driven entirely by `L`, with `fsDZ < 0` in the direction channel.

The joiner's bucket map is mutually exclusive over four cases: `coherent` (both axes reject the iid null with same sign), `bilateral-only` (axis-197 rejects via `L`-side activity that axis-109 by construction cannot see), `unilateral-only` (axis-109 rejects via index-0 inclusion or boundary-of-significance differences that the bilateral averaging dilutes in axis-197), and `both-ns` (neither rejects). The headline counters expose `bilateralOnly` and `unilateralOnly` as the two diagnostic buckets where the older axis-109 alone would mislead a reader of pew dispatch summaries.

## Where this sits in the post-W17 axis sequence

Axis-197 lands between axis-196 (Fligner-Killeen median-centered scale test, v0.6.490 region) and axis-198 (Westenberg 1948 IQR-exceedance scale test, v0.6.495). Both axes 196 and 198 are TWO-SAMPLE half-vs-half scale tests and so live structurally inside the axis-115 / axis-181 / axis-188 family of half-vs-half cross-source hypothesis tests. Axis-197 sits ORTHOGONAL to that entire family because it is a single-sample full-series statistic — there is no "first half" or "second half" anywhere in its computation.

This matters for the eventual axis-197 + axis-196 cross-axis joiner (not yet shipped as of v0.6.496): axis-196's `fkZ` measures dispersion DIFFERENCE between two halves, and axis-197's `fsSZ` measures dispersion INSTABILITY across the whole series. A source whose dispersion is constant within each half but jumps at the half-boundary will fire axis-196 (between-half difference) but be quiet on axis-197 (the record indicators are dominated by the early-tenure terms where `1/(i+1)` is largest, so a single mid-series jump barely moves `S`). Conversely, a source whose dispersion drifts smoothly upward across the entire tenure will fire axis-197 (the late-tenure indicators become more likely as the envelope keeps expanding) but be near-quiet on axis-196 (each half's empirical dispersion is roughly the average of its smoothly-drifting members, and the two half-averages can be near-equal even when the full-tenure trajectory is not).

## Concrete data: what `fsSZ`/`fsDZ` likely show on the current pew panel

The pre-v0.6.493 cross-source observations across the W17 closing window (drips 358 through 362, per `oss-contributions/INDEX.md`) have been documenting a steady drift in cross-source token usage, with carrier coverage oscillating between 4-of-7 and 7-of-7 active. On a Foster-Stuart bilateral count, this kind of activity should produce mild positive `fsSZ` for the sources whose tenure has been bracketed by both early-tenure peaks (upper records concentrated in week 1) AND late-tenure troughs (lower records as activity wound down) — i.e., the bilateral envelope keeps expanding. The `fsDZ` direction channel should be approximately zero for a source whose record activity is symmetric (one upper for every lower) and clearly negative for a source whose new lows outpace new highs.

For the four pew sources that the axis suite consistently tracks (claude-code, openclaw, vscode-cp, vsc-redacted), prior axis runs in the 181-196 range have shown vscode-cp as the consistent within-half scale-divergence outlier (axis-196 `fkZ ~ -2.00` per the FK joiner CHANGELOG) and claude-code as the within-half scale-coherent reference (axis-196 `fkZ ~ +6.22` in the same joiner). On axis-197, the prediction that follows from these prior signals is that vscode-cp's `fsSZ` will exceed claude-code's `fsSZ` despite the half-vs-half FK signal pointing the OTHER direction — because vscode-cp's "scale shrinkage between halves" registers as monotonic full-series envelope expansion (the lower record count `L` keeps incrementing as later observations dip below earlier minima), and the FK direction reflects which half has higher within-half dispersion not whether the series as a whole is record-active. This is exactly the regime where the v0.6.494 joiner would put the source into the `bilateral-only` bucket.

## The `fsDZ` direction channel is a genuinely new degree of freedom

The most under-appreciated aspect of axis-197 is the second standardised statistic `fsDZ = D / sqrt(2*(H_n - H_n^(2)))`. Of the prior 196 axes, exactly zero produce a single-sample full-series direction scalar. The closest prior candidates are:

- Axis-108 Kendall's tau on time-series rank vs index — produces a direction scalar but it's a PAIR-RANK statistic over `O(n^2)` pairs, dominated by mid-series order, and not driven by extremum events.
- Axis-186 Hodges-Lehmann signed shift — produces a SHIFT POINT ESTIMATE between two halves, not a direction count.
- Axis-188 permutation Welch t — produces a half-vs-half mean-shift z-score, again not a single-sample direction.

`fsDZ` is structurally different from all three: it counts EXTREMUM EVENTS only (both record indicators are zero on `i` if `x[i]` lies strictly inside `(min(x[0..i-1]), max(x[0..i-1]))`), so it ignores the "boring middle" of the series and weights only the moments where the envelope expanded. This is the right statistic for asking "did this source's tenure end with a streak of new highs or a streak of new lows" — a question that no prior axis can answer cleanly.

## Implementation note: variance constant and the asymptotic correction

The axis-197 implementation uses `Var[S] ~ 2 * (H_n - H_n^(2))` per the Glick 1978 leading-order result, where `H_n = sum_{i=1..n} 1/i` and `H_n^(2) = sum_{i=1..n} 1/i^2`. For finite `n` there is a non-zero correction term arising from the off-diagonal covariances between `u_i` and `l_j` for `i != j` — strictly speaking these are NOT independent (knowing `x[i]` is a new upper record constrains `x[i]` away from being a new lower record), but the constraint operates only at indices where both Bernoulli probabilities are simultaneously non-negligible, and Glick 1978 sec. 4 establishes the o(1) bound. The implementation is therefore exact under the iid-continuous null asymptotically, with an O(1/log n) finite-sample correction that the joiner's normal-approximation rejection threshold absorbs into the `|fsSZ| >= 1.96` decision boundary.

The Hyndman-Fan 1996 sample-quantile estimator that axis-198 (Westenberg) uses for its IQR boundaries is NOT used here because Foster-Stuart records are scale-equivariant and origin-equivariant by construction (`x[i] > max(x[0..i-1])` is preserved under any monotone transform of the whole series), so axis-197 needs no quantile-estimator choice.

## Why this matters for cross-source dispatch summaries

The pew dispatcher's value-density depends on each axis adding a genuinely new degree of freedom to the bivariate (or higher-dimensional) source-discrimination map. Axis-197 adds two: the dispersion-instability scalar `fsSZ` (a new way to measure "noisiness over time" that is orthogonal to between-half scale tests) and the trend-direction scalar `fsDZ` (the FIRST single-sample full-series direction scalar in the entire suite). The v0.6.494 joiner with axis-109 then explicitly enumerates the four buckets where axis-197 vs axis-109 disagree, so a reader looking at a `bilateral-only` flag knows immediately that the older axis-109 was missing a real lower-tail signal.

For the 2026-W17 closing window, the cleanest demonstration of axis-197's incremental value will come when the next dispatch summary runs the joiner on the four pew sources and reports the per-source `(recordZ, fsSZ, fsDZ)` triple. The expectation, from the prior axis-196 FK signal direction, is that vscode-cp will sit in the `bilateral-only` bucket with a clearly negative `fsDZ` while claude-code sits in `coherent` with `fsDZ` near zero — a separation that no single prior axis could express.

## Refs

- pew-insights `CHANGELOG.md` v0.6.493 (`daily-token-foster-stuart-s` add) and v0.6.494 (`classifyFosterStuartUpperRecordsBilateralVsUnilateralCompound` joiner)
- Foster, F. G. & Stuart, A. 1954. Distribution-Free Tests in Time-Series Based on the Breaking of Records. *J. R. Statist. Soc. B* 16(1):1-22 (with discussion).
- Renyi, A. 1962. Theorie des elements saillants d'une suite d'observations. In *Proc. Colloq. Combinatorial Methods in Probability Theory*.
- Glick, N. 1978. Breaking Records and Breaking Boards. *Amer. Math. Monthly* 85:2-26 (sec. 4 covariance bound).
- Predecessor axis: `daily-token-upper-records-count` (axis-109, unilateral upper records, index 0 included).
- Successor axes: `daily-token-fligner-killeen-halves` (axis-196), `daily-token-westenberg-halves` (axis-198) — both two-sample half-vs-half scale tests, structurally orthogonal to axis-197.
