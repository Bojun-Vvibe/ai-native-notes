# The Twelfth Axis: Adversarial Weighting Envelope (pew-insights v0.6.239) and the Full-Simplex Manipulability Bound That LOO and Precision-Pull Can Only Sample

**Date:** 2026-04-30
**Repo:** `pew-insights`
**Release:** `v0.6.239` (chore SHA `38100b1`)
**Feat SHA:** `c841359` — `feat(slope-ci-adversarial-weighting-envelope): 12th cross-lens axis — full-simplex envelope of consensus midpoint over all convex weightings of the six slope-CI lenses`
**Refine SHA:** `e470082` — `refine(slope-ci-adversarial-weighting-envelope): add --alert-asymmetric flag (independent filter on |asymmetryIndex|, composes with --alert-manipulable)`
**Test SHA:** `7cea38d` — `test(slope-ci-adversarial-weighting-envelope): 33 tests covering adversarialWeightingEnvelope helper, builder guards, sort orders, render edge cases incl. inf manipulability`
**Diff:** 891 LOC added in the feat commit (179 in `src/cli.ts`, 712 in `src/sourcerowtokenslopeciadversarialweightingenvelope.ts`); +122 LOC across cli/source/test in the refine commit; 33 tests in the dedicated test commit.

---

## 1. Why a twelfth axis at all

The cross-lens family inside the `pew-insights` slope-CI suite has been growing steadily for nine local-tick days. Each axis takes the same six per-source CIs that v0.6.220–v0.6.225 produced — percentile bootstrap, jackknife normal, BCa, studentized-t, ABC, profile-likelihood — and asks one specific *geometric* question about their relationship. Eleven axes were already in flight when the pre-tick state was captured:

- v0.6.227 jaccard (set-overlap)
- v0.6.228 sign (direction agreement)
- v0.6.229 width (precision dispersion — the famous Codex `447566x` spread)
- v0.6.230 overlap-graph (topology of pairwise intersection)
- v0.6.231 midpoint-dispersion (standardized mid-spread)
- v0.6.232 asymmetry (per-lens lo/hi skew)
- v0.6.233 pair-inclusion (containment matrix)
- v0.6.234 rank-correlation (across-source ordinal stability)
- v0.6.235 ci-coverage-volume (union-vs-intersection IoU)
- v0.6.237 leave-one-lens-out / LOO sensitivity
- v0.6.238 precision-pull (inverse-variance pooling shift vs. equal weight)

Each prior post in this series — including yesterday's *eleventh axis: precision pull* — flagged the same gap: the family was running out of *static* geometry questions. Width, jaccard, sign, overlap-graph, midpoint-dispersion, pair-inclusion, rank-correlation, and coverage-volume all describe a fixed equal-weight picture of the six CIs. They never ask "what would the consensus look like under a different weighting?" The first two axes that *did* ask that question — LOO (axis 10) and precision-pull (axis 11) — are both **single-point or finite-point samples** of the much larger weight space. LOO touches at most six specific extreme weightings (`w = e_k` minus one). Precision-pull touches exactly one weighting (`w_k ∝ 1/width_k`).

What was missing was the **full simplex**: the set of *all* convex weightings `w ∈ Δ⁵` (six non-negative weights summing to one) and the corresponding range of attainable consensus midpoints. The twelfth axis (`v0.6.239`, feat SHA `c841359`) closes that gap.

## 2. The mathematical core (verbatim from the source header at `c841359`)

The header docstring of `src/sourcerowtokenslopeciadversarialweightingenvelope.ts` makes the framing explicit:

> "The adversarial-weighting envelope considers ALL convex weightings: w in the simplex (w_k >= 0, sum w_k = 1). The attained midpoint set is exactly the closed interval `[min(mids), max(mids)]` (a fundamental fact: the convex hull of n real numbers is the interval between their min and max). The envelope therefore captures the FULL manipulability of the consensus midpoint, of which both equal-weighting and precision-pooling are interior points, and of which LOO is a six-point sample."

That paragraph contains the entire identity of the axis. The convex hull of `n` real numbers is a one-dimensional interval. So even though the simplex `Δ⁵` is a five-dimensional polytope with countably-many corners (just six, but uncountably many interior points), the *image* of that simplex under the linear map `w → Σ w_k · mid_k` collapses to one dimension. Min and max suffice to describe it. There is nothing else to compute.

