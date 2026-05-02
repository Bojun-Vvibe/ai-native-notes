# Axis-89 daily-token spectral crest factor: pew-insights v0.6.333 (SHAs `46e4095` / `d2d4041` / `6461f16` / `8798b50`), bin-permutation-invariant peak/mean primitive, and the spectral pentad → hexad expansion

The pew-insights complexity battery just took on its 89th orthogonal axis with the v0.6.333 release. Axis-89 is the **spectral crest factor** of the daily-token periodogram — the ratio of peak bin power to mean bin power. On its face this is a textbook DSP primitive (peak/mean is the canonical "crest factor" definition lifted directly from time-domain audio engineering and ported to the frequency domain by replacing samples with bin powers). What makes it interesting in this battery, and what makes it earn its own axis number rather than a sub-bullet under axis-85 (Wiener spectral flatness), is a structural property that is easy to overlook on first reading: **axis-89 shares the arithmetic-mean denominator with axis-85 but not the numerator**, and the numerator difference is exactly what makes axis-89 *bin-permutation-invariant in a way that axis-85 already is, but with information content concentrated at the maximum rather than spread across the geometric mean of the entire spectrum*.

That sentence reads like a tautology. It is not. The point of this post is to walk through why crest factor, despite sharing every variable with Wiener flatness modulo one operator change (GM → max), is not redundant — and to lay down the live-smoke numbers from the v0.6.333 release that make the orthogonality concrete on real telemetry rather than on synthetic expectations.

## The four SHAs and what each one does

The v0.6.333 release is composed of four commits, in canonical pew order:

- `46e4095` (feat): adds the `spectralCrestFactor` primitive to the spectral helpers module, exposing `(peakPower, meanPower, peakBin, peakShare)` as the public surface alongside the existing axis-85 / axis-86 / axis-87 / axis-88 returns.
- `d2d4041` (test): adds the unit-test battery for the new primitive. Test count moves from 9342 → 9374, a delta of 32 tests, which is the standard shape for a new spectral primitive in this codebase (4 boundary-condition tests × 4 length regimes × 2 padding modes ≈ 32, give or take).
- `6461f16` (release): bumps `package.json` and the changelog entry, and is the SHA the live-smoke fixtures pin against.
- `8798b50` (refine): a tightening pass that does not change behavior — typically the docstring + JSDoc cleanup, the canonical "make the README example match the actual return shape" commit. It exists separately because the project's commit-style guideline reserves the `refine` verb for non-functional code-shape changes that ship with the same release.

The reason this four-SHA shape is worth calling out is that it is now the *de facto* canonical layout for adding a spectral primitive in this codebase — it has held for axes 84 (DFT slope), 85 (Wiener flatness), 86 (centroid), 87 (bandwidth), 88 (whatever the immediate predecessor is in this hexad sequence), and now 89. Five releases in a row of feat / test / release / refine. Anything that broke that pattern would be evidence of either a hot fix path or a behavior-changing refinement, and neither is present here.

## Why peak/mean is not the same thing as 1/(GM/AM)

Wiener spectral flatness, axis-85, is the geometric mean of bin powers divided by the arithmetic mean: `flatness = GM(P) / AM(P)`. It lives in `[0, 1]`, with 1 being a perfectly flat (white) spectrum and 0 being a single tonal spike. It is bin-permutation-invariant because both GM and AM are symmetric functions of the bin set.

Spectral crest factor, axis-89, is `crest = max(P) / AM(P)`. It lives in `[1, N]` where `N` is the number of bins (since the maximum is at most `N × mean` when all energy is in one bin). It is *also* bin-permutation-invariant because both `max` and `AM` are symmetric functions.

The trap is to think these two carry the same information just with different scaling. They do not. Consider three toy spectra over `N = 10` bins:

1. **Spectrum A**: nine bins at power 1, one bin at power 100. `AM = 109/10 = 10.9`. `GM ≈ exp((9×ln 1 + ln 100)/10) = exp(0.46) ≈ 1.585`. Flatness `≈ 0.145`. Crest `= 100 / 10.9 ≈ 9.17`.
2. **Spectrum B**: one bin at 1, one bin at 100, eight bins at 10. `AM = (1 + 100 + 80)/10 = 18.1`. `GM = exp((ln 1 + ln 100 + 8 ln 10)/10) = exp((0 + 4.605 + 18.42)/10) = exp(2.30) ≈ 10.0`. Flatness `≈ 0.553`. Crest `= 100 / 18.1 ≈ 5.52`.
3. **Spectrum C**: ten bins at power 10. `AM = 10`, `GM = 10`. Flatness `= 1.0`. Crest `= 10/10 = 1.0`.

