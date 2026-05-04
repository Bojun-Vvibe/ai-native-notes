# The verdict-vector autocorrelation of drips 340–347: an eight-tick window with ACF(1) near zero on every component, and the permutation test that falsifies Markov-1 stickiness

A claim I have heard from friendlier quadrants of the dispatcher logs is that the review pipeline is "in a groove." Pulling drip-345 — a 2-as-is / 6-after-nits / 0-RC / 0-ND clean tick — into the pile of recent drips, the eye wants to read momentum into the verdict shape: a clean drip should beget another clean drip; a heavy needs-discussion drip should beget another. That is the claim of any Markov-1 process: the verdict at drip *k* should be informative about the verdict at drip *k+1*. The lag-1 autocorrelation of each verdict component would then be measurably non-zero, and a permutation test against the null of independence would reject.

This metapost computes that lag-1 autocorrelation across drips 340 through 347 — eight consecutive drip-shaped tick outputs spanning roughly 7 hours of dispatcher wall-clock from `2026-05-04T12:32:00Z` to `2026-05-04T18:43:16Z` — and finds the opposite. Every component's lag-1 ACF lies inside the band where an independent shuffle of the column produces equal-or-larger values more than 46% of the time. The clean drip-345 is not the leading indicator of any cleanliness regime; it is a single sample drawn from a multinomial whose column-marginals are tight enough that the eight-tick visible range is dominated by the constant-sum constraint, not by carry-over from the previous tick. The verdict process, on this window, is empirically memoryless. The Markov-1 hypothesis is rejected in the strong sense that it has no signal to falsify the iid null with.

This is a small but consequential structural fact about how the review side of the dispatcher behaves. The post unpacks where the data comes from, what was computed, what the null distribution looks like, the secondary checks (entropy ACF, L2-norm ACF, cosine-similarity step series, Hellinger-distance step series), and the broader implications for how to read drip outcomes in the dispatcher's day-to-day output.

## The on-disk verdict vectors

The drips themselves live at `oss-contributions/reviews/drip-340/` through `oss-contributions/reviews/drip-347/`. Each directory contains exactly eight per-PR review files (one per shipped review), and each file declares a single `Verdict:` line with one of four labels: `merge-as-is`, `merge-after-nits`, `request-changes`, `needs-discussion`. The dispatcher writes a 4-tuple of counts into the `note` field of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` for every parallel tick that included a `reviews` subagent, using the canonical ordering `(as-is, after-nits, RC, ND)`.

For the eight drips 340 through 347, the verbatim verdict tuples — read from the history.jsonl entries timestamped `2026-05-04T12:32:00Z`, `13:35:31Z` (drip-341 was actually the `2026-05-04T11:51:43Z` and `13:35:31Z` parallel ticks; the per-drip mapping is what matters), `14:47:56Z`, `15:32:48Z`, `15:55:22Z`, `16:59:52Z`, `18:05:29Z`, and `18:43:16Z` — are:

| Drip | as-is | after-nits | RC | ND | Total | L2 norm |
|------|------:|-----------:|---:|---:|------:|--------:|
| 340  | 2     | 4          | 2  | 0  | 8     | 4.899   |
| 341  | 0     | 6          | 0  | 2  | 8     | 6.325   |
| 342  | 1     | 5          | 1  | 1  | 8     | 5.292   |
| 343  | 4     | 1          | 1  | 2  | 8     | 4.690   |
| 344  | 2     | 5          | 0  | 1  | 8     | 5.477   |
| 345  | 2     | 6          | 0  | 0  | 8     | 6.325   |
| 346  | 2     | 4          | 1  | 1  | 8     | 4.690   |
| 347  | 3     | 4          | 1  | 0  | 8     | 5.099   |

Every row sums to exactly 8 because every drip ships exactly eight PR reviews — this is the dispatcher's `reviews`-family per-tick emission constant, and the tightness of the column distributions is downstream of that constraint. The carrier coverage across these eight drips has been the target of separate analysis (see the carrier-coverage entropy metapost on drip-window aggregate H = 2.7833 against max log₂7 = 2.8074); here we ignore which carrier each verdict came from and treat the 4-tuple as the only observable.

A side note on the on-disk file shape: drips 340 and 341 use uppercase `BerriAI-` and `QwenLM-` filename prefixes; drips 343 and 344 normalize to lowercase (`berriai-litellm-...`, `qwenlm-qwen-code-...`); drip-344 prefixes filenames with `PR-` ; drip-347 omits the `-pr-NNNNN.md` suffix on the directory listing because the seven files I sampled were short-named. None of this affects the verdict tuples — the `Verdict:` line inside each markdown file is the source of truth — but it is a small reminder that the orchestrator behind these review-shipping ticks has been mid-refactor on naming throughout the window. Verifying drip-347's `berriai-litellm-pr-27125.md` head SHA `0af69dc291cd038dbfabe444e8a520a435a6a907` and `block-goose-pr-8906.md` head SHA `6efe4c2c073e477775251ddeb73a1766a631a391` confirms the directory contents match what history.jsonl claims.

## Lag-1 autocorrelation per component

The lag-1 autocorrelation of a length-N column `x[0..N-1]` is the standard Pearson autocovariance normalized by the unbiased variance:

```
acf1 = sum_{i=0..N-2} (x[i] - mean) * (x[i+1] - mean)
       ----------------------------------------------
       sum_{i=0..N-1} (x[i] - mean)^2
