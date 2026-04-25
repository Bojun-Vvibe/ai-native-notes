# Bucket-intensity histograms as a model-class fingerprint

*2026-04-25 — pew-insights v0.4.64, `bucket-intensity` subcommand (commit `62a98a7`)*

## The shape of an hour

Most token-accounting tools answer one question: "how many tokens did each
model burn in some window?" That number is useful for billing and useless for
almost anything else. It collapses a distribution into a sum, and the
distribution is where the interesting structure lives.

`pew-insights` ships a subcommand that refuses to do the collapse. It groups
events into `(model, UTC hour_start)` buckets and reports, per model: count of
buckets, min / p50 / p90 / p99 / max of `total_tokens` in those buckets, mean,
and a "spread" defined as `p99 / p50`. It also prints a six-band histogram —
`[1, 1k)`, `[1k, 10k)`, `[10k, 100k)`, `[100k, 1M)`, `[1M, 10M)`, `[10M, +inf)`
— of how many of each model's buckets fall into each magnitude band.

The histogram, more than any single percentile, ends up being a fingerprint
for *what kind of work the model is doing*. Two models with the same total
token count can have wildly different histograms, and the histogram tells you
whether the model is being used for sustained heavy lifting, for short
exploratory pokes, or for a flat steady stream of mid-size turns.

## Real numbers from one local queue

Snapshot run from `~/Projects/Bojun-Vvibe/pew-insights` at HEAD `a33ada4`,
binary at version 0.4.64, against the local `~/.config/pew/queue.jsonl`. The
floor was set with `--bucket-tokens-min 1000` to drop heartbeat noise; 95
buckets were dropped as below that floor.

```
as of: 2026-04-25T05:37:24.881Z   models: 14 (shown 14)
buckets: 1,145    tokens: 8,387,822,396    sort: tokens

per-model bucket-size summary
model               buckets  tokens          p50        p90         p99         spread
claude-opus-4.7     274      4,679,970,507   7,897,263  50,695,274  79,856,882  10.11
gpt-5.4             374      2,477,988,324   3,702,991  14,625,762  46,466,936  12.55
claude-opus-4.6.1m  167      1,108,978,665   3,244,687  16,660,674  51,557,347  15.89
claude-haiku-4.5    30       70,717,678      1,937,478  5,637,974   7,814,903   4.03
unknown             56       35,575,800      400,731    628,897     8,368,144   20.88
gpt-5               133      830,177         4,063      14,418      27,335      6.73
gemini-3-pro-preview 28      150,133         3,647      10,189      22,021      6.04
claude-sonnet-4.5   26       100,957         2,856      8,294       14,366      5.03
```

And the histogram for the same models:

```
model                [1,1k) [1k,10k) [10k,100k) [100k,1M) [1M,10M) [10M,+inf)
claude-opus-4.7      0      0        10         36        111      117
gpt-5.4              0      1        2          41        269      61
claude-opus-4.6.1m   0      1        10         33        88       35
claude-haiku-4.5     0      0        1          8         21       0
unknown              0      0        1          52        3        0
gpt-5                0      109      24         0         0        0
gemini-3-pro-preview 0      25       3          0         0        0
claude-sonnet-4.5    0      24       2          0         0        0
```

Eight models, eight different shapes. Let me read them.

## Three populations stand out

**The heavy-lift model (`claude-opus-4.7`).** 274 buckets total, but the
histogram is bottom-heavy in the wrong direction: 117 of those buckets — 43%
— are in the `[10M, +inf)` band. The next biggest band (`[1M, 10M)`) holds
another 111. Only 46 of its buckets are below 1M tokens. p50 sits at 7.9M,
p99 at 79.9M, max at 108M. Spread `p99/p50 = 10.11`. This is a model that
*never wakes up to do small work*. Every hour it's active it's processing
something the size of a paper or a long agent transcript. There's no
"exploratory poke" mode in evidence at all; the floor of activity is
already an order of magnitude above what other models call peak.

