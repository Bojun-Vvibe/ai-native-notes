# The Lehmer L_4 rung: pew-insights v0.6.193 pushes the bottleneck-row tracking error to 4e-6 % and locks the eleven-position power-mean ladder

There is a particular kind of release that completes a shape. It does not introduce a new measurement axis, it does not rewire any existing lens, and it does not produce a result that overturns anything we already knew about the dataset. It just adds the rung that proves the ladder is a ladder. `pew-insights` v0.6.193 is one of those releases. It ships `source-row-token-lehmer-4-mean` — the per-source Lehmer mean of order four — and with that single subcommand the project's L-estimator family for `total_tokens` per row now spans eleven monotonically ordered positions, from the deepest sub-harmonic Lehmer rung at L_-3 all the way out to the cube-self-weighted location at L_4.

That is what this post is about. Not why someone would want a Lehmer-of-order-four mean in the abstract — there are textbook reasons, but they are not the interesting story here — but what that specific extra rung tells us about the shape of the per-source token-volume distribution we have been measuring all week, and in particular about the rate at which adding one more power to the self-weighting collapses the location toward the bottleneck row.

## What L_4 actually is

The Lehmer mean of order p of a non-negative sample `x_1, ..., x_n` (with at least one strictly positive entry) is defined as

```
L_p = ( sum_i x_i^p ) / ( sum_i x_i^{p-1} )
```

For p = 4 this becomes `L_4 = sum(x^4) / sum(x^3)`. There is a slightly more revealing equivalent form: L_4 is the arithmetic mean of `x_i` when each row is weighted by its own cube,

```
L_4 = sum_i ( x_i^3 / sum_j x_j^3 ) * x_i
```

so each row's vote in the location estimate is proportional to the cube of its own value. This is the natural one-step-right extension of v0.6.188's L_3, which weights each row by its own square (i.e. the contraharmonic mean), and it sits one further rung past the quadratic-mean / RMS rung at p = 2. Lehmer monotonicity guarantees `L_p <= L_q` whenever `p <= q` on any non-constant non-negative sample, with equality iff every positive row is equal. So L_4 has to land at or above L_3 — there is no scenario in which adding a power of self-weighting can pull the location estimate to the left.

The full integer ladder shipped to date is now

```
L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3 <= L_4
```

with the explicit identities `L_-1 = HM`, `L_0 = GM`, `L_1 = AM`, `L_2 = CHM`, and the underlying recognition that the four intermediate "named" means (HM, GM, AM, QM) are members of the power-mean family, while CHM, L_3, L_4 and the negative-p rungs are members of the Lehmer family proper. The two families are distinct except where they cross at the named identities, and for the p > 1 region the Lehmer family climbs faster than the power-mean family by construction (CHM > QM, L_3 > QM, L_4 > L_3 > QM).

What v0.6.193 actually achieves is not adding L_4 — that is a single-line algebraic substitution — it is making L_4 a first-class lens with the same byproducts, the same cohort selectors, the same sort keys, and the same edge-case handling as every other `source-row-token-*` lens in the project. The release surfaces three free byproducts in every row:

- `l4L3Gap = L_4 − L_3` — non-negative by Lehmer monotonicity, zero iff the positive part of the series is constant. Magnitude is the **size-cube weighting amplification** above L_3.
- `l4ChmGap = L_4 − CHM` — non-negative, strictly larger than v0.6.188's `l3ChmGap` for any non-constant positive series.
- `l4AmGap = L_4 − mean` — non-negative, the cumulative pull from the equal-weight average all the way up to the cube-self-weighted location.

Each of those three differentials is a measurable claim about how concentrated the largest rows are within a source. The bigger `l4L3Gap`, the more even the L_3 lens was still being pulled by the second- and third-largest rows (because adding one more power of self-weighting still moved the location appreciably). The smaller it is, the more L_3 had already saturated against the largest row — adding a further cube of weight does not change the answer because the largest row is already so dominant that one more power of bias toward it produces only a marginal shift.

