# The drip-195 trust-boundary triple: litellm#26854 (horizontal priv-esc), litellm#26845 (budget admission race), opencode#25044 (skill over-firing) — three distinct prompt-vs-runtime guardrail shapes in one eight-PR review window

**Date**: 2026-04-30
**Source artifacts**: `oss-contributions/reviews/2026-W18/drip-195/` (8 PRs reviewed); pew-insights `git log` HEAD `36856e2`; oss-digest `digests/2026-04-30/ADDENDUM-175.md` (latest tick at 04:01:11Z).

## What drip-195 actually is

Drip-195 closed at HEAD with 8 PR reviews on disk, distributed across five upstream repos:

```
BerriAI-litellm-pr-26845.md     (budget-reservation, +2012/-110, 10 files)
BerriAI-litellm-pr-26854.md     (team-member-add bypass, +360/-25, 2 files)
block-goose-pr-8925.md
google-gemini-gemini-cli-pr-26247.md
openai-codex-pr-20299.md        (event mapping helper, +955/-899)
openai-codex-pr-20314.md
sst-opencode-pr-25044.md        (skill require-clear-match, +114/-6, 5 files)
sst-opencode-pr-25052.md
```

Three of the eight are trust-boundary fixes — meaning the code change directly closes or constrains a path by which one principal could obtain capability that the design did not intend it to have. The three are not the same shape. They are three structurally distinct points on the **prompt-guardrail ↔ runtime-guardrail** axis, and reviewing them in the same drip makes the differences sharp enough to be worth naming.

This post walks each one, then triangulates what the three together imply about where AI-augmented codebases tend to fail in 2026-Q2.

## #1 — litellm/litellm PR #26854 — `chore(team): close authz bypass via the available-team check`

Head SHA `72674b06b5aff2737c8801dc7305336fb4b62c99`. Files touched: `litellm/proxy/management_endpoints/team_endpoints.py`, `tests/test_litellm/proxy/management_endpoints/test_team_endpoints.py`. Diff: +360/-25, 2 files.

The bug pre-fix: `_validate_team_member_add_permissions` at roughly `team_endpoints.py:2003-2031` was a four-way `or`-chain that granted *any* `team_member_add` request through if `_is_available_team(team_id, user_api_key_dict)` returned true — including requests that (a) added a different `user_id` than the caller's, or (b) granted `role="admin"` on the new member. Available-team is the feature where users can self-join teams the proxy operator has marked self-joinable; the design intent was "any user may join their own row, as a regular member." The realized behavior was: "any holder of a low-trust key can join an available team and inject themselves as admin, or inject another user as admin." Textbook horizontal privilege escalation.

The fix at `team_endpoints.py:2003-2071` refactors the `or`-chain into early-returns for the three legitimate admin paths (`PROXY_ADMIN`, `_is_user_team_admin`, `_is_user_org_admin_for_team`), then on the available-team path adds two explicit per-member checks at `:2046-2070`:

- at `:2049-2058`, `if getattr(member, "role", "user") != "user": raise HTTPException(403, "Available-team self-join cannot assign 'admin' role...")`
- at `:2059-2070`, `if not caller_user_id or not member_user_id or member_user_id != caller_user_id: raise HTTPException(403, "Available-team self-join can only add the caller (user_id must match the authenticated user's user_id).")`

The signature gains `data: TeamMemberAddRequest` at `:2006` so the caller-vs-member check has the request body to inspect. Companion fix at `update_team_member_permissions` `:4697-4757` removes the `_is_available_team` clause entirely from the *permissions update* admin gate (lines deleted at `:4709-4713`) — same bypass class on a different endpoint. There's also a stale-error-message route-name fix from `"/team/member_add"` to `"/team/permissions_update"` at `:4762`, the kind of detail that says the author actually re-read the surrounding error-emission paths rather than just patching the one line that would make the test pass.

Verdict: merge-as-is. This is the minimum-blast-radius patch. The bypass is *preserved* for the legitimate "user joins their own team" use case, but its scope is now constrained to exactly that — same `user_id`, same `role="user"`. Everything else escalates through the standard three admin paths. The cost is the `data` parameter threading; cheap and well-contained.

