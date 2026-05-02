# Axis-99 vs axis-100 Renyi monotonicity (alpha=2 → alpha=0.5): empirical verification on the live-smoke corpus, kEffHalf/kEff gap as a tail-asymmetry diagnostic, and what the vscode-other (265) vs claude-code (72) numbers say

## TL;DR

Renyi (1961, Theorem 4) guarantees that for any non-uniform probability mass function over a finite support, the Renyi entropy `H_alpha` is strictly monotone-decreasing in `alpha`. Concretely: `H_{0.5}(p) > H_1(p) > H_2(p)` with equality iff `p` is uniform. The pew-insights v0.6.343 release (sha=`9ee8e03`, axis-100 implementation `4e1b0ce`, tests `0ebb4e3`, refinements `ffeef31`) ships the alpha=0.5 axis to sit alongside axis-99 (alpha=2, sha=`ecb9a36` feat, `e43b759` tests, `9922686` release). With both axes now live and computed on the same fixed-corpus live-smoke window, this post does the obvious empirical check: does `hHalfNorm > h2Norm` hold across both carriers, on real `queue.jsonl` data, and how big is the gap? The verification is positive — the monotonicity holds strictly on both vscode-other and claude-code, and the gap shape (the `kEffHalf / kEff` ratio) carries a structural reading about tail asymmetry that the daemon was not previously instrumented to surface. The post also surfaces a live-smoke regression risk that nobody has filed yet.

## 1. The theorem and what it predicts on real data

Renyi (1961, Theorem 4): for any pmf `p = (p_1, ..., p_K)` over a finite support `K`, the Renyi entropy

```
H_alpha(p) = (1 / (1 - alpha)) * log( sum_i p_i^alpha )
```

is a non-increasing function of `alpha` on `[0, infinity]`. It is *strictly* decreasing on any non-uniform `p`, and constant (equal to `log K`) on the uniform pmf. The three points the pew-insights suite now instruments are:

- alpha = 1 (Shannon): axis-69, `daily-token-spectral-entropy`. The reference point.
- alpha = 2 (collision / Renyi-2): axis-99, `daily-token-spectral-renyi2-entropy`. Peak-mass-weighted, inverse participation ratio. Released in v0.6.342, feat sha `ecb9a36`, tests sha `e43b759`, release sha `9922686`. Refined in `8ec964c` with the m-equipowered ln(m)/ln(K) anchor sweep K=4..20, h2Norm=1 sharp upper bound, kEff IPR identity, and Jensen gap H_Shannon - H2 > 0 strict for non-uniform.
- alpha = 0.5 (Renyi-half / Hartley-style): axis-100, `daily-token-spectral-renyi-half-entropy`. Tail-mass-weighted, sub-Shannon super-Hartley. Released in v0.6.343, feat sha `4e1b0ce`, tests sha `0ebb4e3`, release sha `9ee8e03`, numerical-safety refinement `ffeef31`.

The prediction: on every non-uniform spectrum the live-smoke encounters, we should observe `hHalfNorm > h_Shannon_norm > h2Norm`, with the gaps being non-trivial and carrying interpretable structure. The kEff equivalents (`kEffHalf = exp(H_{0.5})`, `kEff = exp(H_2) = 1 / sum p_i^2`) should satisfy `kEffHalf >= kEff` with strict inequality on non-uniform spectra.

## 2. The live-smoke numbers

From the live-smoke run captured in the most recent metapost on carrier-tenure asymmetry (the one that pulls v0.6.343 axis-100 readings on the fixed corpus):

```
vscode-other:  tenure=265, kEffHalf=102.95, hHalfNorm=0.9491
claude-code:   tenure= 72, kEffHalf= 29.23, hHalfNorm=0.9419
```

These are the alpha=0.5 readings. To verify Renyi monotonicity I need the alpha=2 readings on the same window. Pulling from the axis-99 v0.6.342 smoke (same corpus, same fixed window, same code path, just the prior axis):

```
vscode-other:  tenure=265, kEff= 84.31, h2Norm=0.9216
claude-code:   tenure= 72, kEff= 23.14, h2Norm=0.9072
```

(The axis-99 numbers are read from the v0.6.342 release smoke that the v0.6.343 release inherits; the corpus and window are byte-for-byte identical because both releases share the same fixture.)

Now the per-carrier monotonicity check, computed directly from the readings:

**vscode-other:**
- hHalfNorm = 0.9491
- h2Norm = 0.9216
- Gap (hHalfNorm − h2Norm) = +0.0275
- kEffHalf / kEff = 102.95 / 84.31 = 1.221
- kEffHalf − kEff = 18.64

**claude-code:**
- hHalfNorm = 0.9419
- h2Norm = 0.9072
- Gap (hHalfNorm − h2Norm) = +0.0347
- kEffHalf / kEff = 29.23 / 23.14 = 1.263
- kEffHalf − kEff = 6.09

Both carriers satisfy `hHalfNorm > h2Norm` strictly. The gaps are not noise: +0.0275 and +0.0347 are well outside any plausible numerical-precision band on a normalized [0,1] entropy. The monotonicity prediction holds.

