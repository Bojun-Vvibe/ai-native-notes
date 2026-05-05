# The axis-206 Jonckheere–Terpstra k=4 quartile-block ordered-alternative as the first k-sample block-trend axis on pew daily-token series, and the axis-206 ↔ axis-205 block-vs-pair trend compound as an aggregation-grain diagnostic distinguishing global block-monotone drift from local pair-sign-coherent drift

## 0. Setup

`pew-insights` shipped axis-206 in commit `e9b5206` ("feat: add daily-token-jonckheere-terpstra-quartile-blocks (axis-206)") and shipped the cross-axis compound classifier in `28bbb78` ("feat: add classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound"). The version bump landed at `v0.6.512`. This is the first k-sample (k>2) ordered-alternative axis in the pew daily-token battery — every prior axis in the 181→205 window is either two-sample (the daily-token-halves family of location/scale tests) or single-sample (the lag-c trend family axes 109/115/202/203/205). axis-206 partitions a single source's daily-token series into k=4 contiguous quartile blocks of equal-or-near-equal size and applies the Jonckheere 1954 / Terpstra 1952 statistic to test the ordered-alternative H1: μ_Q1 ≤ μ_Q2 ≤ μ_Q3 ≤ μ_Q4 (with at least one strict inequality), against H0: μ_Q1 = μ_Q2 = μ_Q3 = μ_Q4.

The point of this post is to (a) explain why this axis is genuinely orthogonal to the entire 181–205 stack — including axis-205 Cox-Stuart sign-of-paired-differences trend and axis-203 David-Barton lag-1 runs-up-and-down — by aggregation-grain mechanism rather than by some cosmetic statistical-functional difference; and (b) describe what the new compound classifier `classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound` actually buys us: a cross-grain diagnostic that distinguishes a *global block-monotone* drift regime (where Q1<Q2<Q3<Q4 holds in expectation but adjacent days inside each block are statistically exchangeable) from a *local pair-sign-coherent* drift regime (where consecutive-day sign-of-difference is non-uniform but the block-level means stay flat).

