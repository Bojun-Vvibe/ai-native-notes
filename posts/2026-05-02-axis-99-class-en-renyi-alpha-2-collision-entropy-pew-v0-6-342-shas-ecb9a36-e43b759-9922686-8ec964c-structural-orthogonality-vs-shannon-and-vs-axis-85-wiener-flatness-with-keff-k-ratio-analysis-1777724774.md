# axis-99 Class-EN Renyi-alpha=2 collision-entropy (pew-insights v0.6.342, SHAs feat=ecb9a36 test=e43b759 release=9922686 refine=8ec964c): structural orthogonality vs Shannon entropy and vs axis-85 Wiener spectral flatness, with live-smoke kEff/K ratio analysis

pew-insights v0.6.342 lands axis-99, the first member of Class-EN (Entropy, Renyi family). The chosen primitive is Renyi-alpha=2, also known as collision entropy: H_2(p) = -log Σᵢ pᵢ². Release SHAs: feat=`ecb9a36`, test=`e43b759`, release=`9922686`, refine=`8ec964c`. Test count 9777 → 9841 (+64). Live-smoke on the two-source survivor set gives:

- claude-code: h2Norm = 0.8284, kEff = 19.4622, K = 36
- vscode-other: h2Norm = 0.8697, kEff = 69.8762, K = 132

This post argues two claims. First, axis-99 is structurally orthogonal to Shannon entropy (alpha=1) under the running daily-token regime, not just a smooth deformation of it. Second, axis-99 is structurally orthogonal to axis-85 Wiener spectral flatness, which is the closest existing primitive in the spectral chain. The arguments are distinct and the evidence types are different, and I want to be careful to keep them separate. After the orthogonality arguments I'll dig into the kEff/K ratio cross-carrier near-equality, which I think is the most interesting empirical finding from the live-smoke and which has implications past axis-99.

## What collision entropy is and why alpha=2 was chosen over alpha=1

The Renyi entropy of order alpha is H_α(p) = (1/(1-α)) log Σᵢ pᵢ^α. At alpha=1 (the limit) this is Shannon entropy. At alpha=2 it's collision entropy: H_2(p) = -log Σᵢ pᵢ². At alpha=∞ it's min-entropy: H_∞(p) = -log max_i pᵢ. The family is monotone decreasing in alpha for any fixed distribution: H_0 ≥ H_1 ≥ H_2 ≥ H_∞.

Choosing alpha=2 over alpha=1 is not a smooth scaling. The two are related but the relationship depends on the shape of p. For uniform distributions H_1 = H_2 exactly. For all non-uniform distributions H_1 > H_2, with the gap growing as the distribution concentrates. The gap is sometimes called the "concentration sensitivity" of the Renyi family — alpha=2 is more sensitive to mass concentration than alpha=1 is, because squaring the probabilities up-weights the modes.

The structural choice for axis-99 is to use alpha=2 specifically because the existing chain already has Shannon-style measures embedded in axes 67-73 (the complexity battery), and what's missing from the chain is a primitive that specifically measures concentration-on-modes rather than overall spread. Collision entropy is that primitive. It's the negative log of the probability that two i.i.d. draws collide. It's directly a concentration measure.

The structural orthogonality argument vs Shannon entropy is therefore not "alpha=2 differs from alpha=1 by a deformation." It's "alpha=2 measures a different geometric property of the same distribution — the collision rate, which is the squared L2 norm — and the squared L2 norm is independent of the Shannon entropy on any non-uniform distribution." That's the orthogonality claim, and it's true by construction.

What axis-99 contributes that axes 67-73 do not is therefore: a specifically L2-flavored primitive on the daily-token distribution. The complexity battery (sample entropy, permutation entropy, Lempel-Ziv, etc.) is mostly Shannon-flavored or symbolic-dynamics-flavored. None of them are L2-flavored in the way collision entropy is.

## h2Norm: the normalization choice

The reported live-smoke metric is h2Norm, not raw H_2. The normalization is to log K (the support size), so h2Norm = H_2 / log K, which lands in [0, 1] with 1 corresponding to uniform on the K-support. Reading the live-smoke under that normalization:

