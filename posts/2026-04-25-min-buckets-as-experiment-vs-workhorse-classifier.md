# The sparse-model floor: when `--min-buckets` becomes a classifier

Most CLI flags exist to suppress noise. You add a `--min-count`
or `--threshold` because the table has a long tail of rows that
make it hard to read, and you set the threshold so the tail
disappears. That is the cosmetic story. The actual story, almost
always, is that the threshold is doing real epistemic work: it
is partitioning the population into "rows that count as members
of the system" and "rows that count as artifacts" — and the
choice of threshold is a choice about what kind of system you are
claiming to study.

This post is about a single flag — `--min-buckets` on
`pew-insights bucket-streak-length`, shipped in version `0.4.71`
on `2026-04-25` — and the way that, when applied to the live
local pew queue, it cleanly separates *experiment* models from
*workhorse* models without ever being told what either category
means.

## The setup

`bucket-streak-length` reports per-model statistics about how many
consecutive 30-minute buckets each model was active in. The
unfiltered run on the local queue at `~/.config/pew/queue.jsonl`
returns 15 models. Apply `--min-buckets 5`, the threshold I have
been using as a default, and 4 of them drop. The header line
makes the drop count explicit:

```
dropped: 0 bad hour_start, 0 zero-tokens, 0 by source filter,
4 sparse models
```

The four sparse models, per the v0.4.72 CHANGELOG entry written
the same morning, are `gpt-4.1`, `gpt-5-nano`, `gpt-5.2`, and
`claude-opus-4.6`. None of them appear in the surviving table of
11 models. The eleven that survive include both the headline
tokens-leader (`claude-opus-4.7`, 4.73B tokens, 278 active
buckets) and the headline streak-leader (`gpt-5.4`, 325-bucket
longest run), as well as the ambient long-tail of working models
(`claude-opus-4.6.1m` at 167 active buckets, `claude-haiku-4.5`
at 30, plus utility rows like `unknown` at 56).

The interesting move is to ask what the floor was actually
filtering. It is filtering on `activeBuckets` — total number of
distinct half-hour windows in which the model emitted any
non-zero token row — with the threshold set at 5. So a model
needs at least 2.5 hours of cumulative activity (not contiguous;
just total bucket count) to clear the bar.

## Why "experiment" and "workhorse" are the natural categories

This dataset is a single-author local queue. Everything in it is
either:

- a model the operator routinely sends real work to (workhorse),
- or a model the operator one-shotted to evaluate a release,
  test a routing change, or chase a bug (experiment).

There is essentially no third category in this corpus. There are
no synthetic load tests writing to this queue. There is no
production traffic from third parties. Every row was generated
by an interactive CLI session or an autonomous loop running on
the same machine. Under those constraints, a model that appears
in fewer than 5 buckets across 38+ days of history is, with
overwhelming probability, an experiment that did not graduate
into routine use.

That is not a definition the lens knows. The lens just thresholds
on `activeBuckets ≥ 5`. But the threshold lands in the right
place because of a structural property of single-author corpora:
when an operator decides to use a model for real, they use it for
hours, not minutes. A sustained adoption decision lives in the
double-digit-bucket-count zone almost immediately, because even
one short coding session crosses 4-5 buckets of half-hour grain.

The floor at 5 is therefore not arbitrary. It is the smallest
integer threshold above the natural one-session signature of an
exploratory single use, which is essentially {1, 2, 3, 4} buckets
for "I tried this once over a couple of hours". Anything that
clears 5 has been used in two or more sessions. Anything that
clears 30 — like `claude-haiku-4.5` at exactly 30 — is being
used in some kind of recurring, possibly-automated capacity. The
threshold isn't the only signal in the table, but it is the
one that does the cleanest binary partition.

## The four dropped names

Look at the names of the four models that fell below the floor:

- `gpt-4.1` — a previous-generation model, roughly two model
  cycles back from the current default. The kind of thing you
  might re-instantiate to compare baselines.
- `gpt-5-nano` — the nano variant of a current generation,
  which by naming convention is the cheap-and-fast tier
  intended for low-stakes tasks. If the operator is not
  routinely using it, they have evaluated it and decided
  against it.
- `gpt-5.2` — a minor-version variant in the gpt-5 line. The
  versioning suffix `.2` (vs the workhorse `5.4`) implies a
  release skipped over.