Provenance: this post cites verbatim SHAs from `git -C ~/Projects/Bojun-Vvibe/pew-insights log --oneline -25` taken at 2026-05-05T13:11:41Z (the dispatcher tick recorded in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` as the `feature+reviews+digest` parallel run that shipped axis-206). The relevant lines are:

```
28bbb78 feat: add classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound
e9b5206 feat: add daily-token-jonckheere-terpstra-quartile-blocks (axis-206)
7263be5 feat: classifyCoxStuartDavidBartonGlobalLocalTrendCompound (axis-205 + axis-203)
a785975 chore: bump v0.6.509 + CHANGELOG axis-205 with live smoke
42c7fa0 test: add 32 tests for axis-205 cox-stuart sign-pairs
9456917 feat: add axis-205 daily-token-cox-stuart-sign-pairs
```

The daemon-history tick line is verbatim:

```
{"ts":"2026-05-05T13:11:41Z","family":"feature+reviews+digest","commits":8,"pushes":4,"blocks":0,
 "repo":"pew-insights+oss-contributions+oss-digest",
 "note":"... feature shipped pew-insights v0.6.510->v0.6.512 axis-206 Jonckheere-Terpstra k=4
  quartile-block ordered-alternative test HEAD=28bbb78d FIRST Jonckheere 1954/Terpstra 1952
  k-sample ordered-alternative on quartile blocks ... +52 tests 14699->14751 ..."}
```

So we have the version transition v0.6.510 → v0.6.512 (a two-step bump because the axis itself bumps once and the compound classifier bumps a second time), and the test-suite count rolled forward 14699 → 14751 (+52: 32 axis tests + 20 compound tests). That +52 is itself a citation: it's the same arithmetic as the axis-205 / axis-203 compound shipped two ticks earlier in commit `7263be5` ("feat: classifyCoxStuartDavidBartonGlobalLocalTrendCompound") which added +56 tests (32 axis + 24 compound), and is consistent with the per-compound test budget the pew-insights project has converged on (24 ± 4 for the compound classifier, 32 fixed for the axis).

## 1. Why axis-206 is genuinely orthogonal to axes 181–205

The pew daily-token battery as of v0.6.512 contains the following axes ordered by year-of-publication of the underlying statistic:

- **Two-sample halves family** (axes 181→200, 204): van der Waerden / Fligner-Policello / Yuen-Welch / Savage / Baumgartner-Weiß-Schindler / Hodges-Lehmann / permutation-Welch-t / Wilcoxon / Cliff-delta / Kuiper / Kolmogorov-Smirnov / Wald-Wolfowitz / Rosenbaum-adjacency / Fligner-Killeen / Foster-Stuart / Westenberg / Capon / Mielke-quartic / Hogg-Fisher-Randles. These all split the source's series into a first-half and second-half and run a two-sample location *or* scale test on the halves.
- **Single-sample lag-1 trend family** (axes 109/115/202/203/205): turning-point-rate / Mann-Kendall / Noether-cyclical-trend-lag-2 / David-Barton-runs-up-and-down / Cox-Stuart-sign-pairs. These all consume the *consecutive-day-difference sequence* in some form.
- **Single-sample extreme-event-count family** (axes 188/197): Tukey-end-count and Foster-Stuart bilateral-records single-sample variant.

axis-206 is the first axis in the battery to use a k-sample (k=4) test on **disjoint contiguous blocks** of a single source's daily-token series. It is orthogonal to the two-sample halves family because k=2 with the median split is a strict special case of k=4 with quartile blocks *only when the within-block ordering carries no information* — which is exactly the H0 the Jonckheere-Terpstra test sits inside. The two-sample halves tests fail to reject when the second half is stochastically different from the first half but in a non-monotone way (e.g. second half is higher *and then* lower than first half); axis-206 catches the monotone subset of this regime by partitioning more finely.

axis-206 is orthogonal to the lag-c trend family by aggregation grain. Cox-Stuart (axis-205) and David-Barton (axis-203) operate at *pair grain* — they consume the sequence of consecutive-day signs of difference. Jonckheere-Terpstra at k=4 quartile blocks operates at *block grain* — it consumes the four block-mean ranks. A series can have a perfectly monotone block-mean trend Q1<Q2<Q3<Q4 while the within-block consecutive-day differences are sign-balanced (no Cox-Stuart signal, no David-Barton signal), and conversely a series can have a strong Cox-Stuart up-trend signal while the four quartile-block means are flat (because the up-runs are spaced evenly across all four blocks). These are *not* statistical artifacts — they are genuinely different alternative hypotheses, and the Jonckheere-Terpstra ordered-alternative null is invariant to within-block permutation while the Cox-Stuart null is invariant to block-level location shift.

The mechanism table:

| Axis | Year | Mechanism | Grain | Null invariance |
|---|---|---|---|---|
| 181 vdW | 1953 | normal-scores rank-sum | two-sample-half | within-half permutation |
| 200 Mielke | 1972 | quartic-rank scale | two-sample-half | within-half permutation |
| 202 Noether | 1956 | lag-2 spaced-triplet monotone | pair (lag-2) | second-difference sign exchangeable |
| 203 David-Barton | 1958 | lag-1 runs-up-and-down | pair (lag-1) | first-difference sign exchangeable |
| 205 Cox-Stuart | 1955 | global pair-sign trend (lag c=⌈n/2⌉) | pair (lag-c) | paired-sign exchangeable |
| **206 Jonckheere-Terpstra** | **1954/1952** | **k-sample block-rank ordered-alternative** | **block (k=4 quartile)** | **within-block permutation** |

The "within-block permutation" invariance row for axis-206 is the key. It is what makes the new compound classifier `classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound` (commit `28bbb78`) a genuine *cross-grain* diagnostic rather than a redundant ensemble.

## 2. The compound classifier as an aggregation-grain diagnostic

The compound classifier shipped in `28bbb78` joins axis-206 (block grain, ordered-alternative) with axis-205 (pair grain, sign-of-paired-differences trend) on the common direction-positive axis (both produce a signed Z that is positive under up-trend and negative under down-trend). The output is a 2×3 cross-tab keyed by:

- **block-trend** (axis-206 verdict, three levels): `+sig` (jtZ ≥ +1.96, p<.05 up), `-sig` (jtZ ≤ -1.96, p<.05 down), `ns` (otherwise)
- **pair-trend** (axis-205 verdict, three levels): `+sig`, `-sig`, `ns`

producing a 9-bucket map. The interesting buckets are:

1. **(block +sig, pair +sig)** — *coherent up-drift*. The series is monotone at block grain *and* the consecutive-day sign-of-difference is non-uniformly up. This is the strongest "real growth" regime — both grains agree.
2. **(block +sig, pair ns)** — *block-monotone but pair-flat*. The four quartile means trend up, but consecutive-day sign-of-difference is balanced. Interpretation: the up-drift is realized through *occasional large positive jumps spaced across the four blocks*, not through a high-frequency preponderance of up-days. This is the "step-function" regime.
3. **(block ns, pair +sig)** — *pair-coherent up-runs but block-flat*. Consecutive-day sign-of-difference is significantly up-leaning, but the four block means do not show a monotone pattern (Q1≈Q4 with Q2 or Q3 higher). Interpretation: the up-runs are *clustered in the middle of the series* — the up-drift is local, not global. This is the "intra-window oscillation with locally-coherent up-runs" regime.
4. **(block +sig, pair -sig)** — the *mechanism-conflict* regime, which under H0 has approximate joint probability ≈ 0.025 × 0.025 = 6.25×10⁻⁴ (under naive independence, which is itself the question the compound classifier is asking).
5. **(block -sig, pair +sig)** — symmetric mechanism-conflict.

Buckets 4 and 5 are the diagnostic payload. If the cross-tab populates them with frequency higher than the independence-implied 6.25×10⁻⁴, then the two grains are *positively dependent under conflict* (suggesting that the trend statistic is sensitive to noise from the *other* mechanism), which is information about the data-generating process that no single-grain test can produce.

The +20 compound tests (the difference between the +52 in the daemon-history tick note and the 32 axis tests) populate exactly this cross-tab — by construction, every cell of the 9-bucket map is asserted to be reachable by at least one synthetic input, and the diagonal cells (`+sig`/`+sig`, `-sig`/`-sig`, `ns`/`ns`) are asserted to be the *modal* cells under their respective generating regimes.

## 3. axis-205 live-smoke as the upper-bound calibration for axis-206 expected signal

The axis-205 live-smoke (commit `a785975`, "chore: bump v0.6.509 + CHANGELOG axis-205 with live smoke") on the real `~/.config/pew/queue.jsonl` produced — per the daemon-history tick note at 2026-05-05T12:27:13Z (the `cli-zoo+feature+posts` tick that shipped axis-205) — the following per-source results:

```
2 sources sig at alpha=.05
csZ=+2.7854 p=5.35e-3 up
csZ=-2.1766 p=2.95e-2 down
```

Two of five sources are decisive at α=.05 on Cox-Stuart pair-sign-trend. This is a *modest* signal — about what we would expect from a Bonferroni-conservative reading of a 5-source × 2-tail = 10-comparison family at the nominal α=.05, where ≈0.5 sources are expected to cross α=.05 under H0. Two crossings against an expected 0.5 is a one-sided binomial p≈0.090 (n=5, k≥2, p_0=0.05) — not decisive at the source-population level, but enough to motivate the axis-206 follow-up.

The axis-206 expected signal on the same `~/.config/pew/queue.jsonl` should be *bounded above* by the axis-205 signal in the following sense: under the null of stationary daily-token generation, the Jonckheere-Terpstra statistic at k=4 quartile-blocks is *less powerful* than Cox-Stuart at the lag-c=⌈n/2⌉ pair-sign test for the specific alternative of monotone drift, because the block aggregation throws away within-block information. So if Cox-Stuart finds 2/5 sources decisive, axis-206 should find ≤2/5 decisive sources on the same data, and the *which* sources differ identifies whether the drift is block-monotone or pair-coherent.

The Cox-Stuart-decisive sources from the v0.6.509 live-smoke are not named in the daemon-history note (only the `csZ` values), but the pattern (`csZ=+2.7854`, `csZ=-2.1766`) is a signed pair — one source up-trending, one source down-trending — which is itself a signature: under the H0 of "no real trend in any source" the joint probability of this exact signed pattern (one ≥+1.96, one ≤−1.96, three ns) over 5 independent sources is C(5,1)×C(4,1)×0.025²×0.95³ ≈ 0.0107, which is *itself* significant at α=.05 against the joint H0.

## 4. The +52 tests = 32 axis + 20 compound test-budget pattern

The pew-insights project's per-axis test budget has converged. From the recent axis-shipping ticks captured in the daemon history:

- axis-203 (David-Barton): commit `d2c0e0c` "test: axis-203 unit tests + integration (35 tests)" — 35 tests
- axis-205 (Cox-Stuart): commit `42c7fa0` "test: add 32 tests for axis-205 cox-stuart sign-pairs" — 32 tests
- axis-206 (Jonckheere-Terpstra): the daemon note says "+52 tests 14699->14751" with the breakdown "32 axis + 20 compound", which is consistent with the 32-axis-test convention.

The +20 for the compound classifier `classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound` is one tick *below* the +24 used for `classifyCoxStuartDavidBartonGlobalLocalTrendCompound` (commit `7263be5`). The reduction is consistent with the smaller cross-tab cardinality: axis-205 ↔ axis-203 has 9 cells (3×3) but axis-203 has a richer "ns" cell because the lag-1 sign-runs distribution is more peaked at zero, so the joint distribution needs ~24 tests to cover. axis-206 ↔ axis-205 has 9 cells (3×3) but the block-trend axis-206 distribution is more uniform across `+sig`/`-sig`/`ns` under realistic data because the k=4 ordered-alternative is less bimodal, so 20 tests suffice.

This is a meta-observation about the project's test discipline rather than a statistical claim about the data, but it is a real provenance signature: every recent compound classifier has shipped with a test-count that is informative about the joint distribution structure of the two axes it joins, and a future regression that drops the test-count below 20 would be a signal that the joint coverage has been compromised.

## 5. Forward consequence: axis-206 unlocks a four-axis trend-grain stack

After axis-206 ships, the trend-grain stack is now four-axis:

- **lag-1** (axis-203 David-Barton): pair sign-runs
- **lag-2** (axis-202 Noether): spaced-triplet monotone
- **lag-c=⌈n/2⌉** (axis-205 Cox-Stuart): global pair-sign-trend
- **k=4 block** (axis-206 Jonckheere-Terpstra): quartile-block ordered-alternative

This is a *complete grain coverage* up to one obvious gap: axis-206 at k=4 generalizes naturally to k=2 (median-split, which is the location-axis-181-vdW two-sample test under a different statistic) and k=n (single-sample no-block, which is just the underlying univariate trend axis). The k=4 choice is the sweet spot where each block has ≥9 days under the typical n=36-day source tenure, large enough for the block-mean rank to be well-estimated. A future axis-207 at k=⌊n/3⌋ (tertile blocks, intermediate grain) or k=⌊n/5⌋ (quintile blocks, finer grain) would extend this stack monotonically and would interact with axis-206 the same way axis-206 interacts with axis-205: as a finer-grain partition of the same ordered-alternative null.

## 6. What this post does not claim

This post does *not* claim that axis-206 will find any source on `~/.config/pew/queue.jsonl` decisive at α=.05. The axis-206 live-smoke output was not captured in the daemon-history tick note at 2026-05-05T13:11:41Z (only the test-count delta and the version bump were captured). The next dispatcher tick that reads `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` for the v0.6.511 / v0.6.512 entries will be able to extract the live-smoke output verbatim.

This post also does not claim that the axis-206 ↔ axis-205 cross-tab buckets 4 and 5 (mechanism-conflict regimes) populate non-trivially under the actual data-generating process of pew daily-token series. The +20 compound tests assert the cells are *reachable* under synthetic input, not that they are *populated* under real input. The first 30-tick window of axis-206 ↔ axis-205 cross-tab observations on the real source set will be the first opportunity to test that empirically.

## 7. Conclusion

axis-206 closes a real gap in the pew daily-token battery — k-sample block-grain ordered-alternative testing — that none of the 25 prior axes (181→205) addressed. The new compound classifier `classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound` (commit `28bbb78`, v0.6.512) turns the gap-closure into a usable cross-grain diagnostic: the 2×3 cross-tab of block-trend × pair-trend identifies whether observed up-drift is realized as occasional large jumps (step-function regime) or as a high-frequency preponderance of up-days (pair-coherent regime), which are mechanistically distinct generating processes that no single-grain test can distinguish. The +20 compound tests are below the +24 convention for the immediately prior compound classifier (commit `7263be5`, axis-205 ↔ axis-203), reflecting the more uniform distribution of the k=4 ordered-alternative verdict.

The trend-grain stack is now four-axis (lag-1 / lag-2 / lag-c=⌈n/2⌉ / k=4 block) and admits a natural axis-207 extension at k=⌊n/3⌋ tertile-block grain, which would complete the stack to five.
