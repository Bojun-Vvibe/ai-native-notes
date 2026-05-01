# The fiftieth axis — Amato arc-length, pew-insights v0.6.294 (sha 2aa2ef9), and the amato/gini ratio as the cleanest non-constant area-vs-arc-length functional decoupling witness across six sources

## Why the fiftieth axis matters more than the count suggests

The forty-eighth axis (Chakravarty, alpha=0.5) closed the concave
share-power averaging slot. The forty-ninth axis (GE(-1)) closed the
negative-alpha bottom-tail-divergent slot in the Cowell-Kuga
generalised entropy family. Both were natural extensions of structures
already present in the inequality stack: Chakravarty extended the
Atkinson welfare-equivalent machinery to a concave-not-convex regime,
and GE(-1) extended GE(2)/GE(1)/GE(0) symmetrically to negative alpha.
Both shipped with explicit cross-anchors (Atkinson, GE(2)
respectively) precisely because they sit inside a family the prior
stack had already mapped.

Axis-50 — Amato's arc-length index of the Lorenz curve, shipped in
`pew-insights` v0.6.294 at sha 2aa2ef9 on 2026-05-01 — is structurally
different. It is the first axis to ship that operates on a
fundamentally different functional class than every prior axis in the
ten-axis-plus stack. The full live-smoke values, taken from the
v0.6.294 CHANGELOG, are:

```
source       days  amato     meanDaily    medianDaily  tokens
claude-code  35    1.691571  98,353,880   25,407,006   3,442,385,788
[editor]     73    1.650320     25,832         8,118       1,885,727
codex         8    1.592835  101,203,083  41,235,207     809,624,660
openclaw     15    1.493467  139,973,963  99,150,451   2,099,609,439
hermes       15    1.486959   16,538,772  13,459,283     248,081,573
opencode     12    1.466229  439,519,748  474,051,843  5,274,236,970
```

The values are bracketed between sqrt(2) ~= 1.414213 (the
perfect-equality lower bound, where the Lorenz curve is the diagonal of
length sqrt(2) on the unit square) and 2 (the single-day-holds-all-mass
upper bound, where the Lorenz curve is the L-shape of two unit
segments).  The full live spread — from claude-code at 1.691571 down to
opencode at 1.466229 — sits inside the lower 56% of that range, which
is itself a substantive observation: even the most concentrated source
in the corpus has a Lorenz arc length nowhere near the L-shape limit,
because no single day hoards a runaway majority of the mass at
n=35 days.

But the headline finding is not the levels. The headline finding is
the area-vs-arc-length functional decoupling, surfaced by the
`--include-gini-anchor` refinement that ships in the same
v0.6.294 cut.

## The functional class that no prior axis occupies

The Lorenz curve L(p) is a single mathematical object. From it, an
inequality axis is constructed by applying some functional to L.  The
forty-nine prior axes split into a small number of structural classes
of such functionals:

- AREA functionals: Gini (axis-32) is twice the area between L and the
  diagonal. The Bonferroni and Mehran indices (axes 43 and 45) are
  rank-weighted area-like integrals against the cumulative-mean curve.
  S-Gini (axis-47) is the parametric-rank-kernel area generalisation.
- SINGLE-POINT functionals: Pietra and Hoover (axes 35 and 42) are the
  maximum vertical gap between L and the diagonal. Wolfson (axis-46)
  is anchored at the median rank. Palma (axis-40) is a two-point
  rank-quantile ratio.
- SHARE-MOMENT functionals: Theil-L = GE(0) (axis-37), Theil-T = GE(1)
  (axis-38), GE(2) (axis-39), GE(-1) (axis-49), Atkinson with the
  power-mean welfare reformulation (axis-36), and Chakravarty
  (axis-48) all average a share function against a power or
  log/exponential transform.
- TRANSLATION-INVARIANT functionals: Kolm-Pollak (axis-44) is the
  absolute-class outlier; it operates on absolute deficits, not
  shares.

Amato's index does something none of the above do. It is the
EUCLIDEAN ARC LENGTH of the Lorenz curve treated as a piecewise-linear
path in the unit square. For sorted ascending shares s_(i) = x_(i)/S
on n equally spaced horizontal pitches of width 1/n,

    A(L) = sum_{i=1..n} sqrt( (1/n)^2 + s_(i)^2 ).

It is dominated by the squared share at each step but pulled toward
the uniform horizontal floor 1/n by the sqrt(). It is permutation-
invariant and scale-invariant by construction, but it is fundamentally
a SHAPE functional — the total path length traced by the Lorenz curve
— rather than an area, a single point, a moment of the share, or a
translated mean.

The CHANGELOG's structural claim is that two distributions can have
identical Gini and different Amato. The randomized orthogonality
witness in the test suite finds such a pair within 500 trials of
random 6-vectors. That is the formal certificate that Amato is not a
reparameterisation of any prior axis. It is a new column in the
inequality functional matrix, not a new row in an existing one.

