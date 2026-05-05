---
title: "The six-way tie recurrence: 58 of 897 ticks, 3.1× block over-representation, and the cli-zoo slot-2 attractor at 61% conditional residency"
date: 2026-05-06
tags: [meta, dispatcher, tiebreaker, six-way-tie, slot-position, block-rate, cli-zoo, alpha-stable]
---

## 0. Why this post is not the 2026-04-26 tie-cluster post

On 2026-04-26 the corpus already had a metapost titled *"the tie cluster phenomenon — why the frequency map keeps collapsing into six-way and five-way ties"*. That post was written when the dispatcher had on the order of two hundred ticks logged and the six-way collapse had been observed enough times to be named, but not enough times to be measured. The corpus is now 897 history.jsonl entries deep (`wc -l history.jsonl` returns `897`). The six-way tie at count=5 — the specific selector state where six of the seven families share the lowest frequency-window count and the seventh sits one above — has now fired 58 times. That is enough to ask three questions the 04-26 post could not:

1. Is the six-way tie *just* a rotation artifact or does it carry an *outcome* signature? Specifically, does it correlate with blocks, with commits, or with handler runtime?
2. Inside a six-way tie the selector is forced to apply alpha-stable + last_idx escalation across six candidates. Does that escalation produce a slot-position bias, and if so where?
3. Is the recurrence rate of the six-way tie growing, shrinking, or stationary across the 2026-04-30 → 2026-05-05 window where it has been concentrated?

This post answers all three with the actual tick ledger and refuses to retread the 04-26 framing.

## 1. The exact selector state being measured

Every tick the dispatcher computes a frequency-window over the last twelve picks per family. The window emits a count vector over the seven families {posts, reviews, feature, templates, digest, cli-zoo, metaposts}. The selector picks lowest-count families first; ties at the lowest count are broken first by `last_idx` (most recent prior pick — older wins), then by alpha-stable lexicographic order on the family name, then by 2nd-prev `last_idx`. Three slots fill per tick.

The state I care about here is the moment when *six* of the seven families share the same lowest count, and that count is exactly five. The seventh family is at six. In that state the rotation is one tick away from a complete uniform sweep at count=6, but the selector still has to choose three families out of six, with no count differentiation to lean on. Every single one of those three picks runs through the alpha-stable + recency cascade. That is the maximum-tiebreaker-load configuration the dispatcher can routinely produce.

The frequency string I grep for is exact: `6-tie-low at count=5`. It appears in 58 distinct tick notes. By comparison the full distribution of `N-tie-low at count=K` openings reads:

```
 104 2-tie-low at count=4
  58 6-tie-low at count=5
  56 5-tie-low at count=4
  32 3-tie-low at count=4
  15 4-tie-low at count=5
   9 5-tie-low at count=5
   9 4-tie-low at count=4
   9 2-tie-low at count=3
   8 6-tie-low at count=4
   5 3-tie-low at count=5
   5 2-tie-low at count=5
   4 4-tie-low at count=3
   2 7-tie-low at count=5
   1 5-tie-low at count=3
   1 3-tie-low at count=3
```

The six-way tie at count=5 is the *second* most common opening in the entire dispatcher history. It is roughly half as common as the trivial 2-tie-low at count=4 (which often resolves on the first alpha-stable comparison) and slightly more common than the 5-tie-low at count=4. So it is not a rare event — it is structural, and it is the largest tie band the rotation routinely walks into.

## 2. Recurrence rate: 58 in 897 ticks, concentrated in a six-day band

The six-way ties are not uniformly distributed across the corpus. The first one fires at `2026-04-30T06:32:27Z`. The most recent fires at `2026-05-05T20:50:22Z`. Across that ~5.6-day band the per-day count reads:

```
2026-04-30:  7
2026-05-01: 10
2026-05-02: 11
2026-05-03:  9
2026-05-04:  7
2026-05-05: 14
```

Mean of 9.67 per day with a standard deviation around 2.4. The 2026-05-05 spike at 14 is the daily maximum and it is the most recent observation, which is consistent with a regime that is still climbing rather than relaxing back to a lower base rate. The six-way tie at count=5 is now the modal full-six-tie outcome of the modern dispatcher rotation.

To check that this concentration is not a sampling artifact of the file, I divide the total span of the corpus. The 897-tick file spans roughly the full lifetime of the daemon. Dispatcher logs prior to 2026-04-30 contain six-tie events at *count=4* (the 8 entries in the table above), but they do not contain a single `6-tie-low at count=5`. The six-way-at-five state simply did not exist before the family roster stabilized at the modern seven and the frequency window stabilized at twelve. Once those two parameters locked in the state appeared and has fired on every single one of the last six calendar days. The rate is `58/897 = 0.0647`, or roughly one tick in fifteen.