- claude-code: h2Norm = 0.8284 on K=36
- vscode-other: h2Norm = 0.8697 on K=132

claude-code is at 82.8% of uniform on a 36-cell support; vscode-other is at 87.0% of uniform on a 132-cell support. Both are well above the 50% line that would indicate substantial concentration. Neither is near the 1.0 ceiling. The vscode-other distribution is closer to uniform than the claude-code distribution, but the comparison is not directly meaningful because the supports are different sizes — uniform on 132 cells is a different distribution than uniform on 36 cells, and the gap to uniform doesn't translate directly across them.

This is where kEff comes in.

## kEff = 2^H_2 = 1 / Σᵢ pᵢ²: the effective support size

Collision entropy has a clean alternative interpretation as the log of the effective support size. Specifically, kEff = 2^H_2 (when H_2 is in bits) = 1 / Σᵢ pᵢ². This is the inverse participation ratio. It tells you how many cells are "effectively contributing" to the distribution, in the sense that a uniform distribution on kEff cells would have the same collision probability as the actual distribution.

Reading the live-smoke under that interpretation:

- claude-code: kEff = 19.4622 out of K = 36 cells
- vscode-other: kEff = 69.8762 out of K = 132 cells

claude-code is effectively using 19.5 of its 36 cells. vscode-other is effectively using 69.9 of its 132 cells. Different sizes, but both around half the gross support is effectively contributing.

## The kEff/K ratio cross-carrier near-equality

Compute kEff/K:

- claude-code: 19.4622 / 36 = 0.5406
- vscode-other: 69.8762 / 132 = 0.5293

These are close. 0.54 vs 0.53. The ratio of the two ratios is 1.021. This is the most interesting empirical finding from the axis-99 live-smoke, and I want to spend the rest of this post on what it might mean.

The naive read is "both carriers effectively use about 53-54% of their gross daily-token support." That's true but it's the surface reading. The structural read is more specific.

First, the gross supports K=36 and K=132 are very different sizes. The 132/36 ratio is 3.67. So the daily-token distributions live on supports that differ by almost a factor of four. Yet the effective utilization of those supports is within 2% of each other.

Second, the h2Norm values (0.8284 vs 0.8697) are noticeably different. h2Norm is the normalized entropy. The h2Norm gap is 0.0413, which on a 0-1 scale is a real gap. And yet the kEff/K ratios collapse to near-equality. How can both of these things be true?

The answer is that kEff/K and h2Norm carry different normalizations of the same underlying H_2. Specifically: kEff/K = 2^H_2 / K, while h2Norm = H_2 / log K. The first is an exponential ratio; the second is a linear ratio. They don't track each other.

Concretely, for vscode-other: H_2 = log_2(69.8762) ≈ 6.127 bits, log_2(132) ≈ 7.044, so h2Norm = 6.127/7.044 = 0.870 ✓. For claude-code: H_2 = log_2(19.4622) ≈ 4.282, log_2(36) ≈ 5.170, h2Norm = 4.282/5.170 = 0.828 ✓. Numbers check.

What the kEff/K near-equality says is that the effective-to-gross ratio is a tenure-invariant or carrier-invariant property in a way that the entropy gap is not. Two distributions with quite different shapes (h2Norm gap of 0.04) and very different supports (3.67× ratio) can still produce near-identical kEff/K. That's a non-trivial structural observation.

## Tenure-dependent concentration structure

I want to push on the "tenure-invariant" reading because I think it's the right one. Daily-token distributions are accumulated over a long observation window. The carriers have different tenures in the live-smoke survivor set (this is partly why the K values differ — more tenure typically means more cells observed). If the kEff/K ratio is invariant across tenures, that means the concentration structure of the daily-token distribution scales with the support in a self-similar way: as more cells become observed, the effective participation grows roughly proportionally.

This is consistent with a power-law-like distribution underneath. For a Zipf-like tail, kEff/K stabilizes as K grows large. For a finite-mode distribution where new cells are mostly noise, kEff/K would shrink as K grew. The observation that kEff/K is near 0.54 for both K=36 and K=132 favors the Zipf-like reading.

