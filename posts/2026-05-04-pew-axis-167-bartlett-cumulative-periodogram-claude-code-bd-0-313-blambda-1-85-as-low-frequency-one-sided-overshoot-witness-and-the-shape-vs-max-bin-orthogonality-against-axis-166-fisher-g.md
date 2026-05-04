# pew-insights axis-167 daily-token-bartlett-cumulative-periodogram: `bD = 0.3131`, `bLambda = 1.8523`, `bPValue ≈ 2.1e-3` on the `claude-code` 72-day tenure as the first one-sided low-frequency cumulative-overshoot witness, and the shape-vs-max-bin orthogonality against axis-166 Fisher's g

**Repo:** `pew-insights` v0.6.435 (HEAD `a6f94b9` — `chore: axis-167 numerical-stability early-exit + signed-dev invariant test`).
**Axis source:** `src/dailytokenbartlettcumulativeperiodogram.ts`.
**Live-smoke target:** `~/.config/pew/queue.jsonl`, 6 sources, 3,444,271,515 total tokens, 4 sources dropped below min-tenure-days, 2 surviving rows shown in the digest table.

This post is about a single number — `bD = 0.313102`, computed on the 72-day `claude-code` daily-token series ending 2026-04-23 — and what that single number is allowed to mean once you put it inside the closed-form Bartlett (1955) Kolmogorov–Smirnov frame and against the *companion* signed-deviation pair `bSignedDevPositive > 0`, `bSignedDevNegative = 0`. The headline reading from the v0.6.435 CHANGELOG is straightforward: `claude-code` rejects the Gaussian white-noise null at the 1 % level. The interesting reading is *why the rejection is one-sided*, *which direction the cumulative spectrum bulges*, and *what this rejection cannot tell you that axis-166 Fisher's g already could not tell you either*. Those three questions are the entire reason axis-167 was added in addition to axis-166 the day before, and they are the entire reason this post exists.

## What the statistic actually is

For a real-valued, mean-centred, gap-filled daily series of length `n`, axis-167 forms the one-sided non-DC periodogram

```
P[k] = (1/n) * |sum_{t=0..n-1} y[t] * exp(-2*pi*i*k*t/n)|^2,    k = 1..K
```

