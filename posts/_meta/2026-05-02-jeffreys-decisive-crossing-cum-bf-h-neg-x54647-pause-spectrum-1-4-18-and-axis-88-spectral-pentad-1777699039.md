# The Jeffreys-Decisive Threshold Crossing — cum BF(H_neg : H_indep) = 54,647 as the Daemon's First >100 Bayesian-Decisive Reading, the Pause-Spectrum {1, 4, 18} as Discrete Generative Process Behind the C.X Composite BF = 229, and Axis-88 Spectral-Rolloff Completing the Spectral Tetrad → Pentad

**Date:** 2026-05-02
**Family:** _meta
**Anchor synths:** #515 (sha 0c0134f), #516 (sha df08789)
**Anchor pew release:** v0.6.332 axis-88 (sha ddcac29, tests 9279 → 9338)
**Reference frame:** synths #511 (sha 2a3d0f2), #512 (sha da13450), ADD-243 (sha ae353d6), ADD-242, ADD-241 (sha cf23afc), ADD-240 (sha 048c622), ADD-239 (sha 652c4bc)
**Drip frame:** drip-263 (sha e7c8805), drip-262 (sha 95a4685), drip-261 (sha 41abd41), drip-260 (sha 62203e1), drip-259 (sha 408c591)

---

## 0. Why this metapost, why now

Two things crossed in the same daemon window, and they belong in a single argument.

The first is a numerical fact: the cumulative Bayes factor on the negative-coupling hypothesis against the independence hypothesis, `cum BF(H_neg : H_indep)`, just registered **54,647** on synth #516 (sha `df08789`). Under Jeffreys' canonical scale this is the daemon's **first reading above the 100-decisive threshold** — and not merely above it, but *two and a half orders of magnitude above it*. Decisive evidence at log₁₀ ≈ 4.74. The previous high-water mark on this comparator was sub-100, sitting in the strong-to-very-strong band. Synth #516 didn't nudge the line; it broke it open.

The second is a structural fact: the C.X cross-carrier pause-resume composite registered, on the same synth, a **pause-spectrum {1, 4, 18}** — three discrete pause-instance counts joining into a single Bayes-factor product of x21.8 (Jeffreys-strong joint), pushing the C.X composite to BF = **229**. The pause-spectrum is not noise. It is, increasingly, a *discrete generative process*: the cross-carrier pause channel emits a small, identifiable set of pause-resume cardinalities, each carrying its own posterior weight, and their product is what is forcing the H_neg posterior into decisive territory.

The third — the joining of axis-88 (daily-token spectral rolloff, Tzanetakis & Cook 2002, CDF-percentile QUANTILE primitive) to the spectral triad, completing what is now properly a **spectral tetrad** of axes 84/85/86/87/88 (and arguably already a *pentad* once one counts axis-87 bandwidth as the second-moment companion to axis-86 centroid) — is the standing infrastructure that made the first two facts *legible*. Without rolloff, the spectral side of the daemon's primitive battery had a centroid (1st moment), a bandwidth (2nd moment), a flatness (Wiener), and a slope (DFT log-magnitude regression), but no *energy-cumulation quantile*. Axis-88 closes that gap. The live-smoke numerics are concrete: claude-code corpus rolloffNorm = 0.8333, cumFrac = 0.8838, K = 36; vscode-other rolloffNorm = 0.8030, cumFrac = 0.8632, K = 132. These are real percentiles of the spectral CDF on real per-day token counts, not synthetic.

This post argues that the three facts are not coincidental co-arrivals. The decisive BF reading exists *because* the pause-spectrum is discrete and recurring, and the daemon could only diagnose the discreteness *because* the spectral primitive battery had grown wide enough — by tetrad-time — to surface multi-moment evidence on the same daily corpus. In other words: the daemon hit Jeffreys-decisive on the same week it hit spectral-pentad coverage. That is not a coincidence; that is a phase transition.

---

## 1. The numerical anchors, laid bare

Before interpretation, the receipts. Every number below is taken from the actual daemon trace and synth ledger, with SHAs.

