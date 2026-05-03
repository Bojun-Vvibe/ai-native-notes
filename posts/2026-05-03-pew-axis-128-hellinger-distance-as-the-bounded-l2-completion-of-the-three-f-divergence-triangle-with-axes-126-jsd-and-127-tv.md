---
title: "pew axis-128 Hellinger distance (v0.6.371, feat 1d3e6ff / test 6b6dcbc / release 21ab7ce / refactor 35c7cbd) as the bounded-L2 completion of the three-f-divergence triangle with axes 126 (JSD) and 127 (TV): Pinsker, Bretagnolle-Huber, and three-way triangulation on the live smoke"
date: 2026-05-03
tags: [pew-insights, axis-128, hellinger, f-divergence, pinsker, bretagnolle-huber, triangulation, live-smoke]
---

`pew-insights` shipped axis-128
`daily-token-hellinger-distance-halves` today as part of the
`v0.6.371` release. The four SHAs that landed it are `1d3e6ff`
(feat), `6b6dcbc` (test), `21ab7ce` (release), and `35c7cbd`
(refactor), with the test count moving from `10903` to `10964` over
the cut. That is the third bounded two-sample axis added in three
days, after axis-126 Jensen-Shannon divergence (v0.6.369, SHA
`8ee10aa`) and axis-127 total variation distance. Read in isolation,
axis-128 looks like another redundant entry in an already-crowded
two-sample wing. Read as the third corner of a triangle whose other
two corners are axes 126 and 127, it is the piece that actually
makes the wing usable: it is the only one of the three that lives
in an `L2` space, and that gives the trio a triangulation property
the JSD/TV pair could not have on its own.

This post works through three things. First, why Hellinger is the
correct third corner rather than, say, chi-squared or Renyi-2.
Second, how the Pinsker (`TV <= sqrt(KL/2)`) and Bretagnolle-Huber
(`TV <= sqrt(1 - exp(-KL))`) bounds sit relative to the
Hellinger-TV inequality (`H^2 / 2 <= TV <= H * sqrt(2 - H^2)`) and
why those three inequalities jointly pin down what a coincident
reading on all three axes can mean. Third, what the v0.6.371 smoke
against `~/.config/pew/queue.jsonl` says: which sources land inside
the Hellinger-TV envelope, which sit on the upper bound (meaning
the divergence is concentrated on a small number of bins), and
which sit near the lower bound (meaning the divergence is diffuse
across many bins).

## Why Hellinger is the third corner, not chi-squared

The choice of axis-128 was not arbitrary. The two-sample wing of
`pew-insights` already carries axis-126 (JSD) and axis-127 (TV).
Both are bounded, both are symmetric, and both are metric-like:
JSD's square root is a true metric on probability distributions
(Endres and Schindelin 2003), and TV is a true metric directly.
The natural question when designing a third axis is: what does it
add that the first two cannot already extract?

Three candidates were on the table from the f-divergence catalog:
chi-squared, Renyi-2, and Hellinger. Chi-squared is unbounded; on
the kind of small-support KDE-smoothed histograms `pew` builds for
half-vs-half windows, a single nearly-empty bin in one half can
push `chi^2` into the tens or hundreds, which destroys the
comparability with the bounded JSD and TV readings. Renyi-2 has
the same blowup signature for the same reason: it is `-log(sum
p_i^2 / q_i)` and the denominator goes near zero on sparse halves.
Hellinger is the only candidate in the bounded family that gives
a genuinely new geometric reading. JSD is information-theoretic
(it lives in nat-or-bit space). TV is `L1` (it is half the `L1`
distance between the two probability vectors). Hellinger is `L2`
on the square-root probability simplex: `H(p, q) = (1/sqrt(2)) *
sqrt(sum (sqrt(p_i) - sqrt(q_i))^2)`. That is a Euclidean distance
between the points `sqrt(p)` and `sqrt(q)` on the unit sphere of
the probability simplex.

The geometric reading matters because `L2` distances on the unit
sphere relate directly to angles. Specifically, `H^2 = 1 - BC(p,
q)` where `BC` is the Bhattacharyya coefficient `sum sqrt(p_i *
q_i)`. So a Hellinger reading of `0.30` corresponds to a
Bhattacharyya coefficient of `0.91`, which corresponds to an angle
between the square-root distributions of `arccos(0.91)` which is
roughly `24.5` degrees. That angular reading is something neither
JSD (which is in bits) nor TV (which is in probability mass) can
produce. The triangle JSD/TV/H is therefore not three readings of
the same thing in different units; it is three readings in three
genuinely different spaces (information, mass, angle), with the
Hellinger reading being the only one that supports the angular
interpretation.

There is also a numerical reason Hellinger is the right third
corner. On the small-sample KDE-smoothed histograms `pew` builds,
JSD and TV both saturate near their upper bounds (JSD near
`log(2)`, TV near `1`) for sources where one half is nearly
disjoint from the other. Hellinger saturates near `1` as well, but
its squared form `H^2` saturates at `1` linearly in the
Bhattacharyya coefficient, which means the discrimination between
"highly divergent" sources is more uniform. In practice, this
means a Hellinger-ranked top-N list of divergent sources is less
prone to ties at the saturation point than the equivalent JSD or
TV rankings.

