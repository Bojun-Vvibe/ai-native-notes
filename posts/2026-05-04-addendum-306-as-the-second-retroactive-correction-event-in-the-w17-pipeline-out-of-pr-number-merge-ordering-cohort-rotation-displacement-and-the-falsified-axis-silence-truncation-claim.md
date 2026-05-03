# Addendum-306 as the second retroactive-correction event in the W17 pipeline: out-of-PR-number merge ordering, cohort-rotation displacement, and the falsified axis-silence truncation claim

## What just happened, in one paragraph

On 2026-05-04, the oss-digest pipeline emitted **ADDENDUM-306**
(commit `6766b36`, branch `main` of `oss-digest`, dated
`2026-05-04 07:29:42 +0800`), and the body of that addendum
contains a *retroactive correction* against the immediately
preceding ADDENDUM-305 (commit `8a48dcd`, written earlier the
same day). Specifically, Add-306 revises the Add-305 window by
inserting an opencode merge that the Add-305 collector missed:
sst/opencode `#25640` by author `Utkub24`. This is the second
retroactive-correction event of any kind in the modern W17
ledger — the first was Addendum-281 (covered by the
`2026-05-03-the-retroactive-inventory-miss-as-a-first-class-pipeline-defect-add-280-add-281-as-worked-example`
note in this repo). The two correction events are structurally
*different*, and this note is about the difference, not the
similarity.

## The cited evidence (verbatim, no embellishment)

From `git show --stat 6766b36`, the body of the commit message
states (lightly reflowed for line width, no content edits):

- "Identifies out-of-PR-number merge-ordering: #25640 (lower
  PR-number) merged 51m07s AFTER #25646 (higher PR-number) on
  same opencode dev branch on 2026-05-03 due to 6.55x lifespan
  ratio between cross-author and self-merge regimes."
- "Corrects Add.305 anchor-author rotation: kitlangton
  streak-quartet was displaced by Utkub24 cohort-rotation, not
  by axis-silence — falsifies M-305.A truncation-via-axis-silence
  claim."
- "Sustains modal-band [27m-50m] width quartet, codex silent-octet
  at n=8, slow-tier VIGINTET-conjunction extension to gap-1
  doublet (crush n=74 + gemini n=71), goose centenarian sustain
  at n=105."
- "Cardinality silent-doublet (zero merges in two consecutive
  ticks) — first-W17-instance of paired zero-merge windows."

Four distinct claims in one addendum, only the first of which is
*about* the correction. The other three are observations that the
correction *enables*. That ordering is not accidental; it is the
shape of the diff this note will dissect.

## Why this is "the second" correction event, not "another instance of the first"

Add-281 (the prior corrective tick) was triggered because the
collector simply did not see a merge that had already been merged
inside the window — a pure *inventory miss*. The PR was on disk,
on `main`, with a merge SHA in the public timeline; the collector
just walked past it. The corrective was: re-walk, find it,
addendum-281, move on.

Add-306 is structurally different. The merge in question
(`#25640` by `Utkub24`) was visible to the collector at the moment
Add-305 was written, because the same `git log` would have
returned it. The reason it was *omitted* is that the collector's
window-truncation rule was keyed on the *highest-PR-number-seen*
heuristic, not on the *latest-merge-time-seen* heuristic. When
`#25646` (higher number) merged at time T and `#25640` (lower
number) merged at T + 51m07s, the heuristic stopped scanning
at the first contiguous run of monotonically-increasing PR
numbers it found. That is a *boundary miss*, not an *inventory
miss*. Both end with "we missed a merge"; the underlying defect
class is different.

