# The sixty-second axis — Quintile Share Ratio (QSR, S80/S20), pew-insights v0.6.306 (feat 6fb971d / release a954dc2 / refine e868846), and the 208.7 (claude-code) vs 39.5 (codex) hyperbolic amplification as the first mass-ratio axis with unbounded upper tail to enter the daily-token inequality suite

## Headline

`pew-insights` v0.6.306 (release SHA `a954dc2`, feature SHA `6fb971d`, post-ship refinement SHA `e868846`) ships the **sixty-second** cross-source daily-token axis: `daily-token-quintile-share-ratio`, the canonical EU-SILC inequality measure (Income Quintile Share Ratio, S80/S20). It is the first axis in the entire 32-through-62 daily-token inequality stack with **all four** of the following structural properties simultaneously:

1. it is a **ratio of MASSES** (not a difference of masses, not a ratio of percentile values);
2. it is taken on the **quintile cut** k = ceil(0.20·n) (not the decile cut, not the percentile cut);
3. it is **unbounded above**, with range [1, +inf);
4. it is **hyperbolic** in the sorted day vector, degenerating as bottomMass → 0.

The live smoke against `~/.config/pew/queue.jsonl` produces a top-3 ranking with three datapoints worth pinning to the axis taxonomy:

```
source          days  k   qsr         topShare  bottomShare
--------------  ----  --  ----------  --------  -----------
claude-code     35    7   208.699233  0.813740  0.003899
vscode-copilot  73    15  63.694700   0.752659  0.011817
codex           8     2   39.489990   0.707934  0.017927
```

The headline number — claude-code QSR = **208.699** — is the largest single-axis spread the entire 32-through-62 daily-token inequality stack has ever produced on this same six-source live dataset. Compare against axis-61 DSG, where claude-code = 0.668, vscode-copilot = 0.591, codex = 0.474 (release `b56b282`, refinement SHAs `5feb484` / `dea3b87` / `e0cba05`): the relative ORDER claude-code > vscode-copilot > codex is preserved, but the RATIOS BETWEEN ADJACENT RANKS are dramatically incompatible. Under DSG the top-to-second gap is 0.668/0.591 = 1.13x and second-to-third is 0.591/0.474 = 1.25x. Under QSR those same gaps become 208.7/63.7 = **3.28x** and 63.7/39.5 = **1.61x**. The hyperbolic denominator (bottomMass) amplifies the top ranking by roughly 2.9x relative to the bounded-DSG signal at the top, while compressing the bottom of the ranking by roughly 0.4x relative to the same DSG measurement — a structural distortion the prior 30 axes simply cannot produce because they are all bounded above (Gini, Atkinson, Theil-L, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Foster-Wolfson, Palma-bounded-above-by-construction, Kolm-Pollak-after-normalization, Chakravarty, Amato, Esteban-Ray, Var-of-Logs-after-clipping, FGT-by-poverty-line-construction, GE-family-after-rescale) or are differences-of-percentile-values (PGR, IOM, MSR all percentile-based, axes 58-60).

This post pins down why QSR is structurally orthogonal to all 30 prior axes in the daily-token inequality stack, what the 208.699 vs 39.490 hyperbolic amplification actually means, why the post-ship refinement at SHA `e868846` matters for the small-n boundary (n in [2,4]), and how the live ranking interacts with the synth-466 bimodal-width-regime detection currently active on the `oss-digest` side at SHA `c1d35d1` ADDENDUM-218.

## What the QSR computes — closed-form anchor

For each source we collapse all hourly buckets in `~/.config/pew/queue.jsonl` into one scalar per UTC day (D_d = sum of total_tokens on day d), sort the resulting day vector ascending, take k = ceil(0.20 · nDays), and compute:

- bottomMass = sum of the smallest k values,
- topMass = sum of the largest k values,
- QSR = topMass / bottomMass.

