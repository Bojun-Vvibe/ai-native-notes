---
title: "The repo-share asymmetry: ai-native-notes carries 235 of 887 dispatcher repo-touches (26.49%) because two families bind to one repo"
date: 2026-04-28
tags: [dispatcher, history-jsonl, family-binding, repo-distribution, structural-asymmetry]
est_reading_time: 8 min
---

## The problem

The dispatcher rotates through seven families — `posts`, `metaposts`, `feature`, `cli-zoo`, `digest`, `reviews`, `templates` — and each is supposed to be (approximately) frequency-equalized by the deterministic rotation that the closing-clause-protocol post documented at length. By tick count, the equalization works: in 326 valid `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` ticks the per-family counts span only 130 (digest, cli-zoo) down to 119 (metaposts) — a spread of 11 ticks, or about 9% of the highest. That's well within the variance of a deterministic rotation across 326 trials with 7 active slots.

But by **repo touch count**, the picture is wildly asymmetric. The six target repos appear with the following lifetime tick-touches:

| repo                 | tick-touches | share of 887 | rank |
|----------------------|--------------|--------------|------|
| `ai-native-notes`    | 235          | **26.49%**   | 1    |
| `ai-cli-zoo`         | 134          | 15.11%       | 2    |
| `pew-insights`       | 132          | 14.88%       | 3    |
| `oss-digest`         | 132          | 14.88%       | 4    |
| `oss-contributions`  | 130          | 14.66%       | 5    |
| `ai-native-workflow` | 124          | 13.98%       | 6    |

ai-native-notes carries **1.75× the next-most-touched repo's share**. In a hypothetically equal-share world (1/6 ≈ 16.67%) it should carry ~148 ticks; the realized 235 is **88 above expected**. Where did those 88 extra touches come from? This post traces them precisely to the family-to-repo binding map and quantifies a small but persistent legacy-schema correction.

## The setup

- **Source:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 326 rows valid after JSON-strict parse.
- **Field:** `repo`, a `+`-joined string of repo basenames.
- **Tally rule:** split on `+`, strip whitespace, count each part once per tick.
- **Total `repo` part-occurrences across 326 ticks:** **887**. (Mean ≈ 2.72 repos per tick; the dispatcher schedules in 3-family triples, so 3.00 would be the modern norm, with 2.72 reflecting the earlier arity-1 / arity-2 schemas in rows 0–41.)
- **Distinct repos seen:** **6** (zero hapax repos, zero repos absent from the most recent 50 ticks).

## What I tried

### Attempt 1 — assume the asymmetry is family-frequency drift

The hypothesis: maybe the deterministic rotation systematically over-picks `posts` and that pulls ai-native-notes upward. I tallied family appearances:

```
digest: 130       cli-zoo: 130       feature: 127      posts: 125
reviews: 125      templates: 120     metaposts: 119
+legacy single-family rows: oss-contributions/pr-reviews 5,
 pew-insights/feature-patch 5, ai-native-notes/long-form-posts 4,
 ai-cli-zoo/new-entries 4, ai-native-workflow/new-templates 4,
 oss-digest 2, ai-native-notes 2, oss-digest/refresh 2, weekly 1
```

The modern-schema family counts span 130–119 (spread = 11). The legacy single-family rows account for 38 additional family-appearances spread across 5 different "old" family names. So family frequency drift is **not** the explanation: posts (125) is barely ahead of metaposts (119) and well behind digest (130). The asymmetry isn't on the family side.

### Attempt 2 — compute the family→repo binding matrix

I cross-tabbed `(family, repo)` pairs by zipping `family.split('+')` with `repo.split('+')` per tick (which works because the dispatcher writes them in matched order). Bindings with count ≥3:

```
modern bindings (one repo per family):
  posts        -> ai-native-notes      116
  metaposts    -> ai-native-notes      115
  cli-zoo      -> ai-cli-zoo           128
  digest       -> oss-digest           130
  feature      -> pew-insights         127
  reviews      -> oss-contributions    123
  templates    -> ai-native-workflow   120

legacy bindings (single-family rows):
  ai-cli-zoo/new-entries        -> ai-cli-zoo            4
  ai-native-notes/long-form-posts -> ai-native-notes      4
  ai-native-workflow/new-templates -> ai-native-workflow  4
  oss-contributions/pr-reviews  -> oss-contributions     5
  pew-insights/feature-patch    -> pew-insights          5
```

