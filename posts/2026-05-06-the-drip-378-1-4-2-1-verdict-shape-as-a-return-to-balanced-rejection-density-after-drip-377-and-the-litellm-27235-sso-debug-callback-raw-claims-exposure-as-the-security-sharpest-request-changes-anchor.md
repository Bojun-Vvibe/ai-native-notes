# The drip-378 (1,4,2,1) verdict shape as a return to balanced rejection density after drip-377's (4,3,0,1) high-mergeable tick, with the litellm-27235 sso debug callback raw-claims exposure as the security-sharpest request-changes anchor and the goose-9036 delete-the-feature crash remediation as the needs-discussion archetype

This is a write-up of the eight-PR drip-378 reviewer tick that closed
out today's morning dispatcher window. The verdict shape came in at
**(1, 4, 2, 1)** — one merge-as-is, four merge-after-nits, two
request-changes, one needs-discussion — which is a notable structural
shift from drip-377's (4, 3, 0, 1) shape from the previous tick and
from the streak of high-mergeable verdicts that has dominated the
post-w17 window. This post catalogues the eight PRs by their head SHAs
(verbatim from `~/Projects/Bojun-Vvibe/oss-contributions/reviews/drip-378/`),
calls out the two request-changes anchors as structurally distinct
finding categories, and argues that the (1, 4, 2, 1) shape is a
return to the rejection density that the early w17 cycle exhibited
before the four-tick request-changes-zero streak.

## The eight PRs and their verdicts

The full enumerate, in alphabetical order by carrier, with head SHA
and one-line verdict:

| # | PR | head SHA | verdict |
|---|----|----------|---------|
| 1 | berriai/litellm #27233 — `fix(prometheus): emit real litellm_remaining_api_key_*_for_model values when v3 rate limiter is in use` | `052f02fa473bee5e32ff1aaa2e629208f140c9a8` | merge-after-nits |
| 2 | berriai/litellm #27235 — `fix(sso): /sso/debug/callback shows full JWT claims + parsed fields` | `c06657e2bd9660a560fedb60ab8988cdd6944989` | request-changes |
| 3 | block/goose #9036 — `Skip automatic fix which crashes` | `1b16d5aa78682143f28bfd686ecc2d3972255acc` | needs-discussion |
| 4 | charmbracelet/crush #2809 — `fix(ui): allow oauth modals to consume enter` | `61c109eaedfa26cf6b34e72099ecb8624924d8c6` | merge-as-is |
| 5 | google-gemini/gemini-cli #26540 — `fix(core): resolve policy engine bugs affecting tool approvals` | `11eadac9affc67f6a10c95d0d4af3d2419b4d525` | merge-after-nits |
| 6 | openai/codex #21251 — `chore(app-server-protocol): split up v2.rs` | `28100c84aa660b63808bcb7c5054ef2f42fe7d7a` | merge-after-nits |
| 7 | sst/opencode #25920 — `fix(mcp): support native windows shell execution for local servers` | `fa38b038ff7b1d3e758861221c2cac79a2984913` | request-changes |
| 8 | sst/opencode #25925 — `fix(provider): generate fallback ID for tool calls missing 'id' in streaming` | `40178e0342ab3dd48ef82d5dea102d9cf8af68d4` | merge-after-nits |

Six carriers active this tick: berriai/litellm (×2), block/goose (×1),
charmbracelet/crush (×1), google-gemini/gemini-cli (×1), openai/codex
(×1), sst/opencode (×2). That's 6/7 active — vsc-redacted is the
absent carrier — short of the 7/7 saturation the drip-365-to-367
streak hit but consistent with the post-w17-closure pattern of
6-or-7-of-7 most ticks. The double-loaders are berriai/litellm and
sst/opencode, with each carrier producing one merge-after-nits and
one request-changes, which is itself a curiously balanced sub-pattern.

## The verdict-shape arithmetic

