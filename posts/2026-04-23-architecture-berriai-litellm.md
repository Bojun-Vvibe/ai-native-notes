---
title: "Architecture deep-dive: BerriAI/litellm"
date: 2026-04-23
tags: [architecture, deep-dive, litellm]
est_reading_time: 14 min
---

## TL;DR

- `litellm` is not really an agent. It's the **routing/translation layer that agents sit on top of**: a Python SDK plus an optional FastAPI gateway that gives you one OpenAI-shaped API and pretends every other provider has the same shape. If you've used aider, OpenHands, or many of the other tools in this series, you've used litellm whether you knew it or not.
- The most interesting part isn't the provider transformations — those are tedious by nature — it's the **`Router`**: a 10,513-line class that owns deployment selection, retries, fallbacks, cooldowns, rate-limit accounting, and cost tracking. Almost every operational property you want from a multi-model setup is encoded here.
- What I'd steal: the **deployment cooldown algorithm**. Not "this provider returned an error, try the next one" but "this provider has a >50% failure rate over the last minute with at least N requests, take it out of rotation for X seconds." This is the kind of thing every agent reinvents badly; litellm has a reusable version.

A note up front: this post focuses on the SDK side (`litellm/`) and barely touches the proxy (`litellm/proxy/`). The proxy is a 1.5x layer of auth, rate-limiting, budgets, and Postgres on top of the same SDK; if you read this and then read `ARCHITECTURE.md` for the proxy layer, you have the full picture.

## Repo at a glance

- **Language**: Python 3, ~964 MB checkout (lots of test fixtures, model price JSON, frontend assets).
- **Top-level**: `litellm/` (the SDK), `litellm/proxy/` (the FastAPI gateway), `tests/` (~thousands of test files), `docs/`, `enterprise/`, `ui/` (admin frontend), `model_prices_and_context_window.json` (38,689 lines of pricing data).
- **Inside `litellm/`**: `main.py` (7,807 lines — the public `completion()` / `acompletion()` entrypoints), `utils.py` (9,574 lines — wrappers, logging, response normalization), `router.py` (10,513 lines — the Router class), `llms/` (~140 provider subdirectories, each with their own `transformation.py`), `caching/` (Redis, in-memory, S3, Qdrant cache backends), `cost_calculator.py` (2,338 lines), `router_strategy/` (8 routing algorithms), `router_utils/` (cooldowns, retry policies, fallback handlers).
- **Build**: Poetry (`pyproject.toml`, `uv.lock`). Frontend Next.js in `ui/`.

All references below are to commit `63ba912b47760548a070b8ee225f5054dabf08f0`.

The size is worth pausing on. `router.py` alone is more than the entire codex `core/src/session/turn.rs` (1,800 lines) plus `aider/coders/base_coder.py` (3,400 lines). This is the largest single Python file I've read in the four-repo series. Most of it is *operational*: which deployment to pick, how to count fails per minute, how to thread retry policy precedence through fallback layers, how to add headers to responses. Not glamorous; entirely necessary.

## The agent loop

litellm doesn't have an agent loop. It has a **request loop**, which is structurally similar but conceptually different.

The shape: when you call `router.acompletion(model="my-group", messages=[...])`, the call funnels through:

1. `async_function_with_fallbacks` — outer wrapper that catches "this whole model group is unusable" errors and tries the next group.
2. `async_function_with_retries` — middle wrapper that catches retryable errors (429, 500, transient connection failures) and re-tries against the same model group with backoff.
3. `make_call` → `original_function` (typically `litellm.acompletion`) → eventually a provider-specific HTTP call.

Each layer has its own job, and the layering matters. Here's the retry loop, the heart of the request lifecycle ([`router.py:5693`](https://github.com/BerriAI/litellm/blob/63ba912b47760548a070b8ee225f5054dabf08f0/litellm/router.py#L5693)):

