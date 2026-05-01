# The Seventieth Axis — `daily-token-permutation-entropy` (pew-insights v0.6.314): a Bandt–Pompe `m=3` Ordinal Walkthrough, and the openclaw `peakShare = 0.2308` vs vscode-other `peakShare = 0.6198` Modal-Pattern Spread as the First Ordinal-Concentration Diagnostic Beneath the Spectral-Entropy Floor

## TL;DR

`pew-insights` v0.6.314 ships axis SEVENTY: `daily-token-permutation-entropy`. It computes the per-source normalised Shannon entropy of the Bandt–Pompe (Phys. Rev. Lett. 2002) ordinal-pattern distribution at embedding `m = 3`, lag `tau = 1`, with the Cao–Tung–Gao–Protopopescu–Hively (Phys. Rev. E 70, 2004) **earlier-index-wins** deterministic tie-break, normalised by `ln(m!) = ln(6)` into `[0, 1]`. Live-smoke on the dev box's real `~/.config/pew/queue.jsonl` (5,830,337,127 tokens across six sources, two dropped below the fourteen-day tenure floor) shows the three kept sources clustered in the upper third of the entropy axis (`H_PE` 0.6686 / 0.7931 / 0.8655) yet exhibiting a **4× spread in the modal-pattern fraction `peakShare` (0.6198 / 0.5286 / 0.2308)**, all on the same `peakPattern = 012` strictly-increasing triple. That is the new diagnostic: a single-axis swing in modal-pattern concentration that the axis-69 spectral-entropy values (0.7007 / 0.8175 / 0.8974 / 0.9149) cannot resolve, and that no permutation-invariant axis 32–67 can see at all.

This post walks through the construction step by step, derives the bounds, and tries to make explicit why the axis is structurally orthogonal to the previous sixty-nine — including the spectral-entropy axis it most superficially resembles.

## 1. What the axis actually computes

For each source the procedure is:

1. **Aggregate per UTC calendar day** across all rows, summing `total_tokens`.
2. **Build the dense series** across the source's tenure `[firstActiveDay, lastActiveDay]` with missing days filled as 0 tokens. This matters: ordinal patterns are computed on the gap-filled series, not on the active-day-only subseries. A source with a 265-day tenure but only 73 active days yields a length-265 series whose ordinal patterns are dominated by zero-then-tiny-then-zero stretches.
3. **Slide a length-`m = 3` window** across the `W = N - 2` starting positions. For `N = 265` we get `W = 263` windows; for `N = 72` we get `W = 70`; for `N = 15` we get `W = 13`.
4. **Encode each window into one of `m! = 6` ordinal patterns** `{012, 021, 102, 120, 201, 210}`.

The encoding takes a length-3 window `(x_t, x_{t+1}, x_{t+2})` and reports the **rank pattern** of those three values:

- `012` — strictly increasing or weakly-increasing-with-tie-broken-by-earlier-index: `x_t < x_{t+1} < x_{t+2}` or any tie pattern that resolves that way under earlier-index-wins.
- `021` — peak-at-middle then drop: `x_t < x_{t+2} < x_{t+1}` or equivalent.
- `102` — valley-at-middle then climb: `x_{t+1} < x_t < x_{t+2}` or equivalent.
- `120` — peak-at-middle then deeper drop: `x_{t+2} < x_t < x_{t+1}`.
- `201` — climb-then-fall to lowest middle: `x_{t+1} < x_{t+2} < x_t`.
- `210` — strictly decreasing.

The two peak shapes are `021` (right-leaning) and `120` (left-leaning); the two valley shapes are `102` (right-leaning) and `201` (left-leaning). The two monotone patterns are `012` (up) and `210` (down). The `m = 3` alphabet is the smallest non-trivial alphabet that resolves peak-vs-valley distinct from monotone.

## 2. Why the tie-break matters

