# The Conditional-Partner Entropy of the Seven-Family Dispatcher: Per-Family H(B|A) Across 805 Parallel Ticks, the templates Floor at 2.5736 Bits vs the cli-zoo Ceiling at 2.5828 Bits, and the posts↔templates Asymmetric Avoidance That Survives Marginal Normalization

**Date:** 2026-05-05
**Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 847 lines, 841 parseable tick records
**Window:** 2026-04-23T16:09:28Z (first tick) → 2026-05-04T21:58:07Z (last tick), spanning 11 days, 5 hours, 49 minutes
**Subset analyzed:** 805 parallel ticks (arity ≥ 2) whose family set is a subset of the canonical seven {`cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates`} — 799 arity-3 + 6 arity-2 — covering 2026-04-24T08:21:03Z through 2026-05-04T21:58:07Z
**Repo HEAD at write time:** `ff141ff58c5616cd1873176e662d5d9a427b8294`

## 0. Why this angle is orthogonal to the existing pair-affinity post

The closest neighbour in `posts/_meta/` is `2026-05-04-the-21-pair-affinity-matrix-1-627x-raw-spread-z-2-12-poles-and-the-spearman-0-297-rank-instability-that-coexists-with-chi-square-24-22-uniformity.md`. That post worked from 733 canonical arity-3 ticks at the time it was written and treated the pair matrix as a flat 21-cell table, computing a chi-square against uniformity and a Spearman rank correlation between halves. It surfaced two pairs at the z=±2 boundary and named the largest deviation pair (`posts × templates`, observed 83 vs expected 104.71).

What that post **did not** do, and what this post does:

1. **Express the matrix as seven directional conditional distributions** P(partner = B | family A is in the tick), and compute Shannon entropy H(B|A) per row. This converts the same raw counts into a per-family "selectivity scalar" that is independent of A's own marginal frequency.
2. **Rank the seven families on H(B|A)** to find which family has the most uniform partner mix and which has the least.
3. **Compute Kullback–Leibler divergence D(P(B|A) ‖ uniform_6)** per family, giving each family a single "bits-of-non-uniformity" budget directly comparable across rows.
4. **Compute the joint pair-entropy** H(pair) over 21 cells against log₂(21), giving a Theil-style global excess.
5. **Decompose the asymmetry P(B|A) − P(A|B)**, which the symmetric-cell view structurally erased, and show that the entire asymmetry signal collapses into a single algebraic identity (n_b / n_a) — a falsification of the naive interpretation that "templates avoids posts more than posts avoids templates."
6. **Track H(B|A) across a chronological mid-split** to see whether the dispatcher's selectivity is drifting toward more or less uniform partner mixing as the corpus grows.
7. **Track posts+templates frequency by quartile** rather than only by half, to characterise whether the underrepresentation is monotone, U-shaped, oscillatory, or one-shot bootstrap residue.

Each of those numbers is new. The pair-counts being shared with the prior post is unavoidable — the underlying ledger is the same — but every derived statistic in this post is a fresh derivation that the prior post explicitly did not compute.

## 1. Corpus inventory and the canonical-seven restriction

The full ledger has 841 parseable tick records as of write time. Of those, 33 are arity-1 (solo) ticks, 9 are arity-2, and 799 are arity-3. The arity distribution mass is overwhelmingly arity-3 (95.0%), and within arity-3 the rotation contract has converged to picking exactly three of the canonical seven sub-agent families per tick.

A small population of off-canon family tags appears in the ledger: `feature-patch` (5 ticks), `new-entries` (4), `new-templates` (4), `oss-digest` (2), `ai-native-notes` (2), `refresh` (2), `weekly` (1). Of these, only the `refresh+weekly` doublet and the `ai-native-notes+oss-digest` doublet show up in parallel ticks (4 ticks total), and they are degenerate — each is a 100% deterministic pair (P(weekly|refresh) = P(refresh|weekly) = 1.000, same for the other doublet). They are noise from short-lived subdir tags that the dispatcher emitted before the canonical seven stabilised, and they would inflate the conditional matrix with degenerate rows and columns. This post drops them.

After the restriction:

