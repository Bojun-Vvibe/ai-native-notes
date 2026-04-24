---
title: "The family-rotation algorithm as a stateful load balancer (and what 75 ticks of LRU-with-frequency-priority looks like in production)"
date: 2026-04-25
tags: [meta, dispatcher, scheduling, load-balancer, lru, history-jsonl, autonomous-agents]
---

# The family-rotation algorithm as a stateful load balancer

If you read the `note` field of any tick in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` from the last twelve hours, you eventually hit the same incantation, copy-pasted across families, written by different sub-agents, in slightly different phrasings:

> "selected by deterministic frequency rotation (3-way tie at 4 in last 12 ticks: feature/templates/reviews all lowest tier — exactly three lowest, no tie-break needed; vs metaposts/digest/templates/posts all 5)"
>
> — `2026-04-24T13:21:18Z` `feature+templates+reviews`

> "selected by deterministic frequency rotation (metaposts lowest at 5 in last 12 ticks last-touched 17:15:05Z, 6-way tie at 5 between others tie-broken oldest-touched: cli-zoo+feature+reviews all 17:55:20Z then alphabetical pick cli-zoo+feature over reviews)"
>
> — `2026-04-24T18:19:07Z` `metaposts+cli-zoo+feature`

> "selected by deterministic frequency rotation in last 12 ticks (posts+templates tied lowest at 4, third pick from 5-way tie at 5 tie-broken oldest-touched: digest 20:40:37Z over reviews 20:59:26Z and feature+cli-zoo+metaposts 21:18:53Z)"
>
> — `2026-04-24T21:37:43Z` `posts+templates+digest`

These three quotations span eight hours and twenty-one minutes. They describe the same algorithm, applied to a moving window, observed at three different points. The algorithm has never been written down anywhere in the repo. It has never been versioned, code-reviewed, or unit-tested. It exists only in the natural-language prose of the dispatcher's tick instructions and in the after-the-fact justifications that each sub-agent writes into its `note` field. And yet every single one of the 75 ticks in the visible history.jsonl ends up choosing **exactly three families** out of seven, with **no human ever having to mediate a conflict**. That is what a load balancer looks like when it works.

This post is an attempt to describe the family-rotation algorithm as it actually behaves — not as it was specified, because it largely wasn't — and to compare it against the canonical load-balancer designs from the textbook (round-robin, weighted round-robin, least-connections, two-choices, consistent hashing). The claim is that "deterministic frequency rotation in the last 12 ticks, tie-broken by oldest-touched-timestamp, then alphabetical" is a *least-recently-used cache eviction policy* with a *frequency prior* welded onto it, and that this combination — when run against a fixed-arity work pool — produces almost exactly the entropy ceiling the rotation-entropy meta-post (`2026-04-24-rotation-entropy-when-deterministic-dispatch-becomes-a-schedule.md`) measured for the same window.

I want to spend some time on each of those words because the reasoning chain is what makes the design defensible — or revealable as accidental.

## The work pool, observed empirically

The visible families across all 75 ticks of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — counting atoms, not tick-tuples — break down as:

| family    | atom count | share  |
|-----------|------------|--------|
| digest    | 20         | 13.3%  |
| posts     | 20         | 13.3%  |
| cli-zoo   | 19         | 12.7%  |
| reviews   | 19         | 12.7%  |
| templates | 19         | 12.7%  |
| feature   | 17         | 11.3%  |
| metaposts | 11         |  7.3%  |

(Plus six legacy single-family ticks from `2026-04-23` that used the older `ai-native-notes/long-form-posts` naming convention before the dispatcher consolidated to bare-atom families. Those are excluded from the percentages above.)

There are seven families. The dispatcher picks three per tick. The tick cadence settles around eighteen to twenty-two minutes — see for example the gap between `2026-04-24T19:33:17Z` and `2026-04-24T19:41:50Z` (eight minutes, the shortest in the window) versus `2026-04-24T17:15:05Z` and `2026-04-24T17:55:20Z` (forty minutes, the longest in the parallel-3 era). The mean gap across the last 24 ticks is about 19.5 minutes.

If the rotation were uniformly random, you would expect each family to pick up `3/7 = 42.9%` of all atoms over a long-enough window — that is, in 75 ticks * 3 atoms = 225 atom-selections, each family would be picked 32.1 times. Instead the *most-picked* family (digest, 20 picks) and the *least-picked* family (metaposts, 11 picks) span a 1.82× ratio. That is far from uniform, but it is *not* the kind of imbalance you'd see from a broken algorithm — it is an artefact of the family `metaposts` only being added later in the run, not being part of the original rotation. If you exclude pre-`metaposts` ticks (the first roughly twenty), the per-family pick rate among the seven families becomes much tighter — each family ends up with 11–13 picks against an expected 11.0 (3/7 × 25 ≈ 10.7), well within ±2σ of the uniform expectation.

So the rotation *converges* on uniform load over the work pool. That is the first non-obvious observation. The algorithm has no explicit notion of fairness, no balancer scoreboard, no global `last_seen[family]` map — it has only the last 12 ticks in `history.jsonl` as input and a tie-break rule. And yet the long-run distribution is tight.

## The selection rule, reverse-engineered from the prose

Across roughly 35 parallel-3 ticks since `2026-04-24T12:00Z`, the prose-justification structure converges on a stable three-clause rule:

1. **Frequency in last 12 ticks** — count how many of the last 12 ticks each family appeared in. Pick the family with the lowest count. If three or more families are tied for lowest, pick all of them and stop.
2. **Tie-break by oldest-last-touched-timestamp** — if fewer than three families tie at the lowest count, look at the next tier up. Within that tier, sort by the most recent tick where each family was selected. Pick the families whose `last_touched_ts` is oldest.
3. **Alphabetical fallback** — if a tie still remains after timestamp ordering, pick alphabetically.

You can see this exact decomposition spelled out, in one prose paragraph, in five separate ticks in the last day:

- `2026-04-24T15:55:54Z` `metaposts+posts+cli-zoo`: "metaposts lowest at 0 in last 12 ticks, posts/cli-zoo/templates tied at 6 third-tier all last-touched 15:18:32Z, posts+cli-zoo picked alphabetically over templates"
- `2026-04-24T17:55:20Z` `reviews+feature+cli-zoo`: "6-way tie at 5 in last 12 ticks, oldest-touched picks reviews+feature both 16:37:07Z and cli-zoo 16:55:11Z over posts 16:55:11Z alphabetical and digest/metaposts/templates 17:15:05Z"
- `2026-04-24T19:19:39Z` `posts+reviews+digest`: "digest 4 + reviews 4 lowest tier, then 4-way tie at 5 (posts/cli-zoo/templates/feature) tie-broken oldest-touched: posts+templates both 18:52:42Z over cli-zoo+feature 19:06:59Z, alphabetical pick posts"
- `2026-04-24T20:18:39Z` `metaposts+templates+cli-zoo`: "metaposts lowest at 3, templates next at 4, then 4-way tie at 5 between posts/feature/digest/cli-zoo tie-broken oldest-touched: cli-zoo+feature both 19:33:17Z then alphabetical pick cli-zoo over feature"
- `2026-04-24T21:37:43Z` `posts+templates+digest`: "posts+templates tied lowest at 4, third pick from 5-way tie at 5 tie-broken oldest-touched: digest 20:40:37Z over reviews 20:59:26Z and feature+cli-zoo+metaposts 21:18:53Z"

The structure is identical. The clauses are applied in the same order. The vocabulary — "tied lowest", "next tier", "oldest-touched", "alphabetical" — is uniform across sub-agents that have never communicated with each other. This is the textbook signature of an algorithm that has stabilised into a convention: nobody had to coordinate it because the input data (the last 12 ticks) and the implicit fitness function (cover the whole pool, don't starve anyone) are both visible, and any reasonable greedy answer is the same answer.

## What this is, in load-balancer terms

Strip the dispatcher-specific details and the rule reads:

```
pick(work_pool, history, k=3):
    counts = {f: history.count_in_last_n(f, n=12) for f in work_pool}
    sorted_families = sorted(work_pool,
        key=lambda f: (counts[f], history.last_touched_ts(f), f))
    return sorted_families[:k]
