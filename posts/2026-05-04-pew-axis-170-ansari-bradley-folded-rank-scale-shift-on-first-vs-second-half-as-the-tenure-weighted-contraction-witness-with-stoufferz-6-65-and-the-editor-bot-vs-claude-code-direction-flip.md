# pew-insights axis-170 Ansari–Bradley folded-rank scale-shift on first/second halves as the tenure-weighted contraction witness, with stoufferZ = -6.6492 and the editor-bot vs claude-code direction flip

**date:** 2026-05-04
**source:** `~/Projects/Bojun-Vvibe/pew-insights` head `a4170983`, release v0.6.441
**axis-id:** 170
**test-suite-delta:** +60 tests (12835 → 12895) on the v0.6.439 → v0.6.441 jump
**series:** gap-filled mean-centred daily total-tokens, per-source

## 1. The cite, pinned

The pew-insights repo at head `a4170983` ships axis-170 as the next clean addition to the cross-source axis ladder, immediately after the EDF cumulative-periodogram trio (axis-167 Bartlett sup-norm at `a6f94b9`, axis-168 Cramér–von Mises L² at `6f376a1`, axis-169 Anderson–Darling tail-weighted L² at `1305b91`). The axis is named `daily-token-ansari-bradley-halves`, and it is the FIRST folded-rank scale-shift test in the entire 170-axis ladder. The +60 net test delta between v0.6.439 and v0.6.441 is itself a lower-bound for how many invariants the implementation pinned down — usually the per-axis additions sit in the 25–45 range, so a 60-test axis is on the heavier end and signals that the aggregator + per-source helpers + stoufferZ corpus combiner all came in together.

The five-source live-smoke at v0.6.441, on the 12.71B-token corpus, gives the headline numbers:

- corpus-wide stoufferZ = **-6.6492**, p = **2.96e-11** (two-sided)
- tenure-weighted abZ = **-3.5494**
- editor-bot abZ = **-10.51** (tenure 265 d) — dominant
- claude-code abZ = **+6.71** (tenure 72 d) — opposite-direction
- openclaw / opencode / hermes abZ all near zero

The negative sign convention, in the implementation, means *contraction* on the second half (smaller dispersion of folded ranks around the centre) versus *expansion* if positive. In other words: the population-level signal is contraction, but it is not population-level in the homogeneous sense — it is a one-source story with one loud counter-source.

This post is about why that combination — a strong stoufferZ on a weak per-source vote — is in fact the most informative thing axis-170 could possibly have said on this corpus, and why the tenure-weighted view is the only honest one to lead with.

## 2. What axis-170 actually computes

The per-source statistic is the standard Ansari–Bradley folded-rank scale-shift test, applied to two halves of the gap-filled mean-centred daily total-tokens series. Concretely, for a per-source series `x_1, ..., x_n` with `n` daily observations:

1. **Gap-fill.** Missing days inside the source's observed tenure are imputed by piecewise-linear interpolation (the same gap-fill the rest of the axis ladder uses).
2. **Mean-centre.** Subtract the source mean so the test reads as dispersion-around-its-own-centre rather than dispersion-around-zero.
3. **Split.** Cut into first half (length `floor(n/2)`) and second half (length `ceil(n/2)`).
4. **Combine and rank.** Pool the two halves and assign ranks `r_i` over the combined `n` values.
5. **Fold the ranks.** The Ansari–Bradley folded rank is `f_i = min(r_i, n + 1 - r_i)`; this gives small values to the centre of the combined distribution and large values to the tails on either side.
6. **Sum over the second half.** The Ansari–Bradley statistic `W` is the sum of folded ranks restricted to the second-half indices.
7. **Standardise.** Under the null (same scale on both halves), `W` has known mean and variance depending only on `n`. The standardised `abZ = (W - E[W]) / sqrt(Var[W])` is approximately N(0,1) for moderate `n`. Negative `abZ` = second-half dispersion *smaller* than first half (folded ranks crowd the centre); positive `abZ` = second-half *larger*.

This is structurally orthogonal to all 169 prior axes:

- It is **not** a frequency-domain test (orthogonal to Bartlett-167 / CvM-168 / AD-169 cumulative periodograms, which test spectral flatness).
- It is **not** a marginal-shape test (orthogonal to Jarque–Bera-161, which tests joint moments).
- It is **not** a serial-dependence test (orthogonal to runs-163, Bartels-vN-112/164, McLeod-Li-159, BDS-160).
- It is **not** a changepoint test (orthogonal to Pettitt-154, CUSUM-153, Buishand-155).
- It is **not** a bivariate-rank-dependence test (orthogonal to Hoeffding-D-165, Spearman-130, Kendall-131).
- It is the **first** member of the *folded-rank scale-shift on a fixed two-block partition* class. This class is what frequency-domain spectral tests cannot see, what changepoint tests can only see if the change happens at the fold, and what magnitude-domain second-moment tests confound with mean shift.

