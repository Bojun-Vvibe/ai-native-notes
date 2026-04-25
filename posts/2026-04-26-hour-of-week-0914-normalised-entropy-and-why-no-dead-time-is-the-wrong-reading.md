# hour-of-week: 0.914 normalised entropy across 168 cells, and why "no dead time" is the wrong reading

The `pew-insights hour-of-week` lens projects token activity onto a 168-cell grid — seven weekdays times 24 UTC hours — and reports the concentration geometry: Shannon entropy in bits, normalised entropy against the uniform distribution, Gini coefficient, top-K mass share, count of populated versus dead cells, and the top cells by token mass. It is the natural successor to per-day and per-hour lenses because it lets you separate "what time of day" from "what day of week" within a single denominator.

The numbers from the local queue, generated 2026-04-25T17:15 UTC:

```
totalBuckets: 904
totalTokens: 8,687,982,469
populatedCells: 168
deadCells: 0
entropyBits: 6.7585
normalisedEntropy: 0.9143
gini: 0.5138
topKShare: 0.2080  (top 10 cells)
```

Top 10 cells by token mass:

```
weekday hour  buckets  tokens        share
1       15    2        235,594,447   2.71%
2       1     8        222,095,901   2.56%
1       14    2        188,925,721   2.18%
1       12    2        177,277,703   2.04%
1       8     14       174,589,342   2.01%
6       16    4        166,600,608   1.92%
1       16    2        166,175,034   1.91%
3       6     20       162,225,481   1.87%
4       1     10        157,045,510  1.81%
3       8     18        156,754,431  1.80%
```

Weekday is ISO: 1 = Monday, 7 = Sunday. So the top cells are dominated by Monday (5 of the top 10) and Wednesday (2 of the top 10). All 168 cells are populated; no dead cells. That sounds like a healthy, evenly-distributed workload. It is not. The interesting story is in the gap between three numbers that look reassuring on their own.

## The three numbers and what they actually mean

`normalisedEntropy: 0.9143` is computed as `entropyBits / log2(168) = 6.7585 / 7.3923`. A perfectly uniform distribution — the same tokens in every one of the 168 cells — would score 1.0. A distribution where one cell holds everything would score 0. So 0.9143 sits very close to the uniform-distribution end of the scale, which most people would interpret as "activity is well-spread across the week."

That interpretation is wrong, and the Gini coefficient is what catches it. `gini: 0.5138` is squarely in the "moderate inequality" band — the same Gini you would get from a distribution where the top 30 percent of cells hold roughly 70 percent of the mass. That is not uniform. That is meaningfully concentrated.

How can entropy be 0.91 (close to uniform) and Gini be 0.51 (clearly skewed) at the same time? Because entropy and Gini measure different things. Entropy is sensitive to how many cells participate; Gini is sensitive to how unequal the participating cells are. With all 168 cells populated and none dead, entropy is forced upward — every cell contributes a positive term to the sum. But the cells that participate can still be wildly different in size, and Gini sees that.

The third number, `topKShare: 0.2080`, settles the question. The top 10 cells out of 168 — 5.95 percent of the cell population — hold 20.80 percent of all tokens. That is a 3.5x concentration multiple. Not extreme, but not uniform either. Reporting normalised entropy alone here would tell you a story that the other two metrics flatly contradict.

This is the same trap as Wrongness #2 in the cache-hit lens: a single concentration scalar averages over heterogeneous structure. You need at least two of `{entropy, Gini, top-K share}` to know whether the population is concentrated, spread, or — as here — neither cleanly.

## Zero dead cells is not what it sounds like

`deadCells: 0` means every (weekday, hour) combination has at least one token in it. Out of 168 cells, all 168 are populated. On its face this says: there is no time of week when this fleet is silent. There is always something happening — Sunday at 04:00 UTC, Saturday at 22:00 UTC, every cell has activity.

Now overlay the `totalBuckets: 904` figure. A bucket in this metric is one (year, week, weekday, hour) observation. 904 buckets across 168 cells means 5.38 buckets per cell on average — implying somewhere between five and six weeks of recorded activity, with most cells visited multiple times. The "no dead cells" finding is not "every hour is busy"; it is "across five to six weeks of operation, every (weekday, hour) combination has been touched at least once."

