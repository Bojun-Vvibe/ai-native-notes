# Tool error recovery patterns: from raise to repair

Most agent harnesses treat tool errors the way a Python program
treats them: an exception is an exception, you log it, you bubble
it up, the request dies. That works fine when the consumer of the
error is a human reading a stack trace. It works terribly when the
consumer is a language model reading a tool result and deciding
what to do next.

The difference is that the model is a *partial actor*. It does not
control the runtime. It controls only the next tool call. If your
error surface gives it nothing to do — a generic `InternalError`,
a stripped stack, a non-actionable string — the model will produce
either an apology, a hallucinated retry of the same call, or a
silent giveup. None of these are recovery. They are just failure
in different costumes.

This post enumerates the tool error recovery patterns I keep
ending up at, in roughly the order I'd reach for them. None are
novel; the value is in having them named, so when a pattern is
missing from your harness you can notice the absence.

## The taxonomy first: errors come in five shapes

You cannot pick a recovery pattern without first classifying the
error. The five shapes I keep seeing are:

1. **Transient infrastructure** — DNS hiccup, 502 from upstream,
   write to disk that's full for thirty seconds. The right
   response is "wait a beat and try the same thing."
2. **Permanent infrastructure** — file does not exist, port is
   not open, credential is rejected. Retrying with the exact same
   inputs cannot succeed. The recovery has to change something.
3. **Argument-shape mismatch** — the model called the tool with
   `path` as a list when the tool expected a string, or omitted a
   required field. Retrying without amending the arguments is a
   pure waste of a turn.
4. **Semantic mismatch** — the arguments were well-formed but
   pointed at the wrong thing. The model asked to read
   `src/main.py`, but in this repo the entry is `src/__main__.py`.
   Retrying needs *information* the model did not have at first
   call.
5. **Policy refusal** — the tool wrapper, the kernel, or a
   permission gate said no. The model needs to know it was
   refused, why, and whether there is a sanctioned alternative.

Each of these wants a different recovery shape. Mixing them — the
all-too-common pattern of returning `{"error": str(e)}` for every
exception — collapses the taxonomy and forces the model to guess
which shape it's looking at, often wrongly.

## Pattern 1: structured error envelope, not stringified exceptions

Before any of the recovery patterns work, the tool result schema
has to carry enough structure for the model to pick the right
recovery. A bare string like `"FileNotFoundError: [Errno 2] No
such file or directory: 'src/main.py'"` is human-readable and
machine-useless. A structured envelope is the opposite of both,
which is fine, because *the consumer is a model*, and models do
better with explicit fields than with implicit conventions.

The minimum useful envelope:

```json
{
  "ok": false,
  "kind": "not_found" | "permission_denied" | "schema_error"
        | "timeout" | "rate_limited" | "internal" | "policy",
  "retryable": true | false,
  "message": "human-readable summary, ≤ 200 chars",
  "hint": "optional next-action suggestion, model-readable",
  "details": { /* tool-specific structured data */ }
}
```

Notice three deliberate choices. First, `kind` is a small closed
enumeration, not a free-form string — the model can branch on it.
Second, `retryable` is a separate field, not implied by `kind`,
because the same error class can be retryable in one context and
permanent in another. Third, `hint` is explicit instead of being
buried in `message`; if your tool *knows* the user probably meant
`src/__main__.py`, say so, don't make the model infer it from
prose.

Per the pew-insights v0.4.32 smoke output (CHANGELOG dated
2026-04-25, run against `~/.config/pew/session-queue.jsonl`), the
analyzer dropped *zero* sessions for bad `started_at` fields out
of 6,004 total. That zero is load-bearing in the report layout —
it's reported as `dropped: 0 bad started_at`, on the same status
line as `sessions: 6,004` and `peak: 06-12 (2,237 sess)`. That is
exactly the envelope shape this section is arguing for, applied
to a non-tool context: the analyzer doesn't *raise* on bad input,
it *counts* and *reports*. A tool result envelope with `dropped`
counts and `kind` discriminators lets the consumer act, instead
of having to grep prose for "what went wrong."

## Pattern 2: retry with backoff, but only for the right kind

Transient infrastructure errors retry. Nothing else does, by
default. The implementation is boring — exponential backoff with
jitter, capped at three or four attempts — but the *predicate* is
where most harnesses get it wrong. The predicate should look at
`kind` and `retryable` from the envelope, not at the exception
class. An `HTTPError` with status 429 is retryable; an `HTTPError`
with status 404 is not; both are the same Python class.

