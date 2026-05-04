# pew-insights axis-169 Anderson–Darling cumulative periodogram and the v0.6.439 corpus aggregator: Fisher combined-p as the tail-weighted EDF trio completion

**date:** 2026-05-04
**source:** `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` head 1305b91, v0.6.438 / v0.6.439
**axis-id:** 169
**test-suite-delta:** +10 tests (12825 → 12835) on the v0.6.439 refinement alone

## 1. The cite, pinned

The pew-insights `CHANGELOG.md` at the current head (1305b91, v0.6.439, dated 2026-05-04) records two back-to-back releases on the same calendar day:

- **v0.6.438** — adds `axis-169: daily-token-anderson-darling-cumulative-periodogram` as the third member of the canonical empirical-distribution-function (EDF) goodness-of-fit trio for the gap-filled mean-centred daily total\_tokens series.
- **v0.6.439** — refines axis-169 with a corpus-level aggregator, exposing two new public helpers: `aggregateAndersonDarlingCumulativePeriodogram(rows)` and `chiSquaredUpperTail(x, k)`. Test suite grows by exactly 10 tests, from 12825 to 12835.

The trio table from the v0.6.438 entry is the cleanest possible statement of where the new axis sits:

| axis | norm                          | reference                  |
| ---- | ----------------------------- | -------------------------- |
| 167  | sup-norm L^∞ (Bartlett-KS)    | Bartlett 1955              |
| 168  | uniform-weight L² (Cramér–vM) | Anderson–Darling 1952      |
| 169  | tail-weighted L² (AD)         | Anderson–Darling 1952/1954 |

This post is about why that aggregator — landing the same day as the axis itself — is more interesting than the axis. The axis is a textbook drop-in. The aggregator is where the operating posture of the project shows up.

## 2. Where axis-169 sits in the test ladder

The per-source statistic is the standard tail-weighted L² discrepancy on the normalised cumulative periodogram. With the one-sided non-DC periodogram bins `P[k]` for `k = 1..K` where `K = floor(n/2)` and `K >= 4`, the cumulative profile is

```
C[j] = (sum_{k=1..j} P[k]) / (sum_{k=1..K} P[k]),  j = 1..K
```

and the AD statistic uses the tail-emphasising weight `w(t) = 1/(t (1 - t))`:

```
adA2     = (1/(K-1)) * sum_{j=1..K-1}
            (C[j] - j/K)^2 / ((j/K) * (1 - j/K))
adAStar  = (K-1) * adA2
adPValue = P(A^2 > adAStar)            (Marsaglia–Marsaglia 2004 JSS)
```

What axis-169 contributes that the previous two members of the trio do not: the weight function `w(t) = 1/(t(1-t))` blows up at both endpoints of the unit interval. Tail bins (very low frequency, very high frequency) get high leverage. Bartlett's `bD` is sup-norm and reports only the worst single bin. Cramér–von Mises is uniform L² and treats all positions equally. AD weights the tails. That ordering of leverage matters because the daily total\_tokens series in the pew corpus is exactly the kind of stream where low-frequency departure from white noise (slow drift, calendar/weekly carrier mass at the low-frequency end of the periodogram) is the realistic alternative hypothesis. Sup-norm catches the worst point. Uniform L² averages everywhere. Tail-weighted L² puts the test's leverage exactly where the realistic departure lives.

That is the cleanest justification for adding a third sister test that, on first inspection, looks redundant against axis-168. It is not redundant. It is the test that finally weighs the bins in proportion to where the realistic alternative actually lives.

## 3. Why the corpus aggregator landed the same day

The v0.6.438 entry describes axis-169 as a per-source axis. The CHANGELOG explicitly anticipates the failure mode that makes a per-source axis insufficient for dashboard use:

> Why this refinement: the per-source axis-169 test is the published deployment, but corpus-level dashboards routinely need a single number summarising "how white-noise-like is the WHOLE pew daily-token stream?". Without this helper, callers were left to either (a) re-run the test on a concatenated series — which inflates `K` and breaks the per-source-tenure asymptotic — or (b) eyeball the per-source p-value column.

Both of those failure modes are real and both are pernicious in different ways. Re-running the test on a concatenated series is the kind of thing that looks principled and is not. The per-source asymptotic distribution of `adAStar` is a function of the per-source `K` (which is roughly `nTenureDays / 2`). Concatenating two sources with tenures `T_1` and `T_2` and re-running the test produces a statistic whose null distribution is not the asymptotic distribution of either per-source statistic — the cumulative periodogram of the concatenation is dominated by the discontinuity at the join, and the spectral leakage from that discontinuity inflates low-frequency bins in a way that has nothing to do with whether either underlying series is white. The naive aggregation is worse than no aggregation.

The eyeball-the-column failure mode is more familiar but also more corrosive. It produces a number — "well, three of the five sources have p < 0.05" — that has no calibrated null. Any reviewer who has been on the receiving end of a dashboard with five p-values knows that the implicit aggregation rule is whatever the speaker wants it to be that day.

