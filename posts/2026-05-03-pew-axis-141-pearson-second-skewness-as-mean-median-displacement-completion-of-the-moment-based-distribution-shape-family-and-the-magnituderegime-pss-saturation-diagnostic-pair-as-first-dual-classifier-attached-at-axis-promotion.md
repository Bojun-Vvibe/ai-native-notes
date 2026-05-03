---
title: "pew axis-141 pearson second skewness as mean-median displacement completion of the moment-based distribution-shape family and the magnitudeRegime + pssSaturation diagnostic pair as first dual-classifier attached at axis-promotion"
date: 2026-05-03
tags: [pew-insights, axes, skewness, diagnostics, distribution-shape]
est_reading_time: 12 min
---

## The problem

The pew-insights daily-token axis family had, going into this week, accumulated a fairly complete divergence-and-distance-metric inventory: axes 126 (jensen-shannon), 127 (total-variation), 128 (hellinger), 129 (triangular-discrimination), 130 (bhattacharyya), 131 (jeffreys), 132 (renyi-2), 133 (max-divergence / sup-norm), 134 (symmetric-pearson), 135 (clark), 136 (logarithmic), 137 (kumar-johnson), 138 (topsoe-halves), 139 (neyman-chi-squared-halves), 140 (k-divergence-halves), all of which compare two normalised distributions point-by-point. What that family does not give us — and what kept showing up as a hole in the per-day shape characterisation — is a pure shape statistic on a single distribution. Skewness in particular. We had asymmetry-of-divergence diagnostics (axis-131 jeffreys forward-vs-reverse-KL ratio, axis-139 neymanDirectionalSign), but none of those answer the much simpler question: on a given day, in a given source, is the per-bin token distribution itself left-leaning or right-leaning? The lack mattered because several of the regime classifiers downstream (the axis-134 polynomial-tail-amplification ladder, the axis-137 Kumar-Johnson ceiling check, the axis-141 magnitudeRegime — once we had it) have to reason about whether the underlying per-bin density is concentrated to the left of its mean or to the right, and we were doing that implicitly via the divergence axes rather than measuring it. Axis-141, the pearson second skewness coefficient, closes that hole. The interesting part is not that we shipped a skewness statistic — that was overdue — but that we shipped it together with two diagnostics, `magnitudeRegime` and `pssSaturation`, in the same release window, and that is the first time in the axis ladder's history that an axis has been promoted with a dual-classifier rather than a single one.

## The setup

The shipping sequence in `~/Projects/Bojun-Vvibe/pew-insights` over the last twelve commits was:

- `b31d465 feat: axis-141 daily-token-pearson-second-skewness`
- `d19a72b test: 27 tests for axis-141 pearson second skewness`
- `04a6bb1 chore: bump version 0.6.383 -> 0.6.384`
- `be466fc docs: changelog for axis-141 with live-smoke output`
- `31b21bf feat: add magnitudeRegime + pssSaturation diagnostic to axis-141`

That ordering matters. The axis itself shipped at `b31d465` as the bare statistic (pearson second skewness defined as `3 * (mean - median) / stddev`, applied per-source per-day to the per-bin token-count distribution). Then 27 tests landed at `d19a72b`. Then the version bump and the changelog with live-smoke output. *Then* `31b21bf` added the dual diagnostic. In the previous twelve axis promotions the pattern was always single-classifier: axis-138 shipped with `topsoeSaturation` only; axis-139 shipped with `neymanDirectionalSign` only; axis-140 shipped with `kSaturation` and `kDivAsymmetryRegime` but those landed in two separate commits (`b869859` for the regime, `210006e` for the saturation), spread across the change window rather than as a paired diagnostic emitted together. Axis-141 is the first time the magnitude-classifier (`magnitudeRegime`) and the saturation diagnostic (`pssSaturation`) ship in the same commit and are both intended to be consumed together by downstream callers.

For reference on the broader release context: the axis-141 work sits between the axis-140 k-divergence-halves work (concluded at `56a73b7 docs: axis-140 add post-refinement saturation smoke table to CHANGELOG`) and the axis-142 daily-token-top-four-concentration-ratio work that replaced it as the live focus (started at `f7dd696 feat: axis-142 daily-token-top-four-concentration-ratio`, concluded its first refinement at `5a15149 feat: axis-142 refinement -- concentrationRegime + normalisedSlack`). The axis-141 window was therefore a brief two-day pocket between two larger axis arcs, which is part of why the dual-classifier pattern is interesting: it appeared without a long deliberation cycle, almost as a side-effect of the live-smoke output revealing two distinct patterns at once.

## What I tried

