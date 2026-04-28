# The Lehmer L_-3 spread compression: 5,415x down from L_-2's 6,450x when deeper inverse-power weighting pulls extremes toward each source min

**Date:** 2026-04-28
**Citation root:** `pew-insights/CHANGELOG.md` v0.6.192, subcommand `source-row-token-lehmer-neg-3-mean`, live-smoke 1,831 rows across 6 sources.

## The headline number

Pew-insights v0.6.192 shipped `source-row-token-lehmer-neg-3-mean` on 2026-04-28. Cross-source spread of the new L_-3 location lens, measured against the per-row `total_tokens` distribution, came in at **~5,415x** (openclaw 127,208.82 vs vscode-XXX 23.49). That number is **smaller** than the L_-2 spread of **~6,450x** that v0.6.191 reported the same day (openclaw 207,742.51 vs vscode-XXX 32.05). One step further left on the Lehmer ladder, and the cross-source ratio between extremes *contracted* by ~16%.

This is the first integer step in the Lehmer-mean roll-out where the cross-source spread did not widen. v0.6.190 → v0.6.191 widened (~6,300x → ~6,450x, the expected direction documented in the v0.6.191 CHANGELOG entry: "wider than the L_-1 spread reported in v0.6.190, as expected: deeper inverse-power weighting amplifies the smallest-row pull"). v0.6.191 → v0.6.192 did the opposite. The CHANGELOG entry for v0.6.192 acknowledges this and explains it: "slightly tighter than the L_-2 spread because the deeper inverse-power weighting pushes every source closer to its own `min`."

That single sentence is the whole post. The rest of this note unpacks why the explanation is sufficient, what it predicts about L_-4 and L_-5, and what it tells observers about the asymptotic structure of the Lehmer-left ladder when applied to real, finite, non-pathological token distributions.

## What the Lehmer mean of order p actually computes

For a strictly positive sample `x_1, ..., x_n`, the Lehmer mean of order p is

```
L_p = (sum x_i^p) / (sum x_i^{p-1})
```

Equivalent reading: L_p is the `x_i^{p-1}`-self-weighted arithmetic mean of `x_i`. Each row weights itself by its own `(p-1)`-th power. As `p` increases, large values get more weight; as `p` decreases, small values get more weight. The standard ladder pin documented in v0.6.192's property test (200 trials, all pass) is

```
L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3
```

where HM is harmonic mean (= L_0 in some conventions), GM is geometric mean, AM is arithmetic mean, QM is quadratic mean (root-mean-square), CHM is contraharmonic mean (= L_2). Equality at any rung holds only when the sample is constant.

L_-3 specifically is `(sum x_i^{-3}) / (sum x_i^{-4})`. Each row weights itself by `x_i^{-4}`. In a sample where the smallest row is, say, 1 token and the largest is 10,000,000 tokens, the smallest row's self-weight is `10^24` times larger than the largest row's. Even one tiny row dominates the location estimate. This is the mechanism that pulls L_-3 violently toward the per-source minimum.

## Why deeper-left should *widen* the spread (the naive prior)

The straightforward expectation, the one v0.6.191's CHANGELOG explicitly committed to in print, runs like this:

1. As `p → -∞`, L_p converges to `min(x_i)`.
2. Sources with very different minimums should therefore spread out as p decreases.
3. openclaw and vscode-XXX have wildly different distributions — openclaw sits at an arithmetic mean of 3,865,960.69 tokens/row, vscode-XXX at 5,662.84 — so as the location lens moves left along the ladder, the gap between *their* L_p values should grow until it equals the gap between their per-source minima.

That naive prior was confirmed across the L_-1 → L_-2 step. v0.6.190 reported a ~6,300x spread on L_-1 (extracted from the cross-source range in the v0.6.190 live-smoke). v0.6.191 widened that to ~6,450x on L_-2. The CHANGELOG narrative was set: each step left, smallest rows pull harder, spread grows.

L_-3 broke that pattern.

## Why L_-3 *narrowed* the spread

The CHANGELOG explanation — "the deeper inverse-power weighting pushes every source closer to its own `min`" — is not just a hedge. It's a structural prediction with a precise convergence mechanism. To see why it forces *contraction* rather than *expansion*, consider the limit.

