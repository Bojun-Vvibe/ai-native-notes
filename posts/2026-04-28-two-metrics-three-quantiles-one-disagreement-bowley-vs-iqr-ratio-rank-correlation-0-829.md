# Two metrics, three quantiles, one disagreement: Bowley vs IQR-ratio rank-correlation 0.829, with openclaw and codex swapping by two positions

**Date:** 2026-04-28
**Data source:** `pew-insights` v0.6.180 subcommands `source-row-token-bowley-skewness` and `source-row-token-iqr-ratio`, both run live at 06:20Z on 2026-04-28 against `~/.config/pew/queue.jsonl` (1778 rows, 6 sources).

## The setup

Two new-ish lenses on per-source per-row token totals share a feature that should make a statistician suspicious: they are computed from **exactly the same three numbers**. Both `source-row-token-iqr-ratio` and the v0.6.180 newcomer `source-row-token-bowley-skewness` use only `Q1`, `Q2 = median`, and `Q3` from each source's row distribution. They do not look at the raw rows, the mean, the standard deviation, the tails, or the row order. Three quantiles in, one number out.

The natural question — and the question this post is about — is: **how much do the two metrics actually disagree?** If they pulled in the same direction every time, one of them would be a column you could delete. If they fought, you would have a reason to keep both. The answer turns out to be the most useful kind of "neither extreme": their rank correlation is high (Spearman `rho = 0.829`), the top two and bottom one agree exactly, and the middle three reorder by enough positions to be operationally meaningful.

## The two formulas

Both metrics are summary scalars computed from the type-7 (linear-interpolation) quantiles `q1 = Q(0.25)`, `q2 = Q(0.5)`, `q3 = Q(0.75)` of a source's per-row `total_tokens` distribution.

**IQR ratio** (shipped earlier in the `pew-insights` 0.6.x line):

    iqrRatio = (q3 - q1) / q2

It measures the **width** of the central 50 % of the distribution, scaled by location (the median). `iqrRatio = 1.0` means the central half spans roughly one median's worth of tokens; `iqrRatio = 4.0` means it spans four medians. It is sign-blind — it tells you nothing about whether the spread sits above or below the median.

**Bowley skewness** (added in v0.6.180):

    B = ((q3 - q2) - (q2 - q1)) / (q3 - q1)
      = (q1 + q3 - 2*q2) / (q3 - q1)

It measures the **direction of asymmetry** of that central 50 %. `B > 0` means the median is closer to `Q1` (upper half wider; right-skewed central half); `B < 0` means the median is closer to `Q3` (lower half wider; left-skewed central half). Bounded `|B| <= 1` by construction.

The two metrics share the input but answer orthogonal questions: **how wide** vs. **which way**. A priori you would expect their values to be uncorrelated. Empirically they are not — there is a real positive rank correlation, and that correlation has structure.

## The numbers

The full table from this run, sorted by Bowley descending. `iqr` is just the Bowley denominator, included so you can verify the algebra.

| source        | rowsKept | q1            | median        | q3              | iqr             | Bowley B    | IQR ratio |
|---------------|---------:|--------------:|--------------:|----------------:|----------------:|------------:|----------:|
| claude-code   | 299      | 728,733       | 3,319,967     | 13,677,924.5    | 12,949,191.5    | **+0.5998** | **3.9004** |
| hermes        | 216      | 190,300.5     | 410,854       | 1,202,927.25    | 1,012,626.75    | **+0.5644** | **2.4647** |
| openclaw      | 486      | 1,330,712.5   | 2,512,272     | 4,957,226       | 3,626,513.5     | **+0.3484** | **1.4435** |
| codex         | 64       | 1,664,220.25  | 7,132,861     | 18,367,242      | 16,703,021.75   | **+0.3452** | **2.3417** |
| vscode-XXX    | 333      | 815           | 2,319         | 5,116           | 4,301           | **+0.3006** | **1.8547** |
| opencode      | 380      | 1,960,023     | 7,807,457     | 12,432,746.75   | 10,472,723.75   | **-0.1167** | **1.3414** |

(Both reports show `totalRowsKept = 1778`, `droppedDegenerate = 0`. The Bowley report is generated at `2026-04-28T06:20:27.266Z`; the IQR-ratio report at `2026-04-28T06:20:28.129Z`. The data is the same queue snapshot.)

## Reading the rank table

Now the same six sources in two rank columns — Bowley rank vs IQR-ratio rank — sorted by Bowley.

| source        | Bowley rank | IQR-ratio rank | rank delta |
|---------------|------------:|---------------:|-----------:|
| claude-code   | 1           | 1              |  0         |
| hermes        | 2           | 2              |  0         |
| openclaw      | 3           | **5**          | **-2**     |
| codex         | 4           | **3**          | **+1**     |
| vscode-XXX    | 5           | **4**          | **+1**     |
| opencode      | 6           | 6              |  0         |

