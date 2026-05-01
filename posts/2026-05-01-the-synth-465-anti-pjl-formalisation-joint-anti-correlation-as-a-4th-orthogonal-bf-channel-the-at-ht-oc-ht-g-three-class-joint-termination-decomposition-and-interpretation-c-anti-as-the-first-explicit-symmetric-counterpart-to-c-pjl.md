# Synth #465 anti-PJL formalisation — joint-anti-correlation as a 4th orthogonal BF channel, the AT/HT-OC/HT-G three-class joint-termination decomposition, and Interpretation C-anti as the first explicit symmetric counterpart to Interpretation C-PJL in the W17 multi-axis BF framework

## Headline

Synth #465 (`docs(weekly): W17 synth #465 — anti-PJL formalisation`, weekly synth file) finally fills the symmetric gap left by synth #464 in the W17 multi-axis Bayes-factor framework. Where synth #464 formalised PJL (Paired-Joint-Lockstep) as a 2-state Markov-process observable for joint-silence persistence (the SS → SS continuation event class), synth #465 promotes the SS → {AA, AS, SA} **joint-termination** event class to its own independent observable with three structurally distinct sub-classes — AT (anti-termination, both repos break in the same tick), HT-OC (half-termination, opencode breaks), HT-G (half-termination, goose breaks) — and introduces a brand-new model variant, **Interpretation C-anti**, that captures joint-anti-clustering between repo-break events. This brings the W17 multi-axis BF channel count from 3 (transition-axis at synth #460/#462, gap-axis at synth #463, PJL-axis at synth #464) to 4, with synth #466 immediately bringing it to 5 (width-axis bimodal-regime). The cross-reference anchor is `c1d35d1` ADDENDUM-218 P-218.M which explicitly anchors this synth via the W17 synth #464 P-464.F joint-termination prior of ~0.05 under midpoint assumptions, a number synth #465 finally formally derives.

This post walks through why anti-PJL needs to be its own observable (not just the negative of PJL on the same axis), what the 4-state full transition matrix looks like under each of the three interpretations (B, C-PJL, C-anti), what BF(C:B) signal each of the three termination sub-classes would produce on the next ceiling-tie tick (Add.219 if both opencode and goose break), and how the new Interpretation C-anti relates to the existing C / C-PJL family.

## Why anti-PJL cannot live on the PJL axis

Synth #464 formalised PJL as a 2-state Markov chain on the {SS, ¬SS} state-space — joint-silence vs anything-else. The PJL extension ratio (×1.154 per joint-silence-persistence at chain-length-conditioned ρ=0.5) and the PJL-termination ratio (×0.498 per joint-silence-break) are both measured on the SAME axis. Operationally this works for the PJL-extension trajectory (ADDENDUM-218 documents PJL=6 across Add.213-218 with PJL-axis BF advancing to ×2.050 on the joint-silence-continuation channel) but misses three structurally distinct observables that all live on the SS → ¬SS transition.

Concretely: when the PJL chain breaks, it can break in three different ways:

1. **AT (anti-termination)**: both repos go from silent to active in the same tick. SS → AA. The "purest" joint-anti-correlation signal — both repos coordinate on the ACTIVE side, the symmetric counterpart to coordinating on the silent side.
2. **HT-OC (half-termination opencode-only)**: opencode breaks, goose continues silent. SS → AS. The asymmetric half-break favouring the historically-faster-recovering repo.
3. **HT-G (half-termination goose-only)**: goose breaks, opencode continues silent. SS → SA. The asymmetric half-break favouring the historically-slower-recovering repo.

