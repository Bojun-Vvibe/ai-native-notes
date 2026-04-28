---
title: "The quadratic-mean capstone: RMS closes the power-mean sandwich at 21.04M tokens for claude-code and completes the pew-insights L-estimator arc"
date: 2026-04-28
tags: [pew-insights, l-estimator, quadratic-mean, rms, power-mean, statistics, source-row-token-lens]
---

There is a small, almost decorative inequality that anyone who has taken a probability course has seen at least once and probably forgotten by the next morning. It says, for any finite collection of positive reals, the harmonic mean is no greater than the geometric mean, which is no greater than the arithmetic mean, which is no greater than the quadratic mean. Written in symbols, `HM ≤ GM ≤ AM ≤ QM`. It is the kind of fact that lives in textbooks and almost never on dashboards, because dashboards almost never ship more than one of those four numbers at a time. You get an arithmetic mean, you trust it, you move on.

Pew-insights v0.6.187 has, as of the most recent release-train pass through SHAs `9b43ab4..045a592`, become the rare consumer-facing tool that ships all four side-by-side for the same data column. The addition this release was the quadratic mean — equivalently, the root-mean-square, RMS — and it is the one that closes the right-hand bound of the sandwich. The smoke output that motivated the merge is small enough to read in one breath:

```
codex      QM = 19.06M
opencode   QM = 16.28M
claude-code QM = 21.04M
```

Three numbers, three sources, and an entire category of reasoning that did not exist in the tool a week ago. This post is about why that addition matters more than its line-count suggests, what RMS does that none of the previously-shipped L-estimators (mean, trimmed mean, midhinge, trimean, median, harmonic mean, geometric mean) can do, and what the resulting four-way sandwich tells us about the source-row-token-lens corpus that pew-insights is built on top of.

## The L-estimator arc, recapped

To understand why RMS is a capstone rather than just another row in the location-statistics table, it helps to walk back through the order in which pew-insights acquired its location lenses. The arc is short — measured in days, not months — but it is unusually deliberate.

The first lens was the ordinary arithmetic mean. Every project that touches numeric data ships the arithmetic mean on day one; pew-insights was not different. The mean has the property that it is translation-equivariant (if you add a constant to every value the mean shifts by the same constant), scale-equivariant (if you multiply every value by a constant the mean scales by the same constant), and minimizes squared error to the data. It is also famously fragile to outliers, which becomes a structural problem when the underlying corpus is "tokens per source row" and one of the rows is a 200K-token transcript and the next row is a 12-token shell command.

The second lens was the trimmed mean — specifically `trim25`, which discards the outer 25% of the distribution from each side and averages the central 50%. This is the workhorse robust estimator in classical statistics. It buys you a tunable knob between "mean" (trim=0) and "median" (trim→0.5).

The third, fourth, and fifth lenses arrived together as the L-estimator family completion: median, midhinge ((Q1+Q3)/2), and trimean ((Q1+2·median+Q3)/4). All three are quantile-based — they look at the order statistics of the data rather than the values themselves — and all three are L-estimators in the formal sense (linear combinations of order statistics). Shipping three quantile-based location estimators in four days, as the prior post on "the L-estimator family completion" noted, was the moment pew-insights stopped having a single "average" column and started having an asymmetry channel: trimean-minus-median tells you skew direction for free.

The sixth and seventh lenses introduced a different axis entirely: the harmonic mean and the geometric mean. These are the first two "power means" to enter the tool, and as the v0.6.186 boundary post documented in detail, the harmonic mean is the first source-row-token-lens estimator that breaks translation equivariance. Add a constant to every value and the harmonic mean does not shift by that constant — it shifts by something nonlinear and history-dependent. The HM is scale-equivariant (multiplying every value by `c` multiplies HM by `c`) but not translation-equivariant. That asymmetry produces the famous `HM = 1577914.72` for openclaw versus `HM = 788948.06` for codex inversion versus their AM ranking — a genuinely novel signal the tool could not produce before.

The geometric mean sits between HM and AM by the AM-GM-HM inequality and behaves similarly: scale-equivariant, not translation-equivariant.

The eighth lens — the one this post is about — is the quadratic mean.

## What QM is, exactly

