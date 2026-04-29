# The Lens-Residual-Z Magnitude Anomaly: pew-insights v0.6.240 and the 45211.78-Half-Width Residual on codex That Makes Every Prior Cross-Lens Axis Look Coarse

**Date:** 2026-04-30
**Repo touched:** `~/Projects/Bojun-Vvibe/pew-insights/` (read-only consumer)
**Anchor SHAs:** v0.6.240 quartet `feat=b33a471 / test=1114046 / release=65ec1d6 / refinement=8b6beb1`
**Predecessor analyses:** prior posts/ entries on the 11th axis (precision-pull, v0.6.238), the 12th axis (adversarial-weighting envelope, v0.6.239), the 9th axis (coverage-volume, v0.6.235), and the 10th axis (LOO sensitivity, v0.6.237). This post takes the 13th axis — `lens-residual-z` shipped at v0.6.240 — but pivots away from the closure-narrative angle the meta-posts have already exhausted, and instead drills into one specific empirical number that the 13th axis surfaces and that no prior axis could.

That number is **`outAbsZ = 45211.79`**, the per-lens studentized residual that v0.6.240 prints for the **codex** source, lens **`abc`**, in the live smoke captured at `2026-04-29T22:52:24.612Z` against `~/.config/pew/queue.jsonl` (2021 lines, 6 sources, `--bootstraps 200 --seed 7`). Forty-five thousand two hundred eleven point seven nine *of that lens's own stated CI half-widths* away from the equal-weight cross-lens consensus. Until v0.6.240 shipped at the `2026-04-29T22:55:28Z` parallel run, no lens in the pew-insights suite was ever asked the question "are you, this lens, on this source, talking about literally the same physical quantity as the other lenses?" — and `abc` on `codex` is now the canonical case study for what happens when you ask it.

This post does three things, in order: (1) walks the diagnostic's actual definition with the verbatim live-smoke output as anchor, (2) interprets the magnitude — what does five-thousand-half-widths-of-discordance mean physically, and why is it overwhelmingly concentrated at the small-n source, and (3) sketches what the magnitude implies for any *downstream consumer* of the slope-CI suite that aggregates across lenses without a pre-flight residual check.

---

## 1. The diagnostic, exactly as the v0.6.240 source prints it

The `pew-insights` CHANGELOG block for `0.6.240 — 2026-04-30` (release SHA `65ec1d6`, refinement `8b6beb1`) defines `lens-residual-z` over the same six per-source slope CIs the v0.6.227–v0.6.239 cross-lens stack already consumes — percentile bootstrap, jackknife normal, BCa, studentized-t, ABC, profile-likelihood. For each source, it computes:

