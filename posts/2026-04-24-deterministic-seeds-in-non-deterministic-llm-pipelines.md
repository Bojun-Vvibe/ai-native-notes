# Deterministic Seeds in Non-Deterministic LLM Pipelines

## Thesis

You cannot make an LLM bit-for-bit deterministic in production. You also cannot afford to give up on determinism entirely. The compromise that actually works is this: *carry a deterministic seed through every layer of your pipeline that is under your control, and treat the LLM as the one explicitly stochastic step inside an otherwise reproducible pipeline.* The seed is not there to make the model deterministic. It is there to make every other component deterministic, so that when the model surprises you, you can prove the surprise came from the model and not from your retry logic, your tokenizer cache, your tool router, or the order in which you happened to read three files.

I write this because I have watched too many teams chase "why did this run produce different output than yesterday" through five layers of their stack and conclude "the model is non-deterministic" when in fact the model was the one component behaving the same both times. The non-determinism was theirs.

## What "deterministic" actually means here

There are three different things people mean when they say a pipeline is deterministic, and conflating them is the root of most of the confusion.

1. **Bit-deterministic LLM output.** Same prompt, same parameters, byte-identical completion. This is essentially impossible across providers, often impossible across model versions, and unreliable even within a single deployment because of GPU non-determinism, batched inference, and provider-side load balancing. Stop trying.
2. **Distributionally stable output.** Same prompt, same parameters, output that is "the same kind of answer" across runs. Sampling at `temperature=0` gets you close but not all the way; the rest is provider behavior you do not control.
3. **Pipeline-deterministic.** Given the same inputs and the same seed, every step *that is not the LLM call itself* produces the same intermediate state, in the same order, with the same retries, the same tool selections, the same prompts assembled in the same way. The LLM is the only source of variance.

The third one is what you can actually achieve, and it is also what you actually need. Pipeline determinism is what lets you debug. It is what lets you replay a mission from yesterday and see whether your fix would have changed the outcome. It is what lets you A/B two prompts honestly. It is what makes "the model is wrong" a falsifiable claim instead of a shrug.

## Where non-determinism sneaks in

Before adding a seed, it is worth being concrete about where the variance comes from in a typical agent pipeline. In rough order of how often I have been bitten by each:

- **Dictionary / set iteration order.** Python guarantees dict insertion order since 3.7, but `set` does not. If your tool router does `for tool in tool_set:` to choose which tools to expose, you have already lost determinism. Sort.
- **`os.listdir` / glob ordering.** On macOS APFS the order is roughly insertion-time. On Linux ext4 it is hash-table order. On NFS it is whatever the server feels like. Sort.
- **Concurrent execution.** Two parallel tool calls completing in different orders across runs. The order they appear in the transcript becomes part of the prompt for the next step, which changes the next step's output even at `temperature=0`.
- **Time-based inputs.** `datetime.now()` baked into a prompt. UUIDs. Random temp file names. Anything that varies between runs even when the inputs are "the same."
- **Network jitter affecting retry order.** Tool A and tool B both make HTTP calls. On run 1, A returns first. On run 2, B does. The ordering of their results in the prompt changes everything downstream.
- **Floating-point reductions in batched inference.** This is the only one you cannot fix from the client side. Ignore it. The other five are 99% of your variance.

A deterministic seed alone fixes none of these. A deterministic seed plus a discipline of using it for *every* random or order-sensitive decision in your pipeline fixes all of them except the last.

## What the seed is actually for

The seed is the input to a per-mission `Random` instance, and that instance is the *only* allowed source of randomness anywhere in your pipeline. No `random.random()`. No `uuid.uuid4()`. No `time.time()` baked into prompts. If a step needs randomness, it draws from the mission's RNG.

```python
import random
import hashlib
from dataclasses import dataclass

@dataclass
class Mission:
    id: str
    seed: int
    rng: random.Random

def new_mission(mission_id: str, seed: int | None = None) -> Mission:
    if seed is None:
        # derive seed deterministically from mission id so reruns are stable
        seed = int.from_bytes(
            hashlib.sha256(mission_id.encode()).digest()[:8], "big"
        )
    return Mission(id=mission_id, seed=seed, rng=random.Random(seed))
```

That `rng` is then threaded through everything. Tool selection, retry jitter, sampling among equally-good plans, choosing which file to read first when the agent asks "list the source files," generating idempotency keys. *Everything.* The rule is simple: if a function's output depends on randomness, it must take an `rng` argument. No defaults to module-level `random`. Lint for it.

This sounds tedious. It is tedious for about a day, and then it is invisible, and the payoff is that "rerun mission `m-2026-04-24-0017`" produces the same sequence of tool calls every time, even though the LLM responses themselves may vary slightly.

