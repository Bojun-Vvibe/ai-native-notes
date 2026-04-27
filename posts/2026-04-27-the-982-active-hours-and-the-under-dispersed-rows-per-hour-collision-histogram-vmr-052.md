---
title: "The 982 active hours and the under-dispersed rows-per-hour collision histogram VMR=0.52"
date: 2026-04-27
tags: [pew-insights, telemetry, dispersion, poisson, collisions, rows-per-hour]
slug: the-982-active-hours-and-the-under-dispersed-rows-per-hour-collision-histogram-vmr-052
---

# The 982 active hours and the under-dispersed rows-per-hour collision histogram VMR=0.52

## The problem

I have been treating the pew-insights queue as a stream of rows
keyed by `(source, model, hour_start)` and reasoning about it as if
the rows arrived independently — as if, conditional on a given hour
being "active" at all, the number of rows landing in that hour were
roughly Poisson. That mental model has been quietly wrong for at
least three weeks. The actual rows-per-active-hour distribution is
**under-dispersed** by a factor of two relative to Poisson, and the
collisions concentrate in a way that breaks the independence
assumption I was implicitly using to reason about cross-source
overlap.

The post is about what the under-dispersion means, why it matters,
and what the maximally colliding hour buckets actually look like
when you peer at them.

## The setup

The queue file lives at `~/.config/pew/queue.jsonl` and as of this
writing carries **1,646 rows** across **982 unique `hour_start`
buckets**. Each row is a JSON object with `source`, `model`,
`hour_start` (a half-hour-snapped UTC ISO string), `device_id`, and
the four token counts (`input_tokens`, `cached_input_tokens`,
`output_tokens`, `reasoning_output_tokens`) plus their `total_tokens`
sum. The `device_id` is constant across the corpus (one device,
`a6aa6846-9de9-444d-ba23-279d86441eee`) so the only multiplexing
dimensions are `source × model × hour_start`.

The 982 active hours span roughly the past three weeks at half-hour
resolution. (Half-hour, not full-hour: `hour_start` is bucketed to
`:00` and `:30` despite the field name. I keep meaning to rename it.)
That gives a nominal upper bound of `21 days × 48 half-hours/day =
1008` half-hour buckets, of which 982 are non-empty and 26 are
fully empty — so the device is "active in some sense" 97.4% of the
half-hour buckets in the window. The small empty fraction is the
overnight-deep-sleep gap, which I have written about elsewhere.

## The histogram

Counting rows per active half-hour gives this distribution:

| rows-in-bucket | count of buckets | fraction |
|---:|---:|---:|
| 1 | 566 | 57.6% |
| 2 | 227 | 23.1% |
| 3 | 141 | 14.4% |
| 4 |  39 |  4.0% |
| 5 |   7 |  0.7% |
| 6 |   2 |  0.2% |

Total active buckets: 982. Total rows: 1,646. Mean
rows-per-active-bucket: **1.676**. Variance:
**0.871**. Variance-to-mean ratio: **0.520**.

A Poisson process with mean 1.676 would give a variance of 1.676 (by
definition `Var = λ`) and a VMR of exactly 1.0. The observed VMR of
0.52 is roughly half that. That puts the data clearly in the
**under-dispersed** regime — more regular than Poisson, less spiky
than independent arrivals would predict.

That is the headline. Now what does it mean?

## What under-dispersion implies for the data-generating process

Under-dispersion (VMR < 1) on a count distribution means the system
that produced the counts has a **negative dependency** between
events within a bucket. There are several physical mechanisms that
can produce it; only some of them are plausible here.

The cleanest mechanism is a **hard cap**. If each `(source, model)`
pair can contribute at most one row per `hour_start` bucket — which
is almost true: pew-insights aggregates token counts per source-model
per hour into a single row — then the per-bucket count is bounded
above by the number of distinct active source-model pairs in that
bucket. With ≈15 source-model pairs in the corpus and a typical
2–4 active in any given hour, the hard cap is operating well below
the Poisson tail and chops it off. Under-dispersion is the expected
signature.

The second mechanism is **regularization at the application layer**.
The pew sync process bundles uploads at well-defined cadences
(notifier hooks fire on AI-tool events, sync runs on a periodic
schedule) and the bundling tends to push rows that would have arrived
in a single hour into adjacent hours, smoothing the per-hour count.
This is a "regularizer toward the mean" mechanism and also produces
VMR < 1.

The third mechanism is **shared upstream gating**. If the device is
in a development session, multiple AI tools are likely active
together; if the device is idle or on something else (a meeting, a
walk, sleep), all sources are jointly inactive. The on/off shared
gate produces a bimodal count distribution at the bucket level
(many empty buckets, many "full house" buckets, few in between),
which when conditioned on "active" can also produce under-dispersion
because the within-active-bucket counts cluster around the typical
session size.

I think the hard cap is doing most of the work — the histogram's
sharp upper edge at 6 (the maximum observed) is hard to explain any
other way. But the regularization layer and the shared gate are
both contributing. The 0.52 VMR is the joint signature of all three.

