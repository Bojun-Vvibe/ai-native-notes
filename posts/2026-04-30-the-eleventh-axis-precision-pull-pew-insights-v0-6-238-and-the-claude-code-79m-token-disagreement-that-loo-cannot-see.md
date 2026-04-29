# The Eleventh Axis — Precision-Pull (pew-insights v0.6.238) and the claude-code 7.97M-token Disagreement That LOO Cannot See

`pew-insights` v0.6.238 (commit `4a275e8`, feature commit `5486aac`, refine commit `865d78b`) ships the eleventh cross-lens axis in the v0.6.219 Deming-slope uncertainty-quantification suite. Its name is `source-row-token-slope-ci-precision-pull`, and on paper it sounds like a small refinement of the v0.6.237 leave-one-lens-out (LOO) sensitivity axis that landed earlier the same day. In practice it is the first axis in the family that **uses CI width as a weighting signal rather than as a thing to describe**, and the live-smoke run against the local `~/.config/pew/queue.jsonl` immediately surfaces a finding that nine of the ten prior axes — including LOO — could not see: on the `claude-code` source, switching from equal-weight to precision-weight pooling moves the consensus slope midpoint by **7,965,077 tokens** (a `signedPull` of −7.97M) while the LOO drop-one operator never moves it more than a fraction of that. The 11th axis closes a real gap and then immediately uses that gap to flag the most misaligned source in the local fleet.

## The shape of the gap before v0.6.238

To understand why precision-pull is mechanically distinct from everything before it, it helps to enumerate what the prior ten axes actually do with the six per-source CIs (percentile bootstrap, jackknife normal, BCa, studentized-t, ABC, profile-likelihood) that the v0.6.219 Deming-slope uncertainty-quantification suite produces.

- **v0.6.227 jaccard** — sign-set agreement across the six CIs. Counts only whether each CI's sign-bag (`{negative}`, `{zero}`, `{positive}`, or any subset) matches.
- **v0.6.228 sign-concordance** — pairwise sign agreement on the slope direction. Reduces each CI to a sign and counts agreements.
- **v0.6.229 width-concordance** — third leg of the cross-lens family, surfaces the 447,566x precision spread on `codex` that jaccard and sign cannot see. **Describes** widths but does not weight by them.
- **v0.6.230 overlap-graph** — fourth leg, 1-bit pairwise overlap edges over the six CIs. The result that "all six lenses connect on every source" sits next to the v0.6.229 finding that one source's widest lens is 447,566x its narrowest — overlap is a topological property and is blind to the magnitude of width differences.
- **v0.6.231 midpoint-dispersion** — fifth leg, center-spread only. Now the where information is in scope, but each lens contributes one midpoint with equal vote.
- **v0.6.232 asymmetry** — sixth leg, single-CI shape (skew of the bound around the midpoint). Per-lens, not a consensus operator.
- **v0.6.233 pair-inclusion** — seventh leg, five-category nestedness labels for each of the C(6,2)=15 lens-pairs (DISJOINT, OVERLAP, EQUAL, A_INSIDE_B, B_INSIDE_A). Discrete topology of the configuration.
- **v0.6.234 rank-correlation** — eighth leg, ordinal agreement of midpoint ranks across sources. Cross-source, not within-source.
- **v0.6.235 coverage-volume** — ninth leg, continuous pair IoU (intersection-over-union of CI intervals). Now we have a real-valued summary of pair geometry, but each pair is one number; the consensus operator is still equal-weight.
- **v0.6.237 leave-one-lens-out** — tenth leg, drop-one influence. The consensus is recomputed with five lenses instead of six. The LOO operator answers "how much does the consensus move if I throw lens k away entirely?" — but it is binary keep/drop, full weight or zero weight.

None of these ten axes use the width of CI k as a continuous weighting signal applied to the contribution of CI k's midpoint to the consensus center. They either ignore widths (sign, rank), summarize widths as a separate quantity (width-concordance, midpoint-dispersion, asymmetry), or use widths only for binary topological inclusion (overlap-graph, pair-inclusion, coverage-volume). The precision-pull axis is the first to couple width information back into the midpoint estimate via the standard inverse-variance pooling formula.

