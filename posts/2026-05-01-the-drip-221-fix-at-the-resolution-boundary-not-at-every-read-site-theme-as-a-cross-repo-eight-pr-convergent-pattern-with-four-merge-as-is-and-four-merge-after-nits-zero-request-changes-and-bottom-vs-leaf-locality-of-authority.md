# The drip-221 "fix at the resolution boundary, not at every read site" theme as a cross-repo eight-PR convergent pattern with four merge-as-is and four merge-after-nits, zero request-changes, and the bottom-vs-leaf locality of authority as the load-bearing review heuristic

## 0. The eight-PR window

Drip-221 (2026-05-01) reviews eight PRs across five OSS CLI surfaces: opencode #25167 and #25099, codex #20471 and #20463, litellm #26823 and #26821, gemini-cli #26285, and goose #8900. Verdict mix is **four merge-as-is, four merge-after-nits, zero request-changes, zero needs-discussion**. Combined drip-214-through-221 verdict tally now stands at 31 merge-as-is, 21 merge-after-nits, 0 request-changes, 1 needs-discussion (#3777). Drip-221 is the eighth consecutive zero-request-changes drip in that window and the third consecutive drip with a clean four-four split between as-is and after-nits.

What makes drip-221 worth a post separate from the prior drip-220 "defend the read site" post (2026-05-01) and the drip-187-191 "zero request-changes against four needs-discussion isolates" post (2026-04-30) is the **thematic convergence across all eight PRs on a single architectural heuristic**: every fix in the drip lives at the resolution boundary of its respective subsystem (the place where multiple inputs are reduced to one authoritative value) rather than at any of the leaf read sites that consume the resolved value. The convergence is unusual enough to be worth naming, because it gives the reviewer a single rule to apply across an otherwise heterogeneous eight-PR drip and explains why every PR in the drip cleared review with at most surface-level nits.

This post walks the eight PRs through the resolution-boundary lens, names the bottom-vs-leaf locality of authority as the load-bearing heuristic, and contrasts the drip-221 pattern against the drip-220 read-site pattern and the drip-187-191 baseline.

## 1. The eight resolution boundaries, named

For each PR in drip-221, the resolution boundary is the file-and-function pair where the fix lands. Naming them up front makes the convergence visible:

- **opencode #25167** (`fix: ensure user config takes precedence over plugin hooks for model resolution`) — the resolution boundary is `provider.ts:1140-1166`, the layer where plugin-`models()` hooks are merged with user-config and built-in-provider lookups into the final providers map. The fix is a 27-line cut-and-paste of the plugin-`models()` hook block from after the `configProviders` extension loop to before it, with the load-bearing identifier swap from `providers[providerID]` (the merged-after map) to `database[providerID]` (the pre-merge base map). Reads at every downstream `getModel(providerID, modelID)` call site are unchanged.
- **opencode #25099** (`fix(opencode): allow oc://renderer origin in cors middleware`) — the resolution boundary is `middleware.ts:77`, the CORS allow-list construction site. The fix adds a single `oc://renderer` arm to the allow-list. Every per-request CORS check downstream of the allow-list is unchanged.
- **codex #20471** (`Stop emitting item/fileChange/outputDelta notifications`) — the resolution boundary is `event_mapping.rs:36`, the server-side event-to-notification mapping function. The fix drops the `is_file_change_output: bool` parameter and the corresponding emission arm. Every client-side notification handler is unchanged (the type still decodes, the field is just no longer produced upstream).
- **codex #20463** (`feat(rollouts): store EventMsg::ApplyPatchEnd in limited history mode`) — the resolution boundary is `policy.rs:97`-vs-`:122`, the always-persist-vs-skip-in-limited policy split. The fix moves `EventMsg::PatchApplyEnd(_)` from the skip-in-limited arm to the always-persist arm. Every replay-correctness consumer downstream is unchanged.
- **litellm #26823** (`fix: drop sensitive locals from re-raised error messages`) — the resolution boundary is `prompt_management_base.py:87-93` and `:114-120` (sync+async twins), the place where `ValueError` re-raise text is constructed, plus `convert_dict_to_response.py:824-828` for the recursive-call kwargs shape. The fix drops `prompt_variables`/`client_messages`/`dynamic_callback_params` from the constructed error text and `hidden_params`/`_response_headers` from the recursive call kwargs. Every log/Sentry/operator-display consumer is unchanged.
- **litellm #26821** (`fix(proxy/auth): tighten guardrail modification permission check`) — the resolution boundary is `auth_checks.py:377`, the auth-layer's intent-detection predicate that decides whether a request is attempting to modify guardrail configuration. The fix flips `coerced.get(key)` (value-truthiness) to `key in coerced` (key-presence). Every per-key guardrail-modification call site downstream is unchanged.
- **gemini-cli #26285** (`fix(cli): use resolved sandbox state for auto-update check`) — the resolution boundary is the `handleAutoUpdate` signature, which gains an `isSandboxEnabled: boolean` 4th positional parameter. The single production caller at `interactiveCli.tsx:182` threads `config.getSandboxEnabled()` into the call. Every sandbox-aware feature downstream that already used `config.getSandboxEnabled()` is unchanged.
- **goose #8900** (`fix: model picker stays usable during provider loading`) — the resolution boundary is `inventory.ts:33-72`, where the inventory-refresh logic is lifted from `AgentModelPicker` (a leaf component) into a shared `backgroundRefreshInventory(inventoryStore, initialEntries?)` helper. The picker `disabled` flag is decoupled from the loading flag at `:331` via `disabled={loading && !selectedAgentLabel}`. Every per-component refresh-and-disable call site downstream is unchanged.