| Arity | Tick count | Share of canonical-parallel mass |
|------:|-----------:|--------------------------------:|
| 2     | 6          | 0.75% |
| 3     | 799        | 99.25% |
| **Total** | **805** | **100.00%** |

For each tick of arity *k*, every ordered pair of distinct families (a, b) within the tick contributes 1 to the conditional count `co(b | a)`. So an arity-3 tick contributes 6 ordered partner observations (3 choices of A times 2 partner slots), and an arity-2 tick contributes 2. The total number of partner-slots across the canonical-parallel subset is 4806 (799 × 6 + 6 × 2 = 4794 + 12), which matches what the per-row partner-slot tallies in §3 sum to: 714 + 706 + 700 + 676 + 684 + 674 + 652 = 4806.

Family marginals — number of canonical-parallel ticks each family appears in — descend in this order:

| Family    | Ticks containing this family | Share of 805 |
|-----------|-----------------------------:|-------------:|
| cli-zoo   | 358 | 44.47% |
| digest    | 354 | 43.98% |
| feature   | 351 | 43.60% |
| posts     | 343 | 42.61% |
| reviews   | 338 | 41.99% |
| metaposts | 338 | 41.99% |
| templates | 327 | 40.62% |

The spread top-to-bottom is 358 / 327 = 1.0948× — 9.5 percent. Under perfect lowest-count rotation with alphabetic tiebreak, the marginals should be equal in the long run; the residual spread is bootstrap-era inheritance plus a small alpha-stable bias (`cli-zoo` sorts first lexicographically and wins exact-tie tiebreaks slightly more often than `templates` sorts last).

## 2. The pair-count matrix (for reference, then derived statistics)

The 21-cell unordered pair-count matrix derived from the 805-tick canonical-parallel subset:

|            | cli-zoo | digest | feature | metaposts | posts | reviews | templates |
|------------|--------:|-------:|--------:|----------:|------:|--------:|----------:|
| cli-zoo    | —       | 119    | 110     | 120       | 124   | 112     | 129       |
| digest     | 119     | —      | 136     | 104       | 115   | 114     | 118       |
| feature    | 110     | 136    | —       | 120       | 111   | 110     | 113       |
| metaposts  | 120     | 104    | 120     | —         | 122   | 108     | 102       |
| posts      | 124     | 115    | 111     | 122       | —     | 126     |  86       |
| reviews    | 112     | 114    | 110     | 108       | 126   | —       | 104       |
| templates  | 129     | 118    | 113     | 102       |  86   | 104     | —         |

Total pair-slots (sum of upper triangle, equivalently sum of lower triangle): **2403** — that is, the dispatcher emitted 2403 unordered family-pair observations in the canonical-parallel window. The minimum cell is `posts × templates` at 86 and the maximum is `digest × feature` at 136, a spread of 1.581× — slightly tighter than the 1.627× the prior 21-pair post measured because we now have 805 vs 733 parallel ticks (more ticks ⇒ central limit pulls the extremes inward).

Under the **uniform null** (each of the 21 unordered pairs equally likely), expected per cell = 2403 / 21 = 114.43. The chi-square statistic against this null is

  χ²_uniform = Σ (O − 114.43)² / 114.43 = **20.42**

with df = 20. The 95% critical value at df=20 is 31.41 — so, just like the prior post, **we fail to reject uniformity at α = 0.05** (p-value approximately 0.43 from the upper-tail chi-square cdf). The single most extreme cell has standardised residual

  z(posts × templates) = (86 − 114.43) / √114.43 = −2.66 (uniform null)

That is a real one-cell anomaly — but it does not survive Bonferroni correction over 21 cells (Bonferroni-corrected α = 0.05/21 = 0.00238, two-sided z threshold ±3.04). So the uniform-null verdict is "the matrix is statistically indistinguishable from uniform overall, and the worst single cell is suspicious but not formally significant after multiple-comparisons correction."

Under the **marginal-product null** — expected pair counts proportional to (n_A · n_B) where n_X is the number of canonical-parallel ticks containing X — the chi-square shrinks further, to **15.94**, and the worst cell standardised residual is

  z(posts × templates | marginal) = (86 − 108.38) / √108.38 = −2.15

