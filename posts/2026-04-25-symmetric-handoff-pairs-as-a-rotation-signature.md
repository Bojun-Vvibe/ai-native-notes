---
title: "Symmetric Handoff Pairs as a Rotation Signature: When the Top Two Models Trade Buckets in Lockstep"
date: 2026-04-25
tags: [pew-insights, handoffs, model-rotation, fleet-analytics, telemetry]
---

# Symmetric Handoff Pairs as a Rotation Signature

There is a particular shape that shows up in the `bucket-handoff-frequency`
report when you look at it on a quiet afternoon: the top two handoff pairs
are not just adjacent on the leaderboard, they are *the same pair twice*,
once in each direction, with the same count on both rows. From the
2026-04-25T09:14:54Z run of `pew-insights@0.4.76`, against an 888-bucket
workspace with 887 considered pairs and a 14.88% handoff share, the top of
the table looked like this:

```
claude-opus-4.7 -> gpt-5.4         count: 34
gpt-5.4         -> claude-opus-4.7 count: 34
claude-opus-4.6-1m -> gpt-5.4      count:  6
gpt-5.4         -> claude-opus-4.6-1m count: 5
```

Two `34`s. Not 33-and-35. Not 32-and-36. Two perfectly equal counts in
opposite directions, contributing 68 of the 132 total handoff pairs in the
workspace — **51.5% of all model-to-model bucket transitions in a fleet of
888 buckets pass between exactly two models, in both directions, in equal
volume**. The third and fourth rows show the same shape attenuated:
opus-4.6-1m and gpt-5.4 trade 6 and 5, off-by-one. That off-by-one is
itself diagnostic, and we will come back to it.

This post is about what that signature *means* operationally, why it's
different from a stickiness reading, why the off-by-one matters more than
the perfect symmetry, and what fleet-design decisions you should consider
making once you see it.

## The shape, formally

A "handoff" in `bucket-handoff-frequency` is defined as: in two adjacent
hour-buckets in the workspace's active set, the *primary model* (the model
with the most tokens in that bucket) changed. The pair is recorded as
`(from -> to)` and counted. Critically, the pair is *directional*: the
report distinguishes `A -> B` from `B -> A` and reports both rows
independently in `pairs[]`.

A symmetric pair, then, is the situation where `count(A -> B) == count(B
-> A)` to within a small tolerance. Perfect symmetry — the case we saw
this morning — is `count(A -> B) == count(B -> A)` exactly. Near
symmetry is `|count(A -> B) - count(B -> A)| <= 1`. The opus-4.6-1m row
satisfies near symmetry (6 vs 5).

There is no statistical reason this should hold by default. If two models
were independently sampled from a stationary distribution of session
preferences, you'd expect the two directional counts to be Poisson-ish
around the same mean, but the variance would produce visible asymmetry —
something like 30 vs 38, or 28 vs 40 — for a true count of 34. Two
identical 34s is not what independent sampling looks like. It is what
*alternation* looks like.

## Alternation vs. random switching

There are three regimes a workspace can be in when its top two models
exchange primacy across buckets:

1. **Random switching.** Each bucket independently picks the primary
   model based on whatever happens to be running at the time. Directional
   pair counts will be approximately equal in expectation but visibly
   noisy in any finite sample. Long runs of the same model are common
   (geometric-distributed). This is what most "natural" multi-model
   workspaces look like.

2. **Alternation / rotation.** The workspace alternates between the two
   models in a near-deterministic way, either because of an explicit
   policy (an orchestrator that round-robins) or because of an emergent
   behavior (a single human is using two coding agents back-to-back, one
   for plan-mode and one for execute-mode, and the boundary falls on
   bucket edges). Long runs are rare; the directional counts converge to
   each other much faster than random switching predicts.

3. **Asymmetric drain.** One model is being phased out and the other is
   absorbing its work. `count(old -> new)` will exceed `count(new ->
   old)` by a wide margin and you'll see one of the two rows decay over
   time. This is the *opposite* of symmetric handoffs and shows up
   clearly when you graph pair counts over a window.

The two-`34`s shape is regime 2, not regime 1. Specifically: the perfect
symmetry plus the size of the count (34 transitions in ~888 buckets, so
roughly one symmetric pair per 26 buckets) implies that opus-4.7 and
gpt-5.4 are *both primary, alternately*, with very few stretches of
either dominating for long. The contiguous-handoff count from the same
report is illuminating here: `contiguousHandoffs: 5, gappedHandoffs:
127`. Of the 132 handoff pairs, only 5 are between buckets that are
literally adjacent in time (i.e. consecutive hours). The other 127 are
across gap-bucket boundaries — buckets where neither model was active.
The dominant signature is *episodic* alternation, not continuous
back-and-forth.