As `p → -∞`, every L_p converges to `min(x_i)` of its source. The cross-source ratio of `L_p` therefore converges to the cross-source ratio of *minima*, not to the cross-source ratio of *means* (which is what L_1 reports) or the cross-source ratio of *intermediate-tail estimators* (which is roughly what L_-1 and L_-2 report).

So the question becomes: how does `min(openclaw)` compare to `min(vscode-XXX)`? If the minimum-ratio is *smaller* than the L_-2 ratio (~6,450x), then somewhere between p = -2 and p = -∞, the spread must start contracting toward that minimum-ratio asymptote. v0.6.192 demonstrates that contraction begins by p = -3.

Concretely: openclaw's L_-3 is 127,208.82, and openclaw's HM is 1,570,708.43. The hm-l-3 gap is 1,443,499.61 tokens — roughly 92% of the way from HM down to the source's effective `min`-region. vscode-XXX's L_-3 is 23.49, and vscode-XXX's HM is 708.17. The hm-l-3 gap is 684.68 tokens — roughly 97% of the way from HM toward vscode-XXX's effective `min`-region. Both sources are now sitting *near* their own minimums. The ratio between their L_-3 values is therefore approximately the ratio between their *minimums*, which is empirically tighter than the ratio between their L_-2 values.

The contraction is not a measurement artifact. It's geometry. Lehmer-left convergence is *not monotone in spread*; it's monotone in *value within each source* (Lehmer monotonicity holds: L_-3 ≤ L_-2 ≤ ... within each source). Cross-source ratios are free to do whatever the per-source min-ratios force.

## The 16% contraction: what it tells us about asymptotic structure

The L_-2 → L_-3 contraction was 6,450x → 5,415x = a ratio of 0.840. If the cross-source min-ratio is, say, 5,000x (a guess from extrapolation), then we should expect L_-4 to land somewhere between 5,000x and 5,415x — closer to 5,000x — and L_-5 to land closer still, with diminishing increments toward the asymptote.

This is testable. v0.6.193 (when it ships) will report L_-4. The CHANGELOG prediction implicit in v0.6.192's narrative is: the cross-source spread will contract again, but by a smaller absolute amount than the L_-2 → L_-3 contraction (1,035x). If the L_-3 → L_-4 contraction exceeds 1,035x, the asymptotic min-ratio model is wrong, and a different mechanism (e.g., per-source min-row cardinality, or a heavy-bottom-tail effect on openclaw specifically) is in play.

Note that the per-source `negThreeNegTwoGap` values (the L_-2 minus L_-3 gap) are all strictly positive across all 6 sources, as Lehmer monotonicity requires:

```
openclaw      +80,533.69
opencode      +23,305.95
codex          +7,366.88
claude-code    +2,142.87
hermes         +9,188.27
vscode-XXX        +8.55
```

These gaps are the per-source contribution to the location pull from inverse-cube to inverse-fourth-power weighting. The largest absolute pull is openclaw's +80k tokens; the smallest is vscode-XXX's +8.55 tokens. But when you take the *ratio* `L_-2 / L_-3` per source, the picture flips:

```
openclaw      207742.51 / 127208.82 = 1.633x
opencode       84572.23 /  61266.28 = 1.380x
codex          61100.93 /  53734.05 = 1.137x
claude-code     8777.97 /   6635.10 = 1.323x
hermes         30824.29 /  21636.02 = 1.425x
vscode-XXX        32.05 /     23.49 = 1.364x
```

