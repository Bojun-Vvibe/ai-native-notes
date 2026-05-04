# openai/codex #20937 at 53dbdbf as the server-driven nudge pattern, and the closed Rust enum versus open JSON string forward-compat trap

**Source PR:** `openai/codex#20937` — *Thread backend-selected near-limit prompt payload through rate limits*
**Head SHA:** `53dbdbfa9d603270d405d5f8aac78d1844a1e8b0`
**Scope:** +558 / −3 across 27 files
**Drip:** drip-338 (2026-05-04)
**Verdict:** merge-after-nits

This is one of those PRs where the *line count* understates the design surface. 558 added lines spread across 27 files looks at first like a noisy mechanical change — schemas regenerated, protocol enums extended, TUI tests bumped. But once you read the diff carefully, it's a clean instance of two well-known patterns colliding: the *server-driven UX nudge* pattern, which is how modern API products avoid hard-coding policy in clients, and the *closed-enum-over-open-wire* trap, which is how those same products eventually break their own clients. Both are worth pulling apart in detail because the same shape shows up in carrier after carrier — `sst/opencode`, `BerriAI/litellm`, `block/goose`, `charmbracelet/crush` all run into a version of it the moment the backend wants to push policy down.

## Section 1 — what the PR actually does, in one paragraph

The change adds a single new optional field, `currentUsageLimitNudge`, to the rate-limits notification payload. The field is shaped as `UsageLimitNudge { action, threshold }` where `action` is one of `add_credits` | `upgrade` and `threshold` is a `uint8` percentage. Today's client behaviour is "when usage crosses 80%, show the upgrade CTA"; with this PR, the *backend* tells the client which CTA to show and at what threshold. The client becomes a renderer.

Concretely: the schema files are touched in three flavours — root, v2, and per-notification (`schema/json/`, `schema/json/v2/`, and notification-specific files). The protocol crate (`codex-rs/protocol/src/protocol.rs`) gains the `UsageLimitNudge` Rust types. The v2 app-server protocol (`codex-rs/app-server-protocol/src/protocol/v2.rs`) extends `GetAccountRateLimitsResponse` with the new optional field. The backend client (`codex-rs/backend-client/src/client.rs`) reads the field from upstream. The openapi-models crate (`codex-backend-openapi-models/src/models/usage_limit_nudge.rs`) is regenerated. And the TUI status renderer (`codex-rs/tui/src/status/`) plus chatwidget (`codex-rs/tui/src/chatwidget/`) get updated tests that exercise the new path.

That's the whole change. No business logic moved. No behaviour visible to users *yet*. Just a new field threaded end-to-end through five crates.

## Section 2 — why the server-driven nudge is the right pattern

Hard-coded client UX rules age badly. The naive approach to "show an upgrade prompt when the user is near their quota" is to write something like, in pseudocode:

```rust
if usage_pct >= 80.0 {
    show_nudge(NudgeKind::Upgrade);
} else if usage_pct >= 95.0 {
    show_nudge(NudgeKind::AddCredits);
}
```

This is how every CLI does it on day one. Then three things happen in sequence:

1. **Pricing changes**. The product wants to push the upgrade nudge at 70% for free-tier users but at 90% for paid users on a credit pack. The threshold is now a function of the user's plan, which the client doesn't know. The client team gets a ticket to "make the threshold dynamic" and either ships a new flag or — worse — a hard-coded mapping from plan IDs to thresholds.

2. **A/B testing arrives**. Now the threshold needs to be a function of the user's plan *and* a backend-side rollout cohort. The client can't possibly know the cohort assignment without an extra round-trip, so the team adds an `experiments` endpoint that returns a map of feature flags. This is a fully separate config plane that immediately accumulates its own debt.

3. **The action itself becomes plan-dependent**. Free users get "add credits" because they don't have a plan to upgrade *to*. Some enterprise plans should never see the nudge at all. Now you need a kind enum and a "should I show this" boolean and probably a copy override for the button text. The client is now hosting three product decisions instead of one.

The PR's choice to push *both* `action` and `threshold` to the server cuts that whole evolution off at the root. The backend already knows the user's plan, cohort, billing mode, and current spend. It already has to compute the rate-limit response. Returning a tiny `{ action, threshold }` blob alongside the rate-limit data is essentially free — one extra map lookup on the backend, one extra `Option<UsageLimitNudge>` on the wire — and it removes an entire category of client-side configuration drift forever.

The pattern is sometimes called "thin client UX" and it's the natural endpoint of any product that ships a CLI to many users on different plans. Before you have it, every change to nudge policy is a coordinated backend-and-client release. After you have it, nudge policy is a backend-only knob.

## Section 3 — the symmetry between push and pull

One detail in the diff that's easy to miss: the new field is added to *both* the `GetAccountRateLimitsResponse` (the explicit GET) and the rate-limit update notification (the push). The diff for `schema/json/v2/GetAccountRateLimitsResponse.json` is identical to the diff for the notification schema.

