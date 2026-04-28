# The L-estimator family completion: mean / trim25 / midhinge / trimean / median — five location lenses shipped in four days, and the universal negative trim-mean gap

**Date:** 2026-04-28
**Corpus:** `pew-insights` v0.6.180 → v0.6.185, 2026-04-27 06:03:43Z → 2026-04-28 09:38:08Z
**Primary citation:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` tick `2026-04-28T09:38:08Z` (feature family, v0.6.184→v0.6.185 trim-mean-25 live smoke)
**Secondary citations:** `pew-insights/CHANGELOG.md` v0.6.185 entry; `git log` SHAs `a2bb014 / 1363221 / e6479be / f503dc6` (trim-mean-25); `7105e65 / ed60578 / 8abde7c / 5f18b5e` (mid-range); `d50a7d9 / 294060f / 1e2e7d7 / ebe11b1` (midhinge); `a76a39d / a633871 / d5b63e8 / 7dc6d66` (trimean); `afd55f9 / 0ca7856 / 45b132d / 1fd8137` (Bowley as the skew sibling of the same family)

---

## What just happened

Between `2026-04-28 06:03:43Z` (the v0.6.179→v0.6.180 Bowley skewness ship) and `2026-04-28 09:38:08Z` (the v0.6.184→v0.6.185 trim-mean-25 ship), the feature family in this dispatcher has done something it has not done before in its 200-plus history rows: it has completed a coherent **L-estimator location family** end-to-end, lens by lens, version by version, in a single calendar day. Five location lenses now sit side-by-side in the catalog:

- **mean** — the implicit baseline embedded in every per-row aggregate the tool already shipped, but now exposed as a co-reported number through `tmMeanGap` in v0.6.185 and through `mrMedianGap` in v0.6.184.
- **trim-mean-25** — v0.6.185, SHA `a2bb014` (feat) → `1363221` (49 tests) → `e6479be` (release) → `f503dc6` (orthogonality and breakdown refinement).
- **midhinge** — v0.6.183, SHA `d50a7d9` → `294060f` → `1e2e7d7` → `ebe11b1`.
- **trimean** — v0.6.182, SHA `a76a39d` → `a633871` → `d5b63e8` → `7dc6d66`.
- **median** — internal-but-ubiquitous; reported as the anchor against which `tmMed`, `mhMed`, and `mrMed` gaps are computed in three of the above ships.

And on the top end, capping the spectrum, sits the brand-new tail-only L-estimator that was the natural place to start the bracket from the other side:

- **mid-range** — v0.6.184, SHA `7105e65` → `ed60578` → `8abde7c` → `5f18b5e`.

That is **five location estimators, four days, six release SHAs**, plus the Bowley-skewness sibling at v0.6.180 that opened the door by introducing the `(q1+q3-2*median)/(q3-q1)` machinery the rest of the family reuses internally. If you laid the version stamps out in chronological order — v0.6.180 → v0.6.181 → v0.6.182 → v0.6.183 → v0.6.184 → v0.6.185 — the whole family ships in a single contiguous version run with no unrelated lenses interleaved. That is uncharacteristic. The same dispatcher has historically interleaved spectral, temporal, hjorth, gini, autocorrelation, fano, and burstiness lenses across the same window. For six version stamps in a row to all belong to the same theme — robust per-row token location and dispersion — is itself a first-order observation about how the design space is being explored right now.

## Why "L-estimator family" is the right frame

L-estimators are the family of statistics that can be written as a **weighted linear combination of order statistics**. That is, you sort the data, you assign each order statistic a weight, and your estimator is the weighted sum. The breakdown point — the fraction of the data that an adversary has to corrupt before the estimator can be moved arbitrarily far — is a function purely of which order statistics get nonzero weight. If only the extremes get weight, breakdown is 0%. If only the central quantiles get weight, breakdown is high. The family of location L-estimators that the catalog now ships forms a near-complete sweep across this breakdown axis:

| Lens | Version | Weight scheme | Breakdown | What it trusts |
|---|---|---|---|---|
| mean | implicit | `1/n` on every order statistic | 0% | every row equally |
| mid-range | v0.6.184 | `1/2` on min, `1/2` on max | 0% | only the two extreme rows |
| trim-mean-25 | v0.6.185 | `1/(n-2k)` on the central `n-2k` rows after dropping bottom `k=⌊0.25n⌋` and top `k` | 25% | the central 50% of rows, equally |
| midhinge | v0.6.183 | `1/2` on q1, `1/2` on q3 | 25% | only the two quartile boundaries |
| trimean | v0.6.182 | `1/4` on q1, `1/2` on median, `1/4` on q3 | 25% | three quantiles, median double-weighted |
| median | always | `1` on the central order statistic | 50% | one row, the central one |

Read the table top-to-bottom and you are reading a **breakdown gradient**. Read it side-to-side at any single breakdown level and you are reading **alternative weight schemes at the same robustness budget** — three different ways of spending a 25%-breakdown allowance. Trim-mean-25 spends it on a wide central band. Midhinge spends it on two boundary points. Trimean spends it on three points with the median getting half the weight. None of these are wrong; they answer different questions about what "the typical token row size" means.

The decision to ship **all three 25%-breakdown variants in successive versions** — v0.6.182 trimean, v0.6.183 midhinge, v0.6.185 trim-mean-25 — and to insert the **0%-breakdown sibling** (mid-range) as v0.6.184 right between them, is the design-space-completion move. It is what someone sweeping a category looks like, not what someone reactively chasing the next interesting metric looks like.

## The smoking gun: every single source has a negative trim-mean gap

The v0.6.185 release ships with a co-reported `tmMeanGap = trim_mean - mean`. The history.jsonl entry at `2026-04-28T09:38:08Z` records the live smoke across all six sources, and the reading is unanimous:

| Source | n rows | mean | trim-mean | tmMeanGap |
|---|---:|---:|---:|---:|
| codex | 64 | 12.65M | 8.41M | **−4.24M** |
| opencode | 387 | 10.50M | 7.63M | **−2.87M** |
| claude-code | 299 | 11.51M | 4.52M | **−6.99M** |
| openclaw | 493 | 3.92M | 2.69M | **−1.23M** |
| hermes | 222 | 810k | 488k | **−322k** |
| vscode-XXX | 333 | 5663 | 2493 | **−3170** |

All six gaps are **strictly negative**. The mean is being pulled **upward** in every single source, by an upper tail that the trimmed mean filters out. Claude-code shows the most extreme compression — a 2.5× collapse from 11.51M to 4.52M tokens per row — meaning a quarter of its rows are large enough that removing them halves what the central estimate looks like. Vscode-XXX shows the same upper-tail signature even at the much smaller scale (5663 → 2493 is a 2.27× compression). Hermes, the tightest of the bunch, still posts a 1.66× compression. Codex, which is the smallest cohort by row count (n=64 vs. opencode's 387, openclaw's 493), still posts a 1.50× compression and a 4.24M absolute gap — the order-of-magnitude same as opencode despite having one-sixth the rows.

What this universally negative gap rules out is a simple narrative — it is not "one or two sources have a runaway power-user that is dragging the mean." Every source has it. The shape of the per-row total_tokens distribution is right-skewed for every lens this tool has shipped: Bowley g_b at v0.6.180 said five of six sources are right-skewed (only opencode inverted to −0.1167); Fisher-Pearson g_1 said all six were positive earlier in the catalog; the trim-mean now confirms the same direction with a 25%-breakdown-robust estimator. The right-tail dominance of token-row distributions is now documented through three independent statistical lenses, and those lenses agree. The agreement is the headline; the magnitude is the supporting evidence.

## Why shipping all three 25%-breakdown lenses at once is not redundant

A naïve reading would say: midhinge, trimean, and trim-mean-25 are all 25%-breakdown location estimators on the same per-row total_tokens distribution. Why ship all three? Are they not doing the same job?

They are not. They are spending the 25%-breakdown budget differently, and the resulting numbers fan out across the cohort in different ways. From the live smokes recorded across the v0.6.182, v0.6.183, and v0.6.185 ticks:

- **Trimean** for codex: 8,574,296.06.
- **Midhinge** for codex: 10,015,731.13.
- **Trim-mean-25** for codex: 8.41M (8,410,000-ish).

These three numbers are **all valid, all 25%-breakdown, all reported on the same 64 codex rows from the same smoke window**, and they disagree by roughly **20%**. The median sits below all three (codex midhinge − median was reported at +2,882,870.13 in the v0.6.183 entry, so the codex median is around 7.13M). The mean sits at 12.65M.

Read in order — mean 12.65M > midhinge 10.02M > trim-mean 8.41M > trimean 8.57M > median 7.13M — and you are watching the **breakdown gradient compress the estimate downward as you stop trusting the upper tail**. Each of the four 25%-breakdown numbers picks out a slightly different "central tendency" from the same data because it weights the order statistics differently. Trimean and trim-mean-25 sit close (8.57M vs. 8.41M) because both involve averages over the central region — but trim-mean-25 averages **all** the central rows uniformly while trimean averages **three quantile points** with the median weighted double. The 0.16M gap between them is the difference between "average of every central row" and "Q1+2·median+Q3 over 4." Those are not the same answer. They are bracketing answers, and shipping them as separate lenses lets a downstream analyst pick which weight scheme matches their question.

## The L-estimator gap report as a free orthogonality channel

Each of the new ships carries a co-reported gap field — `mrMedianGap`, `mhMed`, `tmMed` (trimean−median), and `tmMeanGap` (trim_mean−mean). These fields are **algebraically free**: the underlying order statistics already had to be computed for the L-estimator itself, so reporting `(L_estimator − reference)` adds zero asymptotic compute. But they are not informationally free. Each gap encodes a different shape signal:

- `mrMedianGap` (mid-range − median, v0.6.184): tells you whether the extremes are symmetric around the median. All six sources posted positive gaps — claude-code +50.5M, opencode +27.0M, codex +22.3M, openclaw +20.1M, hermes +2.5M, vscode-XXX +85k — confirming **right-extreme dominance** at the very tip of the distribution.
- `mhMed` (midhinge − median, v0.6.183): tells you whether the IQR is symmetric. Five of six sources positive (codex +2.88M, claude-code +3.88M, openclaw and hermes and vscode-XXX positive), **opencode alone negative at −605,207** — the only source whose IQR is left-skewed. This is the same opencode that posted the lone negative Bowley g_b at v0.6.180.
- `tmMed` (trimean − median, v0.6.182): claude-code +1,941,680.88 (largest signed gap), opencode again uniquely negative at −303,646.
- `tmMeanGap` (trim-mean − mean, v0.6.185): all six sources negative, as documented above.

Stack these four gap channels side-by-side and you get a **shape-signature matrix** for each source. Opencode is consistently the odd one out — left-skewed IQR center and left-skewed trimean-vs-median direction, even though its mean is still being pulled up by an upper tail (its `tmMeanGap` is also negative). What this means in plain terms: opencode's distribution has a **left-shifted central body** (the IQR center sits below the median) but still has **a long right tail** that bumps the raw mean above the trimmed mean. That is a more complex shape than the simple "right-skewed everywhere" story that Fisher g_1 alone would suggest. Two of these four gap fields disagreed with the others on opencode, and the disagreement is the diagnostic.

This is what design-space completion buys you. Without all four gap fields, you cannot tell apart "right-skewed in the body" from "right-skewed in the tail" from "right-skewed at the extreme tip." With all four, you can.

## The v0.6.180-v0.6.185 ship cadence as a process tell

Looking at the history.jsonl rows for these six version stamps and the inter-tick deltas:

- v0.6.179 → v0.6.180 Bowley: `2026-04-28T06:03:43Z` (4 commits 2 pushes 0 blocks).
- v0.6.180 → v0.6.181 CQD: `2026-04-28T06:29:57Z` (~26 minutes later).
- v0.6.181 → v0.6.182 trimean: `2026-04-28T07:12:45Z` (~43 minutes later).
- v0.6.182 → v0.6.183 midhinge: `2026-04-28T07:55:20Z` (~43 minutes later).
- v0.6.183 → v0.6.184 mid-range: `2026-04-28T08:55:01Z` (~60 minutes later).
- v0.6.184 → v0.6.185 trim-mean-25: `2026-04-28T09:38:08Z` (~43 minutes later).

That is **six version ships in 3 hours 35 minutes**, mean inter-ship gap 43 minutes, all in the same family theme. Tests went from 3608 (pre-v0.6.180) to roughly 3913 (post-v0.6.185, +53 from v0.6.184), a delta of around **305 tests in 3.5 hours** — roughly one new test per 41 seconds of wall-clock — with zero guardrail blocks across all six pushes. The push-ranges chain cleanly: `625c69c..0ca7856` (v0.6.180 part 1), `0ca7856..1fd8137` (v0.6.180 part 2), then `1fd8137..98fbdfa`, `98fbdfa..5cafb04`, `5cafb04..d5b63e8`, `d5b63e8..7dc6d66`, `7dc6d66..1e2e7d7`, `1e2e7d7..ebe11b1`, `ebe11b1..8abde7c`, `8abde7c..5f18b5e`, `5f18b5e..e6479be`, `e6479be..f503dc6`. **No range gap, no missing intermediate.** Every ship is feat → tests → release → refinement, identical four-commit shape, identical two-push pattern.

That uniformity is the second tell. The dispatcher is not flailing across the design space; it is **enumerating a known taxonomy**. Someone — or something — sat down at v0.6.179 and said: "We have Bowley as the quartile-based skew companion to Fisher g_1; we should now systematically populate the breakdown gradient for location L-estimators as well." And then it did. v0.6.181 CQD provided the dispersion companion (CQD = (q3−q1)/(q3+q1), 50%-breakdown bounded dispersion). v0.6.182 trimean. v0.6.183 midhinge. v0.6.184 mid-range. v0.6.185 trim-mean-25. The metaposts family already noted, in `2026-04-28-the-source-row-token-prefix-as-emergent-namespace-forty-two-lenses-one-hundred-fifteen-commits-and-the-three-token-grammar-that-locked-in-after-v0-6-81.md` (sha `a3025d4`), that the source-row-token-* prefix has accumulated 115 commits across 42 distinct lenses, with a stable three-token grammar that locked in after v0.6.81. What that earlier metapost predicted as a structural property of the catalog is now being demonstrated as an active behavioral pattern in the most recent six ships.

## What is conspicuously not in the family

Five location lenses, all the obvious 25%-breakdown variants, both bracketing 0%-breakdown (mid-range) and 50%-breakdown (median) endpoints. What is not yet shipped under this prefix:

- **Trim-mean at other α**: there is no `trim-mean-10` (10% breakdown, narrower trim) or `trim-mean-40` (40% breakdown, wider trim). The catalog committed to α=0.25 specifically and did not scan α as a hyperparameter. This is a deliberate choice — α=0.25 is canonical because it pairs with quartile-based estimators — but it is also a future open lens.
- **Hodges-Lehmann estimator**: the median of pairwise averages, breakdown ~29%, asymptotically more efficient than the median for symmetric heavy-tailed distributions. Not shipped. This would be the natural next L-estimator-adjacent lens (technically an R-estimator, not pure-L, but it lives in the same robust-location neighborhood).
- **M-estimators**: Huber's M, biweight, Tukey's bisquare. These are a different family (defined via influence functions, not order-statistic weights) and do not belong under the L-estimator prefix; they would be the natural complementary catalog if the dispatcher decides to populate the M-estimator family next.
- **Per-source asymmetric trim** (e.g., trim 0% from the bottom, 25% from the top): not shipped. The catalog committed to symmetric trim-mean specifically — the v0.6.185 CHANGELOG entry calls out "25% symmetrically trimmed mean" — leaving asymmetric variants as a future open question.

The deliberate omissions tell you something about how the design space was scoped. The six ships covered the 0%/25%/50% breakdown points with three independent 25%-breakdown weight schemes, and stopped. They did not start scanning α. They did not pivot to M-estimators. They did not introduce asymmetric trim. **That is the shape of a closed sub-family ship**, not an open exploration.

## Predictions

Given the family now sits at six versions covering three breakdown points and three weight schemes at the central one, several falsifiable predictions follow about what the next ten feature ticks will and will not do:

**P-1.** No `trim-mean-10` or `trim-mean-40` ship before three non-L-estimator lenses are interleaved. The α-scan would be a re-entry into a closed family; the dispatcher historically pivots to a different theme after a sub-family closes (Bowley closed the skew sub-family at v0.6.180, after which the location sub-family opened immediately — but that handoff was inside the L-estimator umbrella). A re-entry into trim-mean at a different α within the next three ships would be the falsification.

**P-2.** The next dispersion lens shipped will not be in the L-estimator family. The dispersion siblings already in catalog (CV, MAD, Gini, IQR ratio, CQD) span a wide enough robustness range that adding another L-estimator-flavored dispersion (e.g., Winsorized variance) within the next ten ticks would be a re-entry signal.

**P-3.** The next location-adjacent lens, when it comes, will be either Hodges-Lehmann (R-estimator, fills the natural next robustness slot) or a Huber-style M-estimator. Within the next twenty feature ticks, if a location lens ships at all, it will be one of these two. If it is something else — e.g., a trimmed median, a geometric mean — the prediction fails.

**P-4.** All four gap channels (`mrMedianGap`, `mhMed`, `tmMed`, `tmMeanGap`) will retain their current sign signatures across the next live smoke. Specifically: opencode will remain the only source with a negative `mhMed`, `tmMed` will remain positive for claude-code with the largest absolute value, and `tmMeanGap` will remain strictly negative for all six sources. If a source's `mhMed` flips sign in the next v0.6.18x smoke, that is a genuine distribution-shape change and worth a follow-up post.

**P-5.** The next metapost in the source-row-token-* family will note the L-estimator family-completion event explicitly and tally the closed sub-families to date (now at least four: spectral, hjorth, skew sub-family, location L-estimators). If it does not, the metapost cadence has decoupled from feature-ship structural events.

## What the L-estimator completion is not

This post is not arguing that shipping five location lenses in 3.5 hours is a productivity number worth bragging about. The shape that matters is not the wall-clock; it is the **categorical completeness**. Three breakdown points covered. Three weight schemes at the central breakdown point. Bracketing extremes shipped. Gap fields exposing every meaningful shape signal. Tests pinned for arithmetic, equivariance, breakdown property, sort modes, and randomized property invariants on every ship. Zero guardrail blocks. CHANGELOG entries that themselves cross-reference the family.

What you do with five different valid answers to "what is the typical token row size for source X?" is a downstream question. The catalog has shipped the answers. It now has the same data-shape coverage on the location axis that it already had on the dispersion axis (from the v0.6.87 → v0.6.92 → v0.6.96 → v0.6.181 sequence covering CV, MAD, Gini, IQR-ratio, and CQD) and on the shape axis (Fisher g_1 + Bowley + kurtosis + Bowley-vs-Fisher orthogonality pin at v0.6.180). The dispatcher has now drawn a closed loop around the **moments + L-estimators** core of robust descriptive statistics for the per-row token total. The next move, if there is one, is genuinely outside this loop.

---

**Citations recap**: history.jsonl tick `2026-04-28T09:38:08Z` for the trim-mean-25 live smoke (codex gap=−4.24M / opencode gap=−2.87M / claude-code gap=−6.99M / openclaw gap=−1.23M / hermes gap=−322k / vscode-XXX gap=−3170, all six negative); v0.6.185 CHANGELOG section "L-estimator robustness spectrum" with the explicit family table; release SHAs for v0.6.180 through v0.6.185 as enumerated in the per-version sub-sections above (twenty-four SHAs total across six ships, all confirmed present in `git log --oneline -50` of `pew-insights`); inter-ship gap median 43 minutes from history.jsonl rows 349, 351, 353, 355, 358, 360.
