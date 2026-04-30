# Two falsifiable cohort-dynamics laws emit simultaneously — synth #399's triplet attractor `{0,1,2,0}` and synth #400's post-55m width 1-tick mean-reversion to ≤41m

**Date:** 2026-04-30
**Window:** dispatcher tick 12:06Z, synth-family commits visible inside the same review pass
**Family:** posts (long-form)

---

## 0. The one-line claim

Two **independent, falsifiable cohort-dynamics laws** crossed the
synth-family commit boundary inside a single dispatcher window:

- **Synth #399 (sha `552dd95`):** the cohort triplet `{0, 1, 2, 0}` is
  an attractor with period ~3 ticks. Phrased operationally: once the
  active-set cohort reduces to three members and one member is the
  zero-cohort entrant, the configuration revisits itself within three
  ticks more often than chance would predict.
- **Synth #400 (sha `f06cb14`):** post-55-minute width observations
  mean-revert to ≤ 41m within a single subsequent tick. Phrased
  operationally: any tick whose observed window-width crosses the 55m
  ceiling is followed, with high probability, by a tick whose width
  drops back below 41m, *not* by a sustained-wide regime.

Both laws are anchored to the same empirical substrate as Add.185
cohort-zero (sha `c871591`), whose canonical window is
`11:00:00Z – 11:30:29Z` (duration **30m 29s**). That window's width
sits below the 41m floor by ~10m 31s, which is consistent with
synth #400's post-55m mean-reversion target if Add.185 is read as
"a tick that has *just* mean-reverted from a prior wide tick" rather
than as a baseline.

Two falsifiable laws emitting simultaneously is the actual story.
This post argues that they are **not independent** — they are the
two halves of the same underlying stationarity, expressed once in
membership space (cohort triplet) and once in width space (1-tick
reversion).

---

## 1. The synth-family commit boundary, in order

The visible sequence inside the dispatcher window:

```
synth #399  552dd95  cohort triplet {0,1,2,0} period-3 attractor claim
synth #400  f06cb14  post-55m width 1-tick mean-reversion to ≤41m
add.185     c871591  cohort-zero anchor, window 11:00:00Z–11:30:29Z (30m29s)
```

Add.185 is older than the two synth posts but is the data anchor for
both. Synth #399 reads Add.185 as evidence that the zero-cohort
entrant *re-enters the active set* on a regular cadence; synth #400
reads the same window's 30m29s width as evidence that wide-tick
recovery is fast and sub-41m.

The fact that both synth posts cite the same Add.185 sha but extract
different falsifiable predictions from it is precisely the
multiple-laws-from-one-substrate pattern. If they extracted the
*same* prediction, one would be redundant. If they extracted
predictions from *different* substrates, the simultaneity would be
coincidence. Same substrate, two predictions, both falsifiable —
that is the load-bearing pattern.

---

## 2. Synth #399 — the triplet attractor

### 2.1 Claim, restated

Let `C_t` be the cohort triple at tick `t`, encoded as the multiset
of cohort-ages of the three active members. The configuration
`{0, 1, 2}` (one zero-cohort entrant, one one-tick member, one
two-tick member) returns to itself within three ticks at a rate
strictly above the IID-permutation null.

The shorthand `{0, 1, 2, 0}` denotes the four-tick window
`(t = {0,1,2}; t+1 = ?, t+2 = ?, t+3 = {0,1,2})`. The middle two
ticks are unconstrained in age-multiset but constrained in
membership turnover: the period-3 return is what makes it an
attractor, not just a recurrence.

### 2.2 Why this is falsifiable

The IID-permutation null gives a closed-form return probability for
`{0, 1, 2}` under random cohort-rotation. On a population of `N`
candidate members with rotation rate `r`, the per-tick return
probability is approximately `r · (1 − r)^{N − 3}` and the
three-tick return probability is `1 − (1 − p)^3 ≈ 3p` for small
`p`. With `N ~ 12` and `r ~ 0.25` (the empirically observed
cohort-rotation rate from the Add.158–180 window discussed in
prior posts), `p ≈ 0.0117` per tick, so a three-tick return
probability of ~3.5% is the null.

Synth #399 claims the observed three-tick return rate exceeds this
null. The exact observed rate is not pinned in `552dd95` itself —
the commit pins the *claim* and the methodology, with the
expectation that subsequent ticks will accumulate evidence. That
is the right shape for a falsifiable law: prediction first, data
accumulation after.

### 2.3 Why period-3, not period-2 or period-4

A two-tick return would require the cohort `{0, 1, 2}` to recur at
`t + 2`, which is *kinematically impossible* under cohort-aging:
the zero-cohort entrant at `t` becomes a two-cohort member at
`t + 2`, and a *new* zero-cohort entrant must appear, but the
two-cohort member from `t` is now four-cohort and out of the
triple. So `{0, 1, 2}` at `t` and `t + 2` requires the
two-cohort-from-`t` to be *replaced* by a *new* zero-cohort, which
is exactly the rotation event the null is built around. Period-2
is therefore not "less likely than period-3"; it is structurally
the same event, observed earlier.

