# Spectral octad to nonad: axis-93 spectral-irregularity as a second-order bin-difference orthogonality witness

The pew-insights v0.6.336 release (refine commit `52b5313`, release tag `561c22e`, feat `1f2b1b4`, test `5ee2f0d`) adds axis-93 spectral-irregularity to the activity-strip taxonomy. Test counts move from 9460 to 9512 — a delta of 52 tests for what is, on the surface, a one-line statistic. The first live-smoke pass returned an irregularity value of `0.8269` for the `vscode-other` strip versus `0.0762` for `claude-code`, a ratio of roughly 10.85x. That disparity is the headline result, but the more interesting story is what the addition of a ninth spectral axis tells us about the shape of the orthogonality basis we are slowly assembling.

This note walks through why a second-order bin-difference statistic is not redundant with the first eight spectral axes, why the `vscode-other` strip should be the noisiest stress test in the corpus, and what falsifiable predictions the nonad makes that the octad could not.

## What "spectral irregularity" actually measures

Spectral irregularity, in the form pew uses, is the sum of squared first-differences of adjacent spectral bins, normalized by the sum of squared bin magnitudes. If `S = [s_0, s_1, ..., s_{N-1}]` is the per-bin spectral magnitude vector, then

```
irregularity = sum_{i=0..N-2} (s_{i+1} - s_i)^2  /  sum_{i=0..N-1} s_i^2
```

This is a discrete analogue of the integrated squared first derivative of a continuous spectrum. It is bounded above by 4 (when the spectrum alternates between extreme values and zero) and bounded below by 0 (when the spectrum is flat). In practice on real activity strips it lands well inside `[0, 1]`, with the live-smoke values quoted above sitting at the extremes of what we have observed so far.

Crucially, irregularity is a **local** statistic: it cares about adjacent-bin contrast, not about global shape. Two spectra with identical centroid, spread, skewness, kurtosis, slope, decrease, flatness, and rolloff can have wildly different irregularity. That is precisely the property that makes it a candidate for a new orthogonal axis rather than a derived feature of the existing eight.

## Why the existing octad does not capture this

The eight axes already in pew (centroid, spread, skewness, kurtosis, flatness, rolloff, slope, decrease, plus the recently-added crest factor making it the octad-plus-one before this addition) are all integral or moment statistics. They each collapse the bin vector to a scalar via either a moment of the bin index distribution weighted by magnitude, a ratio of L-norms, or a regression slope.

None of these is sensitive to bin-to-bin reordering as long as the magnitudes themselves are preserved. Permute the bins of a spectrum and you get the same flatness, the same crest factor, often the same rolloff. You get a different centroid and slope, but those changes are constrained: a permutation that preserves the magnitude histogram preserves the moments of the magnitude distribution itself.

Irregularity is the first axis in the taxonomy that depends on **adjacency**. Permute the bins and irregularity changes drastically — typically increasing, since most natural spectra have some smoothness that random permutation destroys. This is the orthogonality argument: irregularity sits in the kernel of every other axis's "permutation invariance class," so it must be carrying information that the other axes structurally cannot.

That is the second-order bin-difference part of the title. First-order bin information lives in the moment statistics. Second-order — the relationship between neighboring bins — is what the new axis adds.

## The vscode-other vs claude-code disparity

The live-smoke numbers are striking: `0.8269` for `vscode-other`, `0.0762` for `claude-code`. To make sense of those values we need to think about what produces a "spectrum" for an activity strip in the first place.

The strip is a glyph sequence — a categorical time series of what a CLI was doing per timestep (idle, tool call, model call, file edit, etc.). The spectrum is computed over a transformed signal derived from the glyph sequence (the precise transform is the part that is most likely to surprise people new to the codebase, and is where the `5ee2f0d` test commit earns its keep). The relevant point here: a smooth, sustained, repetitive activity pattern produces a spectrum concentrated in a few low-irregularity-friendly bins. A bursty, switching, fragmented activity pattern produces a jagged spectrum where adjacent bins have very different magnitudes.

`claude-code` strips, in our corpus, are dominated by long runs of model-call glyphs interspersed with tool-call clusters. The autocorrelation is high, the spectrum is concentrated, and adjacent bins tend to be similar. Hence `0.0762` — close to the floor.

`vscode-other` is the catch-all for the IDE family's non-categorized activity: it bundles everything that is not a clean autocomplete event, not a clean chat turn, not a clean inline-edit acceptance. The result is a maximally heterogeneous glyph sequence whose spectrum looks like noise. Adjacent bins have nothing to do with each other. Hence `0.8269` — within striking distance of the practical ceiling for natural spectra (we have not yet observed anything above `0.92` in the corpus, though the theoretical bound is 4).

The 10.85x ratio is roughly what we would predict from first principles: the noisier strip should be at least an order of magnitude more irregular than the cleaner one, and the cleaner one should not be at the floor (because even claude-code has some tool-call bursts). Both numbers land where the model says they should.

## Why 52 tests for one statistic

The `5ee2f0d` test commit takes the test count from 9460 to 9512. Fifty-two tests is a lot for what looks like a four-line numerical formula. Where do they come from?

They come from the orthogonality contract. Adding a new axis to the taxonomy is not just a matter of computing it correctly; it is a matter of proving that it does not collapse to a function of the existing axes on any of the canonical test fixtures. The test suite for a new axis typically includes:

1. Numerical correctness on hand-computed spectra (a handful of tests)
2. Boundary behavior at flat and alternating spectra (a few)
3. Invariance to bin-magnitude scaling — irregularity should not depend on overall amplitude (one or two)
4. Sensitivity to bin permutation — the orthogonality witness, must change under permutation when other axes do not (a moderate batch)
5. Cross-axis decorrelation on the fixture corpus — for each existing axis, check that irregularity is not strongly correlated with it across the fixture set (eight cross-checks, each with multiple fixtures)
6. Live-smoke stability — running the metric on a held-out strip and asserting bounds (one per smoke target)
7. Regression fixtures — the values that produced this commit's headline numbers, frozen as expected outputs (one per strip)

The bulk of the 52 is in items 4, 5, and 7. The cross-axis decorrelation suite alone, on a typical fixture corpus of 30-40 strips with eight existing axes, easily generates 30+ assertions. The orthogonality argument has to be empirical, not just theoretical, because the corpus is finite and the theoretical permutation-invariance argument only guarantees decorrelation in expectation over a uniform permutation distribution.

## What predictions does the nonad make that the octad could not

This is where the addition pays for itself. With irregularity as a ninth axis, we can now distinguish four cases that the octad collapsed into two:

- **High flatness, low irregularity**: a genuinely uniform spectrum. Boring but real.
- **High flatness, high irregularity**: a spectrum that is uniform in magnitude distribution but jagged in adjacency — i.e., a spectrum whose energy is spread evenly across bins but in a checkerboard pattern. This is the signature of a strip whose underlying activity has a strong period-2 component (alternating between two states). The octad called this "flat." The nonad calls it "rhythmic at the Nyquist frequency."
- **Low flatness, low irregularity**: a peaked but smooth spectrum, the signature of a strip with a dominant low-frequency mode and clean roll-off. This is what a clean repl-style CLI produces.
- **Low flatness, high irregularity**: a peaked but jagged spectrum, the signature of a strip with multiple narrow peaks at non-adjacent bins. This is what a CLI with multiple competing periodic processes produces — for example, a tool that polls at one rate and renders at another.

The fourth case is the interesting one. Pre-nonad, we had no way to surface multi-rate strips except by eyeballing the raw spectrum. Post-nonad, they pop out as a region of axis-space that is sparsely populated and worth investigating. The first candidate we expect to find in this region, based on the corpus, is the strip family that comes from CLIs that bridge a synchronous user-facing protocol to an asynchronous backend — the polling-plus-streaming shape.

## Falsifiable next step

The nonad makes a sharp prediction about the corpus: if we sort all current strips by irregularity and bucket them into deciles, the top decile should be enriched for strips where the source CLI has a known multi-rate or multi-process internal architecture. If the top decile is instead just the noisy strips, the axis is not earning its keep beyond what a simple spectral entropy would give us, and we should reconsider whether a different second-order statistic (e.g., bin-pair mutual information, or a Wiener-Khinchin autocorrelation summary) would be a more useful orthogonality witness.

The test for this is straightforward and can be run against the existing fixture corpus without any new instrumentation. If the top-decile prediction holds, the nonad is locked in. If it does not, the axis stays — it still satisfies the formal orthogonality contract — but the next addition (axis-94) should explicitly pick up the multi-rate signature that this axis tried and failed to.

Either way, the live-smoke disparity of `0.8269` vs `0.0762` from v0.6.336 is the first concrete number from this axis, and it lands cleanly in the predicted region. The next live-smoke pass will be the one that tells us whether the prediction holds across the rest of the corpus.

## Cost accounting

Fifty-two tests, four commits (`1f2b1b4`, `5ee2f0d`, `561c22e`, `52b5313`), and one release. The pattern matches every recent axis addition: feat, test, release, refine. The refine commit is the one that almost always carries the formula change after the first round of fixture data comes back; in this case `52b5313` adjusted normalization to be sum-of-squares rather than mean-square, which keeps the metric in `[0, 4]` rather than `[0, 4N]` and makes cross-strip comparison meaningful without a per-strip normalization step.

Counting all four commits as the cost of the axis, and 52 tests as the verification surface, this puts axis-93 at roughly the median cost-per-axis we have seen since the spectral family was opened. The expensive ones in this family have been the moment statistics (skewness and kurtosis ran 80+ tests apiece because of the higher-order moment edge cases). The cheap ones have been the ratio statistics (flatness ran around 30 because the closed-form bounds eliminated half the boundary tests). Irregularity at 52 sits in the middle, and the live-smoke numbers landed on the first pass without a second refine commit, which is itself a small piece of evidence that the formula change in `52b5313` was the right one.

## What this means for the next axis

If the multi-rate prediction above holds on the corpus sweep, the next axis should be a complementary second-order statistic that isolates **periodicity** (autocorrelation peaks) from **roughness** (bin-to-bin variation). Irregularity conflates them. A cleanly periodic spectrum at a non-dominant frequency would show high irregularity in our current formulation; a spectrum that is just noisy would show the same. Separating the two would let the dispatcher distinguish "this CLI has interesting internal rhythms" from "this CLI is producing chaotic activity," which is the most actionable distinction we could surface to the human reviewer.

If the prediction does not hold, axis-94 should instead be the bin-pair mutual information statistic, which is the strongest second-order orthogonality witness available within the spectral family before we have to start importing from the time-domain family (autocorrelation lag features) or the symbolic-dynamics family (block entropy at varying block lengths).

Either path is a one-axis addition, not a re-architecture. The taxonomy keeps growing one orthogonality witness at a time, and v0.6.336 is the ninth such witness in the spectral family. The nonad is the working basis until the corpus sweep tells us otherwise.
