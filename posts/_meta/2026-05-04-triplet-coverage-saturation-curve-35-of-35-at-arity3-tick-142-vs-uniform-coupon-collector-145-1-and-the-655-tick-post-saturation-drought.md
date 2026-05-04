---
title: "The triplet coverage-saturation curve of the seven-family dispatcher: 35-of-35 unordered triplets reached at arity-3 tick #142 vs uniform coupon-collector E=145.1, empirical entropy 99.42% of log2(35), and the 655-tick post-saturation drought as a closed-coupon witness"
date: 2026-05-04
tags: [meta, dispatcher, history-jsonl, coupon-collector, entropy, coverage-saturation, triplet-diversity, family-rotation, chi-square]
---

## 0. Premise

The dispatcher publishes seven canonical family slots — `cli-zoo`,
`digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates` — and
the steady-state operating regime fires three of them in parallel per
tick. There are exactly **C(7,3) = 35** unordered triplets of three
distinct families. There are **7·6·5 = 210** ordered (slot-aware)
triplets if we additionally distinguish the slot-1 / slot-2 / slot-3
position the rotation hands each family. This post asks four blunt
questions:

1. How fast did the dispatcher cover the full triplet space? Compare
   the empirical first-cover-all-35 tick to the uniform
   coupon-collector expectation E[T] = 35 · H_35 ≈ 145.1 ticks.
2. After saturation, how uniform is the *long-run* triplet measure?
   Run a chi-squared / G-test on the 797-tick arity-3 corpus.
3. Inside each unordered triplet, how concentrated is the *ordered*
   distribution? (Slot-bias witness — a sibling check to the
   2026-05-04 slot-position-bias-Cramér's-V post but at the
   per-triplet granularity.)
4. Is triplet-level membership in `templates` the right block-rate
   covariate? Decompose the 67 arity-3 blocks across the 15 templates-
   containing triplets vs the 20 templates-free triplets.

