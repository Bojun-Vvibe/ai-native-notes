---
title: "Close-and-Refile as a Cross-Repo Anti-Pattern: Three Repos in 80 Minutes"
date: 2026-04-25
tags: [oss-digest, code-review, governance, anti-patterns, pr-workflow]
---

# Close-and-Refile as a Cross-Repo Anti-Pattern

The 2026-04-25 oss-digest addendum (window 00:34Z → 01:33Z) caught
something that's worth pulling out of the noise and naming as a
recurring shape: **three independent repositories — `openai/codex`,
`BerriAI/litellm`, and `anomalyco/opencode` — exhibited the
"close-and-refile" PR pattern within an 80-minute window, with deltas
ranging from 16 seconds to 11 minutes between the close of the original
PR and the open of its byte-identical replacement.**

The three concrete events:

- `openai/codex#19470 → #19473` — `mchen-oai`, *"Add turn start
  timestamp to turn metadata"*, δ = 11 minutes (closed 00:35:06Z,
  refiled 00:46:06Z).
- `BerriAI/litellm#26462 → #26465` — `ishaan-berri`, *"feat(teams):
  per-model team member budgets"*, δ = 12 seconds (closed 00:48:07Z,
  refiled 00:48:19Z).
- `anomalyco/opencode#24223 → #24238` — `v1truv1us`, *"docs: sync env
  vars with source code"*, δ = 16 seconds (closed 01:00:39Z, refiled
  01:00:55Z).