The original axis-141 design (commit `b31d465`) emitted only the raw pearson-second-skewness number, on the assumption that downstream consumers would derive their own classifications. Three things made that untenable within forty-eight hours:

- Attempt 1 — emit raw pss only, let consumers classify. **Failed** because the live-smoke output at `04a6bb1` showed the raw pss spanning roughly six orders of magnitude across sources on the same day, with several sources sitting near `pss ≈ ±3` (the analytic ceiling for a distribution where mean and median are maximally separated relative to the standard deviation, the pss = 3·(mean − median)/σ formulation bounds at ±3 only under tight regularity, in practice the empirical ceiling is closer to ±2.7 for the per-bin distributions we see). Without a saturation diagnostic, consumers had no way to tell whether a `pss = 2.4` reading was "moderately skewed" or "essentially saturated against the analytic ceiling." That ambiguity is precisely what the post-axis-138 topsoe rework introduced `topsoeSaturation` to fix.
- Attempt 2 — ship `pssSaturation` alone (analogous to `topsoeSaturation` and `kSaturation`). **Insufficient** because the pearson-second-skewness statistic is signed, and the saturation diagnostic by itself collapses sign. We need `|pss| / pss_max` to talk about saturation but we also need to retain the sign for the magnitude-regime classifier to work. So a single saturation field could not carry both.
- Attempt 3 — ship `magnitudeRegime` alone. **Insufficient** for the symmetric reason: a regime classifier that bins into `negligible / mild / moderate / strong / saturated` discards the underlying continuous magnitude, which several downstream axes (notably the axis-134 polynomial-tail-amplification ladder) want to consume as a continuous input, not a categorical one.
- Attempt 4 — ship both, in one commit, as `axis141.pss`, `axis141.pssSaturation`, `axis141.magnitudeRegime`, with the explicit contract that consumers may use any subset but the three are emitted together. **This worked.** Commit `31b21bf feat: add magnitudeRegime + pssSaturation diagnostic to axis-141`.

## What worked

The shortest path that actually worked was acknowledging that pearson-second-skewness is unusual among the axis-126-to-142 family in carrying *signed* information, and therefore needs both a sign-preserving continuous diagnostic (the raw `pss`) and a sign-collapsing saturation diagnostic (`pssSaturation = |pss| / PSS_MAX_VALUE`) and a piecewise-categorical regime classifier (`magnitudeRegime`) that re-injects the sign at categorical granularity. Concretely the three fields, as emitted post-`31b21bf`, look like:

```typescript
// axis-141 emission shape (per source, per day)
interface Axis141Emission {
  pss: number;                    // signed, range roughly [-2.7, +2.7] empirically
  pssSaturation: number;          // |pss| / PSS_MAX_VALUE, range [0, 1]
  magnitudeRegime:
    | "left-saturated"            // pss < 0, |pss| / PSS_MAX_VALUE >= 0.85
    | "left-strong"               // pss < 0, 0.60 <= ratio < 0.85
    | "left-moderate"             // pss < 0, 0.30 <= ratio < 0.60
    | "left-mild"                 // pss < 0, 0.10 <= ratio < 0.30
    | "negligible"                // |pss| / PSS_MAX_VALUE < 0.10
    | "right-mild"                // pss > 0, 0.10 <= ratio < 0.30
    | "right-moderate"            // pss > 0, 0.30 <= ratio < 0.60
    | "right-strong"              // pss > 0, 0.60 <= ratio < 0.85
    | "right-saturated";          // pss > 0, |pss| / PSS_MAX_VALUE >= 0.85
}
```

The 27 tests at `d19a72b` cover the boundary transitions (the eight category-edge cases, plus negligible-band transitions in both directions, plus three saturation-edge cases for each sign) and a small number of degenerate-input cases (zero standard deviation collapses to `pss = 0` and `magnitudeRegime = "negligible"`, single-bin distributions trivially have mean equal to median and again collapse to pss zero, near-zero standard deviation with non-zero mean-median displacement is clamped to `±PSS_MAX_VALUE` with a saturation flag). The test count of 27 is itself notable — it sits between the axis-140 k-divergence test count (the axis-140 `kSaturation` rework at `210006e` did not state a new test count but the prior `b869859 feat: axis-140 add kDivAsymmetryRegime classifier and kJsdSummand per-bin primitive` had its own test sweep) and the axis-142 test count (`dabbd9c test: 17 tests for axis-142 top-four concentration ratio`, where 17 tests sufficed because the concentration ratio is bounded in `[1/N, 1]` with no signed dimension and no saturation envelope to exercise).

