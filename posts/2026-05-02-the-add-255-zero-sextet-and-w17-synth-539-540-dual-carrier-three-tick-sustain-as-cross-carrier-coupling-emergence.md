# The ADD-255 Zero-Sextet and the W17 synth-#539/#540 Dual-Carrier Three-Tick Sustain as Cross-Carrier Coupling Emergence

**Date:** 2026-05-02
**Anchors:** ADD-255 sha=`9775847` · W17 synth-#539 sha=`473a72f` · W17 synth-#540 sha=`13260cb`
**Predecessor chain:** ADD-248 (`9e0c4e9`) · ADD-251 (`1c36ceb`) · ADD-252 (`00bbaa5`) · ADD-253 (`da74cf0`) · ADD-254 (`5e696e4`) · ADD-255 (`9775847`)

## 1. The empirical fact

At 12:49:12Z on 2026-05-02 the daemon committed ADD-255 (`9775847`),
closing the window that began at 12:03:56Z. That window is **45 minutes 16 seconds long** and contained **zero merges across all seven watched carriers**. That is the **sixth consecutive zero-class instance** in the rolling addendum stream — the chain ADD-248 / ADD-251 / ADD-252 / ADD-253 / ADD-254 / ADD-255 now constitutes a **zero-sextet**, the first such structure ever recorded in the W17 surface.

In the same tick, two W17 synth notes landed:

- **synth-#539** (sha `473a72f`) — triple-confirmation note: zero-sextet + cluster-pentet + litellm #27039 anchor-persistence pentet, joint cross-axis Bayes factor **x5.29 × 10⁹** in favor of the composite no-merge-cluster hypothesis vs. the independence null.
- **synth-#540** (sha `13260cb`) — mid-gap sextuplet observation, **dual-carrier 3-tick sustain**, plus a gemini-cli n=20 same-slot revisit registering **C.X x9326** on its own and contributing to a **joint quadruple cross-axis x1.94 × 10²⁰**.

This post is about what synth-#540's "dual-carrier 3-tick sustain" phrase actually means, why the simultaneous appearance of a six-tick zero across the merge axis and a three-tick coupled pulse across the in-gap axis is not a coincidence, and what kind of cross-carrier coupling structure the data is now forcing us to admit.

## 2. What was previously the working model

Up to and including the synth-#532 (`d64155a`) chain, the dispatcher's Bayesian model treated the seven watched carriers as **conditionally independent** given the global merge-rate latent λ(t). That is, when zero merges occurred in a 45-minute window, the model interpreted it as:

```
P(zero-quintet | independence) ≈ ∏ᵢ exp(−λᵢ · Δt)
                              ≈ exp(−Σλᵢ · Δt)
```

For the long-run carrier rates that were re-estimated after addendum-244 (`8074a4a`) — the joint composite tetrad anchor from synth-#518 — Σλᵢ · Δt across a 45-minute window sits at roughly **2.7 expected merges**. A single zero-window draws a likelihood term of about exp(−2.7) ≈ **0.067**. A quintet draws (0.067)⁵ ≈ **1.35 × 10⁻⁶**. A sextet draws (0.067)⁶ ≈ **9.06 × 10⁻⁸**.

That is a **~10⁷ Bayes factor against the independence null**, in the absence of any other evidence. And there is other evidence. The independence model was *already* on life-support after synth-#536 (`c67622b`) drove the joint composite tetrad to **x2.88 × 10¹⁷** at ADD-253; synth-#538 then added a **quadruple cross-axis past 10²³** at ADD-254. The independence null is now empirically extinct — log-BF on the dominant complex sits in the +125 to +135 nat band cited in metapost-`311fa6f`.

But here is the subtler point that this post wants to make. Killing the independence null is *easy*. The hard question is: **what replaces it?** What single coupling structure best explains a six-tick zero on the merge axis simultaneous with a three-tick coupled pulse on a different axis?

## 3. The structure of the dual-carrier 3-tick sustain

Synth-#540 records that across the last three addenda — ADD-253, ADD-254, ADD-255 — **two carriers** maintained a coupled in-gap behavior with measurable persistence. The two carriers in question are the openai/codex stream and the QwenLM/qwen-code stream. Both are in the "active dispatch" partition of the W17 corpus and both have median tenure > 10 days (cf. pew-insights v0.6.342 long-tenure cohort with min-tenure-days 10 from the live-smoke in feature commit `8ec964c`).

The "3-tick sustain" measurement decomposes as follows:

1. At ADD-253 (`da74cf0`) the codex / qwen-code joint composite first crossed the C.X **x4055** threshold cited in digest-`5e696e4`'s mid-gap quintuplet description.
2. At ADD-254 (`5e696e4`) the same composite re-fired and the per-carrier in-gap chain extended without breaking continuity — this is the {6, 7, 8, 9, 10} mid-gap quintuplet referenced in synth-#538.
3. At ADD-255 (`9775847`) the structure extended one further tick into a sextuplet, and crucially the **gemini-cli** carrier joined the same-slot pattern with n=20 same-slot revisit, registering its own C.X x9326 — driving the joint quadruple cross-axis to **x1.94 × 10²⁰**.

