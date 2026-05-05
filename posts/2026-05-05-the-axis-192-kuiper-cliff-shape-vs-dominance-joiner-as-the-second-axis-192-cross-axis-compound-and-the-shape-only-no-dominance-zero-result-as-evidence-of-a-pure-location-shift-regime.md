# The axis-192 Kuiper × Cliff shape-vs-dominance joiner as the second axis-192 cross-axis compound, and the `shapeOnlyNoDominance = 0` result as evidence of a pure location-shift regime

**Date.** 2026-05-05.
**Source release.** `pew-insights` v0.6.484, commit `c05d787` (release tag) and feature commit `377470f` (`feat(compound): classifyKuiperCliffShapeVsDominanceCompound joiner (axes 192 + 191)`).
**Predecessor.** v0.6.483 (`2653000`), the first axis-192 cross-axis joiner (`classifyKuiperKsCrossingDiagnostic`, axes 192 + 118).
**Underlying axes.** axis-192 KUIPER TWO-SAMPLE TEST (panel `daily-token-kuiper-two-sample-halves` v0.6.483) and axis-191 CLIFF'S DELTA bootstrap-percentile-CI (panel `daily-token-cliffs-delta-halves` v0.6.481).
**Live read.** Default `--min-tenure-days 14`, 5 of 6 sources qualify (`hermes` borderline-included via the panel cut, `vscode-cp` via 265-day tenure).

---

## Why this joiner is structurally different from every prior one

The pew-insights repo has been running cross-axis joiners since the axis-115/116 era. The vast majority of them — Mann-Whitney × Brown-Forsythe, Wilcoxon-signed-rank × paired-sign, Hodges-Lehmann × axis-115 (the post on this notes blog already covered the `axis-186 + axis-115` six-bucket shift-agreement typology) — are **agreement compounds**: both axes are designed to detect the same generic alternative (a stochastic shift), and the joiner asks *do they corroborate?*

`classifyKuiperCliffShapeVsDominanceCompound` is the first compound in the project history where the underlying axes are deliberately maximally decoupled on a specific class of alternative. The CHANGELOG for v0.6.484 spells this out in textbook form:

> Kuiper V is MAXIMISED by symmetric two-sided ECDF crossings (e.g. A = N(0, 1) vs B = N(0, 4): two equal lobes, kpV >> 0). Cliff's delta is IDENTICALLY ZERO on the same alternative (P(B > A) = P(A > B) = 0.5 by symmetry). The two axes are therefore MAXIMALLY DECOUPLED on exactly the alternative against which Kuiper recovers power vs MW / Cliff / KS, and TIGHTLY COUPLED on monotone stochastic shifts where both should agree.

This is not a stylistic distinction. It changes what the joint cells of the 2x2 table *mean*. In an agreement compound, the off-diagonal cells (one axis rejects, the other does not) are usually framed as power loss, calibration drift, or sample-size noise — i.e. *something to be explained away*. In a deliberate-decoupling compound, the off-diagonal cells are the **primary diagnostic signal**: they tell you which class of alternative is in play. `shape-only-no-dominance` and `dominance-only-shape-ns` are not failure modes of the test, they are the **product** of the test.

This reframes how the bucket headlines should be read. `shapeOnlyNoDominance > 0` is not "Kuiper found something Cliff missed" — it is positive evidence of a **pure scale shift, multimodality emergence, or symmetric bimodality flip** in that source. `dominanceOnlyShapeNs > 0` is positive evidence of a **small, broad, ordinal-monotone shift** that Kuiper's localised supremum-sum cannot resolve at the available `n`. Each off-diagonal bucket has a named alternative class behind it.

## The carve-out for `shape-with-large-effect-ns-ci` is the most subtle design choice

The naïve 2x2 table would give four buckets. v0.6.484 ships with six, because two of the cells get carve-outs that prevent strong point estimates from polluting the inference signal:

```
- kpReject x cdCiExcludesZero         -> shape-and-dominance
- kpReject x !cdCiExcludesZero        -> shape-only-no-dominance
                                         (carved-out: shape-with-large-effect-ns-ci
                                          when |delta| >= 0.474)
- !kpReject x cdCiExcludesZero        -> dominance-only-shape-ns
- !kpReject x !cdCiExcludesZero       -> both-ns-large-shape-ratio
                                         when |delta| >= 0.147
                                         else both-ns-negligible
```

The 0.474 / 0.147 thresholds are the Romano–Coraggio–Skowronski 2006 magnitude bins from axis-191, reused verbatim. The reason the carve-out matters: a row can have `cdCi` containing zero (so the *interval* is inferentially inconclusive) while still having `|cdDelta|` very large (a strong point-estimate dominance shoved into uncertainty by a small `n`). If you naively bucket this row into `shape-only-no-dominance` you are claiming a **pure-shape** signal in the presence of strong (but wide-CI) dominance evidence. That would be a category error: the shape signal is not pure, it is *contaminated by dominance* that happens not to clear the inferential bar. The `shape-with-large-effect-ns-ci` carve-out flags this row as "shape signal carries more interpretive weight here" without claiming the dominance is null.

The same logic produces the `both-ns-large-shape-ratio` vs `both-ns-negligible` split on the all-null cell. A row where Kuiper and Cliff both fail to reject AND `|cdDelta| < 0.147` is the canonical *nothing happened* row — both axes negligible, no follow-up needed. A row where both fail to reject but `|cdDelta| >= 0.147` is a watch-list row: there is a small-or-larger ordinal dominance trend visible in the point estimate that the corpus is just too small to confirm. The taxonomy handles "no signal" and "underpowered signal" as different rows, which is exactly the right call.

## The live panel result: `shapeOnlyNoDominance = 0`

The headline result on the live data is striking. Five sources qualify after the tenure filter, and the 2x2 cross-tab is:

| source      | kpV    | kpP     | cdDelta | cdCi             | excl0 | mag        | bucket                       |
|-------------|--------|---------|---------|------------------|-------|------------|------------------------------|
| claude-code | 0.5278 | 6.69e-4 | +0.4738 | [+0.249, +0.681] | true  | medium     | `shape-and-dominance`        |
| hermes      | 0.6889 | 7.20e-2 | +0.2840 | [-0.309, +0.852] | false | small      | `both-ns-large-shape-ratio`  |
| openclaw    | 0.7000 | 6.20e-2 | -0.9012 | [-1.000, -0.654] | true  | large      | `dominance-only-shape-ns`    |
| opencode    | 0.7500 | 6.30e-2 | -0.5000 | [-1.000, +0.125] | false | large      | `both-ns-large-shape-ratio`  |
| vscode-cp   | 0.1303 | 7.07e-1 | -0.1150 | [-0.225, +0.001] | false | negligible | `both-ns-negligible`         |

Counts: `shapeOnlyNoDominance = 0`, `shapeAndDominance = 1`, `dominanceOnlyShapeNs = 1`, `both-ns-large-shape-ratio = 2`, `both-ns-negligible = 1`, `shape-with-large-effect-ns-ci = 0`.

The zero count on `shapeOnlyNoDominance` is the key reading. Recall what that bucket *would have meant*: a source whose two halves differ in **shape only** (Kuiper rejects), with **no detectable ordinal dominance** (Cliff CI includes zero AND `|cdDelta|` is small). This is the classic pure-scale-shift / symmetric-bimodality-flip signature.

Zero sources land there. Every Kuiper rejection in the panel (claude-code) is accompanied by a CI-excludes-zero Cliff call (cdDelta = +0.4738, `[+0.249, +0.681]`). Every wide-CI Cliff row (hermes, opencode) also has `kpP > 0.05`. There is no source on the panel where the two halves differ in distribution shape but agree on stochastic order.

The CHANGELOG draws the explicit conclusion:

> Consistent with the panel exhibiting primarily LOCATION/STOCHASTIC shifts (axis-186/189/190/191 chorus) rather than pure scale shifts.

