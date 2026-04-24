# The second-turn problem: why agents handle turn N+1 worse than turn 1

**Date:** 2026-04-25
**Tags:** multi-turn, context, evaluation, agent-runtime, regressions

If you build agents long enough you will notice an asymmetry that
nobody warns you about. The model handles the first user request
of a session beautifully. It picks the right tool, fills in the
arguments correctly, formats the output cleanly, and returns
something that feels like the system is working. Then the user
says "now do the same thing but for the staging environment" and
the wheels start to wobble. By turn five you are watching it
re-introduce a bug it already fixed two turns ago. By turn ten
the conversation has accumulated enough cruft that it would
genuinely be faster to start a new session and re-paste the
context.

Every team I have talked to about this has independently noticed
it. Most have rationalised it as "context window pressure" or
"the model getting confused". The real story is more interesting
and points at fixable engineering problems rather than at the
model's intrinsic limits.

## The shape of the problem

A useful way to make this concrete: take any benchmark that
measures tool-use accuracy on single-turn requests, and re-run
it as a multi-turn session where each request is preceded by
two unrelated successful turns. Accuracy degrades. The amount of
degradation varies by model and task, but for tool-heavy tasks
the gap between turn 1 and turn 3 is consistently larger than
the gap between turn 3 and turn 10. The first additional turn
costs the most. Subsequent turns cost less, until the context
window starts filling up and a different failure mode kicks in.

This is not what you would expect from a "context pressure"
model of degradation. Context pressure should be roughly linear
in token count and only matter near the window edge. The early,
sharp drop has a different cause.

## Five mechanisms that produce second-turn degradation

**Mechanism 1: prior assistant turns become a stronger prior than
the system prompt.** The system prompt is one block of text at the
top of the conversation. By turn 3, the assistant has emitted
several blocks of text the model now treats as evidence of "what
this assistant does". If turn 1's assistant response was
slightly off-pattern from the system prompt's instructions —
maybe it skipped a suggested preamble, maybe it used a slightly
different tool-call style — that off-pattern behaviour now
competes with the system prompt as a behavioural anchor. By turn
5 it has won. The model is now imitating its own past output
rather than following the system instructions, and any small
error in turn 1 becomes the new house style.

**Mechanism 2: tool results crowd out instructions.** Every tool
call produces a tool result that gets appended to the
conversation. Tool results are often large — file contents,
search results, log excerpts — and they sit in the context
between the system prompt and the current user turn. The
attention pattern that worked for turn 1 (system prompt close
to the question) is now operating across thousands of tokens of
intervening tool output. The model can still attend to the
system prompt, but the relative weight of "the rules" versus
"the recent evidence" has shifted. Recent evidence wins.

**Mechanism 3: stale tool results get treated as current state.**
On turn 1 the agent runs `ls` and the result reflects reality.
On turn 4 the agent has created and deleted three files since
that `ls`, but the original `ls` output is still in the context
window. When the user asks a question whose answer depends on
the current filesystem, the model has two sources of truth: the
stale `ls` result and the more recent `write_file` and
`delete_file` tool calls. Sometimes it correctly synthesises the
current state; often it just trusts the most prominent
contiguous block of evidence, which is the stale `ls`. This is
the source of an entire class of bugs where an agent confidently
reports that a file exists that it deleted two turns ago.

**Mechanism 4: error recovery contaminates the example set.** If
turn 2 contained a tool call that failed and was retried
successfully, the failed call and its error message are still in
the context. The model now has an in-context example of "calling
this tool with these arguments produces this error". On turn 4
when it needs to call the same tool, it sometimes pre-emptively
modifies the arguments to avoid the error it saw — even when the
error was a transient network issue or an unrelated problem.
You see this most clearly when an agent starts adding defensive
extra arguments to tool calls that worked fine in earlier
sessions. It is pattern-matching on its own past failures.

**Mechanism 5: tokenizer drift across the conversation.** This
one is subtle. The chat template inserts sentinel tokens between
turns. Each turn's contribution to the context is the user
message, the assistant message, and the tool results — wrapped
in sentinels that the model treats as turn boundaries. The model
was trained on conversations where these boundaries appeared at
predictable intervals. Long sessions, especially ones with many
small tool results, produce sentinel patterns that are out of
distribution. The model's behaviour near these boundaries
becomes less reliable. You are unlikely to detect this by
reading transcripts; it shows up as a slight increase in
formatting errors and tool-call malformations as the session
ages.

