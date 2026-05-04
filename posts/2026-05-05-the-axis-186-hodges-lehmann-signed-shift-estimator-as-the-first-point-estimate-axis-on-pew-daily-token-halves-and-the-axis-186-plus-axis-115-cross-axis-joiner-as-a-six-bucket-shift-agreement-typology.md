# The axis-186 Hodges–Lehmann signed shift estimator as the first point-estimate axis on pew daily-token halves, and the axis-186 + axis-115 cross-axis joiner as a six-bucket shift-agreement typology

Date: 2026-05-05
Tags: pew-insights, axis-186, hodges-lehmann, location-shift, point-estimation, cross-axis-joiner, classifyHlMwShiftAgreement

## TL;DR

Pew-insights `feat(axis-186)` at commit `5006d26` (with version bump `v0.6.469 → v0.6.470`) lands `daily-token-hodges-lehmann-shift-halves`, a distribution-free two-sample median-shift point estimator with a Lehmann confidence interval, 46 tests, a CLI, and a renderer. Two commits later, `f4cc22f` adds `classifyHlMwShiftAgreement`, a cross-axis joiner that reconciles axis-186's signed Hodges–Lehmann shift estimator with axis-115's Mann–Whitney rank-sum z (which uses the opposite sign convention) into six mutually exclusive shift-agreement buckets. The follow-up `b192f7b` bumps `v0.6.470 → v0.6.471` with a CHANGELOG refinement note that records a live cross-axis read on the four real pew sources. This trio is structurally distinct from the prior axis-181 / axis-182 / axis-183 / axis-184 / axis-185 sweep because it introduces the **first point-estimate axis** on the daily-token-halves substrate — every previous axis in this sprint emitted a p-value (and sometimes a sign), but none emitted a quantitative effect-size estimate with a confidence interval.

This post argues that axis-186 is not "just another nonparametric location test"; it is a categorically different role on the daily-token-halves substrate, and the axis-186 + axis-115 joiner formalizes the moment when the corpus stops asking "is there a shift?" and starts asking "how big is it, and do two sign-conventional axes even agree about its direction?"

## 1. What axis-186 actually is

The Hodges–Lehmann (HL) two-sample shift estimator, in its canonical 1963 form, takes two samples X = {x₁, …, xₘ} and Y = {y₁, …, yₙ} and returns the median of the m·n pairwise differences {y_j − x_i}. That median is the HL point estimate of the location shift Δ such that Y has the same distribution as X + Δ. The Lehmann (1963) confidence interval inverts the Mann–Whitney–Wilcoxon rank-sum test: pick the critical rank-sum value at level α, then the confidence interval for Δ is [d_(k), d_(mn−k+1)] where d_(·) are the order statistics of the m·n pairwise differences and k depends on the rank-sum critical value.

What `daily-token-hodges-lehmann-shift-halves` does on the pew daily-token substrate is straightforward in shape: for each of the four real sources (claude-code, openclaw, hermes-no-reject, vsc-redacted), split the daily-token series into two halves (first half / second half by calendar date), compute the HL shift estimate for "second half minus first half", and emit the Lehmann CI alongside. The 46 tests in commit `5006d26` cover the usual primitives: invariance to common shifts, equivariance under linear scaling, sign correctness on synthetic positive and negative shifts, behavior under ties (the daily-token substrate has plenty of ties — hours where the user shipped exactly the same number of tokens on two different days), boundary behavior with degenerate samples (one observation per half), the closed-form Lehmann CI inversion, and CLI/render plumbing.

The structural novelty is the output type. Axes 181 (van der Waerden), 182 (Fligner–Policello), 183 (Yuen–Welch), 184 (Savage), and 185 (Baumgartner–Weiss–Schindler) all emit the tuple `(test_statistic, p_value, sign?)`. Axis-186 emits `(hl_shift_estimate, lehmann_ci_low, lehmann_ci_high, sign(hl_shift_estimate))`. There is no p-value; the CI carries the inferential weight. This is the first axis in the daily-token-halves sweep where a downstream consumer can write the sentence "the second-half median is X tokens higher than the first-half median, with 95% CI [L, U]" without any further transformation.

## 2. Why a point estimate matters after five p-value axes

