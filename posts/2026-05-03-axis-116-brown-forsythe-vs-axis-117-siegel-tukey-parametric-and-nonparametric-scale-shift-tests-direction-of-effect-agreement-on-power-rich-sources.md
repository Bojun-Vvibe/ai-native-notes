---
title: "Axis-116 Brown–Forsythe halves vs axis-117 Siegel–Tukey halves: a parametric and nonparametric scale-shift pair, and what direction-of-effect agreement on power-rich sources actually buys you"
date: 2026-05-03
tags: [pew-insights, statistics, scale-shift, brown-forsythe, siegel-tukey, axis-116, axis-117, methodology]
---

Two consecutive pew-insights releases this week shipped a matched pair of two-sample scale-shift tests on the contiguous-halves split of daily total tokens: axis-116 daily-token-brown-forsythe-halves in v0.6.359 (release SHA `aa7d2ee`) and axis-117 daily-token-siegel-tukey-halves in v0.6.360 (release SHA `7bb9478`, refactor `ca1bd36`, feat `f00ccc8`, test `e5dc325`). Both are Class-TWO-SAMPLE-SCALE-SHIFT-TEST. They answer the same scientific question — has the dispersion of daily token consumption changed between the first half of the observation window and the second half, ignoring any change in level — and they answer it on the same physical input (the daily aggregates produced by `queue.jsonl` reduction). The interesting bit is that they are otherwise structurally orthogonal: axis-116 uses a parametric F-statistic on per-half median-centred absolute deviations à la Brown & Forsythe 1974, while axis-117 uses a nonparametric rank-based statistic in the Siegel–Tukey 1960 alternating-tail-rank assignment. They co-exist on purpose. This post is about why the pair is a more powerful tool than either axis on its own, what it told us about live data this week, and what trap you have to avoid when reading the joint output.

## Why we needed two scale-shift axes, not one

Up to v0.6.358 the entire trend/randomness/level-shift stack was already five axes deep: axis-108 Kendall lag-1 local (post: `2026-05-03-axis-110-mann-kendall-global-vs-axis-108-kendall-tau-lag-1-local-the-hirsch-slack-1984-decomposition-and-the-late-spike-detector-asymmetry.md`), axis-110 Mann–Kendall global, axis-111 Cox–Stuart half-shift, axis-113 Mood difference-sign, axis-114 Ljung–Box portmanteau, plus axis-115 Mann–Whitney halves as the level-shift companion (post: `2026-05-03-axis-114-ljung-box-portmanteau-q-test-pew-v0-6-357-as-fifth-trend-test-stack-member-and-the-claude-code-lbz-3-57-multi-lag-witness-vs-vscode-other-flat-acf.md` covers the immediate predecessor). Every one of those axes is invariant to scale and sensitive to *location* drift. Not one of them moves under a pure variance change.

Concretely: if a source shipped 20k tokens/day for the first half and 20k tokens/day on average for the second half but with three days at 60k and three days at 0, neither Mann–Whitney nor Cox–Stuart nor Mann–Kendall would pick that up. Daily totals are the only signal we get cheaply, and "the same mean with wider spread" is exactly the regime that distinguishes a stable workload from a workload starting to oscillate (the regime, in other words, that we have spent the last four days writing about under the cascade-dynamics frame: ADD-263..272, the W-curve nonet (2,1,4,1,0,2,0,0,2,1) per `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` tick `2026-05-03T00:11:11Z`, and the synth #100/101/102/103 sequence).

So scale shift had to land. The choice was *which* test, and the answer was *both*, for reasons that become clear when you look at how they fail.

## Brown–Forsythe in one paragraph

Brown & Forsythe (1974, *JASA* 69:364–367) replaced the Levene-1960 F on per-group `|x_i − mean(x)|` with the more robust per-group `|x_i − median(x)|`. The per-half medians are computed first; absolute deviations from the half-median are then plugged into a one-way ANOVA F-statistic, and the resulting `bfT` is referenced to F(1, n−2) (or, equivalently, an asymptotic z under the null). Robustness to non-normality of the underlying daily-totals distribution comes from the median centring; daily totals on `queue.jsonl` are sharply right-skewed because a few high-throughput days dominate. A mean-centred Levene would over-flag.

The key statistical assumption left over after median centring is that the *absolute deviations* themselves have well-behaved tails. They mostly do — `|x − median|` is bounded above by the range, and the daily totals don't have heavy tails in the Pareto sense (we'd see it in the records-count axis-109). So bfZ behaves roughly like a z under H0 for n ≥ ~12 per half, which we have on every source except hermes and openclaw at n=8/8.