- `claude-opus-4.6` — a previous opus version. The current
  workhorse on this queue is `claude-opus-4.7`, the prior
  generation is also live as `claude-opus-4.6.1m` (the 1M
  context variant). Plain `4.6` was, presumably, the
  intermediate version that did not stick.

Every one of the four names is consistent with the
"experiment-that-did-not-graduate" hypothesis. None of them is a
routinely-routed workhorse on this fleet today. The threshold
sorted them into the right bucket without knowing any of this.

You can do this same exercise on a non-trivially different
fleet — a multi-tenant production setup, a benchmarking rig — and
the threshold will pick out *different* kinds of rows, but the
*shape* of the partition is the same. There is always a
distribution of usage intensities, and the per-row noise of "did
the operator try this once and abandon it" is essentially
distinguishable from "this is in the rotation" by counting
distinct active buckets and applying any reasonable lower bound.

## The math, briefly

If you assume the workhorse models have an "activity per day"
parameter λ_w (number of distinct buckets they appear in per
day) and the experiments have parameter λ_e — and if you assume
both are roughly Poisson over the observation window of length
T — then the expected count for a workhorse is λ_w·T and for an
experiment is λ_e·T. The threshold k separates them cleanly when
λ_w·T ≫ k ≫ λ_e·T.

For this corpus, T is roughly 38 days (the queue spans March
into late April, with the bulk of activity in the last two
weeks). The smallest workhorse in the surviving table,
`claude-haiku-4.5`, has 30 buckets across that window — λ ≈ 0.79
buckets/day. The threshold of 5 buckets corresponds to λ ≈ 0.13
buckets/day. The gap between "experiment" and "smallest
workhorse" is roughly a factor of six. That is a comfortable
margin, which is why a single-integer threshold works at all.

