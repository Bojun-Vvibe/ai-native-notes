---
title: "MCP server patterns in production: what survives contact with real users"
date: 2026-04-23
tags: [mcp, agents, architecture, production]
est_reading_time: 15 min
---

## TL;DR

Model Context Protocol (MCP) servers are easy to write and hard to operate. The reference examples make a server look like "wrap a function, return JSON" — and for a single developer on a single laptop, that is true. The moment a server is shared across a team, called by an autonomous agent, or wired into a paid model's tool loop, four problems show up that the spec does not solve for you: **auth boundaries, idempotency, response shaping, and failure semantics**. This post is the set of patterns I have converged on after running ~a dozen MCP servers in production for several months. None of these are novel ideas in distributed systems; the contribution is mapping them onto the specific shape of MCP, where the "client" is a token-billed LLM that will retry creatively when confused.

## The problem

The first MCP server I shipped was a thin wrapper around a Postgres read replica. It worked perfectly in demos. The first day a team of agents pointed at it, three things happened: (1) one agent looped on the same query 40 times because the response was a 200KB JSON blob and the model kept re-asking for "a smaller version"; (2) another agent ran an `EXPLAIN ANALYZE` on a heavy query, hit a 30-second statement timeout, retried, and pinned a CPU; (3) a third agent fed the output of one tool directly back into another tool's input field that expected a different shape, and got a 500 with no useful message. None of these are bugs in MCP. They are the consequences of building a synchronous, untyped-from-the-client's-perspective RPC surface and pointing a probabilistic caller at it.

## The setup

- MCP servers written in TypeScript using the official `@modelcontextprotocol/sdk`.
- Hosts: a mix of agent CLIs (open-source coding agents) and a small in-house orchestrator.
- Transport: stdio for local servers, streamable HTTP for shared ones behind an internal gateway.
- Observability: structured logs to a local file plus OpenTelemetry traces to a shared collector.

The patterns below assume you have at least: a logger, a request ID per call, and the ability to deploy a new version of the server without coordinating with every agent.

## Pattern 1: auth at the edge, identity in the call

The MCP spec leaves auth largely up to the transport. For stdio servers, "auth" is "the user who launched the server." For HTTP servers, you have real options, and the temptation is to put auth inside the server next to the business logic. Don't.

The pattern that has held up: terminate auth at a small gateway in front of the server, and pass a verified identity into the server as a request header. The server itself should never see a raw token.

```ts
// gateway (simplified) — verifies a JWT and forwards.
app.post("/mcp", async (req, res) => {
  const token = req.header("authorization")?.replace(/^Bearer /, "")
  const claims = await verifyJwt(token, JWKS_URL)
  if (!claims) return res.status(401).end()

  // Strip the token, inject identity. The server trusts the gateway, not the client.
  const upstream = await fetch(MCP_UPSTREAM, {
    method: "POST",
    headers: {
      "content-type": "application/json",
      "x-actor-sub": claims.sub,
      "x-actor-scopes": (claims.scopes ?? []).join(","),
      "x-request-id": req.header("x-request-id") ?? crypto.randomUUID(),
    },
    body: JSON.stringify(req.body),
  })
  res.status(upstream.status)
  upstream.body?.pipeTo(Writable.toWeb(res))
})
```

Inside the server, every tool handler reads `x-actor-sub` and `x-actor-scopes` and decides what the actor is allowed to do. Two benefits: (1) you can swap auth providers without touching tool code; (2) you can run the server unauthenticated in dev with a fake header, which makes local testing painless.

A subtler benefit: when an agent uses your tool on behalf of a user, you get a verified `sub` you can audit. "The agent did it" is not an acceptable answer to a security question; "the agent acting as user X did it, here is the request ID, here is the tool input" is.

## Pattern 2: idempotency keys, because models retry

Frontier models retry tool calls when responses look wrong. This is a feature for read-only tools and a disaster for write tools. The fix is the same one every payments API learned a decade ago: idempotency keys.

For an MCP write tool, the convention I use:

- The tool schema exposes an optional `idempotency_key` argument.
- If the model omits it, the server generates one from a hash of `(actor, tool_name, normalized_args, minute_bucket)` and logs a warning.
- The server stores `(idempotency_key) -> (first_response)` in a small KV with a TTL of, say, 24 hours.
- Repeat calls return the stored response *with a flag in the metadata* that says "this is a replay."

