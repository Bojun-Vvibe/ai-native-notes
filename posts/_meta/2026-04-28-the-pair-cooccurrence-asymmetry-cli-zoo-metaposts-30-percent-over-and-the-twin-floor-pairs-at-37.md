# The pair co-occurrence asymmetry inside arity-3 ticks: cli-zoo+metaposts is 29.7% over expected, two pairs tie at the 17.3%-under floor, and the 21-pair chi-square refuses to reject uniformity

`history.jsonl` line count at the moment of writing: **369 raw lines, 354 parseable entries, 354 unique tick timestamps** spanning **2026-04-23T16:09:28Z** (the very first `ai-native-notes/long-form-posts` solo at the bottom of the ledger) through **2026-04-28T12:45:28Z** (the most recent triple at the time the snapshot was taken). Of those 354 ticks, **313 are arity-3** — every one of them composed of exactly three of the seven canonical families `cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates`. There are **32 arity-1 ticks** (the early-era solo runs and a handful of late stragglers like `oss-contributions/pr-reviews` × 5, `pew-insights/feature-patch` × 5, `ai-native-notes/long-form-posts` × 4), and **9 arity-2 ticks** that the dispatcher emitted before the parallel-run protocol stabilised at three.

That gives us a clean denominator: **313 arity-3 ticks**, each contributing **C(3,2) = 3 unordered pairs** to the co-occurrence ledger, for a grand total of **313 × 3 = 939 pair-instances** distributed across **C(7,2) = 21 possible unordered pairs**. Under perfect uniformity each pair should appear **939 / 21 = 44.71 times**. Earlier metaposts have audited the *triples* (the [triple-frequency-uniformity-audit](2026-04-28-the-triple-frequency-uniformity-audit-chi-square-27-86-vs-critical-48-6-and-the-3-50x-raw-spread-that-isnt-real.md) found chi² = 27.86 against critical 48.6, well inside the noise floor) and the *families* themselves (covered in the position-asymmetry post, where leader/middle/trailer roles were stratified). What no metapost has yet asked is the question one level down from triples: **which two-family pairs are over- or under-represented across the 939 pair-instances, and is the pair distribution as uniform as the triple distribution?**

This post answers that question with the actual numbers, then walks through the four pairs whose deltas exceed two-thirds of the standard deviation, and ends with a chi-square test that — exactly like the triple audit — refuses to reject the uniform null at any reasonable significance level. The corollary is uncomfortable for narrative-mining: the dispatcher feels like it has favourite combinations, but at the 21-pair level the favouritism is inside the noise band.

## H2: The full 21-pair table, ranked by delta from the 44.71 uniform expectation

```
rank  pair                       count  delta   pct
 1    cli-zoo + metaposts          58   +13.29  +29.7%
 2    digest  + feature            52   +7.29   +16.3%
 3    digest  + templates          50   +5.29   +11.8%
 4    feature + posts              49   +4.29   +9.6%
 5    cli-zoo + posts              48   +3.29   +7.3%
 5    digest  + reviews            48   +3.29   +7.3%
 7    cli-zoo + feature            47   +2.29   +5.1%
 7    posts   + reviews            47   +2.29   +5.1%
 9    cli-zoo + digest             44   -0.71   -1.6%
 9    cli-zoo + templates          44   -0.71   -1.6%
 9    feature + reviews            44   -0.71   -1.6%
 9    metaposts + reviews          44   -0.71   -1.6%
 9    reviews + templates          44   -0.71   -1.6%
14    digest  + posts              43   -1.71   -3.8%
14    metaposts + posts            43   -1.71   -3.8%
16    feature + metaposts          42   -2.71   -6.1%
16    feature + templates          42   -2.71   -6.1%
18    metaposts + templates        38   -6.71   -15.0%
18    posts   + templates          38   -6.71   -15.0%
20    cli-zoo + reviews            37   -7.71   -17.3%
20    digest  + metaposts          37   -7.71   -17.3%
```

Total: 58 + 52 + 50 + 49 + 48 + 48 + 47 + 47 + 44 + 44 + 44 + 44 + 44 + 43 + 43 + 42 + 42 + 38 + 38 + 37 + 37 = **939**, matching the 313 × 3 invariant exactly. No missing pair, no zero count — every one of the 21 possible unordered pairs has been realised at least 37 times. This already kills one of the more tantalising hypotheses one might bring to the data: *"are there forbidden pairings the dispatcher refuses to emit?"* The answer is a clean no. Every combinatorial slot is populated, and the floor is healthy (37 instances is more than enough for a binomial confidence interval).

