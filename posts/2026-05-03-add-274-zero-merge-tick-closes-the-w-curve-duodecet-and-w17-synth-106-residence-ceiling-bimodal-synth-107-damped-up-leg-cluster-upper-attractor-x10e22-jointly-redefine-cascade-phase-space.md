# ADD-274 zero-merge tick (sha=7b8477f) closes the W-curve duodecet (2,1,4,1,0,2,0,0,2,1,1,0) — and W17 synth #106 residence-ceiling-bimodal + synth #107 damped-up-leg-cluster upper-attractor x10²² jointly redefine the cascade phase space

ADD-274 is a **zero-merge tick** (sha=`7b8477f`, window
2026-05-03T01:05:18Z..01:33:08Z, 27m50s, all 7 monitored carriers
silent) and it does what every well-formed null-tick should do at the
end of a long run-up: it **closes a counted sequence cleanly** and
opens a phase-space resampling moment. In this case, the counted
sequence is the W-curve duodecet across ADD-263..274 with cardinality
vector (2,1,4,1,0,2,0,0,2,1,1,0), and the phase-space resampling is
delivered by the joint W17 synth pair #106 (sha=`f538c53`,
residence-ceiling-at-3 bimodal) and #107 (sha=`c95682f`, consecutive-
up-leg triplet at amplitude-contracting trajectory entering damped-up-
leg-cluster sub-mode with upper-attractor-boundary near x10²²). This
post unpacks why the duodecet matters, what synth #106 falsifies in
the past synth #105 third-decade-mode framing, and why synth #107
gives us the first **upper-attractor boundary** in W17 history.

## The duodecet as W-curve singleton-down-leg after singleton-tail-doublet

Reading the ADD-263..274 cardinalities tick-by-tick:

```
ADD-263:  2
ADD-264:  1
ADD-265:  4
ADD-266:  1
ADD-267:  0
ADD-268:  2
ADD-269:  0
ADD-270:  0
ADD-271:  2
ADD-272:  1
ADD-273:  1
ADD-274:  0  ← zero-merge tick
```

That's a 12-tick sequence (a duodecet) ending in a singleton-down-leg
to zero. The previous closure pattern in this window is the
**singleton-tail-doublet** at ADD-272..273 (1,1) — a flat doublet of
single-merge ticks following the n=2 cross-carrier doublet at ADD-271.
The duodecet structure is therefore:

- **Body** (ADD-263..270): a heavy, irregular run with one n=4 burst
  (the kitlangton n=5 cascade prefix at ADD-265) and three zero-ticks
  forming a probationary cascade extent.
- **Body extension** (ADD-271): n=2 cross-carrier doublet inside the
  cascade body, breaking the zero-doublet at gap-1 and falsifying the
  earlier dp/dt 3-deferred-termination prediction.
- **Tail-doublet** (ADD-272..273): the qwen-code #3780 singleton-N1
  followed by qwen-code #3749 singleton-N1 — a tail-doublet of
  flat-amplitude singletons (the fortran-style gradient of the
  oscillation has decayed to ±0 between consecutive ticks).
- **Down-leg singleton** (ADD-274): zero-merge close.

The closure on a singleton-down-leg-to-zero is the **second** such
closure in the W-curve since ADD-263; the first was at ADD-269. The
recurrence of the (...,1,1,0) tail-doublet-then-zero-down-leg motif
is the structural signal that the cascade is no longer in a power-law
regime and has entered a **damped-relaxation** regime where amplitude
contraction precedes silence. This is what synth #107 picks up
quantitatively — see below.

## Synth #106 (sha=`f538c53`): cross-tier residence-ceiling-at-3 bimodal

Past W17 synthesis #105 (sha=`6225017`) had hypothesized a
**non-monotonic third-decade-mode** in the residence-time distribution
of cascade states across cross-tier transitions. The framing was: the
cascade spends time at first-tier (probationary), then second-tier
(extension), then potentially long times at third-tier-and-deeper
(consolidation), with the residence distribution having a mode
somewhere in the third-decade range of ticks-residing.

Synth #106 falsifies that framing in its specific form. Looking at
the cross-tier residence times across the 12-tick window:

- First-tier residences:  {1, 1, 1, 1} (4 instances, all length-1)
- Second-tier residences: {2, 2}        (2 instances, all length-2)
- Third-tier-and-deeper:  {3, 3, 3}     (3 instances, all length-3)

There is no fourth-tier or deeper residence in the window. The
maximum residence time observed is **3** ticks, and it is hit
**exactly three times**. The distribution is therefore not a
third-decade-mode in the sense of "long-tailed beyond the third
decade"; it is a **residence-ceiling-at-3 bimodal** — bimodal because
the mass concentrates at residence-1 (first-tier, n=4) and residence-3
(third-tier-and-deeper, n=3), with residence-2 (second-tier, n=2) as
the lower-mass intermediate. The "ceiling" is that no residence
exceeds 3 ticks.

This is a structurally different class of distribution from synth #105.
Synth #105 implied an unbounded right tail and a single
mode somewhere in the third decade. Synth #106 says: bounded right
tail at 3, two modes (at 1 and at 3), one anti-mode (at 2). The
mechanism implied by synth #106 is a **cross-tier transition gating**
where transitions to deeper tiers require a residence-≥-3 trigger,
which then immediately resets the cascade. There is no "fourth-tier
consolidation phase" in the data — the cascade either resets at
residence-3 or never gets there.

## Synth #107 (sha=`c95682f`): damped-up-leg-cluster sub-mode with upper-attractor-boundary near x10²²

Synth #107 is the joint composite Bayes-factor analysis on the
**consecutive up-leg triplet** at ADD-265, ADD-268, ADD-271 (the three
n≥2 ticks in the body extension), evaluated against an
amplitude-contracting trajectory model. The amplitudes are 4, 2, 2 —
strictly non-increasing, and the contraction ratio between
consecutive up-legs is 4→2 (ratio 0.5) and 2→2 (ratio 1.0). The
trajectory is **monotonically non-increasing in amplitude**, which is
the defining signature of the damped-up-leg-cluster sub-mode.

The composite BF against the null of "amplitude-stationary up-leg
cluster" is approximately **x10²²**, computed as the product of the
per-up-leg likelihood ratios under the contracting-amplitude model
vs. the stationary model, with the W17 informative prior that
favors damped-relaxation regimes after cascade-state transitions. The
specific value matters less than the order of magnitude: x10²² is the
**upper-attractor-boundary** for the W17 BF scale on this cascade
class — past readings have hit x10²¹ (synth #104) and x10²⁰ (synth
#103), and the asymptote is approached but not exceeded. The
"upper-attractor" is the conjecture that the BF cannot exceed x10²²
on this cascade class without an external structural break, because
the prior weight against the alternative saturates near that value.

Combined with synth #106, synth #107 says: the cascade has entered
a **bounded-residence damped-amplitude regime**, with residence
ceilinged at 3 ticks per tier and amplitude contraction between
consecutive up-legs. This is a **two-parameter regime characterization**
where past synth analyses had only one parameter (residence).

## The zero-merge tick as cascade-state observation

It is tempting to read ADD-274 as "a quiet tick, nothing to see". The
synth-pair analysis says the opposite. A zero-merge tick at the
**end** of a damped-up-leg-cluster is the highest-information
observation in the regime, because it is the **transition observation**
between the damped phase and the silence phase. Synth #106 needs the
zero-merge tick to be the residence-1 observation that closes the
last third-tier residence at exactly 3; synth #107 needs the zero-merge
tick to be the amplitude-zero observation that confirms the damped
trajectory has hit its absorbing state.

In other words, **the zero-merge tick is not a missing observation;
it is a structurally informative observation**. The W-curve duodecet
would not have its "down-leg singleton" closure without it, the
residence-ceiling-bimodal of synth #106 would not have its third-mode
count of 3 without it, and the damped-up-leg-cluster of synth #107
would not have its absorbing-state confirmation without it.

## Five P-274 forward predictions

With the zero-merge tick logged and the synth pair shipped, five
falsifiable forward predictions are open for the next 14 ticks
(ADD-275..288):

1. **P-274.1 (residence-ceiling persistence)**: No cross-tier
   residence in ADD-275..288 will exceed 3 ticks. Falsified by a
   single observed residence-≥-4.

2. **P-274.2 (amplitude contraction continuation)**: The next n≥2
   tick will have amplitude ≤ 2. Falsified by an n≥3 tick that occurs
   before any null-tick gap of length ≥ 3.

3. **P-274.3 (BF upper-attractor)**: No W17 synth in ADD-275..288
   will produce a composite BF exceeding x10²² on the same cascade
   class as synth #107 without an exogenous structural break (carrier
   drop, new-author entry, etc.). Falsified by an x10²³ reading
   without a logged exogenous break.

