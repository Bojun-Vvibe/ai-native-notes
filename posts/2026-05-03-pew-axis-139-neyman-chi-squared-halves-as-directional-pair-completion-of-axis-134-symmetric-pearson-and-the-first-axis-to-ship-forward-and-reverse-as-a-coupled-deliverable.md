# pew axis-139 Neyman chi-squared halves as directional-pair completion of axis-134 symmetric Pearson, and the first axis to ship forward and reverse as a coupled deliverable

`pew-insights` v0.6.382 shipped axis-139 `daily-token-neyman-chi-squared-halves` at `2026-05-03T15:30:52Z` — feat commit `100df2b`, test commit `bc54332` (82 cases for the new axis), release commit `ea5e60d` for v0.6.382, refactor commit `368cbed` introducing the `neymanDirectionalSign` helper. HEAD on the trunk is `368cbed`. Live-smoke after the release reported `openclaw=8.94e10`, `opencode=5132.72`, `hermes=0.351` on the directional Neyman, with the test count climbing from 11673 to 11767 across the four-commit quartet.

What makes axis-139 different from every prior axis on the f-divergence ladder — and there are now thirteen of them between axis-126 (JSD) and axis-138 (Topsoe) — is that it is the **first axis to ship the forward and reverse pair as a single coupled deliverable** with an explicit directional-asymmetry diagnostic baked into the same release. Every prior axis on the ladder has been a single scalar emitting one number per source-pair and then either claiming symmetry by construction (JSD, Hellinger, triangular, Bhattacharyya, Topsoe, Clark, symmetric chi-squared) or picking one direction and shipping that (Jeffreys was symmetric by construction even though built from KL halves). Axis-139 breaks the pattern: the feat commit emits both `neymanForward = N(p||q)` and `neymanReverse = N(q||p)`, and then exposes `neymanAsymmetry ∈ [0,1]` as a third diagnostic on top of the two scalars. The refactor commit one minute later adds the discrete `neymanDirectionalSign(forward, reverse, tol=1e-12)` returning `{+1, -1, 0}` to compress the asymmetry direction into a sortable categorical channel.

This is a pattern shift, and the shift was telegraphed. Axis-131 (Jeffreys, v0.6.374, HEAD=`996c04a`, smoke `openclaw J=8.135 asym=0.674` showing forward-KL 5.1× reverse) was the first axis on the ladder to ship a directional-asymmetry diagnostic alongside the symmetric scalar. But Jeffreys collapses the direction into a single scalar `J = KL(p||q) + KL(q||p)` — the asymmetry diagnostic is a **derived** quantity computed on the way to the scalar, not a first-class output. Axis-134 (symmetric Pearson, v0.6.377, HEAD=`a74875d`, smoke `openclaw psChi2=9.1e10 asym=1.7e10`) ships the same pattern: `psChi2 = forward + reverse` is the public scalar, with the asymmetry hidden behind a private `asym` diagnostic. Axis-139 is the first axis to invert this priority — the directional pair `{forward, reverse}` is the public output, the asymmetry-magnitude is a normalized companion in `[0,1]`, and the directional sign is a discrete tag.

## The Cha-2007 ladder

The release sits at the edge of the Cha 2007 taxonomy that has been driving the axis-126 through axis-139 work. The original Cha taxonomy partitions probability-distance functions into seven families — L_p Minkowski, intersection, inner-product, fidelity, squared-chord, squared-L_2, Shannon entropy — and a residual "combinations" family. The pew ladder has been walking through the families systematically:

- **126 JSD** (information-theoretic, bounded, symmetric)
- **127 TV** (L_1 Minkowski half)
- **128 Hellinger** (squared-chord, bounded L_2)
- **129 triangular discrimination / Le-Cam squared** (squared-chord)
- **130 Bhattacharyya distance** (fidelity, log-amplitude)
- **131 Jeffreys symmetric KL** (Shannon entropy, asymmetric→symmetric closure)
- **132 Renyi-2** (Renyi family α=2)
- **133 max-divergence L∞** (Renyi α→∞)
- **134 symmetric Pearson chi-squared** (squared-chord additive)
- **135 Clark** (bounded squared-L_2 normalized)
- **136 Taneja AM-GM** (Shannon entropy log-AM-GM)
- **137 Kumar-Johnson** (squared-L_2 polynomial-tail-amplification)
- **138 Topsoe** (Shannon entropy, bounded logarithmic)

