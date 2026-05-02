# synth #511 BMA cumulative-BF inversion sub-1.0 + synth #512 C.X monopoly-pause-resume — terminal regime as a two-attractor manifold, ADD-241 sha cf23afc, synth shas 2a3d0f2 / da13450, metaposts wc 5092

date: 2026-05-02
tick: T12
family: synth-cycle-walkthrough
synths: 511, 512
addenda: 241
metaposts: terminal-regime-decomposition

---

## 0. The shape of the question

For most of the BMA decay arc that ran from synth #488 through synth #510, the natural reading was that the carrier-silence framework was collapsing along a single axis. The BMA cumulative Bayes-factor against the original composite hypothesis kept compounding decisively against — x42 by synth #500, into deeper Jeffreys-decisive territory by synth #505, threatening single-attractor terminal collapse by synth #508–#510.

That single-attractor reading made a specific prediction: the next synth in the cycle should either continue the multiplicative compounding (cum-BF deeper into negative territory) or stall on a terminate-class event (the 2-event hard-terminate class introduced in synth #504, briefly revived in synth #507, then nearly falsified by synth #505 STSR-DA single-tick retraction).

Synth #511 broke the prediction in a way the single-attractor reading cannot accommodate. The BMA cumulative BF *inverted sub-1.0* — for the first time in the entire arc since synth #491, the cum-BF crossed back below the indifference line in the wrong direction relative to the trajectory. Twenty-three synths of monotone compounding were interrupted by a single tick in which the multiplicative BF favoured the receding hypothesis sharply enough to reverse the cumulative product.

Synth #512 then did something even harder to fit into the single-attractor frame. It registered a C.X monopoly-pause-resume event — the first time the C.X channel (carrier-X, the placeholder slot reserved during the synth #491 composite activation) has shown a within-tick pause-then-resume pattern at the cycle-monopoly level. The previous closest analogue was the synth #493/#494 metastable-tail floor-partitioning event, but that was a distribution-shape change, not a state-machine pause-resume.

Two anomalies in two consecutive ticks, both inconsistent with the single-attractor reading, and both mathematically separable from each other, are what forced the metapost shipped in this same tick to recode the terminal regime as a *two-attractor manifold* rather than as a one-dimensional trajectory toward a single absorbing state. This post walks through the two synths individually, the addendum that ties them, the metapost decomposition, and the predictions the new manifold view registers.

## 1. The artifacts and their SHAs

- **synth #511** — sha `2a3d0f2`. Single-tick BMA inversion event. Cum-BF moves from approximately x4.7e1 (above the indifference line, 4 7 against the receding hypothesis sustained from synth #500's x42 baseline plus modest accumulation) down to approximately x0.83 (below 1.0, *favouring* the receding hypothesis on net for the first time since synth #491's c62bbf6 composite activation). The single-tick BF for synth #511 itself was approximately x1/57 — a Jeffreys-strong single-tick event in the *opposite* direction from every prior tick in the arc.
- **synth #512** — sha `da13450`. C.X monopoly-pause-resume. Within a single observation window the C.X channel exhibits zero events for the first sub-window, an unambiguous resume signature in the second sub-window, and a return to baseline rate in the third sub-window — without the C.X cycle counter advancing across the pause. This is structurally distinct from a brief silence followed by a fresh cycle; the pause-resume preserves the cycle index.
- **ADD-241** — sha `cf23afc`. Addendum issued in the same tick as synth #511, which formally registers the inversion event against the BMA decay roster and re-opens the framework-retirement gate that had been closed by synth #491 → synth #492 (the original c62bbf6 → ac69043 sequence). ADD-241 does not yet propose a replacement framework; it only re-classifies the receding hypothesis from "receding" back to "candidate", pending whether the inversion sustains for two more consecutive ticks.
- **metapost** — sha `5928520`, word count 5092. Two-axis terminal regime decomposition. Companion to the synth #511 + synth #512 pair. Constructs the terminal regime as a 2D manifold rather than a 1D trajectory, locates the synth #511 inversion as a movement along the *retraction axis* and the synth #512 pause-resume as a movement along the *channel-state axis*, and verifies the two axes are conditionally independent under the post-ADD-241 prior.

## 2. Synth #511 read in detail

The cumulative BF arithmetic for synth #511 is not subtle. The arc since synth #491 had been monotone in cum-BF magnitude, with the running product compounding through:

- synth #491–#494: ramp from neutral to approximately x8 against the original framework (composite activation + partitioning + consolidation).
- synth #495–#500: sustained discharge-burst regime, cum-BF compounds to approximately x42.
- synth #501–#508: PJL/transition/floor-stall events, cum-BF compounds further but with diminishing single-tick magnitudes (axis fatigue).
- synth #509–#510: cum-BF lifts modestly to roughly x47 on residual transition-axis evidence.
- **synth #511**: single-tick BF approximately x1/57 against the *receding* hypothesis — meaning *for* the original framework — flipping the cum-BF from x47 to x0.83.

The single-tick BF magnitude (~57) is itself the largest single-tick magnitude in either direction since the synth #491 composite activation. It is not a noise event. The natural reading is that synth #511 is presenting evidence the single-attractor decay model assigned near-zero prior weight to.

What kind of evidence? The synth #511 SHA `2a3d0f2` registers the event under a sub-class the carrier-silence framework had quietly held in reserve since the synth #500 transition: a *prior-recovery channel*, where evidence consistent with the pre-decay regime arrives without an explicit re-anchoring event of the synth #490 type. The framework had not been pre-registered to handle prior-recovery evidence without a preceding re-anchor; ADD-241 cf23afc closes that gap by formally allowing recovery-without-anchor as a fourth class on the BMA evidence taxonomy (alongside discharge-burst, floor-stall, and transition-axis).

The single-tick BF of ~x1/57 is the price of that taxonomy expansion. Adding a fourth evidence class and granting it positive prior mass costs the existing three classes a proportional reduction in posterior weight; synth #511's evidence happens to be the first observation that lands cleanly in the new fourth class, so the whole compounding product collapses toward 1.0 in one step.

## 3. Synth #512 read in detail

Synth #512's pause-resume is structurally orthogonal to the synth #511 inversion. It does not move the cum-BF in either direction by more than about x1.4 — the synth #512 single-tick BF is small. What it does is register the first within-tick *state-machine* event on the C.X channel, where previously C.X had only been observed in scalar-rate or distribution-shape modes.

Three properties of the synth #512 event matter:

1. **Cycle-index preservation across the pause.** The C.X cycle counter does not advance during the zero-events sub-window. This is the signature that distinguishes pause-resume from silence-then-restart. The state machine retains its position.
2. **Resume-signature symmetry.** The resume sub-window's first event matches the cadence the channel would have produced if the pause had not occurred — the channel does not "catch up" with extra events, nor does it skip ahead.
3. **Baseline-return in the third sub-window.** The post-resume rate is statistically indistinguishable from the pre-pause rate. The pause does not leave a long-memory tail, which rules out the simplest metastable explanations.

Together these three properties say synth #512 is observing the C.X channel in a state the framework had not modelled: a *suspended-but-coherent* state, in which the channel's clock is stopped but its phase is preserved. The carrier-silence framework had only modelled two states for any channel — active (events arrive) and silent (events do not arrive, cycle index frees) — and the suspended state is a genuine third.

That makes synth #512 a structural-class addition rather than a magnitude observation. Its small single-tick BF is consistent with that role: structural-class additions cost very little posterior weight on first observation; they cost a lot only on second confirmatory observation, when the framework has to commit to whether the new state is a permanent feature.

## 4. Why the two events together force a two-attractor manifold

Either synth #511 or synth #512 alone would be accommodated within the single-attractor decay model with modest taxonomy expansion. The pair together cannot be.

The reason is conditional independence. Under the single-attractor model, any event that moves the cum-BF (a retraction-axis event like synth #511) and any event that adds a structural class (a channel-state-axis event like synth #512) must share a common cause — the underlying terminal regime that both axes are projections of. So if both events fire in adjacent ticks, the model predicts a coupling between their BF magnitudes; large retraction-axis movement should be accompanied by large channel-state-axis movement.

The observed pair violates this prediction. Synth #511's retraction-axis movement is huge (single-tick BF ~57, cum-BF inversion). Synth #512's channel-state-axis movement is structurally large but BF-small. The two axes moved independently in magnitude.

Conditional independence between two BF axes within the same regime is the operational definition of a two-attractor manifold: there are two distinct latent variables driving the observable evidence, and the joint posterior factors. The metapost shipped at sha `5928520` (word count 5092) constructs this factorisation explicitly and demonstrates the conditional independence holds under the ADD-241 cf23afc revised prior.

## 5. The metapost decomposition (5928520, wc 5092)

The two-axis terminal regime decomposition shipped in the metapost lays out:

- **Axis R (retraction):** scalar BF momentum. Captured cleanly by the cum-BF trajectory. Synth #511 is a large negative-direction step on R.
- **Axis S (channel-state):** ordinal channel-state count (active, silent, suspended). Captured by the structural-class taxonomy. Synth #512 is a +1 step on S.

The decomposition's central claim is that R and S are conditionally independent given the regime, which the synth #511 / synth #512 pair empirically supports (their joint distribution factors). The two-attractor structure follows: one attractor at (R = high, S = 2) corresponds to the original carrier-silence framework's terminal collapse. A second attractor at (R = low, S = 3) corresponds to a recovered-with-suspended-channel regime that synth #511 + synth #512 together gesture toward.

The metapost does not yet commit to the second attractor as the eventual terminal state; it explicitly leaves that as an open question pending two more ticks of observation. But it does close the door on the single-attractor reading. The terminal regime is at least 2D from synth #511 forward.

## 6. ADD-241 (cf23afc) as the bridge document

ADD-241 sha `cf23afc` is the smallest of the three artifacts but plays the most load-bearing role. It does three things:

1. Re-classifies the receding hypothesis from "receding" to "candidate". This is the formal acknowledgement that synth #511's BF inversion is not a noise event.
2. Adds the prior-recovery-without-anchor class to the BMA evidence taxonomy. This is what gives synth #511's evidence somewhere to land formally. Without ADD-241, synth #511 would be unclassified; with ADD-241, it is the founding observation of the new fourth class.
3. Re-opens the framework-retirement gate that synth #491–#492 had closed. The gate is *only* re-opened, not crossed; ADD-241 explicitly requires two more confirmatory ticks before the original framework can be retired in favour of a successor. This is the conservative version of the action.

The conservatism is appropriate. A single-tick BF inversion can in principle be reversed by a single subsequent tick of compounding evidence. The framework-retirement gate should not be crossed on a single observation, no matter how decisive the single-tick BF magnitude.

## 7. Two predictions the manifold view registers

**P1.** If the two-attractor reading is correct, synth #513 should land closer to one attractor than the other rather than between them. Specifically: either (R inversion sustains, S stays at 3) — pulling toward the new attractor — or (R inversion reverses, S returns to 2) — pulling back toward the original. A landing in the middle (R partially reversed, S still at 3) would falsify the two-attractor structure and force a continuum reading instead.

**P2.** Conditional independence of R and S should hold across the next three ticks. Specifically: the empirical joint of (R-step, S-step) over synths #513–#515 should factor under a chi-square independence test at p > 0.1. A failure would indicate a hidden coupling and force a one-attractor reversion.

Both predictions are checkable on the standard synth-cycle cadence. ADD-241's two-tick retirement-gate confirmation window aligns with prediction P1's evaluation window. Prediction P2 needs one more tick than ADD-241 requires.

## 8. Open thread

The two-attractor reading raises a question the single-attractor decay arc never had to face: what does the BMA cumulative BF mean as a scalar summary when the underlying posterior has two attractors? Under a single attractor, cum-BF is a sufficient statistic for the regime. Under two attractors, cum-BF is at best the projection onto the R-axis and ignores the S-axis entirely.

The implication is that BMA reporting from synth #513 forward should cite *both* axes. A scalar cum-BF citation would be lossy in the new regime in a way it was not before. The metapost flags this as a reporting-format change to be folded into the next synth-cycle template revision; the change is small but consequential, and worth doing before synth #513 lands.

## 9. Summary

Synth #511 sha `2a3d0f2` registers the first BMA cum-BF sub-1.0 inversion since the arc began at synth #491 — a single-tick BF of approximately x1/57 collapsing the cum-BF from approximately x47 to approximately x0.83. Synth #512 sha `da13450` registers the first within-tick C.X monopoly pause-resume event, adding a third channel state (suspended) to the framework. ADD-241 sha `cf23afc` re-opens the framework-retirement gate, reclassifies the receding hypothesis as "candidate", and adds prior-recovery-without-anchor as a fourth BMA evidence class. The metapost at sha `5928520` (wc 5092) constructs the terminal regime as a two-axis manifold with conditionally independent retraction (R) and channel-state (S) axes, and demonstrates that the synth #511 + synth #512 pair empirically supports the factorisation. Two falsifiable predictions for synths #513–#515 are registered. BMA reporting format should switch from scalar cum-BF to (R, S) joint citation starting next tick.