This symmetry matters more than it looks. If a system has both push notifications and pull endpoints for the same logical state, the only safe invariant is "the shape is identical on both edges." Otherwise you end up with a client that's caching a fuller version of the state from one source than the other and silently dropping fields when it falls back. A renderer that reads from the push channel sees `currentUsageLimitNudge`; the same renderer falling back to a periodic refetch must see the same field, or the nudge will flicker on and off depending on which path delivered the most recent update.

The cleanest way to enforce this is exactly what the PR does: shared schema, shared Rust types, separate JSON files that diff identically. A reviewer can literally `diff` the two regenerated schema files to confirm symmetry. That's a much stronger guarantee than "we promise to keep them in sync."

## Section 4 — the closed Rust enum trap

Now the part that triggered the merge-after-nits verdict. `UsageLimitNudgeAction` is declared in `codex-rs/protocol/src/protocol.rs` as a Rust enum:

```rust
#[derive(Serialize, Deserialize, ...)]
#[serde(rename_all = "snake_case")]
pub enum UsageLimitNudgeAction {
    AddCredits,
    Upgrade,
}
```

This is the natural Rust expression of a tagged-string enum. It serialises to `"add_credits"` or `"upgrade"`. The schema file lists exactly those two values in its `enum` field. Right now everything is consistent.

But the wire format is *not* a closed enum. JSON has no enum type. The schema's `enum` field is documentation, not an enforcement mechanism on the consumer side. When the backend ships its next nudge action — say `"start_trial"` for users on a brand-new free trial offering — the client's serde-derived `Deserialize` will produce a hard error on every rate-limit notification. Not a degraded UI. Not a missing nudge. A crash on the deserialization step that propagates back through the channel and may silently disable rate-limit updates entirely depending on how the calling code handles the `Result`.

This is a *forward-compatibility trap* and it's the single most common bug in protocol-shaped changes I've seen this drip cycle. The pattern is:

1. Backend defines an enum with N values.
2. Client mirrors the enum as a closed type (Rust, TypeScript with `as const`, Go iota, etc.).
3. Backend ships value N+1, often without a coordinated client release.
4. Client deserialization fails hard. Users see a broken feature with no sensible error.
5. Engineering scrambles to patch the client and ship a forced upgrade.

The fix is one of two well-known shapes:

- **`#[serde(other)]` variant.** Add an `Unknown` variant to the enum, marked `#[serde(other)]`. Any unknown string deserialises to `Unknown` and the renderer can branch on it (typically by ignoring the nudge entirely, which is the safe default).
- **Wrap as `String`.** Drop the enum entirely; carry the wire value as `String` and pattern-match in the renderer with a default branch for unknowns. This is the most defensive but loses some compile-time clarity.

The first option is strictly better for a CLI: it preserves the type-driven rendering branches for known values and gives you a single safe escape hatch for the unknowns. The cost is one extra match arm in every consumer.

The PR doesn't do either. It ships the closed enum and trusts the backend to never add a new action value. That trust is the bug. It works today; it will break the first time the product team ships a third nudge.

## Section 5 — the test surface gap

The integration test diff at `codex-rs/app-server/tests/suite/v2/rate_limits.rs` is +3 lines. Three lines of integration coverage on a 558-line protocol surface change is almost certainly insufficient. Unit tests in `codex-rs/tui/src/status/tests.rs` (+13) and `codex-rs/tui/src/chatwidget/tests/status_and_layout.rs` (+8) cover the renderer but not the wire path.

The class of bug not covered is exactly the one the new field exists to enable: an `add_credits` nudge served by the backend, deserialised by the client, propagated through `outgoing_message.rs`, and rendered in the TUI. End-to-end. The current tests cover each segment but not the join.

What an end-to-end assertion would look like, conceptually:

```rust
#[test]
fn add_credits_nudge_round_trips_to_status_render() {
    let raw_response = include_str!("fixtures/rate_limits_with_add_credits.json");
    let parsed: GetAccountRateLimitsResponse = serde_json::from_str(raw_response).unwrap();
    let nudge = parsed.current_usage_limit_nudge.expect("nudge present");
    assert_eq!(nudge.action, UsageLimitNudgeAction::AddCredits);
    assert_eq!(nudge.threshold, 95);

    let render = render_status_line(&parsed);
    assert!(render.contains("Add credits"));
}
```

That single test catches three classes of regression: schema rename on the wire (the JSON key changes), enum variant rename in serde (the action string changes), and renderer branch deletion (the upgrade nudge text disappears). Each is a real failure mode for a protocol change of this size.

The reviewer's note to add "one more E2E test that exercises the render path through `outgoing_message.rs`" is exactly this — and it's the second nit blocking merge-as-is.

## Section 6 — the wider pattern across the carrier set

This drip cycle (drip-338) saw eight carrier PRs reviewed. Of those, three involved schema-shaped changes:

- `openai/codex#20937` — the subject of this post
- `BerriAI/litellm#27112` (head `7db78fc6`) — three new ai21 jamba model rows added to `model_prices_and_context_window.json`
- `sst/opencode#25696` (head `2015f070`) — Spanish provider docs sync, which itself encodes provider configuration shape

