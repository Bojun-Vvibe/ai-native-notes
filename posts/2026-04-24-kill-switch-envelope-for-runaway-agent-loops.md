# Designing a Kill-Switch Envelope for Runaway Agent Loops

**Thesis:** Every agent loop you ship to production needs a kill-switch envelope — a small, opinionated layer of static and dynamic budgets that wraps the loop and forcibly terminates it when it crosses the line from "working hard" to "running away." Most teams don't have one. They have a `max_iterations = 50` constant somewhere and call it a day. That's not a kill switch; that's a polite suggestion. A real kill-switch envelope has multiple independent budgets, fails closed, leaves a forensic trail, and is invoked by a watcher process that the agent itself cannot disable. This post walks through the failure modes a kill switch needs to catch, the seven budget dimensions that together form an envelope, and a worked design for the watcher process.

## What "runaway" actually looks like

Before designing the envelope, it helps to enumerate the failure modes. In rough order of how often I've seen them in production agent traces:

**1. The same-tool-call loop.** The agent calls `read_file("README.md")`, gets the content, decides it needs to read it again, calls it again, gets the same content, decides it needs to read it again. This can run for hundreds of iterations on a single tool with no progress. Token usage climbs linearly. Wall time climbs linearly. The agent is "doing work" by every local definition.

**2. The two-step ping-pong.** Agent calls tool A, gets a result it doesn't like, calls tool B to fix something, the fix invalidates A's result, agent re-calls A, result still wrong, calls B again. Period-2 oscillation. The same two tool calls alternate forever. This is harder to spot than the same-call loop because the trace looks like it's making decisions.

**3. The exploding fan-out.** Agent calls a tool that returns a list of 200 items, then iterates and calls a per-item tool 200 times, and one of those returns another list of 50 items, and so on. No single iteration is wrong; the cumulative blast radius is. By the time you notice, you've made 12,000 tool calls and burned through your daily quota.

**4. The slow leak.** Each iteration adds a small amount of context to the running summary, the running summary feeds back into the next prompt, the next prompt is a tiny bit longer, and 200 iterations later you're sending a 180k-token prompt for a question that started at 4k tokens. The agent isn't *wrong*, it's just monotonically getting more expensive.

**5. The retry storm.** The model returns an invalid tool call schema, the harness retries with a "your last call was malformed" reminder, the model returns the same invalid call, repeat. Or: the tool returns 503, harness retries, tool returns 503, repeat. With no exponential backoff or attempt cap, this can melt a quota in minutes.

**6. The deadlock-on-cleanup.** Agent receives a stop signal, attempts a graceful shutdown, the shutdown calls a tool, the tool fails, the failure handler calls another tool, that tool waits for input that will never come. The agent is technically not running the main loop anymore, but the process is alive and burning resources.

**7. The plan-and-never-execute.** Agent enters a planning phase, writes a 5-step plan, decides the plan needs revision, writes a new 7-step plan, decides *that* needs revision. Pure thinking, zero side effects. This one is sneaky because most "is the agent making progress?" heuristics look at tool calls or file changes, and there are none.

A kill switch that only catches one or two of these is not a kill switch. It's a partial workaround. The envelope needs to handle all seven.

## The seven budget dimensions

Each failure mode above maps to a budget. None of them is sufficient alone. All of them together form the envelope.

### 1. Wall-clock budget

Hard cap on elapsed real time from turn start. The least controversial budget; everyone has one. Typical values:

- Interactive turn: 5 minutes
- Batch turn: 30 minutes
- Long-running mission: 4 hours, with a heartbeat requirement (see #6)

This catches most cases of "the agent is alive but going nowhere." It does not catch fan-out within the budget, retries that complete fast, or expensive-but-quick prompt growth.

### 2. Token budget (input + output, cumulative)

Hard cap on total tokens consumed by the turn across all model calls. This is your spend control. A reasonable interactive cap is 200k–500k tokens; for long missions, 5M–20M. The number is less important than the existence of the cap and the fact that it's monitored *cumulatively*, not per-call.

This catches the slow leak (#4), the exploding fan-out (#3) when fan-out is driven by model calls, and bounds the damage from any other runaway. It does not catch loops that are cheap per iteration — a same-call loop that reuses cached prompts can run for thousands of iterations without spending much.

### 3. Tool-call count budget

Hard cap on total tool calls per turn. Typical: 50 for interactive, 500 for batch, 5000 for long missions. Combined with #2 and #1, this catches most fan-out and most loops.

### 4. Per-tool repetition budget

Hard cap on calls to *the same tool with the same arguments* within a turn. This is the specific catch for the same-tool-call loop (#1). Implementation: hash the (tool_name, normalized_args) tuple; if the count for any hash exceeds N (say, 5), trip the switch.

"Normalized args" is doing a lot of work in that sentence. You probably want to canonicalize whitespace, sort dict keys, strip volatile fields like timestamps. You probably do not want to hash file contents that the tool returned — that's dynamic, not part of the call.

### 5. n-gram repetition budget on the tool-call sequence

Hard cap on how often any short sequence of tool calls repeats. This catches ping-pong (#2) and longer-period oscillations.

Concretely: maintain a rolling list of the last K tool-call hashes (say K=20). After each new call, check whether the last 2-, 3-, and 4-call suffixes have appeared earlier in the rolling window. If any n-gram has appeared more than M times (say M=4), trip the switch.

This is a coarse heuristic — it has false positives on legitimate repetitive workloads like "for each file in this list, run the linter." Mitigate by exempting tools whose semantics are explicitly idempotent-iteration (e.g., a `for_each` tool) or by widening M for those tools.

### 6. Heartbeat / progress budget

For long-running missions, require the agent to produce a *progress event* every N minutes. A progress event is a structured emission like `{"event": "progress", "summary": "completed step 3 of 7", "next": "...”}`. If no progress event arrives within N minutes, trip the switch.

This catches the deadlock-on-cleanup (#6), the plan-and-never-execute (#7), and any other case where wall time accumulates without forward motion.

The reason this is a *separate* budget from #1 is that wall-clock alone gives you a single timeout — once it fires, the turn is over. Heartbeat lets you have long missions that *would* be valid (a 4-hour evaluation run) but trips early when those missions get stuck (no progress event for 15 minutes).

### 7. Retry budget per logical operation

Hard cap on retry attempts per logical operation. Apply this at *two* levels:

- Per tool call: max 3 attempts, with exponential backoff.
- Per turn: max R total retries across all tool calls and all model calls (say R=10).

This catches the retry storm (#5). The per-turn cap is the important one — without it, even a sane per-call cap can be defeated by a sequence of distinct calls all retrying once each.

## Failing closed

When any budget trips, the watcher must:

1. **Send the stop signal first.** Do not wait for cleanup. Send it.
2. **Record the trip event** with: which budget, the value at trip time, the budget threshold, the last 20 events in the turn's event log, and a copy of the final pending tool call (if any).
3. **Wait a short grace period** (5–15 seconds) for the agent to acknowledge and flush.
4. **Force-terminate** if the grace period expires. SIGTERM, then SIGKILL.
5. **Mark the turn as `killed:<budget_name>`** in the persistence layer. This is critical for downstream metrics — you want to be able to query "how often is the wall-clock budget tripping vs the token budget" without parsing logs.

"Failing closed" specifically means: if the watcher itself fails (crashes, can't reach the persistence layer, can't read its own config), the *default* is to kill the agent, not to let it run. An agent without a watcher should not exist. The kill switch must not become a single point of failure that can be defeated by being broken.

## Why the watcher must be out-of-process

The most common implementation of a kill switch is `if iter > MAX: break` inside the agent loop itself. This does not work as a kill switch for two reasons:

**1. The agent can be wrong about whether it's looping.** If the loop is happening because of a bug in the loop's accounting code, the same bug will affect the budget check. I have personally shipped an "infinite loop" that was caused by an off-by-one in the same iteration counter that was supposed to terminate it.

**2. The agent can be stuck in a way that prevents the check from running.** If iteration N hangs inside a tool call, the check at the top of iteration N+1 never runs. The whole point of a kill switch is to handle the case where the agent isn't behaving normally, and "the agent's own code reliably runs at every iteration" is a normality assumption.

The watcher must be a separate process (or at minimum a separate thread on a real OS thread, not a coroutine on the same event loop) that monitors the agent's external behavior — its tool-call stream, its token usage as reported by the proxy, its heartbeat events — and can issue a stop signal regardless of what the agent is doing internally.

## A worked watcher design

Assume the agent emits events to a JSONL file as it runs (this is the same event stream you'd want for observability anyway). The watcher is a small process that tails this file and maintains the budget state.

```python
import json, os, signal, time
from collections import deque, Counter

class Envelope:
    def __init__(self, turn_id, pid, config):
        self.turn_id = turn_id
        self.pid = pid
        self.start = time.time()
        self.tokens = 0
        self.tool_calls = 0
        self.retries = 0
        self.last_progress = self.start
        self.call_hashes = deque(maxlen=20)
        self.call_counts = Counter()
        self.cfg = config

    def update(self, event):
        e = event["event"]
        if e == "model_response":
            self.tokens += event["input_tokens"] + event["output_tokens"]
        elif e == "tool_start":
            self.tool_calls += 1
            h = hash_call(event["name"], event["args"])
            self.call_hashes.append(h)
            self.call_counts[h] += 1
        elif e == "retry":
            self.retries += 1
        elif e == "progress":
            self.last_progress = time.time()

    def check(self):
        now = time.time()
        if now - self.start > self.cfg["wall_s"]:
            return "wall_clock"
        if self.tokens > self.cfg["max_tokens"]:
            return "token_budget"
        if self.tool_calls > self.cfg["max_tool_calls"]:
            return "tool_call_count"
        if any(c > self.cfg["max_per_tool"] for c in self.call_counts.values()):
            return "per_tool_repetition"
        if ngram_repeats(self.call_hashes, n=3) > self.cfg["max_ngram"]:
            return "ngram_oscillation"
        if now - self.last_progress > self.cfg["heartbeat_s"]:
            return "heartbeat"
        if self.retries > self.cfg["max_retries"]:
            return "retry_storm"
        return None

    def kill(self, reason):
        record_trip(self.turn_id, reason, self.snapshot())
        os.kill(self.pid, signal.SIGTERM)
        time.sleep(self.cfg["grace_s"])
        try:
            os.kill(self.pid, 0)  # check still alive
            os.kill(self.pid, signal.SIGKILL)
        except ProcessLookupError:
            pass
```

The main loop of the watcher:

```python
def watch(envelope, event_path):
    with open(event_path) as f:
        f.seek(0, 2)  # seek to end
        while True:
            line = f.readline()
            if line:
                envelope.update(json.loads(line))
            reason = envelope.check()
            if reason:
                envelope.kill(reason)
                return
            time.sleep(0.5)
```

This is a sketch — production code needs error handling, proper file rotation handling, and a way to be told the turn ended cleanly so it can shut itself down. But the shape is right: a small loop, simple state, decisions based on observable behavior.

## Tuning the budgets

Concrete starting values for an interactive coding agent, based on what I've found works:

| Budget | Interactive | Batch | Long mission |
|--------|-------------|-------|--------------|
| Wall clock | 5 min | 30 min | 4 hr |
| Cumulative tokens | 200k | 2M | 20M |
| Tool call count | 50 | 500 | 5000 |
| Per-tool repetition | 5 | 10 | 20 |
| n-gram (n=3) repeats | 4 | 8 | 16 |
| Heartbeat interval | n/a | 5 min | 15 min |
| Retry budget | 10 | 30 | 100 |
| Grace period | 10s | 15s | 30s |

These are starting points, not gospel. Tune them by collecting trip-rate data: if `wall_clock` is tripping 5% of the time and most of those trips are *legitimate long turns being killed*, raise it. If `per_tool_repetition` is tripping 0.1% of the time and every trip is a real loop, leave it. If a budget never trips, it's still useful — it caps your blast radius — but you might be able to lower it.

## What you get from having this

The first time you ship a kill-switch envelope, you'll see two things in your trip-rate data:

**You were running away more than you thought.** Every team I've worked with that adopted a real envelope discovered they had at least one budget that tripped on 1–3% of turns. That's 1–3% of turns that *would have* burned through quota, hung indefinitely, or ping-ponged forever, and that you were previously catching only by user complaints (or by the fact that you had a daily spend cap that papered over the symptom).

**Your incident MTTR drops.** When something is wrong with the agent — bad prompt, model regression, tool returning malformed data — the kill switch trips before the user notices. The trip event itself is your alert. You'd rather page on "5x increase in `ngram_oscillation` trips in the last 10 minutes" than on "user reports the agent is slow."

The kill-switch envelope is not optional infrastructure for an agent in production. It's the difference between a system that fails gracefully under conditions you didn't anticipate and one that fails catastrophically. The small amount of work to build the watcher and define the budgets pays for itself the first time it saves you from a runaway loop that would otherwise have consumed your quota, your reputation, or both.
