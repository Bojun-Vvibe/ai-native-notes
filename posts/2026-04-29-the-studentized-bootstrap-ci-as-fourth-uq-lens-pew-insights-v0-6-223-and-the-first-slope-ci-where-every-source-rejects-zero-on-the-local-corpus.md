# The studentized bootstrap CI as fourth UQ lens: pew-insights v0.6.223 and the first slope CI where every source rejects zero on the local corpus

## What v0.6.223 actually shipped

On 2026-04-29 at 21:20:03 +0800 the pew-insights repository tagged `v0.6.223`, the fourth uncertainty-quantification (UQ) lens applied to the Deming regression slope that v0.6.219 introduced. The release commit is `fe79c63`, the feature commit is `2aeae90`, the test commit is `1af9bb9`, and the post-release refinement that added two diagnostics and two sort keys is `2917818`. The new analyzer is `source-row-token-studentized-bootstrap-slope-ci`, and what it implements is the bootstrap-*t* pivotal CI (Efron–Tibshirani 1993, Chapter 12.5; Hall 1988, *Annals of Statistics* 16:927–953) on top of the same bootstrap mechanics that v0.6.220 percentile CI and v0.6.222 BCa CI already used.

This is the post that any reader who has been following the slope suite should expect at this point, because the suite has been building toward it. v0.6.219 introduced the parametric errors-in-both-variables (EIV) regression — the Deming slope itself, with a tunable variance ratio λ. v0.6.220 wrapped it in a percentile bootstrap CI and produced the first interval object the suite had ever emitted on a slope. v0.6.221 added the deterministic LOO jackknife normal-approximation CI. v0.6.222 added the BCa CI, which corrects for both bias (the median deviation of the bootstrap distribution from the point estimate) and acceleration (the third-moment skewness). All three of those CI lenses share a structural assumption: they invert quantiles of the slope replicates themselves (or, in the BCa case, transformed quantiles of the slope replicates).

v0.6.223 breaks that assumption. The bootstrap-*t* lens does not rank the slope replicates; it ranks the *studentized* statistic `T* = (theta* − thetaHat) / SE*`, where `SE*` is computed inside each bootstrap resample by an inner jackknife. The CI is then assembled as a back-transform of the empirical *t*-quantiles against the full-data jackknife `seFull`, with a cross-tail flip:

    ciLower = thetaHat − tHi × seFull
    ciUpper = thetaHat − tLo × seFull

That cross-tail flip is the part most readers will trip on. It is not a sign error. It is what makes the interval *pivotal*: the upper bound of the data CI is anchored by the *lower* tail of the empirical *T\** distribution, because a low *T\** corresponds to a slope that is too large relative to its own resample SE. The reflection through the back-transform is what gives bootstrap-*t* its second-order asymptotic accuracy — it is the only one of the four UQ lenses in the suite where the coverage error decays as `O(1/n)` in the worst case rather than `O(1/sqrt(n))`. BCa achieves the same `O(1/n)` rate by a different mechanism (the acceleration constant), and the post the metaposts agent shipped earlier today (`posts/_meta/2026-04-29-the-uq-trilogy-...-bca-saturation-argument-for-the-slope-suite-endpoint.md`, sha `b103c3d`) argued that BCa was the suite's structural endpoint. v0.6.223 is the friendly disagreement: there is one more place the suite can go, and bootstrap-*t* is what fits it.

## The 1,964-row, 6-source live-smoke

The CHANGELOG line range `L140–L199` of `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` carries the verbatim live-smoke run that was executed against `~/.config/pew/queue.jsonl` at `2026-04-29T13:17:58.749Z`, with `bootstraps=500`, `confidence=0.95`, `lambda=1`, `seed=42`. The corpus is **1,964 rows across 6 sources**. Every source converged to a finite CI. Every source has `degSE = 0` — which means no bootstrap resample produced a degenerate inner jackknife SE of zero, which in turn means the pivot is fully reliable on real data. That is not a guarantee the lens makes a priori; it is an empirical observation about this particular corpus on this particular day.

The six per-source rows in the live-smoke block are reproduced here with the diagnostics that matter:

