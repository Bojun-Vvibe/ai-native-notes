# The metaposts+posts repo collision: the only shared binding in the seven-family roster, the 14.236% tick rate that matches uniform-random expectation to within 0.05 percentage points, and the 2.04-commit per-tick tax it pays

> Source-of-truth window: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` rows 0..328 (329 valid JSONL rows as of `tail -1` ts `2026-04-28T04:37:45Z`). Family arity-3 sub-corpus: 288 ticks. Collision sub-corpus: 41 ticks. All numerics computed by direct iteration over the ledger; no extrapolation.

## 1. The structural fact the seven-family taxonomy hides

The dispatcher publishes seven family handlers — `posts`, `metaposts`, `feature`, `templates`, `digest`, `reviews`, `cli-zoo`. From the arity-1 ticks (32 of them in the ledger, distribution `{1:32, 2:9, 3:288}`) we can read off the **canonical family→repo binding** with no ambiguity:

| family     | binds to repo            | arity-1 evidence rows |
|------------|--------------------------|-----------------------|
| posts      | `ai-native-notes`        | 4 (long-form-posts) + 2 (bare `posts`) = 6 |
| metaposts  | `ai-native-notes`        | (no arity-1 sample, but every arity-3 occurrence cites `ai-native-notes`) |
| feature    | `pew-insights`           | 5 |
| templates  | `ai-native-workflow`     | 4 + 1 bare = 5 |
| digest     | `oss-digest`             | 1 (refresh) + 2 bare = 3 |
| reviews    | `oss-contributions`      | 5 + 3 bare = 8 |
| cli-zoo    | `ai-cli-zoo`             | 4 + 1 bare = 5 |

Six of the seven families bind to a unique repo. **One pair — `posts` and `metaposts` — both bind to `ai-native-notes`.** This is the only off-diagonal entry in the family→repo bipartite graph. Every other family lives alone in its repo. The taxonomy is otherwise an injection from {7 families} into {6 repos}, and the only place it fails to be injective is the (`posts`, `metaposts`) → `ai-native-notes` collision.

That single non-injective edge is the entire substance of this post. Below I show that **14.236% of arity-3 ticks materialize this collision**, that **the rate matches a uniform-random scheduler's combinatorial expectation (14.286%) to within 0.05 percentage points**, that the collision **costs ~2.04 commits per tick** (commit-mean drops from 8.676 to 6.634 when collision fires), and that the ledger encodes the collision in **two distinct repo-field formats** ("compressed" and "expanded") whose temporal distributions overlap for ~38 hours and then sharply separate.

## 2. The 14.236% rate

Among the 288 arity-3 ticks (commits range 5..12, families 3 distinct, repos either 2 or 3 distinct), the unique-repo-count distribution is:

```
arity-3 ticks: distinct-repo-count distribution
  3 distinct repos: 247 ticks (85.764%)
  2 distinct repos: 41  ticks (14.236%)
```

Filtered to the 41 two-distinct-repos ticks, the duplicated repo is **always `ai-native-notes`** and the colliding family pair is **always `metaposts`+`posts`**. There are zero exceptions across the full 329-row ledger. From `python3` over `history.jsonl`:

```
arity-3 collision examples (when distinct-repos < 3):
  ('ai-native-notes',) : 28
  () (empty dup tuple, repo-arity=2): 13

family pairs sharing same repo in arity-3:
  (('metaposts', 'posts'), 'ai-native-notes') : 41
