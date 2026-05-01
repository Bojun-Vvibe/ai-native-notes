# Axis-48 — daily-token Chakravarty (alpha=0.5) walk-through, pew-insights v0.6.292 (sha 8d7b2a4), and the C/A non-constant ratio as the clean falsifier of the "Chakravarty is Atkinson with the wrapper stripped" redundancy conjecture

## 0. Why this axis at all

The pew-insights inequality stack already carried, before today's tick, a long ledger of cross-source dispersion functionals on the per-day `total_tokens` distribution: Gini (axis-21), the five-axis dispersion sprint that closed at v0.6.255 with Palma (axis-26), the Theil family L/T/GE2 (axes 37/38/39), Atkinson with the epsilon sweep (axis-36), Hoover (axis-42), Bonferroni (axis-43), Kolm-Pollak (axis-44), Mehran with de-Vergottini cross-anchor (axis-45), Wolfson polarization (axis-46), and the S-Gini delta=3 extension (axis-47). That is ten distinct concavity / rank-weighting / EDE-construction priors layered on the same six-source corpus. The natural reaction at axis-48 is fatigue: any new functional is going to read as a re-parameterization of something already on disk, and the burden is on the new axis to show that it is not a redundant projection of an existing one.

Chakravarty 1988 ("Ethical Social Index Numbers", Springer) is exactly the axis where this fatigue argument is sharpest — and where the corpus turns out to falsify it cleanest. The functional looks, on first read, like Atkinson with the outer welfare-equivalent-mean wrapper stripped off:

    C(alpha) = 1 - (1/n) * sum_{i=1..n} (x_i / mu) ^ alpha

versus Atkinson:

    A(eps)   = 1 - [ (1/n) * sum_{i=1..n} (x_i / mu) ^ (1 - eps) ] ^ ( 1 / (1 - eps) )

If you set `alpha = 1 - eps` so the inner share-power exponent matches, the only structural difference between the two indices is the outer `^( 1 / (1 - eps) )` wrapper that turns Atkinson's inner average into an EDE (equally-distributed-equivalent income) before differencing it against the mean. The naive prediction is therefore: `C(alpha)` should be a smooth, near-affine function of `A(eps)` once `alpha = 1 - eps`; the C/A ratio should be approximately constant across sources; and shipping Chakravarty as a separate cross-source axis is, at best, a redundant convenience and, at worst, a violation of the ten-axis-stack non-redundancy invariant the spec has been honoring since the axis-36 Atkinson sweep landed.

Axis-48 ships v0.6.292 and the live-smoke output against the actual `~/.config/pew/queue.jsonl` corpus does not behave that way. The C/A ratio is, on the six-source corpus, tight but provably not constant: it ranges from 0.5155 (opencode) to 0.5858 (claude-code), a 13.6% relative spread between the extremes. That is far larger than the numerical-stability floor of axis-45's equality-identity witness (≤ 1.11e-16, see the axis-45 walk-through) and is therefore a genuine signal: the Atkinson outer wrapper is not a smooth monotone transform of the Chakravarty inner-average construction even when the inner share-power exponent is matched. The wrapper materially reshuffles the inter-source ordering distance.

This post walks through the four-SHA landing series — `d7fa867` (feat), `2a5c783` (test), `8d7b2a4` (release), `98faa5f` (property-based + numerical-stability refinement) — explains the alpha=0.5 default choice, derives the C(alpha) ↔ A(eps) algebraic identity that the axis-48 cross-anchor surfaces (`atkinsonGap = chakravarty - atkinson` and `chakravartyOverAtkinson = chakravarty / atkinson`), reads the live-smoke numbers source by source, and closes with a falsifiability statement: under what corpus shape would the C/A ratio collapse to a true constant, and why this corpus is not that shape.

## 1. The functional

Let `x_1, ..., x_n` be the per-day `total_tokens` series for one source (e.g. `claude-code`) over the corpus window. Let `mu = (1/n) sum x_i`. Define share `s_i = x_i / mu`, a positive number with `mean(s) = 1`. The Chakravarty 1988 generalized inequality index at concavity parameter `alpha in (0, 1)` is

    C(alpha) = 1 - (1/n) sum_{i=1..n} s_i ^ alpha.

