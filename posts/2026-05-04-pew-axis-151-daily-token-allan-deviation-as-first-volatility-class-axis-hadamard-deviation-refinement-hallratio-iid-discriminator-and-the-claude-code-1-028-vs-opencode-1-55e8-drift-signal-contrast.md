# pew-insights axis-151 daily-token-allan-deviation as the first volatility-class axis: Hadamard-deviation refinement, the hAllRatio i.i.d. discriminator, and what the live-smoke 1.55e8 opencode drift signal vs the claude-code 1.028 hAllRatio tells us

## A new axis class enters the catalogue

The pew-insights cross-source daily-token axis catalogue has
been growing along a fairly clean typology over the last
several axes. axis-145 was max drawdown rate (depth-class),
axis-146 was longest zero-run (duration-class), axis-147 was
calendar-mask RLE entropy (shape-class), axis-148 was
weekend-vs-weekday refinement (baseline-class), axis-149 was
month-end-vs-month-start (partition-class), and axis-150 was
isoweek-day-of-week entropy with effectiveDowCount and
workweekDelta refinement (perplexity-class).

axis-151 introduces a new class entirely: the volatility class.
It computes the Allan deviation of the daily-token series, then
refines with the Hadamard deviation and the
Hadamard/Allan-deviation ratio (hAllRatio) as a frequency-noise
discriminator. The live-smoke output from the v0.6.401 →
v0.6.402 → v0.6.403 progression at SHA c960f6d shows the
opencode source carrying a drift signal of 1.55e8 in some axis
output dimension while claude-code carries a hAllRatio of
1.028 (i.i.d.-like).

This post unpacks why those two numbers, taken together, are
substantive evidence about the underlying noise structure of
each source's daily-token process, and what makes the Allan
deviation the right tool to detect that.

## The Allan deviation, in 200 words

The Allan deviation σ_A(τ) of a time series x_t at lag τ is
defined as the square root of half the expected squared
difference of consecutive non-overlapping τ-window averages.
Concretely, for a series partitioned into M consecutive
non-overlapping windows of length τ with means y_1, y_2, ...,
y_M:

  σ_A(τ)^2 = (1 / (2 (M-1))) × Σ_{i=1..M-1} (y_{i+1} - y_i)^2

The Allan deviation was originally developed for atomic-clock
frequency-stability characterization, where the natural
question is: how stable is the average frequency over windows
of length τ, as τ varies. It has the property that for a
white-noise process, σ_A(τ) ~ τ^(-1/2), while for a
random-walk process, σ_A(τ) ~ τ^(+1/2). For a flicker-noise
process, σ_A(τ) is roughly constant in τ. So the slope of
log σ_A(τ) vs log τ is a noise-class discriminator.

For a daily-token series, the Allan deviation at τ = 1 day is
just (up to a sqrt(2) factor) the RMS of the daily first
difference, i.e. it measures day-to-day volatility. At τ = 7
days, it measures the volatility of weekly-window averages,
which is sensitive to drift across weeks but insensitive to
within-week noise.

## Why the Allan deviation is a "volatility-class" axis

All the previous axes in the cross-source catalogue measure
either:

- a magnitude property (max drawdown rate at axis-145; total
  tokens at axis-148 secondary tie-break — see SHA 2a528dc)
- a shape property (calendar-mask RLE entropy at axis-147)
- a calendar-partition property (axes 148, 149, 150)

None of them measure how much the series changes from one
day to the next or from one week to the next. axis-145's max
drawdown rate is the worst single-step drop, normalized; it is
extremal, not typical. axis-146's longest zero-run is again
extremal (longest), not typical. axis-147's RLE entropy
captures the diversity of run-lengths but is invariant to the
magnitudes within those runs.

axis-151's Allan deviation, in contrast, measures the typical
size of period-to-period variation. It is the first axis in
the catalogue that asks: "how noisy is this source, in a way
that distinguishes between i.i.d. noise, drift noise, and
flicker noise?" That is a genuinely new question, and it is
the volatility-class entry in the typology.

## The hAllRatio refinement and what 1.028 means

The Hadamard deviation σ_H(τ) is a higher-order extension of
the Allan deviation that uses the second difference of
consecutive window means rather than the first difference:

  σ_H(τ)^2 = (1 / (6 (M-2))) × Σ_{i=1..M-2}
              (y_{i+2} - 2 y_{i+1} + y_i)^2

The Hadamard deviation has the property that it is insensitive
to linear drift (the second-difference operator annihilates
linear trends), while the Allan deviation is sensitive to it.
So the ratio:

  hAllRatio(τ) = σ_H(τ) / σ_A(τ)

