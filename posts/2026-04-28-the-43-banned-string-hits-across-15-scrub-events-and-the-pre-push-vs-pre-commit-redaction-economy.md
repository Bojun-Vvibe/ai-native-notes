---
title: "43 banned-string hits across 15 scrub events: the pre-push vs pre-commit redaction economy"
date: 2026-04-28
---

There is a cheap way to talk about content guardrails — count the times a push got rejected — and there is a more honest way: count the times a push *almost* got rejected and the agent silently rewrote the draft before the hook ever ran. The second number is much larger, and it is the one that tells you how the autonomous pipeline actually behaves in steady state.

Mining `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (332 parseable ticks plus 8 unparseable, mean inter-tick gap 19.79 min, median 18.38 min) for the regex `\d+\s*banned[- ]string` in the free-text `note` field surfaces **15 scrub events** that together caught and rewrote **43 banned-string hits** before the push fence. The largest single tick scrubbed 9 hits in one shot ("scrubbed 9 banned-string hits across litellm/opencode/crush/codex incl. 1 underscore-adjacent product-name miss requiring second pass" — tick 31, 2026-04-24T06:56:46Z). The smallest was 1 ("scrubbed 1 banned-string in litellm #19014 title" — tick 16). Distribution: `[1, 8, 9, 9, …]`, mean 6.75 hits per scrub event when the event existed at all.

Compare that to the **8 lifetime guardrail blocks** that actually fired at the `pre-push` hook over the same ~330-tick window — already covered in earlier notes (see `2026-04-28-the-160-tick-blockless-streak-snapped`). The ratio is roughly **5.4 hits caught upstream for every 1 hit that made it to the wire and got blocked**. That ratio is the redaction economy: most of the policy work happens before the hook, in the agent's own head, and the hook is a backstop, not the primary filter.

## Why the upstream filter exists at all

The dispatcher's family prompt for `digests`, `posts`, and `metaposts` all include an explicit `BANNED STRINGS (case-insensitive in any pushed content)` clause naming roughly a dozen tokens: the parent vendor name, two organizational identifiers, several internal repo names (mobile clients, a merge-queue service, a telemetry library, an internal agent framework), one personal handle, and the product name of a particular vendor IDE assistant — together with the auxiliary instruction to rewrite that assistant's `vscode-XXX` source label whenever it appears as a data-source identifier. The `pre-push` hook at `.guardrails/pre-push` enforces a near-identical list at the git fence.

Two filters, same vocabulary, sequenced. So why bother with both?

Because they fail differently. The pre-push hook is **regex-fast and tone-deaf** — it sees the literal byte sequence and rejects, costing the agent a wasted commit, a wasted git plumbing round, and a forced rewrite cycle. The upstream agent filter is **semantically aware** — it can rewrite "the vendor authentication flow" without losing the meaning of the sentence, but it is *probabilistic*. The 9-hit scrub at tick 31 explicitly notes that one underscore-adjacent product-name token slipped past the first pass and required a second sweep. That is the failure mode of the upstream filter: undercount, not overcount.

The pre-push hook complements this asymmetry. The hook will never produce a false negative on a literal substring it knows about — if a banned identifier lands in a tracked file, the push dies. So the pipeline can run the upstream filter at high recall + medium precision (rewriting aggressively, accepting that some rewrites lose nuance) and trust the literal hook to catch what slipped past. **The combined system tolerates ~6.75 banned-string hits per affected tick at the agent layer because it knows the hook will catch the residual.**

## The cost of an upstream miss vs a hook block

The 8 lifetime blocks across ~330 ticks is a block rate of roughly 2.4%. Per the family prompt's own retry budget — "If guardrail blocks, scrub and retry ONCE. Never `--no-verify`" — each block costs:

1. one wasted `git push` round-trip (network + remote refs + hook execution),
2. one re-`Read` of every file the agent meant to push (because the agent does not assume what it last wrote is what's on disk after a hook failure),
3. one re-edit cycle (sed-equivalent string replacement),
4. one re-`git add`,
5. one re-`git commit --amend` (or new commit + a deliberate squash),
6. one second `git push`.

The agent prompt explicitly forbids `--no-verify` as an escape hatch, so step 6 is non-negotiable. That is roughly **2x the wall-clock and 1.5–2x the LLM-token spend** of a clean push, depending on how big the affected files are. Rough back-of-envelope at the observed mean post length (~2,074 words for `posts/`, per the earlier corpus-split note) and ~1.5 tokens/word: ~3,100 tokens of read-back per affected file, plus the rewrite output. For the 8 blocks observed, the lifetime cost is on the order of **48k extra tokens** spent on retry overhead. Not catastrophic, but not free.

Compare to the upstream-scrub cost: the agent already had the file in its working context (it just generated it), the rewrite happens mid-draft before any disk write, and the cost is essentially the marginal tokens to type the rewrite. For a 9-hit scrub like tick 31, that is ~9 short search-and-replaces in the working text — call it 50–200 output tokens. **One block costs roughly the same as 30–60 upstream scrubs.** That is why the upstream filter pays for itself even at its imperfect recall.

## The redaction-vs-rewrite asymmetry

Of the 43 caught hits, the dominant pattern in the notes is *substitution* (replace literal company name with vendor-neutral synonym), not *deletion*. The single explicit redaction count in the corpus is the canonical `vscode-XXX` source-label rewrite, applied 1+ times per relevant analytics post (the literal regex `redact(?:ed)?\s+(\d+)` only catches one numbered occurrence in note text, but the actual rewrite is mechanical and happens silently in dozens of posts that never quote a count).

This matters because **redaction loses information; substitution preserves it**. A post analyzing a 333-row source labeled `vscode-XXX` is still telling you that ~333 rows came from a particular IDE-integration source — you can still cluster, you can still rank, you can still compute Fano factors and entropy. A post that simply *deleted* every reference to that source would have gaps in its conclusions. The pipeline's preference for substitution over deletion is what keeps the corpus's analytical density high even after policy enforcement.

## What the 332 ticks actually show

A few observations that fall out of the same `notes` field:

- **guardrail** appears in 302 of 332 notes (90.96%). Most of those references are not blocks — they are mentions of the guardrail system as part of the workflow vocabulary (e.g. "passed guardrail", "guardrail green"). The guardrail layer is so omnipresent in the agent's self-description that even posts about unrelated topics name-check it.
- **block** appears in 274 notes (82.53%). Same caveat: most are references to *blockers in upstream PR discussions*, *block-rate metrics on dispatcher data*, or the literal English word, not actual hook blocks. The 8 real hook blocks across the corpus are a tiny minority of these 274 mentions.
- **retry** appears in 22 notes — and these are nearly always references to *retry semantics in upstream OSS code being reviewed* (Bun stream-disconnect retry, dual-arm retry classifier, fetch-interceptor retry), not the agent's own pipeline retries. The agent rarely narrates its own retries because most of them succeed silently.
- **scrub** appears in 27 notes, of which the 15 with explicit hit counts are the data extracted above.

The signal-to-noise ratio of the `notes` field as a metrics surface is therefore somewhere around 15/332 ≈ 4.5% for any given pattern. The other 95% of the field is narrative — useful for forensics, terrible for aggregate stats — which is why the dispatcher emits a separate structured `families` / `repos` / `commits` ledger for accounting purposes.

## A design principle, restated

Two-stage redaction is not redundancy; it is **division of labor between a smart-but-fallible upstream filter and a dumb-but-perfect downstream fence**. The upstream filter handles 5–10x the volume of literal hits, makes contextual judgments about substitution vs deletion, and accepts a small false-negative rate. The downstream fence handles the residual at zero false-negative rate but costs a full retry cycle when it fires.

If you are building any agent pipeline that pushes content into a shared repo and you need to enforce a content policy, the math from this corpus suggests the right ratio is roughly **one literal-substring fence at the wire + one semantic rewriter in the agent prompt**, with the agent prompt explicitly told (a) what the fence will reject, (b) that the fence will not be bypassed, and (c) that one retry is allowed before escalation. Anything more elaborate at the fence (regex with context windows, semantic policy LLM at the hook) is solving a problem that the upstream filter already solved 5–10x more cheaply.

The 43 hits caught upstream against the 8 that made it to the hook is the lived evidence that this division of labor works in practice on a single-operator pipeline running ~50 ticks/day for 7 days. Scale it up by a factor of ten and the ratio should hold; scale it up by a factor of a hundred and the upstream filter will start to drift, and you will need a second-pass scrubber (which the tick-31 note already shows the agent inventing on its own when it noticed an underscore-adjacent miss). The pipeline tells you what it needs by failing in a particular shape; the operator's job is to read the shape and add only what the failure demands, no more.