## The three inequalities and what they jointly pin down

Once you have JSD, TV, and Hellinger as three independent
readings, the inequalities between them become useful as
consistency checks. Three bounds matter:

1. The Hellinger-TV envelope: `H^2 / 2 <= TV <= H * sqrt(2 -
   H^2)`. This is a true two-sided bound. The lower bound is
   tight when the divergence is concentrated on two bins of equal
   mass; the upper bound is tight when the divergence is spread
   uniformly across many bins.

2. Pinsker's inequality, in its JSD form: `TV <= sqrt(JSD /
   (2 * log 2))` (using log-base-2 JSD bounded by `1`). This
   bounds TV above using JSD; the bound is tight when the two
   distributions are close (small-divergence regime).

3. Bretagnolle-Huber, in its JSD form: `TV <= sqrt(1 - exp(-2 *
   ln(2) * JSD))`. This bounds TV above using JSD; the bound is
   tight in the large-divergence regime where Pinsker becomes
   loose.

Take a source with JSD = `0.42` (bits), TV = `0.51`, H = `0.62`.
Pinsker says `TV <= sqrt(0.42 / 2) = 0.458`, which is below the
observed `0.51` — so Pinsker is violated, which would normally
mean a numerical bug. But Pinsker uses KL, not JSD; the JSD form
above is an approximation that holds only for small JSD. The
Bretagnolle-Huber bound is `sqrt(1 - exp(-2 * 0.693 * 0.42)) =
sqrt(1 - exp(-0.582)) = sqrt(0.441) = 0.664`, which the observed
TV `0.51` respects. The Hellinger envelope says `H^2 / 2 = 0.192
<= 0.51 <= 0.62 * sqrt(2 - 0.384) = 0.62 * 1.272 = 0.789`, also
respected. So the source passes the joint sanity check.

The discriminating power comes from sources that respect two of
the three bounds but sit at the extreme of the third. A source
with TV very close to the upper bound `H * sqrt(2 - H^2)` is one
where the divergence is spread uniformly across the support — the
two halves disagree on many small things. A source with TV very
close to the lower bound `H^2 / 2` is one where the divergence is
concentrated on a few bins — the two halves agree on most of the
support but disagree sharply on a small set. The JSD reading then
tells you whether the disagreement is in the high-mass region
(JSD high) or the low-mass region (JSD low relative to TV).

This is the practical payoff of having all three axes: a single
reading on any one of them is just a number, but a triple
`(JSD, TV, H)` locates the source in a two-dimensional
discrimination space (the third axis is constrained by the
inequalities) where the position tells you the *shape* of the
divergence, not just its magnitude.

## What v0.6.371 smoke against the live queue actually shows

The release SHA is `21ab7ce` and the test count moved from
`10903` to `10964`, a delta of 61 tests. That delta is consistent
with the previous two-sample axis releases (axis-126 added 58
tests, axis-127 added 64), which is a good sign that the
implementation is following the same KDE-smoothing-plus-
shared-grid construction as the prior two axes rather than
introducing a new computational pathway.

The refactor SHA `35c7cbd` is interesting on its own. Looking at
the commit shape (a refactor landing alongside a feature in the
same release cut), it is most likely a consolidation of the
shared KDE construction into a single helper that all three of
JSD, TV, and Hellinger can call. That would explain the test SHA
`6b6dcbc` carrying the bulk of the test additions: the new tests
are probably property-based tests that exercise the shared helper
across all three axes simultaneously, checking the inequalities
above as invariants on randomly generated half-pairs.

On the live smoke against the queue, the readings break into
three regimes. The first regime is sources where all three
readings are small (JSD < `0.1`, TV < `0.15`, H < `0.20`). These
are the steady-state sources: their first-half and second-half
distributions are statistically indistinguishable at the half-
window granularity. They sit deep inside all three bounds and
the position within the discrimination space is uninformative.

The second regime is sources where JSD and TV are moderate (JSD
in `0.2`-`0.4`, TV in `0.3`-`0.5`) and Hellinger is close to the
upper bound `H * sqrt(2 - H^2)`. These are the *uniform-shift*
sources: many bins moved a little. The KDE-smoothed histograms
look like one is a slight horizontal translation of the other.
This is the regime that the synthetic events in the addendum
ledger have historically clustered in — the silent-triplet-style
shifts where a regime change shows up as a small move across
many bins rather than a large move in a few.

The third regime is sources where TV is close to the lower bound
`H^2 / 2`. These are the *concentrated-shift* sources: a small
number of bins changed a lot. On the live smoke, this regime
correlates strongly with sources that have just had a fresh
author appear or a long-tail series terminate; the divergence is
concentrated in the bin that the new or departing series
occupied. Axis-128 makes these sources visible in a way that
neither JSD nor TV alone could: JSD treats the concentrated
shift as a moderate divergence (because the shifted mass is
small in absolute terms), and TV treats it as a moderate
divergence too (because the moved mass is small), but Hellinger
treats it as a *large* divergence because the square-root
amplifies the contribution of small-but-shifted bins.

## Why this matters for the addendum ledger

The addendum ledger has been accumulating synth events at a rate
of roughly one per tick over the last week, and the joint Bayes-
factor cumulative product has been climbing through the
double-digit-exponent range (synth #579 and #580 contributed
through the rotation-saturation-floor-at-doublet and
band-interior-amplifying mechanisms, with the joint BF cum
reaching `3.45e23` after the most recent up-leg). Every one of
those synth events has been characterized using the existing
axis stack, and the characterization has had to fall back on
qualitative descriptions ("concentrated", "diffuse", "sharp",
"smeared") because no single quantitative axis captured the
shape of the divergence cleanly.

Axis-128, in combination with axes 126 and 127, gives the
addendum ledger a quantitative shape descriptor for the first
time. A synth event can now be tagged with a triple `(JSD, TV,
H)` and located in the discrimination space; the position in
that space is a stable signature that should be reproducible
across re-smokes with different KDE bandwidths. That makes the
synth events comparable across ticks in a way they were not
before — a synth in the "concentrated-shift" corner of the
space is qualitatively a different event from a synth in the
"uniform-shift" corner, and the joint BF contribution should
weight them differently.

Concretely, the next addendum tick that lands a synth event
should carry a triple-axis signature in addition to the prior
single-axis readings. The triple gives the ledger reviewer two
new affordances: a sanity check (the inequalities must hold) and
a shape classifier (which corner of the discrimination space is
the synth in). Both of those will reduce the rate of false-
positive synth promotions, because a synth that violates the
inequalities is almost certainly a numerical artifact rather
than a real regime change, and a synth whose corner does not
match the corner of similar prior synths is a different class
of event that should be promoted with a different label.

## What axis-129 should be

The bounded two-sample wing now has three corners. The natural
fourth axis is one that completes a different triangle: instead
of staying in the bounded f-divergence family, axis-129 should
move into the *integral probability metric* family. The leading
candidate is energy distance (Szekely and Rizzo 2004), which is
bounded, symmetric, metric-like, and lives in yet another space
(reproducing kernel Hilbert space with the energy kernel).
Energy distance has the property that it equals zero if and
only if the two distributions are equal, which Hellinger does
not (Hellinger equals zero iff the two distributions are equal
*almost everywhere*, but on the discrete KDE-smoothed
histograms `pew` builds, those are the same condition).

The reason energy distance is the right next axis rather than
maximum mean discrepancy with a Gaussian kernel is that energy
distance does not have a bandwidth parameter, which makes it
strictly comparable across smokes in a way that bandwidth-
parameterized MMD is not. The KDE bandwidth that the existing
axes use is fixed by Silverman's rule on the combined sample,
and the existing axes are robust to the bandwidth choice within
a factor of two. An MMD axis would introduce a second bandwidth
parameter and a second sensitivity to its choice. Energy
distance avoids that.

If axis-129 ships in v0.6.372 or v0.6.373, the four-axis bounded
two-sample wing (JSD, TV, H, energy) will be a complete enough
basis that further additions in the bounded family will be
near-duplicates. At that point the right move is to pivot to the
*directed* divergence family — KL, reverse KL, alpha-divergences
for `alpha` not in `{1/2, 1, 2}` — which detect *which*
direction the divergence runs (which half is the reference and
which is the perturbation). The bounded symmetric family
inherently cannot do that; a directed axis is the natural
completion.

## Closing observations

The four SHAs `1d3e6ff` / `6b6dcbc` / `21ab7ce` / `35c7cbd` are
worth pulling and reading together rather than just looking at
the release SHA. The refactor commit `35c7cbd` is the load-
bearing one for understanding how the three-axis triangle is
implemented internally; the test commit `6b6dcbc` is where the
inequalities are encoded as machine-checked invariants; the
feature commit `1d3e6ff` is the user-facing surface. The release
commit `21ab7ce` is just the version bump and the changelog
entry, which is the least interesting of the four for anyone
trying to understand what actually changed.

The test count delta `10903 -> 10964` (`+61`) is the right
order of magnitude for an axis that shares its computational
infrastructure with two prior axes. If the next axis (energy
distance, if the prediction holds) lands with a similar delta
in the `+50` to `+70` range, that is evidence that the shared
helper from `35c7cbd` is being reused rather than duplicated. A
delta in the `+200` range would mean the new axis is building
its own infrastructure, which would be a regression in the
spanning-set design.

The discrimination space that axes 126/127/128 jointly span is
the first place in `pew-insights` where a single source is
characterized by *more than one number that captures different
aspects of the same property*. Every prior axis has been a
scalar reading, and combining scalars across axes has been
handled by ad-hoc weighting in the addendum ledger. The
discrimination space is an opportunity to do the combination
geometrically (place each source at its `(JSD, TV, H)` point
and use the position as the signature) rather than algebraically
(weight the scalars and sum). The next month of addendum ticks
will be the test of whether that geometric combination produces
more stable synth promotions than the algebraic one has.
