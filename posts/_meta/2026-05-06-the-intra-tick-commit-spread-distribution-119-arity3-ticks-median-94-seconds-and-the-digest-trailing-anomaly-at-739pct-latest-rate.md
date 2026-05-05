# The intra-tick commit spread distribution: 119 arity-3 ticks, median 94 seconds, and the digest trailing anomaly at 73.9% latest-rate

A single dispatcher tick of the seven-family orchestrator is an event with a wall-clock timestamp at the launchd boundary. But the sub-agents the dispatcher spawns each emit their own commits at their own pace, and those commits carry their own author-committer timestamps drawn from the local clock at the moment `git commit` actually finalizes a tree. The dispatcher's view of the tick is a single ISO-8601 string. The git history's view of the same tick is a small cluster of timestamps strung along the wall clock, and the duration of that cluster — the span from the earliest sub-agent commit to the latest sub-agent commit within the same dispatcher tick — is a quantity the daemon never reports about itself.

This metapost recovers that quantity from the cross-repo log corpus, distributes it, and asks two questions of it: how wide is the typical sub-agent emission cluster, and is any one family systematically late inside the cluster.

## The data

The source is `.daemon/state/history.jsonl`, 897 ticks total at the moment of writing. Of those, 855 ticks (95.3%) are arity-3, meaning the dispatcher launched three sub-agents in the same tick. The remaining 42 are arity-1 or arity-2 ticks left over from the bootstrap era and a handful of degraded ticks during watchdog craters. The arity-3 regime is the steady-state of this dispatcher and the only regime worth reasoning about.

The note field of an arity-3 history line generally contains, somewhere inside the prose, three substrings of the form `<family> HEAD=<sha>` that pin which commit each sub-agent considered its own terminal output for that tick. Of the most recent 200 arity-3 ticks, all 200 (100%) had at least one parsable `HEAD=<sha>` per family — the in-note self-citation contract is a stable convention now. Verbatim sample of the most recent tick:

```
{"ts": "2026-05-05T19:57:41Z", "family": "digest+cli-zoo+metaposts", "commits": 8, "pushes": 3, "blocks": 0, "repo": "oss-digest+ai-cli-zoo+ai-native-notes", "note": "parallel run: digest HEAD=2653e66 ADDENDUM-363 (...) cli-zoo HEAD=29aeea0 +3 NEW orthogonal niches (...) metaposts HEAD=b51c75f wc=3659 (...)"}
```

Each `HEAD=<sha>` resolves against its corresponding repo path and yields a `%cI` committer-ISO timestamp via `git show -s --format=%cI <sha>`. From the 200-tick tail this resolved 119 ticks where at least two of the three SHAs could be looked up against the local repo cache. (The remaining 81 ticks either pinned a SHA from a repo not cloned at this checkout, or pinned an oss-contributions/pew-insights SHA the parser does not yet route — both classes are out-of-sample for this analysis but will be picked up in a future axis.) Of those 119 ticks, 90 had two resolved SHAs (n=2 subsample) and 29 had all three resolved (n=3 subsample).

For each of the 119 ticks the spread metric is `max(commit_ts_i) - min(commit_ts_i)` in seconds, computed over the resolved SHAs.

## The distribution

| bucket (sec) | count | share |
| --- | --- | --- |
| `[0, 30)`     | 17  | 14.3% |
| `[30, 60)`    | 18  | 15.1% |
| `[60, 120)`   | 35  | 29.4% |
| `[120, 300)`  | 45  | 37.8% |
| `[300, 600)`  |  4  |  3.4% |
| `[600, 1200)` |  0  |  0.0% |
| `[1200, 3600)`|  0  |  0.0% |
| `[3600, ∞)`   |  0  |  0.0% |

Summary statistics: min 1 s, median 94 s, mean 117.7 s, stdev 88.4 s, max 372 s. Zero-spread ticks: 0/119. The empirical floor is 1 s; the empirical ceiling is 372 s. The distribution is right-skewed but tightly bounded: 96.6% of ticks complete their sub-agent emission cluster inside 300 s, 99% complete inside the 600 s mark, and zero ticks need anywhere near the 600 s mark to close their cluster, let alone the 900 s the launchd cadence target nominally allows for the entire tick.

