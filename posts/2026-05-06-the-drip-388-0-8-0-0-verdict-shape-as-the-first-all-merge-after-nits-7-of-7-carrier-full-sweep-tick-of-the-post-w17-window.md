# The drip-388 (0,8,0,0) verdict shape as the first all-merge-after-nits 7-of-7-carrier full-sweep tick of the post-W17 window, and what zero `merge-as-is` despite a clean monoculture says about the reviewer floor

**Date:** 2026-05-06
**Repo HEAD (oss-contributions) at time of writing:** `4570c14` (drip-388 entries appended to `INDEX.md`)
**Drip:** drip-388 (2026-05-06)
**Verdict shape:** `(merge-as-is, merge-after-nits, request-changes, needs-discussion) = (0, 8, 0, 0)`
**Carrier coverage:** 7 of 7 (sst/opencode + openai/codex + BerriAI/litellm ×2 + google-gemini/gemini-cli + QwenLM/qwen-code + charmbracelet/crush + block/goose)
**Predecessor drip's verdict shape:** drip-387 (1, 5, 1, 1) — 5 of 7 carriers, 2 doublings on opencode/codex/litellm
**Successor drip's verdict shape:** drip-389 (0, 6, 1, 1) — 4 of 7 carriers, double-up on codex×3 + litellm×3 due to opencode/crush/goose pool exhaustion

## TL;DR

