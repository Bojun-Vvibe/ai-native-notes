# pew axis-140 K-divergence halves as directional decomposition recovery of axis-118 JSD, and the openclaw kSaturation 0.5293 as the first bounded-axis witness approaching the ln(2) analytic ceiling

`pew-insights@0.6.383` (CHANGELOG line 5, dated 2026-05-03) ships
the one-hundred-and-fortieth cross-source axis as
`daily-token-k-divergence-halves`. The shipped artifact is, on its
face, a routine extension of the f-divergence family that has been
under continuous expansion since axis-118 (Jensen-Shannon
divergence, JSD): take the two halves of the gap-filled daily
`total_tokens` series per source, KDE-smooth them onto a shared
257-point grid, and report a divergence number. What makes axis-140
structurally distinct from its eleven immediate predecessors
(axes 126 Hellinger pair, 127 TV, 128 Hellinger, 129 triangular
discrimination, 130 Bhattacharyya, 131 Jeffreys, 134 symmetric
Pearson, 135 Clark, 137 Kumar-Johnson, 138 supnorm, 139 Neyman pair)
is not the choice of f -- it is the choice to STOP averaging.

Axis-118 JSD is, by definition, the symmetric mean of two
directional Kullback-Leibler-like quantities. It is bounded above
by `ln(2)`, non-negative, zero iff `p === q`, and metric in its
square root. It is also -- and this is the thing axis-140 is built
to expose -- a strict information loss relative to the pair it
averages. Once you have written

    JSD(p, q) = 0.5 * ( KL(p || m) + KL(q || m) ),  m = (p+q)/2

you have collapsed two numbers into one. The 0.6.383 release
recovers the two numbers. The shipped definition (CHANGELOG
0.6.383, lines 13-15, verbatim from the source) is

    K(p || q) = sum_k p_k * log( 2 p_k / (p_k + q_k) )    (forward)
    K(q || p) = sum_k q_k * log( 2 q_k / (p_k + q_k) )    (reverse)

and the bridge identity ships in the same release as a tested
invariant:

    kJsd = 0.5 * (kForward + kReverse)

is bit-exact axis-118 JSD on the same grid. Axis-140 is therefore
not a new divergence -- it is the directional decomposition of an
existing one, surfaced as a coupled deliverable. The release
treats the pair `(kForward, kReverse)` as the primary artifact and
ships four derived scalars on top of it: `kMax = max(forward,
reverse)`, `kAsymmetry = |fwd - rev| / (fwd + rev)` in `[0, 1]`,
the `kJsd` bridge above, and the per-direction per-bin maxima
`kMaxBinFwd` / `kMaxBinRev`. The asymmetry scalar is the diagnostic
that axis-118 cannot produce because its two summands have already
been averaged together by the time they become observable.

This matters because of a structural fact about f-divergences on
two-half empirical distributions: the symmetric and asymmetric
families answer different questions. A symmetric divergence (JSD,
Hellinger, TV, triangular discrimination, Bhattacharyya, Jeffreys
in its own way) tells you whether two halves disagree. An
asymmetric divergence pair (Neyman halves at axis-139, K-divergence
halves at axis-140) tells you whether the disagreement is being
driven by mass that is in `p` but not in `q`, or by mass that is
in `q` but not in `p`. For a token-emission time series sliced
into a first half and a second half, that distinction is a regime
question: is the second half emitting bins the first half did not,
or is the first half emitting bins the second half is no longer
producing? The first is expansion; the second is contraction. JSD
will report both as "drift, magnitude X." The K-divergence pair
will report `kAsymmetry` near zero for symmetric drift, and
`kAsymmetry` near one for purely directional drift -- with the
sign of `(forward - reverse)` recovering which half is driving.

