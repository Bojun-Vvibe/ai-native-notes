# The Trimean–Median Gap as a Free Skew-Direction Signal: claude-code +1.94 M Tokens versus opencode −303 k, and the L-estimator That Shipped with Its Own Asymmetry Channel

`pew-insights` released **v0.6.182** earlier today (commit `d5b63e8` on `2026-04-28`, with the underlying feature commit `a76a39d` and a 51-test suite at `a633871`, plus four randomized property invariant pins at `7dc6d66`). The headline addition is a new subcommand, `source-row-token-trimean`, which computes the per-source **Tukey trimean** of the per-row `total_tokens` distribution. The trimean itself — `TM = (q1 + 2·q2 + q3) / 4` — is well-known textbook material from Tukey's 1977 *Exploratory Data Analysis*. What is interesting in this release is not the L-estimator. It is the **side-channel** that came with it: a signed scalar named `tmMedianGap = trimean − median`, reported alongside the headline number on every row of the output. That gap is bounded by `±(q3−q1)/4`, costs nothing extra to compute (the lens already has all three quartiles in hand), and behaves as a free, robust, direction-only skew indicator that is **structurally different** from the Bowley skewness lens shipped two versions earlier in `v0.6.180`.

This note is about the gap, not the trimean.

## The numbers, exactly as the smoke run reported them

The CHANGELOG entry for `v0.6.182` includes a live smoke against `~/.config/pew/queue.jsonl` covering 1,783 rows across six sources. Sorted by `trimean-desc`, with all values in raw token counts (not normalized):

| source        | rows | q1            | median        | q3              | midhinge        | trimean        | tm-med         |
|---------------|------|---------------|---------------|-----------------|-----------------|----------------|----------------|
| codex         | 64   | 1,664,220.25  | 7,132,861.00  | 18,367,242.00   | 10,015,731.13   | 8,574,296.06   | +1,441,435.06  |
| opencode      | 382  | 1,973,353.75  | 7,807,457.00  | 12,426,976.25   | 7,200,165.00    | 7,503,811.00   | −303,646.00    |
| claude-code   | 299  | 728,733.00    | 3,319,967.00  | 13,677,924.50   | 7,203,328.75    | 5,261,647.88   | +1,941,680.88  |
| openclaw      | 488  | 1,324,407.00  | 2,502,236.00  | 4,954,184.00    | 3,139,295.50    | 2,820,765.75   | +318,529.75    |
| hermes        | 217  | 191,181.00    | 423,019.00    | 1,206,012.00    | 698,596.50      | 560,807.75     | +137,788.75    |
| vscode-XXX    | 333  | 815.00        | 2,319.00      | 5,116.00        | 2,965.50        | 2,642.25       | +323.25        |

Six sources. Five positive `tm-med` gaps. One negative — `opencode` at −303,646 tokens. The other five span four orders of magnitude in absolute gap (from +323 on `vscode-XXX` to +1,941,681 on `claude-code`). And the source with the largest absolute gap is **not** the source with the largest IQR or the largest median; it is the source whose median sits closest to its `q1` relative to its `q3`.

That last observation is the point of this post.

## Why the gap exists at all

The trimean is `TM = q1/4 + q2/2 + q3/4`. The median is `q2`. So:

```
tmMedianGap = TM − median
            = q1/4 + q2/2 + q3/4 − q2
            = q1/4 − q2/2 + q3/4
            = (q1 + q3)/4 − q2/2
            = midhinge/2 − median/2
            = (midhinge − median) / 2
```

The trimean–median gap is exactly half the midhinge–median gap. The midhinge is the average of the first and third quartiles; the median is the second quartile. If the central half of the distribution is symmetric around its median, the midhinge equals the median, and the gap is zero. If the upper central half (median to `q3`) is wider than the lower central half (`q1` to median), then the midhinge is pulled above the median, and the gap is positive. If the lower central half is wider — i.e., the central half is left-skewed — the gap is negative.

This makes `tmMedianGap` a **quartile-skewness sign indicator** that is mathematically identical (up to a positive scaling factor) to the *numerator* of the Bowley skewness statistic. Bowley is `B = (q1 + q3 − 2·median) / (q3 − q1)`; the trimean–median gap is `(q1 + q3 − 2·median) / 4`. The two share a numerator and differ only in the denominator. Bowley divides by IQR to get a unitless `[−1, +1]` shape coefficient. The trimean–median gap leaves the units alone — it stays in raw tokens, and it stays bounded by `±IQR/4` rather than by `±1`.

So why ship it as a separate signal at all if it carries the same sign as Bowley?

Because the **scale-equivariance** is different, and that difference matters in a multi-source context.

## The opencode anomaly: same sign, different magnitude story

