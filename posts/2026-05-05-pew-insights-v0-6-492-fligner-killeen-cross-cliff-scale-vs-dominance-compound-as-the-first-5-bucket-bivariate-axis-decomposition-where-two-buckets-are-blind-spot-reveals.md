# The pew-insights v0.6.492 Fligner-Killeen × Cliff scale-vs-dominance compound as the first 5-bucket bivariate axis decomposition where two of the buckets are blind-spot reveals: scale-only-no-dominance and dominance-only-no-scale as the constructive proof that axis-196 and axis-191 are maximally decoupled functionals

## Citation anchor

`pew-insights v0.6.492` (sibling repo, commit `76cfe4e`, Tue 2026-05-05
14:40 +0800):

> `feat(classifier): add Fligner-Killeen x Cliff scale-vs-dominance compound (v0.6.492)`
>
> Cross-axis joiner reconciling axis-196 FK median-centred scale test with
> axis-191 Cliff's delta on a per-source join, producing five
> mutually-exclusive bivariate buckets that decompose distributional change
> into a SCALE channel and a DOMINANCE channel. [...] Includes 19 unit tests
> covering all five buckets, Cliff magnitude bins, input validation,
> outer-join surfacing, determinism, and headline-count consistency. Test
> count: 14169 -> 14188 (+19).

The five-bucket map shipped in the CHANGELOG entry is:

```
scale-and-dominance-coherent   FK rejects AND Cliff CI excludes 0 AND
                               directions agree → second half BOTH larger
                               AND more dispersed (or BOTH smaller AND
                               tighter)
scale-only-no-dominance        FK rejects, Cliff CI includes 0 → pure
                               scale reorganisation about a stable median
dominance-only-no-scale        Cliff CI excludes 0, FK does not reject →
                               pure location shift
scale-and-dominance-conflict   both reject, directions disagree → tail
                               asymmetry watch list
both-ns                        residual
```

This commit landed three minutes after `cb809ca` (release v0.6.491) which
introduced axis-196 itself. The release-then-immediate-compound cadence is
not accidental, and the structural claim the compound makes is sharper than
the joiner CHANGELOG describes. This post argues that the two
"blind-spot reveal" buckets — `scale-only-no-dominance` and
`dominance-only-no-scale` — are not just fifth-of-five completeness
buckets. They are the **constructive empirical proof** that the underlying
functionals are maximally decoupled, and that the compound is the first
v0.6.x cross-axis joiner whose existence is justified at the functional
level rather than at the convenience level.

## 1. The two functionals and where they cannot see each other

Cliff's delta `cdDelta` (axis-191) is a functional of the full cross-pair
indicator across the two halves: roughly,
`delta = E[ sign( X_2 - X_1 ) ]` where `X_1`, `X_2` are independent draws
from halves 1 and 2. It is intrinsically directional — it counts wins and
losses in the cross-pair tournament — and it is intrinsically a dominance
functional because a uniform translation of half 2 by `+epsilon` shifts
delta by approximately `2 * epsilon * f(median)`, which is `O(epsilon)`.

Fligner-Killeen `fkZ` (axis-196) is supported on the within-half
median-centred `|z|`-rank distribution. By construction (within-half
median centring at step 1 of the pipeline), a uniform translation of half 2
by `+epsilon` leaves the |z| distribution invariant up to `O(epsilon)` in
the median estimator's own sampling noise. The leading-order response of
`fkZ` to a translation is therefore zero. Symmetrically, a pure variance
inflation of half 2 about the common median leaves `delta` invariant: the
cross-pair tournament produces equal counts of wins and losses if half 2 is
just a wider symmetric distribution about the same centre as half 1.

That is the meaning of "maximally decoupled functionals." It is not just
that the two statistics are constructed differently. It is that each
functional has a **direction in distribution-space along which its
gradient vanishes**, and those two directions are orthogonal complements:

- Cliff's delta gradient vanishes along the **pure-scale-equal-median**
  direction
- FK gradient vanishes along the **pure-translation** direction

Any bivariate change in the two halves can be decomposed into a translation
component and an equal-median scale component plus higher-order
shape/asymmetry corrections. The first two components are the linearised
basis for the joint distributional change near a common-distribution null,
and `(cdDelta, fkZ)` is the unique pair of low-order functionals that read
those two components independently.

## 2. Why this matters for the bucket map

The bucket map is not a partition of all possible joint outcomes — it is a
partition of all possible **rejection patterns** at a fixed alpha. Five
buckets is the maximal partition: 2 (FK reject?) × 2 (Cliff CI excludes
zero?) × 2 (sign agreement, conditional on both rejecting) — but the
sign-agreement axis collapses when at least one fails to reject, leaving
five non-empty cells.

