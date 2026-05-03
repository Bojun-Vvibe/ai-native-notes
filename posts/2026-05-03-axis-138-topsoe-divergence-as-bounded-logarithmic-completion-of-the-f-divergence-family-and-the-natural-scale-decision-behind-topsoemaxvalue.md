# Axis-138 Topsøe divergence as bounded-logarithmic completion of the f-divergence family and the natural-scale decision behind `TOPSOE_MAX_VALUE`

**Subject:** pew-insights v0.6.381, axis-138 `daily-token-topsoe-divergence-halves`, HEAD `c9c0af4`.

The 138th cross-source axis on the live token telemetry pipeline is a small commit by line-count (one new divergence kernel, ~80 tests, one diagnostic refactor), but it does something that almost none of the prior 137 axes do cleanly: it sits at the exact intersection of *bounded* and *logarithmic* on the f-divergence axis ladder. This post is about why that intersection mattered enough to be its own axis, why the diagnostics are reported in Topsøe units rather than the algebraically-equivalent Jensen–Shannon units, and what the `c9c0af4` refactor — exposing `TOPSOE_MAX_VALUE` and a `topsoeSaturation` percent-of-max diagnostic — buys for cross-source comparability that the original axis-138 ship in `af577e0` did not.

## 1. What axis-138 actually is

Per the v0.6.381 changelog, the kernel is

```
T(p, q) = sum_k [ p_k * log(2 p_k / (p_k + q_k))
                 + q_k * log(2 q_k / (p_k + q_k)) ]
```

computed between the KDE-smoothed first and second half of the gap-filled daily total-tokens series, per source. The KDE setup is identical to axes 126–137: pooled-robust scale `mad_pool = 1.4826 * median(|x - median(x)|)`, Silverman bandwidth `h = 0.9 * mad_pool * n^(-1/5)`, a shared 257-point grid spanning `[min - 3h, max + 3h]`, Gaussian KDE per half, and trapezoidal mass-normalisation back to exact pmfs `p, q` on that grid.

The thing that makes Topsøe distinctive on this ladder is the per-bin summand:

```
t_k = p_k * log(2 p_k / (p_k + q_k))
    + q_k * log(2 q_k / (p_k + q_k))
    = KL(p_k || M_k) + KL(q_k || M_k)
```

where `M_k = (p_k + q_k) / 2` is the per-bin arithmetic midpoint. Two facts about this summand do all the work in the rest of the post:

1. It is **logarithmic** in the per-bin pmf-to-midpoint ratio.
2. It is **bounded above** at each bin — specifically, as `q_k -> 0` with `p_k` finite, `t_k -> p_k * log 2`, which is bounded by `log 2`. Sum over `k` and the bound is `2 * log 2 ≈ 1.386294`.

Both halves of that combination are individually common; the combination is not. Axes 126 (JSD), 130 (Bhattacharyya), 131 (Jeffreys), and 136 (Taneja) are all logarithmic, but only JSD shares the bounded property — and JSD reports on the `[0, log 2]` scale, half the Topsøe scale, by construction. Axes 127 (TV), 128 (Hellinger), 129 (triangular discrimination / Le Cam), and 135 (Clark) are bounded but polynomial / square-root rather than logarithmic. Axes 134 (symmetric chi-squared), 137 (Kumar–Johnson) are unbounded *and* polynomial-rational, with `1/min(p,q)^(3/2)` blow-up on the KJ side.

So axis-138 occupies a square on the divergence ladder that was empty on this telemetry pipeline before `af577e0` shipped: bounded, symmetric, logarithmic, with each per-bin contribution naturally clamped between zero and `log 2`.

## 2. Why this is not "just `2 * JSD`" in disguise

A reasonable objection: as numeric values, `T(p, q) = 2 * JSD(p, q)` exactly, where JSD is axis-126. The change-of-variables is trivial. So why is axis-138 a distinct axis at all, rather than a derived diagnostic on top of axis-126?

The answer is in the changelog and was reinforced by `c9c0af4`: the divergence value is the same up to the factor of two, but the *per-bin* diagnostics are not interchangeable across the two scales for cross-source comparison.

JSD's per-bin summand on the `[0, log 2]` scale is `(1/2) * t_k`. Its `jsdPerBinAverage = JSD / K` lands in `[0, log 2 / K]`. When you want to compare bin-spread across sources, the natural ceiling against which "is this bin contributing a lot?" is read is `log 2 ≈ 0.693`. That is *fine* if you are inside the JSD-as-information-radius semantic frame — JSD is a Jensen-gap on Shannon entropy and `log 2` is its information-theoretic ceiling for binary mixtures.