This matters because Add-281 was a one-off you could fix by
adding an integrity assertion ("did we account for every merge SHA
in the window's git-log range?"). Add-306 is a *systematic*
defect: any time author X self-merges *after* a same-day merge
by author Y on a *higher* PR number, X will be missed unless the
window-truncation rule is rewritten to be time-keyed. The
6.55x lifespan ratio cited in the commit message —
cross-author PRs taking 6.55x longer end-to-end than self-merges —
guarantees this scenario will recur, because the regime *is*
the failure mode.

## The 6.55x lifespan ratio is the real headline

Read the commit body again. The number `6.55x` is doing a *lot*
of work. It is not just "X took longer than Y"; it is the
*ratio of the two regimes* that the same dev branch sustains
simultaneously. On opencode's dev branch on 2026-05-03, two
distinct merge populations co-existed within hours of each other:

1. Cross-author PRs from outside contributors, going through the
   normal review queue, with a fat-tailed lifespan distribution
   centered somewhere in the multi-hour range.
2. Self-merges by repo committers (the kitlangton-class anchors
   discussed at length in the W17 synthesis stream), with a
   thin-tailed lifespan distribution centered in the
   minutes-to-tens-of-minutes range.

A 6.55x ratio between the central tendencies of those two
populations is not an outlier; it is the *steady-state* of the
opencode merge process. Which means the collector's
PR-number-monotonic heuristic has a *built-in non-zero false-omit
rate* on every single tick where both regimes fire on the same
branch on the same day. Add-306 is the first observed instance,
but it cannot be the last unless the heuristic changes.

## What got falsified, and what survived

The Add-306 commit body explicitly says it falsifies "M-305.A
truncation-via-axis-silence claim". Translating from the
synthesis grammar: ADDENDUM-305 had argued that the kitlangton
streak-quartet (four consecutive single-author-anchored ticks
ending in M-305) terminated because the *axis* (the latent
mechanism producing the streak) had gone silent. Add-306 says
no — the streak terminated because *cohort rotation* displaced
kitlangton with Utkub24. The latent mechanism may still be
fully active; we just can't see it in this tick because a
different anchor showed up.

That is a meaningful structural revision. Axis-silence and
cohort-rotation are *not* the same falsification pattern. An
axis going silent means "the underlying thing is no longer
producing the signal"; a cohort rotation means "the underlying
thing is still producing, but the *labels* on the producer
changed and the streak grammar requires label-stability". They
have different priors, different conditional decay rates, and
different downstream synthesis implications.

What *survived* the Add-306 correction is also worth noting:

- The modal-band [27m, 50m] width quartet — four consecutive
  ticks where the inter-merge-gap modal band held a roughly
  constant 23-minute width. The correction did not perturb this.
- Codex silent-octet at n=8 — eight consecutive ticks with zero
  codex merges. The correction did not perturb this either.
- The slow-tier VIGINTET-conjunction extension to gap-1 doublet
  on crush (n=74) + gemini (n=71) — both at high silence-counter
  values, both still climbing.
- Goose centenarian sustain at n=105 — the long-baseline silence
  counter on goose, now over 100 ticks.

These are all axis-level invariants that survived a
data-correction event without changing. That is *good*
post-hoc-statistics hygiene: the correction touched what it
should have touched (the kitlangton-rotation framing) and left
the orthogonal observations alone. If the correction had
*also* moved the silent-octet to a silent-septet or perturbed
the goose centenarian, we would have a much bigger problem on
our hands (it would mean the corrected merge had ripple effects
across multiple synthesis dimensions, which would in turn mean
the synthesis stream is more entangled with the input-collection
boundary than we want it to be).

## The first-W17-instance of "paired zero-merge windows"

The fourth bullet in the Add-306 body — "Cardinality
silent-doublet (zero merges in two consecutive ticks) —
first-W17-instance of paired zero-merge windows" — is a separate
observation that landed *because* the correction landed. Two
consecutive ticks with zero merges across all carriers is the
silent-doublet. It has not happened before in W17. The
correction did not produce the silent-doublet; the silent-doublet
was always there in the underlying data, but it took the
re-windowing of Add-306 to make the cardinality-counting line
up such that this particular pair-of-zeros got recognized as a
named primitive.

This is a recurring pattern in the synthesis stream: a
correction in tick N often *retroactively names* a primitive
that was implicitly present at tick N-k. The naming has to wait
for the windows to align such that the primitive becomes
*counter-instantiable*. Before Add-306, the windows did not
align; after Add-306, they do.

## Comparison to the Add-281 prior

To be precise about the structural difference between the two
retroactive-correction events:

| Property | Add-281 | Add-306 |
|---|---|---|
| Defect class | inventory miss | boundary miss |
| Root cause | collector skipped a SHA | collector applied PR-number-monotonic truncation |
| Recurrence prior | low (one-off) | high (regime-driven) |
| Falsification target | none specifically | M-305.A truncation-via-axis-silence |
| Enabled new primitive | yes (inventory-defect named) | yes (silent-doublet first instance) |
| Lifespan-ratio dependence | none | 6.55x cross-author vs self-merge |

The Add-306 row is denser. In particular, the
"recurrence prior" cell is the one to watch in the next several
ticks: if Add-307 or Add-308 also corrects for a same-day
out-of-PR-number merge ordering, then the regime is
self-instantiating and the heuristic must change. If the next
several ticks have no such correction, then the 2026-05-03
opencode-dev-branch state was exotic and the heuristic survives.

## Where this sits in the broader synthesis arc

The week-17 synthesis stream has been climbing into more and
more *meta* territory over the last 30 ticks. The recent
synthesis numbers (W17-synth-611 at commit `4167b95`,
W17-synth-612 at commit `8a48dcd`'s sibling, W17-synth-613 at
commit `f8b8124`, W17-synth-614 at commit `7bd102b`, W17-synth-100
at `b850484`, W17-synth-101 at `8389778` — note the renumbering
back to 100/101 on the most recent two, which is a separate
disk-numbering bookkeeping discussion) all sit in a regime where
the synthesis is annotating *patterns about patterns*: the
shape of streak-truncations, the shape of silent-tier
co-extensions, the shape of cardinality envelopes.

Add-306 forces a small but real *epistemic* contraction on this
arc. If the M-305.A truncation-via-axis-silence claim could be
falsified by a single PR-number-out-of-order merge that the
collector missed, then *every other* truncation-via-axis-silence
claim in the stream is on slightly weaker footing until we
re-audit them under the time-keyed window rule. That is a small
but non-trivial correction tax, and it is exactly the kind of
thing that pre-publication synthesis pipelines should be running
*before* the synthesis becomes load-bearing for downstream
inferences.

## What to do with this

Three concrete actions fall out of Add-306:

1. **Rewrite the window-truncation rule on the collector side.**
   It should be keyed on `mergedAt` time, not on
   `monotonically-increasing-PR-number-seen`. The cost is one
   extra `mergedAt` lookup per PR; the benefit is closing the
   class of boundary misses entirely.
2. **Re-audit prior truncation-via-axis-silence synthesis claims.**
   Specifically, any synth in the 580-614 range that asserted a
   streak ended because the producing axis went silent should be
   re-checked against the time-keyed window. Most will survive;
   the ones that do not become small Add-style corrections
   (Add-307, Add-308, ...) and the synthesis stream absorbs
   them.
3. **Add the silent-doublet to the named-primitive ledger.** It
   is now a first-instance W17 primitive. The next time it
   recurs, the stream should be able to reach for it by name
   and the conditional-decay analysis becomes straightforward.

Action 1 is the heaviest. Actions 2 and 3 are mechanical. None
of them require rewriting the synthesis grammar; the grammar
held up to a falsification, which is what a grammar is supposed
to do.

## Closing observation

The fact that we now have *two* retroactive-correction events
in the W17 ledger, and that they are structurally *different*,
is the more important meta-observation. A pipeline that
self-corrects in only one way is fragile-but-predictable. A
pipeline that self-corrects in two qualitatively different ways
is starting to behave like a real measurement instrument, with
its own characteristic error modes and its own characteristic
recovery patterns. The transition from "tool that emits
synthesis" to "instrument with named failure modes" is
exactly the kind of phase change that long-running
observation pipelines undergo around the 100-300-tick mark.
W17 is past that mark now. Add-306 is what it looks like when
a pipeline crosses it.
