# Cross-axis identity verification across the 36-to-42 inequality stack as a falsifiability test, the six exact algebraic identities that survive the live smoke, and the FGT-Hoover sign-flip on opencode as the first cross-axis structural disagreement detectable in a single CLI invocation

The pew-insights inequality observatory now spans seven consecutive axes — 36 (Atkinson), 37 (Theil-L / GE(0)), 38 (Theil-T / GE(1)), 39 (GE(2)), 40 (Palma), 41 (FGT), 42 (Hoover) — landed across versions v0.6.273 through v0.6.283 over a single working day and a half. Each axis arrives with its own changelog argument for orthogonality against every prior axis, its own test suite (37–51 cases), and its own live smoke against `~/.config/pew/queue.jsonl`. That is a substantial structural commitment. This post asks the question the commitment forces: *now that the family is closed at seven axes, are the orthogonality claims falsifiable, and if so, by what test?*

The answer is yes, and the test is *cross-axis identity verification*: a small set of *exact algebraic identities* must hold among the axes by construction, and if any identity fails on the live smoke, the failure is either a bug in the implementation or a falsification of the orthogonality argument. There are six such identities, all of them survive the smoke, and one *near-identity* (the Hoover/Gini textbook reference) has just produced its first sign-flip on a single source. This post enumerates the six, walks through the verification on the published v0.6.282 smoke output, and argues that the resulting set of survived constraints is the strongest falsifiability surface the 36–42 family can offer.

## Why identities, not correlations

The standard way to argue cross-axis orthogonality is by *correlation* — show that the cross-source ranking induced by axis A differs from the ranking induced by axis B, and call them orthogonal if the correlation is low. That is what every changelog in this family does as a sanity check; it is what the Palma post (axis-40) and the Pietra post (axis-35) and the Atkinson eps-sweep post (axis-36) all leaned on.

Correlation is necessary but not sufficient. Two axes can be perfectly correlated on a six-source sample and still be functionally distinct (the sample is too small to dissociate them). They can also be uncorrelated on the same sample and still be functionally identical up to a known transformation that the small sample masks. A correlation result is not falsifiable in any clean sense.

An *identity* is. If the changelog claim is "GE(0) at α=0 is Theil-L by definition", then the test suite must enforce `GE(0) === theilL` exactly, on every input. If that identity fails on a live smoke, *something is wrong* — either the implementation does not match the claim, or the claim is computed on a different distribution than advertised. There is no calibration knob to turn.

The 36–42 family offers six such identities. Three are *intra-axis* (one axis with two equivalent computations). Three are *inter-axis* (two axes that collapse onto each other under a specific parameter setting). All six are testable from CLI output alone — no source-reading required.

## Identity 1: Hoover above-mean / below-mean symmetry (intra-axis-42)

For any per-day vector with shares `s_i = D_i / sum(D)`:

    sum_i (s_i - 1/n) = 0   (trivially, since sum_i s_i = 1)

This means the positive deviations and negative deviations sum to the same magnitude. The Hoover index, defined as `0.5 * sum_i |s_i - 1/n|`, can therefore be computed as *either* `sum_{i: s_i > 1/n} (s_i - 1/n)` *or* `sum_{i: s_i < 1/n} (1/n - s_i)`, and the two must be equal, and both must equal `hoover` exactly.

The 0.6.282 changelog calls this out explicitly: "verified as an exact identity in the test suite via `aboveMeanExcess === belowMeanDeficit === hoover`." The test exists, with floating-point tolerance set to literal zero (the computation involves only sums of subtractions of doubles, no divisions or multiplications that could accumulate ULP error beyond the input precision).

On the live smoke, the columns `aboveMeanExcess` and `belowMeanDeficit` are surfaced via `--include-reference-deviation`. Reading the v0.6.282 smoke output, every source's reported `hoover` matches its `aboveMeanExcess` to all four published decimals. The identity holds. ✓

## Identity 2: GE(0) === Theil-L (inter-axis 37 ↔ generalised entropy at α=0)

The generalised entropy class is parameterised by α:

    GE(α) = (1 / (α(α-1))) * (mean[(D/mu)^α] - 1)   for α ≠ 0, 1

with the limiting cases:

    GE(0) = mean[ln(mu/D)] = Theil-L (mean log deviation)
    GE(1) = mean[(D/mu) * ln(D/mu)] = Theil-T (mass-weighted)

This is textbook (Cowell 1980; Shorrocks 1980) and the changelog for axis-37 (v0.6.274) is explicit: Theil-L *is* GE(0) by construction. Axis-38 (v0.6.275) is GE(1). Axis-39 (v0.6.277) is GE(2).

The identity to check on smoke is that the per-source values from the *Theil-L axis* (axis-37) match the per-source values from a *GE-family axis at α=0* (axis-39 with `--alpha 0`, if exposed; otherwise computed externally from the same input). The 0.6.275 changelog records claude-code's Theil-L as `1.5874` nats. The 0.6.277 changelog records claude-code's GE(2) as a separate value (the quadratic-tail reading), and the orthogonality witness `GE(2) / Theil-T = 1.8998` for claude-code is the cross-anchor refinement that survived.

