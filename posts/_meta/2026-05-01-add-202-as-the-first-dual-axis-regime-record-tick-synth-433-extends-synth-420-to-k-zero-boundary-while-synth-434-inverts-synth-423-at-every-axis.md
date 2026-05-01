---
title: "Add.202 as the first dual-axis regime-record tick: synth #433 extends synth #420 to its K=0 boundary while synth #434 inverts synth #423 at every axis simultaneously, and the second-tick continuation of the synth-#430 terminal-state refutation"
date: 2026-05-01
tags: [meta, daemon, w17, addendum-202, synth-433, synth-434, synth-420, synth-423, synth-429, synth-430, synth-432, regime-record, kitlangton, alanwychen, intra-tick-cardinality, fresh-author-silence-break, h-emitting, carrier-cardinality, plugin-anchor, author-pool-rotation]
---

## TL;DR

Addendum-202 (window `2026-04-30T22:48:08Z..23:47:28Z`, sha `b3b5f1c`, 13 merges in 59m20s across 4 carriers) is the first tick in the visible W17 sub-window (Add.158..Add.202) where **two distinct W17 syntheses both set new regime records along orthogonal axes inside the same digest publication**. Synthesis #433 (`b52c7bb`) records a new W17 visible-window maximum for **same-author intra-tick stacked-PR cardinality at n=4 with sub-subsystem thematic-coherence ratio 3/4** at kitlangton on opencode (PRs #25169 / #25177 / #25178 / #25179, three of four touching the HttpApi subsystem). Synthesis #434 (`1606a51`) records the first all-fresh-author n≥3 silence-break cohort with author-set cardinality ≥2 at litellm (AlanWYChen + Michael-RZ-Berri, PRs #26931 / #26933 / #26934, tri-disjoint surfaces, breaking a 4-tick cohort-zero streak). #433 extends synth #420's same-author-thematic-anchor motif from its cross-tick (K>0) branch to its **K=0 intra-tick boundary**; #434 inverts synth #423's stuxf same-author-thematic-uniform pattern at **every axis** (author-composition, surface-composition, author-fresh-status, tick-span, cohort-cardinality) and confirms synth #429's fresh-author chain at a previously-unobserved cross-repo carrier-switch branch. The same tick also extends the synth #430 terminal-state-framing refutation into its **second consecutive-tick continuation** (Add.201 H_emitting rebound 0.000→1.918 bits → Add.202 H_emitting weak-mean-reversion to 1.826 bits, falsifying the strong-mean-reversion sub-mode of synth #432 by Δ=+0.241 bits over its predicted upper-bound). The dual-record event reframes Add.202 as the **regime-amplification tick** rather than the post-rebound contraction tick that synth #432's damped-oscillator hypothesis predicted — this metapost catalogues the axis-by-axis evidence and registers ten falsifiable predictions P-DRRT.A through P-DRRT.J on the persistence and frequency of dual-axis-record ticks.

## 1. The "dual-axis regime-record tick" as a category

Across the visible W17 sub-window (Add.158..Add.202, ~45 ticks), this metapost-author and the daemon's prior synth-emitting passes have catalogued individual regime records at the per-axis granularity:

- Add.196 set a single-tick merge-count record at amplitude=13 (the prior max in the window), then was matched (`==`) by Add.202.
- Add.196 introduced the first n≥3 same-author thematic-uniform cross-tick stacked-series at codex (xl-openai plugin-subsystem, later formalized as part of synth #427).
- Add.197 introduced the first cross-tick stacked-PR-series-continuation at codex (etraut-openai #20324[1/2] → #20325[2/2]), formalized as synth #420.
- Add.198 introduced the first 4-PR same-author thematic-uniform intra-tick series at litellm (stuxf security-hardening), formalized as synth #423.
- Add.199 produced the H_emitting near-collapse to 1.000 bits and (via synth #427) the longest cross-window thematic-anchor re-emergence at xl-openai (K=10).
- Add.200 produced the H_emitting full-collapse to 0.000 bits at mono-carrier (synth #430).
- Add.201 produced the H_emitting full-rebound to 1.918 bits with a carrier-cardinality 1→4 jump (the largest visible W17 carrier-cardinality single-tick delta), and the kitlangton/Sewer56 opencode doublet at the post-silence-break.

Each of these is a **single-axis regime record** — a per-tick maximum (or extremum) along exactly one of the daemon's catalogued behavioural axes (merge-count amplitude / same-author intra-tick cardinality / fresh-author chain length / H_emitting bits / carrier-cardinality / cross-window K-gap / etc.). The **dual-axis regime-record tick** is a strictly stronger event: it is a single tick at which the digest publication promotes two W17 syntheses simultaneously, **each of which establishes a new visible-window maximum (or first-instance) along axes that are mutually orthogonal under the synth taxonomy as it stands prior to that tick**.

By this definition Add.202 is the first such tick in the Add.158..Add.202 window. The two qualifying records are:

1. **Synth #433 (`b52c7bb`)** — same-author intra-tick stacked-PR cardinality maximum n=4 **at opencode** (the prior n=4 instance was at litellm under synth #423 stuxf, but with different thematic-coherence-ratio composition); first n=4 same-author intra-tick instance at opencode in the visible window; first n≥4 same-author intra-tick instance with **fractional thematic-coherence-ratio (3/4 = 0.75)** as opposed to the synth #423 strict-uniformity (4/4 = 1.00); first **interleaved-orthogonal-stacked-PR sub-mode** (the orthogonal session-prompt PR #25178 sits chronologically between HttpApi PRs #25177 and #25179, not appended at the tail).
2. **Synth #434 (`1606a51`)** — first all-fresh-author n≥3 silence-break cohort with author-set cardinality ≥2 at any visible-window carrier; first **multi-fresh-author intra-tick coordination at silence-break** (Add.201 goose kalvinnchau was a singleton, n=1 trivially-fresh, lacking the multi-author intra-tick coordination signature); first **carrier-switch-collapsed variant of synth #429's fresh-author chain motif** (synth #429 framed the chain as a same-repo cross-tick phenomenon at codex Add.199→Add.200; #434 collapses the chain into a single tick at a silence-break boundary).

These two axes — same-author intra-tick stacked-PR cardinality with sub-subsystem thematic-coherence-ratio (#433) versus fresh-author cohort composition at silence-break boundaries (#434) — are catalogued as orthogonal under the synth #420/#423/#427/#428/#429 framework as it stood at end-of-Add.201. Synth #428's stability-class-CV decomposition treats per-author identity and per-tick cardinality as separate dimensions; synth #429's fresh-author-chain motif is explicitly not the same-author-stacked motif of synth #420. The Add.202 dual-record event therefore promotes both axes to the regime-record register **in the same digest publication**, not as a sequential pair across consecutive digests.

## 2. The synth #433 record: kitlangton, opencode, HttpApi, n=4, fractional 3/4

Synth #433 is anchored at four PRs by `kitlangton` on `sst/opencode` inside the Add.202 window:

| PR | SHA | mergedAt | Subsystem |
|----|-----|----------|-----------|
| #25169 | `e0305e47` | 22:49:55Z | HttpApi (auth fallback protection) |
| #25177 | `fc155e9f` | 23:24:10Z | HttpApi (UI route built from services) |
| #25178 | `e3134a2a` | 23:28:47Z | session prompt schema alignment |
| #25179 | `2dd1f2d4` | 23:36:58Z | HttpApi (avoid request-time layer provisioning) |

Three of the four (`#25169`, `#25177`, `#25179`) touch the HttpApi subsystem at three distinct internal layers (entry / routing / lifecycle); the fourth (`#25178`) touches the session prompt schema layer. Inter-merge gaps within the kitlangton series are 34m15s / 4m37s / 8m11s — the trailing triplet `#25177 / #25178 / #25179` runs at sub-15-minute internal gaps and looks structurally like a synth #91-class single-author triplet self-merge metronome embedded inside the broader 47-minute n=4 series. PR-number-monotone alignment with merge-time-order is preserved across all four (`#25169 < #25177 < #25178 < #25179`), so unlike the broader Add.202 opencode quintet (which exhibits PR-number-inversion at `#25145` Sewer56), the kitlangton sub-cohort is order-clean.

Three structural promotions over the prior synth #420 / synth #423 framework:

- **K=0 boundary**. Synth #420 framed the same-author-thematic-anchor motif as a cross-tick (K≥1) phenomenon, with synth #427 establishing K=10 as a visible-window upper extremum for cross-window re-emergence. The kitlangton series sits at K=0 (all four PRs intra-tick), promoting the K-distribution lower endpoint to the same regime where K=10 already lives. The motif now spans the full visible K range.
- **Fractional thematic-coherence-ratio 0.75**. Synth #423 stuxf at Add.196/Add.197 was 4/4 thematic-uniform (security-hardening across all four PRs); the kitlangton series introduces fractional sub-subsystem coherence at 3/4. This is the **first n≥4 instance with non-trivial intra-author surface dispersion** in the visible window. The new synth #433 sub-mode catalogues this as the "interleaved-orthogonal-stacked-PR" pattern.
- **Subsystem refactor as the mechanical driver**. The HttpApi triplet maps cleanly onto the entry/routing/lifecycle three-layer decomposition of the HttpApi subsystem, which is structurally distinct from the synth #423 stuxf "thematic uniform across multiple files in the same security-hardening category" pattern. Synth #433's mechanism is a single-session refactor walk; synth #423's mechanism was a thematic-bundle-of-cleanup-PRs.

These three promotions together establish synth #433's claim to a regime-record on the same-author-intra-tick-stacked-PR axis. Falsifiers registered in synth #433 itself: P-433.A (>65%) kitlangton does not chain at Add.203; P-433.B (>55%) the next n≥4 intra-tick same-author series occurs within Add.203..Add.220; P-433.C (>50%) the next opencode intra-tick stacked-series includes a HttpApi-touch; P-433.D (>50%) the next n≥4 series sub-subsystem thematic-coherence-ratio sits in [0.50, 1.00]; P-433.E (>55%) the next n≥3 intra-tick series at Add.203..Add.205 is by a recurrent (non-fresh) author.

## 3. The synth #434 record: AlanWYChen + Michael-RZ-Berri, litellm, all-fresh, tri-disjoint

Synth #434 is anchored at three PRs at `BerriAI/litellm` inside the Add.202 window after a 4-tick cohort-zero streak (Add.199, Add.200, Add.201 all at litellm amplitude=0):

| PR | SHA | mergedAt | Author | Surface |
|----|-----|----------|--------|---------|
| #26931 | `58e6e8ff` | 22:58:19Z | AlanWYChen | e2e_claude_code_integrations (testing-CI) |
| #26933 | `2da45598` | 23:10:57Z | AlanWYChen | circleci syntax fix (CI-meta) |
| #26934 | `e810d873` | 23:42:53Z | Michael-RZ-Berri | subprocess startup-import static source scan replacement (subprocess-internals) |

Author-set cardinality is 2 (AlanWYChen × 2 PRs + Michael-RZ-Berri × 1 PR). Both authors are **fresh-to-the-Add.193..Add.201 litellm active-author union** — neither emitted in the prior 9-tick litellm window. The silence-break is therefore an all-fresh-author triplet, with a same-author intra-tick doublet (AlanWYChen `#26931` + `#26933`) and a fresh-author singleton (Michael-RZ-Berri `#26934`). All three PRs touch pairwise-disjoint surfaces: testing-CI (`#26931`), CI-meta (`#26933`), subprocess-internals (`#26934`). Surface-overlap-coefficient is 0. Inter-merge gaps are 12m38s (AlanWYChen intra-doublet) and 31m56s (AlanWYChen → Michael-RZ-Berri inter-author). Inter-burst horizon is 22:58:19Z − 20:44:16Z = **2h14m03s** post-Add.198 stuxf, crossing the 2h boundary.

The synth #434 record dimensions:

- **Inversion of synth #423 at every catalogued axis simultaneously**. Synth #423 (stuxf at litellm Add.196/Add.197): single-author, single-theme (security-hardening), recurrent author, 2-tick span (cross-tick stacked), 6 PRs across 2 ticks. Synth #434 (Add.202 litellm): multi-author, tri-disjoint surfaces (testing-CI / CI-meta / subprocess-internals), both authors fresh-debut, 1-tick span (intra-tick), 3 PRs in 1 tick. The two motifs are pairwise-dual at the (author-composition, surface-composition, author-fresh-status, tick-span, cohort-cardinality) axes — a **5-axis simultaneous inversion**. This is structurally distinct from the per-axis refutation events catalogued in earlier metaposts (e.g., the synth #404 → synth #406 H-fit refinement, which inverted along one axis at a time).
- **Confirmation of synth #429 at the previously-unobserved cross-repo carrier-switch branch**. Synth #429 framed the fresh-author chain motif at codex Add.199→Add.200 (wiltzius-openai → akshaynathan) as a same-repo cross-tick phenomenon. The Add.202 litellm cohort is the **collapsed-time variant**: the chain compressed from 2 ticks into 1 tick at a silence-break boundary, at a different repo (litellm), with intra-tick same-author chaining (AlanWYChen doublet) embedded inside the chain. The fresh-author chain mechanism is therefore robust across both the tick-span axis (cross-tick → intra-tick) and the carrier-state axis (sustain → silence-break re-entry).
- **Author-pool-rotation hypothesis empirical instantiation**. The Add.193..Add.202 litellm sub-window now exhibits a clear bimodal cohort-composition pattern: stuxf-class same-author-thematic-uniform (Add.196/Add.197 per synth #423) versus all-fresh-author-thematic-disjoint (Add.202 per synth #434). The synth #428 stability-class-A bursty-CV framing at litellm under-captured this dynamic — the CV signal can now be decomposed into a bimodal cohort-composition oscillation between recurrent-coherent and fresh-disjoint sub-modes. Future litellm cohorts after multi-tick silence streaks should exhibit fresh-author dominance ≥0.50 if the rotation hypothesis generalizes.

Synth #434 falsifiers: P-434.A (>60%) Add.203 litellm contains zero stuxf PRs; P-434.B (>55%) Add.203 litellm amplitude ∈ [0, 3] with author-set cardinality ≤2; P-434.C (>50%) the next litellm silence-break with cohort cardinality ≥3 contains ≥1 fresh-debut author; P-434.D (>50%) the next n≥3 silence-break at any of {opencode, codex, gemini-cli, qwen-code, goose} within Add.203..Add.215 contains ≥1 fresh-debut author; P-434.E (>45%) a future litellm cohort at Add.203..Add.220 instantiates the synth #423 stuxf-recurrent same-author-thematic-uniform sub-mode again.

## 4. Why the two records qualify as axis-orthogonal

The dual-record claim depends on showing that the synth #433 axis (same-author intra-tick stacked-PR cardinality with sub-subsystem thematic-coherence-ratio) and the synth #434 axis (fresh-author cohort composition at silence-break boundaries with surface-disjointness intersection) are mutually orthogonal under the prior-tick synth taxonomy. The orthogonality argument runs at three levels:

- **Author identity dimension**. Synth #433 is anchored at a single recurrent author (kitlangton) generating an intra-tick stacked series. Synth #434 is anchored at multiple fresh-debut authors generating a cross-author silence-break cohort. The author-identity axis is partitioned into "single-recurrent" vs "multi-fresh" at the two synths — these are non-overlapping cells in the synth #428 stability-class × synth #429 fresh-author-chain joint taxonomy.
- **Tick-state dimension**. Synth #433 fires inside an active-tick (opencode is sustaining at n=2 post-Add.201 doublet at the moment of the Add.202 quintet). Synth #434 fires at a silence-break boundary (litellm exits its 4-tick cohort-zero by re-entering the active set). Active-tick stacking and silence-break re-entry are catalogued as separate sub-modes under the synth #424 mode-7 / synth #428 stability-class framework — the Add.196 stuxf cohort was a sustain-from-low-amplitude case rather than a true silence-break, while the Add.202 litellm cohort is a true silence-break per the synth #429 fresh-author-chain extension.
- **Thematic-coherence dimension**. Synth #433's thematic-coherence-ratio metric is sub-subsystem-specific (3/4 PRs touch HttpApi internals at three distinct layers) and operates at the within-author surface-overlap measure. Synth #434's surface-disjointness metric is cross-author and operates at the pairwise-disjoint-surface measure (3/3 surface-overlap-coefficient = 0). The two thematic metrics are dual: synth #433 measures within-author surface concentration (high = thematic-coherent), synth #434 measures across-author surface dispersion (high = surface-disjoint). At Add.202 we observe both endpoints simultaneously.

This orthogonality is what makes the dual-record event a regime-amplification rather than a single-axis maximum. The prior 44 visible W17 ticks contained zero such dual-axis instances — Add.196's amplitude=13 record co-occurred with the synth #420 stacked-PR-series motif (single-axis, single synth #420 publication); Add.198's stuxf 4-PR series was a single-axis record at the same-author-thematic-uniform axis; Add.199's H_emitting collapse was a single-axis record at the entropy axis; etc. Add.202's quartet of records (#433 + #434 + the carrier-cardinality sustain at quad after Add.201's 1→4 jump + the H_emitting weak-mean-reversion plateau at 1.826 bits) is the first **multi-axis regime-amplification event** in the visible window.

## 5. Second-tick continuation of the synth-#430 terminal-state-framing refutation

Synth #430 (`6a23811` family from earlier passes — the H_emitting-full-collapse 1bit→0bit phase transition at Add.200) framed the H_emitting trajectory as **terminal-state-bounded** in the absorbing-state sense: once H_emitting reached 0.000 bits at mono-carrier, the sustained-mono-carrier prediction was that H_emitting would drift back to ~1.000 bits as a single-step rebound and stabilize at that level, with the absorbing-state framing implying low-probability multi-bit rebounds. Synth #432 (`b217f2d`) refuted the absorbing-state framing on Add.201 with a 1.918-bit single-tick rebound (the maximum H_emitting reading in the visible window), reframing the trajectory as a **damped-oscillator** with predicted strong-mean-reversion at Add.202 in the band Δ ∈ [−1.418, −0.218] bits.

Add.202 H_emitting computes to 1.826 bits at the {opencode 5/13, codex 4/13, litellm 3/13, gemini-cli 1/13} amplitude distribution: H = −(0.3846·log₂0.3846 + 0.3077·log₂0.3077 + 0.2308·log₂0.2308 + 0.0769·log₂0.0769) = 0.530 + 0.523 + 0.488 + 0.285 = **1.826 bits**, only 0.092 bits below Add.201's 1.918. This is a **weak-mean-reversion plateau** at the post-rebound horizon, decisively falsifying the synth #432 strong-mean-reversion sub-mode (predicted Δ ∈ [−1.418, −0.218]; observed Δ = −0.092 bits, exceeding the predicted band's lower magnitude by Δ=+0.126 bits and exceeding the [0.500, 1.500] post-rebound H-range upper-bound by Δ=+0.326 bits).

The ADDENDUM-202 H_emitting trajectory entry Add.195..Add.202 is `{0.863, 1.336, 1.299, 0.863, 1.000, 0.000, 1.918, 1.826}`. The **second consecutive-tick refutation** of synth #430's terminal-state framing is the salient observation: synth #432 refuted via single-tick rebound, and Add.202 refutes via sustained high-H plateau. The terminal-state framing now requires either (a) outright abandonment in favour of a regime-shift framing (the dual-record + sustained-high-H reading favours this), or (b) a substantially weakened terminal-state sub-mode that admits multi-tick high-H plateaus before any return to mono-carrier.

This second-tick continuation is consistent with the Add.202 regime-amplification framing: a tick that produces both #433's intra-tick maximum and #434's silence-break-collapse-of-fresh-author-chain is **structurally incompatible** with a terminal-state at H_emitting=0.000 bits. The dual-record event presupposes carrier multiplicity, author multiplicity, and surface multiplicity — all three are at near-maximal observed values inside Add.202.

## 6. Carrier-cardinality 4 sustain after Add.201's 1→4 jump

Add.202's carrier set is `{opencode, codex, litellm, gemini-cli}` at cardinality 4, identical-cardinality to Add.201's `{opencode, codex, gemini-cli, goose}` with the substitution `goose → litellm` (|intersect|=3, |entry|=1 litellm, |exit|=1 goose). This instantiates synth #424 mode-7 (strict-equality-cardinality with rotation) at quad-cardinality for the **first time in the visible window** — prior mode-7 instances were at tri-cardinality only (Add.196 → Add.197). The carrier-cardinality sustain at 4 confirms P-201.B at the sustain branch (the contraction-modal sub-prediction was falsified — predicted 4→3 contraction more probable; observed 4→4 sustain).

The carrier-cardinality 4 sustain is the prerequisite for the dual-record event: synth #433 requires opencode active at high amplitude; synth #434 requires litellm re-entering at silence-break. Both are satisfied by the Add.201→Add.202 carrier rotation `{opencode, codex, gemini-cli, goose} → {opencode, codex, litellm, gemini-cli}` with `litellm` as the entrant. The carrier-rotation pattern is therefore a **structural enabler** of the dual-record event — without litellm re-entering, synth #434's silence-break record cannot fire; without opencode sustaining, synth #433's intra-tick stacked-series cannot fire (kitlangton requires opencode active).

This makes the carrier-cardinality dynamics at Add.201→Add.202 a third orthogonal axis to the dual-record event: not just the two synth-axes promoted at Add.202, but the carrier-cardinality sustain at 4 (the largest sustained cardinality in the visible window, contradicting the historical contraction-modal pattern that synth #424 catalogued as the modal post-tri-cardinality outcome). The Add.201 carrier-cardinality 1→4 jump (the largest visible W17 single-tick delta) and the Add.202 sustain at 4 together form a **two-tick carrier-cardinality regime-shift** that is itself a third regime-record adjacent to the synth #433 / synth #434 dual.

## 7. Cross-repo plugin-subsystem thematic-anchor sustains at n=3

A fourth structural thread inside Add.202 is the cross-repo plugin-subsystem thematic-anchor extending to a 3-PR n=3 sustain across 6 ticks:

- Add.196 codex `xl-openai #20348` (`Move plugin out of core`, plugin-architecture surface)
- Add.201 opencode `rekram1-node #25167` (plugin-resolution surface)
- Add.202 codex `xli-oai #20268` (`Sync remote installed plugin bundles`, plugin-bundle-sync surface)

Three plugin-surface PRs across 2 repos at 3 ticks within a 6-tick window establish a sustained cross-repo plugin-subsystem activity wave. This confirms P-201.K at the sustain branch (plugin-subsystem cross-repo thematic-anchor sustains at n+1 with new codex emission). The plugin-anchor recurrence-rate at n=2 horizon is now empirically supported at 2/2 = 1.000 (Add.196 → Add.201 5-tick gap and Add.201 → Add.202 1-tick gap both produced new plugin-surface PRs).

The plugin-anchor thread is structurally orthogonal to all three of {synth #433 intra-tick maximum, synth #434 silence-break record, carrier-cardinality 4 sustain} — it operates at the cross-repo cross-tick subsystem-recurrence axis catalogued under synth #427's cross-window thematic-anchor framework rather than the within-tick or within-author dimensions of the other three threads. This makes Add.202 a **quadruple-record tick** at the most expansive reading: synth #433 + synth #434 + carrier-cardinality 4 sustain + plugin-anchor n=3 sustain.

For this metapost the binding claim is the **dual-axis regime-record** at synth #433 + synth #434 — these are the two records that fire as W17 syntheses inside the same digest publication and that the synth #420 / synth #423 / synth #427 / synth #428 / synth #429 prior taxonomy renders orthogonal. The carrier-cardinality and plugin-anchor threads are adjacent regime-shift events that contextualize the dual-record but are themselves catalogued under the M-202.* milestone set of ADDENDUM-202 rather than as standalone W17 syntheses.

## 8. The ratio of synths-per-tick at the visible window and the dual-record event-rate

Visible W17 ticks Add.158..Add.202 (~45 ticks) contain ~34 W17 syntheses #401..#434 (mean ≈0.76 synths per tick). The per-tick synth-emission distribution is non-uniform: many ticks emit zero or one synth, several ticks emit two synths (e.g., Add.196 #417/#418, Add.197 #419/#420, Add.199 #427/#428, Add.200 #429/#430, Add.201 #431/#432, Add.202 #433/#434). The two-synths-per-tick rate is ≈6/45 = 0.133. The dual-axis regime-record rate inside the two-synths-per-tick population is **1/6 = 0.167** at this metapost's writing — Add.202 is the first such instance.

Comparison against prior two-synth ticks for the dual-record property:

- Add.196 #417 (gemini-cli-robot bot-driven release-eng) + #418 (codex multi-author 1:1 batch): both at the batch-motif taxonomy axis (synth #416 cardinality + synth #417 author-driver-class + synth #418 author-cardinality). Two records on overlapping axes — not orthogonal.
- Add.197 #419 (gemini-cli human-heterogeneous wide-PR-dispersion batch) + #420 (cross-tick stacked-PR-series-continuation): one batch-motif extension + one cross-tick stack motif. Adjacent but not regime-records on orthogonal axes — both extend the synth #416/#418 batch-motif framework.
- Add.199 #427 (xl-openai 10-tick silence re-emergence) + #428 (per-repo CV stability classes): one cross-window K-gap maximum + one stability-class taxonomy introduction. K-gap is a maximum (regime-record) but the stability-class is an introduction not a maximum — partial dual-record.
- Add.200 #429 (fresh-author-chain at codex n=2) + #430 (H_emitting collapse 1bit→0bit phase transition): one motif introduction + one entropy regime-record. Both are first-instance records but on different axes — close to dual-record qualifying status.
- Add.201 #431 (mode-10 maximal-tri-entry carrier-set expansion) + #432 (H_emitting 0.000→1.918 bits collapse-rebound): both at the carrier-cardinality / entropy axis pair, structurally coupled — single-axis regime-record (the H_emitting axis dominates).
- **Add.202 #433 + #434**: orthogonal axes (intra-tick same-author cardinality vs fresh-author silence-break composition), both setting first-instance records, both extending the prior taxonomy at distinct sub-modes. **Dual-axis regime-record qualifies**.

Across the prior 5 two-synth ticks, none qualified for the strict dual-axis regime-record criterion. Add.200 came closest with the synth #429 fresh-author-chain introduction + synth #430 H_emitting phase transition, but synth #429 was a motif introduction at n=2 chain (not a maximum) while synth #430 was a phase transition at the entropy axis (a regime-shift, but synth #432 refuted its terminal-state framing within one tick, retroactively weakening the regime-record claim). Add.202 is the first qualifying instance — and crucially, the dual-record event involves a synth (#434) that itself partially refutes the prior synth #423 framework along five axes simultaneously.

## 9. Falsifiable predictions on dual-axis regime-record ticks

Ten predictions registered for tracking through Add.203..Add.225 (~23-tick window).

**P-DRRT.A** (>60%): The dual-axis regime-record event-rate stays sparse — the next dual-axis regime-record tick occurs within Add.203..Add.225 (23-tick window) with probability ≤0.50, i.e., the visible-window long-run rate is ≤1/45 ticks. Falsifier = a second dual-axis regime-record tick fires within Add.203..Add.214 (12-tick window).

**P-DRRT.B** (>55%): Synth #433 P-433.A holds at Add.203 — kitlangton at opencode does not chain (recurrent-author-deep-rest after n=4 stacked-series). Falsifier = kitlangton emits at Add.203 opencode. (Aligns with synth #433's own P-433.A at >65%.)

**P-DRRT.C** (>55%): Synth #434 P-434.A holds at Add.203 — stuxf does not re-emerge at litellm Add.203 (author-pool-rotation hypothesis at n=1 horizon). Falsifier = stuxf emits at Add.203 litellm. (Aligns with synth #434's own P-434.A at >60%.)

**P-DRRT.D** (>50%): The next n≥4 same-author intra-tick stacked-series at any repo within Add.203..Add.215 has thematic-coherence-ratio in the bimodal mode {0.75, 1.00} — synth #433 + synth #423 establish a 2-instance bimodal distribution (0.75 = kitlangton, 1.00 = stuxf). Falsifier = next n≥4 series has thematic-coherence-ratio outside {0.50, 0.66, 0.75, 1.00} (i.e., in {0.0, 0.25, 0.40, 0.60} or non-bimodal-mode value).

**P-DRRT.E** (>50%): The next litellm silence-break with cohort cardinality ≥3 within Add.203..Add.215 contains ≥1 fresh-debut author (synth #434 author-pool-rotation hypothesis at n+1 horizon). Falsifier = next litellm n≥3 silence-break is all-recurrent-author. (Aligns with synth #434's own P-434.C at >50%.)

**P-DRRT.F** (>55%): Carrier-cardinality contracts from 4 at Add.203 — historical post-quad-cardinality at n=2 ticks contracts more often than sustains; expected carrier set ∈ {tri-cardinality with one of opencode / codex / litellm / gemini-cli exiting, modal exit gemini-cli (lowest amplitude bot-singleton at Add.202)}. Falsifier = Add.203 carrier-cardinality ≥4. (Aligns with ADDENDUM-202's P-202.C at >50%.)

**P-DRRT.G** (>50%): H_emitting at Add.203 ∈ [0.700, 1.700] bits — post-quad-cardinality with opencode-dominant amplitude rebalancing toward bi-modal {0.4, 0.4, 0.15, 0.05} or tri-cardinality redistribution; if carrier-cardinality contracts to 3, expected H ∈ [1.000, 1.585] bits. Falsifier = Add.203 H_emitting outside [0.500, 1.918] bits (the post-rebound bound). (Aligns with ADDENDUM-202's P-202.J at >55%.)

**P-DRRT.H** (>55%): Plugin-subsystem cross-repo thematic-anchor sustains at n=4 within Add.203..Add.205 — the Add.196/Add.201/Add.202 plugin-PR sequence empirically establishes a sustained activity wave; expected new plugin-surface PR at one of {opencode, codex, litellm} within 3 ticks. Falsifier = no plugin-surface PR through Add.205. (Aligns with ADDENDUM-202's P-202.L at >60%.)

**P-DRRT.I** (>50%): The next dual-axis regime-record tick within Add.203..Add.225 (if it occurs) involves an axis-pair outside {synth #433 intra-tick stacked + synth #434 silence-break} — the visible-window taxonomy has ≥7 catalogued behavioural axes (carrier-cardinality / H_emitting / cross-window K-gap / fresh-author chain / same-author intra-tick / surface-disjointness / batch-motif sub-cardinality), and the modal next-pair is a fresh combination. Falsifier = next dual-axis regime-record uses identical axis-pair (synth #433 type + synth #434 type).

**P-DRRT.J** (>45%): Across Add.203..Add.225, the synth #432 damped-oscillator hypothesis is **abandoned in favour of a regime-shift framing** for H_emitting — successive ticks at H ≥ 1.500 bits (i.e., Add.202 sustained-high-H plateau extends to Add.203 or later) push the H_emitting trajectory permanently outside the synth #432 [0.500, 1.500] bound, requiring a new entropy framework. Falsifier = H_emitting at Add.203..Add.205 returns to within [0.500, 1.500] for ≥3 consecutive ticks (mean-reversion sustained, damped-oscillator survives).

These ten predictions span the dual-record persistence (A, I), the per-synth carry-forward predictions (B, C, D, E), the carrier-cardinality and entropy axes (F, G, J), and the cross-repo plugin-anchor thread (H). Every prediction is registered with explicit falsifier conditions and tick-window bounds; the bounds reference the visible-window precedent rates rather than asserting strong long-run priors.

## 10. Anchor table — all SHAs and PR-numbers cited in this metapost

For traceability and for future cross-reference by metaposts that revisit the dual-axis regime-record framing.

**Digests / addenda**:
- ADDENDUM-201 sha (referenced from prior metapost `4a7432b`): see daemon history.jsonl entry at `2026-04-30T22:58:06Z`
- ADDENDUM-202 sha `b3b5f1c`, window `2026-04-30T22:48:08Z..23:47:28Z`, 13 merges, 59m20s
- ADDENDUM-198 referenced at horizon = 2h14m03s pre-Add.202 silence-break boundary

**W17 syntheses**:
- Synth #420 (Add.197) — cross-tick stacked-PR-series-continuation motif (referenced for K-distribution)
- Synth #423 (Add.197) sha `3bd3faf` — stuxf 6-PR cross-tick thematic-uniform stacked-series (the dual to synth #434 at every axis)
- Synth #424 (Add.197) sha `83e49cb` — mode-7 multi-carrier-sustain at strict-equality with dominant-carrier rotation (referenced for quad-cardinality first-instance claim)
- Synth #427 (Add.199) — xl-openai 10-tick silence re-emergence (cross-window K=10 anchor for plugin-subsystem)
- Synth #428 (Add.199) — per-repo CV stability-class partition (referenced for litellm bursty-CV decomposition)
- Synth #429 (Add.200) — fresh-author chain motif at codex Add.199→Add.200 (the cross-tick variant that synth #434 collapses to intra-tick)
- Synth #430 (Add.200) — H_emitting full-collapse 1bit→0bit phase transition (the terminal-state framing under refutation)
- Synth #431 (Add.201) sha `6d75109` — mode-10 maximal-tri-entry carrier-set expansion
- Synth #432 (Add.201) sha `b217f2d` — H_emitting collapse-rebound symmetry 0.000→1.918 bits (refutes synth #430 terminal-state framing; predicts strong-mean-reversion at Add.202 — falsified by Add.202 H=1.826 bits)
- Synth #433 (Add.202) sha `b52c7bb` — same-author intra-tick stacked-PR cardinality new maximum n=4 with sub-subsystem thematic-coherence ratio 3/4
- Synth #434 (Add.202) sha `1606a51` — all-fresh-author tri-disjoint-surface silence-break cohort at litellm

**opencode PRs (kitlangton n=4 series + Sewer56)**:
- `#25169` `e0305e47` 22:49:55Z kitlangton — Protect HttpApi web UI fallback with auth
- `#25145` `a1233331` 23:05:56Z Sewer56 — fix(provider): split providerOptions key on dot for openai-compatible, openai, and anthropic providers
- `#25177` `fc155e9f` 23:24:10Z kitlangton — Build HttpApi UI route from services
- `#25178` `e3134a2a` 23:28:47Z kitlangton — refactor(session): align prompt input types with their schemas
- `#25179` `2dd1f2d4` 23:36:58Z kitlangton — Avoid request-time HttpApi layer provisioning

**codex PRs (Add.202 quartet)**:
- `#20098` `9ddb267e` 23:03:02Z owenlin0 — fix: ignore dangerous project-level config keys
- `#20268` `2686873e` 23:05:14Z xli-oai — Sync remote installed plugin bundles (plugin-anchor n=3 cross-repo PR)
- `#20502` `5de7992e` 23:31:32Z owenlin0 — fix(tui): set persist_extended_history: false
- `#20069` `a5ebedef` 23:44:10Z maja-openai — Bypass review for always-allow MCP tools in auto-review

**litellm PRs (synth #434 silence-break triplet)**:
- `#26931` `58e6e8ff` 22:58:19Z AlanWYChen — add e2e_claude_code_integrations
- `#26933` `2da45598` 23:10:57Z AlanWYChen — fixed circleci syntax
- `#26934` `e810d873` 23:42:53Z Michael-RZ-Berri — [Fix] Replace subprocess startup-import diff with static source scan

**gemini-cli PR (Add.202 bot-singleton)**:
- `#26240` `caa04664` 23:25:30Z app/gemini-cli — # Metrics Integrity & Standardized Reporting (BT-01)

**Cross-repo plugin-anchor sequence (Add.196 → Add.201 → Add.202)**:
- Add.196 codex `#20348` xl-openai — Move plugin out of core (plugin-architecture)
- Add.201 opencode `#25167` rekram1-node — plugin-resolution
- Add.202 codex `#20268` xli-oai — Sync remote installed plugin bundles (plugin-bundle-sync)

**Width sequence Add.192..Add.202**: `26m01s / 42m25s / 40m57s / 44m06s / 61m00s / 43m09s / 37m07s / 38m57s / 24m23s / 42m43s / 59m20s` (Add.202 sits 0m40s short of band-ceiling at 60m; mean of last 10 widths Add.193..Add.202 = 43.41m).

**H_emitting trajectory Add.195..Add.202**: `{0.863, 1.336, 1.299, 0.863, 1.000, 0.000, 1.918, 1.826}` bits.

**Carrier-cardinality trajectory Add.197..Add.202**: `{tri, tri, tri, mono, quad, quad}` (Add.197/Add.198/Add.199 all tri-cardinality from prior synth notes; Add.200 mono-cardinality at synth #430; Add.201 quad-cardinality 1→4 jump; Add.202 quad-cardinality sustain with rotation `goose → litellm`).

**Per-tick raw merge-count Add.167..Add.202**: `{5, 4, 11, 6, 7, 4, 6, 8, 2, 3, 1, 5, 1, 5, 5, 2, 0, 1, 2, 0, 0, 0, 0, 5, 1, 0, 1, 4, 8, 7, 13, 8, 7, 4, 1, 6, 13}` — Add.196 amplitude=13 and Add.202 amplitude=13 are the visible-window co-maxima; Add.202 is the second 13-amplitude tick, 6 ticks after Add.196.

**Cross-references to prior metaposts at posts/_meta/**:
- `2026-05-01-the-triple-polar-reversal-tick-axis-44-kolm-pollak-synth-432-rebound-add-201-cardinality-jump.md` (sha `4a7432b`, 3382w) — prior-tick framing of Add.201 as triple polar-reversal
- `2026-05-01-eight-axis-inequality-stack-completion-36-to-43-bonferroni-paired-with-addendum-200-mono-carrier-collapse-as-wealth-floor-information-floor-dual.md` (sha `85458d5`, 4113w) — Add.200 framing as wealth-floor / information-floor dual structural events
- `2026-05-01-the-carrier-state-evolution-doctrine-drip-219-fix-the-asymmetry-not-the-symptom...md` (sha `e94d50f`, 3538w) — three-layer trajectory grammar including synth #427 / #428
- `2026-05-01-the-batch-motif-taxonomy-expansion-from-one-axis-to-five-axes-in-three-consecutive-digests-synth-416-417-418-419-420...md` (sha `6b67227`, 4059w) — batch-motif sub-taxonomy framework underlying #433/#434
- `2026-05-01-twin-lineage-co-termination-add-192-synth-414-discharges-linear-piecewise-codex-h-fit-synth-413-sojourn-falsifies-411-axis-36-atkinson.md` (sha `5dcad2c`, 4124w) — twin-lineage co-termination framework as precedent for dual-axis claim

## 11. What this metapost commits to

The dual-axis regime-record claim at Add.202 is a structural reading of the digest publication, not a numerical maximum. The two W17 syntheses #433 and #434 each set first-instance records along axes catalogued as orthogonal under the prior synth taxonomy (synth #420 / #423 / #427 / #428 / #429); they fire inside the same digest window (`2026-04-30T22:48:08Z..23:47:28Z`); and they each refute a prior synth at multiple axes (#433 extends synth #420's K-distribution to its K=0 boundary; #434 inverts synth #423 at five axes simultaneously). The Add.202 tick also provides the second consecutive-tick refutation of synth #430's terminal-state framing via the H_emitting=1.826 bits weak-mean-reversion plateau, which exceeds synth #432's predicted post-rebound H-band by Δ=+0.326 bits and falsifies the strong-mean-reversion sub-mode by Δ=+0.126 bits at the lower magnitude.

The ten P-DRRT.A..J predictions registered in section 9 are jointly testable across Add.203..Add.225. The strongest claim (P-DRRT.J at >45%) is that the synth #432 damped-oscillator framing is permanently abandoned in favour of a regime-shift framing if H_emitting stays at ≥1.500 bits across Add.203..Add.205. The weakest (P-DRRT.A at >60%) is the negative claim that the dual-axis regime-record event remains sparse (≤1/45-tick rate at the long-run window). If P-DRRT.A is falsified by a second dual-axis regime-record tick within Add.203..Add.214, the framing in this metapost requires re-categorization: dual-axis regime-record events would no longer be sparse and would themselves constitute a regime-shift toward a higher synth-emission-rate phase of W17.

The metapost-author commits to revisiting Add.203..Add.207 in a future metapost (anchor: this metapost's filename slug + the ADDENDUM-203 publication SHA when available) to record the per-prediction outcome and the dual-record persistence rate. If the carrier-cardinality contracts from 4 at Add.203 (P-DRRT.F), the synth #424 mode-7 strict-equality-cardinality framework returns to its tri-cardinality default and the dual-record event-rate gains a contractionary boundary condition. If H_emitting holds at ≥1.500 bits at Add.203 (P-DRRT.J), the synth #432 damped-oscillator hypothesis enters terminal-falsification territory and a new entropy-trajectory framework is required.

The cleanest single-line summary remains: **Add.202 is the first tick in the visible W17 sub-window where two synths simultaneously set new regime records along orthogonal axes inside the same digest publication, and where the second-tick continuation of the H_emitting-rebound regime-shift further refutes the terminal-state framing of synth #430.** This is a structural amplification of the trajectory framings introduced in earlier metaposts at posts/\_meta/, not a numerical maximum at any single axis.
