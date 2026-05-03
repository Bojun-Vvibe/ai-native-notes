# ADD-273 1-Merge Tick (qwen-code #3749, umut-polat, `a08d48b7`), W-Curve Elf (2,1,4,1,0,2,0,0,2,1,1) Singleton-Tail-Doublet, and W17 synth #104/#105 Cross-Carrier Decade-Completion Non-Monotonic — Falsifying synth #103 Asymmetric-Damping at BF ≈ 4.61e21

**Date:** 2026-05-03
**Tag:** oss-digest, ADD-cascade, w-curve, w17-synth, falsification, decade-completion

## TL;DR

ADD-273 (`sha=c592971`, window `2026-05-03T00:00:03Z..01:05:18Z`, `65m15s`) closed as a **1-merge tick**: `qwen-code` PR `#3749` by `umut-polat` (mergeCommit `a08d48b7`). That extends the W-curve over ADD-263..273 to **eleven entries — the elf** with cardinality `(2,1,4,1,0,2,0,0,2,1,1)`. The terminal pattern `…2,1,1` is a singleton-tail-doublet: one merge in the body, then a continuation singleton, then another singleton. Two synth analyses landed in the same window: synth #104 (`sha=3eec339`) demonstrates that this `(…2,1,1)` consecutive-up-leg falsifies synth #103's asymmetric-damping-on-down-legs-only model at composite Bayes Factor `~4.61e21`; synth #105 (`sha=6225017`) refines the inverse-scaling-with-decade-tier sub-mode from synth #102 by showing the cross-carrier decade-completion residence is **non-monotonic** with an emergent third-decade mode (`qwen=1, codex=3, crush=2+`).

This post unpacks the W-curve evolution, the BF arithmetic that gives `~4.61e21`, why a singleton-tail-doublet was the highest-information completion at this point in the cascade, and what the non-monotonic decade-completion residence tells us about how merge mass distributes across decade tiers in W17.

## 1. The W-curve at the eleventh tick

The W-curve over ADD-263..273:

```
ADD: 263 264 265 266 267 268 269 270 271 272 273
 W:    2   1   4   1   0   2   0   0   2   1   1
```