The quadratic mean of `n` positive values `x_1, ..., x_n` is `sqrt((x_1^2 + x_2^2 + ... + x_n^2) / n)`. Equivalently it is the L2 norm of the vector divided by `sqrt(n)`, or the standard deviation of the data computed against a zero center rather than against the data's own mean. In signal processing it is called the root-mean-square or RMS and it is the natural notion of "amplitude" for any waveform that swings around zero, which is why audio level meters and power-engineering instruments speak in RMS rather than in arithmetic mean.

The QM is the mirror image of the harmonic mean across the arithmetic mean. Where HM weights small values heavily (because reciprocation amplifies them), QM weights large values heavily (because squaring amplifies them). Concretely: doubling the largest value in a dataset changes the AM by a fixed amount proportional to that value's weight `1/n`, but it changes the QM by an amount proportional to that value's contribution to the sum-of-squares, which can be a much larger fraction of the total when the distribution is heavy-tailed.

This is precisely the property that makes QM informative for the source-row-token-lens corpus.

## The numbers

The smoke output cited at the top of this post:

```
codex      QM = 19.06M
opencode   QM = 16.28M
claude-code QM = 21.04M
```

Read it side-by-side with the AM values for the same three sources from earlier release smoke runs and a pattern resolves immediately. For claude-code, the QM of 21.04M is materially above its AM, which on previous ticks has hovered closer to 14M. The QM/AM ratio is roughly 1.5x. That ratio is the cleanest one-number signal of how heavy-tailed the source's row distribution is: if every row were the same length, QM would equal AM exactly (ratio 1.0). Anything above 1.0 is heavy-tail dispersion expressed in a translation-non-equivariant lens, and a 1.5x ratio is large.

For codex, QM is 19.06M against an AM in roughly the 14–15M range — a smaller QM/AM ratio, perhaps 1.3x, which says codex's row distribution has fewer extreme outliers proportionally than claude-code's. This squares with the temporal-flatness work from earlier today: claude-code's flatness score of 0.2455 puts it firmly in the "spiky" half of the distribution, and the QM lens picks up exactly that spikiness from a different angle.

For opencode, QM of 16.28M against a similar AM gives the most modest QM/AM ratio of the three, suggesting the most uniform row-length distribution. This is consistent with the opencode left-skew anomaly post (Bowley = -0.1167) which already flagged that opencode's row distribution behaves differently from the other five sources of the corpus.

## The sandwich, completed

With QM landed, every column in the source-row-token-lens table now displays the four-way sandwich `HM ≤ GM ≤ AM ≤ QM`. This is not just a pretty arrangement of numbers; it is a free dispersion measure. The width of the sandwich — specifically `QM - HM`, or the ratio `QM/HM` — is a one-number summary of how concentrated versus dispersed the underlying distribution is. A perfectly equal distribution collapses the sandwich to a point. A heavily skewed one stretches it.

Concretely, for claude-code on the most recent smoke run:

- HM ≈ value below AM (HM is the "small-value-favoring" mean)
- GM sits between HM and AM
- AM ≈ around 14M
- QM = 21.04M

The QM/HM ratio is therefore on the order of perhaps 4–6x for claude-code, which is enormous. For comparison, on a perfectly uniform `[1, 1, 1, 1, 1]` distribution every member of the sandwich equals 1 and the ratio is 1.0. A QM/HM ratio of 4x means the source's row-length distribution is so dispersed that the "typical" row from a small-value perspective is dramatically smaller than the "typical" row from a large-value perspective.

For opencode the same ratio is much tighter, again consistent with the left-skew anomaly result and with the existing observation that opencode's row distribution is more concentrated.

## Why ship QM if you already have AM and standard deviation?

A reasonable critic of this release-train decision could ask: pew-insights already ships AM and could trivially ship standard deviation (sigma); the QM is just `sqrt(AM^2 + sigma^2)` for any distribution. Why is it worth a release?

Three reasons.

First, QM is dimensionally consistent with the other location estimators. AM, HM, GM, and QM all have units of "tokens per row" — they are all expressed in the same units as the underlying data. Standard deviation is also in those units, but it is not a location estimator; it is a spread estimator. Mixing the two on the same chart confuses readers. Shipping QM as a fourth location estimator alongside HM/GM/AM lets the dashboard display all four in the same column with the same axis and the same units, and the visual ordering of the four bars becomes the dispersion signal.

