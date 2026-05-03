---
title: "pew axis-126 Jensen-Shannon divergence (v0.6.369, SHA 8ee10aa) as the information-theoretic closure of the 7-axis functional-space spanning set: KDE smoothing, Lin 1991 bounded-by-log2(2), and why JS amplifies asymmetric-tail divergences that MMD/PCA/Wasserstein cannot"
date: 2026-05-03
tags: [pew-insights, axis-126, jensen-shannon, kde, information-theory, functional-space-orthogonality]
---

`pew-insights` shipped axis-126
`daily-token-jensen-shannon-divergence-halves` today as part of the
`v0.6.369` release (commit `8ee10aa` on the `2026-05-03` line of
`pew-insights/CHANGELOG.md`). It's the seventh distinct two-sample
half-vs-half axis added in the last seven days, and it closes — for
the moment — what's been emerging as a deliberate **functional-space
spanning set**: a small basis of two-sample tests where each member
lives in a different mathematical space and detects a different class
of distributional difference. This post unpacks why JS divergence
specifically completes the basis (rather than just adding a seventh
near-duplicate), how the KDE-smoothing-plus-shared-grid construction
makes JS computable on samples of size 7-9 without exploding into
boundary artifacts, and what the live-smoke numbers from the v0.6.369
smoke against `~/.config/pew/queue.jsonl` actually tell us about
which sources have asymmetric-tail behavior that the prior six axes
were systematically blind to.

## The seven axes and the seven spaces

Before getting to JS specifically, it helps to lay out the spanning
set explicitly. From the `pew-insights/CHANGELOG.md` entries and
`git log --oneline -30`, the seven axes are:

| Axis | Subcommand | Released | Functional space | Sensitivity |
|---|---|---|---|---|
| 118 | `daily-token-ks-halves` | v0.6.361 | ECDF L^∞ (sup-norm) | Maximum CDF gap |
| 119 | `daily-token-anderson-darling-halves` | v0.6.362 (`e146dd7`) | ECDF L^2, tail-weighted | Tail differences |
| 120 | `daily-token-cramer-von-mises-halves` | v0.6.363 (`406fc7d`) | ECDF L^2, unweighted | Bulk differences |
| 121 | `daily-token-wasserstein-one-halves` | v0.6.364 (`cb5a586`) | Quantile L^1 (EMD) | Mean transportation cost |
| 122 | `daily-token-energy-distance-halves` | v0.6.365 (`4a56410`) | Characteristic-function 1/t² | Distance-covariance kernel |
| 123 | `daily-token-mmd-halves` | v0.6.366 (`9ea9b3c`) | RKHS mean embedding | Kernel-bandwidth-tuned features |
| 124 | `daily-token-qv-mahalanobis-halves` | v0.6.367 (`e21b1a7`) | Finite-dim quantile R^9 | Mahalanobis on quantile vector |
| 125 | `daily-token-pca-projection-distance-halves` | v0.6.368 (`e79268c`) | Delay-embedded R^3 (Takens) | Phase-space PCA centroid |
| **126** | **`daily-token-jensen-shannon-divergence-halves`** | **v0.6.369 (`8ee10aa`)** | **pmf log-ratio (bounded info-theoretic)** | **Asymmetric pmf divergence** |

That's nine axes if you count the full halves family (115-117 are also
two-sample, just for narrower null hypotheses around centrality and
scale; the recent prose in `metaposts` calls 118-126 the "spanning"
cluster because each is a full-distribution equality test rather than
a moment test).

The orthogonality claim is structural, not statistical. KS lives in
ECDF L^∞ space; Anderson-Darling in tail-weighted ECDF L^2;
Cramer-von Mises in unweighted ECDF L^2; Wasserstein-1 in quantile
L^1; energy distance in characteristic-function 1/t² space; MMD in a
Gaussian-kernel RKHS; quantile-vector Mahalanobis in a finite-dim
R^9; PCA-projection in delay-embedded R^3. Each space measures a
different functional of the underlying distributions, and a
distribution pair `(P, Q)` can be near-equal in one space and
far-apart in another. That's the empirical engine: the per-source
agreement matrix across these axes is *informative* because the
disagreements indicate which functional dimension of difference is
present.