The fix in v0.6.439 is the textbook one: Fisher's combined-p (Fisher 1932; Mosteller & Fisher 1948 Am. Stat. 2(5)). Under the null that all per-source p-values are independent and uniformly distributed, the statistic

```
T = -2 * sum_i log(p_i)
```

is distributed as `Chi^2_{2m}` for `m` rows. The combined p-value is the upper-tail probability `P(Chi^2_{2m} > T)`. That is the entire mathematical content of the aggregator. The implementation choices around it are where the engineering happens.

## 4. The aggregator's defensive surface

The CHANGELOG's bullet list of book-keeping fields is, on its own, a small statement about what a corpus-level helper is for:

> Outputs:
>
> - `tenureWeightedAdAStar` — average of `adAStar` weighted by `nTenureDays - 1` (matches the natural d.o.f. of the per-source statistic under the Brownian-bridge asymptotic).
> - `fisherCombinedPValue` — Fisher's (1932) combined-p `P(Chi^2_{2m} > -2 sum_i log(p_i))`, with `p_i` clamped to `[1e-300, 1]` so that numerically-zero per-source p-values do not produce `-Infinity` contributions.
> - `totalTenureWeight`, `rowsUsed`, `rowsSkipped` — defensive book-keeping. Malformed rows (non-finite `adAStar` / `adPValue`, non-integer `nTenureDays`, `nTenureDays < 2`) are SKIPPED with a counter rather than throwing.

Three things deserve attention here.

**First, the `tenureWeightedAdAStar` weights match the per-source d.o.f.** Under the Brownian-bridge asymptotic for the per-source statistic, the natural degrees of freedom scale with `K - 1` which is roughly `nTenureDays/2 - 1`. Weighting the per-source `adAStar` by `nTenureDays - 1` is therefore a proportional weight on the right quantity — longer tenures have more bins, more bins means tighter null distribution, and the weight reflects exactly that. This is a corner the aggregator could have cut. It did not.

**Second, the `[1e-300, 1]` clamp on `p_i` is a numerical correctness fix, not a statistical fudge.** Numerically zero per-source p-values are an inevitable consequence of the Marsaglia–Marsaglia 2004 series for the AD null distribution at large `adAStar`. The series produces values that round to zero in IEEE 754 double precision well before they are mathematically zero. Without the clamp, `log(0) = -Infinity` propagates through the sum and corrupts the statistic. With the clamp, the worst that happens is that a single per-source contribution saturates at `-2 * log(1e-300) ≈ 1380.8`, which is still finite and still produces a well-defined `Chi^2_{2m}` upper-tail probability. The clamp is the right call. The alternative — propagating `-Infinity` and reporting `combinedPValue = 0` for any input where any per-source p-value rounds to zero — is the kind of "technically correct" behaviour that wastes a downstream operator's afternoon.

**Third, the malformed-row policy is SKIP-WITH-COUNTER, not THROW.** Four distinct malformed shapes are explicitly skipped in the test suite. This is the correct posture for a corpus-level helper: a single malformed row from a single source should not take down the whole dashboard, but it should also not silently disappear. The `rowsUsed` and `rowsSkipped` counters mean a downstream caller can detect "we expected 12 sources, got `rowsUsed=11, rowsSkipped=1`" and react. Throwing on the first malformed row would force every dashboard caller to wrap the helper in their own try/catch and re-implement the same defensive logic. Returning a counter pushes the policy into the helper, where it belongs.

## 5. The chi-squared upper tail helper, and why it is its own export

`chiSquaredUpperTail(x, k)` is exported as a public helper and not just inlined into the aggregator. The CHANGELOG describes it as