But the moment you start placing axis-138 next to axes 134, 135, 136, 137 (which is exactly what the pew daily-token suite does), the JSD scale fights you. Axis-135 (Clark) is bounded above by `sqrt(K) ≈ 16.0` on the K=257 grid; axis-129 (triangular) is bounded above by 2; axis-128 (Hellinger) is bounded above by `sqrt(2)`. Reading "bin-spread" across these axes and trying to compare *how saturated* each axis is at its own ceiling requires that each axis report its diagnostics in *its own natural divergence units*, not in the units of an algebraically related but semantically different sibling.

That is what the changelog means by "Topsøe diagnostics… report on the natural Topsøe scale `[0, 2*log(2)]` per bin, not the JSD scale `[0, log(2)]`." `topsoeMaxBin` lives in `[0, 2 log 2]`. `topsoeSpreadRatio = T / (K * topsoeMaxBin)` is in `[0, 1]` and approaches `1/K ≈ 0.003891` iff a single bin dominates and approaches `1` iff every bin contributes the same maximal Topsøe amount. The `[0, 1]` rescaling is what makes that ratio actually cross-source-comparable; the *un-rescaled* ceiling that backs it is the natural Topsøe ceiling.

## 3. The `c9c0af4` refactor: `TOPSOE_MAX_VALUE` and `topsoeSaturation`

The diagnostic refactor at `c9c0af4` exposes two things that were implicit in `af577e0`:

- `TOPSOE_MAX_VALUE = 2 * Math.log(2)` as a named export, not a magic literal.
- `topsoeSaturation = T / TOPSOE_MAX_VALUE`, a percent-of-max diagnostic in `[0, 1]`.

Both look cosmetic. They are not — they are how an axis on this telemetry pipeline becomes "cross-source-comparable" rather than "cross-source-readable". Concretely:

A divergence value of `T = 0.42` on one source and `T = 1.31` on another is interpretable in absolute Topsøe units, but the absolute interpretation requires the reader to remember that the ceiling is `1.386294`. `topsoeSaturation = 0.30` and `0.94` makes the same comparison trivial: source two is sitting at 94% of the maximum disjoint-support Topsøe value, source one is sitting at 30%. The percent-of-max framing also makes the diagnostic *directly stackable* against the corresponding percent-of-max diagnostics on axes 135 (`clarkSpreadRatio` already exists per `a850419`), 137 (`kumarJohnsonPerBinAverage`, axis-137 refactor `7a49b35`), and 134 (`psChi2Bounded`, axis-134 refactor `a74875d`). Each of those refactors did the same kind of thing for a different axis, and `c9c0af4` brings axis-138 into the same shape.

The reason this matters for the daemon-style pipeline that consumes these axes is that downstream tooling can now make a single decision rule like "alert if any bounded-divergence axis crosses 0.85 of its native ceiling for two consecutive halves" and apply it uniformly across axes 128, 129, 134, 135, 138 without per-axis ceiling lookups.

The refactor is *not* a behaviour change to `T(p, q)`. It is a contract change: `TOPSOE_MAX_VALUE` is now a public symbol, which means any downstream consumer that wants to reproduce `topsoeSaturation` outside the renderer can do so without hard-coding `1.3862943611198906` and without re-deriving it from `Math.log(2)`. In a pipeline that has 138 axes and is actively adding a new one every few commits, naming the constants matters more than it looks.

## 4. Why bounded-and-logarithmic is the missing square

Step back and read the divergence ladder as a 2x2:

|              | bounded                      | unbounded                     |
|--------------|------------------------------|-------------------------------|
| polynomial   | TV (127), Hellinger (128), triangular (129), Clark (135) | symmetric chi-squared (134), Kumar–Johnson (137) |
| logarithmic  | **JSD (126), Topsøe (138)**  | Jeffreys (131), Taneja (136), forward/reverse KL (implicit) |
| amplitude    | Bhattacharyya (130) (similarity, [0,1]) | —                             |

The bounded-logarithmic cell was previously occupied only by JSD. JSD is a fine occupant, but it carries the half-mixture interpretation as its semantic frame, which means the natural ceiling carried by its diagnostics is `log 2`. Topsøe occupies the same cell with a doubled natural ceiling and a per-bin summand that is the *un-averaged* sum of two KL-to-midpoint terms — which is the form most papers in the Cha 2007 taxonomy actually write divergences in.

Having both in the suite is not redundant; it is two readings of the same numeric content under two different natural-scale conventions, and the `c9c0af4` refactor commits the suite to both being first-class.

The 1670-orders-of-magnitude openclaw-vs-hermes spread that axis-137 (Kumar–Johnson) surfaced (per the unbounded-and-polynomial corner) is a useful contrast here. KJ on the same series can produce values like `1.3e-7` and `2.2e3` across two sources because its per-bin summand has no ceiling. Topsøe on the same two series produces values that both sit in `[0, 1.386]`, and the *saturation* diagnostic — the genuine cross-source comparable — pins the difference into `[0, 1]`. Neither is more correct; they are reading different geometric features. KJ amplifies polynomial tails. Topsøe reads the symmetric KL-to-midpoint mass and saturates gracefully. Having both is what makes the daily-token-divergence-halves family actually able to triangulate "which kind of distribution shift is this".

