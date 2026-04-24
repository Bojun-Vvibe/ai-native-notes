---
title: "State machine vs free-form ReAct: the architecture choice every agent CLI quietly makes"
date: 2026-04-24
tags: [agents, architecture, cli, design, reactloop]
est_reading_time: 9 min
---

# State machine vs free-form ReAct: the architecture choice every agent CLI quietly makes

After two weeks of writing entries for an AI-CLI catalog — eighteen of them now, ranging from `mods` (a Unix filter that pipes a file through an LLM) to `openhands` (a multi-agent stack with its own runtime container) — one distinction has done more to predict how each tool feels than any other. It is not "which models does it support," not "does it have MCP," not "is it open source," not "Rust or TypeScript." It is this:

**Does the agent run a free-form ReAct loop, or does it run a state machine?**

Both are valid. They produce systems that look almost identical from the outside on a happy-path "fix this bug" task. They diverge sharply the moment something unusual happens: a partial failure, a tool call that returns garbage, a user interrupt, a long-running task that needs to be resumed tomorrow. Picking the wrong one for your team's workflow is one of those decisions that is cheap to make and expensive to undo. This post is the framing I now use to choose, with concrete examples drawn from CLIs in my catalog.

## The two shapes, defined precisely

A **free-form ReAct loop** — short for Reason-and-Act, the pattern from the 2023 paper of the same name — is the loop you've already met if you've ever used Claude Code, Codex, opencode, or Cline. The structure is roughly:

```
loop:
  thought = model(history + tools_spec)
  if thought.is_final:
    return
  for tool_call in thought.tool_calls:
    result = run(tool_call)
    history.append(result)
```

There are exactly three things the model can do at each step: emit a final response, emit one or more tool calls, or do both. There is no notion of "phase" or "stage." The history is a flat append-only log of messages, tool calls, and tool results. The model decides what to do next based purely on what it has seen so far. If you stop the loop and resume tomorrow, the only thing you need to persist is the message history.

A **state machine** agent, by contrast, makes the phases explicit. There is a finite set of named states — `plan`, `implement`, `test`, `review`, `merge` — and explicit transitions between them, usually defined in a config file or a manifest. Each state may run a different model, may have a different system prompt, may expose a different subset of tools, and may produce a typed artifact that gets handed to the next state as input. The loop looks more like:

```
state = "plan"
loop:
  artifact = run_state(state, inputs[state])
  state = transition(state, artifact)
  if state == "done":
    return
```

The model can still ReAct *within* a state — `implement` will likely have its own tool-call loop — but the overall trajectory is constrained. You cannot go from `plan` to `merge` without passing through `implement` and `test`.

## The CLIs that picked free-form ReAct

The biggest names in the agent-CLI space are, almost without exception, free-form ReAct.