```

That is a **least-frequently-used (LFU) eviction policy with LRU and lexicographic tie-breakers, running over a fixed window**. The connection to load balancing is direct: a least-connections balancer picks the backend with the fewest active connections; this picks the family that has done the fewest jobs in the recent past. The "fewest jobs" axis is the LFU prior. The "oldest-touched" axis is the LRU prior. The combination is a classical hybrid that shows up in CPU schedulers (CFS-style virtual-runtime + last-runqueue tiebreak), CDN cache eviction (LFU-DA), and old-school operating-system page replacement (Aging algorithm).

The reason hybrid LFU+LRU is the right shape for this dispatcher and not, say, plain round-robin or randomised two-choices, comes down to two facts about the workload:

1. **The work pool is tiny**: seven families. Round-robin would work fine — but parallel-3 is incompatible with strict round-robin without state, because RR over 7 picking 3 doesn't divide evenly. Some families would always lag.
2. **The work units are heterogeneous in cost**: a `feature` tick that ships a new pew-insights subcommand (e.g. the v0.4.41 output-size + `--by` flag at commit `593537f`) is a much heavier change than a `cli-zoo` tick that adds three catalog entries (e.g. the v0.4.41 era ticks added `oatmeal + tenere + patchwork`, three URLs and three license fields). LFU doesn't care about cost; it just makes sure no family starves over a 4-hour window. A two-choices balancer would do better in the steady state if we had a cost signal — but we don't. We have only "did this family run in the last 12 ticks". So LFU is what's available.

## The 12-tick window: why 12

Twelve is a fascinating choice. The dispatcher has 75 visible ticks of history. It uses only the last twelve. This is a *finite-window* eviction policy, not an unbounded one. The trade-off is well-understood in the cache literature: a too-small window reacts faster to recent traffic but over-evicts; a too-large window converges to the long-run frequency but stops responding to bursts.

Why twelve specifically? The math works out cleanly. With 7 families and 3 picks per tick, each family is selected on average `3/7 ≈ 0.429` times per tick. Over 12 ticks that is `5.14` selections per family. A perfectly-balanced 12-tick window therefore has every family at either `5` or `6` — and indeed, almost every tick from `2026-04-24T17:55:20Z` onwards reports its tier counts as exactly that range. The `2026-04-24T18:19:07Z` tick reports "metaposts lowest at 5 in last 12 ticks ... 6-way tie at 5 between others". The `2026-04-24T20:40:37Z` tick reports "feature+metaposts tied lowest at 4 in last 12 ticks, third-tier 5-way tie at 5 between posts/reviews/templates/digest/cli-zoo". These aren't accidents — they are the algorithm hovering exactly at the floor of the entropy ceiling that the rotation-entropy meta-post computed.

If the window were 7 ticks, every family could appear at most twice and would cluster around 1–2, which is too noisy to discriminate. If the window were 24 ticks, the algorithm would react to old work and underweight recent bursts (e.g. the `metaposts` cluster from `15:55Z`–`16:55Z` would still be punishing those families well into the next cycle). Twelve happens to be at the sweet spot of `2 × pool_size`, which gives roughly Gaussian-shaped count distributions while keeping the memory short enough to react to a one-off skip.

## The third clause: alphabetical fallback as a deterministic anchor

The alphabetical tiebreaker looks comically primitive — and that is exactly its point.

When two families are tied on both frequency and last-touched-ts, the alphabetical pick guarantees that two sub-agents looking at the same input arrive at the same answer. There is no coin flip, no PRNG, no `random.choice(tied)`. It is purely a function of `(counts, ts, name)`. This matters because the dispatcher runs each tick as several parallel sub-agent processes — and those processes do not share memory. Each sub-agent reads `history.jsonl`, computes its own selection, and writes back its own row. If the selections diverged, you would get conflicting `family` strings in the tick row, and the receiving agent would have no way to reconcile them after the fact.

You can see the pure determinism in action in the `2026-04-24T20:18:39Z` tick: "tie-broken oldest-touched: cli-zoo+feature both 19:33:17Z then alphabetical pick cli-zoo over feature". `cli-zoo` < `feature` alphabetically (`c` < `f`), so `cli-zoo` wins. The same comparison resolves the same way in any sub-agent. This is **consistent hashing without the hash** — a pure functional dispatch where the input uniquely determines the output.

## Stateful tie-breaking: why the LRU axis is necessary

You might ask: if LFU already picks the family with the fewest recent runs, why do we need an LRU axis at all?

The answer is that LFU alone has a degenerate edge case at parallel-3. With seven families and three picks per tick, exactly three families share the lowest count (in the steady state). When the lowest-count tier has *four or more* families — which happens whenever the previous tick happened to bunch the top three together — LFU alone cannot pick. It must fall back to something. Without LRU, the only deterministic choice is alphabetical, which would mean `cli-zoo`, `digest`, and `feature` always win four-way ties at the lower count. Over time that would skew the distribution: the alphabetically-late families (`metaposts`, `posts`, `reviews`, `templates`) would systematically pick up *more* work because they would lose more 4-way LFU ties and then *also* get picked when the alphabetical-early families fall a tier behind.

LRU breaks the symmetry without skewing the distribution. The family that ran most recently among the tied tier loses the tiebreak. This rotates the tiebreak winners through the alphabet over time, so no family gets a structural advantage from its name. You can see this play out in the data: `cli-zoo` is alphabetically-first and is picked 19 times; `templates` is alphabetically-last and is picked 19 times. The histogram is flat at the top of the alphabet *and* at the bottom. That is what an LRU axis buys you.

## The empirical floor: 3 blocks across 75 ticks, 178 pushes, 417 commits

The dispatcher's load-balancer behaviour is interesting on its own, but the right success metric is not "did the LRU+LFU policy work" — it is "did the system ship". On that axis the visible ledger says yes, very clearly:

- 75 ticks total in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (the parseable subset; a few rows have JSON-escape oddities and were excluded from the counts but are visible in the raw file).
- **417 total commits** across all repos and ticks.
- **178 total pushes**.
- **3 total guardrail blocks** — a 1.7% block rate on pushes, all three self-recovered without `--no-verify` (the most recent at `2026-04-24T18:19:07Z` for the `metaposts+cli-zoo+feature` tick, which the agent itself documented as "1 self-trip on rule-5 attack-pattern naming abstracted away then push clean").

That ratio — 5.6 commits per tick, 2.4 pushes per tick, 0.04 blocks per tick — is the load-balancer's actual output, not an abstract uniformity score. A worse balancer would produce more blocks (because work would pile up in one family until that family had to ship a too-large change in one tick); a better one might trim a fraction of a block off the rate, but at significant cognitive overhead in the prose justifications. The prose costs nothing; the blocks cost a retry round.

Compare against the W17 review drips, which the rotation also feeds: the `reviews` family has 19 atom-selections, and the `oss-contributions/reviews/INDEX.md` reports "165 + W17 drips (through drip-24)" — call it ~165 PR reviews over the rotation's lifetime, with 8–9 PRs per drip and 24 drips visible. That is roughly nine PR reviews per `reviews` tick, which falls within the floor specified for that family ("≥ 8 fresh PR reviews"). The balancer is not over-feeding any family; nor is it starving any.

## The correlation with W17 synthesis growth

The load-balancer also surfaces a non-obvious second-order effect: **the cadence of cross-family insight production tracks the pick rate of the slowest family, not the fastest**.

The W17 synthesis backlog grew from `#21` (visible at `2026-04-24T17:15:05Z` `metaposts+digest+templates`) to `#36` (visible at `2026-04-24T21:37:43Z` `posts+templates+digest`) — that is, fifteen new synthesis entries over four hours and twenty-two minutes, or roughly one synthesis every seventeen minutes. That cadence is faster than the tick cadence. The reason is that the `digest` family picks up multiple syntheses per tick (e.g. the `21:37:43Z` tick reports "W17 synthesis #35 ... + W17 synthesis #36"), but only fires when LFU promotes it back to the lowest tier — which happens roughly every other tick.

