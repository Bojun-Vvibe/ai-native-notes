# Pew axis-162 (Durbin–Watson detrended-residual lag-1) as the residual-vs-level orthogonality witness, where `vsc-redacted` `dwZ = -2.3411` and `hermes` `rhoHatE = +0.3553` pin the detrended-residual persistence band

> **Date:** 2026-05-04
>
> **Source citations:** `pew-insights` v0.6.426 commits `a0cf3a9` (version + CHANGELOG) and `456890b` (axis-162 implementation). Live-smoke values quoted verbatim from the `0.6.426 — 2026-05-04` CHANGELOG entry against the local `~/.config/pew/queue.jsonl` snapshot at the time of the bump.

## Why this axis is interesting before you even read a number

The cross-source axis ledger inside `pew-insights` is now 162 entries deep. By construction every new axis has to clear an **orthogonality bar** against all of its predecessors — meaning: there must be at least one pair of synthetic time-series for which the new axis disagrees with every prior axis on the ranking of those two series. The bar gets harder to clear linearly in the count. Axis-162 — the **Durbin–Watson lag-1 residual-autocorrelation test on the OLS-detrended daily total-tokens series** — is interesting because it is the first axis that operates on **detrended residuals** rather than the raw level series, and it is the first axis whose null hypothesis is "white-noise residual structure around a fitted linear trend" rather than "stationary level" or "permutation-invariant marginal shape."

Concretely, axis-162 fits per-source

```
x_t = a + b * t + e_t      (closed-form OLS, t = 0..n-1)
```

and then reports the Durbin–Watson statistic on the residuals `e_t`:

```
DW   = sum_{t=1..n-1} (e_t - e_{t-1})^2 / sum e_t^2     in [0, 4]
rhoE = sum_{t=1..n-1} e_t * e_{t-1}    / sum e_t^2
dwZ  = (DW - 2) * sqrt(n) / 2                            approx N(0, 1)
```

with verdict cutoffs from the asymptotic `N(0, 1)` null:

```
positive-autocorr      dwZ <= -2.576       (p <= 0.005)
borderline-positive   -2.576 < dwZ <= -1.645 (p <= 0.05)
independent           -1.645 < dwZ <  +1.645
borderline-negative   +1.645 <= dwZ < +2.576
negative-autocorr      dwZ >= +2.576
```

A clean algebraic identity, verified by the unit tests in commit `456890b`, ties the two reported quantities together:

```
DW = 2 * (1 - rhoE) - (e_0^2 + e_{n-1}^2) / sum e_t^2
```

The boundary-correction term `(e_0^2 + e_{n-1}^2) / sum e_t^2` is what distinguishes DW from a naive `2 * (1 - rho)` reduction; it carries the contribution of the two endpoint residuals, which on short-tenure series can shift `dw` away from `2 * (1 - rho)` by a non-trivial amount. We will see this matter for `hermes` and `opencode` below.

## The live-smoke table at the bump

The CHANGELOG entry for `0.6.426` ran the new axis against the live local pew home and reported the per-source row, sorted by `dwZAbsDesc` (most-significant first):

```
source         tenure  slope          rhoHatE   dw       dwZ       verdict
-------------  ------  ------------   -------   ------   -------   -------------------
vsc-redacted   265     12.22          0.1437    1.7124   -2.3411   borderline-positive
claude-code    72      2920460.88     0.2022    1.5793   -1.7848   borderline-positive
hermes         18      81288.23       0.3553    1.1910   -1.7161   borderline-positive
openclaw       18      -11320736.76   0.1478    1.4573   -1.1513   independent
opencode       15      -9486929.76    -0.1583   1.6421   -0.6931   independent
```

There are five real-data observations packed into this five-row table. Each one is independently load-bearing for the orthogonality argument.

### Observation 1: `vsc-redacted` is the most autocorrelated source by `dwZ`, but the smallest by `slope`

`vsc-redacted` has a 265-day tenure — by far the longest in the table — and a microscopic upward slope of `+12.22 tokens/day`. The fitted line is essentially flat across the whole window. Yet the residuals around that flat line carry `rhoHatE = +0.1437` lag-1 autocorrelation, and because `n = 265` is large the test statistic scales as `sqrt(n)/2 = 8.14` and the resulting `dwZ = -2.3411` lands firmly in the `borderline-positive` zone (`p ≈ 0.019` under the asymptotic null).

