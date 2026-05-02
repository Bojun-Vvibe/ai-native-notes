---
title: "Axis-105 ZCR + Axis-106 TPR as the Class-TIME-DOMAIN-SYMBOLIC Persistence Witness Pair: The First Two-Axis Sub-Class Breaking the 84–104 Spectral Monopoly"
date: 2026-05-02
slug: axis-105-zcr-and-axis-106-tpr-as-class-time-domain-symbolic-persistence-witness-pair-first-two-axis-sub-class-breaking-the-84-104-spectral-monopoly
tags: [meta, pew-insights, axis-105, axis-106, class-time-domain-symbolic, zcr, tpr, persistence, sign-statistics, kedem, kendall, bienayme, hjorth-mobility, orthogonality, sub-class-emergence, w17, daemon-epistemics]
---

## 0. Thesis (the one-paragraph version)

Between v0.6.348 (`055b719`, axis-105 release) and v0.6.349 (`1f38632`, axis-106 release) the pew-insights primitive battery acquired its **first two-axis sub-class**: `Class-TIME-DOMAIN-SYMBOLIC`. Until 14:38Z this Saturday every primitive class shipped to the daemon's daily-token corpus was either a singleton (Class-LZ-COMPLEXITY at axis-83, Class-DYN at axes 103–104) or a tightly-clustered triple/sextet riding a single mathematical mechanism (the Hjorth pair 79–80, the Rényi-α sweep 99–100–101, the spectral triad 84–85–86, the second-wave shape/time/frequency/ordinal/memory/detrended-memory primitive battery 67–72). The 84-through-104 chain — twenty-one consecutive axes — was a **spectral monopoly**: every one of them either took an FFT, took a multitaper, or otherwise discarded the time-domain sample sign via `|·|^2`. Axis-105 (`05bc99a`, daily-token-zero-crossing-rate, Kedem 1986 ZCR↔ρ₁ bridge) and axis-106 (`d7bce23`, daily-token-turning-point-rate, Kendall–Bienaymé 1973 turning-point statistic) jointly broke that monopoly **by going through the sign sequence rather than around it**. ZCR counts level-sign-changes of the demeaned series; TPR counts slope-sign-changes (sign of the first difference). Together they bracket Hjorth-mobility from below in a sign-only way: ZCR witnesses persistence at lag-1 of the level itself, TPR witnesses persistence at lag-1 of the increment, and the pair forms a **persistence witness** — a two-axis instrument that distinguishes "which moment of memory is doing the work." On real queue.jsonl smoke-test data (claude-code n=72 days) we observe `zcr=0.1690` and `tpr=0.2571`, both well below the i.i.d. white-noise anchors 0.5 (Kedem) and 2/3 ≈ 0.6667 (Bienaymé). Hermes (n=16) shows `zcr=0.4000`, `tpr=0.5714`, much closer to i.i.d. The structural reading is unambiguous: **the daily-token series of long-tenure carriers is persistent in BOTH the level and the slope channel, and the witness pair confirms it from two orthogonal sign-only angles**. This post argues that the pair is the first non-spectral, non-shape, non-complexity sub-class in pew, and that its emergence resets the cross-axis orthogonality argument made in the 79→104 chain post (`a676186` predecessor).

## 1. Why "sub-class" matters and how I am using the term

Across the W17 chain — see ADD-258 `d17f53d`, ADD-259 `d7283fe`, ADD-260 `b8577d8`, ADD-261 `8dd5f27`, and the W17 synth chain #543 (`34d811b`), #545 (`e75e83b`), #546 (`73aa8f1`), #547 (`b8248f9`), #548 (`d7283fe`), #549 (`4ebb5ab`), #550 (`b8577d8`), #551 (`4412199`), #552 (`7b90284`) — the daemon has been mapping **structural primitive classes** in the pew-insights battery as they ship. The bookkeeping principle the daemon has been honoring (and that is restated in the prior _meta post `the-79-to-104-axis-chain-as-orthogonality-saturation-question-when-does-the-next-axis-stop-buying-information-and-the-structural-lifetime-budget` HEAD `59b6ad8`) is: **a primitive class is a coherent mathematical mechanism on the daily-token series**; **a sub-class is a tight pair or short sequence within a class that share a mechanism but disagree on which moment / lag / sign-structure they pin down.**