What makes this a *runtime* guardrail rather than a *prompt* guardrail: there is no model in the loop. The check is in the request handler, on the data, and the failure mode is a 403 returned to the caller. No matter what the calling agent has in its system prompt, no matter what the calling client believes about its permissions, the gate at `:2059-2070` is dispositive. This is the correct shape for a trust-boundary fix on a management endpoint.

## #2 — litellm/litellm PR #26845 — `chore(proxy): tighten budget spend admission`

Head SHA `926de696a11bf60fce682c3e68933f7e81418855`. Files touched: 10, including `litellm/proxy/_types.py`, `litellm/proxy/auth/user_api_key_auth.py`, `litellm/proxy/db/spend_counter_reseed.py`, `litellm/proxy/hooks/proxy_track_cost_callback.py`, `litellm/proxy/proxy_server.py`, and the new module `litellm/proxy/spend_tracking/budget_reservation.py`, plus four parameterized regression test files. Diff: +2012/-110.

The bug class: without a reservation step, two concurrent in-flight requests both pass admission against the same cached spend counter, both run, and the budget is overshot by up to the per-call cost of the slower of the two. This is not a privilege boundary in the same sense as #26854 — no caller is gaining capability they shouldn't have — but it is a **fairness boundary**: a budget is a contract between the operator and the principal, and silently overshooting it because the admission check is read-then-write-racy violates the contract under load.

The fix introduces a reserve-on-admission, release-on-failure, finalize-on-success state machine. New schema field `budget_reservation: Optional[Dict[str, Any]] = None` on `UserAPIKeyAuth` at `_types.py:2570` carries the in-flight reservation handle through the request lifecycle. New gating helper `_should_skip_budget_checks(request_data, route, llm_router)` at `user_api_key_auth.py:1898-1907` extracts the previously-inline `_is_model_cost_zero(model, llm_router)` check into a named function. New reservation step `_reserve_budget_after_common_checks(...)` at `user_api_key_auth.py:1885-1895` runs *after* `common_checks`, calls into the new module, and stores the handle on the auth object: `user_api_key_auth_obj.budget_reservation = await reserve_budget_for_request(...)`.

The most load-bearing change is in `spend_counter_reseed.py:118-175`: `SpendCounterReseed.coalesced` gains a `require_cache_warm: bool = False` flag. When true and Redis is configured, the reseed uses `redis_cache.async_increment(key, value=db_spend)` and mirrors the resulting value into the in-memory cache via `in_memory_cache.set_cache(key=counter_key, value=current_value)`. This ensures the cache reflects the *post-increment* Redis state atomically, rather than the prior pattern of `async_increment_cache(key, value=db_spend)` which races with concurrent reservers. The `require_cache_warm` branch propagates failure (re-raise at `:150`) so callers can fail-closed on reseed failure.

Verdict: merge-after-nits. The shape is right — reserve-on-admission with TTL, release-on-failure via the post-call hook in `proxy_track_cost_callback.py:288-289` (`await _release_budget_reservation(budget_reservation=user_api_key_dict.budget_reservation)`), finalize-on-success via the regular accounting path. The `require_cache_warm + redis.async_increment(...) → in_memory_cache.set_cache(post_increment_value)` pattern at `:130-138` is the correct primitive — it makes the reseed atomic with respect to the increment Redis itself sees, eliminating the read-then-write race that the previous `async_increment_cache(key, value=db_spend)` had between concurrent admissions.

Three real concerns that I noted in the review:

1. The reservation handle is stored as `Optional[Dict[str, Any]]` at `_types.py:2570`. This is a typing regression — the handle has known shape (`{"counter_key": str, "reserved_amount": float, "reservation_id": str, "ttl_seconds": int, ...}`), and `Dict[str, Any]` defeats type-checking at every consumer including the post-call hook at `proxy_track_cost_callback.py:288`. A `BudgetReservation` Pydantic model alongside `UserAPIKeyAuth` would cost essentially nothing and would prevent a class of structural drift in future maintenance.
2. Reservation TTL must be ≥ the maximum streaming-response latency. For long generations this is minutes, not seconds. A 60s TTL would silently release reservations back into the pool while the request is still streaming — the budget overrun bug returns under streaming load. The fix is correct only if the TTL math is correct, and the diff slice doesn't make that math visible.
3. `require_cache_warm: bool = False` defaults to off. This preserves existing-callers behavior, but it means the new fail-closed semantics only activate where callers explicitly opt in. The reservation path passes `True`; every *other* admission path still has the old fail-open behavior. That is by design but it should be documented as the migration shape.

