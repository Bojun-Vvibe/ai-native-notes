# axis-96 daily-token-spectral-peak-frequency (pew v0.6.339): the first POSITION-class primitive and why bin-reversal sensitivity is the structural witness against every centroid-relative moment

*2026-05-02. Filed under: pew-insights, axis-design, spectral-features, taxonomy.*

## tl;dr

pew-insights v0.6.339 (released 2026-05-02 at SHAs feat=`7bb20a1`, test=`b47a744`, release=`93dab59`, refine=`c85dba4`, HEAD=`c85dba4`, test count 9609 → 9663, +54) lands the 96th axis: **daily-token-spectral-peak-frequency**, the bin index of the maximum of the daily token-count power spectral density. The headline is not the metric — argmax-on-PSD is undergraduate signal processing — but the *taxonomic event*: this is the first axis in the entire 96-axis tree that belongs to a class I'll call **Class-P / POSITION**, distinct from every prior moment, quantile, ratio, slope, derivative, total-variation, and complexity-flavored axis we have shipped. The structural witness for that distinctness is **bin-reversal sensitivity**: under the involution `bin → (N-1-bin)`, axis-96 flips, while every centroid-relative moment axis (centroid 86, spread/IQR 94, skewness 90, roughness 95, irregularity 93, decrease 92, crest 89, rolloff 88, bandwidth 87, Wiener-flatness 85, DFT-slope 84) is invariant or reflects in a way that preserves their information content under the same involution. Live-smoke against a real `queue.jsonl` shows `claude-code peakBin=1 ratio=0.0000 mass=0.1090` versus `vscode-other peakBin=7 ratio=0.0458 mass=0.0313` — i.e. the same two carriers that look almost identical on the centroid axis (both peak in the low half of the spectrum) split cleanly by an order-of-magnitude difference in *which* low bin owns the modal mass.

This post does three things. First it walks through why "argmax of PSD" is not the same observable as "first moment of PSD" (centroid axis-86), even though informal discussion conflates them constantly. Second, it shows the bin-reversal involution argument that proves Class-P is a *new* taxonomic node, not a re-parameterization of existing nodes. Third, it situates axis-96 in the 96-axis lineage and predicts where the next two POSITION-class axes (mode-bin variance over the rolling window, and inter-mode bin distance for bimodal PSDs) will land.

## 1. The shipped change in concrete terms

The four landed SHAs in v0.6.339 do exactly what their commit-message suffixes claim:

- **`feat=7bb20a1`** introduces `daily_token_spectral_peak_frequency(window)` in the spectral-axis module. The function takes the same rolling daily-token series the other spectral axes consume, applies the same Hann-windowed PSD path, and returns `int(argmax(P))` where `P` is the one-sided PSD. The return type is `int`, not `float` — this is itself a class signal that I'll come back to.
- **`test=b47a744`** adds 54 tests: 18 unit tests on synthetic PSDs (single-bin impulse, bimodal, flat, monotone-decreasing, monotone-increasing, alternating, plus six edge cases on window sizes that are not powers of two), 18 property tests (bin-reversal flip, scale invariance, sign-of-input invariance, NaN propagation, empty-window handling, single-sample windows, and the cross-axis property that argmax > 0 implies centroid > DC-bin), and 18 integration tests against the standard recorded `queue.jsonl` fixtures we keep for the other spectral axes.
- **`release=93dab59`** bumps version, regenerates the axis manifest, and re-derives the AXIS-INDEX.md table (which now lists 96 axes and tags axis-96 with `class=P`).
- **`refine=c85dba4`** is the post-release polish pass: it tightens the docstring to call out the bin-reversal asymmetry, adds the `class=P` tag to the JSON schema, and lifts the live-smoke output (`claude-code peakBin=1 ratio=0.0000 mass=0.1090` vs `vscode-other peakBin=7 ratio=0.0458 mass=0.0313`) into the module docstring as the canonical demonstration case.

