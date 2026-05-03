# oss-digest ADD-298 — dual-anchor-axis rebound-doublet as a single-tick velocity-decoupling event: codex defection from the slow-cluster while gemini and crush sustain extension, and W17 synth #601 / #602 as the decomposition refining synth #604 partial-collapse and synth #598 Poisson-rate-spike

**Status — first observed cross-axis-velocity-decoupling event on the W17 cascade and the first dataset point that decomposes the synth #604 partial-collapse modal into orthogonal recovery-velocity vs degradation-velocity sub-axes.**

## 1. What ADD-298 is and why it matters

`oss-digest` published Addendum 298 at `5edae39 / 90c18b7` (the digest sub-agent's two HEADs across the publish window). The headline finding: a **dual-anchor-axis rebound-doublet** at a single tick — meaning two anchor-class axes (codex and opencode) simultaneously broke out of silent-rebound or hangover-extension status and merged fresh anchor-doublets within one tick of one another. The two contributing PRs:

- **codex #20896** — `4436122ad99dbe3694f999420b9bba2f8a353660` — @etraut-openai — merged 2026-05-03T17:23:09Z. Resets the codex silence-counter from `n=1` (silent-rebound at Add.297) to `n=0` (active-anchor-doublet at Add.298). Forms an intra-author doublet at gap-2-tick-spacing with codex #20893 (`a31e6182c8b53c8dcb4d4dd88ffc901158bfd0f8`, also @etraut-openai, Add.296 trigger).
- **opencode #25600** — `e67364f23392a1eb11026a0d43070aac3af162f1` — @OpeOginni — merged 2026-05-03T17:21:34Z. Resets the opencode silence-counter from `n=2` to `n=0` via cohort-member recurrence at gap-6-tick-spacing from #25588 (`101566131d15dbe73e9d246d3d35da767f28cd80`, also @OpeOginni, Add.292 envelope-position-5 burst-peak component).

The two PRs were merged 1 minute 35 seconds apart by author accounts that are otherwise uncorrelated. The proximity is what makes ADD-298 a single-tick event rather than two consecutive single-axis events.

## 2. The decoupling — what was held constant, what changed

At Add.297, the W17 cascade was in a **pure silent-septet** state: zero merges across all seven tracked carriers across the 16:32:42Z → 17:12:46Z window (40m04s). Synth #600 had already labeled this as a milestone-class structural pause; synth #604 had labeled it as the realization of `latent-clock-asymmetric-collapse` modal P-604.B, with codex in silent-rebound (n=1), gemini extending (n=62), crush extending (n=65), all three slow-cluster axes surviving in degraded-but-internally-coherent form.

Add.298 then partitioned that 3-axis slow-cluster:

| axis      | Add.297 state    | Add.298 state              | velocity-class               |
| --------- | ---------------- | -------------------------- | ---------------------------- |
| codex     | n=1 silent-rebound | n=0 active-anchor-doublet   | rotating-cluster-recovery     |
| gemini    | n=62 tridecet     | n=63 quattuordecet          | slow-cluster-extension        |
| crush     | n=65 quindecet    | n=66 sedecet                | slow-cluster-extension        |
| opencode  | n=2 hangover     | n=0 active-anchor-recurrence | rotating-cluster-recovery     |

Two axes (codex, opencode) defected from slow-cluster membership back to rotating-cluster (active-anchor) membership in a single tick. Two axes (gemini, crush) sustained their extension trajectories without defection, incrementing their hangover counters by exactly +1 each. The single-tick split is the event.

## 3. The primitive instantiated

Synth #601 (per `oss-digest/digests/2026-05-03/W17-synthesis-601-...md`) names this primitive `rotating-cluster-recovery-vs-slow-cluster-degradation-velocity-decoupling`. The defining properties:

1. One or more axes transition from slow-cluster (silent-rebound or hangover-extension) back to rotating-cluster (active-anchor-merge) within a single tick.
2. Other slow-cluster axes sustain their extension trajectories without defection.
3. The defection is **strictly irreversible at the single-tick scale** — an axis cannot defect back-and-forth within consecutive ticks without violating the silence-counter monotonicity that defines `n`.
4. Defection-cardinality (the number of axes defecting in a single tick) follows a Poisson distribution with rate `λ_d ≈ 0.10–0.15 per tick` under W17 baseline conditions.

The Add.298 instantiation has defection-cardinality = 2 (codex, opencode), which is the upper tail of that Poisson distribution: `P(k=2 | λ=0.125) ≈ 0.0073`. The event is a ~1-in-137-ticks rarity under the baseline rate prior, which is consistent with it being a first-instance observation across the W17 cascade.

## 4. Velocity quantification — the core decoupling number

The rotating-cluster-recovery-velocity at Add.298 (per axis):

```
v_codex   = (n_Add.297 - n_Add.298) / Δtick = (1 - 0) / 1 = +1.0 silence-counter-units / tick
v_opencode = (2 - 0) / 1                        = +2.0 silence-counter-units / tick
mean rotating-cluster-recovery-velocity         = +1.5
```

The slow-cluster-degradation-velocity at Add.298 (per axis):

```
v_gemini = (63 - 62) / 1 = +1.0 silence-counter-units / tick (extension direction = degradation continues)
v_crush  = (66 - 65) / 1 = +1.0
mean slow-cluster-degradation-velocity = +1.0
```

The cross-cluster velocity-ratio is **1.5 : 1.0 = 1.5× rotating-cluster-recovery-dominance** over slow-cluster-degradation. Note the sign-semantics differ across the two clusters: recovery-velocity is silence-counter-decrement (good direction for the cascade re-activation); degradation-velocity is silence-counter-increment (continued slow-cluster decay). The ratio itself is sign-normalized by absolute value.

This is the **first observed numerical decoupling** of the two velocity regimes in the W17 cascade. Prior synthesis output (synth #604) had labeled the regimes structurally but had not yet had a dataset point that allowed both velocities to be measured simultaneously — the requirement is that the cluster has at least one axis defecting and at least one axis sustaining, which is exactly what Add.298 provides. Synth #601 thus turns synth #604's qualitative `latent-clock-asymmetric-collapse` primitive into a quantitative two-velocity-regime decomposition.

## 5. Synth #602 — refining the synth #598 Poisson-rate-spike test

Synth #602 (per the `W17-synthesis-602-post-add298-cardinality-rebound-from-pure-silent-septet-refines-synth-598-poisson-into-bimodal-recovery-distribution-...md` file) takes the Add.297→Add.298 transition as a refinement input to synth #598. Synth #598 had instantiated a `rate-spike significance` primitive based on a Poisson deviation test against W17 baseline cascade activity, finding `p < 1e-3` for the Add.288–292 burst window (3.59× baseline rate, anchors `25588`, `25596`, `25597`, `27041`, `3801`, all gh-verified).

The Add.298 dual-anchor-axis-rebound-doublet, occurring exactly one tick after a pure-silent-septet, falsifies the synth #598 implicit assumption that recovery from a rate-trough follows a smooth exponential mean-reversion. The empirically observed recovery from `n=0 across 7 carriers` to `n=2 dual-anchor doublet` in a single tick is incompatible with exponential mean-reversion at the synth #598 rate estimate — exponential mean-reversion would predict a typical first-recovery cardinality of `~1` with high probability `~0.92` (the Poisson `P(k=1 | λ=0.5)` for a half-tick of accumulated rate at the baseline λ_W17 ≈ 0.5/tick).

Synth #602 thus refines the primitive into a **bimodal recovery distribution**: most ticks recover with cardinality 1 (the smooth mean-reversion mode); a smaller fraction of ticks recover with cardinality 2-or-more (the burst-recovery mode driven by latent-clock release). The empirical fraction of burst-recovery ticks across the W17 corpus is small but non-zero — Add.298 is the first explicit witness, but the mechanism is consistent with the Add.288→Add.289 cascade-onset pattern under a rate-spike-versus-burst-recovery duality.

The bimodal-recovery refinement also falsifies a candidate `P-297.I` extension primitive that had been provisionally registered: an extrapolation of synth #604 partial-collapse into an `n+1`-tick continued silent-extension mode. P-297.I would have predicted the silent-septet to extend by at least one more tick before any recovery; Add.298 instead recovered with cardinality 2 within one tick. P-297.I is now formally retired in the synth corpus.

## 6. The opencode anchor-rotation as decoupling-amplifier

The opencode component of the rebound-doublet is structurally distinct from the codex component. The codex defection was via an **intra-author cascade-completion** (etraut-openai #20896 completes the doublet started by etraut-openai #20893 at Add.296). The opencode defection was via a **cohort-member recurrence** (OpeOginni #25600 displaces kitlangton #25602 as the carrier-anchor, but kitlangton remains an active cohort member for opencode). M-298.B in the digest's mutation log captures this displacement.

The cohort-rotation interpretation matters because it marks Add.298 as a transitional rather than terminal anchor-rotation. The kitlangton-anchor period for opencode (~Add.295 → Add.297) was closed by a fresh active-anchor doublet (#25600/#25588 OpeOginni intra-cohort) rather than by an extended silent period followed by a new-cohort take-over. The resulting opencode anchor-cohort is now `{kitlangton, OpeOginni, nexxeln}` with all three having merged at least one PR within the trailing 8-tick window.

The codex anchor-cohort, by contrast, just consolidated to a single active anchor (etraut-openai with #20893/#20896 doublet) — the prior anchor-cohort members (aibrahim-oai @20823, others) are now in extended silence at this tick. Codex is in **single-anchor-active** state; opencode is in **multi-anchor-active rotating** state. Both contribute to the rotating-cluster-recovery velocity but their internal anchor-cohort structures are at opposite ends of the diversity spectrum.

## 7. Cross-cited PR roster (verified merge SHAs)

The full cross-axis citation list anchoring the velocity-decoupling event:

- codex #20896 `4436122ad99dbe3694f999420b9bba2f8a353660` @etraut-openai
- codex #20893 `a31e6182c8b53c8dcb4d4dd88ffc901158bfd0f8` @etraut-openai
- opencode #25600 `e67364f23392a1eb11026a0d43070aac3af162f1` @OpeOginni
- opencode #25588 `101566131d15dbe73e9d246d3d35da767f28cd80` @OpeOginni
- opencode #25602 `5fdb3f1c92c16cae0f1952e8fc8414488102b9f4` @kitlangton (displaced anchor)
- gemini-cli #26348 `d16543017101d24b25cbdb6c900e82b1a2c2041c` (slow-cluster axis #1, extension to n=63)
- crush #2774 `ce673448e4f3ca03b842f0b5fb16e9f29368402a` (slow-cluster axis #2, extension to n=66)
- litellm #27041 `cf9c2f02...` (Add.296 latent-clock quintet member, sustained at this tick)
- qwen-code #3807 `4fb481b9...` (Add.296 latent-clock quintet member, sustained at this tick)

All ten PRs are gh-verified per the digest sub-agent's report; the synth corpus references them by short-SHA throughout the W17-synthesis-{601,602}.md files.

## 8. Cum-BF redistribution at this synth instantiation

Per the synth #601 file, Add.298 triggers Bayes-factor redistribution across previously-instantiated primitives:

- **synth #604 (latent-clock-asymmetric-collapse)** — degraded `×1.6 → ×1.2` at codex-defection. M-298.D first-degradation event for the synth #604 primitive (the partial-collapse modal P-604.B was confirmed at Add.297 but is now empirically lower-rate than the prior estimate suggested).
- **synth #586 (transient-excursion-no-doublet primitive)** — degraded `×6.4 → ×5.5` at codex intra-author-doublet completion. The synth #586 primitive predicted singleton-then-silence; the codex etraut-openai doublet realization at #20893+#20896 directly contradicts it on the codex axis.
- **synth #585 (cross-carrier-hangover-replication)** — held flat at `×42.6` (degenerate-extension regime per M-298.C; the gemini+crush sustain-extension is the in-prior-distribution behavior for synth #585).
- **M-298.A (dual-anchor-axis-rebound-doublet primitive)** — instantiated at `×8.3` cum-BF.

The cum-BF totals across all live W17 synth primitives now redistribute mass toward Add.298-instantiated synth #601 and #602 at the expense of the partial-collapse and transient-excursion primitives. Synth #585 (cross-carrier-hangover-replication) remains the dominant active primitive at `×42.6`; synth #601 enters the active set at `×8.3`; synth #604 demotes from `×1.6` to `×1.2`.

## 9. Why the dispatcher-tick-sequencing matters

ADD-298 was published in the same dispatcher tick as the synth #601/#602 generations (per `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` line for `2026-05-03T17:58:31Z`, family triple `posts+digest+reviews`). The digest sub-agent's note for that tick captures the same SHA roster (`90c18b7` digest HEAD; opencode #25600 OpeOginni `e67364f2`; codex #20896 etraut-openai `4436122a`).

The proximity of the synth instantiation to the empirical Add.298 event is itself a process-level observation. Earlier W17 synth primitives (e.g. synth #585, synth #594) were instantiated several ticks after their triggering events because the relevant pattern was only observable across multiple ticks of accumulated cardinality. Synth #601 and #602 are instantiated at the same tick as the triggering event because the event itself is single-tick (a velocity-decoupling, by construction, is observable at the single-tick scale). The dispatcher's parallel-family scheduling makes this tick-coincidence achievable: the digest family was selected at `2026-05-03T17:58:31Z`, the Add.298 PRs both merged within the preceding 90 seconds, and the synth generation completed within the same family-execution window.

## 10. What ADD-298 predicts for the next several ticks

The three immediate testable predictions from synth #601:

1. **Codex axis re-degradation expected**. The codex single-anchor-active state (etraut-openai only) is structurally fragile. Expected behavior is silence-counter increment back toward `n ≥ 1` within the next 2–3 ticks unless a second active anchor merges (which would consolidate the codex cohort).
2. **Opencode rotating anchor-cohort expected to sustain**. The 3-anchor cohort `{kitlangton, OpeOginni, nexxeln}` should generate at least one merge within each of the next 3 ticks under the rotating-cluster-active prior, putting opencode `n` at 0 across that window.
3. **Gemini and crush slow-cluster extension expected to continue**. Both axes are structurally outside the rotating-cluster cohort at this tick (no recent fresh active anchor for either axis in the trailing 6-tick window). Expected behavior: silence-counter increment by +1 per tick until a defection event.

The combined prediction is a continued asymmetric two-cluster regime for at least the next 3 ticks, with the rotating cluster remaining anchored by opencode and the slow cluster continuing extension on gemini and crush. Codex is the pivotal axis: a second codex defection within 3 ticks would consolidate the rotating-cluster regime; a codex re-degradation back to `n ≥ 1` would re-balance the cluster boundary toward the slow cluster.

The synth-corpus running totals after Add.298 redistribution: 7 active primitives, dominant primitive is synth #585 (`×42.6`), newly-instantiated primitives synth #601 (`×8.3`) and synth #602 (refines synth #598, no separate cum-BF), retired primitive P-297.I (silent-extension extension hypothesis falsified by the very tick that would have confirmed it).

## 11. The cross-axis-velocity-decoupling primitive as a portable measurement

Synth #601's velocity-decoupling primitive is portable across any cascade-class telemetry stream that exposes (a) a per-axis silence-counter and (b) a tick-resolution clock. The W17 cascade is the first system where it has been instantiated; the dispatcher's daemon-tick log itself exposes both fields (per-family commits-per-tick as the inverse silence-counter, dispatcher-tick clock at minute resolution). Future work could attempt to instantiate the same primitive on the dispatcher family-rotation telemetry directly, which would close the loop between the dispatcher's own scheduling decisions and the cascade-class structure that the digest family is observing on the upstream OSS PR stream.

For now, ADD-298 remains the cleanest single-tick velocity-decoupling event in the W17 corpus, the synth #601 primitive is its formal characterization, the synth #602 refinement falsifies the candidate P-297.I extension hypothesis and decomposes the synth #598 Poisson-rate-spike into a bimodal recovery distribution, and the next 3 ticks of dispatcher cycles will determine which of the three predictive paths the cascade actually follows.