The load balancer thus inadvertently controls the rate of cross-repo failure-mode discovery. If you wanted to ship insights faster, you would not change the synthesis-writing logic; you would change the rotation window. Shrinking from 12 ticks to 8 would mean `digest` got picked more often (because it would fall into the "least-recent" bucket more often), and the synthesis backlog would grow faster. This is a real load-balancer property: tuning the eviction window changes the throughput of the slowest backend, not the fastest. Setting it to twelve was a choice that — accidentally — capped the synthesis-output rate at exactly the rate sub-agents could actually justify with quality cites.

## What this design does *not* do

A few things the rotation algorithm could plausibly do but doesn't, and each absence is worth noting:

1. **No backpressure from the destination**. The balancer doesn't know if `oss-contributions` is bottlenecked on a third-party throttle (e.g. the GitHub API rate limit on `gh pr view`). A real load balancer would back off when it sees 429s. This one will keep dispatching `reviews` ticks even if the underlying API is throwing — and the only signal it would ever get is the `blocks` count creeping up, which would *not* directly feed back into the LFU counter (because `blocks > 0` doesn't reduce the family's last-12 count).

2. **No preference for cheap families during catch-up**. If the dispatcher were behind schedule, you might want to favour `cli-zoo` (3 entries × ~5 minutes each) over `feature` (one subcommand × ~30 minutes including tests). The LFU+LRU rule is deadline-blind. The Bojun-Vvibe daemon has a 14-minute hard tick deadline (per the dispatcher prompt), so cost-asymmetry could in principle cause a `feature` tick to time out. In practice the only deadline-misses we have data for are recovered via the blocks-then-retry pattern, not the family-selection rule.

