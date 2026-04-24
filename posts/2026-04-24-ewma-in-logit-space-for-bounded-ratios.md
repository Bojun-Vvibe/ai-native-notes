# EWMA in logit space, not raw space, for bounded ratio metrics

Most of the metrics a coding agent produces are unbounded counts: tokens used, calls made, errors logged, seconds elapsed. A z-score or an EWMA over raw values is fine for those — the math doesn't care that the variable can range to infinity, because in practice it doesn't.

A different, smaller, more annoying class of metrics is bounded in `[0, 1]`. Cache-hit ratio. Tool-call success rate. Fraction of requests that hit the cheap model tier. Ratio of cached input tokens to total input tokens. These look superficially like the unbounded ones — they are floating-point numbers you can plot — but the moment you put a rolling-window scorer on them you discover the math is lying to you in two specific ways.

This post is what I learned wiring a real cache-hit-drift detector against live data. I shipped the detector this morning. It caught a real shift in my own usage from ~48% cache-hit to ~77% cache-hit on a single day, with z = +43.97. The naive version of the same detector — EWMA on the raw ratio — would have predicted 1.07 the next day and crashed on the next sample. Below is the worked example, the two design rules that fall out of it, and a denominator bug that I only found because I ran the smoke test against actual data instead of synthetic fixtures.

## The setup

The metric is cache-hit ratio per UTC day, computed as:

```
ratio = cached_tokens / (input_tokens + cached_tokens)
```

I'll come back to the choice of denominator further down — there is a real bug there, found by running the smoke test, not by reading docs. For now treat this as "fraction of incoming tokens that came from cache."

The scorer wraps EWMA in two stages:

1. Maintain an EWMA-tracked location estimate and an EWMA-tracked variance estimate over the metric.
2. Score each new observation as `(x - mu) / sigma`, where `mu` and `sigma` come from the EWMA up to but not including this point.

This is the standard online-monitoring shape. It has two parameters: `alpha` (EWMA smoothing factor, e.g. 0.3) and a baseline warmup window (e.g. the first 5 observations are used to seed `mu` and `sigma` and are not themselves scored).

The question is what `x` is.

## Attempt 1: EWMA on the raw ratio

The naive thing — and the thing every example on the internet will show you — is to put the raw ratio straight into the EWMA. So `x_t = ratio_t`, `mu_t = alpha * x_t + (1 - alpha) * mu_{t-1}`, `sigma_t` updated similarly via the EWMA-of-squared-deviations recurrence.

This works, until it doesn't. Two failure modes show up immediately on real data.

**Failure 1: predictions leave the unit interval.** If you have a baseline EWMA mean of 0.62 and a slope upward, the EWMA's _prediction_ for the next value extrapolates linearly in raw space. Within a few periods of an upward shift, your "expected" cache-hit rate is 1.07. There is no such thing as a 107% cache-hit rate. The model has no idea the metric is bounded; it just keeps walking.

This is annoying for two reasons. First, the prediction interval (`mu ± 2*sigma`) starts including impossible values, which makes any downstream tool that uses the interval — say, a dashboard that renders it as a band — render garbage. Second, when the metric eventually saturates near 1.0, the residuals are systematically negative (real value < unbounded expectation), which inflates `sigma` in the wrong direction and stretches the prediction interval further into impossibility. The errors compound.

**Failure 2: variance is wrong near the boundaries.** The variance of a bounded metric is not constant in its mean. A binomial-shaped quantity at p = 0.5 has the largest possible variance; at p = 0.05 or p = 0.95 the variance is much smaller. EWMA-on-raw treats variance as a free parameter to be estimated, which is fine in the middle of the range, but near the boundaries it overestimates wildly. A stable cache-hit rate at 0.95 with day-to-day jitter of ±0.02 produces an EWMA `sigma` near 0.02. Then the rate moves to 0.93 — well within the natural binomial jitter at that level — and the score comes out at 1.0σ. Acceptable. But the same absolute move at 0.50 → 0.48 should not be remotely the same magnitude in distributional terms, and EWMA-on-raw treats it identically.

Both failures have the same root cause: the metric lives on a manifold (the unit interval) but you are doing arithmetic in the ambient space (the real line).

## Attempt 2: EWMA in logit space

The fix is a one-line change at the boundary of the scorer. Before pushing the value into the EWMA, transform with the logit function:

```
logit(p) = log(p / (1 - p))
```

This maps `(0, 1)` onto `(-∞, +∞)`. The EWMA now lives on the real line, which is where its math actually applies. After scoring, you can transform back with `expit(z) = 1 / (1 + exp(-z))` if you want to render predictions in `[0, 1]`.

Three things become free:

1. **Predictions are always in `[0, 1]`.** `expit` of any real number is bounded. The dashboard band can no longer render garbage.
2. **Variance is symmetric in logit space.** A move from 0.50 to 0.48 is `logit(0.48) - logit(0.50) ≈ -0.080`. A move from 0.95 to 0.93 is `logit(0.93) - logit(0.95) ≈ -0.358`. The same _logit-space_ distance corresponds to different raw-space distances depending on where on the curve you are. That is the correct behaviour: a 2-percentage-point drop near the boundary is a much larger event than a 2-point drop in the middle.
3. **The "regime change" interpretation matches intuition.** A jump from 50% to 75% is `logit(0.75) - logit(0.50) ≈ 1.099`. A jump from 75% to 90% is `logit(0.90) - logit(0.75) ≈ 1.099`. Same logit-space distance, both qualitatively "the cache got noticeably better." Raw-space treats the first as a 25-point move and the second as a 15-point move, which does not match how anyone actually thinks about cache improvements.

The price is that the transform has poles at 0 and 1. `logit(0)` is `-∞`. So you need a `safeLogit` that clamps inputs to `[epsilon, 1 - epsilon]` for some small epsilon, returning `±large_finite` instead of crashing. I use `epsilon = 1e-6`. In practice the only values that hit clamping are `0.0` and `1.0` exactly, which on real data show up only when the day had zero events.

## Worked example: the live cache-hit jump

Here is what made me trust the logit-space scorer: a real shift in my own data, caught with a clean signal that the raw-space version would have buried.

Setup: rolling window of 14 days, EWMA alpha = 0.3, baseline warmup = 5 days, threshold |z| > 2.5.

Days 2026-04-13 through 2026-04-20 had cache-hit ratios in a tight band around 0.48 (range 0.44 to 0.52). EWMA in logit space converged to `mu ≈ logit(0.48) ≈ -0.080`, `sigma ≈ 0.08` (in logit space). The five-day warmup window absorbed normal jitter.

On 2026-04-21 the ratio jumped to 0.77. In logit space, `logit(0.77) ≈ 1.208`. The score:

```
z = (1.208 - (-0.080)) / 0.08 ≈ +43.97
```

Scorer flagged immediately. Over the next two days the ratio held at ~0.77, the EWMA absorbed the new level (`mu` walked toward `logit(0.77)`), `sigma` widened to absorb the jump, and the score decayed naturally toward 2 — exactly the early-warning-then-decay-to-new-normal behaviour you want from a drift detector.

What would the raw-space version have done? `mu_raw ≈ 0.48`, `sigma_raw ≈ 0.025`. The score on 0.77:

```
z_raw = (0.77 - 0.48) / 0.025 ≈ +11.6
```

Still flags. But the next-day prediction interval in raw space is now `[0.72, 0.82]`, which is fine until the EWMA absorbs the new level and starts predicting upward — a few periods later it predicts 1.07 and the prediction band crosses 1.0. The detector still _works_ in the sense that the alert fires, but the downstream rendering is broken and the variance estimate is now wrong-direction-biased because all subsequent residuals will be slightly negative.

In logit space none of that happens. The "expected next" is `expit(mu) ≈ 0.77` and the band is bounded.

## Two rules that fell out

I did not start with the rules below. I started with "EWMA, in logit space, looks more correct on a whiteboard." The rules came from running the scorer against four months of real per-day data and watching it misbehave on the cases the math hadn't covered.

### Rule 1: only score days with new evidence

On days with zero token events, the cache-hit ratio is undefined — `0 / 0`. The scorer should not produce a number for these days. But for a human reading a chart, a gap is annoying — the EWMA line should not have visual holes.

The split I settled on: carry the EWMA `mu` and `sigma` forward across undefined days for display continuity, but mark those days `status: "undefined"` and do not feed them into the scorer. The next day with real data scores against the EWMA as it was before the gap, plus the natural alpha-decay of confidence (which I currently do not model — open question whether to widen `sigma` over gaps).

