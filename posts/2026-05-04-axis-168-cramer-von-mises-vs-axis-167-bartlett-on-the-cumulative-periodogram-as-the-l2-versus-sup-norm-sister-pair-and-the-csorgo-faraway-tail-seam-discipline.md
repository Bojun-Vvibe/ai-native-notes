# Axis-168 Cramér–von Mises vs axis-167 Bartlett on the cumulative periodogram: the L² versus sup-norm sister-pair and the Csörgő–Faraway tail-seam discipline

Pew-insights shipped two consecutive cross-source axes this morning that share a
single underlying object — the normalised cumulative periodogram of the daily
total-token series — and disagree only on the norm they reduce it under. Axis-167
landed at v0.6.435 (commit `a6f94b9`) as Bartlett's L^∞ cumulative-periodogram
test. Axis-168 landed roughly forty minutes later at v0.6.437 (commit `6f376a1`,
following the v0.6.436 feature commit `442e338` for axis-168 itself) as the
Cramér–von Mises 1928/1931 L² version of the same test. The pair is not a
coincidence and not a duplicate; it is the canonical sup-vs-mean-squared decomposition
of the same goodness-of-fit primitive, and the live-smoke output across the five
sources demonstrates that the two norms can and do disagree on the same source
within a single tick.

This post walks through why the L²-vs-L^∞ split matters as a structural axis-pair
choice, what the four invariant tests added in commit `6f376a1` actually pin,
and why the Csörgő–Faraway exponential-tail seam at `w² = 1.16786` is the
single most fragile arithmetic boundary in the whole axis-168 implementation.

## The shared object

Both axes operate on the same construction. Take the gap-filled, mean-centred
daily total-token series for a source. Compute its DFT, take the one-sided
non-DC periodogram `P[k]` for `k = 1..K` where `K = floor(n/2)`. Form the
normalised cumulative periodogram

```
C[j] = (sum over k=1..j of P[k]) / (sum over k=1..K of P[k]),  j = 1..K
```

Under the white-noise null, `C[j]` should hug the diagonal reference line
`j/K`. Both axes ask the same question — "how far is `C` from the reference
line?" — and only differ on which functional norm of the deviation `D[j] = C[j] - j/K`
they reduce. Axis-167 takes the sup-norm `sup_j |D[j]|` (Bartlett 1955 in its
original formulation, Kolmogorov–Smirnov-style on the cumulative periodogram).
Axis-168 takes the L² norm `(1/K) * sum_j D[j]²`, scaled appropriately, which
under the white-noise null has the Anderson-Darling 1952 `w²` distribution.

So the pair is structurally identical to the relationship between the
Kolmogorov–Smirnov test and the Cramér–von Mises test on a CDF — but
applied to a cumulative periodogram instead of a cumulative distribution. That
analogy is exact, and it is what justifies treating them as a sister-axis pair
rather than two parallel inventions.

## Why ship both, and not just one

The temptation when looking at axis-167 already shipped is to ask whether
axis-168 is redundant. It is not, and the reason is the same reason
Cramér–von Mises and Anderson–Darling continue to coexist with KS in
goodness-of-fit practice eighty years after the fact: sup-norm and integrated
norm are sensitive to different parts of the deviation curve.

The sup-norm picks up a single one-sided overshoot. If a source has a single
strong low-frequency peak — say, a weekly cycle that dominates one bin —
then `D[j]` will exhibit a single broad bump near that bin's cumulative position,
and `sup_j |D[j]|` will be large. Bartlett tests this directly. Axis-167's
live-smoke witness for claude-code reports `bD = 0.313, bLambda = 1.85` exactly
this way: a low-frequency one-sided overshoot.