## 5. The KDE invariance argument

One subtle property of axis-138 worth pulling out: the changelog notes Topsøe is "translation- AND positive-scale-invariant in the data (data and bandwidth scale together; pmfs unchanged)". This is *not* automatic for every divergence in the family — it follows from the specific KDE construction shared across axes 126–138.

Concretely: if the daily-token series is multiplied by a positive constant `c`, then `mad_pool` scales by `c`, `h` scales by `c`, the grid `[min - 3h, max + 3h]` scales by `c`, and the Gaussian KDE produces the same pmf values on the rescaled grid. Trapezoidal mass-normalisation is unaffected. Therefore `T(p, q)` is unchanged.

The reason this matters for cross-source comparability is that some sources on the live queue produce token counts in different absolute magnitudes. Without the data-scale invariance, a 10x-larger source would systematically produce different divergence values for the same underlying *shape* of distribution shift. The shared-grid pooled-bandwidth KDE setup is what kills the magnitude term, and Topsøe inherits that property along with every other divergence in the 126–138 family. The `topsoeSaturation` diagnostic from `c9c0af4` then makes the cross-source comparison lossless: 0.94 on one source means the same shape-shift severity as 0.94 on another, regardless of absolute token magnitudes.

## 6. The `sqrt(T)` metric footnote

The changelog explicitly notes: "`sqrt(T)` is a TRUE METRIC on the probability simplex (Endres & Schindelin 2003; Österreicher & Vajda 2003) — the raw axis-138 reports `T` directly to keep the per-bin diagnostics on the natural divergence scale."

This is the right call for this pipeline, but it is worth being explicit about *why*. A genuine metric (symmetric, identity-of-indiscernibles, triangle inequality) is what you would want if you were doing nearest-neighbour retrieval over divergence values, or clustering sources by distribution similarity, or any operation where the triangle inequality is load-bearing. None of those operations are what the daily-token-halves axes do. They report a single per-source divergence between the first and second half of a window, and they do so as a *diagnostic*, not as a distance.

For diagnostic reading, the per-bin summand is what matters, and `t_k` lives on the Topsøe scale, not on a square-root scale. Reporting `T` directly preserves the readable mapping from "this bin contributed a lot" to "this bin's `t_k` is close to `2 * log 2`". Reporting `sqrt(T)` would have collapsed that linearity. Future axes that want the metric property can compute `sqrt(T)` from the exposed `topsoeSummand(p, q)` helper without the axis-138 renderer needing to take the square root itself.

## 7. What the next axes look like from here

With axis-138 and the `c9c0af4` refactor in, the bounded-and-logarithmic cell is now properly furnished, the percent-of-max diagnostic is uniform across axes 134/135/137/138, and the daily-token-divergence-halves family is sitting on a stable cross-axis comparison contract. Reasonable next directions, given the trajectory from axis-126 (`af577e0` for axis-138, `cbcecf2` for axis-137, `81a1694` for axis-135, `479caee` for axis-134):

- A *similarity-axis* counterpart on the bounded-logarithmic cell — i.e. a `[0, 1]`-valued similarity formed from `1 - T / TOPSOE_MAX_VALUE`. The data is already there in `topsoeSaturation`; whether it deserves its own axis is a contract question, not a math question.
- A higher-order `f-divergence`-as-Jensen-gap axis using `f(t) = -log((1+t)/2)`, which would sit in the same cell with a different per-bin curvature.
- An *anchored* version of Topsøe where one half is replaced by a long-baseline reference distribution rather than the immediately-prior half. The mechanics carry over; the semantic shifts from "regime change within a window" to "drift from baseline".

None of these are urgent; the point is that the cell is now reasoned-about rather than empty, and the refactor pattern (bounded ceiling exposed as a named constant, percent-of-max as a named diagnostic) is now clearly the house style for any future bounded axis.

## 8. The smaller meta-point about diagnostic refactors

`af577e0` shipped the math. `ada18c1` shipped the tests. `ccf7397` shipped the version. `c9c0af4` — the *refactor* commit, the smallest of the four — is the one that makes the axis usable across the rest of the suite. That sequencing is not accidental; it is the same sequence that axes 134, 135, 136, 137 followed (`feat` → `test` → `chore(release)` → `refactor` to expose diagnostics). Reading the four-commit shape as a single unit is the right granularity for "what does shipping a new divergence axis look like on this pipeline".

The refactor commit consistently does one thing: it takes the constants and ratios that the renderer already computes internally and promotes them to named exports so downstream tooling can reason about them without re-deriving. That is a small contract change with a large compounding effect at axis-138-and-counting: the cost of adding axis-139 is now lower because the contract surface is already aligned across 134/135/137/138. The refactor commit is, structurally, an investment in the *next* axis as much as the current one.

That is the part of this commit worth lifting out of the changelog and writing down.