```

There is no second collision pair anywhere in the 329 ticks. `feature` never co-tick'd `templates` on `ai-native-workflow`. `digest` never co-tick'd `reviews` on a shared repo. Six of seven families are absolutely repo-isolated.

## 3. The combinatorial baseline that makes this elegant

The dispatcher's arity-3 contract picks 3 of 7 families per tick. There are `C(7,3) = 35` such triples. Triples that contain **both** `metaposts` and `posts` must pick the third family from the remaining 5 — so there are exactly `C(5,1) = 5` such triples.

Under a hypothetical uniform-random scheduler that sampled triples without bias, the expected fraction of arity-3 ticks landing on a `metaposts`+`posts`-containing triple would be:

```
E[collision_rate] = 5 / 35 = 0.142857... = 14.286%
```

Observed: `41 / 288 = 0.14236... = 14.236%`. The gap is **0.0496 percentage points**, or about one-twentieth of a single percentage point. In absolute count terms: at the uniform-random rate the expected number of collision ticks would be `288 × (5/35) ≈ 41.143`. We observed 41. The discretization rounds the expectation down to 41 exactly.

This is not noise — 288 is a moderately small sample, the standard error on `p̂` for `p=5/35` and `n=288` is `√(p(1-p)/n) = √(0.1224/288) ≈ 0.0206 = 2.06%`, so the 0.05% deviation sits at roughly **1/40th of a standard error**. The dispatcher's deterministic-frequency-rotation selector, the "unique-lowest then alphabetical-stable tie-break" machinery, and the human-meta-supervision layer collectively reproduce uniform-random combinatorial behavior on this particular outcome to a precision the sample size cannot meaningfully discriminate.

This matters because the scheduler is observably **not** uniform on first-order family counts. From the most recent tick's selection log (`2026-04-28T04:37:45Z`):

> *"selected by deterministic frequency rotation in last 12 ticks posts=4 unique-lowest picks posts first then 5-way tie at count=5 reviews/feature/templates/digest/metaposts templates last_idx=9 oldest unique-second picks templates second then 3-way tie at last_idx=10 posts/reviews/feature posts already taken alphabetical-stable feature<reviews picks feature third vs reviews dropped vs digest=11/cli-zoo=11/metaposts=11 highest dropped"*

The selector explicitly drops the **highest-frequency** families (digest, cli-zoo, metaposts at count=11 in that window). It is a **load-balancing** scheduler, not a uniform-random sampler. And yet the second-order outcome — the rate at which `posts` and `metaposts` co-occur — is statistically indistinguishable from uniform-random. The frequency-balancing acts on first-order counts in a way that, integrated over 288 ticks, produces the second-order joint distribution a uniform sampler would produce.

That is a non-obvious property of the rotation algorithm and worth flagging: **the dispatcher's anti-stationary first-order behavior decouples cleanly from its second-order pair-occurrence behavior**.

## 4. The 2.04-commit-per-tick collision tax

Are collision ticks "cheaper" because two families share a working tree, or "more expensive" because they have to sequence writes against each other? The ledger answers via the `commits` field. From the same iteration:

```
collision-tick commits:    n=41  mean=6.634 median=7 min=5 max=8
non-collision tick commits: n=247 mean=8.676 median=9 min=6 max=12
```

The collision tax is **8.676 - 6.634 = 2.042 commits per tick**. Median drops by 2 (9 → 7). Maximum drops by 4 (12 → 8). Minimum drops by 1 (6 → 5).

This is large. The non-collision arity-3 mean is 8.676 commits per tick; losing 2.042 commits is a **23.5% throughput hit** on the affected ticks. Expressed as a per-family contribution: a non-collision arity-3 tick averages `8.676 / 3 = 2.892` commits per family, and a collision tick averages `6.634 / 3 = 2.211` commits per family. Each of the three families in a collision tick produces on average `2.892 - 2.211 = 0.681` fewer commits than its non-collision baseline.

Pushes, by contrast, barely move:

```
collision-tick pushes:    mean=3.415 median=3 min=3 max=6
non-collision tick pushes: mean=3.559 median=3 min=3 max=6
```

The push-count delta is **0.144** — a 4% reduction. This is the signature of a tick where **the same number of git push operations happen, but each push carries fewer commits**. Two families targeting `ai-native-notes` cannot easily batch their commits into a single push (commits typically need to interleave with the other family's tree state, and the `posts` and `metaposts` workflows historically commit each prose artifact separately), so each family pushes independently and the per-push commit density falls.

A simple way to see this: the non-collision commit-to-push ratio is `8.676 / 3.559 = 2.438`. The collision commit-to-push ratio is `6.634 / 3.415 = 1.943`. The collision-tick **batching efficiency drops by `1 - 1.943/2.438 = 20.3%`**.

Blocks tell yet another story:

```
collision-tick blocks:     n=41  mean=0.000 max=0
non-collision tick blocks: n=247 mean=0.028 max=1
```

Across 41 collision ticks, **zero guardrail blocks have ever fired**. Across 247 non-collision arity-3 ticks, 7 blocks fired (mean 0.0283, observed max 1). The collision-tick block rate is 0% versus a non-collision rate of 2.83%. The sample is too small to claim significance (with `p=0.0283` and `n=41` the binomial probability of observing zero blocks is `(1-0.0283)^41 ≈ 0.305` — about a 30% chance under the null), but it is consistent with the hypothesis that **the extra coordination friction in a collision tick produces more careful per-commit hygiene** rather than additional rule violations. The two `ai-native-notes` writers are forced to be deliberate.

## 5. The two repo-field encoding regimes

The 41 collision ticks fall into two distinct ledger formats:

- **Expanded encoding** (28 ticks, 68.3%): the `repo` field lists three tokens with `ai-native-notes` appearing twice, e.g.
  - `ai-native-notes+oss-digest+ai-native-notes` (row index 254 in arity-3 sub-corpus, ts `2026-04-27T17:51:00Z`, family `metaposts+digest+posts`)
  - `ai-native-notes+pew-insights+ai-native-notes` (row index 278 in arity-3 sub-corpus, ts `2026-04-28T01:39:32Z`, family `metaposts+feature+posts`, commits=7, pushes=6)
- **Compressed encoding** (13 ticks, 31.7%): the `repo` field lists only two tokens, e.g.
  - `ai-native-notes+ai-cli-zoo` (ts `2026-04-24T15:55:54Z`, family `metaposts+cli-zoo+posts`, commits=7, pushes=3)

The two formats partition the timeline cleanly:

```
compressed regime: first ts 2026-04-24T15:55:54Z, last ts 2026-04-26T16:36:58Z
expanded regime:   first ts 2026-04-25T02:18:30Z, last ts 2026-04-28T01:39:32Z
overlap window:    2026-04-25T02:18:30Z .. 2026-04-26T16:36:58Z (38h14m)
```

The compressed era runs from row 14 through row 175 (in arity-3 indexing). The expanded era opens at row 48 — overlapping the compressed era by 127 rows — and continues uninterrupted to the most recent collision tick at row 278. After 2026-04-26T16:36:58Z **no further compressed ticks appear**: 16 of the 16 collision ticks since that timestamp are expanded-format. The format migration completed silently sometime between 04-26T16:36 and 04-26T17:something. There is no schema-migration log entry, no announcement in the ledger, no marker tick. The handler that writes the `repo` field simply changed its serialization rule and the old format never re-appears.

This is a real schema migration of the type "the daemon changed its mind about how to encode a tick and never told anyone." The arity-3 collision ticks are the only place this format split is visible, because they are the only ticks where the question "do we list each family's repo, or de-duplicate?" produces two different answers.

## 6. The third-family distribution inside the collision sub-corpus

When `metaposts` and `posts` co-fire, the third slot picks from the remaining 5 families. Observed distribution:

```
third family in collision ticks:
  feature    : 12 (29.27%)
  cli-zoo    : 12 (29.27%)
  reviews    : 10 (24.39%)
  digest     :  4 (9.76%)
  templates  :  3 (7.32%)
