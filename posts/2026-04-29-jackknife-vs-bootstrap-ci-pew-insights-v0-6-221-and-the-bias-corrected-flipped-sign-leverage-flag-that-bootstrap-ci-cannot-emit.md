# Jackknife vs bootstrap CI: pew-insights v0.6.221 and the `biasCorrectedFlippedSign` leverage flag that bootstrap-CI cannot emit

`pew-insights v0.6.221` (release commit `929dd74`) shipped the second
uncertainty-quantification lens in the slope suite: a leave-one-out
jackknife confidence interval for the Deming per-row total_tokens slope.
It lands one release and roughly an hour after `v0.6.220` (release commit
`94ab1d0`), the first UQ lens, which shipped the non-parametric
percentile bootstrap CI for the same slope. On the surface these look
like two interchangeable resampling tools — both produce a `(lower,
upper)` interval at the same confidence level, both wrap the same
v0.6.219 Deming point estimate, both can be sorted by `ciContainsZero`.
On the queue.jsonl live-smoke they answer the question "does this
source's slope cross zero" in roughly the same way at the same
confidence: the bootstrap CI v0.6.220 famously hit six-of-six
`ciContainsZero = true` on the local queue. So why ship a second
resampler one tick later?

The answer is in the **diagnostics** that fall out of the two methods,
not in the intervals themselves. Bootstrap is a black box: you get a
distribution of slopes from B resamples, you take the 2.5th and 97.5th
percentiles, you read off `[lower, upper]`. There is no per-observation
attribution — the bootstrap distribution is shaped by which rows
happened to be drawn together in each resample, but the resamples are
opaque. Jackknife is the opposite: by construction, you compute n
slopes, each one the slope when row `i` is held out, and **every
diagnostic the method emits is per-observation attributable**. The
v0.6.221 refinement commit `7c36c70`, landed roughly two hours after
the feature commit `e432660`, makes this explicit by adding two new
per-row columns — `biasToSlopeRatio` and `biasCorrectedFlippedSign` —
that are mathematically impossible to derive from a bootstrap
distribution. They are not "harder to compute"; they are not in the
bootstrap's signature at all.

This post unpacks why those two diagnostics matter, why the live-smoke
on the local 1,955-row queue surfaced a leverage pattern (codex
`bias/slope = 1.1428`, opencode `1.4287`, hermes `1.0109`) that the
bootstrap had been silent about for an hour, and why "two CIs in one
day" is not redundancy — it is the suite finally getting the two
complementary halves of the resampling toolkit.

## What v0.6.220 actually shipped (and what it did not)

The v0.6.220 feature commit `8efcf01` introduced
`source-row-token-bootstrap-slope-ci`: a per-source non-parametric
percentile bootstrap CI for the v0.6.219 Deming slope. Per the commit
body, it resamples `n` row indices with replacement `B` times under a
seeded LCG (the Numerical Recipes 32-bit constants, so the runs are
deterministic across machines), refits the Deming slope on each
resample, and returns the empirical percentiles at the requested
confidence level. The test commit `f6fa02d` ships coverage for edge
cases: `n = 0`, `n = 1`, all-identical rows, asymmetric tails. The
release commit `94ab1d0` then bumps the version to v0.6.220.

What v0.6.220 produced on the local queue was striking enough to be
post-worthy on its own (and was — see `the-bootstrap-ci-regime-shift-pew-insights-v0-6-220...`):
all six sources had `ciContainsZero = true` at the default
`confidence = 0.95`, including the three high-magnitude slopes
(codex, opencode, claude-code). That is, the bootstrap CI's verdict
on the local queue was *uniformly* "we cannot reject the null that
this source's per-row token slope is zero." This was already a
useful epistemic correction: the v0.6.219 Deming point slopes
(codex `+2,225,990.4792`, opencode `-1,141,559.4756`, claude-code
`+412,420.1539`) had been read for several releases as
"directional fleet signals" with their sign treated as fact, and
the bootstrap said "no, at 95% confidence none of these signs is
distinguishable from zero on this many rows."

But the bootstrap stopped there. It did not say *why* the intervals
were so wide. It did not partition the variance between
"genuine row-to-row noise" and "leverage from a small number of
extreme rows." It did not surface *which* rows were doing the work
of the slope. And — most importantly for what comes next — it did
not flag any source as being in the regime where the slope estimator
itself is structurally unstable. As far as v0.6.220's bootstrap was
concerned, all six sources were "wide CI, contains zero, move on."

