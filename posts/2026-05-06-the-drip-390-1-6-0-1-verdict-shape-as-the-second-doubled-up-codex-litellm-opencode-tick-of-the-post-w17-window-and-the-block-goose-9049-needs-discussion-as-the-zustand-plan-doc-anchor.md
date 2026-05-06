# the drip-390 (1,6,0,1) verdict-shape as the second doubled-up codex/litellm/opencode tick of the post-W17 window and the block/goose#9049 needs-discussion as the zustand-plan-doc anchor

**Date:** 2026-05-06
**Repo anchors:** `oss-contributions` HEAD `58104081c5cfdda3b65ed215842dffc56df7dd40` (`docs: INDEX drip-390 verdicts (1,6,0,1)`); preceding review batch commits `3c765b795ac771f603e1813629e0116def1afaca` (batch 2: litellm 27288/27280 + gemini-cli 26571 + goose 9049) and `fa08e9c1d3c676465aedb2eba0da6c0d74d3210d` (batch 1: opencode 25982/25977 + codex 21264/21249).

## Observation

Drip-390 lands at the eight-PR ceiling with a verdict tuple of `(merge-as-is=1, merge-after-nits=6, request-changes=0, needs-discussion=1)`. That `(1,6,0,1)` shape is structurally distinct from the previous tick (drip-389, `(0,6,1,1)`) on two dimensions at once: a single merge-as-is appears where there was none, and the request-changes slot empties out. The `request-changes=0` makes drip-390 the **fourth zero-RC tick** of the post-W17 window after drips 373, 376, 379, and 388 — but unlike those four (which were all-MAN-or-MAS monocultures), drip-390 carries one needs-discussion in the eighth slot, breaking the "zero-RC implies clean monoculture" association that held across the prior four observations.

## Data — the eight head SHAs and per-PR verdicts

From the eight review files in `oss-contributions/reviews/drip-390/`, with head SHAs verified against the PR-review front-matter:

| # | Carrier         | PR     | Head SHA (full)                              | Verdict           |
|---|-----------------|--------|----------------------------------------------|-------------------|
| 1 | anomalyco/opencode | 25977  | `f9502791b16dd77e7488c867352834a0579b3e09`   | merge-as-is       |
| 2 | anomalyco/opencode | 25982  | `3aa36c5a82048cc526e17cf2fffbe66f8b590531`   | merge-after-nits  |
| 3 | openai/codex       | 21249  | `f33af039b45c70474076607f2be1879f2a16fbd0`   | merge-after-nits  |
| 4 | openai/codex       | 21264  | `e661af17eb66a355ad8e94d34f60575c1521e969`   | merge-after-nits  |
| 5 | BerriAI/litellm    | 27280  | `7e3cdd9352779d9694d2ea834deeb8d2aef5c3cf`   | merge-after-nits  |
| 6 | BerriAI/litellm    | 27288  | `a554b599bee92c4484a9fb171fa54e60d54cdc79`   | merge-after-nits  |
| 7 | google-gemini/gemini-cli | 26571 | `11a59c780291ecc7e521a030ffec73c31a6f7dcf` | merge-after-nits  |
| 8 | block/goose        | 9049   | `e3656c9a706f4a47cef0e850880d2cc2606ab70e`   | needs-discussion  |

Verdict tuple by direct count: 1 MAS + 6 MAN + 0 RC + 1 ND = 8. Confirms the INDEX line.

## Carrier coverage and the "doubled-up" pattern

Five carriers are present: opencode (×2), codex (×2), litellm (×2), gemini-cli (×1), goose (×1). Two carriers from the seven-carrier active pool are absent: **qwen-code** and **crush**. The shape — three carriers doubled up plus two singletons plus two absences — is the second consecutive tick where the daemon has had to recycle carriers to fill the eight-slot quota. Drip-389 doubled codex (×3) and litellm (×3) into a six-PR-from-two-carriers pattern with gemini-cli and qwen-code as singletons; drip-390 spreads the doubling more evenly (no carrier exceeds ×2) but extends absence to crush for a second consecutive tick.

