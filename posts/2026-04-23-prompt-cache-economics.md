---
title: "The economics of prompt caching: why your token costs are not what you think"
date: 2026-04-23
tags: [economics, prompt-cache, claude, gpt, agents]
est_reading_time: 8 min
---

If you read your invoice from a frontier model provider and divide total tokens by the headline per-million price, you will overestimate your cost by something between 3x and 10x. The reason is prompt caching, and most people set it up by accident, badly, and leave 80% of the savings on the table.

This is the post I wish someone had handed me when I started running agent loops continuously instead of one-shot prompts.

## What "prompt caching" actually is

Both Anthropic and OpenAI now expose a server-side cache for the prefix of your prompt. You mark a section of input as cacheable; on the first request, the provider hashes that prefix, stores the attention KV state, and charges you a small write premium. On subsequent requests within the TTL (5 minutes for Anthropic's default, longer with the 1-hour beta; OpenAI's automatic cache has its own opaque window), if the request begins with the same exact bytes, you get cache-hit pricing on those tokens.

Concrete pricing as of writing (per million input tokens, USD), using one frontier-tier model as the example:

- Novel input: $3.00
- Cache write (5-min TTL): $3.75 (1.25x premium)
- Cache read: $0.30 (10x cheaper than novel)
- Output: $15.00

The cache-read line is the whole game. A token you read from cache costs one-tenth of a token you send fresh. Output is unchanged — caching does nothing for what the model emits.

## Why agent loops blow this wide open

A single chat turn doesn't benefit much. The benefit compounds when you run the same agent against the same repo dozens of times per hour. Each turn re-sends:

- The system prompt (tool definitions, style rules, persona) — stable across all turns.
- The repo context (file tree, key file contents, glossary) — stable across the session.
- The conversation history up to turn N — stable; only turn N+1 is novel.

If the prefix is byte-identical, every turn after the first reads almost the entire input from cache. The novel portion is the latest user message plus tool results.

Numbers from one of my own loops, a coding-agent session over 4 hours doing routine refactors on a single repo:

```
total turns:                    412
total input tokens:         18.4M
  cache reads:              17.0M  (92.4%)
  cache writes:              0.9M  (4.9%)
  novel:                    0.5M   (2.7%)
total output tokens:         0.71M

cost without caching:       18.4M * $3   + 0.71M * $15  = $65.85
cost with caching:
  cache reads:              17.0M * $0.30 = $5.10
  cache writes:              0.9M * $3.75 = $3.38
  novel:                     0.5M * $3.00 = $1.50
  output:                   0.71M * $15   = $10.65
  total:                                   = $20.63
```

That is a 3.2x cost reduction on a workload that was already running. Same model, same outputs, no quality difference. The only thing I changed was making sure the prefix was stable.

92% cache hit rate is realistic for a stable agent loop. I have seen 96% on tight loops (single-file refactors with a fixed system prompt). I have seen 40% on misconfigured loops where the system prompt was being regenerated with a timestamp on every turn — more on that in a moment.

## The four things that destroy cache hit rate

1. **Timestamps in your system prompt.** A "current time: 2026-04-23T14:32:01Z" line at the top of your system prompt invalidates the cache on every single turn. If you need the model to know the time, put it in the user turn, not the system prefix.

2. **Reordering tool definitions.** Some agent frameworks build the tool list from a hash map and emit them in arbitrary order. The bytes change; the cache misses. Sort tools by name once, at startup, and never re-sort.

3. **Mutating in-context memory.** A "summary so far" block that gets rewritten every turn pushes the cache boundary backward each time. Append-only is the rule. Insert new content at the end of the conversation, never in the middle of the prefix.

4. **Forgetting the cache_control marker.** On Anthropic's API you have to explicitly mark cache breakpoints. Up to 4 of them. If you don't set any, you get nothing — no cache, full novel pricing on every byte. OpenAI is automatic but only kicks in for prefixes over 1024 tokens.

## How to structure a system prompt for max cache hits

The concrete shape that works for me, in pseudo-Python against the Anthropic SDK:

```python
import anthropic

client = anthropic.Anthropic()

# Build the system prompt as a list of blocks.
# Mark cache breakpoints on the LARGE, STABLE blocks.
# Order matters: most-stable first, most-volatile last.
system_blocks = [
    {
        "type": "text",
        "text": IDENTITY_AND_RULES,        # ~2k tokens, never changes
        "cache_control": {"type": "ephemeral"},
    },
    {
        "type": "text",
        "text": TOOL_USAGE_GUIDE,          # ~3k tokens, changes weekly
        "cache_control": {"type": "ephemeral"},
    },
    {
        "type": "text",
        "text": REPO_CONTEXT,              # ~8k tokens, stable per session
        "cache_control": {"type": "ephemeral"},
    },
    # No cache_control on the volatile tail — it changes per task.
    {
        "type": "text",
        "text": f"Current task: {task_description}",
    },
]

# tools: sort by name once, at process start
TOOLS = sorted(load_tools(), key=lambda t: t["name"])

# messages: pure append-only across turns
messages = conversation_history + [{"role": "user", "content": user_input}]

resp = client.messages.create(
    model="claude-3-7-sonnet-latest",
    max_tokens=2048,
    system=system_blocks,
    tools=TOOLS,
    messages=messages,
)

# Inspect the actual hit rate per turn
u = resp.usage
print(
    f"in={u.input_tokens} novel={u.input_tokens} "
    f"cache_read={u.cache_read_input_tokens} "
    f"cache_write={u.cache_creation_input_tokens} "
    f"out={u.output_tokens}"
)
```

Three things to notice. First, the cache_control markers are placed on the *boundaries* of stable blocks, not on every block. You have a budget of 4 breakpoints; spend them on the largest stable prefixes. Second, the volatile task description has no cache marker — paying to write that into cache for one read would be wasteful. Third, every response includes per-turn cache stats; log them and watch for regressions.

## Cache-aware tool design

Tools deserve special attention because the tool *result* sits in the conversation forever. A tool that returns "current timestamp" or "random nonce" or "list of files in arbitrary order" pollutes the prefix and forces a cache write on every subsequent turn for everything after that point.

Rules I now apply to every tool I write for an agent:

- Deterministic output ordering. If you list files, sort them.
- No timestamps in tool output unless the timestamp is the answer.
- Stable serialization. Same JSON keys in same order every call. `json.dumps(obj, sort_keys=True)`.
- Truncate large outputs predictably. If a result is over N bytes, truncate to the same N bytes every time, not "last N lines" which depends on log volume.

## What I would do differently

I ran agent loops for two months without checking cache hit rates because the bills were "fine." When I finally instrumented it, the first session reported 31% hit rate. Two days of fixing the four problems above pushed it to 91%, dropping spend on that workload by roughly 2.7x. The fixes were small. The waste was structural and would have continued forever.

Instrument first. Optimize second. The cache-hit number is the single most important metric for anyone running agent loops, and almost no one is looking at it.

## Links

- Anthropic prompt caching docs: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- OpenAI prompt caching docs: https://platform.openai.com/docs/guides/prompt-caching
- Anthropic 1-hour cache TTL beta announcement: https://www.anthropic.com/news/prompt-caching
