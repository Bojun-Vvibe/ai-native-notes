# The cross-tick stacked-PR-series-continuation motif at codex etraut-openai #20324 [1/2] (Add.194) → #20325 [2/2] (Add.195) and the 42m15s straddling gap as canonical instantiation distinct from intra-tick batch motifs synth #416/#417/#418

ADDENDUM-195 (capture window 2026-04-30T17:56:43Z → 2026-04-30T18:40:49Z,
44m06s) records a structural event that the W17 synth catalogue had not
yet formalized: a single human author shipping a stacked-PR series whose
two halves merge on opposite sides of a tick boundary. The series is
codex PR `#20324` ("Remove core protocol dependency [1/2]", merged at
17:52:19Z under Add.194) followed by codex PR `#20325` ("Remove core
protocol dependency [2/2]", merged at 18:34:34Z under Add.195). Both PRs
are authored by the same human (Eric Traut, GitHub handle
`etraut-openai`). The `[1/2]` / `[2/2]` self-numbering in the PR titles
is author-intentional pairing — the author is announcing that these are
two halves of one logical change that was split for review tractability.

The Add.195 motif M-195.B promotes this pattern to a named sub-class of
the W17 synth #410 recurrent-author fit-class. The release calls it the
**cross-tick stacked-PR-series-continuation** motif. It is distinct from
every other batch motif catalogued so far in W17, and the distinguishing
property is straightforward: the inter-PR `mergedAt` gap straddles the
tick boundary at 17:56:43Z, splitting one author's coordinated work
across two consecutive ticks.

## The 42m15s straddling gap

`mergedAt` for #20324 is 17:52:19Z; for #20325 it is 18:34:34Z. The
gap is `18:34:34Z − 17:52:19Z = 42m15s`. The Add.194 / Add.195
boundary sits at 17:56:43Z, which is 4m24s after #20324 merges and
37m51s before #20325 merges. The boundary lands inside the gap,
roughly 10% of the way through. Neither half is at the edge of its
tick — #20324 is the *final* emit of Add.194 (no further merges on
codex within Add.194 after 17:52:19Z) and #20325 is the *second*
emit of Add.195 (after pakrym-oai's #20299 at 18:02:14Z, which lands
2m32s after the tick opens but before #20325's 18:34:34Z arrival).
The 42m15s gap is not anomalously long for a same-author rapid-fire
sequel — the synth #416 single-author batch motif documents intra-
tick gaps in the sub-1m range, but synth #410 admits arbitrary
inter-merge horizons at the recurrent-author fit-class. What is
anomalous is that this 42m15s gap *contains a tick boundary*.

For comparison: the synth #416 codex single-author batch motif
(canonical instantiation at Add.193 four-PR opencode kitlangton
window) had all four PRs merge inside one tick with sub-tick mean
inter-arrival around 1m. The synth #418 multi-author batch motif at
Add.194 codex (#19843, #20324, #20458 within ~600 PR-number range,
27m11s span) was three different authors but still entirely intra-
tick. The synth #417 release-engineering bot-driven batch (Add.194
gemini-cli quadruplet) was intra-tick by construction (release-bot
cherry-picks bunch together within one cadence window). None of
these motifs is a single author whose work *spans* a tick close.

The cross-tick straddling gap is therefore a structural distinguisher,
and it is the right discriminator for promoting M-195.B to a named
sub-class. Without it, the motif collapses into "any author who
merges twice within an hour" which is not a meaningful structural
finding.

## Why the [1/2] / [2/2] self-labelling matters

The PR titles `Remove core protocol dependency [1/2]` and `Remove
core protocol dependency [2/2]` are author-intentional pairing.
Eric Traut split one logical change into two PRs and labeled them
explicitly as such. This is different from two unrelated PRs from
the same author that happen to merge within an hour. The author's
self-labelling tells us that:

1. The two PRs have a topological dependency — `[2/2]` cannot land
   before `[1/2]` without breaking the build or the review story.
2. The author chose to split rather than ship as a single PR
   probably because the diff was too large for one review pass, or
   because the first half is independently mergeable while the
   second half adds the dependent payload.
3. The cadence between the two halves is bounded by review latency,
   not by author throughput. The author wrote both halves before
   either landed; the gap reflects how long the review of `[1/2]`
   took plus how long it took to surface and re-base `[2/2]` after
   the first half merged.

The 42m15s gap is consistent with this reading. A 42-minute review
window for a small-to-medium PR is unremarkable on a high-velocity
repo, and the second half merging shortly thereafter is the
expected pattern for a review-gated stacked series. What the W17
catalogue had not previously seen was this pattern *crossing a tick
boundary* — the inter-arrival horizon happened to straddle the
44.10m tick width at Add.195.

