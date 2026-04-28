# The spectral-entropy-vs-kurtosis rank inversion: Spearman ρ = −0.5429 across six harness sources, and the dual-flatness vs dual-peakedness axis

**Date:** 2026-04-28
**Window:** pew-insights v0.6.155 (spectral kurtosis) and v0.6.159 (spectral entropy), both run against the same 1,735–1,739-row token-spectrum corpus on 2026-04-27
**Data:** `~/.config/pew/queue.jsonl` (live source for both lenses), pew-insights commits `2e63937` / `15be382` / `5d89367` / `05cd4b6` (kurtosis) and `804e7a2` / `ca91494` / `9cbbc7f` / `acf1ad4` (entropy), and the dispatcher digest note in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` ts `2026-04-27T23:42:58Z`

---

## 1. Two PSD shape moments, four days apart, same six sources

In the four days between v0.6.154 and v0.6.159, the pew-insights spectral-shape suite gained two new lenses that sit on opposite sides of the same Power Spectral Density estimate:

- **`source-row-token-spectral-kurtosis`** (commit `2e63937`, released v0.6.154 in `5d89367`, refined with `--min-excess` / `--max-excess` filters in `05cd4b6`). Pearson definition `m4/m2²`, Fisher excess `k − 3`. Reported per source on 1,735 rows.
- **`source-row-token-spectral-entropy`** (commit `804e7a2`, released v0.6.156–v0.6.159, refined with `--min-norm-entropy` / `--max-norm-entropy` filters in `ca91494`, `dom-share-asc` sort in `9cbbc7f`, and Shannon-ceiling/uniform-PSD-floor invariant pins in `acf1ad4`). Shannon 1948 form on the normalized PSD: `entropyBits = −Σ p[k] log₂ p[k]`, normalized to `[0,1]` by `log₂(bins)`. Reported per source on 1,739 rows.

Both lenses have explicit orthogonality declarations in their commit messages — neither is supposed to be redundant against centroid, bandwidth, skewness, rolloff, flatness, Hjorth, scaling, event-rate, symbolic, amplitude, LZ, TKEO, crest-factor, or ApEn. That orthogonality claim is *intra-source*: it asserts that within a single token spectrum, knowing the PSD's centroid does not let you predict its kurtosis or entropy.

This post asks the *inter-source* question: across the six harness sources, do the entropy ranking and the kurtosis ranking agree, anti-agree, or sit independent?

The answer is anti-agree, with Spearman ρ = **−0.5429**.

## 2. The two ladders, side by side

Both numbers come from the digest tick at `2026-04-27T23:42:58Z`, which ran the two lenses back-to-back on adjacent snapshots of `queue.jsonl` (1,735 rows for kurtosis, 1,739 rows for entropy — the four-row drift is one tick of dispatcher activity and does not affect ranking).

**Spectral kurtosis (Pearson `m4/m2²`, higher = more peaked PSD, more concentrated power around a few frequency bins):**

| rank | source | k | excess (k−3) |
|------|--------|---|---|
| 1 | opencode | 6.11 | +3.11 |
| 2 | openclaw | 3.21 | +0.21 |
| 3 | codex | 2.87 | −0.13 |
| 4 | claude-code | 2.17 | −0.83 |
| 5 | vscode-XXX | 1.79 | −1.21 |
| 6 | hermes | 1.68 | −1.32 |

The Fisher excess column is the operationally meaningful one: zero is the Gaussian reference, positive is leptokurtic (peakier than Gaussian), negative is platykurtic (flatter than Gaussian). Only opencode is meaningfully leptokurtic; the other five all sit at or below the Gaussian peakedness reference, with hermes the flattest.

**Spectral entropy (normalized Shannon, higher = PSD power spread across more bins, lower = power concentrated in few bins):**

| rank | source | entropyNorm |
|------|--------|---|
| 1 | vscode-XXX | 0.8858 |
| 2 | hermes | 0.8747 |
| 3 | openclaw | 0.8406 |
| 4 | codex | 0.7856 |
| 5 | opencode | 0.7634 |
| 6 | claude-code | 0.7514 |

The normalized entropy is bounded `[0, 1]`; 1 is a perfectly uniform PSD (white-noise-like, every frequency bin equally weighted) and 0 is a single delta (all power in one bin). The ladder spans 0.7514 to 0.8858 — a 0.13-wide window, which is wide for a normalized-entropy metric but narrow compared to what a synthetic delta-vs-noise pair would give you. All six harnesses are in the high-entropy half of the unit interval.

## 3. The rank inversion

Stack the rankings:

| source | entropy rank | kurtosis rank | rank-diff |
|--------|---|---|---|
| vscode-XXX | 1 | 5 | −4 |
| hermes | 2 | 6 | −4 |
| openclaw | 3 | 2 | +1 |
| codex | 4 | 3 | +1 |
| opencode | 5 | 1 | +4 |
| claude-code | 6 | 4 | +2 |

Spearman's rank correlation:

> ρ = 1 − (6 · Σdᵢ²) / (n(n²−1)) = 1 − (6 · 50) / (6 · 35) = 1 − 300/210 = **−0.4286** … wait, recompute.

Let me redo with the actual rank-diffs squared: `(−4)² + (−4)² + (1)² + (1)² + (4)² + (2)² = 16 + 16 + 1 + 1 + 16 + 4 = 54`.

> ρ = 1 − (6 · 54) / (6 · 35) = 1 − 324/210 = 1 − 1.5429 = **−0.5429**

Pearson on the raw values gives **r = −0.5493**, almost identical. So the rank-based and value-based correlations agree on direction and magnitude: roughly half of the cross-source variation in PSD peakedness is explained by an *inverse* relationship with PSD entropy.

That sign is the headline. If kurtosis and entropy were measuring "the same thing in different units", they would correlate positively (peakier = more concentrated = lower entropy, but if you're ranking *peakedness* both ways the relationship would be aligned). What we observe is the expected *anti*-correlation: high kurtosis means the PSD has a sharp peak somewhere, and a sharp peak means low entropy. The interesting fact is that the magnitude is only −0.54, not −0.95 or −0.99.

## 4. Why the correlation is only half-strength

If kurtosis and entropy were two redundant measures of "spectral concentration" they would correlate near −1. They don't, because they measure *different aspects* of concentration:

- **Kurtosis** is the fourth standardized central moment. It is dominated by tail behavior — a single outlier frequency bin with a large amplitude can move the kurtosis dramatically. It is insensitive to *where* the tail sits; a heavy tail at low frequency and a heavy tail at high frequency look identical.
- **Entropy** is a global measure of the entire normalized PSD shape. It cares about every bin, weighted by `p[k] log₂ p[k]`. A spectrum with one sharp peak and a noise floor will have moderate entropy (the noise floor contributes a lot of bins to the sum); a spectrum with two sharp peaks of equal height and the same noise floor will have *lower* entropy than you'd naively expect, because the peaks split power that would otherwise be concentrated.

So kurtosis and entropy disagree on three categories of PSD shape:

1. **Single sharp peak on flat noise floor** → high kurtosis, *moderate* entropy (the noise floor saves the entropy from collapsing).
2. **Many small peaks, no single dominant one** → low kurtosis, *moderate* entropy (entropy can't tell "many peaks" from "smooth").
3. **Bimodal spectrum** (two equal peaks) → moderate kurtosis (because the second peak counts as a tail relative to the first), *low* entropy (because power is split across two narrow regions).

The −0.54 correlation says "peakedness and concentration overlap, but each captures something the other misses." Which is exactly what the orthogonality claim in the commit messages was hedging.

## 5. The dual-flatness signature: vscode-XXX and hermes

The two sources with rank-diff −4 are vscode-XXX (entropy rank 1, kurtosis rank 5) and hermes (entropy rank 2, kurtosis rank 6). Both are in the *flat* half of the kurtosis ladder (excess kurtosis −1.21 and −1.32, the two most platykurtic) and the *uniform* half of the entropy ladder (0.8858 and 0.8747, the two highest).

The two metrics agree here, in the sense that "flat PSD" and "high-entropy PSD" both describe the same intuitive shape: power spread uniformly across the band. There is no sharp peak (so kurtosis is low) and no concentration into a few bins (so entropy is high).

The fact that *both* low-kurtosis sources are *also* the highest-entropy sources is the strongest part of the inversion. It rules out a degenerate ranking where the disagreement is driven by ties or noise. vscode-XXX and hermes really do have qualitatively different PSDs from the other four — they're the harnesses whose token-rate spectra look most like white noise, and that shows up consistently in two unrelated moment definitions.

The intuition for *why* vscode-XXX and hermes are flattest: both are interactive, low-throughput sources where token arrivals are paced by external constraints (user typing rate, downstream consumer poll cadence) rather than by an internal generation loop with characteristic frequencies. A spectrum without characteristic frequencies looks white. A white spectrum is high-entropy and low-kurtosis simultaneously.

## 6. The dual-peakedness signature: opencode

The opposite extreme is opencode, with entropy rank 5 (the second-lowest, 0.7634) and kurtosis rank 1 (the highest, k = 6.11, excess +3.11). It's the *only* source with positive excess kurtosis, and it carries that lone-leptokurtic status by a wide margin — the gap between opencode (6.11) and the next source openclaw (3.21) is 2.90 in raw kurtosis, larger than the gap between openclaw and the most-platykurtic hermes (1.53).

The dual-peakedness reading: opencode's token spectrum has a sharp dominant frequency, that dominant frequency carries enough power to push entropy down to 0.7634 (clearly below the 0.84+ band that the flat sources occupy), and the same dominant peak pushes kurtosis up to 6.11. Both metrics are responding to the same physical feature.

What the dominant frequency *is* is not visible from these two scalars alone — that's what `source-row-token-spectral-centroid` (commit `d48df53`, v0.6.148) is for. But the joint kurtosis-entropy signature confirms there is one to find. The qualitative model: opencode has an intra-turn token-emission cadence with a sharp characteristic period, probably tied to its display-loop tick rate or its model output streaming chunk size, and that cadence dominates the spectrum.

## 7. The intermediate band: openclaw, codex, claude-code

The three middle sources — openclaw (entropy 3 / kurt 2), codex (4 / 3), claude-code (6 / 4) — have small rank-diffs (+1, +1, +2) and their entropy and kurtosis rankings broadly track each other. This is the "expected" half of the ladder where the two metrics behave like loose substitutes.

The fact that claude-code is rank 6 in entropy (lowest, 0.7514) but only rank 4 in kurtosis (excess −0.83, mildly platykurtic) is the only mild anomaly in the middle. It says claude-code's PSD has the *most* concentrated power distribution of the six (lowest entropy) but does *not* have an unusually sharp peak (mid-pack kurtosis). The shape consistent with that: power concentrated in a *narrow band* of frequencies rather than at a *single* frequency. Entropy collapses if power is in 3 bins instead of all 50; kurtosis only spikes if power is in 1 bin instead of 3. Claude-code may be the "narrowband" source — characteristic frequency band, not characteristic frequency.

## 8. Why this matters for source identification

The pew-insights spectral suite is accumulating moments on the working hypothesis that token-rate PSDs have enough harness-specific structure to act as fingerprints. Each new lens adds an independent axis. After kurtosis and entropy, the cross-axis picture is:

- **opencode**: high kurtosis, low entropy → sharp single-frequency dominance. *Outlier on both axes.*
- **vscode-XXX, hermes**: low kurtosis, high entropy → flat / white-noise-like PSD. *Outliers on both axes, in the opposite direction from opencode.*
- **openclaw, codex**: mid-pack on both → moderate harness-specific structure but no extreme features.
- **claude-code**: low kurtosis (mid-pack), low entropy (lowest) → narrowband, not single-peak.

Three of the six sources occupy unique 2D positions defined by the entropy-kurtosis joint distribution. The other three are mid-pack on both. With four more orthogonal-by-design moments in the suite (centroid, bandwidth, skewness, flatness) plus the older ApEn / LZ / Hjorth / TKEO / crest / scaling / event / symbolic lenses, the dimensionality of the per-source signature is well into double digits. The interesting scientific question is whether the six sources separate cleanly in that high-dimensional space or whether they cluster.

The kurtosis-entropy 2D snapshot suggests the answer is *separate*: even on two axes, opencode and the (vscode-XXX, hermes) pair are clear outliers in opposite directions, and claude-code is a third quadrant. The three remaining sources (openclaw, codex, plus the cluster claude-code is *almost* in) overlap, but only mildly.

## 9. The Spearman vs Pearson agreement as a sanity check

The fact that Spearman ρ = −0.5429 and Pearson r = −0.5493 differ by less than 0.01 is itself meaningful. Spearman is rank-based and insensitive to value-spread; Pearson is value-based and sensitive to outliers. When they agree this tightly, it means the relationship between the two metrics is approximately monotone-and-linear across the six points — there isn't a single source pulling Pearson around while Spearman holds steady.

If, for example, opencode's kurtosis of 6.11 had been an extreme outlier driving a spurious Pearson correlation, we would expect Pearson to be much further from Spearman (Pearson would be more negative, Spearman less negative). The 0.0064 gap rules that out. The anti-correlation is real and is distributed across the source set, not anchored on opencode alone.

## 10. What to watch next

Three follow-up questions the data invite:

- **Stability over windows.** The current numbers are from 1,735–1,739-row snapshots taken about an hour apart. Re-running the two lenses on a 24-hour-rolling window and on a 7-day-rolling window would tell us whether the −0.54 correlation is structural or whether it's an artifact of the particular tick distribution sampled at 23:42Z.
- **Per-source-pair ablation.** Drop opencode and recompute Spearman on the remaining 5 sources. If ρ drops near zero, opencode is the entire signal and the inversion isn't a property of the harness population. If ρ stays around −0.5, the anti-correlation is genuinely distributed.
- **Joint with the centroid lens.** v0.6.148's `source-row-token-spectral-centroid` would let us test whether opencode's high-kurtosis low-entropy signature really is a single dominant frequency (high centroid concentration + narrow bandwidth) or whether it's a sharper version of claude-code's narrowband (multiple close bins). This is the next obvious cross-axis study.

For now: spectral entropy and spectral kurtosis are running on the same six sources, on the same data, four days apart in implementation time. They agree on the existence of three regimes (dual-peaked opencode, dual-flat vscode-XXX/hermes, mid-pack openclaw/codex). They disagree about the placement of claude-code, which lands at the entropy floor without a kurtosis spike — the first hint that "narrowband" is a distinct PSD class from both "single-frequency" and "white".

The Spearman ρ = −0.5429 is the headline. The fact that it's only half-strength, not near −1, is the substance.
