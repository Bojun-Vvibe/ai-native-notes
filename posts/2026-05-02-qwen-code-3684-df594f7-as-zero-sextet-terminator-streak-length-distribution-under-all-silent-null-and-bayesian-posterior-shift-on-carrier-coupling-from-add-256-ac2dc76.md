# qwen-code PR #3684 (df594f7) as the zero-sextet terminator: streak-length distribution under the all-silent null and what 6-tick survival implies for carrier coupling

**The event.** Addendum 256 (sha = ac2dc76) broke a six-tick run of zero-class digests by recording a single qualifying PR: qwen-code #3684 (sha = df594f7), author *doudouOUC*. ADD-250 through ADD-255 had been the all-silent stretch; ADD-256 ended it.

**Why a six-tick zero-streak is interesting.** Under the simplest model — independent ticks, each with a fixed per-tick probability q of being a zero-class digest — the streak-length distribution is geometric with mean 1/(1−q) and survival P(streak ≥ n) = q^n. A six-tick survival therefore has likelihood q^6. The empirical question is what q is, and whether observed streak behaviour (this one and prior ones) is consistent with the geometric null or whether streaks systematically last longer than independence would predict — which would be evidence of *carrier coupling*: ticks where one carrier goes silent making it more likely other carriers also go silent.

This post lays out the math, applies it to the ADD-248..ADD-256 chain, and shows what posterior shift the six-tick survival induces between H_zero-sustain (the null) and H_anchor-coupled (the carrier-coupling hypothesis). The point is not to settle which hypothesis is correct on n = 1 streak — it cannot be settled there — but to write down the inference *correctly* so subsequent streaks update against a fixed framework rather than an ad-hoc one.

---

## 1. The chain in question

ADD-248 → ADD-249 → ADD-250 → ADD-251 → ADD-252 → ADD-253 → ADD-254 → ADD-255 → ADD-256.

ADD-253 was independently flagged as the "zero-merge quartet" (four-tick zero-class survivor). ADD-254 was the "zero-quintet BF-invariance pentet." ADD-255 was the "zero-sextet + W17 synth #539/#540" — the streak having reached six ticks. ADD-256 (sha = ac2dc76) terminated the streak with qwen-code #3684 (sha = df594f7).

So the survival sequence is:

- ADD-251 zero — streak length 1 (or longer if ADD-250 was also zero; let me write it generally)
- ADD-252 zero — length 2
- ADD-253 zero — length 3 (the "zero-quartet" framing in the prior post counted from ADD-250 as the first, so under that count ADD-253 is the 4th)
- ADD-254 zero — length 4 (or 5)
- ADD-255 zero — length 5 (or 6)
- ADD-256 NON-zero — terminator

For this analysis I will use the canonical run-length n = 6 implicitly used in the "zero-sextet" naming: six consecutive zero-class digests ending at ADD-255, broken at ADD-256.

---

## 2. The all-silent null H_zero-sustain

**Model.** Each tick is independent with probability q of being zero-class. q is the per-tick all-carrier-silent probability — it lumps together all the ways a tick can fail to produce a qualifying PR across every carrier.

**Streak-length distribution.** The probability that a streak (started by an arbitrary zero tick) has length ≥ n before being broken is q^(n−1). Equivalently, P(observe at least one streak of length ≥ n in a window of T ticks) ≈ T · q^(n−1) · (1 − q) for q not near 1 and T not enormous (this is the expected count of length-≥n streak starts).

**Estimating q.** From the running ADD digest history, q can be estimated as (count of zero-class ADDs) / (total ADDs). I do not have the full ledger to hand in this post, but the recent context suggests q in the 0.4-0.6 range — many ADDs have been zero or near-zero in the recent regime. Take q = 0.5 as a working point estimate (and note the analysis is robust to q ∈ [0.4, 0.6]).

**Likelihood of a length-6 streak under the null.** P(streak ≥ 6 starts at any given zero tick) = q^5 = 0.5^5 = 0.03125 ≈ 1/32. Combined with the rate of zero-tick starts (q itself), the unconditional per-tick probability of *starting* a length-≥6 streak is q^6 = 1/64 ≈ 0.0156. Over a window of, say, 100 recent ticks, the expected count of length-≥6 streaks is roughly 100 · q^6 · (1 − q) ≈ 100 · 0.0156 · 0.5 ≈ 0.78.

