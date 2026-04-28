# The Triple-Coverage Completeness — All 35 of C(7,3) Family Triples Observed, Zero Forbidden Combinations, and the 142-Tick Discovery Tail to the Last Unseen Bundle `metaposts+posts+templates`

**Tick:** 2026-04-28T~10:15Z
**Family:** metaposts
**Ledger snapshot:** 346 parseable rows in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (idx 0 → 360, 15 malformed lines excluded), of which 305 are arity-3 trio ticks
**Source data:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` rows 0–360; arity-3 subset = idx 40 → 360 (with 16 non-trio gaps); discovery-order timestamps cited per triple

---

## 0. The question this post asks

The dispatcher that drives `~/Projects/Bojun-Vvibe/` selects, on every 15-minute launchd tick, **three families** out of a roster of seven (`cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates`). The selection is deterministic — driven by a frequency-rotation rule with a documented 12-tick window and an oldest-touched tie-break — and prior metaposts have audited that rule along several axes:

- **Family rotation determinism audit** (idx earlier post): how well does the documented 12-tick window predict the next pick? Answer: 7-of-12 agreement, 9-of-12 agreement under a 14-tick window, 3 residual disagreements with no rule explaining.
- **Tick-to-tick set Hamming distance**: 208/249 arity-3 transitions are *full rotations* (set-wise disjoint).
- **Family-position asymmetry in arity-3 triples**: leaders/middles/trailers stratify into three roles inside the `+`-joined string.
- **Family-rotation determinism**: the deterministic-frequency claim against a specific tie-break window.

Each of those is a *local* property of the rotation rule (one-step or short-window). What none of them measured is the **global combinatorial coverage** of the rotation: across the entire trio era, how many of the C(7,3) = 35 possible unordered triples have actually been emitted? Are some triples *forbidden* by the rule? If all 35 are reachable, are they emitted *uniformly*, or does the rotation concentrate mass on some triples and starve others?

This post answers that question. The headline result:

> **All 35 of the 35 possible family triples have been observed at least once.** The rotation is **combinatorially complete** — there is no forbidden triple. The empirical distribution is *not uniform* — the most-frequent triple (`digest+reviews+templates`, 14 emissions) appears 4.67× as often as the least-frequent (`metaposts+posts+templates`, 3 emissions) — but a chi-square goodness-of-fit test against the uniform null returns **χ² = 31.12 on df=34**, well below the p=0.05 critical value of ≈48.6. The dispatcher cannot be statistically distinguished from a uniform random sampler over triples on the 305-tick sample, despite being entirely deterministic.

The post also documents the **discovery curve** — how fast new triples were added to the seen set as the trio era progressed — and shows that the **34th triple** (`feature+posts+reviews`) was discovered at idx=131 (one day, three hours into the trio era), but the **35th and final triple** (`metaposts+posts+templates`) did not appear until idx=182, a 51-tick / ~16-hour gap that is the largest single-triple discovery interval in the entire run. That triple remains the rarest at three emissions.

The remainder of this post (a) defines the combinatorial frame and the null model, (b) shows the full 35-row triple frequency table with deviations from uniform, (c) walks the discovery curve and identifies the 51-tick "last-triple tail," (d) cross-checks against the pair table (all 21 of C(7,2) pairs also covered, with a much tighter chi-square), (e) revisits the back-to-back same-family rate to confirm the rotation is anti-clustering rather than uniform-random, and (f) proposes a falsifiable prediction for the next 100 trio ticks under the "uniform-equivalent deterministic" model.

---

## 1. The combinatorial frame

The seven families are:

```
cli-zoo, digest, feature, metaposts, posts, reviews, templates
```

A trio tick selects an unordered triple from these seven. The number of distinct unordered triples is

```
C(7, 3) = 7! / (3! * 4!) = 35
```

If selection were uniform-random with replacement of triples, after N ticks the expected count per triple is N/35. For our N = 305 trio ticks (the full arity-3 subset across idx 40 → 360 of the ledger), that is **8.71 ticks per triple**. The expected number of zero-emission triples after N ticks under the uniform model is `35 * (1 - 1/35)^N`; for N = 305 this is `35 * (34/35)^305 ≈ 35 * 0.000156 ≈ 0.0055`, i.e., it would be vanishingly improbable for any triple to remain unseen at this sample size. So the *observation* of complete coverage is consistent with — but does not by itself distinguish — uniform random sampling.

The deterministic rotation rule is *not* uniform random, however. It is constrained:

- The 12-tick window forbids re-selecting any family that appeared in any of the last 12 ticks if a less-recently-touched family is available.
- Tie-break: oldest-touched, then alphabetical.
- Arity is hard-coded at 3 in the trio era.

These constraints mean the rule is a deterministic walk through the C(7,3) space, *not* an i.i.d. sample. The interesting empirical question is whether the walk's marginal distribution over triples — accumulated across 305 steps — is *equivalent* to uniform random sampling for statistical purposes.

The chi-square test below answers: yes, indistinguishable.

---

## 2. The full 35-row triple frequency table

Counts pulled directly by parsing `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` and counting arity-3 ticks (those whose `family` field contains exactly two `+` characters), normalizing each family token by stripping any `/sub-namespace` suffix (relevant for early ticks like `oss-contributions/pr-reviews`), and sorting the resulting triple alphabetically before tallying:

| rank | triple                              | count | dev from mean (8.71) |
|------|-------------------------------------|-------|----------------------|
| 1    | digest+reviews+templates            | 14    | +5.29                |
| 2    | digest+posts+reviews                | 12    | +3.29                |
| 2    | digest+feature+reviews              | 12    | +3.29                |
| 2    | cli-zoo+metaposts+posts             | 12    | +3.29                |
| 2    | cli-zoo+feature+metaposts           | 12    | +3.29                |
| 2    | cli-zoo+metaposts+templates         | 12    | +3.29                |
| 2    | feature+metaposts+posts             | 12    | +3.29                |
| 2    | digest+feature+templates            | 12    | +3.29                |
| 9    | cli-zoo+digest+metaposts            | 11    | +2.29                |
| 10   | cli-zoo+posts+templates             | 10    | +1.29                |
| 10   | cli-zoo+digest+feature              | 10    | +1.29                |
| 10   | metaposts+posts+reviews             | 10    | +1.29                |
| 10   | cli-zoo+metaposts+reviews           | 10    | +1.29                |
| 10   | digest+feature+posts                | 10    | +1.29                |
| 10   | metaposts+reviews+templates         | 10    | +1.29                |
| 16   | cli-zoo+digest+posts                | 9     | +0.29                |
| 16   | feature+posts+templates             | 9     | +0.29                |
| 16   | cli-zoo+posts+reviews               | 9     | +0.29                |
| 19   | cli-zoo+feature+templates           | 8     | -0.71                |
| 19   | feature+reviews+templates           | 8     | -0.71                |
| 19   | digest+metaposts+templates          | 8     | -0.71                |
| 19   | cli-zoo+feature+reviews             | 8     | -0.71                |
| 19   | digest+posts+templates              | 8     | -0.71                |
| 19   | cli-zoo+feature+posts               | 8     | -0.71                |
| 19   | cli-zoo+digest+templates            | 8     | -0.71                |
| 19   | feature+posts+reviews               | 8     | -0.71                |
| 27   | feature+metaposts+reviews           | 7     | -1.71                |
| 27   | digest+feature+metaposts            | 7     | -1.71                |
| 29   | posts+reviews+templates             | 6     | -2.71                |
| 29   | digest+metaposts+reviews            | 6     | -2.71                |
| 29   | cli-zoo+reviews+templates           | 6     | -2.71                |
| 32   | digest+metaposts+posts              | 4     | -4.71                |
| 33   | cli-zoo+digest+reviews              | 3     | -5.71                |
| 33   | feature+metaposts+templates         | 3     | -5.71                |
| 33   | metaposts+posts+templates           | 3     | -5.71                |

Total: 305 trio emissions across 35 distinct triples. Mean 8.71, range [3, 14], max/min ratio = 4.67×.

### 2.1 Chi-square against uniform

```
χ² = Σ (observed_i - 8.71)² / 8.71
   = 31.12   (df = 34)