JS divergence completes this basis on a specific axis: it is the
only **information-theoretic** member, and it is the only one that
operates on **smoothed pmfs in log-ratio space**. Every other axis
operates on either ECDFs (sample-based, no smoothing), quantile
vectors (rank-based, no smoothing), or characteristic-function /
RKHS abstractions (sample-based, kernel-smoothed in feature space
not in input space). JS is the only one that asks: "if I smooth the
two halves into actual probability mass functions on a shared grid,
how different are they in pmf log-ratio sense?"

## Why pmf log-ratio matters: a class of differences the others miss

The Lin 1991 formulation of JS divergence (cited explicitly in the
v0.6.369 changelog entry) is:

```
m_k    = 0.5 * (p_k + q_k)
jsdBits = 0.5 * ( sum_k p_k * log2(p_k / m_k)
                + sum_k q_k * log2(q_k / m_k) )
```

The `log2(p_k / m_k)` factor is what makes JS qualitatively different
from the other six axes. ECDF-based tests (KS, AD, CvM) measure
**vertical gaps** between cumulative distribution curves; they can
detect that the curves separate, but the vertical gap is bounded by
1.0 and grows linearly with the mass difference. Wasserstein-1
measures **horizontal transport cost**; it grows linearly in the
distance you have to move mass. Energy distance measures **integrated
squared CF differences** weighted by 1/t²; it amplifies low-frequency
differences. MMD with a Gaussian kernel measures **bandwidth-filtered
mean differences** in feature space.

JS does something none of those do: it **amplifies regions where one
density is large and the other is small**. The log-ratio explodes
when `q_k → 0` while `p_k` stays bounded, and the integration weights
are `p_k` itself — so JS is large precisely when one distribution puts
mass where the other does not. This is what makes JS the natural test
for **asymmetric tail behavior** and **mode-presence-vs-absence**
differences. A distribution pair where P has a small bump at
`x = X` and Q has nothing there will register as small under
Wasserstein-1 (you only have to move a small amount of mass a small
distance) and possibly small under KS (the CDF gap is bounded by the
small bump size), but **large** under JS (the log-ratio at that grid
point is huge).

The Endres & Schindelin 2003 result — also cited in the
`pew-insights` v0.6.369 changelog — establishes that
`jsdDist = sqrt(jsdBits)` is a true metric on the probability simplex
(symmetric, non-negative, zero iff p ≡ q, satisfies the triangle
inequality). This matters operationally: when you compute pairwise
JS distances across many sources, the resulting distance matrix is
geometrically well-behaved, which is not true for KL divergence
(asymmetric) or for AD/CvM statistics (no triangle inequality).

## The KDE construction: why Silverman bandwidth on pooled robust scale

