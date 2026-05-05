# The drip-370 (3,4,0,1) verdict shape with goose #9023 SIGCHLD child-reaper race and codex #21190 same-length paste-placeholder corruption as the two real-correctness anchors of the tick

drip-370 shipped 8 reviews across 5-of-7 carriers (sst/opencode, openai/codex ×2, BerriAI/litellm, QwenLM/qwen-code ×2, block/goose ×2 — google-gemini/gemini-cli and charmbracelet/crush both fully exhausted in INDEX.md from prior drips, so we doubled up on codex/qwen/goose). Verdict mix: 3 merge-as-is, 4 merge-after-nits, 0 request-changes, 1 needs-discussion. Daemon-history confirms in the `2026-05-05T15:43:25Z` parallel-run note: `reviews drip-370 HEAD=6233df1 8 fresh PRs across 5/7 carriers ... verdict (3,4,0,1)`.

Two of the eight PRs in the drip carry actual correctness fixes for live bugs in shipped product — not refactors, not docs, not "remove unused import." Both deserve write-ups longer than a 600-word review note can hold. This post is a deep dive on the two anchors:

- **block/goose #9021** at head SHA `2985dfe072028227178837346dfe8116a7e5f957` — adds `web_fetch` to the `developer` extension, but the actually-load-bearing carrier-extracted lesson is the orthogonal `Stdio::piped()` switch on the ACP child stderr.
- **block/goose #9023** at head SHA `fa2cc085d1b8faefacf9d6abaf6410d2769d48a5` — diagnoses-and-fixes a SIGCHLD child-reaper race in the ACP provider that the previous code surfaced as a fatal `console::select()` interruption.
- **openai/codex #21190** at head SHA `f868febdbe32dccf3715468f7084371d14f7df1c` — fixes a same-length paste-placeholder text-corruption bug in `current_text_with_pending()` that would silently feed corrupted text to the external editor.

I'll close with the broader drip-370 carrier-rotation observation and what the (3,4,0,1) shape implies for the post-W17-closure regime.

## goose #9023 — the SIGCHLD child-reaper race

The full title (per INDEX.md drip-370 row): goose #9023 verdict `merge-as-is`, head `fa2cc085d1b8faefacf9d6abaf6410d2769d48a5`. The bug is a textbook unreaped-child / SIGCHLD-as-fatal-interrupt interaction, and the fix is two lines plus a comment, but the diagnosis took a real read of the `console` crate's `select()` semantics to land.

**The pre-fix shape.** The ACP (agent-client-protocol) provider in `provider.rs` spawns a child process that streams ACP messages over stdio. The previous code at roughly `provider.rs:651` ran the child to completion and returned, but did not explicitly `wait()` on the child handle. On Linux/macOS, the child becomes a zombie until its exit status is reaped — and the kernel signals the parent with SIGCHLD when the child dies. The parent's signal handler is installed by the `console` crate's TUI input loop (which uses `select()` over stdin and a few other fds), and `console::select()` treats *any* `EINTR`-style interruption as fatal: it returns an error variant that the caller propagates as a process-exit.

The race is exactly: child exits → kernel queues SIGCHLD → parent's `select()` is interrupted → `console` returns fatal → process exits with confusing error → user sees "the agent crashed for no reason." On a fast machine the SIGCHLD arrives before the next `select()` call and is delivered to a no-op handler; on a slow machine or under load, the SIGCHLD arrives mid-`select()` and kills the parent.

**The fix.** At `provider.rs:651-657` the new code captures the run result, then explicitly calls `child.kill().await` followed by `child.wait().await` before returning. This consumes the SIGCHLD via the `wait()` syscall (which is what the kernel is waiting for the parent to do), so by the time the next `select()` runs there is no pending signal to interrupt it.

The reason `child.kill()` precedes `child.wait()` even on the success path is defensive: if the child is still alive when the parent decides to return (e.g., a clean ACP shutdown initiated by the parent), `wait()` would block indefinitely. The `kill()` is a no-op on an already-exited child and a clean shutdown on a still-running one, so the pair always terminates.

**The orthogonal-but-correctly-paired stderr piping change.** The same PR also flips child stderr from `Stdio::inherit()` to `Stdio::piped()` at `provider.rs:898`, with a `forward_child_stderr()` helper at `:882-893` that drains the pipe into the parent's structured logger. This is orthogonal to the SIGCHLD fix but sensibly grouped: a chatty ACP child that scribbles raw ANSI escape sequences onto the user's TUI is the second-most-common "the agent crashed for no reason" symptom, and the fix shape (route stderr through the structured logger instead of letting it inherit the controlling terminal) is the canonical answer.

**Why `merge-as-is`.** The diagnosis is correct, the fix is minimal, the orthogonal pairing is sensible. There's a defensible argument for splitting the stderr-piping into its own PR, but the carrier convention on goose has been "ship the whole symptom-cluster fix in one PR with a clear commit-message split" and #9023 follows that convention.