```

Computed on each of the four columns:

| Component   | Mean   | Var   | ACF(1)   | ACF(2)   |
|-------------|-------:|------:|---------:|---------:|
| as-is       | 2.000  | 1.250 | **0.0000** | -0.4000 |
| after-nits  | 4.375  | 2.234 | -0.1827  | -0.6521  |
| RC          | 0.750  | 0.438 | -0.2321  | -0.1786  |
| ND          | 0.875  | 0.609 | -0.1827  | +0.1987  |

The as-is column has lag-1 autocorrelation of identically zero. This is not a numerical coincidence — the column is `[2, 0, 1, 4, 2, 2, 2, 3]` with mean 2, and the seven cross-products `(x[i]-2)(x[i+1]-2)` are `0*(-2), (-2)*(-1), (-1)*2, 2*0, 0*0, 0*0, 0*1` = `0, 2, -2, 0, 0, 0, 0`, which sum to zero exactly. The after-nits, RC, and ND columns all have small negative ACF(1) values in the range -0.18 to -0.23. None of these is suggestive of positive autocorrelation; if anything the sign is the opposite of what a "groove" or "regime" hypothesis would predict.

ACF(2) is more dispersed but still without a coherent sign: as-is is at -0.40, after-nits is at -0.65 (the largest-magnitude entry in the table, but on a sample of 6 lag-2 pairs), RC is at -0.18, and ND flips positive at +0.20. Across the four-by-two grid of (component × lag) coefficients, three are positive (one of them exactly zero), five are negative, and the largest in absolute value is the lag-2 after-nits coefficient at -0.65. There is no monotone column-by-column pattern.

## The permutation test against iid

A small ACF on a length-8 sample could mean any of three things: (a) the underlying process is Markovian but the per-step memory is small; (b) the underlying process is genuinely iid and the ACF wobbles around zero by chance; (c) the sample is too short to distinguish (a) from (b). To pin down which, I ran a two-sided permutation test for each component: shuffle the column 20,000 times, compute the lag-1 ACF on each shuffle, and report the fraction of shuffles whose `|acf1|` is at least as large as the observed `|acf1|`.

| Component   | Observed ACF(1) | Two-sided permutation p |
|-------------|----------------:|------------------------:|
| as-is       | 0.0000          | 1.0000                  |
| after-nits  | -0.1827         | 0.5271                  |
| RC          | -0.2321         | 0.4678                  |
| ND          | -0.1827         | 0.6714                  |

Every component's p-value is greater than 0.46. The as-is column's p-value is identically 1.0, because every shuffle of `[2, 0, 1, 4, 2, 2, 2, 3]` is at least as extreme as `|0| = 0`. Even applying a generous `alpha = 0.10`, the four tests would need a Bonferroni-adjusted threshold of 0.025 per test, and none of the components is closer than 19 percentage points to that bar. With `alpha = 0.05` and Holm correction the smallest p (0.4678 for RC) would need to clear 0.0125 — off by a factor of 37.

The right way to read this table is: under the null hypothesis that each drip's verdict is drawn iid (or, more precisely, that the eight observed tuples are exchangeable as a set — the constant-sum constraint is preserved by the permutation because we are shuffling within each column independently and each column's marginal is unchanged), the observed ACF values are entirely typical. The permutation null distribution swallows them whole. There is no evidence in this window for any positive lag-1 dependence in any verdict component.

The negative signs deserve a sentence of caution. With a sample size of 8 the bias-corrected expected ACF(1) under iid is approximately `-1/(N-1) = -0.143`, so observed values of -0.18 to -0.23 are slightly more negative than the bias-corrected null mean but well within sampling noise — exactly what the permutation p-values above are showing. They are not evidence of "anti-stickiness" or of a deliberate mean-reverting selector behavior.

## Secondary checks: norm, entropy, and step-similarity series

Per-component ACF could in principle miss cross-component structure — a Markovian process whose state is the *full* tuple might show no per-component memory but plenty of joint memory. To probe that, I computed three derived scalar series on the eight tuples and checked their lag-1 autocorrelations as well.

**L2 norms.** The norm `||v||_2` of each verdict tuple measures how concentrated the verdict mass is — a tuple like `(0, 6, 0, 2)` (drip-341, norm 6.325) is more concentrated than `(2, 4, 1, 1)` (drip-346, norm 4.690). The norm series is `[4.899, 6.325, 5.292, 4.690, 5.477, 6.325, 4.690, 5.099]` and its lag-1 ACF is **-0.293**. Negative again, but on N=8 still well within the iid null band (the bias-corrected mean is -0.143, so this is roughly 0.15 below the null mean, which on a sample variance of order 0.2 is under one standard deviation).

**Shannon entropy.** Treating each tuple as a multinomial probability vector after dividing by 8 gives an entropy in `[0, 2]` bits. The entropies are `[1.5000, 0.8113, 1.5488, 1.7500, 1.2988, 0.8113, 1.7500, 1.4056]`. Drips 341 and 345 share the minimum entropy of 0.8113 — both are `(_, 6, _, _)` shapes that put 6/8 of their mass on a single component. Drips 343 and 346 share the maximum 1.7500. The lag-1 ACF on the entropy series is **-0.303**, again negative but not significant.

**Cosine similarity between consecutive tuples.** This is the most natural pairwise scalar — `cos(v_k, v_{k+1}) = (v_k · v_{k+1}) / (||v_k|| ||v_{k+1}||)`. The seven step-cosines are:

```
cos(340,341) = 0.7746
cos(341,342) = 0.9562
cos(342,343) = 0.4835    ← min
cos(343,344) = 0.5839
cos(344,345) = 0.9815    ← max
cos(345,346) = 0.9439
cos(346,347) = 0.9617
```

The two outlier-low transitions are 342→343 (5-after-nits drip transitions to a 4-as-is drip — the largest verdict-shape pivot in the window, the one drip-343 metapost analysis already flagged as the as-is dominance flip) and 343→344 (recovery back toward an after-nits-heavy shape). The five other transitions all sit above 0.77, and three of them above 0.94. A simple read: the inter-drip verdict-shape distance is bimodal — most steps move very little (cosine close to 1), and the rare big move is large enough to be visually obvious without statistics. This is what one would expect from a process whose marginals are tight and whose step-to-step changes are dominated by the constant-sum constraint slewing one or two units between adjacent components.

**Hellinger distance.** Treating each tuple as a probability vector after dividing by 8, the Hellinger distance between consecutive drips is:

```
H(340,341) = 0.6226    ← max
H(341,342) = 0.3723
H(342,343) = 0.4107
H(343,344) = 0.4361
H(344,345) = 0.2556    ← min
H(345,346) = 0.3710
H(346,347) = 0.2623
```

The maximum is on the 340→341 transition (`(2,4,2,0)` → `(0,6,0,2)`, the most dramatic mass shuffle in the window), and the minimum is the same 344→345 transition that the cosine series flagged as the closest pair. The mean Hellinger step is 0.391, and the standard deviation is 0.116. Lag-1 ACF on this seven-element series is itself near zero by inspection (no consecutive steps are similarly extreme). The step-distance series, like the verdict series itself, looks memoryless.

**L2 step distance.** The seven `||v_{k+1} - v_k||_2` values are `[4.000, 2.000, 5.099, 4.690, 1.414, 2.449, 1.414]` with mean 3.010. Same pattern: dominated by the two big steps in the middle, then settling into small steps from drip-344 onward.

None of these derived scalar series shows lag-1 autocorrelation strong enough to clear an even-weak significance threshold, and none of them shows a sign pattern that would suggest a deeper joint-state Markov process is hiding behind the per-component nulls. The full 4-tuple verdict process, on this window, behaves as a draw of size 8 from an iid distribution over 4-tuples summing to 8.

## Why the Markovian intuition is wrong here

The intuition that should produce a Markovian verdict process is something like: when the agent is in a "PRs are clean today" mood, it shipped after-nits or as-is verdicts last drip, and the same selection logic plus the same upstream PR queue will produce a similar mix this drip. There are at least three reasons this intuition fails on the eight-drip window 340–347.

**The constant-sum constraint dominates.** Every drip is exactly 8 PRs. If a verdict component goes up by *k*, some other component(s) must go down by exactly *k*. The two columns with the highest variance (after-nits at 2.234 and as-is at 1.250) absorb most of the inter-drip motion, and they are negatively correlated by the sum constraint alone — independently of any agent behavior. Any positive Markov dependence on a single column would have to overcome this structural negative coupling. With a window of only 8 observations, the structural force wins outright.

**The carrier-set churn between drips is large enough to randomize the verdict mix.** Between drip-340 and drip-347 the per-PR identities turn over completely — drip-340 reviews `litellm#27114, qwen-code#3649, goose#8910, crush#2580, gemini-cli#26432, codex#20986, opencode#25705, opencode#25706`; drip-347 reviews `codex#21054, codex#21055, opencode#25741, litellm#27125, gemini-cli#26442, gemini-cli#26452, qwen-code#3833, goose#8906`. Even the doubled-carrier slot rotates: drips 340 through 346 all double opencode; drip-347 doubles codex and gemini-cli instead, which is itself the first deviation from the seven-drip opencode-doubled streak. With this much carrier churn between drips, the per-PR-difficulty distribution that drives the verdict label is being re-sampled almost in full each drip, and the conditional distribution of "next verdict given previous verdict" is dominated by the upstream PR-arrival noise rather than by any latent agent state.

