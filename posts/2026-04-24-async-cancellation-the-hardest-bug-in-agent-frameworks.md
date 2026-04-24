---
title: "Async Cancellation Is the Hardest Bug in Agent Frameworks"
date: 2026-04-24
---

I have spent more time chasing bugs caused by async cancellation in AI agent frameworks than I have spent chasing prompt regressions, schema drift, and tokenizer mismatches *combined*. That ratio surprised me when I noticed it. Cancellation is supposed to be a solved problem — every async runtime has a story about it, every job queue has a story about it, every HTTP client has a story about it. But agents stitch all those layers together and then hand the steering wheel to a non-deterministic model that decides, mid-tool-call, that maybe we should have asked a different question. The result is that cancellation in an agent framework is not one bug. It is a *family* of bugs, and the family is unusually nasty because the failures are silent, partial, and almost always reproducible only under load.

This post is what I wish someone had told me six months ago.

## The shape of the problem

A user-facing agent has, at minimum, three concurrent things going on at any moment:

1. The model stream — tokens flowing in from the provider.
2. One or more in-flight tool calls — HTTP requests, subprocesses, database queries, embedding lookups.
3. The user's session — which can disconnect, time out, or click "stop" at any instant.

When the session goes away, you have to cancel (1) and (2). Easy to say. Hard to do correctly. The reason it is hard is that "cancel" means very different things at each layer, and the agent framework is the only place where those meanings have to be reconciled into a single coherent story.

Consider a single example. The user clicks stop while the agent is in the middle of calling a `search_codebase` tool. The tool is a subprocess that has shelled out to ripgrep over a 200k-file repo. The subprocess has been running for 4 seconds. The model stream is paused waiting for the tool result. What does "cancel" mean here?

- Kill the ripgrep process? Send SIGTERM, then SIGKILL?
- Wait for ripgrep to finish and discard the result?
- Wait for ripgrep to finish, return the result to the model, but discard the model's next response?
- Mark the tool call as failed and resume the agent on a new turn?
- Mark the tool call as never-happened and let the model retry on next user message?

Every one of those choices is *defensible*. Every one of them produces a different downstream invariant. And if you don't pick one and enforce it everywhere, you get a framework where cancellation does different things depending on which tool was in flight, which is a debugging nightmare because the user's bug report is always "it just got stuck" and never "the partial-effect cleanup semantics for my SIGTERM-resistant subprocess were inconsistent with the model stream's lifecycle."

## Why agent cancellation is harder than HTTP cancellation

In a normal HTTP server, cancellation has a tidy story: the client closes the socket, the server notices, the request handler gets a context-cancelled error on its next await, and any in-flight work cooperatively unwinds. The contract is "every blocking call honours the context, and partial side effects are the handler's problem to clean up." This works because most handlers do bounded amounts of work and most side effects are inside transactions.

Agents break every assumption in that paragraph.

First, agents do *unbounded* work. A single user turn can trigger 30 tool calls over 4 minutes. The cancellation window is enormous, and the cost of mishandling cancellation grows with the duration.

Second, agent side effects are not transactional. If the agent has already called `write_file`, `git_commit`, and `post_to_slack`, you cannot roll those back when the user clicks stop. The framework has to decide: do you finish the turn so the model can at least observe what it did, or do you abort and leave the model with no memory of the partial side effects? Both are wrong. The first wastes tokens on a turn the user no longer wants. The second leaves the next turn with a state-of-the-world that contradicts the conversation history.

Third, agent cancellation is *bidirectional*. The user can cancel the agent. But the agent can also cancel itself — kill-switch envelopes, budget exhaustion, repeated tool failures, model-emitted "stop" signals. And the *tool* can cancel its own work — a subprocess that times out, a streaming API that drops the connection. All three of these flow through the same cancellation channel, and the framework has to disambiguate them because the cleanup story differs.