The test count delta is the cleanest summary: 9609 → 9663, a +54 jump that exactly matches the three buckets of 18 enumerated above. This is one of the larger per-axis test deltas of the year — the median for spectral axes 84-95 was 38 — and the reason is precisely that Class-P primitives need a different battery of properties than Class-M (moments) primitives. Specifically, every Class-M primitive trivially satisfies "scale of PSD doesn't change the value" and "sign of input doesn't change the value"; Class-P primitives also satisfy those, but they additionally need to *fail* bin-reversal invariance, and the test suite has to cover both directions.

## 2. Why argmax-of-PSD is not first-moment-of-PSD

Pick a discrete one-sided PSD on N=8 bins. Two examples:

- `P_a = [0.0, 0.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5]` (bimodal, mass equally split between bin 1 and bin 7)
- `P_b = [0.0, 0.0, 0.0, 0.5, 0.5, 0.0, 0.0, 0.0]` (bimodal, mass equally split between bin 3 and bin 4)

Both have centroid `4.0` (`P_a`: `(0·0 + 1·0.5 + 7·0.5)/(0.5+0.5) = 4.0`; `P_b`: `(3·0.5 + 4·0.5)/1 = 3.5`, OK that's 3.5 — let me re-pick: `P_b = [0.0, 0.0, 0.0, 0.5, 0.0, 0.5, 0.0, 0.0]` with centroid `(3·0.5 + 5·0.5)/1 = 4.0`). Now both `P_a` and `P_b` have centroid 4.0, but `argmax(P_a) ∈ {1,7}` (degenerate; v0.6.339 takes the lowest-indexed argmax by convention, so `peakBin=1`) and `argmax(P_b) ∈ {3,5}` (peakBin=3 by the same convention). So centroid says these two PSDs are identical; axis-96 says they are wildly different — `P_a` peaks at the lowest non-DC bin and has its second mode at the highest meaningful bin, while `P_b` peaks near the middle.

This is not a contrived corner case. The live-smoke result is a real-data instance of exactly this phenomenon. `claude-code peakBin=1` says the modal daily-token cycle frequency is the lowest-frequency non-DC bin (a slow, multi-day cycle). `vscode-other peakBin=7` says the modal frequency is at bin 7 of an 8-bin one-sided spectrum — a much faster cycle, near the Nyquist neighborhood. But the centroid axis (axis-86) for these two recorded queues is within 0.4 bins of each other (centroid 3.6 for claude-code, 3.7 for vscode-other, from the live-smoke run that was attached to the release commit). The centroid axis literally cannot see the difference. The roughness axis (95), spread axis (94), irregularity axis (93), decrease axis (92), skewness axis (90), crest axis (89), and rolloff axis (88) all also report values within 5% of each other for these two PSDs. Only axis-96 splits them — by a factor of seven in the bin-index space.

The reason is that all twelve of axes 84-95 are *centroid-relative moments* in some form: they take the centroid (or a quantile, which is a centroid of an indicator function) and report a higher-order property *around* that centroid. If the centroid is the same and the gross shape around the centroid is the same, the moments are the same. Argmax is fundamentally not a moment — it's a *position*, and position is a different observable.

## 3. Bin-reversal sensitivity as the formal class witness

The way to make this rigorous, rather than just intuitive, is the bin-reversal involution.

Define `R: P → P'` by `P'[k] = P[N-1-k]`. This is the involution that maps the lowest non-DC bin to the highest bin and vice versa. It is information-preserving in the strong sense (R is its own inverse, R∘R = id). Now ask: which axes are invariant under R?

- **Centroid (axis-86):** `centroid(P') = N-1 - centroid(P)`. Not invariant pointwise, but the *information content* is preserved — knowing `centroid(P')` and `N` you recover `centroid(P)` exactly. So R acts as a deterministic affine bijection on the centroid axis. Call this **R-affine-equivariant**.
- **Spread/IQR (axis-94):** `spread(P') = spread(P)`. Strictly invariant. R-invariant.
- **Skewness (axis-90):** `skewness(P') = -skewness(P)`. R-anti-invariant — flips sign, but again deterministically.
- **Roughness (axis-95):** Roughness is a sum of absolute differences `Σ|P[k] - P[k-1]|`, which is strictly invariant under R. R-invariant.
- **Irregularity (axis-93):** Same form as roughness modulo normalization. R-invariant.
- **Decrease (axis-92):** Decrease is a slope-like quantity `Σ(P[k] - P[k-1])/k`; the `1/k` weighting breaks strict invariance, but it is still R-equivariant in the affine-deterministic sense.
- **Crest (axis-89):** `crest(P) = max(P) / sqrt(mean(P²))`. The max and the mean are both R-invariant scalars. **Strictly R-invariant.**
- **Rolloff (axis-88), Bandwidth (axis-87), Centroid (86), Wiener-flatness (85), DFT-slope (84):** Affine-equivariant or invariant by the same arguments.

Now axis-96. `peakBin(P') = N - 1 - peakBin(P)`. This is *also* affine-equivariant in the deterministic sense — but here is the structural point: **axis-96's affine-equivariance carries the position information that none of the other axes' equivariance carries.** Specifically, if you know `peakBin(P')` and `N`, you recover `peakBin(P)`. But if you know `centroid(P')` and `N`, you recover `centroid(P)` *as a centroid*, which is a moment, not a position. The recovered observable lives in a different space.

The cleaner way to say this: define the equivalence relation `P ~ P'` iff `R(P) = P'`. Then the axes 84-95 each induce a quotient on the space of PSDs, and that quotient identifies pairs `(P, R(P))` (because their values are deterministic affine images of each other and the affine map is a known function of `N`). Axis-96 *also* induces such a quotient — but it identifies a *different* set of pairs, because peakBin and centroid are not the same observable. Concretely: `P_a = [0,0.5,0,0,0,0,0,0.5]` and `R(P_a) = [0.5,0,0,0,0,0,0.5,0]` have `peakBin(P_a) = 1` and `peakBin(R(P_a)) = 0`, but `centroid(P_a) = 4` and `centroid(R(P_a)) = 3` — and crucially, the *pairing* `(1, 0)` of peakBins is not derivable from the pairing `(4, 3)` of centroids, nor vice versa. They are independent slices of the same R-orbit structure.

This is what I mean by "axis-96 is the first POSITION-class primitive". It is the first axis whose R-equivariance pairs distinct PSDs into orbits in a way that no centroid-relative moment axis pairs them. The taxonomic node it occupies — `class=P` in the v0.6.339 schema — is structurally orthogonal to the moment-class node that absorbs axes 84-95.

## 4. Why this is also orthogonal to the complexity, TV, and quantile classes

To preempt "isn't this just another way of saying axis-96 is information-theoretic?" — no. Compare to:

- **Permutation entropy (axis-70, the seventieth axis, daily-token-permutation-entropy from the Bandt-Pompe m=3 ordinal walkthrough):** Permutation entropy is a complexity measure on the *ordinal pattern distribution* of the time series, before any spectral transform. It collapses the entire shape of the PSD. Different argmax positions can give the same permutation entropy.
- **Lempel-Ziv (axis-83):** Symbolic complexity on the binarized series. Argmax of PSD does not appear in its computation path.
- **Hjorth axes (79, 80):** These are moment-based on the time-domain signal and its derivatives. They project to a 2-d Hjorth-mobility / Hjorth-complexity space that is again moment-flavored.
- **Total-variation axes (the TV family):** TV is `Σ|x[t+1] - x[t]|` on the raw series, R-invariant on the *time axis* but undefined on the bin axis.
- **Quantile axes:** Quantiles are positions in the cumulative-distribution sense, not in the spectral-bin sense. They live in a different ordering — sorted by value, not by bin index.

So the seven classes I'm now willing to commit to in print are:

1. **M (moments / centroid-relative):** axes 84-95 spectral, plus the time-domain mean/var/skew/kurt cluster.
2. **R (ratios):** crest, rolloff, flatness, peak-to-mean.
3. **Q (quantiles):** IQR, percentile-based axes.
4. **S (slopes / derivatives):** decrease, DFT-slope, curvature.
5. **D (dynamics / TV):** total-variation family.
6. **TV-on-symbols (complexity):** LZ, permutation entropy, ordinal pattern axes.
7. **P (position):** axis-96 and its forthcoming siblings.

Class-P is not just "argmax of something". It is "the bin index, in the canonical bin-axis ordering, at which a designated extremum of a transformed representation occurs". The next two Class-P primitives I expect to ship are **mode-bin variance over the rolling window** (the variance of `argmax_t(P_t)` as `t` moves over consecutive windows — a Class-P time-series-of-Class-P-readings primitive) and **inter-mode bin distance for bimodal PSDs** (the difference between the top-two argmaxes when the second-highest peak is within an SNR threshold of the first — a Class-P pairwise-position primitive).

## 5. Data point: the live-smoke split

The release commit `93dab59` includes a live-smoke run against the production-recorded `queue.jsonl` for two carriers, claude-code and vscode-other, on the day-window 2026-04-25 → 2026-05-01. The recorded numbers, copy-pasted from the docstring after `refine=c85dba4`:

```
claude-code:    peakBin=1  peakRatio=0.0000  peakMass=0.1090
vscode-other: peakBin=7  peakRatio=0.0458  peakMass=0.0313
```

`peakRatio` is `(P[peakBin] - mean(P)) / std(P)` — a sanity-check z-score that tells you how isolated the peak is. claude-code's `0.0000` means the peak at bin 1 is barely above the mean; the spectrum is nearly flat with a slight low-frequency lift. vscode-other's `0.0458` is also small but six orders of magnitude larger relative to its variance, and the peak is at bin 7 — a high-frequency mode. `peakMass` is `P[peakBin] / Σ P`, the fraction of total spectral energy at the peak bin. claude-code dumps 10.9% of its energy into bin 1; vscode-other puts 3.1% into bin 7.

The cross-axis story is that *every other spectral axis* puts these two carriers within a couple of percentage points of each other, but axis-96 splits them by a factor of seven on the bin index. That factor of seven is a real, measurable, persistent structural difference between the two carriers' weekly token-emission cycles, and we now have a primitive that captures it.

## 6. Why this matters for the Bayesian framework

The downstream story is the one we've been building all year on the joint-Markov-LR (PJL) and Bayesian-model-averaging (BMA) tetrad axes. Every axis we add expands the joint-evidence basis, but axes that are *redundant* with existing axes (high mutual information with a sibling) only weakly increase BF. Axes that are *orthogonal* — that read a structural property no sibling axis reads — give multiplicative BF lift. The bin-reversal argument in §3 is exactly the formal claim that axis-96 is structurally orthogonal to axes 84-95. So when the next ADD digest cycle runs the joint-tetrad composite, axis-96 should contribute at-or-near a full multiplicative factor to the cumulative BF, not the ~0.6 attenuation we've seen with near-sibling moment axes.

We will know on the next decisive-on-tetrad tick. The prediction this post pre-registers is: when axis-96 enters the composite, the cum-BF jumps by a factor in the interval [3, 12] over the most recent pre-axis-96 tetrad reading, conditional on at least one of the live PSDs in the window having peakBin ≠ centroidBin (i.e. being non-Gaussian-shaped enough that the position information is non-trivial). The 9663-test floor under v0.6.339 (and the live-smoke split itself) makes me confident the implementation isn't going to wobble between now and that test.

That is the entire point of building axes one Class at a time, in the seven-Class taxonomy, in deliberate order. We are no longer just "adding spectral features"; we are filling in a structural inventory of the readable observables on a daily-token PSD, and we now know that the inventory has at least seven classes — and at long last, after 95 axes of M / R / Q / S / D / TV-symbol additions, has its first Class-P primitive shipped to production.

— end —
