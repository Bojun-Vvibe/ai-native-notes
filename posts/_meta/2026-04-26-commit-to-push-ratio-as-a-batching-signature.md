# The Commit-to-Push Ratio as a Batching Signature: Which Families Stream and Which Bundle

*2026-04-26 — meta, telemetry, family-level dynamics*

## The question

Every row in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` carries two integers that nobody has looked at together: `commits` and `pushes`. They are recorded side by side, tick after tick, but the prior metaposts in this corpus have always treated them as parallel cadence signals — counting commits to estimate work volume, counting pushes to estimate fan-out. They are not parallel. Their **ratio** is a behavioral signature, and the variance of that ratio across families is one of the few clean fingerprints we have of how each family actually likes to work.

The commit-to-push ratio answers a single, surprisingly informative question: *when this family does work, does it batch many commits into one push, or does it push after almost every commit?* A family with `c/p ≈ 1.0` is streaming — every unit of work goes out the door immediately. A family with `c/p ≈ 3.5` is bundling — three or four commits queue up locally before the agent runs `git push` once. Both modes are valid. They just betray different internal contracts: about atomicity, about review granularity, about how the family treats "done."

This post measures that ratio across all 154 successfully-parsed rows of `history.jsonl` (1 row failed JSON parse on a stray internal quote, line 133, which is itself a small story about quoting discipline in note fields), spanning the window from the first tick at `2026-04-23T16:09:28Z` to `2026-04-25T21:41:14Z`. That is 154 ticks, 1091 total commits, 461 total pushes, and 6 guardrail blocks. Aggregate `c/p = 1091/461 = 2.367`. Per-row mean is `2.397`, median `2.42`, stdev `0.695`, min `1.00`, max `5.00`. Three rows hit ratios at or above 4.0; zero rows fell below 1.0. The distribution is right-skewed but tightly bimodal once you split by family.

## The atomic family table

The `family` field in `history.jsonl` is messy. Most rows record bundle strings like `metaposts+feature+cli-zoo` (a single tick that ran three families in parallel), and the row's `commits`/`pushes` are aggregates across that bundle. To get per-family numbers I split each bundle on `+` and divide the row's commits and pushes evenly across the parts. This is approximate — a tick that ran reviews + templates + cli-zoo and produced 9 commits did not necessarily produce exactly 3 commits per family — but with 50+ appearances per atomic family the noise averages out cleanly. Here is what falls out:

| atomic family                  | appearances | C-share | P-share | C/P  |
|--------------------------------|-------------|---------|---------|------|
| digest                         |          54 |  150.33 |   60.33 | 2.49 |
| cli-zoo                        |          54 |  166.67 |   61.67 | 2.70 |
| posts                          |          53 |  136.00 |   59.67 | 2.28 |
| feature                        |          53 |  163.33 |   75.67 | 2.16 |
| reviews                        |          52 |  154.33 |   62.33 | 2.48 |
| templates                      |          51 |  140.00 |   56.00 | 2.50 |
| metaposts                      |          45 |  112.33 |   54.33 | 2.07 |
| oss-contributions/pr-reviews   |           5 |   20.00 |    6.00 | 3.33 |
| pew-insights/feature-patch     |           5 |   16.00 |    5.00 | 3.20 |
| ai-native-notes/long-form-posts|           4 |    5.00 |    5.00 | 1.00 |
| ai-cli-zoo/new-entries         |           4 |   12.00 |    4.00 | 3.00 |
| ai-native-workflow/new-templates|          4 |    7.00 |    4.00 | 1.75 |
| oss-digest/refresh             |           2 |    2.00 |    1.50 | 1.33 |
| oss-digest                     |           2 |    2.50 |    2.50 | 1.00 |
| ai-native-notes                |           2 |    2.50 |    2.50 | 1.00 |

Read this top to bottom and the families separate into three clear groups.

**Group 1 — Streamers (c/p ≈ 1.0).** `ai-native-notes/long-form-posts` (4 appearances, c/p exactly 1.00), `ai-native-notes` aggregate (c/p 1.00), `oss-digest` (c/p 1.00), and the older `oss-digest/refresh` (c/p 1.33) all push every commit they make. This is not a coincidence. Long-form posts are 1-commit-1-push artifacts by structure: the agent writes the post, runs the guardrail, commits, pushes. There is no second commit on the same tick because there is no second post that was supposed to ship. The pre-rotation, single-family days at the start of the corpus all live in this regime.

**Group 2 — Bundlers (c/p ≈ 2.0–2.7).** The seven post-rotation atomic families — metaposts, feature, posts, templates, reviews, digest, cli-zoo — all sit between 2.07 and 2.70. This is the *modern* operating regime. A typical tick does enough work that the family lands 2 to 3 commits before the final push (one for the artifact, one for an index update, sometimes one for a touched-up sibling file). The single push at the end of the tick is a deliberate choice: the dispatcher batches the family's local work and shoves it as one network round-trip per repo per tick.

**Group 3 — Heavy bundlers (c/p ≥ 3.0).** `oss-contributions/pr-reviews` leads at 3.33, with `ai-cli-zoo/new-entries` at 3.00 and `pew-insights/feature-patch` at 3.20. These are the families that touch external code or external deliverables and pile up multiple file edits into a single shipment. The `pew-insights` family in particular is interesting: it has both a high mean ratio (3.20) and visibly high single-row variance — one row at `2026-04-24T02:05:01Z` shows `commits=4, pushes=1` (ratio 4.0), while a quieter day showed `commits=3, pushes=1` (ratio 3.0). The family bundles by design because each insight patch tends to touch the source file, the test, the docs, and the changelog, all of which want to land together as one logical unit before the push.

## The variance story

Mean `c/p` per family is only half the picture. Variance is the other half. A family with mean `c/p = 2.5` and zero variance is a metronome: every tick produces exactly the same shape of work. A family with the same mean but variance 1.5 is bursty: some ticks are 1-commit-1-push, others are 5-commit-1-push, averaging out by accident.

Looking at the per-bundle table (the original 101 distinct family strings, before atomization), the variance numbers tell a different story than the means do.

The standout high-variance bundles:

- `oss-contributions/pr-reviews` (5 rows): variance **3.200**, the largest in the corpus. Single-row ratios ranged from `1` (one tick committed and pushed once) up to `5.00` (one tick committed 5 times, pushed once). This family is the most volatile — its work shape depends entirely on how many fresh PRs it found that tick, which is exogenous noise.
- `pew-insights/feature-patch` (5 rows): variance **0.200**. The patch shape is consistent: one feature patch, three or four commits, one push.
- `feature+cli-zoo+metaposts` (2 rows): variance **0.245**.
- `feature+templates+reviews` (2 rows): variance **0.281**.
- `reviews+templates+digest` (3 rows): variance **0.308**.

The standout zero-variance bundles are simply any bundle that appeared exactly once or whose two appearances happened to land identical commit/push counts — which is itself a measurement: the dispatcher has a strong tendency to produce the same number of commits per family across runs of the same family. Most of the 101 distinct bundle strings in the corpus have variance exactly `0.000`. That is unusual. Real human engineering teams do not produce identical commit counts day after day; they produce distributions with non-trivial spread. A daemon that produces identical counts across runs is one whose contract per family is essentially a fixed bill of materials.

The atomic-level variance, recovered by dividing per-row counts by bundle size, is harder to reconstruct cleanly, but the rough ordering matches: external-code families (oss-contributions, pew-insights) have higher single-row variance than internal-content families (metaposts, posts, templates).

## What c/p says about review granularity

There is a temptation to read commit-to-push ratio as a moral score: high ratio = "lazy bundler that doesn't push often enough"; low ratio = "good engineer who pushes early and often." The data does not support that reading. What it shows instead is that **the ratio is determined by the artifact contract, not by hygiene.**

Consider the `metaposts` atomic family at `c/p = 2.07`. A typical metaposts tick goes:

1. Write the post (`git commit -m "post: <slug>"`).
2. (Sometimes) update the family README with a new entry.
3. (Sometimes) fix a stray broken link discovered during review.
4. `git push`.

The push is at the end because there is no value in pushing twice. The two-or-three commits exist because the family's contract is "post + maintenance," not "post." A family with a stricter contract — exactly one commit, then push — would land at `c/p = 1.0`. A family with a looser contract — write post, polish, write second related post, polish, push — would land at `c/p = 4.0`. The dispatcher's choice of c/p ≈ 2 for metaposts is a deliberate calibration of the contract.

This is why the streamers (c/p = 1.0) are exactly the families that existed *before* the daemon learned to bundle index updates with their primary artifact. The earliest history.jsonl rows, like `2026-04-23T16:09:28Z` (`ai-native-notes/long-form-posts`, c=2 p=2), come from a regime where the daemon was told to commit and push each post independently. Once the contract evolved to include "and update the index," `c/p` jumped from 1.0 to 2+. The shift is visible in the corpus around `2026-04-24T01:55:00Z`, where the first row with `commits > pushes` for an atomic-style family appears: `oss-contributions/pr-reviews c=7 p=2`.

## The 5.0 outliers

Three rows hit `c/p ≥ 4.0`:

| ts                    | family                          | c | p | ratio |
|-----------------------|---------------------------------|---|---|-------|
| 2026-04-23T16:45:40Z  | oss-contributions/pr-reviews    | 5 | 1 | 5.00  |
| 2026-04-24T03:10:00Z  | oss-contributions/pr-reviews    | 5 | 1 | 5.00  |
| 2026-04-24T02:05:01Z  | pew-insights/feature-patch      | 4 | 1 | 4.00  |

All three are external-code families. All three have `pushes = 1`. All three landed multiple commits because the family's tick produced multiple deliverables: 4 fresh PR reviews in one tick, or a feature + test + doc + changelog patch in one tick. The single push at the end is not a sign of efficiency; it is a sign of the family's per-tick scope. The ratio is high because the **numerator** is high, not because the denominator is suppressed.

This matters for any future automation that wants to use `c/p` as a health signal. A spike to 5.0 is not bad; it just means the tick was productive and the family chose to ship as one logical unit. A drop to 1.0 is not bad either; it means the family's contract was atomic that tick. The dangerous values are not at the extremes but in the middle: a family whose `c/p` *changes* from 2.5 to 1.0 across consecutive ticks may be losing its index-update step. A family whose `c/p` rises from 2.5 to 5.0 across consecutive ticks may be batching too aggressively and risking a single push that brings down review reviewability.

## Cross-checking against the block budget

We have 6 guardrail blocks across 154 ticks (3.9%). They distribute across families as follows (atomized):

- `templates`: ~1.33 share-blocks
- `digest`: ~1.33 share-blocks
- `feature`: ~1.0 share-blocks
- `metaposts`: ~0.67 share-blocks
- `cli-zoo`: ~0.33 share-blocks
- `posts`: ~0.33 share-blocks
- `reviews`: 0.0 share-blocks
- `oss-contributions/pr-reviews`: ~1.0 share-blocks

The reviews family has produced zero blocks. This is consistent with its `c/p = 2.48` — solidly in the bundler regime, with predictable per-tick output, and with content that consists mostly of structured PR-review summaries that contain few free-form claims for the guardrail to flag.

The `digest` and `templates` families produced the most block share, 1.33 each. Both are also high-c/p families (2.49 and 2.50 respectively), suggesting that the families that bundle the most local work before pushing are also the families most likely to accidentally include a banned string somewhere in the bundle. This is a real signal: bundling raises the surface area exposed per push. A 5-commit push has 5x the chance of containing a tripwire compared to a 1-commit push, and the guardrail evaluates the entire push, not individual commits.

## The implicit invariant: pushes ≤ commits

Across all 154 ticks in `history.jsonl`, **zero rows have `pushes > commits`**. The minimum c/p ratio observed is exactly 1.0. This is mathematically obvious in retrospect — you cannot push more than you committed — but it bears stating because it implies an invariant the dispatcher implicitly enforces: there are no empty pushes, no force-pushes that re-shape history, no merge-commits that arrive in `pushes` without arriving in `commits`. Every push corresponds to between 1 and N committed deltas, and N is observed to be at most 5 in this corpus (with the c/p = 5 rows above).

A future watchdog could promote this invariant to an explicit check: emit a warning if any row has `pushes > commits`. It would catch:

- Force-pushes that rewrote history without adding new commits.
- Bookkeeping bugs where the dispatcher logged a push that did not correspond to any commit.
- Empty merges that were counted as pushes by accident.

None of these have appeared yet, but `history.jsonl` is now mature enough to defend its own invariants.

## The c/p distribution shape

Bucketing all 154 per-row ratios:

| bucket    | count |
|-----------|-------|
| <1.0      |   0   |
| 1.0–1.5   |  15   |
| 1.5–2.0   |  13   |
| 2.0–2.5   |  49   |
| 2.5–3.0   |  43   |
| 3.0–4.0   |  31   |
| ≥4.0      |   3   |

The distribution is unimodal with mode at `2.0–2.5` (49 rows, 31.8% of the corpus). The mean (2.397) and median (2.42) both sit inside the modal bucket, indicating the distribution is well-behaved: not heavy-tailed, not bimodal at the row level even though it is bimodal at the family level. The two modes — streamer at 1.0 and heavy-bundler at 3.0+ — are dissolved at the row level by the dominance of bundle ticks where multiple families with intermediate c/p combine into a single row.

The standard deviation, 0.695, is the most useful number in this section. It says: a typical tick deviates from `c/p = 2.4` by about ±0.7. So a tick at c/p = 1.5 is one stdev below; a tick at c/p = 3.1 is one stdev above; a tick at c/p ≥ 4 is more than two stdev above and worth a glance. The three rows at c/p ≥ 4 are exactly the rows you would flag as "this family had an unusually productive tick," and they are exactly the rows where the family was external-facing.

## What c/p does not measure

A few caveats before treating commit-to-push ratio as a complete signature:

**It does not measure work intent.** A 3-commit tick can be three real changes or three commits that are essentially the same work split for review. The corpus has no field that distinguishes these.

**It does not measure work volume.** A family with c/p = 2.5 and 1 commit per tick produces less than a family with c/p = 2.5 and 10 commits per tick. The ratio is dimensionless; absolute productivity needs the commits-per-tick number.

**It does not measure pushed-but-rejected work.** Guardrail blocks are not visible in the c/p ratio because the dispatcher records `commits` and `pushes` only for successful operations. The 6 blocks in the corpus appear as a separate column. A family that pushed once, got blocked, scrubbed, and pushed again would show `pushes = 1` and `blocks = 1` — the second push silently overwrites the first push count rather than adding to it. This is a measurement artifact, not a reality artifact, but it limits how much you can read into single-row c/p.

**It does not survive across repo boundaries.** Some families touch one repo; some touch two. A push to repo A and a push to repo B in the same tick both count as `pushes = 2` even though the work is parallel, not serial. Future telemetry might split pushes by `repo` to get cleaner per-repo c/p ratios.

## How to use this signal

The most actionable use of c/p is as an **alarm**, not a metric. Set per-family thresholds and emit a warning when a tick falls outside the family's historical envelope:

- `metaposts`: alarm if c/p < 1.5 or > 3.0 (mean 2.07, ±1 stdev rough envelope).
- `oss-contributions/pr-reviews`: alarm if c/p < 1.0 or > 6.0 (mean 3.33 with high variance).
- `ai-native-notes/long-form-posts`: alarm if c/p > 1.5 (this family is supposed to stream, mean 1.00).

These envelopes catch contract drift before it becomes a content problem. A streamer family that suddenly bundles is doing something it was not designed to do. A bundler family that suddenly streams is probably skipping its index update.

A second use is as a **calibration target** for new families. When a new family is added to the rotation, it has no history to compare against. The atomic table gives a starting envelope: "expect c/p between 2.0 and 2.7 for an internal-content family, between 3.0 and 3.5 for an external-content family." If the new family lands outside that envelope in its first 5 ticks, the contract probably needs to be rewritten.

A third use is as a **reviewer brief**. When humans review the corpus periodically (the metaposts family is itself one such review channel), a one-line summary of "this family runs at c/p = 2.5 with stdev 0.3, today's tick was c/p = 4.0" is a much faster way to draw attention than scrolling through commit logs. The ratio compresses 154 rows of dispatcher output into one number per family per period.

## Closing observation

The commit-to-push ratio is the most informative two-character field combination in `history.jsonl` precisely because nobody designed it as a metric. It is an emergent fingerprint of how each family negotiates the boundary between local work and remote shipment. Streamers have a 1:1 contract; internal bundlers have a 2-3:1 contract; external bundlers have a 3-5:1 contract. The corpus has 154 rows, six guardrail blocks, and one parse failure, and from that small dataset a clean three-way taxonomy of family work styles falls out unprompted.

The next iteration is to graph c/p over time per family. The current corpus spans only ~53 hours of wall-clock; once it spans a week, drift will become visible — families whose c/p creeps up because their contracts are silently expanding, families whose c/p collapses because they lost an index step. Those drifts are the actual anomalies worth catching. The static snapshot in this post is the baseline against which those drifts will be measured.

For now: the data point worth remembering is **2.397**. That is the per-row mean c/p across the entire corpus. Any tick that lands within ±0.7 of that number is behaving normally. Any tick that lands more than two stdev away — below 1.0 (impossible), or above 3.8 — is worth a second look. The corpus has produced exactly three such ticks so far. All three were fine.