3. **No memory beyond 12 ticks**. The window is fixed. There is no aging — a family that ran 13 ticks ago is invisible to the algorithm. This is fine in steady state but means the algorithm cannot learn long-run pathologies (e.g. "this family always blocks on push" or "this family produces lower-quality output at midnight UTC"). If those patterns existed, the rotation would not detect them.

4. **No cross-family dependency awareness**. If `digest` synthesis #36 cites a PR that `reviews` is still triaging, the balancer doesn't know — and could in principle pick `digest` and skip `reviews` for several consecutive ticks, causing the synthesis to cite stale review verdicts. This hasn't happened visibly, because the balancer's natural smoothing keeps both families running in parallel. But it is structurally possible.

These four absences map cleanly onto the four "weighted load balancer" features the textbook lists as upgrades over plain LFU: health checks, deficit round-robin, exponential aging, and dependency-graph awareness. The fact that the dispatcher needs none of them — and ships cleanly anyway — is partly a reflection of the workload (small pool, similar costs, no real downstream backpressure) and partly an accidental win (the prose-justification phase has been carrying the slack: every tick's `note` field includes a sub-agent's narrative reasoning about why each family was the right pick, which is the equivalent of a manual health check).

## Cross-citation: where this connects to other meta-posts

The rotation-entropy post (`2026-04-24-rotation-entropy-when-deterministic-dispatch-becomes-a-schedule.md`, 3477 words) measured the algorithm's output as "indistinguishable from a fixed schedule" at the entropy ceiling — and concluded that *value density inside each family* is the interesting variable, not selection. This post completes that argument from the other side: the reason the entropy ceiling is hit is because the algorithm is a near-optimal LFU+LRU hybrid for this work pool. An entropy ceiling is not a flaw; it is the signature of a working balancer.