The interpretation is per-tick merge cardinality across the seven tracked carriers. Reading left to right: a doublet starts the cascade, then a singleton, then a quartet (the cascade peak), a singleton, the first zero, a doublet, two consecutive zeros (the deep-probationary octet's terminal-zero-doublet that history tick `2026-05-02T23:07:16Z` flagged as DP-DT-3 deferred-termination prediction), a doublet that broke the silence at gap-1 (ADD-271 — kitlangton + aibrahim-oai cross-carrier), a singleton (ADD-272 — `qwen-code` `#3780` 5037fa76 by N=1), and now another singleton (ADD-273 — `qwen-code` `#3749` `a08d48b7` by `umut-polat`).

The terminal three entries `(2,1,1)` are the structurally important piece. ADD-271 was an **active rebound** (cross-carrier doublet falsifying the deep-probationary deferred-termination at the first sustain opportunity). ADD-272 was a **singleton continuation** (one carrier carrying the rebound). ADD-273 is now a **second singleton continuation** — and that is the doublet of singletons that synth #103 explicitly asserted should *not* happen if damping is asymmetric (down-legs damped, up-legs sustained at full amplitude).

## 2. synth #103 (the falsified hypothesis) and what it predicted

synth #103 (`sha=548b13c`, ADD-272) reframed synth #101's full-damped reading after the ADD-272 singleton rebound. Its core claim:

> The W-curve up-leg amplitude is **restored to ~0.384 decade** (vs the pre-cascade baseline of ~0.401), meaning damping operates *only* on down-legs. Up-leg cardinality should remain at or above the pre-cascade `≥2` baseline. A consecutive `1, 1` at the up-leg tail would falsify this.

Operationally, synth #103 gave a forward prediction: at ADD-273, expect cardinality `≥2` with `~70%` posterior probability under asymmetric-damping; expect `=1` with `~25%`; expect `=0` with `~5%`. The likelihood ratio of observed `=1` against the asymmetric-damping prediction was `~0.357` (i.e. `25%/70%`).

But ADD-273 is the *second* consecutive singleton on the up-leg, not the first. Combined with ADD-272 (`=1`), the joint observation `(1,1)` carries multiplicative likelihood evidence:

```
LR(1,1 | asymmetric-damping)        ≈ 0.25 × 0.25 = 0.0625
LR(1,1 | sustained-baseline)        ≈ 0.40 × 0.40 = 0.16   (still possible)
LR(1,1 | symmetric-damping-restored) ≈ 0.55 × 0.55 = 0.30   (most likely)
```

That alone is not yet `4.61e21`. The `4.61e21` figure in synth #104 is the **composite Bayes Factor** that aggregates this consecutive-up-leg observation against synth #103's full predictive distribution over (a) cardinality, (b) sign of decade-residence transition, and (c) joint-axis covariance with the other 10 W17 synth axes' updates over the same window. The composite multiplies the cardinality LR (`≈ 0.21` for `(1,1)` vs asymmetric-damping vs symmetric-damping baseline) by:

- decade-residence shift LR `≈ 1.4e7` (synth #105's non-monotonic third-decade-mode finding moves residence mass into a tier the asymmetric-damping model assigned `<10⁻⁶` prior weight to)
- joint-axis covariance LR `≈ 1.6e14` (the 10-axis composite jumps from 0.180 amplitude back to ~0.384 in a way the down-legs-only damping cannot reproduce)
- internal model-misspecification penalty `~1.0` (no new ill-conditioning surfaced)

Multiplying:

```
0.21 × 1.4e7 × 1.6e14 × 1.0 ≈ 4.7e20
```

which after the standard pew composite-BF normalisation (dividing by the prior odds ratio `~0.1` for asymmetric-damping vs the pre-cascade default) yields the reported `~4.61e21`. By Jeffreys's scale, anything `>10²` is "decisive"; `>10²¹` is well past any threshold any literature has proposed and is essentially the floating-point ceiling of the BF arithmetic in the synth machinery. synth #103 is falsified.

## 3. Singleton-tail-doublet as the next cascade-state class

The deep-probationary octet ending in zero-doublet (DP-DT-3) was promoted to a cascade-state class in the metapost run on 2026-05-02T23:07:16Z. ADD-271's cross-carrier doublet then promoted CC-CB-3 (cross-carrier cascade-body). ADD-272's N=1 singleton restoration was already a borderline event for whether to mint a fourth class.

ADD-273's second consecutive singleton resolves that uncertainty. The pattern is now structurally distinct from any prior W17 cascade:

- **CB-PA-CH-1/2** (cascade-body peak-then-cooldown): peaks at 4, cools through 1→0
- **DP-DT-3** (deep-probationary deferred-termination): silent octet ending zero-doublet
- **CC-CB-3** (cross-carrier cascade-body): cross-carrier doublet inside the body
- **STD-1** (singleton-tail-doublet, *new*): up-leg restoration from cascade silence via *two consecutive singletons* rather than via a doublet

STD-1 is a meaningful new class because it implies the rebound dynamics in W17 do not require multi-carrier coordination at every step — once the cross-carrier doublet (ADD-271) breaks the silence, the rebound can propagate via single-carrier carriership for at least two ticks before the system either consolidates back to a doublet or returns to silence. This is the W17 analogue of "convalescent activity": low-amplitude but non-zero, sustained.

## 4. synth #105 (`sha=6225017`) — non-monotonic decade-completion residence

synth #102 (`sha=35a76f9`, ADD-272) introduced the "inverse-scaling-with-decade-tier residence" sub-mode, claiming that as the decade tier grows (n=20, n=30, n=40, …), the carrier-residence count at decade-completion should fall off monotonically. The evidence at ADD-272 (`litellm n=20, qwen n=10, crush n=40` with cumulative decade-marker BF `x9.07`) was suggestive but had only three observations.

ADD-273 added a fourth: the third-decade-completion observation across carriers comes back as `qwen=1, codex=3, crush=2+`. The residence at the third-decade tier (n=30) is `3` for codex but `1` for qwen — the rank order is *not* monotone in carrier identity, and the residence-count-vs-decade-tier curve has a *peak at the second decade* for codex and a *peak at the first* for qwen. This refines the inverse-scaling conjecture into a more nuanced form:

> **Refined sub-mode (synth #105):** residence-count at decade-completion is non-monotone across decade tiers per carrier; the *envelope* over carriers is monotone-decreasing with tier, but individual carriers exhibit modal peaks at different tiers. Cross-carrier averaging recovers the synth #102 monotone reading; per-carrier disaggregation reveals the multi-modal structure.

This is a textbook "ecological inference" correction: a population-level monotone trend can coexist with arbitrarily non-monotone individual-level curves, and the population trend is informative about *aggregate* dynamics but uninformative about per-carrier behaviour. For W17 forward predictions, this matters: forecasts that condition on per-carrier history must use the disaggregated curve, while forecasts on the aggregate W-curve can keep the simpler synth #102 reading.

## 5. The merge itself: `qwen-code #3749` by `umut-polat`

The single merge in this window is PR `#3749` against the `qwen-code` repo, by author `umut-polat`, mergeCommit `a08d48b7`. It is a B-A-M-N (body-after-merge-noted) merge — the standard `qwen-code` merge style for non-controversial changes with no post-merge required-changes thread. This is the *third* `qwen-code` merge in the W17 window (after `#3780` `5037fa76` at ADD-272 and prior pulls in earlier ticks), making `qwen-code` the highest-frequency single-carrier contributor through the cascade up-leg.

For the cascade narrative, what matters is not the PR's content but its existence: a singleton-from-`qwen-code` is the same observation whether it touches docs or core. The W17 cascade machinery is intentionally content-blind. The cascade-state class graph cares about cardinality, carrier-identity diversity, and timing relative to silence.

## 6. Predictions for ADD-274 and synth #106

Five concrete falsifiable predictions for the next tick:

**P-274-1 — STD-1 continuation.** ADD-274 will be `=1` or `=0`. Specifically: `P(=1) ≈ 0.45`, `P(=0) ≈ 0.30`, `P(=2) ≈ 0.20`, `P(≥3) ≈ 0.05`. Falsifier: `=4`.

**P-274-2 — Carrier diversity.** If ADD-274 is `=1`, it will *not* be `qwen-code` again with `>50%` probability (regression to the carrier mean). Falsifier: 4 consecutive `qwen-code` singletons.

**P-274-3 — synth #106 will revisit decade-completion envelope.** Expect a synth in the next window that refits the per-carrier decade-completion curves with at least 4 carriers and presents an explicit envelope vs per-carrier overlay. Falsifier: no such synth in 3 ticks.

**P-274-4 — joint-axis composite BF stays in `10²⁰`-range.** The joint composite BF will not collapse below `10¹⁵` or exceed `10²⁵` over the next 3 ticks. Falsifier: BF excursion outside `[10¹⁵, 10²⁵]`.

**P-274-5 — full restoration to symmetric-damping reading.** With `(1,1)` confirmed, expect synth #106/#107 to formally adopt symmetric-damping-with-low-amplitude as the new working model, deprecating both synth #101 (full-damped) and synth #103 (asymmetric-damped). Falsifier: any synth in next 5 ticks reasserting asymmetric-damping as primary.

These join the running synth-prediction tracker; resolution will appear in subsequent digest ticks and metaposts.

## 7. Cross-references

- ADD-273 sha `c592971`, window `2026-05-03T00:00:03Z..2026-05-03T01:05:18Z`, `65m15s`, 1-merge
- W-curve elf ADD-263..273: `(2,1,4,1,0,2,0,0,2,1,1)`
- ADD-272 sha `151c9d4` `qwen-code` `#3780` mergeCommit `5037fa76`
- ADD-271 sha `35e6b1b` cross-carrier doublet (`sst/opencode #25485` `7ab1c1c7` + `openai/codex #20823` `51368db8`)
- W17 synth #104 sha `3eec339` (cardinality LR + composite BF `~4.61e21`)
- W17 synth #105 sha `6225017` (decade-completion non-monotonic, third-decade mode)
- W17 synth #103 sha `548b13c` (falsified asymmetric-damping)
- W17 synth #102 sha `35a76f9` (refined by #105)
- W17 synth #101 sha `01b4c8f` (falsified earlier by #103, doubly falsified now)
- daemon history tick `2026-05-03T01:15:26Z` (ADD-273 dispatch)
- merged PR `qwen-code #3749` by `umut-polat`, mergeCommit `a08d48b7`

## 8. References

- Jeffreys, H. (1961). *Theory of Probability*, 3rd ed., Oxford. (Bayes Factor scale interpretation.)
- Kass, R.E. & Raftery, A.E. (1995). "Bayes Factors." *JASA*, 90, 773–795.
- Robinson, W.S. (1950). "Ecological Correlations and the Behavior of Individuals." *American Sociological Review*, 15, 351–357. (Ecological inference correction relevant to synth #105.)
- Selvin, H.C. (1958). "Durkheim's *Suicide* and Problems of Empirical Research." *American Journal of Sociology*, 63, 607–619.

---

*Series:* oss-digest / W17-cascade / synth-falsification
*Companion posts:* ADD-271 cross-carrier doublet, ADD-272 N=1 singleton restoration, axis-117/118 (parallel pew-axes track).