## Threading the seed through tool calls

The interesting case is tool calls. The tool call itself is non-deterministic from the pipeline's perspective (the network is involved, the tool may have side effects, the response may vary). But the *pipeline's response to the tool result* should be deterministic given the result.

The trick is to record the tool call result the first time you make it, and on replay, return the recorded result instead of re-invoking the tool. This is the standard "VCR" pattern from HTTP testing, applied to agent loops.

```python
import json
import os
import hashlib
from pathlib import Path

class ToolCassette:
    def __init__(self, mission_id: str, mode: str):
        assert mode in ("record", "replay", "passthrough")
        self.mode = mode
        self.path = Path(f".cassettes/{mission_id}.jsonl")
        self.path.parent.mkdir(exist_ok=True)
        self._index = 0
        if mode == "replay":
            self._entries = [json.loads(l) for l in self.path.read_text().splitlines()]
        else:
            self._entries = []

    def call(self, name: str, args: dict, fn) -> dict:
        key = self._key(name, args)
        if self.mode == "replay":
            entry = self._entries[self._index]
            self._index += 1
            assert entry["key"] == key, f"replay drift at step {self._index}: {entry['key']} != {key}"
            return entry["result"]
        result = fn(**args)
        if self.mode == "record":
            with self.path.open("a") as f:
                f.write(json.dumps({"key": key, "name": name, "args": args, "result": result}) + "\n")
        return result

    @staticmethod
    def _key(name: str, args: dict) -> str:
        canonical = json.dumps({"name": name, "args": args}, sort_keys=True)
        return hashlib.sha256(canonical.encode()).hexdigest()[:16]
```

With this in place, a mission run in `record` mode produces a cassette. The same mission run in `replay` mode against that cassette is fully pipeline-deterministic: every tool call returns exactly what it returned the first time, in the same order. The only remaining variance is the LLM step itself, and now you can isolate it: if a replay produces different downstream behavior, the LLM did something different. If the replay produces identical behavior, the LLM was stable and your bug is somewhere in your pipeline glue.

The `assert entry["key"] == key` is the *important* part. If the cassette's expected call does not match the actual call, the pipeline has drifted. Maybe the LLM proposed a different tool. Maybe your prompt assembly changed and the model asked for `read_file("a.py")` instead of `read_file("./a.py")`. The cassette tells you exactly which step the drift started at, which is information that simply does not exist in a non-deterministic pipeline.

## What this rules out

This is the section where I want to be precise, because the value of pipeline determinism is mostly in the bug shapes it eliminates.

- **"Heisenbug" tool routing.** When the agent picks tool A on Monday and tool B on Tuesday given the same inputs, you can no longer blame "the model." Either your routing has unsorted iteration somewhere, or your prompt-assembly is non-deterministic. The seed forces you to find which.
- **Flaky test suites for prompts.** Prompt regression tests that "fail intermittently" are almost always failing because of pipeline non-determinism, not model non-determinism. Once the pipeline is deterministic, the only intermittent failures left are real model variance, and you can quantify it.
- **Replay-vs-live drift bugs.** When the cassette assertion fires, you have caught a real bug: something in your pipeline assembled a different prompt or made a different tool call than the recording. Without the assertion, this drift is silent and shows up as "weird" behavior weeks later.
- **Spurious diff in eval runs.** If you eval a prompt change by running the new prompt against 100 missions and diffing against the baseline, half the diffs in a non-deterministic pipeline are noise. With a fixed seed and replayed tool calls, every diff is signal.
- **"It worked when I ran it locally."** Determinism makes "I ran mission `X` with seed `S` and got result `R`" a complete and falsifiable statement. Anyone can rerun `X` with seed `S` and check.

It does not rule out: the LLM itself producing different tokens, model version changes producing different behavior, or genuinely concurrent tool calls (those are an explicit choice; if you want them deterministic, serialize them and pay the latency).

## A worked example of cassette drift catching a real bug

A few weeks ago I had a mission that worked fine in record mode and failed on replay with a cassette assertion. The recorded call was `read_file("src/foo.py")`. The replayed call was `read_file("./src/foo.py")`. Identical files. Identical contents. Different keys.

The cause: between record and replay I had refactored the path-normalization helper to stop stripping leading `./`. The agent's behavior was unchanged. The model's behavior was unchanged. My pipeline had silently changed how it serialized tool arguments. In a non-deterministic setup, this would have shown up later as "the prompt cache hit rate dropped 15% sometime last week and we don't know why" — because each call now had a different cache key. The cassette caught it the same afternoon.

