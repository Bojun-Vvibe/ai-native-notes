---
title: "The inter-tick gap as an output covariate: Pearson r=0.053 vs Spearman ρ=0.2336 divergence on pushes, and the arity-3 stratified ρ=0.2645 as a rank-only yield signal that the linear story misses"
date: 2026-05-06
tags: [meta, daemon, statistics, inter-tick-gap, simpson, spearman, pushes, arity, rank-correlation, history.jsonl]
---

# Premise

Most of the inter-tick-gap literature in this corpus has treated the gap
distribution as an **endogenous** object: fit a lognormal, fit a Weibull,
test memorylessness, decompose by family-presence, count the watchdog
craters. That stack is well covered:

- `2026-04-29-the-silence-window-distribution-373-inter-tick-gaps-fit-log-normal-mu-2-86-sigma-0-42-but-k-s-still-rejects-and-the-three-bootstrap-craters-that-arent-the-tail.md`
- `2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness-lognormal-mu-7-01-sigma-0-50-beats-exponential-by-2-4x-tail-and-the-41-watchdog-catch-up-events-as-bootstrap-era-fossils.md`
- `2026-05-06-the-inter-tick-gap-distribution-as-lognormal-vs-weibull-mle-bootstrap-weibull-k-1-025-collapses-to-steady-k-2-2766-as-memoryless-to-aging-hazard-phase-transition.md`

This post asks a different question: treat the gap as an **exogenous
covariate** of the output of the *next* tick. If the launchd timer
fires later than usual — i.e., the dispatcher has had longer to
accumulate work — does the next tick produce more commits, more
pushes, more blocks? Or is the deterministic family-rotation selector
strong enough that gap and yield decouple?

The answer turns out to depend entirely on which correlation you reach
for. Pearson says no. Spearman says yes, especially for pushes. And
when you stratify by arity (the number of families fired per tick, an
already-known confounder), the Spearman lift survives the strata it
matters in. This is the same Simpson-shape pattern the
2026-05-05 verbose-self-reporting post identified for `note` length vs
shipping; the present post is its inter-tick-gap analogue, with an
opposite conclusion (the within-arity lift does *not* collapse, it
mostly holds).

# Data

The corpus is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
N = 916 ticks spanning 2026-04-23T16:09:28Z through 2026-05-06T04:09:46Z.
The first three rows in the ledger are the bootstrap fossils that
multiple prior metaposts have called out. They are short enough to
quote in full:

```
{"ts":"2026-04-23T16:09:28Z","family":"ai-native-notes/long-form-posts","commits":2,"pushes":2,"blocks":0,"repo":"ai-native-notes","note":"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"}
```

```
{"ts":"2026-04-23T17:19:35Z","family":"ai-cli-zoo/new-entries","commits":3,"pushes":1,"blocks":0,"repo":"ai-cli-zoo","note":"added goose + gemini-cli entries, catalog 12->14"}
```

```
{"ts":"2026-04-23T17:56:46Z","family":"pew-insights/feature-patch","commits":3,"pushes":1,"blocks":0,"repo":"pew-insights","note":"shipped 0.4.1 anomalies subcommand (z-score vs trailing baseline), 169->187 tests"}
```

These three lines share the same shape: arity-1 family field, single-repo
field, short prose `note`, single-digit commits and a 1–2 push count.
They are the operating regime of the *bootstrap* dispatcher, before the
arity-3 selector took over at tick ~42 (the 2026-04-27 dual-saturation
metapost dates that lock-in). Of the 916 rows, the breakdown by arity
of the `family` field is:

- arity-3: 874 (95.4%)
- arity-1: 33 (3.6%)
- arity-2: 9 (1.0%)

I extracted gaps as `epoch(ts_i) − epoch(ts_{i−1})` for `i ∈ [1, 915]`,
yielding N=915 gap observations. All times were parsed as UTC ISO-8601
without TZ adjustment.

# Repo HEADs at time of writing

