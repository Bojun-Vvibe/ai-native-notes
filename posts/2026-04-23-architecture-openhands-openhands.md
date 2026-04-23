---
title: "Architecture deep-dive: OpenHands/OpenHands"
date: 2026-04-23
tags: [architecture, deep-dive, openhands]
est_reading_time: 16 min
---

## TL;DR

- `OpenHands` is the most "operating-system-shaped" of the four agents in this series. The agent doesn't run inline with the user; it runs **inside a Docker sandbox** with its own execution server, and the host process talks to it over an event bus. Everything — user messages, agent thoughts, tool calls, file edits, lint output — is an `Event` on a single shared `EventStream`.
- The most interesting design decision is the **Action / Observation duality**. An `Action` is "the agent wants to do X." An `Observation` is "X happened, here's the result." The agent never executes anything itself; it only emits Actions. A separate `Runtime` subscribes to the event stream, executes Actions inside a sandbox, and emits matching Observations. This gives you free auditability, free replay, and free remoting.
- What I'd steal: the **composable `Condenser` pipeline** for context compression. There are nine implementations (recent-events, LLM-summarizing, attention-based, observation-masking, etc.) and you can chain them. Most agents ship one compaction strategy and call it a day; OpenHands lets you stack them.

A note up front: as of `7bc3300` the `openhands/controller/` directory is **legacy V0**. The header of `controller/agent.py` says it's "Deprecated since version 1.0.0, scheduled for removal April 1, 2026." V1 lives in `openhands/app_server/` and depends on an external `software-agent-sdk`. This post covers V0 because that's where the agent loop is still readable in this repo; I'll flag where V1 changes the picture.

## Repo at a glance

- **Language**: Python 3, ~317 MB checkout (lots of test fixtures).
- **Top-level dirs**: `openhands/` (the runtime), `containers/` (Dockerfiles for sandboxes), `enterprise/` (extras), `frontend/` (React UI), `evaluation/` (benchmark harness).
- **Inside `openhands/`**: `controller/` (V0 agent driver, **deprecated**), `app_server/` (V1 FastAPI bridge to the external SDK), `events/` (the event bus and event types), `runtime/` (sandbox abstractions and the Action Execution Server), `memory/condenser/` (context compression strategies), `llm/` (LLM wrappers around litellm), `mcp/` (MCP client), `microagent/` (small special-purpose prompts).
- **Build**: Poetry (`pyproject.toml`, `poetry.lock`). Frontend separately in `frontend/` with its own `package.json`.
- **Tests**: pytest, ~7k lines under `tests/`. Includes a runtime test suite that spins up real Docker sandboxes.

All references below are to commit `7bc3300`.

## The agent loop

The shape to internalize first: **the host process and the agent are decoupled by an event bus.** When the user types a message, the host appends a `MessageAction` to the `EventStream`. The `AgentController` is subscribed to the stream; when it sees a user message, it calls `Agent.step(state)` and gets back an `Action`. That Action is appended to the stream. The `Runtime` is subscribed too; when it sees an Action, it executes it inside the sandbox and appends an `Observation`. Back at the controller, the next `step` is triggered, which sees the new Observation in `state.history` and decides what to do next.