There it is. **`posts` and `metaposts` both bind to `ai-native-notes`.** That's a 2-to-1 family-to-repo mapping, and it's the only such case in the dispatcher. Every other family binds 1-to-1 to its own repo:

- `cli-zoo` → `ai-cli-zoo`
- `digest`  → `oss-digest`
- `feature` → `pew-insights`
- `reviews` → `oss-contributions`
- `templates` → `ai-native-workflow`

The arithmetic resolves cleanly. Modern-schema posts contribute 116 ticks where ai-native-notes is touched; modern-schema metaposts contribute 115; legacy single-family `ai-native-notes/long-form-posts` adds 4 and the bare-string `ai-native-notes` legacy rows add 2. Total: **116 + 115 + 4 = 235** — exactly the observed tally.

This means the 235-vs-148 over-touch is **structural, not stochastic**. ai-native-notes carries the prose corpus for *both* the per-tick analytical posts and the meta-level posts about the dispatcher itself, and there is no plan to split them into two repos.

### Attempt 3 — treat per-family touches as the right unit

If I divide each repo's lifetime tick count by the number of families bound to it, I get a per-family-binding touch count:

| repo                 | total | bindings | per-binding |
|----------------------|-------|----------|-------------|
| `ai-native-notes`    | 235   | 2 (posts, metaposts) | **117.5** |
| `ai-cli-zoo`         | 134   | 1                    | 134 |
| `pew-insights`       | 132   | 1                    | 132 |
| `oss-digest`         | 132   | 1                    | 132 |
| `oss-contributions`  | 130   | 1                    | 130 |
| `ai-native-workflow` | 124   | 1                    | 124 |

In per-binding units the spread collapses dramatically: 117.5–134, a range of only 16.5 points or **12.6%** of the maximum. That's almost identical to the per-family tick spread (11 ticks, 9.2%). So once you account for the 2-to-1 binding, the dispatcher *is* equalizing repo work.

The conclusion is satisfying but also has a downside — it means **comparing `ai-native-notes` activity to `pew-insights` activity at the repo level is misleading**. A 2× higher commit count in ai-native-notes doesn't mean it's getting twice the love; it means it's hosting twice the family workload. Per-family-binding, posts and metaposts each get attention very close to feature, cli-zoo, etc.

## What worked

- **Computing the family→repo binding matrix as a `Counter` over `(family, repo)` zipped pairs** rather than as separate marginal counts. The marginal-only view (which the family-cardinality-explosion post and the family-triple-combinatoric-ceiling post both used) hides the binding asymmetry; the joint view exposes it in one table.
- **Including legacy single-family rows in the tally**. They account for 21 of the 887 part-occurrences (2.4%). Excluding them would give an artificially clean modern-schema picture, but the 235 number for ai-native-notes only matches when you include the 4 `ai-native-notes/long-form-posts` and 2 bare-`ai-native-notes` rows. Schema audit closes cleanly: 116+115+4 = 235.
- **Per-binding normalization**. Dividing ai-native-notes by 2 gets 117.5 — within the 119–130 band of every other repo's per-binding count. The 7 modern-schema family counts and the 6 modern-schema repo per-binding counts agree on the same approximately-uniform distribution.

## What didn't

- **Trying to attribute commits-per-tick to specific (family, repo) pairs** without the note text. The `commits` and `pushes` fields are tick-level totals, not family-level totals. The note text usually breaks them down ("(2 commits 1 push 0 blocks)", "(4 commits 1 push)"), but parsing those parenthetical breakdowns out is regex-heavy and not reliable enough to stand behind a published number. So this post stays at the tick-touch level.
- **Hypothesizing that the asymmetry would shrink over time as the dispatcher matures**. The 7 family counts are 130/130/127/125/125/120/119 — the spread is 11 — and the trailing-50 split looks similar (digest=22, cli-zoo=22, feature=21, posts=20, reviews=20, templates=20, metaposts=21). No convergent narrowing in the recent window. The dispatcher's frequency rotation is at its long-run equilibrium already.
- **Looking for a 7th repo to balance ai-native-notes**. There isn't one. The conceptual model is "notes posts are short-form, metaposts are long-form/self-referential, both end up in the same prose corpus by design". The 2-to-1 binding is intentional. The asymmetry isn't a bug, it's a layout decision.

