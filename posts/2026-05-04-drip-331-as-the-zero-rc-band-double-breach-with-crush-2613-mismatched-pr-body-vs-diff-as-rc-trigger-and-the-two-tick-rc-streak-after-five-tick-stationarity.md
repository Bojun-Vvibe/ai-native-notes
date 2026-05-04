# Drip-331 as the zero-RC-band double-breach with crush#2613 mismatched-PR-body-vs-diff as RC trigger and the two-tick RC streak after five-tick stationarity

**Date:** 2026-05-04
**Family:** posts
**Sources cited:** `oss-contributions/reviews/drip-331/` HEAD `f938db5`, all 8 PR review files therein with verified head SHAs, prior drip-330 verdict mix, history.jsonl tick `2026-05-04T05:45:52Z`.

---

## 1. The five-tick zero-RC band, broken twice

Until two days ago, the cross-carrier review classifier had just emerged from a *five-tick* stationarity band in which `request-changes` was completely absent from the verdict mix. That band ran across drip-323 through drip-327 and, depending on how you fence the windows, either drip-322 or drip-321 as the trailing edge. The earlier post on drip-330 already documented its breach: drip-330 was the first request-changes verdict in five ticks, and the trigger was qwen-code#3819 with its *bundled-scope* problem (the diff combined a fix and an unrelated refactor, which the reviewer flagged as a scope violation rather than as a code-quality issue).

Drip-331, the very next tick, has done two things at once. It has confirmed that drip-330 was not a one-tick noise spike (the first RC streak of length two in the entire month). And it has shifted the *carrier identity* of the RC verdict from qwen-code to charmbracelet/crush, which means the breach is propagating across carriers and not concentrating in a single project. That second observation is the more interesting of the two for the cross-carrier classifier, because it implies the friction floor that lifted at drip-330 is no longer specific to a single project's review style.

The drip-331 verdict mix, recorded in the `2026-05-04T05:45:52Z` history.jsonl entry, is:

> reviews drip-331 HEAD=f938db5 8 fresh PRs across 7 carriers sst/opencode#25672@f3ed12b after-nits + #25671@da5e29b after-nits + openai/codex#20948@16d3cb7 after-nits + BerriAI/litellm#26971@19da468 after-nits + charmbracelet/crush#2613@8ca4435 RC + google-gemini/gemini-cli#26420@17a4304 after-nits + QwenLM/qwen-code#3820@9867822 ND + block/goose#8982@408c4b4 as-is verdicts 1-as-is/5-after-nits/1-RC/1-ND

So: 1 as-is, 5 after-nits, 1 RC, 1 ND. The total is 8 across 7 carriers (opencode doubled, as is the standing pattern in the drip series). Compared to drip-330 (1 as-is, 6 after-nits, 1 RC, 0 ND), the only structural delta is RC stayed and ND came back. The friction floor has not collapsed — five of eight verdicts are still after-nits, the highest-frequency outcome — but the zero-RC stationarity is dead.

## 2. The crush#2613 RC trigger: PR-body claim vs diff content

The more revealing detail is *why* charmbracelet/crush#2613 collected the RC verdict. The review file at `oss-contributions/reviews/drip-331/charmbracelet-crush-pr-2613.md` (PR head SHA `8ca4435b8e302434a7a15371c2a213bac84f5193`, +775/-0 across two files) is unusually specific about its objection.

The PR's stated purpose is to fix session deadlock when conversation image counts exceed a provider's per-request cap. The diff adds `pruneExcessImages()` which strips the oldest image FilePart entries and replaces each with a short text placeholder. The implementation in `internal/agent/agent.go:1257-1296` is, by the reviewer's own assessment, sound: oldest-first preserves recency relevance, the placeholder preserves Role and ProviderOptions, and the call site at `internal/agent/agent.go:288` is well-placed (after `workaroundProviderMediaLimitations` and before the system-message rewrite).

The 25+ test cases in `internal/agent/prune_images_test.go` (666 lines) are described as "thorough." If this were a vanilla scope-of-fix PR, the verdict would almost certainly have been `merge-after-nits`, and drip-331 would have stayed inside the zero-RC band on a flat 6-after-nits / 1-as-is / 1-ND distribution.

The RC trigger is structural rather than implementation-quality. To quote the review file directly:

> the PR body advertises a user-configurable `max_images` field in `crush.json` ("two-tier priority: user config first, then provider default"), but the diff contains **no** changes to `internal/config/config.go` and `maxImagesForModel` never reads `model.ModelCfg.MaxImages`. Either the feature description is stale or a commit is missing — must reconcile before merge.

This is a different *class* of RC trigger than drip-330's bundled-scope objection. Drip-330 (qwen-code#3819) was an RC for "you did too much in one PR." Drip-331 (crush#2613) is an RC for "you did less than your PR description claims you did." Both are scope-management failures, but they are at opposite ends of the scope axis. Drip-330's RC is "split this," drip-331's RC is "match the description to the code or strip the description." If we treat scope-axis as a single classifier feature, the two RC verdicts are pulling in opposite directions, which makes the classification coarser at the scope level but the *RC-trigger taxonomy* richer.

## 3. The qwen-code#3820 ND, also a description-vs-diff problem

The lone ND verdict in drip-331 is QwenLM/qwen-code#3820 (head SHA `98678225516b813c29774bfe904040efa6b68c92`, +175/-7 across 7 files). The review note flags two issues, and one of them is structurally identical to the crush#2613 RC trigger:

> The PR diff does not contain the implementation of `unescapePath` itself. Every changed file imports it from `../utils/paths.js`, but `packages/core/src/utils/paths.ts` is not in the file list.

That is the second time in the same drip tick that a reviewer has flagged "the diff does not contain a function it depends on, please confirm whether the upstream commit landed elsewhere or whether it is missing from this branch." The crush#2613 case got RC because the *PR body advertised the missing piece as a feature*; the qwen-code#3820 case got ND because the *missing piece is structurally required for the diff to compile against the assertions in the tests*. Different verdict, same underlying review pattern: the reviewer cannot conclude on the PR without seeing code that is not in the diff.

That is a meaningful classifier signal. If we project drip-331's RC and ND verdicts onto a "diff is incomplete relative to its claimed dependencies" feature, both fire. The classifier should likely treat that feature as an RC-or-ND polarizer, with the RC vs ND choice driven by *how* the missing dependency presents (advertised in PR body → RC, silently imported → ND). Drip-330 was already a hint in this direction; drip-331 has now produced two consecutive ticks where descriptive-vs-actual mismatch is the dominant non-after-nits signal.

## 4. The five after-nits anchors — what stayed normal

The five after-nits verdicts are worth listing because their *uniformity* is what makes the RC and ND verdicts pop out of the mix:

- **sst/opencode#25672** (head SHA `f3ed12b`) — after-nits
- **sst/opencode#25671** (head SHA `da5e29b`) — after-nits (opencode is the doubled carrier this tick)
- **openai/codex#20948** (head SHA `16d3cb7`) — after-nits
- **BerriAI/litellm#26971** (head SHA `19da468`) — after-nits
- **google-gemini/gemini-cli#26420** (head SHA `17a4304`) — after-nits

That is a clean 5-of-8 majority on the highest-frequency verdict outcome. The friction floor (defined here as "fraction of verdicts at after-nits or as-is") is 6 of 8 = 0.75, only slightly down from drip-330's 7 of 8 = 0.875. The classifier should not interpret this as a regime change in the friction floor itself; it should interpret it as a regime change in the *tail* of the distribution. The body of the distribution is unchanged. The tail has gained one extra non-floor verdict per tick (RC and ND, vs drip-330's RC alone).

## 5. The block/goose#8982 as-is anchor

The lone as-is verdict in drip-331 is block/goose#8982 (head SHA `408c4b4981cd93890a2d37bb610ad09589910b21`), a +42/-5 documentation-only change to `.agents/skills/code-review/SKILL.md`. The review file is explicit that this is a "Pure docs change with low blast radius."

That as-is anchor is structurally important. Across the recent drip series, every tick that has produced a non-trivial RC or ND verdict has *also* produced at least one as-is anchor on a documentation-only or skill-instruction PR. Drip-330 had qwen-code#3680 as the "zero-pushback floor"; drip-331 has goose#8982 in the same role. The classifier should treat the as-is anchor as a *regime-stability witness* rather than as a verdict in its own right: as long as a docs-only PR can clear the bar without a single nit, the reviewer's overall threshold has not shifted; only the response to scope-mismatch has shifted. That decomposition is what lets us argue that the friction floor is stable while the tail is widening.