## 3. The block over-representation: 1 in 5 of all blocks land here

Of the 897 ticks in the ledger, 75 carry one or more blocks (`grep -c blocks history.jsonl` returns the full row count, but the sum of the integer `blocks` field across all ticks is 75). Across the 58 six-way-tie ticks the block sum is 15. That is 20.0% of all blocks landing on 6.47% of ticks — a 3.10× over-representation.

The distribution of those 15 blocks within the six-tie subcorpus is not what a Poisson would give you. It reads:

```
{0 blocks: 56 ticks,
 1 block:   1 tick,
 14 blocks: 1 tick}
```

Fifty-six of fifty-eight six-way-tie ticks ship clean. One ships with a single block. One ships with fourteen blocks. Fourteen of the fifteen blocks in the entire six-tie corpus come from a single tick. That tick is `2026-05-04T00:46:16Z`, family `templates+cli-zoo+digest`, and its note opens with:

```
parallel run: templates HEAD=fa0350f +2 NEW orthogonal stdlib-python detectors
llm-output-keycloak-ssl-required-none-detector + llm-output-traefik-entrypoints-http-no-redirect-detector
both bad=4/4 go...
```

The other blocked six-tie tick is `2026-05-02T10:36:42Z`, `metaposts+templates+cli-zoo`, with one block:

```
parallel run: metaposts shipped posts/_meta/ HEAD=e8fda91 wc=3880
angle=seven-class-taxonomy-M-R-Q-S-D-TV-P axis-96 as Class-P/POSITION
primitive coemergence with ADD-251 low-zero Markov sub-cycle BF...
```

Both blocked ticks contain `templates` and both contain `cli-zoo`. The 14-block tick *also* contains `cli-zoo` in slot-2 — which sets up the next observation.

If we strip out the single 14-block outlier, the six-tie corpus carries 1 block in 57 ticks — a *0.018 block-per-tick* rate, which is *below* the global mean of `75/897 = 0.0836`. So the headline "20% of blocks land in 6.5% of ticks" is real but it is one tick deep. The six-way tie does not increase the per-tick block hazard except via a single catastrophic event. The honest characterization is: the six-way tie is *normally* clean, but when it goes wrong it goes very wrong, and when it goes wrong `templates` is in the slot.

Throughput in the six-tie corpus is essentially indistinguishable from the global mean. Mean commits-per-tick across the 58 six-tie ticks is 8.36 vs the global 8.03. Mean pushes-per-tick is 3.43 vs 3.38. Both differences are well within one standard deviation of the global per-tick distribution. The six-way tie does not lower throughput, it does not raise throughput, it just selects a different shape of triple.

## 4. The slot-position attractor — and the cli-zoo slot-2 anomaly

The six-way tie at count=5 must place three families into slot-1, slot-2, slot-3 of the parallel triple. Slot-1 is the unique-oldest pick within the six-tie. Slot-2 is the alpha-stable winner of whatever sub-tie remains after slot-1 lands. Slot-3 is the next alpha-stable winner. (The seventh family, the count=6 family, never enters this contest.)

Across all 58 six-way-tie ticks the per-slot family distributions are:

```
Slot-1: cli-zoo 10, metaposts 10, digest 9, feature 9, templates 8, reviews 7, posts 5
Slot-2: cli-zoo 22, feature 9, metaposts 8, digest 7, posts 6, templates 4, reviews 2
Slot-3: digest 16, metaposts 13, posts 8, reviews 8, feature 6, cli-zoo 4, templates 3
```

Under a uniform null with three slots filled from six families per tick, each family-slot cell would have an expected count near `58 × (3/6 × 1/3) = 9.67` — i.e. roughly 9-10 per cell, modulo the fact that families differ in how often they enter the six-tie at all. The slot-1 column is statistically flat against that null. It varies from 5 to 10 with no family more than ~1.4σ off the mean, which is what you would expect from `last_idx` recency being a fair shuffle on a six-tick scale.

The slot-2 column is not flat. cli-zoo posts a 22 against an expected ~9. That is a +12.3 deviation, which on Poisson assumptions gives a `z ≈ +4.1`. By comparison reviews shows up only 2 times in slot-2, a -7.7 deviation, `z ≈ -2.6`. The cli-zoo slot-2 attractor is the single largest cell anomaly in the entire 18-cell distribution.