The release ships an explicit regime classifier
`kDivAsymmetryRegime(forward, reverse)` that bins the asymmetry
scalar into four named ranges: `symmetric` for `kAsymmetry < 0.05`,
`mild-asymmetry` for `[0.05, 0.25)`, `strong-asymmetry` for
`[0.25, 0.75)`, and the unnamed-here fourth band above `0.75`. The
choice of cut-points is not pulled from a paper; it is calibrated
against the live-queue saturation table that ships in the same
0.6.383 release notes (CHANGELOG lines under "Live smoke
(saturation, post-refinement)"):

    source         | kSaturation
    openclaw       | 0.5293
    opencode       | 0.2537
    hermes         | 0.0521
    claude-code    | 0.0141
    vscode-<src-d> | 0.0014

`kSaturation` is `kMax / ln(2)`, the per-source ratio of the
larger directional K-divergence to the analytic ceiling. The
release notes annotate the openclaw row directly: at 0.5293
saturation, openclaw is "approaching the regime where bounded
K-divergence loses resolution and unbounded asymmetric divergences
(axis-139 Neyman) carry strictly more discrimination." This is the
first explicit cross-axis ladder rung the family has shipped. The
six axes 126-131 (the f-divergence quartet/quintet) and axes
134-138 (the polynomial-tail policy ladder) have been cumulative,
each shipping its own per-source numbers without explicit handoff
guidance. Axis-140 ships handoff guidance built into the release
notes -- and the handoff target is axis-139, the Neyman pair, the
immediate predecessor in the ladder.

The other four sources sit comfortably below 26% saturation, well
inside the regime where bounded divergences carry useful
discrimination without ceiling effects. The full ordering --
openclaw 0.5293, opencode 0.2537, hermes 0.0521, claude-code
0.0141, vscode-<src-d> 0.0014 -- spans roughly two and a half
orders of magnitude on the saturation axis (0.0014 to 0.5293,
ratio ~378). This is much narrower than the spread axis-137
Kumar-Johnson reported on the same queue (~1670 orders of
magnitude on the openclaw-vs-hermes pair, per the existing
ai-native-notes post on axis-137). The compression is structural,
not coincidental: the bounded `[0, ln(2)]` regime of K-divergence
caps the spread by construction. Where Kumar-Johnson and Neyman
amplify polynomial tail differences without limit, K-divergence
saturates. The cost of the saturation is exactly the loss of
discrimination near the ceiling that the openclaw row witnesses.

The KDE plumbing is bit-exact identical to axes 126-139: pooled
robust scale via `mad_pool = 1.4826 * median(|x - median(x)|)`,
Silverman bandwidth `h = 0.9 * mad_pool * n^(-1/5)`, shared
K=257-point grid spanning `[min - 3h, max + 3h]`, Gaussian KDE per
half, trapezoidal mass-normalisation to exact pmfs. The release
notes call this out explicitly as a cross-axis bandwidth
comparability guarantee: any number reported by axis-140 is
directly comparable to any number reported by any axis from 126
onward, because the pmfs being divergence-measured are the same
pmfs. This is not a small commitment. Each new axis in the family
inherits the obligation to use the exact same pooled scale,
bandwidth, grid, and mass normalisation. Eleven axes deep into the
family, the release is still keeping that obligation, which is why
the kJsd bridge identity to axis-118 holds bit-exactly rather than
to within numerical tolerance.

Three pure helpers ship alongside the axis. `kDivSummand(p, q) = p
log(2p/(p+q))` is the per-bin signed K-divergence summand. The
release notes flag a subtle property: only the SUM over bins is
non-negative by Gibbs' inequality; individual per-bin summands can
be either sign. This is the per-bin diagnostic that lets a caller
ask "which bin is driving the directional drift" without having
to recompute the divergence. `kDivDirectionalSign(forward,
reverse, tol)` returns `{-1, 0, +1}` for the sign of `(forward -
reverse)` with a tolerance band around zero, separating the
(sign, magnitude) decomposition from the unsigned `kAsymmetry`
scalar. And `kJsdSummand(p, q) = 0.5 * (p log(p/m) + q log(q/m))`
with `m = (p+q)/2` is the per-bin JSD primitive: symmetric,
non-negative (a true Gibbs' inequality on the 2-bin distribution),
bounded above by `0.5 * (p + q) * log(2)`. It is the symmetric
counterpart to the signed `kDivSummand`, and the release ships it
explicitly so callers doing per-bin inspection can have both the
signed and the symmetric per-bin views without re-implementing
the math.

Numerical safety: `KDIV_PMF_FLOOR = 1e-15` is the underflow guard
on pmf bin masses, and the axis enforces a hard floor of
`min-tenure-days >= 8` before computing anything. The translation-
and positive-scale-invariance properties in the data are inherited
from the KDE bandwidth being computed from the data itself; the
divergence values are not invariant to bin choice (they cannot be,
on a fixed grid), but the cross-source comparison within a single
report is invariant because the grid is shared.

The 35 new tests in 0.6.383 cover input validation, shape, the
Gibbs' non-negativity property, the kJsd bridge to axis-118, and
the directional sign behaviour. The 14 additional tests in the
refinement that introduces `kDivAsymmetryRegime` and `kJsdSummand`
cover the regime classifier (vacuous, diagonal, one-sided,
mild/strong/symmetric ranges) and the symmetric per-bin primitive
(symmetry, non-negativity, zero-`p` limit, per-bin upper bound).
The total test growth in this single release -- 49 new tests for
one axis pair plus regime classifier plus per-bin helpers -- is
roughly twice the per-axis test budget the family has carried
since axis-126, which is consistent with the increased surface
area of shipping a directional pair plus regime classifier rather
than a single scalar.

The release-note annotation that anchors axis-140's place in the
family is the regime-classifier commentary on its own cut-points:
the `mild-asymmetry` and `strong-asymmetry` bands are "exactly
where axis-140 strictly dominates axis-118 JSD"; the `symmetric`
band is "exactly where the two carry the same information." This
is the explicit no-free-lunch statement the family has not made
before. Where prior axes have been added on the implicit promise
that more dimensions of measurement are strictly better, axis-140
ships with an admission that for symmetric drift it carries no
information beyond axis-118, and it costs the caller two scalars
plus a regime classifier plus a sign helper to find that out.
The diagnostic value is concentrated in the asymmetric regimes,
which the live queue (with openclaw at 0.5293 saturation and the
remaining sources well below 0.26) is already producing.

The takeaway for the cross-source diagnostic stack is that the
f-divergence family is, after axis-140, no longer cumulative in
the naive sense. Axis-140 is the first axis in the family that
ships with explicit handoff conditions to a sibling axis (139,
Neyman, when saturation passes the 0.53 mark and ceiling effects
start to dominate) and explicit equivalence conditions to a parent
axis (118, JSD, when the asymmetry regime is `symmetric`). The
ladder is now self-aware enough to tell the caller which rung to
read. Every prior rung was an unconditional addition; axis-140 is
a conditional one. Whether the family continues in that direction
-- with each new axis shipping its own handoff and equivalence
conditions to existing siblings -- or reverts to unconditional
expansion at axis-141 will determine whether the cross-source
diagnostic surface is consolidating or still growing. The 0.6.383
release notes do not commit either way. The openclaw 0.5293
saturation number, on the live queue, is the first observation
that has forced the question.
