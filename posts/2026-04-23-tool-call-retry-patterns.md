---
title: "Tool-call retry patterns: idempotency, backoff, and the model's role in recovery"
date: 2026-04-23
tags: [agents, tools, retry, reliability, error-handling]
est_reading_time: 14 min
---

## TL;DR

Tools fail. Networks drop, file handles go stale, rate limits kick in, JSON schemas reject malformed arguments. The naive response is to wrap every tool in `try/except` and retry on any exception. That works for about a week before producing a flaky agent that retries non-idempotent operations into a corrupted state, hides systematic failures behind backoff, and accumulates tool results the model has to wade through. The right pattern is a layered retry strategy: idempotency-aware retries at the tool boundary, escalation-aware retries inside the agent loop, and explicit handing of failures back to the model when retries are exhausted. This post documents the seven retry patterns I use, when each applies, and the failure modes they each fix.

The key insight: the model is part of the retry strategy. It can reformulate a failed call. It cannot be expected to handle a transient network error. The split of responsibilities between code-level retries and model-level retries is what makes the difference between a robust agent and a fragile one.

## Section 1: classify the failure first

Before any retry decision, classify the failure. The classification determines the strategy. Five classes I find useful:

1. **Transient infrastructure**: connection reset, DNS timeout, 503 Service Unavailable. The next attempt is likely to succeed without changing anything.
2. **Rate limit**: 429 Too Many Requests, provider-specific rate limit headers. The next attempt will succeed if you wait.
3. **Malformed arguments**: JSON schema validation failure, type mismatch, missing required field. The next attempt with the same arguments will fail identically.
4. **Stale state**: file not found because the path changed, lock contention, optimistic concurrency mismatch. The next attempt may or may not succeed, depending on whether the underlying state changed.
5. **Permanent failure**: 401 unauthorized, 403 forbidden, 404 for an entity that genuinely does not exist. No retry will help.

Each class has a different correct response. Classes 1 and 2 are pure code-level retry. Class 3 requires the model to reformulate. Classes 4 and 5 require feeding the failure back to the model and letting it decide.

```python
class FailureClass:
    TRANSIENT = "transient"
    RATE_LIMIT = "rate_limit"
    BAD_ARGS = "bad_args"
    STALE_STATE = "stale_state"
    PERMANENT = "permanent"

def classify(exc: Exception, status: int | None = None) -> str:
    if status == 429: return FailureClass.RATE_LIMIT
    if status in (401, 403, 404): return FailureClass.PERMANENT
    if status and 500 <= status < 600: return FailureClass.TRANSIENT
    if isinstance(exc, (ConnectionError, TimeoutError)): return FailureClass.TRANSIENT
    if isinstance(exc, (ValueError, KeyError, TypeError)): return FailureClass.BAD_ARGS
    if isinstance(exc, FileNotFoundError): return FailureClass.STALE_STATE
    return FailureClass.PERMANENT  # default to fail-safe
```

The classifier is approximate. When in doubt, default to the most fail-safe class (treat as permanent and let the model see it). False positives in retry classes are worse than false negatives.

## Section 2: pattern one, exponential backoff with jitter

The standard pattern for transient failures and rate limits. Documented to death, often implemented incorrectly.

```python
import random, time

def backoff_retry(fn, max_attempts=4, base=0.5, cap=30.0):
    for attempt in range(max_attempts):
        try:
            return fn()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            cls = classify(e, getattr(e, "status_code", None))
            if cls not in (FailureClass.TRANSIENT, FailureClass.RATE_LIMIT):
                raise
            sleep = min(cap, base * (2 ** attempt)) * (0.5 + random.random())
            time.sleep(sleep)
```

Two things that are easy to get wrong:

The first: not adding jitter. Without jitter, all clients that hit a rate limit at the same time will retry at the same time, immediately re-saturating the service. The `0.5 + random.random()` term spreads retries across a 0.5x to 1.5x window of the deterministic backoff.

The second: not capping the sleep. An exponential at 0.5 base hits 8 seconds at attempt 4, 32 seconds at attempt 6, 128 seconds at attempt 8. For an interactive agent, anything past 30 seconds is effectively a hang. Cap the sleep, then let the agent loop decide whether to keep trying.

## Section 3: pattern two, respecting Retry-After

When a server tells you when to come back, listen. Most rate-limited APIs include a `Retry-After` header (seconds) or a provider-specific header (`X-RateLimit-Reset` as a Unix timestamp).

```python
def smart_backoff(fn, max_attempts=4):
    for attempt in range(max_attempts):
        try:
            return fn()
        except RateLimitError as e:
            if attempt == max_attempts - 1: raise
            ra = e.response.headers.get("Retry-After")
            if ra:
                try:
                    sleep = float(ra)
                except ValueError:
                    # could be HTTP date format
                    from email.utils import parsedate_to_datetime
                    sleep = max(0, (parsedate_to_datetime(ra).timestamp() - time.time()))
            else:
                sleep = min(30, 0.5 * (2 ** attempt)) * (0.5 + random.random())
            time.sleep(sleep)
```