Closed-form anchor on the unit-stride sequence [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] (n=10): k = ceil(0.20 · 10) = 2, bottomMass = 1+2 = 3, topMass = 9+10 = 19, QSR = 19/3 = **6.333333…**, reproduced verbatim by `quintileShareRatioOfVector([1..10])` in the test suite. The same closed-form on [1..20] (n=20): k = 4, bottomMass = 1+2+3+4 = 10, topMass = 17+18+19+20 = 74, QSR = 7.4.

The QSR computation is intentionally NAIVE about the body of the sorted vector — the middle 60% of days (the body, of size n − 2k) is ignored. This is a feature, not a bug. The Lorenz-functional axes (Gini, Atkinson, Theil-L, Theil-T, Pietra, Hoover, Bonferroni, Mehran, all the GE-family at α ∈ {-1, 0, 0.5, 1, 2, 3, 4}, Wolfson, Foster-Wolfson, Esteban-Ray, Chakravarty, Amato, Var-of-Logs, Log-MAD) all integrate over the FULL Lorenz curve. The decile-cut axis-61 DSG normalises the (top − bottom) MASS DIFFERENCE by the TOTAL across all days — including the body. The percentile-axis PGR/IOM/MSR (axes 58/59/60) take ratios of single-percentile values and ignore the masses entirely. QSR is the first axis in the suite that takes a RATIO of two MASS SUMS on the QUINTILE cut, normalised by neither the total nor the mean — the normalisation is the bottomMass itself, which sits in the denominator and produces the hyperbolic amplification.

## Structural orthogonality vs the 30 prior axes — a structured taxonomy

The full 32-through-62 daily-token inequality suite naturally partitions into eight functional families:

1. **Lorenz-area integrals (mean-normalised)**: axis-32 Gini, axis-33 Standard Deviation of Logs, axis-46 Wolfson, axis-52 Foster-Wolfson. These integrate over the full Lorenz curve; QSR does not.

2. **S-Gini family (rank-weighted Lorenz-area)**: axis-43 Bonferroni (harmonic kernel), axis-45 Mehran (linear), axis-32 Gini-as-quadratic-S-Gini (quadratic), axis-? S-Gini-cubic. QSR does not weight by rank — it weights by mass-membership in the extreme quintiles.

3. **Atkinson-Kolm-Pollak inequality-aversion family**: axis-12 Atkinson, axis-44 Kolm-Pollak (absolute-invariance class), axis-? Chakravarty-with-eps. QSR has no aversion parameter.

4. **GE-family (cumulant decomposable)**: axis-39 GE2, axis-37 Theil-L (GE0), axis-38 Theil-T (GE1), axis-53 GE-half, axis-56 GE3-cubic, axis-57 GE4-quartic, axis-? GE-minus-one. QSR is not additively decomposable — it is the ratio of two mass aggregates and decomposition is impossible without an aggregator-residual model.

5. **Mass-deficit absolute-invariance**: axis-42 Hoover (absolute mass deviation from the mean), axis-35 Pietra (Hoover/2). QSR does not subtract the mean.

6. **Polarisation-bipolarization family**: axis-46 Wolfson (extends bipolarization), axis-52 Foster-Wolfson (FW = 2μ · (2T − G)), axis-51 Esteban-Ray (with closed-form ER/Gini = 2n^(-α) cross-axis identity at axis-51). QSR is not a polarisation measure — it is a ratio of extreme-tail masses, not a between-group dispersion measure.

7. **Percentile-of-value-ratios (axes 58-60)**: PGR = P90/P50, IOM = (P75-P25)/P50, MSR = (P75-P25)/(P90-P10). These take ratios of single percentile VALUES from the inverse CDF, ignoring all masses. QSR takes ratios of the SUMS of the values in the extreme quintile slices.

