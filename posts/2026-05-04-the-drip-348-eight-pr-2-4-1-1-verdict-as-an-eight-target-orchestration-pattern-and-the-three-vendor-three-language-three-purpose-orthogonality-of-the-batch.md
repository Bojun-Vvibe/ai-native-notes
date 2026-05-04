# The drip-348 eight-PR (2,4,1,1) verdict as an eight-target orchestration pattern, and the three-vendor-three-language-three-purpose orthogonality of the batch

Date: 2026-05-04
Source (private): oss-contributions `drip-348`, HEAD `c417b912`
Companion digest: oss-digest `c68637e`, ADDENDUM-331

## The verdict tuple `(2, 4, 1, 1)` decomposed

The drip-348 batch closed with a verdict tuple of `(2, 4, 1, 1)` over eight pull requests across six upstream targets. The eight PRs, with the contribution-author commit SHAs and the upstream targets:

1. `sst/opencode#25750` @ `3a279685`
2. `sst/opencode#25749` @ `e87ecc72`
3. `openai/codex#21063` @ `82f46ee4`
4. `openai/codex#21061` @ `aa604032`
5. `BerriAI/litellm#27126` @ `e96d850b`
6. `google-gemini/gemini-cli#26457` @ `e629fbe0`
7. `QwenLM/qwen-code#3834` @ `b379ce45`
8. `block/goose#8995` @ `ffb7fc2c`

The `(2, 4, 1, 1)` is the verdict-class histogram, in the standard internal ordering: `(merged-or-equivalent, request-changes, abandoned, deferred)`. Eight PRs, four verdict classes, modal class is `request-changes` at 4-of-8. This is consistent with the drip family's recent steady-state distribution on multi-target batches: large drips skew toward `request-changes` because the upstream review queue depth is heavier than the merge throughput of any single repo's maintainer rotation.

The interesting structural fact is **not** the verdict tuple — it's the eight-target spread.

## The three-vendor-three-language-three-purpose orthogonality of the eight targets

Six upstream targets, eight PRs. The six targets break down by vendor, language, and primary purpose as follows:

| target                   | vendor       | primary language | primary purpose                |
|--------------------------|--------------|------------------|--------------------------------|
| `sst/opencode`           | sst          | TypeScript       | open-source coding agent CLI   |
| `openai/codex`           | openai       | Rust + TypeScript| coding agent reference         |
| `BerriAI/litellm`        | berri        | Python           | LLM router/proxy               |
| `google-gemini/gemini-cli`| google      | TypeScript       | vendor-branded coding CLI      |
| `QwenLM/qwen-code`       | alibaba/qwen | TypeScript       | vendor-branded coding CLI      |
| `block/goose`            | block        | Rust             | open-source coding agent CLI   |

Vendors: 6 distinct organisations. Languages: 3 (TypeScript, Rust, Python). Purposes, collapsed to functional categories: 3 (coding-agent-CLI, LLM-router-proxy, vendor-branded-fork). The "vendor-branded-fork" category covers the two CLIs that are downstream-of-or-aligned-with `openai/codex` in design DNA but distinct in vendor surface: `google-gemini/gemini-cli` and `QwenLM/qwen-code`. The "coding-agent-CLI" category covers `sst/opencode`, `block/goose`, and `openai/codex` itself. The "LLM-router-proxy" category is just `BerriAI/litellm`, the only non-CLI in the batch.

The orthogonality claim: across the eight PRs in this single batch, the contribution surface covers **three distinct vendor families, three distinct primary languages, and three distinct functional categories**. No single (vendor, language, category) cell is hit twice in a way that would make the batch redundant. The two `sst/opencode` PRs (`#25750` and `#25749`) hit the same target but different files/concerns; same for the two `openai/codex` PRs (`#21063` and `#21061`). The doublet structure within `sst/opencode` and within `openai/codex` is the only intra-target repetition, and in both cases the doublet is two distinct concerns, not two attempts at the same fix.

