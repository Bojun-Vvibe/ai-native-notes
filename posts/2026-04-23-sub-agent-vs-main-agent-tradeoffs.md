---
title: "Sub-agent vs main-agent: the tradeoffs nobody explains"
date: 2026-04-23
tags: [agents, architecture, sub-agents, multi-agent, design]
est_reading_time: 14 min
---

## TL;DR

A sub-agent is just a model call with a fresh context window, dispatched by the main agent and returning a single result. It is not a smaller model, not a cheaper model, not a different kind of model. The interesting thing about sub-agents is the context isolation: the main agent's expensive history does not pollute the sub-agent's prompt, and the sub-agent's verbose intermediate work does not pollute the main agent's history. That isolation is the entire value proposition. Everything else (parallel dispatch, specialization, retry boundaries) is secondary. This post lays out when sub-agents are the right tool, when they are the wrong tool, and the four design questions I now ask before adding one to a system.

The framing: a sub-agent is a context-scoped procedure call. Treat it the way you would treat a function in a normal program. If you would not extract a piece of code into a function, you should not extract it into a sub-agent.

## Section 1: what a sub-agent actually is, mechanically

A sub-agent, in the systems I run, is implemented as a function the main agent can call as a tool. The function takes a prompt and an optional small context payload, runs its own model call (often a fresh agent loop), and returns a single string result. From the main agent's perspective, the sub-agent's entire internal workings are invisible: it sees a tool call going out and a result coming back, no different from a `read_file` or a `web_search`.

```python
def sub_agent_tool(task: str, context: str = "") -> str:
    """Dispatch a task to a fresh agent. Returns final answer text."""
    sub_messages = [
        {"role": "user", "content": f"{context}\n\nTask: {task}\n\nReturn only the final answer, no prose."},
    ]
    sub_resp = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=4096,
        system=SUB_AGENT_SYSTEM_PROMPT,
        messages=sub_messages,
        tools=SUB_AGENT_TOOLS,
    )
    # if the sub-agent uses tools, run a small loop here
    return _extract_text(sub_resp)
```

Notice what is and is not in this code. Not present: a separate process, a queue, a network call to a different service. The sub-agent runs in the same Python process as the main agent. The "isolation" is purely at the prompt level: the sub-agent's context window does not include any of the main agent's history.

This matters because most discussions of sub-agents conflate three different things: process isolation (different OS processes), model isolation (different model providers or sizes), and context isolation (different prompt windows). Only the third is the defining property. The first two are implementation choices, often unnecessary.

## Section 2: when context isolation is the right call

Three concrete situations where I reach for a sub-agent.

The first: a task whose intermediate work would balloon the main agent's history. The canonical case is "search the web for X and return a summary." The search returns 50 KB of HTML, the agent reads it, extracts the answer, and you only want the answer. Without a sub-agent, the main agent's history now contains the 50 KB. With a sub-agent, the main agent's history grows by the 1 KB summary.

The cost math is direct: if the main agent will run 50 more turns after the search, the 50 KB of HTML will be re-sent (or re-cached) on every one of those turns. At sonnet pricing, 50 KB times 50 turns is significant. The sub-agent isolates the cost to a single call.

The second: a task whose system prompt is fundamentally different from the main agent's. The main agent might be a "code editor" persona with file-editing tools and a system prompt about code style. A sub-task might be "write a one-paragraph product description from this spec," which needs a "marketing copy" persona with a totally different system prompt. Cramming both personas into the main agent's system prompt confuses the model and bloats every call.

The third: a task that has its own tool set. If the sub-task needs a database query tool that the main agent has no business using, exposing that tool only to the sub-agent keeps the main agent's tool list short and the sub-agent's tool list focused. The model's tool selection is sharper when the tool list is shorter.

## Section 3: when context isolation is the wrong call

Three situations where I have learned to keep work in the main agent.

The first: when the main agent needs to reason over the sub-task's output. If the main agent is going to think about the sub-agent's result and possibly modify its plan based on it, the sub-agent's intermediate work might have been useful context. Hiding it costs the main agent the ability to spot subtle issues in the result. Better to do the work inline.

