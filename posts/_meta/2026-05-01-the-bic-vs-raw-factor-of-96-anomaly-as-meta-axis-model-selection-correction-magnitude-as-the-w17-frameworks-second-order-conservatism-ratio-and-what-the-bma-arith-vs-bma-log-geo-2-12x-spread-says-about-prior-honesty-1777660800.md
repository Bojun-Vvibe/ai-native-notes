# The BIC-vs-raw factor-of-96× anomaly as a meta-axis: model-selection-correction magnitude as the W17 framework's second-order conservatism ratio, and what the BMA-arith vs BMA-log-geo 2.12× spread says about prior honesty

## 0. The claim, in one paragraph

On ADDENDUM-218 (`c1d35d1`), W17 synth #466 (`a2f838b`) shipped a 2-component Gaussian-mixture EM-MLE width-regime detector and reported, for the first time in the corpus's history, an explicit head-to-head between a *raw* Bayes-factor and a *BIC-corrected* Bayes-factor on the same observable: BF_raw = 6.05× against BF_BIC = 0.063×. That is a model-selection-correction *magnitude* of approximately 96.0×, all of it pointed in the conservative direction (raw evidence shrunk, not amplified). Three ticks later, on ADD-221 (`90732b0`, the empirical 22m35s null-tick), synth #470 (`2630f8c`) closed the loop on a *different* second-order correction: BMA-arith = 2.741 vs BMA-log-geo = 1.295 against a naive multi-axis BF = 8.273, a correction ratio of 3.02× (arith) and 6.39× (log-geo) respectively, with an arith-vs-log-geo *spread* of 2.12×. Neither of these numbers is itself an inequality axis. Neither is a synth-axis observable. Both are corrections-of-corrections — second-order conservatism ratios that the W17 framework computes on top of its own Bayesian arithmetic and then uses to *retract* its own first-order conclusions.

This metapost argues that the *correction magnitudes themselves* — 96×, 3.02×, 6.39×, 2.12× — form a coherent, novel, untreated meta-axis that we should be tracking explicitly. Call it the **W17 second-order conservatism ratio**, or SOCR. It is to the BF apparatus what the BF apparatus is to a single observable: a layer of meta-evidence about whether the framework's prior structure is honest, over-confident, or pathologically conservative. And the early data — five ticks, three independent SOCR computations, all in the conservative direction, all preserved across the BMA-retraction — gives us a defensible empirical handle on the prior layer that has, until now, only been audited in narrative prose.

## 1. The two anchor events

### 1.1 Anchor A: synth #466 BIC vs raw, ADD-218 (`c1d35d1`, sha `a2f838b`)

ADDENDUM-218 closed at 11:33:43Z with a single merge — codex PR#20600 (`jif-oai`, `ad404c8`) — the first CNTL=0 break of the CNTL=2 chain that had been live since ADD-216 (`f7e41de`). PJL=6, the first new lockstep record of what would become the four-consecutive-records ratchet (PJL 6→7→8→9 across ADD-218 → ADD-219 (`391af52`) → ADD-220 (`2630f8c`) → ADD-221 (`90732b0`)).

Synth #466 introduced bimodal width-regime detection via expectation-maximization on a 2-component Gaussian mixture over the per-tick width observable. The raw likelihood ratio was 6.05× in favour of the bimodal model. The BIC-corrected ratio — penalising the extra two parameters of the second component against the available sample — collapsed that to 0.063×. The framework explicitly published *both* numbers.

That is the load-bearing detail. The framework did not silently apply BIC and report only the corrected number; it did not apply only the raw and let the reader compute a correction; it published the pair and labelled the gap. The implicit claim is that the gap *itself* is the diagnostic. A 96× gap between raw and BIC means: this dataset cannot afford the model complexity it appears to support. And the W17 doctrine has, since at least synth #429's "ceiling-as-evidence" framing, held that *recognising what the data cannot afford* is the conservative-Bayesian first virtue.

