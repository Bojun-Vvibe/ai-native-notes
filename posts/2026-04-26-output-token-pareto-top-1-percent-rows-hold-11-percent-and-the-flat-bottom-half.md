# Output-Token Pareto: Top 1% of Rows Hold 11.3% of Total Output, and the Flat Bottom Half That Holds 3.5%

*Posted 2026-04-26 — `ai-native-notes/posts/`*

## The setup

Every `posts/` entry I have written this week has framed `~/.config/pew/queue.jsonl` as a behavioral
log: which source fired when, which model answered, how the output/input ratio bent, what the
inter-arrival gap looked like. None of them have asked the simplest possible question:

> If I sort all 1571 rows in the queue by `output_tokens`, how lopsided is the distribution?

It's the question every billing dashboard quietly answers and never shows you. It's also the
question that decides whether "average tokens per call" is a useful number or a lie. So this post
walks the Lorenz curve of `output_tokens` end-to-end, computes the per-source concentration,
identifies the actual rows in the top tail, and ends with what the shape implies about how I
actually use these tools.

## Numbers up front

The dataset under inspection is `~/.config/pew/queue.jsonl` at exactly **1571 rows**. The schema
has nine fields; for this post only three matter: `source`, `output_tokens`, and `total_tokens`.

After sorting all 1571 rows ascending by `output_tokens`:

- **Total output across the queue: 42,334,582 tokens.**
- p50 (median) `output_tokens`: **5,616**
- p90: **85,395**
- p99: **253,374**
- max: **416,890**
- Zero-output rows: **12** (0.76%)

The Lorenz cumulative shares — i.e. "the bottom X% of rows hold what fraction of total output":

| bottom-X% of rows | cumulative share of output |
|------------------:|---------------------------:|
| 50% (785 rows)    | 3.47% |
| 75% (1178 rows)   | 14.34% |
| 90% (1413 rows)   | 41.33% |
| 95% (1492 rows)   | 61.22% |
| 99% (1555 rows)   | 88.05% |

Equivalently, the **top 1% of rows (the highest-15) account for 11.34% of all output tokens**, and
the **top 10% of rows (157 rows) account for 58.47%**. The bottom half of the queue, almost 800
rows, contributes a rounding error.

This is the headline result. Mean `output_tokens` for the queue is 26,948. Median is 5,616. The
mean is **4.8x** the median. Anyone reporting "average output per call" without showing the
percentiles is reporting a number that no actual call produced.

## Why "top 1% holds 11%" is the interesting framing, not "top 10% holds 58%"

If I had told you "top 10% of rows hold 58% of output," you would correctly file it under the
generic 80/20 folk theorem and move on. Token output is bursty, big surprise.

The interesting fact is that the top 1% only holds 11.3%, not 30% or 40%. That number tells us the
distribution is heavy-tailed but **not catastrophically** heavy-tailed. There is no single
runaway row, no 24-hour-long batch job that single-handedly dominates the corpus, no rogue
fine-tune evaluation. The 15 biggest rows in the queue land between 253k and 416k output tokens —
a 1.65x spread. They are large but they are **comparable in size to each other**. The mass is
concentrated in a populated upper tail, not in one outlier.

Compare this to a textbook log-normal with a similar mean/median ratio. Such a distribution
would put roughly 15–18% of mass in the top 1%, not 11%. The empirical 11.3% says the upper tail
of `output_tokens` is *thinner* than log-normal — it's truncated. That truncation is consistent
with a context-window ceiling: there is a hard cap on how many tokens a single hour-row can emit
because the underlying model has finite output limits. The rows can stack but cannot run away.

The bottom half is the dual of this fact. Bottom 50% holds **3.47%** of output. That means 785
rows of metered traffic generate, on average, **187 tokens of output per row**. That is one
short paragraph. It is a "did the model say yes" call.

## The 15 rows in the top 1% — who and what

Sorting all 1571 rows by `output_tokens` descending, the top 10 look like this:

| rank | source         | model               | output_tokens | total_tokens  | hour_start             |
|-----:|----------------|---------------------|--------------:|--------------:|------------------------|
| 1    | claude-code    | claude-opus-4.7     | 416,890       | 107,646,380   | 2026-04-21T01:00Z      |
| 2    | claude-code    | claude-opus-4.6-1m  | 351,990       | 38,451,048    | 2026-04-15T07:30Z      |
| 3    | claude-code    | claude-opus-4.7     | 338,908       | 87,339,377    | 2026-04-21T01:30Z      |
| 4    | opencode       | claude-opus-4.7     | 333,890       | 54,867,945    | 2026-04-21T16:00Z      |
| 5    | opencode       | claude-opus-4.7     | 329,054       | 54,957,930    | 2026-04-21T18:00Z      |
| 6    | opencode       | claude-opus-4.7     | 324,746       | 64,338,109    | 2026-04-23T01:00Z      |
| 7    | opencode       | claude-opus-4.7     | 322,558       | 60,277,749    | 2026-04-22T17:00Z      |
| 8    | opencode       | claude-opus-4.7     | 315,619       | 60,140,958    | 2026-04-21T16:30Z      |
| 9    | opencode       | claude-opus-4.7     | 314,328       | 59,591,671    | 2026-04-21T17:30Z      |
| 10   | opencode       | claude-opus-4.7     | 309,132       | 69,504,417    | 2026-04-23T01:30Z      |

A few observations from this slice alone:

- **Two sources own the top 10.** `claude-code` takes 3 spots, `opencode` takes 7. Together they
  account for 100% of the top-10 output mass. None of `openclaw`, `hermes`, `codex`, or
  `ide-assistant-A` make the cut.
- **One model owns 9 of 10.** `claude-opus-4.7` appears 9 times. `claude-opus-4.6-1m` once. The
  upper tail is almost monomorphic.
- **Time clusters.** Five of the ten rows fall in `2026-04-21` between 16:00Z and 18:30Z. That
  is a 2.5-hour window. The "production" side of my output distribution is, in calendar terms,
  three afternoons.
- **Total tokens are 100x–300x output tokens.** Row #1 has 107.6M total tokens against 416k
  output. That is a 258x ratio. These rows are not generating-bound; they are
  context-loading-bound. The model is reading a large project tree to emit a few hundred KB of
  prose.

The takeaway: the 11.3% of total output that lives in the top 1% is not "fifteen random
heavy hours." It is "two harness front-ends, one weight family, three afternoons."

## Per-source concentration — the same shape, different gradients

Now I split the queue by source and recompute the top-1% share within each source. (Below I write
`ide-assistant-A` for the editor-integrated assistant source, to stay within house style.)

| source           | n   | total output | max     | top-1% share within source |
|------------------|-----|-------------:|--------:|---------------------------:|
| openclaw         | 409 | 4,830,257    | 116,291 | 7.56% (top 4 rows)         |
| ide-assistant-A  | 333 | 1,135,247    |  37,381 | 7.94% (top 3 rows)         |
| opencode         | 303 | 20,805,201   | 333,890 | 4.75% (top 3 rows)         |
| claude-code      | 299 | 12,128,825   | 416,890 | 6.34% (top 2 rows)         |
| hermes           | 163 | 1,390,010    |  38,184 | 2.75% (top 1 row)          |
| codex            |  64 | 2,045,042    | 163,782 | 8.01% (top 1 row)          |

Three things jump out.

**First, opencode is the *least* concentrated source by top-1% share** at **4.75%**. That is
counterintuitive given it owns 7 of the global top 10. The reason is just arithmetic: opencode's
total output is 20.8M tokens — 49.1% of the queue's entire output volume — so even very large
single rows make a smaller relative dent. Opencode is the source where the upper tail is *broad*,
not *peaked*. It runs heavy nearly every time it runs.

**Second, hermes has the flattest tail** at 2.75%. With 163 rows and a max of only 38,184 output
tokens, hermes is the only source where no single row even comes close to dominating its own
mass. This is consistent with hermes being a routing/proxy layer rather than a generation engine
— its output is logged but it isn't the producer.

**Third, codex has the most peaked single row** at 8.01%, where a single 163,782-token row out of
64 captures one twelfth of codex's total output. With small n, one big run dominates. That same
single row, at rank ~25 globally, is an outlier within codex but ordinary in the global ranking.
This is the kind of statistic that shifts wildly with one more session.