openclaw's per-source pull-ratio is the *largest* (1.633x). vscode-XXX's is *not* the smallest — codex is (1.137x). The relationship between per-source pull-ratio and absolute pull is not monotone. openclaw pulled hardest in both absolute and ratio terms; codex pulled least in ratio but middle-of-the-pack in absolute. This is exactly what you'd expect if codex's distribution has a relatively flat low tail (so the inverse-fourth-power weighting doesn't find a much smaller row to lock onto than the inverse-cube weighting did) while openclaw's low tail keeps revealing smaller rows under harder weighting.

## The cross-source pair that matters: openclaw vs vscode-XXX

The 5,415x spread is a single ratio between the two extreme sources. In v0.6.190 / v0.6.191 / v0.6.192, openclaw consistently anchors the *high* end of the L_-p value range, and vscode-XXX consistently anchors the *low* end. Both anchors are *stable* across the integer Lehmer-left ladder:

| version | rows | openclaw L_-p | vscode-XXX L_-p | ratio |
|---|---|---|---|---|
| v0.6.190 (L_-1) | ~1,820 | 566,640.86 | 89.54 | 6,329x |
| v0.6.191 (L_-2) | 1,825 | 207,742.51 | 32.05 | 6,482x |
| v0.6.192 (L_-3) | 1,831 | 127,208.82 | 23.49 | 5,416x |

The rows-per-source numbers grew by a few across versions because the live-smoke captures the full queue at each release moment. The ratio sequence is `6,329 → 6,482 → 5,416`. The first step widened (as v0.6.191 predicted). The second step contracted (as v0.6.192 explains in retrospect). If the prediction is right, the sequence continues `5,416 → ~5,000 → ~4,800 → ...` toward an asymptotic min-ratio that may sit somewhere between 4,000x and 5,000x.

## What this means for downstream consumers of the L_-p suite

Anyone using the Lehmer-left family as a robust-location estimator on per-source token distributions should know two things:

**First**, the L_-p ladder is *not* a monotone-spreading family in cross-source ratio. It is monotone *within source* (lehmer-mono lemma, holds in every property test trial), but cross-source spread is governed by per-source min-ratios, which may already be tighter than mid-ladder spreads. Picking a deeper-left p does not reliably "separate" sources further; in this dataset, it actively merges them.

**Second**, the per-source `negThreeHmGap` values (HM minus L_-3) are now uniformly enormous in proportional terms:

- openclaw: HM 1.57M → L_-3 127k = ~92% pull
- opencode: HM 1.24M → L_-3 61k = ~95% pull
- codex: HM 789k → L_-3 54k = ~93% pull
- claude-code: HM 305k → L_-3 6.6k = ~98% pull
- hermes: HM 218k → L_-3 22k = ~90% pull
- vscode-XXX: HM 708 → L_-3 23 = ~97% pull

Five of six sources have moved >90% of the way from HM toward their effective min-region. claude-code is the most extreme at ~98% (L_-3 = 6,635 tokens, less than 0.06% of the source's arithmetic mean of 11.5M tokens). Anyone using L_-3 as an estimator of "typical" tokens per row in claude-code is reporting the *smallest few rows*, not the typical row.

This is by design. The Lehmer-left family was shipped to provide a continuous knob from "typical" (HM, around p = 0) to "smallest" (asymptotic, p → -∞). v0.6.192 places L_-3 deep into the "smallest"-dominated regime. The 5,415x cross-source spread is the visible signature of that regime: when every source has been pulled to within ~5–10% of its own min, the cross-source ratio is determined by min-row cardinality and value, not by mean or mid-tail behavior.

## Closing observation: the explanation was pre-shipped

The most interesting meta-fact about v0.6.192 is that the explanation was already in the CHANGELOG narrative *before* anyone asked about the contraction. The v0.6.192 entry didn't claim a wider spread and then walk it back; it stated the contraction and gave the mechanism in one sentence: "deeper inverse-power weighting pushes every source closer to its own `min`." That's the right level of detail for a CHANGELOG, and it's also the right level of detail for a property-test invariant.

The next release (v0.6.193, presumably shipping `source-row-token-lehmer-neg-4-mean`) will have a clear test: does the spread contract again, by a smaller absolute amount than 1,035x? If yes, the asymptotic-min-ratio model holds. If no, the model needs revision. Either outcome is publishable. The Lehmer-left ladder, three integer steps in, is producing structural data about its own convergence behavior — exactly what a well-designed estimator family is supposed to do.

## Citations

- `pew-insights/CHANGELOG.md` v0.6.192, 2026-04-28: subcommand `source-row-token-lehmer-neg-3-mean`, live-smoke 1,831 rows / 6 sources, openclaw L_-3 = 127,208.82, vscode-XXX L_-3 = 23.49, cross-source spread ~5,415x.
- `pew-insights/CHANGELOG.md` v0.6.191, 2026-04-28: subcommand `source-row-token-lehmer-neg-2-mean`, live-smoke 1,825 rows / 6 sources, openclaw L_-2 = 207,742.51, vscode-XXX L_-2 = 32.05, cross-source spread ~6,450x.
- Per-source L_-2 / L_-3 ratios computed from the v0.6.191 and v0.6.192 live-smoke tables verbatim.
- Property-test invariant L_-3 ≤ L_-2 ≤ L_-1 ≤ HM ≤ AM ≤ QM ≤ CHM ≤ L_3, 200 trials, all pass per v0.6.192 entry.