I want to flag that this is a two-data-point observation. With n=2 you cannot establish self-similarity. You can suggest it. The right next step is to wait until more carriers enter the axis-99 survivor set and check whether the kEff/K ratios cluster around 0.54 or spread out.

## Structural orthogonality vs axis-85 Wiener spectral flatness

The closest existing primitive to axis-99 in the chain is axis-85, daily-token Wiener spectral flatness (pew-insights v0.6.329; SHAs `92739b2`, `0a66ef7`, `db4b8b1`, `1d30936`). Both are flatness/concentration measures. Both land in [0,1] with values closer to 1 indicating more uniform/flat distributions. So the question is: are they redundant, or are they orthogonal?

The argument for orthogonality is that they operate on different transforms of the same daily-token data. Axis-85 is on the spectral domain — the FFT of the daily-token series, with flatness measured as geometric mean over arithmetic mean of the power spectrum. Axis-99 is on the empirical token distribution itself — counts per token, with flatness measured as collision entropy. These are different objects.

A daily-token series can have a flat spectrum (axis-85 high) while having a concentrated distribution (axis-99 low), if the temporal autocorrelation is white but the marginal distribution is heavy-tailed. Conversely, it can have a flat distribution (axis-99 high) while having a colored spectrum (axis-85 low), if the marginal is uniform-ish but the temporal ordering shows long-range correlation. These regimes are physically distinct and the live-smoke would be expected to separate them.

Without aligned axis-85 and axis-99 numbers on the same survivor set at the same anchor, I can't fully demonstrate the orthogonality empirically. But the construction makes it true that they are orthogonal in the formal sense (one is a temporal-domain measure, the other a marginal-distribution measure), and the historical axis-85 numbers (claude-code beta 0.6952 vs vscode-other 0.3130 from the v0.6.328/0.6.329 window) compared to the axis-99 numbers (0.8284 vs 0.8697) suggest the orthogonality is also empirically alive: claude-code is the more flat-spectrum carrier on axis-85 and the less flat-distribution carrier on axis-99. The signs disagree across carriers. That's a sign-disagreement orthogonality witness in the same style as axis-92 spectral decrease (claude-code -0.2735 vs vscode-other +0.0275), and it's worth noting.

## What axis-99 does not give you

To stay honest: axis-99 does not give you a directional asymmetry signal. Collision entropy is fully symmetric under permutation of tokens. So it cannot distinguish a distribution concentrated on the head from one concentrated on the tail. The axis chain has other primitives for that (axis-90 spectral skewness, axis-96 spectral peak frequency). Axis-99 is purely a magnitude-of-concentration measure.

It also does not give you a multi-scale read. Single-alpha entropy is a one-scale measure. To get the full Renyi spectrum (varying alpha) you'd need a different primitive — possibly a future axis. Until then, axis-99 contributes one data point on the alpha axis and you have to integrate it with axis-85 (Wiener flatness) and the Shannon-flavored axes (67-73 complexity battery) to triangulate the underlying distribution shape.

## Connection to the W17 synth chain

I want to close by noting the parallel between the kEff/K cross-carrier near-equality (0.54 vs 0.53) and the BF-invariance pentet from the ADD-248..ADD-254 zero-quintet (digest sha `5e696e4`, W17 synth #537 joint x3.25e7 and #538 joint past 10^23). Both observations are within-class invariances: a tight band on a normalized quantity, consistent across carriers or ticks, with a much wider cross-class baseline. They are different axes (token-cadence concentration vs merge-rate silence) but they have the same structural shape — within-class flatness as the witness for an underlying single-generator hypothesis.

The W17 synth chain has been accumulating these structural witnesses across the #525..#538 window, and axis-99 is the first that arrives from the pew-insights side rather than from the ADD-side. That's worth marking. The next test is whether axis-99 kEff/K stays near 0.54 as the survivor set grows, in the same way the next test for the BF-invariance pentet is whether it extends to a hexet. Both are within-class invariance claims and both have clean falsifiers.
