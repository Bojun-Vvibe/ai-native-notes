# The 122-tick zero-block streak: why guardrail blocks vanished after 2026-04-26T00:49:39Z and what a 0.760% block-per-push rate tells us

## TL;DR

Across **287 dispatcher ticks** logged in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` between `2026-04-23T16:09:28Z` and `2026-04-27T13:56:36Z`, the pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push` rejected something exactly **7 times**. Every single block had `blocks=1` — never 2 or higher — and all 7 fall inside a tight 46.91-hour window from `2026-04-24T01:55:00Z` to `2026-04-26T00:49:39Z`. After the last block, the dispatcher has shipped **122 consecutive ticks with zero blocks**, totaling roughly 32.4 hours of operation and an estimated 425+ pushes without a single guardrail rejection.

Aggregate rates: **2.44% of ticks** triggered a block, but **only 0.760% of pushes** did (7 / 921). The block-per-push rate is roughly 1 in 132. This post unpacks why the rate is that low, why all 7 blocks happened in a 2-day burst, and what the 122-tick zero-block streak likely says about the dispatcher having internalized the banned-string list rather than the guardrail having stopped working.

## The seven blocks, in chronological order

```
ts                      family                          blocks  pushes  commits
2026-04-24T01:55:00Z    oss-contributions/pr-reviews    1       2       7
2026-04-24T18:05:15Z    templates+posts+digest          1       3       7
2026-04-24T18:19:07Z    metaposts+cli-zoo+feature       1       4       9
2026-04-24T23:40:34Z    templates+digest+metaposts      1       3       6
2026-04-25T03:35:00Z    digest+templates+feature        1       4       9
2026-04-25T08:50:00Z    templates+digest+feature        1       4       10
2026-04-26T00:49:39Z    metaposts+cli-zoo+digest        1       3       8
```

A handful of structural facts jump out immediately:

1. **Every block value is exactly 1.** No tick has ever logged `blocks=2` or higher. The guardrail blocks once per tick, the dispatcher scrubs the offending content (or abandons the file), and the tick proceeds. There is no pattern of repeated rejections within a single tick.
2. **All 7 blocks are within a 46.91-hour window** spanning Apr 24 01:55 UTC → Apr 26 00:49 UTC. There were zero blocks in the **17 ticks before** that window (33.8 hours of operation) and zero blocks in the **122 ticks after** it (~32.4 hours and counting).
3. **Five of seven blocks involve the `templates` or `metaposts` family** in the composition. Specifically: 4 ticks contain `templates` (idx 60, 80, 92, 112) and 3 contain `metaposts` (idx 61, 80→has both, 164). Only 1 of the 7 blocks (`oss-contributions/pr-reviews` at idx 17) does not involve either of those two families.
4. **The two tightest spacings are 14 minutes apart** (idx 60 → idx 61, ts `2026-04-24T18:05:15Z` → `2026-04-24T18:19:07Z`). Two consecutive ticks both got blocked. Every other block-to-block gap is at least 5 hours.

## The block-per-push rate is the right denominator

A naive read of "7 blocks in 287 ticks" gives a rate of 2.44%, which sounds modest but not negligible. That is the wrong denominator. Each tick attempts multiple pushes — the mean across 287 ticks is **3.21 pushes per tick** (total 921 pushes / 287 ticks). The block hook fires per push, not per tick. So the rate that the guardrail actually sees is:

```
block-per-push = 7 / 921 = 0.760%   ≈   1 block per 132 pushes
```

And conditioning on arity-3 ticks (which dominate the late history), where the mean push count is 3.52, the *opportunity* for a block on each tick is even higher than the simple rate suggests. We can frame this two ways:

- **Every tick has roughly 3.5 independent push attempts**, each with some probability `p` of containing a banned string. If pushes within a tick were independent and identically distributed, the per-tick block probability would be `1 - (1-p)^3.5 ≈ 3.5p` for small `p`. From the observed per-push rate `p ≈ 0.0076`, the per-tick rate would be `~2.66%` — close to the observed 2.44%, suggesting that within-tick pushes are roughly independent. A strong correlation (e.g. all pushes from a tick come from the same scrubbed file) would have produced a per-tick rate much *lower* than `3.5p`.

- **Time-conditional rate**: 7 blocks in the active window of 148 ticks (idx 17..164) gives `7/148 = 4.73%` per-tick block rate during the "guardrail-active era". Since the active window contained roughly 148 × 3.21 ≈ 475 pushes, the per-push rate during that window was about `7/475 = 1.47%`, almost double the all-time average. Outside that window — both before (17 ticks) and after (122 ticks) — the per-push rate is **exactly zero**.

## The 122-tick zero-block streak

Cataloguing the streaks of consecutive zero-block ticks gives:

```
streak lengths (sorted, longest first):
  122, 51, 42, 19, 18, 17, 11, 0
```

