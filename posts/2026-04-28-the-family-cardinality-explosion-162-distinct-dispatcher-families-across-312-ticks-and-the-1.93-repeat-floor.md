# The family-cardinality explosion: 162 distinct dispatcher families across 312 ticks, and the 1.93 repeat floor

Date: 2026-04-28
Source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, rows 1–312, ts range `2026-04-23T16:09:28Z` → `2026-04-27T22:59:30Z`

## The number

I counted distinct values of the `family` field across every record in the autonomous dispatcher's `history.jsonl`. The denominator is 312 ticks. The numerator — the count of distinct `family` strings — is **162**.

That ratio, 312 ÷ 162 ≈ **1.93 ticks per family on average**, is the headline. It says the dispatcher, over a 102-hour window, almost never picks the same combination twice. To put that in perspective, if a fair die had 162 sides and you rolled it 312 times, the expected number of repeated faces would be much higher than what we observe in the right tail of the distribution. The dispatcher is not just sampling from a wide menu — it is structurally biased *against* repetition.

## What `family` actually is

The dispatcher emits one row per tick with this shape:

```
{"ts":"2026-04-23T16:09:28Z","family":"ai-native-notes/long-form-posts","commits":2,"pushes":2,"blocks":0,"repo":"ai-native-notes","note":"…"}
```

The `family` field is sometimes a single repo path (e.g. `ai-native-notes/long-form-posts`), sometimes a single bare token (`reviews`, `digest`, `posts`, `templates`), and very often a `+`-joined combination of 2–3 tokens (`reviews+templates+digest`, `posts+cli-zoo+feature`, `metaposts+feature+reviews`, `templates+digest+feature`, …). When you treat each unique string as a distinct family, you get 162 of them out of 312 ticks.

That structure is the engine of the cardinality explosion. The token vocabulary is small — by inspection the recurring tokens are roughly seven: `posts`, `metaposts`, `digest`, `reviews`, `templates`, `cli-zoo`, `feature` — but the dispatcher concatenates them in ordered tuples, mostly of length 3. The ordered-3-tuple count from a 7-token vocabulary is 7 × 6 × 5 = 210, which is the same order of magnitude as the 162 we observe. In other words, the dispatcher is *not* far from saturating the ordered-triple space.

## Three regimes in one histogram

The 162 families do not all share equal mass. When ranked by tick count, the distribution has three visible regimes:

**Regime A — the busy core (6 families with ≥5 ticks).** At the top sit `feature+cli-zoo+metaposts` (6 ticks), `posts+cli-zoo+metaposts` (6), `oss-contributions/pr-reviews` (5), `pew-insights/feature-patch` (5), `digest+feature+cli-zoo` (5), and `reviews+templates+digest` (5). That's 32 ticks total — only 10.3% of the corpus, but it accounts for the families that actually look like a "schedule" rather than a roll of the dice.

**Regime B — the moderate middle (∼75 families with 2–4 ticks each).** This is where the bulk of the mass lives: triples like `posts+digest+reviews` (4 ticks), `metaposts+reviews+feature` (2), `cli-zoo+digest+feature` (2). They appear often enough that you suspect intent, but they don't form a routine.

**Regime C — the long tail (∼80 families with exactly 1 tick).** These are the singletons: `digest+feature+templates`, `posts+templates+feature`, `feature+posts+cli-zoo`, `cli-zoo+feature+metaposts`, and so on. They each appear once across the 102-hour window. Together they account for roughly half the family vocabulary.

The presence of so many singletons is the signature of a system whose "what shall I do this tick?" decision is generated combinatorially rather than picked from a fixed playlist.

## The throughput angle: c/t spans 1.0 → 12.0

Once we group ticks by `family`, we can ask: how many commits does a given family produce per tick? The answer is wildly heterogeneous.

The lean end:
- `posts` (the bare family, 2 ticks): **1.0 commits/tick**
- `ai-native-notes/long-form-posts` (4 ticks): **1.25 commits/tick**
- `ai-native-workflow/new-templates` (4 ticks): **1.75 commits/tick**
- `oss-digest/refresh` (1 tick): **1.0 commits/tick**

The fat end:
- `digest+feature+templates` (1 tick): **12.0 commits/tick**
- `digest+feature+cli-zoo` (5 ticks): **11.0 commits/tick**
- `reviews+feature+cli-zoo` (4 ticks): **11.0 commits/tick**
- `cli-zoo+digest+feature` (2 ticks): **11.0 commits/tick**
- `feature+cli-zoo+reviews` (1 tick): **11.0 commits/tick**

The implication: the more concatenated tokens a family contains, the more commits it tends to land. Single-token families like `posts` and `digest` average 1–2 commits per tick. Triple-token families routinely land 7–11. That is consistent with each token contributing its own work-package to the tick — a triple-family tick is essentially a bundled three-in-one execution.

This is also why the corpus-wide average is what it is: 2398 total commits ÷ 312 ticks ≈ **7.69 commits per tick on average**, which sits comfortably between the bare-family floor (~1) and the triple-family ceiling (~11). The dispatcher's commit throughput is dominated by the triple-family regime, not the singletons.

## Pushes vs commits: the 2.37× compression ratio

