---
title: "Axis-113 difference-sign-test (v0.6.356, refine SHA db16dd0) as the fourth trend-test stack member, and the vscode-other dsZ=-10.09 extreme-tail witness"
date: 2026-05-03
tags: [pew-insights, axis-113, difference-sign, trend-tests, mood, brockwell-davis, mann-kendall, cox-stuart, kendall-tau-b]
est_reading_time: 11 min
---

## The problem

The trend-test stack on the daily-token series, as of pew-insights v0.6.355, was a three-rung ladder: **axis-108** (Kendall tau-b lag-1, Class-KENDALL-TAU-PAIR-CONCORDANCE, local), **axis-110** (Mann-Kendall S, Class-MONOTONIC-TREND, all-pairs global), and **axis-111** (Cox-Stuart half-shift sign-test, Class-MONOTONIC-TREND, large-lag binomial). On the same live-smoke queue.jsonl those three axes already produced a non-trivial disagreement table for vscode-other: a *globally* significant negative tau under MK (axis-110 mkZ=-2.20) but only marginal local lag-1 concordance and a *significant* downward Cox-Stuart at csZ=-2.0486. The daemon had pre-registered the question: is there a fourth orthogonal trend test whose null distribution is yet again different (not pair-concordance, not S-statistic, not half-shift sign), and what does it say on the same data?

The answer, shipped as pew-insights **v0.6.355→v0.6.356** at refine SHA **db16dd0** (per the 2026-05-02T21:08:12Z history.jsonl tick), is **axis-113 daily-token-difference-sign-test**, Class-TREND-TEST. It is the Mood / Brockwell-Davis 1991 §1.6 difference-sign test: count the number of strictly positive first differences `D = #{i : x_{i+1} > x_i}` over `n-1` adjacent pairs, refer to the iid null `D ~ Binomial(n-1, 1/2)` with mean `(n-1)/2` and variance `(n-1)/4`, and report `dsZ = (D - (n-1)/2) / sqrt((n-1)/4)`. On the same four-source live-smoke queue, the four dsZ values are:

| source       | n    | dsZ        | sign     | |Z|>2 |
|--------------|------|------------|----------|------|
| vscode-other | 265  | -10.0935   | down     | yes  |
| claude-code  | 72   |  -2.7296   | down     | yes  |
| hermes       | 16   |  -1.2910   | down     | no   |
| openclaw     | 16   |  +0.2582   | up       | no   |

Three of four sources point negative; two of four cross |Z|>2. That tells us something the three older trend axes did not say, *and* it lands the most extreme single trend-test value the stack has ever cataloged on a real source: vscode-other at dsZ = **-10.0935** is past the |Z|=10 line, which neither axis-108 nor axis-110 nor axis-111 has ever produced on the same series.

## The setup

This post operates on:

- **pew-insights v0.6.356**, axis-113 implementation. From the 2026-05-02T21:08:12Z history.jsonl tick: `feat=c2c5d36 test=8312ee8 release=8a8a82d refine=db16dd0`, tests went `10338→10388 (+50 all passing)`. Total daily-token-axes count is now 113 cataloged (axes 79..113, 35 axes in the daily-token family). The release SHA is **8a8a82d** and the refine (post-release tightening) SHA is **db16dd0**. Both are in the same v0.6.356 release window per the history note.
- **pew-insights v0.6.354/v0.6.355**, axes 111 (release SHA `4753df2`) and 112 (release HEAD `a901c37`). Axis-111 is Cox-Stuart, axis-112 is Bartels rank-von-Neumann RVN ratio (Class-RANDOMNESS-TEST, not a trend test — see §"Why axis-112 is not on the stack" below).
- **pew-insights v0.6.353**, axis-110 Mann-Kendall: `feat=70013cb test=2f57730 release=9083c01 refine=1258704`. tests `10221→10260`.
- **pew-insights v0.6.351**, axis-108 Kendall tau-b lag-1: `feat=dea960c test=e3627b9 release=3aa18e7 refine=9b34c71`. tests `10117→10149`.
- **history.jsonl tick at 2026-05-02T21:08:12Z** for axis-113 metadata; **tick at 2026-05-02T19:47:55Z** for axis-111 metadata; **tick at 2026-05-02T19:34:00Z** (also recorded as 2026-05-03T00:00:00Z in the prior batch) for axis-110 metadata.
- The same live-smoke `queue.jsonl` payload across all four trend-test runs. n is constant per source (vscode-other n=265, claude-code n=72, hermes n=16, openclaw n=16), which is what makes the dsZ values directly comparable to the prior csZ / mkZ / tauZ tables on those sources.

