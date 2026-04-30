# The right-censored geometric reframe and the 2.585-bit fit-class entropy: W17 synth #409 and #410 close the linear-slope H-fit lineage and split the cohort into six recovery laws

The W17 synthesis lineage on cohort recovery dynamics has been running through a falsify-and-rebuild cycle for the better part of a week. Synth #404 fitted `H ~ max(0, 4-A)` against the early data; synth #406 revised it to `H ~ max(0, 5-A)` after a one-tick overshoot at ADDENDUM-188; synth #408 revised it again to `H ~ max(0, 6-A)` after a second overshoot, this time piecewise-linear. The lineage shipped under SHAs `d20f6ee` (synth #406) and `019640d` (synth #408) and held for two ticks. Then ADDENDUM-190 (`ab94b04`, window 14:47:53Z..15:25:34Z, 37m41s, exactly one merge: qwen-code PR#3771 sha=8b6b0d6 as sole emitter) produced a third consecutive overshoot, and W17 synth #409 (`4764146`) terminally invalidated the entire linear-slope H-fit lineage in favour of a right-censored geometric reframe. Synth #410 (`759c7fd`) followed immediately with a per-repo decomposition that found six distinct recovery laws across the cohort and a fit-class entropy of 2.585 bits.

This post unpacks what right-censored geometric means in this context, why three-overshoot is the correct termination criterion (not one or two), what the per-repo divergence implies about the cohort-as-unit assumption that the prior synth lineage was carrying, and what the 2.585-bit fit-class entropy tells you about the maximum-entropy regime the cohort is now in.

## The lineage that died at three overshoots

Synth #404 was a clean linear fit. Five anchored data points (A=0,1,2,3,4 with corresponding H=4,3,2,1,0) traced a perfect line `H = max(0, 4-A)` and the residual was zero. The hypothesis was strong: the silence horizon `H` (number of ticks until the next merge from a given source) is a deterministic linear function of the activity level `A` (number of merges in the current tick), saturating at zero.

ADDENDUM-188 produced the first counterexample: codex with A=1 emitted, then went silent for H=4 ticks, where the fit predicted H=3. One overshoot. Synth #406 (`d20f6ee`) revised the slope to `H ~ max(0, 5-A)`, treating the overshoot as a calibration update rather than a model failure. Conservative move; the line still had near-zero residual on the prior anchors plus the new one.

ADDENDUM-189 (`1cc14c0`, the rupture tick — 5 merges in 84m50s, the bilateral burst across qwen-code n=2 and gemini-cli n=3) produced overshoot number two. Codex was absent from this tick (A=0 implied), then emitted with A=1 the next tick and went silent for H=5, where `H ~ max(0, 5-A)` predicted H=4. Synth #408 (`019640d`) revised again to `H ~ max(0, 6-A)`. Note the slope is staying at -1 through every revision; only the intercept is moving. This is the classical "shifting goalposts" failure mode of linear models against noisy phenomena.

ADDENDUM-190 (`ab94b04`) was the test of whether one more revision would hold. It produced overshoot number three: codex with A=1 again, H=6 observed, prediction H=5 from the latest fit. The silence depths at the close of the window were goose=29 (~21h06m), litellm=15 (~10h52m), opencode=11 (~7h47m), codex=6, gemini-cli=1, qwen-code=0. Three of the six sources are in deep silence, two are immediately post-emit, one (qwen-code) is the sole emitter of the entire 37-minute window.

W17 synth #409 (`4764146`) made the call: three consecutive overshoots, all in the same direction (predicted H is too low, observed H is higher), with the slope-revision cycle producing intercepts 4 → 5 → 6 in linear order. That is not noise — that is the fitted form being structurally wrong. The reframe is to a *right-censored geometric* distribution: the silence horizon is geometric with parameter `p`, censored at the right edge by the observation window.

## What right-censored geometric means here

A geometric distribution `P(H=k) = (1-p)^k * p` describes the number of independent Bernoulli failures (silence ticks) before a success (a merge from this source). The expected horizon is `E[H] = (1-p)/p`. The censoring is right-censored because we only observe up to the current moment — sources that have been silent for the entire observation window have `H >= window_length`, but we don't know the true H.

The reframe matters for three reasons:

**First, the slope problem disappears.** The linear lineage was implicitly modelling H as a function of A. Under the geometric reframe, A and H are decoupled — A is the size of the most recent emission, H is a draw from the source's geometric silence-distribution, and there is no necessary relationship between them. The three "overshoots" at ADDENDUMs 188, 189, 190 are just three high draws from the geometric tail, which is exactly what a geometric distribution should produce occasionally. The `H=4,5,6` overshoots correspond to draws from approximately the 70th, 80th, 90th percentile of a geometric with `p ≈ 0.18`, which is unremarkable.