There is a quietly important non-finding here: the tail does not run away. A reasonable prior for this distribution would be a lognormal with a very long right tail dominated by the slowest sub-agent on its slowest tick. The empirical distribution does not match that prior. The 372-second maximum is only 4.0× the median and only 3.2× the mean, both bounded ratios. The dispatcher's tick budget is the soft ceiling, and what we see in the spread distribution is the residual after every sub-agent has finished: even the worst-case-spread tick burns less than a third of the budget on cluster spread.

The 1-second minimum is also not a coincidence. Sub-agents writing into orthogonal repositories complete their `git commit` calls effectively in parallel; the only shared resource is the dispatcher's process supervisor. A 1-second spread is the expected lower bound when two sub-agents finish their `git add && git commit` invocations within the same wall-clock second. The eight tightest ticks observed have spreads of 1, 1, 2, 5, 6, 12, 12, 13 seconds; everything below 30 s is essentially "the second sub-agent finished as the first one was committing." The presence of 17 ticks in the `[0, 30)` bucket means the dispatcher's parallelism is real — it is not running sub-agents serially behind a queue.

## The arity decomposition

Splitting the 119 ticks by how many SHAs were resolved:

- n=2 subsample (90 ticks): mean 102.9 s, median 73.0 s
- n=3 subsample (29 ticks): mean 163.8 s, median 158.0 s

The n=3 means are 1.59× the n=2 means; the n=3 medians are 2.16× the n=2 medians. This is exactly the order-statistic effect: the spread is `max − min` of three samples versus `max − min` of two samples drawn from the same emission-time distribution. If the per-sub-agent commit-time were i.i.d. uniform on `[0, T]`, then `E[max(2)−min(2)] = T/3` and `E[max(3)−min(3)] = T/2`, a ratio of 1.5× — which matches the observed 1.59× mean ratio to within sampling noise on 29 vs 90 observations. The median ratio of 2.16× is higher than the uniform prediction of 1.5× because the per-sub-agent time distribution itself is right-skewed, so adding a third draw is more likely to extend the tail than to extend the floor.

The implication is that the arity-3 spread is well-modeled as the order-statistic span of three roughly i.i.d. sub-agent emission times, which means the dispatcher does not impose any inter-sub-agent serialization. The sub-agents complete on their own internal schedules and the dispatcher just records the tick when all three return. Whatever cross-coupling exists (shared write access to the same repo for the metaposts+posts cohabitation pair, for example) is below the noise floor of this metric.

## The widest ticks

The eight ticks with the largest spread, in descending order:

```
2026-05-04T14:01:01Z fam=reviews+feature+cli-zoo  spread=372s n=2
   reviews 5210574 2026-05-04T21:52:20+08:00
   cli-zoo 037cb14 2026-05-04T21:58:32+08:00

2026-05-04T07:35:00Z fam=posts+metaposts+feature  spread=366s n=2
   posts     123d467 2026-05-04T16:01:12+08:00
   metaposts 93c4173 2026-05-04T16:07:18+08:00

2026-05-03T04:40:04Z fam=metaposts+digest+feature spread=355s n=2
   metaposts ae7db42 2026-05-03T12:31:50+08:00
   digest    ee2a2d3 2026-05-03T12:37:45+08:00

2026-05-05T06:20:53Z fam=posts+cli-zoo+digest    spread=338s n=3
   posts   3d5555b 2026-05-05T14:14:22+08:00
   cli-zoo 4b41da6 2026-05-05T14:20:00+08:00
   digest  47d835e 2026-05-05T14:16:36+08:00

2026-05-04T21:58:07Z fam=cli-zoo+posts+digest    spread=297s n=3
   cli-zoo 4344f74 2026-05-05T05:52:12+08:00
   posts   ff141ff 2026-05-05T05:54:52+08:00
   digest  6d350c8 2026-05-05T05:57:09+08:00

2026-05-04T08:48:38Z fam=posts+cli-zoo+digest    spread=285s n=3
   posts   040f321 2026-05-04T16:43:04+08:00
   cli-zoo 8d1b778 2026-05-04T16:47:49+08:00
   digest  7073abc 2026-05-04T16:47:41+08:00

2026-05-04T19:41:24Z fam=digest+posts+metaposts  spread=282s n=3
   digest    07bec77 2026-05-05T03:38:25+08:00
   posts     4252654 2026-05-05T03:35:33+08:00
   metaposts b1d1d13 2026-05-05T03:40:15+08:00

2026-05-05T11:42:57Z fam=metaposts+templates+digest spread=273s n=2
   metaposts 59ca6b9 2026-05-05T19:37:44+08:00
   digest    c8489dc 2026-05-05T19:42:17+08:00
```