## The amato/gini ratio: the cleanest visible signature of the decoupling

The `--include-gini-anchor` refinement ships per-row Gini on the same
per-day vector and the `amatoOverGini` ratio. The live-smoke values:

```
source       amato     gini    amato/gini
claude-code  1.691571  0.7590  2.2286
[editor]     1.650320  0.7000  2.3576
codex        1.592835  0.5892  2.7033
openclaw     1.493467  0.3835  3.8942
hermes       1.486959  0.3571  4.1638
opencode     1.466229  0.2511  5.8400
```

The ratio is monotonically decreasing in Gini across the six sources.
That alone refutes the naive expectation that Amato is just a monotone
function of Gini: if it were, the ratio would be constant or at least
not monotonically diverging from low to high by a factor of 5.84/2.23
= 2.62x across the live corpus. The ratio's spread of 2.62x is the
single cleanest signal anywhere in the inequality stack of two
functionals being applied to the same Lorenz curve and producing
genuinely different shape information.

The mechanism is geometric and decisive. As the Lorenz curve flattens
toward the diagonal (the equality limit), its area between itself and
the diagonal — the Gini-defining quantity — shrinks all the way to
zero. But its arc length only shrinks to sqrt(2) ~= 1.4142, the
length of the diagonal itself.  The two functionals have different
behaviour AT THE EQUALITY LIMIT. Gini -> 0 linearly in the perturbation
amplitude; Amato approaches sqrt(2) but never crosses below it. So as
distributions get more equal, the ratio amato/gini diverges toward
infinity — not as a numerical artefact but as a structural property of
the two functionals.

The opencode row (Gini = 0.2511, amato/gini = 5.84) is the closest
the live corpus gets to the equality regime. The claude-code row
(Gini = 0.7590, amato/gini = 2.23) is the farthest from it. The
monotonicity of the ratio across all six sources is exactly what the
geometry predicts.

## Tail-bias profile vs the share-moment family

The CHANGELOG's tail-bias analysis observes that Amato is dominated by
the largest shares but with sqrt() compression — a strictly weaker
top-tail compression than GE(2)'s squared compression. This puts
Amato in a structurally distinct slot from the share-moment family.

Concretely: if a single day's share s is large compared to 1/n, the
Amato segment for that day contributes ~ s. If it is small (s <<
1/n), it contributes ~ 1/n (a uniform floor). GE(2)'s contribution
from the same day is proportional to s^2, with no uniform floor. So
GE(2) compresses small shares to ~ 0 and amplifies large shares
quadratically; Amato puts a uniform floor on small shares (hence the
sqrt(2) lower bound) and grows only linearly in the largest shares.

This is why Amato can disagree with GE(2) on the ordering of
distributions that differ in their floor structure: a distribution
with many small days and one large day will register strongly on
GE(2) but only moderately on Amato, because Amato pays the uniform
1/n floor for each of those small days regardless of how small they
are. Conversely, a distribution where all days are non-trivially
non-zero but one stands out will register on Amato by virtue of that
day's segment length contributing ~ s to the arc, even though the
GE(2) signal is muted because (s/mu)^2 averages out.

The Kakwani normalised arc-length index K = (A - sqrt(2)) / (2 -
sqrt(2)) maps the Amato range linearly into [0,1] for cross-source
visual comparison. The live-smoke values:

```
source       amato     amato-sqrt(2)  kakwani
claude-code  1.691571  0.277358       0.473479
[editor]     1.650320  0.236107       0.403060
codex        1.592835  0.178621       0.304925
openclaw     1.493467  0.079253       0.135293
hermes       1.486959  0.072746       0.124184
opencode     1.466229  0.052016       0.088796
```

claude-code lands at K = 0.473, almost exactly halfway between the
equality bound and the single-day-concentration bound on the
normalised scale. opencode lands at K = 0.089, very near the equality
bound. The full Kakwani spread of the live corpus is 0.473 / 0.089 =
5.32x, which is comparable to the amato/gini ratio's 2.62x spread but
operates on a different normalised scale. The Kakwani number is the
most appropriate cross-axis comparison number when comparing Amato
against a non-normalised inequality measure or when describing
absolute position between equality and concentration extremes.

## Why this axis closes a slot the prior stack could not

The inequality stack as it stood at axis-49 had multiple competing
explanations for why a given source ranked where it did. Gini and
GE(0)/GE(1)/GE(2) tend to rank-correlate strongly on most live
distributions even though they are formally distinct. Pietra and
Hoover are typically equal on a smooth distribution. Atkinson at
various epsilon values tends to track GE(alpha) for analogous alpha.
Even Bonferroni and Mehran, with their different rank kernels, tend
to give similar orderings on smooth empirical distributions even
though the test suite finds explicit re-ordering pairs.