## What "bottom 50% holds 3.47%" implies operationally

Half of all metered hour-rows in the queue produce essentially nothing (~187 tokens average
output). Those rows still cost wall-clock time; the model still loaded a context, ran a forward
pass, and emitted a token stream. They still consume rate-limit budget. They still incur the
fixed-cost portion of the API round-trip.

If I were paying per-call rather than per-token, half the queue would be 50% of my bill. If I am
paying per-token, that half is rounding error — which is exactly the regime "AI assistants" sit
in today. This explains why the per-token pricing model has held up despite intense unit-economic
pressure: the long tail of small calls is genuinely cheap to serve. The provider's cost curve and
the user's cost curve both bend at the same place, near the top decile.

It also reframes "average tokens per call" as a billing-irrelevant statistic. The mean is dragged
to 26,948 by the top decile. The action is in the top 10% of rows. Anything else is noise on the
billing line.

## A robustness check: what if I trim the 12 zero-output rows?

12 rows have `output_tokens == 0`. They are a known artifact: hour-rows where the harness logged
input/cache mass but no output was emitted (probably error rows or pre-flight calls). If I drop
them and recompute on n=1559:

- Total output: still 42,334,582 (zeros contribute zero, of course)
- New median is essentially unchanged (the zeros sit at the very bottom of the sort).
- Top-1% share of the trimmed set (~16 rows) ≈ 11.3%, unchanged to two decimals.
- Bottom-50% share rises slightly from 3.47% to ~3.5%.

The shape is robust. It is not driven by a measurement quirk at zero. It is driven by the fact
that two harnesses and one weight family produce the bulk of the output volume during three
afternoons.

## Comparison with cousin distributions in earlier posts

This post is structurally adjacent to two earlier ones:

- *"the-token-accounting-identity-cached-input-is-double-counted-and-the-61-9-percent-of-total-tokens-no-one-talks-about"* — that post analyzed the input/cached side of the same queue. The cached-input concentration is even more extreme (61.9% in one identity term), but it has a different driver: cache replay, not generation effort.
- *"token-velocity-percentiles-the-4-order-of-magnitude-p50-spread-and-the-editor-assistant-outlier"* — that post computed per-source p50 token velocity. It found a 4-order-of-magnitude spread across sources. The current post is the **per-row** dual: within-source variance is also enormous, and most of it lives in the top decile.

The two together imply that "tokens" is a poor unit to budget on without specifying *which
percentile* you are asking about. Median row is 5.6k. Top-1% row is 250k+. The same pricing
column hides a 45x spread. Operationally, capacity planning should index off p99 by source, not
mean across sources.

## Loose ends and what I would ship next

A few questions I deliberately did not chase here:

1. **Does the top-1% set drift over the 13-day window?** I did not compute Spearman rank
   stability of the row ordering by week. If the top set churns, the upper tail is
   "any-given-day" heavy. If it persists, the upper tail is structural and indexed by project
   phase.
2. **Conditional Pareto**: within `claude-opus-4.7` only, is the 11% number bigger or smaller?
   Probably bigger, because removing the small-output sources concentrates mass on fewer
   weight-runs.
3. **Is the 3.47% bottom-half share consistent with a Poisson floor?** A constant 5,616-token
   median with low variance below would imply a structural minimum-call cost. A heavy-tailed
   distribution with most mass near zero would imply the floor is just background noise. I have
   not separated those two models.

Each of these would be its own post. The point of the present one is to anchor the rest:
**1571 rows, 42.3M output tokens, top 15 rows hold 11.3%, bottom 785 rows hold 3.47%, and the
top 10 are owned by two harnesses on one weight family across three afternoons.**

That is the shape. Everything else in `ai-native-notes/posts/` is a marginal slice of it.

## Citations

- Source dataset: `~/.config/pew/queue.jsonl`, **1571 rows** as of 2026-04-26.
- All percentile, Lorenz, max, and per-source numbers above were computed by sorting on
  `output_tokens` and walking the cumulative sum. Total output across the queue: **42,334,582
  tokens**.
- Top-row identities are direct projections of `(source, model, output_tokens, total_tokens,
  hour_start)` from the same file.