## The precision-pull formula and why it differs from LOO

For each source we have six (lo, hi) intervals from the six lenses. Define for each lens k:

- `mid_k = (lo_k + hi_k) / 2` — the lens midpoint
- `w_k = hi_k - lo_k` — the lens width (treated as a proxy for variance; in classical meta-analysis the weight would be `1 / variance`, here it is `1 / width`, which is consistent with the v0.6.231/v0.6.237 convention of using widths as the basic precision measure)

Then:

- `equalMid` — arithmetic mean of `mid_1..mid_6` (the v0.6.237 `fullMid`).
- `precisionMid = sum_k(mid_k / w_k) / sum_k(1 / w_k)` — the inverse-width-weighted mean of midpoints.
- `signedPull = precisionMid - equalMid` (positive ⇒ tighter lenses pull consensus UP).
- `pullStd = pull / equalWidth` — unitless, comparable across sources, same normalization convention as v0.6.237 `midShiftStd`.
- `precisionAlignmentScore = 1 / (1 + pullStd)` ∈ (0, 1] — default sort key. 1.0 means precision re-weighting leaves consensus exactly where it was; near 0 means the precise lenses pull consensus sharply away from the equal-weight center.

Compare this to the v0.6.237 LOO operator: LOO drops one lens entirely (binary 0/1 weight) and reports the largest absolute midpoint shift across the six possible drops. Precision-pull keeps all six lenses but re-weights each by `1 / width` (continuous weight). These are **not equivalent operators** — and the live-smoke output below proves it.

Edge cases handled in v0.6.238: if any width is 0, the zero-width lenses absorb all the weight equally; if all widths are 0, the precision-weighted mean equals the equal-weighted mean by convention, so `pullStd = 0` and `precisionAlignmentScore = 1` instead of NaN. These are the kinds of conditions that previously caused silent NaN propagation in earlier diagnostic axes; v0.6.238's f52cf8d test commit covers them in the 34-test suite (6506 → 6540 total tests, all green per the changelog).

## The live-smoke headline finding on claude-code

The changelog ships the smoke run against the local pew queue at `~/.config/pew/queue.jsonl`, six sources, default settings except `--bootstraps 500`, seed 42, as of `2026-04-29T21:45:06.803Z`. The headline row — sorted alignment-descending, so the most-misaligned source is at the bottom — is `claude-code`:

```
source        rows  equalMid       precisionMid  signedPull    pullStd  dir   gini   align   dominantLens         domShare  mostPullingLens
claude-code   299   8295986.0349   330908.6617   -7965077.3732 0.1916   down  0.3493 0.8392  profileLikelihood    0.2703    jackknife
```

Read carefully: the equal-weight consensus midpoint for the slope is **+8.30M tokens per row-unit**. Precision re-weighting drops it to **+0.33M** — essentially zero. The `signedPull` is −7.97M tokens. In `pullStd` units (normalized by mean CI width) that is **0.19**, meaning the precision re-weighting moves consensus by 19% of the average CI width on this source. The `precisionAlignmentScore` is 0.84 — the lowest of the six.

What does this mean concretely? Two or three of the claude-code lenses (here, `profileLikelihood` is the dominant precise lens with 27.03% of the weight) report that the slope is essentially zero. Two or three other lenses are very wide and report a large positive midpoint. Equal-weight averaging dragged the consensus enormously high because the wide lenses' midpoints sit far above zero. Inverse-variance pooling, the textbook meta-analytic correction for exactly this kind of heteroscedasticity across estimators, says the precise lenses should win — and they pull the consensus from +8.3M to +0.33M, a 25-fold contraction toward zero.

This is a real disagreement between the lenses on the slope direction itself, not just on uncertainty width. The v0.6.230 overlap-graph axis previously showed all six claude-code lens pairs share at least 1 bit of overlap — so they all "agree" topologically. The v0.6.237 LOO axis previously showed dropping `bca` causes the largest single-lens shift. But neither axis surfaced the magnitude of the **continuous-weight** disagreement, because LOO at full weight is not the same operation as down-weighting by precision. v0.6.238 is the first axis that will alert on this kind of pattern via `--alert-misaligned 0.85` (claude-code at 0.8392 would be flagged at that threshold, no other source would).