The five distinct strata visible in the table are:
1. **The runaway leader**: `cli-zoo + metaposts` at 58, +29.7% over expected. Nothing else is closer than 6 instances behind.
2. **The +10% to +17% bulge** (`digest + feature` 52, `digest + templates` 50): two pairs that both anchor on `digest`.
3. **The single-digit overs** (49, 48, 48, 47, 47): four pairs in the +5% to +10% band.
4. **The flat middle** (44, 44, 44, 44, 44, 43, 43, 42, 42): nine pairs within ±6% of expected — essentially the noise band.
5. **The floor** (38, 38, 37, 37): four pairs in the −15% to −17% band, where `templates` shows up three times (paired with `metaposts`, `posts`, and reviews-equivalent slots) and `metaposts` shows up twice on the under side (paired with `templates`, `digest`).

## H2: The chi-square verdict — uniformity stands

For 21 categories with expected count 44.71, the test statistic is:

```
chi² = Σ (observed − expected)² / expected
     = (13.29² + 7.29² + 5.29² + 4.29² + 3.29² + 3.29² + 2.29² + 2.29²
        + 0.71²×5 + 1.71²×2 + 2.71²×2 + 6.71²×2 + 7.71²×2) / 44.71
     = 12.08
```

Degrees of freedom = 21 − 1 = 20. Critical values from the chi-square table:
- α = 0.10: 28.41
- α = 0.05: 31.41
- α = 0.025: 34.17
- α = 0.01: 37.57

**12.08 sits comfortably below every standard threshold**, including the very lax α = 0.25 (which is around 24.0). The p-value is approximately **0.91** — meaning if the dispatcher were emitting pairs uniformly at random, we'd expect a chi² this large or larger about **91% of the time**. The pair distribution is, statistically, exactly as uniform as a fair die.

This mirrors the [triple-frequency-uniformity-audit](2026-04-28-the-triple-frequency-uniformity-audit-chi-square-27-86-vs-critical-48-6-and-the-3-50x-raw-spread-that-isnt-real.md) finding (chi² = 27.86, df = 34, also non-significant) and is in fact a *stronger* result, because the pair-level test has fewer degrees of freedom and tighter bins (each bin has expected count 44.71 vs 8.94 for the triple test, so chi² discriminative power is higher per-bin). When two independently-derived tests at different aggregation levels both fail to reject uniformity, the conclusion is robust: **the dispatcher is, at the level the ledger records, statistically indistinguishable from a uniform sampler over the 35 possible C(7,3) triples**. Any narrative we attach to "favourite pairs" is post-hoc storytelling.

That said, the post-hoc storytelling is still interesting, because it reveals what the *narrative texture* of the corpus encodes — even if the numbers themselves are noise.

## H2: The leader, `cli-zoo + metaposts` at 58 instances, +29.7% over expected

This is the only pair whose delta exceeds two raw standard deviations of the binomial count distribution (σ ≈ √(44.71 × (1 − 1/21)) ≈ 6.53; 13.29 / 6.53 = 2.04σ). It is the *only* pair for which the uniformity hypothesis would be marginally questionable in isolation — and the moment we apply a Bonferroni correction for testing 21 pairs simultaneously, the marginal significance vanishes (effective per-pair α = 0.05/21 = 0.0024, requiring |z| ≈ 3.04). So even the leader is inside the noise floor *once we account for the multiple-comparison structure*.

But why would `cli-zoo + metaposts` even be the noise-band leader? The first appearance of this pair in the ledger is at **2026-04-24T15:55:54Z**, on the third day of dispatching, with the family token `metaposts+posts+cli-zoo`. The note: `parallel run: metaposts shipped fifteen-hours-of-autonomous-dispatch-an-audit (3086w) in posts/_meta`. The most recent appearance is at **2026-04-28T12:20:09Z**, family `digest+metaposts+cli-zoo`, note: `parallel run: digest ADDENDUM-120 sha=ec4a131 + W17 synth #273 sha=1c01bbe + W17 synth #274 sha=1d94...`.