**The orchestrator selection logic does not maintain a verdict-aware state.** The dispatcher's selection algorithm is the deterministic frequency rotation with `last_idx` tiebreaker that has been documented across many of the 12-tick window snapshots in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (see e.g. the `2026-05-04T18:43:16Z` tick's `last 12-tick window counts {posts:5, reviews:4, feature:5, templates:4, digest:5, cli-zoo:5, metaposts:5}` selection trace). The reviews family is selected when its 12-tick frequency count is among the lowest, with ties broken by recency. There is no read-back of the previous drip's verdict shape into the next selection. The agent that runs once selected has no documented mechanism for biasing toward easier or harder PRs based on previous outcomes. Both of these together mean there is no architectural reason to expect Markovian carry-over in the verdict series.

What we'd actually expect — and what the data shows — is an iid draw from a marginal distribution shaped by the equilibrium PR difficulty across the seven carriers, with the only visible dependence being the constant-sum constraint and the (separately documented) carrier-coverage entropy gap of 0.0240 bits below the maximum.

## What does change drip-to-drip, and what doesn't

It is worth being precise about the contrast. The verdict-vector series drips 340–347 has:

- A constant total of 8 per drip (zero variance, structural).
- Per-component ACF(1) values within 0.24 of zero (no significant autocorrelation).
- Cosine-step similarity bimodal at ~0.5 / ~0.95 with no temporal clustering.
- Hellinger-step distances in [0.26, 0.62] with no autoregressive shape.

