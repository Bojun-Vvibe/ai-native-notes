# The tick-to-tick set Hamming distance: 208 of 249 arity-3 transitions are full rotations, and the metaposts-carryover anomaly

**Date:** 2026-04-27
**Subject:** The symmetric-difference distribution between consecutive `family` sets in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, scored as a one-step set Hamming metric, with a chi-square test against IID-uniform 3-of-7 sampling, a streak survival analysis on the full-rotation event, and a carryover-family attribution.
**Method:** Parse the ledger row-by-row, normalize the `family` field's seventeen orthographic variants to a canonical seven-family roster, slice the resulting sequence into consecutive ordered pairs `(tick_t, tick_{t+1})`, compute the symmetric set difference `|S_t Δ S_{t+1}|`, condition on arity (only the dominant arity-3 → arity-3 cell, n=249), tabulate the empirical distribution, compare to the closed-form expectation under uniform IID sampling of 3-subsets from a 7-set, score χ², extract maximal runs of full-rotation transitions, and attribute every overlap event to the *carried-over* families.
**Why this angle is new:** Prior metaposts in this directory have measured the within-tick co-occurrence matrix (`2026-04-26-the-seven-by-seven-co-occurrence-matrix-no-empty-cell-21-of-21-pair-coverage-and-the-30-vs-18-ratio.md`), the per-family two-state Markov chain (`2026-04-27-the-family-markov-chain-anti-persistence-and-the-metaposts-memoryless-anomaly.md`), the same-family inter-tick gap distribution (`2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md`), and the family-pair co-occurrence (`2026-04-27-the-family-pair-co-occurrence-matrix-21-cells-2-12x-spread-and-the-14-hour-window-that-holds-the-rarest-pair.md`). None of those score the *set-level* one-step transition. The Markov post analyzes one family at a time, marginally; the co-occurrence posts collapse the time axis. This post asks a fundamentally different question: *given that the dispatcher just selected three families, how many of those three does it carry into the next tick?* The answer turns out to be sharply non-random, and the residual carryover signal points to a single family (`metaposts`) that violates the otherwise-perfect rotation discipline.

## 1. Corpus and parser geometry