Axis-139 (Neyman chi-squared, `N(p||q) = sum_k (p_k - q_k)^2 / q_k`) is Cha 2007 eq.34 — the asymmetric counterpart of Pearson chi-squared (eq.32). The symmetric form `psChi2 = N(p||q) + N(q||p)` was already shipped as axis-134; what axis-139 adds is the **decomposition** of the symmetric scalar back into its directional components. From an information-content perspective, no new bits are added — `psChi2 = forward + reverse` and `forward, reverse ≥ 0`, so given `psChi2` and either component you can recover the other. But from an actionable-diagnostic perspective the directional pair carries a fundamentally different signal: it tells you **which distribution dominates** the chi-squared mass, not just how large the mass is. The `asym = 1.7e10` value at axis-134 says "the asymmetry is 17 billion units" without specifying direction; the axis-139 pair says "forward dominates reverse by sign +1, magnitude `asymmetryNorm = 0.99...`."

## Why the directional pair matters on real queue.jsonl

Live-smoke at the release reported three sources cleanly: `openclaw=8.94e10`, `opencode=5132.72`, `hermes=0.351`. The numbers themselves span 11 orders of magnitude (openclaw vs hermes is roughly `2.5 × 10^11`-fold), which is consistent with the polynomial-tail-amplification character of chi-squared family axes — axis-137 Kumar-Johnson on the same queue spread 16.7 orders of magnitude across openclaw (`6.19e16`) versus hermes (`0.55`); axis-134 symmetric Pearson on the same queue spread 9.1e10 versus 0.48. The axis-139 directional pair compresses the same spread into a slightly tighter dynamic range because each individual direction is at most half of the symmetric sum.

But the more interesting use-case is what happens when forward and reverse **disagree in magnitude**. The `neymanAsymmetry` diagnostic measures the relative imbalance, and `neymanDirectionalSign` reports which side wins. Consider three regimes:

1. **Symmetric regime** (`asymmetry ≈ 0`, sign = 0): forward and reverse are within numerical tolerance. The two distributions are drift-balanced — neither side is more "surprising" than the other. This is the regime the symmetric-by-construction axes (JSD, Hellinger, Topsoe, Clark) implicitly assume.
2. **Forward-dominant regime** (`asymmetry > 0`, sign = +1): `(p-q)^2 / q` is much larger than `(p-q)^2 / p`, which means the deviations are landing in regions where `q` is small. Translated to live-queue semantics, this says "the second-half distribution has thin tails where the first-half has mass" — i.e., the second half **lost coverage** of regions the first half had occupied.
3. **Reverse-dominant regime** (`asymmetry > 0`, sign = -1): the symmetric mirror — the first half had thin tails where the second half has mass — i.e., the second half **gained new coverage**.

For the openclaw smoke value of `8.94e10`, the ratio of forward to reverse will tell you whether openclaw is shedding or gaining mass region-by-region. The symmetric-Pearson axis-134 on the same source reported `psChi2=9.1e10 asym=1.7e10`, implying forward ≈ 5.4e10, reverse ≈ 3.7e10, ratio ≈ 1.46. The axis-139 release post-refactor adds `neymanDirectionalSign(5.4e10, 3.7e10, tol=1e-12) = +1` and `neymanAsymmetry ≈ (5.4 - 3.7)/(5.4 + 3.7) ≈ 0.187` as the public surface for that distinction. The +1 sign is the actionable bit: openclaw's second-half distribution is thinner where the first-half had mass; openclaw is **losing coverage** of some regions across the day.

## The Cha-2007 cross-family asymmetry triangle

Now there are three distinct asymmetric-magnitude diagnostics on the live queue, all on the same KDE-smoothed half-vs-half tokens-per-day grid (K=257, Silverman bandwidth `0.9 × mad × n^(-1/5)`):

- **Axis-131 Jeffreys**: `J = KL(p||q) + KL(q||p)` with `asym = (KL(p||q) - KL(q||p)) / J`. Logarithmic-amplification regime; bounded by Pinsker `J ≥ 2 · TV^2`.
- **Axis-134 symmetric Pearson**: `psChi2 = N(p||q) + N(q||p)` with `asym = N(p||q) - N(q||p)`. Polynomial-tail-amplification regime; unbounded tails.
- **Axis-139 Neyman directional pair**: `{N(p||q), N(q||p)}`, `asymmetryNorm ∈ [0,1]`, `sign ∈ {-1, 0, +1}`. Same polynomial regime as axis-134 but with the direction made first-class.

The triangle closes via inequalities. Pinsker bounds give `2 · TV^2 ≤ J ≤ 2 · psChi2` (the latter only when `psChi2 < ∞`); Topsoe 2000 gives `J ≤ ln(2) · TV` only in the bounded regime. For the openclaw live-smoke values:

- Axis-131 `J = 8.135` with asym 0.674 (forward 5.1× reverse)
- Axis-134 `psChi2 = 9.1e10` with asym 1.7e10
- Axis-139 directional `{forward ≈ 5.4e10, reverse ≈ 3.7e10}`, sign +1, asymmetryNorm ≈ 0.187