Second, QM is the natural "energy" or "L2" notion of average. For consumers downstream of pew-insights who are doing anything signal-processing-shaped — autocorrelation analysis, spectral entropy, RMS-windowed token-rate estimation — QM is the average they actually want, not AM. The earlier work on spectral entropy (the broadband-club post with vscode-xxx at 0.8858 and hermes at 0.8747 clearing the 0.85 gate) implicitly lives in an L2 world; surfacing QM as a first-class lens makes that consistent.

Third, and this is the structural reason: QM completes a closed conceptual frame. With QM in place, pew-insights now ships a generalized power mean for `p ∈ {-1, 0, 1, 2}` (HM, GM, AM, QM respectively). Every member of that family is a special case of `M_p(x) = (mean(x^p))^(1/p)`, and the inequality HM ≤ GM ≤ AM ≤ QM is the special case for `p = -1, 0, 1, 2` of the more general "power-mean inequality" that says `M_p` is monotonically nondecreasing in `p`. Shipping the whole family together means future additions — `M_3`, `M_4`, `M_∞` (the maximum), `M_{-∞}` (the minimum) — slot into a frame that is already coherent.

## What RMS does that no other shipped L-estimator did

The single most important property of QM among the now-eight shipped lenses is that it is the only lens that grows superlinearly in the largest value. For mean, trimmed mean, midhinge, trimean, median, HM, and GM, doubling the largest value in the dataset produces a sublinear or linear change in the estimator. For QM it produces a strictly superlinear change because of the squaring step. This makes QM the most outlier-sensitive of the eight.

That is not a bug; it is a feature paired with the others. The eight lenses now span the full sensitivity spectrum:

- median: insensitive to outliers, sensitive only to ranks
- midhinge, trimean: lightly sensitive to outliers via Q1/Q3 only
- trimmed mean (trim25): sensitive to the central 50% only
- mean (AM): linearly sensitive to all values
- HM, GM: sensitive in a way that depends on the distribution's shape (HM amplifies small values; GM is balanced in log-space)
- QM: superlinearly sensitive to large values

Any analyst reading the eight numbers side-by-side for the same column gets, for free, a portrait of the column's distributional shape that no single estimator could provide. If AM and median agree, the distribution is roughly symmetric. If AM exceeds median, right skew. If QM dramatically exceeds AM, heavy right tail. If HM is far below GM, heavy left tail (or many small values). The eight-number portrait is, in a meaningful sense, a poor person's histogram — and pew-insights now ships it on every numeric column.

## The release-train mechanics

The SHAs cited for this release pass — `9b43ab4..045a592` — span a small number of commits. The `045a592` head adds the QM implementation, the smoke test that produced the three numbers above, the sandwich-display logic for the dashboard, and a documentation paragraph noting the power-mean family closure. The intermediate SHAs are typing-only fixes and one test-fixture update for the existing GM tests where a precision tolerance needed loosening from `1e-9` to `1e-8` because the QM addition shifted some shared rounding intermediates.

That is a remarkably small change for what is, conceptually, the closing of an entire family. It is small precisely because the prior eight releases did the structural work: set up the L-estimator dispatch table, the order-statistic computation pipeline, the per-source rendering layout, the sandwich-display infrastructure for HM-GM-AM (which now generalizes to four entries trivially), and the test scaffolding that lets each new lens land with `+60 tests` in the boundary case and rather fewer in the additive case like this one.

## Where the arc goes next

The obvious open questions, with QM in:

1. Will pew-insights ship `M_∞` (max) and `M_{-∞}` (min) as the asymptotic endpoints of the power-mean family? Both are trivial to compute and would let the dashboard show the full bracket `min ≤ HM ≤ GM ≤ AM ≤ QM ≤ max` for every column. The line count is negligible; the question is whether `min` and `max` count as "L-estimators" in the family the project wants to brand around.

2. Will the QM/AM ratio get its own column? It is the cleanest one-number heavy-tail summary the tool could ship, and the data is already computed.

3. Will the four power means get a "power-mean inequality verifier" assertion in the test suite? Given how often release smoke output is the first place a numerical bug shows up, an assertion that `HM ≤ GM ≤ AM ≤ QM` holds for every source on every release smoke run would catch a category of regression that no individual unit test would.

For the moment, though, the arc is closed. Eight lenses, ranging from rank-only (median) to L2-amplified (QM), shipping inside a single dashboard column with a single axis and consistent units. The 21.04M number for claude-code is, in its small way, the punctuation mark on what has been an unusually focused four-week run on a single feature surface.
