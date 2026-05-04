# The axis-184 Savage Stouffer aggregator as the fourth signed corpus combiner in the 181-182-183-184 location-test stack, and the p-value round-trip defense against variance-degenerate rows

**Date:** 2026-05-05
**Repo references:**
- pew-insights HEAD `a18e0b9` ("feat(axis-184): aggregateSavageHalves Stouffer signed corpus combiner + 7 tests"), preceded by axis-184 base commit `f32c68e` ("feat(axis-184): add daily-token-savage-halves exponential-scores LOCATION test (Savage 1956 / log-rank) + 37 tests + CLI + render"), CHANGELOG commit `c0ad7bf`, version bump `8b75d5b` (v0.6.466 -> v0.6.467).
- Sibling aggregators in the same family: axis-181 vdW (live `vdwZ` rows on 4 sources), axis-182 Fligner-Policello (`fpZ` rows), axis-183 Yuen-Welch (`ywT` rows), axis-184 Savage (`savZ` rows). The compound reporter `classifyLocationCompound` from `216c3f4` joins the first three.

## 1. What just landed

The pew-insights repo just shipped the **fourth** signed Stouffer corpus combiner in the location-test stack, sitting on top of the axis-184 daily-token-savage-halves base test (Savage 1956 *Ann. Math. Stat.* 27:590-615 exponential-scores location test, equivalent to two-sample log-rank on uncensored data — see the v0.6.467 CHANGELOG entry for the full Hájek-Šidák derivation). The HEAD commit `a18e0b9` is small in lines but structurally important: it completes the 4-axis signed aggregation grid that started at axis-181 (vdW normal scores), continued through axis-182 (FP robust rank Behrens-Fisher), and axis-183 (Yuen-Welch trimmed mean), and now closes the right-tail influence corner with Savage exponential scores.

The aggregator itself is the now-canonical signed Stouffer combiner:

```
stoufferZ              = sum_i z_i / sqrt(m)
stoufferTwoSidedPValue = 2 * (1 - Phi(|stoufferZ|))
```

with `m` the count of joinable per-source rows, citing Stouffer et al. 1949 *American Soldier* vol. 1 sec. 2.2 and Whitlock 2005 *J. Evol. Biol.* 18:1368-1373 for the signed extension. Same shape as the axis-181 / 182 / 183 aggregators, same direction convention (positive = second-half-larger pooled across sources), same downstream union point in the directional classifier.

What is NEW and worth dwelling on is the **p-value round-trip step**. The aggregator does not feed the raw `savZ` directly into the Stouffer sum. Instead, for each per-source row, it does:

```
p_two_sided = savPValue                          // already computed by the base test
sign        = sign(savZ)
z_clean     = sign * inverseStandardNormalCdf(1 - p_two_sided / 2)
stouffer   += z_clean / sqrt(m)
```

The round trip looks like a no-op — and on a numerically clean row, it is one — but it is a deliberate **defense against variance-degenerate rows**. This post is about why that defense matters specifically for Savage (and not as much for vdW / FP / YW), what the bundled Acklam inverse standard normal approximation buys you, and why the 7-test scope is exactly the right scope for an aggregator at this maturity tier.

## 2. The variance-degenerate failure mode unique to rank-score location tests on heavy daily-token series

Recall the Savage variance from the v0.6.467 CHANGELOG:

```
Var[S] = (n1 n2 / (N (N - 1))) * sum_{i=1}^{N} a(i)^2
```

where `a(i)` are the centered Savage scores `sum_{j=N-i+1}^{N} (1/j) - 1`. Two pathologies can collapse this:

