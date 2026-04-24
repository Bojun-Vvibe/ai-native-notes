---
title: "Coefficient of variation as a workload classifier (and why your rate limiter cares)"
date: 2026-04-25
tags: [observability, statistics, rate-limiting, telemetry, agents]
---

# Coefficient of variation as a workload classifier (and why your rate limiter cares)

If you operate an AI workload at any scale you eventually run into a question
that sounds simple and isn't: **is this workload steady or bursty?** The answer
matters because every downstream system you care about — rate limiters, capacity
planners, autoscalers, on-call alert thresholds, billing forecasts — quietly
assumes one or the other and silently degrades when the assumption is wrong. A
rate limiter sized for a steady drip will throttle a bursty workload; a capacity
plan built for a bursty workload will leave a steady one paying for headroom it
will never use; an alert threshold tuned for a uniform week will page constantly
during a peak hour it should treat as normal. The trouble is most teams don't
have a single scalar that answers the steady-vs-bursty question. They have
dashboards full of `mean tokens/hour`, `peak tokens/hour`, `p95`, and a vague
sense that "Mondays are heavy."

There is a single scalar that answers it cleanly, it has been sitting in the
statistics literature for a century, and it generalizes to every dimension of
agent telemetry you care about. It is the **coefficient of variation**, defined
as `cv = stddev / mean`, computed over whatever bucketed time series you have —
hourly tokens, per-minute requests, per-day costs, doesn't matter. This post is
about why `cv` is the right scalar to reach for, what threshold values actually
mean operationally, where it breaks, and how to wire it into a workload classifier
that drives real decisions.

## The numbers that motivated this post

Today's release of `pew-insights` v0.4.45 (commit `21f0d77`, "feat: burstiness
subcommand") added a `burstiness` subcommand that reports `cv` per
`(model | source)` over hourly token buckets. The very first live-smoke run
against my local pew queue gave numbers worth staring at:

```
global cv: 1.919    global active hrs: 868    global mean/hr: 9,465,175    global max/hr: 126,620,962

claude-opus-4.7       cv 1.220    burst 14.57×
gpt-5.4               cv 1.365    burst 16.89×
claude-opus-4.6.1m    cv 1.412    burst 17.25×
claude-haiku-4.5      cv 0.840    burst  3.97×
unknown               cv 1.837    burst 20.86×
gpt-5                 cv 1.180    burst 12.56×
```

A global `cv` of `1.919` means the standard deviation across the 868 active
hour buckets is roughly twice the mean. That is not a steady workload. It is
emphatically bursty, and bursty in a specific shape: a few peak hours at
~108M tokens dwarf a long tail of normal hours around ~7M. The peak/median
ratio (`burst = max / p50`) is `14.57×` for `claude-opus-4.7` and `20.86×`
for `unknown` background traffic — meaning a single peak hour is 15-20× the
median hour for those groups.

If you sized a rate limiter against the mean (~9.5M tokens/hour globally,
~17M for `claude-opus-4.7`), you would clip every peak hour. If you sized
it against the max (~108M for `claude-opus-4.7`), you would over-provision
by a factor of `burst`. Neither is right. The right answer is to know `cv`
first, then pick a sizing strategy appropriate to the regime — and the
regimes are well-defined.

## What `cv` actually tells you

`cv` is dimensionless. That is the whole point. `mean` and `stddev` both
have units (tokens/hour, requests/second, dollars/day) and both grow with
your workload, so neither is comparable across models, customers, or time
windows. `cv` divides them out. A model handling 4.5B tokens and a model
handling 850K tokens can have the same `cv` and that means they have the
same *shape* of hourly variation — one is just larger.

The interpretive ladder, give or take:

- **`cv ≈ 0`**: deterministic. Every bucket is the same. In agent
  telemetry this almost never happens organically; when you see it, it
  means n=1 (a single bucket where stddev trivially equals zero) or
  someone wired up a synthetic load generator.
- **`cv` in `[0.1, 0.5]`**: smooth, lightly varying. A well-tuned
  background scrape, a steady cron, a system whose load follows a
  predictable diurnal curve with no spikes.
- **`cv` in `[0.5, 1.0]`**: noisy but bounded. Roughly Poisson-shaped:
  random arrivals, no clustering, stddev of order the mean. A typical
  customer-facing service hits this range during business hours.
- **`cv` ≈ `1.0`**: classical Poisson reference (for a Poisson process
  with rate `λ`, mean and variance both equal `λ` so `cv = 1/√λ` for
  the count, but for *bucket sums* of independent arrivals the ratio
  converges toward 1 in many regimes — the exact theory is messier
  than the heuristic).