> published-table accurate `P(Chi^2_k > x)` via the regularised upper incomplete gamma function (Numerical Recipes 6.2 with Lentz's modified continued-fraction expansion for `x > s+1` and the power series for `x <= s+1`). Convergence to ~1e-12 absolute in <= 100 iterations across the operating range (`k = 2m` for `m` rows; typically `k <= 200`). Uses a self-contained Lanczos `logGamma` (g=7, 9 coefficients) — no external dependency.

The two-branch strategy (continued fraction for `x > s+1`, power series for `x <= s+1`, where `s = k/2`) is the standard switch point in Numerical Recipes §6.2 for the regularised incomplete gamma. It avoids the slow-convergence regime of each method by handing off at the crossover. The Lentz modified continued-fraction algorithm is the standard accelerator. The Lanczos `logGamma` with `g=7` and 9 coefficients is the standard high-precision approximation that is good to about 15 digits across the positive real axis.

What is interesting is the export. By exposing `chiSquaredUpperTail` as its own public helper rather than inlining it in `aggregateAndersonDarlingCumulativePeriodogram`, the v0.6.439 release is implicitly committing to the helper as a reusable primitive. Any future axis that needs a chi-squared tail probability — and there will be many, because chi-squared upper tails are the asymptotic null distribution of every quadratic-form statistic on a Gaussian field — gets a vetted, dependency-free implementation off the shelf. The 10-test count budget for the refinement reflects that: four published Stephens 1974 critical values pinned to 1e-3, plus monotonicity, plus range `[0,1]` across `k ∈ {1, 2, 4, 10, 50}` and `x ∈ [0, 100]`. That is the test surface you build for a helper you intend to call from other places, not for a helper you intend to use exactly once.

The "self-contained, no external dependency" framing is the same posture. A 9-coefficient Lanczos is small enough to inline, fast enough at the precision required, and entirely free of the supply-chain surface that an external `gamma`/`gammainc` package would introduce. For a project that is going to publish dozens of statistical axes against a single daily-token series, owning the chi-squared tail outright is the correct trade.

## 6. The two-row Fisher closed-form pin

The CHANGELOG calls out one specific test: "two-row Fisher-combined-p closed-form pinned to ~0.5963 (matches R `pchisq(4*log(2), 4, lower.tail=FALSE)`)". This is a particularly honest pin. With two rows and per-source p-values both equal to 0.5, the Fisher statistic is `T = -2 * (log(0.5) + log(0.5)) = -2 * (-2 log 2) = 4 log 2 ≈ 2.7726`. The combined p-value is `P(Chi^2_4 > 4 log 2)`. The R reference is `pchisq(4*log(2), 4, lower.tail=FALSE)` which evaluates to approximately 0.5963.

That a corpus-level Fisher aggregation of two sources that are individually exactly at the median produces a combined p-value of 0.5963 — slightly above 0.5, not exactly 0.5 — is the kind of thing a reviewer wants to see pinned. It catches both implementation bugs (a sign error would produce 0.4037) and conceptual confusions (Fisher's combined-p is not "the average p-value"; the combined-p of two median p-values is not 0.5). Anyone reading the test suite who is unsure whether the implementation matches the textbook Fisher rule can run `R: pchisq(4*log(2), 4, lower.tail=FALSE)` and check. That is what reproducibility looks like at the test-pin level.

## 7. The trio is now closed, and what that means for axis-170+

With v0.6.438 and v0.6.439 the EDF goodness-of-fit trio for the cumulative-periodogram-against-uniform null is complete: sup-norm (axis-167 Bartlett 1955), uniform L² (axis-168 Cramér–von Mises 1952), tail-weighted L² (axis-169 Anderson–Darling 1952/1954). Three norms, three published references, three sister tests. The trio is closed in the strong sense that the only other commonly-cited norm in the EDF GoF literature is the Watson U² statistic, which is the Cramér–von Mises analogue for circular/directional data and does not apply here.

What that closure means operationally: subsequent axes (170, 171, ...) cannot be "another EDF GoF norm on the cumulative periodogram". They have to either change the test object (e.g., test some other transform of the daily-token series), change the null hypothesis (e.g., test against a non-uniform null, such as an AR(1) prewhitening residual), or change the leverage profile in some way that is not a third smooth weight on the unit interval. The combinatorics of where to go next have collapsed somewhat — and that is a good sign, because it means the project is exhausting the textbook surface in a particular subfield rather than wandering through it.

The aggregator landing on the same day as the axis is the mature posture: the per-source axis is the published deployment, the corpus aggregator is the dashboard primitive that prevents the per-source axis from being misused by downstream callers. Both are tested. Both are pinned. Both ship in the same release window. The 12835-test floor is the budget that buys the discipline. That is what the v0.6.438 → v0.6.439 cadence is selling.

## 8. What this implies for review of subsequent statistical axes

Three review heuristics fall out of the v0.6.438 → v0.6.439 pair:

1. **A per-source axis without a corpus-level aggregator is a half-axis.** If the next per-source axis lands without an aggregator helper in the same release window, the right review question is "what is the dashboard caller supposed to do with `m` per-source numbers?". Either there is a Fisher-style independence-respecting aggregation, in which case it should be exposed as a helper, or there is not, in which case the missing aggregator is the next ticket.

2. **Defensive book-keeping (`rowsUsed`/`rowsSkipped`) is the marker of a helper that expects to be called from a dashboard, not a notebook.** Notebook callers can afford to throw on malformed input because the human is right there. Dashboard callers cannot. The presence of the counter pair is a signal that the project has internalised the dashboard use case.

3. **Closed-form pins against an external reference (R, scipy, Mathematica) are the cheapest reviewer-facing reproducibility signal.** The "matches R `pchisq(4*log(2), 4, lower.tail=FALSE)`" comment is two lines of test code and roughly four hours of avoided downstream debugging the first time someone questions the implementation. Future axes should aim for at least one such pin per public helper.

The v0.6.439 release passes all three. The cumulative test count of 12835 is the floor for what that level of discipline costs. If the next refinement lands without all three markers, that is the place to push back.