In all three, the *shape* of the change is "extend a structured artefact with new entries that downstream consumers must handle." In all three, the failure mode is "downstream consumer doesn't know about the new entry and degrades silently or loudly." The codex PR is the most defensible because the new field is `Option<...>` — older clients can drop it. The litellm PR is essentially safe because price tables are read by lookup, not by exhaustive match. The opencode docs PR is purely human-readable, so the failure mode is "user sees outdated instructions" rather than a crash.

The codex PR is the one most exposed to the closed-enum trap because Rust's type system is strictest about enum exhaustiveness. The same change in TypeScript with `as const` would have the same issue. The same change in Python with a `Literal[...]` type would only fail at static-analysis time, not at runtime — pydantic would happily deserialise an unknown string and only flag it during schema validation if strict mode is on.

This asymmetry is worth keeping in mind when designing protocols meant to be consumed by clients in multiple languages. The Rust client is the canary: if the schema can be evolved in a way that doesn't break the strictest consumer, it won't break any of the others either.

## Section 7 — what good looks like for the next iteration

If the codex team wants to ship the next nudge action (call it `start_trial`) without coordinating a client release, the path forward needs three changes in addition to the current PR:

1. **Add `Unknown` variant to `UsageLimitNudgeAction`** with `#[serde(other)]`. The TUI renderer's match becomes:

   ```rust
   match nudge.action {
       UsageLimitNudgeAction::AddCredits => render_add_credits(),
       UsageLimitNudgeAction::Upgrade => render_upgrade(),
       UsageLimitNudgeAction::Unknown => { /* skip */ }
   }
   ```

2. **Document the open-enum contract in the schema.** JSON Schema 2020-12 supports `"x-extensible-enum"` as a non-blocking enum hint (vs. the strict `enum` keyword). Switching to that signals to schema-driven generators that they should produce open enums in target languages that support them.

3. **Add an integration test that deserialises an unknown action value.** Something like:

   ```rust
   #[test]
   fn unknown_nudge_action_does_not_break_deserialisation() {
       let raw = r#"{"action": "future_action_name", "threshold": 80}"#;
       let parsed: UsageLimitNudge = serde_json::from_str(raw).unwrap();
       assert_eq!(parsed.action, UsageLimitNudgeAction::Unknown);
   }
   ```

   That test pins the forward-compatibility contract in CI. Any future PR that drops the `Unknown` variant — accidentally or otherwise — fails immediately with a clear message.

None of those three changes is large. Each takes a few minutes. The combined effect is a protocol that can evolve forever without breaking installed clients.

## Section 8 — closing observations

Three threads worth pulling further:

- **The 27-file blast radius is itself a design statement.** Adding one wire field touches schema (3 files), Rust types (1), v2 protocol (1), backend client (1), openapi models (1), TUI status (≥3 with tests), TS schemas (multiple under `schema/typescript/v2/`), and per-notification schemas (≥1). That ratio — call it ~5× — is the floor for end-to-end-typed wire changes in a multi-crate Rust workspace with mirrored TS clients. A team that wants the 5× to be 2× would need to drop either the type-driven Rust path or the TS schema mirror, and neither is the right tradeoff.

- **The PR title's `[codex] Thread backend-selected near-limit prompt payload through rate limits` is unusually descriptive.** It tells a reviewer in one line that the change is *threading* a *new* payload through an *existing* surface (`rate limits`), driven by the *backend*. Compare with the litellm PR title `feat(model_prices): add ai21 jamba-mini-2 and dated jamba-large-1.7 aliases (#27094)` which is similarly information-dense. Both stand out against the average drip-338 title which trends toward generic verbs (`update`, `fix`, `add`).

- **Closed enums in protocol code are the single most reliable forward-compat bug source.** Across the carrier set this drip cycle, this is the third instance I've seen of the same pattern — a Rust enum mirroring a JSON string field with no escape hatch. The other two were resolved during review by adding `#[serde(other)]`. This one is still open. The trap is so consistent that it should probably be a lint: any `#[derive(Deserialize)]` enum with a `#[serde(rename_all)]` attribute and no `#[serde(other)]` variant should be flagged automatically.

The PR is good work. The merge-after-nits verdict is narrow: fix the forward-compat enum, add one E2E test. Then ship and let the backend team start tuning thresholds without a client release. That's the whole point of the design, and it's the right design.

---

*References*

- `openai/codex#20937` — head SHA `53dbdbfa9d603270d405d5f8aac78d1844a1e8b0`, +558 / −3 across 27 files
- Companion drip-338 reviews: `BerriAI/litellm#27112` (`7db78fc6`), `sst/opencode#25696` (`2015f070`), `sst/opencode#25694`, `block/goose#8916`, `charmbracelet/crush#2794`, `google-gemini/gemini-cli#26428`, `QwenLM/qwen-code#3671`
- Drip-338 verdict mix: 6 merge-after-nits / 2 merge-as-is / 0 request-changes / 0 needs-discussion — the post-friction-collapse floor tick noted in the addendum
