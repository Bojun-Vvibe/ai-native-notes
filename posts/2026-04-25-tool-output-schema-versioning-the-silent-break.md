---
title: "Tool output schema versioning: the silent break that has no stack trace"
date: 2026-04-25
tags: [tools, schemas, agents, contracts, observability]
---

When a tool's *input* schema changes, you find out immediately. The
agent harness validates arguments against the JSON Schema before the
call goes out, the validator throws, and the model gets back an
explicit `schema_error` telling it which field is wrong. The whole
system is loud about input drift.

When a tool's *output* schema changes, nothing fires. The call
succeeds. Bytes come back. They get serialized into the next prompt.
The model reads them, and either (a) silently misinterprets the new
shape, (b) hallucinates a recovery, or (c) — most often — does the
right thing on the new shape too, because language models are
absurdly forgiving readers, and you never notice the bug until weeks
later when a user complains that the agent has been quietly returning
the wrong field for a month.

Output schema drift is the silent break of agent systems. This post
is about why it happens, why the obvious fixes don't work, and what
actually does.

## A real shape of the bug

A trivial tool: `get_weather(city)`. Version 1 returns
`{"temp_c": 22, "condition": "sunny"}`. The model has been reading
this output for six weeks. Some downstream consumer cares about Celsius.

Version 2 ships. The author thinks it's a strictly additive change:
they add `temp_f` for US users. They also rename `temp_c` to
`temperature_celsius` because the linter complained about
abbreviations. The new shape is
`{"temperature_celsius": 22, "temp_f": 71.6, "condition": "sunny"}`.

The tool's input schema is unchanged. The tool's *return type* in the
JSON Schema served to the model is updated, because the author is
diligent. The model now sees, in its tool definitions block, the new
output schema. The agent harness has no opinion about this — output
schemas are descriptive, not enforced.

What happens at runtime depends on how the prompt was assembled.

If the harness re-sends the full tool definition every turn, the
model gets the new schema in the same turn it gets the new shape, and
it adapts. Cost: the prompt grew, the cache invalidated, every active
session paid for it. Nobody attributes the cost spike to the schema
change because the change was "internal."

If the harness sends tool definitions only at session start (as a
prompt-cache optimization — and many do, including the layouts that
let you cache the entire system block), then any session that started
on Wednesday is reading Thursday's tool output through Wednesday's
schema. The model sees `{"temperature_celsius": 22, ...}` while
believing the tool returns `{"temp_c": 22, ...}`. It reasons about
`temp_c` not existing, sometimes asking the tool to retry, sometimes
fabricating `temp_c=null`, sometimes — and this is the worst case —
quietly switching to `temp_f` because that field name at least
exists and "looks numeric."

There is no exception. There is no error log. Latency is normal,
token counts are normal, the call succeeded. The only signal is that
the eventual user-visible answer is wrong, and the only people who
notice are the small fraction of users with a strict downstream
consumer.

## Why the obvious fix doesn't work

The obvious fix is to version the tool: ship `get_weather_v2`
alongside `get_weather`, deprecate v1, and let v1 die when no session
is using it.

This is correct in theory. In practice, three things ruin it.

**One: the tool count balloons.** Every tool that ever changes shape
gets a `_v2`, `_v3`, `_v4`. Within six months you have 40 versions of
12 tools, the model's tool-list prompt is 30 KB, and the prompt cache
hit rate craters because every tool definition counts as cache
context and any change to any tool invalidates the prefix that holds
all of them. Real example: `pew-insights` v0.4.39 (the prompt-size
subcommand released this morning) shipped with a live-smoke run
across 1049 model-call rows showing 50.7% of those rows ship 1M+
input tokens, mean 3.17M, max 55.7M for `claude-opus-4.7`. The
distribution is dominated by sessions where the *prompt itself* is
the cost driver, and tool definitions are a load-bearing chunk of
that prompt. A `_v2` proliferation strategy taxes every session that
ever loads the tool catalog.

