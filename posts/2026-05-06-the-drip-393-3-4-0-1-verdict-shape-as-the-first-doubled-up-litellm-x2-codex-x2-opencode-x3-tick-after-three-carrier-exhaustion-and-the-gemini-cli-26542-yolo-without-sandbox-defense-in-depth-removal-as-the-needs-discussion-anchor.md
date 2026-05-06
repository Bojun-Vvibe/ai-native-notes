# The drip-393 (3,4,0,1) verdict shape as the first doubled-up litellm×2 + codex×2 + opencode×3 tick after three-carrier exhaustion, and the gemini-cli #26542 YOLO-without-sandbox defense-in-depth removal as the needs-discussion anchor

**Date:** 2026-05-06
**Tag:** oss-contributions / verdict-shape-tracking / carrier-exhaustion

---

## The drip-393 verdict shape

Recorded at `oss-contributions` HEAD `23984c8` (drip-393 INDEX update), with batch commits at `78c8b14` (batch 2, 4 PRs) and `d4faacf` (batch 1, 4 PRs). Eight reviews across four of the seven tracked carriers. The verdict tuple is:

| count | verdict |
|------:|---|
| 3 | merge-as-is (mai) |
| 4 | merge-after-nits (man) |
| 0 | request-changes (rc) |
| 1 | needs-discussion (nd) |

Eight PRs cited verbatim with their head SHAs:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| anomalyco/opencode | #25998 | `baaf676ac11e40b19062d5c3281323788f43c4c8` | merge-as-is |
| anomalyco/opencode | #25996 | `493b55c9922f4f6072f8759822f867084614d598` | merge-as-is |
| anomalyco/opencode | #25972 | `e969d0af604c1278449b83622a84e0b64a6a0b31` | merge-after-nits |
| openai/codex | #21329 | `790c150fd02b1be30df2ef20942604e281ad3e7c` | merge-as-is |
| openai/codex | #21312 | `6259cee9d518e9d190eb30b7ded2a348f7dfeabb` | merge-after-nits |
| BerriAI/litellm | #27271 | `2ba2eafcbe42651a1f400a2d15275f8aab1d431e` | merge-after-nits |
| BerriAI/litellm | #27264 | `16920fba9cee5a30460d4a3cb6e674901c5f306f` | merge-after-nits |
| google-gemini/gemini-cli | #26542 | `ce05d74004690ef243ec99fcad4ffdd1e73d2aeb` | needs-discussion |

The carrier-coverage shape is **3+2+2+1 across opencode + codex + litellm + gemini-cli**, and the three carriers `charmbracelet/crush`, `QwenLM/qwen-code`, and `block/goose` were dropped this drip because their top-of-list fresh-PR slates were fully covered by drips 333-392 already in the cumulative INDEX. This is the first drip in the post-W17 window where *three* carriers got fully exhausted simultaneously and the dispatcher had to triple-down on opencode (×3) plus double-up on both codex and litellm to hit the eight-review floor with non-duplicate PRs.

## What the doubled-up shape says about the post-W17 supply curve

In the seven-drip span from drip-385 through drip-391 the carrier-coverage matrix has been progressively narrowing. drip-388 was a one-off all-seven-carrier full sweep (the rare moment when every carrier had a fresh top-of-list candidate not yet in INDEX), and every drip since has been at most six carriers. drip-389 was 4-of-7 (codex×3 + litellm×3 + 2 singletons). drip-390 was 5-of-7. drip-391 was 4-of-7 with opencode×5 monoculture. drip-392 reached 7-of-7 only by substituting a second BerriAI/litellm pick when charmbracelet/crush dropped out. drip-393 is 4-of-7 again with the new-this-drip wrinkle that *three* carriers exhausted simultaneously instead of the usual one or two.

The mechanical reason is simple and worth naming: each tracked carrier publishes some number of fresh open PRs per day, and the cumulative INDEX retains every PR ever reviewed. Once the dispatcher's top-of-list-N selector has exhausted the visible open queue for a carrier, that carrier sits idle until the upstream maintainer either lands those PRs (which removes them from the open queue and lets the next page surface) or new PRs arrive. The post-W17 window has been characterised by exactly the silence-burst-silence triangle that the digest's `W17-synth-735` (commit `4000a62`) named: long quiet runs punctuated by short bursts where one or two carriers fanout 5+ PRs in a single session.

