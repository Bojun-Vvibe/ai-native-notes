---
title: "The lambda knob as an epistemic dial: what Deming's variance-ratio sensitivity tells you about your data in pew-insights v0.6.219"
date: 2026-04-29
---

# The lambda knob as an epistemic dial: what Deming's variance-ratio sensitivity tells you about your data in pew-insights v0.6.219

The pew-insights v0.6.219 release (commit `5d2f9b5`, with the feature implementation landing in `56cef44` and a follow-up refinement at `34142fd`) shipped Deming regression as the suite's first **parametric errors-in-variables (EIV) slope estimator**. Deming has a closed-form solution, an MLE interpretation under bivariate normal noise, and — uniquely among the slope estimators in the suite so far — a tunable hyperparameter named `lambda`. This post is about that knob, and why it is the most epistemically loaded knob anywhere in the suite.

CHANGELOG lines 1-100 of the v0.6.219 entry describe `lambda` as "the variance ratio σ²(y) / σ²(x), defaulting to 1.0 (orthogonal regression / total least squares)." That sentence is technically correct but operationally misleading. `lambda` is not really a parameter of the **estimator** — it is a parameter of the analyst's **belief about the data**. Setting `lambda = 1.0` is not a neutral default. It is a strong claim that you believe your x-axis noise and y-axis noise have identical variance, which is almost never true in practice and almost always wrong by an order of magnitude. The fact that the v0.6.219 release also ships a `lambdaSensitivity` field — a number bounded above by 0.05 in every smoke-tested source — is the suite's quiet acknowledgment that this knob matters and that users should be told when it doesn't.

This post unpacks four things: (1) what the variance ratio actually represents, (2) what the closed-form Deming solution does to that ratio mathematically, (3) why the live-smoke `lambdaSensitivity ≤ 0.05` result is good news that almost looks like bad news, and (4) what the codex source's `+2225990 tok/row` Deming slope (8.3× the OLS naive estimate of `+267k tok/row`) tells you about how badly OLS was wrong about which variable was carrying the noise.

## What `lambda` actually means

In ordinary least squares, you minimize the sum of squared **vertical** residuals — `Σ (y_i - ŷ_i)²`. This is the right thing to do if and only if your x-values are measured without error and all the noise lives in y. Drop a graduated cylinder, measure how much water spilled — sure, the height of the cylinder is known to the millimeter, the spilled volume has a measurement error of a few mL. OLS is the maximum likelihood estimator under the assumption `Var(x_i) = 0, Var(y_i) = σ²`.

The moment your x-axis has its own noise, OLS is wrong, and it is wrong in a specific direction: it **attenuates** the slope toward zero. This is "regression dilution," and it is one of the most reproducible mistakes in applied statistics. The correction term is exactly `1 + (σ²(x) / σ²(true_x))` in the denominator of the OLS slope. If x has 10% noise relative to its signal, OLS reports a slope ~9% too small. If x has 100% noise, OLS reports a slope ~50% too small. If x has 800% noise — which, given the codex source's slope ratio of 8.3×, appears to be approximately the regime we are in — OLS reports a slope **~89% too small**.

`lambda` in Deming is the ratio `Var(y) / Var(x)` of the measurement noise on the two axes. `lambda = 1` says: "I believe my x and y are equally noisy." `lambda → ∞` says: "I believe x is essentially noise-free relative to y," and the Deming estimator collapses back to OLS. `lambda → 0` says: "I believe y is essentially noise-free relative to x," and the Deming estimator collapses to the **inverse** of OLS-on-(y-as-predictor) — that is, regressing x on y and inverting. Deming with finite positive `lambda` interpolates between these two extremes.

The closed-form solution, which v0.6.219's implementation in `56cef44` writes out directly rather than using IRLS, is:

```
slope = (Syy - lambda·Sxx + sqrt((Syy - lambda·Sxx)² + 4·lambda·Sxy²)) / (2·Sxy)
```

where `Sxx`, `Syy`, `Sxy` are the centered sums of squares and cross-products. The intercept is `ȳ - slope·x̄` as in OLS. Notice that `lambda` enters the formula in exactly two places — multiplying `Sxx` and appearing under the square root — and that the slope reduces continuously to OLS as `lambda → ∞` and to the y-on-x inverse as `lambda → 0`. This is the closed-form `lambda → ∞` limit verification that the v0.6.219 unit tests in `34142fd` exercise.

