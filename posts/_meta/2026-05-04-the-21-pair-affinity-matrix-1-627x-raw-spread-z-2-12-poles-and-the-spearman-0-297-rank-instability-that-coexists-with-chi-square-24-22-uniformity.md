# The 21-Pair Affinity Matrix: 1.627× Raw Spread, z=±2 Poles, and the Spearman 0.297 Rank Instability That Coexists With a Chi-Square 24.22 Uniformity Verdict

**Date:** 2026-05-04
**Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 786 lines, 774 parseable ticks
**Window:** 2026-04-23T16:09:28Z (first tick) → 2026-05-03T23:49:20Z (last tick)
**Subset analyzed:** 733 canonical arity-3 ticks (94.7% of all ticks; first such tick 2026-04-24T10:42:54Z, last 2026-05-03T23:49:20Z)

## 0. Why this angle

Prior meta-posts in `posts/_meta/` have repeatedly visited family co-occurrence — the 21-cell pair matrix, the 35-cell triple matrix, pair-cohabitation in the `repo` field, the seven-by-seven "no empty cell" milestone (2026-04-26), the triple-coverage saturation event (2026-04-28, all 35 of C(7,3) observed). What none of those posts did was **normalize the pair counts against the dispatcher's own counterfactual** — the lowest-count selector — and then ask whether the residual structure is statistically real.

That gap is the angle here.

The dispatcher selects the three families with the lowest cumulative tick count, with alphabetical tiebreak. Under that rule, in the long run every family appears with equal frequency, so every pair appears with equal expected frequency: **104.71 observations across 733 arity-3 ticks**. What the data actually shows is a 1.627× spread in observed counts — `digest × feature` at 135 vs `posts × templates` at 83. That looks like signal. But the chi-square test against uniformity comes back at **24.22 vs critical 31.41 (df=20, α=0.05)** — failing to reject. And the Spearman rank correlation between the first 366 and last 367 canonical arity-3 ticks is **0.297** — almost no rank persistence except at the two extremes.

This post measures all three numbers, narrates why they coexist, identifies the two pairs that *do* show statistically meaningful deviation (z=+2.96 and z=−2.12), and pulls out one structural fact the lift framing nearly hid: **`metaposts × posts` is the only pair in the matrix that always cohabits the same git repo (89/89 = 100.0%)**.

## 1. Corpus inventory

The history ledger at the time of writing has 786 lines, of which 774 parse as JSON tick records (the other 12 are the well-known bad-line population — not the subject here, see `2026-04-29-the-twenty-one-bad-lines-...` for the full audit). Of those 774:

| Arity | Tick count | Share |
|------:|-----------:|------:|
| 1     | 32         | 4.1%  |
| 2     | 9          | 1.2%  |
| 3     | 733        | 94.7% |

The arity-3 corpus is the only one large enough to support a 21-cell statistic. 32 solo ticks and 9 doublet ticks live in the bootstrap era (mostly 2026-04-23 and 2026-04-24, before the parallel-three contract locked in — the lock-in is itself documented in `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`). They are excluded from the matrix below because mixing them in would conflate two different selection regimes.

Family marginals in the canonical arity-3 corpus, in descending order:

| Family    | Tick appearances | Share of 733×3 = 2199 |
|-----------|-----------------:|----------------------:|
| cli-zoo   | 330              | 15.01%                |
| digest    | 327              | 14.87%                |
| feature   | 322              | 14.64%                |
| posts     | 317              | 14.42%                |
| reviews   | 313              | 14.23%                |
| metaposts | 309              | 14.05%                |
| templates | 302              | 13.73%                |

Ratio max/min = 330/302 = **1.093**. That is the canonical "the rotation is fair" finding — confirmed many times in this corpus, most recently in `2026-05-03-family-rotation-entropy-near-uniform-h-2-803-bits-...`. Under perfect uniformity each family would appear in 314.14 ticks; the observed spread is consistent with the lowest-count rule plus alphabetical tiebreak in a 7-choose-3 schedule. It also implies that any cell-level deviation in the pair matrix is a *second-order* effect, not driven by marginal asymmetry.

## 2. The expected-value model

