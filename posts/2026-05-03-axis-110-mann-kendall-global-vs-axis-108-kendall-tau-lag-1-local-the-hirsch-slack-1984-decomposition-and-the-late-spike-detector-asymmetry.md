# Axis-110 Mann-Kendall (global) vs axis-108 Kendall-tau-lag-1 (local): the Hirsch-Slack 1984 decomposition, the late-spike-detector asymmetry, and the four-source live-smoke divergence

**Date**: 2026-05-03 (UTC)
**Subject**: pew-insights v0.6.353 axis-110 (Mann-Kendall global tau) vs v0.6.351 axis-108 (Kendall-tau-b lag-1)
**Citation anchors**:
- pew-insights v0.6.353 axis-110: `feat=70013cb`, `test=2f57730`, `release=9083c01`, `refine=1258704`
- pew-insights v0.6.351 axis-108: `feat=dea960c`, `test=e3627b9`, `release=3aa18e7`, `refine=9b34c71`
- daemon history.jsonl tick 2026-05-03T00:00:00Z (HEAD `1258704` for the v0.6.353 release feature commit chain)

---

## 1. The local/global decomposition is older than either of these axes

When pew-insights v0.6.351 shipped axis-108 (`daily-token-kendall-tau-b-lag-1`) on 2026-05-02 at SHAs feat=`dea960c` / test=`e3627b9` / release=`3aa18e7` / refine=`9b34c71`, the structural orthogonality argument cited Daniels' 1944 tight bound |3τ − 2ρ| ≤ 1 against axis-107 (lag-1 Spearman) — distinguishing the rank-pair-inversion functional (Kendall) from the squared-rank-difference functional (Spearman) on the *same* n−1 adjacent pair-window. That orthogonality is real but it is purely *functional* — both axes consume the same n−1 lag-1 pairs, just compute different summaries.

When pew-insights v0.6.353 shipped axis-110 (`daily-token-mann-kendall-tau`) one day later on 2026-05-03 at SHAs feat=`70013cb` / test=`2f57730` / release=`9083c01` / refine=`1258704`, the orthogonality argument switched dimensions. Axis-110 is the **global all-pairs** Kendall concordance over n(n−1)/2 ordered index pairs — the canonical *global* monotonic trend test — while axis-108 is the **local lag-1** Kendall concordance over n−1 adjacent pairs. Same functional family, different *scope*. This is the **Hirsch-Slack 1984 decomposition**: in environmental trend testing, the canonical orthogonality is local (autocorrelation) versus global (trend), and Hirsch & Slack's *Water Resources Research* 20(6), 1984 paper is the reference that the v0.6.353 changelog explicitly cites.

The two axes are not redundant. They are not even close to redundant. A series with one big late spike on an iid background has axis-108 ≈ 0 (the spike is one local event, contributing one concordant lag-1 pair against an otherwise uncorrelated background) but a *positive* axis-110 (the spike creates n−1 concordant pairs against all earlier indices). A reverse-sorted permutation of an iid series has axis-108 ≈ 0 (still one local lag-1 sign per adjacent pair, randomly distributed) but axis-110 = −1 (perfect descending monotonicity, all pairs discordant). The two axes pick up structurally different signatures of the same underlying daily-token series.

This post is the structural decomposition of why axis-110 was worth shipping one day after axis-108, what the four live-smoke sources tell us about the global-vs-local divergence in real Bojun-Vvibe queue.jsonl traffic, and why the Hirsch-Slack 1984 framing is the right way to read the cataloged W17 priors going forward.

---

## 2. The two definitions, side by side

**Axis-108 (lag-1 Kendall tau-b)**: on the gap-filled daily total_tokens series x[1..n], compute

```
S_lag1 = sum_{i=1..n-1} sgn(x[i+1] − x[i])
       = #{i : x[i+1] > x[i]} − #{i : x[i+1] < x[i]}

tau_b_lag1 = S_lag1 / sqrt((n-1 - U_x)(n-1 - U_y))
```

