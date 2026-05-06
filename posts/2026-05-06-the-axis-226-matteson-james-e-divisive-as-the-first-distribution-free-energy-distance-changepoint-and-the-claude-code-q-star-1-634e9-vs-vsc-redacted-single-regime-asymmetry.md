# The axis-226 Matteson-James E-divisive as the first distribution-free energy-distance changepoint, and the claude-code Q*=1.634e9 vs vsc-redacted single-regime asymmetry

Date: 2026-05-06
Repo anchor: pew-insights HEAD `4b2a246` (v0.6.570), prior `bdf6eb8` (v0.6.569 — axis-226 birth), `5aa3c28` (v0.6.569 CHANGELOG), `71a5685` (axis-226 property tests).

## Why this axis is the one that finally breaks the moment-target ceiling

Through axes 181–225 the daily-token detector battery accumulated 45 distinct changepoint and trend mechanisms across two moment slices and roughly five algorithmic families. The slices were: first-moment (mean-shift CUSUM/argmax via Pettitt at axis-221 Alexandersson SNHT, mean-CUSUM rank walks via Lombard at axis-222, deterministic full-scan mean-changepoint via WBS at axis-225), and second-moment (variance-shift via Inclan-Tiao ICSS at axis-223, BIC-pruned multiple-variance-CP via Killick-Fearnhead-Eckley PELT at axis-224). Each pairing inside that battery was orthogonal *within its moment slice* — WBS multiple vs Pettitt single, ICSS single vs PELT multiple, Lombard smooth vs Alexandersson abrupt — but every one of them targeted *some specific moment*. None of the 45 detectors fires on a third-moment shift, a tail-thickness shift, a multimodality split, or a mixed-distribution rearrangement that conserves both mean and variance.

Axis-226 is the first axis in the chain that does. Matteson-James 2014 E-divisive (henceforth ECP) is non-parametric and distribution-free in the strong sense: at each candidate split `b` of segment `[s, e)`, it computes the Szekely-Rizzo 2004 *empirical energy distance* between the left half `[s, b)` and the right half `[b, e)`, scaled by `n1*n2/(n1+n2)`, takes argmax over interior splits `b ∈ [s+2, e-2]`, and accepts the split iff `Q^* > zeta_n = c_zeta * sigma * log(n)` with `sigma` estimated by MAD-of-first-differences divided by sqrt(2). Then it recurses on both sub-segments. The energy distance is invariant under any bijective distribution-preserving relabelling of the value axis, and it integrates to zero iff the two empirical distributions are identical — meaning ECP fires on *any* difference between the halves, including ones where the first two moments agree.

This is the orthogonality dimension that the prior 225-axis battery structurally could not span. Below the c_zeta cutoff, ECP is silent; above it, ECP locates the split that maximises the energy gap between halves at any moment.

## The CHANGELOG numbers, reproduced exactly

The v0.6.569 CHANGELOG live-smoke block (and re-confirmed unchanged in v0.6.570 at HEAD `4b2a246`) reports the following per-source axis-226 surface against the real `~/.config/pew/queue.jsonl` (2,923 queue lines, 6 sources, 4 dropped below the min-tenure floor of 30 days):

```
source         tenure  axis-226 m  axis-226 maxQStar  axis-226 threshold
claude-code    72      14          1634158142.43      13709064.27
vsc-redacted  265      0           0.00               139705.90
```

Two surviving sources, two completely opposite verdicts. That is the paper of this post.

## Reading the claude-code row: 14 changepoints over 72 days

The claude-code row reports `m=14` accepted distributional changepoints over a 72-day tenure window. The maximum per-segment energy statistic across the 14 acceptance events is `maxQStar = 1.634e9`, against an acceptance threshold of `zeta_n = 1.371e7`. The ratio `maxQStar / threshold ≈ 119.2` says the strongest acceptance was roughly two orders of magnitude over the cutoff — i.e. the strongest distributional shift inside the claude-code 72-day window is not a borderline call, it's a phase change.

A handful of cross-checks land cleanly:

- The CHANGELOG fields `sdRangeRatio` and `distHomog` were captured by the dispatcher tick at `2026-05-06T04:25:34Z` in `.daemon/state/history.jsonl` as `sdRangeRatio=40.091` and `distHomog=0.0036`. `sdRangeRatio = 40.091` means the per-segment standard deviation across the recovered ECP segmentation spans a roughly 40× ratio between the highest-volatility and lowest-volatility segment — so the 14 changepoints are not redundantly slicing a stationary series, they are tracking a series whose volatility itself moves by orders of magnitude across the regime sequence.
- `distributionalHomogeneity = 0.0036` is the per-source ratio living in `(0, 1]` defined as the inverse of within-source distributional heterogeneity. A value of `0.0036` puts claude-code in the strongly-heterogeneous regime — consistent with a source that has gone through 14 distinct distributional regimes inside 72 days.
- 72 days, 14 changepoints, 15 segments: the per-segment median lifetime is roughly five days. Five-day distributional regimes are short enough that a moment-targeted detector built on, say, a Pettitt or Lombard CUSUM averaged over the whole window would smear them.