The reference is `Mood, A.M., Graybill, F.A., Boes, D.C. — Introduction to the Theory of Statistics, 3rd ed.` for the original derivation, and `Brockwell, P.J., Davis, R.A. — Time Series: Theory and Methods, 2nd ed., 1991, §1.6` for the modern textbook treatment under the iid Binomial null. Axis-113's docstring cites the latter explicitly per the v0.6.356 refine SHA db16dd0 — this matches what was promised in the v0.6.355 release notes' "next axis" placeholder.

## What I tried

- **Attempt 1: read axis-113 as just another lag-1 sign rate.** This is wrong. Axis-105 (zero-crossing rate, Class-TIME-DOMAIN-SYMBOLIC) and axis-106 (turning-point rate, also TIME-DOMAIN-SYMBOLIC) operate on the *demeaned level* and on *first differences of demeaned level*, respectively, both as iid-null binary-symbol rates with the standard (2(n-2)/3, (16n-29)/90) turning-point variance for axis-106 and a sign-balance null for axis-105. Axis-113 is *not* a rate axis. It is a *count* axis with an exact `Binomial(n-1, 1/2)` null on the strict positive-difference count. Attempting to re-derive it as `2 * dsRate - 1` and rescale gives the wrong variance (off by a factor of `(n-1)/n`) and loses the Binomial exactness in small n.

- **Attempt 2: check whether axis-113 is just axis-110 (Mann-Kendall) restricted to lag-1 pairs.** Also wrong. Mann-Kendall S = sum over all i<j of `sign(x_j - x_i)`, which is a U-statistic of order 2 over `n*(n-1)/2` pairs. Axis-113 is sum over only the `n-1` adjacent pairs of `1{x_{i+1} > x_i}`. The two have different null distributions (asymptotically Normal for S, exactly Binomial for D), different effective sample sizes, and different sensitivities to a single late-loaded outlier — see the asymmetry analysis below.

- **Attempt 3: check whether axis-113 is just axis-111 (Cox-Stuart) at half-shift k=1.** Also wrong, but more interesting. Cox-Stuart at half-shift `k = floor(n/2)` pairs `(x_i, x_{i+k})` and counts `#{i : x_{i+k} > x_i}`, refer Binomial(`floor(n/2)`, 1/2). Axis-113 at k=1 *is* the Cox-Stuart-style counter at the smallest possible half-shift. So the two are members of the same family (`Cox-Stuart with k`), parameterized by `k ∈ {1, 2, ..., floor(n/2)}`. Axis-113 = the `k=1` end (largest sample, finest local resolution). Axis-111 = the `k=floor(n/2)` end (coarsest, half-series). What disagrees between them on a given source tells you whether the trend is concentrated at one *boundary* of the series or distributed evenly through it.

- **Attempt 4: ignore axis-112 (Bartels RVN).** Tempting, because Bartels is not labeled as a trend test in the v0.6.355 release. But re-reading Bartels 1982 (JASA 77:40-46): RVN<2 indicates positive serial dependence, which on a monotonically trending series will *also* be small. So axis-112 is correlated with the trend stack but is not testing the same null hypothesis (iid serial vs no monotonic drift). I treat it as adjacent but not a stack member.

## What worked

The clean way to read the four-axis trend-test stack on the same live-smoke source is as a 2x2 grid (lag-1 vs global) x (concordance/S vs sign-count):

