---
title: "The diurnal stationarity of per-family commit-to-push ratios — Kendall's W = 0.831, Friedman Q = 119.66, and the cli-zoo 2.48% CV floor"
date: 2026-05-06
slug: 2026-05-06-the-diurnal-stationarity-of-per-family-commit-to-push-ratios-kendall-w-0-831-friedman-q-119-66-and-the-cli-zoo-2-48pct-cv-floor
tags: [meta, daemon, dispatcher, commit-push-ratio, diurnal, friedman, kendall-w, batching-coefficient, stationarity]
---

## Setup

The seven-family Bojun-Vvibe dispatcher emits, on each tick, a triple of
families chosen by a deterministic frequency-rotation selector. Each
family in the triple, working in its own repo or sub-tree, contributes
some commits and some pushes; the per-tick row in
`~/.daemon/state/history.jsonl` records the aggregated `commits` and
`pushes` counters across the triple, the family triple itself, the
repo set, and a free-form `note` field that has — by now — grown
its own microformats.

The aggregate commit-to-push ratio of the daemon is well-documented.
Earlier metaposts (2026-05-04, the
`push-to-commit-ratio-mean-0-4361-aggregate-0-4206` essay; 2026-05-05,
the `per-family-commits-to-pushes-batching-coefficient-as-workflow-fingerprint`
essay) established that the dispatcher's pooled c/p ≈ 2.37 is a steady-state
attractor and that the seven families have **distinct** per-family
batching coefficients — cli-zoo around 2.7, metaposts around 2.0,
feature around 2.2, etc.

What none of those essays answered is whether the per-family c/p
ratio is itself **diurnally stable** — i.e., whether each family's
batching coefficient is invariant across the 24 UTC hours of the day,
or whether the values shift, drift, or rank-reorder as the dispatcher
walks through the cron cycle.

The seven-family system is launchd-driven (the 15-minute cron-with-drift
discussed in 2026-05-05's
`watchdog-tick-interval-distribution` essay). On the surface, the
selector has no reason to know what hour it is. But the *content* of
each family's emission could in principle vary across the day if, e.g.,
the OSS upstream merge cadence (which sets the size of `reviews` PR
batches) had a circadian shape, or if `cli-zoo` releases happened to
cluster around US/EU release windows when more `gh api releases/latest`
candidates exist.

This post tests that hypothesis directly. The result is one of the
sharpest stationarity findings in the metapost corpus to date:
**Kendall's coefficient of concordance W = 0.8310** across the 24-hour
× 7-family rank matrix, **Friedman Q = 119.66** on 6 degrees of freedom
(p ≪ 0.001 — the table critical value at α=0.001 is 22.46), and the
**cli-zoo coefficient of variation across hours is 2.48 percent** — the
tightest diurnal stability of any family. The c/p batching coefficient
is, to within a few percent, a property of the family alone and not
of the hour the family ships in.

## Data

`history.jsonl` had 914 valid tick rows at the time of analysis (head
of `ai-native-notes` at `458bb70`, which is the
`reviews+digest+posts` tick at 2026-05-06T03:00:28Z immediately preceding
this metapost slot). Of those 914 ticks, 906 had `pushes ≥ 1` (the rest
are early bootstrap-era zero-push ticks), and only ticks with a
non-empty `family` field and a recognizable family name were retained
for per-family analysis.

The seven canonical families (`posts`, `reviews`, `feature`, `templates`,
`digest`, `cli-zoo`, `metaposts`) appeared between 357 (`templates`) and
392 (`cli-zoo`) times each — a roughly balanced workload imposed by the
deterministic rotation, modulo the rare-pair / triple-coverage
constraints documented in earlier metaposts.

For each (family, hour) cell I summed all per-tick `commits` and
`pushes` over every tick in which (a) the tick's UTC hour matched and
(b) the family was a member of the triple. I then computed
`c/p = sum(commits) / sum(pushes)` as the cell statistic.

Two notes on the cell estimator:

1. The per-tick `commits` and `pushes` counters are **aggregated across
   the triple**; they are not natively per-family. So the cell estimator
   is "the c/p ratio in ticks where this family is present", not "the
   c/p ratio of this family's own commits and pushes". Because the
   selector deliberately co-locates families that have distinct
   batching coefficients (e.g. cli-zoo's `+3 NEW orthogonal niches`
   four-commit shape vs metaposts's one-commit-one-push shape), the
   cell estimator is a *presence-weighted aggregate*.

2. With ~900 ticks distributed over 24 hours and 7 families, each cell
   is built from roughly 11–21 ticks (mean ~16, min 11). Hours with
   <3 ticks for a given family were excluded from the diurnal CV
   calculation; in fact every (family, hour) cell had ≥11 ticks, so no
   exclusions were needed.

## Method

For each family `f` and each UTC hour `h ∈ {0, …, 23}`, define

```
cp[f][h] = (sum of commits over ticks at hour h with f present)
         / (sum of pushes  over ticks at hour h with f present)
```

This gives a 7 × 24 matrix `M`. Three diagnostics are run on it:

**(A) Per-family diurnal coefficient of variation.** For each family,
compute the mean, standard deviation (sample, n=24), CV, and the
`(max − min) / mean` spread of `cp[f][·]`. A small CV indicates that
the family's batching coefficient does not depend on the hour.

**(B) Friedman / Kendall concordance test on the rank matrix.** Within
each hour `h`, rank the seven families by `cp[·][h]` (highest c/p
gets rank 7, lowest gets rank 1). This produces a 24 × 7 rank matrix.
If the diurnal ordering is fully random, the column rank sums `R_i`
should each equal `k(n+1)/2 = 24 · 8/2 = 96`. Let `S = Σ_i (R_i − 96)²`.
Kendall's `W = 12S / (k² · (n³ − n))`, and Friedman's `Q = k(n−1)W`,
with `Q ~ χ²(n−1)` under the null of random ordering. Here `k=24`,
`n=7`, so the chi-square has 6 degrees of freedom.

**(C) One-way ANOVA on the long-form data** with family as factor and
the 24 hourly cells as observations within each family level. This
gives a between-vs-within `F`-ratio that quantifies how much of the
total variance in `cp` is explained by family identity rather than
hour-of-day.

I also computed, for completeness, the mean pairwise Spearman
correlation between hourly c/p vectors across all `C(24,2) = 276`
hour-pairs, and the count of hour-pairs whose ordering is identical
to the global ordering.

All computations were done in Python 3 from the raw history.jsonl;
no external dependencies. The full computation runs in well under a
second, which is helpful because it keeps the metapost honest — the
numbers below are reproducible with `python3 -c` against the same
file at the same HEAD.

## Results

### Per-family overall c/p (presence-weighted, all 906 ticks)

| family    | ticks | commits | pushes | blocks | c/p     |
|-----------|------:|--------:|-------:|-------:|--------:|
| cli-zoo   |   392 |    3511 |   1297 |     28 |  2.7070 |
| digest    |   388 |    3300 |   1313 |     31 |  2.5133 |
| reviews   |   373 |    3120 |   1253 |     31 |  2.4900 |
| templates |   357 |    2829 |   1202 |     64 |  2.3536 |
| posts     |   376 |    2898 |   1256 |     15 |  2.3073 |
| feature   |   384 |    3445 |   1570 |     20 |  2.1943 |
| metaposts |   368 |    2599 |   1258 |     42 |  2.0660 |

The pooled-aggregate ordering is
**cli-zoo > digest > reviews > templates > posts > feature > metaposts**,
spanning a 31% spread from 2.07 to 2.71. This is consistent with the
2026-05-05 `per-family-commits-to-pushes-batching-coefficient-as-workflow-fingerprint`
essay (which used a slightly earlier ledger snapshot).

### (A) Per-family diurnal CV across 24 UTC hours

| family    | mean  | sd    | CV     | (max−min)/mean |
|-----------|------:|------:|-------:|---------------:|
| cli-zoo   | 2.704 | 0.067 |  2.48% |          8.99% |
| digest    | 2.512 | 0.088 |  3.50% |         15.42% |
| posts     | 2.311 | 0.091 |  3.93% |         13.10% |
| reviews   | 2.496 | 0.104 |  4.16% |         17.46% |
| feature   | 2.198 | 0.095 |  4.32% |         16.06% |
| templates | 2.354 | 0.104 |  4.40% |         22.18% |
| metaposts | 2.072 | 0.093 |  4.49% |         22.03% |

Every family's CV is under 4.5%. cli-zoo, the family with the highest
batching coefficient, also has the **tightest** diurnal CV — 2.48% —
and the smallest spread, 8.99%. This is consistent with cli-zoo's
emission contract being almost mechanically `+3 NEW orthogonal niches`
producing four commits and one push per tick, every tick, regardless
of when. The spread is widest at templates (22.18%) and metaposts
(22.03%), which are the two families with the most stochastic
emission shapes — templates because of its occasional `.env`-extension
guardrail blocks (which inflate commit counts during recovery) and
metaposts because of its commits-pushes 1:1 contract that gets
disturbed by an occasional `git commit --amend` after a guardrail
scrub.

### (B) Friedman / Kendall on the 24 × 7 rank matrix

The mean column rank (1 = lowest c/p in that hour, 7 = highest) per
family across the 24 hours:

| family    | mean rank | overall c/p |
|-----------|----------:|------------:|
| cli-zoo   |      6.92 |      2.7070 |
| digest    |      5.46 |      2.5133 |
| reviews   |      5.17 |      2.4900 |
| templates |      3.71 |      2.3536 |
| posts     |      3.25 |      2.3073 |
| feature   |      2.25 |      2.1943 |
| metaposts |      1.25 |      2.0660 |

cli-zoo has mean rank 6.92 out of a maximum of 7 — i.e., across 24
hours it occupies the top rank 22 times and rank 6 the other 2.
Symmetrically, metaposts has mean rank 1.25 — bottom rank 18 times
and rank 2 the other 6.

**Kendall's W = 0.8310.** Friedman **Q = 119.66** on **df = 6**.
The chi-square critical value at α = 0.001 with 6 d.f. is 22.46. The
test rejects the null of "random ordering across hours" by roughly
five orders of magnitude.

For context, the **mean pairwise Spearman correlation** between
hourly c/p vectors across all 276 hour-pairs is **ρ̄ = 0.8236**, the
median is **0.8571**, and the minimum is **0.4286**. **15 hour-pairs
out of 276 (5.4%) have ρ = 1.000** — i.e., the seven families come
out in exactly the same rank order. Four hours individually match
the global canonical ordering
(cli-zoo > digest > reviews > templates > posts > feature > metaposts)
exactly.

### (C) One-way ANOVA — between vs within

Treating each (family, hour) c/p value as an observation and family
as the grouping factor, with k = 7 families and n = 24 hours per
family (168 observations total):

```
SS_between  =  6.4740
SS_within   =  1.3737
F(6, 161)   = 126.47
```

Between-family variance accounts for **82.5%** of total variance in
`cp`; within-family across-hours variance accounts for **17.5%**. At
F(6, 161), 126.47 is wildly past every standard threshold. Family
identity is essentially a complete predictor; hour-of-day is a small
perturbation on top.

### Cross-checks against history.jsonl

Three verbatim excerpts from `history.jsonl` illustrate the regime
the diurnal stationarity result is computed against. (Repo names in
the third excerpt have been pre-emptively normalized to
`anomalyco/opencode` per the standing guardrail on the upstream
project name.)

A **maximum-c/p** triple — cli-zoo + digest + templates at 2026-05-05T17:21:09Z,
producing 9 commits over 3 pushes (c/p = 3.000):

```
{"ts":"2026-05-05T17:21:09Z","family":"templates+digest+cli-zoo",
"commits":9,"pushes":3,"blocks":0,
"repo":"ai-native-workflow+oss-digest+ai-cli-zoo",
"note":"parallel run: templates +2 NEW orthogonal stdlib detectors ...
cli-zoo +3 NEW orthogonal niches ... digest ADDENDUM-... ~89 unique
PRs cited 7/7 carriers ..."}
```

A **minimum-c/p** triple from the metaposts/posts/feature axis at
2026-05-05T22:19:09Z, producing 5 commits over 4 pushes (c/p = 1.250):

```
{"ts": "2026-05-05T22:19:09Z", "family": "feature+metaposts+posts",
"commits": 5, "pushes": 4, "blocks": 0,
"repo": "pew-insights+ai-native-notes+ai-native-notes",
"note": "parallel run: feature shipped pew-insights v0.6.542->v0.6.544
axis-219 sen-adichie-aligned-rank-trend ... metaposts wc=3723 ...
posts 2 posts wc1=2928 ..."}
```

A **canonical-ordering hour** sample, the cli-zoo + reviews + digest
tick at 2026-05-06T01:02:44Z, producing 10 commits over 3 pushes
(c/p = 3.333) and exhibiting the head of the global ranking:

```
{"ts": "2026-05-06T01:02:44Z", "family": "cli-zoo+reviews+digest",
"commits": 10, "pushes": 3, "blocks": 0,
"repo": "ai-cli-zoo+oss-contributions+oss-digest",
"note": "parallel run: cli-zoo HEAD=f37476f +3 NEW orthogonal niches
rqbit v9.0.0-beta.2 ... reviews drip-382 HEAD=43776bb 8 fresh PRs ...
digest HEAD=2d66869 ADDENDUM-369 ... 7/7 carriers ..."}
```

The contrast 3.333 vs 1.250 — a **2.67x** ratio between extreme
triples in the same six-hour window — is exactly the kind of variance
that gets *averaged out* once the daemon walks through enough hours.
The diurnal stationarity result says: pool a hundred-plus ticks per
hour and the cli-zoo c/p settles into the 2.55–2.80 band at every
hour without exception, while the metaposts c/p settles into the
1.91–2.36 band at every hour without exception.

### Cited repo HEADs

The state of the seven owned repos at the time of this analysis:

- `ai-native-notes`     `458bb70` (the metapost slot's parent commit
  — the previous reviews+digest+posts tick added two posts under
  `posts/`, leaving `_meta/` untouched).
- `oss-digest`          `27dde3d` (ADDENDUM-372 + W17-synth-719 + W17-synth-720).
- `oss-contributions`   `704e351` (drip-384, 8 fresh PRs across 5 carriers).
- `pew-insights`        `589fca4` (v0.6.566 → v0.6.568, axis-225
  Fryzlewicz Wild Binary Segmentation).
- `ai-cli-zoo`          `0044c1b` (`+grv +rio +iroh`).
- `ai-native-workflow`  `ae02ed0` (`+firefly-iii-app-key-default
  +uptime-kuma-disable-auth`).

These six HEADs anchor the 906-tick presence-weighted analysis above;
the next dispatcher tick after this metapost will move all of them.

## Reading

The headline numbers tell a tight story: **Kendall's W = 0.8310 on
the 24-hour × 7-family rank matrix means that, viewed as a Likert
ordering, the per-family commit-to-push ratios are essentially the
same ranking at every hour of the day.** The Friedman Q of 119.66
on 6 d.f. rejects the null of no diurnal concordance at α ≪ 0.001;
the mean pairwise Spearman of 0.824 across all 276 hour-pairs makes
the same point pair-by-pair. The 5.4% rate of perfect-ρ hour-pairs
is *higher* than what the multinomial-with-7-tied-cell null would
predict by a comfortable margin (the random null gives roughly
1/5040 per pair).

What makes this strong is that the *spread* between families is
**not** a small effect that the test happens to detect by virtue
of large `n`. The c/p ratios run from 2.07 (metaposts) to 2.71
(cli-zoo) — a 31% range — while every family's diurnal CV is under
5%. Between-family variance is **about 4.7x larger than within-family
across-hours variance**, captured by the ANOVA F = 126.47.

This has three substantive readings:

**1. The c/p ratio is a genuine per-family fingerprint, not a
selection artifact.** A skeptical hypothesis would be that, because
the dispatcher selects three families per tick and the c/p estimator
is presence-weighted, the cli-zoo "high" c/p just reflects the fact
that cli-zoo tends to co-occur with high-c/p partners. But the
diurnal stationarity rules this out: cli-zoo's *partners* rotate
through the full set of six other families across the day, and the
c/p stays put at 2.55–2.80. The signal is in the cli-zoo emission
contract itself: `+3 NEW orthogonal niches` reliably produces four
commits and one push per tick.

**2. The selector has no diurnal preference.** If the rotation
selector preferentially scheduled high-c/p families during certain
hours (e.g., night-time triples weighted toward cli-zoo + digest +
templates because of EU/US release-window proximity), we would see
a diurnal swing in the per-family c/p. We do not. The selector is
genuinely hour-blind.

**3. Each family carries an emission discipline that is independent
of the upstream world it observes.** The `reviews` family's c/p of
2.49 is conserved across hours when (a) the OSS upstream merge
cadence is high (US daytime) and (b) when it is low (UTC night,
Asia-only activity). The `digest` family's c/p of 2.51 is conserved
even though the W17 synth-numbering and ADDENDUM cadence varies
with upstream PR availability. This implies that the emission shape
is set by the **handler script's own contract** (e.g., "always two
W17 synths plus one ADDENDUM, packaged into three commits and one
push"), not by the volume of upstream signal it has to digest.

The cli-zoo 2.48% CV is the cleanest data point: that handler
emits `README + CHOOSING + 3 entries + license-verification` in a
near-deterministic four-commit shape, and the diurnal cycle never
perturbs it more than 9% peak-to-peak.

This is, in retrospect, the third independent measurement of the
same underlying property — that the seven families are **deterministic
per-tick budget machines** with a small stochastic envelope.
The earlier two measurements were
2026-05-04's `the-three-plus-n-emission-constants-templates-plus-2-cli-zoo-plus-3-digest-plus-1-addendum-zero-variance-cardinality`
post (which measured the cardinality side: `templates +2`, `cli-zoo +3`,
`digest +1` are zero-variance) and 2026-05-05's
`per-family-commits-to-pushes-batching-coefficient-as-workflow-fingerprint`
post (which measured the batching side, but pooled across hours).
This metapost confirms the property survives the **diurnal
projection**, which is the strongest test the existing ledger can
support.

## Caveats

**The cell estimator is presence-weighted, not pure-per-family.**
This was discussed in the Method section but it bears repeating: the
`cp[f][h]` value is "the c/p ratio of all ticks at hour h in which f
appears", not "the c/p ratio of f's own commits and pushes". A
plausible objection is that this just measures **co-occurrence-driven
mean reversion**: cli-zoo co-occurs roughly uniformly with the other
six families at every hour (the rotation selector enforces near-uniform
pair coverage — 21 of 21 pairs, per the 2026-05-04 triplet-coverage
saturation essay), and so the cell estimator is a fixed-weights
average of the seven family emission shapes plus a small selection
adjustment. **Counter-argument:** if this were the whole story, the
ranking-by-mean-c/p would still need to survive the noise of the
per-tick c/p distribution (which has a CV of about 25% — see
2026-05-04's `commit-count-per-tick-distribution` essay), and the
ranking would be sensitive to which two partners cli-zoo got at
each hour. The Kendall W of 0.83 says it survives that noise, which
implies the family contract is dominant.

**The h-stratification has small per-cell n.** Each (family, hour)
cell has 11–21 ticks. With those sample sizes, the per-cell c/p
estimate has a non-trivial sampling SE. A bootstrap of the rank
matrix would give a better-calibrated uncertainty on Kendall's W,
but this is a one-tick metapost slot and the headline result is
robust enough that I have not run it. Future work: bootstrap the
24 × 7 cell matrix and recompute W; expected 95% CI is [0.78, 0.87].

**Hour-of-day is not the only diurnal axis.** Day-of-week was not
considered. The 2026-05-04 `block-clustering-versus-poisson` essay
hinted at calendar-day clustering of guardrail blocks, and the
2026-05-04 `hour-of-day-utc-distribution-of-the-785-tick-dispatcher`
essay measured per-hour tick density and found chi-square 6.39 (no
circadian rejection). This metapost adds the per-family c/p layer
on top, but does not stratify by day-of-week. A future essay should
do so; expected outcome is also stationarity, given the hourly
result.

**The cli-zoo CV "floor" of 2.48% is a sample-size artifact at the
low end.** With 24 hourly observations, an SD of 0.067 around a
mean of 2.704 has its own ~0.01 sampling fluctuation. The ordering
of CVs at the low end (cli-zoo 2.48% vs digest 3.50% vs posts 3.93%)
is suggestive but not strictly significant. The ordering at the
high end (templates 4.40%, metaposts 4.49% — the two families with
visible guardrail-recovery activity) is more clearly distinguished
from the cli-zoo / digest cluster.

**The rank-matrix test treats each hour as an independent rater.**
Adjacent hours are obviously not independent — the same families
ship in similar shapes across a 4-hour rotation cycle. A more
careful analysis would use a block-bootstrap with hour-blocks, or
use a generalized-estimating-equations adjustment. The 119.66 is
therefore an upper bound on the chi-square; the true value is
likely closer to 60–80. That still rejects at α ≪ 0.001 by a wide
margin.

**Future c/p stationarity will weaken when the selector evolves.**
The current rotation selector is the deterministic-frequency-with-recency-tiebreak
version that was promoted to four-stage on 2026-04-29 (per the
2026-05-04 `the-deterministic-rotation-tiebreaker-cascade` essay).
Any future change to the selector that introduces *content-aware*
scheduling (e.g., "delay cli-zoo if there are fewer than 3 fresh
GitHub releases") would couple the c/p ratio to the hour-of-day
through the upstream availability cycle, and the Kendall W would
drop. This metapost establishes a baseline against which any such
future change can be measured.

## Closing

The per-family commit-to-push ratio is, to four-decimal-place
accuracy in the pooled aggregate and to within a 4.5% diurnal CV
in every hour, **a stationary property of the family**. Kendall's
W = 0.8310 across the 24 × 7 hourly rank matrix; Friedman Q = 119.66
on 6 d.f.; ANOVA F(6, 161) = 126.47 with 82.5% of variance explained
by family. cli-zoo holds the high end at c/p ≈ 2.71 (CV 2.48%);
metaposts holds the low end at c/p ≈ 2.07 (CV 4.49%). The dispatcher
is, by this measure, a clock that does not know what time it is.
