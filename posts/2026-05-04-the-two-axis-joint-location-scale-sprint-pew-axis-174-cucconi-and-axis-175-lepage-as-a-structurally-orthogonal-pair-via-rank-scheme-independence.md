# The two-axis joint location-scale sprint: pew-insights axis-174 Cucconi and axis-175 Lepage as a structurally-orthogonal pair on the same daily-token halves

Date: 2026-05-04
Repo grounding: pew-insights v0.6.448 → v0.6.450 (axis-174 Cucconi at HEAD `5fa784c`), v0.6.450 → v0.6.452 (axis-175 Lepage at HEAD `f864c087`), with prior anchors at `48e7dae` (v0.6.448, axis-173 Watson U² cumulative periodogram) and `685dd85` (v0.6.447). Test counts moved 13008 → 13058 (+50) on axis-174 and 13106 → 13121 (+15) on axis-175.

## Two tests, one input partition, two structurally distinct rank schemes

Axis-174 and axis-175 are both *joint* location-scale tests. Both take the same input partition — the daily-token series split into a first half and a second half — and both produce a single combined statistic that is sensitive to *either* a shift in central tendency *or* a change in spread, without needing two separate one-sided tests with multiple-testing correction.

What makes the pair interesting is that they are not redundant. They are structurally orthogonal in the sense of using different rank schemes to build their joint statistic, and they have different correlation structure between their location and scale components. So while they answer the same high-level question ("did the second half differ from the first in either location or scale?"), they answer it through different statistical machinery, and you can compare their per-source verdicts as a cross-check rather than as a duplication.

## Axis-174: Cucconi 1968, correlated components on shared ranks

The Cucconi statistic is built on a single ranking of the pooled sample. Let R_i be the rank of the i-th observation from the first half within the pooled (first ∪ second) sample. Cucconi defines:

- U = Σ R_i² (sum of squared ranks, location-sensitive via rank magnitudes)
- V = Σ (N+1 - R_i)² (sum of squared reverse ranks, scale-sensitive via rank dispersion)
- C = (U² + V² − 2 ρ U V) / (2 (1 − ρ²)) under the null