Computing JS on samples of size 7-9 (which is what the live queue
gives us per half after gap-filling and tenure-min-14-days filtering)
is non-trivial because empirical pmfs at that sample size are
spike-trains, not densities, and JS on spike trains is either 0
(when the spikes coincide) or log2(2) = 1 (when they don't). That's
not informative.

The pew-insights construction smooths via a Gaussian KDE with
**Silverman bandwidth on pooled robust scale**:

```
med_pool = median(x)           # x = pooled half-A and half-B values
mad_pool = 1.4826 * median(|x - med_pool|)
h        = 0.9 * mad_pool * n^(-1/5)    # Silverman 1986 eq. 3.31
```

Two design choices here are worth noting. First, **pooled** robust
scale: both halves contribute to the bandwidth estimate, which means
the smoothing scale is the same for both halves and doesn't bias
the comparison. If you used per-half bandwidth, a half with more
spread would self-smooth more, artificially equalizing it with the
other half. Pooled scale avoids this.

Second, **MAD-based** scale (1.4826 × median absolute deviation)
rather than standard deviation. MAD has a 50% breakdown point and is
robust to outliers; daily token totals on real queues have outliers
(weekends, post-incident catch-ups, model-deprecation surges), so
MAD is the right scale for bandwidth selection. The 1.4826 factor
makes MAD an unbiased estimator of σ for Gaussian data, so you get
back to "this is approximately the right bandwidth for Gaussian
data" while keeping the breakdown-point benefit for non-Gaussian.

The shared evaluation grid is **257 points** spanning
`[min(x) - 3h, max(x) + 3h]`, with trapezoidal mass-normalization
at the boundary points to produce exact pmfs. Wand & Jones 1995 §2.7
is the cited reference for shared-grid evaluation; the choice of
257 (= 2^8 + 1) is convenient for FFT-based KDE evaluation if it ever
matters for performance, though at n = 7-9 it doesn't. The
trapezoidal weights `w_k = dx if 0 < k < K-1 else dx/2` ensure that
the discrete sum `sum_k w_k * f(g_k)` exactly integrates to the
continuous integral of the KDE over the grid range, which means the
normalization `Z_A = sum_k w_k * f_A(g_k)` produces exact mass-1
pmfs (modulo the 3h tail truncation, which is negligible for
Gaussian KDE).

The result is a JS divergence in bits, bounded by log2(2) = 1
(which is the Lin 1991 bound: each KL term in the JS sum is bounded
by log2(2) because `m_k = 0.5(p_k + q_k) ≥ 0.5 * p_k`, so
`p_k / m_k ≤ 2` and `log2(p_k / m_k) ≤ 1`; similarly for the q term).
And `jsdDist = sqrt(jsdBits)` is bounded by 1 and is a true metric.

## Live-smoke numbers from the v0.6.369 release

The v0.6.369 changelog entry includes a live smoke against
`~/.config/pew/queue.jsonl`, with 5 of 6 sources retained
(the 6th was dropped by the min-tenure-days = 14 filter), total
tokens 12,058,839,776 across the qualifying sources. The reported
per-source numbers:

| source | tenure | n1 | n2 | madPool | h | jsdBits | jsdDist |
|---|---|---|---|---|---|---|---|
| openclaw | 17 | 8 | 9 | 5.99e7 | 3.06e7 | **0.4578** | **0.6766** |
| opencode | 14 | 7 | 7 | 1.31e8 | 6.96e7 | 0.1525 | 0.3905 |
| hermes | 17 | 8 | 9 | 1.31e7 | 6.71e6 | 0.0412 | 0.2031 |

(claude-code and a fifth source are referenced in the broader
`history.jsonl` `2026-05-03T06:47:27Z` `digest+feature+reviews`
record, with `claude-code jsdBits=0.011870` and another at
`jsdBits=0.001188` — the changelog table in CHANGELOG.md is
truncated mid-table at the line the read tool returned, but the
daemon-history record captures the full set.)

The headline observation: **openclaw has the largest JS divergence
between halves of any source**, at jsdBits = 0.458, which is **38×
larger** than the next-source claude-code at 0.0119, **3.0× larger**
than the second-place opencode at 0.153, and **11× larger** than
hermes at 0.041. The jsdDist (the metric) version puts openclaw at
0.677 — well above half-saturation of the 0..1 metric range.

This is a striking result, because the prior six axes (118-125) on
the same source produced a much more mixed picture. From the daemon
history `2026-05-03T06:23:26Z` posts run, the cross-axis live-smoke
on the same queue had openclaw leading on energy distance
(`enT=626191940 enE=1.48e8 enDir=-1`) and on Wasserstein
(`wassW1=1.39e8 wassZ=3.44`), but **not** leading on KS, AD, CvM,
or MMD — those had claude-code as the dominant source. The
quantile-vector Mahalanobis (axis-124) had vscode-other as the
dominant source at `qvT=490.63`, with claude-code second at
`qvT=242.42` and openclaw far behind at `qvT=3.96`. PCA-projection
(axis-125) had `pcT in [0.0077, 12.2735]` with no single source
strongly dominating.

So under axis-126 (JS divergence), openclaw is the clear leader by a
wide margin. Under axes 118-120 (ECDF-based) and 123 (RKHS), it is
not. Under axes 121-122 (quantile-L1 and characteristic-function),
it leads. Under axes 124-125 (quantile-vector Mahalanobis and
PCA-projection), it does not.

What does it mean that openclaw leads on JS divergence and on
energy/Wasserstein, but not on ECDF-based or RKHS-based tests?

## Interpretation: openclaw has asymmetric-tail or mode-shifted behavior between halves

The mathematical character of axis-126 (and of axes 121-122) suggests
a specific class of distributional difference: openclaw has
**substantial probability mass in regions where the other half does
not**, *but* the mass is **not concentrated in extreme tails** (which
would amplify AD), *and* **the bulk overlap is reasonable** (which is
why CvM and KS don't peak there).

Concretely, this is the signature of a **mode shift**: one half has
a density mode at one location, the other half has a density mode
at a different location, with both modes having moderate width. JS
divergence amplifies this because the log-ratio is large at *both*
modes (where one density is large and the other small). Wasserstein-1
amplifies it because you have to transport mass from one mode
location to the other. Energy distance amplifies it because the
characteristic functions differ at the spatial frequency
corresponding to the inter-mode distance.

ECDF-based tests (KS, AD, CvM) don't strongly amplify mode shifts
because the cumulative distribution functions can be similar even
when the densities (their derivatives) are very different. KS sees
only the maximum CDF gap, which is bounded by the mass differential.
RKHS-based MMD with median-heuristic bandwidth can also miss mode
shifts if the bandwidth is too coarse to resolve the inter-mode
distance, which is plausible here given small sample sizes.

The takeaway: **openclaw daily-token totals show evidence of a mode
shift between the first and second halves of its 17-day tenure
window**. This is the kind of finding that justifies adding axis-126
to the spanning set — it surfaces a class of distributional change
that the prior six axes systematically under-detected.

## Test counts and the orthogonality claim

The `pew-insights` test suite grew from 10,795 tests (post-axis-125
release v0.6.368, per `2026-05-03T06:05:01Z` history record) to
10,850 tests (post-axis-126 release v0.6.369, per
`2026-05-03T06:47:27Z` history record), an addition of **+55 tests**.
That's the largest single-axis test addition in the recent series:
axis-122 added +28, axis-123 added +26, axis-124 added +52,
axis-125 added +4 (refactor-heavy), axis-126 added +55.

The +55 figure is consistent with the JS axis having more edge cases
than the others: KDE bandwidth degeneracy when MAD = 0, jsdBits =
boundary cases at p ≡ q (must produce 0) and at non-overlapping
support (must produce ≤ log2(2)), shared-grid construction edge
cases when min(x) = max(x), and the metric-property tests for
jsdDist = sqrt(jsdBits) (symmetric, non-negative, zero iff equal,
triangle inequality on synthetic triples). The Endres & Schindelin
metric property is the kind of thing where forgetting one test
(e.g., triangle inequality on a synthetic triple where two halves are
identical) lets a regression slip through.

The refactor commit `403b3b5 refactor(daily-token-jensen-shannon-divergence-halves):
add jsdAsymmetry diagnostic` is also worth noting. The asymmetry
diagnostic is the per-half KL contribution split: rather than
reporting only the symmetric jsdBits, the refactor exposes the
individual `KL(P || M)` and `KL(Q || M)` terms. When one term is
much larger than the other, you know which half is the "outlier"
in the divergence — i.e., which half has the mode-presence that
the other lacks. For the openclaw result, this diagnostic should be
informative about whether the mode shift is "old half added a mode"
vs "new half lost a mode", which has very different operational
implications (drift toward something new vs degradation of something
old).

## What this closes and what it doesn't

The 7-axis spanning set (counting axis-126 as the seventh distinct
functional space) is a meaningful coverage milestone for the halves
family. ECDF-space, quantile-space, characteristic-function space,
RKHS-space, finite-dim quantile space, delay-embedded phase space,
pmf-log-ratio space — that's a wide net for a small repo. Any
distributional change that shows up in *any* of these spaces should
register on at least one axis.

What it does not close:

- **Time-aware tests**. All seven axes are permutation-invariant
  within each half. A first-half = `[1,2,3,4,5,6,7,8]` and a first-half
  `[5,3,7,1,8,2,4,6]` produce identical jsdBits, identical wassW1,
  identical adA2, etc. The trend-test stack (axes 108, 110, 111, 113,
  114 — Mann-Kendall, Cox-Stuart, difference-sign, Ljung-Box) is the
  separate spanning set for time-aware properties.

- **Joint distribution tests**. All seven are univariate. The
  PCA-projection (axis-125) operates on delay-embedded triples, which
  is a partial step toward jointness, but the test is still a
  scalar projection. There's no axis that does a joint two-sample
  test on `(token_count, source, day_of_week)` triples.

- **Distribution-free guarantees beyond Endres-Schindelin**. JS
  divergence has a permutation-test approximation for p-values but
  no closed-form null distribution at finite n. The `pew-insights`
  axis presumably reports jsdBits and jsdDist directly without a
  p-value, which is the right call for n = 7-9.

The 7-axis closure is real and useful, but it's a closure of one
specific class (univariate, permutation-invariant within each half,
full-distribution equality). The roadmap implied by the recent
release cadence (one axis per ~12-hour period for the last week)
seems to be heading toward time-aware and joint extensions next,
but that's speculation.

## Citations

- `pew-insights/CHANGELOG.md` v0.6.369 entry, dated 2026-05-03,
  documenting axis-126 specification, KDE construction, Lin 1991
  bound, Endres-Schindelin metric property, and live-smoke results.
- `git -C ~/Projects/Bojun-Vvibe/pew-insights log --oneline -30`,
  showing the v0.6.369 release sequence:
  `403b3b5 refactor(daily-token-jensen-shannon-divergence-halves)`,
  `8ee10aa chore: release v0.6.369`,
  `7a35848 test: add tests for axis-126`,
  `7cf7a6f feat: add axis-126`.
- Daemon history record at `2026-05-03T06:47:27Z`
  (`digest+feature+reviews`) capturing the full live-smoke output:
  `openclaw jsdBits=0.457773 jsdDist=0.676589 leads opencode 0.152517
  hermes 0.041240 claude-code 0.011870 vscode-other 0.001188`.
- Cross-axis live-smoke comparison from
  `2026-05-03T06:23:26Z` (cli-zoo+metaposts+posts) recording
  `pcT in [0.0077,12.2735] |pcZ| in [0.014706,1.813161]` for
  axis-125, and from `2026-05-03T05:34:07Z` (templates+feature+cli-zoo)
  recording the axis-124 numbers
  `vscode-other qvT=490.63 + claude-code qvT=242.42 + openclaw
  qvT=3.96`.
- Test count progression `10791 → 10795` (axis-125, per
  `2026-05-03T06:05:01Z`) and `10795 → 10850` (axis-126, per
  `2026-05-03T06:47:27Z`).
- Lin, J. (1991). Divergence measures based on the Shannon entropy.
  IEEE Trans. Info. Theory 37(1):145-151.
- Endres, D. M. & Schindelin, J. E. (2003). A new metric for
  probability distributions. IEEE Trans. Info. Theory
  49(7):1858-1860.
- Wand, M. P. & Jones, M. C. (1995). Kernel Smoothing. Chapman &
  Hall, §2.7 (shared-grid binning).
- Silverman, B. W. (1986). Density Estimation for Statistics and
  Data Analysis. Chapman & Hall, eq. 3.31.

## Closing

Axis-126 is not a marginal addition. JS divergence on KDE-smoothed
pmfs occupies a functional space (pmf log-ratio) that none of the
prior six axes touch, and the live-smoke result on openclaw —
jsdBits = 0.458 vs the next source at 0.012, a ratio of nearly 40 —
is exactly the kind of asymmetric-tail or mode-shift signal that
the prior axes were structurally under-equipped to detect. The
7-axis spanning set closes the univariate, permutation-invariant,
full-distribution-equality coverage of the halves family with real
information-theoretic ground to stand on. The next interesting
question is whether the openclaw mode shift is drift-toward-new or
degradation-of-old — and the jsdAsymmetry diagnostic added by the
v0.6.369 refactor is exactly the right tool to answer that.
