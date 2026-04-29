---
title: The bootstrap-CI regime shift — pew-insights v0.6.220 as the first uncertainty-quantification lens, and the six-of-six CI-contains-zero verdict on the local queue
date: 2026-04-29
---

The slope suite in `pew-insights` has been a parade of point estimators
for two weeks. Theil-Sen at v0.6.213, Mann-Kendall at v0.6.211, Siegel
at v0.6.216, the Geman-McClure M-estimator at v0.6.217, Passing-Bablok
at v0.6.218, Deming at v0.6.219 — every one of them takes the
per-source `total_tokens`-vs-row-index series and returns *one number*
per source. A slope. A point. No spread, no interval, no rejection
region. The shape of the resampling distribution behind that point
estimate has been entirely invisible.

Yesterday's release, **v0.6.220**, breaks that streak. The new
`pew-insights source-row-token-bootstrap-slope-ci` builder
(commit `8efcf01`, kernel + CLI; tests `f6fa02d`; release `94ab1d0`;
follow-up diagnostics `973b61e`) is the **first lens in the entire
slope suite that emits an interval estimate instead of a point
estimate**. The CHANGELOG entry calls it out explicitly: "the **first
uncertainty-quantification lens** in the slope suite. Every prior slope
lens — Theil-Sen (v0.6.213), Siegel (v0.6.216), Passing-Bablok
(v0.6.218), Deming (v0.6.219), the OLS daily-trend slope, the
Mann-Kendall trend test (v0.6.211), and every M-estimator location
lens (v0.6.207-v0.6.217) — emits a point estimate and nothing about
its sampling distribution."

That's a regime shift, and it's worth unpacking what changed
mechanically, what the live-smoke output reveals about the local
queue, and why the verdict it returns is a uniformly negative
one — six of six sources have a 95% bootstrap CI that straddles
zero, including the source with the largest absolute point slope.

## The mechanical change: point → interval

Every prior slope estimator in this suite is a **deterministic
function** of the input rows. Give Theil-Sen the same `(idx, tokens)`
pairs and you get the same median-of-pairwise-slopes, exactly. Give
Deming the same pairs and the same `lambda` and you get the same
maximum-likelihood EIV slope, exactly. The estimators differ in
breakdown point and influence function, but they all return *one
number* and that number is the same every time you run them on the
same data.

The bootstrap CI lens does something categorically different: it
**resamples the input** `B` times (default `B = 1000`, configurable
via `--bootstraps`, validated `>= 100`), refits the estimator on each
resample, sorts the `B` resample slopes, and reports the percentile
interval. The point estimate is still there — it's the slope on the
unresampled full data — but next to it now sit four diagnostics that
no prior lens produced:

- **`bootMean`** — the mean of the `B` resample slopes. If this is
  far from the point slope, the original fit is being driven by a
  small number of high-leverage rows that don't reliably appear in
  resamples.
- **`bootStd`** — the sample standard deviation (n−1) of the resample
  slopes. This is the empirical *standard error* of the estimator on
  this dataset, with no parametric assumption about the residual
  distribution.
- **`ciLower`, `ciUpper`** — the percentile CI at the configured
  confidence level (default 0.95). Computed as
  `sortedSlopes[(1 − confidence)/2]` and
  `sortedSlopes[(1 + confidence)/2]` with linear interpolation
  between adjacent ranks (the standard Hyndman-Fan type-7
  quantile, no Studentization, no BCa correction — pure percentile
  bootstrap).
- **`ciContainsZero`** — a boolean derived from the CI. This is the
  field that turns the lens into a hypothesis-test surface: it
  answers "is the slope statistically distinguishable from zero
  at the configured confidence level under non-parametric
  resampling?"

Determinism is preserved by a **seeded LCG**. The CHANGELOG specifies
Numerical Recipes 32-bit constants (`a = 1664525`, `c = 1013904223`,
`m = 2^32`) — the classic Press-et-al. choices. Same `--seed` (default
42), same `B`, same input rows ⇒ identical resample sequences ⇒
identical CIs. The estimator is randomized in the statistical sense
but reproducible in the engineering sense, which is the only way you
can compare two bootstrap runs across two pew queue snapshots and
attribute the difference to data rather than RNG drift.

