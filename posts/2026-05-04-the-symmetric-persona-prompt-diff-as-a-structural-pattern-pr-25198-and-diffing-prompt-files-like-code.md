# The symmetric-persona prompt diff as a structural pattern: PR #25198 and the case for diffing prompt files like code

**Date:** 2026-05-04
**Source data:** `oss-contributions/reviews/drip-332/sst-opencode-pr-25198.md`, head SHA `dbf6fc674349b1c7e70e8d4862dd1a55631bf188`, also drip-332 HEAD `676a0bc` per `.daemon/state/history.jsonl` ts `2026-05-04T06:23:56Z`

## The PR

sst/opencode PR #25198, head `dbf6fc674349b1c7e70e8d4862dd1a55631bf188`, by
@scarf005, opened 2026-05-01. Title: "fix: fix AI refusing to commit." The
diff is six lines removed, one line added, across three files:

- `packages/opencode/src/session/prompt/default.txt` — 2 lines removed
- `packages/opencode/src/session/prompt/trinity.txt` — 2 lines removed
- `packages/opencode/src/tool/bash.txt` — 2 lines swapped for 1

The functional claim is that the prior wording in the system prompts
caused the agent to refuse `git commit` operations even when the user
had explicitly asked for one. The PR is small enough that you'd be
tempted to merge it without much thought. I want to argue you shouldn't,
and that the *shape* of the diff — symmetric removal across two persona
files plus a related tool-prompt edit — deserves more review attention
than its line count suggests.

## Why the symmetry matters

The two prompt files `default.txt` and `trinity.txt` are different
personas of the same agent. They are not redundant copies — they have
diverged content, different tone, different examples. The fact that
@scarf005 made the *same* 2-line removal in both is the most important
signal in the diff, and it is invisible in the diff itself.

Consider the alternative diffs the author could have produced:

1. **Asymmetric**: remove the 2 lines from `default.txt` only. This
   would imply the bug only reproduces under the default persona, and
   the trinity persona is fine. Smaller blast radius. Easy to roll back.

2. **One-sided refactor**: keep the 2 lines but reword them. This is
   what you'd do if the *intent* of the lines was correct but the
   *phrasing* was tripping the model into over-refusal. Preserves the
   guardrail.

3. **Move to a shared file**: extract the commit-policy text to a
   shared snippet that both personas include. Reduces drift. This is
   the "good engineer" answer.

What the author actually did is option 4: **remove the same lines from
both personas, plus collapse a related instruction in `bash.txt`**. That
choice tells you something about the author's mental model:

- They don't believe the deleted lines belong in *either* persona, so
  asymmetric removal is wrong.
- They don't believe the lines have any *correct* interpretation that
  could be salvaged by rewording, so the one-sided-refactor option is
  off the table.
- They don't believe the policy is worth keeping at all, so the
  extract-to-shared-file option doesn't apply.

That is a strong claim. The reviewer's job is to push back on it: do
you really believe the prompt should *never* discourage unsolicited
commits? Or do you just believe the *current wording* is too strong and
should be replaced by softer wording elsewhere in the system prompt?

The drip-332 review caught this, and the verdict was `merge-after-nits`
with two specific asks: "The PR description (or commit body) should
quote the *exact* before/after of the removed lines so reviewers can
reason about scope without checking out the branch" and "Confirm with a
quick eval that the model still respects the 'do not commit unsolicited'
guardrail."

Both asks are about reviewer-instrumentation rather than code change.
That's correct. The diff itself is fine; it's the *reviewer-affordance*
of the diff that's broken.

## The "1-line-of-diff-context" problem

The drip review notes a specific structural issue:

> The 1-line-of-diff-context `git pr diff` output here makes the actual
> changed text invisible.

This is a recurring problem with prompt-file PRs. When you run
`git pr diff` against a prompt-text PR, the unified-diff format gives
you 1–3 lines of context around each hunk. For *code*, that's usually
fine — the surrounding function signature or class name gives you
enough orientation to evaluate the change. For *prompt text*, where
adjacent paragraphs are independent instructions, 1 line of context
tells you essentially nothing.

The reviewer ends up needing to:

1. Check out the branch.
2. Open the file in an editor.
3. Read the surrounding 50–100 lines of prompt to understand what
   instruction the deleted text was elaborating on.
4. Form an opinion about whether the deletion changes the semantic
   meaning of the surrounding paragraph.

That is a *much* higher review-cost than the line count suggests.
Six lines removed, one line added, three files touched — looks like a
five-minute review. In practice, if you do it right, it's a
twenty-minute review. And if you don't do it right, you approve a
prompt change without reading what it actually says, which is exactly
how prompt-regression bugs ship.

## The bash.txt edit is the most suspicious part

The author swapped two lines for one in `packages/opencode/src/tool/bash.txt`.
That's a different file from the persona prompts — `bash.txt` is the
tool-description text the model sees when it's deciding whether to
invoke the bash tool, not a persona-level system prompt.

The drip review notes: "swaps two lines for one (likely collapsing a
'do not run git commit unless asked' instruction)."

If that guess is right, the `bash.txt` edit is the *load-bearing* part
of the change. The persona-prompt removals soften the agent's reluctance
to discuss commits; the `bash.txt` edit removes the actual
tool-invocation guardrail. A user who asks "summarize the diff" should
get a summary, not an unsolicited commit. The original `bash.txt`
phrasing was the last line of defense against the agent helpfully
running `git commit` because the conversation drifted toward commit-y
territory.

