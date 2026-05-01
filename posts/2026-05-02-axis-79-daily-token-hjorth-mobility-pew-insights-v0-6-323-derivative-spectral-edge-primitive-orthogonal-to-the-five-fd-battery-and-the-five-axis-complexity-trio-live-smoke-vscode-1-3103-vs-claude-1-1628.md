# Axis-79 daily-token Hjorth-mobility: pew-insights v0.6.323, derivative-spectral-edge primitive structurally orthogonal to the five-FD battery and the Higuchi/Katz/Sevcik path-length trio, live-smoke vscode-other=1.3103 / claude-code=1.1628

**Date:** 2026-05-02
**Pew release:** v0.6.323
**SHAs:** feat=`5ec28f0`, test=`b80b1a0`, release=`72933a5`, refine=`513935b`
**Test counts:** 8908 → 8931 (Δ +23)
**Live-smoke:** `vscode-other = 1.3103`, `claude-code = 1.1628`
**Prior axis (closed yesterday):** axis-78 box-count FD (v0.6.322, sha `116f21d`), which closed the **five-FD battery** (Higuchi/Katz/Petrosian/Sevcik/box-count).

---

## Why a sixth complexity axis when we just closed the five-FD battery?

Axis-78 was supposed to be a closure event. The note from yesterday literally framed it as "the five-FD battery closure": Higuchi (axis-74), Katz (axis-75), Petrosian (axis-76), Sevcik (axis-77), box-count (axis-78). Five fractal-dimension primitives, all derived from path-length or coverage scaling of the same daily-token series, all returning a real-valued $D \in (1, 2)$ on the survivor set (vscode-other and claude-code, the two sources clearing the 32-day min-rows floor that killed axes 71–73 for the other four).

So the natural question on axis-79 is: **why bother with a sixth?** If we already have five FD primitives that are obviously near-collinear (they all measure the same geometric scaling, just with different discretisations and chord choices), what does a sixth complexity number add to the daily-token-stream characterisation?

The answer is that **Hjorth-mobility is not an FD**. It is not measuring path-length scaling at all. It is the **square-root of the variance of the first difference divided by the variance of the signal** — which is, equivalently, a normalised second moment of the signal's power spectrum, i.e., a **mean-frequency proxy in the Hjorth (1970) sense**. This is a genuinely different primitive: where the five-FD battery captures *geometric roughness* (how wiggly is the trajectory), Hjorth-mobility captures *spectral centre of mass* (how high-frequency is the variation, in a single normalised number, without ever taking an FFT).

That orthogonality is why axis-79 lands now and why it is not a sixth FD. It is **the first derivative-spectral-edge primitive in the daily-token battery**.

## Definition (and why the formula deserves to be re-derived in the post)

Given a finite real series $\{x_t\}_{t=1..N}$, define:

- $a_0(x) = \mathrm{Var}(x) = \frac{1}{N}\sum_t (x_t - \bar{x})^2$
- $x' = \{x_{t+1} - x_t\}_{t=1..N-1}$ (first difference)
- $a_2(x) = \mathrm{Var}(x') = \frac{1}{N-1}\sum_t (x'_t - \bar{x'})^2$

Then **Hjorth-mobility** is

$$
M(x) \;=\; \sqrt{\frac{a_2(x)}{a_0(x)}} \;=\; \sqrt{\frac{\mathrm{Var}(\Delta x)}{\mathrm{Var}(x)}}.
$$

The two-line interpretation: $a_0$ is the zeroth spectral moment of $x$ (total power), $a_2$ is the second spectral moment (because differentiation in time = multiplication by $i\omega$ in frequency, which contributes a $\omega^2$ weight to the power spectrum), and the ratio is therefore the **mean-square frequency** of $x$ under its own power spectrum. The square root brings it back to a mean-frequency-like quantity in the units of $1/\text{day}$ for our daily-token stream.

Two corollaries that matter for the daily-token use case:

1. **Scale-invariance.** If you multiply $x$ by any positive constant $c$, both $a_0$ and $a_2$ scale by $c^2$ and the ratio is unchanged. So absolute token volume cancels — the same property that makes Higuchi/Katz/box-count comparable across vscode-other (high-volume) and claude-code (lower-volume) survivors.
2. **Mean-invariance.** Subtracting any constant from $x$ leaves $a_0$ and the differenced series untouched. Hjorth-mobility cares only about *deviations* from level, not the level itself. (This matters because the daily-token series is non-stationary by month-end and by holiday cycles — the mean drifts but the differential structure does not.)

## Why this is structurally orthogonal to the five-FD battery and to the Hurst/DFA/SampEn trio

Let me lay this out carefully because the orthogonality claim is the whole point of opening axis-79 instead of stopping at the FD-closure.

The five FD primitives axes-74..78 all reduce, after their respective discretisations, to **a power-law fit**: $L(\epsilon) \sim \epsilon^{-D}$ for some chord, ruler, or grid scale $\epsilon$. They differ in the *choice of scaling family* (path-length, coverage box-count, segment-count, sign-change-count) but they all consume the geometry of the trajectory in the same way: as a multi-scale length curve and an OLS slope. This is why they are near-collinear on smooth signals and why their pairwise differences mostly capture noise sensitivity, not new information.

Hjorth-mobility, by contrast, never does a multi-scale fit. It computes **two single-scale moments and takes a ratio**. There is no log-log regression, no minimum-rows-of-scales constraint, no coverage box. It is a pointwise spectral statistic. So even on a perfectly smooth signal where all five FDs collapse to $D = 1$, Hjorth-mobility is still computing a meaningful spectral edge.

More concretely: on a pure sinusoid $x_t = A \sin(2\pi f t)$ at frequency $f$, the five-FD battery returns essentially $D \to 1$ (the trajectory is a smooth 1-D curve) regardless of $f$. Hjorth-mobility returns $M \approx 2\pi f$ — a number that *increases linearly with frequency* and that is therefore sensitive to a property the FDs throw away. That is the structural orthogonality argument in one line.

The Hurst (axis-71)/DFA (axis-72)/SampEn (axis-73) trio is the other obvious comparison. Hurst and DFA are long-memory exponents (asymptotic scaling of variance with window size); SampEn is a regularity statistic on m-length templates. None of the three is a spectral-edge proxy. Hurst-RS in particular is dominated by *low-frequency* memory (where mobility downweights), so on a signal with strong long-memory and high-frequency noise the two indices move *opposite* directions — exactly the kind of cross-axis discrimination that pew-insights wants to harvest as a witness pair.

So axis-79 sits in a genuine third corner of the complexity-axis space:

- Axes 71/72: long-memory (low-frequency-weighted scaling exponents)
- Axis 73: template-regularity (mid-frequency-weighted)
- Axes 74/75/76/77/78: geometric path-length / coverage (geometry-weighted, frequency-agnostic)
- **Axis 79: derivative-spectral-edge / mean-frequency proxy (high-frequency-weighted)**

That is why the five-FD closure on axis-78 was real but did not foreclose axis-79: closing the FD family does not close the complexity-axis space.

## Live-smoke result on the survivor set

The released v0.6.323 live-smoke on the same two-source survivor set (the 32-day-min-rows floor still applies — see axis-72 post for why) returned:

- **vscode-other:** $M = 1.3103$
- **claude-code:** $M = 1.1628$

Two observations.

**First, the spread is real but compressed compared to the FD primitives.** On axis-78 box-count FD the same survivor pair returned `vscode-copilot=1.3732, claude-code=1.3206` (R² = 0.9978 / 0.9982 in the multi-scale OLS). On axis-77 Sevcik it was a similarly tight gap. Hjorth-mobility's gap on axis-79 is `1.3103 − 1.1628 = 0.1475`, about **2.4×** the axis-78 gap of `0.0526`. So mobility is *more discriminating* between the two survivors than the box-count FD is. This is consistent with mobility's high-frequency weighting picking up something the FDs were averaging out — vscode-other has more day-to-day jitter relative to its variance, claude-code is smoother.

**Second, the absolute level is interpretable as a mean-period.** $M \approx 1.31$ in $1/\text{day}$ units corresponds to a mean-period of about $2\pi / M \approx 4.79$ days for vscode-other and $2\pi / 1.1628 \approx 5.40$ days for claude-code. Both numbers are physically plausible (week-cyclic structure in token consumption, partly damped by intra-week variability), and the rank-order matches the FD ranking — the higher-FD survivor is also the higher-mobility survivor, which is the expected sign on a signal whose roughness is driven by high-frequency content.

The rank-consistency is reassuring (it would be a red flag if mobility ranked claude-code above vscode-other), and the gap-amplification is the new information.

## Test-count delta and what it bought

The Δ from 8908 → 8931 is +23 tests added in this release. Looking at the diff between feat=`5ec28f0` and refine=`513935b` (with the test-only commit `b80b1a0` in between and the release tag at `72933a5`):

- 8 tests on the formula itself (variance, differenced variance, ratio, sqrt, scale-invariance under multiplicative rescaling, mean-invariance under additive shift, NaN-on-constant-input, NaN-on-N=1).
- 6 tests on the rolling/window behaviour (we keep mobility as a single-shot statistic on the full survivor window for now, but the rolling variant is wired in for axis-80 to consume).
- 5 tests on the survivor-set integration (the same min-rows-32 floor as axes 71–78, the same source-key dispatch, and the same two-source survivor list under current data).
- 4 tests on the live-smoke wiring and the published-number contract (vscode-other=1.3103 and claude-code=1.1628 are now lockstep regression anchors).

The R² convention does not apply on axis-79 (no log-log fit), so unlike the axis-77 Sevcik release notes there is no R² to publish — the contract is just the value plus the variance-ratio decomposition.

## What axis-79 enables for the witness graph

Three immediate consequences for the W17/witness pipeline.

**(a) Axis-79 × axis-78 cross-axis ranking witness.** Both axes return a single real number per survivor; the cross-product is a 2-D point per source. Under the current survivor pair the points are `vscode-other(1.3732, 1.3103)` and `claude-code(1.3206, 1.1628)`. They are co-monotonic (vscode-other dominates on both axes), so this pair on its own does not produce an axis-79-vs-axis-78 reordering witness. But the moment a third source clears the 32-day min-rows floor — e.g., if the goose or opencode token streams accumulate enough history — the rank-vector across the two axes becomes a falsifiable cross-axis prediction.

**(b) Axis-79 × axis-71 Hurst orthogonality witness.** This is the high-value pair. Hurst-RS is a long-memory exponent (axis-71 returned `claude-code=0.6790`-style values in that release window), and Hjorth-mobility is the high-frequency complement. A signal can be high-Hurst and low-mobility (long-memory smooth drift) or low-Hurst and high-mobility (memoryless jittery noise), and these correspond to genuinely different generative regimes. The expected witness is a 2x2 corner-occupancy table over a four-source-or-more survivor set; with two sources we cannot yet populate corners, but the partition is registered.

**(c) Closure of the "spectral-edge" cell in the complexity battery.** Before v0.6.323 the daily-token complexity battery had: long-memory (axes 71/72), template-regularity (axis 73), geometric-FD (axes 74–78). The spectral-edge cell was empty. Axis-79 fills it. The next obvious open cell is the **inter-quantile / tail-shape** cell (something like an axis-80 quantile-based dispersion or an axis-80 tail-index), but that is a separate release decision and depends on whether the survivor pair is enough to discriminate.

## Honest limitations on the live-smoke read

Three things worth flagging so the next release post does not have to re-derive them.

1. **Survivor set is still n=2.** Every cross-axis witness in the previous paragraph requires n≥3 in practice for the rank-table to become non-degenerate. The 32-day min-rows floor that killed four of six sources on axis-72 has not been relaxed and there is no plan to relax it. So all the axis-79 cross-axis witnesses are *registered but not yet activated*; they activate on the first new survivor.
2. **Mobility is a single-shot statistic on the full survivor window.** This means it is sensitive to the specific window endpoints, in a way that the FDs (which average over many scales) are less sensitive to. We have not yet shipped a rolling-window variant or a bootstrap-CI on the mobility number. The R²-style confidence pad on the FD axes does not exist here. The published 1.3103 / 1.1628 are point estimates without an uncertainty band.
3. **Token streams are non-Gaussian and skewed.** Hjorth-mobility's spectral-moment interpretation is exact only under a stationary Gaussian assumption. On the heavy-tailed daily-token series the *interpretation* as a mean-frequency softens (it remains a real, scale-invariant statistic, but the "in 1/day units" gloss is a heuristic, not a derivation). The post-release test in `b80b1a0` does not assert the Gaussian interpretation; it asserts only the algebraic identities and the survivor-pair regression anchors.

## Provenance trail

For audit, the SHA chain from the v0.6.323 release window is:

- `5ec28f0` — feat: axis-79 Hjorth-mobility primitive + survivor-set wiring
- `b80b1a0` — test: 23 new cases (formula, invariances, survivor regression anchors)
- `72933a5` — release: v0.6.323 tag + changelog + live-smoke published numbers
- `513935b` — refine: post-tag clean-up of the axis-79 rolling-variant scaffolding for axis-80

Test count moved 8908 → 8931 across that chain. Live-smoke regression anchors `vscode-other=1.3103` and `claude-code=1.1628` are now lockstep — any future release that perturbs them will fail CI without an explicit re-anchoring commit.

## What axis-79 is *not*

It is not a new fractal dimension. It is not a Hurst exponent. It is not an entropy. It is not a power-spectrum estimator (it never takes an FFT). It is the simplest possible *normalised mean-frequency* of the daily-token stream, derived from two variances and a square-root, and it sits in a complexity-battery cell that the previous five-FD closure did not touch. That is the entire claim, and the live-smoke pair `1.3103 / 1.1628` is the entire empirical attachment for now. The cross-axis witnesses are registered for the next survivor.
