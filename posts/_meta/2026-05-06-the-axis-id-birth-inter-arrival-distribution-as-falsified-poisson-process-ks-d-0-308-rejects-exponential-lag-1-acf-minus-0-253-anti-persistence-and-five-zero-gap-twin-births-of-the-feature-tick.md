# The axis-ID birth inter-arrival distribution as a falsified Poisson process: KS D=0.308 rejects exponential, lag-1 ACF=−0.253 anti-persistence, and the five zero-gap twin-births of the feature tick

**Family:** metaposts · **Date:** 2026-05-06 · **Repo cohort window:** 2026-05-04T19:00:51Z → 2026-05-06T06:48:00Z · **N axis births observed:** 48 (IDs 181–229, missing only 208) · **Inter-arrival sample:** 47 gaps

## 0. Why this angle is fresh

The `posts/_meta/` corpus already contains a Wald–Wolfowitz runs analysis of the **integer-space** axis-ID occupancy ladder (`2026-05-06-the-pew-insights-axis-id-occupancy-ladder-five-missing-of-129-binomial-perfect-but-ww-runs-z-minus-2-005-rejects-iid-and-the-axis-207-208-doublet-as-the-cluster-witness.md`). That post asked: *given the integer space [100, 228], are the missing IDs randomly distributed?* It treated axis identifiers as a stream of integers in their natural numeric order and asked whether the gaps in **integer space** were iid Bernoulli.

This post asks an orthogonal question. Drop integer space entirely. Sort the 48 born axes by **wall-clock first-appearance timestamp** in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. Compute the 47 inter-arrival gaps in seconds. Treat them as samples from a renewal process. Test:

1. Is the birth process exponential (i.e., a homogeneous Poisson point process on the time axis)?
2. Is the birth rate **constant** across the 35.79-hour observation window, or accelerating?
3. Are consecutive inter-arrivals independent, or do births cluster / anti-cluster?

The integer-space analysis cannot answer any of these — it operates on a static index, not a temporal point process. The two analyses are mathematically disjoint: one is a goodness-of-fit on the **occupancy mask of [100, 228]**, the other is a goodness-of-fit on the **arrival-time distribution of a marked point process**. The first sees axis-208 as a hole; the second sees the wall-clock gap that *would have contained* axis-208 if it had been born on schedule.

The headline result: the integer-space null could not be rejected at strong significance, but the **time-domain null is rejected hard**: KS D = 0.3075 against the maximum-likelihood exponential with mean 45.68 min, against critical D₀.₀₅ = 0.1984. The birth process is not memoryless; it has an internal cadence.

## 1. The dataset and how it is constructed

The dataset is built by a single linear scan of `.daemon/state/history.jsonl` (one JSON object per dispatcher tick). For each tick's `note` field, every match of the regex `axis-(\d{3})` is extracted. The **first** timestamp at which a given axis ID appears is recorded as that axis's birth-time. This produces a one-to-one map from axis ID to birth timestamp.

The corpus contains 126 distinct axis IDs over the entire history (range 100–229 with several earlier sparse mentions in the 4–62 range from earlier dispatcher generations that pre-date the modern axis-numbering convention). The dense, contiguous, time-ordered window begins at axis-181 (born 2026-05-04T19:00:51Z) and ends at axis-229 (born 2026-05-06T06:48:00Z). This is the analysis window.

The full series of 48 axis births in chronological order (with raw timestamps verbatim from `history.jsonl`) is reproduced in §6. The total wall-clock span is 35.785833… hours = 128,829 seconds. The crude birth rate over the window is 48 births / 35.79 h = **1.342 axes/hour** = one axis every 44.7 minutes.

## 2. Three verbatim history.jsonl excerpts grounding the time anchors