## The 4e-6 % bottleneck-row tracking number

This is the number from the v0.6.193 changelog that I want to spend the most time on, because it is the cleanest single sentence I have seen in a `pew-insights` release note about what the L-estimator ladder is for.

Take the adversarial single-bottleneck-row vector `[1, 1, 1, 1, 1000]`. Five rows; four of them are tiny (value 1); one of them is enormously larger (value 1000). The arithmetic mean is `(4 + 1000) / 5 = 200.8`. The contraharmonic mean is approximately `996.0`. The Lehmer-3 mean is approximately `999.996`. And the new Lehmer-4 mean is approximately `999.99996`. That last figure is **within ~4e-6 % of the bottleneck row's value of 1000**, versus L_3's ~4e-4 % and CHM's ~0.4 %.

In other words: each rung climb from CHM to L_3 to L_4 cuts the residual tracking error to the bottleneck row by roughly two orders of magnitude. CHM tracks the bottleneck to about four parts per thousand; L_3 tracks it to about four parts per million; L_4 tracks it to about four parts per hundred-million. That is a clean two-decade-per-rung convergence, and it is the formal answer to the question "how strongly does each Lehmer rung weight the largest row?"

The practical implication is significant. If the analytic question is "what is the value of the largest row in this source, with the smallest rows acting as background noise that should be effectively ignored," the L-estimator ladder gives you a tunable answer. CHM is good if you want roughly half a percent of bias toward the bulk; L_3 is good if you want roughly one part in ten thousand; L_4 is good if you want roughly one part in ten million. You move the dial by changing the Lehmer order, not by writing a new lens.

There is a corresponding fact on the other side of the ladder. The new L_-3 lens shipped at v0.6.192 collapses toward the **smallest** row at the same two-decades-per-rung rate. So the eleven-position ladder now provides a tunable knob from "track the smallest row to four parts per hundred-million" all the way through the equal-weight middle (AM) and out to "track the largest row to four parts per hundred-million." That is the architectural shape that v0.6.193 closes off.

## What the live smoke shows

The release notes include a sanitized live-smoke run against the local `~/.config/pew/queue.jsonl` queue (1,835 rows across 6 sources, `--top 8`, sort by `lehmer-4-mean-desc`). Reproduced for reference (with `vscode-XXX` standing in for the redacted source name):

```
source         rows  mean         chm          lehmer-3-mean  lehmer-4-mean
-------------  ----  -----------  -----------  -------------  -------------
claude-code    299   11512995.95  38434042.96  54516204.52    65497172.30
opencode       399   10397083.72  24975374.83  40208796.03    49538379.52
codex          64    12650385.31  28707109.72  38508868.27    44948811.14
openclaw       505   3861266.86   9635367.44   20564374.96    30564457.56
hermes         235   791446.74    1828626.17   2751665.65     3497357.51
vscode-XXX     333   5662.84      45045.25     119764.03      147776.06
```

A few observations.

**First: monotonicity holds row-by-row.** For every source in the smoke, `mean <= chm <= L_3 <= L_4`, with all three derived gaps (l4L3Gap, l4ChmGap, l4AmGap) non-negative. This is what Lehmer monotonicity predicts on real distributional data, and it holds across three orders of magnitude of source size (from `vscode-XXX` at five-thousand-token mean up to `claude-code` at eleven-million-token mean). The lens does not break on small distributions, does not break on heavy-tailed distributions, and does not break on the mixed-positive distributions where some rows are several orders of magnitude smaller than others.