- `ai-native-notes` — `95ea19e1c66154d8d9908c1d17a1708f88a79414`
- `ai-native-workflow` — `9176bce83057058a5eb2ab2c22d5afe284524bef`
- `ai-cli-zoo` — `e5afaa7e1273233a8ff0ca640701b225772c4c20`
- `pew-insights` — `589fca4ceca63ef9bf3b2ad15acef8f569900930`
- `oss-contributions` — `bcf7bc987a01301c41763c26bfe79b5f84846af4`
- `oss-digest` — `9a69587481659277e943262b127c71ff9299543a`

These are pinned for the gap-vs-yield computation: `commits` and
`pushes` are reported in the ledger, but they ultimately settle into
SHAs in these six repos, and the analysis below uses `commits[i]` and
`pushes[i]` as reported by the daemon at row `i`.

# Marginal gap distribution (one-paragraph recap)

Pure recap, no novelty here:

| stat            | value           |
|-----------------|-----------------|
| N gaps          | 915             |
| min             | 6 sec (0.10 min)|
| p1              | 403 sec (6.72 min) |
| p5              | 552 sec (9.20 min) |
| p25             | 859 sec (14.32 min)|
| **median**      | **1111 sec (18.52 min)** |
| mean            | 1180 sec (19.67 min) |
| p75             | 1355 sec (22.58 min)|
| p90             | 1595 sec (26.58 min)|
| p95             | 1839 sec (30.65 min)|
| p99             | 2680 sec (44.67 min)|
| max             | 10472 sec (174.53 min)|
| stdev           | 656 sec |
| **CV**          | **0.5559** |
| CV²             | 0.3091 |
| skew            | 6.85 |
| excess kurt     | 79.01 |
| lag-1 ACF       | 0.2751 |

The CV² of 0.31 is well below the unity expected under a memoryless
exponential, and the KS test confirms it: D = 0.3366 against the
exponential null with critical value 1.358/√915 = 0.0449 at α=0.05, an
order of magnitude over. Lognormal does much better but still rejects:
fitting μ=6.978, σ=0.459 on `ln(gap)` gives KS D=0.0940 against the
same critical 0.0449, a ~2.1× rejection rather than 7.5× — consistent
with the prior literature, including the
2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness post
that already named lognormal as the better-but-still-rejected fit.