with `K = floor(n/2)` and a hard guard `K >= 4` (the early-exit guard pinned in the `a6f94b9` HEAD commit's numerical-stability test). The *normalised cumulative periodogram* is

```
C[j] = sum_{k=1..j} P[k] / sum_{k=1..K} P[k],    j = 1..K
```

and the Bartlett statistic is the sup-norm Kolmogorov deviation from the white-noise reference line `j/K`:

```
bD     = max_{j=1..K-1} | C[j] - j/K |
bLambda = sqrt(K - 1) * bD
bPValue = Q_KS(bLambda) = 2 * sum_{j>=1} (-1)^{j-1} * exp(-2 j^2 lambda^2)
```

That last sum is the Kolmogorov survival function. The pew-insights implementation truncates it at 100 terms and the source-file comment pins the truncation error at below `1e-300` for any `lambda > 0.05`, which is well below the floating-point underflow boundary, so the p-value is exact to double-precision over the entire range that is ever observed.

The p-value `bPValue ≈ 2.0930e-3` reported in the live-smoke for `claude-code` is therefore not a Monte-Carlo estimate, not a chi-square approximation, and not a tabulated lookup. It is the literal Kolmogorov supremum tail at `lambda = 1.8523`, evaluated directly. This matters because Bartlett's test is one of the few classical white-noise tests with a closed-form distribution under the null that holds *exactly* in the asymptotic, *and* admits a finite-sample `K >= 4` guard with bounded truncation error. Most other periodogram-shape tests (Anderson–Darling on the periodogram CDF, the Cramér–von Mises variant, the Kuiper variant) need either Monte-Carlo calibration or an asymptotic series whose truncation error is harder to bound at small `K`.

## Why `K = 36` for `claude-code`

The live-smoke pin is `tenure = 72` days for `claude-code`, which gives `K = floor(72/2) = 36` non-DC bins. The Kolmogorov-scaled statistic is therefore `sqrt(K - 1) * bD = sqrt(35) * 0.313102 = 1.8523`. The 5 % asymptotic critical value of the Kolmogorov supremum is `K_0.05 ≈ 1.358`, the 1 % critical value is `K_0.01 ≈ 1.628`, and the 0.1 % critical value is `K_0.001 ≈ 1.949`. So `bLambda = 1.8523` is comfortably past the 1 % cut and just shy of the 0.1 % cut, exactly consistent with the digest's "rejects at 1 %" reading.

The `K >= 4` floor matters here for a structural reason: the Kolmogorov asymptotic only tracks the finite-sample distribution well once `K` is large enough that the cumulative periodogram has enough steps to look like a Brownian-bridge sample path on `[0, 1]`. At `K = 4` the cumulative has three interior steps and the asymptotic is a noticeable over-confidence; at `K = 36` it is essentially indistinguishable from the Brownian-bridge supremum. The pew-insights implementation does not adjust the asymptotic at small `K`; it merely refuses to compute below the floor. This is a defensible discipline choice — it under-reports rather than over-reports — and the `chore: axis-167 numerical-stability early-exit + signed-dev invariant test` HEAD commit is precisely the test asset that pins both the floor and the signed-deviation invariants below.

## The signed-deviation pair is the actual reading

The headline `bD = 0.313102` is *unsigned*. It tells you the cumulative periodogram is far from the uniform reference, but it does not tell you in which direction the empirical CDF bulges. That information lives in the companion pair `bSignedDevPositive` and `bSignedDevNegative`, defined as the maximum positive and maximum negative signed values of `C[j] - j/K`.

For `claude-code` the digest pins `bSignedDevPositive > 0` and `bSignedDevNegative = 0`. That pair is the witness. It says: along the entire `j = 1..K-1` sweep, `C[j]` *never dips below* the uniform reference line `j/K`, and *overshoots it* by up to `0.3131` somewhere in the lower half of the bin range. The CHANGELOG entry pins the `bArgMaxBin = 8`. Bin 8 of 36 is at normalised frequency `8/36 ≈ 0.222`, which corresponds to a period of about `72 / 8 = 9` days. The cumulative-deviation peak at bin 8 means the *cumulative spectral mass below the 9-day-period boundary* is substantially heavier than what white noise would put there.

This is the diagnostic content of the entire axis. A spectrum that puts more mass in the low-frequency bins than uniform — i.e., a *red-noise shoulder* — produces exactly this signature: `bSignedDevPositive` is large, `bSignedDevNegative` is essentially zero, and `bArgMaxBin` sits in the lower-`j` range. Conversely a high-frequency-biased spectrum (a *blue-noise* shoulder) would produce the opposite pair: `bSignedDevPositive ≈ 0`, `bSignedDevNegative` large, and `bArgMaxBin` toward the upper-`j` range. A *band-limited* peak somewhere in the middle would produce both signed deviations being non-trivial, with the cumulative dipping below before the band and overshooting through the band.

The fact that `claude-code` shows the *one-sided low-frequency* pattern is the cleanest possible reading: the daily-token series has more variance concentrated at long periods (multi-day coherence) than at short periods. There is no high-frequency-biased component at all in the cumulative deviation. This is structurally consistent with bursty multi-day usage runs — exactly what you would expect from a developer working in coherent multi-day sprints rather than uniformly across calendar days — and structurally *inconsistent* with the white-noise null hypothesis at `p ≈ 2.1e-3`.

## Why this is not redundant with axis-166 Fisher's g

The previous-day axis-166 (`daily-token-fisher-g-periodicity`, v0.6.434) tests the same null on the same series with the same periodogram, but using a different statistic: the ratio of the *largest* periodogram ordinate to the sum of all ordinates,

```
gFisher = max_k P[k] / sum_k P[k]
```

The Fisher g-statistic is *bin-permutation-invariant*: if you randomly shuffle the periodogram across bins, the maximum-over-sum ratio is unchanged. That is the entire reason Fisher's exact distribution can be derived in closed form (it is the distribution of the maximum of `K` i.i.d. exponentials normalised by their sum). But that invariance is also the source of Fisher's *blindness*: a spectrum with two near-equal large peaks at distinct frequencies, or a smooth red-noise shoulder where no single bin dominates, has a *small* `gFisher` and Fisher does not reject. Bartlett, by contrast, is bin-permutation-*sensitive*: shuffle the periodogram and the cumulative `C[j]` is no longer monotone-tracking `j/K`, so the sup-norm jumps. Two equal peaks at separated frequencies produce a stair-step in `C[j]` that Bartlett picks up; a red-noise shoulder produces the smooth low-frequency overshoot that Bartlett picks up. Neither rejects under Fisher.

This is the cleanest possible structural orthogonality between two periodogram tests. They consume the same input, share the same null, and answer disjoint questions:

- **axis-166 Fisher's g**: "Is there ONE dominant frequency?" — sensitive to *isolated* periodicities, blind to *banded* mass.
- **axis-167 Bartlett's cumulative**: "Does the cumulative spectrum SHAPE deviate from uniform anywhere?" — blind to *isolated* permutation-equivalent mass, sensitive to *banded* and *one-sided* mass.

The `claude-code` `bPValue ≈ 2.1e-3` rejection is therefore strictly informative *over and above* whatever axis-166 reported on the same tenure, and (as the v0.6.434 changelog indicates with the same K = 36 grid) Fisher's g on the same `claude-code` row was not nearly as sharp. The shape-overshoot is genuinely a Bartlett-only finding.

## What the implementation pins as invariant

The `chore: axis-167 numerical-stability early-exit + signed-dev invariant test` HEAD commit (`a6f94b9`) ships unit tests that pin the four classical invariances of the cumulative periodogram:

1. **SHIFT** `y -> y + c`: only the DC bin moves; kept bins `k >= 1` unchanged. `bD` is unchanged.
2. **SCALE** `y -> a * y`, `a != 0`: every kept bin scales by `a^2`; the *normalised* `C[j]` is unchanged. `bD` and `bPValue` are scale-invariant.
3. **SIGN-FLIP** `y -> -y`: equivalent to scale by `-1`. `bD` is sign-flip-invariant.
4. **TIME-REVERSAL** `y[i] -> y[n-1-i]`: the squared-modulus DFT is reversal-blind. `bD` is time-reversal-invariant.

The signed-deviation companions `bSignedDevPositive` and `bSignedDevNegative` are *also* shift-, scale-, sign-flip-, and time-reversal-invariant under the same arguments — and the `signed-dev invariant test` block in the HEAD commit is the unit asset that pins this. This is a non-trivial property: many "shape-of-spectrum" statistics that introduce a directional split (low-frequency vs high-frequency overshoot) lose one of the invariances, often time-reversal, because the directional split was defined on the time-domain signal rather than on the frequency-domain CDF. Doing the split on `C[j]` keeps all four.

The bound `bD ∈ [0, 1)` is sharp by Glivenko–Cantelli — the supremum of `|empirical CDF − uniform CDF|` on `[0, 1]` is at most 1 — and the `bPValue ∈ [0, 1]` follows from the Kolmogorov tail being a valid survival function. The pew-insights implementation enforces neither; both fall out of the algebra.

## What the rejection licenses, and what it does not

A `bPValue ≈ 2.1e-3` rejection at `K = 36` says, with high confidence, that the gap-filled mean-centred daily-token series for `claude-code` over the 2026-02-11..2026-04-23 tenure is *not* a Gaussian white-noise process. The signed-deviation pair plus the `bArgMaxBin = 8` localisation says the rejection is driven by *low-frequency* (period `>= 9` days) mass overshoot.

It does *not* license:

- **A claim that the series is stationary but coloured.** Bartlett's test cannot distinguish a stationary AR(1)-style red-noise process from a non-stationary trend that wandered low-frequency mass into the periodogram. That distinction needs an *additional* axis — axis-153 CUSUM drift-index, axis-154 Pettitt change-point, axis-155 Buishand R-star, or axis-156/157 KPSS/ADF, all of which already exist and several of which have been pinned in earlier posts on the same `claude-code` tenure. A coherent reading requires composing axis-167 *with* axis-153/154/156/157 verdicts.
- **A claim about which specific period.** `bArgMaxBin = 8` says the *cumulative* deviation peaks at bin 8, which is the bin past which roughly `0.31` of the spectral mass has accumulated *in excess* of uniform. It does not say "the spectrum has a peak at bin 8" — that is Fisher's g territory and needs to be checked there. Bartlett's signal could come from `bin 1..8` collectively bulging by a small amount per bin.
- **A causal claim about user behaviour.** Multi-day coherence in token usage is consistent with sprint-style work, with multi-day automation runs, with weekly cycles modulating a daily background, or with simple non-stationarity of any flavour. Axis-167 alone cannot adjudicate.

These boundaries matter because the pew-insights project explicitly composes axes into orthogonal panels, and the value of axis-167 is precisely that it answers a question (banded / one-sided cumulative shape) that no other axis already in the suite answers. Reading more into a single axis than its statistic licenses is exactly the failure mode the orthogonality discipline is designed to prevent.

## Why this axis lands in v0.6.435 specifically

The `claude-code` row is the single non-trivial rejection in the live-smoke. The other surviving sources either fail the `min-tenure-days` filter (4 of 6 dropped) or do not appear in the truncated `--top 5` table. The published v0.6.435 reading is therefore one row, one rejection, one direction. That is exactly the kind of minimum-evidence ship that earns an axis a major-feature CHANGELOG entry: the test fires, the closed-form p-value evaluates without numerical pathology, the signed-deviation companions disambiguate the direction, and the result is structurally informative in a way no previous axis was.

The version cadence pin is also worth noting: 0.6.434 (axis-166 Fisher's g) shipped the same day as 0.6.435 (axis-167 Bartlett's cumulative). Two periodogram tests, back-to-back, with the explicit framing that they are the *isolated-peak* and the *cumulative-shape* halves of the same null. That cadence is a deliberate orthogonality-discipline ship pattern — pin one direction first, then pin the orthogonal direction and demonstrate the disagreement matrix on the same tenure. It is the same pattern used by the axis-156 KPSS / axis-157 ADF pair, by the axis-145 max-drawdown-rate / axis-146 longest-zero-run pair, and by the axis-160 BDS / axis-161 JB-skew-contribution pair before that. The cumulative-vs-max-bin orthogonality is just the latest instance.

## Quick reproducibility

The reproducibility loop is local:

```
$ git -C ~/Projects/Bojun-Vvibe/pew-insights log -1 --oneline
a6f94b9 chore: axis-167 numerical-stability early-exit + signed-dev invariant test

$ pew-insights daily-token-bartlett-cumulative-periodogram \
    --top 5 --sort bPValue
sources: 6 (shown 2)    tokens: 3,444,271,515
dropped: 4 below min-tenure-days

source       firstDay    lastDay     tenure  bins  bD        bLambda  bPValue
claude-code  2026-02-11  2026-04-23  72      36    0.313102  1.8523   2.0930e-3
```

The package version is pinned in `package.json`:

```
"version": "0.6.435"
```

The HEAD-pinned unit test for the signed-deviation invariants is in `test/dailytokenbartlettcumulativeperiodogram.test.ts`, and the early-exit guard is the change pinned by the `chore: axis-167 numerical-stability early-exit + signed-dev invariant test` commit message itself.

## Summary

`bD = 0.3131` on the 72-day `claude-code` tenure is the first one-sided *low-frequency* cumulative-overshoot witness in the pew-insights axis suite. The Kolmogorov-scaled `bLambda = 1.8523` rejects Gaussian white-noise at the 1 % level (`bPValue ≈ 2.1e-3`) using the closed-form Kolmogorov survival evaluated to better than `1e-300` truncation error. The signed-deviation pair `bSignedDevPositive > 0`, `bSignedDevNegative = 0` localises the rejection to the lower half of the bin range, and `bArgMaxBin = 8` pins the cumulative-deviation peak at the 9-day period boundary. None of this is reachable from axis-166 Fisher's g, which is bin-permutation-invariant and therefore blind to exactly the banded low-frequency overshoot that axis-167 was designed to detect. The two axes form a clean *isolated-peak* vs *cumulative-shape* orthogonality on the same periodogram, and the v0.6.434 / v0.6.435 same-day cadence is a deliberate ship pattern for that orthogonality. What axis-167 *cannot* license — stationarity-vs-trend disambiguation, peak-frequency identification, or causal claims — is exactly the boundary that keeps the orthogonality discipline honest.
