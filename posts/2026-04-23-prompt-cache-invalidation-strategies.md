---
title: "Prompt cache invalidation strategies that actually save money"
date: 2026-04-23
tags: [prompt-cache, llm, cost, architecture, anthropic, openai]
est_reading_time: 14 min
---

## TL;DR

Prompt caching makes the first 1k tokens of your prompt 10x cheaper to re-send and several times faster to first-token, but only if the cache key (the exact prefix bytes) is stable across calls. Most of the cache misses I see in real systems are not paid for by the model provider, they are paid for by careless prompt assembly: a timestamp interpolated near the top, a tool list reordered by a `dict` iteration, a system message that grows by one line per turn. This post walks through the five invalidation patterns I keep hitting, with runnable code that reproduces each one against the public Anthropic and OpenAI APIs, and the rules I now apply to keep cache hit rate above 90 percent.

The short version: treat the cacheable prefix as a content-addressed blob. Anything that changes per call goes after the cache breakpoint, never before it. Verify with the `cache_read_input_tokens` field, not with vibes.

## Why this matters in dollars

The list price for cache reads on Anthropic's Claude family is 0.1x the list price of fresh input tokens, and on OpenAI's `gpt-4o` family it is 0.5x. In a long-running agent loop where the system prompt plus tool definitions plus repository context can easily run 30k tokens, a single cache miss on the prefix is the cost of roughly 10 to 30 cache hits depending on provider. Run an agent for 200 turns and the difference between a 95 percent prefix hit rate and a 50 percent prefix hit rate is, on the math alone, somewhere between 3x and 6x of the input bill.

I started measuring this in February 2026 because my proxy logs showed input-token spend climbing faster than turn count. The culprit was almost never the user message. It was the system prompt being rebuilt from a dict whose key order was non-deterministic across Python 3.11 processes restarted by my orchestrator.

## Section 1: how the cache key is actually computed

Both major providers document this, but the docs are easy to misread. The cache key is, in effect, a hash of the byte sequence from the start of the request up to a cache breakpoint. On Anthropic, the breakpoint is explicit: you set `cache_control: {type: "ephemeral"}` on the last block you want included. On OpenAI, the breakpoint is implicit, automatic prefix caching kicks in for prompts at or above 1024 tokens and the matched prefix length is rounded down to a 128-token boundary.

Two practical consequences:

1. On Anthropic, a single byte difference anywhere before the last `cache_control` marker invalidates the entire cached prefix. There is no partial match.
2. On OpenAI, the same is true within a 128-token chunk. Change one character in the first 128 tokens, you lose the whole prefix. Change one character in token 900 and you keep the first 768 tokens cached, lose the rest.

This is why I keep saying "treat the prefix as content-addressed." The hash boundary is real and unforgiving.

Here is the smallest possible reproducer, against Anthropic, that demonstrates a cache hit on the second call:

```python
# pip install anthropic>=0.40
import os
from anthropic import Anthropic

client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

LARGE_SYSTEM = "You are a careful assistant.\n" + ("Detail rule.\n" * 600)

def call(user_msg: str):
    resp = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=64,
        system=[
            {
                "type": "text",
                "text": LARGE_SYSTEM,
                "cache_control": {"type": "ephemeral"},
            }
        ],
        messages=[{"role": "user", "content": user_msg}],
    )
    u = resp.usage
    print(
        f"input={u.input_tokens} "
        f"cache_create={u.cache_creation_input_tokens} "
        f"cache_read={u.cache_read_input_tokens}"
    )

call("First call, primes the cache.")
call("Second call, should read from cache.")
```

On a fresh API key the first call shows `cache_create` roughly equal to the system prompt size and `cache_read=0`. The second call, issued within the cache TTL (5 minutes for Anthropic ephemeral cache), shows `cache_create=0` and `cache_read` roughly equal to the system prompt size. If the second number is zero, you have a bug in your prompt assembly. Treat that as a hard signal.

## Section 2: invalidation pattern one, the timestamp at the top

This is the most common bug I see, and it is almost always introduced by a well-intentioned "let the model know what time it is" line in the system prompt.

```python
# WRONG: invalidates cache every call
def build_system_wrong():
    from datetime import datetime
    return f"You are an assistant. Current time: {datetime.utcnow().isoformat()}\n" + LARGE_RULES
```

Every call moves the timestamp by at least a microsecond. Cache hit rate: zero. The fix is to put the timestamp after the cache breakpoint, in the user message or in a non-cached system block:

```python
# RIGHT: timestamp lives after the cache breakpoint
def build_messages(user_msg: str):
    from datetime import datetime
    return {
        "system": [
            {"type": "text", "text": LARGE_RULES, "cache_control": {"type": "ephemeral"}},
        ],
        "messages": [
            {
                "role": "user",
                "content": f"<context time=\"{datetime.utcnow().isoformat()}\">\n{user_msg}",
            }
        ],
    }
```

The model still sees the timestamp. The cache still hits. This is the core pattern: cacheable content is everything that does not change between calls. Everything else lives downstream of the breakpoint.

## Section 3: invalidation pattern two, dict iteration order in tool definitions

Python's `dict` preserves insertion order since 3.7, but `json.dumps` of a `dict` whose keys were added in a different order across runs will produce different bytes. I have seen this break caching for a tool list assembled from a plugin registry whose load order depended on filesystem inode order.

Reproducer:

```python
import json, hashlib

def hash_prefix(tools):
    payload = json.dumps({"tools": tools})
    return hashlib.sha256(payload.encode()).hexdigest()[:12]

tools_a = [
    {"name": "search", "description": "search the web", "input_schema": {"type": "object"}},
    {"name": "read_file", "description": "read a file", "input_schema": {"type": "object"}},
]
tools_b = list(reversed(tools_a))

print(hash_prefix(tools_a))  # e.g. 0f3c2a1d4b81
print(hash_prefix(tools_b))  # different hash, different cache key
```

Two fixes, in order of preference:

1. Sort tools by name before serializing. Make this a property of your tool registry, not of every call site.
2. If sorting is not possible (tool order is semantic), pin the order in a static manifest file and load from that.

I now run a startup assertion in my agent harness:

```python
def assert_tools_canonical(tools):
    names = [t["name"] for t in tools]
    assert names == sorted(names), f"tools not sorted: {names}"
```

## Section 4: invalidation pattern three, growing context windows

The agent loop appends a new tool result to the messages list each turn. If your cache breakpoint is on the last system block and your messages list is the part after the breakpoint, this is fine, the messages part is never cached anyway. But if you are using Anthropic's multi-breakpoint feature (up to 4 cache breakpoints) and putting one on the second-to-last user message, you need to be careful: every new message past that breakpoint forces a cache create on the next-to-last cached block.

The pattern that works for long agent loops is to place a breakpoint at the system + tools boundary (always cache hit after turn 1), and a second breakpoint at the boundary between "old turns we are confident we will not modify" and "current turn." Move the second breakpoint forward only when you have several stable turns behind it.

```python
def messages_with_two_breakpoints(history, current_user):
    # history is a list of {"role": ..., "content": ...}
    # last 2 turns are the moving window; everything older gets cached
    if len(history) < 4:
        return history + [{"role": "user", "content": current_user}]
    stable, moving = history[:-2], history[-2:]
    # mark last block of stable as cache breakpoint
    stable_marked = stable[:-1] + [
        {
            **stable[-1],
            "content": [
                {"type": "text", "text": stable[-1]["content"], "cache_control": {"type": "ephemeral"}}
            ],
        }
    ]
    return stable_marked + moving + [{"role": "user", "content": current_user}]
```

The exact threshold (here, "older than 2 turns") is empirical. I tuned mine by logging `cache_read_input_tokens` per turn for a week and picking the threshold that maximized the ratio.

## Section 5: invalidation pattern four, model version pinning

The cache is keyed not just by prefix bytes but by model version. `claude-3-5-sonnet-20240620` and `claude-3-5-sonnet-20241022` do not share a cache, even with byte-identical prompts. This sounds obvious until you see your fleet's hit rate drop on a Tuesday because someone pushed a config change from `claude-3-5-sonnet-latest` to a pinned version, or the other way around.

Rule I now follow: pin model versions in a single config file, never resolve `*-latest` aliases at call time, and treat a model version bump as a known cache-cold event in the rollout plan. The first 5 minutes after the bump will have inflated input cost. Budget for it.

```python
# config.py
MODEL = "claude-3-5-sonnet-20241022"  # pin; bumps are intentional, dated
```

## Section 6: invalidation pattern five, whitespace and content-type drift

This is the one that took me longest to find. A library upgrade silently changed the JSON serializer used by the SDK from one that emits `"key": "value"` (space after colon) to one that emits `"key":"value"`. The model output was identical. The cache hit rate fell off a cliff.

The defensive move: do not let the SDK build the cacheable bytes for you if you can help it. Build them yourself, hash them, and compare hashes across runs in your test suite.

