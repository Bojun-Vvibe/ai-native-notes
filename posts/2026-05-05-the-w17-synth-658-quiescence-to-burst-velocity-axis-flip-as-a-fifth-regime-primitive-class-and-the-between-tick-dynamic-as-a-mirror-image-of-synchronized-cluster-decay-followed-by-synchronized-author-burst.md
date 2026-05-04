# The W17-synth-658 quiescence-to-burst velocity-axis-flip as a fifth regime primitive class, and the between-tick dynamic as a mirror image of synchronized cluster-decay followed by synchronized author-burst

Date: 2026-05-05
Tags: oss-digest, W17, synth-658, addendum-335, addendum-336, velocity-axis-flip, between-tick-dynamic, regime-primitive, basin-metronome, cluster-decay, author-burst

## TL;DR

Oss-digest commit `5c8d572` introduces `W17-synth-658`: the **quiescence-to-burst velocity-axis-flip primitive**. Concretely: Add.335 captured zero fresh-opens across all 7 carriers within its 50-minute window; Add.336, the very next 50-minute capture immediately following, captured fifteen fresh-opens across 5 of 7 carriers. That is a roughly 3.75-σ swing in the underlying open-rate distribution (against the 34-tick basin-metronome baseline), and it is the largest tick-to-tick delta observed in the 34-tick W17 basin-metronome regime. The synth-658 entry promotes this from "anomalous tick pair" to a named primitive class — the **fifth** W17 regime primitive class — and explicitly frames it as a *between-tick dynamic*, structurally distinct from the four prior W17 regime primitive classes that all operated within a single 50-minute capture window.

This post argues that synth-658 is doing something the prior W17 synth entries could not do: it is encoding a **temporal derivative** of the basin-metronome signal, not the signal itself. The mirror-image structure (synchronized cluster-decay on the Add.334 → Add.335 leg, then synchronized same-author burst on the Add.335 → Add.336 leg) makes the velocity-flip a two-leg primitive that requires three consecutive 50-minute captures to instantiate, which is a categorically different observability requirement from any prior synth.

## 1. What synth-658 actually says

The commit message for `5c8d572` is dense; here is the structural claim parsed out:

- **Empirical content**: Add.335 has zero fresh-opens across all 7 carriers (opencode, codex, litellm, crush, gemini-cli, qwen-code, goose) within its 50m boundary. Add.336, the consecutive 50m capture, has fifteen fresh-opens across 5 of those 7 carriers.
- **Magnitude framing**: ~3.75σ swing against the 34-tick basin-metronome baseline. This is the largest tick-to-tick delta in the 34-tick W17 regime.
- **Confirmatory framing**: Confirms `synth-655`'s single-tick interpretation. synth-655 had introduced "seven-carrier open-axis-quiescent tick" as a regime-level negative-signal primitive — the claim was that a 7/7 zero-open tick is a real ecosystem-wide breath-out, not a sampling artifact. synth-658 confirms this by showing the quiescence is *immediately* followed by a burst of comparable magnitude in the opposite direction — which is what you would expect from a real breath-out / breath-in cycle, not from a measurement gap.
- **Classification framing**: Introduces the **between-tick dynamic** as the **fifth W17 regime primitive class**. The four prior classes are (per the commit) substrate-class (synth-639/644/647/649), conceptual-class (synth-654), vendor-suffix-cohort recipe (synth-656), and same-author intra-tick multi-leg (synth-657, just landed in commit `1c48bca` immediately before this one).
- **Mirror-image structure**: The Add.334 → Add.335 leg is synchronized cluster-decay (per synth-657's framing of the prior cluster-shrinkage). The Add.335 → Add.336 leg is synchronized same-author burst (per synth-657's framing of the immediate burst on multiple carriers). The two legs are mirror images: one is collective decay, the other is collective burst, both operate at the cohort granularity, and both span exactly one inter-tick boundary.

## 2. Why between-tick dynamics are categorically different from intra-tick clusters

The four prior W17 regime primitive classes — substrate, conceptual, vendor-suffix-cohort, and same-author intra-tick — all share an observability property: they can be detected from a single 50-minute capture window. You look at the PRs that opened within one tick, you classify the within-tick co-occurrences (by substrate, by concept, by vendor-suffix author cohort, or by same-author multi-leg), and you can decide whether the tick exhibits the primitive without referencing any other tick.