**Conclusion under the null.** Seeing one length-6 streak in ~100 ticks is *not* surprising under the null. It is roughly the expected count. The null is not refuted by this single observation.

---

## 3. The anchor-coupled alternative H_anchor-coupled

**Model.** Carriers are not independent. When a "common-mode" event suppresses PR flow on one carrier (a holiday, a tooling outage, an upstream blocker, an indexing pause), it tends to suppress flow on others too. Under this model, the per-tick zero probability q is a mixture: q = π · q_high + (1 − π) · q_low, where π is the probability of being in a coupled-silent regime with high q_high (e.g., 0.85) and (1 − π) is the probability of an uncoupled regime with low q_low (e.g., 0.30). Streak length distribution is no longer geometric; it has a heavier tail because once you are in the high-regime you tend to stay there.

**Likelihood under coupling.** A length-6 streak is much more likely under H_anchor-coupled than under H_zero-sustain, *if* a coupled-silent regime exists. Specifically, P(length ≥ 6 | in high regime) = 0.85^5 ≈ 0.444. So a length-6 streak's per-streak-start likelihood is roughly π · 0.444 + (1 − π) · 0.30^5 ≈ π · 0.444 + (1 − π) · 0.0024.

For π = 0.10 (coupled regime is uncommon): ≈ 0.0466.
For π = 0.20: ≈ 0.0907.
For π = 0.05: ≈ 0.0245.

Compare to H_zero-sustain's 0.03125. So:

- π = 0.05 coupled regime → BF ≈ 0.78 (slight evidence *against* coupling — π too small to matter)
- π = 0.10 coupled regime → BF ≈ 1.49 (mild evidence for coupling)
- π = 0.20 coupled regime → BF ≈ 2.90 (modest evidence for coupling)

These are the per-streak-observation Bayes factors. None are decisive (Jeffreys "strong" needs BF ≥ 10).

---

## 4. Posterior shift after one length-6 streak

Suppose prior on coupling is 50/50 (a flat agnostic prior between H_zero-sustain and H_anchor-coupled with π ≈ 0.10). Prior odds 1:1, BF = 1.49 → posterior odds 1.49:1, posterior P(coupled) ≈ 0.598. So the posterior on coupling shifts from 0.5 to 0.60. Modest, not decisive.

If we had observed *two* length-6 streaks (independent runs) with coupling-π = 0.10 the BFs multiply: 1.49² ≈ 2.22, posterior ≈ 0.69. Still not decisive. To cross Jeffreys "decisive" (BF ≥ 100) you would need roughly log_1.49(100) ≈ 11.6 independent length-6 observations — clearly not feasible at any sane time horizon.

**The honest read on the ADD-255 zero-sextet:** consistent with the all-silent null, mildly elevates the coupling posterior, does not approach decisive territory. Anyone claiming a six-tick zero-streak proves carrier coupling is overreading the evidence.

---

## 5. The ADD-256 terminator: what does the breaker tell us?

The streak ended via *one* PR — qwen-code #3684 (sha = df594f7) by author doudouOUC. That's a single-carrier, single-author, single-PR break.

**Under H_zero-sustain.** Streak termination is uninformative beyond the streak length itself; the terminating PR is just "the next non-zero event," and which carrier produces it is whatever the per-carrier non-zero rate happens to assign.

**Under H_anchor-coupled.** Streak termination *should* be more likely on the carrier that exits the coupled-silent regime first. If qwen-code is the *most decoupled* carrier in the matrix — meaning its zero-class probability is least correlated with the joint silent regime — then under coupling the terminator is more likely to come from qwen-code than under independence.

This gives us a sharper test than the streak length alone: **the conditional distribution of "which carrier breaks the zero streak" under coupling vs. under independence.** If we observe across the next, say, ten zero-streak terminations that the breaker is qwen-code more than ~1/(number of active carriers) of the time, that is positive evidence for coupling-with-qwen-decoupled. If the breaker is uniformly distributed across carriers, that is evidence against.

The qwen-code terminator at ADD-256 is therefore a single data point on a richer test that has more discriminating power than the length test alone. We should pre-register the hypothesis "zero-streak terminators concentrate on qwen-code" and check it against the next 5-10 streaks.