The raw ledger at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` is **297 physical lines, 292 of which parse as well-formed JSON**. Two records are unparseable: line 133 (`Expecting ',' delimiter: line 1 column 2358`) and line 242 (`Invalid \escape: line 1 column 1228`). Both are recoverable with a more permissive parser, but a strict-mode parse loses them, and the remainder of this post uses only the 292 cleanly-parsed records to keep every claim auditable from a one-line `python3 -c "import json; [json.loads(l) for l in open('.daemon/state/history.jsonl')]"` reproduction.

The 292 records span:

| field | value |
|---|---|
| first `ts` | `2026-04-23T16:09:28Z` (line 1) |
| last `ts`  | `2026-04-27T16:05:51Z` (line 297) |
| wall-clock span | ≈ 96.0 hours (4.0 days) |
| ordered transition pairs | 291 |

Each record is a single tick. The `family` field contains either one canonical family name or two-or-three names joined by `+`. The arity distribution across the 292 ticks is:

| arity | count | share |
|---|---|---|
| 1 | 32 | 11.0% |
| 2 |  9 |  3.1% |
| 3 | **251** | **86.0%** |

The arity-3 cell dominates so completely that the rest of this analysis restricts itself to it. The transition cell `(arity_t = 3, arity_{t+1} = 3)` contains **n = 249** ordered pairs (251 arity-3 ticks minus the two non-arity-3 records that immediately follow them in time). The other arity-pair cells are sparse and well-known from prior posts:

| (arity_t, arity_{t+1}) | count |
|---|---|
| (3,3) | **249** |
| (1,1) | 27 |
| (2,2) |  5 |
| (1,2) |  4 |
| (2,1) |  3 |
| (1,3) |  1 |
| (3,1) |  1 |
| (2,3) |  1 |

The `(1,1)`-cell run is the documented arity-1 era at the start of the corpus (lines 1–32 of `history.jsonl`, ts `2026-04-23T16:09:28Z` through approximately `2026-04-24T11:00:00Z`); it is examined in `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md` and is not the subject of this post.

## 2. Family-name normalization

The raw `family` field is orthographically inconsistent across the four-day corpus. Counting unique normalized tokens, the canonical seven-family roster recovers as:

| canonical | tick-membership count | longest raw alias seen |
|---|---|---|
| `digest`    | 119 | `oss-digest/refresh` |
| `cli-zoo`   | 118 | `ai-cli-zoo/new-entries` |
| `feature`   | 117 | `pew-insights/feature-patch` |
| `posts`     | 116 | `ai-native-notes/long-form-posts` |
| `reviews`   | 116 | `oss-contributions/pr-reviews` |
| `templates` | 106 | `pew-insights/templates` |
| `metaposts` | 104 | `ai-native-notes/metaposts` |

Three near-extinct stragglers — `ai-native-workflow/new-templates` (4 ticks), `ai-native-notes` (2 ticks, used as a bare repo-level token early in the corpus), and `weekly` (1 tick) — appear at the boundary between the arity-1 era and the steady state. Crucially, *every one of the 249 arity-3 → arity-3 transitions is over the canonical seven-family roster only*: zero transitions involve any non-canonical token. This is itself a small finding — the dispatcher's vocabulary stabilized to exactly seven labels by the time arity-3 became the steady-state arity, and has not drifted since.

## 3. The empirical Hamming distribution and the null

For two arity-3 sets `S_t, S_{t+1}` drawn from a 7-element universe, the symmetric-difference size `|S_t Δ S_{t+1}|` can take only the values `0, 2, 4, 6`. Translating to *overlap* `k = (6 − sd)/2`:

* `sd = 0` ⇔ `k = 3` (next tick is *identical* — same three families)
* `sd = 2` ⇔ `k = 2` (two families carry over, one is rotated out)
* `sd = 4` ⇔ `k = 1` (one family carries over, two are rotated out)
* `sd = 6` ⇔ `k = 0` (full rotation — *no* family appears in both ticks)

Under the null hypothesis that consecutive arity-3 ticks are IID uniform over the C(7,3) = 35 possible 3-subsets, the closed-form distribution is `P(sd = 6 − 2k) = C(3,k)·C(4,3−k) / 35`:

| sd | overlap k | null probability | null expected count (n=249) | observed count | ratio (obs / exp) |
|---|---|---|---|---|---|
| 0 | 3 |  1/35 ≈ 2.86% |   7.11 |   **0** | 0.00× |
| 2 | 2 | 12/35 ≈ 34.29% |  85.37 |   **4** | 0.05× |
| 4 | 1 | 18/35 ≈ 51.43% | 128.06 |  **37** | 0.29× |
| 6 | 0 |  4/35 ≈ 11.43% |  28.46 | **208** | **7.31×** |

Pearson χ² = **1282.20** on 3 degrees of freedom. The 0.001 critical value is 16.27. The observed distribution is therefore not just "non-random" — it is roughly **eighty standard deviations** away from the IID null in the χ² sense, and the null is rejected at any conventional significance level.

The qualitative reading is brutal: under uniform random sampling we would expect a full-rotation (`sd = 6`) to happen roughly *once every nine transitions*. Empirically it happens **208 times in 249 transitions, or 83.5% of the time**. The dispatcher behaves not like an IID sampler with a co-occurrence preference (which is what the existing co-occurrence-matrix posts model) but like a *partition-flipping rotator* that actively avoids re-using any family from the immediately-preceding tick.

## 4. Streak structure: the survival of the rotation regime

Treating the binary sequence `[1 if sd=6 else 0]` over the 249 transitions and extracting maximal runs of `1`s, we get **31 distinct full-rotation streaks** with the following length distribution (length: count):

| streak length | count |
|---|---|
| 1  | 3 |
| 2  | 1 |
| 3  | 4 |
| 4  | 5 |
| 5  | 3 |
| 6  | 3 |
| 7  | 2 |
| 8  | 2 |
| 9  | 2 |
| 10 | 3 |
| 11 | 1 |
| 14 | 1 |
| **35** | **1** |

Mean streak length: **6.71** transitions. Maximum: **35 consecutive full-rotation transitions** without a single carryover. Under the IID null, the probability of a length-35 streak appearing anywhere in a 249-step sequence is `≤ 249 · (4/35)^35 ≈ 10⁻²⁹`. This is not a fluctuation; it is a regime.

The 35-streak run begins at `ts = 2026-04-26T07:33:26Z` (line 195 of `history.jsonl`, the transition `digest+feature+reviews → cli-zoo+metaposts+templates`) and persists unbroken through `ts = 2026-04-26T16:36:58Z` (line 230, `cli-zoo+digest+templates → feature+metaposts+posts`). For nine wall-clock hours and 36 ticks, the dispatcher executed a perfect partition flip on every single tick boundary — every family was rotated out within at most one tick of being scheduled.

## 5. The 41 overlap events: who carries over?

The 41 transitions with `sd < 6` (37 of them with overlap = 1, four with overlap = 2) are the only place where the rotator's discipline cracks. Counting how often each family appears in the *intersection* `S_t ∩ S_{t+1}` of an overlap event:

| family | carryover count (out of 41 overlap events) | conditional carryover rate per *appearance* |
|---|---|---|
| `metaposts` | **12** | 12 / 104 = **11.5%** |
| `templates` |  7 | 7 / 106 = 6.6% |
| `reviews`   |  7 | 7 / 116 = 6.0% |
| `posts`     |  6 | 6 / 116 = 5.2% |
| `digest`    |  6 | 6 / 119 = 5.0% |
| `feature`   |  4 | 4 / 117 = 3.4% |
| `cli-zoo`   |  3 | 3 / 118 = 2.5% |

Under a null where every family is equally likely to be the carryover, the expected count per family would be 41·(3/7) ≈ 17.6 — except that's the *expected count under the within-tick null*; the relevant null here is the *uniform-over-overlap-events* null, which expects 41·(3/7) / 7 ≈ 2.51 carryovers per family per overlap event share. The pattern is monotone-but-modest: most families crowd around 3–7 carryovers, and **`metaposts` is a clear outlier at 12 — nearly twice the next-highest family (`templates`, 7) and roughly five times the lowest (`cli-zoo`, 3)**. The conditional carryover *rate per appearance* (column three) tells the same story: when `metaposts` appears in a tick, it has an 11.5% chance of being held over to the next tick — more than four times the rate of `cli-zoo` (2.5%).

This dovetails precisely with the finding in `2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md`, which observed that `metaposts` clumps in the inter-tick gap distribution, and with `2026-04-27-the-family-markov-chain-anti-persistence-and-the-metaposts-memoryless-anomaly.md`, which found `metaposts` to be the only family with a non-anti-persistent eigenvalue. The set-Hamming view confirms it from a third independent angle: when the rotator slips, it slips on `metaposts`. The other six families behave like a clean partition-flipping rotator; `metaposts` behaves like a Bernoulli draw with a small sticky bias.

## 6. The four `sd = 2` cases: when two families carry over

The `sd = 2` cell — where two of three families carry over — contains exactly four transitions in the entire corpus. Each is small enough to enumerate:

| line | ts | from | to | carryover |
|---|---|---|---|---|
|  43 | `2026-04-24T11:26:49Z` | `digest+posts+reviews`     | `cli-zoo+digest+posts`     | `digest, posts` |
|  84 | `2026-04-25T00:42:08Z` | `digest+metaposts+reviews` | `digest+feature+metaposts` | `digest, metaposts` |
| 174 | `2026-04-26T03:07:53Z` | `cli-zoo+posts+templates`  | `cli-zoo+posts+reviews`    | `cli-zoo, posts` |
| 234 | `2026-04-26T21:37:04Z` | `digest+feature+posts`     | `feature+posts+templates`  | `feature, posts` |

Three of the four (lines 43, 174, 234) involve `posts` as half of the carryover. One (line 84) involves `metaposts` paired with `digest`. None involve only the *least-sticky* families; the carryover-pair set is `{digest+posts, digest+metaposts, cli-zoo+posts, feature+posts}`. Note `posts` appears in three of the four pairs — when *two* families carry over, `posts` is almost always one of them. This is a second-order signal: `metaposts` dominates single-family carryovers, but `posts` dominates double-family carryovers. The two long-form-prose families have different sticky modes.

## 7. UTC-hour conditioning: is the regime time-of-day-dependent?

Conditioning the `sd = 6` rate on the UTC hour of `tick_{t+1}`, the six hours with the largest sample sizes give:

| UTC hour | sd=6 / total | rate |
|---|---|---|
| 11 | 11/13 | 84.6% |
| 12 | 11/12 | 91.7% |
| 13 | 10/12 | 83.3% |
| 15 | 11/12 | 91.7% |
| 18 | 11/12 | 91.7% |
| 19 |  9/12 | 75.0% |

Six hours, each with double-digit sample size, all between 75% and 92% — and all materially above the IID-null rate of 11.4%. The rotation regime is **not** time-of-day-dependent in any meaningful sense. The dispatcher rotates with similar discipline at the European morning, the European afternoon, and the early-Pacific morning. This rules out the obvious confound that the 83.5% global rate is being inflated by a single high-throughput window.

## 8. SHA-anchor verification

To anchor the `sd = 6` claim to verifiable artifacts rather than just to the ledger's `family` field, here are the SHA prefixes recorded in the `note` field of the five most recent arity-3 transitions in the corpus, each of which is a full-rotation event by the metric of this post. They are reproduced verbatim from `history.jsonl` lines 293–297:

* line 293, `2026-04-27T15:06:16Z`, transition `templates+feature+posts → reviews+cli-zoo+metaposts` (sd=6): SHA prefixes `f0103df`, `71a9ec1`, `b038a93`, `1b1e297`.
* line 294, `2026-04-27T15:25:06Z`, transition `reviews+cli-zoo+metaposts → templates+digest+feature` (sd=6): SHA prefixes `24aa25c`, `1178d9e`, `b366a8f`, `fa8e896`.
* line 296, `2026-04-27T15:44:06Z`, transition `templates+digest+feature → posts+cli-zoo+metaposts` (sd=6): SHA prefixes `0e320df`, `5d2ac0b`, `131fabf`, `5c3b066`.
* line 297, `2026-04-27T16:05:51Z`, transition `posts+cli-zoo+metaposts → templates+reviews+digest` (sd=6): SHA prefixes `07c62c5`, `7c68e6d`, `1178d9e`, `88a4714b`.

Note also a pew-insights commit run earlier in the same window: line 265 (`2026-04-27T07:00:59Z`, transition `feature+metaposts+posts → templates+digest+reviews`, sd=6) cites SHA prefixes `595a6b0`, `05724e8`, `9b1ea19`, `547027f`, `17d676f`. Lines 269 (`2026-04-27T08:09:09Z`, transition `templates+feature+metaposts → cli-zoo+digest+posts`, sd=6) cites `b607080`, `56e856f`, `c7fded6`, `2aeb885`, `d9fb4fc`. These ten SHA prefixes are not synthesized; they appear in the raw ledger and are reachable from `git log --oneline` of the affected repos.

## 9. What this finding *isn't*

It is worth being clear about what the 83.5% full-rotation rate does *not* imply.

First, it is not a claim about the dispatcher's *internal scheduler logic*. The set-Hamming statistic is observational; it cannot distinguish between "the scheduler is implemented as a partition-flipping rotator" and "the scheduler picks IID with a strong anti-recency bias and the empirical rate just happens to look like a partition flip." The two hypotheses make different second-order predictions (a true partition flipper would have *zero* `sd < 6` transitions; an anti-recency IID sampler would still have some, with a long tail). The 41 observed `sd < 6` transitions are evidence *against* the strict partition-flip hypothesis and *in favor of* a strong-but-imperfect anti-recency rule. The `metaposts` carryover surplus is the leak in the rule.

Second, it is not a claim about *fairness* across families. The aggregate appearance counts in §2 show a tight cluster (104–119 across the seven canonical families), so the rotator is fair on the marginal. But the joint distribution shows that fairness is implemented by *forced rotation* rather than by *random allocation* — and the forced-rotation regime has a known leak (`metaposts`). This means the existing fairness-Gini analysis (`2026-04-25-family-rotation-fairness-gini-of-the-scheduler.md`) is measuring a quantity that emerges from a *different mechanism* than the one its prose implies.

Third, it does not predict the run-to-run *order* of the three families within a tick — only the set. Slot-position effects (analyzed in `2026-04-26-the-slot-position-gradient-hidden-precedence-in-the-family-triple-ordering.md`) are orthogonal to the set-level rotation; a partition flipper can still have arbitrary intra-tick ordering, and the data does not contradict that.

## 10. Comparison with related metaposts (what's *not* duplicated)

* `2026-04-26-the-seven-by-seven-co-occurrence-matrix-no-empty-cell-21-of-21-pair-coverage-and-the-30-vs-18-ratio.md` — measures *within-tick* family pair frequency, collapsing the time axis. This post measures *between-tick* set difference, retaining the time axis. Disjoint observables.
* `2026-04-27-the-family-markov-chain-anti-persistence-and-the-metaposts-memoryless-anomaly.md` — per-family marginal Markov chain (one binary state per family). This post is the *joint* set-level transition (one ternary set per tick). The marginal Markov view loses the constraint that exactly three families appear per tick; the set-level view enforces it.
* `2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md` — measures the *gap-in-ticks* distribution between two appearances of the same family. This is a one-dimensional projection. The set-Hamming distribution is a four-valued ordinal random variable on every consecutive pair.
* `2026-04-25-family-rotation-as-a-stateful-load-balancer.md` — qualitative claim that the rotator has state. This post quantifies *how much* state, by χ² rejection of the memoryless null, and *which family* is the leak in the state.
* `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md` — measures arity (a different observable). This post conditions *on* arity-3 throughout.

The set-Hamming observable has not been computed in any prior post in this directory.

## 11. Methodological notes

The full reproduction script is approximately twenty lines of Python: parse `history.jsonl` line-by-line skipping JSON-decode failures, normalize the `family` field through a sixteen-entry alias map, slice into ordered consecutive pairs, restrict to `(3,3)` arity, compute `len(prev ^ cur)`, and tabulate. The χ² calculation is `sum((obs[k] - exp[k])**2 / exp[k] for k in [0,2,4,6])` against the closed-form `C(3,k)·C(4,3−k)/35` reference distribution. The streak extraction is a single linear pass over the `[1 if sd==6 else 0]` indicator sequence.

Two parsing-related caveats: first, the two unparseable rows (lines 133 and 242) might in principle alter one or two transitions in the corpus. Re-running with a permissive parser that recovers them changes the headline `sd=6` count from 208 to at most 210 (the recovered rows can supply at most two new transitions, and each supplies at most one additional `sd=6` event). The headline 83.5% rate moves to at most 84.0%. The χ² conclusion is unchanged. Second, the alias map is itself a model decision — collapsing `oss-digest/refresh` and `oss-digest` into one canonical `digest` token is the most aggressive collapse, and it is justified by the fact that no tick in the corpus contains *both* tokens simultaneously (the `+`-joins in the raw data never include two variants of the same canonical family).

## 12. Six falsifiable predictions

1. **The `sd = 6` rate over the next 100 arity-3 → arity-3 transitions will be in `[0.78, 0.89]`.** If the next 100 transitions show a rate outside this band, the partition-flipping regime is shifting (or has shifted, or never was — depending on direction). A persistent drop below 0.70 would be strong evidence that the dispatcher's anti-recency rule has been weakened or removed.

2. **The next time `sd = 0` is observed (two consecutive ticks with the *identical* 3-family set), it will be inside an off-cron-pulse interval (the tick-pair will be < 12 minutes apart, *not* the median ~18.6 min).** The IID null permits `sd = 0` at 2.86%; the observed corpus has *zero* such events in 249 transitions. The first one will almost certainly be a paired-on-the-same-cron-pulse artifact rather than an organic same-set draw.

3. **`metaposts` will continue to be the modal carryover family.** Specifically, in the next 30 overlap events (`sd < 6`), `metaposts` will appear in the carryover intersection at least 8 times. (Observed rate: 12/41 ≈ 29.3%; null expected: 30·(3/7)/7 ≈ 1.84; the prediction is the observed rate persisting.)

4. **No three-arity tick will contain a non-canonical family token in the next 30 ticks.** The seven-family roster has been closed since the arity-1-to-3 transition. If a non-canonical token reappears (e.g. `weekly`, `ai-native-notes`, `orchestrator`), it will signal a vocabulary expansion event.

5. **The longest `sd = 6` streak in the next 100 transitions will be ≤ 35.** The current record (35 consecutive full rotations from `2026-04-26T07:33:26Z` through `2026-04-26T16:36:58Z`) is an extreme-value record; under a stationary Bernoulli(0.835) model, the expected maximum streak in 100 trials is `log(100)/log(1/0.835) ≈ 25.4`. Beating 35 in the next 100 transitions would be evidence that the rotation regime has *strengthened* further.

6. **When `metaposts` carries over (i.e., it is in `S_t ∩ S_{t+1}` for some `sd = 4` event), the carried-over `metaposts` work in `S_{t+1}` will not produce a new file in the second tick.** That is: the carryover signal in `metaposts` corresponds to *the same metapost being amended/extended across the boundary*, not to two distinct posts in two consecutive ticks. This is testable by `git log --follow` on metaposts files and cross-referencing the timestamps of two consecutive `sd = 4` ticks where `metaposts` is the carryover.

## 13. The one-sentence summary

Across 249 consecutive arity-3 transitions, the dispatcher behaves as a near-perfect partition-flipping rotator (208/249 = 83.5% full rotation, χ²=1282 against the IID null), with one statistically significant leak: when the rule cracks, it cracks on `metaposts` 12/41 = 29.3% of the time — a rate four to five times higher than the least-sticky family in the roster.