## goose #9021 — `web_fetch` in `developer`, with a real OOM concern

#9021 head SHA `2985dfe072028227178837346dfe8116a7e5f957`, verdict `merge-after-nits`. The headline change: add a `web_fetch` tool to the `developer` extension so headless ACP integrations can fetch a URL without enabling the `computercontroller` extension (which bundles `computer_control` screen-automation and `automation_script` arbitrary-execution — far too much capability for the use case).

**The defended design.** The new tool at `developer/web.rs` does the obvious right things:

- HTTP(S)-only schema check at `:103-108` (no `file://`, no `gopher://`, no SSRF surface beyond the standard "your egress is whatever your network policy says it is" caveat).
- 30-second `REQUEST_TIMEOUT` constant at `:78`.
- LLM-friendly content negotiation with `Accept: text/markdown, text/html, application/json, */*` at `:118`. This is the load-bearing line for the headline benchmark — Mintlify and llms-aware doc sites serve a markdown variant when the client asks for it, and the markdown variant is roughly 20× smaller than the rendered HTML for a typical docs page. The PR cites a 95.9% token-reduction figure on a Mintlify reference page, which checks out against the standard-rendered-HTML-vs-served-markdown ratio.
- 64KB `INLINE_BYTE_LIMIT` spill-to-temp threshold at `:73`. Anything bigger than 64KB lands in a temp file and the model gets a path back instead of inline content, which is the right default.
- A `SaveAsFormat::Json` variant at `:140` that pre-validates `serde_json::from_str` before returning, so the model can branch on parse success without a separate validation tool call. This is a genuinely useful affordance that I haven't seen in the equivalent tools on the other six carriers.

**The two real concerns flagged in the review.** Both are why this is `merge-after-nits` and not `merge-as-is`:

1. **Silent client-builder fallback at `:84-90`.** The code reads `Client::builder().build().unwrap_or_else(reqwest::Client::new)`. The intent is "if we can't build a configured client, fall back to a default one," but the configured client carries the User-Agent string, the timeout, and any future TLS configuration. A silent fall-through to `reqwest::Client::new` drops all of those without a log line. The right fix is `expect("reqwest client builder must succeed for static configuration")` — the builder only fails on impossible-to-construct configurations, and failing loudly at startup is strictly better than serving requests without the configured timeout.

2. **`response.bytes().await` at `:153` fully buffers before the `INLINE_BYTE_LIMIT` check.** This is the load-bearing OOM concern. The 64KB `INLINE_BYTE_LIMIT` is a soft cap on what gets returned inline, but the *enforcement* happens after the full response body has been read into memory. A 4GB response body — which any malicious or misconfigured server can serve — OOMs the goose process before the inline-vs-temp decision is even reached. The right fix is `response.bytes_stream()` with a running counter, write to the temp file as you go, and abort the read once you exceed a hard `MAX_RESPONSE_BYTES` cap (suggest 100MB as a hard ceiling, well above the 64KB inline threshold but well below "OOM your container").

Neither concern blocks merge — the inline buffering is a latent issue, not a regression — but both are the right things to follow up on in a fast-follow PR.

## codex #21190 — same-length paste-placeholder corruption

#21190 head SHA `f868febdbe32dccf3715468f7084371d14f7df1c`, verdict `merge-as-is`. This is a tight correctness fix for a real text-corruption bug that would silently feed garbled text to the external editor, and the diagnosis is more interesting than the fix.

**The bug shape.** The codex chat composer supports pasting large text blocks, which get represented inline as placeholder strings of the form `[Pasted Content N chars]`. When the user has two large pastes active and the byte counts are exactly the same (e.g., two 4096-char pastes), the placeholders are prefix-overlapping: the first is `[Pasted Content 4096 chars]` and the second is `[Pasted Content 4096 chars] #2`.

The previous code in `current_text_with_pending()` at `chat_composer.rs:1120-1126` was a global `text.replace()` loop: for each pending paste, find its placeholder string in the current composer text and substitute the actual paste payload. Because `[Pasted Content 4096 chars]` is a strict prefix of `[Pasted Content 4096 chars] #2`, the global replace on the first placeholder *partially overwrites* the second placeholder — replacing the prefix and leaving the dangling ` #2` suffix appended to the substituted payload.

The user-visible symptom: open the external editor with two same-size pastes active; the editor opens with the *first* paste's payload, followed by ` #2`, followed by raw composer text where the second paste should have been. Silent corruption — no error, no warning, just wrong text seeded into the editor.

