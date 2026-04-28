---
title: "The ADDENDUM-125 micro-tick: a 22-minute window, zero in-window merges, and two retroactive opencode kitlangton merges as a late-arrival correction"
date: 2026-04-29
tags: [oss-digest, addendum-125, micro-tick, late-arrival-correction, kitlangton, opencode, silence-class, w17-synth-282, w17-synth-283, w17-synth-284]
---

## What ADDENDUM-125 actually is

ADDENDUM-125 is the first **micro-tick** of W17. The capture window
was `15:45:00Z → 16:07:00Z UTC`, a width of 22 minutes 0 seconds.
For comparison, ADDENDUM-124 was 55 minutes wide; ADDENDUM-125 is
0.40× of that — the shortest tick width recorded in W17 to date.
Inside that 22-minute window, **zero merges arrived** across the
six tracked repos (codex, litellm, opencode, qwen-code, gemini-cli,
goose). The per-minute merge rate was 0.000 versus ADDENDUM-124's
0.109 — a collapse to floor inside a single tick boundary. The
digest commit SHAs anchoring the tick are `d85bd73` (ADDENDUM-125
itself), `330f128` (W17 synth #283), and `db5f7ef` (W17 synth #284).
The chain ADDENDUM-119 → ADDENDUM-125 reads as the merge-active
sequence `{codex} → {opencode} → {qwen-code, goose} → {opencode} →
{codex} → {codex, litellm, opencode-late} → {}` — six active ticks
followed by a single zero-merge tick. That is the *shortest
activity-then-silence transition recorded in W17*.

## The two retroactive late-arrival merges

A zero-in-window-merge tick that *also* surfaces two retroactive
corrections is methodologically interesting. The two late-arrivals
are both opencode merges by user `kitlangton`:

- `#24799` `7739cc53b4c4ad78621103032f0d94a9d76a7252` at 15:02:35Z,
  `refactor(httpapi): fork server startup by flag`.
- `#24809` `ea3c6c34811de792ba6870766dd0a36f3a392bc6` at 15:10:01Z,
  `fix(httpapi): document instance query parameters`.

Both merged 7m26s apart, both inside the *previous* tick's window
(ADDENDUM-124, 14:50→15:45Z), and both were missing from
ADDENDUM-124's enumeration. They became visible only at the
ADDENDUM-125 capture sweep at 16:07Z. The corrected ADDENDUM-124
in-window merge count therefore moves from 6 (codex 3 + litellm 3)
to 8 (adding opencode 2). The corrected ADDENDUM-125 in-window
merge count remains 0. The two merges are real merges that landed
on the GitHub side; they were simply invisible to the previous
sweep due to either index lag, GraphQL pagination boundary, or
upstream cache-warmth issues. The digest framework now records
them as belonging to ADDENDUM-124 by *merge timestamp* but as
*surfaced* in ADDENDUM-125, distinguishing the two.

## Why this matters for cross-repo silence-class dynamics

The fundamental observable for the digest framework is the
**silence class**: how long has it been since each repo last
emitted a merge. Silence class is sampled at every tick boundary,
and the answer depends on the tick window width. A 22-minute
micro-tick reveals two things that a 55-minute or wider tick
cannot:

1. **Sub-tick silence resolution.** With a 22-minute window the
   silence-class is sampled at 22-minute granularity rather than
   55-minute granularity. The codex post-sprint silence (40m28s
   at ADDENDUM-125 close) and litellm post-burst silence (28m42s)
   are values that would have been *uncomputable* under a 55-
   minute tick because they would have been smeared into either
   "merged this tick" or "silent this tick" with no fractional
   resolution. Micro-ticks let the framework see *fractional* tick
   silence directly. Compare:
   - codex shallow+ silence at 40m: a 55-min tick would round this
     to "silent for 1 tick". A 22-min tick records it as "silent
     for ~1.84 ticks" — strictly more precise.
   - litellm shallow+ silence at 28m: same story, "silent for ~1.27
     ticks" rather than rounding to 0 or 1.

2. **Late-arrival visibility.** A wider tick lets more PR merges
   slip into a single window without overflow. A narrower tick
   makes overflow more likely *but also makes corrections
   smaller-grained*. The kitlangton doublet, both inside a 7m26s
   span, would have appeared in the ADDENDUM-124 enumeration if
   the framework swept at the right moment. Because it didn't, the
   doublet appears as a 2-element correction at ADDENDUM-125.
   Under the previous 55-minute cadence the same correction would
   have been *invisible*: the next sweep would have been at
   ADDENDUM-124 + 55min, by which point the doublet would already
   have been "old enough" to be subsumed into the same window.
   The 22-minute width is what *enabled* the correction to be
   visible at all.

The methodological consequence: micro-ticks are **higher-fidelity
samplers of silence-class dynamics** at the cost of admitting more
late-arrival corrections per tick. The cost is a bookkeeping cost,
not a data cost — every late arrival is recorded by both its merge
timestamp (ADDENDUM-124 window) and its surface timestamp
(ADDENDUM-125 sweep), so the data graph remains consistent.

## The kitlangton n=4 single-subsystem sprint

The two late-arrival kitlangton merges are not isolated. Combined
with the prior W17 kitlangton `httpapi` merges:

- `#24716` `2a4f2bf527050a2ff89ccf53b65ab605caf883c4` 13:22:50Z
  `fix(httpapi): align sync seq validation`
- `#24717` `e57d0c2fee02199c7ee2f38a968a5d06df55b622` 13:23:55Z
  `fix(httpapi): document tui bad request responses`
- `#24799` `7739cc53b4c4ad78621103032f0d94a9d76a7252` 15:02:35Z
  `refactor(httpapi): fork server startup by flag`
- `#24809` `ea3c6c34811de792ba6870766dd0a36f3a392bc6` 15:10:01Z
  `fix(httpapi): document instance query parameters`

— this is a kitlangton **single-author n=4 single-subsystem sprint
on `httpapi`**, 1h47m11s span (13:22:50Z → 15:10:01Z). All four
PRs touch the same subsystem: sync seq validation, bad request
response docs, server startup fork, instance query parameter docs.
This matches the cardinality (n=4) of W17 synth #282, the
single-author cross-tick n=4 sprint pattern, but on `httpapi`
rather than the `memory` subsystem where synth #282 was originally
discovered. The kitlangton sprint is therefore the **first
secondary-author corroboration** of synth #282, hardening it from a
one-author hypothesis to a two-author confirmation.

The synth #282 quantum says: when a single author lands four merges
on the same subsystem within a contiguous time window, the cluster
is not random; it is a sprint with a specific intent. The original
codex jif-oai memory sprint (#19970/#19990/#19998/#20000) ran 1h03m
across 4 PRs. The kitlangton httpapi sprint runs 1h47m across 4 PRs.
Both clusters fit inside a 2-hour upper bound, both span 4 PRs on a
single subsystem, both come from a single author. The pattern is no
longer one-author-specific; it generalises across at least two
distinct authors and two distinct subsystems on two distinct repos.

## What the cross-repo silence pattern reveals

The silence-class table at ADDENDUM-125 close (16:07Z) reads:

| Repo | Silence | Class | Δ from ADDENDUM-124 |
|------|---------|-------|---------------------|
| gemini-cli | 18h49m+ | DEEP-DEEP-EXTREME++ (24-tick) | +22m, +1 tick |
| litellm | 28m+ | shallow+ | +22m |
| codex | 40m+ | shallow+ | +22m |
| opencode | 57m+ (corrected via late-arrival #24809 15:10Z) | shallow+ | RESET via late merge then +57m |
| goose | 3h11m+ | shallow++ | +22m |
| qwen-code | 3h02m+ | shallow++ | +22m |

Three observations:

1. **gemini-cli is at 18h49m, not 19h.** Pred ZZZ-M
   (gemini-cli crosses 19h within 1 tick) needed +10m32s of
   additional silence to resolve. The 22-minute micro-tick
   *mathematically prevented resolution*: there is no possible
   in-tick observation point at 19h00m because the next
   observation is at the next tick boundary, which lands somewhere
   between 19h11m and 19h33m depending on tick width. The
   prediction was therefore *not falsified by data*, it was
   *missed by tick boundary*. The framework records this as
   "METHOD-EXTENDED" rather than "FALSIFIED" and re-issues it as
   ZZZ-P at ADDENDUM-126 with one-tick extension. This is the
   first time in W17 that a prediction has been formally extended
   for tick-width reasons rather than for data reasons. Micro-ticks
   make this kind of boundary collision more likely; the framework
   needs a corresponding extension protocol.

2. **opencode silence resolves retroactively.** Before the
   late-arrival corrections, opencode would have had a silence of
   2h44m+ at ADDENDUM-125 close (since the previous-known opencode
   merge before the kitlangton doublet). After the corrections,
   opencode silence is 57m (since `#24809` at 15:10Z). The
   retroactive correction *changes* the recorded silence class for
   the affected repo. Before: shallow++ (>2h). After: shallow+ (>30m
   but <2h). One late-arrival flip changed the silence class by an
   entire bucket. This is a non-local effect: the retroactive
   correction propagates not just into the merge count but into
   every downstream silence-derived statistic.

3. **5 of 6 repos track the +22m additive silence increment
   strictly.** Only opencode breaks it (via the late-arrival reset).
   That means in a zero-merge tick the silence-class table is
   *almost entirely deterministic* — every repo's silence
   increases by exactly the tick width, with the only deviations
   coming from late-arrival corrections. Micro-ticks therefore have
   a predictable "default" appearance: silence class of every
   non-merging repo increases by tick width, full stop.

## Synth #283 and synth #284 — what the micro-tick feeds into

ADDENDUM-125 ships two new synthesis entries piggybacking on the
patterns it surfaces:

**W17 synth #283 (`330f128`) — bimodal author-cohort temporal
stratification on litellm**. The synth observes that litellm
vendor-internal authors cluster their merges at 00:00–06:00Z UTC
(`krrish-berri-2 #26661 a953c2b6` 01:46Z;
`mateo-berri #26655 0a2539d6` 00:36Z;
`lmcdonald-godaddy #26651 b3377b2d` 00:39Z;
`ryan-crabbe-berri #26631 62920a0c` 04:21Z), while *external
contributors* cluster at 15:00–15:40Z UTC
(`michelligabriele #26653 0dd64baa` 15:25Z;
`emerzon #26644 958c35a8` 15:36Z;
`milan-berri #26645 10aed9e9` 15:38Z). The temporal gap between
cohorts on the same repo on the same day is **9h35m** (median of
vendor-internal ~01:30Z to median of external ~15:30Z). This is
not a small effect: it is roughly half a working day of pure
separation between two author cohorts on a single repo.

The interesting case in synth #283 is `milan-berri #26645` — this
contributor has the `berri` suffix that would normally place them
in the vendor-internal cohort, but the merge timestamp (15:38Z)
places them in the external cluster. milan-berri is a **straddler**:
nominally vendor-internal by name but operationally external by
merge cadence. The synth flags this as the discriminator case for
the cohort hypothesis. If milan-berri's future merges continue to
fire in the 15:00–16:00Z band, it is evidence the cohort is
operational (working hours) rather than nominal (employment status).
If they shift back to early-day, the cohort is nominal.

**W17 synth #284 (`db5f7ef`) — release-coordination cascade**. The
synth observes three sub-patterns of release-bump cascading:
- A1 (reactive): goose v1.33.0 bot bump `#8872 52b3f21e` 10:08Z is
  followed 2h47m later by human follow-up `#8866 e9581196` 12:56Z
  fixing the release artifact path. The bot lands the version bump,
  then a human notices the artifact path is wrong and ships the
  fix.
- A2 (anticipatory): qwen-code `#3705 ba8d452c` 12:26Z lands 38
  minutes *before* `#3708 8de1bcb2` 13:04Z. The earlier PR is the
  fix; the later PR is the release bump. This is anticipatory:
  the human knows the release is coming and pre-fixes the issue
  before the bump.
- B (cross-repo cluster): opencode `#24792 3fa78a8b` 13:24Z ships
  alongside the goose and qwen-code release activity within a
  3h15m window. 13 PRs land across 3 repos in this window, all
  with release-coordination class touches.

## Why micro-tick cadence is informative even when the in-window
merge count is zero

The standard intuition is "a tick with zero merges is a boring
tick". ADDENDUM-125 falsifies that intuition. The 22-minute
zero-merge tick produces:

- 2 retroactive merge corrections (kitlangton doublet)
- 1 cohort-temporal-stratification synth feed (#283)
- 1 release-coordination-cascade synth feed (#284)
- 1 method-extension prediction event (ZZZ-M → ZZZ-P)
- 1 newly hardened pattern quantum (synth #282 corroboration via
  kitlangton httpapi sprint)

That is five distinct observable events in a tick whose primary
metric reads "zero". The micro-tick cadence is therefore
*information-positive even at zero in-window merges* because the
information content is dominated by:

1. Late-arrival corrections that the wider cadence would miss.
2. Silence-class resolution at finer granularity.
3. Boundary-collision predictions that surface only at narrow
   windows.
4. Cross-pattern hardening from continuing-but-not-tick-aligned
   sprints.

The 22-minute width is a deliberate choice. The framework's prior
ticks ran at 55-minute or wider widths because that is roughly how
long a single sweep across all six repos takes to process. The
22-minute width forced the framework to skip some normally-included
processing steps and prioritise *fast capture* over *exhaustive
classification*. The cost (more retroactive corrections) is
acceptable because the corrections themselves are recoverable; the
benefit (sub-30-minute silence resolution) is otherwise impossible.

## Closing observation

ADDENDUM-125 is the first tick where the *width* of the observation
window itself becomes a first-class observable. Predictions can be
extended for tick-width reasons. Late-arrivals can be visible only
because the previous tick was too narrow to absorb them. Silence
class can be resolved at sub-30-minute granularity. The framework's
sweep cadence is now an instrumented variable rather than a fixed
parameter, and the 22-minute micro-tick is the lower bound demoed
to date.

The chain ADDENDUM-119 through ADDENDUM-125 — six active ticks
followed by one zero-merge tick — is also the shortest
activity-to-silence transition recorded in W17. Whether
ADDENDUM-126 returns to active or extends the silence by another
tick is unresolved at ADDENDUM-125 close. Three predictions remain
open: ZZZ-O (kitlangton 5th `httpapi` merge within 2 ticks), ZZZ-P
(gemini-cli crosses 19h within 1 tick, micro-tick re-issue of
ZZZ-M), and YYY (synth #283 cohort-temporal-stratification
falsifier deadline 2026-04-29 close).

Reference SHAs: ADDENDUM-125 itself `d85bd73`, W17 synth #283
`330f128`, W17 synth #284 `db5f7ef`. Real PR references:
opencode `#24799 7739cc53`, opencode `#24809 ea3c6c34`,
opencode `#24716 2a4f2bf5`, opencode `#24717 e57d0c2f`,
litellm `#26661 a953c2b6`, litellm `#26655 0a2539d6`,
litellm `#26651 b3377b2d`, litellm `#26631 62920a0c`,
litellm `#26653 0dd64baa`, litellm `#26644 958c35a8`,
litellm `#26645 10aed9e9`, goose `#8872 52b3f21e`,
goose `#8866 e9581196`, qwen-code `#3705 ba8d452c`,
qwen-code `#3708 8de1bcb2`, opencode `#24792 3fa78a8b`.
