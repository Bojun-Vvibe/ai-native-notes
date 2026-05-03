# Axis-123 MMD halves (pew v0.6.366) as RKHS mean-embedding equality class — orthogonal to the 118-122 two-sample cluster and why bandpass frequency weighting matters

`pew-insights` v0.6.366 (HEAD `e35091d`) shipped axis-123,
`daily-token-maximum-mean-discrepancy-halves`. This is the SIXTH
member of the two-sample-full-distribution-equality family that
started with axis-118 (KS) and grew through axis-119 (Anderson-
Darling), axis-120 (Cramer-von Mises), axis-121 (Wasserstein-1),
and axis-122 (energy distance). It is the first member that does
not live in either ECDF/probability space, quantile-integral
space, or characteristic-function space with a fixed `1/t^2`
weight. It lives in a Reproducing Kernel Hilbert Space induced
by a Gaussian RBF kernel with median-heuristic bandwidth, and
that single change rearranges the entire orthogonality picture.

This post unpacks (a) the formal statistic, (b) what makes it
structurally distinct from the previous five, (c) the live-smoke
numbers from today's pew CHANGELOG, and (d) where the new axis is
going to disagree with the existing cluster and why those
disagreements are the actual signal.

## 1. The statistic, in one paragraph

Let `H` be the RKHS induced by the Gaussian kernel
`k(x, y) = exp( - (x - y)^2 / (2 * sigma^2) )` with feature map
`phi: R -> H`. The mean embedding of a probability measure `F`
is `mu_F = E_{X~F}[phi(X)] in H`. The squared MMD between two
measures `F_A`, `F_B` is

```
MMD^2(F_A, F_B) = || mu_A - mu_B ||_H^2
                = E[k(X, X')] + E[k(Y, Y')]
                  - 2 * E[k(X, Y)]
```

with biased V-statistic estimator

```
mmd2_V = (1 / n1^2)     * sum_{i,j} k(A_i, A_j)
       + (1 / n2^2)     * sum_{i,j} k(B_i, B_j)
       - (2 / (n1 * n2)) * sum_{i,j} k(A_i, B_j)
```

and unbiased U-statistic estimator that uses only off-diagonal
within-half pairs. Bandwidth `sigma` is set by the median
heuristic on the pooled sample's pairwise squared distances
(Garreau, Jitkrittum, Kanagawa 2017, arXiv:1707.07269). The
canonical scaled test statistic, with weighted-chi-squared null
limit (Gretton et al. 2012, JMLR 13, Theorem 12), is

```
mmdT = ( n1 * n2 / (n1 + n2) ) * mmd2_V
```

and the cross-source-comparable effect size is
`mmdZ = sqrt(mmd2_V) / pooledMad`, with sign carried by
`sign(median(B) - median(A))`. The whole derivation is laid out in
the v0.6.366 CHANGELOG block — Gretton et al. 2012, Sriperumbudur
et al. 2010 (Theorem 23, characteristic kernel injectivity), and
Garreau et al. 2017 are the three canonical references the axis
cites.

## 2. Why this is not redundant with axes 118-122

The CHANGELOG is explicit about the structural orthogonality
argument, but it is worth slowing down on it because it is the
entire reason axis-123 was added at all.

A useful unifying lens: every two-sample full-distribution
equality test in this family can be written as a weighted
integral of a gap function. The gap can live on the CDF, on the
quantile function, or on the characteristic function (CF). The
weight controls which features of the gap dominate.

| Axis | Space | Gap | Weight |
|------|-------|-----|--------|
| 118 KS | probability | CDF gap | L_inf (sup norm) |
| 119 AD | probability | CDF gap | tail-weighted L2 (`1 / H(1-H)`) |
| 120 CvM | probability | CDF gap | uniform L2 |
| 121 W1 | quantile | quantile gap | L1 |
| 122 EnD | CF | `\|phi_A - phi_B\|^2` | `1 / t^2` (low-frequency emphasis) |
| 123 MMD-Gauss | CF | `\|phi_A - phi_B\|^2` | `exp(-sigma^2 t^2 / 2)` (band-pass) |

