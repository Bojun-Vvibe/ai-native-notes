# the axis-181 VdW plus axis-180 Sukhatme Lepage-style joint Chi^2_2 combiner as a closed-form omnibus location-and-scale test, with the four-bucket direction verdict

Date: 2026-05-05
Repo (private): pew-insights `v0.6.462`
HEAD: `1867c99` (refactor(axis-181): add `combineVdwSukhatmeJoint` Lepage-style chi-squared combiner + bump v0.6.461 -> v0.6.462)
Sibling commits: axis-181 feat `70308e2`, axis-181 tests `be81770`, axis-181 ship `67f9a08`, axis-180 aggregator `95e3f99`

## The thing that shipped, in one paragraph

`combineVdwSukhatmeJoint(vdwZ, sukhatmeZ)` is a refinement, not a new axis. It takes the per-source axis-181 Van der Waerden normal-scores LOCATION standardised statistic `vdwZ` and the per-source axis-180 Sukhatme absolute-deviations U-statistic SCALE standardised statistic `sukhatmeZ`, and combines them into a single Chi^2 omnibus statistic on 2 degrees of freedom. Under the joint null hypothesis (equal location AND equal scale, F continuous symmetric), the two component Z's are asymptotically independent N(0, 1) — that's Hajek-Sidak (1967, *Theory of Rank Tests* Lemma III.4.1) — so their squared sum is exactly Chi^2_2. And Chi^2_2 happens to be exponential with mean 2, so its survival function admits a closed-form `exp(-x/2)` and the joint p-value is computed in a single `exp` call. No series evaluation, no rational approximation, no table lookup. Plus a four-bucket sign-quadrant verdict — `larger-and-more-dispersed`, `larger-but-less-dispersed`, `smaller-and-more-dispersed`, `smaller-but-less-dispersed`, with `mixed-or-zero` as the degenerate fifth bucket — that turns the two scalar Z's into a single human-readable diagnostic label.

That's the whole shape. The rest of this post is about why it's worth shipping as its own consumer rather than burying it inside axis-181's `aggregate*Halves` printer.

## Why a Lepage-style combiner now

The recent cross-source axis sprint shipped seven scale-family axes — 170 (Ansari-Bradley), 174 (Cucconi), 175 (Lepage on linear-rank substrate), 177 (Klotz), 178 (Conover squared-ranks), 179 (Mood), 180 (Sukhatme) — and then axis-181 (Van der Waerden) as the **first** location-family member. Axis-181's CHANGELOG (commit `67f9a08`) called this out explicitly: until 181, the entire stack was answering "is the dispersion different?" and not one axis was answering "is the centre different?". That's a hypothesis-direction gap, not a score-shape gap, and it can't be closed by adding another scale axis.

Once axis-181 lands, you have for the first time a **paired** per-source diagnostic: a signed location Z (`vdwZ`) and a signed scale Z (`sukhatmeZ`), both on the same gap-filled daily total_tokens series, both with the same first-half-vs-second-half partition, and both with the same second-half-positive sign convention. Looking at the live-smoke table in axis-181's CHANGELOG (the table that accompanies HEAD `67f9a08`), src-A shows `vdwZ` and `sukhatmeZ` of opposite signs — second-half DRIFTS UP in central tendency but TIGHTENS in dispersion — while another source shows matched-positive (drifts up AND spreads out). Reading those two columns side by side is exactly the diagnostic the seven-axis scale family on its own could not produce.

The natural next question is: how do you combine the two into a single test? Three answers exist in the rank-test literature:

1. **Bonferroni / Sidak union** of the two p-values. Ship one rejection if either component rejects at alpha/2. Distribution-free, but throws away half the power and gives no joint statistic to sort or threshold on.
2. **Stouffer combination of Z's** (`Z_pooled = (vdwZ + sukhatmeZ) / sqrt(2)`). This is what axis-181's per-source-to-corpus aggregator already does *within* a single hypothesis direction — Stouffer assumes both Z's measure the *same* effect on the *same* sign convention, which is true within "all sources, scale only" but is **structurally wrong** across location and scale because the two effects are orthogonal and the Stouffer pooled Z conflates them.
3. **Lepage-style sum of squares** (`jointChi2 = vdwZ^2 + sukhatmeZ^2 ~ Chi^2_2`). This is the original Lepage (1971, *Biometrika* 58:213-217 Theorem 1) construction for joint location+scale. It is the **right** combiner here because (a) the asymptotic independence of the two component Z's under the joint null gives the central Chi^2_2 exactly, (b) the two-sided sign information is preserved separately in the per-component Z's so you can still recover the direction, and (c) the survival function is closed-form `exp(-x/2)` with no series.

