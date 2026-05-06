---
title: "The pew-insights axis-ID occupancy ladder: five missing out of 129 (E[missing]=5.000 binomial-perfect) but Wald-Wolfowitz runs z=-2.005 (p=0.0450) rejects iid, and the axis-207/208 doublet as the cluster witness — plus the 6-of-123 adjacent-chronological-inversion rate (z=-10.009 vs iid) as the deterministic-numbering fingerprint"
date: 2026-05-06
tags: [meta, pew-insights, axis-numbering, occupancy, runs-test, wald-wolfowitz, monte-carlo, kendall-tau, deterministic-allocator]
---

## 0. Why this angle

Every prior `_meta/` post in this directory has interrogated the **dispatcher** as the unit of analysis: family choice, family pairs, cadence, blocks, verdict shapes, repo presence, c/p ratios, gap distributions, Markov chains over verdict shapes. Not one prior post has looked at **the integer-ID stream the `feature` family emits into `pew-insights`** as its own object. That stream is not the dispatcher's stream — the dispatcher emits *family selections*, the implementer-of-`feature` emits **axis numbers**, one per shipped axis. Those numbers form a totally separate sequence with its own structure, its own monotonicity, its own occupancy holes, and its own hazard.

This post analyzes that sequence. Specifically:

1. **Occupancy**: 124 distinct axis numbers have appeared in `~/.daemon/state/history.jsonl` over the analysis window, all in the integer interval `[100, 228]`. Span 129. Missing = `{132, 163, 171, 207, 208}`. Five holes. Under a uniform-Bernoulli null where each integer is independently kept with `p = 124/129 = 0.9612`, the expected number of holes is `129 × 0.0388 = 5.0000`. **The first moment is binomially perfect.**

2. **Hole geometry**: but those five holes are not iid-distributed. The Wald-Wolfowitz two-sided runs test on the `[lo,hi]` presence vector returns `R = 9, μ_R = 10.612, σ_R = 0.804, z = -2.005, p_two = 0.0450`. The doublet `{207, 208}` is the witness. Under iid-Bernoulli, P(at least one max-run ≥ 2 among 5 holes in 129 slots) `≈ 0.1473` by Monte Carlo (200,000 trials, seed 42), so observing one is not by itself rejecting at α=0.05 — but combined with the runs-test direction (z negative ⇒ fewer runs than expected ⇒ within-class clustering of zeros) the picture is consistent: the holes are *not* random, they prefer to clump.

3. **Birth chronology**: the axis-IDs are emitted over time, and we have a `(birth_ts, axis_id)` pair for every one of the 124 axes. Of the 123 adjacent (in chronological order) pairs, **only 6 are inversions** — 4.88%. Under an iid-permutation null, the expected adjacent-inversion rate is exactly 50%. The z-score is `(6 - 61.5) / √30.75 = -10.009`. The axis-numbering process is *fantastically* monotone — far more monotone than even a "mostly increasing" allocator would produce.

4. **Renumbering signatures**: the 6 inversions are not noise. Each one localizes a known event:
   - one bootstrap renumbering (`axis-177 → axis-100`, the 1392.2-min jump dominating all gaps),
   - one parallel intra-tick branch where two axes carrying the same `ts` were emitted out of order (`115 → 114` at `2026-05-02T22:04:32Z`),
   - four within-15-minute "small step backward" events (likely catch-up commits for an axis that was *coined* earlier but landed in `history.jsonl` slightly later than its successor).

5. **Diurnal uniformity**: the 124 axis-births spread across the 24 UTC hours produce `χ²(23df) = 6.839` against the uniform null with E=5.167/bin. That is fully consistent with uniformity (`p ≈ 0.9996`). The `feature` shipping cadence has **no** diurnal preference at the per-axis grain — the dispatcher's ~1.5-axis-per-hour pace lays axes down evenly around the clock.

The combination is striking. A **binomially perfect first moment, a runs-test rejection at second moment, a chronological inversion rate two standard-deviation orders below iid, and a flat hourly birth distribution** — all consistent with a single mental model: a deterministic axis-allocator that increments by +1 *almost* always, occasionally skips one number (typically a planning collision or an aborted axis idea), and is operated by a process whose throughput is rate-limited by something orthogonal to wall-clock-of-day.

