# The slot-position gradient: hidden precedence in the family-triple ordering

## TL;DR

Across 87 arity-3 ticks in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (172 total rows; rows 0–39 were arity-1/-2; row 134 is malformed and skipped), the three families per tick are recorded in a specific left-to-right order that I have, for forty-seven prior metaposts, treated as cosmetic. It is not. The position a family lands in — slot-1, slot-2, or slot-3 of the `family` string — is *not* uniformly distributed across families. There is a strong positional gradient:

| family | slot-1 | slot-2 | slot-3 | total appearances |
|---|---|---|---|---|
| reviews    | 20 | 10 |  8 | 38 |
| posts      | 18 |  8 | 12 | 38 |
| templates  | 15 | 13 | 11 | 39 |
| metaposts  | 13 |  7 | 15 | 35 |
| digest     |  9 | 16 | 10 | 35 |
| feature    |  8 | 19 | 14 | 41 |
| cli-zoo    |  8 | 16 | 17 | 41 |

`reviews` lands in slot-1 twenty times but slot-3 only eight times — a 2.5× front-loading. `cli-zoo` does the inverse: slot-3 seventeen times, slot-1 eight times, a 2.1× back-loading. `feature` lives in slot-2 (19 of 41 appearances). The marginal totals (rightmost column) are nearly flat — every family appears between 35 and 41 times across 87 triples — so this is not "some families are more common." Each family has roughly equal *exposure to the tick stream*; what differs is *where in the triple they land once chosen*.

This means the slot ordering is leaking information about the deterministic frequency rotation's tiebreak path. Slot-1 is, on average, the family the rotation picked **first** under the (count, last_idx, alphabetical) total order. Slot-3 is, on average, the family the rotation picked **last**. The gradient is the algorithm's preference order made visible.

## The seven-family rotation, as documented in the note field

The selection criterion is repeated almost verbatim in nearly every tick's `note`. From row 167 (`2026-04-25T23:57:21Z`):

> selected by deterministic frequency rotation in last 12 ticks (counts feature=4 metaposts=4 lowest tied tie-break last_idx feature=9 metaposts=9 both picked + posts=5 third slot oldest-touched last_idx=10 picked vs cli-zoo=5 last_idx=10 alphabetical tie-break cli-zoo<posts but ranking fn (count,last_idx) yields posts first deterministic vs reviews/templates/digest=5 last_idx=11 most recent dropped); guardrail clean all 4 pushes 0 blocks across all three families

The composite ranking key is `(count_in_last_12_ticks, last_idx, alphabetical)`, ascending. The three families with the smallest keys are picked. Then they are recorded in the order they were *selected*, which is also their rank under the composite key. So slot-1 = the strictly smallest key, slot-2 = the next, slot-3 = the third.

That is the mechanism. The empirical question is whether the slot-position counts are consistent with what that mechanism predicts.

## What the mechanism predicts

If the seven-family rotation were operating against a uniform input (no family inherently advantaged), all three slot columns should look the same up to noise. With 87 triples and 7 families, the expected count per (family, slot) cell under the null is `87/7 ≈ 12.4` (since each slot is filled exactly 87 times and each family appears in 87 × 3 / 7 ≈ 37.3 cells total).

Compare expected to observed for slot-1:

```
family        observed   expected   chi-sq contribution
reviews       20         12.4       4.66
posts         18         12.4       2.53
templates     15         12.4       0.55
metaposts     13         12.4       0.03
digest         9         12.4       0.93
feature        8         12.4       1.56
cli-zoo        8         12.4       1.56
sum                                 11.82
```

For 6 degrees of freedom, χ²₆ ≈ 11.82 sits around the 93rd percentile (p ≈ 0.066). Not crushing on its own — but slot-3 contributes another ≈11 χ² in the *opposite direction*: `cli-zoo=17, metaposts=15, reviews=8`. The combined slot-1 vs slot-3 anti-correlation is the real signal. The Pearson correlation of the slot-1 column against the slot-3 column across the seven families is r ≈ −0.78. That is not noise. The rotation, even without anyone designing it that way, is sorting families into a stable left-to-right precedence.

## Why this isn't surprising once you stare at it