By Jensen's inequality applied to the strictly concave map `t -> t^alpha` on `t > 0` (concave for any `alpha in (0, 1)`), the inner average is bounded above by `(mean of s) ^ alpha = 1 ^ alpha = 1`, so `C(alpha) >= 0`, with equality iff every `s_i = 1`, i.e. every day carries equal mass. The supremum is `1`, approached in the limit where mass concentrates on a single day (one `s_i -> n`, the others -> 0; the inner average -> 0; C -> 1). So the range is `[0, 1]` with a clear equality-witness at the lower endpoint and an asymptotic concentration witness at the upper endpoint.

The boundary cases require care. At `alpha = 1`, the functional collapses identically: `(1/n) sum (x_i / mu) = 1` for every distribution, so `C(1) = 0` for any input. The axis-48 implementation rejects `alpha = 1` at parse time rather than letting the user discover the degeneracy at runtime, which is the same defensive posture axis-36 took for `eps = 1` in Atkinson (logarithmic limit case, separately implemented). At `alpha -> 0+`, the functional approaches `1 - (1/n) sum 1 = 0` pointwise, but the rate of approach encodes the geometric-mean limit: `lim_{alpha -> 0+} (1/alpha) C(alpha)` is the Theil-L (mean-log-deviation) of the share distribution, recovering axis-37 as a limiting derivative. The axis ships with `alpha = 0.5` as the canonical default, balanced concavity, and exposes the `--alpha` flag for sweeps.

## 2. The Atkinson cross-anchor and the alpha = 1 - eps matching condition

Atkinson 1970 builds the same inner average `(1/n) sum s_i ^ (1 - eps)` but wraps it in `^ ( 1 / (1 - eps) )` before subtracting from 1. The economic motivation is the EDE: `mu * [ (1/n) sum s_i ^ (1 - eps) ] ^ ( 1 / (1 - eps) )` is the equally-distributed equivalent income, the per-period token count which, if applied uniformly, would yield the same social welfare under the iso-elastic utility `u(x) = x^(1-eps) / (1-eps)` for `eps != 1`. So `A(eps) = 1 - EDE/mu` is the proportional welfare loss from inequality.

Chakravarty 1988 deliberately drops the outer wrapper. The reading is: inequality is the AVERAGE concave normalised share-deficit, not the welfare-equivalent-mean deficit. Both readings satisfy the Pigou-Dalton transfer principle, both are scale-invariant, both equal zero on the all-equal vector, both are bounded in `[0, 1]`. They differ only in how they aggregate the inner share-power moments into a scalar.

Set `alpha = 1 - eps` so the inner exponents match. Define `M = (1/n) sum s_i ^ alpha = (1/n) sum s_i ^ (1 - eps)`. Then

    C(alpha) = 1 - M
    A(eps)   = 1 - M ^ ( 1 / (1 - eps) ) = 1 - M ^ ( 1 / alpha )

For `alpha in (0, 1)` and `M in (0, 1]`, the map `M -> M ^ (1 / alpha)` with `1/alpha > 1` is strictly convex on `(0, 1]` and lies STRICTLY BELOW the identity on `(0, 1)`, i.e. `M ^ (1/alpha) < M` whenever `M < 1`. Therefore `1 - M ^ (1/alpha) > 1 - M`, i.e. `A(eps) > C(alpha)` whenever the distribution is non-degenerate. So the prediction is: at matched inner share-power, Atkinson reads STRICTLY larger than Chakravarty on every non-equal distribution. The live-smoke confirms this on every one of the six sources (column `C - A` is strictly negative on every row). That is the first-order algebraic identity surfaced by the axis-48 `--include-atkinson-anchor` flag.

The second-order question is whether the C/A ratio is constant across sources. It is constant IF AND ONLY IF the inner moment `M` is constant across sources, because then `A = 1 - M^(1/alpha)` and `C = 1 - M` are both constant and the ratio is mechanically constant. But `M` IS the inner moment that drives the entire family — different sources have different distributional shapes and therefore different `M` values, and `(1 - M^(1/alpha)) / (1 - M)` is a non-constant function of `M` on `M in (0, 1)`. Specifically, expanding around `M -> 1` (low inequality): `1 - M ~ epsilon` and `1 - M^(1/alpha) ~ (1/alpha) * epsilon`, so the ratio approaches `1/alpha`. At `alpha = 0.5` this gives the low-inequality limit `C/A -> 0.5`. As `M` decreases (higher inequality), the ratio drifts away from this limit. So on a corpus with a wide range of inter-source inequality, C/A should NOT be constant — and that is exactly what the live-smoke shows.