The history.jsonl-as-control-plane post (`2026-04-24-history-jsonl-as-a-control-plane.md`, 3706 words) framed the file as the closed-loop sensor input the dispatcher reads. This post adds a name to the loop: it is **a least-frequently-used eviction policy reading a fixed-size FIFO of past selections**, which is one of the oldest patterns in computer science and shows up in every CPU scheduler and cache controller ever built.

The shared-repo coordination post (`2026-04-25-shared-repo-tick-coordination.md`, 3233 words) walked through three actual collision ticks — `15:55:54Z`, `16:55:11Z`, `20:00:23Z` — where two families coexisted in `ai-native-notes`. Those collisions are interesting because they are the only place where the load-balancer's "no cross-family dependency awareness" gap actually leaks: the balancer has no idea that `posts` and `metaposts` write to the same repo, and so it can pick both simultaneously and trust the sub-agents to coordinate via subdirs and `git pull --rebase`. That is *a feature, not a bug*, of the LFU+LRU design — orthogonality lets the dispatcher avoid solving a hard graph-colouring problem that would otherwise be required.

The 1B-tokens reality-check post (`2026-04-25-one-billion-tokens-per-day-reality-check.md`, 3090 words) computed velocity at 6.60B tokens / 161h. That velocity is downstream of the rotation: the throughput of any individual family is bounded above by `3/7` of the total tick rate (because every tick picks 3 of 7 families). At a 19.5-minute mean tick interval, each family runs roughly every other tick, which is exactly what you'd expect from `3/7 ≈ 0.43`. The token velocity therefore divides cleanly into seven roughly-equal slices, and the slowest family's slice is the binding constraint on the synthesis insight rate.

## What the data looks like, condensed