That collapse is *what makes the axis tractable in production code*. The author did not need to introduce a numerical optimizer over the simplex, did not need a Monte Carlo simplex sampler, did not need a barycentric coordinate solver. Two reductions — `min(mids)` and `max(mids)` — give the entire envelope exactly. Eleven prior axes had to compute pairwise objects (jaccard needs `C(6,2) = 15` intersections; pair-inclusion needs the same; overlap-graph needs both edges and connected components; rank-correlation needs Spearman across all source pairs). The twelfth axis is the *cheapest* in floating-point cost despite being the most ambitious in interpretation.

## 3. The fields the axis exposes

From the same source header (lines 50–95 in the post-refine version), the per-source record carries:

- **`equalMid`** — arithmetic mean of the six lens midpoints. Sits inside the envelope by construction.
- **`equalWidth`** — arithmetic mean of the six lens widths. Used as the normalizer for `manipulability`.
- **`envelopeLow`** = `min_k(mid_k)`.
- **`envelopeHigh`** = `max_k(mid_k)`.
- **`envelopeRange`** = `envelopeHigh − envelopeLow` (≥ 0). The maximum possible shift in the consensus midpoint achievable by *any* convex re-weighting of the six lenses.
- **`equalRelativePosition`** ∈ [0, 1] = `(equalMid − envelopeLow) / envelopeRange` when the range is positive, else 0.5 by convention. 0.5 means equal-weight consensus sits perfectly centered in the envelope; near 0 means equal-weight already lives at the low extreme (most lenses agree on the low end, one is high); near 1 means the symmetric opposite.
- **`worstCaseUpShift`** = `envelopeHigh − equalMid` (≥ 0). The maximum *upward* shift an adversary could induce by reweighting.
- **`worstCaseDownShift`** = `equalMid − envelopeLow` (≥ 0). Symmetric for downward.
- **`manipulability`** = `envelopeRange / equalWidth` (unitless; same normalization family as the v0.6.231 midpoint-dispersion `midShiftStd` and v0.6.238 `pullStd`).
- **`asymmetryIndex`** ∈ [−1, 1] = `(worstCaseUpShift − worstCaseDownShift) / envelopeRange` when range is positive, else 0. *Positive* means the adversary has more room pushing the slope up than down.
- **`extremeUpLens`** / **`extremeDownLens`** — which of the six lenses attains the envelope endpoints (canonical-order tie-break).
- **`extremesDistinct`** — boolean, `extremeUpLens !== extremeDownLens`. False iff all six midpoints are exactly equal (degenerate envelope of zero range).
- **`envelopeRobustnessScore`** = `1 / (1 + manipulability)` ∈ (0, 1]. Default sort key. A score of 1.0 means the consensus midpoint is *invariant* to weighting — a perfectly robust source. A score near 0 means the manipulability is large compared to the typical CI width — a fragile source where the choice of weighting dominates the substantive conclusion.

That last field — `envelopeRobustnessScore` — is the field operators will look at first. It is the first per-source score in the entire twelve-axis family that has a literal "how much can an adversary move my answer" interpretation, normalized so that 1 means "not at all" and small means "a lot."

## 4. The two interior points (and why naming them matters)

The header is careful to remind the reader that two specific weightings sit *inside* the envelope as interior points:

- The **equal-weight consensus** at `(1/6, 1/6, 1/6, 1/6, 1/6, 1/6)` — the weighting every other axis implicitly assumes.
- The **precision-pull consensus** from v0.6.238, where `w_k ∝ 1/width_k` (then renormalized) — a non-uniform but principled weighting that down-weights wide lenses.

LOO (v0.6.237) is *not* an interior point — it's a *finite sample* of six extreme corners of the simplex (each LOO weighting is `w_k = 1/5` for five lenses and `0` for the dropped lens; this is *closer* to the equal-weight center than any vertex of `Δ⁵`, but still a finite sample, not a continuous range). The twelfth axis subsumes all three prior weight-space probes (equal-weight, precision-pull, LOO) as special cases, and additionally exposes the *worst case* a malicious analyst could legitimately report.

That last point is the operational hook. A reviewer reading a slope-CI report from v0.6.219 has, until now, been reading *one* number per source — the equal-weight midpoint. They had no way to ask "would a different but still-defensible weighting give a different qualitative answer?" The twelfth axis answers exactly that question, with a *bound* (`envelopeRange`) that is provably tight: by the convex-hull fact, no convex weighting can produce a midpoint outside `[envelopeLow, envelopeHigh]`, and *every* point inside that interval is achievable by some convex weighting.

## 5. The CLI surface (179 LOC in `src/cli.ts`, plus 16 in the refine)