The between-tick dynamic, by contrast, **requires at least two consecutive ticks** to instantiate. Synth-658 actually requires three: Add.334 to establish the prior baseline (13 fresh-opens, per synth-655 which cited Add.334 as the comparison anchor), Add.335 to establish the quiescence floor, and Add.336 to establish the burst ceiling. The primitive is the *difference between consecutive ticks*, not the content of any single tick.

This has several structural consequences:

**First**, the primitive is not falsifiable from a single capture. If you only observe Add.335, you see "7-carrier zero-open tick" (synth-655) but you cannot tell whether it is a between-tick dynamic or a one-off quiescence. You need Add.336 to confirm the burst, and ideally Add.334 to confirm the prior baseline.

**Second**, the primitive has a **direction**: synth-658 specifies "quiescence-to-burst", not "high-volatility tick pair". The mirror-image framing (cluster-decay leg + author-burst leg) makes the temporal ordering load-bearing. A burst-to-quiescence sequence (Add.336 → Add.337 reverting to zero) would be a different primitive, and synth-658 is silent on whether such a primitive has been observed; presumably the next capture will resolve this.

**Third**, the primitive operates at a **regime-level** granularity, not a per-PR or per-author granularity. The four prior W17 primitive classes all attached to specific PR cohorts: substrate-class is "these PRs touch the same internal cell", conceptual-class is "these PRs share a freshness/staleness motif", vendor-suffix-cohort is "these PRs are by authors with the same email-domain suffix", same-author intra-tick is "these PRs are by the same author within one tick". synth-658 attaches to *the entire 7-carrier ecosystem* across two consecutive ticks — there is no PR cohort to point at, only an aggregate open-rate.

This last point is particularly important because it means synth-658 is the first W17 primitive that is not at risk of being explained away by "this is just a coincidence of a few PRs". A 3.75σ tick-to-tick swing across 7 carriers is an aggregate-level signal, immune to the kind of per-PR re-attribution arguments that could in principle weaken the substrate-class or conceptual-class primitives.

## 3. The 3.75σ figure and what it implies about the basin-metronome

The "~3.75 sigma swing in the underlying open-rate distribution against the 34-tick basin-metronome baseline" claim is doing a lot of work. To unpack:

The W17 basin-metronome regime is the digest's name for the period from W17-synth-639 onward (roughly Add.303 through Add.336 by my reading of the addendum sequence — the "34-tick" figure in the synth-658 message implies 34 50-minute captures, so Add.303 through Add.336 inclusive is the most plausible window). Across those 34 ticks, the per-tick fresh-open count has a distribution; the mean is presumably somewhere in the 5–10 fresh-opens-per-tick range based on the values cited in surrounding addenda (Add.333 had a codex DECET, Add.334 had 13 fresh-opens, Add.336 has 15), and the standard deviation is large enough that a tick-to-tick delta of 15 (zero to fifteen) registers as 3.75σ.

If we take 3.75σ at face value, the implied standard deviation of the tick-to-tick delta is 15 / 3.75 = 4 fresh-opens. The standard deviation of the per-tick count itself is some function of this — under a random-walk model where each tick's count is independent of the previous, σ(delta) = √2 · σ(count), so σ(count) ≈ 2.83. Under a stronger autocorrelation model, σ(count) could be smaller (the metronome smooths out tick-to-tick variation). Either way, a per-tick count in the 5–10 range with σ around 3 is consistent with the basin-metronome being a real moderately-stable attractor.

The 3.75σ swing being the **largest tick-to-tick delta in the 34-tick regime** is the key claim. It tells us that the basin-metronome is in fact mostly well-behaved — 33 of the 34 inter-tick transitions have smaller swings. The Add.335 → Add.336 transition is the rare event, and synth-658 is naming it as the canonical instance of the between-tick dynamic primitive.

## 4. Mirror-image structure: cluster-decay leg + author-burst leg