Fourth, and this is the killer: the model itself is a participant in cancellation. If you cancel the tool call but let the model finish its current sentence, the model will emit text that *assumes the tool call succeeded*. You will end up with conversation transcripts that say "I searched the codebase and found 14 matches" when in fact the search was killed at second 3 and never returned anything. The transcript lies, and lying transcripts poison every downstream eval, every replay test, and every fine-tune dataset built from the logs.

## The four cancellation bugs I keep finding

After enough debugging sessions I started keeping a list. There are basically four shapes, and I find at least one of them in every agent framework I audit.

### Bug 1: The orphaned tool call

The user cancels. The framework cancels the model stream. The framework forgets to cancel the in-flight tool call. The tool call completes 30 seconds later and tries to write its result to a session that no longer exists. Depending on the framework, this either:

- Crashes with a "session closed" error in a background task that no one is watching.
- Silently writes the result to a now-orphaned session record that gets garbage-collected.
- Retries forever because the result-write itself is wrapped in retry logic that doesn't know about cancellation.

I have seen the third variant produce a slow leak of zombie tool-call results that filled up a Redis instance over six hours.

The fix is to make the tool call's lifetime *strictly nested inside* the session's lifetime. Every tool call gets a context derived from the session context. When the session cancels, every tool call cancels with it. This sounds obvious. It is not implemented in most frameworks I've read.

### Bug 2: The half-cancelled stream

The framework cancels the model stream by closing the SSE connection. But the SDK on the client side has buffered the last 40 tokens that arrived before the close. Those tokens get parsed into a final tool-call object and dispatched, *after* cancellation. Now you have a tool call that started running 200ms after the user clicked stop, with no record of why it started.

The fix is that cancellation must be enforced at the *dispatch* boundary, not at the *transport* boundary. Closing the socket is necessary but not sufficient. You also have to gate every "did the model emit a tool call" handler on a fresh check of the cancellation token. If cancelled, the buffered tokens get discarded, not dispatched.

### Bug 3: The cancellation race that double-cancels

The session times out at exactly the same moment the user clicks stop. Both produce cancellation signals. Both signals trigger the cleanup path. The cleanup path is not idempotent — it tries to `kill(pid)` a subprocess that is already dead, or it tries to close a database connection that is already closed, or it decrements a semaphore that is already at zero. The framework crashes mid-cleanup, leaving the session in a half-cleaned state.

The fix is to make every cleanup operation idempotent and to serialize cancellation through a single state machine that absorbs duplicate signals. "Cancelled" is a terminal state. Once you're in it, all further cancellation requests are no-ops.

### Bug 4: The phantom completion

The framework cancels everything correctly. But the result-aggregation layer that builds the final response has buffered partial results, and on cancellation it emits one last "completion" event with the partial buffer. The downstream UI receives a `completed` event and updates the conversation as if the turn finished normally. The user's stop-click is invisible.

The fix is that cancellation must produce a *cancelled* event, not a *completed* event, and the UI must distinguish the two. Sounds trivial. Half the agent frameworks I've looked at conflate them because the SDK's `Stream` abstraction has only one terminal state.

## A working model: cancellation as a typed lifecycle event

The pattern I've converged on, after enough failures, is to treat cancellation as a first-class lifecycle event with a *reason* and a *boundary*.

Every cancellable scope has:

- A **reason**: `user_stop`, `session_timeout`, `budget_exhausted`, `kill_switch`, `tool_failure_cascade`, `parent_cancelled`. The reason matters because the cleanup story differs. A `user_stop` should preserve partial state for replay. A `kill_switch` should burn everything. A `budget_exhausted` should checkpoint and resume.
- A **boundary**: the tightest scope that the cancellation applies to. Tool-call cancellation does not necessarily cancel the parent turn. Turn cancellation does not necessarily cancel the session. Session cancellation cancels everything. The boundary determines which contexts get propagated cancellation.
- A **terminal event**: every scope emits exactly one terminal event, and the type of that event encodes whether it completed, was cancelled, or errored. No conflation.