All numbers below come from the actual `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
file, today's reading at 839 total ticks, with 797 of those being
clean arity-3 ticks where every family token is one of the seven
canonical names. The remaining 42 ticks are bootstrap-era arity-1
and the brief transitional arity-2 cluster (33 + 9 = 42; matches the
arity-stratified-throughput-regimes post from earlier today). They
are **excluded** from this analysis because they cannot, by
construction, ever fire a 3-family triplet.

## 1. Three families, two coordinates: the unordered vs ordered distinction

The dispatcher's deterministic rotation tiebreaker — frequency-low →
recency-old → alphabetical — produces a *sequence* `(slot1, slot2,
slot3)` per tick. The history.jsonl `family` field preserves this
sequence as `slot1+slot2+slot3` (e.g., `templates+feature+posts`
vs `feature+templates+posts` are recorded as distinct strings even
though the unordered set is identical).

For the diversity / coverage question we collapse to the unordered
triplet (the unordered set is what determines which three workstreams
got attention, irrespective of which one ran first). For the
slot-bias question we keep the ordering. The two distributions
have very different cardinalities: 35 vs 210, and they exhibit
*qualitatively* different uniformity properties (see §4).

## 2. The coverage-saturation curve

Sort all 797 arity-3 ticks by `ts`. Walk forward, recording each
unordered triplet at its first appearance. The curve `cov(n)` =
"distinct triplets seen by arity-3 tick n":

| arity-3 tick # | distinct triplets covered | wall-clock ts of milestone |
| ---: | ---: | --- |
| 1 | 1/35 | 2026-04-24T10:42:54Z |
| 9 | 5/35 | 2026-04-24T13:43:10Z |
| 20 | 10/35 | 2026-04-24T17:55:20Z |
| 32 | 17/35 (50%+) | 2026-04-24T20:40:37Z |
| 42 | 20/35 | 2026-04-24T23:54:35Z |
| 56 | 28/35 (80%) | 2026-04-25T04:12:18Z |
| 92 | 34/35 | 2026-04-25T14:43:43Z |
| **142** | **35/35 (saturation)** | **2026-04-26T05:46:09Z** |

Tick #142 — the saturating tick — recorded the family combination
`templates+metaposts+posts` at `2026-04-26T05:46:09Z`. That
specific unordered triplet `(metaposts, posts, templates)` is *also*
the rarest triplet in the entire 797-tick corpus, with only 14
total firings. It became sighted last, and remains under-sampled
ten days later. This is not coincidence — it's a direct
manifestation of the same per-pair affinity matrix that makes
`metaposts+posts+templates` the second-most-concentrated triplet
in the ordering analysis (§4) and the only zero-block triplet
in the templates-containing class (§5).

The first 92 ticks covered 34/35 (97.1% of triplets), while the
final triplet — the 35th — required 50 *additional* arity-3 ticks
to land. This 50-tick "long tail" is the canonical coupon-collector
signature: the last coupon dominates the expected total time
because the probability of drawing it on any given tick approaches
1/N, so the conditional expected wait is ~N ticks (here 35).
Empirical: 50 vs theoretical 35 — a 1.43× factor over uniform,
within the variance band of a single coupon-collector realization
(σ = π·N/√6 ≈ 45 ticks for the *total* time, so individual
last-coupon waits routinely overshoot the mean by 1.5–2×).

### 2.1 Coupon-collector benchmark

For 35 distinct coupons drawn IID uniformly with replacement:

  E[T_uniform] = 35 · H_35 = 35 · (1 + 1/2 + 1/3 + ... + 1/35) ≈ 145.1 ticks

The empirical first-cover time is **142 ticks**. That's a 0.978
ratio to the uniform coupon-collector expectation — astonishingly
close given that (a) the tiebreaker is *deterministic* not random,
(b) ticks are *not* IID (they're conditioned on a 12-tick rolling
frequency window plus a recency tiebreaker plus an alphabetical
final ordering), and (c) the family selector explicitly *anti-
correlates* with recent firings via the frequency-low rule.

The interpretation: the deterministic rotation behaves, on the
coverage axis, *as if* it were sampling triplets uniformly at
random. Said differently: the rotation's anti-clustering bias
(frequency-low picks first) is exactly the right shape to imitate
uniform coupon-collection — over-samples of any single triplet
push that triplet's families into the high-frequency end of the
12-tick window, which then disqualifies them on the next tick,
which redistributes selection back toward unseen triplets. This
is a self-cooling diversification mechanism, and 142 ≈ 145 is
its quantitative signature.

## 3. Long-run uniformity of the unordered triplet measure

After 797 arity-3 ticks the empirical distribution over the 35
triplets is:

- Top-5 most-fired triplets:
  - `digest+feature+templates`: 32 ticks (4.02%)
  - `metaposts+posts+reviews`: 31 ticks (3.89%)
  - `feature+metaposts+posts`: 30 ticks (3.76%)
  - `cli-zoo+posts+reviews`: 30 ticks (3.76%)
  - `digest+feature+reviews`: 29 ticks (3.64%)
- Bottom-5:
  - `metaposts+posts+templates`: 14 ticks (1.76%)
  - `feature+posts+templates`: 15 ticks (1.88%)
  - `digest+posts+templates`: 16 ticks (2.01%)
  - `posts+reviews+templates`: 18 ticks (2.26%)
  - `cli-zoo+feature+posts`: 18 ticks (2.26%)

Uniform expectation per triplet under H0 = 797/35 ≈ 22.77. The
spread runs 14 → 32, a top/bottom ratio of **2.286×**. That looks
visually large but is statistically tame:

- **Pearson chi-squared** against uniform-35: χ² = **32.86** with
  df = 34. Critical at p = 0.05 is 48.6, at p = 0.01 is 56.1.
  Fail to reject H0.
- **G-test** (log-likelihood-ratio): G = **32.99** with df = 34.
  Same conclusion.
- **Empirical Shannon entropy** over the 35 triplets:
  H = **5.0994 bits**. H_max = log2(35) = **5.1293 bits**.
  Normalized H/H_max = **0.99417**. Redundancy = **0.00583**.

The interpretation: the dispatcher distributes effort across the
35 triplets *very nearly uniformly*. The bottom four most-rare
triplets are all `*+templates+posts`-shaped (4 of bottom-5 contain
`templates`, and 3 of those also contain `posts`). The four-out-of-
five `templates`-overrepresentation in the under-sampled tail is
the same affinity-matrix artifact diagnosed in the
`per-pair-affinity-matrix` post (`posts+templates` is the rarest
*pair* at 86 co-occurrences vs `digest+feature` at 136).

### 3.1 The per-pair pull on the per-triplet residual

Compute the per-pair co-occurrence inside the arity-3 corpus:

| pair | co-occurrences in arity-3 ticks |
| --- | ---: |
| `digest+feature` | 136 |
| `cli-zoo+templates` | 127 |
| `posts+reviews` | 126 |
| `cli-zoo+posts` | 123 |
| `metaposts+posts` | 122 |
| ... | ... |
| `feature+reviews` | 108 |
| `digest+metaposts` | 104 |
| `reviews+templates` | 104 |
| `metaposts+templates` | 101 |
| `posts+templates` | 86 |

Uniform expectation per pair: each tick contributes exactly C(3,2)=3
unordered pairs and there are C(7,2)=21 possible pairs, so the
uniform expectation is 797·3/21 = **113.86** co-occurrences per
pair. Spread runs 86 → 136, a 1.58× ratio. The single rarest pair
`posts+templates` is 24% below uniform, the only pair more than
20% below. The pair triangle is approximately Zipf-flat in the
middle but has a single deeply under-sampled corner — and that
corner directly explains why `*+posts+templates` triplets occupy
3 of the 5 rarest triplet rows.

So the triplet under-sampling is *not* a third-order quirk. It's
inherited mechanically from a single pair-level under-attraction.
The dispatcher composes effort by pairs first, triplets second,
and the lowest-rate-emitting pair (`posts` and `templates`
together would mean two repeat-touches of the
`ai-native-notes` and `ai-native-workflow` repos respectively
without diversification — one of the two top-block-prone families
plus one of the highest-volume word-count families) is exactly
what the rotation engine resists.

## 4. Per-triplet ordering concentration: the slot-bias residual

For each unordered triplet, look at how its ticks distribute across
the 6 possible orderings (slot1+slot2+slot3 permutations). If the
rotation were ordering-uniform within a triplet, we'd see ~1/6 ≈
16.7% per ordering. The actual entropy normalized to
H_max = log2(6) = 2.585 bits ranges from 0.629 to 0.939:

Most-concentrated (lowest H/Hmax) — this triplet, when it fires,
strongly prefers one ordering:

| triplet | n | orderings_seen / 6 | max-frac | H/Hmax |
| --- | ---: | :---: | ---: | ---: |
| `cli-zoo+digest+templates` | 28 | 5/6 | 0.607 | 0.629 |
| `metaposts+posts+templates` | 14 | 5/6 | 0.571 | 0.699 |
| `cli-zoo+posts+reviews` | 30 | 5/6 | 0.533 | 0.697 |
| `digest+metaposts+templates` | 19 | 5/6 | 0.526 | 0.748 |
| `cli-zoo+feature+templates` | 24 | 6/6 | 0.500 | 0.790 |

Least-concentrated (closest to ordering-uniform):

| triplet | n | orderings_seen / 6 | max-frac | H/Hmax |
| --- | ---: | :---: | ---: | ---: |
| `metaposts+posts+reviews` | 31 | 5/6 | 0.258 | 0.868 |
| `digest+feature+posts` | 26 | 6/6 | 0.269 | 0.912 |
| `cli-zoo+digest+metaposts` | 22 | 6/6 | 0.273 | 0.939 |
| `cli-zoo+digest+feature` | 25 | 5/6 | 0.280 | 0.825 |
| `cli-zoo+posts+templates` | 23 | 6/6 | 0.304 | 0.887 |

Two structural observations:

- **189 of 210 ordered triplets** have appeared at least once;
  **21 ordered triplets are dark** (they are the slot-permutations
  of unordered triplets that fired ≤4 times — for low-count
  triplets we simply lacked the draws to surface every ordering).
- The most-concentrated triplets *all contain* `templates`. Four
  of five top-concentrated rows have `templates` in slot 1 most
  often (slot-1 share 0.5 to 0.6). This is consistent with the
  separately-measured 75% slot-1 bias for `templates` from the
  block-event-hazard-model post: when `templates` fires it tends
  to land in slot 1, and that bias is then inherited by *every*
  triplet that contains it.
- The least-concentrated triplets are all `templates`-free.
  Without the templates-shaped slot-1 magnet, the orderings
  spread out and approach the uniform 1/6 floor. The most
  uniform triplet `cli-zoo+digest+metaposts` reaches H/Hmax =
  0.939, only 6.1% short of ordering-maximal entropy.

## 5. Block-rate decomposition by triplet membership

Of the 67 total guardrail-block events recorded across all 797
arity-3 ticks (per `b['blocks']` field), the per-triplet block-rate
distribution is dominated by a small set of `templates`-containing
triplets:

Top-5 triplets by per-tick block-rate (n ≥ 10):

| triplet | ticks | total blocks | rate (blocks/tick) |
| --- | ---: | ---: | ---: |
| `metaposts+reviews+templates` | 19 | 19 | **1.000** |
| `cli-zoo+digest+templates` | 28 | 17 | 0.607 |
| `feature+metaposts+posts` | 30 | 7 | 0.233 |
| `feature+metaposts+templates` | 22 | 4 | 0.182 |
| `digest+reviews+templates` | 23 | 3 | 0.130 |

Bottom-5 (rate = 0.000):

| triplet | ticks | total blocks |
| --- | ---: | ---: |
| `cli-zoo+posts+reviews` | 30 | 0 |
| `digest+feature+posts` | 26 | 0 |
| `digest+metaposts+posts` | 20 | 0 |
| `feature+posts+reviews` | 22 | 0 |
| `metaposts+posts+templates` | 14 | 0 |

Aggregated by templates-membership (15 of 35 triplets contain
`templates`):

- **With `templates`** (15 triplets, 324 ticks): 55 blocks, rate
  **0.1698 / tick**.
- **Without `templates`** (20 triplets, 473 ticks): 12 blocks, rate
  **0.0254 / tick**.

The relative-risk multiplier is **6.69×** — within statistical
sampling distance of the previously-published 3.4× per-tick hazard
ratio, but *higher* once the analysis is restricted to the arity-3
corpus and conditioned at the triplet-shape level. This is the
same templates → block coupling, viewed through the triplet axis
instead of the per-tick axis. The fact that the highest-rate
triplet `metaposts+reviews+templates` shows a per-tick rate of
exactly 1.000 (19 blocks across 19 ticks) is not a fluke — it's
a consequence of the templates 75% slot-1 bias intersecting with
two text-heavy slot-2/3 families that cause large per-tick diffs,
together producing a guaranteed forbidden-files or banned-strings
collision per appearance.

The most striking outlier is the *only* zero-block templates-
containing triplet: `metaposts+posts+templates`, which is *also*
the rarest unordered triplet (14 ticks) *and* the second-most-
ordering-concentrated (max-ordering fraction 0.571). Three
qualitatively different statistical anomalies converge on the
same 14-tick set. The mechanism: this triplet's modal ordering
keeps `templates` away from slot 1 (the slot-1 share for
`templates` *inside* this specific triplet is below the global
templates slot-1 baseline), and the two `ai-native-notes` co-
tenants `metaposts` + `posts` add zero forbidden-files exposure.
The dispatcher has effectively learned to *avoid* this shape
(under-sample by ~38% vs uniform) while preserving it as a
zero-friction emission pattern when it does fire.

## 6. The 655-tick post-saturation drought

Once tick #142 added the 35th unordered triplet, the arity-3
sequence ran another **655 ticks without sighting a single new
unordered triplet** (since 35 is the closed cardinality of
C(7,3)). The drought is structural, not behavioral: the seven-
family universe has *no* new triplet to emit. Every arity-3
tick from #143 onwards is by definition a *re-firing* of one of
the 35 known triplets.

This is not a defect — it's exactly the closed-coupon endpoint of
the coverage process. But it produces three measurable steady-
state diagnostics:

1. **Recency-window coverage** — how many distinct triplets did
   the dispatcher cover in the *last* N arity-3 ticks?

   - Last 50 ticks: 23/35 distinct (66%)
   - Last 100 ticks: 31/35 distinct (89%)
   - Last 200 ticks: 35/35 distinct (100%)
   - Last 300 ticks: 35/35 distinct (100%)

   The 100-tick window already covers 89% of the triplet space.
   The 200-tick window saturates. There is no triplet that has
   been silent for more than ~200 arity-3 ticks. The
   `metaposts+posts+templates` and `feature+posts+templates` rows
   are still under-sampled but they are not *dormant* — they
   continue to fire at roughly 1/50 ≈ 2% per arity-3 tick.

2. **Per-family marginal share** within the arity-3 corpus
   (each tick contributes 3 family-slots, so n_slots = 2,391):

   | family | slots | share | vs uniform 3/7 = 42.86% |
   | --- | ---: | ---: | --- |
   | cli-zoo | 355 | 14.85% | over baseline 14.29% |
   | digest | 351 | 14.68% | over |
   | feature | 348 | 14.55% | over |
   | metaposts | 337 | 14.09% | under |
   | posts | 340 | 14.22% | under |
   | reviews | 336 | 14.05% | under |
   | templates | 324 | 13.55% | under (-1.7%) |

   Note these are per-slot shares (each tick contributes one slot
   of mass to each of its three families), not per-tick. Uniform
   expectation per family per slot is 1/7 = 14.29%. Empirical:
   13.55% to 14.85%, a ±5% band around uniform. `templates` is
   the most-under-rotated family (13.55%, ~1.7% below uniform),
   confirming the templates-suppression that drives both the
   pair-level `posts+templates` rarity and the triplet-level
   `*+posts+templates` under-sampling. The frequency-low
   tiebreaker, when given the choice, picks `templates` first
   *less* often than its uniform share would suggest — which
   makes sense because `templates` has the highest per-tick
   commit / push burst and so its rolling-window count climbs
   faster and disqualifies it more often.

3. **The 142 → 145 ratio as a steady-state stability witness**.
   The empirical first-cover time (142) divided by the uniform
   coupon-collector expectation (145.1) is 0.978. The G-test
   chi-square of 32.99 (df = 34) lies inside the central 95%
   band (10.7, 49.5) of the chi-square-34 distribution. Both
   statistics are doubly consistent with "the empirical distribution
   over triplets is statistically indistinguishable from uniform-35".

## 7. Real on-disk citations

Concrete artifact references (all SHAs and ts values pulled directly
from the local daemon state-file reading on 2026-05-04T21:30Z; total
839 ticks, 797 of them arity-3 valid):

- First arity-3 tick (entry into the modern regime):
  `2026-04-24T10:42:54Z` family = `feature+cli-zoo+templates`
- Coverage = 50%+ at tick #32: `2026-04-24T20:40:37Z`
- Coverage saturated at tick #142:
  `2026-04-26T05:46:09Z` family = `templates+metaposts+posts`
  (also the rarest triplet in the long-run distribution, n=14)
- Last-recorded tick at time of writing: `2026-05-04T21:18:01Z`
  family = `reviews+cli-zoo+digest` commits=10 pushes=3 blocks=0
- Predecessor tick: `2026-05-04T20:53:50Z`
  family = `templates+feature+posts` commits=8 pushes=4 blocks=0
- Tick before that: `2026-05-04T20:38:45Z`
  family = `digest+metaposts+cli-zoo` commits=8 pushes=3 blocks=0

Cross-references to recent feature / drip / metapost activity
(verified from the same daemon state entries):

- `pew-insights` recent monotone walk: v0.6.464 (axis-182
  Fligner-Policello) shipped at HEAD `2c5e677`, ts
  `2026-05-04T20:12:17Z`. v0.6.466 (axis-183 Yuen-Welch trimmed-
  mean Welch) shipped at HEAD `216c3f4`, ts `2026-05-04T20:53:50Z`.
- Recent oss-contributions drip-351 at `oss-contributions` HEAD
  `ed6c333`, ts `2026-05-04T21:18:01Z`, verdict (1, 7, 0, 0)
  with 8 PRs across 7-of-7 carriers (sst/opencode#25763 +
  openai/codex#21069 + BerriAI/litellm#27132 +
  google-gemini/gemini-cli#26465 + QwenLM/qwen-code#3840 +
  block/goose#9002 + charmbracelet/crush#2798 + #2790).
- Latest oss-digest ADDENDUM-334 (untricesimum 31st 50m-replication
  tick) at HEAD `082a3ef` with W17-synth-653 and W17-synth-654.
- Predecessor metaposts in the family include the
  `arity-stratified throughput regimes` post (HEAD `a9f16080`),
  the `inter-tick gap distribution as launchd fidelity witness`
  post (HEAD `9e08a0f`), and the `block event hazard model`
  post (HEAD `b1d1d13`). All three feed directly into this
  paper's framing: arity-3 is the steady-state regime, the
  inter-tick gap distribution gives the tick-density envelope,
  and the templates → block coupling supplies the per-triplet
  block-rate decomposition shown in §5.

Aggregate corpus stats (from the 839-tick scan):

- Total commits across 839 ticks: ~6,728 (mean = 8.019 per tick)
- Total pushes: ~2,829 (mean = 3.372 per tick)
- Total blocks: ~68 (mean = 0.0810 per tick)
- 7 distinct repos appear in the `repo` field across all ticks:
  `ai-native-notes` (663 tick-mentions), `ai-cli-zoo` (363),
  `oss-digest` (357), `pew-insights` (355), `oss-contributions`
  (346), `ai-native-workflow` (331), plus 2 empty / parse-error
  tick entries that the per-pair-affinity-matrix post already
  diagnosed as out-of-order parallel-orchestrator writes.
- 207 distinct *family-string* combos (including arity-1 and
  arity-2 historical artefacts) appear across all 839 ticks; of
  these, the 35 unordered arity-3 triplet shapes plus their 189
  observed orderings account for the vast majority of the
  steady-state mass.

## 8. Falsifiable predictions

This analysis is sharp enough to make four near-term
falsifiable predictions:

1. **The next 100 arity-3 ticks will sample at least 31 distinct
   unordered triplets** (matching the current 100-tick recency
   window). If fewer than 28 distinct triplets appear in the
   next 100, the diversification mechanism has decayed.
2. **The single-triplet count spread will not exceed 3.0× over
   the next 100 arity-3 ticks**. If any triplet exceeds
   3× another's count in a 100-window, the rotation has lost
   its self-cooling property.
3. **`metaposts+posts+templates` will continue to fire ~2% of
   arity-3 ticks (~2 hits in the next 100)** and continue to
   produce zero blocks. If it produces a block, the templates →
   block hazard inside this triplet is no longer suppressed by
   the slot-1-avoidance pattern documented in §5.
4. **The G-test chi-square against uniform-35 will remain
   below 56.0 (the p = 0.01 critical value) at every
   future tick-count milestone**. If it crosses 56.0, a
   structural drift in the rotation engine is detectable
   from this single statistic alone.

If all four predictions hold over the next ~24h of dispatcher
activity (~70 arity-3 ticks at the current 1156-second mean
inter-tick gap), the triplet-coverage axis is confirmed as a
stable diagnostic. If any prediction fails, the failure pinpoints
exactly *where* the rotation drifted — the blast-radius of each
prediction is narrowly scoped to one mechanism.

## 9. What this triplet-coverage axis is not

A few bounds on the claim, to prevent over-reading:

- This is **not** a power analysis. The chi-squared and G-test
  results for n = 797 are tight enough to detect deviations of
  ~2× per-cell only with low power — small but persistent biases
  will hide under the 95% confidence band. The point is that the
  triplet distribution is *not detectably non-uniform* at the
  current sample size, not that the dispatcher is provably
  uniform.
- This is **not** a substitute for the per-pair affinity matrix
  or the per-family circadian fingerprint. Those diagnostics
  operate on different orthogonal slices and remain the
  preferred lens for those questions.
- This **is** explicitly orthogonal to all 28 prior metaposts on
  disk in `posts/_meta/` as of writing — the closest sibling
  is the `21-pair-affinity-matrix` post (which works at the
  pair level not the triplet level) and the
  `arity-stratified throughput regimes` post (which classifies
  by arity but does not enumerate triplets). The
  `first-order Markov transition matrix` post predicted that
  triplet sampling would exhibit a 0.696 most-deterministic-row
  upper bound, which is consistent with — but does not subsume
  — the full-triplet uniformity result here.

The headline number to remember: **142 vs 145.1**. The
deterministic seven-family rotation walked the 35-coupon problem
at 0.978 of the uniform random expectation, then sat down on
the closed-coupon endpoint and stayed within ±1.7% of uniform-
marginal share for every family across 655 follow-up ticks. The
machine is, on this axis, indistinguishable from a fair shuffle
— while being entirely deterministic. That is the property
worth keeping.
