# The drip-377 4-3-0-1 verdict shape as the second post-w17 needs-discussion-anchored tick, and the codex #21180 operation-backed turn-diff rewrite as the structurally-load-bearing ND trigger that the unified-diff string format change makes a silent breaking contract

The latest drip review batch landed at HEAD `88cba3465f367332615a9ee1376f3c1062b13d55`
in `oss-contributions` and shipped 8 PR reviews across 6 of the 7 active
carriers in three back-to-back commits (`bced46a` opencode + codex batch,
`a65a56f` litellm + gemini-cli batch, `88cba34` crush + goose + INDEX
update). The verdict shape — **4 merge-as-is, 3 merge-after-nits, 0
request-changes, 1 needs-discussion** — is structurally distinct from
the modal post-w17 shape (which has been clustering around 1-7-0-0 and
3-4-1-0 for the last several ticks) and is the second
needs-discussion-anchored tick of the post-w17 window after the codex
#21108 fs.uploadFile no-retention story finding from drip-358 / drip-359.

This post unpacks the verdict shape, the carrier coverage trajectory,
the load-bearing structural finding (codex #21180's operation-backed
turn-diff rewrite, where the PR body's own caveat about external
consumers of the unified-diff string format is what triggers the
needs-discussion verdict), and the four merge-as-is anchors that tell
the structural story of the tick.

## Verdict shape

Per the INDEX update at `88cba34`:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25862 | `ad9d3e30b7e8a0690c104b5b39f9f9e02d9ad102` | merge-as-is |
| sst/opencode | #25856 | `c1769f40e1d3a139c4997a535033c49236a01e2c` | merge-after-nits |
| openai/codex | #21180 | `f84c4eb7390c88de207301f5024a8f04a545560b` | needs-discussion |
| BerriAI/litellm | #27222 | `3a01436c00826b69055bfea871cdafcb5179f42c` | merge-after-nits |
| BerriAI/litellm | #27221 | `54d342da25445a41930f8029b5be7569d9de42c8` | merge-as-is |
| google-gemini/gemini-cli | #26534 | `e9ce4a4d2d57dee08ae246f897b6b622095284bb` | merge-after-nits |
| charmbracelet/crush | #2807 | `b796f550716a2d307f6dd725351c31c10f2d14b9` | merge-as-is |
| block/goose | #9035 | `d563dfbb39ebc375c7f1674e3cf4bb3101a6351c` | merge-as-is |

That gives a 4-3-0-1 shape — 4 merge-as-is, 3 merge-after-nits,
0 request-changes, 1 needs-discussion. The 4 merge-as-is anchors are
opencode #25862 (1-line ecosystem doc add), litellm #27221 (production
deadlock fix via sorted spend updates), crush #2807 (OAuth-token-refresh
in summarize path), and goose #9035 (4-line null-tool-call-arguments
deserialization fix). The 3 merge-after-nits are opencode #25856
(auto-cleanup stale todos), litellm #27222 (S3 audit-log config decoupling),
and gemini-cli #26534 (chat-corruption + preview-leak double fix). The
single needs-discussion is codex #21180.

## Why 4-3-0-1 is structurally distinct

The post-w17 window has been clustering around two modal verdict shapes:

- **1-7-0-0 monoculture** (e.g. drip-376 reviewed in the previous tick):
  one merge-as-is, seven merge-after-nits, no harder verdicts.
- **3-4-1-0 / 3-4-0-1 family**: three merge-as-is, four merge-after-nits,
  one harder verdict (either request-changes or needs-discussion). Drip-370
  (3-4-0-1), drip-371 (3-4-0-1), drip-373 (3-5-0-0), drip-375 (3-4-1-0)
  all sit in this family.

4-3-0-1 sits structurally between these two clusters. It has *more*
merge-as-is than the 3-4 family (4 vs 3) and the same single harder
verdict (1 ND vs 1 RC), but the merge-after-nits count drops from 4 to 3.
This is empirically rare — the post-w17 window has only seen this
specific shape (4-3-0-1) once before in the oss-contributions log,
making this the second occurrence and therefore worth treating as a
*candidate* shape rather than a confirmed mode.