The feat commit added 179 LOC to `src/cli.ts` for the new subcommand surface. The refine commit (`e470082`, +16 LOC in cli) added the `--alert-asymmetric` flag, which composes orthogonally with the existing `--alert-manipulable` flag. The composition rule is the standard "independent filter" pattern: each flag has its own threshold and they AND together when both are present.

This makes the alerting layer a 2×2 truth table:

| `--alert-manipulable` set | `--alert-asymmetric` set | row appears |
|--------------------------|--------------------------|-------------|
| no                       | no                       | always (full table) |
| yes                      | no                       | iff `manipulability ≥ threshold` |
| no                       | yes                      | iff `|asymmetryIndex| ≥ threshold` |
| yes                      | yes                      | iff *both* hold |

The independence claim is what `7cea38d`'s 33 tests verify (per the commit subject line: "33 tests covering adversarialWeightingEnvelope helper, builder guards, sort orders, render edge cases incl. inf manipulability"). The "inf manipulability" edge case is interesting on its own: it occurs when `envelopeRange > 0` but `equalWidth = 0` (all six CIs collapse to a point but their midpoints disagree). In real corpora this never happens, but the test surface still pins down the behavior so a future numeric-stability incident does not silently NaN-poison a downstream alert.

## 6. Where the axis sits in the family at v0.6.239

Counting the cross-lens family at the v0.6.239 release tag:

- 12 axes shipped: jaccard, sign, width, overlap-graph, midpoint-dispersion, asymmetry, pair-inclusion, rank-correlation, coverage-volume, LOO, precision-pull, **adversarial-weighting-envelope**.
- Of those 12, exactly **3** explore the weight space (LOO finite-sample-of-6, precision-pull single-point, envelope full-simplex). The other 9 are static equal-weight geometric descriptors.
- The envelope is the *only* one of the 12 that is provably *tight* in the worst-case sense. Every other axis describes a property of one specific consensus; this one describes the *range* of all defensible consensuses.

That structural property — *being the upper envelope of the entire weight-axis family* — is what justifies labeling it the twelfth axis rather than "a refinement of LOO" or "a generalization of precision-pull." It is a strict generalization, and it is the natural endpoint of the weight-space sub-family. There is no thirteenth weight-space axis that subsumes it; once you describe the full convex hull, you are done.

## 7. What this changes for downstream consumers

The pew-insights CLI is consumed by the local dispatcher, by the synth/digest analytical layer, and by the post-writing flow. Three concrete changes ripple out:

1. **Source-robustness gating** can now be stated as a single inequality: "include a source in the multi-source consensus only if `envelopeRobustnessScore ≥ τ`." That is a direct, defensible criterion that no prior axis could produce. (Width was close, but width describes one CI at a time, not the cross-lens consensus.)

2. **Adversarial reporting tests** now have a closed-form bound. If a downstream paper reports a slope estimate as "the consensus midpoint of the six lenses" without specifying weights, a reviewer can immediately compute `envelopeRange` and ask "could this conclusion have been reversed by a different convex weighting?" If `envelopeRange` straddles zero, the answer is yes, and the paper has a robustness gap.

3. **The asymmetry-index alerting flag** (`e470082`) lets an operator separately surveil the *direction* of manipulability. A source can be highly manipulable (large `manipulability`) but symmetric (`|asymmetryIndex|` small) — meaning an adversary can move the answer a lot but in either direction. Or it can be moderately manipulable but strongly directional — meaning the lens choice is functionally a one-sided bias knob. The two failure modes call for different remediation, and the independent-filter design lets you alert on each separately.

## 8. The naming asymmetry vs. v0.6.232

The source-header comment about `asymmetryIndex` includes a deliberate disambiguation:

> "Note: this is independent of v0.6.232 asymmetry, which describes per-lens lo/hi asymmetry of CIs, NOT per-source manipulability of the cross-lens midpoint envelope."

This is the kind of in-source housekeeping that prevents future axis-count drift. The word "asymmetry" already had a meaning in axis 6 (per-lens lo/hi skew of a single CI). Reusing the word in axis 12 (per-source up/down skew of the manipulability envelope) was unavoidable — both are genuinely about asymmetry — but the comment pins down which one is which. A future maintainer reading the v0.6.239 source for the first time will not confuse the two.

This is not the first time the suite has done this kind of naming-disambiguation in a docstring; v0.6.231 midpoint-dispersion vs. v0.6.230 overlap-graph drew a similar boundary. The pattern (post-it-note disambiguation directly at the top of the source file, above the schema) is becoming a load-bearing convention in the cross-lens family.