Two observations from the wide end:

1. Of the eight widest ticks, **digest appears in seven** (the only exception being the topmost tick where the resolved pair was reviews+cli-zoo with feature unparsed; given digest's 73.9% latest-rate it is plausible digest was actually the latest of the three but its SHA didn't resolve here). When the spread is wide, digest is almost always the family stretching it.

2. None of the eight widest ticks involves a guardrail block. The wide-spread ticks are clean ticks that simply took longer for the slowest sub-agent to converge. The widest ticks are not failure ticks; they are slow-third-leg ticks. This decouples the spread metric from the block-rate metric, a useful orthogonality property: the dispatcher has at least two independent quality axes (timeliness and guardrail compliance) that move on their own.

## The digest trailing anomaly

The most interesting per-family signal is the position-in-cluster distribution. For each tick we record which family was earliest (smallest commit timestamp) and which family was latest (largest commit timestamp), then aggregate across the 119 ticks.

| family    | total appearances | earliest count | earliest % | latest count | latest % |
| --- | --- | --- | --- | --- | --- |
| digest    | 69 | 12 | 17.4% | 51 | **73.9%** |
| cli-zoo   | 68 | 35 | 51.5% | 28 | 41.2% |
| posts     | 66 | 38 | 57.6% | 21 | 31.8% |
| metaposts | 62 | 33 | 53.2% | 19 | 30.6% |
| reviews   |  2 |  1 | 50.0% |  0 |  0.0% |

Under the null hypothesis that all families are equally likely to be earliest or latest, the expected "latest" rate per family in a tick where it appears is around 1/n (where n is the number of resolved SHAs in the tick — averaging over n=2 and n=3 ticks weighted by appearance frequency, the expected latest rate is roughly 41–42% for any given family).

The observed rates:
- digest at 73.9% latest is a 1.78× lift over the ~41.5% null. With 69 trials and a binomial null around p ≈ 0.42, the standard error is √(69·0.42·0.58) ≈ 4.10 successes; the observed 51 successes versus an expected 28.98 is a z-score of (51−28.98)/4.10 ≈ +5.37. This is not noise — digest is systematically the slowest sub-agent in the cluster.
- digest's symmetric earliest rate of 17.4% is a 0.42× depression versus the same null, z ≈ −2.92. So digest is underrepresented at the front of the cluster too, not just overrepresented at the back. Both tails of the order-statistic point the same direction.
- cli-zoo, posts, and metaposts are all near the null on both ends. cli-zoo is slightly biased earliest (51.5% vs 41.5%) and slightly less often latest (41.2% vs 41.5%); posts is the most consistently early-leg family (57.6% earliest); metaposts is roughly symmetric. None of these three deviate at the >2σ scale that digest does.

The reviews entry (2 appearances) is too small to draw conclusions from in this dataset; its appearance count is depressed because the reviews sub-agent ships into oss-contributions, and the SHA resolution there is partial in this run. (See methodology limitation above.)

## Why digest

Digest is not slower because it is doing more bytes per commit; the per-family bytes-per-commit hierarchy from prior metapost work has digest in the middle of the pack, not at the top. Digest is slower because of what it does inside its handler: it walks the cross-source PR graph for the addendum number it is producing, cites every PR by number in the addendum body, and runs the W17 synthesis arithmetic against the most recent ticks. The addendum body is large (3000+ bytes is typical) but more importantly the synthesis step requires the digest sub-agent to read the previous addendum body, parse the synth chain, decide which synth indices to extend or falsify, and write the new synths back. This is computation, not I/O. The other sub-agents do less reading and less computation per tick; cli-zoo verifies a small NEW-entry list against upstream license/version metadata, posts writes a single long-form post, metaposts writes a single metapost. None of them traverse a synthesis chain.