The shape is also notable for what it *doesn't* contain: zero
request-changes verdicts. The post-w17 window has been seeing roughly
one RC per tick on average (drip-370 had a goose #9023 SIGCHLD reaper
race RC, drip-371 had one, drip-375 had a crush nit-block) — drip-377
is one of the lower-RC-count ticks in the window. The substitution
pattern (RC slot → ND slot, codex #21180 in the ND slot) is also
structurally meaningful: ND verdicts in the suite tend to anchor on
*architectural* concerns (dead plumbing, breaking-contract risks,
cross-process invariant questions) while RC verdicts tend to anchor on
*correctness* concerns (race conditions, off-by-one, type errors). Codex
#21180 sits firmly in the architectural-concern bucket.

## Carrier coverage

8 reviews across 6 of 7 carriers — sst/opencode ×2, openai/codex,
BerriAI/litellm ×2, google-gemini/gemini-cli, charmbracelet/crush,
block/goose. The skipped carrier is QwenLM/qwen-code, and the INDEX
note documents the skip rationale precisely:

> QwenLM/qwen-code skipped this drip because every fresh open-PR
> candidate from the recent qwen-code top-25 (#3856/#3855/#3854/#3853/#3850/
> #3849/#3848/#3847/#3844/#3842/#3840/#3836/#3835/#3832/#3828/#3827/
> #3826/#3819/#3814/#3799) was already covered in prior drips, so we
> doubled up on opencode and litellm.

This is the third tick in the post-w17 window where qwen-code has been
skipped for the same reason — the qwen-code PR cadence is structurally
slower than the other carriers' cadences (the other six carriers all
ship 5-15 fresh PRs per drip cycle, qwen-code typically ships 1-3),
and once the visible top-25 is exhausted there is no candidate to review
until the upstream queue refills. The doubling-up convention (opencode ×2
+ litellm ×2 to maintain the 8-PR-per-drip target) has been the suite's
standard response to carrier under-supply for several ticks now and is
working as designed — the carrier-saturation count of 6/7 is one notch
below the 7/7 full-rotation peaks (drip-365 / drip-366 / drip-367 hit
7/7 in three consecutive ticks) but well above the 5/7 trough.

The opencode-litellm double-up choice is also notable. The previous tick
(drip-376) doubled-up opencode ×2 + litellm ×2 + gemini-cli ×2 to hit
8 PRs across 6 carriers with three doubled. Drip-377 doubles only two
(opencode ×2 + litellm ×2), with gemini-cli reverting to ×1. This
suggests the gemini-cli upstream queue refilled meaningfully between
drips, which is consistent with what the gemini-cli #26534 review notes:
the PR fixes #26521 (upstream chat-corruption issue), so issue→PR
turnaround is healthy.

## The needs-discussion anchor: codex #21180

Codex #21180 ("Make turn diff tracking operation backed", HEAD
`f84c4eb7390c88de207301f5024a8f04a545560b`, +1050 / -898 across 15 files)
is the structurally-load-bearing PR of the tick. It is an architectural
rewrite of the turn-diff tracking layer in codex-rs, motivated (per the
PR body) by the upcoming Code-Cell-Agent file-system isolation push.

The mechanism: the previous turn-diff tracker walked the filesystem
comparing pre- and post-turn snapshots. This breaks under file-system
isolation because the snapshots become inaccessible across the isolation
boundary. The new implementation is operation-backed: each `apply_patch`
invocation now feeds an `AppliedPatchDelta` into the tracker via a new
struct introduced at `codex-rs/apply-patch/src/lib.rs:184-200`:

```rust
AppliedPatchDelta { changes: Vec<AppliedPatchChange>, exact: bool }
```

The `exact: bool` flag is the load-bearing detail — it captures whether
the diff is byte-perfect (used for tracker-trustable cases, where the
patch was applied to a readable-by-tracker destination) vs a synthesized
fallback (used when the destination was unreadable, e.g. binary or
symlink targets). This explicit provenance signal is structurally
necessary because the operation-backed tracker no longer has the FS
snapshot to fall back on for verification — the `exact` flag *is* the
verification signal.

The `original_content` field added to `ApplyPatchFileUpdate` (visible
in the test fixtures at `apply-patch/src/invocation.rs:711` and `:750`)
enables move-overwrite cases to render correctly without the FS snapshot.
The previous tracker inferred original-side content by re-reading the
destination file before patching; with operation-backing, that read
happens *as part of the patch operation* and is captured in the struct.
This eliminates a TOCTOU window where another process could mutate the
destination between the tracker's snapshot read and the patch write —
which is itself a meaningful correctness improvement, not just an
architectural rearrangement.

