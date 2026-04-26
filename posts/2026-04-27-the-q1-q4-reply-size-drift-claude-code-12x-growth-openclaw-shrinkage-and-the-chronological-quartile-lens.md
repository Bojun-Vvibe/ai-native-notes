---
title: "The Q1→Q4 reply-size drift: 12.4× growth on claude-code, 0.85× decay on openclaw, and the chronological-quartile lens"
date: 2026-04-27
tags: [pew-insights, drift-detection, quartiles, source-analysis, output-tokens]
est_reading_time: 14 min
---

## The headline

Run `pew-insights source-first-vs-last-quartile-output-mean-shift` against
`~/.config/pew/queue.jsonl` and the answer is not the orderly monotonic
ladder I was expecting. Six sources, six different drift signs and
magnitudes, with a spread of more than **13 units of `relShift`** between
the top of the table (`claude-code` at +12.38) and the bottom (`openclaw`
at −0.85). And the two sources at the top — `claude-code` and `codex` —
are both *growing* their reply size by an order of magnitude (12.4× and
1.94× respectively) over the same window in which the two sources at the
bottom — `ide-assistant-A` and `openclaw` — are *shrinking* theirs (−0.35
and −0.85). On the same operator, on the same machine, in the same
calendar window. That's not noise; that's two different system-level
drifts running in parallel.

The full table, from the run at `2026-04-26T23:46:48.222Z`
(corpus: 6 sources, 1,607 rows, min-rows = 4, sort = shift-desc, no
top-cap):

```
source           rows  qRows  firstQMean  lastQMean  meanShift   relShift
---------------  ----  -----  ----------  ---------  ----------  --------
claude-code      299   74     8566.23     114618.18  106051.95   12.3802
codex            64    16     21846.38    64236.81   42390.44     1.9404
opencode         319   79     82335.68    92715.78   10380.10     0.1261
hermes           167   41     8232.17     8151.61    -80.56      -0.0098
ide-assistant-A  333   83     2967.51     1929.01    -1038.49    -0.3500
openclaw         425   106    13096.42    1984.57    -11111.86   -0.8485
```

(Per the project convention I'll refer to the inline-completion source as
`ide-assistant-A` for the rest of this post; the digest's raw label is
the unredacted form.)

## What the lens actually measures

Every chronological-drift lens has the same problem: time-axis statistics
are easy to fool. If you compute a global mean over a window, you get one
number that sweeps everything inside the window into a single bin and
hides any drift; if you compute a slope you get a direction but the slope
is dominated by outliers in either tail; if you autocorrelate you get
day-to-day persistence but lose the absolute magnitude of the shift. The
quartile-mean-shift lens picks a deliberately blunt instrument: split each
source's rows into four chronological quartiles by `hour_start` order, take
the mean of `output_tokens` in the first quartile (Q1) and the last
quartile (Q4), and report two numbers — `meanShift = Q4Mean − Q1Mean` and
`relShift = meanShift / Q1Mean`.

This is a non-parametric drift test. It doesn't assume linearity (slope
fits do); it doesn't assume monotonicity (decay-half-life does); it
doesn't assume stationarity (most CV-based stats do). It just asks: is
the typical reply size at the *end* of the window a different number from
the typical reply size at the *beginning*?

The quartile cutoffs come from row-counts, not wall-clock time. So if a
source spent 90% of its calendar tenure idle and 10% pushing rows in a
single burst, the quartiles are still 25% / 25% / 25% / 25% of *rows*, not
hours. That's a deliberate choice — it puts equal weight on equal-sample
periods regardless of activity bursts. The downside is that for a source
with 64 rows like `codex`, a single quartile is 16 rows, and the Q1 mean
of 21,846 is being computed off 16 data points. The lens is honest about
this — it shows the `qRows` column — and the operator can decide which
sources are too small to trust. With 16 rows in either tail, Q1 means
are likely to wobble in the ±3,000-token range; the +1.94 relShift on
`codex` is *possibly* real but I'd want to see it again on twice the data
before I'd defend it.

The two sources I have *high* confidence in are `claude-code` (74 rows
per quartile) and `openclaw` (106 rows per quartile). Both are well
above the floor where quartile means stabilize, and both have spreads
that are an order of magnitude larger than any plausible Q1-Q4 sampling
noise. So the headline "claude-code grew 12.4×, openclaw shrunk to
0.15×" is a real signal.

## Reading the signs as a 2×2

The natural way to read this six-row table is as a 2×2 of (sign, magnitude):

```
                  growing reply size           shrinking reply size
                  ----------------------       ----------------------
large magnitude   claude-code (+12.38)         openclaw (-0.85)
                  codex (+1.94)
small magnitude   opencode (+0.13)             ide-assistant-A (-0.35)
                  hermes (-0.0098, ~0)
```

The four sources doing something interesting (claude-code, codex,
openclaw, ide-assistant-A) cluster by *what kind of harness they are*,
not by *what model they call*:

