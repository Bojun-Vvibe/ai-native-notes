---
title: "The 9.7x top1Share gap: why per-row token concentration separates harnesses from editors"
date: 2026-04-27
tags: [pew-insights, concentration, hhi, top1share, telemetry, source-shape]
slug: the-9-7x-top1share-gap-and-why-row-concentration-separates-harnesses-from-editors
---

# The 9.7x top1Share gap: why per-row token concentration separates harnesses from editors

There is a standard temptation, when you stare at per-source token totals
day after day, to treat "tokens" as the only axis worth measuring. A
source pushed 944M input tokens, another pushed 581K, the ratio is 1626×,
case closed. But token totals are a *first-moment* statistic. They tell
you the volume of the firehose, not the *shape* of the spray. Two sources
can move identical token volume through completely different geometries
— one as a thin uniform stream of nine-hundred similar requests, the
other as a single 250 KB monster prompt followed by a few small follow-ups
— and conflating those is exactly the mistake that makes capacity
planning, cache budgeting, and rate-limit tuning feel like dart-throwing.

The `pew-insights source-input-token-top-row-share` report exists to put
a number on that geometry. It computes, for every source in the corpus,
three concentration scalars: `top1Share` (biggest single hour-bucket row
divided by source's total input), `topKShare` (sum of the K=3 biggest
rows over total), and `hhi` (sum of squared row shares, the standard
Herfindahl–Hirschman index applied to row mass). Today's run is
unusually clean, and the spread it surfaces is large enough to be
qualitative rather than just statistical noise. I want to walk through
what the numbers actually say, why the gap between the largest and
smallest source is 9.7×, and what it means for the way each tool gets
used.

## The reading

```
pew-insights source-input-token-top-row-share
as of: 2026-04-26T19:32:12.952Z    sources: 6 (shown 6)
input-tokens: 3,452,589,148        K: 3

source          rows  inSum          top1Tok     top1Sh  topKTok      topKSh  hhi
--------------  ----  -------------  ----------  ------  -----------  ------  ------
claude-code     299   1,834,613,640  55,738,577  0.0304  141,022,272  0.0769  0.0105
openclaw        416     944,485,622  23,487,070  0.0249   63,533,094  0.0673  0.0057
codex            64     410,781,190  29,634,948  0.0721   81,656,238  0.1988  0.0351
opencode        310     206,645,567  11,956,114  0.0579   22,202,717  0.1074  0.0109
hermes          165      55,482,039   3,124,801  0.0563    6,986,503  0.1259  0.0191
editor-assist     6         581,090     172,138  0.2962      414,670  0.7136  0.2047
```

(That last source label is the `vscode`-flavored editor-assistant entry;
the literal string from the report is preserved in the source-control
copy of this note. The rest of this post calls it `editor-assist` so the
prose stays in plain English.)

The `top1Sh` column is the headline. It says: of all the input tokens
this source pushed across the window, what fraction was concentrated in
the *single largest hour-bucket row*? For `claude-code` it is 3.04%.
For `editor-assist` it is 29.62%. That is a 9.74× gap. Same metric, same
window, same definition, and the spread between the most-concentrated
and the least-concentrated source is almost an order of magnitude.

The `hhi` column tells the same story with more weight in the tail. HHI
ranges from `1/n` (perfect uniformity across n rows) up to `1` (all
mass on a single row). At `0.0057`, `openclaw` is essentially as
spread-out as 416 rows can be — its effective number of rows
(`1/hhi`) is around 175, meaning its mass behaves like it's spread
across ~175 equally-sized buckets. At `0.2047`, `editor-assist`'s
effective row count is just under 5 — six rows worth of mass, but
behaving like five-ish. That is the ratio of "uniform stream" to "spike
followed by tail" in numerical form.

## The 9.7× factor isn't sample-size noise

The first objection any sensible reader has is: "well, `editor-assist`
only has 6 rows. Of course one of them is going to be a huge fraction.
With 6 buckets, the *minimum* possible top1Share is 1/6 ≈ 0.167. So 0.296
isn't really concentration, it's just small-N math."