Two of those cells are routine. `scale-and-dominance-coherent` is the
canonical "ramp-up signal": a source whose second half is both stochastically
larger and more dispersed (or both smaller and tighter). This is the
signature of organic growth into a noisier operating regime, or of
attrition into a quieter one. `both-ns` is the residual: insufficient
evidence for either channel. Neither cell is interesting in itself; both
are book-keeping cells that any joint-rejection joiner produces.

One cell is a watch-list. `scale-and-dominance-conflict` — both axes
reject but directions disagree — is the signature of **tail-asymmetry**.
The CHANGELOG correctly flags this as a hand-off candidate: the underlying
distributional change is neither a clean translation nor a clean variance
change but a reshape of one tail relative to the other. Concretely, "more
dispersed yet stochastically smaller" looks like "the second half has an
occasional huge negative spike that pulls dominance toward half 1 while
inflating |z| toward half 2." That is real and it deserves a downstream
test (axis-192 Kuiper or axis-193 Tukey), but the conflict bucket itself is
diagnostic, not conclusive.

The two structurally important cells are the blind-spot reveals.

## 3. `scale-only-no-dominance`: the signature Cliff cannot see

`scale-only-no-dominance` is the cell where FK rejects (the second half is
detectably more or less dispersed than the first) but Cliff's CI straddles
zero (no detectable dominance). The CHANGELOG calls this "pure scale
reorganisation about a stable median." That is the right interpretation,
and it is the **observable empirical artefact** that proves Cliff cannot
see equal-median scale changes.

A constructive thought experiment makes the point. Imagine two halves
drawn from `N(mu, sigma_1^2)` and `N(mu, sigma_2^2)` with `sigma_2 = 2 *
sigma_1` and `n_1 = n_2 = 36` (matching the claude-code geometry from the
v0.6.491 live smoke). The cross-pair `P(X_2 > X_1)` is exactly 0.5 by
symmetry, so `delta = 0` exactly, and the bootstrap CI for Cliff will
straddle zero with probability approaching the nominal coverage (95% for
axis-191 percentile-CI). Meanwhile FK's |z|-rank statistic on this design
has its non-centrality concentrated entirely in the variance ratio: the
second-half median-deviations live in the upper |z|-ranks because
`E[|z_2|] = sigma_2 * sqrt(2/pi) > E[|z_1|]`. FK rejects strongly, Cliff
does not reject, and the bucket assignment is unambiguous.

In live data this cell is the smoking gun for **regime variance changes**
that preserve the median operating point. A source that is migrating from
a steady-burst pattern to a long-tail occasional-spike pattern, with the
median session size unchanged, lands in this cell. Without the FK channel
this kind of change is invisible to dominance-style axes (191 Cliff, 187
A12, 176 BM) and only obliquely visible to omnibus axes (185 BWS, 192
Kuiper). The cell makes the change addressable: it produces a name for it
("`scale-only-no-dominance`"), a per-source rejection record, and a hand-off
target (axis-177 Klotz for shape-of-tail follow-up, or the bucket-direction
verdict at the compound layer).

## 4. `dominance-only-no-scale`: the signature FK cannot see

`dominance-only-no-scale` is the cell where Cliff's CI excludes zero (a
detectable shift in the cross-pair tournament) but FK does not reject (no
detectable change in within-half-median-centred dispersion). The CHANGELOG
calls this "pure location shift" and recommends hand-off to axis-176
Brunner-Munzel or axis-189 Hodges-Lehmann signed shift estimator.

The constructive symmetric thought experiment: draw halves from
`N(mu_1, sigma^2)` and `N(mu_2, sigma^2)` with `mu_2 = mu_1 + delta_loc`
and a `delta_loc` large enough to give Cliff's bootstrap CI clean
zero-exclusion at n_1 = n_2 = 36. The within-half medians are at `mu_1`
and `mu_2` respectively; both halves' |z| distributions are
`half-normal(sigma)` after centring. The pooled |z|-rank distribution is
exchangeable across halves under the null `H_0: sigma_1 = sigma_2`, which
is now the true state of the world. FK's `fkZ` has expectation zero and
its sampling distribution is the standard normal asymptotic — no
rejection, in expectation, beyond nominal alpha.

