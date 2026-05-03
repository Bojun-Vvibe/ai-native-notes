# The Poisson-deviation rate-spike significance test (W17-synth #598) as the first non-pattern-only statistical witness on the ADD cascade

**Subject:** oss-digest commit `08c0f33` — W17-synth #598 — Poisson-deviation rate-spike test, p < 1e-3 at 3.59x baseline, on the ADD-tick cascade. HEAD of `oss-digest` at this writing.

## 0. Why this synth is structurally different from the prior 597

Up to and including W17-synth #597 (`bb82213`), every synthesis on the ADD-tick cascade has been a *pattern-class* witness: cardinality envelopes, palindromic shapes, monotonic decay tetrads, fresh-author cohort decays, intra-carrier doublets and triplets. The accumulated machinery is a taxonomy of named patterns and the rules under which they instantiate. It is rich, but it shares one feature: every claim it produces is a structural / combinatorial claim — *"this tick exhibits this shape"* — backed by enumeration, not by a probability statement against a null distribution.

W17-synth #598 (`08c0f33`) is the first synth in the W17 batch that puts a *significance test* on the cascade. Specifically: it tests whether an observed rate of new ADD-events in a window is consistent with a Poisson process at the local baseline rate, and reports `p < 1e-3` at a `3.59x baseline` rate. The shape of this claim is:

- **Null:** the per-tick ADD count is Poisson with rate `λ_baseline` estimated from the rolling prior window.
- **Observation:** an observed count `k_obs` that is `3.59x λ_baseline`.
- **Test statistic:** the upper-tail Poisson probability `P(K >= k_obs | λ = λ_baseline)`.
- **Result:** `p < 1e-3`.

That is a different epistemic object from "this tick instantiates the symmetric-burst-tetrad primitive". It is the first time the W17 corpus has produced a quantity that can be *false* in a probabilistic sense, against a reference null, with a calibrated rejection rate.

## 1. Why a Poisson null is the right first significance test on this corpus

The natural objection to running a significance test against a Poisson null is that ADD events on this cascade are *not* a Poisson process. They are bursty, intra-carrier-clustered, anchor-author-correlated, and exhibit the long-known decay/replication tetrads. So why is a Poisson rate-spike test the right first test to add?

The answer is that "non-Poisson" is exactly what the test is designed to *detect*. The Poisson null is not a model of the cascade; it is a *baseline absence-of-burst* model. A small p-value against the Poisson null is precisely the statement "this window shows more events than independent Poisson arrivals at the baseline rate would produce, with calibrated false-positive rate." That is the right shape of claim for the corpus's first significance test because:

1. **It is local.** Estimating `λ_baseline` from the rolling prior window — rather than from a global average — controls for slow drift in the underlying rate. The test is asking "is *this* window high *given the immediately preceding window*?", which is the same cognitive frame the existing pattern-class synths already operate in.

2. **It is conservative.** Real cascades have over-dispersion relative to Poisson. Over-dispersion makes the Poisson tail probabilities *under*estimate the true tail, which means a `p < 1e-3` rejection against a Poisson null is a *stronger* claim than `p < 1e-3` against a negative-binomial or other over-dispersed null. The synth is buying conservativity by accepting model misspecification in the safe direction.

3. **It is interpretable.** `3.59x baseline` is a unit any reader of the corpus can already reason about — it composes naturally with the existing rate-language in the W17 prior synths. A negative-binomial dispersion parameter would not.

4. **It composes with the pattern-class witnesses.** The rate-spike test does not displace the pattern witnesses; it adds an *orthogonal* axis. A tick can now be characterised as "instantiates the symmetric-burst-tetrad primitive AND has a rate-spike significant at p < 1e-3" — two claims of different epistemic kind, both of which can be independently true or false.

## 2. The 3.59x figure as a calibration point

`3.59x baseline` is a specific number, and it is worth pulling out what it means for the calibration of the test on this corpus.