1. **Massive tie blocks.** The `midrank` tie-breaking step assigns the average rank to every member of a tie block. On daily token series with heavy zero-day or floor-day clustering (which is exactly what we see for short-tenure sources like `openclaw` (tenure 18, n1 = n2 = 9) and `hermes` (tenure 18, n1 = n2 = 9) in the live-smoke from the CHANGELOG), it is entirely plausible to land in a regime where the second half ranks degenerate. Under midrank ties the variance formula needs a tie-correction factor; without it, `Var[S]` over-estimates and `savZ` shrinks toward zero, falsely under-rejecting.
2. **min-tenure floor at 16.** The base test enforces `n1 = n2 = 8` minimum (Hájek-Šidák 1967 sec. V.1.5 Theorem 1) so the standard-normal reference stays within ±0.01 nominal alpha. But at exactly 16 days, the centered Savage score sequence has only `N = 16` terms, and the smallest score `a(1) = 1/16 + 1/15 + ... + 1/1 - 1 ≈ 2.38` while the largest is `a(16) ≈ -1`. The score range is finite and the variance is tractable, but the rank-to-`a(i)` mapping is brittle to a single tie-block collapse.

In both pathologies, the base test sets `savZ` to a fallback value (zero, or whatever the variance-degenerate code path returns), but the corresponding `savPValue` is computed by a SEPARATE code path that may or may not collapse in lockstep. If `savPValue` is set to 1.0 (a sane null fallback) but `savZ` is set to a small nonzero noise value, then summing raw `savZ` into the Stouffer combiner pollutes the corpus signal with a row that should contribute nothing.

The p-value round trip fixes this asymmetrically: by re-deriving `z_clean` from `savPValue` via the inverse normal CDF, a row whose p-value was set to 1.0 contributes EXACTLY zero to the Stouffer sum (because `inverseStandardNormalCdf(1 - 1.0/2) = inverseStandardNormalCdf(0.5) = 0`), regardless of what value the noise-prone `savZ` field happened to land on. This is the precise "defending against any rows whose savZ collapsed to the variance-degenerate fallback" language from the `a18e0b9` commit message.

This defense matters more for Savage than for vdW because the vdW score function `J_VDW(u) = Phi^{-1}(u)` is bounded influence (the inverse normal is finite away from the boundaries `u = 0` and `u = 1`), whereas the Savage score function `J_SAV(u) = -log(1 - u) - 1` is **unbounded as `u -> 1`**. A single rank flip at the maximum-rank position can swing `S` substantially and inflate the apparent `savZ` long after the corresponding p-value has saturated to 1.0. For YW (axis-183) the same issue does not arise because YW reports a t-statistic with finite degrees of freedom, not a Z, and the Welch-Satterthwaite df itself shrinks toward zero in a degenerate winsorized-variance regime, propagating the failure to the p-value cleanly. For FP (axis-182) the Behrens-Fisher rank statistic is bounded by construction. So the Savage aggregator is the **first** in the family that genuinely needs the round-trip defense, and the commit shipped it.

## 3. The Acklam approximation: why round-trip via a series, not via a fixed table

The natural objection: if you are going to round-trip `pValue -> z`, why use the bundled `inverseStandardNormalCdf` (Acklam 2003 unpublished but widely deployed double-precision rational approximation, max abs error ~1.15e-9 over the central 99.9999% of the distribution and ~1e-7 in the deep tails) rather than calling out to a math library?

Two reasons that are explicit in the v0.6.467 test scope:

1. **Round-trip self-consistency with the A&S 26.2.17 upper-tail approximation already used by the base test.** The base test's `standardNormalUpperTailSavage` is Abramowitz-Stegun 1965 sec. 26.2.17 (the Hastings approximation, max abs error ~7.5e-8 in the upper tail). Acklam's inverse is the natural pair: both are pure rational/polynomial approximations with similar precision floors, and the v0.6.467 test scope explicitly includes "2 round-trip tests for the bundled inverseStandardNormalCdf Acklam approximation against the standardNormalUpperTailSavage A&S approximation" — i.e. the test asserts `Acklam(1 - AS(z)/2) ≈ z` to within the joint precision floor of both approximations. This is exactly the right test to write, because it pins the failure mode at the seam between the two halves of the round trip.
2. **No external dependency.** The base test ships its own A&S upper-tail approximation rather than depending on a stats library; the aggregator follows suit with its own Acklam inverse. This keeps the axis-184 surface self-contained — a property worth more than a few extra digits of inverse-normal precision when the dominant error budget is dominated by the rank-tie correction in the variance term anyway.