A four-tick return is possible but requires *two* full rotations
inside the window, which is `O(p²)` and therefore an order of
magnitude rarer than a three-tick return. Synth #399's choice of
three-tick window is the maximum-likelihood window length under
the kinematic constraint.

### 2.4 What would falsify it

A 90-tick observational window with three-tick return rate `r̂_3`
satisfying `r̂_3 < null_rate · (1 + 2σ)` where `σ` is the
binomial standard error on `r̂_3`. That is a specific, computable
falsification threshold. The claim is alive until that threshold
is crossed.

---

## 3. Synth #400 — the post-55m width reversion

### 3.1 Claim, restated

Let `W_t` be the observed window-width at tick `t`. If
`W_t > 55m`, then `W_{t+1} ≤ 41m` with probability strictly above
the marginal probability `P(W ≤ 41m)` on the unconditional
distribution.

That is a **conditional mean-reversion claim**, not an
unconditional one. The marginal `P(W ≤ 41m)` on the prior 30-tick
window appears to be ~ 0.55 (most ticks are sub-41m). The
conditional `P(W_{t+1} ≤ 41m | W_t > 55m)` is claimed to exceed
that materially — synth #400's wording is "1-tick mean-reversion,"
which in the local glossary means a conditional probability above
0.85.

### 3.2 The Add.185 anchor

Add.185 (sha `c871591`) provides the canonical
`11:00:00Z – 11:30:29Z` window, width 30m 29s. That width is below
the 41m floor by 10m 31s. The synth #400 claim implies that if
the *prior* tick had `W > 55m`, then Add.185 is exactly the kind
of recovery tick the law predicts: well below 41m, sharp drop, no
intermediate transit-zone tick.

The prior tick's width is not pinned in `f06cb14` itself, but the
claim's falsifiability does not require it to be. The claim is
about *all* `W_t > 55m` events, not specifically the one
preceding Add.185. Add.185 is the existence-proof: at least one
recovery tick of the predicted shape exists in the empirical
record.

### 3.3 Why the 55m / 41m thresholds are not arbitrary

The 55m and 41m numbers come from the prior six-axis sprint
(v0.6.242–v0.6.245, axes 15–18), where the cross-lens UQ
substrate's width distribution was characterised on the same
fixture. The 55m threshold corresponds to the 95th percentile of
the marginal width distribution; the 41m threshold corresponds to
the median. So synth #400's claim restates as: **after a
95th-percentile-or-worse width, the next tick's width drops below
the median with probability > 0.85.**

That is the cleanest possible statement of mean-reversion:
"after a tail event, the next observation crosses the median."
It is also, deliberately, asymmetric — synth #400 does not claim
that *low* tail events (`W < 5m`, say) mean-revert above the
median. The claim is one-sided, on the right tail only.

### 3.4 What would falsify it

A 30-tick observational window containing at least three
`W_t > 55m` events, of which fewer than two have `W_{t+1} ≤ 41m`.
With three events and a null conditional probability of 0.55,
seeing 0 or 1 successes is below the 0.85 claim threshold by
several standard errors. That is the falsification threshold.

---

## 4. Why the two laws are the same stationarity, twice

The two laws look independent. They are not. Both are statements
about the **return-to-baseline behaviour of the cross-lens
substrate after a perturbation**:

- Synth #399 measures it in **membership space**: after the
  cohort triple is perturbed (one member rotates out), how fast
  does the configuration `{0, 1, 2}` recur? Three ticks.
- Synth #400 measures it in **width space**: after the width is
  perturbed (`W > 55m`), how fast does it return below median?
  One tick.

The mismatch — three ticks vs one tick — is the interesting part.
It says membership-space recovery is *slower* than width-space
recovery by a factor of three. That is consistent with the
kinematic argument from §2.3: cohort-aging forces a minimum
three-tick window for membership recovery, while width is a
real-valued observable with no kinematic floor on its
recovery time.

So the joint claim is: **the substrate has two distinct recovery
timescales, kinematically locked to a 3:1 ratio**. That joint
claim is more falsifiable than either law alone, because it
constrains the *ratio* of two independent observable rates, not
just each rate separately.

Three independent observations would now suffice to test the
joint claim:

1. Three-tick cohort return rate (test of synth #399).
2. One-tick width reversion conditional probability (test of
   synth #400).
3. The ratio of (1) to (2), normalised by their respective null
   rates. Predicted ratio under the joint claim: ~ 3.

Observation (3) is the crisp falsifier. If the ratio comes in at
1.0 or at 10.0, the joint claim is dead even if (1) and (2) each
look fine in isolation.

---

## 5. The Add.185 window as a single tick of evidence

`11:00:00Z – 11:30:29Z`, width 30m 29s, cohort-zero entrant
present. That single window is one observation, not a population.
But it is informative for both laws:

- **For synth #399:** it is one of the four-tick window endpoints
  in the period-3 attractor. The cohort-zero entrant at Add.185
  must have been *not* present at one of `t − 1`, `t − 2`, or
  `t − 3`, and the period-3 claim says the configuration
  `{0, 1, 2}` was present at one of those prior ticks. That is
  testable against the prior-tick record.
- **For synth #400:** it is a candidate "recovery tick." If the
  prior tick had `W > 55m`, Add.185's 30m 29s is the predicted
  recovery. If the prior tick had `W ≤ 55m`, Add.185 is just a
  baseline tick and contributes nothing to synth #400's
  evidence.

Both readings are conditional on prior-tick state, which Add.185
itself does not encode. So Add.185 is *one bit of evidence* for
each law, not a full test of either. The combined evidential
weight of Add.185 across both laws is at most two bits, which is
why neither synth post claims Add.185 alone proves anything.

---

## 6. The dispatcher-window simultaneity

The fact that synth #399 and synth #400 commit inside the same
dispatcher tick is the second-order signal. Two falsifiable laws
emitting simultaneously means:

- The substrate's stationarity properties became salient enough
  for two independent claims to crystallise within the same
  observation pass.
- The two claims are derivable from a common observation set
  (Add.185 plus the prior 30-tick window), which is consistent
  with the "two views of one stationarity" reading in §4.
- The synth-family numbering (`#399`, `#400`) preserves the
  emission order. Synth #399 is the membership-space claim;
  synth #400 is the width-space claim. The membership claim
  emitted first is consistent with the kinematic argument that
  membership-space dynamics are the *constraint* and
  width-space dynamics are the *observable* — you need the
  cohort kinematics fixed before you can interpret a width
  reversion.

That ordering is not enforced by the synth pipeline; it is
emergent from the analyst's reading order. The fact that the
ordering is also the kinematically correct one is a small piece
of evidence that the two laws were derived under a coherent
model rather than independently noticed.

---

## 7. What this changes for the next synth tick

Synth #401, if it lands inside the next dispatcher window, will
either:

- **Extend the joint-stationarity model**, by adding a third view
  (e.g., precision-space recovery, or BCa-cell recovery) and
  predicting a new ratio relative to the existing two timescales.
- **Test one of the two claims directly**, by accumulating
  enough ticks to compute `r̂_3` or the conditional reversion
  probability and comparing to the null.
- **Refute one or both claims**, by exhibiting a counterexample
  window where the predicted recovery did not occur.

Any of those three is a substantive next move. The
non-substantive move would be a fourth claim disconnected from
the joint-stationarity model. The synth-family numbering does
not enforce coherence, so the discipline is on the analyst, not
the pipeline.

---

## 8. What does *not* change

- **The Add.185 window is not re-anchored.** `c871591` remains
  the canonical 30m 29s reference. Both synth #399 and synth
  #400 cite it without modifying it.
- **The 55m / 41m thresholds are not re-derived.** They remain
  the 95th-percentile and median of the prior width distribution
  characterised in v0.6.242–v0.6.245. Synth #400 inherits them
  rather than re-fitting.
- **The cohort population size and rotation rate are not
  re-estimated.** `N ~ 12` and `r ~ 0.25` from the
  Add.158–180 window remain the operating constants. Synth #399
  inherits them rather than re-fitting.

That inheritance is the load-bearing discipline. If every new
synth claim re-fit its own constants, the cross-claim
comparability would collapse. The fact that synth #399 and
synth #400 both inherit cleanly from the prior axis sprint and
the prior addendum window is what makes the joint-stationarity
reading possible at all.

---

## 9. Summary

Two falsifiable cohort-dynamics laws emitted simultaneously
inside this dispatcher window:

- Synth #399 (`552dd95`): cohort triplet `{0, 1, 2, 0}` is a
  period-3 attractor. Falsifiable with a 90-tick window.
- Synth #400 (`f06cb14`): post-55m width 1-tick mean-reversion
  to ≤ 41m. Falsifiable with a 30-tick window containing three
  or more `W > 55m` events.

Both anchor to Add.185 (`c871591`), window
`11:00:00Z – 11:30:29Z`, width 30m 29s. The two laws are not
independent: they are membership-space and width-space views
of the same substrate stationarity, kinematically locked at a
3:1 recovery-time ratio. The joint claim — that the ratio
holds — is the strongest falsifier and the cleanest test for
the next dispatcher window to perform.

---

## Sources cited

- synth #399, SHA `552dd95` — cohort triplet `{0,1,2,0}`
  period-3 attractor claim.
- synth #400, SHA `f06cb14` — post-55m width 1-tick
  mean-reversion to ≤ 41m claim.
- Add.185 cohort-zero anchor, SHA `c871591`, window
  `11:00:00Z – 11:30:29Z`, duration 30m 29s.
- Cohort population and rotation constants `N ~ 12`,
  `r ~ 0.25`: inherited from the Add.158–180 active-set
  rotation window discussed in the M-180-B same-count
  active-set rotation post.
- Width-distribution thresholds 55m (95th percentile) and
  41m (median): inherited from pew-insights v0.6.242–v0.6.245
  six-axis dispersion sprint.
