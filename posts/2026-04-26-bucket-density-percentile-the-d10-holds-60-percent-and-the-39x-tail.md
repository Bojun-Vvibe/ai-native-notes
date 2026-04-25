# Bucket-density percentile: the D10 holds 60.3% and the 39× p50→p99 tail

*Posted 2026-04-26. Data from `pew-insights bucket-density-percentile`,
snapshot `as of: 2026-04-25T21:24:58.324Z`, 1,487 active hour-buckets,
8,803,445,877 tokens, mean 5,920,273 tokens per bucket.*

---

## What the lens does

`bucket-density-percentile` is the simplest possible question you can ask
of a token-emission log: *if I rank every active hour-bucket on this device
by its token mass and walk the ranks, what does the distribution look like?*
No grouping by source, no grouping by model, no grouping by day. Pool
everything across the device's entire history, sort, and read the
percentile ladder.

It sounds boring. It is the most useful single table in the suite, because
every other concentration statistic — Gini, HHI, Pareto-tail, top-decile
share — collapses out of this one ladder. If you only ever look at one
report from this dataset, look at this one.

## The numbers

The snapshot at `2026-04-25T21:24:58.324Z` covers 1,487 active buckets
holding 8.80B tokens. The full ladder:

```
p      tokens
-----  -----------
min    20
p1     174
p5     751
p10    1,810
p25    86,020
p50    1,493,121
p75    5,787,576
p90    14,569,760
p95    31,886,437
p99    59,093,095
p99.9  87,339,377
max    107,646,380
```

And the decile-mass companion table:

```
decile  count  tokens         share  lower       upper
------  -----  -------------  -----  ----------  -----------
D1      148    119,711        0.0%   20          1,789
D2      149    667,648        0.0%   1,810       10,082
D3      149    14,039,352     0.2%   10,189      244,740
D4      148    65,156,609     0.7%   245,090     706,478
D5      149    168,226,713    1.9%   708,617     1,486,951
D6      149    300,458,989    3.4%   1,493,121   2,622,496
D7      148    520,929,142    5.9%   2,624,938   4,555,061
D8      149    883,852,423    10.0%  4,588,760   7,498,109
D9      149    1,542,885,998  17.5%  7,518,924   14,520,585
D10     149    5,307,109,292  60.3%  14,569,760  107,646,380
```

This is not a long-tail distribution. It is a *short-head* distribution
masquerading as a long tail.

## Reading the ladder

### The bottom 25% of buckets are functionally empty.

p25 is 86,020 tokens. p10 is 1,810. p5 is 751. p1 is 174. **p25 of the
distribution sits at 1.4% of the mean.** The bottom quarter of all active
hour-buckets contributes less than the rounding error of the top quarter,
and the bottom decile contributes less than 0.01% of total mass.

This matters because most reports that aggregate across "active buckets"
implicitly weight every bucket equally. They shouldn't. Three out of every
four buckets on this device hold less than 0.4% of the mean — they are
present in the activity log but absent from the cost/load picture. If you
are running an alert on "buckets per day with >0 tokens" you are counting
ghosts.

### The median is 4× below the mean.

Mean: 5,920,273. p50: 1,493,121. The ratio is 3.96. Whenever
mean/median > 1.5 in this kind of count data you have a right-skewed
distribution and the mean is being pulled by the tail. Here the mean is
roughly co-located with **p70 of the distribution**, so calling 5.9M the
"average bucket" is misleading by construction. The honest single-number
summary of this device is "the median active hour-bucket spends 1.5M
tokens and the top decile spends an order of magnitude more." Anything
that smooths over that distinction will mis-budget.

### The 39.6× p50-to-p99 spread.

p99 / p50 = 59,093,095 / 1,493,121 = 39.58. The 99th-percentile bucket is
nearly 40× heavier than the typical bucket. For comparison, the
**peak-to-typical ratio** (max / p50) is 72.1; the **peak-to-99th** is
1.82. Translating: once you cross p99, the top 1% is internally
*not* particularly extreme; max is just under 2× the p99. The dramatic
gap is between the median and p99, not between p99 and max.

That shape — flat tail beyond p99, steep climb between p50 and p99 — is
characteristic of a regime where the heaviest buckets are produced by the
same process repeatedly, and the median buckets are produced by a
fundamentally different lighter process. There aren't a few catastrophic
outliers; there are 149 buckets in the top decile and they all behave
similarly.

### D10 is doing 60.3% of the work; D1–D4 contribute 1.0% combined.

The decile mass table is the headline. **The top 10% of buckets emit
60.3% of all tokens. The bottom 40% of buckets emit 0.94%.** A 64×
mass ratio between top and bottom deciles, against a 1× ratio in
*count*. If you wanted a one-sentence Pareto for this device it would
be "10% of hours hold 60% of compute" — slightly steeper than the
classic 80/20 but in the same family.

But it goes further than that. D9 contributes 17.5% and D10 contributes
60.3% — together D9+D10 is 77.8% of all tokens in 20% of the active
buckets. Push to D8+D9+D10: 87.8% of tokens in 30% of buckets. The
"useful" buckets on this device are the top three deciles, full stop;
the bottom seven deciles together account for less than 13% of mass.

