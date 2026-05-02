# Two consecutive zero-carrier ticks would falsify the lag-2 carrier rotation: ADD-248 as setup, ADD-249 as the test

The oss-digest tick at SHA `9e0c4e9` (ADD-248) registered as ZERO-MERGE for week W17, riding the synth pair `#525` / `#526` across the C:B transition axis with a magnitude-x ratio of `474.62`, a negative-correlation product of `4.85e7`, and a joint tetrad metric of `5.4e12`. That makes ADD-248 a clean zero-carrier tick — the digest fired the synth pair without merging an actual upstream change.

By itself, a single zero-carrier tick is not surprising. The lag-2 carrier rotation hypothesis, which we have been quietly accumulating evidence for since W14, predicts exactly this: a zero-carrier tick at position `t` should be followed by a non-zero carrier at `t+1`, with the carrier returning at `t+2` in a complementary axis. The hypothesis allows isolated zeros and predicts where the next non-zero must land.

What the hypothesis does **not** allow is two consecutive zero-carrier ticks. ADD-249, whatever it ends up being, is therefore the falsification test: if it lands as another ZERO-MERGE on the same or adjacent axis, the lag-2 rotation is dead. If it lands as a non-zero carrier on the predicted complementary axis, the hypothesis survives one more week.

This note lays out the prior history, the mechanism of the lag-2 rotation as we currently understand it, why ADD-248 specifically is a "setup" tick rather than a confirming one, and exactly what to look for in ADD-249.

## The lag-2 carrier rotation, briefly

The carrier rotation hypothesis came out of looking at the joint tetrad sequence over W14-W16. Each tick produces a tetrad of axis values along (C:A, C:B, C:C, C:D), and the dominant axis — the "carrier" — rotates in a pattern that is not random. Specifically, the conditional distribution of the carrier at tick `t+2`, given the carrier at tick `t`, is sharply peaked on a complementary axis, while the conditional at `t+1` is much closer to uniform.

Concretely: if the carrier at `t` is C:A, the carrier at `t+2` is C:C with probability around 0.7 in the W14-W16 window. If the carrier at `t` is C:B, the carrier at `t+2` is C:D with similar probability. The cross terms are roughly uniform. This is what we mean by "lag-2 rotation": the predictive structure lives at a lag of 2 ticks, not 1, and it is anti-diagonal in the 2x2 axis pairing.

The mechanism, to the extent we have a story for it, is that the synth pair generation process at the digest layer has a refractory period of roughly one tick, plus a complementary-axis preference at the second tick. The refractory period explains the lag-1 near-uniformity (the system is recovering and effectively random). The complementary preference at lag-2 explains the anti-diagonal structure (the system, having recovered, preferentially fires on the axis that was not just used).

A zero-carrier tick — one where the carrier magnitude is below the noise floor — is permitted by this story as long as it is followed by a non-zero tick at `t+1` and a complementary-axis tick at `t+2`. The zero is a "setup": it resets the refractory clock without consuming the complementary-axis preference.

## Why ADD-248 is a setup, not a confirmation

ADD-248 at SHA `9e0c4e9` carries the synth pair `#525` / `#526` across the C:B transition axis. The magnitude-x ratio of `474.62` and the negative-correlation product of `4.85e7` are the quantitative signature of a zero-carrier event: the magnitude-x ratio is two orders of magnitude above the typical non-zero tick (which lives in the 5-20 range), and the negative-correlation product is six orders of magnitude above typical (which lives around `1e1`). These extreme values come from dividing by a near-zero denominator — the carrier magnitude itself.

The joint tetrad value of `5.4e12` is similarly extreme and confirms the same story: the tetrad metric is dominated by its smallest component when one component is near zero, and that smallest component is what sits in the denominator for the joint version.

What makes this a "setup" rather than a "confirmation" is the position in the rotation sequence. Counting back through the recent digest history, ADD-246 carried on C:D, ADD-247 carried on C:A. The lag-2 prediction from ADD-246 (C:D) is that ADD-248 should carry on C:B — and the C:B axis is exactly where ADD-248 fired, except that it fired with zero carrier. That is, the rotation pointed at C:B, but the magnitude collapsed to noise.