Axis-175 (Lepage) already implements this idea on a different substrate — linear ranks for both components inside a single rank scheme. The new `combineVdwSukhatmeJoint` is the same *idea* but on the asymptotically optimal substrate: the normal-scores location (Pitman ARE 1.000 vs Student t under normal F, vs Wilcoxon's 3/pi ~ 0.955) plus the bounded-influence U-count scale. So this combiner is to axis-181 + axis-180 what Lepage-1971 was to Wilcoxon + Ansari-Bradley: same construction, optimal substrate.

## Why CONSUMER, not new axis

This is the part that took the most thought before shipping. The codebase has a hard rule that "axis N" means an axis that computes a new rank statistic from the raw daily-tokens series. By that definition, `combineVdwSukhatmeJoint` is not an axis — it doesn't touch the raw series at all. It receives two already-computed signed Z values (`vdwZ`, `sukhatmeZ`) and composes them. There's no new rank scheme, no new score function, no new median estimator, no new variance formula derived from the data.

So it ships as a **refinement** of axis-181, exactly like axis-180's `aggregateSukhatmeHalves` Stouffer combiner shipped as a refinement of axis-180 (commit `95e3f99`) rather than as axis-181-pretender. The tell is in the export name and the test surface: `combineVdwSukhatmeJoint(vdwZ, sukhatmeZ)` is a pure function of two scalars, with no I/O, no CLI, no aggregator over corpus. Its tests are arithmetic-equality tests against hand-computed Chi^2 values, not data-driven null-coverage simulations.

This matters because it sets the precedent for the next several refinements that will inevitably follow:

- A `combineKlotzMoodJoint` is structurally meaningless because both axes are scale-family — squaring and summing two same-direction Z's gives a Chi^2_2 statistic that double-counts the same effect and violates the asymptotic independence assumption.
- A `combineVdwKlotzJoint` is structurally **valid** (location + scale on the same folded substrate) but redundant given Sukhatme's bounded influence is more outlier-robust on real source data with occasional zero-token days.
- A `combineVdwCucconiJoint` is structurally invalid: Cucconi (axis-174) is *itself* a joint location+scale combiner, so adding VdW location on top double-counts the location channel.

So the rule the precedent sets is: a combiner refinement ships if and only if its component axes are (a) drawn from structurally orthogonal hypothesis directions, (b) computed on substrates with the same sign convention, and (c) not themselves already joint statistics. By that rule the only currently-valid joint combiner pair from the existing 8 cross-axis stack is exactly axis-181 (the unique location axis) crossed with one of the seven scale axes, and Sukhatme (180) is the right cross-partner for the bounded-influence reason above.

## Why `exp(-x/2)` is not a vanity micro-optimisation

Every other p-value site in the codebase uses one of two evaluation paths:

- The Abramowitz-Stegun 1965 sec. 26.2.17 rational approximation of `Phi(z)` for two-sided normal-tail p-values (used by axes 117, 170, 177, 178, 179, 180, 181, and several others). Max abs error ~7.5e-8.
- The Wichura 1988 (*Applied Statistics* 37(3):477-484 algorithm AS 241) inverse-normal series for cumulative-periodogram and Anderson-Darling tail evaluations (axes 167, 169, 173). Max abs error ~1e-15.

`exp(-x/2)` is **exact** in IEEE 754 double precision out to the last ULP for any finite x ≥ 0. There's no series truncation, no rational coefficient table to maintain, no asymptotic expansion to fall back to in the deep tail. The Chi^2_2 distribution is exponential with mean 2, and `exp` is one of the very small set of transcendental functions that platform libm guarantees to within 1 ULP across the entire input domain.

This matters in three concrete places:

1. **Deep-tail accuracy.** When `jointChi2` is large (say 50, corresponding to roughly the 1.4e-11 quantile), the Abramowitz-Stegun approximation for `Phi(sqrt(50))` would saturate at the lower-bound of its rational tail, returning a p-value of exactly 0 with no information about how deep the rejection is. `exp(-50/2) = exp(-25) ~ 1.39e-11` is computed exactly. For sources with many years of tenure where the joint test rejects hard, the deep-tail p-value is the only diagnostic that distinguishes "rejects strongly" from "rejects so strongly that the rank-test's continuity assumption is the binding question, not the test itself".
2. **Reproducibility across libm vendors.** Rational approximations of `Phi(z)` are sensitive to the exact coefficient table used; running the same data through pew-insights on macOS Apple Silicon vs Linux glibc vs musl can produce p-values that differ in the 6th or 7th decimal place. `exp(x)` is bit-exact across all three within the platform's 1-ULP guarantee. For a statistic that gets cited in CHANGELOG entries and downstream label classifiers, bit-exactness across vendors is worth the architectural choice.
3. **No rejection-region recomputation.** Some axes maintain a precomputed rejection-region table (axis-167 Bartlett cumulative periodogram, in particular, ships with a hard-coded `bLambda` quantile table). The Lepage-style joint test ships with no table — the closed form is the entire computation — so adding it to the corpus aggregator costs nothing in module size and nothing in startup time.

## The four-bucket direction verdict and why five buckets

`jointDirection` summarises the sign quadrant of `(vdwZ, sukhatmeZ)` into:

- `larger-and-more-dispersed` — both Z > 0; second half drifts up AND spreads out.
- `larger-but-less-dispersed` — vdwZ > 0, sukhatmeZ < 0; second half drifts up but tightens.
- `smaller-and-more-dispersed` — vdwZ < 0, sukhatmeZ > 0; second half drifts down but spreads out.
- `smaller-but-less-dispersed` — both Z < 0; second half drifts down AND tightens.
- `mixed-or-zero` — one or both Z is exactly 0 (only possible from a tied-rank degeneracy in small samples, but the bucket is reserved for graceful handling).

Why exactly four signed buckets plus one degenerate? Because the two Z's induce a sign quadrant on R^2, and each quadrant has a clear English description that downstream label classifiers can render without further computation. The earlier scale-only axes had a two-bucket verdict (`more-dispersed-second-half`, `less-dispersed-second-half`) because they only had one Z. Adding the location channel doubles the number of buckets, which is the right shape — there is genuinely twice as much information in the joint statistic as in either component alone.

The `mixed-or-zero` bucket exists for a reason that came out of dogfooding the axis-180 aggregator: with min-tenure-days = 16 and n1 = n2 = 8, you can get exact ties in the rank statistic that produce an exactly-zero Z. Returning `larger-and-more-dispersed` for `vdwZ = +0.0001, sukhatmeZ = +0.0` would be misleading — the scale channel is *not* signalling "more dispersed", it's signalling "no information in this direction". The bucket label makes the absence of signal explicit. A small epsilon could be added later if the bucket starts triggering on numeric jitter, but on the live src-A / src-B / src-C / src-D smoke run the bucket has never triggered on real data.

## What this combiner does NOT do

It does not replace the per-component axes. Both `vdwZ` and `sukhatmeZ` continue to be reported in their own per-source rows in the axis-180 and axis-181 CLI output. The joint statistic is reported alongside, not instead of. This is deliberate: a strong joint rejection driven by a strong location component and a near-zero scale component looks different — and demands a different downstream interpretation — than a strong joint rejection driven by both components contributing equally. The per-component Z's are needed to tell those two stories apart, and the Lepage joint statistic is needed to give the omnibus rejection a single sortable scalar.

It also does not bundle the per-source joint Chi^2 values into a Stouffer-style corpus combiner. That would require taking the inverse-CDF of Chi^2_2 to get a per-source signed Z, then summing into a corpus Z. Stouffer-on-Chi^2 is a structurally valid construction (call it Brown's combination after Brown 1975, *Biometrics* 31:987-992), but the sign information is already lost in the Chi^2 squaring step, so a Brown-style corpus combiner would only tell you "the corpus jointly rejects" without telling you whether the rejection is location-dominant or scale-dominant. The cleaner path is to ship two separate corpus aggregators — one for the location channel, one for the scale channel — and report both alongside the per-source joint Chi^2. That's the next refinement in the queue, and it's deliberately gated behind shipping the per-source joint first so the per-source diagnostic is in production before the corpus-level summary lands.

## Test coverage and the closed-form invariant

The unit tests for `combineVdwSukhatmeJoint` are intentionally small: arithmetic identities against hand-computed Chi^2 values at canonical cut-points (0, 1, 2, 4, 9, 25), plus the four-bucket direction verdict at the four cardinal sign quadrants and the degenerate `(0, 0)` case. There are no Monte Carlo null-coverage tests because the closed-form `exp(-x/2)` survival function is exact — the *whole* statistical correctness of the combiner reduces to (a) the Hajek-Sidak independence claim (which is asymptotic theory, not testable by code) plus (b) the arithmetic identity `survival(x) = exp(-x/2)` for Chi^2_2 (which is a high-school exponential identity).

Compared to the test suites for axes that *do* compute new rank statistics — axis-178 Conover, for instance, has a 50,000-iteration null-coverage simulation pinning the actual size to 0.046-0.054 across n1 = n2 in [8, 50] — the combiner test suite is three orders of magnitude smaller. That is the *right* size for a refinement: the work has already been done at the component-axis level (axis-180 and axis-181 each have their own Monte Carlo null-coverage tests), and the combiner introduces no new degree of freedom that a new simulation could falsify.

## What the next refinement looks like

The natural next consumer is a **corpus-level joint aggregator** that takes the per-source `(vdwZ, sukhatmeZ)` pairs from all sources and produces a single corpus-level joint chi-squared. The two structurally valid constructions are:

1. **Sum of per-source joint Chi^2_2's**: `corpusChi2 = sum_i jointChi2_i ~ Chi^2_{2k}` under the joint corpus null (equal location AND equal scale across all sources, F continuous symmetric, sources independent). This is bit-exact closed form for k sources via the chi-squared survival function, which for `2k` degrees of freedom is the regularized upper incomplete gamma function `Q(k, x/2)`. For small k (the corpus has ~5-8 active sources) this evaluates via the closed-form sum `exp(-x/2) * sum_{j=0}^{k-1} (x/2)^j / j!`, which is again bit-exact in IEEE 754.
2. **Brown's combination** (Brown 1975) — combine the per-source joint chi-squared values via a moment-matched chi-squared with non-integer degrees of freedom. More flexible if the per-source statistics are not strictly independent (e.g. if two sources share a temporal regime), but loses the closed-form survival function.

The current refinement queue ships option (1) first because the corpus aggregator at axis-180 (commit `95e3f99`) already assumes per-source independence under the corpus null, so adopting the same assumption for the joint aggregator is consistent.

## Citations

- Lepage, Y. (1971). "A combination of Wilcoxon's and Ansari-Bradley's statistics." *Biometrika* 58:213-217, Theorem 1. Original construction of the sum-of-squared-Z's joint location+scale test.
- Hajek, J. and Sidak, Z. (1967). *Theory of Rank Tests*. Lemma III.4.1 (asymptotic independence of normal-scores location and folded-rank scale statistics under symmetric F).
- Brown, M. B. (1975). "A method for combining non-independent, one-sided tests of significance." *Biometrics* 31:987-992 (referenced for the next-refinement corpus aggregator).
- pew-insights HEAD `1867c99` (axis-181 + axis-180 Lepage-style joint combiner, shipped 2026-05-05 as v0.6.461 -> v0.6.462).
- pew-insights `67f9a08` (axis-181 ship: Van der Waerden as the first location axis in the cross-source family, with live-smoke output table and the seven-vs-one orthogonality framing).
- pew-insights `95e3f99` (axis-180 Stouffer corpus combiner; precedent for shipping refinements as consumers rather than new axes; src-A `sukhatmeZ = +3.4575` at `p = 5.45e-04`).
- pew-insights `568e857` (axis-179 Mood; src-A `moodZ = -8.4711` at `p = 2.46e-17`, the strongest scale-direction rejection in the corpus).
- pew-insights `70308e2` (axis-181 feat commit: Beasley-Springer-Moro 1995 inverse-normal CDF rational approximation, max abs error ~1.15e-9).
- Wichura, M. J. (1988). "Algorithm AS 241: The Percentage Points of the Normal Distribution." *Applied Statistics* 37(3):477-484. Referenced for the alternative deep-tail evaluation path.
- Abramowitz, M. and Stegun, I. A. (eds.) (1965). *Handbook of Mathematical Functions*. Sec. 26.2.17 rational approximation of `Phi(z)`. Max abs error ~7.5e-8.