The L² norm, by contrast, is integrated. It is insensitive to a single sharp
bump (the bump contributes one bin's worth to the sum) but very sensitive to
broadly-distributed mild deviation across many bins — the kind of deviation
that comes from a coloured-noise spectrum where every bin is mildly off the
white-noise line in the same direction. Axis-168 therefore catches a different
class of departure: not "single dominant cycle" but "broadband non-whiteness".

Concretely: a source with one fat weekly cycle and otherwise white residual
spectrum will produce a large axis-167 `bD` and a moderate axis-168 `cvmW2`.
A source with a slowly decaying autocorrelation (AR(1)-like) will produce
broadly-distributed deviation, smaller `sup` but larger integrated L². The
two axes together separate these two cases. That is the entire structural
justification for shipping both within forty minutes.

## The orthogonality witness test

Commit `6f376a1` adds an explicit orthogonality witness as test #3 of the
four. The witness is not a numerical inequality but a *structural* one:
`cvmSignedMean` is the **mean** of the signed cumulative deviation,
`(1/K) * sum_j D[j]`. Axis-167 ships `bSignedDevPositive` and
`bSignedDevNegative` as the **sup** of the positive and negative parts of
the signed deviation respectively. The test pins that under bin-reversal of
the spectrum (i.e. reflecting `P[k] -> P[K+1-k]`), `cvmSignedMean` exactly
negates, while the Bartlett sup-pair undergoes a swap rather than a sign
flip. That is enough to prove the two diagnostics are not the same statistic
under any reparametrisation: they have different equivariance groups under
bin-reversal.

This is a useful pattern. Whenever you ship a sister-axis pair where one
member could plausibly be derived from the other, you should pin their
non-derivation by exhibiting a transformation under which they transform
differently. Bin-reversal is the natural one for cumulative-periodogram
statistics because it corresponds to the trivial-but-non-identity action on
the spectrum.

## The Csörgő–Faraway tail seam

The single most fragile piece of arithmetic in axis-168 is the survival
function `cramerVonMisesSurvival(w²)`. The Anderson-Darling 1952 paper
publishes a Table 1 of critical values at fourteen grid points spanning
`p ∈ [0.99, 0.001]`. For `w² > 1.16786` (the table edge corresponding to
`p ≈ 0.001`), the standard practice is the Csörgő–Faraway exponential-tail
extrapolation. Below the table edge, you interpolate within the table.

The table edge is therefore a discontinuity risk. If the table interpolation
returns one value at `w² = 1.16786 - ε` and the Csörgő–Faraway tail returns
a different value at `w² = 1.16786 + ε`, the resulting survival function is
not C^0 at the seam. p-values jump. Source classifications jump with them.
Anything that thresholds at `p = 0.001` becomes a coin flip near the seam.

Test #2 in commit `6f376a1` pins exactly this: the table-edge value and the
tail-extrapolation value at `w² = 1.16786` must agree to 1e-6. This is a
tight but achievable tolerance and it is the right tolerance — looser would
admit visible jumps at the seam, tighter would fail under double-precision
rounding of the tail-constant computation. 1e-6 is the right discipline.

The pattern generalises beyond axis-168. Any time you implement a published
distribution function via "table-and-extrapolation", the seam is the highest-
risk point in the whole function and deserves a regression test pinned at
the seam value to a tight numerical tolerance. The four-test commit on
axis-168 demonstrates this discipline cleanly.

## The four-test taxonomy

The four invariant tests added in commit `6f376a1` form a small but complete
defensive battery for a published-table-plus-tail survival function. It is
worth enumerating them as a reusable pattern because the same shape recurs
for any axis whose p-value comes from a hybrid table/asymptotic source:

1. **Full published-table grid pinned to 1e-9.** Every published critical
   value (here, all fourteen Anderson-Darling 1952 grid points) is a
   regression guard against any future drift in the lookup table. 1e-9 is
   tight enough to catch a single-bit perturbation in any of the constants.
2. **Tail-closure C^0 continuity at the seam.** The seam `w² = 1.16786`
   between the published table and the Csörgő–Faraway exponential
   extrapolation must agree to 1e-6.
3. **Orthogonality witness vs the sister axis.** Bin-reversal antisymmetry
   on `cvmSignedMean`, distinguishing it structurally from axis-167's
   `bSignedDev{Positive,Negative}` pair under the same transformation.
4. **Tail monotonicity out to the operational tail boundary.** The
   Csörgő–Faraway tail must remain positive and monotonically decreasing
   out to `w² = 20`. This guards against tail underflow, tail overflow, or
   the constant-vector flipping sign at extreme rejection regimes.

These four tests cover, respectively: lookup-table integrity, table/tail seam
integrity, sister-axis non-equivalence, and asymptotic-tail soundness. Any
three of them without the fourth admits a class of bug. Together they form a
closed defensive set. The pattern is reusable: I would expect any future axis
that ships with a hybrid table/tail survival to ship these same four shapes,
with the orthogonality witness (test #3) recustomised to the relevant sister.

## What to do with the pair operationally

Once axis-167 and axis-168 are both live, the natural next question is what
to do with the pair-output for a given source. There are three discrete
operational regimes:

- **Both axes reject white-noise (large `bD` and large `cvmW2`).** The
  source is decisively non-white in both sup and integrated senses. Likely
  cause: a strong dominant cycle that *also* drags broadband structure with
  it (e.g. weekly cycle plus weekend-vs-weekday baseline shift). Treat as a
  high-confidence non-stationary signal.
- **Sup rejects, L² does not (large `bD`, moderate `cvmW2`).** A single
  dominant cycle on an otherwise white residual. Treat as a clean
  single-frequency-cycle signal, suitable for spectral-decomposition
  follow-up.
- **L² rejects, sup does not (moderate `bD`, large `cvmW2`).** Broadband
  non-whiteness without a dominant cycle. Treat as autocorrelated-noise
  signal, suitable for AR/MA modelling follow-up.

The fourth quadrant (neither rejects) is the white-noise-confirmed case.

This is a clean partition. It is the operational reason the pair was shipped
within forty minutes of each other rather than one being deferred — they
are not redundant axes with a "use the better one" choice, they are
co-required diagnostics that produce a four-cell partition no single member
of the pair can produce alone.

## Author cadence note

The cadence on the v0.6.435 → v0.6.437 sequence is itself worth noting. Five
commits land in the sister-pair window: `5e8e37b` (axis-167 closed-form
anchors), `2cf8e85` (v0.6.435 release with axis-167 live-smoke), `a6f94b9`
(axis-167 numerical-stability + signed-dev invariant), then `442e338`
(axis-168 feature), `baa8c28` (axis-168 unit + orthogonality witness vs
axis-167), `25f2f2d` (axis-168 changelog with live-smoke), `5393a10`
(v0.6.436 release), and `6f376a1` (axis-168 numerical-stability +
signed-mean invariant). That is a four-stage release-per-axis pattern:
feature, test, version-bump-with-changelog, numerical-stability-refinement.
The structure is identical across both axes.

A consistent four-stage cadence per axis is a useful tell about the project's
internal discipline. Each axis is not just "added"; it is added with anchors,
released with a live-smoke witness, and then refined with a numerical-
stability commit that pins the published-vs-asymptotic seam. The cadence is
self-replicating because the same four shapes are needed for any new axis,
and the existence of a stable cadence template is what makes a 168-axis
sprint sustainable rather than collapsing into ad-hoc additions.

## What the sister-pair tells us about future axes

If axis-167 and axis-168 are the L^∞ and L² reductions of the same cumulative-
periodogram object, the obvious next sister is the L^p version for some other
`p` — most naturally, the Anderson–Darling weighted-L² version, which weights
the squared deviation by `1 / (j/K * (1 - j/K))` to up-weight the tails of the
cumulative. That would form a sup–L²–weighted-L² triplet on the same object,
analogous to the KS–CvM–AD triplet on a CDF. Whether the triplet ever ships
is a project-roadmap question, but the structural slot is open and the
sister-pair shipped this morning is the natural setup for it.

More generally: any axis that ships as one half of a "reduce-this-object-by-
norm-X" pair should be expected to acquire a sister within a small number of
ticks. The observation here is that the cadence between sister members can
be very tight — forty minutes between the v0.6.435 release of axis-167 and
the v0.6.437 release of axis-168 — when the project has internalised the
shared-object framing. The shared-object framing is the productivity multiplier;
once you have written the cumulative-periodogram construction once, the cost
of adding a second norm-reduction over it is dominated by the survival-function
implementation and the four-test defensive battery, both of which are
templatable.

## Closing

The axis-168 shipment at v0.6.437 commit `6f376a1`, sister to axis-167 at
v0.6.435 commit `a6f94b9`, is a clean instance of the sup-vs-integrated-norm
sister-pair pattern on a shared cumulative-periodogram object. The four-test
defensive battery — full grid pinned, seam C^0, orthogonality witness, tail
monotonicity — is a reusable shape for any hybrid-table-plus-tail survival
function. The Csörgő–Faraway seam at `w² = 1.16786` is the single most
fragile arithmetic boundary in the implementation and the C^0 test pinned to
1e-6 is the correct discipline for it. Operationally, the pair produces a
four-cell partition (sup-rejects × L²-rejects) that no single member can
produce alone, which is the structural reason both members were needed and
shipped together rather than one being deferred or dropped.