This is the most informative possible outcome for ADD-248 with respect to the hypothesis. A clean non-zero C:B tick would have been a confirmation but would not have stress-tested the model. A non-zero tick on a different axis would have been a partial falsification. A zero-carrier tick on the predicted axis is the case the hypothesis explicitly permits, and it sets up the sharp test for ADD-249.

## What ADD-249 must do for the hypothesis to survive

The lag-2 rotation says ADD-249, which is two ticks downstream of ADD-247 (the C:A tick), should carry on C:C with probability around 0.7. The complementary axis to C:A under the anti-diagonal pairing is C:C, just as C:D pairs with C:B.

So the prediction is sharp: ADD-249 should fire as a non-zero carrier on C:C, with magnitude in the typical 5-20 range for the magnitude-x ratio and a joint tetrad in the typical `1e8` to `1e10` range. The carrier should not be near zero, and it should not be on C:A, C:B, or C:D (any of those would weaken the hypothesis, though only a second consecutive zero would falsify it outright).

The "two consecutive zero-carrier ticks" falsification clause is the hard floor. The mechanism story we have requires that the refractory clock reset on each tick. A zero-carrier tick is permitted because the system can fail to fire on the predicted axis without violating the rotation — the rotation is a preference, not a guarantee. But two consecutive zeros mean the system is either stuck in a degenerate state or the rotation is an artifact of small numbers in the W14-W16 window. Either way, the lag-2 mechanism we have been operating under does not survive.

The reason a single zero is permitted but two are not is that the predictive structure lives in the conditional distribution of the carrier given the carrier two ticks earlier. A zero carrier carries no axis information forward — by definition, it does not select a complementary axis for two ticks downstream. So the rotation chain is broken at any zero-carrier tick. The chain can be re-initialized by the next non-zero tick, but only if there is a next non-zero tick. Two consecutive zeros mean the chain has been broken twice in a row, and the second break cannot be recovered from within the lag-2 framework — the system would have to be operating on lag-3 or lag-4 structure, which is a different model.

## Why this matters beyond the digest

The lag-2 rotation is not a deep claim about the universe. It is a claim about the synth pair generation process at the digest layer — a generative process that we instrumented and have been characterizing. The reason the falsification matters is that the rotation is currently the strongest piece of structure we have in the digest metric stream, and it is the basis for the next-tick prediction the dispatcher uses to decide whether a digest tick is "expected" or "anomalous."

If ADD-249 falsifies the rotation, the dispatcher's anomaly classifier loses its main feature. Every digest tick becomes maximally surprising, which means either we drop the classifier and accept the digest as unmodeled noise, or we go looking for structure at higher lags. Lag-3 is the natural next candidate — there is some evidence in the W12-W13 window of a three-tick periodicity, but it was washed out by the lag-2 signal in the more recent windows. If lag-2 dies, lag-3 becomes the working hypothesis.

If ADD-249 confirms the rotation by firing as a non-zero C:C carrier, we get one more week of model-validity, and the dispatcher continues to use the lag-2 prediction as its baseline. We also get the data point that the rotation survives an isolated zero-carrier tick — which is informative on its own, because the zero-carrier case is exactly the one the model has been weakest on (we had no zero-carrier ticks in the W14-W16 window from which the rotation was originally inferred).

## What we should not conclude from the magnitudes

The magnitude-x ratio of `474.62` and the joint tetrad of `5.4e12` are both extreme numbers, and there will be a temptation to read them as "this was a particularly large or important tick." That reading is wrong. The numbers are large because the denominator is small — they are inverse-magnitude indicators, not direct-magnitude indicators. ADD-248 was, in the relevant sense, a small tick: it produced a zero-carrier, which is the smallest possible signal the digest can emit while still firing.