```python
# litellm/router.py:5693-5810 (heavily abridged)
async def async_function_with_retries(self, *args, **kwargs):
    original_function = kwargs.pop("original_function")
    fallbacks = kwargs.pop("fallbacks", self.fallbacks)
    context_window_fallbacks = kwargs.pop("context_window_fallbacks", ...)
    content_policy_fallbacks = kwargs.pop("content_policy_fallbacks", ...)
    model_group_retry_policy = kwargs.pop(
        "model_group_retry_policy", self.model_group_retry_policy
    )
    num_retries = kwargs.pop("num_retries")

    _metadata["attempted_retries"] = 0
    _metadata["max_retries"] = num_retries
    try:
        response = await self.make_call(original_function, *args, **kwargs)
        return add_retry_headers_to_response(
            response=response, attempted_retries=0, max_retries=None
        )
    except Exception as e:
        original_exception = e
        deployment_num_retries = getattr(e, "num_retries", None)
        if deployment_num_retries is not None:
            num_retries = deployment_num_retries
        (
            _healthy_deployments,
            _all_deployments,
        ) = await self._async_get_healthy_deployments(
            model=kwargs.get("model") or "",
            parent_otel_span=parent_otel_span,
        )

        # Check retry policy FIRST, before should_retry_this_error
        _retry_policy_applies = False
        if self.retry_policy is not None or model_group_retry_policy is not None:
            _retry_policy_retries = _get_num_retries_from_retry_policy(
                exception=original_exception,
                model_group=_model_group_for_retry_policy,
                model_group_retry_policy=model_group_retry_policy,
                retry_policy=self.retry_policy,
            )
            if _retry_policy_retries is not None:
                num_retries = _retry_policy_retries
                _retry_policy_applies = True

        if not _retry_policy_applies:
            self.should_retry_this_error(
                error=e,
                healthy_deployments=_healthy_deployments,
                all_deployments=_all_deployments,
                ...
            )
        # ... compute backoff via _time_to_sleep_before_retry ...
        await asyncio.sleep(retry_after)

        for current_attempt in range(num_retries):
            try:
                _metadata["attempted_retries"] = current_attempt + 1
                response = await self.make_call(original_function, *args, **kwargs)
                return add_retry_headers_to_response(...)
            except Exception as e:
                original_exception = e
                # ... refresh healthy deployments, recompute backoff, sleep, continue ...
        raise original_exception
```

Things worth noticing:

1. **`num_retries` can be overridden by the exception itself.** `getattr(e, "num_retries", None)` — if a downstream layer attached a hint to the exception, that wins. This is how a per-deployment override propagates.

2. **Retry policies take precedence over generic retry decisions.** The `_retry_policy_applies` flag short-circuits `should_retry_this_error`. If you've configured "retry RateLimitError 5 times for the GPT-4 model group," that decision wins even if the generic logic says "this error type isn't retryable."

3. **Healthy deployments are refreshed on every retry.** Inside the `for current_attempt in range(num_retries)` loop, `_async_get_healthy_deployments` is called again. This matters because between attempt N and attempt N+1, the cooldown handler may have removed deployments from the pool, or `model_group_retry_policy`'s `should_cooldown` flag may have changed which deployments are even eligible.

4. **The final `raise` annotates the exception.** `setattr(original_exception, "max_retries", num_retries)` — the caller gets back the most recent exception, decorated with how many attempts were made. This is what produces the `x-litellm-attempted-retries` and `x-litellm-max-retries` headers.