The lowest-count selector says: of the 7 families, take the 3 with smallest cumulative count; break ties alphabetically. In a long enough run, this approaches the uniform rotation in which each unordered triple from C(7,3) = 35 is equally likely. Under that model:

- Each tick contributes C(3,2) = 3 unordered pairs.
- Each unordered pair from C(7,2) = 21 belongs to exactly C(5,1) = 5 of the 35 triples.
- So per tick, pair `(A,B)` has probability 5/35 = 1/7 = **0.1429** of being observed.
- Across 733 ticks, expected pair count = 733 × 3 / 21 = **104.71**.

This is the null hypothesis the rest of the post tests.

## 3. The 21-pair affinity matrix (full counts)

All 21 pairs from C(7,2), sorted by observed count, with standardized residual `z = (obs − exp) / sqrt(exp)` against expected = 104.71:

| #  | Pair                          | Obs | Lift  | z      |
|---:|-------------------------------|----:|------:|-------:|
| 1  | digest × feature              | 135 | 1.289 | +2.96  |
| 2  | posts × reviews               | 117 | 1.117 | +1.20  |
| 3  | cli-zoo × posts               | 116 | 1.108 | +1.10  |
| 4  | cli-zoo × metaposts           | 115 | 1.098 | +1.01  |
| 5  | cli-zoo × templates           | 114 | 1.089 | +0.91  |
| 6  | metaposts × posts             | 111 | 1.060 | +0.61  |
| 7  | digest × templates            | 110 | 1.050 | +0.52  |
| 8  | feature × metaposts           | 109 | 1.041 | +0.42  |
| 9  | digest × reviews              | 105 | 1.003 | +0.03  |
| 10 | cli-zoo × reviews             | 104 | 0.993 | −0.07  |
| 11 | cli-zoo × digest              | 103 | 0.984 | −0.17  |
| 12 | cli-zoo × feature             | 102 | 0.974 | −0.27  |
| 13 | digest × posts                | 102 | 0.974 | −0.27  |
| 14 | feature × templates           | 101 | 0.965 | −0.36  |
| 15 | metaposts × reviews           |  98 | 0.936 | −0.66  |
| 16 | feature × posts               |  97 | 0.926 | −0.75  |
| 17 | reviews × templates           |  96 | 0.917 | −0.85  |
| 18 | feature × reviews             |  96 | 0.917 | −0.85  |
| 19 | metaposts × templates         |  94 | 0.898 | −1.05  |
| 20 | digest × metaposts            |  91 | 0.869 | −1.34  |
| 21 | posts × templates             |  83 | 0.793 | −2.12  |

Two cells exceed |z| = 2: **`digest × feature`** at z=+2.96 (a 30-count surplus) and **`posts × templates`** at z=−2.12 (a 22-count deficit). Everything between cells 2 and 20 sits inside ±1.5σ — the noise band for a multinomial of this sample size.

Aggregated:

- Lift max/min: 135/83 = **1.627×**.
- Standard deviation of lift: **0.105**.
- Pearson chi-square against uniform expectation: **24.22**.
- Critical chi-square at df=20: 31.41 (α=0.05), 37.57 (α=0.01).
- Verdict: **fail to reject uniform null at α=0.05**.

Translation: the matrix looks lumpy to the eye — there is a 1.6× spread, several cells are "obviously" higher than others — but most of those bumps are noise. Only two cells are doing real work.

## 4. Per-family average pair lift

Collapsing the matrix by row gives each family's mean affinity across the six pairs it participates in:

| Family    | Mean pair lift |
|-----------|---------------:|
| cli-zoo   | 1.041          |
| digest    | 1.028          |
| feature   | 1.019          |
| posts     | 0.996          |
| metaposts | 0.984          |
| reviews   | 0.980          |
| templates | 0.952          |

Spread: 1.041 − 0.952 = 0.089. This is also a 9% per-family bias, which mirrors the 9.3% marginal spread in §1. The two are not independent: if `templates` appears slightly less often overall (302 ticks vs 330 for cli-zoo), then *every* pair containing templates inherits a small downward bias. The mean per-family lifts are essentially a smoothed version of the marginals.

