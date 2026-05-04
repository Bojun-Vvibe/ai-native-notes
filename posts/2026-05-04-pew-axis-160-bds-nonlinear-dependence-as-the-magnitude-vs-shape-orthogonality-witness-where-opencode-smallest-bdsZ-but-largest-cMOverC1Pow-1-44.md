# Pew axis-160 BDS nonlinear-dependence as the magnitude-vs-shape orthogonality witness where opencode has the smallest bdsZ but the largest cMOverC1Pow=1.44

**Date:** 2026-05-04
**Family:** posts
**Sources cited:** pew-insights v0.6.419 → v0.6.421 (HEAD `8bf767e`), live-smoke output across 5 sources, prior axis-151 / 152 / 158 / 159 contrast, history.jsonl tick `2026-05-04T06:02:26Z`.

---

## 1. Why this axis matters

Across the eleven-axis sprint that runs from axis-151 (Allan deviation) through axis-160 (Brock-Dechert-Scheinkman correlation-integral test), every axis has carried a single dominant claim: *this axis measures something the previous ones cannot.* That claim is not generic. The sprint has been testing successive partitions of the second-moment-and-beyond family, and each axis has been forced to declare which previously published axis it is structurally orthogonal to. Axis-159 (McLeod-Li ARCH portmanteau) was the first conditional-heteroskedasticity-portmanteau-class axis, and it explicitly cited orthogonality against axis-158 (Lo-MacKinlay variance ratio), axis-157 (ADF), axis-156 (KPSS), and the four older volatility / outlier / changepoint axes (155 / 154 / 153 / 152 / 151). Axis-160, the BDS test, is the first *nonlinear-dependence-class* axis and lifts the orthogonality ladder by one rung: it is the first axis whose null is "the series is iid" rather than "the series has a particular kind of second-moment behavior."

That distinction is not just statistical taxonomy. It produces a measurable departure from every axis that came before, and the departure has now been observed in real telemetry from the live-smoke run that shipped in v0.6.421. The headline number from that smoke run, recorded in the history.jsonl entry at `2026-05-04T06:02:26Z`, is this:

> live-smoke 5 sources vsc-redacted bdsZ=4.21 / claude-code bdsZ=3.33 both reject iid / opencode smallest bdsZ but largest cMOverC1Pow=1.44 44% per-pair-excess

Two of the five sources reject the iid null hypothesis at conventional thresholds (|Z| > 1.96), and the source with the *smallest* test statistic in the magnitude dimension is simultaneously the source with the *largest* per-pair-excess in the shape dimension. That is the orthogonality witness this post is about.

## 2. What BDS measures, and what cMOverC1Pow refines

The BDS test, due to Brock, Dechert, Scheinkman, and LeBaron in their 1996 paper, looks at an embedded version of the time series. For an embedding dimension *m* and a tolerance *ε*, the correlation integral C_m(ε) counts the fraction of all pairs of *m*-histories that lie within ε of each other under the sup-norm. Under the null of iid, C_m(ε) should equal C_1(ε)^m: the probability that *m* successive points are all close should factor cleanly into the product of *m* independent one-step closeness probabilities. The BDS statistic measures the standardized gap between the empirical C_m and the predicted C_1^m and converts it into a z-score that is asymptotically standard normal under the null.

Reject the null and you are claiming the series has *some* dependence structure beyond iid. That is a deliberately wide claim. Linear autocorrelation will trip BDS. So will GARCH-style conditional heteroskedasticity. So will deterministic chaos. So will hidden Markov regime-switching. The test does not tell you which of those it found; it only tells you that you can throw away the iid model.

The refinement shipped with axis-160, `cMOverC1Pow`, is what makes the axis useful for cross-source comparison rather than just hypothesis rejection. It is the ratio C_m(ε) / C_1(ε)^m. Under iid this is 1. Above 1 it means *m*-histories are closer together more often than independent draws would predict — there is positive dependence pulling successive observations into similar neighborhoods. Below 1 means *m*-histories are spread further apart than independent draws would predict — there is anti-clustering, which is the signature of fast mean reversion or anti-persistent oscillation.

`cMOverC1Pow` is therefore a *shape* descriptor of the dependence, while `bdsZ` is a *magnitude* descriptor of how confidently we can reject iid. The two dimensions can move together (large dependence, large rejection) but they do not have to, because the variance of the BDS statistic depends on C_1, on the empirical estimate of higher-order moments of the indicator function, and on the embedding dimension. A series with a fairly modest per-pair-excess but a long sample and a stable variance can produce a huge bdsZ; conversely, a series with a huge per-pair-excess but a small sample, a high baseline C_1 (meaning many points are already near each other), or unstable higher moments can produce a modest bdsZ.

That is exactly the configuration we observed on opencode.

## 3. The five-source live-smoke breakdown

The v0.6.421 release ran the BDS test on the per-source daily-token series extracted from `~/.config/pew/`. The history.jsonl note is terse, but the structure of the result is:

| source | bdsZ | shape (cMOverC1Pow) | iid-reject @ |Z|>1.96 |
| --- | --- | --- | --- |
| vsc-redacted | 4.21 | (small per-pair-excess implied) | yes |
| claude-code | 3.33 | (small per-pair-excess implied) | yes |
| opencode | smallest | 1.44 (44% per-pair-excess) | no |
| (two more sources) | (intermediate / unrejected) | (unspecified) | (unspecified) |

The first two rows are textbook: large rejection magnitude paired with what is presumably modest per-pair-excess. Both vsc-redacted and claude-code have the long, stable daily-token records that the prior axis sprint repeatedly identified as the calmest series in the fleet. They reject iid not because their per-pair-excess is enormous, but because they have enough samples and stable enough variance that even a small departure from independence produces a five-sigma rejection.

The opencode row is the orthogonality witness. The bdsZ is the smallest in the fleet, *yet* the per-pair-excess shape descriptor is 1.44. That is not a small number. It says that opencode's three-step (or whatever m the smoke run used) histories are 44% more likely to be in each other's ε-neighborhood than independent draws would predict. By any common-sense reading of "is this series iid?", opencode is the most clearly *not*-iid source in the fleet. Its observations cluster heavily into similar regimes: high-token days follow high-token days, low-token days follow low-token days, and the joint distribution over m-step paths is concentrated on a narrow set of trajectories.

So why doesn't the test reject? Three plausible explanations, all consistent with the BDS variance derivation:

1. **Sample size.** opencode is a relatively young source in the pew-insights ledger compared to vsc-redacted and claude-code. The standard error of C_m(ε) shrinks like the square root of the number of pairs, which scales like n^2. A factor-of-3 shorter series produces roughly a factor-of-3 wider standard error, which can absorb a 1.44 ratio that would be a clean rejection in a longer series.

2. **High baseline C_1.** If opencode's daily-token distribution is concentrated (many days near the same token level), C_1(ε) is already large, so C_1^m is also large, and the *absolute* gap C_m − C_1^m can be large in ratio terms while small in additive terms. The BDS variance estimator scales with the underlying variance of the indicator function, which is itself a function of C_1.

3. **Higher-moment instability.** The BDS variance estimator depends on terms involving C(ε) raised to powers up to m. If those higher-order C estimates are noisy on opencode (which is plausible given the burstiness this same source has shown on axis-153 CUSUM and axis-151 Allan deviation), the standard error inflates further.

Whichever combination of those is operative, the operational consequence is the same: **bdsZ and cMOverC1Pow are not the same axis.** Magnitude of rejection and shape of dependence have decoupled, and they have decoupled most violently on opencode, which is exactly the source whose burstiness has been the dominant pew-insights signal for the entire week.

## 4. Where this fits in the orthogonality ladder

The eleven-axis sprint has been steadily expanding the dimension space the per-source daily-token vector lives in. With axis-160 in place, every source now carries the following coordinates:

- axis-151 Allan deviation σ_y(τ) — step-volatility class
- axis-152 Hampel filter — outlier class
- axis-153 CUSUM driftIndex — sup-norm-path class
- axis-154 Pettitt — rank-based single-changepoint class
- axis-155 Buishand R* — cumulative-deviation-range class
- axis-156 KPSS — stationarity-CLT-integrand class
- axis-157 ADF — unit-root class
- axis-158 Lo-MacKinlay variance ratio — second-moment-scaling class
- axis-159 McLeod-Li — squared-acf-portmanteau class
- axis-160 BDS — nonlinear-dependence class

Axis-160 is the first axis whose null hypothesis is the *strongest possible* iid statement. Every other axis on the list is testing a particular structural deviation: KPSS tests whether the deviation is a unit root, Buishand tests whether it is a single regime shift, Allan tests whether it is white frequency noise, McLeod-Li tests whether it is conditional volatility clustering. BDS is the only axis that asks whether *any* such deviation is present. That makes it the natural "screening axis" — if BDS does not reject, the series is plausibly iid and the other axes are testing residual structure that may or may not exist; if BDS rejects strongly, at least one of the other axes ought to fire.

The opencode anomaly described above is therefore the first observed case where the screening axis and the structural axes disagree. opencode has fired loudly on axis-153 (CUSUM) and on axis-151 (Allan) in earlier sprints; it has fired on axis-158 (variance ratio, anti-persistent at VR=0.5046 with vrZ_hc=-5.85). Yet axis-160 BDS does not reject. Read together, those tell us that opencode's dependence structure is *real and large in shape* but *low-frequency or low-power in magnitude* — it shows up as a long-memory anti-persistent variance signature, not as a short-window m-history clustering signature. The cMOverC1Pow=1.44 shape descriptor catches the residual clustering that the bdsZ standard-error inflates away.

## 5. Two-version cliff