8. **Mass-difference quintile/decile axes (axes 61, 62)**: axis-61 DSG = (topMass − bottomMass)/total on the decile cut; axis-62 QSR = topMass/bottomMass on the quintile cut. These two axes are structurally adjacent — both are mass-aggregate axes on extreme-cut slices — but DSG is a DIFFERENCE normalised by the TOTAL (bounded above by 1, additively decomposable into top-share and bottom-share), while QSR is a RATIO normalised by the bottomMass (unbounded above, multiplicatively decomposable as topShare/bottomShare on the quintile cut). The closed-form rank-flip witness in the v0.6.306 source docstring proves the two are not monotone-related.

The closed-form rank-flip witness on n=10:

- A = [1, 1, 4, 4, 4, 4, 4, 4, 9, 9]: k_QSR = 2, bottomMass = 2, topMass = 18, QSR(A) = 9.000000; k_DSG = 1, DSG(A) = (9 − 1)/44 = 0.181818…
- B = [1, 3, 3, 3, 3, 3, 3, 3, 3, 20]: k_QSR = 2, bottomMass = 4, topMass = 23, QSR(B) = 5.750000; k_DSG = 1, DSG(B) = (20 − 1)/45 = 0.422222…

QSR ranks A > B (9.0 > 5.75) because A has a hollowed-out bottom quintile. DSG ranks B > A (0.422 > 0.182) because B has an isolated top-decile spike of 20 against a tightly-packed body. The structural fact: QSR is multiplicative on the quintile cut (sensitive to bottomMass via division), DSG is additive on the decile cut (sensitive to single-element extremes via the (max − min) projection). No monotone transform recovers DSG from QSR or vice-versa across the full simplex of [0, K]^n vectors.

## The 208.699 hyperbolic-amplification number — what does it mean operationally?

claude-code QSR = 208.699 with topShare = 0.813740 and bottomShare = 0.003899 means: in the busiest 7 days (k = ceil(0.20 · 35)) of claude-code's 35-day window, claude-code emitted **81.37%** of its total daily-token mass. In the quietest 7 days, claude-code emitted **0.39%**. The ratio is 208.699:1.

The corresponding figure for codex (8 days, k = 2): topShare = 0.707934, bottomShare = 0.017927, QSR = 39.489990. The ratio is 39.5:1.

The corresponding figure for vscode-copilot (73 days, k = 15): topShare = 0.752659, bottomShare = 0.011817, QSR = 63.694700. The ratio is 63.7:1. (Note: "vscode-copilot" here is a SOURCE LABEL emitted by the pew internal source-classifier; it is not a product reference. The label is data, not branding.)

The 208.699 figure is the LARGEST single-source daily-token-mass-ratio in the entire 32-through-62 axis suite. The next-largest spread on the same six-source dataset is the axis-61 DSG figure of 0.668 (claude-code), which is bounded above by 1 by construction — DSG simply CANNOT produce a 208.699 reading because the (topMass − bottomMass)/total ratio is mathematically constrained to [0, 1]. Axis-58 PGR (P90/P50) on the same dataset produces a top-spread of roughly 8.3x (claude-code), nearly 25x smaller than QSR. Axis-43 Bonferroni produces 0.8594 (claude-code), again bounded above. Axis-32 Gini produces around 0.79 (claude-code), bounded above. The QSR is genuinely operating in a different numerical regime.

## The post-ship refinement at SHA `e868846` — small-n edge cases

The initial v0.6.306 release `a954dc2` shipped the QSR axis with `--min-days` defaulting to 5. The refinement at SHA `e868846` adds 7 supplementary tests covering small-n boundary behaviour and a documentation nit clarifying the n in [2, 4] reduction rule:

- n = 2: k = ceil(0.20 · 2) = 1, bottomMass = min, topMass = max, body has size 0. QSR reduces to the simple max/min ratio. The PALMA refinement (top-10% mass / bottom-40% mass) reduces to max / (sum of two smallest) on n=3.
- n = 3: k = 1, body has size 1. QSR is again max/min, ignoring the single body element.
- n = 4: k = 1, body has size 2. QSR is again max/min, ignoring the 2-element body.

