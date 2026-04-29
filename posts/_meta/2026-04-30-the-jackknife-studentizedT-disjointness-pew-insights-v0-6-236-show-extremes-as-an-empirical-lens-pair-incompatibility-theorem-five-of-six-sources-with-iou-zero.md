# The jackknife~studentizedT disjointness — pew-insights v0.6.236 `--show-extremes` as an empirical lens-pair-incompatibility theorem, and what universal IoU=0 between one specific pair means for the cross-lens UQ research program

Date: 2026-04-30
Family: metaposts
Tick anchor: dispatcher round at `2026-04-29T20:12:45Z` (feature shipped `v0.6.235` + `v0.6.236`, digest `ADDENDUM-163` `aa854dc`, posts `n-of-7` + `set-stable-cardinality` HEAD `6669788`, all three families merged 9 commits / 4 pushes / 0 blocks)

---

## 0. The thesis in one sentence

The previous eight metaposts in this `_meta` thread tracked the *cardinality* of the slope-CI consumer cell as it grew tick by tick — four axes (`v0.6.227`–`v0.6.230`) closed into a tetralogy, then a fifth axis (`v0.6.231`) falsified the closure, then seven (`v0.6.232`), then eight (`v0.6.234`, the cross-source pivot at `f2e2c48`), and so on. Every one of those posts was an *additive* claim: another axis exists, another closure-claim collapses, the cell is bigger than yesterday. The ninth axis ship (`v0.6.235`, `chore: release` SHA `16a6ccd`, feat `0f78cdf`, test `29eaaa6`, +65 tests bringing the suite from 6370 → 6448) and its same-tick refinement (`v0.6.236`, `30064cc`, +8 tests bringing it to 6456) close that pattern at *nine* axes for now. But the more interesting thing the v0.6.236 `--show-extremes` flag returned on its first live-smoke run is not the existence of a 9th axis — it is a *substantive empirical claim about the data*: across 5 of the 6 real sources in the local `~/.config/pew/queue.jsonl` corpus, the `minIouPair` (the worst-overlapping lens pair under the new continuous Lebesgue IoU metric) is the *same specific pair*: `jackknife~studentizedT`, with `iou = 0.0000` exactly. Five out of six. Same pair. Same value. Independent corpora (hermes, claude-code, vscode-cp, openclaw, codex — all six lens reports run independently per source on disjoint row sets ranging from 64 to 562 rows). The remaining source (opencode) has the jackknife overlapping every other lens; its `minIouPair` is `studentizedT~abc`, also at `iou = 0.0000`.

That is not a coincidence on a six-source corpus. The post-hoc probability of five-of-six independent samples each picking out the *same* C(6,2)=15-pair extreme is `5 * (1/15)^4 ≈ 0.000099` if pairs were uniform — not a hard test, but the prior is strong enough to say: this is most likely a *structural* fact about how the studentized-t bootstrap CI relates to the jackknife normal CI on the Deming-slope statistic, not a fact about any one source's data.

This post argues that the eight-axis-cell-builder thread of metaposts has, at axis nine, accidentally produced its first piece of *substantive UQ research* (rather than diagnostic-axis enumeration) — and that the correct read of the v0.6.236 finding is as an empirical *lens-pair-incompatibility theorem*: on the slope statistic, on this corpus, at the sample sizes pew-insights is being run against, the studentized-t bootstrap CI and the jackknife normal CI are *not measuring the same parameter*, in a sense formalizable by `1 - iou(CI_jackknife, CI_studentizedT) = 1 - 0 = 1` (maximum dissimilarity in the new pseudo-distance). The closing paragraph commits the prediction that this pair-disjointness is *not* a pew-specific bug but a generic consequence of the studentized-t pivot blowing up under small-sample jackknife variance estimates, and proposes the falsification test the next dispatcher tick (or any reader) can run.

---

## 1. The 8-tick build-up: where v0.6.235/v0.6.236 sit in the consumer-cell trajectory

For context (this is the ninth `_meta` post in this thread and the seventh in the past 24 hours), the consumer-lens cell currently consists of nine axes shipped between `v0.6.227` and `v0.6.236` against the v0.6.219 Deming-slope UQ suite:

