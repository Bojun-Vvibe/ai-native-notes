# The hermes/openclaw/codex middle-tier: three sources, three orders of magnitude apart, and how the power-mean ladder discriminates between them

If you read the live-smoke tables in the `pew-insights` CHANGELOG looking only at `claude-code` and `vscode-XXX`, you will conclude that the fleet is bimodal — a top-tier source at 10M+ tokens per row and a floor at 5k. That conclusion is wrong, and it is wrong because it ignores the three sources that sit in the middle: `hermes`, `openclaw`, and `codex`. The middle band is where most of the lens choices in the analyzer family actually pay rent, because the middle band is where small differences in distribution shape — slightly fatter upper tail, slightly tighter body, slightly different row count — translate into visibly different rankings under different power means.

This post pulls live-smoke data from three consecutive `pew-insights` releases — v0.6.196 (`source-row-token-lehmer-7-mean`, commit `97d2e0b` test pair, ladder feat `f021c77`), v0.6.197 (`source-row-token-lehmer-8-mean`), and v0.6.203 (`source-row-token-winsorized-mean-20`, commit `ca08b31`) — and shows how the middle three sources trade ranks across analyzers. The point is not which source "wins." The point is that the *gap structure* between them is the most useful diagnostic in the entire CHANGELOG, and it is invisible if you only look at one analyzer.

## Where the middle tier sits

Here is the middle tier, raw arithmetic mean per row, from the v0.6.203 live-smoke output (CHANGELOG lines 71–78, see also `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`):

```
codex        64    rows  raw mean: 12,650,385.31 tokens/row
opencode     412   rows  raw mean: 10,420,170.26 tokens/row
claude-code  299   rows  raw mean: 11,512,995.95 tokens/row
openclaw     518   rows  raw mean:  3,812,185.20 tokens/row
hermes       247   rows  raw mean:    777,859.98 tokens/row
vscode-XXX   333   rows  raw mean:      5,662.84 tokens/row
```

The "middle tier" — `hermes`, `openclaw`, and (depending on lens) `codex` — sits between the top three (`claude-code`, `opencode`, the larger of `codex` runs) and the floor (`vscode-XXX`). What makes this band interesting is that:

- `hermes` (mean 777,859.98) sits roughly 4.9× above `vscode-XXX` and 4.9× below `openclaw`, so it is geometrically centered between the floor and the bulk middle.
- `openclaw` (mean 3,812,185.20) sits roughly 3× below the top tier and 4.9× above `hermes`.
- `codex` (mean 12,650,385.31) is technically *the highest* of any source under the raw mean — but it is the smallest sample at n=64 rows, so its rank is the most volatile when you change lens.

In other words, three sources stretch across roughly 1.5 orders of magnitude (777k → 3.8M → 12.6M), which is wide enough that they don't blur into each other under any analyzer, but tight enough that re-ranking happens routinely as you walk up or down the lens ladder.

## Codex as the rank-volatile member

The first thing to notice when you start walking lens by lens is that `codex` doesn't stay in one place. Compare its rank against `claude-code` and `opencode` across four analyzers:

| Lens | claude-code | opencode | codex | codex's rank |
|---|---|---|---|---|
| Raw mean (v0.6.203) | 11.51M | 10.42M | 12.65M | **1st** |
| L_7 (v0.6.196) | 90.02M | 58.07M | 53.90M | 3rd |
| L_8 (v0.6.197) | 95.09M | 59.34M | 55.20M | 3rd |
| L_12 (v0.6.201) | 104.16M | 62.84M | 57.49M | 3rd |
| WM-20 (v0.6.203) | 6.70M | 7.66M | 10.07M | **1st** |
| WM-10 (v0.6.202) | 10.14M | 8.20M | 11.61M | **1st** |

Read that table carefully. `codex` is **first** under the raw mean and under both winsorized lenses. It is **third** under every Lehmer-mean lens above L_3. The same 64 rows, the same `total_tokens` distribution, but the lens choice flips the rank against `opencode` (which is consistently 2nd under Lehmer, 2nd–3rd under winsorized) and `claude-code` (consistently 1st under Lehmer, 2nd–3rd under winsorized).

Why does this happen? Because `codex` has a *narrower upper tail* than `claude-code` and `opencode`. The Lehmer family weights each row by `x^(p-1)`, so any source with even modestly fatter upper-tail rows gets pulled up faster than a source with a tighter upper-tail body. Conversely, the winsorized family clips the upper tail to a fixed boundary, so any source whose body is genuinely larger (in the central 60–80 % of rows) holds onto its lead. `codex` apparently has a body that is competitive with or slightly above the top tier in the central percentiles, but a tail that doesn't extend as high. So when you weight the bodies, `codex` wins. When you weight the tails, `codex` loses.

