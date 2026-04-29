---
title: "Three families of slope estimators in the pew-insights suite: y-asymmetric, non-parametric EIV, parametric EIV — and why v0.6.219 closes the taxonomy"
date: 2026-04-29
---

# Three families of slope estimators in the pew-insights suite: y-asymmetric, non-parametric EIV, parametric EIV — and why v0.6.219 closes the taxonomy

The pew-insights v0.6.219 release (`5d2f9b5`) added Deming regression to the slope-estimator suite. Taken alone, that is a single feature in a single release. Taken in context — that is, alongside Theil-Sen (v0.6.214), Siegel repeated medians (v0.6.216), and Passing-Bablok (v0.6.218) — it is the closing move in a three-family taxonomy that the suite has been quietly assembling over the past two weeks. This post is the taxonomy.

The thesis is simple: **every slope estimator makes a structural assumption about which axis carries the measurement error, and that assumption sorts the estimator into one of three families.** The pew-insights suite, as of v0.6.219, has at least one canonical member of each family. That is not an accident; the release sequence reads like a deliberate completion. Below I walk through each family, place every slope estimator the suite ships into one of them, and show what changes about how you should read each estimator's output once you know which family it belongs to.

## Family 1: y-asymmetric ("regress y on x")

**Assumption**: x is measured without error; all noise lives in y.

**Optimization criterion**: minimize the sum of vertical residuals (squared, absolute, or otherwise transformed).

**Members in the suite**: OLS, Theil-Sen (v0.6.214), Siegel repeated medians (v0.6.216), Mann-Kendall trend slope, plus all of the M-estimator family from v0.6.207 (Huber) through v0.6.217 (Geman-McClure).

**Defining property**: swapping x and y and refitting gives a slope that is **not** the reciprocal of the original. If you fit y on x and get slope `m`, then fit x on y and get slope `m'`, you will find that `m · m' ≠ 1` unless your data has perfect correlation. The discrepancy is the regression-dilution attenuation factor, and it is the diagnostic signature of an y-asymmetric estimator.

The reason this family is by far the largest in the suite is historical and computational. OLS was the first published regression estimator. Theil-Sen (1950, 1968) and Siegel (1982) were proposed as robust alternatives that preserved the y-asymmetric assumption while gaining resistance to outliers. The M-estimator march from Huber (1964) through Geman-McClure (1985) was specifically about replacing OLS's quadratic loss with more robust losses, again preserving y-asymmetry as a structural choice.

**When y-asymmetric is the right family to use**: when you actually have a controlled x. Calendar time is a clean example — `2026-04-29` is not a noisy measurement of itself. Counts of discrete events at known indices are clean. Anything where x is the experimental design variable and y is the measured response is in this family.

**When y-asymmetric is the wrong family to use**: when both axes are observed quantities with their own measurement processes. Token counts vs row counts. Pull request review duration vs PR diff size. Deployment latency vs deployment payload size. In all of these, x is itself an observation, and applying y-asymmetric estimators silently assumes `Var(x) = 0` and silently attenuates the slope toward zero.

The suite has 14+ members of this family — by my count, OLS, Theil-Sen, Siegel, Mann-Kendall, Huber, Tukey, Andrews, Hampel, Welsch, Geman-McClure, plus the trim-mean-based slopes from earlier in the L-estimator march. That is a lot. The reason is that y-asymmetric estimators differ from each other along an orthogonal axis: their **robustness to outliers**. The whole M-estimator march was about exploring different shapes of the influence function — bounded-but-non-redescending (Huber), redescending polynomial (Tukey), redescending transcendental (Andrews), three-part piecewise (Hampel), continuous redescending (Welsch, Geman-McClure). All of those are y-asymmetric; they differ only in how they handle outlying y residuals.

## Family 2: non-parametric EIV ("symmetric in x and y, no assumed noise distribution")

**Assumption**: both x and y carry measurement error, but the analyst makes no parametric claim about the distribution of that error.

**Optimization criterion**: a procedure that gives the same slope when x and y are swapped (modulo reciprocation), without optimizing a likelihood.

**Members in the suite**: Passing-Bablok (v0.6.218).

**Defining property**: fit y on x, get slope `m`. Swap to x on y, get slope `m'`. You will find `m · m' = 1` exactly. This is the x/y symmetry property that defines the EIV families.

Passing-Bablok achieves this by computing all pairwise slopes (like Theil-Sen) but then taking a special median that corrects for the systematic shift introduced by ranking pairwise slopes around zero. The procedure is non-parametric in the strong sense: no assumption about the joint distribution of `(x, y)` measurement errors is required. It is robust to outliers (50% breakdown, like Siegel) and it is symmetric in the two variables.

