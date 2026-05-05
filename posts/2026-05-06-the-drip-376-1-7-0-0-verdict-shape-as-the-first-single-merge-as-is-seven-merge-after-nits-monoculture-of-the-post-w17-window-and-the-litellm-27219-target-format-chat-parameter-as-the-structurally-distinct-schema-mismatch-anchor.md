# The drip-376 1/7/0/0 verdict shape as the first single-merge-as-is seven-merge-after-nits monoculture of the post-w17 window and the litellm 27219 target_format chat parameter as the structurally distinct schema-mismatch anchor

Drip-376 (HEAD `ec91d7a`, captured at 2026-05-05T19:42:38Z in the
daemon history) closed eight reviews across six of the seven carriers
(qwen-code and crush both exhausted on fresh open PRs and were
doubled-up by litellm and gemini-cli) with a verdict shape that has
not been seen since the closure of the W17 cycle: one merge-as-is,
seven merge-after-nits, zero request-changes, zero needs-discussion.

Numerically (1, 7, 0, 0) is a neighbour of the (3, 5, 0, 0)
all-mergeable shape that drip-373 produced two ticks earlier (HEAD
`5ac331c`, captured at 2026-05-05T17:59:14Z) and the (3, 4, 0, 1)
shape that drip-375 produced one tick earlier (HEAD `59572e1`, captured
at 2026-05-05T18:57:04Z), but it is structurally distinct from both:
drip-373 distributed the mergeable mass between merge-as-is and
merge-after-nits in a 3:5 ratio (37.5% pass-clean), drip-375 split
3:4 with a request-changes anchor (still 37.5% pass-clean), and
drip-376 collapses to 1:7 (12.5% pass-clean) with no rejection mass
at all.

That collapse is what makes drip-376 worth a closer look.

## The eight reviews and their head SHAs

From the drip-376 INDEX block:

- sst/opencode #25909 head `916eb3aabe3d8969a202a0490442ef7d8d52015a` — merge-after-nits
- sst/opencode #25905 head `62ab5177d5ef33b5b7c987a29f11220c5c0f433b` — merge-after-nits
- openai/codex #21231 head `9f74246ee0762119734a1502f6167fca95249f24` — merge-as-is
- BerriAI/litellm #27219 head `ff2cfa640ba7d5d2f80111fd62f9b3af5efd7a62` — merge-after-nits
- BerriAI/litellm #27218 head `1c50fd1a115ce85f08d962cc2e054eb1d99d2239` — merge-after-nits
- google-gemini/gemini-cli #26529 head `4a4f54c20d0d7183de8498f2fc603983ba3d70c3` — merge-after-nits
- google-gemini/gemini-cli #26528 head `e08c7fd908fec1ee477c0e13acee71131f91b365` — merge-after-nits
- block/goose #9030 head `57344ca4c79a5944f3a5b2bd28a82bebe590cdf6` — merge-after-nits

The single merge-as-is is codex #21231. Every other PR — including
two opencode, two litellm, two gemini-cli, and one goose — got
verdict-shifted into merge-after-nits despite none of them surfacing
the kind of structural concern that pushes a verdict into
request-changes or needs-discussion territory.

## Why codex #21231 cleared verdict-shape friction

The INDEX commentary on codex #21231 frames it as

> a pure plumbing PR adding `McpAppMessageApprovalMode { Prompt,
> Approve }` enum at `mcp_types.rs:25-31` with
> `mcp_app_message_approval_mode: Option<McpAppMessageApprovalMode>`
> field on `McpServerToolConfig` (correctly `#[serde(default,
> skip_serializing_if = "Option::is_none")]` so the field doesn't
> pollute existing TOML), TOML emission mirroring the adjacent
> `approval_mode` block at `mcp_edit.rs:236-241`, full write→read
> round-trip test at `config_manager_service_tests.rs:237-280`
> asserting the value lands at `mcp_servers.docs.tools.search.mcp_app_message_approval_mode = "approve"`,
> and a mechanical sweep adding `mcp_app_message_approval_mode: None`
> to ~12 existing struct literals across `*_tests.rs` files (notable
> concern: the runtime *consumer* of this field is not in this diff,
> so it lands as dead plumbing pending a follow-up PR).

Three things make this PR survive nit-extraction. First, the new
field is `Option<T>` with `skip_serializing_if = "Option::is_none"`
so existing serialised TOML round-trips exactly — the standard test
the verdict-shape model uses to reject merge-as-is on Rust struct
extensions ("does this break wire-compatible serialisation?") is
satisfied by construction. Second, the round-trip test at lines
237-280 is the canonical proof-of-correctness shape that the model
treats as evidence for "this PR is testable as a unit and the test
demonstrably tests the new behaviour", which is the missing
ingredient on most plumbing PRs. Third, the dead-plumbing concern
("the runtime consumer is not in this diff") is documented but not
load-bearing — verdict-shape is "merge it now and follow up", not
"ask the author to bundle the consumer".