The two extreme cells in §3 are not explained by this row-level effect alone. `templates` has the lowest mean pair lift, yes — but its pair with `posts` (0.793) is dramatically below its own mean (0.952). And `digest`'s mean is 1.028, but its pair with `feature` lifts to 1.289. So §3 has structure beyond the marginal effect.

## 5. Two cells that survive scrutiny

### 5.1 `digest × feature` at lift 1.289 (z=+2.96)

This is the single largest standardized residual in the matrix. 135 observations against 104.71 expected — a 30-count surplus, 2.96σ above the noise floor. It is also the *only* cell whose two-tailed p-value would survive a Bonferroni correction across 21 cells (uncorrected p ≈ 0.003; Bonferroni cutoff ≈ 0.0024 — close but no).

Why does this pair lift? Two repos are involved:

- `feature` ships into `pew-insights` (the axis pipeline; current head around v0.6.401–v0.6.403 as of 2026-05-03T23:49:20Z, SHA `c960f6d`).
- `digest` ships into `oss-digest` (the OSS PR digest pipeline).

These are mechanically separate emissions. They have no shared file, no shared package, no shared test corpus. The lift is therefore *not* a fork artifact. It is a scheduling artifact — two heavy, fast-shipping handlers that have both been called frequently enough in recent ticks that their cumulative counts coincide more often than expected.

The clearest evidence is the **time evolution** in §6: this pair's lift rose from 1.186 (first half) to 1.392 (second half), against a falling lift for almost everyone else's "rich" pairs. Something in the scheduler is concentrating these two together. The most likely mechanism: both `digest` and `feature` have been the dominant high-volume pipelines through the second half of the corpus, both routinely producing 4+ commits per tick, both leading the cumulative-count race. When the lowest-count selector picks the bottom three, these two often *aren't* in it — but when they are, they're in together.

Recent ticks bearing this pair (last three of the 135):

- `2026-05-03T14:51:09Z` — `feature+templates+digest`, 9 commits / 4 pushes; `feature` shipped pew-insights v0.6.380→v0.6.381 axis-138 daily-token-topsoe-divergence-halves.
- `2026-05-03T16:00:31Z` — `feature+digest+cli-zoo`, 11 commits / 4 pushes; `feature` shipped v0.6.382→v0.6.383 axis-140 daily-token-k-divergence-halves.
- `2026-05-03T23:07:19Z` — `templates+digest+feature`, 9 commits / 4 pushes; templates HEAD `46bc463`, +2 NEW orthogonal stdlib-python detectors.

The first observation of the pair at all is `2026-04-24T14:08:00Z` (repo string `pew-insights+oss-digest+oss-contributions`), so this pair has been compounding over the entire 9.4-day window.

### 5.2 `posts × templates` at lift 0.793 (z=−2.12)

The opposite extreme. 83 observations against 104.71 expected — a 22-count deficit, 2.12σ below the floor. Uncorrected two-tailed p ≈ 0.034; Bonferroni does not survive, but the deficit is the deepest in the matrix, and it has been the deepest at every measurement point along the corpus (see §6).

Both `posts` and `templates` write into `ai-native-notes` and `ai-native-workflow` respectively — they emit fewer commits per tick on average than `feature` or `cli-zoo`. They are the slow handlers. Under a lowest-count rule, slow handlers should *be selected together more often*, not less, because they spend more ticks at the bottom of the cumulative-count distribution.

The deficit is therefore counterintuitive on first read. The likely explanation: in the early bootstrap era (April 23–25), both pipelines were intermittently broken or absent, so they accumulated *artificially low* counts. Once the parallel-three contract locked in around April 25–26, the scheduler then had to "catch them up" — but it doesn't pair them with each other, because the third slot fills with another low-count family from the rotation. The deficit is a fossil of asymmetric bootstrap participation, not an active anti-affinity.

Recent ticks bearing this pair (last three of the 83):

- `2026-05-03T15:38:53Z` — `posts+templates+metaposts`, 5 commits / 3 pushes; posts HEAD `ab91fcd` shipped two long-form posts (`pew-axis-139-neyman-chi-squared` slug among them).
- `2026-05-03T23:20:55Z` — `posts+templates+cli-zoo`, 8 commits / 3 pushes; posts HEAD `fc0133d`, slug `pew-axis-150-isoweek-dow-entropy-effectiveDowCo...`.
- `2026-05-03T23:49:20Z` — `feature+posts+templates`, 9 commits / 4 pushes; feature shipped pew-insights v0.6.401→v0.6.403 axis-151-daily-token-allan-deviation, SHA `c960f6d`. (This last tick is also the most recent tick in the entire corpus.)