The resampling rule is also worth flagging: the `n` indices are drawn
with replacement from `0..n-1`, then for each resample the
corresponding `(idx_k, x_k)` pairs are taken **in resample order** and
relabeled to positions `0..n-1` as the new x-axis. This is the
*standard* nonparametric bootstrap for a slope-against-row-index
estimator — it preserves the marginal distribution of `total_tokens`
but destroys the original temporal order. The reported slope is "the
slope you would have gotten if the rows had appeared in this
randomized order." That's exactly the right null hypothesis for
"does the per-row token trend depend on which rows were drawn?"

## Why the bootstrap and not the parametric SE?

A reader familiar with classical regression might ask: Deming at
`lambda = 1` is a maximum-likelihood estimator under bivariate
normal noise — there's a closed-form asymptotic standard error for
its slope. Why bootstrap?

The answer is in what the local queue actually looks like. Three of
the six sources (`opencode`, `openclaw`, `claude-code`) have shown up
in earlier slope-suite live-smoke output as having heavy right tails
on `total_tokens` — large agentic conversations that produce
single-row token counts in the millions, against medians in the
hundreds of thousands. Two slope posts in the recent backlog
explicitly track this: the trim-mean ladder post documents that
`claude-code` has a **7.47M-token asymptote** under TM30 trimming;
the winsorized-mean post documents a **3.44M-token evaporation gap**
between WM10 and WM20 on the same source. Whatever the parametric
SE for Deming says under bivariate normal noise, the local queue
*is not* bivariate normal — it has heavy tails that the asymptotic
formula does not capture.

The bootstrap doesn't require a parametric noise model. It substitutes
"resample from the empirical distribution" for "assume the residuals
are Gaussian," and on heavy-tailed data that substitution is the
entire point. The cost is `B` refits instead of one — but Deming is
closed-form (one quadratic per refit), so `B = 1000` refits on
`n ≈ 500` rows is sub-second. The cost is negligible; the assumption
relaxation is the whole win.

## The live-smoke verdict: six of six straddle zero

Here is the full live-smoke block from the v0.6.220 CHANGELOG, run
against the local `~/.config/pew/queue.jsonl` (1,949 rows after
filters, six sources, `lambda = 1`, `B = 500`, `confidence = 0.95`,
`seed = 42`, sorted `magnitude-desc`):

| source          | rows | point slope     | bootMean        | bootStd         | ciLower (2.5%)   | ciUpper (97.5%)  | ciWidth          | 0 in CI? |
| :-------------- | ---: | --------------: | --------------: | --------------: | ---------------: | ---------------: | ---------------: | :------: |
| codex           |   64 |      +2,225,990 |     −11,536,657 |     218,933,003 |      −88,594,742 |      +74,910,047 |      163,504,789 |   yes    |
| opencode        |  437 |      −1,139,377 |      +2,136,928 |      38,548,252 |      −33,367,939 |      +41,104,521 |       74,472,460 |   yes    |
| claude-code     |  299 |        +412,420 |      +2,428,933 |      48,559,003 |      −43,964,898 |      +63,420,014 |      107,384,912 |   yes    |
| openclaw        |  543 |         −88,809 |        +382,329 |       8,376,411 |       −8,639,503 |      +10,261,084 |       18,900,588 |   yes    |
| hermes          |  273 |         −33,100 |        −312,377 |       7,903,563 |       −2,656,371 |       +2,809,665 |        5,466,036 |   yes    |
| vscode-redacted |  333 |            +761 |            +354 |          47,075 |          −33,663 |          +34,350 |           68,012 |   yes    |

All six sources land in the `0 in CI? = yes` column. The CHANGELOG's
own commentary calls this out without softening: "**Every source on
the local queue has a bootstrap CI that straddles zero at 95%
confidence** (`ciContainsZero = yes` for all 6). That is the
bootstrap's verdict: under the non-parametric resampling distribution
of the Deming slope on this data, *not one* of the per-source point
slopes — not even `codex` at +2.2M tokens/row — is statistically
distinguishable from zero at 95% confidence on `B = 500` resamples."

This is a striking result and it's worth saying clearly: every prior
slope post in this notes corpus that quoted a per-source slope as a
trend signal was implicitly making the claim that the trend was
*real*. The bootstrap CI is the first lens that puts a probabilistic
qualifier on that claim, and on this dataset, the qualifier comes
back as "no, the trend is not robust to which rows you drew."

## Reading the four extreme cases

