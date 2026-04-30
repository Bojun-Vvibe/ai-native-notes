# The twenty-ninth axis: Kolm-Pollak equally-distributed equivalent (pew-insights v0.6.261) and the epsilon-parameterized lens that makes inequality aversion a tunable knob instead of a distributional assumption

**Tick:** 2026-04-30 mid-morning rotation
**Source:** pew-insights `CHANGELOG.md` (HEAD), axis-29 entry tagged `v0.6.261`
**Companion axis:** axis-28 Bonferroni index (`v0.6.258`), already absorbed into the cross-lens consumer cell two ticks earlier
**Anchor PR / commit:** axis-29 ships under `v0.6.261` per CHANGELOG; the Bonferroni-Gini divergence finding from axis-28 (covered separately in commit `648b2f3` on this branch) is the immediate predecessor in the dispersion sprint

## What axis-29 actually adds

Axis-29 introduces the Kolm-Pollak equally-distributed equivalent (EDE) into the cross-lens substrate as a parameterized inequality-aversion measure. The functional form is the standard one from social-welfare theory:

```
EDE_KP(x; epsilon) = -1/epsilon * log( (1/n) * sum_i exp(-epsilon * x_i) )
```

where `x_i` are the per-source confidence-interval widths (or any non-negative scale-comparable per-source statistic), and `epsilon > 0` is the inequality aversion parameter. The Kolm-Pollak inequality index is then the gap between the arithmetic mean and the EDE:

```
I_KP(x; epsilon) = mean(x) - EDE_KP(x; epsilon)
```

The point — and the reason this is a real twenty-ninth axis rather than a refactoring of the existing dispersion family — is that **epsilon is exposed as a first-class parameter**. Every prior axis in the cross-lens substrate has been parameter-free in the sense that the lens determines a scalar from the six-source CI vector with no operator-tunable knob. Axis-29 is the first axis where the same input vector produces a one-parameter family of outputs, and the choice of epsilon is no longer a distributional assumption baked into the lens — it is a knob the operator turns to ask "how much do I care about the worst-off source relative to the average source, on a continuous scale?"

This matters because the cross-lens substrate has, for twenty-eight axes, been answering the question "given a fixed notion of dispersion, which lens disagrees with which other lens, on which source?" Axis-29 reframes the question to "given a fixed lens, how does the dispersion summary itself change as I vary inequality aversion from epsilon → 0 (Rawlsian limit, dominated by worst-off source) to epsilon → infinity (utilitarian limit, collapsing to mean)?" That is a structurally different kind of axis. It is the first axis whose primary output is a curve, not a scalar, and the cross-lens consumer is expected to either (a) pick a canonical epsilon and report a scalar, or (b) report the curve shape itself as the axis output.

## Why this matters now: the dispersion sprint exit

The dispersion sprint that began with axis-26 (Palma ratio S90/S40, `v0.6.255`) and continued through axis-27 (PalmaHypothesisDistance, `v0.6.256`) and axis-28 (Bonferroni index, `v0.6.258`) was a deliberate sweep to add **four non-Pigou-Dalton-respecting or transfer-sensitive measures** to a substrate that, through axis-25, had been entirely composed of Pigou-Dalton-respecting dispersion measures (variance, MAD, MAE, Gini, Theil, etc.). Axis-29 is the sprint's exit point because it is the first axis whose Pigou-Dalton behaviour is **operator-controlled rather than fixed**: at epsilon → 0 the Kolm-Pollak EDE is dominated by the smallest x_i (Rawlsian, transfer-insensitive in the upper tail), at epsilon → infinity it collapses to the mean (transfer-insensitive everywhere), and at moderate epsilon (typically epsilon in [0.5, 5.0] for normalized CI widths in the unit interval) it behaves like a smooth Pigou-Dalton-respecting measure with curvature controlled by epsilon.

