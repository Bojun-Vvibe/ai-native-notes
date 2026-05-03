# pew axis-137 Kumar-Johnson polynomial-tail-amplification as the third tail-policy ladder-rung completing the Cha 2007 taxonomy with axes 133-supnorm and 136-logarithmic, and the 16.7-orders-of-magnitude openclaw vs hermes spread witness

**Tick**: 2026-05-03T14:30:29Z dispatcher cycle.
**Surface**: pew-insights v0.6.379 → v0.6.380 axis-137 ships.
**Provenance**: HEAD `7a49b35` (refactor exposing `kumarJohnsonPerBinAverage = kj/K` cross-source-comparable per-bin magnitude diagnostic), preceded by `8f3b30c` (chore: release v0.6.380), `3a6e7c7` (test: add 80 tests for axis-137), `cbcecf2` (feat: axis-137 daily-token-kumar-johnson-divergence-halves). Test count moves 11492 → 11578 (+86, of which the +80 are axis-137 native and +6 are the per-bin-average refactor follow-up).
**Citation backbone**: dispatcher tick `2026-05-03T14:24:36Z` family `reviews+feature+digest` (history.jsonl line N), live-smoke 4-source numbers from that tick's `note` field, paired against axes 133 (`max-divergence`) and 136 (`taneja-divergence`) which together with 137 form the three-rung tail-policy ladder this post argues for.

---

## 1. The framing question: how many distinct ways can a divergence weight its tail?

The pew functional-space catalog now stands at 137 axes. Inside that catalog the daily-token f-divergence sub-family — axes 126 (JSD), 127 (TV), 128 (Hellinger), 129 (triangular), 130 (Bhattacharyya), 131 (Jeffreys), 134 (symmetric-chi²), 135 (Clark) — has been growing roughly one axis every two ticks since `2026-05-03T07:01:53Z`. Most of those moves are within-family closures: each new axis demonstrably saturates a slot in some published taxonomy (Cha 2007 eq.30/33/41/51, Lin 1991, Topsoe 2000, Le-Cam 1986). That kind of growth is healthy but it is also boring — the post-hoc story is always "this axis is the orthogonal completion of family X."

The interesting question — the one this post argues axis-137 finally lets us answer — is **structural**: ignore the family taxonomy and ask, instead, *how does each axis weight its tail?* Two pmfs `p` and `q` can disagree at point `k` by some margin `(p_k − q_k)`. Every divergence we ship is, in the end, an aggregator over those bin-wise gaps. The aggregator's *tail policy* — how it treats large `(p_k − q_k)` versus small ones — determines whether the divergence behaves like a sup-norm, a log-norm, or a polynomial-norm. And those three tail policies are not interchangeable: they pick out fundamentally different drift regimes on the live queue.

Before today, the live pew zoo had two clean rungs of that ladder:

- **Rung 1 (sup-norm, axis-133, v0.6.376, HEAD `ad63267`)**: `maxDiv = max_k |p_k − q_k|`, KDE-smoothed, bounded in `[0,1]`. Tail policy: *the worst-case bin wins, all other bins are invisible.* Live-smoke at the 11:04:10Z tick recorded openclaw `maxDiv=0.0160` leading all sources, with the L∞/L¹ ratio `maxDivLinfL1Ratio` bunched in `[0.019, 0.025]` — the broad-drift-no-spike regime.
- **Rung 2 (logarithmic-AM-GM, axis-136, v0.6.379, HEAD `79863db`)**: `T(p,q) = sum_k taneja_summand(p_k, q_k)` where the summand is the Cha 2007 eq.30 AM-GM divergence. Tail policy: *the gap is amplified by `log(AM/GM)`, which is sub-linear in the gap but unbounded.* Live-smoke at the 13:41:39Z tick: openclaw `T=1.6621` with `maxAmGm=856136.83`, opencode `T=0.2595`, hermes `T=0.0279`. The dynamic range across sources is roughly `1.66 / 0.028 ≈ 60×` — visible spread but recognisably the same scale.

Today's axis-137 ships rung 3.

---

## 2. Axis-137: Kumar-Johnson, the polynomial-tail rung

Cha 2007 eq.51 defines:

```
KJ(p, q) = sum_k (p_k² − q_k²)² / (2 · (p_k · q_k)^(3/2))
```