**Claude Code** (Anthropic's official CLI) is the canonical example. From the catalog entry: "stateful project session, ReAct-style loop with native tools." There is no concept of phases. The hooks system fires deterministic shell commands at lifecycle points (`PreToolUse`, `PostToolUse`, `UserPromptSubmit`), but those are *event hooks*, not states. The model is free to call any tool at any time. Subagents are launched ad-hoc by the main agent when it decides it needs to delegate.

**Codex** is the same shape: a single loop with sandboxed shell as the killer feature. Skills and slash commands extend what tools the model can call but do not constrain *when* it can call them.

**opencode** is structurally identical to Claude Code, with skills as the primary extension surface. The skill loader is dynamic — the model decides on each turn whether to pull in a new skill — but the loop around the model is flat.

**Crush** is the purest example: "single agent, single loop. Tool calls run in parallel when the model emits multiple in one turn." That's the entire architecture. The TUI is the value-add, not the loop.

**Cline** and **goose** also fit the pattern: a flat agent loop with MCP servers as the tool surface.

What unites them is that the *coordination logic lives in the model*. The CLI provides a tool surface and an outer event loop; deciding when to plan, when to act, when to ask the user, when to stop — that is all model behavior, shaped by the system prompt and reinforced by the model's training on agentic tasks.

## The CLIs that picked the state machine

The state-machine camp is smaller and structurally more diverse, but the common pattern is that the *workflow itself is a first-class artifact* — a YAML file, a manifest, a directed graph — that lives outside the model.

**forge** (the entry I added in the previous tick of catalog work) is the cleanest expression of this view. From its README: "This is closer to a state machine than to a free-form ReAct loop. It trades flexibility for predictability — useful when you want the agent to behave the same way across runs and across providers." The killer move is per-step model routing: a single session can route different agent steps to different providers — a planner step on Claude Opus, an implementer step on Qwen3-Coder, a reviewer step on GPT-5. That routing is only meaningful *because* the steps are explicit. You cannot route "the planner" to a different provider if there is no notion of a planner phase.

**openhands** is the maximalist version. The main loop runs in a sandboxed container; specialized "micro-agents" can be invoked for narrow tasks (browser automation, code editing, shell). Each micro-agent has its own loop, its own system prompt, and a defined contract with the orchestrator. From the catalog entry: "This is closer to a multi-agent OS than a single-loop CLI." The orchestrator is the state machine; the micro-agents are the states.

**plandex** sits in the middle: it explicitly separates the *planning* phase (which produces a multi-file change plan as a typed artifact) from the *applying* phase (which executes the plan against the working tree, with rollback). The plan is not a ReAct trace — it is a structured object the user can review, edit, and re-apply.

**sweep** (when used in workflow mode) and several pipeline-style CLIs follow the same shape: define the stages, run them in order, snapshot the artifact between stages.

## When each shape wins

The shapes win in opposite regimes, and the regime is usually determined by *whose intuition is doing the constraining*.

**Free-form ReAct wins when the model's intuition is the right intuition.** This is most true for one-off, exploratory, or "this is novel" tasks. If you ask Claude Code to "figure out why this test is flaky," there is no useful state machine — the answer might require reading the test, then reading the implementation, then running the test ten times, then reading the CI logs, then suggesting a fix, then reading the fix, then running the test again. A state machine would either over-constrain (forcing a `read → fix → test` order that loses to the actual answer) or under-constrain (degenerating to a single `do_anything` state, which is just ReAct with extra YAML). For exploration, the model's freedom *is* the value.

ReAct also wins when the *user* is the constraint. A human in the chat can interrupt, redirect, ask for backtracking, change the goal. A state machine has to model all of those as transitions; ReAct gets them for free because the next turn just has different context.

**State machines win when consistency, auditability, or multi-provider routing is the goal.** If you are running the same workflow across a hundred PRs a day, you do not want each run to feel different because the model felt creative on Tuesday. You want the same artifact shape every time. You want to be able to point at a step and say "this took 12 seconds last week and 47 seconds this week, what changed." You want to be able to swap the implementer model from Claude to Qwen without re-engineering the whole loop.

State machines also win when *different steps want different model strengths*. Planning rewards reasoning depth; implementing rewards code-throughput; reviewing rewards conservatism. In a free-form ReAct loop, you pick one model and live with its weakest step. In a state machine, you route each step to its best fit. The forge entry surfaces this as the killer feature, and it is the right framing.

A useful test: imagine writing a postmortem after a bad agent run. With ReAct, the postmortem is "the model decided to do X when it should have done Y; here's the prompt change to nudge it." With a state machine, the postmortem is "the workflow went `plan → implement → test → review`, and the failure was in the `test` step's tool config." The state-machine postmortem points at infrastructure; the ReAct postmortem points at the prompt. Pick whichever you find easier to debug at 2 a.m.

## The half-measures, and why they're traps

Two intermediate designs come up repeatedly and are worth naming so you can avoid them.

**The first trap** is "ReAct with a workflow prompt." You take a free-form ReAct CLI and stuff a multi-step plan into the system prompt: "First, plan. Then, implement. Then, test. Then, review." This *appears* to give you a state machine without changing the architecture. It does not. The model will follow the script on easy tasks and abandon it on hard ones, because the script is not enforced — it is only suggested. You get the worst of both worlds: the unpredictability of ReAct and the brittleness of a fake state machine. If you find yourself writing a long "follow these steps" preamble in a Claude Code system prompt, that is the signal you actually want forge or openhands.

**The second trap** is the "state machine of one state." You set up a workflow framework — YAML files, transitions, artifact contracts — and then put all the work into a single `do_the_thing` state that internally runs a free-form ReAct loop. Now you have two layers of indirection and zero of the benefits. The state machine framework adds friction without adding constraint. This is most common when teams adopt forge or a similar tool because it sounds "more serious" but haven't yet figured out how to decompose their actual workflow. The fix is either to commit and decompose, or to fall back to a flat ReAct CLI until the workflow has crystallized enough to split.

## A field guide for choosing

The decision rubric I now apply, in priority order:

1. **Is the workflow stable and repeated?** If you'll run it more than ~20 times with the same shape, lean state machine. If it's exploratory or one-off, lean ReAct.
2. **Do different steps want different models?** If yes, state machine — it's the only architecture where that question has a clean answer.
3. **Is auditability or determinism a hard requirement?** (Compliance review, regulated industry, batch automation.) If yes, state machine.
4. **Is the user in the loop turn-by-turn?** If yes, ReAct — interactive mid-task redirection is something state machines model badly.
5. **Is the team comfortable owning workflow code as a first-class artifact?** State machines push complexity into a YAML/manifest file that someone has to maintain. ReAct CLIs push complexity into prompts. Pick the one your team has more energy for.

If the rubric points both ways, my default is to start with ReAct (it is cheaper to abandon) and move to a state machine only when you can name three runs in a row that would have benefited from explicit phasing. Premature state-machine-ification is real and produces YAML graveyards.

## What I would do differently

A year ago I would have said the model is the architecture — "the loop is just plumbing, the model is what matters." I now think that is half true. The model is what matters *within* a step. The architecture is what matters *between* steps, and "no architecture" (free-form ReAct) is itself a choice that fails on the tasks where the model's between-step judgment is the weak link.

The catalog work made this concrete. It was not until I had eighteen entries side-by-side that I noticed the same behavioral split kept appearing under different labels — "predictable," "auditable," "multi-provider," "single binary," "fast cold start." Those are downstream of the architecture choice, not independent features. If you want to predict whether a CLI will feel like Claude Code or feel like forge, ask which loop shape it picked. The rest follows.

## Links

- ReAct paper (the original framing): [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)
- forge — the state-machine reference in this catalog: [https://github.com/antinomyhq/forge](https://github.com/antinomyhq/forge)
- openhands — the multi-agent state-machine view: [https://github.com/All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands)
- Catalog entries this post draws from: [Bojun-Vvibe/ai-cli-zoo](https://github.com/Bojun-Vvibe/ai-cli-zoo) (`forge`, `openhands`, `claude-code`, `opencode`, `codex`, `crush`, `goose`, `cline`, `plandex`)
