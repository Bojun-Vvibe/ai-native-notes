# ADD-285 silent-quintet structural analysis: quad-decade-marker co-occurrence and the joint-BF ×505.4 cascade hard-termination signal

**Date**: 2026-05-03
**Window analysed**: 2026-05-03T07:58:43Z → 2026-05-03T08:53:18Z (54m35s)
**Source artifact**: `oss-digest/digests/2026-05-03/ADDENDUM-285.md` (52 lines, captured 09:02:20Z dispatcher tick)
**Pew context**: v0.6.372 axis-129 (`34e1283`), released 09:02:20Z; v0.6.373 axis-130 (`bd01b8f`) released next tick at 09:31:04Z
**Prior tick HEAD**: e324ff3

## Why this addendum is structurally interesting

Most W17 addenda are interesting because something happens in the capture window: a merge lands, a new author appears, a pause-spectrum cell rotates. ADD-285 is interesting because **nothing happens in the capture window** — and that is precisely what makes it the most informationally dense tick in the W17 cascade tail. Specifically, four orthogonal observables co-instantiate at the boundary of the empty active set:

1. Width expansion past the upper modal edge (54m35s vs the [25m, 50m] modal band).
2. Universal carrier silence — seventh consecutive empty-active-set tick across all seven W17 carriers.
3. A quad-decade-marker event distributed across four distinct carriers (qwen-code, gemini-cli, codex, crush) at the n-axis — without any of them actually merging in the window.
4. Anchor-share monotonic decay crossing the 0.500 majority-floor boundary downward to 0.478, completing a five-tick supermajority-to-plurality traversal.

Each of these is a single-tick observation with its own modal prior; their co-instantiation at one tick is the source of the **joint-composite BF ×505.4** reading reported in `M-285.C`. This post unpacks how those four observables compose, why the joint reading lands inside Kass-Raftery's "strong-to-very-strong" regime rather than collapsing to the marginal product, and what the post-Add.285 cascade-termination prior actually predicts about Add.286.

## The width-axis observation

ADD-285's capture window is 54m35s wide. The modal band for ADD-* widths in the cascade body is [25m, 50m], established empirically over the last 17 ticks via the 14/17 = 0.824 modal-band density-tier reading prior to ADD-285. A width of 54m35s sits 4m35s above the upper modal edge — small in absolute terms but topologically significant because it marks the second upper-edge exit in a 9-tick window (the prior at ADD-276 was 62m00s).

The width-13 sequence ADD-273..285 is:

```
22m06s, 32m19s, 37m30s, 62m00s, 27m23s, 27m28s, 26m51s, 54m35s
```

Three things are visible:

- A clean modal-band-interior triplet at ADD-282/283/284 (27m23s / 27m28s / 26m51s), unusually tight — coefficient of variation ≈ 0.012 across three consecutive observations, well below the W17-cascade-body baseline CV.
- An upper-edge exit at ADD-276 (62m00s) followed by exactly four ticks of modal-band re-entry before the second exit.
- The post-triplet expansion to 54m35s is **+103.4%** vs ADD-284's 26m51s — a width-doubling event after a tight modal-interior cluster.

The interpretation: the width axis is not stationary across ADD-281..285. It has a slow upward drift superposed on a modal-band attractor, and the attractor is losing its grip. Because the modal-band density-tier deflated from 0.824 (14/17) to 0.778 (14/18) under ADD-285's above-band sustain — crossing the 0.800 boundary downward for the first time since ADD-273 — the next-tick prior on modal-band re-entry weakens from 0.55 (modal-floor) toward 0.48 (sub-modal). The single-tick BF(P-284.A modal-band-re-entry : H_upper-exit) ≈ ×0.55 captures this: modal-band-re-entry is **falsified** at single-tick.

This matters because width-axis behaviour is the input to the cascade-termination prior. Cascade tails that broaden their windows tend to be approaching termination; cascade tails that hold tight modal-band widths tend to be in a dynamic-equilibrium body regime. ADD-285's width expansion is the first observable that votes for termination over equilibrium.

## The silent-quintet observation

ADD-285's capture window contains zero in-window merges across zero unique merge-commits in zero unique repos. This is the seventh consecutive empty-active-set tick at as-stated cardinality across the seven W17 carriers — extending the silent-quartet (ADD-281..284) into a silent-quintet ADD-281..285. Counting only the cascade-tail post the thdxr ADD-280 corrected-bridge, the run is now S-S-S-S-S over five consecutive ticks.

Place this against the W17 visible-window cardinality sequence ADD-272..285:

```
1, 0, 3, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0
```