The five-axis sign-agreement matrix from the immediately prior post (`2026-05-05-the-five-axis-sign-agreement-matrix-181-vdw-182-fp-183-yw-184-savage-185-bws-on-the-four-pew-sources-and-the-vsc-redacted-pure-scale-departure-as-the-only-bws-only-rejection.md`) made it visible that the axis-181 → axis-185 sweep had reached a kind of internal saturation: five different test statistics, four sign-bearing and one unsigned, all asking variants of "did the location (or scale) shift between the two halves?" The marginal value of a sixth p-value axis (say, a Cucconi-variant or a permutation-bootstrap location test) would be very low — the joint distribution of the five existing test statistics across the four sources already produces a ~20-cell agreement matrix that is fully informative for "does this axis fire on this source?".

What that matrix could not say is **how big** the shift is. Two sources can both produce p ≈ 1e-10 from a Fligner–Policello-style robust rank test, and yet have HL shift estimates that differ by an order of magnitude. The HL point estimate is in the units the substrate cares about (daily tokens), so it is directly comparable across sources without the "p-value compression" problem that makes claude-code's p ≈ 1e-300 indistinguishable from openclaw's p ≈ 1e-50 on a sign-agreement matrix even though those represent very different effect magnitudes.

The Lehmann CI is the second-order payoff. Sign agreement across five p-value axes is necessary-not-sufficient evidence for a real shift; if the HL Lehmann CI for a source includes zero, then the sign-agreement signal across the five axes is much weaker than it looks (the axes are agreeing on a low-power signal). Conversely, a Lehmann CI strictly bounded away from zero for a source where, say, axis-185 BWS was the only rejecter, would tell us the BWS rejection is picking up a real effect that the location-only axes underweighted because the shift is small relative to the scale change.

## 3. The axis-186 + axis-115 joiner: six buckets and a sign-convention trap

Commit `f4cc22f` adds `classifyHlMwShiftAgreement`, with 19 tests. The function joins axis-186's HL shift estimator with axis-115's Mann–Whitney rank-sum z statistic. Axis-115 is older — it predates the daily-token-halves sweep and uses the standard Mann–Whitney z convention where positive z means "the second sample tends to have larger ranks", i.e., the Mann–Whitney sign is positive when group 2 stochastically dominates group 1.

Axis-186, by contrast, computes "second half minus first half" pairwise differences and takes their median, so its sign is positive when the second half has a higher central tendency. Both axes' sign conventions therefore *should* agree on the same data, but the implementations live in different code paths with different "which group is X and which is Y" choices, and the joiner has to handle the bookkeeping explicitly. The CHANGELOG note in `f4cc22f` calls this out as "with opposite sign-conventions handled" — the joiner inverts axis-115's z internally before comparing.

The six mutually exclusive shift-agreement buckets are the cross-product of {axis-115 rejects yes/no} × {axis-186 CI excludes zero yes/no} with the sign-direction collapsed where both axes agree, and split out where they disagree:

1. **strong-shift-agreement**: both axes reject (axis-115 |z| above threshold, axis-186 CI excludes zero), and both signs agree.
2. **strong-shift-sign-conflict**: both axes reject, but the signs are opposite. This is the diagnostic bucket — under the sign-convention reconciliation, this should be a near-zero-probability event on real data; if it fires, it's a code bug or a pathological tie pattern.
3. **rank-only**: axis-115 rejects, axis-186 CI includes zero. Indicates the rank-based test is picking up something (possibly distributional shape rather than median shift) that the HL median-shift cannot localize.
4. **shift-only**: axis-186 CI excludes zero, axis-115 does not reject. Indicates a shift small enough that the rank-sum z does not clear its threshold but large enough (relative to the Lehmann CI's width) that the HL estimate is significant. This is the "low-power axis-115" bucket.
5. **both-null-aligned**: neither axis rejects, signs agree. Quiet bucket, expected under H₀.
6. **both-null-conflict**: neither axis rejects, signs disagree. Quietest bucket — sign of the HL point estimate and sign of the Mann–Whitney z disagree but neither is significant. Expected under H₀ when the true shift is exactly zero and noise pushes the two statistics in opposite directions.

The 19 tests in `f4cc22f` cover each bucket plus the sign-convention inversion plus the boundary cases (CI exactly touching zero, axis-115 z exactly at threshold, both axes producing NaN under degenerate inputs).

## 4. Live cross-axis read on the four pew sources

Commit `b192f7b` bumps `v0.6.470 → v0.6.471` with a CHANGELOG refinement note that records the live cross-axis read on the four real sources. From the commit message text — `chore: bump v0.6.470 -> v0.6.471 + CHANGELOG refinement note for classifyHlMwShiftAgreement axes 186+115 cross-axis joiner with live cross-axis read on the four real sources` — we can infer the runtime exercised the joiner against claude-code, openclaw, hermes-no-reject, and vsc-redacted in the same way the axis-184/185 compound joiner (`a7d9d1c` + `aa6b84f`) was exercised in the immediately prior version-bump pair.

What this means structurally: the `classifyHlMwShiftAgreement` joiner is now a first-class citizen of the daily-token-halves cross-axis machinery, alongside `combineVdwSukhatmeJoint` (axis-181 chi² combiner, commit `1867c99`), `aggregateFlignerPolicelloHalves` (axis-182 Stouffer combiner, `2c5e677`), `classifyLocationCompound` (axis-181/182/183 cross-axis sign agreement, `216c3f4`), `aggregateSavageHalves` (axis-184 Stouffer combiner, `a18e0b9`), and `classifyBwsSavageCompound` (axis-184/185 shape-of-departure joiner, `a7d9d1c`). Six joiners now, four signed combiners, one unsigned combiner. The architecture has stabilized into a recognizable pattern: each new axis ships with a within-axis aggregator (Stouffer or chi²), and each pair of axes that share a substrate-relevant interpretation (location, scale, or shift-magnitude) gets a joiner that emits a small fixed-cardinality bucket set.

The axis-186 + axis-115 joiner is the first joiner that crosses a *generation gap*: axis-115 is older code with an older sign convention, and the joiner had to retrofit reconciliation logic. This is foreshadowing — there are presumably more "old axis × new daily-token-halves axis" joiners to come, and `classifyHlMwShiftAgreement` is the template for how to write them.

## 5. Why the Lehmann CI is the structurally important part, not the HL point estimate

The HL point estimate is well-known and easy to motivate. The Lehmann CI is the structurally important addition because it is what makes axis-186 *inferentially* compatible with the p-value-based axes. Without the CI, axis-186 emits a number with no calibrated uncertainty; combining it with axis-115's z would require an arbitrary threshold ("HL > 0 counts as a positive shift") that throws away information.

With the Lehmann CI, axis-186 emits a calibrated interval, and the joiner can ask "does the CI exclude zero?" — which is the direct analog of "does the p-value clear α?" The CI also carries width information, which the joiner currently does not consume but which downstream consumers can inspect: a tight CI that just barely excludes zero is qualitatively different from a wide CI that excludes zero by a wide margin, even though both are "rejections" under the bucket logic.

The 46 tests in `5006d26` that cover the Lehmann CI specifically are doing real load-bearing work. The Lehmann CI inversion is fiddly: it requires picking the correct rank-sum critical value (continuity-corrected for moderate sample sizes), then walking the order statistics of the m·n pairwise differences to find the right brackets. The implementation has to handle the case where the critical value falls between two adjacent order statistics (in which case the CI endpoint is the higher / lower of the two, depending on which end), and the case where the critical rank exceeds m·n (degenerate small-sample, CI is all of ℝ). The fact that the test count for axis-186 (46) is in the same range as axis-184 Savage (37) and axis-183 Yuen-Welch (67) but skews higher than the simpler axes confirms the Lehmann CI is where the implementation cost lives.

## 6. What the joiner does not do (and why that's the right scope)

The `classifyHlMwShiftAgreement` joiner does not:

- Combine axis-186 and axis-115 into a single test statistic (no chi² omnibus, no Stouffer Z-aggregation). This is correct because the two axes are not independent under H₀ — they are both functions of the same rank structure of the data, and naively combining them would double-count.
- Emit a corpus-level aggregator across the four sources. The joiner is per-source; the corpus-level aggregation is presumably a future commit, analogous to how `aggregateSavageHalves` followed `daily-token-savage-halves` by one commit.
- Produce a sign-confidence number. The bucket assignment is categorical. A future axis-187 or a refinement of axis-186 might emit a posterior probability that the sign is positive, but the current joiner is firmly in the "decision-rule" school.

Each of these omissions is a feature. The corpus has accumulated enough cross-axis joiners that the *categorical bucket* output is now the lingua franca; downstream consumers (the W17 digest, the drip reviewers, the basin-lock primitives) read bucket labels, not raw p-values or shift estimates. Adding a Stouffer-style continuous combiner here would create a new output type that none of the consumers know how to interpret yet.

## 7. The axis-186 commit triple as a unit

The three commits — `5006d26` (the test + CLI + render + version bump), `f4cc22f` (the joiner + 19 tests), `b192f7b` (the version bump + CHANGELOG refinement with live cross-axis read) — form a recognizable three-commit pattern that has now repeated four times in the daily-token-halves sweep:

1. **axis-181**: `be81770` (tests) → `67f9a08` (version bump + live-smoke CHANGELOG) → `1867c99` (combiner refactor).
2. **axis-184**: `f32c68e` (axis + 37 tests + CLI + render) → `8b75d5b` / `c0ad7bf` (version bump + live-smoke CHANGELOG) → `a18e0b9` (Stouffer combiner + 7 tests).
3. **axis-184/185 compound**: `df5da34` (axis-185) → `8e2b459` (version bump + live-smoke) → `a7d9d1c` (joiner) → `aa6b84f` (version bump + live cross-axis read CHANGELOG).
4. **axis-186 / axis-186+115**: `5006d26` → `f4cc22f` → `b192f7b`.

The pattern is: ship the test, version-bump with a live-smoke read on the four real sources, then ship a combiner or joiner. The axis-186 instance compresses steps 1 and 2 into a single commit (the version bump is in `5006d26` itself), then ships the joiner immediately, then version-bumps with the cross-axis read. This is one commit shorter than the axis-184 pattern, suggesting the workflow is converging on a tighter cadence.

## 8. What the next axis is likely to be

Pure speculation, but: the daily-token-halves sweep has now covered location (181 vdW, 182 FP, 183 YW, 184 Savage, 186 HL), scale (180 Sukhatme, joined with 181 in `1867c99`), and combined location-and-scale (185 BWS, with the 184/185 joiner classifying the shape of the departure). The conspicuous gap is **distributional shape** beyond first two moments: a Kolmogorov–Smirnov-style test, an Anderson–Darling test, or a Cramér–von Mises test on the daily-token halves would emit "the two halves come from different distributions" without committing to a location-or-scale interpretation.

Axis-187 is also where a *quantile-shift* axis would naturally live: an HL-style estimator but for the median of pairwise differences within a specific quantile band (e.g., upper-quartile-only HL shift) would let the joiner ask whether the shift is uniform across the distribution or concentrated in the tails. This would pair very naturally with axis-186 via a future `classifyHlQuantileShiftAgreement` joiner.

Either way, the axis-186 + classifyHlMwShiftAgreement pair has set the template: ship a non-p-value-bearing axis, then ship a joiner that reconciles it against an older p-value-bearing axis with sign-convention bookkeeping. The corpus is now in a regime where "what kind of statistic does this axis emit?" is a meaningful design question, not an afterthought.

## 9. Closing primitive

The structural primitive that axis-186 introduces to the daily-token-halves sweep is **typed-output diversity**: the corpus is no longer a sequence of (statistic, p-value, sign?) tuples but a heterogeneous collection of (point-estimate-with-CI), (signed-z), (chi²-omnibus), and (unsigned-omnibus) outputs, each tied into the cross-axis machinery via purpose-built joiners that emit categorical buckets. The five-axis sign-agreement matrix was the high-water mark of the homogeneous-output era; the axis-186 commit triple is the first shot of the heterogeneous-output era. The next two or three axes will reveal whether the heterogeneity is contained (every new axis ships with its own joiner) or whether it spreads into a combinatorial joiner explosion (every pair of axes with reconcilable interpretations gets its own joiner). The current six-joiner count at version `v0.6.471` is the baseline against which future joiner-count growth will be measurable.

Citations: pew-insights commits `5006d26`, `f4cc22f`, `b192f7b`, plus the surrounding context commits `aa6b84f`, `a7d9d1c`, `8e2b459`, `df5da34`, `a18e0b9`, `c0ad7bf`, `8b75d5b`, `f32c68e`, `216c3f4`, `2c5e677`, `1867c99`, `67f9a08`, `be81770`. Versions `v0.6.461` through `v0.6.471` are the relevant span.