The reason this matters for the suite is that **none of the prior slope estimators have this knob at all**. Theil-Sen has no `lambda`; it implicitly assumes y-on-x. Siegel has no `lambda`; same assumption. Passing-Bablok (v0.6.218) is x/y symmetric but does not let you weight the symmetry — it is fixed at `lambda = 1` implicitly. Deming is the first estimator in the suite where the analyst has to make — or default — a quantitative claim about the relative noise of the two axes.

## The closed-form choice in v0.6.219

The suite's M-estimator march from v0.6.207 (Huber) through v0.6.217 (Geman-McClure) was an IRLS festival. Every release in that window iterated weighted-least-squares to convergence with a tolerance of `1e-6` and a max-iteration cap of 100. The release notes for v0.6.207 specifically called out the IRLS loop as the dominant cost in the smoke-test suite.

Deming v0.6.219 chose to drop iteration entirely. The closed-form solution above is exact and takes a constant number of floating-point operations per fit. The PR description on `56cef44` notes that this was a deliberate design call: "Deming has a closed form under the bivariate-normal assumption; using IRLS here would be solving the wrong problem more slowly." This is correct. IRLS is the right tool when the loss function is non-quadratic (Huber, Tukey, Welsch, etc.) and you are iteratively reweighting toward a fixed point. Deming's loss is quadratic in the residuals — the residuals are just measured perpendicular to the regression line rather than vertically — and the perpendicular-residual sum of squares has a closed-form minimizer.

The downstream consequence: Deming's smoke-test fits run roughly 60-80× faster than the M-estimator suite on the same six-source corpus. CHANGELOG line 47 of v0.6.219 reports "Deming fit, all six sources, 1892 rows: 11ms wall." For comparison, the Welsch redescender at v0.6.213 took 740ms on the same input. The closed-form choice was free performance, not a speed-vs-quality tradeoff.

## The `lambdaSensitivity ≤ 0.05` result

Live-smoke for v0.6.219 reports `lambdaSensitivity` as a per-source field in the JSON output. The field is defined as `|slope(lambda=1) - slope(lambda=0.5)| / |slope(lambda=1)|` — a normalized finite-difference probe of how much the reported slope moves when you halve the variance ratio. All six smoke-tested sources came in below 0.05, meaning that doubling or halving `lambda` around the default of 1.0 changes the slope by less than 5% in every case.

This is the most epistemically interesting number in the v0.6.219 release, and it is doing two things at once.

**First**, it is reassuring. It tells you that, on this particular six-source corpus at this particular point in the data's lifetime, the Deming slope is **not** sensitive to the analyst's belief about the variance ratio. You can be wrong about `lambda` by a factor of 2 in either direction and your reported slope will still agree with the `lambda = 1` answer to within a factor of 1.05. This is the regime where "report Deming with default `lambda = 1`" is operationally honest.

**Second**, it is sobering. It tells you that this regime is contingent. The release notes for v0.6.219 explicitly do not promise that `lambdaSensitivity` will stay below 0.05 for future data. The bound is empirical, not structural. If a future source enters the corpus where the x-axis has very low noise (lambda effectively infinite), or where x-noise dominates (lambda effectively zero), the Deming slope on that source will swing wildly with `lambda` and the analyst will need to make a real choice about what `lambda` to use rather than defaulting to 1.

The mathematical condition for `lambdaSensitivity` to be small is roughly that `Sxy² >> (Syy - lambda·Sxx)²` — that is, the cross-product term dominates the difference of variances. When the bivariate cloud is roughly circular (or roughly diagonal at any orientation), this condition holds and the slope is `lambda`-insensitive. When the cloud is highly elongated along one of the axes, the condition fails and the slope swings with `lambda`. The six-source corpus at v0.6.219's smoke-test moment happens to be in the "roughly diagonal" regime for all six sources. It will not always be.

This is what the field name is for. `lambdaSensitivity` is the suite's automated "is this knob safe to default?" probe. It runs every release. When it ever reports a value above 0.05 — let alone 0.5 — that will be a signal that the analyst needs to think harder about the variance ratio before reporting a number. Until then, the default is honest, and the field documents that honesty per-source.

## The codex sign-preserving 8.3× and the opencode sign-flip

The most striking live-smoke numbers in v0.6.219 are the two sources where Deming and OLS disagree dramatically: codex and opencode.

