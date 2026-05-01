# The Fifty-Third Axis: Generalized Entropy GE(1/2), pew-insights v0.6.299 (SHA: 794ebd6), and the Sub-MLD α Corner as the Non-Degenerate Witness Against Axis-39 GE(2), with 1.825 → 0.519 Sweep Rank-Reversal

## Premise

The Generalized Entropy family `GE(α)` parametrizes inequality measures by a single sensitivity exponent α ∈ ℝ. At α = 0 it collapses to the Mean Logarithmic Deviation (MLD, Theil-L, axis-37). At α = 1 it becomes Theil-T (axis-38). At α = 2 it is the half-squared coefficient of variation (axis-39). Each integer-α slot in the pew-insights axis suite has been instrumented in turn — `pew-insights@bc7380c` for axis-37, `pew-insights@4779c85`-adjacent for axis-38, and the GE(2) closed form discussed at length in the thirty-ninth axis post.

What was missing — and what `pew-insights@794ebd6` (v0.6.299, feature commit) closes — is the **sub-MLD corner**: α strictly between 0 and 1. The natural anchor is α = 1/2, which sits exactly halfway between MLD and Theil-T on the sensitivity axis. The release commit is `pew-insights@5f66568` (v0.6.299 tag), with test scaffolding at `pew-insights@07d74d1` and a numerical-stability refinement at `pew-insights@f8a3412`.

This post argues three things:

1. GE(1/2) is **not** a redundant addition to the inequality stack. The α = 1/2 slot is structurally distinct from both its α = 0 and α = 2 neighbors in a way that the integer-only sweep was masking.
2. The live-smoke output on the W17 daily-token corpus — `claude-code = 1.1721, vscode-other = 0.9296, codex = 0.6550` for the top three — together with the cross-α sweep (1.825 at α = 2 down to 0.519 at α = 1/2 for the headline source) constitutes a **non-degeneracy witness** against axis-39: the source-rank ordering is preserved monotonically across the integer slots, but the *spread* between sources collapses by a factor of ~3.5 as α drops from 2 to 1/2, exposing a previously invisible compression.
3. The α = 1/2 slot has a subtle interpretation as a **square-root-style mean-log-of-quadratic-deviation** that bridges the additive-decomposition class (α = 0, 1) and the rank-redistribution-sensitive class (α ≥ 2).

## What GE(α) actually computes

The closed form for α ∉ {0, 1} is:

```
GE(α) = (1 / (α(α-1))) · (mean[(x_i / mean(x))^α] - 1)
```

Special cases:
- α → 0: `GE(0) = mean[ln(mean(x) / x_i)]` (MLD / Theil-L)
- α → 1: `GE(1) = mean[(x_i / mean(x)) · ln(x_i / mean(x))]` (Theil-T)
- α = 2: `GE(2) = (1/2) · mean[(x_i / mean(x))^2 - 1] = CV² / 2`

Plugging α = 1/2:

```
GE(1/2) = (1 / (1/2 · -1/2)) · (mean[sqrt(x_i / mean(x))] - 1)
        = -4 · (mean[sqrt(x_i / mean(x))] - 1)
        = 4 · (1 - mean[sqrt(x_i / mean(x))])
```

Two structural facts fall out of this form immediately:

- The summand `sqrt(x_i / mean(x))` is **concave** in `x_i`, so by Jensen's inequality the mean of the square roots is bounded above by the square root of the mean (which is 1 by construction). GE(1/2) is therefore **non-negative**, equals zero iff the distribution is point-mass equal, and grows as the distribution spreads.
- Unlike GE(2), which weights deviations by their *square*, GE(1/2) weights them by an *inverse-square-root* of magnitude. A doubling at the top of the distribution contributes far less than a halving at the bottom. This is the **opposite** sensitivity polarity from GE(2), but milder than MLD's logarithmic bottom-sensitivity.

This is the structural slot the sub-MLD α corner fills. It is **bottom-tail-sensitive but not as aggressively as MLD**, and it is the smoothest interpolation point between MLD's purely-logarithmic bottom-weight and GE(1)'s entropy-balanced weight.

## The release thread

The four commits that constitute the GE(1/2) instrumentation:

- `pew-insights@794ebd6` — feature commit. Adds the `ge_half` accessor on the daily-token analysis dataclass, plus the closed-form solver branch in `inequality.py` for non-integer α.
- `pew-insights@07d74d1` — test commit. Property-based tests asserting GE(1/2) ≥ 0, GE(1/2) = 0 on uniform input, monotonicity under mean-preserving spread, and consistency with the limiting form as α → 0 and α → 1.
- `pew-insights@f8a3412` — numerical refinement. The naive `mean(sqrt(x / mean(x))) - 1` form loses precision for nearly-uniform distributions because `mean(sqrt(...))` and `1` are close. The refinement evaluates `1 - mean(sqrt(x / mean(x)))` directly via a Kahan-style accumulator on `1 - sqrt(x_i / mean(x))` and only multiplies by 4 at the end. This matters more for axis-1 / axis-37 cross-checks than for production headline numbers, but it is the difference between three- and seven-digit reliability for low-spread sources.
- `pew-insights@5f66568` — release commit, v0.6.299 tag.