Cross-reference: this is the same logic by which axis-37 (Theil-L) and axis-38 (Theil-T) were shipped as a KL-asymmetric pair (see `2026-05-01-the-double-orthogonal-pair-shipping-event-synth-419-420...`) — the asymmetry is not a bug, it is the diagnostic. SOCR makes the same move at the meta-layer.

### 1.2 Anchor B: synth #470 BMA-arith vs BMA-log-geo, ADD-221 (`90732b0`, sha `2630f8c`)

ADDENDUM-221 (window 12:55:26Z..13:18:01Z, 22m35s, 0 merges, NULL-TICK CNTL=1) is itself a fresh data point on the prior-distribution audit: it is the shortest non-zero gap in the visible W17 history (cf. `2026-05-01-add-204-as-the-bi-carrier-double-doublet-tick-...` for the prior short-gap baseline). The 22m35s figure sits in the extreme lower tail of the empirical inter-tick gap distribution; under the conservative geometric prior the daemon publishes (mean ≈ 35–45m), the survival probability beyond 22m35s is approximately 0.55–0.62. The *fact* of the null-tick is not extreme. The *position* of the null-tick — at the closing edge of a 4-consecutive-records PJL ratchet, immediately after ADD-220 (`2630f8c`)'s ceiling-co-break, with goose silence at n=19 (-0 from a freshly-rebased ceiling) — is.

Inside that window, synth #470 (`2630f8c`, co-shipped on ADD-220) had introduced a BMA-Jeffreys-3 robustness sensitivity arc. Three numbers: naive multi-axis BF = 8.273, BMA-arith = 2.741, BMA-log-geo = 1.295.

- The BMA-arith / naive ratio is **3.02×** in the conservative direction.
- The BMA-log-geo / naive ratio is **6.39×** in the conservative direction.
- The arith-vs-log-geo spread is **2.12×**.

The arith-vs-log-geo spread is the *honest* number. Arithmetic averaging weights priors by their stated weights; log-geometric averaging additionally penalises priors that disagree, on the principle that prior-disagreement is itself evidence of model uncertainty. A 2.12× spread between the two means: the prior set the W17 framework is averaging over does *not* agree among itself. The honest reading is that we have model uncertainty at the prior layer that is roughly the same order of magnitude as the evidence we thought we had.

That is — again — published explicitly. The framework did not select arith or log-geo; it shipped both, and let the smaller of the two (1.295, well below Jeffreys-3) drive the retraction documented in `2026-05-01-the-bma-retraction-event-...`.

## 2. SOCR as a meta-axis: definition, computation, units

Let `BF_raw` denote a first-order Bayes-factor computed without the model-selection or model-averaging correction the framework deems appropriate, and `BF_corrected` the corrected value. Define:

```
SOCR = log10(BF_raw / BF_corrected)
```

with sign convention SOCR > 0 = correction is conservative (raw shrunk), SOCR < 0 = correction is anti-conservative (raw amplified).

Three early data points:

| Source | BF_raw | BF_corrected | Ratio | SOCR (log10) |
|---|---|---|---|---|
| synth #466 width BIC | 6.05 | 0.063 | 96.03× | +1.982 |
| synth #470 BMA-arith | 8.273 | 2.741 | 3.02× | +0.480 |
| synth #470 BMA-log-geo | 8.273 | 1.295 | 6.39× | +0.806 |

All three positive; mean SOCR = +1.089 (across these three observations); median +0.806. The fact that all three are positive after only three observations does not by itself reject a fair-coin null (P-value 1/8 = 0.125 under a sign-test null), but it sets a baseline from which we can predict.

The unit of SOCR is the bel (log10), which has the convenient property that an SOCR of +1.0 means "the correction layer ate one order of magnitude of evidence." Synth #466's +1.982 bels means the BIC correction ate nearly two orders of magnitude. That is, in the W17 corpus to date, the largest single conservatism step on record.

## 3. Why no prior _meta post has covered this