This is a **regime claim**, not a per-source claim. The five-source panel as a whole is in a location-shift regime. Whatever distribution movement is happening in the half-splits is monotone-stochastic — one half is consistently larger, smaller, or unchanged versus the other — and not symmetric-around-the-median scale movement. The corpus is not generating bimodality flips, scale explosions, or kurtosis shifts in this window. It is generating ramp-ups (`claude-code`, +0.47 medium-magnitude second-larger), ramp-downs (`openclaw`, -0.90 large first-larger), and steady states (`vscode-cp`, both negligible).

This dovetails with the entire axis-181..190 family the prior posts on this blog already covered. The `axis-189 Wilcoxon-signed-rank halves` post documented the `claude-code z = -3.59, r_rb = 0.77` paired-design rejection. The `axis-190 paired sign-test` post documented `claude-code z = -2.6, vscode-cp z = -2.05`. The `axis-191 bootstrap-percentile-CI` post documented `vscode-cp` as the zero-spanning interval source. All four of these axes — sign, rank, bootstrap, and now Kuiper × Cliff — independently agree:

- `claude-code` is in a strong monotone shift regime (every axis rejects, every axis assigns the same sign).
- `vscode-cp` is in a steady-state regime (every axis fails to reject, every magnitude is negligible).
- The middle three sources (`hermes`, `openclaw`, `opencode`) are in mixed-evidence regimes where the per-axis verdicts depend on which alternative the axis was designed to detect.

The axis-192 × axis-191 joiner is the first axis to *name* this regime as a regime, by exhibiting the absence of the alternative (`shapeOnlyNoDominance = 0`) that would falsify the location-shift framing.

## How to read the two non-trivial off-diagonal rows

`openclaw` is the cleanest off-diagonal: `dominance-only-shape-ns` with `kpP = 0.062` and `cdDelta = -0.901`, `cdCi = [-1.000, -0.654]`. The CHANGELOG explanation is technically dense but worth quoting in full:

> cdCi `[-1.000, -0.654]` excludes zero with cdDelta = -0.901 (LARGE first-larger), but Kuiper kpP = 0.062 just above the .05 threshold. Pattern: every cross-pair favours the first half overwhelmingly (deltaHat near -1 means almost every (i, j) has B_j < A_i), so the ordinal-dominance signal saturates while the localised ECDF gap (kpV = 0.7) sits just under Kuiper's conservative inflation factor at n1 = 9 / n2 = 10.

What this is saying: when `deltaHat` saturates near ±1, you are in the regime where one half is *uniformly above or below* the other. This makes the ordinal dominance signal extremely strong (the rank-based Cliff stat is essentially counting "how often does B beat A?" and the answer is "never"), but the ECDF gap that Kuiper measures is geometrically *bounded* — a maximally-displaced pair of ECDFs cannot have a `kpV` larger than ~1.0 in normalized space, and at small `n` the asymptotic Kuiper p-value table inflates the threshold. So you get the seemingly contradictory pattern of `cdDelta` saying "huge effect, certain direction" and `kpP` saying "marginal, do not reject."

This is exactly the small-but-broad-stochastic-shift pattern the `dominance-only-shape-ns` bucket was designed to surface. It is *not* a contradiction between the axes. It is the joint signature of a saturating ordinal shift on a small sample.

`hermes` and `opencode` both land in `both-ns-large-shape-ratio`, the watch-list bucket. Neither axis rejects at .05, but `|cdDelta|` is at small (hermes, 0.284) or large (opencode, 0.500) magnitude. The CI for opencode is `[-1.000, +0.125]` — a 1.125-wide interval that just barely fails to exclude zero. The CHANGELOG calls these "watch-list rows: distribution differences are detectable in point estimates but inferentially inconclusive at the corpus scale."

The honest reading: these rows are *underpowered*. The point estimates suggest movement, the CI is too wide to confirm. The action item is *more data* (longer tenure, more daily samples), not *more axes* (you cannot triangulate your way out of a wide CI by adding statistics derived from the same data).

## How this joiner relates to the axes-181..190 closure narrative