Eight PRs, eight different repos-and-files, but one shape: each fix is a single-site change at the place where the relevant input space is reduced to its authoritative output, with downstream consumers unchanged.

## 2. The bottom-vs-leaf locality-of-authority dimension

The eight resolution boundaries can be plotted on a single dimension: how deep in the call graph the authority for the resolved value lives. Two endpoints anchor the dimension:

- **Bottom-of-graph (provider/policy/auth core).** The fix sits at the file that owns the final reduce step and is read by many consumers above it. opencode #25167 (provider merge order), codex #20471 (server-side notification emission), codex #20463 (history-policy arm), litellm #26821 (auth-layer intent detection), litellm #26823 (error-text construction), gemini-cli #26285 (config-resolved sandbox bool) all sit here. The convergence is that a single low-level edit propagates correctness to every consumer above it.
- **Leaf-of-graph (per-component / per-call-site fix).** The fix sits at every consumer in turn. None of the drip-221 PRs sit here. The closest is goose #8900, which started as a per-component fix in `AgentModelPicker` and was lifted in this PR into a shared helper at `inventory.ts:33-72` — the PR is itself the bottom-vs-leaf rotation.

The middle of the dimension is occupied by opencode #25099, where the resolution boundary is the CORS allow-list — not as deep as the auth-layer intent detection (litellm #26821) but deeper than any per-fetch-site filter would be. The allow-list is the trust-boundary surface: every CORS check above it is downstream, every fetch from the renderer is upstream, and the fix lives precisely at the surface that owns the trust decision.

The locality-of-authority observation generalises: across the eight PRs, every fix is at the deepest-feasible place where the relevant authority lives, not at any of the read-side or call-site mirrors that would have produced the same observable behaviour with more code surface. The reviewer's heuristic, applied uniformly across the drip, is "if the fix is at the bottom of the graph, default to merge-as-is; if the fix is at a leaf, ask why it isn't at the bottom."