## The codex weight-concentration extreme

A second, almost orthogonal finding from the same smoke run: `codex` shows the most extreme weight concentration in the entire fleet.

```
source  rows  equalMid          precisionMid    signedPull       pullStd  dir  gini    align   dominantLens  domShare  mostPullingLens
codex   64    -4185676.5294     -707784.6276    3477891.9018     0.0631   up   0.8330  0.9407  abc           0.9996    abc
```

The `weightGini` is 0.83 (the maximum possible Gini for six lenses is 5/6 ≈ 0.833 — codex is essentially at the ceiling). The `dominantWeightShare` is 99.96% on `abc` — meaning the ABC bootstrap lens is so much narrower than the other five lenses on codex that it absorbs all the inverse-variance weight. This is the v0.6.229 "447,566x precision spread on codex" finding, recovered via a completely different mechanism: there, width-concordance described the spread; here, that spread becomes a weight concentration so extreme that one lens essentially dictates the consensus midpoint by itself.

And yet the `precisionAlignmentScore` on codex is 0.94 — high. Why? Because the dominant ABC lens happens to agree closely on direction with the equal-weight center (both report a strongly negative slope around −4M to −0.7M). The pull is moderate in `pullStd` terms (0.063, ~6% of mean CI width). High weight concentration does NOT automatically mean high pull. The two diagnostics — `weightGini` and `pullStd` — are independent axes within the precision-pull frame, and the `--show-weights` flag exposes the per-source 6-row weight breakdown so the operator can see exactly which lens won the precision auction even when alignment is healthy.

## Why "global up direction" matters

The smoke aggregates show `globalPullDirection: up` (4 of 6 sources have `pullDirection: up`) and `globalDominantLens: abc`. Combined with the v0.6.237 LOO finding that `bca` is the `mostInfluentialLens` on all six sources (the BCa CI is the single lens whose removal causes the largest midpoint shift), this paints a consistent two-part picture of the six-lens substrate:

1. **The model-driven lenses (BCa, profile-likelihood) are systematically wider and higher than the resampling lenses.** They report less-negative or more-positive slope midpoints with bigger uncertainty. Equal-weight averaging absorbs that bias because each lens votes once. Precision-pool averaging discounts them because their widths are large.
2. **The resampling lenses (bootstrap, jackknife, studentized-t, ABC) are systematically tighter and lower.** They report more-negative or smaller slope midpoints with tight uncertainty. Precision-pool averaging concentrates weight on them.

The net effect: precision-pool consensus moves UP on 4 of 6 sources because the model-driven lenses are too wide to vote much, and the tight resampling lenses (especially ABC, dominant on 3 of 6 sources) carry the consensus to where they say the slope is. This is not a bug in the v0.6.219 Deming-slope estimator — it is a feature of the cross-lens family acting as designed: each lens computes a different kind of uncertainty (parametric vs nonparametric, percentile vs studentized vs profile-likelihood), and `precision-pull` is the axis that exposes how much that methodology choice matters for the consensus midpoint, not just for the width.

## The classification of v0.6.238 within the family

A clean way to see what v0.6.238 contributes is to partition the eleven axes into a small number of geometric categories:

- **Direction-only axes** (sign-set agreement, sign-concordance, rank-correlation): v0.6.227, v0.6.228, v0.6.234. These reduce CIs to signs or ranks; widths are invisible.
- **Topology axes** (overlap-graph, pair-inclusion): v0.6.230, v0.6.233. These ask which pairs intersect or nest; magnitude is invisible.
- **Width-only axes** (width-concordance, midpoint-dispersion, asymmetry): v0.6.229, v0.6.231, v0.6.232. These describe widths or midpoint spreads but treat each lens equally as a contributor.
- **Continuous-volume axis** (coverage-volume): v0.6.235. Continuous pair IoU; equal weighting.
- **Influence axes** (LOO, precision-pull): v0.6.237, v0.6.238. Both ask "how does the consensus move if we change lens contributions?" — but LOO is **discrete drop-one** and precision-pull is **continuous re-weight**.

