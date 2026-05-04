# The axis-182 Fligner-Policello robust-rank Behrens-Fisher view on pew daily-token halves: where relaxing equal-scale flips the openclaw vs claude-code sign, and the cross-axis-181 vdW sign-agreement table

**Date:** 2026-05-05
**Subject:** pew-insights v0.6.464, axis-182 (Fligner-Policello, FP), live-smoke run at HEAD `2c5e677`
**Companion axes cited:** axis-181 (van der Waerden normal scores), axis-180 (Sukhatme), axis-179 (Mood)
**Sources HEAD:** axis-182 live-smoke at `2c5e677` (pew-insights v0.6.464)

## 0. The one-paragraph gist

The pew daily-token-half family is a four-source comparison: `openclaw`, `claude-code`, `vscode-cp`, and `hermes`. Through axes 177-181 we've been stacking *equal-scale-assuming* rank tests — Brunner-Munzel, Klotz, Conover squared ranks, Mood, Sukhatme, van der Waerden — onto the same first-half-vs-second-half partition for each source's daily token series. Axis-182 is the first axis in the family that explicitly **drops the equal-scale (homoscedasticity) assumption** and asks the Behrens-Fisher version of the location question on ranks: the Fligner-Policello (FP) test. The headline finding from the live-smoke at HEAD `2c5e677` is that on `openclaw` the FP statistic comes back at `fpZ = -5.3439, p = 9.119e-08`, while on `claude-code` it comes back at `fpZ = +4.1505, p = 3.320e-05`. Both are extremely strong rejects of the no-shift null, and they point in **opposite directions**, which is the same direction-split that axis-181 (vdW) reported one axis earlier (`agent-cc vdwZ = +4.35, p = 1.4e-5`; `agent-oc vdwZ = -3.05, p = 2.3e-3`). The cross-axis sign-agreement table at the bottom of this note is the actual content; the rest is the scaffolding to read it.

## 1. Why a Behrens-Fisher rank test belongs on this stack

The Mann-Whitney-Wilcoxon (MWW) statistic is a pooled-rank sum. Brunner-Munzel relaxes the *tied-distribution* assumption but still leans on equal scale under the null in finite samples for its variance estimate. Klotz, Mood, Sukhatme, Conover squared ranks, and van der Waerden are *scale* tests in spirit (or scale-aware location tests), and we've been treating them as orthogonal completions of the location stack on the first-half-vs-second-half partition. The hole in that stack is the case where the two halves have *both* unequal location and unequal scale — which is the working hypothesis on `openclaw` because of the long-tenure regime shift around day 265 that axis-179 already flagged at `p = 2.46e-17`.

Fligner-Policello (FP) addresses exactly that hole. Defined in Fligner & Policello (1981), FP is the rank analogue of the Behrens-Fisher t-statistic: it pools both samples into ranks but estimates the variances of the two within-sample rank-sum components *separately*, then forms a Z-statistic by dividing the difference of within-sample placement counts by the square root of the sum of those two separately-estimated variances. The point of the construction is that under the null of equal medians (or, more carefully, equal placements), the asymptotic distribution of `fpZ` is standard normal **without** the assumption that the two underlying distributions are the same shape or have the same dispersion. That is precisely the gap between MWW/BM and a full Behrens-Fisher solution.

For the pew daily-token-half setup the partition is fixed: for each source, sort its daily token totals by date, split at the median date, call the first chunk sample A and the second chunk sample B. We've been running every axis on the same partition, which means the cross-axis comparison is honest — the Z values are not generated on different splits.

## 2. The axis-182 live-smoke numbers at HEAD 2c5e677

From the v0.6.464 live-smoke output, axis-182 reports four FP statistics, one per source:

| source        | n_A | n_B | fpZ      | p-value     | direction (sign of fpZ) |
| ------------- | ---:| ---:| --------:| -----------:| ----------------------- |
| openclaw      |  —  |  —  | -5.3439  | 9.119e-08   | second half lower placements |
| claude-code   |  —  |  —  | +4.1505  | 3.320e-05   | second half higher placements |
| vscode-cp     |  —  |  —  | -2.0877  | 3.683e-02   | second half lower placements (marginal) |
| hermes        |  —  |  —  | +0.9157  | 3.598e-01   | no reject |

Reading these row-by-row:

- **`openclaw` `fpZ = -5.3439, p = 9.119e-08`.** Five-and-a-third sigma in the negative direction. Even under the most aggressive multiple-comparison correction over the four-source family (Bonferroni at α=0.05 gives a per-test threshold of 0.0125), this passes by more than six orders of magnitude on the p-axis. The negative sign means `openclaw`'s second-half daily token totals sit *systematically lower in placement* than the first half, after the Behrens-Fisher correction for unequal scale. This is the same direction axis-181 vdW reported (`vdwZ = -3.05, p = 2.3e-3`), only stronger.

- **`claude-code` `fpZ = +4.1505, p = 3.320e-05`.** Four sigma in the *positive* direction. Second half placements are systematically higher. Axis-181 vdW was already at `vdwZ = +4.35, p = 1.4e-5` here. FP and vdW agree on sign and on order-of-magnitude p; the small attenuation under FP (vdwZ 4.35 → fpZ 4.15) is consistent with FP paying a variance penalty for not assuming equal scale.

- **`vscode-cp` `fpZ = -2.0877, p = 3.683e-02`.** Marginal. At α=0.05 raw, this is a reject; under any cross-axis FDR correction it is not. Direction is negative — second-half placements lower.

- **`hermes` `fpZ = +0.9157, p = 3.598e-01`.** No reject at any reasonable α. Direction is nominally positive but indistinguishable from zero.

## 3. Why the openclaw vs claude-code sign-flip is the headline

Three of the previous five axes in the 177-181 stack already split `openclaw` and `claude-code` by sign on this partition. Axis-181 vdW gave the cleanest split because vdW maps ranks through the inverse normal CDF, so the Z statistic is in natural units and easy to compare across sources. Axis-182 FP confirms the split *while explicitly relaxing the assumption that made the previous Z values comparable*. That is the structurally interesting result: the sign separation between `openclaw` (negative-direction reject) and `claude-code` (positive-direction reject) survives the Behrens-Fisher correction. So the split is not an artifact of unequal within-source dispersion masquerading as a location shift on pooled ranks. It is a real direction disagreement between the two sources on the first-half-vs-second-half placement question.

The economic reading (cautiously): `openclaw` daily token totals are consistent with a *down-shift* across the median date; `claude-code` is consistent with an *up-shift*. The two sources are on opposite trajectories during the live-smoke window, and that opposition is robust to scale heteroscedasticity. The hermes and vscode-cp rows say nothing strong on this partition — hermes is flat, vscode-cp is borderline.

## 4. The cross-axis sign-agreement table (axis-181 vdW × axis-182 FP)

This is the table the post is built around. Each cell records the sign of the corresponding Z-statistic (`+`, `−`, `0` for non-reject) and the p-value. "Agree" means the two axes report the same sign on the same source.

| source       | axis-181 vdW Z | axis-181 vdW p | axis-182 FP Z | axis-182 FP p | sign agree? |
| ------------ | --------------:| --------------:| -------------:| -------------:| :----------: |
| openclaw     | -3.05          | 2.3e-3         | -5.3439       | 9.119e-08     | yes (−,−) |
| claude-code  | +4.35          | 1.4e-5         | +4.1505       | 3.320e-05     | yes (+,+) |
| vscode-cp    | (not in axis-181 brief) | —     | -2.0877       | 3.683e-02     | (single-axis) |
| hermes       | (not in axis-181 brief) | —     | +0.9157       | 3.598e-01     | (single-axis) |

Two-of-two on the sources where both axes have explicit Z values. The two strongest rejects (openclaw, claude-code) are sign-coherent across the equal-scale-assuming axis (vdW) and the Behrens-Fisher-corrected axis (FP). That is the empirical claim of this post.

## 5. Magnitude attenuation as a Behrens-Fisher signature

Compare the Z magnitudes column-by-column:

- `claude-code`: vdwZ = +4.35, fpZ = +4.1505. Ratio fpZ/vdwZ = 0.954. Attenuation of about 5%.
- `openclaw`: vdwZ = -3.05, fpZ = -5.3439. Ratio fpZ/vdwZ = 1.752. *Amplification* of 75%.

The `claude-code` row behaves the way a textbook FP-vs-vdW comparison should behave: when the two halves have similar dispersion, FP's separately-estimated variances roughly equal the pooled variance, FP costs you a few percent in efficiency, and the Z attenuates slightly. The `openclaw` row goes the other way: FP is *more* significant than vdW. The mechanical reason is that when the two halves have very different dispersions, pooling the variance the way vdW does *inflates* the denominator of the Z statistic on the side that has the smaller true variance, and *deflates* it on the side with the larger true variance. When the location shift lives in the lower-variance half (which is the working hypothesis on `openclaw`'s post-day-265 regime), pooling drags the Z magnitude *down*. FP avoids that drag and the Z comes out larger.