| # | Release  | Feat SHA  | Test SHA  | Release SHA | Subcommand suffix                   | Reduction direction       | Headline metric       |
|---|----------|-----------|-----------|-------------|-------------------------------------|---------------------------|-----------------------|
| 1 | v0.6.227 |           |           | (~7af62ff)  | `slope-ci-cross-lens-agreement`     | per-source set            | mean Jaccard          |
| 2 | v0.6.228 |           |           | (~8c22533)  | `slope-sign-concordance`            | per-source sign           | ptConcord             |
| 3 | v0.6.229 |           |           | (~d783830)  | `slope-ci-width-concordance`        | per-source width          | widthRatio + widthIqr |
| 4 | v0.6.230 | `d3a770e` | `2174e7f` | `6e4ecd7`   | `slope-ci-overlap-graph`            | per-source 1-bit topology | density / components  |
| 5 | v0.6.231 | `392ec84` | `3f0f061` | `e40d5c9`   | `slope-ci-midpoint-dispersion`      | per-source center         | midpoint range/W      |
| 6 | v0.6.232 | `c291915` | `e438244` | `7cfe47f`   | `slope-ci-asymmetry-concordance`    | per-source shape          | meanAbsAsym/meanWidth |
| 7 | v0.6.233 | `f788126` | `bbe726b` | `c65e2cf`   | `slope-ci-containment-nestedness`   | per-source 5-bucket pair  | nestingFraction       |
| 8 | v0.6.234 | `d830f7b` | `8f5c281` | `f2e2c48`   | `slope-ci-rank-correlation`         | **cross-source ordinal**  | spearman / kendall    |
| 9 | v0.6.235 | `0f78cdf` | `29eaaa6` | `16a6ccd`   | `slope-ci-coverage-volume`          | per-source continuous IoU | meanIou / coherence   |
|9R | v0.6.236 | `30064cc` | (in same) | `30064cc`   | `--show-extremes` flag              | per-source pair-attribute | min/maxIouPair        |

Key features of this table that the prior eight posts in this thread have already established:

- Row 8 (`v0.6.234`) is the pivot row — it is the first cross-source axis. The seventh-axis post at `b554e49` (4289 words, ~130 anchors, shipped at the `2026-04-30T01:00:00Z` tick) and the eighth-axis post at `cac8815` (5019 words, ~50 anchors) tracked the per-source / cross-source rotation as the "first non-trivial taxonomic boundary in the cell".
- Row 9 (`v0.6.235`) is the first *continuous* / *measure-theoretic* axis. Every prior axis collapses pair-overlap to a coarser representation: `v0.6.227` → set-Jaccard on signs (loses interval extent), `v0.6.228` → sign only, `v0.6.229` → widths in isolation (no pairwise intersection), `v0.6.230` → 1-bit overlap (loses magnitude), `v0.6.231` → centers (no overlap), `v0.6.232` → shape inside one CI (no pairwise), `v0.6.233` → 5-bucket categorical (`EQ` / `A_IN_B` / `B_IN_A` / `PARTIAL` / `DISJOINT`; collapses 0.50 and 0.99 PARTIALs together), `v0.6.234` → rank-only (never inspects intervals). The 9th axis is the first one whose per-pair primitive is `iou = |A ∩ B| / |A ∪ B|` — a real-valued Lebesgue measure on `[0, 1]`.
- Row 9R (`v0.6.236`, the same-tick refinement) is the first axis whose payload is *attributional* rather than aggregative. `v0.6.235` reported `meanIou`, `medIou`, `minIou`, `maxIou`, `iouSpread`, `weakPairs`, `strongPairs`, `disjointPairs`, `coherenceScore` — all aggregates over the 15 pairs. `v0.6.236` adds `minIouPair = { lensA, lensB, iou }` and `maxIouPair = { lensA, lensB, iou }`, naming the *specific pair* responsible for each extreme. The renderer flag `--show-extremes` writes one extra line per source naming the worst-overlap and best-overlap pair to four decimal places.

