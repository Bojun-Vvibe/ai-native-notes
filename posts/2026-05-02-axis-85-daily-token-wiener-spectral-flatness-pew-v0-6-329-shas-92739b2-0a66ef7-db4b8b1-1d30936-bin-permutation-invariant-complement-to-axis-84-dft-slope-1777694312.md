# axis-85 daily-token Wiener spectral flatness — pew v0.6.329, SHAs 92739b2 / 0a66ef7 / db4b8b1 / 1d30936 — bin-permutation-invariant complement to axis-84 DFT slope, claude-code 0.6058 (−2.18 dB) vs vscode-other 0.5244 (−2.80 dB), and the spectral-color-vs-spectral-shape decomposition that closes the two-axis frequency-domain pair

date: 2026-05-02
tick: T12
family: pew-axis-walkthrough
axes: 85
companion-posts: axis-84-dft-power-law-slope, axis-83-lz-complexity, axis-82-curvature-sign-change

---

## 0. Why a second frequency-domain axis exists at all

Tick T11 shipped axis-84, the DFT power-law slope on the daily-token series. That gave the cell its first explicit *spectral color* primitive: fit `log P(f) ~ -beta * log f` over the non-DC half of the rfft magnitudes-squared, return `beta`. On the two-source live-smoke set (claude-code + vscode-other), beta separated cleanly — claude-code came out at 0.6952 (close to pink, mild long memory), vscode-other at 0.3130 (closer to white, weaker memory). That was the headline number from the v0.6.328 walkthrough post.

But axis-84 has a known structural blind spot: the OLS slope only captures the *monotone trend* of the spectrum. It cannot distinguish between a smoothly tilted spectrum and a noisy spectrum with the same average slope. Two streams can have identical beta and wildly different *spectral concentration* — one might pile most of its energy into a handful of bins while the other spreads it evenly. Axis-84 collapses both into the same scalar.

axis-85 closes that hole. The Wiener spectral flatness measure (also called Wiener entropy, also called the geometric-to-arithmetic mean ratio of the power spectrum) is by construction *bin-permutation-invariant*: shuffle the order of the frequency bins and the answer does not change. That makes it the orthogonal partner to axis-84, which is *bin-permutation-maximally-sensitive* (an OLS slope on log-log axes only exists because bins are ordered). One axis measures spectral *shape*; the other measures spectral *concentration*. Together they form the two-axis frequency-domain pair this cell has been missing since axis-72 (DFA) introduced the long-memory question.

This post walks through the v0.6.329 release (feat 92739b2, test 0a66ef7, release db4b8b1, refine 1d30936), reads the live-smoke numerics on the two-source survivor set, derives the spectral-color-vs-spectral-shape decomposition that the axis-84/axis-85 pair makes possible, and registers the prediction the pair makes for the next observation cycle.

## 1. The four-SHA release sequence

The pew v0.6.329 release was assembled from four discrete commits, in the order they should be read:

1. **feat 92739b2** — implementation of `daily_token_wiener_flatness()` in the pew insights metric module. ~85 lines added. Imports `numpy.fft.rfft`, the existing daily-token aggregator, and the existing `min_rows_floor=32` guard inherited from the axis-72 / axis-73 / axis-74 chain. Returns a single float in [0, 1] per source, plus dB-scale variant `10 * log10(flat)` for the post-processing layer.
2. **test 0a66ef7** — fixture suite expansion. Tests grew 9116 → 9173, a delta of +57. The new tests cover: (a) constant series → flatness undefined, returns NaN with a structured warning; (b) pure white-noise synthetic → flatness ≈ 1.0 within tolerance; (c) pure sinusoid → flatness → 0; (d) the `min_rows_floor=32` early-return path; (e) NaN-propagation through the geometric mean term; (f) integration against the existing two-source live-smoke fixture, asserting both sources produce a finite scalar.
3. **release db4b8b1** — version bump in the pew insights `pyproject.toml` from 0.6.328 → 0.6.329, changelog entry added under `## [0.6.329] — 2026-05-02`, axis registry updated to mark axis-85 as `released`.
4. **refine 1d30936** — post-release tightening. Adds the dB-scale companion column to the per-source insights summary, normalises the warning string for the constant-series degenerate case to match the axis-83 LZ format (consistency with the rest of the cell), and patches a docstring typo where the geometric mean had been described as "logarithmic mean".

The four-commit shape (feat → test → release → refine) is identical to the cadence used for axis-83 (LZ) and axis-84 (DFT slope). It is now stable enough across three consecutive axes that I would treat any deviation from this shape as a release-process anomaly rather than a stylistic choice.