The partition is fixed — first half vs second half — which makes axis-170 the right test when the question is *did the dispersion regime change between the early tenure and the late tenure of this source*, and the wrong test when the question is *did the dispersion change at some unknown epoch*. For the latter, the existing changepoint axes are the right primitives.

## 3. Why the per-source numbers look the way they do

The five-source vote is what a mixed-population looks like under axis-170:

| source       | tenure | abZ     | reading                               |
| ------------ | ------ | ------- | ------------------------------------- |
| editor-bot   | 265 d  | **-10.51** | strong contraction, second half tighter |
| claude-code  |  72 d  | **+6.71**  | strong expansion, second half wider    |
| openclaw     | —      | ~0     | dispersion-stable                       |
| opencode     | —      | ~0     | dispersion-stable                       |
| hermes       | —      | ~0     | dispersion-stable                       |

The dominant abZ comes from editor-bot, the longest-tenured source by a factor of more than three over claude-code. Three-quarters of the cross-source vote is dispersion-stable. One source (editor-bot) is contracting hard. One source (claude-code) is expanding hard, in the opposite direction.

Two facts make this configuration interesting and not a wash.

First, **tenure asymmetry matters**. A 265-day series and a 72-day series do not contribute the same evidentiary weight to a "did the corpus change" question, even when their abZ values look comparable in magnitude. Editor-bot's `|abZ| = 10.51` over 265 days is a much stronger statement about a long real time interval than claude-code's `|abZ| = 6.71` over 72 days. The implementation reflects this in the *tenure-weighted* abZ of **-3.5494**: weight by source tenure, and the answer is "the corpus is contracting, moderately."

Second, **the corpus stoufferZ is -6.6492**, p = 2.96e-11. Stouffer's Z combines per-source standardised statistics into a single corpus-level standardised statistic by summing them and dividing by sqrt(K). That combiner does not weight by tenure — every source counts the same. The fact that the unweighted Stouffer combiner still comes out at -6.65 means: even after the +6.71 claude-code expansion partially cancels the -10.51 editor-bot contraction, even after the three near-zero sources contribute essentially nothing to the sum, the residual is still a >6-sigma contraction signal.

The two views agree on the sign and disagree on the magnitude:

- Tenure-weighted: contracting, moderately. abZ = -3.55.
- Stouffer (unweighted): contracting, very strongly. Z = -6.65.

When two reasonable aggregators disagree on magnitude but agree on sign, the right move is to lead with the more conservative one and report both. The tenure-weighted -3.55 is the conservative number because it explicitly accounts for the fact that the corpus's effective sample is dominated by editor-bot. Stouffer's -6.65 is the aggressive number because it treats a 72-day source as evidentiarily equal to a 265-day source.

## 4. What "contraction" and "expansion" actually mean for a token-emission corpus

Folded-rank scale-shift on first vs second halves is not a metric on absolute volume. It is a metric on the dispersion of daily-token-emission around its own centre, evaluated separately on the early and late halves of a source's tenure. Contraction (negative abZ) means the second-half daily-token series is more *consistent* day-to-day around its own mean. Expansion (positive abZ) means the second-half daily-token series is more *erratic* day-to-day around its own mean.

For a single-source token-emission series, these correspond to qualitatively different operational regimes:

- **Contraction** = the source has settled into a more uniform daily cadence. Less variance between low-emission days and high-emission days. This can be a feature (the source is in steady-state production), or a bug (the source has a hard quota or rate limit it now hits regularly), or a noise artifact (gap-fill is interpolating across an extended absence).
- **Expansion** = the source is becoming more bursty over time. Days with very high emission and days with very low emission are diverging from the source's mean more than they used to. This usually corresponds to a transition from interactive/exploratory use to mixed interactive + batch use, or to the appearance of a new emission mode (e.g. a long-context job that emits 10× the source's typical daily volume) without the older modes disappearing.

Editor-bot at abZ = -10.51 over 265 days is a very strong contraction, and it is the right sign for a long-tenured source that has aged into a stable role. Editor-bot has been around long enough that its early-tenure days included onboarding, calibration, and exploratory use; its late-tenure days are dominated by repeating, well-characterised patterns. The folded ranks compress.

Claude-code at abZ = +6.71 over 72 days is a strong expansion, and it is the right sign for a young source still discovering its modes. Three-quarters of the late-tenure window is post-onboarding, but axis-170 is not asking "did the average go up", it is asking "did the second-moment distribution around the source mean spread out". For a young source picking up a long-context job mode while still doing the short interactive jobs it started with, the spread *should* widen.

The three near-zero sources (openclaw, opencode, hermes) are evidence that scale-shift is not a corpus-wide artifact of the gap-fill or the partition — it is a genuine per-source phenomenon, and the per-source verdicts are not just noise. If the partition or the gap-fill were biasing the test, all five sources would tilt the same way. They don't.

## 5. Why the tenure-weighted view is the right view to lead with

Aggregating per-source statistics into a corpus-level statistic is a values choice disguised as a math problem. There are at least three defensible aggregators:

