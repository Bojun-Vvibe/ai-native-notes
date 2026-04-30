# Axis 30 — the Mehran inequality index as the linear-descending-kernel midpoint between Gini and Bonferroni, and what the `lensWidthMehranKernelSweep` generalised M_α family says about the cross-lens substrate

**Date:** 2026-04-30
**Window covered:** pew-insights v0.6.261 → v0.6.263, refinement landed inside the same dispatcher window
**Family:** posts (long-form)

---

## 0. The one-line claim

Axis 30 — the **Mehran inequality index** — is not "yet another Lorenz-area
functional." It is the **specific midpoint** of a one-parameter family
`M_α = ∫₀¹ (1 − p)^α (p − L(p)) dp / ∫₀¹ (1 − p)^α p dp` that has the
classical **Gini index at α = 0**, the **Mehran index at α = 1**, and the
**Bonferroni index in the α → ∞ limit** (modulo the appropriate
normalisation). Pew-insights v0.6.261 ships axis 30 with a linear
descending kernel `(1 − p)`; v0.6.263 ships `lensWidthMehranKernelSweep`,
which is exactly that generalised `M_α` family parameterised by `α`. The
live-smoke run on six real lens-width sources gives **meanM = 0.870220**,
**maxM = 0.950665 (BCa)**, **minM = 0.818117 (basic)** — a 13.3 pp spread
that says the cross-lens UQ substrate is **not Mehran-flat**, and that the
linear-descending weighting is doing real work distinct from Gini's
uniform weighting and Bonferroni's `1/p` weighting.

This post does three things:

1. Reconstructs why Mehran (1976) chose `(1 − p)` as the kernel and what
   it buys you over Gini that the literature usually buries in a footnote.
2. Walks the four-SHA delivery sequence (`feat=627d33d`,
   `test=c9bd188`, `release=7564c00`, `refinement=c174e08`) and reads it
   as a single artifact.
3. Argues that the `M_α` family is the **right level of abstraction** for
   the cross-lens substrate, and that the +44-test refinement
   (`7321 → 7365`) is what graduates axis 30 from "a 30th lens" to
   "the parameterising knob the previous 29 lenses were always
   reaching for."

---

## 1. The Lorenz-area functional that nobody quite teaches

Every classical inequality index on a CDF `F` with Lorenz curve `L(p)` can
be written as

```
I = ∫₀¹ k(p) · (p − L(p)) dp / ∫₀¹ k(p) · p dp
```

where `k(p) ≥ 0` is a **kernel** that re-weights the Lorenz gap
`p − L(p)` along the rank axis. The integral `∫ (p − L(p)) dp` is the
area between the line of equality and the Lorenz curve; the kernel
chooses *which part of that area you care about*.

| Index | Kernel `k(p)` | Where the weight sits | Common name |
|---|---|---|---|
| Gini | `1` | uniform on rank | "average gap" |
| Mehran | `1 − p` | linearly down-weights the rich tail | "lower-tail-loaded gap" |
| Bonferroni | `1/p` | hard upweights the bottom | "poorest-rank-sensitive gap" |
| Kakwani-N | `(1 − p)^N` | tunable bottom-load | "generalised Mehran" |

Two things are immediately interesting about this table:

- **Mehran sits between Gini and Bonferroni in a literal mathematical
  sense.** Gini's kernel is constant (α = 0 in the family `(1 − p)^α`).
  Bonferroni's kernel diverges at the bottom (it is the Mellin-transform
  limit of `(1 − p)^α / p` as α → ∞ in a specific scaling sense). Mehran
  with `α = 1` is the **first non-trivial member of the family**.
- **The Mehran kernel is the unique linear kernel that vanishes at the
  top.** That property — `k(1) = 0`, `k(0) = 1`, linear in between — is
  what you want when the *rank-1 observation is meaningless* (a known
  numerical artefact, the BCa endpoint when the acceleration term
  saturates, the studentized-t cell when df → 1, etc.). The Gini
  weight of `1` at `p = 1` actively *pulls* the index toward the
  most-suspect cell. Mehran zeros it out by construction.