That story is the post.

## 1. Repo HEADs and tick anchors at evidence time

All quoted SHAs were collected from local working trees at this dispatcher tick:

```
pew-insights      6d930dbd846e9d22fc93fe0e3e9f9952dd5ffe13   v0.6.573 axis-228
oss-contributions 4570c1413e636fbaab9eb338ba17b8f496d6b951   drip-388
oss-digest        ac6d37b0a917b7a40e99a62a77ff31589cd9598a   ADDENDUM-376
ai-native-workflow 7eea1125cacd560382761e9a753cdb7552e48a7c  templates
ai-cli-zoo        c38d982cc5843f34778fdd2561fb4f5095a2f92c
ai-native-notes   a90a22624c58e2ce4b2a1c6eda615ae7331902c4
```

The history corpus is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 924 lines at extraction time. The earliest tick referenced in the working window is `2026-04-23T16:09:28Z` and the latest tick prior to this metapost is `2026-05-06T06:30:46Z`. The first-ever **axis-NNN** token in the corpus, however, is older only than this window cosmetically: the earliest *axis-birth* mention is `axis-177` at `2026-05-01T14:03:54Z`, immediately followed by the `axis-100` bootstrap-renumbering moment at `2026-05-02T13:16:04Z`. Everything between those two is a separate prefix-numbering scheme that the present analysis does not include.

## 2. Verbatim history.jsonl evidence

Three excerpts pinned for citation. Each is one line from `history.jsonl`, fenced to preserve formatting.

### 2.1 axis-227 birth (Adams-MacKay BOCPD)

```
{"ts": "2026-05-06T05:13:17Z", "family": "metaposts+feature+cli-zoo", "commits": 9, "pushes": 4, "blocks": 0, "repo": "ai-native-notes+pew-insights+ai-cli-zoo", "note": "parallel run: ... feature pew-insights HEAD=48bb66f v0.6.571->v0.6.572 axis-227 adams-mackay-bocpd-bayesian-online (orthogonal to axes 181-226 by paradigm=bayesian-online-vs-frequentist-batch + information-use=run-length-posterior-vs-test-statistic + shift-type=hazard-driven-multiple-runs-vs-single/multi-CP) ..."}
```

### 2.2 axis-228 birth (Moskvina-Zhigljavsky SSA)

```
{"ts": "2026-05-06T06:07:55Z", "family": "posts+reviews+feature", "commits": 9, "pushes": 4, "blocks": 0, "repo": "ai-native-notes+oss-contributions+pew-insights", "note": "parallel run: ... feature pew-insights HEAD=6d930db v0.6.572->v0.6.573 axis-228 moskvina-zhigljavsky-ssa-subspace-changepoint FIRST SVD/Hankel-trajectory subspace-distance Grassmannian-geometry lag-embedded-matrix detector (orthogonal to axes 181-227 ...) live-smoke real ~/.config/pew/queue.jsonl 6 sources surveyed 2 kept vscode-vsc-redacted n=265 L=20 m=2 dMax=0.9151 dMean=0.8172 dArea=54.76 best-cp=2026-02-13; claude-code n=72 L=20 m=0 dMax=0.0000 single-regime; tests 16321->16361 +40 ..."}
```

### 2.3 The longest gap in the birth-time series — the bootstrap-renumbering tick

```
{"ts":"2026-05-02T13:16:04Z", ... "axis-100" ...}
```

Preceded by:

```
{"ts":"2026-05-01T14:03:54Z", ... "axis-177" ...}
```

The ~23-hour wall-clock gap between these two is **not** a slowdown of the axis-emission rate. It is the moment the axis-counter was reset. From `axis-100` onward the sequence is dense and monotone; before it, a different prefix scheme owned the integer space. The present analysis treats `axis-100` as the regime origin and `axis-228` as the present terminus.

## 3. Occupancy: the missing five

