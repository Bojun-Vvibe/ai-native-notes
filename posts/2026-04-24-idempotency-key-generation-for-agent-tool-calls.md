---
title: "Idempotency keys for agent tool calls: it's the key generation, not the retry"
date: 2026-04-24
tags: [agents, tool-calls, idempotency, reliability, distributed-systems]
est_reading_time: 9 min
---

## The problem

I had an agent loop that posted to a webhook, retried on 5xx, and ended up
posting the same payload three times to a downstream system that was, in fact,
healthy the whole time. Classic. The "fix" most people reach for is "add an
idempotency key," and that is correct, but it is only half a fix. The hard
problem is not that idempotency keys exist. The hard problem is **how the agent
generates the key**, because the agent is non-deterministic, the tool call is
non-deterministic, and the wrapper code that the agent never sees is also
making decisions about retries.

I spent a few days last week ripping through three different agent frameworks
and a hand-rolled MCP server looking for where idempotency keys come from in
each. The answer, mostly, is "they don't, and when they do, the key is wrong."
This post is about why the key-generation step is the load-bearing one and
what the tradeoffs are between four strategies I actually shipped.

## The setup

Concretely, I was looking at four side-effecting tool surfaces:

1. A webhook poster (`POST` to a Slack-compatible incoming webhook).
2. An issue-creator (`POST /repos/:owner/:repo/issues` against a GitHub
   instance).
3. A "publish artifact" tool that copied a file into an S3-compatible bucket.
4. An email-send tool over SMTP through a relay that supports the
   `Message-ID` dedup header.

The agent loop was a standard ReAct-ish loop: model emits a `tool_use` block,
the host executes the tool, returns a `tool_result`, model decides what to do
next. Retries happened at three layers: the HTTP client (transport-level), the
tool host (after a tool throws), and the agent itself (after a `tool_result`
that "looks failed"). Each of these layers will, given the chance, replay your
side effect.

The model was Claude Sonnet 4.5 in one run and a local Qwen-class model in the
other. The bug shape was identical in both, which is the point — this is not a
model-quality problem.

## What I tried

- **Attempt 1: no key, just retry on 5xx.** This is the default in most HTTP
  clients. It works for `GET`. It does not work for `POST` against a service
  that does not itself dedup. I got 3× sends every time the network blipped
  during the response phase (request landed, response lost).
- **Attempt 2: random UUID per call.** The agent host generated a fresh
  `Idempotency-Key: <uuid>` per outbound request and sent it on every retry of
  *that* request. This handled transport-level retries — the HTTP client
  re-sent with the same key. It did NOT handle agent-loop retries, because the
  agent decided the call had failed (timeout, garbled response) and emitted a
  *new* `tool_use` block, which generated a *new* UUID, which the downstream
  service treated as a brand-new request.
- **Attempt 3: content hash of the request body.** `sha256(body)` as the
  idempotency key. This handled the agent-retry case: if the agent emitted the
  same `tool_use` block twice, the body was byte-identical, so the key
  collided, so the downstream service correctly deduped. This broke when the
  body legitimately had a timestamp, a nonce, or any other field the agent
  filled in differently on the second attempt. Two semantically-identical
  requests now had different hashes.
- **Attempt 4: caller-supplied semantic key.** The tool *schema* declared an
  `idempotency_key` field; the agent was instructed to pick a key that
  identified the *intent* ("post-tick-summary-2026-04-24-T06"), not the
  payload. This worked when the agent followed instructions. It failed
  silently when it didn't — the agent invented a different key on retry, and
  we were back to attempt 1.
- **Attempt 5: host-derived semantic hash.** The host computed the key from a
  *normalized* projection of the request body — strip volatile fields
  (timestamps, nonces, request IDs), canonicalize JSON key order, hash the
  rest. The model never saw the key; the host owned it. This is what I shipped.

## What worked

The combination that actually held up under three different fault injections
(network blip mid-response, agent-loop retry, host crash mid-call):