The server knows when its rate limit window resets. Your guess based on exponential backoff is just a guess. Use the server's number when it gives you one.

## Section 4: pattern three, idempotency keys

Retries are only safe for idempotent operations. A POST that creates a resource is not idempotent: two retries of the same POST create two resources. The fix is the idempotency key, a client-generated identifier that the server uses to deduplicate.

```python
import uuid, hashlib, json

def idempotency_key(method: str, path: str, body: dict | None) -> str:
    payload = json.dumps({"m": method, "p": path, "b": body}, sort_keys=True)
    return hashlib.sha256(payload.encode()).hexdigest()[:32]

def safe_post(client, path, body, **kw):
    key = idempotency_key("POST", path, body)
    return backoff_retry(lambda: client.post(path, json=body, headers={"Idempotency-Key": key}, **kw))
```

The key is deterministic given the same arguments, so a retry generates the same key and the server returns the original result instead of creating a duplicate. Stripe, Anthropic's batch API, and most modern APIs support this header.

For tools that talk to APIs without idempotency support, the rule is simple: do not retry. Mark them as non-retriable in the tool registry and let the agent loop decide what to do with the failure.

```python
TOOLS = {
    "read_file":   {"retriable": True,  "fn": read_file},
    "write_file":  {"retriable": False, "fn": write_file},  # not safely retriable
    "send_email":  {"retriable": False, "fn": send_email},
    "search":      {"retriable": True,  "fn": search},
}
```

## Section 5: pattern four, validate before retrying

For the BAD_ARGS class, retrying with the same arguments is pointless. The right move is to validate at the tool boundary and return a structured error to the agent so the model can reformulate.

```python
import jsonschema

def validating_tool(fn, schema):
    def wrapper(**kwargs):
        try:
            jsonschema.validate(kwargs, schema)
        except jsonschema.ValidationError as e:
            return {
                "error": "validation_error",
                "message": e.message,
                "path": list(e.absolute_path),
                "schema_hint": schema.get("description", ""),
            }
        return fn(**kwargs)
    return wrapper

read_file_tool = validating_tool(read_file, {
    "type": "object",
    "required": ["path"],
    "properties": {"path": {"type": "string", "minLength": 1}},
    "description": "Reads a file from disk. Provide an absolute path.",
})
```

The error returned to the agent is structured: `error` is a discriminator, `message` and `path` tell the model what went wrong, `schema_hint` reminds the model what the tool expects. The model then issues a corrected call.

The model is good at this. Three of my own measurements over the last month: validation errors auto-corrected by the model on the next call 87 percent of the time, with no human intervention. The remaining 13 percent split between "the model gives up after one retry and reports back" (acceptable) and "the model goes into a loop trying minor variations" (caught by the loop detector from the debugging post).

## Section 6: pattern five, the synthesized error message

For STALE_STATE failures, the error message itself is part of the recovery. A bare `FileNotFoundError: /tmp/xyz` is less useful to the model than `File /tmp/xyz not found. The directory /tmp exists. Recent files in /tmp: a.txt, b.txt, c.txt. If you meant one of these, retry with the corrected path.`

```python
def stateful_error(exc: Exception, context: dict) -> dict:
    if isinstance(exc, FileNotFoundError):
        path = context.get("path", "")
        from pathlib import Path
        parent = Path(path).parent
        siblings = []
        if parent.exists():
            siblings = [p.name for p in sorted(parent.iterdir())[:20]]
        return {
            "error": "file_not_found",
            "path": path,
            "parent_exists": parent.exists(),
            "parent_contents": siblings,
            "hint": "If you meant one of the parent_contents files, retry with corrected path.",
        }
    return {"error": type(exc).__name__, "message": str(exc)}
```

The synthesized error gives the model enough context to fix the call without a separate `list_directory` round-trip. This pattern, applied across the tool set, cut my agent's average tool-call count per task by about 14 percent. The model wastes fewer turns asking "where is the file" because the error already told it.

## Section 7: pattern six, the circuit breaker

When the same tool fails N times in a row across the entire session (not just one call site), something systemic is wrong. Continuing to retry is a waste of time and money. Open the circuit: refuse new calls to that tool for a cooling-off period and let the model adapt.

```python
import time
from collections import deque

class CircuitBreaker:
    def __init__(self, threshold=5, window_s=60.0, cool_s=120.0):
        self.threshold = threshold
        self.window_s = window_s
        self.cool_s = cool_s
        self.failures = deque()
        self.opened_at = None

    def record_failure(self):
        now = time.time()
        self.failures.append(now)
        while self.failures and now - self.failures[0] > self.window_s:
            self.failures.popleft()
        if len(self.failures) >= self.threshold and self.opened_at is None:
            self.opened_at = now

    def record_success(self):
        self.failures.clear()
        self.opened_at = None

    def is_open(self) -> bool:
        if self.opened_at is None: return False
        if time.time() - self.opened_at > self.cool_s:
            self.opened_at = None  # half-open: allow one attempt
            return False
        return True

breakers = {}  # tool_name -> CircuitBreaker

def call_tool_with_breaker(name, fn, **kwargs):
    cb = breakers.setdefault(name, CircuitBreaker())
    if cb.is_open():
        return {"error": "circuit_open", "tool": name, "hint": f"Tool {name} is temporarily disabled. Try a different approach."}
    try:
        result = fn(**kwargs)
        cb.record_success()
        return result
    except Exception as e:
        cb.record_failure()
        raise
```

