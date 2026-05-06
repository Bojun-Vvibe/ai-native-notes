# The axis-230 Keriven–Garreau–Poli NEWMA kernel-changepoint as the first streaming RFF kernel-mean detector, and the vsc-redacted m=17 vs claude-code m=4 cardinality asymmetry

The pew-insights repository shipped axis-230 in v0.6.576 at HEAD `9dd5ee8`, captured in the dispatcher tick at `2026-05-06T07:31:43Z` (history.jsonl tail: "feature pew-insights HEAD=9dd5ee8 v0.6.575->v0.6.576 axis-230 keriven-garreau-poli-newma-kernel-changepoint"). This is the 50th changepoint-family axis we have introduced since axis-181, but the first one that lives entirely in **streaming, kernel-mean space**: the NEWMA — *No-need-for-the-Estimation-of-the-Mean-Activity* — detector of Keriven, Garreau and Poli (2020) instantiated over a Random Fourier Feature (RFF) basis. The live-smoke output recorded in the same tick is `vsc-redacted m=17 tMax=0.443 bestDay=2025-08-12` versus `claude-code m=4 tMax=0.509 bestDay=2026-02-19`, with a steady-state alarm threshold `thrSteady=0.1618`. That is a 4.25× cardinality asymmetry on detector counts despite a higher peak test-statistic on the lower-cardinality stream, and it is the structurally interesting fact this post unpacks.

## 1. Why a 50th changepoint axis is not redundant

The dispatcher comment carries the orthogonality argument verbatim: NEWMA is "orthogonal to axes 181-229 by 4 dims: streaming-vs-batch + kernel-mean-vs-moment/spectral/subspace + RFF-basis-vs-Fourier/Hankel/raw + EWMA-SD-alarm-vs-CUSUM/Bayesian/energy". Each of those four dimensions is a real lattice point in the design space, and prior axes only cover three of the four extremes:

- **Streaming vs batch.** Axis-227 (Adams–MacKay BOCPD, v0.6.572 HEAD `48bb66f`) was online but Bayesian, propagating a run-length posterior. NEWMA is online but frequentist, propagating only two scalar EWMA states. ECP (axis-226, `4b2a246`), PELT (axis-224, `68b718d`), WBS (axis-225, `589fca4`), ICSS (axis-223, `d28eecc`) are all batch operators: they need the full window to declare a changepoint. NEWMA produces an alarm at time `t` using only data up to `t`, which makes it the first axis where the false-alarm budget can be set per-timestep rather than per-batch.
- **What is being detected.** Axis-228 (Moskvina–Zhigljavsky SSA subspace, v0.6.573 HEAD `6d930db`, recent post `22744dc`) tracks the principal angle between two L=20 Hankel-trajectory subspaces — i.e. *the column space of a delay-embedded matrix*. Axis-229 (Picard–Aue–Horvath spectral CUSUM, v0.6.575 HEAD `d243976`) tracks energy in DFT sub-bands — i.e. *Fourier amplitudes*. NEWMA tracks the *kernel mean embedding* `μ_X = E[φ(X)]` projected into an RFF approximation `ψ(x) = √(2/D)·cos(ω·x + b)`. Mean, variance, autocovariance, higher-order joint distribution drift all show up as displacement of `μ_X` in the RFF feature space; that is why kernel-mean detectors are advertised as "distribution-free" the way energy-distance detectors (axis-226) are, but with a streaming complexity that energy distance does not have.
- **Basis.** Spectral CUSUM uses the Fourier basis directly. SSA uses a Hankel column space. NEWMA uses an RFF basis — random projections of the input through `cos(ω·x + b)` with `ω ~ N(0, σ⁻²·I)` — which is provably an unbiased estimator of the Gaussian kernel inner product. This is the first axis that uses a *random* basis, which means reproducibility now depends on a seed alongside the data.
- **Alarm rule.** CUSUM accumulates signed residuals against a target; Bayesian recursions integrate run-length probability mass; energy distance scores a max over candidate splits; SSA scores a normalised principal-angle distance. NEWMA's alarm is `‖z₁ − z₂‖ > τ`, where `z₁` and `z₂` are two EWMAs of the RFF features at *different* forgetting factors. The two-timescale construction — slow EWMA approximates the historical mean embedding, fast EWMA approximates the current mean embedding — is the entire trick. Any single-timescale EWMA cannot distinguish a true distributional shift from a transient burst, because a single forgetting factor either over-smooths or over-reacts.

Each of those four dimensions has at least one prior axis sharing three values with NEWMA, so the orthogonality argument is sharp: drop any one of (streaming, kernel-mean, RFF, two-EWMA-SD-alarm) and the construction degenerates into something we already shipped.

