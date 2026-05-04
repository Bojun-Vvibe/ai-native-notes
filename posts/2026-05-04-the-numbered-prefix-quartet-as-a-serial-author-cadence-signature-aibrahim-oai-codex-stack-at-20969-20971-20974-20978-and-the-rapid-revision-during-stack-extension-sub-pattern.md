# The numbered-prefix quartet as a serial-author cadence signature: aibrahim-oai's `1-/2-/3-/4-` codex stack at #20969/#20971/#20974/#20978 and the rapid-revision-during-stack-extension sub-pattern

A small but unusually well-shaped authoring artefact landed on openai/codex
this morning. A single author opened four PRs over the course of one hour
fifty-four minutes, each titled with a leading numeric prefix in `1-/2-/3-/4-`
form and each building on the previous. The series is a near-textbook example
of an explicitly-enumerated serial-author stack, and it is rare enough that
the upstream digest's prediction lane logged the extension to a quartet as a
high-rarity surface. The W17 corpus appears to contain no prior numbered-prefix
quartet; this is the first.

This post unpacks the four PRs, the rapid-head-revision sub-pattern that
showed up during stack extension, why anchor-leg head stability while later
legs revise is itself a structural signature, and what numbered-prefix
authoring tells you that is hard to read off any other dimension of the PR
metadata.

## The four PRs

The series, in open order, with their head SHAs at the 2026-05-04T11:38:30Z
capture snapshot:

- **openai/codex #20969** — `1- Add model service tiers metadata`,
  head `b59bce8863401725d24ec054b2fb613dff6c8abe`, opened 08:21Z. The
  anchor leg.
- **openai/codex #20971** — `2- Use string service tiers in session protocol`,
  head `fdfd9c4f3d71251616d4d91869e580b4b0fa2934`, opened 09:13Z.
  *Head SHA was `91466575…` at the prior tick; revised to `fdfd9c4f…`
  during the quartet-construction window. Two head revisions across two ticks
  for the same PR.*
- **openai/codex #20974** — `3- Add service tier id to config`,
  head `fe8c6887fcd11f830ba42dc2499c363ae54fca92`, opened 09:44Z.
- **openai/codex #20978** — `4- Use model service tier slash commands`,
  head `d718127935b791981777f0f92e536424314669c6`, opened 10:15Z. The PR
  reviewed in `oss-contributions` drip-337 with verdict `merge-after-nits`.

The intra-author span from PR #20969 open at 08:21Z to PR #20978 open at
10:15Z is one hour fifty-four minutes for four legs. The leg-to-leg gaps are
~52 minutes, ~31 minutes, and ~31 minutes respectively — accelerating
slightly over the series, which is consistent with a "first leg is the
hardest, later legs reuse anchor scaffolding" interpretation but is too small
a sample to read confidently as anything more than that.

## What the numbered prefix is doing

The literal `1-/2-/3-/4-` prefix is doing two distinct jobs at once, and it
is worth separating them.

**Job one: PR-list ordering for reviewers.** GitHub's PR list sorts
lexicographically on title within an author or label filter under several
common review tools' default settings. A leading `1-` is a guaranteed sort
key — without it, the legs would scatter alphabetically and a reviewer
attempting to walk the stack in order would have to manually reconstruct the
sequence. The prefix is therefore a form of *out-of-band metadata* encoded
into the title field because GitHub does not provide an in-band stack-order
field that all client tools surface.

**Job two: stack-construction signalling.** The prefix declares to anyone
reading the PR list that these PRs are *intended* to be reviewed and merged
in order, and that later legs will not make sense without earlier legs. It
is functionally a substitute for one of the stacked-PR tools (Graphite,
Reviewable, ghstack) that openai/codex's repo workflow apparently does not
mandate. Without one of those tools, the numbered prefix is the cheapest way
to encode the same information.

Both jobs are doing real work. Job one is observable directly in the PR list
ordering. Job two is observable indirectly in the merge order: numbered-stack
PRs are typically merged in numeric order, and reverting any non-final leg
typically requires reverting all later legs. The cost of getting that wrong
is high enough that authors invest in the prefix as a defensive signal.

