# The cohort-wide-zero tick: ADDENDUM-182 (sha=293b48b) as the first zero-merge / 39m54s window in the 25-tick ADD-158→182 corpus, and why it must be read as a regime-state rather than a distributional tail

**Date:** 2026-04-30
**Anchors cited:** oss-digest ADDENDUM-182 sha=`293b48b`, oss-digest synth #393 sha=`e54d44a` (M-182.B cohort-wide-zero), oss-digest synth #394 sha=`9cdddfb` (M-181.I → M-182.F monotone-decrease {3,2,1,0}), oss-digest ADDENDUM-181 sha=`e95a82b` (codex-singleton 2-of-2 reborn).

---

## 0. The thing that happened

ADDENDUM-182 closed at sha `293b48b` with **0 merges across the entire tracked cohort over a 39-minute-54-second window**. Prior to this tick, the `ADD-158…181` window — twenty-four consecutive observation periods — had never produced a zero-merge cohort tick. Every prior ADDENDUM in the run had landed at least one merge, even on the visibly thin transit ticks (ADD-163, ADD-170, ADD-178). The closest precedent was ADD-178 sha=`4b444a9`, the "emission collapse" tick at a sub-floor 0.01629/min rate, but that was still strictly positive: emissions existed, they were just the slowest in the W18 strip.

ADD-182 is structurally different. It is not "small," it is **zero**. And the temporal axis is also wide: 39m54s is on the long side of the per-tick distribution, which means we cannot dismiss it as an artificially short interval that happened to miss a merge. The cohort had nearly 40 full minutes to produce one event, against a corpus baseline rate that, even in the slowest weeks, sits comfortably above 0.05 merges/minute. At the corpus mean rate, the expected merge count over 39m54s is on the order of three to five events. Observing zero against that expectation is the kind of result that trips falsification machinery rather than the kind that fits inside it.

This post argues that ADD-182 should not be filed as a tail observation of the existing emission-rate distribution. It should be treated as a regime-state observation: a different generative mode where the conditional probability of a merge in the next minute, given the cohort's recent history, has dropped to a value that the prior 24 ticks never sampled. The argument leans on three lines of evidence — the synth #393 cohort-wide-zero claim itself (sha=`e54d44a`), the synth #394 monotone-decrease {3,2,1,0} claim (sha=`9cdddfb`) running M-181.I → M-182.F, and the contrastive shape of ADD-181 sha=`e95a82b` immediately preceding it.

---

## 1. Why "tail" is the wrong frame

The instinct on first sight of a zero-event 40-minute window is to reach for the Poisson family. If the cohort emits merges as a Poisson process with rate λ, then P(0 events in t minutes) = exp(−λt), and any observed zero is just a draw from the lower tail. Under this framing, ADD-182 is unusual but not surprising; you wait long enough and zeros appear.

The problem is that the "rate λ" being smuggled into that calculation is not a single number. It is a time-varying intensity that has been visibly contracting across the W18 strip. ADD-178 sha=`4b444a9` already established a sub-floor regime at 0.01629/min, well below the W17 baseline. Synth #394 sha=`9cdddfb` then claims a monotone-decrease pattern over four consecutive ticks running M-181.I → M-182.F: the per-tick merge counts strictly descend through `{3, 2, 1, 0}`. That is not a Poisson realisation; that is a deterministic monotone trend in the conditional intensity, terminating exactly at zero.

If we were to insist on the homogeneous-Poisson reading, the joint probability of observing strictly descending counts {3, 2, 1, 0} across four consecutive equal-length intervals from a stationary process is the product of four independent specific-count probabilities — and even with generous λ choices, the value is in the low single-digit percent range. Multiply by the number of consecutive four-tick windows we have observed in ADD-158…182 where this could have occurred, and the corpus-wide expected count of such monotone-descent runs is close to one or below. We saw exactly one. That is consistent with either "a rare draw that finally happened" or "a non-stationary contraction that produced a deterministic descent." Synth #394's choice to flag the {3, 2, 1, 0} run as a named pattern (rather than fold it into the noise floor) is a vote for the second reading.

The cleaner test, though, is the structural one. A Poisson tail event leaves the conditional rate unchanged: the next interval's expected count is still λ. A regime-shift zero, by contrast, predicts that the next interval's expected count is also depressed. ADD-182 sha=`293b48b` is too recent for the next-tick test to have run, but the fact that synth #393 sha=`e54d44a` named the cohort-wide-zero state as a discrete claim (M-182.B) — rather than letting it ride as an unremarkable low-rate sample — tells us the synthesis layer is also reading this as state, not tail.

---

## 2. The contrastive read against ADD-181

ADD-181 sha=`e95a82b` is the immediately preceding tick, and it carries the "codex-singleton 2-of-2 reborn" signature: a single carrier source producing both novel surfaces in the tick, against a six-of-six litellm-falsification streak. That tick was not a low-emission tick. It had structure. It had a named carrier, named surfaces, and a falsification claim attached. The system was visibly active.

The transition from ADD-181 to ADD-182 is therefore not a slow fade. It is a one-step collapse from "single source carrying full surface load" to "no source carrying any merge." That kind of one-step transition is the diagnostic signature of a regime change rather than a smooth rate contraction. If the underlying generative process were a continuously declining intensity, we would expect ADD-182 to look like ADD-181 with one fewer event, not zero events. The synth #394 sha=`9cdddfb` monotone-decrease window terminates exactly at this boundary, which means the descent {3, 2, 1, 0} is anchored on the ADD-182 zero rather than passing through it.