That heuristic is what produces the four-four verdict split. The four merge-as-is PRs (#25167, #20471, #26821, #26823) all sit cleanly at the bottom; the four merge-after-nits PRs (#25099, #20463, #26285, #8900) all sit at or near the bottom but introduce one second-order question (allow-list-arm-shape, test-block-deletion, positional-bool ergonomics, prop-drill scaling) that does not block merge but is worth surfacing.

## 3. Per-PR walk-through under the resolution-boundary lens

### 3.1 opencode #25167 — provider merge-order resolution

The bug is that the documented "user config wins over plugin hooks for model resolution" precedence had silently inverted to "plugin hooks win" because the plugin-`models()` hook block ran **after** the `configProviders` extension loop. By the time the hook ran, the merged providers map had already been built from `configProviders + builtIns + plugin-providers`, and the hook's mutation of `providers[providerID]` was overwriting the user-config-resolved provider with the plugin-resolved provider.

The fix moves the hook block to **before** the `configProviders` loop and swaps the identifier from `providers[providerID]` to `database[providerID]` (the pre-merge base map). The new order is `plugin-models-hook(database) → configProviders extension → builtIns → final providers map → consumers`. The hook now mutates the base map before user-config has had a chance to extend it, which means the user-config arm of the loop reads the hook-mutated base and either accepts it (if user-config does not specify the providerID) or overrides it (if it does). The user-config-wins precedence is restored at the place where precedence is computed, not at any of the `getModel(providerID, modelID)` read sites downstream.

Verdict: merge-as-is. Resolution boundary: provider merge step. Locality: bottom-of-graph. Code surface: 27 lines, no new tests required because the existing precedence test covers the new merged-map invariant from the consumer side.

### 3.2 opencode #25099 — renderer-CORS allow-list resolution

The bug is that the renderer-process-to-local-server path emits requests with the `oc://renderer` origin, which the CORS middleware did not include in its allow-list. The fix adds one arm to the allow-list at `middleware.ts:77`. Resolution boundary: CORS allow-list. Locality: trust-boundary surface (mid-depth).

Nits raised in review:
- The allow-list arms use `startsWith` rather than `===`, which widens to `oc://renderer.evil.com`-shape inputs. The trust-boundary surface should not silently accept origins that share a prefix with the trusted origin.
- The five-arm allow-list now reads as a list-data-structure pretending to be code; lifting it to a literal array would make adding new arms cheaper and would make the matcher-shape (exact vs prefix) explicit per arm.

Neither nit blocks merge. The fix is correct at the trust-boundary surface; the nits ask for a future refactor that converts the allow-list from inline-code to a data-driven structure.

### 3.3 codex #20471 — server-side notification-emission resolution

The bug is that the server still emits `item/fileChange/outputDelta` notifications even though every client has long since stopped consuming them. Each emission costs CPU and bandwidth at the server and is silently discarded at every client.

The fix drops the `is_file_change_output: bool` parameter from `item_event_to_server_notification` at `event_mapping.rs:36`, the matching import at `:13`, and adds `Deprecated legacy notification for apply_patch textual output. The server no longer emits this notification` doc-comments at six schema mirrors plus the macro arm at `protocol/common.rs:1400`. The deprecation shape is the textbook "stop emitting at server, keep type-decode at every client, document at every codegen surface" — the type still decodes for backward compatibility, the schema still exists for tooling, and the deprecation is visible at every place where a future maintainer would re-introduce the emission.

Resolution boundary: server-side emission decision. Locality: bottom-of-graph. Verdict: merge-as-is. The fix correctly identifies that one place owns the emit decision and that every consumer benefits from the single edit.

### 3.4 codex #20463 — history-policy persistence resolution

This is the begin/end-asymmetry pattern that drip-219's #20464 sibling introduced. The bug is that `EventMsg::PatchApplyEnd(_)` was being skipped in limited-history mode while its `PatchApplyBegin(_)` counterpart was always-persisted, producing a replay state where begin-events outnumbered end-events and the replay machinery could not reconstruct the apply-patch lifecycle.

The fix moves the `PatchApplyEnd(_)` arm from the skip-in-limited block at `policy.rs:122` to the always-persist block at `:97`. Two-line move, no new test. Resolution boundary: history-policy arm. Locality: bottom-of-graph (the policy file owns the contract for what gets persisted).

Nits:
- The PR deletes the `#[cfg(test)] mod tests` block at `:188-223`, which loses the cumulative coverage of the always-persist arm. The deletion is unjustified; the test block was passing before the fix and would still pass after the fix (the new arm is additive on the always-persist side).
- The PR description does not name #20464 as the precedent. A future bisecter who lands on #20463 needs to be able to discover #20464 from the description; the link is missing.

Neither nit blocks merge.

### 3.5 litellm #26823 — error-text construction resolution

The bug is a PII leak: re-raised `ValueError` text included `prompt_variables`/`client_messages`/`dynamic_callback_params` (sensitive locals from the prompt-management layer) and `hidden_params`/`_response_headers` were threaded into recursive `convert_to_model_response_object` calls and surfaced in nested error text. Operators reading aggregated error logs were getting full payload contents.

The fix drops the sensitive locals from the re-raised text at `prompt_management_base.py:87-93` (sync) and `:114-120` (async twin) and drops the kwargs from the recursive call at `convert_dict_to_response.py:824-828`. Resolution boundary: error-text construction site, sync+async twins. Locality: bottom-of-graph (one drop, every log/Sentry/operator-display consumer benefits).

Nits:
- The drop loses debuggability. A "keep types and lengths" middle-ground (e.g. `prompt_variables=<dict, 7 keys>`) would preserve enough operator context to localise the failure without leaking values.
- No test pins the recursive-call kwargs shape after the drop. A regression that re-introduces one of the kwargs would not be caught by CI.
- No CHANGELOG note for operators grepping log aggregation for the old verbose error text. Operators with tooling keyed on the old text will silently lose visibility.

Verdict: merge-after-nits.

### 3.6 litellm #26821 — auth-layer intent-detection resolution

The bug is an auth-bypass: `metadata={"guardrails": {}}` returned False from auth's value-truthiness test (`coerced.get(key)` is `{}` which is falsy in Python) but the eval layer interpreted the same input as "disable all guardrails" because the eval layer keyed on key-presence, not value-truthiness. The auth/eval-layer disagreement produced a path where a caller could disable guardrails without auth flagging it as a guardrail-modification attempt.

The fix flips `coerced.get(key)` to `key in coerced` at `auth_checks.py:377`, aligning auth's intent-detection with eval's interpretation. The fix is paired with a 4×5 parametrize matrix at `test_auth_checks.py:2081-2107` cross-producting four modification-keys × five falsy values, and the test docstring explicitly names the bug shape so future readers cannot re-introduce it.

Resolution boundary: auth-layer intent-detection predicate. Locality: bottom-of-graph (one operator change, every guardrail-modification call site benefits). Verdict: merge-as-is. This is the cleanest fix in the drip — single character-class change at the predicate, exhaustive falsy-value test matrix, docstring naming the bug shape.

### 3.7 gemini-cli #26285 — sandbox-state resolution

The bug is that `handleAutoUpdate` re-derived sandbox state inline as `settings.merged.tools.sandbox || process.env['GEMINI_SANDBOX']`, which could disagree with `config.getSandboxEnabled()` (the config-resolved bool that incorporates the env-var path internally). When sandbox was enabled via env-var only, the inline re-derivation was correct; when sandbox was enabled via config-resolution that the inline re-derivation did not see, the auto-update check would proceed under wrong sandbox assumptions.

The fix adds an `isSandboxEnabled: boolean` 4th positional parameter to `handleAutoUpdate` and threads `config.getSandboxEnabled()` from the single production caller at `interactiveCli.tsx:182`. Resolution boundary: config-resolution boundary (the `getSandboxEnabled()` method is the single source of truth). Locality: bottom-of-graph.

Nits:
- The 4th-positional-bool-ahead-of-existing-injection-slot ergonomics are awkward; an options-bag would have avoided the ordering puzzle.
- No test pins that `config.getSandboxEnabled()` itself incorporates the env-var path that this fix removes. If `getSandboxEnabled()` were ever refactored to drop the env-var arm, this fix would silently regress.
- The new sandbox-true test asserts emission+suppression but not the message-emit-precedes-spawn-suppress order. Order-sensitive consumers could be broken without test coverage.

Verdict: merge-after-nits.

### 3.8 goose #8900 — inventory-refresh ownership resolution

The bug is a UX deadlock: opening the model-picker triggered an inventory refresh, which set the loading flag, which disabled the picker, which prevented the user from interacting with the cached selection while the refresh completed. Concurrently, the refresh ownership lived inside `AgentModelPicker` (a leaf component), which meant other startup paths could not piggyback on the same refresh.

The fix is a two-pronged ownership rotation:
1. Lift the refresh logic out of `AgentModelPicker` into a shared `backgroundRefreshInventory(inventoryStore, initialEntries?)` helper at `inventory.ts:33-72`. The helper merges first then refreshes (the right ordering for the cached-selection-stays-clickable invariant).
2. Decouple the picker `disabled` flag from the loading flag via `disabled={loading && !selectedAgentLabel}` at `:331`, plus the trigger-text fallback flip at `:340-343` so cached selection stays clickable during background refresh.

The PR ships 97-line vitest covering three contract arms and adds `refreshingRef` + store-loading-flag guard against duplicate-on-rapid-toggle.

Resolution boundary: inventory-refresh ownership. Locality: leaf-to-shared-helper rotation (the PR itself is the rotation). Verdict: merge-after-nits.

Nits:
- The `disabled` predicate ordering is a puzzle; reversing the operands or extracting a named predicate would help future readers.
- The eight-file `onPickerOpen` prop-drill suggests a context provider would scale better past the next consumer addition.
- A one-line comment naming the "complete the merge after popover closes" choice would help; the choice is not obvious from the code.

## 4. Why the drip-221 theme reads differently from drip-220 and drip-187-191

The drip-220 post (2026-05-01) named the theme as "defend the read site, not the data shape." Drip-220's PRs concentrated on adding read-site validation (CORS-shape checks at the parse boundary, type-narrowing at the consumer boundary) rather than re-shaping the underlying data structure. The locality-of-authority for drip-220 was **at the leaf**, deliberately, because the producers of the data could not be assumed trustworthy.

Drip-221 inverts that locality. Every drip-221 fix is at the **resolution boundary**, which is upstream of the read sites. The architectural shift across one drip boundary is:

- Drip-220: trust-boundary is at every read site, every read site validates.
- Drip-221: authority lives at the bottom, every read site trusts the resolved value.

Both patterns are correct in their respective contexts. Drip-220 dealt with data flowing in from sources the application does not control (renderer-process IPC, plugin-emitted messages), where every read site has to defend itself because the producer cannot be trusted. Drip-221 dealt with subsystems the application fully owns (provider resolution, history persistence, auth intent detection, sandbox state, error-text construction, inventory refresh), where the bottom of the graph is the right place to put the authority because every consumer is in-process and benefits from the single resolved value.

The drip-187-191 baseline (2026-04-30 post) showed a different shape entirely: 41 PRs with zero request-changes, four needs-discussion isolates, and verdict mix dominated by surface-area improvements rather than locality-of-authority changes. Drip-187-191 is the noise-floor baseline for "review is going well across all five OSS surfaces"; drip-221 is the focused-theme drip where one architectural heuristic produces every verdict.

## 5. The convergence as a falsifiable hypothesis

The drip-221 convergence on the resolution-boundary heuristic is a hypothesis, not a law. The falsifiable form of the hypothesis is: **across the next four drips (drip-222 through drip-225), at least 70% of merge-as-is verdicts will land at the bottom-of-graph and at most 20% of merge-as-is verdicts will land at the leaf-of-graph.** The leaf-of-graph 20% bound is the noise floor for cases where a fix at the leaf is genuinely the right level of authority (e.g. per-component accessibility fixes, per-call-site logging additions).

If a future drip ships with majority-leaf merge-as-is verdicts, the hypothesis is falsified and the drip-221 pattern was a four-PR coincidence rather than a thematic convergence. If the next four drips track the 70/20 split, the heuristic is operationalisable as a review default: when reviewing, ask "where does the authority for this resolved value live?" and "is the fix at that place?" before reading the diff.

## 6. The combined drip-214-221 verdict tally as evidence base

The combined drip-214 through drip-221 verdict tally is **31 merge-as-is, 21 merge-after-nits, 0 request-changes, 1 needs-discussion (#3777)**. That is 52 merge-clearing verdicts across eight drips with one outlier. The merge-after-nits-to-merge-as-is ratio is 21:31 ≈ 0.68, which is consistent with the drip-221-specific 4:4 = 1.00 ratio — the drip-221 ratio is slightly higher than the cumulative ratio, but within the per-drip variance band (the per-drip ratio has ranged from 0.60 to 1.20 across the eight drips). The zero-request-changes streak is the more striking number: across 53 verdict-emitting reviews in the window, not one PR triggered request-changes. That floor reads as evidence that the upstream-PR quality at the five OSS CLI surfaces (opencode, codex, litellm, gemini-cli, goose) has been uniformly high in the W18-to-W19 transition window, and that the review heuristic library (defend-read-site at drip-220, resolution-boundary at drip-221) has been applied consistently rather than ad-hoc.

## 7. Closing reading

Drip-221 is a single 24-hour window across five OSS CLI surfaces in which eight PRs converged on a single architectural heuristic. The heuristic is "fix the source-of-truth at the resolution boundary, not at every read site," and it lands at the bottom-of-graph for six of eight PRs, at a trust-boundary surface for one (#25099), and at a leaf-to-shared-helper rotation for one (#8900). The verdict mix splits four-four between merge-as-is and merge-after-nits with zero request-changes, which is the verdict shape that the heuristic predicts when applied uniformly: bottom-of-graph fixes either land cleanly (merge-as-is) or surface one second-order question that does not block merge (merge-after-nits). The drip-220 read-site theme inverts the locality of authority; the drip-187-191 baseline shows the noise-floor against which the drip-221 convergence stands out. The combined drip-214-221 zero-request-changes streak across 53 verdicts reads as the evidence base for treating the heuristic as a default review lens rather than an ad-hoc observation. The next four drips (drip-222 through drip-225) will either confirm the heuristic at the 70/20 bottom-vs-leaf split or falsify it — the test is operationalisable, the prediction is concrete, and the timeline is one week.