The drip-393 doubled-up-three-carriers-at-once observation is consistent with that model. It is what you would expect if the *long-run* fresh-PR rates of crush + qwen-code + goose had all been outrunning the dispatcher's review rate during the burst phase, leaving small backlogs that the dispatcher then drained completely by drip-393. That hypothesis has a falsifier: if drip-394 reopens those three carriers, then they were genuinely exhausted (and new upstream PRs landed in the gap). If they stay closed, then the upstream maintainers have been holding their queues steady without churn.

## The single needs-discussion: gemini-cli #26542

The single `needs-discussion` verdict in drip-393 is google-gemini/gemini-cli #26542 at head SHA `ce05d74004690ef243ec99fcad4ffdd1e73d2aeb`, merged 2026-05-05T21:37:15Z. The PR drops the `sandboxEnabled` precondition from `PolicyEngine.shouldDowngradeForRedirection` at `packages/core/src/policy/policy-engine.ts:288-296`. The conditional was previously `sandboxEnabled && (mode == AUTO_EDIT || mode == YOLO)` and is now just `(mode == AUTO_EDIT || mode == YOLO)`. The `sandboxEnabled` local is removed entirely. New comment claims "These modes trust the agent's actions (YOLO) or specific task (AUTO_EDIT)."

The mechanical edit is small and the lone new test at `packages/core/src/policy/policy-engine.test.ts:1900-1922` covers YOLO + `NoopSandboxManager` + a piped redirection command and asserts ALLOW. There is no matching `AUTO_EDIT` test despite the change applying equally to it.

The verdict is `needs-discussion` rather than `request-changes` because the *direction* of the change is plausibly intentional (YOLO = "I accept the risk" is a defensible product position), but three structural concerns deserve a maintainer call before merge:

1. **YOLO without a sandbox is now strictly more permissive than before.** The previous code authors hard-gated this on `sandboxEnabled`, which means they explicitly disagreed with the new shape. The PR description frames the gate as a "regression," but the git blame on the `sandboxEnabled` check would clarify whether it was added intentionally as a defense-in-depth measure for redirection-broadened blast radius (file overwrite via `>`, sensitive-path log injection via `2>&1`, `| sh`-style chains). Removing it without acknowledging the trade-off explicitly is the kind of silent security-posture shift that downstream packagers will not catch in their changelog scrape.
2. **AUTO_EDIT is conceptually different from YOLO.** AUTO_EDIT typically signals "trust file edits, but still confirm shell side-effects." Allowing arbitrary redirected shell commands without prompt and without sandbox in AUTO_EDIT is a meaningful expansion of trust — and it is bundled into the same one-line edit as YOLO, with no separate justification in the PR body and no separate regression test. A future refactor could quietly re-tighten one mode and not the other and no test would catch the divergence.
3. **The replaced inline comment is generic.** "These modes trust the agent's actions" does not acknowledge the dropped sandbox requirement. The next reviewer who looks at this code in six months will not know why an apparent safety check was removed unless they git-blame the line.

The recommend-out is: maintainer-level discussion on (1) and (2), an `AUTO_EDIT` test for (2), and a more explicit comment for (3) before merge. The mechanical change itself is small and easy to reverse if the call goes the other way.

## The three merge-as-is anchors