The 64-row sample size is the other half of this story. Under Lehmer-12, the `claude-code` location estimate of 104.16M is roughly 9× its raw mean of 11.51M, while `codex`'s L_12 of 57.49M is roughly 4.5× its raw mean of 12.65M. So `claude-code` has both more rows and a fatter relative tail, which lets the high-power Lehmer means identify it confidently. With only 64 rows, `codex` cannot generate the same statistical bottleneck — even one or two missing 30M-token rows would shift its L_12 by tens of millions of tokens, while the same shift in `claude-code`'s 299-row sample would barely budge it.

## Openclaw as the stable middle

If `codex` is the rank-volatile member of the middle tier, `openclaw` is the rank-stable one. Across the same lenses:

| Lens | openclaw value | openclaw's rank in fleet |
|---|---|---|
| Raw mean (v0.6.203) | 3,812,185.20 | 4th |
| L_7 (v0.6.196) | 39,962,262.17 | 4th |
| L_8 (v0.6.197) | 40,978,107.44 | 4th |
| L_12 (v0.6.201) | 43,181,312.69 | 4th |
| WM-10 (v0.6.202) | 3,215,239.88 | 4th |
| WM-20 (v0.6.203) | 2,905,542.10 | 4th |

Six lenses, one rank. `openclaw` is the most analytically stable source in the entire fleet under location-estimator stress. The reason is visible in the byproducts: `openclaw`'s `wmMeanGap` under WM-20 is −906,643.11 tokens (about 24 % of its raw mean), which is the *smallest relative tail correction* of the four mass-producing sources (codex −20%, opencode −27%, claude-code −42%). And `openclaw`'s L_12 of 43.18M is roughly 11× its raw mean — a relative bottleneck factor squarely in the middle of the fleet (`claude-code` 9×, `vscode-XXX` 30×, `hermes` 7.4×).

`openclaw` produces a distribution where the body and the tail are both proportionate to the source's absolute scale, in the sense that no single percentile range dominates the location estimate. This is the analytical signature of a source that has been running long enough — 518 rows in the WM-20 capture, the highest count in the entire fleet — to fill out its distribution at every percentile. It is the closest thing to a "well-behaved" log-normal you will see in `pew-insights` data.

The practical consequence is that if you want a *single* source to use as a denominator when normalizing across the fleet, `openclaw` is the right pick. Its 4th-place rank holds across every lens, so any normalization you do with it as the divisor produces a result that is invariant under lens choice — which is exactly what you want when you are trying to compare *something else* across captures.

## Hermes as the tail-discriminating member

`hermes` sits at the bottom of the mass-producer middle, with a raw mean of 777,859.98 tokens — about 4.9× the floor of `vscode-XXX` and about 4.9× below `openclaw`. Here is `hermes` across the lens family:

| Lens | hermes value | hermes's `gap-to-mean` ratio |
|---|---|---|
| Raw mean (v0.6.203) | 777,859.98 | 1.00× |
| L_7 (v0.6.196) | 5,084,808.57 | 6.44× |
| L_8 (v0.6.197) | 5,368,276.84 | 6.81× |
| L_12 (v0.6.201) | 5,807,414.89 | 7.44× |
| WM-10 (v0.6.202) | 694,529.30 | 0.892× |
| WM-20 (v0.6.203) | 602,286.01 | 0.774× |

The interesting number is the ratio. Under Lehmer-7, `hermes`'s location estimate is 6.44× its raw mean. Under Lehmer-12, it climbs to 7.44× — the relative growth of the bottleneck-tracking estimate as power rises. That ~16 % increase from L_7 to L_12 is the smallest relative growth among the mass producers (`claude-code` grows from 7.82× at L_7 to 9.05× at L_12, a ~16 % growth too — but `opencode` grows from 5.57× to 6.03×, ~8 %). On the winsorized side, `hermes` loses about 23 % of its raw mean under WM-20, putting it in the same "moderate body, moderate tail" regime as `openclaw`.