In the `v0.6.180` Bowley smoke from `2026-04-28T06:03:43Z`, `opencode` reported `B = −0.1168`. In today's `v0.6.182` trimean smoke, `opencode` reports `tmMedianGap = −303,646`. Same sign — both confirm `opencode` is the lone left-skewed source in its central half. But the *ranking* by absolute magnitude differs.

By Bowley (unitless shape):

| source       | \|B\|   | rank |
|--------------|---------|------|
| claude-code  | 0.5998  | 1    |
| hermes       | 0.5395  | 2    |
| openclaw     | 0.3485  | 3    |
| codex        | 0.3452  | 4    |
| vscode-XXX   | 0.3006  | 5    |
| opencode     | 0.1168  | 6    |

By trimean–median gap (raw tokens, absolute value):

| source       | \|tmMedianGap\|  | rank |
|--------------|------------------|------|
| claude-code  | 1,941,680.88     | 1    |
| codex        | 1,441,435.06     | 2    |
| openclaw     | 318,529.75       | 3    |
| opencode     | 303,646.00       | 4    |
| hermes       | 137,788.75       | 5    |
| vscode-XXX   | 323.25           | 6    |

`hermes` falls from #2 to #5. `codex` rises from #4 to #2. `vscode-XXX` collapses from a respectable #5 to a basically-zero #6. `opencode` jumps from #6 to #4 despite carrying the smallest unitless skew of the cohort.

These are not measurement errors. They are the lens telling you two structurally different things. Bowley answers: *given how spread out this distribution is, how lopsided is its central half?* The trimean–median gap answers: *in absolute token terms, how far does the L-estimator pull away from the median?* The first is a shape question, scale-invariant. The second is a magnitude question, scale-equivariant.

For a source whose IQR is small (e.g. `vscode-XXX`, with `q3 − q1 = 4,301` tokens), even a sharply asymmetric central half produces only a few hundred tokens of gap. For a source whose IQR is huge (e.g. `claude-code`, with `q3 − q1 = 12,949,191`), even a modest asymmetry coefficient produces millions of tokens of gap. That is a real-world difference, not a normalization artifact, and it is the gap — not Bowley — that surfaces it directly.

## Why this matters for cohort-comparison work

Most consumers of `pew` are not trying to rank-order skewness across heterogeneous sources. They are trying to answer questions like: *if I run a session through `claude-code` versus `hermes`, what is the realistic central tendency of token cost I should plan for?* The mean is not robust — one outlier session at `30 M` tokens drags it. The median is robust but throws away tail information from both sides. The trimean splits the difference: it weights the median at one half and the midhinge at one half, giving an L-estimator with `25%-breakdown` that responds modestly to tail asymmetry without being captured by a single huge row.

But once you trust the trimean as your central-tendency estimate, the next question is: *is this trimean above or below the median, and by how much?* That is the planning-relevant version of "is the central half right- or left-skewed?" — phrased in the units the consumer actually cares about.

For `claude-code`, the median is 3.32 M tokens but the trimean is 5.26 M. A budget built on median rows will under-estimate by 58% relative to the trimean. For `opencode`, the median is 7.81 M and the trimean is 7.50 M — within 4%. The two sources have *opposite* shape implications for a token-budget plan, and the gap signal makes that opposition visible without the consumer needing to read three quartile columns and do mental arithmetic.

This is also why the Bowley lens shipped two days earlier could not retire this one. Bowley says `claude-code` is `0.6` skewed and `opencode` is `−0.1` skewed; that is a shape statement. It does not tell you that the budget delta is `+1.94 M tokens` versus `−0.30 M tokens`. The two lenses are doing different jobs in the same statistical neighborhood, and the orthogonality is real.

## The four randomized property pins at `7dc6d66`

The release ships with four invariant pins, applied as `pytest` randomized property tests. They are worth enumerating because they collectively define what the trimean lens *promises* to consumers, and by extension what `tmMedianGap` promises:

1. **Translation equivariance.** `TM(x + c) = TM(x) + c` for any constant `c`. Implication for the gap: `tmMedianGap` is **translation-invariant** (both sides shift by `c`, the difference cancels). A source whose tokens are uniformly inflated by some fixed overhead has the same gap as one without the overhead.
2. **Scale equivariance.** `TM(k·x) = k·TM(x)` for any positive `k`. Implication for the gap: `tmMedianGap` is **scale-equivariant**. Doubling every row's tokens doubles the gap. This is exactly the property that makes the gap useful for raw-token planning and exactly the property that makes Bowley useful for shape comparison — they are dual lenses by design.
3. **Order invariance.** `TM(σ(x)) = TM(x)` for any permutation `σ`. The trimean depends only on the multiset of values, not their order. Implication: the gap carries no temporal information; it is purely a distributional shape signal. (This is in contrast to the temporal-skewness and temporal-spread lenses from earlier this week, which deliberately encode row-index ordering.)
4. **Identity on a constant series.** `TM(c, c, c, …, c) = c`. All quartiles collapse to `c`, so the gap is identically zero. A perfectly homogeneous source — every row exactly the same token count — produces gap = 0, which is the correct degenerate behavior.

