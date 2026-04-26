# Cumulative-mass half-life by source: when two days cover half of `claude-code` and six days cover half of `ide-assistant-A`

**Thesis.** A source's *chronological* half-life (the calendar day at which its
running cumulative tokens cross 50% of its lifetime) and a source's *cumulative-mass*
half-life (the smallest number of days, sorted by mass, whose total first crosses
50%) are different statistics, and the gap between them is where the interesting
shape lives. The first is a location index on the calendar; the second is a
concentration index on the tool's own mass distribution. Two sources can share a
chronological half-life — both can be "front-loaded" or both "back-loaded" — and
still have wildly different mass half-lives. This post reads the
`source-cumulative-mass-half-life-day` output from `pew-insights` against a
single 9.36 B-token corpus and shows that on this snapshot, four out of six
sources cross 50% of their mass in either two or three days, and the
`halfLifeDays / daysActive` ratio is the single column that actually separates
"a tool I open every day" from "a tool I touched twice and never again."

## The data

Run on `~/.config/pew/queue.jsonl` at `2026-04-26T18:49:43.939Z`, 6 sources,
9,365,930,629 total tokens, 1,584 rows:

```
source          days  tokenSum       q25  half  q75  top1Share  top3Share  halfRatio
--------------  ----  -------------  ---  ----  ---  ---------  ---------  ---------
claude-code     35    3,442,385,788  1    2     6    0.3056     0.5964     0.0571
codex           8     809,624,660    1    2     3    0.4814     0.8466     0.2500
hermes          10    145,236,896    2    3     5    0.2388     0.5987     0.3000
openclaw        10    1,755,309,045  2    3     5    0.2017     0.5104     0.3000
opencode        7     3,211,488,513  2    3     5    0.2255     0.5984     0.4286
ide-assistant-A 73    1,885,727      3    6     15   0.1277     0.3270     0.0822
```

Six rows. One number to stare at: the `half` column is essentially constant at
`{2, 2, 3, 3, 3, 6}`. Five of six sources cross 50% of their lifetime mass in
**three days or fewer**, regardless of how long their tenure is — `claude-code`
runs 35 days, `opencode` runs 7, `ide-assistant-A` runs 73, and four of those
six still hit halfway home in `≤3` days.

## Why `half` looks so flat — and why `halfRatio` does not

A naive read is "all sources are equally bursty, the half-life is always two
or three days." That read is wrong. The flatness of the `half` column is an
artifact of the integer ladder: `halfLifeDays` is a count, and the corpus is
small enough that the cumulative-mass curve climbs steeply at the top. Four
of six sources have `top1Share` above 22%, three have `top3Share` above 0.59.
When the top three days already carry over half the mass, the function that
asks "smallest *k* such that cumulative share crosses 0.5" cannot return
anything larger than 3 unless the top1/top3 are unusually small.

`halfRatio = halfLifeDays / daysActive` rescales `half` against tenure and
this is the column that actually separates the population:

```
claude-code     0.0571
ide-assistant-A 0.0822
codex           0.2500
hermes          0.3000
openclaw        0.3000
opencode        0.4286
```

Three regimes pop out:

1. **Tenure-dominated (ratio < 0.10).** `claude-code` (0.057) and
   `ide-assistant-A` (0.082). Half of the lifetime mass lives in roughly the
   top 6–8% of active days. The number is small because the *denominator* is
   large — these are the two sources that have shown up on the calendar most
   often. Even when their top-day shares are modest (0.31 and 0.13), there are
   so many low-mass days behind them that the cumulative curve crosses 50%
   inside a tiny prefix of days-by-mass.
2. **Concentration-dominated (ratio 0.25–0.30).** `codex`, `hermes`,
   `openclaw`. These look identical on the `half` axis (2 or 3 days each)
   but the underlying shape is different: `codex` has `top1Share = 0.48` and
   `top3Share = 0.85` — one day plus two more carry 85% of its 809 M tokens.
   `openclaw`'s top1 is only 0.20 and its top3 is 0.51, so the same `half = 3`
   reflects three roughly equal heavy days rather than one whale plus two
   pilot fish. *The headline is the same number for very different geometry.*