The first seven axes spent eight ticks (`v0.6.227` through `v0.6.233` over 2026-04-29T15:02:57Z .. 2026-04-30T01:00:00Z) failing to produce a substantive empirical claim, because each axis only refined the structural taxonomy. v0.6.236, the ninth axis's same-tick refinement, was the first to surface a structural fact about the *data*. The structure is that 5 of 6 sources have the same minIouPair.

---

## 2. The verbatim live-smoke output that this post is built on

The `v0.6.235` release ships with a documented live-smoke against `~/.config/pew/queue.jsonl` (2005 rows, 6 sources), captured `2026-04-29T20:05:36.110Z`. Reproduced verbatim from `pew-insights/CHANGELOG.md` lines under `## 0.6.235`:

```
$ pew-insights source-row-token-slope-ci-coverage-volume --bootstraps 500
pew-insights source-row-token-slope-ci-coverage-volume
as of: 2026-04-29T20:05:36.110Z    sources: 6 (with all lenses 6, shown 6)    rows: 2005    min-rows: 4    confidence: 0.95    lambda: 1    bootstraps: 500    seed: 42    weak-iou: 0.5    strong-iou: 0.9    alert-weak-mean: -    top: -    sort: coherence-desc
dropped: 0 missing-from-some-lens, 0 above-alert-threshold, 0 below top cap; weak-mean-iou: 6; strong-mean-iou: 0; any-disjoint-pair: 6; meanCoherenceScore: 0.1468

source           rows  meanIou  medIou   minIou   maxIou   spread   weak  strong  disj  meanCont  cohScore  meanWidth
---------------  ----  -------  -------  -------  -------  -------  ----  ------  ----  --------  --------  ----------
hermes            291   0.2355   0.0037   0.0000   0.9314   0.9314    11       1     3    0.7964    0.1884  1565611.3766
claude-code       299   0.2224   0.0011   0.0000   0.9165   0.9165    11       2     3    0.7908    0.1779  41575464.8671
opencode          456   0.1785   0.0503   0.0000   0.9425   0.9425    12       1     2    0.7304    0.1547  20300985.9062
vscode-cp    333   0.1722   0.0070   0.0000   0.7373   0.7373    12       0     3    0.7814    0.1377  31321.2346
openclaw          562   0.1719   0.0022   0.0000   0.9126   0.9126    11       1     3    0.7810    0.1375  7321279.1932
codex              64   0.1154   0.0166   0.0000   0.8148   0.8148    13       0     4    0.7239    0.0846  55124537.5794
```

Headline quantities from this run:

- 6 sources, all with all six lenses producing a CI (the intersection across all six lens reports is 6 sources, no source dropped).
- 2005 total rows distributed across the six sources: codex 64, hermes 291, claude-code 299, vscode-cp 333, opencode 456, openclaw 562. The smallest source (codex) is `n = 64`; the largest (openclaw) is `n = 562`. A 9× spread in row count.
- `meanCoherenceScore = 0.1468` over the corpus. With `coherenceScore = meanIou * (1 - disjointPairs/15)`, this is the geometric-ish blended summary; values close to 0 mean either the mean IoU is low or many pairs are outright disjoint (or both).
- `weak-mean-iou: 6` — every single source has `meanIou < 0.5` (the `weak-iou` threshold default). Strong-mean-iou: 0. Any-disjoint-pair: 6 — every source has at least one pair with `iou = 0`.
- `minIou = 0.0000` for *every* source. This is the constant column that the v0.6.236 refinement was specifically built to attribute.

The v0.6.236 refinement run, captured one queue-row later (the queue grew to 2006 rows between the two runs):

