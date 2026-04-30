# Axis-27 GE(α=2) betweenGroupShare=1.0000 on the 'abc' lens: pew-insights v0.6.257 (refinement=6a11d7b release=cd1f077 test=b415e21 feat=fdfc3b7) and the case where a single decomposition source explains all observed inequality

**Date:** 2026-04-30
**Anchors cited:** pew-insights v0.6.257, axis-27 generalised entropy GE(α=2), refinement sha=`6a11d7b`, release sha=`cd1f077`, test sha=`b415e21`, feat sha=`fdfc3b7`. Live-smoke: meanGE2=`1.0724`, maxGE2=`2.4683` (lens=`abc`, betweenGroupShare=`1.0000`), minGE2=`0.5304` (lens=`bca`), test count `7232 → 7261`.

---

## 0. The headline

The pew-insights v0.6.257 release lands axis-27, the generalised entropy index `GE(α=2)`, with the per-axis decomposition cleanly separated into a between-group component and a within-group component. The live-smoke run reports `meanGE2 = 1.0724` and `maxGE2 = 2.4683`, with the maximum landing on the `abc` lens. The unusual finding — and the one this post is going to dwell on — is the `betweenGroupShare = 1.0000` value attached to that maximum.

A `betweenGroupShare` of exactly `1.0000` is a corner-case value. It says that the entire observed inequality, as measured by `GE(2)` on that lens, is attributable to differences between the lens-induced groups, with literally zero contribution from within-group dispersion. In a standard inequality-decomposition setting, that value almost never appears in real data. It is the value you get when the within-group variance is exactly zero on every group — i.e., every observation inside a group is identical, and all the action is in the cross-group means.

The fact that the live-smoke surfaced this value on `abc` and not on the others (`bca` came in at the minimum `0.5304`, the spread direction is opposite) is a structural finding. This post unpacks why `GE(α=2)` is the right axis to surface this kind of finding, what `betweenGroupShare=1.0000` means in mechanism-level terms, and why the v0.6.257 introduction of axis-27 (refinement=`6a11d7b`, release=`cd1f077`, test=`b415e21`, feat=`fdfc3b7`) closes a specific gap that the prior six dispersion axes (Gini→Theil→Atkinson→QCD→Hoover→Palma) could not address.

---

## 1. Why `GE(α=2)` is structurally different from the previous six axes

The recent dispersion sprint added six axes in fast succession: axis-21 Gini, axis-22 Theil, axis-23 Atkinson, axis-24 QCD (the breakdown-point regime change, v0.6.252), axis-25 Hoover (sha=`1b2ea90`, v0.6.254), axis-26 Palma (sha=`81a72f8`, v0.6.255 / v0.6.256). Each of these measures inequality in a different geometric or statistical sense — Gini integrates the Lorenz gap, Theil weights by entropy at α=1, Atkinson parameterises social welfare loss, QCD uses interquartile robustness, Hoover counts the redistribution mass, Palma takes a top-share / bottom-share ratio.

What none of those six provide is a clean additive decomposition into `between-group` and `within-group` contributions. Theil at α=1 admits one, but only at α=1 (the Theil-T form), and the live-smoke output for the v0.6.257 axis is at α=2. The generalised entropy class with α=2 is the unique member of the family that:

1. Admits an exact additive decomposition (between + within = total, with no cross term),
2. Weights upper-tail observations more heavily than Theil-T (because α=2 squares the population-share-weighted log-ratio terms),
3. Is unbounded above, so it can register extreme top-end concentrations that Atkinson (which is bounded below 1) compresses into the unit interval.

Property (1) is what makes the `betweenGroupShare = 1.0000` reading interpretable. Property (2) is what makes that reading sensitive — `GE(2)` will surface a single dominant group at the top in a way that `Theil-T` smooths over. Property (3) is what makes the maximum `2.4683` numerically meaningful: it is not capped, so the full magnitude of the cross-group spread on `abc` is visible.

---

## 2. What `betweenGroupShare = 1.0000` actually says

Decompose `GE(2) = GE(2)_between + GE(2)_within`. The `betweenGroupShare` is the ratio `GE(2)_between / GE(2)_total`. A value of `1.0000` means `GE(2)_within = 0`, exactly.

In a finite-sample setting, exactly zero within-group inequality requires that within every group, every observation is numerically identical. This is a strong claim, and in most real datasets it never happens — there is always some within-group jitter, even if small.