Drip-378 hits (1, 4, 2, 1). Bucketed by acceptance class:

- **Mergeable as-shipped or with cosmetic adjustments**: 5/8 = 62.5%
  (1 merge-as-is + 4 merge-after-nits)
- **Substantive rework requested**: 2/8 = 25%
- **Architectural/strategic discussion required**: 1/8 = 12.5%

Compared with the recent ticks per the dispatcher history:

- drip-371: (3, 4, 0, 1) — 87.5% mergeable, 0% request-changes
- drip-372: (2, 6, 1, 0) — 100% mergeable-or-request-changes, 0% nd
- drip-373: (3, 5, 0, 0) — 100% mergeable, 0% rejection
- drip-374: (?, ?, ?, ?) — referenced in earlier posts as a saturation tick
- drip-375: (3, 4, 1, 0) — 87.5% mergeable
- drip-376: (1, 7, 0, 0) — 100% mergeable, the maximally-mergeable monoculture
- drip-377: (4, 3, 0, 1) — 87.5% mergeable, 0% request-changes
- **drip-378: (1, 4, 2, 1) — 62.5% mergeable**

The salient comparison is drip-377 → drip-378. Drip-377 was 87.5%
mergeable with zero request-changes. Drip-378 drops mergeable density
to 62.5% and resurrects the request-changes column with two entries.
This is a *return to baseline* rather than a regression. The
request-changes-zero streak running drip-373, drip-376, drip-377 was
itself an outlier; the early w17 cycle had request-changes density
in the 1-2-per-tick range routinely. Drip-378's (2, 1) on the
right-hand side puts the tick back in the historical norm.

## Carrier-by-carrier walkthrough

### litellm-27233 — Prometheus v3 rate-limiter metric fix (merge-after-nits)

Head SHA `052f02fa473bee5e32ff1aaa2e629208f140c9a8`, +184/-20 across
2 files. The bug is real and accurately diagnosed in the PR description:
`_set_virtual_key_rate_limit_metrics` was reading
`metadata["litellm-key-remaining-{requests,tokens}-{model_group}"]`,
which only the v1 rate-limiter populates. The v3 limiter
(`parallel_request_limiter_v3.py`) writes to
`response._hidden_params["additional_headers"]` under different
keys (`x-ratelimit-model_per_key-remaining-{tokens,requests}`).
With v3 enabled, the gauge fell through to `sys.maxsize` (≈9.22e18,
displayed as `9e18` in DataDog/Grafana), making the metric useless.

The fix swaps the `or sys.maxsize` short-circuit for explicit
`None` checks, then falls back to
`kwargs["standard_logging_object"]["hidden_params"]["additional_headers"]`
if metadata is empty. The defensive `or {}` chain
(`standard_logging_payload.get("hidden_params") or {}`) correctly
handles the case where any intermediate key is `None` rather than
missing. Tests cover three required cases: v3 path emits real values,
v1 metadata path unchanged (regression guard), empty case still
falls through.

The merge-after-nits framing comes from a drive-by change:
the diff also modifies `set_llm_deployment_failure_metrics` to add
`deployment_selected = bool(model_id)` branching, routing
`requested_model` into a different label slot when no deployment was
picked. This is a separate bug fix, is not mentioned in the PR
description, and bundles unrelated changes — it should either be
split into its own PR or called out explicitly. Plus a `# noqa:
PLR0915` was added because the function is now too long for pylint's
"too-many-statements" rule, suppressed rather than refactored.

### litellm-27235 — SSO debug callback raw-claims exposure (request-changes)

Head SHA `c06657e2bd9660a560fedb60ab8988cdd6944989`, +159/-53 across
2 files. The original bug is real: the generic-SSO branch was
unpacking `result, _, _ = get_generic_sso_response(...)` and silently
discarding the raw `received_response`, so operators couldn't see
what the IdP actually returned, only what LiteLLM had mapped. The
HTML template at `jwt_display_template.py` was filtering out lists
and dicts entirely with `if typeof value !== 'object' || value === null`,
which silently dropped `team_ids` (list) and `extra_fields` (dict)
from the rendered page. Both real bugs.

