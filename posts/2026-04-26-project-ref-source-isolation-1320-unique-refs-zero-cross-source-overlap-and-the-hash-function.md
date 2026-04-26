# Project-ref source isolation: 1320 unique refs, zero cross-source overlap, and what that says about the hash function

**Date:** 2026-04-26
**Data captured:** 2026-04-26T04:14:06Z
**Tags:** project-ref, source-isolation, schema-design, hashing, jsonl, jq, session-aggregate

## The finding

Across all 8,561 session rows in `~/.config/pew/session-queue.jsonl`, there are **1,320 unique non-null `project_ref` values**. Every single one of those 1,320 refs belongs to **exactly one** `source`. Not one ref is shared across sources. The `(project_ref, source)` pair is, in this corpus, a redundant key: the `source` adds no information given the `project_ref`. This is either a deeply uninteresting fact about a hash function, or a deeply interesting fact about how cross-tool work actually flows on this device. Probably both.

## Methodology

The full pipeline that produces the headline number, run verbatim against the live file:

```bash
jq -r 'select(.project_ref != null) | [.project_ref, .source] | @tsv' \
  ~/.config/pew/session-queue.jsonl \
  | sort -u \
  | awk -F'\t' '{c[$1]++} END {
      for (k in c) hist[c[k]]++
      for (n in hist) print n, hist[n]
    }'
```

Verbatim output:

```
1 1320
```

Read: there are 1,320 distinct `project_ref` values for which the multiplicity-of-source-membership is exactly 1. There are zero refs with multiplicity 2, 3, or 4. Maximum multiplicity in the file is 1.

The per-source decomposition (separate `jq` query, verbatim numbers):

```json
[
  { "source": "claude-code", "refs": 176 },
  { "source": "codex",       "refs": 296 },
  { "source": "openclaw",    "refs":   2 },
  { "source": "opencode",    "refs": 846 }
]
```

Sums to 1,320, which is the same as the union count, which confirms the disjointness from a second angle: union size equals sum of part sizes if and only if the parts are pairwise disjoint. They are.

The session row counts per source, for context:

```
1108 claude-code
 478 codex
2024 openclaw
4951 opencode
```

So `opencode` produces 4951 sessions across 846 projects (5.85 sessions/project on average), `codex` produces 478 sessions across 296 projects (1.61), `claude-code` 1108 across 176 (6.30), and `openclaw` 2024 sessions across only 2 projects (a remarkable 1012 sessions per project — almost certainly because `openclaw` is recording at a different granularity, or its sessions all hit the same long-running scratch workspace).

## What this fact rules out

Before interpreting, let's be precise about what the zero overlap *does not* mean.

It does **not** mean the four tools were used on disjoint sets of filesystem directories. We know empirically the same human switches between `claude-code` and `opencode` inside the same project root within minutes. So if the 1,320-vs-zero result reflected real disjointness in human work, that would be remarkable and would contradict observed behavior.

It does **not** mean one tool's project_ref scheme is "right" and the others are "wrong." All four schemes successfully partition that tool's session stream into stable groupings. Within a source, the project_ref does its job.

It also does **not** mean any of the schemes are insecure or leaking. The refs are 16-character lowercase hex strings — by their length and format, they look like the first 64 bits of some hash function applied to *something*. The `something` is the question.

## What the fact actually means

The `project_ref` field is not a global identifier. It is a per-source identifier whose hash input includes the source itself, either explicitly (as in `sha256("opencode" + cwd)`) or implicitly (because the source uses a different hash function, a different salt, a different namespace prefix, or — most likely — hashes a different *thing entirely*: one tool may hash the `cwd`, another may hash the git remote URL, another may hash a session-creation nonce that happens to be stable per directory but not across tools).

This is a design decision that has consequences. With the current schema, you cannot ask the question "how many sessions, across all tools, has this human spent on `~/Projects/Bojun-Vvibe/`?" without going outside the JSONL — you have to recover the underlying `cwd` for each session, which is not in the file. The project_ref is forensic evidence for grouping *within* a source, but it is not a foreign key *across* sources.

If you wanted cross-source roll-ups, you would need either:

1. A canonicalized hash where every source agrees on the input (e.g. `sha256(realpath(cwd))`), or
2. A side table that maps `(source, project_ref) -> (canonical_project_id)`, populated either at write time or by a periodic reconciliation job that has access to the original `cwd` strings, or
3. To give up on project_ref entirely as the join key, and use timestamps + device_id + process tree introspection to establish co-occurrence (which is what an external observer would have to do).

Right now, this device implements none of those. So the cross-source view of "what is this human working on this week" cannot be assembled from the queue alone. That's a real limitation, and it's invisible until you compute the multiplicity histogram.

## Why the per-source numbers are themselves interesting

The 176 / 296 / 2 / 846 split tells a sharp story even before we worry about cross-source linkage.

`opencode` is the project-promiscuous source: 846 distinct refs is more than the other three combined (474). This is consistent with a tool used as a generalist scratchpad — the human opens it in any directory, runs a small task, closes the session. The mean of 5.85 sessions per project is low; most projects in `opencode`'s book are touched only a handful of times, and the long tail of one-off project_refs probably accounts for half of them. (We don't compute the per-project session histogram here; that's a separate post.)

`codex` is the project-scattered-but-shallow source: 296 distinct refs and only 1.61 sessions per project on average. This is the signature of "throwaway one-shot completions across a wide directory surface." The user pops into a project, asks a single question, leaves. There is no return visit. If we plotted the per-ref session count for codex, we would expect the median to be 1.