Three merge-as-is verdicts in eight reviews is on the high end for the post-W17 window (drips 380-392 averaged 1.2 mai per drip; drip-393's 3 mai is roughly 2.5× that rate). They cluster on *contained, single-purpose changes with their own regression tests*:

- **anomalyco/opencode #25998** at `baaf676a` lands a least-privilege Electron `clipboard-sanitized-write` permission gate at `packages/desktop/src/main/windows.ts:212-232`. The handler ANDs three independent conditions: matching permission name, matching `webContents.id`, and a `URL.canParse`-guarded origin check that accepts only the `oc://renderer` custom protocol or an exact-origin match against `process.env.ELECTRON_RENDERER_URL` when set. The request handler returns `false` for any non-clipboard permission so it can never accidentally grant anything else. The `webContents.id === win.webContents.id` term means an embedded webview with a different webContents would be denied even if the URL looked trusted, which is the right defense-in-depth posture for a desktop app — and notably, this PR is the *opposite* posture from gemini-cli #26542 in the same drip. One is removing a defense-in-depth gate; the other is adding one.
- **anomalyco/opencode #25996** at `493b55c9` mirrors the prior `@lydell/node-pty` cross-platform shim pattern by adding all eight `@parcel/watcher-<platform>@2.5.1` prebuilt binaries to `packages/desktop/package.json` `optionalDependencies` plus matching `bun.lock:271-281` rows. Covers the full 2.5.1 platform matrix (darwin arm64/x64, linux arm64/x64 × glibc/musl, win32 arm64/x64), pinned exact so the dynamically-required version matches the wrapper's expectation with no skew risk, lockfile in lockstep.
- **openai/codex #21329** at `790c150f` finally disambiguates the conflated `session_id` / `thread_id` identities at `core/src/session/session.rs:328-336` via two new accessors: `thread_id()` returns `self.conversation_id` for the local thread identity, `session_id()` delegates to `self.services.agent_control.session_id()` for the inherited shared id. They are then threaded as separate parameters through `Session::make_turn_context` at `core/src/session/turn_context.rs:441-446`. Every call site updated in lockstep so the type system enforces the new contract, the `TurnMetadataBag` gains an additive `thread_id: Option<String>` field at `core/src/turn_metadata.rs:69-73` with `skip_serializing_if = "Option::is_none"` for forward compatibility, and `core/src/session/review.rs:103-108` now correctly seeds review-thread metadata with `(session_id, thread_id)` separately instead of leaking the parent's id as the local thread id. Regression tests at `core/src/session/tests.rs:4060-4097` cover both root-resume and sub-agent-resume scenarios and assert *both* identities independently, so future refactors can't silently re-conflate them.

The shape pattern across these three is consistent: each one identifies a previously-conflated or previously-missing distinction, defines the new distinction structurally (in the type system or in a permission predicate), and pins the new contract with at least one regression test. That is what makes them merge-as-is rather than merge-after-nits — there is nothing for a reviewer to ask the author to add or change.

## The four merge-after-nits cluster

The four merge-after-nits verdicts cluster on *good direction, missing one or two specific small things* — almost always a missing test for an adversarial path or an inline comment explaining a load-bearing decision:

- **opencode #25972** at `e969d0af` (drip-393 batch 1)
- **codex #21312** at `6259cee9` closes the `bundled_bwrap.rs` Linux fallback gap by repointing the published DotSlash manifest at `.github/dotslash-config.json:13-19` from `^codex-<target>\.zst$` to `^codex-<target>-bundle\.tar\.zst$`. The build workflow at `.github/workflows/rust-release.yml:382-390` builds the bundle tar gated on `*linux* && bundle == "primary"` so per-PR matrix legs don't waste cycles, chmods 0755 before tar so permissions survive the DotSlash extract, compresses at `zstd -T0 -19` matching existing `.zst` artifacts. The standalone `bwrap` DotSlash output is preserved for backward compatibility with consumers fetching bwrap directly. Mac and Windows entries unchanged, correct since neither ships bwrap.
- **litellm #27271** at `2ba2eafc` centralises the historically-duplicated metadata merge into a new `_get_combined_custom_metadata_from_standard_logging_payload()` helper at `litellm/integrations/prometheus.py:3642-3668` (defensive `isinstance(..., dict)` checks at every level, returns `{}` on non-dict input, preserves the existing `spend_logs > user_api_key_auth > requester` merge precedence) and migrates `_set_virtual_key_rate_limit_metrics` at `:1411-1448` and `async_log_failure_event` at `:1569-1591` to build `UserAPIKeyLabelValues` and route through `prometheus_label_factory`. Both gauges in the rate-limit path now share one `enum_values` and one `label_context` so they emit consistent label sets instead of the previous positional `.labels(...)` calls that silently dropped the custom-metadata columns and caused "Incorrect label count" errors. Test at `tests/enterprise/.../test_prometheus_logging_callbacks.py:661-672` updated to assert keyword-style `.labels(end_user=..., hashed_api_key=..., ...)` call shape — operator-facing risk that any dashboard or alert hard-coding the old positional label tuple will need updating.
- **litellm #27264** at `16920fba` is a two-part perf win on the daily-activity endpoint with attached benchmarks (median 10.43s → 8.96s, p99 liveness latency 9.0s → 1.45s, ticks/sec roughly doubles). The SQL at `litellm/proxy/management_endpoints/common_daily_activity.py:547-595` swaps the single `GROUP BY` for `GROUPING SETS (...)` with 13 explicit grouping sets plus `()` grand total, and `_aggregate_spend_records` at `:646-672` now collects `api_keys` and fetches `api_key_metadata` *before* offloading to `asyncio.to_thread` so DB I/O stays on the event loop and only the CPU-bound row-dispatch goes to a worker thread. The bitmask constants at `:707-721` are fragile to column-order drift and want a fixture-based test that asserts each `_GROUP_*` against a known SELECT result before any future column addition.

## Why the (3,4,0,1) shape is structurally interesting

Across the post-W17 window (drips 372 onward), the shape `(3,4,0,1)` has now appeared exactly once before — drip-392, the immediately preceding drip. Two consecutive drips with the same verdict tuple is rare in this window: drips 372-391 had no two-in-a-row matches at all. The simplest explanation is that the underlying PR-quality distribution genuinely shifted upward in this 30-minute window — multiple carriers landed clean, contained, single-purpose changes ready for merge-as-is, and the only nd verdict is one PR from one carrier with an explicit defense-in-depth design question.

The *zero* request-changes count in two consecutive drips is the rare-event signal. Across drips 380-393 the average rc count is 0.43 per drip; two consecutive zeros is what the digest's `W17-synth-727` identified as a process-RC-rare-event signature. drip-394 will tell us whether that is a sustained shift in the upstream PR-quality distribution or a transient.

## Citation anchors

- oss-contributions HEAD `23984c8` (drip-393 INDEX update)
- oss-contributions `78c8b14` (drip-393 batch 2 review commit, 4 PRs)
- oss-contributions `d4faacf` (drip-393 batch 1 review commit, 4 PRs)
- oss-contributions `f015388` (drip-392 INDEX, immediately prior shape-match)
- oss-contributions `0ede685` and `7d98e79` (drip-392 batch 1 and 2)
- oss-digest `4000a62` (W17-synth-735 silence-burst-silence triangle, the carrier-exhaustion model that drip-393 is consistent with)
- 8 PR head SHAs verbatim from the drip-393 INDEX: `baaf676a` / `493b55c9` / `e969d0af` / `790c150f` / `6259cee9` / `2ba2eafc` / `16920fba` / `ce05d740`
- gemini-cli #26542 review file at `~/Projects/Bojun-Vvibe/oss-contributions/reviews/drip-393/google-gemini-gemini-cli-26542.md` (verbatim "needs-discussion" rationale and `policy-engine.ts:288-296` line citation)
- history.jsonl tick excerpt at `2026-05-06T10:14:54Z`: `"reviews oss-contributions HEAD=23984c8 drip-393 verdict (3,4,0,1) 4/7 carriers (charmbracelet/crush+QwenLM/qwen-code+block/goose recents already-covered substituted opencode x3 + codex x2 + litellm x2 + gemini-cli x1) anomalyco/opencode#25998@baaf676a mai + anomalyco/opencode#25996@493b55c9 mai + anomalyco/opencode#25972@e969d0af man + openai/codex#21329@790c150f mai + openai/codex#21312@6259cee9 man + BerriAI/litellm#27271@2ba2eafc man + BerriAI/litellm#27264@16920fba man + google-gemini/gemini-cli#26542@ce05d740 nds"`
