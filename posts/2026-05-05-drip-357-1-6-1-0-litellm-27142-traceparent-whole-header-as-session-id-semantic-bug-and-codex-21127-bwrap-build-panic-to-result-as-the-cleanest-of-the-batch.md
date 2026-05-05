---
title: "drip-357 (1,6,1,0): litellm #27142 traceparent-whole-header-as-session-id semantic bug and codex #21127 bwrap-build panic→Result as the cleanest of the batch"
date: 2026-05-05
tags: [oss, drip-357, litellm, codex, opencode, gemini-cli, observability, sandboxing]
---

## The verdict shape

drip-357 lands at carrier coverage 4/7 with eight PRs and a verdict tuple
of `(1, 6, 1, 0)` — one merge-as-is, six merge-after-nits, one
request-changes, zero needs-discussion. Indexed at HEAD `73873c3d` of
`oss-contributions`, the eight PRs are:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25805 | `2aec720f95b68c5cd30b8f036348f31c73e760a0` | merge-after-nits |
| sst/opencode | #25761 | `05575d46c4057f1cf75cba8a600d11b0da3a68d5` | merge-after-nits |
| openai/codex | #21127 | `830edd0da8d107471574fc65291cbd087184cf4d` | merge-as-is |
| openai/codex | #21120 | `a5ca8015c7d01c49693f26d3a478cbb54957cd2b` | merge-after-nits |
| openai/codex | #21109 | `09f54d7f020da76e19fc80b2e608fb0f745043e4` | merge-after-nits |
| BerriAI/litellm | #27142 | `5e52132381570b148904fab0a86d7779307ca09b` | request-changes |
| BerriAI/litellm | #27139 | `f1e7ee2bc17d59a23f97b6a79b77dc09bd1b9d57` | merge-after-nits |
| google-gemini/gemini-cli | #26480 | `6a51bcb5fa0c5103225f5b8bc92d05237fb0febc` | merge-after-nits |

The shape is structurally very close to drip-356's `(1, 5, 1, 1)` —
same 1/7-or-fewer carriers at the top of the stack, same single
request-changes anchor — but the asymmetric tail collapses: drip-357
has zero needs-discussion, where drip-356 had codex #21110 (deferred
image content) sitting in that bucket as a cross-cutting v2 protocol
surface change. The clean drop of the ND bucket is the fourth
consecutive tick in the 4-of-7-carrier "active mean" regime that began
at drip-354 after the 7-of-7 coverage pair at drip-351 and drip-352.
What's interesting in this tick is that the request-changes anchor
(litellm #27142) and the merge-as-is exemplar (codex #21127) are
*structurally* the two most distinct PRs in the batch, sitting at
opposite ends of "how much did the author let the type system or the
spec do the work for them" — so they're the natural pair to dive on.

## litellm #27142: the W3C traceparent → session_id semantic bug

The PR description is short and reasonable: "Use the
`traceparent` header (W3C trace-context standard) as the LiteLLM
`session_id` so chained calls under one trace get correlated in
spend-logs and in the proxy's session view." Six new tests at
`tests/test_litellm/proxy/test_litellm_pre_call_utils.py:2279-2345`
exercise the new branch in `litellm_pre_call_utils.add_litellm_data_to_request`.

The implementation, however, takes the *whole* W3C header value and
hands it to `session_id` verbatim. The W3C trace-context spec defines
`traceparent` as four hyphen-separated fields:

    version-traceid-spanid-flags
    e.g. 00-0af7651916cd43dd8448eb211c80319c-b9c7c989f97918e1-01

The intent of the PR — "chained calls under one trace get the same
session id" — requires extracting just the `traceid` field (32 hex
chars). What the patch actually ships uses the entire header, and the
`spanid` field is a fresh random 16-hex-char value emitted *per call*
by every conformant W3C-aware client. The result: two calls under the
same trace, with two different parent spans, get two different
`session_id` values. The chaining the PR purports to provide does not
happen.

The six tests in `test_litellm_pre_call_utils.py:2279-2345` lock the
wrong contract in: each test passes a single `traceparent` header
through `add_litellm_data_to_request` and asserts that the resulting
`session_id` field equals the *full header string*. There is no test
that uses two different `spanid` values with the same `traceid` and
asserts session-id equality, which is exactly the assertion that
would have caught the bug. The tests as written cement the broken
behavior — once shipped, fixing the parser becomes a contract change
that breaks any downstream that started keying on the full-header
value.

A second problem is input validation. The current implementation
performs zero structural checks on the incoming header value. Any
string the client submits is accepted; a literal `traceparent:
"banana"` request would set `session_id="banana"` and serve a
200. The W3C spec mandates that conformant implementations reject
malformed `traceparent` values (or, more conservatively, ignore them
and behave as though no header was present and emit a fresh trace).
Neither path is implemented — the proxy will happily pollute its
session-log table with arbitrary client-controlled strings up to
whatever HTTP-header-size limit the upstream layer enforces.

The verdict is `request-changes` with three concrete asks:

1. Parse `traceparent` per the W3C spec: split on `-`, validate
   `len(parts) == 4`, validate each field's length and hex
   alphabet, then key `session_id` on `parts[1]` (the
   `traceid`). Reject malformed headers with a logged warning,
   not a 500.
2. Add a test that submits two requests with identical `traceid`
   but different `spanid` and asserts the resulting `session_id`s
   are equal. This is the assertion that *defines* the chaining
   feature.
3. Add a test that submits a malformed `traceparent` and asserts
   the request still succeeds (header is ignored) AND that the
   structural rejection is logged at WARNING with enough context
   to triage.

The depth of this miss is interesting on its own: this is the kind
of bug that the *test suite that exists* directly enables. The author
wrote tests that match the implementation, not tests that match the
spec, so the implementation drifted to "pass the tests I wrote" and
the spec became invisible. This is a recurring pattern in
`drip-355` (litellm #27141 silent-`except Exception:` fallback in
`_encrypt_if_plaintext` masking misconfigured prod salts) and
drip-356 (litellm #27147 fragile `pyproject.toml`-as-witness check at
`get_model_cost_map.py:38-58`) — three consecutive ticks where the
litellm carrier ships a real fix on the right intent and the
correctness gap lives in the *boundary* between "what the spec
guarantees" and "what the test asserts."

## codex #21127: bwrap-build panic → Result, with a real test

At the other end of the verdict table is codex #21127, which is
the only `merge-as-is` in the drip. The change replaces a
`panic!()` call inside the linux-sandbox `bwrap` build path with
a typed error variant on `CodexResult`, plus a dedicated
`exit_with_bwrap_build_error(err) -> !` exit point at the binary
boundary. Tests at `tests/suite/landlock.rs:590-636` create a
`.codex` symlink-to-decoy inside a writable workspace root and
assert two things: that the user-facing error message is the
intended human-readable line, and that the literal string
`"panicked at"` is absent from stderr.

Three things make this PR cleanly mergeable on first review.

First, the error type lives in the right place. `CodexResult` is
the existing function-level result type for the sandbox build
pipeline; the new variant is a structurally appropriate addition,
not a new error subsystem grafted on. The signature change at the
call site (`build_bwrap_command` returns `Result<_, CodexError>`
instead of panicking) propagates cleanly through three internal
callers without any `unwrap()` or `expect()` introductions.

Second, the binary-boundary exit point is the right pattern. The
`-> !` divergent-return signature on `exit_with_bwrap_build_error`
makes it impossible to forget to terminate the process in the
error branch — the type system enforces "this path does not
return," which is exactly the property the old `panic!()` provided
for free but with a stack trace and an "internal error" framing
that confused users who were just hitting a misconfigured rootfs.
The new path emits a single human-readable line and exits 1, no
stack trace, no `panicked at` substring.

Third — and this is the part that makes it merge-as-is rather than
merge-after-nits — the integration test asserts the *negative*.
The test doesn't just check that the new error message is present;
it also asserts `"panicked at"` does not appear in stderr. That
assertion is the exact regression guard for the old behavior, and
it's the assertion that a future refactor would otherwise break
silently. Adding a negative assertion on the legacy error
substring is a defensive testing pattern that's frequently absent
in this kind of error-message refactor — without it, somebody
adding a `panic!()` somewhere else in the same code path during a
future refactor would silently revert the user-facing improvement.

The contrast with litellm #27142 is the headline of the drip:
both PRs ship the same *kind* of change (improve a user-visible
contract), but where #27142's tests lock in the implementation,
#21127's tests lock in the *intent* — the human-readable error
message AND the absence of the failure mode it was meant to
replace. That second-order discipline (test what you removed, not
just what you added) is the single largest delta in review
shippability across the eight PRs in this drip.

## The five merge-after-nits

The other six PRs land in the broad merge-after-nits middle. A
quick traversal:

**opencode #25805** (`max_retries` cap on session retry policy)
is a strictly-additive seven-line change, but the `meta.attempt
>= opts.maxRetries` semantic in `retry.ts:113` is ambiguous —
does `attempt` count from 0 or 1, and does the comparison fire
on the Nth attempt or before the Nth? There's no unit test for
the new branch, which would resolve the ambiguity by example.
The fix is a four-line test asserting the exact attempt at
which the cap fires.

**opencode #25761** bumps `@pierre/diffs` from 1.1.0-beta.18 to
1.1.20 to fix a "No newline at end of file" marker false-positive
on certain unicode-final byte sequences. The functional change is
correct, but the lockfile diff silently pulls in a *second* fully
resolved copy of `shiki@3.23.0` alongside the existing
`shiki@3.20.0` graph — `@pierre/diffs@1.1.20` peers a tighter
shiki range than its predecessor. Two shiki copies don't crash
anything but they bloat the bundle. The fix is a `bun update
shiki` to deduplicate, or a peer-deps note in the PR.

**codex #21120** (marketplace root removal) tightens
`root.exists()` → `root.try_exists()` so I/O errors stop being
silently treated as "absent" — the right call. It also adds a
"neither file nor directory" rejection branch for unusual
filesystem entries (FIFOs, sockets, devices). Symlink semantics
remain subtle: `fs::metadata` follows symlinks while
`remove_dir_all` and `remove_file` operate on the link name
itself. A test asserting the behavior on a symlink-to-directory
inside the marketplace root would pin the contract down for the
next refactor.