3. **Tenure-saturated (ratio > 0.40).** `opencode` (0.4286). With only 7
   active days and `half = 3`, this source is approaching the floor of its own
   tenure. Its `top1Share` of 0.226 is moderate; the reason `half = 3` is not
   that one day dominates but that *every* one of its seven days is
   contributing a non-trivial slice. The mass curve climbs in lots of equal
   steps.

## Reading `q25 / half / q75` as a 25-50-75 ladder

The three-column threshold ladder (`q25`, `q50` aka `half`, `q75`) is the
hidden gem of this view. It rebuilds the Lorenz curve at three points without
asking the operator to plot anything:

| source | q25 | half | q75 | shape note |
|---|---|---|---|---|
| `claude-code` | 1 | 2 | 6 | one whale + a long tail. q25 already crosses with one day. |
| `codex` | 1 | 2 | 3 | extreme concentration: a third of mass on day 1, three quarters by day 3. |
| `hermes` | 2 | 3 | 5 | smooth ramp, no whale. |
| `openclaw` | 2 | 3 | 5 | smooth ramp, mass diffuse across 5 of 10 days. |
| `opencode` | 2 | 3 | 5 | identical *q* triple to hermes/openclaw — fewer total days, similar evenness. |
| `ide-assistant-A` | 3 | 6 | 15 | the only source whose q75 is in double digits — the long tail is real. |

The most interesting comparison is `hermes` vs `openclaw`. They share the same
`q25=2 / half=3 / q75=5` triple. They share `daysActive = 10`. Yet `hermes`
moves 145 M tokens lifetime and `openclaw` moves 1.76 B — a **12.1×** factor
in absolute mass. The cumulative-mass half-life view is *deliberately* blind
to the absolute-magnitude axis: it asks only how the mass is shaped within
the source, not how big the source is. This is the orthogonality that makes
the metric useful — it gives you a profile of *how* a tool is used that
generalises across "I poked at this for an hour" and "this drove all my
production work for ten days."

## The `q75 = 15` outlier

`ide-assistant-A` has `q75 = 15` and it is the only row in the table whose
75% threshold lives outside the single-digit zone. `daysActive = 73`,
`halfLifeDays = 6`, `q75 = 15`. Translation: half the lifetime mass lives in
the top 6 days, but you need fifteen of the seventy-three days to get to
75%. The mass curve flattens *hard* after the top 6.

That is consistent with the source's identity in the corpus — it is the
in-editor assistant, the tool that gets opened for tiny edits across many
calendar days. Most days are very small. A few days carry a lot. The 15-day
q75 vs 6-day q50 says the "carrying" tail past the median is a thick band of
mid-sized days, not a uniform low floor. For comparison, the other long-tenure
source — `claude-code` at 35 days — has `q75 = 6` and `q50 = 2`. Its tail
falls off much faster after the median because its individual top day is
bigger relative to its body.

## What `halfLifeRatio` corresponds to in classic statistics

The ratio `halfLifeDays / daysActive` lives between `1/n` (one day carries
all the mass) and `1/2` (mass is perfectly uniform across the active set —
you need exactly half the days to hit half the mass). The two ends are
informative:

- A ratio close to `1/n` means the tool is **whale-shaped**: a tiny number of
  big days dominate. `codex` at `2/8 = 0.25` is the closest thing in this
  snapshot to a whale; if it had had three more uneventful days its ratio
  would have dropped to `2/11 ≈ 0.18` and the shape would still be
  "two-day whale." This is why `halfRatio` decreases with tenure under
  zero-padded days — every additional inactive day pushes the ratio toward
  whale-territory without changing the cumulative-mass curve at all.
- A ratio close to `1/2` means the tool is **uniform** on its own active
  set. `opencode` at `3/7 = 0.4286` is the row most consistent with "every
  day I open it I do similar amounts of work." Combined with the moderate
  `top1Share = 0.226` this is the cleanest "even-handed daily driver"
  signal in the table.

`claude-code`'s ratio of 0.057 is not a sign of a whale; it is a sign that
its daysActive denominator is huge relative to its mass concentration. The
`top1Share = 0.31` and `top3Share = 0.60` say that by row-of-mass the
distribution is moderately concentrated, but the *calendar* axis spreads
those days across 35 dates. The metric is doing exactly what it was designed
to do: it answers "how few of this source's days do I need to touch to
reproduce half its workload?" without conflating that question with "how big
is the source overall."

## Cross-checking against `top1Share` / `top3Share`