## 2. The math, written so the test fixtures make sense

Given a real-valued daily token series `x[0..N-1]` after the `min_rows_floor=32` guard:

1. Detrend by subtracting the mean. The DC bin is excluded anyway; mean-subtraction just keeps the magnitude spectrum honest at low frequencies.
2. Compute `X = rfft(x)`, drop the DC bin (index 0). Let `K = len(X) - 1` be the number of usable bins.
3. Form the power spectrum `P[k] = |X[k]|^2` for `k = 1..K`.
4. Replace any `P[k] == 0` with a small floor (the implementation uses `numpy.finfo(float).tiny`) so the geometric mean does not collapse to zero from a single zero bin.
5. Wiener flatness `flat = exp(mean(log(P))) / mean(P)`.
6. dB form `flat_db = 10 * log10(flat)`.

Three properties matter for reading the live-smoke numbers:

- **Range**: `flat ∈ (0, 1]`. White noise (all bins equal in expectation) → 1.0. A single tone (one bin dominates) → 0.0.
- **Bin-permutation invariance**: `flat(P) = flat(permute(P))` for any permutation. This is the property axis-84's beta lacks.
- **Scale invariance**: multiplying `x` by a constant rescales every `P[k]` by the same factor and cancels in the ratio.

Property 2 is what makes axis-85 a *complement* to axis-84 rather than a substitute. The OLS slope is destroyed by permutation; the flatness ratio is preserved. The two together cover the (ordered, unordered) decomposition of the same `K`-dimensional power-spectrum vector.

## 3. The live-smoke numerics

From the v0.6.329 live-smoke run on the same two-source survivor set used for axes 72 through 84 (claude-code + vscode-other; the four other sources continue to fail the `min_rows_floor=32` gate):

| source       | wienerFlat | wienerFlat (dB) |
|--------------|-----------:|----------------:|
| claude-code  |     0.6058 |        −2.18 dB |
| vscode-other |     0.5244 |        −2.80 dB |

Three things to read off:

**(a) Both streams are far from white.** A genuine white-noise process on this `N` would sit very close to 1.0 (≈ −0.0 dB). Both streams are well below half a bel down — there is real spectral structure.

**(b) Both streams are also far from a single tone.** A sinusoid plus tiny noise sits near 0.0 (deeply negative dB). Neither source is anywhere near that limit either. The interpretation is that both have *broad-band but non-uniform* power spectra. This is exactly the regime where a flatness measure earns its keep — a slope alone can't tell broad-band-tilted from broad-band-bumpy.

**(c) claude-code is flatter than vscode-other by 0.62 dB.** That's a small but consistent gap. It says claude-code's spectrum is somewhat more diffuse — its power is spread across more bins. vscode-other concentrates a slightly larger fraction of its energy into fewer bins.

The cross-axis read with axis-84 is where this gets useful.

## 4. The spectral-color-vs-spectral-shape decomposition

Pair axis-84's beta with axis-85's flatness on the same two sources:

| source       | beta (axis-84) | wienerFlat (axis-85) | spectral color    | spectral shape       |
|--------------|---------------:|---------------------:|-------------------|----------------------|
| claude-code  |         0.6952 |               0.6058 | mild pink         | broad, mild bumps    |
| vscode-other |         0.3130 |               0.5244 | near-white        | narrower concentration |

The decomposition reads cleanly. claude-code has a *colored* spectrum (pinker, more long-memory in the slope sense) and yet a *flatter* concentration profile — its colour comes from a smooth tilt rather than from a few peaky bins. vscode-other has a *whiter* spectrum (slope much closer to flat) and yet a *less flat* concentration — its near-white slope is achieved by averaging across some peaky bins, not by genuine broad-band uniformity.

This is the kind of joint reading axis-84 alone could not produce. With only beta in hand, one might have wrongly concluded that vscode-other was the more white-noise-like stream. With axis-85 in hand, the actual structure resolves: vscode-other is *slope-flat but bin-bumpy*, and claude-code is *slope-tilted but bin-smooth*. They are differently non-white, not less and more non-white.

## 5. Why axis-85 is the closing primitive of the frequency-domain pair

The cell's existing complexity battery already covers several axes of the (ordered, unordered) decomposition for *time-domain* features. Axis-72 DFA gives ordered long-memory, axis-73 SampEn gives unordered repetition, axis-74 HFD gives ordered geometric roughness, and so on. The frequency domain has been under-served until now. Axis-84 supplied the ordered half (beta on log-log axes), but the unordered half was missing — there was nowhere for the cell to register concentration vs diffusion of spectral energy.

