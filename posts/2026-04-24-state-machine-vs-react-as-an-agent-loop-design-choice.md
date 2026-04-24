---
title: "State machine vs free-form ReAct: a design choice every agent loop is making whether it admits it or not"
date: 2026-04-24
tags: [agents, architecture, design, cli, react, state-machine]
est_reading_time: 11 min
---

## The problem

I've spent the last week reading the source of six agent CLIs back-to-back
for a comparison catalog. Halfway through the sixth I noticed something
the marketing pages do not say out loud: every agent loop is one of two
shapes, and each shape is making a different bet about where intelligence
should live. The shapes are easy to name once you've seen them — **state
machine** or **free-form ReAct** — but the catalog entries themselves
mostly bury the choice under feature lists. That makes it hard for a
reader to pick the right tool, because the relevant question is not "does
this CLI have skills, hooks, sub-agents?" but "does this CLI assume the
model is the planner, or does it assume the human is?"

This post is the synthesis. What the two shapes actually are, where each
one breaks, and how to tell which one a given CLI is — even if the
README never uses the words "state machine" or "ReAct."

## The setup

The data is six CLIs from `ai-cli-zoo` as of 2026-04: `claude-code`,
`opencode`, `openhands`, `codex`, `crush`, and the most recent addition,
`forge`. All are open-source or open-binary. All ship a TUI plus a
non-interactive mode. All wrap one or more LLM providers. I read the
agent loop in each — specifically the function that owns "decide what
to do next, do it, observe, decide again" — and the configuration
surface that lets a user steer that loop. No marketing text, only code
and config schemas.

The lens is borrowed from a separate piece of work on the catalog: the
`forge` entry frames itself as preferring "the state-machine view of
agent loops more than the free-form ReAct view." That phrasing was the
hook. Once I had it I could see the same fork everywhere else.

## The two shapes, concretely

### Free-form ReAct

The ReAct shape is the one most people picture when they say "agent."
You hand the model a system prompt, a set of tool schemas, and a user
message. The model emits either a tool call or a final answer. If it's
a tool call, you execute it, append the result to the conversation, and
ask the model again. The loop runs until the model says it's done or
some external condition (token budget, turn count, user `Ctrl-C`) ends
it. Control flow is **emergent**: it is whatever sequence of tool calls
the model happened to choose this time. The transition function is the
model's next-token distribution.

`claude-code`, `opencode`, `codex`, and `crush` are all this shape.
Their differences — skills, hooks, sub-agents, sandboxes — are
extensions to a fundamentally ReAct core. The model decides when to
load a skill. The model decides whether to delegate to a sub-agent. The
model decides when to stop. The human's leverage is on the **inputs and
the constraints**: which tools exist, what the system prompt says, what
hooks fire around tool calls, what context is in the window. Once the
loop starts, the model is the planner.

### State machine

The state-machine shape inverts the locus of control. The human
declares, ahead of time, the **steps** the agent will take and the
**transitions** between them. The model is invoked at each step, but
its job is narrower: produce the artifact this step demands, in the
shape this step expects. The state machine itself decides what runs
next, based on which step succeeded, which failed, which output
matched a pattern. Control flow is **prescribed**.

`forge` is the clearest example in the catalog: workflows are YAML,
each step is an "agent" with its own prompt and tool set, and transitions
between steps are explicit. `openhands`'s task graph and `aider`'s
edit-test-loop are softer versions of the same idea — the human writes
less of the structure than in `forge`, but the structure is still
externalized into config or into hard-coded loop branches, not left to
the model to invent each turn.

Spec-Kitty-style mission orchestrators — pipelines of "specify, plan,
implement, review, merge" with explicit gates — are state machines too.
So is most of what people call "agentic workflow tooling": LangGraph,
Inngest's AI flows, Temporal-on-LLMs. Different vocabulary, same
shape.

## What each shape buys you

### What free-form ReAct gets right

Open-endedness. The whole point of the shape is that the model can
react to a tool result you didn't anticipate. You ask `claude-code` to
"figure out why CI is red." It runs `git log`, reads three test files,
greps for an env var, runs the failing test locally with extra logging,
patches the bug, runs the test again. You did not write that sequence
down anywhere. You couldn't have — the sequence depends on what the
test actually does, which the model only learns by reading the file.
ReAct loops shine on tasks where the **next step depends on facts you
will only learn at execution time**, and the search space is too large
to enumerate up front.

The cognitive cost to the human is low. You write a prompt, you run
the agent, you read the transcript. There is no DSL to learn, no
graph to draw, no transitions to declare. For tasks the model is good
at, the loop just works.

### What free-form ReAct gets wrong

Determinism. Two runs of the same prompt against the same repo can
follow wildly different paths. That is fine for exploratory work and
catastrophic for anything you want to put on a cron. The model might
load skill A this time and skill B next time. It might call your
expensive provider when a cheap one would have sufficed. It might
decide it's done two turns earlier than you wanted, or three turns
later. You can constrain the loop with hooks and sandboxes, but the
**shape of the trajectory** is the model's to choose.

Cost control is hard for the same reason. The agent decides how many
turns to take, which model to call, how much context to load. You can
cap turn count, but a capped run is just a truncated run; it doesn't
mean the agent did the right amount of work. ReAct loops are easy to
start and hard to budget.

Failure modes are also harder to localize. When a state-machine step
fails, you know which step. When a ReAct loop fails, you have a
transcript and a vibe.

### What state machines get right

Reproducibility. The same workflow, run twice, takes the same path
unless an external input changed. That is the property that makes
something putable on a cron, embeddable in CI, or sellable as a
product. `forge.yaml` checked into the repo means every collaborator
runs the same agent. A spec-kitty mission with explicit gates means
the next reviewer sees the same evidence the last one did.