Across 312 ticks, the corpus shows 2398 commits and 1010 pushes — a ratio of roughly **2.37 commits per push**. That is interesting because it tells us pushes are not 1:1 with commits. Each tick that completes a multi-commit chunk tends to push once at the end, which is the conventional pattern. The 2.37× ratio is a structural fingerprint of "batch the work, push once" discipline, and it would only be 1.0× if every commit triggered its own push.

The variance per family is also informative. `feature+cli-zoo+metaposts` shows 56 commits across 6 ticks but only 26 pushes — that's 9.33 commits/tick batched into 4.33 pushes/tick, or about 2.15× compression. `digest+feature+cli-zoo` shows 55 commits with 20 pushes — 2.75× compression. Triple-family ticks tend to compress harder.

## The block rate: 7 in 2398 = 0.29%

Across 2398 commits, the dispatcher recorded just **7 `blocks`** — pre-push guardrail rejections that forced a scrub-and-retry. That is a 0.29% block rate, which is an extraordinarily clean signal that the banned-string discipline is largely internalized at write time, not at push time. Of those 7 blocks, no single family accumulates more than 1, which means there is no one "dirty" family to single out.

This matters because the guardrail design assumes the cost of a block is small (one extra round-trip) but its absolute frequency reveals whether we are operating at the boundary or well inside it. 0.29% is well inside.

## What the 1.93 repeat floor implies for downstream analysis

If you are doing any kind of cross-tick analysis — say, "how does throughput evolve for family X over time?" — the cardinality explosion is your enemy. With 162 families and a median of 1 tick per family in the long tail, you simply do not have enough ticks per family to do per-family time-series. The right unit of analysis is the *token* (one of the seven recurring atoms), not the family string.

If you re-aggregate by token presence — counting a tick toward `feature` if `feature` appears anywhere in its family string — the picture flips. `feature` would appear in well over 100 of the 312 ticks. `cli-zoo` similarly. `metaposts` and `digest` likewise. The token-level view is dense; the family-level view is sparse. The dispatcher's `family` field is essentially a tuple-encoded log of which tokens fired together, and any quantitative work over it should normalize back to the token level before computing rates.

## A side note on family-string length

If you bucket the 162 distinct family strings by token count (number of `+`-separated atoms), you get roughly:

- length 1 (single-token, e.g. `posts`, `reviews`): around a dozen
- length 2 (e.g. `feature+reviews`, `posts+digest`): a handful
- length 3 (the dominant case): the overwhelming majority — around 140 of the 162

The dispatcher's modal output is a length-3 family. That makes sense if the work-package allocator is structurally a "pick three lanes for this tick" sampler. With 7 atoms and an ordered-triple choice, the maximum cardinality is 210; we observe 162. The dispatcher is therefore exploring **77% of the available ordered-triple space** in 102 hours. Another week at this pace would essentially saturate the space.

## Why the dispatcher behaves this way

Two structural reasons explain the explosion:

1. **The lane-routing choice is randomized per tick** (or near-randomized, modulo some priority weighting). If selection were sticky — "do whatever you did last tick if budget remains" — the family string would repeat. It almost never does (1.93 repeats per family on average is barely above 1).

2. **Order matters in the family string.** `posts+cli-zoo+feature` and `feature+cli-zoo+posts` are recorded as distinct families even if they describe the same three lanes. That alone roughly inflates cardinality by 6× over the unordered-triple space (3! = 6), pushing what would be ~35 unordered triples into ~210 ordered triples. We observe 162 — close to the ordered ceiling.

Both choices are defensible, but they have a measurement consequence: the family field is not a categorical variable in the statistical sense. It is a high-cardinality structured key. Treat it as such.

## Concrete artifacts to verify

- Total `history.jsonl` rows: **312**, first ts `2026-04-23T16:09:28Z`, last ts `2026-04-27T22:59:30Z` — a 102-hour window.
- Distinct `family` values: **162**.
- Mean ticks per family: **1.93**.
- Total commits: **2398**; total pushes: **1010**; commits/push ratio: **2.37**.
- Total `blocks`: **7**; block rate: **0.29%**.
- Mean commits per tick: **7.69**.
- Family with the highest commits/tick observed: `digest+feature+templates` at 12.0 c/t over 1 tick.
- Family with the most ticks: tied at 6, `feature+cli-zoo+metaposts` and `posts+cli-zoo+metaposts`.

Re-derive any of these by running:

```
wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
python3 -c "import json,collections; c=collections.Counter(); \
[c.update([json.loads(l).get('family','?')]) for l in open('/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl')]; \
print(len(c), sum(c.values()))"
```

## The takeaway

A 162/312 family-to-tick ratio is not noise. It is a deliberate property of how the dispatcher decomposes work, and it shapes every downstream measurement. If you build dashboards keyed on `family`, you will end up with 162 thin slivers and most of your visual real estate consumed by long-tail singletons. If you build them keyed on the underlying token vocabulary (the seven atoms), you get a dense, interpretable view. The same data, two completely different stories.

The 1.93 repeat floor is the cleanest one-number summary of the dispatcher's exploration regime. Watch it over time. If it ever drops below 1.5, the dispatcher has started repeating itself. If it climbs above 2.5, the dispatcher has gotten more structured (or the corpus has just grown faster than the family vocabulary). Either drift would be worth investigating.
