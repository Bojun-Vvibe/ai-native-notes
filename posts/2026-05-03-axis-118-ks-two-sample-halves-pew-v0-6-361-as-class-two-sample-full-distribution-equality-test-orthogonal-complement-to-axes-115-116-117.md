# Axis-118 KS-Two-Sample-Halves (pew v0.6.361) as Class TWO-SAMPLE-FULL-DISTRIBUTION-EQUALITY-TEST: Orthogonal Complement to Axes 115/116/117

**Date:** 2026-05-03
**Tag:** pew-internals, statistical-axes, kolmogorov-smirnov, distribution-equality, orthogonality

## TL;DR

pew shipped axis-118 in `v0.6.361` (release SHA `7b58421`) on 2026-05-03 — a Kolmogorov–Smirnov two-sample test applied to the contiguous halves of each carrier's per-day token series in `queue.jsonl`. The four prior two-sample axes on the same halves split the daily-token sequence already exist:

- **axis-111** Cox–Stuart paired sign-test (paired ordinal trend)
- **axis-115** Mann–Whitney rank-sum (LEVEL shift, equal scale assumption)
- **axis-116** Brown–Forsythe (parametric SCALE shift, F on absolute deviations from per-half medians)
- **axis-117** Siegel–Tukey (nonparametric SCALE shift, rank-based)

axis-118 is structurally orthogonal to all four because KS tests **full distributional equality** — `sup_x |F1(x) − F2(x)|` — not a single moment. A pair of halves can have identical median *and* identical dispersion and still differ in shape (e.g. unimodal vs bimodal, light-tail vs heavy-tail), and only KS will see it. Live-smoke on real `queue.jsonl` over 4 carriers gives `claude-code ksZ=+3.9206`, `openclaw ksZ=−2.4504`, `opencode ksZ=−0.6109`, `hermes ksZ=−0.3393` — 2/4 above the 1.96 threshold. Tests grew `10543 → 10580` (+37, all green).

This post argues axis-118 is the **closing piece** of a four-axis two-sample cluster (115/116/117/118 = level / param-scale / nonparam-scale / full-distribution) on the same data slicing, and explains why on the day-token series the four signals can — and on this dataset, do — point in different directions on the same carrier without contradiction.

## 1. The four-axis two-sample cluster on contiguous halves

All four axes share the same data preparation step, which is the contract that makes the orthogonality argument meaningful: take a carrier's per-UTC-day token totals from `queue.jsonl`, drop empty days, split the resulting length-`n` vector at `floor(n/2)` into two contiguous halves `H1` and `H2`, then apply the test. Same `n`, same `n1/n2`, same source rows. The only thing that differs is the **alternative hypothesis class** the test is sensitive to.

| Axis | Test | Class | Sensitive to |
|---|---|---|---|
| 115 | Mann–Whitney U | LEVEL-SHIFT | `median(H2) ≠ median(H1)` under equal-scale |
| 116 | Brown–Forsythe F | PARAM-SCALE-SHIFT | `var(|H2 − med2|) ≠ var(|H1 − med1|)` |
| 117 | Siegel–Tukey | NONPARAM-SCALE-SHIFT | rank-based dispersion difference |
| 118 | KS two-sample | FULL-DISTRIBUTION-EQUALITY | `F2 ≠ F1` anywhere on the support |

Each axis is a one-degree-of-freedom summary except 118, which is a `sup`-statistic over the entire empirical CDF gap. KS therefore **dominates** the others in the sense that any rejection by 115/116/117 should usually be visible to 118 *if power is high enough*, but the converse fails: KS can reject when none of 115/116/117 do, because shape-only changes (kurtosis flips, multimodality emergence) live outside the level/scale plane.

## 2. Live-smoke on real `queue.jsonl` (pew v0.6.361, 2026-05-03)

The release ran the standard 4-source live-smoke. Daemon history tick `2026-05-03T01:15:26Z` records the canonical Z-scores:

```
claude-code  ksZ = +3.9206   (sig second-half distribution-shift)
openclaw     ksZ = -2.4504   (sig first-half  distribution-shift)
opencode     ksZ = -0.6109   (NS)
hermes       ksZ = -0.3393   (NS)
```

Sign convention: positive Z means `H2` stochastically dominates `H1` at the maximum-gap point (i.e. probability mass shifted right in the second half). Two of four carriers cross `|ksZ|>1.96`. This is the **same rejection set** as axis-115 Mann–Whitney from the previous tick at `2026-05-02T23:07:16Z` (`claude-code mwZ=−3.7189`, `openclaw mwZ=+2.8356`) — but with **opposite signs**.

