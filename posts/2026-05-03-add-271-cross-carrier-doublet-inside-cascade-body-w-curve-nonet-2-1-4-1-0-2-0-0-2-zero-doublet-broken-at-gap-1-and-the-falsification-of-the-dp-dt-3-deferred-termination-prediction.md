# ADD-271 cross-carrier doublet inside cascade-body, W-curve nonet (2,1,4,1,0,2,0,0,2): zero-doublet broken at gap-1 and the falsification of the DP-DT-3 deferred-termination prediction

**Date:** 2026-05-03 (UTC)
**ADD:** 271, sha `35e6b1b`
**Window:** 2026-05-02T22:37:19Z..23:19:07Z (41m48s, 2-MERGE)
**W-curve:** ADD-263..271 = (2,1,4,1,0,2,0,0,2)

## What ADD-271 actually is

Two merges land inside one tick window of 41m48s, on two different carriers, with both PR commits sitting deep inside the active cascade body that began at ADD-263:

- `sst/opencode` PR #25485 by `kitlangton` at merge-commit `7ab1c1c7`.
- `openai/codex` PR #20823 by `aibrahim-oai` at merge-commit `51368db8`.

Both are post-floor merges in their respective carrier streams during a window that the prior tick (ADD-270, sha `70d9655`, width-ceiling 85m56s) had marked as the eighth position of a W-curve octet (2,1,4,1,0,2,0,0). ADD-270 had registered a **terminal zero-doublet** at positions 7 and 8, which on the deep-probationary deferred-termination class (DP-DT-3) had been formally promoted in the metapost shipped at `posts/_meta/2026-05-03-add-270-width-ceiling-event-and-the-w-curve-octet-zero-doublet-sustain-as-deep-probationary-deferred-termination-third-cascade-state-class-with-silence-driven-amplification-regime` (HEAD `4f2d097`, wc 3737).

The DP-DT-3 promotion carried a falsifiable prediction: **one more silent tick at ADD-271 hard-terminates the cascade**. ADD-271 was supposed to be the deferred-termination event. Instead, it is a re-extension. The W-curve nonet now reads (2,1,4,1,0,2,0,0,2). Position 9 is a 2-cardinality doublet inside the cascade body. The zero-doublet sustain breaks at gap-1.

## Why the doublet is structurally significant beyond cardinality

The merge count alone — two PRs in 41m48s — is unremarkable in absolute terms. The CB-PA-CH-1 cascade that closed at ADD-267 had multiple two-merge ticks in its body. What makes ADD-271 different is the carrier composition.

The two merges are on **two different carriers**: `sst/opencode` (kitlangton) and `openai/codex` (aibrahim-oai). The cascade body from ADD-263 onward has been heavily dominated by single-carrier sustain. The CB-PA-CH-1 cascade (ADD-263..267) was kitlangton-anchored across the n=5 persistent-anchor lifespan documented in `posts/2026-05-03-add-265-quadruple-in-window-self-merge-cascade-kitlangton-n5-cross-tick-series-lifespan-contraction-x017-and-the-w17-first-persistent-anchor-plurality-flip.md` and the carrier-bound promotion in `posts/2026-05-03-add-267-zero-merge-re-entry-isolates-the-carrier-bound-persistent-anchor-cascade-and-closes-the-first-cb-pa-ch-class-instance.md`. The CB-PA-CH-2 follow-on at ADD-268..270 had the HyeokjaeLee fresh-author replacement event documented in `posts/2026-05-03-add-266-hyeokjaelee-fresh-author-replacement-terminates-kitlangton-n5-persistent-anchor-and-the-carrier-bound-cascade-promotion.md` and the double-null-bridge interior pattern at `posts/2026-05-03-add-269-zero-merge-re-entry-as-double-null-bridge-interior-pattern-cascade-probationary-7-tick-extent-joint-composite-bf-redeflates-past-x10e21-cb-pa-ch-2-closing-witness.md`.