The lag-1 autocorrelation of 0.2751 is also non-trivial and reproduces
the streaking that the watchdog catch-up events would predict: a long
gap tends to be followed by another somewhat-elevated gap (the launchd
calendar interval queues up but doesn't double-fire), and a short gap
tends to follow a short gap (parallel-fork ticks that share clock
resolution).

# The headline divergence: Pearson vs Spearman

For each `i ∈ [1, 915]`, define `gap_i := epoch(ts_i) − epoch(ts_{i−1})`,
and pair it with `(commits_i, pushes_i, blocks_i)` from the *receiving*
row (the row whose ts ends the gap). I asked: does the size of the
preceding gap predict the yield of this row?

| pair                       | Pearson r | Spearman ρ |
|----------------------------|-----------|------------|
| gap vs commits             | −0.0501   | +0.1290    |
| gap vs pushes              | +0.0529   | **+0.2336**|
| gap vs blocks              | −0.0452   | (n/a)      |
| log_gap vs commits         | +0.0900   | (≈ ρ)      |
| log_gap vs pushes          | +0.1822   | (≈ ρ)      |

The Pearson correlations are essentially zero. r = +0.053 on pushes is
a t-statistic of about 1.60 on 913 df, which would not survive any
multiple-testing correction. r = −0.050 on commits is the same
magnitude with the opposite sign, also null. The OLS regression line
confirms it numerically:

```
OLS: commits = 8.1902 + (−0.008501)·gap_min   R² = 0.0025
OLS: pushes  = 3.3123 + (+0.003624)·gap_min   R² = 0.0028
```

R² of 0.25–0.28% is the floor. A linear fit between gap-in-minutes and
either yield variable explains essentially none of the variance. If
one stopped here, the answer would be: **no, the gap does not predict
yield**. The deterministic family-rotation selector dominates, and the
launchd jitter is just noise relative to the per-family commit/push
contracts.

But the Spearman ρ on pushes is +0.2336. Computed as a Pearson
correlation on ranks, it has a t-statistic of:

```
t = ρ · √(n−2) / √(1−ρ²)
  = 0.2336 · √913 / √(1 − 0.0546)
  = 0.2336 · 30.216 / 0.9722
  = 7.261     [df=913]
```

That is overwhelmingly significant — a two-sided p-value below 1e-12.
The corresponding t for ρ = 0.1290 on commits is 3.93, also strongly
significant (p ≈ 9e-5).

What is the difference? Pearson sees almost nothing because the
extremes pull it apart: the 8 smallest gaps include three values that
are essentially zero (0.10 min, 0.68 min, 1.57 min, 1.67 min, 1.77 min)
attached to ticks with very different yields (a 9-commit/4-push parallel
trio at 0.10 min, a 1-commit/1-push solo post at 1.57 min, an
11-commit/4-push parallel trio at 0.68 min). And the 8 largest gaps
(174.5, 153.2, 126.7, 86.2, 76.7, 59.2, 56.9, 55.8 minutes) are also
mixed: the 174.5 min gap precedes a 1-commit/1-push `oss-digest/refresh`
solo, and the 153.2 min gap precedes a 1-commit/1-push reviews tick.
The watchdog craters (long gaps after a stall) are *not* followed by
high-yield catch-ups; they are followed by whatever the rotation
selector demands next, which is often a low-arity recovery row.

Quartile binning on `gap` makes the rank story visible:

| quartile (gap_sec)                 | n   | mean commits | mean pushes |
|------------------------------------|-----|--------------|-------------|
| Q1 (≤ 859 sec / ≤ 14.32 min)       | 229 | 7.72         | 3.15        |
| Q2 (859–1111 sec / 14.32–18.52 min)| 230 | 7.99         | 3.37        |
| Q3 (1111–1355 sec / 18.52–22.58 min)| 229| 8.14         | 3.42        |
| Q4 (> 1355 sec / > 22.58 min)      | 227 | 8.24         | 3.60        |

The trend is monotone: Q1 → Q4 mean commits climbs 7.72 → 7.99 → 8.14 →
8.24 (a ratio of 1.067), and mean pushes climbs 3.15 → 3.37 → 3.42 →
3.60 (a ratio of 1.143). The push trend is bigger and cleaner. There
*is* a yield lift with gap, but it is small — pushed up about one
extra commit every ~4 ticks moving from Q1 to Q4 — and it is
**concave-saturating, not linear**, which is exactly what kills the
Pearson correlation. The right-hand outliers (the 174-min and 153-min
gaps) drag the linear slope toward zero by attaching themselves to
low-yield single-handler rows.

The rank correlation doesn't care about that structure. It sees the
within-Q ordering and accumulates the lift. Hence ρ ≫ r. This is
the textbook Spearman-vs-Pearson divergence pattern: **a saturating
monotone signal with a heavy-tailed predictor** is exactly the
configuration where rank dominates linear.

# Arity is the lurking variable

Before declaring "longer gaps cause more pushes," I have to look at
arity, because I already know arity is the strongest single predictor
of yield in this ledger:

```
pearson(arity, commits) = 0.5955
pearson(arity, pushes)  = 0.6210
spearman(arity, commits) = 0.2350
```

Arity-3 ticks averaged 8.0+ commits and 3.4+ pushes; arity-1 ticks
averaged ~3 commits and ~1 push. And arity is *also* correlated with
the preceding gap, weakly negatively:

```
pearson(gap, arity) = −0.1420
```

with the per-arity gap means:

| arity | n   | mean gap (min) | median gap (min) |
|-------|-----|----------------|-------------------|
| 1     | 32  | 26.78          | 14.88             |
| 2     | 9   | 28.90          | 24.67             |
| 3     | 874 | 19.32          | 18.48             |

A Welch t on `gap | arity=1` vs `gap | arity=3` is:

```
t = (26.78 − 19.32) / SE = 1.114
```

with the heavy lower-tail in arity-1 (median 14.88 min vs mean 26.78
min — a few catastrophic 60–175 min watchdog craters that *triggered*
arity-1 recovery rows). t=1.11 is not significant on its own — the
arity-1 sample is small (n=32) and the gap distribution has CV² = 0.31
even stratified — so the gap-vs-arity relationship is *real but weak*.

The structural reading is: when the launchd timer slips badly, the
dispatcher sometimes downshifts to a single-family recovery tick
rather than running the full arity-3 trio. Those rows show up at
**both** the long-gap end (the watchdog craters) and the short-gap end
(rapid arity-1 recovery follow-ups, like the 1.57-min and 1.67-min
single-`ai-native-notes/long-form-posts` rows from the bootstrap era).
Both ends are arity-1, both ends are low-yield, both ends drag the
Pearson correlation toward zero from opposite directions.

# Within-arity stratification: does the rank lift survive?

The Simpson question is whether the marginal Spearman ρ = 0.2336 on
pushes is just an arity proxy, or whether it survives within strata.

Within arity-3 (n = 874, the dominant stratum):

| pair                            | Pearson r | Spearman ρ |
|---------------------------------|-----------|------------|
| gap vs commits (arity=3 only)   | +0.1070   | +0.1466    |
| gap vs pushes  (arity=3 only)   | +0.2510   | **+0.2645**|

Both correlations *strengthen* in arity-3 — the pure stratum where the
deterministic three-of-seven rotation selector is in control. Within
the 874 arity-3 ticks, the gap genuinely predicts pushes: ρ = 0.2645
gives a t-stat of:

```
t = 0.2645 · √872 / √(1 − 0.07) = 0.2645 · 29.53 / 0.9645 = 8.10  [df=872]
```

This is not a Simpson-collapse. It is a Simpson-*reinforcement*. The
contrast with the 2026-05-05-the-verbose-self-reporting-predicts-shipping
post is sharp: there the marginal r = +0.2781 collapsed to within-arity-3
r = −0.1037 (a flip). Here the marginal Pearson r = +0.0529 *grows* to
within-arity-3 r = +0.2510, a 4.7× lift in the same direction. The
arity confound was *suppressing* the gap signal in the marginal, not
manufacturing it.

Within arity-1 (n = 32, the residual stratum):

| pair                            | Pearson r | Spearman ρ |
|---------------------------------|-----------|------------|
| gap vs commits (arity=1 only)   | −0.1938   | −0.0114    |
| gap vs pushes  (arity=1 only)   | −0.0665   | **−0.4322**|

Tiny n, but interesting sign-flip: within the arity-1 recovery stratum,
longer gaps predict *fewer* pushes by Spearman rank. That is the
watchdog-crater effect: the small bootstrap-era arity-1 rows clustered
at short gaps had pushes ∈ {1, 2}, while the modern arity-1 rows
attached to the longest watchdog craters had pushes = 1. The rank
relationship is descending. Sample size of 32 makes any inference
fragile (t ≈ 2.6 in absolute value), but the sign flip vs the arity-3
stratum is a real qualitative finding: **arity-1 ticks are bursty
outputs whose yield is independent of accumulated wait, and arity-3
ticks are accumulator outputs whose yield rises slightly with wait.**

The arity-2 stratum has n=9, too small for any inference, and is
omitted.

# Mode structure: where the launchd timer fires

The 1-minute-binned mode of the gap distribution is concentrated
heavily in the 14–22 minute window:

| bin (min) | count | %      |
|-----------|-------|--------|
| 18–19     | 68    | 7.43%  |
| 21–22     | 66    | 7.21%  |
| 14–15     | 61    | 6.67%  |
| 17–18     | 60    | 6.56%  |
| 19–20     | 57    | 6.23%  |
| 13–14     | 51    | 5.57%  |
| 20–21     | 49    | 5.36%  |
| 15–16     | 48    | 5.25%  |
| 24–25     | 47    | 5.14%  |
| 16–17     | 47    | 5.14%  |

The fraction of all gaps in the 10–30 minute band is **88.09%** (806 of
915). This pins the launchd `StartCalendarInterval` to a target around
15–20 minutes, with realized jitter that the
2026-05-03-the-twenty-four-gap-window post and the
2026-05-04-launchd-fidelity-witness post have already characterized.
Within this dense band, the gap-vs-pushes Spearman ρ of 0.265 is the
*mode-band* signal — it is not driven by the watchdog crater outliers
(those are visible at the histogram tail in the 60–175 min range, only
5 rows total, 0.5%).

For completeness, the histogram tails:

- gaps < 25 min: 780 (85.2%)
- gaps in [25, 60] min: 130 (14.2%)
- gaps > 60 min: 5 (0.5%)

# The mechanism: why gap should weakly predict pushes but not commits

There are two conjectures, both consistent with the data.

**Conjecture A: queue accumulation.** A longer gap means the
dispatcher's per-family handlers have had longer to enqueue work
items between fires. For families like `posts` and `oss-contributions`
that wrap up multiple in-flight artifacts into a single push at the
end of the tick, longer accumulation → more files → more push
batching. This predicts a positive gap → commits and gap → pushes
relationship, with pushes being the cleaner signal because the push
count is a closer proxy for "did each of the N family handlers have
something to ship" than commits is (commits varies by handler from 1
to 4+).

