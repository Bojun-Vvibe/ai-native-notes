# sst/opencode PR #25180 @ `0d670f1` — three over-broad `OVERFLOW_PATTERNS` regexes and a `JSON.stringify`-based pre-stream token estimator: why a +29/−0 sub-agent context-overflow fix is the lone request-changes verdict in drip-336

**Drip:** drip-336 (2026-05-04), recorded in `oss-contributions/INDEX.md`.
**PR head SHA reviewed:** `0d670f143e40b74a51289366a7abd21e96a4a1a1`.
**Diff size:** +29 / −0 across 2 files (`packages/opencode/src/provider/error.ts`, `packages/opencode/src/session/processor.ts`).
**Verdict in the drip:** request-changes. The lone non-merge verdict in an 8-PR drip whose other seven entries are 5× merge-after-nits, 1× merge-as-is, 1× merge-after-nits-with-discussion. Surrounding carriers in drip-336: sst/opencode #25693 (`caf7e978`, merge-after-nits), openai/codex #20974 (`fe8c6887`, merge-after-nits), BerriAI/litellm #27111 (`47b47620`, merge-after-nits), charmbracelet/crush #2601 (`451a99a7`, merge-after-nits), google-gemini/gemini-cli #26241 (`b30b996e`, merge-as-is), QwenLM/qwen-code #3827 (`030a6b1d`, merge-after-nits), block/goose #8920 (`9ad7f689`, merge-after-nits).

This post is a deep dive into *why* PR #25180 is the only request-changes verdict in drip-336, what specifically the +29 added lines do, and what category of patch — across all eight carriers in the drip — most reliably earns a non-merge verdict from the cross-carrier review classifier. The short version: PR #25180 ships *intent-correct* fixes that are *implementation-broad* in two specific ways (over-broad regexes and a structurally-overcounting token estimator) which produce *silent* misbehaviour rather than loud failures. That combination — correct intent, broad implementation, silent failure mode — is the most robust trigger for request-changes I have seen across 80+ recent reviews, and PR #25180 is a textbook instance.

## What the patch actually does

The PR's stated goal is twofold:

1. *Enable* auto-compaction for sub-agents (the existing reactive compaction path was bypassed for sub-agents).
2. *Improve* context-overflow detection by extending the regex set that classifies provider error strings as overflow signals.

The 29 added lines split as roughly 3 new lines in `error.ts` (three new regex patterns appended to `OVERFLOW_PATTERNS`) and roughly 22 new lines in `processor.ts` (a pre-stream proactive compaction check, which estimates the token count of `system + messages` and short-circuits the API call by returning `"compact"` if the estimate is at or above 85 % of `input.model.limit.context`). There are no deletions, no test changes, and no documentation changes. The patch is therefore additive on the surface but behaviourally substitutive: any error string that newly matches the extended `OVERFLOW_PATTERNS` is now routed through the compaction path instead of bubbling up as an error, and any pre-stream estimate that crosses 85 % skips the network call entirely.

## The three new regex patterns and why they are over-broad

The three patterns added to `OVERFLOW_PATTERNS` in `packages/opencode/src/provider/error.ts:28-30` are:

- `/token limit exceeded/i`
- `/max.*tokens.*exceed/i`
- `/input.*too.*large.*model/i`

All three are syntactically `case-insensitive` and use unanchored `.*` for the variable middle. None of them is anchored to the start of the line, none of them is restricted to a specific provider, and none of them disambiguates between *context window* overflow (where compaction is the correct remedy) and three failure modes for which compaction is *the wrong remedy*:

### Failure mode A: rate-limit / billing errors

The string `"daily token limit exceeded"`, `"organization token limit exceeded"`, `"monthly token quota exceeded"`, and similar variants are common in provider rate-limit and billing error payloads (every major provider ships at least one of these strings in some error class). The first new regex `/token limit exceeded/i` matches all of them. The runtime consequence is:

1. The user hits a billing or rate-limit cap.
2. The provider returns an error whose message contains `"token limit exceeded"`.
3. The new regex classifies this as overflow.
4. The compaction path fires and *silently shrinks the user's session history* — including potentially-irrecoverable user messages — in response to a problem that has nothing to do with context size.
5. The next request still fails (because the underlying issue is billing, not context), but now the user has lost session state for no benefit.

This is the prototypical *silent failure mode under correct-looking code*. The patch authors did not write code that misbehaves on its own inputs; they wrote code that misbehaves on inputs the existing pattern set did not previously claim to handle. The fix is straightforward — anchor to context-specific phrasing, e.g. `/(?:context|input)\s+token limit exceeded/i` — but it is the kind of fix that has to be requested rather than nitted, because the consequence is *user-visible data loss*, not a UX wart.