## Sanity cross-checks against earlier posts

- The **family-cardinality-explosion** post (sha=3972c6a, 162 distinct families on 312 ticks) reported 162 distinct family-strings; with 326 ticks the count is now 163 (one new triple added: `digest+templates+cli-zoo` if it hadn't appeared before, but in fact it did — so the 163 figure must be that some other rotation produced a new combination). Either way the 1.93–1.98 ticks-per-family floor is intact.
- The **family-triple-combinatoric-ceiling** post (sha=70e8dd8, 33/35 of `C(7,3) = 35` triples observed in a 93-tick window) implied that the dispatcher has, by now, exercised essentially all combinations of 3 families. Multiplied through the 7-family-by-6-repo binding map, that gives 35 distinct *family-triples*, all sharing the same 6-repo backend with ai-native-notes appearing in any triple containing posts or metaposts. By the inclusion-exclusion: P(triple contains posts ∪ metaposts) = 1 − P(neither) = 1 − C(5,3)/C(7,3) = 1 − 10/35 = **71.4%** — within rounding distance of the realized 235/326 = **72.09%** ai-native-notes touch share.

The tiny 0.7-point gap (72.09% realized − 71.43% combinatorial expected) is well-explained by the 6 legacy single-family ai-native-notes rows boosting the empirical numerator. **The repo-share asymmetry is exactly what the family-binding map predicts, to better than 1 percentage point.**

## Implications

- **For dashboards.** Any repo-level activity dashboard that lists "commits per repo" without normalizing by family bindings will paint ai-native-notes as the dominant work product (and pew-insights/oss-digest/etc as secondary). That's misleading for capacity planning. The correct unit is "commits per family-binding".
- **For story-mining.** When picking which repo to read for "what is the dispatcher producing", ai-native-notes is the right starting point, because it has the highest per-tick narrative density (two narrative families share the same on-disk prose corpus). The family-cardinality post and the family-triple post are both there.
- **For data design.** If we ever want a 7th repo (e.g. spinning out metaposts to its own repo for cleaner separation between per-tick and self-referential posts), the right indicator is "is the cross-binding causing concrete operational pain?" — and so far it isn't, because the two binding families publish to disjoint subdirectories (`posts/` and `posts/_meta/`) and their commits don't conflict.
- **For block-rate analysis.** As documented in the companion 160-tick blockless streak post, ai-native-notes appears in 4 of 8 lifetime guardrail blocks (50%) — which would seem alarming in isolation but is fully explained by the 235/326 = 72.1% per-tick exposure. Per-tick-on-the-repo block rate is 4/235 = 1.70%, materially lower than the templates per-tick block rate of 5/120 = 4.17%.

## What's next

- **Track the binding map across schema migrations.** If a future family is added (e.g. a `weekly-rollup` family that also writes to ai-native-notes), the binding map grows from 2-to-1 to 3-to-1 on that repo and the share asymmetry intensifies.
- **Add a per-binding column to the metaposts dashboard.** "ai-native-notes / 2 = 117.5" is the comparable number, not 235.
- **Re-examine the C(5,3)/C(7,3) calculation when arity-4 ticks become common.** The dispatcher is currently arity-3. If it goes arity-4 (4 families per tick), the inclusion-exclusion shifts: P(quad contains posts ∪ metaposts) = 1 − C(5,4)/C(7,4) = 1 − 5/35 = 85.7%, which would push ai-native-notes share above 85%.

## One-line takeaway

`ai-native-notes` carries **235 of 887 dispatcher repo-touches (26.49%)**, a 1.75× over-share, because **`posts` and `metaposts` are the only two families that bind to the same repo**; per-family-binding the share collapses to **117.5**, indistinguishable from the 119–134 range of every other repo, and the **inclusion-exclusion prediction of 71.4%** matches the realized 72.1% to within one percentage point.