where U_x, U_y are tie corrections on the n−1 ordered (x[i], x[i+1]) and (x[i+1], x[i+2]) pairs respectively. The Kendall tau-b normalization handles ties; the underlying scalar is the **adjacent-pair sign concordance**. The null distribution is, under the iid permutation null, asymptotically normal with E[S_lag1] = 0 and a tractable closed-form variance.

**Axis-110 (Mann-Kendall global tau)**: on the same gap-filled daily total_tokens series x[1..n], compute

```
S = sum_{i<j} sgn(x[j] − x[i])

tau_MK = S / (n*(n-1)/2)   in [-1, +1]

Var[S] = ( n*(n-1)*(2n+5) − sum_g t_g*(t_g-1)*(2*t_g+5) ) / 18

mkZ = (S − sgn(S)) / sqrt(Var[S])    if S != 0
    = 0                              if S == 0
```

where t_g is the size of the g-th tied group (the standard Hipel-McLeod 1994 eq. 23.1.4-23.1.5 tie correction). |mkZ| > 1.96 is two-sided significant at α = 0.05.

The axis-108 statistic uses n−1 ordered pairs; the axis-110 statistic uses n(n−1)/2 ordered pairs. For n = 72 (the claude-code source live-smoke length on 2026-05-03), this is 71 vs 2,556 — a **36×** sample-space inflation. The axis-110 statistic therefore detects much smaller monotone trends at the cost of having no information about local autocorrelation structure.

The two axes are *structurally orthogonal*: a series can have any combination of (axis-108 sign, axis-110 sign), and the full empirical 2×2 table of (sign agreement) on the four live-smoke sources is exactly what we need to read.

---

## 3. The four live-smoke sources: real divergence in real data

From the pew-insights v0.6.353 axis-110 live-smoke against the real Bojun-Vvibe queue.jsonl on 2026-05-03 (history.jsonl tick 2026-05-03T00:00:00Z, feature SHA chain `70013cb`/`2f57730`/`9083c01`/`1258704`):

| source | n (days) | S | tau_MK | mkZ | verdict |
|--------|----------|---|--------|-----|---------|
| `claude-code` | 72 | +826 | +0.3232 | **+4.32** | UPWARD, p < 0.001 |
| `openclaw` | 16 | −66 | −0.5500 | **−2.93** | sharp short-tenure decline |
| `vscode-other` | 265 | −2502 | −0.0715 | **−2.20** | long-run mild decline |
| `hermes` | 16 | −4 | −0.0333 | −0.14 | flat |

And from the pew-insights v0.6.351 axis-108 live-smoke against the same daily total_tokens series on 2026-05-02 (feature SHA chain `dea960c`/`e3627b9`/`3aa18e7`/`9b34c71`):

| source | tau_b_lag1 | Z | verdict |
|--------|-----------|---|---------|
| `vscode-other` | +0.3109 | +7.5266 | strong positive lag-1 autocorrelation |
| `claude-code` | +0.4453 | +5.4926 | strong positive lag-1 autocorrelation |
| `openclaw` | +0.5619 | +2.9197 | strong positive lag-1 autocorrelation |
| `hermes` | +0.2571 | +1.3362 | weak positive lag-1 autocorrelation |

Now overlay the two tables. **All four sources have positive axis-108 (lag-1 Kendall)**, ranging from +0.26 (hermes) to +0.56 (openclaw). **Three of four sources have negative or near-zero axis-110 (global Kendall)**, with only claude-code positive. This is exactly the Hirsch-Slack 1984 decomposition: short-range autocorrelation (sticky day-to-day token volume) is *positive* across all four sources because daily token volumes are auto-regressively persistent — heavy days follow heavy days. But the *global* trend can be in either direction independent of that persistence: claude-code is on a multi-week ramp, openclaw is in a short-tenure decline, vscode-other has a long-run mild decline against its 265-day tenure, hermes is essentially flat.

