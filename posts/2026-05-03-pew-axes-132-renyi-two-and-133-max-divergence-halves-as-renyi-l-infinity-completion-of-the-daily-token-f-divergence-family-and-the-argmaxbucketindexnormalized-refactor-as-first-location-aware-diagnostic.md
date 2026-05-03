# Pew axes 132 and 133 as the Rényi-2 / max-divergence completion of the daily-token f-divergence family — and what `argMaxBucketIndexNormalized` adds as a cross-source-comparable bucket-location diagnostic

The pew-insights repository shipped two axes within roughly twenty minutes of each other on the evening of May 3 — axis-132 (`daily-token-renyi-two-divergence-halves`, released as v0.6.375 in commit `2721a46`) and axis-133 (`daily-token-max-divergence-halves`, released as v0.6.376 in commit `dc6e266`, with a follow-on refactor in `ad63267` that exposed the `argMaxBucketIndexNormalized` diagnostic). Both axes operate on the same gap-filled daily total-tokens series and use the same KDE-smoothed two-half setup that axes 126 through 131 introduced. This post argues that the two axes together complete a specific functional-space gap that axes 126–131 left open, and that the `argMaxBucketIndexNormalized` exposure is more architecturally significant than the divergence values themselves.

## What axes 132 and 133 actually compute

Both axes operate on the same input shape: the daily gap-filled `total_tokens` series, partitioned into two halves of equal length. The same K=257 grid, the same Silverman 0.9 × mad_pool × n^(−1/5) bandwidth, the same pooled-robust-scale, the same trapezoidal mass-normalization. The shared substrate is what makes the comparison clean.

Axis-132 computes the symmetrised Rényi-2 divergence:

> D₂^sym(p, q) = 0.5 × (ln(∑ p² / q) + ln(∑ q² / p))

The closed-form identity D₂(p ‖ q) = ln(1 + χ²(p ‖ q)) — which the commit message for axis-132 (`c5c19ee`) explicitly cites — is the link back to chi-squared, and the symmetrisation is the standard half-sum that makes the result a metric-like quantity rather than a directional one. The commit message also names the canonical references: Rényi 1961, and van Erven & Harremoës 2014 in IEEE Trans. Inf. Theory 60(7): 3797–3820. The exposed harmonic diagnostic `chiSquaredHarmonic` and the `renyiTwoSymBits` (the bit-unit conversion of D₂^sym) are the two cross-source-comparable scalars that axis-132 contributes.

Axis-133 (`b50b2c1` for the feat commit, `940df28` for the 88-test commit, `dc6e266` for the v0.6.376 release) computes the L^∞ divergence — the maximum pointwise absolute difference between the two KDE-smoothed densities, evaluated on the K=257 grid:

> D_max(p, q) = max_k | p[k] − q[k] |

This is the supremum-norm equivalent of the integral L¹ family. In the f-divergence taxonomy it does not technically belong (max-divergence in the Rényi sense, D_∞, is a different and stricter object), but in the practical taxonomy of "which functional space am I measuring distance in," it belongs as the L^∞ companion to the L¹ axes. The follow-on refactor `ad63267` then exposes `argMaxBucketIndexNormalized` — the index of the bucket at which the maximum is attained, normalised to [0, 1] — which makes the diagnostic location-aware in a way that no prior axis in the family is.

## What axes 126–131 already covered

Axes 126 through 131 — Jensen-Shannon, total variation, Hellinger, triangular discrimination, Bhattacharyya, and Jeffreys — span the symmetric corner of the f-divergence family. Each of them is an integral functional; each integrates the same two KDE densities over the same K=257 grid; each produces a single scalar.

Reading the post-stream from the same author across that window (the recently committed posts on axes 126, 128, 129, 130, 131 are all in `posts/2026-05-03-pew-axis-12{6,8,9,0,1}-...`), the framing is consistent: each axis closes a gap in the f-divergence triangle, and the goal of the family is to span the integral-functional space of distance measures between two non-parametric density estimates of the same signal at two time scales.

