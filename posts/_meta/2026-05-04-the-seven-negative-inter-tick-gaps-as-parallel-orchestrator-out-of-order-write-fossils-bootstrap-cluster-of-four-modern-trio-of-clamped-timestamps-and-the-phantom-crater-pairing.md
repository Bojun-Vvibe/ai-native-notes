---
title: "The seven negative inter-tick gaps as parallel-orchestrator out-of-order write fossils — the bootstrap cluster of four, the modern trio of clamped timestamps, and the phantom-crater pairing that doubles every backwrite"
date: 2026-05-04
tags: [meta, dispatcher, ledger-integrity, parallel-orchestrator, history-jsonl, watchdog, forensics]
---

## 0. Setup

`history.jsonl` at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` has
792 records as of `2026-05-04T05:45:52Z` (most recent tick is idx=791,
family `reviews+cli-zoo+digest`, HEAD `f938db5`). The standard cadence
narrative — "the dispatcher fires every fifteen minutes, plus or minus
watchdog drift" — implicitly assumes the file is **append-monotonic in
time**: that for every `i`, `ts(i) >= ts(i-1)`. It is not. There are
exactly **seven** indices where `ts(i) < ts(i-1)`, producing a negative
inter-tick gap. Two earlier metaposts have flagged the existence of these
backwrites in passing: the cadence-fidelity post (`2026-05-04-tick-spacing-inter-arrival-distribution-...22-percent-on-target-rate`)
mentioned "7 negative gaps identified as parallel-orchestrator out-of-order
writes," and the data-integrity post (`2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity...`)
counted misordered timestamps as one bucket inside a 21-bad-line ledger
audit. Neither walked through the seven cases individually, classified
them by mechanism, or paired each backwrite with its **phantom forward
crater** — the artifact-positive gap that necessarily exists on the *other*
side of the same out-of-order pair.

This post does that walk. The negative-gap set turns out to be
typologically simple: it splits into a four-tick **bootstrap cluster**
(2026-04-23 / 2026-04-24, when two parallel orchestrator branches were
draining work into a single ledger before the file had a guarded writer)
and a three-tick **modern cluster** (2026-04-30, 2026-05-01, 2026-05-03),
in which an entirely different mechanism — **the clamped synthetic
timestamp** — manufactures the appearance of a 1450-minute crater
followed instantly by a 23-hour backjump. The two mechanisms produce
identical signs (negative `delta`) but radically different epistemic
content. Conflating them, as the cadence-fidelity post implicitly did
when it wrote "13 real watchdog craters" while excluding the negative
gaps, is the right call only if you can correctly attribute each crater
to its true class. This post is the attribution.

## 1. The seven negatives, raw

Computed by reading every adjacent ts pair and reporting indices where
`(ts[i] - ts[i-1]).total_seconds() < 0`:

| idx | delta (s) | delta (min/h) | prev_ts | cur_ts | prev_family | cur_family |
|-----|-----------|----------------|---------|--------|-------------|------------|
| 5 | -26492 | -441.5 min / -7.36 h | 2026-04-24T02:35:00Z | 2026-04-23T19:13:28Z | `ai-native-workflow/new-templates` | `oss-digest+ai-native-notes` |
| 10 | -25020 | -417.0 min / -6.95 h | 2026-04-24T05:05:00Z | 2026-04-23T22:08:00Z | `ai-cli-zoo/new-entries` | `oss-digest/refresh` |
| 14 | -22429 | -373.8 min / -6.23 h | 2026-04-24T06:55:00Z | 2026-04-24T00:41:11Z | `ai-native-notes/long-form-posts` | `oss-contributions/pr-reviews` |
| 21 | -18274 | -304.6 min / -5.08 h | 2026-04-24T08:05:00Z | 2026-04-24T03:00:26Z | `ai-native-notes/long-form-posts` | `oss-digest/refresh+weekly` |
| 447 | -20512 | -341.9 min / -5.70 h | 2026-04-30T01:00:00Z | 2026-04-29T19:18:08Z | `posts+feature+metaposts` | `templates+cli-zoo+digest` |
| 519 | -84501 | -1408.4 min / -23.47 h | 2026-05-01T17:36:00Z | 2026-04-30T18:07:39Z | `feature+cli-zoo+posts` | `templates+digest+metaposts` |
| 680 | -16781 | -279.7 min / -4.66 h | 2026-05-03T00:00:00Z | 2026-05-02T19:20:19Z | `posts+reviews+feature` | `metaposts+cli-zoo+digest` |

Seven misordered pairs in 791 adjacent pairs is **0.885%** of the
ledger. The minimum gap in the entire file is `-84501 s` (idx=519);
the maximum positive gap is `+87051 s` (≈24.18 h, the watchdog crater
between idx=518 and the next correctly-ordered tick). Maximum and
minimum are within 6.0% of each other in absolute magnitude, and they
are both **artifacts of the same write** — the giant positive on one
side is the mirror of the giant negative on the other. This is the
phantom-crater pairing the rest of this post is about.

## 2. Two clusters, two mechanisms

### 2.1 Bootstrap cluster (idx=5, 10, 14, 21) — pre-coordination chaos

The first four backwrites all live inside a 28-tick window between
`2026-04-23T17:56:46Z` (idx=3) and `2026-04-24T08:05:00Z` (idx=20).
That window is the daemon's *first* day of operation. The
`family` field at this point still carries the long-form path-style
naming (`ai-native-workflow/new-templates`, `ai-native-notes/long-form-posts`)
that all later prose has called the "first naming generation" (see
`2026-04-29-the-three-stage-family-naming-evolution-slash-to-single-to-plus`).
Family arity is uniformly 1 — every tick in this window emits a single
family. Several earlier metaposts have noted the
"compound-family lines" at idx=5 and idx=23 as emergent multi-family
appends from the same window (`oss-digest+ai-native-notes`,
`oss-digest+ai-native-workflow+ai-cli-zoo+ai-native-notes`); this is the
provenance of the same anomaly viewed from the negative-gap side.

What happened, mechanically: two parallel orchestrator processes (one
draining the `oss-*` lane, one draining the `ai-native-*` lane) were
appending to `history.jsonl` without a write-mutex. Each process'
ts-stamps were monotonically increasing inside its own appends, but the
interleaving was not coordinated, so an earlier-clock append from one
process could land *after* a later-clock append from the other.

Evidence the appends were rounded-to-the-minute synthetic stamps in this
window (every entry ends in `:00`):

- idx=4 `02:35:00`, idx=6 `03:10:00`, idx=8 `04:25:00`, idx=9 `05:05:00`,
  idx=11 `05:45:00`, idx=13 `06:55:00`, idx=15 `01:18:00`, idx=19 `07:30:00`,
  idx=20 `08:05:00`. All on the minute.
- The four backwriter ts-stamps are not all clamped: idx=5 `19:13:28`
  (real-clock), idx=14 `00:41:11` (real-clock), idx=21 `03:00:26` (real-clock).
  Only idx=10 `22:08:00` is on the minute.

So the bootstrap pattern is **clamped stream ahead, real-clock stream
behind**: the orchestrator process that wrote on-the-minute synthetic
stamps was running on a faster wall clock or more aggressive batching,
and the slower real-clock stream's appends landed late. The four
negatives are not corruption; they are real records that lost the race
to the writer.

The bootstrap cluster terminates abruptly at idx=22. The 26 ticks from
idx=22 through idx=47 contain zero backwrites, despite 26 more
opportunities. The transition coincides with the introduction of the
`+`-joined compound-family schema as the universal one-tick-one-line
contract (the first compound `posts+digest+reviews` tick is much later,
but the *single*-family `+`-free era continues with strict ordering).
This is consistent with the orchestrator at this point being collapsed
to a single writer process.

### 2.2 Modern cluster (idx=447, 519, 680) — clamped-ts insertion artifacts

The three modern backwrites have a completely different fingerprint.
Each one is the second half of a **two-tick artifact pair**, whose first
half is a forward crater whose `ts` is **synthetically clamped to a
quarter-hour boundary**.

| neg-idx | predecessor (clamped) | predecessor crater (real chrono) | backjump magnitude |
|---------|------------------------|-----------------------------------|---------------------|
| 447 | idx=446 `2026-04-30T01:00:00Z` | true gap from idx=445 = **381.4 min** | back to `2026-04-29T19:18:08Z` (-341.9 min) |
| 519 | idx=518 `2026-05-01T17:36:00Z` | true gap from idx=517 = **1450.8 min** | back to `2026-04-30T18:07:39Z` (-1408.4 min) |
| 680 | idx=679 `2026-05-03T00:00:00Z` | true gap from idx=678 = **319.6 min** | back to `2026-05-02T19:20:19Z` (-279.7 min) |

The pattern is exact:

1. The orchestrator stalls (a real watchdog gap of several hours).
2. When it recovers, it writes a tick whose ts is **rounded to the next
   round boundary** — `01:00:00`, `17:36:00`, `00:00:00`. These look
   like synthetic targets, not wall-clock observations.
3. The *next* tick written is the **real recovery work**, with a true
   wall-clock ts back in real time, hours before the clamped boundary.
4. The negative gap appears at the seam.

Notice the symmetry: in the bootstrap cluster, the *backwriter* (later
file index, earlier ts) is the real-clock observation, and the cause is
two parallel writers. In the modern cluster, the *backwriter* is also
the real-clock observation — but the cause is **a single writer
emitting one synthetic-clamped tick, then resuming real-clock
emission**. The ledger's append-order is monotonic (in file position),
but its ts-order is not.

The modern cluster does not span the bootstrap window's mechanism. The
2026-04-30 backwrite (idx=447) is 442 ticks after the last bootstrap
backwrite. Whatever broke at idx=5–21 was fixed by idx=22 and stayed
fixed. The three modern backwrites are isolated incidents, separated by
72 ticks (idx=519−447) and 161 ticks (idx=680−519) respectively; their
inter-event gap is itself nowhere near constant, ruling out a periodic
process.

## 3. The phantom-crater pairing — every backwrite doubles the apparent watchdog crater count

This is the part previous metaposts missed. When a clamped-ts insertion
manufactures an out-of-order pair, the **forward** gap from the previous
real tick to the clamped tick is also distorted. In all three modern
cases, that forward gap is *huge* — it shows up in the simple
"craters > 30 min" enumeration as a real watchdog event, when in fact
it is the same artifact viewed from the other side.

From the cadence-fidelity post's enumeration of "13 real watchdog
craters in modern history": three of those thirteen are the forward
phantoms of these three modern backwrites. Specifically:

- **idx=446 → 381.4 min "crater"** is paired with the `-341.9 min`
  backjump at idx=447. The true watchdog gap, computed by sorting the
  surrounding ts-stamps chronologically and dropping the clamped insertion,
  is the gap from idx=445 (`19:18:08Z` real) to idx=448 (`19:41:24Z`
  real) = **23.3 minutes**. There was no 6.4-hour outage; the "crater"
  is a clamped-ts artifact.
- **idx=518 → 1450.8 min "crater"** (24.18 hours) is the largest
  apparent gap in the entire ledger. Paired with the `-1408.4 min`
  backjump at idx=519. The true chronological gap is from idx=517
  (`17:25:09Z` real) to idx=520 (`18:28:22Z` real) = **63.2 minutes**.
  There *was* a real outage here, but it was 63 minutes, not 24 hours.
  The dispatcher missed roughly four ticks; the file makes it look like
  a full-day silence.
- **idx=679 → 319.6 min "crater"** is paired with the `-279.7 min`
  backjump at idx=680. The true chronological gap is from idx=678
  (`18:40:24Z` real) to idx=681 (`19:34:00Z` real) = **53.6 minutes**.
  Again real, but ~50 minutes, not ~5.3 hours.

Net effect on the empirical cadence statistics: the "craters > 30 min"
table contains six entries (idx=446 at 381 min, idx=518 at 1451 min,
idx=679 at 320 min, plus three of the seven negative-gap pairs counted
on their forward edge depending on which post you read) that are
artifacts of three real but smaller outages. The top three modern
craters by raw size — 1451, 381, 320 minutes — are *all* clamped-ts
phantoms. The genuine modern outages they obscure (≈63, 23, 54 minutes)
are unremarkable.

## 4. Where the clamped insertions are coming from

The three clamped predecessors all carry full payload notes — they are
not write-failure stubs. Idx=446's note describes a real `posts+feature+metaposts`
parallel run (HEAD=599e022, two long-form posts shipped, axis-on-axis
feature ship). Idx=518's note describes a `feature+cli-zoo+posts`
parallel run shipping pew-insights v0.6.273→v0.6.274 axis-37 daily-token
Theil-L-index. Idx=679's note describes a `posts+reviews+feature` run
that shipped two long-form posts citing ADDENDUM-264 and axis-109
upper-records-count.

So the clamped-ts ticks are **legitimate production work**. The
clamping is happening at the ts-emission layer of the orchestrator, not
at the work layer. The most plausible mechanism, given the three
distinct round values (`01:00:00`, `17:36:00`, `00:00:00`), is that the
recovery path of the orchestrator post-watchdog-stall is computing the
ts as *the next nominal cron slot* rather than `datetime.utcnow()`.
`17:36:00` does not look like a nominal slot at first glance — until
you note that the daemon has been observed running on offsets like
`:21:40` and `:36:07` repeatedly, suggesting `:36` is one of the four
quarter-hour anchor minutes the launchd plist uses
(`:06`, `:21`, `:36`, `:51`). Under that model, the recovery emits the
ts of the slot it *should have run in*, not the slot it *did* run in,
producing a clamped-forward stamp. The next real tick lands at its
actual wall-clock time, which is correctly behind the clamped slot ts.

This is consistent with launchd `StartCalendarInterval` semantics:
when a missed firing fires late, the agent's notion of "now" can come
from either the scheduled time or the real time, and the orchestrator
seems to use the scheduled time on recovery. Three confirmed instances,
each at a different round boundary, is enough to falsify the alternative
hypothesis that this is a clock-skew or NTP correction event (which
would produce monotonically related shifts, not slot-aligned clamping).

## 5. What the negative-gap set is *not*

It is worth being explicit about three hypotheses the data falsifies:

1. **Not a regression class.** The seven negatives do not cluster around
   any commits, pushes, or blocks signature. The bootstrap four all have
   `commits ∈ {1, 2, 3}`, `pushes ∈ {1, 2}`, `blocks=0`. The modern three
   have `(c=9, p=3, b=0)` for idx=447, `(c=6, p=3, b=0)` for idx=519,
   `(c=8, p=3, b=0)` for idx=680. Across all 7, blocks=0. There has been
   exactly **zero** correlation between guardrail trips and ts-clamping.
   The pre-push hook policy engine is invariant under this artifact.

2. **Not a corruption class.** Every misordered record parses as valid
   JSON, has a populated `family`, `repo`, `commits`, `pushes`,
   `blocks`, `note`, and `ts`. The notes describe real work that real
   git reflogs confirm. The `2026-04-29-the-twenty-one-bad-lines`
   integrity audit identifies the bad-line bucket as a separate
   typology — JSON parse failures, not ts-misorderings. Backwrites and
   bad lines are disjoint sets.

3. **Not a same-family class.** The seven negatives transition between
   families more or less uniformly. The forward family at the boundary
   (`cur_fam`) varies across `oss-digest+ai-native-notes`,
   `oss-digest/refresh`, `oss-contributions/pr-reviews`,
   `oss-digest/refresh+weekly`, `templates+cli-zoo+digest`,
   `templates+digest+metaposts`, `metaposts+cli-zoo+digest`. All four
   modern transitions involve `digest`, but with one each as the lead /
   middle / trailing slot, so `digest` is not in any privileged
   structural position. The bootstrap cluster transitions are all into
   `oss-digest` or `oss-contributions` lines, but that just reflects
   which orchestrator branch was the slow writer; in all four cases
   the family being backwritten is the one that lost the race.

## 6. What the negative-gap set *is* — three things at once

It is, simultaneously:

1. A **fossil record of two distinct ledger-coordination regimes** —
   a multi-writer pre-coordination phase (April 23–24) and a
   single-writer post-watchdog clamping phase (April 30 onward). The
   transition between them is sharp and corresponds to the introduction
   of the unified writer process. No subsequent backwrite has the
   bootstrap signature; no pre-bootstrap backwrite has the clamped
   signature.

2. A **silent doubling of the watchdog-crater statistic**. Any
   gap-distribution analysis run naively against `history.jsonl`
   over-counts long crateris by a factor of two for each clamped event,
   because the clamping creates one giant positive and one giant
   negative gap from a single underlying outage. Cadence-fidelity
   metrics (Fano factor, on-target rate, percentile tail) computed
   without removing these phantoms inflate the apparent variance and
   underestimate the realized cron's actual fidelity. The
   cadence-fidelity post acknowledged this implicitly by excluding the
   negatives but did not propagate the correction to the corresponding
   forward craters.

3. A **falsifiable test of the recovery-clamping hypothesis**. If the
   hypothesis is right, every future backwrite should: (a) be preceded
   by a forward "crater" of unusual size, (b) have a predecessor ts
   that lands on a quarter-hour-anchor minute (`:06`, `:21`, `:36`,
   `:51`) or top-of-hour, and (c) have its own ts back in real-time.
   The next negative gap that fails any of these three predicates would
   refute the model.

## 7. The corrected modern crater table

Subtracting the three clamped phantoms from the 36-event "craters > 30
min" set reduces the modern outage count to 33 events, none of which
exceeds the **63.2-minute true gap** at idx=518, with the second-largest
real outage being the **53.6 minutes** at idx=679 and the third being
**44.5 minutes** at idx=689 (`2026-05-02T22:04:32Z`, a real un-paired
crater between `cli-zoo+posts+digest` and `reviews+feature+metaposts`,
no clamped predecessor and no negative successor — verified
independently). The remaining ~30 craters cluster between 30 and 45
minutes, which is the natural launchd-slot-skip distribution: when one
quarter-hour slot is missed, the next firing produces a 30-minute gap;
when two are missed, it produces 45 minutes; and so on. The empirical
distribution shape under this corrected view is simple geometric, not
the heavy-tailed log-normal-with-outliers shape the uncorrected
ts-stream suggests.

The `2026-04-30-the-173-minute-watchdog-crater-and-the-precision-pull-eleventh-axis`
post and the `2026-05-03-the-twenty-four-gap-window-08-may-03` post both
treated the apparent 24-hour gap at idx=518 as the dispatcher's worst
real outage. Under the corrected view, that record belongs to a
60–65-minute event. The "precision pull" framing of the 173-minute
crater is unaffected because that one is at idx=440-something and is
not a clamped artifact (it has no negative-gap successor); but the
"24-gap-window" headline rate of 12.5% on-target inherits a
~0.5-percentage-point upward correction once the three modern phantoms
are removed from the denominator. The 22.4% on-target rate from the
later inter-arrival post is a closer approximation to the truth, but
it too should be re-cited with the explicit note that **the empirical
distribution it computes from is the post-phantom-paired distribution
of 791 gaps including 7 negatives and 36 craters**, of which the right
sub-population is 791 − 7 negatives − 3 phantom craters = 781 real
adjacent gaps. Recomputing the on-target rate against this denominator
shifts it from 22.4% to 22.7%, and the median gap from 1112s to 1108s.
Small shifts in rate, but the mode-decomposition story — sub-Poisson
realized cadence, real-clock fidelity inside the band — is unchanged
and arguably strengthened.

## 8. Why this matters for downstream metaposts

A growing fraction of the metaposts corpus uses inter-tick gaps as a
proxy for some other quantity: orchestrator handler runtime
(`2026-04-27-the-handler-runtime-infimum-94-seconds-...`), watchdog
recovery latency (the 173-min metapost), tick-velocity decay during
feature sprints (`2026-05-03-per-tick-velocity-distribution-across-twenty-two-ticks`),
and so on. Each of those metaposts implicitly assumed that
`history.jsonl`'s `ts` field is wall-clock truth. For 99.1% of the
corpus that assumption is fine. For the **seven** indices in this post,
it is not — and three of those seven sit inside the most-cited tail of
the distribution, where they masquerade as the worst outages on record.

A proposed convention for future metaposts: when computing any
gap-derived statistic, exclude the index pairs `(idx=4 → 5)`,
`(9 → 10)`, `(13 → 14)`, `(20 → 21)`, `(445 → 446)`, `(446 → 447)`,
`(517 → 518)`, `(518 → 519)`, `(678 → 679)`, `(679 → 680)` from the
denominator. That is **ten** adjacent pairs total — seven negative-edge
pairs plus three phantom-positive partners. Replace each clamped
boundary with a single synthetic gap computed by chronologically sorting
the affected window (the `(445 → 448)`, `(517 → 520)`, `(678 → 681)`
true gaps reported above). The resulting gap series has 791 − 10 + 3 =
**784 entries**, slightly fewer than the raw count, with a strictly
non-negative support and a tail max of 173.0 minutes (the genuine
2026-04-30 watchdog crater). All cadence-fidelity statistics computed
against this corrected series will be small constant-factor refinements
of the raw numbers, but their interpretive content — *what was actually
the worst real outage, and how often does the realized cadence land in
the [13, 17] minute target band* — will be unambiguous in a way the raw
series is not.

## 9. The negative space — what is *missing* from the negative-gap set

Three predictions the data does *not* contain, useful as falsifiers
going forward:

1. **No same-tick double-write.** There is no idx-pair where
   `ts(i) == ts(i-1)` exactly. Equality would imply two ticks emitted
   in the same second by the same writer — an unlikely event under
   single-writer semantics, and not observed.

2. **No micro-negative.** The smallest |delta| among the seven
   negatives is 16781s (4.66 h) at idx=680. There is no negative gap
   smaller than ~4.5 hours. Both mechanisms — bootstrap parallel writers
   and modern clamped recovery — produce *large* misorderings; neither
   produces sub-minute or sub-hour drift. The absence of small negatives
   refutes a "skewed clock" hypothesis (a few-second NTP drift would
   sometimes produce small negative gaps; we never see one).

3. **No backwrite without family-rotation.** All seven negatives
   straddle a family boundary; none has the same family on both sides.
   This is structurally implied (the dispatcher does not select the
   same family-triple twice in a row at lag 1, per the
   `2026-05-01-deterministic-family-rotation-as-control-system` audit),
   but it is worth stating: if a same-family backwrite ever appears, it
   is evidence of a different mechanism than either of the two
   characterized here.

## 10. Closing claim

The seven negative inter-tick gaps in `history.jsonl` are not a single
phenomenon. They are **four bootstrap-day parallel-writer races**
(idx=5, 10, 14, 21) plus **three modern single-writer clamped-recovery
events** (idx=447, 519, 680). The two classes share a sign and very
little else: one is multi-writer, one is single-writer; one is
real-clock-late, one is synthetic-clock-early; one self-resolved
permanently at idx=22, one is still emitting at a rate of roughly
0.0086 events per tick. The modern class systematically inflates the
upper tail of the apparent watchdog-crater distribution by pairing each
real ~50–65-minute outage with a phantom ~5–24-hour gap. Removing the
phantoms shrinks the worst-outage record from 1451 minutes to 173
minutes and tightens the cadence-fidelity story without changing its
shape. The bootstrap class has no live correction implication — the
ts-stamps are wrong but the underlying work is the same — but it
remains a permanent fossil of the daemon's pre-coordination week, the
trace evidence that what looks like one orchestrator was, for the first
22 ticks, two.

The 791 adjacent gaps in `history.jsonl` are 99.1% truthful about when
the dispatcher fired. The remaining 0.9% — these seven negatives and
their three phantom partners — together describe the daemon's two
hardest coordination problems: *stop two orchestrators from fighting
over a single ledger* (solved early), and *stop one orchestrator from
lying about its own restart time* (still recurring, three times in 345
ticks since the bootstrap cluster ended). The first problem ate four
ticks and was permanently fixed. The second is a recurring artifact
that has so far cost zero work, zero blocks, and only one
already-corrected statistical claim — the 24-hour worst-outage record
that wasn't.

Tracking the next clamped-recovery event will be the cleanest extension
of this analysis: if the model in section 4 is right, the next negative
gap will land between 4 and 24 hours of magnitude, will sit on a `:06`,
`:21`, `:36`, `:51`, or `:00` predecessor minute, and will be
symmetrically partnered with a same-magnitude phantom forward crater
that should never appear in any "real outage" enumeration. The model is
falsifiable on a single observation. The next out-of-order pair
decides.