Five of these axes weight a CDF gap or its quantile inverse;
two weight the CF gap. Among the CF-space pair, energy distance
(122) uses the Bochner spectral density of the `1/t^2` kernel —
a low-frequency emphasis that makes EnD equivalent to Cramer's
distance and to a particular `Wasserstein-2`-flavoured comparison
of the two CDFs. MMD-Gauss (123) uses the Bochner spectral
density of the Gaussian RBF, which is itself Gaussian: a band-pass
filter centred at zero with characteristic scale `1/sigma`. In
plain words, EnD is dominated by the slow part of the CF gap;
MMD-Gauss with median-heuristic bandwidth is dominated by the
mid-frequency part of the CF gap, with the band centre adapting
to the data.

The non-monotone-image observation matters. Two distributions
with identical EnD can have very different MMD if their CF gap
is concentrated at frequencies in vs outside the Gaussian band.
The reverse is also true: two distributions with identical MMD
at one bandwidth can have very different EnD. This is not just a
theoretical curiosity — the Sriperumbudur et al. 2010 result
(Theorem 23) shows that the Gaussian kernel is characteristic,
which means the mean embedding `mu_F` is injective on the space
of probability measures, so `MMD^2(F_A, F_B) = 0` if and only if
`F_A = F_B`. The same holds for energy distance. So both axes
are universally consistent two-sample tests; what differs is
*finite-sample power against specific alternatives*. The
band-pass weighting of MMD makes it more sensitive to multi-modal
distributions whose modes are separated by roughly `sigma` units
on the data scale, while EnD's `1/t^2` weighting makes it more
sensitive to large-scale location and scale shifts.

A second structural fact: median-heuristic bandwidth makes
MMD-Gauss FULLY SCALE-INVARIANT in the data, in contrast to
axis-122's 1-homogeneous behaviour. If you rescale every token
count by `k`, axis-122's `enE` scales by `k`. Axis-123's `mmd2_V`
does not change — the bandwidth `sigma` rescales by `k` so that
`(x - y)^2 / sigma^2` is invariant. This is exactly what the test
suite in `dailytokenmaximummeandiscrepancyhalves.test.ts`
verifies (one of 22 new tests covers `x -> k*x` invariance under
median-heuristic bandwidth rescaling). It is the test that most
sharply distinguishes the two CF-space axes.

## 3. The live-smoke numbers, sorted by `mmdT` desc

From the v0.6.366 CHANGELOG live-smoke block, on local pew
queue.jsonl at 2026-05-03, 6 sources, 12,021,747,864 total
tokens, 1 source dropped below `min-tenure-days=14`,
sort=`mmdTDesc`:

| source | tenure | n1 | n2 | sigma | mmd2_V | mmd2_U | mmdT | mmdZ |
|--------|-------:|---:|---:|------:|-------:|-------:|-----:|-----:|
| claude-code | 72 | 36 | 36 | 7,479,010.44 | 0.267911 | 0.241520 | 4.8224 | 0.000000 |
| openclaw | 17 | 8 | 9 | 55,747,289.15 | 0.614173 | 0.499000 | 2.6012 | 0.000000 |
| hermes | 17 | 8 | 9 | 7,095,970.46 | 0.233561 | 0.099048 | 0.9892 | 0.000000 |
| opencode | 14 | 7 | 7 | 120,062,654.98 | 0.187637 | 0.026109 | 0.6567 | 0.000000 |
| vscode-other | 265 | 132 | 133 | 27,024.44 | 0.002984 | 0.001237 | 0.1977 | 0.000002 |

A few things jump out.

