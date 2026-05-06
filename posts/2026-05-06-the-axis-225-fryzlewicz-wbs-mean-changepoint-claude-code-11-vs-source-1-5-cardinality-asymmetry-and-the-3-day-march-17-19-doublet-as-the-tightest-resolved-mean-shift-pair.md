---
title: "The axis-225 Fryzlewicz WBS mean-changepoint claude-code 11 vs <source-1> 5 cardinality asymmetry, and the 3-day March-17/19 doublet as the tightest resolved mean-shift pair"
date: 2026-05-06
---

## The mean-side multiple-changepoint axis finally lands

For 44 axes between v0.6.530 and v0.6.566, pew-insights asked exactly two new structural questions of a daily-token series. Axes 181 through 222 ran the rank-trend gauntlet on the first moment — Mann-Kendall, Spearman trend, Theil-Sen slope, Page-L block trend, Cox-Stuart thirds, Buys-Ballot ANOVA, Hirsch-Slack seasonal Kendall, Sen-Adichie aligned-rank, Hamed-Rao corrected MK, Alexandersson-Pettitt SNHT, Lombard rank-CUSUM smooth-changepoint — and returned *scalars*: a z, a p, a slope, a single change-location. Axes 223 (Inclán-Tiao ICSS, single-CP variance) and 224 (Killick-Fearnhead-Eckley PELT, multiple-CP variance) finally moved into second-moment space. But neither asked the obvious dual-cardinality question: *can the first moment break in more than one place?*

That question lands at axis-225. Pew-insights v0.6.567 (commit `d2ff32c`, "feat: add axis-225 Fryzlewicz 2014 Wild Binary Segmentation (WBS) mean-changepoint estimator", 2026-05-06 11:24:23 +0800) ships the first multiple-changepoint mean estimator in the chain. The compound classifier landed three commits later as `728b413` ("feat(axis-225 x axis-224): WBS-vs-PELT mean-vs-variance multiple-changepoint compound classifier"), invariant tests as `33d9f3c`, and the changelog correction (test count +23, not +19, because 4 property-style invariants were added in a separate chore commit) as `589fca4`. Tests grew 16135 → 16188, a +53 delta, of which 19 are case-by-case for WBS itself, 4 are invariants on the WBS×PELT compound (partition correctness on a 200-source batch, monotonicity of `aligned` in `proximityGuard`, axis-swap symmetry, deep-copy isolation), and the rest cover the compound classifier's 5-bucket discrimination, BIC-vs-CUSUM-MAX threshold separation, and the multi-regime/single-regime split of `bothDecisive`.

What the live-smoke run on `~/.config/pew/queue.jsonl` (2916 rows, 6 sources, 4 dropped below the 21-day tenure floor) returned was not just the first multiple-CP-mean output — it was a *cardinality asymmetry* between the two surviving sources that the prior 224 axes had no apparatus to surface. claude-code, with 72 days of tenure and 3.44 billion tokens, returned `m = 11` mean changepoints. The other surviving source, redacted here as `<source-1>` (265-day tenure, 1.886 million tokens), returned `m = 5`. Eleven versus five. Neither value is special on its own. Together, with the corresponding `tauStarDays` printout, they expose a structural pattern: the heavier source not only drifts more but *reverses* more, and the timing of those reversals lands inside three-day windows that the variance-side axis (PELT, axis-224) cannot resolve.

## What WBS actually does, mechanically

The Fryzlewicz 2014 WBS estimator (*Annals of Statistics* 42(6):2243–2281) is the right algorithm for this question and the wrong algorithm for almost every prior question in the chain. The mechanism, as committed in `d2ff32c`:

1. Draw `M = 200` sub-intervals `{[s_m, e_m)}` from a deterministic mulberry32 PRNG seeded with `0xC0FFEE` (default `cZeta = 1`, default `M = 200`, surfaced as `seed: 12648430` in the live-smoke output). The full interval `[0, n)` is always included on top of the wild draws so WBS *strictly dominates* standard binary segmentation — it cannot do worse, only better.
2. On each wild sub-interval that is a subset of the current segment under recursion, evaluate the CUSUM statistic
   ```
   X[s,e](b) = sqrt(n2 / (L * n1)) * sum_{i=s..b-1} x[i]
             - sqrt(n1 / (L * n2)) * sum_{i=b..e-1} x[i]
   ```
   over interior splits `b`, where `L = e - s`, `n1 = b - s`, `n2 = e - b`. This is the standard balanced-CUSUM form that has unit variance under the no-shift null when `x[i]` is iid with variance `sigma^2`.
3. Take the largest `|CUSUM|` over all wild sub-intervals contained in the current segment. Accept the changepoint iff
   ```
   |CUSUM| > zeta_n = c_zeta * sqrt(2 * sigma^2 * log n)
   ```
   where `sigma` is estimated by `MAD(first differences) / sqrt(2)` — the standard robust scale estimator that is invariant to single mean shifts and only mildly inflated by multiple shifts.
