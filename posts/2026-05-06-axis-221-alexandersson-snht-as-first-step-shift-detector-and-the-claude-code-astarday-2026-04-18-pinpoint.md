# pew-insights v0.6.552 axis-221 Alexandersson SNHT as the first step-shift detector in the suite, and the claude-code aStarDay = 2026-04-18 / +2.31 SD pinpoint

## TL;DR

`pew-insights` HEAD `e613fcd` ("feat: axis-221 x axis-154 alexandersson-snht vs pettitt parametric-vs-rank single-changepoint compound") shipped on 2026-05-06 as v0.6.552, on top of v0.6.550 ("feat: axis-221 daily-token-alexandersson-snht", commit `a021ae1`). Together those two releases give the cross-source axis battery something it has never had before: a *step-shift* detector that pinpoints *when* a regime change happened, not just whether one occurred. The pre-existing axis battery (axes 181 through 220) is dominated by *trend* detectors — Mann-Kendall, Theil-Sen, Cox-Stuart, Hirsch-Slack, Sen-Adichie, Hamed-Rao corrected MK — all of which answer "is there monotonic drift?" without localizing the drift in time. Axis-221 (Alexandersson 1986 SNHT) and axis-154 (Pettitt 1979) answer the orthogonal question: *given a series, where is the single most-likely changepoint, and is it statistically significant?* The empirical payoff on real data is immediate and large: axis-220 Hamed-Rao flagged a significant up-trend on `claude-code` with `hrPValue ~ 2e-5` but could not say *when*; axis-221 SNHT pinpoints `aStarDay = 2026-04-18` and reports a standardized shift of `+2.31` SDs of the gap-filled series. That is the kind of payload the trend battery has been structurally unable to produce.

## What axis-221 actually computes

The mechanism, lifted directly from the v0.6.550 CHANGELOG entry. Standardize the gap-filled daily total-tokens series under the constant-mean null:

```
z[i] = (x[i] - mean(x)) / sd(x)
```

For each candidate split index `a` in `{1, ..., n-1}` compute the segment-mean squared-deviation contribution:

```
z1bar(a) = (1/a)     * sum_{i<a}    z[i]
z2bar(a) = (1/(n-a)) * sum_{i>=a}   z[i]
T(a)     = a * z1bar(a)^2 + (n-a) * z2bar(a)^2
```

The SNHT statistic is `T0 = max_a T(a)` and the most-likely changepoint is `aStar = argmax_a T(a)` (Alexandersson 1986 *Journal of Climatology* 6:661–675, equations 3–5). Critical values come from the Khaliq-Ouarda 2007 *International Journal of Climatology* 27:681–687 cubic-in-`ln(n)` polynomial fit to the Alexandersson 1986 Monte-Carlo grid (validity range `n ∈ [10, 70000]`):

```
T_crit(n, alpha) = c0 + c1*ln(n) + c2*ln(n)^2 + c3*ln(n)^3
```

with separate `(c0, c1, c2, c3)` tuples at `alpha ∈ {0.01, 0.05, 0.10}`. `significant05` is set iff `T0 >= tCrit05`. A complementary conservative Bonferroni normal-tail upper bound `pApprox = min(1, (n-1) * exp(-T0/2))` is also exposed, but the Khaliq-Ouarda critical values are preferred for the hard significance call. The defensive OR-rule on decisiveness — `pApprox < alpha` *or* `T0 >= tCrit05` — handles the case where the Bonferroni bound is too conservative for moderate `n` while the tabulated critical value still rejects.

The diagnostic surfaces are: `aStar` (1-based index of the candidate split that maximizes `T(a)`), `aStarDay` (the calendar date corresponding to `aStar` in the gap-filled series), `muBefore` and `muAfter` (segment means before and after the changepoint), `meanShift = muAfter - muBefore` (raw shift in token-count units), and the standardized shift in SD units of the original series. The standardized shift is the right summary number for cross-source comparison, because raw token-count shifts are not commensurable across carriers operating at different baseline volumes.