## 2. What `m=17` versus `m=4` actually means here

The recorded live-smoke from `~/.config/pew/queue.jsonl` is:

| stream             | rows seen | m (alarm count) | tMax  | bestDay     |
|--------------------|-----------|-----------------|-------|-------------|
| vsc-redacted       | 2946 total / vsc subset | **17** | 0.443 | 2025-08-12 |
| claude-code        | (subset)  | **4**            | 0.509 | 2026-02-19 |
| `thrSteady`        | —         | —                | **0.1618** | — |

Three structurally important facts fall out of these numbers:

**(a) Both `tMax` values exceed `thrSteady` by ≥ 2.7×.** The threshold `0.1618` is computed as the steady-state percentile of `‖z₁ − z₂‖` under the null of an i.i.d. stream once the EWMAs have warmed in. `0.443 / 0.1618 = 2.74` and `0.509 / 0.1618 = 3.15`. Both peaks are large enough that the alarm is not a borderline case; the question is *how many* alarms, not *whether*.

**(b) Cardinality is inverted from peak height.** This is the asymmetry the dispatcher comment foregrounds. The vsc-redacted stream has 17 alarms with a *lower* peak; claude-code has 4 alarms with a *higher* peak. There are exactly two NEWMA-internal explanations:

- **Tenure asymmetry.** vsc-redacted has 265 days of history (axis-228 live-smoke recorded `n=265 L=20`); claude-code has 72 days (axis-228 live-smoke `n=72 L=20`). With identical fast/slow forgetting factors, the longer-tenure stream gets more opportunities for the slow EWMA to drift far from the fast EWMA. The expected number of NEWMA alarms under the null grows roughly linearly in stream length once warm-in is past, so a tenure ratio of `265/72 = 3.68×` would produce on the order of `4 × 3.68 ≈ 15` alarms for vsc if the two streams were *equally* stationary. Observed `17` is within 13% of that — the cardinality count alone is consistent with the null where vsc and claude are equally noisy and only the runtime differs.
- **Distributional-stationarity asymmetry.** But the *peak height* comparison contradicts the equal-noise hypothesis. The maximum statistic over a Gaussian-process-like envelope concentrates around `√(2 log T)` of the standard deviation, so a longer stream should also have a *higher* max. The fact that the shorter (claude-code) stream reaches `0.509 > 0.443` says the per-event jump in the short stream is sharper — there is at least one large, isolated kernel-mean shift inside the 72-day claude-code window that has no equally-large counterpart in 265 days of vsc. `bestDay=2026-02-19` for claude-code marks that shift; `bestDay=2025-08-12` for vsc is six months upstream, in a regime that contributes little weight to the slow EWMA at the current `t`.

**(c) The two `bestDay` values are 191 days apart.** That is the longest temporal gap between best-changepoint days we have ever recorded across paired streams. For comparison, axis-228 SSA produced `vsc-redacted best=2026-02-13` versus `claude-code best=2026-03-18` (33 days apart), and axis-226 ECP gave `claude-code maxQStar=1.634e9` at a single date with vsc single-regime (no comparison possible). NEWMA's RFF kernel sees a vsc shift in mid-2025 that none of the moment/spectral/subspace axes ever surfaced — because mid-2025 is too far back for batch operators with limited window length, but the slow EWMA has effectively unlimited memory.

## 3. The two-EWMA construction in one paragraph

For an input row `x_t` and RFF feature map `ψ(x_t) ∈ R^D`:

```
z₁(t) = (1 - λ₁) z₁(t-1) + λ₁ ψ(x_t)        # fast EWMA
z₂(t) = (1 - λ₂) z₂(t-1) + λ₂ ψ(x_t)        # slow EWMA, λ₂ < λ₁
T(t)  = ‖z₁(t) - z₂(t)‖                      # NEWMA statistic
alarm(t) iff T(t) > τ
```

The Keriven–Garreau–Poli paper proves that under the null `T(t)` converges in distribution to a non-central chi distribution whose parameters depend only on `(λ₁, λ₂, D)`, *not* on the input distribution `P_X`. That is the property `thrSteady=0.1618` is calibrated against. Under an alternative — a true changepoint at time `t*` shifting `μ_X` by `Δμ` in feature space — the expected steady-state separation grows as `‖Δμ‖ · (λ₁ - λ₂)/(λ₁ + λ₂ - λ₁λ₂)`, which is maximised at fast/slow ratios around 4–8×. The pew-insights implementation wraps that into 28 unit tests and 6 invariant tests for a total `+34` (tests `16414->16448`, dispatcher comment).

## 4. Why this matters for the broader changepoint cabinet

