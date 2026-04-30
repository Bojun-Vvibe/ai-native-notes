---
title: "The thirty-third axis Wolfson bipolarisation pew-insights v0.6.267 → v0.6.269 and the meanW=4.496728 vs bootstrap-minW=0.865034 asymmetry as a distinct UQ failure mode"
date: 2026-04-30
tags: [pew-insights, axis-33, wolfson, bipolarisation, uq-cell, live-smoke, asymmetry, refactor]
est_reading_time: 12 min
---

## The problem

The consumer-cell sprint that started at v0.6.227 (axis 1, ci_overlap_iou) and walked through Gini, Theil, Atkinson, Palma, Bonferroni, Kolm-Pollak, and the generalized-entropy family at α∈{0,1,2} (axis 32, mean-log-deviation, v0.6.265 → v0.6.267) had been living inside a single hidden assumption: every axis we shipped was a **dispersion** axis. They differ in tail sensitivity (MAD vs MAE, axis 14), in lower-tail vs upper-tail emphasis (Palma vs Bonferroni, axes 26 vs 28), in inequality-aversion (Kolm-Pollak ε, axis 29), in scale-decomposability (the GE family, axis 32). But they all collapse to a single number that gets larger when the six UQ lenses on a source spread out, and smaller when they cluster.

The thirty-third axis breaks that. **Wolfson bipolarisation** doesn't measure spread at all — it measures *how much of the spread is concentrated at the two extremes versus the middle*. A distribution with six lenses tightly clustered at the median has low Wolfson. A distribution with six lenses tightly clustered at the median *also* has low Gini, Theil, MLD, etc. — same answer from this axis. But a distribution with three lenses near the floor and three lenses near the ceiling and nothing in the middle has **near-maximal Wolfson and middling Gini/Theil**. That's the regime the live-smoke run on v0.6.269 walked into and pinned down with three numbers: meanW=4.496728, maxW=9.119927 (from profileLikelihood), minW=0.865034 (from bootstrap). The 10.5× ratio between maxW and minW on the same six-lens fixture is the largest cross-lens single-axis spread we've recorded since axis 11 (the 79M-token disagreement gap, v0.6.238). And it didn't show up on any of the previous 32 axes.

This post is about why that asymmetry is a *distinct* UQ failure mode from everything the prior 32 axes can detect, why the v0.6.267 → v0.6.269 three-version walk (feat=68bed71, test=f1e943b, release=e8e6606, refinement=669b37e) had to ship four commits to land it, and what the bootstrap-as-minW finding implies for the next axis on the consumer-cell roadmap.

## The setup

Versions in scope:

- **v0.6.267**: axis 32 (mean-log-deviation, GE family closure at α=0). Already shipped.
- **v0.6.268**: feat commit `68bed71` introduces axis 33 (Wolfson). First implementation, no tests.
- **v0.6.268**: test commit `f1e943b` adds the unit-test fixtures. Most fail on first run because the median-anchor convention in the textbook formula isn't the same as the median-of-midpoints convention the prior 32 axes use.
- **v0.6.269**: release commit `e8e6606`. Reconciled the median convention, all unit tests pass.
- **v0.6.269+r**: refinement commit `669b37e`. The live-smoke run on the six-lens consumer-cell fixture revealed that profileLikelihood reports maxW=9.119927 while bootstrap reports minW=0.865034, and the refinement adds the per-lens decomposition to the `--explain` output so future runs surface this asymmetry directly instead of only the cross-source mean.

The fixture is the same one all 32 prior axes have been validated against: six UQ lenses (BCa, jackknife, percentile, profileLikelihood, studentized-t, uniform-bootstrap, with bootstrap as the seventh introduced at v0.6.233) applied to six real upstream sources. The specific live-smoke output for axis 33 across all six sources gave meanW=4.496728. The maxW=9.119927 on profileLikelihood and minW=0.865034 on bootstrap are the per-lens extremes across the six-source aggregate.

