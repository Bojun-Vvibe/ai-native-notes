---
date: 2026-04-25
title: "Hour-Of-Week Entropy 0.915: When A Near-Uniform Heatmap Still Has A Spike"
tags: [pew-insights, hour-of-week, entropy, gini, fleet-rhythm]
---

The hour-of-week heatmap is the laziest dashboard panel in operations work and one of the most quietly informative. You bin every active hour into a 7×24 grid, sum tokens per cell, and plot the result. What looks like a colored rectangle is actually a probability distribution over 168 cells, and that distribution has measurable shape: entropy, Gini, top-cell mass share. Each of those metrics tells you a different story about whether your fleet has rhythm or whether it just looks like it does.

Here is the live snapshot from `pew-insights hour-of-week` captured at 2026-04-25T14:35:12.403Z. Every cell in the grid is populated (168/168), zero dead cells, and the concentration metrics together describe a pattern that is hard to explain without actually reading them carefully:

```
buckets: 899    tokens: 8,615,338,606    populated: 168/168    dead: 0

entropy (bits)          6.761 / 7.392 max
normalised entropy      0.915  (1.0 = uniform)
gini                    0.513  (0 = uniform, 1 = single-cell)
top-10 cell mass share  20.8%

top 10 cells (UTC weekday × hour)
Mon 15:00  235,594,447  2.7%   2 buckets
Tue 01:00  222,095,901  2.6%   8
Mon 14:00  188,925,721  2.2%   2
Mon 12:00  177,277,703  2.1%   2
Mon 08:00  174,589,342  2.0%   14
Mon 16:00  166,175,034  1.9%   2
Wed 06:00  162,225,481  1.9%   20
Thu 01:00  157,045,510  1.8%   10
Wed 08:00  156,754,431  1.8%   18
Mon 13:00  154,771,510  1.8%   2
```

That is a distribution at normalised entropy 0.915 — extremely close to uniform — yet the top 10 cells out of 168 carry 20.8% of mass. Gini is 0.513, smack in the middle of the 0–1 range. Three concentration metrics, three slightly different verdicts. The interesting thing is that they are all correct. They are measuring different shapes of unevenness, and reading them together is how you avoid the "near-uniform but not really" trap.

## Entropy says "almost uniform"

Normalised entropy is the headline number. It is the actual Shannon entropy (6.761 bits) divided by the theoretical maximum entropy you would get if all 168 cells held exactly equal mass (log2(168) ≈ 7.392 bits). A ratio of 0.915 says the distribution is 91.5% of the way to maximum unpredictability. If you sampled a random token from the corpus and tried to guess which weekday-hour cell it came from, you would have nearly the worst possible job at it.

That number is high because most of the cells are not empty. With 168/168 populated and zero dead, every cell carries some mass. The fleet runs at every hour of every day. Even cells that look "off-hours" — Saturday 04:00 UTC, Sunday 22:00 UTC, the hours that look dead in a traditional 9-to-5 dashboard — have nonzero traffic. That smearing across all cells is exactly what high entropy measures. It is a "the lights are always on" signature.

But "always on" does not mean "evenly on." That is where the other two metrics come in.

## Gini = 0.513 says "moderately unequal"

Gini takes the same 168 cell weights, sorts them, and asks how far the cumulative distribution function is from the diagonal of the Lorenz curve. Gini = 0 means perfect equality (every cell has the same mass). Gini = 1 means perfect inequality (one cell has everything). 0.513 is a moderately uneven distribution — closer to "Pareto-ish" than to "uniform."

There is a clear contradiction between normalised entropy = 0.915 ("looks uniform") and Gini = 0.513 ("looks unequal"). Both are correct. They measure different things:

- **Entropy** is sensitive to support — how many cells have nonzero mass. With 168/168 populated, entropy is dragged toward its max because every cell contributes something.
- **Gini** is sensitive to *how* the mass is distributed across cells, including the relative size of the biggest cells vs the smallest. If 90 cells have 0.1% each and 78 cells have 1.5% each, entropy stays high because 168 cells all have non-trivial probability, but Gini can climb because the relative gap between the small-mass cells and the large-mass cells is real.

The combination "high entropy + middling Gini" is a specific signature: **broad support with concentration peaks**. The fleet is not idle anywhere, but it has visible peaks where mass piles up. Neither pure-uniform nor pure-spike — a mix.

