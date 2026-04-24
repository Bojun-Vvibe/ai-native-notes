---
title: "The empty-tool-result trap: how silent successes corrupt agent reasoning"
date: 2026-04-24
tags: [agents, tool-use, reliability, failure-modes]
---

There is a class of agent failure that doesn't look like a failure. The
tool returned. The status was `ok`. The model received the result. The
loop continued. And somewhere ten turns later, the agent confidently does
the wrong thing — deletes the wrong file, claims a test passed when it
never ran, summarises a directory it never read. Tracing back, the
proximate cause is always the same: a tool returned an empty result, and
the agent treated empty as evidence rather than as absence.

Call this the empty-tool-result trap. It is more common than crashes,
harder to reproduce, and impossible to alert on with the usual metrics.
It is also fixable, with discipline at exactly two layers: the tool
boundary and the prompt that consumes the result.

## The shape of the bug

Consider a tool that searches a codebase for a pattern:

```python
def grep(pattern: str, path: str) -> dict:
    matches = run_ripgrep(pattern, path)
    return {"matches": matches}
```

If `path` is a typo and resolves to a non-existent directory, ripgrep
exits with code 2. Depending on how the wrapper handles that, you might
return:

- `{"matches": []}` — empty list, status implicitly ok.
- `{"matches": [], "error": null}` — the same but with structure.
- `{"matches": []}` and a log line nobody reads.

The model receives `matches: []` and concludes "no occurrences exist." It
proceeds to refactor on the assumption that the pattern is unused.
Whatever happens next is, technically, the agent's fault — but the agent
was lied to.

Empty results that look identical regardless of cause are the trap. The
model cannot distinguish:

1. The tool ran and genuinely found nothing.
2. The tool ran against the wrong target and found nothing because it was
   looking in the wrong place.
3. The tool failed silently and returned its default empty value.
4. The tool was rate-limited / circuit-broken and returned a stub.
5. The tool succeeded but truncated to zero items because of a paging bug.

All five render as `[]`. Only one of them is "no matches exist."

## Why this corrupts reasoning specifically

A normal error — `{"error": "ENOENT: /srv/code/billling"}` — is a
reasoning input. The model reads it, notes the typo, retries with
`/srv/code/billing`. The loop self-corrects.

An empty success is an *epistemic* input: it updates the model's
posterior about the world. "There are no matches for `parse_invoice` in
this repo" is a load-bearing fact. Subsequent turns plan around it. By
turn five, the model has produced a refactor that removes a function it
believes is unused, and the chain of reasoning is internally consistent
— it is just rooted in a lie.

The harm scales with how downstream the bad belief travels. A crash at
turn one wastes a turn. An empty-result lie at turn one can poison every
subsequent turn for the rest of the run.

## The minimum-viable fix at the tool boundary

Every tool result needs three fields, not one:

```python
@dataclass
class ToolResult:
    status: Literal["ok", "empty", "partial", "error"]
    payload: Any                # the actual data, possibly empty
    provenance: dict            # what was actually run, against what
```

The key move is treating `empty` as a first-class status, separate from
`ok`. An empty list is data. An empty result is an outcome. They are not
the same.

```python
def grep(pattern: str, path: str) -> ToolResult:
    resolved = os.path.realpath(path)
    if not os.path.exists(resolved):
        return ToolResult(
            status="error",
            payload=None,
            provenance={"pattern": pattern, "path_requested": path,
                        "path_resolved": resolved, "exists": False},
        )
    matches = run_ripgrep(pattern, resolved)
    if not matches:
        return ToolResult(
            status="empty",
            payload=[],
            provenance={"pattern": pattern, "path_resolved": resolved,
                        "files_scanned": count_files(resolved),
                        "bytes_scanned": bytes_under(resolved)},
        )
    return ToolResult(
        status="ok",
        payload=matches,
        provenance={"pattern": pattern, "path_resolved": resolved,
                    "files_scanned": count_files(resolved),
                    "bytes_scanned": bytes_under(resolved),
                    "match_count": len(matches)},
    )
```

The provenance block is doing the heavy lifting. "I scanned 1,847 files
totalling 23 MB under `/srv/code/billing` and found zero matches for
`parse_invoice`" is a believable empty. "I returned `[]`" is not.

## The pattern in five common tools

The same shape recurs across almost every tool you'll write.

**read_file** — empty when the file is zero bytes vs. when the file
doesn't exist vs. when the path is a directory vs. when read permission
is denied. All four can produce an empty string downstream. The fix:

```
status=ok    payload=""   provenance={size: 0, type: regular_file}
status=error payload=None provenance={errno: ENOENT}
status=error payload=None provenance={errno: EISDIR}
status=error payload=None provenance={errno: EACCES}
```

**list_directory** — empty because the directory has no entries vs.
empty because the listing was filtered to nothing vs. empty because a
permission error swallowed everything. The provenance must include
`unfiltered_count` so the model can tell the difference.

**http_fetch** — empty because the body was empty vs. empty because the
response was 204 No Content vs. empty because a 200 returned an empty
JSON array vs. empty because the response was truncated mid-stream.
Provenance: status code, content-length, bytes-received, truncated flag.

**database_query** — empty result set vs. empty because the WHERE clause
silently matched a column that doesn't exist (depending on dialect) vs.
empty because the prepared statement bound a NULL where the model meant
a value. Provenance: row count, query plan summary, bound parameters.

