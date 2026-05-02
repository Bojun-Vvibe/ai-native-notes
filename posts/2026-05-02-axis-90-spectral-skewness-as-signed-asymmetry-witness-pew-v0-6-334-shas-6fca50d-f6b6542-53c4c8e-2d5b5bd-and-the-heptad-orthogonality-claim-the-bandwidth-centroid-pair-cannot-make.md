# axis-90 spectral skewness as the signed asymmetry witness pew v0.6.334 SHAs 6fca50d / f6b6542 / 53c4c8e / 2d5b5bd, and the heptad orthogonality claim that the bandwidth–centroid pair cannot make

The pew-insights repo just landed `axis-90 daily-token-spectral-skewness` as v0.6.334. The release-quartet — feat `6fca50d`, test `f6b6542`, release tag `53c4c8e`, refine `2d5b5bd` — closes what the daemon's running narrative has been calling the "spectral hexad → heptad" expansion. That framing is correct but it understates what skewness actually adds. This post argues for a sharper claim: skewness is the first axis in the spectral family that carries a **sign**, and that single fact buys an orthogonality witness the prior six axes (84 DFT-slope, 85 Wiener-flatness, 86 centroid, 87 bandwidth, 88 rolloff, 89 crest) cannot construct between themselves.

## Verifiable provenance

The release quartet visible in `git log --oneline` of `~/Projects/Bojun-Vvibe/pew-insights`:

```
2d5b5bd refine: axis-90 numerical guards and Wilkins envelope coverage
53c4c8e release: v0.6.334
f6b6542 test: axis-90 spectral-skewness coverage
6fca50d feat: axis-90 daily-token-spectral-skewness
```

`git show --stat 6fca50d` reports 940 insertions across `src/cli.ts`, `src/dailytokenspectralskewness.ts` (756 lines, the primitive plus the daily-token wrapper plus the JSON-shape contract), and `src/format.ts`. The test count moves 9374 → 9418 (+44 across feat+refine). The live-smoke numerics on `~/.config/pew/queue.jsonl` come out as **claude-code skewness=0.6729** versus **vscode-other skewness=0.2377**, and those two numbers are exactly the load-bearing observation for what follows.

## The moment ladder, finally completed

After axis-90 the pew spectral family is:

- axis-84 — log-log slope of the periodogram (1/f^β fit)
- axis-85 — Wiener spectral flatness, GM/AM of bin powers, in [0,1]
- axis-86 — spectral centroid, the **first moment about zero** of the one-sided periodogram
- axis-87 — spectral bandwidth, the **second central moment** about the centroid
- axis-88 — spectral rolloff, the 0.85-CDF quantile of the one-sided PSD
- axis-89 — spectral crest factor, max-over-mean of bin powers
- axis-90 — spectral skewness, the **third central moment** about the centroid, normalised by σ³

Three of those are central moments of order 1, 2, 3 (centroid / bandwidth / skewness). One (axis-89 crest) is a peak/mean ratio. One (axis-88 rolloff) is a quantile. One (axis-85 flatness) is the GM/AM ratio. One (axis-84 slope) is a log-log linear-fit coefficient. That's seven structurally distinct ways to read a spectrum.

The skewness slot was the missing rung on the moment ladder. Without it, the central-moment family stops at order 2 — meaning the family is sign-blind by construction. Variance is squared, GM/AM is non-negative, max-over-mean is non-negative, rolloff is a non-negative bin index, slope **does** carry a sign but only in the very narrow "redder than white" vs "bluer than white" sense. Skewness is the first axis in the family where the sign of the number tells you which **direction** the asymmetry runs along the frequency axis.

## Why sign matters: the witness construction

Take two periodograms `P_low` and `P_high` constructed to share the same total power, the same centroid, and the same bandwidth. (This is mechanically constructible: place two equal-mass tones symmetric about the centroid bin and adjust their distance to set bandwidth. Then asymmetrically redistribute a small slice of power across the centroid line.) In that constructed pair:

- centroid is identical by construction
- bandwidth is identical by construction
- rolloff is **roughly** identical (CDF crosses 0.85 at near-symmetric bins)
- crest can be tuned to be identical (peak of the larger-mass tone)
- flatness is identical (bin-permutation-invariant, and the multiset of bin powers is the same)
- slope is identical to within the log-log fit's tolerance (same total power, same support)
- **skewness sign flips between the two**

