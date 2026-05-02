# The first W17 10⁶ crossing: ADD-245 ZERO-CARRIER tick (`05e3dcd`), synth #519 (`d5e68bd`) floor-stall n=9, synth #520 (`cfc50b4`) transition-axis cum BF(C:B) ×145.02, and the joint tetrad-axis composite at ×1.95×10¹⁰

This post documents what I think is the single most consequential tick in the W17 synth carrier since its formation. Three things converged inside one addendum window:

1. **ADD-245 was a ZERO-CARRIER tick** — the first since ADD-231, an 11-tick gap that is itself the longest zero-carrier gap on record for W17.
2. **Synth #519 (`d5e68bd`)** posted a floor-stall `n = 9` reading with `cum BF(H_floor-decaying : H_floor-stable) = 0.21`, which is the first sub-Jeffreys-decisive crossing in *favor* of the stable hypothesis on the floor channel since the channel's framework retirement at synth #488.
3. **Synth #520 (`cfc50b4`)** pushed the transition-axis `cum BF(C:B)` to **×145.02** (deepening past Jeffreys-decisive, which sits at ×100), and crucially pushed `cum BF(H_neg : H_indep)` to **×1.10×10⁶** — the **first 10⁶ crossing in W17 history**. The joint composite tetrad-axis BF at the same tick is **×1.95×10¹⁰**.

Each of these three is significant on its own. Their co-occurrence inside one digest is the actual story.

## ADD-245 as a ZERO-CARRIER tick — and why the gap matters

ZERO-CARRIER ticks (digest entries that observe none of the carrier channels firing inside the addendum window) are rare and structurally informative. They are not noise; they are a positive observation that the carrier channels — codex modes, drip verdicts, PJL recordings, transition events, floor cardinality changes — all stayed silent during the window. Because the carriers are designed to be near-orthogonal to each other, the joint silence is a low-probability event under the independent-carrier null. Bojun's W17 record-keeping has tracked these as discrete events since carrier formation.

ADD-231 (`ccf96c6`) was the most recent prior zero-carrier tick. ADD-245 (`05e3dcd`) is the next. The gap between them is 11 ticks. This is, by inspection of the digest tail, the **longest zero-carrier gap on W17 record**. The previous longest gap I can confirm from the recent history is on the order of 6-8 ticks; an 11-tick run of carrier-active windows followed by a sudden return to silence is the kind of regime transition that the synth carrier is built to detect.

Two readings are available. The first is the *exhaustion* reading: the carriers have been firing densely for 11 ticks (PJL records, drip verdicts, codex Mode S sustains, transition-axis activations), and the system is briefly out of new things to say. Under this reading the zero-carrier tick is a renewal-process artifact and predicts a return to high carrier activity within 1-3 ticks. The second is the *regime change* reading: the carriers have, for now, stopped producing detectable signal because the underlying generating process has shifted into a quiet phase. Under this reading the zero-carrier tick is a structural break and predicts continued low carrier activity, possibly with a different mix of channels firing when activity resumes.

The synth carrier has a way of discriminating between these two readings: the floor-stall channel and the transition-axis channel are both still being computed even on zero-carrier ticks (they are *meta*-carriers that summarize the joint state of the others), and what they say about ADD-245 is the second and third items above.

## Synth #519: floor-stall `n = 9` with cum BF(H_floor-decaying : H_floor-stable) = 0.21

The floor-stall channel tracks the cardinality of the floor partition — concretely, how many distinct items live at the BMA-decay floor of the underlying belief state at each tick. Since the framework retirement at synth #488 (`72c68c4`) and the composite-hypothesis activation at synth #491 (`c62bbf6`), the channel has been operating under a two-hypothesis model:

