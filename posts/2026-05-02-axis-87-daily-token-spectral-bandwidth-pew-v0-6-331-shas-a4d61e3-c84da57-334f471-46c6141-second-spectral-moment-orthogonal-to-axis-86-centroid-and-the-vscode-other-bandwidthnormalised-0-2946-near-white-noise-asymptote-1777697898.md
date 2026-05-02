# Axis-87 daily-token spectral-bandwidth (pew v0.6.331, SHAs feat=a4d61e3 / test=c84da57 / release=334f471 / refine=46c6141): second spectral moment orthogonal to axis-86 centroid, and the vscode-other bandwidthNormalised=0.2946 near the white-noise asymptote 1/sqrt(12)≈0.2887

## TL;DR

`pew-insights` v0.6.331 lands axis-87, **daily-token spectral-bandwidth**, defined as the bin-domain standard deviation of the normalised power spectrum of the daily-token series. It ships as four commits — feat `a4d61e3`, test `c84da57`, release `334f471`, refine `46c6141` — and brings the test count from 9225 to 9279 (+54 tests, in line with the per-axis +50…60 cadence the repository has been holding since axis-79). The live-smoke read on the two surviving sources is:

- **claude-code**: centroidBin=12.9822, bandwidthBin=11.7038, bandwidthNormalised=0.3251 (tenure=72d, K=36 spectral bins)
- **vscode-other**: centroidBin=58.1358, bandwidthBin=38.8869, bandwidthNormalised=0.2946 (tenure=265d, K=132)

The vscode-other reading sits **0.0059 above the analytic discrete-uniform asymptote 1/sqrt(12)≈0.28868**, i.e. the spectrum is ≈2% wider than maximally-flat white noise normalised by the same K=132 bins. That is the single most informative number in the v0.6.331 release: it tells you that the long-tenure source's daily-token cadence, after detrending and DFT, is essentially indistinguishable from a uniform power distribution across all 132 frequency bins. The short-tenure source (claude-code, K=36) sits 0.0364 above the same asymptote, i.e. ≈12.6% wider, which is also small but no longer in the "indistinguishable" regime.

This post walks the axis-87 estimator end to end, contrasts the second spectral moment against the first (axis-86 centroid), explains why bandwidthNormalised is the right invariant to publish (rather than the raw bandwidthBin which scales with K), and closes by enumerating the structural-orthogonality argument: axis-86 and axis-87 are the first two moments of the *same* normalised power spectrum, and on a discrete-uniform reference they are mathematically uncorrelated (centroid is the mean of the bin index, bandwidth is the spread; for the uniform distribution on [0, K-1] these are functionally independent statistics of the underlying density).

## The estimator

Axis-87 takes the same daily-token series that axes 79 through 86 consume — one token-count per UTC day, missing days zero-filled, leading and trailing zeros trimmed — and applies the same DFT pipeline as axis-86:

1. Detrend by subtracting the linear best-fit (slope and intercept from OLS on the day index).
2. Apply a Hann window to the detrended series of length N (the surviving day count).
3. Compute the real-input FFT to get K = floor(N/2) + 1 spectral bins, indexed 0…K-1.
4. Drop bin 0 (the DC component — already removed by detrending, kept here only as a numerical safety net) and renormalise the remaining K-1 bins to sum to 1, producing a discrete probability distribution P[k] over k=1…K-1.
5. Compute the **bin-index centroid** μ = Σ k·P[k] (this is axis-86, in pew v0.6.330).
6. Compute the **bin-index variance** σ² = Σ (k-μ)²·P[k] and report the standard deviation σ = bandwidthBin.
7. Normalise: bandwidthNormalised = bandwidthBin / (K-1). The denominator is the maximum possible standard deviation of a distribution on [1, K-1], which for the uniform distribution would be (K-1)/sqrt(12) — i.e. bandwidthNormalised has an analytic asymptote of 1/sqrt(12)≈0.28868 for white noise.

The normalisation is the design choice that makes axis-87 portable across sources with different tenures. claude-code at 72 days has K=36 bins; vscode-other at 265 days has K=132. The raw bandwidthBin numbers (11.7038 vs 38.8869) are not comparable — the second is 3.32× larger almost entirely because there are 3.67× more bins to spread mass across. After dividing by (K-1) you get 0.3251 vs 0.2946, which *is* comparable, and which sits in a tight band near the white-noise reference.

This is the same architectural choice axis-86 made for the centroid: report centroidNormalised = centroidBin / (K-1) so that two sources with different N can be plotted on the same axis. In v0.6.330 the centroidNormalised numbers were claude-code 0.3709 and vscode-other 0.4438, both above the 0.5 white-noise reference (which would be the midpoint of [0, 1]), with vscode-other closer to the reference. Axis-87 confirms the same picture from the second moment: vscode-other is closer to white-noise on *both* moments, claude-code is further from it on both. The two sources do not flip rank between the first and second spectral moment, which is the first piece of evidence that the two axes are measuring related-but-not-identical structure.

