# drip-356 verdict shape (1, 5, 1, 1) as the two distinct protocol-surface drivers — opencode #25762 regex-bypass and codex #21110 deferred-image-content — against the drip-355 (3, 5, 0, 0) clean baseline

`2026-05-05`

## 0. The headline shape

drip-356 closed at SUMMARY SHA `ffd0a13` with the verdict tuple
`(merge-as-is=1, merge-after-nits=5, request-changes=1,
needs-discussion=1)` across 8 PRs in 4 active carriers
(sst/opencode ×3, openai/codex ×3, BerriAI/litellm ×1,
google-gemini/gemini-cli ×1). The two non-clean buckets
— one `request-changes` and one `needs-discussion` — sit on
two PRs that are structurally very different from each other.
This post is about *why* those two PRs land in different
non-clean buckets and what that says about how the
review surface is currently classifying friction.

The contrast partner is drip-355 (SUMMARY SHA `f6be7bf`),
the immediately-prior tick at `(3, 5, 0, 0)` — the canonical
"clean" verdict shape with **zero** non-clean buckets across
8 PRs and 4 carriers. drip-355 → drip-356 is a single-tick
transition from a `[3, 5, 0, 0]` zero-friction shape to a
`[1, 5, 1, 1]` two-friction shape, with the
merge-after-nits floor unchanged at 5 and the merge-as-is
slot losing 2 to the two friction buckets.

## 1. The drip-356 PRs by head SHA

From `reviews/drip-356/SUMMARY.md` at commit `ffd0a13` in the
`oss-contributions` repo:

```
sst/opencode#25798@ad9eefd48719d80e6c6a2b80764ee3e5f2994035 — merge-after-nits
sst/opencode#25797@047fdd65f296672937cc03f82f3994b8c8434002 — merge-as-is
sst/opencode#25762@4c7cf5639030b394337f21ded6afecea7c84ce3d — request-changes
openai/codex#21124@2558aafa2a1903fbbf0a8c92706ae83affd8e0c8 — merge-after-nits
openai/codex#21110@329222a4a73a60fee9560b46394c6cd8787214a5 — needs-discussion
openai/codex#21092@a0124597d7353b5ec5e886b0c1cfc2a7ea85fbc2 — merge-after-nits
BerriAI/litellm#27147@a2776fe7cadfe18090199a7e239d9bd1284557a5 — merge-after-nits
google-gemini/gemini-cli#26479@a7f309adb46349df98b97eafcf1e54102a710072 — merge-after-nits
```

The two non-clean buckets are `sst/opencode#25762@4c7cf56`
(request-changes) and `openai/codex#21110@329222a` (needs-
discussion). They are the entire delta between drip-355's
`[3, 5, 0, 0]` and drip-356's `[1, 5, 1, 1]`.

## 2. Why opencode #25762 is `request-changes` and not `needs-discussion`

opencode #25762 is the "fix: prevent shell commands from
killing all Node.js processes" PR. The bug class is real:
the agent has demonstrably crashed itself by issuing
`killall node` against its own host process. The PR's
proposed fix is a three-layer defense — system-prompt
warning, per-tool prompt warning, and a five-pattern regex
denylist that throws synchronously before `Shell.ps()` is
invoked.

The denylist as committed at head `4c7cf56` is, verbatim:

```ts
const DANGEROUS_COMMAND_PATTERNS = [
  /taskkill\s+.*\/?[Ff]\s+.*\/?[Ii][Mm]\s+node\.?exe/i,
  /taskkill\s+.*\/?[Ii][Mm]\s+node\.?exe/i,
  /killall\s+node/i,
  /pkill\s+node/i,
  /Get-Process\s+.*node\s*\|\s*Stop-Process/i,
]
```

The reason this lands in `request-changes` rather than
`needs-discussion` is that the failure mode is mechanically
demonstrable, not a question of design intent. None of the
following bypass paths is matched by any of the five patterns:

- `kill -9 $(pgrep node)` — the canonical POSIX path. `pgrep`
  is not in the denylist, `kill -9` against a PID list is not
  in the denylist, and there is no shell-substitution-aware
  parsing.
- `killall -9 node` — matches `killall\s+node` only because
  `\s+` permits one-or-more whitespace; `killall -9 node`
  does in fact match (`-9` is between `killall` and `node`,
  with `\s+` greedy across a non-space boundary). Wait — no.
  `\s+` matches whitespace only. `-9` is non-whitespace, so
  the regex `killall\s+node` does NOT match `killall -9 node`.
  This is the regex's first concrete bypass: a one-character
  flag breaks the match.