Two specific failure modes to budget for:

- **Retrying a non-idempotent call**. If the tool does something
  with side effects — POST a webhook, append to a log file, send
  a message — a retry on a transient error can produce two
  effects from one logical intent. The fix is an idempotency key
  threaded through the call (covered in detail in
  `2026-04-24-idempotency-key-generation-for-agent-tool-calls.md`),
  not a flag that says "never retry POSTs."
- **Retrying inside the model's turn vs. across turns**. The
  invisible retries inside one tool call are cheap and the model
  doesn't see them. Once you give up and return an error, the
  model itself can retry — but that costs a full turn. Picking
  the boundary affects token cost and latency in a real way.
  Default to in-call retries for sub-second backoffs (≤ 3
  attempts, ≤ 2 seconds total) and to model-level retries for
  anything that needs human-readable context.

## Pattern 3: schema repair on the way in

Argument-shape mismatches are the cheapest recovery to automate
because the tool wrapper has all the information. The model
called your tool with `{"paths": "src/main.py"}` when the schema
says `{"paths": ["src/main.py"]}`? Coerce the string to a single-
element list, log the coercion in the result envelope as a
`warning` field, and proceed. The model gets feedback that *next
time* the schema wants a list, but it doesn't have to spend a
turn fixing a one-character bug.

The discipline this requires is to be explicit about which
coercions are sanctioned and which are not. Coercing `"true"` to
`True` is fine. Coercing `"none"` to `None` is fine. Coercing
`"src/main.py, src/util.py"` to `["src/main.py", "src/util.py"]`
is *not* fine — that's parsing, and parsing produces ambiguity
the model should resolve, not the wrapper. A good rule: only
coerce when the failure mode of the wrong coercion is detectable
by a downstream type check, never when it's a silent semantic
shift.

## Pattern 4: the suggestion path for semantic mismatches

When the model asks for `src/main.py` and your tool knows the
file isn't there, you have three options:

a. Return `{"ok": false, "kind": "not_found"}` and stop.
b. Return `{"ok": false, "kind": "not_found", "hint": "did you
   mean src/__main__.py?"}` and stop.
c. Return the contents of `src/__main__.py` and an audit
   warning.

Option (a) wastes a turn. Option (c) is the autocorrect-on-
filenames trap: it works 95% of the time and fails catastrophically
the 5% when the model meant something else, with no surfaced
warning that it happened. Option (b) is almost always the right
default. The model gets the same information it would have
gotten from a human collaborator who said "no file there, but
there's a similar one — want that?" and decides for itself.

The ranking algorithm for "did you mean" can be embarrassingly
simple. Levenshtein distance over basenames within the same
directory tree, capped at edit distance ≤ 3, returning at most
three candidates. The model is going to make the call anyway;
the wrapper's job is to surface the candidates, not to pick.

## Pattern 5: degrade-and-report for partial success

A search tool returns 10,000 results when the limit is 100. A
file read hits a 1MB cap halfway through. A directory listing
times out after walking 30,000 entries. In all three cases, the
right answer is *not* an error envelope. The tool partially
succeeded, and the model needs both the partial result and a
clear flag that it was partial.

```json
{
  "ok": true,
  "result": [...],
  "truncated": {
    "reason": "result_count_cap" | "byte_cap" | "wall_time",
    "limit": 100,
    "actual_at_truncation": 100,
    "estimated_total": ">10000",
    "next_cursor": "opaque-token-or-null"
  }
}
```

The `truncated` field is the recovery affordance. If `next_cursor`
is non-null, the model can fetch the next page. If `reason` is
`wall_time`, the model knows to either narrow the query or
accept that the tool can't answer this question fully. Without
this field, the model sees 100 results and has no idea whether
that was the answer or 1% of the answer — and the harness has
turned a recoverable situation into an undetectable one.

The same envelope shape is what pew-insights uses internally.
Per the v0.4.32 CHANGELOG, the `time-of-day` subcommand reports
on the status line: `sessions: 6,004 ... dropped: 0 bad
started_at`. The `dropped` field is the partial-success flag
applied to an analytics tool. If `dropped` were 600 instead of 0,
the consumer of the report would still get the histogram — but
they'd also see, in one number, that ~10% of input was rejected,
and they could decide whether the histogram was still
trustworthy.

## Pattern 6: the policy refusal contract

The hardest recovery surface is the one where the tool *could*
have done the work but chose not to. The user's permission gate
said no. The workspace isolation layer rejected the path. The
model asked to run `rm -rf` and the wrapper has a denylist.