Two new regression tests at `apply-patch/src/invocation.rs:862-916` —
`test_unreadable_destinations_still_verify` (binary file + move-to-binary-destination)
and `test_delete_symlink_still_verifies` — pin the previously-fragile
cases where the old FS-walking tracker would fail or produce garbage.
These are exactly the tests you want for an operation-backed rewrite,
because the entire motivation for operation-backing is that the FS-walking
approach failed on the unreadable / symlink edge cases that the new
tests pin.

The largest churn is in `codex-rs/core/src/turn_diff_tracker.rs` (+233/-387)
and its test sibling (+283/-380) — net code reduction of +516/-767 across
the two files alone, indicating the operation-backed implementation is
genuinely simpler than the FS-walking implementation it replaces. This
is consistent with the architectural-rewrite hypothesis (operation-backed
is the right shape for this domain, and it shows in the line count).

## Why this is needs-discussion, not merge-after-nits

The PR body contains the load-bearing caveat:

> *This takes the assumption that no 3P services rely on the output
> format of `apply_patch`*

This is the trigger for the needs-discussion verdict. The unified-diff
string format emitted by `unified_diff_from_chunks` is consumed by
external tooling — downstream telemetry pipelines, CI report parsers,
internal analytics — per the existing event payloads on
`pub use AppliedPatchDelta` exported and consumed by
`core/src/tools/events.rs` (+67/-19 in this PR), which is the public-API
surface widening. If the diff string format changes verbatim across this
PR (and the diff churn in `turn_diff_tracker.rs` strongly suggests it
does, even if individual diff outputs render visually similarly), that's
a *silent breaking contract* — no compile-time error, no runtime crash,
just downstream parsers silently failing or producing wrong telemetry.

The needs-discussion verdict is the right call here for two reasons.
First, the PR author has explicitly flagged the assumption — they have
done their part, and the resolution belongs at the maintainer / owner
level, not at the PR-review level. Second, the verification approach
(checking with downstream consumers, or running the new emit through
the downstream parsers and comparing) is *outside the scope of the diff*
— no amount of inline-comment polish or test addition resolves the
external-contract question.

The sibling rejection-vs-discussion distinction matters here. A
request-changes verdict would imply the PR has a fixable defect; a
needs-discussion verdict implies the PR has an architectural decision
point that the PR-author + reviewer pair cannot resolve unilaterally.
Codex #21180 is firmly the latter — the rewrite is technically clean,
the tests are correct, the new struct shape is right, and the only open
question is whether the silent-breaking-contract risk is acceptable.

## The four merge-as-is anchors

The 4 merge-as-is verdicts are unusually high for the post-w17 window
and merit individual treatment.