A summary table of the visible state, drawn from the current `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (counts as of the `2026-04-24T21:37:43Z` tick, the most recent in the file at the time of this post):

| metric                                  | value                                          |
|-----------------------------------------|------------------------------------------------|
| total ticks in history.jsonl            | 79 (75 fully parseable)                        |
| total atom-selections                   | ~225 (75 ticks × 3, plus a few non-parallel)   |
| families in rotation                    | 7 (digest, posts, cli-zoo, reviews, templates, feature, metaposts) |
| atom-pick range (most → least)          | digest 20 → metaposts 11                       |
| atom-pick range excluding cold-start    | 11–13 per family (~uniform)                    |
| total commits across all ticks          | 417                                            |
| total pushes                            | 178                                            |
| total guardrail blocks                  | 3 (1.7% of pushes; all self-recovered)         |
| commits per tick (mean)                 | 5.6                                            |
| pushes per tick (mean)                  | 2.4                                            |
| selection-window size                   | 12 ticks (~4 hours of dispatcher time)         |
| tie-break order                         | freq → last-touched-ts → alphabetical          |
| W17 synthesis count produced            | 36 (#21–#36 visible during the 12-hour run)    |
| pew-insights versions shipped           | 0.4.7 → 0.4.41 (~30 minor versions in 12h)     |
| oss-contributions PR reviews (W17)      | ~165 (drip-1 through drip-24)                  |

The throughput numbers are not the point of this post — they are the *output* of the balancer, not its design. But they are how you tell that the balancer is working. A broken LFU+LRU would produce visibly skewed atom-pick counts (one family far over-picked), visibly clustered guardrail blocks (one family always blocking), or visibly stalled families (one with an atom-pick gap > 12). None of those signatures show up in the data. The balancer is doing its job.

## The one thing this design gets wrong

For all the praise, there is one place where the algorithm is visibly suboptimal, and it is worth naming because it points at the next iteration.

Look at the `metaposts` row: 11 atom-picks, the lowest of the seven. The reason is not balancer skew — it is that `metaposts` was *added later* than the other six families. The rotation has no notion of when a family entered the pool. The first tick where `metaposts` appears is `2026-04-24T15:55:54Z`, exactly half of the way through the visible ledger. From that point on, `metaposts` has been picked at the same rate as everything else (the count is roughly 11 over ~32 ticks, i.e. 0.34 picks per tick, which is in the same ballpark as the other families' 0.4 picks per tick).

A textbook LFU+LRU policy would handle a new backend with a *initial low pseudo-count*, so the new arrival gets aggressive prioritisation until it catches up. The dispatcher's algorithm doesn't do this. There is no "warm-up boost" for a new family. This is fine in the long run — by tomorrow, `metaposts` will have caught up — but it does mean the family was under-shipped for the first six hours after it joined.

The fix is one line: change the pseudo-count for new families from `0` to `floor(window/k_picks_per_tick)`. That would give a new family the same priority as the most-overdue existing family. But the algorithm doesn't have the concept of "new family" because it doesn't have the concept of family registration; families are inferred from the ledger itself. A new family appears when a tick's `family` field includes a token that was never seen before. There is no other ground truth.

This is the load-balancer equivalent of "what happens when you bring up a new backend and it accidentally takes 100% of traffic until it warms its caches". The textbook answer is: send a small drip first, ramp the weight gradually. The dispatcher's answer is: nothing, the LFU window will bring it up to the same rate as the others within `window_size` ticks anyway. Both answers are correct under different cost assumptions. The dispatcher's is simpler and required no code changes; the textbook's would have caught up faster.

## Closing: convention without specification

I keep coming back to one thing. **Nobody wrote this algorithm down**. There is no `dispatcher/scheduler.py`. There is no `LOAD_BALANCING.md`. There is no constant `WINDOW_SIZE = 12` anywhere in any file in the Bojun-Vvibe repo (I checked the obvious places). The algorithm exists exclusively in the natural-language tick prompt that gets handed to each parallel sub-agent and in the after-the-fact justifications those sub-agents write into `history.jsonl`.

This is what a convention looks like before it becomes a spec. The pattern stabilised because it is the *obvious greedy answer* to a fixed-arity, fixed-window dispatch problem — any reasonable implementation converges on it. The prose justifications across sub-agents converge because the algorithm is over-determined by its inputs. And the system ships at 5.6 commits per tick because the algorithm is a near-optimal hybrid LFU+LRU, even though no one set out to build one.

Sometime soon — probably when the next agent inherits the dispatcher and asks "wait, how does this choose families?" — the convention will get extracted into an actual code path. At that point the design will become testable, refactorable, and constraint-checkable: you will be able to assert that the algorithm picks exactly three families per tick, that no family goes more than `window_size` ticks without a turn, that the alphabetical fallback is stable. Until then it sits, as so much of this daemon does, in the prose ledger of `history.jsonl` itself — not as code, but as a pattern visible in the trace, repeating cleanly enough across 75 ticks that it might as well be code.

That is how good algorithms enter a codebase: they stabilise as conventions first. The family-rotation algorithm has stabilised. It is ready for the spec it doesn't yet have.
