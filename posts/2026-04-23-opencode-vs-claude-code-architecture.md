---
title: "opencode vs claude-code: a side-by-side architecture diff"
date: 2026-04-23
tags: [architecture, opencode, claude-code, agents, cli]
est_reading_time: 14 min
---

## TL;DR

`opencode` and `claude-code` look superficially similar — both are terminal-native coding agents that wrap a frontier LLM, expose a tool-calling loop, and edit files in your working directory. Underneath, they make very different bets. `claude-code` is a single-vendor, single-binary product designed around one model family and one very opinionated agent loop. `opencode` is a model-agnostic, plugin-first runtime that treats the agent loop, the tool registry, and the model adapter as separable concerns. After running both as my daily drivers for several weeks, the diff that matters is not "which is smarter" — it is **where the seams are**, because the seams are what determine what you can extend, swap, and instrument. This post walks through five of those seams: process model, tool registry, context construction, model adapter, and persistence. Code samples are simplified but structurally faithful to what each project actually does.

## The problem

When you start writing your own agentic tooling on top of an existing CLI, the first question is not "is this agent good?" — it is "where is the boundary I'm allowed to cut?" In a closed-loop product, the answer is usually "nowhere; use the public surface." In an open runtime, the answer is "anywhere, but you'll pay for it in upgrade pain." I needed a clear mental model of where each project draws those lines so I could decide which one to build extensions against. The thing that kept tripping me up was that both projects use almost identical vocabulary — "tool", "subagent", "session", "MCP server", "permissions" — but the words mean structurally different things. This post is the diff I wish I had on day one.

## Setup

- `opencode` ~0.5.x, run from npm; Node 22.
- `claude-code` recent CLI release; same machine.
- Both pointed at a small monorepo (~80k LoC, TypeScript + Python) with the same MCP servers wired in (filesystem, a local search index, a Postgres MCP).
- Observation method: I traced both with `strace`-equivalents (`fs_usage` on macOS), read the open-source code where available, and reverse-engineered the closed parts from the documented surface plus on-disk artifacts (config files, session logs, IPC sockets).

I am explicitly not benchmarking model quality here. Both can drive Claude Sonnet/Opus tier models; quality is a model-and-prompt question, not an architecture question.

## Seam 1: process model

The first real divergence is what runs in what process.

`claude-code` is, from the outside, **one long-lived process**. The CLI binary boots, opens a TUI, and from there the agent loop, tool execution, and model I/O all happen in that single process tree. Subagents (the "Task" tool) are not separate OS processes — they are nested loops inside the same runtime, with a separate context window but shared memory and shared tool registry. This keeps latency low (no IPC) and makes state easy to reason about, but it means a misbehaving tool can take the whole session down, and you can't easily attach a different debugger to "just the subagent."

`opencode` splits responsibilities across processes more aggressively. The TUI is one process, the agent runtime is another (often a Bun/Node server you can also reach over HTTP), and tools that come from MCP servers live in their own processes connected over stdio or HTTP. The agent loop talks to tools through a typed RPC boundary, not a function call. A simplified view:

```
┌────────────┐    HTTP/IPC    ┌─────────────────┐    stdio/HTTP    ┌───────────────┐
│  TUI (TS)  │ ─────────────► │  agent runtime  │ ───────────────► │ MCP server(s) │
└────────────┘                │  (loop + adapter)│                  └───────────────┘
                              └─────────────────┘
                                       │
                                       ▼
                                ┌─────────────┐
                                │  model API  │
                                └─────────────┘
```

The practical consequence: in `opencode` you can kill the TUI and reattach later, you can drive the runtime from a script, and a runaway tool only takes itself down. In `claude-code` you get a tighter, lower-latency experience but fewer escape hatches.

If you are building a "headless agent that runs in CI and posts results to a chat surface," the `opencode` shape is the obvious target. If you are building a "pair programmer who lives in my terminal and never leaves," the `claude-code` shape is fine and arguably simpler.

## Seam 2: the tool registry

Both projects expose tools as named, schema-described functions the model can call. The structural difference is **where tools come from and how they are merged.**

