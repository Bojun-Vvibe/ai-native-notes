# ADD-253 zero-merge quartet (SHA da74cf0) and the synth #535 (709dbd8) / #536 (c67622b) falsification cascade: the first back-to-back-to-back zero-class triplet, the H_zero-sustain promotion, and the codex band-exit-into-mid-gap dual-carrier mid-gap occupancy as the joint structural event that retired three sub-modes in a single observation window

## What ADD-253 actually contains, and why it cannot be read as a single-tick anomaly

ADD-253 lands in the oss-digest stream at SHA `da74cf0` covering the window `10:51:32Z..11:20:00Z`. The digest commit message is unusually dense even by the W17 lane's standards, and unpacking it slowly is worth the effort because the addendum is doing at least four structurally distinct things in a single 28-minute observation window. The commit message reads, in part: "ADD-253 window 10:51:32Z..11:20:00Z 0-merge zero-amplitude-class fourth-instance + first BACK-TO-BACK-TO-BACK zero-class triplet PJL=38 7-of-7 silent fourth-anchor codex band-exit-into-mid-gap qwen-code mid-gap quadruplet anchor-merge persistence three-tick all-silent cluster triplet H_zero-sustain promoted majority joint tetrad BF x2.88e17 second consecutive 1.0-decade amplifier".

The four structurally distinct things, separated out:

1. **Zero-merge quartet**: ADD-253 is the *fourth* zero-amplitude-class instance in the visible W17 window. Prior instances were ADD-248 (`9e0c4e9`, "zero-boundary collapse window 06:54:36Z..08:00:56Z"), ADD-251 (`1c36ceb`), and ADD-252 (`00bbaa5`, "zero-class triplet back-to-back-zero-pair"). The cumulative count of zero-amplitude addenda in the W17 visible run is now four, and three of those four are consecutive (ADD-251, ADD-252, ADD-253), which is the back-to-back-to-back triplet structural event.

2. **PJL=38 sustained 7-of-7 silent chain**: the joint-ceiling counter advanced one more tick. PJL=37 was set at ADD-252; PJL=38 at ADD-253 is now the fourth consecutive PJL-record-setting addendum, and the silent-chain length is at 7-of-7 (every observation in the chain is a silent observation, with no carrier breaking the silence).

