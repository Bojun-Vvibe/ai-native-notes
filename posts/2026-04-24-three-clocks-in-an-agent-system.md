---
title: "The Three Clocks in an Agent System: Wall, Monotonic, and Budget"
date: 2026-04-24
---

There are three clocks running inside any non-trivial AI agent system, and confusing them is the source of an embarrassingly large fraction of the bugs I find in production agent code. The three clocks are:

1. **Wall-clock time** — what `datetime.now()` returns. Calendar time. The thing humans care about.
2. **Monotonic time** — what `time.monotonic()` returns. A counter that only goes forward, immune to NTP adjustments and DST shifts. The thing your latency measurements care about.
3. **Budget time** — the agent's *internal* notion of how much of its allotted compute, tokens, money, or wall-time it has spent so far. The thing your kill-switch and your timeouts and your retry budgets care about.

These three clocks measure different things, drift relative to each other, and have different failure modes. If you reach for `time.time()` every time you need "the current time" inside an agent loop, you will, eventually, ship a bug. Probably several. This post is about which clock to use for what, and the specific failure shapes I've seen when people pick wrong.

## Why agents need three clocks

The intuition for why a simple program needs only one clock is that "now" is not interesting in a simple program. You measure how long a function took, you log a timestamp, you check whether a deadline has passed. The same `time.time()` call can do all three reasonably well in code that runs for milliseconds and finishes.

Agents do not finish in milliseconds. A single agent turn can run for 4 minutes. A long-running session can run for hours. Background sub-agents can run for days. During those durations, four things become true that are not true in normal code:

1. **The wall clock will jump.** NTP corrections, leap seconds, DST transitions, virtualization timer drift — over the span of an agent's runtime, wall-clock time will move non-monotonically. If you measure latency as `t_end_wall - t_start_wall` you will eventually see negative latencies. (I have a Slack screenshot of one of mine.)

2. **The monotonic clock will not match user expectation.** When an agent reports "this took 3.2 seconds," the user wants wall-clock seconds, not monotonic-counter seconds. They are usually the same. Over long durations across NTP corrections, they are not.

3. **Compute budget is not time.** A turn that hits the token limit at second 2 has exhausted its budget despite having 178 wall-clock seconds remaining in its deadline. A turn that runs for 5 wall-clock minutes but spent 4.5 of those minutes blocked on a slow tool has used very little compute. Budget is its own currency and you cannot derive it from a clock.

4. **The agent reasons about time.** The model will see timestamps in its context. If your wall-clock and monotonic-clock measurements disagree, the model will see contradictions and either hallucinate explanations or refuse to act.

These four facts together force the three-clock distinction. You cannot collapse them into one without producing wrong behaviour somewhere.

## Clock 1: Wall-clock time

Wall-clock time is for two things and two things only in an agent system: **(a) anything the user will read**, and **(b) anything that interoperates with external systems that have their own calendars** (databases, log aggregators, cron schedules, calendars, audit trails).

The failure shape when you use wall-clock time for anything else is "negative durations and inverted ordering." A common version: you record `start_time = datetime.now()` at the beginning of a tool call, then `end_time = datetime.now()` at the end, then store `duration = (end_time - start_time).total_seconds()` in your latency telemetry. NTP slews the clock backwards by 200ms during the call. Your latency record says the tool call took -180ms. Your p95 chart breaks because some downstream aggregator chokes on negatives. You spend an afternoon trying to figure out which timezone is wrong.

The fix: never derive a *duration* from two wall-clock samples. Wall-clock is for *instants you will display or persist*, not for *intervals you will measure*.

Concretely, in Python:

```python
# WRONG
start = datetime.now()
result = call_tool()
duration = (datetime.now() - start).total_seconds()  # can be negative

# RIGHT
start_wall = datetime.now(timezone.utc)  # for the audit log
start_mono = time.monotonic()             # for the duration measurement
result = call_tool()
duration = time.monotonic() - start_mono  # always non-negative
audit_log.write(started_at=start_wall, duration_s=duration, ...)
```

The audit log gets a wall-clock timestamp because humans and external systems will read it. The duration is computed from monotonic time because it must be non-negative and accurate. These are different concerns and they need different clocks.

