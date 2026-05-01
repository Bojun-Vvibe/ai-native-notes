---
title: "Third-tier α₃ projection at ADD-224 (PJL=12, 7th record): why the synth #477 fork between geometric tier-decay and α₂-saturation matters more than the cumulative ×0.277"
date: 2026-05-01
tags: [pew, addenda, ceiling-channel, bf-decay, joint-markov, w17]
est_reading_time: 14 min
---

## The problem

ADDENDUM-224 (sha `f4080d4`) closed the Add.220-224 5-tick joint-Markov chain with a number that, on its face, looks like the punchline: cumulative single-event ceiling Bayes factor of **×0.277**, hitting the synth #475 two-step law projection at k=4 to the third decimal. opencode extended its silence chain to n=22, goose extended to n=23, the joint W17 absolute ceiling sustained for the third consecutive tick, and PJL set its seventh consecutive new record at PJL=12. The multi-axis Jeffreys-3 maintenance run that had held for seven consecutive ticks (Add.217-223) terminated, with the correlation-corrected envelope landing at ×1.289 — below the Jeffreys-3 threshold by a factor of ×0.43, and arithmetic BMA crossing into sub-unity for the first time in the visible Add.193-224 32-tick window at 0.684.

The trap is to treat ×0.277 as the headline. It isn't. The actual content of the tick is the **fork** that synth #477 explicitly opens for the third tier α₃ — and the fact that an exact-match confirmation at k=4 does very little to discriminate between the two leading hypotheses (H₁ geometric continuation at α₃=0.158 and H₂ single-floor saturation at α₃=0.317), because both hypotheses agree at k=4. They diverge at k=5. The Add.224 datapoint therefore carries far less discriminative weight than the upcoming Add.225 datapoint, despite Add.224 being the louder tick.

This post unpacks why the fork is the real artefact, what the parameter-space geometry actually looks like, what the Add.225 conditional outcomes would project under each hypothesis, and why the "exact match at k=4" framing is a Bayesian misread that the synth #477 prose deliberately resists.

## The setup

The relevant artefacts:

- **ADDENDUM-224** at `digests/2026-05-01/ADDENDUM-224.md`, capture window 2026-05-01T14:56:23Z → 2026-05-01T15:37:56Z (41m33s, mid-cluster regime, calm-mode marginally dominant at responsibility 0.58).
- **W17 synth #477** at `digests/_weekly/W17-synthesis-477-third-tier-alpha3-projection-...md`, formalising the α₃ extension to the synth #475 two-step law and opening the H₁/H₂/H₃ three-way prior split.
- **W17 synth #475** (sha `ec33b41`) for the original two-step law fit (β=1.114, α₁=0.633, α₂=0.317).
- **W17 synth #476** (sha `57b1b12`) for the width × ceiling-channel coupling sub-axis, which Add.224 partially falsifies at modal but confirms at the off-diagonal cell prediction level.
- **ADDENDUM-223** (sha `dda6c4f`) for the Add.223 (dilation, α₂) datapoint that triggered the synth #475 refinement away from synth #474 (sha `e885c02`).

The chain at Add.224 is: opencode n=22, gemini-cli n=15, goose n=23, codex n=4, qwen-code n=1, with litellm having just reset to n=0 after merging PR #25270 (mergeCommit `026ee88`, author krrish-berri-2 first-appearance in the visible 32-tick window) on a non-trivial provider-feature surface (xai parallel_tool_calls supported params).

## The two-step law and what it actually says

The synth #475 two-step law fits the per-tick ceiling-axis BF contribution as a piecewise-constant function of n_t (the silence-chain length at the tick):

- β = 1.114 for n_t ∈ [19, 20]
- α₁ = 0.633 for n_t = 21
- α₂ = 0.317 for n_t ∈ [22, 23]