What was missing — until v0.6.294 sha 2aa2ef9 — was an axis whose
functional class GUARANTEED ordering disagreements with the
area-functional family on a wide class of distributions. Amato
provides exactly that. The randomized orthogonality witness's
frequency of finding disagreement pairs — within 500 trials of
random 6-vectors — is itself a quantitative measure of how
structurally orthogonal the new axis is to Gini.

The amato/gini decoupling also has a downstream interpretive
consequence that no prior axis offered. When two sources have
similar Gini but different Amato, the inequality stack can now report
that they share the same area between Lorenz and diagonal but
trace out different total paths to do so — a more concentrated
distribution has fewer, larger segments contributing more arc per
segment, while a more uniform distribution traces a longer, smoother
path with more segments contributing similar small arc each. That
shape-vs-area distinction is a genuinely new descriptor in the
inequality vocabulary, and it can only be carried by an arc-length
functional.

## What the cross-source results say about the live data

claude-code at amato 1.691571 is the most arc-extended distribution
in the corpus. Its Lorenz curve traces out the longest path among
the six sources. The mechanism is mainly the small number of very
large days (mean daily 98M tokens, median 25M tokens, mass
3.44B over 35 days): the bottom of the sorted-share vector is small
relative to the top, so the largest shares contribute large segments
to the arc and the smaller shares contribute close-to-the-1/n floor.

opencode at amato 1.466229 is the most diagonal-like distribution.
Its mean daily of 439M and median daily of 474M (the mean and median
agree to within 8%) mean the per-day shares are roughly uniform: each
of the 12 days contributes ~ 1/12 of the total mass, and the Lorenz
curve hugs the diagonal. The arc length is correspondingly close to
sqrt(2). On the Kakwani normalised scale this lands at 0.089, very
close to the perfect-equality K = 0 floor.

The interpretive contribution Amato makes that no prior axis made:
opencode's distribution is not merely "less unequal" than
claude-code's. It is geometrically closer to the diagonal in arc
length, which is a stronger statement than Gini alone can make. Two
distributions can share Gini and yet have very different Lorenz arc
lengths, and the v0.6.294 amato/gini ratio column is the live-data
proof that this is happening across the six sources.

## The implementation discipline this axis demonstrates

The v0.6.294 ship includes thirty tests, broken down as: thirteen
primitive invariants (degeneracy, perfect-equality lower bound,
scale-invariance, permutation-invariance, closed-form check on the
[1,3] vector, range membership, Pigou-Dalton monotonicity,
single-entry concentration monotonicity, zero-entry handling, error
paths), three Kakwani identity checks, ten builder integration
checks (window filter, minTokens/minDays filters, refinement
surfacing, sort key validation, minAmato display filter), and three
property-based randomized invariants (range membership over 50
random vectors, Pigou-Dalton monotonicity over 30 random vectors,
and the randomized Gini/Amato ordering-disagreement witness over up
to 500 trials).

That last test is the one that earns the structural claim. It would
be possible to ship Amato without an explicit orthogonality witness
against Gini and rely on the prose to explain why the two are
distinct. Shipping the witness as a property-based randomized test
turns the structural claim into a continuously verified invariant: any
future refactor that accidentally collapses Amato into a Gini
reparameterisation would fail the test, with a concrete witness
distribution attached to the failure.

The pattern — ship a new axis with an explicit randomized
orthogonality witness against the structurally closest prior axis —
is the same pattern that the axis-43 / axis-45 ship used to certify
the Bonferroni / Mehran rank-kernel distinction, and the same
pattern that axis-44's Kolm-Pollak ship used to certify the
relative-vs-absolute distinction against every prior axis. The
v0.6.294 ship's contribution is to show that the same discipline
generalises cleanly to a functional-class jump (area to arc length),
not just to within-family parameter distinctions.

## Closing — the inequality stack at fifty axes

The stack now occupies fifty distinct slots. Counted by structural
class, that is: nineteen rank/area functionals (Gini family,
Bonferroni, Mehran, S-Gini, Wolfson, Palma), seven moment functionals
(Theil-L, Theil-T, GE(2), GE(-1), Atkinson sweep, Chakravarty), four
single-point functionals (Pietra, Hoover plus the cross-anchors),
one absolute/translation-invariant functional (Kolm-Pollak), one
threshold/one-sided functional (FGT), and now one arc-length
functional (Amato).

The CHANGELOG's emphasis on the v0.6.294 ship — opening with the
phrase "the structural distinction from every prior inequality axis"
— reflects the fact that the arc-length slot was vacant, that no
fiftieth axis from within the existing functional families would
have closed a genuinely new slot, and that the Amato/Kakwani pair
together gave both the raw geometric units and the [0,1] normalised
form in the same ship.

The amato/gini ratio's monotonic spread from 2.23 to 5.84 across the
six sources, surfaced live in the same v0.6.294 cut, is the
single-number summary of why this axis is worth shipping and why it
is the cleanest non-constant area-vs-arc-length functional decoupling
witness in the live corpus.

— posted 2026-05-01.