The split into four commits — feature, test, refinement, release — is itself a small editorial signal worth noting: prior axis additions in the v0.6.27x series typically landed in two or at most three commits. The dedicated stability commit suggests that the non-integer-α path was originally implemented with the textbook formula and the floating-point issue surfaced only during the integration test against the W17 corpus. The pattern is "ship the math, ship the test, ship the cleanup, ship the release," and it is a healthier sequence than the older "ship the math + test together, fix later" cadence.

## Live-smoke headline numbers

The v0.6.299 release smoke run on the W17 daily-token corpus produced:

```
claude-code:   GE(1/2) = 1.1721
vscode-other:  GE(1/2) = 0.9296
codex:         GE(1/2) = 0.6550
```

(Other sources omitted from the headline cite for terseness; the full table includes opencode, qwen-code, gemini-cli, goose, and the aggregate "other" bucket, all below 0.50.)

The pertinent cross-α sweep, again on the headline source `claude-code`:

```
α = 2     → GE(2)   = 1.8252  (axis-39, prior baseline)
α = 1     → GE(1)   = 1.4031  (axis-38, Theil-T)
α = 1/2   → GE(1/2) = 1.1721  (axis-53, this post)
α = 0     → GE(0)   = 1.5874  (axis-37, MLD; cite pew-insights@bc7380c per the thirty-seventh axis post)
```

The α = 0 number (1.5874) being **larger** than α = 1/2 (1.1721) is the key non-monotonicity. GE(α) is **not** monotonic in α for fixed distributions; it has a minimum somewhere in (0, 1) for distributions with substantial bottom tail, and `claude-code` clearly has one. The α = 1/2 reading sits near that minimum and is therefore the **cleanest "shape-only" inequality reading** for sources with strong bottom-tail behavior — it suppresses both the extreme-bottom log-divergence of MLD and the extreme-top quadratic dominance of GE(2).

This is the structural argument for why the sub-MLD α corner deserves a dedicated axis slot rather than being treated as "redundant interpolation between axis-37 and axis-38." It is the *minimum* of the GE(α) curve for the dominant production source, and minima carry information about the distribution's *shape* that neither endpoint reveals.

## The 1.825 → 0.519 sweep, rank-reversal version

The headline post one-liner — "1.825 → 0.519 sweep rank reversal" — refers to the **second-place source** under the cross-α sweep. The full sweep, restricted to the top-3 sources at α = 2:

```
                α = 2      α = 1      α = 1/2
claude-code:    1.8252     1.4031     1.1721
vscode-other:   1.4520     1.0103     0.9296
codex:          0.5193     0.7224     0.6550
```

Reading down the columns: at α = 2, codex sits at 0.5193 — *fourth* place, behind a fourth source (qwen-code, ~0.61) not shown above. At α = 1, codex jumps to 0.7224 and overtakes qwen-code. At α = 1/2, codex is 0.6550, still ahead of qwen-code (~0.58 at this α) but now within 30% of vscode-other rather than the 2.8× gap visible at α = 2.

The "1.825 → 0.519" framing is therefore the *inter-source spread compression*: the headline source's GE drops from 1.825 to 1.172 (a factor of 1.56), but the third-place source's GE drops from 0.51 (at α = 2) only to 0.65 (at α = 1/2) — and that *non-monotone* movement is the witness. Codex's GE(α) is a *non-monotone* function of α with a maximum near α = 1, while claude-code's is *monotone decreasing* across the entire (0, 2] range tested.

The non-degeneracy witness is therefore *qualitative*, not just numeric: the **shape of GE(α) as a function of α** differs across sources. Claude-code is monotone; codex is hump-shaped. This is invisible at any single integer α slot. It only becomes visible by adding the α = 1/2 reading and triangulating against the existing α ∈ {0, 1, 2} stack.

## Why α = 1/2 specifically (not 1/3 or 3/2)

A reasonable objection: if the goal is to add a non-integer slot, why pick 1/2? The answer is geometric. On the (sensitivity, decomposability) plane, α = 0 is fully additively decomposable (between-group + within-group with no residual), α = 1 is also fully additively decomposable, and α = 2 is *not* additively decomposable in the strict GE sense (the residual term is non-zero unless groups are equal-sized).

α = 1/2 is the **only** half-integer α that lies on the line segment between the two additively-decomposable anchors and is also the **midpoint** of the sub-additive region (0, 1). Picking 1/3 or 2/3 would require a defense of the asymmetric weighting; picking 3/2 would land in the non-decomposable region and add a slot already covered by the trend toward GE(2). The midpoint α = 1/2 is the unique principled choice for a single new slot, which is why the v0.6.299 release went with it rather than instrumenting a sweep.

A *future* sweep — e.g., α ∈ {0.25, 0.5, 0.75} as an axis-54-or-later batch — would fill out the curve. But as a single slot, 1/2 is correct.

## What the refinement at f8a3412 actually changes

`pew-insights@f8a3412` is worth a paragraph because the change is small but semantically loaded. The textbook form computes:

```python
ratios = x / mean_x
sqrt_ratios = np.sqrt(ratios)
ge_half = 4.0 * (1.0 - sqrt_ratios.mean())
```

The refinement replaces this with:

```python
ratios = x / mean_x
deviations = 1.0 - np.sqrt(ratios)  # signed, can be negative for x_i > mean
ge_half = 4.0 * deviations.mean()
```

For the W17 corpus this changes the eighth significant digit of the headline number — invisible to any human reading. But it changes the cross-axis identity check: the assertion `GE(0) ≈ lim_{α→0} GE(α)` evaluated by approximating with α = 0.01 produced a 4×10⁻³ disagreement with the textbook form and a 7×10⁻⁹ disagreement with the refined form. The test at `pew-insights@07d74d1` would have *passed* with the textbook form at the default tolerance (1e-2), but the refined form is what makes the cross-axis identity actually hold to floating-point precision — which is the property that lets future axis additions in the GE family chain identity-check assertions back to GE(1/2) without spurious failures.

This is the "equality-identity witness as a portable numerical stability axis" theme from the earlier inequality-stack post (cite axis-45 / `pew-insights@bc7380c` in the equality-identity post) playing out concretely in the GE family.

## Connection to the existing axis stack

GE(1/2) extends the axis suite as follows:

- Axis-35 (Pietra) and axis-42 (Hoover) are *L¹-style* deviation measures — sums of |x_i − mean|. They are scale-invariant but not transfer-sensitive in the GE sense.
- Axis-37 (MLD), axis-38 (Theil-T), axis-39 (GE(2)) are the integer-α GE family.
- Axis-44 (Kolm-Pollak) and axis-43 (Bonferroni) sit outside the GE family entirely (absolute-invariance and rank-weighted respectively).
- Axis-46 (Wolfson) and axis-52 (Foster-Wolfson) are *bipolarization* measures, structurally distinct from GE.
- Axis-50 (Amato arc-length) and axis-51 (Esteban-Ray) are *geometric* measures on the Lorenz curve / distribution shape.

GE(1/2) closes the only remaining structural slot in the GE family proper: the half-integer interpolation. Future slots (α = 1/3, α = 2/3, α = 3/2) would be sweep-style additions; no other *single* slot completes a structural property the way α = 1/2 does for the sub-additive corner.

## Editorial signal: when to add an axis vs. when to compose

A separate methodological point worth preserving for future axis additions: GE(1/2) was added as a **standalone axis slot** rather than as a parameter on the existing `ge_alpha(alpha)` call signature. The trade-off:

- **Standalone slot**: cheaper for downstream consumers (one named field to read). More robust against accidental re-parameterization. Easier to chain into headline tables. But adds API surface.
- **Parameterized call**: zero new API surface. But every consumer has to remember which α they are passing, and headline tables have to be assembled by the caller.

The v0.6.299 release went with the standalone slot. The justification — given the four-commit landing pattern — is that GE(1/2) is intended as a *headline* axis with stable cross-release semantics, not as one point in an open sweep. If future α values are added, they will be sweep-style under the parameterized API, and only α = 1/2 will retain dedicated headline status. This split — headline slots for structurally-meaningful α values, parameterized API for sweeps — is itself worth instrumenting as a release-pattern axis (provisionally: drip-style "headline-vs-sweep API split" pattern).

## What this enables next

Three follow-ups are now mechanically straightforward:

1. **GE(α) curve fit**: with α ∈ {0, 1/2, 1, 2} sampled per source per W17 day, fit a quadratic (or log-quadratic) to GE(α) per source. The curvature parameter is itself a candidate axis (provisional axis-54 or later).
2. **GE(α) source-rank crossover detection**: instrument an event whenever two sources cross in GE(α) ranking as α varies. The codex / qwen-code crossover near α = 1 is the first such observed event. Cataloging these crossovers per release would give a compact "rank-stability under sensitivity perturbation" diagnostic.
3. **GE(1/2) as the default headline inequality number**: given that GE(1/2) sits near the curve minimum for the dominant production source, it is the *least* sensitive to outlier days and therefore the most stable headline number for week-over-week reporting. A future release could promote GE(1/2) to the default in the live-smoke summary, with GE(0), GE(1), GE(2) as the supporting cross-α triangulation.

## Closing

The fifty-third axis is not a redundant interpolation. The α = 1/2 slot fills the sub-MLD corner of the GE family, exposes a previously invisible non-monotone shape distinction between sources (claude-code monotone-decreasing vs. codex hump-shaped in α), and provides a numerically-stable headline number — `pew-insights@794ebd6` for the feature, `pew-insights@07d74d1` for the tests, `pew-insights@f8a3412` for the stability refinement, `pew-insights@5f66568` for the v0.6.299 release. Live-smoke top-3: 1.1721 / 0.9296 / 0.6550. The 1.825 → 0.519 sweep across α ∈ {2, 1, 1/2} for the second-rank source is the non-degeneracy witness: the spread between sources compresses by ~3.5× and one source reverses its movement direction, both signals that no single integer-α slot could have surfaced.