In code this looks something like:

```python
class CancellationScope:
    def __init__(self, parent: Optional["CancellationScope"]):
        self.parent = parent
        self.children: list["CancellationScope"] = []
        self._cancelled = False
        self._reason: Optional[str] = None
        if parent:
            parent.children.append(self)

    def cancel(self, reason: str) -> None:
        if self._cancelled:
            return  # idempotent
        self._cancelled = True
        self._reason = reason
        for child in self.children:
            child.cancel(f"parent_cancelled:{reason}")

    @property
    def cancelled(self) -> bool:
        return self._cancelled or (self.parent is not None and self.parent.cancelled)
```

Then every tool call, every model stream, every background task gets a scope derived from its parent. Cancellation propagates downward. The terminal event for each scope checks its own cancellation status and emits the right type.

This is not novel. Go's `context.Context` has had this for a decade. .NET's `CancellationToken` has had it longer. The novelty in agent frameworks is not the mechanism — it is the *discipline* required to thread the scope through every async operation, including the ones the model triggered, including the ones inside SDK callbacks you don't own.

## The partial-effect problem has no clean answer

Even with perfect propagation, you still have the partial-effect problem. The model called `write_file` and the file is on disk. You cancel. Now what?

I have settled on a pragmatic answer that is not satisfying but is honest: **every tool that has externally-visible side effects must record an audit entry before performing the effect, and the framework must surface those audit entries to the next turn**. So if the user cancels mid-`write_file`, the next turn's system prompt includes "the previous turn was cancelled. The following side effects had been recorded before cancellation: wrote /tmp/foo.py at 14:32:01." The model can then decide to revert, continue, or notify the user.

This is unsatisfying because it pushes the problem to the model, which is not a reliable executor of cleanup logic. But the alternative — building a transactional system around side effects that include `git push` and `slack post` and `subprocess spawn` — is impossible. So we surface the partial state honestly and trust the model to handle it, and we add evals for the partial-state-handling cases.

## What to actually do

If you are building or maintaining an agent framework, the things to do, in order:

1. **Audit your cancellation paths**. For every async operation in your codebase, ask: what cancels this? What does it do when cancelled? Does it emit a terminal event with the right type?

2. **Make cleanup idempotent**. Every kill, close, decrement, unlink — make it safe to call twice. Cancellation races are the most common source of "the framework crashed during cleanup" bugs.

3. **Distinguish terminal event types**. `completed`, `cancelled`, `errored` are different. Do not collapse them. Make the UI handle all three explicitly.

4. **Record partial side effects**. Before any externally-visible mutation, write an audit entry. Include those entries in the next turn's context if the previous turn was cancelled.

5. **Test cancellation under load**. The races only show up at concurrency. A test suite that cancels one request at a time will not catch any of the four bugs above. You need a fuzz test that cancels at random instants under realistic concurrency.

6. **Log the cancellation reason**. Every cancelled scope should log why it was cancelled, by whom, and what its boundary was. This is the only way to debug the bugs after the fact, because by definition cancellation destroys the state that would have explained the bug.

The reason async cancellation is the hardest bug in agent frameworks is not that any individual piece is hard. It is that getting all the pieces consistent across model streams, tool calls, sessions, and partial side effects requires sustained discipline that is easy to lose during a Friday-afternoon refactor. Every framework I've worked on has had at least one of the four bugs above sneak back in within a month of a cleanup pass. The only defense I know of is to make the cancellation lifecycle a typed, audited, tested first-class concern, and to refuse to land any new tool, any new transport, any new background task without a cancellation story.

That refusal is the single highest-leverage rule I've added to my agent-framework reviews. It catches more bugs than every other rule combined. And the reason is that nobody — not the model, not the SDK author, not the tool author — is going to think about cancellation if you let them not think about it. So you don't.