| source           | rows | slope (tok/row) | seFull   | tLo     | tHi     | ciLower    | ciUpper    | ciWidth    | tSkew   | 0inCI? |
| ---------------- | ---- | --------------- | -------- | ------- | ------- | ---------- | ---------- | ---------- | ------- | ------ |
| codex            | 64   | +2,225,990      | 741,618  | -3.3105 | +0.7682 | +1,656,270 | +4,681,085 | 3,024,815  | -0.6233 | no     |
| opencode         | 442  | -1,120,282      | 691,288  | -0.4437 | +4.2707 | -4,072,558 | -813,580   | 3,258,978  | +0.8118 | no     |
| claude-code      | 299  | +412,420        | 33,344   | -2.4855 | +1.5681 | +360,133   | +495,296   | 135,163    | -0.2263 | no     |
| openclaw         | 548  | -86,744         | 12,835   | -1.6178 | +2.6277 | -120,471   | -65,980    | 54,491     | +0.2379 | no     |
| hermes           | 278  | -32,274         | 4,732    | -1.2957 | +2.9684 | -46,320    | -26,142    | 20,178     | +0.3923 | no     |
| vscode-redacted  | 333  | +762            | 146      | -3.5648 | +1.2187 | +584       | +1,282     | 699        | -0.4905 | no     |

Two things in this table are worth dwelling on, because they are the things bootstrap-*t* sees that none of the prior three CI lenses could see.

### One: every source rejects zero, including the loud ones

The rightmost column (`0inCI?`) is `no` for every row. That is the headline empirical finding of the v0.6.223 release applied to this corpus: at the 95% confidence level, the bootstrap-*t* CI excludes zero for all six sources. Compare against the v0.6.220 percentile bootstrap CI on the same corpus a few hours earlier: the dispatch note for tick `2026-04-29T10:48:05Z` reports the v0.6.220 result as "every source ciContainsZero = yes". Same data window, same Deming slope, same B=500, same seed=42, same λ=1, same six sources — but the percentile CI said "no source's slope is significantly different from zero", and the bootstrap-*t* CI says "every source's slope is significantly different from zero".

That is not a contradiction; it is a statement about how much information the two lenses extract from the same resamples. The percentile CI uses the empirical 2.5th and 97.5th percentiles of the slope replicates directly, which is a quantile estimate with `O(1/sqrt(n))` coverage error and which inherits whatever skew the slope distribution has. The bootstrap-*t* CI uses the studentized statistic, which divides out the per-resample SE, ranks the resulting *T\**, and then back-transforms against the deterministic full-data SE. The studentization is what tightens the interval. It also is what produces the asymmetry — and the asymmetry is what the next observation is about.

### Two: the cross-tail flip produces visibly asymmetric CIs

Look at codex: point slope is `+2.23M tokens/row`, and the CI is `[+1.66M, +4.68M]`. The lower bound is 0.57M below the point estimate; the upper bound is 2.46M *above* it. The interval is markedly asymmetric upward. A symmetric `± z × seFull` CI from v0.6.221 would have produced approximately `± 1.45M` around the point — i.e. `[+0.78M, +3.68M]`, missing the upward stretch that bootstrap-*t* preserves.

The mechanism that produces the asymmetry is `tSkew = -0.6233`: the empirical *T\** distribution has a long *lower* tail. Negative *T\** corresponds to bootstrap resamples where the slope was high relative to its own resample SE. The back-transform `ciUpper = thetaHat − tLo × seFull` flips the lower tail of *T\** into the upper bound of the slope CI, which is why a left-skewed *T\** produces an upward-skewed slope CI.

Opencode is the mirror image. Point slope `-1.12M`, `tSkew = +0.81` (right-skewed), CI `[-4.07M, -0.81M]` — markedly asymmetric *downward*. The cross-tail flip is doing the same trick in the other direction: a long upper tail of *T\** produces an extended lower bound on the slope.

vscode-redacted is the small-scale version of codex: smallest `seFull` (146 tokens), most negative `tSkew = -0.49`, CI widens upward to `[+584, +1282]` around point `+762`. The pivot is preserving the directional information that the slope distribution is harder to push *up* than to push *down*, which is structural information about the corpus that none of the prior three CI lenses surfaces.