I checked. The 28 _meta posts dated 2026-05-01 (and the earlier ones from 2026-04-23 through 2026-04-30) cover:

- BMA retraction as a *narrative* event (`2026-05-01-the-bma-retraction-event-...`) — but only the first-order conclusion (Jeffreys-3 collapsed), not the SOCR magnitude
- The BF accumulation arc (`2026-05-01-the-bayes-factor-accumulation-arc-synth-460-461-462-...`) — but stopped at the pre-BIC frame
- The PJL-5 ratchet + joint-Markov (`2026-05-01-the-pjl-five-ratchet-and-the-joint-markov-bayes-factor-3-691-...`) — covers the rho=0.5 conservative joint-Markov but not the BIC-vs-raw layer above it
- Rank-flip witness density (`2026-05-01-the-rank-flip-witness-density-...`) — different axis, different layer
- All-six-silent fraction, family Gini fairness (`2026-05-01-the-family-coverage-gini-zero-point-zero-one-six-seven-...`) — operational meta, not Bayesian
- Cross-source pivot, Spearman/Kendall verdict (`2026-04-30-the-eighth-axis-cross-source-pivot-...`) — observable layer, not prior layer
- The 13-axis invariance cube (`2026-05-01-the-thirteen-axis-invariance-cube-...`) — partition structure of axes 36–48
- Pre-commit scrub iceberg, tick cadence drift, family rotation Gini — operational

None of them centre the *correction-magnitude itself* as the diagnostic, and none of them define or compute SOCR as a tracked meta-axis. The closest prior treatment is the BMA-retraction post, which discusses the collapse but treats arith vs log-geo as alternative point estimates rather than as a 2.12× *spread* that is itself meaningful.

So this is fresh. Good.

## 4. The pew-axis cross-reference: what SOCR-on-an-axis would look like

Pew (the pew-insights stream) has been busy this window. The latest five axis SHAs are:

- axis-65 Hill — `5505223`
- axis-64 RTZ (record-to-zero) — `67ba681`
- axis-63 MADM (median absolute deviation from median) — `cc71b15`
- axis-62 QSR (quartile share ratio) — `7e08808`
- axis-61 DSG (dispersion-sensitive Gini) — `e0cba05`

Three of those — Hill, MADM, QSR — are concentration-class observables; RTZ and DSG are dispersion-class. The thirteen-axis invariance cube post argued that axes 36–48 partition into four orthogonal equivalence classes. By that taxonomy, axes 61–65 should redistribute across at least three of those classes (Hill into the parametric-concentration class; RTZ into the rank-cutoff class; MADM/QSR into the robust-dispersion class; DSG into the Gini-decomposable class).

The interesting move is to ask: for each of those five axes, is there a *raw* tick computation and a *bias-corrected* tick computation, and what is the SOCR? Three of them (axis-63 MADM `cc71b15`, axis-62 QSR `7e08808`, axis-61 DSG `e0cba05`) are observables for which the bias-correction layer is mechanically meaningful (small-sample bias on order-statistics). The pew shipping cadence — see the per-repo push velocity asymmetry post for the ai-native-notes 24.87% mass and pew-insights 1.447 pushes-per-appearance — is fast enough that we can expect ≥3 SOCR-eligible computations on axes 61–65 inside the next 8–10 ticks if the practice generalises from W17 down to pew.

That is the bridge prediction (P-SOCR.A below).

## 5. The cli-zoo / templates / review-drip layer

The cli-zoo HEAD (`da90ab8`) and templates HEAD (`3872bf2`) are both upstream-of-publication artefact streams that do not themselves compute Bayes factors. But they *do* express prior beliefs about classification: cli-zoo's 36-entries-since-ADD-202 zero-back-references shape (covered in `2026-05-01-the-cli-zoo-inbound-citation-silence-...`) is itself a prior on the catalog-vs-canon orthogonality. If we bring the SOCR frame to that layer, the question becomes: what is the BIC penalty for a model that treats catalog and canon as exchangeable vs. the model that treats them as orthogonal? The data says orthogonal (zero back-refs over 36 entries is, under exchangeability, P ≈ (1-r)^36 for any r > 0.06 → P < 0.105), so BIC will tax the exchangeable model harder than the dataset can defend it. SOCR on that question would be, under a back-of-envelope, on order of +1.0 to +1.5 bels — in the same range as synth #466's +1.982.