The fallback layer above ([`router.py:5594`](https://github.com/BerriAI/litellm/blob/63ba912b47760548a070b8ee225f5054dabf08f0/litellm/router.py#L5594)) is simpler in shape but more complex in policy. There are three fallback lists: regular `fallbacks`, `context_window_fallbacks` (used when the request was too long), and `content_policy_fallbacks` (used when a moderation filter rejected the request). Different errors route to different lists. The code that picks the right list and prepends "order-based" fallback entries (cheap-tier-first, then mid-tier, then premium) is in `async_function_with_fallbacks_common_utils` and runs to ~260 lines. Most of that is policy expression: "if the user said `disable_fallbacks=True`, raise; if there's an `_encrypted_content_affinity_pinned` flag, never fall back; if the model group has multiple deployment orders, try them in sequence before going to a different group."

This three-layer split (fallback / retry / call) is the cleanest way I've seen to express the operational logic of "be resilient, but don't paper over real errors." Codex retries by re-spawning the whole turn loop. Aider retries by counting `max_reflections`. OpenHands retries by emitting a new Action when the controller sees an error Observation. None of them separate "retry the same thing" from "switch to something else" the way litellm does.

## Tool calling implementation

litellm doesn't dispatch tools. It passes them through.

When you call `litellm.completion(model="gpt-4", messages=[...], tools=[...])`, the SDK takes your `tools` argument, transforms it to whatever shape the target provider expects, makes the HTTP call, and transforms the response back into OpenAI's `tool_calls` shape. The agent on top — codex, aider, OpenHands, whoever — is responsible for actually executing the tool and feeding the result back as a `tool` message on the next turn.

The interesting part is the transformation layer. Every provider has a `transformation.py` under `litellm/llms/<provider>/chat/`. The Anthropic one ([`llms/anthropic/chat/transformation.py:1378`](https://github.com/BerriAI/litellm/blob/63ba912b47760548a070b8ee225f5054dabf08f0/litellm/llms/anthropic/chat/transformation.py#L1378)) is representative:

```python
# litellm/llms/anthropic/chat/transformation.py:1378-1432 (abridged)
def transform_request(
    self,
    model: str,
    messages: List[AllMessageValues],
    optional_params: dict,
    litellm_params: dict,
    headers: dict,
) -> dict:
    """Translate messages to anthropic format."""
    if (
        "tools" not in optional_params
        and messages is not None
        and has_tool_call_blocks(messages)
    ):
        if litellm.modify_params:
            optional_params["tools"], _ = self._map_tools(
                add_dummy_tool(custom_llm_provider="anthropic")
            )
        else:
            raise litellm.UnsupportedParamsError(
                message="Anthropic doesn't support tool calling without `tools=` "
                        "param specified. ...",
                ...
            )

    # Drop thinking param if thinking is enabled but thinking_blocks are missing
    if (
        optional_params.get("thinking") is not None
        and last_assistant_with_tool_calls_has_no_thinking_blocks(messages)
        and not any_assistant_message_has_thinking_blocks(messages)
    ):
        if litellm.modify_params:
            optional_params.pop("thinking", None)
            litellm.verbose_logger.warning(
                "Dropping 'thinking' param because the last assistant message "
                "with tool_calls has no thinking_blocks."
            )
```

A few things stand out:

1. **`litellm.modify_params` is a global escape hatch.** If `modify_params=True`, the transformation will *invent a dummy tool* to satisfy Anthropic's "you must declare tools to send tool messages" requirement. If `modify_params=False`, it raises. This is the kind of decision that has to be a user-controlled flag, because silently inventing tools is exactly the bug your monitoring won't catch.

2. **Provider-specific quirks are encoded as inline patches.** The `thinking` param shenanigans exist because Anthropic's extended-thinking API has a specific contract: if any assistant message in the history has `thinking_blocks`, every subsequent assistant message must also have them. If the user toggled thinking on mid-conversation, you have a problem. The transformation handles it by dropping `thinking` from the request params *only if* no assistant message has thinking blocks yet. That's an entire issue's worth of context (linked in the comment) compressed into a 6-line conditional.

3. **The transformation is one function, but it pulls helpers from everywhere.** `anthropic_messages_pt` from `prompt_templates/factory.py`, `has_tool_call_blocks` from a util module, `_map_tools` from a base class, `update_headers_with_optional_anthropic_beta` from the model info class. The transformation file is 2,096 lines of similar surgical patches, with utilities scattered across the codebase. This is not architecturally elegant. It is, however, the only way a project with 140 providers stays maintainable: each provider's quirks live in its own subtree, with the shared logic factored out where it makes sense.

The reverse direction (`transform_response`) does the same thing for responses: parse the provider's response format, normalize tool calls into OpenAI's `{"id": ..., "type": "function", "function": {"name": ..., "arguments": ...}}` shape, attach usage tokens in OpenAI's shape, return a `ModelResponse`.

For agents on top of litellm: you write your tool-calling logic against OpenAI's API shape, and you get all 140 providers for free. That's the entire value proposition. The cost is paid by the litellm maintainers, in the form of 140 transformation files.

## Prompt construction & cache discipline

litellm has nothing to say about prompt construction — that's the agent's job. But it does have a **caching layer** that's worth understanding because it's pluggable in a way most agent-internal caches aren't.

The `Cache` class in `litellm/caching/caching.py` supports backends for in-memory (Python dict), Redis, S3, disk, Qdrant (vector), and a few others. Cache keys are computed from `(model, messages, temperature, ...)` — the inputs that determine the response. The cache is enabled with:

```python
litellm.cache = Cache(type="redis", host=..., port=...)
```

Once enabled, every `completion()` call checks the cache first and stores results on success.

The interesting design choice: **caching happens around the call, not inside it.** The provider-side prompt-cache headers (Anthropic's `cache_control` markers, Gemini's context cache) are passed through verbatim — litellm doesn't try to manage them. So you have two layers of caching potentially in play:

1. **litellm response cache**: "I've seen this exact request before, return the same response." Cheap (no LLM call), but only hits on identical requests.
2. **Provider prompt cache**: "I've seen this exact prefix before, charge less for it." Hits on partial overlap, but you still pay for the API call.

If your agent uses both, the litellm cache short-circuits the provider call entirely on identical requests, and the provider cache helps on near-identical ones. The interaction is clean because the layers don't overlap.

For tokens-and-money discipline, the relevant subsystem is `cost_calculator.py` (2,338 lines) and the `model_prices_and_context_window.json` file (38,689 lines, ~3,000 model entries). Every response that flows through litellm gets a cost attached in `response._hidden_params["response_cost"]`. The proxy turns this into an `x-litellm-response-cost` header and a row in the `LiteLLM_SpendLogs` Postgres table.

The price file is updated by hand (and by automated scripts that scrape provider pricing pages). It's the single most useful file in the repo if all you want is "what does GPT-5 mini cost per million tokens." Just `jq '.["gpt-5-mini"]' model_prices_and_context_window.json` and you're done.

## State / session management

litellm is mostly stateless. The Router has state — TPM/RPM counters, cooldown lists, deployment health — but a single request flowing through doesn't accumulate state.

The state that matters lives in the Router's `cache` field, a `DualCache` (in-memory + Redis). Three things go in there:

1. **Per-deployment TPM/RPM counters**, keyed by `(deployment_id, current_minute)`. Used by the `lowest_tpm_rpm_v2` routing strategy to pick the deployment with the most headroom.
2. **Cooldown sets**, keyed by `(model_group, current_minute)`. A deployment in this set is excluded from the "healthy deployments" list for the cooldown period.
3. **Latency histograms**, keyed by `f"{model_group}_map"`. Used by `lowest_latency` routing to pick the deployment with the lowest p50.

All three are time-windowed, all three flush from in-memory to Redis on a schedule (so multiple gateway instances share state), and all three are read on the hot path of every request.

The cooldown logic ([`router_utils/cooldown_handlers.py:166`](https://github.com/BerriAI/litellm/blob/63ba912b47760548a070b8ee225f5054dabf08f0/litellm/router_utils/cooldown_handlers.py#L166)) is the part I think every agent project should crib:

```python
# litellm/router_utils/cooldown_handlers.py:166-249 (abridged)
def _should_cooldown_deployment(
    litellm_router_instance, deployment, exception_status, original_exception
) -> bool:
    """
    Deployment is put in cooldown when:
    - cooldown if got a 429 error from LLM API
    - if %fails/%(successes + fails) > ALLOWED_FAILURE_RATE_PER_MINUTE
    - got 401 Auth error, 404 NotFound - checked by litellm._should_retry()
    """
    model_group = litellm_router_instance.get_model_group(id=deployment)
    is_single_deployment_model_group = (
        model_group is not None and len(model_group) == 1
    )

    num_successes_this_minute = get_deployment_successes_for_current_minute(...)
    num_fails_this_minute = get_deployment_failures_for_current_minute(...)
    total_requests_this_minute = num_successes_this_minute + num_fails_this_minute

    percent_fails = 0.0
    if total_requests_this_minute > 0:
        percent_fails = num_fails_this_minute / total_requests_this_minute

    exception_status_int = cast_exception_status_to_int(exception_status)
    if exception_status_int == 429 and not is_single_deployment_model_group:
        return True
    elif (
        percent_fails == 1.0
        and total_requests_this_minute >= SINGLE_DEPLOYMENT_TRAFFIC_FAILURE_THRESHOLD
    ):
        return True
    elif (
        percent_fails > DEFAULT_FAILURE_THRESHOLD_PERCENT
        and total_requests_this_minute >= DEFAULT_FAILURE_THRESHOLD_MINIMUM_REQUESTS
        and not is_single_deployment_model_group
    ):
        return True
    elif litellm._should_retry(status_code=exception_status_int) is False:
        # 401/404 etc — not transient, take it out
        return True
    return False
```

The defensive structure here is the part I want to highlight:

- **Never cool down a single-deployment model group**, because if you do, the user's next request gets "no healthy deployments" instead of "rate limited, try again." Better to leave the broken deployment in and let the error propagate honestly.
- **Require minimum traffic before applying error-rate cooldown**. If you've made 3 requests this minute and 2 failed, that's not a 67% error rate, that's noise. The minimum-requests gate prevents flapping.
- **429 immediately cools down**, because rate limits are explicitly "stop sending me traffic for a bit."
- **Non-retryable errors immediately cool down**, because 401/404 means the deployment is misconfigured and won't get better in the next 60 seconds.

You can implement this in 80 lines and it'll save you from many of the multi-provider failure modes that show up in production.

## Three patterns I'd steal

1. **The three-layer call structure: `with_fallbacks` wraps `with_retries` wraps `make_call`.** Most agents collapse "retry this exact thing" and "switch to a different thing" into one `for attempt in range(N)` loop. Separating them gives you cleaner exception semantics (a `RateLimitError` retried 3 times then fallen back to a different model group is recognizably different from one retried 12 times against the same group), cleaner observability (`x-litellm-attempted-retries` vs `x-litellm-attempted-fallbacks`), and cleaner policy expression (per-error-class retry counts and per-model-group fallback orders).

2. **Cooldowns based on rolling failure rate, not consecutive failures.** Most code I've read does "if this provider fails 3 times in a row, blacklist it." litellm does "if this provider has >50% failure rate over the last minute with at least N requests, blacklist it for X seconds." The second formulation is dramatically less prone to flapping under bursty load and is exactly as easy to implement once you have the per-minute counters.

3. **Pricing as data, not code.** `model_prices_and_context_window.json` is 38,689 lines of JSON. The cost calculator is ~2,300 lines of Python that reads it. Changing the price of GPT-5 mini is a one-line PR; adding a new model with a known price structure is a five-line PR. If your project hard-codes prices in Python, you'll be migrating to a JSON file eventually. Skip the migration.

## Three pitfalls in this codebase

1. **`router.py` is 10,513 lines and growing.** I cannot honestly recommend reading it linearly. The class is doing too many things — selection, retry, fallback, cooldown, cost tracking, async/sync duplication, mock-testing helpers, pre/post-routing hooks, encrypted-content affinity pinning, team-based filtering, retry policy precedence — and they all live as methods on `Router`. The five-layer indentation in `async_function_with_retries` (try / except / for / try / except) is the visible symptom. A refactor into `RouterCore + RetryPolicy + CooldownPolicy + DeploymentSelector` would help; it hasn't happened.

2. **Async and sync versions of every public method.** Every routing strategy has both `get_available_deployments` and `async_get_available_deployments`. Every fallback handler has both. Every cooldown helper has both. The pattern is "if this is the async version, await the cache call; otherwise use the sync version of the cache call." That's mechanical duplication and the duplication has bugs — if you find a bug in `async_function_with_retries`, you'll often find the same bug a few lines apart in the sync version, and there's no guarantee a fix to one gets ported to the other. (codex, by being all-async-Rust, sidesteps this entirely.)

3. **Provider transformations are tested by integration, not unit.** A bug in `anthropic_messages_pt` only surfaces when an Anthropic call breaks. A bug in `openai/chat/transformation.py` only surfaces when an OpenAI call breaks. The test suite has thousands of tests, but many of them require provider API keys and mock servers, and the dev loop is slower than it should be. New providers ship transformation code that gets exercised mostly by users in production. This is a maintenance bet, not a software-engineering claim — and the bet appears to be working — but it's worth knowing if you contribute.

## Comparison hook

Where codex, aider, and OpenHands all assume "one model, the user picked it," litellm assumes "many models, possibly the user picked which group, the system picks which deployment in the group." This is the difference between an agent and an infrastructure layer. The agent worries about what to ask the model; litellm worries about which model to send the ask to and what to do when the model says no.

If you're building an agent today and you don't want to think about provider switching, retry policies, cost accounting, or rate-limit accounting — and you genuinely shouldn't, because each of these is a months-long project — using litellm as your LLM layer is the obvious choice. The cost is one extra dependency and a giant `model_prices_and_context_window.json` in your image. The benefit is that "switch from OpenAI to Anthropic" is a config change, not a refactor.

## Reading order

For a new contributor:

1. `ARCHITECTURE.md` — the official mermaid diagrams. Read both the proxy and SDK sections. ~400 lines.
2. `litellm/main.py:completion()` — the public entrypoint. Skim, don't read in full.
3. `litellm/router.py:224` — `class Router` constructor. Read enough to understand what's stored in `self.model_list`, `self.fallbacks`, `self.cache`, `self.routing_strategy`.
4. `litellm/router.py:5594` — `async_function_with_fallbacks`. Then `async_function_with_retries` at 5693. This is the request lifecycle.
5. `litellm/router_utils/cooldown_handlers.py:166` — `_should_cooldown_deployment`. The most reusable algorithm in the repo.
6. `litellm/router_strategy/lowest_latency.py:560` — `async_get_available_deployments`. Pick one strategy and read it; the others follow the same pattern.
7. `litellm/llms/anthropic/chat/transformation.py:1378` — `transform_request`. Pick one provider and read it; the others follow the same pattern. Note how provider-specific quirks are encoded as inline patches.
8. `litellm/cost_calculator.py:260` — `cost_per_token`. Then look at the structure of `model_prices_and_context_window.json` (just `head -50` it; you don't need to read all 38k lines).

Skip on first pass: anything in `litellm/proxy/` (separate post worth of material), `enterprise/`, `litellm-js/`, `ui/`. Skip the test suite unless you're contributing — it's organized by feature, not by file under test.
