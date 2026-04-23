---
title: "Architecture deep-dive: openai/codex"
date: 2026-04-23
tags: [architecture, deep-dive, codex]
est_reading_time: 16 min
---

## TL;DR

- `openai/codex` is a Rust workspace shaped like a small operating system for an agent: ~150 crates, with the agent loop, sandbox, MCP client, rollout store, and TUI all isolated behind crate boundaries. The `core` crate is the engine; everything else either feeds it or rides on top of it.
- The single most interesting design decision is the **submission/event split**: the public API of a thread is `submit(Op) -> id` and `next_event() -> Event`. There is no synchronous "send a message and get a reply" call anywhere. The agent loop runs in its own task and emits a stream of typed events. Every UI (TUI, app-server, exec) is just a different consumer of that stream.
- What I'd steal: their `RwLock<()>` "parallel-vs-serial" gate for tool execution. One line of code lets parallel-eligible MCP tools fan out concurrently while a `shell` call grabs an exclusive write lock. Trivially correct, no bespoke scheduler.

## Repo at a glance

- **Language**: Rust (~95%) plus a thin TypeScript `codex-cli` shim and a small `sdk/` directory.
- **Size**: 406 MB checkout. The Rust workspace alone (`codex-rs/`) holds ~150 member crates. `core/src/session/turn.rs` is 2,285 lines; `core/src/session/mod.rs` is 3,308; `core/src/client.rs` is 2,052. The non-test core is around 40k LOC.
- **Top-level layout** (`codex-rs/`): `core/` (the engine), `tools/` (tool registry + dispatcher), `tui/` (Ratatui UI), `exec/` (headless one-shot mode), `app-server/` (long-running JSON-RPC backend), `mcp-server/` and `mcp/` (MCP client + server halves), `sandboxing/`, `apply-patch/`, `rollout/` (transcript store on disk), `protocol/` (the wire types every crate depends on).
- **Build**: Cargo workspace plus a Bazel overlay (`BUILD.bazel`, `MODULE.bazel`) and a `flake.nix`. `justfile` for common tasks.
- **Tests**: built-in `#[test]`, `tokio::test`, plus snapshot tests via `insta`. Many `*_tests.rs` files sit next to their implementation (`session/tests.rs` is 7,104 lines on its own).

All references below are to commit `17ae906`.

## The agent loop

The thing to internalize about codex's architecture is that **the agent loop is fully decoupled from any UI**. A `CodexThread` ([`core/src/codex_thread.rs`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/codex_thread.rs)) exposes only two interesting methods:

```rust
pub async fn submit(&self, op: Op) -> CodexResult<String> { ... }
pub async fn next_event(&self) -> CodexResult<Event> { ... }
```

`Op` is a sum type — `UserInput`, `Compact`, `Interrupt`, etc. `Event` is the matching output stream — `AgentMessageContentDelta`, `OutputItemDone`, `TurnStarted`, `Warning`, `HookStarted`, and dozens more. The TUI, the headless `exec` binary, and the JSON-RPC `app-server` all consume the exact same event stream. This is the OpenAI Realtime model applied internally: every state change is an event on a queue, never a return value.

The loop itself lives in `core/src/session/turn.rs::run_turn`. The doc comment is the single most useful paragraph in the whole repo:

```rust
// codex-rs/core/src/session/turn.rs:120-133
/// Takes a user message as input and runs a loop where, at each sampling request, the model
/// replies with either:
///
/// - requested function calls
/// - an assistant message
///
/// While it is possible for the model to return multiple of these items in a
/// single sampling request, in practice, we generally one item per sampling request:
///
/// - If the model requests a function call, we execute it and send the output
///   back to the model in the next sampling request.
/// - If the model sends only an assistant message, we record it in the
///   conversation history and consider the turn complete.
```

That's the canonical "ReAct-shaped" loop, but the implementation is not a simple `while`. `run_turn` does a lot of orchestration *before* the loop starts:

1. **Pre-sampling compaction**. If recorded token usage is already over the model's `auto_compact_token_limit`, run an inline compact task before sending anything to the model ([`turn.rs:156-165`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/session/turn.rs#L156-L165)).
2. **Skill resolution**. The user message is parsed for explicit `@skill` and `@app` mentions; matching skills are loaded, MCP dependencies are auto-installed if needed, and warnings are surfaced as `EventMsg::Warning` events.
3. **Plugin / connector resolution**. Same pattern, for plugins and MCP "connectors."
4. **`UserPromptSubmit` hooks**. A user-defined hook chain runs and can `should_stop` to abort the turn before the model is even called. Returned `additional_contexts` are injected into history.

Only then does the inner loop begin ([`turn.rs:379-505`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/session/turn.rs#L379-L505)):

```rust
// codex-rs/core/src/session/turn.rs:379
loop {
    if run_pending_session_start_hooks(&sess, &turn_context).await {
        break;
    }
    // ... drain pending input (messages the user sent while the model was running) ...
    let sampling_request_input: Vec<ResponseItem> = {
        sess.clone_history()
            .await
            .for_prompt(&turn_context.model_info.input_modalities)
    };

    match run_sampling_request( /* ... */ ).await {
        Ok(SamplingRequestResult { needs_follow_up, last_agent_message, .. }) => {
            // auto-compact if over token limit
            if total_usage_tokens >= auto_compact_limit && needs_follow_up {
                run_auto_compact(...).await?;
                continue;
            }
            if !needs_follow_up { break; }  // turn complete
        }
        Err(e) => { /* retry, fall back transport, or fail */ }
    }
}
```

Two things stand out. First, `pending_input` — the user can keep typing while the model is mid-response, and those messages are queued and drained at the *next* iteration of the loop, not interleaved into the current sampling request. Second, the loop's exit condition is the inverse of "model produced no tool calls": `needs_follow_up = model_needs_follow_up || has_pending_input`. So even if the model is "done," a queued user message will keep the turn alive.

`run_sampling_request` ([`turn.rs:1015`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/session/turn.rs#L1015)) is where the tool router gets built per-iteration:

```rust
// codex-rs/core/src/session/turn.rs:1026-1043
let router = built_tools(
    sess.as_ref(),
    turn_context.as_ref(),
    &input,
    explicitly_enabled_connectors,
    skills_outcome,
    &cancellation_token,
).await?;

let base_instructions = sess.get_base_instructions().await;

let tool_runtime = ToolCallRuntime::new(
    Arc::clone(&router),
    Arc::clone(&sess),
    Arc::clone(&turn_context),
    Arc::clone(&turn_diff_tracker),
);
```

Note that the router is rebuilt every sampling request — not every turn. That's because what tools are visible can change mid-turn: a skill might get loaded, an MCP server might come online, the user might enable a connector via a queued message. It's expensive (it walks every MCP server, every plugin, every connector) but it's correct, and codex is a tool that's already paying the latency tax of an LLM call so a few extra ms in the router is invisible.

The rest of `run_sampling_request` is a retry loop wrapping `try_run_sampling_request`, with a clever escape hatch:

```rust
// codex-rs/core/src/session/turn.rs:1104-1120
let max_retries = turn_context.provider.info().stream_max_retries();
if retries >= max_retries
    && client_session.try_switch_fallback_transport(
        &turn_context.session_telemetry,
        &turn_context.model_info,
    )
{
    sess.send_event(
        &turn_context,
        EventMsg::Warning(WarningEvent {
            message: format!("Falling back from WebSockets to HTTPS transport. {err:#}"),
        }),
    )
    .await;
    retries = 0;
    continue;
}
```

If the WebSocket transport keeps failing, codex transparently falls back to HTTPS streaming and resets the retry counter. The user gets a single `Warning` event explaining what happened. This is the kind of operational detail you only build after an outage taught you to.

Back inside the streaming response, `try_run_sampling_request` walks `ResponseEvent::*` deltas. Tool calls trigger `tool_runtime.handle_tool_call()`; the result becomes a `ResponseInputItem::FunctionCallOutput` that's recorded in history. When the streamed response signals `Completed`, control returns to the outer loop, which decides whether to iterate again or exit.

## Tool calling implementation

Tools live in two places: the static set in `core/src/tools/` plus the auxiliary `codex-rs/tools/` crate that defines schema types like `ToolSpec` and `ToolName`. The runtime three-layer stack is `ToolRouter` → `ToolCallRuntime` → `ToolRegistry::dispatch_any`.

A `ToolCall` is built from a streamed `ResponseItem::FunctionCall` ([`tools/router.rs:178-202`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/tools/router.rs#L178-L202)):

```rust
// codex-rs/core/src/tools/router.rs:185-202
let tool_name = ToolName::new(namespace, name);
if let Some(tool_info) = session.resolve_mcp_tool_info(&tool_name).await {
    Ok(Some(ToolCall {
        tool_name: tool_info.canonical_tool_name(),
        call_id,
        payload: ToolPayload::Mcp {
            server: tool_info.server_name,
            tool: tool_info.tool.name.to_string(),
            raw_arguments: arguments,
        },
    }))
} else {
    Ok(Some(ToolCall {
        tool_name,
        call_id,
        payload: ToolPayload::Function { arguments },
    }))
}
```

The interesting part is the dispatcher in `core/src/tools/parallel.rs`. Codex needs to support both *parallel* tools (e.g. read-only MCP queries that can run concurrently) and *serial* tools (e.g. `shell` — you can't run two `git commit`s at the same time). Their solution is a single `RwLock<()>`:

```rust
// codex-rs/core/src/tools/parallel.rs:115-133
res = async {
    let _guard = if supports_parallel {
        Either::Left(lock.read().await)
    } else {
        Either::Right(lock.write().await)
    };

    router
        .dispatch_tool_call_with_code_mode_result(
            session,
            turn,
            invocation_cancellation_token,
            tracker,
            call.clone(),
            source,
        )
        .instrument(dispatch_span.clone())
        .await
} => res,
```

That's it. Parallel-eligible tools take a *read* guard; serial tools take a *write* guard. Tokio's `RwLock` does the rest. Many readers, one writer, FIFO-ish fairness — no scheduler, no priority queue, no per-tool semaphore. A serial `shell` call will wait for in-flight parallel reads to drain, then block any parallel reads from starting until it finishes. I can't think of a simpler way to express "these tools can interleave but those cannot."

Each call also runs inside `tokio::spawn` wrapped in `AbortOnDropHandle`, so a cancelled turn drops the handle and the tool task is killed. The outer `tokio::select!` races the cancellation token against the dispatch future and synthesizes an `aborted_response` ([`parallel.rs:174-195`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/tools/parallel.rs#L174-L195)) — including a special `Wall time: X.X seconds\naborted by user` format for shell-family tools that the model has been trained to recognize.

Tool *specs* (the JSON schemas the model sees) come from `ToolRouter::model_visible_specs()`. This is the only tool list passed to the API; "deferred" tools like dynamic MCP tools that haven't been "discovered" yet are filtered out. That filtering is per-prompt:

```rust
// codex-rs/core/src/session/turn.rs:954-962
let tools = if deferred_dynamic_tools.is_empty() {
    router.model_visible_specs()
} else {
    router
        .model_visible_specs()
        .into_iter()
        .filter_map(|spec| filter_deferred_dynamic_tool_spec(spec, &deferred_dynamic_tools))
        .collect()
};
```

The pattern of "static tool list, filtered at the last second" is the right move for cache: most turns reuse the same spec set, so the prompt prefix stays stable.

## Prompt construction & cache discipline

`build_prompt` ([`turn.rs:942`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/session/turn.rs#L942)) is anticlimactically small:

```rust
// codex-rs/core/src/session/turn.rs:964-974
Prompt {
    input,
    tools,
    parallel_tool_calls: turn_context.model_info.supports_parallel_tool_calls,
    base_instructions,
    personality: turn_context.personality,
    output_schema: turn_context.final_output_json_schema.clone(),
    output_schema_strict: !crate::guardian::is_guardian_reviewer_source(
        &turn_context.session_source,
    ),
}
```

The heavy lifting happens upstream. `base_instructions` is fetched from the session (which built it once at session start from the configured personality, the `AGENTS.md` files in `cwd`, the active permission profile, and any user-defined system-prompt overrides). The full system prompt is composed from a tree of small Markdown fragments under `core/src/context/prompts/` — there's a separate file for each `(approval_policy, sandbox_mode)` combination, and they get glued together by the personality builder.

For the prompt cache, the relevant line is in `client.rs`:

```rust
// codex-rs/core/src/client.rs:878
let prompt_cache_key = Some(self.client.state.conversation_id.to_string());
```

That key is sent on every Responses API request. Within a single conversation, the system prompt + tool specs + prior history form a long stable prefix, and the cache key tells the API "this is the same conversation, reuse the prefix." Codex makes no attempt to manage prompt-cache shape itself — there are no `cache_control` markers in the codebase, no manual breakpoints. It relies entirely on the API doing the right thing with `prompt_cache_key`. The trade-off: simpler code, but no fine-grained control if you wanted to (say) cache the tool spec but not the recent history.

The cache discipline is *implicit and structural*. Tool specs come from a deterministic registry; system prompts come from versioned Markdown files; the conversation history is always passed in the same canonical order via `clone_history().for_prompt(...)`. As long as nothing reorders or mutates those, the prefix stays cacheable. The risk is that any new "context fragment" injected mid-conversation (a freshly loaded skill, a hook-added context) will invalidate the cache from that point forward. Codex doesn't appear to track this — it just pays the cost.

## State / session management

There are two layers of state and they are deliberately distinct.

**In-memory session state** lives in `core/src/session/session.rs::Session`. It owns the conversation history, the pending-input queue, the active turn context, hook state, MCP connection manager, plugins, and analytics. Every method is `async fn` because almost everything is behind a `tokio::sync::Mutex` or `RwLock`. The history is not a flat `Vec<Message>` — it's a `Vec<ResponseItem>` where `ResponseItem` is the same enum the Responses API uses, so reconstructing a prompt is just `clone_history().for_prompt(modalities)` with no transformation.

**On-disk state** is the rollout store, in the dedicated `rollout/` crate. Every `ResponseItem`, every `EventMsg`, every config change, is appended to a JSONL file in `~/.codex/sessions/<thread-id>/`. The recorder is `rollout::recorder::Recorder` and it writes synchronously per-record. On thread restart, `rollout::reconstruction::reconstruct_session` replays the file to rebuild the in-memory session.

Compaction is the third leg. When `total_usage_tokens >= auto_compact_token_limit`, `run_auto_compact` fires. The "compact" is itself a model call — a sub-turn with a special prompt loaded from disk:

```text
# codex-rs/core/templates/compact/prompt.md
You are performing a CONTEXT CHECKPOINT COMPACTION. Create a handoff summary
for another LLM that will resume the task.

Include:
- Current progress and key decisions made
- Important context, constraints, or user preferences
- What remains to be done (clear next steps)
- Any critical data, examples, or references needed to continue

Be concise, structured, and focused on helping the next LLM seamlessly continue the work.
```

That's the entire compaction prompt. No few-shot examples, no enforced structure, no JSON schema. They trust the model to do the right thing and just keep the prompt minimal.

There are two compaction strategies — *inline* (codex calls the model itself with the compaction prompt and replaces history with the result) and *remote* (the provider supports server-side compaction, so codex sends a special request and lets the API handle it; selected by `should_use_remote_compact_task(provider)` at [`compact.rs:62`](https://github.com/openai/codex/blob/17ae906048d1ad9682b6f94f1513ba5f807cd038/codex-rs/core/src/compact.rs#L62)). Either way, after compaction the websocket session is reset (`client_session.reset_websocket_session()`) because the prompt prefix has changed and the cache is dead.

The `ThreadMemoryMode` field is a separate concern — it gates whether this thread is eligible for *future* memory generation (i.e. extracting durable user preferences or facts to persist across threads). Compaction is per-turn; memory is per-account.

## Three patterns I'd steal

1. **`RwLock<()>` as a parallel-vs-serial gate.** I described this above. It's three lines of real code and replaces a custom scheduler. Any agent codebase that wants to interleave read-only tools with mutating tools should look at this exact pattern before reaching for anything fancier.

2. **Submission/event split as the only public API.** A thread is a `submit() / next_event()` pair. There is no `send_message_and_wait_for_reply()`. Every UI is forced to handle the event stream, which means streaming, cancellation, and out-of-band events (warnings, hook outputs, rate-limit notices) all fall out for free. The cost is that simple synchronous use cases require a small amount of glue, but you can ship that glue once as a helper and never write it again.

3. **Tool router rebuilt per sampling request, not per turn.** Surfaces (skills, plugins, MCP servers, connectors) can change mid-turn. Rebuilding the visible tool set every iteration costs ms; getting it wrong means the model sees stale tools or missing ones. Cheap insurance, and the pattern composes well with caching because the tool *specs* are still deterministic given the same inputs.

## Three pitfalls in this codebase

1. **`session::turn.rs::run_turn` is 2,285 lines and growing.** The function has a doc comment that's perfectly clear about its job, but the body has accumulated every cross-cutting concern: hook execution, skill loading, plugin resolution, MCP discovery, telemetry, compaction, rate-limit handling, transport fallback, and stop-hook re-entry. The `#[expect(clippy::await_holding_invalid_type, reason = "...")]` annotation tells you the file has known correctness traps that the team has decided to live with. A fresh contributor trying to add "one more" pre-turn step has to read all 2,285 lines first. I'd refactor by extracting an explicit `TurnPipeline` of stages with a typed state machine, so each stage is a `fn(state) -> Result<state>`.

2. **No cache-control markers anywhere.** Relying on `prompt_cache_key` plus structural stability is fine until it isn't. The moment a new context-injection feature lands (a fresh skill, a hook-injected context, a mid-turn personality change), the cache gets invalidated for the rest of the conversation and there's no way to say "cache up to *here*, treat the rest as scratch." For an agent that runs hour-long sessions this matters in real money. I'd want explicit cache breakpoints and a `Prompt` builder that knows which segments are cacheable.

3. **The `Codex` / `CodexThread` / `Session` ownership is hard to follow.** `CodexThread` wraps `Codex`, `Codex` owns `Session`, `Session` is held in an `Arc` and passed everywhere. About a third of the code I read is plumbing that exists because of this layering. I don't fully understand why `Codex` and `CodexThread` are separate types — my best guess is that `Codex` was the original and `CodexThread` is a newer wrapper that adds rollout-path / out-of-band-elicitation accounting without touching the original. If that's right, this is technical debt to be paid down rather than a deliberate design.

## Comparison hook

Where `aider` (the Python pair-programmer in the same list) builds its agent loop around a **`Coder` class with subclasses per edit format** (`EditBlockCoder`, `UDiffCoder`, `WholeFileCoder`, etc.), codex builds its loop around a **single `run_turn` function with extensive per-turn configuration** (`TurnContext`). Aider's "shape of the response" is encoded in the type system; codex's is encoded in tool specs. Aider trades flexibility for cache stability (the system prompt is exactly the same every turn for a given coder); codex trades cache stability for flexibility (skills and tools come and go). Different sweet spots: aider for "I want a deterministic refactor right now," codex for "I want to compose 30 MCP servers and have the agent figure it out."

## Reading order

For a new contributor, read in this order:

1. `codex-rs/README.md` — the workspace map.
2. `codex-rs/protocol/src/lib.rs` — the wire types every other crate depends on. `Op`, `Event`, `EventMsg`, `ResponseItem` are the vocabulary.
3. `codex-rs/core/src/codex_thread.rs` — the public surface of a thread. Short and clean.
4. `codex-rs/core/src/session/turn.rs` — the agent loop. Read the doc comment at line 120 first, then `run_turn`, then `run_sampling_request`. Skip the streaming-event handlers on first pass.
5. `codex-rs/core/src/tools/router.rs` and `parallel.rs` — how tools get dispatched. Small and worth reading in full.
6. `codex-rs/core/src/compact.rs` plus `core/templates/compact/prompt.md` — the compaction strategy in one sitting.
7. `codex-rs/rollout/src/recorder.rs` — how state hits disk. Once you've read this, the persistence story is complete.
8. Pick one binary (`codex-rs/exec/src/main.rs` is the simplest) to see how a UI consumes the event stream.

After that, the rest of the workspace is a list of capabilities you can read in any order.
