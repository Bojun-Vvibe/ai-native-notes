---
title: "Sub-agent context isolation: why your helper agents need their own room"
date: 2026-04-23
tags: [agents, sub-agents, context, prompts, architecture]
est_reading_time: 14 min
---

## TL;DR

A "sub-agent" is whatever you call the smaller agent that the main agent dispatches to do focused work — search a codebase, summarize logs, draft a migration. Most agent frameworks expose this as a single tool call ("Task" or similar) and most users treat the result as just another tool output. That works until the moment you discover the parent agent has been silently inheriting the sub-agent's mistakes, drift, and even its hallucinations into its own context. The fix is **context isolation**: a sub-agent should have its own context window, its own tool scope, its own system prompt, and a strictly narrowed return channel back to the parent. This post explains why each of those four boundaries matters, what breaks when you skip them, and a working pattern (with code) that I have used to keep multi-agent setups predictable across long sessions.

## The problem

The first time I noticed this failure mode, I was using a parent agent to drive a refactor. The parent dispatched a sub-agent to "find every callsite of `legacyAuth` in the repo." The sub-agent did the search, plus — without being asked — opened three of the files, read them, and decided two of them "looked unrelated and could be ignored." It returned a list of callsites that excluded those two files. The parent, trusting the sub-agent's summary, made the refactor based on the filtered list, and missed those two files. Tests passed. The bug shipped. When I went back to debug, the sub-agent's full transcript was not in the parent's context — only its final summary was — so neither I nor the parent had any easy way to see that the filtering had happened. The lesson was not "sub-agents are bad." It was "I had not actually thought about what the sub-agent was allowed to decide on its own."

## What "isolation" actually means

When people say "sub-agent" they usually mean one of three different things, and the isolation requirements are different for each:

1. **A scoped query** — the parent wants an answer to a narrowly defined question. Example: "list the files matching this pattern." The sub-agent should not be making decisions; it should be gathering facts.
2. **A delegated task** — the parent wants a self-contained piece of work done. Example: "write a unit test for this function and confirm it passes." The sub-agent has authority over how, but the scope is bounded.
3. **A peer collaborator** — multiple agents working on different parts of a larger problem, exchanging structured messages. Example: a planner agent and an implementer agent working on a sprint.

Most frameworks ship one primitive ("Task") and try to cover all three with prompt engineering. That is the source of the mess. Each shape needs different defaults; isolation is what makes them distinguishable.

The four boundaries that constitute isolation:

- **Context window.** The sub-agent does not see the parent's full transcript; the parent does not see the sub-agent's intermediate steps.
- **Tool scope.** The sub-agent has only the tools its job requires.
- **System prompt.** The sub-agent has its own persona and rules, not a copy of the parent's.
- **Return channel.** The parent receives a typed result, not a free-form transcript.

Skipping any one of these creates a specific bug class. Walk through them in order.

## Boundary 1: context window

The parent has, at any point, a long history of user input, file reads, tool results, and its own previous reasoning. A naive sub-agent dispatcher copies all of that into the sub-agent's first message ("here is what we're working on"). Two problems:

1. **The sub-agent gets distracted.** Given the full context, it will start trying to be helpful about things outside its job. This is where my "filter the file list" failure came from — the sub-agent had read the parent's plan and decided to "save work" by pre-filtering.
2. **Cost balloons.** Every sub-agent dispatch becomes a full prompt re-send. If you dispatch five sub-agents in a session, you have effectively paid for five full transcripts, not one.

The fix: send the sub-agent only the **task brief** — a small, structured payload designed for its job — plus whatever shared anchors it needs (project root, an `AGENTS.md` slice, the relevant file paths). Treat the parent's transcript as private.

```ts
type SubAgentBrief = {
  job: "scoped_query" | "delegated_task" | "peer_message"
  goal: string                  // one paragraph, written by the parent
  inputs: Record<string, unknown> // structured, not free-text
  return_schema: JsonSchema     // what the parent expects back
  budget: { max_tool_calls: number; max_tokens: number }
}
```

The brief is what the sub-agent's first turn sees. The parent's transcript is not appended. This forces the parent to *write down* what it actually wants, which has the side benefit of catching about a third of the bugs at the dispatch step rather than after the sub-agent runs.