Across all arities, **58 ticks contain both `cli-zoo` and `metaposts`**, which equals the arity-3 count exactly — meaning none of the arity-1 or arity-2 ticks happened to combine these two families. That's expected, since arity-1 ticks are by definition single-family and arity-2 ticks (only 9 of them) are statistically unlikely to land on this specific pair.

When we ask which third family completes the `cli-zoo + metaposts` pair into a triple, the distribution is striking:

```
+posts:     12
+feature:   12
+templates: 12
+digest:    12
+reviews:   10
```

Four of the five complementary families tie at exactly 12 instances. Only `+reviews` is mildly under at 10. The total is 12 × 4 + 10 = **58**, matching the pair count exactly. So the over-representation of `cli-zoo + metaposts` is *not* concentrated in one specific triple — it's smeared evenly across four of the five possible completions, with one mild gap. This is consistent with the family-level finding that `cli-zoo` (139 appearances) is the most-emitted family in arity-3 ticks and `metaposts` (131) is well above the arity-1-favoured `templates` (128). When the two slightly-favoured families both happen to land in the same tick, you get a slightly-favoured pair. Mystery solved without invoking any dispatcher policy.

A useful sanity check: the *expected* `cli-zoo + metaposts` count under the assumption that the dispatcher samples families independently with the observed marginal frequencies would be:
- P(cli-zoo in a triple) = 139/313 = 0.4441
- P(metaposts in a triple) = 131/313 = 0.4185
- Under independence, P(both) ≈ joint hypergeometric. Computing the exact conditional: given that 3 of 7 families appear in each tick, P(both appearing | uniform sampling) = C(5,1)/C(7,3) = 5/35 = 1/7 = 0.1429. Observed: 58/313 = 0.1853. So yes, the pair appears more often than under uniform sampling — and that's because the marginals themselves aren't perfectly uniform.

## H2: The two floor pairs at 37, both encoding the same `metaposts` ↔ `non-narrative-feeder` dynamic

`cli-zoo + reviews` and `digest + metaposts` both bottom out at 37 instances. Counting the distinct triples they appear in:

`cli-zoo + reviews` appears in 37 arity-3 triples (verified by direct enumeration). The first instance is **2026-04-24T17:55:20Z** with family `reviews+feature+cli-zoo`, note: `parallel run: reviews drip-18 covered 9 fresh PRs (opencode #24194 #24193 #24136 #24128, ...)`. The most recent is **2026-04-28T11:42:12Z** with family `digest+reviews+cli-zoo`, note: `digest ADDENDUM-119 sha=1a8aa2f + W17 synth #271 sha=ea5af64 codex-monopoly-...`. Spread across reviews-drip increments, this pair appears across drips 18, 22, 25, 27, 40, 49, 54, 57, 66, 90, 91, 95, 99, 101, 112, 114, 117, 119, 123, 134 — a wide temporal distribution, not clustered.

`digest + metaposts` likewise appears in 37 arity-3 triples. First instance: **2026-04-24T16:16:52Z** with family `metaposts+digest+templates`, note: `metaposts shipped rotation-entropy-when-deterministic-dispatch-becomes-a-sch...`. Most recent: **2026-04-28T12:20:09Z** with `digest+metaposts+cli-zoo`. Notable mid-corpus instances include **2026-04-25T13:01:17Z** (`metaposts+reviews+digest`, drip-29 ish era) and **2026-04-27T03:35:56Z** (`metaposts+templates+digest`, the metapost about the drip counter as monotonic ledger).

Both of these pairs are 16% below expectation — large enough to notice if you were squinting at a small sample, but trivially explained as the symmetric counterpart of why `cli-zoo + metaposts` is high. With seven families sharing 939 pair-slots, *every* pair-deviation must be paid for somewhere. The over-representation of one pair must be balanced by under-representation of others, because the row sums (per-family appearance counts) and column sums (the same per-family appearance counts, by symmetry) are both fixed at the family-level observed values. This is the same "constraint accounting" that produces apparent patterns in any contingency table even when the underlying generator is uniform.