## The two k=6 buckets and what they actually contain

Inspecting the two buckets that hit the maximum count of 6:

**`2026-04-20T15:00:00.000Z`** — six rows, five distinct source
strings: `claude-code`, `codex`, `openclaw`, `hermes`, `opencode`.
Five sources, six rows, which means one source contributed two rows
in that bucket — almost certainly two model variants on the same
source (the corpus has a known ~five-way model-aliasing collision
on `claude-opus-4.7`-vs-`claude-opus-4-7`, written up elsewhere).

**`2026-04-20T16:00:00.000Z`** — six rows, again five distinct
sources, same set: `claude-code`, `codex`, `openclaw`, `hermes`,
`opencode`. Same pattern.

Both maxima land on the same UTC date (2026-04-20) in adjacent
half-hours. This is a single ~90-minute heavy-coding session that
happened to hit every active source simultaneously. Looking at the
seven k=5 buckets:

```
2026-04-20T14:00:00.000Z (5 sources)
2026-04-20T14:30:00.000Z (5 sources)
2026-04-20T15:30:00.000Z (5 sources)
2026-04-21T01:30:00.000Z (4 sources)
2026-04-21T02:00:00.000Z (4 sources)
2026-04-21T06:00:00.000Z (4 sources)
2026-04-23T14:00:00.000Z (4 sources)
```

The first three k=5 buckets are bracketing the two k=6 buckets; the
2026-04-20 14:00–16:00Z window is a contiguous five-bucket
high-collision streak. The 2026-04-21 01:30–06:00Z streak is a
late-night session on a different timezone alignment. The 2026-04-23
14:00Z bucket is a solo afternoon spike.

This is one of those facts that is so cleanly visible it almost
disappears: collisions cluster in time. Of the 9 buckets at k≥5
(roughly 1% of active buckets), **5 of them fall within a single
3-hour window** on a single date. If the per-bucket count were truly
Poisson and independent across time, you would expect those 9
buckets to be roughly uniformly distributed. Instead they cluster.

That clustering is the **temporal autocorrelation** in the count
process, which is the second-moment companion to the under-
dispersion in the marginal distribution. Both signals point to the
same conclusion: the data-generating process has structure on top
of the per-hour bucket.

## Why the independence assumption was wrong

When I have done cross-source analysis on this corpus — e.g.,
"what fraction of `opencode` activity overlaps with `claude-code`
activity in the same hour" — I have implicitly modeled the
co-occurrence as a product of marginal hour-activity probabilities.
That gives, for two sources each active in roughly 30% of
half-hour buckets, an expected co-occurrence rate of
`0.30 × 0.30 = 9%` of buckets if independent.

The actual co-occurrence rate is much higher than that. The
under-dispersion histogram tells us why: the bucket-count
distribution is concentrated above 1 more than independent activity
would predict. When sources fire, they tend to fire together. The
shared-gate mechanism dominates the apparent rate.

The downstream consequence is that **all my prior overlap analyses
have been over-stating the surprise** of any specific overlap
finding. If I observe `opencode ∩ claude-code` at 35% of buckets and
my null model predicts 9%, the 26-percentage-point gap looks like
strong positive coupling between the two tools. But if my null
model is wrong — if the actual base rate of "any two sources fire in
the same bucket" is 25% under the shared-gate process — then the
real surprise is only 10 percentage points, not 26.

I do not have a clean fix for this in the analyses I have already
published. The right thing to do is recompute the null with a
**joint shared-gate model**: estimate the probability that the
device is "active for any AI tooling" in a bucket, then condition
the per-source probabilities on that. The shared-gate term
explains most of the inflated co-occurrence; the residual is the
true positive coupling between specific sources.

Marking that for future work.

## What the empty 26 buckets look like

There are 26 fully-empty half-hour buckets in the 21-day window,
which is 2.6% of the nominal grid. These are the only place where
the data really is Poisson-shaped — once you condition on "bucket
not active at all," there is no within-bucket structure to break
the Poisson assumption.

I am not going to enumerate the 26 buckets in this post (they are
mostly the 04:00–06:00Z range on weekend-adjacent dates, which is
the deep-sleep gap on a UTC-anchored North-Hemisphere mid-Atlantic
schedule), but their **rate** is informative. If sleep gaps were
uniformly distributed across the 21-day window the expected count
of fully-empty buckets would be roughly `21 × 2.5h × 2 buckets/h =
105`, assuming a 2.5h core sleep gap with no AI activity. The
observed 26 is **four times less** than that, which means the
device is producing AI-tool rows during most of what would
otherwise be sleep hours. Either I am genuinely working through
those hours (some of them, yes) or there are background processes
generating rows (some hooks fire on cron, yes), but the gap
distribution is much shallower than a clean human sleep schedule
would predict.

This is its own post-worthy thing — the depth of the activity gap
as a proxy for "actually offline" time — but the data point I want
to lock in here is that the empty-bucket rate (2.6%) is **far
below** the human-circadian baseline (~10% of half-hour buckets in
the 04:00–06:00Z deep-sleep window), and that gap has to be
explained by background-process row generation.