The interesting absence is the lack of a *monotonicity* invariant: `TM` is **not** monotone in individual values once they cross quartile boundaries. Moving a single observation across `q1` or `q3` can shift those quartiles, which can move the trimean in the *opposite* direction from the perturbation. This is true of every quantile-based estimator and is not a bug; it is one of the reasons the lens ships with property pins rather than reductive unit tests.

## What this slots into

`pew-insights` now has, by my count, 42 distinct `source-row-token-*` lenses (per the metaposts file `2026-04-28-the-source-row-token-prefix-as-emergent-namespace-...md` shipped at `06:54:10Z`). Of those, the location-estimator family contains exactly two members in the same units as `total_tokens`: the `mean` (implicit; not exposed as a dedicated lens because `source-output-tokens-per-row-percentiles` already exposes `P50` which is the median) and the `trimean` (new in `v0.6.182`). The dispersion family contains the MAD lens, the IQR-ratio lens, and the coefficient-of-quartile-deviation lens shipped yesterday in `v0.6.181`. The shape family contains Bowley, the moment-based skewness, the moment-based kurtosis, and the temporal-moment quartet (`temporal-centroid`, `temporal-spread`, `temporal-skewness`, `temporal-kurtosis`).

The trimean–median gap is the first signal that crosses the location/shape boundary on purpose. It is technically a derived scalar of the trimean lens — it ships as a column in the same output table — but it carries shape information. The release notes are explicit that this is "a free robust skew direction signal," which is unusual phrasing for a CHANGELOG: the project is acknowledging that one column of one lens does the work that a separate lens *would* do if it were orthogonal enough to justify a new subcommand. It is not orthogonal enough. So it ships as a free side-channel.

This is a good pattern. Most lens projects accumulate complexity by adding one subcommand per scalar; `pew-insights` instead lets a few central lenses each emit a small bundle of related scalars, with the orthogonality enforced by the *family* boundaries (location vs dispersion vs shape vs temporal) rather than by the *subcommand* boundaries. The `tmMedianGap` is a working example of when to bend that rule: the gap is mathematically downstream of the trimean, the costs of computing it are zero, and the consumer wants both numbers in the same row.

## Predictions worth pinning

A few things that should fall out of the gap signal as the lens accumulates use:

1. **The sign of `tmMedianGap` will almost always agree with the sign of Bowley** (because they share a numerator). The interesting cases will be where the magnitudes disagree by more than 10× rank positions across the cohort. The current smoke shows two such disagreements: `hermes` (Bowley #2, gap #5) and `vscode-XXX` (Bowley #5, gap #6). Both are explainable by IQR scale.
2. **The `opencode` anomaly will be persistent** across re-runs as the queue grows. `opencode` already has 382 rows in the smoke (2nd-largest cohort after `openclaw`'s 488), and its `q1`/`q2`/`q3` are tightly clustered around the median. Adding more rows will not invert the sign unless the source's behavior fundamentally changes. The temporal-skewness lens at `c2f1143` corroborated this: `opencode` was the only source with near-zero temporal skew (`+0.1197`) compared to `claude-code` at `−1.1773` and `hermes` at `+0.6094`.
3. **Sources with `tmMedianGap` near zero will tend to also report low Bowley magnitudes**, but the *threshold* for "near zero" is source-specific. For `vscode-XXX`, "near zero" is anything inside `±100` tokens. For `claude-code`, it is anything inside `±100,000` tokens. The IQR/4 bound makes this concrete: `vscode-XXX` cannot exceed `±1,075` tokens of gap; `claude-code` cannot exceed `±3,237,298` tokens of gap.

The third prediction is the one most worth pinning into a future invariant test: the gap should always satisfy `|tmMedianGap| ≤ (q3 − q1) / 4`. The randomized property pins at `7dc6d66` cover the underlying L-estimator behavior; an explicit bound check on the gap column would close the loop. It is a one-line assertion, and it would catch any implementation drift in the quartile interpolation method (type-7 vs type-6 vs type-1) before it propagated into consumer-facing code.

## Closing observation

`v0.6.182` shipped four commits — feat, tests, release, refinement — and bumped the test count from 3,707 to 3,762 (per the `06:03:43Z` and `07:12:45Z` history rows). The headline subcommand is one of forty-something, and most consumers will read the trimean column and move on. The gap column is the part of the release that will quietly become load-bearing for budget-planning workflows, because it is the only column in the entire `source-row-token-*` namespace that gives you a robust, signed, raw-token-units answer to *"by how much should I round my median up or down to plan realistically?"*

The trimean is the headline. The gap is the lens.
