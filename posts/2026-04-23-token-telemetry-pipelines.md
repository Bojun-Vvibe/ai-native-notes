---
title: "Token telemetry pipelines: knowing what you spent before the bill arrives"
date: 2026-04-23
tags: [telemetry, observability, tokens, cost, agents]
est_reading_time: 14 min
---

## TL;DR

If you run agents in any kind of production setting, the most expensive metric you are not measuring is "tokens per outcome." Provider dashboards show you tokens per minute and dollars per day, neither of which lets you answer "did this agent run pay for itself?" or "which prompt change blew up our bill last Tuesday?" This post walks through the telemetry pipeline I have settled on after a few painful surprise bills: a thin client-side emitter, a stateless collector that does enrichment, a columnar warehouse for analysis, and a small set of derived metrics that have actually changed decisions. The goal is not a vendor-quality observability product; it is the minimum pipeline that turns "the bill was bigger than expected" into "the bill was bigger because of these three sessions, here are their transcripts."

## The problem

For about two months I ran agents on instinct. I knew roughly which models I was using, I had the provider dashboard bookmarked, and I checked it every few days. Then one Monday morning the previous week's spend was 4x the prior week's. The dashboard told me which model and which API key, which narrowed it to "everything" and "the one I use." There was no way to tie a token count back to a session, a user, an agent profile, or — critically — a single change I had made to a system prompt the previous Wednesday. I spent the better part of a day reconstructing the spike from logs that had not been designed for this purpose. Once I built the pipeline below, the same investigation would have taken about four minutes.

## What "token telemetry" actually has to do

Before designing anything, I wrote down the questions I needed to answer:

1. How many tokens did session `S` cost, broken down by input / cached / output / tool-result tokens?
2. Which prompt or tool change correlates with a step change in cost-per-session?
3. Which actor (user, agent profile, automation) is the most expensive, and why?
4. Are we paying for tokens we should be hitting from cache?
5. Did this specific agent run produce enough value to justify its cost?

Every architectural choice below traces to one of these five questions. If a metric does not help answer one of them, it is noise.

## The setup

- Agents: a mix of open-source coding agents and a small in-house orchestrator, all using OpenAI-compatible or Anthropic-native APIs, often via a gateway.
- Wire layer: a local proxy (one of the OSS gateways) that already emits per-request logs.
- Storage: DuckDB for local analysis, ClickHouse for the shared deployment. Either works; the schema is the same.
- Glue: small Python and shell scripts. No vendor observability product.

## Stage 1: the emitter

The first decision is **where the token counts come from**. There are three options, in increasing order of trustworthiness:

1. The model client library's reported usage (what the SDK tells you).
2. The provider's response `usage` block (what the API actually returned).
3. The provider's billing export (what you are actually charged).

You want all three eventually, but you want #2 first because it is per-request, available immediately, and matches billing closely enough for engineering decisions. Tokenizer-based estimates ("count tokens in the prompt locally") are only useful for pre-flight budgeting; they drift from billed tokens as soon as you involve cached prefixes, tool results, or vision blocks.

The emitter lives in the gateway, not in each agent. Putting it in the gateway means every caller — coding agent, batch script, cron job — gets instrumented for free, and you only have one place to update when a provider adds a new field (e.g., cached input tokens, or reasoning tokens for thinking models).

A minimal emitter as a gateway middleware:

```python
# gateway middleware (FastAPI-ish pseudocode)
import json, time, uuid, httpx

async def token_emitter(request, call_next):
    req_id = request.headers.get("x-request-id") or str(uuid.uuid4())
    actor  = request.headers.get("x-actor-sub", "unknown")
    session = request.headers.get("x-session-id", "unknown")
    profile = request.headers.get("x-agent-profile", "unknown")

    t0 = time.time()
    response = await call_next(request)
    body = b"".join([chunk async for chunk in response.body_iterator])
    payload = json.loads(body)

    usage = payload.get("usage", {}) or {}
    event = {
        "ts": time.time(),
        "request_id": req_id,
        "actor": actor,
        "session": session,
        "agent_profile": profile,
        "provider": request.headers.get("x-provider", "unknown"),
        "model": payload.get("model"),
        "input_tokens":          usage.get("prompt_tokens", 0),
        "cached_input_tokens":   usage.get("prompt_tokens_details", {}).get("cached_tokens", 0),
        "output_tokens":         usage.get("completion_tokens", 0),
        "reasoning_tokens":      usage.get("completion_tokens_details", {}).get("reasoning_tokens", 0),
        "duration_ms": int((time.time() - t0) * 1000),
        "status": response.status_code,
    }
    await emit(event)  # write to local jsonl + ship to collector
    return Response(content=body, status_code=response.status_code,
                    headers=dict(response.headers))
```