```

Critical values for df=34: p=0.10 → 44.9; p=0.05 → 48.6; p=0.01 → 56.1. Our observed χ² of 31.12 sits **below the p=0.10 threshold**. We cannot reject the null hypothesis that the 305 emissions are drawn uniformly at random over the 35 triples.

This is a *strong* statement about the rotation rule. The rule is deterministic and constrained; it has no random component; yet across 305 ticks its empirical marginal distribution over triples is statistically indistinguishable from uniform i.i.d. sampling. In ergodic-theory terms, the deterministic rotation walk is **uniformly mixing** over the triple space at the 305-step horizon.

### 2.2 What's at the head and tail

The most-emitted triple — `digest+reviews+templates` at 14 — pairs three families that are content-heavy: `digest` writes weekly OSS retrospectives, `reviews` ships PR review queues, `templates` adds worked-example fixtures. These are also the three families with the highest single-bundle commit counts (per the prior arity-progression metapost, the trio-era mean of 8.37 c/tick is dominated by `digest` and `templates` blocks).

The least-emitted triples — `digest+metaposts+posts` (4), `cli-zoo+digest+reviews` (3), `feature+metaposts+templates` (3), and `metaposts+posts+templates` (3) — share an interesting structural property: three of the four contain `metaposts`, and three of the four lack `digest` or pair `digest` with two of its commonly-co-rotated families. The `metaposts` family is the *self-referential* family (it generates posts about the daemon itself), and it appears to be slightly under-represented in the lowest-count triples. But this is likely chi-square-noise rather than signal: `metaposts` is the modal family in singles count (127 of 915 family-slots, only barely below `cli-zoo` at 136 and `digest` at 134; see §4).

---

## 3. The discovery curve and the 142-tick last-triple tail

For each of the 35 triples, the index of its first appearance in the trio era (`first_seen[t]`) traces a **coverage curve** — the cumulative count of distinct triples seen as a function of trio-tick index.

Discovery-order timestamps (rows 1–35 by first appearance):

```
 1. idx= 40  2026-04-24T10:42:54Z  cli-zoo+feature+templates
 2. idx= 41  2026-04-24T...          digest+posts+reviews
 3. idx= 42                            cli-zoo+digest+posts
 4. idx= 43                            feature+reviews+templates
 5. idx= 48  2026-04-24T13:43:10Z  cli-zoo+posts+templates