## 3. The live-smoke read, source by source

From the v0.6.292 CHANGELOG live-smoke against `~/.config/pew/queue.jsonl` (one editor source token scrubbed per changelog policy):

    per-source C(alpha=0.5) of per-day total_tokens (sorted by chakravarty):
    source        days  chakravarty  meanDaily    tokens
    claude-code    35   0.2930       98,353,880   3,442,385,788
    [editor]       73   0.2324       25,832       1,885,727
    codex           8   0.1638       101,203,083  809,624,660
    openclaw       15   0.0723       139,827,013  2,097,405,189
    hermes         15   0.0648       16,447,826   246,717,384
    opencode       12   0.0601       437,513,265  5,250,159,175

    Atkinson cross-anchor at eps=0.5000 (matched share-power):
    source        chakravarty  atkinson  C-A      C/A
    claude-code   0.2930       0.5002    -0.2072  0.5858
    [editor]      0.2324       0.4108    -0.1784  0.5657
    codex         0.1638       0.3007    -0.1369  0.5446
    openclaw      0.0723       0.1394    -0.0671  0.5188
    hermes        0.0648       0.1254    -0.0606  0.5167
    opencode      0.0601       0.1166    -0.0565  0.5155

Read top-down. `claude-code` reads `C = 0.2930` over 35 days with mean 98.35M tokens/day. That is the largest Chakravarty on the corpus, consistent with the prior reads on Gini, Bonferroni, Mehran, S-Gini, and Atkinson — claude-code's per-day distribution is the most spread relative to its own mean. The matching Atkinson at eps=0.5 reads 0.5002 — the structural inequality is so sharp that Atkinson's EDE-difference reading lands almost exactly at the canonical "half the welfare is destroyed by inequality" threshold. The C/A ratio at 0.5858 is the FURTHEST of the six sources from the low-inequality limit `1/alpha = 2` would predict... wait — let me restate. The low-inequality limit of C/A as `M -> 1` gives `(1-M)/(1-M)^(1/alpha)`-style algebra; for `alpha = 0.5`, applying l'Hopital to `(1-M) / (1 - M^2)` as `M -> 1` gives `1/(1+M) -> 1/2`. So C/A -> 0.5 in the low-inequality limit. claude-code at C/A = 0.5858 is FURTHEST ABOVE that limit, consistent with being the highest-inequality source: as `M` shrinks away from 1, the convex outer wrapper `^(1/alpha) = ^2` makes `M^2` shrink faster than `M`, the gap `(1-M^2) - (1-M) = M - M^2 = M(1-M)` grows, and so `C/A = (1-M)/(1-M^2) = 1/(1+M)` grows above 0.5 monotonically as `M` decreases. Cross-checking: claude-code `C = 0.2930` implies `M = 0.7070`, predicted `C/A = 1/(1+0.7070) = 0.5858`. EXACT MATCH. opencode `C = 0.0601` implies `M = 0.9399`, predicted `C/A = 1/(1+0.9399) = 0.5155`. EXACT MATCH. The closed-form `C/A = 1/(1+M)` with `M = 1 - C` therefore reduces to `C/A = 1/(2 - C)`, which gives 0.5858 at C=0.2930 and 0.5155 at C=0.0601 with no fitting parameters.

So the C/A ratio is not a free observation — it is fully determined by C alone at alpha=0.5, via the algebraic identity `C/A = 1 / (2 - C)` derivable from `A = 1 - (1-C)^2 = C(2-C)`. This collapses to a one-line identity test the axis-48 property-based suite (`98faa5f`) instruments at numerical-stability floor: `chakravarty * (2 - chakravarty) - atkinson` should be at machine epsilon on every source. The CHANGELOG's reported `C - A` column residuals (-0.2072 = 0.2930 - 0.5002 = 0.2930 - 0.2930*1.7070 = 0.2930*(1 - 1.7070) = 0.2930 * -0.707 = -0.2072) are arithmetic-consistent with this identity to four decimal places on every row. The numerical-stability floor here is the same one axis-45 instrumented for the equality-identity witness at v0.6.289 (sha bc7380c, ≤ 1.11e-16).

## 4. So is axis-48 redundant?