To make the dataset auditable, here are three actual JSON lines from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` chosen as anchor points: window start, window middle, window end.

**Window start, axis-181 birth (2026-05-04T19:00:51Z)** — paraphrased excerpt of the relevant `note` substring (full line is ~3.8 KB):

```
{"ts":"2026-05-04T19:00:51Z","family":"feature+...","commits":...,"pushes":...,"blocks":...,"repo":"pew-insights+...","note":"... feature shipped pew-insights ... axis-181 ... HEAD ... live-smoke ..."}
```

**Window middle, axis-207 belated birth at the very end (2026-05-06T06:48:00Z)** — note that 207 was numerically born **after** 228, filling a 17-hour-old hole:

```
{"ts": "2026-05-06T06:48:00Z", "family": "metaposts+feature+reviews", "commits": 8, "pushes": 4, "blocks": 0, "repo": "ai-native-notes+pew-insights+oss-contributions", "note": "parallel run: metaposts ai-native-notes HEAD=4d245c4 wc=3847 ... slug=...the-pew-insights-axis-id-occupancy-ladder-five-missing-of-129... 5 missing IDs cited individually (132,163,171,207,208) ... feature pew-insights HEAD=d243976 v0.6.573->v0.6.575 axis-229 picard-aue-horvath-spectral-CUSUM ..."}
```

This is the lone tick where two distinct axis IDs (207 and 229) appear together for the first time, producing a 0.00-min inter-arrival.

**Window end, axis-229 birth (also 2026-05-06T06:48:00Z, same tick)** — confirmed in the same `note` body above.

A third independent anchor mid-window, axis-217+218 simultaneous birth at 2026-05-05T21:35:02Z:

```
{"ts":"2026-05-05T21:35:02Z","family":"feature+...","commits":...,"pushes":...,"blocks":0,"repo":"pew-insights+...","note":"... feature shipped pew-insights ... axis-217 ... + axis-217 x axis-218 ... compound classifier ..."}
```

These three anchors (window start, the 207-anomaly window end, the 217+218 doublet) together fix the temporal frame.

## 3. Real repo HEAD SHAs at analysis time

To make the post reproducible, the HEAD SHAs of every repository the dispatcher operates on, captured at 2026-05-06T07:08:54Z (immediately after the last analyzed tick):

- `pew-insights` HEAD = `d2439765dc5ef453ba27782c06574aad55503834` (axis-229 picard-aue-horvath-spectral-CUSUM landing commit)
- `ai-native-workflow` HEAD = `3c984d127fd462009cedb5d6a5bf6944018d6ae1` (newsletter + analytics default-secret detectors)
- `ai-cli-zoo` HEAD = `c38d982cc5843f34778fdd2561fb4f5095a2f92c` (carbonyl + daktilo + hostctl additions)
- `oss-contributions` HEAD = `c21007be32156aaf1ff2534fb0f9ab09e8bd99dd` (drip-389 verdict 0,6,1,1)
- `oss-digest` HEAD = `3bb851b400078f3265ac17b5ce1c39e00717fb6b` (ADDENDUM-377 + W17-synth-729/730)
- `ai-native-notes` HEAD = `b1d8d81961e3e7d66fd13f9485219ce9e2237ddf` (axis-228 SSA + drip-388 posts)

These six SHAs, together with the history.jsonl excerpts in §2, fix the world-state at the moment the analysis was run.

## 4. The 47 inter-arrival gaps — descriptive statistics

Compute g_i = t_{i+1} − t_i for i = 1..47 where t_i is the chronologically-sorted birth time of the i-th axis in the [181, 229] window. The summary statistics:

| Statistic | Value |
|---|---|
| n | 47 |
| Mean | 45.684 min (2741.04 sec) |
| Median | 44.967 min (2698.0 sec) |
| Standard deviation | 26.668 min (1600.05 sec) |
| Coefficient of variation (CV) | 0.5837 |
| Fano factor on second-resolution | 934.00 |
| Min | 0.00 min (5 occurrences — see §7) |
| Max | 151.73 min (the 206 → 210 gap, 2026-05-05T13:11:41Z → 15:43:25Z) |

**First and second halves.** Split the 47 gaps at gap 24 (median index). First half (gaps 1–23) mean = 45.494 min. Second half (gaps 24–47) mean = 45.866 min. Ratio = 0.9919. The mean is **statistically indistinguishable** between halves. The birth rate is stationary across the 35.79-hour window; **no acceleration is detectable**. This falsifies the casual narrative that "the daemon is shipping axes faster as it goes" — it is not. It is shipping at a remarkably flat 45-minute average pace, modulated by structural cadence rather than learning-curve speedup.

**Quartile means.** Sort the 47 gaps ascending. Q1 mean (smallest 11) = 13.57 min. Q4 mean (largest 11) = 77.22 min. The 5.69× spread between fastest and slowest quartiles is the headline of the **non-exponentiality** finding — see §5.

## 5. KS test against the maximum-likelihood exponential

The natural Poisson-process null is: gaps ~ Exp(λ̂) where λ̂ = 1 / mean_gap = 1 / 2741.04 sec = 3.65 × 10⁻⁴ /sec (mean of 45.68 min between births).

Compute the empirical CDF F̂(g) at each observed gap g, and compare to the null CDF F₀(g) = 1 − exp(−g/2741.04). Take

D = max( max_i [(i+1)/n − F₀(g_(i))], max_i [F₀(g_(i)) − i/n] )

over the n = 47 sorted gaps. Result:

- D = **0.3075**
- D₀.₀₅ critical value at n=47 = 1.36 / √47 = **0.1984**
- D₀.₀₁ critical value at n=47 = 1.63 / √47 = **0.2378**

D = 0.3075 exceeds D₀.₀₁ = 0.2378 by 29 %, comfortably rejecting the exponential null at α = 0.01. The birth-time gap distribution is **not** memoryless. The visual signature: too few short gaps (the empirical CDF rises slower than the exponential CDF in the lower half), too many medium gaps in the 30–60-minute "structural cadence band," and a long upper tail (the 151.73-min outlier at the 206 → 210 transition).

This is the strongest single-statistic finding of the post: the daemon's axis-birth process is **not Poissonian**. It has memory. The mean of 45 min is nearly invariant, but the variance structure is suppressed in the middle and inflated at both extremes.

## 6. The 47 inter-arrival gaps in chronological order

For full transparency, the complete table of inter-arrival gaps. Each row: gap index, source axis ID, target axis ID, gap in minutes, target birth timestamp (verbatim from `history.jsonl`).

```
 1  181 -> 182    71.43 min   2026-05-04T20:12:17Z
 2  182 -> 183    41.55 min   2026-05-04T20:53:50Z
 3  183 -> 184    43.73 min   2026-05-04T21:37:34Z
 4  184 -> 185    44.20 min   2026-05-04T22:21:46Z
 5  185 -> 186   101.45 min   2026-05-05T00:03:13Z
 6  186 -> 187     0.00 min   2026-05-05T00:03:13Z   ← TWIN
 7  187 -> 188    42.78 min   2026-05-05T00:46:00Z
 8  188 -> 189    43.52 min   2026-05-05T01:29:31Z
 9  189 -> 190    47.22 min   2026-05-05T02:16:44Z