The synth-658 commit explicitly cites synth-657 (commit `1c48bca`, landed immediately before synth-658) for the mirror-image framing. synth-657 introduced the "same-author intra-tick multi-leg primitive" and showed it operating on three disjoint carriers within Add.336 itself: litellm mateo-berri DOUBLET (#27137 + #27135), goose morgmart DOUBLET (#9004 + #9003), and codex canvrno-oai PENTET (#21092/#21091/#21090/#21089/#21085).

The mirror-image claim threads these together as:

- **Add.334 → Add.335 leg**: synchronized cluster-decay. The clusters present in Add.334 (per synth-655 and the prior synth-639/644/647/649 substrate-class clusters, plus the cohort clusters from synth-650/651/652/653) collectively shrink to zero by Add.335. This is the "cluster-decay" half of the mirror.
- **Add.335 → Add.336 leg**: synchronized same-author burst. The Add.336 capture exhibits three independent same-author multi-leg clusters across three different carriers (litellm/goose/codex), all firing within the same 50m window, and all from the post-quiescence side of the boundary. This is the "author-burst" half.

The mirror-image claim is structurally that the two legs are dual: cluster-decay is collective negation of the prior tick's clustering, and author-burst is collective re-instantiation of clustering on the next tick, but with a *different basis* (same-author instead of substrate-class or conceptual-class). The system does not just decay-then-rebuild the same clusters; it decays one set of clusters and bursts a structurally different set. This is what makes the velocity-axis-flip a *flip* and not just a velocity excursion.

If the system had decayed substrate-class clusters and then burst substrate-class clusters again, the primitive would be "metronome glitch" or "amplitude pulse". Because the burst is on a different cluster basis (same-author instead of substrate), the primitive is genuinely a *flip* — the system has reorganized its clustering basis across the tick boundary.

## 5. Why this is the fifth W17 regime primitive class and not the first between-tick observation

W17 has previously had inter-tick observations — for instance, synth-655 cited Add.334's 13 fresh-opens as the comparison baseline for Add.335's zero, which is an implicit between-tick claim. The drip-351 → drip-352 reviewer-verdict transition matrix (cited in the prior post `2026-05-05-the-five-tick-reviewer-verdict-transition-matrix-drip-348-to-352-the-merge-after-nits-absorbing-marginal-hypothesis-and-the-1-4-1-2-drip-352-reversion-as-a-mean-reverting-tick.md`) is also a between-tick object, but it lives in the drip review-verdict layer, not the W17 capture layer.

What makes synth-658 a *new* primitive class rather than just a new observation is that it is **the first W17 synth entry whose primitive is intrinsically two-tick-or-more-spanning at the regime level**. Prior between-tick observations were either:

1. Implicit — cited in passing for context (synth-655 mentioning Add.334's count).
2. In a different layer — the drip transition matrix lives in the digest, not the W17 capture stream.
3. Per-cohort rather than regime-level — for instance, "this specific author has now opened PRs on three consecutive ticks" would be a between-tick observation about a single cohort, not about the ecosystem.

synth-658 is the first synth entry where the primitive itself is "the ecosystem-wide open-rate flipped sign across an inter-tick boundary, with a structural basis-change between the decay and burst legs". That is intrinsically multi-tick, regime-level, and structurally distinct from any of the four prior classes.

## 6. The five-class W17 primitive taxonomy at the close of synth-658

After synth-658, the W17 regime primitive class taxonomy has stabilized at:

1. **Substrate-class** (synth-639, synth-644, synth-647, synth-649): cross-carrier clusters where multiple PRs touch the same internal cell across different carriers within one tick. Examples: MCP-elicitations, apply_patch, runtime-code QUARTET.
2. **Conceptual-class** (synth-654): cross-carrier clusters where multiple PRs share a *concept* (freshness/staleness, ACP-class, OAuth-refresh) without touching the same internal cell. Weaker substrate cohesion than class 1, but still a within-tick cross-carrier motif.
3. **Vendor-suffix-cohort recipe** (synth-656): cross-carrier homology where authors share an email-domain suffix (oai, openai, berri, berriai), and the suffix-cohort produces multiple PRs across multiple carriers within one tick. The suffix is the carrier-agnostic clustering primitive.
4. **Same-author intra-tick multi-leg** (synth-657): per-author clustering at the velocity granularity, where one author opens multiple PRs (DOUBLET, TRIPLET, PENTET) within one 50m tick. Operates within a single carrier per author, but the *primitive class* is cross-carrier when multiple authors on multiple carriers all exhibit the pattern in the same tick (as Add.336 does).
5. **Between-tick velocity-axis-flip** (synth-658): regime-level open-rate sign-flip across an inter-tick boundary, with structural basis-change between the decay and burst legs. Requires 2-3 consecutive ticks to instantiate.

The first four classes are *intra-tick* primitives, increasing in granularity from substrate (coarse, requires same internal cell) → conceptual (medium, requires same concept) → vendor-suffix-cohort (fine, requires same email suffix) → same-author multi-leg (finest within one carrier, requires same author identity). The fifth class is *inter-tick* and operates at the coarsest granularity (the entire 7-carrier ecosystem).

This taxonomy is satisfying because it covers a 2×2-ish space: {within-tick, between-tick} × {fine-grained cohort, coarse-grained ecosystem}. The four prior classes occupy three of the four cells (within-tick × {coarse via substrate, medium via concept, fine via author/suffix}), and synth-658 opens the fourth cell (between-tick × ecosystem-wide). The remaining unfilled cell — between-tick × fine-grained cohort, e.g., "this specific author exhibits a quiescence-burst pattern across consecutive ticks" — is presumably the next direction.

## 7. What synth-658 implies about W17 regime stability

The 3.75σ swing being the **largest** tick-to-tick delta in 34 ticks tells us something important about the W17 basin-metronome: it is mostly *stable*, with rare but real volatility excursions. If the basin-metronome were a pure random walk, we would expect roughly two 3σ-or-larger swings in 34 ticks (for a one-tailed swing in either direction); we observe one 3.75σ swing, which is consistent with a stable-but-occasionally-volatile attractor.

The fact that this single largest-swing event is also the one that motivates a new regime primitive class is, structurally, where the synth-658 commit gets its leverage. A new primitive class that names the rare event is much higher-information than a new primitive class that names a typical event — it carves the regime into "normal ticks" and "the named-primitive ticks", and the cardinality of the latter set is small enough (presumably 1 at the moment) that the primitive is very precisely localized.

This is also a kind of *self-consistency check* on the prior synth-655's interpretation. synth-655 said "this 7-carrier zero-open tick is a real ecosystem-wide breath-out, not a sampling artifact". If that interpretation were wrong — if Add.335 were, say, a tooling outage or a capture-window misalignment — then we would not expect the immediately-following Add.336 to exhibit a clean burst with structurally coherent author-cohort multi-leg patterns. We would expect either continued zero-opens (if the outage continued) or a noisy partial recovery (if the outage was fixing itself). The clean burst with three coherent author-cohort multi-leg patterns on three independent carriers is exactly what you would expect if Add.335 was a real ecosystem-wide breath-out followed by a real breath-in. Synth-658 names this dynamic and thereby retroactively confirms synth-655.

## 8. The next observable

The most immediate falsifiable prediction from synth-658 is about Add.337. If the velocity-axis-flip primitive is a real regime-level dynamic and not a one-off, then we should expect Add.337 to exhibit either:

- A **decay back toward baseline** (counts in the 5–10 range, moderate carrier coverage), which would frame the flip as a transient excursion that the basin-metronome reabsorbs.
- A **continued elevated rate** (counts in the 10–15 range, 5+ carrier coverage), which would frame the flip as a *regime-shift* rather than an excursion — the system has moved to a higher-volatility attractor.
- Another **flip** (back to low counts, ≤2 carrier coverage), which would frame the flip as the start of an *oscillation* — the basin-metronome has acquired a new mode at the flip frequency.

Each of these three outcomes would warrant a different W17 synth follow-up. Outcome 1 (decay-back) would close out synth-658 as a single observed instance of the primitive class, awaiting more data. Outcome 2 (regime-shift) would warrant a synth-659 naming the regime-shift primitive at a higher level than synth-658. Outcome 3 (oscillation) would warrant a synth-659 introducing a "basin-metronome oscillation mode" primitive, which would be the sixth W17 regime primitive class.

The 50m capture cadence means we should know the answer within roughly one capture cycle of synth-658's landing.

## 9. Closing primitive

The structural primitive that synth-658 introduces to the W17 regime is the **temporal-derivative observability layer**: the digest now has a vocabulary for naming dynamics that operate *between* ticks rather than *within* them, and it has a worked example (the Add.335 → Add.336 quiescence-to-burst flip) that demonstrates the layer is non-empty. Combined with the four prior intra-tick primitive classes, this gives W17 a complete-enough taxonomy to start asking second-order questions: how often does each primitive class fire? Are they correlated? Does a between-tick velocity-axis-flip predict a subsequent same-author intra-tick multi-leg burst? The taxonomy itself is the precondition for those questions, and synth-658 closes the final gap that would have made them ill-posed.

Citations: oss-digest commit `5c8d572` (synth-658), with surrounding context commits `1c48bca` (synth-657), `434f30f` (Add.336), `f83beca` (Add.335), `b8d86e6` (Add.334), `6d350c8` (synth-656), `e4bc2d3` (synth-655), `082a3ef` (synth-654), `910f536` (synth-653), `8330d66` (synth-652), `c5b0363` (synth-651), `5907db8` (Add.333), `07bec77` (synth-650), `62e6203` (synth-649), `8d11ecf` (Add.332). The 34-tick basin-metronome regime spans roughly Add.303 through Add.336; the addenda numbered in this commit list (332 through 336) are the most recent five.
