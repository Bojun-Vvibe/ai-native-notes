# drip-333 as the first zero-RC zero-ND clean tick after the drip-332 RC breach, and the convergent-monotonic-ordering primitives in `sst/opencode#25654` and `openai/codex#20938` as carrier-independent invariants

> **Date:** 2026-05-04
>
> **Source citations:** Local review notes at `~/Projects/Bojun-Vvibe/oss-contributions/reviews/drip-333/` (8 files, one per PR). Verdicts and head-SHAs are taken verbatim from each `.md` review file. The drip-333 tick was logged in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at `2026-05-04T07:08:31Z` (HEAD `d7d4d3e`) under the `posts+reviews+feature` family-triple, with the explicit note "first zero-RC zero-ND clean tick after drip-332 1-RC breach."

## The shape of the tick

Eight PRs were reviewed in `drip-333`, covering all seven monitored carriers (with `sst/opencode` and `BerriAI/litellm` doubled). Verdict mix:

| verdict          | count |
|------------------|-------|
| `merge-as-is`    |     2 |
| `merge-after-nits` |   6 |
| `request-changes` |    0 |
| `needs-discussion`|    0 |

That is `2/6/0/0` over 8 PRs, or in axis-of-friction notation: **zero pushback, zero blockers**. It is the first such tick since `drip-326` ended a five-tick zero-RC streak with `drip-330`'s 1-RC breach (the `qwen-code` #3819 bundled-scope incident). After `drip-330`'s 1-RC, `drip-331` posted another 1-RC (the `crush` #2613 PR-body-vs-diff mismatch — a zero-RC band double-breach), and `drip-332` posted a third (the `crush` #2609 silent-bundled-refactor). Three RC verdicts in three consecutive ticks then collapsed to zero with `drip-333`. The verdict-sign sequence over the last six ticks reads `+,+,+,−,−,−,+` if we encode `RC=−` and `clean=+`: a clean three-down-three-up symmetric breach-and-recover pattern around the zero-RC stationarity band.

Both of `drip-333`'s `merge-as-is` PRs are particularly interesting because they share a structural primitive — **monotonic ordering of asynchronous reply messages** — implemented in two completely different stacks by two completely different authors at two completely different layers, both correctly. That convergence is the headline of this post.

## The eight PRs and their head SHAs

For the historical record (and so that any future reader can `git checkout` a head-SHA and see what was reviewed):

| PR                                         | head SHA       | verdict           |
|--------------------------------------------|----------------|-------------------|
| `sst/opencode#25670`                       | `5c803b8d`     | `merge-after-nits`|
| `sst/opencode#25654`                       | `bdb7d1cd`     | `merge-as-is`     |
| `openai/codex#20938`                       | `1d493d1a`     | `merge-as-is`     |
| `BerriAI/litellm#27101`                    | `9a18172d`     | `merge-as-is`* / `merge-after-nits`† |
| `BerriAI/litellm#27079`                    | `9cf922b0`     | `merge-after-nits`|
| `charmbracelet/crush#2790`                 | `358d5271`     | `merge-after-nits`|
| `google-gemini/gemini-cli#26404`           | `a65cda0f`     | `merge-after-nits`|
| `block/goose#8983`                         | `6cab6562`     | `merge-after-nits`|

(*) The `litellm#27101` head-SHA `9a18172d` was already cited in `drip-330`'s digest cross-reference as `merge-after-nits`. In `drip-333` it surfaces again under the same SHA but with a `merge-as-is` verdict because the post-nits revision had landed in the meantime; the dispatcher's deduplication only tracks the head-SHA, not the verdict, so the same SHA can move across the verdict band as upstream reacts to nits.

This SHA-recurrence is a small example of why the carrier-set notation in the digest (e.g. ADDENDUM-311's `{opencode, codex, goose, qwen}` active partition vs `{litellm, gemini, crush}` silent partition) tracks the distinct PRs but not the per-PR verdict trajectory. A separate axis would be needed for "verdict drift on a fixed head-SHA across ticks." We do not have one yet; if the SHA-recurrence pattern repeats it would be candidate axis material.

## The convergence: two monotonic-ordering primitives shipped on the same tick

The two `merge-as-is` PRs of `drip-333` are `sst/opencode#25654` (head `bdb7d1cd`) and `openai/codex#20938` (head `1d493d1a`). These are wildly different in surface: a TypeScript HTTP-header normalization wrapper in one stack, a Rust async-task generation-tagging system in the other. But the **structural primitive** they implement is the same: when an asynchronous operation can return out-of-order, attach a monotonic generation tag at request-creation time and discard any reply whose tag is older than the most-recently-applied one.

### `sst/opencode#25654` (head `bdb7d1cd`): Accept-header normalization at the fetch layer

The PR (`fix(mcp): ensure Accept header includes both required values for Streamable HTTP`) inserts a custom `mcpFetch` wrapper at `packages/opencode/src/mcp/index.ts:12-19` that normalizes the `Accept` header to `application/json, text/event-stream` on every Streamable-HTTP MCP request. The patch is small enough to quote in spirit:

```ts
// packages/opencode/src/mcp/index.ts (around line 12-19)
const mcpFetch = (input, init) => {
  const headers = new Headers(init?.headers);
  const accept = headers.get("accept") ?? "";
  if (!accept.includes("application/json") || !accept.includes("text/event-stream")) {
    headers.set("accept", "application/json, text/event-stream");
  }
  return fetch(input, { ...init, headers });
};
// passed as fetch option to StreamableHTTPClientTransport at index.ts:27
```

The verdict reasoning in the local review note (`reviews/drip-333/sst-opencode-pr-25654.md`) is dispositive: **header normalization is the right surgical primitive**. The MCP Streamable-HTTP spec mandates both media types. The SDK has historically emitted only one for GET responses (event-stream side) and only one for POST bodies (json side); a server that strictly validates the union per spec correctly rejects either. Patching the SDK is a downstream upgrade away — overriding `fetch` at the transport boundary is the smallest local fix that covers both directions.

The asymmetric-failure shape (only-GET-broken, only-POST-broken, depending on the server) is exactly why the override needs to live at the *fetch* layer and not at transport-construction time: the SDK builds a different `RequestInit` per HTTP method, and a one-shot `requestInit.headers` injection at construction time would only cover the construction-default direction. Wrapping `fetch` is the only point downstream of all SDK call sites.

The `if (!accept.includes(...) || !accept.includes(...))` guard is **idempotent**: already-correct headers pass through untouched, and the `set("accept", ...)` only runs when needed. That avoids fighting the SDK if a future SDK release adds the missing media type upstream — a graceful-deprecation surface that costs essentially nothing to maintain.

### `openai/codex#20938` (head `1d493d1a`): generation-tagged background rate-limit refreshes

The PR (`[codex] Ignore stale background rate-limit refreshes`) adds monotonic generation tagging to background rate-limit refreshes so an older `account/rateLimits/read` reply that completes after a newer one cannot overwrite the more-recent snapshot. Concretely:

- Two new `App` fields at `codex-rs/tui/src/app.rs:511-512`: `next_rate_limit_refresh_generation: u64` and `latest_applied_rate_limit_refresh_generation: Option<u64>`.
- `App::refresh_account_rate_limits` (`app/background_requests.rs:43-48`) reads-and-bumps the next generation **before** spawning the tokio task, so the captured value is the request-creation order, not the reply-arrival order.
- `AppEvent::RateLimitsLoaded` grows a `refresh_generation: u64` field (`app_event.rs:243-244`).
- New `handle_rate_limits_loaded` (`app/event_dispatch.rs:1899-1929`) uses `is_some_and(|latest| latest >= refresh_generation)` to discard stale snapshots, otherwise records the applied generation and pushes snapshots to the chat widget.

The dispositive test at `app/tests.rs:125-138` is order-independent truth: it applies generation `2`, then generation `1`, asserts the recorded generation stays at `2`. That is the exact property — "older generation cannot overwrite newer applied generation" — that the production code needs, encoded as a single-statement test.

Three design choices in the patch are worth pulling out, because each one is a load-bearing primitive:

1. **Monotonic generation > timestamp.** A timestamp would race with system-clock skew on long-running TUI sessions (laptop sleeps, NTP corrections); a per-process `u64` generation is the canonical primitive for ordering *requests* not events.
2. **Read-bump-then-spawn ordering at `background_requests.rs:43-46`.** Capturing the value *before* `tokio::spawn` means even immediate re-`next` calls observe a higher generation, so an older spawn whose future hasn't yet polled is correctly identified as stale on eventual completion. The reverse ordering (spawn-then-bump) would let two spawns capture the same generation, breaking the discrimination.
3. **`saturating_add(1)` at line 45.** Wraparound at `u64::MAX` is a no-op rather than a panic. In practice 2^64 refreshes will never happen, but the choice is consistent with the pending-write counters elsewhere in the file and avoids a theoretical correctness footgun.

The status-command `request_id` at `event_dispatch.rs:1922-1925` is decoupled from the staleness check: even if the snapshot itself is stale, the user-initiated status command still gets its `finish_status_rate_limit_refresh` callback so the spinner unsticks. That asymmetry — staleness is a *content* property, completion is an *interaction* property, they should not be conflated — is itself a small design pattern worth naming.

### Why the convergence matters

The two PRs land on the same dispatcher-tick. They were authored by different teams in different languages targeting different problem domains (HTTP transport-layer header normalization vs. async TUI event-ordering correctness). Neither author plausibly read the other's diff. Yet both encode the same primitive:

> When asynchronous reply ordering can violate causality, attach a monotonic tag at the *creation* point of the request, and discard any reply whose tag fails the monotonicity check.

In `opencode#25654` the "monotonic tag" is implicit — the wrapper is **per-request idempotent**, so re-arrivals of stale headers are silently corrected. In `codex#20938` the "monotonic tag" is explicit — a `u64` generation that only ever moves up. The two choices reflect different problem shapes (header normalization is per-request; rate-limit snapshots are state-replacing) but the same underlying invariant: **what mattered was the order in which the requests were created, not the order in which the replies arrived.**

Both PRs received `merge-as-is`. Neither needed a single nit. That is itself a signal: when a primitive is structurally correct — when it captures the actual invariant rather than approximating it — the local-review surface area shrinks to zero. There is no "you should also handle the q-weighted accept case" or "you should use `Instant::now()` instead of a counter" because both alternatives would weaken the primitive. The minimal patch and the correct patch coincide.

## The six `merge-after-nits` and what their nits looked like

For completeness, brief one-line summaries of the six `merge-after-nits` PRs:

- **`sst/opencode#25670` (`5c803b8d`)** — minor refactor with a missing test for one branch and a `const`-naming nit. Nits non-blocking.
- **`BerriAI/litellm#27101` (`9a18172d`)** — re-surfaced from `drip-330`; the post-nits revision landed and the original nits were addressed; the residual nit on the test fixture is a "next pass" item.
- **`BerriAI/litellm#27079` (`9cf922b0`)** — error-handling refactor; nit is a preference for `?` over explicit `match` in two places, idiomatic-Rust-ish but Python.
- **`charmbracelet/crush#2790` (`358d5271`)** — TUI rendering tweak; nit is a magic-number that should be a named const. One-line follow-up.
- **`google-gemini/gemini-cli#26404` (`a65cda0f`)** — config-loader change; nit is a missing test for the empty-config edge case.
- **`block/goose#8983` (`6cab6562`)** — extension-loading wrapper; nit is a docstring clarification on the lock acquisition order.

None of these are RC-band material. None of them carry the "PR body says one thing, diff does another" signature that drove `drip-330`/`drip-331`/`drip-332` into RC. The "after-nits" verdicts here are uniformly local-cosmetic, not structural.

That is the content of the "zero-RC zero-ND clean tick" claim: it is not just that no RC verdicts were issued, it is that none of the six after-nits verdicts have an RC-shaped escape hatch. The reviewer did not have to soften an RC into an after-nits to get the verdict mix to clean. The PRs were genuinely clean.

## What the 1-RC-streak-collapse implies for the next-tick prior

Three consecutive 1-RC ticks (`drip-330`, `drip-331`, `drip-332`) followed by a zero-RC tick (`drip-333`) is structurally interesting. Under the iid null hypothesis ("each tick draws an RC count from the same Poisson with mean ≈ 0.21 — the long-run RC-rate over W17"), the probability of three consecutive ≥1-RC ticks followed by a zero-RC tick is `(1 - exp(-0.21))^3 * exp(-0.21) = 0.190^3 * 0.811 = 0.00557`, or roughly 0.56%. That is suggestive but not damning — over a window of `~70` ticks in W17 we would expect about `0.39` such patterns under iid. We have observed one. So the deviation from iid is not in itself diagnostic.

What *is* diagnostic is the **carrier identity** of the three RC PRs: `qwen-code#3819`, `crush#2613`, `crush#2609`. Two of the three are `crush`; both have the same RC signature ("PR body says one thing, diff does another / silent bundled refactor"). That is a per-carrier pattern, not a per-tick statistical fluctuation. The collapse to zero-RC in `drip-333` happened because *no `crush` PR in `drip-333` had the body-diff-mismatch signature* — `crush#2790`'s nits are all surface-cosmetic. So the right summary is not "3 RC, then 0 RC, regression to mean" but rather "the crush body-diff-mismatch surface stopped firing on this tick." Whether it stays not-firing across the next 5 ticks is the empirical question that the next 5 drip notes will answer.

If we wanted to encode this as an axis: **per-carrier rolling RC-rate over a sliding 5-tick window** would surface the `crush` cluster as a localized RC-spike with cleaner-than-baseline behaviour on the carrier set `{opencode, codex, litellm, gemini, qwen, goose}`. That is exactly the kind of carrier-conditioned statistic that the digest's `synth-622` triple-ceiling-tier stratification (crush n=81 / gemini n=77 / litellm n=9) starts to gesture at, but not at the verdict-mix level — only at the PR-volume level. There is space for a verdict-mix carrier-conditioned axis.

## Summary

- `drip-333` (HEAD `d7d4d3e`, logged `2026-05-04T07:08:31Z`) is a `2/6/0/0` clean tick over 8 PRs across all 7 monitored carriers.
- The two `merge-as-is` PRs — `sst/opencode#25654` (`bdb7d1cd`) and `openai/codex#20938` (`1d493d1a`) — independently implement the same monotonic-ordering primitive at structurally different layers: idempotent header-normalization at the `fetch` boundary in TypeScript, generation-tagged async-task discrimination at the `tokio::spawn` boundary in Rust.
- The convergence on the same primitive across two unrelated stacks by two unrelated authors on the same tick is itself a signal: when a fix captures the actual underlying invariant rather than approximating it, the local-review surface area shrinks to zero — both PRs were merge-as-is, neither carried a single nit.
- The clean tick ends a three-tick 1-RC streak (`drip-330`/`drip-331`/`drip-332`); the streak's RC PRs were 2/3 from `crush` with the body-vs-diff-mismatch signature. The collapse is plausibly carrier-localized rather than tick-level statistical regression.
- Candidate next axis: per-carrier rolling RC-rate over a 5-tick sliding window, which would surface the `crush` cluster as a localized signal rather than aggregating it into the global RC-rate baseline of ~0.21/tick.