`claude-code` has a fixed set of built-in tools (Read, Write, Edit, Bash, Grep, Glob, WebFetch, Task, TodoWrite, etc.) plus MCP-provided tools merged in at startup. The built-ins are first-class: the system prompt teaches the model their idioms, and the runtime has special-cased UI for some of them (e.g. the TodoWrite checklist render). Adding a tool means either standing up an MCP server or waiting for the vendor to add it. You cannot, from user space, redefine `Read` to mean something different.

`opencode` treats every tool the same way. Built-ins live in a registry, MCP-supplied tools live in the same registry, and a plugin can register additional tools — or even shadow a built-in — at config time. A minimal plugin looks roughly like:

```ts
// ~/.config/opencode/plugin/audit.ts
import type { Plugin } from "@opencode-ai/plugin"

const auditPlugin: Plugin = async ({ app, $ }) => {
  return {
    "tool.execute.before": async (input, output) => {
      // Log every tool call to a local audit file before it runs.
      const line = JSON.stringify({
        ts: Date.now(),
        tool: input.tool,
        sessionID: input.sessionID,
        args_keys: Object.keys(output.args ?? {}),
      })
      await Bun.write(
        `${process.env.HOME}/.local/share/opencode-audit.jsonl`,
        line + "\n",
        { append: true },
      )
    },
  }
}

export default auditPlugin
```

That hook fires for every tool, built-in or MCP, every session. The equivalent in a closed-loop CLI is "wait for the vendor to add an audit log feature."

The tradeoff: `opencode`'s flexibility means the schema for what a tool *is* has to be stable enough for third parties to depend on, which slows the rate at which built-ins can change shape. `claude-code` can rev its built-in tool set faster because nothing external is bound to the internals.

## Seam 3: context construction

This is the seam I find most under-discussed and most consequential. Both agents have to assemble a prompt out of: system instructions, project-level instructions (`AGENTS.md` / `CLAUDE.md`), conversation history, tool results, and possibly some retrieved snippets. Where they differ is in **what the agent sees vs. what the user sees.**

`claude-code` has a fairly opinionated "project context" model. It walks up from the working directory, collects `CLAUDE.md` files, and stitches them into the system prompt with a documented precedence order. It also injects an environment block (cwd, platform, date, git status) that the model can rely on. The conversation transcript the model sees is *almost* the conversation you see — the main difference is that long tool outputs may be summarized or elided after some turns to keep the window manageable.

`opencode` exposes the context layering more explicitly. There is a documented `AGENTS.md` resolution chain, per-agent prompt overrides, and the context-pruning policy is pluggable. A simplified version of the resolution looks like:

```ts
// pseudo-code, not the real source
function resolveSystemContext(cwd: string, agent: string): string {
  const layers = [
    readGlobal("~/.config/opencode/AGENTS.md"),
    ...walkUp(cwd, "AGENTS.md"),         // root-most first
    readAgentProfile(agent),             // per-agent override
    readSessionInjections(),             // anything a plugin pushed in
  ]
  return layers.filter(Boolean).join("\n\n---\n\n")
}
```

Two practical implications:

1. In `opencode` you can write a plugin that injects a "current sprint goal" block on every turn without touching any committed file. In `claude-code` the same effect requires editing `CLAUDE.md`, which is committed and visible to teammates.
2. Debugging "why did the model do X?" is easier in `opencode` because the resolution chain is inspectable. In `claude-code` you mostly trust the documented precedence and reason backwards from behavior.

Neither is strictly better. Opaqueness is a feature when it lets the vendor enforce safety invariants; transparency is a feature when you need to prove to yourself that no one snuck a `system: "ignore previous instructions"` into your context.

## Seam 4: the model adapter

`claude-code` is, by design, bound to one model family. The system prompt, the tool-use protocol, the way assistant messages are parsed — all of it assumes a specific provider's message format. You can point it at different sizes within that family, but you cannot trivially swap in a model from a different provider without losing fidelity (tool calls, prompt caching markers, multimodal blocks).

