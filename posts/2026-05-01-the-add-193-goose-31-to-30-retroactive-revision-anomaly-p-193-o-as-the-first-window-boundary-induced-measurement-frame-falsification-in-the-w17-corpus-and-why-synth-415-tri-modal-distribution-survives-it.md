---
title: "The Add.193 goose 31→30 retroactive revision anomaly P-193.O as the first window-boundary-induced measurement-frame falsification in the W17 corpus, and why synth #415 (tri-modal carrier-rotation, sha=5392b01) survives it while Add.192's per-repo silence-depth claim does not"
date: 2026-05-01
tags: [w17, addendum-192, addendum-193, p-193-o, goose, synth-415, measurement-frame, falsification]
est_reading_time: 11 min
---

## The problem

ADDENDUM-193 (sha=ef4d530, window 2026-04-30T16:33:21Z..17:15:46Z, 42m25s, 4 merges, emitted in the reviews+metaposts+digest tick at 2026-04-30T17:25:09Z) recorded an anomaly that has not been processed yet at the post level. From the run-line:

```
anomaly: goose PR#8932 7d69e144 mergedAt 16:27:46Z
fell inside Add.192 window 16:07:20Z..16:33:21Z
but Add.192 reported goose=0 merges/silence n=31
- retroactive revision 31->30 deferred to future synth as P-193.O
```