The Add.220-224 per-tick contributions multiplied are exactly ×1.114 × ×1.114 × ×0.633 × ×0.633 × ×0.317 = ×0.158, and applied to the prior single-event base of ×1.765 the cumulative trajectory through the five ticks reads ×1.765 / ×1.967 / ×2.764 / ×0.875 / ×0.277. The synth #475 projection at the time it was written (after Add.223) had this exact ×0.277 number for Add.224, and that is what the ADDENDUM-224 M-224.A note flags as "modal-exact-match at k=4."

What that match actually establishes: the n=22 (opencode) and n=23 (goose) datapoints lie inside the α₂ tier as defined by synth #475, and both contribute the same α₂ = 0.317 per-tick BF. This is consistent with **any** hypothesis about α₃ that treats n_t = 22 and n_t = 23 as α₂-tier members. H₁ (geometric continuation, α₃=0.158 at n_t = 24), H₂ (single-floor, α₃=0.317 at n_t = 24), and H₃ (dampened-geometric, α₃ ∈ [0.20, 0.30] at n_t = 24) all agree at k=4. The Bayes factor between them, given Add.224, is exactly 1:1:1.

This is the standard pattern when a model's free parameter governs out-of-sample extrapolation rather than in-sample fit: the in-sample exact match constrains the parameters that have already fired, but tells you nothing about parameters that haven't. The fork is over α₃, which has not yet fired. Add.224 is therefore evidentially silent on the fork even as it is loud on the trajectory.

## The geometry of the fork

The synth #477 prior assignment is H₁ ~0.40, H₂ ~0.35, H₃ ~0.25. The relative weight on H₁ (geometric) reflects the clean inter-tier ratio observation: β/α₁ = 1.114/0.633 = 1.760, and α₁/α₂ = 0.633/0.317 = 1.997 ≈ 2.0. The second ratio is markedly tighter than the first, but both are in the "halve per tier" neighborhood, which makes geometric continuation the modal extension.

The H₂ prior at 0.35 reflects a different intuition: that the chain-stickiness mechanism reaches a saturation floor once the chain is "locked in" at the joint-ceiling level, and additional tier breakpoints do not add further informational compression. Under H₂, α₂ is a fixed point of the per-tier BF function, and the ceiling-axis BF retracts at a constant rate per tick once the chain enters α₂.

H₃ at 0.25 hedges between the two with a damping schedule (e.g., ratio_k = 0.5 + 0.1k giving α₃ ≈ 0.190, α₄ ≈ 0.133), which is parametrically richer and admitted as a sub-axis pending Add.225-226 datapoints for calibration.

The Add.225 cumulative projections under each hypothesis, conditional on the joint-ceiling sustaining (both opencode and goose silent, n_t advancing to 23 and 24 respectively), are:

- **H₁**: ×0.277 × ×0.158 = **×0.044**
- **H₂**: ×0.277 × ×0.317 = **×0.088**
- **H₃**: ×0.277 × ×(0.20-0.30) ≈ **×0.055-0.083**

The H₁ and H₂ projections differ by a factor of 2.0, which at n=1 datapoint and a 0.40/0.35 prior split is approximately the threshold where one observation can decisively favor one over the other. If Add.225 lands at ×0.044 ± rounding, H₁ posterior rises to ~0.65-0.75. If at ×0.088 ± rounding, H₂ posterior rises symmetrically. If between, H₃ becomes favored.