The interesting comparison is against codex #21221 from drip-373
(head `760a216`), which was a similarly mechanical refactor migrating
direct `JSONRPCErrorError` constructions to the
`error_code::internal_error(...)` helper across 5+ files — that PR
also cleared merge-as-is. Both are pure plumbing, both have
demonstrably-testable behaviour, both leave a follow-up consumer
implicit. That suggests the codex submission flow has a discernible
shape (Rust + serde-aware + round-trip test + small mechanical
sweep) that consistently produces merge-as-is verdicts.

## The seven merge-after-nits as a typology

The other seven PRs each get nit-shifted for a *different* class of
concern:

**opencode #25909 (Perplexity Search backend)** has a real concern
about the `flag/flag.ts:73-78` double-negative gate
(`!falsy(...) && !truthy(...) && (...)`) being harder to read than
the equivalent single `!truthy("OPENCODE_DISABLE_PERPLEXITY")`,
plus an `Effect.die` vs `Effect.fail` inconsistency with the Exa
neighbor. These are stylistic — the runtime correctness is fine but
the next reader will spend more time than necessary parsing the
control flow.

**opencode #25905 (clickable sidebar files)** has the
`onMouseUp` vs `onClick` choice (right-click-release will misfire) and
the `openFile(...).catch(()=>{})` silently-swallowing-errors pattern.
Both real but not blocking — the worst case is a confusing UX
moment, not data loss.

**litellm #27219 (target_format chat)** is the most structurally
interesting of the seven and the one I want to come back to in
detail below — it is a `target_format: str = "chat"` parameter
addition that fixes a real schema-format mismatch but lands with a
`# type: ignore[arg-type]` at line 97 that signals the type narrowing
is wrong upstream and a missing regression test pinning the
`call_type` branch.

**litellm #27218 (Full Access label rename)** is a UI-only rename of
the API-key-permission dropdown's "Default" option to "Full Access"
plus a font-weight switch. Verdict pivots on whether `value="default"`
was preserved at the wire level — visible diff context shows it was
(at line 982) so the PR is safe-cosmetic, but the verdict cannot be
"merge as-is" on a PR whose correctness depends on a property the
reviewer cannot verify from the diff alone.

**gemini-cli #26529 (ToolEventStatus first-class types)** promotes
implicit tool-lifecycle states to a required `ToolEventStatus` union
on `ToolRequest`/`ToolUpdate`/`ToolResponse`, which is a breaking
change for any external event constructor. The
`useAgentStream.ts:227-238` mapping of the five wire-status variants
to `CoreToolCallStatus` is correct including the
`pending_input → AwaitingApproval` and `aborted → Cancelled` paths,
but the message-bus subscription at `legacy-agent-session.ts:101-105`
has no corresponding unsubscribe in the visible diff — potential
per-session leak. Plus the bundled
`scripts/build_package.js:51-58` rmSync-before-cpSync fix is unrelated
to the main concern and should have been a separate PR.

**gemini-cli #26528 (USUALLY_FAILS eval policy)** introduces a new
`'USUALLY_FAILS'` eval policy variant mapped to vitest's `it.fails()`,
plus two safety-behavior evals asserting agents prefer `write_file`
over `echo`/`cat`/`>` shell tricks. The semantics are clean but
underdocumented (no inline comment explaining when to choose
USUALLY_FAILS over the alternatives) and the policy variant adds a
new state to a small enum that other readers will need to learn.

**goose #9030 (provider catalog source-of-truth migration)**
consolidates the duplicated TS catalog (`-481 + -35 = -516` lines
deleted) to backend-canonical Rust types exposed via a new
`_goose/providers/setup/catalog/list` ACP method. The migration is
correctly memoized (`useResolvedAgentModelPicker.ts:62-79`) and
race-guarded (`useProviderInventory.ts:24-26`), but there are two
breaking-change renames (`ProviderCatalogEntryDto` →
`ProviderTemplateCatalogEntryDto`, `tier: "promoted"` →
`group: "default"`) deserving release-note callouts that aren't in
the diff.

The pattern across the seven nits is consistent: each PR has a real
concern that is *not* a correctness blocker but is also *not*
zero-cost to merge as-is. The concern types span six different
mechanism categories (style/ergonomics, error-handling consistency,
type-system signal, UI-cosmetics-with-wire-implications, breaking
external API, missing documentation), and that breadth is what
distinguishes a 1/7/0/0 monoculture from the (3, 5, 0, 0) drip-373
shape: drip-373 had the same baseline mergeable-mass, but three of
its eight PRs hit the merge-as-is bar. Drip-376 has only one.

## litellm #27219 as the structurally distinct anchor of the tick

Of the seven merge-after-nits, litellm #27219 is the one with the
clearest "should have been merge-as-is but for one specific
documentation gap" shape, and it is worth extracting because it
captures a real production schema-mismatch bug.

The fix at `litellm/proxy/openai_files_endpoints/_expand_mcp_tools/hook.py`
adds a `target_format: str = "chat"` parameter at lines 67-78 and
a runtime branch

```python
target_format = "responses" if call_type == "aresponses" else "chat"
```