```ts
server.tool("create_ticket", createTicketSchema, async (args, ctx) => {
  const key = args.idempotency_key
    ?? hash(ctx.actor, "create_ticket", normalize(args), bucketMinute())

  const cached = await idempo.get(key)
  if (cached) {
    return { ...cached, _meta: { replay: true, original_ts: cached.ts } }
  }

  const result = await ticketing.create({ ...args, requested_by: ctx.actor })
  const payload = { ticket_id: result.id, url: result.url, ts: Date.now() }
  await idempo.set(key, payload, { ttlSeconds: 86400 })
  return payload
})
```

Two notes on the response. First, returning the cached payload with a `replay: true` flag is more useful than returning a 409 — the model treats it as a successful call, doesn't loop, and the audit log still shows the retry. Second, `normalize(args)` matters: if the model sends `{"title": "Bug ", "body": "..."}` once and `{"title": "Bug", "body": "..."}` (trailing space stripped) the next, you want the same key. Pick a normalization (trim, lowercase enums, sort arrays, drop nulls) and document it.

## Pattern 3: response shaping for an LLM consumer

The most expensive class of bugs I have seen in MCP servers is **response shape that is correct but unfit for an LLM**. A 200KB JSON blob with 40 fields per row is correct. A model fed that blob will either (a) burn tokens summarizing it, (b) re-ask the tool with narrower filters it has to guess at, or (c) hallucinate field names that don't exist.

The pattern I use: every tool returns a pair, `{summary, data}`, where `summary` is short prose and `data` is structured. The summary is what the model reads first and what most loops will be satisfied with; the structured `data` is there for when the model genuinely needs to operate on it.

```ts
server.tool("search_docs", searchSchema, async (args, ctx) => {
  const hits = await index.search(args.query, { limit: args.limit ?? 10 })

  const summary = hits.length === 0
    ? `No matches for ${JSON.stringify(args.query)}.`
    : `Found ${hits.length} matches; top result: "${hits[0].title}" `
      + `(${hits[0].path}, score ${hits[0].score.toFixed(2)}).`

  return {
    summary,
    data: hits.map(h => ({
      title: h.title,
      path: h.path,
      score: Number(h.score.toFixed(3)),
      snippet: truncate(h.snippet, 240),
    })),
  }
})
```

Three concrete rules that fall out of this pattern:

1. **Truncate strings inside the response, not just the response.** A single 50KB log line will blow your token budget even if there are only three of them.
2. **Round numbers.** A model does not need 14 digits of float precision in a search score, and the extra digits cost real money over a day of tool calls.
3. **Drop null/empty fields.** `{"description": null}` teaches the model nothing and costs tokens.

For tools that return genuinely large data (file contents, query results), add an explicit `cursor` argument and return paginated chunks. Do not rely on the model to "ask for less."

## Pattern 4: failure semantics the model can act on

When a tool fails, the model has to decide: retry, give up, ask the user, or try a different tool. The decision is only as good as the error message. The reference servers tend to return raw exception strings, which leads to two failure modes: the model retries unrecoverable errors, or it gives up on transient ones.

The pattern: classify every error into one of a small set of categories and put the category in a structured field.

```ts
type ToolError = {
  ok: false
  category:
    | "bad_input"        // model's fault — do not retry, ask user or fix args
    | "not_found"        // resource doesn't exist — do not retry
    | "permission"       // actor not allowed — do not retry, escalate
    | "transient"        // network/db blip — retry with backoff is fine
    | "rate_limited"     // explicit backoff signal
    | "timeout"          // operation exceeded budget — maybe retry, maybe narrow
    | "internal"         // bug — do not retry, surface to operator
  message: string         // short human-readable
  retry_after_ms?: number // present iff transient/rate_limited
  request_id: string
}
```

Then teach the system prompt (once) what each category means. The categories are stable, so a single line in the prompt covers every tool:

> Tool errors include a `category` field. Only retry `transient`, `rate_limited`, and (with narrower input) `timeout`. For `bad_input` or `permission`, surface the issue to the user; do not loop.

This single change — categorized errors plus one line of system prompt — cut my "agent is stuck in a tool-call loop" incidents by something like 80%. The remaining 20% were genuine bugs in the agent's decision-making, not in the tool surface.

## Pattern 5: per-actor budgets, not just rate limits

Rate limits ("100 requests per minute per IP") are necessary but not sufficient. The failure mode they don't catch: a single agent session that, while well under the rate limit, racks up an hour of compute because the model is exploring. The pattern that works is a **per-actor budget** measured in whatever resource actually hurts (database CPU-seconds, external API spend, model tokens consumed by tool outputs).