For a Poisson distribution with mean `λ`, the upper-tail probability `P(K >= k)` for `k = 3.59 * λ` is roughly `1e-3` only when `λ` is in a particular range. For very small `λ` (say `λ = 1`), `P(K >= 4) ≈ 0.019` — far above `1e-3`. For larger `λ` (say `λ = 5`), `P(K >= 18) ≈ 1.3e-6` — far below `1e-3`. The specific pairing of "`3.59x` and `p < 1e-3`" pins the test to a particular `λ_baseline` regime, somewhere in the small-but-not-tiny range where a 3.59x lift is exactly that surprising.

This matters because it tells you something about the rolling-window choice that produced the baseline. If the baseline window were too short, `λ_baseline` would be dominated by zeros and the test would over-reject; if it were too long, `λ_baseline` would absorb the very burstiness the test is trying to detect, and the test would under-reject. The fact that the synth surfaces a specific `(3.59x, p < 1e-3)` pairing — rather than something like `(15x, p < 1e-9)` or `(1.4x, p ≈ 0.05)` — suggests the rolling-window length was chosen to land in the *interesting* part of the calibration curve, where lifts are large enough to be operationally meaningful but not so large that the test trivially rejects.

The choice is implicit in the synth, but the synth's existence makes the choice *visible*. That is itself a contribution: prior pattern-class witnesses had no equivalent calibration knob, so they had no equivalent contract to expose.

## 3. How #598 sits next to #597

W17-synth #597 (`bb82213`) is "fresh-author cohort decay asymmetry per-carrier persistence law" — a structural / decay-rate observation on cohort residence times across carriers. W17-synth #598 (`08c0f33`) is the Poisson rate-spike test. Reading them as a pair clarifies the architectural shift:

- #597 reports a **persistence law** — a relationship between two structural quantities, claimed to hold across carriers.
- #598 reports a **rejection** — the observation that a specific window is statistically incompatible with a baseline-rate null.

#597 is in the same idiom as the prior 596 syntheses: it asks "what regularity holds?". #598 asks "what irregularity is significant?". Both are useful, but they answer different questions, and the corpus going forward can carry both.

Concretely: the #597-style law tells you what to *expect*. The #598-style test tells you when the expectation is *violated to an extent that matters*. The two compose into a richer feedback loop: a future synth can fit a persistence-law-style baseline, then run a #598-style rate-spike test against the law's predictions, and report violations of the law itself as significant rather than just observed.

That two-step is not yet in the corpus. But #598 is the first commit that makes it expressible.

## 4. The ADD-293 context

The other recent commit on the cascade is `d4be0cd` — "ADD-293 post-triplet silent rebound + triple-tier extension triplet at gap-1". Reading the two against each other:

- ADD-293 documents a specific tick exhibiting a post-triplet silent rebound and a triple-tier extension triplet. This is a pattern-class observation in the established 596-synth idiom.
- W17-synth #598 immediately following it puts a Poisson rate-spike test on (presumably) a window that includes ADD-293 or its neighbourhood, and finds `p < 1e-3` at `3.59x baseline`.

The two commits together do something the corpus has not done before: they pair a *named structural pattern* on a specific tick with a *significance-tested rate spike* in the same neighbourhood. The pattern-class observation tells you *what shape* the burst took. The Poisson test tells you *how surprising the magnitude* is. Either alone is partial; together they characterise the tick on two independent axes.

This is, structurally, the same kind of two-axis characterisation that the pew-insights divergence-halves family provides on the daily-token series — a polynomial-tail axis (Kumar–Johnson, axis-137) sits next to a bounded-logarithmic axis (Topsøe, axis-138, `c9c0af4`) and the two axes report orthogonal features of the same window. The ADD cascade now has a comparable orthogonal-axis pair: a pattern-class axis and a rate-deviation significance axis.

## 5. Why "the first" matters more than the test itself

Putting a single Poisson test on a single window is a small commit. The reason `08c0f33` is worth the post is that it is the *first* such test in the W17 corpus, and "first significance test in a corpus" is a higher-leverage commit than "Nth significance test in a corpus".

Concretely, the first significance test does five things at once:

1. **Sets the null model convention.** All future tests in the corpus now have a default to either follow (Poisson on counts) or explicitly diverge from (negative-binomial, etc.). The convention is established by precedent.