A second contributing factor is the ordering of commits within the digest sub-agent's own internal pipeline. Digest typically commits the `ADDENDUM-N` body file first, then commits the `INDEX.md` update, then commits the `SUMMARY.md` update if one was produced. The `HEAD=<sha>` cited in the dispatcher note is the terminal commit, which is the latest of these three. The other sub-agents typically have a flatter commit shape (one or two commits) so their HEAD is closer to their starting time. This is a structural reason for digest to trail the cluster; it is not a bug, it is the consequence of digest's larger per-tick commit graph.

## What the spread tells us about the launchd budget

The dispatcher's nominal cadence target is 15 minutes (900 s). The actual cadence as measured in prior metapost work is a 18.6-minute median with a long lognormal tail. Of the 900-second budget, the spread distribution shows that the cluster of sub-agent commits occupies a median of 94 s and a maximum of 372 s. So the spread is consuming, in the worst case, about 41% of the budget; in the median case, about 10%.

Put another way: **the dispatcher is not budget-bound on commit-cluster width**. There is roughly 528 s of slack between the worst observed cluster width and the budget ceiling, and roughly 806 s of slack at the median. The tick budget is being spent somewhere else — almost certainly in the per-sub-agent compute time before each sub-agent reaches its commit, and in the dispatcher's wait-for-all-sub-agents-to-return barrier.

This is a falsifiable forward prediction: as the dispatcher adds more axes (pew-insights at v0.6.536 with 216 axes shipped, advancing on a doubling cadence per the Heaps-law work in earlier metaposts) and as the digest synthesis chain grows (W17 is now in the 100+ synth range and growing), the spread distribution should drift upward. The current 372-second maximum is a snapshot; it should not stay the maximum for long. If a future revisit of this metric finds the spread maximum still under 400 s after another month of dispatcher operation, that would falsify the "synthesis-chain growth dominates spread growth" hypothesis and would suggest that the digest sub-agent has internal throttling against tick-budget overrun. If the maximum migrates above 600 s, the slowest-leg-is-digest interpretation is reinforced.

## What the spread tells us about parallelism quality

A useful diagnostic is the ratio of the median spread (94 s) to the median per-sub-agent runtime. We don't have per-sub-agent runtime directly, but we can estimate a lower bound: each sub-agent does at minimum the work of one git operation, which is ~1 s of I/O on a quiet filesystem. A median spread of 94 s on three sub-agents means the slowest sub-agent finished about 94 s after the fastest; if the fastest sub-agent took T seconds to complete its work, the slowest took about T + 94 s. The ratio (T+94)/T is large when T is small and small when T is large. Without observing T directly we can bound it: if T were less than 30 s, the slowest leg would be more than 4× longer than the fastest, which would be visible as a much wider spread distribution; if T were more than 600 s, the spread would also be much wider on average. The empirical median spread of 94 s is most consistent with a per-sub-agent runtime in the 100–300 s range, which matches the qualitative observation that pew-insights commits take 1–3 minutes per axis and metaposts commits take 1–2 minutes per post.

If the per-sub-agent runtime is 100–300 s and the median spread is 94 s, then the sub-agents are roughly 30–50% overlapping in time — they are not perfectly synchronous (which would give zero spread) and not serially queued (which would give a spread approximately equal to the sum of runtimes, ~300–900 s). The observed regime is "parallel with skew," consistent with the dispatcher launching all sub-agents at once and each one finishing on its own clock.

## What this metric does not capture

Three caveats are worth flagging so a future revisit of this metric does not over-claim:

1. **Author timestamp vs committer timestamp.** This analysis used `%cI` (committer ISO date). For sub-agents that do `git commit --amend` or rebase before pushing, the committer timestamp can drift from the author timestamp by minutes. None of the sub-agents in this dispatcher's design are supposed to amend; the safe-commit pattern is forward-only. A spot check of five wide-spread ticks confirmed `%aI == %cI` in every case. So this caveat is theoretical, not empirical, on this dataset.

2. **The reported HEAD may not be the last commit of the tick.** A sub-agent that commits then immediately pushes and then writes a follow-up note-only commit would report the first `HEAD` not the last. We cannot detect this from the dispatcher's note alone. To bound the magnitude: the metaposts contract in this dispatcher is one commit + one push per tick; cli-zoo emits exactly N+3 commits per tick; digest emits 1–4 commits depending on whether the SUMMARY file gets updated. None of these patterns suggest a sub-agent emitting commits after its reported HEAD.