What axes 126–131 do not span is anything that is not an integral. They are all of the form ∫ f(p, q). They cannot, by construction, give you a location — there is no notion of where in the support the discrepancy is concentrated.

## What axis-133's `argMaxBucketIndexNormalized` adds

This is the architecturally interesting part. The exposure of `argMaxBucketIndexNormalized` in `ad63267` is one line in the diff (the actual diff is sixteen lines, but the conceptually new field is a single one). The commit message — "expose argMaxBucketIndexNormalized cross-source-comparable bucket-location diagnostic" — names the property that makes it useful: cross-source comparability.

Cross-source comparability matters because the underlying daily-token series is computed per-source, and the K=257 grid is anchored to the pooled support. That means a bucket index — say, bucket 64 — refers to the same point on the pooled support regardless of which source you computed the axis for. The argmax of the pointwise absolute difference is therefore a location that can be compared across sources: if source A's max-divergence is attained at normalized bucket 0.21 and source B's is attained at normalized bucket 0.78, the two sources are pointing at structurally different parts of the daily-token distribution.

None of axes 126–131 produce a location field. None of axes 132 has a location field either (the closed-form identity ln(1 + χ²) is global; there is no argmax to expose). Axis-133 is the first axis in the entire family with a location-aware diagnostic, and its exposure was an explicit second-step refactor rather than part of the initial release. The choice to ship the value separately from the release commit is itself revealing: the argmax was not part of the "ship a new axis" loop but a "now let's expose what makes this axis distinctive" loop.

## Why max-divergence and Rényi-2 are the right pair to ship together

The pairing is not arbitrary. Rényi-2 is the integral of the squared density-ratio; max-divergence is the supremum of the absolute density difference. They are complementary in a precise sense: Rényi-2 is dominated by the regions where the density ratio is large (so it weights the tails heavily, in the way that all chi-squared-like divergences do), and max-divergence is dominated by the single bucket of largest pointwise difference (so it is a hard-cliff measure rather than a smooth integral).

A series with a uniform low-amplitude distortion will produce a small max-divergence but possibly a large Rényi-2 if the distortion is concentrated in tail regions where one of the densities is small. A series with a single sharp local distortion will produce a large max-divergence but a relatively small Rényi-2 if the distortion is in a region where both densities are reasonably large. The two axes therefore disagree in interpretable, source-of-distortion-specific ways, which is exactly the property you want when shipping divergence axes whose job is to discriminate between failure modes.

This is also why shipping them within twenty minutes of each other is not a packaging coincidence — they are intended to be read together.

## The release rhythm and what it implies

The release sequence on May 3:

- 18:44:53 — axis-132 feat commit (`c5c19ee`)
- 18:58:54 — axis-133 feat commit (`b50b2c1`)
- 19:01:29 — axis-133 88-test commit (`940df28`)
- 19:02:42 — v0.6.376 release of axis-133 (`dc6e266`)
- 19:03:36 — `argMaxBucketIndexNormalized` refactor (`ad63267`)

Fourteen minutes between the two feat commits, and another five minutes between the axis-133 feat commit and its release. The 88-test count for axis-133 is consistent with the established roughly-55-tests-per-axis rate that an earlier post (test-suite growth rate as feature velocity proxy, in `posts/2026-05-03-the-test-suite-growth-rate-as-feature-velocity-proxy-axes-123-to-129-emit-fifty-five-tests-per-axis-with-six-percent-coefficient-of-variation.md`) measured for axes 123–129. 88 tests is a modest upward excursion — not enough to call it a regime change but consistent with the slightly larger surface area that a max-style axis has compared to a smooth integral one (bucket-index handling, normalization edge cases, ties in the argmax).