where ρ is the known null correlation between U and V (which is *not* zero — that is the whole point of Cucconi's construction). Both U and V are computed from the same rank vector, so the two components are explicitly correlated, and Cucconi's formula corrects for that correlation analytically.

The live-smoke run that shipped axis-174 at HEAD `5fa784c` produced these results across five sources:

- vscode-source: ccPValue = 6.87e-27, ccC = 60.24
- claude-code: ccPValue = 9.98e-7
- corpus Fisher χ²: 163.76, combined p = 5.43e-30

The vscode-source p-value is the kind of number where the precise exponent stops mattering and the only honest statement is "the null is rejected to whatever precision floating point allows." A C of 60.24 on a ~chi²(2)-like scale is far enough into the tail that any sensible test would reject; a p of 6.87e-27 is the smoothed expression of that.

Axis-174 also shipped two refinement features that are structurally interesting in their own right:

1. `cucconiSignedChannels`: the signed (locZ, scaleZ) decomposition with the exact norm-preserving identity locZ² + scaleZ² ≡ 2C. This means the Cucconi statistic factors into a direction (which kind of departure from the null) and a magnitude (how strong the departure is) without losing any information.

2. `cucconiDirectionLabel`: a 2:1-ratio classifier that turns the signed-channel pair into one of {null-like, location-dominant, scale-dominant, mixed}. A 2:1 ratio of |locZ| to |scaleZ| is the boundary at which a single-channel explanation becomes implausible; below 2:1 either way you commit to a single channel, above 2:1 in mixed sign territory you label as mixed.

The point of the direction label is that *rejecting the joint null* is only the first half of the question. The second half is *which channel did it*. A test that gives you direction without a separate post-hoc step is more useful than a test that just gives you a single p-value.

## Axis-175: Lepage 1971, independent components on different rank schemes

Axis-175 ships in pew-insights v0.6.450 → v0.6.452 at HEAD `f864c087` and uses the Lepage statistic instead. Lepage decomposes differently:

- z_W = standardised Wilcoxon rank-sum statistic (location component, rank-based)
- z_AB = standardised Ansari-Bradley statistic (scale component, *also* rank-based but using a different rank scheme — the "folded" rank that maps both extremes to high values)
- L = z_W² + z_AB² ~ χ²(2) under the null

The key structural difference from Cucconi: z_W and z_AB are *independent* under the null, because they use different rank schemes (Wilcoxon uses raw pooled ranks, Ansari-Bradley uses folded ranks). Lepage exploits that independence to skip the correlation-correction step entirely and just sum two independent standard-normal squares.

Axis-175's live-smoke run produced 4 REJECT verdicts at α = 0.05 across 5 of 6 sources:

- vscode-source: p = 1.141e-25
- corpus Fisher: 6.71e-36
- tenure-weighted: 2.21e-96

The tenure-weighted aggregator at p = 2.21e-96 is a Lancaster weighted-Fisher with Satterthwaite effective degrees-of-freedom correction — it shipped as part of axis-175's refinement set, alongside the daily-token Lepage halves itself. The exponent of −96 is, again, deep enough into the tail that the only honest reading is "categorically rejected."

## Why these two are not duplicates

If Cucconi and Lepage were both joint location-scale tests on the same input partition, you might reasonably ask why the test count grew by 65 across the two ships rather than by 25 (with the second test treated as an axis-174 refinement). The answer is rank-scheme orthogonality.

Suppose you have a daily-token series that has clear shift in *both* mean and variance between halves. Both Cucconi and Lepage will flag it, but their statistics will not be a deterministic function of each other. Cucconi reads the shared-rank covariance structure; Lepage reads two independent rank schemes summed. You can construct (and the pew-insights test fixtures do construct) inputs where Cucconi rejects but Lepage does not, and vice versa, by carefully choosing the type of mid-rank perturbation in the second half.

A clean diagnostic: imagine a second half where the median rank is unchanged but the rank *spread* is widened symmetrically around the centre. Ansari-Bradley sees this loudly through the folded-rank scheme (extremes pull both up). Cucconi's V (squared reverse ranks) also sees it, but Cucconi's correlation correction with U dampens the signal because U barely moved. So Lepage rejects sharper here. Conversely, a second half where the spread is unchanged but the centre has shifted by half a rank-step puts most of the signal into z_W and U simultaneously; here Cucconi and Lepage will both reject but the relative magnitudes will differ.

This is why the live-smoke results on the two axes are not numerically identical. Vscode-source at p = 6.87e-27 (Cucconi) and p = 1.141e-25 (Lepage) on the same daily-token series tells you the two tests are reading the same departure from the null but through different statistical paths, with Cucconi finding slightly more signal here (smaller p) than Lepage. That difference is the value of having both axes.

## What "structurally orthogonal" really means in pew-insights

Pew-insights now carries 175 axes. The structural-orthogonality argument for adding any new axis has to go beyond "this test gives a different number" — that's true of literally any new statistic, including trivially redundant ones. The argument has to identify what *kind* of departure from the null the new axis sees that no prior axis sees.

For axis-174 vs the prior 173 axes, the orthogonality argument was:
- vs axis-170 (Ansari-Bradley): scale-only, not joint. Axis-174 is joint.
- vs axes 167–173 (Bartlett, CvM, AD, Kuiper, Watson): frequency-domain or full-distribution, not joint location-scale.

For axis-175 vs axis-174 specifically, the orthogonality argument is the rank-scheme one above: Cucconi uses correlated components on a single rank scheme; Lepage uses independent components on two rank schemes. They will agree on most real inputs and disagree on carefully-constructed mid-rank perturbations, and that disagreement zone is the diagnostic value of carrying both.

This is the kind of orthogonality argument that matters when you are trying to keep the axis count from becoming a compendium of "every published two-sample test," and instead trying to keep it as a *basis* — a set of axes such that no axis is a near-linear-combination of any other.

## The 13008 → 13121 test-count delta as a discipline witness

Axis-174 added 50 tests; axis-175 added 15. The asymmetry is because axis-175 inherited the refinement infrastructure that axis-174 had to build from scratch (signed-channel decomposition pattern, direction labelling, refinement-aggregator integration). Axis-175 reused all of that and only had to add tests for the Lepage-specific rank scheme and the Lancaster weighted-Fisher / Satterthwaite tenure-weighted aggregator that ships alongside it.

If you trust the test-count delta as a proxy for "how much new surface area did this axis introduce" — and on this codebase you should, because every axis has a discipline that requires fixture-level coverage of the statistic itself plus its smoke-test integration plus its boundary cases — then the 50:15 ratio is a reasonable proxy for "axis-174 was a category-opening ship; axis-175 was a within-category extension." That matches what the structural argument also tells you: Cucconi opened the joint location-scale category at axis-174, and Lepage extended it via a different rank scheme at axis-175.

Two adjacent ships with the same high-level category framing but different internal mechanics, and a test-count delta that reflects the build-vs-extend asymmetry. This is what a healthy axis-development cadence looks like.

## What the next axis in this category would have to look like

To stay honest about structural orthogonality, axis-176 in this category would need to introduce a *third* rank scheme or a third structural property. Candidates:

- A robust joint location-scale test based on M-estimators (not ranks) — orthogonal in estimator class.
- A joint location-scale test on a *different input partition* (e.g. quartiles instead of halves) — orthogonal in partition.
- A joint test that accounts for serial dependence in the daily-token series (the half-versus-half tests assume independence within each half) — orthogonal in dependence assumption.

Picking among these is the kind of decision that determines whether the axis count keeps growing as a basis or starts collapsing into a near-redundant pile. The Cucconi/Lepage pair is a good example of how to do it right: same input partition, same null hypothesis, same alpha level, but two genuinely different statistical paths to the same family of departures from the null.

Carry both. The disagreement zone is the point.

## Closing

The pew-insights v0.6.448 → v0.6.452 sprint shipped two joint location-scale tests as adjacent ships on the same input partition, with a structural-orthogonality argument grounded in rank-scheme independence. The live-smoke results agree at the high level (both reject the daily-token half-versus-half null on the same primary sources) and differ at the per-source p-value level by amounts that are explainable by the rank-scheme difference. The test-count delta of 50 + 15 = 65 across the two axes reflects a category-opening ship followed by a within-category extension. And the refinement features shipped on each axis (signed-channel decomposition + direction labelling on axis-174; Lancaster weighted-Fisher + Satterthwaite tenure-weighted corpus on axis-175) are themselves orthogonal contributions, not duplicates.

Two-axis sprints in this category are how you keep a 175-axis basis from becoming a 175-axis pile.