Each row in the live-smoke table tells a different story about how
the bootstrap distribution interacts with the empirical data, and
the per-source structure is worth walking.

**`codex` (n = 64, point +2.23M, bootMean −11.54M, bootStd 219M)**.
This is the textbook signature of a small-n, high-leverage failure.
The point slope on the full 64 rows is positive and large, but the
bootMean across resamples is *negative* and seven times larger in
absolute value than the point. That happens when one or two extreme
rows (very large `total_tokens` near the top or bottom of the index
range) are pulling the full-data Deming fit hard in one direction;
when those rows happen to be excluded from a resample, the slope
flips to the bootMean's neighborhood. With only 64 rows, the
probability that any single high-leverage row is excluded from a
bootstrap resample is `(1 − 1/64)^64 ≈ 36.6%`, so a substantial
fraction of resamples see a *very* different fit than the full data.
The CI width of 163M is comically large relative to the point of
2.2M — a CI/point ratio of ~73. The honest report of this row is
"the point slope is not interpretable; collect more data."

**`opencode` (n = 437, point −1.14M, bootMean +2.14M)**. The point and
bootMean are not just different magnitudes — they're *opposite signs*.
This is the same high-leverage pattern as `codex` but with a much
larger sample (437 rows), so the bootstrap can move the slope without
the resamples being dominated by the absence of one or two rows. The
interpretation: a small subset of the 437 rows is doing all the slope
work, in the negative direction, and most resamples that omit some
of those rows pull the slope up to a small positive trend. Either
way, the CI of [−33.4M, +41.1M] straddles zero by a wide margin.

**`claude-code` (n = 299, point +0.41M, bootMean +2.43M, same sign)**.
Sign agreement but the bootMean is roughly six times the point.
That's the milder version of the leverage signature — the resampling
distribution is centered well above the full-data fit, but at least
in the same half-line. CI of [−44M, +63M] is still firmly straddling
zero. The interpretation is that there *is* a positive trend that
keeps surviving resampling, but the spread is enormous.

**`vscode-redacted` (n = 333, point +761, bootStd 47K, ciWidth 68K)**.
The cleanest source by absolute scale — the entire CI is on the order
of tens of thousands of tokens, not millions. This source's
`total_tokens` distribution is much tighter (the median is 2,319
tokens per row per the v0.6.217 Geman-McClure live-smoke), so the
bootstrap distribution of the slope is correspondingly tight. But
even here, the CI of [−33.7K, +34.3K] straddles zero — the point
slope of 761 tokens/row is well within one bootStd of zero, and the
honest verdict is "no detectable per-row token trend on this source
either."

The pattern across all six is the same: the **point slope tells you
about the observed series, not about the population that generated
it**. With only a few hundred rows per source and heavy right tails
on `total_tokens`, the resampling distribution of the slope is wide
enough that zero sits inside the 95% interval in every single case.

## What this changes about the prior slope-suite posts

Reading back through the slope-suite posts in this notes directory,
several of them quote per-source slope numbers as trend evidence —
e.g., the Theil-Sen-vs-Mann-Kendall divergence post on claude-code
(+105,180 tokens/row Theil-Sen, +35K/row gap to the OLS endpoint),
the Siegel-vs-Theil-Sen breakdown-jump post (29% → 50% breakdown
point), the Passing-Bablok sign-flip cohort post (claude-code and
hermes as the two direction-disagreers). All of those analyses are
still valid as **descriptions of what the chosen estimator does on
the observed series**. None of them are invalidated.

What v0.6.220 adds is a meta-claim that no prior post could make:
**the slope numbers themselves are not statistically distinguishable
from zero on the local queue under non-parametric resampling**. That
is not the same as "the slopes are wrong" — the point estimators are
all correctly computed. It's a claim about the *information content*
of those point estimates relative to the noise floor, and on this
dataset the bootstrap returns a uniformly low information content
verdict.

The right operational reading is: *if* you want to use a slope
estimator from the suite to make a forward-looking prediction about
how `total_tokens` will evolve at row `n+1`, the bootstrap CI is
telling you that the prediction is dominated by noise. *If* you want
to use the slope as a descriptive summary of the observed series
("the trend you would draw if you were eyeballing it"), the point
estimators remain the right tool and the bootstrap CI is orthogonal
to that use case.

## What v0.6.220 doesn't do (and what would come next)

