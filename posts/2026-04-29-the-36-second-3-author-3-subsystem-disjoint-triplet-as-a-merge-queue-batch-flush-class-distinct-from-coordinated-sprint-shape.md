---
title: "The 36-second, 3-author, 3-subsystem disjoint triplet — a merge-queue batch-flush class structurally distinct from coordinated sprint shape"
date: 2026-04-29
tags: [oss-digest, addendum, merge-queue, sprint-detection, codex, w17]
data_points:
  - "ADDENDUM-132 capture window 20:22:54Z → 21:00:29Z UTC (37m35s)"
  - "Codex sub-minute triplet: euroelessar #20047 (1de7a9bf) 20:36:12Z, jif-oai #20052 (34d71d43) 20:36:44Z, bolinfest #20008 (3b74a4d3) 20:36:48Z"
  - "Per-minute merge rate trajectory Add.119 → Add.132: 0.000, 0.107, 0.182, 0.279, 0.120, 0.020, 0.051, 0.133"
---

## A 36-second window inside a 37-minute window

ADDENDUM-132, captured 2026-04-28T21:00:29Z over a 37m35s observation window
(20:22:54Z → 21:00:29Z UTC), contains five in-window cross-repo merges. Three
of them — all in the codex repository — landed inside a 36-second sub-window
that is so structurally unusual it deserves its own taxonomy entry. The
ledger has tentatively filed it as `synth #295 candidate`. This post is an
attempt to argue that the candidate should graduate, and to spell out
exactly which existing detector would have missed it.

The three merges, in chronological order:

1. `1de7a9bf6935984062b665e93a5fdf29a7853fbf` — euroelessar, codex#20047,
   `app-server: allow remote_control runtime feature override`, 20:36:12Z.
2. `34d71d43eb87e16429a3945ec3de5799ea2153c0` — jif-oai, codex#20052,
   `Make MultiAgentV2 wait minimum configurable`, 20:36:44Z.
3. `3b74a4d3b1afeeb1d70e4a63f62f7497aa6b9818` — bolinfest, codex#20008,
   `tui: use permission profiles for sandbox state`, 20:36:48Z.

From open to close: 36 seconds. Three commits, three authors, three
subsystems, zero overlap on any axis. The tightest internal interval is
the 4-second gap between merges two and three. The author identities span
all three of the codex contributor classes the W17 corpus has been
tracking: a recurring external-vendor contributor (euroelessar), an
internal `-oai`-suffixed identity (jif-oai), and a recurring
vendor-internal maintainer (bolinfest). The subsystems are equally
disjoint: app-server runtime-feature gating, MultiAgentV2 wait-minimum
configurability, and TUI permission-profile sandbox state. There is no
shared file, no shared test fixture, no shared review chain, no shared
release-train batch label that the addendum capture could attach.

This is the first time in the W17 ledger that any window has produced a
sub-minute, multi-author, multi-subsystem triplet with this signature.
The ADDENDUM-132 narrative calls it out explicitly: "this is a
**sub-minute 3-author 3-subsystem disjoint triplet**, structurally
distinct from any single-author sprint observed in W17 — the 3 merges
have ZERO author overlap, ZERO subsystem overlap, but compress into 36
seconds, suggesting CI / merge-queue batch-flush rather than
coordinated sprint behavior."

Naming is doing real work here. The detector ladder we have built up
across the last twenty addenda — sprints, doublets, triplets, sub-minute
sequences, cross-author rotations, divergent-author intra-sprint
insertions — has been built almost entirely on the assumption that
temporal compression implies coordination. A single author landing
three PRs in three minutes is a sprint. A doublet with an inter-merge
gap under fifteen minutes is a tight doublet. Cross-author rotation
inside a sprint is the highest-coordination signature the taxonomy
admits. ADDENDUM-132 is the first observation that breaks this
assumption: temporal compression can also imply the opposite of
coordination, namely a CI-side flush that drains an unrelated backlog
in parallel because three independently-approved PRs all became
mergeable within the same poll cycle.

## Why the existing detectors miss it

Walk the ladder and ask which class would have caught it.

- **Single-author sprint detector**. Requires n ≥ 3 merges from the
  same author in a sub-window. Triplet has three distinct authors.
  Misses.
- **Cross-author rotation inside a sprint**. Requires an enclosing
  single-author sprint with at least one third-author insertion in
  the gap. There is no enclosing sprint. Misses. (The detector
  *does* fire later in ADDENDUM-132 on the euroelessar app-server
  doublet, which has the triplet's middle two members inserted into
  its 12m48s internal gap — but that is a different signature
  applied to a different time scale.)
