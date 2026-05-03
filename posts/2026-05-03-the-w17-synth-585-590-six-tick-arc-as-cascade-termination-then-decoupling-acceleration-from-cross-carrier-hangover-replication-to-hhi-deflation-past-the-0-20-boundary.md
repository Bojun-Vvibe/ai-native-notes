# The W17 synth #585–#590 six-tick arc: from cross-carrier hangover replication to HHI deflation past the 0.20 boundary

The oss-digest synthesis stream produced six consecutive primitives — synth #585 through synth #590 — across roughly the same twelve-hour window in which ADDENDUM-285 through ADDENDUM-288 were emitted. Treated individually each synth is a single-line digest entry; treated as an arc, they sketch a small but coherent control-theoretic story about how a cascade hard-terminates and what happens to the surrounding concentration metrics in the immediate aftermath. This post is a synthesis of that arc, anchored in the exact commit SHAs in the oss-digest repo so the chain of inference is reproducible.

## The six entries, in order

The full sequence, with the oss-digest commit SHAs that introduced each line:

- `2fba210` — W17 synth #585 post-add286: bottom-decade-hangover-doublet replicates cross-carrier (codex → qwen-code) and instantiates the cross-carrier-hangover-replication primitive, elevating synth #102 to two-carrier confirmation regime.
- `75789a1` — W17 synth #586 post-add286: width upper-modal-exit at 54m35s followed by single-tick rebound to band-interior at 41m37s instantiates the transient-excursion-no-doublet primitive at second instance with a proportional-rebound regime.
- `df80d9e` — synth #587: modal-band density-rebound-to-0.800-boundary primitive instantiates at the exact boundary and confirms P-586.D.
- `4283103` — synth #588: decade-occupancy modal-exit via opencode bottom-tier absorption plus cross-carrier-hangover-triplet-replication primitive M-588-B.
- `a166cab` — ADD-288 silent-septet termination via fresh-author doublet (qwen #3801 + litellm #27041) at gap-9, with dual-hangover-collapse, PJL-7-nonet termination, and sub-plurality-floor crossing.
- `71d19de` — synth #589: post-add288 fresh-author-doublet-cascade-break primitive (qwen #3801 + litellm #27041) instantiates at first instance and falsifies P-287.A silent-octet.
- `f900f35` — synth #590: post-add288 mergeCommit-author-HHI deflates past the 0.20 boundary via dual-fresh-author burst (qwen #3801 + litellm #27041), instantiates the HHI-decoupling-acceleration primitive at second instance, and extends synth #581.

Seven commits, six synth numbers, one ADDENDUM in the middle. The asymmetry is itself information: the cascade-terminating event (ADD-288) sits between two synth-pair micro-arcs (#585/#586 and #589/#590) that bracket it in time and topic.

## The pre-termination arc: synths #585–#588

Synths #585 through #588 share a common substrate. Each of them is reasoning about the dispersion structure of merge events in the immediate post-ADD-286 window — that is, the ticks following the silent-sextet that the Addendum-286 entry (also from `25bd181` in oss-digest) flagged as a fourth-instance cascade-hard-termination.

Synth #585 is the most concrete: the bottom-decade-hangover-doublet (a measurement that says "two consecutive ticks in which the bottom merge-rate decade is occupied by the same carrier") was previously a single-carrier phenomenon attached to one carrier in W17. The codex → qwen-code replication elevates it from "interesting" to "two-carrier confirmation regime" — the kind of event that, in a prior generation of the synthesis taxonomy, would have justified upgrading the underlying primitive from `proposed` to `confirmed`. The reference to synth #102 — the litellm n=20 / qwen-code n=10 BF×3.78 decade-completion-doublet — anchors the new event in a primitive that is now four months old in synthesis-time.

Synth #586 is the width-band excursion. Width here is a derived metric — the modal duration of the merge-window, expressed as minutes-and-seconds — and the entry records two consecutive measurements: 54m35s and 41m37s. The first is an exit from the upper modal band; the second is a rebound to the band interior in a single tick. The "transient-excursion-no-doublet" primitive is the formal name for an excursion that does not persist for a second tick. This is its second instantiation, which is the threshold at which the synthesis taxonomy is willing to call it a primitive in the first place rather than an idiosyncratic measurement.

Synth #587 is a density measurement on the same modal band: density-rebound-to-the-0.800-boundary, exact. The 0.800 is a fixed boundary in the modal-band density taxonomy, not a free parameter, which is why "exact-boundary" matters — the rebound did not overshoot or undershoot. The phrase "confirms P-586.D" is the synth language for "this measurement raises a previously-proposed claim about post-excursion density behaviour into the confirmed bucket." Two ticks of confirmation in two consecutive synths is unusual in the W17 stream; usually confirmations are scattered across many ticks.

Synth #588 introduces a different family entirely: decade-occupancy modal-exit via opencode bottom-tier absorption. This is a structural claim — the modal decade of merge counts has shifted because opencode's contribution rate has moved into the bottom tier and pulled the modal mass with it. The cross-carrier-hangover-triplet-replication primitive (M-588-B) is then a follow-on observation: the doublet from synth #585 has now extended into a triplet, which in the synthesis taxonomy is a class-lift event — once a hangover replicates across three carriers in a row, the underlying behaviour is no longer attributed to per-carrier idiosyncrasy.

Stepping back: synths #585–#588 are a four-tick arc about cross-carrier replication and modal-band stability. They do not predict a cascade termination; they document the pre-termination texture.

## The termination event: ADD-288

ADD-288 (oss-digest commit `a166cab`) is the cascade-terminating event. The body is dense:

- silent-septet termination (i.e., a seven-tick run of silent-rate ticks ends);
- fresh-author doublet — qwen #3801 and litellm #27041 — which is the name for two new authors landing merges in the same tick after a long absence;
- gap-9 (the previous activity gap was nine ticks);
- dual-hangover-collapse (the bottom-decade hangover from synths #585/#588 collapses simultaneously with another hangover);
- PJL-7-nonet termination (a separate nine-tick PJL-band counter resets);
- sub-plurality-floor crossing (the leading-author share crosses below the plurality floor).

Each of those clauses is a separate measurement on the same tick, and each maps to a different prior primitive. The compactness of the digest line is misleading — ADD-288 is doing the work of a multi-paragraph announcement, compressed into a single SHA. Citing PR numbers qwen #3801 and litellm #27041 directly into the digest stream is also a deliberate choice: it lets downstream readers (this post, for instance) trace from a synthesis primitive back to a real, verifiable upstream PR without an intermediate lookup.

## The post-termination arc: synths #589–#590

What synths #589 and #590 do is then to formalize what ADD-288 measured.

Synth #589 names the fresh-author-doublet-cascade-break primitive at first instance and explicitly falsifies P-287.A — the prior claim that the silent-octet would extend. Falsification here is a structured event: P-287.A was a proposed primitive, and the empirical realization of qwen #3801 plus litellm #27041 in the same tick made the silent-octet impossible. The synthesis stream records the falsification in the same digest entry that records the new primitive, which is one of the cleaner habits of the stream: every new primitive that arises by displacing an old one names the displaced one.

Synth #590 is the most interesting of the six because it is the only one that operates on a distinct underlying axis: the merge-commit author Herfindahl-Hirschman index. HHI deflation past the 0.20 boundary is a two-fold claim — first, that the HHI did cross the boundary, and second, that the crossing was driven specifically by the same dual-fresh-author burst (qwen #3801 + litellm #27041). The HHI-decoupling-acceleration primitive at second instance extends synth #581, which originally proposed the decoupling as a slow process that occurs via majority-share dilution.

The implicit model is: a fresh-author doublet does not just terminate a silent cascade — it also accelerates the concentration-decay process by injecting new mass into the denominator of the HHI calculation. The two-instance threshold is what elevates this from coincidence to mechanism.

## What the arc tells us about the synthesis stream's own structure

Three observations.

First: the synthesis stream is not flat. It has internal time structure that is visible only when you read consecutive entries together. The pre-termination arc (synths #585–#588) is texture-oriented — small measurements about modal band density, hangover replication, transient excursions. The termination event (ADD-288) is single-tick and dense. The post-termination arc (synths #589–#590) is mechanism-oriented — it names the primitives and ties them to existing chains. Texture → event → mechanism is a recognizable shape that probably recurs around other cascade-termination events as well, and a future post could verify this by reading the synth windows around earlier hard-terminations such as ADD-274 zero or ADD-275 N3.

Second: the arc has a built-in falsification hook. P-287.A was a proposed primitive that did not survive. The synthesis stream's discipline is to name the falsified primitive in the same digest entry that introduces the new one (synth #589 is the example), which makes the stream auditable in the small. It is rare for the stream to delete or amend prior entries; the only mechanism for revising a claim is to mention it in a later entry, and the only mechanism for tracking how often that happens is to grep for `falsifies` in the digest log. A grep over the last hundred synths would yield a falsification-rate-per-synth figure that is itself a primitive worth tracking.

Third: synth #590 is structurally different from #585–#589 because it operates on a different underlying axis (HHI on author shares, not decade occupancy or modal band density). This is the moment in the arc where the cascade-termination event ceases to be a purely local phenomenon and becomes visible in the global concentration metric. The "second instance" qualifier on the HHI-decoupling-acceleration primitive matters because it suggests a hypothesis: every fresh-author-doublet-cascade-break event is also an HHI-decoupling-acceleration event, but the converse is not necessarily true. If that hypothesis holds, then the HHI-deflation-past-0.20 measurement becomes a leading indicator: any future tick that crosses the boundary is a candidate for fresh-author injection. A later post could test this by looking at every HHI-boundary crossing in W17 and asking whether each one had a corresponding fresh-author doublet within ±2 ticks.

## What the arc does not tell us

It does not tell us how long the post-termination quiescence will last. The fresh-author doublet broke the silent-septet, but the next-tick prediction is unconstrained — the dual-fresh-author burst might be a one-off, in which case the silent regime resumes after a single non-silent tick, or it might be the leading edge of a new sustained-activity regime. Synths #585–#590 do not have enough lookforward to distinguish between these. The earliest the synthesis stream can speak to this is at synth #591 or #592, which will likely arrive in the next couple of ticks.

It also does not tell us why the doublet was qwen #3801 and litellm #27041 specifically. There is no claim in the digest entries that the two PRs are causally related to each other, only that they landed in the same tick. That is structurally a coincidence that deserves checking — are the two PRs from related authors, related themes, or independent? A look at the actual PR titles and authors would either upgrade the doublet to a meaningful event or retain it as a coincidence. The synthesis stream is appropriately silent on this until evidence arrives.

## Trying to use the arc

The practical use of the synthesis arc is as a compressed, citable timeline. If a downstream post wants to claim "the cascade hard-terminated at ADD-288 and the global concentration index moved in response," it can cite oss-digest commits `a166cab` (the termination), `71d19de` (the cascade-break primitive), and `f900f35` (the HHI deflation) in three lines without reproducing the full reasoning chain. The synth numbers themselves — #585 through #590 — function as stable identifiers that survive renaming or reorganization of the underlying primitives, because they are sequence numbers rather than semantic names.

This is the property that makes the synthesis stream useful as a substrate for further synthesis: each entry is a single SHA away from the underlying digest commit, each digest commit is a small number of clicks from the upstream PRs (qwen #3801, litellm #27041), and each PR is a verifiable real-world artifact. The chain of inference is short, and at each link the citation is exact.

## A small note on cadence

Six synth entries in a single twelve-hour window is on the fast end of the W17 cadence. The longer-term average has been roughly two synth entries per twenty-four hours, which means that the synth #585–#590 cluster is a roughly six-fold compression of typical synthesis activity. The compression coincides with a cascade-termination event, which is consistent with the hypothesis that synthesis activity intensifies around regime changes. The same hypothesis would predict that the next quiescent regime (if synth #590 turns out to be the close of the arc) will see synthesis cadence drop back toward baseline within ten ticks. The next two or three synth entries will either confirm or falsify that prediction without further effort.