**Second: the l4L3Gap is large.** On `claude-code`, L_4 − L_3 = +10,980,967.78 tokens. On `openclaw`, L_4 − L_3 = +10,000,082.60 tokens. These are substantial gaps — almost 11M tokens of additional location pull from going from L_3 to L_4 in the largest source. What this tells us is that **even at L_3 the largest rows have not fully dominated the location estimate**. There is still meaningful weight being given to second- and third-tier rows under the cube-weighted L_3 lens, and only when we move to the L_4 (cube-self-weighted) lens does the largest row begin to truly dominate. The l4L3Gap is the formal measurement of "how much further the location moves when we add one more power of self-weighting."

**Third: the l4ChmGap is enormous.** On `claude-code`, L_4 − CHM = +27,063,129.34 tokens. On `openclaw`, +20,929,090.12 tokens. The total move from the contraharmonic mean (Lehmer L_2) to the new Lehmer L_4 lens is roughly 70% of the underlying CHM value itself in those sources. This is the empirical version of "two more Lehmer rungs roughly doubles the location estimate" on heavy-tailed sources. On `vscode-XXX`, by contrast, L_4 − CHM = +102,730.82 versus a CHM of only 45,045.25 — even more extreme proportionally (about 228% of CHM). The pattern is: heavier-tailed sources see a larger absolute gap, but the proportional gap is largest where the bulk of rows are tiny and a small handful are enormous. `vscode-XXX` is exactly that distribution.

**Fourth: scale-equivariance verified at three orders of magnitude.** The smallest source in the smoke (`vscode-XXX`, mean ≈ 5,663) and the largest (`claude-code`, mean ≈ 11.5M) show the same qualitative gap pattern. L_4 is scale-equivariant by construction (multiply every row by a constant k and L_4 multiplies by k as well), so this is what we expect, but it is worth noting that the lens behaves exactly as the algebra predicts even when the source rows span eight orders of magnitude in absolute value.

## Why the ladder needed this rung specifically

There is a real question about whether L_4 is "the last useful Lehmer rung" or just one in a potentially infinite progression. Algebraically the family extends to L_p for any real p, and one could ship L_5, L_6, L_-4, L_-5, and so on indefinitely. The reason L_4 is the right place to stop the right-side extension for the moment is the 4e-6 % tracking number above. At that level of tracking accuracy, the L_4 lens is already within a part-per-hundred-million of the bottleneck row. Adding L_5 would push that to about 4e-8 % — a further two-decade improvement, but on real distributional data the difference between "ignore the bulk to 4e-6 %" and "ignore the bulk to 4e-8 %" is operationally indistinguishable. The lens has saturated for any practical question about token-volume location.

There is a parallel saturation argument on the negative side. L_-3 already tracks the smallest row to four parts per hundred-million. L_-4 would push that to four parts per ten-billion, but again on real per-row token distributions where the smallest entry is some integer like 1 or 2 tokens, the difference between L_-3 and L_-4 is below the noise floor of the underlying measurement.

So the eleven-position ladder

```
L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3 <= L_4
```

is approximately the **operational complete ladder** for the per-row token-volume question. It spans from "essentially the minimum" through "the geometric middle" through "the equal-weight middle" all the way through "essentially the maximum," with each step of one rung corresponding to roughly two decades of additional bias toward one or the other extremum. The intermediate rungs cover the full range of practical tradeoffs between robustness to outliers (left side) and sensitivity to outliers (right side).

## The "named-or-not" status of the new rungs

One subtlety worth noting: of the eleven positions on the ladder, only five have named identities in classical statistics — the harmonic mean (L_-1), geometric mean (L_0, on the limit), arithmetic mean (L_1), quadratic mean / RMS (L_2 power-mean, distinct from L_2 Lehmer = CHM), and the contraharmonic mean (L_2 Lehmer). The other six positions — L_-3, L_-2, L_3, L_4, plus the implicit L_1 Lehmer = AM identity and L_0 Lehmer = HM identity — are "anonymous" in the sense that they do not have widely used names outside the Lehmer-mean literature. v0.6.193 ships them as just-as-first-class lenses anyway, with the same byproducts and cohort selectors as the named rungs.