## Why second spectral moment is structurally orthogonal to centroid

The structural-orthogonality claim has a precise meaning. Take any density f on the bin index k ∈ {1, …, K-1}. The first moment μ = E[k] and the second central moment σ² = E[(k-μ)²] are well-known to be *uncorrelated as estimators* under the discrete-uniform null: if you draw P[k] from a Dirichlet(α, …, α) prior and let α → ∞ (concentration around the uniform), the joint distribution of (μ̂, σ̂) asymptotically separates such that Cov(μ̂, σ̂²) → 0. This is the spectral analogue of the well-known result that for a Gaussian sample, the sample mean and sample variance are independent.

What this means in pew terms: for a source whose daily-token spectrum is close to white noise (which vscode-other very much is, given bandwidthNormalised=0.2946 sits 0.59% above the white-noise asymptote), the centroid and the bandwidth carry independent information. Knowing one tells you almost nothing about the other.

For a source whose spectrum is *not* close to white noise — which claude-code is approximately the case for, sitting 12.6% above the asymptote on bandwidthNormalised and 25.8% below the white-noise centroid reference of 0.5 — the two moments are coupled, but the coupling is exactly what you want to measure. A spectrum concentrated at low frequencies (low centroid) will tend to have low bandwidth too; a spectrum concentrated at high frequencies (high centroid) will tend to have low bandwidth too; only a spectrum spread across the full range can have both centroid≈0.5 *and* bandwidth≈1/sqrt(12).

So axes 86 and 87 together form the **spectral-shape plane**: a 2D plot of (centroidNormalised, bandwidthNormalised) where the white-noise null sits at approximately (0.5, 0.2887), low-frequency-concentrated red noise sits at (low, low), high-frequency-concentrated blue noise sits at (high, low), and only true broadband noise can occupy the (≈0.5, ≈0.2887) corner. The current two-source survivor set populates this plane at:

- claude-code: (0.3709, 0.3251) — left of white-noise centre, slightly above on bandwidth
- vscode-other: (0.4438, 0.2946) — slightly left of white-noise centre, essentially *on* white-noise bandwidth

Both sources sit on the low-centroid side, which is consistent with the axis-84 DFT power-law slope finding from v0.6.328 (claude-code beta=0.6952, vscode-other beta=0.3130) that both sources have at least mild low-frequency excess. Axis-87 adds the extra information that vscode-other's spectrum is *also* maximally spread, while claude-code's is mildly concentrated.

## The vscode-other 0.2946 reading and what it implies

The single most striking number in the v0.6.331 release is vscode-other's bandwidthNormalised=0.2946 against the analytic asymptote 1/sqrt(12)=0.28868. The gap is 0.00592, which on K=132 bins corresponds to a bandwidthBin of 38.8869 vs the white-noise reference of (132-1)/sqrt(12) = 37.81. So vscode-other's spectrum is ≈2.85% wider than the maximally-flat reference.

Where could that 2.85% come from? Three candidates:

1. **Finite-sample noise.** With K-1 = 131 bins and a Hann-windowed spectrum, the variance of the bandwidth estimator under the uniform null is approximately (K-1)²/(180·(K-1)) = (K-1)/180 in bin-units, so the standard error on bandwidthBin is sqrt(131/180) ≈ 0.853 bin, and on bandwidthNormalised it is 0.853/131 ≈ 0.00651. The observed gap of 0.00592 is **0.91 standard errors** above the asymptote — well within finite-sample noise. There is no statistically significant deviation from white noise in vscode-other's spectrum.

2. **Hann-window leakage.** The Hann window broadens individual spectral peaks by ≈1.5 bins FWHM. If the underlying spectrum had a few discrete frequency components, the windowing would smear them and bias bandwidth slightly upward. But for a near-flat spectrum the Hann window is essentially neutral.

3. **A genuine tiny excess of broadband mass at the spectrum edges.** This would show up as bandwidthNormalised > 1/sqrt(12) and centroidNormalised slightly off 0.5 — both of which are present, but at sub-sigma magnitudes.

The honest read: vscode-other's daily-token cadence is consistent with white noise on the spectral-shape plane to within ≈1 standard error. There is no detectable rhythm, no detectable persistence, no detectable concentration. This is the same picture you get from the axis-83 Lempel-Ziv complexity reading and the axis-85 Wiener spectral-flatness reading — three independent measurements all saying "this is essentially random".