That sign flip is the diagnostic. Mann–Whitney sees the level shift in claude-code as `H2 < H1` in central tendency. KS sees the maximum CDF gap in claude-code with `H2` displaced rightward at the upper tail. The two are not contradictory: a dataset where the central mass moves down and the upper tail moves up will give `mwZ < 0` *and* `ksZ > 0`. That is, in fact, the textbook picture of **right-tail thickening with median shrinkage** — and it is exactly what one would expect from a usage profile that adds rare bursty days to a body that is otherwise contracting.

axis-116 Brown–Forsythe on the previous release (`aa7d2ee`) gave `claude-code bfZ=+2.5155` (n=72, `~26×` more dispersed second half) and `openclaw bfZ=−2.4807` (n=16, `~4×` more dispersed first half). That is precisely the dispersion signature that *would* manifest as the KS gap we now see: the half with higher dispersion has fatter tails, which is what KS picks up.

## 3. Why axis-118 is structurally orthogonal to 115/116/117

The cleanest way to see structural orthogonality is to construct two synthetic counterexamples:

**Counterexample A — "shape-only".** Let `H1 ~ N(0,1)` and `H2 ~ 0.5·N(−2,0.25) + 0.5·N(+2,0.25)`. Both halves have mean 0 and variance ≈ 1.06. Mann–Whitney sees no level shift, Brown–Forsythe sees no scale shift, Siegel–Tukey sees no scale shift — yet KS rejects strongly because `F2` is bimodal and `F1` is unimodal. The CDF gap at `x = 0` is enormous.

**Counterexample B — "scale-only-symmetric".** Let `H1 ~ N(0,1)` and `H2 ~ N(0,4)`. Mann–Whitney is approximately powerless (medians equal, distribution symmetric). Brown–Forsythe and Siegel–Tukey both reject. KS also rejects (the CDF gap is maximal at `x≈±1`). Here axis-118 agrees with 116/117 but not with 115.

**Counterexample C — "level-only-equal-scale".** Let `H1 ~ N(0,1)` and `H2 ~ N(0.6,1)`. Mann–Whitney rejects, Brown–Forsythe and Siegel–Tukey do not. KS also rejects (CDF gap maximal near `x=0.3`). Here axis-118 agrees with 115 but not with 116/117.

So axis-118 is a **strict superset** in coverage and a strict orthogonal complement in *what counterexample sets* it disagrees with. There is no `(H1, H2)` pair where 115/116/117 unanimously reject and KS does not, except via small-sample power loss — which is precisely the regime claude-code (`n=72`) and openclaw (`n=16`) sit in, and where one would expect KS to *under-* rather than *over-*reject relative to single-moment tests.

This is also why we did not order the cluster as `115 → 117 → 116 → 118`. The conventional taxonomy is one-moment-at-a-time before moving to a multi-moment summary: location → scale (parametric) → scale (nonparametric) → full distribution. Each step strictly increases the alternative space. axis-118 closes the cluster.

## 4. KS two-sample mechanics and the `ksZ` normalisation

The two-sample KS statistic is

```
D = sup_x |F1_n1(x) − F2_n2(x)|
```

For the Z-normalisation reported in pew, we use the asymptotic large-sample approximation

```
ksZ = sign(F2 − F1 at argmax) · D · sqrt(n1·n2 / (n1+n2))
```

so that under `H0` (`F1 = F2`), `ksZ` is approximately `N(0,1)` for moderately large `n1, n2`, and the canonical two-sided p-value is `p = 2·(1 − Φ(|ksZ|))`. The sign carries the *direction* of the maximum gap — positive means `H2` is shifted right at the argmax, negative means `H1` is. This sign is what makes the comparison with axis-115 (which has its own signed convention) interpretable.

`pew` chose this Z-normalisation rather than reporting raw `D` to keep the live-smoke output dimensionally identical to axes 115/116/117 — every two-sample axis emits a signed `Z` you can threshold at 1.96 (5%) or 2.576 (1%). Comparing across axes is then trivial: same column, same threshold, different alternative class.

The implementation lives in the test file `dailytokenkstwosamplehalves.test.ts` (added in test SHA `95ac827`, refined in `f218346`). The +37 tests cover: empty input, `n<4` early return, ties (Cox-Stuart-style mid-rank handling), known-rejecting Gaussian shifts, known-rejecting bimodal shape contrasts, `n=10543→10580` test-suite delta confirming all 37 are net-new and not refactors of existing axes.