- **`cv` in `[1.0, 2.0]`**: heavy-tailed, clustered. Some hours dwarf
  others. Most LLM workloads I've measured live here. The numbers
  above (`1.22` to `1.84` per model, `1.92` global) are textbook
  examples.
- **`cv` > `2.0`**: pathological. A handful of huge buckets and a sea of
  near-zero ones. If you see this in a customer-facing system, look for
  a batch job that misclassified itself as interactive traffic.

The reason this ladder is useful is that it suggests **different sizing
strategies for different ranges**, and you can pick the strategy
automatically once `cv` is in your telemetry pipeline.

## Sizing strategies by `cv` regime

### Regime 1: `cv < 0.5` — size to mean + small headroom

If your workload is smooth, the mean is a good predictor of the next
bucket. Size capacity to roughly `mean × 1.2` and you'll catch almost
every bucket without over-provisioning. Alert thresholds can be tight:
`mean + 3σ` is a sensible page-out boundary because three-sigma events
are genuinely rare in a smooth series. Forecasting is easy: a moving
average works.

### Regime 2: `cv` in `[0.5, 1.0]` — size to a high quantile

Poisson-ish workloads have a non-trivial tail but no extreme spikes.
Size to `p95` or `p99` of the bucket distribution, not to the mean.
Alert on quantile excursions, not on absolute counts. Forecasting needs
an EWMA with bounded variance, not a flat moving average.

### Regime 3: `cv` in `[1.0, 2.0]` — size to expected peak, plan for retry

This is the regime where most agent traffic actually lives. The mean is
nearly useless as a sizing input because half your buckets are below it
and a few are 10-20× above it. The right move is twofold: size to the
expected peak (use `p99` or `max` over a representative window) **and**
build a retry/queue layer that can absorb bursts beyond the sized peak
without dropping requests. The `burst = max / p50` ratio tells you how
deep the queue needs to be.

For `claude-opus-4.7` with `cv 1.22`, `p50 7.4M tokens/hour`, `max 108M
tokens/hour`, the `burst 14.57×` says: if you size to `p50` you'll need
a queue that can hold ~14× a median hour's worth of work. If you size to
`p95` (`60M`), you still need a queue that can hold ~1.8× a `p95` hour
during the rare `max` event.

### Regime 4: `cv > 2.0` — segment first, size second

If `cv` is above 2 you almost certainly have two populations mixed
together. Don't size the combined workload; split it. Look for the
batch job, the misclassified background scan, the cron that fires once
a week. Once you separate them, each subpopulation will have a `cv` in
one of the lower regimes and you can size each appropriately.

## Where `cv` breaks (and how the v0.4.46 refinement handles it)

`cv` has two well-known failure modes and both showed up in the same
live-smoke run.

**Failure mode 1: n=1.** A group with a single active bucket has stddev
exactly zero, so `cv = 0`. The burstiness output above shows three of
these: `gpt-5.2`, `gpt-5-nano`, `gpt-4.1`, each at `cv 0.000`. Sorting
by `cv ascending` would put them at the top of a "steady workloads"
list — but they aren't steady, they are unobserved. They are workloads
we have one data point for, and one data point is consistent with any
shape.

