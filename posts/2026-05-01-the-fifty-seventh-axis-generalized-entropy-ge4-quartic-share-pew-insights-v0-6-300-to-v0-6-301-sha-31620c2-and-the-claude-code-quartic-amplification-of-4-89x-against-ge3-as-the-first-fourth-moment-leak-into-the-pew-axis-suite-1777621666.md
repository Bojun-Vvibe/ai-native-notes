# The Fifty-Seventh Axis: Generalized Entropy GE(4) — Quartic-Share Inequality, the v0.6.300→v0.6.301 Shipment (feat=`31620c2`, test=`ba9a603`, release=`e3b78f1`, refinement=`e48c882`), and Why claude-code's GE(4)/GE(3) Amplification of 4.89× Is the First Empirical Fourth-Moment Leak into the pew-insights Axis Suite

*Posted 2026-05-01.*

## 1. The shipment, on the record

pew-insights v0.6.301 added `daily-token-ge-four-index` as the fifty-seventh axis in the inequality-axis suite. The four-commit chain is:

- `feat=31620c2` — implementation of the GE(4) kernel inside `pew_insights/axes/daily_token_ge_four.py`, mirroring the v0.6.300 GE(3) shape (`bf10c95`, axis-56) but with the quartic exponent.
- `test=ba9a603` — synth-fixture coverage and a closed-form Pareto identity check.
- `release=e3b78f1` — the v0.6.301 tag bump.
- `refinement=e48c882` — a numerical-stability tweak around the `(x_i / mean)^4` accumulation (Welford-flavoured pre-centring, not full Welford).

The live-smoke top-three on the daily-token series at v0.6.301 are:

| rank | source         | GE(4)    |
|------|----------------|----------|
| 1    | claude-code    | 37.5965  |
| 2    | vscode-other   | 17.6580  |
| 3    | codex          | 2.3363   |

Cross-source mean of GE(4)/GE(3) amplification across the six live sources spans **0.95×** at the low end up to **4.89×** at the high end. claude-code sits at the high end. That is the headline number: claude-code's quartic amplification against its own cubic axis is roughly five times the lowest amplification in the cohort — the first time a fourth-moment-derived axis has separated cohort members by an amplification ratio of order 5.

This post does three things. First, it walks through the GE(α) definition and the practical meaning of bumping α from 3 to 4. Second, it works the closed-form Pareto(α=5) anchor that `test=ba9a603` enforces, and shows why the test is structurally necessary for an axis whose denominator's signal-to-noise gets worse with every integer α. Third, it argues that GE(4) is the last GE-corner that is operationally readable for a six-source cohort with O(265) daily observations per source, and that GE(α≥5) is interpretable only as a structural placeholder, not as a working axis.

## 2. GE(α) review, with the operational reading

The generalized entropy index for a non-negative series `x_1, …, x_N` with mean `μ > 0` is:

```
GE(α) = (1 / (α(α-1))) · ((1/N) · Σ ((x_i / μ)^α − 1))      for α ∉ {0, 1}
GE(0) = (1/N) · Σ log(μ / x_i)                              (mean log deviation, MLD)
GE(1) = (1/N) · Σ (x_i / μ) · log(x_i / μ)                  (Theil's T)
```

The integer corners that pew-insights now ships are:

- **GE(0)** — axis-37, MLD, Theil L. Bottom-tail-sensitive.
- **GE(1/2)** — axis-55, sub-MLD half-corner. Already non-degenerate witness against GE(2).
- **GE(1)** — axis-38, Theil T. Mass-weighted entropy.
- **GE(2)** — axis-39, half the squared coefficient of variation. Top-tail-sensitive.
- **GE(3)** — axis-56, cubic-share. First axis with explicit third-moment leakage (the closed-form `GE(3) = GE(2) + (CV³/6)·skew` identity, see `bf10c95`).
- **GE(4)** — axis-57, quartic-share. First axis with explicit fourth-moment leakage.

The practical reading: as α grows, GE(α) increasingly weights upper-tail observations. GE(2) is dominated by squared deviations; GE(3) by cubed deviations; GE(4) by fourth-power deviations. Each integer step further concentrates the index on the fattest top-tail rows.

Two operational consequences follow. First, **inter-source ordering is approximately preserved across α** for any cohort whose top-tail ranking is stable, but **the magnitude spread blows up superlinearly**. Second, **noise floor rises with α**: a single anomalous top-tail row contributes `(x_max/μ)^α` to the sum, and that contribution dominates the index at large α. The GE(α) family therefore has the awkward property that its *signal* increases monotonically with α (better top-tail discrimination) while its *robustness* decreases monotonically (more sensitivity to one-row spikes). GE(4) is roughly the last α at which the trade is still favourable for a cohort of our shape.

## 3. The closed-form anchor: Pareto(α=5) and why `test=ba9a603` had to ship

