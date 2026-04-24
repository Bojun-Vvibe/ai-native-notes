---
title: "Why agent timeouts should be budget-shaped, not duration-shaped"
date: 2026-04-24
tags: [agents, reliability, timeouts, design]
---

The first timeout most people put around an agent loop looks like this:

```python
result = run_agent(task, timeout_seconds=600)
```

Ten minutes. Reasonable-feeling. It survives the first week of shipping the
agent. Then a long-context summarization task arrives, the model spends 540
seconds streaming, the post-processing tool call takes 80, and the whole loop
is killed at second 600 with a half-written artifact, no commit, and no log
line that explains what was actually consumed. The user retries. The retry
also dies at 600 seconds, because of course it does — the work is genuinely
that big, and the timeout was a property of the wall clock, not of the work.

This is a duration-shaped timeout. It treats time as the unit of permission.
A budget-shaped timeout treats *resource consumption* as the unit, lets the
agent know how much of each resource remains, and refuses to start subtasks
it cannot afford. The difference is small in code and large in behaviour.

## What "budget-shaped" actually means

A budget is a structured allowance. At minimum it has:

- **Wall-clock seconds remaining** — the hard ceiling, but not the only one.
- **Tokens remaining** (input + output, possibly priced separately).
- **Tool-call slots remaining**, optionally per-tool.
- **Dollar cents remaining**, derived from the above and the price book.
- **Retries remaining**, separate from raw call count.

The agent is given the budget at the start of the run and must consult it
before each non-trivial action. The budget object is the only authority on
"do I have permission to do this next thing?" — neither the wall clock alone
nor a token counter alone is sufficient.

```python
@dataclass
class Budget:
    deadline_mono: float        # time.monotonic() target
    tokens_remaining: int
    tool_calls_remaining: dict[str, int]   # per-tool
    cents_remaining: int
    retries_remaining: int

    def can_afford(self, cost: "Cost") -> bool:
        if time.monotonic() + cost.expected_seconds > self.deadline_mono:
            return False
        if cost.tokens > self.tokens_remaining:
            return False
        if cost.cents > self.cents_remaining:
            return False
        slot = self.tool_calls_remaining.get(cost.tool, 0)
        if cost.tool and slot <= 0:
            return False
        return True
```

The crucial verb is `can_afford`. Duration-shaped timeouts only ever ask
"have I died yet?" — a budget asks "would the next step kill me?" before
taking it.

## Why duration alone is the wrong knob

Three failure modes show up reliably with pure-duration timeouts.

### 1. The amputated commit

A 10-minute timeout on a workflow whose last 30 seconds are "write the file,
git add, git commit" produces partial state on every overrun. The artifact
exists but is unsigned, unwitnessed, and not in the index. A budget-shaped
timeout reserves a tail allowance — say, 45 seconds — that the loop is not
allowed to spend on anything except finalisation. When `time_remaining()`
crosses that line, the planner is forced into a "wrap up now" branch.

```python
TAIL_RESERVE = 45.0

def time_remaining(b: Budget) -> float:
    return max(0.0, b.deadline_mono - time.monotonic())

def must_finalize(b: Budget) -> bool:
    return time_remaining(b) <= TAIL_RESERVE
```

The planner reads `must_finalize(b)` at every loop iteration and switches
prompt mode. This is the single highest-value change you can make to a
duration-shaped agent: even without changing the deadline, partial-write
incidents drop sharply.

### 2. The token blowup that wasn't a timeout

A model goes off the rails generating a 90KB JSON blob. The wall clock
shows 73 seconds elapsed. The duration timeout is 600. Nothing fires. The
streaming finishes, the next tool call attaches the blob to its prompt, and
the next call costs $4.20 in input tokens. By the time anyone notices, the
budget is gone but the deadline isn't. A budget-shaped timeout would have
killed the runaway generation at the token cap, not the second cap.

The asymmetry matters: tokens are cheap to count and the only honest
estimate of "how much work the model thinks it has left." Wall-clock seconds
say nothing about whether the model has decided to enumerate every file in
`node_modules`.

### 3. The retry that costs more than the original

A tool call fails. The agent retries. The retry is a fresh subprocess with
its own duration timeout but no awareness of the parent's spent budget. Run
this loop three times and the user is billed for four attempts at the
nominal cost of one. Per-tool retry slots in the budget object make this
impossible: each retry decrements `retries_remaining`, and when the slot
hits zero the loop has to either escalate or give up.

## A small state machine

The minimum loop with a budget looks like this:

```
            +----------------+
            |   plan-step    |
            +--------+-------+
                     |
        budget.can_afford(next)?
            /                 \
          yes                  no
           |                    |
           v                    v
     +-----+------+      +------+------+
     | tool-call  |      | finalize    |
     +-----+------+      +------+------+
           |                    |
     deduct cost           write artifact
           |                    |
           v                    v
     +-----+------+      +------+------+
     | observe    |      |    end      |
     +-----+------+      +-------------+
           |
           v
     must_finalize(b)? --yes--> finalize
           |
           no
           v
       plan-step
```

There are exactly two terminal transitions: budget-exhausted-graceful and
deadline-tail-graceful. There is no "killed by timer" path, because the
timer never gets to fire — the budget refuses the step that would have
overrun it.

## What a sensible budget looks like in practice

For a coding agent invoked over an SSH session, recent runs cluster like:

| percentile | seconds | input tokens | output tokens | tool calls |
|-----------:|--------:|-------------:|--------------:|-----------:|
| p50        |     94  |       12,300 |         4,100 |          7 |
| p90        |    340  |       58,000 |        18,000 |         24 |
| p99        |    870  |      210,000 |        62,000 |         71 |
| max seen   |   1620  |      480,000 |       148,000 |        163 |