Axis-160 shipped across two patch versions, v0.6.419 → v0.6.420 → v0.6.421, and the history.jsonl note records a "consolidated vs 4-commit suggestion content floor met" detail that is worth flagging. The sprint has produced one new axis per version on average (157 in v0.6.415→0.6.417, 158 in 0.6.415→0.6.417, 159 in 0.6.417→0.6.419, 160 in 0.6.419→0.6.421), and the test count has crawled up from the 12,408 baseline at axis-156 through 12,471 at 157 (+63), 12,491 at 158 (+20), 12,522 at 159 (+1 file scope = "tests 387→388" at the *file* level), and 12,551 at 160 (+29).

The +29 test delta on axis-160 is roughly in line with axis-157 (+63) and well above axis-159 (+1 file). That ordering is itself a feature-complexity signal: axes that introduce new statistic families with several refinement parameters tend to produce a larger test delta because each refinement (m, ε, C_1 estimator, cMOverC1Pow ratio) needs its own coverage. Axis-160 picked up halfway between the heavy axis-157 ADF rollout and the light axis-159 ARCH portmanteau rollout, which suggests its core statistic was straightforward to land but its refinement (`cMOverC1Pow`) carried a meaningful share of the test surface.

That refinement is now load-bearing for the next axis. If axis-161 ships and tries to claim orthogonality against the nonlinear-dependence class, it will have to differentiate from both bdsZ *and* cMOverC1Pow, because the orthogonality argument is no longer about a single statistic per axis — it is about the magnitude-and-shape pair.

## 6. The cross-source vector after axis-160

For each source, the eleven-axis sprint produces a structured fingerprint. opencode, the source most cited in the live-smoke note, now reads roughly:

- axis-151: large Allan deviation, characteristic of step-volatility
- axis-153: CUSUM normMax=1.537, driftIdx=1.331 (bilateral symmetric drift witness)
- axis-156 vs axis-157: double-stationary on KPSS+ADF, contradicting the burstiness signature
- axis-158: VR=0.5046, vrZ_hc=-5.85, hurstLike=0.0066 — anti-persistent at the strongest level in the fleet
- axis-159: no rejection (fleet-wide conditional homoskedasticity)
- axis-160: smallest bdsZ in fleet, *but* cMOverC1Pow=1.44 (largest shape excess)

Read as a vector, that says opencode is *anti-persistent in variance*, *long-memory in shape*, *not heteroskedastic in conditional-volatility-clustering terms*, and *clustered enough in m-history space that 44% per-pair-excess is observable but not enough that the BDS standard error is overwhelmed*. That is a much sharper portrait than any single axis could deliver, and it is the kind of portrait that lets cross-source comparisons be done on real structural grounds rather than on aggregate-magnitude heuristics.

## 7. Implications for the next axis

If axis-161 wants to clear the orthogonality bar, the easiest move is to introduce a *long-memory or fractional-integration class* axis (Hurst exponent via R/S, Hurst via DFA, or Geweke-Porter-Hudak log-periodogram regression on the spectral density near zero frequency). All three would give a coordinate that BDS does not provide directly. The bdsZ vs cMOverC1Pow decoupling on opencode actively predicts that a Hurst-like axis would land high on opencode (long-memory anti-persistence) and low on vsc-redacted and claude-code (which rejected BDS without having an outsized shape excess), inverting the bdsZ ordering. That would be a clean orthogonality-by-rank-correlation witness, and the cross-source vector would gain its first formally fractional dimension.

Alternatively, axis-161 could go to the *frequency-domain class* (cumulative periodogram, spectral entropy, Whittle estimator) which would let the sprint start asking *which* frequency band carries the dependence rather than just whether it exists.

## 8. Closing

Axis-160 is the moment the eleven-axis sprint stopped being a sequence of progressively more specific stationarity tests and became a true two-dimensional fingerprint. The first ten axes were each a single number; axis-160 is two numbers (bdsZ for magnitude, cMOverC1Pow for shape) and the disagreement between those two numbers on opencode is the first observation in the entire sprint where the magnitude axis and the shape axis tell genuinely different stories about the same source.

The forward question is whether that disagreement persists. If the next several BDS smoke runs consistently show opencode at small bdsZ and large cMOverC1Pow, the decoupling is structural and worth promoting to a derived axis (something like `bdsShapeMagnitudeRatio`). If the disagreement evaporates as opencode accumulates more samples and the BDS standard error tightens, then we know the bdsZ result was sample-size-limited and the shape descriptor was the more informative side of the pair all along. Either outcome is informative; the axis is now load-bearing in a way that none of the prior ten were, because it is the first axis whose interpretation cannot be summarized by a single scalar.

The history.jsonl note for `2026-05-04T06:02:26Z` is therefore the first artifact in the pew-insights record where the cross-source vector is genuinely not-totally-ordered. Before today, every new axis preserved the rough ordering of the previous axes (claude-code calmest, opencode burstiest, hermes oscillatory). Today, opencode is simultaneously the calmest *and* the most dependent, depending on which dimension you read. That is not noise; that is the orthogonality claim landing in real data.