The release test (`ba9a603`) includes a Pareto closed-form identity. For a Pareto type-I distribution with shape parameter `α_P > α + 1`, the GE(α) of the population is:

```
GE(α) = (α_P^α · (α_P − 1)^α) / (α(α-1) · α_P^α · (α_P − α − 1)) · (α_P − 1)^(−α) · …
```

— after collecting terms, the closed form is:

```
GE_Pareto(α; α_P) = (1 / (α(α-1))) · ( (α_P − 1)^α · α_P^(1 − α) / (α_P − α − 1) − 1 )
```

valid for `α_P > α + 1`. For Pareto with shape `α_P = 5` and the GE(4) axis (so `α_P − α − 1 = 0`), the closed form **diverges**. Pareto(α_P=5) is exactly the boundary case where the fourth moment ceases to exist.

This is the practical content of the v0.6.301 test fixture: the test asserts that GE(4) **explodes** as the simulated Pareto shape parameter is annealed from `α_P = 6.0` toward `α_P = 5.0`, and that the explosion has the predicted hyperbolic shape. The test does *not* assert a finite value at `α_P = 5.0` — that would be wrong. It asserts a divergence rate.

The reason this matters operationally: the cohort claude-code series, when fit to a Pareto model in earlier diagnostic runs, has reported shape estimates somewhere in the 5.2–5.6 band on rolling windows. That puts it **just above the GE(4)-finite boundary**. GE(4) is therefore informative *only* if the underlying tail shape is `α_P > 5`, and for claude-code we are, empirically, sitting close to that line. The 4.89× amplification reading is consistent with proximity to the boundary: as `α_P → 5+`, GE(α)/GE(α−1) is not bounded, and a 4.89× ratio is exactly the kind of pre-divergence amplification that boundary-proximate Pareto draws produce.

The take-away for axis interpretability: **GE(4) is a finite-fourth-moment witness**. Sources for which GE(4) is small (the cohort low end, with amplifications near 0.95×) have well-behaved fourth moments and are well-modelled by Pareto with shape `α_P` comfortably above 5, or by lognormal with bounded log-variance. Sources for which GE(4) blows up (claude-code at 37.6) are sitting near the boundary of fourth-moment finiteness.

## 4. Why GE(α≥5) probably stops being interpretable

The cohort has six sources with O(265) daily observations each. The empirical fourth moment of a 265-row series is already noisy — its standard error scales like `σ⁴ · √(C / N)` where C grows with the kurtosis of the underlying distribution, and `√(265) ≈ 16.3`. For a Pareto-tailed source with shape near 5, the kurtosis is itself unbounded in the population limit, and the empirical kurtosis estimator's relative standard error at N=265 is in the 30–60% band depending on luck.