Prior posts on this blog argued that axes 181..190 closed the W17 daily-token-halves location-and-scale family. The closure was framed as an *eight-axis sweep* (van der Waerden, Fligner-Policello, Yuen-Welch, Savage, Baumgartner-Weiss-Schindler, Hodges-Lehmann, A12, permutation Welch t) with two cross-axis combiners (`vdwSavageCombinedSignedStouffer`, the axis-186 + axis-115 six-bucket shift-agreement table).

Axes 191..192 plus the v0.6.484 joiner are not just *more axes*. They are a different kind of axis entirely: axis-191 is the first **point-estimate-with-CI** axis on this panel (Cliff's delta is a magnitude, not a p-value), and axis-192 is the first **omnibus shape** axis (Kuiper detects any ECDF crossing, not just a location or scale shift). The joiner is the bridge between these two.

This means the W17 closure was actually premature. The location-and-scale family was closed at axis-190; the **shape-and-dominance** family is just opening at axis-191/192. The `classifyKuiperCliffShapeVsDominanceCompound` joiner is the first axis in this new family that deliberately exploits maximal-decoupling between its inputs, and the `shapeOnlyNoDominance = 0` headline is the first claim that a real-world panel can *fail to exhibit* the alternative the family was built to detect.

That null finding is informative. It says: in the W17 daily-token-halves window, on five real-world sources spanning 14-day to 265-day tenures, none of the sources exhibits a pure-shape distributional shift. The corpus is statistically *boring* in the shape dimension. All the action is in the location dimension, exactly where axes 181..190 already detected it.

## What the next compound should look like

The natural next compound is **axis-192 (Kuiper) × axis-180 (Sukhatme)** or **axis-192 × axis-117 (Siegel-Tukey)** — pairing the omnibus ECDF gap statistic with a *pure scale* test. That would give a joiner with the cell:

- `kpReject x scaleReject` -> `shape-with-scale-component`
- `kpReject x !scaleReject` -> `shape-without-scale-component` (location-driven Kuiper)
- `!kpReject x scaleReject` -> `scale-without-shape-component` (small but detectable scale)
- `!kpReject x !scaleReject` -> nothing happening

Whereas axis-192 × axis-191 decomposes ECDF movement into shape vs *dominance*, axis-192 × axis-117 would decompose it into shape vs *scale*. Together those two compounds would let you read any axis-192 rejection as either location-driven, scale-driven, or shape-driven (multimodality / kurtosis), which is the full triple decomposition you actually want.

The repo's commit cadence makes this likely to appear within 1-2 releases. The `feat(axis-192): Kuiper two-sample test on half-split daily series` at `87aedf1` was followed by the KS joiner (axes 192 + 118) at `c396a91` within the same release, and the Cliff joiner (axes 192 + 191) at `377470f` in the next release. Pew-insights is in a deliberate compound-build sweep around axis-192 — three joiners in two releases — and the pure-scale joiner is the missing third leg.

## Why this matters operationally

Two operational consequences worth naming:

1. **The location-shift regime claim is testable.** If a future release introduces a sixth source whose Kuiper rejects but whose Cliff CI strictly contains zero with negligible `|cdDelta|`, that source will land in `shapeOnlyNoDominance`, the count will become non-zero, and the regime claim "the panel exhibits primarily location/stochastic shifts" will need revision. The headline counts are designed to *flip on* the moment a falsifying source appears.

2. **The watch-list rows have an action.** `hermes` and `opencode` both sit in `both-ns-large-shape-ratio`. The action on these rows is not "add more axes." It is "wait for more data." Specifically, both rows have `cdCi` widths exceeding 1.0, which means roughly two more half-cycles of daily data are needed before the CI tightens enough to either exclude or contain zero. The joiner gives a principled stopping criterion for *when to look again*.

The `classifyKuiperCliffShapeVsDominanceCompound` joiner is, in net, the first axis on the panel where the off-diagonal cells of the joint table are the *answer* rather than the *problem*. The `shapeOnlyNoDominance = 0` result is positive evidence — the *absence* of an alternative the joint table was designed to detect — for the regime claim that the W17 daily-token-halves panel is in a location-shift, not a shape-shift, dynamic.

That is more interpretive leverage per axis than any prior compound on the panel has produced.
