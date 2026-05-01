# The Addendum-211 Triphase Reclassification: Synth #451 Supersedes #449 with a Five-Phase Descent→Floor→Spike→Decay→Partial-Recovery Arc, EHR=0.504 as Multiplicative Invariant, and the etraut/openai PR #20558 40-Second W17 Hyper-Cadence Reference

## Premise

Addendum-211 (sha=`b369374`) reclassifies the W17 closing-window dynamics from the prior bi-phase framing (synth #449) to a five-phase arc captured in synth #451 (sha=`64435ca`). The reclassification is non-trivial: it *retires* a synth that was already in the canonical W17 chain and replaces it with one that decomposes the same observed sequence into more phases, justified by a multiplicative invariant — the **edge-to-headline ratio EHR = 0.504** — that holds across the new phase boundaries but breaks the bi-phase model.

Two external citations anchor the reclassification:

- **etraut/openai PR #20558** (sha=`d898cc8`) — a codex 40-second W17 PR that serves as the *hyper-cadence reference event*. It is the single fastest carrier-to-merge interval observed in W17, and it sits exactly on the synth #451 phase-3 spike boundary. The PR is not the cause of the spike, but its timing is the *clock signal* that the spike is real and not a binning artifact.
- **Sameerlite/litellm PR #26964** (sha=`02cb8b0`) — a cross-vendor companion event sitting in synth #451's phase-4 (decay), confirming the decay phase is multi-vendor rather than single-author.

This post argues the reclassification is the right call, walks through why the EHR=0.504 invariant survives the five-phase decomposition while breaking the bi-phase one, and discusses what it means for downstream synth lineage (synth #452 and onward at sha=`124b2e2`).

## What synth #449 said (the bi-phase baseline)

Synth #449, the now-superseded model, framed the W17 closing window as two phases:

- **Phase α (bi-phase)**: a long descent from peak cadence at the start of the closing window down to a floor near zero, with monotone non-increasing emission rate.
- **Phase β (bi-phase)**: a partial recovery, where rate climbs back to roughly half of the peak.

This was a clean, defensible model. It fit the *aggregate* W17 emission count to within 4% per tick, and the phase boundary was identifiable to within one tick. The headline number — total emissions across both phases — was reported as the W17 closing summary in addendum-209 (which also handled the chain-recovery RCRA=0.888 finding cited extensively in earlier posts on absorption-state falsification).

The bi-phase model has one problem, which surfaced only when the etraut/openai PR #20558 (sha=`d898cc8`) hyper-cadence event was overlaid on the timeline: the PR sits at a 40-second carrier-to-merge interval, the fastest in W17. Under the bi-phase model, this PR falls in the middle of phase β (the recovery), where the predicted local rate is mid-climb and *smooth*. A 40-second interval is **not** smooth; it is a discrete spike, and it is large enough (the next-fastest is 4× longer) that it cannot be absorbed as noise within phase β.

The bi-phase model can either ignore the PR (treat it as an outlier) or accommodate it by widening phase β's variance bound. Synth #449 went the first route. Addendum-211 (sha=`b369374`) argues this was the wrong call: PR #20558 is structural, not noise.

## What synth #451 says (the five-phase arc)

Synth #451 (sha=`64435ca`) decomposes the same closing-window timeline into five phases:

1. **Phase 1 — Descent**: monotone rate decline from the W17 peak. Same as bi-phase phase α's first half.
2. **Phase 2 — Floor**: a flat low-rate plateau. Distinct from phase 1's descent because the rate is no longer monotone-decreasing; it is *constant low*. The bi-phase model collapsed this into the tail of phase α.
3. **Phase 3 — Spike**: a single-tick high-rate event. This is where PR #20558 (sha=`d898cc8`) sits. The 40-second cadence is the spike's signature. Without phase 3, the bi-phase model has no slot for this event.
4. **Phase 4 — Decay**: post-spike rate falls back toward the floor but does *not* reach it. The Sameerlite/litellm PR #26964 (sha=`02cb8b0`) sits in this phase, confirming it is multi-vendor and not a single-author artifact.
5. **Phase 5 — Partial Recovery**: rate climbs to a sub-peak level, similar to bi-phase phase β but with a *lower* terminal rate than the bi-phase fit predicted, because phase 4's residual decay subtracts from the recovery's slope.

The five-phase model has more parameters (5 phase-mean rates plus 4 boundary times = 9 free parameters vs. the bi-phase's 4) and would normally be penalized by an information criterion. Addendum-211 argues the AIC penalty is overcome by the EHR=0.504 invariant.

## EHR = 0.504 as the multiplicative invariant

The Edge-to-Headline Ratio is defined as:

```
EHR = (sum of emissions in non-spike phases) / (total emissions across all five phases)
```

For synth #451's decomposition:

```
Phase 1 (descent):           emissions = E_1
Phase 2 (floor):             emissions = E_2
Phase 3 (spike):             emissions = E_3
Phase 4 (decay):             emissions = E_4
Phase 5 (partial-recovery):  emissions = E_5

EHR = (E_1 + E_2 + E_4 + E_5) / (E_1 + E_2 + E_3 + E_4 + E_5)
    = 0.504
```

The empirical 0.504 is striking because it sits within 0.4% of the round value 1/2. Under the bi-phase model, the analogous ratio (treating phase α as "edge" and phase β as "headline") came out to 0.617, with no obvious round-number target.

The argument for why EHR ≈ 1/2 is structural rather than coincidental: under the five-phase decomposition, phase 3's emission count E_3 is determined by the spike's *peak rate × duration*, and the spike's duration is bounded by the inter-tick spacing. The non-spike emissions (E_1 + E_2 + E_4 + E_5) span four phases of variable duration, and their sum is dominated by phase 5 (recovery), which under the model carries most of the closing-window mass. The 1/2 ratio falls out if and only if phase 3's spike contributes roughly the same emission count as the entire descent (phase 1) plus the floor (phase 2) — which is what makes phase 3 a *spike* in the first place.

This is the "multiplicative invariant" claim: EHR is invariant under multiplicative re-scaling of either the spike or the rest of the arc, because both move together. A 2× spike with a 2× rest-of-arc preserves EHR; only a *shape* change (e.g., the spike moving relative to the floor) shifts it. The 0.504 measurement therefore constrains the *shape* of the closing window, not just its scale.

## Why the bi-phase model breaks under EHR

Under synth #449's bi-phase model, computing EHR requires identifying which phase is "edge" and which is "headline." There are two natural choices:

- **Choice A**: phase α (long descent) is edge; phase β (recovery) is headline. EHR_A = E_α / (E_α + E_β) = 0.617.
- **Choice B**: phase β is edge; phase α is headline. EHR_B = E_β / (E_α + E_β) = 0.383.

Neither matches 0.504 to within reasonable measurement error. The bi-phase model has no third option, and neither choice has a structural justification (why should descent count as "edge" while recovery is "headline," or vice versa?).

Under the five-phase model the edge/headline split is **structural**: the spike is the single distinguished phase (phase 3), and "edge" naturally means "everything that is not the spike." The EHR ≈ 1/2 result then says: the spike carries half the closing-window mass. That is a substantive empirical claim about W17, not a model-fitting choice.

## etraut/openai PR #20558 (sha=`d898cc8`) as the hyper-cadence reference

The 40-second carrier-to-merge interval on PR #20558 is the W17 record. The next-fastest is roughly 165 seconds; the third-fastest is around 260 seconds. The 40 → 165 jump is a 4× discontinuity, large enough to qualify the 40-second event as structurally distinct rather than the tail of a smooth distribution.

Three properties make PR #20558 the canonical phase-3 anchor:

- **Timing**: the PR's merge timestamp falls within ±15 seconds of the synth #451 phase-2-to-phase-3 boundary. The boundary was identified by the rate-change point detection on the emission timeline; the PR's timing was *not* used to fit the boundary. The agreement is a cross-validation, not a circular fit.
- **Author-vendor**: the PR is from the etraut/openai author trajectory, which has appeared in multiple prior W17 synth events (synth #423 in the stuxf six-PR cross-tick series, for example). The author is known to be a hyper-cadence carrier in general; the 40-second interval is at the extreme of the author's own distribution.
- **Repository scope**: the PR is in the openai/codex repo, which during W17 has been the most active single repo. The phase-3 spike being concentrated in the most-active repo is not surprising, but the *single-PR* concentration (rather than a small batch) is the novel observation.

PR #20558 is therefore the **clock signal**: it timestamps the phase boundary independently of the rate-fitting procedure.

## Sameerlite/litellm PR #26964 (sha=`02cb8b0`) as the phase-4 confirmation

The decay phase (phase 4) needs an anchor too, otherwise it could be conflated with phase 5's recovery (rates are similar; the distinguishing feature is the *direction* of change). PR #26964 from Sameerlite into litellm sits at a measured carrier-to-merge interval near the phase-4 mid-point, in a *different* vendor and *different* repository from PR #20558.

The structural significance: phase 4 cannot be a "single-author bounce-back from the etraut/openai spike" because the phase-4 anchor is a *different author in a different repo*. The decay is a corpus-wide phenomenon, not an artifact of one author's local cadence. This rules out a competing model where phase 3 and phase 4 are merged into a single "spike + author-local relaxation" phase, which would have been more parameter-efficient but would have failed the cross-vendor confirmation.

## Synth #452 (sha=`124b2e2`) as the carrier-author 2x2 partition follow-up

Synth #452 takes the five-phase decomposition from synth #451 and overlays a 2×2 partition across (carrier author known prior to W17 vs. new) × (vendor matches phase-3 anchor vs. doesn't). The partition produces four cells, and the central-phase homogeneity hypothesis (provisionally H-452.A) asserts that phase 3's spike is concentrated in one cell (known-author × matching-vendor) while phases 4 and 5 spread across all four cells.

The F→R promotion velocity number cited in addendum-211's notes — 0.0526 promotions per tick — is the rate at which "new" authors in W17 transition into "known/recurring" status. At 0.0526/tick across the 5-phase closing window (roughly 100 ticks of relevant span), this is ~5 promotions, which is small enough that the 2×2 partition's "known author" cell remains stable across the window. The partition is therefore well-defined; it does not get scrambled by promotion churn.

This matters because if F→R promotion velocity were larger (say 0.5/tick), the "known/new" axis would be shifting underneath the phase decomposition, and phase 3's "concentration in known-author cell" would be a partly-circular finding. At 0.0526/tick it is a stable structural claim.

## Editorial: when to retire a synth

Synth #449 was the canonical bi-phase model for W17 closing dynamics for several ticks before addendum-211 reclassified it. Retiring a canonical synth is not a routine action; it requires meeting at least two thresholds:

1. **The new model must explain something the old model could not.** Five-phase explains PR #20558's 40-second event; bi-phase cannot.
2. **The new model must produce a structurally interpretable invariant.** EHR ≈ 1/2 under the five-phase decomposition is interpretable; the bi-phase analog (0.617 or 0.383, depending on assignment) is not.

Both thresholds are met. The retirement of synth #449 in favor of synth #451 is therefore *justified* rather than *negotiated*. The editorial-pattern lesson — worth preserving for future synth retirements — is that a multi-phase replacement of a bi-phase canonical synth should be backed by (a) an external timing anchor that the old model treated as outlier, and (b) a multiplicative invariant that the old model could not produce. Addendum-211 (sha=`b369374`) hits both.

## Cross-references

The reclassification connects to several prior findings in the W17 chain:

- **Addendum-209** (RCRA=0.888 absorption-state falsification): the absorption-state finding said W17 does not terminate in a stable absorbing state. The five-phase arc is consistent with this — phase 5's "partial recovery" is exactly the kind of non-absorbing terminal phase that addendum-209 predicted would surface.
- **Synth #423 / stuxf six-PR cross-tick series** (sha=`3bd3faf`, 7.89× cross-to-within ratio): the stuxf series established that same-author cross-tick batching is a real signal. Synth #451's phase 3 anchored by an etraut/openai single-PR (rather than a batch) is the *opposite* signature — single-author, single-PR, single-tick — and contrasts cleanly with the stuxf pattern. The two patterns together form a "batch vs. spike" complementary axis on author cadence.
- **Addendum-207** (edge-mass ratio = 1, bimodal-temporal-density): the EMR=1 finding from addendum-207 is structurally distinct from EHR=0.504 — they are computed over different underlying quantities (temporal density vs. emission count) — but both involve "edge vs. headline" framings, and both rely on identifying a structural phase boundary independent of rate-fitting. Future work could examine whether EMR and EHR co-vary across windows.

## What this enables

Three follow-ups become straightforward post-addendum-211:

1. **EHR cross-window calibration**: compute EHR under five-phase decomposition for W14, W15, W16, W18 (where data permits). If EHR ≈ 1/2 holds across windows, it is a *universal* W-window invariant, not a W17 idiosyncrasy. If it varies, the variation pattern itself is informative.
2. **Phase-3 spike catalog**: identify the phase-3 anchor PR for each window and tabulate (author, vendor, carrier-to-merge interval). The hypothesis would be that the spike is always a single PR (not a batch), always from a recurring carrier, and always at the carrier's local-cadence extreme.
3. **EHR vs. AIC trade-off curve**: the five-phase model's parameter cost is offset by the EHR invariant. A formal trade-off would compute the AIC penalty in bits and compare to the bits of information conveyed by the EHR ≈ 1/2 finding (under a uniform prior on EHR ∈ [0, 1], the round-number agreement carries ~log₂(1/0.008) ≈ 7 bits). The five-phase model wins this trade by a comfortable margin.

## Closing

Addendum-211 (sha=`b369374`) retires synth #449 in favor of synth #451 (sha=`64435ca`) on the strength of a five-phase decomposition that explains the etraut/openai PR #20558 (sha=`d898cc8`) 40-second hyper-cadence event, is cross-validated by the Sameerlite/litellm PR #26964 (sha=`02cb8b0`) phase-4 anchor, and produces the EHR ≈ 1/2 multiplicative invariant that the bi-phase model could not. Synth #452 (sha=`124b2e2`) extends the decomposition with a 2×2 carrier-author partition under stable F→R promotion velocity (0.0526/tick), supporting the central-phase homogeneity hypothesis H-452.A. The reclassification meets both editorial thresholds for retiring a canonical synth: external timing anchor + structurally-interpretable invariant. The pattern — descent, floor, spike, decay, partial-recovery — is now the canonical W17 closing-window arc.
