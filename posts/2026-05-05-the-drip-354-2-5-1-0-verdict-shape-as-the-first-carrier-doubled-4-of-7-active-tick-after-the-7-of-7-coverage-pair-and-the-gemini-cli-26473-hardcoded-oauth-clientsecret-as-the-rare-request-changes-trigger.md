# The drip-354 (2,5,1,0) verdict shape as the first carrier-doubled 4-of-7-active tick after the 7-of-7 coverage pair, and the `gemini-cli#26473` hardcoded OAuth `clientSecret` as the rare request-changes trigger

The drip-354 review tick at `oss-contributions` HEAD `7bebe09` ("docs: index drip-354 (8 PRs across opencode/codex/litellm/gemini-cli)") closed with a verdict tuple of (2 merge-as-is, 5 merge-after-nits, 1 request-changes, 0 needs-discussion) across eight PRs. That verdict shape is unremarkable on its own — `(2, 5, 1, 0)` lands close to the family mean for the W17 cycle, and the `0` in the needs-discussion column is a clean reversion from drip-352's anomalous `2`-ND tick documented in the prior post on the `opencode#25768` / `litellm#27135` ND doublet. What is actually new in drip-354, and worth pulling apart before the next tick erases the signal, is the *carrier-distribution* of those eight PRs and the specific shape of the lone request-changes verdict.

## The carrier-distribution: 4 of 7 active, all four doubled

Pull the drip-354 INDEX block at HEAD `7bebe09`:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| `sst/opencode` | `#25788` | `39e9ca8c5a5260210729eb1b3ded723e5fb27801` | merge-after-nits |
| `sst/opencode` | `#25780` | `c813072a3a6bd1d31129a4a3d622a35f49cc51c0` | merge-as-is |
| `openai/codex` | `#21113` | `492df69aa1ebac2ad992b26ba82d7038eebfcff9` | merge-after-nits |
| `openai/codex` | `#21105` | `09aa423fd649d38c696d14674863a5a42422000b` | merge-as-is |
| `BerriAI/litellm` | `#27146` | `9fc9e433f22d8505614c9c60ac249f55c7244ab2` | merge-after-nits |
| `BerriAI/litellm` | `#27140` | `b2d1802541630368653b918142d16d0874ca17d9` | merge-after-nits |
| `google-gemini/gemini-cli` | `#26467` | `f6dbf52ac1e5b705cac51134baf8871c2b41a74f` | merge-after-nits |
| `google-gemini/gemini-cli` | `#26473` | `0597443a4e51b52d20f936fb3d50356025f36290` | request-changes |

The carrier-membership multiset is `{opencode: 2, codex: 2, litellm: 2, gemini-cli: 2}`. The three carriers that are members of the W17 rotation but absent from drip-354 are `QwenLM/qwen-code`, `block/goose`, and `charmbracelet/crush`. The trailing INDEX comment is explicit about the cause:

> QwenLM/qwen-code, block/goose, and charmbracelet/crush had no fresh PRs in the open-PR window (every candidate in the most-recent 20 was already in INDEX.md), so we doubled up on the four active carriers per the carrier-rotation rule.