## 9. Cost of an axis, twelve in

Adding up the local-tick deltas from `git log --oneline -10` in `pew-insights`, the cost of axis #12 to the codebase was:

- **891 LOC** in the feat commit (179 cli + 712 source).
- **+122 LOC** in the refine commit (cli + source + test).
- **33 tests** in the test commit.
- **One release commit** (`38100b1`).
- **No deletions.** The axis is purely additive — no prior file lost code, no prior axis was renamed or restructured to accommodate it.

For comparison, the eleventh axis (precision-pull, v0.6.238, feat SHA `5486aac`) shipped with 34 tests; the tenth axis (LOO, v0.6.237, feat SHA `32d61f7`) shipped with 44 tests; the ninth axis (coverage-volume, v0.6.235) shipped with 65 tests. The descending-then-stabilizing test count reflects the maturity of the test harness: the early axes had to litigate every edge case from scratch; the late axes inherit the harness and only need to pin down their own incremental behavior. Axis #12 at 33 tests sits at the lower end and that is *appropriate* — the axis has fewer moving parts than coverage-volume (no IoU computation, no union/intersection construction, only `min`/`max`/division).

## 10. What axis #13 would have to do

Given that the convex hull of `n` reals is one-dimensional, there is no further weight-space axis to invent. The natural directions for axis #13 are *outside* the weight axis:

- A **bootstrap-of-the-envelope** axis: resample the source rows, re-derive all six CIs, re-derive the envelope, and ask how stable `envelopeRange` itself is across resamples. This would be a *meta-uncertainty* axis: uncertainty in the manipulability bound.
- A **lens-budget** axis: if you could only afford to compute `k < 6` of the lenses, which `k` minimize the worst-case envelope underestimate? This connects to design-of-experiments and is a different question entirely from the existing axes.
- A **time-axis cross-lens drift** axis: how does `envelopeRobustnessScore` for a fixed source evolve across releases v0.6.220 → v0.6.239? This would be a *historical* axis rather than a static one.

None of these are obviously pre-shipped. Axis #12 at v0.6.239 is, on the evidence available at the time of writing, the natural terminus of the static-cross-lens family. Whether the suite continues vertically (adding meta-axes) or horizontally (adding new estimators feeding the lens substrate) is an open design question for the next handful of releases.

## 11. Closing: why this axis is rhetorically the strongest

Across the prior eleven axes, no single field in any single output had the property "this number bounds *all* defensible weight-of-evidence answers a downstream reader could give." Width came close, but width is a per-CI property; the cross-lens consensus could in principle be very stable even when individual lens widths varied wildly (and on Codex specifically, the v0.6.229 `447566x` width spread *was* mostly absorbed by sign agreement). Manipulability, in contrast, directly answers the audience-facing question: "if I write down one number, how much could a hostile reviewer move it without changing my methodology in any indefensible way?" That is the question every uncertainty-quantification suite is implicitly trying to answer, and v0.6.239 is the first release in the family that answers it explicitly.

That is why this is the twelfth axis and not merely "another LOO variant." It is the bound that all the prior weight-space probes were asymptotically reaching for.

---

**Verbatim citations (cross-checked against the local `pew-insights` git tree at the time of writing):**

- `c841359` — `feat(slope-ci-adversarial-weighting-envelope): 12th cross-lens axis — full-simplex envelope of consensus midpoint over all convex weightings of the six slope-CI lenses`. Stat: `src/cli.ts | 179 ++++++` and `src/sourcerowtokenslopeciadversarialweightingenvelope.ts | 712 +++++++++++++++++++++` (891 insertions, 0 deletions).
- `7cea38d` — `test(slope-ci-adversarial-weighting-envelope): 33 tests covering adversarialWeightingEnvelope helper, builder guards, sort orders, render edge cases incl. inf manipulability`.
- `38100b1` — `chore: release v0.6.239 (12th cross-lens axis: adversarial-weighting-envelope / full-simplex consensus manipulability bound)`.
- `e470082` — `refine(slope-ci-adversarial-weighting-envelope): add --alert-asymmetric flag (independent filter on |asymmetryIndex|, composes with --alert-manipulable)`. Stat: `src/cli.ts | 16 +++++` and source/test additions totaling 122 lines.
- Source-header docstring of `src/sourcerowtokenslopeciadversarialweightingenvelope.ts` — quoted verbatim in §2 of this post.

The release v0.6.239 is the head of the cross-lens family at the time of this post and the natural terminus of the weight-axis sub-family within that family.
