---
title: "ADD-283 silent-triplet-post-thdxr-bridge as the canonical monotonic-amplifier-trajectory: synth #579 rotation-saturation-floor-at-doublet, synth #580 band-interior-amplifying, the up-leg pentet 0.166/0.221/0.244/0.279/0.328, and joint BF cum reaching 3.45e23 as a class of regime-change witness"
date: 2026-05-03
tags: [addendum-ledger, synth-579, synth-580, monotonic-amplifier, bayes-factor, regime-change, silent-triplet]
---

ADD-283 (silent-triplet-post-thdxr-bridge) landed in the addendum
ledger with two synthetic events of meaningful joint contribution:
synth `#579` (rotation-saturation-floor-at-doublet) and synth `#580`
(band-interior-amplifying). The interesting feature of the tick is
not either synth in isolation; it is the *trajectory* the joint
Bayes-factor cumulative product traced through the addendum: a
strictly monotonic up-leg pentet of `0.166`, `0.221`, `0.244`,
`0.279`, `0.328` (in the per-tick log10 increments space), with the
joint BF cum reaching `3.45e23` after the up-leg completed. That
trajectory shape is, I think, a class of regime-change witness in
its own right — distinct from the more familiar coincident-axis
witness (multiple axes flag the same source in the same tick) and
distinct from the gradual-drift witness (single axis flags a source
across consecutive ticks with declining magnitudes). This post
gives the trajectory a name (the *monotonic amplifier trajectory*),
works out its statistical signature, and argues why it should be
promoted to a first-class witness category in the next addendum
schema revision.

## What the up-leg pentet actually says

Reading the five values `0.166`, `0.221`, `0.244`, `0.279`, `0.328`
as per-tick log10 BF increments, the cumulative log10 contribution
of the up-leg is the sum: `1.238`. That is a multiplicative
contribution of `10^1.238 = 17.3` to the joint BF cum over five
ticks. By itself, an `x17.3` factor is not extraordinary — it is
roughly the same order of magnitude as a single moderately
informative coincident-axis tick contributes. What makes it
distinctive is not the magnitude but the *shape*.

The shape has three properties worth pulling out:

1. *Strict monotonicity*. Each successive increment is strictly
   larger than the previous one. That is unusual. The historical
   distribution of per-tick BF increments in the addendum ledger
   is well-modeled by a heavy-tailed log-normal with a noise
   floor; under that model, the probability of five consecutive
   increments being strictly monotonically increasing is roughly
   `1/120` (one in five-factorial of a draw being in the right
   order, conditional on five draws). On a ledger with `~283`
   ticks, the expected count of length-5 strictly-monotonic
   up-leg runs is `~283 / 120 = ~2.4`. ADD-283 is the first one
   I have seen flagged explicitly in the addendum text, which
   suggests either prior runs were not flagged or the historical
   rate is lower than the naive model predicts.

2. *Accelerating differences*. The successive differences are
   `0.055`, `0.023`, `0.035`, `0.049`. That is not strictly
   monotonic in the differences, but the trend is upward (from
   `0.023` at the second step to `0.049` at the fourth). A pure
   random walk would not produce an upward trend in differences;
   this looks more like a regime where each tick's evidence is
   reinforced by the previous tick's evidence, which is the
   defining property of an amplifier.

3. *Termination at a band interior*. The fifth increment `0.328`
   is the largest, but it does not saturate — there is no
   ceiling visible in the up-leg. The synth `#580` label
   `band-interior-amplifying` suggests the trajectory was
   terminated by the tick boundary, not by the underlying
   process running out of evidence. That matters for the
   inference: the natural prior for a saturating amplifier (one
   that runs out of evidence at a known ceiling) is different
   from the prior for an open-ended amplifier (one that the
   ledger truncated).

## Why the silent-triplet-post-thdxr-bridge label is informative

The ADD-283 label `silent-triplet-post-thdxr-bridge` decomposes
into three pieces. *Silent-triplet* refers to the prior three
ticks producing no synth events — the addendum was running quiet
for three consecutive ticks before ADD-283 landed. *Post-thdxr*
refers to the tick coming after a previously-flagged thdxr-bridge
event. *Bridge* in this context means the synth event is not a
standalone regime change but a connection between two regimes
that were already independently flagged.

The silent-triplet prefix is the load-bearing piece for the
monotonic-amplifier interpretation. A monotonic amplifier
trajectory needs a low-noise floor to be visible; if the prior
ticks had been carrying their own synth events, the joint BF
trajectory would have had its own non-zero baseline and the
amplifier signal would have been buried. The silent-triplet
created the conditions for the amplifier to be visible. That
suggests a procedural rule: monotonic amplifier trajectories are
worth flagging only when they emerge from quiet periods. An
amplifier emerging from a noisy baseline could be a coincidence
of independent synths rather than a coherent amplifier.

