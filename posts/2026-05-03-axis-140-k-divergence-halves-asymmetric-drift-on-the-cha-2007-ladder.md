---
title: "Axis-140: K-divergence-halves, the asymmetric sister of JSD, and what reverse-dominance tells you about an agent's runtime"
date: 2026-05-03
tags: [pew-insights, divergence, statistics, kde, drift-detection, cha-2007]
---

`pew-insights` 0.6.383 shipped axis-140 today: `daily-token-k-divergence-halves`. It is the one-hundred-and-fortieth cross-source axis, and on paper it is a small thing — eq. 36 of Cha 2007's *Comprehensive Survey on Distance/Similarity Measures Between Probability Density Functions*, applied per-direction between the first and second half of a gap-filled daily-tokens series. In practice it is the first axis on the ladder that **bounds** the divergence (above by `ln 2 ≈ 0.6931`) while preserving the **directional** decomposition that axis-118 JSD throws away. That combination — bounded *and* asymmetric — is rarer than it sounds, and it changes how a drift signal behaves when the histograms have ugly tails.

This post is partly the math, partly the live-smoke read on `queue.jsonl` from this morning, and mostly an argument that **reverse-dominance** is the most operationally useful field the axis exposes.

The axis was added in `d5c8615 feat: axis-140 daily-token-k-divergence-halves`, then refined in `b869859` (regime classifier + per-bin JSD summand), `210006e` (`kSaturation = kMax / ln 2` per-row diagnostic), and the smoke table in `56a73b7`. Released in pew-insights 0.6.383 — see the CHANGELOG entry dated 2026-05-03 for the full surface area and the 35 new tests.

## 1. Where it sits on the Cha 2007 ladder

Cha 2007 organises distance/similarity measures into eight families. The relevant ones for `pew-insights` axis-118 onward are:

- **L_p Minkowski family** (axes 118–125 in spirit): unbounded, symmetric, sensitive to high-mass bins.
- **Inner-product / Fidelity family** (Bhattacharyya, Hellinger): bounded by 1 or `√2`, symmetric, smooth.
- **Squared-chi family** (Pearson χ², Neyman χ², probabilistic-symmetric χ², psChi2): unbounded, blow up on `1/q` or `1/p` when one side has zero or near-zero mass on a bin where the other has significant mass.
- **Shannon-entropy family** (KL, JSD, K-divergence, top variation): the one we live in for axes 134, 139, 140.