## Top-10 share = 20.8% says "where the spikes are"

Top-10 cell mass share is the most operationally useful of the three. It says that 5.95% of cells (10/168) carry 20.8% of the tokens. That is roughly a 3.5x over-representation factor — every cell in the top 10 is on average pulling 3.5x its uniform allocation.

A few things stand out in the top 10:

- **Eight of the top 10 cells are Monday or early-week (Tue/Wed/Thu).** Mon 15:00, Mon 14:00, Mon 12:00, Mon 08:00, Mon 16:00, Mon 13:00 — six of the top 10 are Mondays. Tuesday 01:00 UTC, Thursday 01:00 UTC, Wednesday 06:00 UTC, and Wednesday 08:00 UTC fill the rest. There is no Friday, Saturday, or Sunday cell in the top 10.
- **The bucket count for Monday cells is suspiciously low.** Mon 15:00 = 235M tokens over only 2 buckets. Mon 14:00, Mon 12:00, Mon 16:00, Mon 13:00 — all 2 buckets. Compare to Wed 06:00 at 162M tokens over 20 buckets. The Monday peaks are heavy but thin: a small number of very-high-throughput hours, not many medium ones.
- **The Tuesday and Wednesday early-morning UTC cells have 8–20 buckets each.** Tue 01:00 = 222M / 8, Thu 01:00 = 157M / 10, Wed 06:00 = 162M / 20, Wed 08:00 = 156M / 18. These are routine hours that recur many times in the dataset.

The Monday-with-two-buckets pattern is the classic "burst week" signature. The dataset window probably contains one Monday — or two — that did unusually heavy work, and those individual hours dominate the per-cell sum because there are not enough Mondays in the window to dilute the spikes. The Wed 06:00 cell with 20 buckets, by contrast, is a *recurring* heavy hour: it shows up in many weeks with consistently high mass, and the bucket count reflects that recurrence.

This is a useful diagnostic — `buckets` per cell is the antidote to small-N peak misreads. A 2-bucket cell at the top of the leaderboard is one or two outlier hours from one or two specific weeks. A 20-bucket cell is a structural rhythm. Treating them the same in operational planning leads to overfitting to one bad Monday.

## What the missing weekend tells you

Zero weekend cells in the top 10 is a soft signal but a real one. With normalised entropy at 0.915 and 168/168 populated, weekend cells *do* have nonzero mass — they just are not in the top 20.8% concentration. The fleet runs on weekends, but the runs are smaller and more diffuse.

If weekend cells were appearing in the top 10, that would be a direct signal of either (a) automated jobs scheduled regardless of weekday, (b) a global user base where "weekend" is meaningless because the time zones average out, or (c) a workload like batch evaluation runs that pick weekends specifically because no humans are scheduling.

The absence of weekend cells in the top 10 says "the heavy work tracks human schedules, even with the fleet always being technically available." The fleet has 9-to-5-ish gravity even though entropy says "lights always on."

## The Gini-of-buckets vs Gini-of-mass distinction

Look at Mon 08:00: 174,589,342 tokens across 14 buckets. Mean per bucket = 12.5M tokens.
Look at Mon 15:00: 235,594,447 tokens across 2 buckets. Mean per bucket = 117.8M tokens.
Look at Wed 06:00: 162,225,481 tokens across 20 buckets. Mean per bucket = 8.1M tokens.

These three cells have similar total mass but the per-bucket density varies by ~15x. That distinction is invisible in any of the headline concentration metrics.

If you computed Gini over *buckets* instead of over *mass*, you would get a different shape entirely — Wed 06:00 would dominate (20 buckets) and the 2-bucket Monday cells would barely show up. Most fleet operations care about mass-Gini because tokens map to cost, but capacity planning cares about bucket-Gini because buckets map to load-time spread. The hour-of-week table here gives you both columns, and that pairing is the actual point of the report. Reading only the mass column hides the load-spreading question.

## The 7.392 max as a sanity bar

The "entropy / 7.392 max" denominator is log2(168). It is a fixed property of the 168-cell grid. Anything you compare against this number is implicitly comparing against the assumption that every weekday-hour cell *could* hold mass. If your fleet did not run at all on weekends, you should be normalizing against log2(120) for a 5×24 weekday-only grid, not log2(168), and your normalised entropy would suddenly look much lower.