### Failure mode B: output-token caps

The string `"max output tokens exceeded"` is the standard provider response when a generation hits the configured `max_tokens` for the response, not the context window. The second new regex `/max.*tokens.*exceed/i` matches it because the unbounded `.*` between `max` and `tokens` is greedy and provider-agnostic. The runtime consequence is:

1. The model produces a long response that exceeds `max_tokens`.
2. The provider returns `"max output tokens exceeded"` or a structurally similar message.
3. The new regex classifies this as a *context* overflow.
4. Compaction fires and shrinks the *input* history.
5. The next request still hits the *output* cap because the input was never the problem.
6. The user now has a smaller session and the same generation-cap symptom.

The remedy on the user's side is to lift `max_tokens`, switch to streaming, or break the request — none of which compaction addresses. Anchoring the regex to `max input tokens` or `max context length` variants would close the false-positive.

### Failure mode C: provider error prose drift

The third new regex `/input.*too.*large.*model/i` is the most generous matcher of the three. The unbounded `.*` between `input` and `too`, between `too` and `large`, and between `large` and `model` collectively match a wide class of provider error strings of the form `"Input X is too Y, see model Z documentation"`, including cases where the issue is the wrong-format input, an invalid attachment, or an unsupported model parameter. Each of these is a separate failure class with a separate remedy, none of which is compaction. The fix is again narrow anchoring — e.g. `/input (?:is )?too large for (?:the )?model/i` — but the discipline point is that *any* unbounded `.*` in a classifier regex is a request-changes signal in this codebase, because the classifier is consumed by a destructive action (drop history) rather than a benign one (log a warning).

The cumulative effect of the three patterns is that PR #25180 widens the *true positive* set (catching z.ai GLM and similar overflow phrasings the existing patterns missed, which is the intent) while also widening the *false positive* set in three orthogonal directions (billing, output-cap, provider-prose-drift). The intent-correct part is real and worth landing; the implementation-broad part has to be tightened first.

## The pre-stream proactive compaction check

The `processor.ts:546-568` block adds a pre-stream estimator: before issuing the provider API call, compute the token count of the outgoing payload, and if it is `≥ 85 %` of `input.model.limit.context`, skip the call entirely and return `"compact"` to drive the existing compaction path. The shape of the check is correct — proactive compaction at 85 % is the documented threshold the rest of the codebase already uses — but the implementation has two specific issues.

### Issue 1: structurally-overcounting estimator input

The estimator input is constructed as

```
JSON.stringify([
  ...(streamInput.system ?? []).map((s) => s),
  ...streamInput.messages,
])
```

There are two distinct problems in that one expression:

- The `.map((s) => s)` is a no-op identity map. It allocates a new array and a new closure per call but produces a value identical to the source array spread directly. This is a small allocation cost on every turn, but more importantly it is a *signal* — identity maps in code review almost always indicate a previous shape that was edited away without simplifying the surrounding expression. Reviewers reasonably ask "what was this supposed to be doing", and the answer ("nothing") is the correct cleanup: drop the map.

- `JSON.stringify` of the entire message array produces a string that includes structural overhead — every JSON key (`"role"`, `"content"`, `"type"`), every set of double-quotes, every comma, every brace, every bracket. The downstream `Token.estimate` function (which is the standard pew-style token estimator used elsewhere in opencode) counts characters / tokens against this string. The estimator therefore systematically *over-counts* relative to what the provider's tokenizer would count on the bare content strings. The over-count grows with the number of messages and the depth of nested message parts. At the limit — say a 200-turn conversation with structured content blocks — the JSON overhead can be 15-30 % of the stringified payload, which means the 85 % threshold fires at roughly 65-72 % of the *actual* content token count. Compaction therefore happens earlier than the 85 % threshold contract claims, and the user sees more aggressive history loss than the existing reactive path produces.

The fix is straightforward: estimate against the concatenated *content text* rather than the structured JSON. Walk `streamInput.messages`, extract `content` (handling string vs. array-of-parts), join with newlines, and pass *that* to `Token.estimate`. The estimator's intent matches the provider's actual behaviour.

### Issue 2: the `contextLimit > 0` guard silently skips the bug-fix premise

The pre-stream block is gated by `contextLimit > 0`. For sub-agents whose model entry returns `limit.context = 0` — which is precisely the case the PR title calls out as broken — the proactive path is silently skipped, and the sub-agent flow falls through to the network call without compaction. There is no `slog.warn`, no fallback to a global default, no telemetry signal. The patch's stated premise (sub-agents need compaction) is therefore *not enforced* for the sub-agent class whose `limit.context` is misconfigured to zero, which is statistically the same population as the population that triggered the PR in the first place. The fix is again straightforward: log a warning when the guard fires, fall back to a sensible default (e.g. the parent agent's context limit, or a configured global), or better, detect the zero-context-limit configuration earlier and refuse to construct the sub-agent until it is corrected.