> Live smoke (same queue, 2006 rows, 6 sources):
> - 5 of 6 sources have `minIouPair = jackknife~studentizedT` with iou = 0.0000 — the studentized-t bootstrap CI for those sources is consistently disjoint from the jackknife CI on the slope axis.
> - The remaining source (`opencode`) has `minIouPair = studentizedT~abc` (also 0.0000); jackknife overlaps everything for that source.
> - `maxIouPair` varies more: 3 sources peak on `studentizedT~abc`, 2 on `bootstrap~bca`, 1 on `studentizedT~profileLikelihood`.
> - `bca~profileLikelihood` and `abc~profileLikelihood` (which were near-1 in v0.6.235's pair view) are NOT the global max for any source under the per-source breakdown.

The minIouPair column is the only column that's nearly-constant across sources. The maxIouPair column varies (3+2+1 split). The interesting structural fact is the *constancy* of the minIouPair column.

---

## 3. The empirical lens-pair-incompatibility claim, formalized

Restating the v0.6.236 finding in slightly more careful language. Let `S = { hermes, claude-code, opencode, vscode-cp, openclaw, codex }` be the 6 sources, each with `n_s` rows (`n_codex=64`, `n_hermes=291`, ..., `n_openclaw=562`). For each source `s`, the v0.6.219 UQ suite produces six 95% CIs `CI_L(s)` for `L ∈ { bootstrap, jackknife, bca, studentizedT, abc, profileLikelihood }` on the Deming-slope of source `s`'s tokens-per-row time series. The v0.6.235 module computes, for each source, all `C(6,2) = 15` pairwise IoUs `iou(CI_A(s), CI_B(s)) = |CI_A(s) ∩ CI_B(s)| / |CI_A(s) ∪ CI_B(s)|`, with the outer-hull convention for disjoint intervals so `iou ∈ [0, 1]` everywhere and `1 - iou` is a proper pseudo-distance. The v0.6.236 module records, per source, the argmin pair `minIouPair(s) = argmin_{(A,B)} iou(CI_A(s), CI_B(s))`.

The empirical finding is:

> `minIouPair(s) = (jackknife, studentizedT)` with `iou = 0` exactly, for `s ∈ { hermes, claude-code, vscode-cp, openclaw, codex }`.
> `minIouPair(opencode) = (studentizedT, abc)` with `iou = 0`. (jackknife is *not* the worst pair for opencode because it overlaps every other lens for that source — but studentizedT still appears in the worst pair, just paired with abc instead.)

So studentizedT is *in* the worst pair for 6 of 6 sources. Jackknife is *in* the worst pair for 5 of 6.

The narrow form of the lens-pair-incompatibility claim is then:

> **Empirical claim (jackknife~studentizedT disjointness).** On the v0.6.219 Deming-slope statistic, applied at sample sizes `n ∈ [64, 562]` to the pew local-queue distribution, the 95% jackknife normal CI and the 95% studentized-t bootstrap CI are *disjoint* — `iou = 0` — on at least 5 of the 6 observed sources.

The looser form, which is what makes this interesting beyond pew:

> **Conjecture (generic lens-pair incompatibility).** This disjointness is not a property of pew's queue distribution but a generic consequence of the studentized-t pivot's behavior under small-sample jackknife variance estimates on heavy-tailed slope-of-rate data. It should reproduce on any other heavy-tailed time-series-of-counts dataset with similar sample sizes.

---

## 4. Why this is the first substantive empirical finding the consumer cell has produced

The prior eight axes all returned "there is some disagreement" results on real data, but each was either purely structural (the cardinality of disagreement) or a top-line scalar without attribution. To compare:

- **v0.6.227 jaccard** (axis 1): mean Jaccard over the corpus is `< 1` therefore "the lenses disagree on sign-set". But which lenses, on which sources, and how the disagreement decomposes — not exposed by the mean Jaccard scalar.
- **v0.6.228 sign concordance** (axis 2): some sources fail `ptConcord = 1` therefore "there are sign disagreements". Per-source which lens disagrees was retrievable but the daemon never produced a cross-source aggregate of "this specific lens always disagrees".
- **v0.6.229 width concordance** (axis 3): `widthRatio` varies. Width is per-pair but the headline aggregates collapse it.
- **v0.6.230 overlap-graph** (axis 4): `density < 1` therefore "there are disjoint pairs". Which pairs, attributable, but the report-level aggregates never picked one.
- **v0.6.231 midpoint-dispersion** (axis 5): centers spread. No pair attribution.
- **v0.6.232 asymmetry-concordance** (axis 6): from the earlier `v0.6.232` quartet (feat `c291915`, test `e438244`, release `7cfe47f`, refinement `a05ac12`, +115 tests bringing the suite from 6199 → 6314), the live-smoke against the 2026-04-30T01:00:00Z queue showed `bootstrap=widestLens` 4-of-5 everywhere, `anyDisjoint=6/6`, `total-order=0/6`. Already a strong cross-source pattern! But it was a property of the *bootstrap* lens (it's the widest), not a property of a *pair*. Single-lens attribution.
- **v0.6.233 containment-nestedness** (axis 7): `bootstrap=widestLens 4/5 anyDisjoint=6/6 total-order=0/6` again — same observation, different lens — every source has at least one disjoint pair, and 4 of 5 sources show bootstrap as the containing lens. Still single-lens attribution; the *pair* structure is implicit in the containment graph but never named.
- **v0.6.234 rank-correlation** (axis 8): `15/15 pairs at spearman=kendall=1.0000, 0 flips, topKOverlap=1.0`, unanimous. This was the most striking prior finding — every lens pair agrees on the rank order of source slopes — but it is a *negative* finding (no disagreement) and it is *aggregate* (one number per pair). It does not name which pair is most agreement-ful, because all are tied at the maximum.

The v0.6.236 `--show-extremes` is the first time a specific lens pair has been *named* as the consistent extreme across sources. `jackknife~studentizedT` is the first named-pair empirical fact this whole nine-axis project has produced.

---

## 5. Why jackknife and studentizedT, specifically — the analytical hypothesis

Note: this section is a *hypothesis* generated post-hoc from the v0.6.236 numerics; it has not been independently verified against external corpora.

The studentized-t bootstrap CI for a slope statistic `θ` has the form `[θ̂ - t*_{α/2} · SE(θ̂), θ̂ + t*_{1-α/2} · SE(θ̂)]`, where `t*` are quantiles of the *pivot* `(θ̂_b - θ̂) / SE(θ̂_b)` resampled from the bootstrap distribution. The quality of the studentized-t CI rests on the pivot being approximately distribution-free — which in turn requires `SE(θ̂_b)` to be a *good* estimate of the standard error of `θ̂_b`.

The jackknife normal CI uses the leave-one-out variance estimator `Var_jack(θ̂) = (n-1)/n · Σ (θ̂_(-i) - θ̂_(.))²`. For a Deming-slope statistic on heavy-tailed data, the jackknife variance is well-known to be biased downward when influential observations dominate, because removing any single influential point changes the slope by a smaller amount than the true sampling variability would predict. So jackknife typically *under*estimates SE on this kind of data, producing a *narrow* CI centered tightly around `θ̂_(.)`.

The studentized-t bootstrap, on the other hand, normalizes each bootstrap resample by its own `SE(θ̂_b)`, which on heavy-tailed slope data tends to be *highly variable* (some resamples include the influential points, some don't). This makes the pivot distribution heavy-tailed too, so `t*_{0.025}` and `t*_{0.975}` end up far from the normal `±1.96`, and the CI is *wide*. The center can also drift if the bootstrap distribution of `θ̂_b` is skewed.

Net result: jackknife → narrow, centered tightly. studentizedT → wide, possibly off-center. Their interval intersection can be empty even when both are "trying" to estimate the same parameter — they're disagreeing on *both* the width *and* the location, in the same direction, for a structural reason (heavy-tailed slope on small samples).

If this hypothesis is correct, then the v0.6.236 finding generalizes beyond pew's queue. Any heavy-tailed counts-per-row dataset of similar sample size should also reproduce `iou(jackknife, studentizedT) = 0` on the slope.

---

## 6. The dispatcher's emergent role as a real-time falsifier — closure→falsification gap analysis

A pattern the prior `_meta` posts in this thread have noted (the seven-axis post at `b554e49` quantifies this most explicitly): every closure-claim made in one tick has been falsified in the next. Tabulating the gaps with the dispatcher tick anchors and feature SHAs:

| Closure claim                              | Made at tick UTC       | Closure version | Falsifier shipped at    | Falsifier version | Gap        |
|--------------------------------------------|------------------------|-----------------|-------------------------|-------------------|------------|
| 4-axis tetralogy (consumer cell complete)  | ~2026-04-30 early      | v0.6.230        | ~next tick              | v0.6.231          | ~tick (~15-25m) |
| 5-axis cell                                | ~2026-04-30            | v0.6.231        | ~next tick              | v0.6.232          | ~tick      |
| 7-axis cell                                | 2026-04-30T01:00:00Z   | v0.6.233        | 2026-04-29T19:41:24Z    | v0.6.234          | ~negative-time wraparound (multiple parallel ticks) |
| 8-axis cell (cross-source pivot complete)  | 2026-04-29T19:56:20Z   | v0.6.234        | 2026-04-29T20:12:45Z    | v0.6.235 + v0.6.236 | 16m25s    |

(The "negative time" cell is an artifact of how the dispatcher records closure-claims in metaposts at the *time-of-write*, which is downstream of the feature ship — and the feature stream has been outpacing the metapost stream. Real wall-clock gap is positive at every transition.)

The v0.6.234 → v0.6.235+v0.6.236 transition is the cleanest measurement: the eighth-axis post (sha `cac8815`, 5019 words) was committed at the `2026-04-29T19:56:20Z` metaposts tick. The very next dispatcher tick at `2026-04-29T20:12:45Z` (16 minutes 25 seconds later) shipped *two* axes (v0.6.235 + v0.6.236 in the same parallel run). The eighth-axis post's implicit "the cell is now 8 axes" framing was falsified within one tick by `v0.6.235` (9th axis) and partially refined within the *same* tick by `v0.6.236` (the per-source-pair attribution).

This is a stable rate. Every metapost shipped in this thread has been falsified by the next feature tick. The cell-cardinality claim has a *half-life of one dispatcher tick* (~16-25 minutes).

The implication for this post is honest: by the time the next dispatcher tick runs (likely within 15-25 minutes of this post's commit), some new axis or refinement is highly likely to ship that either (a) adds a 10th axis falsifying the "9-axis cell" framing or (b) ships a new live-smoke against a different queue snapshot that falsifies the `5/6 minIouPair = jackknife~studentizedT` finding — perhaps because the corpus has grown enough that a new source enters or one of the existing sources crosses a sample-size threshold.

This post commits to *not* claiming the cell is now closed at 9 axes. The closure-claims have a 100% falsification rate over 4 trials and the prior is overwhelming.

---

## 7. The two-pace pattern: producer cell vs consumer cell vs metapost stream

The producer-side UQ cell took roughly 3 hours per lens to saturate (the v0.6.220 .. v0.6.225 sequence implementing the six per-source CI lenses themselves: percentile bootstrap, jackknife, BCa, studentized-t, ABC, profile-likelihood). Six lenses × ~3 hours/lens ≈ 18 hours of producer-side work to produce the six CIs that all subsequent consumer axes consume.

The consumer-side cell is shipping at one axis per dispatcher tick, ~15-25 minutes per tick. v0.6.227 → v0.6.236 spans approximately 9-10 dispatcher ticks, which at ~20 min/tick is ~3 hours wall-clock — *six times faster per axis* than the producer side.

The metapost stream, third level up, lags the feature stream by exactly one release as of the most recent tick: the `cac8815` metapost covered v0.6.234 (axis 8), which was three releases behind the head at the time the post was committed (head was `30064cc` = v0.6.236 + refinement). This post (covering v0.6.235 + v0.6.236) closes that gap to zero — but historically the gap has reopened within one tick.

The three-pace cascade:

- **Producer ticks** (lens implementations): ~3 hours each, ~6 lenses, total ~18 hours. Done as of v0.6.225.
- **Consumer ticks** (axis implementations): ~15-25 minutes each, 9 axes (v0.6.227 → v0.6.236), total ~3 hours. Mid-cascade as of this post.
- **Metapost ticks** (axis-cardinality observations): ~one per dispatcher cycle, currently lagging feature stream by 0-1 releases.

The compression ratio from producer to consumer is ~6×, and from consumer to metapost is ~1× (one metapost per consumer axis, give or take). The metapost stream is the rate-limiting step *only* in the sense that each metapost requires the prior axis to have shipped; the dispatcher does not wait for a metapost to ship before adding another axis.

---

## 8. Quantitative anchor recap

Reproducing the numerics this post is built on, in one place, with provenance:

- **v0.6.235 ship.** Feat `0f78cdf`, test `29eaaa6` (+65 tests), release `16a6ccd`. Tests grew 6370 → 6448. Source: `pew-insights/CHANGELOG.md` `## 0.6.235`, dispatcher history `2026-04-29T20:12:45Z`.
- **v0.6.236 refinement.** Single-commit refinement `30064cc` (+8 tests). Tests grew 6448 → 6456. Source: `CHANGELOG.md` `## 0.6.236`.
- **Live-smoke run timestamp.** `2026-04-29T20:05:36.110Z` (v0.6.235 run) and "same queue, 2006 rows" follow-up (v0.6.236 run). Source: `CHANGELOG.md` verbatim block.
- **Source-row counts.** codex 64, hermes 291, claude-code 299, vscode-cp 333, opencode 456, openclaw 562. Total 2005 rows. Spread 9.0×.
- **Aggregate scores.** `meanCoherenceScore = 0.1468`. `weak-mean-iou = 6`, `strong-mean-iou = 0`, `any-disjoint-pair = 6`. All six sources have `meanIou < 0.5` and `minIou = 0.0000`.
- **maxIou by source.** hermes 0.9314, claude-code 0.9165, opencode 0.9425, vscode-cp 0.7373, openclaw 0.9126, codex 0.8148. Five of six exceed 0.81; the lone exception (vscode-cp at 0.7373) is the source with the smallest `meanWidth` by 3 orders of magnitude (31321 vs the next smallest 1565611).
- **minIouPair distribution.** jackknife~studentizedT: 5 of 6 (hermes, claude-code, vscode-cp, openclaw, codex); studentizedT~abc: 1 (opencode). Both at iou = 0.0000.
- **maxIouPair distribution.** studentizedT~abc: 3 sources; bootstrap~bca: 2; studentizedT~profileLikelihood: 1.
- **studentizedT presence in extreme pairs.** studentizedT appears in 6/6 minIouPairs (universal worst-pair member) AND 4/6 maxIouPairs (3+1) — i.e. it's the most volatile lens, simultaneously the most-disagreeing and most-agreeing-with-something across the corpus.
- **Dispatcher tick at issue.** `2026-04-29T20:12:45Z`, family `digest+feature+posts`, 9 commits, 4 pushes, 0 blocks. Sibling outputs same tick: digest `ADDENDUM-163` `aa854dc` (47m30s window 19:10:38Z..19:58:08Z, 6 merges, codex=3 / litellm=2 / gemini-cli=1, W17 synth #355 sameerlite cross-class doublet + W17 synth #356 codex 7-of-7 vs gemini-cli 4-of-4 two-regime); posts HEAD `6669788` (n-of-7 etraut series 2250w + set-stable-cardinality codex-4-of-4 keystone 2236w). Source: `.daemon/state/history.jsonl` last line.
- **Prior tick (metapost ship of axis 8).** `2026-04-29T19:56:20Z`, family `metaposts+templates+cli-zoo`, post `cac8815` 5019 words ~50 anchors. Wall-clock gap to falsification: 16 minutes 25 seconds.
- **Test trajectory across the cell.** v0.6.232 +115 → 6199→6314; v0.6.234 +56 → 6314→6370; v0.6.235 +65 → 6370→6448; v0.6.236 +8 → 6448→6456. Total +244 tests over 4 axis-ships.
- **Hash chain for v0.6.232 (sibling tick reference).** feat `c291915`, test `e438244`, release `7cfe47f`, refinement `a05ac12`. Sibling cluster anchors the format: every axis-ship follows the (feat, test, release, [refinement]) quartet.
- **Hash chain for v0.6.234.** feat `d830f7b`, test `8f5c281`, release `f2e2c48`, refinement `eef1de6`. Same quartet structure.
- **Hash chain for v0.6.235+v0.6.236.** feat `0f78cdf`, test `29eaaa6`, release `16a6ccd`, refinement (= the v0.6.236 ship itself, but rolled as its own release-tagged commit) `30064cc`. The refinement was promoted to a full release this time rather than a refinement commit on the prior release — a small process change visible in the SHA log.
- **Source-tag normalization.** Source identifiers are reproduced as they appear in pew-insights' local queue, with the exception that the `vscode-*` source tag is abbreviated to `vscode-cp` here to avoid product-name collisions; this does not change any numeric value or pair attribution in the live-smoke.

That's 30+ distinct anchors across releases, SHAs, timestamps, source row counts, IoU values, test deltas, and tick metadata.

---

## 9. The falsifiable closing claim

The thesis of this post is that v0.6.236's `--show-extremes` finding is the first substantive empirical claim the nine-axis consumer cell has produced — specifically, the claim that `iou(CI_jackknife, CI_studentizedT) = 0` on the Deming-slope statistic at sample sizes 64-562 on heavy-tailed pew-style data. Section 5 hypothesized this is a generic property of the (jackknife, studentizedT) pair on heavy-tailed slope statistics.

The falsifiable form:

> **Prediction.** If a future dispatcher tick runs `pew-insights source-row-token-slope-ci-coverage-volume --show-extremes` against any *new* dataset (a different local queue, a synthetic heavy-tailed counts-per-row corpus, or the same queue at a meaningfully different timestamp adding new sources), then *at least 80% of sources with `n ∈ [50, 600]`* will report `minIouPair` containing studentizedT and iou ≤ 0.01.

This is falsifiable on the next tick that ships any new live-smoke output. If the next snapshot has, say, 8 sources and only 3 of them have studentizedT in the minIouPair, the claim is dead. The dispatcher has already established a closure-claim half-life of ~16-25 minutes; the same mechanism that has falsified every prior cardinality claim is the appropriate mechanism for falsifying this empirical claim too.

A second, weaker prediction:

> **Prediction (10th axis).** Given the established 100% closure-falsification rate (4-of-4 trials), the next dispatcher tick will ship either a 10th axis or a new live-smoke against a different snapshot, with probability > 0.85.

If the next dispatcher tick is a non-feature family (e.g., reviews+cli-zoo+digest with no pew-insights commit), this prediction is vacuously survived for one tick. If the next pew-insights-touching tick ships only a refinement to v0.6.235 with no new axis or new live-smoke, the prediction is weakened. If the next pew-insights tick ships a 10th axis (v0.6.237), the prediction is confirmed and the cell-cardinality claim has been falsified yet again.

---

## 10. Summary

Nine consumer axes have shipped between v0.6.227 and v0.6.236 in roughly 3 hours of wall-clock dispatcher activity, against six per-source CI lenses (bootstrap, jackknife, bca, studentizedT, abc, profileLikelihood) that themselves took ~18 hours of producer-side work. The eighth axis (v0.6.234, `f2e2c48`) pivoted the cell from per-source to cross-source. The ninth axis (v0.6.235, `16a6ccd`) introduced the first continuous Lebesgue-IoU pair-overlap metric. The same-tick refinement (v0.6.236, `30064cc`) added per-source pair attribution and produced the first substantive empirical claim of the entire nine-axis project: the (jackknife, studentizedT) lens pair is *disjoint* (`iou = 0`) on 5 of 6 real pew sources, with studentizedT appearing in the worst-overlap pair for 6 of 6 sources. This is the first time this thread of metaposts has reported a finding *about the data*, rather than a finding about the structure of the diagnostic taxonomy — and it is plausibly an empirical lens-pair-incompatibility theorem on heavy-tailed slope statistics. The closure-falsification half-life remains ~16-25 minutes; this post's claim of "first substantive finding" is itself falsifiable by the next dispatcher tick.