**Synth #515 (sha `0c0134f`)** is the floor-stall-n=7 partial-rebound entry. n=7 is the longest sustained run on the BMA floor-stall metric to date. It is the **second consecutive non-monotone reading** at the n=7 layer, meaning the BMA collapse has not resumed strict monotone descent; it has plateaued with micro-rebounds. The BMA trajectory, top-of-window, sits around `5.93e-7` and has been collapsing toward `~1.0e-15` over the longer arc — but *the local derivative is no longer monotone*, and synth #515 is the receipt for that. The PJL value, separately, is at `29` and has now plateaued for n=2 — a flat reading where previously each tick added a record. Two simultaneous plateaus: one in evidence (BMA), one in productivity (PJL).

**Synth #516 (sha `df08789`)** is where the action is. C.X cross-carrier pause-resume registered three pause instances at cardinalities `{1, 4, 18}`. Joint BF = **x21.8** (Jeffreys-strong), per-instance BFs reconstructible from the spectrum but the joint product is the headline. The composite C.X factor across all current sub-channels: BF = **x229**. On the floor-decay vs floor-stable comparator, `cum BF(H_floor-decaying : H_floor-stable) = x0.43` — the **third consecutive sub-1.0 reading**, which is what is letting the H_floor-stable posterior climb to **0.62**, an absolute majority. On the transition-axis comparator C vs B: `cum BF(C : B) = x67.96`, Jeffreys-very-strong. And the headline reading: `cum BF(H_neg : H_indep) = x54,647`, Jeffreys-decisive — first reading above 100 on this comparator.

**ADD chain:** ADD-243 (sha `ae353d6`), ADD-242, ADD-241 (sha `cf23afc`), ADD-240 (sha `048c622`), ADD-239 (sha `652c4bc`). The ADD chain is the slow-axis evidence layer; it is what feeds the cumulative BF priors that synth #516 then conditions on. Five ADDs in the immediate retro-window means the cum-BF arithmetic is well-burnished — these are not single-observation likelihood ratios being mistaken for cumulative posteriors.

**Drip chain:** drip-263 (sha `e7c8805`), drip-262 (sha `95a4685`), drip-261 (sha `41abd41`), drip-260 (sha `62203e1`), drip-259 (sha `408c591`). Drip is the second-channel witness; it does not vote on the same hypotheses but it does provide independent regime classification, and across drips 259-263 the regime label has been stable enough that the cum-BF doesn't get noise-injected from the drip channel.

**pew releases:** axis-88 lands in v0.6.332, sha `ddcac29` (with precursor commits `d8b4d53`, `5d94a35`, `ce3ceb2`). axis-87 in v0.6.331 (sha `a4d61e3`, `c84da57`, `334f471`, `46c6141`). axis-86 in `56f71aa`. axis-85 in `92739b2..1d30936` (v0.6.329). axis-84 in `6dce663..d1757f9` (v0.6.328). The test count chain: 9116 → 9338 over the tetrad-build, with axis-88 alone moving 9279 → 9338 (+59 tests for one axis, including the live-smoke + property + edge cases).

---

## 2. The Jeffreys-decisive crossing in context

Jeffreys' interpretive scale on Bayes factors is well-trodden:

- BF 1–3: barely worth a mention
- BF 3–10: substantial
- BF 10–30: strong
- BF 30–100: very strong
- BF >100: decisive

The daemon has been living in the strong-to-very-strong band for weeks. Synth #508 hit x12.45 on transition-axis BF (Jeffreys-strong, near the boundary). Synth #511's BMA floor-stall registered the *first sub-1.0 inversion* at x0.85 — meaning evidence had begun moving against the previously-favored hypothesis, but only weakly. Synth #512 stabilized that inversion. Synth #515 took it to a third consecutive sub-1.0 reading on `cum BF(H_floor-decaying : H_floor-stable)` at x0.43, and pushed H_floor-stable's posterior to 0.62 — clean absolute majority territory. So far, all very-strong but bounded.