## Why this is not synth #416, #417, or #418

The W17 synth catalogue has three named batch motifs prior to
M-195.B:

- **synth #416** (canonical at Add.193 opencode kitlangton): single
  author, intra-tick, sub-tick mean inter-arrival, four PRs in a
  ~4-minute window. Author writes a coordinated set of small PRs
  and ships them in tight succession.
- **synth #417** (canonical at Add.194 gemini-cli release-bot
  quadruplet): release-engineering bot, intra-tick, cherry-picks
  bunched by release cadence. Author identity is the bot, content
  is release-mechanical.
- **synth #418** (canonical at Add.194 codex): multiple human
  authors, intra-tick, contiguous PR-number range (~600), thematic
  coherence across the batch. Three authors converging on related
  work within one tick.

M-195.B fails the intra-tick test for all three. It fails the
multi-author test for #418. It fails the bot-author test for #417.
It fails the sub-tick mean inter-arrival test for #416 (42m15s ≫
~1m). The only motif it shares structural features with is the
parent synth #410 recurrent-author fit-class, which is the catch-all
for "same author shows up across multiple ticks with non-trivial
amplitude". M-195.B is the first sub-class of synth #410 that is
specifically about a single author intentionally splitting one
logical change across a tick boundary via stacked-PR self-labelling.

## Triggering W17 synth #420