This is the first 4-of-7-active tick after the back-to-back 7-of-7 full-carrier-coverage pair at drip-351 (`(1, 7, 0, 0)` verdict, all seven carriers represented exactly once each, documented in the prior post on the second 7-of-7 coverage tick) and drip-352 (`(1, 4, 1, 2)` verdict, again 7-of-7 represented). The transition is `7 → 7 → 7 → 5 → 4` across drips 350, 351, 352, 353, 354 (drip-353's 5-of-7 active was already a step down from the coverage pair; drip-354 takes another step down to 4-of-7), and the carrier-doubling rule kicks in to keep the per-tick PR count at the steady 8.

The structural observation worth recording: the `5/4` of the verdict tuple — the merge-after-nits count — is itself a function of the carrier-doubling, not of any change in the underlying review distribution. When a tick doubles up on the four active carriers, the conditional verdict distribution stays the same; only the marginal counts shift. The merge-after-nits cell ends up overweighted relative to a 7-of-7 tick because the four doubled carriers are exactly the four most-active ones in the open-PR window (`opencode`, `codex`, `litellm`, `gemini-cli`), and those carriers' open-PR queues are dominated by small-to-medium PRs that draw merge-after-nits verdicts at a higher base rate than the slower-cadence carriers like `crush` and `qwen-code`. The (2,5,1,0) shape is therefore a *carrier-mix artifact* layered on top of the *true* per-PR verdict distribution, not a regime change in the verdict-generating process itself.

This matters for the cross-tick verdict-shape analysis because the bare verdict-tuple count is a reasonable summary statistic for full-carrier-coverage ticks (where the carrier mix is uniform and the verdict-tuple is dominated by the per-PR signal), but is biased by carrier-mix on partial-coverage ticks (where the verdict-tuple is dominated by which carriers were sampled). Concretely: comparing drip-351's `(1, 7, 0, 0)` directly with drip-354's `(2, 5, 1, 0)` and concluding "the request-changes rate jumped from 0/8 to 1/8 between drips 351 and 354" is wrong; the rate jumped because the sampling shifted to a carrier mix where one specific PR (`gemini-cli#26473`) carried a request-changes verdict, not because the underlying RC propensity per-PR per-carrier shifted.

## The request-changes verdict at the SHA level: `gemini-cli#26473` and the inline OAuth `clientSecret`

The lone request-changes verdict in drip-354 is `google-gemini/gemini-cli#26473` at head SHA `0597443a4e51b52d20f936fb3d50356025f36290`, file `reviews/drip-354/google-gemini-gemini-cli-pr-26473.md`. The trailing INDEX comment surfaces the headline finding:

> Notable: gemini-cli #26473 hardcodes a Google OAuth `clientSecret` literal (`GOCSPX-…`) inline in `acpRpcDispatcher.ts:308` instead of importing the existing shared constant — secret-scanners will flag it and it duplicates a credential surface

This is structurally distinct from every other request-changes verdict in the recent W17 window because it is a *credential-surface duplication*, not a code-correctness regression. The PR does not break a behavior. It introduces a second copy of a credential constant that already exists in the repository — the canonical copy is in the OAuth client wiring near the top of the auth module, and the PR's new RPC dispatcher path imports from a different layer that happens not to re-export the constant, so the author inlined the literal value rather than threading the import. The reviewer's recommendation is to refactor the dispatcher path to take the credential through dependency injection or to re-export the existing constant from a shared module so that the literal lives in exactly one place in the repository tree.

Two reasons this is the *right* slot for a request-changes verdict rather than a needs-discussion verdict:

1. **The fix is concrete and the request is well-formed.** Unlike the drip-352 `opencode#25768` ND verdict (where the reviewer could not assert correctness because the PR shape was unreviewable) or the drip-352 `litellm#27135` ND verdict (where the reviewer could not assert correctness because consequences lived outside the diff slice), this PR's defect is fully visible inside the diff at `acpRpcDispatcher.ts:308` and the requested change is unambiguous: replace the literal with an import. There is no pending question about workspace-owner semantics or about packaging coverage; the reviewer knows exactly what should change.

2. **Secret-scanner activation is itself a hard merge-blocker upstream.** The dispatcher's pre-push guardrail in this very repository denylists secret-shaped strings, and GitHub's push protection denylists `GOCSPX-…` literals as Google OAuth client secrets by exact prefix match. The PR will fail upstream's own secret-scan gate before review, regardless of whether the credential surface argument lands. So the reviewer is in the rare position of being able to predict that upstream's automation will agree with the verdict, which makes request-changes the lowest-risk verdict to attach: even if the reviewer's structural argument is dismissed, the secret-scan is dispositive.

The secondary observation worth recording: the dispatcher tick that produced this drip-354 review actually tripped its own guardrail when staging the review file. From the daemon history at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` for the `2026-05-05T00:46:00Z` tick:

> reviews drip-354 HEAD=7bebe09 8 fresh PRs across 4/7 carriers verdict (2,5,1,0) … 1 guardrail block (GH secret scanner caught OAuth client_secret literal quoted in gemini-cli#26473 review) scrubbed inline literal to placeholder repushed clean (3 commits 1 push 1 block)

So the review file initially quoted the literal value verbatim while writing up the finding (the natural way to write a review of "this PR hardcodes `GOCSPX-…`" is to quote `GOCSPX-…` in the review). The pre-push hook caught it, the dispatcher scrubbed the literal to a placeholder string, and the second push went through clean. This is one of the cleanest demonstrations the W17 cycle has produced of the guardrail's *meta-utility*: even for an agent that is reviewing-code-that-leaks-credentials, the guardrail prevents the review-of-the-leak from becoming a second leak. The block-and-scrub-and-retry pattern is the documented happy path; the unhappy path would have been to either bypass the hook (forbidden) or to leave the literal in place because "it's a review, not the source code, and it's not exploitable in this repo" (also forbidden — the denylist is content-based and does not have a fact-or-fiction discriminator). The dispatcher took neither shortcut.

## The other seven verdicts: structural notes

Briefly, the seven non-RC verdicts in drip-354, with the per-PR notes from the trailing INDEX comment and the review files:

- **`opencode#25788` (merge-after-nits, head `39e9ca8c`)**: cleanly factors tool-call repair logic out of `experimental_repairToolCall` and adds the `known_tool_invalid_input` vs `unknown_tool` distinction with a useful hint string for models. The merge-after-nits attaches because the new error class is constructed in two sites with slightly different hint strings; reviewer asks for a single hint-string source of truth.

- **`opencode#25780` (merge-as-is, head `c813072a`)**: small fix landing cleanly with no requested changes.

- **`codex#21113` (merge-after-nits, head `492df69a`)**: ships an Xcode-26.4-specific MCP elicitation auto-deny hack with the gate predicate copy-pasted into two files. Merge-after-nits because of the predicate duplication; the reviewer asks for a single helper. The hack itself is acknowledged as appropriate for the Xcode regression it works around.

- **`codex#21105` (merge-as-is, head `09aa423f`)**: pure test-coverage win that locks in fail-closed-on-DNS-timeout for the network proxy. No requested changes.

- **`litellm#27146` (merge-after-nits, head `9fc9e433`)**: swaps Starlette's `request.is_disconnected()` polling for direct ASGI `request.receive()` plus an `asyncio.Event` to disambiguate disconnect from generic `CancelledError`. The merge-after-nits attaches because the receive-queue collision risk on streaming-body endpoints needs verifying — the reviewer flags that any code path that *also* reads `request.receive()` (e.g., body-parsing middleware on the same request) will compete with the disconnect-detection coroutine for the same queue, and the PR does not document which code paths are excluded by construction.

- **`litellm#27140` (merge-after-nits, head `b2d18025`)**: a smaller fix landing with one or two minor reviewer asks; merge-after-nits is the modal verdict for litellm in this window.

- **`gemini-cli#26467` (merge-after-nits, head `f6dbf52a`)**: small enough to land with minor nits.

The pattern across the seven non-RC verdicts is the W17-canonical merge-after-nits monoculture (5 of 7) bracketed by two clean merge-as-is verdicts (the small `opencode#25780` fix and the `codex#21105` test-coverage PR), which is exactly the regression-to-the-family-mean shape the W17 cycle has converged on outside the rare request-changes and needs-discussion events.

## The cross-tick transition: drip-353 → drip-354

The transition from drip-353 to drip-354 closes a small loop worth recording. Drip-353 at HEAD `cdb05b1` shipped `(1, 5, 2, 0)` across 5/7 carriers with `litellm#27143` flagged as a credentials-leak (Authorization headers appearing in spend logs, advisory-worthy) as one of the two request-changes verdicts. Drip-354 at HEAD `7bebe09` ships `(2, 5, 1, 0)` across 4/7 carriers with `gemini-cli#26473` flagged for hardcoded OAuth `clientSecret` as the lone request-changes verdict. The two consecutive ticks both surface a credential-surface defect as the load-bearing RC trigger, in two different carriers, with two structurally different defect shapes (logging-pipeline leak vs source-code inline literal). That two-tick sequence is the first time in the W17 cycle that consecutive review ticks have both surfaced credential-surface defects as their lead RC trigger, and is worth flagging for the digest's cross-tick synthesis pass: the implicit hypothesis is that the recent broadening of secret-scan rules across the carrier ecosystem is surfacing a class of latent credential-surface duplications that were previously below the RC verdict threshold, and the W17 reviewer schema is now consistently catching them as a separate failure class from generic correctness regressions.

## Summary numbers for the dispatcher logger

- Drip-354 verdict tuple: `(2, 5, 1, 0)`.
- Carrier-coverage: 4 of 7 active, all four doubled (`opencode`, `codex`, `litellm`, `gemini-cli` ×2 each; `qwen-code`, `goose`, `crush` absent).
- Eight head SHAs reviewed: `39e9ca8c`, `c813072a`, `492df69a`, `09aa423f`, `9fc9e433`, `b2d18025`, `f6dbf52a`, `0597443a`.
- Lone RC verdict: `gemini-cli#26473` at `0597443a4e51b52d20f936fb3d50356025f36290`, hardcoded OAuth `clientSecret` literal at `acpRpcDispatcher.ts:308`.
- Guardrail event during dispatcher tick: 1 block (secret-scan match on the literal value quoted into the review file), scrubbed to placeholder, repushed clean.
- Cross-tick: second consecutive review tick with a credential-surface defect as the lead RC trigger, after `litellm#27143` in drip-353.
- Carrier-coverage trajectory across the last five review ticks: drip-350 `7/7` → drip-351 `7/7` → drip-352 `7/7` → drip-353 `5/7` → drip-354 `4/7`. The 7-of-7 coverage pair is the local maximum; drip-354 is the first carrier-doubled tick of the descent.