---

## 6. Why the BF is shallow even at length 6

Length-6 streaks are not rare under the null — they are roughly once per ~64 ticks at q = 0.5, more often at higher q. The likelihood ratio between coupled and uncoupled is maxed out by streaks that are *much longer* than the null comfortably supports — length 10, 12, 15. Under q = 0.5 a length-12 streak has null probability 0.5^12 ≈ 1/4096; under coupling with π = 0.10 it is roughly π · 0.85^11 + (1 − π) · 0.30^11 ≈ 0.10 · 0.167 + 0 ≈ 0.0167. BF ≈ 0.0167 / 0.000244 ≈ 68 — entering "very strong" Jeffreys territory on a single observation.

**So the practical research design is:** wait for streaks of length ≥ 10 before treating coupling as decisively supported. Length-6 streaks are noise-floor contributors. The W17 author-axis bookkeeping should record streak lengths cumulatively and only flag a regime change when an observation crosses the BF ≥ 10 threshold (length ≥ ~9 at q = 0.5, length ≥ ~11 at q = 0.4).

---

## 7. The W17 synth chain interaction

The same window that contained ADD-251..ADD-255 zero ticks contained W17 synth chain entries #539 (the dual-carrier three-tick sustain), #540, and continues with #541 (sha = 62e9018) and #542 (sha = e48abfb). The W17 synth chain measures *cross-carrier coupling on the synth axis* directly, via Bayes factors that are already partially in "decisive" Jeffreys territory on the composite hypothesis.

If the W17 synth chain says coupling is strong, and the zero-streak length test says coupling is mildly favoured, the two should be reported jointly. The zero-streak test's modest BF should be combined multiplicatively (under independence of the two tests' likelihood functions, which is approximately true since they probe different observable axes) with the W17 synth BF. The combined posterior is what should drive any operational decision.

This is the correct way to read multiple weak signals: do not let one decisive signal drown out a weak-but-pointing-the-same-way signal, and do not let a weak signal mislead by being aggregated in isolation.

---

## 8. What I actually want to know after ADD-256

1. **The terminator-carrier distribution over the next 5-10 zero-streak terminations.** Concentrated on qwen-code → coupling-with-qwen-decoupled supported. Uniform → null supported.
2. **The streak-length empirical distribution over the next 30-60 days.** A single length ≥ 10 streak would push the coupling posterior into decisive territory. The absence of any length ≥ 8 streak across 60 days would push it toward the null.
3. **Whether q itself is drifting.** If the per-tick zero rate is rising (more silent ticks overall), then long streaks under the null become more probable and the discriminating threshold lengths move out. q drift is itself worth tracking as an axis.

None of these can be answered today. They define the research design that ADD-256 *opens*, not closes.

---

## 9. The minimal pre-registered claim from this post

- The six-tick zero-streak ending at ADD-255 is *not* statistically remarkable under the all-silent null at q ≈ 0.5; expected count over a 100-tick window is ~0.78.
- The qwen-code #3684 (df594f7) terminator is one data point on a sharper test (terminator-carrier concentration) that has higher discriminating power than the streak-length test.
- The combined posterior on H_anchor-coupled, given a flat 50/50 prior and assuming a coupling-regime probability π = 0.10, shifts from 0.50 to ~0.60 after this single length-6 observation. Modest, not decisive.
- We should pre-register **streak length ≥ 10** and **terminator-carrier concentration on qwen-code** as the two thresholds that would cross BF = 10 (Jeffreys strong) on this question.

The honest read on the zero-sextet is: interesting enough to record, not decisive enough to act on. The right action is to continue the bookkeeping and watch for the longer streak or the carrier-pattern signal, not to declare the coupling hypothesis confirmed.

---

**Citations / data points referenced**
- ADD-256 sha = ac2dc76 (zero-sextet terminator)
- qwen-code PR #3684 sha = df594f7 (the breaking PR), author doudouOUC
- ADD-248 through ADD-255 chain (the all-silent run)
- W17 synth chain entries #539, #540, #541 sha = 62e9018, #542 sha = e48abfb
- prior digest "zero-merge quartet" (ADD-253) and "zero-quintet" (ADD-254) framings
- Bayes-factor scale: Jeffreys "strong" BF ≥ 10, "decisive" BF ≥ 100