The second: when the sub-task is small. The overhead of a sub-agent dispatch is real: a separate model call, the prompt restating context, the answer being re-summarized. For a task that takes the model 200 tokens of thought to do, the dispatch overhead is roughly equal to just doing the task. Sub-agent only when the task would otherwise consume thousands of tokens of intermediate work.

The third: when the main agent's context already contains exactly what the sub-task needs. If the main agent has been building up state that the sub-task depends on, sending it to a fresh agent means re-sending all that state in the dispatch payload. At that point the "isolation" is just paying twice for the same context. Better to do the work inline.

## Section 4: the four design questions

When considering whether to use a sub-agent for a piece of work, I now ask four questions in order. If the answers do not all line up, I keep the work inline.

**Question 1: would the intermediate work bloat the main context by more than 5x the final answer size?**

If yes, the sub-agent saves money on every subsequent turn. If no, you are paying overhead with no offsetting savings. The 5x threshold is a rule of thumb based on my measurements; finer tuning is workload-dependent.

**Question 2: can the task be specified in 200 words or less?**

A sub-agent is a fresh call. Whatever context the sub-agent needs, you must include in the dispatch prompt. If the task's full specification is 5000 words, the dispatch payload is 5000 words, and you have not saved anything. Sub-agents work for tasks with crisp interfaces.

**Question 3: is the result self-contained?**

The sub-agent returns a single string. If the main agent needs structured output, you have to parse the string. If the main agent needs to ask follow-up questions, the sub-agent is gone and the conversation cannot continue. Use a sub-agent only when the answer is final at the moment of return.

**Question 4: does the failure mode of the sub-agent have a sensible fallback?**

The sub-agent might return a wrong answer, refuse, or time out. The main agent has to handle this. If "the sub-agent failed" leaves the main agent with no plan B, you have introduced a single point of failure for a critical step. Better to do critical work inline where the main agent's full context is available to recover.

If all four answers are yes, dispatch. Otherwise, inline.

## Section 5: the parallel dispatch case, separately

Sub-agents are also the natural unit of parallelism. The main agent can dispatch three sub-agents simultaneously and wait for all three before continuing. This is genuinely useful for tasks like "search three sources and combine the results" or "review this diff from three angles."

```python
import concurrent.futures

def parallel_dispatch(tasks: list[dict]) -> list[str]:
    with concurrent.futures.ThreadPoolExecutor(max_workers=len(tasks)) as ex:
        futures = [ex.submit(sub_agent_tool, t["task"], t.get("context", "")) for t in tasks]
        return [f.result() for f in futures]

results = parallel_dispatch([
    {"task": "Find recent papers on prompt caching"},
    {"task": "Find recent papers on KV cache memory management"},
    {"task": "Find recent papers on speculative decoding"},
])
```

The wall-clock latency of this is roughly the latency of the slowest sub-agent, not the sum. For a research-style step with three independent strands, the speedup is significant.

But: the cost is three sub-agent calls' worth, not one. Parallelism saves time, not money. If your bottleneck is dollars, parallel sub-agents make things worse. If your bottleneck is latency, they help. Know which one you are optimizing for before reaching for parallel dispatch.

## Section 6: the model-size question

A common temptation is to make the sub-agent a smaller, cheaper model than the main agent. "The sub-agent only does a focused task, so a small model is fine."

This sometimes works. Often it does not. Two failure modes:

The first: the sub-agent's task is focused but hard. Summarizing a 50 KB document into one paragraph sounds easy but is actually a high-skill task that small models can do clumsily. The main agent then receives a clumsy summary and makes worse decisions. The "savings" from the smaller model are eaten by the worse downstream behavior.

The second: the sub-agent's task is easy but requires tool use. Small models are noticeably worse at picking from a list of tools and constructing valid arguments. A small-model sub-agent that needs to use three tools will fail more often, requiring escalation or retry, costing more in the end than just using the larger model.

My current default: sub-agents use the same model as the main agent unless I have specifically measured that a smaller model handles the task. The cost discipline is in not making the call (the context isolation savings) rather than in making it cheaper.

## Section 7: the retry boundary, accidentally a feature

A side benefit of sub-agents that I noticed only after a year of using them: they are a natural retry boundary. A sub-agent call is a single function invocation; if it fails, the main agent can retry it without the failure polluting the main context.