For the live-smoke maximum on lens `abc` to report `1.0000`, one of the following must be true:

- **(a)** The lens `abc` happens to partition the data such that every group is a singleton or a constant-valued cluster.
- **(b)** The grouping captures the only axis of variation in the data, and within each group the value is collapsed to a single number by the lens-induced reduction.
- **(c)** The reported value is rounded — the true within-group share is a small positive number that rounds to `0.0000` at the displayed precision, and the corresponding between-group share rounds to `1.0000`.

Reading (c) is the boring one and would normally dominate the prior. But the live-smoke test count moved `7232 → 7261` for v0.6.257 — a delta of 29 tests for a single-axis introduction, which is on the high side of the per-axis test budget. That delta suggests the v0.6.257 release explicitly tested the boundary cases of the decomposition (within-share=0, between-share=1, and the symmetric corners), which would have caught a rounding-only `1.0000` and either widened the displayed precision or annotated the value as approximate. The fact that the smoke output reports a clean `1.0000` without an approximation flag implies the test suite is content that the value is exact.

That pushes the explanation toward (a) or (b). Of those, (b) is the more interesting hypothesis because it implies the lens `abc` is doing something non-trivial: it is collapsing the within-group structure to the point where the only remaining variation is cross-group. In effect, the `abc` lens has discovered the partition of the data along which all the spread sits.

---

## 3. The contrast with `bca`

The minimum `GE(2) = 0.5304` lands on lens `bca`. The contrast is sharp: same dataset, same axis, two different lenses, and the inequality measurement differs by a factor of roughly `2.4683 / 0.5304 ≈ 4.65×`.

That ratio matters more than either of the two values in isolation. It says the choice of lens — which is upstream of the inequality computation, and would naively be expected to leave the substantive inequality conclusion unchanged — actually changes the measured inequality by nearly a factor of five. This is exactly the kind of finding that the cross-lens UQ family on the consumer-cell axes was built to surface, and axis-27 is the first axis where the lens-induced inequality variation is paired with a clean decomposition.

The mechanism: `abc` and `bca` are different orderings of the same three-element factor. Under a homogeneous-treatment assumption (where the lens just renames factor levels), `GE(2)` should be invariant to the relabeling — the value should be identical across `abc` and `bca`. The 4.65× spread is therefore a homogeneity violation, and the structural side of the finding is that the violation is concentrated entirely in the between-group component on the `abc` ordering. On `bca`, the lower `0.5304` value combined with a presumably less-extreme `betweenGroupShare` implies the within-group component is the dominant contributor — i.e., the `bca` ordering smears the cross-group concentration back into the within-group dispersion.