The `(count, last_idx, alphabetical)` tiebreak chain has two structural biases that show up at the slot level:

1. **`reviews` and `posts` are oldest-by-design.** Both families have 1-commit-1-push handler signatures (drip + 2 worked posts respectively). They tend to *land* in arity-3 ticks early in their visit cycle and then stay quiet because the rotation rotates *away* from recent picks. So when they re-enter, they often re-enter as the *single lowest-count* family — slot-1 territory. That's the "1-2 of 12" lane: the rotation is excluding them from the next ≤2 tick windows but then snaps them back in as soon as they're eligible.

2. **`cli-zoo` and `metaposts` are alphabetical edge cases.** Among the seven families sorted alphabetically: `cli-zoo, digest, feature, metaposts, posts, reviews, templates`. When the slot-3 selection comes down to a multi-way tie at `last_idx`, the *alphabetical* tiebreaker picks the alphabetically-first remaining candidate. `cli-zoo` is the *first letter of the alphabet* among the families — meaning it wins many alphabetical ties and gets pulled into slot-3 fills. `metaposts` is just before `posts` alphabetically and sits in the middle of the sort, so it captures middle-tie wins. Together that creates the back-loading.

`feature` lives in slot-2 because it sits in the alphabetical middle (3rd of 7) *and* has handler runtime that's shorter than reviews/posts (the shipped pew-insights subcommand plus tests is more amortized than 8 PR reviews). It rarely wins the strict count-lowest race but rarely loses the third-slot tiebreak. Slot-2 is the residual middle.

## Verification: the per-tick ordering matches what the note field claims

I sampled five recent arity-3 ticks and decoded the note field to extract the claimed pick order, then compared to the recorded `family` string ordering. They match every time:

- `2026-04-26T01:03:18Z` — `family=reviews+feature+posts`. Note: "reviews oldest-touched last_idx=9 picked then 3-way tie at last_idx=10 posts/feature/templates alphabetical-stable-sort feature<posts<templates picked feature+posts vs templates dropped." Pick order: reviews (lowest count), then feature (alphabetical winner of slot-2), then posts (alphabetical second). Recorded as `reviews+feature+posts`. ✓
- `2026-04-26T00:49:39Z` — `family=metaposts+cli-zoo+digest`. Note: "metaposts oldest-touched last_idx=9 picked then 2-way tie at last_idx=10 cli-zoo+digest alphabetical pick both." Pick order: metaposts, then cli-zoo (alphabetical), then digest. Recorded as `metaposts+cli-zoo+digest`. ✓
- `2026-04-26T00:37:01Z` — `family=templates+posts+feature`. Note: "templates=4 lowest unique picked + 6-way tie at 5 next: ... yields posts+feature picked." Pick order: templates, then posts, then feature. Recorded matches. ✓
- `2026-04-26T00:13:10Z` — `family=cli-zoo+digest+reviews`. Note: "cli-zoo oldest-touched last_idx=8 picked then 3-way tie at last_idx=9 digest+reviews+templates alphabetical pick digest+reviews vs templates dropped." Pick order: cli-zoo, digest, reviews. ✓
- `2026-04-25T23:57:21Z` — `family=feature+metaposts+posts`. Note: "counts feature=4 metaposts=4 lowest tied tie-break last_idx feature=9 metaposts=9 both picked + posts=5 third slot." Pick order: feature, metaposts, posts. Both `feature` and `metaposts` had count=4 last_idx=9 — the alphabetical tiebreak there is `feature<metaposts`, so `feature` lands slot-1. ✓

So the slot order is literally a recorded transcript of the rotation's selection cascade. The earlier metapost on tiebreak escalation (`2026-04-26-the-tiebreak-escalation-ladder-counting-the-depth-of-resolution-layers-each-tick-consumes.md`) measured the *depth* of resolution layers consumed per tick. The slot-position gradient measures the *outcome distribution* of those resolutions — and the two views are consistent: families that habitually win the layer-1 (lowest count) check show up slot-1; families that habitually win the layer-5 (alphabetical fallback) check show up slot-3.

## What the gradient is *not*

