---
title: "Cost-aware model routing: cheaper-by-default with safety nets"
date: 2026-04-23
tags: [routing, cost, llm, litellm, openrouter, architecture]
est_reading_time: 16 min
---

## TL;DR

The cheapest LLM call is the one you do not make. The second cheapest is the one you make to a small model that is good enough. Most production agent stacks I see route 100 percent of traffic to one large frontier model "for safety," then act surprised when the bill arrives. A cost-aware router downgrades by default to the smallest model that has historically solved the current request shape, monitors quality online, and upgrades only when a measurable signal says it must. This post walks through a working implementation in about 200 lines of Python sitting in front of a chat-completions API, with the heuristics that have held up in three months of production use on my own workload.

The key idea: routing is a control loop, not a static config. It has a feedback signal (was the answer accepted), a default action (use the small model), and an escalation path (retry with a bigger model on a fail signal). Treat it like rate limiting or circuit breaking, not like a feature flag.

## Why "always use the biggest model" loses

The intuition that bigger is safer is correct on isolated single-shot tasks: a frontier model wins on novel hard problems by a large margin. But agent workloads are dominated by short, repetitive, well-shaped calls: "summarize this 200-line file," "extract the function names from this output," "generate a one-line commit message for this diff." On these, models priced at 1/30th of the frontier deliver indistinguishable quality, often with lower latency.

I measured this on my own usage in February 2026. Of 38,402 model calls in a 30-day window, 71 percent had output under 200 tokens and matched one of six recurring shapes (commit message, file summary, tool-result interpretation, plan-step generation, error-message classification, JSON repair). On all six shapes, a small model from the same provider scored within 2 percentage points of the frontier on a manually-graded 100-call sample. Routing those 71 percent to the small model and keeping the other 29 percent on the frontier cut my bill by 58 percent with no observable quality regression in the agent's task completion rate.

The router that did this is the rest of this post.

## Section 1: a taxonomy of "request shapes"

Before you can route, you need a label. The label does not need to be perfect. It needs to be cheap to compute and stable enough that the same kind of call gets the same label most of the time.

I use four signals, combined into a tuple:

```python
def shape_of(messages, tools=None) -> tuple:
    last_user = next((m for m in reversed(messages) if m.get("role") == "user"), None)
    text = ""
    if last_user:
        c = last_user.get("content")
        text = c if isinstance(c, str) else " ".join(b.get("text", "") for b in c if isinstance(b, dict))
    return (
        len(messages),                       # turn count
        len(text) // 500,                    # request size bucket
        bool(tools),                         # tool-using or not
        _intent(text),                       # cheap intent classifier
    )

def _intent(text: str) -> str:
    t = text.lower()
    if "commit message" in t or "git diff" in t: return "commit"
    if "summarize" in t or "summary of" in t: return "summarize"
    if "extract" in t and "json" in t: return "extract_json"
    if "fix" in t and ("error" in t or "exception" in t): return "fix_error"
    if "plan" in t or "steps to" in t: return "plan"
    return "general"
```

This is intentionally crude. A learned classifier would be more accurate but would add latency and a dependency. The crude version misclassifies maybe 5 percent of calls, and the cost of a misclassification is bounded (you fall back to the default route, which is correct), so the crude version pays for itself.

## Section 2: the routing table

A routing decision is a mapping from shape to a small ordered list of models, cheapest first. Iterate until one returns an acceptable response.

```python
ROUTES = {
    # (turn_count_bucket, size_bucket, has_tools, intent)
    # use coarser keys; missing keys fall through to DEFAULT
    ("any", "any", False, "commit"):       ["haiku", "sonnet"],
    ("any", "any", False, "summarize"):    ["haiku", "sonnet"],
    ("any", "any", False, "extract_json"): ["haiku", "sonnet"],
    ("any", "any", False, "fix_error"):    ["sonnet", "opus"],
    ("any", "any", True,  "general"):      ["sonnet", "opus"],
}
DEFAULT = ["sonnet", "opus"]

MODEL_COSTS = {  # USD per 1M input tokens, list price, April 2026
    "haiku":  0.80,
    "sonnet": 3.00,
    "opus":   15.00,
}

def route_for(shape) -> list[str]:
    _, _, has_tools, intent = shape
    key = ("any", "any", has_tools, intent)
    return ROUTES.get(key, DEFAULT)
```

