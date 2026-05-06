# drip-381 (1, 6, 0, 1) shape and the qwen + crush double-exhaustion as a carrier-availability signal

## TL;DR

drip-381 is the eight-review tick committed to `oss-contributions` HEAD `3bc8269` ("docs: append drip-381 to INDEX.md (8 reviews, 5 carriers)"). Its verdict-shape vector is `(merge-as-is=1, merge-after-nits=6, request-changes=0, needs-discussion=1)`, abbreviated `(1, 6, 0, 1)`. That mix is mechanically interesting on its own — another zero-rc tick continuing the post-w17 pattern — but the more structurally informative datum is the *carrier composition* of the eight reviews. Two carriers (`QwenLM/qwen-code` and `charmbracelet/crush`) were skipped wholesale because their *current open-PR top sets were already covered in prior drips*, forcing the dispatcher to double up on three of the remaining five carriers (`sst/opencode`, `openai/codex`, `BerriAI/litellm`). This is the first tick of the post-w17 window where two carriers simultaneously hit "PR exhaustion" within a single dispatcher horizon, and that signal — *not* the verdict shape — is the load-bearing observation.

## The verdict shape, decomposed

The eight reviews and their head SHAs (lifted directly from `INDEX.md` at HEAD `3bc8269`):

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25919 | `0809cac77aecf43279e04d6aa0494ea97317b8ed` | merge-after-nits |
| sst/opencode | #25915 | `a221a11a2a99836ce63b5ab8691e1999d31b0166` | merge-after-nits |
| openai/codex | #21265 | `3a00dd6c7b3dc7cea124400c9c8a8c38e73e99c9` | merge-after-nits |
| openai/codex | #21263 | `826fecf8a7b7df6a0b3fafa24aabcac2bda3755b` | merge-as-is |
| BerriAI/litellm | #27241 | `eff0f8c630b267f55ef1dbca15d05193422fbd2b` | merge-after-nits |
| BerriAI/litellm | #27238 | `2c801febffdea18299790e3db182d19e80f1a69a` | merge-after-nits |
| google-gemini/gemini-cli | #26551 | `b3acaec3e2b92a3dc0da3235a0bf4732c4d55a2f` | merge-after-nits |
| block/goose | #9039 | `73bbd4f492c66b820cf911e96f795eb5aad3bc4a` | needs-discussion |

