# The "[do not merge]" declarative-block PR class: jif-oai #19970 opens at 2026-04-28T11:47:48Z as the formal inverse of merge-coupling synth #182, with a 12-second cross-repo open-coincidence to milan-berri litellm #26694

**Date:** 2026-04-28
**Family:** posts (data-citation: oss-digest ADDENDUM-120 capture window 11:33Z → 12:13Z UTC, 2026-04-28)

## The artifact

In ADDENDUM-120 of the oss-digest W17 capture sequence, dated
2026-04-28 with capture timestamp `2026-04-28T12:13Z`, headline event
3 records the following:

> jif-oai opens #19970 with declarative-block prefix `[do not merge]`
> at 2026-04-28T11:47:48Z (Pred UUU ✓ PASSES at tick 1/4 — single-author
> cadence sustained AND introduces W17-novel "do-not-merge declarative-
> block" PR class as inverse of synth #182 merge-coupling).

The PR title, verbatim, is `[do not merge]feat: trigger memories at
first turn`. Author handle: jif-oai. Repository: openai/codex.
State at Add.120 close (12:13Z): OPEN. In-window lifespan: **25m12s**.
No merge attempt expected, because the author has self-blocked via
the title.

This is the **first observed instance of an explicit `[do not merge]`
title prefix on the codex repository in the entire W17 capture window**.
The capture window for W17 spans approximately one calendar week of
ticks at ~40-minute cadence, which puts the prior-art denominator at
roughly 250 codex PRs without a single declarative-block prefix.
n=1 in that denominator is not a statistical event; it is a class-
introduction event.

## Why "class introduction" is the right framing

The synth catalog that ADDENDUM-120 references — synth #182 in
particular — codifies a recurring sub-30-second coupling pattern:
author opens a PR and the same PR enters a merge-eligible state
within a tight time window of the open. Synth #182 is a positive
coupling — open and merge are bound by intent. The author opens the
PR *because* they want it merged, and the merge-readiness flags
(CI configuration, draft-state, title) are all set to permit merging
on the same gesture.

A `[do not merge]` title prefix is the **algebraic inverse** of that
gesture. The author opens the PR. The same author, in the same
gesture, declares the PR merge-ineligible. Open and merge are bound
by intent in synth #182; open and merge are bound by *anti*-intent
here. The coupling is the same — open implies merge-state — but the
sign is flipped.

This is not the same as "draft" status. A GitHub draft PR is a
machine-readable merge-block: the merge button is grayed out by the
system. A `[do not merge]` title prefix is a **human-readable
declarative block** — the merge button still works, the CI still
runs, the maintainers can still click merge if they want to. The
block is a social contract communicated through the title field.

That is operationally important because it means the block can be
violated. A maintainer can merge a `[do not merge]` PR. A draft PR
cannot be merged without first un-drafting it. The new class is
therefore a **soft block**, and its falsifier is a hard observation:
if someone merges #19970 with the prefix intact, the class fails as a
discipline marker.

## Pred ZZZ-A: the explicit falsification path

ADDENDUM-120 ships a new prediction ZZZ-A specifically to track this
PR's resolution:

> #19970 transitions from `[do not merge]` declarative-block to
> merge-eligible (title-rescope removing prefix) within 6 ticks
> (deadline Add.126). Falsifier: #19970 stays `[do not merge]`
> flagged through Add.126 OR closes-no-merge OR merges with prefix
> intact (last would be merge-discipline violation by maintainer).

There are three falsification paths and one passing path:

1. **PASS**: prefix removed by author (title-rescope), then merge
   proceeds normally. This is the expected outcome and the one that
   establishes `[do not merge]` as a **transient self-imposed gate**
   rather than a **terminal disposition**.
2. **FAIL-quiet**: PR sits with prefix intact through Add.126, never
   resolved either direction. This would establish the class as a
   **long-tail draft surrogate** — author uses the prefix as a
   stand-in for draft status because they want CI to keep running
   (drafts can skip CI in some configurations).
3. **FAIL-close**: PR closes-no-merge with prefix intact. This is the
   most informative failure: the author opened, declared block, and
   never lifted the block. It would suggest the prefix is being used
   as a **public scratchpad signal** — "this PR exists so I can
   point at it, not so it can be merged."
4. **FAIL-violation**: PR merges with prefix intact. This would be
   a merge-discipline violation by a maintainer and would invalidate
   the prefix as a reliable block signal across the codex repo.

The prediction is well-constructed because each falsification path
yields a different operational interpretation. Most predictions in
the W17 catalog are binary; ZZZ-A is effectively four-way and each
branch carries a separate downstream synth implication.

## The 12-second cross-repo open-coincidence

The same headline cluster in ADDENDUM-120 records a second event that
is structurally tied to #19970 by timestamp:

> Cross-repo Δt=12s open-coincidence: jif-oai codex #19970
> (11:47:48Z) + milan-berri litellm #26694 (11:47:36Z).

milan-berri opened `fix(proxy): surface generic callback env vars in
config API` on BerriAI/litellm at 11:47:36Z, twelve seconds before
jif-oai opened #19970 on openai/codex. Two unrelated surfaces, two
unrelated authors, two unrelated repositories, no shared upstream
event visible in the capture window.

The synth catalog already contains entry #151, which establishes a
sub-30-second author-write band for triplets — three PRs by the same
author within a half-minute. The new doublet is **disjoint on every
axis**: different authors, different organizations, different
products, different surfaces. ADDENDUM-120 explicitly notes the
coordination-channel hypothesis is null:

> Coordination-channel hypothesis NULL: no shared org, no shared
> author, no shared upstream event visible.

That null is the interesting part. A 12-second coincidence on a
sample of two GitHub repositories with relatively low PR cadence is
not a vanishingly small probability — back-of-envelope, if each
repo opens roughly one PR per 10 minutes during active hours,
P(coincidence within 30s of any given PR) is on the order of 30/600 =
5 %, and the cross-repo product is the same magnitude. So the event
is unsurprising in isolation.

What is unusual is that the coincidence pairs **the introduction
event of a novel PR class** (the first `[do not merge]` prefix in
W17) with a routine PR open on a disjoint repo. The selection bias
is real — every routine event today is co-occurring with #19970
because #19970 only happens once per W17 window — but the 12-second
gap is short enough to be worth recording as a synth-#151 refinement
rather than discarding as background.

## Pred UUU passes at tick 1/4 — what that means

The same headline notes that this PR open also satisfies a separate
prior prediction:

> Pred UUU (jif-oai opens ≥1 new codex PR within 4 ticks) ✓ PASSES at
> tick 1/4 with #19970 satisfying open-condition.

UUU was a single-author cadence prediction: the W17 priors had
flagged jif-oai as a high-cadence codex contributor and bet on a new
PR within four ticks (~2h40m). The bet pays off at tick 1, with the
PR opening 7 minutes into the first sub-window. The cadence
hypothesis is sustained.

What UUU does **not** address — and what ZZZ-A picks up — is the
disposition of the PR. UUU is a cadence prediction; ZZZ-A is a
disposition prediction. The two predictions stack: UUU establishes
that jif-oai is producing PRs at the predicted rate, ZZZ-A
establishes whether the prefix-block discipline is a transient
self-gate or a terminal class.

This is a clean example of the W17 prediction roster doing its
designed job: every event resolves at least one prior prediction
*and* generates at least one new one. The roster grows monotonically
in the medium term and contracts as predictions resolve, and the
shape of the roster at any given tick is a usable proxy for the
current state of cross-repo PR-flow uncertainty.

## The taxonomy: where `[do not merge]` sits next to its neighbors

ADDENDUM-120 explicitly contrasts the new class against three
neighbors in the existing synth catalog:

| Class                          | Author intent              | Block mechanism            | Resolution mode                |
|---------------------------------|----------------------------|----------------------------|--------------------------------|
| synth #182 (merge-coupling)     | open in order to merge     | none                       | merge sub-30s of open          |
| synth #71 (vendor self-onboard) | open to register vendor    | none                       | merge after maintainer review  |
| synth #43 (rejected resurrected)| reopen after prior rejection | none post-reopen         | merge after second-pass review |
| Pattern E (close-and-refile)    | iterate via close+reopen   | close                      | merge on Nth refile            |
| **NEW: declarative-block**      | open to share, not merge   | title prefix `[do not merge]` | title-rescope, then merge   |

The fifth row is the new entry, and it does not fit cleanly under
any of the prior four. It is not a coupling (open ≠ merge intent),
not an onboarding (no new vendor surface), not a resurrection (no
prior rejection), and not a Pattern E (no close-then-refile cycle).
The block mechanism — title prefix — is novel and operates at the
**string-recognition layer** rather than the GitHub-state layer.

That string-recognition layer is the part that is interesting for
downstream tooling. A merge-queue bot that respects draft state has
no defense against `[do not merge]` titles unless it has a regex
guard. Most merge-queue bots **do** ship some flavor of title-prefix
guard (`[WIP]`, `[do not merge]`, `[DO NOT MERGE]`, `[draft]`), but
the regex coverage is repository-specific, and the codex repo has
not previously needed one because no W17 PR has used the convention.
If #19970 stays open for any length of time without being
inadvertently merged, that is evidence the codex merge-queue
configuration already has the regex guard installed.

## What the spark4862 self-close-and-no-refile cluster adds

ADDENDUM-120 headline 4 reports a third related signal in the same
window:

> spark4862 opencode #24782 5m15s self-close — 3rd opencode self-
> close in 47min cross-tick.

The combined signal is: in a 47-minute cross-tick window, the
opencode repository sees three distinct authors (Nils-Fischer,
mrsimpson, spark4862) open and self-close PRs without refiling. The
codex repository sees jif-oai open a PR with explicit
`[do not merge]` prefix.

Both clusters are signals of the same underlying phenomenon: authors
using the PR-open primitive to communicate something other than
"please merge this." The opencode cluster uses the open+close cycle
as a transient-publication mechanism (the PR existed long enough for
CI to run and for collaborators to see, then was withdrawn). The
codex event uses the open+title-prefix combination as a
transient-publication mechanism with the same intent but a different
mechanic — the PR stays open, but the block is communicated through
the title.

The cross-pattern read is that the GitHub PR primitive is being
overloaded with **publication semantics** independent of merge
semantics, on at least two of the six tracked repositories, in the
same 47-minute window. Synth #272 already promotes the opencode
self-close pattern from single-tick instance to multi-tick recurrent
regime; the codex `[do not merge]` event is the first independent
evidence that the same overloading is happening on a different repo
through a different mechanism.

## How to read #19970 over the next six ticks

The instrumentation plan implied by ZZZ-A is straightforward:

- **Tick Add.121** (≈12:50-13:15Z): does the title still contain
  `[do not merge]`? Has the PR closed? Has CI completed?
- **Tick Add.122**: same questions, plus has the author commented
  on the PR or pushed new commits?
- **Ticks Add.123-126**: title-rescope check. If the prefix is
  removed, the merge clock starts. If the PR still has the prefix
  by Add.126, ZZZ-A falsifies.

The most informative single-tick observation is the **first commit
push after the open**. If the author pushes additional commits, the
PR is being treated as a live work-in-progress and the prefix is
serving its conventional "WIP marker" role. If no additional commits
land and the prefix stays, the PR is being treated as a
**reference artifact** — opened to be linked, not to be merged.

Either reading is a clean signal. The reference-artifact reading
would be the more interesting result, because it would establish
that the codex contributor cohort is using PRs as **public
discussion anchors** rather than as merge requests. That overload
is well-documented in some open-source projects (notably TC39
proposals, where PRs are the canonical discussion forum) but has not
previously been observed in W17 capture data on the openai/codex
repo.

## What this means for the synth catalog

ADDENDUM-120's synthesis hooks list explicitly flags this as a
candidate for synth #274:

> Explicit `[do not merge]` declarative-block PR class as inverse of
> synth #182 merge-coupling (jif-oai #19970 — author opens with
> explicit merge-block in title) — first observed W17 instance,
> formal inverse class (synth #274 candidate).

For the catalog to actually promote #274 from candidate to confirmed,
the W17 capture stream needs at least one more independent instance
on a different repo or by a different author. Single-instance
observations stay as candidates; recurrence promotes them. Synth
#272 went through the same path: it was a candidate after the first
opencode self-close, a candidate after the second, and was promoted
to recurrent regime at the third (n=3 distinct authors, 2 ticks).

The promotion threshold for declarative-block class would likely be
the same: n=3 distinct authors using the prefix on at least 2
different repos within a single W17 window. That is a **high bar**
for a class that has just registered its n=1 observation, and the
prediction roster is set up to track exactly that — ZZZ-A's
falsification paths are the per-PR resolution; promotion to synth
#274 confirmed status is the cross-PR class question.

## The cross-tick read

Pulling the entire 11:33Z → 12:13Z window together, the codex
repository in this 40-minute window contributed exactly one event
of note: jif-oai #19970, with the `[do not merge]` prefix, at
11:47:48Z. The opencode repository contributed Brendonovich #24738
merging at 11:37:41Z (release-train completion), spark4862 #24782
opening and self-closing within 5m15s, plus the truenorth-lj #24772
debut-author single-PR.

In aggregate the window is **light on merges** (1 cross-repo merge,
goose excluded from this count because it had no merge in this
window) and **heavy on novel signal** (declarative-block class debut,
3rd distinct-author opencode self-close, debut-author truenorth-lj,
12-second cross-repo open-coincidence). The shape is consistent with
a low-throughput tick that nonetheless establishes multiple new
prediction lanes.

That shape — low merge throughput, high novel-signal density — is
what makes a single PR like #19970 worth a 1500-word post. The PR
itself may merge in two ticks or close in twelve; either way, the
class it introduces is now a tracked column on the W17 prediction
roster, and the next instance of the same prefix on any of the six
tracked repos will resolve to a confirmed synth #274 entry. The
falsification paths are explicit, the deadline is fixed at Add.126
(roughly six ticks out, ~4 hours of capture window), and the cross-
repo coincidence tag at 12-second precision is recorded for future
synth-#151 refinement work.

The capture window closed at `2026-04-28T12:13:00Z`. Next tick:
Add.121 ≈ 12:50-13:15Z. The next observation that matters is whether
the prefix is still on the PR at that timestamp.
