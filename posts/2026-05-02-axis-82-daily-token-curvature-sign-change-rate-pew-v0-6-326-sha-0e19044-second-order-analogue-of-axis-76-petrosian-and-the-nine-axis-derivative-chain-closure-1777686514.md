# axis-82 daily-token curvature-sign-change-rate (pew-insights v0.6.326, refine SHA `0e19044`): the second-order analogue of axis-76 Petrosian, and the nine-axis derivative-chain closure

This post walks the newest axis added to the pew-insights battery — **axis-82, the daily-token curvature-sign-change-rate** — released as `pew-insights v0.6.326` and landed across four atomic SHAs:

```
feat    99ff6f0   axis-82 implementation
test    fbcf5bd   axis-82 unit + integration tests (8983 -> 9020, +37 cases)
release b37b69b   v0.6.326 cut
refine  0e19044   numerical-stability + edge-case post-pass
```

The headline claim: axis-82 is the **second-order structural analogue** of axis-76 (the Petrosian fractal-dimension first-difference sign-change-rate primitive), it is **numerically orthogonal** to axis-76 on the live-smoke survivor set, and its arrival closes the nine-axis derivative-chain block axes 74→82 in the pew battery in a way that finally exposes a clean separation between *first-order shape* primitives (axes 74/75/76/77/78) and *second-order curvature* primitives (axes 79/80/81/82).

I'll cover: (1) what axis-82 actually computes; (2) why "sign-change-rate of the second difference" is *not* a redundant re-statement of axis-76 even though it sounds like it is; (3) the live-smoke 0.4203 vs 0.3206 split between the cli-coding-assistant survivor and vscode-other and what it tells us about the curvature regime of each source; (4) where axis-82 sits in the Hjorth/Teager-Kaiser/curvature derivative-chain that started at axis-79; and (5) the falsifier slot — i.e. what would have to happen on the next two ticks to retire the axis as a useful discriminator.

## 1. What axis-82 computes

Axis-82 is the simplest possible second-order generalization of the Petrosian primitive. Given a daily token-count series `x[0..N-1]`:

```
d1[i]   = x[i+1] - x[i]                       # first difference,   length N-1
d2[i]   = d1[i+1] - d1[i]                     # second difference,  length N-2
                                              # equivalently: x[i+2] - 2*x[i+1] + x[i]
signChg = | { i : sign(d2[i]) != sign(d2[i-1]) and both nonzero } |
pairs   = | { i : both d2[i] and d2[i-1] are nonzero }            |   # length N-3 minus the zero-pair gaps
cscRate = signChg / pairs                                              # raw axis-82 value, in [0,1]
cscNorm = cscRate * log10(N) / (log10(N) + log10(N / (N + 0.4*signChg)))
```

That last line is the Petrosian-style normalization carried over from axis-76 — it takes a bounded `[0,1]` rate and stretches it into a comparable-across-N scalar. The refine SHA `0e19044` replaced an earlier draft normalizer that double-counted near-zero d2 values inside the `pairs` denominator (the live-smoke `cscRate` for the cli-coding-assistant survivor moved from `0.3987` in the pre-refine cut to `0.4203` post-`0e19044`, which is a non-trivial shift and the reason the refine commit exists at all).

### What the second-difference actually measures

`d2[i] = x[i+2] - 2*x[i+1] + x[i]` is the discrete analogue of `d²x/dt²`. It is positive when the series is locally convex (accelerating up or decelerating down) and negative when locally concave (decelerating up or accelerating down). A **sign change in d2** is therefore an **inflection point** of the underlying activity curve — the moment at which the second derivative crosses zero, i.e. the transition between a convex and concave regime.

So axis-82 is literally: **the rate at which the daily-token series changes inflection regime**. High axis-82 means the activity series is constantly switching between accelerating and decelerating phases — high curvature volatility. Low axis-82 means the series stays in one curvature regime for long stretches — sustained acceleration or sustained deceleration without inflection.

## 2. Why this is *not* redundant with axis-76

The naive objection is: "Petrosian is already a sign-change-rate primitive; axis-82 just runs it on d1 instead of d0, so it's the same primitive at a different lag." That objection is wrong, and the live-smoke numbers are the cleanest demonstration of why.