This matches the operator experience: you sit down for an afternoon
session with opus-4.7 in one terminal and gpt-5.4 in another, you
alternate which one you ask, and several hours later you walk away. The
handoff at the end of the session is a "primary changed" event because
the next time you come back you happen to start with the other one. It
isn't that you're round-robin-ing minute by minute. It's that *the
workspace's identity is "the two of them, together"* and the report is
correctly identifying that pair as the operational unit.

## Why the off-by-one matters

Look at the next two rows:

```
claude-opus-4.6-1m -> gpt-5.4      count: 6
gpt-5.4 -> claude-opus-4.6-1m      count: 5
```

The 6 vs 5 is not noise. It is a **terminal asymmetry**. With 11 total
transitions between this pair, the off-by-one tells you which direction
the pair *ended* on at the time of the snapshot. If gpt-5.4 went into
opus-4.6-1m 6 times and opus-4.6-1m went into gpt-5.4 only 5 times, then
the workspace is currently sitting in a state where `opus-4.6-1m` is the
last-observed primary model in a bucket that participated in this
exchange. The +1 in one direction is the *outstanding session*. It is the
one transition that hasn't yet been "closed" by a return trip.

This is a useful invariant. For any pair `(A, B)`, `|count(A -> B) -
count(B -> A)|` is either 0 or 1, and the sign tells you which model is
currently holding the bucket lead in the most recent exchange. With
perfect symmetry (the 34/34 case), the workspace closed cleanly — the
last transition between the pair was a return trip, and neither model is
"holding the ball" right now. The 6/5 case has gpt-5.4 having handed off
to opus-4.6-1m one extra time, and opus-4.6-1m hasn't returned yet.

This matters for operational interpretation in two ways:

- **Fleet-state at snapshot time.** If you're using this report to
  understand "which model is currently primary in this workspace?", the
  off-by-one tells you, *in aggregate over the pair*, which side last
  pulled. It is a weak signal individually but adds up across many pairs.
- **Symmetry restoration as a freshness proxy.** A workspace that
  consistently shows perfect symmetry on its top pair has been running
  long enough for the alternation to "close". A workspace where the top
  pair shows a +5 imbalance is either young (haven't completed many
  cycles) or genuinely drifting away from symmetry — you need other
  signals (`source-decay-half-life`, `source-tenure`) to disambiguate.

The same `source-tenure` run from this morning, also at 09:14:54Z, showed
six total sources with a combined 1,323 active buckets and 8.49B tokens.
The dominant token producer is `claude-code` at 3.44B tokens across 267
buckets (12.89M tokens per active bucket), but the dominant *bucket*
producer is `openclaw` at 348 buckets in only 198.5 span-hours — a
density of 1.75 buckets per span-hour vs `claude-code`'s 0.156. The two
top handoff models (`opus-4.7` and `gpt-5.4`) are almost certainly both
flowing through the same source surfaces. The handoff signature isn't
telling you about model preference per se; it's telling you about *the
source-level orchestration choice* of "use these two models together".

## What this signature is NOT

It is worth being precise about what symmetric handoffs do *not* tell
you, because it's easy to over-read.

- **Not a stickiness signal.** From the same report,
  `stickiestModel: gpt-5.4, stickiestModelBuckets: 204`. Stickiness here
  means: when gpt-5.4 is primary in bucket N, it's also primary in
  bucket N+1 with high probability. That's a per-model self-loop count,
  not a pair count. The fact that gpt-5.4 is the stickiest model *and*
  the most-handed-off-with model is consistent — it means gpt-5.4 has
  long runs interrupted by alternations with opus-4.7. The two metrics
  are orthogonal.
- **Not a load-balancing claim.** Symmetric pair counts do not mean
  symmetric *token* allocation. If opus-4.7 sessions are 3x larger than
  gpt-5.4 sessions, the pair can still be 34/34 in handoff count while
  the token split is 75/25. Use `source-tenure` and per-model token
  reports to ground that.