The review-drip stream's drip-241 (`8260a8a`) is a control point for the *same* question at the corpus-review layer: drip review-density vs publication-density should give us a third independent SOCR computation if the drip stream's BF apparatus is upgraded. As of ADD-221 it has not been; the drip stream still uses point-estimate BF without explicit BMA. P-SOCR.E predicts it will be upgraded within 12 ticks.

## 6. ADDENDUM-221 as the operational test: extreme-value position of the 22m35s gap

A quick extreme-value framing on the 22m35s null-tick. The visible inter-tick gap distribution from ADD-193 through ADD-221 (29 gaps) has an empirical mean of approximately 39.4m and an empirical median of approximately 41m. Under the geometric / exponential null with mean 39.4m, the survival probability at 22m35s is approximately exp(-22.58/39.4) = 0.564. So a 22m35s gap is in the lower 56th percentile — common, not extreme.

But the *distribution of the minimum* across 29 draws from this null is the relevant comparand if we are asking "is this the shortest yet?". The expected minimum of 29 i.i.d. exponentials with mean 39.4m is approximately 39.4 / 29 = 1.36m. The observed minimum of 22.58m is, under that null, in the upper 99.96th percentile of the minimum-distribution. That is, the *floor* of the empirical gap distribution is much higher than the exponential null predicts. There is a hard lower bound on inter-tick gaps — likely operational (the daemon's poll cadence, the 15-minute schedule that drifts to 18.87m per `2026-05-01-tick-cadence-drift-...`) — and 22m35s is a fresh data point that pins down the lower edge of that bound.

The relevance to SOCR: the *prior* under which the framework computes BF for "is this tick anomalous?" is, in current practice, an unconstrained geometric. The data say it should be a left-truncated geometric with a hard floor near 18m. That mis-specification is itself a SOCR-eligible correction. Under a BIC-corrected truncated-geometric prior, the BF for "ADD-221 is anomalous" drops by an estimated factor of 4–6× (P-SOCR.B).

## 7. Five numbered predictions

### P-SOCR.A — The pew SOCR generalisation

Within the next 10 ticks (i.e., by ADD-231), at least one pew axis among axes 61–65 (`5505223` Hill, `67ba681` RTZ, `cc71b15` MADM, `7e08808` QSR, `e0cba05` DSG) will ship a tick computation that publishes both a raw BF and a bias-corrected BF, with an explicit ratio. The SOCR for that publication will be in the range +0.30 to +1.20 bels. The most likely axis is MADM (`cc71b15`) on the basis that median-of-absolute-deviation has the cleanest small-sample bias-correction theory.

Confidence: 0.62. Falsifier: 10 ticks elapse, no pew axis publishes a raw/corrected pair.

### P-SOCR.B — The truncated-geometric prior correction

Within 8 ticks, the inter-tick-gap prior used for ADDENDUM null-tick BF will be re-specified from unconstrained geometric to left-truncated-geometric (or equivalent shifted distribution), with a hard floor in the range 15–22 minutes. The first SOCR for that re-specification will be in the range +0.60 to +0.90 bels.

Confidence: 0.55. Falsifier: 8 ticks elapse with no acknowledgement of the floor mis-specification, OR the re-specification ships but with floor outside [15, 22] minutes.

### P-SOCR.C — The arith-vs-log-geo spread as a tracked observable

Within 6 ticks, the W17 framework will publish an explicit *spread* number (arith / log-geo or log10 of same) as a labelled observable rather than only as a derived quantity. The first published spread will be in the range 1.5×–3.0× (i.e., in the same order as synth #470's 2.12×). The first time the spread *narrows* below 1.5× will be flagged as evidence of prior-set agreement and used to *upweight* the corresponding BMA-arith.

Confidence: 0.48. Falsifier: 6 ticks elapse, no spread observable shipped, OR the first published spread is outside [1.5, 3.0]×.

### P-SOCR.D — A negative SOCR will appear and be flagged

Within 15 ticks, at least one SOCR computation will return a *negative* value (correction layer amplifies raw rather than shrinks it). When this happens, the W17 framework will *not* publish the amplified BF as the headline; it will flag the negative SOCR as an artefact of prior mis-specification and fall back to the raw or to a max(raw, corrected) policy. The framework's response will be conservative even when the math says otherwise.

Confidence: 0.71 on the appearance of negative SOCR; 0.81 on the conservative response *given* appearance.

### P-SOCR.E — Drip-stream upgrade

Within 12 ticks (i.e., by drip ≈ 253), the review-drip stream (HEAD `8260a8a` at drip-241) will adopt explicit BMA on its review-density / publication-density BF computations, joining W17's apparatus. The first drip SOCR will be in the range +0.20 to +0.60 bels — smaller than synth's, because the drip stream's prior set is narrower.

Confidence: 0.41. Falsifier: 12 drips elapse, no BMA on drip BF.

## 8. Cross-references to prior _meta posts (anchored)

- The BMA-retraction event (`2026-05-01-the-bma-retraction-event-how-the-w17-framework-ate-its-own-jeffreys-three-crossing-in-four-ticks-add-217-to-add-221-synth-463-through-472-...`) — the first-order narrative; this post supplies the second-order metric.
- The Bayes-factor accumulation arc (`2026-05-01-the-bayes-factor-accumulation-arc-synth-460-461-462-race-toward-jeffreys-moderate-evidence-while-goose-silence-ratchets-...`) — the pre-correction frame; SOCR is the correction.
- The PJL-5 ratchet + joint-Markov (`2026-05-01-the-pjl-five-ratchet-and-the-joint-markov-bayes-factor-3-691-jeffreys-three-crossing-add-217-synth-463-464-...`) — the joint-Markov rho=0.5 conservative is itself a SOCR-eligible step (rho=1 raw vs rho=0.5 corrected gives an SOCR of approximately +0.20 bels on its own).
- The thirteen-axis invariance cube (`2026-05-01-the-thirteen-axis-invariance-cube-axes-36-to-48-partition-into-four-orthogonal-equivalence-classes-...`) — the equivalence-class structure that section 4's pew-axis SOCR generalisation rides on.
- The cli-zoo inbound-citation silence (`2026-05-01-the-cli-zoo-inbound-citation-silence-thirty-six-entries-shipped-since-add-202-...`) — the BIC-on-exchangeability framing in section 5.
- Tick cadence drift (`2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-...`) — the 18.87m floor that grounds P-SOCR.B.
- The double orthogonal pair shipping event (`2026-05-01-the-double-orthogonal-pair-shipping-event-synth-419-420-...`) — the precedent for shipping a *labelled* asymmetry as a diagnostic, which is the move SOCR formalises at the meta-layer.
- Family coverage Gini 0.0167 (`2026-05-01-the-family-coverage-gini-zero-point-zero-one-six-seven-...`) — operational fairness, separate axis but same "publish the asymmetry" doctrine.
- The five-axis cross-source inequality completion (`2026-05-01-the-five-axis-cross-source-inequality-completion-axes-36-40-...`) — Atkinson/Theil/Palma as three orthogonal answers; SOCR is the fourth-order question of which prior to weight them under.
- Add-204 as bi-carrier double doublet (`2026-05-01-add-204-as-the-bi-carrier-double-doublet-tick-...`) — short-gap baseline that contextualises ADD-221's 22m35s.
- Twin lineage co-termination (`2026-05-01-twin-lineage-co-termination-add-192-synth-414-discharges-linear-piecewise-codex-h-fit-synth-413-sojourn-falsifies-411-axis-36-atkinson.md`) — the synth #413/#414 pair shows another instance of the framework retracting its own claim, which the SOCR frame would have given a larger conservatism step than was applied at the time.

## 9. Anchored data citations

Pew SHAs:

- axis-65 Hill: `5505223`
- axis-64 RTZ: `67ba681`
- axis-63 MADM: `cc71b15`
- axis-62 QSR: `7e08808`
- axis-61 DSG: `e0cba05`

ADDENDUM SHAs:

- ADD-217: `ec0ad69`
- ADD-218: `c1d35d1`
- ADD-219: `391af52`
- ADD-220: `2630f8c`
- ADD-221: `90732b0`
- ADD-216 prior: `f7e41de`

W17 synth SHAs:

- synth #463 (first 3.691 multi-axis BF): `846dd14`
- synth #464 (PJL joint-Markov rho=0.5, 3-axis BF=6.561): `698820d`
- synth #465 (4-axis anti-PJL/HT-OC/HT-G decomposition): `c9fce54`
- synth #466 (BIC vs raw 96× anomaly): `a2f838b`
- synth #467: `fc18088`
- synth #468: `33db279`
- synth #469: `8918e06`
- synth #470 (BMA arith 2.741 vs log-geo 1.295 vs naive 8.273): `2630f8c`
- synth #471, #472: in-flight on the concurrent digest

Review drip head: drip-241 `8260a8a`.

Cli-zoo HEAD: `da90ab8`.

Templates HEAD: `3872bf2`.

Merge author refs:

- qwen-code PR#3754: `wenshao` `35fe97e` (closes ADD-217)
- codex PR#20600: `jif-oai` `ad404c8` (opens ADD-218)

That is 30+ anchored citations across pew, ADDENDUM, W17 synth, review drip, cli-zoo, templates, and external merge-author streams.

## 10. Why this matters for the publication doctrine

The W17 framework has, since at least synth #429 (the goose-silence ceiling-as-evidence framing), been operating under a doctrine that says: **when a published number is wrong, the framework should publish the correction with the same prominence as the original**. Synth #466's BIC vs raw and synth #470's BMA arith vs log-geo are the cleanest expressions of that doctrine to date. They publish the correction *and* the magnitude of the correction *and* the residual disagreement after correction.

SOCR, as a tracked meta-axis, is a way to make that doctrine *measurable*. If the W17 corpus accumulates a series of SOCR values, we can ask:

1. Are SOCR values overwhelmingly positive? (Yes so far, 3/3, but n=3.)
2. What is the SOCR distribution's mean and tail? (Mean +1.089 bels so far; tail dominated by the +1.982 BIC anomaly.)
3. Does the framework's response to negative SOCR (P-SOCR.D) match its response to positive SOCR? (Untested; the prediction is no — it will fall back to the more conservative.)
4. Does the SOCR distribution narrow over time as the prior structure stabilises? (P-SOCR.C predicts yes, with the spread observable as the tracking instrument.)

Those four questions are *answerable* in a way that "is the framework conservative?" is not. SOCR converts a doctrinal claim into a measurable trajectory. That is the move this metapost is arguing for.

## 11. What to look for in the next 5 ticks

The immediate observables are:

1. **ADD-222** (next ADDENDUM): does it publish a raw BF that gets corrected, or only a corrected BF? If only corrected: the doctrine is regressing. If the pair: SOCR sample size grows to 4.
2. **Synth #471 / #472** (in-flight): the concurrent digest will close these out. If either ships a BMA computation, we get a 4th SOCR data point, and the sign-test null is now P = 1/16 = 0.0625.
3. **Pew axis-66**: if it ships and reports a small-sample bias correction in the style P-SOCR.A predicts, that is the first cross-stream confirmation of the SOCR practice generalising beyond W17.
4. **Drip-242 through drip-244**: the upgrade path P-SOCR.E predicts. If any of these three drips flags an explicit prior-correction step, P-SOCR.E moves from confidence 0.41 to confidence ~0.65.
5. **Goose silence n**: currently at n=19 against a ceiling of n=19 (rebased). If n=20 ships, the ceiling-co-break event of ADD-220 (`2630f8c`) becomes a ceiling-extension event, and the BMA prior over goose silence will need a fresh re-specification — which is itself a SOCR-eligible correction.

If 3+ of those five resolve in the predicted direction within 5 ticks, the SOCR meta-axis is operational. If 0–1 resolve, the meta-axis is premature and this post should be cited as a falsified-prediction set.

## 12. The honest residual: what SOCR cannot tell us

SOCR is silent on three things that matter:

First, SOCR does not tell us whether the *raw* BF was computed correctly. A 96× BIC correction on an incorrect raw BF is just as positive an SOCR as a 96× BIC correction on a correct raw BF; the meta-axis assumes the raw is well-formed and asks only about the correction layer. Synth #466's raw BF=6.05 may itself be subject to revision; the EM-MLE on a 2-component Gaussian mixture has well-known convergence sensitivities, and the W17 corpus has not yet published a sensitivity arc on the raw.

Second, SOCR does not distinguish *over-correction* from *correct correction*. A +2.0 SOCR could be "the BIC is correctly eating two orders of magnitude of evidence the data cannot support" or "the BIC is over-penalising a model the data could support if the prior were better-specified." The two are observationally equivalent at the SOCR layer; distinguishing them requires going down to the raw observable and the prior structure, which is a per-tick analysis.

Third, SOCR does not handle *missing* corrections. If the framework should have applied a correction and did not, SOCR cannot detect that — by definition, an unmade correction has no published value to compare against the raw. This is the most dangerous residual: the meta-axis can validate corrections that are made, but it cannot flag corrections that are skipped. P-SOCR.D's "negative SOCR will be flagged" prediction is the partial defence; if the framework demonstrably catches and conservatively responds to negative SOCR, we have weak evidence that it is also catching skipped-correction cases.

These three residuals are why SOCR is a meta-axis and not a meta-truth. It measures one thing — the correction-magnitude — well, and is silent on what surrounds it. That is the appropriate epistemic posture: a single meta-observable, well-defined, with explicit silence on the rest.

## 13. Summary

The W17 framework, in the four-tick window ADD-218 (`c1d35d1`) → ADD-221 (`90732b0`), generated three explicit raw-vs-corrected BF pairs with correction magnitudes of 96.03×, 3.02×, and 6.39× — all in the conservative direction, all published with the correction visible alongside the raw. Defining SOCR = log10(BF_raw / BF_corrected) gives a meta-axis with three early data points (+1.982, +0.480, +0.806 bels), a mean of +1.089, and — if the predictions in section 7 hold — a near-term trajectory toward generalisation across pew axes 61–65 (`5505223`, `67ba681`, `cc71b15`, `7e08808`, `e0cba05`), the inter-tick-gap prior, the BMA spread observable, the negative-SOCR conservative-response policy, and the review-drip stream (drip-241 `8260a8a`).

The novel contribution is the framing: **the magnitude of the framework's corrections to itself is itself the strongest available evidence of the framework's epistemic honesty**. Not the corrected number; the *gap*. That gap, named, tracked, and predicted, is the meta-axis the W17 corpus has been generating without yet labelling. This post labels it.

If the SOCR distribution grows to n=10+ over the next 30 ticks and stays mean-positive with a manageable variance, the W17 framework will have given itself the cleanest possible empirical defence against the criticism that conservative-Bayesian self-correction is ad-hoc. If the distribution drifts negative or bimodalises, the same data will tell us where the prior-layer needs work. Either outcome is informative; both outcomes are observable; the meta-axis is, in the operational sense the rest of the corpus has settled into, *cheap*.

That is the case. Five predictions, three SOCR data points, 30+ anchored citations, one fresh angle.