The percentile bootstrap is the simplest of the bootstrap CI families.
It has known biases on skewed distributions and small samples — the
CI doesn't correct for the difference between the bootstrap median
and the point estimate, and it can have poor coverage when the
estimator is biased. The two standard refinements would be:

1. **BCa (bias-corrected and accelerated)** intervals, which add a
   bias correction term and an acceleration term derived from the
   jackknife. These have asymptotically correct coverage on skewed
   estimators and would be a natural follow-on for the same lens.
2. **Studentized bootstrap** (the "bootstrap-t"), which divides the
   resample slopes by their own bootstrap-estimated standard errors.
   These have second-order accuracy and would tighten the CIs
   substantially on the cleaner sources like `vscode-redacted`.

Neither is in v0.6.220. The CHANGELOG entry doesn't promise either as
a follow-up. But once the percentile bootstrap is in the codebase,
both refinements become natural extensions — the resampling
infrastructure is identical, only the post-resample summary
calculation changes.

The other thing v0.6.220 doesn't do is provide a **block bootstrap**
for the time-ordered structure. The current resampling treats rows as
exchangeable — it shuffles the row order before refitting. If
`total_tokens` has serial autocorrelation (rows close in time are
more similar), the standard bootstrap will *underestimate* the CI
width because resampling destroys the autocorrelation that produced
some of the variance. A moving-block bootstrap that resamples
contiguous chunks of rows would correct for this. Whether the local
queue actually has serial autocorrelation worth correcting for is an
empirical question that none of the lenses currently answers — but
it's the obvious next diagnostic to ship.

## Test-coverage anchor

The release also crossed a test-count milestone worth recording:
**5,450 → 5,526 (+76 tests)** for the new `sourcerowtokenbootstrapslopeci`
suite, per the CHANGELOG live-smoke block. That's the largest
single-feature test addition in the slope suite this week (Geman-
McClure at v0.6.217 added 26; Passing-Bablok at v0.6.218 added a
similar count). The breakdown the CHANGELOG gives — kernel coverage
(`percentileSorted` empty/single/edge cases, `makeLcg` determinism,
`bootstrapResample` length/membership/seeded-determinism,
`bootstrapDemingSlope` n<2/all-equal/ascending/descending/lambda
propagation) plus builder validation, data flow, bootstrap math,
and `ciContainsZero` refinement — is heavier than usual because the
percentile bootstrap has more failure modes than a closed-form point
estimator. The math has to be right not just for "compute a slope"
but for "compute a slope on each of B resamples, sort them, and
interpolate the percentile correctly."

## What the next slope-suite release should ship

If the suite continues at the same cadence (eleven releases v0.6.207
→ v0.6.217 in one week per the M-estimator march post, plus
v0.6.218–v0.6.220 in the days since), the natural next steps are:

- A bootstrap CI variant for the **Theil-Sen** and **Siegel** point
  estimators, not just Deming. The same resampling infrastructure
  applies; only the kernel changes.
- A **block bootstrap** option for any of the slope CIs, to handle
  serial autocorrelation in the time-ordered queue.
- A **BCa** or **studentized** refinement on the existing Deming
  bootstrap, to tighten the intervals on the cleaner sources.

None of these are essential — v0.6.220 already provides the regime
shift from point to interval, which is the categorically novel
addition. But each would extend the uncertainty-quantification
lens in a direction the suite hasn't yet covered.

## Citations

- pew-insights `CHANGELOG.md` v0.6.220 entry, lines 1–95 (feature
  description, live-smoke table, six-of-six straddle-zero verdict).
- pew-insights commit `8efcf01` — `feat: source-row-token-bootstrap-slope-ci builder + CLI`.
- pew-insights commit `f6fa02d` — `test: source-row-token-bootstrap-slope-ci coverage`.
- pew-insights commit `94ab1d0` — `chore: release v0.6.220`.
- pew-insights commit `973b61e` — `feat: bootMedian + bootSkewMeanMinusMedian diagnostics + boot-skew-magnitude-desc sort` (v0.6.220 follow-up).
- Test count growth `5,450 → 5,526 (+76)` per the v0.6.220 CHANGELOG
  test block.
- Live-smoke configuration: `~/.config/pew/queue.jsonl`, 1,949 rows
  after filters, six sources, `lambda = 1`, `B = 500`,
  `confidence = 0.95`, `seed = 42`, sort `magnitude-desc`.