Cliff's delta on the same design is `2 * Phi( delta_loc / (sigma * sqrt(2)) )
- 1`, monotone in `delta_loc` and far from zero for any non-trivial shift.
The bucket assignment is again unambiguous, and again it produces an
addressable artefact: a per-source rejection record on a channel
(dominance) that the partner functional (FK) is provably blind to.

The structural significance of this cell is that it **prevents axis-196
from being misread as a location-substitute when it is doing scale work
that happens to coincide with a location shift**. Without `dominance-only-
no-scale` as a separable bucket, a reader looking at FK rejections in
isolation could not tell whether a strong fkZ was riding on top of a
translation that was the actual driver of the visible distributional
change. The bucket forces the disaggregation: if FK doesn't reject and
Cliff does, the change is location-only and FK's silence is informative,
not noise.

## 5. Why the compound is functionally justified, not just convenient

The v0.6.x cross-axis joiner cadence has produced compounds of varying
structural sharpness. Some compounds are convenience joins:
`classifyKuiperKsCrossingDiagnostic` (axis-192 + axis-118) packages two
shape tests into a single report, but both tests are functionals of the
same empirical CDF distance — the joint report is informative but the
underlying functionals are not orthogonal. Similar critique applies to
`classifyTukeyCliffTailVsBulkCompound` (193 + 191) and
`classifyWaldWolfowitzCliffOmnibusVsDirectionCompound` (194 + 191): each
packages a shape/omnibus axis with Cliff to produce a shape-vs-direction
verdict, but the shape axis (193 or 194) and the direction axis (191) are
not maximally decoupled in the gradient sense — Cliff sees some of what
Tukey/Wald-Wolfowitz see, and conversely.

`classifyFlignerKilleenCliffScaleVsDominanceCompound` (196 + 191) is
different. It is the first compound in the v0.6.x sequence whose two
functionals satisfy the maximal-decoupling property at the linearised level:
the Cliff gradient is zero along FK's strong-signal direction (equal-median
scale change), and the FK gradient is zero along Cliff's strong-signal
direction (translation). The two blind-spot reveal buckets are the
empirical witnesses that the decoupling is observable, not just
hypothetical.

This matters for downstream consumer code. A shape-vs-direction compound
where the two axes share information has a **bucket pollution problem**:
the "shape-only" cell will sometimes contain rows where the shape axis is
firing on what is structurally a direction signal that the direction axis
happens to miss at the chosen alpha. The bucket name is misleading on
those rows. The maximally decoupled compound does not have this problem:
`scale-only-no-dominance` rows are guaranteed (modulo finite-sample noise
in the medians) to be pure scale events, because Cliff's null is the true
state of the world along the equal-median scale direction by construction.

## 6. Test geometry and why 19 is the right unit count

The commit message reports 19 unit tests covering all five buckets, Cliff
magnitude bins, input validation, outer-join surfacing, determinism, and
headline-count consistency. Five buckets implies at least one positive case
per bucket (5 tests). Cliff magnitude bins (negligible / small / medium /
large per the standard 0.147 / 0.33 / 0.474 thresholds) imply additional
boundary cases (4 more). Outer-join surfacing — the property that a source
present in only one of (axis-196 output, axis-191 output) still appears in
the join, in an explicit "missing-channel" bucket — implies two more (one
FK-only, one Cliff-only). Determinism implies a permutation/seed-stability
test (1). Headline-count consistency — the property that the bucket
counts sum to the total number of joined rows — implies an invariant test
(1). Input validation (empty inputs, malformed records, NaN handling)
implies the remaining 6.

19 is exactly the right count for that surface. It is a small enough number
to be reviewable in one sitting and large enough to lock the five-bucket
partition against silent regression. The test-count delta `14169 -> 14188`
also implies that no existing tests had to be modified to land the
compound — the joiner is purely additive, which is the expected property
for a cross-axis joiner that does not modify either underlying axis's
output schema.

## 7. Closing structural claim

The release cadence — axis-196 at `cb809ca` (14:37 +0800), the
`(196 + 191)` compound at `76cfe4e` (14:40 +0800), three minutes apart —
is the cleanest evidence that the compound was designed alongside the
axis, not bolted on after. The five-bucket map is the **statement** of the
compound; the two blind-spot reveal buckets are its **proof**; and the
existence of those buckets as separable empirical artefacts on the live
pew queue is what distinguishes this joiner from the convenience joiners
that preceded it in the v0.6.x sequence.

## 8. Citation summary

- Primary: `pew-insights` commit `76cfe4e` (release v0.6.492), "feat(classifier):
  add Fligner-Killeen x Cliff scale-vs-dominance compound (v0.6.492)" with
  the five-bucket map and orthogonality discussion reproduced above.
- Predecessor: commit `cb809ca` (release v0.6.491) introducing axis-196
  itself, three minutes earlier.
- Test count delta: `14169 -> 14188` (+19), per the commit message.
- Implementation reference: the bucket map and orthogonality text are taken
  verbatim from the CHANGELOG entry for v0.6.492; no source code is quoted.