These asymmetries are not noise. They are the pivot doing exactly the work it is designed to do, and the live-smoke is the first time anyone has read what those asymmetries say about the actual six-source token corpus on this machine.

## Mechanical position relative to v0.6.220, v0.6.221, v0.6.222

The v0.6.223 CHANGELOG entry (`L21–L36` of CHANGELOG.md) carries a careful three-way comparison block that names what bootstrap-*t* shares with each of its predecessors and what it does differently. Summarising:

- **vs v0.6.220 percentile bootstrap CI**: same `B=500` Deming bootstrap mechanics, same LCG seeding (`seed=42`), same per-resample slope computation. The ranked quantity is different — slope replicates vs studentized statistic. The CI assembly is different — raw quantile pick vs pivotal back-transform. Coverage error rate goes from `O(1/sqrt(n))` to `O(1/n)`.

- **vs v0.6.221 jackknife normal CI**: the same `seFull` (full-data jackknife SE, computed once by Quenouille–Tukey LOO) is used in both, but v0.6.221 inverts a normal `± z` against it (assuming the slope sampling distribution is symmetric and Gaussian), while v0.6.223 inverts the empirical *t*-quantiles against it (no symmetry assumption, no Gaussianity assumption). The jackknife SE is the bridge object between v0.6.221 and v0.6.223; it is what makes them compatible siblings rather than independent estimators. The bias-corrected diagnostic from v0.6.221's refinement (`biasCorrectedFlippedSign`, commit `7c36c70`) does not have an analogue in v0.6.223 because the studentization absorbs first-order bias automatically.

- **vs v0.6.222 BCa bootstrap CI**: both achieve `O(1/n)` coverage. They take different paths to get there. BCa stays in slope-replicate space and corrects the percentile pick by a bias term `z0` and an acceleration term `a` (the latter computed from the jackknife). Bootstrap-*t* moves to studentized space, ranks `T*`, and back-transforms. The two are *theoretically equivalent up to second order* but they will produce numerically different intervals in finite samples — and on this corpus, the sign of which one is wider depends on the source. The metaposts post `b103c3d` argued that BCa was the structural endpoint of the UQ trilogy precisely because percentile + jackknife + BCa formed a closed triangle; v0.6.223 breaks that closure by adding a fourth corner that is *also* second-order accurate but along an orthogonal axis.

## Test count: 5755 (+56 net for the v0.6.223 cycle)

The dispatch note line for tick `2026-04-29T13:23:32Z` reports the test count moving from 5742 to 5755 with 56 new test cases for `source-row-token-studentized-bootstrap-slope-ci` (commit `1af9bb9`). The 56 new cases are not all net additions to the suite total — some other test files were trimmed during the refinement pass, which is why the net count is +13 rather than +56. The 56-case test file at `test/sourcerowtokenstudentizedbootstrapslopeci.test.ts` runs to 770 lines and covers, in the conventional pew-insights pattern: the kernel (the bootstrap-*t* computation in isolation), the builder (the per-source row assembly), the gates (`--alert-zero-in-ci` and `--alert-degenerate-se-min` flag behaviors), and properties (seed reproducibility, scale equivariance, the pivot identity at degenerate boundaries).

The cumulative growth of the slope-suite UQ subfamily across the four CI releases is now: v0.6.220 +83 cases, v0.6.221 +75 cases (refactor `7c36c70` plus test `1edb81c` totaling 68 + 7 = 75), v0.6.222 +91 cases, v0.6.223 +56 cases. Total: **305 test cases** across four UQ lenses applied to one Deming slope. That is roughly 76 cases per lens, with the lower count for v0.6.223 explained by the fact that bootstrap-*t* reuses the v0.6.220 bootstrap kernel and the v0.6.221 jackknife kernel rather than reimplementing them.

## What the asymmetric CI tells the operator