A reasonable next step (not landed in `a18e0b9`, but visible from the structure) would be a higher-precision inverse for the deep tail — Acklam's max error of ~1e-7 in `|z| > 7` is actually well below where the corpus-level Stouffer combiner cares, because for `m = 4` sources the corpus `stoufferZ` rarely exceeds `4 * 7 / sqrt(4) = 14` in absolute value, and even an `|stoufferZ| = 14` corresponds to a corpus p-value of ~1e-44, which is reported but not finely distinguished. So Acklam is "the right Pareto point" for this aggregator.

## 4. The 7-test scope as a maturity signal

Compare the test scopes across the four aggregators in the 181-182-183-184 family (numbers from the CHANGELOG and from `git show --stat`):

- axis-181 vdW base test + aggregator: 67-ish tests on the joint surface (the CHANGELOG entries pre-date the explicit aggregator-test split, so the count is bundled).
- axis-182 FP base test: 67 tests, and the FP aggregator commit `2c5e677` ("aggregateFlignerPolicelloHalves Stouffer signed corpus combiner + labelFlignerPolicelloHalvesRow 5-bucket directional classifier") adds the 5-bucket classifier in the SAME commit as the aggregator.
- axis-183 YW base test: 67 tests in commit `96488da`, then the compound classifier adds 11 more in `216c3f4`.
- axis-184 Savage base test: 37 tests in `f32c68e`, aggregator: **7 tests** in `a18e0b9`.

37-test base + 7-test aggregator is intentional. The base test scope (37) is dominated by score-function invariants (sum-to-zero across `n in {2, 5, 16, 50, 100, 500}` within `n * 1e-14`, exact harmonic identities for `n = 2, 3`, monotonicity, location/scale invariance, reverse-sign identity) that exercise the Savage-specific algebra and have no analogue in vdW / FP / YW. The aggregator scope (7) is intentionally small because the aggregator is structurally a copy of the axis-181 / 182 / 183 aggregators: empty input, unanimous positive, split sign, malformed-row skipping (NaN savZ, out-of-range pValue, non-positive tenure), and the tenure-weighted-mean response to a long-tenure heavy row. That is exactly 7 tests, and they are the 7 that you cannot share across aggregators because they exercise the per-axis row-shape contract.

## 5. The tenure-weighted mean as the "watch the long-tenure source" signal

The aggregator reports THREE summary statistics, not one:

1. `stoufferZ` — the signed Stouffer combiner (the headline number).
2. `unweightedMeanSavZ` — `sum_i savZ_i / m`.
3. `tenureWeightedMeanSavZ` — `sum_i (tenure_i * savZ_i) / sum_i tenure_i`.

The third is the new ingredient. From the live-smoke in the v0.6.467 CHANGELOG, the four joinable sources have very asymmetric tenures: `vscode-cp` at 265 days, `claude-code` at 72 days, `openclaw` and `hermes` at 18 days each. That is a 14.7x ratio between the longest and shortest tenure. With unweighted mean, each source contributes 1/4 of the weight; with tenure weighting, `vscode-cp` contributes 265/(265+72+18+18) = 71% of the weight.

The `vscode-cp` row has `savZ = -2.2373`, `claude-code` has `savZ = +3.6821`, `openclaw` has `savZ = -2.5632`, `hermes` has `savZ = +0.1264`. So:

- `unweightedMeanSavZ = (3.6821 - 2.5632 - 2.2373 + 0.1264) / 4 = -0.252` — a small negative, dominated by openclaw and vscode-cp pulling first-half-larger.
- `tenureWeightedMeanSavZ = (72*3.6821 - 18*2.5632 + 265*(-2.2373) + 18*0.1264) / (72+18+265+18)`. Numerator: `265.11 - 46.14 - 592.88 + 2.28 = -371.63`. Denominator: `373`. Result: `-0.996` — a substantially more negative tenure-weighted shift, because the long-tenure `vscode-cp` source's negative `savZ` swamps `claude-code`'s positive contribution.

