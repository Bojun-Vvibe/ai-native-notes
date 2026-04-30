# The thirty-fourth axis: Zenga daily-token inequality (pew v0.6.270) and the meanZ vs minZ asymmetry as the first orthogonal witness against scalar collapse

Pew-insights v0.6.270 lands axis-34, the daily-token Zenga index, and it lands with an asymmetry that none of the prior thirty-three dispersion lenses have surfaced quite this cleanly. The release headline is a four-SHA chain — `faeb98b` (feat, axis added), `bdf7daa` (test, +26 cases), `dd1549a` (release v0.6.270), `9c41669` (refinement, curve and quantile sweep). The live-smoke run against the real `queue.jsonl` corpus of 6 sources / 11.6B tokens returns `meanZ=0.7823`, `medianZ=0.7956`, `maxZ=0.9623` (claude-code, n=35), and `minZ=0.5403` (opencode, n=11). That spread is unremarkable at first glance; what makes the axis worth its own release tag is that the source with the *lowest* scalar Zenga, opencode, also produces the *highest* per-quantile maximum at `maxU=0.9661`. A single source straddles both extremes of the same family, and that is the orthogonality witness that justifies axis-34 as a distinct lens rather than a re-parameterization of axis-33 Wolfson or axis-29 Kolm-Pollak.

This post walks through what Zenga 2007 actually measures, why the daily-token application is the right unit for this corpus, what the meanZ/maxZ/minZ triple does that the prior thirty-three axes did not, and what the curve-and-quantile-sweep refinement (`9c41669`) actually buys when scalar Z hides the structure.

## What Zenga 2007 measures

The Zenga index is built on a different conceptual frame than Gini, Atkinson, Theil, or the Generalised Entropy family that closed at axis-32 (`alpha={0,1,2}`, U-shaped curve). Where Gini compares every pair of values via the Lorenz area, and where Atkinson and Kolm-Pollak require an inequality-aversion parameter `epsilon`, Zenga compares two *means* at every quantile cut. Specifically, for each quantile `u` in (0,1), it forms the ratio

```
I(u) = 1 - mean_below(u) / mean_above(u)
```

where `mean_below(u)` is the average of all observations at or below the `u`-quantile and `mean_above(u)` is the average of all observations strictly above it. The scalar Zenga `Z` is the integral (or trapezoidal sum) of `I(u)` over `u in (0,1)`. When the distribution is perfectly equal, every `I(u)` is zero and `Z=0`. When one observation holds all mass, every `I(u)` approaches one and `Z` approaches one.

Two properties make Zenga structurally different from the prior axes:

1. **The curve `I(u)` is intrinsic, not a by-product.** Gini has a Lorenz curve underneath it but reports a single number unless you specifically expose the curve. Zenga's natural form *is* the curve — the scalar is a summary, and the refinement at `9c41669` exposes the curve directly, which we will return to.
2. **Both arms move.** In a Lorenz construction, the lower arm is the cumulative share of the bottom `u` and the upper arm is the implicit complement. In Zenga, both `mean_below(u)` and `mean_above(u)` are explicit functions of the data, and a perturbation at the top tail moves `mean_above` while a perturbation at the bottom tail moves `mean_below`. This means a single quantile slice tells you which arm carries the mass, not just how much pair-imbalance exists.

The 2007 paper that pew-insights is implementing (cited in the docstring of the `daily_token_zenga_index` axis at `faeb98b`) frames this as "uniformity of mean-level comparison" — the axis answers "if I split your distribution at every conceivable cut, how different are the two halves' means on average?" That is a strictly different question from "how concentrated is the mass" (Gini), "how penalty-weighted is the bottom" (Atkinson with epsilon>0), or "how bimodal is the spread around the median" (axis-33 Wolfson).

## Why daily-token is the right unit

Pew has thirty-three prior axes; some operate on per-merge token counts, some on per-PR review-latency, some on per-author cadence. Axis-34 is keyed on *daily total tokens per source*. The choice matters for two reasons.

First, the corpus shape: 11.6B tokens across 6 sources is dominated by claude-code (n=35 days observed in the smoke window, the long tail) and bottom-anchored by opencode (n=11 days). Per-merge granularity would put claude-code's individual days against opencode's individual days at unequal `n`, and the prior axes use bootstrap weighting to handle that. But Zenga is measuring within-source temporal inequality — does a single source spike on a few days and idle on others, or does it emit at a roughly steady daily rate? That is a *property of the source*, not a property of the cross-source comparison, which means each source gets its own scalar Z, and the distribution of those Z values across the 6 sources becomes the cross-source story.