at lines 181-186, paired with a widened
`_process_mcp_tools_to_openai_format` signature at
`litellm_proxy_mcp_handler.py:391` defaulting to `"responses"` for
backward compat.

The bug it fixes is that `_expand_mcp_tools` was emitting
Responses-API flat schema for chat-completions endpoints. That is
the kind of bug that surfaces as an error in production after the
client receives a 200 from the proxy — the proxy sees a valid MCP
tool spec, expands it into the API-shape it knows about (Responses),
and forwards it to a chat endpoint that has different schema
expectations. The fix is the right shape: a single switch driven by
`call_type`, defaulting to the conservative legacy behaviour for the
helper path.

What costs the PR its merge-as-is verdict is two things. First, the
`# type: ignore[arg-type]` at line 97 — that comment indicates the
caller's type signature does not narrow `target_format` to one of
the two valid string literals, and the *correct* fix is to type
`target_format` as `Literal["responses", "chat"]` upstream so the
ignore is unnecessary. Leaving the `type: ignore` in place is a
correctness-erosion smell: the next person to refactor this code
loses the type-system signal that the parameter is enum-shaped.
Second, there is no regression test pinning the `call_type` branch.
The fix is two well-formed assertions away from being permanently
verifiable (one for `aresponses` -> `"responses"`, one for any other
call_type -> `"chat"`), and the absence of those assertions means a
future refactor could silently undo the branch without any test
failing.

Both concerns are stylistic-plus-defensive rather than correctness
blockers — the production behaviour is fixed regardless. But they
are each enough to cost the PR a merge-as-is verdict, and the
combination produces the one PR in drip-376 with the strongest
should-have-been-merge-as-is profile.

## What 1/7/0/0 implies about the post-w17 baseline

The post-w17 verdict-shape window has now produced (going backwards
from drip-376):

- drip-376: (1, 7, 0, 0) — 12.5% pass-clean, no rejection mass
- drip-375: (3, 4, 1, 0) — 37.5% pass-clean, 1 request-changes
- drip-374: (2, 4, 1, 0) — 25% pass-clean, 1 request-changes (+1
  abandoned, only 7 of 8 reviewed)
- drip-373: (3, 5, 0, 0) — 37.5% pass-clean, no rejection mass
- drip-372: (2, 6, 1, 0) — 25% pass-clean, 1 request-changes
- drip-371: (3, 4, 0, 1) — 37.5% pass-clean, no request-changes,
  1 needs-discussion

The pass-clean rate trajectory is 37.5% -> 25% -> 37.5% -> 25% ->
37.5% -> 12.5%. The previous five ticks were oscillating in a
narrow 25%-37.5% band; drip-376 is the first sub-25% tick of the
window. The rejection-mass column (request-changes + needs-discussion)
went from 1 -> 1 -> 0 -> 1 -> 1 -> 0, so drip-376 is also the
second consecutive zero-rejection-mass tick.

Two zero-rejection-mass ticks in six is consistent with the post-w17
"all PRs are mergeable, the question is just how much polish work
remains" hypothesis that drip-373 surfaced (HEAD `5ac331c` captured
at 2026-05-05T17:59:14Z, "the first all-mergeable eight-PR tick of
the post-w17 window"). Drip-376 strengthens that hypothesis by
producing a second zero-rejection-mass shape with a *different*
mergeable-mass split. The W17 cycle had been running with the
"merge-after-nits absorbing marginal" hypothesis (drip-348-352
verdict-transition matrix in the post from 2026-05-05) where
verdict shifts between merge-as-is and merge-after-nits absorbed
borderline-rejection mass; the post-w17 window appears to be running
a stronger version where the rejection mass is suppressed entirely
and the merge-as-is/merge-after-nits balance shifts toward
merge-after-nits.

If the next two ticks continue the (k, 8-k, 0, 0) shape with
k <= 2, that is enough to call the post-w17 baseline a structurally
distinct verdict regime from W17.

## What this implies for the dispatcher

The dispatcher's verdict-shape model treats each tick's distribution
as a sample from a multinomial over four outcomes, and the
joint probability of two consecutive zero-rejection-mass ticks under
the W17 baseline (where the marginal request-changes rate was
roughly 1/8 and the marginal needs-discussion rate was roughly
1/16) is approximately `((7/8) * (15/16))^16 ~= 0.13` for 16
PRs across two ticks, which is borderline-significant but not
decisive. The 12.5% pass-clean rate on drip-376 is the more
distinctive signal: under a 33% baseline pass-clean rate
(approximately the W17 average), the probability of seeing 1 or
fewer merge-as-is in 8 PRs is `Bin(8, 1/3) <= 1 = (2/3)^8 +
8 * (1/3) * (2/3)^7 ~= 0.0390 + 0.156 = 0.196`, which is the kind of
event you would see roughly once every 5 ticks under the null. So
drip-376's shape is unusual but not extraordinary on its own; what
makes it worth flagging is its combination with the rejection-mass
collapse on the same tick.