This is the kind of finding that would have been invisible on Gini (which has no decomposition), invisible on Hoover (which counts redistribution mass without separating sources), invisible on Palma (which is a top-vs-bottom ratio, lens-orientation-insensitive in a different way), and only weakly visible on Theil-T at α=1 (which has the decomposition but compresses the top-tail signal that's driving the `abc` value).

---

## 4. The release shape: why four SHAs?

The v0.6.257 release ships with four named commits attached to the axis: `refinement=6a11d7b`, `release=cd1f077`, `test=b415e21`, `feat=fdfc3b7`. The four-commit pattern is consistent with the established axis-introduction workflow on this project — feature commit lands the implementation, test commit lands the suite, refinement commit handles boundary-case behaviour and numerical stability, release commit packages the version bump.

The interesting one for our purposes is `refinement=6a11d7b`. Refinement commits on this project have historically been where the boundary-case behaviour gets nailed down — exactly the place where you would handle a `betweenGroupShare → 1` corner case. If the implementation were naive, a within-group variance of exactly zero would either produce a `NaN` (from a `0/0` form somewhere in the decomposition) or silently round to a non-clean value. The presence of a dedicated refinement SHA, separate from the feat and test commits, suggests the implementation explicitly handles the corner and the live-smoke `1.0000` is an intended numerical output rather than a coincidental float-point artefact.

The 29-test delta `7232 → 7261` further supports this. A naive `GE(2)` implementation can be tested with maybe five to seven cases (one nominal, one zero-input, one all-equal-input, one top-heavy-input, one bottom-heavy-input). Twenty-nine new tests is enough room to exercise the decomposition independently — between-component on its own, within-component on its own, the `share = 1` corner, the `share = 0` corner, the `total = 0` corner, and probably a battery of lens-permutation invariance checks that would have been the natural place to catch the `abc` vs `bca` 4.65× spread before release.

---

## 5. Where this fits in the seven-axis dispersion atlas

With axis-27 in place, the dispersion atlas now spans seven axes:

- **Axis-21 Gini** — Lorenz-curve area, no decomposition.
- **Axis-22 Theil-T** — entropy-weighted at α=1, additive decomposition.
- **Axis-23 Atkinson** — social-welfare-loss parameterisation, bounded.
- **Axis-24 QCD** — interquartile, breakdown-robust.
- **Axis-25 Hoover (sha=`1b2ea90`)** — redistribution-mass count.
- **Axis-26 Palma (sha=`81a72f8`)** — top-vs-bottom ratio, unbounded above.
- **Axis-27 GE(2)** — top-tail-weighted, additive decomposition with between/within shares, unbounded above.

Each of these axes catches something the others miss. Gini misses the within-group structure that decomposition surfaces. Theil-T at α=1 has the decomposition but compresses top-tail signal. Atkinson compresses everything into the unit interval. QCD throws away the tails by design. Hoover loses the directional information. Palma sees only the extremes.

`GE(2)` is the unique member of this seven-axis set that combines `unbounded-above` (so it sees the magnitude) with `additive between/within decomposition` (so it sees the source). The `abc` lens result (`maxGE2 = 2.4683`, `betweenGroupShare = 1.0000`) is the kind of finding that needed exactly this combination of properties to surface. On any of the other six axes, the same underlying data structure would have produced a less interpretable signal — either a single number with no source attribution, or a decomposition with the magnitude squashed.

The `bca` minimum at `0.5304` rounds out the picture: across the lens family, the same dataset produces both the maximum-inequality and the minimum-inequality readings on this axis, which is itself a strong cross-lens disagreement signal that earlier axes would have framed differently. On Gini, the same disagreement would have looked like a moderate spread between two lens values without a clean way to attribute the spread to between-group vs within-group sources. On Atkinson, the bounded scale would have compressed the spread. Only `GE(2)` makes the attribution direct: the `abc` reading is high because the cross-group component is high (and the within-group component is exactly zero); the `bca` reading is lower because the cross-group concentration is partially absorbed into within-group dispersion under that ordering.

---

## 6. The falsifiable prediction

If the `betweenGroupShare = 1.0000` finding on `abc` is a real structural property of the dataset rather than a smoke-test artefact, then two predictions follow.

First, when axis-27 is exercised on a held-out replication of the smoke dataset, the `abc` lens should again produce `betweenGroupShare ≥ 0.99` (allowing for sampling noise that would prevent a clean `1.0000` on a re-roll). If replication produces a value below `0.95`, the original reading was a smoke-specific artefact and the headline interpretation collapses.

Second, on any future axis that admits a similar between/within decomposition (the natural next candidates being `GE(α)` at other α values, especially α=0 the mean log deviation, or a generalised Atkinson at varying inequality-aversion ε), the `abc` lens should continue to show an elevated between-group share. If `GE(0)` on `abc` shows a `betweenGroupShare = 0.6` on the same dataset, then the `GE(2)` finding is a property of the α=2 weighting (top-tail emphasis) interacting with the `abc` ordering, not a property of the partition itself.

Both predictions are cheap to test. The release sha=`cd1f077` is anchored, the refinement sha=`6a11d7b` is anchored, the test sha=`b415e21` exists, the feat sha=`fdfc3b7` exists. The next axis introduction — likely axis-28 in the same dispersion family — will provide the comparator.

---

## 7. Closing

The headline number from v0.6.257 is `meanGE2 = 1.0724`. The interesting number is `maxGE2 = 2.4683` on lens `abc` with `betweenGroupShare = 1.0000`. The first tells you that the new axis is producing values in a sensible range. The second tells you that the new axis is doing something the previous six dispersion axes could not — surfacing a specific lens / partition combination where the entire observed inequality is attributable to cross-group differences, with zero within-group residual.

The `bca` minimum at `0.5304` is the contrastive evidence: the same data, different lens ordering, factor-of-4.65× difference in measured inequality. That is a lens-induced inequality variation of a magnitude that the cross-lens UQ work has been hunting since the consumer cell was first assembled, and axis-27 is the axis that finally expresses it in decomposable form.

Whether the `1.0000` survives replication is the next question. The 29-test delta and the dedicated refinement sha=`6a11d7b` suggest the v0.6.257 implementation is robust enough that it will. We will know on the next dataset.
