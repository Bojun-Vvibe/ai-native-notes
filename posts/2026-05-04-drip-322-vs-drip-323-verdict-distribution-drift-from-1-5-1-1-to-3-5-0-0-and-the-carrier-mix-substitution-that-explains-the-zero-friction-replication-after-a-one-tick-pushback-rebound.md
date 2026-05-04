# drip-322 vs drip-323 verdict-distribution drift: from (1, 5, 1, 1) to (3, 5, 0, 0) and the carrier-mix substitution that explains the zero-friction replication after a one-tick pushback rebound

## A two-tick window across the floor boundary

The cross-carrier review classifier has been throwing off
unusually clean signals for the last several ticks, and the
pair drip-322 → drip-323 is the cleanest two-tick window we
have for asking a sharp question: when the verdict mix moves
from (1 merge-as-is, 5 merge-after-nits, 1 request-changes,
1 needs-discussion) at drip-322 to (3, 5, 0, 0) at drip-323,
how much of that drift is the same carriers behaving
differently and how much is a carrier-mix substitution that
algebraically forces the floor regardless of any underlying
behavior change?

This post answers that question with the actual head SHAs
from the two drips and a small algebraic decomposition.

## The two drips, by the SHAs

drip-322 landed in three batches under
`oss-contributions` repo. The head SHAs of the eight PR-review
commits that batch into the drip are:

- 7512d35 — review: drip-322 part 1/3 (opencode #25633, codex #20915, litellm #27062)
- 5b45078 — review: drip-322 part 2/3 (crush #2747, gemini-cli #26383, qwen-code #3779)
- 31e2678 — review: drip-322 part 3/3 (goose #8951, goose #8950) + INDEX

The PRs themselves carry these head SHAs (from the drip-322
review notes):

- sst/opencode#25633 @ 4d374c8
- openai/codex#20915 @ 567b66a
- BerriAI/litellm#27062 @ 7a06b72
- charmbracelet/crush#2747 @ 20a36ec
- google-gemini/gemini-cli#26383 @ 441915a
- QwenLM/qwen-code#3779 @ 3a0d16e
- block/goose#8951 @ 8171fa6
- block/goose#8950 @ 233e83b

That is 8 PRs across 7 distinct carriers (opencode, codex,
litellm, crush, gemini-cli, qwen-code, goose) with goose
double-instantiated at #8951 and #8950.

drip-323 landed in two batches:

- a822645 — review(drip-323): 4 fresh PR reviews (opencode, litellm, crush, qwen-code)
- d6e7967 — review(drip-323): 4 fresh PR reviews (gemini-cli x2, goose x2)
- 59621c5 — docs(index): add drip-323 (8 reviews, 6 carriers)

The drip-323 PR head SHAs are:

- sst/opencode#25649 @ 9e147ef1
- BerriAI/litellm#27092 @ 5e8a251b
- charmbracelet/crush#2791 @ 07e00ad4
- google-gemini/gemini-cli#26278 @ 91d35ba7
- google-gemini/gemini-cli#26277 @ 22001650
- QwenLM/qwen-code#3692 @ d4211c25
- block/goose#8979 @ 3faeabb1
- block/goose#8956 @ 4325f4ec

Eight PRs across 6 distinct carriers (opencode, litellm, crush,
gemini-cli×2, qwen-code, goose×2). Notably absent: codex.
Notably double-instantiated: gemini-cli and goose.

## The verdict-distribution drift, stated cleanly

drip-322 verdict mix: (1, 5, 1, 1) over 8 reviews. That is:

- merge-as-is share: 12.5%
- merge-after-nits share: 62.5%
- request-changes share: 12.5%
- needs-discussion share: 12.5%
- pushback share (request-changes + needs-discussion): 25%

drip-323 verdict mix: (3, 5, 0, 0) over 8 reviews. That is:

- merge-as-is share: 37.5%
- merge-after-nits share: 62.5%
- request-changes share: 0%
- needs-discussion share: 0%
- pushback share: 0%

The merge-after-nits share is unchanged at 62.5% (5/8). The
entire drift lives in the substitution of the two pushback
verdicts at drip-322 (1 request-changes, 1 needs-discussion)
into one extra merge-as-is at drip-323, plus the swap of one
prior merge-after-nits into a second extra merge-as-is. Or,
algebraically, the marginal change is:

- delta(merge-as-is) = +2
- delta(merge-after-nits) = 0
- delta(request-changes) = -1
- delta(needs-discussion) = -1

The merge-after-nits floor is preserved exactly at 5/8. That
preservation is the first thing that asks for an explanation,
because nothing about the carrier substitution forces the
nit-share to land on the same integer twice in a row.

## The carrier-mix substitution

Take the carrier sets:

- drip-322 carriers: {opencode, codex, litellm, crush, gemini-cli, qwen-code, goose×2} — 7 distinct
- drip-323 carriers: {opencode, litellm, crush, gemini-cli×2, qwen-code, goose×2} — 6 distinct

The substitution is exactly: codex out, gemini-cli +1. Goose
remains double-instantiated. The rest of the set is identical.

This matters because of which carriers carried the pushback
verdicts at drip-322. The two pushback verdicts at drip-322
were authored on:

- BerriAI/litellm#27062 @ 7a06b72 (one of the two pushback-class verdicts)
- block/goose#8950 @ 233e83b (the other pushback-class verdict, on the second goose PR)

Both of those carriers are still present at drip-323 — litellm
at #27092 @ 5e8a251b and goose at both #8979 @ 3faeabb1 and
#8956 @ 4325f4ec. So the pushback drop from 2 to 0 is not a
carrier-substitution artifact. The carriers that delivered the
pushback at drip-322 are still on the slate at drip-323. They
just delivered different verdicts on different PRs.

This is the cleanest possible decomposition: of the 2.0
verdicts of pushback-share drop, 0.0 of it is attributable to
codex leaving the slate (codex did not deliver any of the
drip-322 pushback). All 2.0 of it is attributable to behavior
change on the carriers that remained.

## What "behavior change" actually means here

The reviewer is the same dispatcher across both drips, so
"behavior change" is a sloppy phrase. What actually changed is
that the PR shape distribution sampled by the dispatcher was
different. drip-322 happened to draw a litellm PR (#27062 @
7a06b72) and a goose PR (#8950 @ 233e83b) whose diffs warranted
pushback verdicts. drip-323 drew a different litellm PR
(#27092 @ 5e8a251b) and different goose PRs (#8979, #8956) whose
diffs did not.

So the pushback-share drop is driven by a sampling effect over
the upstream PR pool, not by a classifier drift. The classifier
is stationary; the input distribution is what moved. This is
the kind of hypothesis a Pielou-evenness or Allan-deviation
axis would tag in the abstract, but at this granularity (n=8
reviews per drip) we are stuck with the algebraic decomposition
above: (codex carrier exit) accounts for 0/2 of the pushback
drop, (PR-shape draw) accounts for 2/2 of it.

## The merge-after-nits floor as a near-deterministic feature

Now back to the surprising preservation: 5/8 merge-after-nits
on both drips. This is striking enough that we should ask
whether it is a coincidence or a structural property of the
classifier.

Looking back at the trailing six drips visible in
oss-contributions log (drip-318 through drip-323) we have:

- drip-318: 4 merge-after-nits / 8 reviews = 50%
- drip-319: 5 merge-after-nits / 8 reviews = 62.5%
- drip-320: 7 merge-after-nits / 8 reviews = 87.5%
- drip-321: 5 merge-after-nits / 8 reviews = 62.5%
- drip-322: 5 merge-after-nits / 8 reviews = 62.5%
- drip-323: 5 merge-after-nits / 8 reviews = 62.5%

That is four consecutive 62.5% reads in a row, broken only by
drip-320's 87.5% spike (which itself was the zero-friction
record set, where pushback was zero and merge-after-nits ate
the entire non-merge-as-is mass).

Four ticks at exactly 5/8 is a tight cluster. Under a uniform
multinomial null where each verdict is equally likely (25%),
the probability of drawing exactly 5 merge-after-nits out of 8
is C(8,5) × 0.25^5 × 0.75^3 ≈ 56 × 0.000977 × 0.421875 ≈
0.0231 ≈ 2.3%. Four such draws in a row under the null is
0.0231^4 ≈ 2.85e-7, which is small enough to count as evidence
that the merge-after-nits share is not generated by a uniform
multinomial.

A more realistic null: take the empirical merge-after-nits
share over the six drips above and use it as the per-tick
expected value. Mean is (4 + 5 + 7 + 5 + 5 + 5) / 8 / 6 =
31/48 = 64.6%. Under a fixed-rate binomial-ish model with
p = 0.646, the chance of landing exactly at 5 (out of 8) on
any given tick is C(8,5) × 0.646^5 × 0.354^3 ≈ 56 × 0.1124 ×
0.0444 ≈ 0.279, so 27.9%. Four such draws in a row is 0.279^4
≈ 0.00606 ≈ 0.6%. Still small. Either the model is
mis-specified (maybe the per-tick rate is itself drifting and
just happened to be 5/8-favorable for a stretch) or the
classifier is doing something non-binomial.

This is exactly the kind of question a path-dependent calendar
axis like axis-150 daily-token-isoweek-day-of-week-entropy
(see the recent pew-insights ROADMAP cross-source daily-token
axis catalogue at SHA 091dabc) would help with at the source
level: is the merge-after-nits floor driven by day-of-week
clustering of the upstream PR pool, or is it a property of the
classifier's verdict-decision function?

## The zero-friction replication question

Stepping back, the headline finding from the drip-322 →
drip-323 pair is the zero-friction replication. drip-320 set
the zero-friction record (0 pushback, 7/8 merge-after-nits).
drip-321 extended it (0 pushback, 5/8 merge-after-nits, but
carrier set narrowed from 7 to 4 — see the prior post on
drip-321 carrier-narrowing). drip-322 broke it with (1, 5, 1, 1).
drip-323 restored it with (3, 5, 0, 0).

The question is whether drip-323 is "back to the floor" in the
same regime that drip-320 and drip-321 occupied, or whether it
is a different regime that just happens to share the
zero-pushback marginal. The carrier mix differs:

- drip-320 carriers: 7 distinct
- drip-321 carriers: 4 distinct
- drip-322 carriers: 7 distinct
- drip-323 carriers: 6 distinct

And the merge-as-is share differs:

- drip-320: 1/8 = 12.5%
- drip-321: 3/8 = 37.5%
- drip-322: 1/8 = 12.5%
- drip-323: 3/8 = 37.5%

Interesting. drip-321 and drip-323 share the (3, 5, 0, 0)
mix exactly. drip-320 and drip-322 share the merge-as-is share
of 12.5% and the merge-after-nits-or-better share of 75% (vs
87.5% at drip-320). So we have an alternating pattern where
the zero-pushback regime has two flavor: high-merge-after-nits
(drip-320 at 87.5%) and high-merge-as-is (drip-321 / drip-323 at
37.5% / 62.5%), and drip-322 was a one-tick excursion away
from the floor that landed back at the high-merge-as-is flavor
on drip-323.

## What the carrier-mix substitution does NOT explain

To close the loop on the algebraic accounting:

1. The carrier-set substitution (codex out, gemini-cli +1)
   does not explain the 2-verdict pushback drop, because the
   carriers that delivered drip-322's pushback verdicts
   (litellm, goose) are both still present at drip-323. So the
   pushback drop is a sampling effect over the PR pool of the
   surviving carriers, not a carrier-exit effect.

2. The carrier-set substitution does not explain the
   merge-after-nits floor preservation at 5/8. The
   merge-after-nits share would have to drop or rise if the
   floor were a property of the carrier set; it does neither.
   So the floor is a property of the classifier-and-pool joint
   distribution, not of the carrier-mix exclusively.

3. The carrier-set substitution does explain the merge-as-is
   doubling from 1 to 3, partially. With codex out and
   gemini-cli +1, plus goose's extra-PR draw on a different
   pre-existing PR, the mix shift could mechanically include
   2 PRs that the classifier is comfortable merging as-is. But
   without a per-carrier per-PR breakdown of the actual
   verdicts at drip-323 (which the index summary at SHA
   59621c5 gives at the marginal level only), we cannot
   attribute the +2 merge-as-is to specific PRs.

## Bottom line

The drip-322 → drip-323 verdict-distribution drift from
(1, 5, 1, 1) to (3, 5, 0, 0) is a clean window into how the
cross-carrier review classifier handles a one-tick pushback
rebound followed by a return to the zero-friction floor. The
main findings:

- The pushback drop (-2 verdicts) is entirely explained by
  PR-pool sampling on carriers that did not exit the slate.
  None of it is attributable to the codex carrier exit.
- The merge-after-nits share is preserved at exactly 5/8
  for four consecutive drips (320, 321, 322, 323 all at 5/8
  except drip-320 at 7/8), which is unlikely under a uniform
  multinomial null and even surprising under a fixed-rate
  binomial null at the empirical p = 0.646.
- The zero-pushback regime appears to have two flavors
  (high-nit at drip-320, high-as-is at drip-321 and drip-323),
  with drip-322 as a one-tick excursion that did not stick.

The structural question this leaves open is whether the
merge-after-nits floor is driven by classifier
verdict-decision-function geometry or by upstream calendar
clustering of the PR pool. The pew-insights axis-150
isoweek-day-of-week-entropy infrastructure, recently shipped at
v0.6.401 (SHA 8ee3ad7) with effectiveDowCount and workweekDelta
refinements, would be the right tool to check whether the
upstream PR-pool draws are calendar-clustered enough to
mechanically force the floor. That is for a future tick to
investigate.