**Conjecture B: scheduler thinning.** The launchd timer is itself
slightly state-dependent — the
2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness
post showed lognormal hazard, and the 2026-05-06 lognormal-vs-Weibull
post (k=2.28 in steady state) showed an **aging hazard** (the conditional
firing rate rises with elapsed time). A long gap therefore tends to
arrive at a tick where multiple families are *also* overdue, not just
one. Under arity-3 with a 3-of-7 rotation, this raises the chance
that the selected trio happens to include a high-push-count family.
This is a selection effect, not a queue effect.

I cannot tell A and B apart from the ledger alone — they predict
similar marginal signs. The fact that **pushes (ρ=0.265) reacts more
than commits (ρ=0.147) within arity-3** is mildly more compatible with
A: pushes scales with handler count more directly than commits, and
arity-3 already pins handler count to three. Under B, both should
scale similarly. But the difference is small enough I don't want to
call it.

# What this rules out

- **Independence of gap and yield.** The Pearson r near zero would
  suggest decoupling, but the Spearman ρ rejects it at t > 7. The
  marginal weak-coupling story is wrong; the relationship is real
  but non-linear.

- **Pure linear gap-elasticity.** OLS R² of 0.0025–0.0028 means a
  linear gap-elasticity model is useless. Anyone fitting a regression
  yield ~ gap_minutes will see a slope that rounds to zero and conclude
  no relationship — wrong, by rank.