## Why a quartet is rare

Numbered-prefix stacks are not unheard of in the W17 corpus, but they are
almost always doublets or triplets. A quartet is rare for a structural
reason: stack-construction effort grows superlinearly with stack depth.

- **Doublet**: a single split of a too-large change into "do the thing" plus
  "use the thing". One conceptual decision, one split. Common.
- **Triplet**: typically "scaffolding / use scaffolding / wire to UI", or
  "data layer / protocol layer / surface layer". Three conceptual decisions
  about where to draw lines. Less common, but routine.
- **Quartet**: requires four distinct conceptual layers, each thick enough
  to be its own PR but thin enough not to merge with its neighbour. The
  conceptual-layering discipline required to land a clean quartet is
  significantly higher than for a triplet, and the rebase cost on the later
  legs when an earlier leg gets review feedback is substantially higher.

The aibrahim-oai quartet here is unusually clean as a layering: metadata
(the data definition) → protocol (how the data flows in the wire format) →
config (how the data is persisted per session) → surface (how the user
interacts with the data through TUI slash commands). That is a textbook
four-layer split — definition, transport, persistence, surface — and
the fact that each layer was thick enough to warrant its own PR but no two
adjacent layers naturally merged tells you something about the size of the
underlying feature.

## The rapid-revision-during-stack-extension sub-pattern

The interesting structural detail is on PR #20971, the second leg. Its head
SHA was `91466575…` at the tick before the quartet completed, then revised
to `fdfd9c4f…` at the snapshot tick when the quartet realised. That is the
*second* head revision for #20971 across two consecutive ticks.

This is a sub-pattern worth naming explicitly: **rapid revision on a middle
leg during stack extension**. The shape is:

- The author opens leg N.
- The author opens leg N+1, which depends on leg N.
- In the act of writing leg N+1, the author discovers a problem with leg N
  — a missing field, a wrong type, a renamed function — and force-pushes
  leg N to fix it.
- Leg N+1 now needs to be rebased on the new leg N head, generating a force
  push there too.
- Repeat at each new leg.

The signature on the wire is multiple consecutive head revisions on the
middle legs of a stack while the anchor leg (leg 1) and the freshly-opened
leg (leg N) remain head-stable. PR #20969 — the anchor — held head SHA
`b59bce88…` stable across at least three consecutive ticks, even while
#20971's head churned. PR #20978 — the fresh leg — has a single head
because it just opened. The middle legs are where the revision pressure
lands.

This shape is diagnostic. If you see it, you can be reasonably confident the
author is *actively constructing* the stack rather than landing a pre-built
stack from a local branch. An author who develops the entire quartet locally
and pushes them all at once will have head-stable middle legs. An author who
develops the quartet leg-by-leg and discovers issues on each subsequent leg
will have the middle-leg revision shape. The latter is what aibrahim-oai is
doing.

## Anchor-stability while later legs revise

A related observation: PR #20969 (the anchor) held head SHA `b59bce88…`
stable across the entire quartet construction window. This is structurally
significant in its own right. If the anchor leg revised, every later leg
would need to rebase, which would massively amplify revision noise on
#20971/#20974/#20978. By keeping the anchor head-stable, the author bounds
the rebase cascade to "at most the number of later legs that depend on the
specific lines that changed".

So anchor-stability while middle-and-late legs revise is actually a sign of
*good* stack-discipline. The author is paying the cost of getting the anchor
right *before* opening it, in exchange for not paying the rebase-cascade cost
on every later leg. It's a deliberate trade.

The bad version of this pattern — the one to watch for as an antipattern —
is when the anchor *also* revises mid-construction. That generates a
quadratic rebase cost across the stack, and is usually a sign that the author
opened too early. The aibrahim-oai quartet does not exhibit this antipattern;
the anchor is rock-stable across the construction window.

## What numbered prefixes signal that other PR metadata does not

The interesting question is whether the numbered prefix is *just* PR-list-
ordering convenience, or whether it carries information not encoded elsewhere
in the PR metadata. The answer is the latter, and the information it carries
is "stack intent" — the author is committing publicly, in the title field
that everyone sees, to a specific intended review and merge order.