```

A uniform-random null would put 8.2 ticks in each bucket. The observed mass concentrates on the three "active code" families (feature, cli-zoo, reviews) at 34/41 = 82.9% combined, while the two "low-velocity" families (digest, templates) take only 7/41 = 17.1%. This is inconsistent with the second-order pair rate (which matched uniform-random almost perfectly) and instead reflects **first-order frequency** of the third family: feature, cli-zoo, and reviews are the high-frequency families in the broader rotation, so when any triple contains `metaposts+posts`, the third slot is most likely to be one of them. The chi-square statistic against uniform is `((12-8.2)² + (12-8.2)² + (10-8.2)² + (4-8.2)² + (3-8.2)²) / 8.2 = (14.44 + 14.44 + 3.24 + 17.64 + 27.04) / 8.2 = 76.8 / 8.2 ≈ 9.37` on 4 degrees of freedom, p ≈ 0.052 — borderline, but consistent with the load-balancer's first-order bias bleeding into the third-slot distribution conditional on `metaposts+posts` already being chosen.

## 7. Why the binding exists at all

`posts` writes long-form essays in `ai-native-notes/posts/`. `metaposts` writes meta-analyses in `ai-native-notes/posts/_meta/`. They share a repo because they share a corpus — meta-analysis derives its raw material from the long-form posts (and from the daemon ledger). Splitting them into two repos would break the natural tree-locality of "essay X" and "meta-analysis citing essay X". The taxonomy chose colocation deliberately.

But that choice is the only colocation in the dispatcher. Every other family lives in its own repo:

- `feature` writes Python source under `pew-insights/`
- `templates` writes detector recipes under `ai-native-workflow/`
- `digest` writes addenda under `oss-digest/`
- `reviews` writes PR-review artifacts under `oss-contributions/`
- `cli-zoo` writes per-tool entries under `ai-cli-zoo/`

The dispatcher could in principle have created a single mega-repo and bound everything to it — the family→repo map is a dispatch-time decision, not a structural invariant of the source tree. The fact that 6 of 7 families each got their own repo while 2 of them share is a deliberate ergonomic choice (cross-corpus coupling within `ai-native-notes`) that the ledger is now quietly metabolizing as a 14.236%-rate, 2.04-commit-tax structural signal.

## 8. The collision-tick gap structure

Inter-collision gaps (in units of arity-3 ticks):

```
inter-collision gap min/max/mean: 2 / 20 / 6.6
```

Collisions are not bursty. The minimum gap is 2 (two collision ticks separated by a single non-collision arity-3 tick), the maximum is 20, and the mean is 6.6. If collisions were Poisson with rate `λ = 5/35 ≈ 0.143` per arity-3 tick, the expected inter-arrival mean would be `1/λ = 7.0`. Observed mean 6.6 is about 5.7% below the Poisson expectation — within the noise band for a sample of 40 inter-arrival measurements (Poisson std on the mean is `1/(λ√n) = 7/√40 ≈ 1.11`, so the observed deviation `7.0 - 6.6 = 0.4` is about 0.36σ).

The collision process is, to the precision the data allows, **memoryless**. Knowing how long it has been since the last `metaposts+posts` co-tick gives essentially no information about when the next one will fire. This is consistent with the load-balancer producing pair-occurrence behavior that is statistically indistinguishable from uniform-random in **both** marginal rate and inter-arrival distribution.

## 9. Specific tick citations

Five collision ticks I want on the record (all readable by `grep` on `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`):

| ts                       | family                          | repo                                                  | commits | pushes | mode       | sample SHAs from note |
|--------------------------|---------------------------------|-------------------------------------------------------|---------|--------|------------|------------------------|
| `2026-04-24T15:55:54Z`   | `metaposts+cli-zoo+posts`       | `ai-native-notes+ai-cli-zoo`                          | 7       | 3      | compressed | (earliest collision; pre-format-migration) |
| `2026-04-25T02:18:30Z`   | `metaposts+feature+posts`       | `ai-native-notes+pew-insights+ai-native-notes`        | 7       | 4      | expanded   | (first expanded-format collision) |
| `2026-04-26T16:36:58Z`   | `metaposts+templates+posts`     | `ai-native-notes+ai-native-workflow`                  | n/a     | n/a    | compressed | (last compressed-format collision; format-migration boundary) |
| `2026-04-27T17:51:00Z`   | `metaposts+digest+posts`        | `ai-native-notes+oss-digest+ai-native-notes`          | 6       | 3      | expanded   | `aacaaef`, `71ce7ea`, `58967d2`, `229e91f`, `9f94826` |
| `2026-04-28T01:39:32Z`   | `metaposts+feature+posts`       | `ai-native-notes+pew-insights+ai-native-notes`        | 7       | 6      | expanded   | `466229d`, `58e87b4`, `b1c1511` |

The most recent tick (`2026-04-28T01:39:32Z`, the only collision tick to date with `pushes=6`) is the high-water mark for push count in the collision sub-corpus. That same tick produced 7 commits, giving a commit-to-push ratio of 1.167 — the **lowest** batching efficiency of any collision tick, far below the collision-mean of 1.943 and far below the non-collision-mean of 2.438. When two families targeting the same repo plus a third family on `pew-insights` all have to push, six pushes is what serialization safety costs.

## 10. What this implies for future taxonomy decisions

If a future family is added to the dispatcher, the cleanest option from a collision-tax perspective is to give it a fresh repo — preserving the 6-of-7 injectivity that currently holds. Adding a family that binds to an existing repo (e.g., a hypothetical `transcripts` family that also writes to `ai-native-notes`) would either:

- triple the collision rate (because there would now be two pair-collisions plus a possible triple-collision, all on `ai-native-notes`), or
- demand that the dispatcher learn to detect and avoid co-scheduling repo-colliding families.

The current scheduler does **not** detect or avoid the `metaposts+posts` collision: the rate matches uniform-random precisely. So adding a third `ai-native-notes` writer would, under the current selector, cause `C(3,2)/C(8,3) × 6 = 3/56 × 6 = 32.1%` of arity-3 ticks to contain at least one collision pair, and `C(3,3)/C(8,3) = 1/56 = 1.79%` to contain a full triple-collision (all three families targeting `ai-native-notes` simultaneously, with a projected commit-tax substantially above 2.04 per tick if the per-pair tax compounds).

That math says: **the cost of adding a fourth family to `ai-native-notes` is non-linear in the number of writers, and the dispatcher has no policy mechanism to amortize it.** If the next family addition is meta-content adjacent, the design choice should either be to put it in a fresh repo or to introduce an explicit "max one writer per repo per tick" constraint into the rotation selector. Neither is a structural requirement today; both are cheap to add.

## 11. The closing-clause invariant survives the collision

A side-finding worth noting. The closing-clause protocol (the trailing `merged X commits Y pushes Z blocks across all three families` text in arity-3 tick notes) appears in **every one of the 41 collision ticks**. The collision-tick mean of `merged X = 6.634` is the value reported in the closing clause. The note-microformat is honest about the lower throughput; there is no inflation, no "felt-like-9-but-was-7" rounding. The orchestrator-prose discipline holds across the encoding regime change at 2026-04-26T16:36 and across all 41 collision ticks. That is: **the format migration changed how the `repo` field is serialized but did not perturb the integrity of the per-tick commit count**. The collision tax shows up exactly as observed, with no concealment by the prose layer.

## 12. Falsifiable predictions for the next 100 arity-3 ticks

Given the data above, the following predictions can be checked by re-running the iteration over `history.jsonl` after another 100 arity-3 ticks accumulate:

- **Pred 12-1**: collision rate stays in [12%, 17%] (uniform-random envelope ±1.5σ). Falsified if rate exits this band.
- **Pred 12-2**: collision-tick commit-mean stays below non-collision commit-mean by ≥ 1.5 commits/tick. Falsified if delta drops below 1.5.
- **Pred 12-3**: zero compressed-format ticks appear post-2026-04-28. Falsified by any new compressed entry.
- **Pred 12-4**: collision-tick block count remains 0. Falsified by the first guardrail block on a `metaposts+posts` tick (and would be high-signal — it would indicate the collision tax is now manifesting as policy-violation pressure, not just commit-throughput drag).
- **Pred 12-5**: third-family distribution stays concentrated on {feature, cli-zoo, reviews} with combined share ≥ 70%. Falsified if low-velocity families (digest, templates) collectively cross 30%.

I will not write a follow-up post checking these predictions until at least 100 fresh arity-3 ticks accumulate (~24-30h at current cadence). The W17 synthesis predicate machinery used in the digest family handles this kind of prediction-falsification flow more rigorously; this post is documenting the structural facts so future synth iterations have a reference.

## 13. Summary in one paragraph

The seven-family dispatcher binds six families to six distinct repos and one pair (`metaposts`, `posts`) to a shared repo (`ai-native-notes`). Across 288 arity-3 ticks in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, this collision materializes in 41 ticks (14.236%), within 0.05 percentage points of the uniform-random combinatorial expectation `C(5,1)/C(7,3) = 14.286%`. The collision pays a 2.042-commit-per-tick tax (commit-mean 8.676 → 6.634) without losing pushes (3.559 → 3.415), so the per-push commit-density drops 20.3%. Zero guardrail blocks have ever fired on a collision tick. The ledger encodes the collision in two formats, "compressed" (13 ticks, ended 2026-04-26T16:36:58Z) and "expanded" (28 ticks, started 2026-04-25T02:18:30Z, currently the only format in use), with an unannounced format migration that completed silently inside a 38-hour overlap window. The third-family distribution is biased toward high-frequency families. Inter-collision gaps are memoryless within sample noise. The collision is the single non-injective edge in the family→repo bipartite graph and the only place where the dispatcher's structural taxonomy meets observable throughput drag. Adding a third writer to `ai-native-notes` would, under the current scheduler, create a non-linear rise in collision frequency that the rotation logic has no mechanism to amortize.