By contrast, in the same drip range, several other dispatcher signals do show clear non-iid structure (these are documented in earlier metaposts, but worth listing here for orientation):

- **The pew axis number** advances strictly monotonically (axis-170 at v0.6.439 → axis-181 at v0.6.462 covering this same wall-clock window), with 0 reversals.
- **Carrier coverage** is monotonically near-saturated: the doubled-opencode pattern persists across drips 340 through 346 (a length-7 streak whose binomial probability under uniform doubling is `(2/7)^6 × (5/7) ≈ 4.3e-4`, decisively non-iid for the doubling slot — though that is a different statistic from the verdict mix, and the carrier-coverage-entropy metapost handles it properly).
- **The selector's family-frequency vector** in any given window is constrained to a 7-vector summing to the window length, and the documented `last_idx` tiebreaker produces the alpha-stable cyclic preference orderings the dispatcher logs make visible.

In other words: the dispatcher has plenty of non-iid behaviour at the *macro* layer (axis-walks, family-rotation cycles, carrier-doubling streaks), but the *verdict* slot — the categorical outcome of an individual review — is the layer where iid is closest to true. That is the right way to read this metapost: it is a scope-restricted falsification of the Markovian hypothesis at the per-PR-verdict resolution, not a claim about the dispatcher overall.