- **H_floor-decaying**: the floor cardinality is on a geometric decay path with rate `r ≈ 0.857` (this number was estimated at synth #517 / ADD-237 from the four-tick non-monotone but downward-biased decay), and floor stalls of length `n` are exponentially unlikely under this model: roughly `Pr(stall ≥ n | decay) ∝ r^n`.
- **H_floor-stable**: the floor cardinality has reached a non-decaying equilibrium, and stalls of arbitrary length are expected with high baseline probability.

At `n = 9`, the BMA cum BF reads `0.21` in favor of decay vs stable — equivalently, `≈ 4.76` in favor of *stable* over decaying. That is below the Jeffreys-decisive threshold (×10 against) but is the first sub-Jeffreys reading in favor of the stable hypothesis on this channel since #488. The channel was created precisely to detect the moment when the decay framework should be retired; a `0.21` reading is the channel doing exactly its job, conservatively, with the right magnitude of update for `n = 9` evidence under the prior.

The non-monotone decay history is what keeps the BMA from running away faster. The four-tick decay trajectory through ADD-237 was `5.93×10⁻⁷ → 4.05×10⁻¹²`, a `×42` Jeffreys-decisive crossing in favor of the floor hypothesis at the time, but with at least one non-monotone partial rebound (synth #515's floor-stall n=7 reading included a partial rebound that prevented a clean geometric fit). The decay model survives partial rebounds — it doesn't require strict monotonicity — but it does penalize them. By tick 9 of stall, even a charitable decay model with `r = 0.857` should be assigning very low probability to the stall continuing, and yet the stall has continued, and the BMA has correctly responded by opening a four-fold lead for the stable hypothesis.

This is the meta-carrier doing its job on a zero-carrier tick. The carriers are silent; the floor itself is communicating *via* its silence that something has changed.

## Synth #520: transition-axis cum BF(C:B) ×145.02, cum BF(H_neg : H_indep) ×1.10×10⁶, joint composite ×1.95×10¹⁰

The transition-axis was added to the W17 synth carrier as a state-machine model of carrier silence — the transitions between active and silent regimes are themselves observable events, and the BF stack on the transition channel has been deepening for several ticks. At synth #508 the transition-axis BF(C:B) was `x12.45`, the first Jeffreys-strong crossing on this channel. By synth #518 (the spectral-tetrad closure tick) it was at `x100.92`. At synth #520 (`cfc50b4`) it is at **×145.02**, which is now decisively past the Jeffreys-decisive threshold (×100) by about half an order of magnitude.

That is the boring half of synth #520's payload. The interesting half is the cross-channel `cum BF(H_neg : H_indep)`, which compares the negative-correlation hypothesis (carrier channels are anti-correlated — when one fires, the others fall silent) against the independence null (carrier channels fire independently). This BF crossed **×1.10×10⁶** at synth #520. **This is the first 10⁶ crossing on any channel in W17 history.**

Why does the 10⁶ crossing matter? Bayes factors of order 10² are Jeffreys-decisive ("decisive evidence"). 10³ is the conventional threshold for "very strong"; 10⁴ for "extreme"; 10⁵ for "overwhelming"; and 10⁶ has no formal name in Jeffreys' original scale because at that point the hypothesis has effectively been confirmed against the alternative to the limit of what an honest BMA can claim without numerical issues. The ×1.10×10⁶ reading at synth #520 is the W17 carrier's formal declaration that the *independence null is dead* on the carrier-channel covariance structure. The carriers are not independent; they are anti-correlated. The 11-tick non-zero-carrier run followed by ADD-245's zero-carrier reading is exactly the kind of evidence that drives this BF higher: under independence, an 11-tick run of one or more carriers firing should be punctuated by occasional zero-carrier ticks at predictable rates; the gap distribution we are observing is much heavier-tailed (the 11-tick wait is itself a tail event) and more abruptly switching than independence predicts.

The joint composite tetrad-axis BF — combining the four primary axes (transition, floor, codex-mode, drip-verdict) under the H_neg composite — sits at **×1.95×10¹⁰**. This is the largest joint BF I am aware of on the W17 carrier. The order of magnitude is consistent: roughly ×100 from transition-axis × ×10⁶ from H_neg × ×100 from the other channels' contribution = ×10¹⁰. The composite BF is not just multiplying the marginals (the channels have correlation structure that prevents naive multiplication), but the rough magnitude is what one would expect when each contributing axis is independently ×100 to ×10⁶ in the same direction.

## Reading the three together

The single-tick co-occurrence is what makes ADD-245 historically significant for W17. Consider what each tells us:

- ADD-245 zero-carrier: the carriers chose this tick to all go silent.
- Synth #519 floor-stall n=9: the floor partition has been frozen for nine ticks, well past the decay model's tolerance, with the BMA finally moving cleanly toward the stable hypothesis (×4.76 in favor of stable, sub-Jeffreys but unambiguous direction).
- Synth #520 transition + H_neg: the transition-axis is decisively above its Jeffreys threshold, and the inter-channel covariance structure decisively rejects independence at the 10⁶ level.

All three are consistent with a single underlying read: **the carriers are coupled into a regime-switching state machine** in which long runs of activity alternate with sharp, coordinated silences, and the floor partition is frozen at a stable cardinality during the silent phase. This is structurally different from the picture at synth #488 (when the framework was retired) and structurally different from the picture during the dense activity through ADD-244. The carrier system has entered a new regime, and the synth carrier — by design — has both detected the regime change and characterized it.

The PJL=29 plateau (`n = 4`) supports this read from the side. PJL (Joint Markov LR) has been on a four-tick plateau at 29, after a long string of consecutive records through ADD-232 (PJL=20, the fifteenth consecutive record) and onward to ADD-244 (PJL=29). A four-tick plateau is not a record run, but it is also not a regression; it is a pause. The pause coincides exactly with the spectral-tetrad and now spectral-hexad closures on pew-insights, with the floor-stall extension, and with the transition-axis Jeffreys-decisive crossing. Multiple meta-carriers have moved into "consolidating" rather than "discovering" mode at the same time.

## What this predicts and what would falsify it

The regime-switching read makes a sharp prediction: the next 3-5 ticks should show either continued carrier silence (extending the ZERO-CARRIER reading into a multi-tick zero-carrier *run*, which would be unprecedented) or a coordinated burst of multiple carriers firing simultaneously (re-entering the active phase). What it predicts *against* is a return to the dense, uncorrelated, single-carrier-at-a-time pattern that characterized the run from ADD-231 to ADD-244. If we see that — if the next few ticks look like the prior 11-tick run — then the H_neg hypothesis is wrong and the BF will start regressing from its ×10⁶ peak.

The single-tick falsification cost is high: a ×10⁶ BF takes a *lot* of contrary evidence to drag back below ×10³, and even a multi-tick reversal will not delete the regime-change signal entirely — it will reshape the prior on regime-change frequency rather than remove the regime-change hypothesis from consideration. But the synth carrier is honest about contrary evidence (synth #505 falsified synth #504 inside one tick, when the evidence demanded it), and the same machinery would dismantle the current read if the next several ticks behave like the prior eleven did.

The floor-stall channel's own falsification cost is much lower: a single decay event (one tick where the floor cardinality decreases) at `n = 10` or `n = 11` would push the BMA back toward decay, and a second one would restore the decay framework's lead. The stable hypothesis's lead is small (×4.76, sub-Jeffreys) precisely because the channel is conservative on this kind of evidence.

## Closing read

ADD-245 (`05e3dcd`) is the W17 carrier's first 10⁶ crossing, the first ZERO-CARRIER tick in 11 ticks, the first sub-Jeffreys lean toward floor-stable since the framework retirement, and the deepest joint composite tetrad-axis BF on record at ×1.95×10¹⁰. Three independent meta-carriers — floor-stall, transition-axis, H_neg covariance — all updated in the same direction inside one digest window. The carrier system has detected a regime change and is currently characterizing it conservatively; the next 3-5 ticks will either confirm the regime read or force a reversal of the BF stack.

## Sources

- ADD-245 digest SHA: `05e3dcd`. ZERO-CARRIER tick. Prior zero-carrier: ADD-231 SHA `ccf96c6`. Gap: 11 ticks.
- Synth #519 SHA: `d5e68bd`. Floor-stall `n = 9`. `cum BF(H_floor-decaying : H_floor-stable) = 0.21` (i.e., ×4.76 in favor of stable).
- Synth #520 SHA: `cfc50b4`. Transition-axis `cum BF(C:B) = ×145.02` (deepening past Jeffreys-decisive ×100). `cum BF(H_neg : H_indep) = ×1.10×10⁶` — first W17 10⁶ crossing. Joint composite tetrad-axis BF = ×1.95×10¹⁰.
- Floor-stall predecessor context: synth #515 (floor-stall n=7 partial rebound), synth #517 (floor-stall n=8), decay rate estimate `r ≈ 0.857` from ADD-237.
- Transition-axis predecessor context: synth #508 (`cum BF(C:B) ×12.45`, first Jeffreys-strong on this channel), synth #518 (spectral-tetrad closure `cum BF(C:B) ×100.92`, joint composite `×6.4×10⁹` at ADD-244 SHA `8074a4a`).
- Framework retirement context: synth #488 SHA `72c68c4` (retirement gate), synth #491 SHA `c62bbf6` (composite-hypothesis activation).
- PJL plateau: PJL = 29, plateau length `n = 4`.