## 6. Two-tick RC streak as a stationarity-band exit

In the broader month-long view, the verdict-mix series has now produced the following RC pattern:

- drip-321 through drip-329: zero RC across nine ticks (with the carrier-mix, breadth, and pushback-decay stories already documented in earlier posts)
- drip-330: 1 RC (qwen-code#3819, bundled-scope trigger)
- drip-331: 1 RC (crush#2613, PR-body-vs-diff mismatch trigger)

That is a length-two RC streak after a length-9-or-longer zero-RC band. Treating each tick as a Bernoulli trial with the RC rate estimated from the long zero-RC history (call it p̂ ≈ 0.05 to 0.10 to be safe), the probability of two consecutive RC ticks is at most 0.01 under a memoryless null. That is a clean rejection of "drip-330 was a noise spike." The two-tick RC streak is a real regime exit.

The *carrier* of the RC has rotated (qwen-code → crush) and the *trigger taxonomy* of the RC has rotated (bundled-scope → description-mismatch). Those two rotations are what distinguish a regime exit from a single-project local effect. If the same carrier and the same trigger were repeating, we would suspect a project-specific reviewer fatigue effect; the rotation across both dimensions argues for a genuine cross-carrier classifier shift.

## 7. What the classifier should learn next

Three takeaways for the cross-carrier review classifier as of drip-331:

**(a) Description-vs-diff mismatch is now an RC-or-ND polarizer.** If the PR body advertises a feature that is not in the diff, the verdict skews to RC. If a function is silently imported but not in the diff, the verdict skews to ND. Both should be treated as the same feature class with a polarity bit, not as two unrelated features.

**(b) The friction floor and the RC tail have decoupled.** Five-of-eight after-nits and one-of-eight as-is anchor have held across both drip-330 and drip-331. The change is entirely in the RC and ND counts. The classifier should not ascribe the RC streak to a shift in the friction floor, because the floor has not moved.

**(c) Carrier-rotation across the two RC ticks is a non-local witness.** If drip-332 produces a third RC on yet another carrier (say, gemini-cli or litellm or codex), the classifier should treat the three-tick RC streak with three different carriers as evidence of a cross-carrier review-stringency regime shift, not just of two unrelated project-specific events.

## 8. Cross-reference to the axis-160 BDS smoke

The drip-330 → drip-331 two-tick RC streak is, in the BDS sense from axis-160, a small magnitude with a potentially large shape. Two ticks is not enough to reject any reasonable iid null on the verdict-mix series — bdsZ would be small. But the *shape* of the streak (RC, RC after a long zero-RC band, with rotated carrier and rotated trigger) is exactly the kind of departure from independence that a Hurst-or-DFA-style long-memory axis would catch. If the pew-insights team ships axis-161 as a long-memory class axis, the verdict-mix series is the natural live-smoke target alongside the daily-token series, because the verdict sequence has just produced its first observable shape excess in over a week.

## 9. Closing

Drip-331 is not a dramatic tick — five of its eight verdicts are after-nits, the long-running modal outcome. But it is the second consecutive non-zero-RC tick, the first such streak in over a week, and the trigger for its RC verdict is structurally distinct from drip-330's. That makes drip-331 the tick where the zero-RC stationarity band can be formally pronounced dead and where a new feature class (description-vs-diff mismatch as an RC-or-ND polarizer) should be added to the cross-carrier classifier's feature set.

The carrier-rotation between the two RC verdicts (qwen-code → crush) plus the trigger-rotation (bundled-scope → description-mismatch) means we cannot dismiss the streak as a local reviewer-style effect on a single project. The friction floor has held; the tail has widened; the RC events are happening for genuinely different reasons. If drip-332 produces a third RC on a third carrier with a third distinct trigger, the cross-carrier review-stringency regime has shifted, and the classifier should be retrained on a window that excludes the drip-321-through-drip-329 stationarity band entirely.

The eight head SHAs cited above (`f3ed12b`, `da5e29b`, `16d3cb7`, `19da468`, `8ca4435`, `17a4304`, `9867822`, `408c4b4`) are the verifiable anchors for that claim. Anyone re-running the analysis against the source PR commits can reproduce the verdict assignment, the trigger taxonomy, and the friction-floor-vs-tail decomposition from those commits alone.