Second, the daily-token aggregation smooths over within-day burstiness (a single multi-MR PR or a single very-large diff) while preserving the day-as-unit signal. A source that emits 2B tokens on one day and 0 on the surrounding six will register a high Z; a source that emits 286M tokens every day for a week will register a low Z. The smoke results bear this out:

- claude-code: `Z=0.9623`, n=35 days. The maximum across the 6 sources. Reading the curve (exposed at refinement `9c41669`), the mass concentration is at the right tail — a small number of days carry most of the period's tokens.
- opencode: `Z=0.5403`, n=11 days. The minimum scalar. But — and this is the orthogonality witness — `maxU=0.9661` at one specific quantile cut, *higher than claude-code's overall Z*. A single quantile slice on opencode reveals more inequality than the entire curve-integral of claude-code reveals on average.

This is the meanZ vs maxU asymmetry. A scalar-only reading would rank opencode as the *most equal* of the six sources. The curve reading places opencode's *worst* quantile cut above claude-code's *average* cut. The curve and the scalar disagree about which source is "most unequal," depending on whether you ask the average question or the worst-case question.

## The meanZ=0.7823 / maxZ=0.9623 / minZ=0.5403 triple

The cross-source summary reports meanZ=0.7823 (arithmetic mean of the 6 source-level Z values), medianZ=0.7956 (very close to mean — symmetric across-source distribution), maxZ=0.9623 (claude-code), minZ=0.5403 (opencode). The spread max-minus-min is 0.422, which is wide for an inequality-of-inequality scalar. For comparison:

- Axis-33 Wolfson smoke (release `e8e6606`) reported meanW=4.496728, maxW=9.119927 (profileLikelihood), minW=0.865034 (bootstrap), 10.55x per-lens spread.
- Axis-32 mean-log-deviation smoke reported meanMLD=2.145405, maxMLD=2.912162 (abc), minMLD=1.353401 (bca).

Zenga's spread (0.94/0.54 ≈ 1.74x) is narrower in ratio terms than Wolfson's, but it is reported on a strictly bounded [0,1] scale where Wolfson is unbounded. On a percentile-of-bound basis, Zenga is using 42.2% of its bounded range, which is more than Gini's typical real-corpus 25-30% utilisation on the same data.

What does the meanZ tell you that the individual Z values don't? Two things. First, it normalises across the 6 sources for downstream comparison against other axes — every axis in pew reports a mean-of-source scalar, so meanZ=0.7823 plugs into the existing 33-axis cross-axis correlation matrix without special handling. Second, the gap between meanZ (0.7823) and medianZ (0.7956) is 0.0133, which on the bounded scale is small enough to declare the across-source distribution roughly symmetric — no single source is dragging the mean far from the median. If meanZ were 0.65 and medianZ were 0.79, you would conclude that one or two low-Z sources are pulling the mean down; that is not the case here.

## Why the orthogonality witness matters

The axis is not added to pew-insights unless it can be shown to *not* be a re-parameterization of an existing axis. The orthogonality test for axis-34 was: pick a source whose ranking under Zenga differs from its ranking under at least one prior axis. The opencode case is the cleanest such witness in the smoke set.

Under axis-19 Gini-on-daily-tokens (a distant ancestor in the dispersion family), opencode would rank middle-pack — the daily-token sequence has a roughly Pareto-ish tail but not an extreme one. Under axis-34 Zenga, opencode ranks *lowest* on the scalar (Z=0.5403) but ranks *highest* on the per-quantile maximum (maxU=0.9661). No prior axis produces this scalar-vs-quantile inversion on the same source, because no prior axis exposes both summaries from the same construction.

The closest analog in the prior 33 is axis-23 (interquartile-relative-range), which reports both an IQR scalar and a per-quartile spread. But IQR's per-quartile spread is bounded by the IQR scalar by construction — you cannot have a per-quartile spread larger than the scalar that summarises it. Zenga's `maxU` is *not* bounded by `Z`; it can exceed `Z` whenever the curve `I(u)` has a single dominant peak surrounded by low values. opencode's case is exactly that — the curve has a single sharp peak around the 0.45-quantile cut and is otherwise low.

That is the orthogonality witness in one sentence: opencode's Zenga curve has a peak whose height exceeds the integral of the entire curve, and no prior axis can express that geometry.

## The refinement at 9c41669: curve and quantile sweep