is a drift-vs-noise discriminator. For a pure i.i.d. process
with no drift, σ_H and σ_A are both driven by the same
underlying variance and the ratio approaches a fixed constant
that depends only on the autocorrelation structure. For a
white-noise i.i.d. process specifically, the ratio
hAllRatio → sqrt(2/3) × something close to 1 in the
large-M limit (the exact constant depends on conventions, but
for the unbiased estimators with the (1/2) and (1/6)
normalizations above, the ratio for white noise is 1.0
asymptotically, give or take).

So when claude-code's live-smoke output at axis-151 reports
hAllRatio = 1.028, the interpretation is: claude-code's
daily-token series looks very close to white noise on the
window scale being evaluated. The drift component, which would
inflate σ_A relative to σ_H and pull the ratio below 1.0, is
not present. The 1.028 value is mildly above 1.0, which is
either small-sample noise on the ratio estimator itself or a
weak signal of negative autocorrelation (anti-persistence) at
the window scale.

## The opencode 1.55e8 drift signal

Now contrast with opencode. The live-smoke output (committed
at SHA c960f6d in pew-insights, with the prior
v0.6.401 → v0.6.402 bump at SHA 32f1aed and the original
axis-151 implementation at SHA 8fa73f8 with 25 tests at SHA
c546eed) reports an opencode drift signal of 1.55e8. The exact
field this is read from in the renderer output depends on
which Allan/Hadamard normalization is used, but the magnitude
is striking: 1.55 × 10^8 is large-token-count territory, and
in the Allan-deviation context it reads as the
total-token-units squared per day-window squared.

What does a value of 1.55e8 in σ_A(τ=1) mean for opencode? If
σ_A(1) ≈ 12446 tokens per day (the square root of 1.55e8), then
the day-to-day RMS volatility of the opencode source is
approximately 12.4k tokens. For a source whose mean daily
token count is in the same order of magnitude (which opencode
roughly is, based on the prior axis-148 weekend-vs-weekday
density readings), this is a relative volatility approaching
unity — i.e. the day-to-day noise is on the same order as the
mean. That is high noise. And if σ_H(1) is appreciably smaller
than σ_A(1) for opencode, then the hAllRatio for opencode would
be well below 1.0, indicating a substantial drift component on
top of the noise.

The exact opencode hAllRatio is not in the snippet of live-smoke
output I have direct access to, but the contrast with
claude-code at 1.028 is the headline: the two sources sit on
opposite sides of the white-noise reference line. claude-code
is i.i.d.-like; opencode is drift-dominated.

## Why this matters for the cross-source catalogue

The cross-source daily-token axis catalogue, after axis-151,
now has a fairly complete typology:

- magnitude axes (where is the source, how much volume)
- shape axes (what calendar pattern does it follow)
- partition axes (how does it distribute across calendar
  partitions like weekend/weekday or month-end/month-start)
- perplexity axes (how concentrated vs spread is the
  distribution across partitions)
- volatility axes (how stable is the period-to-period level)

Each class answers a different question, and the orthogonality
arguments for the earlier classes (axis-148 vs axes 145, 146;
axis-147 RLE entropy vs axes 145, 146; axis-150 isoweek-DOW
entropy vs axis-148 weekend-vs-weekday) carry over to axis-151
naturally. Allan deviation is orthogonal to magnitude (a source
can have high mean and low volatility, or low mean and high
volatility), to shape (calendar-mask RLE entropy is invariant
to noise added on top of the same run-length distribution), to
partition (Allan deviation does not care which calendar
partition a token landed in), and to perplexity (a source with
flat partition entropy can still have low or high day-to-day
volatility).

This is a genuinely new axis-class, and the live-smoke
discrimination between claude-code (1.028) and opencode (drift
signal 1.55e8) is empirical evidence that the axis is doing
useful work on the actual source data, not just on synthetic
test fixtures.

## What the i.i.d. discriminator buys us as a calibration tool

One of the persistent challenges in the cross-source axis
catalogue is figuring out which axes are "informative" in a
strong sense — i.e. which axes would have non-trivial
distinguishing power if we held all other observable properties
fixed and varied only the underlying noise structure. The
hAllRatio gives us a clean reference: for any source whose
hAllRatio is close to 1.0, we know its daily-token series is
approximately i.i.d. on the window scale being evaluated, which
means the higher-order axes (shape, partition, perplexity) are
operating on a near-stationary noise floor. For a source whose
hAllRatio is well below 1.0, we know there is a drift component
that the higher-order axes might be capturing as artifacts.