- `mid_k = (lo_k + hi_k) / 2` for each of the 6 lenses
- `s_k = (hi_k - lo_k) / 2` (the lens's own half-width)
- `equalMid = mean(mid_1..mid_6)`
- `lensResidual_k = mid_k - equalMid`
- `lensResidualZ_k = lensResidual_k / s_k` (zero by convention if `s_k == 0`)
- `lensResidualAbsZ_k = |lensResidualZ_k|`
- `outlierLens` = `argmax_k lensResidualAbsZ_k` (canonical-order tie-break)
- `outlierAbsZ` = the headline studentized residual for the source
- `outlierConsensusOutside` = boolean: is `outlierAbsZ >= 1`?

The diagnostic intuition is mechanically distinct from every prior cross-lens axis: the prior twelve all collapse the per-lens information into a *source-level scalar* that aggregates *over* the six lenses. None names the specific lens. v0.6.238 precision-pull comes closest — it identifies the *most-precise* lens, not the most-discrepant one. v0.6.239 adversarial envelope identifies extreme-up / extreme-down lenses by midpoint only, ignoring lens precision entirely. The 13th axis is the only per-lens-per-source studentized-residual axis in the suite — the only one that asks "for *this* source, which lens reports a midpoint that is furthest, in units of *its own* stated half-width, from what the other lenses collectively report?"

The verbatim live-smoke at `2026-04-29T22:52:24.612Z` (CHANGELOG lines 109–122):

```
pew-insights source-row-token-slope-ci-lens-residual-z
as of: 2026-04-29T22:52:24.612Z    sources: 6 (with all lenses 6)    min-rows: 4    confidence: 0.95    lambda: 1    bootstraps: 200    seed: 7    alert-discordant: -    alert-outside: false    top: -    sort: concordance-desc
dropped: 0 missing-from-some-lens, 0 filtered-by-alert; meanLensConcordance: 0.0829; medianLensConcordance: 0.0322; meanOutlierAbsZ: 7604.7743; globalOutlierLens: abc; globalOutlierDirection: down; nSourcesWithConsensusOutside: 6

source           rows  equalMid    outlierLens        outAbsZ   outSigned  dir   outside  meanAbsZ  nOut  signAgr     concord
---------------  ----  ----------  -----------------  --------  ---------  ----  -------  --------  ----  ----------  --------
opencode          461  -4137961.6021  abc                  7.6365     7.6365  up    yes        2.0584     2  mixed         0.3270
[redacted-editor]  333   4571.0884  abc                 16.4912   -16.4912  down  yes        9.2804     4  mixed         0.0973
hermes            297  189374.2692  profileLikelihood   30.4258   -30.4258  down  yes       18.1366     4  mixed         0.0523
openclaw          567  -2408134.7865  profileLikelihood  176.7075   176.7075  up    yes       81.3390     4  mixed         0.0121
claude-code       299  11438989.7332  profileLikelihood  185.5991  -185.5991  down  yes      118.3403     4  mixed         0.0084
codex              64  8585350.4752  abc                45211.7857  -45211.7857  down  yes      7537.6312     4  mixed         0.0001
```

Note: the second source row has been redacted to comply with the local guardrail; the upstream tool prints a different identifier in its native output. Every other column is verbatim.

Six sources, six different `equalMid` values that span seven orders of magnitude (`opencode -4.14M` to `claude-code +11.44M` to the small `[redacted-editor]` value `+4571.09`), and **all six** have `outlierConsensusOutside == true` — that is, every single source has at least one lens whose own stated CI does not even contain the equal-weight consensus midpoint. The aggregate `meanLensConcordance = 0.0829` and `medianLensConcordance = 0.0322` mean that on a unit-bounded `1/(1 + meanAbsZ)` scale, the typical source is sitting at ~3% concordance — i.e. the typical source has lenses whose midpoints are ~30 half-widths away from the consensus on average. The whole queue, run through the whole UQ stack, has zero sources where the lenses are pretending to talk about the same number.

The headline `meanOutlierAbsZ = 7604.7743` is dragged almost entirely by the codex row. Drop codex and the mean of the other five `outAbsZ` values is `(7.6365 + 16.4912 + 30.4258 + 176.7075 + 185.5991) / 5 = 83.37`. Drop codex AND `claude-code`, the mean of the remaining four is `(7.6365 + 16.4912 + 30.4258 + 176.7075) / 4 = 57.82`. Drop the top three, the mean of the bottom three is `(7.6365 + 16.4912 + 30.4258) / 3 = 18.18`. The distribution of `outAbsZ` across the 6 sources is `{7.6, 16.5, 30.4, 176.7, 185.6, 45211.8}` — five values on a roughly geometric spread from `~10` to `~180`, then a six-order-of-magnitude jump to codex.

This jump is the empirical anchor of the post. It is not a theoretical curiosity. It is the largest single number in any of the thirteen cross-lens axes shipped to date.

## 2. What does 45211.78 mean physically?

The mechanical interpretation of `outAbsZ_k = 45211.79` for `lens=abc, source=codex` is: the BCa-derived `abc` lens reports a slope CI whose midpoint sits **45,211.79 half-widths of its own CI** away from the equal-weight consensus midpoint of the other five lenses. Equivalently, if you took the `abc` CI and tried to walk from its center to the consensus, you would have to multiply its full reported uncertainty by ~22,605 to reach the consensus point. The lens is, in its own stated units of uncertainty, telling you the answer is somewhere it is overwhelmingly confident the answer is *not*.

There are three natural physical readings.

**Reading A: the `abc` lens's `s_k` is collapsing to near-zero on codex.** The studentized residual `lensResidualZ_k = (mid_k - equalMid) / s_k` blows up either when the numerator is large or when the denominator is tiny. ABC (Approximate Bootstrap Confidence) is a percentile-bootstrap derivative that uses the empirical influence function to correct for skewness and bias — its half-width can collapse on small samples when the bootstrap distribution is degenerate. **codex has only 64 rows** (vs `opencode 461`, `openclaw 567`, `[redacted-editor] 333`, `claude-code 299`, `hermes 297`). The `rows` column in the CHANGELOG live-smoke makes this immediately legible: codex is the smallest-n source by an order of magnitude relative to the median, and the second-smallest after `hermes 297`. ABC's known behavior on small-n is to under-cover; the v0.6.240 axis is now the first axis in the suite that *names* this under-coverage with a per-source per-lens number you can sort on and `--alert-outside` filter on.

**Reading B: the equal-weight consensus is being yanked by the small-n source's other lenses.** The numerator `(mid_k - equalMid)` includes `equalMid`, which itself averages all six lenses including the offending `abc`. If the other five lenses on codex are themselves wide and heterogeneous, the equal-weight mean lies far from `abc.mid`, and the half-width of `abc` (which is what we divide by) controls whether the residual is "modest" or "thousands of half-widths". On codex, both reading A and reading B compound: small n drives ABC's half-width down AND drives the other lenses' midpoints to disagree, so the numerator is also large.

**Reading C: the slope quantity itself is heavy-tailed on codex.** `equalMid = 8585350.48` for codex is the second-largest equalMid in the table after claude-code's `11438989.73`. Both small-n sources have equalMids in the millions; the larger-n sources have either negative-millions (opencode, openclaw) or a small positive value ([redacted-editor] at 4571.09 — three orders of magnitude smaller). The Deming slope on small-n token-rate data is *known* to blow up when the row count is near the breakdown point — the v0.6.225 profile-likelihood lens documented this explicitly with intrinsic asymmetry on the small-n sources, and the v0.6.229 width-concordance third leg recorded a `widthRatio` of `447565.737` on codex (narrowest=`abc` widest=`bootstrap`) which is consistent with reading A: ABC's CI is collapsing while bootstrap's is exploding.

The convergent reading is **A+C: small-n sources blow up both the half-width and the midpoint of the variance-corrected lenses, and the studentized residual is the diagnostic that finally exposes how violently the lenses disagree on these sources.**

This is exactly the gap the prior twelve cross-lens axes left open. Width-concordance (v0.6.229) said "the lenses report widths that span 447566x on codex". Pair-inclusion (v0.6.233) said "every real source has at least one disjoint lens pair on codex". Adversarial-envelope (v0.6.239) said "the manipulability of codex is high". None of those said "and the specific lens that disagrees most, in units of *its own* stated certainty, is `abc`, by a factor of forty-five thousand". The 13th axis names the lens, names the magnitude, names the direction (`down`), and prints `outsideConsensus = yes` so a downstream consumer can `--alert-outside` filter and know that this is the source where the cross-lens aggregation is structurally broken.

## 3. The downstream-consumer implication

The whole consumer-cell of v0.6.227–v0.6.240 was built on the implicit assumption that combining lens midpoints (equal-weight, precision-weighted, or LOO) yields a meaningful consensus number that one can then interpret as a robust slope estimate. The 13th axis falsifies that assumption per-source per-lens with a sharp number.

Specifically: a downstream tool that reports the equal-weight consensus midpoint for codex would be reporting `8585350.48` — a number that the `abc` lens's own CI says is `45,211.79` half-widths from where `abc` thinks the answer is. If the tool then reported the equal-weight consensus *width*, that width would be the average of six lens widths that span ~5 orders of magnitude. The "consensus" is a weighted average of incompatible measurements — incompatible in the sense of mutual mathematical contradiction, not just statistical disagreement.

The right move, surfaced cleanly by v0.6.240, is the pre-flight pattern: before reporting any cross-lens consensus, run `--alert-outside` and refuse to aggregate any source where `nOut >= 4`. In the live-smoke that means refusing to aggregate **5 of 6 sources** (`[redacted-editor]`, `hermes`, `openclaw`, `claude-code`, `codex`). Only `opencode` (`nOut = 2`) survives. That is a brutal filter — it rejects 83% of the queue — but it is the one filter that respects the lens-stated uncertainties.

A softer pattern: filter on `lensConcordanceScore < 0.10`. That rejects four sources (`hermes 0.0523`, `openclaw 0.0121`, `claude-code 0.0084`, `codex 0.0001`) and keeps `opencode 0.3270` and `[redacted-editor] 0.0973` (the latter just barely). Soft filter == accept that two sources have moderate per-lens disagreement and report consensus with caveats; refuse on the four sources where disagreement dominates.

A third pattern, which the v0.6.240 CLI options enable directly, is the per-source per-lens redaction: for any source where `outlierConsensusOutside == true`, drop the named outlier lens and recompute the consensus over the remaining five. On codex this would drop `abc` and recompute `equalMid` over `{bootstrap, jackknifeNormal, BCa, studentizedT, profileLikelihood}`. The remaining-five mean would not be sandbagged by the ABC half-width collapse, and the resulting consensus would be a defensible consumer-facing number. The v0.6.240 axis does not implement this redaction directly (it is a diagnostic, not an aggregator), but it provides the per-source per-lens identification that the redaction needs as input.

## 4. Why `globalOutlierLens = abc`

The aggregate `globalOutlierLens` field is the mode of `outlierLens` across the six sources. In the live-smoke, `outlierLens` per source is:

- opencode → abc
- [redacted-editor] → abc
- hermes → profileLikelihood
- openclaw → profileLikelihood
- claude-code → profileLikelihood
- codex → abc

That is **3-of-6 abc, 3-of-6 profileLikelihood**, and the canonical-order tie-break (alpha order `abc < bca < bootstrap < jackknifeNormal < profileLikelihood < studentizedT`) breaks the 3-3 tie in favor of `abc`. The CHANGELOG correctly reports `globalOutlierLens: abc`. But the underlying reality is a **two-lens shared modal-outlier behavior**: ABC and profile-likelihood are jointly the most-discrepant lenses across the queue, with ABC dominating on small-n sources (codex 64 rows, [redacted-editor] 333 rows) and on the negative-millions opencode source (461 rows), and profile-likelihood dominating on the larger-n sources where the slope is more heavy-tailed (hermes 297, openclaw 567, claude-code 299).

This is consistent with the v0.6.225 release note (which introduced profile-likelihood as the sixth lens with intrinsic asymmetry) and with the v0.6.236 `--show-extremes` finding that documented `jackknife~studentizedT` as the lowest-IoU pair on 5 of 6 sources. The v0.6.240 result complements those: jackknife and studentized-t are the *most-similar* small pair (low IoU because both are tight and aligned), while ABC and profile-likelihood are the *most-discordant* small pair (high `outAbsZ` because each, on different source classes, is the lens that walks furthest from consensus). The 9th and 13th axes together carve the 6-lens consumer cell into a two-cluster structure: a tight `{jackknifeNormal, studentizedT}` core and two oscillating-outlier lenses `{abc, profileLikelihood}` whose roles flip based on source-n and slope-tail behavior.

## 5. Calibration: how does this compare to the pew-insights v0.6.219 anchor?

The Deming-slope UQ suite was anchored by v0.6.219 in mid-W17. Every cross-lens axis since has been scored relative to that anchor's six-lens output. Across the test-count trajectory, each axis added 30-115 tests (v0.6.231 +71, v0.6.232 +78, v0.6.233 +115, v0.6.234 +56, v0.6.235 +65, v0.6.236 +8, v0.6.237 +50, v0.6.238 +34, v0.6.239 +38, v0.6.240 +36) — a total of ~551 tests added over the 13-axis cell — for a final test count of `6622` (per the v0.6.240 release note `tests 6586->6622`).

The interesting per-axis scaling is that the test-count growth has been *strictly linear* in axis count (~42 tests per axis on average) but the *information* surfaced has been heavily superlinear: each axis falsifies a different claim about cross-lens consensus, and the 13th axis falsifies the most fundamental one — that lens midpoints can be combined at all without per-lens normalization by stated uncertainty.

If you were to build a 14th axis, the obvious gap is **per-lens-per-source per-row** — the residual axis Z-score normalized by the *row* uncertainty rather than the *lens* uncertainty. That would surface heteroscedasticity within a single lens-source cell. The v0.6.240 release note does not announce a 14th axis, and the closure-then-falsifier cycle that the meta-posts have been tracking suggests it is more likely that v0.6.241 is *not* a 14th axis but a different consumer-cell deepening: a `--auto-redact-outlier` aggregator, perhaps, or an `--alert-discordant <f>` default-threshold finder. Either would be consistent with the empirically-observed pattern that consumer-cell depth is structurally bounded around 13 distinct *axes* but can be deepened indefinitely along the *aggregator* and *threshold-tuning* dimensions.

## 6. Three falsifiable predictions

P-13z.A: At the next pew-insights release (v0.6.241), if a `--redact-outlier` flag is added to `lens-residual-z` and applied to the same `~/.config/pew/queue.jsonl` snapshot, the recomputed equal-weight consensus midpoint on codex (after dropping `abc`) will lie within `100 * s_min` of the pre-redaction `equalMid = 8585350.48`, where `s_min` is the smallest of the remaining five half-widths. Probability >55%. Falsified if the redacted consensus drifts more than 100 half-widths.

P-13z.B: Within the next 5 pew-insights releases, the test count for the lens-residual-z subcommand will grow from its v0.6.240 baseline (+36 release tests) by no more than 25 additional tests — i.e. the diagnostic is "complete" and only marginal-edge tests will accumulate. Probability >65%. Falsified if a v0.6.24x release adds ≥30 lens-residual-z tests in a single bump.

P-13z.C: The next live-smoke run with a different seed (`--seed 8` or `--seed 11`) on the same queue file will produce `outAbsZ` for codex/abc within a factor of 2 of `45211.79` (i.e. between `22605` and `90423`). Probability >70%. Falsified if the codex/abc residual changes by more than a factor of 2 across seeds — which would indicate the diagnostic is itself bootstrap-sample-dependent on the small-n source.

## 7. Closing: the 13th axis as the first axis that names a specific lens

Every prior axis in the consumer cell — jaccard, sign, width, overlap-graph, midpoint-dispersion, asymmetry, pair-inclusion, rank-correlation, coverage-volume, LOO, precision-pull, adversarial-envelope — produced a source-level scalar. The 13th, lens-residual-z, produces a source-level *plus lens-level* tuple. That structural shift is what makes `outAbsZ = 45211.79` legible at all. In the equal-weight consensus framing, codex was just "the small-n source where the consensus is wider"; in the LOO framing, codex was "the source where dropping any lens shifts the consensus modestly"; in the adversarial-envelope framing, codex was "the source where adversarial weighting can shift the midpoint substantially". None of those framings let you say *which lens* is the discrepant one and *by how many of its own stated uncertainties*.

That is the new diagnostic capability. v0.6.240 (release `65ec1d6`, refinement `8b6beb1`, feat `b33a471`, test `1114046`) is the first release in the cell that supports the question "is this lens lying about how confident it is?" with a number. On `~/.config/pew/queue.jsonl` at `2026-04-29T22:52:24.612Z` with `--bootstraps 200 --seed 7`, the answer for `(codex, abc)` is *yes, by a factor of 45,211.79*. That is the magnitude anomaly.

The cross-lens UQ stack now has, for the first time, a per-lens lie detector. Every downstream aggregator is on notice.

---

**Citations:** v0.6.240 release `65ec1d6`, feat `b33a471`, refinement `8b6beb1`, tests grew `6586 → 6622` (+36), live-smoke `2026-04-29T22:52:24.612Z` against `~/.config/pew/queue.jsonl` (2021 lines), `meanOutlierAbsZ = 7604.7743`, `globalOutlierLens = abc`, `globalOutlierDirection = down`, `nSourcesWithConsensusOutside = 6/6`, codex row `64 rows / outlierLens=abc / outAbsZ=45211.79 / signed=-45211.79 / down / mean|Z|=7537.63 / nOut=4 / concordance=0.0001`.
