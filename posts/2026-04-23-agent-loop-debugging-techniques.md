---
title: "Agent loop debugging: turning the black box into a tape"
date: 2026-04-23
tags: [agents, debugging, observability, llm, tracing]
est_reading_time: 15 min
---

## TL;DR

When an agent loop misbehaves, the bug is almost never in the model. It is in the message list you handed the model, in the order tool results came back, in a silently-truncated tool output, or in a JSON parse that the SDK swallowed. The single most useful debugging move is to dump every turn of the loop to a single newline-delimited file and replay it. After that, the next most useful move is a diff between two adjacent turns. This post documents the seven debugging techniques I rely on for stuck, looping, or just confused agents, with runnable code that drops into any agent built on top of a chat-completions style API.

The mental shift: stop thinking of the agent as a process and start thinking of it as a tape. A tape you can rewind, slice, and replay deterministically up to the model call itself. Once the loop is a tape, debugging is the same kind of work as debugging any other event-sourced system.

## Why "just add a print statement" does not work

Agent loops have three properties that defeat normal debugging.

First, they are non-deterministic at the model boundary. The same input produces different output across calls, even at temperature zero, because tokenizers, server-side load balancing, and version drift add noise. You cannot reliably reproduce a failure by re-running the loop, only by replaying the exact byte sequence into the model.

Second, the interesting state is enormous. A tool-using agent at turn 30 has a messages list that may be 50 to 100 KB of mixed text and JSON. Printing it every turn gives you a wall of text that no human reads.

Third, the failure modes are emergent. The agent does not crash. It loops. It calls the same tool with the same arguments four times in a row. It silently abandons the user's original goal halfway through. None of these show up as exceptions.

The techniques below are organized to address all three.

## Section 1: the turn-tape, the single most useful artifact

Before any other technique, write every turn of the loop to a JSONL file. One line per "thing the loop did." That includes the model request, the model response, every tool call, and every tool result.

```python
import json, time, uuid
from pathlib import Path

class TurnTape:
    def __init__(self, run_id: str | None = None):
        self.run_id = run_id or uuid.uuid4().hex[:8]
        self.path = Path.home() / ".cache" / "agent-tapes" / f"{self.run_id}.jsonl"
        self.path.parent.mkdir(parents=True, exist_ok=True)
        self.turn = 0

    def write(self, kind: str, payload: dict):
        self.turn += 1
        rec = {
            "ts": time.time(),
            "turn": self.turn,
            "kind": kind,
            "payload": payload,
        }
        with self.path.open("a") as f:
            f.write(json.dumps(rec, default=str) + "\n")

# usage in the loop
tape = TurnTape()
tape.write("model_request", {"messages": messages, "model": MODEL})
resp = client.messages.create(model=MODEL, messages=messages, ...)
tape.write("model_response", {"content": resp.content, "usage": resp.usage.model_dump()})
for tc in resp.tool_calls:
    tape.write("tool_call", {"name": tc.name, "args": tc.input})
    result = tools[tc.name](**tc.input)
    tape.write("tool_result", {"name": tc.name, "result": result})
```

Two rules I now treat as non-negotiable: never truncate inside the writer (truncate at read time if you must), and never let the writer raise. A debugger that crashes the process it is debugging is worse than no debugger.

The tape file is the source of truth for everything that follows.

## Section 2: replay, the second most useful artifact

Once you have a tape, you can re-run the loop deterministically up to the model boundary. The point of replay is not to reproduce the model's output, that is impossible. The point is to reproduce everything else, so you can change one thing (the system prompt, the tool schema, the next user message) and see how the rest of the run reacts.

```python
import json
from pathlib import Path

def replay(run_id: str, up_to_turn: int):
    path = Path.home() / ".cache" / "agent-tapes" / f"{run_id}.jsonl"
    state = {"messages": [], "tools": []}
    for line in path.read_text().splitlines():
        rec = json.loads(line)
        if rec["turn"] > up_to_turn:
            break
        if rec["kind"] == "model_request":
            state["messages"] = rec["payload"]["messages"]
        elif rec["kind"] == "tool_result":
            state["messages"].append(
                {"role": "tool", "name": rec["payload"]["name"], "content": rec["payload"]["result"]}
            )
    return state
```