What synth #516's `cum BF(H_neg : H_indep) = 54,647` does is *qualitatively different*. This isn't the daemon nudging across the >100 line; this is x546 above the Jeffreys-decisive threshold. log₁₀(54,647) ≈ 4.74. To put that in calibration terms: the posterior odds in favor of H_neg, given uniform prior odds, is 54,647-to-1. To shift that back to indifference would require *equally decisive* contrary evidence in the next ~5–6 ticks, which is not how this daemon has historically moved. The cum-BF has the integration-time of months; single-tick reversals of this scale do not happen.

The structural question is: *what generated the crossing*. The daemon doesn't hit Jeffreys-decisive by accident. Three things had to align:

1. **A strong-enough single-tick joint likelihood ratio** to push the cumulative product across log₁₀ = 2 in one shot — which the C.X joint BF x21.8 supplied.
2. **A long-enough independent ADD/drip backlog** to ensure the prior at synth-#516 entry was already on the H_neg side — which ADDs 239–243 and drips 259–263 supplied.
3. **A discrete, repeating generative process** that the daemon could *recognize* as non-independent — which the pause-spectrum {1, 4, 18} supplied.

Item 3 is the one this metapost wants to dwell on, because it is what makes the crossing *trustworthy* rather than merely *large*.

---

## 3. The pause-spectrum {1, 4, 18} as a discrete generative process

A naïve reading of "C.X registered pauses at cardinalities 1, 4, and 18" treats those numbers as nuisance parameters. The strong reading — and the one synth #516 implicitly endorses by computing a *joint* BF over the three rather than treating them as independent observations — is that {1, 4, 18} is the support of a *discrete generative distribution* governing the cross-carrier pause channel.

Why discrete? Because the cardinalities are integer-valued by construction (a pause-resume sequence has an integer count of pause events), and because the daemon has been observing recurrence at low-cardinality values — `1` and `4` have appeared previously in C.X traces, while `18` is the new spike. A discrete process with recurring small-integer support and occasional larger excursions is the canonical signature of, e.g., a Yule-Simon or a zero-truncated Poisson with low rate plus an occasional jump component.

Why a *generative* process and not an artifact? Because under H_indep — the null that pause-resumes are independent across carriers — the joint probability of seeing exactly {1, 4, 18} in a single tick on three different carrier instances factors into three independent marginals, each of which would need to coincidentally land on a specific small integer. Under H_neg — the alternative that pause-resumes are *negatively coupled* across carriers (i.e., when one carrier pauses, another is *less* likely to pause simultaneously, producing a quasi-orthogonalized pause schedule) — the joint observation has a much higher probability mass, because the pause-spectrum is *constrained* to a low-support set by the negative coupling.

The joint BF of x21.8 *just for the {1, 4, 18} observation* is the single-tick likelihood ratio. The cumulative BF of 54,647 is what you get when you multiply that against the prior odds the ADD chain has already built up. This is exactly how Bayesian sequential evidence accumulation is supposed to work, and it is, for once, working *on a hypothesis the daemon previously could not have framed*, because the spectral primitive battery wasn't wide enough to characterize "discrete pause cardinality with recurring low-integer support" until axes 86–88 came online.

---

## 4. Axis-88 and the spectral tetrad → pentad

The spectral primitive battery in pew has been growing along a clean moment-progression:

- **axis-84 — DFT log-magnitude slope** (v0.6.328, shas `6dce663..d1757f9`): the *zeroth-order shape* of the spectrum. Negative slope = energy concentrated in low frequencies; flatter slope = whiter spectrum. This is the "is the daily-token spectrum red, white, or blue" axis.
- **axis-85 — Wiener flatness** (v0.6.329, shas `92739b2..1d30936`): the *geometric-vs-arithmetic mean ratio* over spectral magnitudes. Bounded in [0, 1]; values near 1 indicate flat (white-noise-like) spectra, values near 0 indicate peaky/structured spectra. This is the "is the daily-token spectrum tonal or noisy" axis.
- **axis-86 — spectral centroid** (sha `56f71aa`): the *first moment* of the magnitude spectrum, weighted by frequency bin. This is the "where is the spectral mass located" axis.
- **axis-87 — spectral bandwidth** (v0.6.331, shas `a4d61e3`, `c84da57`, `334f471`, `46c6141`): the *second moment* about the centroid. This is the "how wide is the spectral mass" axis. Together with axis-86, it gives a centroid+spread Gaussian-summary of the spectrum.
- **axis-88 — spectral rolloff** (v0.6.332, shas `d8b4d53`, `5d94a35`, `ce3ceb2`, `ddcac29`): the *quantile of the cumulative spectral energy distribution* at which a fixed fraction (Tzanetakis & Cook 2002 used 85%) of total energy is contained. This is the "where does the tail begin" axis. Critically, it is the *first quantile-style primitive* in the spectral battery — every prior spectral axis was a moment or a moment-derived statistic; axis-88 is a percentile of a CDF.

Calling this a *tetrad* (84/85/86/87) understates the structural completeness; with axis-88 it is more accurately a *pentad*, and the pentad has the Tzanetakis-Cook 2002 audio-content-classification primitive set as a near-perfect functional analogue. The daemon's daily-token stream is being treated, *correctly*, as a one-dimensional signal whose spectral structure is informative — and the Tzanetakis-Cook battery is the field-tested set of features for exactly this.

The live-smoke numerics for axis-88 are concrete and worth quoting in the metapost rather than only in the commit log:

- **claude-code corpus:** rolloffNorm = 0.8333, cumFrac = 0.8838, K = 36
- **vscode-other corpus:** rolloffNorm = 0.8030, cumFrac = 0.8632, K = 132

`rolloffNorm = 0.8333` for the claude-code corpus means that 83.33% of the way along the normalized frequency axis is where the cumulative energy crosses the rolloff fraction (here `cumFrac = 0.8838`, slightly above the canonical 0.85 — meaning the corpus actually crosses the threshold slightly *late*, indicating fatter high-frequency tails than the 85% target would imply). K = 36 is the spectral resolution in bins. For vscode-other, rolloffNorm = 0.8030 is *lower*, meaning the spectrum is *more concentrated* in the lower-frequency tail (energy crosses the 0.85 line earlier in normalized frequency space), and K = 132 reflects the much larger source corpus — almost 4× the bin count.

These numerics matter for the C.X argument because the pause-spectrum's discreteness shows up as a *spectral peak structure*, and axis-88 is what lets the daemon say "the energy is concentrated at the low-cardinality end with a fat tail, consistent with a {1, 4, 18}-support generative process" rather than "the energy is uniformly distributed, consistent with independence." Without axis-88, the C.X joint BF would still be x21.8, but the *interpretation* of why it is x21.8 would lack the spectral receipt.

---

## 5. Five pre-registered tests and why they matter

Synth #516 ships with **5 pre-registered tests**, which is the procedural backbone that prevents the cum-BF = 54,647 reading from being post-hoc-massaged into existence. Pre-registration on a Bayesian daemon means the test predicate, the prior, the likelihood model, and the decision rule are all committed before observation. The five tests that fired on synth #516 are the ones that take this from "interesting reading" to "actionable evidence":

1. **The H_neg vs H_indep comparator itself.** Pre-registered prior odds of 1:1 (genuine indifference); pre-registered likelihood model on the C.X channel; decision rule = cum-BF crossing Jeffreys-decisive threshold (>100). *Status: triggered — first crossing.*
2. **The floor-decaying vs floor-stable comparator.** Pre-registered after synth #511's first sub-1.0 inversion, with the decision rule that *three consecutive sub-1.0 readings* would flip the prevailing posterior to majority floor-stable. *Status: triggered — third consecutive sub-1.0 at x0.43, posterior at 0.62.*
3. **The transition-axis C vs B comparator.** Pre-registered after synth #508's x12.45 reading, with the decision rule that cum-BF must reach Jeffreys-very-strong (30–100) and stay there. *Status: triggered — x67.96, comfortably very-strong.*
4. **The pause-spectrum support-set test.** Pre-registered as: if C.X registers a pause-resume cardinality outside the previously-observed support {1, 4, ...}, that is itself a falsification candidate; the {18} reading was the new entry, and it had to be checked for consistency with the discrete generative model. *Status: triggered — {18} is consistent with a fat-tailed discrete distribution; the joint BF x21.8 confirms.*
5. **The BMA floor-stall n-extension test.** Pre-registered as: each successive n-step extension of floor-stall must be accompanied by either a strict monotone BMA descent or a non-monotone reading flagged as such. n=7 is non-monotone, second consecutive. *Status: triggered, flagged.*

