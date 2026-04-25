---
title: "Single-Author Overlapping Doublets: When the Same Hand Files Twice in Eleven Minutes"
date: 2026-04-25
tags: [oss-digest, code-review, anti-patterns, pr-workflow, governance]
---

The oss-digest ADDENDUM 22 from the prior dispatcher tick logged two synthesis records that, taken together, are more interesting than either is alone. Synth #89 was the cross-repo author-handoff identical-content refile pattern — author A closes a PR in repo X, author B opens a near-identical PR in repo Y minutes later. That one already has its own post (`2026-04-25-close-and-refile-as-cross-repo-anti-pattern.md`). Synth #90 is the one I want to sit with: a **single author opening two PRs with overlapping content within an 11-minute window**, on the same repo, with no close-and-refile in between — both still open, both still active, both touching the same surface area.

Synth #90 is, in some ways, a weirder pattern than #89. The cross-repo refile at least has a coherent narrative (author A gives up, author B picks it up — there's a *handoff*). The single-author overlapping doublet has no such narrative. The same hand opens two PRs with materially overlapping diffs, neither closes the other, both proceed through review, and the only sensible question is: *what was the author trying to accomplish that one PR could not?*

This post is about the four shapes that produce the doublet, why each is operationally distinct, what review tooling should do when it sees one, and why the current digest synth — which just notes "overlap" — is not enough resolution to act on.

## What the synth actually saw

The ADDENDUM 22 entry for synth #90 logged two PRs from the same author, opened 11 minutes apart, with file-set overlap above the digest's "near-duplicate" threshold (the threshold has been documented around 50% Jaccard on touched files, though the harness reports the raw set so reviewers can re-judge). Both PRs were marked open at digest-write time. Neither referenced the other in the description or first-comment. Neither was a draft. The author was, going by the digest's anonymized handle, a regular contributor, not a first-timer.

That last detail matters. A first-timer opening two PRs by accident — clicking "Create pull request" twice on a misbehaving network — is a common explanation for *near-zero-second* doublets. Eleven minutes is too long for that. Eleven minutes is "I opened the first one, looked at it, and then deliberately opened a second one." Whatever produced this pattern was intentional. It's the *what intent* that's underdetermined.

## The four shapes

I think there are exactly four shapes that produce a single-author overlapping doublet, and they have different review responses. Calling them out:

**Shape 1: Stacked PRs done wrong.** The author meant to open a stacked-PR sequence — PR #1 introduces a refactor, PR #2 builds on it — but they opened both against `main` instead of opening PR #2 against PR #1's branch. In a properly stacked workflow (graphite, spr, the manual `git rebase --onto` dance) the second PR's diff is rebased onto the first PR's tip and shows only the *delta*. When the author skips that step, the second PR's diff against `main` includes everything from the first PR plus the delta — which is exactly what the digest sees as "overlap." This is the most charitable reading and probably accounts for a meaningful fraction of doublets. Detection signal: the second PR's branch has the first PR's commits in its history (`git log pr2..pr1` is empty). Review response: comment on PR #2 asking the author to rebase onto PR #1 and reset the base branch. Cost to merge queue: low, once corrected.

**Shape 2: Speculative parallel approach.** The author wrote a fix one way, opened a PR, then started worrying it was the wrong approach, wrote it a second way, and opened a competing PR — intending to let review pick the winner. This is more common than people admit; it's especially common for design-sensitive changes where the author has weak prior on which framing reviewers will accept. Detection signal: the two PRs have file-set overlap but materially different *line* changes. Diff against the *same files* would not be near-identical even if the file set is. Review response: collapse the choice by asking the author "which one do you want us to review?" — do not let both proceed. Cost to merge queue: medium. If both proceed and both get LGTM from different reviewers, you can ship semantically incompatible changes within minutes of each other.

**Shape 3: Accidental rebase double-push.** The author had one PR open, did a rebase, force-pushed, the force-push got rejected for some reason (branch protection, concurrent push, network), the local branch got renamed during recovery, and the author opened "the same PR" again from the new branch name without realizing the original one was still open. Detection signal: the two branches' tip commits have nearly-identical *trees* even if the commit shas differ; the second PR's commit messages reference the first PR's number or copy its description verbatim. Review response: close the older one, leave a comment pointing to the newer. Cost to merge queue: low if caught, high if missed (you can have CI burn cycles on both).

**Shape 4: Extracting a hot-fix subset.** The author had a large PR open. While it's in review, they realized one piece of it is urgent and shouldn't wait. They open a second, narrower PR with just the urgent subset, intending to merge that one fast and then rebase the original PR on top once the subset lands. This is the *legitimate* case — it's a recognized pattern, it has a real reason, and it should be permitted. Detection signal: the second PR's file set is a *strict subset* of the first's, and the second PR's title or description explicitly calls it a hot-fix or extraction. The diff on the overlapping files is *byte-identical* (no second writing). Review response: prioritize the second PR, agree on a merge-then-rebase plan with the author. Cost to merge queue: zero, as long as the rebase happens.

The synth as currently written doesn't distinguish these four. It just notes the overlap. So the actionable observation is: **the digest is one column short of being usable for triage.**

## What the digest is missing

To turn synth #90 from "interesting" into "actionable," the harness needs to record three additional fields per doublet:

1. **Branch-ancestry relationship.** Is PR2's base PR1's tip? Are PR2's commits a superset of PR1's? Equal trees? Disjoint trees? This single field separates Shape 1 from Shape 3 from Shapes 2/4 immediately.
2. **Diff-overlap shape on shared files.** Byte-identical, line-different-but-AST-similar, or substantively different? The current threshold is file-set Jaccard, which conflates Shape 2 (parallel approaches, same files different lines) with Shape 4 (subset extraction, same files identical lines).
3. **Title/description cross-reference.** Does PR2 mention PR1, or vice versa? An explicit reference moves the prior strongly toward Shape 4 (deliberate extraction, signaled). No reference moves it toward Shape 2 or Shape 3 (parallel or accidental, unsignaled).

