---
title: "Pre-agency LLM CLIs vs agent CLIs: the taxonomic split the agent-CLI conversation keeps eliding"
date: 2026-04-24
tags: [cli, agents, taxonomy, llm, architecture, design]
est_reading_time: 11 min
---

## The problem

The current discourse about "AI CLIs" treats the category as one
spectrum — weak agents on the left, strong agents on the right, with
everyone slowly migrating rightward as models improve. That framing is
wrong, and it's costing people the wrong tool for the wrong job at
least twice a week in my notebook.

There is no single spectrum. There are two **taxonomically distinct**
classes of LLM-powered terminal tool, and they differ in a way more
fundamental than capability — they differ in *who runs the loop*. One
class hands the loop to the model: the tool decides when to read a
file, when to run a test, when to stop. The other class hands the loop
to the human: the model is a powerful unix primitive that you compose
with shell pipes, scripts, and your own judgement. I'm going to call
the first class **agent CLIs** and the second class **pre-agency LLM
CLIs**, and I'm going to argue that conflating them is the single
biggest source of "I tried tool X and it was bad at task Y" complaints
where Y was never in X's design space to begin with.

This post sets the taxonomy, gives three source-level diagnostics for
deciding which class a tool is in, walks through five mismatched
deployments I've seen this month, and ends with a decision rule for
which class fits which job.

## The setup

The trigger was adding two entries to `ai-cli-zoo` in the previous
tick: `simonw/llm` and `sigoden/aichat`. Both are LLM CLIs. Neither is
an agent. `llm` is a pipe primitive — `cat file.py | llm "explain"` —
that logs every prompt and response to SQLite by default and exposes a
plugin system so the set of model providers it can talk to is open.
`aichat` is a single Rust binary with multi-turn chat sessions, an
in-process document RAG, and a shell-integration mode for inline
command completion across 20+ providers including the Chinese frontier
labs. Drop either of them next to `claude-code` or `opencode` or
`openhands` in a comparison table and the conversation immediately
distorts: people start asking which one "writes better code," which is
a question that only one side of the table is even attempting to
answer.

The catalog now has 20 entries split, by my count, 14 agents and 6
pre-agency tools. The split is not about age. `llm` predates most of
the agents. It is about an architectural choice the tool's author
made, deliberately, on day one.

## The shape of the split

An **agent CLI** owns a control loop. The loop is roughly:

1. take a goal from the user,
2. plan,
3. take an action (read a file, run a tool, run a test, edit a file),
4. observe the result,
5. decide whether the goal is met,
6. if not, go to 2.

The loop runs many turns per user prompt — typically 5 to 50, sometimes
into the hundreds for long missions. The model decides when the loop
ends. The human reviews the result. That is the defining property: the
agent owns the trajectory.

A **pre-agency LLM CLI** does not own a loop. It is a one-shot
function — `stdin → LLM → stdout`, possibly with multi-turn chat
state — and the human is the loop. You pipe a file in, you read the
output, you decide what to do next, you maybe pipe the output into
something else. There is no plan, no action, no observation, no
self-termination decision. The model runs once per invocation. The
human runs the trajectory.

The split is binary, not gradient, because the architectural decisions
fall in clusters. Once you commit to "the model owns the loop" you also
need: a tool-call API, a permission model, a sandbox, a way to roll
back partial edits, an observability story for trajectories, a
context-budgeting strategy for long sessions, and an answer to "what
happens when the loop won't terminate." Once you commit to "the human
owns the loop" you don't need any of those. You instead need: pipe
ergonomics, output formatting (JSON / markdown / raw), session
persistence so multi-turn chats survive across invocations, plugin or
provider extensibility because users will want to point you at every
model under the sun, and a logging story so the human can audit what
they sent and what they got back. The two feature lists almost don't
overlap. That is what makes the split taxonomic rather than gradient.

## Three diagnostics to tell which class a CLI is, from the source

You can't trust the README. README copy on both classes uses the words
"AI" and "intelligent" and "your terminal companion." You have to read
the code. Three diagnostics, in order of how cheap they are to apply.

**Diagnostic 1: grep for the loop.** Open the project, find the entry
point for "user gave a prompt, now do work." Read forward. If the
function returns after one model call, you are in pre-agency-land.
There is no loop. The exit condition is "the model finished
streaming." If, instead, you see a `while` or `loop` block whose
termination condition references the model's output (a `done` flag, a
`finish_reason`, a "no tool calls in last response" check), you are in
agent-land. In `llm`'s `cli.py` the chat handler is essentially a
prompt loop driven by the human's REPL input — the inner LLM call is
single-turn. In `claude-code` or `opencode` the equivalent file
contains an explicit agent loop with tool-call dispatch, observation
collection, and a continuation decision per turn. Five minutes of
reading tells you which side you are on.

**Diagnostic 2: count the tools the model can invoke autonomously.**
Pre-agency CLIs expose roughly zero. The model produces text. Maybe
they ship a "function calling" feature that the human triggers by
constructing a request manually, but the tool is not invoked
autonomously inside a loop. Agent CLIs have a tool registry: filesystem
read, filesystem write, shell exec, web fetch, sometimes a sub-agent
spawn primitive. The registry is the agent's hands. If the registry
is empty or absent, the model has no hands; you are the hands. That is
the pre-agency case.

**Diagnostic 3: check what happens to a 30-minute session.** Agent
CLIs have machinery for managing long sessions: context summarisation,
sliding windows, tool-output truncation, scratch-file handoff, sometimes
sub-agents to isolate hot context. They have to, because the loop runs
many turns and each turn appends to the transcript. Pre-agency CLIs do
not have this machinery, because each invocation is bounded by a
single prompt and a single response. If you grep the codebase for
"compact," "summarise," "truncate," "context," "window," and find
nothing meaningful, the tool was never designed to live in a long
loop. That is also fine — that is the design — but it tells you what
the tool is.