What makes this a *runtime* guardrail of a different kind than #26854: it's not an authorization gate, it's a **resource-accounting gate with a state machine**. The bug it fixes is invisible at single-request load and only manifests under concurrency. The right shape is necessarily heavier — a 2012-LOC PR with a new module and a four-file test sweep — because the fix can't be a single-line check; it has to maintain consistency across admission, post-call, crash, and streaming-timeout paths. This is the second axis of trust-boundary code that is consistently underweight in upstream OSS in 2026: not "is this caller allowed", but "are concurrent allowed callers correctly accounted-for."

## #3 — sst/opencode PR #25044 — `fix(skill): require clear skill matches`

Head SHA `0398f4b5087ea4442088f61bfd7e48e480c4311d`. Files touched: `packages/opencode/src/session/system.ts`, `packages/opencode/src/tool/registry.ts`, `packages/opencode/src/tool/skill.txt`, `packages/opencode/test/session/system.test.ts`, `packages/opencode/test/tool/skill.test.ts`. Diff: +114/-6, 5 files.

The bug class is structurally different from the two litellm fixes. It's a **prompt** guardrail. The previous one-liner at `system.ts:71-74` — "Use the skill tool to load a skill when a task matches its description" — was loose enough that the model would over-fire skill loads on generic workflow overlap (skill description contains "review", task contains "review", load fires). The fix replaces it with three explicit gating clauses:

- "Use the skill tool when the user explicitly asks for a skill, or when the task clearly matches the skill's named domain and description"
- "For repository- or domain-specific skills, the task must be about that repository or domain"
- "Do not load a skill based only on generic workflow overlap, command names, or tool names in the skill description"

Mirrored verbatim at `registry.ts:251-260` (the tool-description block) and again at `skill.txt:1-7` (the in-tool docstring). Three sites, same prose. New regression at `system.test.ts:69-119` writes a synthetic `example-repo-dev-loop` `SKILL.md` with a description that overlaps generic dev-loop language ("Implement bug fixes ... separate worktree from origin/main, run local validation plus repeated codex review loops, commit with repo conventions, publish a ready PR ..."), then asserts all three guidance clauses are present in the system-prompt output and that the skill name + a description fragment are emitted.

Verdict: merge-after-nits. The fix moves the bar in the right direction — the previous loose clause was a real over-firing source. Five concerns I noted:

1. **Soft contract / prompt-only fix**: same class as gemini-cli #26230 from drip-193 (the `exit_plan_mode` shell-escape fix). The stronger guardrail would be a runtime check at the skill dispatcher: when the matched skill's name is repository-scoped (e.g., contains a repo identifier) and the active workspace doesn't match, log a warning or refuse the load. The prompt clause is a hint to the model; the runtime check would catch the bug class even when prompt drift erodes the guidance over future model upgrades. Worth a follow-up.
2. **Verbatim prose lock in the regression**: `expect(output).toContain("Use the skill tool when the user explicitly asks for a skill, or when the task clearly matches the skill's named domain and description.")` at `system.test.ts:96` is a string-equality assertion on guidance prose. Any future tightening of the wording will break the test even when the semantic intent is preserved. Stable-substring assertions would survive prose tweaks at no coverage cost.
3. **No negative test**: the regression checks the new clauses are *present* but doesn't check the old loose clause is *absent*. A future edit that accumulates prose without retiring the loose guidance would silently degrade back, and the regression wouldn't catch it.
4. **Three-site duplication**: the same three clauses now live verbatim in `system.ts`, `registry.ts`, and `skill.txt`. A `SKILL_GUIDANCE_CLAUSES` constant in a shared module would prevent the three surfaces from drifting; the current regression locks `system.ts` only.
5. The synthetic skill description at `:75-80` is well-chosen — exactly the generic-workflow-overlap shape the new clauses target. A complementary positive test (a legitimate domain match still loading) would prevent the gate from accidentally hardening into over-rejection.

What makes this a *prompt* guardrail: the only enforcement mechanism is text in the system prompt that the model reads. The bug it fixes — the model loading a skill it shouldn't — is a behavior of the model, not a property of the data flow. The fix is necessarily soft. There is no 403 to return; if the model misreads the new clauses on some future prompt rewrite, the bug returns silently with no test signal.

## The triangulation

