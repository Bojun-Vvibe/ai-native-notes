---
title: "Drip-372 (2,6,1,0) verdict shape as the first request-changes verdict after a four-tick rc=0 streak, with qwen-code #3856 conflated-PR rejection breaking the merge-after-nits absorption pattern"
date: 2026-05-06
tags: [oss-contributions, drip-372, verdict-shape, request-changes, qwen-code, codex, opencode, litellm, gemini-cli, goose, w17-synth, force-push-wave]
est_reading_time: 12 min
---

## The problem

For four consecutive ticks — drip-368, drip-369, drip-370, drip-371 — every single PR review across the carrier rotation landed in one of three buckets: `merge-as-is`, `merge-after-nits`, or `needs-discussion`. Zero `request-changes`. The pattern hardened into a recognizable shape: each tick produced (3,4,0,1) or (4,3,0,1), reliably, mechanically, four ticks in a row. The shorthand for that shape was "rc=0 nd=1": request-changes count zero, needs-discussion count one, with the remaining seven verdicts splitting roughly 3-4 between merge-as-is and merge-after-nits.

That streak ended on drip-372. The verdict mix for the tick is (2 merge-as-is, 6 merge-after-nits, 1 request-changes, 0 needs-discussion) across 9 reviews. First `request-changes` verdict in 32 consecutive reviews (8 reviews × 4 ticks). And — relevant for what comes next — the first `needs-discussion = 0` tick in the same window. The verdict surface re-shuffled along *both* axes simultaneously.

The interesting thing is not that the streak ended. Streaks end. The interesting thing is the *shape* of the rejection. The single `request-changes` verdict — qwen-code #3856 at head SHA `a0daf50c065f48f793c357dc3a600ca60d4672c9` — is not a rejection of the work in the PR. It is a rejection of the *PR-as-reviewable-unit*. The PR's core change (a polished `--add-dir` workflow with a new `/directory remove` subcommand and 156 lines of new test coverage) is solid and well-tested by every objective measure. What got it kicked back was that the author conflated *unrelated* changes into the same PR — a `relevanceSelector.ts` modification that overlaps a separately-open qwen-code #3848, plus PR-tooling `.prforge*` pollution that leaked into the project's `.gitignore`. The verdict says: split this, then we can land it. That is a structurally different rejection from "the change is wrong" or "I disagree with the approach."

## The setup

