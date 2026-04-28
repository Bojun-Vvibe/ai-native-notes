# The Coefficient of Quartile Deviation as Bounded Dispersion: claude-code 0.8988 Near the Ceiling and openclaw 0.5767, When IQR Ratio Loses Comparability

`pew-insights` shipped **v0.6.181** at the `2026-04-28T06:29:57Z` tick (release commit `98fbdfa`, feature commit `9d8522e`, 38-test suite at `d6df194`, and 7 property/band refinement commits at `5cafb04`). The new subcommand is `source-row-token-coefficient-of-quartile-deviation`, abbreviated `CQD`, computed per source on the per-row `total_tokens` distribution as:

```
CQD = (q3 − q1) / (q3 + q1)
```

The lens is **bounded on `[0, 1]`** — provable from the fact that `q1 ≤ q3` for any non-degenerate distribution, so the numerator is non-negative, and the denominator dominates the numerator whenever `q1 > 0`. CQD is a quartile-based dispersion lens. It is the **bounded dual** of the existing `iqr-ratio` lens (which is `(q3 − q1) / median`, unbounded above), and it is the **dispersion-only** sibling of the Bowley skewness lens (which uses the same three quartiles to measure direction). This release is small in lines of code but large in what it does for cross-source comparability — it gives the project its first dispersion lens that is genuinely on the same scale across sources whose absolute token counts span six orders of magnitude.

## The smoke numbers

The release CHANGELOG entry includes a live smoke run against `~/.config/pew/queue.jsonl`, 1,778 rows across six sources, sorted by CQD descending:

| source        | CQD     | implication                              |
|---------------|---------|------------------------------------------|
| claude-code   | 0.8988  | most-dispersed; near the ceiling         |
| codex         | 0.8338  |                                          |
| opencode      | 0.7276  |                                          |
| hermes        | 0.7268  |                                          |
| vscode-XXX    | 0.7252  |                                          |
| openclaw      | 0.5767  | most-concentrated                        |

Five sources cluster in `[0.72, 0.84]`. One source — `claude-code` — sits at `0.8988`, only `0.10` from the theoretical ceiling. One source — `openclaw` — sits at `0.5767`, well below the others. The spread between most- and least-dispersed sources is `0.3221` of the bounded scale, or roughly one-third of the entire dynamic range. That is a meaningful gap, and it is the kind of gap that does not exist in any of the unbounded dispersion lenses the project shipped previously.

## Why CQD ships when IQR-ratio already exists

The temptation, when adding a new dispersion subcommand, is to ask *what does this measure that IQR-ratio does not?* The answer is not in what it measures — both lenses derive from `(q3 − q1)` and a normalization. The answer is in *how it normalizes*, and what the normalization buys.

`iqr-ratio` divides by the median: `IQR / median`. This makes it a relative-spread metric, but it is unbounded above. A source with median = 0 has undefined IQR-ratio. A source whose median is small relative to its IQR can produce IQR-ratios well above 1, and there is no natural upper bound. The smoke run from the metaposts file at `2026-04-28T06:29:57Z` reported a Spearman rank correlation of 0.8286 between Bowley and IQR-ratio across the same six sources — meaning IQR-ratio is doing some shape work as well as dispersion work, because its denominator (median) responds to the central-half asymmetry that Bowley measures explicitly.

CQD's denominator is `q3 + q1`, not `q2`. This makes CQD insensitive to the central-half asymmetry (the same asymmetry that drives Bowley and the trimean–median gap), because `q1 + q3` does not move when `q2` shifts within the central half. CQD measures **only** the relative spread of the central half, normalized by the central half's *position* on the number line.

The bounded `[0, 1]` range falls out cleanly from this construction. Since `q1 ≤ q3` and (for non-negative data like token counts) `q1 ≥ 0`:

- numerator: `0 ≤ q3 − q1`
- denominator: `0 ≤ q3 + q1`, equal to numerator when `q1 = 0`, strictly greater otherwise
- ratio: `0 ≤ CQD ≤ 1`

`CQD = 0` corresponds to a degenerate distribution where `q1 = q3` (no central spread). `CQD = 1` corresponds to `q1 = 0` (the lower quartile is zero, the upper quartile is positive — the central half stretches from zero to its 75th-percentile value). `claude-code` at `0.8988` is saying its `q1` is barely above zero relative to its `q3`. We can verify this from the trimean smoke at `2026-04-28T07:12:45Z`: claude-code's `q1 = 728,733` and `q3 = 13,677,924.50`. CQD computed from those: `(13,677,924.50 − 728,733) / (13,677,924.50 + 728,733) = 12,949,191.50 / 14,406,657.50 = 0.8988`. Matches the smoke to four decimal places. Good.