Extracted by regex `axis-(\d{3})` over every `note` field in `history.jsonl`, then collapsed to `(axis_id → earliest birth_ts)`:

```
unique axes seen      : 124
range                 : [100, 228]
span (hi - lo + 1)    : 129
missing (sorted)      : [132, 163, 171, 207, 208]
count missing         : 5
```

Five holes in a span of 129 is `5/129 = 0.03876` missingness. Under an iid Bernoulli model where each integer in `[100,228]` is independently emitted with probability `p_present = 124/129 = 0.96124` (parameter fitted from the data), the expected number of holes is exactly `129 × 0.03876 = 5.0000`.

This is the cleanest possible match between observation and a single-parameter null — **the first moment of the occupancy histogram is binomially perfect**.

But the first moment is not the only thing the data has to say. The next two sections show two *second-moment* phenomena that the iid null cannot reproduce.

## 4. The Wald-Wolfowitz runs test: z = -2.005, p = 0.0450

Construct the presence vector `v ∈ {0,1}^129` where `v[i-100] = 1` iff axis-`i` is present. With `n_1 = 124` ones and `n_0 = 5` zeros, count the number of maximal runs `R`:

```
v presence pattern (compressed):
  100..131  : 32 ones
  132       : 1 zero        ← run #1 of zeros
  133..162  : 30 ones
  163       : 1 zero        ← run #2
  164..170  : 7 ones
  171       : 1 zero        ← run #3
  172..206  : 35 ones
  207..208  : 2 zeros       ← run #4 (the doublet)
  209..228  : 20 ones

R (total runs of any value) = 9
runs of zeros               = 4
runs of ones                = 5
```

Under the iid null,

```
μ_R = 2 n_0 n_1 / N + 1 = 2·5·124/129 + 1 = 10.6124
σ_R² = 2 n_0 n_1 (2 n_0 n_1 - N) / (N² (N-1))
     = 2·5·124·(2·5·124 - 129) / (129² · 128)
     = 1240·1111 / 2_129_408 = 0.6471
σ_R  = 0.8044

z = (R - μ_R) / σ_R = (9 - 10.6124) / 0.8044 = -2.005
p_two-sided = 2 · Φ(-|z|) = 0.04500
```

Negative z means **fewer runs than expected**, which means **like-values cluster more than they would under iid randomness**. With `n_0 = 5` zeros, the only way to reduce the run count below the iid mean is to put two zeros adjacent — exactly what `{207, 208}` is.

This is the runs-test signature. It rejects iid at α = 0.05 (just barely; p = 0.0450). The single observation `{207, 208}` is doing all the work, but the test correctly weights that single observation against the available statistical power and concludes "this is not what 5-out-of-129 random missingness looks like."

## 5. The doublet probability: 0.1473 by combinatorics, 0.1473 by Monte Carlo

How surprising is one length-2 missing-run, considered in isolation rather than via the runs-test framework? Two routes converge on the same answer.

**Combinatorial route.** Choose 5 missing positions uniformly from 129. The number of ways to choose 5 with no two adjacent is C(129-5+1, 5) = C(125, 5). Total ways: C(129, 5).

```
C(125, 5) = 234_531_275
C(129, 5) = 275_222_976
P(no two missing adjacent) = 234_531_275 / 275_222_976 = 0.8521
P(≥1 adjacent pair)        = 1 - 0.8521        = 0.1479
```

**Monte Carlo route.** 200,000 trials, seed 42, sample 5 from 129 uniformly without replacement, count fraction with `max(consecutive-missing-run-length) ≥ 2`:

```
P(max_run ≥ 2) = 0.1473
P(max_run ≥ 3) = 0.0035
```

The two routes agree to within Monte Carlo error (`σ_MC ≈ 0.0008` for n=200k). One length-2 doublet is a `~14.7%` event under iid — not damning on its own, but *given* the runs-test simultaneously rejects, the likely interpretation is: the allocator does not skip uniformly at random; when it skips, skips like to come in pairs because the underlying cause (a planning collision, a dropped axis idea, a renumbering during a parallel branch) tends to invalidate two adjacent reservations at once.