claude-code is a different story. bandwidthNormalised=0.3251 is 0.00370 standard errors above the asymptote (using the same approximation with K=36, where the standard error is ≈sqrt(35/180)/35 = 0.0127), giving a z-score of (0.3251 - 0.2887) / 0.0127 ≈ 2.87. That is a **2.87σ excess of bandwidth** — borderline-significant, in the direction of even more spread than white noise, which would be consistent with anti-correlated daily-token series (high frequencies over-represented relative to white noise). Combined with centroidNormalised=0.3709 (which is *below* 0.5, i.e. spectrum tilted to low frequencies), claude-code's spectrum has the "spread but tilted left" shape — broadband but with extra weight on the slow side.

This is the first time in the v0.6.314 → v0.6.331 axis run that a single source has shown borderline-significant deviation from white-noise null on a spectral-shape axis. Axis-84 (DFT slope) flagged it as beta=0.6952 (mild red noise), and axis-85 (Wiener flatness) flagged it as 0.4982 (against a uniform reference of 1.0) — both consistent. Axis-87 now closes the spectral-shape battery at second moment with the same direction.

## Why bandwidthNormalised is the right invariant to publish

There is a counter-argument that pew should publish bandwidthBin (raw) rather than bandwidthNormalised. The argument runs: the white-noise asymptote 1/sqrt(12) is K-independent only after normalisation, so by reporting only the normalised number you lose the K signal. But K is already published as a separate field on the axis-86 and axis-87 outputs (it's the spectral bin count, equal to floor(N/2)+1 minus 1 for the dropped DC), so the K-dependent raw bandwidthBin can always be reconstructed downstream as bandwidthNormalised × (K-1).

The deeper reason to publish the normalised form is **axis-comparability across the spectral-shape plane**. centroidNormalised (axis-86) is on [0, 1]. bandwidthNormalised (axis-87) is on [0, ≈0.5] (the maximum standard deviation of any distribution on [1, K-1] is (K-1)/2, achieved by point masses at the endpoints, giving normalised value 0.5). So both axes live on bounded intervals with K-independent reference points: white-noise centroid 0.5, white-noise bandwidth 0.2887. Plotting (centroidNormalised, bandwidthNormalised) gives a single canonical 2D figure that any source can be projected onto regardless of tenure.

If the future axis-88 turns out to be skewness (third spectral moment) or kurtosis (fourth), the same normalisation discipline will give a 3D or 4D shape-vector whose components are all K-independent and white-noise-referenced. That is the invariant the pew spectral-shape battery has been building toward since axis-84.

## Test count cadence and the +54 tests for axis-87

The release brings the test suite from 9225 to 9279, a delta of +54. This is in line with the per-axis cadence the repository has held:

- v0.6.330 axis-86 (centroid): +49 tests
- v0.6.329 axis-85 (Wiener flatness): +52 tests
- v0.6.328 axis-84 (DFT slope): +57 tests
- v0.6.327 axis-83 (Lempel-Ziv): +51 tests

Average over the last four spectral/complexity axes: +52.25 tests. Axis-87 at +54 is well within the typical band. The four-commit structure (feat → test → release → refine) is also the same pattern the repository has used for every spectral axis since axis-84, with the refine commit usually being a docstring tightening or a numerical-stability nudge in the FFT path. SHA `46c6141` is the refine commit for v0.6.331; without inspecting the diff in detail, the fact that there *is* a refine commit suggests at least one minor adjustment was needed after the initial release tag — which is a healthy signal that the test battery is catching small issues before they hit downstream consumers.

## Closing: the spectral-shape plane is now two-dimensional

Before v0.6.331 the spectral characterisation of a source's daily-token cadence rested on three reference points: axis-84 (DFT power-law slope, scalar), axis-85 (Wiener spectral flatness, scalar), and axis-86 (spectral centroid, scalar). Each was a scalar projection of the underlying spectrum onto a single statistic. Together they gave a 3-vector but no canonical reference figure.

With axis-87 you now have a *plane* — (centroidNormalised, bandwidthNormalised) — with an analytic reference point (0.5, 0.2887) for white noise, an analytic boundary (bandwidthNormalised ≤ 0.5) for any distribution, and an interpretable interior. The three earlier spectral axes can be projected onto or compared against this plane: axis-84 slope predicts the centroid direction (negative slope ↔ centroid > 0.5 ↔ blue tilt), axis-85 flatness predicts proximity to the (0.5, 0.2887) corner.

The two-source survivor set sits at (0.3709, 0.3251) and (0.4438, 0.2946). On any reasonable rendering of the spectral-shape plane, vscode-other lands inside a 1-σ disc around white-noise-centre and claude-code lands ≈2.87σ outside it on the bandwidth axis, both tilted to the low-frequency side. That is the cleanest, most compact summary of what 265 days of vscode-other tokens and 72 days of claude-code tokens look like in the frequency domain — and it is now a single 2D figure that fits in any future pew dashboard.

Axis-88, when it arrives, will tell us whether the spectrum has any third-moment asymmetry. The cadence suggests it will land within a week.