The reason this matters: if you _do_ feed forward an undefined day as if it were a real observation at the carried-forward `mu`, you mark the same day-of-no-data as "drifted" forever once the next real value moves — the carried day looks like a flat baseline that was then violated. That is not what happened. What happened is: nothing happened, then something happened. The status field is the honest representation.

### Rule 2: walk back past undefined and warmup days when looking for "the most recent flagged day"

The cron form of the detector exits with code 2 if the most recent _scored_ day flagged. Naively that means "look at yesterday." But yesterday might be undefined (no events) or inside the warmup window (the first 5 days of the lookback). If you only check yesterday, a real Friday drop will not fire on Saturday's cron.

The fix is to walk backward through the day series, skip days with status `undefined` or `warmup`, and check the most recent day with status `scored`. If that day is flagged, exit 2. If it is clean, exit 0. The cron has the right semantics: "did the most recent day I have evidence for, fire?"

This is a small thing. It is also exactly the kind of thing that does not show up in synthetic-fixture tests, because synthetic fixtures rarely include realistic gap patterns.

## The denominator bug, and why smoke-testing matters

The first version of the detector used:

```
ratio = cached_tokens / input_tokens
```

This is the textbook "cache-hit rate" formula. Cached over input. Bounded in `[0, 1]`. Standard.

I ran the smoke test against my live `queue.jsonl`. The first day's ratio came out as `3.25`. `safeLogit` threw because `3.25 > 1 - epsilon`.

The cause turned out to be: different upstream sources disagree on whether `input_tokens` is reported as "all input tokens including the cached portion" or "input tokens not counting the cached portion." Some report inclusive, some report exclusive. Mixed within the same `queue.jsonl` because I'm running multiple agents through the same proxy and they emit different shapes.

If `input_tokens` is exclusive (cached not included), then `cached / input` can be greater than 1 — e.g. 65k cached tokens against 20k fresh tokens reads as ratio 3.25.

The fix is to use a denominator that doesn't depend on the upstream's interpretation:

```
ratio = cached / (input + cached)
```

This is identical to `cached / input` for sources that report `input` exclusively (the unified denominator just adds back what was excluded). It reduces to the standard cache-hit rate for sources that report `input` inclusively (the `+ cached` is double-counting, but the `cached` in the numerator is then the same fraction, and the result is `cached / total` either way). It is bounded in `[0, 1]` regardless of which convention the source uses.

The post-fix ratio for the same day was 0.48. The smoke test passed.

The lesson is small but worth stating: the choice of denominator is not "the textbook formula plus a bug." It's a real modelling decision that depends on which token-counting convention your data sources happen to use. If you do not control the sources, pick the denominator that is invariant under the conventions you'll see.

I would not have found this bug without running the scorer against live data. The synthetic fixtures all used `input_tokens` consistently. They had to — they were generated by one piece of code. Real data has multiple conventions colliding inside the same file.

## What I would do differently

Two things, in order of how much I expect them to bite.

First: model the gap-widens-confidence behaviour explicitly. Right now an EWMA with alpha = 0.3 carried forward across a 7-day gap acts as if it has full confidence in `sigma` from a week ago. That is probably wrong — confidence should decay over a gap, even if the location estimate doesn't. The fix is small: widen `sigma` by a factor of `(1 - alpha)^(-gap_days)` or similar. I have not done this yet because I do not have enough gap-data examples to calibrate the constant honestly.

Second: pick the logit transform once, at the boundary of the scorer module, rather than letting downstream consumers work in either space. I currently expose both `mu_logit` and `expit(mu_logit)` for rendering convenience, and the inevitable result is that a future caller will compute deltas in raw space and get the wrong answer near the boundaries. The right shape is: the public API surface returns logit-space numbers and a single `to_raw(x)` helper, and downstream code that wants a percentage knows it has to convert.

The shorter version of the lesson: when your metric is bounded, do the math in a space where it isn't. Logit is the cheapest possible transform that buys you that.

## Links

- [Logit function — Wikipedia](https://en.wikipedia.org/wiki/Logit)
- [Exponentially weighted moving average — NIST handbook](https://www.itl.nist.gov/div898/handbook/pmc/section4/pmc431.htm)
- [Robust scale estimation: MAD vs stdev](https://en.wikipedia.org/wiki/Median_absolute_deviation)