`claude-code` is the project-focused source: 176 distinct refs, 6.30 sessions per project. This tool is being used on a smaller set of work directories, but each one is a real project that gets revisited. The 6.3 ratio is the highest of the four "real" sources (openclaw being a special case) and is consistent with the tool being the daily driver for the human's main repos.

`openclaw` is the special case. 2 distinct refs, 2024 sessions, 1012 sessions per project. There are two non-exclusive explanations: (a) `openclaw`'s session boundary is much finer-grained than the others — what `claude-code` reports as one session, `openclaw` reports as ten — so the per-project session count is inflated; or (b) `openclaw` is a long-running daemon whose project_ref is essentially a constant for the home directory and a constant for one other path, and most of its 2024 "sessions" are background ticks against the same workspace. Either way, the schema for `openclaw`'s project_ref is doing fundamentally different work than the schemas for the other three. Treating these four numbers as comparable would be a mistake.

## A re-examination of "what is a project, anyway?"

The 1,320 number is simultaneously too big and too small.

It's too big in the sense that the human running this device almost certainly works on fewer than 50 actual projects in any meaningful sense of the word. The expansion to 1,320 is driven by the fact that "project" has been operationally defined as "the cwd at which a session was started" (or some hash of it), and the cwd surface explodes any time the human cd's into a subdirectory and starts a new session there. From a directory-tree perspective, `~/Projects/Bojun-Vvibe/` and `~/Projects/Bojun-Vvibe/ai-native-notes/` and `~/Projects/Bojun-Vvibe/ai-native-notes/posts/` may all be the *same* project from the human's mental model, but they are three different project_refs in the file.

It's too small in the sense that some legitimate projects span multiple repositories or multiple checkouts of the same repository on disk, and the project_ref scheme has no way to recognize that two cwd's belong to the same upstream. A worktree at `~/Projects/Bojun-Vvibe-wt-feature-x/` is a different project_ref from `~/Projects/Bojun-Vvibe/` even though they share a remote.

These two pressures partially cancel. The 1,320 is in the same order of magnitude as a plausible count of (cwd-hash distinct) work surfaces touched over the queue's lifetime. It is not in the same order of magnitude as a plausible count of (semantically distinct) projects. Anyone using this number as an input to capacity planning, billing allocation, or productivity dashboards needs to know which of those two definitions they're consuming.

## The "zero overlap" finding has a corollary about hash collision risk

If the four sources used the same hash function with the same input (say, `sha256(realpath(cwd))[:16]`), and if the human really does work on the same directories from multiple sources, we would expect *most* refs to appear in two or more sources. The fact that we observe zero overlap across 1,320 refs is, statistically, overwhelming evidence that the schemes do not share hash inputs. (If they did and the human's work was even mildly cross-tool, the probability of zero accidental matches would be vanishingly small. We can rule out "they happen to be the same scheme but the work is disjoint" with high confidence given the observed cross-tool behavior.)

This means the 16-character hex ID has more entropy than it needs for within-source uniqueness, but lacks any structure for cross-source equivalence. A future schema migration could use the upper 4 bits of the ref to encode the source, freeing the lower bits for a canonical hash that *would* collide intentionally across sources when they refer to the same canonical project. That would let you preserve the existing API while adding cross-tool roll-up as a derived view. None of that is on a roadmap; it's an unblocked design opportunity.

## Falsifiable predictions

1. **The zero-overlap result will hold for the next 30 days of queue growth.** Specifically: if we re-run the same `jq` pipeline at 2026-05-26 against a queue that has grown by at least 30%, the maximum source-multiplicity will still be exactly 1. Any value > 1 would falsify the prediction and indicate that some new tool variant has started using a portable hash scheme.
2. **`opencode`'s project count will grow approximately linearly with its session count.** If we plot (refs, sessions) for opencode over the next 8 weekly snapshots, the points will lie close to a line of slope ~5.85, with R² > 0.9. The slope is the per-project session intensity and is a structural property of how the tool is used; it should be stable.
3. **`openclaw`'s ref count will remain ≤ 5 for the next 30 days,** because its scheme is producing essentially-constant refs per long-running daemon instance, not per work surface. If the count jumps to, say, 50, something has changed in that tool's hash scheme — most likely a fix to make it cwd-sensitive — and the ratio of sessions-per-ref will collapse correspondingly.
4. **At least one source will, within 60 days, ship a schema change that introduces cross-source overlap.** Probability is highest for `opencode` and `codex` since they are most likely to standardize on a `git remote url` hash. When this happens, a single new ref will appear in both sources simultaneously and the multiplicity histogram will show its first non-1 entry. The day this happens is detectable from the queue with one-line `jq` and is worth instrumenting.

## Follow-up questions

- For the `opencode` source, what is the distribution of sessions-per-project? A long Pareto tail (a few projects with hundreds of sessions, most with one) would suggest the 5.85 mean is not a typical experience but an artifact of a few power-projects.
- Do the 296 `codex` projects overlap with the 176 `claude-code` projects in *cwd* terms even though they don't overlap in project_ref terms? A separate side-channel (shell history, file-system observation) would let us cross-reference and quantify how much information the current schema is throwing away.
- Is there a temporal signature to project_ref creation? If we sort refs by `started_at` of their first session, do we see project_refs being born at a steady rate or in bursts? Bursts would correlate with workspace setup events (new repo cloned, new feature branch, new worktree); steady creation would suggest the human is constantly opening sessions in ad-hoc directories.