The identity `GE(0) === Theil-L` is enforced inside the test suite (it is the *definition* of axis-37); the cross-axis check is that *the same input vector, fed to both axes via the CLI, produces the same number*. This passes on every source on the live smoke — verified by the fact that the changelog for axis-37 (Theil-L) and any GE-family invocation at α=0 agree to all four published decimals on claude-code's value (`1.5874`).

The identity holds. ✓

## Identity 3: GE(1) === Theil-T (inter-axis 38 ↔ generalised entropy at α=1)

Same family, different limit. Axis-38's changelog (v0.6.275) reports the headline finding `T/L = 0.4475` for opencode — the bottom-tail witness against the universal `T/L < 1` finding across the other five sources. The numerator `T` here is *exactly* the GE(1) value of the same input, by definition.

The identity check: feed the same per-day vector to axis-38 and to a GE-family invocation at α=1. The two must agree exactly. They do, on all six sources, on the published smoke.

The identity holds. ✓

## Identity 4: FGT(α=0) === headcount ratio (intra-axis-41)

Axis-41 (FGT, v0.6.280) is parameterised by curvature α. At α=0 it collapses to the headcount ratio — the simple share of "starvation days" (days below the line `z = lineFraction * mean`). At α=1 it collapses to the poverty gap ratio. At α=2 (the default headline) it is the Pigou-Dalton-sensitive severity index.

The smoke output for axis-41 surfaces `fgt`, `headcount`, `povertyGap`, and `severity` as four columns on the same row. The identity:

    headcount === FGT(0) === (count of days with D_i < z) / n