The v0.4.46 release (commit `5c14499`, "chore: bump v0.4.45 -> v0.4.46";
feature commit `17b1e6f`, "feat(burstiness): add --min-cv <x>
refinement") added two display-only floors: `--min-active-hours <n>`
(borrowed from the existing weekday-share / peak-hour-share pattern)
and `--min-cv <x>` (new). The combined `--min-active-hours 5
--min-cv 1.0` filter shrinks the kept-model list from 15 to 9 and
keeps only models that are both well-observed (≥5 active hours) and
genuinely bursty (stddev ≥ mean). Importantly the global denominator
(`8,221,565,624` total tokens, `1.917` global cv) does *not* shift —
the floor is display-only, not population-truncating. That separation
matters operationally: you want to filter what you *show* an operator
without changing the *truth* you are computing against.

**Failure mode 2: heavy bimodal mixture.** A group that is genuinely
two populations — say, a small steady customer-facing flow plus a
large weekly batch — will compute a high `cv` that doesn't match
either subpopulation's true shape. The fix is the same as Regime 4
above: segment, then re-measure. In the live-smoke output, `unknown`
at `cv 1.837` is almost certainly this case — "unknown" is the catchall
bucket for traffic without a clean model attribution, so by definition
it mixes anything that fell out of the producer/model resolver.

## Wiring `cv` into an agent telemetry pipeline

The minimum viable pipeline is:

1. **Bucket your raw event stream** by a fixed time unit (hour is a
   good default; minute if you have enough volume; day if you don't).
   Sum the metric you care about per bucket per group dimension.
2. **Compute mean, popstd, p50, p95, max per group** over the active
   buckets — buckets with zero are excluded so a quiet group doesn't
   look smoother than it is. (This is the same `active-hours-only`
   convention `pew-insights` uses; see `peak-hour-share` and
   `weekday-share` for prior art.)
3. **Compute `cv = popstd / mean` and `burst = max / p50`** per group,
   plus the same scalars over the global flattened series.
4. **Apply two display floors**: minimum active buckets (drop
   under-observed groups from rankings) and minimum `cv` (drop
   genuinely steady groups from spikiness rankings).
5. **Classify each group into a regime** by `cv` band and emit the
   regime label alongside the raw scalars. Downstream consumers
   (rate-limiter sizer, alert threshold picker, capacity planner)
   read the regime label, not the raw `cv`, so the policy is
   centralized.

Step 5 is the one most teams skip and it's the one that pays back the
fastest. Once your dashboard says `claude-opus-4.7: regime=heavy-tail
(cv 1.22, burst 14.6×)` instead of just `cv 1.22`, the on-call engineer
who has never opened the burstiness docs immediately knows what to do:
size to peak, build a queue, expect a 15× ratio between peak and
median.

## Why the `active-buckets-only` denominator matters

A subtle and important choice in the burstiness implementation is that
mean and stddev are computed over *active* hour buckets, not over the
full wall-clock window. If a group has 100 active hours over a 7-day
(168-hour) window, the inactive 68 hours don't dilute the variance.

The alternative — wall-clock denominator — would let a group look
"smoother" simply by being inactive more often, which is the wrong
shape. A workload that fires once a week for 60 minutes at full blast
and is silent the rest of the time is *extremely* bursty when active;
diluting it with the 167 silent hours would compute a tiny `cv` and
hide the spike entirely.

The cost of the active-bucket convention is that `cv` is now sensitive
to your bucket-membership rule. A group that fires for 30 seconds in
each hour over a long window will count those hours as "active" and
report high `cv` if the 30-second bursts vary in size. That is usually
what you want, but it does mean the metric is not directly comparable
across pipelines that bucket differently. If you publish `cv` to other
teams, publish the bucket size and the active-bucket rule with it.

## What `cv` does *not* tell you

Three things `cv` is silent on, and you need other metrics to answer:

- **Which buckets are the spikes.** `cv` is a scalar over a series; it
  doesn't say whether the spikes are at 9am or 3pm or scattered. Pair
  it with `peak-hour-share` (which hour holds the most mass) and
  `weekday-share` (which day holds the most mass) for the *when*.
- **Whether spikes are clustered or independent.** A cv of 1.5 from
  one giant week and a cv of 1.5 from many small weekly spikes look
  identical scalar-wise. Pair it with `streaks` / `gaps` for
  contiguity structure.
- **Whether spikes correlate across groups.** If `claude-opus-4.7` and
  `gpt-5.4` both spike at the same hour, that is a different operational
  fact (probably a single underlying user pattern) than if they spike
  at different hours (independent workloads sharing infrastructure).
  `cv` per group can't tell you; you need a cross-group covariance or
  at least a hour-aligned overlay plot.

## Closing

`cv` is the cheapest single number you can add to an agent telemetry
pipeline that meaningfully changes downstream decisions. It costs one
pass over your bucketed series. It is dimensionless so it compares
across groups and across time. It maps cleanly to a small set of
operational regimes that suggest concrete sizing and alerting
strategies. And it has two well-understood failure modes — n=1 and
bimodal mixtures — both of which are addressable with display floors
and segmentation rather than statistical wizardry.

The numbers from today's `pew-insights` v0.4.45 / v0.4.46 release
(commits `21f0d77` and `17b1e6f`) are a textbook case: a global cv
near 2, per-model cv in the 1.2-1.8 range, peak/median ratios of
12-20×. That is heavy-tailed agent traffic, and it is the regime
nearly every team running a real LLM workload is actually in. Knowing
the number doesn't make the bursts go away — but it does keep you
from sizing your rate limiter against a mean that lies.