A duration timeout at 600 seconds cuts off something like 4% of legitimate
runs. A budget that allows `1200s / 300k input tokens / 80k output tokens /
60 tool calls` and refuses to start a step that would exceed any of them
gives you a much sharper boundary: the runs that hit the wall now have a
clear reason in the log ("refused tool-call: would exceed token cap by
~22k") rather than a `SIGTERM`.

## Per-tool slots earn their keep

Global retry counts hide pathologies. If your budget is "20 retries total"
and one flaky tool eats 19 of them on its own, the other 14 tools each get
one shot — and a different transient blip kills the run. Per-tool slots
look like:

```python
tool_calls_remaining = {
    "read":   60,
    "write":  20,
    "bash":   15,
    "grep":   40,
    "fetch":   8,    # network, expensive
    "exec":    5,    # privileged, hard cap
}
```

The numbers are not magic. They come from looking at the per-tool
distribution from your last week of runs and picking p95 plus one. Tools
with low caps force the planner to think before invoking them; tools with
high caps stay invisible. Either way, the cap is observable and the kill
reason is specific.

## Surfacing the budget to the model

A budget you don't tell the model about is a budget the model will overrun.
A clean pattern is to inject a compact budget line into every system turn:

```
[budget] 142s / 38k tokens / 11 tool calls / $0.21 remaining; tail reserve in 97s
```

This costs about 25 tokens per turn. In exchange, the model spontaneously
shortens responses, drops speculative tool calls, and starts producing
"finalising now because tail reserve is approaching" lines on its own.
Surfacing the budget to the model is not a substitute for refusing
unaffordable steps — the model will still misjudge — but it raises the
floor on planning quality noticeably.

## What changes in the failure log

A duration-timeout failure log looks like:

```
2026-04-24T11:02:18Z agent.run task=summarize-pr-3140
2026-04-24T11:12:18Z agent.run KILLED reason=timeout duration=600s
```

That is two lines and zero diagnostic value. A budget-timeout failure log
looks like:

```
2026-04-24T11:02:18Z agent.run task=summarize-pr-3140 budget=600s/200k/40
2026-04-24T11:08:41Z agent.refuse tool=fetch reason=cents_cap remaining=3
2026-04-24T11:08:41Z agent.replan from=fetch_more to=summarize_with_known
2026-04-24T11:09:55Z agent.finalize reason=tail_reserve_45s
2026-04-24T11:10:12Z agent.commit sha=abc1234 files=1
```

You can read the second log and tell the user "your task ran out of network
budget; raise the cents cap or narrow the scope." You cannot do that with
the first log; you can only say "it timed out, try again."

## Composition: budgets are values, not globals

Resist the urge to make the budget a process-global. Pass it explicitly into
sub-agents, and let sub-agents return a *spent* report so the parent can
decide whether to fund another sub-agent or finalise. A sub-agent that does
not know its own budget will either be over-cautious (waste it) or
over-eager (blow it).

```python
def parent(task, b: Budget):
    sub_b = b.allocate(seconds=120, tokens=30_000, cents=5, retries=2)
    spent, result = sub_agent(task.subtask, sub_b)
    b.charge(spent)
    if not b.can_afford(Cost.next_step()):
        return finalize(b, partial=result)
    ...
```

`allocate` is a pure split: it deducts from the parent and returns a child
budget that cannot exceed what was given. `charge` is reconciliation: the
sub-agent reports actual spend, which may be less than allocated; the
remainder returns to the parent. This is the same shape as a transaction
log, and it composes cleanly across as many levels as you care to nest.

## Anti-patterns to refuse

A few patterns look like budgets but aren't:

1. **A single number called `budget` that's actually just seconds.** This
   is a duration timeout in a fancy hat. If the only thing you can spend it
   on is wall-clock, it is wall-clock.
2. **Budgets the agent can refill itself.** "Increase your own token cap by
   10% if you think you need it" is how runs cost ten dollars instead of
   ten cents. Refills come from the parent only.
3. **Hidden budgets enforced only at the SDK layer.** If the loop refuses
   the step but the SDK silently retries inside the call, you are paying
   for invisible work. The budget must be the outermost authority.
4. **Budgets that don't include a tail reserve.** Without one, every
   exhaustion produces a partial write, and your incident reports fill up
   with "the run almost finished."

## Migration path

If you have a duration-timeout agent today, the migration is gradual and
each step is independently valuable:

1. Add a `Budget` object that contains only `deadline_mono`. Replace
   `time.time() > deadline` checks with `time_remaining(b)`. No behaviour
   change yet, but the code is now budget-shaped.
2. Add `tail_reserve` and the `must_finalize` branch. Partial-write
   incidents drop the next day.
3. Add token tracking. Wire the streaming callback to decrement
   `tokens_remaining`. Make the planner refuse a generation that would
   exceed the cap.
4. Add per-tool slots. Pull the initial caps from the last week's p95.
5. Add `cents_remaining` derived from the price book. Now the user can ask
   "what did this cost?" and the answer is in the log.
6. Surface a compact budget line to the model in every system turn.
7. Pass budgets explicitly into sub-agents; require them to return spend
   reports.

By step 4 you will already have stopped killing runs at the deadline; by
step 6 the model is collaborating with the budget; by step 7 you have a
composable resource discipline that scales to nested orchestration.

The takeaway is small and worth repeating: time is one resource among
several, and it is rarely the one your agent actually runs out of first.
Shape the timeout around what is actually being spent.