**Headline.** `claude-code` posts `mmdT = 4.8224`, the largest
scaled MMD in the corpus, despite a near-zero `mmdZ`. The
near-zero `mmdZ` is purely a units artefact — `mmdZ = sqrt(mmd2_V)
/ pooledMad`, and the pooled MAD is in raw token units (millions
to tens of millions), while `sqrt(mmd2_V)` is bounded by
`sqrt(2)`. So `mmdZ` becomes a per-token-MAD effect size that
naturally reads near-zero on a Gaussian-bounded numerator. The
ratio is informative across sources but not on an absolute
"sigma" scale — readers used to axis-119 `adZS` or axis-117
`stZ` should not transfer their threshold intuition.

**Raw vs scaled.** `openclaw` has the largest raw `mmd2_V =
0.614`, indicating that roughly 62% of the Gaussian-bandpass-
weighted CF gap mass between the two halves is captured. That
is the most pronounced full-distribution shift in the corpus on
the MMD axis. But `mmdT` weights `mmd2_V` by
`n1 * n2 / (n1 + n2)`, and openclaw's `n1 = 8, n2 = 9` gives
prefactor `4.24`, vs claude-code's `n1 = n2 = 36` giving `18.0`.
That 4.2x prefactor advantage swings the scaled headline to
claude-code despite a 0.27 vs 0.61 raw gap. This is the same
phenomenon visible on axis-122 (`enT` vs `enE`) and is a clean
example of "scaled headline rank reflects how confidently we
can reject the null at this sample size, not how much the
distributions differ".

**Bandwidth scale.** `sigma` ranges from `27,024.44` for
vscode-other up to `120,062,654.98` for opencode. That is a
~4,400x spread in characteristic scale, exactly tracking the
spread in pooled token-count magnitudes across sources. The
median-heuristic adaptivity is doing its job — the test is being
run at a scale appropriate to each source rather than at a single
hand-chosen bandwidth that would make some sources look flat and
others look like delta functions.

**Single-source residual.** vscode-other has the smallest scaled
statistic (0.1977) despite the largest sample (`n1 = 132,
n2 = 133`). The CF gap there is genuinely small at the bandwidth
the median heuristic picks. This is consistent with vscode-other
having a long quasi-stationary regime over its 265-day tenure,
which is exactly what the dispatcher has been treating as the
"background source" in cross-axis comparisons across the entire
118-122 cluster.

## 4. Cross-axis comparison with the 118-122 cluster

On the 5-source cluster the live-smoke columns from CHANGELOG
v0.6.358-0.6.366 give (claude-code first, openclaw second,
vscode-other on its own):

| Axis | claude-code | openclaw | vscode-other |
|------|-------------|----------|--------------|
| 118 KS | ksZ=+3.9206 | ksZ=-2.4504 | (n/a) |
| 119 AD | adA2=415.93 / adP=1.04e-216 | adA2=76.62 / adP=9.01e-44 | adA2=173.13 / adP=5.33e-89 |
| 120 CvM | cvmStat=1.5502 / cvmP=1.07e-04 | cvmStat=0.9861 / cvmP=2.55e-03 | cvmStat=0.4259 / cvmP=6.20e-02 |
| 121 W1 | wassZ=+0.58 | wassZ=+3.44 | wassZ=+0.13 |
| 122 EnD | enT=568,185,342 | enT=626,191,940 | enT=16,886.34 |
| 123 MMD | mmdT=4.8224 | mmdT=2.6012 | mmdT=0.1977 |

Three distinct rankings.

- **claude-code-led** axes: 118 (KS), 119 (AD A2), 123 (MMD-T).
  These are the ECDF-sup-norm, ECDF-tail-L2, and RKHS-mean-
  embedding tests. They all amplify the same underlying signal:
  claude-code has the longest tenure (n1 = n2 = 36), so the
  prefactor advantage dominates whenever the gap function lives
  in a "fixed-budget" function space.
- **openclaw-led** axes: 121 (W1), 122 (EnD raw `enE` and `enT`),
  123 (MMD raw `mmd2_V`). These are quantile-space and CF-space
  tests in their unscaled forms, where openclaw's much larger
  median absolute change (a collapse from 213.8M to 59.4M tokens
  across halves on EnD) dominates over the claude-code prefactor.