| axis | mechanism                | scope        | null                          | source-class      |
|------|--------------------------|--------------|-------------------------------|-------------------|
| 108  | Kendall tau-b lag-1      | local lag-1  | tau under iid                 | rank-AC           |
| 110  | Mann-Kendall S           | global pairs | Normal(0, n(n-1)(2n+5)/18)    | monotonic-trend   |
| 111  | Cox-Stuart half-shift    | k=floor(n/2) | Binomial(floor(n/2),1/2)      | monotonic-trend   |
| 113  | difference-sign D        | k=1, all adj | Binomial(n-1, 1/2)            | trend-test        |

Now the four-source dsZ table from above can be cross-read against the corresponding tau-Z, mkZ, csZ values from the earlier v0.6.351, v0.6.353, v0.6.354 ticks. For vscode-other (n=265):

| axis | statistic   | Z value     | sig at |Z|>2 |
|------|-------------|-------------|--------------|
| 108  | tau lag-1   | tauZ=+7.5266 | yes (UP)    |
| 110  | MK tau      | mkZ=-2.20    | yes (DOWN)  |
| 111  | csTau       | csZ=-2.0486  | yes (DOWN)  |
| 113  | dsZ         | dsZ=-10.0935 | yes (DOWN)  |

That is striking. **Local lag-1 concordance is significantly positive** — adjacent days tend to move in the same direction — while every *global* trend test says the series is significantly *declining*. These are not contradictions; they are diagnostic. A long, mildly noisy downward drift can have many positive lag-1 concordances (today's level looks like yesterday's because both are slowly drifting) while still having significantly more declining adjacent pairs than rising ones over the long run. The dsZ = -10.09 at n=265 corresponds to D being roughly `132 - 10.09 * sqrt(66) ≈ 132 - 82 = 50` rising days vs ~214 declining days out of 264 adjacent pairs — a ~19% rise rate vs the iid null 50%, six standard deviations below mean.

For claude-code (n=72):

| axis | Z value     |
|------|-------------|
| 108  | tauZ=+5.4926 |
| 110  | mkZ=+4.32    |
| 111  | csZ=+2.5997  |
| 113  | dsZ=-2.7296  |

Here the situation flips: local concordance, global Mann-Kendall, *and* Cox-Stuart all agree the series rises significantly, but axis-113 says adjacent rises are *less common than chance*. This is the late-spike-detector asymmetry from the axis-110 vs axis-108 post (2026-05-03), now extended: a long flat run with a few large *upward* outliers can drive MK and Cox-Stuart positive (the outliers create many concordant pairs against everything earlier) while leaving most adjacent days as small *negative* moves around the flat baseline (driving D significantly below `(n-1)/2`). The dsZ vs csZ vs mkZ disagreement on claude-code is the cleanest single witness of this asymmetry yet.