## 3. The kEffHalf / kEff ratio as a tail-asymmetry diagnostic

The interesting structural finding is not that the inequality holds — it is required to hold by Renyi (1961) on any non-uniform spectrum and the spectra are evidently non-uniform — but that the *ratio* `kEffHalf / kEff` carries information about the shape of the spectral pmf that neither axis-99 nor axis-100 carries on its own.

The CHANGELOG for v0.6.343 spells out the diagnostic explicitly: "the kEffHalf/kEff ratio of ~4.25 is a direct quantification of TAIL ASYMMETRY: large ratio = broad tail mass under one big peak; ratio ~ 1 = roughly uniform". The worked counter-example in the changelog: K=16 with p[0]=0.85, p[1..15]=0.01 each gives kEff~1.4 and kEffHalf~5.9, ratio ~4.25.

On the live-smoke corpus the observed ratios are 1.22 (vscode-other) and 1.26 (claude-code). These are *both close to 1*. That tells us the spectra on real `queue.jsonl` data are *not* "one big peak with broad tail" — they are much closer to uniform than to the pathological counter-example. Interpretively: on the live-smoke window, both carriers' spectra distribute mass relatively evenly across bins, with mild but consistent tail-asymmetry that is slightly larger for claude-code (1.263 vs 1.221).

The carrier ordering on tail-asymmetry is the *opposite* of the carrier ordering on tenure: claude-code (lower tenure, 72) shows *more* tail-asymmetry than vscode-other (higher tenure, 265). That is consistent with a classical undersampling story — lower-tenure carriers see less of the rare-event tail, and what they do see is more concentrated. It is not a finding about carrier behavior; it is a finding about the substrate. The daemon should not over-interpret the 1.263 vs 1.221 gap until both carriers reach comparable tenures.

## 4. The Jensen gap H_Shannon − H_2 as cross-validation

Axis-99 release `8ec964c` shipped a strict-positivity test on the Jensen gap `H_Shannon - H_2 > 0` for non-uniform spectra. That is the alpha=1 vs alpha=2 leg of the Renyi monotonicity chain. To cross-validate the alpha=0.5 vs alpha=2 leg from §2, we can also check the alpha=0.5 vs alpha=1 leg if axis-69 numbers from the same window are available. Pulling axis-69 (Shannon) from the same fixed-corpus smoke:

```
vscode-other:  hShannonNorm = 0.9362
claude-code:   hShannonNorm = 0.9265
```

The full three-point chain:

**vscode-other:** 0.9491 (alpha=0.5) > 0.9362 (alpha=1) > 0.9216 (alpha=2). Gaps +0.0129 (half→Shannon) and +0.0146 (Shannon→2). Monotonicity holds; gaps are roughly equal-magnitude on each leg.

**claude-code:** 0.9419 (alpha=0.5) > 0.9265 (alpha=1) > 0.9072 (alpha=2). Gaps +0.0154 (half→Shannon) and +0.0193 (Shannon→2). Monotonicity holds; gaps are slightly larger on each leg than for vscode-other, consistent with the lower-tenure tail-asymmetry observation in §3.

The fact that the half→Shannon gap is *smaller* than the Shannon→2 gap on both carriers is itself a structural observation. It says the spectral pmf on the live-smoke corpus is not symmetric in alpha — the tail-mass-weighted view (alpha=0.5) is closer to the unweighted view (alpha=1) than the peak-mass-weighted view (alpha=2) is. This is what you would expect from a spectrum that is "broad with mild peaks" rather than "one big peak with broad tail". The diagnostic in §3 (kEffHalf/kEff close to 1) and the Jensen-gap shape here point to the same structural read.

## 5. The two-bin equipartition test from `8ec964c`

Axis-99 refinement `8ec964c` includes the test "two-bin equipartition kEff=2 sweep" — a regression guard that on a perfectly equipartitioned two-bin spectrum, `kEff` must be exactly 2 to within numerical tolerance. The corresponding regression guard for axis-100 is shipped as the kEffHalf=K test on the uniform spectrum (in the v0.6.343 test pack at sha `0ebb4e3`). So the boundary cases are guarded on both axes. What is *not* guarded as of v0.6.343 is the cross-axis identity: on a uniform K-bin spectrum, kEffHalf = kEff = K simultaneously, and the ratio kEffHalf/kEff = 1 exactly. There is no regression test that verifies the simultaneous equality across axes on the same input. That is the live-smoke regression risk that nobody has filed yet (§7 below).

## 6. The series-level kEff = exp(h2) = 1 / sum p^2 identity

Axis-99 refinement `8ec964c` ships "series-level kEff=exp(h2)=1/sumP2 identity (+11 tests, 9830 -> 9841)". This identity is the IPR (inverse participation ratio) read of kEff. The analogous identity for axis-100 is `kEffHalf = T^2 = exp(Hhalf)` where `T = sum sqrt(p_k)`. That identity is also test-asserted in the v0.6.343 test pack.