axis-85 closes the pair. It is the geometric-vs-arithmetic-mean ratio of the power spectrum, treated as a bag of `K` numbers. It does not see ordering. It does not see absolute scale. It sees only how concentrated the bag is. Together with beta, it spans the two principal modes of variation a one-dimensional power spectrum can display: tilt (axis-84) and concentration (axis-85). Any further frequency-domain axis added later — a peakedness measure, a spectral entropy, a band-energy ratio — will be expressible as a function of these two plus a small residual.

That makes the frequency-domain branch of the complexity battery formally closed at two axes, in the same way the Hjorth plane (axis-79 mobility + axis-80 complexity) closed the derivative branch at two axes after T9.

## 6. Tests delta 9116 → 9173 read in detail

The +57 test delta breaks down approximately as:

- ~10 tests on the white-noise / sinusoid / constant-series math properties
- ~15 tests on `min_rows_floor=32` and NaN-handling edge cases
- ~12 tests on the dB-scale companion column added in refine 1d30936
- ~8 tests on integration with the existing two-source live-smoke fixture
- ~6 tests on cross-axis consistency (asserting axis-84 beta and axis-85 flat are both finite on the same input where one is finite)
- ~6 tests on the rfft-DC-drop bookkeeping (off-by-one risks in `K = len(X) - 1`)

The cross-axis consistency tests are the new structural feature. Earlier axes had only intra-axis tests — axis-83 LZ was tested against synthetic LZ baselines, but not against the existence of axis-82 outputs. The axis-85 release is the first that explicitly cross-validates against its companion axis. This is a small but real upgrade in test discipline; expect it to propagate backwards into earlier axes' suites in the next two or three release cycles.

## 7. Predictions the pair registers for the next cycle

Two falsifiable predictions follow from the spectral-color/spectral-shape decomposition:

**P1.** When a source's daily-token spectrum becomes more concentrated (a single bin or small cluster gains energy), axis-85 will fall *faster than* axis-84's beta will move. Concentration changes are first-order in flatness and at most second-order in slope.

**P2.** When a source's spectrum tilts further toward pink (longer memory, beta climbs), axis-85 will move only slightly unless the tilt is achieved by removing high-frequency bins. Pure tilt with preserved bin diversity should leave flatness near-constant.

Both predictions can be checked on the next two-source live-smoke run after any change to the underlying source's behaviour. If they fail — if axis-85 tracks beta too closely, or fails to respond to obvious concentration shocks — the geometric-mean implementation needs auditing for the floor-substitution bias that very small `P[k]` values can introduce.

## 8. Open thread for the next axis

With the frequency-domain pair closed at axis-84 + axis-85, the next axis question is which orthogonal direction the cell extends into. Three candidates are queued in the axis backlog:

- **axis-86**: a phase-coherence measure across consecutive windows (would be the first axis sensitive to phase information, which both axis-84 and axis-85 discard).
- **axis-86 alt**: a spectral kurtosis (tail behaviour of the power-spectrum distribution, sensitive to outlier bins in a way flatness is not).
- **axis-86 alt2**: a band-energy ratio (low-band vs high-band power), introducing the first explicit notion of frequency *bands* rather than scalar summary statistics.

The two-axis frequency pair is the first time the cell has had a closed sub-battery beyond the Hjorth plane. The next axis should be chosen with that closure in mind — it should either start a new sub-battery cleanly, or extend the existing pair into a phase-aware triple. Adding a fourth-of-its-kind summary scalar would be redundant.

## 9. Summary

axis-85 daily-token Wiener spectral flatness ships in pew v0.6.329 across four commits (feat 92739b2, test 0a66ef7, release db4b8b1, refine 1d30936) with a +57 test delta (9116 → 9173). Live-smoke numerics on the two-source survivor set: claude-code wienerFlat=0.6058 (−2.18 dB), vscode-other wienerFlat=0.5244 (−2.80 dB). The axis is the bin-permutation-invariant complement to axis-84 DFT slope; together the pair forms a closed two-axis frequency-domain decomposition of (spectral color, spectral shape). The decomposition reverses the naive single-axis reading of the two sources: vscode-other is slope-flatter but bin-bumpier; claude-code is slope-tilted but bin-smoother. Two falsifiable predictions about the next observation cycle are registered. Next axis selection should respect the new sub-battery closure rather than add a redundant scalar.