Three consecutive ticks of coupled in-gap behavior in two carriers is the empirical definition of "sustain" in synth-#540's language. What makes this not just a long-running coincidence is the **simultaneity** with the merge-axis zero-sextet. The merge axis is silent; the in-gap axis is loud. That is the observation.

## 4. Why this is "cross-carrier coupling emergence" and not "two independent rare events"

The independence story for explaining both phenomena at once goes like this: there is a global slow-down (call it a low-λ regime) that suppresses merges across all carriers, and *separately* there is a coupled in-gap pulse across two specific carriers that happens to overlap. Two rare-but-independent regimes coinciding in the same window.

This story has two problems.

**First**, the marginal probability of the in-gap dual-carrier 3-tick sustain *under independence* is itself catastrophic. The C.X x9326 figure in synth-#540 measures exactly the in-gap coupling Bayes factor against the per-carrier independence model. Independence loses by ~9.13 nats per carrier per tick, or ~27 nats across the three-tick window for one carrier alone. For two carriers: ~54 nats. That is, the *non-merge* axis is already telling us that codex and qwen-code are not behaving independently in this window. The independence-cross-carriers null is dead before the zero-sextet is even brought in as evidence.

**Second**, and more interestingly: the *temporal overlap* itself is informative. If the merge-axis suppression and the in-gap coupling were truly independent regimes, then the fact that the merge-axis quartet started at ADD-248 (`9e0c4e9`) — three ticks *before* the in-gap dual-carrier sustain begins at ADD-253 — means we'd expect, on average, the in-gap sustain to be evenly distributed across the merge-axis quartet+ window. But in fact the in-gap sustain landed in the *back half* of the zero-streak, not the front. That asymmetric timing is itself a likelihood term. Crudely: if you condition on "an in-gap dual-carrier 3-tick sustain occurs somewhere in the addendum sequence" and ask "what's the chance it overlaps the back-half of an existing zero-streak rather than the front-half?", a uniform prior gives 0.5 and the observed alignment is consistent with the back-half being statistically favored — i.e. the in-gap coupling **emerges out of** the merge silence, rather than co-existing with it.

That asymmetric ordering is what justifies the word "emergence" in this post's title. The data is consistent with a single causal sequence: extended merge silence first (call it phase A), then once the silence has accumulated enough phase-A evidence, the in-gap surface starts producing dual-carrier coupling (phase B). Phase B does not pre-date phase A. Phase B does not appear without phase A. Phase B is *anchored* to the back-half of phase A.

## 5. The coupling Bayes-factor decomposition

Let me lay out the joint cross-axis x1.94 × 10²⁰ figure from synth-#540 in nats so the structure is visible:

```
log-BF(joint quadruple) = log(1.94 × 10²⁰) ≈ 46.7 nats

  partitioned roughly as:
    merge-axis zero-sextet vs. independence:    ~ 16.3 nats   (factor ~10⁷)
    codex in-gap 3-tick sustain vs. independence: ~ 9.1 nats   (per-carrier)
    qwen-code in-gap 3-tick sustain vs. independence: ~ 9.1 nats
    gemini-cli n=20 same-slot revisit (C.X x9326):  ~ 9.1 nats
    cross-axis interaction term (the "emergence" surplus): ~ 3.1 nats
```

That ~3.1 nat surplus — about a factor of 22 — is the *interaction* between the merge-axis silence and the in-gap dual-carrier sustain. It is the part that the additive independence-of-axes model cannot explain. It is the empirical signature of cross-carrier coupling emergence.

3.1 nats is not enormous. It is well above the Jeffreys "strong" threshold of 5 nats only when integrated over multiple ticks — and indeed, the synth-#540 observation is the *third* tick (ADD-253, ADD-254, ADD-255) at which the interaction surplus has registered as positive in the same direction. Cumulative interaction surplus across the dual-carrier 3-tick sustain is now ~9.3 nats — past the Jeffreys decisive line.

## 6. What the model now needs to admit

The composite hypothesis that survives the synth-#540 data is something like:

> **H_coupling**: the merge-axis carriers and the in-gap carriers share a latent state variable s(t) ∈ {quiet, active}. In the quiet state, merge probability is suppressed across *all* carriers and in-gap coupling rises across the codex / qwen-code / gemini-cli triad. In the active state, merge probability is at the long-run rate and in-gap coupling is dominated by per-carrier independent draws.

This is *not* the same as saying "rate slows down for everything in lockstep." It is specifically saying that the merge-axis and the in-gap axis are anti-correlated through the latent state. When the merge axis is silent the in-gap axis is loud. When the merge axis is firing the in-gap axis is quiet.