## Why the bottom decile is full of 20-token entries

The minimum bucket holds 20 tokens. p1 is 174. These are not zeros —
zero-token buckets were dropped (`dropped: 0 zero-tokens`). They are
genuine emission events that produced almost nothing. Three plausible
sources of these micro-buckets:

1. **Failed or aborted requests** that still emitted a system prompt
   counted as input. A 20-token bucket is consistent with a tokeniser
   counting exactly the prompt-cache header of a request that never
   produced output.
2. **Health-check or probe traffic** from a launchd-style watchdog
   that periodically pokes a model endpoint with a minimal prompt.
3. **First-bucket-of-session warmup** — the operator opening a tool,
   a session starting, the first round-trip happening, then the
   operator switching contexts before generating anything substantive.

The fact that 148 buckets fall into D1 (`upper: 1,789`) and another
149 fall into D2 (`upper: 10,082`) tells you these aren't accidents.
~20% of all active hour-buckets on this device carry less than 10K
tokens. That is a *lot* of probe-shaped traffic, and it should make
anyone using bucket-count as a proxy for activity suspicious.

## The implications for cost modelling

Cost-per-bucket distributions inherit this shape directly. The companion
report `cost-per-bucket-percentiles` shows per-source dollar p50/p90/p99,
and they are spread by the same factor. If a budget alarm is calibrated
on the mean cost per bucket, it will **fail to fire on the median day**
(median cost is 25% of the mean) and **fire constantly on top-decile
days** (D10 is 60% of mass concentrated in 10% of buckets, so any
top-decile day will look catastrophic against a mean baseline).

The right calibration is two-stage: a low threshold on D5–D8 to catch
medians drifting up, and a high threshold on D9–D10 to catch the tail
getting worse. A single-threshold alert on this distribution is
guaranteed to be wrong in one direction or the other.

## A falsifiable test

The decile table makes one prediction sharp enough to falsify in a week.
**The Gini-equivalent of this distribution is structural, not noise.**
Specifically: if the lens is rerun on `2026-05-03T21:24Z` (one week
forward) the D10 share will land in `[57%, 63%]`. The bound comes from
the observation that across the most recent four sources to onboard
(hermes, opencode, openclaw, codex — see the half-life report), all of
them produced top-decile-eligible buckets within their first 48 hours.
There is no plausible mechanism by which the device suddenly stops
producing extreme buckets without a corresponding drop in total volume,
which would itself violate the trend in `forecast`.

If the next-week D10 share lands outside `[57%, 63%]`, one of three
things has changed: (a) a heavy source has gone silent (codex-style
exit); (b) a new heavy source has joined that produces *flatter*
buckets — implausible given the model-mix data; (c) the snapshot is
contaminated by a backfill or a clock jump. All three are diagnosable
without reaching for this lens; the lens just gives you the alarm
condition.

## The three things this lens is good for that nothing else replaces

1. **Sanity-checking other reports.** Anything that talks about
   "average bucket" cost, latency, or volume should be cross-checked
   against this median. A 4× mean/median gap means the average is
   never the typical case.

2. **Choosing the right grain for downstream stats.** If you are
   building a per-bucket alert, you want the threshold inside the
   D9–D10 band (`> 14.5M tokens`) because below that you are alerting
   on what the device does normally. If you are building a
   per-bucket SLA, the median (1.49M) is the right anchor.

3. **Detecting onboarding events.** A new source that produces
   D10-band buckets within its first 48 hours is making a
   commitment. A new source whose first 48 hours stays in D5 or
   below is being evaluated. Half-life-fraction confirms this
   in retrospect; the decile assignment of the *first ten buckets*
   would predict it in real time.

## Cross-references

- `tail-share` (per-source Pareto) reports the per-source equivalent of
  this device-wide Pareto and shows that the editor-side assistant has
  the steepest per-source concentration despite the smallest mass.
- `cost-per-bucket-percentiles` translates this token ladder into
  dollar terms per-source.
- `bucket-intensity` reports per-model token-per-bucket distributions
  and will show that the D10 buckets are dominated by long-context
  request shapes, not by output bursts.
- `source-decay-half-life` (snapshot `2026-04-25T21:24:57.819Z`)
  identifies that claude-code's 3.44B tokens are concentrated at
  half-life-fraction 0.932 — directly explaining a chunk of the D10
  mass.

## TL;DR

Snapshot `2026-04-25T21:24:58.324Z`. 1,487 active hour-buckets,
8.80B tokens. **D10 holds 60.3% of all mass; D1–D4 together hold 1.0%.**
The median bucket is 1.49M tokens, the mean is 5.92M (3.96×), and the
p99 is 39.6× the median. The bottom quarter of buckets is functionally
empty (p25 = 86K, ~1.4% of mean). 20% of buckets carry less than 10K
tokens and are best read as probe / abort / warmup traffic rather than
real work. Any alert calibrated on the device-wide mean will fire
wrong in both directions; the right calibration is a low threshold
inside D5–D8 plus a high threshold inside D9–D10. The next-week
D10-share prediction is `[57%, 63%]` and is falsifiable.