Two non-obvious choices in that snippet. First, the emitter logs to a **local JSONL file first**, then ships to the collector asynchronously. If the collector is down, you don't lose data and you don't slow down agent calls. Second, identity (`actor`, `session`, `agent_profile`) is read from headers the agent sets, not inferred from the request body. The gateway should not know about agent semantics; it should just propagate whatever the caller passed in.

## Stage 2: the collector and enrichment

The collector is stateless and does three things: dedupe, enrich, and forward. It does not store anything long-term.

**Dedupe** is necessary because retries happen and because the local-first emitter occasionally sends the same event twice after a restart. A 5-minute LRU on `request_id` is sufficient.

**Enrichment** is where you add the things the emitter could not know:

- Dollar cost, computed from a price table keyed on `(provider, model, token_kind)`. Storing dollars at write time is much cheaper than computing them at every query and means historical rows are still correct after a price change.
- Cache-hit ratio per request: `cached_input_tokens / input_tokens`.
- A `prompt_fingerprint`, computed from a hash of the system prompt slice the agent reports it sent. This is the single most useful enrichment for question (2) above.

```python
PRICE = {
  # USD per 1K tokens. Adjust to your contract.
  ("anthropic", "claude-opus", "input"):  0.015,
  ("anthropic", "claude-opus", "cached"): 0.0015,
  ("anthropic", "claude-opus", "output"): 0.075,
  ("openai",    "gpt-4.1",     "input"):  0.0025,
  ("openai",    "gpt-4.1",     "cached"): 0.000625,
  ("openai",    "gpt-4.1",     "output"): 0.010,
}

def enrich(ev: dict) -> dict:
    p = ev["provider"]; m = ev["model"]
    def $(kind, n): return (PRICE.get((p, m, kind), 0.0) * n) / 1000
    ev["cost_usd"] = round(
        $("input",  ev["input_tokens"] - ev["cached_input_tokens"])
      + $("cached", ev["cached_input_tokens"])
      + $("output", ev["output_tokens"]),
        6,
    )
    denom = max(ev["input_tokens"], 1)
    ev["cache_hit_ratio"] = round(ev["cached_input_tokens"] / denom, 3)
    return ev
```

The price table belongs in the collector, not in the agent. When prices change, you update one file and redeploy one service. Agents do not need to know what anything costs.

**Forwarding** writes to two sinks: a hot one (the warehouse, for analysis) and a cold one (object storage, as raw JSONL by date, for replay if you ever need to recompute). The cold sink has saved me twice when I changed a price wrong.

## Stage 3: the warehouse schema

Keep the schema flat and wide. Agents emit hundreds of events per session; you will join them constantly.

```sql
CREATE TABLE token_events (
  ts                   TIMESTAMP,
  request_id           VARCHAR PRIMARY KEY,
  actor                VARCHAR,
  session              VARCHAR,
  agent_profile        VARCHAR,
  provider             VARCHAR,
  model                VARCHAR,
  prompt_fingerprint   VARCHAR,
  input_tokens         INTEGER,
  cached_input_tokens  INTEGER,
  output_tokens        INTEGER,
  reasoning_tokens     INTEGER,
  duration_ms          INTEGER,
  status               INTEGER,
  cost_usd             DOUBLE,
  cache_hit_ratio      DOUBLE
);

CREATE INDEX idx_session ON token_events (session);
CREATE INDEX idx_ts      ON token_events (ts);
CREATE INDEX idx_actor   ON token_events (actor);
```

Two notes:

- No nested JSON. Every field you might filter or aggregate on is a column. If you discover a new field you care about, add a column; do not start querying into a JSON blob, because you will never stop.
- `prompt_fingerprint` is the foreign key to a smaller `prompts` table that stores the actual prompt text once. This keeps the events table cheap to scan.

## Stage 4: the queries that change behavior

A telemetry pipeline is only worth what its queries change. Five queries earn their keep:

**Q1 — cost per session, last 24 hours, top 20**:

```sql
SELECT
  session,
  any_value(actor)         AS actor,
  any_value(agent_profile) AS profile,
  count(*)                 AS turns,
  sum(input_tokens)        AS in_tok,
  sum(cached_input_tokens) AS cached_tok,
  sum(output_tokens)       AS out_tok,
  round(sum(cost_usd), 4)  AS cost_usd
FROM token_events
WHERE ts > now() - INTERVAL '1 day'
GROUP BY session
ORDER BY cost_usd DESC
LIMIT 20;
```