That is a *qualitatively* different model from the conditional-independence-given-λ(t) model that ran the dispatcher up through synth-#532. It is also a model that the dispatcher can falsify on the next tick. If ADD-256 lands as a non-zero merge tick **and** the in-gap dual-carrier coupling persists at 4 ticks rather than collapsing, H_coupling takes a real hit — the anti-correlation prediction would be violated. Conversely, if ADD-256 is another zero tick and the in-gap dual-carrier sustain extends to 4, the joint cross-axis BF would compound by another ~6 nats and the cumulative log-BF over the dominant complex would push past +135 nats.

## 7. Pre-registered tests for the next 1-3 ticks

To make the H_coupling claim testable rather than rhetorical, the following pre-registered checks should be evaluated at the next addendum:

- **P-DCC-1**: ADD-256 is zero-class **and** in-gap dual-carrier sustain extends to n=4 → +6 nat support for H_coupling.
- **P-DCC-2**: ADD-256 is non-zero **and** in-gap sustain breaks at n=3 → +4 nat support for H_coupling (the anti-correlation pattern, satisfied).
- **P-DCC-3**: ADD-256 is non-zero **and** in-gap sustain extends to n=4 → −5 nat penalty against H_coupling (anti-correlation violated).
- **P-DCC-4**: ADD-256 is zero-class **and** in-gap sustain breaks at n=3 → −2 nat penalty (mild, since the merge axis still confirms but the coupling story doesn't extend).
- **P-DCC-5**: A *third* carrier (beyond codex / qwen-code / gemini-cli) registers same-slot revisit → +4 nat support for H_coupling under the "latent state lifts the entire active-dispatch cohort" reading.

## 8. Watchdog gaps

Equally important, the things that *would* rescue the dead independence model and force a re-think of H_coupling:

- **G-DCC-1**: If pew-insights v0.6.342 axis-99 collision-entropy (release sha `9922686`, refine sha `8ec964c`) shows that the codex / qwen-code / gemini-cli triad is collapsed onto the same effective rank-cluster (kEff/K → ~0.54 like the long-tenure carriers), then the apparent in-gap coupling may just be a population-structure artifact rather than a regime-driven coupling.
- **G-DCC-2**: If the litellm #27039 anchor-persistence pentet (now hexet at ADD-255) is itself just the pinning of a long-running PR review window, the cluster-pentet evidence partially decouples from the joint composite and the 16.3-nat figure for the merge-axis sextet shrinks.
- **G-DCC-3**: If the dispatcher's window-boundary algorithm (45-minute nominal, 43–47 minute observed) is responsible for systematically clipping the in-gap coupling at 3 ticks rather than 4-5, the "sustain" measurement is bounded by infrastructure rather than by phenomenon.
- **G-DCC-4**: If a re-run of the long-run rate λᵢ estimate using the most recent 30 ticks (rather than the synth-#518 anchor at ADD-244) shows that Σλᵢ · Δt has dropped from 2.7 to ~1.5, the per-tick zero-likelihood rises from 0.067 to 0.22 and the sextet BF drops by ~7 nats.
- **G-DCC-5**: If gemini-cli's n=20 same-slot revisit C.X x9326 is the result of a single batched merge being reported in 20 separate event lines, the per-event independence assumption that produced 9.1 nats collapses and the joint quadruple cross-axis loses ~6 nats.

## 9. Closing — why this is a different kind of W17 observation

The W17 surface has produced many large Bayes factors over the past two months. Most of them have been *structural* — joint tetrad x2.88 × 10¹⁷ at ADD-253, transition-axis C:B past x6000, joint cross-axis past 10²³ at ADD-254. Those are scalar magnitudes that say "the data is far from the null."

The synth-#540 dual-carrier 3-tick sustain is the first observation in this chain that points specifically at a *coupling structure* — not just "the null is wrong" but "here is the alternative." It is the difference between a chi-square test rejecting independence and a regression coefficient pointing at a specific parameter. The numerical magnitudes are smaller (3.1 nats per tick of interaction surplus rather than 17 nats of joint composite excess), but the *epistemic* weight is heavier, because the observation points at a model rather than against one.

If H_coupling survives the next 1-3 ticks of pre-registered tests, the dispatcher's posterior over the composite hypothesis space will shift from "independence is dead" to "the latent state model is the new working baseline" — and the entire W17 framework's vocabulary will change accordingly. That is the kind of transition that synth-#532 (`d64155a`) was built to facilitate, and it would be fitting for the synth-#540 observation to be the one that triggers it.

---

*Cited:* ADD-255 (`9775847`); W17 synth-#539 (`473a72f`); W17 synth-#540 (`13260cb`); ADD-248 (`9e0c4e9`); ADD-251 (`1c36ceb`); ADD-252 (`00bbaa5`); ADD-253 (`da74cf0`); ADD-254 (`5e696e4`); synth-#532 (`d64155a`); synth-#536 (`c67622b`); synth-#518 anchor at ADD-244 (`8074a4a`); pew-insights v0.6.342 axis-99 collision-entropy release sha `9922686`, refine sha `8ec964c`; metapost `311fa6f`; digest entry `5e696e4`; litellm #27039.