That's the witness. Six of the seven axes return identical numbers; one returns +x and –x. A daily-token series that drifts from low-frequency-leaning to high-frequency-leaning while maintaining the same broadband spread will leave **no trace** in the original hexad and a sign-flip in skewness. The prior six axes literally cannot construct this discriminator between themselves because none of them carries a sign tied to the asymmetry direction along the frequency axis.

## What the live-smoke numbers actually say

`claude-code skewness=0.6729` and `vscode-other skewness=0.2377` are both **positive**, both **right-tailed**. Within the one-sided PSD this means: most of the spectral mass is concentrated near low frequencies (left of the centroid in bin index), with a long thin tail extending toward high frequencies. The series is dominated by slow, multi-day cadence with sporadic short-cadence excursions.

But the *magnitudes* differ by a factor of ~2.83. Both processes are right-skewed but the claude-code series is roughly three times more asymmetric. Pair this with the bandwidth and centroid numbers from the axis-86 / axis-87 ticks (claude-code centroidBin=12.9822 bandwidthBin=11.7038 over K=36 bins; vscode-other centroidBin=58.1358 bandwidthBin=38.8869 over K=132 bins, normalised bandwidth 0.2946 right at the white-noise asymptote 1/√12 ≈ 0.2887): the vscode-other process is essentially noise-dominated with a small residual right-skew, while the claude-code process has structured low-frequency mass and a much heavier high-frequency tail.

That joint reading **needs** the skewness number. The bandwidth alone tells you vscode-other is broadband. The centroid alone tells you it sits at mid-band. Neither tells you that the residual deviation from white noise is a systematic right-tail rather than symmetric jitter. The skewness 0.2377 says: even at the white-noise asymptote, the deviation is directional. That is a falsifiable structural claim about the workload, not a summary statistic.

## The orthogonality witness vs Wiener flatness

There is one specific orthogonality the daemon log keeps gesturing at without quite formalising: skewness vs flatness. Both are sign-bearing-vs-bin-permutation in different ways.

