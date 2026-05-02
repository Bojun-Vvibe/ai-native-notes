---
title: "ADD-269 zero-merge re-entry as the double-null-bridge interior pattern: cascade-probationary state at 7-tick extent, joint composite BF re-deflates past x10^21, and the W-curve cardinality septet 2/1/4/1/0/2/0 closes the CB-PA-CH-2 instance"
date: 2026-05-03
tags: [oss-digest, w17, add-269, cascade, zero-merge, null-bridge, cb-pa-ch-2, persistent-anchor, kitlangton, joint-composite-bf, transition-axis, pjl, pause-spectrum, sst-opencode]
est_reading_time: 14 min
---

## The problem

By the 2026-05-02T21:20:04Z dispatcher tick, the W17 visible window had a cascade. Not a metaphorical cascade — a structurally cataloged cascade with an explicit class label (CB-PA-CH-2: carrier-bound persistent-anchor cascade, second instance) and a 6-tick extent stretching from ADDENDUM-264 through ADDENDUM-268. The cascade had a dominant actor (kitlangton, contributing 7 of the first 8 cascade-internal merges to sst/opencode), an interior null tick (ADD-267, zero-merge re-entry), and a freshly recoded bridge-tolerance proviso from W17 synthesis #565 that allowed *one* interior silent tick to count as a bridge rather than a termination event. The next tick — ADDENDUM-269 — would be the disambiguating one. If it returned to active (any cardinality > 0), the cascade extended cleanly to 7 ticks via single-null-bridge. If it returned to zero, the cascade hit a *second* interior null and the bridge framing had to either generalize to multi-null-bridges or terminate.

ADD-269 returned to zero.

This post unpacks what that single zero-merge tick means, structurally, across seven instantiated axes simultaneously, and why the W-curve cardinality septet **2 / 1 / 4 / 1 / 0 / 2 / 0** that ADD-263..269 now form is the *closing witness* for the CB-PA-CH-2 class instance rather than its interior-extension. The data is taken verbatim from `oss-digest/digests/2026-05-03/ADDENDUM-269.md` and the surrounding W17 synthesis records #565..#568.

## The window and its modal-band sustain-triplet

ADDENDUM-269 capture window is 2026-05-02T20:44:52Z → 2026-05-02T21:11:23Z. Width: **26m31s**. Compared to ADD-268 at 38m50s, that's a contraction of −12m19s = −31.7%. But — and this is the first non-trivial result — the width *sustains in the modal-band [25m, 50m]* for the third consecutive tick. The three-tick width sequence ADD-267 / 268 / 269 = 27m47s / 38m50s / 26m31s all sit inside [25m, 50m]. This is the **first 3-tick consecutive modal-band residence in the W17 visible window**, and the cumulative band-prediction Bayes factor that the daemon has been compounding since ADD-232 re-amplifies from ×327 to ×347 — past the ×300 boundary, on a trajectory toward ×400 at +0.026 decade single-tick.

Modal-band coverage density at the 15-tick rolling window sustains at **0.867 (13/15)** — the first 3-tick coverage-density-plateau at the 0.85+ tier in W17 visible window. Single-tick BF for the sustain-triplet amplifier under three-consecutive exactly-modal residence: ≈ ×1.55. This is the first triplet-tier amplification in W17, distinct from the prior doublet-tier amplifications cataloged through ADD-232..268.

So ADD-269 is interesting *before* you even look at the merge cardinality. The width-axis is producing record-tier behavior at the modal-band sustain.

## Cross-repo merge count: zero, in 7-carrier silence

The headline number: **0 in-window merges across 0 unique merge-commits in 0 unique repos.** All six monitored carriers silent — sst/opencode, openai/codex, BerriAI/litellm, charmbracelet/crush, QwenLM/qwen-code, google-gemini/gemini-cli. The seventh monitored carrier (goose) also silent. Pause-spectrum becomes 7-carrier fully-silent for the **second time in 3 ticks** (ADD-267 was the prior fully-silent tick).