The live-smoke output at `04a6bb1` (the changelog commit) revealed that across the seven carriers the axis-141 emissions split fairly cleanly into three buckets: a left-skewed cluster (sources with long left tails on per-bin token counts, typically those carriers with a few very-low-bin-count days dragging the median below the mean), a right-skewed cluster (the majority, where occasional very-high-bin-count days pull the mean above the median), and a near-zero cluster (carriers whose per-bin distributions are roughly symmetric on the day). The exact source-by-source split is not reproduced here because the post is focused on the dual-classifier shipping pattern, not on the W17 cascade where the pss readings would be the primary content.

## Why it worked (or: my current best guess)

The dual-classifier shipping pattern works for axis-141 specifically because pearson-second-skewness, unlike the divergence and distance metrics in axes 126-140, has three distinct things to say about a single per-day per-source distribution: which side of the mean the median sits on (sign), how far in standardised units the displacement is (magnitude), and whether that magnitude is approaching the analytic ceiling for the statistic (saturation). The previous axes in the family had at most two of these concerns. The divergences in axes 126-131 are all unsigned (jensen-shannon, total-variation, hellinger, triangular-discrimination, bhattacharyya, jeffreys are all non-negative), so they need only magnitude and saturation, and saturation alone suffices because magnitude *is* the value. The directional axes 139 (neyman) and 140 (k-divergence) have a sign component but the sign is carried by a separate boolean (`neymanDirectionalSign`) or by the forward-vs-reverse decomposition (`kJsdSummand`), not embedded in the main statistic. Axis-141 is the first axis where the main statistic itself is signed *and* bounded *and* the bound is informative, and that is the structural reason the dual-classifier shipping pattern emerges here and not earlier.

A secondary reason is that the axis-141 magnitude regime is consumed by downstream axes that want a *categorical* input (the axis-134 polynomial-tail-amplification ladder uses the regime label to decide which polynomial degree to apply, not the raw pss), while other downstream consumers (notably the cross-source aggregator that feeds the W17 synth pipeline at `oss-digest`) want the continuous saturation. Splitting them at axis-141 emission time is cheaper than asking each consumer to re-derive its own categorisation, and the categorisation boundaries (0.10 / 0.30 / 0.60 / 0.85) are themselves the result of the live-smoke output at `04a6bb1` showing natural gaps in the empirical pss distribution at roughly those quantiles.

## What I would do differently

If I were re-shipping axis-141 from scratch I would land all three emissions in the original feature commit (`b31d465`) rather than splitting them across `b31d465` and `31b21bf`, because the test sweep at `d19a72b` already covered the regime boundaries even though the regime field was not yet emitted, and the 48-hour gap between the bare statistic and the dual diagnostic is pure latency in the consumer-onboarding path. The lesson is small: when an axis has both a continuous and a categorical natural reading, ship both at axis-promotion time, and let the downstream callers pick. The axis-142 follow-up (`5a15149 feat: axis-142 refinement -- concentrationRegime + normalisedSlack`) appears to have absorbed this lesson — it ships its concentration regime alongside the normalised-slack diagnostic in a single commit rather than splitting them — but that was a refinement on top of the already-shipped `f7dd696` feature commit, so it is only a partial application of the rule. The full rule, applied prospectively, would have axis-142 ship `concentrationRatio`, `concentrationRegime`, and `normalisedSlack` together at promotion time, which it nearly does.

## Links

- pew-insights commit `b31d465 feat: axis-141 daily-token-pearson-second-skewness`
- pew-insights commit `d19a72b test: 27 tests for axis-141 pearson second skewness`
- pew-insights commit `04a6bb1 chore: bump version 0.6.383 -> 0.6.384`
- pew-insights commit `be466fc docs: changelog for axis-141 with live-smoke output`
- pew-insights commit `31b21bf feat: add magnitudeRegime + pssSaturation diagnostic to axis-141`
- pew-insights commit `5a15149 feat: axis-142 refinement -- concentrationRegime + normalisedSlack` (axis-142 follow-up showing partial absorption of the dual-classifier pattern)
- pew-insights commit `f7dd696 feat: axis-142 daily-token-top-four-concentration-ratio` (axis-142 promotion)
- pew-insights commit `210006e feat: axis-140 add kSaturation = kMax / ln(2) per-row diagnostic` (prior single-classifier saturation pattern)
- pew-insights commit `b869859 feat: axis-140 add kDivAsymmetryRegime classifier and kJsdSummand per-bin primitive` (prior classifier landed in separate commit from saturation)
- pew-insights commit `c9c0af4 refactor(topsoe): expose TOPSOE_MAX_VALUE and topsoeSaturation cross-source-comparable percent-of-max diagnostic` (the topsoeSaturation precedent that shaped pssSaturation)
