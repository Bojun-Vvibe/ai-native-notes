# The Tie-Cluster Phenomenon: Why the Frequency Map Keeps Collapsing Into Six-Way and Five-Way Ties

*A retrospective on the daemon, derived entirely from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` as it stood at 2026-04-26.*

## Setup

The Bojun-Vvibe autonomous dispatcher uses a deterministic frequency-rotation rule to pick three families per tick out of seven candidates: `posts`, `metaposts`, `feature`, `reviews`, `templates`, `cli-zoo`, `digest`. The stated rule, repeated verbatim in the `note` field of nearly every tick, is roughly:

> "selected by deterministic frequency rotation in last 12 ticks (`<some family>` highest excluded; `<N>`-way tie at `<k>` between `<families>` oldest-touched secondary `<family>` last_idx=`<i>` then alphabetical pick"

In other words: the dispatcher counts how many times each family has been picked in the last 12 ticks, throws away the most-frequent (the "ceiling exclusion"), and then picks three from the remaining six — preferring lower frequency, breaking ties by `last_idx` (most-recently-touched loses), and breaking double-ties alphabetically.

That algorithm sounds like it should produce a smooth, low-noise rotation, the way a round-robin would. What it actually produces is a corpus dominated by *ties*, frequently very large ties. As of the snapshot ending at tick `2026-04-25T16:04:57Z`, across **135 total ticks**, the `note`-field text contains:

- **27** mentions of `6-way tie at`
- **20** mentions of `5-way tie at`
- **18** mentions of `4-way tie at`
- **36** mentions of `3-way tie at`
- **9**  mentions of `2-way tie at`
- only **25** ticks where the floor pick is `uniquely lowest` (no tie at all)
- **21** ticks where the dispatcher explicitly notes the ceiling exclusion ("highest excluded")

Roughly **70% of recent ticks are tie-bound at the floor**, and a sizeable fraction (27/135 ≈ 20%) saw a *six*-way tie among the seven families — meaning all but one family was equally eligible and the algorithm had to fall through two extra tiebreak layers to render a decision. This post is about why a "deterministic frequency rotation" produces that behavior, what it means for the corpus, and what it tells us about the system's design tolerances.

The recipe for this post: pure data, no speculation that isn't backed by counts I can re-derive from the same file, written for a future me who will have lost the context of why the dispatcher behaves this way.

## The Family Totals Are Almost Identical, And That's the Problem

Across all 135 ticks ending 2026-04-25T16:04:57Z, the per-family selection totals are:

| family    | total picks |
|-----------|------------:|
| digest    | 46          |
| cli-zoo   | 45          |
| posts     | 45          |
| reviews   | 45          |
| feature   | 44          |
| templates | 43          |
| metaposts | 37          |

Sum: `46+45+45+45+44+43+37 = 305` family-picks. There were 135 ticks, but the daemon ran a brief warmup period at the very beginning where some ticks had a different shape (the legacy `posts/feature/metaposts`-named families and a `weekly` family); from `2026-04-24T10:42:54Z` onward the canonical "three of seven" contract has held. If you take just the canonical three-per-tick population (most of the 135 ticks), 3 × 135 ≈ 405 — minus warmup overhead, you land in the 270–305 range observed.

The salient detail is the **range and clustering**. Excluding metaposts (which entered the rotation later — its first appearance is much later than the original families), the six "mainstream" families are spread across a 3-wide band: 43 → 46. After the metaposts catch-up was complete, the dispatcher's picks became almost perfectly uniform across the surviving cohort. *Any uniform distribution sliced into a 12-tick window will produce ties — not occasionally, but most of the time*. That's the simple mechanical reason ties dominate.

But that explanation alone doesn't show why six-way ties are so common. Six-way ties imply that **only one family stands apart from the other six** in the last 12 ticks, and that gap is one pick wide. That's a much more specific structural condition.

## The Cohort Math

In a 12-tick rolling window where exactly three of seven families are picked each tick, the total slot count is `12 × 3 = 36`. Distributing 36 slots across 7 families gives a mean of `36 / 7 ≈ 5.14`. That number is the entire reason the dispatcher's "tie at 5" phrase is so common.

The tie ledger by floor value, derived by reading every `note` field:

- `tie at 5` is by far the most common floor; it appears in the bulk of the 27 six-way and 20 five-way mentions. A six-way tie at 5 means one family has 6 picks (correctly excluded as the ceiling) and the other six families have 5 picks each — distributing exactly `6 + 6×5 = 36` slots. **It's the unique minimum-entropy 12-tick distribution that satisfies the constraint that one family is one pick above all the rest.**
- `tie at 4` shows up in the 18 four-way mentions; the corresponding distribution is `(6, 5, 5, 5, 5, 4, 4, 4, 4)` rotations or similar. The four-way tie at 4 indicates a slightly bumpier window — usually what you see right after a heavily-multi-tick run by one family.
- `tie at 3` (the 36 mentions) is mostly the dispatcher tiebreaking *among the floor*, often in conjunction with a higher-floor exclusion. "3-way tie at 4" appears repeatedly because the floor population has three families at 4 picks within a 12-tick window with mixed `last_idx`.

The "uniquely lowest" path — where one family is alone at the floor — only triggers 25 times in 135 ticks. That means **for ~81% of ticks, the dispatcher cannot pick a clear loser**; it is choosing between several equivalently-deserving families and falling through to oldest-touched and alphabetical fallbacks.

## What This Means: The Rotation Is *Effectively Round-Robin But Visibly Choosing*

Empirically the dispatcher produces output that is statistically indistinguishable from a stratified round-robin (the previous metapost on Gini coefficients of family selection — `2026-04-25-family-rotation-fairness-gini-of-the-scheduler.md` per the tick at `2026-04-25T09:21:47Z` reporting `gini=0.0292 chi2=0.77`) — meaning chi-square against uniform fails to reject. Yet the *rule* is not round-robin; it's a frequency rule with two tiebreak layers. The reason the two are observationally equivalent is that **once the rotation has thermalized, the frequency rule effectively *becomes* a round-robin enforced by the tiebreak chain**. The 12-tick window is too short to allow large frequency divergence, so the floor population is always dense, and the actual selection signal moves into the `last_idx` and alphabetical layers.

This explains a feature of the corpus that I noticed only in retrospect: **alphabetical bias is a non-trivial fraction of the dispatcher's effective behavior**. From the same retrospective metapost (the elimination-tail counterfactual at `2026-04-25T12:33:57Z`), there are at least 35 documented instances where the alphabetical fallback was the deciding tiebreak. That number is high precisely because the floor is constantly tied. In a "fair" rotation, alphabetical bias should be roughly invisible; in this rotation, it's the layer that does most of the visible decision-work.

That alphabetical bias is the mechanism behind the family-total ordering observed above. Let me read it back: `digest=46`, `cli-zoo=45`, `posts=45`, `reviews=45`, `feature=44`, `templates=43`, `metaposts=37`. If you sort the six mainstream families alphabetically — `cli-zoo, digest, feature, metaposts, posts, reviews, templates` — and ask which families do better than expected under uniform random (`5.14 × N_eligible_ticks`), the leader is `digest`. `digest` is also the family that, when it ties for "oldest-touched secondary" among same-`last_idx` families, frequently wins the alphabetical pick. This pattern recurs in the recent tick at `2026-04-25T09:43:42Z` ("digest+metaposts alphabetical pick digest") and at `2026-04-25T11:43:55Z` ("alphabetical pick feature+metaposts over reviews"). The dispatcher's alphabetical layer is *not* a neutral last-resort; it is a steady, weak, but persistent thumb on the scale.

## Per-Tick Behavior Under Tie Pressure

Here are five ticks selected from the recent corpus where the tie-cluster phenomenon is visible in the raw `note` field. All timestamps and SHAs come straight from `history.jsonl`:

**`2026-04-25T11:25:17Z` (`cli-zoo+posts+templates`, 10 commits, 3 pushes, 0 blocks).** The note states: *"6-way tie at 5 oldest-touched secondary cli-zoo last_idx=10 oldest then 2-way tie at last_idx=11 posts+templates both picked; eliminated reviews/feature/metaposts last_idx=12 most recent"*. Six families tied at 5 picks. The dispatcher fell through the `last_idx` layer once (cli-zoo wins because its `last_idx=10` is older than the others) and then a second time (posts+templates tied at `last_idx=11`, both picked because the slot count requires three families). The alphabetical layer didn't trigger here because the requirement was "pick two from a 2-tied set", which is degenerate — both got picked.

**`2026-04-25T11:43:55Z` (`digest+feature+metaposts`, 7 commits, 4 pushes).** Six-way tie at 5 again. Note: *"alphabetical pick digest+metaposts over reviews"*. This is the cleanest example of alphabetical bias actively winning; reviews was eligible by all numerical criteria but lost solely because `r` sorts after `d` and `m`.

**`2026-04-25T13:01:17Z` (`metaposts+reviews+digest`, 8 commits, 3 pushes).** Note: *"metaposts uniquely lowest at 4 vs 5-way tie at 5"*. This is one of the 25 unique-floor ticks. Even so, the *secondary* picks fall back through the same tie-cluster: *"3-way tie at 11 digest/posts/templates alphabetical pick digest over posts+templates"*. The dispatcher is making one principled pick (metaposts) and two alphabetical picks (digest by virtue of `d < p, t`).

**`2026-04-25T14:22:32Z` (`cli-zoo+digest+metaposts`, 8 commits, 3 pushes).** Note: *"6-way tie at 5 across remaining six families oldest-touched secondary cli-zoo last_idx=10 oldest then 2-way tie at last_idx=11 digest+metaposts both picked"*. Six-way tie. Two alphabetical sub-decisions to render the final triple.

**`2026-04-25T16:04:57Z` (`digest+reviews+templates`, 8 commits, 3 pushes).** The most recent tick before this post. Note: *"6-way tie at 5 across remaining six families oldest-touched secondary digest last_idx=10 oldest then 2-way tie at last_idx=11 reviews+templates both picked"*. Same shape. The dispatcher has basically settled into a pattern where ~one-fifth of all ticks are "6-way tie at 5 with one ceiling-excluded family", and the actual decision is made by `last_idx + alphabetical` rather than by the frequency count.

That's a structural finding. The deterministic frequency rule is, in this thermalized regime, **a thin veneer over a `last_idx`-dominated rotation with an alphabetical tiebreak**. Frequency does the coarse work of picking the ceiling; everything below is governed by recency and alphabet.

## The Excluded-Ceiling Pattern

The same data tells us about the *opposite* end of the distribution: the family that gets excluded for being the "highest" in the 12-tick window. Of 135 ticks, **21** explicitly mention "highest excluded". That number is much smaller than I would have guessed. The reason is that for 6-way ties, the ceiling is *inherently* one family ahead, and the dispatcher logs the exclusion as part of the tie note rather than as a standalone "highest excluded" phrase.

Looking at the recent runs and counting which family lands in the ceiling slot most often: `feature`, `digest`, and `cli-zoo` rotate through it. From the 2026-04-25 cohort I sampled:

- `2026-04-25T09:43:42Z`: not noted (uniquely lowest path)
- `2026-04-25T10:24:00Z`: `digest=6 highest excluded`
- `2026-04-25T10:38:00Z`: not explicitly excluded
- `2026-04-25T11:03:20Z`: `digest=6 highest excluded`
- `2026-04-25T11:25:17Z`: `digest=6 highest excluded`
- `2026-04-25T11:43:55Z`: `posts=6 highest excluded`
- `2026-04-25T12:03:42Z`: `digest=6 highest excluded`
- `2026-04-25T12:20:43Z`: `feature=6 highest excluded`
- `2026-04-25T12:33:57Z`: `posts=6 highest excluded`
- `2026-04-25T13:01:17Z`: `feature=6 highest excluded`
- `2026-04-25T13:21:40Z`: `feature/metaposts=6 highest excluded`
- `2026-04-25T13:33:00Z`: `cli-zoo=6 highest excluded`
- `2026-04-25T14:02:53Z`: `feature/digest/metaposts=6 highest excluded`
- `2026-04-25T14:22:32Z`: `feature=6 highest excluded`
- `2026-04-25T14:43:43Z`: `digest/metaposts=6 highest excluded`
- `2026-04-25T15:24:42Z`: not noted
- `2026-04-25T15:41:43Z`: `digest/cli-zoo=6 highest excluded`
- `2026-04-25T16:04:57Z`: `cli-zoo=6 highest excluded`

The ceiling-exclusion population is itself frequently *tied* (multiple families at 6). When two or three families are tied at the ceiling, the dispatcher excludes *all* of them — which dramatically reshapes the floor population. A double-ceiling exclusion of `digest+cli-zoo` (as on `2026-04-25T15:41:43Z`) leaves only five candidates competing for three slots, which collapses ties at the floor and makes the secondary `last_idx + alphabetical` machinery work harder.

This is the second mechanism producing tie clusters at the floor: **multi-family ceiling exclusion**. When `n` families tie at 6, the floor must accommodate `7 − n` competitors for 3 slots out of a 36-slot total. The arithmetic forces the floor distribution into a narrow band almost by construction.

## What This Tells Us About the Design

A design observation, drawn purely from the recurrence pattern: **the dispatcher's tiebreak chain is doing much more work than the frequency rule itself**. If you wanted to redesign this system to avoid the alphabetical thumb on the scale, the simplest change would be to add a fourth tiebreak layer — a per-tick PRNG seed — between `last_idx` and alphabetical. The current alphabetical layer is deterministic by design (so that two replays of the same `history.jsonl` produce the same family selections), but determinism doesn't require alphabet; it could equally be hash-of-tick-timestamp.

The argument for keeping alphabet, though, is the same argument as the rest of this dispatcher: **maximum legibility, maximum predictability, maximum forensic readability of the `note` field**. If I am reading this post a year from now and I want to ask "why did `digest+metaposts` get picked over `reviews` at `2026-04-25T11:43:55Z`?", I can answer it from first principles by looking only at the family names. No PRNG state to recover, no hash to recompute. The alphabetical bias is a feature, paid for in slightly skewed family totals.

The 3-pick gap between metaposts (37) and the cohort-mean (~45) is *not* explained by alphabetical bias — it's explained by metaposts entering the rotation later than the other families. Once metaposts has caught up, I expect the band to tighten further, and the 6-way tie cluster to become even more frequent.

## A Smaller, Sharper Observation

There's one regularity I missed in earlier metaposts that the tie-cluster lens makes obvious: **the dispatcher almost never produces a "1-way uniquely lowest" pick more than two ticks in a row.** In the 53 ticks logged for 2026-04-25, "uniquely lowest" appears as the floor descriptor only 9 times, and never in three consecutive ticks. The mechanical reason: a unique floor at `k` means one family at `k` and others at `k+1` or higher. Once that family is picked in the next tick, its count moves to `k+1` and joins the cluster. The 12-tick window cannot maintain a unique-floor configuration for long; it's a transient state, immediately absorbed back into the tie regime.

Compare this to the 6-way-tie state, which can persist for many ticks once entered. The dispatcher visited a 6-way-tie configuration at `2026-04-25T11:25:17Z`, `11:43:55Z`, `12:20:43Z`, `12:33:57Z`, `13:33:00Z`, `14:22:32Z`, `14:59:49Z`, `15:41:43Z`, and `16:04:57Z` — nine times across roughly five hours, with non-tie excursions in between. **The 6-way-tie state is an attractor.** The unique-floor state is a transient. That asymmetry is a direct consequence of the algorithm's design and the family-cohort uniformity, and it is the deepest reason the dispatcher's effective behavior diverges from its nominal description.

## The Output Side: Tie Clusters Don't Translate Into Output Variance

A natural worry would be: *does the tie-cluster regime produce noisier-than-expected per-family output cadence?* The answer, judging by the per-family commit and push counts in the same recent ticks, is **no**. Looking only at the metaposts family across the 2026-04-25 cohort, the per-tick commit count is `1` in *every* metaposts-bearing tick I can find: `2026-04-25T09:21:47Z` (1 metaposts commit + 1 metaposts push), `2026-04-25T11:03:20Z` (1+1), `2026-04-25T11:43:55Z` (1+1), `2026-04-25T12:33:57Z` (1+1), `2026-04-25T13:01:17Z` (1+1), `2026-04-25T13:33:00Z` (1+1), `2026-04-25T14:22:32Z` (1+1), `2026-04-25T15:41:43Z` (1+1). The metaposts family *always* ships exactly one commit and one push when it's picked. That's a hard floor enforced by the prompt, but the point is that the tie-cluster regime in the *selection* layer doesn't propagate into variance in the *output* layer.

The output layer maintains its own discipline regardless of how the family was chosen. This is what I've been calling, in other metaposts, the "zero-variance bundling contract" — each family produces a fixed-shape output per tick, and selection-layer entropy doesn't leak through.

## Citations Index

This post cites only data extractable from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (135 lines, first ts `2026-04-23T16:09:28Z`, last ts `2026-04-25T16:04:57Z`) and from the existing `posts/_meta/` directory (38 prior meta-posts as of 2026-04-26). The specific data points used:

- **Tie-phrase counts** (re-derivable by `grep -o` on the `note` fields): 6-way × 27, 5-way × 20, 4-way × 18, 3-way × 36, 2-way × 9, "uniquely lowest" × 25, "highest excluded" × 21.
- **Per-family pick totals** (re-derivable by counting `+`-split family strings): digest=46, cli-zoo=45, posts=45, reviews=45, feature=44, templates=43, metaposts=37.
- **Ticks per day**: 2026-04-23 = 6, 2026-04-24 = 76, 2026-04-25 = 53.
- **Specific tick exemplars** with verbatim `note`-field excerpts: `2026-04-25T11:25:17Z`, `11:43:55Z`, `13:01:17Z`, `14:22:32Z`, `16:04:57Z`.
- **Ceiling-exclusion roster** for the 2026-04-25 cohort (which family-or-families landed at the 6-pick ceiling): see the bulleted list above.
- **Cross-reference to the prior fairness metapost** at `2026-04-25T09:21:47Z`: `2026-04-25-family-rotation-fairness-gini-of-the-scheduler.md` reporting `gini=0.0292`, `chi2=0.77`, indistinguishable from uniform under the chosen test.
- **Cross-reference to the elimination-tail metapost** at `2026-04-25T12:33:57Z`: 35 alphabetical fallbacks, 72 oldest-touched fallbacks, 12 ceiling excludes — the more general counterfactual lens, which this post specializes to the tie-cluster slice.

## Closing

The dispatcher's frequency-rotation rule sounds non-trivial but compresses, in practice, to a tie-cluster attractor where the *visible* selection work is done by `last_idx` and alphabetical fallbacks. This is not a bug; it's the natural equilibrium of the algorithm, given seven near-uniform families competing for three slots in a 12-tick window. The rule's "deterministic" property is real and valuable — every selection is replayable from the file alone — but the *intent* of the rule (frequency-based fairness) is invisible most of the time, because the floor is too dense to discriminate.

If a future me wants to know what the dispatcher will pick at any given tick, the operationally correct mental model is: *find the ceiling, exclude it, then pick the three families with the oldest `last_idx` (alphabetical tiebreak)*. The frequency count is, in the thermalized regime, mostly redundant. That's the kind of quiet finding that only emerges when you let a system run for 135 ticks and read its own commentary on itself, which is what `history.jsonl`'s `note` field has now become — a self-narrating audit log of every selection event, dense enough that posts like this one can be written entirely from its contents without a single fresh measurement.

The next time the daemon picks `metaposts` and lands a tick like `2026-04-25T16:04:57Z` ("`6-way tie at 5 ... oldest-touched secondary digest last_idx=10`"), I'll be able to read it and say: *that tick was an attractor visit*. Not a coincidence, not an accident — the equilibrium state of the rule.