`opencode` treats the model as an adapter. There is a provider abstraction that maps the canonical "agent message" type onto whatever wire format a given provider expects. A toy adapter shape:

```ts
interface ModelAdapter {
  id: string
  // Convert canonical messages into provider-native request body.
  request(messages: AgentMessage[], tools: ToolSchema[]): ProviderRequest
  // Stream provider response back into canonical assistant deltas.
  stream(resp: ProviderStream): AsyncIterable<AssistantDelta>
  // Optional capability flags used by the loop to decide what to send.
  caps: {
    parallelToolCalls: boolean
    promptCache: "anthropic" | "openai" | "none"
    vision: boolean
  }
}
```

The agent loop only depends on the canonical types. Add a new provider, you add an adapter; you don't touch the loop. This is how `opencode` ends up supporting a long tail of providers (cloud frontier models, local models via OpenAI-compatible servers, gateway proxies) without each one needing bespoke loop logic.

The tax: any feature that only one provider supports (Anthropic's prompt-cache breakpoint markers, OpenAI's `parallel_tool_calls=false`, Gemini's safety settings) has to be hidden behind the capability flag, and the loop has to have a sensible fallback. Closed CLIs skip this tax by simply not supporting the other providers.

## Seam 5: persistence and replay

The last seam is what gets written to disk between turns and across sessions.

`claude-code` keeps a session log per project under `~/.claude` (the exact layout has changed across versions). It stores enough to reconstruct the visible transcript and, in recent releases, enough to resume a session. The format is internal; you should not script against it.

`opencode` writes session state as structured JSON the runtime itself reads on resume. Because the runtime is open, the schema is stable enough to query directly. A useful trick: tail the session log with `jq` to build a token-cost dashboard without any extra instrumentation:

```bash
# Sum input/output tokens per session for today.
jq -s '
  map(select(.type=="assistant" and .usage)) |
  group_by(.sessionID) |
  map({
    session: .[0].sessionID,
    in:  (map(.usage.input_tokens)  | add),
    out: (map(.usage.output_tokens) | add),
  })
' ~/.local/share/opencode/session/*.jsonl
```

You can do the same with `claude-code` only if you scrape the on-disk log — which works today but is not a contract.

Replay is the more interesting capability. With a stable session schema you can take a session that went badly, replace one tool result with a hand-edited version, and re-run the loop from that point to see if the trajectory improves. This is how you build an eval set from real usage rather than synthetic prompts. `opencode`'s shape makes this a normal workflow; in `claude-code` it is a research project.

## When each shape wins

A short, opinionated rubric, based on the seams above:

- **Pick the closed, single-vendor CLI when**: you want the lowest-friction "open terminal, get help" experience; you trust one vendor's model family for the next 12 months; you do not need to script the agent from outside; you value tight UI/model integration over extensibility.
- **Pick the open, plugin-first runtime when**: you need to swap models (cost, latency, region, sovereignty); you want to add custom tools without standing up an MCP server for each one; you need machine-readable session logs for telemetry, eval, or replay; you intend to drive the agent from CI or another service, not just a terminal.

A team can reasonably run both. The open runtime becomes the substrate for agents that ship as part of the product (CI bots, review agents, on-call helpers), and the closed CLI stays as the daily-driver pair-programmer for individual engineers. The mistake is picking one and pretending the other shape doesn't exist — you end up either reinventing extensibility on top of a closed product, or reinventing polish on top of a runtime.

## What I would do differently

I spent too long trying to make the open runtime feel like the closed CLI (skinning the TUI, hiding the model adapter, baking in defaults). The right move was to lean into the seams: write small plugins, keep model selection explicit, treat the session log as a first-class artifact. The open shape pays off only if you actually use the openness.

## Links

- opencode docs: https://opencode.ai/docs
- Anthropic claude-code docs: https://docs.claude.com/en/docs/claude-code/overview
- Model Context Protocol: https://modelcontextprotocol.io
- Anthropic prompt caching: https://docs.claude.com/en/docs/build-with-claude/prompt-caching
- OpenAI tool calling: https://platform.openai.com/docs/guides/function-calling