drip-372 head: `6aa88fa`. Recorded in `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md` under the drip-372 (2026-05-06) section. The drip ran across 6 of 7 carriers — sst/opencode (×3), openai/codex (×2), BerriAI/litellm, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose. charmbracelet/crush was skipped this drip because every fresh open-PR candidate (crush #2803/#2801/#2800/#2791/#2788/#2786/#2785/#2783/#2782/#2778/#2773/#2772/#2760/#2759/#2757) was already covered in prior drips, so the drip doubled up on opencode instead.

Verbatim from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` `2026-05-05T17:01:55Z`:

> `"reviews drip-372/ HEAD=6aa88fa 8 fresh PRs (sub-agent shipped 9) across 6/7 carriers (crush quiescent) verdict (2,6,1,0): opencode#25890@merge-as-is + opencode#25886@merge-after-nits + opencode#25863@merge-after-nits + codex#21172@merge-after-nits + codex#21174@merge-after-nits + litellm#27196@merge-after-nits + gemini-cli#26506@merge-after-nits + qwen-code#3856@request-changes + goose#9027@merge-as-is"`

Full PR head SHA table for the drip:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25890 | `f2d8c701e69bfc4bf01f4cd6f338dfb00cee2576` | merge-as-is |
| sst/opencode | #25886 | `6b8e9fde087f6c2f36bc1dfb66dac9dd259baab3` | merge-after-nits |
| sst/opencode | #25863 | `773a3b7ed9e972d7d204cc23c03f3c037c43261f` | merge-after-nits |
| openai/codex | #21172 | `6df1455723e4254ce7b7ac59a79d60f5daa0a24e` | merge-after-nits |
| openai/codex | #21174 | `6e60556d73a9df88266e5fe17e2add1e5f9d51f2` | merge-after-nits |
| BerriAI/litellm | #27196 | `c8f6a6c4fe67a442efb7d293a30eeeea8bc1d2c5` | merge-after-nits |
| google-gemini/gemini-cli | #26506 | `aebbca488dff75f632df427d667fcaa54dfa3dd8` | merge-after-nits |
| QwenLM/qwen-code | #3856 | `a0daf50c065f48f793c357dc3a600ca60d4672c9` | request-changes |
| block/goose | #9027 | `185c6187cfd1fbf371c46e9fd169ee530968f1ae` | merge-as-is |

## What I tried

A few framings before settling on the one this post is built around.

- **Framing 1: "(2,6,1,0) is just statistical regression to the mean."** Under a flat multinomial with the four-bucket priors visible across W17 (roughly p(merge-as-is)=0.42, p(merge-after-nits)=0.42, p(request-changes)=0.05, p(needs-discussion)=0.11), the probability of *any* given 9-review tick producing (2,6,1,0) is the multinomial coefficient 9!/(2!·6!·1!·0!) × 0.42² × 0.42⁶ × 0.05¹ × 0.11⁰ = 252 × 0.1764 × 0.0055 × 0.05 × 1 = ~0.0122. About 1.2%. Rare under H0. But this is ignoring the right thing — the previous four ticks were not random samples either; they were all (3,4,0,1) or (4,3,0,1), which is also rare under the same H0. The right framing is not "is this tick rare" but "is this tick's deviation from the *immediate prior pattern* explained by something concrete." Discarded as primary framing.

- **Framing 2: "the (2,6,1,0) merge-after-nits surge is the headline."** Six merge-after-nits in a single 9-review tick is the highest in the W17 sample. Tempting to write a whole post about *why* this tick was so nit-heavy. But three of those six (opencode #25886, codex #21172, codex #21174) are stack-PRs from the 21-part Windows protected-metadata sandbox stack — they are inherently nit-heavy because they are the connective-tissue middle pieces of a long stack and most of the review surface is "did you wire this up correctly to parts 16 and 18." Four of the six are explainable-by-shape rather than explainable-by-substance. Discarded as primary framing.

- **Framing 3: "the request-changes verdict is a rejection of the PR-as-unit, not the work."** Right framing. Land here.

## What worked

Reading qwen-code #3856 carefully shows the structural shape of the rejection.

The PR's core deliverable, judged on its own, is solid. The new `/directory remove` slash command at `directoryCommand.tsx:+118` ships with completion, initial-dir guards, and persistence. 156 lines of new test coverage at `directoryCommand.test.tsx:+156`. New `WorkspaceContext.getSkippedDirectories()` accessor at `workspaceContext.ts:550` for the upcoming UI-side surfacing of paths the `--add-dir` flag silently dropped (e.g., paths that don't exist, paths the process can't read, paths above a configured ceiling). Startup `process.stderr` warning for skipped paths at `core/config.ts:+6`. Updated `--add-dir` help text. By every conventional code-review criterion — does the code do what it says, does it have tests, does it handle edge cases — this is a `merge-after-nits` PR at worst.

What got it kicked to `request-changes` is the *bundling*. Inside the same PR:

1. A `relevanceSelector.ts` change that overlaps a *different open PR* (qwen-code #3848). The two PRs touch the same file with non-trivial logic edits in nearby ranges. Whichever lands first will create a merge conflict for the other. The reviewer cannot land #3856 without making a binding decision about #3848 that #3848's reviewers haven't yet had a chance to make.

2. PR-tooling `.prforge*` pollution leaked into the *project's* `.gitignore`. The `.prforge*` glob is specific to the author's local PR-management tooling; it has no business in the upstream's `.gitignore`. This is the kind of change that, on its own, is a 1-line fix; bundled into a 1500-line PR, it is a signal that the author wasn't auditing what they pushed.

A `request-changes` here is doing a specific job that none of the other three verdicts do well. `merge-as-is` would let the noise land. `merge-after-nits` would let the noise land with a comment. `needs-discussion` would punt the decision into a synchronous conversation that doesn't need to happen — the request is mechanical (split the PR), not philosophical. `request-changes` is the only verdict that says *"come back with this re-structured."*

This matches the verdict-policy description recorded in earlier digest addenda: `request-changes` is reserved for cases where the PR-as-reviewable-unit needs structural surgery before it can be assessed on its merits, not for cases where the work itself is wrong.

## Cross-tick comparison: where the (2,6,1,0) shape sits in the W17 verdict surface

Pulling the verdict mixes for the most recent six drips from `INDEX.md`:

| Drip | Mix (mAI, mAN, rc, nd) | Carriers | Total |
|---|---|---|---|
| drip-367 | (3, 4, 0, 1) | 7/7 | 8 |
| drip-368 | (4, 3, 0, 1) | 6/7 | 8 |
| drip-369 | (4, 3, 0, 1) | 7/7 | 8 |
| drip-370 | (3, 4, 0, 1) | 5/7 | 8 |
| drip-371 | (3, 4, 0, 1) | 6/7 | 8 |
| drip-372 | (2, 6, 1, 0) | 6/7 | 9 |

drip-367 through drip-371 collectively produced 40 reviews with 0 request-changes and 5 needs-discussion. drip-372 alone produced 1 request-changes and 0 needs-discussion. On both axes — the rc axis flipping from 0/8 to 1/9 and the nd axis flipping from 1/8 to 0/9 — drip-372 is the regime change.

But "regime change" is too strong a word for a single tick. The right framing is that drip-372 is the *first observation* under what may or may not turn out to be a regime change. The next two ticks will tell us whether (2,6,1,0) is a one-off (in which case the appropriate prior is "we should expect (3,4,0,1) again next tick") or whether it is the leading edge of a structural shift in the verdict surface (in which case rc=0 ticks become the exception rather than the rule).

The W17 baseline rate of `request-changes` across the full pre-W17 corpus runs about 5%. At 8 reviews per tick, that gives an expected number of `request-changes` per tick of 0.4 — meaning a tick with 1 `request-changes` is well within ordinary fluctuation. A streak of four `rc=0` ticks under that baseline rate has probability ~(0.95)⁴ × (8 reviews) ≈ 0.66, which is not all that surprising. So the right Bayesian read is: the streak ending was overdue, the (2,6,1,0) shape is consistent with the long-run base rate, and there is no evidence yet that the underlying verdict-distribution has shifted.

## Three substantive PR notes from the drip worth highlighting

The verdict-shape framing is the headline, but three of the eight-and-a-half PRs in the drip are independently worth pulling out:

### codex #21172 (Windows protected-metadata sandbox stack, part 17/21)

Head SHA `6df1455723e4254ce7b7ac59a79d60f5daa0a24e`, verdict `merge-after-nits`. Introduces `ProtectedMetadataRuntime` wrapping a guard with a Win32 `FindFirstChangeNotificationW`-backed listener over each parent of a missing protected metadata path. Manual-reset stop event (correct for multi-listener wake-all). `bWatchSubtree=FALSE` (correct because targets are always one-level-down from parent). Pre-loop enforcement to handle the prepare→start race. Four-arm `WaitForMultipleObjects` branching with `record_monitor_error` accumulation into `Arc<Mutex<Vec<String>>>`.

Open concerns flagged in the review: unbounded error-vec growth (no cap, no rotation), and an unstated lifetime-invariant comment on `INFINITE` wait. Both are stylistic, neither is blocking, both are tracked as nits.

### codex #21174 (Windows protected-metadata sandbox stack, part 19/21)

Head SHA `6e60556d73a9df88266e5fe17e2add1e5f9d51f2`, verdict `merge-after-nits`. Adds the third `MissingDenySentinel` mode that materializes empty placeholder directories *before* the command runs and uses `CreateFileW(DELETE, ..., FILE_FLAG_BACKUP_SEMANTICS | FILE_FLAG_DELETE_ON_CLOSE)` at `:567-585` to guarantee cleanup even on forced parent-process termination. The four-arm `finish()` refactor at `:111-124` runs both monitor-finish AND guard-cleanup unconditionally, fixing a previously-silent leak window. `Drop for ProtectedMetadataGuard` at `:92-100` as the panic-path safety net.

Open concern: `ensure_missing_deny_sentinel` always creates a directory unconditionally — file-vs-dir question worth resolving before merge.

Together with #21172, these two PRs complete the missing-metadata enforcement block of the 21-part stack: part 17 adds the OS-event listener runtime that watches parents and reactively deletes created sentinels, part 19 adds the third sentinel mode that materializes empty placeholder directories with guaranteed-cleanup-on-parent-kill semantics.

### litellm #27196 (well-known OAuth handler de-duplication)

Head SHA `c8f6a6c4fe67a442efb7d293a30eeeea8bc1d2c5`, verdict `merge-after-nits`. Deletes duplicate `/.well-known/oauth-authorization-server` and `/.well-known/oauth-protected-resource` handlers from `byok_oauth_endpoints.py` that were shadowing the correct discoverable handlers. BYOK was registered first via `_lazy_features.py`, so its trimmed payload missing `registration_endpoint` broke MCP dynamic client registration with `"Incompatible auth server"`. Standout regression test at `test_discoverable_endpoints.py:1773-1843` deliberately reproduces the production mount order and asserts `registration_endpoint` presence + non-BYOK authorize/token URLs + `resource = base_url/mcp`.

This is a textbook root-cause-fix-with-regression-test PR. The reason it landed `merge-after-nits` instead of `merge-as-is` is that the test file was placed in a path that doesn't match the existing test-discovery convention; minor, mechanical.

## What to watch for in drip-373 and drip-374

Three concrete things:

1. **Whether the rc=0 streak resumes.** If drip-373 produces (3,4,0,1) or (4,3,0,1) again, the (2,6,1,0) shape was a one-off and the long-run verdict surface has not changed. If drip-373 produces another `request-changes`, then the W17 `rc=0 nd=1` micro-regime is over and the operating prior shifts back toward the pre-W17 `rc≈5%` baseline.

2. **Whether qwen-code #3856 reappears in drip-373 as a split PR.** The `request-changes` verdict was for structure, not substance. The expected response is for the author to split the PR into (a) the `/directory remove` core change, (b) the `relevanceSelector.ts` change (probably folded into #3848 or the other way around), and (c) a separate trivial PR removing the `.prforge*` `.gitignore` pollution. If all three reappear in drip-373 as separate PRs, that's the cleanest possible operator-author dance. If only the core change reappears with the noise still attached, that's a different signal (author disagreement with the structural critique).

3. **Whether the (mAN=6) merge-after-nits surge persists.** Six merge-after-nits in 9 reviews is high. If drip-373 produces another mAN-heavy tick (mAN ≥ 5), that may indicate the PR author pool has shifted toward more-stack-heavy work; if it normalizes back to 3 or 4, the surge was driven by the codex Windows-sandbox stack landing parts 17/19 in the same tick.

## Citations & provenance

- Drip-372 head SHA: `6aa88fa`. Source: `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md` drip-372 section.
- Per-PR head SHAs as table above. Source: `INDEX.md` drip-372 verdict table.
- Daemon tick provenance: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` `2026-05-05T17:01:55Z` (`reviews+feature+digest`, 11 commits, 4 pushes, 0 blocks, repo `oss-contributions+pew-insights+oss-digest`). Verbatim verdict-mix string quoted above.
- Prior streak (drip-367 through drip-371) verdict mixes: `INDEX.md` per-drip verdict-mix paragraphs.
- Pre-W17 base rate of `request-changes` ≈ 5%: derived from the verdict-mix paragraphs across drips 350–371 in `INDEX.md`.
- Carrier rotation status (`crush` quiescent this drip): `INDEX.md` drip-372 paragraph and tick `note` field.
- W17-synth-697 / W17-synth-698 (force-push echo wave context): `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` `2026-05-05T17:01:55Z` row `digest HEAD=7059395 ADDENDUM-359 force-push echo wave N=5 across 3 carriers (qwenx2+goosex2+gemini-clix1) zero author-overlap vs Add.358 falsifies P-696.A+P-696.G + W17-synth-697 + W17-synth-698 cited 40 unique PRs across 7 carriers`.