The Jeffreys asymmetry magnitude (0.674) is much larger in normalized terms than the Neyman asymmetry (0.187) on the same source pair. This is the expected logarithmic-vs-polynomial signature: Jeffreys uses `log` to amplify the asymmetry between the two halves, and `log` is a non-linear amplifier that disproportionately stretches the small-probability differences. Polynomial chi-squared `(p-q)^2 / q` is closer to a linear-in-probability response, so the asymmetry magnitude is more conservative. The **direction agrees** across all three diagnostics (forward dominates), which is the actionable invariant the cross-family triangle was designed to expose.

## What the test count growth says about axis maturity

The test counts across the recent ladder rungs:

- v0.6.367 axis-124 quantile-Mahalanobis: +56
- v0.6.368 axis-125 PCA projection: +60
- v0.6.371 axis-128 Hellinger: +50
- v0.6.372 axis-129 triangular discrimination: +54
- v0.6.373 axis-130 Bhattacharyya: +63
- v0.6.374 axis-131 Jeffreys: +75
- v0.6.376 axis-133 max-divergence: +88
- v0.6.377 axis-134 symmetric Pearson: +51
- v0.6.378 axis-135 Clark: +54
- v0.6.379 axis-136 Taneja: +58 (full pass)
- v0.6.380 axis-137 Kumar-Johnson: +86
- v0.6.381 axis-138 Topsoe: ~80
- **v0.6.382 axis-139 Neyman: +94** (11673 → 11767)

The +94 delta is the second-largest in the recent window (only axis-133 max-divergence at +88 was comparable on a single release). The reason is clear from the feat commit stat: 996 lines of new source across `cli.ts` (+120), `dailytokenneymanchisquaredhalves.ts` (+788), and `format.ts` (+88) — the +788 axis-implementation file is one of the largest single-axis files in the codebase. Three named exports (`neymanForward`, `neymanReverse`, `neymanAsymmetry`) plus the helper `neymanSummand(p,q) = (p-q)^2/q` plus the post-release `neymanDirectionalSign` add up to four user-visible scalars on the directional-pair contract — each one needs its own test coverage matrix (zero-handling, near-equal-handling, large-tail-handling, KDE-smoothing-handling, identical-input-handling), and that explodes the test count budget.

The fact that the refactor commit (`368cbed`, +1 minute after release) adds yet another diagnostic helper — `neymanDirectionalSign` — without bumping the version is significant. It signals that the axis is **still being shaped** post-release rather than frozen — the directional-pair contract is being treated as an evolving surface rather than a closed deliverable. Compare against axis-128 Hellinger (v0.6.371) which shipped feat → test → release → refactor in a tight 4-commit quartet with no post-release diagnostic additions; or axis-131 Jeffreys (v0.6.374) which shipped a `jNorm` diagnostic in the refactor commit but never expanded beyond it. Axis-139's `neymanDirectionalSign` is the second post-release diagnostic addition pattern in the recent ladder window (axis-138 Topsoe also got `topsoeSaturation` + `TOPSOE_MAX_VALUE` in its refactor commit `c9c0af4`), suggesting the ladder is converging on a "diagnostic-rich first-class output" pattern rather than the "scalar-only" pattern of the early axes (126-130).

## Predictions

The next axes on the ladder (140+) should follow one of two trajectories under the axis-139 precedent:

- **P-139.A**: The next chi-squared-family axis (whether Pearson-asymmetric, Hellinger-asymmetric, or KL-asymmetric) ships with a directional-pair contract (forward + reverse + asymmetryNorm + directionalSign) following the axis-139 template. **Modal P 0.55**.
- **P-139.B**: The next axis is a non-divergence test (e.g., a moment-comparison test, a quantile-equality test, or a goodness-of-fit test) and breaks the chi-squared sequence at axis-139. **Modal P 0.30**.
- **P-139.C**: The ladder transitions back to the two-sample family (axes 115-122) for a missing entry — most likely a sign-test or rank-correlation variant. **Modal P 0.15**.

The release pace itself (axis-138 → 139 in 39 minutes, the tightest axis-to-axis interval in the recent window) suggests the ladder is in a high-velocity regime that won't sustain the +94-test-per-axis budget for many more releases — at the current pace, the test suite would cross 12,000 by v0.6.385. That ceiling is itself a soft constraint that will likely force the next 2-3 axes to be slimmer (~50-60 tests each) or to share infrastructure with axis-139 via the `neymanSummand` helper that was deliberately exposed for downstream tooling.

The single most actionable bit from the axis-139 release is `neymanDirectionalSign(forward, reverse, tol=1e-12) ∈ {-1, 0, +1}` — it is the first axis to ship a categorical-direction output on the f-divergence ladder, and that categorical output is exactly what makes asymmetry diagnostically usable rather than just numerically present. The axes 131-138 had asymmetry magnitudes everywhere; axis-139 is the first one where you can sort source-pairs by **which** side dominates without re-deriving it from raw forward/reverse pairs at the call site.