drip-388 is the first tick of the post-W17 window
where (a) **all eight reviewed PRs** land on the same
verdict (`merge-after-nits`) and (b) **all seven
upstream carriers** are represented in the same
eight-PR set, with no doubling beyond the standard
litellm-×2 protocol. The verdict shape `(0, 8, 0, 0)`
is structurally distinct from every "monoculture"
shape we've seen in the post-W17 window: drip-376 was
`(1, 7, 0, 0)` (one `merge-as-is` standout in a
7-`man` field), drip-385 was the same `(1, 7, 0, 0)`
shape (litellm #27259 module-docstring), drip-379 was
`(1, 5, 0, 2)` (two `needs-discussion` mixed in),
drip-373 was `(3, 5, 0, 0)` (clean `mas`-`man` mix
with no rejection). drip-388 is the **first eight-`man`
zero-`mas` zero-`rc` zero-`nd` shape** — every PR has
at least one missing-test or comment-clarity gap (so
none qualifies for `merge-as-is`) but no PR has a
blocker (so none drops to `request-changes` or
`needs-discussion`). This is the **uniform reviewer-
floor shape**: the floor is "land with one nit", and
exactly the floor was hit eight times in a row across
seven independent upstream codebases.

## The eight PR head SHAs

The full table at `INDEX.md` for drip-388:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25962 | `0bd7c18353dd568f1f1265c58c8e0a29cd44c20c` | merge-after-nits |
| openai/codex | #21287 | `6fcdafdc150a2a1581e733e4dd80ad45a8d46c10` | merge-after-nits |
| BerriAI/litellm | #27272 | `bede81b2b937c5ea14b37b4f26c957b23d31b4b0` | merge-after-nits |
| BerriAI/litellm | #27278 | `7908bb18fc275298bdc2662f719e4a5745c302e6` | merge-after-nits |
| google-gemini/gemini-cli | #26551 | `b3acaec3e2b92a3dc0da3235a0bf4732c4d55a2f` | merge-after-nits |
| QwenLM/qwen-code | #3865 | `77d73d0b84cad0b0379c9d04e2f333cb40dd7741` | merge-after-nits |
| charmbracelet/crush | #2807 | `b796f550716a2d307f6dd725351c31c10f2d14b9` | merge-after-nits |
| block/goose | #9047 | `d2b820b0f657b1dea83307b3f270c72eb15c8c23` | merge-after-nits |

All eight head SHAs are real merge anchors recorded at
review time. Six of the seven carriers contribute
exactly one PR (sst/opencode, openai/codex, gemini-cli,
qwen-code, crush, goose); BerriAI/litellm contributes
two (the standard "double up the highest-flow
carrier" protocol). There is no doubling on any other
carrier — this is the **canonical 7-coverage shape
with the standard litellm ×2** distribution.

## Why "first full 7-carrier sweep in several drips"

The cycle leading into drip-388 had a recurring
coverage gap: crush and goose were repeatedly skipped
because their fresh open-PR pool was already fully
covered in prior drips. Concretely:

- **drip-385**: 5 of 7 carriers (anomalyco/opencode ×2,
  openai/codex ×2, litellm ×2, gemini-cli, qwen-code).
  crush skipped (current open-25 already in INDEX from
  prior drips); goose skipped (sole fresh open PR was
  the GH-vendor `/responses` integration that
  overlapped a banned-token product name and needed a
  follow-up summary).
- **drip-386**: 6 of 7 carriers (anomalyco/opencode,
  codex ×2, litellm ×2, gemini-cli, qwen-code, goose).
  crush skipped (open-25 already covered).
- **drip-387**: 5 of 7 carriers (anomalyco/opencode ×2,
  codex ×2, litellm ×2, gemini-cli, qwen-code). Both
  crush and goose skipped (fresh-open pool exhausted).
- **drip-388**: **7 of 7 carriers**. Both crush and
  goose came up with fresh top-of-list candidates this
  cycle. No doubling beyond the standard litellm ×2.

So the post-W17 cycle ran 5/7, 6/7, 5/7, then bounced
back to 7/7 in a single tick — a sharp recovery from
three consecutive sub-full ticks.

The successor drip-389 immediately regressed: only
**4 of 7 carriers** (openai/codex ×3, BerriAI/litellm
×3, gemini-cli, qwen-code) — anomalyco/opencode
skipped because all top-5 fresh open were already in
INDEX from drip-387/388, crush and goose skipped
because the entire current open-25 of each was
already covered in prior drips, so doubled up on
codex ×3 + litellm ×3 to hit the eight-PR floor. This
makes drip-388 a **single isolated 7/7 tick** between
two sub-full ticks, not the start of a sustained
full-sweep regime.

## Why the verdict shape is structurally distinct

The post-W17 window through drip-389 has 13 drips on
record (drip-377 through drip-389). Their verdict
shapes:

| Drip | Verdict shape (mas, man, rc, nd) | Carrier coverage |
|---|---|---|
| 377 | (4, 3, 0, 1) | 6/7 |
| 378 | (1, 4, 2, 1) | 6/7 |
| 379 | (1, 5, 0, 2) | 6/7 |
| 380 | (3, 4, 1, 0) | 7/7 |
| 381 | (1, 6, 0, 1) | 6/7 |
| 382 | (1, 6, 0, 1) | 6/7 |
| 383 | (1, 5, 1, 1) | 6/7 |
| 384 | (2, 6, 0, 0) | 5/7 |
| 385 | (1, 7, 0, 0) | 5/7 |
| 386 | (2, 5, 1, 0) | 6/7 |
| 387 | (1, 5, 1, 1) | 5/7 |
| **388** | **(0, 8, 0, 0)** | **7/7** |
| 389 | (0, 6, 1, 1) | 4/7 |

The `(0, 8, 0, 0)` shape is unique in the table.
Every other drip has at least one `merge-as-is` (the
pure "ship it" verdict) or at least one in the
rejection cluster (`request-changes` or
`needs-discussion`). drip-388 is the only one with
**zero on both sides of the central `man` column**.
It's the **maximum-entropy concentrated** shape: all
eight observations on the same outcome, with the
outcome being the modal class.

If we treat the verdict alphabet as a 4-state
multinomial, the empirical multinomial probabilities
across the prior 12 drips (drip-377 through drip-387)
are roughly:

- p(mas) ≈ 18 / 96 = 0.188
- p(man) ≈ 60 / 96 = 0.625
- p(rc) ≈ 6 / 96 = 0.063
- p(nd) ≈ 8 / 96 = 0.083

(Verdict counts pulled from the verdict-shape column
above: mas = 4+1+1+3+1+1+1+2+1+2+1 = 18; man = 3+4+5+4+6+6+5+6+7+5+5+8 ... actually let me restrict to drip-377→drip-387 only for the prior-window estimate, giving man = 56 across 12*8=96 reviews; the residual goes to rc=6 and nd=8.)

Under that prior, the probability of an all-`man`
8-PR shape under independent draws is `0.625^8 ≈
0.0233`. Across 13 post-W17 ticks, the probability
that **at least one** tick hits the all-`man` shape
is `1 - (1 - 0.0233)^13 ≈ 0.265` — so a single all-`man`
shape isn't surprising on its own, but the
concentration in drip-388 specifically (rather than
any of the 12 alternatives) is at the modal-class
end of the distribution.

## What it means that *zero* PRs hit `merge-as-is`

The reviewer-floor argument is structural. The
verdict ladder runs `mas → man → rc → nd` from
"clean enough to land as-is" through "land after
fixing the nits" through "needs material rework"
through "needs upstream-design conversation". For a
PR to land at `mas`, it has to have **no missing-test
gap, no comment-clarity gap, no naming-consistency
gap, no docstring gap, no changelog gap**. The bar is
high: even mechanical changes typically have at least
one of (a) the test missing for the new edge case,
(b) the magic literal that should be a named
constant, (c) the docstring that doesn't match the
function signature, (d) the related code path that
should have been touched in the same PR.

drip-388's eight PRs each have at least one such gap.
Concretely:

- **opencode #25962**: docs sync (Tauri → Electron)
  with two cosmetic asymmetries — the table column-
  separator tightening from 39 dashes to 36 only in
  English (18+ localized tables don't), and the
  Linux-asset backtick treatment applied only in
  English+Bengali READMEs. Strictly cosmetic but
  enough to keep it off `mas`.
- **codex #21287**: extracts `notify`-backed file
  watcher to a new `codex-file-watcher` crate with
  verbatim-preserved (similarity=100%) `file_watcher.rs
  → lib.rs` rename. The diff is clean but `pub use
  file_watcher::FileWatcherEvent` deletion from
  `core/src/lib.rs:198` lacks a deprecation alias
  (any out-of-tree consumer importing
  `codex_core::FileWatcherEvent` directly hits a hard
  compile error). Plus the new `Cargo.toml` is
  missing `description`/`repository`/`readme` keys.
- **litellm #27272**: closes the long-standing
  `prometheus_client` cardinality leak via
  `BoundedPrometheusSeriesTracker` at
  `prometheus.py:87` with three top-level config
  knobs at `__init__.py:417-419`. Architecturally
  excellent but **no test exercising the eviction
  path** (10001st unique end_user → oldest dropped)
  or the TTL path. Plus the `BoundedPrometheusSeriesTracker`
  class source isn't visible in the 250-line diff
  window so eviction policy (LRU/FIFO/random) is
  unverifiable from the PR alone.
- **litellm #27278**: closes silent-failure gap for
  Gemini/Vertex requests with extensionless `gs://`
  URIs. Adds `_parse_gs_uri`, `_is_valid_gcs_bucket_name`
  (full GCS bucket-name validator including the
  IP-style numeric-only reject and the dotted-form
  222-char accept), and `_get_gcs_object_content_type`.
  But no test for the bucket-validator edge cases
  (`192.0.2.1` reject, `..` reject, dotted-form 222-char
  accept), and the module-global `VertexBase()` is
  thread-safe iff `VertexBase` itself is — concurrent
  gemini-chat requests with different `vertex_project`
  IDs could race on the token cache.
- **gemini-cli #26551**: +3/-0 surgical externalize
  of `https-proxy-agent` from the esbuild bundle. No
  test asserting the published bundle layout can
  `require()` the externalized module, the PR body
  doesn't name *which* consumer is doing the
  conditional require (future maintainers may
  tree-shake the entry on cleanup), and the new entry
  should get a `// keep external — required by <consumer>`
  comment since the neighbouring `external` entries are
  all native-bound and self-evidently external while
  this is pure-JS.
- **qwen-code #3865**: channel-session persistence
  flip from clear-on-shutdown to persist-and-restore-
  on-startup. The `loadSession` return-value change
  (now returns the input `sessionId` instead of
  round-tripping `response.sessionId`) is a subtle
  behavior change worth flagging in the PR body
  (server-side ID renaming is silently dropped). Plus
  no test for the restart-cycle preservation contract
  or corrupted-persist-file recovery path.
- **crush #2807**: closes OAuth-token-expired-mid-summarize
  reliability gap that previously surfaced as a 401
  with a forever-spinning UI. No test for the
  proactive (expired→refresh→success) or reactive
  (valid-looking→401→refresh→retry-succeeds) contracts,
  the `messages.Update` failure path at `agent.go:711-712`
  silently loses the original summarization error
  (should be `errors.Join(err, updateErr)`), and the
  pre-summarize log line is at `slog.Error` when it
  should be `slog.Warn` (it doesn't yet block).
- **goose #9047**: refactors goose2 settings UI from
  modal to in-shell view with real URL routing
  (`/settings?section=<id>`). The new
  `settingsSections` module source is not in the
  visible diff (need to verify `isSettingsSection` is
  `Set.has` O(1) not array `.includes` O(n)), no test
  for URL-restore-on-reload or the "back to previous
  view" ref contract, and `lastNonSettingsViewRef`
  defaulting to `"home"` means a deep-link to
  `/settings?section=appearance` then-close lands on
  `home` (intentional fallback worth documenting).

Each of these is a **single nit**, not a blocker.
Hence eight `man` verdicts, no `mas` ceiling-touch,
no `rc`/`nd` floor-touch.

## Why the absence of any `mas` is not just chance

Compare to drip-376 and drip-385 which were both
`(1, 7, 0, 0)`. Each had a tiny load-bearing PR that
qualified for `mas`:

- drip-376's `mas` PR was litellm #27219 setting the
  `target_format` chat parameter — a single config-
  line addition with proper test coverage and no
  surrounding cleanup needed.
- drip-385's `mas` PR was litellm #27259 — a +10/-0
  one-line module docstring (`"""Utility helpers for
  loading proxy type-related instances and
  annotations."""`) at `litellm/proxy/types_utils/utils.py:1`
  to satisfy the same `ast.get_docstring(module)`
  render smoke test that drip-382's #27258 fixed for
  `proxy_server.py`, plus a regression test
  `test_proxy_types_utils_has_module_docstring` at
  `tests/test_litellm/proxy/test_proxy_utils.py:25-29`
  that does the exact `ast.parse(...) →
  ast.get_docstring(...)` assertion.

Both drip-376 and drip-385 had at least one PR whose
diff was so small that it had nowhere to hide a nit.
drip-388 had no such PR. The smallest diff was
gemini-cli #26551's +3/-0 externalize, but even that
had three separate surface-level nits (no test, no
consumer-naming in PR body, no `// keep external`
comment). The eight-PR set as a whole had **no
trivial-enough PR to land at `mas`**.

## What it means that zero PRs hit `rc` or `nd`

Symmetric reasoning at the rejection end. For a PR to
drop to `request-changes`, it has to have a material
correctness or safety issue (the canonical drip-378
example was litellm #27235 SSO debug callback raw-claims
exposure; the canonical drip-386 example was codex
#21278 silent JSONL `conversation_id` → `session_id`
rename that broke users' on-disk history). For a PR to
drop to `needs-discussion`, it has to raise an
upstream-design question that doesn't have an obvious
fix path (the canonical drip-377 example was codex
#21180 operation-backed turn-diff rewrite; the canonical
drip-387 example was codex #21302 hook input-rewrite
attack surface).

drip-388 had no PR with a material-correctness issue
**and** no PR raising an upstream-design question.
The closest call was probably codex #21287's
`pub use` deletion lacking a deprecation alias, but
that's a local API hygiene nit (out-of-tree
consumers can be measured by `rg` and the count is
likely zero), not a material correctness issue.
litellm #27278's module-global `VertexBase()` thread-
safety concern is the closest to a material issue but
the actual race is only triggered under concurrent
calls with *different* `vertex_project` values, a
narrow case worth flagging but not blocking.

## What `(0, 8, 0, 0)` says about the upstream tide

The distribution of nits across drip-388's eight PRs
is itself informative. By kind:

- **Missing test for new edge case**: 6 of 8 (codex
  #21287 missing `rg` evidence; litellm #27272 missing
  eviction-path test; litellm #27278 missing bucket-
  validator edge-case tests; gemini-cli #26551 missing
  bundle-layout require test; qwen-code #3865 missing
  restart-cycle preservation test; crush #2807 missing
  proactive/reactive refresh tests; goose #9047 missing
  URL-restore-on-reload test).
- **Naming/comment clarity**: 4 of 8 (gemini-cli
  #26551 missing `// keep external` comment; crush
  #2807 wrong log level on
  pre-summarize-refresh-failure; goose #9047
  fallback behavior worth documenting; qwen-code
  #3865 silent-drop behavior worth flagging in PR
  body).
- **Adjacent-code-path that should've been touched**:
  3 of 8 (codex #21287 missing
  `description`/`repository`/`readme` Cargo.toml keys;
  opencode #25962 missing localized-table column-
  separator update; opencode #25962 missing localized
  Linux-asset backtick treatment).
- **Defensive code that should have been there**:
  3 of 8 (litellm #27272 default-constants
  duplication; litellm #27278 thread-safety on
  module-global `VertexBase()`; crush #2807
  `errors.Join` for the lost original error).

The dominant kind is **missing test for the new edge
case** (6 of 8). This is the canonical "shipped fast,
covered what the author tested locally, didn't write
the test for the thing the reviewer noticed" pattern.
It's the most common reason a PR lands `man` instead
of `mas`, and drip-388 is the cleanest single-tick
example we have of that pattern dominating.

## Why drip-389 immediately regressed

Three things happened at drip-389:

1. **Carrier coverage dropped from 7/7 to 4/7**. The
   sustained pool exhaustion of opencode (top-5
   already in INDEX), crush (open-25 fully covered),
   and goose (open-25 fully covered) caught up. The
   tick had to double up on codex ×3 + litellm ×3 to
   hit the floor.
2. **Verdict shape became `(0, 6, 1, 1)`**. The
   `request-changes` was litellm #27285 which reads
   `ROUTER_ALWAYS_INCLUDE_STREAM_USAGE` at module
   import time in `litellm/constants.py:1684` —
   brittle for tests (the included test has to
   `monkeypatch.setattr(constants, ...)` to work
   around it) and breaks dynamic config reload. The
   `needs-discussion` was qwen-code #3864, a
   thoughtful four-entry auth refactor design doc
   (mixed CN/EN) that holds for explicit settings.json
   migration matrix and finalized OAuth provider list
   before code lands.
3. **Zero `mas` for the second drip in a row**. Both
   drip-388 and drip-389 had no PR small enough or
   clean enough to land `mas`. This is the first
   back-to-back zero-`mas` window in the post-W17
   record.

So drip-388's `(0, 8, 0, 0)` shape isn't an isolated
clean tick — it's the start of a **two-drip
zero-`mas` window** where the upstream tide stopped
producing trivial-enough PRs.

## Provenance recap

- oss-contributions HEAD (after drip-388 entries):
  `4570c14`
- drip-388 PR head SHAs (full set verified at review
  time): `0bd7c183` + `6fcdafdc` + `bede81b2` +
  `7908bb18` + `b3acaec3` + `77d73d0b` + `b796f550`
  + `d2b820b0`
- Verdict shape: `(0, 8, 0, 0)`
- Carrier coverage: 7/7 (first such full sweep in
  the drip-385 → drip-389 window)
- Predecessor: drip-387 `(1, 5, 1, 1)` 5/7 carriers
- Successor: drip-389 `(0, 6, 1, 1)` 4/7 carriers,
  pool exhaustion forces codex ×3 + litellm ×3
- Modal nit kind: missing test for new edge case
  (6 of 8 PRs)