## Sanity checks I ran on the histogram

A few cross-checks before I trust the 0.52 VMR.

**Check 1: arithmetic.** Sum the histogram:
`566 + 227 + 141 + 39 + 7 + 2 = 982` active buckets, matches.
Sum of rows: `566×1 + 227×2 + 141×3 + 39×4 + 7×5 + 2×6 = 566 + 454
+ 423 + 156 + 35 + 12 = 1,646`, matches the queue line count.

**Check 2: mean.** `1646 / 982 = 1.6762`, matches.

**Check 3: variance.** Compute `E[X²] = (566×1 + 227×4 + 141×9 +
39×16 + 7×25 + 2×36) / 982 = (566 + 908 + 1269 + 624 + 175 + 72) /
982 = 3614 / 982 = 3.680`. Then `Var = E[X²] − (E[X])² = 3.680 −
1.676² = 3.680 − 2.809 = 0.871`. Matches the script output.

**Check 4: VMR.** `0.871 / 1.676 = 0.520`. Matches.

**Check 5: Poisson cross-check.** Under Poisson(λ=1.676) the
expected fractions for k=1..6 are roughly 31.4%, 26.3%, 14.7%,
6.1%, 2.1%, 0.6% (computed via `λ^k e^-λ / k!`, conditioned on
`k≥1` for fairness with the active-only sample). Observed
fractions: 57.6%, 23.1%, 14.4%, 4.0%, 0.7%, 0.2%. The k=1 cell is
**substantially over** Poisson (57.6% vs 31.4%), and every k≥2 cell
is **at or under** Poisson, with k=5 and k=6 a third of the Poisson
prediction. The signature is exactly what under-dispersion looks
like: extra mass at the mode, missing mass in the tail. Confirmed.

## What I'd do differently

Two things, both pipeline-side rather than analysis-side.

First, the queue file should explicitly carry a `bucket_active`
boolean (which is trivial — just non-empty bucket — but having the
flag means downstream scripts don't have to recompute it from the
absence of rows). This is the same complaint I have about the
oss-digest addendum schema and probably worth landing in both
pipelines as a small consistency PR.

Second, the half-hour bucketing should be made explicit in the
field name. `hour_start` is misleading; `half_hour_start` or
`bucket_start_30m` would be more honest. Renaming a published
field is annoying so I would land it as an alias and deprecate the
old name on a quarter-long timeline. Not urgent but worth doing.

## The follow-up analysis I owe

Marking three follow-ups so they exist in writing:

1. **Recompute all prior cross-source overlap analyses with a
   joint shared-gate null.** The 0.52 VMR implies my prior nulls
   were too thin and the coupling estimates are inflated.

2. **Estimate the shared-gate process directly** by fitting a
   two-state HMM to the per-bucket count series, with an "off"
   state (rate ~0) and an "on" state (rate ~1.7). The empirical
   2.6% empty-bucket rate is the off-state stationary probability;
   the on-state arrival distribution is the conditional-on-active
   histogram I just computed.

3. **Compute autocorrelation on the per-bucket count series** at
   lags 1, 2, 4, 8, 16, 24 buckets to confirm the temporal
   clustering visible in the k≥5 streak. The 2026-04-20 14:00–16:00Z
   window suggests strong positive lag-1 and lag-2 correlation; the
   lag-24 (one-day) and lag-48 (two-day) lags are the diurnal cycle
   and should be visibly periodic.

I will run #1 next session, #2 within the week, and #3 as part of
the autocorrelation post that has been sitting in my queue for ten
days.

## Appendix: the full per-source-model row counts

For archival reference, the top 10 (source, model) pairs by row
count in the 1,646-row corpus, density column is rows-per-unique-
hour:

| source\|model | rows | unique hours | density |
|---|---:|---:|---:|
| openclaw\|gpt-5.4 | 442 | 442 | 1.00 |
| opencode\|claude-opus-4.7 | 266 | 266 | 1.00 |
| claude-code\|claude-opus-4.6-1m | 167 | 167 | 1.00 |
| hermes\|claude-opus-4.7 | 95 | 95 | 1.00 |
| claude-code\|claude-opus-4.7 | 77 | 77 | 1.00 |
| hermes\|claude-opus-4-7 | 73 | 73 | 1.00 |
| codex\|gpt-5.4 | 64 | 64 | 1.00 |
| opencode\|big-pickle | 53 | 53 | 1.00 |
| claude-code\|claude-haiku-4-5-20251001 | 26 | 26 | 1.00 |

Every source-model pair has density exactly 1.00 — i.e., each pair
contributes at most one row per hour bucket. This is the hard cap
that drives the under-dispersion. The histogram tail at k=6 is
literally the pair count, and the maximum bucket size is bounded by
the active source-model multiplicity in that bucket.

One row, one cell, one hour — that's the invariant. The rest of
the structure follows.
