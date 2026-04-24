# Cache-Key Design For Prompt + Context + Tool-Set Hashing

The first time you build an LLM-backed system you do not cache. You call the model on every request, you eat the latency, you pay the bill, and you ship. The second time, somebody points at the bill and you bolt on a cache. The cache key is `hash(prompt)` and you congratulate yourself. The third time, you discover that your cache hit rate is 4% in production despite the fact that 70% of your traffic is "obviously the same question," and you start the long, painful walk into actually understanding what a cache key for an LLM call needs to contain.

This post is about that walk. Specifically, about the design of cache keys for agent systems where the input to a model call is not just a prompt — it is a prompt, a context window full of prior turns and tool results, a tool-set definition, a set of sampling parameters, and a model identity. Hashing this correctly is harder than it looks, and getting it wrong produces two distinct flavors of bug: silent cache poisoning (you serve the wrong answer) and silent cache miss (you serve the right answer but pay full price). Both are expensive. The poisoning kind is also a correctness bug.

## The naive key is wrong in at least four ways

`key = sha256(prompt)` is wrong because:

1. **Two requests with the same prompt but different tool definitions produce different model behavior.** If yesterday's request had `fetch_url` available and today's does not, the model's response is allowed to differ. Caching across that boundary serves stale tool calls.
2. **Two requests with the same prompt but different system messages produce different model behavior.** Obvious in retrospect. Frequently forgotten when the system message lives in a different config file from the prompt template.
3. **Two requests with the same prompt but different sampling parameters can produce different responses.** `temperature=0` vs `temperature=0.7` is not the same call. `top_p`, `presence_penalty`, `max_tokens`, `stop` sequences — all of these affect output and must be in the key.
4. **Two requests with the same prompt but different model versions are different calls.** `gpt-4-2024-05-13` and `gpt-4-2024-08-06` are different functions. A cache that ignores model identity is a cache that lies after every model upgrade.

Any of these, alone, is enough to make a production cache wrong. In combination they make it impossible to reason about. You will see "ghost regressions" where the model output suddenly changes for no apparent reason; the apparent reason is that someone rotated the system prompt and the cache happily served output computed against the old one for the next 72 hours.

## The full key, written out

A correct cache key for an agent system contains, at minimum:

```
key = hash(
  model_id,
  model_version,
  system_message_normalized,
  messages_normalized,
  tool_definitions_canonicalized,
  tool_choice,
  sampling_params_canonicalized,
  response_format,
  cache_namespace,
)
```

Each of these has subtleties. The interesting ones:

### `messages_normalized`

The conversation history. Subtleties:

- **Whitespace.** `"hello"` and `"hello "` and `"hello\n"` should probably be the same key for most use cases, but for code-completion they are emphatically not. Pick a normalization policy and write it down. The default I use: collapse runs of internal whitespace to a single space; strip leading and trailing whitespace from each message; do not normalize whitespace inside fenced code blocks.
- **Tool call results.** A previous turn may contain a tool result. That tool result may contain a timestamp, a request ID, a session token. If those vary across requests but the *meaning* is the same, you will see zero cache hits. You need a tool-result normalizer that strips known-volatile fields. This is per-tool: `get_weather` results have a `timestamp` field that is volatile; `read_file` results do not.
- **Image / multimodal inputs.** Hash the bytes, not the URL. A URL pointing to mutable content is not stable. If you must use a URL, fetch the bytes and hash them; the cost of the HEAD or GET is usually less than the cost of a cache miss.

### `tool_definitions_canonicalized`

The tool-set is part of the model's input. Two cache key disasters live here:

- **JSON key ordering.** `{"name":"x","desc":"y"}` and `{"desc":"y","name":"x"}` hash to different values under naive hashing but represent the same tool. Canonicalize: sort keys recursively, use a single JSON serializer with stable settings.
- **Tool order.** Most APIs accept tools as an ordered list, but the model's behavior is approximately invariant to tool order *for tool selection*, not for response framing. The conservative choice: include tool order in the key. The aggressive choice: sort tools by name and accept that you might serve a slightly different response across reorderings. Pick one and be consistent.

A subtler point: if your tool definitions include descriptions that contain volatile information (a current date, a version string, a user ID), every request mints a fresh key. I have seen production systems where the cache hit rate was 0% because the tool descriptions contained `"as of {today}"` interpolated at request time. The fix is to either remove the volatile substring from the tool description or strip it before hashing.

### `sampling_params_canonicalized`

Numeric parameters are floating point. `0.7` and `0.7000000001` are different bytes and hash differently. Round to a fixed precision before hashing — for `temperature`, three decimal places is plenty. For `top_p`, four. For `max_tokens` and `seed`, integers; no rounding needed.

If a parameter is unset versus set to its default value, normalize. The OpenAI API treats `temperature: null` and `temperature: 1.0` as the same call; your hash function should too. Define your defaults in one place and fill them in before hashing.

### `cache_namespace`

The escape hatch. A short string that you can bump to invalidate the entire cache without redeploying the cache layer. You will need this. Use it for:

- Bug fixes in your normalization (you discovered the whitespace policy was wrong, every old key is suspect).
- Behavioral changes you want to flush even though no input changed (you noticed the model started hallucinating last Tuesday).
- A/B tests where you want one cohort to bypass the cache.

The namespace is mixed into the hash but is also human-readable in your config, so changes are auditable.

## What to actually hash, byte-for-byte

The single most useful thing you can do is write down the canonicalization function and put it under test. Pseudocode:

```python
import hashlib, json

def canonicalize_messages(msgs, tool_normalizers):
    out = []
    for m in msgs:
        c = {"role": m["role"]}
        if m["role"] == "tool":
            tool_name = m.get("name", "")
            normalizer = tool_normalizers.get(tool_name, lambda x: x)
            c["name"] = tool_name
            c["content"] = normalizer(m["content"])
        else:
            c["content"] = normalize_whitespace(m.get("content", ""))
        if "tool_calls" in m:
            c["tool_calls"] = sorted(
                ({"name": tc["name"], "arguments": canonicalize_json(tc["arguments"])}
                 for tc in m["tool_calls"]),
                key=lambda x: x["name"],
            )
        out.append(c)
    return out

def canonicalize_tools(tools):
    return sorted(
        ({"name": t["name"],
          "description": t["description"].strip(),
          "parameters": canonicalize_json(t["parameters"])}
         for t in tools),
        key=lambda x: x["name"],
    )

def canonicalize_sampling(p):
    out = {}
    for k in ("temperature", "top_p", "presence_penalty", "frequency_penalty"):
        if k in p and p[k] is not None:
            out[k] = round(float(p[k]), 4)
    for k in ("max_tokens", "seed"):
        if k in p and p[k] is not None:
            out[k] = int(p[k])
    if "stop" in p and p["stop"]:
        out["stop"] = sorted(p["stop"])
    return out

def cache_key(req, namespace, tool_normalizers):
    payload = {
        "ns": namespace,
        "model": req["model"],
        "system": normalize_whitespace(req.get("system", "")),
        "messages": canonicalize_messages(req["messages"], tool_normalizers),
        "tools": canonicalize_tools(req.get("tools", [])),
        "tool_choice": req.get("tool_choice", "auto"),
        "sampling": canonicalize_sampling(req.get("sampling", {})),
        "response_format": req.get("response_format", {"type": "text"}),
    }
    blob = json.dumps(payload, sort_keys=True, separators=(",", ":")).encode()
    return hashlib.sha256(blob).hexdigest()
```

The reason to write it like this — not as a one-line hash of the request body — is that every line is a place where a bug will hide. When the cache hit rate is wrong you can step through and inspect the canonical form. When you find the bug, you bump the namespace, redeploy, and move on.

## When *not* to cache

A correct cache key tells you *whether two requests are the same*. It does not tell you *whether the response should be cached*. Some responses must not be cached even if the key matches:

- **Responses that include the current time, current date, or "today's" anything.** If the model's output includes "as of today, ..." then yesterday's response is wrong even though the input is identical.
- **Responses to non-deterministic generation (temperature > 0).** These are still cacheable for cost reasons — you save tokens — but you should know that you are serving a single sample of a distribution, not "the answer." Sometimes that is fine; sometimes it makes the system feel uncanny because users notice that the same question always gets the same answer when they expect variation.
- **Responses to tool-calling turns where the tool will be called against external state.** If the model output is "call `transfer_money(to='alice', amount=100)`," caching that response is correct (the model decided to call the tool); but make sure your downstream actually re-executes the tool call. Caching the *tool result* is a different question, governed by the tool's own semantics.
- **Responses you intend to use as training data.** A cache hit is not a fresh sample. If your data flywheel depends on diverse outputs, caching collapses your distribution.

The right mental model: the cache stores the model's response to a fully specified input. It does not store an answer to a question. The question of "does this answer still apply" is a separate question, and it is yours to answer per use case.

## Two-tier caching: prefix vs full

Frontier models support prompt prefix caching natively now. If your first 4096 tokens are stable across requests, the provider will charge you a fraction of the input cost for those tokens. This is *not the same* as your application-level response cache. It saves input cost; it does not save the round trip; it does not save output cost.

The right architecture has both:

1. **Application-level response cache** keyed on the full canonical input. On hit, return cached response. Saves the entire call.
2. **Provider-level prefix cache** triggered by stable prefixes in your prompt. Saves input tokens on cache misses.

To make the prefix cache fire reliably, structure your prompt so that the *stable* parts come first: system message, tool definitions, long-lived context. Then the volatile parts: recent messages, current user query. Models that support prefix caching look for the longest matching prefix, not arbitrary substrings.

A common mistake: putting the timestamp or request ID at the top of the system prompt "for logging." This destroys prefix-cache hit rate. Put it at the bottom, or strip it out before sending.

## Diagnosing a cache that does not work

When the cache hit rate is lower than you expect, the diagnostic procedure is:

1. **Log the canonical input** (or its hash) for every request and every cache lookup.
2. **Find pairs of requests that you believe should hit the same key but didn't.**
3. **Diff the canonical inputs.** The diff will show you the field that is varying.
4. **Either fix the canonicalization or accept that those requests are genuinely different.**

The diff will surprise you. The first time I did this exercise I found that one out of every five requests was unique because the system prompt contained `"You are running on host {hostname}"` and the autoscaler was rotating hosts every few minutes. The fix was a one-line change to drop the hostname interpolation. The cache hit rate jumped from 11% to 58% overnight.

## The summary card

If you remember nothing else:

- The cache key includes the model, the system message, every message, every tool definition, every sampling parameter, the response format, and a namespace.
- Canonicalize each of those before hashing: sort JSON keys, normalize whitespace, round floats, sort tool lists.
- Normalize tool-result content per tool, stripping known-volatile fields.
- Bump the namespace whenever the canonicalization changes or you want a manual flush.
- Layer the application response cache on top of the provider's prefix cache; structure prompts stable-first to make the prefix cache fire.
- Decide caching policy per use case: a matching key is necessary, not sufficient, for serving a cached response.

The reason cache-key design is worth this much attention is that it sits exactly at the boundary where correctness, latency, and cost all collide. A wrong key serves wrong answers. A too-strict key wastes money. A too-loose key fails compliance. Get it right once, write it down, put it under test, and you will spend the next two years gradually appreciating how much pain you avoided.