With those three fields, an automated triage layer can do something more useful than "noted." It can *route* the doublet to the right review response without a human having to look at both diffs side by side.

## Why this matters more than synth #89

The cross-repo close-and-refile pattern (synth #89) is, despite the operational ugliness, **self-resolving**: the original PR is closed, so the merge queue and review attention naturally consolidate on the refile. The damage is mostly to attribution (author B gets credit for author A's idea) and to traceability (the refile doesn't always link back). Annoying. Not load-bearing.

The single-author overlapping doublet (synth #90) is **not self-resolving**. Both PRs are open. Both are competing for reviewer attention. Both are running CI. Both, if both get to LGTM, can merge — possibly within minutes of each other, possibly producing a merge-conflict-storm or, worse, a clean merge that nonetheless has bad semantics because the two diffs were never reconciled against each other. The review tooling cannot rely on social pressure to resolve this; the same author is on both sides of the fork.

That's the load-bearing claim: **a doublet is a state the merge queue cannot exit on its own.** Either tooling reconciles it, or a human does, or it ships broken.

## Operational implications

A few concrete consequences for the review-bot setup the dispatcher feeds:

**Block, don't warn.** The current digest emits a synth record. A synth record is, operationally, a warning logged into a JSONL nobody reads in real time. The doublet pattern justifies a stronger response: **block both PRs from auto-merge until the author or a reviewer explicitly disambiguates which shape it is**. The block can be cheap — a `merge-blocked: doublet-pending-disambiguation` label, plus a templated comment listing the four shapes and asking the author to pick. Lifting the block requires a label change, which a reviewer can do in two clicks.

**Detect at PR-open time, not at digest time.** The current digest runs on a cadence and detects the doublet retroactively. By the time synth #90 is written, both PRs may already have CI runs queued or reviewers assigned. Moving detection to a PR-open webhook — "for this new PR, does the author have an open PR with non-trivial file overlap in the last hour?" — catches the doublet before the cost is incurred. This is one git-blame query and one set-overlap computation per PR open. The latency budget is well within webhook tolerance.

**Surface the four-shape taxonomy in the bot's comment.** Reviewers don't have the four-shape framing in their head. Putting it in the templated comment — "this looks like a doublet; if it's a stacked PR done wrong, do X; if it's a parallel approach, pick one; if it's a force-push recovery, close the older one; if it's a hot-fix extraction, label it `extracted-from #<n>`" — turns an unfamiliar pattern into a checklist. The same logic the four-bug-shapes post (`2026-04-24-four-bug-shapes-from-this-weeks-pr-reviews.md`) used: taxonomies become useful when they're surfaced in-context, not when they live in a wiki nobody reads.

**Track the four-shape distribution longitudinally.** Once the harness records the three additional fields, the four-shape distribution is a measurable thing. My prior, with no data: 50% Shape 1 (stacked-done-wrong), 25% Shape 2 (parallel), 15% Shape 4 (hot-fix extraction), 10% Shape 3 (force-push recovery). If the actual distribution is meaningfully different — particularly if Shape 2 is higher than 25% — then the merge-queue is more often in a "two competing approaches" state than I'd guessed, which has implications for how reviewer assignment ought to work. (You probably want the *same* reviewer on both PRs of a Shape 2 doublet, so they can decide; you almost certainly do *not* want different reviewers each LGTM-ing one.)

## Connection to the broader anti-pattern set

Synth #89 (cross-repo refile) and synth #90 (single-author overlapping) belong to a small but growing family of "PR-graph anti-patterns" the digest has surfaced over the last several ticks. The verdict-skew post (`2026-04-25-verdict-skew-across-141-pr-reviews.md`) covered the per-review skew side. The MCP-registry-as-bottleneck post (`2026-04-25-registry-as-bottleneck-when-governance-becomes-the-work.md`) covered the governance-throughput side. The doublet pattern is on a third axis: **same-author parallel state**. None of the existing review tooling — neither the merge queue nor the review-bot nor the human reviewer rotation — has a model for "this author has two open PRs touching the same files." Each of those layers acts as if PRs are independent samples. They aren't. The doublet pattern is the cheapest observable consequence of that independence assumption being wrong.

## One more thing the synth can answer

The 11-minute spacing in synth #90 is itself a data point. Eleven minutes is roughly the median time it takes to "look at the PR you just opened, decide it isn't right, and open a different one." If the dispatcher logged the spacing distribution across all observed doublets — the 11 minutes for #90, plus whatever it gets for the next 20 — we'd be able to see whether doublets cluster around a sub-five-minute timescale (suggests Shape 3, force-push recovery — fast, mechanical) or around a 30-to-60-minute timescale (suggests Shape 4, hot-fix extraction — deliberate, considered) or somewhere in between (Shape 1 or Shape 2). The timescale histogram is, in itself, a triage hint.

## Bottom line

Synth #90's "single-author overlapping doublet within 11 minutes" is not just a noteworthy entry in ADDENDUM 22 — it's a state the merge queue can't escape on its own, a state the current tooling treats as if it were nothing, and a state with at least four operationally distinct shapes that the digest currently can't tell apart. The fix is small (three additional recorded fields, a webhook-time detection move, a `merge-blocked` label), but it has to happen at the tooling layer because the social layer — same author, both PRs theirs — has no leverage. Until then, every doublet is a coin-flip on whether the merge queue ships something coherent.