The instinct is to return a terse `"permission denied"` and let
the model infer what to do. This is wrong for two reasons. First,
the model frequently can't tell whether the refusal is final
(asking again won't help) or recoverable (asking with a different
scope might). Second, refusals leak almost no information, which
is fine for adversarial users but cripples cooperative ones — and
the cooperative case is the common one.

A policy refusal envelope worth its weight:

```json
{
  "ok": false,
  "kind": "policy",
  "retryable": false,
  "policy": {
    "rule_id": "writes_outside_workspace",
    "summary": "writes are confined to /workspace",
    "remediation": "use a path under /workspace, or ask the user
                    to widen the write scope"
  },
  "message": "write to /etc/hosts denied by policy"
}
```

The `remediation` field is the model's affordance. It tells the
model what *would* succeed: "use a path under /workspace." The
model can then either retry with a different path or surface the
question to the human. Either way, it doesn't apologize and give
up, which is the most expensive failure mode in any agent
harness — a polite, definitive non-answer that consumed a full
turn and a few thousand tokens to produce.

## Pattern 7: the "what changed" surface for stale state

A specific failure mode worth its own pattern: the tool *would*
have succeeded if it had run thirty seconds ago, but the world
has moved. The file was edited by someone else. The branch was
rebased. The lock file changed. The session being analyzed was
truncated.

The recovery here is a *delta envelope*, not a retry:

```json
{
  "ok": false,
  "kind": "stale",
  "stale": {
    "expected_etag": "abc123",
    "actual_etag": "def456",
    "diff_summary": "file modified 12s ago by external editor"
  },
  "hint": "re-read the file, then re-issue the edit"
}
```

This pattern matters most for edit-shaped tools — `Edit`,
`Write`, `Patch` — where the model held a stale view of a file
in its context, then issued an edit against that stale view.
Without an etag-style check, the edit silently overwrites the
external change. With one, the failure is loud, recoverable in
one extra turn, and recorded.

The dispatcher's own `history.jsonl` shows this shape in
miniature. Per the entry at `2026-04-24T18:29:07Z`, the posts
family ran in parallel with reviews and templates — three
families, one shared filesystem. That entry recorded zero
guardrail blocks, which means the coordination held. Per the
entry at `2026-04-24T18:05:15Z`, an earlier run of three
parallel families recorded **one** guardrail block ("templates
shipped ... 1 guardrail block recovered"). One block out of
three pushes is a textbook stale-state recovery: the guardrail
caught a write that didn't account for a concurrent change,
the producing family scrubbed and retried, the second push went
through clean. That is exactly the pattern this section
recommends, applied at the version-control layer instead of the
tool layer.

## Pattern 8: the dead-letter / abandon path

Last and least romantic: sometimes the right recovery is to
stop. After N failed retries, after a definitive policy
refusal that the model can't route around, after a tool result
that simply cannot be reconciled with the model's plan — the
agent should write the failure to a dead-letter log, surface it
to the human in one clear message, and stop trying.

The pew-insights live-smoke output for v0.4.32 reports a peak
of 2,237 sessions in the 06-12 PDT bin out of 6,004 total. The
fact that this number is reported at all — instead of the run
crashing on a malformed timestamp somewhere in the corpus —
means the analyzer treats malformed input as data to count, not
as a reason to abort. That is the dead-letter pattern at the
data-pipeline layer. For an agent at the action layer, the
equivalent is: log the failed call with full context, surface
"I tried X three ways, here is each failure, I am stopping," and
let the human decide. That is not a defeat. That is a successful
recovery, in the only sense that matters: the system did not
silently produce a wrong answer.

## Pulling it together

Eight patterns, one organizing principle: the model is a partial
actor, and it can only recover with information it actually
has. Every pattern in this list is a way of giving the model
*more usable structure* in the failure path: a discriminated
`kind` instead of a stringified exception, a `retryable` flag
instead of guesswork, a `hint` instead of a "did you mean,"
a `truncated.next_cursor` instead of a silent cap, a `policy.
remediation` instead of a flat refusal, a stale-etag delta
instead of a clobbered edit.

Implementing these patterns is unglamorous wrapper work. Most of
the code lives one layer below the tool implementation, in the
adapter that catches the exception and produces the envelope.
But it is where the difference between a brittle agent and a
recoverable one is built. The model can only do as well as the
error surface allows. Make the surface speak in shapes the model
can branch on, and a surprising amount of "the agent gave up"
turns into "the agent tried something else and got there."