```python
def sub_agent_with_retry(task: str, context: str = "", max_attempts: int = 3) -> str:
    last_err = None
    for attempt in range(max_attempts):
        try:
            result = sub_agent_tool(task, context)
            if _looks_valid(result):
                return result
            last_err = "invalid_result"
        except Exception as e:
            last_err = str(e)
    return f"[sub-agent failed after {max_attempts} attempts: {last_err}]"
```

The main agent sees one tool call and one result. The retries are invisible. Compare to inline work: if the main agent tries something three times in a row and fails, the failures are in its history, possibly biasing its next move. The sub-agent's hidden retries do not have this bias.

This is the same argument as for any abstraction boundary. The boundary lets you change implementation details without disturbing the caller. With sub-agents, "implementation details" includes "how many times we tried."

## Section 8: the anti-patterns I have unlearned

A short list of sub-agent uses I tried, found wanting, and stopped doing.

**Sub-agent as a substitute for prompt engineering.** "The main agent keeps making this mistake; let me dispatch the work to a sub-agent with a more specific prompt." Usually the right answer is to fix the main agent's prompt, not to add a layer of indirection. Sub-agents are not free; if you reach for one because the main agent's prompt is unclear, fix the prompt first.

**Sub-agent as a unit of organization.** "I want this task to feel like a separate concern." Sub-agents are not Python modules. The fact that something is conceptually a separate concern does not mean it should be dispatched. The only reason to dispatch is one of the three conditions in section 2. "Cleaner architecture" is not one of them.

**Sub-agent as a way to use a different provider for one step.** "I want this step to go to a different provider, so I will dispatch to a sub-agent against that provider." This works mechanically but rarely pays off. The dispatch overhead plus the cross-provider context translation usually exceed any per-call savings, and the failure modes (different provider's outage) are now inside the agent loop. Better to make the provider choice at the routing layer for the whole session.

**Recursive sub-agents.** A sub-agent dispatching a sub-sub-agent. Theoretically composable, in practice a debugging nightmare. Two levels deep, the trace is unintelligible. I keep dispatch flat: main agent dispatches at most one level of sub-agents, never more.

## Section 9: what the main agent's prompt should say about sub-agents

A subtle point: the main agent needs to know when to use the sub-agent tool. The system prompt has to explain the trade-off in a way the model can act on.

The version that works for me, more or less verbatim:

> You have a `dispatch_subtask` tool. Use it when a task would require reading or producing more than 2000 tokens of intermediate text but the final answer is a short summary. Do not use it for tasks that need follow-up questions or that share context with the rest of your work. The sub-agent has no memory of this conversation; you must include all needed context in the dispatch payload.

The model follows this most of the time. When it gets it wrong (dispatching trivially small tasks, or trying to dispatch tasks that need follow-up), the cost is small enough that I have not built corrective tooling. But if the misuse rate were higher, I would consider a wrapper that rejected dispatches that did not meet a token-size threshold.

## Section 10: what the data shows

After two years of running sub-agents in production, the workload-level numbers:

Sub-agent calls account for about 12 percent of total model calls but only 6 percent of total input tokens (because their inputs are smaller, since the main agent's history is excluded). They account for about 9 percent of total cost.

The estimated savings vs an inline-everything baseline: roughly 35 percent of input cost, primarily from not re-sending sub-agent intermediate work on subsequent main-agent turns. The estimate is approximate because I cannot run the same workload twice with identical inputs to get a clean A/B; the number comes from forensic analysis of which main-agent turns would have included sub-agent intermediate work without isolation.

The interesting non-cost outcome: the main agent's average context length is about 40 percent shorter than it would be without sub-agents. This is the real win. Shorter contexts are easier for the model to attend to coherently, so the main agent makes fewer mistakes, finishes tasks in fewer turns, and produces more readable transcripts for human review.

The sub-agent pattern, used with discipline, makes the main agent better at being a main agent. That is the value, more than the dollar savings.

## References

- Anthropic on multi-agent patterns: https://www.anthropic.com/research/building-effective-agents
- "Tree of Thoughts" paper (related dispatch ideas): https://arxiv.org/abs/2305.10601
- ReAct paper (single-agent baseline): https://arxiv.org/abs/2210.03629
- Concurrent futures docs: https://docs.python.org/3/library/concurrent.futures.html