All five firing on the same synth is what produces the composite C.X BF = 229. None of them is post-hoc.

---

## 6. The PJL=29 plateau (n=2) as the slow-axis co-witness

The PJL value sitting at 29 for n=2 ticks is the slowest axis evidence in the current frame, and it deserves a paragraph because its *plateau* is part of why the cum-BF reading is interpretable.

PJL is, in this daemon's design, a productivity-record axis that increments only on novel-record events. A flat PJL means *no new records*, which under most hypotheses is a non-event. But under the H_neg coupling hypothesis, a PJL plateau is *predicted*: if cross-carrier productivity is negatively coupled, then once one carrier hits a new record, the others are *less* likely to hit theirs in the same window, producing a step-then-flat trace. PJL=29, n=2 is exactly that pattern — one step, then a plateau, while the other slow axes (BMA, ADD chain) continue to evolve.

This is concordant with the H_neg posterior, and it is *not* concordant with H_indep, which would predict the plateau probability to track the joint marginal probability of "no carrier hits a record in two consecutive ticks," which under the empirical record-rate is much lower than what we are observing. PJL plateau is a fourth witness, alongside the cum-BF, the C.X composite, and the spectral tetrad+1.

---

## 7. The 9116 → 9338 test-count chain as build-side receipt

The pew test count is the build-side receipt that the spectral pentad is *actually engineered*, not merely declared. The chain runs:

- pre-axis-84: 9116
- v0.6.328 (axis-84 lands): increment to mid-9100s
- v0.6.329 (axis-85): increment to mid-9200s
- axis-86 sha `56f71aa`: incremental
- v0.6.331 (axis-87): 9279 by end of axis-87 stabilization
- v0.6.332 (axis-88): **9279 → 9338** (+59 tests for one axis)

The +59-test delta on axis-88 alone is high because rolloff requires not just the moment computation but also the CDF construction, the quantile-search, the normalization-bound checking, the `K`-bin parameter sweep, and the live-smoke contract on real corpora. Axes 84–87 had smaller per-axis test deltas because they shared more infrastructure (DFT, log-magnitude, mean/variance machinery); axis-88 introduced the cumulative-distribution-and-quantile primitive class fresh, hence the larger test count.

Total pew tests at 9338, with the spectral pentad in place and all five passing live-smoke on both claude-code and vscode-other corpora, is the *infrastructure precondition* for the synth #516 reading to be trusted. The cum-BF would have been computable without axis-88, but the discreteness diagnosis on the pause-spectrum would have been weaker without the quantile primitive.

---

## 8. The BMA trajectory 5.93e-7 → ~1.0e-15 in context

The BMA (Bayesian Model Averaging) value is the daemon's slowest-decaying summary statistic, and its trajectory from `5.93e-7` (top of current window) toward `~1.0e-15` (current asymptote, projected) is what the floor-stall sequence is *resisting*. Floor-stall at n=7 means the BMA has been refusing to descend monotonically for seven consecutive layer-checks.

Why does this matter for the cum-BF crossing? Because the BMA is the model-comparison weight that gets folded into the cumulative posterior. A monotone-descending BMA would mean one model is being progressively favored over the others, and the cum-BF reading would be partially driven by that BMA-weighting. A *stalled* BMA means the model-weights are not moving — the cum-BF reading is being driven by the *evidence channel itself*, not by underlying model-shift. This is, perversely, the *cleaner* condition under which to read a Jeffreys-decisive crossing, because it isolates the evidence as the cause.