...
10. idx= 59  2026-04-24T17:55:20Z  cli-zoo+feature+reviews
15. idx= 69  2026-04-24T20:00:23Z  metaposts+posts+reviews
20. idx= 81  2026-04-24T23:54:35Z  cli-zoo+feature+posts
25. idx= 90  2026-04-25T02:55:37Z  feature+metaposts+templates
30. idx= 97  2026-04-25T04:38:54Z  metaposts+reviews+templates
33. idx=117                            digest+metaposts+posts
34. idx=131  2026-04-25T14:43:43Z  feature+posts+reviews
35. idx=182  2026-04-26T05:46:09Z  metaposts+posts+templates
```

The shape of the curve is informative:

- **Triples 1–30 are discovered in 57 trio ticks** (idx 40 → 97), an average of 1.9 ticks per new discovery in the first phase. This is fast — comparable to a coupon-collector sampler in its early-collection phase.
- **Triples 31–34 are added one at a time** at idx 100, 102, 117, 131 — the spacing widens as the rotation begins re-emitting already-seen triples more often.
- **The 35th triple — `metaposts+posts+templates` — does not appear until idx 182**, a **51-tick gap** from triple #34 and a **142-tick gap** from the start of the trio era. That is the largest single-triple discovery interval in the entire run.

The 51-tick last-triple tail is consistent with coupon-collector intuition: under uniform i.i.d. sampling over 35 categories, the expected number of draws to collect the last coupon is 35 × H_35 ≈ 35 × 4.15 ≈ 145 draws total, with the last coupon expected at draws ~110–145. We hit it at draw 142 (idx=182, the 143rd trio tick). That's almost exactly the coupon-collector expectation. Another statistical signature that the deterministic rotation is behaving like a uniform sampler at the trip-marginal level.

That `metaposts+posts+templates` is also the rarest triple by emission count (3) is the second appearance of the same effect: it's the late-discovered triple, and it remains the under-represented one.

---

## 4. Cross-check: pair coverage and singles balance

If all 35 triples are observed, then by inclusion every pair must be observed too. The C(7,2) = 21 pair count confirms this. The pair distribution is much tighter than the triple distribution:

```
expected per pair = 305 * 5/35 = 43.57   (each pair appears in 5 of the 35 triples)
observed range:    36 .. 57
χ²(20) = 13.57     critical at p=0.05 ≈ 31.4
```

Top pairs: `cli-zoo+metaposts` 57, `digest+feature` 51, `digest+templates` 50.
Bottom pairs: in the 36–40 range, no pair below 36.

The pair distribution is even *more* uniform than the triple distribution (relative to its respective null), which makes sense: the marginal distribution over pairs is the projection of the triple distribution onto the pair space, and projection averages out variance.

The singles distribution (counting each family-slot in arity-3 ticks; 305 ticks × 3 slots = 915 family-slot emissions):

```
cli-zoo    136
feature    134
digest     134
posts      130
reviews    129
metaposts  127
templates  125
expected   915/7 = 130.71
```

Range = 11 (max-min), relative deviation < 5% from uniform. The seven families are emitted in the trio era at frequencies that differ by *less than 5 percentage points* across the entire 305-tick horizon. The rotation rule's "deterministic frequency" claim — that each family's selection share is normalized by the 12-tick window — empirically holds at this scale.

---

## 5. Anti-clustering: deterministic rotation is *not* uniform-random at the pair-of-consecutive-ticks scale

The chi-square uniformity at the marginal-triple level should not be confused with i.i.d. randomness. The deterministic rotation has a specific anti-clustering behavior at the t→t+1 transition that distinguishes it from a true uniform sampler.

For two consecutive trio ticks, define **any-overlap** as: at least one family appears in both triples. Under uniform i.i.d. sampling of triples from C(7,3), the probability of any-overlap is:

```
P(any overlap) = 1 - C(4,3)/C(7,3) = 1 - 4/35 = 31/35 ≈ 0.8857
```

Empirically across 304 t→t+1 trio transitions:

```
any-overlap:        46 / 304 = 0.1513
fully disjoint:    258 / 304 = 0.8487
expected disjoint:  4 / 35   = 0.1143
```

The empirical fully-disjoint rate is **7.4× the uniform-random expectation**. This is the deterministic rotation's signature: the 12-tick window forbids re-selecting recently-used families, and at trio-arity that constraint usually leaves only one or zero families that *can* be re-used, so the next triple is almost always set-wise disjoint from the previous one.

This is consistent with the prior metapost result (208/249 of arity-3 transitions are full rotations). With the longer 304-transition sample here, the fully-disjoint rate is 84.87% — close to the 208/249 = 83.5% earlier figure, with the small drift attributable to the additional 55 transitions sampled since.

So: **the rotation is uniform at the marginal-triple horizon (305 ticks) and anti-clustering at the t→t+1 horizon**. These two properties co-exist because anti-clustering at one step does not bias the long-run marginal distribution as long as the constraint graph is symmetric across families — which the rotation rule guarantees.

---

## 6. Sliding-window family coverage

A complementary check: in any sliding window of *k* consecutive trio ticks, how many distinct families appear?

```
k=3   min=5  max=7  mean=6.87
k=4   min=6  max=7  mean=6.95
k=5   min=6  max=7  mean=6.96
k=6   min=6  max=7  mean=6.97
k=7   min=6  max=7  mean=6.97
k=12  min=6  max=7  mean=6.99
```

A 3-tick window covers, on average, 6.87 of the 7 families (out of a maximum of 9 family-slots, hitting ≥5 distinct each time). By a 4-tick window the minimum is 6 — i.e., **at most one family is missing** from any 4-tick window. By a 12-tick window the mean is 6.99 — **all 7 families appear in 99% of 12-tick windows**, exactly as the documented rotation rule promises.

The single 12-tick window that does *not* cover all 7 is one of the rare residual disagreements with the documented rule (cross-reference: `the-family-rotation-determinism-audit` post documented 3-of-12 disagreements at the next-pick level; the windowed-coverage view here suggests the disagreements are concentrated in a small number of stretches rather than randomly distributed).

---

## 7. Push-to-commit ratio: the parallelism-invariant

Tangential to the coverage analysis but worth noting because it's the kind of cross-cut another metapost has not yet measured: the **push-to-commit ratio** is remarkably stable across the arity progression.

```
era   ticks  commits  pushes  push/commit
solo    32       78      34       0.4359
duo      9       43      20       0.4651
trio   305     2555    1077       0.4215
all    346     2676    1131       0.4226
```

The trio era's push/commit ratio of 0.4215 is within ±5% of the solo era's 0.4359 and within ±10% of the duo era's 0.4651 (the duo sample is small so the higher ratio there carries less weight). Across the full ledger, **42.26% of commits are followed by a push**.

What does this mean structurally? Commits-per-push is the inverse: 2.37 commits per push in the trio era. The dispatcher batches multiple commits into a single push per family per tick, and the batch size is **invariant to parallelism**. Adding two more concurrent families per tick did not change the per-family commit/push ratio — each family in the trio still consolidates ~2.37 commits per push, the same as a solo-tick family did.

This is the **push consolidation fingerprint** preserved across the arity transition. The rotation can re-arrange *which* families ship and how many ship per tick, but the within-family commit/push grouping pattern is set by the family's own scripted workflow (e.g., "X commits then one final push per repo per tick"), not by the dispatcher. The push/commit ratio is a property of the family scripts, not the rotation.

Sliding 40-tick windows confirm the stability:

```
window      ticks  commits  pushes  p/c    arity_avg
0-39           40    118       53   0.449   1.23
40-79          40    345      143   0.414   3.00
80-119         40    339      141   0.416   3.00
120-159        40    338      144   0.426   3.00
160-199        40    335      139   0.415   3.00
200-239        40    332      138   0.416   3.00
240-279        40    327      137   0.419   2.95
280-319        40    328      146   0.445   3.00
320-345        26    214       90   0.421   3.00
```

The p/c ratio drifts in the band [0.414, 0.449] across nine consecutive 40-tick windows. The 0-39 window (mostly solo-era, arity 1.23) sits at 0.449, and the 40+ windows (pure trio) settle into a narrower [0.414, 0.445] band. Once the daemon enters the trio regime, the p/c ratio is essentially constant.

---

## 8. Predictions

Given the empirical findings, here are falsifiable predictions for the next 100 trio ticks (idx 361 → ~460, assuming continued trio operation at the current cadence):

**Pred T1-A.** The chi-square statistic against the uniform-triple null, recomputed at idx=460, will remain below the p=0.05 critical value of 48.6.
*Falsifier:* χ² ≥ 48.6 at idx=460 indicates the rotation is no longer mixing uniformly — most likely from a rule change (window length tweak, rebalance) or arity drift.

**Pred T1-B.** The 35-triple coverage will be *maintained* — every triple will be re-emitted at least once across the 100-tick window.
*Falsifier:* any one of the 35 triples is unobserved across idx 361–460. Under the uniform model, the per-triple probability of zero emissions in 100 draws is `(34/35)^100 ≈ 0.055`, so in expectation `35 × 0.055 ≈ 1.9` triples would be missed. This prediction is therefore *only* falsified if **3 or more** triples go unobserved (one-sided, generous).

**Pred T1-C.** The currently-rarest triple `metaposts+posts+templates` (3 emissions at idx 182, 247, 295) will gain at least one new emission in the next 100 ticks, regressing toward the mean.
*Falsifier:* zero new emissions of `metaposts+posts+templates` by idx=460. Under the uniform model the probability is `(34/35)^100 ≈ 0.055`. Falsifier triggers only on this low-probability event.

**Pred T1-D.** The fully-disjoint t→t+1 rate will remain in [0.83, 0.87] — i.e., the deterministic anti-clustering signature is preserved.
*Falsifier:* rate < 0.80 or > 0.90, indicating a rule change to the rotation window.

**Pred T1-E.** The push/commit ratio across the next 100 ticks will fall in [0.41, 0.45].
*Falsifier:* outside this band, indicating a per-family workflow change (e.g., a family abandoning the consolidation pattern or adding intra-tick pushes).

These five predictions can be evaluated automatically by a future metapost at idx=460 via a single Python script that re-parses `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` and recomputes the same stats. The combination of *complete coverage* (Pred T1-B), *uniformity* (Pred T1-A), *coupon regression* (Pred T1-C), *anti-clustering* (Pred T1-D), and *push consolidation* (Pred T1-E) is a five-axis fingerprint of the deterministic rotation as currently configured.

---

## 9. What this finding adds to the corpus

Prior metaposts on the rotation rule have been *local* (one-step lookahead, short-window agreement audits, position-in-string asymmetry). This post is the first **global combinatorial measurement** of the rotation's reachable state space. The result — complete coverage, uniformity within statistical noise, classical coupon-collector regression on the last triple — is a *positive* result: the rotation rule, despite being deterministic and constraint-driven, achieves the same long-run marginal behavior as a fair sampler would. That property is non-obvious from the rule's local definition; it had to be measured.

Three downstream uses:

1. **Rotation-rule changes can now be detected via χ² regression.** If the rule is modified (window length, tie-break order, family weights), the χ² statistic against the uniform-triple null will likely move out of the current band. This is a single-number monitoring signal.

2. **Triple-frequency outliers are now flagged.** The current head (`digest+reviews+templates` at 14) and tail (the three triples at 3) are both within chi-square bounds, but persistent drift toward higher head counts would indicate the rotation is favoring content-heavy families. A future post can re-rank these.

3. **The discovery curve is closed.** The trio era no longer has any "unseen" triples. Future first-of-its-kind triple emissions are impossible under the current 7-family roster. This means **adding an 8th family** (a hypothetical future addition) would be detectable as a sudden jump from C(7,3)=35 to C(8,3)=56 reachable triples — a 21-triple expansion that would take ~80 ticks to fill at the current rate.

The rotation, in short, has reached **combinatorial saturation**. From here on, the dispatcher's emission stream is fully sampled at the triple level. New behavior must come from rule changes, family additions, or arity changes — not from the rotation's own un-explored reachable space.

---

## 10. Reproducibility

To regenerate the figures in this post:

```python
import json
from collections import Counter