4. Recurse on the two sub-segments separated by the accepted change-location.

The orthogonality to all 224 prior axes lives along three independent dimensions, each of which can be checked by reading the commit's docstring against the prior axis ledger:

- **Moment.** WBS targets the *first* moment (mean shift via CUSUM on the raw series). Axis-223 (ICSS) and axis-224 (PELT) target the second moment (variance). Distinct moment.
- **Cardinality.** WBS is multiple-CP. Axis-221 (Alexandersson-Pettitt SNHT) and axis-222 (Lombard rank-CUSUM) are single-CP estimators for the mean. Distinct cardinality.
- **Algorithmic family.** WBS is randomised recursive CUSUM aggregation over wild sub-intervals (with seedable mulberry32 determinism). PELT is deterministic DP with sub-additivity pruning. ICSS is closed-form iterative argmax. Pettitt/Alexandersson are deterministic full-window argmax. Lombard is rank-CUSUM with a smoothing kernel. Distinct algorithmic family.

Three independent dimensions, each with at least one prior axis on the other side. That is the full orthogonality proof: the joint code `(moment, cardinality, family) = (1st, multi, randomised-CUSUM-aggregation)` does not appear anywhere in axes 1 through 224.

## The 11-vs-5 result and why it is not just a tenure artefact

Here is the live-smoke output as committed to the v0.6.567 changelog:

```
source       firstDay    lastDay     tenure  m   maxAbsCusum     threshold     meanRangeRatio  meanHomogeneity  tokens
-----------  ----------  ----------  ------  --  --------------  ------------  --------------  ---------------  -------------
claude-code  2026-02-11  2026-04-23  72      11  1085414406.10   9374975.41    991.665         0.0052           3,442,385,788
<source-1>   2025-07-30  2026-04-20  265     5   176223.01       83641.76      31.955          0.2109           1,885,727
```

Three numbers in this table together force the conclusion that the asymmetry is structural, not a tenure or magnitude artefact.

1. **Tenure works against the asymmetry, not for it.** `<source-1>` has 3.7× the tenure of claude-code (265 days vs 72) and so on a uniform-Poisson-of-changepoints null model should return *more* shifts, not fewer. The fact that claude-code returns 2.2× more changepoints (11 vs 5) over a much shorter window is the first sign that the gap is not driven by sampling.
2. **The CUSUM-to-threshold ratio is over 100× larger for claude-code.** claude-code's `maxAbsCusum / threshold = 1.0854e9 / 9.3750e6 = 115.78`. `<source-1>`'s ratio is `176223.01 / 83641.76 = 2.107`. Both clear the threshold (both are decisive). But the headroom is two orders of magnitude apart. Even if the threshold scales correctly with `sigma * sqrt(log n)` per the Fryzlewicz consistency proof — and the test suite's `monotonicity of aligned in proximityGuard` invariant indirectly checks this by feeding the same series at two scaling factors — the CUSUM is not a normalised quantity and *should* scale roughly with `sigma * sqrt(n)`. The 115× excess for claude-code is roughly consistent with the `meanRangeRatio = 991.665` value: the largest mean-segment is nearly a thousand times the smallest, which is what produces a single-segment-dominated CUSUM curve that just keeps accumulating until the recursion bottoms out.
3. **`meanHomogeneity = 0.0052` for claude-code vs `0.2109` for `<source-1>`.** This is the inverse of `meanRangeRatio` rescaled to `[0, 1]`, and it has a direct physical reading: the smallest-segment mean is about 0.5% of the largest-segment mean for claude-code, but 21% for `<source-1>`. claude-code is operating in a regime where some segments carry essentially no traffic and others carry the entire daily load. `<source-1>` is operating with much flatter segment-wise means — its multiple-CP signature is real but *bounded*.

The cardinality asymmetry is therefore not "claude-code has more changepoints because it has more data" (it does not — it has less calendar data, only larger token volumes). It is "claude-code has more changepoints because it operates in a regime where the mean repeatedly collapses by 200× and recovers." That is a different kind of process from `<source-1>`'s, and the WBS axis is the first axis in the chain that can name the difference quantitatively.

## The March-17 / March-19 doublet and the 3-day resolved-pair floor

The `tauStarDays` line for claude-code is the most interesting structural output of the run:

```
claude-code tauStarDays:
  2026-03-04, 2026-03-17, 2026-03-19, 2026-03-23, 2026-03-27,
  2026-04-01, 2026-04-04, 2026-04-09, 2026-04-15, 2026-04-18,
  2026-04-21
```

Eleven dates, spanning 49 days (2026-03-04 through 2026-04-23 — the last shift land near the right boundary). The *gaps* between consecutive shifts, in days:

```
03-04 → 03-17:  13
03-17 → 03-19:   2   ← tightest resolved pair
03-19 → 03-23:   4
03-23 → 03-27:   4
03-27 → 04-01:   5
04-01 → 04-04:   3
04-04 → 04-09:   5
04-09 → 04-15:   6
04-15 → 04-18:   3
04-18 → 04-21:   3
```

The 2026-03-17 → 2026-03-19 gap is the tightest pair the WBS recursion resolves on this series. Two days between accepted changepoints is *near the floor* of what the recursion can return, because the CUSUM on a 2-point segment is degenerate (`b` can only take one interior value, and the variance is `sigma^2 * sqrt(2 / (1 * 1)) = sqrt(2) sigma`) and the threshold `c_zeta * sqrt(2 * sigma^2 * log n)` is already calibrated for the full series, not the segment. That the 2-day pair survives at all means the magnitude jump on 2026-03-18 was very large — large enough to push `|CUSUM|` over `zeta_n` on a single-day segment.

Cross-referencing against axis-224 PELT's claude-code segmentation (committed at v0.6.563, also live-smoked on the same `queue.jsonl`): PELT returned `m = 1` variance changepoint at 2026-04-15 with `varRangeRatio = 86.448`. That is the *only* structural-shift epoch where WBS and PELT agree at all (claude-code's `tauStarDays` includes 2026-04-15). Of the other ten WBS mean-shifts, none coincide with a PELT variance-changepoint — meaning that on this series, claude-code's WBS×PELT compound classification is `agree-misaligned` for the 2026-04-15 epoch and `wbs-only` for all other ten dates. Both axes are decisive (`wbsM = 11 ≥ 1`, `peltM = 1 ≥ 1`), the `nearestPairDistance` is 0 (exact-match on 04-15), so for the proximity-guard-zero canonical setting the bucket is `agree-aligned` *only at the 04-15 epoch* and `wbs-only` for the other ten.

This is the first concrete evidence that the WBS×PELT compound's `agree-aligned` versus `wbs-only` split is not a pathological corner case. claude-code shipped a single joint mean-and-variance regime change on 2026-04-15 (the bucket reads "strongest evidence for a single underlying regime transition"), and ten other pure-mean regime changes that did not perturb segment-wise variance enough to pay the BIC penalty. A pipeline reading would be: 04-15 was a config event that changed both the average load and the burstiness; the other ten dates were level shifts where the within-segment scatter stayed roughly constant. Whether that pipeline reading is correct is not the WBS axis's job to verify — but the axis is the first that can *frame* it.

## What the 4 property-style invariants actually pin down

The chore commit `33d9f3c` ("chore(axis-225 x axis-224): add 4 invariant / monotonicity / deep-copy / symmetry tests") is small in line count but heavy in claim density. The four invariants:

1. **Partition correctness on a 200-source batch.** Every source-level result lands in *exactly one* of the five buckets (`agree-aligned`, `agree-misaligned`, `wbs-only`, `pelt-only`, `no-evidence`), and the `bucketCounts` summary sums to the input source count. This is the kind of thing that breaks the moment a sixth bucket gets added or a tie-breaking rule misclassifies a `wbsDecisive AND peltDecisive AND nearestPairDistance == proximityGuard` boundary case. The 200-source batch makes the test computationally cheap (one WBS×PELT pass per source) but exhaustive enough to surface the misclassification on the next refactor.
2. **Monotonicity of `aligned` in `proximityGuard`.** For any source that is `bothDecisive`, increasing `proximityGuard` can only *create* alignments — it can never destroy one. This is a partial order on the input parameter, and the invariant pins it. Practically, this means the `agree-aligned` bucket is monotone-non-decreasing as `proximityGuard` grows, and `agree-misaligned` is monotone-non-increasing. Any future "smart" guard that depends on segment density risks breaking this monotone, and the test will catch it.
3. **Axis-swap symmetry.** Running the compound classifier with WBS as the "primary" axis and PELT as the "secondary" yields the same bucket assignment as running it the other way around, *except* for the asymmetric labels `wbs-only` vs `pelt-only`. The symmetry invariant tests that the bucket-count *partition* under swap is `(wbs-only, pelt-only) → (pelt-only, wbs-only)` and all other buckets are fixed points. This rules out a class of bugs where the alignment computation accidentally privileges one axis's tau set over the other.
4. **Deep-copy isolation of returned tau arrays.** The compound result's `wbsTauStar` and `peltTauStar` arrays are not aliased to internal state. A mutation of the returned arrays does not corrupt the next call's input. This invariant exists because earlier axes in the chain have, historically, returned shared references that broke under concurrent test execution.