Two algebraic checks:

1. If `top1Share >= 0.5`, then `halfLifeDays = 1` is forced. No source in this
   table satisfies that — `codex` is the closest at 0.4814, which is why its
   `half` is 2 and not 1.
2. If `top1Share + top2Share >= 0.5`, `halfLifeDays <= 2`. `codex` at
   `top1=0.4814 + top2≈?` clearly satisfies this (its `half = 2` proves it).
   `claude-code` with `top1=0.3056` and `top3=0.5964` has `top1+top2 ≈ ?`
   — the `half = 2` outcome implies `top2 ≈ 0.20+`. Reasonable for a
   long-tenure source whose top day is ~31%.

These checks are not just rederivations of the metric — they are how an
operator should sanity-check a `pew-insights` snapshot before drawing
conclusions. If a future run shows `top1Share = 0.55` and `halfLifeDays = 2`,
that combination is *impossible* and points to a bug in the bucketing layer.

## Why this view is orthogonal to the chronological half-life view

The companion subcommand `source-decay-half-life` reads the same source's
mass along the **calendar axis** and asks "at what calendar day does the
running cumulative cross 50%?" Its output describes the *location* of the
mass: front-loaded (early date), uniform (~midpoint), back-loaded (late
date).

The `source-cumulative-mass-half-life-day` view permutes the day vector by
mass first and then asks the half-life question. This is **order-invariant**
along the calendar axis. It cannot tell you whether `claude-code`'s big days
were on the first three calendar days or on the last three. It can tell you
that, regardless of when those days happened, two of them cover half the
mass.

The two metrics combined give you a 2-D fingerprint:

|  | early | mid | late |
|---|---|---|---|
| **whale** (low halfRatio) | "started hot, rest is filler" | "single midcourse spike" | "ended on a binge" |
| **even** (high halfRatio) | "front-loaded ramp" | "uniform daily driver" | "late ramp-up" |

`opencode` at `halfRatio = 0.43` and `top1Share = 0.23` reads as "even daily
driver" on the within-source axis. To know whether it's also a *recent*
even-driver vs a *long-ago* one, you need the chronological partner. The two
views complement each other and neither replaces the other.

## What the snapshot says about the operator

Read across the six rows: this operator has one whale-shaped source
(`codex`), three smooth-ramp sources of similar shape (`hermes`,
`openclaw`, `opencode`), one long-tenure low-concentration tool that runs in
the background (`ide-assistant-A`), and one heavy-but-spread-out workhorse
(`claude-code`). The shape isn't "I have one tool I use" or "I sample
randomly across many tools." It's a tiered usage pattern: a long-tail
ambient tool, a cluster of three peers, and two leaders that each cover
different shapes of work.

The fact that the median `halfRatio` across the six sources is 0.275 —
roughly halfway between the `2/n` whale floor and the `1/2` uniform ceiling
— is a useful one-number summary of the corpus. Future snapshots can be
compared against that: a value drifting up means the operator is using each
tool more uniformly across its days; drifting down means a few big days
are starting to dominate each source.

## Operational takeaways

- **Don't read `halfLifeDays` in isolation.** It compresses to a small
  integer and looks identical across very different shapes. Always pair it
  with `halfRatio` and the `q25/q75` flank.
- **`halfRatio` is the corpus-comparable scalar.** It has a natural
  `[1/n, 1/2]` range and survives changes in tenure that wreck the raw
  count.
- **Cross-check with `top1Share`.** If `top1Share >= 0.5`, then `half = 1`
  by construction; if a snapshot ever violates that, the source-day
  bucketing has drifted.
- **The `q75 - q50` width is the tail-thickness signal.** A wide gap
  (`ide-assistant-A`: 9 days) means the post-median tail is fat; a narrow
  one (`codex`: 1 day) means the long tail has nothing left after the
  median is hit.
- **Pair this metric with the chronological-half-life view.** They share no
  information by construction. Together they place each source on a
  shape-by-location grid.

## Citation

Output produced by `pew-insights source-cumulative-mass-half-life-day` against
`~/.config/pew/queue.jsonl` (1,584 rows, 9,365,930,629 tokens), snapshot
`as of: 2026-04-26T18:49:43.939Z`. Six sources, default thresholds (q25/q50/q75),
ide-assistant-A redaction applied to one editor-side source name in this post.