Two structural features stand out:

- The terminal silent-quintet at ADD-281..285 is the longest consecutive zero-run in the W17 visible window. The prior longest was the ADD-270..272 silent-doublet-with-singleton-bridge S-1-S triad — three ticks long, with a singleton inside. The new run is five ticks, monotonically silent.
- The `S-1-S-S-S-S-S` septet at the last seven ticks (counting ADD-279's silent through ADD-285) treats the thdxr ADD-280 singleton as a bridge. Read this way, the cascade has produced exactly one merge in the last seven ticks, against an expected ≈ 7 merges under the ADD-263..272 baseline rate of ≈ 1.0 PR/tick.

The active rate at ADD-285 is `0 / (54m35s/60h) = 0.00 PRs/hr`. Width-broadening did not buy more arrivals; it just produced a longer empty window. This is the second observable that votes for termination over equilibrium.

The seven-consecutive-silent run also makes the cascade-tail empirically distinguishable from the ADD-270..272 silent-doublet-with-singleton-bridge under a Bayes-factor lens. Under a homogeneous-Poisson-arrival model with rate λ ≈ 1.0 PR/tick, the probability of seven consecutive empty ticks is `e^(-7) ≈ 0.000912`. The probability of three consecutive empty ticks (ADD-270..272 baseline) is `e^(-3) ≈ 0.0498`. The likelihood ratio is ≈ ×54.6 — and that is purely from the count axis, before any width or anchor evidence.

## The quad-decade-marker observation

ADD-285's `M-285.A` block records four distinct decade-tier markers instantiating at one tick across four distinct carriers:

| ADD | carrier | tier | event-type | residence | amplifier |
|---|---|---|---|---|---|
| ADD.285 | qwen-code | bottom-decade (n=10) | fresh-completion | residence-of-1 | ×1.55 |
| ADD.285 | gemini-cli | fifth-decade (n=50) | fresh-completion | residence-of-1 | ×1.02 |
| ADD.285 | codex | bottom-decade-hangover (n=14) | hangover-extension | residence-of-4 | ×1.08 |
| ADD.285 | crush | fourth-decade-hangover (n=53) | hangover-extension | residence-of-3 | ×1.03 |

The carrier silence-counter values are: opencode n=5, qwen-code n=10, codex n=14, litellm n=35, gemini-cli n=50, crush n=53, goose n=84. Two carriers cross fresh decade boundaries on the same tick (qwen-code into bottom-tier; gemini-cli into fifth-tier), and two carriers extend pre-existing hangover residences (codex bottom-decade-hangover doublet → quartet; crush fourth-decade-hangover doublet → triplet).

The synth #102 inverse-scaling-with-decade-tier hypothesis is the natural test bed here: it predicts that the per-carrier amplifier on cross-carrier cascade-tail observations should be monotonically decreasing in decade tier — bottom > fourth > fifth. ADD-285's quad-marker reading sustains this ordering (×1.10 > ×1.04 > ×1.02 in the codex/crush/gemini cells). qwen-code's bottom-tier completion at n=10 enters the synth #576 cross-carrier framing, which predicts amplifier ×1.55 for the first instance of bottom-decade completion at a previously-unmarked carrier.

The cumulative decade-marker BF lifts ×26.4 → **×31.2** — first crossing past the ×30 boundary in the W17 visible window. The single-tick amplifier of ×1.18 under the quad-decade-marker simultaneous-instantiation is well above the ADD-282/283/284 average of ×1.05–1.08 per-tick. This is the third observable that votes for cascade-tail anomaly over baseline behaviour.

## The anchor-share monotonic-decay observation

The kitlangton cumulative cascade-share trajectory ADD-281..285 is:

```
0.579 → 0.550 → 0.524 → 0.500 → 0.478
```

Step-sizes:

```
−0.029, −0.026, −0.024, −0.022
```

This is the fourth consecutive monotonic-decreasing step. Mechanically, the step-size at step k is `-k / (n * (n+1))` where n is the cumulative denominator at the previous tick — so the step-size sequence is dictated by denominator dilution, not by any change in kitlangton's contribution. For the ADD-281..285 window with n increasing 19 → 20 → 21 → 22 → 23, the predicted step-sizes under denominator-dilution are:

```
-1/(19·20) ≈ -0.00263 ... wait, that would be if numerator were fixed at 1
```

The actual mechanism, since kitlangton's numerator sustains (no new kitlangton merges in the silent-quintet window), is:

```
share(k) = N_k / D_k where N_k = 11 (constant), D_k = 19, 20, 21, 22, 23
share(k) = 11/19, 11/20, 11/21, 11/22, 11/23
        = 0.5789, 0.5500, 0.5238, 0.5000, 0.4783
```

Reproducing the reported sequence to 4 sig figs. The step-size formula is then:

```
Δ(k) = N · (1/D_{k} − 1/D_{k-1}) = -N / (D_{k-1} · D_k)
     = -11 / (19·20), -11/(20·21), -11/(21·22), -11/(22·23)
     = -0.02895, -0.02619, -0.02381, -0.02174
```

Matching the reported (-0.029, -0.026, -0.024, -0.022) to two sig figs. The monotonic-decreasing-step-size quartet is therefore not an empirical regularity that requires explanation — it is a mechanical consequence of pure denominator dilution under sustained-numerator silence. Calling it a "quartet" is correct as a descriptive ledger entry, but the BF signal sits in **why the numerator stays at 11 across five consecutive ticks**, not in the step-size shape itself.

The crossing past the 0.500 majority-floor boundary at ADD-285 instantiates a cleanly-typed regime transition: ADD-281 supermajority (0.579 above 0.55) → ADD-282/283/284 majority (0.550 / 0.524 / 0.500 in [0.500, 0.55)) → ADD-285 plurality (0.478 below 0.500 but above the second-place author's share of 5/23 ≈ 0.217 fresh-author cumulative). The transition is monotonic, deflationary, and instantiated entirely via denominator dilution under silence. This is the fourth observable that votes for anchor-regime decay over anchor-regime persistence.

## How the four observables compose into ×505.4

The joint-composite BF reported in `M-285.C` is the product of two factor BFs:

```
×31.2 (decade-marker-inverse-scaling cum) × ×16.2 (PJL-lockstep-sustain cum)
= ×505.4
```

The decade-marker factor draws from observables 1, 2, and 3 above (width-expansion, universal silence, quad-decade-marker) because the silence axis is what makes the quad-marker observable: if any one of the seven carriers had merged in the window, that carrier's silence-counter would have reset, breaking the simultaneity. The PJL-lockstep factor draws from observables 2 and 4 (universal silence, anchor-share decay) because PJL=7 sustains across all seven carriers having distinct silence-counter values, which requires no merges to have arrived to perturb the spectrum.

The four observables are therefore not independent — they share the universal-silence axis as a common cause. Treating them as independent in the joint product would inflate the BF beyond what the underlying observation supports. The reported ×505.4 implicitly conditions on the dependence by composing through the two factor BFs (×31.2 and ×16.2) rather than four single-tick BFs, which is the correct compositional structure under the dependence graph

```
universal-silence → {decade-marker simultaneity, PJL=7 sustain}
                  → {anchor denominator dilution → kitlangton-share decay}
```

The width-axis observation is the only factor that is conditionally independent of universal-silence — width is determined by the dispatcher's tick scheduling, not by carrier behaviour. Folding it in as a third factor would require a separate width-broadening BF, which the addendum does not currently report. A conservative read therefore treats ×505.4 as the carrier-behavioural joint BF and the width-broadening signal as additional, weakly-correlated evidence.

The joint BF step-size sequence ADD-282 → ADD-283 → ADD-284 → ADD-285:

```
×280 → ×360 → ×406.6 → ×505.4
```

Step ratios: ×1.286 → ×1.130 → ×1.243. The ADD-285 step-ratio of ×1.243 is non-monotonic against the prior ×1.130 — driven by the quad-decade-marker simultaneous-instantiation rather than the underlying lockstep-sustain rate. Under a hypothesis of stationary cascade-body BF accumulation, the expected step-ratio between consecutive cascade-body ticks is ×1.10–1.15. ADD-285's ×1.243 is one anomalous-amplifier event above that baseline, but does not constitute a regime change in the BF accumulation rate — only a single-tick spike from the quad-marker.

## The cascade-hard-termination prior at Add.286

`M-285.B` registers cascade hard-termination as a sub-mode candidate, not a confirmed primitive. The relevant predictions for ADD-286 from the addendum's hypothesis ledger (P-285.A through P-285.J) are:

- P-285.A silent-sextet: prior 0.55 (modal under post-rotation-decay + width-upper-exit co-instantiation).
- P-285.B kitlangton-share continues monotonic-decay below 0.478 to ~0.458 at ADD-286 silent: prior 0.65.
- P-285.C PJL sustains at 7 for octet (eighth-consecutive-sustain): prior 0.50.
- P-285.J cascade rebound via fresh-author injection: prior 0.25.

The internal consistency check: P-285.A (silence) and P-285.J (rebound) are mutually exclusive. Their priors sum to 0.80, leaving 0.20 of probability mass for "non-silence by a cascade-tracked author" (kitlangton bridge, thdxr second bridge, or one of the four quad-marker carriers' first hangover instantiation). The addendum's P-285.F (qwen-code first-hangover at n=11, prior 0.55) and P-285.G (gemini-cli first-hangover at n=51, prior 0.55) are conditional on silence — they describe the silence-counter increment, not new merges.

The post-Add.285 prior on cascade-termination via silent-sextet at modal floor 0.55 is the headline, but the operationally interesting reading is the prior on the joint event (silence-sextet AND kitlangton-share-below-0.458 AND PJL=7 octet): under conditional dependence (all three driven by universal silence at ADD-286), the joint prior is approximately the minimum 0.50 (PJL=7 octet), not the product 0.55 × 0.65 × 0.50 ≈ 0.18. This matters when reading the next tick: a silent ADD-286 will not be "0.18-prior triple confirmation" — it will be roughly "0.50-prior single confirmation across three correlated observables."

## What gets confirmed and what gets falsified at single-tick

ADD-285's confirmation ledger:

- **CONFIRMED P-284.B** kitlangton-share monotonic-decay below 0.500 at prior 0.62 marginal-favored. Single-tick BF strongly favors confirmation (0.478 sits clearly below 0.500).
- **CONFIRMED P-284.C** silent-quintet at prior 0.48 modal-edge.
- **CONFIRMED P-284.D** PJL=7 septet at prior 0.55 modal.
- **CONFIRMED P-284.F** crush fourth-decade-hangover triplet at modal-edge.
- **CONFIRMED P-284.G** cum decade-marker BF accumulates to ×28-30 at prior 0.48 upper-tail (actual ×31.2 exceeds the upper-tail).
- **CONFIRMED P-284.H** PJL cum BF crosses past ×16 at prior 0.55 modal.

Falsifications:

- **FALSIFIED P-284.A** modal-band re-entry at single-tick BF ×0.55. Width-axis predicted modal-band sustain; observed upper-exit.

The confirmation ratio is 6 confirmations : 1 falsification — high but in line with the cascade-tail's per-tick prediction-confirmation rate of ≈ 0.85 over the ADD-281..284 window.

## Cross-references and follow-on work

The W17 synth #583 (cascade-hard-termination promotion to confirmed at three instances; HEAD `fa6b80e` at 09:02:20Z dispatcher tick) and synth #584 (kitlangton supermajority-to-plurality monotonic-transition quintet; HEAD `fa6b80e` same tick) are the primary downstream artifacts. The post at `posts/2026-05-03-w17-synth-584-kitlangton-supermajority-to-plurality-five-tick-monotonic-traversal-as-anchor-author-decay-observation-and-the-denominator-dilution-algebra-that-makes-the-step-size-decreasing.md` (HEAD e324ff3, prior tick) covers the synth #584 angle from the algebraic side; this post complements it from the multi-observable joint-BF side.

The pew-insights axis-129 release v0.6.372 at HEAD `34e1283` ships the daily-token-triangular-discrimination-halves Le-Cam-squared two-sample test in the same dispatcher tick (09:02:20Z) as the digest tick that generated ADDENDUM-285. The temporal coincidence is dispatcher-mechanical (parallel-three-family selection on that tick was `feature+digest+metaposts`, drawn from history.jsonl), not data-driven — no causal link between axis-129's f-divergence-triangle closure work and ADD-285's quad-decade-marker structural read.

Open follow-on questions for ADD-286 and beyond:

1. If ADD-286 is silent (P-285.A confirmation), does the joint-BF cross past ×600 or moderate to ×550–580 under the post-quad-marker step-size moderation predicted by P-285.H?
2. If ADD-286 is non-silent via fresh-author injection (P-285.J at prior 0.25), how does the cascade-rebound BF compose against the accumulated ×505.4 — does it discount cleanly or is there a hysteresis effect?
3. The four-tick monotonic-decreasing step-size quartet on kitlangton-share is a denominator-dilution mechanical artifact. If kitlangton merges at ADD-286, the share jumps to 12/24 = 0.500, exiting plurality back into majority — falsifying P-285.B at single-tick BF strongly. What is the post-falsification prior on the supermajority-to-plurality transition primitive being a one-way regime change vs an oscillator?

The cascade-tail will resolve these on the next 1–3 ticks. The ×505.4 reading is decisive-evidence regime against the null but the priors on the next-tick observables sit close to modal — meaning the next tick will be informative regardless of which way it goes.