I use this constantly. "What did the messages list look like at turn 17, just before the agent went off the rails?" Two seconds with `replay(run_id, 17)`.

## Section 3: the turn-to-turn diff

The third artifact: a unified diff between adjacent turns of the messages list. Most agent bugs show up as "the messages list grew in a way I did not expect." The diff makes that visible.

```python
import difflib, json
from pathlib import Path

def turn_diff(run_id: str, turn_a: int, turn_b: int):
    path = Path.home() / ".cache" / "agent-tapes" / f"{run_id}.jsonl"
    requests = {}
    for line in path.read_text().splitlines():
        rec = json.loads(line)
        if rec["kind"] == "model_request":
            requests[rec["turn"]] = json.dumps(rec["payload"]["messages"], indent=2, default=str)
    a = requests[turn_a].splitlines(keepends=True)
    b = requests[turn_b].splitlines(keepends=True)
    return "".join(difflib.unified_diff(a, b, fromfile=f"turn-{turn_a}", tofile=f"turn-{turn_b}", n=2))

print(turn_diff("a1b2c3d4", 16, 17))
```

The first time I ran this against a real run, I found that my tool result for `read_file` was being JSON-encoded twice, so the model was seeing `"\"file contents\""` instead of `"file contents"`. The diff showed the extra quoting immediately. I had been staring at that bug for two days.

## Section 4: detecting the same-call-same-args loop

The most common pathological loop: the agent calls the same tool with the same arguments multiple turns in a row, expecting a different result. You can detect this in 10 lines.

```python
def detect_repetition(messages, window=3):
    # collect last N tool calls
    calls = []
    for m in messages:
        for c in (m.get("content") or []):
            if isinstance(c, dict) and c.get("type") == "tool_use":
                calls.append((c["name"], json.dumps(c["input"], sort_keys=True)))
    if len(calls) < window:
        return None
    last = calls[-window:]
    if all(c == last[0] for c in last):
        return last[0]  # the offending call
    return None
```

In my agent harness, I call this after every model response. If it returns non-`None`, I inject a synthetic system message before the next call: `"You have called {name} with the same arguments {window} times. The result will not change. Try a different approach or ask the user."` The agent recovers about 70 percent of the time. The other 30 percent is usually a sign the user's request is impossible with the available tools, which is also useful information.

## Section 5: the tool-call budget and the budget violation log

A loop that runs forever is a loop that ran out of budget without anyone noticing. Set a per-session budget for tool calls, model calls, and tokens. Log every violation with full context, then halt.

```python
class Budget:
    def __init__(self, max_tool_calls=50, max_model_calls=20, max_input_tokens=200_000):
        self.max_tool_calls = max_tool_calls
        self.max_model_calls = max_model_calls
        self.max_input_tokens = max_input_tokens
        self.tool_calls = 0
        self.model_calls = 0
        self.input_tokens = 0

    def charge(self, *, tool=0, model=0, in_tok=0, tape=None):
        self.tool_calls += tool
        self.model_calls += model
        self.input_tokens += in_tok
        for k, cur, lim in [
            ("tool", self.tool_calls, self.max_tool_calls),
            ("model", self.model_calls, self.max_model_calls),
            ("in_tok", self.input_tokens, self.max_input_tokens),
        ]:
            if cur > lim:
                if tape:
                    tape.write("budget_exceeded", {"resource": k, "current": cur, "limit": lim})
                raise BudgetExceeded(f"{k}: {cur} > {lim}")

class BudgetExceeded(RuntimeError): pass
```

The budget is not just a safety net. It is a forcing function for asking "is this loop converging?" If the budget runs out, that is the loop telling you it does not know how to finish. Treat that as a planning bug, not as a need for a bigger budget.

## Section 6: structured logging of model decisions

A useful intermediate signal between "tape line" and "human-readable trace" is a one-line summary per turn. I emit one of these to stderr in addition to the tape.

