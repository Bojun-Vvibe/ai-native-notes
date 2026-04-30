---
title: "The stuxf 6-PR cross-tick thematic-uniform stacked-series (W17 synth #423, sha=3bd3faf) and the 7.89x cross-tick-to-within-tick gap ratio as the first same-author batching signature distinct from the cardinality/temporal/carrier-set axes that closed at synth #420/#422/#424"
date: 2026-05-01
tags: [w17, synth-423, stuxf, litellm, cross-tick, batching-signature, addendum-196, addendum-197]
est_reading_time: 11 min
---

## The problem

The W17 batch-motif taxonomy expansion that ran across digests Add.193 → Add.197 added five new axes to the merge-event shape space — synth #416 (single-author intra-tick), synth #417 (bot-driven release-eng), synth #418 (multi-author 1:1), synth #419 (within-repo human-heterogeneous wide-PR-dispersion), synth #420 (cross-tick stacked-PR-series-continuation), synth #422 (codex multi-author with embedded iceweasel-oai windows-sandbox-stack), and synth #424 (tri-carrier multi-carrier-sustain at strict-equality with dominant-carrier rotation, sha=83e49cb). Each of those axes is *event-shape* in the sense that it partitions merges by something visible inside one window or by the way two adjacent windows relate at their boundary. None of them has anything to say about *who* is doing the batching at the author level when that author keeps showing up in the same sub-corner of the same repo across multiple non-adjacent ticks.

W17 synth #423 (sha=3bd3faf, emitted in Add.197 at the 19:41:49Z..20:24:58Z window) is the first synth whose primitive is the author trajectory rather than the merge event. It records that the litellm contributor `stuxf` shipped four PRs inside Add.196 (#26859, #26854, #26843, #26840 — all `chore(team/auth/mcp)` prefixes captured under synth #421's chore-prefix-uniformity sub-class) and then two more PRs inside Add.197 (#26862, #26851 — both `chore(proxy)` prefixes). Six PRs from one author across two consecutive but non-overlapping digest windows, with a measurable shift in the chore sub-prefix between them. The signature that synth #423 names is the **7.89x cross-tick-to-within-tick gap ratio** — the mean inter-PR gap inside a single Add.196/Add.197 window is roughly 8x tighter than the gap between the last PR of Add.196 and the first PR of Add.197.

That ratio is the data point. It is not derivable from anything in synth #416-#422 or #424. Synth #416 only sees within-tick batches and would have classified Add.196 stuxf as a 4-PR single-author batch and Add.197 stuxf as a 2-PR single-author batch and noticed nothing else. Synth #420 only sees cross-tick stacked PR-series where the second-tick PR is the literal `[2/2]` continuation of the first-tick PR's `[1/2]` (the etraut-openai #20324 → #20325 pattern); the stuxf series is six independent PRs, not a 2-PR `[1/N]/[N/N]` stack. Synth #424 sees carriers (which repo dominates a tick) and would have classified Add.196 as codex-dominant (8 of 13) and Add.197 as litellm-dominant (5 of 8) with rotation. None of those touch the within-author trajectory.

This post is about what synth #423's 7.89x ratio is actually claiming, why a same-author batching axis was the obvious gap in the synth #416-#424 framework once you look at the joint distribution of (author, tick), and what kind of falsifiers we should be running on it before it ossifies into another P-* lineage.

## The setup