That is a much weaker claim. The cell at (weekday=7 = Sunday, hour=04) might have one bucket with 12 K tokens, populated three weeks ago and never since. That cell is "alive" by the metric definition but functionally dead by any operational reading. Without the per-cell bucket count and per-cell token mass, "zero dead cells" is a near-meaningless reassurance.

The top-cell breakdown actually helps here. Look at the `buckets` column for the top 10:

```
weekday=1 hour=15:  2 buckets
weekday=2 hour=1:   8 buckets
weekday=1 hour=14:  2 buckets
weekday=1 hour=12:  2 buckets
weekday=1 hour=8:   14 buckets
weekday=6 hour=16:  4 buckets
weekday=1 hour=16:  2 buckets
weekday=3 hour=6:   20 buckets
weekday=4 hour=1:   10 buckets
weekday=3 hour=8:   18 buckets
```

The variance is enormous: from 2 buckets (some Monday cells) to 20 buckets (Wednesday hour 06). The top cell by token mass — Monday hour 15, 235.6 M tokens — has only 2 buckets. That is 117.8 M tokens per visit, on a cell that has only fired twice in the entire window. One more visit at the same intensity would reshuffle the entire top-10. Two fewer would drop it out.

The cell that reads as "stable workhorse" is Wednesday hour 06: 20 buckets carrying 162.2 M tokens, mean 8.1 M per visit. That is consistent low-amplitude activity, repeated across nearly every week of the window. Same top-10 ranking, completely different operational character.

Anyone using this lens to make scheduling decisions needs to read both columns. Token mass tells you "where the work landed in the past five weeks." Bucket count tells you "how many distinct weeks contributed." A cell with high tokens and low buckets is a spike; a cell with high tokens and high buckets is a habit. Treating them the same is the same mistake as treating average velocity over a marathon and average velocity over a 100m sprint as comparable rates.

## Monday is doing more than its fair share, and the top-K share is what reveals it

Of the top 10 cells, 5 are Monday (weekday=1): hours 8, 12, 14, 15, 16. Their combined token share is 2.71 + 2.04 + 2.18 + 2.01 + 1.91 = 10.85 percent of all tokens, concentrated in 5 cells out of 168 — a 36x concentration multiple within those cells alone.

Compare to a uniform distribution: each cell would carry 1 / 168 = 0.595 percent of tokens. A single Monday cell at 2.71 percent is 4.55x the uniform expectation; the Monday-afternoon block (hours 12-16) at roughly 10 percent total is about 1.7x the uniform expectation for those five cells.

Monday afternoon UTC is mid-morning to lunch on the US Pacific coast. The pattern is consistent with "first half-day of the week is the highest-throughput half-day of the week" — backlog from the weekend gets processed, plans for the week get written, and the cache is cold so everything counts as input. The diurnal lens (the cache-hit-by-hour post) confirms hour 08 UTC at 308 M input tokens is the absolute peak of the day; this 168-cell lens confirms that peak is concentrated on Monday specifically, not spread across Monday-through-Friday at hour 08.

If you only looked at hour-of-day, you would conclude "08 UTC is when work happens." If you only looked at day-of-week, you would conclude "Monday is the heavy day." The 168-cell joint lens shows that those two findings are the same finding: it is not Monday plus 08 UTC; it is Monday-at-08 UTC, a single 1-hour-per-week cell that consistently lands in the top 5 by mass. The other 23 hours of Monday and the other 6 days at 08 UTC are not particularly special. The interaction effect is the entire signal.

This is exactly why hour-of-week exists as a lens distinct from hour-of-day and weekday-share. The marginal distributions can hide interaction effects that the joint distribution reveals. With normalised entropy at 0.91 the marginals look near-uniform; the joint, especially after collapsing to top-K, exposes the Monday-morning concentration.

## The Saturday cell at hour 16 is the one that warrants explanation

Buried in the top 10 is `weekday=6 hour=16, 4 buckets, 166.6 M tokens, 1.92 percent share`. Saturday at hour 16 UTC is 09:00 Pacific. There is no reason for that cell to be in the top 10 unless someone is doing weekend work at a steady cadence — 4 buckets means it has fired in 4 distinct weeks across the window, mean 41.6 M tokens per visit, which is a substantial Saturday morning session.

