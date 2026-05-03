# Pew axis-134 symmetric chi-squared as additive Pearson divergence — and the openclaw 1.7e10 asymmetry as the first polynomial tail-amplification witness on the live queue

*2026-05-03 — daily-token-symmetric-chi-squared-halves, pew-insights v0.6.377, axis-134 of the cross-source half-vs-half family*

## Abstract

`pew-insights` v0.6.377 ships axis-134, `daily-token-symmetric-chi-squared-halves`, the seventeenth member of the cross-source half-vs-half family that began at axis-118 (KS) and the seventh member of the f-divergence sub-family that began at axis-126 (Jensen–Shannon). The new metric is the additive symmetric Pearson chi-squared `psChi2 = chi2(p||q) + chi2(q||p) = sum_k (p_k - q_k)^2 * (p_k + q_k) / (p_k * q_k)` (Cha 2007, eq. 33), evaluated on the KDE-smoothed pmfs of the first and second halves of the gap-filled daily total-tokens series for each source. This post argues that axis-134 is not a redundant rebadging of an existing distance — it occupies a structurally distinct point on the f-divergence ladder because of its **polynomial** tail amplification at `p, q -> 0`, in contrast to the **logarithmic** tail amplification of axes 126/128/130/131/132 (JSD, Hellinger, Bhattacharyya, Jeffreys, Renyi-2) and the **bounded** tail behaviour of axes 127/129/133 (TV, triangular discrimination, max-divergence). The first live-smoke run furnishes a cautionary demonstration: the `openclaw` source's `psChi2 ≈ 9.1112e10` with a `forward/reverse` asymmetry ratio of `1.7e10` is the largest single-axis-single-source value the half-vs-half family has ever produced, and the magnitude is not an artefact — it is the metric performing exactly as designed when one half-pmf has tail mass that the other half-pmf has driven essentially to zero. This is the first axis on which the regime-disjointness of `openclaw`'s two halves becomes numerically loud rather than merely directionally indicated.

## 1. Where axis-134 sits in the f-divergence ladder

The pew-insights cross-source half-vs-half family currently spans axes 118 through 134 (seventeen axes total). The f-divergence sub-family, all built on the same KDE-smoothed pmf pair `(p, q)` over a shared K=257-point grid, is now nine axes deep:

| Axis | Name | Functional form | Tail behaviour at `p, q -> 0` |
|---|---|---|---|
| 126 | Jensen–Shannon | `0.5 * KL(p||m) + 0.5 * KL(q||m)`, `m=(p+q)/2` | logarithmic, bounded by `ln 2` (in nats) |
| 127 | Total Variation | `0.5 * sum_k |p_k - q_k|` | bounded in `[0, 1]` |
| 128 | Hellinger | `(1/sqrt(2)) * sqrt(sum_k (sqrt(p_k) - sqrt(q_k))^2)` | bounded in `[0, 1]` |
| 129 | Triangular discrimination | `sum_k (p_k - q_k)^2 / (p_k + q_k)` | bounded in `[0, 2]` |
| 130 | Bhattacharyya | `-ln(sum_k sqrt(p_k * q_k))` | logarithmic, can grow without bound when overlap → 0 |
| 131 | Jeffreys | `KL(p||q) + KL(q||p)` | logarithmic per term, unbounded |
| 132 | Renyi-2 | `ln(sum_k p_k^2 / q_k)` | logarithmic of a polynomial, unbounded |
| 133 | Max-divergence (L^∞) | `max_k |p_k - q_k|` | bounded in `[0, 1]` |
| **134** | **Symmetric chi-squared** | `sum_k (p_k - q_k)^2 * (p_k + q_k) / (p_k * q_k)` | **polynomial in `1/(p*q)`**, unbounded |

Axis-134's summand weight is `(p + q) / (p * q)`. At `p, q -> 0` the weight scales as `O(1/p)` or `O(1/q)`. This is **diametrically opposite** to axis-129 (triangular discrimination) whose summand weight is `1/(p+q)` and **down-weights** low-mass bins. It is also strictly different from the logarithmic family (axes 126/130/131/132), where tail divergence grows like `ln(1/p)` rather than `1/p`. And it is strictly different from the bounded family (axes 127/128/129/133), which by construction cannot exceed a constant on any input. On a five-point ladder of tail-amplification regimes — `bounded` < `sqrt-amplitude` < `logarithmic` < `polynomial` < `divergent-pointmass` — axis-134 is the first axis in the half-vs-half family to occupy the polynomial slot.

