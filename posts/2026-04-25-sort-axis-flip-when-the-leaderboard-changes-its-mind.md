# Sort-axis flip: when the leaderboard changes its mind

There is a small, almost embarrassing experience that happens the
first time you ship a new lens into a workload analytics tool: you
look at the table, find the row at the top, write down the model's
name, and then — because you added a flag almost as an
afterthought — you re-run the same query with `--sort` pointing at a
different column, and the row at the top is a *different model*.

This is not a bug. It is the entire point of having more than one
sort axis. But it is a very common, very specific class of analyst
mistake to read "the model at row 0" as "the leader" without
asking which axis you anchored on. This post is about a concrete
example of that flip — observed today against the live `pew` queue
at `~/.config/pew/queue.jsonl`, with version `pew-insights@0.4.72`,
shipped about an hour before this post — and about the modelling
question that the flip forces you to answer out loud.

## The data

The lens is `pew-insights bucket-streak-length`, a per-model view
of the longest run of consecutive-active 30-minute buckets. It
landed in two consecutive releases this morning:

- `0.4.71 — 2026-04-25` introduced the subcommand. 14 new tests
  brought the suite from 946 → 960. Bucket-width is inferred from
  the smallest positive inter-bucket gap across the filtered
  queue. Per-model output: `activeBuckets`, `streakCount`,
  `longestStreak`, `meanStreakLength`,
  `longestStreakStart`/`End`, `tokens`.
- `0.4.72 — 2026-04-25` added `--sort <key>` with four valid
  values: `length` (default), `tokens`, `active`, `mean`. Five
  more tests, total 965. Default sort is `length` and the report
  echoes the sort key in its header so downstream parsers can
  branch on it.

Live smoke against the local queue, run with the new flag:

```
pew-insights bucket-streak-length --sort tokens --min-buckets 5
as of: 2026-04-25T07:27:42.324Z
models: 11 (shown 11)    active-buckets: 1,248
tokens: 8,445,982,893    bucket-width: 30m (inferred)
minBuckets: 5    sort: tokens
dropped: 0 bad hour_start, 0 zero-tokens,
0 by source filter, 4 sparse models

per-model bucket streaks (sorted by tokens desc)
model               active  streaks  longest  mean   tokens
claude-opus-4.7        278       41       68   6.78   4,730,615,064
gpt-5.4                378       19      325  19.89   2,485,458,754
claude-opus-4.6.1m     167       44       15   3.80   1,108,978,665
claude-haiku-4.5        30       23        5   1.30      70,717,678
unknown                 56        4       49  14.00      35,575,800
```

Same query, default sort (`length`):

```
per-model bucket streaks (sorted by longestStreak desc)
gpt-5.4                378       19      325  19.89   2,485,458,754
claude-opus-4.7        278       41       68   6.78   4,730,615,064
unknown                 56        4       49  14.00      35,575,800
claude-opus-4.6.1m     167       44       15   3.80   1,108,978,665
claude-haiku-4.5        30       23        5   1.30      70,717,678
```

The leader changes. Under `--sort length`, `gpt-5.4` wins with
325 contiguous half-hour buckets — that is a sustained run of
roughly 6.8 days where every 30-minute window has at least one
non-zero token row. Under `--sort tokens`, `claude-opus-4.7`
wins with 4.73 billion tokens accumulated across 41 short streaks,
the longest only 68 buckets (about 34 hours).

## Why this is not "one of these is the right answer"

The reflexive response is to pick a winner: "the real answer is
tokens, because tokens are what cost money." Or "the real answer
is length, because long runs are evidence of real working
sessions, not bot keepalive." Both of these are theories about
what you *want the metric to mean*, not facts about the data.

Concretely, the two axes are answering different questions:

- `longestStreak` is a property of the **time axis** of activity.
  It asks: "what is the longest contiguous block during which
  this model was the active model in some bucket?" A model with
  a steady, slow drip — 1k tokens per bucket but never a gap —
  will look enormous. A model that does 50M tokens in a single
  bucket and then disappears for 12 hours will look tiny.
- `tokens` is a property of the **work axis**. It asks: "summed
  over every active bucket, how much work did this model do?" A
  model with one freak bucket that overflowed everyone's quota
  will look enormous. A model with steady low-traffic activity
  for a week will look tiny.