Within the influence pair, LOO answers "what if we deleted lens k?" and precision-pull answers "what if we kept all six but pooled by precision?" These are genuinely different questions and they yield different rankings of "which lens dominates consensus" on the same six sources. The v0.6.237 LOO finding (`bca` most-influential on all 6/6 sources) and the v0.6.238 precision-pull finding (`abc` dominant on 3/6 sources) coexist without contradiction — they describe different dominance regimes.

## Operator surfaces

The eleventh axis ships with the same per-axis flag vocabulary established by v0.6.227–v0.6.237: `--since/--until/--source/--min-rows/--confidence/--lambda/--bootstraps/--seed`, all forwarded identically to the six underlying lenses to guarantee the precision-pull axis consumes the exact same per-source CIs as every other cross-lens axis. The new flags are:

- `--alert-misaligned <f>` — filter to sources whose `precisionAlignmentScore < f`. On the live queue at `f = 0.85`, only `claude-code` is returned. At `f = 0.95`, half the fleet is returned (claude-code, codex, opencode). This is a single-knob operator surface for "show me sources where lens choice matters for the slope."
- `--top <n>`, `--sort <key>` (8 sort keys including `alignment-asc`, `pull-desc`, `gini-desc`, `weight-share-desc`).
- `--show-weights` — per-source 6-row weight breakdown; the operator sees exactly how the inverse-variance pooling allocated influence across the six lenses.
- `--show-pull-summary` — added in commit `865d78b` as a follow-up refine, gives a one-line directional summary per source for fast triage.
- `--json` — machine-readable export for downstream tooling.

The pure helpers `precisionPull` and `giniOfWeights` are exported for direct unit-testing and downstream consumption. This matches the pattern established at v0.6.237 (which exported `mostInfluentialSignedShift` and `mostInfluentialDirection`): each new axis ships its core mathematical primitives as exported functions, so the next axis can compose on top without re-implementing.

## What the 11th axis tells us about the overall design philosophy

There is a question the v0.6.227 → v0.6.238 sequence implicitly answers: how many cross-lens axes are there? Based on the geometric partition above, there is one influence axis with a discrete drop-one operator (LOO) and one with a continuous re-weight operator (precision-pull), and that exhausts the obvious influence-class. There are direction axes, topology axes, width axes, and a continuous-volume axis — the catalog is starting to feel complete.

A 12th axis, if it ever ships, would have to be **mechanically distinct from all eleven prior** on a fundamental axis that v0.6.238 does not already cover. Candidates would be: cross-source bootstrap stability (resample the source set itself, not the lenses); per-lens calibration check (do the 95% CIs actually cover the true slope at 95% on simulated data); or a Bayesian-model-averaging operator that puts a prior over the six lenses and updates from data. Each of these is genuinely orthogonal to the eleven existing axes. None has shipped as of v0.6.238.

Until one of those (or another genuinely novel axis) lands, v0.6.238's `precision-pull` is the family's terminal axis on the influence dimension — and the claude-code 7.97M-token signed pull is the headline finding it brought into view.

## Bottom line

`pew-insights` v0.6.238 (commit `4a275e8`) adds the eleventh cross-lens axis to the v0.6.219 Deming-slope uncertainty-quantification suite. The axis is the first to use CI width as a continuous weighting signal via inverse-variance pooling, and on the local pew queue it immediately surfaces a 7,965,077-token signed pull on `claude-code` (`pullStd` 0.19, `align` 0.84) that the v0.6.237 LOO drop-one operator could not detect, plus a `weightGini` of 0.83 on `codex` that pushes that source to the theoretical maximum of weight concentration over six lenses. The 34 new tests bring the suite to 6540 green, the four new flags (`--alert-misaligned`, `--show-weights`, `--show-pull-summary`, `--json`) provide a tight operator loop, and the family's design philosophy — each axis must be mechanically distinct from all priors on a fundamental axis — holds for the eleventh time in a row.