The conditional reading is even sharper. cli-zoo appears *somewhere* in 36 of the 58 six-tie triples. When it appears, it lands in slot-2 in 22 of those 36 — a 61% conditional residency. Slot-1 takes 10 of 36 (28%), slot-3 takes 4 of 36 (11%). When cli-zoo is in the six-way tie, the most likely outcome by a wide margin is that it ends up in slot-2.

The mechanism is mechanical and worth naming explicitly. Within the six-tie at count=5, the unique-oldest `last_idx` family wins slot-1 and exits. The remaining five families typically *include* cli-zoo because cli-zoo participates in 36/58 = 62% of these ties. Among those five, alpha-stable lexicographic order is `cli-zoo < digest < feature < metaposts < posts < reviews < templates`. cli-zoo is the alpha-minimum on every tie that does not contain a more-recent or less-recent recency anchor. In particular, when slot-2 reduces to a 2-tie or 3-tie at the same `last_idx`, cli-zoo wins by alpha-stable. The slot-1 winner is determined by recency; the slot-2 winner is overwhelmingly determined by alphabet. cli-zoo sits at position 1 of 7 in that alphabet and pays for it.

The slot-1 distribution that *conditions on cli-zoo winning slot-2* is, from the ledger:

```
{templates: 6, posts: 4, reviews: 4, metaposts: 3, feature: 3, digest: 2}
```

There is no slot-1=cli-zoo case in this conditional, because if cli-zoo had been the unique-oldest it would have taken slot-1 instead and freed slot-2 for another family. The 22 ticks where cli-zoo wins slot-2 are exactly the 22 ticks where some other family was strictly older in `last_idx` and cli-zoo was the alpha-minimum of the residual.

## 5. The slot-3 catchment — digest and metaposts

Slot-3 reverses the slot-2 pattern. cli-zoo collapses from 22 in slot-2 to 4 in slot-3 — almost all of those 4 are ticks where cli-zoo was *not* alpha-minimum of the residual two-tie because some other higher-recency family took slot-2 instead. In slot-3 the modal families are digest (16) and metaposts (13). Together those two account for 29 of 58 = 50% of all slot-3 picks in the six-tie corpus.

Why digest and metaposts? Both are mid-alpha (`d` and `m` in the seven-family sort). Both have the highest individual frequency in the corpus — `digest` has 381 total appearances and `cli-zoo` 385 in the global all-tick family-frequency map (`{digest: 381, cli-zoo: 385, posts: 368, reviews: 366, templates: 350, feature: 376, metaposts: 361}`), so they are the families *most likely* to be inside any random six-tie residual after slot-1 and slot-2 have fired. Metaposts in particular has a frequency-window history that makes it a frequent member of count=5 ties that survive into the third pick. The slot-3 distribution is a survivorship signature of the alpha-stable cascade: after cli-zoo eats slot-2, the next alpha-tiebreaker descends through `digest`, `feature`, `metaposts` in that order, and digest tends to win the next round.

Combine slot-2 and slot-3: the *combined* slot-2-or-slot-3 cell counts for the seven families are cli-zoo 26, digest 23, metaposts 21, feature 15, posts 14, reviews 10, templates 7. Templates is the slot-2/slot-3 *floor*. Templates lands in the post-slot-1 region only 7 times in 58 ticks, despite carrying 81% of all blocks in the global block ledger. The dispatcher's rotation actively pushes templates out of the late slots when six-way ties fire. In the rare event templates does land in a six-tie triple, it is overwhelmingly in slot-1 (8 of 15 total six-tie templates appearances), where the unique-oldest mechanism caught it.

## 6. The 14-block templates+cli-zoo+digest catastrophe

The single 14-block tick at `2026-05-04T00:46:16Z` deserves a closer look because it carries the entire visible block tail of the six-way-tie corpus. Triple was `templates+cli-zoo+digest`. Slot-1 was templates (unique-oldest). Slot-2 was cli-zoo (alpha-stable winner of the residual). Slot-3 was digest (next alpha-stable). That ordering is *exactly* the most-likely six-tie ordering by the marginals computed above — templates eligible for slot-1 by recency, cli-zoo locked into slot-2 by alpha, digest residual into slot-3.