## Boundary 2: tool scope

The default in most frameworks is "the sub-agent inherits all the parent's tools." This is the wrong default for the same reason you don't give every microservice the same database credentials. Tools are capabilities; capabilities should match jobs.

A sub-agent doing a "find callsites" job needs `grep` and `glob`. It does not need `write_file`, `run_shell`, or `delete`. Restricting the tool set has three benefits:

- **Hard guarantees.** A sub-agent that has no `write_file` tool cannot edit a file, no matter how creatively it interprets the prompt.
- **Smaller schemas in context.** Tool schemas count against the context window and against the prompt cache. Five tools instead of twenty saves real tokens on every turn.
- **Cleaner reasoning.** The model spends less effort considering tools that are obviously irrelevant.

A practical pattern is a small registry of pre-baked sub-agent profiles, each with its own tool list:

```ts
const SUBAGENT_PROFILES = {
  searcher: {
    tools: ["grep", "glob", "read_file"],
    system: "You answer narrow factual questions about a codebase. Do not modify files. Do not interpret intent — return facts.",
  },
  test_writer: {
    tools: ["read_file", "write_file", "run_tests"],
    system: "You write and run unit tests. You may modify only test files. If asked to modify production code, refuse and report.",
  },
  reviewer: {
    tools: ["read_file", "git_diff"],
    system: "You review proposed changes. You produce structured findings. You do not modify files.",
  },
}
```

When the parent dispatches, it picks a profile by job, not by improvising. If a new kind of job appears often enough, it gets its own profile. The friction of adding a profile is itself useful — it forces you to name what the sub-agent is for.

## Boundary 3: system prompt

This is the boundary most often skipped, because it feels redundant ("the parent already knows the rules; surely the sub-agent inherits them"). It does not, and it should not.

The parent's system prompt is built around being a generalist driver. It usually contains things like "use TodoWrite to plan", "prefer parallel tool calls", "ask the user when unsure." None of those are appropriate for a sub-agent doing a scoped query. The searcher profile above explicitly tells the sub-agent **not** to interpret intent and **not** to "ask the user." Asking the user from inside a sub-agent is, structurally, asking the parent — which is a recipe for an infinite ping-pong loop I have actually seen in production.

A sub-agent system prompt should answer four questions in that order:

1. **What is your job?** (one sentence)
2. **What are you not allowed to do?** (explicit refusals)
3. **What does your output look like?** (the return schema, restated)
4. **What do you do when stuck?** (return a structured error, not a question to the user)

A concrete example for the `searcher` profile:

```
You are a code search sub-agent. Your job is to answer factual questions
about the contents of a repository.

Hard rules:
- Do not modify any file. You have no write tools.
- Do not interpret the user's intent. Return what was asked, not what
  you think was meant.
- Do not ask follow-up questions. If the question is ambiguous, return
  an error with `reason: "ambiguous"` and a brief explanation.

Output format: you MUST return a JSON object matching the schema you
were given. Do not include prose outside the JSON.

When stuck: return {"ok": false, "reason": "...", "details": "..."}.
Do not invent results to satisfy the schema.
```

That last rule — "do not invent results to satisfy the schema" — is the single most important sentence in any sub-agent system prompt. Models will hallucinate to fit a schema if they don't see an explicit out.

## Boundary 4: the return channel

The return channel is what the parent sees after the sub-agent finishes. The wrong default is "return the sub-agent's final assistant message verbatim." The right default is "return only data, in a schema the parent declared."

```ts
type SubAgentResult<T> =
  | { ok: true;  data: T;       used: { tool_calls: number; tokens: number } }
  | { ok: false; reason: string; details?: string;
      used: { tool_calls: number; tokens: number } }
```

Why this shape:

- **`ok` flag.** The parent can branch cleanly without parsing prose.
- **`data` is typed.** Whatever the parent declared as `return_schema` is what it gets, validated. If the sub-agent returns malformed JSON, the dispatcher catches it and surfaces a `bad_output` error to the parent — the parent never sees garbled data.
- **`used` is mandatory.** Every sub-agent reports its own resource usage, which is what makes per-session cost accounting actually work (see token telemetry — the same `Agent-Session` ID flows through).

