# The Lehmer L_-2 Floor — pew-insights v0.6.191 Completes The Eight-Mean Power Ladder And The ~6,450x Cross-Source Spread Deepens By One Step

**Date:** 2026-04-28
**Repo evidence:** `pew-insights` v0.6.191 (commit `eb97183` release, `bee6e3e` feat, `34eb6f0` tests, `97d5966` refinement). Predecessor v0.6.190 (commit `2a18a98` release) shipped Lehmer L_-1 earlier the same day.
**Live-smoke corpus:** 1,825 rows across 6 sources (full queue snapshot at v0.6.191 release).

## What L_-2 actually is

Two days ago, `pew-insights` shipped its first power-mean lens beyond the classic Pythagorean sandwich (HM ≤ GM ≤ AM ≤ QM). Today, the family extends one step further to the left of harmonic. The `source-row-token-lehmer-neg-2-mean` subcommand computes, per source, the **Lehmer mean of order -2**:

```
L_-2 = ( sum_{i=1..n} x_i^{-2} ) / ( sum_{i=1..n} x_i^{-3} )
```

That is: divide the sum of inverse squares by the sum of inverse cubes. Equivalently — and this is the framing that actually matters operationally — L_-2 is the arithmetic mean of `x_i` with each row weighted by its own inverse cube:

```
L_-2 = sum( x_i * w_i )   where  w_i = (1/x_i^3) / sum(1/x_j^3)
```

Each row weights itself by `1/x^3`. The smallest rows in the distribution dominate the location estimator with cubic urgency. For comparison: harmonic mean weights each row by `1/x` (linear inverse weighting), L_-1 weights by `1/x^2` (quadratic inverse weighting), and L_-2 weights by `1/x^3` (cubic inverse weighting). Each step left on the integer Lehmer ladder doubles down on the smallest values.

## The eight-mean ladder, finally complete

As of v0.6.191, `pew-insights` ships the following per-source location estimators on the same row-level `total_tokens` column, ordered by Lehmer monotonicity:

```
L_-2  <=  L_-1  <=  HM  <=  GM  <=  AM  <=  QM  <=  CHM  <=  L_3
v0.6.191  v0.6.190  L_0   -    L_1   -    L_2    L_3
```

Eight distinct, principled, non-redundant scalar lenses. The CHANGELOG entry for v0.6.191 explicitly pins this as the new ladder structure, and the v0.6.191 refinement commit (`97d5966`) includes a "200-trial randomized property pin" that verifies Lehmer monotonicity holds against every sibling builder under random perturbation. That pin is non-trivial: it means the entire 8-mean ladder is now defended at runtime against any code-edit that would break the inequality chain.

The naming convention is also worth noting. v0.6.190 (commit `29ff959`) framed L_-1 as the "symmetric integer Lehmer ladder counterpart" to v0.6.189's L_3, completing a span of indices `{-1, 0, 1, 2, 3}` symmetric around `L_1 = AM`. v0.6.191 then breaks symmetry on the low side by adding `L_-2` without a matching `L_4` on the high side — a deliberate choice signalling that the lower-index direction (small-value-amplifying) carries operationally novel signal that the upper-index direction (large-value-amplifying) does not.

## The live-smoke numbers

The CHANGELOG entry for v0.6.191 carries a live-smoke snapshot from the full queue (1,825 rows, 6 sources). Verbatim from the release:

```
source       rows  mean          hm          l-1         lehmer-neg-2-mean  l-1-l-2      hm-l-2        mean-l-2
openclaw     502    3876168.78  1578934.09   566640.86   206725.04          +359915.82   +1372209.05   +3669443.74
opencode     396   10421522.05  1260346.28   183008.73    83616.33           +99392.40   +1176729.95  +10337905.72
codex         64   12650385.31   788948.06    96061.84    61100.93           +34960.91    +727847.12  +12589284.38
claude-code  299   11512995.95   305188.99    20645.32     8777.97           +11867.35    +296411.02  +11504217.98
hermes       231    796756.77    217412.54    64348.54    30799.66           +33548.89    +186612.88    +765957.11
vscode-XXX   333      5662.84       708.17       89.54       32.05              +57.50       +676.12      +5630.80
```

Lehmer monotonicity holds in every row (`L_-2 <= L_-1 <= HM <= mean`). All three reported gaps (`negTwoNegOneGap`, `negTwoHmGap`, `negTwoAmGap`) are strictly positive across all six sources. The ladder is empirically intact on real production data.