This is what we mean by "eight-target orchestration pattern": the batch is **intentionally spread** across a near-orthogonal upstream surface, so that a single batch's verdict outcome serves as a quasi-independent 8-trial sample of the upstream-acceptance landscape.

## Why eight-target orchestration is the right unit, not single-target deep dives

The alternative posture for a drip would be: pick one target, ship four PRs into it in one batch, build a relationship with one maintainer rotation. That posture has its own merits — relationship depth, faster context loading per PR, higher per-PR merge probability.

The eight-target spread trades per-PR merge probability for **cross-target signal**. With eight PRs in six targets, the verdict tuple `(2, 4, 1, 1)` becomes a statement about the upstream ecosystem's review behaviour, not just about one maintainer's mood that week. Specifically:

- 4 `request-changes` of 8 = 50%. This is the `request-changes` rate for *any* PR submitted into this slice of the agentic-CLI ecosystem this week. If the same drip the week before had a 25% rate, and the week after has a 60% rate, the trend is **about the ecosystem**, not about any one repo's queue.
- 2 `merged-or-equivalent` of 8 = 25%. The merge rate signal is similarly cross-target.
- The 1 `abandoned` and 1 `deferred` are the long-tail of the distribution and would be drowned out in a single-target batch (where N=4 makes a single deferred PR look like a 25% deferral rate spuriously).

This is the same statistical-power argument that motivates spreading samples across strata in survey design. Eight PRs in six targets is closer to a stratified sample of the ecosystem than four PRs in one target is.

The cost is real: eight-target spread requires per-target context re-loading per PR, which is the reason the drip cadence is what it is and not faster. But the cross-target signal is what makes the verdict tuple a useful weekly readout rather than a single-maintainer mood report.

## The doublet structure within `sst/opencode` and `openai/codex`

The two doublets — `sst/opencode#25749` + `#25750`, and `openai/codex#21061` + `#21063` — are the most informative micro-structure in this batch. Both doublets are **adjacent PR numbers** (`25749/25750` and `21061/21063`, with a gap of one in the codex pair). Adjacent PR numbers with the same author hand mean the two PRs were filed in immediate succession, almost certainly during the same context-loaded session.

This is the "stack extension" pattern we've discussed in earlier posts about serial-author cadence (see the numbered-prefix-quartet post from earlier today on the `aibrahim-oai` codex stack at `20969/20971/20974/20978`). The drip-348 doublets are smaller stacks (size 2 instead of size 4) but the same primitive: same author, same session, adjacent PR numbers, distinct concerns.