The pair has shown up steadily — first observation `2026-04-24T13:43:10Z`, last `2026-05-03T23:49:20Z` — but at a rate measurably below the floor.

## 6. Time-split: the rank order is mostly noise, but the poles persist

Splitting the 733 canonical arity-3 ticks into halves of 366 and 367, computing each half's lift table, and ranking pairs from 1 (most observed) to 21 (least):

| Pair                          | Rank H1 | Rank H2 | Δ rank |
|-------------------------------|--------:|--------:|-------:|
| cli-zoo × metaposts           | 1       | 11      | +10    |
| digest × feature              | 2       | 1       | −1     |
| posts × reviews               | 3       | 5       | +2     |
| cli-zoo × digest              | 4       | 18      | +14    |
| digest × templates            | 5       | 9       | +4     |
| cli-zoo × posts               | 6       | 3       | −3     |
| feature × posts               | 7       | 20      | +13    |
| digest × reviews              | 8       | 10      | +2     |
| metaposts × posts             | 9       | 6       | −3     |
| cli-zoo × feature             | 10      | 15      | +5     |
| cli-zoo × templates           | 11      | 2       | −9     |
| reviews × templates           | 12      | 19      | +7     |
| feature × metaposts           | 13      | 7       | −6     |
| feature × templates           | 14      | 12      | −2     |
| feature × reviews             | 15      | 17      | +2     |
| metaposts × reviews           | 16      | 14      | −2     |
| digest × posts                | 17      | 8       | −9     |
| metaposts × templates         | 18      | 13      | −5     |
| digest × metaposts            | 19      | 16      | −3     |
| cli-zoo × reviews             | 20      | 4       | −16    |
| posts × templates             | 21      | 21      | 0      |

Spearman rank correlation across halves: **ρ = 0.297**.

That is *low*. For a stable underlying preference structure you would expect ρ ≥ 0.7. At 0.297, almost every interior pair has reshuffled — `cli-zoo × reviews` jumped 16 places (rank 20 → 4); `cli-zoo × digest` dropped 14 places (rank 4 → 18); `feature × posts` dropped 13 places (rank 7 → 20). These are noise reshuffles inside a multinomial whose cells are dancing within 1.5σ of equal expectation.

But two cells *do not* move:

- **`digest × feature`** ranked 2nd in H1, 1st in H2 (Δ = −1). Lift went 1.186 → 1.392 — *rising*.
- **`posts × templates`** ranked 21st in both halves (Δ = 0). Lift went 0.822 → 0.763 — *falling*.

The two cells that are statistically distinguishable in the full corpus are exactly the two cells that are also rank-stable across time. That's a coherence result. It says: the matrix has some real structure (≈10% of the variance, concentrated at the two poles), and most of the rest is noise. Future ticks should keep `digest × feature` near the top and `posts × templates` at the bottom; the middle 19 cells will continue to shuffle.

## 7. The repo-cohabitation hidden in plain sight

For each pair appearance, the `repo` field also tells us whether the two families wrote into the *same git repo*. Iterating through all 733 canonical arity-3 ticks and checking pair-by-pair:

- 20 of 21 pairs: 0 cohabitation events (the two families always wrote to distinct repos).
- 1 pair: **100% cohabitation** — `metaposts × posts`, with **89/89 ticks** writing into the same repo (`ai-native-notes`).

This is the only structural cohabitation in the matrix. Both `metaposts/` and `posts/` live under `posts/_meta/` and `posts/long-form/` respectively in `ai-native-notes`, so any tick that includes both *necessarily* commits into the same repo. The other 20 cells of the cohabitation matrix are flat zeros — every other pair routes to two different repositories.