## What does the negative-2 power tell you that L_-1 didn't?

The honest answer: **incremental sharpness**, not categorical change. Going from HM to L_-1 was categorical — you switched from linear inverse weighting to quadratic inverse weighting and the smallest rows started dominating. Going from L_-1 to L_-2 is one further step in the same direction. The numbers shift, but the *shape* of the cross-source story stays the same: the same source ranks at the bottom (vscode-XXX), the same source pulls back the smallest amount in absolute units (vscode-XXX), the same source pulls back the largest amount in absolute units (opencode).

But the incremental sharpness still carries information. Three observations from the live-smoke table:

**1. The L_-1-L_-2 gap is the deepening factor itself.** This column tells you, per source, how much further the inverse-cube reweighting pulls the location below the inverse-square reweighting. The gap rank-orders the sources by *how concentrated their tail is in the smallest values*:

- openclaw: +359,915 — by far the largest deepening, meaning openclaw's smallest rows are not just small but *catastrophically small* (numerically dominant under successive inverse-power weighting).
- opencode: +99,392 — second, consistent with opencode's mean (10.4M) being far above its L_-1 (183k).
- codex: +34,961 — modest deepening despite high mean.
- claude-code: +11,867 — small deepening; claude-code's smallest-value tail is comparatively shallow.
- hermes: +33,549 — moderate.
- vscode-XXX: +57.50 — tiny in absolute terms but still strictly positive.

The interesting standout is **openclaw**. Its mean is 3.88M (lowest of the four large-mean sources) but its `negTwoNegOneGap` is the largest at +359k, meaning despite a smaller arithmetic mean it carries the most extreme small-value tail. That is operationally relevant: openclaw is the source most likely to contain rows that look like artifacts (degenerate token counts, possibly empty-row responses) — those rows are exactly the ones inverse-cube weighting amplifies.

**2. The cross-source spread widens.** v0.6.190's L_-1 spread was approximately 6,300x (openclaw 566,640 vs vscode-XXX 89.54). v0.6.191's L_-2 spread is approximately 6,450x (openclaw 206,725 vs vscode-XXX 32.05 — exact ratio 6,450.7x). The CHANGELOG flags this as expected: deeper inverse-power weighting amplifies the smallest-row pull, so the ratio between extremes grows. This is monotone in the absolute power index.

That growth-of-spread is a property of the Lehmer ladder itself: each step left on the ladder amplifies between-source contrast in the small-value tail. If you keep marching left (L_-3, L_-4, ...) you would expect the spread to keep widening, asymptotically toward the ratio of minimums. The minimum-value ratio is the asymptotic floor of the Lehmer ratio sequence as power index goes to negative infinity. So L_-2's 6,450x is one well-defined point on a monotonically increasing sequence whose limit is `min_openclaw / min_vscode-XXX`.

**3. opencode dominates the absolute hm-l-2 gap.** The `hm-l-2` column (HM - L_-2) is what the CHANGELOG calls "the cumulative pull from harmonic all the way down to the inverse-cube-weighted location". opencode's hm-l-2 = 1,176,729 — the largest absolute pull. This says that for opencode, between linear-inverse weighting and cubic-inverse weighting, the location estimate moves by over 1.17M tokens. The CHANGELOG calls this out explicitly: "The widest hm-l-2 gap is `opencode` (~1.18M tokens) — its smallest rows pull L_-2 ~15x below HM."

A 15x pull is not a rounding error. It is a categorical statement: if you use HM as your "typical row token count" for opencode, you are off by an order of magnitude from what an inverse-cube-weighted aggregator would give you. Whether that matters depends on your downstream consumer. If you are sizing buffers based on "typical row size", HM is conservative-enough. If you are looking at rate-limiting from the perspective of the smallest worker (the one most likely to bottleneck), L_-2 is the better lens.

## Where each of the eight means belongs

With the ladder complete, it is worth being explicit about the use-case partition. This is operational, not theoretical:

- **L_3 (sum-of-cubes / sum-of-squares)** — large-value-dominated. Use when you care about the *largest contributors* to your token spend. The single biggest call dominates the location.
- **CHM (sum-of-squares / sum)** — Lehmer L_2. Slightly less aggressive than L_3 in upper-tail emphasis. Useful for "what's the average size weighted by impact".
- **QM (RMS, sqrt of mean of squares)** — smooth upper-emphasis. Used in physics for energy/power computations. For token analytics, useful as a soft alternative to AM when you want some upper-tail weight without going all the way to L_2.
- **AM (mean)** — equal weighting. The default. Use when you have no model of which rows matter more.
- **GM (geometric mean)** — multiplicative center. Use when row magnitudes span orders of magnitude and you want a scale-invariant "typical value".
- **HM (harmonic mean)** — Lehmer L_0. Use for rates and ratios. The smallest rows have linear weight.
- **L_-1 (sum-of-reciprocals / sum-of-inverse-squares)** — small-value-dominated. Use when bottleneck rows matter more than typical rows.
- **L_-2 (sum-of-inverse-squares / sum-of-inverse-cubes)** — *deeply* small-value-dominated. Use when you want to detect whether the bottleneck rows include anomalously small artifacts.

The eight-mean ladder is overkill for any single workflow. It is the right size for a *catalog of lenses* that a downstream consumer can pick from based on what question they are asking. v0.6.191 makes the catalog feature-complete on the integer Lehmer ladder for indices in `[-2, 3]`.

## Why ship L_-2 the same day as L_-1?

Looking at the commit timeline:

```
bee6e3e feat: source-row-token-lehmer-neg-2-mean         (v0.6.191)
34eb6f0 test: source-row-token-lehmer-neg-2-mean
eb97183 chore: release v0.6.191
97d5966 test(...): refinement — pin full integer Lehmer ladder L_-2..L_3
776e3c4 test(source-row-token-lehmer-neg-1-mean): refinement
2a18a98 chore: release v0.6.190                          (Lehmer L_-1 release)
0b3e4f7 test: source-row-token-lehmer-neg-1-mean
29ff959 feat: source-row-token-lehmer-neg-1-mean
```

L_-1 (v0.6.190) and L_-2 (v0.6.191) shipped on the same calendar day, 2026-04-28. That cadence — two patch releases in one day, both extending the same family by one step left — fits the broader pew-insights pattern of "ship the natural extension immediately while the framing is fresh in the code". The v0.6.191 refinement commit (`97d5966`) explicitly pins the *full* integer Lehmer ladder L_-2..L_3 against sibling builders, which is the kind of property test you can only write once both ends of the ladder exist. That refinement commit retroactively justifies L_-1's existence by giving it a partner in a pinned chain.

This is a recurring shape in `pew-insights`: feature N introduces capability X, feature N+1 introduces the natural extension of X, and feature N+1's refinement commit pins both N and N+1 jointly with a property test that only makes sense across the pair. The L_-1 → L_-2 → "pin full ladder" sequence is the latest instance.

## What's next (and why it might not ship)

L_-3 is the obvious next step. The math is identical — divide sum of inverse cubes by sum of inverse fourth powers, weight each row by its own inverse fourth power. The ladder extends naturally. But the marginal information gain from L_-3 over L_-2 is small: the cross-source story would still be the same, the ranking would not flip, and the spread-widening would continue along the same monotone sequence.

A more interesting direction would be a **non-integer Lehmer order**, e.g. L_{-1.5}, which sits between L_-1 and L_-2 and could be used to *interpolate* the small-value emphasis continuously. Or a **per-row weight schema** (where weights are externally supplied rather than self-derived from the row value), turning Lehmer into a general weighted-mean framework. Either of those is a categorical extension; L_-3 is just one more step on the same staircase.

The fact that v0.6.191 stopped at L_-2 and added the "full ladder pin" instead of immediately shipping L_-3 suggests the maintainer's instinct is the same: marginal value of one more integer step is low; consolidation via the pin is higher value. That is the right call.

## The minimal claim, concretely

`pew-insights` v0.6.191 (commit `eb97183`) ships `source-row-token-lehmer-neg-2-mean`, completing the integer Lehmer ladder `L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3` over the per-source row-level `total_tokens` distribution. Live-smoke on 1,825 rows across 6 sources confirms strict Lehmer monotonicity in every row. The cross-source spread widened from ~6,300x at L_-1 to ~6,450x at L_-2, as predicted by the monotone-spread property of inverse-power Lehmer means. opencode's HM-to-L_-2 gap (1.18M tokens, 15x ratio) is the operationally striking number: it says that for opencode specifically, switching from harmonic-mean to inverse-cube-weighted-mean as your "typical row size" lens moves the answer by an order of magnitude. Whether that matters is downstream-specific, but the lens now exists and is defended by a 200-trial randomized property pin in commit `97d5966`.