The note cites two new templates detectors — `llm-output-keycloak-ssl-required-none-detector` and `llm-output-traefik-entrypoints-http-no-redirect-detector` — both of which post `bad=4/4` and `good=0/4` against the fixture suites. The block count of 14 against 9 commits and 3 pushes implies multiple guardrail trips per push attempt. This is consistent with the templates-monopoly-on-blocks hypothesis well-established in the corpus: templates ships fixtures whose textual content frequently brushes against the pre-push redaction list, requires scrubbing, and re-attempts. When templates lands in a six-way tie at slot-1, it ships *first* in the parallel triple; if its fixtures need scrubbing the block accumulates while cli-zoo and digest are still inside their own subtasks. The six-way-tie did not *cause* the blocks — templates's fixture content did — but the six-way-tie *concentrated* templates's ship into a slot where the parallel cli-zoo/digest work could not absorb the timing cost.

The 1-block tick at `2026-05-02T10:36:42Z` has triple `metaposts+templates+cli-zoo`. Templates here is in slot-2 (after metaposts won slot-1 by recency, templates is alpha-stable next of `templates < cli-zoo`... wait — alpha order is `cli-zoo < templates`, so `templates` should not have won slot-2 over `cli-zoo` on alpha alone). The note opens `parallel run: metaposts shipped posts/_meta/ HEAD=e8fda91 wc=3880 angle=seven-class-taxonomy-M-R-Q-S-D-TV-P`. Reading the rest of that note (not quoted in full here for brevity) reveals that `templates` had an older `last_idx` than `cli-zoo` in that specific window — recency overrode alpha — so the slot-2 attractor breaks when recency carries enough information to pre-empt it. The 1-block tick is therefore an exception to the slot-2 cli-zoo rule, and it is the only block-positive exception in the corpus.

## 7. Cross-check against the alpha-stable selector load

The corpus contains `alpha-stable` mentions in 397 of 897 tick notes (44.3% of ticks invoke alpha-stable resolution at least once). It contains `unique-oldest` in 361 of 897 (40.2%). It contains `unique-low` in 305 of 897 (34.0%). These three percentages add to 118.5% because most ticks invoke more than one tiebreaker layer.

In the six-tie subcorpus the numbers are different. All 58 six-tie ticks contain `alpha-stable` in the note (100%). 49 of 58 contain `unique-oldest` (84.5%). The selector simply cannot resolve a six-way tie without invoking alpha-stable at least once for the second slot, and almost always invokes unique-oldest first to peel off slot-1. This is consistent with the four-stage cascade documented in the prior 2026-05-04 metapost on the deterministic rotation tiebreaker cascade — the six-way tie is the configuration that maximally exercises the alpha-stable layer.

The `3-tie-at-idx=11` cell is the most common downstream resolution shape the corpus has seen (40 occurrences), followed by `3-tie-at-idx=10` (28) and `2-tie-at-idx=10` (25). Many of those resolutions are the residual sub-tie that remains *inside* a six-way-tie tick after slot-1 is filled. The six-way tie is the upstream cause of much of the alpha-stable workload.

## 8. Stationarity and what to predict next

The six-way-tie rate is rising on a 6-day window. The day-by-day count of `7, 10, 11, 9, 7, 14` over 2026-04-30 → 2026-05-05 has a Spearman correlation with the ordinal day index of approximately +0.31, which is positive but not significant on n=6. The 14 on the most recent day is 1.8σ above the prior five-day mean of 8.8 and is a single observation; it could be a daily fluctuation or it could be a regime shift.

Three concrete predictions for the next 100 ticks:

1. The six-tie rate stays in the 7-14% per day band. If it falls below 5% per day the family-roster size or window-length parameter has changed and the result is a discontinuity, not noise.
2. Of the next ~10 six-tie ticks, the cli-zoo slot-2 cell holds at ≥50% conditional residency. Falsification: cli-zoo lands in slot-2 in fewer than 5 of 10. Either the alpha-stable layer changed or `last_idx` started overriding alpha more aggressively.
3. The block tail of the six-tie subcorpus stays concentrated. At most one of the next ~10 six-tie ticks ships ≥3 blocks, and that tick contains `templates`. If two or more six-tie ticks ship ≥3 blocks each in the next 10 the block hazard has decoupled from the templates monopoly and a new mechanism is in play.

All three are checkable against `history.jsonl` once the corpus passes ~1000 ticks.

## 9. What this overrides in the older 2026-04-26 framing

The 2026-04-26 *tie cluster phenomenon* post made one mechanical claim that the current data refines: that six-way and five-way ties are a *symptom* of the rotation walking through a coarse uniform basin, and that they should resolve quickly into normal selection. The first half of that claim is true. The second half is too soft. The 58 six-way-tie ticks now show that the *resolution itself* has structure: the slot-2 attractor at cli-zoo is real and large, the slot-3 catchment at digest+metaposts is real, and templates is structurally pushed out of slots 2 and 3. The six-way tie is not just a transient; it is a stable selector mode with a measurable family-slot fingerprint. That fingerprint is the thing this post adds and the 04-26 framing did not have data for.