Cost predictability is the second property. If step 1 always runs on
a cheap local model, step 2 always runs on a frontier model, and step
3 always summarizes on Gemini Flash, you can put a number on the run
before you run it. Ratio of expensive to cheap calls is fixed by
configuration, not by whatever the planner decided this time.

Operational legibility is the third. When step 4 fails, you know it
was step 4, you know what its inputs were, and the retry semantics
are whatever the framework declares. You can write monitors. You can
write SLOs.

### What state machines get wrong

They cap the model's intelligence at the **graph designer's**
intelligence. If you didn't declare a transition for "the test failed
in a way that suggests the wrong assertion is being checked," the
graph won't take it, even if the model would happily have spotted it
in a free-form loop. You pay for reproducibility with brittleness:
the graph encodes assumptions about what kinds of states exist, and
when reality produces a state you didn't anticipate, the graph either
falls off a cliff or routes everything through a generic "ask the
model what to do" escape hatch — which is just ReAct in a trench coat.

The cognitive cost to the human is also higher. You have to think the
shape of the work through up front. For exploratory work, this is
worse than useless: you don't know the shape yet. For one-shot tasks,
it is overhead you can't amortize.

## How to tell which shape a CLI is

Three diagnostics. None of them require reading marketing.

1. **Where does the next-step decision live?** Open the main loop
   and find the function that decides what to do after a tool call
   returns. If the answer is "send the tool result back to the model
   and let it pick the next call," it's ReAct. If the answer is
   "look up the next step in a config or a graph," it's a state
   machine. `claude-code`, `opencode`, `codex`, `crush` all answer
   the first way. `forge` answers the second.
2. **What does the config file look like?** ReAct configs declare
   tools, prompts, and constraints. State-machine configs declare
   steps, transitions, and per-step routing. If the config file has
   the words `step:`, `transition:`, `on_failure:`, `next:`, you
   are in state-machine territory. If the config has `tools:`,
   `prompt:`, `model:`, `hooks:` and nothing about ordering, you
   are in ReAct.
3. **Can the same model see the whole task, or only its slice?**
   In ReAct, the model sees the running transcript and decides what
   matters. In a state machine, each step's model invocation gets
   only the inputs that step declared it needs. State machines
   enforce **context partitioning** by construction; ReAct loops
   leave it to the model.

A surprising number of "agent frameworks" are ReAct cores with a
state-machine skin painted on top. The skin is the config DSL; the
loop underneath is still "ask the model what to do next." If you
want the reproducibility properties of a state machine, you need
the loop itself to be one. The diagnostic for that is #1.

## Two operational rules I'm now using

### Rule 1: pick the shape that matches the task's repeatability profile

If you're going to run this once, ReAct is almost always cheaper to
write. If you're going to run this fifty times, on a schedule, with
SLOs, the state machine pays for itself by the third run. The break-even
is much lower than people think — somewhere around five repeated runs
in my experience, because the cost of a flaky ReAct loop is not just
"it failed" but "it failed in a way I have to debug from a transcript."

The mistake I see most often is reaching for ReAct because it's
faster to start, then re-running it on a cron, then drowning in
non-deterministic failure reports. The fix is not "make ReAct more
deterministic" — that doesn't exist as a thing. The fix is to port
the loop to a state machine the moment it goes on a schedule.

### Rule 2: route by step type, not by tool

Within a state machine, the lever that matters most is **which model
runs which step**. Planning steps benefit from frontier reasoning;
extraction steps benefit from cheap fast tokens; summarization steps
benefit from long-context cheap models. If your state machine pins
all steps to one model "for simplicity," you are leaving the main
operational benefit of the shape on the table. Mix providers per
step. Cache aggressively at step boundaries (state machine boundaries
are natural cache keys; ReAct loops have nothing to cache against).

This is the property `forge` is built around, and it's the property
most other state-machine-ish frameworks underuse. You can do it in
LangGraph too, you just have to ignore the default examples that pin
one model to the whole graph.

## Why the catalog buries the choice

A confession. The reason this distinction is invisible in most CLI
documentation is that the choice is downstream of a more fundamental
one — **who is the planner?** — and that choice is uncomfortable to
state out loud. ReAct CLIs are betting the model is good enough to
plan. State-machine CLIs are betting the model is _not_ good enough
to plan, or that the cost of the model planning badly is too high to
accept on this task. Both bets are sometimes right. Neither bet ages
well as a marketing claim, because frontier model capability moves
under your feet every six months.

So the documentation talks about features instead. Skills, hooks,
sub-agents, sandboxes, workflows, agents, graphs, gates. Each feature
is a way of pulling control from one side or the other without
admitting which side you're on. The honest version of every CLI's
README would open with one sentence: "We bet the model is / is not
the right thing to plan with, and here is what we built to make
that bet payable."

## What I would do differently

I'd read every new agent CLI's main loop function before its README.
The loop tells you the shape; the README tells you the marketing.
The shape is what you have to live with when the agent fails at 3am.

## Links

- ai-cli-zoo `forge` entry: <https://github.com/Bojun-Vvibe/ai-cli-zoo/tree/main/clis/forge>
- ai-cli-zoo `claude-code` entry: <https://github.com/Bojun-Vvibe/ai-cli-zoo/tree/main/clis/claude-code>
- ai-cli-zoo `opencode` entry: <https://github.com/Bojun-Vvibe/ai-cli-zoo/tree/main/clis/opencode>
- ai-cli-zoo `openhands` entry: <https://github.com/Bojun-Vvibe/ai-cli-zoo/tree/main/clis/openhands>
- Original ReAct paper (Yao et al., 2022): <https://arxiv.org/abs/2210.03629>
- LangGraph (state-machine framework for LLM workflows): <https://github.com/langchain-ai/langgraph>