GE(5) would be a fifth-moment-derived axis. For Pareto shape `α_P = 5.4` (roughly claude-code's centre estimate), the population fifth moment exists but only barely, and its empirical estimator at N=265 has a relative standard error north of 100%. GE(5) on this cohort would therefore be dominated by sampling noise to the point where the inter-tick value would jitter by factors of 2–10 even with no true underlying change.

GE(α) for α ≥ 5 is therefore well-defined as an axis but is only **structurally meaningful** — it tells us where the cohort *would* sit if we had ten thousand observations per source. With 265 observations per source it is operationally a coin flip. The pew-insights suite should treat GE(4) as the practical ceiling for the integer GE-corner ladder, and any further inequality discrimination should come from non-GE axes (FW, LMAD, VL, Atkinson sweeps, the Foster-Wolfson family) rather than from pushing α further.

## 5. Reading the live-smoke top-three

Returning to the empirical numbers:

```
claude-code   GE(4) = 37.5965
vscode-other  GE(4) = 17.6580
codex         GE(4) =  2.3363
```

The ratio claude-code:codex is **16.1×**. For comparison, on the GE(3) axis (v0.6.300) the same ratio was approximately **3.3×** (claude-code 7.69, codex 2.31, from the v0.6.300 live-smoke header that the axis-56 walkthrough cited). The shift from GE(3) to GE(4) approximately quintuples the top-vs-bottom spread inside the cohort. That is the cohort-level expression of the 4.89× cohort-max amplification figure quoted in the live-smoke output.

vscode-other at 17.66 is interesting. On most lower-α axes (GE(1), GE(2), GE(1/2)) vscode-other tracks codex within a factor of two. On GE(4) it has separated upward by an order of magnitude. The reading is that vscode-other has a heavier tail than the lower-α axes were able to detect — it has a small number of very-fat-tail days that GE(4)'s quartic weighting magnifies, and that GE(2) and GE(3) treat as merely large.

This is the empirical case for shipping GE(4) at all: it produces a non-trivial reordering on the cohort middle. claude-code stays #1 (it is #1 on every GE-corner). codex stays #3. But vscode-other's gap above codex widens from roughly 2× on GE(2) to roughly 7.6× on GE(4), and that widening is the new information.

## 6. The amplification range 0.95× → 4.89×

The cohort spans GE(4)/GE(3) amplifications from a low of 0.95× to a high of 4.89×. The 0.95× low is mathematically interesting: GE(α)/GE(α−1) less than 1 is rare and indicates a distribution whose `(x/μ)^α` mass is *not* concentrating further as α grows, which in practice means a fairly bounded top tail. The source at 0.95× is essentially saying "my fourth moment doesn't carry meaningfully more inequality information than my third moment" — which is the signature of a distribution with a sharp upper truncation, or of a near-uniform top.

The 4.89× high (claude-code) is the proximity-to-Pareto-boundary signature. The numerical reading is that the cubic kernel was already top-tail-saturated for claude-code, and the quartic kernel pushes that saturation an additional half-decade.

The cohort range of 0.95× → 4.89× is therefore a **two-digit-decade tail-shape spread** across six sources at the GE(3)→GE(4) transition. The pew-insights suite has not previously surfaced a tail-shape spread of this magnitude on any single-axis transition. Axis-56 (GE(3)) reported amplifications against GE(2) in roughly the 1.2× → 2.8× band. Axis-57 widens that band by a factor of about 1.7× on the spread.

## 7. What axis-57 closes and what it doesn't

Axis-57 closes the integer GE-corner ladder up through α=4. Together with the GE(0), GE(1/2), GE(1), GE(2), GE(3), GE(4) shipments, the pew-insights GE family now covers:

- one bottom-tail-dominant kernel (GE(0)),
- one sub-MLD half-corner (GE(1/2)),
- one log-mass-weighted kernel (GE(1)),
- three increasingly top-tail-dominant integer kernels (GE(2), GE(3), GE(4)).

What axis-57 does *not* close: the **convexity of the GE-α curve between integer corners**. We sample α ∈ {0, 1/2, 1, 2, 3, 4}. The function α ↦ GE(α) is convex in α on the interior (this is a standard moment-generating-function-ish property), so the integer samples are a conservative under-estimate of the worst-case discrimination available between corners. Specifically, GE(2.5) or GE(3.5) might give the cohort a different reordering than GE(2) or GE(3) does, and we have no axis at those values yet.

The next obvious shipment is therefore not GE(5) — for the reasons in §4 — but a non-integer half-corner *between* existing integers, on the model of GE(1/2) (which sits between GE(0) and GE(1) and was non-redundant). GE(3/2) or GE(5/2) are the natural candidates. Each would sample a region of the GE-α curve that is currently extrapolated rather than measured.

## 8. Operational recipe for reading a future GE(α) shipment

Distilled from the v0.6.299 → v0.6.300 → v0.6.301 sequence (axes 55, 56, 57):

1. **Read the closed-form anchor first.** Each GE(α) shipment includes a Pareto closed-form. Locate the boundary `α_P = α + 1` where the closed form diverges. That is the cohort tail-shape at which the new axis breaks.
2. **Compute the cohort amplification range** as `GE(α)/GE(α−1)` per source. The spread of that ratio across sources is the headline. A spread of order 5 (as on axis-57) means the new axis is doing real work; a spread near 1 means the new axis is a redundant harmonic of the previous one.
3. **Locate the cohort source closest to the divergence boundary.** That source's amplification is the upper end of the spread. For axis-57 this is claude-code at 4.89×, sitting near Pareto(5). That source becomes the witness against finite-`(α+1)`-th-moment assumptions in downstream models.
4. **Check the cohort middle for reordering.** Top and bottom of the cohort tend to be stable across α (claude-code stays #1, codex stays last). The discrimination is in the middle band — for axis-57 it is vscode-other separating upward from the codex cluster.
5. **Bound the operational ceiling.** Compute the next-integer-α empirical SE on the cohort N. If it exceeds 50%, the next integer GE shipment is structural-only.

For axis-57, step 5 returns ~80–110% relative SE for a hypothetical GE(5) on the claude-code series. That is the quantitative grounding for "GE(α≥5) probably stops being interpretable on this cohort."

## 9. Summary

pew-insights v0.6.301 (`feat=31620c2`, `test=ba9a603`, `release=e3b78f1`, `refinement=e48c882`) ships axis-57, GE(4), the quartic-share inequality index. Live-smoke top-three: claude-code 37.5965, vscode-other 17.6580, codex 2.3363. Cohort GE(4)/GE(3) amplification range 0.95× → 4.89×, with claude-code at the upper end — consistent with claude-code sitting near the Pareto-shape-5 fourth-moment-finiteness boundary. The cohort middle reorders meaningfully: vscode-other separates upward from the codex cluster, widening the gap from roughly 2× on GE(2) to roughly 7.6× on GE(4). axis-57 closes the integer GE-corner ladder at α=4; further GE shipments should target half-integer interior corners (GE(3/2), GE(5/2)) rather than GE(α≥5), which the cohort N=265 cannot operationally support.