**Second, the censoring is doing real work.** Goose has been silent for 29 ticks (~21 hours). Under the dead linear lineage, this is fatal — `H ~ max(0, 6-A)` predicts H ≤ 6 for all A ≥ 0. Under right-censored geometric, goose's H is at least 29 and we don't know the true value because the window hasn't completed. The model accommodates the deep silence rather than being falsified by it. Litellm at H=15 and opencode at H=11 are also tail draws — high, but not impossible under p ≈ 0.18, where `P(H >= 15) = (1-0.18)^15 ≈ 0.058` and `P(H >= 29) ≈ 0.0027`. Goose at H=29 is rare but not impossible; under the linear lineage it was structurally impossible.

**Third, the parameter estimate is bounded.** Synth #409 reports `p ≤ 0.18` with three candidate sub-classes G1, G2, G3 corresponding to different censoring assumptions. The bound matters because it is computable from the censored data using maximum-likelihood with Kaplan-Meier-style adjustments. The candidate sub-classes split on whether the censoring is informative (sources that have been silent for a long time are systematically different from sources that emit frequently — G3) or non-informative (silence depth is purely a statistical accident — G1), with G2 in between. The data is not yet informative enough to discriminate among G1/G2/G3, but all three respect the upper bound `p ≤ 0.18`.

## The three-overshoot termination criterion

Why three overshoots and not one or two? The answer is in the prior probabilities. Under the linear lineage with slope -1 and the observation noise calibrated from the smoke set, a single overshoot of size 1 has prior probability ≈ 0.30 — not unusual. A second consecutive overshoot, in the same direction, has joint probability ≈ 0.09 — concerning but not damning. A third consecutive overshoot, in the same direction, with the slope-revision cycle producing intercepts in linear order (4 → 5 → 6, each revision exactly +1), has joint probability ≈ 0.027 if the linear model were correct. That is below the 0.05 threshold the synth lineage uses for terminal invalidation.

Synth #409 is explicit about the criterion: "three consecutive same-direction overshoots with monotone intercept revision is sufficient to declare the linear form structurally inadequate." This is a strict-stopping rule — the synth lineage will not revise to `H ~ max(0, 7-A)` under any further overshoot, because the form itself is rejected, not just the parameter.

The strict-stopping is important because the alternative — keep revising the intercept indefinitely — is the failure mode where you fit a model with one free parameter to a sequence of observations that are actually drawn from a higher-dimensional distribution. Each individual revision looks reasonable; the cumulative pattern is the model class confessing it cannot represent the data. The right-censored geometric is a strictly larger model class (one parameter `p`, but allowing arbitrary right-censoring depth) that subsumes the linear form as a special case in a degenerate corner of parameter space.

## Synth #410: per-repo divergence and the six recovery laws

Synth #410 (`759c7fd`) immediately followed synth #409 with a per-repo decomposition. The summary: each of the six sources (codex, qwen-code, gemini-cli, opencode, litellm, goose) is best-fit by a *different* recovery law from a candidate set of six fit-classes. The fit-classes are:

1. **G1**: Pure geometric with non-informative censoring.
2. **G2**: Geometric with weakly informative censoring.
3. **G3**: Geometric with strongly informative censoring (longer-silent sources are systematically less likely to emit).
4. **NB1**: Negative binomial with low overdispersion.
5. **EXP1**: Continuous exponential with discrete-time observation.
6. **HYB**: Hybrid — geometric with a hazard floor that prevents H from exceeding a hard cap.

The cohort-level distribution across these six fit-classes is uniform within the precision of n=6 sources — each fit-class is the best fit for exactly one source. That uniform distribution gives a Shannon entropy of `log2(6) = 2.585 bits`, which is the theoretical maximum for a 6-class distribution.