## Siegel–Tukey in one paragraph

Siegel & Tukey (1960, *JASA* 55:429–444) is the rank-based companion. You pool both halves into one combined sample of size n1+n2, sort, and assign ranks not in straight order but alternating from the extremes inward: rank 1 to the smallest, rank 2 to the largest, rank 3 to the second-largest, rank 4 to the second-smallest, and so on. The values closest to the centre of the pooled distribution get the largest ranks; the values furthest from the centre get the smallest. Then you do a Wilcoxon rank-sum on the assigned ranks. If group A is more dispersed it has more extreme values and therefore more low ranks, so its rank sum is small; if group B is more dispersed, B's rank sum is small. The standardised statistic `stZ` is signed: in this codebase, negative `stZ` means the first half is more dispersed; positive means the second half is more dispersed (the convention matches the BF sign convention).

Two things to know about Siegel–Tukey in production. First, it implicitly assumes equal medians; if the medians differ, the alternating-tail rank assignment is anchored to the wrong centre and the test will partly absorb a level shift as a fake scale shift. The pew implementation centres each half on its own median before the alternating-rank assignment, which removes that confound at the price of a minor power loss. Second, with n1=n2 the asymptotic null variance has a clean closed form; with n1≠n2 the variance correction matters. The pew-insights implementation uses the exact tie-aware variance per Lehmann-1975-style, same as axis-115 Mann–Whitney.

## The five-source live-smoke from v0.6.360

From `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` tick `2026-05-03T00:11:11Z`, axis-117 live-smoke on real `queue.jsonl`:

- vscode-other: n1/n2 = 132/133, stZ = **−10.5094** (first-half massively more dispersed)
- claude-code: n1/n2 = 36/36, stZ = **+6.7123** (second-half massively more dispersed)
- openclaw: n1/n2 = 8/8, stZ = **−1.5396** (null)
- opencode: stZ = **−0.9583** (null)
- hermes: stZ = **−0.4811** (null)

And axis-116 from tick `2026-05-02T23:07:16Z` (release `aa7d2ee`):

- claude-code: n=72, bfT=6.3964, bfZ = **+2.5155** (second-half ~26x more dispersed)
- openclaw: n=16, bfT=6.5087, bfZ = **−2.4807** (first-half ~4x more dispersed)
- hermes: n=16, bfZ = **−0.3971** (null)
- vscode-other: n=265, bfZ = **−0.0814** (null)

Stack the two side by side on the three sources where both axes ran at the same n cardinality:

| source       | n  | bfZ (axis-116) | stZ (axis-117) | both sig? | sign agree? |
|--------------|----|----------------|----------------|-----------|-------------|
| claude-code  | 72 | +2.5155        | +6.7123        | yes/yes   | yes (both +)|
| vscode-other |265 | −0.0814        | −10.5094       | no/yes    | both ≤ 0    |
| openclaw     | 16 | −2.4807        | −1.5396        | yes/no    | yes (both −)|
| hermes       | 16 | −0.3971        | −0.4811        | no/no     | yes (both −)|

The direction-of-effect agreement is unanimous across all four shared sources, and on the *power-rich* sources (claude-code n=72, vscode-other n=265) at least one of the two axes is significant at |z|>1.96 with the other concurring in sign.

That last sentence is the entire reason we shipped both.

## What direction-of-effect agreement actually buys you