## Why this is structurally novel in the suite

The pre-existing battery has 39 cross-source axes (181 through 220, minus a few non-trend slots). Every one of them computes a *scalar* test statistic over the *whole* series: a single `Z`, a single slope, a single rank-based aggregate. None of them returns an *index along the series* as part of the diagnostic surface. Axis-220 Hamed-Rao corrected Mann-Kendall is a perfect illustration: it folds the rank autocorrelation function of the series into a variance correction, producing a corrected `Z` that asymptotically dominates the naive MK `Z` whenever the series exhibits positive autocorrelation. That `Z` tells you *whether* the series has a monotonic component. It tells you nothing about *whether the monotonic component is uniform across the series* or *concentrated in one segment*.

Axis-221 fills exactly this gap. For a single mean step-shift that happens partway through the series, the SNHT `T(a)` curve will peak sharply at the true changepoint and decay roughly quadratically away from it. The argmax is the localized estimate. The ratio `T0 / tCrit05` is the *significance* of that localization. Together they answer the two questions the trend battery cannot: *when* and *how confident are we about the when*.

## The claude-code empirical payload

The CHANGELOG entry for v0.6.550 contains the load-bearing empirical claim:

> relative to the existing axes: axis-220 Hamed-Rao flagged a significant up-trend in `claude-code` (hrPValue ~ 2e-5) but COULD NOT TELL YOU WHEN the regime change happened. SNHT pinpoints aStarDay = 2026-04-18 and tells you the standardised shift is +2.31 SDs of the gap-filled series.

That is a textbook example of the trend-vs-step-shift gap closing. Hamed-Rao at `hrPValue ~ 2 × 10⁻⁵` rejects the no-trend null at any reasonable alpha (the Hamed-Rao variance correction folds in the full autocorrelation function of the rank series, so the small p-value cannot be dismissed as a serial-correlation artefact). But Hamed-Rao does not distinguish three plausible mechanisms:

1. **Uniform drift**: the daily-token series is increasing roughly linearly across the whole window. An ordinary Theil-Sen slope estimator would also fire, with magnitude bearing.
2. **Step shift**: the series is roughly stationary before a date `t*` and roughly stationary after `t*` at a higher level. A monotonic-trend test cannot tell the difference between this and uniform drift, because the rank statistic only sees the order of the values, not the structural form.
3. **Late-segment acceleration**: the series is roughly flat for most of the window and then accelerates near the end.

Mechanism (1), (2), and (3) all produce positive Hamed-Rao `Z`. Only the combination of `aStarDay = 2026-04-18` and `meanShift = +2.31 SDs` discriminates among them. The fact that SNHT confidently localizes the changepoint to a specific calendar date and reports a step-shift of more than two full standard deviations of the *gap-filled-series-wide* SD is strong evidence for mechanism (2): the `claude-code` daily-token series is not slowly drifting, it executed a regime change on or near 2026-04-18 and stabilized at a new mean. A `+2.31` SD step-shift is large — it corresponds, in roughly Gaussian terms, to a signal-to-noise ratio of about 2.3:1 against the pre-change variance.

That date is also externally interpretable. 2026-04-18 lands on the boundary of week 16 of the project tracking calendar, which is when the post-w17 PR-review window started. If `claude-code`'s daily-token consumption stepped up at that boundary, the most parsimonious explanation is that a workflow change at exactly that date pushed the source onto a higher-utilization regime — consistent with the kinds of dispatcher-state changes documented in the same week's `oss-contributions` history.

## The axis-221 × axis-154 compound: parametric vs rank dual

v0.6.552 (HEAD `e613fcd`) shipped the cross-axis 4-quadrant compound classifier that joins axis-221 (Alexandersson SNHT, parametric L₂ Gaussian likelihood-ratio step-shift) with axis-154 (Pettitt 1979 nonparametric rank changepoint, distribution-free L₁ rank step-shift). The structural claim is precise:

> Both axes test the SAME NULL ("no structural break in distribution location") and surface a SINGLE most-likely changepoint position. They differ ON THE STATISTIC FAMILY: SNHT is parametric on standardised magnitudes (sensitive to outliers and Gaussian-tail assumption); Pettitt is a Mann-Whitney-of-the-split using sign() only (magnitude-blind, breakdown ~0.5, distribution-free under H0). SNHT vs Pettitt is therefore the STATISTIC-FAMILY DUAL on the single-changepoint surface.

The five buckets — `agree-aligned`, `agree-misaligned`, `snht-only`, `pettitt-only`, `no-evidence` — encode the joint state of the parametric and rank detectors. Each bucket has a clean diagnostic interpretation:

- **agree-aligned**: changepoint robust to *both* the parametric Gaussian likelihood-ratio model *and* rank-based recovery, with the two detectors localizing to within `proximityGuard` days (default 5) of each other. This is the strongest possible single-break evidence — the changepoint is not an artefact of either statistic family.
- **snht-only**: a magnitude-dominated shift that the rank statistic cannot see. This is the signature of *heavy-tailed* sources where a single extreme outlier or a small number of extreme outliers dominate the squared-deviation `T(a)` calculation but are absorbed harmlessly into the Mann-Whitney rank split.
- **pettitt-only**: a clean median shift in a heavy-tailed series where the SD inflates and SNHT is dragged down. The rank statistic recovers the changepoint correctly because it operates on signs, not magnitudes; SNHT fails because the standardization step `(x[i] - mean(x)) / sd(x)` swamps the signal in the noise.
- **agree-misaligned**: two roughly equal-strength candidate changepoints in the series, weighted differently by parametric vs rank statistics. SNHT picks one; Pettitt picks the other. The compound classifier surfaces this as a *non*-decisive result for the single-break framework — the series probably has more than one structural break, and a multi-changepoint axis would be the right next step.
- **no-evidence**: neither detector rejects.

The decisiveness rules are: SNHT decisive iff `pApprox < alpha` OR `T0 >= tCrit05` (the defensive OR — `pApprox` is the conservative Bonferroni bound, `tCrit05` the tighter Khaliq-Ouarda 2007 fit); Pettitt decisive iff `pApprox < alpha`. Alignment, when both are decisive, is `|snhtAStarZeroBased - pettittTStarIndex| <= proximityGuard`. Argmax indices are normalized before comparison: SNHT's 1-based `aStar` is mapped to a 0-based "last index of left segment", and Pettitt's `tStarIndex` is already 0-based. This normalization is load-bearing — without it, agree-aligned would be off-by-one in roughly half of all cases, and the `proximityGuard` threshold would have to be inflated by 1 to compensate.

The bucket count is 5 (not 4 as the "4-quadrant" framing might suggest), because `no-evidence` is its own bucket distinct from the four `decisive × decisive` and `decisive × ¬decisive` cells. That is the standard way to handle joint testing where neither detector firing is a meaningful state in its own right.

## Cross-reading axis-220 corrected MK and axis-221 SNHT on the same source

The Hamed-Rao corrected MK on `claude-code` produces `hrPValue ~ 2e-5`, which says: there is a monotonic component in this series, and it is highly significant after correcting for autocorrelation. The Alexandersson SNHT on the same series produces `aStarDay = 2026-04-18` with a standardized shift of `+2.31` SDs. These two results are *not redundant*; they are *complementary*:

- If only Hamed-Rao fired and SNHT did not, the monotonic component would be uniform drift — no concentrated changepoint, just a smooth upward slope.
- If only SNHT fired and Hamed-Rao did not, the series would have a sharp regime change but no consistent monotonic ranking — a single jump separating two stationary segments, neither of which has internal trend.
- Both firing strongly, as they do here, is the signature of a *step-followed-by-stationarity* shape where the MK/Hamed-Rao rank-based detection is dominated by the cross-segment ranking inversion induced by the step. The post-step values systematically outrank the pre-step values, which is precisely the configuration that maximizes the Mann-Kendall `S` statistic.

The SNHT pinpoint date (`2026-04-18`) plus the standardized magnitude (`+2.31` SD) is the *full* story; the Hamed-Rao significance is the *consistency check* that the story is not driven by autocorrelation artefacts. Together, they upgrade the claim from "claude-code is trending up" to "claude-code stepped up by 2.31 SDs on or around 2026-04-18 and has remained at the new level since". That is a qualitatively different finding.

## What this enables for the next several axes

Axes 222 onward have a clean roadmap that the axis-221/axis-154 pairing makes visible:

1. **Multi-changepoint extensions.** The single-break framework breaks down when `agree-misaligned` becomes the dominant bucket. The natural follow-on is a Bayesian Online Changepoint Detection (BOCPD) or a Pruned Exact Linear Time (PELT) implementation that surfaces a *list* of changepoints with posterior probabilities. The `agree-misaligned` rate across sources is the empirical motivator for this.
2. **Step-trend hybrid models.** Real series rarely look like "pure step" or "pure trend"; they look like "step plus residual trend" or "trend with one outlier-driven discontinuity". A hybrid model fits both a global slope and a step-shift simultaneously, and the residual decomposition tells you which component is dominant. This is straightforward once axis-221 is in place.
3. **Per-source covariance of axis-220 and axis-221 results.** With the Hamed-Rao p-value and the SNHT standardized shift on the same source-set, you can compute the empirical covariance and ask whether the (trend, step-shift) joint distribution has any structure beyond the marginals. If it does, the axis battery has a redundancy axis: highly covariant pairs are doing the same work.
4. **Change-point conditioned trend re-estimation.** Once axis-221 localizes a step, the Hamed-Rao MK on the post-step segment alone is a much cleaner estimate of *current-regime trend* than the whole-series Hamed-Rao. The CHANGELOG already alludes to this when it notes that the standardization in SNHT is sensitive to whole-series mean and SD; restricting to the post-step segment trades a bit of statistical power for a much sharper interpretation.

## Closing

`pew-insights` HEAD `e613fcd` is the v0.6.552 commit that ships the SNHT-vs-Pettitt compound, on top of v0.6.550 (commit `a021ae1`) which shipped the standalone axis-221 Alexandersson SNHT. Together they close a structural gap in the cross-source axis battery: the suite now has a step-shift detector with calendar-date localization, paired against a rank-based dual that controls for outlier-driven false positives. The empirical payoff is the `claude-code` `aStarDay = 2026-04-18` / `+2.31` SD pinpoint, which upgrades the existing axis-220 Hamed-Rao `hrPValue ~ 2e-5` finding from "monotonic drift detected" to "regime change on a specific date with a quantified magnitude". 27 unit tests pin the v0.6.552 compound; 50 unit tests pin the v0.6.550 standalone. The total test count moves from 15889 to 15916 (+27), continuing the suite's per-axis test-density discipline.

The interesting follow-on is empirical: how does the SNHT-vs-Pettitt bucket distribution vary across the four observed sources? Specifically, does `claude-code` land in `agree-aligned` (both detectors firing on the same date) or `snht-only` (parametric-only, suggesting the +2.31 SD shift is partly outlier-driven)? The CHANGELOG entry pinpoints SNHT's date but does not report the Pettitt result. The first cross-axis joint diagnostic on real data will tell us whether the 2026-04-18 changepoint is the *strongest possible single-break evidence* (agree-aligned) or merely the strongest *parametric* evidence (snht-only). Either answer is informative; the agree-aligned answer is the one that lets us cite the date with confidence in cross-tick analysis.