The full transcript of the sub-agent is logged for replay and audit, but it is not pasted into the parent's context. If the parent needs more detail, it can ask follow-up questions through new dispatches; it does not get a free read on the sub-agent's reasoning.

## Putting it together: a minimal dispatcher

A small dispatcher that enforces the four boundaries:

```ts
async function dispatch<T>(
  parentCtx: ParentContext,
  brief: SubAgentBrief,
): Promise<SubAgentResult<T>> {
  const profile = SUBAGENT_PROFILES[briefProfile(brief)]
  if (!profile) return { ok: false, reason: "unknown_profile",
                         used: { tool_calls: 0, tokens: 0 } }

  // 1. Fresh context. Do NOT pass parent transcript.
  const messages: Message[] = [
    { role: "system", content: profile.system },
    { role: "user",   content: renderBrief(brief) },
  ]

  // 2. Restricted tool set.
  const tools = profile.tools.map(name => parentCtx.toolRegistry.get(name)!)

  // 3. Run the loop with explicit budgets.
  const session = await runAgentLoop({
    messages,
    tools,
    maxToolCalls: brief.budget.max_tool_calls,
    maxTokens:    brief.budget.max_tokens,
    sessionId:    `${parentCtx.sessionId}.sub.${nano()}`,
  })

  // 4. Validate output against return_schema.
  const last = session.lastAssistantMessage()
  let parsed: unknown
  try { parsed = JSON.parse(last.text) }
  catch { return { ok: false, reason: "bad_output_json",
                   used: session.usage } }

  const validated = validate(parsed, brief.return_schema)
  if (!validated.ok) {
    return { ok: false, reason: "schema_violation",
             details: validated.error, used: session.usage }
  }

  // 5. Persist transcript for audit, return only data.
  await archive(session)
  return { ok: true, data: validated.value as T, used: session.usage }
}
```

Five points worth flagging:

1. The sub-session ID is a child of the parent's session ID. This is what lets you reconstruct a full tree of agent activity later.
2. Budgets are per-dispatch and enforced by the loop, not by the model. The model cannot exceed them by being persuasive.
3. JSON validation is a hard wall. A sub-agent that cannot produce valid JSON gets `bad_output_json`; the parent decides whether to retry with a clearer schema or escalate.
4. The transcript is archived but not returned. The audit log is for humans and for replay; the parent works with `data`.
5. There is no "follow-up" capability inside a single dispatch. If the sub-agent needs more, it returns `ok: false, reason: "needs_more_input"` with structured questions, and the parent dispatches again. This prevents the sub-agent from holding a conversation with itself for 30 turns.

## Common failure modes after isolation

Two failure modes survive even with all four boundaries in place, and both have prompt-level fixes:

**Sub-agent over-narrows.** A scoped query agent, told not to interpret intent, may return zero results because the question was slightly malformed. The fix is to require the sub-agent to also return a `query_understood` field — a one-sentence restatement of what it actually searched for. The parent compares this to the brief and re-dispatches if the restatement is wrong. Cheaper than letting the parent guess why it got zero results.

**Sub-agent loops on validation.** If the schema is fiddly, the sub-agent may spend its entire budget producing JSON that fails validation. The fix is to give the sub-agent a small "draft and check" tool — a no-op tool that takes a candidate JSON and returns whether it would validate. The model uses it like a linter and converges in one or two tries.

## What I would do differently

I built the sub-agent surface as a single `Task` tool first, like everyone does. That collapsed all three job shapes (scoped_query, delegated_task, peer_message) into one, which made every prompt a compromise. Splitting them into named profiles with their own tool lists and system prompts was the change that made multi-agent setups actually predictable. If you remember nothing else from this post: **do not give your sub-agents your parent agent's tools by default.** That single decision will prevent the majority of "the sub-agent did something I did not authorize" incidents.

## Links

- Anthropic on multi-agent research systems: https://www.anthropic.com/research
- "Reflexion" paper (sub-agent loops as a learning signal): https://arxiv.org/abs/2303.11366
- LangGraph subgraph patterns: https://langchain-ai.github.io/langgraph/
- JSON Schema: https://json-schema.org/
- Model Context Protocol (for thinking about tool scoping): https://modelcontextprotocol.io