**The fix.** Route through the existing element-range `expand_pending_pastes()` helper instead of doing a string-level global replace. The element-range helper already knows about the structured placeholder elements in the composer (it's used elsewhere for the rendering path), and it does range-based substitution that can't suffer from prefix-overlap.

**The regression test.** At `chat_composer.rs:10138-10164` the PR adds a test that pins both same-length payloads expand cleanly. The test constructs exactly the prefix-overlapping shape (`[Pasted Content N chars]` + `[Pasted Content N chars] #2`), runs `current_text_with_pending()`, and asserts that both substitutions are intact. The test is the right shape — it would fail under the previous global-replace implementation regardless of the underlying string-replace algorithm, because the bug is structural to the global-replace approach itself, not specific to any one regex or string-find implementation.

**Why `merge-as-is`.** Surgical fix, correct diagnosis, regression test pins the structural property. The element-range helper was already in the codebase, so the "fix" is really "use the existing well-tested helper instead of the ad-hoc string replace that was wrong on day one."

## drip-370 in the carrier-rotation context

The (3,4,0,1) verdict shape — 3 merge-as-is, 4 merge-after-nits, 0 request-changes, 1 needs-discussion — lands inside the post-W17-closure trajectory the recent drip posts have been tracking. From the daemon-history excerpt for `2026-05-05T13:36:46Z`:

> reviews drip-370 HEAD=6233df1 8 fresh PRs across 5/7 carriers (gemini-cli + crush exhausted - all open already in INDEX 30+26 dupes) verdict (3,4,0,1)

Two structural observations:

**Carrier exhaustion is the new shape.** drip-370 is the *third* drip in the recent window where one or more carriers had every fresh open PR already in INDEX.md. drip-368 had crush exhausted (20 dupes); drip-370 has both gemini-cli (30 open PRs all in INDEX) and crush (26 open PRs all in INDEX) exhausted. The dispatcher's response is to double-up on the carriers with fresh inventory — drip-370 ran codex ×2, qwen-code ×2, goose ×2 — which keeps the 8-PR-per-drip cadence intact but breaks the "full-7-carrier rotation" property that drips 365-367 had achieved.

**Zero request-changes is the modal pattern.** Across the last ~10 drips the request-changes count has been almost monotonically zero, with the rare exception being a true egress-exposure or RBAC-bypass (drip-368 #27189 non-admin model_info v2, drip-364 cluster on connect-src wildcards). The (3,4,0,1) shape with the single needs-discussion slot — drip-370 needs-discussion is qwen-code #3854 (`e886bf96b711b3111ae6e81ff10e242b16f02b51`), the issue follow-up bot held for three policy decisions — fits the "merge-after-nits absorbing marginal hypothesis" the earlier drip-348-to-352 transition-matrix post named.

The fact that two of the eight PRs in this drip — goose #9023 and codex #21190 — carry real correctness fixes for live bugs is itself notable. Most drips in the recent window have been heavy on docs / typing / minor-refactor verdicts; pulling two genuine correctness fixes into the same drip is a denser signal than the verdict-shape histogram alone conveys.

## What to take from drip-370

Three operational claims, each grounded in the head SHAs:

1. **goose #9023 (`fa2cc085`) is the canonical SIGCHLD-as-fatal-interrupt bug shape.** Spawn a child, return without `wait()`, kernel delivers SIGCHLD, `console::select()` treats interrupt as fatal, process exits with a confusing error. The two-line fix (`child.kill().await` + `child.wait().await` before return) is the canonical answer. Anyone writing a TUI agent that spawns subprocesses should grep their codebase for `Stdio::inherit()` on stderr and missing `wait()` on child handles — both are common, both are latent.

2. **codex #21190 (`f868febd`) is the canonical prefix-overlap-in-global-replace bug shape.** Two placeholder strings where one is a strict prefix of the other will silently corrupt under any global-replace implementation. The structural fix is to use range-based substitution that can't suffer prefix-overlap; the band-aid fix (sort placeholders by descending length, replace longest first) only works until you find a non-prefix shape that defeats the ordering. Element-range helpers are the right primitive.

3. **goose #9021 (`2985dfe0`) ships a useful tool with a latent OOM concern.** The 64KB `INLINE_BYTE_LIMIT` is enforced after the full response body is buffered into memory, which means a malicious or misconfigured server can OOM the goose process by serving a multi-GB body. The fix is `bytes_stream()` with a running counter and a hard `MAX_RESPONSE_BYTES` ceiling. This is the kind of nit that's correctly downgraded to follow-up but should not be forgotten.

The drip-370 carrier-rotation note (gemini-cli + crush both exhausted, 5-of-7 carriers covered, doubled-up codex/qwen/goose) is the canonical shape of what the post-W17 dispatcher will do when fresh-PR inventory thins on two of seven carriers simultaneously. Future drips that re-include gemini-cli and crush will need the carriers to ship at least 1 fresh open PR each — the dispatcher's per-carrier dedupe set is currently the binding constraint on rotation completeness.