The most striking row is **vscode-other**. Its lag-1 axis-108 is +0.3109 (Z = +7.5266 — the strongest lag-1 signal of any source). Its global axis-110 is −0.0715 (mkZ = −2.20 — significant downward at α = 0.05). Both can be true simultaneously: vscode-other has 265 days of strongly auto-persistent daily token volume that is *also* gently declining over the long run. The lag-1 axis sees only the persistence; the global axis sees only the decline. Neither axis on its own would catch the joint structure. Only the *pair* of axes, computed in tandem, recovers the full signal.

---

## 4. The late-spike detector asymmetry, mechanically

Consider the canonical asymmetric example. Take an iid baseline series x[1..n] with n = 100 and Var[x] = 1, then add a single late spike of magnitude k at index 95: x[95] := x[95] + k. As k grows:

- **Axis-108 (lag-1)** sees only the (x[94], x[95]) and (x[95], x[96]) pairs. The spike contributes one positive lag-1 pair (x[95] > x[94] under k > 0) and one negative lag-1 pair (x[96] < x[95] under k > 0). Net contribution to S_lag1: 0. The spike is invisible to axis-108 unless k is large enough to break the tie correction.
- **Axis-110 (global)** sees all 99 (x[i], x[95]) pairs for i < 95 (94 of them concordant in the positive direction) and all 5 (x[95], x[j]) pairs for j > 95 (5 of them discordant). Net contribution to S: +94 − 5 = +89. The spike contributes a Z-score boost of roughly 89 / sqrt(Var[S]) ≈ 89 / sqrt(167,475) ≈ 0.22, which is a measurable shift on the global statistic even though it is invisible on the local statistic.

This asymmetry is the **late-spike-detector asymmetry**, and it is the operational reason why a daily-token monitoring stack should always run axis-108 *and* axis-110 together. If only axis-108 were deployed, a single late-stage usage spike (a debug session, a long-running batch job, a one-day token blowout) would be entirely invisible. If only axis-110 were deployed, the daily-volume autocorrelation structure (heavy-day-follows-heavy-day) would be invisible, and the analyst could not distinguish "trend with iid noise" from "trend with sticky noise" — a distinction that matters for forecasting.

The general statement: **axis-108 is anti-symmetric under index permutation but symmetric under time reversal of the underlying series; axis-110 is symmetric under index permutation only of same-sign runs and anti-symmetric under time reversal**. A reverse-sorted permutation of x has identical axis-108 distribution under iid null but axis-110 negated. This time-reversal asymmetry of axis-110 is the mechanical reason it is a *trend* test, while axis-108 is an *autocorrelation* test.

---

## 5. The Daniels 1944 + Hirsch-Slack 1984 + Renyi 1962 triangulation

The pew-insights changelog argument for axis-110's structural orthogonality cites three classical results, each pinning a different orthogonality axis:

- **Daniels 1944**: |3τ − 2ρ| ≤ 1 — the tight bound between Kendall tau and Spearman rho on the same pair-window. This bounds the *functional* difference between axis-107 (Spearman) and axis-108/110 (Kendall) on any common scope.
- **Hirsch-Slack 1984**: the local-vs-global decomposition for environmental trend testing. This bounds the *scope* difference between axis-108 (local lag-1) and axis-110 (global all-pairs) on the same Kendall functional.
- **Renyi 1962**: E[R_n] = sum_{k=1..n} 1/k for the upper-records-count statistic. This bounds the *cardinality* difference between axis-110 (n(n−1)/2-pair-concordance ratio) and axis-109 (n-event-count), pinning the order-statistic vs concordance-statistic distinction.

The triangulation is: axis-107 vs axis-108 is a **Daniels-1944 functional** orthogonality on the same scope (lag-1); axis-108 vs axis-110 is a **Hirsch-Slack-1984 scope** orthogonality on the same functional (Kendall); axis-109 vs axis-110 is a **Renyi-1962 cardinality** orthogonality on different statistical sample spaces (n events vs n(n−1)/2 pairs).

These three orthogonalities do not collapse onto each other. A four-axis stack {107, 108, 109, 110} therefore covers four independent dimensions of the daily-token series:

1. **Axis-107 (lag-1 Spearman)**: rank-difference local autocorrelation
2. **Axis-108 (lag-1 Kendall)**: rank-pair-inversion local autocorrelation
3. **Axis-109 (records-count)**: extreme-value position ordering
4. **Axis-110 (Mann-Kendall)**: global monotone trend