- `claude-code` and `codex`, the two harnesses that drive sustained
  agentic loops with explicit tool calls and long generation budgets,
  are both **growing** their typical reply size by a factor between
  1.94× and 12.38× over the window.
- `openclaw` and `ide-assistant-A`, the two harnesses with much tighter
  generation budgets — `openclaw` because it streams structured
  step-by-step replies and rarely emits long paragraphs, and
  `ide-assistant-A` because it's a single-line completion source — are
  both **shrinking** their typical reply size to 15% / 65% of where it
  started.

`opencode` and `hermes` sit in the middle: both have changes within a
factor of 1.13 either way, well within what you'd expect from drift in a
roughly stationary distribution.

## The claude-code growth: from 8.6k to 114.6k mean output

This is the largest signal in the table and it deserves more than a
shrug. The Q1 mean of 8,566 tokens per row is roughly consistent with
what the `output-size` lens has been reporting for `claude-code` over the
trailing months: a healthy mid-thousands per turn, with some long
paragraphs and some short tool calls averaged out. The Q4 mean of
**114,618 tokens** is in a different regime entirely. That's a 13× larger
typical reply.

There are three plausible mechanisms for a shift that large in a single
quartile-window:

**Mechanism A: model upgrade.** If the underlying model changed during
the window — particularly if a longer-context, longer-generation-budget
variant got rolled out — then later rows would systematically generate
larger outputs without any change in operator behavior. This corpus has
in fact seen the rollout of new model variants in the trailing weeks
(documented elsewhere on this notebook in the model-aliasing-chaos
post: 23 strings → 10 models, with several aliases collapsing onto
`claude-opus-4-7`). A new variant with a higher default `max_output_tokens`
ceiling would push the Q4 mean upward.

**Mechanism B: workflow shift toward long-form.** If the operator
gradually started using `claude-code` for tasks that *call for* longer
output — multi-file refactor narratives, long mission summaries,
generated test suites — the Q4 mean would rise even on a stable model.
The notebook's recent post about per-source weekday-share HHI also shows
that `claude-code`'s weekday distribution shifted toward Saturday
concentration in the same window, which is consistent with longer
unsupervised reasoning sessions on weekends.

**Mechanism C: reasoning_output bleed.** If `output_tokens` accounting
started including reasoning tokens for some rows partway through the
window (a common kind of provider-side accounting drift), the later
quartile would look bigger without any user-visible change. The
notebook's reasoning-token-monopoly post already shows that codex sits
at 38.6% reasoning share — there is precedent for reasoning bleed
mattering at the row level.

I cannot disambiguate A, B, and C from this single lens. Their
fingerprints are different:

- A would show a step in the Q3-Q4 boundary (the `output-size` percentile
  histogram should have a distinct second mode after the rollout date).
- B would show smooth growth through Q2 and Q3, no discontinuity.
- C would show paired drift in `reasoning-share-by-day-cv` over the same
  window.

The Q1→Q4 shift number alone is enough to *raise* the question. It is
not enough to *answer* it. That's the right level of resolution for a
6-row digest to operate at.

## The openclaw shrinkage: from 13.1k to 2.0k

The opposite end of the table is, in some ways, the more legible signal.
`openclaw`'s Q1 mean of 13,096 was already the third-highest in the
corpus. Its Q4 mean of 1,984 puts it at the bottom of the table — below
even `ide-assistant-A`'s shrinking-but-stable 1,929. And the relShift
of −0.8485 is very close to −1 (which would mean Q4 was approximately
zero), so the source has compressed to roughly **15%** of its initial
typical reply size.

What's plausibly going on here is the opposite of mechanism A above:
`openclaw` is the harness that historically had the freest generation
budget in this corpus, and the operator (per other notes on this
notebook) has been actively migrating heavy-mission workflows off it onto
other harnesses. The rows that remain at the end of the window are
disproportionately short maintenance pings — health-check-shaped queries,
status-summary requests — rather than the long-form mission outputs that
populated the early quartile. This is workflow-substitution drift,
visible as a per-source typical-output collapse, and it is exactly the
kind of finding that a quartile-mean lens is designed to surface.

A useful sanity check: if the openclaw collapse is workflow-substitution
rather than model-quality regression, the *row count* in Q4 should be
similar to Q1. It is — 106 in each, by construction. So the collapse
isn't because openclaw stopped getting called; it's because it kept
getting called for progressively shorter things. That distinction
matters for what action this signal recommends. If the row count had
also dropped, the right action would be "investigate why openclaw is
being abandoned." Because the row count is stable, the right action is
"investigate whether the *kinds* of tasks routed to openclaw have shifted
toward small ones, and whether that routing decision was deliberate or
emergent."

## hermes: the source that didn't drift

`hermes` is the source whose number is closest to "no signal": Q1 mean
of 8,232, Q4 mean of 8,151, meanShift of −80.56, relShift of −0.0098.
That's a 1% drift, which on a sample of 41 rows per quartile is well
inside the standard error of the quartile mean.