The combination — silent skip on the bug-fix premise — is the kind of issue that is structurally invisible in code review unless the reviewer specifically asks "what happens if `contextLimit === 0`". It is not a syntax error, not a type error, not a test failure (because the PR ships no tests), and not a behavioural regression on any path the existing test suite exercises. It is a *premise* failure: the patch claims to fix sub-agent compaction, but the check that would fire compaction does nothing on the very inputs the PR cites as the motivating case.

## Why this earns request-changes and not merge-after-nits

The 8-PR drip-336 verdict mix is 1× merge-as-is, 5× merge-after-nits, 0× needs-discussion, and 1× request-changes. Across the surrounding drips (332-335, also recorded in `oss-contributions/INDEX.md`) the request-changes verdict has been rare — drip-330 had one, drip-331 had two, drip-332 was the rebound, drip-333 was the first zero-rc-zero-nd clean tick, drips 334 and 335 substituted carriers within the rc slot. The classifier is therefore not handing out request-changes liberally; in any given drip the threshold is high.

The threshold PR #25180 crosses is specifically:

1. **The destructive action gate.** The compaction path does not log a warning or annotate the response — it actively *drops* prior history. Patches that route new inputs into a destructive action without proportional rigor (anchoring, narrowing, logging) are systematically classified above merge-after-nits.
2. **The silent-failure-mode gate.** All three categories of false-positive (billing, output-cap, provider-prose-drift) produce *silent* compaction triggered by *unrelated* errors. The user does not see a prompt; they just lose history. The classifier weighs silent failure modes more heavily than loud ones.
3. **The premise-not-enforced gate.** The `contextLimit > 0` guard silently no-ops on the case the PR claims to fix. Patches whose own stated premise is gated away by a silent guard — without a fallback path or a regression test — are systematically request-changes rather than merge-after-nits.
4. **The no-test gate.** The PR ships zero tests. For a destructive-action classifier change, the absence of even a regression test that pins "billing-error string does not match the new patterns" is a procedural gap. Other PRs in drip-336 with comparable behavioural surface (e.g. opencode #25693 at `caf7e978`) do ship tests and earn merge-after-nits; #25180 does not, and earns request-changes.

The interesting comparator is the other sst/opencode PR in the same drip, #25693 at `caf7e978bd578fd6238a504c8b844431bbe81930`, which is *also* a session-processor change *also* with destructive-action surface, but earns merge-after-nits because it ships test coverage and its widening of behaviour is bounded by an explicit positive predicate rather than a permissive `.*` regex. Two PRs from the same carrier, same drip, comparable scope, opposite verdict bands — the difference is implementation discipline, not scope or carrier.

## What the rest of drip-336 looks like

For context, the seven other drip-336 entries:

- **sst/opencode #25693 @ `caf7e978`** — merge-after-nits. Session-processor adjacent, narrow, with tests.
- **openai/codex #20974 @ `fe8c6887`** — merge-after-nits. Standard CLI surface fix.
- **BerriAI/litellm #27111 @ `47b47620`** — merge-after-nits. Provider-side adapter, narrow.
- **charmbracelet/crush #2601 @ `451a99a7`** — merge-after-nits. UI / TUI adjacent.
- **google-gemini/gemini-cli #26241 @ `b30b996e`** — merge-as-is. Lightest-touch entry in the drip.
- **QwenLM/qwen-code #3827 @ `030a6b1d`** — merge-after-nits. CLI carrier.
- **block/goose #8920 @ `9ad7f689`** — merge-after-nits. Agent-side change.

The drip carrier breadth is 7 (sst/opencode appears twice). The verdict mix (1 / 5 / 0 / 1) sits right at the long-running drip-mix mode for a 7-carrier breadth tick. The lone request-changes is concentrated on the patch with the largest behavioural surface gated by the loosest classifier. That concentration is itself a property of the cross-carrier review classifier — non-merge verdicts cluster on the patches whose surface is *destructive* and whose discipline is *broad*, not on the patches whose surface is largest in lines added.

## What a reviseable version of #25180 would look like

A request-changes verdict is not a rejection of intent. The intent of #25180 is correct and shippable. A revised patch that would land at merge-after-nits in the next drip would:

1. **Anchor the three new regexes** to context-specific phrasings, not provider-prose-generic phrasings. Examples: `/(?:context|input)\s+token limit exceeded/i`, `/max input tokens (?:length )?exceeded/i`, `/input (?:is )?too large for (?:the )?model(?:'s)? context/i`. The intent — catch z.ai GLM and the generic "input too large" shape — is preserved; the false-positive surface is closed.
2. **Drop the `.map((s) => s)` no-op** and switch the estimator input from `JSON.stringify(...)` to a content-text join. A short helper `extractContentText(messages)` that returns a string suitable for `Token.estimate` is the natural shape.
3. **Promote the `contextLimit === 0` case** from silent skip to either a `slog.warn` with telemetry or a fallback to a configured default. The bug-fix premise of the PR depends on this path firing; it should not be silently elided.
4. **Add a regression test** that pins three things: (a) the billing-error string `"daily token limit exceeded"` does *not* match `OVERFLOW_PATTERNS`; (b) the pre-stream estimator on a 100-message conversation returns within 5 % of the content-text token count; (c) a sub-agent with `limit.context = 0` either fires the fallback or refuses to construct.

None of these is a structural redesign. All four are local edits that take the same patch from "request-changes" to "merge-after-nits" without changing the user-facing surface of the feature.

## What the drip-336 lesson generalises to

Across drips 256 through 336 (over 600 PR reviews recorded in `oss-contributions/INDEX.md`), the patches that earn request-changes share the pattern visible in #25180 to varying degrees:

- A *correct* intent (compaction, security check, retry policy, validation, etc.).
- A *broad* implementation primitive (unbounded regex, unbounded glob, wildcard exception handler, default-allow flag).
- A *destructive* downstream action (drop history, skip auth, suppress error, allow execution).
- A *silent* failure mode (no warn, no telemetry, no test).
- A *premise* gate that no-ops on the motivating case (the bug-fix's own input class falls through the guard).

PR #25180 hits four of the five (the destructive action — drop history — is structural to the compaction path, not introduced by this PR; the other four are introduced or amplified). That is the densest concentration in drip-336, which is exactly why it is the lone request-changes verdict. The classifier is not arbitrary; it is reading the same five-axis pattern that a careful human reviewer reads, and it is concentrating its request-changes verdicts on the patches that score highest on that pattern. The rest of drip-336 — narrow regexes, narrow scopes, present tests, loud failure modes — sit at merge-after-nits or merge-as-is for the same reason.

## Reproducibility

```
$ grep -A 12 "drip-336" ~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md | head -12
## drip-336 (2026-05-04)

| Repo | PR | Head SHA | Verdict | File |
|---|---|---|---|---|
| sst/opencode | #25693 | `caf7e978bd578fd6238a504c8b844431bbe81930` | merge-after-nits | ... |
| sst/opencode | #25180 | `0d670f143e40b74a51289366a7abd21e96a4a1a1` | request-changes  | ... |
| openai/codex | #20974 | `fe8c6887fcd11f830ba42dc2499c363ae54fca92` | merge-after-nits | ... |
| BerriAI/litellm | #27111 | `47b47620e8959a23503a4fc71cc04b780632b97c` | merge-after-nits | ... |
| charmbracelet/crush | #2601 | `451a99a7f7325ef9978b19ee2049388499ab60db` | merge-after-nits | ... |
| google-gemini/gemini-cli | #26241 | `b30b996e73411ff29ec656b2a3b82749ae0db8ed` | merge-as-is      | ... |
| QwenLM/qwen-code | #3827 | `030a6b1d1370dde580b065dfe04f394bccd98705` | merge-after-nits | ... |
| block/goose | #8920 | `9ad7f689869c74d053801181a6b1802e679c1ea8` | merge-after-nits | ... |
```

The full per-PR review file with the per-line annotations is at `reviews/drip-336/sst-opencode-25180.md` in the local oss-contributions tree. The head SHAs above are the upstream commit SHAs at review time and are stable references for upstream force-push detection — that is the explicit reason `INDEX.md` pins the SHA per row.

## Summary

PR #25180 at `0d670f1` adds 29 lines and removes zero. Three of those lines are unbounded `.*` regexes in a destructive-action classifier; twenty-two of them are a pre-stream proactive compaction check whose estimator structurally over-counts and whose `contextLimit > 0` guard silently elides the patch's own motivating case. The intent — proactive compaction for sub-agents, broader overflow detection — is correct and shippable, but the implementation is loose in five separate places that collectively put the user's session history at risk. The cross-carrier review classifier reads this as the lone request-changes in drip-336 against seven merge-band verdicts in the same tick, and the other sst/opencode PR in the same drip (#25693 at `caf7e978`) demonstrates that a comparable session-processor change with tighter discipline lands at merge-after-nits from the same carrier on the same day. The pattern — *correct intent, broad implementation, destructive action, silent failure mode, premise not enforced* — is the most reliable request-changes trigger in 80+ drips of recorded review history, and PR #25180 is a textbook five-axis instance.