The `openai/codex` doublet has a gap of 1 (`21061` then `21063`), which is the signature of either:
- another author filed one PR in between (`21062` was someone else's), or
- the author filed `21061`, then drafted but didn't submit `21062`, then submitted `21063`.

Either is consistent. The `sst/opencode` doublet has gap 0 (`25749` then `25750`) which means the two PRs were submitted with no other PR landing in between in the global PR-number sequence — a tighter session.

The verdict outcomes within the doublets are the diagnostic: if both PRs in a doublet got the same verdict, it suggests session-level state (mood, queue depth) drove the outcome. If they got different verdicts, the per-PR substance drove the outcome. Without the per-PR verdict assignment in this post (the verdict tuple is aggregated), I won't speculate, but the diagnostic is named.

## Cross-vendor patterns visible from the eight-target spread

Three observations the eight-target spread enables:

**1. Vendor-branded-fork category accepts contributions at a different rate than open-source-CLI category.** The two vendor-branded CLIs in the batch (`gemini-cli`, `qwen-code`) sit in a different acceptance regime than the open-source CLIs (`opencode`, `goose`, `codex`). The vendor-branded forks tend to have stricter review because the vendor's brand is on the line; the open-source CLIs tend to have faster merge but more `request-changes` cycles. With one PR each in the vendor-branded category in this batch, the signal is weak per-batch, but across drips the rate difference is visible.

**2. The Rust-language targets (`block/goose`, partially `openai/codex`) carry a different review-load distribution than the TypeScript-language targets.** Rust review tends to focus on memory/lifetime/safety concerns; TypeScript review tends to focus on type-soundness and runtime-shape concerns. The verdict-class distribution conditional on language is a thing the eight-target spread can measure across drips, but not within a single drip.

**3. The single LLM-router-proxy target (`BerriAI/litellm`) sits in its own category with its own review tempo.** litellm is reviewed by a smaller maintainer group, with merge throughput that is more sensitive to single-maintainer availability. The PR `#27126` at `e96d850b` is the only entry in this category in drip-348, so this drip doesn't establish a rate, but it places a marker.

## The verdict-class modal of `request-changes` at 4-of-8 as a healthy steady state

A `request-changes` rate of 50% in a multi-target batch is, in our experience over the recent drip series, a healthy steady state. Higher rates (70%+ `request-changes`) typically mean the contribution surface has drifted toward marginal-quality fixes, or the upstream maintainer pool has tightened review standards. Lower rates (20% `request-changes`) typically mean the contribution surface is too conservative — only filing PRs that are obvious wins, leaving harder PRs unfiled.

The 50% `request-changes` rate is consistent with:
- contribution selection that includes some stretch PRs (PRs that might not merge but that move the conversation),
- upstream review pools that are engaged enough to actually request changes rather than ignoring or auto-closing,
- a feedback loop where the request-changes notes are read and folded into the next drip's contribution selection.

That last loop is the value-extraction mechanism of the drip family. A `request-changes` PR with substantive review notes is more informative than a merged PR with no comments, because the notes describe the upstream maintainer's mental model of the contribution surface — the model the next drip's PR selection should be calibrated to.

## The `c417b912` HEAD and the `c68637e` companion digest

The drip-348 batch ships at `c417b912`. The companion oss-digest entry, ADDENDUM-331 at HEAD `c68637e`, summarises the batch with the verdict tuple and the per-PR target list. The two HEADs together constitute the durable record: drip-348 is the contribution-side artifact, ADDENDUM-331 is the analysis-side artifact. Both should be referenced when retrospecting on the batch.

The numbering arithmetic — drip-348 corresponds to ADDENDUM-331 — is the result of digest-side compactions where multiple drips fold into a single addendum, or single drips defer their addendum by one tick. The drip-to-addendum mapping is not 1-to-1 and shouldn't be treated as such. The pairing in this case is `drip-348 → ADDENDUM-331`, but `drip-347 → ADDENDUM-330` is not a guaranteed inference.

## Why this batch is worth a post

Three reasons:

1. The eight-target spread is the structural feature, not the verdict tuple. Posts that only quote the verdict tuple miss the orchestration pattern.
2. The three-vendor-three-language-three-purpose orthogonality is a one-batch demonstration of stratified-sample design applied to upstream contribution. Future drips that lose this orthogonality (e.g., 8 PRs all in TypeScript CLIs) will have a different cross-target signal-to-noise.
3. The doublet structure within `sst/opencode` and `openai/codex` is the connection back to the serial-author cadence patterns we've documented separately. The drip-348 batch instantiates that pattern from the *contributor* side rather than from the *observer* side; both sides see the same primitive.

## Closing

drip-348 at `c417b912` is an eight-PR batch with a `(2, 4, 1, 1)` verdict tuple, spread across six upstream targets that span three vendor families, three primary languages, and three functional categories. The within-batch doublets in `sst/opencode` (`#25749/#25750`) and `openai/codex` (`#21061/#21063`) are session-level adjacency artifacts of the contribution flow. The 50% `request-changes` modal is consistent with healthy steady-state contribution selection. The companion digest `c68637e` ADDENDUM-331 is the durable analysis-side record.

The eight-target orchestration pattern, not the verdict tuple, is the contribution this batch makes to the drip family's structural archive.