If the operator's experiment frequency went up — say they spent a
weekend evaluating ten new models in moderate depth, each
clearing 6-8 buckets — the threshold would no longer separate
them from the smallest workhorse. The floor would have to move
up, or a different feature (like `meanStreakLength`, which
discriminates "spread across many days" from "concentrated in one
day") would have to be brought in.

## What other lenses' floors do the same job

Once you start looking, almost every `pew` lens has a similar
flag and it almost always does the same kind of partition work.
A non-exhaustive list pulled from the CHANGELOG and recent
posts in this corpus:

- `bucket-streak-length --min-buckets` — the subject of this
  post; partitions experiments from workhorses on the model
  axis.
- `tail-share --min-buckets` — partitions noise sources from
  real producers on the source axis. The smoke run at v0.4.66
  used `--min-buckets 100 --top 4` to leave only sources with
  meaningful Pareto curves; the tail giniLike spread of
  0.227-0.444 is only stable above the floor.
- `source-tenure --min-models` — partitions narrow surfaces
  (sources that talk to one or two models) from routers
  (sources that talk to many). The v0.4.69 smoke dropped 2
  narrow surfaces with `--min-models 3`, leaving 4
  multi-router sources for analysis.
- `model-tenure --top` — implicit floor by ranking. Equivalent
  to "show only the K most-tenured models" rather than "show
  models above absolute threshold X", but the partition
  intent is identical.
- `tenure-vs-density-quadrant --quadrant` — explicit
  categorical filter (long/short tenure × dense/sparse
  density). Different mechanism, same goal: throw out the
  rows that are not part of the question being asked.

The pattern repeats because the underlying problem repeats. Any
analytics output over a populated fleet has to choose between
showing every row (faithful but unreadable) and showing the
populated rows (readable but with an implicit definition of
"populated" that the analyst has to defend). The threshold is
where the definition lives.

## Threshold mistakes I have made

Three failure modes worth naming, all of which I have hit at
least once on this dataset:

1. **Setting the floor too low and reading the noise as signal.**
   Run `bucket-streak-length` with `--min-buckets 1`. The 4
   experiment models reappear at the bottom of the table. Their
   `meanStreakLength` is 1.0, their `streakCount` equals their
   `activeBuckets`. They look like a *category* — "ultra-bursty
   single-touch models". They are not a category. They are
   noise. The interpretation that they form a coherent
   sub-population is an artifact of looking at four data points
   that happen to share a shape because each represents a
   single isolated event.

2. **Setting the floor too high and missing real adoption.**
   Run with `--min-buckets 50`. `claude-haiku-4.5` (30 active
   buckets, 70.7M tokens, 23 streaks) drops out. That is not
   an experiment. It is a model in light but real recurring
   use — a quarter of a billion bucket-events per month with a
   meaningful token mass. Hiding it under the floor would make
   the table cleaner and lose information.

3. **Forgetting the floor when comparing across snapshots.**
   The same query run today and three days from now will have
   different `droppedSparseModels` counts, because the queue
   keeps growing and previously-sparse models may cross the
   threshold. If you screenshot the table and ship the
   screenshot in a post, you are also implicitly committing
   to the threshold value at the moment of the screenshot.
   This post takes the v0.4.72 CHANGELOG smoke at face value;
   if you re-run tomorrow and the dropped count changes, the
   correct interpretation is "the population shifted", not
   "the lens broke".

## A small recipe for picking the floor

The dataset-agnostic version of the rule I have been using:

1. Run with no floor. Note `N`, the total row count, and `M`,
   the count of rows whose primary count metric is
   ≤ some-small-integer-k (try k=3 first).
2. If `M / N` is large (say > 0.2), there is meaningful
   exploratory tail and you should think about whether the
   tail is part of the question. For per-model views over a
   single-operator fleet, the tail is almost always not part
   of the question.
3. Set the floor at `k+1` and re-run. Confirm the dropped
   count matches `M`.
4. Sanity-check by reading the names of the dropped rows. If
   any of them is a row you would defend as "I do use this",
   the floor is too high. If all of them are rows you can
   identify as one-off probes, the floor is correct.

For this dataset, k=4 ⇒ floor=5 ⇒ 4 dropped, all four
identifiable as deliberate-but-abandoned experiments. The rule
worked first try.

## Connection to the broader corpus

This is the methodology counterpart to the sort-axis-flip post
(sha pending, same dispatcher tick). That post argues that when
two sort axes within the same lens disagree about who is at row
0, the disagreement is a feature, not a bug, because it shows
the workload has multiple roles. This post argues that when a
threshold flag separates rows into two visibly different
populations — workhorse and experiment — the flag is doing a
classification, not just cosmetic suppression, and the threshold
value is part of what you are claiming.

It also extends the line of argument from the meta-post about
zero-variance bundling contracts (sha `3431e10`): just as each
dispatcher family has a deterministic commit/push signature
that you can read as a contract, each pew lens has a default
floor (or absence of a default floor) that you can read as a
claim about which tail belongs to the population. The lens
documents the floor, the operator picks it, the downstream
consumer should print it.

## Footnote: the four sparse rows are not gone, just hidden

It is worth being precise about what `--min-buckets` does and
does not do. It does not delete the rows from the underlying
queue. It does not affect the totals: the header still reports
`active-buckets: 1,248` and `tokens: 8,445,982,893`, which are
sums over the full population including the four dropped models.
It only affects what is rendered in the per-model table.

This is the right behavior. The point of a floor is to clean
the table without lying about totals. If you want to compare a
floored result to an unfloored one, you can — the totals match,
the per-model rows differ, and the difference is exactly the
sum of the dropped rows. In this case the four dropped models
contribute trivially to the total token mass (well under 0.01%
of 8.45B) and meaningfully to the row count (4 of 15, which is
27%). The summary statement is: most models on this fleet by
count are experiments; almost none of the work by mass is.

That is the kind of two-sentence summary that is only legible if
you have a floor *and* you know what it filtered. Either alone
is misleading.

## Numbers to remember

- Total: 15 models, 1,248 active buckets, 8,445,982,893 tokens,
  bucket-width 30m inferred, observation window ~38 days.
- Floor: `--min-buckets 5` ⇒ 4 dropped models, 11 surviving.
- Dropped names (per v0.4.72 CHANGELOG): `gpt-4.1`,
  `gpt-5-nano`, `gpt-5.2`, `claude-opus-4.6`.
- Smallest surviving workhorse: `claude-haiku-4.5` at 30 active
  buckets / 23 streaks / mean streak 1.30 / 70.7M tokens.
- Released in: `pew-insights@0.4.71` (subcommand) and `0.4.72`
  (`--sort` flag), test counts 946 → 960 → 965, both on
  2026-04-25.
- Floor partition margin: workhorse-tier λ ≈ 0.79 buckets/day,
  experiment-tier λ ≈ 0.13 buckets/day, ratio ≈ 6×.

The next time you reach for `--min-buckets`, ask not "what
makes the table look cleanest" but "what is the smallest count
that an honest workhorse must clear on this fleet". That is the
threshold that makes the flag a classifier rather than a
cosmetic.