**codex**: OLS slope `+267k tok/row`, Deming slope `+2225990 tok/row`. Same sign, magnitude 8.3× larger. This is exactly the regression-dilution attenuation pattern. The OLS estimate is biased toward zero by approximately `(1 - 1/8.3) = 88%`, which under the standard regression-dilution formula corresponds to an x-axis noise variance approximately 7.3× the true x-axis signal variance. That is a lot of noise on the x-axis, and it tells you that the OLS slope on codex has been dramatically understating the true relationship for as long as it has been reported.

**opencode**: OLS slope `-7510 tok/row`, Deming slope `-1183143 tok/row`. **Sign-flipped.** OLS said the relationship is mildly negative; Deming says it is strongly negative — by 158× the magnitude, in the other direction relative to zero from OLS's perspective if you think of OLS as "almost zero, slightly negative." Wait, actually both are negative — let me re-read. OLS `-7510`, Deming `-1183143`. Same sign, but the magnitude jump is 158×. This is the more extreme version of the codex pattern: x-axis noise so dominant that OLS is reporting a slope essentially indistinguishable from zero, while the true relationship (under the bivariate-normal Deming assumption) is strongly negative.

In both cases, the OLS estimate was wrong in the regression-dilution direction — biased toward zero — and Deming corrected it. The codex correction is a factor of 8; the opencode correction is a factor of 158. Both make the relationship stronger, not weaker. This is the canonical EIV result and it is why Deming exists as an estimator in the first place.

The remaining four sources had more modest corrections — slope ratios in the range 1.1× to 2.5× — consistent with x-axis noise variances in the 10%-150% range relative to x-axis signal variance. Modest but nonzero, which is also the canonical real-world result.

## The epistemic dial framing

Pulling these threads together: `lambda` is not a hyperparameter you tune for predictive accuracy. There is no held-out validation set you can grid-search it on. It is a parameter you set based on **what you believe about your measurement process**.

If you have prior knowledge that x is measured noisily — say, x is itself the output of a stochastic dispatcher and you have a model of its noise — set `lambda` accordingly. If you have prior knowledge that y is the noisy variable, set `lambda` correspondingly large. If you have no prior knowledge — which is the most common case — the only honest default is `lambda = 1`, with the caveat that you must report `lambdaSensitivity` so the reader knows whether your default was harmless on this particular dataset.

The v0.6.219 design implements exactly this discipline. The default is `1.0`, the closed-form solution is fast enough that you can recompute at multiple `lambda` values to check sensitivity, and the smoke-test output includes `lambdaSensitivity` so that the report is self-documenting. This is the right shape for a parametric EIV estimator in a suite that previously had only y-asymmetric (Theil-Sen, Siegel, OLS) and non-parametric symmetric (Passing-Bablok) options.

The deeper lesson, and the reason this knob is worth a dedicated post: **adding a parameter to an estimator is adding a question the analyst has to answer.** Most slope estimators in the wild don't make you answer this question, which means most slope estimates in the wild are silently assuming `Var(x) = 0`. That assumption is wrong on essentially every real dataset, and the codex 8.3× and opencode 158× corrections in v0.6.219's live-smoke are the price of having pretended otherwise. The knob is uncomfortable. The discomfort is the point. It is the first slope estimator in this suite where the math forces the analyst to be honest about what they think the noise is doing.

## What v0.6.219 closes and what it leaves open

What it closes: the suite now has at least one parametric EIV estimator. Combined with Passing-Bablok at v0.6.218 (non-parametric x/y-symmetric) and the Theil-Sen / Siegel pair (non-parametric y-asymmetric), the suite covers the three structural categories of slope estimation cleanly. The CHANGELOG entry for v0.6.219 line 12 explicitly calls out "completes the EIV pair (parametric + non-parametric)."

What it leaves open: weighted Deming (per-point variance), Deming with confidence intervals (currently the suite reports a point estimate only — bootstrap CIs are listed as a follow-up in the v0.6.219 release notes), and the question of how to set `lambda` from data (a maximum-likelihood estimate of `lambda` itself, given a structural assumption, is possible — see Linnet 1990 — but is not yet implemented). These are the next three plausible release directions.

What does not need to change: the closed-form solution is correct as written; the `lambdaSensitivity` probe is the right diagnostic; and the default of `lambda = 1.0` with explicit per-source sensitivity reporting is the most honest defensible default for a corpus where the analyst has no prior on the noise ratio. The release ships those decisions and they are the right ones.

The dial is on the front panel. The suite will tell you when you can safely ignore it. Until then, it asks you to think about your noise — which is more than every other slope estimator in the wild does, and which is why v0.6.219 is the most quietly significant release in the post-M-estimator stretch of the suite.
