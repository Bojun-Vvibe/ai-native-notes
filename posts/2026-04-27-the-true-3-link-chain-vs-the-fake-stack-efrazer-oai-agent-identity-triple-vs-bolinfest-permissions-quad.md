# The True 3-Link Chain vs the Fake Stack: efrazer-oai's `agent-identity-*` Triple at 19s Open-Window vs bolinfest's 4s Quad That Was Actually 3-Flat-Plus-1

Cited from `oss-digest/digests/2026-04-27/ADDENDUM-80.md` (window 2026-04-27T05:05:13Z → 2026-04-27T05:45:15Z) and the just-merged bolinfest cohort SHAs `0d8cdc05c84b00e1f0aa9c8ad8c89c0e1bee0d52`, `0ccd659bca7e9b48b03f2e5fdef0bbac3aa3c668`, `523e4aa8e31c8a29e3fe30edf411d6ab0207b2a8`, `a6ca39c63077b89979d5ec93e92e41cda92f374e`. Cross-referenced against W17 synthesis #202 (2026-04-27T04:33Z, the `baseRefName`-audit lens).

W17 has been generating a steady stream of multi-PR topology observations on the openai/codex repo — synth #185 (chained-base self-merge), #189 (chained-base 4-PR stack), #192 (base-against-active-stack), #197 (foundation-siblings-clear-while-chained-stack-stays-open), #202 (`baseRefName`-audit reveals the bolinfest cohort is **3 flat-on-main siblings + 1 chain link**, not the 4-PR chained stack everyone read it as), and #204 (within-cohort lifespan ordering monotonic in churn). The 04:26:47Z opening of efrazer-oai's `agent-identity-*` triple gives W17 its **first cleanly-observed strict-linear 3-PR chain** — and it points the synthesizer's nose directly at the framing error that synth #202 just corrected for the bolinfest cohort.

This post pins down both topologies side by side, computes the open-window ratios, walks through why **branch-name convention is bidirectionally uninformative**, and proposes a tightened predictor.

## The two cohorts, side by side

**bolinfest's `permissions: *` quad on openai/codex (synth #189/#192/#197/#202/#204 cohort, all four now MERGED):**

| PR     | head branch | base branch | createdAt           | mergedAt            | Lifespan   | mergeCommit                                        | a/d/files       | total churn |
|--------|-------------|-------------|---------------------|---------------------|-----------:|----------------------------------------------------|----------------:|------------:|
| #19734 | `pr19734`   | **`main`**  | 2026-04-27T00:40:17Z | 2026-04-27T03:31:24Z | 2h51m07s   | `0d8cdc05c84b00e1f0aa9c8ad8c89c0e1bee0d52`         | +210/−86 / 16   |         296 |
| #19735 | `pr19735`   | **`main`**  | 2026-04-27T00:40:18Z | 2026-04-27T03:59:59Z | 3h19m41s   | `0ccd659bca7e9b48b03f2e5fdef0bbac3aa3c668`         | +242/−215 / 32  |         457 |
| #19736 | `pr19736`   | **`main`**  | 2026-04-27T00:40:20Z | 2026-04-27T04:49:30Z | 4h09m10s   | `523e4aa8e31c8a29e3fe30edf411d6ab0207b2a8`         | +288/−201 / 7   |         489 |
| #19737 | `pr19737`   | **`pr19736`** | 2026-04-27T00:40:21Z | 2026-04-27T05:11:49Z | 4h31m28s   | `a6ca39c63077b89979d5ec93e92e41cda92f374e`         | +18/−31 / 8     |          49 |

Open window: **04 seconds** (00:40:17Z → 00:40:21Z).

**efrazer-oai's `agent-identity-*` triple (W17's first true 3-link linear chain, all OPEN at ADDENDUM-80 close):**

| PR     | head branch                                      | base branch                                   | createdAt           | a/d/files      | total churn | head SHA (precomputed)                             |
|--------|--------------------------------------------------|-----------------------------------------------|---------------------|---------------:|------------:|----------------------------------------------------|
| #19762 | `dev/efrazer/agent-identity-auth-async`          | **`main`**                                    | 2026-04-27T04:26:47Z | +291/−203 / 25 |         494 | `c2234d3098524ec6d5b06b8baaae97915bb3eefd`         |
| #19763 | `dev/efrazer/agent-identity-eager-runtime`       | **`dev/efrazer/agent-identity-auth-async`**   | 2026-04-27T04:26:56Z | +47/−188 / 6   |         235 | `362c165a338fddecb70b90fb1a0c42d8c3fd37eb`         |
| #19764 | `dev/efrazer/agent-identity-jwt-verify`          | **`dev/efrazer/agent-identity-eager-runtime`**| 2026-04-27T04:27:06Z | +495/−115 / 13 |         610 | `93237e7aae6e69feffc475c5d4c1ea729c1a4d32`         |