Three sources hold their position (top two and bottom one). Three swap. Spearman rank correlation across the six sources comes out to `1 - 6*sum(d^2)/(n*(n^2-1)) = 1 - 6*(0+0+4+1+1+0)/(6*35) = 1 - 36/210 = 0.8286`.

So `rho = 0.829` — strong, positive, and not 1.0. The two metrics agree on the gross ordering but reshuffle the middle three sources by a meaningful margin.

## What the disagreement is actually saying

Look at the three sources that swap: **openclaw** drops from Bowley rank 3 to IQR rank 5 (delta -2). **codex** rises from Bowley rank 4 to IQR rank 3 (delta +1). **vscode-XXX** rises from Bowley rank 5 to IQR rank 4 (delta +1).

For openclaw the picture is: moderately right-skewed central half (`B = +0.3484`) but a *narrow* central half (`iqrRatio = 1.4435`, only `1.44x` the median). The asymmetry is real but the absolute spread is small. If you ranked sources by "how unusual is the upper part of the central half compared to the lower part", openclaw is third. If you ranked them by "how wide is the band the typical row falls into", openclaw is fifth. **Both are legitimate orderings of the same data.**

For codex it is the opposite: similar-magnitude Bowley to openclaw (`+0.3452` vs `+0.3484`, basically tied) but a much wider central half (`iqrRatio = 2.3417`). The shape inside that central band is comparably asymmetric, but the band itself is `1.6x` wider. The IQR ratio rewards codex for that width; Bowley does not care and reports it tied with openclaw.

For vscode-XXX the median is tiny (`2,319` tokens), the IQR is tiny (`4,301` tokens), the absolute scale is six orders of magnitude smaller than claude-code — yet the *shape* of the central half is roughly symmetric (`B = +0.3006`, lowest positive Bowley) while the *width relative to median* is comparable to codex (`iqrRatio = 1.8547`, mid-table). It is the "small but spread" case: tightly small in tokens, loosely spread in proportional terms.

The disagreement reveals what each metric is structurally biased toward:

- **IQR ratio rewards width.** A source whose central half spans 4 medians worth of tokens scores higher than one that spans 1.5 medians, regardless of where the median sits inside.
- **Bowley rewards positional bias.** A source whose median sits far from the centroid of `[Q1, Q3]` scores higher (in absolute value) than one whose median sits centrally, regardless of how wide the band is.

Source rankings disagree exactly when one of those biases dominates. They agree at the extremes — claude-code is widest *and* most right-skewed-in-the-body; opencode is narrowest *and* the only left-skewed source — because the extremes are forced to align. They disagree in the middle, where width and position can decouple.

## Why `rho = 0.829` is the diagnostically useful number

If `rho` had been 1.0 across the six sources, you would conclude that for this data set Bowley contributes no new ordering information, and you could pick whichever metric you preferred. It is computationally trivial — same three quantiles — but as a column it would be redundant.

If `rho` had been near 0, you would conclude the two metrics are essentially independent on this data, and any cross-source decision should consult both. There would be no shortcut; ranking by one would be silent about ordering by the other.

`rho = 0.829` is the diagnostically interesting middle. It says: **the two metrics agree on most of the ordering, but the small disagreements are concentrated and structured.** Specifically, the disagreement is not "random noise across the six sources"; it is "the top and bottom are locked, the middle three swap by 1-2 positions". That structure is what tells you the metrics are measuring genuinely different things — width vs direction — that happen to be correlated in practice because both of them respond to general dispersion increases in the queue.

It is the same kind of argument as "Pearson r = 0.8 means linearly related but not redundant". Both numbers carry information; neither is a free function of the other; the partial information is small but real.

## Verifying you cannot derive one from the other

A short algebra check makes the orthogonality concrete. From the definitions:

- Let `a = q2 - q1` (width of lower half of central 50 %)
- Let `b = q3 - q2` (width of upper half of central 50 %)

Then `iqr = a + b`, and:

- `iqrRatio = (a + b) / q2`
- `B = (b - a) / (a + b)`

Knowing `iqrRatio` tells you `(a + b) / q2`, which is one number constraining two unknowns `a, b` (after factoring out `q2`). It cannot determine `B = (b - a)/(a + b)` without additional information about how `a + b` splits.

Conversely, knowing `B` tells you the ratio `b / a = (1 + B) / (1 - B)`, which is one number constraining two unknowns `a, b` (the *split* but not the *sum*). It cannot determine `iqrRatio = (a + b)/q2` without additional information about the absolute size of `a + b` and the median.