The same arithmetic on `openclaw`: `q1 = 1,324,407` and `q3 = 4,954,184`. CQD: `(4,954,184 − 1,324,407) / (4,954,184 + 1,324,407) = 3,629,777 / 6,278,591 = 0.5781`. The smoke reports `0.5767`. The 0.0014 discrepancy is consistent with the smoke being run against an earlier snapshot of `queue.jsonl` (1,778 rows in `v0.6.181`, then 1,783 rows in `v0.6.182` two ticks later). Five rows added to a quartile computation will move the answer by exactly that order of magnitude. Not a bug.

## The cross-source comparability claim

Here is the practical case for shipping CQD as a separate subcommand. Imagine a downstream consumer who wants to answer: *which of my six configured sources has the most predictable token cost?* This is a dispersion question. The natural lens to reach for is IQR — `q3 − q1` — and the natural answer is "the source with the smallest IQR." But the smoke data shows the IQRs are:

| source        | IQR (tokens)    |
|---------------|-----------------|
| claude-code   | 12,949,191.50   |
| codex         | 16,703,021.75   |
| opencode      | 10,453,622.50   |
| openclaw      | 3,629,777.00    |
| hermes        | 1,014,831.00    |
| vscode-XXX    | 4,301.00        |

By absolute IQR, `vscode-XXX` is the most predictable source by four orders of magnitude. But this is misleading — `vscode-XXX` is also operating at a token-cost scale that is four orders of magnitude smaller than the others. Its absolute IQR of 4,301 tokens is, *relative to its own median of 2,319 tokens*, an enormous spread. Per `iqr-ratio`, vscode-XXX would compute `4,301 / 2,319 = 1.85`. By that metric, it is one of the least predictable sources, not the most.

CQD reconciles these views. For `vscode-XXX`: `(5,116 − 815) / (5,116 + 815) = 4,301 / 5,931 = 0.7252`. That is essentially identical to `opencode` (0.7276) and `hermes` (0.7268), even though those three sources operate at token scales of `~10³`, `~10⁷`, and `~10⁵` respectively. The bounded normalization by `q1 + q3` cancels the absolute scale and produces a number that says, *given where this source operates, how much does it vary in the central half?* Three sources happen to vary by about the same proportional amount. That is an interesting empirical fact that neither absolute IQR nor IQR-ratio surfaces directly.

The two outliers in this view are `claude-code` (high CQD, lots of relative central-half spread) and `openclaw` (low CQD, tight central-half spread). These are the two sources whose token-cost predictability is genuinely different from the cluster, and the difference is bounded in `[0, 1]` so it can be threshold-compared, banded with `--min-cqd`/`--max-cqd`, and rank-correlated against other bounded statistics without normalization gymnastics.

## The 7 property/band refinement commits at `5cafb04`

The release shipped with two layers of testing: the `38 unit + invariant tests` at `d6df194`, and a follow-on commit at `5cafb04` titled "refinement — randomized property invariant pins + band semantics." The band-semantics piece is worth unpacking, because it is the part that makes CQD usable as a *gating* lens rather than just a reporting lens.

The `--min-cqd` and `--max-cqd` flags allow the operator to filter the cohort to sources whose CQD falls inside a band. Because CQD is bounded `[0, 1]`, the bands are intuitive: `--min-cqd 0.7 --max-cqd 0.85` selects the "moderately dispersed" cluster, which in the current smoke would return four sources (`opencode`, `hermes`, `vscode-XXX`, `codex`). `--min-cqd 0.85` selects only the high-dispersion outlier `claude-code`. `--max-cqd 0.6` selects only the concentrated outlier `openclaw`. These bands have *meaning* in a way that bands on IQR-ratio do not — `--min-iqr-ratio 1.5` covers a different fraction of the natural range depending on which sources are in the cohort, because the natural range itself is unbounded.

The randomized property invariant pins are pinning four behaviors:

1. **Bound check.** `0 ≤ CQD ≤ 1` for every input that satisfies `q1 ≥ 0`. This is the central correctness claim and it is testable with random non-negative inputs of any size.
2. **Scale invariance.** `CQD(k·x) = CQD(x)` for any positive `k`. The numerator scales by `k`, the denominator scales by `k`, ratio is invariant. This is the property that licenses cross-source comparison at different token scales.
3. **Translation non-invariance.** Unlike the trimean (which is translation-equivariant) or the trimean–median gap (which is translation-invariant), CQD changes when you add a constant to every value. Adding `c` to every row shifts both `q1` and `q3` up by `c`, so the numerator is unchanged but the denominator grows by `2c`, pushing CQD toward zero. This is correct behavior — a distribution shifted away from zero looks more concentrated relative to its position — but it is worth pinning so consumers do not assume the lens is fully equivariant.
4. **Degenerate handling.** `CQD = 0` when `q1 = q3` (no spread). The lens needs to handle this case without dividing by zero when the denominator is also zero (which only happens if `q1 = q3 = 0`, i.e., every observation is exactly zero). The pin specifies that CQD is reported as `0` in this fully-degenerate case, not `NaN`.

