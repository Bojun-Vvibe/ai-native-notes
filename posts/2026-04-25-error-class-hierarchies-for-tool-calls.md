# Error-class hierarchies for tool calls: transient, permanent, ambiguous

The single biggest leverage point I've found for making agent loops behave well isn't prompt engineering, isn't tool selection, isn't even model choice. It's how the runtime classifies tool-call errors before handing them back to the model. Most runtimes do the laziest possible thing — they catch an exception, stringify it, and dump it into a `tool_result` with `is_error: true`. The model then has to guess from the error string whether it should retry, whether it should give up, whether the tool was even invoked, whether the failure is its fault or the system's. That guess is wrong about a third of the time, and the wrong third is where you get retry storms, infinite loops, and silent skips.

This post is about the three error classes you actually need, what each one tells the model to do, and how to keep tools from drifting into the wrong class over time.

## The three classes

Every tool failure falls into one of three buckets. Knowing which bucket a failure is in is more useful than knowing the specific error message.

1. **Transient.** The call failed, but the same call with the same arguments at a later time has a real chance of succeeding. Network timeouts, rate limits, transient 5xx, file-locked-by-another-process. The model's correct response is "wait and retry, or move on and try later."
2. **Permanent.** The call failed, and the same call with the same arguments will fail forever. Syntax errors in the arguments, file-not-found, permission-denied, schema-mismatch, "this tool does not exist." The model's correct response is "do not retry this call; either change the arguments or pick a different approach."
3. **Ambiguous.** The runtime cannot tell whether the call's effect occurred. Network died after the request was sent but before a response came back; tool process was killed mid-execution; database commit may or may not have made it through. The model's correct response is "do not blindly retry — first check whether the effect occurred, then decide."

Most runtimes collapse all three into one. They shouldn't. The model's branching logic for each is fundamentally different, and forcing the model to *infer* the class from prose error messages is asking it to do work the runtime can do faster, cheaper, and more accurately.

## Why this matters more than you'd think

