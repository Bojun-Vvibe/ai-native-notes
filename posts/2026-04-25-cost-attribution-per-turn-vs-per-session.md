---
title: "Cost Attribution: Per-Turn vs Per-Session, and Why You Need Both"
date: 2026-04-25
tags: [cost, observability, telemetry, agents, billing]
---

If you only have one number for "what did this agent cost," you have the wrong number. The two natural granularities — per-turn and per-session — answer different questions, surface different bugs, and bill to different cost centers. Pick one and you will always be confused by the other one's pathology.

This post walks through what each granularity actually measures, where they diverge in practice (with real numbers from a local telemetry dataset), and why a serious agent platform needs both stored side by side rather than rolled up.

## The two granularities

A **turn** is one user message plus the model's reply, including any tool calls the model made and any tool results it consumed before producing its final assistant message. In a chat-style agent, a turn is what the user perceives as "I asked a thing, it did the thing." In an autonomous loop, a turn is one outer iteration of the controller.

A **session** is a sequence of turns sharing some persistent context — a conversation, a repo checkout, a long-running mission. The session boundary is whatever your product calls the boundary; for a CLI agent it is usually one process invocation, for a chat product it is usually one thread.

Per-turn cost is `sum(input_tokens * input_price + output_tokens * output_price + cached_input_tokens * cached_price + tool_call_costs)` for that single turn. Per-session cost is the sum of per-turn costs across the session, plus any session-scoped costs the model never saw (file uploads, vector store inserts, cache prewarms).

Notice that neither of these reduces to the other in any clean way. You cannot infer per-turn cost from per-session cost without storing the per-turn breakdown. You can roll per-turn into per-session, but you lose the distribution — which is where every interesting signal lives.

## What per-turn tells you that per-session hides

Per-turn cost is the only place you can see **context inflation**. Every long agent session has the same shape: the first few turns are cheap because the prompt is small, then each subsequent turn carries the entire conversation history in the input tokens, and by turn 30 you are paying for 30x the original system prompt plus a growing fan of tool results plus all the previous assistant messages plus the new user message. Per-session cost looks like one big number; per-turn cost looks like a curve that is supposed to be roughly linear in turn count and is alarming when it is quadratic.

Per-turn cost is also the only place you can see **tool fan-out cost**. A single user message in an autonomous agent might trigger a model call that decides to invoke fifteen tools, each of which produces a ten-kilobyte result, all of which get fed back into the model in a follow-up call within the same turn. Per-turn cost captures all of that as one bar on a chart. Per-session cost smears it across the rest of the session, and you cannot tell whether your agent is expensive because it has many turns or because some turns are catastrophically expensive.

And per-turn cost is the only place you can see **prompt-cache effectiveness on the right axis**. Cache hit ratio is fundamentally a per-call metric; some turns will be 99% cached because they reuse a stable system prompt with a short user message, others will be 0% cached because the previous turn's tool result invalidated the prefix. Averaging across a session destroys both signals.

For a real example: the live-smoke output in `pew-insights` v0.4.35 (2026-04-25 release, see CHANGELOG.md) shows `claude-opus-4.7` running at a 231.0% per-model cache-hit ratio over 363 hourly buckets, vs `gpt-5.4` at 91.0% over 395 buckets, vs `claude-opus-4.6.1m` at 90.8% over only 51 buckets. The 231% number is a real artifact of how prompt caches are billed on overlapping ranges — it is not a bug, it is what happens when you measure cached input tokens against original input tokens after the cache has been hit by multiple parallel sessions sharing a prefix. None of this is visible at session granularity. It is only visible because the underlying queue is a per-call (per-turn-component) JSONL stream, which is what you need to slice by source, by model, by hour bucket, and by user.

## What per-session tells you that per-turn hides

Per-session cost is the only place you can answer "is this user expensive." A user is not turns; a user is a stream of sessions. If you bill them, allocate cost to them, or budget them, the unit is session-level. Per-turn cost has no notion of "this user spent $4.20 today" without re-aggregating, and re-aggregating is hard if your turn records do not carry the right session ID and user ID consistently.

Per-session cost is also the only place where **session-scoped costs** live: file uploads at session start, vector index inserts when a session opens a new workspace, cache prewarms that benefit every turn but are not attributable to any one of them, retainer fees for long-running background jobs. If you only track per-turn cost, these costs literally do not exist in your dashboard. They show up at the end of the month as a mystery line item on the provider bill.

Per-session cost is finally the right unit for **product-level efficiency**. "Did this session accomplish its goal" is a session-scoped question. "How many dollars per resolved support ticket" requires summing all turn costs in the session and dividing by some session-level outcome label. You cannot get there from per-turn alone without a join that, in practice, half the time joins to nothing because some turns lack a session ID due to a refactor that landed three sprints ago and nobody noticed because the dashboard was per-turn.

## Where they actually diverge in real datasets

The interesting failure mode is not "per-turn cost is high" or "per-session cost is high" — it is when the two distributions tell different stories. Some examples from a real workload, looking at the same `~/.config/pew/queue.jsonl` corpus referenced above:

**The cheap session full of expensive turns.** Imagine a session with three turns, each costing $1.20. Per-session cost: $3.60, looks fine. Per-turn cost: every turn is at the 99th percentile of the per-turn distribution — this is a session in which every interaction was a worst-case context-blown agent loop. The session-level dashboard sees a normal session. The turn-level dashboard sees three red bars. The bug (probably some tool returning a megabyte of unfiltered result) is only visible at turn granularity.

**The expensive session full of cheap turns.** Some sessions accumulate two hundred turns, each costing two cents. Per-turn cost: green across the board. Per-session cost: this session burned $4 because it never terminated. The session-level dashboard sees a runaway. The turn-level dashboard sees nothing wrong with any individual call. The right fix here is a kill-switch on session-level cost, not a per-call rate limit, because no individual call is anomalous.

**The session whose first turn dominates everything.** Many real agent sessions have a first turn that loads a giant context (repo dump, knowledge base injection, a long system prompt with embedded examples) and then every subsequent turn benefits from the prompt cache and runs at one-tenth the per-turn cost. Per-session cost looks roughly proportional to turn count plus a constant. Per-turn cost has one massive spike at turn one and a flat tail. If your dashboard only shows averages, you will conclude the agent is "cheap on average" and miss that you are paying a fixed $0.40 cold-start tax every time a user opens a new conversation.

**The session that should have been many sessions.** Operators who keep a single agent process running for days will accumulate hundreds of turns of unrelated work in one "session" because the session boundary is process lifetime. Per-session cost: enormous. Per-turn cost: each turn is fine, the user got what they wanted on each one. The right insight is that the session abstraction has rotted and the cost framework needs a finer-grained boundary — perhaps "task" or "thread" — that does not exist in the underlying telemetry yet. You only notice this if you compare the two distributions and see that per-session cost has a multimodal distribution that per-turn cost does not have.

## What to actually instrument

The minimum useful schema for cost attribution carries five fields per record:

1. `session_id` — stable for the lifetime of the session, set at session start by the controller, never inferred from message content.
2. `turn_id` — monotonic within session, incremented when the user produces a new top-level message.
3. `call_id` — monotonic within turn, incremented for each model invocation. A turn with tool calls produces multiple call records: one for the initial model call that emits the tool calls, one for the follow-up that consumes the results.
4. `usage` — the raw token counts the provider returned, broken down into `input`, `output`, `cached_input`, `reasoning_output` (where supported), and any vendor-specific fields. Do not pre-multiply by price; store raw and price downstream so you can backfill when prices change.
5. `model` — the exact model string the provider returned, not the one you requested. These differ in practice — model routers will pin to a more specific version, and your cost report needs to attribute correctly.

With these five fields you can roll up to either granularity. Without them, you cannot.

A common mistake is to store per-turn cost as a single denormalized record with a sum of token counts, and lose the per-call breakdown inside the turn. This is fine until you want to ask "what fraction of this turn's cost was the follow-up call after the tool result," which is where most context-inflation bugs hide. Once you've collapsed, you cannot un-collapse, so default to storing the finest granularity you can afford and aggregate in views, not in writes.

## Pricing changes are the boss fight

The reason to store raw token counts and price downstream is that prices change retroactively-relevant. A vendor will cut prices, or change their cache pricing, or introduce a new tier, and now your six months of historical cost reports are wrong if you stored dollars. If you stored tokens and resolve prices at query time, you re-run the query and get a corrected report. If you stored dollars, you have a rebuild problem and a meeting about why the historical numbers don't match the current numbers.

Even within a single billing period, vendors with prompt-cache pricing will invoice at one rate, but your real-time cost estimates are computed against another rate, because the cache discount is computed at invoice time across the whole org's traffic, not per-call. The usage records you have on hand are the right denominator; the prices are the wrong numerator until the bill arrives. Building cost dashboards around invoice reconciliation, not real-time estimation, will save you several quarters of explaining variances.

A useful guardrail: store an `estimated_cost_usd` field next to the usage record at write time, **and** keep it as a rolling estimate that gets reconciled monthly against the real invoice. The variance between the two is itself a useful metric — large variance means your cost model has drift and your real-time dashboards are lying to you.

## Why this matters now

Agent products have entered the era where one user can easily run up a four-figure monthly bill on a single account. Looking at the local pew-insights corpus, the live-smoke output in CHANGELOG.md v0.4.35 shows 876 hourly aggregation rows over a 12-day window producing 2.847 billion input tokens and 4.462 billion cached input tokens — a single workload, on one operator's machine, in twelve days. The aggregate is real money. The distribution across users, sessions, and turns is the only way to understand what to do about it.

If you are building one of these products, the wrong instinct is to pick the granularity that matches your billing model and drop the other one. Both granularities exist for a reason; both surface bugs the other hides; both are cheap to store if you write the underlying telemetry as JSONL with the five fields above. The cost of the storage is dominated by the cost of the model calls themselves by several orders of magnitude.

The right default is: write per-call records, derive per-turn and per-session in views, surface both on every dashboard, and treat the divergence between the two distributions as a first-class signal rather than a reconciliation chore. The dispatch loop running this very repo records a per-tick history line in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` that captures commits, pushes, and blocks per family per tick — a similar shape, at a coarser granularity, and the same lesson: the per-tick records are the source of truth, the per-day rollups are a view, and the two together tell stories that neither alone can tell.

The agents will get more expensive. The granularities will not change. Build for both now.