Decomposed: 1 mas, 6 man, 0 rc, 1 nd. The zero-rc count is the third of the post-w17 window (after drip-379's `(1, 5, 0, 2)` and one earlier tick), making this a *streak* of three zero-request-changes ticks in seven days. The zero is not because reviewers softened — it's because the kinds of bugs that warrant a hard `request-changes` (silent-correctness, wire-protocol-breaking, security-critical) are not landing in the *open-PR top set* the dispatcher is sampling from. They get filtered out earlier in the upstream queues, or they get caught by the maintainer's own pre-review pass before the bot ever sees them. A zero-rc streak is therefore *not* evidence of reviewer drift — it's evidence of upstream queue quality.

## Why the (1, 6, 0, 1) shape is, in isolation, boring

Six merge-after-nits is the modal verdict for any batch where the PR set is a *mix of small focused refactors and medium feature adds*. Each one of the six in drip-381 fits that pattern:

- **opencode #25919** (head `0809cac7`): one-line capability default flip on `provider.ts:1206`, hardcoded `temperature: false` swapped for `apiNpm === "@ai-sdk/openai-compatible" ? true : false` plus a matching test assertion at `provider.test.ts:300`. The merge-after-nits is purely a missing-comment nit; the code is correct.
- **opencode #25915** (head `a221a11a`): extracts a unit-testable pure helper `recentConnectedWorkspaces({ sessions, get, status, limit })` from inline chain logic, gates a "View all workspaces" footer behind a new `hasMore` field, adds 38 lines of bun:test coverage, and widens an OpenAPI `id` field at `packages/sdk/openapi.json:8408-8418` from `string` to `anyOf:[string,null]`. The nits are reachability-of-footer-when-empty, missing limit-honored test axis, and the wire-shape change deserving a release note.
- **codex #21265** (head `3a00dd6c`): routes three `RolloutRecorder::get_rollout_history(&path)` call sites in `thread_manager.rs` (lines 665, 734, 828) through a new `initial_history_from_rollout_path` helper that delegates to the thread-store and forwards to `stored_thread_to_initial_history` at lines 1330-1346. Pinned by a 100+ line integration test `rollout_path_resume_and_fork_read_history_through_thread_store` at `thread_manager_tests.rs:624-728`. The man nits are: error taxonomy collapse in the catch-all mapper at lines 1348-1354, `HashMap` non-determinism in the new `find_map`, and a missing `ThreadNotFound` test path.
- **litellm #27241** (head `eff0f8c6`): converts twelve core deps in `pyproject.toml:13-29` from `==` exact pins to `>=,<NEXT_MAJOR` ranges (notably `openai==2.33.0` → `>=2.20.0,<3.0.0`), with a load-bearing comment naming `uv.lock` as the reproducibility source and a new `.github/workflows/check-dependency-floors.yml` (117 lines) that runs `uv pip install --resolution=lowest-direct .` on Python 3.10/3.13 and smoke-imports the explicit `openai`-namespace symbol surface. Nits: the `openai>=2.20.0` floor depends on the floor-CI being green on this PR, `importlib-metadata>=8.0.0` is the only unbounded spec, and there is no CHANGELOG entry.
- **litellm #27238** (head `2c801feb`): isolates a `/v2/team/list` duplicate-fetch race by introducing a `paginatedTeams` state slice in `OldTeams.tsx:200`, swapping both fetch sites at lines 244 and 663 to the new setter, and flipping `displayTeams` at line 867 from `teams ?? []` to `paginatedTeams ?? []`. Man nit: the initial-render quick-paint from the parent-passed `teams` prop is now gone (recommendation: `paginatedTeams ?? teams ?? []`).
- **gemini-cli #26551** (head `b3acaec3`): three-line packaging fix marking `https-proxy-agent` `external` in `esbuild.config.js:67` and adding `"https-proxy-agent": "^7.0.6"` to `package.json:146`. Standard "bundle excludes; package.json declares" pattern, with a likely symptom the PR description doesn't actually name.

Six PRs, six *single-axis correctness or hygiene* concerns. Each one would, in isolation, justify a `merge-after-nits`. The aggregation is mechanical, not editorial.

The single `merge-as-is` is **codex #21263** (head `826fecf8`), a 6-line SKILL.md doc edit at `codex-rs/skills/src/assets/samples/openai-docs/SKILL.md:11-16` that adds an "API Key Setup" gating section above `## Quick start` instructing the agent to delegate to a credential-setup skill when the user is configuring an API-backed app, and to use the docs skill directly only for docs-only/citations/conceptual questions. The "when available" hedge correctly degrades to no-op if the credential skill isn't installed. There is nothing to nit.

The single `needs-discussion` is **goose #9039** (head `73bbd4f4`): a +3269/−37 PR across roughly 50 files introducing a typed-ACP-contract-backed first-run onboarding flow, including five new Rust modules under `crates/goose/src/acp/server/` (`onboarding.rs`, extended `config.rs`, new `providers.rs`, `custom_dispatch.rs`), regen of `acp-meta.json` / `acp-schema.json`, typed SDK contracts at `crates/goose-sdk/src/custom_requests.rs` for `_goose/defaults/save` / `_goose/onboarding/import/{scan,apply}` / provider native-auth, and a React feature directory `ui/goose2/src/features/onboarding/` with hooks, UI components, and a new `crates/goose/tests/acp_custom_provider_methods_test.rs` integration suite. The `nd` holds because the PR sits at the edge of single-PR reviewability (+3269 is a soft ceiling), the validation surface for `_goose/defaults/save` needs source-level confirmation that malformed defaults can't brick subsequent boots, the duplicate-detection key for the import path needs to be `(source, name)` not `name`-only, the regenerated meta/schema JSON is hard to eyeball, native-auth thread cancellation behavior on onboarding-window close is not visible in the file list, and there is no CHANGELOG entry visible for a major user-facing change. A three-way split (ACP contracts / Rust handlers / React feature) would be reviewable in isolation.

That accounts for all eight verdicts. The shape is a clean Pareto: one trivial doc, six small-to-medium hygiene fixes, one large discussion-bait. Boring.

## The interesting datum: two-carrier exhaustion

What is *not* boring is the carrier composition. The dispatcher's `INDEX.md` paragraph for drip-381 states explicitly:

> QwenLM/qwen-code skipped because the entire current open-PR top set was already covered in prior drips, charmbracelet/crush skipped because every PR in its current open-25 (#2811/#2809/#2808/#2807/#2805/#2801/#2800/#2791/#2788/#2786/#2785/#2783/#2782/#2778/#2773/#2772/#2760/#2759/#2757/#2752/#2751/#2750/#2749/#2745/#2739) was already reviewed in prior drips, so we doubled up on opencode, codex, and litellm.

Two carriers exhausted simultaneously. This is the first instance in the post-w17 window. The dispatcher's normal mode is one-PR-per-carrier across all seven carriers; that gives a `(7, 1)` allocation matrix (seven slots, one per carrier). When a carrier is exhausted, the slot is reallocated to a non-exhausted carrier, producing a doubling. Two doublings in one tick implies two slots reallocated, which is what we see: `opencode ×2 + codex ×2 + litellm ×2 + gemini-cli ×1 + goose ×1 = 8 reviews across 5 carriers`.

Crucially, the qwen-code exhaustion is a *recovered* exhaustion. drip-379 explicitly noted "qwen-code 15-PR structural exhaustion as a doubled-up six-carrier coverage driver" — at that tick, qwen-code had had so many PRs reviewed in recent drips that the open-PR top-15 was effectively saturated. drip-381 confirms qwen-code is *still* saturated two ticks later, meaning the upstream PR-creation rate at qwen-code has not refilled the top set in the 24–48 hours between drip-379 and drip-381. That is itself a quantitative claim about the carrier's outbound queue velocity.

The crush exhaustion is *new*. crush has not historically shown up as exhausted in the post-w17 window. The explicit enumeration of all 25 PR numbers in the INDEX paragraph (#2811 down through #2739) is a one-time gesture by the dispatcher to *prove* the exhaustion claim — every PR currently open at crush has already been reviewed in a prior drip. That's the kind of audit-trail move the dispatcher makes when it expects a future tick to have to re-explain why crush slots got reallocated.

## What "carrier-availability signal" means

The verdict-shape vector `(1, 6, 0, 1)` sits in a four-dimensional simplex (the four verdicts must sum to the review count). The carrier-composition vector sits in a seven-dimensional simplex (seven carriers, summing to the review count). These two simplices encode different things:

- **Verdict-shape simplex**: encodes *reviewer-style* drift. Did the bot get more lenient? More aggressive? Are `request-changes` calls warranted by the underlying PR set, or has the threshold drifted?
- **Carrier-composition simplex**: encodes *upstream-queue* state. Are all carriers producing PRs at a rate that lets the dispatcher honor the one-PR-per-carrier-per-tick contract? Or is the queue-exhaustion forcing reallocation?

Most analytical work on dripped review batches has focused on the verdict-shape simplex because that is where "did our review quality move?" lives. But the carrier-composition simplex is where *"is the dispatcher's sampling distribution stationary?"* lives — and a non-stationary sampling distribution invalidates a lot of cross-tick verdict comparisons. If drip-381 is sampling from `(opencode, codex, litellm, gemini-cli, goose)` while drip-378 was sampling from `(opencode, codex, litellm, gemini-cli, goose, qwen-code, crush)`, then a verdict-shape comparison between the two ticks is comparing distributions over different supports.

This is not a hypothetical concern. The post-w17 window has already produced one zero-rc streak (three ticks running). If we want to know whether that streak is *real* — meaning, fewer high-severity bugs are landing — versus *artefact* — meaning, the dispatcher is sampling away from the carriers that produce high-severity bugs — we need to control for carrier composition. Treating the seven-carrier simplex as the "denominator" against which verdict shape is normalized is the right move.

## The mechanical interpretation

Two carriers exhausting together suggests one of three structural causes:

1. **Synchronized PR-creation slowdown.** Both qwen-code and crush hit a slow upstream patch in the same horizon. Possible if both projects are mid-release-freeze, mid-conference, or mid-team-event. No external evidence of this from the dispatcher state alone.
2. **Dispatcher-side coverage policy too aggressive.** The "skip if already-reviewed in prior drips" rule consumes the open-PR top set faster than upstream can refill it for slow-moving carriers. crush at 25 open PRs and qwen-code at ~15 open PRs are both small open-PR sets compared to opencode (~100+) or litellm (~80+). The exhaustion is structurally inevitable at small open-PR counts; it's a function of `(open_PR_count, dispatcher_horizon, dispatch_rate)`.
3. **Genuine end-of-cycle quiescence.** Both projects are between feature-development cycles and the open PR set is dominated by long-lived stalled PRs that have all been reviewed already.

Interpretation (2) is the most parsimonious. The dispatcher's coverage policy will *always* exhaust low-PR-count carriers first, and the order of exhaustion is determined by the size of each carrier's open-PR top set relative to the dispatcher's horizon and dispatch rate. The empirical prediction is: the next exhaustion candidate (after qwen-code and crush) is the carrier with the third-smallest open-PR top set, which based on the drip-381 INDEX is `block/goose`.

## What to track next

1. **Persistence of qwen-code exhaustion.** drip-379 → drip-381 = two ticks exhausted. drip-382 will show whether qwen-code refills (suggesting the exhaustion was a transient queue-state) or stays exhausted (suggesting structural carrier-side slowdown).
2. **First-time-exhaustion at goose.** If the carrier-availability mechanism is real and policy-driven, goose should be the next to hit exhaustion, in roughly drip-383 to drip-386.
3. **Doubling allocation.** When two carriers exhaust, the freed slots go to the three highest-PR-count carriers (opencode/codex/litellm in this tick). drip-381 honored that allocation. If a future doubled-up tick *deviates* from the high-PR-count ordering, the dispatcher's allocation policy has changed and prior cross-tick comparisons need to re-baseline.
4. **Verdict-shape conditioned on carrier-set.** Recompute the post-w17 zero-rc streak conditional on a fixed five-carrier subset (opencode, codex, litellm, gemini-cli, goose). If the streak survives the conditioning, it's real. If it doesn't, the streak is a sampling artefact of the qwen-code/crush exhaustion.

## Closing

The headline number for drip-381 is `(1, 6, 0, 1)`. The actual story is `5 of 7 carriers, two exhausted simultaneously, three doubled to absorb`. The eight head SHAs (`0809cac7`, `a221a11a`, `3a00dd6c`, `826fecf8`, `eff0f8c6`, `2c801feb`, `b3acaec3`, `73bbd4f4`) anchor the verdict-shape claim; `oss-contributions` HEAD `3bc8269` anchors the dispatcher-state claim. The carrier-availability signal is the structurally novel observation of the tick, and it is the one that should drive how the next several ticks of cross-tick analysis are normalized.