**shell** — empty stdout with exit 0 vs. empty stdout with exit non-zero
vs. empty stdout because output went to stderr vs. empty because the
shell substituted a missing variable to nothing. Provenance: exit code,
stderr length, command as actually executed after expansion.

In each case the rule is the same: an empty payload must be paired with
enough provenance to falsify the wrong inference.

## Prompt-side discipline

Boundary fixes alone aren't enough. The model must be taught to read the
status field, not just the payload. The system prompt for tool-using
agents needs an explicit clause:

```
When a tool returns status=empty, this is NOT evidence of absence in the
underlying world. It is evidence that the tool ran and found nothing
under the specific provenance reported. Before drawing a conclusion of
"X does not exist", you must:
1. Read the provenance and confirm the tool actually examined the right
   target (correct path, correct query, correct parameters).
2. If provenance suggests a narrow or surprising scope, broaden the
   query and re-run.
3. Only after at least one broadened re-run with status=empty may you
   record "no occurrences" as a fact.
```

This sounds heavy. In practice it adds about one tool call per genuinely
empty result, and it eliminates the worst class of confident-and-wrong
behaviour.

## A worked example

The agent is asked: "Is `parse_invoice` still called anywhere?"

Bad version (no provenance):

```
> grep("parse_invoice", "/srv/code/billling")
< {"matches": []}
agent: "parse_invoice is unused. I will remove it."
```

Good version:

```
> grep("parse_invoice", "/srv/code/billling")
< {"status": "error", "payload": null,
    "provenance": {"path_requested": "/srv/code/billling",
                   "path_resolved": "/srv/code/billling",
                   "exists": false}}
agent: "Path doesn't exist; likely a typo for /srv/code/billing. Retrying."

> grep("parse_invoice", "/srv/code/billing")
< {"status": "empty", "payload": [],
    "provenance": {"files_scanned": 1847, "bytes_scanned": 23_412_991,
                   "pattern": "parse_invoice"}}
agent: "Empty against the right path. But this only checked the billing
        subtree. Broadening to /srv/code."

> grep("parse_invoice", "/srv/code")
< {"status": "ok", "payload": [{"file": "/srv/code/legacy/etl.py", ...}],
    "provenance": {"files_scanned": 12_440, "bytes_scanned": 184_002_133,
                   "match_count": 3}}
agent: "Three call sites in legacy/etl.py. Not unused; will not remove."
```

The same model, the same tools, the same task. The only difference is
that the boundary refused to lie and the prompt refused to be credulous.

## Detecting the trap in production

You can audit your existing logs for the empty-result trap without
changing any code, by counting:

- For each tool, the rate of empty results.
- The rate of subsequent turns that contain the word "no", "none",
  "doesn't exist", "is unused", or similar negative absolute claims,
  within three turns of an empty result.
- The rate of subsequent destructive actions (write, delete, refactor)
  within five turns of an empty result.

Healthy ratios look like: empty results 5-15% of calls, negative-claim
follow-ups 30-50% of empties (most are correctly inferred), destructive
follow-ups under 5%. If your destructive-after-empty rate is higher than
that, you almost certainly have the trap, and you can find specific
incidents by sampling.

## Why "just check exit codes" isn't enough

Engineers reading this for the first time often respond: "the tool
should just return an error when the path doesn't exist." Yes — but the
trap also fires when:

- The path exists and is empty.
- The query is syntactically valid but semantically wrong.
- The credentials are valid but scoped to nothing.
- The cache is stale and returns the empty pre-warm value.
- A feature flag has hidden the data the tool would have returned.

None of these are exit-code failures. All of them produce honest
`status=ok, payload=empty` results that look identical to genuine
emptiness without provenance. The exit code is necessary; it is not
sufficient.

## A small taxonomy of empties

It helps to name the cases out loud. In rough order of frequency:

1. **Genuine empty.** The world really has nothing matching. Provenance
   confirms the tool looked in the right place.
2. **Misdirected empty.** The tool ran against the wrong target due to a
   typo, stale id, wrong env, etc. Provenance reveals the resolved
   target differs from intent.
3. **Truncated empty.** Paging or row limits clipped the result to
   nothing. Provenance includes a `truncated` or `page` field.
4. **Filtered empty.** A post-processing filter removed every row.
   Provenance includes both the unfiltered and filtered counts.
5. **Permission empty.** ACLs hid the data without raising an error.
   Provenance includes scopes/principal used.
6. **Cache-stub empty.** A circuit breaker or cold cache returned the
   defined-empty stub. Provenance includes `source: cache_stub` or
   `source: breaker_open`.
7. **Race empty.** Data existed but was concurrently deleted. Provenance
   includes a snapshot timestamp.

Each of these wants a different remediation. Conflating them under a
single `[]` makes every remediation impossible.

## What this costs you to fix

The tool-boundary refactor is mechanical: introduce `ToolResult`, route
every tool return through it, and add provenance fields one tool at a
time. Most teams can do their five most-used tools in an afternoon, and
those five tools account for the bulk of incidents. The prompt-side
clause is two paragraphs. The auditing query is fifty lines of SQL or a
log scan.

In exchange, you stop shipping confident-and-wrong agents. The class of
incident where the postmortem reads "the agent thought X didn't exist
because the tool said `[]`" disappears. Other incidents will replace it
— this is software — but they will be incidents you can debug, because
the failure will be in the reasoning rather than in the silent absence
of evidence.

## The single line worth remembering

Empty is data. Absence is an outcome. Tools must report which one they
mean. Agents must read the report.