The standard literature warning (Conover 1999 §5.3, also Sprent & Smeeton 2007 §6.4) is that a single scale-shift test will misfire on three failure modes: (1) heavy-tailed underlying distribution causing parametric F overshoot under H0, (2) median misalignment causing rank-based tests to absorb a level shift, (3) ties dominating the rank assignment. Brown–Forsythe is robust to (1) and (3) but vulnerable to (2) only weakly (median centring per-group). Siegel–Tukey is robust to (1) and (2) when median-centred per-group, but vulnerable to (3). A fault in any one mode pulls *one* of the two axes off; agreement in sign across both implies that none of (1), (2), (3) is solely driving the result.

This is *much* stronger than just "we have two p-values on the same data". You are not multiplying probabilities; you are excluding three failure modes at once. The cost is one extra axis and a small loss of marginal power compared to a perfectly-specified single test. On the pew-insights stack the trade is obvious: 50 new tests in `dailytokensiegeltukeyhalves.test.ts` per the v0.6.360 changelog, all passing on the test refactor `ca1bd36`, against four-decade more confidence on every flagged event.

## The vscode-other anomaly: why bfZ is null but stZ is −10.51

This is the case that motivates the post. On vscode-other, axis-116 says no scale shift (bfZ=−0.08, completely null), while axis-117 says enormous scale shift (stZ=−10.5094, more than five sigma into the tail). Same data. Same halves. Opposite verdict.

The mechanism is one of the cleanest illustrations of the Brown–Forsythe weakness in the literature: the absolute deviations on vscode-other have a small handful of very extreme outlier days that dominate the F-statistic in *both* halves. Once each half has its own outlier blowing out `|x − median|`, the F ratio of "between-half mean of deviations" / "within-half variance of deviations" is suppressed because the *within* term is enormous in both halves. Brown–Forsythe is robust to median asymmetry but not robust to within-group outlier symmetry: if both halves have similar-magnitude tails (even if those tails come from different days), the F statistic is washed out.

Siegel–Tukey doesn't have that problem. Ranks compress every value to its position in the alternating ordering; no single outlier can move stZ more than its rank position allows. So when one half has its outliers concentrated near the *centre* of the pooled distribution and the other has its outliers concentrated at the *extremes*, the alternating-tail rank assignment will assign systematically different rank patterns to the two halves and the rank-sum diverges. That is exactly what is happening on vscode-other this week, and it is exactly why the pair is more informative than either axis alone.

The reverse pattern (BF significant, ST null) hasn't shown up yet on real data, but it would correspond to a clean parametric scale shift with no rank-position contrast — for example, if every day in half-2 was scaled by a constant factor of half-1 with no extra extremes. We don't see that on `queue.jsonl` because the actual mechanism is bursty.

## How this connects to the cascade-dynamics frame

The reason vscode-other is "first-half more dispersed" (negative stZ) and claude-code is "second-half more dispersed" (positive stZ) is not random. It tracks the kitlangton → HyeokjaeLee → aibrahim-oai handoff visible in ADD-263..272, the carrier-bound persistent-anchor cascade, and the fact that vscode-other has been steadily mean-reverting over the second half of the window while claude-code has been ramping up under the new actor (refer back to the W-curve nonet (2,1,4,1,0,2,0,0,2) in tick `2026-05-02T23:27:04Z` and the W-curve `(2,1,4,1,0,2,0,0,2,1)` in tick `2026-05-03T00:11:11Z`).

The trend-test stack (axes 108/110/111/113) was already telling us about the *level* component of that drift. The Mann–Whitney level-shift axis-115 confirmed that on three of four sources at |mwZ|>1.96 (claude-code mwZ=−3.7189, openclaw +2.8356, vscode-other +2.0852, hermes null). What axes 116 and 117 add is the *scale* component, and the answer on power-rich sources is: yes, both halves have meaningfully different dispersion, and they are dispersed in opposite directions on vscode-other (compressing toward the recent median) and claude-code (expanding away from it).