- **Not a routing-policy claim.** This pattern can emerge from explicit
  policy *or* from emergent operator behavior. Without source-level
  context (which orchestrator is making the choice), you cannot
  attribute the symmetry to a router decision. With sources like
  `claude-code` and `openclaw` both showing high bucket density, the
  more parsimonious explanation in this workspace is operator-driven
  alternation, not policy.

## The 0.1488 handoff share as context

`handoffShare: 0.1488162344983089` says that 14.88% of all
adjacent-bucket pairs in this workspace involve a primary-model change.
That is a *low* number — 85.12% of bucket transitions keep the primary
model the same. A workspace with truly chaotic model rotation would push
that share above 0.4 or 0.5; a workspace with a single dominant model
would push it below 0.05. The 0.1488 reading places this workspace
firmly in the regime where the top two models are stable for stretches,
then trade, with very little involvement from the long tail.

The long tail is real but small. After the top eight rows, the report
notes `droppedBelowTopCap: 23` — twenty-three additional pairs fell
below the `--top-handoffs 10` display cap, contributing the rest of the
132 total handoff pairs. The fact that you can capture the workspace's
handoff geometry with the top *two* rows (51.5%) and the top *eight*
rows (the symmetric quartet plus four singleton-ish entries) tells you
the workspace's effective routing graph has very low cardinality.

## What to do when you see this signature

If you're looking at your own `bucket-handoff-frequency` report and the
top of the table shows two equal counts in opposite directions for the
same pair, here's a short decision tree:

1. **Check the off-by-one of the next-rank pair.** Perfect symmetry on
   rank 1 plus near symmetry on rank 2 is the alternating-pair-fleet
   signature. Perfect symmetry on rank 1 with an asymmetric rank 2 is a
   subtler reading: the top pair is your stable rotation, but the rank-2
   pair is in transition.
2. **Cross-reference with `stickiestModel`.** If the stickiest model is
   *one of* the rotating pair, the workspace has a clear primary that
   occasionally swaps — you can probably consolidate billing/quotas
   around the stickier of the two. If the stickiest model is *neither*
   of the pair, you have a dominant single model plus a separate
   alternating dyad on the side, which is unusual and worth looking
   into.
3. **Check `contiguousHandoffs` vs `gappedHandoffs`.** With the 5/127
   ratio we saw, the alternation is episodic (across gap buckets, i.e.
   across sessions). A high contiguous ratio would mean the alternation
   is *within* a session — a much more aggressive routing pattern that
   warrants different treatment.
4. **Pull `source-tenure` for the same window.** The handoff-pair
   identity is meaningless without knowing which surface(s) those models
   live on. With 6 distinct sources and 1,323 active buckets, the
   workspace I'm looking at is small enough that you can map every
   handoff to a source by hand if needed.

## A note on the version

`pew-insights@0.4.76` (CHANGELOG entry dated 2026-04-25) added
`--min-handoffs <n>` to this report specifically because the long tail
of one-shot pair rows like `claude-haiku -> gemini-3-pro x1` was padding
the table and obscuring exactly the kind of rotation signature this post
is about. The `--min-handoffs 3 --top-handoffs 10` composition gives you
a clean view of *recurring* routes vs *flukes*. The default behavior
(no floor) is what produced the report I read this morning, and the
symmetry stands out even without the new flag — but on a busier
workspace I'd expect the flag to be load-bearing for the same reading.

## Summary

A `bucket-handoff-frequency` row of `(A -> B, n) (B -> A, n)` with
identical counts is not a coincidence and not statistical noise. It is
the signature of an alternating-pair workspace: two models that share
operational primacy via session-boundary transitions rather than
within-session rotation. The off-by-one in the next-rank pair tells you
which direction the most recent exchange went. The `handoffShare` and
`contiguousHandoffs` ratios contextualize whether the alternation is
gentle or aggressive. And the underlying `source-tenure` reading
explains where the alternation is happening — which surface is making
the routing choice.

In the workspace I looked at this morning, the answer is: opus-4.7 and
gpt-5.4 are co-primary in episodic alternation across multiple sources,
with gpt-5.4 holding a slightly stickier within-bucket grip but the two
trading session-level primacy in perfect balance over the snapshot
window. That's a single legible sentence about the fleet's identity,
read off four data points in a JSON blob. Which is what these reports
are for.

---

*Source: `pew-insights@0.4.76` `bucket-handoff-frequency --json`,
generated 2026-04-25T09:14:54.430Z, 888 active buckets, handoff share
0.1488; `source-tenure --json` same timestamp, 6 sources, 1,323 active
buckets, 8.49B total tokens.*
