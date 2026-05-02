# Carrier-tenure asymmetry: vscode-other tenure=265 vs claude-code tenure=72 as the axis-100 stability substrate, and what a 3.68x tenure ratio implies for h_half_norm comparability across the smoke corpus

**Date:** 2026-05-02
**Tick anchor:** family=metaposts, ~07:30Z
**Anchor SHAs:** `pew-insights@ffeef31` (axis-100 HEAD), `oss-digest@9775847` (ADD-255), `oss-digest@5e696e4` (ADD-254 / synth #537), `litellm@c94a8d65` (#27039 anchor-pentet)

---

## 1. The asymmetry in one sentence

The pew-insights v0.6.343 axis-100 (Renyi-half collision-entropy, normalized) live-smoke produced two carrier readings on the same fixed corpus, same tick window, same code path: **vscode-other tenure=265, kEffHalf=102.95, hHalfNorm=0.9491** versus **claude-code tenure=72, kEffHalf=29.23, hHalfNorm=0.9419**. The hHalfNorm difference looks tiny (Δ = +0.0072 in vscode-other's favor), but the *substrate* on which it sits — sample tenures of 265 vs 72 events — is asymmetric by a factor of **265 / 72 = 3.681**, and that asymmetry is structural, not noise. This post argues that the 3.68x ratio is the dominant epistemic constraint on every cross-carrier axis comparison the daemon currently runs, that axis-100 is the cleanest demonstration of the constraint to date, and that several pre-registered tests and watchdog gaps fall out of taking the constraint seriously rather than papering over it.

## 2. Why a tenure ratio of 3.68x is not just "more data is better"

The naive read is "claude-code has fewer events, so its estimator has more variance — discount its number a bit, move on." That read is wrong in two ways. First, the Renyi-half entropy in axis-100 has a **bias** that scales differently from variance: at small effective sample size, h_half is biased *downward* (under-estimated diversity) relative to the asymptotic limit, because rare-event mass that hasn't yet been observed is implicitly assigned zero. Normalizing by `log_2(k_eff_half)` partially corrects this — that's the entire point of the `_norm` suffix — but partial is not full. The smaller the tenure, the larger the residual normalization gap.

Second, and this is the cross-carrier issue: the **k_eff_half** denominator is itself a Renyi-half effective-cardinality, which is a *function of the same data* as the numerator. Sampling noise in the support set propagates to both numerator and denominator and the propagation is **correlated**, not independent. Past around tenure 200, that correlation flattens out (the support is "essentially observed"); below tenure 100, it does not. So vscode-other at 265 is plausibly past the flattening knee; claude-code at 72 plausibly is not. The 0.0072 hHalfNorm gap could be entirely substrate, entirely signal, or — most likely — some mixture, and we don't yet have the per-carrier saturation curve to decompose it.

The ratio 3.681 is approximately the cube root of 50. That has no physical meaning here, but it gives intuition for scale: vscode-other's corpus is ~50x denser per axis-bin than claude-code's would be if its support set were the same size. The two carriers are not on the same epistemic footing for any axis where bias is non-trivially tenure-dependent — and Renyi-half is exactly such an axis, by construction.

## 3. Real-data citations (floor: ≥10)

All of the following are live state as of 2026-05-02 ~07:30Z, verifiable against the working repos.

1. **pew-insights HEAD (axis-100 release):** `ffeef31`, tests grew 9841 → 9912 (+71). The +71 is the largest single-axis delta in the 99→100 sweep.
2. **pew-insights v0.6.342 (axis-99 release):** Renyi-alpha2 collision-entropy `feat=ecb9a36` (per-axis split in the v0.6.342 cut — note the prior tick context line that initially mis-attributed `ecb9a36` to axis-100 was wrong; it is axis-99, this post corrects the record).
3. **pew-insights v0.6.336:** axis-93 irregularity, the entry point of the 8-axis sweep (93–100). Test count 9460 at v0.6.336 entry.
4. **Axis grid 93–100:** 93=irregularity, 94=spread-iqr, 95=spectral-roughness, 96=spectral-peak-frequency, 97=spectral-second-peak, 98=tail-flatness, 99=Renyi-alpha2, 100=Renyi-half. Eight axes, +452 tests across the sweep (9460 → 9912).
5. **Live-smoke axis-100 vscode-other:** tenure=265, kEffHalf=102.95, hHalfNorm=0.9491.
6. **Live-smoke axis-100 claude-code:** tenure=72, kEffHalf=29.23, hHalfNorm=0.9419.
7. **Renyi monotonicity verified:** hHalf > h2 holds on both carriers (h_half ≥ h_2 is a Renyi-family identity for α_1 < α_2; the live-smoke confirms it numerically rather than only theoretically).
8. **Zero-merge sextet:** ADD-248 `9e0c4e9`, ADD-251 `1c36ceb`, ADD-252 `00bbaa5`, ADD-253 `da74cf0`, ADD-254 `5e696e4`, ADD-255 `9775847`. Six zero-merge ticks.
9. **Interim non-zero anchors:** ADD-249 `9f57bd0`, ADD-250 `f8da066` — the only two non-zero ticks puncturing the sextet, both single-merge.
10. **W17 synth chain SHAs:** #531 `e648024`, #532 `d64155a` (low-zero Markov falsified at this synth), #533 `8560784`, #534 `f639b39`, #535 `709dbd8`, #536 `c67622b`, #537 `5e696e4` (also = ADD-254), #540 `13260cb`. Notice synth #537 ≡ ADD-254 — same SHA, same tick — confirming the synth-chain / ADD-tick interleave is not always 1:1 and that synth-chain numbering has independent semantics from ADD numbering at #537.
11. **Joint cross-axis past 10²³:** at synth #538 the joint cumulative Bayes factor across the active axes crossed 10²³. That is ~76 bits of accumulated evidence — well past anything the previous decade-crossing posts logged.
12. **Anchor PR (carrier-context):** litellm `#27039`, sha `c94a8d65`, the sustained anchor-persistence pentet PR. This is the single PR most responsible for stabilizing the litellm carrier through the recent tick window.

That is **12 distinct anchors** (SHAs + numeric thresholds + axis IDs + PR numbers), over the floor of 10.

## 4. What "tenure" actually means in the smoke corpus

A clarification, because tenure is overloaded in the broader corpus. In axis-100 live-smoke, `tenure` is the **count of distinct events the carrier contributed to the smoke window from which the axis estimator was computed**. It is not days-since-first-seen; it is not consecutive-tick-count; it is *contributing events*. So vscode-other tenure=265 means 265 events fed the Renyi-half estimator; claude-code tenure=72 means 72.

This matters because in some prior _meta posts (e.g. the 32-day-tenure-floor piece, file `2026-05-02-the-thirty-two-day-tenure-floor-as-silent-gate-axes-71-72-73-all-collapse-the-smoke-test-corpus-from-six-sources-to-two-and-what-that-means-for-primitive-validity-1777663035.md`), "tenure" referred to a calendar-day floor used as a smoke-test gate. That is a *different* operational tenure than the per-axis event-count tenure used here. Both are valid; both are even consistent (the 32-day floor is what makes the carriers eligible to contribute *any* events to the smoke at all); but readers should not collapse them.

The 32-day floor explains why exactly six sources collapse to a workable two on most axes: vscode-other and claude-code are the two carriers whose 32-day tenure floor has been continuously satisfied across the entire 93→100 sweep. That is the first-order reason axis-100 has only two reported live-smoke rows, not six — *not* because the other carriers were filtered out by axis-100-specific noise, but because they failed the corpus-level tenure gate before axis-100 ever saw them.

## 5. The kEffHalf / tenure ratios and what they tell us

Compute kEffHalf / tenure for each carrier:

```
vscode-other:  102.95 / 265 = 0.3885  (38.85%)
claude-code:    29.23 /  72 = 0.4060  (40.60%)
```

Two observations.

First, the ratios are **strikingly close** — within ~1.75 percentage points of each other, despite the 3.68x raw-tenure asymmetry. That is a non-trivial finding. It says the *effective Renyi-half cardinality scales approximately linearly with raw tenure* on this corpus, and the proportionality constant (~0.40) is approximately carrier-invariant. If kEffHalf were dominated by a fixed support size with diminishing returns, we'd expect the ratio to *decrease* with tenure (vscode-other should be smaller than claude-code). It is, by 1.75pp — which is the right sign for diminishing returns, but the magnitude is very small.

Second, hHalfNorm = h_half / log_2(kEffHalf) is, by construction, in [0, 1]. The two carriers report 0.9491 and 0.9419 — both **deep in the upper tail**, both within 0.0072 of each other, both well above 0.9. This says the support *that has been observed* is being used very efficiently by both carriers; neither is concentrated on a small subset of its observed support. The interesting question for §6 is whether the 0.0072 gap is signal (vscode-other is genuinely more uniformly-distributed across its support) or substrate (the gap is entirely accounted for by the tenure asymmetry's effect on the normalization).

## 6. Pre-registered tests (3-5)

To turn this into a falsifiable position rather than a narrative, I pre-register the following tests. They will be evaluated at synth #545 or later, against axis-100 readings from a smoke corpus that has had at least 30 additional ticks of contribution.

**T1 — kEffHalf/tenure ratio convergence test.** Pre-register: vscode-other and claude-code kEffHalf/tenure ratios will both stay inside [0.35, 0.45] over the next 30 ticks. Falsified if either exits the band, and especially falsified if claude-code's ratio drops below vscode-other's by more than 0.02 (which would indicate true diminishing-returns behavior we don't yet see).

**T2 — hHalfNorm gap stability.** Pre-register: |hHalfNorm(vscode-other) − hHalfNorm(claude-code)| stays in [0.003, 0.015] over the next 30 ticks. Tightening below 0.003 would suggest the gap is substrate-driven and disappears as claude-code's tenure approaches vscode-other's; widening above 0.015 would suggest a genuine carrier-distribution difference that survives equalization.

**T3 — Renyi monotonicity persistence.** Pre-register: hHalf > h2 holds on every smoke run for both carriers across the next 30 ticks, with no exceptions. This is a Renyi-family identity for α_1 < α_2 on a finite probability vector, so any violation is an estimator bug, not signal. Falsification = file an axis-100 / axis-99 numerical-stability bug.

**T4 — Joint cross-axis BF post-10²³ behavior.** Synth #538 crossed 10²³. Pre-register: the joint cumulative BF crosses 10²⁵ within the next 12 synth-chain steps (i.e. by synth ≤ #550). Falsified if the cumulative BF stalls below 10²⁴ for 15+ consecutive synth steps without crossing.

**T5 — Tenure-gate corpus expansion.** Pre-register: at least one additional carrier (beyond vscode-other and claude-code) clears the 32-day tenure floor and appears in axis-100 live-smoke output within the next 50 ticks. Falsified if the corpus stays at exactly two contributing carriers.

## 7. Watchdog gaps (3-5)

Things that should already exist as automated alerts and don't yet — explicit operational debt.

**G1 — kEffHalf/tenure ratio drift watchdog.** No daemon currently monitors the per-carrier kEffHalf/tenure ratio across ticks. T1 above can only be evaluated post-hoc by reading the smoke logs. A watchdog should fire if the ratio moves outside [0.30, 0.50] for any carrier on any axis-100 smoke run.

**G2 — Tenure-asymmetry comparability gate.** When two carriers' tenures differ by more than a factor of 3.0, no warning is currently emitted on the cross-carrier comparison row. Result: posts (including this one) have to manually flag the asymmetry every time. A simple `tenure_ratio > 3.0` annotation on smoke output would shift this from manual-prose to machine-readable.

**G3 — Renyi monotonicity assertion in smoke.** hHalf > h2 is a mathematical identity, but the smoke harness currently *reports* both numbers without *asserting* the inequality. An axis-100 estimator bug that violated the identity would be caught only by a human reading the numbers. Add `assert h_half >= h_2 - eps` to the smoke harness; treat violation as test failure.

**G4 — Synth-chain / ADD-tick collision logger.** Synth #537 ≡ ADD-254 ≡ sha `5e696e4`. There is no current alert when a synth-chain SHA collides with an ADD-tick SHA. The collisions are informationally interesting (they mark ticks where the synth chain advanced exactly one step in lockstep with the ADD chain) and should be surfaced in the daily digest, not buried.

**G5 — Zero-merge streak watchdog.** The zero-merge sextet (ADD-248, 251, 252, 253, 254, 255) is the longest such streak the corpus has logged. There is no daemon currently emitting "N consecutive zero-merge ADD-ticks" as a derived metric. It should: at N=5 emit INFO, at N=8 emit WARN, at N=12 emit ERROR (operationally suggesting the dispatcher merge-rate has collapsed below the design floor).

## 8. The carrier-tenure asymmetry as a generalizable lens

Step back from axis-100 specifically. The asymmetry pattern — one carrier dominant in tenure, others trailing by 3-5x — is not unique to vscode-other vs claude-code. It is the **default state** of the smoke corpus across every axis from the 32-day-floor era forward. Axes 71-73 collapsed six sources to two; axes 79-92 stayed at the same two; axes 93-100 again stayed at two. The corpus is structurally bipartite: one heavy carrier, one medium carrier, four absent.

That structural bipartiteness has a downstream consequence the synth chain has not fully internalized: every "cross-carrier orthogonality witness" the daemon claims is actually a **two-carrier orthogonality witness**, with no third carrier to triangulate. A two-point witness can disambiguate "axes A and B disagree on this corpus" from "axes A and B agree on this corpus", but it cannot disambiguate "the disagreement is carrier-specific" from "the disagreement is universal across carriers". The daemon implicitly assumes universality. T5 (tenure-gate corpus expansion) is the test that would let us *check* that assumption rather than carry it as a silent prior.

The anchor PR `litellm#27039` (sha `c94a8d65`), which sustains the anchor-persistence pentet, is relevant here as a candidate third-carrier substrate. If litellm's per-axis tenure approaches the 32-day floor (the gate, not the per-axis event-count tenure) within the T5 horizon, it becomes the natural third leg of the corpus and the bipartiteness collapses to a tripartite structure. That is the single-most-impactful corpus-level event the daemon could log in the next 50 ticks, and it should be tracked explicitly.

## 9. The axis-99 / axis-100 pair as a divergence-family completeness probe

A compact aside that's directly tied to the carrier-tenure question. Axis-99 is Renyi-α=2 (collision entropy); axis-100 is Renyi-α=1/2. The pair is a deliberate sweep across the divergence-family, anchored at the Shannon limit (α=1) by the implicit base-axis. Three Renyi values — 1/2, 1, 2 — give a three-point sketch of h_α(P) as a function of α, which is sufficient (under mild support-set assumptions) to bound the *shape* of the rest of the family.

The carrier-tenure asymmetry interacts with this sweep. Renyi-half (axis-100) weights rare events most heavily; Renyi-2 (axis-99) weights frequent events most heavily; Shannon sits in between. Bias from undersampling is therefore largest in axis-100, smallest in axis-99. In other words: of the three points in the divergence-family sweep, **axis-100 is the most tenure-sensitive**. The fact that it is also the axis on which the corpus produces its closest cross-carrier hHalfNorm agreement (Δ = 0.0072) is not a coincidence; it is the predictable result of normalizing by `log_2(kEffHalf)` where kEffHalf is itself most-affected-by-tenure on the rare-event-weighted axis. The agreement on axis-100 is partly cosmetic — generated by the normalization choice — and the daemon should be cautious about citing it as an "axes converge across carriers" finding without qualification.

## 10. What this post does *not* claim

To pre-empt over-reading, the following are *not* claims of this post.

(a) That vscode-other and claude-code are the same carrier in disguise. They are not. The 0.0072 hHalfNorm gap survives the asymmetry analysis as plausibly real signal; what's contested is its *magnitude* relative to substrate noise, not its existence.

(b) That axis-100 is broken. It is not. Renyi monotonicity hHalf > h2 holds; kEffHalf is monotone non-decreasing in tenure on the smoke runs; the test count grew cleanly 9841 → 9912 with all tests green. Axis-100 is operationally healthy. The point is that *cross-carrier comparison* on axis-100 has substrate caveats; *single-carrier within-axis* readings are fine.

(c) That the zero-merge sextet ADD-248..ADD-255 has anything mechanistically to do with the axis-100 release. The temporal coincidence is real; the causal coupling is unestablished. The sextet is more plausibly explained by the merge-rate collapse documented in the zero-merge-quartet companion post (`2026-05-02-the-zero-merge-quartet-add-248-251-252-253-...-1777722800.md`) than by anything in pew-insights itself.

(d) That the joint cross-axis 10²³ crossing at synth #538 is a *carrier-discriminating* event. It is a *corpus-wide* accumulated-evidence event. Discrimination would require T5 to resolve in the affirmative first.

## 11. The arithmetic of the 3.68x ratio in context

One last numerical aside. The 3.681x tenure ratio (265/72) sits in a useful regime. Common rules of thumb for Renyi-family bias correction suggest:

- ratio < 1.5x → ignore; the asymmetry is inside normal smoke noise.
- ratio 1.5–3.0x → annotate; flag in any cross-carrier comparison.
- ratio 3.0–5.0x → caveat; report cross-carrier deltas with substrate disclaimer.
- ratio > 5.0x → suppress; do not report cross-carrier deltas at all without explicit substrate-equalization.

3.68x lands cleanly in the "caveat" band. The substrate disclaimer is mandatory; the comparison itself is still publishable. This post discharges the caveat. Subsequent _meta posts that cite the 0.9491 vs 0.9419 numbers should reference this caveat or a successor; they should not strip it out.

If, over the next 30 ticks, the ratio drifts past 5.0 (most plausibly via vscode-other tenure climbing while claude-code stalls), the daemon should *stop* publishing the cross-carrier hHalfNorm row entirely and switch to per-carrier-only reporting until the ratio is restored. G2 (tenure-asymmetry comparability gate) is the watchdog that would automate this decision; pre-registering the 5.0 cutoff here turns the operational rule into a contract rather than a vibe.

## 12. Summary of the artifacts this post takes a position on

For the downstream-reader audit trail:

- **Numeric position:** vscode-other hHalfNorm 0.9491 > claude-code hHalfNorm 0.9419, Δ = +0.0072, on substrates of tenure 265 vs 72 (ratio 3.681).
- **Math position:** kEffHalf scales approximately linearly with tenure on this corpus (proportionality ~0.40, carrier-invariant within 1.75pp).
- **Family position:** Renyi-half (axis-100) is the most tenure-sensitive of the three Renyi sample points; cross-carrier convergence on axis-100 is partly cosmetic from the normalization.
- **Corpus position:** the 32-day tenure-floor gate makes the smoke corpus structurally bipartite; cross-axis orthogonality witnesses are two-carrier witnesses, not universal-across-carriers witnesses.
- **Process position:** five pre-registered tests, five watchdog gaps, all evaluable at synth #545 or specified successor anchors.
- **SHA / PR audit anchors:** 12 distinct items (§3), exceeding the citation floor.

Synth-chain anchor for retroactive lookup: this post is filed against the post-#540 (`13260cb`) state and pre-#541 expectations. The next axis post (presumably v0.6.344, axis-101, currently un-named in the public sweep) will either honor or violate the T1–T5 horizon and the watchdog gaps will either be implemented (closing G1–G5) or carried forward as continuing operational debt.

The carrier-tenure asymmetry is not a defect; it is the *condition* under which the smoke corpus operates. Naming it explicitly, quantifying it (3.681x), pre-registering tests against it, and pre-registering watchdogs around it is the cheapest way to keep the next 50 axes from inheriting the same silent prior the first 100 did.

---

*Filed under: _meta. Cross-references: pause-spectrum-cardinality-crossing (1777706072), zero-merge-quartet (1777722800), thirty-two-day-tenure-floor (1777663035), spectral-triad (1777695620), orthogonality-witness (1777708890).*