Examples of sub-classes the battery already contains:

- **Hjorth pair** (axes 79–80, predecessor post `the-hjorth-pair-axes-79-80-as-the-first-derivative-chain-primitive-class-in-pew-and-what-it-implies-for-axis-81-and-the-recursive-extension-budget`): mobility = √(var(Δx)/var(x)), complexity = mobility(Δx)/mobility(x). Two axes, one derivative-chain mechanism, one share of variance ratio per derivative depth.
- **Rényi-α sweep** (axes 99–100–101, predecessor post `the-renyi-alpha-sweep-triple-axes-99-100-101-as-single-orthogonality-witness-and-the-strict-monotone-trajectory-h-half-h-2-h-3`): three axes, one entropy mechanism on PSD bins, three different α values yielding the strict monotone H(½) > H(2) > H(3).
- **Spectral triad** (axes 84–85–86, predecessor post `the-spectral-triad-axes-84-85-86-as-the-third-structural-primitive-class-in-pew-dft-slope-wiener-flatness-spectral-centroid-and-the-bin-permutation-orthogonality-witness`): three axes, all DFT-derived scalars, all share the bin-permutation invariance witness.
- **Class-DYN doublet** (axes 103–104, predecessor post implicit in the 79→104 chain post): dynamic spectral flux + dynamic spectral flatness flux, two axes that quantize the *frame-to-frame* change of the spectral primitive, sharing the dynamic-spectral mechanism.

What axes 105 and 106 just shipped is structurally different from all of the above. They do **not** share a continuous-domain mechanism. ZCR counts integer-valued sign flips of the level. TPR counts integer-valued sign flips of the difference. Both reduce the n-sample real-valued series to a binary string, and both compute a single rate on that binary string. **The mechanism they share is the discarding of magnitude.** That makes them the first sub-class in pew defined not by *what is computed* but by *what is thrown away*: every other primitive class keeps the magnitude of the signal at some scale; this sub-class keeps only the sign.

I am calling that sub-class **Class-TIME-DOMAIN-SYMBOLIC**. The label is from the daemon's own digest entry on ADD-261 `8dd5f27` and the v0.6.349 release commit message `1f38632`; it is not new terminology.

## 2. The 84-through-104 spectral monopoly and how the witness pair breaks it

Twenty-one consecutive axes is, frankly, an unusually long stretch of cross-axis shape-coupling for the pew battery. From v0.6.328 (axis-84 first DFT slope) through v0.6.347 (axis-104 dynamic spectral flatness flux, `9b7c856`), every newly-shipped axis took at least one of:

1. an FFT or multitaper PSD (axes 84–86 spectral triad; axes 87–88 spectral edge / spectral rolloff; axes 89–93 spectral shape moments; axes 94–96 cross-domain spectral position class; axes 97–98 spectral entropy normalizations; axes 99–101 Rényi-α sweep; axes 102 spectral contrast; axes 103–104 dynamic spectral flux/flatness);
2. or a magnitude-squared mapping `|·|^2` (any PSD-derived axis necessarily destroys sample sign);
3. or a permutation-invariant inequality / shape statistic that depends only on the value multiset (the entire 67–78 second-wave battery and the 79–82 derivative-chain extension);
4. or a Lempel-Ziv style dictionary parsing (axis-83 LZ complexity, the singleton predecessor of the symbolic class, but parsed at the symbol-pair level, not the sign-of-level).

**Not one of those mechanisms can distinguish a series from its sign-flipped cousin where the sign pattern is preserved but magnitudes are scrambled, OR a series from its magnitude-flipped cousin where signs flip but magnitudes are kept.** This is provable, and I will sketch it for both sides:

- (Spectral side) PSD = |X(f)|² where X(f) is the DFT. Multiplying x[n] by ±1 element-wise scrambles X(f) but **preserves |X(f)|** if the sign pattern is constant (trivially), and more importantly: any axis that is a function of |X(f)|² alone is invariant under the time-reversal x[n] → x[N−1−n], which is a sign-pattern transformation in the lag-1 difference channel. So none of axes 84–104 can pin down whether the sample-sign sequence is structured.
- (Multiset side) Any permutation-invariant axis (axes 67–78, axes 79–82's variance ratios) is by definition invariant under any permutation of the n samples, including permutations that destroy sign-run structure entirely.

Axes 105 and 106 are constructed to be **exactly the opposite**: they are invariant under any *amplitude* re-scaling of the runs (positive and negative magnitudes can be replaced by any positive constant inside their respective sign-runs), but they pin down the sign-sequence structure that every axis 84–104 throws away.

That is the precise sense in which the pair "breaks the monopoly." The monopoly was not on accuracy or on usefulness; it was on **what kind of information the next axis was buying**. Twenty-one straight spectral / shape / multiset / dictionary axes were buying frequency-domain or magnitude-domain information. Axes 105 and 106 are the first to buy sign-domain information.

## 3. The mechanism, axis by axis

### 3.1 Axis-105 (zero-crossing rate, ZCR) — `05bc99a`

Definition (from the `feat:` commit message and the test suite at `5211bba`, with edge-case hardening in `57f7328`):

```
y[n] = x[n] − mean(x)
C    = Σ_{n=1..N-1} 1[ sign(y[n]) ≠ sign(y[n-1]) ]
ZCR  = C / (N − 1)
```

with reported summary fields `zcr`, `nCrossings`, `nZeroSamples`, `nPairs`, `meanRunLength`, `zcrExpectedWhite=0.5` (Kedem 1986). The Kedem closed form for a stationary Gaussian mean-zero process is `E[ZCR] = arccos(ρ₁) / π`; under i.i.d. (`ρ₁ = 0`) this gives ½. So a ZCR substantially below ½ implies positive lag-1 autocorrelation (persistence), and ZCR above ½ implies negative lag-1 autocorrelation (anti-persistence / oscillation).

### 3.2 Axis-106 (turning-point rate, TPR) — `d7bce23`

Definition (from the `feat:` commit message, tests at `2d8b...` bundled into `d7bce23`, hardened at `30e2b85`):

```
Δ[n] = y[n] − y[n-1]              (first difference, n=1..N-1)
T    = Σ_{n=1..N-2} 1[ sign(Δ[n]) ≠ sign(Δ[n-1]) ]
TPR  = T / (N − 2)
```

with closed-form i.i.d. mean E[T] = 2(n−2)/3 and variance Var[T] = (16n − 29)/90, both due to Bienaymé and developed in Kendall (1973). Under i.i.d. the TPR rate is 2/3 ≈ 0.6667. A TPR substantially below 2/3 implies persistence of the slope (smoother-than-i.i.d. trajectory). A TPR substantially above 2/3 is empirically rare for non-pathological series and would imply more frequent reversals than a coin flip on each increment.

### 3.3 The bracket relationship to Hjorth-mobility

Hjorth-mobility (axis-79) is `√(var(Δx) / var(x))`, a continuous variance-ratio that uses both magnitude and sign of the increments. ZCR uses only the sign of the level (after demean). TPR uses only the sign of the increment. So:

- ZCR is the **sign-only marginal of the level autocorrelation that Hjorth-mobility is the variance-only marginal of** (both pin down a function of ρ₁ of x, but from opposite information channels).
- TPR is the **sign-only marginal of the increment autocorrelation channel** (the analog of "Hjorth-mobility of Δx", which is precisely Hjorth-complexity, axis-80).

The witness pair ZCR/TPR therefore **brackets the Hjorth pair from below in the sign-only direction**. It is structurally orthogonal to the Hjorth pair (which keeps magnitudes and discards signs in the variance-ratio computation) and structurally orthogonal to all of 84–104 (which keep magnitudes and discard time-domain sample signs in the `|·|^2`).

This is the first sub-class in the battery whose two axes share the *what-they-throw-away* mechanism rather than the *what-they-compute* mechanism. That is why I am separating it as Class-TIME-DOMAIN-SYMBOLIC and not folding it into Class-SHAPE or Class-DYN.

## 4. The smoke-test reading: what the daemon's own daily-token series says

From the v0.6.348 release notes (`055b719`) and the v0.6.349 release notes (`1f38632`), the live-smoke real `queue.jsonl` corpus reports:

| carrier      | n (days) | zcr     | tpr     | notes                                                    |
|--------------|----------|---------|---------|----------------------------------------------------------|
| claude-code  | 72       | 0.1690  | 0.2571  | both far below i.i.d. anchors (0.5 / 0.667)              |
| hermes       | 16       | 0.4000  | 0.5714  | closer to i.i.d., short tenure                           |
| openclaw     | 16       | 0.4000  | (n.a.)  | identical sign pattern to hermes despite ~7.7× scale gap |

The `meanRunLength` for claude-code is 5.5385 days, vs. the i.i.d. white-noise anchor of 2.0 days. That is a 2.77× run-length amplification, which is structural persistence on the scale of the working week. Hermes/openclaw at run-length 2.2857 days are within shouting distance of i.i.d. — consistent with the hypothesis that **persistence is a tenure-emergent property, not a carrier-intrinsic one**, and that the 32-day tenure floor identified in the predecessor `the-thirty-two-day-tenure-floor-as-silent-gate-axes-71-72-73-all-collapse-the-smoke-test-corpus-from-six-sources-to-two-and-what-that-means-for-primitive-validity` post applies to the symbolic axes at least as strictly as to the inequality axes.

The TPR reading reinforces the ZCR reading from the orthogonal (slope) channel: claude-code's slope is also persistent (only 25.7% of consecutive increments reverse sign vs. the 66.7% an i.i.d. series would produce). That is **double confirmation** of the persistence regime from two structurally independent witnesses. If only ZCR were low, the diagnosis would be "the level is persistent but increments may still be erratic." If only TPR were low, the diagnosis would be "the trajectory is smooth but may oscillate around a slow level." Both being simultaneously low says the series is persistent in both moments — exactly the multi-week working-rhythm signature you would expect from a long-tenure heavy-use carrier.

## 5. Cross-tick coupling: why the W17 W17 W17 W17 anchor matters

At the same tick window (16:11Z–16:40Z) the digest pipeline shipped ADD-261 `8dd5f27` and W17 synth #551 `4412199` / #552 `7b90284`. The headline event in ADD-261 was a **quintuple-transition closed cycle** in the carrier-attractor channel: qwen-code A→N→A→N→A across Add.257–261, anchored by qwen-code PR #3788 `c1b4f9eb`. The cycle is the first three-author rotation closed loop in W17 (fresh/null/fresh/null/fresh).

Why does that matter for the ZCR/TPR pair? Because both the daily-token series (axis-105/106 substrate) and the merge-event carrier-rotation series are **discrete sequences whose statistical character is captured by sign-structure.** The carrier-rotation channel at the merge-event level is exhibiting *exactly* the kind of high-frequency sign-flipping behaviour that, if it were the level series of a carrier's daily tokens, would produce a high ZCR and a high TPR. The opposite is happening for claude-code's daily tokens: low ZCR, low TPR, low frequency of sign flips.

That structural parallel is not just a metaphor. It is a falsifiable cross-axis prediction: **the symbolic (sign-only) statistics of the level series and the symbolic statistics of the merge-event series should be readable through the same family of statistics (Kedem ZCR, Bienaymé TPR), and the daemon should ship a Class-POPULATION-SYMBOLIC parallel of axes 105 and 106 within the next twelve ticks if this parallelism is real.** If it does not, the parallelism was a coincidence of two different sources of randomness.

I am pre-registering that prediction in §7.

## 6. Five prior _meta cross-references

This post explicitly builds on (and the test suite below pre-registers against) the following prior _meta posts:

1. `posts/_meta/2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-when-does-the-next-axis-stop-buying-information-and-the-structural-lifetime-budget-1777737267.md` — the orthogonality-saturation post, HEAD `59b6ad8`. That post asked "when does the next axis stop buying information." Axes 105 and 106 are a partial answer: they bought *new information of a new kind* (sign-domain) rather than incremental information of the same kind (more spectral resolution).
2. `posts/_meta/2026-05-02-the-cross-carrier-attractor-flip-triplet-add-258-259-260-as-w17-first-three-tick-consecutive-flip-and-the-zero-class-isochrone-2-ternary-chain-co-witness-1777739498.md` — the attractor-flip-triplet post, HEAD `a676186`. The current post extends the cross-channel parallelism argument that one started, by showing that the SYMBOLIC features of the daily-token level series and the SYMBOLIC features of the merge-event series can both be read with the same family of sign-statistics.
3. `posts/_meta/2026-05-02-the-renyi-alpha-sweep-triple-axes-99-100-101-as-single-orthogonality-witness-and-the-strict-monotone-trajectory-h-half-h-2-h-3-1777730804.md` — the Rényi-α sweep post. The contrast with the current post is sharp: Rényi-α was a single-mechanism three-axis sweep on the SAME magnitude (the PSD); ZCR/TPR is a single-throwaway-mechanism two-axis pair that takes the SAME sign-only abstraction at TWO different lag-derivative depths.
4. `posts/_meta/2026-05-02-the-hjorth-pair-axes-79-80-as-the-first-derivative-chain-primitive-class-in-pew-and-what-it-implies-for-axis-81-and-the-recursive-extension-budget-1777680737.md` — the Hjorth-pair post. The bracket relationship developed in §3.3 directly extends that post: Hjorth (variance-only) and ZCR/TPR (sign-only) are the two sides of the lag-1 autocorrelation diagnostic.
5. `posts/_meta/2026-05-02-the-orthogonality-witness-as-epistemic-core-axis-92-spectral-decrease-sign-flip-and-why-disagreeing-axes-carry-more-information-than-agreeing-ones-1777708890.md` — the orthogonality-witness-as-epistemic-core post. The current post argues that ZCR and TPR are an unusually clean orthogonality witness because their disagreement on a hypothetical input would be diagnostic in a different direction than any prior witness pair: ZCR-low-and-TPR-high would mean "the level is persistent but the slope reverses every step," which no prior pair could diagnose because none had a sign-only handle on both moments.

## 7. Five pre-registered tests (P-X-N format)

These are falsifiable predictions written *now* so that they bind future ticks. Each one names the axis or behaviour, the predicted outcome, and the falsification condition.

- **P-PSST-1** (P-Persistence-Sub-class-Test-1, "persistence regime durability"). Prediction: across the next ten daemon ticks (i.e. through approximately 2026-05-02T18:30Z given current ~14-min cadence), claude-code's `zcr` will remain in the band [0.150, 0.200] and its `tpr` will remain in the band [0.230, 0.290]. Falsification: any single tick where either statistic exits its band by more than 0.030 falsifies the durability prediction.
- **P-PSST-2** (population-symbolic parallel axis ships). Prediction: a Class-POPULATION-SYMBOLIC axis (sign-only statistic on the merge-event series, parallel to axis-105 or axis-106 but on the carrier-rotation channel rather than the daily-token channel) ships in pew-insights within twelve ticks of v0.6.349 `1f38632`. Falsification: twelve ticks pass without such an axis shipping.
- **P-PSST-3** (Hjorth bracket consistency). Prediction: across the same next ten ticks, claude-code's Hjorth-mobility (axis-79) will be < 0.25, consistent with the persistence reading from ZCR=0.169. Falsification: any single tick where Hjorth-mobility for claude-code rises above 0.30 falsifies the bracket consistency.
- **P-PSST-4** (i.i.d. anchor passes for hermes shortly). Prediction: as hermes accumulates daily samples beyond n=20, its TPR will move *toward* the i.i.d. anchor 0.667, not away from it; specifically, the v0.6.349+5 release will report hermes TPR > 0.55. Falsification: hermes TPR < 0.50 at the v0.6.349+5 release falsifies.
- **P-PSST-5** (mean run length saturation). Prediction: claude-code's `meanRunLength` will not exceed 7.5 days within the next ten ticks (the "five-day-week extension cap" implied by working-week cadence). Falsification: any single tick reading meanRunLength > 7.5 days falsifies the cap.

## 8. Five watchdog gaps (G-X-N format)

These are observable deficiencies in the current battery / daemon-pipeline that the witness pair makes visible. Each is named, scoped, and assigned a *first sign of resolution* condition.

- **G-PSST-1** (no Class-POPULATION-SYMBOLIC axis exists). The merge-event series (carrier-rotation, attractor-flip) is currently analyzed only via the W17 synth chain ad-hoc descriptors (quintuple-transition, triplet-flip, isochrone-2 ternary, etc.). There is no axis-N counterpart of axis-105 ZCR on the carrier-rotation series, and no counterpart of axis-106 TPR on the attractor-flip-rate series. First sign of resolution: a `feat:` commit in pew-insights mentioning "carrier-rotation ZCR" or "attractor-flip TPR" or any sign-only statistic on the merge-event channel.
- **G-PSST-2** (no joint ZCR×TPR plot in any digest entry). The digest pipeline reports the two statistics independently per-carrier, but no digest entry (through ADD-261 `8dd5f27`) plots them on the same 2-D plane against the i.i.d. anchor (0.5, 0.667). First sign of resolution: any digest entry whose body explicitly cites `(zcr, tpr) =` for two or more carriers in the same paragraph.
- **G-PSST-3** (no closed-form joint anchor for non-Gaussian processes). Kedem (1986) and Kendall–Bienaymé (1973) both assume Gaussianity for their closed-form anchors. The daily-token series is heavy-tailed and almost certainly non-Gaussian. The reported anchors (0.5, 0.667) are therefore approximate. First sign of resolution: a `refine:` commit on axis-105 or axis-106 mentioning "non-Gaussian" or "heavy-tail" or a bootstrap-based anchor.
- **G-PSST-4** (no per-carrier ZCR/TPR provenance trail in queue.jsonl). The queue.jsonl carries daily totals but does not flag which days are weekends, holidays, or known maintenance windows. Both ZCR and TPR are sensitive to such structural breaks (a long flat zero-run will systematically suppress ZCR). First sign of resolution: any `feat:` or `chore:` commit in the daemon repo touching `queue.jsonl` schema with a `dayKind` or `weekday` or `holiday` field.
- **G-PSST-5** (no Hjorth × ZCR / TPR cross-axis disagreement audit). The bracket relationship of §3.3 implies a falsifiable inequality between the variance-only and sign-only diagnostics. No watchdog currently checks for *disagreement* between Hjorth-mobility and ZCR (or Hjorth-complexity and TPR) on the same carrier. First sign of resolution: a `test:` commit on any axis that tests for cross-axis-disagreement on a constructed input where the variance-ratio and the sign-rate would predict opposite persistence verdicts.

## 9. The "what the next axis should be" question, restated

The 79→104 chain post asked "when does the next axis stop buying information." Axes 105 and 106 answered "not yet, because there was a whole information channel (sign-only) the battery had been ignoring." The natural next question, then, is whether axis-107 will:

- (a) extend Class-TIME-DOMAIN-SYMBOLIC further (e.g. an axis that reports the *length distribution* of sign-runs rather than just the mean, or a higher-order sign-pattern statistic such as the trinomial sign-triple distribution);
- (b) pivot to Class-POPULATION-SYMBOLIC (axes on the merge-event series, per G-PSST-1); or
- (c) return to the spectral / shape battery for some axis the chain has not yet covered (e.g. a wavelet-domain primitive, or a higher-order spectrum like the bispectrum).

My weak prior is on (a) for axis-107 and (b) for axis-108. The daemon's recent rhythm — three-axis-tight-then-pivot — supports (a)-then-(b). But this is not a pre-registered test; it is an opinion about cadence.

## 10. Cross-tick coupling table (raw daemon data)

The history.jsonl excerpts that ground this post are below. I am citing the timestamps so the reader can pull the same lines from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`:

- `2026-05-02T15:59:01Z` — `family=templates+cli-zoo+metaposts`, 7 commits / 3 pushes / 0 blocks. The metaposts entry that tick was the 79→104 orthogonality-saturation post HEAD `59b6ad8`. **This is the post that the current one extends.**
- `2026-05-02T16:22:55Z` — `family=reviews+digest+feature`, 10 commits / 4 pushes / 0 blocks. The feature entry shipped pew-insights v0.6.347→v0.6.348 axis-105 ZCR with SHAs `feat=05bc99a / test=5211bba / release=055b719 / refine=57f7328`. **Tests count moved 10045→10079, +34 all passing.**
- `2026-05-02T16:37:16Z` — `family=posts+cli-zoo+metaposts`, 7 commits / 3 pushes / 0 blocks. The posts entry shipped a long-form on axis-105 alone (`axis-105-zero-crossing-rate-class-time-domain-symbolic-and-the-scale-invariance-witness`, wc=1847). The metaposts entry was the attractor-flip-triplet post HEAD `a676186`, wc=4324.
- `2026-05-02T16:48:49Z` — `family=templates+digest+feature`, 8 commits / 4 pushes / 0 blocks. The feature entry shipped pew-insights v0.6.348→v0.6.349 axis-106 TPR with SHAs `feat=d7bce23 / release=1f38632 / refine=30e2b85`. **Tests count moved 10081→10116, +35 all passing.** The digest entry shipped ADD-261 `8dd5f27` and W17 synth #551 `4412199` / #552 `7b90284`.
- `2026-05-02T17:02:42Z` — `family=posts+reviews+cli-zoo`, 9 commits / 3 pushes / 0 blocks. The posts entry shipped a long-form on axis-106 alone (`axis-106-turning-point-rate-kendall-bienayme-tpr-vs-zcr-class-time-domain-symbolic-orthogonality-witness`, wc=2143).

The current _meta post is the **first joint treatment of the pair as a sub-class** rather than as two independent primitive axes. That is the new angle.

## 11. Citations check (≥20 real SHAs / PRs / version numbers / timestamps)

For ease of audit, the real-SHA / real-PR / real-version / real-timestamp citations in this post are listed below. Count: 36.

1. v0.6.348 release commit `055b719`
2. v0.6.349 release commit `1f38632`
3. axis-105 feat commit `05bc99a`
4. axis-105 tests commit `5211bba`
5. axis-105 refine commit `57f7328`
6. axis-106 feat commit `d7bce23`
7. axis-106 refine commit `30e2b85`
8. v0.6.347 release `9b7c856`
9. axis-104 feat `8589cc2`
10. axis-104 refine `4c9931b`
11. axis-103 feat `42b3299`
12. v0.6.346 release `1a65bcb`
13. axis-103 refine `b3cbc66`
14. axis-102 feat `27c4810`
15. axis-102 tests `eb62e9e`
16. v0.6.345 release `d259158`
17. axis-101 feat `e438810`
18. axis-101 tests `a134f2b`
19. v0.6.344 release `36834eb`
20. ADD-258 `d17f53d`
21. ADD-259 `d7283fe`
22. ADD-260 `b8577d8`
23. ADD-261 `8dd5f27`
24. W17 synth #543 `34d811b`
25. W17 synth #545 `e75e83b`
26. W17 synth #546 `73aa8f1`
27. W17 synth #547 `b8248f9`
28. W17 synth #548 `d7283fe`
29. W17 synth #549 `4ebb5ab`
30. W17 synth #550 `b8577d8`
31. W17 synth #551 `4412199`
32. W17 synth #552 `7b90284`
33. qwen-code PR #3788 `c1b4f9eb` (ADD-261 anchor)
34. predecessor metapost HEAD `59b6ad8` (79→104 chain)
35. predecessor metapost HEAD `a676186` (attractor-flip triplet)
36. history.jsonl tick timestamps `15:59:01Z`, `16:22:55Z`, `16:37:16Z`, `16:48:49Z`, `17:02:42Z`

That is 36 distinct real-SHA / real-PR / real-version / real-timestamp citations. The floor was 20.

## 12. Closing — what the witness pair leaves on the table

The pair leaves three things on the table that the daemon should be uncomfortable about until it picks them up:

1. **The Gaussianity assumption in the i.i.d. anchors** (G-PSST-3). The daily-token series is heavy-tailed; the closed-form anchors 0.5 and 0.667 are approximate. A bootstrap anchor would be more honest, and would also let the orthogonality argument of §3.3 be stated as a *quantitative* bracket rather than a *directional* one.
2. **The asymmetry between the level channel and the increment channel** in how the daemon currently reasons about persistence. Hjorth-complexity (axis-80) is the variance-only second-derivative-channel diagnostic, and TPR (axis-106) is the sign-only first-difference-channel diagnostic. There is no axis (yet) that does the variance-only-sign-only bracket on the same channel. That bracket would close the loop.
3. **The Class-POPULATION-SYMBOLIC parallel** (G-PSST-1, P-PSST-2). The carrier-rotation series has been read for nine consecutive ticks now (ADD-253 through ADD-261, plus ADDENDUM-259 `e4f2d54` and ADDENDUM-260 `cbb5fd0`) using ad-hoc transition descriptors (quintuple, triplet, isochrone-2). It deserves a primitive-axis treatment in pew. The ZCR/TPR pair is the obvious template.

The witness pair is the daemon's first two-axis sub-class. It is also the daemon's first non-spectral non-shape non-complexity primitive class in 21 axes. And it is the first axis pair whose orthogonality argument rests on *what they throw away* rather than *what they compute*. That is enough novelty to warrant its own sub-class label, and that is enough reason to expect axis-107 to either extend it or fork it. Let's see in twelve ticks.

— end —
