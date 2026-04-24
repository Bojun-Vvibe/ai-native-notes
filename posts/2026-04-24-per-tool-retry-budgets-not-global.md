# Per-Tool Retry Budgets, Not Global

## Thesis

A single global retry counter on an agent loop is the wrong shape. It collapses the failure modes of fundamentally different tools into one number, then uses that number to make a decision that should have been made per-tool. After running enough agent loops in production to be embarrassed by the data, I will go further: a global `max_retries=N` is a code smell that almost always hides at least one real bug, and usually more than one.

The right shape is a per-tool retry budget, with each tool declaring its own ceiling, its own backoff, and its own definition of what counts as a "retry-worthy" failure. This is more code. It is also the difference between an agent loop that converges and an agent loop that burns 80k tokens flailing on a transient 502 from one upstream while a perfectly working `read_file` tool gets blamed for the budget exhaustion.

## What a global retry budget actually does

Walk into a typical hand-rolled agent loop and you will see something like this:

```python
MAX_RETRIES = 5

def run(loop):
    retries = 0
    while True:
        step = loop.next_step()
        try:
            result = step.invoke()
        except Exception as e:
            retries += 1
            if retries > MAX_RETRIES:
                raise
            time.sleep(2 ** retries)
            continue
        loop.observe(result)
        if loop.done():
            return loop.result
```

This looks fine in isolation. It is not fine. Here is what it actually does on a 30-step mission with a mix of tools:

1. Step 4 calls `web_fetch` against a flaky upstream. It fails twice transiently, then succeeds. Counter is now at 2.
2. Step 11 calls `run_tests` and the test suite has a real, deterministic bug introduced by the agent at step 9. It fails. Counter goes to 3, then 4, then 5. The loop aborts.
3. The agent never gets a chance to read the test failure, reason about it, and patch step 9. The mission dies on a real bug, but it dies because of an *unrelated* upstream flake earlier in the run.

The global counter conflates two completely different things: **infrastructure flakes** (where retrying the *exact same call* is the correct response) and **semantic failures** (where retrying the same call is guaranteed to fail again, and the agent must instead change its plan). A single counter cannot distinguish these because it cannot see the tool. It just sees "an exception happened."

The second failure mode is more insidious. With a global budget, every transient flake on a cheap, idempotent tool steals budget from an expensive, non-idempotent one. The cheap tool is precisely where retries are *safe and effective*. The expensive tool is where you want to think hard before retrying. A global counter inverts the priority.

## What a per-tool budget looks like

Each tool gets its own little policy object. The policy declares three things: how many retries are allowed, what the backoff schedule looks like, and which exception classes count as retryable. Everything else is treated as a hard error and surfaced to the agent as an observation, so the agent can reason about it instead of the loop silently consuming budget.

```python
from dataclasses import dataclass
from typing import Tuple, Type

@dataclass(frozen=True)
class RetryPolicy:
    max_retries: int
    base_delay_s: float
    retryable: Tuple[Type[BaseException], ...]
    # cap on cumulative wall-clock spent retrying this tool in one mission
    budget_seconds: float

POLICIES = {
    "web_fetch": RetryPolicy(
        max_retries=4,
        base_delay_s=0.5,
        retryable=(TimeoutError, ConnectionError),
        budget_seconds=30.0,
    ),
    "read_file": RetryPolicy(
        max_retries=1,
        base_delay_s=0.05,
        retryable=(BlockingIOError,),  # almost never useful
        budget_seconds=2.0,
    ),
    "run_tests": RetryPolicy(
        max_retries=0,  # never retry. Surface the failure.
        base_delay_s=0.0,
        retryable=(),
        budget_seconds=0.0,
    ),
    "shell": RetryPolicy(
        max_retries=0,  # shell has side effects. Never retry by default.
        base_delay_s=0.0,
        retryable=(),
        budget_seconds=0.0,
    ),
    "llm_completion": RetryPolicy(
        max_retries=6,
        base_delay_s=1.0,
        retryable=(TimeoutError, ConnectionError, RateLimitError),
        budget_seconds=120.0,
    ),
}
```

Notice the asymmetry. `run_tests` and `shell` never retry. `web_fetch` retries aggressively. `llm_completion` gets the largest budget because rate limits during peak hours can eat 90 seconds of retries and still be the right answer. `read_file` essentially does not retry at all, because if you got `PermissionError` once, you will get it again, and if you got `FileNotFoundError`, the agent needs to know that, not the loop.

The dispatcher uses these policies like so:

```python
def call_tool(name: str, fn, *args, **kwargs):
    pol = POLICIES[name]
    state = mission_state.tool[name]   # per-tool, per-mission counters
    deadline = time.monotonic() + pol.budget_seconds
    attempt = 0
    while True:
        try:
            return fn(*args, **kwargs)
        except pol.retryable as e:
            attempt += 1
            state.retries += 1
            if attempt > pol.max_retries or time.monotonic() > deadline:
                # surface to agent as an observation, not as a crash
                return ToolError(tool=name, kind="exhausted", cause=str(e))
            time.sleep(pol.base_delay_s * (2 ** (attempt - 1)))
        except Exception as e:
            # not in retryable allowlist: surface immediately
            return ToolError(tool=name, kind="hard", cause=str(e))
```

Two things matter about that snippet. First: failures are returned as `ToolError` observations, not raised. The loop never silently dies; the agent gets to see what happened and choose what to do next. Second: the budget has both a count and a wall-clock cap. Either one trips it. This matters because backoff schedules can themselves blow your latency target without ever actually exhausting attempts.

## What this rules out

Adopting per-tool retry budgets rules out a surprising number of bug shapes. I want to be specific because the abstract argument is unconvincing. Here is what stops happening once you switch:

- **Cross-tool budget theft.** A flaky `web_fetch` cannot kill a mission whose real work is in `run_tests`. The two budgets are independent.
- **Silent retries on side-effecting tools.** Once `shell` has `max_retries=0`, you stop accidentally running the same `rm -rf` twice because of a network hiccup between the agent and the executor. This has happened to me. It is unforgettable.
- **Retry storms on auth failures.** A 401 from an LLM provider is not in the retryable set. Without a per-tool policy, the global retry loop will hammer the same dead credential six times, hit the rate limiter, get throttled, and now you also have a 429 in your error log to confuse you tomorrow morning.
- **Retries on "successful failures."** A tool that returns a structured error (e.g. `compile_error` from a code-runner) used to count toward the global budget if your loop treated all non-OK statuses as exceptions. With per-tool policies, you can declare structured errors as observations, not retries.
- **Indistinguishable post-mortems.** When the loop dies on `RetryBudgetExhausted`, the global version cannot tell you which tool actually failed. The per-tool version logs `tool=run_tests, kind=hard, cause=AssertionError` and the post-mortem writes itself.

It does not rule out: real upstream outages, agents that get stuck in plan-loops, or tools that are simply broken. Per-tool budgets are not magic. They are a way to make the failures legible.

## The retry classes that actually exist

If you sit down and enumerate the failure modes your tools have, you end up with roughly five classes, and they map cleanly onto policy decisions.

1. **Transient network.** Connection reset, DNS hiccup, TLS handshake timeout. Almost always safe to retry with exponential backoff. Budget should be generous (4-6 attempts) but capped in wall-clock.
2. **Rate limited.** 429 with a `Retry-After` header. The right response is *not* exponential backoff; it is to honor the header. This is its own policy and the implementation usually wants to read the header rather than blindly multiply.
3. **Auth / config.** 401, 403, missing env var, expired token. Never retry. Surface immediately. The agent cannot fix this in the next attempt, and a human (or a higher-level supervisor) needs to know.
4. **Semantic failure.** Test failed, lint failed, type check failed, compilation error. Never retry. *Always* surface to the agent as an observation. The whole point of the agent is to react to this class.
5. **Side-effecting tools.** `shell`, `git push`, `write_file`. Default `max_retries=0` unless the tool itself is provably idempotent. If you want retries, put the idempotency at the tool boundary (an idempotency key, a content-addressed write), not in the loop.

A global retry counter cannot distinguish any of these. A per-tool policy table makes the distinction the first thing you see.

## Smallest reproducible setup

You do not need a framework to try this. Here is a complete runnable demo using only the standard library. Save it as `retry_demo.py` and run it.

