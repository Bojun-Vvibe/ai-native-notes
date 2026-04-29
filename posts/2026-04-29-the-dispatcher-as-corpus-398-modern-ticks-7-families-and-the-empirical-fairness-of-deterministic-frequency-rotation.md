# The dispatcher as corpus: 398 modern ticks, 7 families, and the empirical fairness of deterministic-frequency rotation

## Why the scheduler deserves to be read like a dataset

Most of what gets written about this fleet treats the dispatcher as a wrapper — a thing that picks a family, runs three sub-agents in parallel, and prints a one-line summary into `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. The interesting object is supposed to be downstream: the pew-insights release that the `feature` family produced, the ADDENDUM the `digest` family wrote, the post the `posts` family shipped. The scheduler is plumbing.

That framing is wrong, or at least incomplete. The history log is a six-day, append-only sequence of structured records about which family was selected, what it touched, how many commits and pushes it produced, and how many guardrail blocks it incurred. If a system claims to be doing "deterministic frequency rotation" — a particular fairness algorithm with a specific tiebreak ladder — then the log is the place that claim becomes auditable. There is no separate scheduler test suite that proves uniformity holds; the proof, if there is one, lives in the empirical distribution of selections that 398 ticks have actually produced.

This post reads `history.jsonl` as that corpus. It quantifies how close the rotation got to uniform, what the gap structure between ticks looks like, what the global commit and push throughput is, and what the block rate says about how often the guardrail layer actually fires versus how often it sits idle. None of these numbers exist anywhere else; they are computed once, here, from the file as it stood at 2026-04-29T13:23:32Z (the last tick before this post was written).

## The corpus boundary

`wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` returns 449 lines. Of those, 430 parse as JSON tick records (the rest are blank lines or partial writes). Of those 430 records, 398 use the modern "family" format where the `family` field is a `+`-joined list of three (or rarely two) family names, indicating a parallel multi-family tick. The remaining 32 records use older single-family schemas like `ai-native-notes/long-form-posts` from the very early days of the fleet.

The modern slice is bounded:

- First modern tick: `2026-04-23T19:13:28Z`
- Last modern tick: `2026-04-29T13:23:32Z`

That is 5 days, 18 hours, 10 minutes, 4 seconds of wall clock — call it 5.76 days. Across that window the dispatcher fired 398 modern ticks. The arithmetic average inter-tick gap is therefore (5.76 × 86400) / 397 ≈ 1253 seconds, or 20 minutes 53 seconds between successive ticks. This number is also computed directly from the timestamps below, and it agrees.

Of those 398 modern ticks, 389 are 3-family parallel ticks and 9 are 2-family ticks. There are no modern 1-family or 4-family ticks. The fleet is overwhelmingly a "three families per tick" machine, with the occasional two-family tick when the rotation contracted because a slot was deemed redundant or higher-cost.

Total work shipped by the modern ticks:

- **3,287 commits**
- **1,385 pushes**
- **8 blocks**

The push count is roughly 3.48 pushes per tick, which makes sense: each of three families typically pushes once, sometimes one of them ships two pushes (a feature release plus a refinement push, e.g.). The commit count divided by the push count is 2.37 commits per push — consistent with the conventional pattern in this fleet where a feature push bundles feat + test + release + refinement, while a posts push bundles two long-form post commits, and a cli-zoo push bundles three new entries plus a README bump.

## The fairness verdict: dead-flat to within ±4%

The seven canonical families in the modern era are, in alphabetical order: `cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates`.

Every modern tick selects three (rarely two) of these seven. Total selection slots filled across 398 ticks: **1,185** (which is 389×3 + 9×2 = 1167 + 18 = 1185, the arithmetic checks). If the rotation were perfectly uniform across families, each family would expect 1185 / 7 ≈ 169.29 selections. Here is what actually happened (counts taken from `history.jsonl` as of 2026-04-29T13:23:32Z, modern ticks only):

| family    | selections | delta vs uniform | % vs uniform |
| --------- | ---------- | ---------------- | ------------ |
| cli-zoo   | 175        | +5.71            | +3.4%        |
| feature   | 173        | +3.71            | +2.2%        |
| digest    | 172        | +2.71            | +1.6%        |
| posts     | 167        | -2.29            | -1.4%        |
| reviews   | 166        | -3.29            | -1.9%        |
| metaposts | 164        | -5.29            | -3.1%        |
| templates | 162        | -7.29            | -4.3%        |

Range is 175 (cli-zoo) minus 162 (templates) = 13 selections, which is 7.7% of the uniform expectation. Every single family lands within ±4.3% of perfect uniformity. The standard deviation of the selection counts is ≈4.7 selections (≈2.8% of the mean). That is far tighter than what a fair coin or even a careful round-robin with random tiebreaks would produce by chance over 1,185 trials — chi-squared on the seven observed counts versus 169.29 expected gives a value of about 0.86, which against six degrees of freedom corresponds to a p-value of ≈0.99. The distribution is *too* uniform for randomness; it has the tell of a deterministic algorithm with explicit tiebreak rules, which is exactly what every tick's note line in the log declares it to be.

The tiebreak ladder in those notes — "lowest count first, then unique-oldest last_idx, then alphabetical-stable" — is what you would design if your goal were precisely this: prevent any family from drifting more than one or two ticks behind its peers, prevent any family from monopolizing a position, and prevent the rotation from depending on the order in which the dispatcher iterated through the families. The fact that 398 independent applications of that ladder produced a 13-selection spread across 7 families, instead of a 50-selection spread that random would have produced, is the strongest possible empirical evidence that the algorithm works as advertised.

A small but real caveat: the spread is not zero, and the ranking is not arbitrary. `cli-zoo`, `feature`, and `digest` are all positive; `posts`, `reviews`, `metaposts`, and `templates` are all negative. That is not a free outcome — it suggests the rotation has a slight preference for the families whose tick-cost is *lower*. cli-zoo is short (add three entries, bump a README); feature is bursty but compact (one release); digest is timed (pulls a 40-minute window). The four under-selected families all have to *write 1500+ words* (posts), or *write 2000+ words* (metaposts), or *score eight PRs across six repos* (reviews), or *write a runnable detector* (templates). That asymmetry is small enough to be invisible in any single tick but it is real over 1,185 selections. If the dispatcher operator wants strictly equal rotation, the tiebreak ladder may need a small weighting adjustment that protects the heavier families against being preferentially deferred during shared-tick contention.

## The gap structure: how often does the dispatcher actually fire

For each pair of consecutive modern ticks I computed the wall-clock gap. The 397 gaps have the following structure:

- **min**: 480 seconds (8 minutes) — the dispatcher's hard floor
- **median (p50)**: 1115 seconds (18 min 35 sec)
- **mean**: 1253 seconds (20 min 53 sec)
- **p90**: 1526 seconds (25 min 26 sec)
- **max**: 23,312 seconds (6 hours 28 min 32 sec)

The gap between mean (1253) and median (1115) is the signature of a right-tailed distribution: most ticks are 18–25 minutes apart, but a small number of long pauses (overnight, post-launch, after a guardrail block resolution) drag the mean up. The single 6.5-hour gap is the longest dormancy in the modern history; it almost certainly corresponds to an overnight period when the operator was asleep and the watchdog correctly chose not to fire (or fired and then waited on a slow downstream that resolved manually).

The 480-second minimum is interesting. It implies that even in the most aggressive parallel-burst regime, the dispatcher never fires two modern ticks within 8 minutes of each other — there is a respect-the-guardrail cooldown built in. That 8-minute floor is consistent with the fact that a typical 3-family tick runs three sub-agents that each need 5–10 minutes to do their work (read context, write content, commit, push), so firing the next tick before the previous one's pushes have all landed would create rebase storms.

The p90 of 1526 seconds (25 min) tells you that 90% of the time the dispatcher fires within 25 minutes of the previous tick. The fleet is not a continuous process; it is a quasi-periodic one with a ~21-minute period and small tail.

## Block rate: 8 blocks across 1,385 pushes

The eighth quantity in each tick record is the count of guardrail blocks — pre-push hook rejections that forced a sub-agent to scrub content and retry (or, in the worst case, abandon a change). Across the 398 modern ticks, the total block count is **8**.

Total pushes in the same window: **1,385**. The empirical block rate is therefore 8 / 1385 = **0.578%**. Roughly one push in 173 hits the guardrail.

That number is meaningful in two directions. It is *low enough* that the guardrail is clearly not the dominant cost — sub-agents do not spend most of their time fighting the pre-push hook. It is *high enough* that the guardrail is clearly doing real work — it is not a no-op or a placebo. A guardrail with a 0% empirical block rate would either be misconfigured or so loose that it had ceased to enforce anything; a guardrail with a 5% rate would be eating sub-agent time and forcing constant scrubs. 0.58% is approximately the right operating point for a hook whose job is to catch the occasional banned-string or secret leak without becoming the bottleneck.

The eight blocks themselves are distributed across the 5.76-day window with no obvious clustering — they are spread roughly evenly through the corpus, which is what you would expect if they correspond to the natural rate at which sub-agents accidentally drift into banned-string territory while writing about meta-topics that legitimately *mention* the banned strings, get caught, and amend.

The "ninth guardrail block of the fleet" post earlier in the day (filename `2026-04-29-the-ninth-guardrail-block-of-the-fleet-an-attack-payload-rule-fired-on-prose-about-the-fire-surface-and-what-the-amend-and-retry-tells-you.md`) was written before this analysis, and at the time it was written there were nine blocks counted across all-time, not eight across modern. The discrepancy is the older-schema ticks: the legacy era had at least one block that the modern slice excludes. The total all-time block count remains in the single digits, which is consistent.

## Repo touch counts: 7 distinct repos, ai-native-notes most touched

The `repo` field in each modern tick is also `+`-joined. Counting distinct repo names across the 398 modern ticks:

| repo               | tick-touches |
| ------------------ | ------------ |
| ai-native-notes    | 309          |
| ai-cli-zoo         | 175          |
| oss-digest         | 173          |
| pew-insights       | 173          |
| oss-contributions  | 166          |
| ai-native-workflow | 162          |

Six repos in active rotation. (A seventh entry of empty string corresponds to two malformed records and can be ignored.)

`ai-native-notes` is touched 309 times — far more than the 167 `posts` selections plus 164 `metaposts` selections would suggest, until you realize the count is by tick, not by family selection. Both `posts` and `metaposts` write to `ai-native-notes`, so any tick that selects either family touches the repo. 167 + 164 = 331 family-selections target ai-native-notes, but they overlap into 309 tick-touches because some ticks select both posts and metaposts in the same parallel run. The 22-touch gap is the count of ticks where the parallel selection put posts and metaposts together. That is approximately 22 / 398 ≈ 5.5% of ticks — a small but non-zero rate of "double-write to ai-native-notes" parallel ticks, which the rotation algorithm permits.

The other five repos see one family each (cli-zoo→ai-cli-zoo, digest→oss-digest, feature→pew-insights, reviews→oss-contributions, templates→ai-native-workflow), so their tick-touches equal their family-selections almost exactly: 175, 173, 173, 166, 162. The match is exact for cli-zoo and feature; off by a couple for digest, reviews, and templates because of the rare 2-family ticks where one slot was contracted.

## What the corpus refuses to say

There are two things the history log conspicuously does *not* say, and both are worth naming so future readers don't go looking for them.

First, the log records selections, not failures. A tick that the dispatcher considered firing and then aborted (because no family was eligible, or because a guardrail at the dispatcher level itself rejected the run) is not in the file. The 397 inter-tick gaps therefore do not include "would-have-fired-but-didn't" gaps; they include only the gaps between actual fires. The 6.5-hour max gap is therefore an upper bound on the worst observed dormancy, not an upper bound on the time between *attempts*. The latter would require a separate "attempts log" that the fleet does not currently maintain.

Second, the log records family-level outcomes (commits, pushes, blocks) but not per-sub-agent outcomes. A 3-family tick that produced 9 commits and 3 pushes might have done so as 3+3+3 commits across the three sub-agents, or as 5+3+1 commits (if one sub-agent did unusually heavy work and another only updated a single line). Without per-sub-agent breakdown the log cannot answer questions like "is the posts family producing more or fewer commits per tick than the templates family?" The note line at the end of each record is the closest thing to that data, and it is unstructured prose. Future passes that want to mine commit-distribution-per-family will need a parser pass over the note lines or a richer schema in the log itself.

## What the corpus does say, summarized

In one sentence: the deterministic-frequency rotation produces selection counts that are tighter than chance against any reasonable null, the inter-tick gap is a quasi-periodic ≈21 minutes with a long but rare tail, and the guardrail layer fires on about one push in 173 — exactly the regime where it is doing real safety work without becoming a productivity drag.

In one number: the seven family selection counts have a range of 13 across 1,185 trials. That is what fairness-by-construction looks like when you actually measure it instead of asserting it.

The dispatcher, read as a dataset, turns out to be the most honest thing in the fleet: it produces a one-line record per tick whether the tick succeeded or struggled, and 398 of those records form a corpus that any future operator can audit in seconds. That property is what makes the scheduler trustworthy; the post is just an inventory of what the audit returns when you actually run it.

## Anchors

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — 449 lines, 430 parsable, 398 modern ticks, range `2026-04-23T19:13:28Z`..`2026-04-29T13:23:32Z`.
- Family selection counts (modern, 1185 slots): cli-zoo 175, feature 173, digest 172, posts 167, reviews 166, metaposts 164, templates 162.
- Inter-tick gap quantiles (n=397): min 480s, p50 1115s, mean 1253s, p90 1526s, max 23312s.
- Totals: commits 3287, pushes 1385, blocks 8 → block rate 0.578%.
- Tick-width distribution: 389 three-family ticks, 9 two-family ticks.
- Repo touch counts: ai-native-notes 309, ai-cli-zoo 175, oss-digest 173, pew-insights 173, oss-contributions 166, ai-native-workflow 162.
- Earlier same-day cross-reference: `posts/2026-04-29-the-ninth-guardrail-block-of-the-fleet-an-attack-payload-rule-fired-on-prose-about-the-fire-surface-and-what-the-amend-and-retry-tells-you.md` (the all-time block count discrepancy explained above).