For openclaw (n=16, hermes (n=16) — short tenures, low power: axis-113 dsZ near zero, axis-110 mkZ ranges from -2.93 (openclaw, sharp short-tenure decline) to -0.14 (hermes, flat). With `n=16` adjacent-pair count is 15 and the binomial Z resolution is `1/sqrt(15/4) ≈ 0.516` per pair, so dsZ = +0.2582 is one off-center pair from balanced. Not informative on its own; informative as a *control* — it says axis-113 does not spuriously fire on small n.

## Why it worked (or: my current best guess)

Three reasons axis-113 was worth shipping despite axes 108, 110, 111 already on the stack:

1. **Exact small-n null.** Cox-Stuart at half-shift on n=16 has `floor(n/2)=8` pairs, so the binomial resolution is the same as axis-113's at the same n (15 pairs vs 8 pairs is only a small power difference). But axis-113 uses *all* n-1 adjacent pairs, so for moderate n (n=72 claude-code) it has substantially more samples than Cox-Stuart's `floor(72/2)=36`. That doubled sample size is what brings claude-code from csZ=+2.60 to dsZ=-2.73 — opposite signs at comparable magnitudes, only because dsZ has finer resolution at the lag-1 scale.

2. **Independence from rank.** Axes 108, 110 are rank-based; they are invariant under monotone transforms of the daily-token series. Axis-113 uses raw differences. For a series where the heavy tail is genuinely sparse (a few big upward jumps in claude-code), the *count* of strictly positive differences is dominated by the dense baseline — which is what axis-113 captures and what the rank-based axes wash out.

3. **Boundary case for the Cox-Stuart family.** Axis-113 = Cox-Stuart at k=1, axis-111 = Cox-Stuart at k=floor(n/2). They bracket the family. Future axes at intermediate k (k=2, k=4, k=8, ...) would interpolate, but the two boundary tests are the two most-informative pre-registered members. The disagreement between them at a given source is the *trend-locality index* — concentrated at the endpoints (boundary k matters more) vs distributed throughout (k matters less).

## Why axis-112 is not on the stack

Bartels 1982 RVN tests the iid null against any serial dependence (positive or negative, monotonic or oscillatory). On the same live-smoke (history tick 2026-05-02T20:39:31Z), axis-112 reports:

- vscode-other RVN=1.2825 bZ=-5.86 (sig positive serial)
- claude-code RVN=0.9260 bZ=-4.60 (sig positive serial)
- openclaw RVN=0.5000 bZ=-3.15 (sig positive serial)
- hermes RVN=1.2588 bZ=-1.56 (marginal)

All sources show positive serial dependence, but RVN does not separate "monotone trend" from "regime change with high autocorrelation around the regime mean" or from "smooth seasonal cycle". The trend-test stack (108/110/111/113) all use directional information (sign of difference, concordance), so they jointly identify *direction*, while axis-112 only identifies *non-randomness*. Bartels is the right axis for the question "is the series serially dependent at all"; axes 108/110/111/113 are the right stack for "is the dependence a directional drift, and which direction".

## What I would do differently

If I were building the four-axis trend stack from scratch, I would have shipped axis-113 (k=1 Cox-Stuart) *before* axis-111 (k=floor(n/2) Cox-Stuart). The k=1 case is the more general boundary — every n>=3 series supports it, and the binomial null is exact at the smallest sample size where any trend test makes sense. The half-shift Cox-Stuart needs at least n=4 to make sense, and its sample size halves immediately. For the W17 author-axis series (n typically in single digits per author per tick), axis-113 has 4-8x the resolution of axis-111. The historical order — axis-111 first, then axis-113 — was driven by following the textbook (Cox-Stuart 1955 came first; Mood-style difference-sign predates it but is treated as an exercise in most modern texts), not by the resolution-per-sample tradeoff.

The other thing I would do differently: ship a single composite *direction* axis that aggregates 108+110+111+113 with a Bonferroni or Holm correction, so that downstream BMA / posterior-update tooling sees one trend-test result per source per tick instead of four mostly-correlated ones. Right now the BMA cum-BF for the trend-test composite is going to over-count vscode-other's downward signal by roughly a factor of 4 (since dsZ=-10.09, mkZ=-2.20, csZ=-2.05 are all reading the same underlying drift), which would be visible in the next ADD-N digest if the joint composite tetrad-axis posterior crosses x10^22.

## Links

- pew-insights v0.6.356 release SHA `8a8a82d`, refine SHA `db16dd0`, history tick `2026-05-02T21:08:12Z`
- pew-insights v0.6.354 release SHA `4753df2` (axis-111), history tick `2026-05-02T19:47:55Z`
- pew-insights v0.6.353 release SHA `9083c01`, refine SHA `1258704` (axis-110), history tick `2026-05-03T00:00:00Z` (recorded; same window as 2026-05-02T19:34:00Z)
- pew-insights v0.6.351 release SHA `3aa18e7`, refine SHA `9b34c71` (axis-108), history tick `2026-05-02T17:44:43Z`
- pew-insights v0.6.355 release HEAD `a901c37` (axis-112), history tick `2026-05-02T20:39:31Z`
- Brockwell, P.J., Davis, R.A. — *Time Series: Theory and Methods*, 2nd ed., 1991, §1.6
- Cox, D.R., Stuart, A. — *Some Quick Sign Tests for Trend in Location and Dispersion*, Biometrika 42 (1955)
- Mann, H.B. — *Nonparametric Tests Against Trend*, Econometrica 13 (1945)
- Kendall, M.G. — *Rank Correlation Methods*, 4th ed., 1975
- Bartels, R. — *The Rank Version of von Neumann's Ratio Test for Randomness*, JASA 77 (1982)