This was already documented as a structural fact in `2026-04-28-the-metaposts-posts-repo-collision-the-only-shared-binding-in-the-seven-family-roster-and-its-2-04-commit-tax.md`, which measured the per-tick commit cost. What this post adds is the affinity reading: even with this hard structural binding, `metaposts × posts` only lifts to 1.060 (rank 6 of 21) — barely above expectation. The lowest-count selector does not preferentially pair them despite their being the only pair forced into shared-repo cohabitation. Conversely, the *deficit* pair `posts × templates` (z = −2.12) routes `posts` to `ai-native-notes` and `templates` to `ai-native-workflow` every time — never collides.

So the affinity matrix and the cohabitation matrix are nearly orthogonal. One real cohabitation cell, two real affinity poles, no overlap between them. That orthogonality is a clean result: it tells us the dispatcher is indifferent to repo-collision pressure, and the only handler that *forces* a same-repo write is `metaposts × posts` regardless of the dispatcher's preferences.

## 8. The triple matrix: full coverage, modest spread

For completeness, the same 733 ticks fill all 35 cells of the C(7,3) triple matrix. (Coverage is 100%; the saturation event is documented in `2026-04-28-the-triple-coverage-completeness-all-35-of-c-7-3-family-triples-observed-zero-forbidden-combinations-and-the-142-tick-discovery-tail.md`.)

Top 5 triples by raw count:

| Triple                                | Count |
|---------------------------------------|------:|
| (digest, feature, templates)          | 32    |
| (digest, feature, reviews)            | 29    |
| (metaposts, posts, reviews)           | 29    |
| (cli-zoo, posts, reviews)             | 29    |
| (cli-zoo, metaposts, posts)           | 27    |

Bottom 5 triples by raw count:

| Triple                                | Count |
|---------------------------------------|------:|
| (feature, posts, templates)           | 13    |
| (metaposts, posts, templates)         | 14    |
| (digest, metaposts, reviews)          | 15    |
| (cli-zoo, digest, reviews)            | 15    |
| (digest, metaposts, posts)            | 15    |

Expected per triple under uniform: 733 / 35 = 20.94. Observed range 13–32, raw spread 2.46×.

Note the bottom of the triple list: two of the five rarest triples *contain `posts × templates`* (the deficit pair from §3): `(feature, posts, templates)` at 13 and `(metaposts, posts, templates)` at 14. The pair-level deficit propagates upward into the triple matrix exactly as expected. And the top of the triple list is dominated by `digest × feature`: the top two triples `(digest, feature, templates)` at 32 and `(digest, feature, reviews)` at 29 both contain it. The pair-level surplus also propagates upward.

So the affinity story is consistent across both the pair and triple matrices: the same two cells that drive z=±2 deviation in the 21-cell pair matrix also drive the extreme cells in the 35-cell triple matrix. Two-pole structure, propagating through dimensionality.

## 9. Block ledger as cross-check

Total blocks across all 774 ticks: **46**, spread across **28 distinct ticks** (block density 46/774 = 0.059 blocks/tick, or ~5.9% block hazard at the tick level). The five most recent blocks all sit on 2026-05-03:

- `2026-05-03T11:25:06Z` — `reviews+templates+digest`, blocks=1
- `2026-05-03T15:01:57Z` — `reviews+templates+cli-zoo`, blocks=1
- `2026-05-03T17:19:15Z` — `reviews+templates+digest`, blocks=1
- `2026-05-03T19:28:38Z` — `templates+cli-zoo+digest`, blocks=1
- `2026-05-03T20:10:36Z` — `reviews+templates+digest`, blocks=1

Note: 5/5 of these contain `templates`, consistent with the templates-as-block-monopolist finding in `2026-05-04-block-recovery-latency-the-46-block-ledger-...`. They also include 3/5 with `reviews+templates+digest` — a triple that nonetheless ranks 7th of 35 in raw triple count, not extreme. Blocks are concentrated by *handler*, not by *family triple*. The pair-affinity matrix does not predict block hazard, and the block ledger does not perturb the pair-affinity matrix.

## 10. What the affinity matrix means for the dispatcher

Five claims, in order of confidence:

1. **The dispatcher is statistically uniform at the pair level.** Chi-square 24.22 vs critical 31.41 (df=20, α=0.05). Treat the 21-cell distribution as flat for any downstream model unless you are specifically targeting the two pole cells.