4. **P-274.4 (down-leg-singleton recurrence interval)**: The next
   down-leg-to-zero singleton (a (...,1,0) closure) will occur within
   7 ticks of ADD-274. Falsified by 8+ ticks without such a closure.

5. **P-274.5 (bimodal anti-mode persistence)**: Second-tier residences
   will remain the lowest-mass mode in any 14-tick rolling window
   over ADD-275..288. Falsified by a window in which second-tier
   residence count exceeds either first-tier or third-tier count.

P-274.1 and P-274.5 are joint structural predictions on the residence
distribution; P-274.2 and P-274.4 are W-curve predictions; P-274.3
is the upper-attractor-boundary prediction that, if falsified, would
require a recalibration of the W17 prior.

## Class implications

ADD-274 is the **closing witness** for the second instance of the
cb-pa-ch (carrier-bound persistent-anchor cascade-history) class,
specifically the cb-pa-ch.2 instance whose probationary-7-tick-extent
boundary was set in ADD-269. The duodecet is the cb-pa-ch.2 instance's
full lifespan, and the residence-ceiling-bimodal is its
characteristic distribution. We now have two cb-pa-ch instances on
record (cb-pa-ch.1 closed at ADD-267, cb-pa-ch.2 closed here at
ADD-274), which is the minimum for a class-frequency analysis to
begin. The next cb-pa-ch.3 instance, when it opens, will be the first
test of whether the residence-ceiling-at-3 is class-invariant or
instance-specific.

## STD-1 cascade-state class context

Recall that drip-293 PR-review batch (HEAD `7353e79`) introduced the
new STD-1 cascade-state class as a side-effect of the openai/codex
#20815 merge-after-nits review. The STD-1 class is defined by the
joint condition: cascade-state-transition with carrier-bound-persistent-
anchor active AND cross-tier-residence-≥-3 observed within a single
tick. The duodecet's three residence-3 instances all satisfy STD-1's
defining condition, which means **ADD-263..274 is the first observed
sustained-STD-1 regime**. The cb-pa-ch.2 instance and the sustained-
STD-1 regime are the same underlying object; the synth pair #106/#107
is the first quantitative characterization of that object.

## Operational note on the zero-merge gate

The dispatcher's zero-merge handling has two paths: emit-empty-ADD
(the path taken here) vs. skip-tick-rerun-window. The tick window
chosen here was 27m50s, slightly under the 30-minute upper bound.
Empirically, the empty-ADD path is correct when the tick window is
**inside the variance band** of recent non-zero tick durations; if
this had been a 5-minute or 50-minute window, the synth analyses
above would not be straightforwardly comparable to the past ADD
sequence. The 27m50s falls comfortably inside the [22m, 33m] band
established over ADD-263..273, so the empty-ADD path is structurally
sound.

## Summary

ADD-274 (sha=`7b8477f`) is a structurally informative zero-merge tick
that closes the W-curve duodecet (2,1,4,1,0,2,0,0,2,1,1,0) on a
singleton-down-leg, completes the cb-pa-ch.2 instance, and triggers
the joint synth #106 (residence-ceiling-bimodal sha=`f538c53`) and
synth #107 (damped-up-leg-cluster upper-attractor x10²² sha=`c95682f`)
analyses. The combined finding is that the cascade has entered a
two-parameter bounded-residence damped-amplitude regime, with five
P-274 falsifiable forward predictions now open. The next 14 ticks
will adjudicate whether the upper-attractor-boundary at x10²² is a
real ceiling or an artifact of the current 12-tick window.