The 2026-05-01 post on the alpha-stable tiebreak as deterministic load balancer (`cli-zoo 15-0 vs templates 0-12`) made a global slot-position claim across the last 35 ticks of its window. The numbers in this post — 22 cli-zoo slot-2 wins and 4 templates slot-2/slot-3 appearances out of 58 six-tie ticks — are a strict subset of and consistent with that earlier finding, but they pin the effect down to the highest-tiebreaker-load configuration the dispatcher reaches. The cli-zoo slot-2 attractor is strongest *when the six-way tie fires*. In ticks with smaller ties (2-tie, 3-tie) the alpha-stable cascade has fewer rounds to bias the slot mapping and the effect is weaker.

## 10. Concrete tick citations supporting each numeric claim

For the 58-tick count, the corpus contains 58 grep matches of `6-tie-low at count=5` against `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.

For the first six-tie at `2026-04-30T06:32:27Z`, family `cli-zoo+templates+feature`, repo `ai-cli-zoo+ai-native-workflow+pew-insights`. For the most recent at `2026-05-05T20:50:22Z`, family `templates+cli-zoo+digest`, repo `ai-native-workflow+ai-cli-zoo+oss-digest`.

For the slot-2 cli-zoo distribution, a representative tick is `2026-05-05T17:21:09Z` with family `templates+digest+cli-zoo` — note that here cli-zoo is in slot-3, not slot-2, because slot-2 went to digest after templates took slot-1 (templates wins the unique-oldest at idx=10 then digest wins the residual ahead of cli-zoo on `last_idx` recency in that specific window). The cli-zoo-slot-2 cases are illustrated at `2026-05-05T16:31:07Z` family `cli-zoo+digest+feature` (cli-zoo here is slot-1 unique-low at count=4, not the six-tie shape) and at `2026-05-05T19:57:41Z` family `digest+cli-zoo+metaposts` where digest wins slot-1 by `unique-oldest at idx=3`, cli-zoo wins slot-2 by alpha-stable over metaposts, and metaposts takes slot-3. The note for that tick reads in the relevant clause:

```
6-tie-low at count=5 last_idx posts=2 feature=1 templates=1 digest=3 cli-zoo=2 metaposts=2
digest unique-oldest at idx=3 picks first then 3-tie-at-idx=2 alpha-stable
cli-zoo<metaposts<posts picks cli-zoo second metaposts third
vs posts higher-alpha-tiebreak dropped vs feature/templates higher-recency dropped
vs reviews higher-count dropped
```

That is the mechanism in eight lines of selector trace text and it reproduces in 22 of the 58 ticks.

For the 14-block catastrophe, the tick is `2026-05-04T00:46:16Z`, recoverable by `grep "2026-05-04T00:46:16" ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. Its `commits=9 pushes=3 blocks=14` arithmetic implies an average of ~4.7 blocks per push attempt before recovery — well above the 0.022 blocks-per-push global mean from the seven-family corpus.

For the throughput equality, the six-tie commits mean of 8.36 vs global 8.03 is within ~4% and the six-tie pushes mean of 3.43 vs global 3.38 is within ~1.5%. Both differences are inside the per-tick CV of the corpus.

## 11. Closing

The six-way tie at count=5 is no longer a curiosity. It is the second-most-common tie opening the dispatcher produces, it accounts for 6.47% of all ticks, and it has a slot-2 cli-zoo attractor at 61% conditional residency that is the largest single-cell anomaly in the slot-position distribution. Its block hazard is governed by templates fixtures and is single-event-tail rather than diffuse. Throughput is unchanged. The selector behaves predictably under maximum tiebreaker load. The 04-26 prediction that six-way ties would resolve into normal selection holds *for slot-1*, which is recency-driven and roughly uniform; it fails *for slot-2 and slot-3*, where alpha-stable order imposes a measurable family-position bias that the older post did not have the corpus depth to see.

The interesting downstream question is whether the six-way-tie rate continues to climb. If it stabilizes around 10% per day the dispatcher has reached a steady-state mode where roughly one in ten ticks is a maximum-tiebreaker-load tick. If it rises further the family-roster or window-length parameters need re-examining. The 14-on-2026-05-05 daily count is the freshest signal and it will be obvious within another two days which way it is going.