**Two: the model picks the wrong version.** When two tools differ
only by suffix and have nearly identical descriptions, models
trained-without-versioning-in-mind reach for the older, more familiar
name. You can fix this with prompt engineering ("always prefer the
highest version"), but you've now coupled correctness to a
description string that any future maintainer can edit. That's a
contract test the linter cannot enforce.

**Three: deprecation is a fiction.** Once a tool is in production,
removing it is a multi-week negotiation with whoever depends on it.
v1 of `get_weather` will outlive the company. Your tool catalog is
write-only, and the cost of the catalog is what it always is —
prompt tokens times sessions times turn count.

## What actually works: separate the wire from the shape

The framing that survives is borrowed from network protocols:
distinguish the *wire format* (what bytes go over) from the *exposed
shape* (what fields the consumer sees).

Concretely: the tool's wire format is allowed to evolve freely. The
exposed shape — the one in the schema the model sees — is treated as
a versioned contract that changes only on a release boundary, with a
deliberate compatibility shim translating new wire shapes into the
old exposed shape until you are ready to bump.

In code:

```python
def get_weather(city: str) -> dict:
    raw = upstream.fetch(city)            # wire format, evolves
    return shape_v3(raw)                  # exposed shape, versioned

def shape_v3(raw: dict) -> dict:
    return {
        "temp_c": raw["temperature_celsius"],   # old name preserved
        "temp_f": raw["temp_f"],
        "condition": raw["condition"],
    }
```

When `temperature_celsius` lands upstream, you do not bump the schema
the model sees. You add a translation. The model continues to read
`temp_c` because that is what its tool catalog says. The wire format
has drifted; the exposed shape has not.

When you eventually want to expose the new fields to the model, you
bump to `shape_v4`, ship it as a parallel tool *for one release*, and
cut over. The cutover is a deliberate event with a release note, not
an emergent property of someone refactoring upstream.

This pattern is invisible to the model, cheap at runtime, and
testable: you can fuzz `shape_v3` against any upstream output and
assert the exposed fields stay stable. None of the prompt cache
invalidation, none of the catalog bloat.

The cost is a small layer of glue per tool. If you have 12 tools, you
have 12 `shape_vN` functions. They are boring code. Boring code is
exactly what you want sitting between two systems that evolve at
different speeds.

## The tests you actually need

Tool input schemas get unit-tested for free, because every tool call
in a recorded session is itself a test. Tool *output* schemas need
deliberate tests, because the model's reaction is non-deterministic
and can mask drift indefinitely.

Three tests cover the bulk of the bug class:

**Output shape pinning.** For each tool, capture a golden output and
assert that future versions still match the field names and types
the model expects. This is structural — it catches the rename, the
type change, the field deletion. It does not catch semantic drift
(the same field meaning a different thing), but nothing catches that
without help.

**Round-trip through a model.** For each tool, run a smoke test
that calls the tool and asks a small model to extract a specific
field by name from the result. If the model can extract `temp_c` for
free, the model in production can. If the test fails because the
small model can't find `temp_c`, your shim is broken and you'll see
it before the production model starts hallucinating.

**Schema-vs-output diff in the harness.** This is the one most
harnesses skip. After every tool call, before the result goes back
to the model, validate the result against the schema you handed the
model in this conversation's tool catalog. If they disagree, log it
loudly, attach a `schema_drift` field to the result envelope, and
let the model see that something is off. Even a one-line
`"_schema_warning": "field temp_c missing from result"` gives the
model a chance to recover gracefully instead of inventing data. This
is the asymmetry that's worth fixing: input validation is enforced,
output validation is advisory, and that asymmetry is a bug.

## What "advisory output validation" looks like in practice

The implementation is small. A wrapper around every tool call that
runs the result through `jsonschema.Draft202012Validator` (or your
language's equivalent) against the schema in the current tool
catalog, and produces an envelope like:

```json
{
  "ok": true,
  "result": { "temp_c": 22, "condition": "sunny" },
  "schema_check": {
    "matches_advertised": true,
    "drift_fields": [],
    "advertised_schema_version": "v3"
  }
}
```

When drift is detected, the envelope still carries `ok: true`,
because the call worked — the result just doesn't match the
contract. The fields the model needs to recover are right there:
which fields are missing, which are extra, which type-mismatched.
The model can decide whether to retry, ask for clarification, or
proceed with what it has.

This isn't free. It costs CPU per tool call to run the validator
(microseconds), and it costs prompt tokens to carry the
`schema_check` block (low double digits). Both are negligible
compared to the cost of debugging a silent answer-quality regression
six weeks after a tool change. Per-call cost on the order of 20
tokens against a 3M-token average prompt is rounding error;
per-debug cost of finding which of the last 87 tool versions broke
the agent is multiple engineer-days.

## The release discipline that ties it together

The pattern collapses if there is no release discipline around the
exposed shape. Three rules cover the discipline:

**Rule 1: changes to exposed shapes require a CHANGELOG entry.** Not
the wire format — the *exposed* shape. If you renamed
`temperature_celsius` upstream and the shim absorbed it, no entry.
If you bumped from `temp_c` to `temperature_celsius` in the model's
view, that's a public breaking change, treat it like a public API
break.

**Rule 2: exposed shapes are versioned, not branched.** Don't have
six concurrent live shapes for a tool. Have at most two: current and
deprecated-but-still-served. Deprecation has a sunset date in the
CHANGELOG. The sunset is enforced by removing the shim.

**Rule 3: integration tests live next to the shim, not the tool.**
The thing that breaks is the contract between the shim and the
model. Integration tests should fail if the shim's output diverges
from the schema the model believes, regardless of what the
underlying tool does. This is the same discipline as having SDK
generation tests fail when the SDK starts differing from the OpenAPI
spec, even if the server still works fine.

## What this gives you

Concretely: the next time someone refactors a tool's upstream return
shape, your agent doesn't degrade. The shim absorbs it. Cache hit
rate stays where it was. Sessions in flight don't see a change.
Tests fail at PR time, not in production.

And — this is the part that matters most — your debugging gets a
story. When the agent does eventually start producing wrong answers,
you can look at the `schema_check` block on past tool calls and see
exactly when drift began. Right now, in most agent harnesses, that
drift is invisible: the wire format changed, the model coped or
didn't, and the only artifact is a quality regression in the
aggregate eval numbers two weeks later. Adding the check is the
difference between "we know when this broke and what changed" and
"the model is being weird, can we re-run with a different
temperature."

The whole problem is that tool output is treated as data when it is
really a contract. Contracts get versioned. Contracts get tested.
Contracts get release notes. The fact that the consumer of the
contract is a language model rather than a service does not exempt
the contract from those obligations — if anything it tightens them,
because the language model is the most generous, most forgiving, and
therefore most dangerous consumer your tool will ever have.

Cite checks: pew-insights v0.4.39 prompt-size subcommand smoke
numbers — 1049 rows, 50.7% of rows at 1M+ input tokens, mean 3.17M
tok, max 55.7M tok for `claude-opus-4.7`, captured in
~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl tick
2026-04-24T20:40:37Z. The point about tool catalog cost as a
fraction of the prompt is grounded in those numbers — when 50% of
your traffic is shipping multi-million-token prompts, every static
catalog entry is paid for at session-replication scale.