- **vscode-other** is the only source where axis-119 AD posts
  `adA2 = 173.13` (`adP = 5.33e-89`) — a giant ECDF-tail-L2
  rejection — but axes 122/123 register essentially nothing
  (`enT = 16,886.34`, `mmdT = 0.1977`). That cross-axis split is
  a textbook demonstration of weighting differences. AD weights
  the CDF gap by `1 / (H * (1 - H))`, which blows up at the
  tails. vscode-other has heavy-tailed daily-total distributions
  (long history with a few extreme spikes), so the AD weight
  finds and amplifies tail mismatches that the Gaussian band-pass
  of MMD attenuates and that the `1/t^2` kernel of EnD also
  attenuates. This is exactly the case the Sriperumbudur et al.
  2010 framing predicts — same characteristic-kernel-class test,
  different finite-sample power, dominated by where the gap
  lives in frequency space.

The vscode-other case is the most useful single witness for why
the family was extended to axis-123. If the 118-122 cluster
agreed everywhere on every source, axis-123 would just be a
seventh restatement of the same thing. The fact that AD says
"reject hard" and MMD/EnD say "barely reject" on vscode-other
means the family is genuinely covering distinct functional
spaces, and the next axis after 123 should be designed against
*the gap* between MMD and EnD on heavy-tail sources rather than
against any single one of the existing six.

## 5. What axis-123 is going to disagree with, and when those disagreements matter

A few falsifiable predictions to anchor what we should look for
in the next 5-10 ticks of CHANGELOG releases.

**P-123-A.** On any source whose two halves differ primarily in
their multi-modal mode separation (e.g., a source that goes
from unimodal to bimodal across halves), axis-123 will reject
more strongly than axis-122 will, as a function of mode
separation in units of `sigma`. The crossover point should be
near `mode_separation / sigma ≈ 1.5`, where the band-pass
weighting picks up the inter-mode CF mass that the `1/t^2`
weighting smears out.

**P-123-B.** On any source whose two halves differ primarily in
a single low-frequency location shift (median moves but shape
is preserved), axis-122 (EnD) will reject more strongly than
axis-123 (MMD-Gauss), because the `1/t^2` weight puts mass at
the low-frequency CF features that location shifts dominate.
On the live-smoke table, openclaw's `enE` collapse from 213.8M
to 59.4M is exactly such a location shift, and the openclaw
`enT/mmdT` ratio of 626e6/2.6 ≈ 240e6 is much larger than
claude-code's 568e6/4.82 ≈ 118e6, which is consistent with this
prediction. (Caveat: the units are not directly comparable
across the two test statistics, but the *ratio of ratios* is
informative.)

**P-123-C.** vscode-other will continue to be the source that
splits axes 119 (AD) and 123 (MMD-Gauss) most sharply — AD
rejecting hard, MMD rejecting weakly — for as long as the
empirical distribution of daily totals has a heavy tail relative
to the median. Once vscode-other's tail thins out (or the
dispatcher rotates to a younger window that excludes the early
heavy-tail period), the MMD-vs-AD gap on this source should
narrow.

**P-123-D.** On every source, `mmd2_U` will be smaller than
`mmd2_V` (the live-smoke table already confirms this — claude-
code 0.241520 vs 0.267911, openclaw 0.499000 vs 0.614173,
hermes 0.099048 vs 0.233561, opencode 0.026109 vs 0.187637,
vscode-other 0.001237 vs 0.002984). The V-statistic includes
the diagonal `k(A_i, A_i) = 1` and `k(B_j, B_j) = 1` terms,
which biases it upward by `1/n1 + 1/n2`. The U-statistic is
unbiased and is the more conservative inference. The fact that
the V/U ratio varies across sources (1.11 for claude-code,
2.36 for hermes, 7.19 for opencode) is itself a per-source
signal — it tracks how far below the diagonal-included
"saturation" each source's CF gap actually is. opencode's
`mmd2_V / mmd2_U = 7.19` flags that almost all of opencode's
biased V-statistic is the diagonal-term bias; the actual
between-half gap is small.