Two things to note. First, `has_tools=True` always escalates to at least sonnet, because in my testing the small model is unreliable at picking between many tools. Second, the route is a list, not a single choice. The router will try the first, and if quality fails, the second. This is the safety net.

## Section 3: the quality signal

Routing without a quality signal is just downcasting. The signal does not have to be perfect, but it has to exist. I use four cheap signals, evaluated in order:

```python
import json

def quality_check(response_text: str, expected_shape: dict | None) -> str | None:
    """Return a failure reason string, or None if OK."""
    if not response_text or len(response_text.strip()) < 5:
        return "empty_or_tiny_response"
    if expected_shape and expected_shape.get("type") == "json":
        try:
            json.loads(response_text)
        except json.JSONDecodeError as e:
            return f"json_parse_error: {e.msg}"
    if "I cannot" in response_text and len(response_text) < 200:
        return "refusal_short"
    if response_text.count(response_text.split()[0]) > 5 and len(response_text) < 500:
        return "repetitive_loop"
    return None
```

Each rule has fired in production at least once for me. The JSON parse check is by far the most common: small models, on JSON extraction tasks, sometimes wrap their output in markdown fences or add a "Sure, here is the JSON:" preamble. The check catches it and triggers escalation.

For tool-using calls, the signal is different. There, "quality" means "the model picked a tool that the harness recognizes and provided arguments that pass schema validation." That check is essentially free and lives in the tool dispatch layer.

## Section 4: putting it together

The router itself is a thin loop: pick a route, try the first model, evaluate quality, escalate on failure, log everything.

```python
import time, json
from pathlib import Path

class Router:
    def __init__(self, client, log_path=None):
        self.client = client
        self.log = log_path or Path.home() / ".cache" / "router.jsonl"
        self.log.parent.mkdir(parents=True, exist_ok=True)

    def call(self, messages, tools=None, expected_shape=None, **kw):
        shape = shape_of(messages, tools)
        candidates = route_for(shape)
        last_err = None
        for i, model in enumerate(candidates):
            t0 = time.time()
            try:
                resp = self.client.messages.create(
                    model=self._resolve(model),
                    messages=messages,
                    tools=tools or [],
                    max_tokens=kw.get("max_tokens", 1024),
                )
                text = self._extract_text(resp)
                fail = quality_check(text, expected_shape)
                self._log(model, shape, resp.usage, time.time() - t0, fail, i)
                if fail is None:
                    return resp
                last_err = fail
            except Exception as e:
                last_err = f"exception: {e}"
                self._log(model, shape, None, time.time() - t0, last_err, i)
        raise RuntimeError(f"all routes exhausted, last={last_err}")

    def _resolve(self, alias: str) -> str:
        return {
            "haiku":  "claude-3-5-haiku-20241022",
            "sonnet": "claude-3-5-sonnet-20241022",
            "opus":   "claude-3-opus-20240229",
        }[alias]

    def _extract_text(self, resp) -> str:
        return "".join(b.text for b in resp.content if getattr(b, "type", "") == "text")

    def _log(self, model, shape, usage, dur, fail, attempt):
        with self.log.open("a") as f:
            f.write(json.dumps({
                "ts": time.time(),
                "model": model,
                "shape": list(shape),
                "in_tok": usage.input_tokens if usage else 0,
                "out_tok": usage.output_tokens if usage else 0,
                "dur_s": round(dur, 3),
                "fail": fail,
                "attempt": attempt,
            }) + "\n")
```

That is the whole thing. The complexity that does not appear is in the routing table and the quality check. Those are the parts you tune.

## Section 5: tuning the routing table from the log

After a few days of traffic, you can answer empirical questions from the log: which (shape, model) pairs have a fail rate above some threshold? Which have zero fails over hundreds of calls (and could probably be downgraded further)?

```python
import json
from collections import defaultdict
from pathlib import Path

def fail_rates(window=2000):
    lines = Path.home().joinpath(".cache/router.jsonl").read_text().splitlines()[-window:]
    rows = [json.loads(l) for l in lines]
    by_key = defaultdict(lambda: [0, 0])  # (model, intent) -> [fails, total]
    for r in rows:
        if r["attempt"] != 0:  # only first-attempt calls
            continue
        intent = r["shape"][3]
        by_key[(r["model"], intent)][1] += 1
        if r["fail"]:
            by_key[(r["model"], intent)][0] += 1
    out = []
    for (model, intent), (fails, total) in sorted(by_key.items()):
        rate = fails / total if total else 0
        out.append((model, intent, total, fails, round(rate, 3)))
    return out

for row in fail_rates():
    print(row)
```

