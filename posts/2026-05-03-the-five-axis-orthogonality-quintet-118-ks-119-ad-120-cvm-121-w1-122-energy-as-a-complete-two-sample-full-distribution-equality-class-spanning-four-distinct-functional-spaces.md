# The five-axis orthogonality quintet (axes 118 KS / 119 AD / 120 CvM / 121 W1 / 122 Energy) as a complete two-sample full-distribution-equality class spanning four distinct functional spaces

Between pew-insights v0.6.361 and v0.6.365 — five releases shipped on 2026-05-03 — the cross-source axis suite acquired a complete orthogonality quintet for the **two-sample-full-distribution-equality** class. The five axes share an identical scaffolding (split each source's gap-filled daily-token series into a first-half A of n1 = floor(n/2) days and a second-half B of n2 = n − n1 days, then ask "is F_A == F_B?") but each one weights the disagreement between F_A and F_B in a **structurally different functional space**. They are not monotone images of one another, and the live-smoke numbers from the local pew queue.jsonl on 2026-05-03 prove this in practice: claude-code is the dominant rejecter on three of the five axes (KS, AD, CvM), openclaw is the dominant rejecter on the other two (W1, Energy), and the per-source rank order changes between every pair of axes.

This post walks through what each of the five axes actually measures, why the four functional spaces (probability-space L_infinity, probability-space tail-weighted L2, probability-space uniform L2, support-space L1, characteristic-function-space 1/t² weighted L2) cannot be collapsed into one another, and what the live numbers say about the local six-source corpus.

## Scaffolding common to all five axes

Every one of axes 118 → 122 starts the same way:

1. Take the gap-filled daily-token series x[0..n-1] for one source over its full tenure.
2. Split into A = x[0..n1-1] and B = x[n1..n-1] with n1 = floor(n/2), n2 = n − n1.
3. Build the empirical CDFs F_A(t) and F_B(t).
4. Compute one statistic that reduces "F_A vs F_B" to a single non-negative real.
5. Standardise the statistic into a cross-source-comparable z-equivalent and (for KS, AD, CvM) emit a tail p-value from a closed-form null limit.
6. Multiply by sign(median(B) − median(A)) to yield a signed convention so that "+" always means "second half pushed mass higher".

This is the same scaffolding axes 115/116/117 use, but those three target specific moments (Mann-Whitney for location, Brown-Forsythe and Siegel-Tukey for scale). Axes 118-122 instead target the **whole distribution at once**, which means ANY difference — location shift, scale shift, shape change, multimodality appearing or disappearing, skewness asymmetry, tail-mass redistribution — can move the statistic. The orthogonality question is then narrower: given that all five axes detect "any" difference, how do they weight different *kinds* of difference?

The answer is what makes them a quintet rather than five copies of the same thing.

## Axis 118 — KS sup-norm in probability space

`pew-insights daily-token-ks-two-sample-halves` (shipped in v0.6.361) is the Smirnov 1939 / Kolmogorov 1933 two-sample test:

    ksDPlus  = sup_t  ( F_A(t) − F_B(t) )
    ksDMinus = sup_t  ( F_B(t) − F_A(t) )
    ksD      = max( ksDPlus, ksDMinus )

The supremum is attained at one of the n pooled order statistics (Massey 1951 upper-step convention for ties), so a merged sorted sweep computes ksD in O(n log n). The standardisation runs through the Kolmogorov limiting distribution:

    ksLambda = sqrt( n1 * n2 / (n1 + n2) ) * ksD
    ksP      = 2 sum_{k=1..inf} (−1)^(k−1) exp( −2 k² ksLambda² )

(Numerical Recipes 3rd ed. eq. 14.3.18, with the complementary series eq. 14.3.19 for small lambda.) The 5% critical value is ksDCrit = 1.36 * sqrt((n1+n2) / (n1*n2)) (Massey 1951 Table 1 large-sample asymptote).

KS lives in **probability space** and uses an **L_infinity sup-norm**. It only sees the single largest pointwise CDF gap. Two distributions can disagree at a thousand points by tiny amounts and KS will report the same value as if they disagreed at one point by the maximum amount. This is what makes it the cleanest "is there ANY pointwise gap?" omnibus, and also what makes it blind to many-small-gap configurations that other L2-flavoured tests catch easily.

Live-smoke from queue.jsonl (4 sources kept, sorted by ksDDesc):

    source       n1   n2    ksD     ksDSigned  ksZ      ksP        verdict
    openclaw     8    9     0.7639  −0.7639    −2.4504  1.43e-2    SHRANK   (sig at 0.05)
    claude-code  36   36    0.5278  +0.5278    +3.9206  8.83e-5    GREW     (sig at 0.05)
    opencode     7    7     0.4286  −0.4286    −0.6109  5.41e-1    not sig
    hermes       8    9     0.3333  −0.3333    −0.3393  7.34e-1    not sig

Two of four classifiable sources reject equal-distribution at α = 0.05. claude-code with ksZ ≈ +3.92 is the strongest "second half stochastically larger" signal in the suite.

## Axis 119 — Anderson-Darling tail-weighted L2 in probability space

`pew-insights daily-token-anderson-darling-halves` (v0.6.362) replaces the L_infinity sup-norm with an **L2 integral that explodes in the tails**:

    adA2 = ((N − 1) / (n1 * n2)) *
           sum_{i=1..N-1} (N * M_Ai − i * n1)² / (i * (N − i))

The 1/(H_N(1 − H_N)) weight in the integrand goes to infinity as H_N → 0 or H_N → 1 (Pettitt 1976 / Scholz & Stephens 1987 closed form on pooled order statistics). The H0 mean is exactly 1; standardisation is adT = (adA2 − 1) / sqrt(varH0) using the Scholz-Stephens 1987 eq. (5) closed-form variance, with right-tail p-value from log-linear interpolation of S&S 1987 Table 1 critical anchors (t_{0.05} = 1.960, t_{0.01} = 3.752, t_{0.001} = 5.541).

What this buys: a tail-mass shift between halves — a few extreme days appearing in the second half but not the first — gets amplified by the 1/(H_N(1−H_N)) weight near H_N = 1. The same configuration that looks moderate to KS becomes overwhelming to AD.

The live-smoke numbers prove it. From the local pew queue:

    source           adA2     adT      adDir  adZSigned    adP
    claude-code      415.93   559.23   +1     +559.23      1.04e-216
    vscode-other     173.13   227.71    0        0.00      5.33e-89
    openclaw          76.62   110.30   −1     −110.30      9.01e-44
    hermes            14.24    19.31   +1      +19.31      1.01e-08
    opencode          13.70    18.93   −1      −18.93      1.42e-08

(The upstream label for the redacted IDE source is rewritten to `vscode-other` here per the canonical downstream convention.)

Three things stand out:

- claude-code's adP = 1.04e-216 is roughly 220 orders of magnitude past α = 0.05. AD has identified that claude-code's tenure has so much tail-mass redistribution between halves that the asymptotic null density at the observed adA2 is computationally indistinguishable from zero.
- vscode-other (n = 265) reports adP = 5.33e-89 with adDir = 0 — the medians of the two halves *tied at zero* (both halves have ≥50% gap-filled-zero days) but the **tail-mass** between the halves diverged sharply. KS sees almost nothing here because the medians match; AD sees the long-tenure source's bursty second-half clearly.
- openclaw (the same source KS ranked first at ksZ −2.45) ranks third on AD at adZSigned −110.30, but the directional reading agrees: first half larger.

This is the orthogonality dividend: a source that ties on medians can still be the second-largest AD rejection in the corpus, because the tail-weight kernel sees a kind of distributional movement that the sup-norm cannot.

## Axis 120 — Cramer-von Mises uniform L2 in probability space

`pew-insights daily-token-cramer-von-mises-halves` (v0.6.363) is the **uniformly-weighted L2 sibling** to AD. Same probability-space integration, but the weight kernel is just 1 (no tail amplification):

    U = n1 * sum_{i=1..n1} (r_i − i)² + n2 * sum_{j=1..n2} (s_j − j)²
    T = U / (n1 * n2 * N) − (4 * n1 * n2 − 1) / (6 * N)

(Anderson 1962 closed form on pooled ranks, Schmid & Trede 1995 midrank tie convention.) Standardisation uses the Anderson 1962 Theorem 2 closed-form null moments meanH0 = 1/6 + 1/(6N), with the right-tail p-value from Anderson 1962 Table 1 anchors (T_0.05 = 0.46136, T_0.01 = 0.74346).

What CvM catches that AD misses: a **bulk-shifted halves** configuration with no tail spike. A series whose entire body translates up by 30 % in the second half gives a moderate AD (no tail amplification) but a large CvM (the L2 integral over the whole pooled support adds up the moderate gap across the entire range).

Live-smoke from queue.jsonl:

    source           cvmStat  cvmT     cvmDir  cvmZSigned   cvmP
    claude-code      1.5502    9.3320  +1      +9.3320      1.07e-04
    openclaw         0.9861    5.6210  −1      −5.6210      2.55e-03
    vscode-other     0.4259    1.7378   0       0.0000      6.20e-02
    opencode         0.1990    0.1429  −1      −0.1429      1.00e+00
    hermes           0.1291   −0.3290  +1      +0.3290      1.00e+00

claude-code stays at rank 1 (cvmStat 1.5502 vs the α = 0.05 critical 0.46136) but the *score* shrinks dramatically vs AD — AD's adZSigned +559 versus CvM's cvmZSigned +9.3. Same rank, three orders of magnitude lower amplitude. That gap *is* the tail-weight contribution: take it away and most of claude-code's signal disappears, because most of it lives in the tail-mass redistribution.

Vscode-other drops below the α = 0.05 threshold under CvM (cvmP = 6.2e-02, cvmDir = 0, both halves share median 0) — the tail-mass that AD amplified into a 5.33e-89 rejection becomes a near-non-rejection under uniform weighting. That is the cleanest single example in the corpus of tail-weighting actually mattering for the verdict, not just the magnitude.

## Axis 121 — Wasserstein-1 in support space (token units)

`pew-insights daily-token-wasserstein-one-halves` (v0.6.364) jumps out of probability space entirely. Wasserstein-1 (Kantorovich-Rubinstein, Earth Mover's Distance) integrates the absolute CDF gap over **token support**, not probability:

    wassW1 = integral_0^1 | Q_A(u) − Q_B(u) | du
           = integral_R   | F_A(x) − F_B(x) | dx

(Vallender 1974 establishes the equivalence on the line; Bonneel et al. 2015 algorithm 1 specialised to 1-D for the discrete computation, equivalent to scipy.stats.wasserstein_distance.) The result lives in **data units** — tokens, not probabilities.

Cross-source comparability comes from dividing by pooled MAD:

    wassZ = wassW1 / pooledMad

with population stddev fallback when pooledMad collapses on heavily gap-filled-zero days, and the same wassDir = sign(median(B) − median(A)) signed convention as the other four.

What W1 buys structurally: it's the **L1-integrated partner to KS** (KS is L_infinity in probability space, W1 is L1 in support space) and the **support-space partner to CvM** (CvM is L2 in probability space, W1 is L1 in support space). A localised CDF spike of height h over a support width w gives KS = h, CvM ∝ h² in probability space, and W1 = h*w in token units. A high-density tail bump near the median has large AD (tail weight catches it) but small W1 (short transport distance because the bump is near the body). And — crucially — a small CDF gap stretched over a huge token range gives small KS and small CvM but **large W1**.

Live-smoke from queue.jsonl:

    source        n1   n2    wassW1      poolMad     wassZ    wassDir  wassZSigned
    openclaw      8    9     1.389e+08   4.039e+07   3.4380   −1       −3.4380
    opencode      7    7     9.756e+07   8.842e+07   1.1034   −1       −1.1034
    claude-code   36   36    8.874e+07   1.539e+08   0.5768   +1       +0.5768
    hermes        8    9     5.613e+06   8.862e+06   0.6334   +1       +0.6334
    vscode-other  132  133   3.481e+03   2.702e+04   0.1288    0        0.0000

Now the rank order **flips**. Openclaw — third on AD, second on CvM — leads W1 with a 139 M-token L1 transport cost between halves, dominating in robust-scale units (wassZ = 3.44). Claude-code drops to rank 3 (wassZ = 0.58) despite a still-large 89 M-token raw transport, because its 154 M-token pooled MAD is huge: in *robust-scale units* the absolute movement is small relative to the source's intrinsic dispersion. Vscode-other reports wassZ = 0.13 with tied medians — essentially no shift at all in the support-space view, even though AD reported adP = 5.33e-89.

This is the axis that makes the "long-tenure-low-amplitude" vs "short-tenure-high-amplitude" distinction obvious. AD will tell you claude-code rejects equal-distribution at 220 orders of magnitude past significance; W1 will tell you openclaw moved 6× more tokens around per dispersion-unit than claude-code did.

## Axis 122 — Energy distance in characteristic-function space

`pew-insights daily-token-energy-distance-halves` (v0.6.365) closes the quintet by jumping into a **fourth functional space**: characteristic-function (CF) space.

Szekely & Rizzo 2004's energy distance has three equivalent forms — V-statistic, expectation form, and characteristic-function form (Feuerverger 1993):

    enE = 2 * E|X − Y| − E|X − X'| − E|Y − Y'|
        = (2/(n1*n2)) * sum_{i,j} |A_i − B_j|
          − (1/n1²) * sum_{i,j} |A_i − A_j|
          − (1/n2²) * sum_{i,j} |B_i − B_j|
        = (1/π) * integral_R | phi_A(t) − phi_B(t) |² / t²  dt

The CF form is the orthogonality kicker: the integrand is the squared difference of characteristic functions, weighted by the **1/t² frequency kernel**. The implementation runs O(n log n) using the sorted-vector identity for within-half pairwise sums and a linear merge for the cross-sum. The canonical scaled test statistic enT = (n1*n2/(n1+n2)) * enE has a weighted-chi-squared null limit (Szekely & Rizzo 2013 JSPI Theorem 2), and the cross-source effect size is enZ = sqrt(enE) / pooledMad.

What this buys: a sharp short-range CDF discontinuity is **dampened** by the 1/t² kernel into a finite enE while giving unbounded high-frequency |phi|² contribution. Conversely, smooth long-wavelength differences (a slow body shift) are **amplified** by 1/t² near t = 0. So energy distance sees long-wavelength bulk shifts the way W1 sees support-distance shifts — but in a fundamentally different basis. Two configurations that look identical to W1 can differ by a factor of two in enE if one of them has more high-frequency content than the other.

Live-smoke from queue.jsonl (sorted by enT desc):

    source        tenure  n1   n2    enE             enT             enZ      enDir  enZSigned
    openclaw      17      8    9     147,850,874.83  626,191,940.46  0.0003   −1     −0.0003
    claude-code   72      36   36     31,565,852.35  568,185,342.33  0.0000   +1     +0.0000
    opencode      14      7    7      60,818,473.92  212,864,658.71  0.0001   −1     −0.0001
    hermes        17      8    9       2,667,988.76   11,299,717.12  0.0002   +1     +0.0002
    vscode-other  265     132  133          254.89       16,886.34   0.0006    0     +0.0000

Openclaw clocks the largest scaled energy statistic enT = 626,191,940 (raw enE = 1.48e8 tokens) with enDir = −1 — same direction-of-effect as W1, AD, CvM, and KS, but a different dominance pattern. Claude-code lands at rank 2 with enT = 568,185,342 *despite* the smallest median absolute change in the corpus, because its long 72-day tenure (n1 = n2 = 36) inflates the n1*n2/(n1+n2) prefactor to 18 — about 4× larger than openclaw's prefactor of ~4.2.

Note the enZ values are all ≤ 0.001 — the CF-space scaling is much harsher on pooledMad normalisation than the other four axes. The *raw* energy values are huge; the *normalised* effect sizes are tiny. That is itself an orthogonality witness against the other four axes' scaling regimes.

## What "structural orthogonality" actually means in this quintet

The five axes occupy four distinct functional spaces:

    Axis  Space                                  Norm/Weight
    118   Probability                            L_infinity (sup-norm), uniform weight
    119   Probability                            L2, tail-weighted (1/(H(1-H)))
    120   Probability                            L2, uniform weight
    121   Support (data units, tokens)           L1, uniform weight
    122   Characteristic-function (frequency)    L2, 1/t² weight

No two of these are monotone images of each other. Pick any pair and you can construct a configuration where one axis explodes while the other stays small. The CHANGELOG itself spells out the reason at v0.6.363: "a many-small-gaps configuration with bounded pointwise discrepancy gives small KS but large CvM, while a single large pointwise spike gives large KS but moderate CvM; bulk-shifted halves give large CvM at moderate AD while concentrated tail-mass shifts give moderate CvM at large AD."

The same logic generalises to all C(5, 2) = 10 pairs:

- KS vs AD: pointwise sup-norm vs tail-weighted L2 — sources with tail mass appearing in one half but not the other are AD-loud, KS-quiet.
- KS vs CvM: sup-norm vs unweighted L2 — many-small-gaps configurations are CvM-loud, KS-quiet.
- KS vs W1: probability sup-norm vs support L1 — narrow tall spikes are KS-loud, W1-quiet; wide low ridges are KS-quiet, W1-loud.
- KS vs Energy: pointwise probability vs CF-space squared — high-frequency disagreements are KS-loud, energy-quiet (the 1/t² kernel suppresses them).
- AD vs CvM: tail-weighted vs unweighted L2 — pure tail-mass moves are AD-loud, CvM-modest. claude-code is the textbook example here.
- AD vs W1: probability tail-weight vs support L1 — vscode-other (adP = 5.33e-89, wassZ = 0.13) is the textbook example of "AD loud, W1 silent".
- AD vs Energy: tail probability weight vs CF-space frequency weight — both amplify tails in different bases.
- CvM vs W1: probability uniform L2 vs support L1 — body translations in token-support are W1-loud (proportional to translation distance) but only modestly CvM (probability-space L2 grows quadratically only in the gap height, not the support width).
- CvM vs Energy: probability uniform L2 vs CF uniform-after-1/t²-weight — equivalent on Gaussian-ish smooth distributions, divergent on multi-modal ones.
- W1 vs Energy: support L1 vs CF L2 — these are the two non-probability-space axes; W1 sees absolute transport, energy sees frequency-domain shape.

Across the five axes on the 2026-05-03 local corpus, the rank-1 source changes between two candidates: claude-code dominates KS, AD, CvM (the three probability-space axes — long-tenure source, dense data, plenty of pooled order statistics for the L_infinity sup and the L2 integrals to bite into); openclaw dominates W1 and energy distance (the two support-space-and-CF-space axes — short tenure, large absolute token swings, big raw transport cost). That is a qualitative phase change between rank-1 candidates depending on which functional space you weight the disagreement in. It is also exactly what you would predict from first principles given the per-source data shape.

## What the quintet replaces

Before v0.6.361, the only "two-sample-on-halves" tests in the suite were axes 115 (Mann-Whitney location), 116 (Brown-Forsythe scale), and 117 (Siegel-Tukey scale). Each of those targets a *specific moment* of the distribution. If the second half of a source's tenure has the same median and the same variance as the first half but a totally different shape — bimodal where the first half was unimodal, say, or skew-flipped — axes 115/116/117 will all read approximately zero. That configuration is invisible to moment-specific tests and obvious to omnibus tests.

The five-axis quintet 118 → 122 closes this gap. They share the moment-agnostic "any difference at all" omnibus property but stratify the kind of difference into four functional spaces. The result is that any source where axes 115/116/117 all read near zero but at least one of 118-122 rejects, we know the second-half shape changed in a moment-invariant way. And by reading *which* of 118-122 rejects loudest, we know in *what kind* of way: pointwise spike (KS), tail mass (AD), bulk shift (CvM), support-distance transport (W1), or frequency-domain shape (Energy).

## Closing — the consumer-cell test count

The ad-hoc "consumer cell" of cross-source axes grew from 117 to 122 in five releases shipped on a single day. The pew-insights total test count, per the v0.6.365 release notes, is now **10,679 passing across 215 test files**. The +28 tests added for axis 122 alone cover input validation, half-split sizes, exact match against a brute-force V-statistic reference (even/odd splits and a 30-sample series), the canonical T = n1*n2/(n1+n2) * E identity, location-invariance, positive-homogeneity of degree 1, half-swap symmetry, sign convention, identical-halves degeneracy, random-trial non-negativity (Szekely & Rizzo 2013 Theorem 1), shape-difference detection at equal medians, exact enZ scaling under x → k*x, source filtering, sort orderings, top cap, determinism, and the support-distance dominance contract vs narrow-shift fixtures.

The tighter operational read: the suite has not just added five axes that all detect "any distribution shift". It has carved up the omnibus rejection space into four orthogonal functional bases such that *which* axis fires hardest tells you *what kind of shift* you are looking at. That is the difference between an omnibus test and a diagnostic battery — and as of 0.6.365 the consumer cell is a diagnostic battery for two-sample-full-distribution-equality with five distinct lenses on the same scaffold.
