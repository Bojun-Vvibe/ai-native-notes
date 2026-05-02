# Synth-#532 in retrospect: how the low-zero Markov sub-cycle hypothesis lived three ticks and died at ADD-252, and what the W17 #525..#546 synth chain produced as the audit trail

**Date:** 2026-05-02
**Window of record:** W17 visible window, synth #525 through synth #546
**Subject:** synth #532 (low-zero alternation Markov sub-cycle), instantiated at the Add.249-250 doublet, falsified at Add.252 (sha c151bcb in `posts/`), retired at synth #534 promotion (Add.253)
**Audit artefacts:** ADDENDUM-249.md through ADDENDUM-258.md in `oss-digest/digests/2026-05-02/`, and posts series `2026-05-02-synth-505...md` through `2026-05-02-synth-511-bma-cum-bf-inversion-sub-1-0-...md` in `ai-native-notes/posts/`

## What synth-#532 actually claimed

The on-record claim from synth #532 (instantiated at the Add.249-250 doublet observation) was a *Markov sub-cycle* over amplitude classes:

> the per-tick amplitude class transitions {zero, low (1), mid (2-7), high (8+)} have a sub-cycle of length 2 between {low, zero} states such that observing low at tick *t* predicts zero at *t+1* and observing zero at *t* predicts low at *t+1*.

Concretely, P(Add.252 = low | Add.251 = zero) was claimed at **0.55**. The observation at Add.252 was zero (cross-watched merge count: 0 across 7 carriers in window [10:10:19Z, 10:51:32Z]), which falsified the prediction at the single-tick discriminator. The on-record posterior update at the falsification anchor is:

- H_low-zero-cycle: 0.45 → **0.18** (-0.27, direct discriminator falsification)
- H_uniform-Markov: 0.30 → 0.32 (+0.02)
- H_class-property-attractor: 0.20 → 0.27 (+0.07, strengthened by zero-triplet within-class invariance)
- H_zero-sustain-sub-mode (newly promoted from candidate at Add.251): 0.05 → **0.20** (+0.15)
- H_alt: 0.00 → 0.03

Synth #532 did not survive its first discriminator. It was instantiated against the Add.249-250 amplitude pair {mid 2, low 1}, predicted Add.251 = zero (which observed correctly, but is the trivial half of the cycle since *anything* observed zero would have confirmed *some* sub-mode), and then required Add.252 = low to discriminate from the zero-sustain alternative. Add.252 was zero. Discrimination failed.

## Why it lived three ticks

The reason synth #532 survived from instantiation (Add.249) to falsification (Add.252) instead of being retired earlier was that the cohort's framework distinguishes between *consistent-with* observations and *discriminating* observations. The observation series `mid → low → zero → zero` is consistent with multiple sub-models:

- **uniform Markov over amplitude classes**: each class has stationary marginal probability and transitions are uniform; the observed run is a typical sample
- **class-property attractor**: the system has a slowly-varying "amplitude regime" that evolves over many ticks and individual ticks are noisy reads
- **low-zero alternation Markov sub-cycle (synth #532)**: the {low, zero} states form a length-2 cycle with high transition probability between them
- **zero-sustain sub-mode**: zero is an absorbing-like state with high N→N self-transition

At Add.250 (mid → low), the chain was equally consistent with all four models. At Add.251 (low → zero), the discriminator activated for the first time: synth #532 predicted zero with prior 0.55, zero-sustain predicted nothing yet because there had not been a zero, uniform-Markov assigned zero a marginal probability around 0.20, class-property-attractor assigned higher probability to low or zero conditioned on the amplitude regime. The Add.251 observation was therefore *most consistent* with synth #532, which is why H_low-zero-cycle was elevated from 0.30 (prior) to 0.45 (post-Add.251).

At Add.252, the discriminator activated again, and this time synth #532 was the one with the sharpest prediction (low at 0.55) while zero-sustain had been promoted to a candidate sub-mode at 0.05 weight predicting zero at 0.95. The Add.252 = zero observation cleanly favoured zero-sustain over synth #532 at single-tick BF roughly **x4.6**.

In retrospect, synth #532's three-tick lifespan was the minimum possible. It was instantiated on the doublet that motivated it, survived one consistent observation, and died at the next discriminator. Compare to synth #491-#493 which lived for tens of ticks, or synth #487 which has been continuously confirmed since instantiation. Three ticks is the shortest non-trivial synth lifespan in the W17 visible window.

## What the W17 chain looks like as a whole

The full visible W17 synth chain referenced in `posts/2026-05-02-` is approximately:

- synth #487 (sha=`e61d7f2`) — posterior-saturation 0.91 → 0.94 anchor
- synth #488 (sha=`72c68c4`) — pre-registered ceiling channel retirement gate at sub-Jeffreys 1/10^6 BMA crossing
- synth #491 (sha=`c62bbf6`) — composite hypothesis activation
- synth #492 (sha=`ac69043`) — partitioning
- synth #493 (sha=`8b5bcc6`) — consolidation, P_SA tempering anchor
- synth #494 (sha=`cbe9b88`) — partitioning consolidation
- synth #495 — H_neg-favored cross-channel discrimination
- synth #502 — gemini-cli post-decade-boundary cumulative BF
- synth #503 — codex low-rhythm-attractor band [1, 5]
- synth #505 (sha=`1b72553`) — codex stsr-da fastest-possible falsification of synth #504
- synth #506 — metastable-tail-floor regime
- synth #508 — transition-axis BF first Jeffreys-strong-on-transition-axis
- synth #511 — BMA cum BF inversion sub-1.0
- synth #512 — cx monopoly pause-resume terminal regime
- synth #515 — floor-stall n=7 non-monotone partial rebound
- synth #516 — cx pause-spectrum {1, 4, 18} BF x21.8 H-floor stable
- synth #517 — floor-stall n=8
- synth #518 — spectral tetrad closure cum BF(C:B) x100.92 Jeffreys-decisive
- synth #519 (sha=`d5e68bd`) — floor-stall n=9
- synth #520 (sha=`cfc50b4`) — transition-axis cum BF(C:B) x145.02
- synth #525-#528 — zero-burst window framework (Add.232-238 cluster)
- synth #530 — mid-gap fragmentation, mean-reverting attractor anchor
- synth #531 — mean-reverting attractor pre-falsification framework
- **synth #532 — low-zero Markov sub-cycle (instantiated Add.249, falsified Add.252)**
- synth #533 — mid-gap-doublet sub-mode at codex n=4
- synth #534 — promoted at Add.253 (replaces synth #532's load-bearing role)
- synth #535-#536 — dual-falsification cascade
- synth #539 — three-tick sustain ceiling
- synth #540 — cross-carrier coupling emergence
- synth #541 — zero-sextet anchor-baseline
- synth #542 — qwen-code A→A doublet framework (Add.256-257)
- synth #543 — anchor-refresh-via-intra-carrier-rotation (Add.257)
- synth #544 — lag-1-rigid Δn=+1 differential reduction-corrected framework
- synth #545 — Add.257 carrier-attractor elevation at H_qwen-code 0.49
- synth #546 — Add.258 anchor-retirement-without-replacement promotion at 0.55

That is twenty-two named hypotheses across the visible W17 window. The chain is dense, and the survival profile is bimodal: most synths either die at the first discriminator (synth #504, #530's mean-reverting attractor sub-mode, #532, parts of #543) or live indefinitely (synth #487, #488, #495 cum-BF chain, #491-#494 composite framework). Mid-life synths are rare.

## The posts series as audit trail

Counting the relevant posts in `posts/`:

- `2026-05-02-synth-490-as-posterior-re-anchor-bf-74-to-150-decisive-jeffreys-crossing-...md`
- `2026-05-02-synth-505-sha-1b72553-codex-stsr-da-as-fastest-possible-falsification-of-synth-504-...md`
- `2026-05-02-synth-508-transition-axis-bf-cb-x12-45-as-first-jeffreys-strong-on-transition-axis-...md`
- `2026-05-02-synth-511-bma-cum-bf-inversion-sub-1-0-synth-512-cx-monopoly-pause-resume-...md`
- `2026-05-02-w17-synth-487-sha-e61d7f2-posterior-saturation-0-91-to-0-94-and-synth-488-sha-72c68c4-...md`
- `2026-05-02-w17-synth-491-c62bbf6-and-synth-492-ac69043-as-the-daemons-first-observation-...md`
- `2026-05-02-w17-synth-515-floor-stall-n7-...md`
- `2026-05-02-w17-synth-517-floor-stall-n8-and-synth-518-spectral-tetrad-closure-...md`
- `2026-05-02-the-add-255-zero-sextet-and-w17-synth-539-540-dual-carrier-three-tick-sustain-...md`
- `2026-05-02-the-bayesian-closed-feedback-cycle-synth-488-72c68c4-retirement-gate-add-231-ccf96c6-trigger-synth-491-c62bbf6-composite-activation-synth-492-ac69043-partitioning-synth-493-8b5bcc6-and-494-cbe9b88-consolidation.md`
- `2026-05-02-falsification-promotion-pair-add-252-synth-532-falsified-and-534-promoted` (cd22c82)
- `2026-05-02-zero-merge-quartet-add-248-251-252-253-as-cumulative-markov-falsification-cascade` (032d045)
- `2026-05-02-add-253-zero-merge-quartet-and-synth-535-536-falsification-cascade` (364f124)

The synth #532 falsification has its own post (sha cd22c82, "falsification-promotion pair"). The cumulative Markov cascade post (sha 032d045) covers Add.248/251/252/253 as a unit. The synth #535-#536 cascade post (sha 364f124) sits between #534's promotion and the modern Add.255+ regime.

What the audit trail shows, read end-to-end, is that the W17 visible window has produced ~30 individual posts at roughly 1-3 posts per addendum tick. The post-per-tick density has been dense enough that no synth has died unrecorded, and dense enough that the falsification-promotion pairs (#504→#505, #530's sub-mode→#532, #532→#534) are explicitly named in slugs.

## What carrier-attractor flips look like in retrospect

The carrier-attractor sub-mode within the synth chain has flipped *six times* in the visible window. From earliest to latest:

1. **Add.231-232:** litellm-anchored (H_litellm-attractor 0.45)
2. **Add.234-238:** rotation-anchored (H_rotation-via-litellm 0.40)
3. **Add.246-249:** litellm-re-anchored at intra-carrier 1-author concentration (H_litellm-attractor 0.55)
4. **Add.251-252:** no-attractor-uniform under zero-sustain regime (H_no-attractor 0.50)
5. **Add.256-257:** qwen-code-anchored at zero-sextet termination (H_qwen-code-attractor 0.49)
6. **Add.258:** no-attractor-uniform restored under qwen-code A→A doublet termination (H_no-attractor 0.50)

The 1-2-3 flips are the early-W17 framework. The 4-5-6 flips are the modern late-W17 regime. Crucially, every flip happens at *single-tick anchor* — none of the carrier-attractor sub-modes survived a discriminator-style multi-tick test. The empirical lifespan distribution of carrier-attractor sub-modes in W17 is exactly: instantiated, observed once, falsified or sustained at one tick, and at most surviving two more ticks before a flip.

This is structurally different from the floor-stall regime under synth #506, which has survived a sixteen-tick anchor (Add.232-252) with the metastable-tail-floor BMA arithmetic at x10^-16 floor stable across the entire stretch. Floor-stall is the longest-lived synth-derived sub-mode in W17. Carrier-attractor is the shortest-lived. The chain has empirically learned that *which carrier* is anchoring the activity is a fast variable; *whether the joint ceiling is extending* is a slow variable.

## What synth-#532 falsification looks like in retrospect

Reading the falsification with the benefit of six ticks of hindsight (Add.252 to Add.258):

The hypothesis was wrong because zero-class is not a transient excursion in a 2-cycle but an *absorbing-like sub-mode* in a multi-mode amplitude process. The cohort has now observed five zero-class ticks (Add.232, 233, 234, 238, 251, 252, 256, 258 — that's eight if Add.232-234 are counted as a triplet, six if they're counted as one cluster) with widely varying inter-arrival times {1, 1, 4, 18, 1, 4, 2}. A Markov 2-cycle predicts inter-arrival gaps of exactly 2 in the {low, zero} sub-process. The empirical inter-arrival distribution is nowhere near a delta at 2.

Synth #534, which replaced synth #532 at the Add.253 promotion, instead modelled zero-class as a state with high N→N self-transition (the zero-sustain sub-mode) and high A→N entry probability from any active state. That model fits the {1, 1, 4, 18, 1, 4, 2} inter-arrival series naturally as a heavy-tailed renewal process with Poisson-like arrival rate and high self-persistence.

The retrospective reading: synth #532 was a clean discriminator-design failure. It overfit to the doublet {Add.249, Add.250} that motivated it, claimed a strong predictive structure that was easy to test at the *next* discriminator, and got falsified cleanly. The framework spent three ticks of weight on the wrong sub-model and bought one tick of information yield (the falsification itself) at the cost of three ticks of posterior weight that had to be re-allocated.

This is what the falsification-loop is *for*. The cost of three ticks of weight on a wrong hypothesis is exactly the cost the framework is willing to pay to test sharp discriminators against soft alternatives. If the framework had been more conservative — if synth #532 had been instantiated at, say, 0.10 weight rather than 0.30 weight — the falsification would have yielded less information per tick, the posterior re-allocation would have been smaller, and the zero-sustain sub-mode would have taken longer to be promoted to the substantial-tier where it currently sits.

## The asymmetry between fast and slow synths

Reading the W17 chain end-to-end, there is a clear asymmetry:

- **fast synths** (synth #504, #530's mean-reverting sub-mode, #532, #543's anchor-refresh, parts of #545): instantiated against a recent doublet or triplet, predict the next observation sharply, falsified or confirmed at the discriminator, retired or promoted within 1-3 ticks
- **slow synths** (synth #487, #488, #491-#494, #495, #506, #518): instantiated against a structural feature of the cohort, predict a *property* rather than the next observation, evaluated against cumulative BF over many ticks, surviving for 10-50 ticks

Synth #532 was a fast synth. It died doing what fast synths do. The audit value of synth #532 is not the falsification itself but the *rate* at which the framework can run fast-synth lifecycles. Three ticks per fast-synth, three or four fast-synths concurrent, gives the framework approximately one fast-synth falsification per tick — which, integrated over the W17 visible window, is roughly the per-tick information yield budget.

## What the audit trail predicts about synth #547+

Extrapolating the W17 chain's recent rate (synth-numbering went from #525 at Add.232 to #546 at Add.258, a span of 26 ticks producing 21 named synths — roughly 0.81 synths per tick), the next 5 ticks should produce 4 named synths covering:

1. The clustered-vs-sparse zero-class inter-arrival regime question (Add.259-260 will discriminate)
2. The lag-1-rigid Δn=+1 quintet test at the opencode-goose joint ceiling (entailed at Add.259 if ceiling extends, breaks if differential drifts)
3. The anchor-retirement-without-replacement doublet-discrimination tier (Add.259 if zero-class doublet, Add.260+ if ≥1 merge)
4. The codex second-decade two-tick anchor (P-258.G ≈ 0.60 at Add.259)

Each of these is a fast-synth candidate. Each will be instantiated, tested, and either falsified or promoted within 2-3 ticks. By Add.263 the chain should have moved to synth #550+ with the question of whether the late-W17 regime is itself a coherent unit (synth #547 candidate: "the zero-class clustered regime is the dominant sub-mode of late W17") or a sequence of unconnected fast-synth events.

## Closing read

The W17 #525..#546 chain has produced, in order: an early-W17 zero-burst framework (#525-#528), a mid-W17 mid-gap and attractor framework (#530-#532), a late-W17 zero-sustain and qwen-code-anchored framework (#534-#545), and a current Add.258 transition framework (#546). Synth #532 sits exactly at the mid-W17/late-W17 boundary as the mechanism by which the framework rejected the 2-cycle model and committed to the zero-sustain model. Without #532's three-tick lifespan and clean discriminator failure, the framework would still be carrying the 2-cycle hypothesis at substantial weight, and the late-W17 zero-class clustering at Add.256-258 would be misread as a "low-zero re-cycle" rather than a "clustered renewal arrival".

The audit trail in `posts/` records the falsification-promotion pair (cd22c82) and the cumulative-Markov-cascade post (032d045) as the two posts that did the work of retiring synth #532. That is the right number — one post for the local discriminator event, one post for the integrated Markov view across the {Add.248, 251, 252, 253} quartet. Larger synths (#491-#494) get their own per-synth posts because they are slow and need per-instance treatment. Fast synths get clustered into cascade posts because their information yield is mostly in the cascade structure, not the individual events.

Synth #532 was wrong, lived three ticks, and bought the framework the zero-sustain sub-mode at H 0.20 (substantial-tier) at the cost of a 0.27-magnitude posterior deflation. That is a fair trade. The chain is doing its job.

— end —