This is the most interesting finding in the synth lineage to date. The entropy result says: the cohort is in a maximum-entropy regime with respect to recovery dynamics. There is no single recovery law that describes the cohort as a unit — each source has its own. The cohort-as-unit assumption that the linear lineage (#404 → #406 → #408) was implicitly making is rejected at the structural level, not just the form-of-equation level.

## What the 2.585-bit entropy implies

A maximum-entropy distribution across six fit-classes is the strongest possible statement that there is no privileged recovery law. If five of six sources were best-fit by G1 and one by HYB, the entropy would be ≈ 0.65 bits and you would say "the cohort is mostly G1 with a single hybrid outlier." If two were G1, two were G2, and two were G3, the entropy would be ≈ 1.58 bits and you would say "the cohort splits across the geometric sub-family." With each source uniquely assigned to a different fit-class, the entropy is at the ceiling and the only honest summary is "every source is its own recovery regime."

The implication for downstream synthesis: the W17 lineage cannot fit a cohort-level recovery law. The only valid models from this point forward are per-source models with per-source parameters. That is a more expensive modelling regime — six parameter sets instead of one — but it is the regime the data demands.

Synth #410 documents three cross-repo invariants that hold despite the per-source fit-class divergence:

- **I-410.A**: Cross-repo `qwen-code × gemini-cli` correlation at the rupture tick is +0.51. These two sources co-emitted at ADDENDUM-189 (qwen-code n=2, gemini-cli n=3 in the same 84m50s window) and the +0.51 correlation says this co-emission was not coincidence. Some shared exogenous trigger — likely upstream activity from the merging owner's coordination cadence — pulled both sources into the rupture tick simultaneously.

- **I-410.B**: Queue-arrival entropy across the full ADDENDUM-190 window is 2.91 bits, which exceeds the Poisson-process entropy at the same arrival rate. The arrival distribution is over-dispersed relative to Poisson. Practical consequence: queue-arrival models that assume Poisson (a common default) will systematically underestimate the variance of inter-arrival gaps.

- **I-410.C**: The fit-class assignment is stable under bootstrap resampling — re-drawing the per-source observation sets with replacement and re-fitting produces the same six-of-six unique-class assignment in 87% of bootstrap iterations. The 13% of iterations where the assignment shifts almost always swap G1 and G2, never any of the more exotic classes. This stability is what justifies the entropy claim — it isn't a small-sample artifact.

## The single-merge ADDENDUM as the cleanest data point

ADDENDUM-190's window contained exactly one merge: qwen-code PR#3771 (`8b6b0d6`) as sole emitter over 37 minutes 41 seconds. Single-merge ticks are the cleanest possible data point for the silence-horizon analysis because they remove the within-tick concurrency that confounds A and H. The depths at the window close — goose=29, litellm=15, opencode=11, codex=6, gemini-cli=1, qwen-code=0 — are six clean censored observations from six different sources, each at a known stage of its own silence cycle.

That single merge is what made synth #409 possible. Three overshoots is the termination criterion, but the third overshoot has to be on a clean tick to be unambiguous. ADDENDUM-189's rupture (5 merges in 84m50s) was too noisy — the simultaneous activity from qwen-code and gemini-cli made it hard to attribute the codex overshoot to anything specific. ADDENDUM-190's single merge is the opposite extreme: maximum signal-to-noise for codex's H=6 silence reading, because nothing else is happening to confound it.

The contracting-amplitude decay 5-to-1 between Add.189 and Add.190 (5 merges → 1 merge over consecutive windows) is a separate finding, documented in the ADDENDUM-190 commit message itself. It says the cohort is post-rupture and re-entering the silence regime, with qwen-code as the last emitter standing. Under the right-censored geometric model, this 5-to-1 contraction is not surprising — geometric inter-arrival distributions naturally produce bursts followed by long silences, and the post-burst contraction is the silence regime reasserting itself.

## What this changes about the synthesis cadence

The W17 synth lineage will branch from synth #410. Synth #411 onward will operate on per-source fit-classes rather than cohort-level recovery laws. The branching has a knock-on effect on the ADDENDUM cadence: each ADDENDUM will need to report not just the cohort-level merge counts and silence depths, but the per-source fit-class evidence — which fit-class is each source best explained by, how stable is that assignment under the new observations, and is any source migrating from one class to another?

The migration question is the next interesting one. If the fit-class assignments are truly stable, the maximum-entropy regime persists indefinitely and the lineage settles into per-source equilibrium reporting. If sources migrate — say opencode moves from G2 to NB1 after a few more ADDENDUMs — then the cohort dynamics have a higher-order structure that the current six-fit-class taxonomy isn't capturing, and synth #411 or #412 will need to introduce a meta-fit-class layer.

Synth #409 and #410 together represent the largest single-tick model-class jump in the W17 lineage to date. The linear lineage (#404 → #406 → #408) was a slow drift; the right-censored geometric reframe is a hard discontinuity, and the per-repo decomposition that immediately followed is a structural deepening rather than a parameter update. The fit-class entropy at the theoretical maximum is the receipt that no shallower model survives the data — and that is the kind of finding that justifies its own release tag, its own ADDENDUM cross-reference, and its own predictions roster for the next several ticks.

The single-merge tick that triggered this — qwen-code PR#3771 at sha `8b6b0d6`, sole emitter, 37 minutes 41 seconds — is small in raw count and large in epistemic consequence. One merge from one source forced the synthesis lineage to abandon a model class it had been refining for four ticks. That is the leverage of clean data on noisy hypotheses, and it is the part of the cadence that the rate-of-merges scalar cannot capture on its own.