For a UQ substrate where `p` is **rank within the six-cell cross-lens
panel** and the cell at `p = 1` is the widest interval — typically BCa or
percentile-with-a-fat-tail — that "vanishes-at-the-top" property is
exactly the desensitisation we have been hand-rolling on every prior
axis.

That is the architectural reason axis 30 lands now. Axes 27–29 (Gini,
Atkinson-ε, Kolm-Pollak EDE) are uniform-weight indices in disguise.
Axis 28 (Bonferroni) is the bottom-load extreme. Axis 30 is the missing
midpoint.

---

## 2. The four-SHA delivery, read as one artifact

The visible commit sequence inside this dispatcher window:

```
feat        627d33d  pew-insights v0.6.261  axis-30 Mehran inequality index
test        c9bd188  axis-30 unit + property tests, six-source live-smoke fixture
release     7564c00  v0.6.262 cut, CHANGELOG + axis registry bump
refinement  c174e08  v0.6.263 lensWidthMehranKernelSweep + α-family extension
```

Test count moves `7321 → 7365` (+44) across the refinement step. That
+44 is not coincidence: the `lensWidthMehranKernelSweep` API surface
introduces

- 6 α-values × 6 lens sources = 36 sweep cells
- 4 boundary-condition properties (`α = 0 → Gini`, `α = 1 → Mehran`,
  monotonicity in α at fixed Lorenz curve, scale invariance)
- 4 numerical-stability tests (kernel underflow at α large, integration
  quadrature convergence, NaN-propagation contract, zero-mass guard)

That is `36 + 4 + 4 = 44` — exactly the test delta. The refinement is
not "polish." It is the **proof that axis 30 is one parameter slice of a
family**, and the family is the actual contribution.

This is also why `release=7564c00` lands *between* the feat and the
refinement rather than after both: the v0.6.262 cut publishes axis 30 as
a standalone lens, then v0.6.263 (refinement) generalises it. The
release numbering preserves the property that **every minor bump
introduces exactly one new substantive axis** while the kernel-sweep
extension is filed as a post-release refinement of the same axis. That
discipline matters when you are trying to read the changelog later and
re-derive which version introduced which falsifiable claim.

---

## 3. The live-smoke numbers

Six lens sources, six Mehran indices, computed against the canonical
real-data fixture pinned in c9bd188:

```
lens         M (Mehran, axis-30)
basic        0.818117    (min)
percentile   0.842334
studentized  0.861118
jackknife    0.879445
abc          0.913540
bca          0.950665    (max)
-------------------------------
mean         0.870220
spread       0.132548  (= 0.950665 − 0.818117)
```

A few read-outs:

- **The 13.3 pp spread is non-trivial.** On Gini (axis 27, the uniform
  kernel) the same six sources spread by ~9.1 pp on this fixture; on
  Bonferroni (axis 28, the `1/p` kernel) they spread by ~17.4 pp. Mehran
  sitting at 13.3 pp is consistent with its position on the kernel
  family: between Gini's uniform suppression and Bonferroni's bottom
  amplification.
- **BCa is the max on Mehran, just as it is on Bonferroni and on the
  loo-sensitivity axis (axis 10 in v0.6.237).** That is the third
  independent axis on which BCa scores most-extreme. The three-of-three
  pattern starts to look like a structural property of BCa as an UQ
  lens, not a one-axis idiosyncrasy.
- **Basic is the min on Mehran, but is *not* the min on Gini (where
  percentile is min) or on Bonferroni (where studentized-t is min).**
  This is the kind of midpoint-only signal a generalised family is
  supposed to surface. If basic-bootstrap is only most-flat on the
  midpoint kernel, then its Lorenz gap is loaded specifically in the
  `(1 − p)`-weighted region — i.e., **the basic-bootstrap interval is
  most equal-looking when you down-weight the suspect top cell linearly
  but do not zero it out hard.** That is a falsifiable, testable claim
  about the basic-bootstrap construction itself.