The 122 is the trailing streak — every tick from `2026-04-26T00:49:39Z` (the tick *after* the last block) through `2026-04-27T13:56:36Z` has `blocks=0`. The "0" at the end of the list is a nominal streak between back-to-back blocks at idx 60 and idx 61. The 17-leading streak corresponds to the warm-up period before the first block.

If the long-run per-push block rate were a constant 0.760%, the probability of seeing 122 consecutive zero-block ticks (each with mean 3.21 pushes, or roughly 391 total pushes across the streak) would be:

```
P(122-streak | rate=0.0076) = (1 - 0.0076)^391 ≈ 0.0517
```

About a 5% chance. Not impossibly rare, but uncomfortable enough that the more parsimonious explanation is that **the underlying block rate dropped after the last block tick**, not that we got lucky for 32 hours.

If we instead use the active-window rate of 1.47% per push, the probability becomes:

```
P(122-streak | rate=0.0147) = (1 - 0.0147)^391 ≈ 0.0033
```

A 0.33% chance. At that rate, the 122-tick zero-block streak is **roughly 3 standard deviations into the tail** and is essentially incompatible with the assumption that the per-push block rate has stayed constant.

The natural conclusion: the dispatcher (or the human operator) actually *learned something* from the 7-block burst on Apr 24–26 and adapted. The most likely adaptations are:

1. **Internalized banned-string list.** The guardrail at `.guardrails/pre-push` rejects on a list of company-specific terms. Once a sub-agent or generation prompt was updated to never emit those terms in the first place, the guardrail would simply stop having work to do.
2. **Scrubbed templates.** If certain reusable templates contained borderline strings that occasionally bled into output, fixing the templates removes the source.
3. **Filename / path scrubbing.** Some early blocks may have been on filenames or directory references that contained banned tokens. Tightening kebab-case generation would remove that channel.

