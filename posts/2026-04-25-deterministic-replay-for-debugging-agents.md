# Deterministic Replay for Debugging Agents

Date: 2026-04-25

The single biggest reason debugging an agent feels worse than debugging a regular program is that you can't re-run it. You can re-run the *prompt*, but the model gives you a different completion. You can re-run the *completion*, but the tool calls hit a filesystem that's now in a different state. You can re-run the *tools*, but the timestamps and PIDs and tempfile names won't match. By the time you finish reproducing the bug, you've usually fixed it by accident or convinced yourself it doesn't exist.

Deterministic replay is the answer that systems people have been giving for thirty years to "this concurrent program has a Heisenbug." It's how rr works on Linux, how Pernosco builds its time-traveling debugger, how Hermit gives you record-and-replay on top of arbitrary x86. The technique transfers almost cleanly to agent loops if you're willing to do the engineering. Almost nobody is. Here's what it costs and what you get.

## What "deterministic" has to mean for an agent

A normal recorded execution captures: syscall returns, signal deliveries, RDTSC, /dev/urandom, scheduler decisions on shared memory. Replay re-injects those at the same dynamic instruction count and the program does the same thing.

An agent loop is much higher-level but the principle is identical. You need to capture and replay:

1. **Every model completion.** Given the same request body, replay returns the same response body, byte-for-byte, including the streaming chunk boundaries if you care about them.
2. **Every tool result.** `Bash("ls")` returned `foo.txt\nbar.txt\n` and exited 0 at this point in the trace; replay returns that, not whatever `ls` says today.
3. **Every clock read and every "random" seed** the agent loop itself uses. Cache keys, request IDs, idempotency tokens, retry jitter, file-name suffixes — all of it.
4. **The order of parallel tool calls.** If the model emitted three tool calls and the agent dispatched them concurrently, the order in which their results were appended back into the message log matters for the next turn.

If you capture all four, you can re-run the agent and it will produce the same final state. If you miss any one, you have a flaky replay, which is worse than no replay because it gives false confidence.

## Where to record

There are three honest layers, each with a different cost/fidelity tradeoff.

**Layer A: Record at the wire.** Stick a transparent proxy between the agent process and the model API. Save the full HTTP request/response bytes to disk, indexed by request hash. Replay by swapping the OpenAI/Anthropic base URL to the proxy. litellm has had a primitive version of this for ages — see the `cache` module in litellm 1.67.x, plus the more recent disk-cache work in PR #11842 that landed semantic caching at the request level. The wire layer is the cheapest to implement (HTTPS MITM with mkcert + a tiny aiohttp server is a weekend project) and the most portable across agents.

The catch: you only see what the agent sent, not what the agent decided. If the agent loop has its own non-determinism (UUID generation, time-based cache keys, parallel tool dispatch ordering), the wire trace doesn't capture it. You'll see the model returned identical bytes and the agent did something different anyway.

**Layer B: Record at the agent loop.** Instrument the agent's main message-passing loop. Every assistant message, every tool call, every tool result, every clock read, every `uuid4()` — append to a structured log. opencode's session log under `~/.local/share/opencode/project/<hash>/storage/session/` is genuinely close to this; it captures the full message stream including tool outputs. crush keeps a similar log under `~/.local/share/crush/`. codex has its rollout files under `~/.codex/sessions/`.

To turn any of those into a *replay* substrate, you have to do two things they don't do today:

- Capture a salt/seed at session start, and use it for every "random" decision in the agent (cache key entropy, suffixes, jitter).
- Capture the wallclock at every clock read site and play it back via a fake clock during replay.

Without those two, you have an excellent post-mortem log but you can't re-run.

**Layer C: Record at the syscall.** Use rr or Hermit to record the entire agent process. This is total fidelity — every malloc, every signal, every scheduler decision is reproducible. It's also enormous (gigabytes per session), Linux-only, and breaks the moment your agent does anything cleverly multi-process or relies on GPU. I have not seen anyone seriously use this layer for an LLM agent in production. It's overkill for the bug shapes you actually hit.

The pragmatic answer is **B with a wire-level fallback for the model calls**. You instrument the agent loop you control, and you proxy the model calls so you don't have to trust the model SDK to be deterministic.

## The seed-and-clock contract

Every place in the agent loop that reads a non-deterministic input has to be funneled through one of two interfaces:

```python
class ReplayContext:
    def now(self) -> datetime: ...
    def monotonic(self) -> float: ...
    def random_bytes(self, n: int) -> bytes: ...
    def uuid(self) -> str: ...
    def env(self, key: str) -> str | None: ...
```

In record mode, each call appends `(callsite_id, return_value)` to the trace. In replay mode, each call pops the next value with the matching `callsite_id` and asserts the order matches. Mismatch = the agent's deterministic behavior has drifted, which is itself useful diagnostic info ("you changed the order of two calls between record and replay, here's where").

The hardest one is `env`. Agents read env vars constantly — `OPENAI_API_KEY`, `PATH`, `HOME`, `TERM`, `USER`, `PWD`. You don't want the replay to use the recorded `OPENAI_API_KEY` (it might have rotated) but you do want it to use the recorded `PWD` (otherwise relative paths in tool calls go sideways). My solution has been a pluggable env policy: secrets are re-read live, everything else is replayed. This is gross but it works.

The *easiest* one is `monotonic`. Just record monotonic deltas, replay them as-is, never let real wallclock leak in.

## Tool result capture is the rabbit hole

Here's where it gets interesting. The agent issues `Bash("git status")`. In record mode, you log the stdout/stderr/exit code/duration. In replay mode, you return the recorded values without actually running `git status`.