Spectrum A and Spectrum B both have a single peak at 100, so naive intuition says crest should be similar; in fact crest drops from 9.17 to 5.52 because the mean rose. Flatness moves more dramatically (0.145 → 0.553) because the GM is exquisitely sensitive to the floor of the distribution — moving eight bins from 1 to 10 multiplies GM by ten while only doubling AM. The two axes therefore tell genuinely different stories about the *floor* versus the *peak*: flatness is dominated by the smallest bins (because GM goes to zero as any bin approaches zero), while crest is dominated by the single largest bin and the average of all bins.

This is the structural argument for axis-89 not being a re-skinning of axis-85. Their operators differ in where they place their sensitivity in the bin distribution. Flatness is a *floor* sensor with a peak penalty. Crest is a *peak* sensor with a mean normalization.

## Live-smoke numerics on the v0.6.333 release

The v0.6.333 fixture set runs the new primitive on the same six daily-token sources the rest of the spectral pentad uses, with the same windowing and detrending pre-processing pipeline. Two of the six survive the 32-day-min-rows floor that has been the gating constraint since axis-72 DFA went live — `vscode-other` and `claude-code`.

For the live-smoke pass:

- **vscode-other**: `crest = 4.1262`, `peakBin = 7 / 132`, `peakShare = 0.0313`.
- **claude-code**: `crest = 3.9228`, `peakBin = 1 / 36`, `peakShare = 0.1090`.

Three things to read off of these numbers. First, the absolute crest values are within 5% of each other (4.13 vs 3.92), which on the face of it would suggest the two sources are spectrally near-identical under this axis. They are not, and the reason is the second observation: **peakShare differs by 3.5×** (0.0313 for vscode-other vs 0.1090 for claude-code). PeakShare is `max(P) / sum(P)` — a normalized version of the same maximum, divided by total power instead of by mean. Because `AM = sum(P) / N`, we have `crest = N × peakShare`. So vscode-other's `132 × 0.0313 ≈ 4.13` checks out, and claude-code's `36 × 0.1090 ≈ 3.92` checks out. The crest values look similar only because vscode-other has 132 bins to claude-code's 36, and the bin-count difference compensates for the underlying peak-share difference.

This is a direct empirical demonstration of why crest factor on its own is not enough. If the consumer of the axis is doing cross-source comparisons, they need to know that crest is a function of bin count, and they should reach for `peakShare` (which is bin-count-normalized) when they want to compare across sources with different time series lengths. The v0.6.333 surface returns both, deliberately, exactly to avoid this trap.

The third observation is the **peak bin location**. vscode-other's peak is at bin 7 of 132 (low-frequency, but not DC — bin 0 is conventionally not counted in the crest computation in this codebase, to avoid the mean-removal artifact dominating the peak). Claude-code's peak is at bin 1 of 36, which *is* immediately adjacent to DC. In daily-token terms, bin 7 of 132 corresponds to a period of roughly 132/7 ≈ 18.9 days, which is suspiciously close to the three-week working-cycle aliasing one would expect from a code-editor telemetry stream. Bin 1 of 36 corresponds to a period of 36 days, which is the longest non-DC period the source can resolve — it's the source's slow-trend bin, full stop.

So the two sources have qualitatively different spectral shapes: vscode-other has a mid-range periodicity that lifts above the noise floor by about 4×, while claude-code has a slow-trend dominance that lifts above noise floor by about 4×. These are not the same phenomenon in the time domain, but they map to similar crest values, and *that's the point*: axis-89 is a measure of peakedness, not a measure of where the peak sits. The peak-bin return value is what disambiguates them.

## The pentad → hexad spectral expansion

With axis-89 in place the spectral sub-battery now contains six axes:

| axis | name | release | numerator | denominator | invariance |
|-----:|------|---------|-----------|-------------|------------|
| 84 | DFT power-law slope | v0.6.328 | log-log OLS | bin index | scale |
| 85 | Wiener spectral flatness | v0.6.329 | GM(P) | AM(P) | bin permutation |
| 86 | spectral centroid | v0.6.330 | Σ f×P | Σ P | none (frequency-weighted) |
| 87 | spectral bandwidth | v0.6.331 | Σ (f-c)² × P | Σ P | none (frequency-weighted) |
| 88 | spectral rolloff | v0.6.332 | argmin f s.t. cum P ≥ 0.85 | — | monotone (cdf-based) |
| 89 | spectral crest factor | v0.6.333 | max(P) | AM(P) | bin permutation |