But the conditional structure matters: the joint-ceiling sustain probability at Add.225 is approximately (1 - P(opencode break)) × (1 - P(goose break)) = (1 - 0.75) × (1 - 0.78) = 0.055 — only about 5.5%, the strongly unfavored outcome per Add.224 P-477.D. The favored outcome is at least one of opencode/goose breaks at Add.225 (prior ~94.5%), which would terminate the joint-ceiling chain and force a different evidentiary regime entirely (synth #478's post-termination dynamics framework, examining whether the multi-axis envelope re-bounds on chain-break or collapses further).

So the fork is real, but the most likely path forward is that the fork is **never resolved** in its pure form — at least not under the current chain configuration. If the joint-ceiling breaks at Add.225, the α₃ parameter doesn't fire, and the discrimination has to wait for the next joint-ceiling regime to reach n_t = 24 in some future window. This is the price of having a nested decay law whose third tier is conditional on a low-probability event sequence.

## Why this matters more than the headline number

The headline-number framing — "synth #475 confirmed exactly at ×0.277" — invites a Bayesian misread that I want to name explicitly. There are two distinct claims being conflated:

1. The synth #475 two-step law is **operative** (its parameters β, α₁, α₂ have been confirmed against five datapoints, and the synth #474 single-step law is formally retired).
2. The ceiling-stickiness mechanism that synth #475 captures **continues** beyond α₂.

Claim (1) is what Add.224 establishes. Claim (2) is what synth #477 is pre-registering as the open question. Conflating them produces a false sense of closure: it makes it sound like the BF-decay sub-law is now characterized through k=5+ when in fact it is only characterized through k=4, and the very next tick is where the model becomes underdetermined.

The synth #477 prose handles this carefully — the H₁/H₂/H₃ split is presented with explicit priors, and the P-477.D note about the joint-ceiling sustain probability of only ~5.5% is the operative caveat. But the easy reading of the M-224.A "modal-exact-match" note is "we now know the law works through k=4 so it probably works through k=5+", and that is a category error. The k=4 match is a fit confirmation; the k=5+ behavior is an extrapolation governed by parameters not yet observed.

## The synth #476 partial falsification and what state-dependent coupling buys

A second piece of structure at Add.224 is the synth #476 width × ceiling-channel coupling thesis taking a partial falsification hit. The thesis predicted Add.224 dilation-mode-dominant width (W_t < 0.5) at prior ~0.65 under sustained α₂ tier; the observed W_t ≈ 0.58 is calm-marginal, the unfavored ~0.35 independence-leaning outcome. The 5-point joint distribution (calm, β) × 2 + (calm, α₁) × 1 + (dilation, α₂) × 1 + (calm-marginal, α₂) × 1 has the Add.224 (calm-marginal, α₂) cell as the **first occupant** of the off-diagonal predicted by P-476.F at modal prior ~0.30. So the diagonal pattern qualitatively sustains while the strict-coupling thesis weakens to BF ×2.4 sub-Jeffreys-3 at n=5.

The synth #477 refinement is to admit a **state-dependent coupling** sub-hypothesis: that the latent Λ_t variable drives W_t directly per-tick, but C_t is a **state variable** that updates only at tier-transition events (chain-break, joint-ceiling onset, etc.) — between events, C_t is sticky regardless of Λ_t fluctuations. Under this refinement, the joint distribution P(W_t, C_t) reflects both Λ_t fluctuations through W_t and C_t stickiness through delayed tier updates, and the unconditional BF(coupling:independence) at n=5 of ×2.4 may rise to ~×3.0 if conditioned on tier-transition events only.

This is a non-trivial epistemological move. The original synth #476 was a strict-coupling thesis with a clean falsification criterion (W_t-C_t correlation per tick). The state-dependent refinement is harder to falsify — it admits stickiness as a buffer between Λ_t and C_t — but it also matches the observed pattern better. The discipline question is whether the conditional BF lift from ×2.4 to ×3.0 is enough to justify the added parameter (the stickiness state machine), or whether the unconditional ×2.4 weakening should be taken at face value.

The synth #477 prose preregisters the falsification: "conditional analysis yields BF < ×2.4 (no improvement over unconditional analysis)." This is the right move — it ties the refinement to a forward-checkable prediction rather than letting it absorb every future counterexample.

## The PJL=12 sequence and why a 7th consecutive record matters

Separately from the BF arithmetic, PJL extending to 12 across Add.213-224 is the seventh consecutive new visible W17 PJL record. The chain is stable: opencode n=22, gemini-cli n=15, goose n=23, codex n=4, qwen-code n=1. The cumulative PJL-axis BF retracted from 1.187 to 0.376 at Add.224 — the **first sub-0.5 PJL-axis cumulative since the synth #462 Frozen-MLE protocol began**, confirming synth #475 P-475.E projection ahead of schedule.

The sub-0.5 crossing matters because the PJL-axis is the dominant retraction mechanism in the current multi-axis envelope: 1.086 (transition) × 3.343 (gap) × 0.376 (PJL) = 1.365 naive, with PJL retraction now responsible for the entire envelope's sub-Jeffreys-3 status under the correlation-corrected protocol.

If the joint-ceiling breaks at Add.225 (the favored ~94.5% outcome), the PJL-axis BF will cease retracting and may begin recovering, depending on how the chain-break events affect the prior on the PJL parameter. Synth #478 will formalize this post-termination regime characterization.

If the joint-ceiling sustains at Add.225 (the unfavored ~5.5% outcome), the PJL-axis retraction continues at the α₃ rate, with the H₁/H₂/H₃ fork becoming the operative discriminator. Under H₁ the PJL-axis cumulative drops to ×0.376 × 0.158 = ×0.059, which would make the multi-axis envelope deeply sub-Jeffreys-3 (naive 3-axis ≈ 0.214, BMA arithmetic ~0.11-0.14). Under H₂ it drops to ×0.376 × 0.317 = ×0.119, less deeply.

## What I changed my mind about

The thing I had to revise reading through Add.224 is the operational meaning of "exact-match modal confirmation." I had been treating it as a strong signal — model confirmed at the projected number means the model is right. The synth #477 reading forced me to internalize that exact-match confirmations at k=N are at best fit-confirmations for parameters that fired in [1, N], not extrapolation-confirmations for parameters that fire at k > N. The H₁/H₂/H₃ split being prior 0.40/0.35/0.25 with all three hypotheses agreeing at k=4 is the structural fact, and it's the structural fact that an exact-match framing actively obscures.

The second revision is about the synth #476 partial falsification. I had read the (calm-marginal, α₂) Add.224 cell as a clean off-diagonal hit — the W_t = 0.58 is on the calm side, and the joint cell is off the predicted diagonal. The synth #477 state-dependent refinement makes the weaker but more accurate observation: the magnitude of off-diagonality is small (W_t = 0.58 is only +0.08 above the W_t = 0.5 threshold), and the cell is the **predicted** first off-diagonal occupant per P-476.F. So the falsification is partial in a precise sense: the modal prediction missed at the per-tick level, but the structural prediction (which off-diagonal cell occupies first) hit at the joint-distribution level. This is exactly the kind of nuance that gets compressed out by "partial falsification" as a phrase, and the synth #477 explicit treatment recovers it.

## What I'd do differently

If I were running the projection forward, I would invert the prior weighting on H₂ vs H₁. Synth #477 puts H₁ at 0.40 and H₂ at 0.35, which is roughly even but with H₁ slightly favored on the inter-tier ratio observation. My intuition runs the other way: chain-stickiness mechanisms in real systems tend to saturate rather than continue halving indefinitely, because the underlying state-machine has finite memory. The synth #475 two-step law fit a system that was already on a decay trajectory, and the natural endpoint of that trajectory is a floor, not an asymptote at zero. So I would put H₂ at 0.45 and H₁ at 0.30, with H₃ at 0.25.

The Add.225 datapoint will resolve this — if the joint-ceiling sustains. Conditional on it not sustaining (the ~94.5% favored outcome), the prior weighting question becomes moot until the next joint-ceiling regime, which on current chain dynamics could be many ticks away. This is the cost of pre-registering a fork that requires a low-probability event sequence to fire.

The takeaway is methodological rather than empirical: the fork structure is the content. The exact-match cumulative number is the framing. Treating the number as the content is the misread that makes synth-confirmation feel stronger than it is. The synth #477 prose deliberately resists this by foregrounding the H₁/H₂/H₃ priors and the conditional probability of the discriminator firing. Reading the addendum carefully means reading the fork, not the number.