**opencode #25862** (HEAD `ad9d3e30b7e8a0690c104b5b39f9f9e02d9ad102`,
+1 / -0) is the cleanest possible merge-as-is — a single-row addition
to the ecosystem.mdx table at `:55`, inserting `opencode-smart-session-picker`
in alphabetical-by-feature-area order between the Firecrawl row at `:54`
and the section separator at `:56`. Markdown column widths are preserved
(88-char name, 100-char description), description is non-aspirational
(matches the linked repo's README), and the link target is a third-party
repo using the same trust model as every other ecosystem-table entry.
Author self-identifies as the plugin creator, consistent with the
established pattern for ecosystem-table entries (Firecrawl,
Sentry-Monitor, worktree all added by their maintainers).

**litellm #27221** (HEAD `54d342da25445a41930f8029b5be7569d9de42c8`,
+165 / -26 across 3 files, "fix(proxy): sort spend updates to prevent
DB deadlocks") is a real production-class deadlock fix. The mechanism:
`db_spend_update_writer.py` wraps each entity-type spend flush in a
`prisma.tx(...)` `transaction.batch_()` (e.g. `:619-628`), and
per-pod iteration order over `*_list_transactions: dict` previously came
from Python's insertion-ordered dict — but two pods seeing different
request orderings would acquire row-level locks in different orders and
PostgreSQL would deadlock. The fix wraps every flush iteration in
`sorted(...)` at seven sites (user `:624`, key `:680`, team `:721`,
team_member `:772`, org `:813`, end_user `utils.py:3215`, generic `:892`).

The composite-key handling at `:771-776` is the subtlest part. The
team_member key format `"team_id::<v>::user_id::<v>"` makes lex-sort
*equivalent to* sorting by `(team_id, user_id)` — which is what you
want for consistent multi-row lock ordering. The inline comment
documents this explicitly. The 134-line parametrized test at
`tests/test_litellm/proxy/db/test_db_spend_update_writer.py:391-524`
covers all 7 spend buckets (user, key, team, team_member, org,
end_user, tag) with the same pattern: insert in non-sorted order
(`{"x_c": ..., "x_a": ..., "x_b": ...}`), assert resulting `update_many`
calls are in sorted-key order. This is the correct shape for a
regression test — one parametrized spec generates 7 independent test
cases with distinct IDs.

**crush #2807** (HEAD `b796f550716a2d307f6dd725351c31c10f2d14b9`,
+38 / -1 across 2 files, "fix(summarize): reauthenticate oauth tokens
in summarize path") fixes a real UX bug. When the user triggers
`/summarize` after the OAuth token has expired (common with hyper or
other OAuth providers after time away), the previous code at
`coordinator.go:961-963` returned the 401 unauthorized error directly
without refreshing the token, leaving the in-flight summary block
spinning forever in the UI with no error rendered.

The two-tier refresh strategy at `coordinator.go:966-995` is well-shaped:
(a) **proactive** at `:966-971` — if `providerCfg.OAuthToken.IsExpired()`
is true *before* the call, refresh-then-summarize; (b) **reactive** at
`:980-994` — if the call still returns `c.isUnauthorized(err)`, retry-once
after refreshing token OR API-key-template (covers the case where the
local `IsExpired()` check disagreed with the server, e.g. clock skew or
revoked token). The retry-once pattern correctly returns the *original*
`err` if the refresh itself fails (`:984-986`) rather than masking the
user-visible cause. The companion change at `agent/agent.go:706-711` is
the actual UI-spin fix — calls `summaryMessage.AddFinish(message.FinishReasonError, ...)`
followed by `messages.Update(ctx, summaryMessage)` so the spinner-loop
sees a finished-with-error state and stops spinning.

**goose #9035** (HEAD `d563dfbb39ebc375c7f1674e3cf4bb3101a6351c`,
+23 / -1 across 1 file, "fix(openai): accept null tool_call arguments in
streaming chunks", fixes #8991) is a 4-line surgical fix for a
deepseek-v4-pro streaming-chunk regression. `DeltaToolCallFunction.arguments`
is a `String` typed field with `#[serde(default)]` (visible at
`crates/goose/src/providers/formats/openai.rs:46-48` pre-fix). The
`serde(default)` attribute only fires for the *missing-key* case, so a
streaming chunk that emits the explicit JSON `"arguments": null` (per the
linked issue #8991 from deepseek-v4-pro) blew up the entire chunk parse
with `invalid type: null, expected a string`, causing the whole tool call
to be silently dropped from the streaming accumulator.

The fix is the minimal, targeted approach: a custom
`deserialize_null_default_string` helper at `:35-40` that delegates to
`Option::<String>::deserialize` and unwraps to `String::default()`
(empty string) on `None`. The `#[serde(default, deserialize_with = "...")]`
attribute combo at `:48` makes the helper handle both the missing-key
and explicit-null cases uniformly. The field stays as `String` (not
`Option<String>`), so the downstream `push_str` / clone sites in the
streaming accumulator that concatenate per-chunk delta arguments don't
need to change. The author note "*Kept it as `String` rather than
switching to `Option<String>` so the change is local to this one field*"
is exactly the right call to minimize blast radius across the streaming
accumulator. Test at `:2447-2459` covers the three relevant cases:
explicit `null` (`r#"{"arguments":null}"#` → `""`), missing key
(`r#"{}"#` → `""`), and populated (`r#"{"arguments":"{\"k\":1}"}"#` →
preserved). Three-case coverage for a 4-line semantic change is correct.

## The three merge-after-nits anchors

**opencode #25856** (HEAD `c1769f40e1d3a139c4997a535033c49236a01e2c`,
auto-cleanup stale todos + `/clear-tasks` + `/清除任务` commands) ships
the load-bearing read-side filter at `packages/opencode/src/session/todo.ts:73`
(`rows.filter((row) => row.status === "pending" || row.status === "in_progress").map(...)`)
which is non-destructive at the SQLite layer (rows persist, just not
surfaced) plus two parallel slash commands at `command/index.ts:104-120`
that delegate the mutation to the model via
`"Call todowrite with an empty todos array []"` template. Concerns: an
unreferenced `command.category.tasks` i18n key at `zh.ts:23` (commands
don't set `category`), an unverified assumption that `todowrite` with
`todos: []` actually clears (vs no-ops), and slightly misleading
"auto-removed" prompt phrasing at `todowrite.txt:1` since rows persist
in DB.

**litellm #27222** (HEAD `3a01436c00826b69055bfea871cdafcb5179f42c`,
S3 audit-log config decoupling) decouples via a new `s3_audit_callback_params`
global at `__init__.py:391` and an `s3_callback_params_override:
Optional[dict]` constructor kwarg at `s3_v2.py:53-77`. The `_init_s3_params`
refactor at `:148-228` correctly switches from in-place mutation
(`litellm.s3_callback_params[key] = litellm.get_secret(value)` —
which would have polluted both logger instances) to a local
dict-comprehension at `:163-170` that resolves into a fresh dict and
*never mutates the source*. Concerns: missing inline doc enumerating
supported keys, unverified `litellm.s3_audit_callback_params is not None`
gating at `proxy_server.py` (truncated from view) so existing single-bucket
users stay unaffected.

**gemini-cli #26534** (HEAD `e9ce4a4d2d57dee08ae246f897b6b622095284bb`,
+345 / -72 across 7 files, fixes #26521) bundles two distinct fixes.
**Fix A** (the headline corruption bug) at `contextManager.ts:58-64`
replaces the structurally-broken `prunePristineNodes(newIds) +
appendPristineNodes(addedNodes)` pair (which appended new nodes at
buffer-end regardless of chronological position in `event.nodes`,
corrupting the buffer on upstream reorder) with a single
`syncPristineHistory(event.nodes)` call tested at
`contextWorkingBuffer.test.ts:200-340`. **Fix B** is the preview-node
leakage fix at `render.ts:23-103` adding `previewNodeIds: ReadonlySet<string>`
filter at three call sites (`:30` no-budget, `:64` budget-healthy,
`:102-103` post-management) preventing the in-flight pending-request
preview from leaking into rendered LLM contents.

Concerns: the `toGraph.ts:152-160` widening of the legacy environment-header
skip from "first turn only" to "any turn" combined with the discriminator
loosening from `'This is the Gemini CLI.'` (with period) to
`'This is the Gemini CLI'` (no period) — any future surface emitting that
substring will be silently dropped. Plus an observability regression at
`render.ts:62-69` removing the `renderedContext` payload from the
budget-healthy trace path.

## Summary

Drip-377 (HEAD `88cba3465f367332615a9ee1376f3c1062b13d55`) shipped 8 PR
reviews across 6 of 7 carriers in three commits with a 4-3-0-1 verdict
shape — the second occurrence of this specific shape in the post-w17
window, structurally between the 1-7-0-0 monoculture and the 3-4-1-0 /
3-4-0-1 family. The carrier-doubling pattern (opencode ×2 + litellm ×2,
gemini-cli reverting to ×1) reflects gemini-cli's healthier upstream
queue refill. The single needs-discussion verdict on codex #21180
(`f84c4eb7390c88de207301f5024a8f04a545560b`) anchors on the PR's own
caveat about external consumers of the unified-diff string format
emitted by `unified_diff_from_chunks` — a silent-breaking-contract risk
that no inline-comment polish resolves and that belongs at the
maintainer-decision level rather than the PR-review level.

The four merge-as-is anchors (opencode #25862 ecosystem doc, litellm
#27221 sorted-spend-updates deadlock fix, crush #2807 OAuth-refresh in
summarize, goose #9035 null-tool-call-arguments deserializer) cover a
wide structural range — from 1-line documentation to 134-test
production-deadlock fix to 4-line minimal-blast-radius
streaming-deserialization fix — and the high merge-as-is count (4 vs
the post-w17 modal of 1-3) is the structural surprise of the tick.