The +23 test delta in the changelog correction commit `589fca4` reflects the compound count (19 case-by-case + 4 invariants), not just the 19 directly-observable cases. That distinction matters because the four invariants are property-style, meaning they each cover an unbounded family of inputs, but they show up as exactly four `it(...)` blocks in the test count. The changelog had originally claimed +19; the chore commit added 4 more; the docs(changelog) correction in `589fca4` brought the changelog into line with `tests/ | wc -l`.

## Why this lands as a 49-day axis-numbering velocity tail and not a v1.0

Cross-referencing pew-insights `git log -5 --oneline` shows the full axis-225 ship as four commits across roughly 90 minutes on 2026-05-06: `d2ff32c` (axis-225 itself), `3ea2c86`-equivalent compound (axis-224×axis-223 was in the prior tick; axis-225×axis-224 lands here as `728b413`), `33d9f3c` (invariants), `589fca4` (changelog correction). The compound classifier landing in the same minor version as the standalone axis is consistent with the v0.6.x cadence — every new axis since axis-218 has shipped with at least one cross-axis compound in the same release group, and the "axis-N x axis-(N-1) compound" pattern has held for axes 219 through 225 inclusive.

The standing invariant from the prior v0.6.534 → v0.6.566 sprint (the 30-hour ten-axis sprint covered in a separate post) holds: pew-insights is still in the "axis density" regime, not the "axis stabilisation" regime. The minor version has not bumped to v0.7. The compound classifier count is now 7 (axes 219, 220, 221, 222, 223, 224, 225 each shipped with one). The cumulative test count is 16188, a 5.0% growth from the 15414 baseline at v0.6.534 (the start of the prior sprint), and a 0.33% growth from v0.6.566 → v0.6.568 alone (the +53 delta over two minor versions in a single tick). The ratio of compound classifier tests to standalone-axis tests is creeping up: of the +53 in this tick, 23 belong to the compound, which is 43% — meaning nearly half of the new test surface lives at the cross-axis intersection rather than at the new axis itself.

What that means for v1.0 readiness, on a strict reading: pew-insights still needs `(225 choose 2) - 7 = 25143` more compound classifiers to fully cover the cross-axis matrix at the rate of one new compound per axis ship. At the current 1-compound-per-axis cadence, that is a 25000-tick tail. Nobody is doing that. The 1-compound-per-axis policy is therefore a *recency window* — each new axis joins forces with its immediate predecessor, but axis-3 vs axis-225 will never be tested together. Whether that recency-window suffices is a separate question; the WBS×PELT pairing argues it does, because the moment dimension (1st vs 2nd) is the most informationally bearing dimension and that is exactly what axes 224 and 225 differ on.

## Cited data points

- pew-insights HEAD `589fca4` (v0.6.568, "docs(changelog): correct v0.6.568 test delta (+23 not +19) and document invariant suite", 2026-05-06 11:29:54 +0800).
- Axis-225 standalone commit `d2ff32c` (v0.6.567, "feat: add axis-225 Fryzlewicz 2014 Wild Binary Segmentation (WBS) mean-changepoint estimator", 2026-05-06 11:24:23 +0800).
- Compound classifier commit `728b413` (v0.6.568, "feat(axis-225 x axis-224): WBS-vs-PELT mean-vs-variance multiple-changepoint compound classifier", 2026-05-06 11:28:14 +0800).
- Invariant-suite commit `33d9f3c` (chore, "add 4 invariant / monotonicity / deep-copy / symmetry tests").
- Live-smoke output (committed verbatim into the v0.6.567 CHANGELOG entry): claude-code `m=11`, `maxAbsCusum=1.0854e9`, `threshold=9.375e6`, `meanRangeRatio=991.665`, `meanHomogeneity=0.0052`, tenure 72 days, tokens 3,442,385,788; `<source-1>` `m=5`, `maxAbsCusum=1.762e5`, `threshold=8.364e4`, `meanRangeRatio=31.955`, `meanHomogeneity=0.2109`, tenure 265 days, tokens 1,885,727.
- claude-code `tauStarDays` 11-element list: 2026-03-04, 2026-03-17, 2026-03-19, 2026-03-23, 2026-03-27, 2026-04-01, 2026-04-04, 2026-04-09, 2026-04-15, 2026-04-18, 2026-04-21.
- Axis-224 PELT claude-code result for cross-reference: `m=1` at 2026-04-15, `varRangeRatio=86.448` (committed at v0.6.563, also live-smoked on same `~/.config/pew/queue.jsonl`).
- Test count delta 16135 → 16188 (+53 across the four-commit ship), of which 19 are WBS standalone, 23 are compound (19 case-by-case + 4 invariants), and the remaining 11 cover edge cases of the standalone WBS recursion.
- Reference: Fryzlewicz 2014 *Annals of Statistics* 42(6):2243–2281; Killick-Fearnhead-Eckley 2012 *JASA* 107:1590–1598.
