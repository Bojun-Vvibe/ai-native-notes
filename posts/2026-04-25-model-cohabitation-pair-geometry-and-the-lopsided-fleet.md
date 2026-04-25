# Model cohabitation pair geometry and the lopsided fleet

The first time you look at a fleet that uses more than one model, you
reach instinctively for switching. You ask: when this session was on
A, what did it fall back to when A faltered? You compute A→B and B→A
inside a single session key, you draw a transition matrix, and you
declare yourself done with multi-model analysis. That ritual is
useful, but it answers a strictly *intra-session*, strictly
*sequential* question. There is a second question — older, simpler,
weirder — that the switching matrix cannot see at all: **which model
pairs are alive in the same wall-clock hour, regardless of whose
session they live inside?**

That is what cohabitation measures, and once you build it, the
geometry of the fleet stops looking like a graph of fallbacks and
starts looking like a sky map: a few bright binary stars, a halo of
loose pairings, and a long tail of accidental conjunctions where two
models happened to be lit during the same UTC hour for reasons that
have nothing to do with each other.

This note is about that geometry. The numbers come from
`pew-insights model-cohabitation` at v0.4.58 (commit `5f2ff24`),
running on the local queue at `~/.config/pew/queue.jsonl` as of
`2026-04-25T03:39:51.812Z`. Total tokens in scope:
8,325,823,145 across 877 hourly buckets, 15 distinct normalised
models, and 23 unordered model pairs that share at least one bucket.

## The cohabitation primitive

Cohabitation is built on a deliberately blunt primitive. For every
UTC `hour_start` in the queue, collect the set of distinct
normalised models that have positive token mass in that bucket.
That gives you, per hour, a small set — usually 1 model, sometimes
2, occasionally 3 or 4. Then, across all hours, count for each
unordered pair `(A, B)` how many buckets carry **both** A and B.
That is `coBuckets`. From there:

- `cohabIndex` = `coBuckets / |buckets(A) ∪ buckets(B)|`. Jaccard
  on bucket-presence sets, in `[0, 1]`. A pair scoring 0.7 means
  70% of all buckets that touch either model carry both.
- `coTokens` = sum across shared buckets of `min(tokens(A, h),
  tokens(B, h))`. A Jaccard-flavoured token weight: it stops the
  larger model from dominating every pair it touches and instead
  rewards pairs that are *balanced* in token mass per hour.
- `P(B|A)` = `coBuckets / |buckets(A)|`. Asymmetric. "When A is
  active, how often does B also show up in the same hour?"
- `P(A|B)` is the same with the roles reversed.

This is bucket-parallel, not session-sequential. Two models in the
same bucket might never have shared a session key, never have been
fallbacks for each other, never even have known the other was
running. They are cohabiting only in wall-clock time.

That sounds like a weak signal until you actually run it on a real
queue and look at the shape.

## What the fleet actually looks like

Here is the per-model presence summary from the v0.4.58 run. I am
keeping it raw because the shape is the point:

```
model                 tokens         buckets  cohabitants
--------------------  -------------  -------  -----------
claude-opus-4.7       4,625,901,757  271      7
gpt-5.4               2,470,012,313  370      8
claude-opus-4.6.1m    1,108,978,665  167      5
claude-haiku-4.5      70,717,678     30       4
unknown               35,575,800     56       2
claude-sonnet-4.6     12,601,545     9        3
gpt-5                 850,661        170      2
claude-opus-4.6       350,840        4        3
gpt-5.2               299,605        1        2
gemini-3-pro-preview  154,496        37       2
gpt-5.1               111,623        53       2
gpt-5-nano            109,646        1        2
claude-sonnet-4.5     105,382        37       3
claude-sonnet-4       53,062         26       1
gpt-4.1               72             1        0
```

Two things jump out before you even look at pairs.

First, the two largest models by tokens — `claude-opus-4.7` and
`gpt-5.4` — are not the two with the most live hours. `gpt-5.4`
has the widest bucket footprint (370 hours), but `claude-opus-4.7`,
with only 271 buckets, carries nearly twice as many tokens. That
already tells you the fleet is using opus-4.7 in a *bursty,
heavy-per-hour* mode and gpt-5.4 in a *broad, ambient* mode. Same
fleet, two completely different temporal personalities. Per-source
mix views can't see this because they collapse the time axis.

Second, look at the bottom of the table. `gpt-4.1` has `0`
cohabitants. It was alive for exactly one bucket and no other
model joined it that hour. That is a singleton in the truest
sense — a token-weighted ghost whose entire lifetime is one hour
of complete solitude. Cohabitation makes those visible without
having to invent a special "solo bucket" report.

## Pair geometry: one binary star and a long tail

The pair table at v0.4.58 looked like this (truncated to the top
half — the full report has 23 pairs):