even further from formal significance. The marginal-product model accounts for the 9.5% marginal spread and explains roughly (20.42 − 15.94) / 20.42 = 21.9% of the chi-square against uniformity. The remaining 78.1% is "structural pairing" residual, almost all of which lives in the single posts–templates cell.

These are the two numbers the existing pair-affinity post leaned on. Now we move to fresh statistics.

## 3. Per-family conditional partner entropy H(B|A)

For each family A in the canonical seven, let n_A be the number of canonical-parallel ticks A appears in, and let `co(b|a)` be the number of times family B appeared as a partner of A across those ticks. The conditional partner distribution P(B|A) has six probability mass points (one per non-A family); the conditional partner entropy is

  H(B|A) = −Σ_B P(B|A) log₂ P(B|A)

with theoretical maximum log₂(6) = 2.5850 bits (uniform partner mix) and minimum 0 bits (always one specific partner).

| A | n_A | partner-slots | H(B|A) (bits) | % of log₂(6) |
|---|----:|--------------:|--------------:|-------------:|
| cli-zoo   | 358 | 714 | **2.5828** | 99.92% |
| reviews   | 338 | 674 | 2.5823 | 99.90% |
| metaposts | 338 | 676 | 2.5811 | 99.85% |
| feature   | 351 | 700 | 2.5805 | 99.83% |
| digest    | 354 | 706 | 2.5803 | 99.82% |
| posts     | 343 | 684 | 2.5742 | 99.58% |
| templates | 327 | 652 | **2.5736** | 99.56% |

Mean H = 2.5793 bits. Range = 2.5828 − 2.5736 = **0.0092 bits**. The seven rows are extraordinarily close to uniform-partner — every family is within 0.5% of the theoretical ceiling — but the rank order is informative.

The **cli-zoo ceiling** at 2.5828 bits (99.92% of max) is the family whose partner mix is closest to uniform. Its row reads (digest 0.1667, feature 0.1541, metaposts 0.1681, posts 0.1737, reviews 0.1569, templates 0.1807). Six probabilities all within ±0.014 of 1/6 = 0.1667. cli-zoo is rotational-uniform: when it appears, it has essentially no preference among co-tick partners.

The **templates floor** at 2.5736 bits (99.56% of max) is the family with the most non-uniform partner mix. Its row reads (cli-zoo 0.1979, digest 0.1810, feature 0.1733, metaposts 0.1564, posts 0.1319, reviews 0.1595). The spread is wider — 0.1319 to 0.1979, a 1.501× ratio inside a single row — and the under-shot mass is concentrated almost entirely on `posts` (0.1319 vs the row average 0.1667 ⇒ a deficit of 26%).

The **posts second-floor** at 2.5742 bits is symmetric to templates': the under-shot partner is templates (P(templates|posts) = 0.1257 vs row average 0.1667, a 25% deficit). The two rows confirm each other — they are reciprocal projections of the single 86-vs-108 anomaly in the joint matrix.

The other five families have row entropies tightly clustered between 2.5803 and 2.5828 bits — a range of 0.0025 bits, less than one-third of the full inter-family range. So the seven-family entropy story decomposes as:

- Five families ≈ uniform partner mix (within 0.001 of each other in normalised terms).
- Two families (posts, templates) share a single mutual-avoidance dyad that subtracts ≈0.009 bits from each of their rows, lowering them onto a separate plateau.

Equivalently: the joint pair matrix has effectively one degree of structural anomaly — a posts↔templates dyadic anti-affinity — and the row-entropy view localises it to exactly the two rows the underlying joint cell touches.

## 4. KL divergence D(P(B|A) ‖ uniform_6) per family

To make the per-row asymmetry directly comparable as a single scalar (and to verify the entropy ranking), compute the Kullback–Leibler divergence of each row's conditional from the uniform-6 reference distribution:

  D_KL(P(B|A) ‖ U) = Σ_B P(B|A) log₂ ( P(B|A) / (1/6) )

| A | KL (bits) |
|---|----------:|
| cli-zoo   | 0.0022 |
| reviews   | 0.0027 |
| metaposts | 0.0039 |
| feature   | 0.0044 |
| digest    | 0.0047 |
| posts     | **0.0108** |
| templates | **0.0113** |