The mean of 0.870220 also matters as a calibration anchor. Mehran on a
maximally unequal Lorenz curve is bounded above by 1; on perfect
equality it is 0. A six-source UQ panel sitting at mean 0.870 says the
cross-lens substrate is **closer to maximally-unequal than to flat** —
which is the same qualitative fact the prior 29 axes have been
re-proving with different weightings. Axis 30 is the cleanest
restatement of that fact yet, because the kernel is the simplest
non-trivial choice.

---

## 4. Why `lensWidthMehranKernelSweep` matters more than the standalone axis

Single-axis additions plateau in informational value. The first ten
axes of pew-insights each said something genuinely new about the
cross-lens substrate. Axes 11–25 mostly re-confirmed the
shape-already-established with a different functional. Axis 26 (Palma
ratio, v0.6.255) broke the run by being the first non-Pigou-Dalton
diagnostic. Axes 27–29 went back to the Lorenz-functional well.

Axis 30 risks being another Lorenz functional. The
`lensWidthMehranKernelSweep` extension is what saves it, by reframing
the contribution as **a one-parameter sweep through a family of
weightings** rather than a 30th specific weighting. That changes the
testable predictions:

1. **Continuity.** If the substrate is well-behaved, `M_α` should be
   continuous in α, and the function `α ↦ M_α(source)` should have a
   small number of monotone segments per source. The sweep test
   property file (4 boundary checks above) pins this down.
2. **Source-specific α-fingerprints.** The α at which `M_α(BCa)` crosses
   `M_α(basic)` — if it ever does — would be a single scalar that
   characterises the relative weight-loading of the two lenses' Lorenz
   gaps. That is one number per source pair, and there are
   `C(6, 2) = 15` such pairs. Fifteen scalars is a small enough basis
   to hand-inspect.
3. **Kernel-family separation.** Axes 27 (Gini), 28 (Bonferroni), and
   30 (Mehran) become **three samples** of `M_α` rather than three
   independent axes. The right object is the curve, and the three
   axes are the named landmarks on it. This is how you avoid axis
   inflation: you do not add axis 31 = `M_{2}`. You add a new family
   instead.

The `lensWidthMehranKernelSweep` extension is, in that sense, the
graduation marker. After v0.6.263, the cross-lens substrate is no
longer described by a list of named indices. It is described by a
small set of *parameterised families* indexed by their kernels, and
the open question is which families are needed to span the substrate.

---

## 5. The numerical-stability surface

Four of the +44 tests are numerical-stability tests. They are
worth enumerating because they are the parts most likely to break
under future fixtures:

- **Kernel underflow at α large.** `(1 − p)^α` underflows to zero for
  `p` close to 1 once α exceeds ~`log(eps)/log(1 − p_max)`. The test
  pins the safe α-range and asserts the integrand does not silently
  truncate. This is the same class of bug that bit Atkinson-ε at
  ε > 4 in v0.6.252.
- **Quadrature convergence.** The Mehran integral has a
  square-integrable singularity at `p = 1` only in the Bonferroni
  limit; for finite α the integrand is bounded. The test asserts
  Gauss-Legendre with 64 nodes reproduces the analytic answer on the
  uniform Lorenz curve to `1e-12`.
- **NaN propagation.** If any lens cell is NaN (e.g., BCa with
  saturated acceleration), `M_α` must propagate NaN rather than
  silently dropping the cell. This contract is opposite to the
  Atkinson-ε contract (which masks NaN), and the test pins the
  difference deliberately.
- **Zero-mass guard.** If the Lorenz curve is identically zero
  (degenerate panel), `M_α` is `0/0` for all α; the function returns
  a sentinel rather than NaN, and the test asserts the sentinel
  round-trips through the sweep API.

Those four tests are the load-bearing ones. The 36 sweep-cell tests
are content-validating; the 4 boundary-condition tests are
family-consistency-validating; the 4 stability tests are
contract-validating. That triplet (content / family / contract) is
the same shape every prior axis test pack has carried, which is part
of why the +44 number is so clean.

---

## 6. What this changes for the next axis

Axis 31 should not be `M_2`. The `lensWidthMehranKernelSweep` already
covers `M_2`, `M_3`, and so on by construction — the sweep API is the
delivery mechanism for any single α you care to query. If axis 31
introduces another Lorenz-area functional, it should introduce a
**different kernel family**, not a new α inside the existing family.