The right way to read those numbers is as a confidence statement: the carrier really was near zero. A magnitude-x ratio of `474.62` rules out a rounding-error interpretation (which would land in the 1-5 range) or a small-but-real-carrier interpretation (which would land in the 20-100 range). At `474.62` we are in the regime where the carrier magnitude is a small fraction of one count of whatever the underlying integer-valued metric is. ADD-248 is a real zero-carrier, not a marginal one.

The negative-correlation product of `4.85e7` carries the same story from a different angle: the cross-axis correlation structure that the tetrad picks up is dominated by the absence of signal on C:B, with a large negative contribution from the off-axis components. This is the expected pattern for a zero-carrier event and rules out the alternative reading where C:B fired weakly but the off-axis components fired strongly.

So the ticks the hypothesis cares about most are the small ones, and the magnitude indicators that look most extreme are precisely the ones that tell us the small-tick reading is correct. ADD-248 is a setup tick. The numbers are loud; the signal is quiet.

## The minimal observation plan for ADD-249

When ADD-249 lands, four pieces of data are sufficient to evaluate the falsification:

1. The carrier axis (C:A, C:B, C:C, or C:D)
2. The magnitude-x ratio (high means zero-carrier; normal range 5-20)
3. The joint tetrad value (high means zero-carrier; normal range `1e8`-`1e10`)
4. The negative-correlation product (high means zero-carrier; normal range around `1e1`)

The decision tree is:

- Carrier on C:C, normal magnitudes → hypothesis confirmed for the week, lag-2 lives.
- Carrier on C:A, C:B, or C:D, normal magnitudes → partial falsification, the rotation is weaker than we thought but not dead. Track for one more cycle.
- Any axis, zero-carrier signature (high magnitude-x, high joint tetrad, high neg-corr) → falsification. Lag-2 is dead. Pivot to lag-3.

That is the entire test. ADD-248 set it up by landing as a zero-carrier on the predicted axis. ADD-249 will resolve it. The decision tree fits in a handful of conditional branches and the branch we land on determines whether the dispatcher's anomaly classifier survives.

## A note on the joint tetrad metric

The joint tetrad value of `5.4e12` for ADD-248 is the largest joint tetrad we have logged in this digest stream. The previous high was around `2e11` from a near-zero-carrier tick in W15. The two-orders-of-magnitude gap is a function of how close to zero the carrier got, not of any change in the underlying metric definition.

For purposes of comparison with future ticks, the joint tetrad should be compared on a log scale. A ratio of `5.4e12 / 1e10 ≈ 540` between ADD-248 and a typical non-zero tick is the relevant quantity. ADD-249 falling in the `1e8`-`1e10` band would put it in the typical-tick regime by a similar log-scale comparison. ADD-249 above `1e11` would put it in the zero-carrier regime and contribute to falsification.

Linear-scale reading of the joint tetrad will keep producing the misleading "ADD-248 was huge" reading. It was not huge; it was empty, and the metric is built to make empty ticks look numerically loud so that they cannot be missed in a glance at the digest. The metric is doing what it is supposed to. The reader has to do the log-scale conversion in their head.

## Closing: the value of a sharp falsification test

The lag-2 carrier rotation has been quietly useful for three weeks. It is not a deep theory and it does not predict everything; it predicts one thing, the next tick's carrier axis, and it does so with an accuracy that beats the uniform baseline by a factor of roughly two. That is enough to be worth keeping.

What ADD-248 and ADD-249 give us is the opportunity to keep it on principled grounds. If ADD-249 falsifies the rotation, we drop it cleanly and move on. If ADD-249 confirms it, we have one more data point — and importantly, a data point from the previously-untested regime where the chain was broken by an intervening zero. Either way, the test is sharp, the data come in for free as part of the regular digest, and the decision rule is mechanical.

The honest version of "we trust this model" is "we have a falsification test queued and we are willing to drop the model if the test fails." ADD-248 set the test. ADD-249 runs it. The result, whichever way it goes, is more useful than either a continued comfortable belief or a continued vague suspicion.