```ts
// host-derived idempotency key, computed from a normalized projection
// of the tool input. The MODEL never sees this key. The HOST owns it.

import { createHash } from "node:crypto";

// per-tool config — declared once, reviewed by humans, frozen in source
type ToolKeyConfig = {
  // fields that participate in identity ("what is this call about")
  identityFields: string[];
  // fields stripped before hashing (timestamps, nonces, request IDs,
  // anything the agent fills in differently on retry)
  volatileFields: string[];
  // an optional scope — usually a session ID or a "logical operation" ID
  // that the host injects from outside the agent's control
  scope: () => string;
};

function deriveIdempotencyKey(
  toolName: string,
  input: Record<string, unknown>,
  cfg: ToolKeyConfig,
): string {
  const projected: Record<string, unknown> = {};
  for (const f of cfg.identityFields) {
    if (f in input) projected[f] = input[f];
  }
  // canonicalize: sorted keys, stable JSON
  const canonical = JSON.stringify(
    projected,
    Object.keys(projected).sort(),
  );
  const h = createHash("sha256");
  h.update(toolName);
  h.update("\x00");
  h.update(cfg.scope());
  h.update("\x00");
  h.update(canonical);
  return h.digest("hex").slice(0, 32);
}
```

The webhook tool config:

```ts
const webhookKeyConfig: ToolKeyConfig = {
  identityFields: ["channel", "text", "thread_ts"],
  volatileFields: ["timestamp", "client_msg_id", "trace_id"],
  scope: () => currentTickId(), // injected by the host
};
```

The issue-creator config:

```ts
const issueKeyConfig: ToolKeyConfig = {
  identityFields: ["owner", "repo", "title", "body"],
  volatileFields: ["nonce", "submitted_at"],
  scope: () => currentSessionId(),
};
```

Two things to notice. First, the `scope()` function is injected by the host
and is **not visible in the agent's context window**. The agent cannot
accidentally — or, in adversarial settings, deliberately — change the scope
to bypass deduplication. Second, `identityFields` is an explicit allowlist,
not a denylist of `volatileFields`. New fields default to NOT participating in
identity, which means a schema migration that adds `request_priority` to the
issue-create payload won't silently change every key in your system.

## Why it worked (or: my current best guess)

The reason caller-supplied keys (attempt 4) failed and host-derived semantic
keys (attempt 5) succeeded is that the agent is not a reliable source of
intent — it is a reliable source of *content*. If you ask the agent for a
key, you are asking it to summarize its own intent into a string, which is a
generation task and therefore non-deterministic. If instead you ask the agent
for a payload and let the host project a key out of that payload, you have
moved the problem from generation (hard, non-deterministic) to projection
(easy, deterministic).

The retry-storm scenario is then very clean:

| Scenario                          | Agent action               | Payload      | Key       | Result                |
|-----------------------------------|----------------------------|--------------|-----------|-----------------------|
| Network blip mid-response         | none (HTTP client retries) | identical    | identical | dedup at server       |
| Tool host crashes mid-call        | host re-runs same input    | identical    | identical | dedup at server       |
| Agent retries after `tool_error`  | agent re-emits tool_use    | ~identical*  | identical | dedup at server       |
| Agent retries with edited payload | agent emits modified call  | different    | different | new call (correct)    |

The asterisk on row 3 is where most of the engineering goes. The agent's
re-emission is *almost* identical but usually has small differences — a
slightly reworded `text` field, a regenerated trace ID. The `volatileFields`
allowlist absorbs the regenerated trace ID. The slightly reworded `text`
field will, in fact, produce a different key, and that is the *correct*
behavior — the agent is making a different decision the second time, and you
should treat that as a different call.

This last point is where I disagree with the "the agent should always retry
with byte-identical payloads" school. If the agent has new information after
the failure (it saw the error message, it re-read its own scratchpad), it
*should* be allowed to make a different call. The idempotency layer's job is
to dedup *the same decision*, not to forbid *different decisions*.

## The four strategies, ranked

1. **Host-derived semantic hash.** Best default. Agent owns content, host
   owns identity. Works under all four retry scenarios above.