Why this is interesting: a flat-trend source whose residuals are mildly persistent is exactly the regime where **raw-series serial-correlation axes would also fire** — there is no trend to subtract, so the residuals look like the raw series. But the `dwZ` magnitude here (`2.34`) sits at the boundary of significance. If we walked `vsc-redacted`'s slope up to even `+1000 tokens/day` (still tiny relative to its day-to-day variance) the raw-series Ljung–Box `Q` statistic would inflate spuriously while `dwZ` would stay essentially unchanged. That is the orthogonality of axis-162 in operation: it is a regression-diagnostic, not a level-test.

### Observation 2: `claude-code` shows that a strong trend can coexist with persistent residuals

`claude-code` has the second-largest `|dwZ|` at `1.7848` (`borderline-positive`, `p ≈ 0.037`), with `rhoHatE = +0.2022` over `n = 72` days. The fitted trend slope is enormous: `+2,920,460.88 tokens/day`. That is to say, the source is on a steep linear ramp, ramping by roughly +2.9 million tokens per calendar day across its 72-day tenure.

Naively you might expect that subtracting such a steep ramp would scrub all the dependence out of the residuals — the eye sees a monotone-up curve and assumes everything that wasn't trend was noise. The data say otherwise. After OLS subtracts the best-fitting line, the residuals retain a `+0.20` lag-1 autocorrelation. Translation: there are **multi-day streaks of above-trend or below-trend usage**, even after the ramp itself is removed. The token-burn pattern is not "trend + iid Gaussian noise"; it is "trend + AR(1)-like residual structure."

This is a real-data example of why the DW vs. KPSS / ADF orthogonality argument from the CHANGELOG matters. Axis-156 KPSS and axis-157 ADF would both classify `claude-code`'s level series as "non-stationary with unit-root-like behaviour" — which is correct but trivially explained by the +2.9M/day ramp. After detrending, the level non-stationarity vanishes by construction. What axis-162 then tells us is that the residual process around the ramp is **also** not iid. The two facts are independent, and only axis-162 surfaces the second one.

### Observation 3: `hermes` has the highest `rhoHatE` in the table — but barely clears the borderline because `n = 18`

`hermes` has `rhoHatE = +0.3553`, the largest residual lag-1 autocorrelation of any source in the table, by a comfortable margin (next-highest is `claude-code` at `+0.2022`). But `hermes` only has an 18-day tenure, so the test statistic scales as `sqrt(18)/2 = 2.12` and the resulting `dwZ` is `-1.7161` — barely inside the `borderline-positive` band (`p ≈ 0.043`).

This is a textbook short-tenure-vs-effect-size tradeoff. If `hermes` had `vsc-redacted`'s 265-day tenure with the same `rhoHatE = +0.3553`, its `dwZ` would be `+0.3553 * (-2) * sqrt(265)/2 = -5.78` (using the leading-order approximation `dwZ ≈ -rhoHatE * sqrt(n)`), which would be deeply into the `positive-autocorr` band (`p < 1e-8`). The information content of `hermes`'s row is therefore "the residual persistence is real and large, but the data window is too short to formally cross the conventional threshold by itself."

It also illustrates why the algebraic identity matters. Plugging the row's numbers into

```
DW = 2 * (1 - rhoE) - (e_0^2 + e_{n-1}^2) / sum e_t^2
```

we get `2 * (1 - 0.3553) = 1.2894`, and the reported `dw = 1.1910`, so the boundary-correction term `(e_0^2 + e_{n-1}^2) / sum e_t^2` evaluates to `1.2894 - 1.1910 = 0.0984`. On an 18-point series, the two endpoint residuals carry roughly 9.84% of the total residual sum-of-squares — a large boundary contribution, exactly as expected for short windows. On `vsc-redacted`'s 265-point series the same calculation gives `2 * (1 - 0.1437) = 1.7126`, vs. reported `dw = 1.7124`, so the boundary term is `0.0002`, four orders of magnitude smaller. The DW statistic gracefully degenerates toward `2 * (1 - rhoE)` as `n` grows — and the pew implementation reports the boundary-corrected version, not the asymptotic shortcut, which is why the test suite's identity-check is non-trivial.

### Observation 4: `openclaw` has a hard-down trend with iid residuals — a tractable forecasting regime

`openclaw` has `slope = -11,320,736.76 tokens/day` — falling by 11.3 million tokens per day across an 18-day window — and `rhoHatE = +0.1478`, basically the same as `vsc-redacted`'s residual autocorrelation but on `n = 18` rather than `n = 265`. Result: `dwZ = -1.1513`, comfortably inside the `independent` band (`p ≈ 0.249`).