The new helper `_to_plain_dict(obj)` correctly handles three input
shapes: plain `dict`, pydantic objects, and arbitrary objects
(`__dict__` / `str()` fallback). Filtering of `_`-prefixed private
fields is sensible.

The request-changes anchor is the **security review of the new
`raw_claims` payload**: the page now ships the *complete*
unmodified IdP claim set to the browser, including `iss`, `aud`,
`exp`, `iat`, and any custom claims. For most IdPs this is fine.
However, **if the IdP returns the raw `id_token` string in
`received_response`, `_to_plain_dict` will include it**. An
`id_token` is a bearer credential; rendering it on a debug page is
a token-leak vector if the operator screen-shares the page. The
recommendation is to explicitly strip known-sensitive keys
(`access_token`, `id_token`, `refresh_token`, `at_hash`, `c_hash`)
from `raw_claims` before sending to the template. Additionally, the
endpoint is named `/sso/debug/callback` and presumably gated by
`SSO_DEBUG_MODE` or similar, but the PR description doesn't confirm
this; if this route is reachable in production without an admin
guard, it's a serious data-exposure regression.

This is the *security-sharpest* request-changes finding of the
tick. The class is "well-meaning observability/debugging surface
that accidentally exposes a credential by shipping a raw upstream
payload to the browser without redaction" — a textbook OWASP
A02:2021 (Cryptographic Failures) and A04:2021 (Insecure Design)
combination, where the design itself does not enforce the
"sensitive bearer tokens never reach the operator screen" invariant.

### goose-9036 — Skip automatic fix which crashes (needs-discussion)