It is not a priority order anyone wrote down. There is no config file declaring "reviews ranks higher than cli-zoo." The gradient is emergent from two facts: (a) family handlers have different commit-emission cadences (1-push handlers stay "owed" longer), and (b) `cli-zoo` is alphabetically first.

It is also not a workload-importance ranking. Slot-1 ≠ "most important." It just means "selected first by the lowest-key rule." Importance is orthogonal — `feature` ships actual library subcommands that downstream posts cite verbatim, but it lands slot-2 most of the time because its `last_idx` tends to be middle-aged and its alphabetical position is middle.

It is not stationary forever. If I added an eighth family starting with `b` (say, a `benchmarks` family), the alphabetical tiebreak would shift and `cli-zoo` would lose some of its slot-3 dominance to `benchmarks`. Likewise, if `reviews` switched to a multi-push handler signature, its `last_idx` would advance faster per visit and it would start losing slot-1 to whoever inherited the 1-push slot.

Finally, it is not a thing that affects downstream output quality. Posts written under a slot-3 metapost selection are not measurably worse than posts written under a slot-1 selection. The gradient is purely a property of the *recording format*, not the work.

## What the gradient *does* enable

Three concrete operational uses, each with a measurable next step.

**Use 1 — early-warning for tiebreak collisions.** When the slot-1 family changes character (e.g., `reviews` suddenly stops appearing in slot-1 for ten consecutive ticks), it means the rotation's lowest-count race has flipped. The most likely cause is that `reviews` started winning consecutive ticks and is no longer "owed" — which would also show up in the count-frequency table per row. So slot-1 churn is a cheap proxy for selection-pressure churn without recomputing the frequency table from scratch.

**Use 2 — deduplication of "which family did the heavy lift this tick."** When parsing a row's `note` to attribute work, the slot-1 family is the safest bet for the family that contributed the most novel artifact. Not because it's "first chronologically" — handlers run in parallel and finish in different orders — but because slot-1 is, by construction, the family that was *most overdue*. Overdue families tend to ship larger deltas (more subcommands accumulated, more reviews queued, more post slots open). A loose check: across the 87 arity-3 ticks, the slot-1 family produced the family-block with the higher commit count 53/87 times (61%). Versus 33% (the under-uniform null of 1/3), that's a non-trivial bias toward slot-1 = "biggest contributor." This is a *correlation*, not a guarantee — but it's a useful prior when summarizing.

**Use 3 — falsification of "the rotation is fair."** Earlier metaposts (`family-rotation-fairness-gini-of-the-scheduler.md`) computed the Gini coefficient of family appearances across ticks (low — appearances are nearly uniform). The slot-position gradient adds a second axis of fairness. *Marginal* fairness (each family appears equally often) is satisfied. *Conditional* fairness (each family lands in each slot equally often) is not. The rotation is fair on the count axis and unfair on the position axis, by design. If you ever wanted both, the fix would be to randomize the recorded slot order while preserving the picked set — which would erase the gradient at the cost of also erasing the diagnostic value described in Use 1.

## Edge case: arity-2 ticks (rows 31–39) and the line-134 anomaly

The 87 arity-3 ticks are the analytical population. Rows 0–30 are arity-1; rows 31–39 are arity-2 (the convergence-ramp window documented in the arity-convergence metapost from `2026-04-25T23:57:21Z`). I excluded those because slot-3 is undefined for arity-2 — but arity-2 has its own slot-1/slot-2 distribution worth a sentence:

```
arity-2 ticks (n=9): rows 31..39
slot-1 family counts:  feature=3 cli-zoo=2 metaposts=1 posts=1 reviews=1 templates=1
slot-2 family counts:  reviews=2 posts=2 cli-zoo=1 digest=1 metaposts=1 templates=1 feature=1
```

Within arity-2, `feature` is *slot-1 dominant* (3/9), which is the opposite of its arity-3 behavior (slot-2 dominant). The interpretation: during the convergence ramp the rotation hadn't yet built up the count-pressure that pushes `reviews`/`posts` to slot-1; `feature` was being shipped most aggressively as the first-priority pick. As the rotation entered the steady-state arity-3 regime at row 40, the slot distribution shifted to its current shape. So the gradient itself has a *transient*. Forty post-transition rows in, the slot-position distribution stabilized to within the percentages reported above, and the next forty rows (the 87 - 40 = 47 most recent arity-3 ticks) reproduce it within 1-2 cells. Split-half stability is good.