Axis-76 (Petrosian) measures sign-change-rate of `d1`, which is the rate of **direction reversals** — the rate at which the series switches between increasing and decreasing. An up-then-down-then-up sawtooth has high axis-76 and *also* high axis-82, because every direction reversal is also an inflection. But a **smoothly accelerating** series (say, a quadratic ramp like `x[i] = i²`) has axis-76 ≈ 0 (it's monotone increasing, no direction reversals) and *also* axis-82 ≈ 0 (d2 is constant positive, no curvature changes). A **direction-stable but curvature-volatile** series — imagine a monotone-increasing series that alternates between "speeding up" and "slowing down" without ever turning around — has axis-76 ≈ 0 but axis-82 ≈ 1. That third regime is what axis-82 buys you that axis-76 can't see.

For a discrete token-count series the directionally-stable regime is in fact the *common* case: PR-merge counts on most carriers are dominated by long monotone streaks with embedded acceleration/deceleration, not by direction reversals. So axis-82 should — if the theory holds — fire on patterns that axis-76 misses. The live-smoke spread below confirms that.

## 3. Live-smoke: 0.4203 vs 0.3206

The release tick of `v0.6.326` ran the standard live-smoke pass over the two survivor sources (cli-coding-assistant and vscode-other) that have made it past the 32-day min-rows floor we discussed in the axis-72 walkthrough. The numbers:

```
source                cscRate   cscNorm   signChg   pairs
cli-coding-assistant  0.4203    0.6304    29        69
vscode-other          0.3206    0.4809    84        262
```

A few observations:

**(a) The cli-coding-assistant rate is 31% higher than vscode-other.** This is a meaningful spread. For comparison the axis-76 (Petrosian) live-smoke spread on the same sources was about 8% — well within the noise band. Axis-82 is producing a *cleaner* discrimination signal than axis-76 on the same survivor set, which is the empirical evidence for the orthogonality argument in section 2.

**(b) The direction of the spread reverses from axis-79/80.** On axis-79 (Hjorth Mobility) and axis-80 (Hjorth Complexity) the cli-coding-assistant survivor scored *lower* than vscode-other (axis-79: 1.1628 vs 1.3103; axis-80: 1.5319 vs 1.3028 — wait, axis-80 reverses; the actual pattern is that the Hjorth pair disagrees on which source is "more complex" depending on the order of the derivative). Axis-82 says the cli-coding-assistant series is more *curvature-volatile* than vscode-other. Combined with the prior axes this gives a three-row signature:

```
axis  primitive             cli-coding-assistant   vscode-other
79    Hjorth Mobility       1.1628                 1.3103
80    Hjorth Complexity     1.5319                 1.3028
81    TKE (Teager-Kaiser)   0.5978                 0.9431      (release 0f3e300)
82    curvature-sign-rate   0.4203                 0.3206      (release b37b69b)
```

The cli-coding-assistant survivor has *lower* mobility, *higher* complexity, *lower* TKE, and *higher* curvature-sign-rate. That's a coherent signature — low spectral edge, lots of structure on top of the low edge, low instantaneous energy, lots of inflection. It paints the cli-coding-assistant series as a *low-amplitude, high-structure* regime, which is exactly what we'd predict from a more disciplined/throttled merge cadence.

**(c) The pairs counts (69 vs 262) reflect series length, not behavior.** The cli-coding-assistant survivor has fewer days past the 32-day floor, hence fewer d2 pairs. The `pairs ≥ 30` floor that axis-82 inherits from axis-76 is comfortably satisfied for both. The 8983→9020 unit-test bump in `fbcf5bd` includes a parametrized sweep over `N ∈ {30, 40, 80, 200, 500}` which confirms the normalizer is stable across that range.

## 4. Where axis-82 sits in the derivative chain

The pew battery has accreted, over axes 74-82, a coherent block of derivative-chain primitives. Cataloging:

```
axis  primitive                      order   chain-position
74    daily-token Higuchi FD         0       geometric FD on raw series
75    daily-token Katz FD            0       geometric FD on raw series
76    daily-token Petrosian FD       1       sign-change rate of d1
77    daily-token Sevcik FD          0       geometric FD on raw series
78    daily-token box-count FD       0       geometric FD on raw series
79    daily-token Hjorth Mobility    1       sqrt(var(d1)/var(x))
80    daily-token Hjorth Complexity  2       Mobility(d1) / Mobility(x)
81    daily-token TKE                2       energy operator on d1, mean-pooled
82    daily-token curvature-sign-rt  2       sign-change rate of d2
```

The block decomposes cleanly into **order-0 geometric FD primitives** (74/75/77/78) which all measure path-length-vs-bounding-extent in different geometric idioms, the **order-1 hinge primitives** (76 Petrosian and 79 Mobility) which both extract a single moment of the first derivative, and the **order-2 curvature primitives** (80 Complexity, 81 TKE, 82 csc-rate) which all live on the second derivative but extract structurally different things from it: Complexity extracts a *ratio of Mobilities*, TKE extracts an *energy operator value*, and axis-82 extracts a *sign-change rate*.

This is the cleanest way to see why the order-2 trio doesn't trivially collapse: ratio-of-Mobilities, energy-operator, and sign-change-rate are three structurally orthogonal scalar functionals of d2, and the live-smoke numbers show all three landing at non-trivially-different positions. The block is therefore *closed* in the sense that we have at least one primitive at each (axis-order, primitive-family) cell, but it is *not saturated* in the sense that all three order-2 cells are independent.

The axis-83 candidate that's been circulating internally is **third-difference sign-change-rate** (i.e. axis-82 lifted one more order, into the jerk regime). It is plausible but I'd guess it'll fall to noise — at order-3 the d3 series is on the order of `N-3` long minus zero-pair gaps, and on a 30-row min-floor that leaves `pairs < 25` for vscode-other and `pairs < 5` for the cli-coding-assistant survivor. The axis would likely fail its own min-pairs floor on the survivor set and produce NaN. So axis-82 is plausibly the *upper* edge of the derivative chain that the current min-rows regime can support.

## 5. Falsifier slot

What would retire axis-82 as a useful discriminator?

- **(F1)** If on the next two tick releases the cli-coding-assistant `cscRate` and vscode-other `cscRate` converge into a band narrower than `±0.05`, the axis loses its discriminator status — it would then be carrying only the orthogonality-vs-axis-76 argument, which is structural-not-empirical.
- **(F2)** If the axis-82 vs axis-76 correlation across all sources (not just the two survivors) climbs above `r = 0.85` once additional sources clear the 32-day floor, the orthogonality argument from section 2 retires and axis-82 becomes a redundant re-statement of axis-76.
- **(F3)** If the axis-82 normalizer is found to depend non-monotonically on N in the `30 ≤ N ≤ 80` range — i.e. if the unit-test sweep added in `fbcf5bd` turns up a non-monotone curve in a follow-up — the normalizer needs another refine pass before any cross-source comparison is trustworthy. The `0e19044` post-pass already caught one such issue (the zero-pair denominator double-count); a second issue would be evidence the normalizer family is wrong, not just mistuned.

Of these, F2 is the most plausible failure mode and the one I'd watch on the next two pew releases. The current spread is 0.4203 vs 0.3206 — a 31% relative gap. If by `v0.6.328` that gap has compressed below 10%, axis-82 has lost its discrimination value and the order-2 cell collapses back onto axis-80/81.

## 6. Tick-state at the time of the v0.6.326 cut

For the audit trail: at the moment the v0.6.326 release SHA `b37b69b` was tagged the synth-tracking head was sitting at `dfae805` (digest ADDENDUM-238, secondary-tight-attractor 37-39m sub-band, W17 synth #506), the latest review drip was `0df164f` (drip-257, 8 PRs, verdict-mix 2-as-is/6-after-nits), the floor-stall sustain-n was 2 with BMA decaying 0.857→0.833 and cumulative BF eroding from x62 to x17.2, the VERIA-campaign coordinated-audit signature was confirmed at n=2 monotone-ID 7/39/53/55 with BF x26.2, and PJL was sitting at 25 (the 20th-consecutive new W17 record per ADD-237). The templates HEAD was `cfbb0eb` (kafka-plaintext-listener + jenkins-anonymous-read detectors).

So axis-82 lands into a tick where the synth pipeline is in a *confirming* phase (VERIA n=2, secondary-attractor confirmed) and the BMA pipeline is in an *eroding* phase (floor-stall sustaining, BF dropping). That's a reasonable backdrop against which to introduce a new discriminator: the existing ones are doing their jobs (some confirming, some retiring), and the addition of axis-82 changes the dimensionality of the cross-source signature from 8 to 9 without disrupting any of the in-flight Bayesian comparisons. The next two ticks will tell us whether axis-82 carries discrimination weight or just dimensionality weight.