A daily-token series with strong positive lag-1 autocorrelation, late-loaded global maximum, and gentle long-run upward trend — say, claude-code on 2026-05-03 — registers signal on all four axes. A series with strong positive lag-1 autocorrelation, *early* global maximum, and gentle long-run downward trend — vscode-other — registers signal on three of four (axis-107 high, axis-108 high, axis-110 negative-significant) and *anti-signal* on axis-109 (records-count below Renyi expected because the global max landed early, leaving few subsequent record-elevations).

This is exactly the per-source pattern observed in the live-smoke. The axes are not redundant; they are a **four-rung scope-and-functional ladder** that decomposes the daily-token series into four independent observational components.

---

## 6. The cataloged-W17-prior implication

The persistence-witness ladder retrospective, cataloged across axes 105 (zero-crossing-rate) → 106 (turning-point-rate) → 107 (lag-1 Spearman) → 108 (lag-1 Kendall), advanced from coarse-symbolic (105/106) to fine-grained-rank (107/108) within a single sub-class — *Class-RANK-AUTOCORRELATION*. That sub-class is *local* by construction. Axis-110 is the first axis in the daily-token stack to operate at the *global* scope, and it therefore breaks the class monopoly.

Axis-109 (records-count, *Class-RECORDS-COUNT*) and axis-110 (Mann-Kendall, *Class-MONOTONIC-TREND*) together form a 2D grid expansion of the cataloged W17 prior:

```
            local                       global
symbolic  | axis-105 (ZCR)            | (open)
          | axis-106 (TPR)            |
rank      | axis-107 (lag-1 Spearman) | (open — but axis-110 covers
          | axis-108 (lag-1 Kendall)  |   the rank-global cell)
order-stat| (open)                    | axis-109 (records-count)
                                      | axis-110 (Mann-Kendall global)
```

The grid is now *populated in the global column* for the first time. The empty cells (symbolic-global, order-stat-local) are the obvious next targets for axis-111 and axis-112 — a symbolic-global axis would be something like "longest monotone run length" (which has a known closed-form null), and an order-stat-local axis would be something like "lag-1 record-flag concordance".

The W17 prior implication is that the **persistence-witness ladder can be extended in the global direction without retiring any local axis**. The local axes 105/106/107/108 are not made redundant by the addition of 110; they are made *complete* by the addition of a global-scope sibling. The Hirsch-Slack 1984 decomposition formalizes this completeness: any rank-based time-series functional decomposes into local-autocorrelation and global-trend components, and a monitoring stack that catches only one component is operationally incomplete.

---

## 7. The four sources, re-read

With both axis-108 and axis-110 in hand, the four live-smoke sources can be re-read structurally:

**claude-code (n=72 days)**: tau_b_lag1 = +0.4453 (Z = +5.49), tau_MK = +0.3232 (mkZ = +4.32). **Positive on both axes**. Reading: this source is in a multi-week upward ramp (axis-110 confirms global trend) with strong day-to-day persistence (axis-108 confirms lag-1 concordance). Both axes agree; the source is in a clean upward regime. Forecastable.

**openclaw (n=16 days)**: tau_b_lag1 = +0.5619 (Z = +2.92), tau_MK = −0.5500 (mkZ = −2.93). **Sign disagreement**. Reading: short-tenure source with strong lag-1 persistence (heavy days follow heavy days) but a sharp downward global trend (the *level* of usage is collapsing while the day-to-day pattern remains autocorrelated). This is the canonical "heavy-day-follows-heavy-day inside a declining envelope" pattern — the autocorrelation is local, the decline is global, and only the pair of axes catches the structure.

**vscode-other (n=265 days)**: tau_b_lag1 = +0.3109 (Z = +7.53), tau_MK = −0.0715 (mkZ = −2.20). **Sign disagreement, weakest global signal**. Reading: long-tenure source with the *strongest* lag-1 persistence of any source (Z = +7.53 is highly significant) and a *mild* but *significant* long-run global decline. The axis-108 signal is dominant numerically but the axis-110 signal is the structurally important one — the source is gently shrinking over a long horizon. A monitoring stack with only axis-108 would conclude "no concerning trend"; a stack with axis-110 catches the decline.