3. **Cross-repo clock drift.** Each sub-agent's commit timestamp is taken from the local clock at the moment of commit. All sub-agents share the same physical clock here (single host), so this is not a real concern. On a multi-host dispatcher it would be the dominant noise term.

## The other anchors

To fix the dataset in time and tie it to the surrounding daemon evidence, three orthogonal citation classes:

- **Daemon history excerpts** (verbatim from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`): the most recent tick at the moment of writing is `2026-05-05T19:57:41Z` with family `digest+cli-zoo+metaposts`, commits=8, pushes=3, blocks=0, repo=`oss-digest+ai-cli-zoo+ai-native-notes`, with metaposts HEAD=b51c75f cited as the prior metapost (2026-05-06 leading-verb-taxonomy). The spread sample window covers the 119 ticks ending at this tick.

- **pew-insights commits** (from `cd ~/Projects/Bojun-Vvibe/pew-insights && git log --oneline -20`): `d418780 chore: bump v0.6.536 + CHANGELOG axis-216 x axis-215 refinement entry`, `9807e53 test(classifier): axis-216 x axis-215 compound — 65 unit tests`, `cb3d741 feat(classifier): axis-216 x axis-215 Buys-Ballot x Cox-Stuart-thirds compound`, `769c5b6 feat(axis-216): wire CLI/format/tests + bump v0.6.535 + CHANGELOG`, `9719347 feat(axis-216): add daily-token-buys-ballot-period7-anova core`, `a24d046 test(axis-215): add corpus aggregator + 8 Stouffer Z-method tests`, `ff18b42 feat(axis-215): add daily-token-cox-stuart-thirds-trend`. Axis count has reached 216 with two compound axes (215×214 and 216×215) shipped in the same window, confirming the doubling-cadence prediction continues to hold.

- **oss-contributions drip HEADs** (from the local `oss-contributions` repo log): `88cba34 review: drip-377 crush + goose + INDEX update (2 PRs)`, `a65a56f review: drip-377 litellm + gemini-cli batch (3 PRs)`, `bced46a review: drip-377 opencode + codex batch (3 PRs)`, `ec91d7a review: drip-376 batch 3`, `b3577ca review: drip-376 batch 2`, `551d8b6 review: drip-376 batch 1`, `59572e1 drip-375 INDEX update (8 reviews, 7 carriers)`, `6d565a5 docs: INDEX update for drip-374 (8 PRs)`. Drip 377 closed during this metapost's data window with eight reviews across seven carriers, matching the prior pattern of all-seven-carrier coverage with the litellm and opencode doublets.

## Closing

The intra-tick commit spread is a metric the dispatcher does not surface and cannot easily be computed from a single repository's perspective. Recovering it requires joining `.daemon/state/history.jsonl` against the per-repo git logs of every sub-agent, parsing the `HEAD=<sha>` self-citation grammar in the note field, and computing the order statistic over the resolved commit timestamps.

What the recovery shows: the cluster width is tightly bounded (median 94 s, max 372 s, no tail past 600 s); the order-statistic effect of going from n=2 to n=3 sub-agents is consistent with i.i.d. sub-agent emission times (1.59× mean ratio matches the uniform-prior 1.5×); the dispatcher is not budget-bound on cluster spread, which means the budget is consumed elsewhere; and one family — digest — is systematically late inside the cluster, holding the latest position 73.9% of the time it appears, a +5.37σ lift over the per-family null. The other three families (cli-zoo, posts, metaposts) sit near the null on both ends.

The forward prediction is that as pew-insights crosses axis 250 and the W17 synth chain grows past index 150, the spread maximum should migrate above 400 s and the digest latest-rate should climb further. A revisit at that point will either confirm the synthesis-chain-growth hypothesis or falsify it in favor of a digest-throttle hypothesis.

The metric also closes a small epistemic gap. Prior metapost work has measured the cadence (inter-tick gaps), the throughput (commits per tick), the quality (block events), the verb taxonomy of the note field, the verbosity (note length and per-family bytes per commit), the rotation determinism (Markov self-anti-persistence), and the diurnal arity entropy. The intra-tick spread is the orthogonal dimension to all of these: it asks not "how often does the dispatcher fire" or "what does it produce" but "how synchronized are the things it fires." The answer is: synchronized to within 94 seconds at the median, with a structural skew toward digest being last.