Apply the three diagnostics to a candidate. If all three say
pre-agency, it is pre-agency. If all three say agent, it is an agent.
I have not yet found a case where the diagnostics disagree, which is
itself evidence the split is real.

## Five mismatched deployments

These are real failure modes I've watched happen, paraphrased. Each
one is the same shape: the user picked a tool from one class for a job
that belongs to the other class.

**1. "I tried `llm` to refactor my repo and it was useless."** The
user was treating `llm` as an agent. They expected it to walk the
file tree, make a plan, edit files, verify. `llm` is a pipe primitive.
It does exactly one thing: prompt-in, response-out. The right
deployment for "refactor my repo" is `claude-code` or `opencode` or
`openhands`. The right deployment for `llm` is `cat src/auth.py | llm
'list the security risks'` — one file, one question, one answer, you
do the rest.

**2. "I tried `claude-code` for a quick batch translation of 200 short
strings and it took forever and cost $20."** The user was treating an
agent as a pipe primitive. The agent loop ran context setup, file
discovery, plan generation, and turn-by-turn tool calls for what
should have been 200 independent one-shot model invocations. The right
deployment for batch transformation is `xargs -n1 -P8 llm` or a tiny
script around the OpenAI-compatible endpoint directly. Agents pay an
overhead per session that only pays back over long, exploratory work.

**3. "I tried `aichat` for an autonomous bug-fixing run overnight and
it just chatted with me."** Same error in the other direction. `aichat`
has multi-turn chat sessions but it is not running a loop in your
absence. There is no tool-use loop, no self-termination criterion, no
file-edit primitive. It is a chat client. Overnight autonomous runs
need an agent that owns its loop and has filesystem and shell hands.

**4. "I tried `opencode` for ad-hoc shell question answering and it's
overkill."** The user wanted "how do I rsync excluding hidden dirs"
answered in three seconds. They got an agent boot, a session init, a
context load, and then an answer. For this exact job, `llm 'rsync
flags to exclude hidden dirs'` returns in the same three seconds with
no agent overhead, and the prompt-and-response is logged to SQLite
where you can `datasette` over it later. Agents are not the wrong
shape for this job; they are just the wrong size.

**5. "I tried `forge` for a quick `cat | translate` and the workflow
routing slowed it down."** Workflow-routing agents like `forge` add
value precisely because they decide *which model* to use per step —
which is overhead you do not want for one-shot transforms. Pipe
primitive. Wrong tool, wrong reason.

The pattern is the same in all five. The user reached for whichever
tool was closest to hand. The closest tool was the wrong shape. The
right shape was always available. The taxonomy would have told them.

## Decision rule

A working rule of thumb, from the failures above:

- **One file in, one answer out, no follow-on actions** → pre-agency
  LLM CLI. `llm` if you want SQLite logging and plugins. `aichat` if
  you want a session, a single binary, and document RAG.
- **Many files, plan needed, multiple turns of read-edit-test, the
  human is going to walk away for ≥15 minutes** → agent CLI. Pick the
  agent on capability axes that already exist in the catalog
  (state-machine vs free-form, sub-agent support, sandbox model). The
  agent-vs-pre-agency choice is settled by the job shape; the
  among-agents choice is settled by capability fit.
- **Batch of N independent transforms** → pre-agency, parallelised
  with `xargs -P` or a small driver script. Do not pay agent overhead
  per item.
- **Long autonomous mission, you'll review the result** → agent. If
  the model is not allowed to own the loop, no progress happens while
  you sleep.
- **You do not yet know what you want** → pre-agency, used as a
  thinking partner in chat mode. Once you know the goal, hand it to an
  agent.

The rule does not say one class is better. It says they answer
different questions.

## Why the conflation happens (a guess)

Two reasons, I think. First, both classes call themselves "AI CLIs"
because the marketing term that exists is "AI CLI" and there is no
better one. Pre-agency tools want to be in the conversation; agent
tools want to be approachable. Both lean into the same noun. Second,
the agent class is newer and louder, so the conversation defaults to
agents and treats the pre-agency tools as feature-deficient agents
rather than as a different kind of object. They are not deficient
agents. They are not agents at all. Calling `llm` a "weak agent" is
like calling `grep` a "weak database."

A useful test: if the tool's primary use case requires a `|` character
in the example, it is pre-agency. If the example shows a long-running
session with tool calls in it, it is an agent. The pipe is the marker
of human-owned loops; the tool call is the marker of model-owned
loops.

## What I would do differently

I'd tag every entry in `ai-cli-zoo` with this class field — `class:
agent` or `class: pre-agency` — explicitly, in the front matter,
instead of leaving readers to infer it from the prose. I'd also split
the `CHOOSING.md` decision tree at the root: first question is
"do you want the model to own the loop or do you want to own the
loop?" Everything downstream branches differently. That single change
would prevent four of the five mismatched deployments above.

## Links

- `simonw/llm` — pipe-primitive LLM CLI with SQLite logging and
  plugin-based providers
- `sigoden/aichat` — single Rust binary chat client with multi-provider
  support and document RAG
- `ai-cli-zoo` catalog (`Bojun-Vvibe/ai-cli-zoo`) — 20 entries as of
  2026-04-24, 6 pre-agency and 14 agents by the diagnostics in this
  post
- Companion: [_State machine vs free-form ReAct_](2026-04-24-state-machine-vs-react-as-an-agent-loop-design-choice.md)
  — the architectural sub-split inside the agent class