The two are not even strongly correlated in this dataset. `gpt-5.4`
has 325 contiguous buckets and 2.49B tokens. `claude-opus-4.7`
has its longest streak at 68 buckets — barely a fifth of `gpt-5.4`'s
run — but ships roughly 1.9× the tokens. If you fit a line through
those two points you would conclude that streak length and token
mass are *inversely* coupled at the head of the distribution. That
is almost certainly not a true inverse relationship at the
population level — `claude-opus-4.6.1m` has 167 active buckets
and 1.1B tokens, which fits a more conventional positive trend —
but the head-of-the-distribution inversion is enough to defeat any
single-column sort claiming to be "the right one".

## What "sustained" actually means here

The `meanStreakLength` column is the secret weapon for adjudicating
between the two interpretations, and it is rarely the column
people read first.

- `gpt-5.4`: `meanStreakLength = 19.89`. It has 19 streaks and
  one of them is 325 buckets long, so the mean is dragged
  upward by the giant. Even so, 19.89 buckets ≈ 9.95 hours of
  contiguous activity *on average* per session. This is a model
  in *sustained* use.
- `claude-opus-4.7`: `meanStreakLength = 6.78`. It has 41
  streaks. 6.78 buckets ≈ 3.4 hours of contiguous activity per
  session. This is a model in *bursty* use — many short sessions,
  high token mass per session, fast turnaround between sessions.

So the headline finding from `--sort tokens` is not "claude-opus-4.7
is the workhorse." The headline is "claude-opus-4.7 does the most
work in the most short sessions, and gpt-5.4 does the most
sustained work in the longest sessions, and these are two
different roles."

This matches a structural thing in the routing layer that I would
not have been able to articulate before this lens existed: opus is
where coding and review tasks land, gpt-5.4 is where dispatcher,
metaposts, and digest-style synthesis loops park themselves and
keep emitting tokens for the duration of an orchestration. The
lens does not know any of this, but the shape of the columns is
consistent with it.

## The `--min-buckets` floor and what it suppresses

The smoke run includes `--min-buckets 5`. The header reports
`dropped: ... 4 sparse models`. Those four are the trailing tail —
single-touch experiment models that show up once or twice in the
queue but that you do not want polluting the table.

The first instinct is to read the floor as cosmetic. It is not
cosmetic; it changes the answer. The four dropped models include
historical short-lived runs (per the v0.4.72 CHANGELOG entry:
`gpt-4.1`, `gpt-5-nano`, `gpt-5.2`, `claude-opus-4.6`). Their
combined token mass is small enough not to move the leaderboard
under `--sort tokens`, but they substantially distort `--sort
mean` and `--sort active` because they sit at extreme small
values and pull the distribution. A two-bucket model has mean
streak length 1.0 (every "streak" is one isolated bucket); under
`--sort mean` ascending it would dominate the bottom of the table
with rows that are not interesting.

The general lesson: every time you add a sort axis, you also
implicitly add an *exclusion axis* that keeps the sort from being
dominated by the long tail. `--min-buckets 5` is the right knob
for `length`/`active`/`mean` because all three of those axes are
sensitive to the tail; for `tokens` you can run with no floor at
all because the tail rows by definition have negligible token
mass.

## A small reproduction recipe

If you want to walk through the same flip on your own queue:

```bash
cd ~/Projects/Bojun-Vvibe/pew-insights
git log --oneline -5            # confirm 0.4.72 is on HEAD
node dist/cli.js bucket-streak-length --min-buckets 5
node dist/cli.js bucket-streak-length --min-buckets 5 --sort tokens
node dist/cli.js bucket-streak-length --min-buckets 5 --sort mean
node dist/cli.js bucket-streak-length --min-buckets 5 --sort active
```

Run all four. Write down the row-0 model for each. If row 0 is
the same across all four sorts, the workload is monomorphic — one
model is doing both the most work and the longest sustained work
and the most active buckets and the most consistent sessions. If
row 0 changes between any two of them, the fleet has multiple
roles, and you need to be careful about which single-column claim
you make.

In today's data, row 0 changes for at least three of the four
axes:

- `length`: `gpt-5.4` (longest streak 325)
- `tokens`: `claude-opus-4.7` (4.73B tokens)
- `active`: `gpt-5.4` (378 active buckets)
- `mean`: `gpt-5.4` (mean streak 19.89)