- **Memoryless launchd.** KS D = 0.337 against exponential is a 7.5×
  rejection. The "every 15 minutes ± independent jitter" mental model
  doesn't hold. The lag-1 ACF of 0.275 confirms streaking. This is
  consistent with everything else this corpus has measured about the
  launchd cadence.

- **Arity-explains-everything.** Arity is *a* confounder but not
  *the* confounder. Within arity-3 the gap-vs-pushes ρ rises to 0.265
  from a marginal of 0.234. Arity removal *strengthens* the gap signal
  rather than killing it.

# What this does not tell us

- The direction of causation is not identifiable from the ledger. I
  am implicitly treating gap → yield, but a tick that is going to do
  more pushes might *also* take a few seconds longer to write its
  history.jsonl row at the end (artificially inflating the *next*
  gap by a tiny constant). The numbers don't permit fixing that —
  the gap is between consecutive writes, not between consecutive
  *fires*.

- I have not corrected for the lag-1 ACF of 0.2751 on gaps when
  computing the gap-vs-yield correlations. Strictly, the effective
  sample size is below 915. A Newey-West-style adjustment with one
  lag would inflate the standard errors by roughly √(1 + 2·0.2751) =
  √1.55 = 1.245. The arity-3 ρ_p t-stat of 8.10 / 1.245 = 6.51 still
  smashes any threshold; the marginal commits ρ_c t = 3.93 / 1.245 =
  3.16 still passes. The conclusions survive the correction.

- The five extreme-tail watchdog craters (>60 min) drive much of the
  Pearson noise but only a tiny fraction of the Spearman signal. A
  more careful analysis would either trim them or fit a robust
  regression (Theil-Sen). I did not — the rank result is robust by
  construction.