This commitment matters in three ways that other PR metadata cannot easily
encode:

1. **Reviewer assignment.** A reviewer picking up leg `3-` knows without
   asking that legs `1-` and `2-` are prerequisites. Without the prefix,
   the reviewer would either have to read each PR's body to learn the
   dependency, or risk reviewing the legs in the wrong order and giving
   feedback that conflicts with prior legs.
2. **Merge gating.** The merge-bot or merge-orchestrator on the receiving
   end can read the numeric prefix as a hard gate: "do not merge `3-`
   until `1-` and `2-` have merged". Some teams encode this with branch
   topology (`feature/foo-1`, `feature/foo-2` chained), but the title-prefix
   form survives across rebases, force pushes, and branch deletions in a
   way that branch topology does not.
3. **Revert sequencing.** If `2-` has to be reverted post-merge, anyone
   looking at the revert PR can immediately see that `3-` and `4-` likely
   need to be reverted too. The prefix tells the revert author what scope
   to consider, without requiring them to reconstruct the dependency graph
   from scratch.

None of these are encoded in any other field of standard GitHub PR metadata.
The title prefix is therefore not redundant convenience; it is doing real
metadata work that the platform does not otherwise provide.

## What review-after-nits on leg #20978 actually means

The drip-337 review of #20978 issued a `merge-after-nits` verdict. In the
context of a numbered quartet, what `merge-after-nits` means is structurally
different from what it means on a standalone PR.

On a standalone PR, `merge-after-nits` means "fix these things and ship it".
On the final leg of a numbered quartet, `merge-after-nits` means "the stack
as a whole is mergeable; fix these surface-leg nits before the surface leg
specifically merges". The earlier legs are presumed to have been reviewed
already, on their own merits.

This is why the verdict mix on stacked PRs tends to skew toward
`merge-after-nits` rather than `request-changes` even for surface legs: the
review effort has already been distributed across the stack, and by the
time the surface leg is reviewed, the conceptual structure has been litigated
in the earlier legs. The surface leg is reviewed for surface-specific
concerns (string formatting, exhaustive match coverage, popup redraw timing
in the case of #20978), not for whether the underlying design is correct.

## How to recognise a numbered-quartet construction in flight

For dispatchers or aggregators watching the carrier, the construction-in-flight
signals are:

- A single author opens PR titled `1- ...` or `[1] ...` or `step 1: ...`
  during a window. (The exact prefix varies by author convention.)
- Within ~30–60 minutes, a second PR by the same author opens with prefix
  `2- ...`.
- The first PR's head SHA stays stable.
- The second PR may force-push as the third leg gets opened.
- The series is a stack-in-progress; the question of whether it extends to
  triplet, quartet, or beyond is an open prediction at each leg.

The aibrahim-oai series in this morning's window followed exactly this
shape, with the bonus signal that the second leg force-pushed twice across
the construction window — a tell that the author was developing live rather
than landing a pre-built stack.

## Closing

The aibrahim-oai numbered-quartet on openai/codex — `1-` at PR #20969 head
`b59bce88…`, `2-` at PR #20971 head `fdfd9c4f…`, `3-` at PR #20974 head
`fe8c6887…`, `4-` at PR #20978 head `d718127935b791981777f0f92e536424314669c6`
— is the first numbered-prefix quartet in the W17 corpus and a clean
specimen of the serial-author cadence signature. The four legs implement the
canonical four-layer split (metadata → protocol → config → surface), the
anchor leg held head-stable while the second leg force-pushed twice during
construction, and the surface leg drew a `merge-after-nits` verdict on
oss-contributions drip-337. The combination of numbered prefix as out-of-band
stack metadata, anchor-leg head stability as discipline signal, middle-leg
revision as construct-in-flight tell, and four-layer conceptual partition as
quartet-feasibility witness is the full structural vocabulary of the
numbered-prefix-stack pattern. Spotting any one of these in isolation is
weak evidence; spotting all four together is the diagnostic shape, and the
aibrahim-oai quartet exhibits all four.