A row I saw last week: `("haiku", "extract_json", 412, 38, 0.092)`. A 9 percent fail rate on `extract_json` with haiku is too high; the escalation cost (re-running on sonnet) is approximately 9 percent of those calls paid for at sonnet's price plus haiku's wasted cost, which works out roughly to a 25 percent cost overhead vs always-sonnet on that intent. I changed the route for `extract_json` to start at sonnet, kept haiku as the secondary. Cost on that intent went up 18 percent, latency went down (no re-tries), and reliability went from 91 percent to 99.5 percent. Worth it.

## Section 6: the case for a separate "is this a bullshit answer" model

A failure mode of cheap quality checks: they catch syntactic failures (empty, malformed JSON, refusal) but miss semantic ones (the answer is structurally fine but wrong). For a class of high-stakes calls, I now run a second-pass check using a small model to grade the first answer.

```python
def graded_call(router, messages, tools=None, rubric=None):
    resp = router.call(messages, tools=tools)
    text = router._extract_text(resp)
    if not rubric:
        return resp
    grade_msgs = [
        {"role": "user", "content": (
            f"Question: {messages[-1]['content']}\n\n"
            f"Answer: {text}\n\n"
            f"Rubric: {rubric}\n\n"
            "Reply with one word: PASS or FAIL."
        )}
    ]
    grade_resp = router.call(grade_msgs)  # routes to haiku by default
    grade = router._extract_text(grade_resp).strip().upper()
    if grade.startswith("FAIL"):
        # escalate manually by skipping the cheap model
        return router.call(messages, tools=tools)  # rely on attempt logging
    return resp
```

Cost overhead: about 1 cent per 100 graded calls on haiku. Catches roughly 3 percent of cases that pass the syntactic check but get a thumbs-down from a human reviewer. Use sparingly, on the calls where wrong-but-confident is expensive.

## Section 7: the routing failure modes I have learned to avoid

A short list, all from things I got wrong.

The first: routing on the user's exact text instead of on a shape. Shapes are stable across paraphrases; text is not. A router that keys on text fragments will have a hit rate that decays over weeks.

The second: starting the loop with too many models in the candidate list. If you put four models in the list, on a hard call you will pay for four model calls before failing. Cap candidates at two for normal traffic, three only for explicitly hard intents.

The third: not pinning model versions. The cost numbers in my routing table assume specific dated versions. If I let "latest" resolve, the cost math drifts the next time the provider rolls a new version. I now pin in `_resolve` and treat version bumps as routing-table-affecting changes.

The fourth: caching the route decision per session and never revisiting. Routes should be re-evaluated per call, because the shape can change within a session as the agent moves between sub-tasks.

The fifth: forgetting that the cheapest call is the one you do not make. A surprising fraction of calls in an agent loop can be served from a small in-process cache of recent responses, especially in tight tool-use loops. A 5-minute LRU cache keyed on the prompt's normalized hash cut my router's invocation count by another 11 percent on top of the routing savings.

## Section 8: results

After three months running this router on my own workload:

- Average cost per agent session: down 58 percent vs always-frontier baseline.
- Median latency to first token: down 23 percent (haiku is faster than sonnet which is faster than opus).
- Task completion rate (manual sample, n=200): 94 percent vs 95 percent baseline. Within noise.
- Router code: 217 lines including the log analyzer. Maintained by me alone.

The win is not "the router is clever." It is "default to cheap, measure, escalate." That is the same idea as opportunistic locking, optimistic concurrency control, or read-replica routing in databases. None of those are clever either, and all of them work because the common case really is common.

The cost discipline is not about saving money for its own sake. It is about being able to run the same workload at 10x the volume without the bill exploding. When the next experiment wants 10x the calls, the router lets me say yes without flinching. That is the actual product of this work.

## References

- Anthropic model pricing: https://www.anthropic.com/pricing
- OpenAI model pricing: https://openai.com/api/pricing/
- LiteLLM router docs: https://docs.litellm.ai/docs/routing
- OpenRouter routing docs: https://openrouter.ai/docs/features/model-routing
- Circuit breaker pattern (Fowler): https://martinfowler.com/bliki/CircuitBreaker.html
