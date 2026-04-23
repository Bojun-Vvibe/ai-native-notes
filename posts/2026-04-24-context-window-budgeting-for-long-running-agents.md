---
title: "Context window budgeting for long-running agents"
date: 2026-04-24
tags: [agents, context, tokens, economics]
est_reading_time: 9 min
---

## The problem

A long-running agent is not the same animal as a chat assistant. Chat is bursty: the user types, the model responds, the window resets when the conversation ends. A long-running agent — a daemon that ticks every fifteen minutes, a code-review bot that sweeps a thousand PRs overnight, a research agent that crawls and summarizes for hours — does not get to reset. Every tick adds to its accumulated context, or it doesn't, and that "or it doesn't" is the most consequential design decision in the system.

The failure mode is not "the agent runs out of context and crashes." Modern providers will quietly truncate, or they'll error with a clear message, and you'll handle it. The actual failure mode is subtler: the agent runs at full context for hours, paying full per-token rates on every call, accumulating irrelevant scrollback, and producing worse outputs the longer it runs. By hour six it costs ten times what it cost at hour zero and is measurably dumber. You don't notice because the outputs still look like outputs.

## The setup

A typical long-running agent loop looks like this:

```
loop:
  observe()        # read state, list issues, poll a queue
  think()          # call the model with current context
  act()            # tool calls, edits, posts
  record()         # write to history
  sleep(interval)
```

The question this post is about is: between iterations of that loop, what does "context" mean? Three answers, in order of how often I see them in the wild:

1. **Everything.** Append every tool call, every observation, every model response into a single growing array. Send the whole thing every tick. Easy to write. Catastrophic by hour two.
2. **Last N turns.** Keep a sliding window of the last N tool calls or N tokens. Easy to write, mostly works, silently drops information that mattered.
3. **Curated working memory.** A small, hand-shaped context that the agent rebuilds from durable storage at the top of each tick.

Option 3 is what every production long-running agent ends up at. The interesting question is how to get there without overengineering the first version into a database project.

## What I tried

- **Attempt 1: just append everything.** Wrote a daemon that called the model every fifteen minutes and kept the full conversation array in a JSON file on disk. Loaded it, sent it, appended the response, saved it. By tick 40 the file was 380 KB and a single tick cost six cents. By tick 200 a single tick cost forty-one cents and the agent was confidently citing facts from tick 12 that were no longer true. Killed it.

- **Attempt 2: last 20 turns.** Trimmed to the last 20 messages before each call. Cost dropped immediately. Quality dropped less obviously: the agent forgot which repos it had already processed and started doing duplicate work. The duplicate work was hard to spot because the outputs were superficially correct — different commit messages, different prose — but pointing at the same underlying changes.

- **Attempt 3: last 20 turns + a "facts" file.** Added a tiny `facts.md` that the agent updated each tick with one or two lines: "processed repo X at tick N, next due at tick N+96." Loaded `facts.md` at the top of every tick alongside the trimmed window. Worked. Cost stayed flat. The agent stopped duplicating work. But the facts file got noisy fast and started contradicting itself.

- **Attempt 4: structured working memory in JSONL.** Replaced `facts.md` with `state/last-tick.json` (latest snapshot only) and `state/history.jsonl` (one line per tick, append-only, never read in full by the agent — only the tail). The agent reads `last-tick.json` and the last 20 lines of history at the top of each tick. That is the entire context, plus whatever the current task itself produces. This is what's running now.

## What worked

The structure that survives contact with reality is layered:

```
.daemon/state/
  last-tick.json       # full snapshot of most recent tick (overwrite)
  history.jsonl        # one-line summary per tick (append only)
  facts/               # durable facts the agent must not forget
    repos.json
    quotas.json
```

At the top of each tick the agent loads:

1. `last-tick.json` — what did I just do?
2. The last ~20 lines of `history.jsonl` — what's the recent trend?
3. The relevant fact files for this tick's task — what must I not violate?

That's it. Nothing else from prior ticks enters the model context. The current tick's tool calls and observations live and die inside this tick.

A minimal loader looks like this:

```python
from pathlib import Path
import json

STATE = Path.home() / "Projects/Bojun-Vvibe/.daemon/state"

def load_context() -> dict:
    last = {}
    if (STATE / "last-tick.json").exists():
        last = json.loads((STATE / "last-tick.json").read_text())

    history = []
    hist_path = STATE / "history.jsonl"
    if hist_path.exists():
        # tail the last 20 lines without reading the whole file
        with hist_path.open("rb") as f:
            f.seek(0, 2)
            size = f.tell()
            f.seek(max(0, size - 16384))
            tail = f.read().decode("utf-8", errors="replace").splitlines()
        history = [json.loads(l) for l in tail[-20:] if l.strip()]

    return {"last_tick": last, "recent_history": history}
```

The total context sent to the model at the top of a tick stabilizes at roughly 2–4 KB regardless of how long the agent has been running. The agent's memory of what it's done lives on disk, in formats it itself wrote, in shapes it itself can re-read.

## Why it worked (or: my current best guess)

Three reasons, in increasing order of how confident I am:

**The cheap reason.** Token cost is roughly linear in context size, and the context size for a long-running agent is dominated by the prefix, not the per-tick work. Capping the prefix caps the cost. This is just arithmetic and it's not interesting.

**The medium-confidence reason.** Models are worse at retrieving from a long context than from a short one. Every paper on "lost in the middle" effects says the same thing: information buried in a 100K-token prefix is recalled less reliably than information in a 4K-token prefix. So a smaller, curated context is not just cheaper, it's more accurate. A daemon that consults a 3 KB summary outperforms a daemon that has 300 KB of scrollback to "draw on."

**The high-confidence reason, but the one I'm least sure I understand correctly.** Long contexts encode the agent's past mistakes as if they were facts. When tick 12 made a wrong call and tick 40 sees that wrong call in scrollback, tick 40 treats it as evidence. By tick 200 the agent is hallucinating a self-consistent worldview built on its own earlier hallucinations. Trimming context breaks this loop. A fresh context per tick, hydrated only from durable summaries the agent had to decide were worth writing down, forces the agent to relitigate its assumptions instead of inheriting them.

I'm not certain that third reason is the dominant effect. It might be ninety percent the second reason and only ten percent the third. But the behavior I observed — agents getting confidently wrong over time, in ways that traced back to specific earlier wrong calls — is consistent with it, and the fix (durable state, ephemeral context) makes the symptom go away.

## A note on what to put in the durable state

The hardest part of this design is not the file format. It's deciding what's worth promoting from "happened this tick" to "the next tick needs to know." The discipline I've landed on is to ask, at the end of each tick, three questions:

1. **What did I commit to?** If this tick scheduled, queued, or promised something — "process repo X next" — that promise has to survive into the next tick or it never happened.
2. **What did I rule out?** Negative results are content. If the agent tried approach A and it failed, the next tick should not retry approach A without new information. A `tried_and_failed.jsonl` is just as important as a `succeeded.jsonl`.
3. **What's the current numeric state of the world?** Counters, quotas, last-seen timestamps, current rate-limit windows. These are the things the agent will silently get wrong if it has to re-derive them from scrollback.

Anything else — the model's reasoning, intermediate observations, the specific phrasing of a tool response — is ephemeral. It dies with the tick. If you find yourself wanting to preserve it, that's a signal that you haven't yet identified the durable fact it implies; preserve the fact, not the trace.

The honest version of this rule is that I get it wrong about a third of the time. I'll lose a fact, notice three days later that the agent is doing something dumb, and add a new field to the state file. The state schema grows over weeks, not days. JSONL is forgiving about this — new fields just appear in new lines, old lines stay valid — which is one more reason it beats a strict schema for this job.

## What I would do differently

Start with attempt 4 next time. The intermediate attempts were not learning, they were procrastination disguised as iteration. The shape of the answer — durable state on disk, ephemeral context per tick, a tiny loader function — is obvious in retrospect, and the cost of jumping straight to it is one afternoon of design work versus three weeks of mounting bills.

The other thing I'd do differently: track cost-per-tick from tick zero, not from the moment cost becomes a problem. The graph that would have told me to stop attempt 1 was a graph I wasn't drawing. A daemon without a cost graph is a daemon that will surprise you, and the surprise is never pleasant.

## Links

- Anthropic, "Long context prompting for Claude 2.1": https://www.anthropic.com/news/claude-2-1-prompting
- Liu et al., "Lost in the Middle: How Language Models Use Long Contexts": https://arxiv.org/abs/2307.03172
- OpenAI cookbook, "How to handle long inputs": https://cookbook.openai.com/examples/how_to_handle_rate_limits