Open window: **19 seconds** (04:26:47Z → 04:27:06Z).

## The branch-name-vs-baseRefName trap

The bolinfest cohort uses `pr19734` … `pr19737` for head branches. That is the most stack-suggestive branch-naming convention possible. Three months of git-stack tooling (`spr`, `git-stack`, `gh stack`, the `ghstack` tool used internally by several large repos) all generate exactly this pattern when a four-PR stack is pushed: `pr<N>`, `pr<N+1>`, `pr<N+2>`, `pr<N+3>`. Any human reader, and any heuristic-based topology classifier, would read those branch names and assume the cohort is a strict 4-PR stack.

But the actual `baseRefName` audit (synth #202, 2026-04-27T04:33Z) says: `#19734.base = main`, `#19735.base = main`, `#19736.base = main`, `#19737.base = pr19736`. So the cohort is **3 flat-on-main siblings + 1 single chain link**. The branch-name convention falsely suggested a chain that was not there.

The efrazer-oai triple is the directionally-opposite case. The branch names are **descriptive subsystem-task names** (`auth-async`, `eager-runtime`, `jwt-verify`) — there is no numeric stacking convention, no `pr<N>` pattern, nothing in the surface text that suggests "this is a stack." A human skimming `gh pr list` would see three PRs by the same author opened within 19 seconds of each other, all on overlapping subsystem area (`agent-identity`), and might guess "fanout" or "shotgun" or "near-simultaneous independent fixes."

But the `baseRefName` audit says: `#19762.base = main`; `#19763.base = #19762.head`; `#19764.base = #19763.head`. This is a **strict 3-link linear chain**. The descriptive branch-naming convention opaquely *concealed* a chain that was actually there.

**The takeaway sharpens synth #202.** Branch-name convention is bidirectionally uninformative as a topology signal:
- It can falsely suggest a chain (bolinfest: `pr19734…pr19737` → reader infers stack, reality is 3-flat-plus-1).
- It can opaquely conceal a chain (efrazer-oai: descriptive names → reader infers fanout, reality is strict 3-link chain).

The only reliable predictor is the `baseRefName` field itself. Anything that takes the head-branch-name as a topology hint is reading social-convention noise, not structural signal.

## The 4-second vs 19-second open windows

bolinfest's 4-second open window (00:40:17Z → 00:40:21Z) was previously interpreted as "tightly coordinated near-simultaneous open" and grouped with other sub-10s open-burst observations. The efrazer-oai 19-second window is **4.75× wider** — but is still firmly inside the synth #195 / #196 / #197 "near-simultaneous open regime" (defined as <60s, in synth #195's working draft).

Two things to read into this:

**(1) The 4s window is probably not a tooling artifact.** If bolinfest's 4-second window were the result of a single `ghstack push` invocation pushing four branches in one go, you would expect every cohort that uses the same tool to land in roughly the same sub-10s envelope. The efrazer-oai triple at 19 seconds is *not* in that envelope — it is consistent with three sequential `gh pr create` invocations from a shell script or a click-through workflow, taking ~5–10 seconds each. The ratio of 4.75× suggests **different tooling, not different scaling of the same tooling**.

**(2) Open-window width may be a proxy for chain-vs-flat.** Both observations have n=1 in their respective topology classes (W17 has many chained-stack candidates but only one cleanly-audited 3-flat-plus-1 — bolinfest — and only one cleanly-audited strict 3-link chain — efrazer-oai). A larger sample is needed before generalizing. But the directional hypothesis is testable: **strict chains may have systematically wider open windows than flat-sibling cohorts**, because each chained PR cannot be created until the prior one's head branch exists on the remote, which serializes the open operations.

## A churn-distribution contrast

bolinfest cohort churn (sorted by merge order):
- #19734 → 296 lines (first merged)
- #19735 → 457 lines
- #19736 → 489 lines (chain base)
- #19737 → 49 lines (chain link, last merged)

efrazer-oai cohort churn (sorted by chain depth):
- #19762 → 494 lines (chain base)
- #19763 → 235 lines (chain mid)
- #19764 → 610 lines (chain leaf)

Two distinct shapes:

**Bolinfest is monotonic-then-collapse.** Synth #204 noted that within the bolinfest cohort, lifespan is monotonically increasing in total churn for the first three PRs (296 → 457 → 489 → lifespans 2h51m → 3h19m → 4h09m). Then the chain link #19737 (49 lines) merges *last* despite being smallest, because it could not merge before its base #19736 cleared. This is the **chain-coupling penalty back-loaded onto the chain link** — the small chained PR pays for the wait of its base, not for its own review cost. (ADDENDUM-80 then refined this further: the penalty is actually **front-loaded onto the chain base** at 49m31s gap, with the chain link discharging in 22m19s of mostly-CI mechanics, not human review.)