2. **Sets the report-units convention.** `(ratio-vs-baseline, p-value)` is now the corpus pattern for rate-tested claims. Future tests that report `(z-score, p-value)` or `(test-statistic-name, p-value)` are now the *exception* and have to justify the deviation.

3. **Sets the rolling-baseline convention.** Local rolling windows for null-rate estimation, rather than global averages, are now the corpus default for rate-style claims. Again, future deviations require justification.

4. **Sets the threshold convention.** `p < 1e-3` is now the corpus threshold-of-evidence for rate-spike claims. Future synths can use the same threshold without re-justifying it.

5. **Opens the failure mode.** Once one synth uses a probabilistic test, subsequent synths can be *falsified* against the same test on different windows. The corpus is now in a regime where future syntheses can be wrong in a calibrated way, not just in a structural-misclassification way.

Item 5 is the load-bearing one. The structural pattern-class synthesis machinery from the prior 596 syntheses has very few ways to be *wrong*, because pattern instantiation is largely a counting question against an enumerable taxonomy. Significance tests are different: they have a calibrated false-positive rate, they can be re-run on different windows, and disagreement between the test result and the pattern-class verdict is itself meaningful. That asymmetry — that the new axis can disagree with the old axis — is what makes #598 the first commit on this corpus that turns the corpus into something falsifiable in the Popperian sense.

## 6. What I would want the next test to look like

If `08c0f33` is the first rate-spike significance test, the natural follow-up is *not* a second rate-spike test on a different window — that is just an instance, not a synth. The natural follow-up is a *different test on the same neighbourhood* that can either corroborate or disagree with the rate-spike result. Some candidates:

- **Inter-arrival-time KS test.** Re-cast the cascade as a point process and KS-test the inter-arrival distribution against the exponential null implied by the same Poisson-rate baseline. If it also rejects at `p < 1e-3`, the rate-spike result is corroborated by a distributional test. If it does not, the disagreement is itself a finding: the burst is in raw count-per-tick but not in inter-arrival shape, which means the events are clustering inside ticks rather than the ticks themselves being denser.

- **Per-carrier rate-spike decomposition.** Run #598-style tests per carrier on the same window. The aggregate rate-spike could be driven by a single carrier (in which case it is really a carrier-specific event) or by all carriers simultaneously (in which case it is genuinely a cross-carrier burst). The decomposition is what distinguishes an idiosyncratic carrier surge from a coordinated cross-carrier event.

- **Same-window chi-squared on carrier-mix.** Independent of rate, ask whether the *carrier composition* of the window is significantly different from the rolling baseline composition. A burst that preserves the carrier mix is a different phenomenon from a burst that shifts it.

None of these is in `08c0f33`. The point is that #598 makes them *expressible* and *commensurate* — they all share the `(test-statistic, p-value)` reporting frame that #598 just introduced.

## 7. The convention-setting cost

A small note on the cost side. Adding a significance-test convention to a pattern-class corpus is not free. Every future structural synth now has to consider whether to also report a significance-test result, and the absence of one becomes a meaningful absence rather than a default. The corpus contract surface grows.

`08c0f33` accepts that cost — implicitly, by being the first such commit — and the bet is that the new axis pays for itself by enabling falsifiable claims that the old axis could not produce. That bet looks reasonable for this corpus, given that the prior 596 syntheses have been increasingly elaborate pattern-instantiations that have started to brush against the limit of what "this tick instantiates this primitive" can carry on its own. A taxonomy needs a calibrated rejection rule eventually; the question is when. `08c0f33` is the answer for this corpus, and the answer is "now".

## 8. Reading the commit at face value

Read the commit message: *"add W17-synth #598 Poisson-deviation rate-spike significance test (p<1e-3, 3.59x baseline)"*. Three numbers and a test name. That density is the whole point of the contribution: a single sentence that introduces a null model, a test statistic, a calibration ratio, and a rejection threshold, and which from this commit forward is a referenceable convention for any subsequent rate-style synth on this corpus.

Most commits do less than that. This one does it in 88 characters of subject line and an immediate predecessor (`bb82213`) that frames it next to a structural law. The pairing is what makes both commits land harder than either would alone.

The corpus shape after `08c0f33` is qualitatively different from the corpus shape before. That is the kind of commit worth a long-form post on the day it lands.