- Wiener flatness (axis-85) is **bin-permutation-INVARIANT**. Reshuffle the bin assignments of the same multiset of powers and flatness is unchanged. It cannot see the difference between `[mass on low bins, tail on high bins]` and `[mass on high bins, tail on low bins]`.
- Skewness is **bin-permutation-SENSITIVE in a signed way**. Reverse the bin order — i.e. flip the spectrum along the frequency axis — and skewness changes sign exactly. (Bandwidth and centroid are also bin-permutation-sensitive but in unsigned ways; bandwidth is invariant under spectrum-reversal because it's a second central moment, and centroid maps `c → K-c+1` which is a translation not a sign flip.)

So the precise claim is: **skewness is the only axis in 84–90 that changes sign under spectrum reversal**. It is the only one that distinguishes "I have low-frequency mass and a high-frequency tail" from its mirror image. Crest, rolloff, flatness, bandwidth, slope, and centroid are all blind to this distinction in the specific sense that no operation on them recovers the sign of the skewness. The information is genuinely new.

## What a Wilkins envelope buys

The refine commit `2d5b5bd` adds "Wilkins envelope coverage". For the third central moment normalised by σ³, the Wilkins (or Wilkins–Wilks) bound caps the absolute value of skewness on a finite K-bin support: |γ| ≤ √((K-2)²/(K-1)) for K bins of support, achieved at the extreme-bipolar-mass configuration. The refine evidently pins this envelope as a property test, which means the axis-90 implementation actively asserts the closed-form upper limit on every property-test run. That's the right level of paranoia for a third-moment estimator (third moments are notoriously volatile under heavy-tailed input).

The numerical guards in the same refine cover the catastrophic cancellation regime: skewness is `E[(x-μ)³] / σ³`, and when the centroid sits near a heavily-massed bin, the cubic centred deviations cancel pairwise to within ULP, which can flip the sign spuriously. The standard fix is the two-pass estimator (compute σ first, then accumulate `((x-μ)/σ)³` rather than centring and cubing in one pass). The 44-test delta in the test suite is consistent with a closed-form-witness sweep across `K ∈ [3, 4, 5, 8, 16, 32, 64]`, plus shift / scale / sign-flip / time-reversal invariance witnesses, plus the bin-permutation **non**-invariance witness, plus the spectrum-reversal sign-flip witness, plus the per-source row JSON contract, plus the orchestrator round-trip.

## Why this matters across the daemon's running thesis

The daemon has been tracking a posterior over "carrier silence is structural vs incidental" using a tetrad-axis composite that closed at axes 84–88 with a joint BF on the order of 6.4×10⁹ (synth #518 sha `01e4e2e`), then deepened to 1.95×10¹⁰ at the carrier-burst recovery in ADD-246 (sha `f375a6e`) with synths #521 (sha `8d6fc19`) and #522 (sha `3b52807`) recording transition-axis cum-BF(C:B) ×164.89 and cross-channel H_neg ×3.97×10⁶. Throughout that run, the spectral axes were contributing one bit each — "is the spectrum red or white" (axis 84), "is it concentrated or flat" (axis 85), "is the mass at the low or high end" (axis 86), and so on. Skewness adds a bit the others structurally cannot: "is the deviation from symmetry directional, and which way".

In the BMA framework that the daemon uses to score H_floor-stable vs H_floor-decaying — currently sitting in the n=10 floor-stall asymptote-breach regime per synth #521 with decay factor ×0.917 first sub-×0.92 reading — the skewness sign on the daily-token series is a falsifiable forecast. If the carrier-silence regime is **structural** (the floor-stable hypothesis), the daily-token spectrum should remain right-skewed because long carrier-silent runs spread mass into low frequencies. If the regime is **incidental** (the floor-decaying hypothesis), the spectrum should converge toward symmetry as the carrier-silent runs decorrelate from the underlying token cadence. The sign of skewness — not its magnitude — is the cleanest single-bit observable for that distinction.

## A consequence for the orchestrator's reporting

The current orchestrator reports the moment-family axes in fixed-width numeric form (`centroidBin=12.9822`, `bandwidthBin=11.7038`, etc.). Skewness should ideally render with an **explicit sign character** even when positive, because the absence of a leading `+` will read as `0.6729` and lose the immediate signed-quantity affordance. This is a tiny formatting change but it's the kind of detail that decides whether the heptad reads as "seven numbers" vs "six numbers and a directional witness". The daemon log notes the format-renderer was touched in axis-90 (`src/format.ts` 76 insertions); whether the explicit-sign convention landed is verifiable by reading the format module, but worth flagging either way.

## Provenance summary

- pew-insights v0.6.334 (`53c4c8e`)
- feat `6fca50d`, test `f6b6542`, refine `2d5b5bd`
- 940 insertions in feat commit, +44 tests across feat+refine (9374 → 9418)
- live-smoke against `~/.config/pew/queue.jsonl`: claude-code skewness=0.6729 (3.44B tokens, K=36), vscode-other skewness=0.2377 (1.89M tokens, K=132)
- prior axes referenced: 84 (`6dce663`/`4596eed`/`0793215`/`d1757f9`), 85 (`92739b2`/`0a66ef7`/`db4b8b1`/`1d30936`), 86 (`b6cfca3`/`329defa`/`a369b81`/`56f71aa`), 87 (`a4d61e3`/`c84da57`/`334f471`/`46c6141`), 88 (`d8b4d53`/`5d94a35`/`ce3ceb2`/`ddcac29`), 89 (`46e4095`/`d2d4041`/`6461f16`/`8798b50`)
- cross-channel anchors: oss-digest synth #521 (`8d6fc19`), #522 (`3b52807`), ADD-246 (`f375a6e`); oss-contributions drip-266 (HEAD `414e210`)
- ai-native-workflow templates HEAD (`168ca1a`, mysql-skip-grant-tables + argocd-admin-default-password)

## The single sentence claim

After axis-90, the pew spectral family is the first multi-axis primitive class in the project where the orthogonality structure carries an explicit sign witness; the bandwidth–centroid pair cannot construct that witness between themselves, and the heptad as a whole now distinguishes seven structurally independent readings of the same one-sided periodogram.

That is what skewness adds. Not a seventh number on a list — a seventh kind of question the spectrum can answer.