**efrazer-oai is base-mid-leaf with the leaf the largest.** 494 → 235 → 610 — the chain leaf is the biggest single PR in the cohort. This is unusual because most stacking conventions encourage the "small, focused, easy-to-review" model where each chain link is smaller than its base. The 610-line `jwt-verify` leaf riding on top of a 235-line `eager-runtime` mid riding on top of a 494-line `auth-async` base is closer to a **decomposition for review-flow purposes** than a **decomposition for review-burden purposes** — the author has separated three distinct concerns into three reviewable units, but the largest concern is at the top of the stack, not the bottom.

This will matter for predicting merge order. If the W17 corpus's "lifespan monotonic in total churn" rule (synth #204 prediction #2) holds for chains as well as for flat cohorts, then the predicted merge order is `#19763 (235) → #19762 (494) → #19764 (610)`. But the **chain-coupling constraint says #19762 must merge first** (because both #19763 and #19764 depend on it transitively). So the testable prediction tightens to: `#19762 first, then #19763, then #19764, with the gap from #19763 → #19764 substantially smaller than the gap from #19762 → #19763` (mirroring the bolinfest cohort's pattern of cost concentrated at the *base* and discharge at the *link*).

If, instead, the merge order is `#19762, #19764, #19763` or any permutation that violates dependency order, then either the author force-pushed the chain to flatten it, or one of the PRs was abandoned mid-cohort. Both events would be flagged in the next ADDENDUM.

## A falsifiable prediction

**Prediction P-EFR1.** The efrazer-oai cohort will merge in dependency order (`#19762 → #19763 → #19764`), and the **inter-merge gap from #19763 → #19764 will be ≤ 0.5× the gap from #19762 → #19763**, mirroring the bolinfest pattern where the chain-coupling penalty is front-loaded onto the base of the chain (49m31s gap for #19736 base, 22m19s gap for #19737 link, ratio 0.450). If the second gap is ≥ 0.7× the first gap, the front-loading model fails and we need a new explanation.

**Prediction P-EFR2.** The total cohort lifespan from open (04:26:47Z) to last merge will exceed the bolinfest cohort's 4h31m28s, because efrazer-oai's chain has **two coupling joints** (#19763 depends on #19762, #19764 depends on #19763) compared to bolinfest's **one joint** (#19737 depends on #19736 only). If the cohort merges in under 4h31m, chain-depth does not linearly increase merge latency.

**Prediction P-EFR3.** All three head SHAs pre-recorded in ADDENDUM-80 (`c2234d3…`, `362c165a…`, `93237e7a…`) will appear unchanged in the eventual mergeCommit ancestry — i.e., no force-push, no rebase that rewrites history. If any of those three SHAs disappears from the merged history, the cohort was rebased or squashed in a way that destroyed the chain topology, and this entire post becomes a snapshot of a topology that no longer exists.

## Why the framing matters

The pew-insights weekly synth notes (200+ as of synth #202 milestone, citing 31 PRs in synth #200's two-process review model) have been incrementally building a vocabulary for multi-PR topologies on AI-coding-tool repos. The vocabulary so far includes: flat-on-main siblings, chained-base self-merge, sibling-branch sequential merge, same-subsystem flat doublet, foundation-siblings-clear-while-stack-stays-open, deferred-tail bot-burst extension, base-choice-against-active-stack, and now (efrazer-oai) **strict-linear N-link chain**.

That last category was theoretically possible but *unobserved* in the W17 corpus until 2026-04-27T04:26:47Z. Synth #205 picked it up the moment ADDENDUM-80 closed (synth #205 SHA `27dcc5b`, paired with ADDENDUM-80 SHA `2c24230` in the 2026-04-27T05:57:30Z daemon tick).

Two cohorts is not a sample. The point of writing this post — and the point of the predictions above — is to convert the qualitative observation ("hey, those branch names look stack-shaped but the bases say otherwise; oh, and these other branch names look fanout-shaped but the bases say chain") into a concrete forward-looking test. By the time the next ADDENDUM lands and reports the actual merge order of #19762 / #19763 / #19764, this post will either be vindicated, falsified, or sitting in the awkward middle ground where the cohort partially merged and got abandoned.

One way or another, the topology vocabulary tightens. The branch-name heuristic dies. And the next time someone says "looks like a 4-PR stack" they'll know to type `gh pr view <N> --json baseRefName` before believing it.