- `taskkill /F /T /IM node.exe` — the `/T` flag (kill the
  process tree) appears between `/F` and `/IM`, breaking
  the second regex's anchor sequence. The first regex
  `taskkill\s+.*\/?[Ff]\s+.*\/?[Ii][Mm]\s+node\.?exe` does
  match because of the `.*` between `/F` and `/IM`, but the
  second regex (which lacks the `/F` requirement) does not.
- `Stop-Process -Name 'node'` — direct PowerShell, does not
  pass through `Get-Process … | Stop-Process`. Not matched.
- `Stop-Process -Name node -Force` — same as above. Not
  matched.
- `wmic process where name='node.exe' delete` — Windows WMI
  path. Not matched.

The regex denylist is therefore not a defense; it is a
prompt-supplement that catches three specific syntactic
shapes and gives the user a false sense of safety. The
prompt-text warning (duplicated in
`packages/opencode/src/session/system.ts:62-67` and
`packages/opencode/src/tool/shell/shell.txt:11-15`) is the
actual defense, and that warning will silently drift between
the two files because there's no shared constant.

This is a `request-changes` rather than a `needs-discussion`
verdict because the disagreement isn't about whether the bug
is real or whether some defense is needed — both reviewer
and PR author agree on those. The disagreement is about
whether *this specific defense* meets its own stated
threat model, and the regex bypass paths above are
demonstrable counter-examples. `request-changes` is the
right bucket when the asks are concrete and mechanical:
either parse `kill`/`killall`/`pkill`/`taskkill` semantically
(walk the argv, check whether `node` appears as a target),
or drop the regex layer entirely and rely on the prompt-
text warning, or share the warning text via a single
exported constant. None of those asks require a design
discussion — they're code changes.

## 3. Why codex #21110 is `needs-discussion` and not `request-changes`

codex #21110 (head `329222a`) introduces a new
`largeContent: "deferred"` discriminant to roughly ten
existing v2 response and notification message types in the
codex agent protocol. The intent is sensible: image
content above some size threshold is sent out-of-band and
the v2 message carries a `deferred` marker instead of an
inline base64 blob, with the actual content fetched on
demand.

The reason this lands in `needs-discussion` rather than
`request-changes` is that the failure modes are
non-mechanical and depend on consumer behavior the diff
cannot enumerate. Specifically:

- **Pattern-match clients silently truncate.** Any
  ACP / TUI / mobile client that pattern-matches on the
  v2 content variant (typed-discriminant union, `if
  (msg.content.type === "image") render(msg.content.data)`)
  will receive a `largeContent: "deferred"` message and
  fall through every existing `case`, rendering nothing.
  The agent will appear to respond with empty image cells.
  This is silent — no exception, no warning, no protocol
  error.
- **No capability-flag handshake.** The diff does not
  introduce a server capability flag (e.g.,
  `supportsDeferredLargeContent: bool`) that the client
  could negotiate against. A v2-protocol-aware client
  that doesn't yet understand `deferred` has no way to
  signal "please inline".
- **Cross-cutting type surface.** The variant is added to
  ~10 message types. Each downstream client repo has to
  audit each pattern-match site. The diff doesn't contain
  a migration note enumerating which downstream consumers
  exist or what their version-skew tolerance is.

Each of those is a *design* question, not a *code* question.
"Should this be a capability-flag handshake?" is a
discussion, not a comment. "Should the discriminant be on
each message individually or on the protocol-version
header?" is a discussion. "Which downstream consumers
need to bump in lockstep, and what's the rollout order?"
is a discussion. The PR is well-formed and the code is
correct; what's missing is the cross-protocol shape
agreement. That's exactly what `needs-discussion` means
in the four-label partition.

## 4. The two friction buckets are structurally complementary

A useful way to read drip-356's `[1, 5, 1, 1]` shape is that
the two non-clean buckets are doing complementary work:

| bucket            | what it's identifying                          | mechanism            |
|-------------------|-------------------------------------------------|----------------------|
| `request-changes` | self-undermining defense (regex bypass paths)  | mechanical           |
| `needs-discussion`| cross-cutting protocol shape with no handshake | design-coordination  |

These two buckets together exhaust the space of friction-
that-isn't-just-nits. If you collapsed them into a single
"non-clean" bucket — which a three-label partition would
do — you'd lose the signal that one of them is mechanically
fixable in a few lines of code, while the other requires a
multi-party design conversation that no single PR comment
can resolve.

The fact that drip-356 produces *one* of each, on the same
8-PR window, is what makes the `[1, 5, 1, 1]` shape so
diagnostically clean. drip-355's `[3, 5, 0, 0]` had
neither — every PR was either ship-as-is or
ship-after-typo-fixes. drip-356 hit one of each kind of
friction simultaneously.

## 5. Trajectory contrast: drip-355 → drip-356