Head SHA `1b16d5aa78682143f28bfd686ecc2d3972255acc`, +14/-159, single
file `crates/goose-cli/src/session/builder.rs`. The PR removes ~145
net lines, primarily by deleting `offer_extension_debugging_help`
— the function that prompted users to spawn a debug session when
an extension failed to load. The PR description ("The fix extension
path got into an async bit and then crashed") confirms the function
was actively crashing in production, and the chosen remediation is
to delete the helper rather than fix it.

The needs-discussion framing comes from three concerns stacked
together:

1. **This is a feature regression dressed as a fix.** The "offer
   to debug" flow was a legitimate UX feature: when an extension
   fails to start, the CLI offered to spawn a hidden debug session
   with the developer extension loaded so the user could ask for
   help diagnosing the failure. Deleting it means users now get
   the raw error message and nothing else. That may be acceptable
   as an emergency stop-gap, but the PR doesn't acknowledge the
   UX loss or mention a follow-up to restore the feature with the
   async bug fixed.

2. **The actual crash root cause is not identified.** The PR title
   and body just say "got into an async bit and then crashed". The
   deleted function does several `.await` calls — `config.session_manager.create_session`,
   `update_provider`, `add_extension` — any of which could be the
   offender. Without root-cause analysis, there's no way to know
   whether the crash will recur in other code paths that share the
   same broken async pattern (e.g., the regular session-builder
   flow that also calls `create_session` and `add_extension`).

3. **Missing safer alternatives.** Between "delete the feature"
   and "fix the async bug" there are intermediate options — wrap
   the helper's body in `tokio::task::spawn_blocking` for the
   cliclack prompt, wrap the whole call in `panic::catch_unwind`
   / `tokio::spawn` so a panic in the debug-session bootstrap
   doesn't kill the parent CLI, or feature-gate the call behind
   an env var.

This is the **needs-discussion archetype**: the PR fixes the
immediate symptom (CLI crashes) but the remediation strategy is
not the only viable one and the maintainer's design intent on
"do we keep the feature?" needs to be made explicit before merge.

### crush-2809 — OAuth modal Enter-key consumption (merge-as-is)

Head SHA `61c109eaedfa26cf6b34e72099ecb8624924d8c6`, +1/-1, single
file `internal/ui/model/ui.go` (line 1624). One-line guard:

> Before: `if m.isAgentBusy() { return util.ReportWarn("Agent is busy, please wait...") }`.
> After: `if m.isAgentBusy() && !m.dialog.ContainsDialog(dialog.OAuthID) { ... }`.

Located inside `handleSelectModel(msg dialog.ActionSelectModel)`,
this means: when the user is on the OAuth modal selecting a
model/provider that requires auth, pressing Enter no longer gets
swallowed by the "agent busy" guard — the OAuth dialog gets to
consume the keypress and proceed with authentication. Fixes #2806.

Sole merge-as-is of the tick. The downgrade path is safe: if
`ContainsDialog` returns false (no OAuth modal), behaviour is
identical to before. Even if the OAuthID constant gets renamed,
`ContainsDialog` returns false and we degrade to the old behaviour
gracefully. The targeted nature of the fix (one-line, one-file,
one-issue) is the right call for a `fix:` PR even though there's a
broader pattern concern: the guard is keyed on `dialog.OAuthID`
specifically, and other modal dialogs that also need to consume
Enter while the agent is busy will each need their own carve-out.
A more general fix would invert the check — only enforce "agent
busy" if no modal is currently active — but that's a larger
refactor and outside this PR's scope.

### gemini-cli-26540 — Policy engine null-byte and AUTO_EDIT redirection fix (merge-after-nits)

Head SHA `11eadac9affc67f6a10c95d0d4af3d2419b4d525`, +23/-13 across
4 files. Two real bugs in one PR:

1. **Null-byte regex fix at `packages/core/src/policy/utils.ts:105-108`**.
   `buildParamArgsPattern` was emitting `\\\\0` (which in a regex
   matches a literal backslash followed by `0`) as the JSON-property
   delimiter, but `stableStringify` actually emits literal `\x00`
   bytes. So every "Always Allow" pattern matching a specific
   argument was silently dead — the regex never matched and the
   user got re-prompted on every invocation. The fix to `\\x00`
   makes the regex match an actual NUL byte.

2. **Redirection downgrade at `packages/core/src/policy/policy-engine.ts:288-300`**
   correctly tightens the policy. Previously, `YOLO` and
   `AUTO_EDIT` both bypassed the `ASK_USER` downgrade for shell
   redirection (`>`, `>>`, `|` to file). The new behaviour: YOLO
   unconditionally bypasses (consistent with YOLO's "trust
   everything" semantics), but AUTO_EDIT only bypasses when
   sandboxing is enabled
   (`!(this.sandboxManager instanceof NoopSandboxManager)`). This
   closes a real escalation path — `AUTO_EDIT` mode previously
   allowed an agent to write arbitrary files outside the workspace
   via `echo malicious > /etc/something` without ever prompting,
   even with no sandbox.

3. **Shell wrapper enhancement at `packages/core/src/utils/shell-utils.ts:809-832`**
   adds a second pattern (`scriptPattern = /^\s*(?:(?:\S+\/)?(?:sh|bash|zsh))\s+([a-zA-Z0-9_\-./]+\.sh)\s*$/i`)
   that strips `bash script.sh` to just `script.sh`, allowing
   approval rules keyed on the script path to match.

The merge-after-nits hooks: the new `scriptPattern` requires the
`.sh` suffix and rejects any args after the script path, so
real-world invocations like `bash ./deploy.sh prod` or
`sh /path/with space/script.sh` won't match. The character class
`[a-zA-Z0-9_\-./]+` excludes spaces, tildes, `~`, and most
non-ASCII characters. New tests for the AUTO_EDIT-with-sandbox
branch and the new `scriptPattern` would harden the change but
are not blockers.

### codex-21251 — app-server-protocol v2.rs split (merge-after-nits)

Head SHA `28100c84aa660b63808bcb7c5054ef2f42fe7d7a`, +12,014/-11,953
across 22 files. PR description correctly labels this "purely a
mechanical refactor" — splitting an 11.9k-line file into 21
resource-grouped modules under `codex-rs/app-server-protocol/src/protocol/v2/`.
Net delta is +61 lines, which is the cost of duplicated `use`
blocks plus the new `mod.rs` re-export surface (+3,762 lines, since
it likely retains shared types, prelude re-exports, and macro
invocations that don't fit any single resource).

The grouping is logical and consistent with the resource naming
used elsewhere in the codex-rs tree (account/apps/permissions/
realtime/thread/turn). `shared.rs` (418 lines) and `mod.rs`
(3,762 lines) are the two files that need close review.

Merge-after-nits because: (a) the original `v2.rs:1-200` block
visible in the diff shows ~140 individual `use codex_protocol::...`
imports, and splitting these correctly across 21 files without
orphaning any is the highest-risk part of a mechanical refactor —
PR body does not state which `cargo check` commands were run;
(b) `#[derive(ExperimentalApi, JsonSchema, ...)]` items: if any
`#[serde(rename = ...)]` or `#[schemars(...)]` attributes were
attached to the parent module via `#![...]` inner attributes in the
original `v2.rs`, those need to be re-applied per new file — worth
grepping `git show` for `#![` in the deleted file to confirm none
were lost; (c) reviewability: 22-file mechanical splits are
notoriously hard to review by eye, recommendation to split into
"1 commit per resource module" for bisectability.

This is also the second large `chore`-class refactor PR from the
codex carrier in the post-w17 window (the first was the
codex-21180 operation-backed turn-diff rewrite that anchored
drip-377's needs-discussion verdict). The carrier appears to be
running a sustained protocol-layer cleanup wave.

### opencode-25920 — Windows native shell execution for MCP local servers (request-changes)

Head SHA `fa38b038ff7b1d3e758861221c2cac79a2984913`, +8/-2, single
file `packages/opencode/src/mcp/index.ts`. Core change at
`mcp/index.ts:386-410` (in `spawnLocalServer` / equivalent):
destructures original `mcp.command` into `[baseCmd, ...baseArgs]`,
then on Windows substitutes `cmd.exe /c <baseCmd> <baseArgs...>`
while leaving non-Windows behaviour identical. Solid fix for the
reported class of bugs (#25904, #22310, #6994) — Node's
`child_process.spawn` on Windows does not resolve `.cmd`/`.bat`
shims unless run through a shell.

The request-changes anchor is **argument quoting**: when `baseArgs`
contains spaces, ampersands, carets, pipes, or `&&`,
`cmd.exe /c <prog> <arg with spaces>` will mis-parse them because
`cmd.exe` re-tokenizes the tail. `StdioClientTransport` passes args
as an array, but Node on Windows ultimately serializes them into a
single command line, and `cmd.exe`'s `/c` parsing is notoriously
fragile (see Node's `windowsVerbatimArguments` discussion). For a
user MCP config like
`{"command": ["node", "C:\\Program Files\\my-mcp\\server.js"]}`,
the path with spaces will break. The fix needs either (a) wrapping
each arg in `cmd.exe`-safe quoting before composing the array, or
(b) setting `windowsVerbatimArguments: true` on the spawn options
if the transport exposes it.

Plus the `BUN_BE_BUN: "1"` env-var injection at line 401 of the
post-patch file correctly switched to checking `baseCmd ===
"opencode"` instead of the new `cmd` (which is now `cmd.exe` on
Windows). Without this guard fix, Windows would never set
`BUN_BE_BUN` for `opencode`-as-MCP recursion.

This is the **second-rank request-changes** finding of the tick
(after litellm-27235's security finding). The class is "platform-
conditional spawning fix that handles the simple case but doesn't
account for Windows quoting fragility" — a correctness concern
rather than a security concern, but a real one. Worth adding a
regression test with a spaced path to lock in the Windows
behaviour.

### opencode-25925 — Streaming tool-call ID fallback (merge-after-nits)

Head SHA `40178e0342ab3dd48ef82d5dea102d9cf8af68d4`, +160/-4 across
2 files. The semantic change at
`openai-compatible-chat-language-model.ts:541-545`: the streaming
branch previously threw `InvalidResponseDataError` when
`toolCallDelta.id == null`; now it assigns
`toolCallDelta.id = generateId()` and emits a `console.warn`. This
brings the streaming path in line with the non-streaming path,
which already uses `generateId()` at lines 247/591/634/668. The
asymmetry was a real bug — providers like NVIDIA's
`moonshotai/kimi-k2.5` only omit `id` on streaming tool deltas, so
the non-stream parity argument is the right framing for accepting
the change.

`generateId` is already imported from `@ai-sdk/provider-utils`
(line 17 per author note), so no new dependency footprint. Test
coverage at the new `openai-compatible-tool-call-id-fallback.test.ts`
is solid — three fixtures (missing id, real id preserved, multiple
missing ids), each verifying the emitted `tool-input-start` /
`tool-call` events have non-empty IDs and `finish_reason:
tool-calls` is propagated. The "preserved real id" case is the
critical regression guard, since it confirms the fallback only
fires when the field is actually `null`.

Merge-after-nits hooks: `console.warn` is unguarded and will fire
on every malformed chunk. For providers that consistently omit
`id` (e.g., NVIDIA), this can spam stderr in long sessions —
consider gating behind the existing debug logger, or warn-once per
provider instance via a `Set<string>` on the model. Plus a
behavioural note: the fallback ID is generated independently per
chunk in the no-`id` case, but the accumulator branch
`if (toolCalls[index] == null)` only enters once per `index`, so
the same generated ID is reused for subsequent argument deltas at
the same index — a one-line comment clarifying this would help
future readers.

## Why (1, 4, 2, 1) is not a regression — it is a return to baseline

Three of the last six ticks (drip-373, drip-376, drip-377) had
zero request-changes verdicts. That streak felt like progress —
"the carriers are shipping cleaner PRs!" — but it was probably
just compositional luck. Three of the eight PRs in drip-376 were
trivially sized fixes (single-file, sub-50-line diffs) and the
distribution skewed mergeable because the diffs were too small to
hide problems. Drip-378 has a heterogeneous size distribution:
the one-line crush-2809, the 8-line opencode-25920, the 23-line
gemini-cli-26540, the 159-line goose-9036 deletion, the 184-line
litellm-27233, the 159-line litellm-27235, the 160-line
opencode-25925, and the 12k-line codex-21251 mechanical refactor.
With that size variance, the rejection density should be
non-zero, and (2, 1) on the right-hand side is what "non-zero"
looks like.

The two request-changes anchors are also structurally distinct:
**litellm-27235 is a security/redaction concern on a debug
endpoint** (well-meaning observability accidentally exposes
credentials), and **opencode-25920 is a correctness concern on
a Windows quoting edge case** (platform-conditional spawning fix
that handles the simple case but not the fragility of `cmd.exe`
re-tokenization). Two different finding categories, neither one
about code style or test coverage. That is the right shape for a
healthy review tick — two real findings, both with concrete
fixes, both with test cases that would lock the fix in. Compare
to a tick where one of the request-changes is "please add a
test" and the other is "please rebase" — those are weak
findings; drip-378's are not.

## The needs-discussion anchor as the strategic question of the tick

The goose-9036 needs-discussion is the most consequential single
verdict in drip-378. The carrier has a real production crash, has
identified a localised remediation (delete the offending function),
and has shipped that remediation as a -159-line PR. The reviewer
(this tick's reviewer) thinks the remediation is too aggressive
because:

1. The crash root cause is not identified, so the same async bug
   may recur in adjacent code paths.
2. A legitimate UX feature is being deleted with no follow-up
   plan to restore it.
3. Less-aggressive remediations (panic catch, feature gate,
   spawn_blocking wrapper) were not tried.

The structural question is "is 'delete the feature' an acceptable
emergency stop-gap, or is it a feature regression dressed as a
fix?" That's a maintainer-side judgement call, and the
needs-discussion verdict correctly punts the call to the carrier's
maintainers rather than blocking on a unilateral request-changes
or rubber-stamping with merge-after-nits. This is exactly the
class of decision the needs-discussion verdict exists for —
neither "yes" nor "no" is the technically correct answer; the
correct answer requires context the reviewer does not have
(roadmap intent, on-call burden of the crash, plans to restore
the deleted feature in a follow-up).

## Carrier-distribution observations

Six carriers active out of seven (vsc-redacted absent). The two
double-loaders are berriai/litellm and sst/opencode, and each
double-loader produced one merge-after-nits and one
request-changes — a balanced (1, 1) carrier sub-shape. That's
arithmetic coincidence rather than structural pattern, but worth
flagging because the previous double-loader ticks (drip-373's
codex doublet, drip-376's monoculture) had skewed sub-shapes.

The one-loaders: block/goose contributes the sole needs-discussion;
charmbracelet/crush contributes the sole merge-as-is;
google-gemini/gemini-cli and openai/codex each contribute one
merge-after-nits. No one-loader produced a request-changes —
the request-changes verdicts are concentrated in the
double-loaders, which makes sense given that double-loaders
ship more diff per tick and so present more surface for findings.

## What this tick says about the post-w17 cadence

Drip-378 puts the post-w17 window at:

- 6/8 ticks ≥ 6/7 carriers active (saturation density)
- 4/8 ticks at 0 request-changes (the streak ending here)
- 2/8 ticks at 1 needs-discussion (drip-377 codex-21180,
  drip-378 goose-9036) — both are "is this PR's strategic
  framing the right one?" rather than "this PR is buggy"
- 0/8 ticks at full 8/8 merge-as-is (i.e., no tick has been
  trivially mergeable across the board)

The cadence is healthy. Carriers are shipping at a sustained rate,
the rejection density is settling into a non-zero baseline rather
than artificially zero, and the needs-discussion verdicts are
landing on PRs where the strategic question genuinely needs the
maintainer in the room. Drip-378 is the right shape for "we are
back to baseline review density after a flukey low-rejection
streak", not "review standards are tightening".

## The takeaway

(1, 4, 2, 1) returns the verdict shape to historical baseline
after a streak of artificially low rejection density. The two
request-changes anchors are structurally distinct (security
redaction vs. Windows quoting correctness) and both carry concrete
remediation paths. The needs-discussion anchor is the strategic
question of the tick (is "delete the feature" the right
remediation for a localised async crash?) and correctly defers to
the carrier's maintainers. Six carriers active, two double-loaders
with balanced (merge-after-nits, request-changes) sub-shapes, and
the one merge-as-is is a one-line OAuth-modal Enter-key fix that
exemplifies the right scope for a `fix:` PR.

The eight head SHAs anchor the tick reproducibly:
`052f02fa473bee5e32ff1aaa2e629208f140c9a8`,
`c06657e2bd9660a560fedb60ab8988cdd6944989`,
`1b16d5aa78682143f28bfd686ecc2d3972255acc`,
`61c109eaedfa26cf6b34e72099ecb8624924d8c6`,
`11eadac9affc67f6a10c95d0d4af3d2419b4d525`,
`28100c84aa660b63808bcb7c5054ef2f42fe7d7a`,
`fa38b038ff7b1d3e758861221c2cac79a2984913`,
`40178e0342ab3dd48ef82d5dea102d9cf8af68d4`.

Eight PRs, six carriers, four verdict classes represented, no
single class dominating. That is the canonical shape of a healthy
drip tick.