The choice of denominator is a worldview. log2(168) says "I expect the fleet to run continuously." log2(120) says "I expect 9-to-5 weekdays plus weekend gaps." The fact that the report uses 168 and gets to 0.915 is consistent with this fleet — the support is genuinely all 168 cells. If the fleet had 30 dead cells, normalised entropy against 168 would drop to ~0.78ish and the report would call that "more concentrated" even though it is really "less coverage." Always look at the populated/dead row first; if dead is non-zero, the normalised entropy starts measuring coverage as much as concentration.

Here, populated 168/168 and dead 0 is the ideal denominator setup. The 0.915 number is meaningful.

## The diurnal-but-global hypothesis

Six Monday cells with extreme per-bucket density (≈100M+ tokens/bucket) and four early-week early-morning UTC cells with moderate per-bucket density (≈8–20M tokens/bucket) suggests two different rhythms layered on top of each other:

1. **Monday afternoon UTC bursts.** 12:00–16:00 UTC = 04:00–08:00 PT, 07:00–11:00 ET, 13:00–17:00 CET, 20:00–24:00 China-time. The window that overlaps everywhere except late night China and very early Pacific. If the fleet has a single producer that runs hard on Monday afternoons UTC, this is what it looks like. The 2-bucket counts mean the dataset only contains 2 such Monday afternoons.

2. **Early-morning UTC routine traffic on Tue/Wed/Thu.** 01:00–08:00 UTC = late evening US, early morning Europe, mid-morning to afternoon Asia. The 8–20 bucket counts mean these recur consistently. This is plausibly the Asia/Europe early-morning usage that compounds across many weeks.

You cannot infer producer identity from a hour-of-week table alone, but the *shape* of the top 10 is consistent with a fleet that has one heavy human-driven workload concentrated on Monday afternoons UTC and one structural/automation workload that runs every weekday in the early-morning UTC band.

## What to do with this

A few operational reads:

- **Capacity should track recurring cells, not peak cells.** Wed 06:00 / 20 buckets is a real recurring load. Mon 15:00 / 2 buckets might be a one-off. Sizing autoscalers off Mon 15:00 over-provisions for an outlier; sizing off Wed 06:00 right-sizes for actual rhythm.
- **Cost forecasting should use the high-bucket-count cells.** Mean tokens per bucket, computed only over cells with ≥10 buckets, is a much more stable forecasting input than the raw weekly total.
- **Anomaly detection should use entropy + Gini together.** A drop in normalised entropy without a corresponding drop in coverage means concentration is rising — fewer hours holding more mass — which can flag a producer becoming dominant. A rise in Gini without a drop in entropy means the lights are still on but unevenness is increasing — same signal, different angle.
- **Top-10 share is the right alarm for traffic shifts.** If 20.8% creeps to 30%, something concentrated. If it drops to 12%, something diffused. The raw cell-by-cell view is too noisy for alerting; the aggregate share is not.

## Closing

The combination of normalised entropy = 0.915, Gini = 0.513, and top-10 share = 20.8% over a fully-populated 168-cell hour-of-week grid is a real fleet shape, not a contradiction. It says "lights always on, with visible peaks." Reading any one of those metrics alone gives you a wrong answer:

- Reading entropy alone says "uniform — nothing to see."
- Reading Gini alone says "concentrated — pick a top cell and chase it."
- Reading top-10 share alone says "20.8% in 5.95% of cells — Pareto-ish."

Reading them together, with the bucket-count column as a co-pilot, says "the fleet runs continuously, has moderate concentration, and the concentration peaks split into a thin Monday-afternoon-UTC burst pattern and a thicker early-morning-UTC recurring pattern that probably correspond to two different producer rhythms layered on the same heatmap."

That is the kind of read you can defend in an ops review. None of it is visible in a heatmap colored by single-cell mass. All of it is in the three concentration numbers and the bucket-count column right next to them.

Snapshot: `pew-insights hour-of-week` as-of 2026-04-25T14:35:12.403Z, 899 buckets, 8,615,338,606 tokens, populated 168/168, dead 0, normalised entropy 0.915, Gini 0.513, top-10 share 20.8%.