**The high-volume mid-band model (`gpt-5.4`).** 374 buckets — the most of
any model — but with a totally different shape. 269 of those (72%) sit in
`[1M, 10M)`. Only 61 cross 10M. p50 is 3.7M, p99 is 46M. Spread 12.55. So
gpt-5.4 has the broadest *footprint* (most active hours) but a much tighter
*per-hour mass*. It is the workhorse: showing up for many more hours, doing
mid-sized work each time, occasionally spiking. Its histogram is unimodal
and centered.

**The exploratory tier (`gpt-5`, `gemini-3-pro-preview`,
`claude-sonnet-4.5`).** All three live entirely below 100k tokens per
bucket. `gpt-5` has 109 buckets in `[1k, 10k)` and 24 in `[10k, 100k)` and
zero above. `claude-sonnet-4.5` is tinier still — 24 + 2 buckets, never
crossing 100k. These are not "small models doing what big models do
poorly"; they're showing a categorically different *use pattern*. p50 of
2.8k–4k tokens per hour bucket means a few short prompts at a time, then
silence until the next hour. They're being routed to (or chosen by humans
for) probe-shaped work.

The point is that the histogram column is the easiest read of all of these.
You don't have to compute anything. The column with the most counts tells
you the model's home magnitude band, and the spread of counts tells you how
disciplined that home is.

## Why this is more useful than rate-of-burn

A naive dashboard would compute `tokens / active_hours` and call that a
density metric. For our snapshot:

- claude-opus-4.7: 4.68B tokens / 274 buckets = 17.08M tok/bucket
- gpt-5.4: 2.48B / 374 = 6.63M tok/bucket
- claude-opus-4.6.1m: 1.11B / 167 = 6.64M tok/bucket

By this number gpt-5.4 and claude-opus-4.6.1m look identical. They are
not. claude-opus-4.6.1m has 35 buckets above 10M (21% of its buckets);
gpt-5.4 has 61 above 10M (16% of its buckets). The means coincide
because gpt-5.4 has many more mid-band buckets pulling the mean up,
while claude-opus-4.6.1m has fewer buckets but more of them are huge.
Same average, opposite character. The histogram catches it; the
average doesn't.

This is the same lesson hidden in every "summary statistic considered
harmful" essay, but it lands harder when you can see the histogram from
your own machine in one shell command. The mean is a smear. The
histogram is the fingerprint.

## The spread number is doing more work than it looks

`spread = p99 / p50` is a one-number summary of how heavy the tail is
relative to the center. For our data:

- claude-haiku-4.5: 4.03
- claude-sonnet-4.5: 5.03
- gemini-3-pro-preview: 6.04
- gpt-5: 6.73
- claude-opus-4.7: 10.11
- gpt-5.4: 12.55
- claude-opus-4.6.1m: 15.89
- unknown: 20.88

A spread under 5 means the busy hours are roughly the same size as the
median hour — flat workload, predictable. A spread above 10 means the
busy hours are an order of magnitude bigger than typical, so a quota
that's sized for the median will routinely be blown by the p99. The
"unknown" bucket having a spread of 20.88 is a useful red flag on its
own: it's the second-tallest tail in the dataset and we can't even
attribute it to a model. That's an instrumentation gap to chase down,
not a metrics curiosity.

For provisioning, spread is the number that should drive your headroom
calculation. A model with spread 4 can be sized to median + 50% and
rarely overflow. A model with spread 16 needs median + 1500% headroom
to absorb a p99 hour without backpressure, which usually means you do
not in fact size for p99 — you accept that p99 hours queue or shed
load, and you make that explicit instead of accidental.

## The histogram changes the conversation about cost

When a finance partner asks "why did our token spend jump?", the
default answer involves a per-model totals table and some
hand-waving. With histograms in hand, the answer becomes specific.
For example, if next week's snapshot shows claude-opus-4.7 going from
117 to 200 buckets in `[10M, +inf)` while keeping the same 274
total buckets, the story is "we're not using it for more hours; we're
using it for heavier hours." That's a workload story (someone is
feeding it bigger contexts), not a usage story (more sessions
started). The right intervention is different: the workload story
points at prompt design or context window discipline; the usage
story points at session lifecycle or routing.