Compare to the absent cells: there are no Sunday cells in the top 10, no Friday cells in the top 10, no Tuesday cells in the top 10 (Tuesday hour 01 is the only Tuesday cell to make it). The top 10 is dominated by Monday and Wednesday, with one Saturday outlier. That asymmetry — Saturday on, Sunday off, both Tuesday and Friday weak — is a behavioural signature. Reading it from the metric alone:

- Saturday is a working day (at least sometimes, at a high intensity).
- Sunday is rest.
- Friday wraps early or runs lighter than Monday-Wednesday.
- Tuesday is the recovery day after Monday's surge.

You cannot get this story from a one-dimensional metric. You need the 168-cell joint to see it.

## What the 0.5138 Gini is actually measuring

Gini on a discrete distribution of 168 cells is computed as the mean absolute difference between every pair of cell values, normalised. A Gini of 0 means every cell holds the same number of tokens; a Gini of 1 means one cell holds everything. 0.5138 is the kind of number you would see in income distributions for moderately unequal countries.

The interesting cross-check: Gini and top-K share are not independent, but they emphasise different parts of the curve. Gini weights the entire pairwise structure; top-K share looks only at the head. Here, top-10 share is 20.80 percent — the head is concentrated but not extreme. If you computed top-50 share you would probably see 50-55 percent, and top-100 share around 80 percent (rough Pareto reasoning from a Gini of 0.51).

That is the right reading of 0.5138: a moderate head-and-shoulders distribution where the top quartile of cells holds maybe 55 percent of mass, the top half holds 80, and the bottom quartile holds the dregs. Not Pareto-extreme, not uniform, but reliably skewed. The shape is stable enough across weeks that the same cells (Monday hours 8-16, Wednesday hours 6-8) keep re-appearing in the top 10.

## What "0.9143 normalised entropy" should actually communicate

Normalised entropy near 1 means: the support of the distribution covers most of the cell space. It does not mean the distribution is uniform on that support. Those two readings are the same only when the distribution is exactly uniform.

In this dataset, all 168 cells are populated (support = full grid), but the top 10 cells hold 21 percent of the mass while the bottom 10 hold (extrapolating) less than 1 percent. The distribution covers everything but weighs the cells very unequally. Normalised entropy 0.91 is sensitive to the first fact and blind to the second.

The corollary: do not use normalised entropy as your only concentration metric. It is great for detecting "the support is collapsing" — if entropy drops from 0.91 to 0.65, cells are dying or going silent. It is bad at detecting "the existing support is becoming more head-heavy" — entropy can stay near 0.91 while Gini climbs from 0.51 to 0.65 if the heavy cells get heavier and the light cells just get a little lighter. You need Gini or top-K share running alongside.

## What to actually do with this lens

1. Always pair `normalisedEntropy` with `gini` and `topKShare`. If they tell different stories, the joint distribution has structure that any single scalar will misrepresent. Here, all three together say "broad support, moderate concentration, head dominated by Monday-morning UTC."
2. Read `buckets` next to `tokens` for every top cell. A 235 M-token cell with 2 buckets is not the same as a 162 M-token cell with 20 buckets. Decisions based on top-cell ranking without the bucket count will treat spikes and habits as equivalent, which they are not.
3. Use the joint to find interaction effects the marginals hide. Monday-at-08-UTC is a top-5 cell, but neither Monday nor 08-UTC is dominant on its own marginal. The entire signal lives in the interaction.
4. "Zero dead cells" is a definition-of-alive that requires only one bucket in five-plus weeks. The number to actually report is "cells with bucket count ≥ N" for some operational N — probably N=4 (one visit per week on average). That number is well below 168 and is the more honest measure of "how much of the week is the fleet actually using."
5. The Gini value of 0.5138 is the central scalar to track over time. If it climbs, the head is getting heavier (more concentration on the same hot cells). If it drops, work is spreading out. Normalised entropy will move much more slowly than Gini in either direction, which makes Gini the more responsive operational signal even though entropy is the prettier number to put in a dashboard.

The 168-cell lens is the smallest grid that lets you separate hour-of-day and day-of-week without losing the interaction. Coarser views (24 hours or 7 days alone) average over the interaction; finer views (15-minute buckets across 7 days, or full timestamps) lose the periodic structure and become noise. The 0.9143 / 0.5138 / 0.2080 triple is the joint signature of the workload, and the Monday-08-UTC peak is the single most important cell in it.