```python
def turn_summary(turn: int, resp) -> str:
    text_blocks = sum(1 for b in resp.content if b.type == "text")
    tool_blocks = [b for b in resp.content if b.type == "tool_use"]
    tool_names = ",".join(b.name for b in tool_blocks) or "-"
    text_chars = sum(len(b.text) for b in resp.content if b.type == "text")
    return (
        f"t{turn:03d} "
        f"in={resp.usage.input_tokens:>5} "
        f"out={resp.usage.output_tokens:>4} "
        f"text={text_chars:>4}c "
        f"tools={tool_names}"
    )

import sys
sys.stderr.write(turn_summary(turn, resp) + "\n")
```

Sample output from a healthy run:

```
t001 in=12480 out= 240 text= 180c tools=read_file
t002 in=12892 out=  84 text=   0c tools=grep,read_file
t003 in=14210 out= 312 text= 250c tools=-
t004 in=14210 out= 180 text= 180c tools=-
```

Sample output from a stuck run:

```
t011 in=18204 out= 60 text=  0c tools=read_file
t012 in=18412 out= 60 text=  0c tools=read_file
t013 in=18620 out= 60 text=  0c tools=read_file
t014 in=18828 out= 60 text=  0c tools=read_file
```

You see the loop in three lines.

## Section 7: comparing two runs with `diff -u`

Sometimes the question is not "what happened in this run" but "why does this run behave differently from yesterday's good run." If both runs have tapes, you can extract a flat sequence of `(kind, key)` events and diff them.

```python
def event_sequence(run_id: str):
    out = []
    path = Path.home() / ".cache" / "agent-tapes" / f"{run_id}.jsonl"
    for line in path.read_text().splitlines():
        rec = json.loads(line)
        k = rec["kind"]
        if k == "tool_call":
            out.append(f"{k}:{rec['payload']['name']}")
        elif k == "model_response":
            tools = rec["payload"].get("content", [])
            names = [b.get("name") for b in tools if isinstance(b, dict) and b.get("type") == "tool_use"]
            out.append(f"{k}:{','.join(names) or 'text'}")
        else:
            out.append(k)
    return out

# write to files, then `diff -u good.txt bad.txt`
Path("good.txt").write_text("\n".join(event_sequence("a1b2c3d4")))
Path("bad.txt").write_text("\n".join(event_sequence("e5f6g7h8")))
```

This catches divergences like "the bad run took an extra `search` tool call between turn 4 and turn 5." Often that one extra tool call is the bug.

## Section 8: the post-mortem template I now use

After every loop that misbehaved badly enough to debug, I fill out a five-line template before closing the terminal. This forces me to convert ad-hoc debugging into a repeatable check.

```
run_id: a1b2c3d4
symptom: agent called read_file 6 times on the same path
first divergent turn: 11
root cause: tool result returned "" instead of file contents because the path had a trailing newline
fix: strip whitespace in the path normalizer
```

I keep these in a single `~/.cache/agent-tapes/postmortems.md` file. After about 30 entries, patterns become obvious. Three of the seven techniques in this post came directly from re-reading those entries and noticing I was running the same investigation by hand for the third time.

## Section 9: what I do not bother debugging

A short anti-list, for honesty.

I do not debug "the model gave a worse answer than yesterday" without a tape. The model is non-deterministic; without a byte-for-byte reproduction, you are debugging vibes.

I do not debug a loop that ran past budget without first asking "should this task even be possible in this many calls?" Half the time, the answer is no, and the right fix is to decompose the task or hand it back to a human.

I do not debug single failures. If a behavior happens once and never again, I log it and move on. The tape is the artifact that lets me come back to it later if it recurs.

The debugging techniques in this post are not clever. They are the tracing and replay patterns that distributed-systems engineers have used for two decades, applied to a new kind of system. The novelty is in what to trace (the message list, the tool call sequence, the budget) and what kinds of failures are common (silent looping, double-encoding, dict-order drift). The mechanics are old, and old mechanics are the ones that work under pressure.

## References

- Anthropic messages API reference: https://docs.anthropic.com/en/api/messages
- OpenAI chat completions reference: https://platform.openai.com/docs/api-reference/chat
- Event sourcing pattern (Fowler): https://martinfowler.com/eaaDev/EventSourcing.html
- JSONL spec: https://jsonlines.org/
- Python `difflib` docs: https://docs.python.org/3/library/difflib.html