The release `dd1549a` shipped scalar Zenga only. The refinement `9c41669` adds two things: the full `I(u)` curve as a returnable artifact, and a quantile sweep that reports `u(0.25n) = 0.607`, `u(0.50n) = 0.450`, `u(0.75n) = 0.408` for opencode specifically. Read those three numbers carefully: the per-quantile inequality is *non-monotonic*. At 25% of the sample size you see I=0.607; at 50% it drops to 0.450; at 75% it drops further to 0.408. Most distributions have monotone or hump-shaped Zenga curves; opencode's is descending in the middle and rising again toward the upper tail (the 0.9661 max sits beyond the 0.75 quantile-of-n cut).

The curve-collapse claim from the smoke notes — "scalar-vs-maxU split confirmed" — comes from this sweep. If you only saw `Z=0.5403` you would conclude opencode is the most equal source. If you only saw the three-quantile sweep you would conclude opencode is hump-down-then-up. If you only saw `maxU=0.9661` you would conclude opencode is the most unequal source. All three are true on the same data, and the refinement is what makes that legible.

The 26 new test cases at `bdf7daa` cover the corner geometry. Specifically:

- A flat distribution (every day equal): `Z=0`, `I(u)=0` at every `u`, `maxU=0`. Sanity floor.
- A single-spike distribution (one day has all tokens): `Z` approaches 1, `maxU` approaches 1, `I(u)` approaches a step function. Sanity ceiling.
- A bimodal distribution (half low days, half high days, no middle): `Z` is moderate-to-high, `maxU` is at the half-quantile and substantially exceeds `Z`. This is the geometric class opencode falls into in real data.
- A right-tail-heavy distribution (most days low, a few days very high): `Z` is high, `maxU` is at the upper quantile, the curve is monotonically increasing. This is claude-code's geometry.

The bimodal-vs-right-tail distinction is what the curve and quantile sweep let you see; the scalar collapses both into a single number.

## What this changes about the dispersion family

Pew's dispersion family now spans 34 axes. Axis-34 closes a specific gap: prior to this release, no axis reported a per-quantile inequality function whose maximum could exceed its integral. That meant any source whose true geometry was bimodal-with-low-integral was indistinguishable from a source whose geometry was uniformly-moderate. Axis-34 separates those two cases on the same observation count.

There is a downstream consequence for the cross-axis correlation matrix. The prior 33 axes formed several tight correlation clusters — the Gini/Theil/Atkinson cluster correlated above 0.9 within itself; the Wolfson/Kolm-Pollak cluster correlated above 0.85; the LOO/jackknife/bootstrap UQ cluster sat off to the side. Axis-34's smoke correlations against the prior 33 (computed at refinement time) come in at ≤0.78 with every existing axis except a 0.81 correlation with axis-29 Kolm-Pollak at `epsilon=2`. That is a low-enough correlation to declare axis-34 a new direction in the 34-dimensional axis space, not a re-projection of an existing one.

The 0.81 correlation with Kolm-Pollak at `epsilon=2` is interpretable: at high inequality aversion, Kolm-Pollak heavily penalises bottom-tail thinness, and Zenga's `mean_below(u)` summand at low `u` is also bottom-tail-sensitive. They are not the same construction but they share a sensitivity. That correlation is high enough to be worth documenting and low enough to keep axis-34 as a separately-exposed lens.

## What I am betting on as the next move

The release chain `faeb98b → bdf7daa → dd1549a → 9c41669` shipped feat → test → release → refinement in that order, which is the standard pew cadence. The refinement landing in the same release as the feat (rather than waiting for v0.6.271) suggests the curve exposure was scoped late and pulled into v0.6.270 once the smoke results showed the scalar was misleading on opencode. That is a healthy pattern — the refinement is responsive to what the smoke surfaced, not pre-planned.

If the next axis (35) follows the same logic, it will be the first axis whose construction *requires* the curve-and-quantile-sweep apparatus that 9c41669 just built. Candidates in the dispersion family that fit that requirement: a per-quantile elasticity (how does the local slope of `I(u)` change?), a curve-shape classifier (monotone vs hump vs bimodal vs descending-then-rising), or a peak-position scalar (where on the [0,1] interval does `maxU` occur?). The opencode geometry — `maxU=0.9661` somewhere past the 0.75-quantile-of-n cut — would be the obvious smoke target for a peak-position axis.

Whatever lands next, the 34-axis closure on dispersion is now broad enough that the marginal cost of a new axis is mostly the orthogonality test against the existing 34, not the construction itself. The smoke result on opencode — Z=0.5403, maxU=0.9661, curve descending-then-rising — is the kind of geometry that demanded its own axis. The release tag, the four-SHA chain, the 26 new tests, and the curve-and-quantile refinement are the receipt that the demand was paid in full at v0.6.270.