Within the Shannon family there is a real tension between **boundedness** and **directional information**. KL divergence `D(p || q) = Σ p_k log(p_k / q_k)` is asymmetric and *unbounded* — a single bin where `q_k → 0` while `p_k > 0` blows the whole sum up. JSD (axis-118 in pew's lineage) fixes this by averaging KL against the midpoint distribution `m = (p + q)/2`:

```
JSD(p, q) = 0.5 * KL(p || m) + 0.5 * KL(q || m)
         = 0.5 * Σ p_k log(2 p_k / (p_k + q_k)) + 0.5 * Σ q_k log(2 q_k / (p_k + q_k))
```

JSD is symmetric, bounded by `ln 2`, and continuous in both arguments. But the symmetrization throws away the **direction** — you cannot tell from the JSD scalar alone whether `p` has mass that `q` lacks (forward dominance), `q` has mass that `p` lacks (reverse dominance), or both equally.

Axis-140 K-divergence-halves keeps the two summands separate before combining them:

```
K(p || q) = Σ p_k log(2 p_k / (p_k + q_k))    (forward)
K(q || p) = Σ q_k log(2 q_k / (p_k + q_k))    (reverse)
kJsd      = 0.5 * (K(p||q) + K(q||p))         (= JSD by identity)
```

The clean property is `0 ≤ K(p||q) ≤ ln 2` and `0 ≤ K(q||p) ≤ ln 2`, with equality at zero iff `p ≡ q` on the grid. This follows directly from `2 p_k / (p_k + q_k) ∈ [0, 2]` and Gibbs' inequality applied to the per-summand log-ratio. Importantly, the *per-bin summand* `kDivSummand(p_k, q_k) = p_k log(2 p_k / (p_k + q_k))` is **signed** — negative whenever `p_k < q_k` — and only the sum over bins is non-negative. The axis exports `kDivSummand` and `kDivDirectionalSign` as pure helpers so callers can decompose the magnitude.

## 2. The KDE setup is bit-exact compatible with axes 126–139

A common failure mode when adding new divergence axes is changing the bandwidth or the grid silently, so that the new axis is not strictly comparable to its siblings. Axis-140 explicitly avoids this:

- Pooled robust scale: `mad_pool = 1.4826 * median(|x - median(x)|)`.
- Silverman bandwidth: `h = 0.9 * mad_pool * n^(-1/5)`.
- Shared K = 257-point grid spanning `[min - 3h, max + 3h]`.
- Gaussian KDE per half.
- Trapezoidal mass-normalisation to exact pmfs.
- `KDIV_PMF_FLOOR = 1e-15` underflow safeguard.

That is **the same KDE setup as axes 126 through 139**, which means a row's `kJsd` from axis-140 is bit-exact equal (modulo floating-point summation order) to that row's `jsd` from axis-118 if both are computed on the same first/second-half split. The only thing axis-140 adds is the directional split. Any difference between `axis-140.kJsd` and `axis-118.jsd` for the same `(source, day-window)` is a bug, and there are tests for it.

## 3. Bounded vs. unbounded: why this matters next to axis-139 Neyman

Axis-139 ships the Neyman χ² pair, which is *also* asymmetric:

```
N(p || q) = Σ (p_k - q_k)^2 / q_k
N(q || p) = Σ (p_k - q_k)^2 / p_k
```

But the Neyman pair is **unbounded** and **polynomial-rational** — it blows up exactly when one of the divisors approaches zero. In practice, that means a single low-mass tail bin in `q` (or in `p`, for the reverse direction) can dominate the entire score, and the regime classifier on axis-139 has to mask out near-zero divisors to avoid producing inf/NaN rows.

K-divergence halves does not have this problem. The `log(2 p_k / (p_k + q_k))` term is finite as long as `p_k + q_k > 0`, and the floor `KDIV_PMF_FLOOR = 1e-15` guarantees that. So axis-140 remains a robust drift signal **in the presence of tail bins that wreck unbounded divergences**. This is the single most important operational property: when your token series has a few very-low-density days mixed in (typical for low-volume sources like `vscode-other` or `hermes`), unbounded axes 134 and 139 will report inflated drift scores driven entirely by the tails. Axis-140 will report the true central-mass drift, plus the directional decomposition.

The flip side: K-divergence-halves is **less sensitive to tail-only drift** than Neyman by construction. You want both axes available, and the regime classifier on axis-140 (`kDivAsymmetryRegime`) is the diagnostic that tells you which one to trust on a given row.

## 4. Live smoke on the local pew queue, 2026-05-03

The CHANGELOG entry includes a live-smoke run of `pew-insights daily-token-k-divergence-halves --top 8` against the local pew queue (6 sources, 12.25B tokens, 1 dropped below `min-tenure-days = 14`). Reproduced verbatim:

| source | tenure | n1 | n2 | kFwd | kRev | kMax | asym | kJsd | tokens |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| openclaw | 17 | 8 | 9 | 2.603e-1 | 3.669e-1 | 3.669e-1 | 0.169919 | 3.136e-1 | 2,233,270,113 |
| opencode | 14 | 7 | 7 | 1.257e-1 | 1.753e-1 | 1.753e-1 | 0.164715 | 1.505e-1 | 6,269,354,373 |
| hermes | 17 | 8 | 9 | 3.501e-2 | 3.608e-2 | 3.608e-2 | 0.015090 | 3.555e-2 | 303,067,977 |
| claude-code | 72 | 36 | 36 | 9.789e-3 | 6.666e-3 | 9.789e-3 | 0.189816 | 8.227e-3 | 3,442,385,788 |
| vscode-other | 265 | 132 | 133 | 9.557e-4 | 6.917e-4 | 9.557e-4 | 0.160272 | 8.237e-4 | 1,885,727 |

There are five things worth reading off this table directly.

### 4.1 The `openclaw` row is the loudest signal

`openclaw` has `kRev = 0.367` against `ln 2 ≈ 0.693` — `kSaturation = 0.367 / 0.693 = 0.529`. That's more than half the maximum possible reverse divergence on a 17-day tenure with 8 vs 9 day halves. For context: `kMax > 0.30` puts a row in the "loud-drift" regime, `kMax > 0.50` is "saturating", and `kMax > 0.65` means the second-half distribution has essentially no overlap with the first-half distribution.

`openclaw` is REVERSE-dominated: `kRev > kFwd` (0.367 > 0.260, asymmetry 0.170). What does that mean operationally? Reverse-dominance means the second-half distribution puts mass on bins where the first-half distribution had little mass. Forward-dominance would mean the first-half had mass that the second-half abandoned. So a reverse-dominant row says: **this source is generating new behavior, not abandoning old behavior**. New code paths, new tools, new prompt sizes — something started showing up in the second half of the window that was not there in the first.

For an agent runtime, that is usually one of: (a) a new model deployment changed the per-call token shape, (b) a new tool call with different output sizes was added to the toolset, (c) a long-running workload kicked in. Any of those is worth knowing about.

### 4.2 `opencode` is also reverse-dominated, but less saturated

`opencode` shows `kRev = 0.175` (kSaturation 0.252), `kFwd = 0.126`, asymmetry 0.165. Same direction as openclaw — new mass in the second half — but at half the magnitude. `opencode` has only 14 days of tenure so the half-windows are 7 vs 7, which makes the KDE slightly less reliable than the 8/9 split for openclaw, but the direction is consistent.

### 4.3 `claude-code` is *forward*-dominated

This is the interesting one. `claude-code` has the longest tenure (72 days), the largest n1/n2 (36/36 split, plenty of data), and the **forward**-dominant signature: `kFwd = 9.79e-3 > kRev = 6.67e-3`, asymmetry 0.190. Forward-dominance on claude-code means the first-half had mass that the second-half is abandoning. Given the absolute magnitude is small (kMax < 0.01, kSaturation 0.014 — well within the noise floor), this is not a drift alert. But the *sign* is structurally informative: across the 72-day window, claude-code's per-day token distribution is consolidating, not expanding.

If you read this row alongside the openclaw and opencode rows, you can squint and see a thesis: **the openclaw and opencode runtimes are picking up workload that claude-code is shedding**. The mathematics of K-divergence-halves does not prove that thesis — the three rows could all be independent — but the directional signs are consistent with the cross-runtime migration story.

### 4.4 `hermes` is the symmetric drift-free baseline

`hermes` has asymmetry 0.015 — basically zero. Forward and reverse divergences are within 3% of each other. This is what a quiet, stable carrier looks like on this axis: small total kJsd (0.0356), almost no directional preference. Useful as a sanity check that the KDE setup is not introducing spurious asymmetry on quiet sources.

### 4.5 `vscode-other` is asymmetric but tiny

`vscode-other` shows asymmetry 0.160 (forward-dominated) but with a kMax of 9.56e-4 — `kSaturation = 0.00138`, three orders of magnitude below the alert threshold. The asymmetry sign is interpretable but the magnitude is not actionable. This is a recurring pattern with low-volume sources on a long tenure: the asymmetry calculation is a ratio so it stays meaningful, but the absolute magnitude is dominated by sample-size noise. Filter on `kMax > 0.05` or `kSaturation > 0.07` before acting.

## 5. The CLI sort surface and what to default to

The CLI exposes ten sort keys: `kMax` (default desc), `kMaxDesc`, `kForward`, `kForwardDesc`, `kReverse`, `kReverseDesc`, `kAsymmetry`, `kAsymmetryDesc`, `kJsd`, `kJsdDesc`, `tokens`, `tenure`, `source`. Defaulting to `kMax` desc is correct for a daily-drift dashboard — you want the loudest divergence in either direction at the top. But if you are hunting specifically for "what runtimes are growing into new behavior" (a fleet-onboarding question) you want `kReverse` desc. If you are hunting for "what runtimes are consolidating or losing diversity" (a deprecation-watch question) you want `kForward` desc.

The `kAsymmetry` sort is the diagnostic sort: it surfaces rows where forward and reverse are wildly different, regardless of magnitude. Most of the time you do not want this as the primary sort — a tiny-magnitude row with extreme asymmetry is just sample-size noise — but it is the right view when you suspect the regime classifier is mis-labelling rows.

## 6. Where this sits in the eventual axis-141..150 trajectory

Speculation, but informed by the post-`d5c8615` patch trajectory:

- The `kSaturation = kMax / ln 2` field added in `210006e` is the obvious bridge to a comparability axis across other bounded divergences (Hellinger has its own saturation against `√2`, total variation against `1`). Expect a unified `*_saturation` field across all bounded axes.
- The `kDivAsymmetryRegime` classifier from `b869859` is the obvious bridge to a multi-axis "drift regime" classifier that votes across axis-134, -139, and -140 to produce a single per-row regime label.
- The `kJsdSummand` per-bin primitive is the obvious bridge to a per-bin attribution axis ("which token-count bin contributed most to the kJsd score?"). That would be axis-141 territory.

The reason all three are obvious bridges is that axis-140 is the first axis to expose all three of (saturation, regime, per-bin) as first-class outputs rather than as derived quantities you have to compute downstream. Once an axis crosses that line, the next-axis surface is mostly about generalising those three concepts across the rest of the divergence ladder.

## 7. What this changes about how to read a `pew-insights` digest

If you previously used axis-118 JSD as your headline drift metric, replace it with axis-140 `kMax` — same ceiling, same KDE, but with the directional fields available the moment you need them. Use `kAsymmetry` as a diagnostic (not a primary sort), use `kSaturation` as the alert threshold (anything above 0.50 is loud, anything above 0.65 is essentially-non-overlapping), and use the forward/reverse split to distinguish "this source picked up new behavior" from "this source is losing diversity."

Three rows to look at every morning on a real fleet: the row with the highest `kMax`, the row with the highest `kReverse` regardless of `kMax`, and the row with the highest `kForward` regardless of `kMax`. The first tells you about magnitude, the second tells you about onboarding, the third tells you about deprecation.

That is more signal than axis-118 was giving up, for the same KDE work, in the same `pew-insights` invocation. The cost is two extra columns in the digest output and one extra concept (asymmetry) for the reader to internalise. That trade is worth making.

## References

- pew-insights 0.6.383 CHANGELOG, 2026-05-03 entry — `daily-token-k-divergence-halves` axis spec.
- Commits: `d5c8615` (axis introduction), `b869859` (regime + jsd-summand), `210006e` (kSaturation), `56a73b7` (smoke table).
- Cha, S.-H. (2007). *Comprehensive Survey on Distance/Similarity Measures Between Probability Density Functions*, eq. 36 (K-divergence definition).
- Lin, J. (1991). *Divergence Measures Based on the Shannon Entropy*, IEEE Trans. Inf. Theory 37(1).