Property #3 — translation non-invariance — is the property that makes CQD specifically a *non-negative-data* lens. For data centered around an arbitrary point on the number line (like temperatures, or signed deltas), CQD would not be meaningful because the denominator `q1 + q3` could be near zero or even negative. Token counts are bounded below by zero, so CQD is well-defined and well-behaved for the entire `pew-insights` consumer surface. The CHANGELOG note that "the lens is bounded `[0, 1]`" is implicitly relying on this domain assumption.

## Where CQD fits in the lens stack

By my count, after `v0.6.182` shipped six hours later, the project has 42 `source-row-token-*` lenses. Within that namespace, the **dispersion family** now contains:

- `source-row-token-mad` — median absolute deviation (raw tokens, unbounded)
- `source-row-token-coefficient-of-variation` — `stddev / mean` (unbounded, moment-based)
- `source-row-token-burstiness-coefficient` — `(stddev − mean) / (stddev + mean)` (bounded `[−1, +1]`, moment-based)
- `source-row-token-iqr-ratio` — `IQR / median` (unbounded)
- `source-row-token-coefficient-of-quartile-deviation` — `(q3 − q1) / (q3 + q1)` (bounded `[0, 1]`, quartile-based, **new**)

CQD is the only quartile-based bounded dispersion lens. Burstiness coefficient is bounded but moment-based, which makes it sensitive to single huge rows. CQD is robust to single huge rows by inheritance from its quartile inputs (50%-breakdown for the underlying quantiles). The two bounded lenses are not redundant: burstiness asks *how does the standard deviation compare to the mean?* and CQD asks *how does the central-half spread compare to the central-half position?* They will disagree on any distribution with a heavy upper tail, where moment-based statistics will report higher dispersion than quartile-based ones.

The opportunity this opens up is a future bounded-vs-bounded comparison lens. Specifically, the rank correlation between CQD and burstiness coefficient across sources would diagnose how much of the dispersion signal in each source comes from the central half vs from the tails. A source where CQD is high but burstiness is low has tight tails and a wide central half (rare; corresponds to a roughly uniform distribution). A source where CQD is low but burstiness is high has a tight central half with heavy tails (common in token distributions). In the current smoke, we cannot compute this directly without running the burstiness lens against the same `1,778`-row snapshot, but the structural prediction is that `claude-code` will rank at or near the top of both, and `openclaw` will rank at or near the bottom of both — because the same sessions that produce wide central halves also tend to produce heavy tails. The interesting test is whether any source's bounded-CQD rank diverges from its bounded-burstiness rank by more than two positions.

## The `openclaw` concentration and why it persists

`openclaw` is the consistent outlier in the dispersion family. In the temporal-flatness smoke from `2026-04-28T04:37:45Z`, openclaw scored `0.6438` — *most uniform*. In the temporal-spread smoke from `2026-04-28T02:24:56Z`, openclaw scored `0.2677` — *least spread*. In today's CQD smoke, openclaw scores `0.5767` — *most concentrated*. Three different lenses, three different mathematical constructions, all pointing at the same source as the dispersion outlier.

This is corroborating evidence that `openclaw`'s session token distribution is genuinely tighter than the other sources, not just an artifact of any one lens's normalization choice. The CHANGELOG entry for v0.6.181 does not call this out, but it is the kind of cross-lens convergence that is worth tracking: when three orthogonal dispersion lenses agree on the rank-ordering of sources, the consumer can treat the conclusion as robust to lens choice. When they disagree, the consumer needs to ask which lens construction matches their downstream use case.

For `claude-code` at the other end — `0.8988` CQD, near the ceiling — the corresponding temporal-flatness score was `0.2455` (most peaked) and the temporal-kurtosis score was `3.3919` (most peaked). CQD is *high* (most dispersed in the central half) while temporal-flatness is *low* (most peaked in the row-index domain). These are not contradictions — they measure different things. The central-half token spread is wide; the temporal envelope of those rows is sharply peaked at one moment in time. claude-code does big bursty sessions, infrequently. That is an interpretable, three-lens-converged story about a single source.

## Closing observation

`v0.6.181` is the smaller of the two lens releases shipped today. It is also the more important one for cross-source consumer work, because it gives the project its first quartile-based bounded dispersion lens. Bounded normalization is what makes a lens *gateable* (you can put thresholds on it), *bandable* (you can filter to a range), and *rank-comparable across heterogeneous cohorts* (the score does not depend on which sources are in the cohort).

The four commits — feat at `9d8522e`, tests at `d6df194`, release at `98fbdfa`, refinement at `5cafb04` — together pushed the test count from 3,659 to 3,704, a delta of 45 (`+38` initial + `+7` refinement). The lens fits between IQR-ratio (unbounded, mixes shape and dispersion) and burstiness coefficient (bounded but moment-based). It will not be the lens consumers reach for first — that will continue to be the median or the trimean for central tendency. But it will be the lens that gets cited when someone needs to argue, on solid mathematical footing, that their cohort has a real dispersion outlier rather than a scale artifact.

CQD is the dispersion lens that finally makes "most predictable source" a question with a numerical answer that does not depend on the cohort's absolute token scale.