A minimal implementation:

```ts
server.tool("run_query", runQuerySchema, async (args, ctx) => {
  const remaining = await budget.remaining(ctx.actor, "db_cpu_seconds")
  if (remaining <= 0) {
    return errorResponse("rate_limited", "actor budget exhausted",
      { retry_after_ms: msUntilNextWindow() })
  }
  const t0 = process.hrtime.bigint()
  try {
    return await db.runWithTimeout(args.sql, { maxSeconds: Math.min(remaining, 10) })
  } finally {
    const elapsedSec = Number((process.hrtime.bigint() - t0) / 1_000_000n) / 1000
    await budget.spend(ctx.actor, "db_cpu_seconds", elapsedSec)
  }
})
```

Budgets work best when they're visible to the model. Returning the remaining budget as part of the response metadata (`_meta: { budget_remaining: 42.3 }`) gives the agent a chance to slow down on its own. You will be surprised how often it actually does.

## Pattern 6: schema evolution without breaking the model

MCP tool schemas are part of the model's context. Changing a schema is a prompt change. Two failure modes I have hit:

1. Renaming a field from `customer_id` to `account_id`. Old sessions that were paused mid-loop resumed with a tool call using `customer_id`, got `bad_input`, and gave up.
2. Tightening a regex on a string argument. Model had been generating values that worked for weeks, suddenly started getting validation errors, retried with mutations that were *more* wrong.

Rules I follow now:

- **Never rename a field; deprecate.** Add the new field, accept both for a release cycle, log when the old field is used, then remove.
- **Loosen, don't tighten, validation in place.** If a stricter rule is needed, add a new tool with a different name and migrate.
- **Version the server, not the tool.** Bump the server's reported version and let hosts negotiate. Tool names should be stable.
- **Keep schemas small.** A tool with 12 optional arguments is a tool the model will call with a different subset every time, making your usage logs useless for refactoring.

## Pattern 7: observability that survives an autonomous caller

Logs designed for human callers ("user X clicked button Y") fall apart when the caller is an agent making 200 tool calls per session. The pattern: emit one structured log line per tool call with a fixed shape, plus a span if you have a tracing backend.

```ts
function instrument<T>(name: string, fn: ToolHandler<T>): ToolHandler<T> {
  return async (args, ctx) => {
    const start = Date.now()
    const reqId = ctx.requestId
    let outcome: "ok" | "error" = "ok"
    let category: string | undefined
    try {
      const result = await fn(args, ctx)
      if ((result as any)?.ok === false) {
        outcome = "error"
        category = (result as any).category
      }
      return result
    } catch (e) {
      outcome = "error"
      category = "internal"
      throw e
    } finally {
      log.info({
        evt: "mcp.tool.call",
        tool: name,
        actor: ctx.actor,
        session: ctx.sessionId,
        request_id: reqId,
        outcome,
        category,
        duration_ms: Date.now() - start,
      })
    }
  }
}
```

The shape matters more than the contents. Once every tool emits the same fields, you can answer questions like "which tool errors out most for which actor" with a single query, and you can build a dashboard that does not need to be updated when you add a new tool.

## Anti-patterns I have removed

A short list of things I tried that turned out to be bad ideas:

- **Auto-retrying inside the server.** It hides transient errors from the model, which then can't learn to back off, and it doubles the load when something is genuinely down.
- **Returning HTML or Markdown in tool responses.** The model will sometimes echo it verbatim into chat. Return data; let the host render.
- **One giant "do everything" tool with a `mode` argument.** Models cannot reliably pick the right mode. Three small tools with clear names beat one polymorphic tool every time.
- **Streaming partial JSON.** The current generation of agent loops doesn't make use of it, and partial JSON makes error handling much harder. Wait until the final shape is ready, then return.

## What I would do differently

Start every MCP server with the pair `{summary, data}` response shape and the categorized error envelope. Those two decisions are nearly free on day one and very expensive to retrofit on day 90. Auth and budgets can wait until you have a second caller; observability cannot — add structured logs from the first deployment.

## Links

- Model Context Protocol spec: https://modelcontextprotocol.io
- Official SDKs: https://github.com/modelcontextprotocol
- Stripe's idempotency design (still the canonical reference): https://stripe.com/blog/idempotency
- OpenTelemetry semantic conventions: https://opentelemetry.io/docs/specs/semconv/