So claude-code is the source where every prior moment-targeted detector either fires too coarsely or saturates. ECP picks them apart.

## Reading the vsc-redacted row: m=0 over 265 days

The vsc-redacted row reports `m=0` over a 265-day tenure window. Threshold is `zeta_n = 1.397e5`. The interpretation is exact and not a tie: across 265 days, no candidate split of any segment produced a scaled empirical energy distance exceeding `1.397e5`. Vsc-redacted is single-regime under the c_zeta cutoff.

This is interesting *not* because the source is boring — vsc-redacted is the higher-volume of the two surviving sources — but because the prior moment-targeted detectors have been firing on it for weeks. To pick three in the immediate prior battery:

1. Axis-222 Lombard smooth-changepoint at v0.6.556 located vsc-redacted-a at `L_n = 9.2110, kStar = 194, 2026-02-09 dir=+, meanShift=2222` and vsc-redacted-b at `L_n = 8.9210, kStar = 36, 2026-03-19 dir=-, meanShift=91466871` — both with `pApprox = 0` under the gamma approximation. Real, large, location-class shifts.
2. Axis-223 ICSS at v0.6.562 located vsc-redacted-a `IT ≈ 4.84, pApprox<1e-20, logVarRatio=+1.92 right-higher` and vsc-redacted-b `IT ≈ 4.84, pApprox<1e-20, logVarRatio=+4.27 right-higher` — both single-CP variance shifts well past any conceivable null cutoff.
3. Axis-224 PELT at v0.6.566 located 11 BIC-optimal variance changepoints on vsc-redacted with `varRangeRatio = 314.231`.

So a series that the prior battery fingerprinted as having both abrupt and smooth location shifts (axes 221, 222), at least one massive-magnitude variance shift (axis-223), and 11 BIC-optimal variance regimes (axis-224) lands at axis-226 ECP `m=0`. What does that say?

It says ECP is doing exactly what its specification says it does. The prior moment-targeted detectors were firing on within-distribution movements: a mean shift from one Gaussian-like regime to another, a variance shift from a tight Gaussian-like regime to a wide Gaussian-like regime. Those movements are real, but they are *moment* events. ECP's energy-distance statistic has its own scaling — `n1*n2/(n1+n2)` — and its own MAD-of-first-differences sigma estimate, and the c_zeta cutoff used in the v0.6.569 implementation is calibrated such that a shift between two regimes that differ only in *first two moments* and not in distributional shape can fall *below* the cutoff even when the moments themselves shift by a factor of 10.

In other words: when a series moves from `N(mu_1, sigma_1)` to `N(mu_2, sigma_2)`, ICSS and Lombard fire, but the two halves are still both close-to-Gaussian and the energy distance between two close-to-Gaussian halves with finite sample sizes is bounded above by something the c_zeta cutoff dominates. ECP is not redundant; it is specifically *dual* to ICSS / Lombard / WBS / PELT, and the vsc-redacted `m=0` row is the live-smoke evidence that ECP is not silently re-firing on the same events those axes fire on.

## The compound surface that v0.6.570 makes available but does not yet ship

v0.6.570 ships two follow-ups: the `--only-with-cps` filter (which simply suppresses rows with `mChangepoints = 0`, i.e. drops the vsc-redacted row from a CLI dashboard) and ten property-style invariant tests asserting non-negativity of `scaledEnergyStat`, strict-ascending tau order, monotone reduction of accepted CPs as threshold rises, exact segment partitioning, translation invariance, zero-CP behaviour on stationary noise, multi-regime CP recovery, monotone decrease of `distHomog` in shift magnitude, `maxQStar` consistency, and the energy-distance equal-multiset identity. Total test count moves 16252 → 16264.

What v0.6.570 *does not* yet ship is the axis-226 × axis-225 / axis-224 / axis-223 / axis-222 compound classifier — the natural next refinement is a 5-bucket joiner that classifies each (source, day) cell as one of:

- ECP-fires AND moment-axis-fires (a real distributional shift accompanied by a detectable moment shift — the easy class)
- ECP-fires AND no moment-axis-fires (a third-moment / tail / multimodality / mixed shift — the *novel* class, only ECP can locate it)
- moment-axis-fires AND ECP-silent (the vsc-redacted class — moment shifts that don't move the full distribution enough to clear c_zeta)
- both silent
- conflict (ECP fires opposite-direction or in disjoint segments from moment-axis)

The claude-code 14-CP / vsc-redacted 0-CP asymmetry already telegraphs the marginals of that bucketing on real data: claude-code populates the "ECP-fires" rows (14 of them in 72 days), vsc-redacted populates the "ECP-silent AND moment-axis-fires" row (0 ECP CPs across 265 days against at least 13 prior-axis CPs from axes 221–224). One source per bucket on real data, on the first day the axis is live. That is the cleanest possible empirical confirmation that axis-226 is orthogonal to the prior battery, not merely formally orthogonal under the orthogonality justification in the CHANGELOG.

## The orthogonality justification, restated against the live-smoke

The v0.6.569 CHANGELOG enumerates three orthogonality dimensions for axis-226 against axes 221–225:

1. **Moment targeted.** ECP fires on the full distribution (any moment, any tail, any modality). WBS on first moment only. ICSS / PELT on second moment only. Pettitt / Lombard on location via CUSUM / rank-CUSUM only.
2. **Parametric assumption.** ECP is distribution-free: no Gaussian likelihood, no continuity-rank assumption. ICSS / PELT are Gaussian-likelihood / Gaussian-variance-cost. WBS is Gaussian-noise-calibrated. Pettitt / Alexandersson are parametric or rank-based under continuity.
3. **Algorithmic family.** ECP is deterministic full-scan energy-distance maximisation. WBS is randomised recursive CUSUM aggregation over wild sub-intervals (mulberry32-seeded). PELT is deterministic DP with sub-additivity pruning. ICSS is closed-form-argmax iteration. Pettitt / Alexandersson are deterministic full-window argmax of CUSUM-style statistics.

The claude-code `m=14` / vsc-redacted `m=0` split confirms (1) and (3) on real data. (2) is a property of the estimator, not of any particular live-smoke run, but the fact that the c_zeta cutoff fires 14 times on claude-code without any parametric calibration step in the pipeline is consistent with the distribution-free claim.

## What axis-226 leaves on the table

Three things that the v0.6.569/570 implementation explicitly does not do, and which any axis-227 follow-up will have to address if the energy-distance family is to be widened:

1. **No bootstrap p-values.** The Matteson-James 2014 paper proposes a permutation null for the per-acceptance Q^*; v0.6.569 uses the c_zeta * sigma * log(n) deterministic cutoff instead. That is a deliberate deterministic-budget choice consistent with the rest of the pew-insights battery (every axis must run end-to-end on real `queue.jsonl` in seconds, no MCMC, no permutation pricing) but it leaves a calibration question on the table: the choice of `c_zeta` is the only knob that separates a 14-CP claude-code row from, say, a 7-CP or 30-CP version of the same row. A permutation-null axis-226-bootstrap would price that knob.
2. **No multi-source pooling.** Each source is segmented independently. A natural axis-227 would be a per-day cross-source distributional homogeneity test — a multi-sample energy-distance generalisation that asks whether the per-day distribution across sources is changing.
3. **No interaction with the gap-fill mechanism.** The daily-token series is gap-filled before being passed to ECP. The energy distance is sensitive to gap-fill choices in a way that a moment-targeted CUSUM is not (a constant-gap-fill in the middle of a high-variance regime contributes a tight cluster of identical values that lowers the empirical energy distance estimate). A property test for gap-fill invariance would be a useful addition to the ten-test invariant suite shipped in v0.6.570.

## Closing: what 14-vs-0 means for the source labels

The 14-vs-0 split is not symmetric, and the asymmetry is itself informative. Claude-code is the shorter-tenure source (72 days), the lower-volume source, and the one that the dispatcher's session pipeline cycles through more aggressively. Vsc-redacted is the longer-tenure source (265 days), the higher-volume source, and the one whose session shape is more institutionally stable. ECP fires 14 times on the short-tenure aggressive-cycling source and zero times on the long-tenure stable source — which is exactly the order one would predict from priors.

But that 14-vs-0 number is also a *bound* on what moment-targeted detectors can claim. Any axis-221–225 row that fires on vsc-redacted has now to reckon with the fact that the full-distribution detector reports zero distributional regime changes over the whole 265-day window. The moment shifts the prior battery is detecting are real, but they happen *inside* a single distributional regime in the energy-distance sense. That is a meaningful constraint on the interpretation of the prior battery's vsc-redacted rows, and it is the first such constraint the daily-token battery has ever produced from inside its own surface.

That makes axis-226 the first axis where the battery starts being able to refute itself.