The CHANGELOG entry for v0.6.218 explicitly described Passing-Bablok as "the first x/y-symmetric slope in the suite." That phrase was deliberate. It signaled the opening of the second family. The release notes did not yet say "non-parametric EIV" — they used the looser phrase "x/y-symmetric" — but the structural distinction was there. The v0.6.219 release notes for Deming retroactively named the family by introducing a contrast: "completes the EIV pair (parametric + non-parametric)."

The reason there is currently only one member of family 2 in the suite is that non-parametric EIV is a small and well-defined class. Passing-Bablok is the canonical and essentially the only widely-used member. There are variants (Passing-Bablok with cusum-based outlier handling, weighted Passing-Bablok) but they are extensions of the same procedure rather than structurally distinct estimators.

The expected future addition to family 2 would be a **rank-based EIV estimator** — something like a Spearman-correlation-derived slope that preserves x/y symmetry. The suite does not yet ship one. That is a plausible v0.6.220-v0.6.225 direction.

## Family 3: parametric EIV ("symmetric in x and y, with an assumed noise distribution")

**Assumption**: both x and y carry measurement error, AND the analyst commits to a parametric form for the joint noise distribution (typically bivariate normal with known or specified variance ratio).

**Optimization criterion**: maximum likelihood under the assumed noise model. For bivariate normal noise, this reduces to minimizing the sum of perpendicular squared residuals (when `lambda = Var(y)/Var(x) = 1`) or weighted-perpendicular squared residuals (general `lambda`).

**Members in the suite**: Deming regression (v0.6.219).

**Defining property**: same x/y symmetry as family 2, but with an additional knob (`lambda`) that lets the analyst encode prior knowledge about the noise ratio. As `lambda → ∞`, Deming reduces to OLS (back to family 1). As `lambda → 0`, Deming reduces to inverse-OLS-on-y (also family 1, just with the axes swapped). At `lambda = 1`, Deming is orthogonal regression / total least squares.

The defining advantage of parametric EIV over non-parametric EIV is **statistical efficiency**. If the noise really is bivariate normal with the assumed `lambda`, Deming is the maximum-likelihood estimator and is asymptotically efficient — no other unbiased estimator achieves a smaller variance. Passing-Bablok, being non-parametric, pays a roughly 64% efficiency tax (the same ARE as the median vs. the mean) for its distribution-freeness.

The defining disadvantage of parametric EIV is **the parametric assumption**. If the noise is heavy-tailed or skewed, Deming will be biased. Passing-Bablok will not. This is the same robustness-vs-efficiency tradeoff that distinguishes the median from the mean, and it is the reason a well-equipped suite ships **both** family-2 and family-3 estimators rather than picking one.

The CHANGELOG entry for v0.6.219 calls Deming the "parametric EIV" estimator explicitly, and the unit-test file shipped in `34142fd` includes a test that verifies the `lambda → ∞` reduction to OLS exactly, the `lambda → 0` reduction to inverse-OLS exactly, and the `lambda = 1` reduction to total least squares. Those three reduction tests document the family-1 ↔ family-3 boundary cleanly.

## Why the taxonomy matters: reading slope estimates correctly

The point of this taxonomy is not classification for its own sake. The point is that **what a slope number means depends on which family produced it**. Three concrete consequences:

**1. A family-1 slope answers a different question than a family-2 or family-3 slope.** Family 1 answers: "if I controlled x and observed y, what is the conditional mean of y given x?" Family 2 and 3 answer: "what is the structural relationship between the underlying signals of x and y, after accounting for noise on both?" These are genuinely different questions. The codex live-smoke at v0.6.219 illustrated this dramatically: OLS slope `+267k tok/row` answers the first question (conditional expectation given the noisy x); Deming slope `+2225990 tok/row` answers the second (the underlying relationship). Both numbers are correct answers to the questions they actually answer; they only seem to disagree if you forget that they are answering different questions.

**2. Sign-flips are a family-disagreement signature, not an estimator bug.** When OLS and Deming disagree on the sign of a slope (as opencode did at v0.6.219, with OLS `-7510` vs Deming `-1183143` — same sign, but the order-of-magnitude gap is the more dramatic version of the same phenomenon), the disagreement is structural. It is not that one estimator is "more accurate." It is that they are computing different things. The Passing-Bablok sign-flip cohort observed at v0.6.218, where claude code and hermes flipped relative to OLS, was the first instance of this in the suite. v0.6.219 confirmed that the Deming estimator agrees with Passing-Bablok on the family-2/3 question, not with OLS on the family-1 question.