The model sees `circuit_open` as a tool result and can replan. In practice it usually picks a different tool or asks the user for help. Both are better outcomes than continuing to fail.

## Section 8: pattern seven, the explicit handoff to the model

When all the code-level retries are exhausted, the failure goes to the model. But not as a Python exception, as a structured tool result the model can reason about.

```python
def dispatch_tool(name, fn, schema, **kwargs) -> dict:
    """Returns either a success result or a structured failure dict."""
    try:
        return backoff_retry(lambda: validating_tool(fn, schema)(**kwargs))
    except Exception as e:
        cls = classify(e, getattr(e, "status_code", None))
        return {
            "error": cls,
            "tool": name,
            "args": kwargs,
            "message": str(e),
            "attempts_made": "exhausted",
            "hint": _hint_for_class(cls),
        }

def _hint_for_class(cls: str) -> str:
    return {
        FailureClass.TRANSIENT: "The service is unreachable. Try a different tool or ask the user.",
        FailureClass.RATE_LIMIT: "Rate limit reached. Wait or use a different tool.",
        FailureClass.BAD_ARGS: "Tool arguments were invalid. Reformulate.",
        FailureClass.STALE_STATE: "The state has changed. Recheck and retry with new info.",
        FailureClass.PERMANENT: "This will not succeed. Try a different approach.",
    }.get(cls, "Unknown failure. Reconsider.")
```

The model now sees a tool result that is itself well-structured. It can read the `error` field, the `hint` field, and decide what to do. In my experience, the model handles these correctly about 80 percent of the time, which is much better than treating tool failures as fatal.

## Section 9: anti-patterns

A short list of retry patterns I have removed from production code.

**Retry forever.** Any `while True: try: ... except: continue` is a bug. Always cap attempts. The cap should be small (3 to 5 for code-level retry, plus the model's own judgment for higher-level retries). Tools that genuinely need many retries should expose that as a parameter, not hide it in a loop.

**Retry without classification.** Catching `Exception` and retrying on everything is the most common version. It hides programming bugs (KeyError, AttributeError) behind backoff and turns them into mysterious slowdowns. Always classify before retrying.

**Retrying non-idempotent calls without idempotency keys.** Discussed above. The combination of "retry on any error" plus "this tool is a POST that creates resources" is how databases get duplicate rows and how systems get duplicate emails.

**Hiding all failures from the model.** A code-level retry that succeeds on the third attempt is fine to hide. A code-level retry that fails on the third attempt should always surface to the model. The model needs to know when its plan is not working so it can replan.

**Globally configured backoff.** Different tools have different acceptable retry budgets. A `read_file` should not retry for 30 seconds; a `web_search` reasonably might. Per-tool config beats a single global value.

## Section 10: the daily metrics I watch

Three numbers, watched in the same JSONL-backed dashboard from the earlier post:

The first: tool failure rate by class. Transient failures should be low (under 1 percent of calls). Rate limit failures should be near zero (if not, raise the rate limit ceiling or add throttling). Bad-args failures should trend down over time as the model learns the tool's quirks; if they stay flat, the tool's schema needs better description text. Stale-state failures vary; a baseline of a few percent is fine.

The second: retries per successful call, distribution. Most calls should succeed on attempt 1 (above 95 percent). The right tail of the distribution shows the cost of unreliable infrastructure or unstable tools. If the median moves above 1, something has degraded.

The third: time spent in retries, per session. Backoff time accumulates. A session that retries 20 times with 2-second sleeps spent 40 seconds doing nothing. If this number creeps up, it usually means a downstream service is degrading and the agent is masking it.

These three numbers, plotted as sparklines, sit on my dashboard. They have caught two production issues in the last month before the rest of the system noticed: an upstream provider degradation (rate of TRANSIENT failures rose from 0.3 percent to 4 percent over an hour), and a tool-schema regression I introduced (rate of BAD_ARGS rose from 0.5 percent to 8 percent immediately after deploy).

The retry strategy is not "make failures invisible." It is "handle failures correctly." The metrics are the difference.

## References

- Stripe idempotency keys: https://stripe.com/docs/api/idempotent_requests
- Google SRE book on retries: https://sre.google/sre-book/handling-overload/
- AWS Architecture blog on jitter: https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- Circuit breaker pattern: https://martinfowler.com/bliki/CircuitBreaker.html
- JSON Schema validation: https://json-schema.org/