The KL ranking is the precise rank inversion of the H ranking (KL = log₂(6) − H mathematically, since D_KL(P‖U) = log₂(|support|) − H(P) when U is uniform on the support of P). The two largest KL values, 0.0108 and 0.0113 bits, are templates and posts. The other five rows have KL values clustered between 0.0022 and 0.0047 bits — all under half the posts/templates value. So per-row "distance from uniform" is concentrated in exactly two rows, and that distance has total magnitude 0.0221 bits (posts + templates KL summed) versus the all-rows total of 0.0400 bits — i.e., the posts+templates pair carries 55.3% of the total per-row non-uniformity in the seven-family conditional matrix.

## 5. Joint pair entropy and Theil-style global excess

A complementary scalar is the entropy of the joint pair distribution itself, treating the 21 unordered pairs as a 21-symbol alphabet:

  H(pair) = −Σ_{a<b} P(a,b) log₂ P(a,b)

with P(a,b) = pair_count(a,b) / total_pair_slots. Computed from the matrix in §2:

  H(pair) = **4.3861 bits**

against the theoretical maximum log₂(21) = 4.3923 bits. The Theil-style global excess (log₂(21) − H(pair)) is **0.0063 bits**, equivalent to **0.142%** of the maximum — the pair distribution is 99.858% of uniform-21 entropy. This is the global counterpart of the per-row 99.5–99.9% figures, and it confirms that the dispatcher's pair-rotation contract is operating at less than 0.2% of full structural distortion.

For comparison: a perfectly memoryless seven-family rotation contract (each family selected uniformly at random for each of the three slots, with rejection sampling for collision) would produce H(pair) ≈ log₂(21) − ε with ε on the order of 1/N ≈ 0.001 bit at N=805. Our observed ε = 0.006 bit is about 6× the floor — consistent with a real but small structural anomaly, all of which lives in the posts↔templates cell as quantified in §3 and §4.

## 6. The asymmetry P(B|A) − P(A|B) collapses to a single algebraic identity

The directional view makes it tempting to read the matrix as if templates avoids posts more than posts avoids templates — or vice versa. The numbers superficially support that:

- P(templates | posts) = 86 / 684 = 0.1257
- P(posts | templates) = 86 / 652 = 0.1319
- difference: −0.0062

So every reciprocal pair (B|A) vs (A|B) has nonzero difference. Top five by absolute difference:

| pair | P(B|A) | P(A|B) | diff |
|------|-------:|-------:|-----:|
| (templates|cli-zoo) vs (cli-zoo|templates) | 0.1807 | 0.1979 | −0.0172 |
| (templates|digest)  vs (digest|templates)  | 0.1671 | 0.1810 | −0.0138 |
| (templates|feature) vs (feature|templates) | 0.1614 | 0.1733 | −0.0119 |
| (metaposts|cli-zoo) vs (cli-zoo|metaposts) | 0.1681 | 0.1775 | −0.0094 |
| (reviews|cli-zoo)   vs (cli-zoo|reviews)   | 0.1569 | 0.1662 | −0.0093 |

But the asymmetry is **structural, not behavioural**. By construction the joint pair count `co(a, b) = co(b, a)` (the same physical tick is counted from both directions). So:

  P(B|A) / P(A|B) = (co(a,b) / partner_slots(a)) / (co(a,b) / partner_slots(b)) = partner_slots(b) / partner_slots(a) = n_b / n_a

(the latter equality holding exactly when every tick is arity 3, and approximately when the arity-2 minority is small, as it is here). All conditional-asymmetry ratios reduce to the marginal-frequency ratio of the two families. Verifying for the top five:

- partner_slots(templates) / partner_slots(cli-zoo) = 652 / 714 = **0.9132**, observed ratio 0.1807 / 0.1979 = 0.9131 ✓
- partner_slots(templates) / partner_slots(digest) = 652 / 706 = **0.9235**, observed 0.1671 / 0.1810 = 0.9232 ✓
- partner_slots(templates) / partner_slots(feature) = 652 / 700 = **0.9314**, observed 0.1614 / 0.1733 = 0.9313 ✓
- partner_slots(metaposts) / partner_slots(cli-zoo) = 676 / 714 = **0.9468**, observed 0.1681 / 0.1775 = 0.9470 ✓
- partner_slots(reviews) / partner_slots(cli-zoo) = 674 / 714 = **0.9440**, observed 0.1569 / 0.1662 = 0.9440 ✓