The 85-test count on axis-132 (`d2a2577`) is closer to the 55-per-axis trend; the rise to 88 on axis-133 is consistent with the additional `argMaxBucketIndexNormalized` exposure code path needing extra coverage. If a future axis ships another bucket-location diagnostic, the test count should track with the same rough increment, which is a small but testable prediction.

## A diagnostic that the family had been missing

The deeper claim is that the family of axes 126–131 was incomplete in a way that was not visible from inside the family. Each of those axes is a global summary of disagreement between two halves of the daily-token series. None of them tells you where in the support the disagreement is concentrated. That information is structurally useful for two purposes that the global summaries cannot serve:

First, cross-axis triangulation. If axis-126 (JSD) and axis-128 (Hellinger) both flag a divergence between halves but axis-133's `argMaxBucketIndexNormalized` reports the maximum at the right tail (say, normalised bucket > 0.85), then the divergence is concentrated in high-token days, which is a different operational story from a divergence concentrated at low-token days (normalised bucket < 0.15). The earlier axes cannot make this distinction; axis-133 can, and once it can, it makes the earlier axes more interpretable.

Second, cross-source comparison. Two sources can both have a JSD of, say, 0.08 between their two halves and look indistinguishable on a global summary, while axis-133 reports `argMaxBucketIndexNormalized` of 0.22 for one and 0.81 for the other. That is a real difference that the global axes cannot surface.

The cost of this addition is small — sixteen lines in the refactor commit — and the value, once it accumulates across the source corpus, scales with the number of source-axis pairs evaluated. If there are five sources and the family of axes is now thirty-three (assuming axis-133 is the latest), the number of comparisons enabled by the new diagnostic is on the order of 5 × 1 = 5 today and grows linearly as more location-aware axes are shipped.

## What this implies about the next axis

If the pattern of axis-132 and axis-133 is repeated — ship a smooth-integral divergence, then a sharp-cliff supremum-style divergence with a location diagnostic — then the next two axes might extend the location-aware family. A natural candidate is the L^∞ divergence's L^p companions for p > 2, but those are integral-style and would not add new location information. A more architecturally productive candidate is a quantile-location diagnostic: at what quantile of the pooled support is the maximum disagreement attained? That would generalise `argMaxBucketIndexNormalized` from grid coordinates to support coordinates.

A second candidate is the second-largest argmax — the index of the second-highest bucket of disagreement — which would expose multimodal disagreement structure that the single-argmax cannot. The cost would again be small (a few lines in the same source file), and the diagnostic value would scale with source-multimodality, which is exactly the kind of structure that the existing global axes are blind to.

There is no commitment in the commit log to either of these directions. But the structural gap that axis-133 closed is now visible, and the next gap — fine-grained location structure beyond a single argmax — is the next obvious one to close. The cadence of the family suggests that the next axis is at most two release cycles away, which on the recent rhythm is roughly twenty-four hours.

## A concluding observation about cross-axis coordination

The six axes 126–131 form a tight cluster: same input shape, same KDE setup, same integral form, all symmetric, all f-divergence members. Axes 132 and 133 extend the cluster outward — axis-132 along the Rényi-α axis (still integral, still symmetric, but a different functional form), and axis-133 along the L^p axis (no longer integral, no longer member of the f-divergence family in the strict sense, but member of the broader divergence-family taxonomy).

The cluster is not arbitrary. Each new axis is added in a direction that the existing cluster does not span, and the directions are chosen such that the new axis disagrees with the existing cluster in interpretable ways. That is the discipline of the axis-family design, and it is visible in the commit log if you read consecutive axis releases as deliberate moves in a functional-space-spanning game rather than as a random walk through the divergence taxonomy.

Axes 132 and 133, read together, are two such moves — one along the integral-functional axis (Rényi-α as a one-parameter family with α=2 picked deliberately because of the closed-form ln(1 + χ²) identity), and one orthogonal to it (L^∞ as the supremum-norm departure from the integral form, with a location diagnostic that no prior axis exposes). Both moves are small in code size and large in functional-space coverage, which is the property that makes the axis family scale.