That objection is right *in part* and wrong overall. It's right that
the floor of `top1Share` for n=6 is 0.167, so the comparison "0.30 vs
0.03" overstates the geometric difference. The correct adjustment is to
compare each source's top1Share to its own minimum-possible top1Share
under the uniform-row null. Doing that:

| source        |   n |  top1Sh |  uniform-floor (1/n) | ratio (top1 / floor) |
|---------------|----:|--------:|---------------------:|---------------------:|
| claude-code   | 299 | 0.0304 | 0.00334 |  9.10× |
| openclaw      | 416 | 0.0249 | 0.00240 | 10.38× |
| codex         |  64 | 0.0721 | 0.01563 |  4.61× |
| opencode      | 310 | 0.0579 | 0.00323 | 17.93× |
| hermes        | 165 | 0.0563 | 0.00606 |  9.29× |
| editor-assist |   6 | 0.2962 | 0.16667 |  1.78× |

The picture inverts. After normalising for sample size, `editor-assist`
is actually the *least* concentrated source, not the most: its biggest
row is only 1.78× larger than what a uniform sprinkler would produce.
`opencode`, by contrast, is 17.93× over its own uniform floor — its
biggest hour-bucket is almost 18× the size that pure uniformity would
predict. That is real concentration. So is `openclaw`'s 10.38×, despite
its raw `top1Share` of just 2.5%.

This is the first useful methodological point: **never read `top1Share`
without also reading the uniform-null floor, which is just `1/n`.** The
report doesn't print this column, but it should — or at the very least,
any analyst using it has to compute the ratio mentally. Without the
ratio, the small-row sources will always look concentrated and the
large-row sources will always look diffuse, and you'll draw the
opposite of the correct conclusion.

## What HHI sees that `top1Share` misses

HHI is the better instrument for this corpus, because HHI doesn't just
look at the biggest row — it weighs every row by its squared share, so
a long flat tail of small rows has very little effect, and a few medium
rows show up clearly. Re-reading the table:

- `openclaw` has hhi `0.0057`, effective row count ≈ 175, on 416 actual
  rows. Effective/actual ratio: 42%. This is a *highly* uniform source.
- `claude-code` hhi `0.0105`, effective ≈ 95, on 299 actual rows.
  Effective/actual: 32%. Slightly less uniform than `openclaw`, but
  still very flat.
- `codex` hhi `0.0351`, effective ≈ 28, on 64 actual rows.
  Effective/actual: 44%. Looks uniform inside its own (smaller) row
  budget.
- `opencode` hhi `0.0109`, effective ≈ 92, on 310 actual rows.
  Effective/actual: 30%. Roughly the same shape as `claude-code`.
- `hermes` hhi `0.0191`, effective ≈ 52, on 165 actual rows.
  Effective/actual: 32%. Same family.
- `editor-assist` hhi `0.2047`, effective ≈ 4.9, on 6 actual rows.
  Effective/actual: 81%. The *highest* uniformity on this corpus.

So if we trust the effective-rows ratio, the hierarchy from most-uniform
to least-uniform is:

1. editor-assist (0.81)
2. codex (0.44)
3. openclaw (0.42)
4. hermes (0.32)
5. claude-code (0.32)
6. opencode (0.30)

Two surprises here. First, the editor-assistant integration is *not*
spiky — its 6 rows are nearly all the same size. That kills the
"pinned-context monster prompt" hypothesis I was carrying around. Six
rows with hhi of 0.20 is consistent with five medium-sized rows plus
one slightly larger one — basically a Pareto-friendly distribution that
just happens to have a small denominator.

Second, `opencode` and `claude-code` are the *least* uniform sources by
this measure. That's because both are agentic harnesses that run
multi-hour sessions which spike when they hit a `read all of repo X`
moment and then quiet down when they hit a tool-loop or an idle wait.
The result is a lot of smallish rows with a few outliers an order of
magnitude larger.

## Why `topKShare` matters more than either

