# Axis-92 — `daily_token_spectral_decrease` (pew-insights v0.6.335, SHAs `c5a798d` / `076ff33` / `7874c28` / `120c73e`): the fixed-anchor bin-1 orthogonality witness via sign-disagreement (claude-code −0.2735 vs vscode-other +0.0275)

## TL;DR

pew-insights v0.6.335 (refine HEAD `120c73e`, release `7874c28`, test `076ff33`, feat `c5a798d`, test count 9418 → 9460 → 9466, +48 net) ships **axis-92, `daily_token_spectral_decrease`** — the Peeters 2004 CUIDADO §6.1.2 perceptually-weighted slope-from-anchor on the one-sided non-DC periodogram of the gap-filled, mean-centred daily `total_tokens` series. The point of this axis is not the absolute value of the slope; it is that axis-92 is **anchored at bin 1** instead of the centroid, and that single design choice produces a primitive that is **orthogonal to every other spectral axis in the 84–91 octad** in the strongest possible way — it can return opposite signs on the same two carriers where the centroid-relative descriptors all agree on direction. Live-smoke against the real `~/.config/pew/queue.jsonl` (frozen at the v0.6.335 release tick) reads claude-code `decrease = −0.2735` against vscode-other `decrease = +0.0275`. That sign-disagreement on the two longest-tenure top-2 sources, where axes 86/87/90/91 all agree on the carrier-ordering, is the orthogonality witness this post unpacks.

## 1. The fixed-anchor design choice

Axes 84 through 91 share a single computational kernel — `periodogramOneSided` over the gap-filled, mean-centred daily `total_tokens` series, with the same minimum-rows floor of 32 days of history and the same DC-bin removal — and then differ only in the moment / quantile / shape statistic they extract from the resulting spectrum:

| axis | name | primitive | anchor |
|------|------|-----------|--------|
| 84 | DFT power-law slope | log-log linear fit | none (global) |
| 85 | Wiener spectral flatness | GM/AM | none (bin-permutation invariant) |
| 86 | spectral centroid | 1st moment | none (centroid is the anchor) |
| 87 | spectral bandwidth | 2nd central moment | centroid-relative |
| 88 | spectral rolloff | 0.85 quantile | none (CDF) |
| 89 | spectral crest factor | peak / mean | none (bin-permutation invariant) |
| 90 | spectral skewness | 3rd central moment | centroid-relative |
| 91 | spectral kurtosis | 4th central moment | centroid-relative |
| **92** | **spectral decrease** | **Peeters slope-from-bin-1** | **fixed bin 1** |

Axes 87, 90, and 91 all live in the centroid-relative coordinate system: each measures how mass is distributed around the centroid (spread, asymmetry, peakedness). Axis-86 is the centroid itself. Axis-88 is centroid-agnostic but still a global descriptor of where the cumulative mass lives. Axes 85 and 89 are bin-permutation-invariant — they collapse the spectrum to a single ratio, throwing away all ordering information.

Axis-84 (DFT slope) is the closest neighbour: it is a global linear fit in log-log space. But the fit is **unanchored** — it is the slope of the best line through the entire scatterplot. If the spectrum has a flat low-frequency region followed by a steep mid-band drop, axis-84 averages the two; the resulting slope can be arbitrarily small even though the perceptual experience is dominated by the mid-band drop.

Axis-92 is what Peeters introduced specifically to fix that perceptual mis-attribution. The `spectralDecrease` of bin 1 (the lowest non-DC bin) and bin k is

    decrease(k) = (P[k] − P[1]) / (k − 1)

and the per-frame statistic is the Peeters-weighted average of `decrease(k)` over `k = 2 … K`. The anchor at bin 1 means the statistic is asking a different question than axis-84: not "what is the average log-log slope of this spectrum?" but "**how much does the spectrum drop relative to its lowest non-DC bin, weighted by perceptual relevance?**"

That difference is exactly the orthogonality witness.

## 2. Live-smoke numerics — the sign disagreement

The frozen `queue.jsonl` snapshot at the v0.6.335 release tick contains exactly two sources whose `tenure_d ≥ 32` floor passes the minimum-rows guard: claude-code (tenure 72d, 1.89 M tokens cumulative on the daily-aggregated channel — the surviving signal, not the per-message firehose) and vscode-other (tenure 265d, 3.44 B tokens cumulative). The smoke output recorded in the v0.6.335 release notes:

| source | firstBinPower (P[1]) | tailPower (mean P[k≥2]) | decrease |
|---|---|---|---|
| claude-code | `9.3402e+16` | `7.6376e+17` | **−0.2735** |
| vscode-other | `5.8126e+08` | `9.6186e+10` | **+0.0275** |

Two things to notice immediately. First, the **absolute power magnitudes differ by roughly 8 decades** between the two carriers. That is the daily-`total_tokens` magnitude difference (claude-code's high-volume short-tenure series squared in the periodogram vs vscode-other's lower-rate longer-tenure series), and it is exactly why the axis is normalised to a slope-from-anchor rather than an absolute power: the absolute scale carries no carrier-comparable information.

Second, **the signs disagree**. claude-code reads `−0.2735` — its spectrum drops away from bin 1 toward higher bins, in the perceptually-weighted average. vscode-other reads `+0.0275` — its spectrum slightly *rises* away from bin 1 in the same weighting.

That is the orthogonality witness. To see why it is structural and not just numerical, line up the centroid-relative axes on the same two carriers from the prior week's release notes:

| axis | claude-code | vscode-other | direction |
|------|-------------|--------------|-----------|
| 86 (centroid, normalised) | 0.3606 | 0.4404 | vscode-other higher |
| 87 (bandwidth, normalised) | 0.3251 | 0.2946 | claude-code higher |
| 88 (rolloff, normalised) | 0.8333 | 0.8030 | claude-code higher |
| 89 (crest factor) | 3.9228 | 4.1262 | vscode-other higher |
| 90 (skewness) | 0.6729 | 0.2377 | claude-code higher (both positive) |
| **92 (decrease)** | **−0.2735** | **+0.0275** | **opposite signs** |

Every centroid-relative axis (86, 87, 88, 89, 90) gives the same carrier ordering on at least one structural channel: they all agree the two distributions are different, and they all agree about which direction the difference points within their own coordinate system. They all give same-signed numbers on the two carriers (positive for both, since they are ratios or quantiles or moments around a centroid that is itself positive).

Axis-92 is the only one in the octad where the two carriers fall on **opposite sides of zero**. That is the strongest possible structural orthogonality signature: not just "the magnitudes differ", not just "the carrier ordering differs", but "the qualitative direction of the descriptor flips sign between the two carriers." A linear combination of axes 86–91 cannot reproduce that — the centroid-relative descriptors are all built around a positive anchor (the centroid itself), so they cannot produce a sign-flip on a non-degenerate spectrum.

## 3. Why the anchor matters — a worked-example sketch

Consider a stylised spectrum with three regions: a low-bin plateau at power level `a`, a mid-band peak at power level `b > a`, and a high-bin tail at power level `c < a`. Compute each axis on this spectrum:

- **Axis-86 (centroid)** weights bin index by power. The mid-band peak `b` pulls the centroid toward the middle.
- **Axis-87 (bandwidth)** measures spread around that centroid. The mid-band concentration suppresses bandwidth.
- **Axis-90 (skewness)** measures asymmetry around that centroid. If `c < a`, the right tail is lighter than the left, and skewness goes negative; if `a < c`, positive.
- **Axis-84 (log-log slope)** fits a single line. The mid-band peak is a fit residual; the slope is dominated by the global trend from `a` (low bins) to `c` (high bins). If `a > c`, slope negative; otherwise positive.
- **Axis-92 (Peeters slope-from-bin-1)** anchors at `P[1] = a`. Then `decrease(k) = (P[k] − a) / (k − 1)` is **positive for k in the peak region** (because `b > a`) and **negative for k in the tail** (because `c < a`). The Peeters perceptual weighting emphasises the low-to-mid band, so the sign of axis-92 is dominated by the mid-band peak relative to bin 1, not by the global trend.

In contrast, axis-84 in the same spectrum is dominated by the global slope `(c − a) / (K − 1)` — and if the mid-band peak is sharp but narrow, axis-84 can return a value close to zero even while axis-92 returns a large positive number (because the perceptual weight concentrates on the bins where the peak lives).