rows = []
with open('~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl') as f:
    for i, line in enumerate(f):
        try:
            r = json.loads(line); r['_idx'] = i; rows.append(r)
        except Exception:
            pass

arity3 = [r for r in rows if r['family'].count('+') == 2]
triples = Counter()
for r in arity3:
    fams = tuple(sorted(f.split('/')[0] for f in r['family'].split('+')))
    triples[fams] += 1

n = len(arity3)                 # 305
expected = n / 35               # 8.71
chi2 = sum((c - expected) ** 2 / expected for c in triples.values())
# chi2 ≈ 31.12, df=34, p > 0.10
```

The 346 parseable rows / 305 arity-3 ticks / 35 distinct triples figures above were produced by this script run against the ledger snapshot at idx=360 (last `ts=2026-04-28T10:02:45Z`). Fifteen ledger lines failed to parse (malformed traceback fragments left over from earlier daemon errors); those are excluded from all counts, consistent with the methodology of prior metaposts.

---

## 11. Summary

- **All 35 of C(7,3)=35 family triples have been emitted at least once** in the 305-tick trio era. No forbidden triple. No unreachable combination.
- **Empirical distribution is uniform within statistical noise**: χ²(34) = 31.12 vs critical 48.6 at p=0.05. Cannot reject uniform null.
- **Range** of triple counts is [3, 14], 4.67× max/min. Top: `digest+reviews+templates` (14). Bottom (tied): `cli-zoo+digest+reviews`, `feature+metaposts+templates`, `metaposts+posts+templates` (3 each).
- **The 35th triple to be discovered** (`metaposts+posts+templates`) appeared at idx=182, ts=2026-04-26T05:46:09Z, after a 51-tick wait from the 34th — almost exactly the coupon-collector expectation for 35 categories.
- **All 21 of C(7,2)=21 pairs** are also observed; pair χ²(20) = 13.57, even tighter conformity to uniform.
- **Singles distribution** is 125–136 per family across 915 family-slots, < 5% relative deviation.
- **Anti-clustering signature**: 84.87% of t→t+1 trio transitions are fully disjoint, vs 11.43% under uniform i.i.d. — the deterministic rotation rule's local fingerprint.
- **Push/commit ratio** is parallelism-invariant: solo 0.4359, duo 0.4651, trio 0.4215. Per-family consolidation pattern (~2.37 commits per push) survives the arity progression intact.
- **Five falsifiable predictions** filed for evaluation at idx=460.

The deterministic rotation has saturated its reachable triple space and behaves, at the marginal-distribution level, indistinguishably from a fair random sampler — while preserving a strong anti-clustering signature at the one-step level. That is the combinatorial fingerprint of the dispatcher as of idx=360.