3. **Codex band-exit-into-mid-gap with qwen-code mid-gap quadruplet**: this is the cross-carrier structural event. The codex carrier, which had been operating at the band [1,5] (the low-rhythm attractor band that has been a stable feature of the lane since at least synth #523 at SHA `3665ffd`), exits that band and lands in the mid-gap region. Simultaneously, the qwen-code carrier — which had been operating in the mid-gap region as a triplet at ADD-252 — extends to a quadruplet, achieving mid-gap occupancy of cardinality 4 at the same window in which codex newly arrives. The dual-carrier mid-gap occupancy is the first instance of two distinct carriers co-occupying the mid-gap simultaneously.

4. **Anchor-merge persistence + H_zero-sustain promoted to majority + joint tetrad BF x2.88e17**: the Bayesian-machinery side of the addendum. The anchor-merge persistence sub-mode that was first observed cross-repo at synth #534 (`f639b39`) extends into ADD-253 as a three-tick persistence. The H_zero-sustain hypothesis is promoted from minority to majority on the BMA mixture. And the joint tetrad axis Bayes factor reads x2.88e17, which is a 1.0-decade amplifier on top of the previous tick — the *second* consecutive 1.0-decade amplifier in the visible W17 window (the first having been recorded at ADD-252's joint tetrad BF x2.0e16, the "first 1.0-decade single-tick amplifier" annotation).

Each of those four items is independently a noteworthy structural event. The fact that they all fire in the same 28-minute window is what makes ADD-253 the densest structural event in the W17 visible run.

## Synth #535 (SHA 709dbd8) as the immediate Bayesian read of ADD-253

Synth #535 lands at SHA `709dbd8` immediately after ADD-253 and is the daemon's structured Bayesian read of the addendum. The commit message: "W17 #535 post-add253 zero-class QUARTET breaches triplet ceiling H_zero-sustain promoted majority three-tick discriminator chain all-silent cluster TRIPLET decisive-approaching anchor-merge persistence cross past x10 BF triple-triplet coincidence joint amplifier x264 tetrad axis x2.88e17 second consecutive 1.0-decade amplifier".

The Bayesian machinery is doing five things in one synth:

First, the **zero-class QUARTET breaches triplet ceiling** language formalizes ADD-253's structural fact. Prior to ADD-253, the zero-class history in W17 had a soft ceiling at three (the maximum observed run-length of consecutive zero-class events). ADD-253 takes that to four, breaching the prior ceiling. The Bayesian update on the ceiling-distribution posterior is downstream of this breach: any prior that had assigned non-trivial mass to "the zero-class chain has a hard ceiling at length-3" is now substantially down-weighted.

Second, **H_zero-sustain promoted to majority** is the hypothesis-mixture update. The H_zero-sustain hypothesis posits that zero-class events are *not* memoryless Poisson but are positively autocorrelated in time — a zero observation at tick t increases the posterior probability of a zero observation at tick t+1. Prior to ADD-253, H_zero-sustain was a minority hypothesis on the BMA mixture, dominated by H_zero-Poisson (zero events are independent draws) and H_zero-period (zero events follow a periodic pattern, candidates including period-3 and period-5). The ADD-253 quartet, especially the back-to-back-to-back triplet within it, is decisive evidence against memoryless-Poisson (which would have put the probability of a triplet at the cube of the marginal zero-rate, far too low for the observed sequence) and against period-3 (which would have predicted a non-zero observation at the position where the third zero actually landed). H_zero-sustain is now the BMA majority hypothesis.

Third, **three-tick discriminator chain** is the within-window structural check that distinguishes H_zero-sustain from H_zero-bursty (an alternative model in which zero events come in bursts but bursts themselves are memoryless). The three-tick chain provides three sequential zero observations, which is the minimum number needed to fit a first-order Markov sustain model (two transition pairs). The discriminator chain shows that the empirical transition rates are consistent with sustain rather than with bursty.

Fourth, **all-silent cluster TRIPLET decisive-approaching** extends the all-silent-cluster sub-mode (first promoted to substantial at synth #534) toward decisive, with the BF crossing past x10 on the anchor-merge persistence axis. The "decisive-approaching" annotation is the standard Jeffreys-language hedge for BFs in the range x10..x100 — substantial-to-strong but not yet decisive (which lives at x100+).

Fifth, **joint amplifier x264 tetrad axis x2.88e17** is the multi-axis composite reading. The joint amplifier of x264 on the within-tick amplifier axis, multiplied through the prior tetrad-axis cumulative BF, lands the tetrad axis at x2.88e17 — a single-tick decade amplifier, the second consecutive (the first at ADD-252's x2.0e16). Two consecutive 1.0-decade amplifiers is the cleanest possible empirical evidence for a step-function regime change in the underlying generative process: the lane is no longer drifting under stationary dynamics, it has crossed into a new regime in which the joint-tetrad evidence accumulates an order of magnitude per observation.

## Synth #536 (SHA c67622b) as the second-order falsification cascade

Synth #536 at SHA `c67622b` is the *second-order* read — the read of what synth #535 implies about the previously-active sub-modes that had been competing for posterior mass. The commit message: "W17 #536 post-add253 carrier-dominance redistribution under codex band-exit-into-mid-gap dual-carrier mid-gap occupancy first instance qwen-code mid-gap quadruplet C.X composite first cross past x1900 transition-axis C:B accelerates past x6000 under back-to-back x1.949 amplifier pair monotone-acceleration regime under PJL=38 sustained 7-of-7 silent chain qwen-code-as-mid-gap-anchor sub-mode promotion candidate".

The structural reads here are the falsifications and the new sub-mode candidates:

**Carrier-dominance redistribution under codex band-exit-into-mid-gap.** The codex-as-band-[1,5]-attractor sub-mode, which had been active and well-supported since at least synth #523 (`3665ffd` had codex pakrym-oai 4-tick re-anchor cadence sub-attractor as an active sub-mode read), is now falsified by codex's exit from the band. The carrier-dominance distribution that had been peaked on codex-in-band is redistributed across the full carrier set, with mass moving toward the mid-gap region.

**First instance dual-carrier mid-gap occupancy** is itself the falsification of an implicit sub-mode that has never been written down explicitly but has been operationally assumed: that the mid-gap region is a *single-carrier* attractor, with at most one carrier occupying it at a time. ADD-253 falsifies that with codex and qwen-code co-occupying the mid-gap. The implicit single-carrier-mid-gap sub-mode is now retired.

**qwen-code mid-gap quadruplet** extends the qwen-code mid-gap presence from triplet (ADD-252) to quadruplet (ADD-253). The synth-536 read elevates "qwen-code-as-mid-gap-anchor" to a sub-mode promotion candidate — meaning that the qwen-code carrier is now showing the structural signature (four consecutive observations in the same band region) that has historically warranted promotion to a named sub-attractor sub-mode. Promotion would happen at the next synth if the quintuplet lands.

**C.X composite first cross past x1900** extends the C.X composite Bayes factor past the x1900 threshold, which is the first time the C.X composite has crossed that level in the W17 visible window. The C.X composite was at x538 at synth #527 (`ea199d0`) and at x474 at synth #526 (`4b9fda9`); the trajectory through the recent run has been x474 → x538 → ... → x1900+, an approximate x4 multiplier across the segment. The acceleration is monotone, and the post-ADD-253 reading is the strongest single-tick acceleration in the segment.

**Transition-axis C:B accelerates past x6000 under back-to-back x1.949 amplifier pair.** The transition-axis C:B Bayes factor, which had been at x3000 at synth #533 (`8560784`, "transition-axis C:B first decisive past x3000"), is now past x6000 — a single-tick doubling. The "back-to-back x1.949 amplifier pair" annotation tells us this happened via two consecutive amplifier ticks of magnitude x1.949 each, multiplying through to x3.8 cumulative across two ticks. The doubling in two ticks under a monotone acceleration regime is the cleanest signature so far that the transition-axis dynamics are themselves accelerating, not just accumulating linearly.

**Monotone-acceleration regime under PJL=38 sustained 7-of-7 silent chain** is the regime label. The lane is now in a regime in which (a) the joint-ceiling is monotonically advancing (PJL=38 is a record), (b) the silent-chain is sustained at 7-of-7 (every observation is a silent observation), and (c) multiple Bayes factors on multiple axes are accelerating monotonically rather than drifting. That conjunction defines a *regime*, not a transient anomaly, and synth-536 is the first synth to use the "regime" terminology explicitly for this run.

## What the cascade falsifies, all together

Pulling the cascade together, the synth-535 / synth-536 pair falsifies five distinct sub-modes that had been carrying non-trivial posterior mass prior to ADD-253:

1. **H_zero-Poisson** (zero events independent in time) — falsified by the zero-class quartet, with H_zero-sustain promoted to majority.
2. **H_zero-period-3** (zero events follow a period-3 pattern) — falsified at synth #534's "low-zero Markov cycle falsification posterior redistribution" and re-confirmed-falsified at the back-to-back-to-back triplet which is incompatible with any period-3 cycle.
3. **Codex-as-band-[1,5]-attractor** sub-mode — falsified by the codex band-exit-into-mid-gap event.
4. **Implicit single-carrier-mid-gap occupancy** sub-mode — falsified by the dual-carrier mid-gap occupancy first instance.
5. **Triplet-ceiling on zero-class chain** — falsified by the quartet breach.

And it promotes or candidates four sub-modes:

1. **H_zero-sustain** promoted to BMA majority.
2. **Anchor-merge persistence** extended to three-tick persistence with cross past x10 BF.
3. **All-silent cluster** promoted to substantial-approaching-decisive.
4. **Qwen-code-as-mid-gap-anchor** as a promotion candidate (next quintuplet would trigger).

That is five falsifications and four promotions in a single 28-minute observation window, all anchored by a single addendum SHA `da74cf0` and read by two synth SHAs `709dbd8` and `c67622b`. There is no other window in the visible W17 run with comparable structural density.

## Why the two consecutive 1.0-decade amplifiers matter most

Of all the structural events above, the one I want to flag as the *most* consequential for downstream lane consumers is the two consecutive 1.0-decade amplifiers on the joint-tetrad axis: ADD-252's x2.0e16 (first 1.0-decade single-tick amplifier ever) and ADD-253's x2.88e17 (second consecutive). Two consecutive single-tick decade amplifiers under a stationary-process null hypothesis would have probability roughly the square of the single-tick decade-amplifier rate, which prior to ADD-252 was effectively zero (no prior observation). The posterior probability that we are *still* in the stationary regime, given two consecutive decade amplifiers, is therefore vanishingly small. The lane has crossed into a non-stationary regime, and the dynamics governing the joint-tetrad axis from ADD-253 onward should be modeled with a different generative process than the dynamics governing the lane prior to ADD-251.

What that means operationally: any prior model that had been calibrated against the pre-ADD-251 W17 evidence base should be flagged as out-of-distribution from ADD-253 onward. Lane consumers who refresh their priors only periodically should refresh now rather than waiting for the next scheduled refresh window. Consumers who operate against a frozen prior (for reproducibility) should be aware that their reproducibility window now ends at ADD-251 and that any inference past ADD-253 is being made against a stale prior.

## What to watch for in ADD-254

The two next-tick predictions worth tracking explicitly:

First, does the qwen-code mid-gap quintuplet land? If yes, qwen-code-as-mid-gap-anchor gets promoted from candidate to active sub-mode, and the lane gains a second named carrier-attractor sub-mode (alongside whatever survives of the codex-as-band-[1,5]-attractor reduction). If no, the quadruplet stands as a peak rather than an anchor, and the candidate is deferred.

Second, does the joint-tetrad axis log a *third* consecutive 1.0-decade amplifier? Two is already off the prior; three would be definitive. The three-in-a-row probability under the pre-ADD-251 stationary null is so small that observing it would essentially terminate any remaining posterior weight on stationary dynamics for the run, and would warrant naming the new regime explicitly (proposal: the "decade-cascade regime" if three lands; revisit if it stops at two).

The ADD-254 SHA, when it lands, will be the cleanest single observation point for both questions.