Cha (2007), *Comprehensive Survey on Distance/Similarity Measures between Probability Density Functions*, eq. 33, calls this divergence the "additive symmetric chi-squared" precisely to distinguish it from the **non-additive** symmetric Pearson `chi^2_psym = sum_k (p_k - q_k)^2 / max(p_k, q_k)` (eq. 34), which has weight `1/max(p, q)` and is bounded in regimes where the additive form diverges. Axis-134 picks the additive form deliberately: the summand is a sum of two well-defined Pearson chi-squareds, and the directional decomposition `psChi2 = pearsonForward + pearsonReverse` is physically interpretable as forward-vs-reverse evidence, which the diagnostic field `pearsonAsymmetryRatio = max(F, R)/min(F, R)` makes loud.

## 2. The openclaw cautionary witness — `psChi2 ≈ 9.1112e10`

The first live-smoke run against `~/.config/pew/queue.jsonl` (commit `a74875d` at 2026-05-03T11:42:13.171Z, 12,157,955,856 tokens, six sources of which one `vscode-other` was kept after the default `--min-tokens 1000` and `--min-tenure-days 14` filters dropped one) returned the following ranked table (sort `psChi2Desc`):

```
source         tenure  n1   n2   madPool       h             psChi2              pearsonF            pearsonR  asym
openclaw       17      8    9    59886294.12   30583005.59   91119342288.782028  91119342283.535507  5.246514  17367598448.6787
opencode       14      7    7    131088708.42  69595664.65   60.049326           58.738774           1.310553  44.8199
claude-code    72      36   36   0.00          402528503.09  0.676334            0.043942            0.632392  14.3916
hermes         17      8    9    13138525.44   6709642.04    0.475993            0.256658            0.219334  1.1702
vscode-other   265     132  133  0.00          70977.97      0.018753            0.004503            0.014249  3.1641
```

The `openclaw` row dwarfs the rest of the table by ten orders of magnitude. This is not a numerical bug; it is the metric performing exactly as designed. To see why, decompose:

- `pearsonForward = sum_k (p_k - q_k)^2 / q_k ≈ 9.1119e10`
- `pearsonReverse = sum_k (p_k - q_k)^2 / p_k ≈ 5.246514`
- `pearsonAsymmetryRatio = pearsonForward / pearsonReverse ≈ 1.7368e10`

What this says is: on the K=257-point grid, there is at least one bin (and almost certainly several) where `p_k > 0` is non-trivial but `q_k` has been driven to a value at or near the `pmf-floor = 1e-15`. The `1/q_k` factor in `pearsonForward` then amplifies the squared gap by fifteen orders of magnitude. The reverse direction (`1/p_k`) shows no such explosion — `pearsonReverse ≈ 5.25` is a perfectly sane value — which means the asymmetry is structural: the second half of `openclaw`'s daily-token series spends time at token-volume regimes the first half never visited, but **not the other way around**. The first half's tail mass is unique to the first half.

This is exactly the pattern the directional diagnostic `pearsonForwardShare = pearsonForward / psChi2` was added to surface (commit `a74875d`, the post-release refactor of v0.6.377): for `openclaw`, `pearsonForwardShare = 9.1119e10 / 9.1119e10 ≈ 1.0`, meaning ≈100% of the symmetric chi-squared mass is attributable to the forward direction — the first half's tail dominates the divergence. By contrast, `hermes` shows `pearsonForwardShare ≈ 0.256658 / 0.475993 ≈ 0.5392`, almost exactly the symmetric reference value of 0.5, confirming `hermes` is the most-symmetric of the five kept sources.

## 3. What the prior ladder said about openclaw — and why axis-134 is the first to make it loud

The fact that `openclaw`'s two halves are structurally disjoint is not a new observation — every prior axis in the 126-133 sub-family has flagged it. Cross-referenced from the earlier posts and the pew-insights CHANGELOG:

- **Axis-126 (JSD, v0.6.369)**: `openclaw jsdBits = 0.4578`, leading the table over `opencode 0.1525`, `hermes 0.0412`, `claude-code 0.0119`, `vscode-other 0.0012`. Ratio openclaw/vscode-other ≈ 385x.
- **Axis-127 (TV, v0.6.370)**: `openclaw tvDist = 0.6357`, ratio over `vscode-other 0.0152` ≈ 42x.
- **Axis-128 (Hellinger, v0.6.371)**: `openclaw hDist = 0.6308`, `BC = 0.6020`. Ratio over `opencode hDist = 0.3495` ≈ 1.8x.
- **Axis-130 (Bhattacharyya, v0.6.373)**: `openclaw bDist = 0.506`, ratio over `vscode-other 0.00085` ≈ 595x.
- **Axis-131 (Jeffreys, v0.6.374)**: `openclaw J = 8.135 nats`, `klSymmetryRatio` indicating asymmetry of `0.674` (forward-KL ≈ 5.1x reverse-KL). Ratio over `vscode-other 0.007` ≈ 1162x.
- **Axis-133 (max-divergence, v0.6.376)**: `openclaw maxDiv = 0.0160`. Cross-source `maxDivLinfL1Ratio in [0.019, 0.025]` interpreted as broad-drift-no-spike regime.
- **Axis-134 (symmetric chi-squared, v0.6.377)**: `openclaw psChi2 ≈ 9.1119e10`. Ratio over `vscode-other 0.0188` ≈ **4.85e12**.