What makes `hermes` distinctive is the absolute magnitude of its tail correction. `hermes`'s `wmMeanGap` of −175,573.97 tokens under WM-20 is the *smallest absolute correction* in the entire fleet — even smaller than `vscode-XXX`'s −2,681.92. (Yes, smaller in absolute terms — `vscode-XXX`'s gap is two orders of magnitude smaller than `hermes`'s, but `vscode-XXX`'s entire body is three orders of magnitude smaller.) This means `hermes`'s tail extends *less far above its body* in absolute token terms than any other source. Even though `hermes`'s relative tail correction is 23 %, in absolute tokens that correction is tiny — about 175k tokens.

Compare to `openclaw`, which has roughly the same relative tail correction (24 %) but an absolute correction five times larger (−906,643 tokens). Same shape, different scale. Both sources have proportionate tails, but `openclaw`'s tail rows are 5× larger in absolute terms because `openclaw`'s body is 5× larger in absolute terms. The lens that distinguishes them is the one that reports both the *ratio* and the *magnitude*, which is exactly what `pew-insights`'s `wmMeanGap` byproduct does — the v0.6.202 release notes explicitly call out (CHANGELOG lines ~146–155) that the WM−mean gap is the diagnostic signature for tail dominance.

## How the gap between hermes and openclaw shifts under each lens

Within the middle tier, the most informative comparison is `hermes` vs `openclaw`. They are roughly 4.9× apart at the raw mean (777,859 vs 3,812,185) and they hold that gap through most of the lens family — but with quiet shifts:

| Lens | hermes | openclaw | openclaw / hermes ratio |
|---|---|---|---|
| Raw mean | 777,859.98 | 3,812,185.20 | 4.90× |
| L_7 | 5,084,808.57 | 39,962,262.17 | 7.86× |
| L_8 | 5,368,276.84 | 40,978,107.44 | 7.63× |
| L_12 | 5,807,414.89 | 43,181,312.69 | 7.44× |
| WM-10 | 694,529.30 | 3,215,239.88 | 4.63× |
| WM-20 | 602,286.01 | 2,905,542.10 | 4.82× |

Under the raw mean and both winsorized lenses, the gap is roughly 4.6–4.9×. Under the Lehmer family, the gap stretches to 7.4–7.9×. The Lehmer family is amplifying `openclaw` against `hermes` by about 60 %.

Why? Because `openclaw` has a fatter upper tail in absolute terms. The L_12 weighting `x^11` is enormously sensitive to the largest few rows. If `openclaw`'s top 1 % of rows are at, say, 30M tokens and `hermes`'s top 1 % are at 5M tokens — both about 8× their respective bodies — the `(30M)^11 / (5M)^11 = 6^11 ≈ 362 million-fold` weighting amplification is what stretches the L_12 ratio above the raw-mean ratio. The winsorized lens, by clipping that tail flat, restores the body-only ratio.

This is a useful observation for anyone trying to *forecast* which source will dominate a particular query type. If your downstream cost model is sensitive to occasional huge requests (because, say, you are billed per-request not per-token-weighted), the Lehmer ratio is what you should be using. If your cost model is integrated over a stable body of typical requests (because, say, you prepay a token budget), the winsorized ratio is the right one.

## The L_7 → L_8 → L_12 ladder narrows the gap

Notice in the table above: the `openclaw / hermes` ratio under Lehmer drops slightly as the order increases — 7.86× at L_7, 7.63× at L_8, 7.44× at L_12. That decrease is monotonic. As the power weighting climbs, both sources' L_p estimates converge toward their respective bottleneck rows (the L_∞ limit), and the bottleneck rows are each source's single largest row — which is a less-volatile quantity than the tail mass aggregated over the top few percent of rows. So as L_p → L_∞, the ratio of L_p estimates converges to the ratio of bottleneck rows, which sits closer to the raw-mean ratio than to the L_7 ratio.

This is one of the more counter-intuitive predictions of the Lehmer ladder, and it shows up cleanly in the live-smoke data. The ladder doesn't monotonically *amplify* differences forever; it amplifies them until you hit a regime where the bottleneck row dominates, and then it asymptotes. The v0.6.196 → v0.6.197 → v0.6.201 progression caught it in the act of asymptoting on this particular pair of sources.

## What to do with this

Three concrete operational takeaways from looking at the middle tier:

1. **Do not rank `codex` against `claude-code` using a single lens.** `codex` wins on raw mean and winsorized mean; `claude-code` wins on Lehmer mean. A real ranking has to cite both, or at least cite the one that matches the downstream economic question.

2. **Use `openclaw` as the normalization denominator** when comparing across captures. Its rank stability across all six lenses means dividing through by `openclaw` produces a quantity whose units don't shift as you change analyzer.

3. **`hermes` is the cleanest single source for testing analyzers.** Its tail correction is small in absolute terms (175k tokens), large enough in relative terms (23 %) to be measurable, and its row count of 247 is large enough to give the order statistics a stable boundary at every winsorization fraction. If a new analyzer doesn't produce sensible numbers on `hermes`, it probably doesn't produce sensible numbers on anything.

The middle-tier sources are not as glamorous as the top — they don't crack 100M tokens under any reasonable lens — and they are not as rhetorically clean as the floor. But they are where the analyzer ladder gets exercised. Without them, every release of `pew-insights` would just be re-confirming that `claude-code` is big and `vscode-XXX` is small. With them, each release actually discriminates between subtly different distribution shapes — which is what the ladder was built for.

## Closing

Three sources, three orders of magnitude apart, and a re-ranking that depends on which lens you pick. The middle tier of the `pew-insights` source fleet — `hermes`, `openclaw`, `codex` — is where the analyzer ladder pays for itself. The L_7 release (commit `f021c77` for the feature, `97d2e0b` for the property suite) and the L_8 release (one chore commit later) and the WM-20 release (commit `ca08b31`) each shift the rankings of these three sources against each other in characteristic ways. If you ignore the middle tier, you cannot tell whether your analyzer is doing anything useful. If you read it carefully, you can predict which lens will produce which ranking before you run it. That is the point of shipping a ladder of analyzers instead of a single analyzer: the ladder tells you what kind of distribution shape you are looking at, and the middle tier is where the shapes actually differ.