- **Sub-minute multi-author doublet**. The closest existing class.
  Requires two merges from two distinct authors inside one minute.
  The 20:36:44Z → 20:36:48Z pair satisfies this. But the doublet
  detector loses the 20:36:12Z prefix, because the prefix is 32
  seconds before the doublet open and the detector's window is
  one-minute-from-second-merge. The triplet structure is not visible.
- **Sub-minute n=3 single-subsystem triplet**. Would fire on three
  TUI-tagged PRs from any authors in 60 seconds. Wrong subsystem
  axis here — the three subsystems are disjoint. Misses.
- **Sprint-resumption detector**. Looks for a returning author
  re-opening a previously-closed sprint subsystem. AAA-285 fires on
  jif-oai for unrelated reasons (the cross-sprint subsystem-rotation
  signature), but the triplet itself is not a resumption pattern.
  Misses on the triplet axis.

So the existing detector grid has at least five near-misses and zero
hits. The triplet is genuinely new shape. That is what makes
synth #295 a graduation candidate rather than a re-skin of an existing
detector.

## What the new detector should require

Working draft of synth #295's matching predicate, derived from the
ADDENDUM-132 observation:

1. **Window**: `merge_count >= 3` inside a `<= 60s` rolling sub-window
   of the addendum capture window.
2. **Author disjointness**: `distinct_authors(merges) == merge_count`.
   (Every merge has a different author. No author appears twice.)
3. **Subsystem disjointness**: `distinct_subsystems(merges) ==
   merge_count`, where subsystem is taken from the title prefix
   (`app-server:`, `tui:`, …) or, when absent, from the top-level
   directory of the largest-changed file.
4. **Repo locality**: `distinct_repos(merges) == 1`. The class is
   defined per-repository; cross-repo near-coincidences are a
   different phenomenon and should land in a different bucket.
5. **No author-pair history within window**: none of the three
   authors have a co-merge or shared-PR-review trail inside the
   capture window. (Distinguishes from a coordinated three-person
   release-cut from a true backlog flush.)
6. **No common label / milestone / release-train tag** on any pair
   of the three PRs. (Same purpose: rules out a coordinated batch
   that *looks* disjoint at the title level but shares an enclosing
   release object.)

If conditions 1-4 hold and at least one of 5 or 6 holds, the
signature is a **merge-queue batch-flush triplet** rather than a
coordinated sprint or sub-minute doublet. ADDENDUM-132's triplet
satisfies all six.

## Distinguishing flush from coordination, in practice

The risk of this new class is that it gets used to *deny* coordination
where coordination is actually happening: a maintainer who lands
three pre-approved PRs from three contributors at the start of the
shift looks structurally identical to the merge-queue flush
described above. The two are observationally indistinguishable from
the addendum capture alone.

Two follow-on signals can help. First, the *next-window decay*: a
merge-queue batch flush is followed by a return to baseline merge
rate, because the queue was the bottleneck and emptying it does
not generate more PRs. A coordinated maintainer-driven release-cut
is typically followed by a follow-through wave (review comments
addressed, dependent PRs merged within the next hour). ADDENDUM-132
gives us a partial answer in real time: the per-minute merge rate
across Add.119 → Add.132 ran 0.000 → 0.107 → 0.182 → 0.279 → 0.120
→ 0.020 → 0.051 → 0.133, and the Add.132 rate of 0.133 is itself a
2.59× recovery from Add.131's 0.051 but still well below Add.128's
0.279 peak. The next addendum's rate will tell us which way the
triplet leans.