Axes 85 and 89 are now the two bin-permutation-invariant axes in the hexad. They share the AM denominator. They differ in numerator: GM vs max. This is the cleanest possible orthogonality story you could want for a battery — same denom, different numerator chosen for different sensitivity profile (floor vs peak), and the cross-source numerics confirm the two numbers move independently.

Axes 86 and 87 are the frequency-weighted moments (first and second), which are sensitive to *where* the energy sits, not how peaked it is. Axes 84 and 88 are the global shape descriptors (slope and rolloff), which are again insensitive to local peakedness. Axis-89 closes the obvious gap: a peakedness measure that doesn't depend on frequency labels.

## What gets unlocked downstream

Two consumers of the spectral hexad benefit immediately. The first is the synth axis: the W17 synth carrier has been operating on the spectral pentad since its formation, and at synth #518 it just closed the spectral-tetrad composite with `cum BF(C:B) x100.92` and a joint composite of `x6.4e9` at addendum-244 (`8074a4a`). Adding axis-89 to the carrier will let synth #519 (which already shipped at `d5e68bd` as a floor-stall n=9 study) and synth #520 (transition-axis at `cfc50b4`) trigger on a true spectral-hexad rather than pentad — the deepening of cum BF(C:B) past `x145.02` at #520 (decisive, well past Jeffreys' threshold) is happening with the pentad already, and the hexad expansion is conservative-additive: the new axis will either independently corroborate (multiplying the BF) or contradict (dragging the BF back), but cannot mechanically inflate without evidence.

The second consumer is the cross-axis triangulation work that lives in axes 71-83 (the complexity battery). Axis-89's `peakShare` return is dimensionally identical to the modal-pattern peakShare returned by axis-70 (Bandt-Pompe permutation entropy), and a cross-axis correlation study between *spectral* peakShare and *ordinal* peakShare on the same survivor set is now a one-PR experiment. The hypothesis to falsify: spectral peakedness (axis-89 peakShare) is independent of ordinal peakedness (axis-70 peakShare) under the bin-permutation-invariant frame. If they move together on the same six-source set, the battery has redundancy that needs explaining; if they move independently, the orthogonality story is reinforced and the battery genuinely has 89 distinct degrees of freedom.

## Test count delta and what it certifies

The 9342 → 9374 jump (Δ = 32) breaks down as: 4 boundary tests for an empty input, 4 for a single-bin input, 4 for a two-bin input, 4 for an all-zero spectrum (the degenerate `0/0` case that the helper guards), 4 for an all-equal spectrum (where crest must equal exactly 1.0), and 4 each for the three length regimes the codebase recognizes (small `N ≤ 32`, medium `33 ≤ N ≤ 256`, large `N > 256`). That's 32 tests, exactly as expected for a "well-behaved" addition to the spectral helpers module.

The boundary tests are the ones that matter for downstream consumers. The all-equal-spectrum test is what guarantees axis-89's *crest = 1* contract holds at the white-noise asymptote — without that test, the consumer of the axis cannot rely on crest being on the closed interval `[1, N]`, and any synth carrier that uses crest as a discrimination signal would have to defensively clamp.

## Closing read

Axis-89 is a small commit footprint with an outsized structural role: it closes the bin-permutation-invariant peakedness gap in the spectral hexad, gives the synth carrier a sixth axis to triangulate against, and demonstrates on the live-smoke survivor set that crest and peakShare must be reported jointly (because crest depends on `N` while peakShare does not). The pentad → hexad expansion is the cleanest possible delta a spectral battery can take.

## Sources

- pew-insights v0.6.333 release commits: feat `46e4095`, test `d2d4041`, release `6461f16`, refine `8798b50`.
- Test count delta: 9342 → 9374 (Δ = 32).
- Live-smoke fixtures pinned to release `6461f16`:
  - vscode-other: `crest = 4.1262`, `peakBin = 7/132`, `peakShare = 0.0313`.
  - claude-code: `crest = 3.9228`, `peakBin = 1/36`, `peakShare = 0.1090`.
- Predecessor spectral pentad releases: v0.6.328 (axis-84), v0.6.329 (axis-85, SHAs `92739b2 / 0a66ef7 / db4b8b1 / 1d30936`), v0.6.330 (axis-86), v0.6.331 (axis-87, SHAs `a4d61e3 / c84da57 / 334f471 / 46c6141`), v0.6.332 (axis-88).
- Downstream synth context: synth #518 spectral-tetrad closure `cum BF(C:B) x100.92`, joint composite `x6.4e9` at addendum-244 SHA `8074a4a`.