`AgentController._step` ([`controller/agent_controller.py:815`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/controller/agent_controller.py#L815)) is the inner loop:

```python
# openhands/controller/agent_controller.py:815-867 (abridged)
async def _step(self) -> None:
    if self.get_agent_state() != AgentState.RUNNING:
        return

    if self._pending_action:
        # Don't step while we're waiting for an Observation
        return

    self.state_tracker.sync_budget_flag_with_metrics()
    if self.agent.config.enable_stuck_detection and self._is_stuck():
        await self._react_to_exception(
            AgentStuckInLoopError('Agent got stuck in a loop')
        )
        return

    try:
        self.state_tracker.run_control_flags()
    except Exception as e:
        await self._react_to_exception(e)
        return

    action: Action = NullAction()
    if self._replay_manager.should_replay():
        action = self._replay_manager.step()
    else:
        try:
            action = self.agent.step(self.state)
            if action is None:
                raise LLMNoActionError('No action was returned')
            action._source = EventSource.AGENT
        except (LLMMalformedActionError, LLMNoActionError, ...) as e:
            self.event_stream.add_event(ErrorObservation(content=str(e)), EventSource.AGENT)
            return
```

A few things stand out compared to codex's `run_turn`:

1. **The controller is not the loop.** `_step` runs *one* step. The actual loop is "every time the controller sees an Observation, schedule another `_step`." There is no `while True` here. That makes pause/resume free (just stop dispatching) and replay free (a `_replay_manager` short-circuits the `agent.step` call and returns a recorded Action instead).

2. **`self._pending_action` is the gate.** Once the agent emits an Action, the controller refuses to step again until the matching Observation arrives. This is how the system avoids racing on the model — but it also means the controller can only have one outstanding Action at a time. There's no parallel tool dispatch in V0.

3. **Stuck detection runs every step.** `_is_stuck()` ([`controller/stuck.py`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/controller/stuck.py)) implements heuristics like "the same Action with the same args has been emitted three times in a row" or "the same error keeps coming back." When it fires, the controller raises `AgentStuckInLoopError` and reacts to it. Codex does not have this; aider has its `max_reflections=3` cap but no semantic detection. OpenHands paid the price of writing 488 lines of pattern matching to catch stuck loops; it's the most defensive of the three.

4. **Context-window error handling has hand-coded provider strings.** The `BadRequestError` branch ([`agent_controller.py:882-902`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/controller/agent_controller.py#L882-L902)) literally does `'contextwindowexceedederror' in error_str or 'prompt is too long' in error_str or 'sambanovaexception' in error_str and 'maximum context length' in error_str`. The comment says "this is a hack until a litellm fix is confirmed." It's been in the codebase for a while. This is what production looks like.

The actual call to the model lives in the `Agent.step` method, which is abstract in V0:

```python
# openhands/controller/agent.py:112-117
@abstractmethod
def step(self, state: 'State') -> 'Action':
    """Starts the execution of the assigned instruction. This method should
    be implemented by subclasses to define the specific execution logic.
    """
    pass
```

In V0, `step` was implemented by classes like `CodeActAgent` in the (now-removed) `agenthub/` directory. In V1, `step` is implemented by the external `software-agent-sdk`. Either way the contract is the same: take a `State` (which contains `state.history`, the list of past Events), return one `Action`. The controller does the rest.

The Event Stream itself ([`events/stream.py`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/events/stream.py)) is a hand-rolled pub/sub bus on top of a queue and a thread pool:

```python
# openhands/events/stream.py:43-69 (abridged)
class EventStream(EventStore):
    secrets: dict[str, str]
    _subscribers: dict[str, dict[str, Callable]]
    _lock: threading.Lock
    _queue: queue.Queue[Event]
    _queue_thread: threading.Thread
    _queue_loop: asyncio.AbstractEventLoop | None
    _thread_pools: dict[str, dict[str, ThreadPoolExecutor]]
    _thread_loops: dict[str, dict[str, asyncio.AbstractEventLoop]]

    def __init__(self, sid: str, file_store: FileStore, user_id: str | None = None):
        super().__init__(sid, file_store, user_id)
        self._stop_flag = threading.Event()
        self._queue: queue.Queue[Event] = queue.Queue()
        self._thread_pools = {}
        self._thread_loops = {}
        self._queue_loop = None
        self._queue_thread = threading.Thread(target=self._run_queue_loop)
        self._queue_thread.daemon = True
        self._queue_thread.start()
        self._subscribers = {}
        self._lock = threading.Lock()
```

Every subscriber gets its own `ThreadPoolExecutor` and its own `asyncio` loop, both keyed by `(subscriber_id, callback_id)`. That's a lot of machinery, and the reason is that callbacks can be sync or async, can do I/O, and must not block each other. The cost is a thread-zoo and some asyncio-bridging fragility (note the `_init_thread_loop`-style scaffolding); the benefit is that adding a new subscriber is one `subscribe(...)` call and the bus does the rest.

Persistence falls out for free: `EventStream` extends `EventStore`, which writes every event to `FileStore` (local disk by default, but pluggable to S3 / GCS). Reconstructing a session is `EventStore.search_events(start=0)` plus replaying.

The Runtime side ([`runtime/base.py:377`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/runtime/base.py#L377)) is similarly small:

```python
# openhands/runtime/base.py:377-435 (abridged)
def on_event(self, event: Event) -> None:
    if isinstance(event, Action):
        asyncio.get_event_loop().run_until_complete(self._handle_action(event))

async def _handle_action(self, event: Action) -> None:
    if event.timeout is None:
        event.set_hard_timeout(self.config.sandbox.timeout, blocking=False)
    try:
        await self._export_latest_git_provider_tokens(event)
        if isinstance(event, MCPAction):
            observation: Observation = await self.call_tool_mcp(event)
        else:
            observation = await call_sync_from_async(self.run_action, event)
    except PermissionError as e:
        # ... build an error observation ...
```

The Runtime subscribes to Events. When it sees one that's an `Action`, it dispatches to either MCP (for MCP-shaped tools) or `run_action` (for built-ins like `CmdRunAction`, `FileEditAction`, `BrowseURLAction`). The implementation of `run_action` in the Docker runtime forwards the request as JSON to the Action Execution Server running inside the sandbox container. Result comes back, gets wrapped in an `Observation`, gets appended to the stream. The agent sees it on the next step.

This separation has consequences I genuinely admire. The Runtime is swappable: there's `LocalRuntime` for direct execution, `DockerRuntime` for Docker, `RemoteRuntime` for Kubernetes-backed sandboxes, plus enterprise variants. The Agent doesn't know or care which one is running. The Action Execution Server inside the sandbox is the same regardless of how the sandbox was provisioned. And because everything goes through Events, the `Resolver` (the GitHub-issue-fixing bot) can subscribe to the same stream and watch the agent work without modification.

## Tool calling implementation

OpenHands has two parallel tool systems and one of them is unusual.

**System 1: Native function-calls.** The agent's `tools` list is an array of `ChatCompletionToolParam` (litellm's type). When the model returns a tool call, the agent parses it, instantiates the matching `Action` class, and emits it. MCP tools are added via `set_mcp_tools` ([`controller/agent.py:171`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/controller/agent.py#L171)):

```python
# openhands/controller/agent.py:171-191
def set_mcp_tools(self, mcp_tools: list[dict]) -> None:
    logger.info(
        f'Setting {len(mcp_tools)} MCP tools for agent {self.name}: '
        f'{[tool["function"]["name"] for tool in mcp_tools]}'
    )
    for tool in mcp_tools:
        _tool = ChatCompletionToolParam(**tool)
        if _tool['function']['name'] in self.mcp_tools:
            logger.warning(
                f'Tool {_tool["function"]["name"]} already exists, skipping'
            )
            continue
        self.mcp_tools[_tool['function']['name']] = _tool
        self.tools.append(_tool)
```

Both built-in actions and MCP tools end up in the same `self.tools` list. From the model's perspective they're indistinguishable. Routing happens at execution time in `Runtime._handle_action`: if the Action is an `MCPAction`, it goes to the MCP client; otherwise it goes to the built-in dispatch.

**System 2: CodeAct-style code execution as a meta-tool.** The historic `CodeActAgent` — and many derivatives — exposed *one* tool: "run this Python code." Everything else (file edits, web browsing, even file reads) was implemented as Python that called helpers exposed inside the sandbox runtime. The model writes Python, the sandbox executes it, the stdout/stderr come back as the Observation. This trades schema strictness (no JSON validation) for expressiveness (the model can compose helpers in arbitrary ways).

V1's external SDK appears to have moved fully to System 1 (proper tool definitions for everything). The CodeAct heritage is visible in the runtime: the sandbox includes a Jupyter kernel and a `bash` plugin specifically to support code-as-tool agents.

The Action Execution Server itself ([`runtime/action_execution_server.py`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/runtime/action_execution_server.py)) is a FastAPI app that runs *inside* the Docker container. It exposes endpoints like `/execute_action`, `/list_files`, `/read_file`, plus a websocket for the browser plugin. The host runtime (e.g. `DockerRuntime`) is essentially an HTTP client to that server. This is what lets the same agent code work against a local Docker container, a Kubernetes pod, or a remote provisioned sandbox — they're all just URLs.

There's no parallel tool dispatch in V0. The `_pending_action` gate guarantees one Action at a time. V1 may differ; I haven't read the SDK.

## Prompt construction & cache discipline

V0's prompt assembly lives in `openhands/utils/prompt.py::PromptManager`. It loads Jinja2 templates from `microagents/`, fills them with conversation history, repository info, and a small set of variables, and returns a single string.

The interesting part is the **microagent system**. A microagent is a small Markdown file (in `openhands/microagent/` or in user-defined locations) that contains:

- A YAML frontmatter with metadata (`name`, `triggers`, `agent`).
- A body with prompt text.

When a microagent's trigger keywords appear in the user message, its body gets injected into the system prompt for that turn. There are microagents for "the user mentioned Docker" (inject Docker tips), "the user mentioned GitHub" (inject GitHub workflow knowledge), "the user is in a Python repo" (inject pytest patterns), etc. It's the same idea as cursor's `.cursor/rules` or aider's `--read` files, but with declarative trigger conditions.

Cache discipline in V0 is mostly inherited from litellm. The `LLM` class ([`llm/llm.py`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/llm/llm.py)) has a `_completion` method that calls `litellm.completion(...)`. If the model supports prompt caching, OpenHands relies on litellm to handle it transparently. There are no `cache_control` markers placed by OpenHands itself in the V0 codepath I read — unlike aider, which manages its own breakpoints.

The lack of explicit cache control hurts in long sessions because microagent injection changes the prompt prefix mid-conversation: if the user says "deploy" and a Docker microagent gets injected, the prompt prefix from that point on is different from before. Without cache breakpoints around the microagent block, you can invalidate a large prefix for what's effectively a small addendum.

V1 may handle this differently in the SDK; I can't tell from this repo alone.

## State / session management

The state model is the Event Stream and one accessor on top of it.

`State` ([`controller/state/state.py`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/controller/state/state.py)) is a dataclass that exposes:

- `history: list[Event]` — every event in the conversation, in order.
- `iteration_flag`, `budget_flag` — control-flag counters that the controller increments per step.
- `delegate_level` — non-zero means this state belongs to a sub-agent that was spawned via an `AgentDelegateAction`.
- A few computed properties for "last user message," "last assistant action," etc.

`State.history` is the canonical truth. When the controller calls `agent.step(state)`, the agent rebuilds its own prompt from `state.history`. There's no separate "agent memory."

**Compaction** is where OpenHands has the most sophisticated story of any agent in this series. The `Condenser` ABC ([`memory/condenser/condenser.py`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/memory/condenser/condenser.py)) defines a single method:

```python
# openhands/memory/condenser/condenser.py:91-92
@abstractmethod
def condense(self, View) -> View | Condensation:
```

A `View` is a slice of the event history. `condense` either returns a smaller `View` (passive truncation) or a `Condensation` object containing a summary `Event` that replaces a chunk of history.

The implementations under `memory/condenser/impl/`:

- `recent_events_condenser` — keep the last N events, drop the rest.
- `conversation_window_condenser` — sliding window with token budget.
- `llm_summarizing_condenser` — periodically summarize the prefix using a "weak" LLM (same idea as aider).
- `llm_attention_condenser` — use the model itself to pick which events are most relevant; drop the others.
- `amortized_forgetting_condenser` — exponential-decay style: forget older events with increasing probability.
- `observation_masking_condenser` — keep the Action but mask the Observation (useful when the Observation is huge stdout).
- `browser_output_condenser` — special-case for browser screenshots.
- `structured_summary_condenser` — model-generated structured summary.
- `pipeline.py` — chain them.

Most agents pick one strategy. OpenHands lets you stack them in a pipeline: e.g. *first* mask big browser outputs, *then* summarize anything older than 50 events, *then* keep the last 20 verbatim. The composition is clean because every condenser takes a `View` and returns a `View | Condensation`. This is the single best-engineered subsystem in the V0 codebase and it's the pattern I'd most want to copy.

Persistence: the Event Store writes every event to disk through `FileStore` (local, S3, GCS). Resuming a conversation is `EventStream.replay()` plus rehydrating the controller. Because the agent's prompt is always derived from `state.history`, replay is deterministic up to LLM nondeterminism.

## Three patterns I'd steal

1. **Action / Observation as the unit of agent communication.** Once you accept that "the agent emits intent, something else executes intent, the result comes back as a new event," every problem you'd otherwise solve with bespoke plumbing — replay, audit, remoting, cancellation — becomes a property of the event bus. The cost is forcing every agent action through a typed enum; the benefit is that the agent code stops worrying about *how* anything happens.

2. **Composable Condenser pipeline.** Already covered. If your agent runs long enough to need any context compression, you'll eventually need more than one strategy. Building it as a pipeline of small `View → View | Condensation` transforms is the right factoring.

3. **Action Execution Server inside the sandbox, host as HTTP client.** The `DockerRuntime` is mostly a thin client to a FastAPI server running inside the container. This is the right shape for any agent that needs strong isolation: the host process can't accidentally `rm -rf` the user's filesystem because it doesn't have shell access; everything goes through a typed HTTP API that the sandbox image exposes. Swapping Docker for Kubernetes or a remote provisioner is a one-class change.

## Three pitfalls in this codebase

1. **The V0 / V1 split is in plain view and confusing.** The agent loop in `controller/` is marked legacy. The new bridge in `app_server/` doesn't contain the loop — it just talks to the external `software-agent-sdk`. A new contributor reading this repo finds two parallel structures, one of which is mostly stubs. The README and architecture docs are honest about this, but the migration is mid-flight. Until V0 actually gets removed, the cognitive load is real. (And the V1 SDK lives in a separate repo, so to actually understand the agent loop today you have to read two repos.)

2. **String-matching for context-window errors.** The `'contextwindowexceedederror' in error_str` chain ([`agent_controller.py:882`](https://github.com/OpenHands/OpenHands/blob/7bc3300981fa1cb4689d6e1b0c0bdd7fd77ac954/openhands/controller/agent_controller.py#L882)) is exactly the kind of thing that causes silent failures when a new provider ships. Either litellm needs to grow a richer error taxonomy, or OpenHands should match against `e.__class__.__name__` and `e.code` instead of a lowercased message body.

3. **Event-bus cost.** A `ThreadPoolExecutor` plus an `asyncio` loop per `(subscriber_id, callback_id)` is a lot of machinery for what's logically a function-call. In a benchmark scenario where 50 conversations are running in the same process, you can have hundreds of executors and loops alive. I haven't profiled it, but the structure is set up to encourage exactly this kind of resource sprawl. A cleaner design would be one shared executor and a per-subscriber queue.

## Comparison hook

Where `aider` keeps state in two Python lists (`done_messages`, `cur_messages`) and the agent-runtime boundary is "the same process," OpenHands keeps state in an `EventStream` that's persisted to disk and the agent-runtime boundary is "different processes, possibly different machines." Aider can't be paused and resumed across hosts; OpenHands can be. Aider can't run on infrastructure that isolates the agent from the user's filesystem; OpenHands' whole architecture is built around exactly that. The price OpenHands pays is ~10x the code volume and a lot more moving parts; the price aider pays is "you can't actually trust this thing in production without a separate sandbox layer."

## Reading order

For a new contributor (assuming you'll work on V0 long enough to understand it before V1 lands fully):

1. `openhands/architecture/system-architecture.md` and `agent-execution.md` — the official mermaid diagrams. Read these first or you will be lost.
2. `openhands/events/event.py` — the base `Event` class. Short.
3. `openhands/events/action/__init__.py` and `openhands/events/observation/__init__.py` — the catalog of Action and Observation types. Skim, don't memorize.
4. `openhands/events/stream.py` — the event bus. ~290 lines. Read in full.
5. `openhands/controller/agent.py` — the abstract Agent. 191 lines. Read in full.
6. `openhands/controller/agent_controller.py:815` — `_step`. Then `on_event` at line 407. Then `_react_to_exception` at line 276. This is the V0 loop.
7. `openhands/runtime/base.py:377` — `on_event` and `_handle_action`. The runtime side of the bus.
8. `openhands/memory/condenser/condenser.py` — the `Condenser` ABC. Then pick one impl (start with `recent_events_condenser.py`).
9. `openhands/app_server/README.md` — the V1 bridge. Then `openhands/app_server/v1_router.py` to see the FastAPI surface. After this, you'll need to read the external `software-agent-sdk` repo to go further.

Skip on first pass: `enterprise/`, `frontend/`, anything in `openhands/integrations/`, anything in `evaluation/`.