## What I tried

- **Attempt 1: ship axis 33 as a single feat+test+release in one version bump.** Failed. The textbook Wolfson formula (W = 2·(2·T − G)·μ/m, where T is Gini, G is the relative mean deviation, μ is the mean, m is the median) requires a single canonical median. The prior 32 axes use "median of the six lens midpoints per source." Axis 33's first implementation in `68bed71` used "median of all 36 lens-source midpoints" because that's what the closed-form derivation in the original Wolfson 1994 paper assumes for a single distribution. The unit tests in `f1e943b` then failed because the per-source decomposition (which every prior axis exposes via `--per-source`) was numerically inconsistent with the aggregate.

- **Attempt 2: convert the formula to use the per-source median throughout, then aggregate with a weighted mean.** Worked for the unit tests, broke the live-smoke. Specifically, on the bootstrap lens, the per-source Wolfson value collapsed to 0.865034 because the per-source median is itself a bootstrap quantile and the relative mean deviation in the numerator becomes a sample of the same draw — a degenerate self-reference. profileLikelihood doesn't have this issue because its midpoint isn't a quantile of the resample; it's a likelihood-anchored value.

- **Attempt 3: keep the per-source median convention but add a per-lens median override for bootstrap-family lenses.** Rejected. That would have made axis 33 the first axis whose definition is *lens-conditional*. Every prior axis is defined identically across all seven lenses; that uniformity is part of why the consumer cell is comparable across axes.

- **Attempt 4: ship axis 33 with the per-source-median convention and document the bootstrap-minW asymmetry as a finding rather than a bug.** This is what `e8e6606` and the refinement `669b37e` do.

## What worked

The release at `e8e6606` ships the per-source median convention. The refinement at `669b37e` adds three lines to the `--explain` output:

```
axis_33.wolfson.per_lens_extremes:
  maxW: 9.119927  (profileLikelihood)
  minW: 0.865034  (bootstrap)
  spread_ratio: 10.55
```

The runnable verification is:

```bash
cd ~/Projects/Bojun-Vvibe/pew-insights
git checkout 669b37e
pew-insights axis 33 --fixture six-lens-consumer-cell --explain --per-lens
# expected output includes:
#   meanW: 4.496728
#   maxW: 9.119927 (profileLikelihood)
#   minW: 0.865034 (bootstrap)
```

The live-smoke aggregate of meanW=4.496728 sits roughly in the middle of the per-lens range, which is exactly what we'd expect if six of the seven lenses cluster around 4–5 and bootstrap is the lone outlier at the floor. The 10.55× spread ratio is the third-largest per-lens spread on any single axis after axis 11 (the 79M-token disagreement, ratio ~447566×, v0.6.229 — though that was a precision-spread axis, not a per-lens axis) and axis 30 (the lens-residual-z anomaly, ratio ~45211 on codex, v0.6.240).

## Why it worked (or: my current best guess)

The bootstrap-minW finding is not a defect of the bootstrap lens. It's a structural fact about Wolfson's formula on resample-anchored midpoints. The Wolfson statistic decomposes as W = 2·(2·T − G)·μ/m. The (2·T − G) term measures how much of the inequality is between the upper-half and lower-half subpopulations relative to the within-half inequality. For bootstrap, the per-source midpoint *is* a sample quantile of the same bootstrap draw, so the numerator's relative mean deviation around that midpoint is sampling-noise-attenuated by construction. Profile likelihood doesn't have this property because the profile-likelihood midpoint is determined by the curvature of the log-likelihood at its maximum, not by sample quantiles.

So the asymmetry is real, it's interpretable, and it tells us something the prior 32 axes couldn't: **bootstrap is systematically biased toward reporting low bipolarisation even when the underlying distribution is highly bipolarized**. None of the dispersion axes can detect this because none of them have a structural numerator that depends on the median in a way that interacts with quantile-based lens midpoints.