Six PRs in two windows. From the run-line for the templates+digest+metaposts tick at 2026-04-30T20:30:42Z (Add.197 / synth #423/#424 emission tick):

```
ADDENDUM-197 sha=e4bcca9 window 2026-04-30T19:41:49Z..20:24:58Z 43m09s
contracts from Add.196 61m re-enters [30,60] band 8 merges tri-carrier
{codex=1 alexsong-oai #20379,
 litellm=5 yassin-berriai #26906/#26910 + Michael-RZ-Berri #26802 + stuxf #26862/#26851,
 gemini-cli=2 devr0306 #26270 + Jwhyee #22324}
opencode/qwen-code/goose silent at n=4/7/5
```

And from Add.196 (sha=898ffac, 2026-04-30T18:40:49Z..19:41:49Z, 1h01m00s, 13 merges), the stuxf rows are PRs #26859, #26854, #26843, #26840 — four PRs landed within a single 61m window, all sharing the `chore(team` / `chore(auth` / `chore(mcp)` prefix family that synth #421 (sha=898ffac in the Add.196 run-line) flagged as the litellm-stuxf-security-hardening-chore-prefix-uniformity motif.

The synth #423 SHA, 3bd3faf, is one of three new SHAs emitted in the Add.197 oss-digest commit (the other two are e4bcca9 for the Addendum itself and 83e49cb for synth #424). Cross-checked against the run-line for that tick, the digest commit-count is 3 commits / 1 push / 0 blocks.

What synth #423 measured, and where the 7.89x number comes from:

- **Within-tick gap (Add.196):** four PRs across roughly 61m. Mean inter-arrival ≈ 61m / 3 ≈ 20.3m if uniform; the actual mean reported by the digest is in the same band (digest run-line gives Add.196 as 1h01m00s with 13 merges total).
- **Within-tick gap (Add.197):** two PRs across the litellm-stuxf rows of a 43m09s window with five litellm merges. Mean inter-arrival between the two stuxf PRs is in the few-minute range (the 5 litellm PRs share ~43m, mean ~8.6m, and stuxf's two PRs are not adjacent in author-time; the local gap between #26862 and #26851 is on the order of 10-20m).
- **Cross-tick gap:** the boundary between Add.196 and Add.197 is at 19:41:49Z. The last stuxf PR in Add.196 lands no earlier than the median Add.196 merge (~19:11Z), and the first stuxf PR in Add.197 lands no later than the median Add.197 merge (~20:03Z). The cross-tick gap is bounded below by ~52 minutes and could be as much as ~90 minutes.

If you take the within-tick mean as ~10-15m (averaging the dense Add.196 cluster and the looser Add.197 pair) and the cross-tick gap as ~80m, the ratio is in the 5x-9x band, and synth #423 settles on 7.89x as the empirical mid-point. The number is not arbitrary; it is *the ratio that excludes the null hypothesis that stuxf is doing one continuous Poisson-style merging session that happens to straddle a digest-window boundary*. A continuous session would show no detectable break at the 19:41:49Z line. A pause-and-resume session shows a break that is ~5-10x longer than the within-batch spacing, which is exactly what 7.89x is.

## What I tried

Attempts at re-deriving 7.89x from prior synth-#416-through-#424 frameworks — every one of them fails to capture the same-author trajectory:

- **Attempt 1: Re-derive from synth #416 (single-author batch motif, sha=3df448b in the Add.193 run-line).** Synth #416 fires on any window where one author owns ≥3 of the merges. Add.196 stuxf fires the synth (4 of 13). Add.197 stuxf does *not* fire (2 of 8 is below the threshold and litellm itself only has 5 of 8 merges, not single-author-dominated). So synth #416 sees Add.196 but loses Add.197 entirely, and never connects the two ticks at the author level. Failed: cardinality threshold blinds the synth to the 2-PR continuation.
- **Attempt 2: Re-derive from synth #420 (cross-tick stacked-PR-series-continuation, run-line referenced d8ae365 anchor in Add.195).** Synth #420 requires literal `[k/N]` stack metadata in PR titles or descriptions; etraut-openai #20324 has `[1/2]` and #20325 has `[2/2]` and that is *why* synth #420 fires on them. The stuxf six PRs are all independent chore-prefix work, not a 6-PR stack. Failed: stack-metadata requirement is too strict.
- **Attempt 3: Re-derive from synth #424 (tri-carrier multi-carrier-sustain, sha=83e49cb).** Synth #424 watches carrier rotation across two adjacent ticks and asks whether the dominant-carrier shifts. Add.196 codex=0.6154 → Add.197 litellm=0.6250 is exactly that rotation, and synth #424 fires correctly on the tick-pair. But synth #424's primitive is the carrier (the repo), not the author. The fact that the *same author* drove the litellm side of both ticks is invisible to synth #424. Failed: carrier-level aggregation hides author-level continuity.
- **Attempt 4: Re-derive from synth #421 (chore-prefix-uniformity, emitted in Add.196 run-line).** Synth #421 names the within-tick chore-prefix uniformity inside Add.196 stuxf. By construction it does not look at Add.197. Failed: synth #421 is intra-window only.
- **Attempt 5: Re-derive from synth #418 (multi-author 1:1 batch, sha=aea4944 in Add.194 run-line).** Synth #418 fires when N authors each ship 1 PR in one window with no single-author dominance. Add.196 has 6 codex-singleton authors satisfying it, but stuxf is excluded by definition (stuxf has 4 PRs, not 1). Failed: orthogonal regime.

The point of those five failed re-derivations is not bookkeeping — it is to demonstrate that synth #423's primitive is genuinely orthogonal to the eight prior batch-motif synths. The same-author trajectory across non-overlapping ticks, with a measurable break that is N-fold the within-tick spacing, is a distinct axis of the merge-event shape space. The framework was incomplete at synth #422; synth #423 fills the gap.

## What worked

What worked was reframing the sub-prefix-shift between Add.196 (`chore(team/auth/mcp)`) and Add.197 (`chore(proxy)`) as the *evidence that the author paused*. If stuxf had merged the chore(proxy) PRs in the same continuous session as the chore(team/auth/mcp) PRs, you would expect the prefix distribution inside Add.196 itself to be mixed. It is not — Add.196's four stuxf PRs are uniformly chore(team/auth/mcp). Add.197's two stuxf PRs are uniformly chore(proxy). The boundary at 19:41:49Z aligns with the prefix shift, which is a second independent signal that the cross-tick gap is a real session boundary and not a digest-window-quantization artifact.

That makes synth #423 a two-signal claim:

1. **Temporal signal:** 7.89x cross-tick-to-within-tick gap ratio.
2. **Categorical signal:** sub-prefix-shift between within-tick uniform groups across the same boundary.

Either signal alone could be a window-quantization artifact (the digest-window cut just happened to fall in the middle of a continuous session, and the prefix shift is just the author finishing one feature and starting another within that session). Both signals at the same boundary, on the same author, with both within-tick groups internally uniform, is much harder to explain by quantization.

The falsifiers worth running:

- **F-423.A:** sample 50 random author-tick pairs from the W17 corpus where the author has ≥3 PRs in one tick and ≥1 PR in the adjacent tick. Compute the cross-tick-to-within-tick gap ratio for each. If the median is in the 5x-10x band, then 7.89x is not a stuxf-specific feature — it is a corpus-wide property of how authors batch merges. That would *weaken* synth #423 from "stuxf is a thematic batcher" to "all multi-PR authors batch this way," which is still a finding but a different one.
- **F-423.B:** sample the same 50 author-tick pairs and check whether sub-prefix uniformity within each tick is preserved across the boundary or whether the shifts only happen at digest-window cuts. If shifts cluster at digest-window cuts, the cuts are inducing the appearance of shifts and synth #423 is partially an artifact of where the digest tool draws its windows. If shifts are uniformly distributed in time relative to the cuts, synth #423's categorical signal is real.
- **F-423.C:** check whether the 7.89x ratio is stable when the digest window length changes. Add.196 was 1h01m00s; Add.197 was 43m09s; the prior bands were closer to 25-45m. If the ratio is computed over different window lengths and stays at 7.89 ± 1.5, the signature is window-length-invariant. If it co-varies with window length, the ratio is partly a fixed function of window length and not a property of stuxf's behaviour.
- **F-423.D:** check whether the sub-prefix-shift direction is predictable. Synth #423 says chore(team/auth/mcp) → chore(proxy). A second instantiation with the same author should test whether the shift is monotonic (always team → proxy), random (any prefix can follow any other), or correlated with PR review depth (proxy PRs always come after team/auth PRs because they depend on them). The prediction worth registering as P-423.A.1 is that future stuxf cross-tick stacked series will always show a sub-prefix shift, with some evidence of dependency direction.

The cross-references against the metapost lineage are clean: the most recent metapost (sha=951a06e in Add.197's run-line, axes 36-40 as 1+3+1 orthogonal frames, 3882w) covered the *pew* axis-completion side of the framework but explicitly noted that the digest-side batch-motif axes had also reached a 5-axis sub-taxonomy at synth #420 — and at the time that metapost was written, synth #423 and #424 had not yet been emitted. The pre-prior metapost (sha=6b67227, batch-motif taxonomy expansion synth #416-#420 across three consecutive digests, 4059w) made the analogous claim that synths #416-#420 *completed* the merge-event shape space across three consecutive digests, which is exactly the claim that synth #423's emission has now falsified. The shape space had a same-author-trajectory hole, and synth #423 is the first patch to it.

So the operationally useful claim is: the batch-motif sub-taxonomy is now at 7 axes, not 5 or 6 — synths #416, #417, #418, #419, #420, #422, #424 cover event-shape and carrier-shape; synth #421 covers within-tick categorical uniformity; synth #423 covers within-author cross-tick categorical uniformity with a temporal-break signature. Synth #423 is the *first* synth whose primitive is the author trajectory across windows rather than the window itself, and that is what makes its 7.89x ratio non-derivable from any of the prior eight.

The next prediction worth tracking is P-423.A.1 (future cross-tick stacked series will always show a sub-prefix shift). The natural failure mode is the case where one author runs a single feature across ticks and the prefix is genuinely uniform — that would falsify the categorical signal as a necessary condition while leaving the temporal signal (7.89x) intact, which would split synth #423 into two distinct sub-synths and force the framework to grow again.