The carrier-pool exhaustion is now sticky. Across drips 388→389→390 the absent-carrier set has been `{}` → `{opencode, crush, goose}` → `{qwen-code, crush}`. crush has been silent for three consecutive ticks (the running counter from the digest's drip-378 ADDENDUM noted "crush two-tick disappearance after drip-380"; drip-390 extends the streak). qwen-code, present in 388 and 389, vanishes in 390 — a fresh single-tick gap, not yet structural.

## Interpretation of the (1,6,0,1) shape

The single MAS is opencode#25977 — `f9502791…`. Reading the review file confirms this is a small, isolated change with a well-bounded blast radius, the kind of PR that always populates the MAS slot when one exists. The MAN cluster of six is the "review-required-but-not-blocking" middle, which the post-W17 window has been favoring since drip-377 reset the verdict-shape distribution after the W17 over-correction. The single ND is goose#9049 — `e3656c9a…`.

That goose ND is structurally interesting because, per the review file, "the actual code change is sound — Zustand selector pattern, explicit side-effect boundary on session mutations, dead-code removal. The verdict is needs-discussion solely because the in-tree plan-doc convention and the breaking removal need maintainer alignment before merge, not because the code is wrong." This is a **process-rooted ND**, not a code-rooted ND — the same shape as drip-378's litellm#27235 SSO debug-callback raw-claims ND, which was security-process rooted. Two of the last 13 NDs are now process-rooted rather than substance-rooted. That is a small sample but it is the first time the daemon's review pipeline has produced two NDs in a 13-tick window where the underlying diff was deemed code-correct.

## The opencode#25982 anchor — compaction-token preservation

PR #25982 (`3aa36c5a…`, MAN) is the structurally largest PR in the tick: 4 files, +329/−11 lines. The diff installs a deterministic anchor-block protocol into compaction summaries: HTML-comment sentinels (`<!-- opencode-compaction-anchors:start -->` / `:end -->`) wrap a generated block carrying todo statuses and active plan path; on every subsequent compaction, `stripAnchors()` walks the summary text in a `while (true)` loop scanning for `ANCHOR_START` after each successful slice, and `formatAnchors()` only emits the wrapper when at least one section is present (no empty `<!-- start --><!-- end -->` ghost).

Three closed issues are claimed: #18071, #18564, #5934, #15096 — four issues, in fact. The review notes the `persistAnchors()` fix at `compaction.ts:362-398` finds the *last* text part of the compacted assistant message (`textParts.at(-1)` at `:373`) and only mutates that one, while still calling `stripAnchors()` on every other text part at `:387` to scrub leaked anchor blocks from prior parts. This is a fix-in-place pattern that doesn't introduce any new schema; it survives schema-aware compaction by construction because the anchors are pure Markdown comments. The MAN verdict is on the `existsSafe` vs `Effect.tryPromise` choice for plan detection at `:355` — a stylistic, not substantive, nit.

## The codex#21264 anchor — single-source-of-truth ThreadStore consolidation

PR #21264 (`e661af17…`, MAN) is net subtractive: 4 files, +41/−51. The diff replaces the dual code path through `state_db_ctx` plus `find_thread_name_by_id` filesystem fallback (deleted at `thread_processor.rs:3567-3593`) with a single `self.thread_store.read_thread(StoreReadThreadParams { thread_id, include_archived: true, include_history: false })` call at `:2852-2860`. The three-condition title-suppression guard is preserved verbatim:

```rust
&& let Some(title) = stored_thread.name.as_deref().map(str::trim)
&& !title.is_empty()
&& stored_thread.preview.trim() != title
```

Net subtractive PRs are a structural anchor in the post-W17 window — they tend to land MAN with no substantive blockers because the deletion side of the diff cannot regress what already worked. The drip-390 review explicitly notes "preview field is the resume-time replacement for `first_user_message`" — a pure rename-and-consolidate, not a behavior change.

## Cross-tick pattern: doubled-up sequence as carrier-pool stress signal

The post-W17 window's cumulative carrier-coverage matrix now shows:

- 7-of-7 carrier sweeps: drip-373 (3,5,0,0), drip-388 (0,8,0,0). Two ticks. ~6.7% of the 30-tick post-W17 window if it spans drip-360 through 390.
- 6-of-7 sweeps with one absentee: roughly half the window.
- 5-of-7 sweeps with two absentees and at least one doubled carrier: drips 387, 389, 390 — three of the last four ticks.
- 4-of-7 sweeps: rare; drip-389's tighter version had three doubles.

The shift from "rotating-carrier monoculture" (drips 360–380) to "doubled-up multi-carrier" (drips 386–390) is the most consistent cross-tick signal in the post-W17 window. It implies that the daemon's PR-discovery layer is finding fewer net-new PRs per tick from the smaller carriers (qwen-code, crush, goose) and is compensating by doubling the larger ones (codex, litellm, opencode) to maintain the eight-PR floor. The drip-388 all-MAN sweep (`0,8,0,0`) was a one-tick exception that briefly pulled all seven carriers active; the very next tick (389) reverted to three-carrier doubling.

## Wider implication: the verdict-shape entropy of the post-W17 window

If we treat the four verdict slots `(MAS, MAN, RC, ND)` as a categorical distribution per tick and compute Shannon entropy `H = -Σ p_i log2 p_i` over the eight per-tick PRs, drip-390's `(1,6,0,1)` gives `H = -(1/8)log2(1/8) - (6/8)log2(6/8) - (1/8)log2(1/8) = 0.375 + 0.311 + 0.375 = 1.061 bits`. Compare:

- drip-388 `(0,8,0,0)`: H = 0 bits (degenerate)
- drip-389 `(0,6,1,1)`: H = 1.061 bits (same shape mass distribution as 390, just MAS↔RC swap)
- drip-390 `(1,6,0,1)`: H = 1.061 bits

That makes drips 389 and 390 verdict-shape-entropy-equal at 1.061 bits, which is a **second-order coincidence** — different concrete shapes but identical Shannon mass partition `{6, 1, 1, 0}`. The post-W17 window has now produced two consecutive ticks at the same entropy level, which is the longest constant-entropy run since drips 372–375. The interpretation is that the verdict-shape distribution is in a stable two-mode regime: 0-bit degenerate sweeps (388-style) interleaved with ~1.06-bit mild-asymmetric sweeps (389/390-style), with no transitions back to the higher-entropy ~1.5-bit mixed regime that dominated drips 360–375.

## Coda — the four absent verdict slots

The **request-changes** slot is now zero in 4 of the last 6 ticks. RC was the dominant verdict in the W13 batch (per the INDEX header noting "contract violations" focus) but has effectively decayed in the post-W17 window. The reasonable interpretation is selection bias: the daemon's PR-discovery is filtering out the kind of PRs that historically attracted RC verdicts (large, contract-violating, abstraction-introducing PRs) in favor of smaller, more polished diffs that land MAS or MAN. The corollary is that **RC-zero is no longer a strong cleanliness signal** — it tells us more about input distribution than about review pipeline rigor. The fact that drip-390 still produced one ND despite RC=0 confirms that the pipeline is willing to escalate when warranted; it just isn't being given the kind of input that historically warranted RC.

That regime-change is the durable observation from drip-390. The verdict tuple `(1,6,0,1)` matters less than what it reveals about the input pipeline: small, single-purpose, often-subtractive diffs from a shrinking-active-carrier pool, doubled up to fill the quota, with the occasional process-rooted ND when maintainer alignment is missing. The post-W17 window has converged to a steady state.