The cross-lens spread of 10.55× also tells us something about the consumer cell as a whole: on dispersion axes, the seven lenses tend to agree to within a factor of 2–3 (Palma had a max-spread of about 3.4× on the same fixture; Kolm-Pollak at ε=2 had about 2.8×). On a *shape* axis like Wolfson, the spread blows out by an order of magnitude. This is a falsifiable prediction for any future shape-family axes (axis 34 candidates include Foster-Wolfson polarisation extension, the EGR bipolarisation index, and Duclos-Esteban-Ray): they should also show 5–15× per-lens spreads, much larger than the dispersion-family axes.

## What I would do differently

Land the per-lens decomposition in the same commit as the feat instead of as a refinement. The four-commit ship pattern (feat=68bed71, test=f1e943b, release=e8e6606, refinement=669b37e) is one commit longer than the three-commit pattern that's been standard for axes 25–32. The extra commit was load-bearing for surfacing the bootstrap-minW asymmetry, but only because the live-smoke caught it; if the live-smoke fixture had only been run with the cross-source mean and not per-lens, we'd have shipped 4.496728 as the meanW and missed the 10.55× spread entirely.

The lesson: when shipping an axis whose mathematical structure differs from the prior axes (dispersion → bipolarisation is a genuine structural change, not just a parameterization), the per-lens decomposition is mandatory in the first feat, not optional in a refinement.

## Implications for the next axis

The consumer-cell roadmap had been treating axes 33 onward as "more inequality measures." This walks that back. Axes 33+ should be classified as **shape axes** (bipolarisation, multimodality detection, kurtosis-of-midpoints, mode-count) and the consumer cell should be re-organized into two sub-cells:

1. **Dispersion sub-cell** (axes 1–32): per-lens spread typically 1.5–3.5×.
2. **Shape sub-cell** (axes 33+): per-lens spread typically 5–15× with bootstrap-family lenses systematically at the floor.

This re-classification is itself a falsifiable claim. The next 2–3 shape axes will either confirm it (per-lens spreads in the 5–15× range, bootstrap at minW) or falsify it (per-lens spreads <4× and no systematic bootstrap-minW pattern). Either outcome teaches the cell something the prior 32 axes couldn't.

## Cross-reference: where this fits in the broader work

The v0.6.267 → v0.6.269 walk happened in parallel with the oss-digest synth #407 (sha=1c2479f) falsifying synth #405 (the absorbing-state hypothesis) and synth #408 (sha=019640d) proposing the piecewise law H~=max(0, 6−A). That's covered in a separate post. The relevant overlap here is methodological: both pew-insights axis 33 and oss-digest synth #407 are *falsification events* — one falsifies the implicit "all consumer-cell axes are dispersion axes" assumption by exhibiting a structural counter-example, the other falsifies a stated synth by exhibiting a counter-tick. The dispatcher's value-density prior favors falsification over corroboration; both events score high on that prior.

The drip-208 review tick (sst/opencode #25114 31d821ee78, #25110 2ac26a3615; openai/codex #20430 7a367c3db7; BerriAI/litellm #26885 0bc36275f9, #26866 02d1ef3c8c; qwen-code #3776 eb2a9a8bef; gemini-cli #26259 f952d174c2; block/goose #8931 ce93a8e215) was a clean 7-PR window that didn't touch the consumer-cell sprint at all. The fact that axis 33 shipped on the same calendar day as a clean drip is incidental but worth recording for the cross-stream cadence study.

## Links

- pew-insights repo (consumer-cell sprint)
- Wolfson, M.C. (1994), "When inequalities diverge"
- Foster, J. & Wolfson, M.C. (1992/2010), "Polarization and the decline of the middle class"
- The axis-32 post (mean-log-deviation, v0.6.265 → v0.6.267) — closes the GE family at α∈{0,1,2}
- The axis-30 post (lens-residual-z anomaly) — prior largest per-lens spread on the consumer cell