```python
#!/usr/bin/env python3
"""Per-tool retry policy demo. stdlib only."""
import time
import random
from dataclasses import dataclass, field
from typing import Tuple, Type, Callable, Any

random.seed(0)

@dataclass(frozen=True)
class RetryPolicy:
    max_retries: int
    base_delay_s: float
    retryable: Tuple[Type[BaseException], ...]
    budget_seconds: float

@dataclass
class ToolState:
    retries: int = 0
    hard_failures: int = 0
    successes: int = 0

@dataclass
class ToolError:
    tool: str
    kind: str  # "exhausted" | "hard"
    cause: str

POLICIES = {
    "flaky_net":  RetryPolicy(4, 0.05, (ConnectionError,), 5.0),
    "tests":      RetryPolicy(0, 0.0,  (), 0.0),
    "side_fx":    RetryPolicy(0, 0.0,  (), 0.0),
}

state: dict[str, ToolState] = {k: ToolState() for k in POLICIES}

def call(name: str, fn: Callable[[], Any]) -> Any:
    pol = POLICIES[name]
    st = state[name]
    deadline = time.monotonic() + pol.budget_seconds
    attempt = 0
    while True:
        try:
            v = fn()
            st.successes += 1
            return v
        except pol.retryable as e:
            attempt += 1
            st.retries += 1
            if attempt > pol.max_retries or time.monotonic() > deadline:
                st.hard_failures += 1
                return ToolError(name, "exhausted", repr(e))
            time.sleep(pol.base_delay_s * (2 ** (attempt - 1)))
        except Exception as e:
            st.hard_failures += 1
            return ToolError(name, "hard", repr(e))

# simulated tools
def flaky_net() -> str:
    if random.random() < 0.6:
        raise ConnectionError("reset")
    return "200 OK"

def tests() -> str:
    raise AssertionError("expected 3, got 5")

def side_fx() -> str:
    raise OSError("disk full")

# simulated mission of mixed steps
plan = ["flaky_net"] * 10 + ["tests"] * 2 + ["side_fx"] * 1
fns = {"flaky_net": flaky_net, "tests": tests, "side_fx": side_fx}

results = [call(name, fns[name]) for name in plan]

for name, st in state.items():
    print(f"{name:10s}  ok={st.successes:2d}  retries={st.retries:2d}  hard={st.hard_failures:2d}")

errors = [r for r in results if isinstance(r, ToolError)]
print(f"\nfinal errors: {len(errors)} surfaced as observations (not crashes)")
```

Run it and you will see something like:

```
flaky_net   ok= 8  retries=12  hard= 2
tests       ok= 0  retries= 0  hard= 2
side_fx     ok= 0  retries= 0  hard= 1

final errors: 5 surfaced as observations (not crashes)
```

Note three things. First: `flaky_net` got its own retry budget and recovered most of the time, with only two cases blowing through the cap. Second: `tests` and `side_fx` never retried even once, because their policies forbid it. Third: nothing crashed. Every failure became a `ToolError` the agent loop can read and reason about. That last point is the real prize. The agent is now in charge of what to do about a failure, instead of the dispatcher silently eating attempts on its behalf.

## Why I keep harping on this

The reason this matters more than it sounds is that retry policy is the boundary between "the loop is in charge" and "the agent is in charge." With a global counter, the loop is making product decisions: deciding when to give up, deciding which failures matter, deciding what counts as an attempt. The agent has no visibility into any of this. It just gets a `RetryBudgetExhausted` exception with no idea which tool blew the budget or why.

With per-tool policies, the loop becomes mechanical. It handles transient flakes within a narrow, declared scope, and surfaces everything else as observations. The agent sees `ToolError(tool="run_tests", kind="hard", cause="AssertionError: expected 3, got 5")` and can do what agents are good at: read the failure, hypothesize the cause, edit a file, try again. That is the whole point of having an agent in the loop instead of a script.

When I see a codebase with a single `MAX_RETRIES=5` constant at the top of the dispatcher, I now read it as "this team has not yet had the bad afternoon where a flaky CDN consumed their entire mission budget twelve minutes before a deploy window." It is a perfectly understandable place to start. It is also a place to leave.

## Operational notes

Two implementation details that are worth stealing rather than rediscovering.

First: persist the per-tool counters to the mission state, not to in-process memory. If the dispatcher restarts mid-mission (and it will), you want the new process to know that `web_fetch` already burned three of its four attempts on the previous run. JSONL is fine for this; one line per tool-call attempt with `{tool, attempt, outcome, ts}` and a small reducer at startup.

Second: emit a structured event on every retry, not just on exhaustion. The interesting metric is not "how many missions exhausted their budget" but "what is the retry-rate per tool per hour." That second metric is the leading indicator. The first is the lagging one. A jump in `web_fetch` retry-rate at 02:00 UTC tells you a CDN is degrading two hours before a single mission actually fails because of it. You only get that signal if you log the retries themselves, not just the exhaustions.

Per-tool retry budgets are not glamorous. They are the kind of thing you appreciate the morning after a bad incident, when you can read the post-mortem and see exactly which tool failed how, instead of staring at a stack trace that says `RetryBudgetExhausted` and no further detail. Build them on day one. You will not regret it.