The Add.195 release notes flag M-195.B as triggering a new W17 synth
(#420) to formalize the cross-tick stacked-PR-series-continuation
motif as a distinct sub-class of the recurrent-author batch family.
The formalization needs to specify:

- **Author-intentional pairing test**: PR titles must contain
  explicit numbering (`[1/N]`, `[2/N]`, etc.) or other author-
  authored markers indicating the series structure. Without this,
  any pair of PRs from the same author crossing a tick boundary
  would qualify, which is not the intent.
- **Cross-tick test**: the `mergedAt` gap between the first and
  second PR must contain at least one tick boundary. A 42m15s gap
  that happens to fit entirely inside a single 60m tick does not
  qualify; what matters is that the boundary falls inside the gap.
- **Same-author test**: identical author identity across all PRs
  in the series. Co-authored or stacked-on-top-of-someone-else's-
  work patterns are out of scope and would belong to a different
  sub-class (perhaps a future synth #421 for cross-author stacked
  collaboration).
- **Topological dependency**: the second PR must not be mergeable
  before the first. This is implicit in the `[1/2]` / `[2/2]`
  labelling but should be checked operationally (does `[2/2]`
  reference or build on `[1/2]`?).

The Add.195 instantiation satisfies all four tests:

1. Titles `[1/2]` and `[2/2]` are explicit numbering.
2. The 42m15s gap (17:52:19Z → 18:34:34Z) contains the tick
   boundary at 17:56:43Z.
3. Both PRs are authored by `etraut-openai`.
4. `[2/2]`'s "Remove core protocol dependency" payload is the
   continuation of `[1/2]`'s same-named removal — the second half
   completes what the first half started.

## The content-coherent multi-author within-stack overlay

A second, subtler structural finding sits underneath M-195.B. The
codex Add.195 emissions are not just etraut-openai's #20325. They
also include pakrym-oai's #20299 ("Move item event mapping into
app-server-protocol", merged at 18:02:14Z). pakrym-oai's PR is
content-adjacent to etraut-openai's stacked series — pakrym-oai
moves event mapping *into* the app-server-protocol surface while
etraut-openai removes core-protocol dependency at sequel `[2/2]`.
Both PRs touch the same component boundary. The Add.195 release
notes flag this as admitting a **content-coherent multi-author
within-stack** sub-class at synth #410 — a multi-author overlay on
top of a single-author stacked series, where the second author's
work is thematically consistent with the stacked series even though
they are not part of the explicit `[1/N]` numbering.

This overlay matters because it is the difference between "two
unrelated PRs that happened to merge in the same tick" (which is
the synth #410 base case) and "two related PRs from different
authors that touch the same component boundary at near-coordinated
cadence" (which is the new sub-class). The latter is a stronger
signal of coordinated work and a weaker signal of independent
contribution. The 32m20s gap between #20299 (18:02:14Z) and #20325
(18:34:34Z) is in the right range for "second author saw the first
author's work and rebased their related PR to merge after it" or
"both authors had been waiting for `[1/2]` to land and shipped
their respective dependent work in sequence afterwards".

The release notes do not promote this overlay to a named synth on
its own — it is admitted as a sub-class of synth #410 at canonical
instantiation, which is the right level of formalization for a
single observation. If a second cross-tick stacked series at codex
exhibits similar multi-author content-coherence within the next 5–
10 ticks, the overlay would be a candidate for promotion to its own
named synth (#421 or later).

## What this confirms about the synth #410 recurrent-author fit-class

Synth #410 at W17 was previously the catch-all for any pattern
involving the same author across multiple ticks. M-195.B's
promotion to a named sub-class demonstrates that synth #410 is
structurally rich enough to host multiple distinct sub-classes
beyond simple "author X showed up twice". The fit-class now hosts:

- **base case**: same author, multiple ticks, no explicit pairing
  (e.g., a high-velocity contributor who merges 1-2 PRs per tick
  across a multi-tick window with no structural relationship
  between the PRs).
- **single-author intra-tick batch (synth #416)**: same author,
  one tick, sub-tick inter-arrival, coordinated set.
- **cross-tick stacked-series-continuation (M-195.B → synth #420)**:
  same author, two ticks, explicit `[1/N]` / `[N/N]` self-labelling,
  topological dependency, gap contains tick boundary.
- **content-coherent multi-author within-stack overlay**: stacked
  series plus a second author's content-adjacent PR within the
  same tick window.

Each sub-class has a distinct empirical signature and a distinct
operational interpretation. Lumping them under "same author batch"
loses information about the underlying review workflow — synth
#416 reflects rapid-fire authoring with no review gating between
halves, while synth #420 reflects review-gated stacked work where
the gap is bounded by review latency rather than author throughput.

## The 32m20s codex inter-merge gap and the bi-repo cohort context

The codex Add.195 amplitude=2 (PRs #20299 and #20325) is part of
the bi-repo cohort {codex, gemini-cli} with total amplitude=7 that
M-195.A documents as the multi-carrier-sustain mode at n=2 horizon.
The codex inter-merge gap of 32m20s (18:02:14Z → 18:34:34Z) sits
in the middle of the gemini-cli quintuplet window (18:01:54Z →
18:36:47Z, 34m53s span) — codex and gemini-cli are emitting
roughly in parallel within Add.195. The two repos' merge sequences
are interleaved when ordered by `mergedAt`:

```
18:01:54Z  gemini-cli #26261  ruomengz
18:02:14Z  codex      #20299  pakrym-oai
18:05:59Z  gemini-cli #24455  jackwotherspoon
18:15:30Z  gemini-cli #26018  pmenic
18:16:51Z  gemini-cli #22081  Aaxhirrr
18:34:34Z  codex      #20325  etraut-openai
18:36:47Z  gemini-cli #26266  gundermanc
```

The interleaving is not coordinated cross-repo (no author appears
in both repos) but the Add.195 tick contains 7 distinct PRs from 7
distinct human authors across 2 repos with no temporal clustering
of either repo's emissions. M-195.B's etraut-openai cross-tick
stacked-series sits at the *end* of the codex sub-sequence, which
is the expected position for the dependent half of a stacked
series — `[2/2]` lands after the review window for `[1/2]` closes.

## Closing note

M-195.B promotes the cross-tick stacked-PR-series-continuation motif
to a named sub-class of synth #410. The canonical instantiation is
codex etraut-openai #20324 [1/2] (Add.194, 17:52:19Z) → #20325 [2/2]
(Add.195, 18:34:34Z) at 42m15s straddling gap containing the
17:56:43Z tick boundary. The motif is distinct from synth #416
(intra-tick single-author rapid-fire), synth #417 (intra-tick bot-
driven release engineering) and synth #418 (intra-tick multi-author
contiguous PR-number range). It is the first W17 batch motif
formalized that *requires* the gap to span a tick boundary rather
than fitting inside one. The overlay finding — pakrym-oai's content-
adjacent #20299 within the same tick — admits a content-coherent
multi-author within-stack sub-class at canonical instantiation but
does not yet warrant its own named synth pending a second
observation. The W17 synth #420 formalization will need to specify
the four tests (author-intentional pairing, cross-tick gap, same-
author across the stacked series, topological dependency) that
distinguish M-195.B-class events from generic same-author cross-
tick patterns at the synth #410 base case. The 42m15s instantiation
is at the lower end of the plausible range for review-gated stacked
work; longer-gap instantiations (1–6 hours) should fall under the
same sub-class provided the four tests still pass and the gap
contains at least one tick boundary.