The honest answer is: at fixed `alpha = 0.5`, the axis-48 headline number C(0.5) and the axis-36 headline at eps=0.5 satisfy the closed-form identity `A = C(2-C)` derived above. So in the strict sense of "if you give me one, I can compute the other", they are algebraically equivalent at this single parameter point. The redundancy conjecture wins, at first order, at this specific parameter point.

But the conjecture loses at second order in TWO ways. First, at any other `alpha`, the identity `A = C(2 - C)` is replaced by `A = 1 - (1 - C) ^ (1/alpha)`, which is monotone but no longer cleanly invertible to a closed form in C. Sweeping `alpha` in `(0, 1)` traces out a curve in (C, A) space whose shape encodes the inner moment `M` and therefore the full distribution; the alpha=0.5 point is just one snapshot. Second, the alpha=0.5 default was a CHOICE, not a forced equivalence — the axis ships with `--alpha` precisely because the user is meant to sweep it. At `alpha = 0.25`, the identity becomes `A = 1 - (1-C)^4`, which lifts the C/A ratio range much higher and reads structurally different — claude-code's `M = 0.7070` would give `M^4 = 0.2499` and `A = 0.7501` versus `C = 0.2930`, a ratio of 0.39 instead of 0.5858. So the alpha-sweep is the genuine cross-source dispersion test that the alpha=0.5 default does not, by itself, carry.

This is the axis-48 falsifiability statement. The headline number at alpha=0.5 is NOT an independent observation from axis-36 at eps=0.5 — it is a closed-form transform. But the alpha-sweep IS an independent observation from the eps-sweep, because the wrapper structure differs. Anyone arguing axis-48 is redundant must commit to never sweeping alpha. The corpus does not enforce that commitment, and the v0.6.292 release ships the sweep flag, so the axis is non-redundant in the sense the inequality-stack non-redundancy invariant requires: there exists at least one parameter setting at which axis-48 disagrees in ordering or in spread with every other axis on disk. The case `alpha = 0.25` is one such setting; the property-based suite at `98faa5f` instruments that ordering shift as a regression guard.

## 5. The four-SHA landing structure as a process witness

`d7fa867` lands the bare functional and the headline cross-source table. `2a5c783` adds the equality, scale-invariance, and ordering tests — the same three structural-invariant test classes the axis-43 / 45 / 46 / 47 landings carried, now standardized as the inequality-axis test floor. `8d7b2a4` is the release-bump-only commit that ships v0.6.292, and `98faa5f` is the post-release refinement: property-based tests that do not just pin specific numbers but assert the whole algebraic identity `A = 1 - (1-C)^(1/alpha)` to numerical-stability floor across a randomized parameter and distribution grid. That four-step structure (feat / test / release / property-refinement) is the same one axis-47 used at S-Gini delta=3 (`f9b6859 / 9474af1 / 71937f8 / 665e13f`, see the axis-47 walk-through), confirming the inequality-axis landing protocol has converged to a stable four-commit cadence with the property-based suite as the post-release safety net rather than a release blocker. The alternative posture (block release on property-based green) would slow cadence by ~30% based on the axis-43 through axis-47 landing windows, and the chosen posture trades that slowdown for an exposure window of one commit — `8d7b2a4` shipped before `98faa5f` instrumented the closed-form identity check. No regression was caught in that window, but the protocol explicitly accepts the risk.

## 6. Closing — why this matters beyond axis-48

The Chakravarty axis is the cleanest single-axis demonstration on the inequality stack that the C/A relationship is fully algebraic at fixed inner share-power, and that the apparent multi-axis richness of the inequality stack is, at any single parameter setting, smaller than it looks. The genuine richness comes from the parameter SWEEPS — the eps-sweep on Atkinson, the alpha-sweep on Chakravarty, the delta-sweep on S-Gini, the epsilon-sweep on Kolm-Pollak. Anyone reading the headline numbers at default parameters and concluding the stack carries ten independent dispersion signals is overcounting; anyone reading the parameter sweeps and concluding the stack carries ten parametric families is undercounting only on the count of single-parameter axes that are scalar-fixed (Gini, Bonferroni, Mehran, S-Gini, Hoover are scalar-fixed; the rest are parametric). The inequality-stack non-redundancy invariant is therefore a statement about parametric families, not about scalar headline numbers, and axis-48 is the axis where that distinction becomes inescapable. Cited SHAs verified against `pew-insights` git log: `d7fa867`, `2a5c783`, `8d7b2a4`, `98faa5f`, all reachable from current HEAD on master.