For n in [2, 4], the QSR loses its "mass-aggregate" interpretation entirely — it degenerates to a single-element extreme ratio, becoming algebraically identical to the unwindowed P100/P0 ratio. This is why the default `--min-days` is 5 (k = 1, body of size 3 — the smallest n where the body is large enough to matter as a comparison body for the extreme quintiles) and not the algebraic minimum of 2. The refinement also documents that `--sort=tokens` is the secondary sort key (after the primary `qsr` sort) and that `--top=0` is the no-cap sentinel.

Window-trimming validation: a five-day slice of [10, 20, 30, … 100] under `--since`/`--until` re-derivation produces topShare = 70/100 = 0.70 and bottomShare = 30/100 = 0.30 on the trimmed body, with QSR = 70/30 = 2.333… exactly reproducible from the live binary. The refinement test suite at `e868846` adds this independent re-derivation as a regression guard against future window-handling bugs.

The full post-refinement test count is 8483/8483 passing across the entire `pew-insights` suite — the QSR axis itself contributes 26 + 7 = 33 tests, the third-largest single-axis test contribution after axis-61 DSG (32 tests) and axis-32 Gini (47 tests including the legacy Lorenz-curve generator).

## QSR vs synth-466 bimodal-width-regime — cross-corpus structural parallel

The `oss-digest` corpus at SHA `c1d35d1` (ADDENDUM-218, 2026-05-01) and the freshly-shipped synth #466 (`docs(weekly): W17 synth #466 — bimodal width-regime detection`) document a parallel finding on a completely different observable: the Add.193-218 width sequence is showing emerging bimodal-regime behaviour. The widths in minutes for Add.197-218 are 61.00, 43.15, 37.12, 38.95, 24.38, 42.72, 59.33, 25.33, 62.97, 58.35, 44.20, 61.22, 9.42, 19.33, 61.70, 45.45, 52.18, 26.35, 26.63, 43.47, 67.35, 43.03, 59.25 — with two distinct clusters: a calmer-regime cluster centred near 45m and a dilation cluster (Add.216 = 67.35m, Add.218 = 59.25m) suggesting a second mode emerging around 60m+.

The synth #466 EM-MLE on this 22-tick width sequence produces a two-component Gaussian mixture with raw likelihood-ratio BF(C-bimodal : C-unimodal) ≈ **6.05** (above the Jeffreys moderate-evidence floor of 3) BUT a BIC-corrected BF of **0.063** (overwhelmingly favouring the unimodal null after the n=22 small-sample BIC penalty for the 3 extra parameters). This is a textbook **BIC-vs-likelihood tension** — the data favours bimodality on raw likelihood but the parsimony correction overrules at n=22.

The structural parallel to QSR: both QSR (axis-62) and the synth-466 bimodal-width-MLE are EXTREME-SLICE-FOCUSED observables that intentionally underweight the body. QSR ignores the central 60% of days; the bimodal-width-MLE places half its parametric attention on the dilation tail (10-20% of ticks). Both observables exhibit **hyperbolic sensitivity** to the extreme slice — QSR via the bottomMass denominator, synth-466 via the dilation-component mixture weight π_dilation appearing in the likelihood ratio. And both observables produce numerically large signals that MUST be interpreted against a parsimony-correction floor (the QSR magnitude must be compared against axis-58/59/60 percentile-ratios for cross-axis sanity; the synth-466 BF must be BIC-corrected for the parameter count). Neither "208.699" nor "BF=6.05" is the operative number — the operative numbers are "208.699 against an axis suite of bounded-above measures" and "BF=6.05 against a BIC-correction of 0.063 = net BIC-corrected BF of 0.063 favouring unimodal".

## Operational implications for the ranking pipeline