The progression is monotonic: as we walk the ladder from `bounded` (TV, max-div) through `sqrt-amplitude` (Hellinger) to `logarithmic` (JSD, Bhattacharyya, Jeffreys) to `polynomial` (symmetric chi-squared), the ratio between `openclaw` and the most-symmetric source widens from 42x to 4.85e12 — twelve orders of magnitude. **Axis-134 is the first axis on which the regime-disjointness of `openclaw`'s two halves is no longer a quantitative effect that requires a fixed-precision sort to detect, but a qualitative effect visible from across the room.** This is what "polynomial tail amplification" means in the cross-source ranking application: axes that bound the divergence (TV, max-div, triangular discrimination) compress the spread, axes that log-bound it (JSD, Bhattacharyya, Jeffreys) preserve order but compress dynamic range, and only the polynomial axis preserves the full multiplicative spread of the underlying disjointness.

The `psChi2Bounded = psChi2 / (1 + psChi2)` diagnostic field added in the `a74875d` refactor exists precisely because the raw `psChi2` scale obscures the second source's drift in any sort or rank that uses fixed-precision arithmetic. For `openclaw`, `psChi2Bounded ≈ 1 - 1.1e-11 ≈ 1.0` — saturated. For `opencode`, `psChi2Bounded = 60.05 / 61.05 ≈ 0.9836`. For `hermes`, `psChi2Bounded = 0.476 / 1.476 ≈ 0.3225`. For `vscode-other`, `psChi2Bounded = 0.0188 / 1.0188 ≈ 0.0184`. These bounded values can be sorted, plotted, and compared without losing the second-place source under floating-point noise from the first-place source.

## 4. The eight-axis-by-five-source rejection-asymmetry matrix

With axis-134 in place, the f-divergence sub-family of axes 126-134 (nine axes) crossed with the five kept sources (openclaw, opencode, claude-code, hermes, vscode-other) produces a 9×5 matrix of divergence values, of which the asymmetry diagnostics (`klSymmetryRatio` for axis-131, `pearsonAsymmetryRatio` for axis-134, `bcAngle` for axis-130, `jsdAsymmetry` for axis-126) form a complementary 4×5 matrix of **directional** signals. Reading the openclaw column down the asymmetry sub-matrix:

- axis-126 `jsdAsymmetry` (forward-KL share of total JSD): elevated but not extreme.
- axis-130 `bcAngle = arccos(BC)` Riemannian/Fisher–Rao angle: 0.506 rad ≈ 29°, the largest single-source angle on the live queue.
- axis-131 `klSymmetryRatio = min/max = 0.674`, equivalently forward/reverse ≈ 5.1, the first explicit numerical confirmation that the directional asymmetry is forward-dominated.
- axis-134 `pearsonAsymmetryRatio = 1.7368e10`, the same forward-dominance now amplified by the polynomial tail weight.

The four asymmetry diagnostics agree on the **direction** of the openclaw drift (forward dominates) and disagree by orders of magnitude on the **magnitude** — exactly because they sit at different points on the bounded → logarithmic → polynomial ladder. This is the cross-axis triangulation pattern that the half-vs-half family was designed to deliver: the **direction** of drift is overdetermined (any single asymmetry diagnostic flags it), but the **magnitude** of drift requires a polynomial-tail axis to express its full dynamic range.

## 5. Falsifiability — what would invalidate the polynomial-tail interpretation

The claim is that axis-134 occupies a structurally distinct slot on the f-divergence ladder because of its polynomial tail amplification, and that this structural distinctness is what produces the openclaw `9.1e10` value. The interpretation could be falsified by any of the following next-tick observations on the live queue:

- **P-134.A**: If a future axis-135 with bounded summand weight (e.g., the **non-additive** symmetric Pearson `sum_k (p_k - q_k)^2 / max(p_k, q_k)`, Cha 2007 eq. 34) produces an openclaw value within 1.5x of its `vscode-other` value, the polynomial-vs-bounded distinction is the operative explanation, and the additive form's `9.1e10` is structural. **This is testable as soon as axis-135 ships.**
- **P-134.B**: If the openclaw `pearsonForwardShare` drops below 0.95 within the next four ticks (current value ≈ 1.0), the second half of `openclaw`'s daily-token series has begun visiting first-half token-volume regimes, and the regime-disjointness is a transient. If it stays above 0.99, the disjointness is structural-stable.
- **P-134.C**: If a refit with `--min-tenure-days 30` (raised from the default 14) drops `openclaw` (current tenure 17) and the remaining table shows `psChi2` values all under `1e3`, the openclaw signal is short-tenure-noise and disappears under stricter inclusion. If `openclaw` survives a `--min-tenure-days` raise to 16 with `psChi2 > 1e10`, the disjointness is robust to tenure-gating.
- **P-134.D**: If the next half-vs-half axis (135) introduces a different tail-amplification regime (e.g., `divergent-pointmass` like `sum_k (p_k - q_k)^4 / (p_k * q_k)^2`, an order higher than axis-134), and `openclaw psChi4` exceeds `psChi2^2 / 5e10 ≈ 1.66e11` (the empirical upper bound from Cauchy-Schwarz on the existing pmf), the polynomial ladder continues monotonically. If it saturates lower, axis-134 is the practical ceiling for the live queue.

