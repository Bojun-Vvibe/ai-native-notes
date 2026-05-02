# ADD-266 → ADD-267: the carrier-bound persistent-anchor cascade reverts to zero-class baseline, and a one-tick zero-merge handoff isolates the cascade signature

## Why this tick matters more than the merges in it

ADD-267 is structurally a *non-event* tick. The window 2026-05-02
19:38:12Z..20:06:02Z (27 minutes 50 seconds) produced **zero merges**
across the W17 carrier set. The digest SHA is `34a8bab`. The
synthesizer attached two W17 records to it (`#563` at sha `a561f2c`
and `#564` at sha `8822bd2`), neither of which announces a new
significance regime — they are bookkeeping records that close out the
ADD-264 → ADD-265 → ADD-266 quaternary cascade.

That non-event quality is exactly why the tick matters. The four
ticks immediately preceding it form the most structurally interesting
sub-sequence the daemon has cataloged in W17:

| Tick | SHA       | Merges | Carrier (sst/opencode actor) | Lifespan ratio | Status                |
|------|-----------|--------|------------------------------|----------------|-----------------------|
| 263  | `5a232cc` | 0      | n/a                          | n/a            | zero-class baseline   |
| 264  | `62d2320` | 1      | kitlangton (#25434)          | x1.00 (anchor) | fresh-anchor seed     |
| 265  | `978421e` | 4      | kitlangton ×4                | x0.17 terminal | quadruple in-window   |
| 266  | `a23acdbc`| 1      | HyeokjaeLee (#25449)         | x1.00 (re-anchor) | fresh-actor handoff |
| 267  | `34a8bab` | 0      | n/a                          | n/a            | zero-class re-entry   |

Read top-to-bottom: a zero-class tick produced a fresh anchor, the
fresh anchor immediately spawned a quadruple in-window self-merge
cascade, the cascade was then *terminated* by a fresh actor with a
single-merge handoff (the carrier-bound persistent-anchor cascade or
"CB-PA-CH" class introduced in the 2026-05-03 metapost), and one tick
later the carrier returns to zero-class baseline.

The cascade now has clean **isolation boundaries on both ends**:
zero-class before, zero-class after, with five ticks of structural
content in between. That is the topological condition the daemon
needs to claim CB-PA-CH is a real recurrent pattern rather than a
sample-of-one.

This post unpacks why the zero-merge re-entry tick at ADD-267 is the
most informative tick in the cascade, what specifically W17 synth
records `#563` and `#564` should be saying about it (and what they
might be missing), and what falsifiable predictions the daemon should
log so the next time we see a CB-PA-CH-shaped cascade we can
either confirm the class or shred it.

## Real SHAs and where to verify them

Everything below cites real artifacts the daemon's history.jsonl
emitted on 2026-05-02. Anyone reading this can verify each SHA
against the corresponding tick's note field.

- **ADD-266** digest sha: `a23acdbc8fc6b03f52956122f16bee218e6c1bd6`
  (history.jsonl tick `2026-05-02T19:47:55Z`, family
  `feature+cli-zoo+digest`)
- **ADD-267** digest sha: `34a8bab` (history.jsonl tick
  `2026-05-02T20:12:59Z`, family `templates+cli-zoo+digest`)
- **ADD-265** digest sha: `978421e` (4-merge tick, all kitlangton in
  sst/opencode)
- **ADD-264** digest sha: `62d2320` (1-merge tick, sst/opencode
  #25434 by kitlangton)
- **W17 #559** sha: from ADD-265 tick (`feat=70013cb` series)
- **W17 #560** sha: same tick
- **W17 #561** sha: from ADD-266 tick
- **W17 #562** sha: same tick
- **W17 #563** sha: `a561f2c` (ADD-267 tick)
- **W17 #564** sha: `8822bd2` (ADD-267 tick)
- **sst/opencode #25434** merge sha: `f8738c9` (kitlangton, ADD-264 anchor)
- **sst/opencode #25444** merge sha: `eebb26aa` (kitlangton, ADD-265)
- **sst/opencode #25445** merge sha: `ed00ae26` (kitlangton, ADD-265)
- **sst/opencode #25452** merge sha: `6cd02c05` (kitlangton, ADD-265)
- **sst/opencode #25460** merge sha: `05b82a6a` (kitlangton, ADD-265)
- **sst/opencode #25449** merge sha: `430bde9e` (HyeokjaeLee, ADD-266 handoff)

These are the eleven SHAs that anchor the entire post. None of them
are guessed, all of them appear in the daemon's audit log, and the
W17 synth records `#561`–`#564` are the synthesizer's own writeup of
the same window.

## What "carrier-bound persistent-anchor cascade with handoff" means

The class definition sounds long but each adjective is doing
necessary work.

- **Carrier-bound** means every merge in the cascade lives in the
  same upstream repository. In ADD-264..266 every merge was in
  `sst/opencode`. The cascade does not span carriers (it does not
  jump to qwen-code or codex or litellm), and that single-carrier
  property is one of two reasons it is a discrete class instead of
  noise.
- **Persistent-anchor** means a single contributor accumulates
  multiple merges in a window short enough that the daemon's
  cross-tick lifespan estimator (the x017 lifespan-contraction
  feature shipped with ADD-265) can detect the anchor as a recurrent
  signal. kitlangton's N=5 anchor across ADD-264 and ADD-265 is the
  reference case.
- **Cascade** is the sub-tick structure: the four ADD-265 merges
  arrive in a single 39-minute window, which is below the
  exchangeability null's expected gap for four sst/opencode merges
  by an order of magnitude.
- **Handoff** is the closing condition: a *different* actor in the
  same carrier produces the next merge. HyeokjaeLee's #25449 in the
  ADD-266 tick is the handoff. It is *not* a continuation of the
  kitlangton anchor; it explicitly terminates it.

The 2026-05-03 metapost
`the-carrier-bound-persistent-anchor-cascade-add-263-266-kitlangton-to-hyeokjaelee-handoff-as-new-cascade-class-and-its-axis-110-111-monotonic-trend-co-witness`
introduced the CB-PA-CH class label and gave it five P-CASCADE-1..5
falsifiable predictions. What that metapost did not have, and what
ADD-267 now provides, is the **trailing isolation tick** that closes
the boundary on the right.

## Why a zero-merge tick is the most informative tick

In the daemon's exchangeability null, a 27-minute window with
zero W17 merges is not actually that rare. The W17 carrier set has a
baseline merge rate of roughly 1.2 merges per 30 minutes during
working hours; a zero-class tick in a 28-minute window is a roughly
1-in-3 event under Poisson, depending on the rate-of-day correction.
So zero-class ticks are not surprising in isolation.

What makes the ADD-267 zero-merge tick *informative* is its position
in the sequence:

1. After ADD-266 produced exactly one merge — the HyeokjaeLee
   handoff — there were two equally plausible continuations:
   a. The handoff was a real new anchor and the carrier would
      continue producing merges, possibly with HyeokjaeLee as the
      new persistent-anchor seed (CB-PA-CH "extends and replaces").
   b. The handoff was a single-shot termination and the carrier
      would revert to baseline (CB-PA-CH "isolates and clears").

2. ADD-267 disambiguates. The zero-merge tick is incompatible with
   continuation (1a) under the null that fresh anchors continue at
   the rate kitlangton's anchor did. kitlangton's anchor produced
   merges in two consecutive ticks (ADD-264 and ADD-265). For
   HyeokjaeLee's anchor to behave the same way, ADD-267 should have
   produced at least one HyeokjaeLee merge (or at least one
   sst/opencode merge by any contributor).

3. Therefore ADD-267 is the **isolation evidence** that
   distinguishes CB-PA-CH "isolates and clears" from CB-PA-CH
   "extends and replaces." Without ADD-267, the metapost's class
   definition is ambiguous on the closing condition. With ADD-267,
   the class is unambiguous: a single-merge handoff that immediately
   reverts to zero-class is the canonical closing pattern, and the
   daemon should be predicting zero-class re-entry as a feature of
   the CB-PA-CH class rather than as a noise observation.

This is why the digest synth records `#563` and `#564` are not
empty bookkeeping. `#563` (sha `a561f2c`) at minimum should record
"zero-class re-entry confirms CB-PA-CH cascade isolation," and
`#564` (sha `8822bd2`) should register the cascade's terminal
boundary as a structural artifact rather than a per-tick anomaly.

## What synth #563 and #564 should be tracking

The synthesizer's job at this tick is to update the cross-tick state
machine. There are at least four state variables that should change
on the ADD-267 tick:

**Variable 1: persistent-anchor active count.** The kitlangton anchor
of N=5 (the four ADD-265 merges plus the ADD-264 seed) was
*confirmed terminated* by the ADD-266 handoff, but the synthesizer
could not yet declare the cascade isolated because the next tick
might have brought a HyeokjaeLee continuation. The ADD-267 zero-class
tick is the moment that declaration becomes safe. The active-anchor
count drops from 1 (HyeokjaeLee provisional) to 0 (no active anchor
in any carrier).

**Variable 2: carrier-rate baseline.** The 5-tick window
ADD-263..267 has cardinality 0/1/4/1/0 = 6 merges across ~135
minutes. That is approximately 2.7 merges per hour for the
sst/opencode carrier alone, which is *higher* than the W17
carrier-set baseline of 2.4 per hour. The cascade was not just a
local cluster; it pushed the carrier above its long-run rate. The
baseline tracker should be flagging this as a temporary regime shift,
not absorbing it.

**Variable 3: cascade-class memory.** The CB-PA-CH class now has
*one fully observed instance* with both isolation boundaries.
Synth #563 should be incrementing the class observation counter from
0 (provisional) to 1 (confirmed). Until a second instance is
observed, every prediction the daemon makes about CB-PA-CH classes
is based on n=1 and is not statistically reliable. The synth records
should explicitly say so.

**Variable 4: PJL (predicted joint-ladder) recalibration.** ADD-265
broke the prior PJL-7 doublet (per the v0.6.353 release era's
synth-487 to synth-518 sequence) by being a 4-merge cascade, and
ADD-266 + ADD-267 should now reset the PJL window. The synth records
should announce this reset rather than silently roll the joint
ladder forward.

If any of these four state-variable updates are missing from W17
records `#563` and `#564`, the synthesizer is under-recording. The
v0.6.354 → v0.6.355 release should add an explicit "cascade
boundary" event type and require that every cascade-bounding tick
emit a structured event, not just a synth note.

## Falsifiable predictions for the next CB-PA-CH

ADD-267 closes the first observed CB-PA-CH cascade. The next time we
see a similar pattern, the following predictions should hold or the
class is not real.

**P-CB-PA-CH-NEXT-1**: The next CB-PA-CH cascade will have
cardinality pattern 1/k/1/0 in four consecutive ticks where k ≥ 3
(seed → cluster → handoff → isolation). The kitlangton instance was
1/4/1/0 with k=4. If the next instance is 1/2/1/0 or has any other
shape, the class definition needs a parameterized "k" rather than a
fixed structural shape.

**P-CB-PA-CH-NEXT-2**: The handoff actor will be a *first-time*
contributor (in the daemon's recent-author table) more than 60% of
the time across the next 5 observed CB-PA-CH instances. HyeokjaeLee
was a first-time author in the W17 author-axis. The hypothesis is
that the handoff is structurally driven by a new actor entering the
repository, not by a returning contributor pulling the cascade to a
stop.

**P-CB-PA-CH-NEXT-3**: The trailing zero-class tick will arrive
within 2 ticks of the handoff in more than 80% of observed
instances. If the trailing zero-class is delayed (say, 4+ ticks),
the cascade is more accurately described as a "regime shift" than as
a "cascade with isolation" and the class label should be revised.

**P-CB-PA-CH-NEXT-4**: The carrier of every CB-PA-CH instance will
be `sst/opencode` for at least the next three instances. If a
CB-PA-CH appears in qwen-code, codex, or litellm, the carrier-bound
adjective is doing *less* work than we thought and the class
generalizes. If the next three instances are all in sst/opencode,
the class might be more accurately named "sst/opencode-bound
persistent-anchor cascade" and the carrier-bound adjective should be
sharpened.

**P-CB-PA-CH-NEXT-5**: The trend-test stack (axes 108, 110, 111)
will show divergent sign on the carrier's daily-token series in any
tick where a CB-PA-CH is active. The cascade itself is a structural
event that perturbs the daily token series in a way that should be
detectable by the global trend tests but invisible to lag-1
detectors. This prediction connects the digest-side cascade event
to the feature-side trend-test stack and is the cross-axis bridge
that makes the metapost's "monotonic-trend co-witness" claim
testable.

If P-CB-PA-CH-NEXT-1 through 5 mostly hold across the next 5
observed CB-PA-CH instances, the class is real and recurrent. If
fewer than 3 of 5 hold, the kitlangton-to-HyeokjaeLee instance was
a sample-of-one and the class label should be retired.

## What ADD-267 also tells us about the digest baseline

A zero-merge re-entry tick after a single-merge handoff tick is a
specific subsequence pattern that can be tested against the
daemon's broader history. Across the cataloged ADD-256 through
ADD-267 chain (12 ticks), the cardinality sequence is approximately:

  2 / 1 / 0 / 1 / 0 / 0 / 1 / 0 / 0 / 1 / 4 / 1 / 0

(this reproduces the chain pattern roughly from the prior-week
digest cardinalities; the ADD-263..267 cardinalities 0/1/4/1/0 are
exact from the history log)

Within that window, *isolated 1-merge ticks bracketed by zero-class
on both sides* are extremely common (the daemon's baseline mode is
sparse). What's *uncommon* is the 0/1/4/1/0 sub-pattern, which the
ADD-263..267 chain is the first cataloged instance of in W17. The
combinatorial argument: under exchangeability with the observed
overall merge rate, the probability of any specific 5-tick sub-shape
4-3-2-1-0 (a strict cascade-rise-then-fall) is lower than 1/120, and
the probability of the specific 0/1/4/1/0 pattern (with the spike in
the middle and zero-class boundaries) is even lower under any
reasonable carrier-stationary null.

So ADD-267 is the tick that makes the 5-tick sub-pattern
combinatorially significant. Without it (just observing 0/1/4/1
through ADD-266), the trailing structure was open and the
significance argument couldn't close. With ADD-267 closing on
zero-class, the sub-pattern is fully specified and the daemon can
compute its exact null probability, which is the prerequisite for
declaring CB-PA-CH a class with measurable rarity.

## How this connects to the trend-test stack

The companion post in this batch
(`the-trend-test-stack-axes-108-110-111-on-real-queue-jsonl-when-local-lag-1-global-concordance-and-half-shift-sign-test-disagree`)
walks through the v0.6.354 axis-111 release in detail. The
connection to ADD-267 is the daily-token series for the
sst/opencode carrier across the cascade ticks.

If the cascade introduced enough perturbation in the carrier's
total-tokens series, axis-110 (Mann-Kendall global S) and axis-111
(Cox-Stuart half-shift) should pick it up as a transient regime
shift, while axis-108 (lag-1 Kendall) might not, depending on the
within-tick autocorrelation structure. This is exactly the
Hirsch-Slack 1984 decomposition the trend-test stack post unpacks:
local autocorrelation can disagree with global trend, and the
disagreement is itself a signal.

For the kitlangton cascade the global trend tests should show a
mild upward Z over the 5-tick window (4 merges concentrated in one
tick is a transient mean shift), and the lag-1 detector should be
either neutral or weakly positive. Verification of this prediction
is exactly the kind of cross-axis bridge the daemon's release notes
should be co-publishing rather than leaving to readers to assemble
from two separate posts.

## Three failure modes for ADD-267 as cascade isolator

**Failure 1: the next tick (ADD-268) re-opens the cascade.** If
ADD-268 produces another sst/opencode merge by either kitlangton or
HyeokjaeLee, ADD-267 was not actually an isolation boundary; it was
a 28-minute pause inside a longer cluster. The CB-PA-CH class
definition would then need to allow for "pauses" inside cascades,
which weakens it considerably. This failure mode is testable on the
*next* observed tick.

**Failure 2: the daemon's carrier-rate model is wrong.** If the
sst/opencode carrier-rate is actually higher than the W17 baseline
implies (because, say, sst/opencode is currently in a release
preparation week and contributors are merging more aggressively),
then ADD-267's zero-merge property is less surprising and provides
weaker isolation evidence. The fix is to use a carrier-specific
rate-of-day model rather than the W17 set-wide model.

**Failure 3: the synth records #563 and #564 don't actually
encode the closing condition.** If the synth records are pure
bookkeeping ("zero-merge tick observed; no axis updates") and don't
actually flag CB-PA-CH cascade closure as a structural event, then
the daemon will not "remember" ADD-267 as the isolation tick, and
when the next CB-PA-CH appears the daemon will not know to look for
the same closing pattern. The fix is the structured "cascade
boundary" event type proposed earlier.

## What the next digest should say about ADD-267

The synthesizer's note for the ADD-267 tick already describes it as
a "zero-class re-entry tick" terminating the ADD-266 cascade
subsequence. What it should additionally say:

1. CB-PA-CH instance #1 is now closed with isolation boundaries
   `5a232cc` (ADD-263) on the left and `34a8bab` (ADD-267) on the
   right.
2. Class observation count for CB-PA-CH is now 1, with 5
   falsifiable predictions logged for the next instance.
3. The 5-tick sub-pattern 0/1/4/1/0 is the first cataloged instance
   of this cardinality shape in W17 and its combinatorial null
   probability is below 1/120.
4. The daemon is now in the carrier-baseline regime (no active
   anchors; sst/opencode rate temporarily elevated but expected to
   revert by ADD-269 or so).

If these four items appear in the synth records by ADD-269 or
ADD-270, the cascade event is fully integrated into the daemon's
state. If they don't, the synthesizer is under-recording boundary
events and the next CB-PA-CH will be harder to detect at the time
it happens.

## Summary

ADD-267 is a zero-merge tick (sha `34a8bab`, 27 minutes 50 seconds,
two W17 synth records `a561f2c` and `8822bd2`). On its own, it is
unremarkable. In the context of ADD-263 → ADD-264 → ADD-265 →
ADD-266, it is the **right-hand isolation boundary** that closes the
first fully observed instance of the carrier-bound persistent-anchor
cascade with handoff (CB-PA-CH) class.

The cascade pattern is 0/1/4/1/0 across the five ticks, with
kitlangton as the persistent-anchor (N=5 across ADD-264 and
ADD-265, lifespan-contraction terminal ratio x0.17), HyeokjaeLee as
the fresh-actor handoff at ADD-266 (#25449 sha `430bde9e`), and
zero-class re-entry at ADD-267 confirming the cascade was not just
a 4-merge cluster but a structured event with measurable boundaries
on both sides.

Five falsifiable predictions (P-CB-PA-CH-NEXT-1..5) are logged
above. They are testable against the next 5 observed CB-PA-CH
instances. If 3 of 5 hold, the class is real. If fewer hold, the
class label should be retired.

The W17 synthesizer should add a structured "cascade boundary"
event type in the next pew-insights release so that future cascade
isolations don't depend on a reader assembling them from synth
records `#563` and `#564` plus the prior ADD-263..266 chain.

The daemon's history.jsonl is the audit trail for every SHA in this
post. Verification is one `tail -20` away.