Conversely, if next week claude-opus-4.7 shows 100 buckets in
`[100k, 1M)` and 117 still in `[10M, +inf)`, the story is "we
acquired a new use case for the model that lives in a different
magnitude band than the existing one." That's a population shift,
not a per-event shift, and it shows up as histogram bimodality —
two humps where there used to be one. You can't see bimodality in
a mean.

## Caveats baked into the subcommand

A few things the subcommand does *not* let you fool yourself about,
and which the help output (`pew-insights bucket-intensity --help`) is
explicit about:

- The bucket width is "whatever pew emits" — typically UTC hours,
  occasionally half-hour buckets in the model-tenure variant. So
  histogram counts are *bucket counts*, not request counts. A model
  with 274 buckets is not necessarily a model with 274 sessions;
  it's a model that touched 274 distinct hour slots.
- Percentiles are nearest-rank R-1 over per-bucket totals. With
  small bucket counts (claude-opus-4.6: 4 buckets) p99 just collapses
  onto max. The summary still prints, but you should not read it as
  a tail-risk estimate when the support is that small.
- The `--bucket-tokens-min` floor (1000 in our run) drops 95
  near-zero buckets that would otherwise live in `[1, 1k)` and
  pull the picture toward heartbeat traffic instead of work
  traffic. The exact floor is taste; the column header in our run
  shows zero entries in `[1, 1k)` for everyone, which is correct
  given we filtered there.
- "unknown" is a real model name in the input — it means the
  upstream tool didn't propagate the model field. Treating it as
  a model in the histogram is the right call, because at least
  then its share is visible.

These constraints are worth stating because the histogram looks
like a precise instrument and is, in fact, a *coarse* one. Six
bands over a 1-to-100M range is roughly one band per decade. That's
enough resolution to distinguish "exploratory" from "workhorse"
from "heavy-lift," but not enough to distinguish two models that
both live in `[1M, 10M)`. If you need that, you compute your own
deciles off the JSON output. The defaults are tuned for
fingerprinting, not for fine-grained tail analysis.

## What this means in practice

If you run `pew-insights bucket-intensity` once, the histogram tells
you which models in your fleet are heavy-lift, which are workhorse,
and which are exploratory. If you run it weekly, the *change* in
which band has the most counts per model tells you whether the
workload has migrated, whether new use cases have appeared, and
whether your spread numbers have crept upward (which means your
provisioning floor is now further from your peak).

For the snapshot above, the practical reading is:

- claude-opus-4.7 is a heavy-lift model with no exploratory tail to
  speak of and a tendency to live in the top band. Provisioning for
  it should treat 80M tokens/hour as a normal occurrence, not a
  surprise.
- gpt-5.4 is the workhorse, broadest footprint, tightest spread
  among the high-volume models. It's the model whose hourly cost is
  most predictable.
- claude-opus-4.6.1m is in the process of being aged out (its
  `lastSeen` from `model-tenure` is 2026-04-17, eight days before the
  snapshot) but its histogram still shows the heavy-lift profile,
  which is presumably why claude-opus-4.7 inherited the job.
- The exploratory tier (`gpt-5`, `gemini-3-pro-preview`,
  `claude-sonnet-4.5`) is doing categorically smaller work and should
  not be benchmarked against the heavy-lift tier on per-hour token
  metrics — that comparison is meaningless and will make the small
  models look "underutilized" when in fact they are doing exactly
  what they're routed to do.

The single most actionable thing to take from this: when someone
shows you a per-model totals table, ask for the histogram. The
totals will almost always agree with whatever story is being
told. The histogram will sometimes disagree, and when it does, it's
right.

## Reproduce

```
git -C ~/Projects/Bojun-Vvibe/pew-insights rev-parse --short HEAD
# a33ada4
node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js bucket-intensity \
  --bucket-tokens-min 1000
```

The `--bucket-tokens-min` flag was added in commit `62a98a7` and
shipped in v0.4.62. The subcommand itself landed in `7f6975a`. Both
sit in `src/bucketintensity.ts`; the histogram-band edges are
hardcoded to powers of ten because the goal of the subcommand is
fingerprinting, not parameter-tuning. If you want different bands,
fork it; the JSON output mode (`--format json`, available on the
same subcommand) gives you the per-bucket totals so you can build
whatever histogram you want downstream.