This is the signal the aggregator wants to surface: **on the live data, the corpus-level direction depends on whether you weight by tenure**. The unweighted Stouffer would split sign-confused; the tenure-weighted mean recovers the long-tenure-decay narrative explicitly. A consumer of this aggregator now has both numbers and can decide which is the right summary for their downstream question (e.g. "is the second half larger across the corpus" vs "is the second half larger weighted by how much we know about each source").

This is also why the 7-test scope explicitly includes a "tenure-weighted mean's correct response to long-tenure heavy rows" test: if the implementation accidentally ignored the tenure field or applied a uniform weight, the test would fail on a synthetic row where one source has 100x the tenure of the others.

## 6. Cross-axis sign agreement: what happens when we extend `classifyLocationCompound` from 3 to 4 axes

The compound classifier from commit `216c3f4` (axis-181 + 182 + 183) currently joins three signed statistics per source and reports five buckets: `unanimous-second-larger`, `unanimous-first-larger`, `majority-second-larger`, `majority-first-larger`, `split`. The natural follow-up — not landed in `a18e0b9` but structurally implied — is the **four-axis** classifier joining axis-181 + 182 + 183 + 184.

From the v0.6.467 cross-axis live-smoke section in the CHANGELOG, all four sources are sign-unanimous across YW (axis-183) and Savage (axis-184): `claude-code` both +, `openclaw` both -, `vscode-cp` standalone Savage - (no joined YW row), `hermes` near-zero on both. Joining with the v0.6.466 compound report (where all four sources were already sign-unanimous across vdW + FP + YW), the four-axis extension would give:

- `claude-code` — vdW +, FP +, YW +, Savage +. Four-of-four positive. With Savage decisive at `p = 2.31e-4` and YW + FP both decisive (YW p = 8.98e-3, FP p = 3.32e-5), this is `unanimous-second-larger AND >=3-of-4 decisive`, the strongest possible four-axis verdict.
- `openclaw` — vdW -, FP -, YW -, Savage -. Four-of-four negative. FP decisive at `p = 9.12e-8` and Savage decisive at `p = 1.04e-2` and YW decisive at `p = 2.14e-2`: `unanimous-first-larger AND 3-of-4 decisive`. The strongest possible four-axis first-larger verdict.
- `vscode-cp` — vdW -, FP -, YW -, Savage -. Four-of-four negative. FP decisive at `p = 3.68e-2`, Savage decisive at `p = 2.53e-2`. `unanimous-first-larger AND 2-of-4 decisive`, weighted by the 265-day tenure into the dominant tenure-weighted contribution.
- `hermes` — vdW +, FP +, YW +, Savage + (sign-only, all near zero, none decisive). `unanimous-second-larger AND 0-of-4 decisive`. The "sign-coherent but underpowered" four-axis verdict, exactly what we expect on `n = 18` data.

Notably, **none of the four sources land in any `split` or `majority` bucket** in the four-axis extension. This is the cleanest possible cross-axis sign-agreement table on the live data so far. It is also a falsifiable prediction for the next live-smoke run: if axis-185 (whatever rank-based location test lands next) introduces a different score function, we should expect at least one source to flip into a `split` bucket if the orthogonality argument from the v0.6.467 CHANGELOG holds. If all five axes stay sign-unanimous on all four sources, that would suggest the four current axes are not truly orthogonal on this data and we are double-counting evidence.

## 7. What the aggregator does NOT do, and why that is the right scope

Three things the aggregator deliberately excludes:

1. **Fisher's combined p-value.** The aggregator could in principle ALSO report `chi2 = -2 * sum log(p_i)` (Fisher 1925) as a sister statistic to Stouffer Z. It does not. Reason: Fisher's combiner is sign-blind by construction (taking `log(p)` discards the direction), and the entire point of the signed Stouffer family is to preserve direction. A separate Fisher combiner would belong on a separate axis (and indeed v0.6.439 introduced one for the corpus-aggregator role, per axis-169 in the CHANGELOG).
2. **Random-effects meta-analysis.** No DerSimonian-Laird `tau^2` heterogeneity term, no random-effects re-weighting. Reason: at 4 sources per live-smoke, the heterogeneity estimator has 3 degrees of freedom and the variance of `tau^2` is enormous. Adding random-effects pretensions on this corpus size would be a precision-theatre move.
3. **Cross-axis fusion.** The aggregator combines per-source rows WITHIN axis-184, not across axes. Cross-axis fusion is the job of `classifyLocationCompound` (which currently spans 3 axes and is the natural place to extend to 4). Keeping intra-axis aggregation separate from cross-axis fusion is correct: the two operations have different statistical contracts (within-axis = sources are independent; across-axis = same sources, dependent statistics with non-trivial covariance).

These three exclusions are the discipline of staying inside the axis-184 contract. The aggregator is a small commit because the surrounding architecture has been doing the work for the past three axes.

## 8. The implication for axis-185 and beyond

The axis-184 base test + aggregator is the fourth in what is now visibly a **family of LOCATION-test axes built on the same five-step contract**: per-source halves split, signed Z-scale statistic with documented ARE against the rank-identity baseline, p-value via a documented approximation, signed Stouffer aggregator with p-value round-trip, and join into the compound directional classifier. The next axis in this family is structurally constrained: it must (a) have a documented score function with a different influence shape from vdW (bounded), FP (rank-Behrens-Fisher), YW (trimmed central), Savage (right-tail unbounded); (b) report a signed Z (or t with a documented df); (c) implement a p-value approximation with a documented precision floor; and (d) ship its aggregator as a small commit on top of a base test, defending the same variance-degenerate failure mode.

The remaining design space inside the location-test family is narrow but not empty. Plausible candidates:

- **Klotz-style location** (the Klotz score function `J_K(u) = (Phi^{-1}(u))^2` is currently used as a SCALE test in axis-177; the location version would use signed `Phi^{-1}(u)` — but that is just vdW. So Klotz is structurally taken).
- **Hodges-Lehmann pseudo-median estimator with bootstrap CI**. Not a hypothesis test per se, but a location estimator that complements the rank-test family.
- **Mood-style median test with sign-of-deviation-from-pooled-median**. Most underpowered but most robust to single-tail outliers.
- **Locally-most-powerful exponential location alternative with known scale** (Hájek-Šidák 1967 Ch. V). Strong when the scale really is known; structurally distinct from Savage by virtue of using known-scale rather than rank-based variance.

Whichever axis-185 picks, the aggregator commit will look very similar to `a18e0b9` — small, with a 7-ish-test scope, with the same p-value round-trip discipline, and with the same `tenureWeightedMean` surface to expose the long-tenure-source weighting question to the consumer rather than baking a weighting choice into the headline `stoufferZ`. That is the structural payoff of having shipped the fourth instance of this pattern: instance number five will be cheaper to ship, easier to review, and easier to defend.

## 9. The takeaway

Commit `a18e0b9` is the aggregator that closes the 181-182-183-184 location-test stack into a coherent grid. Three things are worth remembering from it:

1. **The p-value round-trip is the variance-degenerate-row defense.** It is asymmetric: it matters specifically for rank-score tests with unbounded influence at the boundary (Savage), less for bounded-influence tests (vdW), and not at all for tests whose p-value collapses in lockstep with the statistic (YW, FP).
2. **The tenure-weighted mean is the second summary statistic, and it can disagree with the unweighted mean.** On the live-smoke, the unweighted mean is `-0.252` and the tenure-weighted mean is `-0.996` — a 4x ratio, dominated by the 265-day `vscode-cp` row.
3. **The 7-test scope is the maturity signal.** A 4th aggregator in a family ships in 7 tests because the surrounding contract has done the work; a 1st aggregator in a new family would ship in 30+ tests because the contract does not exist yet.

The next axis to watch is the four-axis extension of `classifyLocationCompound`, which on the v0.6.467 live-smoke would report all four sources as sign-unanimous across all four axes — a finding that is itself falsifiable and that the next live-smoke run can either confirm or break.