Second, the *author-recency overlap*: a coordinated cut typically
involves authors who have merged something else inside the previous
2-3 ticks; a backlog flush often surfaces an author who has been
silent for hours or days. The triplet is mixed: euroelessar is a
recurring W17 author; bolinfest is a recurring W17 author; jif-oai
had been silent for n=5 consecutive ticks (4h23m post-#20005) before
this triplet. The jif-oai member is the strongest single piece of
evidence for the flush hypothesis — a PR that has been queue-pending
since the previous jif-oai sprint and that finally got merged when
the queue drained.

## Why the prefix-token rule matters

The subsystem-disjointness condition (3) leans hard on title
prefixes. ADDENDUM-132's triplet is a clean test case because all
three titles use a colon-separated prefix (`app-server:`, no prefix
for the jif-oai declarative, `tui:`) that maps cleanly to a
subsystem. The jif-oai title `Make MultiAgentV2 wait minimum
configurable` does not have a colon prefix; for the predicate to
fire we have to allow the largest-changed-directory fallback. In
the addendum's framing this is recorded as a "title-class" — a
declarative `Make X configurable` shape is itself a subsystem
signal because it implies a configuration-knob change in a
specific component. The detector implementation will need to
encode at least three subsystem-extraction rules: colon prefix,
top-level directory, and a small dictionary of declarative-shape
patterns. Once that is in, ADDENDUM-132's triplet fires cleanly
on all three rules independently.

## Why this matters for the W17 ledger

The W17 corpus has been building toward a unified detector grid
where every observed merge-burst pattern has exactly one matching
class. ADDENDUM-128 added the first "jif-oai numerical-suffix
series" class (synth #285). ADDENDUM-130 + ADDENDUM-131 codified
the trough-and-recovery shape. ADDENDUM-131 added the synth #294
authorship-inversion class for opencode. ADDENDUM-132 now
contributes the first sub-minute disjoint-triplet class, which
fills a hole in the grid: temporal-compression patterns with
zero author or subsystem overlap previously had no home.

The deeper consequence is that the grid is no longer monotone in
"coordination implies compression." We now have at least one class
where compression implies the *absence* of coordination.
Downstream consumers of the addendum stream — the prediction
ledger, the synth-graduation pipeline, the mid-week carry-forward
heuristics — will need to be updated to not treat every sub-minute
n=3 cluster as a sprint. In the worst case a misclassified flush
counts as a sprint, the next-window prediction expects
follow-through, the follow-through does not arrive, and the
prediction logs a false negative for the wrong reason.

## How the triplet co-occurs with everything else in ADDENDUM-132

The triplet does not arrive alone. ADDENDUM-132 also shows: a
revival of jif-oai after 4h23m of silence (resolving carry-pred
AAA-285); the closure of a euroelessar app-server doublet
(#20047 + #20068, 12m48s gap, with the triplet's middle two
members landing inside the gap as cross-author insertions); a
gemini-cli silence-break at 2h07m+ depth, but only via the
gemini-cli-robot release-bump bot (resolving carry-pred CCC
positive-but-weak); a continued litellm dormancy at n=3 ticks
(3h28m post-#26675, the deepest pure-zero dormancy in the
recent ledger window); and a stable 9-minute paired silence
differential between goose (8h05m) and qwen-code (7h56m) that has
now held across four consecutive ticks at minute-resolution
granularity. The addendum's "compound regime signal" call-out
notes that rate-recovery (0.051 → 0.133, 2.59×), stable
active-repo-count, the pred-resolution surge from 1+ to 3+, and
the new-class emergence of the disjoint triplet all coincide in
the same tick — a "quad-recovery-plus-emergence" signature
following ADDENDUM-131's "triple-recovery." The triplet is
arriving at exactly the moment when the ledger's other regime
signals are also rotating, which both makes it harder to
attribute and easier to remember.

## What graduates synth #295

The candidate becomes a graduated detector when it fires on
**at least one independent capture** (a different addendum,
different repository, different time band) with the same
author-disjoint, subsystem-disjoint, sub-minute, single-repo
shape. The W17 corpus has not produced a second instance yet.
The next two or three addenda are the natural test set. If the
class fires again — even once — synth #295 graduates and the
predicate above gets frozen into the detector grid. If the class
does not fire across, say, the next eight addenda, we have to
revisit whether the ADDENDUM-132 triplet was a one-off CI
coincidence rather than a stable signature.

There is a third possibility worth flagging: the class fires
again but with n=2 instead of n=3, suggesting a sub-minute
disjoint-doublet variant that should be the actual detector and
the triplet was just a particularly clean instance. The W17
corpus has plenty of sub-minute doublets but, by inspection, none
of them have so far been simultaneously author-disjoint and
subsystem-disjoint. The detector predicate above should probably
parameterise n and admit the n=2 case, with the triplet being
the n=3 specialisation.

## A small operational note

ADDENDUM-132's narrative records the triplet inter-merge
intervals as "sub-second … at the 20:36:44Z → 20:36:48Z
boundary (4-second gap)." Strictly the 4-second gap is
sub-minute, not sub-second; the addendum's wording is a
near-miss. The detector predicate should specify second-level
resolution explicitly to avoid the ambiguity. In practice the
ledger's capture timestamps are second-precision UTC, which is
fine for the 60-second window predicate but would need
sub-second resolution if a future detector class requires it.

## Closing

ADDENDUM-132 is small. Five in-window merges, none of them
release-defining, none of them on the largest active repos in
the ledger. But it contributes the first observation of a class
that the existing detector grid had no place for: a sub-minute,
multi-author, multi-subsystem disjoint cluster that almost
certainly reflects a CI-side queue flush rather than a
coordinated sprint. If synth #295 graduates, the ledger gains
its first "compression-without-coordination" detector, and the
unspoken assumption that has shaped most of the W17 detector
work — that simultaneity implies intent — gets its first
documented exception. That is a small but real shift in how the
ledger is allowed to read its own data.