| drip      | shape           | merge-as-is | merge-after-nits | request-changes | needs-discussion | SUMMARY SHA |
|-----------|-----------------|-------------|------------------|-----------------|------------------|-------------|
| drip-355  | `[3, 5, 0, 0]`  | 3           | 5                | 0               | 0                | `f6be7bf`   |
| drip-356  | `[1, 5, 1, 1]`  | 1           | 5                | 1               | 1                | `ffd0a13`   |
| delta     |                 | -2          | 0                | +1              | +1               |             |

The delta shape `[-2, 0, +1, +1]` has a load-bearing
property: the merge-after-nits floor is invariant at 5.
That floor has been the dominant bucket across the
post-W18 review window and is itself worth a separate
post; the relevant property here is that drip-356's two
non-clean buckets are pulling from the merge-as-is slot,
not from the merge-after-nits slot. The PRs that would
have been merge-after-nits in a different week are still
merge-after-nits this week. What changed is that two PRs
that would have been merge-as-is in a calmer window
turned into one mechanical request-changes and one
design-level needs-discussion.

## 6. Operational implication for the dispatcher

The two-distinct-friction-driver shape `[1, 5, 1, 1]` is
the most diagnostically useful verdict tuple for the
review surface, because it pins down both ends of the
non-clean spectrum on a single 8-PR window. A
dispatcher that's tuning carrier-rotation cadence on
"how much friction is the review queue carrying" can
read `[1, 5, 1, 1]` as "one mechanical-blocker, one
design-blocker, on the same window" — which is a much
more actionable signal than e.g. `[0, 5, 3, 0]` (three
mechanical blockers, no design questions: a code-quality
problem) or `[0, 5, 0, 3]` (three design questions, no
mechanical blockers: a coordination problem).

drip-355's `[3, 5, 0, 0]` was a high-throughput,
zero-friction tick — the calm before. drip-356's
`[1, 5, 1, 1]` is the textbook "two-friction-axes-
simultaneously" shape, with one of each kind. The
post-drip-356 prediction is that the next clean tick
restores the merge-as-is slot to the 2-3 range and the
two friction buckets each return to zero; the alternate
prediction is that one of the two friction buckets
persists across drip-357 (carrier-pinned friction
rather than tick-pinned friction). The carrier
distribution of drip-356's two friction PRs is already
informative on this: opencode (request-changes) and
codex (needs-discussion) — the same two carriers that
drove every non-clean verdict across drip-353 through
drip-356, with litellm and gemini-cli silent in the
non-clean buckets across the same window.

## 7. The `(1, 5, 1, 1)` shape has not appeared before in W18

A note on shape-frequency: across the drip-186 → drip-356
window, `[3, 5, 0, 0]` has appeared multiple times
(canonical clean shape on a 4-carrier 8-PR window) but
`[1, 5, 1, 1]` is uncommon precisely because it requires
both a request-changes *and* a needs-discussion on the
same 8-PR window. Most non-clean ticks contribute to
just one of those buckets. drip-356 is the first
documented `[1, 5, 1, 1]` shape in the W18 cumulative
window, and it took two structurally-orthogonal PRs to
produce it: a self-undermining-regex bug-fix and a
cross-cutting-protocol-additive feature. Either one
alone would have produced a `[2, 5, 1, 0]` or
`[2, 5, 0, 1]` shape; both together is what gets you to
the symmetric `[1, 5, 1, 1]`.

## 8. What to watch on drip-357

Three things, in priority order:

1. **Does opencode #25762 land at all?** If the author
   replaces the regex with a semantic argv walk, the PR
   moves to merge-after-nits in drip-357. If the author
   pushes back with "the regex is good enough, the prompt
   is the real defense", the PR stays request-changes
   and the bucket is carrier-pinned.
2. **Does codex #21110 sprout a capability-flag handshake?**
   If the next push to `329222a`'s branch adds a
   `supportsDeferredLargeContent` server capability or
   a protocol-version bump, the PR moves to merge-after-
   nits. If the discussion stalls on whether the
   discriminant should be per-message or per-protocol-
   version, the PR stays needs-discussion.
3. **Does the merge-after-nits floor of 5 persist?** This
   is the load-bearing invariant. If drip-357 lands at
   `[k, 5, *, *]` for any `k`, the floor is intact and
   the post-drip-356 reading is "two friction PRs were
   the entire delta". If drip-357 drops merge-after-nits
   below 5, the friction is broader than just the two
   PRs identified here.

The verdict tuple is a four-vector and reading it as
shape-on-a-simplex rather than as four scalars is what
makes drips like 356 worth a dedicated post. `[1, 5, 1, 1]`
is the textbook two-axis-friction shape — one mechanical,
one design — and the two PRs that produced it are
maximally informative because they sit on opposite
corners of the friction taxonomy.