This is the first query to run when a bill looks wrong. Almost always, three sessions are responsible for most of the spike.

**Q2 — prompt-fingerprint regression detector**:

```sql
WITH per_fp AS (
  SELECT
    prompt_fingerprint,
    date_trunc('day', ts) AS day,
    avg(input_tokens) AS avg_in,
    count(*) AS n
  FROM token_events
  WHERE ts > now() - INTERVAL '14 days'
  GROUP BY 1, 2
)
SELECT
  prompt_fingerprint,
  max(avg_in) FILTER (WHERE day = current_date)        AS today_avg_in,
  max(avg_in) FILTER (WHERE day < current_date)        AS prior_avg_in,
  round(max(avg_in) FILTER (WHERE day = current_date)
      / nullif(max(avg_in) FILTER (WHERE day < current_date), 0), 2) AS ratio
FROM per_fp
GROUP BY 1
HAVING ratio > 1.3 AND today_avg_in > 5000
ORDER BY ratio DESC;
```

This catches "you added another paragraph to the system prompt and now every call is 30% bigger." Run it daily; alert if any fingerprint shows up.

**Q3 — cache miss leaderboard**:

```sql
SELECT
  agent_profile,
  model,
  round(avg(cache_hit_ratio), 3) AS avg_cache_hit,
  sum(input_tokens - cached_input_tokens) AS uncached_in_tok,
  round(sum(cost_usd), 2) AS cost_usd
FROM token_events
WHERE ts > now() - INTERVAL '7 days'
  AND input_tokens > 1000
GROUP BY 1, 2
ORDER BY uncached_in_tok DESC
LIMIT 20;
```

If an agent profile has a low cache-hit ratio on a model that supports prompt caching, there is almost always a fixable reason: the system prompt has a varying header, the cache breakpoint is in the wrong place, or messages above the breakpoint are being mutated.

**Q4 — output:input ratio outliers**:

A normal coding agent turn produces output that is a small fraction of input. When that ratio inverts, the model is usually generating a long file or stuck in a verbose loop:

```sql
SELECT session, request_id, input_tokens, output_tokens,
       round(output_tokens::DOUBLE / nullif(input_tokens, 0), 2) AS out_in
FROM token_events
WHERE ts > now() - INTERVAL '1 day'
  AND output_tokens > 4000
ORDER BY out_in DESC
LIMIT 20;
```

**Q5 — cost per outcome**:

This is the only one that requires extra plumbing. The agent has to emit an "outcome" event at session end (succeeded / abandoned / human-takeover), with the same `session` ID. Joined against `token_events`, you get cost-per-success, which is the metric that should drive every optimization decision.

## Sampling, retention, and PII

A few practical points the spec-style write-ups skip:

- **Sample inputs, never sample usage.** Token counts are tiny rows; keep them all forever. Full prompt text is large and sensitive; sample at, say, 1-in-50 and store with strict retention.
- **Do not store tool *arguments* in the events table.** Some tool calls contain customer data. Store them in a separate, access-controlled table keyed by `request_id`.
- **Default retention: 90 days for raw events, 2 years for daily aggregates.** Anything longer than 90 days is for trend analysis, which only needs daily rollups.

## What changed once I had this

Three concrete decisions came out of the first month of having the pipeline:

1. I moved a 4KB block of "examples" from the system prompt into a tool that the agent calls only when needed. Q3 had told me the prompt was the cause of a chronically low cache-hit rate; the cache-hit rate jumped from 0.3 to 0.85 and weekly cost dropped ~30%.
2. I capped output tokens on a specific agent profile after Q4 surfaced one session that generated a 12,000-token "summary." The cap prevented a recurrence and cost nothing in quality (the summary was useless anyway).
3. I killed an experimental agent profile entirely after Q5 showed its cost-per-success was 8x the next worst profile, with no measurable quality benefit.

None of these decisions were possible from the provider dashboard.

## What I would do differently

I built the cost enrichment last and regretted it for two weeks, because every ad-hoc query needed the price table joined in by hand. Build cost enrichment into the collector on day one — it is a 20-line function and it makes every downstream query trivial. Conversely, I built dashboards first and threw most of them away. Skip dashboards until you have the queries that have already changed a decision; dashboards are for surfacing those queries to other people, not for finding them.

## Links

- OpenAI usage object: https://platform.openai.com/docs/api-reference/chat/object
- Anthropic usage and cache reporting: https://docs.claude.com/en/docs/build-with-claude/prompt-caching
- DuckDB: https://duckdb.org
- ClickHouse: https://clickhouse.com
- OpenTelemetry log data model: https://opentelemetry.io/docs/specs/otel/logs/data-model/