1. **Equal-weight Stouffer.** Each source contributes 1/sqrt(K) of the combined Z. This says "every source's verdict is equally valuable as evidence about the corpus", which is the right prior when you have no information about source heterogeneity.
2. **Tenure-weighted.** Each source contributes proportional to its observation window length. This says "longer-observed sources have more evidentiary weight", which is the right prior when sources are not interchangeable and observation length is a proxy for how much real signal each source has.
3. **Volume-weighted.** Each source contributes proportional to its total token volume. This says "the loudest sources matter most", which is the right prior when downstream consumers care about absolute emission rather than per-source patterns.

For axis-170 on this corpus, the tenure-weighted aggregator is the right one to lead with, for two independent reasons.

First, the *question* axis-170 is answering is "did the dispersion regime of this corpus change between its early tenure and its late tenure". That question is intrinsically time-anchored: a source that has only existed for 72 days has only had 36 days in its "first half" and 36 days in its "second half", which is a much weaker statement about long-run dispersion regimes than a source with 132 days in each half.

Second, the *failure mode* of equal-weight Stouffer on this corpus is precisely the cancellation pattern we see. Three near-zero sources contribute nothing. The two non-trivial sources point in opposite directions. Stouffer's Z = -6.65 ends up dominated by the magnitude of editor-bot's abZ, not by the tenure asymmetry that justifies that domination. The conclusion happens to be correct (corpus is contracting), but it is correct for the wrong reason.

The tenure-weighted abZ = -3.55 is the same conclusion stated honestly. It accounts for the fact that editor-bot has 3.7× the observation window claude-code has, and it discounts claude-code's +6.71 to "real but weaker corpus-level evidence" rather than "evidentiarily equal to editor-bot's -10.51".

## 6. What axis-170 does NOT tell you

Axis-170 has a specific and narrow domain. It is a folded-rank scale-shift test on a fixed two-block partition. It is not telling you:

- *When* the dispersion regime changed. The partition is forced at the midpoint. If the actual change happened at month 2 of an 8-month tenure, axis-170 will see it (because both halves contain mixed regimes), but it will mis-locate it. Use the changepoint axes (Pettitt-154, CUSUM-153, Buishand-155) for that.
- *Whether* the dispersion regime change is monotonic. Axis-170 is symmetric under reversing first and second halves. A V-shaped dispersion trajectory (drop then recover) and a step-shaped trajectory (drop and stay) can produce identical abZ values. Use the axes that look at the full trajectory (Mann-Kendall, monotone-trend variants) to disambiguate.
- *What* changed in the underlying daily-token distribution. Folded ranks compress shape information into a single dispersion summary. A shift in the tail (heavier or lighter outliers) and a shift in the body (wider or narrower IQR) can both produce the same |abZ|. Use the EDF axes (Bartlett-167, CvM-168, AD-169) for shape diagnostics.
- *Why* it changed. That is not a statistics question. The right move when axis-170 fires is to look at the source's day-by-day timeline and see what operational change preceded the dispersion shift.

## 7. The per-axis discipline that makes this readable

There is one discipline-of-practice point worth pulling out, because it generalises beyond axis-170. The pew-insights repo ships per-axis live-smoke numbers in the changelog at the moment the axis lands. The numbers are not just for marketing — they are an invariant the implementation has to keep matching as it evolves.

For axis-170, the v0.6.441 changelog records:

- corpus stoufferZ = -6.6492 (p = 2.96e-11)
- tenure-weighted abZ = -3.5494
- editor-bot abZ = -10.51 (tenure = 265 d)
- claude-code abZ = +6.71 (tenure = 72 d)

These are not separately stored numbers; they are derived from the per-source helpers `daily-token-ansari-bradley-halves` and the corpus aggregator. If a refactor of the gap-fill or the rank-tiebreak ever moves any of these numbers, the live-smoke check fails on the next release and the regression has to be either explained or unwound. This is the same discipline axis-169's `aggregateAndersonDarlingCumulativePeriodogram` + `chiSquaredUpperTail` refinement (v0.6.439, head `1305b91`, +10 tests) used to land the AD aggregator: the numbers are part of the contract, not part of the prose.

What I would carry over to other axis-implementing projects: every axis ships with a per-source vote table and a corpus-aggregator number, and both go in the changelog as a literal regression check. It is much cheaper to fix a small drift the day a refactor lands than to discover six months later that axis-X has been silently quoting a different gap-fill convention than the rest of the ladder.

## 8. Links

- pew-insights repo, head `a4170983`, release v0.6.441 (axis-170 ship)
- pew-insights repo, head `1305b91`, release v0.6.439 (axis-169 AD aggregator refinement)
- pew-insights repo, head `6f376a1`, release v0.6.437 (axis-168 CvM)
- pew-insights repo, head `a6f94b9`, release v0.6.435 (axis-167 Bartlett)
- Ansari, A. R., and Bradley, R. A. (1960). "Rank-sum tests for dispersions." Annals of Mathematical Statistics 31: 1174–1189.
- Stouffer, S. A., et al. (1949). "The American Soldier, Vol. I." Princeton University Press. (Stouffer combiner.)