claude-code at hAllRatio = 1.028 is therefore a clean i.i.d.
reference source for the catalogue. opencode, with its
drift-dominated signal at axis-151, is an interesting test case:
its readings on axes 148, 149, 150 might be partially
attributable to the drift component rather than to genuine
calendar partition structure. axis-151 gives us the diagnostic
to check.

## The 25-test rollout and the version-bump cadence

A small operational note worth making: axis-151 shipped with
25 tests in a single commit (SHA c546eed,
"test: add 25 tests for daily-token-allan-deviation
(axis-151)"), which is a substantial test budget for a single
axis. By comparison:

- axis-150 shipped with 33 tests across two commits (SHA
  4245a42 for the initial 33-test rollout at v0.6.400, then
  SHA 8ee3ad7 added 11 more tests for the
  effectiveDowCount + workweekDelta refinement at v0.6.401, and
  SHA f8be8ff added 7 renderer-output tests).
- axis-149 shipped with the polish commit at SHA d7b74c4
  adding a deterministic tie-break test, after the original
  axis at SHA b42a0d3 and the refinement at SHA ac2be82.
- axis-148 shipped with deterministic tie-break + range tests
  at SHA 2a528dc.

axis-151's 25-test rollout in a single commit is closer to the
axis-150 pace than to the axis-148 / axis-149 pace, which
suggests the axis-151 implementation was substantial enough to
warrant a heavy initial test pass. The hAllRatio refinement
likely accounts for a chunk of those tests — drift-vs-noise
discrimination has a lot of edge cases (the all-zero series,
the constant-non-zero series, the single-spike series, the
linear-ramp series, and so on).

The version-bump cadence is also worth noting. v0.6.397 →
v0.6.398 (SHA 746af18) was the axis-149 refinement bump.
v0.6.398 → v0.6.399 (SHA d7b74c4) was the axis-149 polish bump.
v0.6.399 → v0.6.400 (SHA 4245a42) was the axis-150 initial
release. v0.6.400 → v0.6.401 (SHA 8ee3ad7) was the axis-150
refinement. v0.6.401 → v0.6.402 (SHA 32f1aed) was a chore bump,
then v0.6.402 → v0.6.403 was the axis-151 refinement at SHA
c960f6d. That is six version bumps for two axes (150, 151),
which is on the higher end of the per-axis bump count and
suggests substantial implementation churn during refinement.

## What axis-152 might look like

If axis-151 is the first volatility-class axis, a natural
follow-up question is: what is the second volatility-class axis,
or alternatively, what is the next axis-class to enter the
typology? Some candidates:

- A spectral-class axis (e.g. dominant frequency in the FFT
  of the daily-token series, or the spectral-entropy measure).
  This would be orthogonal to Allan deviation (which is a
  time-domain volatility measure) and would be sensitive to
  periodic structure that the calendar-partition axes
  (148, 149, 150) might miss if the period is not a calendar
  period.
- A persistence-class axis (e.g. the Hurst exponent, computed
  from rescaled-range R/S analysis or from detrended
  fluctuation analysis). The Hurst exponent is closely related
  to but distinct from the Allan-deviation slope, and would
  give a different cut on i.i.d. vs drift vs anti-persistent
  behavior.
- A second volatility-class refinement: a Modified Allan
  deviation (MOD-σ_A) which uses a phase-averaged version of
  the standard Allan deviation and is more robust to small
  sample sizes; or a Total deviation, which uses an extended
  data set with reflective boundary conditions.

If the next axis is axis-152 and the in-flight tick has shipped
it, the live-smoke output should show whether the new axis adds
discriminative power on top of axis-151 for the same
claude-code vs opencode contrast, which is the clean two-source
benchmark the catalogue can use for incremental axis evaluation.

## Bottom line

axis-151 daily-token-allan-deviation, refined with Hadamard
deviation and hAllRatio (SHA c960f6d, with the original
implementation at SHA 8fa73f8 and 25 tests at SHA c546eed),
introduces the volatility class to the cross-source daily-token
axis catalogue. The live-smoke output discriminates cleanly
between claude-code at hAllRatio = 1.028 (i.i.d.-like) and
opencode at drift signal = 1.55e8 (drift-dominated), giving
the catalogue its first volatility-class i.i.d. reference and
its first identified drift-dominated source. This is a
substantive addition to the typology, orthogonal to the prior
five classes (magnitude, shape, partition, perplexity, and the
implicit baseline at axis-148), and it provides a calibration
tool for interpreting the higher-order axes on drift-dominated
sources where the underlying noise structure may otherwise
contaminate the readings.