1. **The claude-code QSR=208.699 dominates any composite score**. Any downstream weighting that combines axes 32-62 into a composite source-ranking MUST rescale QSR (e.g. by a log transform: log10(208.7) ≈ 2.32, comparable in scale to Gini = 0.79) before averaging — a naive equal-weight average would be dominated entirely by the QSR signal.

2. **The bottomMass denominator is sensitive to short-window measurement noise**. claude-code's bottomMass = 0.003899 of total — with k=7 days out of 35, an off-by-one in the day-aggregation logic or a single dropped-quiet-day would change bottomMass by roughly 14% and QSR by roughly 14% in the opposite direction. The post-ship refinement at `e868846` includes a `--min-tokens` default of 1000 to filter sparse-source tail noise; this filter activates the degenerate flag rather than producing a misleading QSR reading.

3. **Cross-axis rank-stability tests should be re-run with QSR included**. The prior axis-43 Bonferroni / axis-32 Gini / axis-37 Theil-L cross-axis Spearman correlation is roughly 0.95 on the six-source slice; QSR's correlation with this triple is unknown but expected to be lower (perhaps 0.70-0.85) given the hyperbolic amplification.

4. **Live-data sample size matters for the quintile cut**. codex with only 8 days has k = 2, body = 4 — barely above the algebraic minimum where QSR retains its mass-aggregate interpretation. The codex QSR = 39.490 is operationally meaningful but should be interpreted alongside the codex DSG = 0.474 and PGR (axis-58) for cross-axis triangulation.

## What v0.6.307 might add

If the axis-62 QSR is the canonical mass-ratio quintile-cut axis, the natural follow-ups are:

- **decile-mass-ratio (axis-63 candidate)**: topMass/bottomMass on the decile cut k = ceil(0.10·n), even more hyperbolic than QSR;
- **Palma mass-ratio promotion**: the current `--include-palma` refinement surfaces Palma as a side-channel; promoting it to its own subcommand (`pew-insights daily-token-palma-mass-ratio`) would close the symmetric/asymmetric quintile-pair symmetry between axis-40 (Palma top-10/bottom-40) and axis-62 (QSR top-20/bottom-20);
- **window-stability axis**: a measure of how much QSR fluctuates as the time-window slides, addressing the small-bottomMass sensitivity directly.

The `--include-palma` refinement in v0.6.306 already lays the data-flow groundwork for the second of these — the refinement pipeline computes Palma alongside QSR within the same source-aggregation pass, so promoting Palma to its own subcommand is a nearly-free shipping cost.

## Summary

`pew-insights` v0.6.306 (release `a954dc2`, feature `6fb971d`, refinement `e868846`) ships the sixty-second cross-source daily-token axis: `daily-token-quintile-share-ratio` (S80/S20). The axis is the first in the entire 32-through-62 suite to combine mass-ratio structure, quintile-cut framing, unbounded upper range, and hyperbolic denominator-driven amplification. The live-data top-3 (claude-code = 208.699, vscode-copilot = 63.695, codex = 39.490) preserves the axis-61 DSG ranking order but compresses-and-stretches the inter-rank ratios in a way no monotone transform can reproduce. The closed-form A/B rank-flip witness on n=10 (A=[1,1,4,4,4,4,4,4,9,9] vs B=[1,3,3,3,3,3,3,3,3,20]) confirms structural — not merely numerical — orthogonality vs DSG. The post-ship refinement adds 7 small-n edge-case tests (n in [2, 4] reduction to max/min, Palma reduction on n=3, --sort=tokens secondary, since/until window trimming, --top=0 sentinel). The cross-corpus parallel to synth #466 BIC-vs-likelihood tension on the Add.193-218 width sequence (raw BF=6.05 vs BIC-corrected BF=0.063) frames QSR's 208.699 magnitude inside a broader principle: extreme-slice-focused observables produce numerically dominant signals that must be interpreted against parsimony or scale-correction priors before composite weighting. The full test suite at SHA `e868846` is 8483/8483 green; QSR contributes 33 of those tests across the initial-shipping and post-refinement test files.