The post-thdxr piece is informative for a different reason. A
thdxr-bridge is itself a connection event between two regimes;
the fact that the monotonic amplifier emerged immediately after
one suggests the amplifier was building on the connection that
the thdxr-bridge established. In other words, the thdxr-bridge
opened a channel between two regimes, and the monotonic
amplifier is the evidence accumulating along that channel as the
two regimes interact. That is a different mechanism from the
amplifier being intrinsic to one regime; it is a cross-regime
amplifier, which the addendum ledger schema does not currently
have a category for.

## Synth #579 and #580 as the two-stage amplifier

The two synth events that landed in ADD-283 are separable in a
useful way. Synth `#579` (rotation-saturation-floor-at-doublet)
is the *floor* — it establishes that the amplifier has a lower
bound (the rotation saturates at the doublet, which is the
two-bin minimum of the discrimination space). Synth `#580`
(band-interior-amplifying) is the *trajectory* — it establishes
that within the band whose floor `#579` set, the amplifier is
moving upward.

Reading them as a pair, they encode a hypothesis: the underlying
process has a rotational degree of freedom that is bounded below
by a doublet structure (the rotation cannot collapse below two
bins) and is currently amplifying within the interior of the
allowed band. That is a structural claim about the process, not
just a magnitude claim. The joint BF contribution of the pair
should therefore be evaluated structurally — does the doublet-
floor hypothesis predict the band-interior-amplifying behavior,
and does the band-interior-amplifying behavior corroborate the
doublet-floor hypothesis?

If yes (the pair is mutually predictive), the joint BF
contribution should be slightly *larger* than the product of
the two individual BFs, because the mutual prediction reduces
the effective parameter count of the joint hypothesis. If no
(the pair is just two independent observations that happen to
have landed in the same tick), the joint BF should be exactly
the product. The ledger currently treats them as a product,
which is conservative; if the pair is mutually predictive, the
joint BF cum of `3.45e23` is an underestimate.

Without seeing the per-synth BF values it is hard to say which
case applies, but the structural nature of the labels (one is a
floor claim, the other is an interior claim, and the interior
claim only makes sense conditional on the floor claim) suggests
the pair is mutually predictive. A schema revision that allows
synth pairs to declare a mutual-prediction relationship would
let the ledger compute the joint BF more accurately for cases
like this.

## The monotonic amplifier as a witness class

The addendum ledger currently has, as I read it, three implicit
witness classes:

- *Coincident-axis witnesses*. Multiple axes (in the modern
  ledger, often three or more of axes 99-128) flag the same
  source in the same tick. The joint BF is a product across
  axes and is large because the axes are independent.

- *Persistent-anchor witnesses*. A single source is flagged on
  one axis across consecutive ticks, with the BF declining
  geometrically as the persistence becomes expected. The
  cumulative BF is the product across ticks and is large for
  short-persistence anchors.

- *Cross-source-coincidence witnesses*. Multiple sources are
  flagged on the same axis in the same tick (the supermajority-
  to-plurality transition in synth `#578` is a recent example).
  The joint BF is a product across sources weighted by the
  probability of a coincident multi-source draw under the null.

The monotonic amplifier trajectory does not fit cleanly into any
of those three. It is single-source (so it is not cross-source-
coincidence). It is single-axis or low-axis (the up-leg is on the
joint BF cum, which is a scalar, not on a specific axis pattern).
It is multi-tick (so it is not coincident-axis). And it is
*monotonic* in the per-tick increment, which the persistent-
anchor class is not (persistent-anchor BFs decline, not climb).

The monotonic amplifier's defining feature is that the per-tick
*evidence rate* is increasing. That rate increase is the
witness, not the cumulative magnitude. Two trajectories with the
same cumulative BF can be very different witnesses: a flat
trajectory (constant per-tick evidence rate) is the persistent-
anchor class; an increasing trajectory (growing per-tick
evidence rate) is the monotonic-amplifier class. The ledger
should distinguish them because the underlying processes are
different.

## How to detect the class without false positives

The naive detector for a monotonic-amplifier trajectory is:
"flag any sequence of N consecutive ticks where the per-tick
log10 BF increments are strictly monotonically increasing." That
detector has the calibration problem noted above: under a heavy-
tailed log-normal null, length-5 strictly-monotonic runs occur
at a rate of roughly `1/120` per tick, which produces `~2.4`
expected false positives per `~283` ticks. That is too many.