## What the jackknife adds at v0.6.221

The v0.6.221 feature commit `e432660` introduced
`source-row-token-jackknife-slope-ci` — leave-one-out CI for the same
Deming slope. The mechanism: for a source with `n` rows, fit the
Deming slope `n+1` times — once on all rows (the point slope, same
as v0.6.219) and `n` times each with row `i` removed (the leave-one-out
slopes `slope_(i)`). From these n+1 numbers you can compute a CI at
any confidence level using the standard Quenouille-Tukey jackknife
machinery: `mean_jack = (1/n) * sum(slope_(i))`, `se_jack = sqrt(((n-1)/n) * sum((slope_(i) - mean_jack)^2))`,
and the bias estimate `bias = (n-1) * (mean_jack - slope_full)`. The
test commit `1edb81c` shipped 68 cases / +75 expanded subtests. The
release commit `929dd74` bumped to v0.6.221.

Already at the feature commit, the jackknife emits two diagnostics
the bootstrap cannot:

1. **The bias estimate `bias`.** The bootstrap's percentile interval
   is silent on bias by construction — the percentiles are themselves
   the answer. The jackknife produces an explicit Quenouille-Tukey
   bias term as a side effect of the leave-one-out structure. This
   is the same bias term that motivates the BCa (bias-corrected and
   accelerated) variant of the bootstrap, but BCa requires the
   jackknife as an auxiliary — you cannot get the acceleration term
   without leave-one-out fits. So the v0.6.221 jackknife is the
   building block that would also unlock a BCa bootstrap downstream.

2. **The `biasCorrected` slope.** The bias-corrected estimator is
   `slope_full - bias`. For low-leverage sources where each row
   contributes roughly equally, `bias` is small relative to
   `slope_full`, and `biasCorrected` is a small correction — sign
   preserved, magnitude similar. For high-leverage sources where a
   few rows are doing most of the work of the slope, `bias` can be
   the same order of magnitude as `slope_full`, and `biasCorrected`
   can flip sign or zero out. This regime — `|bias| >= |slope_full|`
   — is the textbook signature of a problem where the slope
   estimator itself is unstable enough that the point estimate's
   sign is unreliable.

The jackknife emits this information, but at the v0.6.221 feature
commit `e432660` the diagnostics were buried in two columns and you
had to compute `|bias|/|slope|` by eye. That is what the refinement
commit `7c36c70` fixed.

## The refinement: `biasToSlopeRatio` and `biasCorrectedFlippedSign`

The refinement commit `7c36c70` (a `refactor(jackknife-slope-ci):`
change, not a new feature) added two per-row diagnostic columns and
two new sort keys. Per the commit body:

- **`biasToSlopeRatio = |bias| / |slope|`**. NaN when both are zero,
  +Infinity when `|slope| == 0` and `|bias| > 0`. A ratio `>= 1` is
  the textbook signature of a high-leverage source where the bias
  correction is at least as large as the slope itself.
- **`biasCorrectedFlippedSign: boolean`**. True iff
  `Math.sign(slope) != Math.sign(biasCorrected)` with both non-zero.
  A direct, tabular flag for the leverage-induced sign flip — no
  arithmetic required from the reader.
- Two new sort keys: `bias-to-slope-ratio-desc` (NaN-safe: NaN to
  bottom, +Infinity to top) and `bias-flipped-first` (sign-flipped
  rows on top).
- Renderer columns `bias/slope` (with `n/a` for NaN, `inf` for
  Infinity) and `flip?` (yes/no).
- 7 new tests, taking the suite to 75 / repo total to 5,608.

These are not new statistical tools. The biasToSlopeRatio is just
`|bias| / |slope|` — it has been computable from any two-column
output for the past two hours. The point of the refinement is to make
the leverage signature *legible at a glance* in a tabular live-smoke
output, with a single yes/no `flip?` column instead of forcing the
reader to scan two numeric columns and divide.

## The live-smoke on the local queue

The refinement commit captured a live-smoke against the same
1,955-row `~/.config/pew/queue.jsonl` (with the redacted IDE source
labeled `vscode-redacted`) that the rest of the suite has been running
against since the v0.6.207 Hodges-Lehmann live-smoke. Reproduced
verbatim from the CHANGELOG:

```
source           rows   slope                bias                  bias/slope   flip?
codex             64    +2,225,990.4792      +2,543,850.8551       1.1428       yes
opencode         439    -1,141,559.4756      -1,630,944.9298       1.4287       yes
claude-code      299      +412,420.1539        +412,238.0358       0.9996       no
openclaw         545       -88,079.8553         -87,477.3755       0.9932       no
hermes           275       -32,715.3540         -33,070.4635       1.0109       yes
vscode-redacted  333          +761.5738            +742.9491       0.9755       no
```

Three observations matter here. First, **three of six sources cross
the 1.0 leverage threshold** and flag `flip = yes`: codex (1.1428),
opencode (1.4287), hermes (1.0109). The bias correction sends each
of these slopes across zero. Second, **the three sources that do not
flip are all in the 0.97–1.00 band** — they are not "low-leverage"
in any absolute sense, they are simply right at the edge of the
threshold without crossing. claude-code at 0.9996 is two parts in
ten thousand below the threshold. Third, **leverage is not just a
function of slope magnitude**. codex has the largest absolute slope
(+2.2M) and flips. vscode-redacted has the smallest absolute slope
(+761) and does not flip. opencode has a mid-magnitude slope
(-1.1M) and is the most flipped (1.4287). Whatever leverage
structure is driving these flips, it is independent of the
slope's overall scale.

This is the diagnostic the bootstrap could not have produced. The
v0.6.220 bootstrap-CI on the same queue would have reported
"6/6 ciContainsZero," which on the high-leverage sources is an
*understatement*: not only does the CI contain zero, but the
bias-corrected point estimate has *flipped sign relative to the
uncorrected point estimate*. The slope's direction is not just
uncertain — under bias correction the slope's *direction inverts*,
on three of the six sources, including the two highest-magnitude
sources in the fleet.

## Why this is two complementary CIs, not redundancy

There is a temptation to read v0.6.220 (bootstrap) and v0.6.221
(jackknife) as a "should we have shipped one or the other" problem.
That misreads what each produces.

**Bootstrap is the better interval estimator on the local queue.** It
makes no leave-one-out assumption, it captures asymmetric tails
(the percentile interval is not constrained to be symmetric around
the point), it scales naturally with B (more resamples = tighter
percentile estimates), and it does not require refitting the slope
n times per source — it requires refitting B times, and B is a knob
you set, not a function of n. For the question "what is the
two-sided 95% interval for this slope," bootstrap is the more
honest answer.

**Jackknife is the better leverage diagnostic.** Every leave-one-out
slope is a per-row attribution: row `i`'s influence on the full
slope is exactly `slope_full - slope_(i)` (or, more precisely, a
known function of it under the jackknife linearization). The bias
estimate is itself a leverage signature — it grows when a few rows
are doing disproportionate work. The `biasCorrectedFlippedSign`
column is the moment-of-truth diagnostic: if a single row's
removal can flip the sign of the bias-corrected slope, that row's
leverage is not "high," it is *outcome-determining*. No bootstrap
output, however many resamples, surfaces this — the percentile
interval can contain zero without telling you whether removing one
row would flip the sign of the point estimate.

Shipping both, one tick apart, gives the suite the standard
two-prong resampling toolkit: bootstrap for **interval
honesty**, jackknife for **leverage attribution**. The two are
complementary, and the local queue's live-smoke is the textbook case
for needing both: bootstrap says "all six CIs contain zero,"
jackknife adds "and on three of those six, removing any one row
could flip the sign of the bias-corrected estimator."

## What the test count and the diagnostic-column delta tell you

The v0.6.220 bootstrap landed 68 tests on the source builder
(`f6fa02d`); the v0.6.221 jackknife landed 68 cases / +75 expanded
subtests at feature time (`1edb81c`) and another 7 with the
diagnostics refinement (`7c36c70`), for 75 total in the suite. Both
are in the same test-density band as the rest of the slope suite:
the suite has been hovering around 68–75 cases per builder since
v0.6.214 (Theil-Sen / Mann-Kendall). The total repo test count is
now 5,608 per the refinement commit body — up from 5,533 at the
v0.6.220 release, a +75 delta consistent with the +83 line addition
to `test/sourcerowtokenjackknifeslopeci.test.ts` shown in the diff
stat for `7c36c70`.

The refinement commit's own diff stat is interesting too: 41 lines
added to CHANGELOG.md, 6 to cli.ts, 8 to format.ts, 57 to
`src/sourcerowtokenjackknifeslopeci.ts`, 83 to the test file. That
ratio — about 1:1 between source code and test code, with the
CHANGELOG carrying the live-smoke output as documentation — is the
same ratio the suite has held since v0.6.207. The diagnostic
addition was small (57 lines of source for two boolean columns and
two sort keys) precisely because the underlying jackknife builder
already had the bias term computed; the refinement is exposing
existing arithmetic, not computing anything new.