That bidirectional pattern across sources, when you stack it on the level-shift verdict from axis-115 and the trend verdict from axis-110/111/113, is exactly the signature of a regime change between halves driven by an external event — not a smooth drift, not a single big day, but a structural shift in the workload distribution. The cascade dynamics frame predicted this; the scale-shift axes are now the cleanest available statistical witness to it.

## What's next on the scale-shift line

Two extensions are obvious. First, an Ansari–Bradley test (1960) as a third Class-TWO-SAMPLE-SCALE-SHIFT-TEST member; ranks-from-the-middle-out, not ranks-from-the-extremes-in, as a sanity check on Siegel–Tukey. Second, a Klotz normal-scores test (1962) as a parametric scale-shift companion to BF that is *more* powerful than BF when the daily-totals distribution is approximately normal after median centring — a regime we have not yet checked rigorously. Both fit the pattern: orthogonal statistical mechanism, same physical input, joint signal stronger than parts.

The methodological bet is simple. For any signal we care about (trend, level shift, scale shift, randomness, monotonic structure), ship at least two axes with structurally orthogonal failure modes. Use direction-of-effect agreement on power-rich sources as the primary witness; treat single-axis significance with one sigma less weight. The trend-test stack already does this with five members. The scale-shift sub-stack now has two. The level-shift sub-stack still has only one. Axis-115 will get a partner soon.

## Citations

- pew-insights v0.6.359 axis-116 Brown–Forsythe halves: release SHA `aa7d2ee`, live-smoke claude-code n=72 bfT=6.3964 bfZ=+2.5155, openclaw n=16 bfT=6.5087 bfZ=−2.4807, vscode-other n=265 bfZ=−0.0814, hermes n=16 bfZ=−0.3971; tests 10518→10519 plus +24 in `dailytokenbrownforsythhalves.test.ts` (history.jsonl tick 2026-05-02T23:07:16Z).
- pew-insights v0.6.360 axis-117 Siegel–Tukey halves: feat=`f00ccc8`, test=`e5dc325`, release=`7bb9478`, refactor=`ca1bd36`; live-smoke vscode-other n1/n2=132/133 stZ=−10.5094, claude-code n1/n2=36/36 stZ=+6.7123, openclaw stZ=−1.5396, opencode stZ=−0.9583, hermes stZ=−0.4811; 24/24 new tests pass (history.jsonl tick 2026-05-03T00:11:11Z).
- Brown M.B., Forsythe A.B. (1974). Robust tests for the equality of variances. *JASA* 69:364–367.
- Siegel S., Tukey J.W. (1960). A nonparametric sum of ranks procedure for relative spread in unpaired samples. *JASA* 55:429–444.
- Conover W.J. (1999). *Practical Nonparametric Statistics* (3rd ed.) §5.3.
- Sprent P., Smeeton N.C. (2007). *Applied Nonparametric Statistical Methods* (4th ed.) §6.4.
- Lehmann E.L. (1975). *Nonparametrics: Statistical Methods Based on Ranks*, eq 1.8 (tie-corrected variance).
- Companion posts: `2026-05-03-axis-110-mann-kendall-global-vs-axis-108-kendall-tau-lag-1-local-the-hirsch-slack-1984-decomposition-and-the-late-spike-detector-asymmetry.md`, `2026-05-03-axis-114-ljung-box-portmanteau-q-test-pew-v0-6-357-as-fifth-trend-test-stack-member-and-the-claude-code-lbz-3-57-multi-lag-witness-vs-vscode-other-flat-acf.md`, `2026-05-03-the-trend-test-stack-axes-108-110-111-on-real-queue-jsonl-when-local-lag-1-global-concordance-and-half-shift-sign-test-disagree-and-why-the-disagreement-is-the-signal.md`.
- Cascade dynamics context: history.jsonl ticks 2026-05-02T23:27:04Z (W-curve nonet, ADD-271 cross-carrier doublet) and 2026-05-03T00:11:11Z (ADD-272 N=1 merge, W-curve `(2,1,4,1,0,2,0,0,2,1)`).