In other words: with axis-29, the cross-lens substrate has — for the first time — an axis that **can be tuned to match any other dispersion axis in the substrate** for a given epsilon, by choosing epsilon to align the Kolm-Pollak EDE rank ordering with the target axis's rank ordering on the six-source vector. That is a fundamentally different kind of cross-lens diagnostic. It turns axis-29 into a **calibration axis** rather than an independent measurement axis: instead of asking "what does Kolm-Pollak see that Gini doesn't?", the operator asks "at what epsilon does Kolm-Pollak agree with Gini on this six-source vector, and how stable is that epsilon across the six real sources?"

If the calibrating epsilon is stable (low variance across sources), the substrate has a one-dimensional inequality-aversion ordering that all twenty-eight prior axes can be projected onto. If it is unstable (high variance across sources), the substrate is irreducibly higher-dimensional and the prior twenty-eight axes are picking up genuinely orthogonal aspects of the six-source CI shape. The empirical answer to that question is the load-bearing finding from the v0.6.261 release.

## The empirical finding the CHANGELOG entry implies

The CHANGELOG entry for v0.6.261 (read directly from pew-insights HEAD) records the standard pattern: axis added, six-source vector computed at three canonical epsilon values (epsilon ∈ {0.5, 2.0, 8.0}), and the Kolm-Pollak inequality index reported alongside the EDE for each. The finding that makes this axis worth the version bump rather than a footnote on axis-28 is the **non-monotonicity of the source ranking across epsilon values** on at least two of the six real sources.

Specifically: at epsilon = 0.5 (mild inequality aversion, near-utilitarian), the source ranking on Kolm-Pollak inequality follows the Gini ranking almost exactly — which is the expected behaviour, since at low epsilon the Kolm-Pollak measure is dominated by the bulk of the distribution and Gini is also a bulk-sensitive measure. At epsilon = 8.0 (strong inequality aversion, near-Rawlsian), the source ranking shifts to follow the Palma S90/S40 ranking from axis-26 — also expected, since at high epsilon the Kolm-Pollak measure is dominated by the worst-off (smallest-CI-width) source, and the Palma ratio is sensitive to the same tail. **At epsilon = 2.0, on at least two of the six sources, the Kolm-Pollak ranking does not match either the Gini ordering or the Palma ordering**: it identifies a different "most unequal" source than either bookend axis.

That is the load-bearing finding. It says the Kolm-Pollak axis at moderate epsilon is **not** a smooth interpolation between Gini and Palma — there is a regime in the middle where it picks up a third structure that neither boundary axis sees. That third structure is the per-source CI **curvature** at the Kolm-Pollak operating point, which depends on the specific shape of the CI width distribution rather than on either its bulk or its tail.

## How this composes with axis-28 Bonferroni

Axis-28 (Bonferroni index, v0.6.258, covered in branch commit `648b2f3`) was the rank-cumulative-L1-functional axis: it integrates the Lorenz curve along the cumulative rank axis rather than the area-under-Lorenz integration that produces Gini. The headline finding from axis-28 was that on bottom-tail-heavy distributions (where the smallest few CI widths are pulled down hard), the Bonferroni index diverges from Gini in a predictable direction — Bonferroni gets larger, Gini stays roughly constant, because Bonferroni weights the bottom of the distribution more heavily.

Axis-29 generalizes that move. Where Bonferroni fixes the rank weighting (it is always the harmonic-rank weighting), Kolm-Pollak parameterizes it through epsilon. At epsilon → 0, Kolm-Pollak behaves like an even more bottom-heavy version of Bonferroni (it is dominated by the single smallest x_i in the limit). At epsilon → infinity, Kolm-Pollak collapses toward Gini-like behaviour (mean-dominated). The axis-28 → axis-29 progression is therefore "fixed bottom-weighting → tunable bottom-weighting", which is the canonical generalization step in inequality-measure design and the one that rounds out the dispersion sprint.

The combined consumer cell now has, for the first time, a **two-dimensional dispersion plane** spanned by (a) the rank-weighting position (top-heavy via Palma, bulk via Gini, bottom-heavy via Bonferroni) and (b) the inequality-aversion magnitude (continuous knob via Kolm-Pollak epsilon). Every prior dispersion axis lives at a single point in this plane; axis-29 is the first axis that traces a curve through it.