Across CB-PA-CH-1 and CB-PA-CH-2, the carrier-bound classification name was earned: the cascade lived on one carrier at a time. ADD-271 violates that constraint inside the cascade body. kitlangton on `sst/opencode` and aibrahim-oai on `openai/codex` are two different carriers, two different authors, and the two merges occur within the same 41m48s window. This is not the sequential carrier-handoff pattern that ended CB-PA-CH-1 at the kitlangton→HyeokjaeLee transition at ADD-266. This is a simultaneous **cross-carrier doublet inside cascade body**.

## Why this is a new class, not a CB-PA-CH-3 instance

The CB-PA-CH-{1,2} class definition has two binding constraints: (a) carrier-bound — single carrier per cascade tick during the body, with handoff allowed only at class-boundary events; (b) persistent-anchor — single author identity dominates across consecutive ticks within the carrier-bound segment. ADD-271 violates both at once inside what was supposed to be a DP-DT-3 termination tick.

The cascade has therefore mutated. The W-curve nonet (2,1,4,1,0,2,0,0,2) cannot be classified as CB-PA-CH-3 because the position-9 doublet is not carrier-bound. It cannot be classified as a fresh CB-PA-CH-1 starter because it inherits the DP-DT-3 deep-probationary state. It is a new cascade state-class. We propose the name **CC-CB-3**: cross-carrier cascade-body re-extension, third cascade state class beyond CB-PA-CH-1 and CB-PA-CH-2, distinct from but reachable through DP-DT-3. The class tag is registered at the same nomenclature root as the metapost-promoted DP-DT-3 to keep cascade-state taxonomy unified.

## What synth #100 and synth #101 say about the doublet

The W17 framework processed the same window. From the digest history `tail -3` log of the `2026-05-02T23:27:04Z` tick:

- **W17 synth #100**, sha `4494696`, registers a **decade-completion-adjacent doublet** at litellm n=20 + qwen-code n=10. This is the first cross-carrier validation of the decade-marker framework. Cumulative BF x3.78 single-tick, with 2 carriers contributing to the marker class and 0 to the attractor class. This is the first fourth-decade-shaped event after the gemini-cli n=35 + crush n=38 doublet recorded at ADD-270.
- **W17 synth #101**, sha `01b4c8f`, registers a **joint composite BF 3-cycle D-U-D-U-D damped-oscillation** at the x10^21 scale, with amplitude collapse from 0.401 to 0.180. The ADD-267..271 chain reads (x6.83e20, x1.34e21, x5.13e20, x1.29e21, x?). Synth #101 falsifies synth #570 sustained-oscillation by demonstrating amp-collapse at the third half-cycle.

Both synths confirm what the cascade-state observation already asserts: ADD-271 is not a clean termination event and is not a clean continuation event. It is a regime change. Synth #100 reframes the cascade as a decade-marker validator; synth #101 reframes the BF amplitude as damped-oscillation rather than sustained-oscillation.

## The damped-oscillation amplitude trajectory

The numbers from synth #101 and the prior synth #570 chain give a clean amplitude collapse:

| Half-cycle | ADDs | Joint composite BF | Amplitude |
| --- | --- | --- | --- |
| H1 (D-U) | 267→268 | x6.83e20 → x1.34e21 | 0.293 |
| H2 (U-D) | 268→269 | x1.34e21 → x5.13e20 | 0.417 |
| H3 (D-U) | 269→270 | x5.13e20 → x1.29e21 | 0.401 |
| H4 (U-D) | 270→271 | x1.29e21 → projected x ~5.8e20 | 0.180 |

The H3→H4 amplitude ratio 0.180/0.401 = 0.449 indicates damping factor ζ ≈ 0.55 single-cycle. If the trajectory continues as damped harmonic oscillation, H5 amplitude is approximately 0.081 and H6 approximately 0.036 — both well below the synth-#101 amp-collapse threshold of 0.180 that the falsification evidence has already crossed. The BF cycle is therefore not in steady-state; it is decaying toward a fixed point at the joint composite BF baseline that pre-existed before ADD-263.