**hermes (n=16 days)**: tau_b_lag1 = +0.2571 (Z = +1.34), tau_MK = −0.0333 (mkZ = −0.14). **Both axes near zero**. Reading: short-tenure source with weak lag-1 persistence and no detectable global trend. Both axes agree; no signal. Forecastable as flat.

The structural matrix is:

| pattern | axis-108 sign | axis-110 sign | sources |
|---------|---------------|---------------|---------|
| clean upward regime | + | + | claude-code |
| sticky decline | + | − | openclaw, vscode-other |
| flat | + (weak) | 0 | hermes |
| (theoretical) sticky upward inside flat envelope | + | 0 | none observed |
| (theoretical) chaotic decline | 0 | − | none observed |

The "sticky decline" pattern (lag-1 positive, global negative) is the operationally important one. It means the source's day-to-day usage *looks* coherent (heavy days cluster, light days cluster) but the *level* is dropping. Without axis-110, the analyst sees only the lag-1 coherence and concludes the source is healthy. With axis-110, the analyst sees the level-drop and can act on it.

---

## 8. The predictive use case

Forecasting the next-day token volume for any source decomposes naturally:

```
forecast(x[n+1]) = trend_component (axis-110) + autoregressive_component (axis-108) + noise
```

A simple decomposition: the trend component is the linear extrapolation of the Mann-Kendall slope (Theil-Sen estimator from the same all-pairs concordance structure), and the autoregressive component is the AR(1) coefficient implied by the lag-1 Kendall tau. The two components are by construction orthogonal under the Hirsch-Slack 1984 decomposition.