2. **Caller-supplied semantic key.** Useful when the agent has external
   context the host doesn't (e.g. "this is a retry of yesterday's failed
   billing run"). Fragile in practice — agents drift.
3. **Content hash of full body.** Use only when the body has no volatile
   fields. The moment someone adds a timestamp, you're back to attempt 1
   without realizing it. Add a CI check that fails if a tool input schema
   contains anything timestamp-shaped.
4. **Random UUID per call.** Useful for transport-level retries only. Pair
   with one of the above; do not use alone.

## Failure modes I missed at first

A few things bit me before I converged on the host-derived approach. Naming
them so you can shortcut to the answer:

- **Scope too narrow.** I first set `scope` to "the current tool_use block
  ID." That made attempt-2's UUID-per-call problem worse, not better, because
  every replay of the block had a fresh block ID. Scope must outlive the
  block — usually a session, a tick, or a user-initiated operation.
- **Scope too wide.** When I set `scope` to "the current user," two
  legitimately-distinct calls with the same payload (same Slack channel, same
  text — yes, this happens, "ping" gets sent more than once) deduped into
  one. Scope should be the smallest unit that *contains the retries* but
  *does not contain unrelated repeated intents*. For my case, "the current
  tick" was right; for a chat product, "the current user turn" is probably
  right.
- **Hash truncation collisions.** I truncated the SHA-256 to 16 hex chars
  (64 bits) and got a collision in production within a week on a high-volume
  surface. 32 hex chars (128 bits) has been fine. Don't over-truncate to
  save log bytes.
- **Server doesn't actually dedup.** Half the "idempotency-key-aware"
  services I tested either ignore unknown headers or only honor the key for
  ~24 hours. Test with a real duplicate before trusting it. The webhook
  endpoint I was using turned out to dedup on `(channel, text)` for 5
  seconds and ignore `Idempotency-Key` entirely. The actual fix there was
  client-side: the host kept a small in-memory LRU of `(toolName, key) →
  result` for the last N minutes and short-circuited replays before they
  ever hit the wire.
- **JSON canonicalization is not free.** `JSON.stringify(obj, sortedKeys)`
  works for flat objects. Nested objects need a recursive canonicalizer.
  Arrays-of-objects are worst — do you sort the array? It depends on whether
  array order is part of identity. For tool inputs, I sort everything except
  fields explicitly marked `ordered: true` in the schema.
- **The model can see the key in error messages.** If the downstream
  service echoes the idempotency key back in an error response, and your
  tool host surfaces that error verbatim into the next `tool_result`, the
  agent now knows the key and can in principle game it. I redact any field
  matching `/idempotency.?key/i` from tool errors before they re-enter the
  context window. This has not bitten me but feels right.

## A note on observability

The idempotency key is the single most useful thing to log per tool call.
With it you can answer:

- "How many duplicate sends did the agent attempt this hour?" (count of
  cache hits in the host LRU)
- "Which tools have the worst dedup rate?" (key collision rate by toolName)
- "Did the agent retry the same intent or a different intent after this
  failure?" (compare keys before/after the error)

Log `(tickId, toolName, key, hit_or_miss, duration_ms)` per call. That
five-tuple has paid for itself many times over. It's also exactly the shape
you want to back-pressure on — if dedup hit rate spikes above some
threshold, the agent loop is stuck and should be paused, not retried harder.

## What I would do differently

I would skip attempts 1–4 and go straight to host-derived semantic hashing
with an explicit `identityFields` allowlist and a host-injected `scope`.
The interesting design question is not "should we have idempotency keys" —
it is "who owns the projection from payload to identity," and the answer is
always the host, never the model.

## Links

- Stripe's idempotency key docs (the canonical write-up of the wire-level
  contract): <https://docs.stripe.com/api/idempotent_requests>
- AWS's request-ID conventions on side-effecting APIs:
  <https://docs.aws.amazon.com/AWSEC2/latest/APIReference/Run_Instance_Idempotency.html>
- The MCP spec on tool calls and result handling:
  <https://modelcontextprotocol.io/specification>