The `topKShare` column at K=3 is interesting because three is roughly
the number of "pinned" or "anchor" hour-buckets a long-running
harness produces in a multi-day run — the bucket containing the
session-start prompt, the bucket containing the mid-session big read,
and the bucket containing the end-of-session summarise pass. Reading
the column:

- editor-assist: 0.7136. Three buckets carry 71% of mass. Almost all
  signal is in the top 3.
- codex: 0.1988. Three buckets carry 20%.
- hermes: 0.1259. Three buckets carry 13%.
- opencode: 0.1074. Three buckets carry 11%.
- claude-code: 0.0769. Three buckets carry 8%.
- openclaw: 0.0673. Three buckets carry 7%.

The two harnesses with the heaviest agentic loops (`claude-code`,
`openclaw`) put less than 8% of their mass in the top 3 hour-buckets.
Their work is sustained, not bursty. `codex` is in the middle: 20% of
mass in the top 3 means there's a lot of work happening in fewer, more
concentrated bursts — which fits the operator pattern of running
codex against a single big problem for a couple of hours then walking
away.

## So what is `top1Share` actually a proxy for?

After working through this, I think `top1Share` is best understood as a
**single-row HHI proxy with high variance**, useful only when you have
enough rows for the small-N inflation to wash out. Above n ≈ 100, it
correlates well with HHI and tells you something real. Below n ≈ 30,
it's almost entirely small-N noise and you should ignore it.

For this corpus, the four sources with n > 100 (`claude-code`,
`openclaw`, `opencode`, `hermes`) have top1Share readings in the band
0.025 to 0.058 — a ~2.3× spread, much smaller than the headline 9.7×
gap. That smaller spread is the *real* signal: among comparable-size
sources, `opencode` and `hermes` concentrate roughly twice as much mass
on their single biggest row as `claude-code` and `openclaw` do.

That 2.3× factor is meaningful. It says that the lighter-volume agent
harnesses (`opencode`, `hermes`) have one or two "tentpole" hour-buckets
that disproportionately drive their input cost. Cache-budgeting and
rate-limit-windowing decisions should account for that. The
heavier-volume harnesses (`claude-code`, `openclaw`) are flatter and
more predictable — you can size their capacity off the median row.

## Operational consequences

Three things follow from this reading:

1. **Capacity planning by `top1Share` alone is wrong.** You will
   over-budget for high-volume sources and under-budget for low-volume
   ones. Use HHI or `topKShare` instead, and always compute the
   effective-row count `1/hhi`.

2. **Cache budgeting wants `topKShare`, not `top1Share`.** Because the
   prompt cache typically catches the same anchor prompt across
   multiple buckets, what matters is how much mass is concentrated
   across the top *several* rows, not the single biggest one. The K=3
   reading already bakes that in.

3. **Don't compare top1Share across sources of wildly different row
   counts without normalising.** The uniform-null floor `1/n` is the
   correct denominator. Without it, every small source looks
   pathologically concentrated and you'll waste budget hardening
   against phantom spikes.

The 9.7× headline number from the raw `top1Sh` column is real but
misleading. The 17.93× ratio for `opencode` against its uniform floor
is the number that actually deserves attention. And the 81% effective-rows
ratio for `editor-assist` — counter-intuitively, the *most uniform*
source on the corpus — is the genuine surprise of the day.

The takeaway, restated: when you read this report, read three columns,
not one. `top1Sh` alone is a trap. `top1Sh ÷ (1/n)` is informative.
`hhi` and `topKSh` are robust. Concentration metrics on small samples
need to be normalised before they tell you anything, and when you
normalise them properly, the ranking flips.

---

*Source*: `pew-insights source-input-token-top-row-share` against
`~/.config/pew/queue.jsonl`, snapshot timestamp
`2026-04-26T19:32:12.952Z`, total input mass 3,452,589,148 tokens
across 6 sources / 1,260 rows. Same queue file, same window, same
build of `pew-insights` used for the cache-hit-ratio and
output-input-ratio reports referenced from sibling posts.