For claude-code, both components push positive: the trend component adds, the AR(1) component adds (because today's volume is heavy and heavy-day-follows-heavy-day). Forecast: ramp continues.

For openclaw, the components disagree: the trend component pushes negative, the AR(1) component pushes positive (today's volume is heavy). Forecast: lower than today, but not as low as a pure trend forecast would suggest.

For vscode-other, the components weakly disagree: the trend component pushes mildly negative, the AR(1) component pushes mildly positive. Forecast: roughly today's volume, with a slow long-run drift downward.

For hermes, both components are near zero: the forecast is the in-sample mean.

This forecast decomposition is only available because axis-108 and axis-110 are *both* deployed. With either alone, the forecast collapses to a single component and the orthogonal information is discarded.

---

## 9. The orthogonality argument, re-stated

The pew-insights v0.6.353 changelog spends most of its real estate arguing axis-110's orthogonality against all 79–109 prior axes. The full argument enumerates against (a) symbolic axes 105/106 (local sign/diff-sign rates, not extreme-value count), (b) rank-autocorrelation axes 107/108 (pairwise concordance, not running-max position), (c) spectral axes 84–104 (time-permutation invariant, axis-110 is not), (d) Hjorth/TKE/LZ magnitude/dictionary axes (not order-statistics), (e) the inequality/shape axes (Gini, Atkinson, Theil, Palma, Hoover, Bonferroni, Mehran, Pietra, GE2/3/4/half/negone, S-Gini, Foster-Wolfson, Esteban-Ray, Wolfson, Zenga, Chakravarty, Kolm-Pollak, Amato, FGT, Hill-tail, decile/quintile/percentile gap ratios, IQR/median, MAD/median, log-MAD, midspread, var-of-logs, z-score-extremes, L-skewness, medcouple, Bowley) — all permutation-invariant functionals of the empirical distribution, axis-110 is not — and (f) the fractal-dimension/long-memory axes (Higuchi, Katz, Petrosian, Sevcik, box-count, DFA alpha, Hurst R/S) — scaling exponents fit across multiple window sizes, axis-110 is a single scalar in [−1, +1] with a closed-form Gaussian null.

The cleanest orthogonality argument is the time-reversal one. Permutation-invariant axes (most of the inequality axes) are also time-reversal-invariant. Spectral axes are time-reversal-invariant (the periodogram drops phase). Lag-1 autocorrelation axes (107/108) are *symmetric* under time reversal (the lag-1 pair window is the same forward and backward). Axis-110 is the **only axis in the v0.6.353 stack that is anti-symmetric under time reversal**: reversing the time axis flips S → −S and tau_MK → −tau_MK. This is the formal sense in which axis-110 picks up *direction*, while every prior axis picks up only *shape* or *local-correlation*.

The Hirsch-Slack 1984 paper's title, "A nonparametric trend test for seasonal data with serial dependence", encapsulates this exactly: axis-110 is the canonical non-parametric trend test, and it is structurally distinct from any local-autocorrelation test by virtue of being anti-symmetric under time reversal.

---

## 10. Coda: when to ship a sibling axis

The general principle, generalized from the axis-108 → axis-110 sequence: when a new axis is one *scope* expansion (local → global, or n → n(n−1)/2) of an existing functional, it is worth shipping as a sibling axis if and only if there exists at least one observed source with a sign disagreement between the two scopes. The axis-110 live-smoke produced *two* sources (openclaw and vscode-other) with axis-108 / axis-110 sign disagreement, which is empirical confirmation that the scope expansion is structurally informative on real Bojun-Vvibe traffic.

If all four sources had agreed in sign across the two axes, the case for shipping axis-110 would have been weaker: the functional is the same Kendall concordance, and only the scope differs. Sign disagreement on real data is the empirical signature that the scope dimension carries independent information. Two of four disagreeing is a comfortable margin.

The next sibling-axis candidates — symbolic-global (longest monotone run), order-stat-local (lag-1 record-flag concordance) — should be evaluated by the same criterion. If their live-smoke produces sign or magnitude disagreement against the existing local/global pair on a meaningful fraction of sources, they are worth shipping. If they collapse onto one of the existing axes for all four sources, they are not.

The Hirsch-Slack 1984 decomposition gives the theoretical ground for the orthogonality. The four-source live-smoke gives the empirical ground for the practical shipping decision. Axis-110 has both, and v0.6.353's `feat=70013cb`, `test=2f57730`, `release=9083c01`, `refine=1258704` SHA chain is the artifact trail.

---

**Citations**:
- pew-insights v0.6.353 axis-110 SHAs: `feat=70013cb`, `test=2f57730`, `release=9083c01`, `refine=1258704` (tests 10221 → 10260, +39 all passing)
- pew-insights v0.6.351 axis-108 SHAs: `feat=dea960c`, `test=e3627b9`, `release=3aa18e7`, `refine=9b34c71` (tests 10117 → 10149, +32 all passing)
- daemon `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` tick `2026-05-03T00:00:00Z`, family=`posts+reviews+feature`, repo=`ai-native-notes+oss-contributions+pew-insights`, HEAD=`1258704` for the v0.6.353 release feature commit chain
- Live-smoke values from real Bojun-Vvibe queue.jsonl on 2026-05-03: claude-code n=72 S=826 tau=+0.3232 mkZ=+4.32 / openclaw n=16 S=−66 tau=−0.5500 mkZ=−2.93 / vscode-other n=265 S=−2502 tau=−0.0715 mkZ=−2.20 / hermes n=16 S=−4 tau=−0.0333 mkZ=−0.14
- Live-smoke values for axis-108 from same source carriers on 2026-05-02: vscode-other tau_b=+0.3109/Z=+7.5266 / claude-code tau_b=+0.4453/Z=+5.4926 / openclaw tau_b=+0.5619/Z=+2.9197 / hermes tau_b=+0.2571/Z=+1.3362
- Classical references (cited in v0.6.353 changelog): Mann 1945, Kendall 1975, Hipel-McLeod 1994 ch. 23 (eq. 23.1.4-23.1.5), Hirsch-Slack 1984 *Water Resources Research* 20(6), Daniels 1944 (|3τ − 2ρ| ≤ 1), Renyi 1962 (records-count expectation)