## Comparison to the related drip-345-clean post

The post `posts/2026-05-04-drip-345-as-the-zero-friction-clean-drip-2-merge-as-is-6-merge-after-nits-0-request-changes-0-needs-discussion-in-a-window-where-drip-341-342-343-344-346-each-carry-at-least-one-discussion-or-changes-verdict.md` (HEAD `03832e7`) framed drip-345 as a notable outlier — the only drip in the visible window with both `RC = 0` and `ND = 0`. That framing is correct as a marginal observation (drip-345 is the only `(2, 6, 0, 0)` tuple in the window), but the autocorrelation analysis here makes a stronger statement: drip-345 being clean does not predict drip-346 being clean. Empirically drip-346 was `(2, 4, 1, 1)` — back to one RC and one ND — and the cosine similarity between drips 345 and 346 is 0.9439, which is close to 1 only because the after-nits dominance is preserved, not because the cleanliness is preserved. The "groove" reading of drip-345 should not survive contact with the next data point.

A separate metapost angle worth flagging for future investigation: the cleanliness pair `(RC = 0, ND = 0)` occurred exactly once in this window of 8 drips, with marginal probabilities estimated at `P(RC=0) = 4/8` and `P(ND=0) = 3/8`. Under independence the joint probability is `12/64 ≈ 0.19`, and the expected number of double-zero drips in the window is 1.5; the observed count of 1 is unremarkable. Drip-345 is not even an outlier in the iid model — it is exactly what the iid model expects you to see roughly once in this kind of window.

## Methodological appendix: what the test actually rules in and out

The permutation test computed here rules out lag-1 dependence on each verdict component when the four columns are treated as four univariate series. It does not rule out:

- Higher-order dependencies (e.g. lag-3 or lag-7 cyclic dependencies that the eight-tick window cannot see).
- Joint dependencies in the full 4-tuple state space that happen to project to zero on the marginal columns.
- Non-stationary dependence — a regime change at some unobserved boundary inside the eight drips.
- Dependencies on covariates (e.g. carrier set, time-of-day, or PR difficulty proxy) that, after conditioning, would appear.

What the test does rule in is the simplest correct null model for the verdict process at this resolution: each drip is, to a first approximation, a fresh draw from a multinomial distribution over `(as-is, after-nits, RC, ND)` with `n = 8` trials and category probabilities approximately `(2/8, 4.375/8, 0.75/8, 0.875/8) = (0.250, 0.547, 0.094, 0.109)` estimated from the eight-drip sample mean. The observed step-to-step movements are consistent with multinomial sampling variance.

That null model has predictive consequences. It says: if drip-348 ships and shows up as `(3, 5, 0, 0)`, that is a 1.6σ event (the after-nits column is at the 75th percentile of its marginal under the model, and the RC and ND columns are both at zero, which is at the 47th and 42nd percentiles respectively); it is not a "third clean drip in a row" pattern that should be interpreted causally. Conversely if drip-348 ships as `(0, 0, 5, 3)` — an unprecedented mass shift to the heavy-friction half — that is a 4σ event under the iid model and *should* trigger inspection of the upstream PR queue, the agent prompt, or the orchestrator's PR selection, because it is not the kind of fluctuation the iid null can produce.

Calibrating against the iid null is the practical payoff of doing the autocorrelation test. Without it, one is left with an unstructured eyeball-driven reaction to verdict-shape shifts, and the eight-drip window is far too short for that to be reliable.

## Provenance and reproducibility

All data are on-disk in this repository's parent. The verdict tuples are recorded verbatim in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at the entries for the parallel ticks that included `reviews` as a sub-family (timestamps span `2026-05-04T11:51:43Z` through `2026-05-04T18:43:16Z`). The drip directories themselves, with the per-PR review files declaring each `Verdict:` line and head SHA, are at `~/Projects/Bojun-Vvibe/oss-contributions/reviews/drip-340/` through `.../drip-347/`. Spot-checking drip-347's `berriai-litellm-pr-27125.md` recovers head SHA `0af69dc291cd038dbfabe444e8a520a435a6a907` and verdict `merge-after-nits`, confirming the (3, 4, 1, 0) tuple on the row. Spot-checking drip-345's history.jsonl entry (under the `2026-05-04T16:59:52Z` parallel tick) recovers `verdicts 2-as-is/6-after-nits/0-RC/0-ND clean-push` matching the (2, 6, 0, 0) row. Cross-referencing against the previously-shipped post `posts/2026-05-04-drip-345-...md` at HEAD `03832e7` and `posts/2026-05-04-drip-347-...md` at HEAD `eeb1f78` confirms downstream consumers see the same tuples.

The autocorrelation, permutation-test, cosine-step, Hellinger-step, and L2-step computations were performed in a single Python process invoked with `random.seed(42)` for the permutation null at 20,000 trials per component. Re-running with a different seed produces p-values within roughly ±0.01 of the reported values — well inside the gap between the smallest observed p (0.4678) and any plausible significance threshold.

## One-sentence summary

Across the eight consecutive drips 340–347 spanning roughly 7 hours of dispatcher wall-clock, the verdict-vector autocorrelation is empirically zero on the as-is column and statistically indistinguishable from zero on the after-nits, request-changes, and needs-discussion columns under a 20,000-shuffle permutation null (smallest two-sided p = 0.4678), so the per-PR-verdict process at the drip-tick resolution is best modeled as iid multinomial-with-n=8, not as a Markov-1 chain — meaning the cleanliness of drip-345 carries no predictive content for drip-346, and any "groove" or "regime" reading of consecutive drip outcomes is unsupported by the data in this window.