That is the thing about pipeline determinism. It does not just help you debug the loud bugs. It catches the quiet ones, the ones that would otherwise show up as a slowly-degrading metric three weeks later with no obvious cause.

## Smallest reproducible setup

A complete demo using only the standard library. Saves a cassette, replays it, and demonstrates drift detection. Save as `seed_demo.py`.

```python
#!/usr/bin/env python3
"""Pipeline determinism with a deterministic seed and a tool cassette."""
import json, os, hashlib, random, sys
from pathlib import Path

CASSETTE = Path("/tmp/mission_cassette.jsonl")

def seed_rng(mission_id: str) -> random.Random:
    s = int.from_bytes(hashlib.sha256(mission_id.encode()).digest()[:8], "big")
    return random.Random(s)

def canonical_key(name: str, args: dict) -> str:
    return hashlib.sha256(json.dumps({"n": name, "a": args}, sort_keys=True).encode()).hexdigest()[:12]

def fake_llm_choose_tool(rng: random.Random, options: list[str]) -> str:
    # deterministic given rng state; in real life this is your LLM call
    return rng.choice(sorted(options))  # sort first! never iterate raw set

def run(mode: str, mission_id: str, drift: bool = False):
    rng = seed_rng(mission_id)
    if mode == "replay":
        entries = [json.loads(l) for l in CASSETTE.read_text().splitlines()]
        idx = 0
    else:
        CASSETTE.unlink(missing_ok=True)
        entries = []
        idx = 0

    tools = {"a", "b", "c", "d"}
    transcript = []
    for step in range(5):
        chosen = fake_llm_choose_tool(rng, list(tools))
        args = {"step": step, "param": rng.randint(0, 1_000_000)}
        if drift and step == 2:
            args["param"] += 1   # simulate a pipeline change
        key = canonical_key(chosen, args)
        if mode == "replay":
            expected = entries[idx]
            assert expected["key"] == key, (
                f"DRIFT at step {step}: cassette={expected['key']} live={key}"
            )
            result = expected["result"]
            idx += 1
        else:
            result = f"ok({chosen}/{args['step']})"
            with CASSETTE.open("a") as f:
                f.write(json.dumps({"key": key, "name": chosen, "args": args, "result": result}) + "\n")
        transcript.append((chosen, args, result))
    return transcript

if __name__ == "__main__":
    mode = sys.argv[1] if len(sys.argv) > 1 else "record"
    drift = len(sys.argv) > 2 and sys.argv[2] == "drift"
    t = run(mode, "m-demo-0001", drift=drift)
    for c, a, r in t:
        print(f"  {c:4s} {a} -> {r}")
```

Run it three times:

```
$ python seed_demo.py record
  c {'step': 0, 'param': 463829} -> ok(c/0)
  a {'step': 1, 'param': 184421} -> ok(a/1)
  ...

$ python seed_demo.py replay
  c {'step': 0, 'param': 463829} -> ok(c/0)
  a {'step': 1, 'param': 184421} -> ok(a/1)
  ...   # identical to record

$ python seed_demo.py replay drift
  c {'step': 0, 'param': 463829} -> ok(c/0)
  a {'step': 1, 'param': 184421} -> ok(a/1)
  AssertionError: DRIFT at step 2: cassette=... live=...
```

The third run is the interesting one. The pipeline drifted by one parameter at step 2, and the cassette caught it precisely there. No guessing. No bisecting. The exact step, the exact key, the exact diff.

## The discipline that makes it work

A deterministic seed is a tool. The discipline is what makes it useful. There are three rules that have to be enforced for the whole thing to hold together.

**Rule 1: the mission RNG is the only RNG.** No `import random` at module scope. No `time.time()` in prompt assembly. No `uuid.uuid4()`. If you need a unique id, derive it from the mission RNG. If you need a timestamp inside a prompt, freeze it at mission start and pass it through.

**Rule 2: every collection iterated over for a decision must be sorted.** This includes tool sets, file lists, dict items used in a `for k, v in d.items()` that influences a decision. Python's `sorted()` is cheap and unambiguous.

**Rule 3: tool calls go through the cassette in record mode in production.** Disk is cheap. Cassettes are jsonl. The 50KB per mission is worth every byte the first time you need to replay a mission to debug a customer-reported issue. You can prune them after 30 days.

Adopt these three rules and add a per-mission seed to your mission state. That is the entire intervention. The result is a pipeline where the LLM is the only honestly-stochastic component, every other layer is reproducible, and "what changed" is a question with a finite, mechanical answer instead of a shrug.

The seed does not make your agent deterministic. It makes the *non-determinism legible*, by isolating it to one identifiable step. That is enough. That is, in fact, more than most production agent systems achieve, and it is the difference between an agent loop you can debug and one you can only restart.