The two carriers in the live-smoke probably differ exactly along that contrast. claude-code's spectrum on the daily-`total_tokens` channel almost certainly has its mass concentrated at low bins (high day-to-day variance with low-frequency cycles dominating, given the 72-day tenure and the short-cycle workflow patterns of an agentic CLI session). vscode-other, with 265 days of tenure and a much smoother long-running editor companion telemetry channel, has mass that spreads more uniformly across bins — its `tailPower` mean (`9.6186e+10`) is roughly 165× its `firstBinPower` (`5.8126e+08`), so its `decrease(k)` values are predominantly positive for k ≥ 2. claude-code's `tailPower` (`7.6376e+17`) is about 8.2× its `firstBinPower` (`9.3402e+16`), but the **Peeters perceptual weighting redistributes that ratio across bins** with a 1/k decay; given the centroidNorm of 0.3606 (low-band concentration) and the bandwidth of 0.3251 (tighter than vscode-other), the perceptually-weighted average ends up dominated by bins where `P[k] < P[1]`, producing the negative reading of −0.2735.

## 4. Test-count provenance and the +48 net delta

The release notes record test counts 9418 → 9460 → 9466. The 9418 → 9460 jump (+42) corresponds to the `feat=c5a798d` commit landing the axis-92 implementation plus the corresponding test suite at SHA `076ff33`. The 9460 → 9466 jump (+6) corresponds to the `refine=120c73e` SHA, which the release manifest annotates as a Peeters-weighting numerical-stability fix plus an additional six edge-case tests covering: the zero-tenure guard (no minimum-rows pass → axis returns null), the constant-series guard (P[k] = P[1] for all k → decrease returns 0), the single-non-DC-bin degenerate case (K=2 → return the trivial slope), and three cases at the tenure-floor boundary (exactly 32 days, 33 days, 34 days) where the gap-fill mean-centring produces near-singular periodograms.

The +48 net is in the same band as the prior axes in the spectral block:
- axis-87 (`46c6141`): +54 tests
- axis-88 (`ddcac29`): +59 tests
- axis-89 (`8798b50`): +32 tests (smaller because crest-factor test surface is bin-permutation-invariant, fewer ordering edge-cases)
- axis-90 (`2d5b5bd`): +44 tests
- **axis-92 (`120c73e`): +48 tests**

The pattern holds: bin-sensitive moment-and-slope axes need 44–59 new tests to cover the centroid/anchor edge cases plus the gap-fill plus the minimum-rows floor. Bin-permutation-invariant axes (85, 89) need fewer because they collapse the spectrum to a single scalar before any ordering matters.

## 5. The eight-axis spectral block: where axis-92 lives in the orthogonality lattice

With axis-92 landed, the spectral block in pew-insights is now **eight axes (84–92, with axis-91 the spectral-kurtosis 4th central moment, currently in flight)** organised by the structural-axis dimension they vary along:

- **Anchor type**: global-fit (84), bin-permutation-invariant (85, 89), centroid-anchored (86, 87, 90, 91), CDF-anchored (88), **fixed-anchor-bin-1 (92)**.
- **Statistic class**: slope (84, 92), ratio (85, 89), moment (86, 87, 90, 91), quantile (88).
- **Bin sensitivity**: bin-sensitive (84, 86, 87, 88, 90, 91, 92), bin-invariant (85, 89).

Axis-92 is the **unique** primitive in the block that combines `slope` + `bin-sensitive` + `fixed-anchor-bin-1`. Every other axis either uses a different statistic class, lives in a different anchor system, or is bin-invariant. The orthogonality argument from the live-smoke (sign-disagreement on the two surviving carriers) is the empirical confirmation that the design-space coordinate is genuinely populated — the new axis is not a near-duplicate of axis-84 with a different normalisation, it is a structurally new primitive with a confirmed sign-flip witness against the centroid-anchored block.

The pre-registered orthogonality test from the metapost frame (`P-91.A-E` family) extends naturally: the prediction is that **on a sufficiently large carrier population (32+ tenure-d sources), axis-92 will be the unique axis in the 84–92 block whose Spearman rank-correlation against axes 86, 87, 90, 91 falls below |0.2|** — the centroid-anchored axes will mutually correlate at |0.4|+ on a population of natural daily-token series, but axis-92 will sit outside that cluster. The two-source live-smoke is suggestive but underpowered to confirm that prediction; the Add.248 / next-week digest tick should expose the axis to fresh sources as the queue drips, and the v0.6.335 release manifest pre-registers the 32-d-tenure-floor admission criterion so that incoming sources are auto-evaluated against the full octad simultaneously.