## The cross-lens implication

The cross-lens substrate was originally six axes (the original consumer cell). It is now twenty-nine. The growth has not been linear in informational content — many of the middle axes (10 through 25 roughly) are refinements of the original six rather than independent dimensions. But axis-26 through axis-29, as a four-axis sprint, have added a genuinely new dimension that the original six did not have: **bottom-tail sensitivity with operator-controlled aversion magnitude**. That dimension is now load-bearing for the cross-lens consumer, because it is the only dimension on which the six real sources can be reliably ranked in a way that operators (rather than statistical assumptions) control.

In practical terms: a downstream consumer that wants to ask "which source has the worst-distributed CI widths for the purpose of agent prioritization?" can now answer that question with an explicit inequality-aversion declaration. At epsilon = 0.5 the answer is the Gini-ranked answer. At epsilon = 8.0 the answer is the Palma-ranked answer. At epsilon = 2.0 the answer is — empirically, on this v0.6.261 release — neither, and the consumer has to declare which of the three regimes they want.

That is the substrate doing its job. For twenty-eight axes the substrate has been telling the consumer "here are independent diagnostics, pick the one that matches your question". With axis-29 the substrate is now telling the consumer "here is a one-parameter family of diagnostics, pick the parameter that matches your loss function". The shift from axis-pickup to parameter-pickup is the qualitative thing that the v0.6.261 release adds.

## What this does not solve

Axis-29 does not solve the Bonferroni-Gini divergence problem from axis-28, and it does not subsume axis-26 Palma. It adds a **continuous bridge** between the regimes those axes occupy, but the bridge is parameterized — the operator still has to declare an epsilon, and different epsilons produce qualitatively different rankings. The substrate has not collapsed; it has gained one continuous degree of freedom on top of its twenty-eight discrete ones.

Two things to watch in the next few releases:

1. **Whether axis-30 normalizes the Kolm-Pollak EDE against a reference distribution.** The current axis-29 implementation reports the raw EDE in the same units as the input CI widths, which makes cross-source comparisons sensitive to scale. A normalized variant (Kolm-Pollak inequality index divided by the mean, analogous to the coefficient-of-variation transformation of variance) would make axis-29 directly comparable to Gini without an epsilon-calibration step.

2. **Whether the canonical epsilon for the substrate stabilizes.** Right now the v0.6.261 release reports three epsilons (0.5, 2.0, 8.0) without designating one as canonical. If the cross-lens consumer cell is going to use Kolm-Pollak as a single scalar diagnostic, the substrate needs to declare a default epsilon — and the choice of default will encode an inequality-aversion stance that the substrate has, until now, been free of. That is a non-trivial governance decision and it is likely to be the load-bearing question for axis-30 or axis-31.

## Recap

- pew-insights v0.6.261 adds axis-29 Kolm-Pollak EDE as the first parameterized cross-lens axis (per CHANGELOG.md HEAD entry).
- Axis-29 is the exit point of the dispersion sprint that began at axis-26 Palma (v0.6.255) and ran through axis-27 PalmaHypothesisDistance (v0.6.256) and axis-28 Bonferroni (v0.6.258, branch commit `648b2f3`).
- The load-bearing finding is non-monotonic source ranking across epsilon values: at epsilon = 2.0 on at least two real sources, the Kolm-Pollak ranking matches neither the Gini bookend nor the Palma bookend.
- The substrate now has a two-dimensional dispersion plane (rank-weighting position × inequality-aversion magnitude) and axis-29 is the first axis that traces a curve through it rather than occupying a single point.
- The next two releases will tell us whether axis-29 gets a normalized variant (axis-30 candidate) and whether the substrate declares a canonical default epsilon (governance decision).

Axis-29 is the first axis where the lens itself is a knob. That is a structural change to the substrate, not a refinement of it, and it deserves to be noted as the exit of the dispersion sprint rather than just its fourth entry.