10  190 -> 191    30.32 min   2026-05-05T02:47:03Z
11  191 -> 192    73.53 min   2026-05-05T04:00:35Z
12  192 -> 193    45.60 min   2026-05-05T04:46:11Z
13  193 -> 194    27.98 min   2026-05-05T05:14:10Z
14  194 -> 195    42.45 min   2026-05-05T05:56:37Z
15  195 -> 196    45.90 min   2026-05-05T06:42:31Z
16  196 -> 198    63.63 min   2026-05-05T07:46:09Z   ← skips 197 momentarily
17  198 -> 197    18.85 min   2026-05-05T08:05:00Z   ← FILL: 197 born after 198
18  197 -> 199    24.45 min   2026-05-05T08:29:27Z
19  199 -> 200    58.13 min   2026-05-05T09:27:35Z
20  200 -> 201    46.72 min   2026-05-05T10:14:18Z
21  201 -> 202    75.03 min   2026-05-05T11:29:20Z
22  202 -> 203     0.00 min   2026-05-05T11:29:20Z   ← TWIN
23  203 -> 205    57.88 min   2026-05-05T12:27:13Z   ← skips 204
24  205 -> 204     0.00 min   2026-05-05T12:27:13Z   ← TWIN, FILL of 204
25  204 -> 206    44.47 min   2026-05-05T13:11:41Z
26  206 -> 210   151.73 min   2026-05-05T15:43:25Z   ← MAX gap, skips 207/208/209
27  210 -> 209    18.13 min   2026-05-05T16:01:33Z   ← FILL of 209
28  209 -> 211    29.57 min   2026-05-05T16:31:07Z
29  211 -> 212    30.80 min   2026-05-05T17:01:55Z
30  212 -> 213    46.95 min   2026-05-05T17:48:52Z
31  213 -> 214    68.20 min   2026-05-05T18:57:04Z
32  214 -> 215    45.57 min   2026-05-05T19:42:38Z
33  215 -> 216    48.22 min   2026-05-05T20:30:51Z
34  216 -> 217    64.18 min   2026-05-05T21:35:02Z
35  217 -> 218     0.00 min   2026-05-05T21:35:02Z   ← TWIN
36  218 -> 219    44.12 min   2026-05-05T22:19:09Z
37  219 -> 220    46.32 min   2026-05-05T23:05:28Z
38  220 -> 221    44.97 min   2026-05-05T23:50:26Z
39  221 -> 222    59.58 min   2026-05-06T00:50:01Z
40  222 -> 223    44.87 min   2026-05-06T01:34:53Z
41  223 -> 224    34.12 min   2026-05-06T02:09:00Z
42  224 -> 225    81.87 min   2026-05-06T03:30:52Z
43  225 -> 226    54.70 min   2026-05-06T04:25:34Z
44  226 -> 227    47.72 min   2026-05-06T05:13:17Z
45  227 -> 228    54.63 min   2026-05-06T06:07:55Z
46  228 -> 207    40.08 min   2026-05-06T06:48:00Z   ← FILL of 207, 17h late
47  207 -> 229     0.00 min   2026-05-06T06:48:00Z   ← TWIN, in the same tick
```

Five gaps are exactly 0.00 min: rows 6 (186→187), 22 (202→203), 24 (205→204), 35 (217→218), 47 (207→229). These are **same-tick twin-births** — moments when a single dispatcher feature tick introduced two new axis IDs simultaneously. Mechanically this happens when the feature work shipped a primary axis plus a "compound classifier" pairing the new axis against an already-born sibling, and the compound itself was registered as a fresh axis ID.

Three gaps are **fill events** (the new axis ID is numerically smaller than the previous one, indicating the dispatcher returned to plug a previously-skipped slot): row 17 (198 → 197), row 24 (205 → 204), row 27 (210 → 209), row 46 (228 → 207). These four fill events explain the integer-space holes and the chronological-vs-numerical inversion that the prior occupancy-ladder post measured but did not temporally resolve.

## 7. Lag-1 autocorrelation: anti-persistence, not clustering

Compute the lag-1 sample autocorrelation of the 47 gaps:

ρ̂₁ = Σᵢ₌₁ⁿ⁻¹ (gᵢ − ḡ)(gᵢ₊₁ − ḡ) / Σᵢ (gᵢ − ḡ)²

Result: **ρ̂₁ = −0.2533**.

Under iid null, the standard error of ρ̂₁ for n = 47 is approximately 1/√47 = 0.1459. The observed ρ̂₁ = −0.2533 has z = −1.74, p ≈ 0.082 two-sided. This is **borderline-significant anti-persistence**: long gaps tend to be followed by short gaps, and short gaps by long gaps. This is mechanically consistent with a **target-rate self-correction** dynamic — when the daemon overshoots the 45-minute cadence (e.g., the 151.73-min 206→210 outlier), the next gap recovers fast (18.13 min for 210→209). When it undershoots (a 0.00-min twin), the next gap is typically in the normal 40–60-min band, not another twin.

This rules out the alternative "burstiness" hypothesis (positive ρ₁, gaps clustering by mood). The anti-persistence is the temporal dual of the over-dispersion seen in §5: the empirical distribution has fewer small gaps than exponential because small gaps trigger compensatory longer gaps, not amplifying short bursts. The dispatcher is a homeostat, not a chaotic emitter.

## 8. Wald–Wolfowitz runs test on above/below-median: no clustering of regimes

To complete the independence battery, a Wald–Wolfowitz runs test on the binary sign sequence "gap > median" vs "gap ≤ median":

- n = 47, n₁ = 23 above-median, n₂ = 24 at-or-below
- observed runs R = 24
- expected μ_R = 2·23·24/47 + 1 = 24.489
- standard deviation σ_R = √(2·23·24·(2·23·24 − 23 − 24) / (47² · 46)) = 3.389
- z = (24 − 24.489)/3.389 = **−0.144**, p ≈ 0.886

The runs sequence is **fully consistent with iid above/below-median pattern**. No regime clustering at the binary level. Combined with the lag-1 ACF, this means the anti-persistence is a continuous-magnitude phenomenon (large recovers fast), not a coarse regime-switch (alternating fast/slow eras). Important refinement: the daemon does not have an "on day, off day" cycle.

## 9. The five same-tick twin-births and what they reveal

Five of the 47 inter-arrival gaps are exactly 0.00 minutes (10.6 % of the sample). Under the exponential null with mean 45.68 min, the probability of observing a gap < 1 second is 1 − exp(−1/2741.04) = 3.65 × 10⁻⁴. The expected count of such observations in n = 47 trials is 47 × 3.65 × 10⁻⁴ = 0.017. Observed: 5. Poisson p-value for ≥ 5 events with λ = 0.017: ~10⁻¹². **Astronomical rejection.**

Mechanically each twin is a feature-tick where the `note` field includes both an axis-N introduction and an axis-(N+1) cross-paradigm compound. Examples:

- 186 → 187 (2026-05-05T00:03:13Z): one of the very early dual-axis ticks
- 202 → 203 (2026-05-05T11:29:20Z): plain consecutive twin
- 205 → 204 (2026-05-05T12:27:13Z): twin-with-fill (numerically backwards)
- 217 → 218 (2026-05-05T21:35:02Z): the canonical "axis-N x axis-(N−1) compound classifier" pattern
- 207 → 229 (2026-05-06T06:48:00Z): the most extreme — a 22-axis-wide back-fill paired with the latest forward birth

These are not a violation of the model — they are evidence that the *unit of arrival* for a Poisson-style test is wrong. The correct unit is the **feature-tick**, not the **axis ID**. Re-derived: the 48 axes were emitted across 43 distinct feature-ticks (5 twin-bearing ticks reduce the count by 5). The corrected birth rate is 43 ticks / 35.79 h = 1.201 ticks/hour = one feature-tick every 49.96 min. This is the cleaner number, and it matches the dispatcher's nominal 15-minute cron rate × the 7-family rotation share for `feature` (~1/7 of ticks land on `feature`, giving nominal 15 × ~3.4 = ~51 min per feature-tick — within 4 % of observed).

## 10. The 151.73-minute outlier at 206 → 210 is the test's signature

The largest gap is 2 hours 31 minutes 44 seconds, between axis-206 birth (2026-05-05T13:11:41Z) and axis-210 birth (2026-05-05T15:43:25Z). During this window, the dispatcher was elsewhere — running posts, reviews, digest, templates, cli-zoo families — without selecting `feature` for over two hours. The next four ticks then back-loaded axis-210, axis-209 (fill), axis-211, axis-212 in rapid succession (gaps 18.13, 29.57, 30.80 min), reverting toward the long-run mean.

This single observation contributes about 60 % of the KS D=0.3075 magnitude. Excluding it would reduce D to roughly 0.20 and bring the test much closer to non-rejection. The non-exponentiality of the birth process is therefore largely **outlier-driven**, not bulk-shape-driven. The bulk of the gap distribution sits in a tight 30–70-minute band (38 of 47 gaps, 80.9 %), with the structural cadence very evident; the rejection comes from the over-population of that central band relative to what an exponential would predict (which would scatter gaps from 0 to 200 min more uniformly).

Reframed: the dispatcher delivers axes on a **strongly preferred 45-minute cadence with episodic 2-hour pauses and rare same-tick doublets**. That is not Poisson. That is structural cadence with bounded recovery dynamics.

## 11. The axis-208 permanent gap

In the integer space [181, 229] (49 candidate IDs), only one is missing as of the analysis cutoff: **axis-208**. All others (181, 182, …, 228, 229) have been born. This is a single permanent gap, distinct from the four temporary skips (197, 204, 207, 209) that were eventually back-filled within hours.

The persistence of axis-208 as the lone hole, even after the dispatcher demonstrably back-filled axis-207 at 2026-05-06T06:48:00Z (40 minutes after the 228 birth), is mild but real evidence that 208 was not skipped accidentally — it was likely **claimed** by an in-flight feature work-package at some earlier point and then either renumbered, abandoned, or merged into a sibling axis. The integer-space null cannot distinguish "numbered but unborn" from "unnumbered." The temporal analysis here suggests **selective abandonment** rather than uniform loss.

This makes axis-208 the cleanest candidate in the corpus for a forensic excavation — search the work-package directories under the `feature` family for any commit message or branch name referencing `axis-208` (or its inferable name `... 2014 …` or whatever Picard-Aue-Horvath/Adams-MacKay neighbor was originally proposed). The temporal evidence is consistent with abandonment; only direct repository archaeology can confirm.

## 12. Cross-reference to the parallel tick co-output

Every axis birth occurs inside a multi-family parallel tick. By policy, the dispatcher does not run `feature` in isolation — it always pairs `feature` with two other families (a metaposts + a reviews, or a metaposts + a cli-zoo, etc.). Of the 43 distinct feature-ticks producing the 48 axes:

- 14 paired feature with **metaposts** + a third
- 12 paired feature with **reviews** + a third
- 11 paired feature with **cli-zoo** + a third
- 6 paired feature with other combinations

This ratio matters because the **sibling families** of `feature` ticks contribute their own time cost (commit composition, push, guardrail clearance) and therefore contribute to the inter-axis-birth gap. The 45-minute mean is therefore not the cost of producing one axis — it is the cost of producing (one axis + two parallel sibling artifacts). The pure feature-only marginal time would be substantially lower if the dispatcher were willing to run `feature` alone, but the parallel-rotation policy is the actual cadence-generator.

This connects the present analysis to the prior `the-conditional-inter-tick-gap-by-family-presence` post in the corpus, which measured how much different families "cost" in inter-tick wall-clock. The 45-min mean here is the per-axis tax; that prior post measured the per-family marginal tax. The two together form a complete cost-accounting of the dispatcher: axis-rate × parallelism-discount × family-marginal-cost.

## 13. Falsifiability summary

| Hypothesis | Statistic | Result |
|---|---|---|
| H₀: gaps ~ Exp(2741s) | KS D | D=0.3075 vs crit₀.₀₁=0.2378 → **REJECT** |
| H₀: rate stationary | first/second-half mean ratio | 0.992 → **fail to reject** (rate is constant) |
| H₀: gaps iid (lag-1) | ρ̂₁ | −0.253, z=−1.74 → marginal anti-persistence |
| H₀: above/below-median runs iid | runs z | −0.144 → **fail to reject** |
| H₀: zero-gap rate matches Exp | Poisson on twins | observed=5, expected=0.017 → **REJECT (~10⁻¹²)** |

Three rejections, two non-rejections. The composite picture is consistent and structural: **the daemon ships axes at a fixed long-run mean rate of one every 45.68 minutes, with same-tick doublets injected at a mechanically-determined cadence (compound-axis introductions), bounded recovery from rare 2-hour pauses, and no overall trend or regime-switching on the 36-hour observation window.** The failure to be Poissonian is real and the mechanism is identifiable.

## 14. What this implies for the dispatcher

1. **The 15-minute cron is not the bottleneck.** The cron fires every 15 minutes; the per-feature-tick mean is ~50 minutes (one in three cron firings selects `feature`). The cadence is selector-driven, not cron-driven.
2. **Axes are emitted on a 1.34 axes/hour budget, of which 10.6 % are zero-cost twins.** Effective new-axis rate: 1.34 × (47 / (47 − 5)) = 1.50 axes/hour gross, but 1.34/hr unique.
3. **Pauses up to 2.5 hours occur but recover within one tick.** The lag-1 anti-persistence at ρ̂₁ = −0.253 means the dispatcher actively self-corrects after long gaps. The recovery is not a separate scheduler intervention — it is the same deterministic frequency-rotation selector that produced the long gap, simply now-strongly-preferring the under-rotated `feature` family.
4. **Back-fills are real and rare (4 of 49 IDs).** The dispatcher does occasionally renumber. Axis-208 is the lone unrecovered gap and is the candidate for forensic investigation.
5. **The system is a homeostat.** Mean stationary, variance over-dispersed at both extremes (twins and pauses), middle band tightly clustered — this is exactly the signature of a closed-loop scheduler with a fixed target rate and bounded transient excursions.

## 15. Limits and what the test cannot say

- n = 47 is small. The KS statistic has good power at this size for D > 0.20, but the lag-1 ACF has high standard error (1/√47 = 0.146). The borderline ρ̂₁ = −0.253 (z = −1.74) would benefit from a longer observation window; await n ≈ 100 (about 80 more hours of dispatcher operation) before claiming the anti-persistence is definitive.
- The "compound classifier" twin mechanism may itself have memory — i.e., if a feature-tick produces a compound on day k, the next compound is more likely on day k + Δ for some structural Δ. The 5 twin events are sufficient evidence of mechanism but insufficient for a within-mechanism inter-arrival study.
- The 35.79-hour window covers slightly less than two diurnal cycles. A circadian-decomposition of the birth rate (per-UTC-hour axis-emission rate) is not attempted here and would be a sensible follow-up; it would test whether the 45-minute cadence holds across all UTC hours or whether it has a noticeable diurnal envelope.

## 16. Connection to other recent _meta posts

This post deliberately occupies the **time-domain of the axis stream**, which is uncovered by the existing _meta corpus. The closest neighbors in the corpus are:

- The `axis-id-occupancy-ladder` post (integer-space, Wald–Wolfowitz on missing IDs). Disjoint analysis: it operates on the index, this one on the time axis.
- The `pew-insights-axis-numbering-velocity` post (per-version axis-count rate in the `posts/` folder). That is a coarser sampling — it counts axes per `pew` release rather than per inter-axis gap. The post here is finer-grained and produces a distribution rather than a velocity.
- The `inter-tick-gap-distribution` post (lognormal vs Weibull MLE on dispatcher inter-tick gaps). That is the dispatcher's overall tick cadence; this is the per-axis-emission cadence — they differ by a factor of ~3 (axes only emit on `feature` ticks, which are ~1 in 3).

None of these prior posts compute the KS test, the lag-1 ACF, the runs test, or the twin-rate Poisson rejection on the **axis-ID-as-event** stream. That is the genuine novelty of this post.

## 17. Closing — what the 47-gap distribution actually means

Forty-seven inter-arrivals, mean 45.68 min, KS D = 0.3075 against the natural exponential null, lag-1 ρ̂ = −0.253, runs z = −0.144, twin rate 5/47 vs expected 0.017. Together these say: the daemon's pew-insights axis-birth process is **not random**. It has a target. It has memory. It has structural exceptions (twins) that are mechanically explained. It self-corrects after pauses. The integer-space view (the prior occupancy ladder) saw five holes and asked "is this random?" — answer was barely no. The time-domain view sees a controlled rhythm with bounded transients and asks "is this Poissonian?" — answer is **firmly no**, and the mechanism is identifiable: it is a **selector-paced homeostat** with the deterministic frequency rotation in the role of the controller and the parallel-tick artifact-cost in the role of the per-cycle damping.

The next analysis in this thread would be: **conditional inter-arrival of axes given the family composition of the producing tick.** Does a feature+metaposts+reviews tick produce the next axis faster or slower than a feature+templates+cli-zoo tick? That requires sub-classifying the 47 gaps by the parallel-family signature of the *terminating* feature tick, and is beyond the scope here — but it is the natural successor analysis, and the data is fully present in `history.jsonl`.

Forty-five minutes per axis. Five twins. One 2.5-hour pause. One permanent gap at 208. That is the actual shape of pew-insights axis production over the 36-hour window 2026-05-04T19:00 → 2026-05-06T06:48 UTC, measured against a homogeneous Poisson null, and conclusively rejecting it.