```
modelA                modelB                coBuckets  cohabIndex  P(B|A)  P(A|B)
--------------------  --------------------  ---------  ----------  ------  ------
claude-opus-4.7       gpt-5.4               259        0.678       95.6%   70.0%
gpt-5.4               unknown               54         0.145       14.6%   96.4%
claude-opus-4.7       unknown               53         0.193       19.6%   94.6%
claude-haiku-4.5      claude-opus-4.6.1m    8          0.042       26.7%   4.8%
claude-opus-4.6.1m    gpt-5.4               7          0.013       4.2%    1.9%
claude-sonnet-4.5     gemini-3-pro-preview  5          0.072       13.5%   13.5%
claude-sonnet-4.6     gpt-5.4               4          0.011       44.4%   1.1%
claude-opus-4.7       claude-sonnet-4.6     3          0.011       1.1%    33.3%
```

The headline pair, `claude-opus-4.7 × gpt-5.4`, dwarfs everything
else. 259 shared buckets out of a possible union of 382 (271 + 370 −
259 = 382), giving a Jaccard index of **0.678**. To put that in
words: in two thirds of every hour where *either* of these two
models is alive, *both* are alive. That is not a fallback pattern.
That is not even a routing pattern. That is two models that have
been welded to the same workload by whoever is producing the
traffic — they run together, they retire together, they take their
breaks together.

Now look at the conditionals. `P(gpt-5.4 | claude-opus-4.7) = 95.6%`
means: in 259 of the 271 hours where opus-4.7 was lit, gpt-5.4 was
also lit. There is essentially no opus-4.7 hour without gpt-5.4 on
its shoulder. The reverse is weaker (`P(opus-4.7 | gpt-5.4) =
70.0%`) only because gpt-5.4 has a longer tail of solo hours
(370 − 259 = 111 hours where it ran alone). The asymmetry is the
shape: opus-4.7 is the dependent partner; gpt-5.4 is the carrier
that occasionally goes off doing its own ambient work.

After this binary star, the pair table falls off a cliff. The next
two pairs are both "X with `unknown`" — gpt-5.4 with unknown (54
buckets) and opus-4.7 with unknown (53 buckets). The Jaccard
indices are tiny (0.145 and 0.193) because `unknown` only has 56
buckets total. The conditionals tell the truth about what unknown
is: `P(gpt-5.4 | unknown) = 96.4%` and `P(opus-4.7 | unknown) =
94.6%`. Whatever produces "unknown" rows almost never appears
without one of the two heavyweight models also being alive — it is
parasitic on the binary star, not an independent third source.

After the unknown halo, every remaining pair has `coBuckets` in the
single digits. Eight pairs have `coBuckets = 1`. Those are
accidental conjunctions in the strictest sense: two models that
were each lit for very few hours total and happened to overlap once.

## The `--by-model` lens

v0.4.58 added `--by-model <name>`, a display filter that restricts
the pair report to pairs touching a named model. It is byte-identical
on the top-level numbers (same `totalBuckets`, same `multiModelBuckets`,
same global `models[]` list) — only the pair list shrinks. The point
is to ask "what does this one model cohabit with?" without having
to scan the global table.

Smoke-test from the v0.4.58 changelog, restricted to opus-4.7
(`--by-model claude-opus-4.7 --top 10`):

```
modelA              modelB             coBuckets  cohabIndex  P(B|A)  P(A|B)
------------------  -----------------  ---------  ----------  ------  ------
claude-opus-4.7     gpt-5.4            259        0.678       95.6%   70.0%
claude-opus-4.7     unknown            53         0.193       19.6%   94.6%
claude-opus-4.7     claude-sonnet-4.6  3          0.011       1.1%    33.3%
claude-haiku-4.5    claude-opus-4.7    2          0.007       6.7%    0.7%
claude-opus-4.7     gpt-5.2            1          0.004       0.4%    100.0%
claude-opus-4.6.1m  claude-opus-4.7    1          0.002       0.6%    0.4%
claude-opus-4.7     gpt-5-nano         1          0.004       0.4%    100.0%
```

`droppedByModelFilter: 16`. Sixteen of the 23 global pairs do not
touch opus-4.7 at all — most prominently the `gpt-5.4 × unknown`
pair, which is the second-largest pair globally but completely
invisible from opus-4.7's vantage. The lens makes a useful claim
explicit: opus-4.7 has *one* meaningful cohabitant (gpt-5.4) and
six accidental ones, and you do not need to eyeball the global
table to see it.

Run the same lens on gpt-5.4 (`--by-model gpt-5.4 --top 8`) and
the picture is still anchored by the same binary star but with a
different long tail:

```
modelA              modelB   coBuckets  cohabIndex  P(B|A)  P(A|B)
------------------  -------  ---------  ----------  ------  ------
claude-opus-4.7     gpt-5.4  259        0.678       95.6%   70.0%
gpt-5.4             unknown  54         0.145       14.6%   96.4%
claude-opus-4.6.1m  gpt-5.4  7          0.013       4.2%    1.9%
claude-sonnet-4.6   gpt-5.4  4          0.011       44.4%   1.1%
claude-haiku-4.5    gpt-5.4  2          0.005       6.7%    0.5%
gpt-5.2             gpt-5.4  1          0.003       100.0%  0.3%
gpt-5-nano          gpt-5.4  1          0.003       100.0%  0.3%
claude-opus-4.6     gpt-5.4  1          0.003       25.0%   0.3%
```

gpt-5.4 cohabits with eight other models (one more than opus-4.7)
because of its broader bucket footprint. The pairs `gpt-5.2 ×
gpt-5.4` and `gpt-5-nano × gpt-5.4` show `P(A|gpt-5.4) = 0.3%` but
`P(gpt-5.4 | A) = 100.0%` — gpt-5.2 and gpt-5-nano were each alive
for a single bucket, and gpt-5.4 happened to be lit that same hour.
That is the canonical shape of an accidental conjunction: one model
with vanishing presence, one heavyweight that is alive most hours
anyway, and a 100% conditional that means almost nothing.

## What this is good for

Cohabitation is not a substitute for switching analysis, and it is
not a substitute for per-source mix or per-source entropy. It
answers a question those reports cannot:

- **Workload coupling.** A high-Jaccard pair in the cohabitation
  table is a strong hint that two producers — or one producer with
  two routing rules — are riding the same temporal envelope. They
  start together, they idle together, they peak together. That
  is the kind of coupling you would not see if you only looked at
  per-source token shares, because both producers might use both
  models at different times within the same bucket.
- **Drift detection.** If next week's run shows the binary star
  pair has dropped from `cohabIndex = 0.678` to `0.45`, something
  has decoupled the two models temporally — maybe one was retired
  for a class of work, maybe a router started preferring one in
  off-hours. The Jaccard number is robust to overall volume
  changes (it normalises by the union), so it is a good "shape
  has changed" alarm even when totals shift.
- **Solo hours.** `|buckets(A)| − Σ_B coBuckets(A, B)` (with
  appropriate inclusion-exclusion if you want to count multi-overlap
  buckets correctly) is the count of hours where A ran alone. That
  is your floor for "what is this model used for when nothing else
  is competing for the same hour" — useful when reasoning about
  fallbacks, scheduled jobs, or background work.

## What this is *not* good for

Cohabitation has three failure modes worth being explicit about.

First, **bucket size matters.** UTC hour buckets are coarse. If
your fleet has a model that fires for 30 seconds every hour and a
model that fires for 50 minutes every hour, they will register as
fully cohabiting (`cohabIndex ≈ 1.0`) even though they never run
at the same wall-clock minute. If you want minute-level
cohabitation, you need a different bucketing — and the resulting
graph will be sparser and noisier.

Second, **the `unknown` model is a sink.** Any row with a
normalised model of `unknown` represents producer output we could
not classify, and it tends to cohabit with whatever is busy at
that hour because it inherits the temporal envelope of the busy
producer. The 54-bucket and 53-bucket pairs against `unknown`
above are not really telling you about cohabitation — they are
telling you that the unknown rows are mostly being produced
during the binary-star hours. Treat unknown pairs with extra
suspicion.

Third, **small-N pairs are not pairs.** Eight of the 23 pairs in
the v0.4.58 run have `coBuckets = 1`. Their Jaccard indices
range from 0.002 to 0.011. Their conditionals include several
100% values that look impressive but are arithmetic artefacts of
`1/1 = 1`. The `--min-co-buckets` flag exists for exactly this
reason: in any analytical pass that is not "what is the total
shape of the long tail", set it to 3 or higher and let the
`droppedBelowMinCoBuckets` counter remind you what was hidden.

## The shape, in one sentence

After 877 buckets, 8.3 billion tokens, and 15 normalised models,
this fleet has the geometry of a binary star in a thin halo. One
pair (`claude-opus-4.7 × gpt-5.4`) accounts for almost all of the
multi-model bucket overlap. A second tier of "X × unknown" pairs
shows that the unclassified rows are parasitic on that binary
star. Everything below is single-digit `coBuckets`, single-bucket
flukes, and a `gpt-4.1` ghost that ran exactly once and never met
anyone. Whatever this fleet is doing, the temporal coupling
between its two top models is the single most informative number
on the screen — and it took building a bucket-parallel,
session-blind, switching-blind primitive to surface it.

That is the case for keeping cohabitation in the toolbox even when
you already have switching, mix-entropy, and per-source share.
The other reports tell you *who used what*. Cohabitation tells
you *who was alive at the same time as whom*. Different question,
different answer, and on this queue at least, a much sharper
shape.