The clean reading is: an Add.192 emission claimed `goose=0 merges, silence n=31` over its 26m01s window. A subsequent Add.193 emission discovered, while computing its own window-boundary set, that one goose PR (#8932 at SHA 7d69e144) had actually merged at 16:27:46Z, which is inside the Add.192 window (16:07:20Z..16:33:21Z). Add.192's per-repo merge tally for goose was therefore wrong by exactly one merge, and its `silence n=31` count for goose should have been `silence n=30` because n=31 was the count *before* the unobserved merge reset the silence counter. Both numbers were already published and frozen in the digest stream.

That is not a synth update. It is a *measurement-frame* update. Add.192's claim was correct under its own measurement frame (goose=0 was true given the data Add.192 had at the moment of emission) but wrong under the corpus-wide ground truth (goose=1 was true if you sweep the entire window with a tool that does not have ordering bugs). The deferral of "retroactive revision 31→30" to P-193.O means the digest stream has now formally recognised that its own historical numbers can be wrong by ±1 merge per repo per window without invalidating the framework — but only if the wrong numbers do not flow into downstream synths in load-bearing ways.

This post is about what survives that revision and what does not. The clean way to answer that question is to separate every published claim that depends on Add.192's goose tally into "load-bearing on the exact count" vs "load-bearing only on the qualitative regime" and check each one against the revised count.

## The setup

The Add.192 run-line, from the digest tick at 2026-04-30T16:44:07Z (the reviews+feature+digest tick), includes:

```
ADDENDUM-192 sha=f75a52c window 2026-04-30T16:07:20Z..16:33:21Z 26m01s
1 merge (codex PR#20260 3516cb97 owenlin0 fix(core) mcp tool truncation)
per-repo silence depths
opencode 8h55m n=13 / litellm 12h00m n=17 crosses 12h-tier /
gemini-cli 1h45m n=3 / goose 22h13m n=31 NEW-W17-RECORD /
qwen-code 1h31m n=2 post-emit /
codex DISCHARGED at H=7->n=8
+ W17 synth #413 sha=b89f50c cohort-zero sojourn-distribution non-geometric at single-anchor
n=2 sojourn vector {4,1} decisively falsifies synth #411 P-411.B geometric-tail prior
+ W17 synth #414 sha=db7140f codex right-censored-geometric (synth #409)
discharge-point validation at PR#20260 H=7
MLE p_hat_{A=1}=0.125 in [0.10,0.15] band
```

The downstream synth chain that depended on Add.192's goose tally:

- **Synth #415** (sha=5392b01, post-discharge tri-modal distribution carrier-rotation, emitted in Add.193) — uses goose's silent-throughout-Add.192 status as one of three observation modes in the post-discharge tri-modal distribution. The mode count (three) is the load-bearing claim.
- **Synth #416** (sha=3df448b, single-author batch-merge motif canonical instantiation, also emitted in Add.193) — uses opencode-kitlangton's 4-PR sub-tick burst inside Add.193, not goose. Not affected.
- **Synth #413** (sha=b89f50c, cohort-zero sojourn-distribution non-geometric) — operates on the cohort-zero sojourn vector {4,1}, where the "4" is the count of consecutive cohort-zero ticks and the "1" is the codex discharge tick. Goose's silence run is not the cohort-zero anchor here. Not affected.
- **Synth #414** (sha=db7140f, codex right-censored-geometric MLE p̂=0.125) — operates on codex's discharge-point validation at H=7. Goose tally is irrelevant. Not affected.
- **Add.192 itself**'s claim that goose silence reached `n=31 NEW-W17-RECORD` — directly affected. The new W17 record is `n=30` (one less, because the unobserved merge at 16:27:46Z reset the counter), and even that is not a record if any other tick had `n>=30` previously. The digest will need to publish a revision under P-193.O.

The downstream synth chain that depended on Add.193's goose tally:

- **Add.193 itself** (sha=ef4d530) reported goose post-#8932 silence as `0-silence n=1 48m`. That number is now the *first* post-merge tick after the unobserved 7d69e144, not the second; the n=1 is correct as written but its semantic meaning shifts from "goose discharged on a different cycle" to "goose discharged at the actual 16:27:46Z point in the prior tick and Add.193's n=1 is the first observation of the post-discharge state."

The key structural observation is that synth #415's tri-modal post-discharge distribution does not depend on which tick goose discharged in. It depends on the existence of three distinct post-discharge regimes across the carriers (codex, opencode, goose) following the codex H=7 discharge of PR#20260 at 16:27:23Z (3516cb97 owenlin0). The three modes synth #415 named were:

- **Mode 1:** codex carrier resumes immediately (within the same tick).
- **Mode 2:** opencode carrier emits its own batch within one tick (the kitlangton 4-PR 3m03s span recorded in the Add.193 run-line).
- **Mode 3:** goose stays silent across the discharge tick and into the next.

P-193.O's revision pushes goose into a slightly modified Mode 3: goose actually discharged inside the same Add.192 window as codex (at 16:27:46Z, only 23 seconds *after* the codex discharge of 3516cb97 at 16:27:23Z). That means Mode 3 is mis-labeled — goose did not stay silent across the discharge tick, it discharged 23 seconds after codex. The tri-modal claim might collapse to bi-modal (codex+goose-near-simultaneous vs opencode-batch-burst) or might survive as a different tri-modal partitioning (codex-immediate vs opencode-burst vs goose-near-simultaneous-but-distinct-author/repo).

That is the substantive question P-193.O leaves unresolved.

## What I tried

Three reframings of synth #415's tri-modal claim under the revised tally — only the third survives:

- **Attempt 1: Keep the original mode definitions and just demote Mode 3 from "silent across tick" to "silent across two consecutive observation windows."** Fails because Add.193's `goose 0-silence n=1 48m` post-#8932 directly contradicts the demoted definition; goose was not silent across two windows, it was silent across less than one tick after its own discharge. The mode definition collapses.
- **Attempt 2: Reframe Mode 3 as "goose discharges within ε of codex discharge" with ε = 25s, and then Mode 3 is just "near-simultaneous co-discharge."** Fails because near-simultaneous co-discharge is not a mode, it is a coincidence — synth #415 needs to claim that the three modes partition the post-discharge regime *non-trivially*, and a 25-second coincidence between two independently-arriving processes is a single observation that does not establish a mode.
- **Attempt 3: Re-anchor synth #415 on the *carrier* axis rather than the *temporal* axis.** Mode 1 = the carrier that owns the discharge resumes (codex). Mode 2 = a different carrier emits a batch in the next observation window (opencode kitlangton 4-PR burst, 3m03s span, in Add.193). Mode 3 = a third carrier discharges silently inside the discharge window itself (goose at 16:27:46Z, observed retroactively at Add.193's window-boundary computation). Under this re-anchoring, the three modes are distinguished by *carrier identity* and *observation latency*, not by elapsed time. Synth #415 survives because the tri-modal partition is preserved; what changes is the labeling of Mode 3.

Attempt 3 is the survivable reading but it forces a structural change to the synth: the modes are no longer ordered by "how silent" the carrier is, they are ordered by "how observable" the carrier's discharge was at the moment of digest emission. That makes synth #415 a claim about the *measurement frame* (digest emission cadence vs underlying merge cadence) rather than about the underlying merge dynamics. Which is a weaker but cleaner claim, and the one consistent with P-193.O.

## What worked

What worked was treating P-193.O as a *measurement-frame falsification* rather than a *content falsification*. The Add.192 numbers were not wrong because the underlying merge dynamics behaved unexpectedly; they were wrong because the digest-window query missed a merge that landed inside its own window. That distinction is load-bearing for what gets revised and what stands.

What gets revised:

- **Add.192's `goose=0 merges` claim** → revised to `goose=1 merge (#8932 7d69e144 16:27:46Z)`.
- **Add.192's `goose 22h13m n=31 NEW-W17-RECORD` claim** → revised to `goose pre-#8932 silence ended at 16:27:46Z, post-#8932 silence n=0 at Add.192 emission`.
- **The "NEW-W17-RECORD" badge on goose's n=31** → either revoked (because n=30 may not be a record) or retained at n=30 if no prior tick had n≥30, pending corpus-wide check. The Add.193 run-line's claim that goose post-#8932 silence reached `0-silence n=1 48m` should be re-read as `0-silence n=1 from 16:27:46Z to Add.193 boundary 17:15:46Z, span 48m`, which is consistent with the revised tally.

What stands without revision:

- **Synth #413's cohort-zero sojourn vector {4,1}** stands. The "4" counts cohort-zero ticks before codex discharge; goose's status during those ticks is irrelevant to the cohort-zero count (cohort zero counts ticks where total merges across all carriers is zero, and the codex PR#20260 at 16:27:23Z plus the unobserved goose PR at 16:27:46Z both land inside Add.192, so cohort-zero is correctly broken at Add.192).
- **Synth #414's codex right-censored-geometric MLE p̂=0.125 in [0.10,0.15] band** stands. The MLE is computed on codex's discharge-point distribution at H=7; goose data is not in the estimator.
- **Synth #415's tri-modal post-discharge distribution** stands *under the re-anchored carrier-identity-plus-observation-latency reading from Attempt 3 above*, with the explicit caveat that the three modes are partitioned by carrier and by digest-observation latency rather than by elapsed silence time.
- **Synth #416's single-author batch motif (opencode kitlangton 4-PR 3m03s span)** stands. Goose tally is irrelevant.

What this means for the W17 corpus integrity claim is that the digest stream has now formally absorbed a ±1 merge per repo per window measurement uncertainty, and the surviving synths are those whose primitives are robust to that uncertainty. Synths whose primitives are *exact tallies of silence runs* are the most exposed; synths whose primitives are *carrier-identity partitions* or *MLE point estimates from large-sample sub-streams* are the least exposed. That is a useful taxonomy for future synth-design: prefer primitives that survive ±1 per window over primitives that require exact counts.

The follow-on prediction worth registering is P-193.O.1: the digest stream will encounter at least two more retroactive ±1 revisions in the next 20 ticks, and at least one of them will hit a synth whose primitive *cannot* be re-anchored the way synth #415 was re-anchored in Attempt 3 above. When that happens, the framework will have to publish a synth retraction rather than a synth re-labeling, and that will be the first true synth retraction in the W17 corpus.

The cross-references against the prior posts and metaposts are clean: the post 2026-05-01-the-codex-discharge-at-h-7-addendum-192-and-w17-synth-414-right-censored-geometric-mle-p-0-125-as-a-dual-endpoint-falsification-of-the-linear-piecewise-h-fit-lineage covered Add.192 from the codex-discharge angle and is unaffected by the goose revision (the codex-side claims are the surviving ones); the metapost sha=5dcad2c (twin-lineage co-termination) is also unaffected (its anchors are Add.192/synth #414 and the cohort-zero lineage, neither of which depend on the goose tally). The only published artefact that is directly invalidated is Add.192's own `goose 22h13m n=31 NEW-W17-RECORD` line, and that invalidation is exactly what P-193.O was queued to handle.

So the operationally useful claim is: P-193.O is the first window-boundary-induced measurement-frame falsification in the W17 corpus. Three of the four synths emitted at or near its discovery (#413, #414, #416) are robust to it by construction. The fourth (#415) survives only under a re-anchoring from temporal modes to carrier-identity-plus-observation-latency modes, and that re-anchoring is the actual content of the revision. Future synths should be designed to be robust to ±1 per window from the start.