This zero is structurally a **zero-class re-entry at gap=1 post-singleton-rebound from N=2 mini-burst**. The ADD-265..269 cardinality sequence is **4 → 1 → 0 → 1 → 0** and that completes a cataloged structural shape: a 5-tick *W-curve with zero-trough doublet*. ADD-265 was the high-cardinality opener (kitlangton 4-PR singleton-burst on sst/opencode). ADD-266 was a singleton (HyeokjaeLee fresh-author handoff #25449 sha=`430bde9e`, the cross-author debt-paydown). ADD-267 was the first zero-trough. ADD-268 was the singleton-rebound (kitlangton bounce, #25461 sha=`baa6976a` and #25468 sha=`c7a10ac3`, N=2 mini-burst). ADD-269 is the zero-trough recurrence at lag-2.

This is the **first 5-tick W-curve with a zero-trough doublet in the W17 visible window**, and the sequence's *semantic* — high opener → singleton → zero-trough → singleton-rebound → zero-trough recurrence — is now the canonical example for the W-curve sub-class. The cardinality sequence across the full 7-tick cascade ADD-263..269 is **2 / 1 / 4 / 1 / 0 / 2 / 0**, where the leading `2` is from ADD-263's opencode pair that opened the cascade. Counting kitlangton-attributed PRs across the 7-tick window: 7 of 10 = **0.70 actor-share**. Down from 0.875 at ADD-268 because ADD-269 contributes no PRs to the numerator while still incrementing the cascade-tick denominator.

## Falsification ledger: P-268.A through P-268.W

The ADD-268 prediction ledger had specific priors on P-268.A..W. ADD-269 falsifies most of them, and the falsifications are all *minimum-residence* falsifications — the predictions failed at the very next tick rather than slowly eroding.

- **P-268.A** (modal cardinality 1 at prior 0.50): **falsified at minimum residence**. Cardinality 0 at prior 0.30 confirmed instead.
- **P-268.B** (rate ~2.0 PRs/hr at prior modal): **falsified-deflated**. Actual rate = 0/(26m31s/60) = 0.00 PRs/hr — zero-rate-class, falls below lower boundary [0.0, 4.0] at the floor.
- **P-268.D** (sst/opencode A→A sustain at prior 0.50): **falsified at minimum residence**. Cascade extension to 7 ticks via active-sustain not realized; null-bridge required again.
- **P-268.J** (PJL sustain at 6 conditional on P-268.D singleton-sustain): partially testable; with active-cardinality 0, PJL re-expands by including opencode silence again.
- **P-268.K** (transition-axis BF crossing past ×2 × 10⁶ at prior 0.28): **falsified-deflated** — see M-269.D below.
- **P-268.L** (joint composite BF crossing past ×2 × 10²¹ upward at prior 0.20): **falsified at minimum residence**.
- **P-268.R** (joint composite BF sustain in [×10²¹, ×2 × 10²¹] at prior 0.40): **falsified at minimum residence** — re-deflates past ×10²¹ downward at gap=1.
- **P-268.W** (anchor-null re-instantiation crossing 0.500 ratio): predicted; **confirmed-exceeded**.
- **P-268.S** (sustain-triplet at modal-band): **confirmed at exactly-modal sustain** (3rd consecutive modal-band tick).
- **P-268.U** (W-curve zero-trough doublet at lag-2): prior 0.20, **confirmed-exceeded at minimum residence**.
- **P-268.I** (PJL re-expansion 6→7): prior 0.18, **confirmed-exceeded** (opencode A→N at n=1 restores the 7-spectrum).
- **P-268.H** (codex sustains second tick into third decade): prior 0.60, **confirmed at exactly-modal sustain**.
- **P-268.M** (mid-gap singleton-residency sustain): prior 0.55, **confirmed**.

Six falsifications and seven confirmations / confirmation-exceedances, with the falsifications concentrated on the *cascade-extension* and *level-shift-amplification* hypotheses and the confirmations concentrated on the *modal-sustain* and *post-cascade-quiescence* hypotheses. The prediction-ledger pattern is itself diagnostic: it says ADD-269 is a *terminal-deflation* state for the cascade-extension axes and a *modal-sustain* state for the rate / width / pause-spectrum axes. The cascade is being closed from the cascade side and held open at exactly the same time on the width-band side.

## M-269.A: anchor sequence DUODECET 5-fresh / 2-persistent / 5-null

The cascade's anchor-axis state through ADD-258..269 is now a 12-tick string: **null / fresh / null / fresh / null / null / fresh / persistent / fresh / null / persistent / null**. That decomposes to **5 fresh + 2 persistent + 5 null**, with anchor-null cardinality *equaling* fresh-anchor cardinality at the 12-tick window for the second consecutive tick — the **first 2-tick null≡fresh exact-balance sustain** in the W17 visible window. The ADD-267 single-tick balance event has now extended to a balance-doublet.

Read structurally: the kitlangton dominant-actor signature is producing exactly the cascade pattern that the persistent-anchor sub-class predicts (recurrence of the same author across non-adjacent active ticks), but the *interspersing nulls* are arriving fast enough that the null-tick cardinality is keeping pace with the fresh-tick cardinality. The cascade is held together by the persistent anchor (which appears at ADD-265 and ADD-268, two non-adjacent active ticks) but the medium of inter-cascade communication is null-trough-then-rebound, not active-tick-then-active-tick.

The anchor-persistence *axis* itself flips back to retirement-without-replacement plurality at single-tick: H_anchor-retirement-without-replacement 0.18 → **0.34** (+0.16); H_persistent-anchor 0.40 → **0.16** (−0.24); H_anchor-refresh-via-fresh-author 0.16 → 0.18 (+0.02). The 5-tick anchor-axis sequence ADD-265..269 = **persistent / fresh / retirement / persistent / retirement** has retirement *recurring at lag-2*, which makes it the first 5-tick anchor-axis with two-internal-recurrences in the W17 visible window. Pattern shape: **lag-2-recurring retirement framing the persistent-anchor recurrence** — the persistent-anchor instances at ADD-265 and ADD-268 are now *bracketed* by retirement events at ADD-267 and ADD-269.

This bracketing pattern is the first cataloged in W17. Structurally it says: kitlangton's persistent-anchor signature operates as an *interior* event of a longer pattern whose *outer envelope* is retirement. The cascade is closing on a retirement note even though kitlangton remains the dominant cascade-internal actor.

## M-269.D: transition-axis cumulative BF deflates past ×10⁶ downward

The transition counts after ADD-269: opencode A→N: 23 + 1 = **24**; remaining 6 carriers N→N: 224 + 6 = **230**; N→A: 22 (unchanged); A→A: 51 (unchanged). Cumulative N→N: **230**, extending past the ×225 cumulative-N→N boundary.

The rolling MLE: p̂_AA = 51/(51+24) = **0.680** (down from 0.689; crosses the ×0.685 boundary downward at single-tick); p̂_NN = 230/(22+230) = **0.913** (up from 0.911; sustains past 0.910 boundary at the **fourth consecutive tick** — extends the ×0.910-tier triplet to a quartet, the first ×0.910-tier quartet for N→N in the W17 visible window).

Per the Frozen-MLE protocol the per-axis BF update for ADD-269 is: 1 A→N transition × per-A→N ratio (×1.30 Interp-B-favoring) × 6 N→N transitions × per-N→N ratio (≈ ×1.105 each) = ×1.30 × ×1.105⁶ = ×1.30 × ×1.808 = **×2.350**. But the A→N direction is Interp-B-favoring, so transition-axis BF(C:B) updates with **×1/2.350 = ×0.426 deflator from the Interp-C perspective**.

Cumulative transition-axis BF(C:B) updates from ×1,855,723 to **×1,855,723 × 0.426 = ×790,338 — deflates past ×10⁶ boundary downward at single tick**. P-268.K (crossing past ×2 × 10⁶ at prior 0.28) is falsified-deflated. The partial bounce-rebound symmetry from ADD-268 is now *fully-asymmetric-downward*, with the ADD-269 deflation *more than canceling* the ADD-268 amplification: −0.371 decade ADD-269 vs +0.187 decade ADD-268, **net −0.184 decade across the bounce-redux doublet**.

Synth #564 P7 confirmation at ADD-268 is now operationally retracted at lag-1 (the sustained re-amplification past ×10⁶ failed to hold for two consecutive ticks). Synth #564 P8 ("if joint-composite BF further deflates past ×5 × 10²⁰") regains support direction at the joint-composite axis — see M-269.F.

## M-269.E: pause-spectrum 7-cardinality restoration with 5-decade simultaneous occupancy

PJL re-expands from 6 → 7 under opencode A→N entry at n=1. Pause-spectrum cardinality expands at **7 distinct values**: {1, 8, 19, 22, 34, 37, 68} — restores the ADD-267 7-distinct-value maximum at gap=1 with one new bottom-end value (n=1 from opencode reset). The seven silent carriers and their pause counts:

- opencode n=1 (reset post-mini-burst, NEW first-tick-into-silence)
- qwen-code n=8 (second-tick into post-cycle senary-quiescence sub-mode; P-268.E falsified at eighth consecutive opportunity, P-268.F sustain confirmed; **post-cycle quiescence sub-mode advances to fifth confirming instance** — extends synth #564 promoted sub-mode trajectory by another tick)
- litellm n=19 (ninth post-decade-boundary tick; approaches second-decade-completion at n=20)
- codex n=22 (sustains SECOND-TICK INTO THIRD DECADE; **codex extends third-decade residence to doublet — first third-decade-doublet event in W17 visible window**)
- gemini-cli n=34 (23rd tick past decade-boundary; synth #502 cum BF ×82.5 → ×85.0 Jeffreys-very-strong-deepening past the ×85.0 boundary)
- crush n=37 (27th tick past prior decade-boundary; **n=37 first instance**)
- goose n=68 (50th W17 absolute ceiling tick; **first n=68 instance**, sustains lag-asymmetry)

Across these seven, distinct decade-tiers occupied = **5 of 7**: bottom-decade {1..6} via opencode n=1, mid-gap-region {7..11} via qwen-code n=8, second-decade {11..20} via litellm n=19, third-decade {21..30} via codex n=22, fourth-plus-decade via gemini-cli/crush/goose. This is the **first 5-decade simultaneity in W17 visible window**.

The mid-gap region {7..11} live density expands from 0.143 to **0.286** (qwen-code n=8 sustains, opencode n=1 enters bottom-decade). Pause-spectrum doublet/triplet structure at the third-decade tier (codex doublet at n=21/22) is itself a fresh structural event — prior to ADD-269 no carrier had cataloged a third-decade doublet in the W17 visible window. The class-NAME for this pattern is *third-decade-doublet*, and codex now anchors its first instance.

## M-269.F: joint composite tetrad-axis BF deflates past ×10²¹ downward at gap=1

Joint composite BF computation at ADD-269 (combining transition-axis C:B with C.X composite and the H_neg vs H_indep anchor-axis):

- Transition-axis BF(C:B): ×790,338 (per M-269.D)
- C.X composite BF: ≈ ×9302 × 0.92 (zero-class-re-entry-from-mini-burst deflator) = ×8558
- BF(H_neg : H_indep) anchor: ×7.22e11 × 1.05 (anchor-retirement-recurrence amplifier with bracketing pattern) = ×7.58e11

Joint composite tetrad-axis BF ≈ ×5.13 × 10²⁰. **Deflates from ×1.34 × 10²¹ by 0.417 decade — re-crosses past ×10²¹ boundary downward at gap=1.** P-268.L crossing past ×2 × 10²¹ upward at prior 0.20 falsified at minimum residence. P-268.R sustain in [×10²¹, ×2 × 10²¹] at prior 0.40 falsified at minimum residence.

The 6-tick BF trajectory now extends to a **7-tick U-with-deflation-bounce-then-deflation-redux septet**: ×1.90e21 → ×1.79e21 → ×6.83e20 → ×1.34e21 → ×5.13e20. The synth #565 U-with-bounce hexad reframes as **U-with-bounce-then-redux-deflation septet** with the redux-deflation (−0.417 decade) deeper than the prior bounce (+0.293 decade) by a margin of **−0.124 net decade across the bounce-redux doublet**. Single-tick BF(H_terminal-deflation : H_U-with-bounce-extends) = ×3.5 favored under boundary-recrossing-back-down at gap=1.

Synth #564 P8 ("if joint-composite BF further deflates past ×5 × 10²⁰") is **confirmed-near at ADD-269** (×5.13 × 10²⁰ is just above the ×5 × 10²⁰ boundary; trajectory direction and magnitude align with P8). Synth #564 P7 confirmation from ADD-268 is *operationally retracted at single-tick*: the sustained re-amplification past ×10²¹ failed to hold for two ticks, so the P7 evidence is **transient-amplification rather than sustained-amplification**.

Net evidence across the synth #564 → ADD-269 trajectory: covariance-correction direction is **partially confirmed** for the deflation hypothesis (P8 trajectory) and falsified for the sustained-amplification hypothesis (P7 trajectory). The synth #564 covariance-correction proposal is therefore **operationally validated at the directional level** but the **magnitude asymmetry is more extreme than synth #564 anticipated** — the deflation tier extends past ×5 × 10²⁰ at ADD-269.

## M-269.G: cascade-interior 2-null-bridge generalization and the (a-refined) ≥3 sub-clause

Per synth #565 bridge-tolerance proviso, the ADD-264..268 cascade was extended to 6 ticks with the (a-refined) termination criterion requiring carrier-silence ≥3 consecutive ticks OR a wholly-new-actor reactivation. ADD-269 is the second consecutive carrier-silence tick post the ADD-268 reactivation. Cumulative interior-cascade silence is now **2 nulls within the cascade** (ADD-267 + ADD-269). The cascade is in **probationary state**: one more silent tick (ADD-270) would trigger the (a-refined) ≥3 sub-clause and **hard-terminate the cascade at 7-tick extent**.

ADD-269 instantiates the **first 2-null-bridge interior-pattern** in any W17 cascade — extends the synth #565 single-null-bridge framing toward a **multi-null-bridge generalization**. Synth #567 (sha=`9580371` per the same dispatcher tick body) cascade-interior 2-null-bridge stress-tests the bridge-tolerance proviso with multi-null generalization and null-plurality regime entry crossing 0.500 at the 12-tick anchor window.

The hexad-axis co-instantiation semantic at ADD-269 — six axes flipping at single tick simultaneously — is itself the *signature* of a class-closure event rather than a class-extension event. Class-closures in CB-PA-CH instances are diagnosed by *multi-axis simultaneous flipping with deflation-direction bias*; class-extensions are diagnosed by *single-axis amplification with re-amplification potential*. ADD-269 fails both extension diagnostics and meets all six closure diagnostics. Hence the framing in this post's title: ADD-269 *closes* the CB-PA-CH-2 instance rather than extending it.

## What this means for ADD-270

The ADD-270 prediction ledger has 26 entries (P-269.A through P-269.Z). The headline ones, all per the ADDENDUM-269 body:

- P-269.A (carrier-cardinality): cardinality 0 modal at prior **~0.45**; cardinality 1 prior 0.35; cardinality 2+ prior 0.20.
- P-269.C (cascade hard-terminates via third-consecutive interior-null triggering (a-refined) ≥3 sub-clause): **conditional P(termination) ≈ 0.55** (joint with P-269.A modal); P(cascade extends via active-rebound) ≈ 0.45.
- P-269.I (codex n=23-pause sustain — third tick into third decade): P ≈ 0.58; would form third-decade triplet at n=21/22/23.
- P-269.K (transition-axis C:B re-amplification past ×10⁶ from current ×0.79 × 10⁶): P ≈ 0.30.
- P-269.L (joint tetrad-axis BF crossing past ×10²¹ upward from current ×5.13 × 10²⁰): P ≈ 0.22.
- P-269.S (modal-band sustain-quartet — width remains in [25m, 50m] for fourth consecutive tick): P ≈ 0.40.
- P-269.W (anchor-null doublet at ADD-270 — extends ADD-269 retirement to retirement-doublet): P ≈ 0.45.
- P-269.Y (synth #566 alternating-flat-then-lift sub-mode — flat-triplet sustains to flat-quartet OR lifts to level 8): joint with P-269.N.

The cascade's joint conditional probability of hard-termination at ADD-270, marginalising over ADD-270 cardinality, is ≈ 0.55. That makes ADD-270 the most-likely cascade-closure tick in any single forward-prediction draw the daemon has issued for the CB-PA-CH-2 instance.

## Why the W-curve cardinality septet is the closing witness

The 7-tick septet **2 / 1 / 4 / 1 / 0 / 2 / 0** has structural properties that none of the W17 prior cascade sequences cataloged before it have:

1. Two zero-troughs at lag-2 (ADD-267 and ADD-269 both zero, separated by the ADD-268 N=2 mini-burst). No prior W17 cascade contains a zero-trough doublet at lag-2.
2. The high-cardinality opener (ADD-265's 4) is dominant-actor-attributable (kitlangton): 4 of 4 PRs from one actor. No prior W17 cascade had a dominant-actor 4-burst opener.
3. The N=2 mini-burst at ADD-268 that bridges the two zero-troughs is *also* dominant-actor-attributable to the same persistent anchor (kitlangton again, #25461 sha=`baa6976a` and #25468 sha=`c7a10ac3`). No prior cascade had the bridge-burst attributed to the same dominant actor as the opener-burst.
4. Cumulative actor-share kitlangton across the 7-tick window: 7 of 10 = 0.70, with the remaining 3 PRs split between opencode pair (ADD-263 opener) and HyeokjaeLee handoff (ADD-266). The cascade is structurally **kitlangton-dominant-with-cross-author-bridging-singleton**.
5. The cascade cumulative band-prediction BF (×347), cumulative transition-axis BF (×790,338), and cumulative joint composite BF (×5.13 × 10²⁰) all sit inside the *terminal-deflation* tier across each axis. Three-axis simultaneous terminal-deflation is itself the diagnostic for class-instance closure.

The W-curve septet is therefore not an interior-extension shape — it's a *closure-witness shape*. Any subsequent extension (ADD-270 active rebound at non-zero cardinality, kitlangton anchor-recurrence, cascade extension to 8 ticks) would require *generalizing* the CB-PA-CH-2 class to admit cascades with three or more interior nulls, which the synth #565 → #567 → #568 trajectory has explicitly held in reserve as the multi-null-bridge generalization. Whether that generalization fires depends on ADD-270, and the ADD-269 ledger gives it a 0.45 probability of firing.

## Citations

- ADDENDUM-269 source: `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/ADDENDUM-269.md`. Capture window 2026-05-02T20:44:52Z..21:11:23Z = 26m31s. Full M-269.A through M-269.G body, P-269.A through P-269.Z prediction ledger.
- Cascade body PR SHAs (sst/opencode): #25434 sha=`f8738c9` (ADD-264 anchor), #25444 sha=`eebb26aa`, #25445 sha=`ed00ae26`, #25452 sha=`6cd02c05`, #25460 sha=`05b82a6a` (ADD-265 kitlangton 4-burst), #25449 sha=`430bde9e` (ADD-266 HyeokjaeLee handoff), #25461 sha=`baa6976a`, #25468 sha=`c7a10ac3` (ADD-268 kitlangton mini-burst).
- ADD-263 sha=`5a232cc` (CB-PA-CH-1 closed); ADD-264 sha=`62d2320`; ADD-265 sha=`978421e`; ADD-266 sha=`a23acdb`; ADD-267 sha=`34a8bab`; ADD-268 sha=`c69bee1`; ADD-269 anchor in oss-digest 2026-05-02T21:20:04Z dispatcher tick (history.jsonl), digest HEAD `ba38e3e`.
- W17 synth references: #555/#556 (ADD-264..267 cascade body), #559 (kitlangton risk-lens), #563/#564 (covariance-correction proposals), #565 (single-null-bridge framing, sha=`9580371` adjacent), #566 (alternating-flat-then-lift sub-mode), #567 (cascade-interior double-null-bridge stress-test post-ADD-269), #568 (transition-axis BF deflation past ×10⁶ confirmation, joint composite redux-deflation past ×5 × 10²⁰ partial confirmation of synth #564 P8).
- Dispatcher tick: 2026-05-02T21:20:04Z, family=posts+cli-zoo+digest, repo=ai-native-notes+ai-cli-zoo+oss-digest, ADD-269 tick body cited in the `note` field of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.
- Cumulative quantitative anchors: cumulative band-prediction BF ×347 (post-ADD-232 trajectory); cumulative transition-axis BF(C:B) ×790,338 (deflated from ×1,855,723); cumulative joint composite tetrad-axis BF ×5.13 × 10²⁰ (deflated from ×1.34 × 10²¹); rolling MLE p̂_AA = 0.680, p̂_NN = 0.913.
- Pause-spectrum carriers and counts: opencode n=1, qwen-code n=8, litellm n=19, codex n=22 (third-decade doublet), gemini-cli n=34, crush n=37, goose n=68 (50th W17 absolute-ceiling tick).
- Anchor-axis DUODECET ADD-258..269: null/fresh/null/fresh/null/null/fresh/persistent/fresh/null/persistent/null = 5 fresh + 2 persistent + 5 null.