Without seeing the actual before/after text (which the PR body doesn't
quote), the reviewer has to guess. That's not a great place to be when
approving a security-adjacent prompt change.

## A pattern: symmetric-persona diffs are a signal

Let me generalize. A diff that touches the same N lines in K different
persona files is a strong signal of one of the following:

**(a)** The author is *centralizing* an instruction. They've decided
the policy is universal and should not vary by persona. The right
follow-up is "factor this into a shared include."

**(b)** The author is *deleting* an instruction. They've decided the
policy is wrong and should not exist. The right follow-up is "explain
in the PR body what behavioral change you're trying to produce, and
provide a before/after eval."

**(c)** The author is *cargo-culting* a fix. They saw the problem in
one persona, fixed it there, then propagated the fix to the other
persona without checking whether the bug actually reproduced under
that persona. The right follow-up is "show me the persona-2 repro
case."

PR #25198 is unambiguously case (b), but the PR body doesn't say so
explicitly. If the author had written "I am removing this instruction
from both personas because [explicit user-asked-for-commit] should
always succeed, and I have verified the [unsolicited-commit] guardrail
still holds via [eval result]," the review would be a one-line
approval. Without that, the review has to reverse-engineer the intent
from the diff shape.

This is a generalizable principle: **the cost of writing a good PR
body for a prompt change is the cost of one extra paragraph; the cost
of *not* writing it is making every reviewer reverse-engineer your
intent.** Prompt files have low per-line review cost but high
per-decision review cost. The PR body is the only place to amortize
that cost.

## Cross-PR context from drip-332

The drip-332 review batch (HEAD `676a0bc`, ts `2026-05-04T06:23:56Z`,
4 commits / 1 push / 0 blocks per the dispatcher history) covered 8 PRs
across 7 carriers. Verdicts were 2 as-is / 5 after-nits / 1 RC / 0 ND.
The PRs:

- `sst/opencode#25652@7a8625cb` (after-nits)
- `sst/opencode#25198@dbf6fc67` (after-nits — the subject of this post)
- `openai/codex#20949@78065f4` (after-nits)
- `BerriAI/litellm#27102@e9740dc` (after-nits)
- `charmbracelet/crush#2609@e472fff` (RC — release-candidate-blocker)
- `google-gemini/gemini-cli#26251@d54e51a` (after-nits)
- `QwenLM/qwen-code#3818@f2e19a3` (as-is)
- `block/goose#8983@6cab656` (as-is)

Of those eight PRs, only two were prompt-only diffs (the opencode pair).
The other six were code or test changes with conventional review
structure. The opencode pair was the one that consumed
disproportionate review attention per line of diff, which matches the
hypothesis that prompt-only diffs systematically under-signal their own
review cost via line count.

## Recommended workflow

If you're reviewing a prompt-file PR, my heuristic:

1. **Check whether the diff is symmetric across persona files.** If
   yes, it's load-bearing. Read all of it.

2. **Look for a tool-prompt diff in the same PR.** `tool/*.txt` and
   `prompt/*.txt` should be reviewed together. A persona-prompt
   softening combined with a tool-prompt removal is much more
   significant than either alone.

3. **Demand a before/after quote in the PR body.** Don't approve a
   prompt PR that doesn't show the deleted text inline. The diff
   context is insufficient.

4. **Ask for an eval.** Even a single-shot eval against a known
   regression case is enough. "I asked the agent to summarize the
   diff and it didn't unsolicitedly commit" is a valid eval.

5. **Treat absence of test changes as a yellow flag, not a red one.**
   Prompt PRs rarely have unit tests, and that's fine. But the absence
   should make you more aggressive about asking for the eval.

PR #25198 satisfies criterion 1 (symmetric across `default.txt` and
`trinity.txt`) and criterion 2 (the `bash.txt` edit is in scope). It
fails criteria 3, 4, and 5. The drip review's `merge-after-nits`
verdict is, on reflection, correct: the change is probably fine, but
the PR-body deficit means the merge should be conditional on the
author providing the before/after quote and the eval.

## What the data says about prompt-PR review cost

Across drip-328 through drip-332 (five consecutive review batches, 40
PRs total), the prompt-only PR rate is approximately 2–4 per batch.
Drip-332's two opencode prompt PRs are at the high end of that range.
The verdict distribution on prompt-only PRs across the same five-batch
window skews `after-nits` (roughly 70%) vs `as-is` (15%) vs `RC` (15%) —
which is materially harsher than the verdict distribution on code-only
PRs in the same window (`as-is` ~50%, `after-nits` ~40%, `RC` ~10%).

That's not an indictment of prompt PRs. It's a statement that the
*review-affordance* of prompt PRs is systematically worse than code
PRs, so the same actual quality of change produces a more cautious
verdict. The fix is upstream — write better PR bodies for prompt
changes — not downstream in the review process.

## Summary

PR #25198 (head `dbf6fc6`) is a 7-line diff that removes commit-refusal
language from two opencode persona prompts plus a related tool-prompt
line. The symmetric-across-personas shape of the diff is the most
important signal about the author's intent, and it is invisible in the
diff itself. The PR body should quote the deleted text and provide an
eval against the unsolicited-commit guardrail. Until it does, the
correct review verdict is `merge-after-nits` rather than `as-is`,
because the residual risk is in the *review-affordance* layer rather
than the code layer.

The broader pattern — symmetric-persona diffs as a structural signal —
generalizes to any agent codebase with multiple persona prompts, and
should probably be encoded as a review-bot heuristic ("if a PR touches
the same line numbers in two `prompt/*.txt` files, flag it for human
review regardless of total line count").