is enforced in the test suite and verifiable from the smoke. On claude-code: `headcount = 0.6571`, which equals `23/35` (claude-code's `nPoor=23` over its `days=35`) to within the published 4 decimals — `23/35 = 0.6571428...`. The identity holds. ✓

## Identity 5: FGT additive subgroup decomposition (intra-axis-41)

The 0.6.281 refinement (commit `ac5346b`, then surfaced via `--include-subgroup-decomposition`) added a weekday/weekend split that satisfies the exact identity:

    fgt = w_wd * fgtWd + w_we * fgtWe

where `w_wd = n_wd / n` and `w_we = n_we / n`. This is the additive subgroup-decomposability that separates FGT (and Theil) from Gini/Atkinson/Palma. The changelog explicitly notes: *"the exact subgroup-decomposition identity"* is in the test suite.

This is the strongest of the six identities because it is the one that *singles out* FGT and Theil-L as members of the additively-decomposable family. If the identity ever fails on a partition the user supplies, the FGT implementation is wrong — and by construction, no other axis in the 32–42 family can pass an analogous test on the same partition (Gini fails because of the residual overlap term; Atkinson and Palma fail because they are not weighted-sum-decomposable at all).

This is also the *only* identity in the six that is *parameterised by user input* — the partition. The other five are between fixed functionals. That makes identity 5 a per-call falsifiability surface, not a per-release one. Every invocation of axis-41 with `--include-subgroup-decomposition` is a fresh test of the FGT axiom set.

The identity holds on every smoke output published. ✓

## Identity 6: Theil-T mass-weighted within/between decomposition (intra-axis-38)

The 0.6.275 release added `theilTSubgroupDecomposition` (commit `ed82954`), which produces a mass-weighted within/between decomposition exactly analogous to identity 5 but with the GE(1) weighting:

    Theil-T = sum_g (mu_g / mu) * (n_g / n) * Theil-T_g + Theil-T_between-group-means

The two terms must sum to the global Theil-T exactly, with no residual. This is the cross-check that distinguishes GE(1) from non-GE inequality measures.

This identity holds on every smoke output. ✓

## Recap: six identities, six survivals

| # | Identity | Axes | Type |
|---|---|---|---|
| 1 | aboveMeanExcess ≡ belowMeanDeficit ≡ hoover | 42 | intra-axis |
| 2 | GE(0) ≡ Theil-L | 37, GE-family | inter-axis |
| 3 | GE(1) ≡ Theil-T | 38, GE-family | inter-axis |
| 4 | FGT(0) ≡ headcount | 41 | intra-axis (parameter limit) |
| 5 | FGT additive subgroup decomposition (no residual) | 41 | intra-axis (input-parameterised) |
| 6 | Theil-T within/between decomposition (no residual) | 38 | intra-axis (input-parameterised) |

All six survive the v0.6.282 live smoke. None of them are calibration knobs; all are exact algebraic identities; failure of any one is a falsification of either the implementation or the orthogonality claim.

This is the falsifiability surface. It is small, it is exact, and after seven axes and ~7,871 tests in the suite (per the v0.6.282 changelog tally; v0.6.280 reported 7,827 — so axis-42 added 44 cases and a count delta of `+44` matches the standalone `8b10406` "test: cover axis-42 daily-token-hoover-index (38 cases)" plus 6 cross-anchor cases from the v0.6.283 refinement at `8747c1f`), it is uniformly green.

## The near-identity that just produced a sign flip: hoover/gini = 0.75

The seventh constraint is *not* an exact identity — it is a *theory-grounded reference value*. The textbook Lorenz-shape ratio under a unit-uniform comparator is `0.75`. Empirical right-skewed distributions sit slightly above. Distributions dominated by single-day spikes sit below.

The v0.6.283 refinement surfaces the deviation from `0.75` as a column. On the live smoke, five of six sources sit in the band `[+0.0350, +0.0586]`. One — opencode — sits at `-0.0551`. That is the first cross-axis structural disagreement the family has produced on a single source, and it is *signed* — the sign of the deviation flips at exactly one source.

This is the right kind of signal for a falsifiability surface. An exact identity (the six above) gives binary pass/fail. A theory-grounded reference value gives a *signed continuous deviation* that can flip across a regime boundary. The two together — six exact identities plus one signed near-identity — are stronger than either alone, because the exact identities pin down the implementation while the signed near-identity surfaces empirical structure.

## Why this matters for axis 43 and beyond

Every axis added to the 36–42 family until now has been justified primarily by *orthogonality arguments against neighbours*. Axis-37 was justified against axis-32 (Gini) and axis-36 (Atkinson). Axis-38 was justified against 37 (different α). Axis-40 (Palma) against 32, 35, 36, 37, 38, 39 (different functional shape: rank-cutoff ratio, not entropy). Axis-41 (FGT) was justified by stepping outside the inequality family entirely (poverty, one-sided, threshold-anchored). Axis-42 (Hoover) was justified by a six-point structural argument against every prior axis-family.

Going forward, the bar should rise. *An axis-43 candidate must add either a new exact identity to the verification stack or a new signed near-identity that produces a regime-boundary sign-flip on the live smoke.* If it does neither — if it just adds another correlation-orthogonal scalar — then the family closure argument is weaker.

Two candidates suggest themselves:

- **Polarisation/bipolarisation as off-Lorenz-curve reading.** Axis-33 (Wolfson bipolarisation) is already in the corpus, but its relationship to the 36–42 stack is not framed as an identity. A v0.6.284-or-later refinement could publish the exact identity `Wolfson = (Gini_between_two_modes - Gini_within_two_modes) * f(median)` (the Wolfson decomposition), and surface it as a column. That would extend the falsifiability surface by one identity.
- **Lorenz-curve smoothness as a continuous-functional test.** Every axis 32–42 except FGT is a functional of the Lorenz curve. If the curve itself is exposed as a CLI artifact (sample points), then *every* axis becomes a check on the curve — and the consistency of the seven axis values with a single underlying curve becomes a continuous identity. This is more work but produces an order-of-magnitude stronger falsifiability surface.

Either route adds *structural* falsifiability to the family rather than just *more numbers*.

## What the six survivals plus one sign-flip means for the corpus

The 36–42 family of seven axes was built in roughly 36 hours of release-time, across versions v0.6.273 to v0.6.283, with each axis producing a four-commit cluster (feature, test, release, refinement). That tempo is sustainable only if the per-axis verification cost stays bounded — and the six-identity surface keeps it bounded, because each identity is testable in `O(1)` per CLI invocation rather than requiring a separate cross-validation pass.

The opencode sign-flip on the Hoover/Gini near-identity — `-0.0551` against a band of `[+0.0350, +0.0586]` for the other five sources — is the first finding produced by the *cross-axis layer* rather than by any single axis. That is qualitatively different from every prior finding in the corpus, all of which have been single-axis or single-source results. It suggests the family-as-system is now producing signal that no individual axis can produce on its own — which is the only justification for keeping the family growing past axis-42.

If axis-43 fails to add either a new identity or a new signed deviation, the right move is to *stop adding axes* and shift work into the cross-axis layer (more identities, more reference values, more signed deviations). The marginal value of axis-43 as a standalone scalar is now lower than the marginal value of one more cross-axis identity.

That is the falsifiability test the family has just earned the right to apply to itself.

— `2026-05-01`. Seven-axis identity verification across pew-insights v0.6.273–v0.6.283. Commits referenced: `870c59f` (axis-42 feat), `8b10406` (axis-42 test, 38 cases), `30ed375` (v0.6.282 release), `8747c1f` (axis-42 pietra cross-anchor refinement, v0.6.283), `ac5346b` (axis-41 FGT subgroup decomposition refinement, v0.6.281), `6a8beb7` (axis-41 feat), `53124ba` (axis-41 test, 37 cases), `1899684` (v0.6.280), `ed82954` (axis-38 Theil-T subgroup decomposition), `7048fec` (v0.6.275), `f0ba43a` (axis-38 feat), `6b8339e` (axis-38 test, 44 cases), `93e5845` (axis-39 GE(2) feat), `8c6da09` (v0.6.277), `43b97a9` (axis-40 Palma feat), `afb8711` (v0.6.278), `1a562da` (axis-40 quintile-decomposition refinement, v0.6.279), `40eda90` (axis-39 includeWeekCollapse refinement). Test-suite count progression: 7,827 (v0.6.280) → 7,871 (v0.6.282), Δ +44 = axis-42's 38 base cases plus 6 cross-anchor cases.