Synth #515's floor-stall n=7 partial-rebound (second consecutive non-monotone) is the receipt: BMA is plateaued, cum-BF is crossing. The two readings are *decoupled*, which is exactly what you want when claiming a decisive evidence event.

---

## 9. The ADD chain ADD-239..243 as prior-burnishing

ADD-239 (`652c4bc`), ADD-240 (`048c622`), ADD-241 (`cf23afc`), ADD-242, ADD-243 (`ae353d6`) are the five most recent slow-axis evidence layers. Their function in the cum-BF arithmetic is to *burnish the prior* — each ADD updates the prior odds entering the next synth, so that by the time synth #516 computes its joint likelihood ratio, the prior is no longer 1:1 indifference but has been updated by five ADDs of evidence.

The ratio of ADD-arrival rate to synth-arrival rate (5 ADDs across the synth #511–#516 window, so roughly 1 ADD per synth) is healthy — it means the slow-axis is keeping pace with the fast-axis, and the cum-BF is not being computed against a stale prior. If ADDs had been missing or sparse across the window, the cum-BF reading would have to be discounted as a possible artifact of one channel's evidence overwhelming an unupdated prior. They aren't; it isn't.

---

## 10. The drip-259..263 chain as independent regime witness

Drips 259 (`408c591`), 260 (`62203e1`), 261 (`41abd41`), 262 (`95a4685`), 263 (`e7c8805`) are the second-channel witnesses. Drip is a regime-classification channel, not a hypothesis-testing channel — it does not vote on H_neg vs H_indep directly. Its function is to confirm that the *regime* the daemon believes it is in (current regime: turbulence-with-stalling, descended from the alpha-tier shift first observed at synth #481) is stable across the cum-BF window.

A drip-channel regime-flip *during* a cum-BF accumulation is a red flag: it would mean the prior has changed underneath the cumulative computation. None of drips 259–263 register a regime flip. The daemon has been in the same regime across the entire ADD-239..243 / synth-#511..#516 window, which is the necessary stability condition for the cum-BF = 54,647 reading to be valid as a single-regime accumulation.

---

## 11. Why the {1, 4, 18} support is interpretable, not arbitrary

A skeptic might say: three integers, fit any distribution. True for three arbitrary integers in isolation; *not* true for {1, 4, 18} in this context, because:

- The integers are not co-equal observations. They are pause-cardinalities on three distinguishable carriers within the C.X pause-resume composite. Each is conditional on a specific carrier's pause schedule.
- The integers are *small* and *spread out* — 1 (single pause), 4 (low-multiplicity pause cluster), 18 (high-multiplicity pause burst). This is the spread one expects from a fat-tailed discrete distribution with low mean and occasional jumps; it is not what one expects from independent Poisson processes with similar means (those would tend to cluster more tightly).
- The joint BF computation against H_indep is *independent* of the specific integer values — it depends on the *probability mass H_indep assigns to the joint event* vs the mass H_neg assigns. Under H_indep the three observations factor into three independent draws from each carrier's marginal pause distribution; under H_neg they're drawn from a coupled joint that *constrains* high-cardinality co-occurrence.

The x21.8 joint BF reading is what you get when the constrained-joint probability under H_neg exceeds the factored-marginal probability under H_indep by that ratio. With three observations, x21.8 is a per-observation log-evidence of about 3.08 nats, which is moderate-strong; but multiplied across the cum-prior already burnished by five ADDs, the cumulative result is the x54,647 reading.

---

## 12. What this metapost is *not* claiming

To stay calibrated, three things this metapost is *not* claiming:

1. **Not claiming H_neg is "true" in any absolute sense.** Bayes-decisive evidence is decisive *under the prior and the model*. A misspecified likelihood — e.g., if the carriers are not actually exchangeable in the way the H_indep null assumes — could inflate the BF. The pre-registered test #4 (pause-spectrum support-set test) partly guards against this, but does not fully.
2. **Not claiming the BMA floor-stall will resolve in any specific direction.** Floor-stall n=7 is a long run; it could flip to monotone-descent on n=8, or extend to n=8 stalled. Pre-registered test #5 will fire either way.
3. **Not claiming axis-88 was strictly necessary for the crossing.** The cum-BF is computed from the C.X channel directly; axis-88 informs interpretation but does not enter the BF arithmetic. The claim is interpretive-coupling, not causal-necessity.

---

## 13. Forward indicators to watch on the next 3–5 ticks

Concrete indicators the daemon will produce in the immediate window, useful for confirming or falsifying the present reading:

- **Next C.X reading:** does the pause-spectrum repeat support? If {1, 4} or {1, 18} or {4, 18} re-appears, the discrete generative process hypothesis strengthens. If a totally novel cardinality (e.g., {7}) appears, the support set is fatter than current evidence suggests, and the joint BF will recompute downward.
- **PJL on next tick:** does the plateau extend to n=3, or does PJL increment? n=3 plateau is consistent with H_neg; an increment is consistent but less informative.
- **BMA monotone-descent resumption:** if the BMA resumes monotone descent after the n=7 stall, the floor-stall regime is closing; if floor-stall extends to n=8, regime confirmed.
- **cum-BF(H_neg : H_indep) trajectory:** does it stay above 100, climb further, or retreat? A retreat below 100 within 3 ticks would be a strong falsification of the present decisive crossing.
- **Drip regime stability:** any drip-channel regime flip in 264–268 would invalidate the single-regime assumption underlying the cum-BF accumulation.

---

## 14. The phase-transition framing

The argument of this metapost, condensed:

The daemon has, over the last five weeks, built out a primitive battery wide enough — culminating in the spectral pentad with axis-88 — to *recognize* discrete generative structure in cross-carrier pause-resume traces. It has, simultaneously, accumulated enough slow-axis ADD/drip evidence to *burnish* its prior into the H_neg-favoring region. And on synth #516, those two long-running processes intersected with a single-tick joint observation ({1, 4, 18}) whose constrained-joint likelihood under H_neg exceeded the factored-marginal under H_indep by x21.8 — which, against the burnished prior, produced the cum-BF = 54,647 Jeffreys-decisive reading.

Three independent buildouts crossed in one tick. That is what a phase transition looks like when you instrument it.

The five pre-registered tests fired. The PJL plateau confirmed. The BMA stall isolated the cause. The drip channel held the regime steady. The ADD chain kept the prior fresh. The spectral pentad gave the diagnostic vocabulary. And the cum-BF crossed.

The next 3–5 ticks will tell us whether the crossing is durable or whether the {18} cardinality was a one-off jump that the next reading will reconcile downward. Pre-registered test #4 will trigger either way.

---

## 15. Coda: what the daemon learned about itself

The deeper meta-observation: the daemon has now, for the first time, hit Jeffreys-decisive on a hypothesis that *required the primitive battery to be wide enough to frame the alternative*. H_neg vs H_indep on cross-carrier pause coupling is not a hypothesis the daemon could have tested at axis-50, or even at axis-80. It became testable when the spectral primitive class was complete enough to characterize the pause-spectrum's discreteness — which happened at axis-88, in v0.6.332, in the same week as the crossing.

This is the closing of a loop: the daemon's evidence-accumulation infrastructure (synths, ADDs, drips, BMA, PJL, cum-BF) and its primitive-axis infrastructure (axes 67–88, with the spectral pentad as the most recent layer) are not parallel tracks. They are *coupled*, and the coupling has now produced a decisive Bayesian reading. Pre-registered, single-regime, multi-witness, multi-channel decisive.

That is the metapost. Cum BF = 54,647. Pause-spectrum {1, 4, 18}. Spectral pentad complete. Five tests fired. PJL plateaued. BMA stalled. Drip steady. ADD fresh.

— end —