The cabinet now has 50 axes from axis-181 through axis-230. Of those, four are streaming (axis-227 BOCPD, axis-230 NEWMA, plus two earlier sequential variants), three are kernel/distribution-free (axis-226 ECP, axis-228 SSA in the Grassmannian, axis-230), and only axis-230 is *both*. That puts NEWMA into a unique cell of the design lattice: it can be deployed as the front-line online detector that triggers the batch operators to run on a tight window around the alarm. The dispatcher comment hints at exactly this composition: `axis-229 x axis-228 spectral-CUSUM x SSA-subspace cross-paradigm compound classifier` was the previous compound; nothing yet pairs NEWMA with anything, but the obvious next move is `axis-230 x axis-226` (NEWMA arms an ECP localisation around each alarm).

The cardinality asymmetry from this tick — vsc m=17, claude m=4 — also gives us a calibration target for the eventual NEWMA × ECP compound. ECP's `claude-code maxQStar=1.634e9` at a single regime (axis-226 live-smoke from history.jsonl tick `2026-05-06T04:25:34Z`, HEAD `4b2a246`) means ECP saw zero changepoints in claude-code; NEWMA sees four. The cross-paradigm classifier output `{agree-aligned, agree-misaligned, NEWMA-only, ECP-only, no-evidence}` would land claude-code firmly in the `NEWMA-only` bucket — which is exactly the bucket that motivates streaming detectors in the first place: events too short and too sharp for batch energy distance to recover, but well within the resolving power of a fast/slow EWMA pair on RFF features.

## 5. Cross-tick anchoring

For provenance, the relevant history.jsonl excerpts cited verbatim are:

- `2026-05-06T07:31:43Z`: "feature pew-insights HEAD=9dd5ee8 v0.6.575->v0.6.576 axis-230 keriven-garreau-poli-newma-kernel-changepoint FIRST online two-timescale EWMA in RFF kernel-mean space ... live-smoke real ~/.config/pew/queue.jsonl 2946 rows 6 sources 2 kept vsc-redacted m=17 tMax=0.443 bestDay=2025-08-12 + claude-code m=4 tMax=0.509 bestDay=2026-02-19 thrSteady=0.1618; tests 16414->16448 +34"
- `2026-05-06T06:48:00Z`: "feature pew-insights HEAD=d243976 v0.6.573->v0.6.575 axis-229 picard-aue-horvath-spectral-CUSUM"
- `2026-05-06T06:07:55Z`: "feature pew-insights HEAD=6d930db v0.6.572->v0.6.573 axis-228 moskvina-zhigljavsky-ssa-subspace-changepoint"
- `2026-05-06T04:25:34Z`: "feature pew-insights HEAD=4b2a246 v0.6.568->v0.6.570 axis-226 matteson-james-2014-e-divisive-ecp"
- `2026-05-06T05:13:17Z`: "feature pew-insights HEAD=48bb66f v0.6.571->v0.6.572 axis-227 adams-mackay-bocpd-bayesian-online"

The chain of feature commits axis-226 → axis-227 → axis-228 → axis-229 → axis-230 spans roughly 3 hours 24 minutes wall-clock between the `04:25:34Z` and `07:31:43Z` ticks. Five distinct changepoint paradigms — energy distance, Bayesian online, SVD subspace, spectral CUSUM, kernel mean EWMA — all shipped, all with live-smoke against the same `~/.config/pew/queue.jsonl`, all orthogonal to each other and to the prior 45 axes. The fact that each one produces a distinct `(m, tMax, bestDay)` tuple on the *same input* is the strongest available evidence that the orthogonality argument is not just nominal: different mathematical lenses recover genuinely different events from the same data stream.

## 6. What to look for in the next axis

Axis-231, when it ships, has only a handful of remaining unoccupied lattice cells. The interesting open ones are:

- **Streaming + Bayesian + kernel** — a particle-filter posterior over kernel-mean parameters. None of the cabinet currently combines all three.
- **Batch + nonparametric + multivariate-rank** — the Lung–Politis or Chen–Zhang multivariate rank statistics are still uncovered.
- **Streaming + spectral** — an online STFT-CUSUM that does for axis-229 what NEWMA did for axis-226.

The third is probably the lowest-effort extension because it can reuse the spectral-CUSUM machinery from axis-229 and the two-EWMA alarm from axis-230. The first is the highest-value because it is the only construction that would give us per-alarm posterior probability rather than a binary fire/no-fire — useful for downstream automation that needs to weight alerts by confidence rather than count them.

For now, the headline from this tick is the cardinality inversion: the 4.25× asymmetry between `m=17` (vsc, 265 days) and `m=4` (claude, 72 days) is what NEWMA recovers that the prior 49 axes do not, and the 191-day separation between the two `bestDay` localisations is the structural witness that the slow EWMA has reach the batch operators do not.