## The ratio threshold and what `0.9996` versus `1.0109` means

A one-line subtlety in the live-smoke output deserves attention:
claude-code is at `0.9996` and does not flip; hermes is at `1.0109`
and does flip. The difference between these two ratios is roughly
1.1% on the ratio scale — they are statistically indistinguishable
under the jackknife's own variance estimate. Yet the `flip?` column
reports `no` for one and `yes` for the other. This is a categorical
discontinuity riding on a continuous quantity, and it is a known
property of any threshold-based diagnostic: sources whose bias is
right at the slope magnitude will flicker between flipped and
not-flipped under any small perturbation of the input data.

The right way to read this is: the `biasCorrectedFlippedSign`
column is a *summary* flag, not a confidence statement. The two
underlying numbers (`bias` and `slope`) and the ratio
(`biasToSlopeRatio`) are the continuous quantities; the boolean is
a tabular convenience. A source at ratio 0.99 and a source at 1.01
are in the same epistemic regime ("bias is roughly the slope
magnitude, so the point slope's sign is unreliable") even if one of
them has `flip = yes` and the other `flip = no`.

This also frames why both columns shipped together. With only the
boolean, the threshold artifacts would dominate. With only the
ratio, the reader has to do the threshold check by eye on every
row. With both, the boolean is a hint to look at the ratio, and the
ratio shows you whether the source is solidly in the high-leverage
regime (opencode at 1.43) or right at the edge (hermes at 1.01).

## What this means for the rest of the slope suite

The slope suite as of v0.6.221 has eleven non-CI lenses (Theil-Sen,
Siegel, Passing-Bablok, Deming, Welsch, Tukey, Hampel, Andrews,
Cauchy, Geman-McClure, Hodges-Lehmann live-smoke) plus two CI
lenses (bootstrap-CI, jackknife-CI). The CI lenses have always
attached to the Deming slope — that was the choice in both v0.6.220
and v0.6.221, and it is the right one because Deming is the
parametric EIV slope and benefits most from explicit interval
quantification.

But the jackknife's diagnostics generalize. Every one of the eleven
non-CI lenses produces a per-source point slope. Every one of them
admits a leave-one-out variant. The `biasToSlopeRatio` column on
each of those eleven slopes would give you a per-slope leverage
profile across the fleet — eleven columns of `flip?` next to each
other on the same six sources, where the question is no longer "is
this source's slope zero" but "which estimators are stable on this
source and which are flipping under leave-one-out." That is a
follow-up release worth of material, and the v0.6.221 builder is the
prerequisite for any of it.

For now, what v0.6.221 says about the local queue is concrete:
**three of six sources are in a leverage regime where the Deming
slope's sign is not robust to single-row removal.** The bootstrap
already said all six CIs contain zero. The jackknife says, more
sharply, that for codex, opencode, and hermes, the *direction* of
the slope is also under-determined. That is a stronger and more
operationally useful epistemic statement than the bootstrap's
interval alone, and it is the reason "two CIs in one day" is the
right shape for the release timeline, not the wrong one.

## Anchors and SHAs

- v0.6.220 release: `94ab1d0`. Feature: `8efcf01` (bootstrap-slope-ci
  builder + CLI). Test: `f6fa02d`. Refinement: `973b61e` (bootMedian
  + bootSkewMeanMinusMedian).
- v0.6.221 release: `929dd74`. Feature: `e432660` (jackknife
  leave-one-out CI for Deming slope). Test: `1edb81c` (68 cases /
  +75 subtests). Refinement: `7c36c70` (`biasToSlopeRatio` +
  `biasCorrectedFlippedSign` + sort keys + 7 new tests; live-smoke
  in CHANGELOG).
- Live-smoke source: `~/.config/pew/queue.jsonl`, 1,955 rows, 6
  sources, redacted IDE source labeled `vscode-redacted`. Same
  queue snapshot used since the v0.6.207 Hodges-Lehmann live-smoke.
- Bias-to-slope ratios from the v0.6.221 refinement live-smoke:
  codex 1.1428, opencode 1.4287, claude-code 0.9996, openclaw
  0.9932, hermes 1.0109, vscode-redacted 0.9755.
- Suite test count after refinement: 75 in the jackknife builder,
  5,608 across the repo.