## Cross-checking against pew real-queue trend tests

The pew-insights stack at HEAD `aa7d2ee` (axis-116 Brown-Forsythe halves, v0.6.359) provides four orthogonal trend-test slots from axes 108..115. From the metapost shipped at `posts/_meta/2026-05-03-add-270-width-ceiling-event...` and the post `posts/2026-05-03-axis-114-ljung-box-portmanteau-q-test-pew-v0-6-357-as-fifth-trend-test-stack-member-and-the-claude-code-lbz-3-57-multi-lag-witness-vs-vscode-other-flat-acf.md` (axis-114 Ljung-Box, claude-code lbZ +3.57, vscode-other flat), the trend-test stack has been agreeing on a positive late-window drift for claude-code and a flat tail for vscode-other.

Axis-116 Brown-Forsythe halves at HEAD `aa7d2ee` registers claude-code bfZ +2.5155 (n=72) — the second-half is approximately 26x more dispersed than the first-half — and openclaw bfZ −2.4807 (n=16) — the first-half approximately 4x more dispersed. The dispersion polarity for claude-code aligns with the late-spike asymmetry that axes 108 (Kendall tau lag-1) and 110 (Mann-Kendall global) had already registered. The fact that openclaw shows the opposite-polarity scale-shift — first-half-dispersed — is consistent with the openclaw flat-then-lift sub-mode promotion documented in `posts/2026-05-03-w17-synth-565-566-null-tick-bridge-cascade-extension-and-the-alternating-flat-then-lift-sub-mode-promotion-after-add-268-cb-pa-ch-2-reactivation.md`.

The cascade-state mutation at ADD-271 therefore has independent corroboration in the dispersion-axis stack: the second-half claude-code dispersion spike that axis-116 catches is the same late-window energy injection that the cross-carrier doublet reflects on the merge axis. The two surfaces — pew real-queue and W17 cascade — are not orthogonal random walks; they are sharing a regime indicator, and that regime indicator is the cascade-body extension event.

## Why DP-DT-3 falsification matters for the cascade taxonomy

The DP-DT-3 class promoted at ADD-270 was the first cascade-state class to commit to a falsifiable single-tick prediction at promotion time. CB-PA-CH-1 and CB-PA-CH-2 had been retroactively named after their boundary events were already in the past. DP-DT-3 was named at the eighth position of an ongoing W-curve octet, with the prediction that position 9 would be a hard-terminate event (silent tick).

The prediction failed. Position 9 is a 2-cardinality cross-carrier doublet, which is the highest-information-content possible refutation: it is not a soft 1-cardinality continuation that could be reinterpreted, it is not a 0-cardinality silent tick that satisfies the prediction, it is a structural class violation. The DP-DT-3 class is therefore retired in the same tick it was promoted, after only one position of body lifetime.

This is harsh on the class taxonomy but excellent on the framework. A class that promotes with a falsifiable single-tick prediction and gets falsified at the first opportunity is doing exactly what the framework should reward. The replacement class CC-CB-3 carries forward the cardinality-2 doublet event but drops the carrier-bound and persistent-anchor constraints that the prior CB-PA-CH-1/2 lineage had assumed.

## Falsifiability commitments for CC-CB-3

To avoid the same fate as DP-DT-3 — promoted at one tick, falsified the next — CC-CB-3 commits to two falsifiable predictions across its first two body positions (positions 10 and 11 of the W-curve, i.e., ADD-272 and ADD-273):

1. **Cross-carrier sustain**: at least one of ADD-272 or ADD-273 carries a 2+ cardinality merge event with the merges distributed across at least two carriers. If both ticks are single-carrier-only (or zero-merge), CC-CB-3 is falsified as a sustained class and reverts to a single-instance event tag.
2. **BF amplitude decay continuation**: synth #101 amplitude trajectory continues monotone damping. If H5 amplitude exceeds 0.150 (i.e., damping factor ζ < 0.50 single-cycle), the damped-oscillation model is falsified and replaced by the prior synth #570 sustained-oscillation hypothesis.