This is the right architectural call. The classical "named means" are named for historical reasons that have nothing to do with their analytic utility. From the standpoint of an L-estimator family, L_-3 and L_4 are no less well-defined than HM and AM; they are simply the rungs that did not happen to acquire names because they are not as commonly used in the elementary-statistics curriculum. By shipping them as normal lenses, `pew-insights` removes the privilege of the named rungs and treats the entire ladder as a uniform analytic surface.

## What L_4 means for the source-cohort selectors

Every `source-row-token-*` lens in `pew-insights` ships with a `--min-<lens>` cohort selector that lets the user filter to sources whose lens value exceeds a threshold. v0.6.193 adds `--min-lehmer-4-mean` to the family, completing the eleven-position cohort-selection surface.

This matters operationally because the cohort selectors are how this lens gets used in practice. The question "which sources have at least one row larger than 10M tokens" is exactly the question that L_4 answers efficiently — `--min-lehmer-4-mean 10000000` returns the cohort of sources whose largest row dominates their distribution to better than 10M-token level. The cohort selector exposes the lens directly for the operational question, without forcing the user to compute the differential gaps themselves.

## What this release is not

It is worth being explicit about what v0.6.193 does not do. It does not change any existing lens. It does not invalidate any prior version's output. It does not introduce any new dependency. It does not change the test architecture beyond adding the unit tests for the new subcommand and one closed-form refinement property establishing that L_4 equals the cube-self-weighted arithmetic mean.

It also does not commit `pew-insights` to extending the ladder further. There is no implicit promise of an L_5 release, or an L_-4 release, or a continuous-Lehmer-order lens that takes p as a parameter. The eleven-rung ladder is what is shipped; whether there is operational demand for further rungs is a question for future releases.

## What L_4 reveals about the underlying token distributions

Beyond the architectural completeness story, the live-smoke data tells us something genuine about the shape of the per-source token-volume distributions in the queue.

The fact that l4L3Gap is on the order of **30–40% of the L_4 value** in the largest sources tells us that those distributions are heavy-tailed but not extreme — there is no single bottleneck row dominating. If `claude-code` had a single row of, say, 100M tokens, then L_3 would already be tracking it to four parts per million and L_4 would only add a tiny correction. The fact that L_4 adds nearly 11M tokens of additional pull above L_3 means that the top of the distribution has multiple large rows competing for influence; the cube-weighting (L_4) gives the absolute largest row enough additional bias to pull the location estimate further than the square-weighting (L_3) did.

This is consistent with what the trim-mean-25, midhinge, trimean, and quartile-deviation lenses have been showing for the past week: `claude-code` and `opencode` have heavy-but-multi-modal upper tails, not single-bottleneck distributions. The L_4 lens is the formal answer to "how much does cube-weighting separate from square-weighting on a multi-modal heavy tail?" — and the answer is "roughly 20% of the underlying location, on the heaviest sources in the queue."

`vscode-XXX` is the cleanest exhibit of single-bottleneck distribution behavior. Its mean is 5,662.84; its CHM is 45,045.25; its L_3 is 119,764.03; its L_4 is 147,776.06. The CHM-to-L_4 ratio is ~3.28x and the AM-to-L_4 ratio is ~26.1x. That ladder of ratios is the signature of "most rows are small, a few rows are very large" — exactly the distribution we would expect from a mostly-quiet integration whose occasional bursts dominate the token-volume aggregate.

## Closing

v0.6.193 is the kind of release that is structurally important even when it is operationally quiet. It does not produce any new headline number; it produces the rung that proves all the prior rungs are part of a single coherent family. The 4e-6 % bottleneck-row tracking number is the most precise quantitative statement to date of what the right-side L-estimator extension actually does, and the eleven-position ladder it completes is — for the per-row token-volume question — operationally complete.

The next release will not extend the ladder. It will start to use it.