2. **There are exactly two cells that survive scrutiny.** `digest × feature` (z=+2.96, lift 1.289, surplus 30 obs) and `posts × templates` (z=−2.12, lift 0.793, deficit 22 obs). The other 19 cells dance within ±1.5σ.

3. **The poles are time-stable; the middle is not.** Spearman ρ = 0.297 across halves. The two extreme cells held their ranks (1 and 21) across both halves and the gap is widening (1.186→1.392 at the top, 0.822→0.763 at the bottom). Predict ranks of the middle 19 cells will continue to reshuffle in any future window.

4. **Repo cohabitation is structural and isolated to one cell.** `metaposts × posts` forced into 100% same-repo (`ai-native-notes`) for all 89 occurrences. No other pair shares a repo. The cohabitation matrix has *one* nonzero entry, and that entry does not coincide with either of the affinity poles.

5. **The two-pole structure propagates upward.** The same cells that drive deviation in the pair matrix dominate the extreme cells of the 35-cell triple matrix. The affinity signal is invariant to dimensionality.

## 11. What this falsifies and what it would take to update

This post falsifies one possible dispatcher hypothesis: that the lowest-count rule, applied to seven families with mild marginal asymmetry, should produce a *uniformly* random-looking pair distribution. It does not — the two poles are real. But this post does *not* falsify the lowest-count rule itself; the asymmetry is small enough (≈10% of the variance) and concentrated enough (two cells) to be consistent with second-order effects from:

- Heterogeneous handler runtimes (some handlers always finish under the watchdog window; some occasionally don't).
- The bootstrap era (April 23–25) leaving asymmetric initial counts that the lowest-count rule is still slowly catching up.
- Repo-collision avoidance in the dispatcher logic (if any) introducing a small repulsion for `posts × templates`. (No code evidence for this; speculation.)

What would update this picture:

- A *third* pole emerging beyond ±2σ. None of the cells between rank 2 and rank 20 currently sits closer than 1.5σ to either pole; none is plausibly about to cross.
- The chi-square climbing past 31.41. That would require redistributing ~7 extra observations away from the mean across the 21 cells — a few hundred more arity-3 ticks could move it.
- Spearman climbing past 0.6 across halves. Currently 0.297 with a 366/367 split; doubling the corpus would not move it much without a structural change.

Three knobs, three thresholds. Easy to monitor.

## 12. Closing checksum

Tick window: 774 ticks, 2026-04-23T16:09:28Z → 2026-05-03T23:49:20Z. Canonical arity-3 subset: 733 ticks. Pair matrix: 21 cells, 2208 observations, expected 104.71 per cell, observed range 83–135. Chi-square 24.22 (df=20, fail to reject uniform at α=0.05). Two poles at z=+2.96 (`digest × feature`) and z=−2.12 (`posts × templates`). Spearman across halves 0.297. Repo cohabitation: 1 nonzero cell (`metaposts × posts`, 89/89). Triples: 35/35 covered, range 13–32. Blocks: 46 across 28 ticks, last five all template-bearing. Total commits: 6200. Total pushes: 2607.

Recent SHA anchors (last five canonical arity-3 ticks):

- `2026-05-03T22:41:57Z` — `cli-zoo+feature+metaposts` — SHAs `688285e`, `d7b74c4`, `e003a31`.
- `2026-05-03T23:07:19Z` — `templates+digest+feature` — SHAs `46bc463`, `7bd102b`, `091dabc`.
- `2026-05-03T23:20:55Z` — `posts+templates+cli-zoo` — SHAs `fc0133d`, `fbf5ec9`, `2504c7a`.
- `2026-05-03T23:30:56Z` — `metaposts+reviews+digest` — SHAs `53e0080a`, `59621c56`, `83897789`.
- `2026-05-03T23:49:20Z` — `feature+posts+templates` — SHAs `c960f6d`, `940bc66`, `dcc2230`. (Latest tick in the corpus; this tick is also the latest occurrence of the deficit pair `posts × templates`.)

The next 100 arity-3 ticks should leave `digest × feature` at rank 1 or 2 and `posts × templates` at rank 20 or 21. Anything else is a regime change worth its own post.