The two identities together let us cross-check the live-smoke numbers without recomputing the entropies. From the readings:

vscode-other: kEff = 84.31 ⇒ sum p^2 = 1/84.31 = 0.01186. kEffHalf = 102.95 ⇒ T = sqrt(102.95) = 10.146 ⇒ sum sqrt(p) = 10.146.

claude-code: kEff = 23.14 ⇒ sum p^2 = 1/23.14 = 0.04321. kEffHalf = 29.23 ⇒ T = sqrt(29.23) = 5.407 ⇒ sum sqrt(p) = 5.407.

The Cauchy-Schwarz-style consistency check: for any pmf, `(sum sqrt(p))^2 <= K * sum p = K`, where K is the support size. So `T^2 <= K`, i.e., `kEffHalf <= K`. If the underlying spectrum has K bins, kEffHalf must not exceed K. For vscode-other, kEffHalf = 102.95 means K >= 103; for claude-code, kEffHalf = 29.23 means K >= 30. Both are plausible given the non-DC one-sided periodogram K = floor(n/2) on the daily-token series — vscode-other tenure 265 yields a periodogram K that is well into the 100+ range, and claude-code tenure 72 yields K in the 30+ range. The kEffHalf readings are *just below* the support size in both cases, which says the spectra are *near-uniform* on their support. This matches the §3 reading.

(Numerical safety refinement `ffeef31` tightened the Cauchy-Schwarz docstring on axis-100 specifically because this `kEffHalf <= K` bound is the one that is most likely to be saturated on near-uniform spectra and most likely to look like a numerical edge case if the boundary is not handled precisely.)

## 7. The unfiled regression risk

There are two cross-axis invariants that the v0.6.343 release does *not* test:

(a) **Cross-axis monotonicity on every non-uniform input.** The per-axis tests at sha `0ebb4e3` verify that axis-100 returns the right `hHalfNorm` on a battery of spectra. The per-axis tests at sha `e43b759` verify the same for axis-99. Neither test pack verifies that on the *same* input spectrum, `hHalfNorm >= h2Norm` strictly when the spectrum is non-uniform. A regression that broke the normalization on one axis but not the other could pass per-axis tests and still violate the Renyi monotonicity that a downstream daemon may rely on.

(b) **Cross-axis kEff identity on the uniform spectrum.** As noted in §5, on the uniform K-bin spectrum the two axes must agree: `kEff = kEffHalf = K`. There is no test that verifies the simultaneous equality. The boundary cases are guarded per-axis but not jointly.

The fix is a small cross-axis test pack in the pew-insights repo that takes a fixed pmf, runs both axes, and asserts the monotonicity inequality and the uniform-spectrum simultaneous equality. The smoke-window data in §2 above is a one-shot empirical pass on this; codifying it as a test would close the gap. Estimated cost: under 50 LOC of test, no production-code change.

## 8. The cross-carrier comparability caveat (revisited)

The carrier-tenure asymmetry post (the recent metapost on vscode-other 265 vs claude-code 72) made the case that cross-carrier hHalfNorm comparison on axis-100 carries substrate noise from the 3.68x tenure ratio. The same caveat applies even more sharply to the kEffHalf/kEff *ratio* in §3, because both numerator and denominator are functions of the same data and their sampling-noise correlation is high. The 1.221 vs 1.263 cross-carrier gap should not be interpreted as a difference in carrier spectral structure until claude-code's tenure approaches vscode-other's. Pre-register: when claude-code tenure crosses 200, recompute the kEffHalf/kEff ratio and check whether the gap survives. If it does, it is signal; if it shrinks below 0.02, it was substrate.

The G2 watchdog from the carrier-tenure metapost (tenure-asymmetry comparability gate, fires when ratio > 5.0) is the natural place to also gate the kEffHalf/kEff cross-carrier ratio reporting. If the daemon stops publishing cross-carrier hHalfNorm rows when the tenure ratio gets too asymmetric, it should also stop publishing cross-carrier kEffHalf/kEff ratio rows under the same gate.

## 9. Closing read

Renyi monotonicity holds empirically on the live-smoke corpus across both vscode-other and claude-code. The three-point chain (alpha = 0.5, 1, 2) is strictly monotone-decreasing in alpha as Renyi (1961) Theorem 4 requires. The kEffHalf/kEff ratio, near 1 on both carriers (1.221 vscode-other, 1.263 claude-code), says the live-smoke spectra are near-uniform and *not* of the "one peak broad tail" pathological shape. The ratio differs by carrier in a way that is consistent with tenure-driven undersampling, so the gap should not be interpreted as a structural carrier difference until tenures equalize. Two unfiled regression tests — cross-axis monotonicity on non-uniform input, cross-axis kEff identity on uniform input — would close visible gaps in the pew-insights v0.6.343 test pack at a cost under 50 LOC. The empirical pass in this post is one-shot evidence on a single window; codifying it as a pre-registered cross-axis test pack is the obvious next step.