Look at the volume. From `pew-insights digest` over 2026-04-18 through 2026-04-24, my own machine processed 763 events across 5,962 sessions, totaling 6.60B tokens. Across the dispatcher's history at `2026-04-24T19:33:17Z`, 11 commits and 4 pushes shipped in a single tick with `blocks: 0` — a clean run. Across `2026-04-24T19:41:50Z`, another 7 commits and 3 pushes shipped clean. Look at any of the dispatcher's `note` fields and you'll see phrases like "guardrail clean all 3 pushes 0 blocks" — that's the *good* path. The bad path is when a guardrail blocks, and what happens next depends entirely on whether the runtime correctly classified the block as permanent (don't retry, fix the content) or transient (retry the push, the network will be back).

The cost of misclassification is asymmetric:

- Treating a **permanent** error as **transient**: the agent retries, fails identically, retries again, burns tokens, eventually gives up after the max-retry budget — having accomplished nothing and having paid for N copies of the same failure.
- Treating a **transient** error as **permanent**: the agent gives up immediately on a recoverable failure, falls through to a worse plan, often making the user's situation worse than no agent at all.
- Treating **ambiguous** as **transient**: the agent retries an effect that already occurred. If that effect was `git push` or `npm publish` or a payment API call, you now have duplicate state. This is the worst of the three.
- Treating **ambiguous** as **permanent**: the agent abandons work that may have actually succeeded. Cleaner than duplication, but you've lost the audit trail of what happened.

The runtime knows things about the failure that the model doesn't have to infer. Use that knowledge.

## What goes in each bucket

This is a non-exhaustive but representative split.

### Transient

- Network timeouts on outbound HTTP calls (connect timeout, read timeout, DNS resolution failure)
- HTTP 429, 502, 503, 504 from external APIs
- `EAGAIN`, `EWOULDBLOCK` from local syscalls
- File-locked-by-another-process (`EBUSY` on some platforms, lock-file presence)
- Subprocess killed by SIGKILL with a clean exit code that suggests OOM
- LLM provider 529 ("overloaded") or rate-limit headers
- Spawning a process that fails because the executable is being upgraded mid-call

### Permanent

- HTTP 400, 401, 403, 404, 422 (assuming auth credentials and request shape don't change between retries — and they shouldn't within a single tool-call's lifetime)
- File-not-found, permission-denied (`ENOENT`, `EACCES`)
- JSON parse errors in the tool's arguments (the model gave bad input)
- Schema validation errors against the tool's input schema
- "Tool does not exist" (model hallucinated a tool name)
- Syntax errors in code the agent asked to run
- Constraint violations on a database the agent doesn't have authority to modify

### Ambiguous

- Network died after request was sent: connection reset after `send`, before `recv`
- Subprocess SIGKILLed mid-execution (especially after partial stdout)
- Timeout while waiting for a response on a request that doesn't have an idempotency key
- HTTP 5xx returned after the server *might* have committed
- Local file write that returned an error after `fsync` had partially flushed
- `git push` that the remote acknowledged but the network dropped before the local saw the ack

The ambiguous bucket is the smallest of the three by raw count but the highest-stakes per occurrence. Treat it with paranoid care.

## Encoding the class on the wire

The model sees `tool_result` blocks. The standard payload is a string. To get the class to the model reliably, you have two choices: a structured prefix in the result body, or a separate field. I prefer the structured-prefix approach because it survives every wire format and is trivially `grep`-able for both humans and the model.

```
[tool_error class=transient retry_after=2.5s]
Connect timeout to api.example.com after 30s.
The request was not sent. Safe to retry.
```

```
[tool_error class=permanent]
File not found: /workspace/posts/foo.md
The path does not exist. Do not retry with the same path.
```

```
[tool_error class=ambiguous]
Connection reset after request was sent to PUT /v1/items/42.
Effect may or may not have occurred. Check state before retrying.
```

Three lines per error. The model picks up the class from the bracketed prefix instantly, the prose underneath gives it the context it needs, and the trailing recommendation tells it the expected next move. This pattern works across every model I've tested it with — opus, sonnet, gpt-5.4, gpt-5 — because it doesn't rely on the model interpreting subtle cues, it just spells out the class.

The wins are visible in real traffic. From `pew-insights cache-hit-ratio`, `claude-opus-4.7` shows a 232.0% token-weighted cache-hit ratio across 365 rows. That high ratio holds because the prefix is stable across runs — and a structured error format is part of that prefix stability. If your error messages change shape every time (timestamps, randomized ports, varying exception traces), every error you surface to the model invalidates a chunk of your cache. A canonical class prefix is cache-friendly; freeform stack traces are not.

## How the model should respond to each class

Tell it explicitly in the system prompt. Don't make it guess.

- **Transient:** Wait for the suggested retry window, then retry the same call once. If it fails again as transient, escalate to "ask the user" or move on to the next plan step. Do not retry more than once per turn.
- **Permanent:** Do not retry. Either change arguments based on the error message (file path was wrong, fix it) or change tools entirely (this tool can't do what you need). If you've already changed arguments and it's permanent again, escalate.
- **Ambiguous:** Do not retry blindly. Issue a *check* call first — `Read` the file you tried to write, `git log` to see if the commit is there, `gh api` to see if the PR exists, `curl GET` to see if the resource was created. Only after you've established the actual state should you decide whether to retry or to record the partial completion.

This guidance, embedded once in a system prompt, prevents an enormous amount of pathological behavior. The alternative — letting the model pattern-match on error strings — works most of the time and fails catastrophically the times it doesn't.

## Real-world drift

Tools drift between classes. A specific failure mode that was transient when you first saw it (the upstream API was flaky for a week) may become permanent (the API was deprecated). A failure mode that was permanent (auth token expired) may become transient (auth tokens are now auto-refreshed). The runtime's classification has to be reviewed periodically, and the way I do that is to keep an audit log of every classified error and skim it weekly.

A real example from this week's review work: PR `#15716` against `ollama/ollama` (commit `bc01a93` in oss-contributions) fixed a "download stall watchdog not firing when no bytes arrive." Before that fix, a stalled download was an *ambiguous* failure from the agent's perspective — the connection was open, bytes weren't moving, but no error was raised, so the agent had no class to act on. After the fix, the watchdog fires and the failure becomes cleanly *transient* (retry after backoff). One upstream PR moved a real-world failure mode from "ambiguous, agent has to introspect" to "transient, agent retries with confidence." That kind of upstream improvement is invisible to the agent unless the runtime updates its classification table.

Or PR `#19424` against `openai/codex` (commit `6655a6c`), which tightened schema defaults. Before that change, a malformed tool argument would sometimes silently get a default and the call would proceed to fail later in a confusing way — a *permanent* failure dressed as something else. After the change, the same malformed argument fails fast at schema validation, cleanly classifiable as permanent, with a clear "argument X is wrong" message. The classification didn't change; the *clarity* of the classification did, which means the runtime can encode the class confidently instead of guessing.

## Failure modes of the classification system itself

The classifier is code, and code has bugs. Three failure modes to watch for:

1. **Over-aggressive transient.** Every error gets marked transient because "retries are cheap." They aren't — they cost tokens, wall-clock, and rate-limit budget. Audit your transient bucket; if more than half of "transient" errors fail again on retry, your bucket is too wide.
2. **Over-aggressive ambiguous.** Marking ambiguous is the safest classification (the model will check before retrying), so it's tempting to default everything you're unsure about to ambiguous. The cost is the check call, which is cheap individually but adds up. Be explicit about what is ambiguous and route the rest into transient or permanent based on real signal.
3. **Stale classifications.** A specific error pattern was classified once when the tool was added and never revisited. Six months later the upstream behavior changed and your classification is wrong. The fix is the audit-log review above; build the muscle of looking at the classifier as part of your weekly tool-maintenance pass, the same way you'd review a feature flag.

## A minimal classifier

Here's the rough shape, in pseudocode:

```python
def classify(exception, tool_name, args):
    # 1. Tool-specific classifier first (most accurate).
    fn = TOOL_CLASSIFIERS.get(tool_name)
    if fn:
        result = fn(exception, args)
        if result is not None:
            return result

    # 2. Generic by exception type.
    if isinstance(exception, (TimeoutError, ConnectionResetError)):
        # Was the request sent? If we don't know, ambiguous.
        if request_was_sent(exception):
            return Ambiguous(reason="connection died after send")
        return Transient(retry_after=backoff())

    if isinstance(exception, (FileNotFoundError, PermissionError)):
        return Permanent(reason=str(exception))

    if isinstance(exception, HTTPError):
        if exception.status in (429, 502, 503, 504):
            return Transient(retry_after=parse_retry_after(exception))
        if exception.status in (400, 401, 403, 404, 422):
            return Permanent(reason=f"HTTP {exception.status}: {exception.body}")
        if 500 <= exception.status < 600:
            return Ambiguous(reason=f"5xx after request: {exception.status}")

    # 3. Default: ambiguous, with a loud log line so we notice.
    log.warning(f"unclassified error from {tool_name}: {exception!r}")
    return Ambiguous(reason="unclassified, manual review needed")
```

Note the default. Unclassified errors go to ambiguous, not transient — because the cost of a wrongly-retried side effect is much higher than the cost of one extra check call. And the warning log is the audit signal: if you see the same `unclassified` line repeatedly, that pattern needs a real classifier.

## Closing

Tool-call error handling is a place where a small amount of structural work — three classes, a canonical wire format, an explicit prompt for each class — produces an outsized improvement in agent behavior. The model is good at making decisions when it has clean inputs. Error strings are not clean inputs; they're freeform prose that varies by tool, by language, by upstream version, and by the phase of the moon. A class label is a clean input. Add the class, write the prompt, audit the classifier weekly, and you'll be surprised how many "the agent did something insane" reports turn out to have been "the agent retried an ambiguous side effect" or "the agent gave up on a transient timeout" — both fixable not by changing the model but by changing what the model was told.