This is fine for `git status`, `ls`, `cat`. It is not fine for `git push`, `npm install`, anything with side effects. So replay mode has to operate in two sub-modes:

- **Pure replay**: every tool call returns recorded result, nothing executes for real. Good for "why did the agent decide to do X?" — you just want to retrace its reasoning.
- **Branched replay**: replay up to point P, then let the agent run live from there with real tool execution. Good for "the agent went off the rails at turn 12, let me fix the prompt and re-run from turn 11."

The branched-replay mode is where the value is, and it's also where the engineering hurts. You have to make sure the world state at point P matches the recorded world state, otherwise the live execution from P diverges immediately. For filesystem-touching agents that means snapshotting the working directory at every tool call. Btrfs/ZFS snapshots are ideal; on macOS, APFS clones via `cp -c` work; on a non-snapshotting filesystem, you fall back to per-tool-call git stashes inside a hidden worktree, which is slow but correct.

A real example from a debugging session last week: agent had been editing a file across 14 turns and at turn 11 it deleted a function it later needed. I wanted to replay turns 1–10 exactly and then see what happened if I prompted it differently at turn 11. With pure replay, turns 1–10 reproduce trivially. With branched replay, I needed to also restore the working directory to its end-of-turn-10 state. The session-scoped APFS clone made this 200ms instead of "re-execute all 10 turns of edits, hope they're idempotent."

## Determinism vs. the model

Even with everything captured at the agent layer, the model itself is non-deterministic in two ways:

1. **API-level**: `temperature > 0` gives different completions per call. Even `temperature=0` is not perfectly deterministic on most providers — token-level ties get broken by floating-point noise that depends on the underlying GPU's batch composition. Anthropic and OpenAI both document this. (See OpenAI's deterministic outputs guide; the `seed` parameter helps but doesn't promise bit-equality.)

2. **Provider-level**: model versions roll forward. Today's `claude-sonnet-4.5` is not byte-identical to last month's, even at the same name, even at the same `seed`.

This is why the wire-layer recording matters. You're not trying to get the model to be deterministic. You're trying to record what the model said *that one time* and replay that recording. The agent loop never re-calls the model during replay. It calls the proxy, the proxy returns the bytes from the trace, and the agent processes them as if they were fresh. From the agent's perspective the world is fully deterministic.

## What this gets you

Once replay works:

- **Bug reports become tarballs.** "Here's the session, replay it, you'll see the agent picks the wrong file at turn 7." No more "works on my machine."
- **Regression tests for prompt changes.** Take 50 recorded sessions, replay each with a modified system prompt. Diff the resulting tool calls. Did the new prompt make the agent better or just different?
- **Cheap A/B between models.** Take a recorded session that ran on model X. In replay, intercept the model call and re-route to model Y with the same input. Compare. This is much cheaper than running the full session live on both, and it isolates the model variable from everything else.
- **Time travel in debugging.** "Let me see the message log at turn 5" becomes free. "Let me re-run from turn 5 with this hypothesis" becomes a 2-second loop instead of a 5-minute loop.
- **Fuzzing.** Replay with random perturbations to one tool result at a time. Find tool result shapes that crash the agent.

## What it costs

I've now built two of these and the cost is real:

- **Code complexity.** Every non-deterministic call site has to go through the ReplayContext. Miss one and replays drift silently. Static analysis helps a little (forbid `import time` outside one module) but not enough.
- **Storage.** Wire-level traces of long sessions get big. A 30-minute coding session with parallel reads across 200 files plus 80 model turns is ~80MB raw, ~12MB gzipped. Times N developers times M sessions per day times retention. Plan a TB.
- **CI integration.** Replay-based tests are the right thing but you have to teach your CI to load fixtures, mount the proxy, point the agent at it, and assert on the resulting message graph. There's no off-the-shelf framework. You write it.
- **Snapshot management.** Branched replay needs filesystem snapshots. APFS clones are cheap; if your team has Linux laptops, btrfs snapshots are cheap; if anyone's on a network drive, you're cooked.

## The state of the art

Nobody ships this end-to-end yet. The closest pieces I've seen:

- opencode's session storage is structured enough that you could rebuild a replay loop on top of it with maybe two weekends of work. The `Session` and `Message.Part` types in the recent refactor (PR #4421 et al.) made the message graph navigable by ID, which is what you need.
- litellm's caching layer plus its router already gives you wire-level recording for free if you point it at a disk cache and lock the mode to read-only. Crude but functional. See the `cache` integration tests in the repo for the shape.
- charm/crush has the cleanest in-process event log of any of the agents I've looked at; it's a Bubble Tea program so the message dispatch is already a first-class data structure rather than ad-hoc function calls.
- pew-insights 0.4.28 added a `--replay` flag to its evaluator that re-runs scored sessions from a session.jsonl. It's read-only replay (no branching) but it's the most "real" implementation I know of in the open.

What I want and don't have: an agent runtime where `--record` and `--replay` are first-class flags, where the replay is bit-exact for the agent loop and approximate-tolerant for the model layer, where branched replay is one keystroke, and where the recording is small enough to attach to a bug. None of the major agents are there. The first one to ship it well will, I think, change how the rest of us debug overnight.

Until then: build the cheapest version you can. Pipe model calls through litellm with disk cache, log every tool call to jsonl with a callsite id, capture a session-scoped seed, and accept that your replay is "good enough to recover the message graph but not the working directory." That alone makes 80% of agent bugs tractable. The remaining 20% is what justifies the full-fidelity build later.