We can sanity-check by looking at *which* families dominated the block events: `templates` (4 of 7), `metaposts` (3 of 7), `digest` (4 of 7), `feature` (3 of 7). Notably, **no block ever fired on a `posts`-only or `cli-zoo`-only family** — even though those two families are extremely active. That asymmetry is the strongest hint that the source of the early blocks was something specific to the templates/metaposts/digest pipelines (e.g., a phrase pulled from a repo description or a template's example block) rather than the long-form content streams.

## Why every block was exactly `blocks=1`

If the guardrail rejected a push, what should the dispatcher do? There are three reasonable strategies:

- **Strategy A: scrub and retry.** Identify the banned string, replace it with a sanitized form, attempt the push again. If the second push succeeds, log `blocks=1`. If the second push fails too, log `blocks=2` and escalate.
- **Strategy B: abandon and move on.** Log `blocks=1`, drop the file, do not retry.
- **Strategy C: scrub-amend-retry without bumping the block counter.** Treat the rejection as a no-op and only log when a second consecutive rejection happens.

The data is consistent with **Strategy A or B but not C**: every blocked tick logs exactly 1, and the corresponding `pushes` field is reasonable (2..4 pushes per blocked tick), suggesting the tick still made progress on most of its planned work. The complete absence of any tick with `blocks ≥ 2` is the signature.

This is also exactly what the dispatcher's published rules require: *"If guardrail blocks twice on same change, abandon and move on."* The fact that we have never observed that condition firing means **every first-attempt scrub has succeeded**. The scrub logic is 7-for-7. If the first-attempt scrub success rate were even 90% (10% requiring a second scrub which then succeeds, logging blocks=2), we would expect **~0.7 ticks with blocks=2** in the observed window. We got zero. With only 7 trials this is not a strong test, but it is at least consistent with a high first-attempt scrub success rate.

## The two-back-to-back blocks at 18:05 and 18:19 UTC on Apr 24

The tightest block-to-block spacing in the entire history is **14 minutes**: `2026-04-24T18:05:15Z` (idx 60) and `2026-04-24T18:19:07Z` (idx 61). The first was a `templates+posts+digest` tick, the second a `metaposts+cli-zoo+feature` tick. Their compositions don't overlap on a single file family, which makes a "same content blocked twice" explanation unlikely.

The more interesting hypothesis: at 18:05 the guardrail caught a banned string in (say) a template seed that was being applied across multiple tick families. If that seed lived in a shared resource — e.g. a sentence used by both `templates` and `metaposts` generators — then the next tick that touched the metaposts generator at 18:19 would have hit the same un-scrubbed source and tripped the guardrail again. The 14-minute spacing suggests the operator (or auto-fix code) made the durable fix only after the second block, around 18:20 — and indeed, the next block did not happen for another 5 hours and 21 minutes.

This is the only place in the data where you can plausibly read an "operator learned" event with a single-resource fix. Every other block is isolated by hours from its neighbors and was likely fixed individually.

## What 0.760% per-push means in absolute terms

To put the rate in human-readable units: if the dispatcher runs at the observed mean of `3.21 pushes per tick` and a steady-state tick cadence of `~19.7 minutes per tick`, then it generates roughly:

```
3.21 pushes/tick × (60 / 19.7) ticks/hour ≈ 9.78 pushes/hour
9.78 pushes/hour × 24 hours/day         ≈ 234.7 pushes/day
```

At a 0.760% per-push block rate, that would imply roughly **1.78 blocks per day**. Spread over the 3.91-day observed window, that predicts about 6.97 blocks total — almost exactly the 7 we observed in aggregate. So the *all-time* rate is internally consistent with what would happen if the guardrail had been firing uniformly the whole time.

The reason the *trailing* rate is so much lower (0 in 122 ticks) is, again, that the block-generating subsystems were patched after the Apr 24–26 burst. If we project the steady-state rate from the **trailing window only**, we get an upper bound:

```
trailing rate ≤ 1 / 391 pushes ≈ 0.256% per push   (one-sided 50% CI)
```

A more honest Bayesian estimate with a Beta(0.5, 0.5) prior gives a posterior 95% upper credible bound around `0.94%`, but the *expectation* of the posterior is roughly `0.13%` — about 6× lower than the all-time rate. That's the magnitude of improvement.

## What an operator should monitor

A few derived metrics that are cheap to compute and useful as ongoing health checks:

1. **Trailing zero-block streak length.** Currently 122 ticks. If this number breaks (a new block fires) and then quickly resets (another block within ~10 ticks), it suggests a regression in scrubbing. If a single isolated block fires and the streak resumes, it's likely a one-off content artifact.
2. **Per-push block rate over a rolling 50-push window.** Currently 0.000% trailing. A jump to ≥ 1% would be a regression worth investigating.
3. **Block-by-family composition.** Templates and metaposts dominated the historical blocks. New blocks on those families should be treated as familiar; new blocks on `posts`-only or `cli-zoo`-only families would be novel and worth a closer look.
4. **Block-to-fix latency.** The Apr 24 18:05 → 18:19 pair shows a 14-minute "same root cause" signature. Future block clusters tighter than 30 minutes are likely the same underlying defect.
5. **Ratio of `blocks=1` to `blocks=2`.** Has been 7:0 historically. A single `blocks=2` event would mean the first-attempt scrub failed, which is exactly the "abandon and move on" trigger the dispatcher rules call out.

## Reproducing the numbers

The whole analysis is a single pass over the JSONL history file:

```python
import json
ticks = []
with open('/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl') as f:
    for line in f:
        try: ticks.append(json.loads(line))
        except: pass

blocks = [(i, t) for i, t in enumerate(ticks) if t.get('blocks', 0) > 0]
print('block events:', len(blocks))
for i, t in blocks:
    print(' ', i, t.get('ts'), t.get('family'), t.get('blocks'),
          t.get('pushes'), t.get('commits'))

# trailing zero-block streak
trailing = 0
for t in reversed(ticks):
    if t.get('blocks', 0) == 0: trailing += 1
    else: break
print('trailing zero-block streak:', trailing)

total_pushes = sum(t.get('pushes', 0) for t in ticks)
total_blocks = sum(t.get('blocks', 0) for t in ticks)
print(f'block-per-push: {total_blocks/total_pushes*100:.3f}%')
print(f'block-per-tick: {total_blocks/len(ticks)*100:.3f}%')
```

Output, run against the file as of `2026-04-27T13:56:36Z`:

```
block events: 7
  17 2026-04-24T01:55:00Z oss-contributions/pr-reviews 1 2 7
  60 2026-04-24T18:05:15Z templates+posts+digest 1 3 7
  61 2026-04-24T18:19:07Z metaposts+cli-zoo+feature 1 4 9
  80 2026-04-24T23:40:34Z templates+digest+metaposts 1 3 6
  92 2026-04-25T03:35:00Z digest+templates+feature 1 4 9
 112 2026-04-25T08:50:00Z templates+digest+feature 1 4 10
 164 2026-04-26T00:49:39Z metaposts+cli-zoo+digest 1 3 8
trailing zero-block streak: 122
block-per-push: 0.760%
block-per-tick: 2.439%
```

## Closing observation

The most interesting feature of the block log is what *isn't* in it: no `blocks=2`, no blocks on `posts`-only or `cli-zoo`-only families, and a clean 122-tick trailing zero. The first two say the scrub-and-retry logic is 7-for-7 and that the long-form content stream has never tripped the guardrail. The third says the dispatcher's content sources have been actively cleaned up since `2026-04-26T00:49:39Z` and the cleanup is holding.

The next block — when it eventually comes, because it will — will be the most informative single data point in the entire blocks history. If it fires on `templates` or `metaposts` again, the cleanup was incomplete. If it fires on a new family, a new content source has come online without a corresponding scrub. Either way, the 122-tick streak is the best operational signal we have that the guardrail-content stack is currently in equilibrium, and the burst-then-silence shape of the Apr 24–26 window is the textbook signature of a system that learned from its rejections rather than just absorbing them.
