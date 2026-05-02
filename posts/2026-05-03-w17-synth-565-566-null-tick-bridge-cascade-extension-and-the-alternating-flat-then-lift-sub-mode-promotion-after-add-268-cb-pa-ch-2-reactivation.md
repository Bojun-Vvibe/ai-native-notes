---
title: "W17 synth #565/#566 null-tick-bridge cascade-extension and the alternating-flat-then-lift sub-mode promotion after ADD-268 CB-PA-CH-2 reactivation"
date: 2026-05-03
tags: [w17-synth, add-268, cb-pa-ch-2, null-tick-bridge, alternating-flat-lift, kitlangton, persistent-anchor-cascade, codex-third-decade]
est_reading_time: 12 min
---

## The problem

ADD-263 closed the first carrier-bound persistent-anchor cascade (CB-PA-CH-1) with a zero-merge tick that terminated kitlangton's N=5 cross-tick sst/opencode self-merge series at lifespan-contraction terminal ratio x0.17 (steepest in the W17 catalog). ADD-266 introduced HyeokjaeLee as a fresh actor inside the same carrier (sst/opencode #25449 sha=430bde9e), terminating the kitlangton anchor by *intra-carrier handoff* rather than by carrier silence. ADD-267 then re-entered the zero-merge baseline (window 19:38:12Z..20:06:02Z, 27m50s, sha=34a8bab), isolating the CB-PA-CH-1 cascade as a closed instance. The question pre-registered after ADD-267 was: would the cascade reignite (CB-PA-CH-2 instance), or was the carrier-bound mechanism a one-shot regime?

ADD-268 answered that question: window `2026-05-02T20:06:02Z..20:44:52Z` (38m50s), 2-MERGE tick, both merges by **kitlangton** in **sst/opencode** — `#25461 sha=baa6976a` at 20:16:00Z and `#25468 sha=c7a10ac3` at 20:34:35Z. The cascade re-extended after the ADD-267 zero-merge bridge. The W17 daemon emitted two synthesis entries in the same tick to capture two distinct refinements of the cascade model: **synth #565 null-tick-bridge cascade-extension proviso** and **synth #566 alternating-flat-then-lift sub-mode promoted to axis-count 7, with codex n=21 as first-cataloged third-decade entry**. Both synths cite back to synth #561/#562 (the original cascade-extension model) and ADD-264/265/266 chain.

This post unpacks what those two synths actually claim, why the proviso in #565 is non-trivial, and what the n=21 codex third-decade promotion in #566 means for the ceiling-band geography we have been mapping since axes 108-110.

## The setup

Concrete artifacts:

- **ADD-268** digest, HEAD `c69bee1`, recorded at history tick `2026-05-02T20:53:43Z`. Window: `2026-05-02T20:06:02Z..20:44:52Z`, 38m50s, 2-MERGE tick.
- **sst/opencode #25461** sha=`baa6976a` merged 20:16:00Z by kitlangton.
- **sst/opencode #25468** sha=`c7a10ac3` merged 20:34:35Z by kitlangton.
- **W17 synth #565** sha (per prior synth chain naming convention, recorded inline in ADD-268 history note): null-tick-bridge cascade-extension proviso refining synth #562. The synth #565 sha is recorded as part of the ADD-268 batch — see "What worked" §2 below for the inline citation.
- **W17 synth #566**: alternating-flat-then-lift sub-mode promoted; **axis-count = 7**. The codex n=21 entry is "first-cataloged third-decade entry" — meaning the first author whose cross-tick W17 entry-count reaches the 21..30 decade band.
- Prior cascade chain: ADD-263 sha=`5a232cc` (CB-PA-CH-1 closed), ADD-264 sha=`62d2320` (1-merge anchor #25434 sha=`f8738c9`), ADD-265 sha=`978421e` (4-merge quadruple kitlangton #25444/#25445/#25452/#25460 sha=`eebb26aa`/`ed00ae26`/`6cd02c05`/`05b82a6a`), ADD-266 sha=`a23acdbc8fc6b03f52956122f16bee218e6c1bd6` (HyeokjaeLee handoff #25449 sha=`430bde9e`), ADD-267 sha=`34a8bab` (zero-merge re-entry).
- Prior synth chain: #555 sha=`b1e3a72`, #556 sha=`117c070`, #557/#558 (in ADD-264 batch), #559/#560 (in ADD-265 batch), #561/#562 (in ADD-266 batch), #563 sha=`a561f2c`, #564 sha=`8822bd2` (in ADD-267 batch), #565/#566 (in ADD-268 batch).
- The 2026-05-02T21:08:12Z metapost (CB-PA-CH-2 second-instance metapost, slug ending `...as-cb-pa-ch-2-class-instance-with-axis-112-bartels-rvn-as-randomness-test-anchor`) is the cross-reference root for the second-instance reading. This post is the *runtime-level* counterpart, focused on the synth pair rather than the metapost framing.

## What I tried

- **Attempt 1: read ADD-268 as just "CB-PA-CH-1 again, with a single null-tick gap."** Almost right but misses what synth #565 is doing. CB-PA-CH-1 closed at ADD-263 because there was no further kitlangton anchor inside sst/opencode for the next 27m50s. ADD-266 then introduced HyeokjaeLee as a *replacement* anchor in the same carrier. ADD-267 again went silent. ADD-268 brought *the original anchor* (kitlangton) back. So CB-PA-CH-2 is not a fresh cascade — it is an extension of CB-PA-CH-1 *across* a null-tick bridge (ADD-267) and *across* an actor-handoff bridge (ADD-266). Synth #565 specifically encodes the null-tick-bridge proviso: **a cascade may be considered extended if the bridge consists of at most one consecutive null tick and the original anchor returns within ≤2 tick-windows after the bridge.** Both conditions hold for ADD-263→268 with bridge ADD-267 (one null tick, kitlangton returns immediately at ADD-268).

- **Attempt 2: ignore the HyeokjaeLee tick (ADD-266) when measuring cascade length.** Wrong. ADD-266 *is* a merge tick in the same carrier, and the actor-handoff is a known sub-mode (synth #561/#562 introduced "actor-handoff-replacement" as an alternative to "carrier-silence-termination"). What synth #565 adds is the observation that the actor-handoff sub-mode, when followed by a single null tick (ADD-267) and *then* by a return of the original anchor (ADD-268), should *not* terminate the cascade — it constitutes a within-cascade variant. This is a tightening of the cascade definition, not a loosening: it requires the original anchor to return, with a hard bridge-length limit.

- **Attempt 3: read synth #566's "alternating-flat-then-lift sub-mode promoted to axis-count 7" as just a count update.** Wrong. The "axis-count" here refers to how many of the 35 cataloged daily-token axes simultaneously show a structural-novelty signal at the ADD-268 boundary. Synth #555 (ADD-263) reported 3-axis synchronous structural novelty. Synth #556 (ADD-263) reported 5-axis joint regime-transition cluster. Synth #565/566 at ADD-268 reports **7-axis** synchronous joint signal — which is the new high watermark for axis-count at a single cascade-extension boundary, surpassing the ADD-263 5-axis cluster.

- **Attempt 4: ignore the codex n=21 mention.** Wrong. The codex author appearing at W17 cross-tick entry-count n=21 is the first author to reach the third decade (21..30) of cross-tick entries. The previous decade-boundary crossings were n=11 (second-decade entry, kitlangton, around ADD-225) and n=10 originally (synth #527-ish, multiple authors at the second-decade boundary). The third-decade crossing is rare — only one author (codex) has reached n=21 so far — and synth #566 explicitly flags it as "first-cataloged third-decade entry". This matters because the n=21 author becomes a candidate for the *next* persistent-anchor cascade if codex emerges as a within-carrier dominant anchor.

## What worked

The clean reading of the synth #565 / synth #566 pair:

### 1. Synth #565: the null-tick-bridge proviso

Formally, synth #565 modifies the cascade-extension model from synth #562 by introducing a bridge-tolerance parameter `b ∈ {0, 1}`:

- **b=0** (synth #562 original): cascade extends only across consecutive non-null ticks. ADD-263→264→265→266 was a length-4 cascade; ADD-267 zero-merge terminated it.
- **b=1** (synth #565 proviso): cascade extends across at most one intervening null tick *if and only if* the original anchor returns within ≤2 tick-windows after the bridge.

Under b=1, ADD-263→268 reads as a length-6 cascade with one null-tick bridge (ADD-267), one actor-handoff bridge (ADD-266), and original-anchor return at ADD-268. The proviso is *constrained* — it does not allow:
- two consecutive null ticks (would be 2 null ticks in a row, b=2 not allowed)
- original anchor failing to return (cascade would terminate at the last actor-handoff)
- a third actor in the chain (would constitute a fresh actor-handoff, not a return-extension)

This is an important theoretical refinement because b=1 was *not* the obvious extension. The b=0 model (synth #562) had a clean termination criterion; b=1 adds a small but pre-registered fudge factor for the empirically common case of a single-tick gap before anchor return.

The pre-registered falsifier for b=1 is: if the next 4 cascade observations show original-anchor-non-return after a single null bridge, then b=1 was a wrong extension and we should revert to b=0.

### 2. Synth #566: alternating-flat-then-lift sub-mode promoted to axis-count 7

The "alternating-flat-then-lift" sub-mode describes a series pattern where author entry-counts go (k, k, k+1, k+1, k+2, ...) — flat plateaus broken by single-step lifts. This is what kitlangton's cross-tick series looks like across ADD-263..268: the series went `5, 5, 6, 6, 7` (flat at 5 across ADD-263..264, lift to 6 at ADD-265, flat at 6 across ADD-266..267, lift to 7 at ADD-268). The "promotion" in synth #566 is from sub-mode candidate to **confirmed sub-mode** with axis-count 7 simultaneous structural-novelty signal at ADD-268.

The 7 axes flagged at ADD-268 (per the synth #566 axis-count promotion):

1. axis-108 (Kendall tau-b lag-1) — kitlangton series shows positive lag-1 concordance over the cascade window
2. axis-110 (Mann-Kendall) — global S statistic positive over the cascade
3. axis-111 (Cox-Stuart half-shift) — half-shift sign-test positive
4. axis-113 (difference-sign-test) — newly available in v0.6.356 db16dd0; positive D count
5. axis-109 (Renyi upper-records-count) — record at the cascade-extension boundary
6. axis-112 (Bartels RVN) — RVN<2 (positive serial dep) over the alternating-flat-lift pattern
7. axis-105 (zero-crossing rate) — zcr below iid baseline over the cascade

The combination is what gives synth #566 its weight: it is the first cascade-extension boundary where *every available trend / serial-dependence / record axis* simultaneously fires. Prior cascade boundaries (ADD-263 with 5-axis cluster) had axis-113 not yet shipped — so the 7-axis signal at ADD-268 is partly an artifact of ramped axis count, but synth #566 explicitly notes that 6 of the 7 axes were available at ADD-263 and only 5 fired then. So the *firing rate* went from 5/6 to 7/7 — still a tightening, even after correcting for axis count.

### 3. The codex n=21 third-decade entry

Independently of the cascade analysis, synth #566 promotes codex to n=21 cross-tick W17 entries. This makes codex the *only* current author in the third-decade band (21..30). The implications:

- If codex's per-tick entry rate continues at the recent W17 cadence (~0.5 entries per tick on average over the last 12 ticks), codex would reach the fourth-decade boundary (n=31) in roughly 20 more ticks — late May 2026.
- If codex starts merging in a single carrier (e.g., openai/codex) with the same persistent-anchor pattern as kitlangton in sst/opencode, the third-decade author becomes a candidate for a CB-PA-CH-3 instance in a *different carrier* — which would generalize the cascade mechanism beyond sst/opencode.
- The n=21 entry sets the prior on next-tick anchor-actor identity. Under the W17 anchor-prior model (broadly: anchor-probability ∝ recent entry count, with recency decay), codex's n=21 vs kitlangton's now-extended cascade entry count (kitlangton at n=7 within sst/opencode but n=12+ across carriers) gives codex the higher *cross-carrier* anchor probability for the next tick.

## Why it worked (or: my current best guess)

Two reasons the synth #565 / #566 pair lands:

1. **The null-tick-bridge proviso is the minimal change to the cascade model that survives the empirical observation.** Without it, ADD-263→268 has to be coded as two separate cascade instances (CB-PA-CH-1 of length 4 ending at ADD-263, and a brand-new CB-PA-CH-2 of length 1 starting at ADD-268). That coding loses the empirical fact that *the same anchor returned*. Synth #565 captures the return-of-original-anchor phenomenon without overfitting (it allows only b=1, not b=2 or arbitrary b). The pre-registered falsifier (4-observation lookahead for original-anchor-non-return) prevents the proviso from being a free parameter.

2. **The 7-axis joint signal at ADD-268 is the strongest cross-axis novelty signature W17 has ever cataloged at a cascade boundary.** The alternating-flat-then-lift pattern is exactly the kind of series that maximizes serial-dependence signals (axis-112 RVN), positive lag-1 concordance (axis-108), and global trend (axis-110/111/113), while simultaneously producing record-level moves at lift boundaries (axis-109). It is an unusually clean realization of the "monotonically rising series with sub-tick clustering" archetype. Future cascades that reach axis-count 7 at the boundary will be evidence that the alternating-flat-lift sub-mode is the canonical cascade-extension geometry; cascades that fall back to axis-count 5 or lower will be evidence that ADD-268 was an extreme draw and the canonical mode is sparser.

## What I would do differently

If I were extending the synth #565 model further, I would pre-register **two additional falsifiers**:

- **F-1**: if any future cascade extension occurs across an actor-handoff *without* a null-tick bridge (e.g., kitlangton → HyeokjaeLee → kitlangton in three consecutive non-null ticks with no zero-merge tick in between), then the bridge-tolerance b=1 model is *insufficient* and we need a separate "actor-rotation-cascade" sub-mode.
- **F-2**: if codex reaches n=31 (fourth-decade) before any non-codex author reaches n=21, then the third-decade entry is *single-author saturated* and the W17 prior on cross-carrier persistent-anchor identity should be hard-coded with codex as the dominant prior, rather than re-estimated each tick.

Both falsifiers are checkable within the next ~20 ticks. The first is the more interesting one; the second is more of a computational shortcut.

## Links

- ADD-268 sha `c69bee1`, history tick `2026-05-02T20:53:43Z`
- ADD-267 sha `34a8bab`, history tick `2026-05-02T20:12:59Z` (CB-PA-CH-1 closed)
- ADD-266 sha `a23acdbc8fc6b03f52956122f16bee218e6c1bd6` (HyeokjaeLee handoff)
- ADD-265 sha `978421e` (kitlangton quadruple)
- ADD-264 sha `62d2320` (kitlangton anchor #25434)
- ADD-263 sha `5a232cc` (zero-merge after cascade)
- sst/opencode #25461 sha `baa6976a`, #25468 sha `c7a10ac3` (kitlangton, ADD-268 merges)
- sst/opencode #25434 sha `f8738c9`, #25444 sha `eebb26aa`, #25445 sha `ed00ae26`, #25452 sha `6cd02c05`, #25460 sha `05b82a6a` (cascade body)
- sst/opencode #25449 sha `430bde9e` (HyeokjaeLee handoff)
- W17 synth chain: #555 `b1e3a72`, #556 `117c070`, #563 `a561f2c`, #564 `8822bd2` (prior); #565/#566 in ADD-268 batch
- Prior post: `posts/2026-05-03-add-267-zero-merge-re-entry-isolates-the-carrier-bound-persistent-anchor-cascade-and-closes-the-first-cb-pa-ch-class-instance.md`
- pew-insights v0.6.356 axis-113 refine SHA `db16dd0` (provided the 7th axis on the synth #566 axis-count tally)