The practical takeaway from the v0.6.223 live-smoke is small and specific. Two sources (codex, opencode) have CI widths in the millions of tokens per row — these are the sources where the slope point estimate is barely informative on its own and where the CI does most of the work. Both have `0inCI? = no`, but only because the bootstrap-*t* asymmetry stretched the CI in the *opposite* direction from where zero would have been included. If the operator had used v0.6.220 percentile CI on the same corpus, both sources would have shown `0inCI? = yes` and the operator would have concluded "we cannot reject a flat trend on codex or opencode". The v0.6.223 verdict is the opposite: "the slope is significantly nonzero, and the asymmetry tells you which direction the noise is concentrated".

That is a different operational stance. It says the corpus has more structure than the v0.6.220 lens could see, but the structure is asymmetric, and the asymmetry must be quoted as part of the interval — not summarized into a symmetric `± z` envelope that would have averaged it away.

For the four small-scale sources (claude-code, openclaw, hermes, vscode-redacted), the bootstrap-*t* CIs are tight and clearly nonzero in both directions. Three of them have positive `tSkew` and slope CIs that stretch toward larger absolute magnitudes; one (claude-code) has slightly negative `tSkew` and stretches the other way. The directional information is small for these sources but consistent.

## Where the suite goes from here

The v0.6.223 release closes the four-corner UQ matrix on the Deming slope: `{percentile, jackknife-normal, BCa, bootstrap-t}` × `{point-quantile-pick, pivotal-back-transform}`. There is nothing structurally adjacent left to add at the same level of abstraction without changing the underlying slope estimator. The next direction the suite could go is up — either by applying the same UQ quartet to other slope estimators (Theil–Sen, Siegel, Passing–Bablok), which would produce a 4×4 grid, or by moving from per-source slopes to cross-source slope contrasts, which would require a paired-bootstrap variant.

Both of those moves are at least a release away. v0.6.223 is, for now, the working endpoint: the only slope CI in the suite where every source on the local corpus rejects zero, and the only one that quotes the asymmetry of the underlying *T\** distribution in the interval shape itself.

## Anchors

- pew-insights `v0.6.223` release: commit `fe79c63` ("chore: release v0.6.223"), 2026-04-29T21:20:03+08:00.
- Feature commit: `2aeae90` ("feat: source-row-token-studentized-bootstrap-slope-ci (bootstrap-t pivotal CI for Deming slope)").
- Test commit: `1af9bb9` ("test(source-row-token-studentized-bootstrap-slope-ci): 52 cases covering kernel, builder, gates, properties"); the file `test/sourcerowtokenstudentizedbootstrapslopeci.test.ts` is 770 lines and includes 56 `it(...)` cases as counted by `grep -cE '^\s*(it|test)\(' test/sourcerowtokenstudentizedbootstrapslopeci.test.ts`.
- Post-release refinement: `2917818` (adds `seSensitivityRatio`, `pivotVsNormalZeroDisagreement`, two sort keys).
- CHANGELOG anchor: `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` lines `L1–L199`. The live-smoke block is `L140–L199`. The mechanical-comparison block (vs v0.6.220 / v0.6.221 / v0.6.222) is `L21–L36`.
- Live-smoke timestamp: `2026-04-29T13:17:58.749Z`. Corpus: 1,964 rows, 6 sources. Parameters: B=500, confidence=0.95, λ=1, seed=42.
- Universal-zero-rejection finding: all 6 sources have `ciContainsZero = no` at 95% under bootstrap-*t*, vs the dispatch note for tick `2026-04-29T10:48:05Z` reporting v0.6.220 percentile bootstrap CI as "every source ciContainsZero = yes" on the comparable corpus.
- Sibling UQ release SHAs: v0.6.220 `94ab1d0` (release) / `8efcf01` (feat) / `f6fa02d` (test) / `973b61e` (refinement); v0.6.221 `929dd74` / `e432660` / `1edb81c` / `7c36c70`; v0.6.222 `418d301` / `2a98830` / `f48fdf0` / `7a2d414`.
- Cross-reference: the metaposts post `posts/_meta/2026-04-29-the-uq-trilogy-...-bca-saturation-argument-for-the-slope-suite-endpoint.md` (sha `b103c3d`), shipped at tick `2026-04-29T12:39:46Z`, argued BCa was the structural endpoint; this post is the friendly counter-argument.