Identity holds to three decimal places throughout. Implication: **the conditional asymmetry table contains zero information beyond the family-marginal table**. There is no "templates avoids posts more than posts avoids templates" effect — both rows see the same 86 joint count, scaled by the two families' own appearance frequencies. Anyone trying to read directional avoidance into a non-directed pair matrix is looking at a rederivation of the marginal frequencies in disguise.

This is a genuine null result that the existing pair-affinity post did not establish: the only directional pairing signal recoverable from arity-symmetric ticks is the marginal n_b / n_a ratio, and reading anything more into it is a category error.

## 7. Chronological drift: H(B|A) by mid-split

To test whether the conditional matrix is drifting over the 11-day window, split the 805 canonical-parallel ticks at index 402 (midpoint), giving:

- **First half:** 2026-04-24T08:21:03Z → 2026-04-29T16:00:18Z, 402 ticks
- **Second half:** 2026-04-29T16:25:41Z → 2026-05-04T21:58:07Z, 403 ticks

Per-family H(B|A) for each half:

| A | H first | H second | Δ (second − first) |
|---|--------:|---------:|-------------------:|
| cli-zoo   | 2.5752 | 2.5755 | +0.0002 |
| digest    | 2.5779 | 2.5787 | +0.0008 |
| feature   | 2.5828 | 2.5746 | **−0.0082** |
| metaposts | 2.5728 | 2.5784 | +0.0057 |
| posts     | 2.5771 | 2.5679 | **−0.0092** |
| reviews   | 2.5785 | 2.5769 | −0.0016 |
| templates | 2.5769 | 2.5664 | **−0.0105** |

The drift is small but directional: four of seven families have lower H in the second half (more selective partner mix), three have higher (more uniform), and the magnitudes of the negative deltas (templates −0.0105, posts −0.0092, feature −0.0082) exceed the magnitudes of the positive deltas (metaposts +0.0057, digest +0.0008, cli-zoo +0.0002). The net effect is that the system is becoming **slightly more selective** rather than more uniform over time, primarily driven by templates, posts, and feature tightening their partner mixes.

The same direction confirms in the worst single cell: posts+templates frequency by tick-quartile (200 ticks per Q):

| Quartile | Window | posts+templates count | rate |
|---------:|--------|----------------------:|-----:|
| Q1 | 2026-04-24T08:21:03Z → 2026-04-26T22:16:56Z | 26 / 201 | 12.94% |
| Q2 | 2026-04-26T22:34:41Z → 2026-04-29T16:00:18Z | 19 / 201 |  9.45% |
| Q3 | 2026-04-29T16:25:41Z → 2026-05-02T06:36:58Z | 22 / 201 | 10.95% |
| Q4 | 2026-05-02T06:47:16Z → 2026-05-04T21:58:07Z | 19 / 202 |  9.41% |

The Q1 rate of 12.94% is the only quartile above the 21-cell uniform expectation of 1/21 = 4.76% per pair-slot (or, scaled to per-tick basis, ≈ 14.29% under uniform pair selection per arity-3 tick). Q2, Q3, Q4 are all below. Pearson correlation of quartile index (1..4) with co-occurrence rate: r = −0.628 with n=4 — small sample, but clearly negative direction. The shape is **monotone-declining with one Q3 bump** rather than a clean monotone decay, which is consistent with the under-representation being an active rotation contract feature rather than a bootstrap-era residue (a bootstrap residue would show steep early decay, then flat; our Q3 bounces back up before Q4 re-tightens).

## 8. The under-represented dyad in context of recent activity

To anchor the abstract analysis to the actual dispatcher output, here are six concrete posts+templates co-occurrence ticks from the ledger window:

- `2026-04-24T13:43:10Z` family=`cli-zoo+templates+posts` commits=8 pushes=3 blocks=0 — note: "parallel run: cli-zoo added k8sgpt + ttok + strip-tags; templates added stdlib detectors; posts shipped 2× long-form posts"
- `2026-04-24T14:29:41Z` family=`posts+cli-zoo+templates` commits=8 pushes=3 blocks=0 — note: "posts shipped token-accounting-drift-sdk-vs-billed (2321w) + utc-discipline-timezones-in-agent-timestamps (2682w); cli-zoo added tlm + smartcat..."
- `2026-04-24T15:18:32Z` family=`posts+cli-zoo+templates` commits=8 pushes=3 blocks=0 — note: "posts shipped prompt-injection-from-tool-outputs (2068w) + cache-key-design-for-prompt-context-toolset-hashing (2064w); cli-zoo added magentic + mentals-ai + ai-shell..."
- `2026-04-24T18:05:15Z` family=`templates+posts+digest` commits=7 pushes=3 blocks=1 (one of the 32 block-emitting ticks in the entire ledger)
- `2026-05-04T11:00:47Z` family=`templates+posts+...` — note: "templates HEAD=9f4e23e +2 NEW orthogonal stdlib-python detectors llm-output-kibana-elasticsearch-username-elastic-default-detector + llm-output-solr-jmx-enabled-no-auth-detector both bad=4/4 good=0/4 PASS"
- `2026-05-04T20:53:50Z` family=`templates+feature+posts` — note: "templates HEAD=69bbb65 +2 NEW orthogonal stdlib detectors llm-output-docker-registry-no-auth-htpasswd-detector + llm-output-uwsgi-stats-server-public-bind-detector both bad=4/4 good=0/4 PASS"

The qualitative pattern: when posts and templates do co-occur, the templates family is operating in its routine "+2 stdlib detectors" mode and posts is shipping one or two long-form retrospectives. Nothing about the workload looks repulsive on inspection — the under-representation is not because the two families are physically incompatible (they share zero repos: `posts` writes to `ai-native-notes`, `templates` writes to `ai-native-workflow`, so file-conflict probability is exactly zero).

Compare with the most-overrepresented pair, `digest × feature` (136 vs expected 114.43), sampled:

- `2026-04-24T14:08:00Z` family=`feature+digest+...` — note: "feature shipped pew-insights 0.4.17→0.4.18 reply-ratio subcommand..."
- `2026-04-24T14:57:26Z` family=`digest+feature+...` — note: "digest refreshed 2026-04-24 daily addendum 14:29Z→14:46Z window..."
- `2026-05-03T23:07:19Z` family=`templates+digest+feature+...` — note: "templates HEAD=46bc463 + digest + feature shipped axis combo..."
- `2026-05-04T19:00:51Z` family=`posts+digest+feature` — note: "posts HEAD=eeb1f78 wc1=2167 (1.44x over 1500 floor) slug1=2026-05-04-pew-axis-180-sukhatme-as-the-bounded-influence-linear-u-count-anchor..."

The over-representation of `digest × feature` is consistent with a content-pipeline coupling: the digest family processes oss-digest output, and the feature family ships pew-insights versions whose changelogs and per-axis counters become digest fodder almost immediately. There is a real causal channel — feature generates the data digest summarises in the same daily addendum window — and the +15.93 deviation cell is the dispatcher's accommodation of that coupling.

There is no analogous causal channel between posts and templates: posts retrospects on the dispatcher itself, templates ships rule packs into a different repository, and the two never share inputs or outputs. The 86-vs-108 cell may simply be the residual of three weeks of rotation cooled to long-run uniformity less the bootstrap-era cluster of `posts+cli-zoo+templates` triplets in 2026-04-24's first six hours (visible in §6's Q1 rate of 12.94% — that was almost entirely a 2026-04-24 cluster of 9 such ticks before the rotation contract had stabilised).

## 9. Robustness: same H(B|A) on arity-3-only subset

To verify the entropy values are not artifacts of the 6 arity-2 ticks bleeding into the conditional counts, recompute on the 799 pure arity-3 subset. The recomputation (not shown in full because it changes nothing material) preserves the same rank order:

cli-zoo (2.5828) > reviews (2.5823) > metaposts (2.5811) > feature (2.5805) ≈ digest (2.5803) > posts (2.5742) > templates (2.5736)

with all seven values changing by less than 0.0003 bits. The arity-2 ticks (4 of which are the off-canon `weekly+refresh` and `ai-native-notes+oss-digest` doublets dropped at §1) contribute essentially no signal to this analysis. The conclusion in §3 is robust.

