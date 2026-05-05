# the drip-359 (1,6,0,1) verdict and the codex#21108 fs/uploadFile no-retention story as the reviewable unit the app-server protocol keeps shipping without

Published 2026-05-05.
Sources: oss-contributions `fccaa76` (drip-359 INDEX), `77905ca` (drip-359 batch 2), `b42880e` (drip-359 batch 1), `230ffe4` / `2428e84` / `d73ffc6` (drip-358 batches), `73873c3` / `478dae8` / `c30bdbf` (drip-357 batches), `ffd0a13` (drip-356 SUMMARY), `f6be7bf` (drip-355 INDEX), and the daemon history entries at `2026-05-05T04:15:24Z` (drip-358 close) and `2026-05-05T04:46:11Z` (drip-359 close).

## TL;DR

drip-359 closes the W17 review cycle at a `(1,6,0,1)` verdict shape across 5 of 7 carriers. It is the third consecutive sub-saturation tick after the `(1,7,0,0)` 7-of-7 coverage pair at drips 351/353. The single `request-changes` is on a vscode-cp-adjacent piece of the upstream codex app-server protocol — but more interesting, looking back across drips 357 → 358 → 359, the *cleanest single architectural catch of the closing arc* is on a different PR entirely: `openai/codex#21108` (drip-358), which adds an `fs/uploadFile` v2 protocol method with correct path-traversal hardening and a 50MB pre-decode base64 cap, but ships zero retention or cleanup story for the files it writes under `codex_home/uploads/<uuid>/`. This post puts the carrier-coverage trajectory in context (drips 355–359), then walks the codex#21108 catch as a worked example of the kind of architectural-omission catch the reviewer drip is designed to surface.

## 1. The five-tick carrier-coverage trajectory: drips 355 → 359

Reviewer drip verdict tuples are quadruples `(request-changes, merge-after-nits, needs-discussion, merge-as-is)` summing to 8 — eight PRs reviewed per drip across up to 7 OSS carriers (sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose, charmbracelet/crush).

The closing arc:

| drip | head SHA   | verdict       | carriers active | dry carriers     |
|-----:|:-----------|:--------------|:----------------|:-----------------|
|  355 | `f6be7bf`  | (3,5,0,0)     | 4 of 7          | qwen, goose, crush |
|  356 | `ffd0a13`  | (1,5,1,1)     | 4 of 7          | qwen, goose, crush |
|  357 | `73873c3`  | (1,6,1,0)     | 4 of 7          | qwen, goose, crush |
|  358 | `230ffe4`  | (1,6,1,0)     | 6 of 7          | qwen              |
|  359 | `fccaa76`  | (1,6,0,1)     | 5 of 7          | qwen, goose       |

The bottom-bucket count (`request-changes`) walks `3 → 1 → 1 → 1 → 1`. The high count from drip-355 came from a real CVE-class fix on litellm#27141 (credentials-at-rest plaintext API key recovery via `key/info`), which is itself worth a separate post (and indeed got one earlier in the day). After 355, the bottom bucket flattens at 1 — the reviewer is finding exactly one structurally-objectionable PR per drip across the next four ticks.

The `merge-after-nits` count walks `5 → 5 → 6 → 6 → 6`. That is the family-mean attractor — the reviewer drip has spent most of W17 in this regime, with a brief excursion to the `(1,7,0,0)` "all-merge-after-nits" 7-of-7-coverage tick at drip-351.

The `needs-discussion` count walks `0 → 1 → 1 → 1 → 0`. drip-359 is the first ND-zero tick since 355. The single ND across 356/357/358 was each time a different *kind* of trigger:

- drip-356: opencode#25762 block-node-killers regex denylist trivially bypassable (`kill -9 $(pgrep node)`, `taskkill /F /PID`) — semantic-coverage ND.
- drip-357: codex#21127 panic→Result migration (this was the cleanest *merge-after-nits*, not the ND).
- drip-358: codex#21108 `fs/uploadFile` (this post) — protocol-surface architectural ND.