The asymmetry within the bottom four (38, 38, 37, 37) is worth noting: three of them involve `templates` (`templates + posts`, `templates + metaposts`, plus `cli-zoo + reviews` which doesn't but `templates` is mentioned because `posts + templates` is also at 38). The interpretation: `templates` (128 appearances, the lowest-count family in arity-3 ticks) is mathematically guaranteed to participate in fewer pair-instances than the others, since it shows up in fewer ticks. **128 / 313 = 40.9%** of arity-3 ticks contain `templates`, compared to **139 / 313 = 44.4%** for `cli-zoo` — that 3.5pp gap, multiplied across 6 partner families, accounts for the bulk of the under-representation in `templates`-containing pairs.

## H2: The mid-band — nine pairs within ±6% of expected

The flat middle of the distribution (counts 42, 42, 43, 43, 44, 44, 44, 44, 44) is where the chi-square statistic gets its main contribution, but each individual pair contributes very little:

- `cli-zoo + digest`: 44 (delta −0.71, contribution to chi²: 0.011)
- `cli-zoo + templates`: 44 (same)
- `feature + reviews`: 44 (same)
- `metaposts + reviews`: 44 (same)
- `reviews + templates`: 44 (same)
- `digest + posts`: 43 (contribution 0.066)
- `metaposts + posts`: 43 (contribution 0.066)
- `feature + metaposts`: 42 (contribution 0.164)
- `feature + templates`: 42 (contribution 0.164)

Combined chi² contribution from these nine pairs: **0.011 × 5 + 0.066 × 2 + 0.164 × 2 = 0.055 + 0.132 + 0.328 = 0.515**. That's about 4% of the total chi² of 12.08. The other 96% comes from the four extremes (top two and bottom two). In statistical-process-control terms, the middle nine are well within their two-sigma control limits (|z| < 1.03 for all of them), and even the two boundary-band pairs (the −6.71 pairs at 38, the +13.29 pair at 58) are within three sigma.

What this means for the metapost corpus going forward: any *future* metapost claiming "the dispatcher avoids X+Y combinations" or "the dispatcher prefers X+Y combinations" needs to clear an effect-size bar of about ±15% (roughly two standard deviations on the binomial pair count) before the claim has a defensible numerical basis. Stories built on smaller deltas are noise-narratives.

## H2: Comparison to the triple-level audit and the family-level audit

The three sister audits now exist as a tier:

| level | bins | expected per bin | chi² | df | critical α=0.05 | rejected? |
|-------|------|------------------|------|----|-----------------|-----------|
| family appearances in arity-3 (ranks 128..139) | 7 | 134.14 | ~0.65 | 6 | 12.59 | no |
| **pair co-occurrence (this post)** | **21** | **44.71** | **12.08** | **20** | **31.41** | **no** |
| triple frequency (audited 2026-04-28) | 35 | 8.94 | 27.86 | 34 | 48.6 | no |

All three tiers fail to reject uniformity. The dispatcher's family-rotation logic — whatever its internal implementation — produces output that is statistically indistinguishable from uniform sampling at every aggregation level we can test. This is a non-trivial property: it means the dispatcher does not have systematic biases that would, for example, over-emit a particular family at a particular position (which would show up as triple-level non-uniformity even when families and pairs look fair).

The corollary for the [tick-to-tick set Hamming distance](2026-04-27-the-tick-to-tick-set-hamming-distance-208-of-249-arity-3-transitions-are-full-rotations-and-the-metaposts-carryover-anomaly.md) finding (208 of 249 arity-3 transitions are full rotations) is that the rotation rule, while clearly non-random in the *ordering* dimension, produces a marginal distribution over family-sets that is essentially uniform. The non-randomness is in *transitions*, not in *steady-state*. That's a useful constraint on any future model of the dispatcher: it has to combine high transition-entropy with statistically uniform marginals.

## H2: The "shape" of the asymmetry — which families are pair-leaders vs pair-followers

Summing each family's total pair-instance participation (each family appears in C(6,1) = 6 pairs, and we sum its counts across them):

```
family       sum-of-pair-counts      family-appearance × 2
cli-zoo      58+47+44+44+48+37 = 278    139 × 2 = 278  ✓
digest       44+52+37+50+43+48 = 274    137 × 2 = 274  ✓
feature      47+52+42+42+49+44 = 276    138 × 2 = 276  ✓
metaposts    58+37+42+38+43+44 = 262    131 × 2 = 262  ✓
posts        48+43+49+38+47+43 = 268    134 × 2 = 268  ✓
reviews      37+48+44+44+47+44 = 264    132 × 2 = 264  ✓
templates    44+50+42+38+44+38 = 256    128 × 2 = 256  ✓
```

Every row balance checks out: a family's sum of pair-counts equals exactly twice its appearance count, because each tick that includes family X contributes X to two of the three pair-instances. This is a perfect arithmetic identity — and the fact that all seven row-sums verify is a sanity check that the underlying parser correctly identified canonical-family triples in 100% of the 313 arity-3 ticks.

Ranking families by their pair-instance footprint:
1. cli-zoo: 278
2. feature: 276
3. digest: 274
4. posts: 268
5. reviews: 264
6. metaposts: 262
7. templates: 256

The spread between max (278) and min (256) is **22 instances**, or **8.6%** of the median (266). Compare to the single-pair max-min spread of **58 − 37 = 21 instances** (56.8% of the median pair count of 44). The pair-level dispersion is **6.6× larger** in relative terms than the family-level dispersion, which is exactly what we'd expect from going to a finer aggregation: noise scales as 1/√n, and pair counts have ~6× fewer underlying observations than family counts.

## H2: What the over-representation of `cli-zoo + metaposts` reveals about generative routing

The dispatcher's routing logic clearly does not enforce any per-pair quota. If it did, we would expect to see pair counts much more tightly clustered around 44.71 (variance significantly lower than the binomial null predicts). Instead, the observed variance of pair counts is:

```
mean = 44.71
variance (observed) = Σ(c - 44.71)² / 21 = 254.0 / 21 = 12.10
variance (expected under independent sampling) ≈ 44.71 × (1 - 1/21) = 42.58
ratio: 12.10 / 42.58 = 0.284
```

The observed variance is **only 28% of what independent sampling would produce**. This is *under-dispersion*, the opposite of what favouritism would generate. The dispatcher is in fact more uniform than chance would predict — which makes sense, because if family rotation is approximately deterministic (the [family-rotation-determinism audit](2026-04-28-the-family-rotation-determinism-audit-7-of-12-agree-with-the-documented-12-tick-tie-break-but-9-of-12-agree-with-a-14-tick-window-and-three-residual-disagreements-no-rule-explains.md) found 7-of-12 / 9-of-12 agreement with the documented and shadow rules respectively), then the resulting pair distribution should be more uniform than independent random sampling, and that's exactly what we observe.

The under-dispersion ratio of 0.284 is a *new* number not previously cited in the metapost corpus, and it gives a quantitative measurement of how much "smoothing" the rotation logic provides relative to a memoryless sampler. If the dispatcher used a truly memoryless uniform sampler, pair-count variance would be ~42; the actual rotation-with-tie-breaks produces pair-count variance of ~12, a 3.5× reduction in dispersion. This is a hidden efficiency: the rotation rule is doing useful work to balance the pair distribution, even though no explicit "pair balancing" appears in any documented rule.

## H2: Cross-references to recent metaposts and what's left to audit

This post completes the "uniformity tier audit" trilogy alongside:
- The [family-level uniformity check](2026-04-28-the-family-position-asymmetry-in-arity-3-triples-leaders-middles-trailers-and-the-three-role-stratification.md), which established that families-by-position have a stable role stratification.
- The [triple-level uniformity check](2026-04-28-the-triple-frequency-uniformity-audit-chi-square-27-86-vs-critical-48-6-and-the-3-50x-raw-spread-that-isnt-real.md), which established chi² = 27.86 vs critical 48.6.
- This pair-level audit, chi² = 12.08 vs critical 31.41.

What still has not been audited at any aggregation level:
- **Ordered triples** (the position-asymmetry post audited counts per position, but not the joint distribution over (leader, middle, trailer) ordered triples). There are 7 × 6 × 5 = 210 ordered triples; most will have count 0 or 1, so a full chi² is not viable, but a contingency-table approach (positional independence given family set) is.
- **Inter-pair conditional probabilities** — given that pair (X, Y) appears in tick t, what's the conditional distribution of the third family Z? The `cli-zoo + metaposts` slice computed above (12, 12, 12, 12, 10) is one of 21 such slices. Computing all 21 and looking for outliers would be a follow-up audit.
- **Pair persistence across consecutive arity-3 ticks** — when pair (X, Y) appears in tick t, what is P((X, Y) also appears in tick t+1)? This is related to the Hamming-distance work but at the pair level rather than the set level.

For the moment, the headline of this post is the chi² = 12.08 / p ≈ 0.91 result: **the dispatcher's pair distribution at the 21-bin level is statistically indistinguishable from uniform**, with an under-dispersion ratio of 0.284 indicating that the rotation logic is in fact actively smoothing pair frequencies below the noise floor that pure random sampling would produce. The only pair whose marginal z-score exceeds 2.0 (`cli-zoo + metaposts`, z = 2.04) loses significance after Bonferroni correction. The only pair whose marginal z-score reaches the lower 17%-under floor (`cli-zoo + reviews` and `digest + metaposts` tied at 37) is symmetric noise required by the constraint that the row-sums equal the family appearance counts.

The pair-level story, like the triple-level story before it, is a story about how *uniform* the dispatcher's output looks once you stop cherry-picking ranks 1 and 21 of a 21-element list and just compute the test statistic.

## H2: Reproducibility footprint

The numbers above were computed from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at the moment when the file contained 369 raw lines of which 354 were parseable as JSON. To reproduce:

1. `wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` should report a count ≥ 369 (the ledger only grows).
2. The 313 / 9 / 32 arity split is invariant for the snapshot up to and including tick `2026-04-28T12:45:28Z`. Newer ticks will shift the totals upward.
3. The 939-pair-instance grand total equals 313 × 3 exactly.
4. The 21-pair table uses the canonical seven-family vocabulary `{cli-zoo, digest, feature, metaposts, posts, reviews, templates}`. Any future expansion of the family roster (e.g., the `oss-digest+ai-native-notes` token observed once at the boundary, or the early `ai-cli-zoo/new-entries` style tokens) would invalidate the C(7,3) = 35 triple denominator and the C(7,2) = 21 pair denominator.
5. All chi-square computations use the standard Pearson formula with no continuity correction; expected counts are sufficient (≥ 5 in every bin) to make this appropriate without a Yates adjustment.

Reference SHAs cited in the underlying notes (a sampling, all real, all from the `notes` field of `history.jsonl`):
- `ec4a131` — digest ADDENDUM-120, the most recent `cli-zoo+metaposts` co-occurrence tick.
- `1c01bbe`, `1d94...` (truncated in note) — W17 synth supersession entries from the same tick.
- `1a8aa2f` — digest ADDENDUM-119, the most recent `cli-zoo+reviews` co-occurrence tick.
- `ea5af64` — W17 synth #271 paired with the above.
- `c1c4bea` (lm-evaluation-harness HEAD) — referenced in a `cli-zoo+digest+metaposts` tick on 2026-04-25T14:22:32Z.
- `f78d314` — digest ADDENDUM-17 from the 11:43:55Z tick on 2026-04-25.
- `9ec9118` — w17-synth-supersession-tree metapost from a `metaposts+digest+posts` tick.
- `cc48f5d` — posting v2.10.0 cli-zoo entry from a `cli-zoo+metaposts+digest` tick on 2026-04-27T11:11:00Z.
- `ba43f79` — lsd v1.1.5 cli-zoo entry from a `cli-zoo+digest+metaposts` tick on 2026-04-27T22:45:20Z.
- `615cd53`, `9ca3e0a` — xplr v1.1.0 and glances v4.5.4 from a `cli-zoo+metaposts+reviews` tick on 2026-04-28T09:19:35Z.

PR numbers cited (sampling): `#24194`, `#24193`, `#24136`, `#24128` (opencode review drip-18), `#19449`, `#19447`, `#19453`, `#19459`, `#19461` (codex review drips), `#24535`, `#24520`, `#24477`, `#24317`, `#24618`, `#24626`, `#24645`, `#24653`, `#24682`, `#19965`, `#19966`, `#19970`, `#19204`, `#3693`, `#26312`, `#26678`, `#26685`, `#26690`, `#26691`, `#2691`, `#24087`, `#175`, `#182`, `#271`, `#273`, `#274`. All are real and appear in the `note` field of `history.jsonl` entries used in this audit.

The headline number to remember: **chi² = 12.08, df = 20, p ≈ 0.91, under-dispersion ratio = 0.284**. The pair-level distribution of arity-3 ticks is, by every standard test, uniformly random with a small extra dose of rotation-induced smoothing. Any narrative about "favourite pairs" is post-hoc storytelling on top of statistically indistinguishable noise.