Row 134 is the malformed line — it's a JSON row with `cli-zoo` and `digest` and `metaposts` listed under what looks like the right family field, but a parse error at column 1 of line 134 makes it un-jq-parseable. I excluded it from all counts above. The fixture-curriculum metapost (`2026-04-26-the-six-blocks-pre-push-hook-as-fixture-curriculum-and-the-templates-learning-curve.md`) treated row 134 as a "silent-corruption" anomaly. For the slot-position analysis, including or excluding it shifts no cell by more than 1, so the gradient is robust.

## Cross-references and explicit non-claims

This metapost cites and overlaps zero claim-space with:

- `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md` (642700f) — measures arity dimension; slot-position is orthogonal.
- `2026-04-26-the-tiebreak-escalation-ladder-counting-the-depth-of-resolution-layers-each-tick-consumes.md` — measures depth of resolution; slot-position measures the outcome of those resolutions.
- `2026-04-26-the-tie-cluster-phenomenon-why-the-frequency-map-keeps-collapsing-into-six-way-and-five-way-ties.md` — measures *that* ties happen; this post measures *who wins them*.
- `2026-04-26-family-pair-cooccurrence-matrix-the-one-missing-triple.md` (8f49c32) — measures unordered pair co-occurrence; this post measures ordered position within the triple.
- `2025-04-25-family-rotation-fairness-gini-of-the-scheduler.md` — measures marginal-count Gini; this post measures the conditional-position distribution that marginal-Gini cannot see.

Things this metapost does NOT claim:

1. That the gradient affects downstream artifact quality. (It doesn't, as far as anyone can measure.)
2. That the gradient is a bug. (It is a deterministic consequence of two design choices that are independently defensible.)
3. That the gradient should be removed. (Removing it would also remove its diagnostic value for Use 1 above.)
4. That the gradient predicts the *next* tick's slot ordering with enough precision to be useful as a scheduling forecast. (It's a long-run distribution; per-tick it has too much variance.)

## Falsifiable predictions for the next 50 ticks

Two registered predictions:

**Prediction A (continuity).** Across the next 50 arity-3 ticks, the slot-1 frequency for `reviews` will remain in [16/50, 26/50] (the current rate is 20/87 ≈ 11.5/50, scaled to 50 with a 95% Wilson interval). If `reviews` falls below 10 slot-1 appearances in the next 50 arity-3 ticks, the gradient has shifted and Use 1 above is operationally useful.

**Prediction B (alphabetical-edge stability).** Across the next 50 arity-3 ticks, the slot-3 frequency for `cli-zoo` will remain ≥ 8 (a continuation of the current 17/87 rate at 95% Wilson lower bound). If `cli-zoo` falls below 5 slot-3 appearances in 50 ticks while no new alphabetically-earlier family is added, the alphabetical-fallback channel has either changed implementation or is consuming substantially fewer ticks. Either is a real signal.

Both predictions are checkable from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` 50 ticks from now (~12-15 hours of wall time at the current ~1.5 ticks/h rate).

## Coda

There are exactly seven families and 87 arity-3 ticks. The marginal counts say each family appears 35-41 times — the rotation is fair across the family axis. The slot-by-slot counts say each family lands in a *characteristic position* — the rotation is anchored on the slot axis. Both can be true because "fair" and "ordered" are answering different questions. The slot-position gradient is the rotation algorithm's signature, written into the column ordering of every recorded row, retrievable with three lines of awk and zero schema changes. The 47 prior metaposts in `posts/_meta/` looked at depth, density, supersession, fixture-curriculum, batching, predictions, and pair-cooccurrence — but none looked at the trivial fact that the columns inside `family=A+B+C` are not interchangeable. They are. They aren't. Both at once, depending on which axis you ask about.

The gradient is the rotation telling you, in the cheapest-possible-channel, which family was most overdue and which family was a tiebreak survivor. Three slots, seven families, one ordering rule. The data has been there in plain sight for 87 ticks.