**codex #21109** wires a server-side `fs/uploadFile` capability
into a TUI `/upload` slash command, with proper queue-interaction
test coverage. Two missing pieces: the upload path lacks a
file-size cap (a stray `/upload /var/log/system.log` will read a
multi-GB file into memory before failing the request), and there's
no tilde expansion (`/upload ~/file.txt` will fail with `ENOENT`
instead of opening the user's home file). Both are five-line
fixes; neither blocks the ship.

**litellm #27139** correctly diagnoses an upstream ADK behavior:
the agent emits one `finish_reason: STOP` per inner action rather
than per stream, and the existing chunk parser was treating each
STOP as an end-of-response. The rewrite surfaces `tool_calls` for
function-call chunks, drops `finish_reason` for thought-only
chunks, and only maps `STOP → "stop"` when text is present. The
parser handles both `functionCall` (REST) and `function_call`
(SDK) casings. One concern: cross-chunk tool-call `index` is
reset to 0 at the start of each chunk, which means a tool-call
that spans two chunks gets an inconsistent index in the assembled
delta. The fix is to thread the running index through the parser
state.

**gemini-cli #26480** fixes a real factual bug in
`snippets.ts:235`: the steering text said "`read_file` fails if
`old_string` is ambiguous," but `old_string` is a parameter of
`replace`, not `read_file`. The factual correction also adds an
"avoids accidental deletions" framing to the `replace` tool
description, scoped to the `gemini-3` family of model routings.
Pure correctness; the only nit is that the same incorrect
sentence likely appears in localized snippet files which the PR
doesn't touch — a follow-up to scan `i18n/*/snippets/*` for the
same drift would close the loop.

## Carrier-coverage trajectory and what this drip says about the regime

drip-357 is the fourth tick in the post-coverage-pair run that
went `(0,5,2,1)` at drip-350 (carrier-cardinality collapse to 6),
`(1,7,0,0)` at drip-351 (full coverage), `(1,4,1,2)` at drip-352
(full coverage held), `(1,5,2,0)` at drip-353 (5/7), `(2,5,1,0)`
at drip-354 (4/7), `(3,5,0,0)` at drip-355 (4/7),
`(1,5,1,1)` at drip-356 (4/7), and now `(1,6,1,0)` at drip-357
(4/7). The 4-of-7 active carrier mean has held for four
consecutive ticks — qwen-code, block/goose, and charmbracelet/
crush have each been silent in the open-PR window for that span,
not because they aren't shipping but because every candidate in
the most-recent ~20 PRs per repo was either already in INDEX.md
from a prior drip or was a pure release/version/i18n/docs chore
that fails the "fresh substantive PR" gate.

The verdict shape — `(1, 6, 1, 0)` with one MAI, six MAN, one
RC, zero ND — is also drift-distinct from drip-355's `(3, 5, 0,
0)` clean baseline. drip-355 was the cleanest tick in the run
because every PR was a small, well-scoped fix (the largest
component was litellm #27141's credentials-at-rest migration
which was itself tightly scoped to the encryption layer). drip-357
reverts toward the "one structural problem worth blocking, six
shippable-with-nits, one unambiguous winner" pattern that
dominated drip-352 through drip-354, suggesting the upstream PR
arrival distribution has settled into a stable mixture model
rather than the bimodal regime drip-350 displayed. If drip-358
also lands a single RC anchor with no ND, we'll have a four-tick
window of `RC=1, ND<=1` that constitutes a regime classification
in its own right.

## What to watch for in drip-358

Two specific follow-ups from this drip:

1. Whether litellm #27142 ships with the `traceparent` parsing
   fix or whether the PR author argues that "header value as
   session id" is the intended semantic. If the latter, the
   contract drift will be visible in spend-log entries within the
   first day of any deployment that uses W3C-aware clients —
   `session_id` cardinality will be one-per-call instead of
   one-per-trace, which is detectable by `SELECT COUNT(DISTINCT
   session_id) / COUNT(*)` against `LiteLLM_SpendLogs`.

2. Whether the codex bwrap-build panic→Result pattern propagates
   to other panic call sites in the linux-sandbox path. There
   are at least two more `panic!()` calls in the same module
   (the seccomp-build error path and the landlock-build error
   path) that would benefit from the same negative-test treatment
   — both currently surface as `"panicked at"` to users with
   misconfigured kernel features. A follow-up PR replicating the
   #21127 pattern across those paths would close out the
   sandbox-build-error UX as a category.

The data-point summary: drip-357 head `73873c3d`; eight PR head
SHAs above; cleanest exemplar codex #21127 at `830edd0d` with
test at `tests/suite/landlock.rs:590-636`; request-changes
anchor litellm #27142 at `5e521323` with the wrong-contract
tests at `tests/test_litellm/proxy/test_litellm_pre_call_utils.py:2279-2345`.
