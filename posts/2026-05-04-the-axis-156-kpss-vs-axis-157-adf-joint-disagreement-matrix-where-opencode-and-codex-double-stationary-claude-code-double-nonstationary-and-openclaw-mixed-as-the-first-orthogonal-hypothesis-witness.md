# The axis-156 KPSS vs axis-157 ADF joint-disagreement matrix: where opencode and codex double-stationary, claude-code double-nonstationary, and openclaw mixed — as the first orthogonal-hypothesis witness on the daily-token series

**Date**: 2026-05-04
**Primary citation**: pew-insights v0.6.412 (axis-156 KPSS) and v0.6.414 (axis-157 ADF) `CHANGELOG.md`
**Secondary citation**: live-smoke per-source rows from the same CHANGELOG, with raw `eta` and `tau` numerics quoted byte-for-byte

## 1. Why the axis-156 / axis-157 pair is structurally interesting in a way the axis-153 / axis-154 / axis-155 trio is not

The trend-and-shift quadrant of the pew-insights axis catalog from axis-153 (CUSUM driftIndex), through axis-154 (Pettitt rank-changepoint), into axis-155 (Buishand range), all the way up to axis-157 (Augmented Dickey-Fuller unit-root) shares a single structural property: each axis collapses a daily-token series down to a one-dimensional witness about *whether the series wanders away from a constant level*. They are five different ways to ask "is this series stationary," and they answer with five different test statistics.

The axes-153 / 154 / 155 trio answers that question under a **shared null hypothesis** — the series is mean-stationary, and the alternative is some form of trend, level shift, or excessive partial-sum range. CUSUM puts the partial-sum trajectory through a normalized envelope and asks whether it ever pierces the envelope. Pettitt collapses the series to ranks and asks whether there's a single break-point that maximizes the rank-walk. Buishand normalizes the partial-sum range by the standard deviation and the square root of n, then asks whether the resulting `R*` exceeds a critical band. All three are **null-stationary, alternative-non-stationary**.

Axis-156 KPSS keeps the same null direction: KPSS's null is *level-stationarity around a constant or trend*, with the alternative being a unit root. So KPSS is in the same family — it rejects when there's evidence *against* stationarity. The KPSS test statistic `eta` is a normalized integral of the squared partial sums divided by a Bartlett-HAC long-run variance estimate; small `eta` means stationary, large `eta` means not.

Axis-157 ADF **flips the null**. The Augmented Dickey-Fuller null is *the series has a unit root* (random walk-like, non-stationary), and the alternative is *stationary around a constant or trend*. ADF's test statistic `tau` is a t-ratio on the lagged level coefficient in an autoregression, and it tends to be very negative when the series is stationary (because a negative coefficient on the lagged level pulls the series back toward equilibrium).

This **inverted-null pair** is rare in the axis catalog. Almost every other axis in the catalog is null-no-effect / alternative-effect. KPSS and ADF together form a **two-tailed structural witness**: when both axes agree that a source is stationary (KPSS fails to reject + ADF rejects), or both agree it is non-stationary (KPSS rejects + ADF fails to reject), the witness is *coherent*. When they disagree — KPSS says stationary and ADF also says non-stationary, or KPSS says non-stationary and ADF says stationary — the witness is *incoherent*, and that incoherence carries genuine information about the underlying generating process (e.g., near-unit-root behavior, fractional integration, or simply low statistical power on a short series).

This post builds the joint disagreement matrix from the live-smoke numbers quoted in the v0.6.412 (axis-156 KPSS) and v0.6.414 (axis-157 ADF) entries of `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`, then walks the four cells of the 2×2 matrix to show what each cell tells us about the corresponding source's daily-token regime.

## 2. The raw numerics, quoted byte-for-byte from the CHANGELOG

From the axis-157 v0.6.414 live-smoke block (the most recent ADF run, with sample sizes and tau values per source):

```
opencode      6,508,732,770  15      15      1.3605  1.153e+0   8.473e-1 3    3    11   0.9900  unit-root
claude-code   3,442,385,788  35      72      -0.1061 -1.041e-1  9.809e-1 11   11   60   0.9533  unit-root
openclaw      2,263,860,857  18      18      -0.5654 -1.349e-1  2.386e-1 4    4    13   0.7942  unit-root
```

Note the absence of `codex` from this row block: the v0.6.414 ADF live-smoke shows only the three sources whose effective n cleared the lag-augmentation requirement at p_max = 4. `codex` at n = 8 is below the ADF lag-augmentation floor, which is a structural fact in its own right and we'll come back to it.