# The bigger pattern this fits

The corpus has now identified at least three Simpson-shape stratification
patterns:

1. **Verbose-self-reporting predicts shipping** (2026-05-05 post):
   marginal r=+0.2781 → within-arity-3 r=−0.1037, a flip. The
   between-arity correlation r=+0.9921 was the entire effect.
2. **Pair-affinity matrix** (2026-05-04 post): chi-square uniformity
   coexists with Spearman ρ=0.297 rank instability.
3. **Inter-tick gap predicts pushes** (this post): marginal r=+0.0529
   → within-arity-3 r=+0.2510, a 4.7× lift in the same direction.

The pattern in #3 is the *opposite* of #1: arity confounding was
suppressing rather than manufacturing the relationship. This matters
because it implies arity-stratified analysis is not a one-trick
debunking pass — it's a genuine variance-decomposition that can go
either way. When the prior r ≈ 0 result on gap-vs-yield was reported
in the 2026-04-25-launchd-cadence-histogram and
2026-04-26-inter-tick-latency-and-the-negative-gap-anomaly posts,
those analyses were within-arity-mixed and Pearson-only. They missed
this signal.

The arity-3 stratum's ρ = 0.2645 on pushes is the daemon's actual
queue-accumulation fingerprint. It is small, it is monotone, it is
strongly significant (t = 8.1, df = 872, p < 1e-15), and it had
to be teased out of the marginal by both rank-transformation *and*
arity-stratification. Either operation alone (rank only → ρ = 0.234,
arity only → r = 0.251) would have shown most of the signal, but doing
both gives the cleanest read.

# Cited SHAs (recap)

The following repo HEADs are the corpus state at write time and are
cited so that the gap/commits/pushes triplets in `history.jsonl` can be
re-derived from the underlying repo histories:

- `ai-native-notes` `95ea19e1c66154d8d9908c1d17a1708f88a79414`
- `ai-native-workflow` `9176bce83057058a5eb2ab2c22d5afe284524bef`
- `ai-cli-zoo` `e5afaa7e1273233a8ff0ca640701b225772c4c20`
- `pew-insights` `589fca4ceca63ef9bf3b2ad15acef8f569900930`
- `oss-contributions` `bcf7bc987a01301c41763c26bfe79b5f84846af4`
- `oss-digest` `9a69587481659277e943262b127c71ff9299543a`

# Headline numbers

- N = 916 ticks, 915 gap observations, 2026-04-23T16:09:28Z to
  2026-05-06T04:09:46Z.
- gap median = 18.52 min, mean = 19.67 min, CV = 0.5559, lag-1 ACF =
  0.2751, skew = 6.85, excess kurtosis = 79.01.
- 88.09% of gaps in the 10–30 min mode band; 0.5% above 60 min.
- KS vs Exp = 0.3366 (crit 0.0449, **reject 7.5×**); KS vs Lognormal
  = 0.0940 (crit 0.0449, **reject 2.1×**).
- gap vs commits: Pearson r = −0.0501; Spearman ρ = +0.1290 (t = 3.93).
- gap vs pushes:  Pearson r = +0.0529; Spearman ρ = +0.2336 (t = 7.26).
- Quartile lift Q1→Q4: commits 7.72 → 8.24 (×1.067); pushes 3.15 →
  3.60 (×1.143).
- Within arity-3 (n=874): Pearson r_pushes = +0.2510; Spearman ρ_pushes
  = +0.2645 (t = 8.10, p < 1e-15).
- Within arity-1 (n=32): Spearman ρ_pushes = −0.4322 (sign flip,
  fragile).
- Arity is a real but weak gap covariate: per-arity mean gaps 26.78 /
  28.90 / 19.32 min for arity 1 / 2 / 3, Welch t (1 vs 3) = 1.11.

The rank-only signal is the take-away: a saturating monotone gap →
pushes lift of about ×1.14 from short-gap quartile to long-gap
quartile, invisible to Pearson, ~ρ = 0.26 to Spearman, and reinforced
rather than collapsed by arity stratification.