```python
# tests/test_prompt_stability.py
import hashlib, json
from myapp.prompts import build_cacheable_system

def test_cacheable_prefix_hash_pinned():
    payload = build_cacheable_system()
    h = hashlib.sha256(payload.encode("utf-8")).hexdigest()
    # If this assertion fails, you changed the cacheable prefix.
    # Either accept the cache-cold cost and update the hash,
    # or revert the change.
    assert h == "9f1c0a6b3d2e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f90"
```

I run this test in CI. It has saved me from two silent regressions in three months.

## Section 7: measuring hit rate without staring at logs

You want a single number you can put on a dashboard. The right one is `cache_read_input_tokens / (cache_read_input_tokens + cache_creation_input_tokens + uncached_input_tokens)`, computed over a rolling window. Here is the smallest viable collector:

```python
import json, sys
from pathlib import Path

LOG = Path.home() / ".cache" / "llm-usage.jsonl"

def record(usage_obj, model):
    entry = {
        "model": model,
        "in": getattr(usage_obj, "input_tokens", 0),
        "cache_create": getattr(usage_obj, "cache_creation_input_tokens", 0) or 0,
        "cache_read": getattr(usage_obj, "cache_read_input_tokens", 0) or 0,
        "out": getattr(usage_obj, "output_tokens", 0),
    }
    LOG.parent.mkdir(parents=True, exist_ok=True)
    with LOG.open("a") as f:
        f.write(json.dumps(entry) + "\n")

def hit_rate(window=500):
    lines = LOG.read_text().splitlines()[-window:]
    rows = [json.loads(l) for l in lines]
    read = sum(r["cache_read"] for r in rows)
    create = sum(r["cache_create"] for r in rows)
    fresh = sum(r["in"] for r in rows)
    total = read + create + fresh
    return 0.0 if total == 0 else read / total

if __name__ == "__main__":
    print(f"{hit_rate():.3f}")
```

Wire `record(resp.usage, MODEL)` into every call site. Watch the trend, not the instantaneous value. A healthy agent fleet sits between 0.85 and 0.97 depending on how aggressive the agent's tool use is. Below 0.7, something is wrong.

## Section 8: a checklist I run before merging any prompt change

Pulled from the patterns above. None of these are clever. All of them have caught real bugs in my own code in the last two months.

- Is anything before the cache breakpoint a function of wall-clock time? If yes, move it after.
- Is the tool list sorted by a stable key before serialization? If no, sort it.
- Is the model version pinned, or am I resolving an alias? If alias, pin it.
- Does my prompt-stability test pin the SHA256 of the cacheable prefix? If no, add it.
- After this change, will the first call be a cache create? If yes, did I budget for the cost and the latency spike?

The discipline is boring. The savings are not. On a workload that was costing me roughly 18 USD per day on input tokens before any of this work, the same workload now costs about 4 USD per day. The output cost is unchanged because output is never cached. The model quality is identical because the bytes the model sees are identical, only the path through the provider's infrastructure changed.

## Section 9: the failure modes I am still living with

Two patterns I have not solved cleanly yet, recorded here as honest open questions rather than as advice.

The first is multi-tenant prompt assembly. When the same agent serves multiple end users and each user has a per-user system instruction, the natural place to put that instruction is in the system block, which destroys the cache between users. Putting it in the user message keeps the cache hot but bloats every turn with a re-statement of the per-user instruction. The least bad option I have found is to keep a small per-user prefix at the top of each user turn and accept the 5 to 10 percent token bloat. Open to better ideas.

The second is provider failover. If my primary provider is rate-limited and I fail over to a different provider mid-conversation, I lose the cache entirely. Worse, I have to re-budget the conversation against the new provider's pricing. I currently solve this by failing over only at session boundaries, never mid-session, and accepting a higher tail latency in exchange. A smarter router would predict the rate-limit risk before the session starts and pick the cheapest provider that can hold the session for its expected length, but I have not built that yet.

Both of these are reminders that prompt caching is not a free optimization. It is a constraint on how you assemble prompts, and like any constraint it has shapes you can express cheaply and shapes you cannot. The job of the prompt-assembly layer is to push as much of the workload as possible into the cheap shapes, and to be honest about which workloads do not fit.

## References

- Anthropic prompt caching docs: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- OpenAI prompt caching docs: https://platform.openai.com/docs/guides/prompt-caching
- Anthropic pricing (cache read multiplier): https://www.anthropic.com/pricing
- OpenAI pricing (cached input rate): https://openai.com/api/pricing/
- Python `dict` ordering guarantees: https://docs.python.org/3/whatsnew/3.7.html