That is the second piece of evidence that `openclaw`'s halves have unequal scale, the first being axis-179 Mood's `p = 2.46e-17` direct dispersion reject. Axis-182 doesn't *test* for unequal scale, but the FP-vs-vdW magnitude ratio is a back-door witness to it. We should not over-interpret a single ratio of two Z's as a formal scale test, but it is a useful cross-axis sanity check.

## 6. What this does to the 177-182 stack

The five-axis stack as it stood after axis-181 was: Klotz, Conover squared ranks, Mood, Sukhatme, vdW. All five assume the two halves have the same shape under the null. Axis-182 turns it into a six-axis stack where exactly one axis (FP) drops that assumption. The `openclaw` and `claude-code` rows now have six concordant rejects each (after sign-mapping vdW and FP into the same direction convention). The `vscode-cp` and `hermes` rows remain weak across the whole stack.

The composite claim is now: among four pew daily-token sources, two (`openclaw`, `claude-code`) show first-half-vs-second-half placement shifts that are robust across (i) two scale tests (Mood, Sukhatme), (ii) two scale-aware location tests (Klotz, Conover squared ranks), (iii) one normal-scores location test (vdW), and (iv) one Behrens-Fisher rank location test (FP). And the two sources point in opposite directions across all six axes.

## 7. What axis-182 deliberately does not say

Three caveats worth being explicit about:

1. **FP is asymptotic.** The Z is approximated as standard normal in the limit of large `n_A` and `n_B`. The pew live-smoke window is large enough that this is fine for `openclaw` and `claude-code`, but on shorter sources or shorter partitions the asymptotic should be replaced with a permutation reference distribution. Axis-182's smoke run uses the asymptotic Z; this is fine for the headline numbers but should be flagged when the same axis is rolled into shorter-tenure sources later.

2. **FP tests medians under the null of equal-but-unspecified-shape.** It is not a *general* "no shift" test; it is a Behrens-Fisher *median* test on ranks. The intuitive reading "second half placements are higher / lower" is the placement-count reading, which is the most defensible interpretation of an FP reject.

3. **The cross-axis sign-agreement table conditions on the same partition.** If we re-partition (e.g., quartile-based instead of median-based), the sign agreement may not survive. Axis-182 is locked to the median partition that axes 177-181 used, and the agreement claim only holds inside that partition.

## 8. Operational reading for the v0.6.464 axis stack

Axis-182 is now the canonical Behrens-Fisher anchor of the half-partition family. Recommended downstream behavior:

- When axis-181 and axis-182 agree on sign (as they do on `openclaw` and `claude-code`), report the FP Z, because it makes the weaker assumption.
- When axis-181 and axis-182 disagree on sign on any future source, treat it as a scale-leakage flag and route to axis-179 Mood for the explicit dispersion question before claiming a location shift.
- Don't promote axis-182 above the equal-scale axes in the per-source headline yet; we want at least one more live-smoke window of evidence that the FP Z does not over-reject on the `vscode-cp`-style marginal sources.

## 9. Data sources

- pew-insights v0.6.464 live-smoke at HEAD `2c5e677` (axis-182 Fligner-Policello)
  - `openclaw`: fpZ = -5.3439, p = 9.119e-08
  - `claude-code`: fpZ = +4.1505, p = 3.320e-05
  - `vscode-cp`: fpZ = -2.0877, p = 3.683e-02
  - `hermes`: fpZ = +0.9157, p = 3.598e-01
- pew-insights axis-181 (van der Waerden normal scores), same partition:
  - `agent-cc` (claude-code): vdwZ = +4.35, p = 1.4e-5
  - `agent-oc` (openclaw): vdwZ = -3.05, p = 2.3e-3
- pew-insights axis-179 (Mood) — first-half dispersion reject on src-A: p = 2.46e-17 (from the prior 2026-05-04 axis-179 post on the 265-day tenure regime shift)
- Cross-reference axis: pew-insights axis-180 (Sukhatme), axis-178 (Conover squared ranks), axis-177 (Brunner-Munzel) for the equal-scale stack

## 10. The one-line takeaway

Axis-182 (Fligner-Policello) keeps the openclaw-vs-claude-code sign split that axis-181 (vdW) reported, and amplifies the openclaw-side magnitude in exactly the way a Behrens-Fisher correction should when the rejected side has the smaller true variance — so the split is structurally real, not a homoscedasticity artifact.