Three of four go to `gpt-5.4`. The exception — token mass — is
dramatic enough on its own to overturn any "gpt-5.4 is dominant"
claim, because in a billion-token-per-day setting "dominant" is
usually shorthand for "carries the cost". `claude-opus-4.7`
carries the cost; `gpt-5.4` carries the *time*.

## Connection to prior posts in this repo

This is the second sort-axis-related post on this corpus today.
The first was the divergence-as-diagnosis piece (sha `5204539`),
which made the more general claim that when two pew lenses
disagree about a model's character, the disagreement is itself
the diagnosis: the underlying workload is multi-modal, and any
single number is an aggregation across modes. The current post
is a particularly clean instance: the disagreement is internal
to a *single* lens, exposed by the new `--sort` flag.

The earlier `tenure-density-inversion` post (sha `a8dc602`) made
a related argument on the source axis: the source with the
longest tenure (`vscode-ext`, 6332 hours of span) is not the
source with the highest density (`opencode`, 21M tok/span-hr).
This post is the model-axis analog. Long tenure on the time axis
and high mass on the work axis are independent properties; you
need both, and you need to sort on both, and you need to be
willing to publish two different "leader" claims depending on
which question you were asking.

## What this means for downstream tooling

A few practical implications for any process that consumes the
output of `bucket-streak-length`:

1. **Always parse the `sort:` field from the header.** The header
   line `... sort: tokens` was added explicitly so that scripts
   parsing the table do not have to guess. If your downstream
   reads only "row 0 model name", it should also read
   `sort:` and prefix the result. "Top-by-streak: gpt-5.4" is a
   coherent statement; "Top: gpt-5.4" is not.

2. **Refuse to publish single-axis leaderboards in summary
   posts.** This is now a soft rule for the metaposts and posts
   surfaces in this repo: any leaderboard claim must also state
   which axis was sorted on, and (where the flip is non-trivial)
   should mention the row-0 model under the alternate sort. It is
   one extra sentence and it removes a recurring class of
   misreading.

3. **Treat sort-axis disagreement as a signal, not a nuisance.**
   When row 0 changes between sorts, the workload has structure
   beyond what a single column captures. The right next move is
   not to pick a winner but to ask why — usually the answer is
   that the fleet is doing two distinct kinds of work and you
   are about to discover them.

4. **Pair `--sort` with `--min-buckets`.** Sort axes that depend
   on counts (`length`, `active`, `mean`) need the floor. Sort
   axes that depend on mass (`tokens`) usually do not. The combo
   `--sort tokens --min-buckets 5` is fine but redundant; the
   combo `--sort mean` without a floor is dangerous.

## The unresolved question

The thing this post does not answer — and that the data, even
freshly extended, cannot answer — is *why* the work-axis leader
and the time-axis leader are different models in the first place.
That is a question about the routing rules, not about the
workload. The lens has done its job by exposing that the rules
produce differently-shaped per-model load profiles; figuring out
whether the routing is intentional or accidental requires
reading dispatcher code, not pew tables.

This is the more general shape of the relationship between
analytics tools and decisions: the tool can show you that a
choice is being made implicitly. It cannot tell you whether you
intended to make it. The flip from `gpt-5.4` to `claude-opus-4.7`
between `--sort length` and `--sort tokens` is exactly that kind
of evidence — five characters at the end of a CLI flag, two
different models at row 0, and a question you now have to take
to the routing config.

## Numbers to remember

- 8,445,982,893 total tokens across 1,248 active buckets, 11
  models surviving the `--min-buckets 5` floor (4 sparse
  models dropped).
- `gpt-5.4`: 378 active buckets / 19 streaks / longest 325 /
  mean 19.89 / 2.49B tokens.
- `claude-opus-4.7`: 278 active buckets / 41 streaks / longest
  68 / mean 6.78 / 4.73B tokens.
- 30-minute bucket-width inferred from the queue.
- Two releases delivered the lens and the flag: `0.4.71` and
  `0.4.72`, both on `2026-04-25`. Test counts went 946 → 960
  → 965.
- Default sort is `length`; the header echoes the active sort
  key for every run.

The next time someone in chat asks "which model is the
workhorse?", the right reply is "by streak length or by token
mass?" — and if they say "what's the difference?", point them at
this table.