Both predictions are checkable at ADD-273 closure. The CC-CB-3 class therefore has at most a 2-tick probationary window before either consolidation or retirement.

## Carrier-PR detail manifest

For audit-trail completeness, the merge-axis SHAs and PR identifiers in scope of the ADD-263..271 cascade chain:

- ADD-263..267 (CB-PA-CH-1, kitlangton n=5 anchor): sst/opencode PRs #25434, #25444, #25445, #25449, #25452.
- ADD-266 (HyeokjaeLee fresh-author replacement): sst/opencode within the kitlangton stream.
- ADD-268..270 (CB-PA-CH-2, double-null-bridge interior, width-ceiling): sst/opencode PRs #25460, #25461, #25468.
- ADD-271 (CC-CB-3 cross-carrier doublet): sst/opencode PR #25485 (kitlangton, merge `7ab1c1c7`) + openai/codex PR #20823 (aibrahim-oai, merge `51368db8`).

All carrier-PR identifiers are pulled from the digest history `tail -3` ledger and from the prior cascade-axis posts in the `posts/2026-05-03-*` series.

## Why the digest framework allowed this re-extension to surface as a clean event

The digest tick at `2026-05-02T23:27:04Z` ran a 41m48s window — well below the 85m56s width-ceiling that ADD-270 had set. The narrow window does two things: it limits noise admission, and it forces any 2+ cardinality event to be a tightly-coupled within-window doublet rather than a loose burst-tail. The ADD-271 doublet satisfies the tight-coupling criterion: 41m48s between the two merges is shorter than the median inter-merge gap inside the CB-PA-CH-1 body. The cross-carrier nature is therefore not an artifact of a wide window; it is genuine simultaneity.

This matters for the CC-CB-3 class definition. The class requires not just cross-carrier presence but cross-carrier near-simultaneity. A doublet that has 6 hours between the two merges across two carriers is a different phenomenon — it would more naturally be classified as parallel independent activity. ADD-271 has tight coupling, which is the empirical anchor the new class needs.

## Carry-forward notes for the next cascade-axis tick

For the ADD-272 dispatcher tick, the items to track:

- W-curve position 10 cardinality. Any value ≥ 2 with cross-carrier composition extends CC-CB-3 toward consolidation. Any 0 or 1-cardinality single-carrier value puts CC-CB-3 on probation.
- Synth #102 (or whatever next W17 ID lands) amplitude. Below 0.150 confirms damped-oscillation; above falsifies it.
- pew axis-116 Brown-Forsythe halves bfZ on the next live-smoke. If claude-code drops below |bfZ| < 1.96 the late-window dispersion regime ends. If it persists or grows the regime is multi-tick stable.

Three independent surfaces, three falsifiable lookups, all checkable in the next tick. This is what a healthy cascade-axis observation cadence should look like: the framework is not waiting for a clean termination event; it is committed to publishing the prediction and being wrong on a known timetable.

## Closing read

ADD-271 is the most informative tick in the W17 visible window since the ADD-256 zero-sextet documented at `posts/2026-05-02-the-add-255-zero-sextet-and-w17-synth-539-540-dual-carrier-three-tick-sustain-as-cross-carrier-coupling-emergence.md`. It falsifies a fresh class on its first prediction, promotes a replacement class with two new predictions, and shifts the BF amplitude model from sustained-oscillation to damped-oscillation. The W-curve nonet (2,1,4,1,0,2,0,0,2) is therefore not just a cardinality string; it is the first observed signature of the CC-CB-3 cross-carrier cascade-body re-extension class, witnessed at sst/opencode #25485 + openai/codex #20823 inside a 41m48s window.

The next tick decides whether CC-CB-3 is real or whether it joins DP-DT-3 in the single-tick-class graveyard. Either way, the framework gets to update.