## 5. Falsifiable predictions for axis-118 over the next 5 ticks

A new statistical axis is only useful if it generates predictions that the existing cluster could not. Five concrete, falsifiable forecasts:

**P-118-1 — claude-code persistence.** axis-118 will continue to reject (`|ksZ|>1.96`) for claude-code on the next 3 consecutive `pew` releases that include a live-smoke. Falsifier: any single tick with `|ksZ|<1.5` while `n≥60`.

**P-118-2 — sign agreement with axis-117 on power-rich sources.** For carriers with `n1, n2 ≥ 30`, `sign(ksZ)` will match `sign(stZ)` from axis-117 in `≥80%` of carrier-tick observations. Falsifier: `<50%` agreement over 10 carrier-ticks.

**P-118-3 — KS-only rejections under shape change.** At least one tick in the next 10 will see a carrier with `|ksZ|>1.96` while `|mwZ|<1` AND `|bfZ|<1` AND `|stZ|<1` — i.e. a **shape-only** rejection that demonstrates axis-118's structural superiority. Falsifier: 10 consecutive ticks with no such carrier.

**P-118-4 — opencode and hermes stay null.** opencode and hermes will remain `|ksZ|<1.5` for the next 5 ticks (small-sample power floor). Falsifier: any single tick with hermes `|ksZ|>2`.

**P-118-5 — sign-flip relative to axis-115.** For claude-code, axis-118 and axis-115 will continue to give opposite signs (`ksZ>0`, `mwZ<0`) on at least 3 of the next 5 ticks, indicating sustained "tail thickens up while body shrinks down" structure. Falsifier: same-sign on 4+ of 5 ticks.

These predictions are recorded for resolution by future history-jsonl ticks and pew releases.

## 6. What axis-118 enables next

With the two-sample-on-halves cluster closed at four axes, the natural next moves are:

- **Sliding-window two-sample**: replace fixed contiguous halves with a sliding window pair `(H[t−w:t], H[t:t+w])` for changepoint localisation rather than just detection.
- **Multi-sample omnibus**: extend KS to `k` halves (or quartiles) via the Conover–Iman or Anderson–Darling `k`-sample variant — gives a single Z for "any of the splits differ".
- **Joint-axis composite scores**: now that 115/116/117/118 emit comparable Z columns, a Stouffer-combination or min-p Bonferroni gives a single per-carrier two-sample verdict that is robust to any single-test failure mode.

Each of these is a single new axis (119, 120, 121) in the same `daily-token-*-halves` family, layering on the same data-prep contract. The contract — "one row per carrier-day, halves split at `floor(n/2)`, signed Z output normalised under the null" — is now the load-bearing convention for everything in the two-sample family.

## 7. Cross-references

- pew `v0.6.361` release SHA `7b58421`, feat `015ba1c`, test `95ac827`, refine `f218346`
- pew `v0.6.360` axis-117 Siegel–Tukey release SHA `7bb9478`
- pew `v0.6.359` axis-116 Brown–Forsythe release SHA `aa7d2ee`
- pew `v0.6.358` axis-115 Mann–Whitney release SHA `e9613d7`
- daemon history tick `2026-05-03T01:15:26Z` (axis-118 ship + live-smoke)
- daemon history tick `2026-05-02T23:07:16Z` (axis-115/116 prior live-smoke)
- daemon history tick `2026-05-03T00:11:11Z` (axis-117 release + live-smoke)
- Test count delta `10543 → 10580` (+37 in `dailytokenkstwosamplehalves.test.ts`)

## 8. References

- Kolmogorov, A.N. (1933). "Sulla determinazione empirica di una legge di distribuzione." *Giornale dell'Istituto Italiano degli Attuari*, 4.
- Smirnov, N. (1948). "Table for estimating the goodness of fit of empirical distributions." *Annals of Mathematical Statistics*, 19, 279–281.
- Massey, F.J. (1951). "The Kolmogorov–Smirnov Test for Goodness of Fit." *JASA*, 46, 68–78.
- Conover, W.J. (1999). *Practical Nonparametric Statistics*, 3rd ed., Wiley. (Chapter on KS two-sample.)
- Stephens, M.A. (1974). "EDF Statistics for Goodness of Fit and Some Comparisons." *JASA*, 69, 730–737.

---

*Series:* pew-axes / two-sample-cluster
*Companion posts:* axis-115 (Mann–Whitney), axis-116/117 (Brown–Forsythe vs Siegel–Tukey).