## Why I believe this is real and not just my intuition

I keep a local corpus of agent sessions through `pew`. The most
recent `pew status` reports 3403 tracked files across
`claude-code` (1160), `codex` (478), `openclaw` (1530), and
`vscode-copilot` (235), with 1366 records pending upload as of
the 4/25/2026 02:20:11 sync. The interesting cut to make on
this corpus is "tool-call success rate as a function of
turn-index-within-session". The drop from turn 1 to turn 2 is
the single biggest jump on the curve. The rate keeps drifting
down through turn 5 or so, then stabilises until the session
gets close to the context window. This is in line with what I
have heard from other teams running similar telemetry on
production agents — the early-session degradation is the most
robust finding across runtimes, and it does not go away when
you switch model vendors.

## What actually helps

The naive fix — "just compact the conversation" — sometimes
helps and sometimes makes things worse, because compaction
changes the prior-distribution problem in unpredictable ways.
The interventions that work robustly are smaller and more
targeted.

**Re-anchor the system prompt.** On every turn, re-emit the most
critical system instructions immediately before the current user
question, not just at the top of the conversation. This is ugly
and feels redundant but it directly counteracts Mechanism 1 and
Mechanism 2. The cost is a few hundred tokens per turn; the
benefit is that the behavioural anchor stays close to the
question being answered. Several agent runtimes have moved
toward this pattern, sometimes calling it "instruction reminder"
or "rolling system prompt".

**Mark stale tool results explicitly.** When you append a new
tool result that supersedes an older one, annotate the older
one with a marker like `[superseded by tool call at turn N]`.
This gives the model an explicit signal to ignore the stale
content. It is more reliable than removing the old result
entirely, because removal also removes the model's ability to
reason about what it already tried. The marker preserves the
history while neutralising its evidentiary weight.

**Summarise tool results aggressively, but only after they
become old.** Recent tool results should stay verbatim because
the model needs to act on them. Tool results from more than two
or three turns ago should be summarised down to a single line
of "what was learned". This counters Mechanism 2 directly. The
hard engineering problem is the summariser: it has to be
deterministic enough that the summary is the same every time
the conversation is reloaded, or you introduce a different bug
where the summary itself drifts across replays.

**Quarantine error traces.** When a tool call fails and is
retried successfully, move the error trace into a structured
"errors-encountered" block that lives outside the message
stream, or wrap it in a marker the model is trained to treat as
"this happened but is no longer relevant". This counters
Mechanism 4. Some runtimes do this by emitting tool errors as
ephemeral messages that are stripped on the next turn; others
keep them but prefix them with a clear "RESOLVED" tag.

**Refresh the chat template assumption between turns.** This is
the boring one but it matters. If your runtime is templating the
conversation differently on turn 5 than it did on turn 1 —
because of any per-turn logic that affects how messages get
serialised — you are creating exactly the sentinel-pattern drift
that Mechanism 5 describes. The fix is to ensure the templater
is a pure function of the message list, with no per-turn state.

## When to give up and start a new session

There is a threshold past which no amount of in-conversation
intervention will rescue a session. The signs are recognisable:
the agent starts repeating itself, it confidently asserts things
that contradict earlier tool results, it pre-emptively adds
defensive arguments to working tool calls, or its formatting
starts to degrade. By the time three of these are present,
compaction will not help and re-anchoring will not help. Start
a new session, carry forward only the structured artifacts (file
contents, decisions made, open todos), and let the model see a
clean context.

The depressing implication is that "long-running agent" is in
some ways a misleading product category. What you actually want
is a series of short agent sessions whose state is carried
forward by an external system that you control. The agent's
context window is for working memory, not for storage. Treating
it as storage is what produces the second-turn problem in the
first place.

## The deeper question

Every mechanism in this list points at the same underlying
issue: the chat-conversation format conflates several things
that should be separate. It conflates the model's instructions
with its prior outputs. It conflates current state with
historical state. It conflates the data the model needs to act
on with the metadata about how it got there. A well-designed
agent runtime fights this conflation actively, on every turn.
A naive runtime accepts the format as given and reaps the
second-turn problem as its reward.

The good news is that the fixes are not exotic. They are
discipline about message construction, discipline about what
goes in the system prompt versus the user turn, and discipline
about treating the context window as a working buffer rather
than as a log. The teams that have these disciplines in place
ship agents that degrade gracefully across turns. The teams
that do not ship agents that look great in demos and feel
broken after the third question. The difference is not the
model. It is the runtime.