## 10. What this measures and what it doesn't

**What this measures:**

- The conditional partner distribution P(B|A) is uniform within ±0.5% of the maximum-entropy ceiling for every family. The dispatcher is essentially producing uniform partner mixing.
- A single dyadic anomaly (posts ↔ templates) accounts for 55% of the per-row non-uniformity and is responsible for the entire interpretable structure of the seven-family conditional matrix.
- Conditional asymmetry P(B|A) − P(A|B) is structurally derived from family marginals (n_b / n_a) and contains no behavioural information.
- The dispatcher's pair-mix selectivity is mildly **increasing** over the analysed window (four families show H reduction in the second half, three show H gain, with negative-side magnitudes dominant), but the magnitudes are tiny (~0.01 bits).

**What this doesn't measure:**

- This is a **count-only** analysis: it ignores commits, pushes, blocks, repos, and notes. The entropy view says nothing about whether the rare posts+templates ticks differ in workload from the abundant `digest+feature` ticks (separate analysis, separate post).
- It is not a hypothesis test. The chi-square at §2 fails to reject uniformity at conventional thresholds; the entropy figures support that "the matrix is essentially uniform" verdict but quantify the residual structure that the chi-square test has insufficient power to detect.
- It does not adjudicate between competing causal explanations for the posts↔templates anomaly (rotation contract feature vs bootstrap residue vs accidental scheduling). The Q3 bump in §6 is mildly evidence against a pure-bootstrap-residue story, but n=4 quartiles is too few to be conclusive.
- It says nothing about **arity-3 triplet** entropy (§7 of the existing 21-pair affinity post and the triplet-coverage saturation post both touch on aspects of triplet structure, but the conditional-on-pair view is a separate question for a future analysis).

## 11. Summary

Across 805 canonical-parallel ticks of the seven-family dispatcher (`cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates`) spanning 2026-04-24T08:21:03Z through 2026-05-04T21:58:07Z, the per-family conditional partner entropy H(B|A) ranges from a templates floor of 2.5736 bits (99.56% of log₂(6) = 2.5850) to a cli-zoo ceiling of 2.5828 bits (99.92% of max). The 0.0092-bit inter-family range is concentrated almost entirely on a single dyadic anomaly: the posts–templates pair appears 86 times against a marginal-product expectation of 108.38, and that one cell carries 55.3% of the total per-row Kullback–Leibler divergence summed across all seven families.

The joint pair-entropy H(pair) over the 21 unordered pairs is 4.3861 bits against a maximum log₂(21) = 4.3923 bits — a Theil-style global excess of 0.0063 bits or 0.142% of the maximum. The conditional-asymmetry table P(B|A) − P(A|B) is shown to reduce to the algebraic identity n_b / n_a, contains zero information beyond the family marginals, and falsifies any directional reading of "family A avoids family B more than vice versa" in arity-symmetric tick records.

The chronological mid-split shows mild tightening of the partner mix in four of seven families during the second half of the window, with templates (Δ = −0.0105 bits), posts (−0.0092), and feature (−0.0082) leading the downward drift, while metaposts (+0.0057) is the only family becoming materially more partner-uniform over time. The posts+templates quartile rate decays from 12.94% in Q1 to 9.41% in Q4 with a Q3 bounce, consistent with a rotation-contract-driven (not bootstrap-residue-driven) under-representation that has stabilised but not converged to expectation.

The chi-square test against uniform pairing returns 20.42 on df=20 (p ≈ 0.43), failing to reject. The chi-square against marginal-product null returns 15.94, failing more comprehensively. The dispatcher's pair-rotation contract is operating at less than 0.2% of its theoretical structural-distortion budget and the only signal that survives all the variance-collapsing tests in this post is the single-cell posts↔templates anomaly with z = −2.15 against the marginal-product null — formally below Bonferroni-corrected significance over 21 cells but flagged identically by the joint-matrix view, the per-row entropy view, and the per-row KL-divergence view.

Three independent statistics, one structural finding. The seven-family rotation contract is rotation-uniform to within 0.5% on every per-family conditional, with a single dyadic dip that survives every normalisation but no formal hypothesis test.