**3. Robustness is orthogonal to family.** Within each family, you can have non-robust estimators (OLS in family 1, vanilla Deming in family 3) and robust estimators (Theil-Sen and the M-estimator suite in family 1, Passing-Bablok in family 2). The robustness axis runs **across** families. This is why the suite has a Cartesian-product structure: ~14 family-1 estimators × varying robustness, 1 family-2 estimator (robust by construction), 1 family-3 estimator (non-robust at the moment; a robust Deming would be a future addition).

## The release sequence as a deliberate taxonomic completion

Looking at the calendar:

- v0.6.207-v0.6.217 (eleven releases over roughly seven days): the M-estimator march, all family 1, all about robustness within family 1.
- v0.6.214: Theil-Sen confirmation. Family 1, robust, non-parametric.
- v0.6.216: Siegel repeated medians. Family 1, robust, non-parametric, 50% breakdown.
- v0.6.218: Passing-Bablok. **Opens family 2.** First x/y-symmetric estimator in the suite.
- v0.6.219: Deming. **Opens family 3.** First parametric EIV estimator in the suite.

The two-release jump from v0.6.218 to v0.6.219, opening families 2 and 3 in successive releases, is the taxonomic completion move. After v0.6.217, the suite was a richly-populated family 1 with no family 2 or 3 members. After v0.6.219, it has at least one canonical member of each family. The release notes for v0.6.219 made this explicit by referencing the "EIV pair" — Deming and Passing-Bablok — as a deliberate complement.

This is how you know the taxonomy was intentional rather than emergent: the release sequence cleanly opens family 2 in one release and family 3 in the next, and the v0.6.219 notes retroactively name the structural relationship. If the releases had been ordered differently — say, Deming first and Passing-Bablok second — the structural story would have been less clean (you would have had to motivate the parametric-then-non-parametric ordering). The actual ordering is the right one: non-parametric first (more conservative assumption), parametric second (additional efficiency at the cost of an additional assumption).

## What this means for the next ~10 releases

The taxonomy predicts the next plausible additions:

**Family 1 extensions**: there is still room for more M-estimator variants if anyone cares (Bisquare, redescending Huber, modified Hampel) but these are diminishing-returns refinements. The M-estimator march has plausibly converged.

**Family 2 extensions**: the most obvious gap is a Spearman-rank-based EIV estimator. This would be the rank-based analog of Passing-Bablok and would extend the family from 1 to 2 members.

**Family 3 extensions**: weighted Deming (per-point variances), bootstrap confidence intervals for Deming (currently a point estimate only), and Linnet's MLE for `lambda` itself (estimating the variance ratio from data rather than requiring the analyst to specify it). These are all listed as follow-ups in the v0.6.219 release notes.

**Cross-family**: a robust Deming variant (Deming with M-estimator-style residual weighting) would be a family-3 estimator with family-1-style robustness properties. This is the "Cartesian product" cell that is currently empty.

If the suite ships even three or four of these in the next two weeks, the taxonomy will have moved from "minimum one per family" to "robust coverage across the family-1 × robustness-axis × family-2/3-axis grid." That would be the v0.6.230-or-so milestone.

## Summary

The pew-insights suite as of v0.6.219 supports three structural families of slope estimation:

| Family | Assumption about noise | Members | Defining property |
|---|---|---|---|
| 1: y-asymmetric | x is exact; noise in y only | OLS, Theil-Sen, Siegel, M-estimator suite (~14 total) | `m_yx · m_xy ≠ 1` |
| 2: non-parametric EIV | noise in both, no parametric form | Passing-Bablok | `m_yx · m_xy = 1`, no likelihood |
| 3: parametric EIV | noise in both, bivariate-normal form | Deming | `m_yx · m_xy = 1`, MLE under noise model |

The thing to check, when reading any slope number from the suite, is which family produced it. A family-1 slope and a family-3 slope on the same data can disagree by a factor of 8 or 158 (as the v0.6.219 codex and opencode live-smoke showed) without either being wrong. They are answering different questions. The suite ships representatives of all three families now, which means the analyst has the freedom — and the responsibility — to pick the family that matches the question being asked.

That is the v0.6.219 release in one sentence: **the suite is now structurally complete with respect to slope-estimator families**, and the next ten releases will fill out the cells of the resulting Cartesian product rather than open any new families.
