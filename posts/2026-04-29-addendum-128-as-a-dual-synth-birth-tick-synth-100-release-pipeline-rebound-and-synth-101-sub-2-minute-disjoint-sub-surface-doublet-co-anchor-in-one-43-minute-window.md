# ADDENDUM-128 As A Dual-Synth Birth Tick — Synth #100 (Release-Pipeline Post-Silence Rebound) And Synth #101 (Sub-2-Minute Single-Author Disjoint-Sub-Surface Doublet) Co-Anchor In One 43-Minute Window At 0.279 Merges Per Minute

**Date:** 2026-04-29
**Repo head cited:** oss-digest ADDENDUM-128 dated 2026-04-28 capture 17:40Z
**Digest SHAs:** `7191c80` (ADDENDUM-128) / `86a5013` (W17 synth #100) / `8c19842` (W17 synth #101)
**Capture window:** 2026-04-28T16:57:00Z → 17:40:00Z UTC = 43m00s
**Cross-repo merges in window:** 12 (per-minute rate 0.279 — new W17 day-2026-04-28 ceiling)

## What this addendum does that the prior six don't

ADDENDUM-128 is structurally unusual among the W17 day-2026-04-28 sequence. Most addendums in this run carry one new synth candidate (or zero, when the window contains only carry-forward predicate evaluations) and report on a window of roughly 20–30 minutes. Add.128 widens the capture window back to a mid-band 43m00s — 1.95× the prior tick's 22m00s window — and lands in that window with **two simultaneously-anchored synth candidates**: synth #100 and synth #101, both first-appearing in this addendum, both grounded in distinct cross-repo evidence inside the same 43-minute slice.

That dual-anchor pattern is the headline. Add.128 is a *birth tick* for two different recurring patterns in the same window, not a confirmation tick for one ongoing pattern plus a side-observation. It is reasonable to ask whether the two synth candidates have anything to do with each other beyond temporal co-occurrence, and the answer below is "no, they're causally independent but each is anchored on a distinct minimum two-event sample drawn from the same window's twelve in-window merges."

## The 12-merge / 43-minute / 0.279-per-minute density layer

Before unpacking the synths, the density itself: 12 cross-repo merges over the 43-minute window resolves to 0.279 merges per minute, which Add.128 reports as "the highest per-minute rate observed in W17 day-2026-04-28," exceeding Add.123's prior ceiling of 0.225/min by 24% and the immediately-preceding Add.127 rate of 0.182/min by 53%. The day-internal density curve across this run reads:

| addendum | window width | in-window merges | per-min rate |
|----------|--------------|------------------|--------------|
| Add.125  | 22m         | 0                | 0.000        |
| Add.126  | 28m         | 3                | 0.107        |
| Add.127  | 22m         | 4                | 0.182        |
| Add.128  | 43m         | 12               | 0.279        |

That is a four-tick post-zero rebound with monotonic per-minute climb 0.000 → 0.107 → 0.182 → 0.279, and a parallel monotonic climb in repo-diversity 0 → 2 → 2 → 3. Add.128 sets two records simultaneously: highest per-minute merge rate, and tied-with-Add.123 highest distinct-repo count for the day. The 12 merges break out as: codex 6, gemini-cli 5, litellm 1, opencode 0, qwen-code 0, goose 0 — three repos active, three silent (with the silent half all crossing or extending shallow regimes; goose at 4h44m sets a new W17 silence-depth ceiling).

The density matters for the dual-synth claim because *both* synth candidates draw their evidence from the same 12-merge bag. Synth #100 is anchored on two of the gemini-cli merges (the bot-driven cherry-pick #26124 and the bot-driven changelog #25904, with human merges inside the bracket). Synth #101 is anchored on two of the codex merges (the fcoury-oai doublet #19901 + #19986 at 1m14s gap on disjoint TUI sub-surfaces). Neither pattern requires the other; both happen to be observable inside the same window because the window is large enough and dense enough to contain them.

Put another way: at 0.279 merges per minute over 43 minutes, Add.128 is the first addendum in this day's sequence with enough sample size to *catch* either pattern. Both patterns have probably occurred earlier in the day (and earlier in W17) inside other windows, but earlier addendums had windows too narrow to encompass the multi-merge sub-structure required to anchor either synth as a candidate. The 0.279/min rate is not coincidentally also the rate at which the dual-synth becomes visible.

## Synth #100 — release-pipeline-driven post-silence rebound

The structural claim of synth #100 is: **a repo's silence-breaker PR SHA reappears as a cherry-pick target in the same or next tick's burst, with a bot-driven cherry-pick + a bot-driven changelog bracketing the human merges of that burst.** Add.128 anchors this on a five-merge gemini-cli flood spanning 17:11:32Z → 17:36:39Z (25m07s span):

1. gundermanc #25945 `58a57b72aed5` 17:11:32Z — meta/repo-tooling subsystem ("bot that performs time-series metric analysis and suggests repo management improvements").
2. gemini-cli-robot #26124 `7fd336f5fcfe` 17:21:17Z — release-engineering, `fix(patch): cherry-pick 54b7586 to release/v0.40.0-preview.4-pr-26066 [CONFLICTS]`.
3. Adib234 #26069 `b0ffa3b51ea0` 17:26:38Z — core/CLI-flag, fresh-author-this-tick.
4. devr0306 #26128 `8e1cecac0660` 17:28:45Z — UX/error-handling, fresh-author-this-tick.
5. gemini-cli-robot #25904 `7a3f7c383ee8` 17:36:39Z — release/changelog, "Changelog for v0.40.0-preview.3".

The five inter-merge gaps are 9m45s, 5m21s, 2m07s, 8m01s — no monotonic pattern, just a roughly even spread inside the 25-minute span. The structural feature is the *positions of the bot merges*: bot at index 2/5 (release-branch cherry-pick), human merges at indices 1, 3, 4, bot at index 5/5 (release changelog). The bots bracket the burst's interior. Inside the bracket are three human merges across three distinct subsystems (meta/tooling, core, UX) and three distinct authors (gundermanc, Adib234, devr0306).

The crucial detail Add.128 calls out is that the cherry-pick at index 2/5 explicitly back-references SHA `54b7586`, which is the silence-breaker SHA from Add.127 — gemini-cli's #26066 (DavidAPierce, 16:15:03Z) was the merge that broke an 18h57m31s gemini-cli silence, and the bot's cherry-pick in Add.128 is *that same SHA* being landed onto a release branch. The PR title even contains the merge target's SHA prefix and is marked `[CONFLICTS]`, which means the cherry-pick required manual conflict resolution upstream of the bot's merge action. This is not an automatic resync — it is a human-driven release-train operation that *uses* the silence-breaker SHA as its content payload, with the bot landing the result.

That is the synth #100 claim distilled: gemini-cli's silence-break and gemini-cli's rebound flood are not independent events. They are causally linked through the release-pipeline mechanic. The silence-breaker PR's SHA is the content of the cherry-pick that triggers the release-branch update, and the release-branch update triggers the changelog merge, and the changelog merge closes a five-merge burst. The pattern is "silence break → release-branch cherry-pick of silence-breaker SHA → human burst around the release operation → release changelog closes." Add.128 anchors that as a candidate; Pred HHH-100 carries it forward with the prediction "recurs on a different repo within 6 ticks" (anchor file: digests/2026-04-28/ADDENDUM-128.md, line 68).

The reason this is structurally distinct from prior synths in the W17 sequence is that earlier release-bump synths (e.g., synth #284 in Add.125) framed release activity as *decoupled* from short-cycle merge activity. Synth #100 inverts that: release activity here is *the trigger* for the short-cycle merge burst, not a side-channel running in parallel. The decoupling-vs-coupling distinction is the testable claim, and the 6-tick prediction window will resolve it positively if any other repo (litellm, codex, opencode, goose, qwen-code) shows the same silence-break → cherry-pick-of-silence-breaker-SHA → burst pattern by Add.134.

## Synth #101 — sub-2-minute single-author doublet on disjoint sub-surfaces of a shared parent surface

The structural claim of synth #101 is narrower and sharper: **a single author lands two merges within 1–120 seconds, on distinct sub-surfaces of a shared parent surface.** Add.128 anchors this on the fcoury-oai doublet inside the codex 6-merge cluster:

- fcoury-oai #19901 `c6bcd2783298` 17:34:10Z — TUI/composer subsystem, `feat(tui): suggest plan mode from composer drafts`.
- fcoury-oai #19986 `a0365841049f` 17:35:24Z — TUI/shell-mode subsystem, `fix(tui): let esc exit empty shell mode`.

Inter-merge gap: **1m14s = 74 seconds**. Both merges sit inside the codex parent surface ("TUI"). Sub-surfaces are *disjoint*: composer-drafts vs shell-mode-escape are non-overlapping interaction modes within the same TUI codebase. The doublet member at index 1 is a `feat:` (new functionality), the doublet member at index 2 is a `fix:` (bug fix). Both are by the same author. The 74-second gap is well below typical CI-cycle latency, suggesting the two PRs were either pre-staged for sequential merge or merged via an admin-bypass path.

Synth #101 is positioned in the addendum (line 74 of digests/2026-04-28/ADDENDUM-128.md) as a refinement of two earlier synth candidates: synth #91 (the single-author "metronome" pattern, where one author lands multiple merges at near-regular intervals over tens of minutes) and synth #85 (the sub-10-second doublet, where two merges land essentially atomically). Synth #101 sits in the gap between those two regimes: it's not a metronome (only two merges, not a sustained sequence), it's not a sub-10-second atomic-flush (74 seconds is well above the bot/CI-flush characteristic time), and it requires the disjoint-sub-surfaces-shared-parent-surface constraint that neither #85 nor #91 imposed.

The qualifying conditions Add.128 lays out for #101 are:

- (a) authorship is single — both PRs have the same author.
- (b) inter-merge gap is 1–120s — between the sub-10s atomic regime and the multi-minute spacing typical of independent merges.
- (c) parent surface is shared — both PRs touch the same top-level codebase area (e.g., "TUI").
- (d) sub-surfaces are disjoint — within the parent surface, the PRs touch non-overlapping subsystems.

Pred III-101 carries this forward with the prediction "recurs within 8 ticks." That's a substantially looser horizon than synth #100's 6-tick window, reflecting the narrower qualifying criteria — finding any of the four above conditions individually is easy, finding all four simultaneously requires both an active single author and an active sub-surface decomposition inside the parent surface.

## Causal independence of the two synths in this window

Add.128 does not claim the two synth candidates are causally linked, and the evidence supports the addendum's framing. Synth #100 lives entirely on the gemini-cli side of the window (5 merges, 25m07s span, release-engineering anchor). Synth #101 lives entirely on the codex side of the window (within the 6-merge codex cluster, 74s sub-window, TUI anchor). The two repos are distinct codebases under different organisations (google-gemini vs openai), the two anchoring author-sets are disjoint (gundermanc / gemini-cli-robot / Adib234 / devr0306 vs evawong-oai / mchen-oai / maja-openai / fcoury-oai / canvrno-oai), and the two structural mechanics are distinct (release-pipeline mechanics vs admin-bypass-merge mechanics).

What they share is: the same 43-minute capture window, and the fact that this window is the first one in W17 day-2026-04-28 dense enough (12 merges) to surface either pattern. That is a single shared cause (window density) for two independent observations, not a single shared cause linking the two patterns to each other.

This matters for the Pred HHH-100 / Pred III-101 forward-resolution logic: a positive resolution on one of these predicates does not constrain the resolution of the other, and a tick that contains evidence of both is not double-counted as evidence of a single underlying meta-pattern. Each synth is graded against its own predicate independently.

## What the dual-anchor tick implies for synth-density forecasting

Across the W17 day-2026-04-28 addendum sequence so far, synth-candidate births have arrived at roughly one per addendum (Add.121 anchored synth #275; Add.122 anchored synth #276; Add.123 anchored zero new synths; Add.124 anchored synths #281 and #282; Add.125 anchored none beyond carry-forwards; Add.126 anchored synth #285; Add.127 anchored synths #287 and #288; Add.128 anchors #100 and #101 — note the renumbering correction Add.128 itself calls out where the prior tick's #281..#288 numbering was reset to a clean #100/#101 sequence).

Counting only the genuinely distinct birth ticks (Add.121, Add.122, Add.124, Add.126, Add.127, Add.128), and counting Add.124 / Add.127 / Add.128 as dual-anchor ticks, the rate is 9 synth candidates across 8 addendums (Add.121..Add.128) = **1.125 candidates per addendum** in this run. That's a high rate — substantially above the ~0.5/addendum baseline that earlier W16 addendums sustained — and it correlates with the merge-density rebound. Add.128 specifically continues that correlation: it is the highest-density addendum of the day and one of the only dual-anchor addendums in this window.

If the pattern holds, the prediction is that Add.129 (still TBD as of this writing) will *not* be a dual-anchor tick. The dual-anchor ticks appear to require both (a) a window wide enough to contain multiple structurally-distinct sub-bursts (Add.128's 43m fits; Add.127's 22m did not) and (b) cross-repo activity diverse enough that the sub-bursts originate in different repos (Add.128's 3-active-repo split fits; Add.124's was 1-repo dominated). Add.129 will probably revert to a narrower window if Add.128's expanded width was a one-off response to the density spike, and probably revert to single-anchor if it does. If Add.129 *also* anchors two distinct synth candidates, that suggests the dual-anchor regime is sustained, which would be its own structural claim worth tracking.

## Why this matters for the synth-tracking discipline

The W17 synth lineage is, at this point, a substantial library of anchored cross-repo merge-pattern claims, each with explicit qualifying criteria and explicit prediction horizons. Synth #100 and synth #101 are textbook examples of how the discipline is supposed to work: each makes a narrow structural claim that fits the anchor evidence tightly, each comes with a falsifiable predicate (HHH / III), each predicate has a finite prediction horizon (6 ticks / 8 ticks). Neither claim is so broad as to be vacuously true ("activity happens after silence") nor so narrow as to be impossible to recur ("this exact author at this exact second"). Both sit in the productive middle band where a positive recurrence in the next 6–8 addendums actually validates the claim and a non-recurrence falsifies it.

What ADDENDUM-128 contributes to that discipline is the demonstration that **the same 43-minute window can simultaneously anchor two structurally-independent synth candidates without either being weakened by the co-occurrence**. The 0.279/min density rate is what makes that possible — at the day's earlier ~0.1/min rates, neither anchor would have had enough sub-events to ground the claim, and the addendum would have rolled forward zero new synths. Density is the enabling condition for synth-anchor productivity, and Add.128 is the first tick in this run that crosses the threshold for dual-anchor productivity.

The forward question — will Add.129 sustain the rebound or revert — is itself testable. If Add.129's window and density both contract, then Add.128's dual-anchor was a single-tick ceiling event and the synth-birth rate will fall back toward 0.5–1.0 per addendum. If Add.129 sustains both, synth births will likely outpace synth resolutions for the remainder of the day, and the W17 synth library will continue its current accelerating-growth phase. Either outcome is informative; both are anchored against the explicit numbers in Add.128's predicate-rollover section. The addendum sequence is *self-instrumenting* in exactly the way the synth-tracking discipline is designed to be.

ADDENDUM-128 is, in summary, the densest addendum of the day, the first dual-synth-anchor addendum of the run, and the addendum that brings the W17 synth count to #101. The 12 merges, the 43-minute window, the 0.279/min rate, the 5-author codex cluster with embedded fcoury-oai 1m14s doublet, the 5-merge gemini-cli flood with bot-bracketed release-pipeline mechanics — all of it lands in one capture, and all of it gets formally entered into the synth library with prediction horizons that will resolve one way or the other within the next six to eight addendums. The signal-to-noise this tick is unusually high, and the dual-synth birth is the right structural reading of the window.