This is the third quadrant of the JB × DW joint table that the CHANGELOG sketches: heavy-tailed marginal (axis-161 already flagged `openclaw` as not-Gaussian) but **independent residuals around the trend**. From a practical forecasting perspective this is the most tractable of all the regimes. You can fit a trend, you can model the residual marginal as a heavy-tailed iid distribution, and the conditional structure adds nothing — there is no autoregressive component to estimate. Whether the trend will *continue* at -11.3M/day is a separate question (answered by axis-156 / axis-157), but at least the residual process is well-behaved.

### Observation 5: `opencode` is the only source with negative residual autocorrelation

The most striking row in the table is the last one. `opencode` has `rhoHatE = -0.1583` — a **negative** lag-1 residual autocorrelation, the only such case in the table. Its `dwZ = -0.6931` is inside `independent` (`p ≈ 0.488`), so the magnitude is small enough that we cannot statistically reject "iid residuals around the trend." But the **sign** is qualitatively different from every other source.

Negative residual autocorrelation means: an above-trend day tends to be followed by a below-trend day, and vice versa — the residual process **oscillates** rather than persists. In token-burn terms: a heavy-usage day on `opencode` predicts a *lighter*-than-trend day next, even after we control for the steep -9.5M-tokens/day downward ramp. That is consistent with a "burst-and-recover" usage pattern: a burst pushes the residual up, then the recovery period pushes it below trend.

This is the kind of qualitative structural difference that almost every other axis in the 162-axis ledger would miss. Axis-156 (KPSS) and axis-157 (ADF) operate on the level series and would reject `opencode`'s level as non-stationary; axis-161 (Jarque-Bera) operates on the marginal shape and would say `opencode` is closer to Gaussian than `claude-code`; axis-160 (BDS) operates on m-history embeddings of the raw series and would conflate the trend with the dependence. Only axis-162 isolates the **detrended residual sign** as a per-source primitive.

## Compositional reading: the JB × DW joint table

The CHANGELOG closes with a 2×2 typology that is worth quoting verbatim because it is the cleanest statement of why axis-162 multiplies — rather than just adds to — the discriminative power of the ledger:

| profile                    | JB       | DW          |
|----------------------------|----------|-------------|
| iid Gaussian               | low      | approx 2    |
| iid heavy-tailed           | high     | approx 2    |
| Gaussian AR(1) residuals   | low      | far from 2  |
| heavy-tail AR(1) residuals | high     | far from 2  |

In the live-smoke data:

- `vsc-redacted` and `claude-code` both have `jbZ` deeply in the strongly-non-Gaussian zone (per axis-161's earlier live-smoke) **and** `dwZ < -1.6`. They sit in the fourth quadrant — heavy-tailed marginal **and** persistent residuals around their trend. This is the "everything is non-stationary about this source" regime: trend, marginal, and conditional dependence all moving.
- `openclaw` is in the third quadrant — heavy-tailed marginal but iid residuals. Tractable.
- `opencode` is closer to the second quadrant — near-Gaussian marginal with mildly oscillating residuals. Also tractable, but in a structurally different way.
- `hermes` would be in the fourth quadrant on effect size, but `n = 18` keeps it formally in the borderline.

Five sources, four distinct (JB, DW) regimes. That is the orthogonality bar being cleared in real data, not just in the CHANGELOG's adversarial counterexamples.

## Why the orthogonality argument is structural, not just empirical

The CHANGELOG's orthogonality argument has six clauses, one per prior axis-class. The most important one — and the one that motivated the choice of axis-162 over a naive "add another raw-series serial-correlation lag" — is the contrast against the existing **raw-series serial-correlation** axes (raw lag-1 / lag-7 autocorrelation, Spearman lag-1, Kendall lag-1, Bartels-rank von Neumann, axis-114 Ljung–Box, axis-159 McLeod–Li, axis-158 variance-ratio, axis-160 BDS).

The structural argument is: a monotone-trend series with iid Gaussian noise has `rhoHat_x ≈ 1` (by construction; the trend dominates the raw correlation) but `DW ≈ 2` (because the trend is exactly what gets subtracted). The two classes of axis are therefore not just empirically uncorrelated — they are **algebraically separated**: any sufficiently strong trend makes the raw-series axes saturate, while DW is invariant to the linear trend by construction. You cannot recover one from the other, even in the limit of infinite data.

The contrast against the **stationarity/unit-root/changepoint axes** (KPSS-156, ADF-157, CUSUM-153, Pettitt-154, Buishand-155) runs the other way. Those axes test the level trajectory — whether it has a unit root, whether it is stationary around its mean, whether there is a changepoint, whether the cumulative-deviation range is anomalous. Axis-162 tests something the level-trajectory axes are blind to: the **short-range conditional structure of the residuals after the trend is removed**. The level can be perfectly stationary around a fitted line and DW can still reject `independent`; the level can have a unit root and DW can still report `DW ≈ 2`. The two are independent dimensions.

The contrast against axis-161 (Jarque–Bera) is the cleanest. JB is permutation-invariant on the raw series — shuffle the daily totals and JB does not move. DW is time-ordered on detrended residuals — shuffle the residuals and DW will move toward 2 (because the iid permutation null is exactly `DW ≈ 2`). A heavy-tailed iid raw series has `JB >> 0` but `DW ≈ 2`; an AR(1)-residual linearly trending series has `JB ≈ 0` but `DW != 2`. The two axes are testing **disjoint properties of the joint distribution** of `(x_t)`.

## Test delta as evidence of behavioural completeness

The version bump from `0.6.425` → `0.6.426` brought the test count from `12,582` → `12,614` — a +32-test delta, all on the new file `dailytokendurbinwatsondetrended.test.ts`. The test list (per the CHANGELOG) covers: input validation (`n < 4`, non-finite, zero level), the algebraic identity `DW = 2 * (1 - rhoE) - (e_0^2 + e_{n-1}^2) / sum e_t^2`, slope/intercept correctness against scipy reference fits, the asymptotic-null `dwZ` formula, the verdict cutoffs at all four boundaries, the boundary-correction degeneration to `2 * (1 - rhoE)` as `n → ∞`, and the DW invariance under linear-trend addition (the single most important behavioural property of the axis).

That last test is the one that operationalizes the orthogonality argument from the CHANGELOG: it constructs a synthetic series, computes DW, then adds an arbitrary linear trend to every point and recomputes DW, and asserts the result is unchanged to within a tight floating-point tolerance. Without that test, the orthogonality claim against raw-series serial-correlation axes would be only an *intended* property; with it, it is a *verified* one.

## What this means for the ledger going forward

Axis-162 is the first axis that operates on detrended residuals. There is now an obvious natural extension surface — **axes that operate on residuals from richer trend models**: piecewise-linear (segmented), polynomial, low-pass-filtered, or HP-filtered. Each one would generate its own residual process and its own residual-serial-correlation diagnostic, and each one would clear its own orthogonality bar against axis-162 via the same trend-invariance argument applied to a richer trend class.

Whether that surface gets walked depends on whether the live-smoke continues to surface qualitatively-distinct per-source behaviour the way the (JB, DW) joint did this round. The five-row table at `0.6.426` already produced four distinct regimes across five sources. If the next axis — running on, say, the same residuals but at lag-7 instead of lag-1 — produces only two regimes, it has not earned its slot. If it produces a fifth, the ledger grows.

For now: axis-162 is in, the orthogonality bar is cleared in code (`456890b`) and in the live data (`0.6.426` CHANGELOG), the test delta of +32 is consistent with the per-axis-introduction floor seen across the last ~10 axes, and the version table moves to `v0.6.426`.

## Summary

- New axis: `daily-token-durbin-watson-detrended` (axis-162), shipped in commit `456890b`, version-bumped in `a0cf3a9`.
- Statistic: `DW`, `rhoHatE`, `dwZ` on OLS-detrended daily total-tokens residuals; `N(0,1)` asymptotic null; five verdict bands.
- Orthogonality cleared structurally (algebraic separability from the seven prior axis-classes) and empirically (four distinct regimes across five live sources).
- Live-smoke headlines: `vsc-redacted dwZ = -2.3411` (most significant), `claude-code dwZ = -1.7848` with `+2.9M tokens/day` trend, `hermes rhoHatE = +0.3553` (largest residual persistence, short-tenure suppressed), `openclaw` independent residuals around a -11.3M/day collapse, `opencode` the only negative-residual-autocorrelation source.
- Test delta `+32` on `dailytokendurbinwatsondetrended.test.ts`; algebraic identity and trend-invariance both unit-tested.
- Joint axis-161 × axis-162 typology produces a 2×2 (JB, DW) regime table that already shows four distinct quadrants populated in five real sources — concrete evidence that the new axis multiplies, rather than just adds to, the ledger's discriminative power.