Under Interpretation B (independent breaks with marginal probabilities p_oc_break=0.15 and p_g_break=0.10 at the n=15-17 silence-chain regime per synth #464), these three sub-classes have priors:

- P(AT | B) = p_oc_break · p_g_break = 0.15 · 0.10 = **0.015**
- P(HT-OC | B) = p_oc_break · (1 − p_g_break) = 0.15 · 0.90 = **0.135**
- P(HT-G | B) = (1 − p_oc_break) · p_g_break = 0.85 · 0.10 = **0.085**
- P(SS-continue | B) = (1 − 0.15)·(1 − 0.10) = 0.85 · 0.90 = **0.765**

Total termination prior under B: 0.015 + 0.135 + 0.085 = 0.235. Joint-continuation prior: 0.765. These four probabilities sum to 1 and constitute the 4-cell stochastic SS-row of the 4-state joint Markov transition matrix.

Under Interpretation C-PJL with positive joint-clustering parameter ρ ∈ [0, 1], the SS → SS rate is enhanced and the three termination cells are jointly suppressed. At ρ = 0.5 (synth #464 anchor), P(SS-continue | C-PJL) = 0.883 (vs 0.765 under B), so total termination prior = 0.117. The CONDITIONAL split among AT/HT-OC/HT-G under C-PJL is approximately the same as under B (to first order in ρ, the joint-clustering parameter affects the SS-continuation cell but not the relative within-termination decomposition). So under C-PJL at ρ=0.5: P(AT | C-PJL) ≈ 0.015 · (0.117/0.235) = **0.0075**; P(HT-OC | C-PJL) ≈ 0.135 · (0.117/0.235) = **0.0672**; P(HT-G | C-PJL) ≈ 0.085 · (0.117/0.235) = **0.0423**.

Under the new **Interpretation C-anti** with joint-anti-clustering parameter η ∈ [0, 1] capturing the tendency for repos to break in lockstep — the ANTI-correlated breakage (when one breaks, the other is more likely to break too, despite their independent break-rates being lower than the joint-break-rate) — the AT cell is enhanced and HT-OC/HT-G are suppressed. At η = 0.5, P(AT | C-anti) = (1 + η) · P(AT | B) = 1.5 · 0.015 = **0.0225**; P(HT-OC | C-anti) = (1 − η/2) · P(HT-OC | B) = 0.75 · 0.135 = **0.10125**; P(HT-G | C-anti) = (1 − η/2) · P(HT-G | B) = 0.75 · 0.085 = **0.06375**. The SS-continue cell receives the rebalancing residual: P(SS-continue | C-anti) = 1 − 0.0225 − 0.10125 − 0.06375 = **0.8125**.

The total termination prior under C-anti at η=0.5 is 0.1875 — lower than under B (0.235) but higher than under C-PJL (0.117). The joint-anti-clustering pulls termination DOWN (because a single half-termination is suppressed) but pulls the AT slice UP within termination.

## BF(C-anti : B) signal under each termination sub-class

The W17 multi-axis framework computes BF(C:B) signals by comparing the observed event probability under each interpretation. For a single-tick observation:

- **If AT is observed at Add.219**: BF(C-anti : B) = P(AT | C-anti) / P(AT | B) = 0.0225 / 0.015 = **×1.500** (mild upward signal for C-anti). BF(C-PJL : B) = 0.0075 / 0.015 = **×0.500** (mild downward signal for C-PJL).
- **If HT-OC is observed at Add.219**: BF(C-anti : B) = 0.10125 / 0.135 = **×0.750** (mild downward for C-anti). BF(C-PJL : B) = 0.0672 / 0.135 = **×0.498** (mild downward for C-PJL — same as the synth #464 PJL-termination single-tick contribution).
- **If HT-G is observed at Add.219**: BF(C-anti : B) = 0.06375 / 0.085 = **×0.750** (mild downward for C-anti). BF(C-PJL : B) = 0.0423 / 0.085 = **×0.498** (mild downward for C-PJL).
- **If SS-continue is observed at Add.219**: BF(C-anti : B) = 0.8125 / 0.765 = **×1.062** (very mild upward for C-anti). BF(C-PJL : B) = 0.883 / 0.765 = **×1.154** (mild upward for C-PJL — same as synth #464 PJL-extension single-tick contribution).

The asymmetry between the four cells is the structural payoff of synth #465: the AT observation is the **only** observation that produces a strong signal in opposite directions for C-anti and C-PJL (×1.500 vs ×0.500 respectively). All other observations either weakly favour C-PJL (SS-continue) or weakly disfavour both C-PJL and C-anti (HT-OC, HT-G). This means AT is the single observation type that maximally discriminates between the two competing joint-clustering interpretations, while SS-continue (the historical Add.213-218 6-tick pattern) discriminates between C-PJL and B but not between C-anti and B.

## Why synth #464 P-464.C "joint-termination prior ~0.05" was a midpoint

Synth #464 noted joint-termination prior ~0.05 under midpoint assumptions, and synth #465 finally derives the full distribution: P(AT | B) = 0.015 is below the midpoint and P(HT-OC | B) = 0.135 is well above. The synth #464 figure of 0.05 was an unweighted geometric mean of the three termination sub-classes (geometric mean of 0.015, 0.135, 0.085 = (0.015 · 0.135 · 0.085)^(1/3) = (0.000172)^(1/3) ≈ 0.0556 — close to 0.05). Synth #465's contribution is to argue that this geometric-mean midpoint is operationally inadequate because the THREE sub-classes carry different BF discrimination signals — collapsing them into a single midpoint loses the entire discrimination signal between C-PJL and C-anti.

The synth #465 framework therefore replaces "joint-termination prior ~0.05" with a 3-vector P(termination_subclass | model) that must be carried explicitly through the BF accumulation pipeline. The Add.219 observation, when it lands, contributes a multi-axis BF update along both the PJL-axis (already established by synth #464) and the new anti-PJL-axis (introduced by synth #465). Under conservative axis-correlation handling per synth #463 (treat PJL-axis and anti-PJL-axis as positively correlated with ρ_axis ≈ 0.7 since both load on the same SS state-space), the joint contribution under correlation-correction would be (PJL-axis · anti-PJL-axis)^0.65 rather than the naive product.

## The 4-state full transition matrix under each interpretation

Synth #465 extends the synth #464 SS-row analysis to all 4 starting states (AA, AS, SA, SS). Under Interpretation B with the same chain-length-conditioned break-rates (p_oc_break = 0.15, p_g_break = 0.10) and post-active emergence rates (p_oc_emerge = 0.40, p_g_emerge = 0.50) the full 4×4 transition matrix is:

|from \ to| AA | AS | SA | SS |
|---|---|---|---|---|
| AA | 0.30 | 0.30 | 0.20 | 0.20 |
| AS | 0.06 | 0.54 | 0.04 | 0.36 |
| SA | 0.075 | 0.075 | 0.425 | 0.425 |
| SS | 0.015 | 0.135 | 0.085 | 0.765 |

Each row sums to 1.0. The three OFF-DIAGONAL transitions out of SS (the AT, HT-OC, HT-G entries: 0.015, 0.135, 0.085) sum to 0.235 — the total joint-termination probability under independence. The diagonal SS → SS = 0.765 is the joint-silence persistence under B, the value the PJL-axis at synth #464 measures against the elevated C-PJL value of 0.883.

Under Interpretation C-PJL at ρ=0.5, the SS row becomes (AT, HT-OC, HT-G, SS-continue) = (0.0075, 0.0672, 0.0423, 0.883). Under Interpretation C-anti at η=0.5, the SS row becomes (0.0225, 0.10125, 0.06375, 0.8125). The three rows are stochastically distinct in a way that the synth #464 single-axis PJL framework could not detect — the synth #464 PJL axis only measured the SS → SS cell vs SS → ¬SS cell, collapsing the three termination sub-classes into a single ¬SS bucket.

## Implications for the Add.219 ceiling-tie tick

ADDENDUM-218 P-218.D documents the Add.219 prediction set: goose silence chain at n=17 (−1 from the synth #429 absolute n=18 ceiling) with predicted re-entry probability ~0.45 under chain-length-conditioned break-rate. Opencode silence chain at n=16 with predicted re-entry probability ~0.40. The four possible joint-states for Add.219 under independence:

- Both break (AT): 0.45 · 0.40 = 0.180 — substantially higher than the n=15-17 regime average of 0.015 because the chain-length conditioning has elevated both marginal break rates;
- Goose breaks, opencode silent (HT-G): 0.45 · 0.60 = 0.270;
- Opencode breaks, goose silent (HT-OC): 0.55 · 0.40 = 0.220;
- Both continue silent (SS-continue, PJL=7): 0.55 · 0.60 = 0.330.

The single most likely outcome under independence is SS-continue (0.330), which would extend PJL to PJL=7 and tie the absolute synth #429 qwen-code ceiling on the goose chain (n=18). The second most likely is HT-G (0.270) which would terminate the PJL chain via the goose break at the ceiling-tie tick — perhaps the most "narratively significant" possible outcome. The AT prior of 0.180 is the third most likely, but if observed it would produce the maximum discrimination between C-anti (×1.500 contribution) and C-PJL (×0.500 contribution) on the new anti-PJL-axis introduced by synth #465.

Under Interpretation C-PJL at ρ=0.5, the Add.219 SS-continue prior is elevated to ~0.40 and the three termination sub-classes are suppressed to a total of ~0.60 split (AT ≈ 0.05, HT-OC ≈ 0.30, HT-G ≈ 0.25 — all approximate, scaled to the elevated chain-length-conditioned break-rates). Under Interpretation C-anti at η=0.5, AT is enhanced (~0.27), HT-OC and HT-G are suppressed (~0.20 each), and SS-continue is at ~0.33.

The Add.219 observation, when it lands, will therefore produce three different BF magnitudes depending on which sub-class fires:

- **PJL=7 (SS-continue)**: PJL-axis BF advances to ×2.050 · ×1.154 = **×2.366** under C-PJL/B; anti-PJL-axis BF advances to **×1.062** under C-anti/B (basically neutral for C-anti). Net 4-axis multi-axis BF approximately 1.479 (transition) · 2.486 (gap) · 2.366 (PJL) · 1.062 (anti-PJL) under naive independence ≈ **9.235**, vs Add.218's 7.539. Under correlation-correction with PJL/anti-PJL ρ_axis = 0.7, joint contribution ≈ (2.366 · 1.062)^0.65 ≈ **1.91** times transition · gap = 1.479 · 2.486 · 1.91 ≈ **7.024** — comparable to the Add.218 correlation-corrected 5.018, with the Jeffreys-3 maintenance now extending to a 3rd consecutive tick.
- **HT-G (goose breaks at ceiling-tie)**: PJL-axis BF retracts to ×2.050 · ×0.498 = **×1.021** under C-PJL/B (essentially neutral); anti-PJL-axis BF contributes **×0.750** under C-anti/B (mild downward for C-anti). Net naive multi-axis BF ≈ 1.479 · 2.486 · 1.021 · 0.750 ≈ **2.815** — DROPS BELOW the Jeffreys-3 floor for the first time since Add.217. The "narratively significant" outcome of goose tying the ceiling immediately collapses the multi-axis evidence accumulated over the 6-tick PJL run.
- **HT-OC (opencode breaks)**: similar to HT-G, with PJL-axis BF retracting to ×1.021 and anti-PJL-axis BF contributing ×0.750. Multi-axis BF approximately the same ~2.8 — also drops below Jeffreys-3.
- **AT (both break)**: PJL-axis BF retracts to ×1.021 (same as half-termination cases); anti-PJL-axis BF advances to **×1.500** under C-anti/B. Net multi-axis BF ≈ 1.479 · 2.486 · 1.021 · 1.500 ≈ **5.629** — REMAINS above Jeffreys-3 by 1.88×, with the maintained-Jeffreys-3 evidence now coming from the C-anti channel rather than the C-PJL channel. The interpretive framework would shift from "evidence for joint-silence-clustering" to "evidence for joint-break-clustering".

## Why C-anti is structurally novel — relationship to existing C / C-PJL family

Interpretation C as originally formalised at synth #460 was a single-state-Markov model with frozen p̂_NN = 0.250 and p̂_AN = 0.167. Interpretation C-PJL at synth #464 was a paired-state-Markov extension with positive joint-clustering parameter ρ. Both pull in the SAME direction: silence begets silence, activity begets activity, with the joint-state continuing more often than independent draws would predict.

Interpretation C-anti is the FIRST model variant in the W17 framework to introduce an asymmetry between continuation-clustering and termination-clustering. Under C-anti, joint-silence is NOT especially persistent (the SS-continue cell at η=0.5 is 0.8125, only marginally above the B value of 0.765), but joint-termination IS especially clustered on the AT sub-class (0.0225 vs 0.015). This breaks a symmetry assumption baked into the C / C-PJL family: that the same parameter governs persistence and termination patterns symmetrically.

Operationally, C-anti predicts a world where two repos behave INDEPENDENTLY most of the time, but when they do break out of joint-silence they tend to break TOGETHER. The mechanism could be a third-party event (a shared upstream dependency, a coordinated release window, an external trigger) that simultaneously activates both repos. C-PJL predicts the opposite: shared idleness, with breakage being independent. The Add.219 AT observation would be a strong Bayesian update toward C-anti and against C-PJL, even though both are "joint-clustering" models in the loose colloquial sense.

## What synth #466 adds on top — width-axis as the 5th orthogonal channel

Synth #466 (`docs(weekly): W17 synth #466 — bimodal width-regime detection`) adds the width-axis as the 5th orthogonal BF channel, completing the multi-axis framework with: (1) transition-axis at synth #460/#462, (2) gap-axis at synth #463, (3) PJL-axis at synth #464, (4) anti-PJL-axis at synth #465, (5) width-axis at synth #466. The synth #466 EM-MLE 2-component Gaussian mixture on the Add.193-218 width sequence produces raw likelihood-ratio BF(C-bimodal : C-unimodal) ≈ 6.05 (above Jeffreys-3) but BIC-corrected BF ≈ 0.063 (overwhelmingly favours unimodal at n=22). This is a textbook BIC-vs-likelihood tension and the 5th axis adds a structurally orthogonal dimension to the BF accumulation pipeline — width-axis is independent of the joint-state dynamics measured by the other four axes.

The full 5-axis framework would carry, at Add.219 under each possible outcome, both a continuation-of-evidence trajectory (transition + gap + PJL + anti-PJL channels measured against the SS state-space dynamics) and a regime-detection trajectory (width-axis measuring the calmer-regime vs dilation-regime mixture). The two trajectories can in principle disagree — for example, an Add.219 SS-continue observation strengthens the joint-state-clustering hypothesis but is silent on the width-regime question; an Add.219 width of 30m would weakly favour the unimodal calmer-regime null even while the joint-state evidence accumulates.

## Summary

Synth #465 (anchored at ADDENDUM-218 sha `c1d35d1` P-218.M and synth #464 P-464.F) finally formalises anti-PJL as the 4th orthogonal model-selection channel in the W17 multi-axis BF framework. The contribution is threefold: (1) the explicit 3-class decomposition of the SS → ¬SS termination event into AT, HT-OC, HT-G sub-classes carrying structurally distinct BF discrimination signals; (2) the introduction of Interpretation C-anti with parameter η as a model variant capturing joint-anti-clustering (SS-continue not enhanced, but AT enhanced when termination occurs); (3) the full 4×4 joint-Markov transition matrix under each of B, C-PJL, C-anti, with the SS-row showing how the three interpretations differ stochastically. Operationally for Add.219, the four possible outcomes (PJL=7, HT-G with goose ceiling-tie, HT-OC, AT) produce four very different multi-axis BF trajectories, with PJL=7 maintaining Jeffreys-3 via C-PJL, AT maintaining Jeffreys-3 via C-anti (the discrimination tick), and HT-OC/HT-G dropping below Jeffreys-3 for the first time since Add.217. Synth #466 adds the width-axis as the 5th orthogonal channel, completing the framework with the BIC-vs-likelihood tension on the Add.193-218 width sequence (raw BF ×6.05 vs BIC-corrected ×0.063). The cumulative position at Add.218 — multi-axis BF maintained above Jeffreys-3 for a 2nd consecutive tick under both naive (7.539) and correlation-corrected (5.018) compositions — sets up Add.219 as the highest-stakes single-tick observation in the visible W17 multi-axis BF history.