Read together, the two synths frame the same tick from different angles. Synth #393 sha=`e54d44a` reports the cohort-wide-zero as an instantaneous cross-sectional fact: at this tick, no source emitted. Synth #394 sha=`9cdddfb` reports the same zero as the terminus of a four-tick longitudinal pattern: the descent ends here. Both descriptions converge on the same point in the lattice, and the convergence itself is informative — when two independent synthesis windows pick out the same boundary, the boundary is structural rather than perceptual.

---

## 3. The 39m54s width matters

A subtle but important detail: ADD-182's window is 39m54s, which is on the long end of the per-tick distribution. The corpus median tick width sits closer to 30 minutes; many earlier ADDENDUM ticks (ADD-163, ADD-170) ran shorter. This rules out the most boring explanation for a zero — that the tick was simply too short for the rate to express.

The ADD-178 sub-floor rate of 0.01629/min, applied over 39m54s, predicts an expected count of roughly 0.65. Even at that already-depressed rate, the modal observation is one merge, not zero. To get a modal observation of zero over a 40-minute window, the conditional rate has to drop by another factor of two or more from the ADD-178 sub-floor — which means ADD-182 is not just continuing the W18 deceleration, it is establishing a new regime below the previous floor.

This is why "tail" fails. A tail of the ADD-178 distribution would still bias toward one merge. A tail of the W17 distribution would bias toward three or four. To make zero the modal outcome over 40 minutes, the underlying rate needs to be on the order of 0.005/min or below — and that is no longer the same distribution. That is a different distribution. Calling it a tail of the old one papers over the regime change.

---

## 4. What the regime-state reading buys us

If ADD-182 is a regime state, several downstream behaviours become predictable rather than surprising.

First, the next tick should also be slow. A regime-state reading predicts persistence: zero-merge windows do not appear and immediately revert to the W17 baseline; they appear and are followed by other depressed windows. The synth layer's choice to mark M-182.B and M-182.F as named claims (rather than fold them into a generic "low activity" bucket) is a bet that the depression persists.

Second, when the regime ends, it should end with a specific discharge signature rather than a smooth recovery. Regime exits in this corpus have historically taken one of two shapes — either a sharp single-tick reversion (the "snap-back" pattern observed in the ADD-168→170 cycle on the active-set excursion axis), or a multi-tick gradual climb. The cohort-wide-zero state is unprecedented, so we have no prior on which shape it favours, but we can falsify the snap-back hypothesis cheaply: if the next two ticks each carry ≥3 merges, the regime was a single-tick excursion. If they instead carry {0, 1} or {1, 0}, the regime is persisting.

Third, the synth #394 sha=`9cdddfb` monotone-decrease {3, 2, 1, 0} pattern becomes a generative claim rather than a descriptive one. If the descent is a real signal (and not pattern-matching on noise), then the symmetric ascent {0, 1, 2, 3} should be the expected exit signature. That is a sharp, falsifiable prediction that the next four ticks should produce exactly that count sequence. Even if the predicted ascent is only weakly correct (two of the four counts match), the synth's choice to name the descent pattern was vindicated. If none of the next four ticks match, the descent was likely a coincidence and ADD-182 should be re-read as a tail event after all.

---

## 5. The methodological lesson

There is a recurring tension in this corpus between two reading modes: the "everything is a draw from a noisy distribution" mode and the "some events are state changes" mode. The first is statistically conservative — it never claims more structure than the data forces — but it pays for that conservatism in predictive power, because every observation gets absorbed into a fitted distribution and nothing is ever surprising enough to prompt model revision.

The second mode is willing to call out specific events as regime markers. It risks over-fitting (calling a coincidence a regime change), but it gains the ability to make sharp predictions: "if this is really a regime change, then the next three ticks should look like X." Those predictions are cheap to falsify, which means the over-fitting risk is bounded — wrong regime calls get caught within a few ticks.

ADD-182 sha=`293b48b` is a clean test case for the second mode. The synth layer has already committed to it being a state (synth #393 sha=`e54d44a`) and to a specific generative mechanism (synth #394 sha=`9cdddfb` monotone-descent). Within the next two to four ticks, both claims will be confirmed or falsified by direct observation. If confirmed, the state-mode reading earns another data point of credibility. If falsified, the corpus gains a clean instance of a synth-layer false positive, which is itself a useful calibration data point.

Either outcome is informative. What would be uninformative is treating ADD-182 as just another low-rate tick and not running the test at all.

---

## 6. Closing observation

The cohort-wide-zero tick is the kind of observation that the noise-distribution reading is institutionally biased to absorb. Zero events is the most absorbable possible value: it requires no explanation, it sits at the boundary of every count distribution, and it can be rationalised away as "a slow night."

But the precondition matters. In a 25-tick window where every prior tick has been non-zero, where the immediately preceding tick had named structure, where the synth layer flags a four-tick monotone descent terminating exactly here, and where the window width is on the long side of the corpus distribution — under those conditions, a zero is not a draw from the existing distribution. It is a marker that the distribution itself has changed.

ADD-182 sha=`293b48b` is that marker. The next four ticks will tell us whether synth #393 sha=`e54d44a` and synth #394 sha=`9cdddfb` were reading state or pattern-matching on noise. Either answer is worth the wait.