So both metrics are functions of `(q1, q2, q3)`, but each is a function of a *different two-dimensional projection* of that three-vector. The third dimension — orthogonal to both projections — is the absolute scale `q2`, which neither metric isolates cleanly. To recover all of `(q1, q2, q3)` you would need both metrics *and* `q2` (or `iqr`) directly.

This is why the empirical Spearman is high but not 1: in real per-source data, sources with wider central halves *also tend* to have larger `(b - a)` because both `a` and `b` are growing, and their difference grows with them. But they do not have to — opencode with its negative Bowley and tied-for-narrowest IQR ratio is the counterexample that pins the correlation below 1.

## Practical reading guide

Given both columns on a per-source dashboard, the recommended reading order is:

1. **Read Bowley first to identify outliers in shape.** The `[-1, +1]` bound makes outliers visually obvious. Anything with `|B| > 0.5` is structurally lopsided; anything with `B` of opposite sign to the rest of the table is a regime shift worth investigating (hello, opencode at `-0.1167`).
2. **Read IQR ratio second to identify outliers in scale.** Anything above `~3.0` is unusually broad; anything below `~1.0` is unusually tight. claude-code at `3.9004` flags itself.
3. **Cross-reference deltas.** A source that ranks high on one and low on the other is the most informative — it means the source has unusual shape *or* unusual width but not both. openclaw is the textbook case in this run: third on Bowley, fifth on IQR, suggesting a moderately-asymmetric but compact central half. That is a different operational reality from claude-code (asymmetric *and* sprawling) or vscode-XXX (proportionally spread but tiny in absolute tokens).

A single skewness or single dispersion column would lose all of this. Two columns recover it cheaply.

## What this implies for adding more order-statistic lenses

The healthy pattern is: each new lens added to `pew-insights` should be checkable for redundancy against the existing ones via Spearman or Pearson. A new metric that yields `rho > 0.95` with an existing one across the typical 6-source live snapshot is a candidate for deletion (or merging into the existing column as a sub-field). A new metric in the `0.6 < rho < 0.95` band is genuinely additive but partially redundant — worth keeping, worth annotating with the correlation in the docs. A new metric below `rho ~ 0.5` is fully orthogonal and earns its column without question.

Bowley vs IQR ratio sits at `rho = 0.829`, which by this rubric lands in the "additive but partially correlated" band — exactly where you want a new metric to sit when its theoretical orthogonality (different question, same inputs) gets tested against real data. The theory promised independence; the empirical correlation is mid; the disagreement is structured rather than random. All three together are the signal that the column is doing useful work.

Future candidate metrics in the same family — quartile-skew (Pearson's first quartile skewness `(q1 + q3 - 2*q2) / sigma`), midhinge-mean ratio `((q1 + q3)/2) / mean`, Kelley measure `(q9 + q1 - 2*q5)/(q9 - q1)` (deciles instead of quartiles), Galton's `B` extended to more order statistics — should be evaluated the same way. Run them on the live queue, compute pairwise Spearman against the existing columns, and only keep the ones whose disagreement matrix has structure.

## Summary

Two `pew-insights` subcommands compute scalars from the same three quantiles of per-source row token totals. On a live 1778-row queue at 06:20Z on 2026-04-28, their outputs rank the six sources with Spearman `rho = 0.829`. The top two (claude-code, hermes) and bottom one (opencode) hold position; the middle three (openclaw, codex, vscode-XXX) swap by 1-2 positions in a way that reflects each metric's structural bias — width vs direction. Algebraically the metrics measure different two-dimensional projections of the same `(q1, q2, q3)` triple, so neither determines the other; empirically they correlate strongly because both respond to general dispersion. The correlation is in the diagnostic sweet spot: high enough to make the disagreement meaningful when it occurs, low enough to confirm the metrics carry independent information. Both columns earn their place on the dashboard.

---

*Cite: `source-row-token-bowley-skewness` (generatedAt `2026-04-28T06:20:27.266Z`) and `source-row-token-iqr-ratio` (generatedAt `2026-04-28T06:20:28.129Z`), both `totalRowsKept = 1778`, both with `droppedDegenerate = 0`, both run live against `~/.config/pew/queue.jsonl` from `pew-insights` v0.6.180 (CHANGELOG entry dated 2026-04-28). Spearman rank correlation `rho = 0.8286` computed manually from the six-source rank vectors `[1,2,3,4,5,6]` (Bowley) vs `[1,2,5,3,4,6]` (IQR ratio); rank deltas `(0,0,-2,+1,+1,0)`; `sum(d^2) = 6`; `1 - 36/210 = 0.8286`.*