## Clock 2: Monotonic time

Monotonic time is for **measurements**: latency, throughput, intervals between events, "has 30 seconds passed since the last token," "how long has this tool call been running." Anything where the answer must be a non-negative number that accurately reflects elapsed physical time.

The failure shape when you use *anything else* for measurements is the inverse of the wall-clock failure: instead of negative durations, you get *wrong* durations. Classic example: an agent loop that uses `time.time()` to enforce a 60-second turn deadline. The system clock gets bumped forward 5 seconds during the turn (NTP correction after a long suspend). The deadline appears to have passed when it has not. The turn is killed at second 55 of real elapsed time. The user sees "request timed out" with no explanation.

Or the inverse: clock gets bumped backwards. The deadline never appears to pass. The turn runs for 65 seconds before some other timeout fires. Your "60-second deadline" telemetry shows you respected the deadline because you computed `elapsed = time.time() - start_time = 58s`, but in fact you ran for 65 wall seconds. Your p95 chart is lying to you about a different metric this time.

Monotonic time fixes both. It is, by definition, the right tool for measuring elapsed real time. The only gotchas:

- Monotonic time has an arbitrary zero. You cannot compare monotonic timestamps across processes, across machines, or across reboots. If you need cross-process latency measurement (distributed tracing) you have to either use wall-clock anchors at the boundaries or use a clock-synchronization protocol designed for it.
- Monotonic time does not advance during deep system suspend on some platforms. If your agent is running on a laptop that closes its lid, the monotonic clock may not move while suspended. This is usually what you want for latency measurement (the user wasn't waiting), but it is not what you want for deadlines (the deadline did pass in wall-clock terms).

The general rule: monotonic for *durations*, wall-clock for *instants*.

## Clock 3: Budget time

Budget time is the one most people don't realize they need until they've shipped a few production bugs. It is the agent's internal notion of how much of its allotted resource it has consumed.

"Budget" can be measured in several units depending on what you are budgeting:

- **Token budget**: total input + output tokens consumed in the turn or session.
- **Money budget**: dollar cost of API calls so far.
- **Tool-call budget**: number of tool invocations remaining before the kill switch fires.
- **Wall-time budget**: deadline for the turn or session.
- **Compute budget**: aggregate seconds of subprocess CPU time consumed.

Each of these is its own clock with its own current value, its own ceiling, its own decay or refresh policy. They are independent. A turn can be at 2% of token budget but 80% of wall-time budget because it spent most of its time blocked on a slow API. Or vice versa.

The failure shape when you don't track budget time as a distinct clock is what I call the "budget-shaped timeout" failure: you set a single timeout (wall-clock seconds) and use it for everything. You get false positives (turns killed for taking too long when they had plenty of compute left) and false negatives (turns that consumed 50,000 tokens in 4 wall-clock seconds and blew past your token budget without tripping any timeout).

The fix is to track budgets explicitly and check them at every loop boundary:

```python
class TurnBudget:
    def __init__(self, max_tokens: int, max_calls: int, deadline_mono: float):
        self.max_tokens = max_tokens
        self.max_calls = max_calls
        self.deadline_mono = deadline_mono
        self.tokens_used = 0
        self.calls_made = 0

    def check(self) -> Optional[str]:
        if self.tokens_used >= self.max_tokens:
            return "token_budget_exhausted"
        if self.calls_made >= self.max_calls:
            return "call_budget_exhausted"
        if time.monotonic() >= self.deadline_mono:
            return "wall_deadline_exceeded"
        return None

    def consume(self, tokens: int = 0, calls: int = 0):
        self.tokens_used += tokens
        self.calls_made += calls
```

Note that the wall-time deadline is stored as a *monotonic* timestamp, not a wall-clock one. The deadline is "60 seconds of elapsed real time from now," which is a duration computation, which means monotonic. If you store the deadline as a wall-clock timestamp, you reintroduce the NTP-correction failure mode.

The budget object is checked at every iteration of the agent loop, before every tool call, before every model call. The check returns a structured reason, not a boolean, because the reason matters: a token-budget exhaustion can be handled by truncating the conversation, a call-budget exhaustion can be handled by blocking further tool calls but allowing one final summary, a deadline exhaustion has to terminate immediately. Different failure modes, different recovery strategies.

## The interactions between the three clocks

The interesting bugs live in the *interactions*. Three patterns I've hit:

### Pattern 1: Wall-clock in the prompt

You include the current wall-clock time in the model's system prompt: "The current time is 2026-04-24 14:32 UTC." The agent runs for 3 minutes. The model emits text that says "as of just now, 14:32 UTC...", but in fact it is now 14:35. The user reads the response 30 seconds later and the time is wrong by minutes.

The fix is to be honest about what "current time" means in the prompt. Either you stamp it as "as of the start of this turn, the time was X" (which is true) or you give the model a tool it can call to get the current time (which costs a tool call but is accurate). Most frameworks pick the first because it's cheap.

### Pattern 2: Monotonic across processes

Your agent spawns a subprocess for a long-running tool. The parent records `start_mono = time.monotonic()`. The subprocess records its own `start_mono` in its own monotonic clock. You try to compute end-to-end latency by subtracting them. The numbers are nonsense because the two monotonic clocks have unrelated zero points.

The fix is to anchor cross-process measurements with wall-clock timestamps at the boundaries (start and end), and use monotonic only for *intra-process* duration measurements. Distributed tracing systems exist for exactly this reason.

### Pattern 3: Budget that doesn't refresh

You set a per-turn token budget of 50,000 tokens. The agent does 30 turns in a session. By turn 30, the model is responding "I don't have budget to do that" because the budget object was never reset between turns. The user is confused because they have not been told that there is a per-turn limit; they think the agent is just being stubborn.

The fix is to make budget lifetimes explicit and *visible*. A turn budget resets at the start of each turn. A session budget accumulates across turns. A daily budget accumulates across sessions. The agent should know which budget it is operating under and surface that to the user when it hits a limit. "Token budget for this turn exhausted (50,000/50,000)" is debuggable. "I cannot continue" is not.

## What to actually do

The discipline I've converged on, after enough wrong-clock bugs:

1. **Wrap all three clocks in named functions**. `now_wall()`, `now_mono()`, `now_budget(name)`. Never call `time.time()` directly. The named functions force you to think about which clock you mean every time you ask for "now."

2. **Type-distinguish the timestamps**. A `WallTime` is a different type from a `MonoTime` is a different type from a `BudgetSpent`. The type checker will catch most of the wrong-clock bugs at code-review time instead of at 3am.

3. **Store deadlines as monotonic timestamps**. A deadline is a duration from a starting point; computing it requires monotonic time. Storing wall-clock deadlines is the source of the NTP-correction class of bugs.

4. **Surface the active budget to the user and the model**. When a turn is killed by a budget, the kill reason should include which budget and how much was consumed. This is the only way to debug agent termination after the fact.

5. **Test clock skew explicitly**. Inject artificial NTP corrections in your test harness. Verify that durations are non-negative, deadlines fire correctly, and audit timestamps are monotonic-ish. Most test suites never exercise the case where the wall clock moves backwards. That is precisely the case that fails in production.

6. **Log all three clocks at incident boundaries**. When something goes wrong and you have to debug it, you want to know what wall-clock time it happened, how long it took (monotonic), and how much budget had been consumed. Three numbers, three different answers, all important.

The reason the three-clocks discipline matters is not that any one of these bugs is catastrophic. It is that they are *systematic*: every time you reach for `time.time()` without thinking about which clock you actually mean, you are taking on a small probability of a wrong-clock bug, and an agent system makes that bet thousands of times per session. The expected number of bugs per session is small. The expected number of bugs per quarter is not.

The fix is cheap. It is a couple of helper functions, a couple of types, and a code-review discipline of "which clock did you mean?" That is all. But it has to be applied universally, because the moment you have a single un-typed `time.time()` call in a hot loop, the bugs come back, and they come back in production, and they come back in the form of latency charts that lie about what they are measuring. Three clocks. Wall, monotonic, budget. Use the right one every time.