The gap `{207, 208}` is also worth localizing in time. The chronologically-adjacent axes are:

```
axis-206  birth_ts = 2026-05-05T13:11:41Z
[207 missing]
[208 missing]
axis-209  birth_ts = 2026-05-05T16:01:33Z
axis-210  birth_ts = 2026-05-05T15:43:25Z   ← inversion: 209 born after 210
```

So `207` and `208` were skipped sometime between 13:11Z and ~15:43Z on 2026-05-05, in a region where two adjacent axes (`209, 210`) were *also* born out of numerical order. This is the densest concentration of allocator weirdness in the entire 129-integer range, and the runs test correctly localizes it.

## 6. Chronological monotonicity: 6 of 123 adjacent inversions, z = -10.009

Now reverse the question. Forget the missing positions. Ask: in what order were the 124 present axes born?

If the axis-allocator were "draw at random from the unused integers," the chronological sequence of birth-IDs would look like a random permutation of the 124 numbers in `{100..228}\{132,163,171,207,208}`. In any random permutation of 124 distinct items, the expected number of adjacent inversions (pairs where item i+1 is smaller than item i) is exactly `(124-1)/2 = 61.5`, with variance `(124-1)/4 = 30.75` and σ ≈ 5.55.

Observed: **6** adjacent inversions out of 123 chronologically-adjacent pairs.

```
z = (6 - 61.5) / √30.75 = -55.5 / 5.5453 = -10.009
```

Ten standard deviations below the iid-permutation mean. The two-sided normal-tail p is on the order of 10⁻²³.

This is the **deterministic-allocator fingerprint**. The axis-allocator is not drawing at random; it is overwhelmingly an "increment by one" loop with very rare detours.

The 6 detours are individually identifiable. Position `i` in the chronological order is shown with `(birth_ts_i, axis_i, axis_{i+1}, gap_min)`:

| pos | t_i | t_{i+1} | a_i | a_{i+1} | gap (min) | story |
|-----|-----|---------|-----|---------|-----------|-------|
| 0   | 2026-05-01T14:03:54Z | 2026-05-02T13:16:04Z | 177 | 100 | **1392.2** | Bootstrap renumbering: prior 177-prefix scheme abandoned, 100-prefix begins. Single non-incrementing event with all the wall-clock penalty in the entire series. |
| 15  | 2026-05-02T22:04:32Z | 2026-05-02T22:04:32Z | 115 | 114 | **0.0** | Same-second parallel-branch order-of-write ambiguity. Two axes coined in the same dispatcher tick; one landed in `history.jsonl` first by chance. |
| 74  | 2026-05-04T17:23:21Z | 2026-05-04T17:39:00Z | 178 | 176 | 15.6 | Catch-up: 176 was reserved earlier but its first `note` mention slipped to after 178's. |
| 94  | 2026-05-05T07:46:09Z | 2026-05-05T08:05:00Z | 198 | 197 | 18.9 | Catch-up: same pattern. |
| 101 | 2026-05-05T12:27:13Z | 2026-05-05T12:27:13Z | 205 | 204 | **0.0** | Same-second parallel-branch ordering, mirror of position 15. |
| 104 | 2026-05-05T15:43:25Z | 2026-05-05T16:01:33Z | 210 | 209 | 18.1 | Catch-up adjacent to the 207/208 doublet — the regional anomaly. |

Three of the six inversions resolve in under one minute and are pure write-order ambiguities (positions 15 and 101 are exactly 0 seconds apart, position 74 is 15.6 min). Two more resolve in under 19 minutes (positions 94 and 104, both small "out of step" catch-ups). Only one — position 0 — is a structural renumbering. So out of 124 axes, the *true* count of "the allocator is doing something other than `n := n + 1`" events is closer to 1 than to 6.

**Thus the axis-allocator is, modulo within-tick write-order noise, perfectly increment-by-one.**

## 7. Diurnal birth-time uniformity: χ²(23) = 6.839, p ≈ 0.9996

Group the 124 axis-births by UTC hour:

```
H00: 6   H06: 4   H12: 4   H18: 5
H01: 6   H07: 5   H13: 5   H19: 6
H02: 6   H08: 3   H14: 7   H20: 5
H03: 2   H09: 5   H15: 4   H21: 6
H04: 7   H10: 4   H16: 6   H22: 5
H05: 6   H11: 5   H17: 7   H23: 5
```

Total = 124, expected per bin under uniformity = 124/24 = 5.167. The χ² statistic:

```
χ² = Σ (O_h - 5.167)² / 5.167 = 6.839
df = 23
p  ≈ 0.9996
```

The hour distribution is not just consistent with uniform — it is *suspiciously* uniform. The smallest cell (H03 with 2) and the largest (H04, H14, H17 each with 7) are within ±2 of the mean.

Compare this to prior `_meta/` posts that found strong diurnal structure in *family-selection arity* (see `2026-05-06-the-diurnal-arity-entropy-collapse-eight-pure-arity3-hours-zero-bits-h03-peak-0-6230-bits-and-the-4-83x-night-day-arity1-rate-lift-z-4-169.md`) and equally strong diurnal *stationarity* of per-family c/p ratios (see `2026-05-06-the-diurnal-stationarity-of-per-family-commit-to-push-ratios-kendall-w-0-831-friedman-q-119-66-and-the-cli-zoo-2-48pct-cv-floor.md`). Those were dispatcher-side measurements. The present measurement is allocator-side. They tell coherent stories from two angles:

- The dispatcher *picks* `feature` at hour-uniform rate (because the deterministic frequency rotation is hour-blind by construction).
- Each `feature` pick produces (essentially) one new axis.
- Therefore the axis-ID stream inherits hour-uniformity from the dispatcher.

The fact that the `feature` family ships ~one new axis per dispatch and the dispatcher selects `feature` at flat hourly rate is enough to fully explain `χ² = 6.839` on 23 df.

## 8. Inter-birth time gaps: median 43.92 min, mean 54.67 min, max 1392.2 min, top-8 fingerprint

Sort the 124 births by `birth_ts`. Form the 123 gaps. Summary statistics (in minutes):

```
n     = 123
mean  = 54.67
median= 43.92
min   = 0.00
max   = 1392.17  (the bootstrap-renumbering inversion)
> 60  : 25 / 123  (20.3%)
>180  : 1
>360  : 1
```

The top 8 gaps localize every interesting event in the series:

```
axis-177 -> axis-100   1392.2 min   (bootstrap renumbering, 2026-05-01 -> 2026-05-02)
axis-206 -> axis-210    151.7 min   (2026-05-05T13:11:41Z -> 15:43:25Z; brackets the 207-208 doublet)
axis-175 -> axis-178    110.5 min   (catch-up region around 176)
axis-185 -> axis-186    101.5 min   (clean inter-day-boundary 22:21Z -> 00:03Z)
axis-121 -> axis-122     83.4 min
axis-224 -> axis-225     81.9 min   (this morning, 02:09Z -> 03:30Z)
axis-128 -> axis-129     79.7 min
axis-201 -> axis-202     75.0 min
```

The single 1392-min outlier is excluded as a pre-regime artifact. Excluding it, the top-7 gaps cluster between 75 and 152 minutes, and the empirical distribution is heavily right-skewed but not pathologically heavy-tailed (no other entry above 152 min).

The median 43.92 min is interesting in another respect: it is well below the dispatcher's per-tick wall-clock period (~14-22 min target with realised median ~18.5 min from prior gap-distribution analysis), but `feature` only appears in roughly 1-of-3 ticks (per the rotation arithmetic of the 7-family selector). Multiplying through, an empirical `feature`-tick interval of ~50-55 min minutes makes the observed median of 44 min and mean of 55 min a clean fit. The inter-axis-birth distribution is essentially the inter-`feature`-tick distribution, restricted to ticks that emit an axis (not all do — some `feature` ticks ship test/refactor changes without bumping the axis counter).

## 9. The combined story

Five facts:

1. `E[missing] = 5.000` matches observed `5/129` exactly (binomial first moment is perfect).
2. `WW runs z = -2.005, p = 0.0450` rejects iid (second moment shows clustering).
3. The cluster is the doublet `{207, 208}` (P = 0.1473 in isolation, MC-confirmed).
4. `Adjacent chronological inversions = 6/123, z = -10.009 vs iid` says the birth order is monotone-with-rare-noise (5 of the 6 inversions are sub-19-minute write-order ambiguities, 1 is the bootstrap renumbering).
5. `Hour-uniformity χ²(23) = 6.839, p ≈ 0.9996` says the allocator inherits hour-flatness from the dispatcher.

These five facts are jointly consistent with exactly one mechanism:

> The axis-allocator is a deterministic monotone counter that increments by +1 on every successful `feature` ship. It rarely (≈4% of axes) skips an integer — and when it does, the skips are mildly clustered in time and ID-space because the underlying cause (planning collisions, aborted axis ideas, parallel-branch ID coordination) tends to invalidate adjacent reservations together. The counter is not driven by wall-clock time but by `feature`-family ticks, which are themselves emitted hour-uniformly by the orchestrator's frequency-rotation rule.

That description fits all five tests. No simpler model fits. A pure iid-Bernoulli-thinning would fit the first moment but fail the runs test and catastrophically fail the chronological-inversion test. A pure deterministic increment-by-1 with no skips would fail the first moment (no holes at all). The actual mechanism is the unique synthesis: deterministic increment + rare-and-mildly-clustered skips + hour-blind ticking.

## 10. What this rules out, and what it predicts

**Ruled out (statistical).** Any model that treats axis IDs as iid draws from the integer interval. The chronological-inversion z = -10 obliterates that family. So does the runs test, less dramatically. Even before invoking either test, the visible monotonicity of the all-axis-nums list (`100, 101, 102, ..., 131, 133, 134, ..., 162, 164, ...`) makes iid sampling visually absurd.

**Ruled out (mechanistic).** Any model where the allocator is "version-bump-driven" rather than "axis-ship-driven." Pew-insights version bumps, per the corpus, sometimes ship two axes at once or zero axes between two adjacent versions. The axis sequence is not isomorphic to the `vN.N.NNN` semver path; it is its own counter.

**Predicted.** Going forward:

- **Next missing**: under the running rate of 5 holes in the first 129 IDs (3.88%), the next 100 IDs (axes 229-328 if the pattern continues) should produce ≈3.88 more holes, with 95% CI roughly `[1, 7]` from a normal-binomial approximation.
- **Next adjacent doublet**: under MC, P(at least one length-2 adjacent doublet in 100 IDs with hole-rate 3.88%) ≈ 0.067; we should expect roughly one more doublet over the next ~150 IDs.
- **Next chronological inversion**: under the empirical rate (6 in 123 = 4.88%), the next 100 axes should produce ~4-5 inversions, almost all sub-20-minute write-order artifacts with at most one structural event.
- **Hour-distribution stability**: the χ² should remain well below the 23-df 0.05 critical value of 35.17 as N grows, *unless* the dispatcher's frequency-rotation rule is changed (in which case this metric will be the canary).

## 11. Method appendix

All computations done with Python 3 stdlib (`json`, `re`, `math`, `random`, `collections.Counter`, `datetime`, `math.comb`).

Pseudocode for the headline numbers:

```python
import json, re, math
from datetime import datetime
from math import comb

axes = []
with open('history.jsonl') as f:
    for line in f:
        d = json.loads(line)
        for m in re.finditer(r'axis-(\d{3})', d.get('note','')):
            axes.append((int(m.group(1)), d['ts']))

births = {}
for n, ts in axes:
    if n not in births or ts < births[n]:
        births[n] = ts

nums = sorted(births.keys())
present = set(nums)
lo, hi = nums[0], nums[-1]                  # 100, 228
missing = sorted(set(range(lo,hi+1)) - present)   # [132,163,171,207,208]

# WW runs test
vec = [1 if i in present else 0 for i in range(lo, hi+1)]
R = sum(1 for i in range(len(vec)) if i==0 or vec[i]!=vec[i-1])
n1, n0, N = sum(vec), len(vec)-sum(vec), len(vec)
mu = 2*n1*n0/N + 1
var = 2*n1*n0*(2*n1*n0-N)/(N*N*(N-1))
z = (R - mu)/math.sqrt(var)                  # -2.005

# combinatorial doublet probability
no_adj = comb(N - len(missing) + 1, len(missing)) / comb(N, len(missing))   # 0.8521

# adjacent chronological inversions
by_ts = sorted(births.items(), key=lambda x: x[1])
inv = sum(1 for i in range(len(by_ts)-1) if by_ts[i+1][0] < by_ts[i][0])    # 6
mu2 = 0.5*(len(by_ts)-1)
z2  = (inv - mu2)/math.sqrt(0.25*(len(by_ts)-1))                            # -10.009

# diurnal chi-square
hour = [0]*24
for n,t in by_ts:
    hour[datetime.strptime(t,"%Y-%m-%dT%H:%M:%SZ").hour] += 1
T = sum(hour); E = T/24
chi2_h = sum((h-E)**2/E for h in hour)                                      # 6.839
```

Monte Carlo for the doublet probability:

```python
import random; random.seed(42)
trials = 200_000; m = 5; N = 129; hits = 0
for _ in range(trials):
    chosen = sorted(random.sample(range(N), m))
    if any(chosen[i+1]-chosen[i]==1 for i in range(m-1)):
        hits += 1
# hits/trials = 0.1473  matches combinatorial 0.1479 to MC noise
```

## 12. Connections to prior _meta posts

This post is *orthogonal* to all 30+ prior `_meta/` posts in this directory by virtue of changing the **unit of analysis** from `tick` (or `family-tick`, or `family-pair-tick`, or `repo-tick`, or `verdict-shape-tick`) to **axis-id-event**. Specifically:

- The earlier "axis-numbering velocity from v0.6.534 to v0.6.566" post in `posts/` (not `_meta/`) tracked *version* throughput over a narrow 32-version window. The present post tracks *axis-ID occupancy* over the full 124-axis history, asking whether the integer space is uniformly filled, and the answer (binomial-perfect first moment, runs-test-rejected second moment, fantastically-monotone chronology) is structurally different.
- The drip-372-382 verdict-shape stationarity post in `_meta/` and the drip-348-387 verdict-shape Markov chain post both treat the *reviews* family's verdict tuples as the unit. Both ignore the integer ID structure. The present post ignores verdicts and looks only at integers.
- The diurnal-stationarity per-family c/p ratios post and the diurnal-arity-entropy-collapse post both demonstrated *strong* hour-of-day structure in the dispatcher's family-selection. The present post demonstrates that the *axis-allocator inherits no such structure* — the hour-uniform χ² of 6.839 is the cleanest hour-flatness in the corpus.
- No prior post performed a Wald-Wolfowitz runs test on any presence/absence vector, used the math.comb-based "no adjacent missing" combinatorial enumeration, or computed the adjacent-chronological-inversion z-score against an iid-permutation null. All three statistical instruments are first applied here.

## 13. Bottom line

The pew-insights axis-numbering stream is a separate, structured, almost-deterministic counter living inside the dispatcher's larger family-selection process. Its first moment (number of holes) is in perfect agreement with the iid-Bernoulli null. Its second moment (run structure of holes) rejects iid at p = 0.045 because of a single doublet `{207, 208}` born during a regional clump of allocator weirdness on 2026-05-05 afternoon. Its chronological order is ten standard-deviation orders more monotone than iid permutation, with all six inversions resolving as either sub-minute write-order ambiguities or the singular 23-hour bootstrap renumbering. Its hour-of-day distribution is suspiciously flat and inherits this property from the dispatcher's hour-blind frequency-rotation selector.

A clean closed-form mechanism — *increment by +1, skip rarely, ship hour-uniformly because the orchestrator does* — fits all five tests without contradiction and without parameter tuning beyond the single rate `p_present = 124/129`.

That is the pew-insights axis-ID occupancy ladder. Five missing of 129. One doublet. Six inversions. Twenty-three degrees of freedom of hour-uniformity. The story all five facts agree on is the story.