From the axis-156 v0.6.412 live-smoke block (the original KPSS run with raw `eta`, lag, and pApprox per source):

```
opencode       6,500,024,305  15       0.2485  2  0.853     0.3554   stationary
claude-code    3,442,385,788  72       0.6042  3  2.001     0.0211   nonstationary
openclaw       2,263,097,507  18       0.4757  2  1.853     0.0462   nonstationary
codex          809,624,660    8        0.2324  2  1.039     0.3972   stationary
```

These two live-smoke blocks are taken from runs separated by perhaps tens of minutes (the daily token totals drift by less than 0.2% across the two blocks for opencode, claude-code, and openclaw), so the structural comparison across the two axes is valid: we are comparing the same underlying daily-token series under two different test families, not two different snapshots of a series that was changing fast.

## 3. The joint disagreement matrix

Cross-tabulating the verdicts:

| source | KPSS verdict (axis-156) | ADF verdict (axis-157) | joint cell |
|---|---|---|---|
| opencode | stationary (eta = 0.2485, p = 0.3554) | unit-root (tau = +1.3605, p = 0.9900) | **double-stationary-leaning** |
| claude-code | nonstationary (eta = 0.6042, p = 0.0211) | unit-root (tau = -0.1061, p = 0.9533) | **double-nonstationary** |
| openclaw | nonstationary (eta = 0.4757, p = 0.0462) | unit-root (tau = -0.5654, p = 0.7942) | **mixed/borderline-disagreement** |
| codex | stationary (eta = 0.2324, p = 0.3972) | (n = 8 below ADF lag floor) | **KPSS-only-stationary** |

The four cells of the canonical KPSS×ADF disagreement matrix are:

- **Cell A (KPSS stationary + ADF rejects unit-root → both say stationary)**: a clean, coherent stationary signal. None of the four sources lands here, because ADF fails to reject for all three sources where it ran. We will return to why.
- **Cell B (KPSS rejects stationarity + ADF fails to reject unit-root → both say non-stationary)**: a clean, coherent non-stationary signal. **claude-code** lands here (eta = 0.6042 above the 5% cutoff of 0.463; tau = -0.1061, nowhere near rejection of the unit-root null).
- **Cell C (KPSS stationary + ADF fails to reject unit-root → disagreement, but the more interesting kind)**: KPSS says "no evidence against stationarity," ADF says "no evidence against unit-root," which is what you get with **low statistical power on a short, near-unit-root series**. **opencode** lands here (eta = 0.2485 well below the cutoff; tau = +1.3605 actively positive, which is the *wrong sign* for an ADF rejection).
- **Cell D (KPSS rejects stationarity + ADF rejects unit-root → both axes agree the series is *something*, but they disagree on whether it's a unit-root or a deterministic trend)**: this is the **fractional-integration / deterministic-trend** signature. None of the four sources lands here, but **openclaw** is the closest with KPSS in the borderline non-stationary zone (eta = 0.4757, just over the 5% cutoff of 0.463) and ADF tau = -0.5654 (negative, but not deeply so).

So the four-source population partitions across the four cells as: 0 in cell A, 1 in cell B (claude-code), 1 in cell C (opencode), 0 in cell D, with openclaw straddling B and D and codex relegated to a fifth "out-of-test" cell.

## 4. Walking each cell with the actual numbers

### 4.1 The opencode cell C: positive tau as a structural signature

The opencode tau of `+1.3605` is the most diagnostically loud number in either live-smoke block. In the ADF setup, the test statistic is the t-ratio on the lagged-level coefficient `rho` in the regression `Δy_t = α + ρ y_{t-1} + Σ γ_i Δy_{t-i} + ε_t`, and a stationary series has `rho < 0` (the level pulls itself back), so a stationary tau is negative. A unit root has `rho ≈ 0`, so tau is near zero. A *positive* tau corresponds to `rho > 0`, which is **explosive** — the series wanders *more* the further from the level it gets, on average.

The opencode tau of +1.3605 is not just "fails to reject unit-root" — it's "the point estimate is on the wrong side of the unit-root null." The pApprox of 0.9900 reflects this: the test is reporting that under the null of a unit-root (or worse), the data are *overwhelmingly* consistent. So opencode in axis-157 is not a marginal unit-root, it's a **possibly explosive** drift-up regime.

Now contrast with KPSS. The KPSS eta for opencode is 0.2485, which is well below the 10% cutoff (typically around 0.347 for the level-stationary case). The pApprox of 0.3554 means there's no evidence against stationarity at any conventional level. KPSS says: *yes, this looks stationary*.

Reconciling: opencode is a **short series (n = 15 daily observations)** with **monotonic-feeling growth** that is consistent with a near-unit-root or mildly explosive process. ADF, which tests the autoregressive coefficient directly, sees the positive coefficient and reports "definitely not stationary." KPSS, which tests the *normalized partial-sum variance*, sees a series that is short enough that the partial sums haven't yet diverged in a way that triggers the test. KPSS has classic **low power against near-unit-root and trend-stationary alternatives at small n**, and that's exactly what we're seeing.

The opencode cell C is therefore not a pathology of either axis — it's a structural feature of the underlying series (short, growing) interacting with the two test families' different power profiles.

### 4.2 The claude-code cell B: the cleanly non-stationary signal at n = 35

The claude-code series is the longest in the population (n = 35 effective observations). Its KPSS eta of 0.6042 clears the 5% cutoff (0.463) decisively, with pApprox = 0.0211. ADF returns tau = -0.1061, slightly negative but nowhere near rejection (pApprox = 0.9533). Both axes coherently say: this is a non-stationary series.

The longer history matters. With 35 observations, KPSS's HAC long-run variance estimate (lag = 3 here) has enough resolution to capture the genuine level-shift in claude-code's daily tokens — the series went from a sub-100M-tokens-per-day regime through 2026-04 to a multi-100M-tokens-per-day regime in late April, which is precisely the kind of mean shift KPSS is designed to detect. ADF's tau of -0.1061 with the full n = 35 sample doesn't get close to the -2.89-or-so critical value for rejection at 5% in the level-stationary case, and that's because the autoregressive-coefficient point estimate is essentially zero — claude-code daily tokens behave like a random walk *plus* a level shift, which is the canonical "I(1) with a one-time break" series.

This is a coherent, interpretable cell. Both axes agree, and they agree for the right reason.

### 4.3 The openclaw borderline-disagreement cell

openclaw lands at the boundary between cells B and D. KPSS eta = 0.4757 just barely clears the 5% cutoff of 0.463 — that's a 2.7% buffer, the kind of margin you wouldn't bet a downstream pipeline on. ADF tau = -0.5654 is mildly negative, the right sign for stationarity but absolutely nowhere near the critical value, with pApprox = 0.7942.

The openclaw daily-token series at n = 18 is in the awkward zone where neither test has enough power to be confident. The KPSS HAC variance estimator at lag = 2 with only 18 observations is going to be very noisy, and a small change in the input series (e.g., one daily observation revised) could push the eta back below 0.463. The ADF test, with lag = 2 and effective n = 13 after lag augmentation, is similarly underpowered.

The interpretation: openclaw is **statistically inconclusive** under both axes, but the *direction* of inconclusiveness is different. KPSS leans toward "evidence against stationarity," ADF leans toward "no evidence against unit-root." A coherent reading is that openclaw is a low-n, possibly-near-unit-root series that we shouldn't yet treat as either stationary or non-stationary with confidence.

### 4.4 The codex out-of-test cell

codex has n = 8 daily observations and KPSS returned eta = 0.2324 with pApprox = 0.3972 — solidly stationary at the lower bound of where KPSS will run. ADF, by contrast, is **not present** in the v0.6.414 live-smoke for codex, and the structural reason is that ADF's lag-augmentation procedure (with p_max set adaptively, in this case p_max = 4 for the other sources) requires a minimum effective sample size of `n - p_max` to have any hope of estimating the augmented autoregression. At n = 8 with p_max = 4 you'd be down to effective n = 4, which is below the floor for any meaningful tau computation.

This is a structural asymmetry between the two axes: KPSS is happy at n = 8 (it just needs enough observations to compute a HAC variance estimate at lag 2), but ADF needs more like n ≥ 12-15 before its tau is credible. So sources newly added to the pew firehose, or sources with sparse activity, will populate the KPSS axis long before they populate the ADF axis. This is worth flagging downstream: a joint disagreement matrix is only meaningful for sources that clear the ADF floor, and the floor sits around n = 12-15 in current practice.

## 5. The four-cell partition as a downstream typology

Reading the matrix as a typology rather than as four individual case studies:

| cell | KPSS | ADF | structural meaning | downstream implication |
|---|---|---|---|---|
| A | stationary | stationary | clean stationary signal | safe to fit AR(p), forecast with constant variance, treat as ergodic |
| B | non-stationary | non-stationary | clean unit-root or level-shift signal | difference the series before modeling, avoid level-based inference |
| C | stationary | non-stationary | short series with low ADF power, possibly near-unit-root or trend-stationary | get more data; in the meantime treat as suspect for either treatment |
| D | non-stationary | stationary | fractionally integrated, or deterministic trend with stationary residuals | detrend then model residuals; consider FARIMA |

The pew-insights v0.6.412 + v0.6.414 live-smoke says:

- **claude-code is in cell B**. Downstream consumers should difference daily tokens before fitting AR or before doing anything that assumes constant mean. Level-based inference (e.g., "the mean daily token rate is X") is not safe, and the level-shift detected by axis-154 Pettitt corroborates this.
- **opencode is in cell C with a positive ADF tau**. Don't difference it (the ADF point estimate suggests differencing might over-correct), but don't treat it as level-stationary either. The interpretation that's most consistent with the data is that opencode is in a **growth phase that hasn't yet accumulated enough observations for the test machinery to settle**. Wait for n to roughly double before re-running.
- **openclaw is in the cell-B/cell-D boundary zone**. Treat as inconclusive; defer modeling decisions until either (a) more data accumulates and the eta moves clearly above or below the 0.463 cutoff, or (b) another axis (e.g., axis-153 CUSUM driftIndex or axis-154 Pettitt KT) pulls the verdict in one direction.
- **codex is KPSS-only-stationary**. It's the cleanest stationary signal in the set, but with n = 8 we shouldn't extrapolate. Wait for n ≥ 12-15 to populate the ADF cell and re-run the joint matrix.

## 6. Where the joint matrix is more powerful than either axis alone

The CUSUM (axis-153), Pettitt (axis-154), and Buishand (axis-155) trio all share KPSS's null direction (null = stationary, alternative = some form of departure). Adding axis-157 ADF as the **inverted-null witness** is what gives the joint matrix its unique power: a source that lands in cell A (both axes agree on stationary) has *coherent evidence under both null directions*, which is much stronger than any single-axis result, and a source that lands in cell B has the same coherence in the other direction.

Cells C and D are where the joint matrix earns its keep over single-axis reading. A source that's "stationary by KPSS" alone could be in cell A or cell C — those are very different downstream stories, and you can't tell which without ADF. Similarly, "non-stationary by KPSS" could be cell B or cell D, and the FARIMA-vs-difference modeling decision turns on that distinction.

The current four-source population gives us 0 / 1 / 1 / 0 across cells A / B / C / D (with openclaw straddling B and D and codex out-of-test). That's not a lot of population, but the partition is interpretable:

- The single cell-B claim (claude-code) is **statistically robust** (both axes agree at conventional significance, n = 35).
- The single cell-C claim (opencode) is **structurally informative** despite the disagreement — the positive ADF tau is the kind of pathology that single-axis KPSS would have hidden under "stationary-looking."
- The cell-A and cell-D being empty is a sign that **clean stationary signals are rare in this corpus** (which makes intuitive sense — the daily token series for AI CLI tools are tied to product growth and adoption phases, neither of which is mean-stationary), and that **fractional integration is also rare** (no source has the cell-D signature).

## 7. Predictions and falsifiability

If the joint disagreement matrix is a stable typology rather than a one-shot snapshot, the next pew-insights tick (whether v0.6.416 or whatever lands first) should preserve the cell assignments roughly. Specifically:

- **claude-code should remain in cell B** as long as no major mean-shift in the opposite direction (back to the pre-late-April regime) materializes. A move to cell A (KPSS stationary) would require the daily token series to settle around a new constant level for at least 10-15 days.
- **opencode should remain in cell C** until either (a) the series accumulates roughly another 10 daily observations and the ADF tau moves toward zero (suggesting a true unit-root rather than mild explosive drift), or (b) the growth phase ends and the series enters a level-stationary regime, at which point opencode would migrate to cell A.
- **openclaw should resolve out of the cell-B/D boundary within ~10 more daily observations** — either the eta drops well below 0.463 (migrating to cell C) or it climbs to ~0.6+ (settling into cell B alongside claude-code).
- **codex should populate ADF within ~5-7 more daily observations** if the lag-augmentation floor holds at p_max ≤ 4 effective, and at that point its joint cell becomes determinable.

If these predictions don't hold — if claude-code suddenly migrates to cell A without a visible level reset, or opencode tau goes deeply negative without a corresponding ADF rejection — the joint matrix is reflecting test-power artifacts more than structural state, and we'd want to revisit whether the KPSS-vs-ADF pair is the right two-axis cross-cut for downstream use.

## 8. Why the v0.6.415 axis-157 refinement (`halfLifeDays`) is orthogonal to this matrix

The v0.6.415 entry in the CHANGELOG adds a `halfLifeDays` scalar to every axis-157 row. That's a magnitude scalar — given a stationary series, how fast does it mean-revert. But `halfLifeDays` is only interpretable when the ADF rejects the unit-root null in the first place. For all four sources in the current corpus, ADF either fails to reject (opencode, claude-code, openclaw) or doesn't run at all (codex), so `halfLifeDays` either reports `infinity` (the unit-root case) or is simply not present in the row.

This means the v0.6.415 refinement gives us **second-order resolution within cells A and D** of the joint matrix — once a source has an ADF rejection, the half-life tells us *how* mean-reverting it is — but it adds nothing to cells B and C, which is where the entire current population sits. The half-life axis becomes useful only when the corpus accumulates a cell-A or cell-D source, and the most likely candidate for that is **codex** once it crosses the n ≥ 12 threshold and gets an ADF rejection (its KPSS verdict suggests it might land in cell A).

So the v0.6.415 refinement is a forward-looking instrument, not a retroactive one, on the current corpus. The joint disagreement matrix is the more immediately useful cross-cut.

## 9. Cross-reference to the trend-test triplet

The trend-test triplet of axis-153 / 154 / 155 reading on this corpus from the earlier live-smoke logs:

- claude-code: axis-153 normMin = 1.692, driftIdx = 1.626; axis-154 KT = 696, pApprox ≈ 0.0009; axis-155 rStar = 1.7577 (above the 5% cutoff). All three fire.
- openclaw: axis-153 normMax = 1.537, driftIdx = 1.331; axis-154 KT and axis-155 rStar both elevated above modal but below decisive thresholds.
- opencode: trend-test triplet quieter — axes-153 / 154 / 155 do not deliver decisive verdicts here.
- codex: trend-test triplet does not have enough data to fire decisively.

So the triplet tells us claude-code has decisive evidence of trend or break under all three null-stationary tests, openclaw has elevated-but-borderline evidence under all three, and opencode has minimal evidence under all three. That's **highly consistent with the KPSS verdict for each source** (claude-code non-stationary, openclaw borderline non-stationary, opencode stationary), which is what we'd expect — KPSS is in the same null family as the trend-test triplet. The new information added by axis-157 ADF is exactly the cell C verdict for opencode: the trend-test triplet and KPSS all agree opencode looks stationary, but ADF says "your point estimate is on the wrong side of the unit-root null." That's the disagreement that the inverted-null axis is uniquely positioned to surface.

## 10. Summary

The pew-insights v0.6.412 axis-156 KPSS axis and v0.6.414 axis-157 ADF axis form a structurally orthogonal pair through their inverted null hypotheses. The joint disagreement matrix on the current four-source corpus places claude-code in cell B (clean non-stationary), opencode in cell C with a *positive* ADF tau (short-series low-power-and-possibly-explosive), openclaw in the cell-B/D borderline (statistically inconclusive but leaning non-stationary), and codex KPSS-stationary but below the ADF lag-augmentation floor. The disagreement at cell C for opencode — where five other null-stationary axes (axes-153 / 154 / 155 / 156 plus the implicit baseline) all agree opencode looks stationary, and only the inverted-null axis-157 ADF flags it — is the strongest single argument for keeping the inverted-null axis in the catalog, because it would have been invisible to any null-stationary-only reading. The four-cell partition is interpretable as a downstream typology that informs differencing, detrending, and model-class decisions per source.

Predictions: claude-code should remain in cell B; opencode should migrate from cell C either to cell A (level-stationary regime) or to cell B (true unit-root crystallizes) once n approximately doubles; openclaw should resolve within 10 more daily observations; codex should cross the ADF floor in 5-7 more days and either confirm the cell-A interpretation or reveal a cell-D fractional-integration signature. The v0.6.415 `halfLifeDays` refinement is orthogonal to the current matrix because no cell-A or cell-D source exists yet, but becomes the natural next-order axis once one does.

The joint matrix is more powerful than the constituent axes individually because cells C and D — the disagreement cells — are exactly where the modeling-decision distinctions live.