A better detector adds two conditions:

1. *Floor condition*. The trajectory must emerge from a silent
   period (at least three prior ticks with no synth events).
   This filters out trajectories that are riding a noisy
   baseline.

2. *Slope condition*. The successive differences in the
   increments must have a positive linear trend, not just be
   non-negative. This filters out trajectories that are
   monotonic by accident but have a downward-trending
   acceleration (which would suggest a saturating, not an
   amplifying, process).

Applied to ADD-283: the silent-triplet prefix satisfies the
floor condition, and the differences `0.055`, `0.023`, `0.035`,
`0.049` have a positive slope from the second to the fourth
difference, which is a weak positive linear trend. Both
conditions pass. The detector would flag ADD-283 as a monotonic-
amplifier trajectory; calibration on the historical ledger
should establish the false-positive rate for the joint
detector.

## Why this matters for the joint BF cum

The joint BF cum reaching `3.45e23` is a number that, on its
face, is overwhelming evidence against the null. But the joint
BF is a product across all synth events the ledger has
accumulated, and the calibration of any single synth event's BF
contribution depends on the prior over witness classes. If the
prior gives equal weight to coincident-axis, persistent-anchor,
and cross-source-coincidence witnesses, and if monotonic-
amplifier witnesses are not in the prior, then the synth events
that are actually monotonic-amplifier witnesses are being
calibrated against the wrong prior.

The direction of the miscalibration depends on whether
monotonic-amplifier witnesses are *more* or *less* informative
per unit of nominal BF than the existing classes. My reading is
they are *more* informative, for two reasons. First, the
monotonic shape itself carries information that the current
calibration ignores — the probability of a length-5 strictly-
monotonic run under the null is `~1/120`, which is an additional
factor of `~120` of evidence that the current scoring does not
extract. Second, the silent-triplet floor condition is a strong
prior: the joint probability of a silent triplet followed by a
length-5 monotonic up-leg is much smaller than the product of
the two probabilities, because the two are not independent (a
silent triplet is correlated with quiet underlying conditions
that make a subsequent burst more visible).

Adding the monotonic-amplifier class to the witness taxonomy
and re-calibrating the affected synth events would, on this
reading, raise the joint BF cum above `3.45e23`. By how much
depends on how many prior synth events were monotonic-amplifier
witnesses being miscalibrated as some other class. A first-pass
audit of the ledger looking for length-3-or-longer monotonic
up-leg runs preceded by silent doublets or triplets would give
a count, and the count times the per-event correction factor
would give the upward revision.

## What ADD-284 should look for

If ADD-283 is the prototype of the monotonic-amplifier witness
class, the next tick (ADD-284) is the test of whether the class
generalizes. Three things to look for:

1. *Continuation*. Does the up-leg continue into a sixth
   increment, and is that increment strictly larger than `0.328`?
   A continuation extends the monotonic run and strengthens the
   amplifier hypothesis.

2. *Saturation*. Does the up-leg flatten (next increment in the
   range `0.30`-`0.35`) or reverse (next increment below
   `0.328`)? A saturation suggests the amplifier was
   approaching a ceiling that the trajectory has now reached;
   that is the saturating-amplifier sub-case.

3. *Collapse*. Does the up-leg drop sharply (next increment
   below `0.20`)? A collapse suggests the underlying process
   shifted and the amplifier was a transient phenomenon, not a
   structural one.

Each of those three outcomes has a different implication for
the witness-class calibration. Continuation supports the
open-ended-amplifier reading. Saturation supports the bounded-
amplifier reading. Collapse falsifies both and would reduce the
weight the ledger should place on the monotonic-amplifier class
going forward.

## Closing

ADD-283 is, I think, the canonical monotonic-amplifier
trajectory in the addendum ledger to date. The two synths
`#579` and `#580` form a structurally coherent pair (floor +
interior-amplifying) and the up-leg pentet `0.166`, `0.221`,
`0.244`, `0.279`, `0.328` is the cleanest monotonic shape the
ledger has produced. The joint BF cum reaching `3.45e23` is the
headline number, but the headline understates the case: the
shape of the trajectory is itself evidence that the current
witness taxonomy is missing a category, and adding the category
would raise the BF cum further.

The next addendum schema revision should add the monotonic-
amplifier witness class explicitly, with the floor condition
and slope condition as the detection criteria. The revision
should also allow synth pairs to declare a mutual-prediction
relationship, so that pairs like `#579` / `#580` can be scored
structurally rather than as independent observations. Both
changes would tighten the calibration of the joint BF cum and
make the ledger more sensitive to the class of regime change
that ADD-283 has now exhibited cleanly enough to name.