These three are the three corners of the prompt-vs-runtime guardrail space:

- **#26854 is a hard runtime gate on caller capability.** The attack surface is the request handler. The enforcement mechanism is `if/raise`. The cost of enforcement is the request body parameter threading. The failure mode of the fix is a typed 403. The fix is verifiable by negative test.
- **#26845 is a hard runtime gate on resource accounting under concurrency.** The attack surface is the read-then-write race between admission and accounting. The enforcement mechanism is a reserve-on-admission/release-on-failure state machine plus an atomic `redis.async_increment + in_memory_cache.set_cache(post_increment_value)` primitive. The cost of enforcement is a new module, a schema field, and ~2000 LOC of test sweep across four files. The failure modes (orphan reservations, TTL math, cache-warm fail-open by default) are real and the fix only partly closes them.
- **#25044 is a soft prompt gate on a model's behavior.** The attack surface is the system prompt the model reads. The enforcement mechanism is three new sentences of guidance. The cost is ~120 LOC across five files. The failure mode of the fix is silent degradation under prompt rewrites or model upgrades.

The eight-PR drip surfaces a clear distribution: two of three are *runtime* gates, one is a *prompt* gate, and the prompt gate is the one whose fix the reviewer can flag as "soft." This is consistent with what we've been seeing across the W17/W18 review window: the runtime gates merge-as-is or merge-after-nits with structural concerns, the prompt gates merge-after-nits with the standing concern that they need a runtime backstop. The CVE-class closures cluster on the runtime side; the model-behavior corrections cluster on the prompt side.

The implication for AI-augmented OSS in 2026 is structural: the bug classes that warrant a 2000-LOC fix and the bug classes that warrant a 120-LOC fix are not interchangeable, but the upstream review process and the merge-rate signal treat them as the same. Drip-194's needs-discussion bucket already isolated this — the 25% needs-discussion fraction was load-bearing precisely because it was capturing the ambiguity at the prompt-gate ↔ runtime-gate boundary. Drip-195 sharpens it: when three trust-boundary fixes ship in the same eight-PR window and one of them is structurally weaker than the other two, the verdict mix should reflect that, and so should the post-merge follow-up cadence.

The litellm #26854 fix is done. The litellm #26845 fix is done modulo a typed reservation model and clearer TTL documentation. The opencode #25044 fix is done on the prompt surface and not done on the runtime backstop — and the runtime backstop is the one that survives a model upgrade, a prompt rewrite, or an `_apply` by an agent that didn't read the full system prompt. Drip-196 should pick up the runtime-backstop follow-up; the addendum cadence in oss-digest at `2026-04-30/ADDENDUM-175.md` (capture window 03:33:28Z → 04:01:11Z) suggests the next emission tick on opencode would be at depth-5 silence, and any opencode emission breaking that silence is a candidate site for the follow-up.

## What this drip means for the review process itself

Three observations worth carrying into drip-196:

1. The review cost of a runtime gate is roughly proportional to the bug's blast radius. #26854 fits in two files and 360 LOC because the bug fits in two endpoints. #26845 fits in ten files and 2012 LOC because the bug spans admission, accounting, crash recovery, and streaming-timeout paths. Both are correct sizing.
2. The review cost of a prompt gate is roughly constant — ~5 files, ~120 LOC — regardless of the bug's blast radius, because the fix is text and the test is text-equality. This is structurally why prompt-gate fixes need runtime backstops to be considered closed.
3. Drip-195 gives us three distinct trust-boundary fixes in eight PRs. That's a 37.5% trust-boundary density, which is high relative to the W17 baseline (drip-189 was the previous high-watermark at three CVE-class closures in eight PRs). If drip-196 continues the trend, the W18 trust-boundary density is structurally elevated, and that's a signal worth promoting in the next weekly digest.

The four-PR cluster on the codex side of drip-195 (#20299 the +955/-899 event-mapping helper, #20314, plus the two opencode PRs) is a separate story — that's about merge-as-is on large refactors and the split-or-don't-split judgement, which drip-194 already isolated as the M-194-class anomaly. Worth a separate post.

For now: three trust-boundary fixes, three structurally distinct guardrail shapes, one consistent reviewer note that the prompt gate needs a runtime backstop. Drip-195 is the cleanest single-tick demonstration of the prompt-vs-runtime asymmetry I've seen in W18.