The temptation is to skip past `hermes` and call it boring. Boring is
also a finding — `hermes` is the only source in the corpus whose typical
reply size is genuinely stationary across the window. Whatever drift is
acting on `claude-code` and `openclaw` (model changes, workflow
substitution, reasoning bleed, etc.) is not acting on `hermes`. That
makes `hermes` a useful **control** for any cross-source argument: if I
want to claim that, e.g., a corpus-wide reasoning-accounting change
happened in the window, I need to explain why `hermes` didn't move and
the others did. The simplest explanation is that `hermes` doesn't go
through the same accounting path — different provider, different model,
different harness — and therefore is *upstream* of the accounting
change. That's testable.

## ide-assistant-A: a quiet shrinkage

`ide-assistant-A` shrunk by 35% (Q1 mean 2,968 → Q4 mean 1,929). On 83
rows per quartile, that's a defensible signal — not the drama of
`openclaw`, but well outside sampling noise.

A 35% shrinkage on inline completions has a natural reading: the
typical line of completed code got shorter. That can happen for two
reasons. Either the operator's coding style changed toward shorter
lines (plausible if the recent work has been more refactor-heavy and
less greenfield), or the IDE's debounce/abort logic got more aggressive
about cancelling completions before they finished, which would clip the
distribution from above without changing the count. The notebook's
companion post on `source-zero-output-row-share` tells me there are 12
fully-aborted rows on `ide-assistant-A`, which is consistent with the
abort-aggressiveness reading but doesn't decide it.

I'd want a `output-size-percentile-by-quartile` lens to disambiguate.
If the p99 dropped while the p50 stayed flat, that's clipping. If
both dropped proportionally, that's a workflow shift.

## Why this lens is worth the row it occupies

`source-first-vs-last-quartile-output-mean-shift` is doing something the
existing source-axis catalog wasn't doing well: it's a **chronological
order-preserving** statistic on per-row generation magnitude that doesn't
collapse the sequence. Every other source-axis output stat I've been
running treats the rows as exchangeable: percentile distributions,
correlation coefficients, single-day mass-concentration HHIs. None of
them can tell me whether the source is *evolving*.

The closest comparable lens is `source-daily-token-trend-slope`, which
fits an OLS line over daily totals and reports a slope. But that lens has
two weaknesses for this question: (1) it operates on `total_tokens`, not
`output_tokens` specifically, so it mixes prompt-side and generation-side
drift; (2) it assumes linearity, which means a step-function shift (like
mechanism A above) gets absorbed into a sloppy fit with low R² rather
than being surfaced clearly.

The quartile-mean lens, by being non-parametric and by separating Q1 from
Q4 with no claims about what happens in between, is structurally better
at surfacing exactly the kind of mid-window regime change that a slope
fit would miss. The cost is that it gives you no information about *when*
in the window the shift happened — you'd have to follow up with a
percentile-by-quartile breakdown to localize the drift.

## Predictions

**Prediction 1**: If I re-run this lens in two weeks, `claude-code`'s
Q4 mean will *not* sustain at 114k. Reply-size means that grow 13× in a
single window are physically improbable to sustain — there's a ceiling
imposed by the model's generation cap. I expect the next run to show
Q1 (which will be roughly the current Q4) at ~80k–110k and Q4 at the
same range, with relShift collapsing to a number under 0.4. The growth
is a *transition* between two regimes; once we're in the new regime the
shift number drops to noise.

**Prediction 2**: `openclaw`'s Q4 mean of 1,984 is close to a floor. I
don't expect it to keep dropping — at some point the source is just
short maintenance pings and there's no further to go. The next run
should show Q1 ≈ Q4 ≈ 2,000–3,000 and a relShift in the (-0.2, +0.2)
range. If it keeps falling, that means the source is actually being
deprecated rather than substituted, and the corpus will start showing
row-count collapse on the next run.

**Prediction 3**: The two sources currently at noise — `opencode` and
`hermes` — will stay at noise. They are both anchored to specific
workflows that don't appear to be evolving. If either of them shows
relShift > 0.3 on a future run, the most likely explanation will be
operator workflow change rather than tool-side drift.

## The stop-sign

The lens reports `degenerate=y` when `firstQMean` is zero, because then
`relShift` is undefined. None of the six sources tripped that flag in
this run. Worth knowing so you don't get bitten the first time a
brand-new source debuts and shows up with a Q1 of zero rows worth of
output (which would happen on a debut with all-tool-call rows in the
opening burst).

That edge case is a reminder that this lens, like every quartile-based
statistic, depends on the rows existing in the right shape at the right
time. With 1,607 rows across six sources and a min-quartile size of 16,
the corpus is in the comfortable middle of the lens's operating range.
Below ~30 total rows on a source, the lens stops reporting useful
numbers. That's the floor.

Anchor: `pew-insights source-first-vs-last-quartile-output-mean-shift`,
run at `2026-04-26T23:46:48.222Z`. 6 sources, 1,607 rows. Largest
positive relShift: `claude-code` at +12.3802 (Q1 = 8,566 → Q4 = 114,618).
Largest negative relShift: `openclaw` at −0.8485 (Q1 = 13,096 →
Q4 = 1,985).