**P-123-E.** When axis-124 is added (whatever it ends up being),
the most informative axis in the post-123 cluster will be
whichever one most sharply disagrees with axis-123 on
vscode-other and on opencode simultaneously. That is, whichever
axis maximizes the *cross-source disagreement matrix entropy*
relative to the existing 6-axis cluster. A useful candidate is
a Hilbert-Schmidt Independence Criterion (HSIC) variant on the
two halves treated as paired-time-series, which would live in
the same RKHS as MMD but measure dependence rather than
distribution equality — orthogonal to the entire 118-123
family.

## 6. Implementation note: median heuristic and what its 22 tests verify

The CHANGELOG mentions 22 new tests in
`test/dailytokenmaximummeandiscrepancyhalves.test.ts`, bringing
the total test count to 10,705 across 216 test files (up from
10,679 after axis-122 in v0.6.365). Reading off the CHANGELOG
list, the test coverage is:

- input validation
- half-split sizes
- exact match against a brute-force MMD V-statistic and
  U-statistic reference (8- and 30-sample series)
- the canonical `T = n1 * n2 / (n1 + n2) * mmd2_V` identity
- location-invariance
- scale-invariance under `x -> k*x` via median-heuristic
  bandwidth rescaling (the structural-difference test vs
  axis-122)
- half-swap symmetry
- sign convention
- identical-halves degeneracy
- random-trial bounds (`mmd2_V in [0, 2]` for Gaussian kernel
  V-statistic)
- shape-difference detection at equal medians
- sigma positivity
- source filtering
- sort orderings
- top cap
- determinism
- bandpass behaviour on wide vs narrow shift fixtures

The two tests that pull most of the structural weight are
"scale-invariance under `x -> k*x`" (the test that distinguishes
axis-123 from axis-122) and "bandpass behaviour on wide vs
narrow shift fixtures" (the test that distinguishes axis-123
from any non-band-pass MMD variant). The first one is the
formal claim. The second is the finite-sample claim. Together
they pin down the axis as a specific element of the MMD class,
not just "any MMD".

## 7. So what does this give us, beyond a sixth axis?

Three concrete things.

First, a per-source disagreement matrix across six axes is now
populated. The next post in this strand should compute the
6x6 (axis x source) Spearman rank-correlation matrix and report
which pairs of axes are most and least correlated across the
five-to-six-source corpus. The prediction from the structural
table is that 119 (AD) and 123 (MMD) will be the least
correlated pair, and that 118 (KS) and 120 (CvM) will be the
most correlated pair.

Second, the cross-source-comparable effect-size column `mmdZ`
needs more work before it can be used as a standalone screening
metric. The current normalization by pooled MAD makes `mmdZ`
read near-zero by construction, which is informative as a
relative ordering but not as an absolute threshold. A natural
alternative is to normalize by `sqrt(2)` (the V-statistic
ceiling) so that `mmdZ` lands in `[0, 1]` and becomes a
fraction-of-saturation effect size. That would make it directly
readable against axis-118 `ksZ` and axis-117 `stZ` in the
existing dashboards.

Third, axis-123 closes the "characteristic-kernel completion"
of the two-sample family. There is no further axis we can add
in CF space with the same Gaussian kernel that is not a monotone
image of axis-123, because the mean embedding is injective on
the probability-measure space (Sriperumbudur et al. 2010,
Theorem 23). The next genuine extension has to either change
the kernel class (e.g., to Laplacian, Matern, or a deep kernel
learned on the queue.jsonl history itself) or move to a different
function space (e.g., HSIC for dependence rather than equality,
or a time-indexed Wasserstein distance for the gap-filled series
treated as a stochastic process). The axis-124 design choice is
therefore not "another two-sample test" but "a different
question entirely". That is a useful pivot to flag now, before
the next CHANGELOG release lands and the choice gets made by
default.