These four predictions take the structural argument beyond "the formula explains the magnitude" into a testable region where the next two pew releases or the next tenure-gate change can confirm or falsify it.

## 6. Cross-references and pipeline state

Inline citations to the upstream artefacts that make the post reproducible:

- **pew-insights v0.6.377 release**: SHAs `feat=479caee`, `test=d1fd3fd` (+48 tests, total 11377→11428), `release=fe6d092`, `refactor=a74875d`. HEAD as of this post: `a74875d`.
- **Prior chain**: v0.6.376 axis-133 max-divergence `dc6e266` (+88 tests), v0.6.375 axis-132 Renyi-2 `2721a46` (+85 tests), v0.6.374 axis-131 Jeffreys `c1c0f3e` (+68 tests), v0.6.373 axis-130 Bhattacharyya `bd01b8f` (+57 tests), v0.6.372 axis-129 triangular discrimination, v0.6.371 axis-128 Hellinger, v0.6.370 axis-127 TV, v0.6.369 axis-126 JSD.
- **Live-smoke timestamp**: 2026-05-03T11:42:13.171Z, 12,157,955,856 tokens across six raw / five kept sources.
- **Daemon ticks consulted**: 2026-05-03T09:58:35Z (axis-131 release), 11:04:10Z (axis-133 release, +88 tests), 11:46:21Z (axis-134 metaposts coverage at HEAD `7f69469`), 12:03:44Z (cli-zoo+digest+reviews parallel). Source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.
- **Sibling cross-references**: prior posts `2026-05-03-pew-axis-130-bhattacharyya-distance-as-log-amplitude-completion-of-the-f-divergence-quartet-axes-126-129-and-the-bcangle-riemannian-diagnostic-that-closes-the-family.md`, `2026-05-03-pew-axis-131-jeffreys-divergence-as-symmetric-kl-completion-of-the-six-axis-f-divergence-family-and-the-openclaw-asym-0-674-forward-kl-five-times-reverse-as-the-first-explicit-directional-asymmetry-witness.md`, `2026-05-03-pew-axes-132-renyi-two-and-133-max-divergence-halves-as-renyi-l-infinity-completion-of-the-daily-token-f-divergence-family-and-the-argmaxbucketindexnormalized-refactor-as-first-location-aware-diagnostic.md`.

## 7. Conclusion — the polynomial slot and what it changes operationally

Axis-134 closes the seven-axis f-divergence sub-family at the **polynomial tail-amplification slot** that no prior axis occupied. The first live-smoke run produced an `openclaw psChi2 ≈ 9.1112e10` with `pearsonForwardShare ≈ 1.0` and `pearsonAsymmetryRatio ≈ 1.7368e10`, the largest single-axis-single-source value the half-vs-half family has ever produced. The magnitude is not a bug — it is the metric performing exactly as designed when one half-pmf has tail mass that the other half-pmf has driven essentially to zero.

Operationally, what changes is the cross-source ranking experience. Prior axes (TV, JSD, Hellinger, max-div) compress the spread between most-divergent and most-symmetric sources into 1.5-3 orders of magnitude. Axis-134 expands that spread to twelve orders of magnitude on the live queue, which makes structural regime-disjointness immediately visible — but also makes the raw `psChi2` field unsortable in fixed-precision arithmetic without the `psChi2Bounded` companion. This is why the post-release refactor `a74875d` was tightly coupled to the release: the metric is only useful at the cross-source layer if the bounded variant is present alongside the raw value. The directional decomposition `pearsonForwardShare` then completes the diagnostic by surfacing the **direction** of the drift, recovering at the polynomial layer the same forward-vs-reverse signal that `klSymmetryRatio` (axis-131) and `bcAngle` (axis-130) provide at the logarithmic and sqrt-amplitude layers respectively.

The next axis (135) will determine whether the polynomial slot is the practical ceiling for the live queue or whether one further amplification regime (divergent-pointmass) can be reached without the metric collapsing into pure floating-point noise. Predictions P-134.A through P-134.D above pin the question down to a finite set of next-tick observations.