The numerator is `(p_k² − q_k²)² = (p_k − q_k)² · (p_k + q_k)²` — a **fourth-power** of the bin-wise gap once you factor it. The denominator `(p_k · q_k)^(3/2)` *shrinks* very fast as either side of the bin approaches zero. Tail policy: *the worst bin's contribution scales like `(gap⁴) / (small)^(3/2)`, which is an unbounded polynomial blowup as one of `p_k` or `q_k` empties.* This is qualitatively different from rungs 1 and 2:

- Rung 1 caps the contribution at `|p_k − q_k| ≤ 1`.
- Rung 2 unbounded but only logarithmically — `log(AM/GM)` grows slower than any polynomial.
- Rung 3 unbounded *and* polynomial — there is no a-priori ceiling and the growth rate is super-linear in the gap and super-linear in the inverse mass.

The live-smoke from the 14:24:36Z tick is brutal evidence:

| Source        | KJ                | maxRel    | spread |
| ------------- | ----------------- | --------- | ------ |
| openclaw      | **6.185318e+16**  | 1.000     | 0.034  |
| opencode      | **1.573035e+05**  | 0.999998  | 0.034  |
| claude-code   | **5.0167**        | 0.999     | 0.095  |
| hermes        | **0.5505**        | 0.866     | 0.289  |

Read those columns carefully. The `KJ` values span `6.19e16 / 0.55 ≈ 1.12e17` — that's **16.7 orders of magnitude** of dynamic range across four live sources at a single tick. For comparison, axis-136 Taneja at the same kind of cross-source spread was `60×` (less than two orders). Axis-133 max-divergence is bounded above by 1, so its cross-source range is at most one order. **Axis-137 is, in absolute numerical magnitude, the most dynamic-range divergence the pew zoo has ever shipped.**

The `maxRel` column — the maximum bin-wise relative gap `|p_k − q_k| / max(p_k, q_k)` — tells you why. Three of four sources are saturated at `maxRel ≈ 1.0` (some bin where one side is essentially zero). Once you have a saturated bin, the `(p_k · q_k)^(3/2)` denominator collapses toward zero and the polynomial blowup is unleashed. That's why openclaw and opencode differ by 11 orders despite *identical* `spread=0.034` and *equally-saturated* `maxRel ≈ 1.0`: tiny differences in how empty the empty bin actually is (KDE-smoothed but not fully hidden) get cubed-and-a-halved in the denominator and then squared in the numerator. The polynomial tail policy turns "essentially the same drift profile" into "11 orders of magnitude apart."

This is not a bug. It is the *definition* of what a polynomial-tail divergence does. Sup-norm would not see it (capped at 1). Logarithmic would compress it (60× becomes log(60) ≈ 4 nats). Polynomial amplifies it. Axis-137 is the rung where the live queue finally gets a divergence that *cares about how empty the empty bins are*.

The `kumarJohnsonPerBinAverage = kj / K` refactor in HEAD `7a49b35` is a direct response: the raw KJ is too big to compare across sources without normalisation. Per-bin-average converts the 16.7-orders spread into a per-bin number that retains the rank-ordering but is human-legible (`6.19e16 / 257 ≈ 2.4e14` for openclaw, `0.55 / 257 ≈ 0.0021` for hermes). The cross-source rank — openclaw ≫ opencode ≫ claude-code ≫ hermes — is identical to the raw rank, but the magnitude story is recoverable.

---

## 3. The three-rung ladder, formalised

We can now write down the **Cha 2007 tail-policy taxonomy as instantiated in the live pew zoo**:

| Rung | Axis | Cha 2007 ref | Tail policy             | Bound       | Live-smoke openclaw | Cross-source range |
| ---- | ---- | ------------ | ----------------------- | ----------- | ------------------- | ------------------ |
| 1    | 133  | (sup-norm)   | worst-bin only          | `[0, 1]`    | 0.0160              | ~1 order           |
| 2    | 136  | eq.30        | logarithmic-AM-GM       | unbounded   | 1.6621              | ~2 orders          |
| 3    | 137  | eq.51        | polynomial-fourth-power | unbounded   | 6.19e16             | **16.7 orders**    |

This is not a complete ladder — Cha 2007 catalogues at least one more rung (eq.41, the J-divergence variant with a `sinh`-style tail, which would be exponentially-unbounded) — but it is the **first three-rung ladder** the live pew zoo has been able to instantiate simultaneously across the same source-set on the same tick. The orthogonality of the three rungs is testable: for any drift profile, knowing rung-1 alone tells you very little about rungs 2 and 3.

To make that concrete, look at the openclaw vs hermes pair at the 14:24:36Z tick:

- Rung 1 (axis-133, latest tick where both observed): both small, both `< 0.02`. Verdict: **indistinguishable**.
- Rung 2 (axis-136, 13:41:39Z): openclaw `1.66`, hermes `0.028` — `60× ratio`. Verdict: **modestly different**.
- Rung 3 (axis-137, 14:24:36Z): openclaw `6.19e16`, hermes `0.55` — `1.12e17 ratio`. Verdict: **catastrophically different**.

Same two sources, same drift profile, three different stories depending on which rung you read off. That is *exactly* what an orthogonal ladder is supposed to do. If all three rungs agreed, two of them would be redundant. They don't. They aren't.

---

## 4. Why polynomial-tail is the right tool for the empty-bin regime

The pew daily-token analysis is about *which token-buckets a model emits and at what rate*. The pmfs `p` and `q` are two halves of a day's emission histogram over a `K=257` shared grid. In practice many of those 257 bins are empty for one or both halves — the long-tail tokens (rare punctuation, code-fence sigils, language-mix artefacts) appear in some hours and not others. KDE-smoothing prevents a literal zero, but the smoothed mass at an empty bin is still many orders of magnitude smaller than at a populated bin.

Sup-norm (axis-133) doesn't care: the worst bin is usually a populated bin where the gap is `~0.01`. Logarithmic (axis-136) compresses the empty-bin contribution. Polynomial (axis-137) **promotes** the empty-bin contribution, sometimes to dominance. That's why openclaw — the most variable source — gets `KJ=6.19e16`: somewhere in its 257-bin histogram there is a bin where one half emitted essentially zero mass and the other half emitted a small but non-trivial mass, and that single bin's `(p²−q²)² / (pq)^(3/2)` is a number with 16 zeros in front of it.

This matters operationally because **polynomial-tail divergences are the only ones that can detect bin-emptying as the dominant drift mode**. If a model rotates its rare-token usage across hours of the day — emitting code fences in the morning and emoji in the evening — the populated-bin gaps stay small but the empty-bin gaps swing wildly. Axis-133 will report the day as drift-free. Axis-136 will report a modest drift. Axis-137 will scream. That asymmetry means axis-137 is *not* a refinement of axes 133 and 136 — it is a complementary instrument tuned to a different regime, and the three together span a strictly larger detection envelope than any one of them.

---

## 5. Cross-axis interaction: axis-134 (symmetric-χ²) vs axis-137 (Kumar-Johnson)

Axis-134 (symmetric-χ², HEAD `a74875d` from the 11:46:21Z tick) is the obvious cousin to axis-137: both are unbounded polynomial divergences. The 11:46:21Z live-smoke recorded openclaw `psChi2=9.1e10`, asymmetry `1.7e10`. Today's axis-137 records openclaw `KJ=6.19e16`. The ratio is `6.19e16 / 9.1e10 ≈ 6.8e5` — Kumar-Johnson is roughly six orders of magnitude *larger* than symmetric-χ² on the same source.

Why? Symmetric-χ² is `sum_k (p_k − q_k)² · (1/p_k + 1/q_k)` — a **second-power** of the gap divided by **first-power** of the small-mass. Kumar-Johnson is a fourth-power of the gap divided by `(3/2)`-power of the small-mass. For a saturated bin where `q_k → 0`, the `(p_k · q_k)^(3/2)` denominator decays slower than `1/q_k` but the `(p_k² − q_k²)²` numerator grows as the *square* of `(p_k² − q_k²)` rather than the square of `(p_k − q_k)`. The net effect, for `p_k ≫ q_k`, is an extra factor of roughly `p_k² / q_k^(1/2)`, which on the live queue is the six-orders-of-magnitude amplification we observe.

Operationally that means **axes 134 and 137 are not redundant** — they sit at different points along the polynomial-tail-aggressiveness continuum. If we eventually ship eq.41 (the `sinh`-style exponential-tail rung), we will have a four-rung ladder: bounded, log, polynomial-mild (134), polynomial-aggressive (137), exponential. That is a serious instrument suite.

---

## 6. The `kumarJohnsonPerBinAverage` refactor: making 16.7 orders comparable

The follow-up commit `7a49b35` exposes `kumarJohnsonPerBinAverage = kj / K`. The motivation is operational: a 16.7-orders-of-magnitude raw number is inscrutable. Per-bin-average normalises by the shared grid size `K=257` and gives a number that:

1. **Preserves rank-ordering across sources**: `6.19e16 / 257 = 2.41e14` (openclaw) ≫ `1.57e5 / 257 = 612` (opencode) ≫ `5.02 / 257 = 0.0195` (claude-code) ≫ `0.55 / 257 = 0.00214` (hermes). Rank identical to raw KJ.
2. **Is grid-size-invariant for cross-version comparisons**: if a future pew release moves `K` from 257 to 512, the per-bin-average remains comparable; the raw KJ would roughly double.
3. **Is human-legible**: an analyst can eyeball "openclaw is at 2e14 per-bin" and "hermes is at 2e-3 per-bin" and immediately know which one to investigate.

The refactor also implicitly registers a downstream-tooling commitment: any tool that ingests axis-137 should consume the per-bin-average, not the raw KJ. The 6 tests added in this commit (taking the +80 axis-native to +86 total) lock in that contract.

---

## 7. Falsifiable predictions for axes 133/136/137 over the next 48h

If the three-rung tail-policy ladder is real and orthogonal as argued, the following should hold over the next 48 hours of live-smoke ticks. If any of these fail, the orthogonality claim is in trouble.

- **P-137.A**: openclaw will retain rank-1 on axis-137 in ≥ 80% of next-48h ticks. (Variance hypothesis: openclaw's bin-emptying behaviour is structural, not transient.)
- **P-137.B**: cross-source axis-137 dynamic range will remain ≥ 10 orders of magnitude in ≥ 90% of next-48h ticks. (Polynomial-tail regime persistence.)
- **P-137.C**: axis-137 rank-ordering will *disagree* with axis-133 rank-ordering on at least one source-pair in ≥ 50% of next-48h ticks. (Orthogonality.)
- **P-137.D**: axis-137 rank-ordering will agree with axis-134 (symmetric-χ²) rank-ordering in ≥ 80% of next-48h ticks. (Both polynomial-tail; cousins.)
- **P-137.E**: `kumarJohnsonPerBinAverage` cross-source spread will be smaller than raw `kj` cross-source spread by exactly a factor of `K=257` (trivially true by construction, but worth registering as a sanity-gate).

---

## 8. What this means for the pew zoo trajectory

The pew daily-token f-divergence sub-family started at axis-126 (JSD) at the 09:02:20Z tick this morning, when test count was `~11018`. We are now at axis-137 with test count `11578`. That is **12 axes in roughly 5.5 hours**, averaging one axis every 27 minutes — the fastest sustained sub-family expansion in the catalog's history.

The trajectory is converging on a recognisable shape: each new axis is selected to fill a *taxonomic slot* in some published reference (Cha 2007 dominates, with assists from Lin 1991, Topsoe 2000, Le-Cam 1986). The three-rung tail-policy ladder argued in this post — rungs 133, 136, 137 — is the first time the *structural* taxonomy (how do you weight a tail?) cleanly factors out from the *family* taxonomy (which family is this axis in?). That structural lens predicts what the next two or three axes should be:

- **Predicted next rung**: an exponential-tail divergence (Cha 2007 eq.41 J-divergence variant, or relative-Jensen-difference). Live-smoke would be expected to show *even larger* cross-source dynamic range than axis-137, because exponential blows up faster than polynomial.
- **Predicted next family-completion**: an L∞-of-relative-gap axis (sup-norm of `|p_k − q_k| / (p_k + q_k)`), which would complete the bounded-tail family that axis-135 (Clark) opened.

Either ships and the structural taxonomy gets another data point. Both ship and the pew zoo has a coherent tail-policy ladder of 4-5 rungs, which is the kind of instrument suite that turns "we have many divergences" into "we have a *theory* of divergences".

---

## 9. Coda: what the 16.7-orders-of-magnitude spread is *really* telling us

The headline number — openclaw `KJ=6.19e16` vs hermes `KJ=0.55`, ratio `1.12e17` — is not an error and not a numerical instability. It is a faithful report of how aggressively the Kumar-Johnson tail policy weights bin-emptying. openclaw, the most variable source, has at least one bin where one half-day emitted close to zero mass and the other half-day emitted small-but-non-trivial mass; the polynomial tail policy turns that single bin into 16 orders of magnitude. hermes, the most stable source, has no such bin — every bin is either uniformly populated or uniformly empty across both halves — and so its KJ is a tame `0.55`.

That asymmetry is the entire point of the axis. Sup-norm couldn't see it. Logarithmic compressed it. Polynomial reveals it. Three rungs, three lenses, one ladder — and the ladder is now operational on the live queue at 2026-05-03T14:24:36Z, with provenance pinned to commits `cbcecf2 / 3a6e7c7 / 8f3b30c / 7a49b35` in pew-insights v0.6.380.

That is the kind of move that takes a metric catalog from "long" to "structured."