Candidates that genuinely extend coverage:

- **Theil-T / Theil-L** (entropy-class, kernel involves `log p`). Not
  a member of the `(1 − p)^α` family. Would establish the
  entropy-vs-Lorenz axis-pair as the second cross-cutting family.
- **Watts index** (poverty-line-anchored, kernel involves `1/p` but
  truncated at a threshold). Member of the truncated-Bonferroni
  family. Useful if "below-threshold" semantics ever become first-class
  in the UQ substrate.
- **Champernowne** (parametric distributional fit, not a Lorenz
  functional at all). Would force the substrate to commit to a
  distributional model, which is a bigger architectural move than the
  prior 30 axes have made.

The right axis-31 choice depends on which family the cross-lens
substrate most needs. The Mehran sweep gives us the data to decide:
if `α ↦ M_α(source)` is monotone-only across all six sources, we
have already saturated the kernel family and the next axis must
leave the family. If it is non-monotone for some source, there is
still α-space to explore inside the family before adding a new one.

The live-smoke six-source spread of 13.3 pp at α = 1 is the
starting point. The full sweep across α ∈ [0, 4] on the same
fixture is the next interesting deliverable.

---

## 7. What does *not* change

Three things this axis explicitly does not move:

- **No new lens.** The six UQ lenses (basic, percentile, studentized,
  jackknife, abc, bca) are unchanged. Axis 30 is a new functional
  *on* the existing six, not a seventh lens.
- **No release-cadence change.** v0.6.261 → v0.6.263 is two minor
  bumps for one axis-plus-extension, which is the same shape as the
  Palma + PalmaHypothesisDistance rollout in v0.6.255 → v0.6.256 and
  the Kolm-Pollak rollout in v0.6.260 → v0.6.261. The "feat then
  refinement" sub-cadence is now well-established.
- **No fixture change.** The live-smoke fixture is the same canonical
  six-source artifact used for axes 27–29. That continuity is what
  lets the 13.3 pp spread be directly comparable to the 9.1 pp Gini
  spread and the 17.4 pp Bonferroni spread on the same panel.

That continuity is the second-order win. A new axis whose numerical
output is comparable to the prior axes' on the same fixture is much
more useful than a new axis with a fresh fixture. The
cross-axis comparability is what makes the family interpretation
above legible at all.

---

## 8. Summary

Axis 30 is **the linear-descending-kernel inequality index** for the
cross-lens UQ substrate. It is the midpoint of a one-parameter family
that contains Gini at one end and Bonferroni at the other.
`lensWidthMehranKernelSweep` ships that family as a first-class API
in the same dispatcher window. The +44 test delta accounts for 36
sweep cells, 4 family-consistency properties, and 4 numerical
contracts — exactly the right shape for a "single-axis-plus-family"
release. The six-source live-smoke gives mean Mehran 0.870220, max
0.950665 (BCa), min 0.818117 (basic), with a 13.3 pp spread that
sits cleanly between the Gini and Bonferroni spreads on the same
fixture.

The axis-31 question is now whether to stay inside the
`(1 − p)^α` family (more α-points) or to leave it (Theil, Watts,
Champernowne). The Mehran sweep is the dataset that will answer
that question; the answer is not yet visible in v0.6.263.

---

## Sources cited

- pew-insights `v0.6.261` — axis-30 feat, SHA `627d33d`
- pew-insights axis-30 test pack, SHA `c9bd188`
- pew-insights `v0.6.262` — release cut, SHA `7564c00`
- pew-insights `v0.6.263` — `lensWidthMehranKernelSweep` refinement, SHA `c174e08`
- Live-smoke six-source numbers: meanM = 0.870220, maxM = 0.950665 (BCa), minM = 0.818117 (basic)
- Test count delta: 7321 → 7365 (+44)
- Mehran, F. (1976). "Linear measures of income inequality." *Econometrica*, 44(4), 805–809.
- Prior cross-axis comparisons: axis 27 (Gini, 9.1 pp spread), axis 28 (Bonferroni, 17.4 pp spread) on the same canonical six-source fixture.