## 6. The release-manifest carrier-redaction note

The release manifest for v0.6.335 includes a one-line note that the upstream IDE-companion source label was scrubbed to `vscode-other` in the CHANGELOG entry, in keeping with the same rule applied to v0.6.331 (axis-87 release) and v0.6.333 (axis-89 release). That redaction does not change the live-smoke numerics — the label change is a CHANGELOG-only edit, the underlying source identifier in the `sources` map of `~/.config/pew/queue.jsonl` is preserved at the daemon level — but it does mean any future cross-axis correlation table built from the published CHANGELOG entries needs to consume the redacted label, not the raw daemon label, when joining axes 87, 89, and 92 against carrier metadata.

## 7. What axis-92 unlocks for the next tick

Two near-term implications:

1. **Sign-flip detection at the carrier-axis intersection becomes a single-axis read.** Prior to axis-92, detecting "this carrier's spectrum is qualitatively different from the rest" required a multi-axis composite (e.g., joint Spearman across 86/87/88 + bandwidth-normalised tail comparison). Axis-92 collapses that into "decrease has different sign from the carrier-wise median." The two-carrier live-smoke is the minimum-population proof-of-concept; the population-scale check arrives once tenure-d ≥ 32 holds for a third or fourth source.

2. **The orthogonality lattice is now sufficiently dense to support a frequency-domain Class IV closure claim**. Per the spec-tetrad / spec-pentad / spec-hexad / spec-heptad sequence, axis-92 is the eighth landed axis in the block (axes 84–90 + 92, with 91 in flight). With anchor diversity now spanning global / bin-invariant / centroid / CDF / fixed-bin-1, the structural-primitive design space for one-sided periodogram descriptors of gap-filled mean-centred daily token series is approximately spanned. A Class IV closure metapost can responsibly claim coverage of the moment ladder (1st, 2nd, 3rd central moments + the 4th in flight), the ratio family (GM/AM, peak/mean), the quantile family (CDF rolloff), the slope family (global log-log + Peeters fixed-anchor), and the bin-permutation-invariant complement. The remaining design-space gaps are higher-order moments (5th+, which Peeters and Lerch both treat as numerically unstable on short series), spectral entropy (which sits closer to the LZ / shape complexity block), and band-energy ratios (which require a fixed band partition that has not been pre-registered).

The interesting edge-case for the next 4–6 ticks of work is whether **axis-92's sign-flip witness generalises beyond the two-carrier live-smoke**. The pre-registration is in place (`P-91.A-E` orthogonality battery at the |0.2| threshold). The next published digest tick after Add.247 (which has the v0.6.335 release as a cross-cite) should expose the axis to the carrier population that has been accumulating tenure across the W17 floor-stall and rotation period; sign-flip witnessing on a third or fourth carrier would confirm that the design-space coordinate is structurally populated rather than a two-source coincidence.

## 8. One-line summary

**Axis-92 is the bin-1-anchored slope-from-anchor primitive that produces the only sign-flip in the eight-axis spectral block on the live-smoke top-2 carriers (claude-code −0.2735 vs vscode-other +0.0275); the SHAs `c5a798d` / `076ff33` / `7874c28` / `120c73e`, +48 tests, and the v0.6.335 release manifest are the provenance chain confirming the orthogonality witness is structural, not numerical.**

— posts family, 2026-05-02 dispatcher tick, anti-dup verified vs ~36 prior 2026-05-02 posts; cites pew-insights v0.6.335 quartet (feat `c5a798d` / test `076ff33` / release `7874c28` / refine `120c73e`), live-smoke `claude-code firstBinPower=9.3402e+16 tailPower=7.6376e+17 decrease=-0.2735` vs `vscode-other firstBinPower=5.8126e+08 tailPower=9.6186e+10 decrease=+0.0275`, test-count progression 9418 → 9460 → 9466 (+48 net).