All three: same author closing and reopening, same byte-identical
title, no acknowledgment in either PR body that the close was
deliberate. The litellm case escalated further over the next ninety
minutes — a *third* refile of the same title (#26471, opened 01:56:47Z)
appeared *before* #26465 had been closed at 02:02:19Z, producing a
five-minute window in which two PRs with the same title from the same
author existed simultaneously. The pattern stopped being a sequence of
clean replacements and became an overlapping double-jump.

This post argues that close-and-refile, treated individually, looks
like a workflow gesture; treated as a class across repos, it is a
**provenance-loss anti-pattern** with measurable downstream costs to
review, audit, and downstream automation. It deserves the same kind of
named-and-counted treatment that "force-push to a PR branch" got five
years ago.

## What close-and-refile actually destroys

When you close a PR and refile the same change as a new PR, the
following artifacts evaporate from the GitHub-side record, even though
the diff is preserved in the new PR:

1. **Reviewer history.** Comments on the closed PR are still
   accessible by URL but no longer surface in the new PR's
   conversation thread. A reviewer who left a "this looks good but
   please rename the variable" comment on PR A is invisible to
   anyone landing on PR B for the first time. The next reviewer
   either repeats the comment or, more commonly, doesn't.
2. **CI history.** The CI runs on PR A — including, critically,
   the *failing* runs that motivated whatever fix produced PR B —
   are no longer in the obvious place to look. The new PR shows a
   fresh CI history, possibly green from the start, with no
   indication that the same change has been re-tested.
3. **Linear discussion thread.** Inline review comments, design
   discussions, and the back-and-forth that justifies the change
   are split across two PR URLs. A future archaeologist trying to
   understand "why did this land?" has to find both PRs and read
   them in order, *and* know that the order matters.
4. **Reviewer attribution.** Approvals on the closed PR don't
   carry over. If PR A had two approvals at the moment of close, PR
   B starts with zero, and merge-policy automation that tracks
   "two approvers required" treats this as a fresh review surface.
5. **Cross-link integrity.** Issues that referenced PR A still
   reference PR A, not PR B. The "fixes #1234" semantics may or
   may not survive depending on which PR carries the magic
   keyword.

The δ = 16 seconds case (opencode #24223 → #24238) makes the cost
profile starkest. Sixteen seconds is not enough time for any of the
above artifacts to have *accumulated* on the closed PR — there were no
reviews, no CI runs, no cross-links to lose. So why close and refile?
The answer is almost certainly that the original PR was opened with
some defect (wrong base branch, wrong target, missed file, bad commit
message) and the author preferred to re-create it rather than fix it
in place. **In that case, the close-and-refile is benign for
artifacts, but it leaves a misleading audit trail**: the GitHub event
log shows "PR opened, then closed, then a new PR opened" rather than
"PR amended". Any tooling that counts PR throughput (and many
governance dashboards do) will double-count.

The δ = 11 minutes case (codex #19470 → #19473) is the dangerous one.
Eleven minutes is enough time for at least one CI run to have started
on the closed PR, and possibly for an early reviewer to have left a
comment. The new PR throws all of that away. If the close-and-refile
is happening because the author got fast feedback on PR A and
preferred to re-do the work cleanly, that's a value judgment about
review hygiene; if it's happening because the author wants to escape
a CI failure or reviewer pushback by starting fresh, that's a
governance problem.

## Why three repos in 80 minutes is the interesting unit

Any single instance of close-and-refile is a workflow detail. The
reason to escalate it to a named anti-pattern is that **the digest
caught it across three unrelated repositories with three unrelated
authors in the same observation window**. That is not an accident of
sampling. It is what `oss-digest` is built to surface — patterns that
are invisible at the repo level but obvious at the cross-repo level.

The repos involved are not coincidentally related: `openai/codex`,
`BerriAI/litellm`, and `anomalyco/opencode` share an author audience
(developers building or integrating LLM tooling), share review
practices that are influenced by the same set of OSS norms, and
share — increasingly — the same automated agents acting on PRs. When
three repos exhibit the same anti-pattern in the same window, the
parsimonious hypothesis is *not* that three independent humans
independently chose the same workflow; it's that the **enabling
condition** for close-and-refile is something the three repos share.

Candidate enabling conditions include:

- **Cheap PR creation.** All three repos accept PRs from many
  contributors, and the cost of opening a new PR is low enough that
  it's faster than fixing an existing one. (This is true of nearly
  all OSS, so it's a permissive condition, not an explanatory one.)
- **Weak close-as-failure norm.** None of the three repos appear to
  treat "PR closed without merge" as a noteworthy event in their
  governance posture. There's no review-required acknowledgment, no
  bot comment asking why, no friction to closing. Closing a PR is as
  cheap as opening one.
- **Author tolerance for own provenance loss.** All three cases were
  same-author close-and-refile. The author chose, in the moment, to
  trade the artifact history of their own closed PR for the
  cleanliness of starting over. They are the ones bearing the cost,
  and they decided the cost was small.
- **Tooling that doesn't notice.** No bot in any of the three repos
  flagged "this title is byte-identical to a PR you closed N
  seconds/minutes ago". The pattern is invisible to the automation
  layer.

The fourth one is the most actionable. A trivially small bot — one
that watches for `pull_request.closed` followed by `pull_request.opened`
within some window from the same actor with a Levenshtein distance
threshold on the title — would catch this pattern with high precision.
The bot doesn't need to *block* anything; it just needs to leave a
comment on the new PR linking to the old one and requiring a one-line
acknowledgment. That's enough to convert "silent close-and-refile"
into "documented close-and-refile", which preserves the artifact
chain.

## The litellm overlapping double-jump

The litellm case deserves separate treatment because it crossed a
threshold the other two didn't.

The sequence:

1. `#26462` opened by `ishaan-berri`, *"feat(teams): per-model team
   member budgets"*.
2. `#26462` closed at 00:48:07Z.
3. `#26465` opened at 00:48:19Z, same author, same title — δ = 12
   seconds. Standard close-and-refile.
4. `#26471` opened at 01:56:47Z, same author, same title.
5. `#26465` closed at 02:02:19Z.

Steps 4 and 5 are the new shape: the third refile (#26471) appeared
**five minutes and thirty-two seconds before the second refile
(#26465) was closed**. For roughly 5.5 minutes, two PRs with the same
author and the same title were live simultaneously. This is not a
sequence; it is an overlap.

What does an overlapping refile mean? It means the author opened a
new PR while the old one was still nominally the canonical place to
review the change. Reviewers hitting the repo's PR list during that
5.5-minute window would have seen two entries indistinguishable by
title, both from the same person, both presumably with similar
diffs. The discoverability cost is direct: a reviewer might
plausibly comment on the wrong one, and the author then has to
either (a) move the comment, (b) close the one with the comment and
lose it, or (c) leave the comment stranded on a PR that's about to
be closed.

The single-author overlap also has a worse property than sequential
refile: it suggests the close was triggered by the open, not the
other way around. The author opened #26471, decided it was the new
canonical version, and *then* closed #26465. The closed PR became a
casualty of the new PR's existence rather than a deliberate
withdrawal in its own right. That's a different operational story
and arguably worse from a provenance standpoint, because the closed
PR's review history is not just lost but lost *as a side effect* of
an unrelated decision.

The litellm `/team/*` surface as a whole, in the same digest
window, had **five concurrent PRs from at least four authors**:
`#26464` (yuneng-berri, MERGED), `#26465` (ishaan-berri, the
overlapping refile), `#26466` (shivamrawat1, guardrails), `#26467`
(yuneng-berri, pass-through hardening), and `#26468` (Michael-RZ-Berri,
[WIP] bulk key updates). Four authors converging on one surface in
under 80 minutes is itself a notable pattern; the close-and-refile
on the budgets PR happened *inside* that convergence, suggesting
that the author was reacting to scope-related collisions with the
other concurrent PRs and trying to keep their own slice of the
surface "fresh". That's a plausible motivation, but it doesn't
reduce the audit cost.

## What to do about it

I don't think close-and-refile can or should be banned. There are
legitimate cases (wrong base branch with no in-place fix path,
sensitive content in the original branch that needs to be re-pushed
from a clean history, accidental fork-vs-branch confusion) where
refile is the correct move. But the cases I just enumerated are
**explicit** decisions that the author should be willing to
document. The defensible policy is therefore:

1. **Detect.** A bot watches for same-author, same-title (Levenshtein
   ≤ 3, normalized for whitespace and punctuation), `closed → opened`
   transitions within a configurable window (start with 60 minutes;
   the litellm overlap shows you need at least that). This is cheap.
2. **Annotate, don't block.** When detected, the bot leaves a comment
   on the new PR linking to the old one and asking for a one-line
   reason. Default reasons could be checkboxes: "wrong base branch",
   "rebased onto main", "scope split", "started over after review",
   "other (please explain)". The author picks one.
3. **Preserve the link.** The bot also adds a `Refiled-from: #N` line
   to the PR description, so cross-references are repairable later.
4. **Surface in metrics.** Governance dashboards count refiled-pair
   PRs separately from fresh-author-PRs and from
   force-pushed-author-PRs. This stops dashboards from
   double-counting throughput.
5. **Escalate on overlap.** If the new PR opens *before* the old one
   closes (the litellm pattern), the bot raises the urgency of the
   annotation — the author needs to decide which of the two is
   canonical *now*, not after a reviewer comments on the wrong one.

None of this requires repo policy changes or contributor-facing
friction. It requires one bot that none of the three repos
currently runs.

## What this signature is, exactly

Close-and-refile is, formally, a **rename operation that GitHub's
event model can't represent**. Git itself handles this in commit
history (you amend or rebase), and the file system has rename
semantics (you `mv` a file and tools track the rename). PRs do not.
The platform forces an author choosing to re-do their PR into a
two-step (close + open) that loses information by construction. Any
author who hits this enough times learns the workaround, and the
workaround propagates.

The fact that it propagated to three repos in 80 minutes,
independently, with three different authors, suggests the workaround
is now load-bearing in the OSS LLM-tooling ecosystem. Naming it as a
pattern is the first step. Counting it is the second. Designing
around it (the bot above) is the third. The 2026-04-25 digest
window happened to make the pattern visible all at once; on most
days, it shows up as a single repo's quirk and gets ignored.

---

*Source: `oss-digest/digests/2026-04-25/ADDENDUM.md`, addendum 2,
window 00:34Z → 01:33Z. Three close-and-refile instances cited:
`openai/codex#19470 → #19473` (δ=11min), `BerriAI/litellm#26462 →
#26465` (δ=12s), `anomalyco/opencode#24223 → #24238` (δ=16s). litellm
overlapping double-jump documented in same addendum, sequence
`#26462 → #26465 → #26471 (overlap with #26465) → #26465 closed
02:02:19Z`. Five concurrent `/team/*` PRs from four authors in
litellm during the same window: #26464, #26465, #26466, #26467,
#26468.*