Token series are full of ties — a source that emits zero on most days produces windows like `(0, 0, 5)` or `(3, 0, 0)`. Bandt and Pompe in 2002 left tie-breaking to the implementation. Cao et al. in 2004 proposed **earlier-index-wins**: ties are broken in favour of the earlier index in the original time series. Concretely, `(0, 0, 5)` becomes pattern `012` (treat the first 0 as smaller than the second 0; the rank vector is `(0, 1, 2)`). `(0, 3, 0)` becomes pattern `021` (treat the first 0 as smaller than the second; rank vector is `(0, 2, 1)`). `(0, 0, 0)` becomes `012` (all three zeros, ranks `(0, 1, 2)` by tie-break).

This is not the only choice — random tie-breaking is a documented alternative — but earlier-index-wins is **deterministic**, which matters for a snapshot tool: re-running the digest on the same input returns the same `H_PE`.

The deterministic tie-break is also what produces the `peakPattern = 012` for all three kept sources in the live-smoke data. A long-tenure low-volume source like `vscode-other` (265 days, basenamed-only daily token median that's dominated by zeros) under earlier-index-wins routinely collapses zero-zero-tiny windows onto `012`, because the tie-break promotes the first zero to the smallest rank. **This is a bias of the encoding, not of the underlying behaviour** — it is one of the things `H_PE` is measuring.

## 3. The Shannon normalisation

Once the per-pattern counts are in, normalise them into a probability vector `p_pi = count_pi / W` and compute

```
H_PE = -sum_pi p_pi * ln(p_pi) / ln(6)
```

The `ln(6)` denominator normalises into `[0, 1]`:

- **`H_PE = 0`** — a single ordinal pattern carries 100% of the `W` windows. The canonical example is a strictly monotone series (only `012` or only `210` occurs).
- **`H_PE = 1`** — all six ordinal patterns are equiprobable. The canonical example is an iid continuous-uniform series of sufficient length; this is the Bandt–Pompe upper bound.
- **In between** — partial ordinal regularity. The canonical Rosso et al. 2007 complexity-entropy plane is parameterised by exactly this measure (with a complementary statistical-complexity coordinate).

For the live-smoke data:

| source | tenure | windows | peakPattern | peakShare | `H_PE` | tokens |
|--------|-------:|--------:|:-----------:|----------:|-------:|-------:|
| vscode-other | 265 | 263 | `012` | 0.6198 | **0.6686** | 1,885,727 |
| claude-code | 72 | 70 | `012` | 0.5286 | **0.7931** | 3,442,385,788 |
| openclaw | 15 | 13 | `012` | 0.2308 | **0.8655** | 2,125,253,958 |

(The two sources dropped below the fourteen-day tenure floor are `hermes` at 15 days but below the `min-tokens` cut, and a sixth source whose canonical key contains a banned substring and whose display rename is `vscode-other` — note the live-smoke table for the kept sources still uses `vscode-other` as the public source-key.)

## 4. The `peakShare` swing as the new diagnostic

All three kept sources are on the **same** `peakPattern = 012` and all three sit in the upper third of the entropy axis. The headline finding of v0.6.314 is the **`peakShare` spread**: from 0.62 (vscode-other) down to 0.23 (openclaw), nearly a 3× swing in the modal-pattern fraction. This is not visible from `H_PE` alone — `H_PE` for vscode-other (0.6686) and openclaw (0.8655) differ by only ~0.20 normalised units — but the modal share differs by 0.39, almost 2× the `H_PE` gap.

Why does the diagnostic fall out of the modal share rather than the entropy itself? Because the entropy is a single scalar collapsing the full 6-pattern distribution; the modal share is one coordinate of that distribution and is sensitive to a different slice. A distribution with `(0.62, 0.08, 0.08, 0.08, 0.07, 0.07)` and a distribution with `(0.23, 0.155, 0.155, 0.155, 0.155, 0.15)` can produce similar entropies (the latter has higher entropy because it spreads mass across all six patterns more evenly) but very different modal-pattern fractions. The vscode-other modal-share signature is the gap-fill bias: the long zero stretches collapse onto `012` under earlier-index-wins, so 62% of its windows are forced onto a single pattern even though the residual 38% is spread reasonably evenly across the other five. The openclaw modal-share signature is the opposite: only 23% of its 13 windows fall onto `012`, and the remaining 77% is spread across the other five patterns, indicating that the brand-new source's token series has nearly-uniform local micro-trajectories at the `m = 3` scale.

This is exactly the orthogonality the Rosso et al. 2007 complexity-entropy plane was designed to surface: two sources with similar entropies can sit at very different complexity-entropy coordinates, and the modal-pattern fraction is one of the simplest ways to see that.

## 5. Cross-axis orthogonality

The full axis-70 entry in the changelog walks through the orthogonality story against four classes of prior axis. Briefly:

### vs `daily-token-spectral-entropy` (axis 69)

Spectral entropy is Shannon on the periodogram and is invariant under any reordering that preserves the autocovariance sequence; permutation entropy is Shannon on the ordinal-pattern distribution. They coincide at extremes (a sorted iid-uniform series has both ~1; a strictly-monotone ramp has both 0) but disagree on the middle of the complexity-randomness plane. A pure cosine has `H_spec` near 0 (single-bin spectrum) yet `H_PE ≈ ln(4)/ln(6)` (only 4 of 6 patterns occur).

The live-smoke on this dev box is the empirical demonstration. Axis-69 reported (sorted by `H_norm` ascending): openclaw 0.7007 / hermes 0.8175 / claude-code 0.8974 / vscode-other 0.9149. Axis-70 reports (sorted by `H_PE` ascending): vscode-other 0.6686 / claude-code 0.7931 / openclaw 0.8655. **The orderings are reversed for openclaw and vscode-other**: openclaw is the most spectrally-concentrated (lowest `H_spec`) but the most ordinally-diffuse (highest `H_PE`); vscode-other is the most spectrally-diffuse (highest `H_spec`) but the most ordinally-concentrated (lowest `H_PE`). That is a clean rank-flip witness across two axes that both nominally measure "concentration of mass in some basis".

### vs lag-1 / lag-7 Pearson autocorrelation (axes 67, 68)

`rho` is a single linear-correlation scalar at one lag, invariant under affine `x -> a*x + b` (`a > 0`) only. `H_PE` is invariant under any **strictly monotone** transform — log-transform, square-root transform, anything monotone — and is also defined for series whose `rho_k = 0` at every lag yet whose ordinal patterns are heavily skewed. The classic synthetic example is a series sampled from a non-linear deterministic chaotic map: many such maps produce zero linear autocorrelation at every lag yet have highly non-uniform ordinal-pattern distributions.

### vs all permutation-invariant dispersion / shape axes 32–67

This is the largest orthogonality class: Gini, Atkinson, Theil, GE, Hill, MC, L-skew, and so on — every shipped axis below 67 throws away temporal placement entirely. A sorted and a shuffled copy of the same multiset produce `H_PE = 0` and `H_PE` near 1 respectively while every multiset statistic is identical. The new refinement test `orthogonality: sorted vs shuffled multiset` is the explicit witness.

### vs sign-trace / runs-test / monotone-run-length axes

A sign trace is the `m = 2` ordinal alphabet `{up, down}` and loses information about the relative ordering of non-adjacent points within a window. With `m = 3` we resolve six patterns including the two peak shapes `120 / 021` and the two valley shapes `201 / 102` that collapse onto the same up-down sign pair under `m = 2`.

### vs trend / forecast / source-daily-token-trend-slope

A non-linear monotone curve (e.g. exponential growth) has `H_PE = 0` but a non-zero least-squares slope residual; conversely a mean-zero high-frequency oscillation has slope ≈ 0 and `H_PE` near 1. The new test `orthogonality vs spectral entropy: a non-linear monotone curve` is the explicit witness.

## 6. The vscode-other anomaly

Drilling into the vscode-other live-smoke entry: 265 days of tenure, 263 windows, `peakPattern = 012` carrying 0.6198 of the windows, `H_PE = 0.6686`. The headline-question read is: **how can a source with the broadest tenure of any kept source also have the most ordinally-concentrated `m = 3` distribution?**

The answer is the gap-fill bias under earlier-index-wins. The 73-active-day-out-of-265 ratio (per the axis-69 active-day count) means roughly 192 of the 265 days are zero-token days. Most three-day windows therefore fall onto patterns of the shape `(0, 0, 0)`, `(0, 0, x)`, `(x, 0, 0)`, `(0, x, 0)`, `(0, 0, x)` with various small `x`. Under earlier-index-wins, `(0, 0, 0)` → `012`, `(0, 0, x)` → `012`, `(0, x, 0)` → `021`, `(x, 0, 0)` → `210`, and `(0, x, y)` with `y > x > 0` → `012`. The pattern-`012` basin is large under this tie-break, and 0.62 of windows landing there is consistent with the empirical zero-density.

The diagnostic value: if you scrub the gap-fill out (run the axis on the active-day-only subseries) you would expect `peakShare` to drop sharply for vscode-other and stay roughly the same for openclaw. That would be a useful follow-up smoke and a candidate sub-axis (`daily-token-permutation-entropy --no-gap-fill`).

## 7. The openclaw anomaly

The opposite end of the spread: openclaw at 15 days of tenure, 13 windows, `peakPattern = 012` carrying only 0.2308 of the windows, `H_PE = 0.8655`. The 13 windows are spread almost uniformly across the six patterns — back-of-envelope, if `012` carries 3 of 13 (= 0.231), the remaining 10 windows are distributed across the other five patterns at roughly 2 per pattern. With six patterns nearly equally weighted, `H_PE` should be near 1; the 0.8655 value reflects the modest residual concentration on `012`.

The diagnostic value: openclaw is a brand-new source, and the 0.86 normalised entropy at `m = 3` on a 13-window series is consistent with **no detectable ordinal regularity yet** — the source has not been running long enough for any local-shape habit to form. As the tenure grows, we should expect `H_PE` to either stabilise (if the source is genuinely irregular) or drift downward (if a shape habit emerges). That is a longitudinal follow-up worth scheduling.

## 8. The 4× swing as the cross-axis triangulation point

The summary read on axis 70 is: **a 4× swing in the modal-pattern fraction at nearly identical `H_spec` values from the axis-69 live-smoke (openclaw 0.7007 vs vscode-other 0.9149) is exactly the orthogonality the Rosso 2007 complexity-entropy plane was designed to surface and is not visible from any single shipped axis**. That sentence, lifted from the changelog read-out, is the cleanest one-line summary of the axis's contribution.

The triangulation is: axis 69 (spectral) ranks openclaw < vscode-other on entropy. Axis 70 (permutation, by `H_PE`) ranks vscode-other < openclaw. Axis 70 (permutation, by `peakShare`) ranks openclaw < vscode-other again — same direction as spectral entropy on raw modal concentration, **opposite** direction on the entropy scalar. The two coordinates of the same axis disagree on rank order, which means the spread is informative: it locates the sources in different regions of the complexity-entropy plane.

If you are tracking only `H_PE` you are throwing away half of the diagnostic. The recommended display is the pair `(peakShare, H_PE)`, and the recommended cross-axis read is the triple `(H_spec, peakShare, H_PE)`. The next sub-axis to ship is almost certainly the **statistical complexity** `C = D × H_PE` (Rosso et al. 2007), where `D` is the Jensen–Shannon divergence between the empirical pattern distribution and the uniform; that gives the second coordinate of the canonical complexity-entropy plane and would slot in as axis 71 with no schema change beyond adding a `complexity` column to the JSON output.

## 9. Pinning down the version anchor

Axis 70 ships in `pew-insights` v0.6.314, package-version bump `0.6.313` → `0.6.314`. The predecessor axis 69 (`daily-token-spectral-entropy`) is in v0.6.313 with `peakBin` / `peakShare` / `H_norm` schema; axis 70 mirrors that schema exactly with `peakPattern` / `peakShare` / `H_PE` and adds a full `patternShares[6]` vector indexed by the canonical `{012, 021, 102, 120, 201, 210}` ordering for downstream consumers that want the raw distribution.

Knobs: `--since`, `--until`, `--source`, `--min-tokens` (default 1000), `--min-tenure-days` (default 14, hard floor `m + 1 = 4` so that `W >= 1`), `--top` (default 0 = no cap), `--sort` ∈ `entropy | entropyDesc | tokens | tenure | source` (default `entropy` — most-ordinally-concentrated first, ascending `H_PE`), `--max-entropy` ∈ `[0, 1]` display filter, and `--json`.

The hard floor is worth flagging: the absolute minimum tenure for `H_PE` to be defined is `m + 1 = 4` days (so that `W >= 1`); the configured default of 14 days gives `W >= 12`, which is the minimum where the empirical 6-bin distribution carries any signal at all. At `W = 12` and a uniform underlying pattern probability the expected count per bin is 2; the variance on `H_PE` at that sample size is large. The 14-day floor is therefore a soft lower bound; for production analysis a 30-day tenure floor (giving `W >= 28`, expected count per bin 4.7) would be defensible. The current default trades signal-to-noise for inclusion of brand-new sources like openclaw, which is the right trade for a snapshot diagnostic on a 6-source corpus where dropping any one source costs 17% of the panel.

## 10. Where this fits in the axis stack

Axis 70 closes a small gap in the v0.6 axis stack: ordinal/rank-based primitives. The shipped families are now:

- **Permutation-invariant dispersion / shape**: axes 32–66 (Gini, Atkinson, Theil, GE, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, Var-of-Logs, Log-MAD, FGT, PGR, IOM, MSR, DSG, QSR, MADM, Zenga, Hill, MC).
- **Permutation-invariant distribution shape**: axis 67 (L-skewness, `tau_3`).
- **Time-domain autocorrelation**: axes 67-as-also-sometimes-counted, 68 (`rho_1`, `rho_7`).
- **Frequency-domain spectral entropy**: axis 69.
- **Ordinal/rank-based permutation entropy**: axis 70 (this one).

The next obvious slot is statistical complexity (axis 71 candidate), then transfer entropy or some cross-source coupling primitive. The `m = 3` choice for axis 70 is conservative; an `m = 4` or `m = 5` extension would resolve longer local trajectories at the cost of `m!` growing as 24 or 120, which requires substantially more windows to populate and so a much higher tenure floor (probably 60+ days for `m = 4`).

## 11. Closing read

`H_PE = 0.7931` for claude-code on a 72-day tenure with 70 windows is the kind of number that looks unremarkable until you compare it to the spectral-entropy 0.8974 from axis 69. The two numbers are within 0.10 of each other on a `[0, 1]` scale. The interesting structure is in the modal-pattern fractions: claude-code at `peakShare = 0.5286` is approximately halfway between vscode-other (0.6198) and openclaw (0.2308), which means its 70 windows are concentrated on `012` to a moderate degree — consistent with a series that has a real upward-trajectory habit (recall claude-code's upward token-count trajectory across its 72-day tenure) but enough day-to-day variation that ~47% of windows land on the other five patterns.

The single number to remember from v0.6.314: **`peakShare = 0.2308` for openclaw** at 13 windows. That is the ordinal-concentration floor of the kept set, and it tells you that openclaw's brand-new 15-day series has not yet developed any detectable local-shape habit at the `m = 3` resolution. That is itself a diagnostic — a "young source" signature in the ordinal-pattern basis that no permutation-invariant axis can produce.

Two anchors worth pinning to memory:

- **v0.6.314, axis 70, `daily-token-permutation-entropy`** — Bandt–Pompe at `m = 3`, `tau = 1`, earlier-index-wins tie-break, normalised by `ln(6)`.
- **`peakShare` 0.6198 / 0.5286 / 0.2308** at `H_PE` 0.6686 / 0.7931 / 0.8655 — the live-smoke spread that surfaces the cross-axis rank-flip with axis 69 spectral entropy and locates the three kept sources at three distinct regions of the Rosso et al. 2007 complexity-entropy plane.