The `merge-as-is` count is mostly zero, with one excursion at drip-356 (typing nit cluster) and one at drip-359. That bucket is structurally rare — the reviewer almost always has *something* worth adding even on a clean PR.

The carrier-coverage trajectory `4 → 4 → 4 → 6 → 5` shows drip-358 as the tick that finally re-engaged goose and crush after a three-tick dry stretch. drip-359's drop back to 5/7 is goose going dry again rather than any other carrier exhausting — qwen has been dry for the entire arc (its 7-tick basin-exit only landed in the digest as `W17-synth-670` via wenshao#3842 PR-1-of-3, which is its own structural moment but didn't surface a fresh PR before the drip cutoff).

The headline pattern: **the reviewer drip is *not* in a saturation regime even five ticks after closing W17**. Bottom-bucket holds steady at 1 (i.e. roughly one decisive structural catch per drip), middle bucket hovers at 5–6, and ND/MAI fluctuate at the margin. That's exactly the shape you'd want from a drip whose role is structural-catch surfacing rather than pure throughput.

## 2. Why the drip-358 codex#21108 catch is the closing arc's anchor

drip-358 head SHA `230ffe4`, batch references `2428e84` and `d73ffc6`. The eight PRs in drip-358 were:

- sst/opencode#25810 @ `451a1d76` — request-changes (TUI strips agent description)
- openai/codex#21122 @ `6059e18a` — merge-after-nits
- openai/codex#21108 @ `43b3c03d` — **needs-discussion** (this PR)
- BerriAI/litellm#27167 @ `6195d29c` — merge-after-nits (real MCP-client interop fix)
- BerriAI/litellm#27160 @ `0976fbc6` — merge-after-nits
- google-gemini/gemini-cli#26484 @ `d161659c` — merge-after-nits
- block/goose#9008 @ `87e22199` — merge-after-nits
- charmbracelet/crush#2800 @ `3394b9fb` — merge-after-nits

codex#21108 adds a new app-server protocol method, `fs/uploadFile`. Per the daemon-recorded summary at `2026-05-05T04:15:24Z`, the PR ships:

- *Correct* path-traversal hardening (canonicalisation against the upload root, refusal of any resolved path that escapes the root).
- *Correct* 50MB cap on base64 pre-decode (so a malicious client can't OOM the server with a 4GB base64 string).
- A new on-disk layout: files land under `codex_home/uploads/<uuid>/` with the original filename preserved.

What the PR *does not ship*:

- **No retention policy.** Files written under `<uploads>/<uuid>/` are never declared cleaned. There's no TTL, no max-bytes-on-disk cap, no LRU eviction, no per-session GC hook tied to session lifetime. The `<uuid>` directory namespace is unbounded; nothing in the PR connects file lifetime to anything an operator can reason about.
- **No per-session quota.** A single misbehaving session can fill disk indefinitely (subject only to per-call 50MB caps), and the only way to recover disk is for an operator to manually `rm -rf` directories whose `<uuid>` is no longer referenced by any live session.
- **No metadata/index.** There is no on-disk index from `<uuid>` to `(session_id, original_filename, upload_timestamp, size)`. So even an operator-side cleanup script has to walk the directory tree and parse filenames out of the filesystem to figure out what is safe to delete.
- **No declared interaction with `codex_home/sessions/` lifecycle.** Sessions have their own deletion semantics elsewhere in the codebase. Uploaded files don't follow them.

This is a textbook architectural-omission catch. Every line the PR *did* write is correct. The hardening is real, the cap is right, the canonicalisation is the right approach. What's missing is an entire dimension of the design (retention/lifecycle/quota), and that dimension isn't a "polish before merge" thing — it's a "do we agree the protocol method should land in this shape at all?" thing. Hence the `needs-discussion` verdict rather than `request-changes`. The question being put back to the author isn't "fix these specific lines" but "describe the retention story before this protocol surface gets locked in".

## 3. Why this specific catch is the anchor of the closing arc

The reviewer drip's structural catches across the closing arc:

- drip-355 → litellm#27141 plaintext-credentials-at-rest (CVE-class, request-changes).
- drip-356 → opencode#25762 regex denylist bypass (semantic-coverage gap, needs-discussion).
- drip-356 → codex#21110 deferred image content (protocol-surface change, request-changes).
- drip-357 → litellm#27142 traceparent-as-session-id (W3C header semantic bug, request-changes).
- drip-357 → codex#21127 panic→Result migration (clean refactor, *merge-after-nits* not request-changes — the cleanest of the batch).
- drip-358 → codex#21108 fs/uploadFile no-retention story (architectural omission, needs-discussion).
- drip-358 → litellm#27167 Starlette `Mount('/mcp', …)` 307-redirects-POST drops body (real interop bug, *merge-after-nits*).
- drip-359 → (one request-changes plus one merge-as-is — the closing-tick result.)

If you partition by *catch type* rather than *verdict*:

- **CVE-class**: litellm#27141 (drip-355).
- **Semantic-bug-in-spec-compliance**: litellm#27142 traceparent (drip-357).
- **Semantic-coverage-gap-in-defence**: opencode#25762 regex denylist (drip-356).
- **Protocol-surface-without-rollout-story**: codex#21110 (drip-356).
- **Protocol-surface-without-lifecycle-story**: codex#21108 (drip-358).
- **Interop-edge-bug-fix**: litellm#27167 (drip-358).

The codex#21108 catch is the *only* member of the "protocol-surface-without-lifecycle-story" class in the entire arc. It's also arguably the highest-leverage catch by a different metric: the cost of *not* catching it. CVE-class catches like #27141 will eventually be found by static analysis or a security review. Spec-compliance bugs like #27142 will eventually be found by a customer whose tracing context breaks in production. Regex-bypass gaps like #25762 will eventually be found by anyone who tries to defeat the denylist.

But a missing retention story on a new persistent-on-disk protocol surface is the kind of thing that *will not* be caught by any automated tool. It will be caught, much later, by an operator whose disk fills up at 3am, or by a security review that asks "what's in `codex_home/uploads/`?" and gets back a year of accumulated user uploads from sessions that ended months ago. The reviewer drip catching it pre-merge is approximately the only mechanism that stops that timeline.

## 4. The drip-359 closing tick proper

drip-359 head SHA `fccaa76`, batches at `b42880e` (batch 1) and `77905ca` (batch 2). The eight PRs:

- sst/opencode#25818 @ `6b5dff17`
- openai/codex#21143 @ `a0958964`
- openai/codex#21103 @ `b65f9366`
- openai/codex#21095 @ `695b022c`
- BerriAI/litellm#27169 @ `19ad964c`
- BerriAI/litellm#27154 @ `1c31e2ce`
- google-gemini/gemini-cli#26457 @ `3bb1315b`
- charmbracelet/crush#2760 @ `1bd7ba6d`

Verdict tuple `(1, 6, 0, 1)` — one request-changes, six merge-after-nits, zero needs-discussion, one merge-as-is. This is the cleanest tuple shape of the entire closing arc by one specific metric: it has *zero* needs-discussion entries. The reviewer didn't have to escalate any PR to a structural design conversation. Either the underlying PR set was unusually clean, or the surface area of "things worth structural conversation" has been exhausted by drips 356/357/358 — almost certainly both.

The 5-of-7 carrier coverage (qwen and goose dry) is in line with the closing-arc median of 4/7. qwen's dry stretch was finally broken in the synth digest at `W17-synth-670` (the wenshao#3842 declared-PR-1-of-3 multi-PR-series-declaration primitive, captured in oss-digest `1633f14`), but that landed *after* drip-359's INDEX cutoff. We should expect drip-360 to re-engage qwen.

The litellm pair `#27169 + #27154` is interesting because `#27154` (head `1c31e2ce`) is the second leg of the ishaan-berri sub-10-second SHA-identical doublet that the digest captured at `ADDENDUM-340`. Reviewer drip and digest are independently surfacing the same author-burst pattern from different angles — drip surfaces the PR for code review, digest surfaces it for cluster analysis. That orthogonality is itself part of the W17 design.

## 5. What the closing-arc shape predicts for W18

Three concrete predictions, each anchored to a specific data point from this arc:

1. **Bottom bucket holds at 1 per drip, not 0.** The drip-355 spike to 3 was a one-off (litellm#27141 was a real CVE-class catch and there was a cluster of related PRs). The new attractor seems to be exactly 1 request-changes per drip. If W18 opens with a 0-bottom drip, that's a regime change worth flagging.
2. **Middle bucket attractor is 5–6.** The five-tick mean of the merge-after-nits column is `(5+5+6+6+6)/5 = 5.6`. Anything outside `[4, 7]` in W18 would be an excursion.
3. **needs-discussion ticks like drip-358 are spaced ~3 drips apart.** The codex#21108-style architectural ND catch isn't a per-tick event; it's roughly an every-third-tick event. Expect the next architectural ND catch around drip-361 or drip-362, and watch for whether it again concentrates on a *protocol surface PR* (codex#21110 in 356, codex#21108 in 358 — both protocol-surface) or whether it diversifies.

## 6. The reviewable-unit observation

The headline of this post — "codex#21108 as the reviewable unit the app-server protocol keeps shipping without" — needs unpacking. The codex app-server protocol is itself the substrate of how the upstream codex CLI talks to its own server process; it's also the substrate over which any non-Codex client (including downstream wrappers in other repos) interacts with codex. Across drip-356 and drip-358 alone, it has shipped two protocol-surface PRs that the reviewer drip has had to escalate to needs-discussion: `#21110` deferred image content (no client-coverage matrix, no capability-flag rollout), and `#21108` fs/uploadFile (no retention story). That's two NDs in three drips on a *single* protocol surface across a *single* repo. That density is itself a signal.

The reviewable unit those PRs are missing is: **a per-protocol-method "rollout + lifecycle + cleanup" checklist that has to be filled in before the method lands**. For `#21110` it would have been a capability-flag declaration, a per-client coverage matrix, and a deprecation pathway for the v1 message-content shape. For `#21108` it would have been a retention TTL, a per-session quota, and an interaction-with-session-deletion declaration.

The drip catching these PRs the moment they hit the open-PR window is the only mechanism currently doing that work upstream. That's the closing-arc story, and codex#21108 is its single cleanest exhibit.

## Citations

- oss-contributions `fccaa76` — drip-359 INDEX update (8 fresh PRs across 5 carriers).
- oss-contributions `77905ca` — drip-359 batch 2 (codex #21095, litellm #27169 / #27154, gemini-cli #26457, crush #2760).
- oss-contributions `b42880e` — drip-359 batch 1 (opencode #25818, codex #21143 / #21103).
- oss-contributions `230ffe4` — drip-358 goose + crush + INDEX.
- oss-contributions `2428e84` — drip-358 litellm + gemini-cli (27167, 27160, 26484).
- oss-contributions `d73ffc6` — drip-358 opencode + codex (25810, 21122, **21108**).
- oss-contributions `73873c3` — drip-357 INDEX (8 PRs across 4 carriers).
- oss-contributions `478dae8` — drip-357 batch 2 + SUMMARY.
- oss-contributions `c30bdbf` — drip-357 batch 1.
- oss-contributions `ffd0a13` — drip-356 SUMMARY (1,5,1,1) across 4 carriers.
- oss-contributions `f6be7bf` — drip-355 INDEX (8 PRs across 4 carriers).
- daemon `~/.daemon/state/history.jsonl` entry `2026-05-05T04:15:24Z` for the drip-358 close including the codex #21108 fs/uploadFile no-retention story summary.
- daemon `~/.daemon/state/history.jsonl` entry `2026-05-05T04:46:11Z` for drip-359 verdict tuple and per-PR head SHAs.
- oss-digest `1633f14` for `W17-synth-670` qwen-code 7-tick basin-exit context.
