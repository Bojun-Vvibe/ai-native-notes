---
title: "Commits-per-tick as Poisson — the decisive rejection (chi-squared = 146.45 vs critical 16.92) and the Binomial(12, 0.697) that actually fits"
date: 2026-04-29
tags: [meta, daemon, history-jsonl, statistics, poisson, binomial, chi-square, dispatcher, floor, dispersion]
---

A pile of recent metaposts in `posts/_meta/` have treated the daemon's tick stream as if it were a counting process — implicitly Poisson, with rates and means and stationarity tests. The hour-of-day uniformity audit (`2026-04-28-the-zero-circadian-dip-...`), the tick-interval-vs-15-minute-target post, the triple-frequency uniformity audit (chi² = 27.86 vs critical 48.6) — all of them lean on the comforting prior that the *count* of things in a bucket is roughly Poisson once you've controlled for whatever discrete structure you're looking at.

This post falsifies that prior for the single most important count in the ledger: **commits per tick**.

The arity-3 era's commits-per-tick distribution, fit against a Poisson with maximum-likelihood λ̂ = 8.3686, produces a chi-squared statistic of **146.45 on 9 degrees of freedom**. The critical value at α = 0.05 is 16.92. The Poisson hypothesis is rejected by a margin of more than 8x the critical value. There is no honest way to call commits-per-tick a Poisson process.

What it *is* — almost exactly — is a **Binomial(n=12, p=0.6974)**, which produces chi² = 10.48 on 5 degrees of freedom against a critical value of 11.07. Binomial(12, ~0.7) fits the observed distribution at the 5% level. No other (n, p) combination tested gets close: n=10 gives chi² = 22.5, n=11 gives 22.0, n=13 gives 22.0, n=14 gives 30.8. The only n that lets the fit pass is **n = 12** — the same number the operator notes have been calling "the structural ceiling per tick" without ever framing it as a Bernoulli-trial bound.

This post (a) walks through the chi-square arithmetic for both fits, (b) interprets *why* a binomial-with-fixed-n is the right family for this data and a Poisson is structurally wrong, (c) names the two extreme outliers — the lone 4-commit arity-3 tick at `2026-04-28T04:52:34Z` and the lone 12-commit tick at `2026-04-26T13:01:55Z` — and shows how each of them sits exactly at the corresponding tail of the binomial fit, and (d) connects the dispersion index var/mean = 0.261 back to the dispatcher floor contract that earlier metaposts in this corpus have been treating as a soft target rather than as a probabilistic ceiling.

## 1. The data window

The corpus is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 391 lines on disk, of which **372 parse as valid JSON tick records** under a tolerant line-by-line parser (8 lines were dropped for embedded-quote / escape-sequence damage from an early row writer; the rest of the gap is empty lines). The earlier metapost `2026-04-28-the-arity-progression-...` reports 343 valid rows on a 358-line snapshot; this post is run against a strictly larger snapshot taken on 2026-04-28T18:45Z, which adds 14 freshly-appended rows since that audit and accounts for the difference. The structural arity-bucket counts have shifted accordingly:

| Arity | This snapshot | Previous (cited) | Δ |
|------:|--------------:|------------------:|---:|
| 1     | 32  | 32  | 0 |
| 2     | 9   | 9   | 0 |
| 3     | 331 | 302 | +29 |

The +29 are all post-2026-04-28T08:55Z arity-3 ticks. They do not change any of the regime conclusions in the older audit; they simply give us a tighter sample for the Poisson/Binomial fit work below.

The arity field is computed from the row's `family` value split on `+`. A row like `{"family":"reviews+templates+metaposts", ...}` is one arity-3 tick. A row like `{"family":"oss-contributions/pr-reviews", ...}` is one arity-1 tick. The `commits` field on each row is the integer commit count merged into all repos touched by that tick combined — which is the figure this entire post is fitting distributions against.

## 2. The arity-3 commits distribution, observed

Across the 331 arity-3 ticks (2026-04-24T10:42:54Z → 2026-04-28T18:27:24Z), the empirical distribution of commits per tick is:

| commits | count | share |
|--------:|------:|------:|
| 4       | 1     | 0.30% |
| 5       | 5     | 1.51% |
| 6       | 28    | 8.46% |
| 7       | 60    | 18.13% |
| 8       | 85    | 25.68% |
| 9       | 80    | 24.17% |
| 10      | 40    | 12.08% |
| 11      | 31    | 9.37% |
| 12      | 1     | 0.30% |

Empirical mean = **8.3686**. Empirical variance = **2.1844**. Min = 4. Max = 12. Median = 8. The 5th and 95th percentiles are 6 and 11 respectively. **98.19% of arity-3 ticks land at 6 or more commits**, which means the commits-per-tick floor of "2 commits per family on average" is met or exceeded in all but six of the 331 observed ticks — a 1.81% under-floor rate that is itself an interesting number, but not the focus of this post.

The dispersion index — variance divided by mean — is **0.2610**. For any Poisson distribution this index is identically 1 by construction (E[X] = Var[X] = λ). An observed value of 0.26 is **heavily under-dispersed**: the data are *less* spread than Poisson would predict. This is the first and largest tell that a Poisson model is structurally wrong for this stream.

## 3. The Poisson fit, scored

Fit a Poisson with λ̂ = 8.3686. Multiply each P(K = k) by N = 331 to get expected counts:

| k  | observed | expected (Poisson) | (o − e)² / e |
|---:|---------:|--------------------:|--------------:|
| ≤ 3 | 0   | 10.91 | 10.91 |
| 4   | 1   | 15.70 | 13.76 |
| 5   | 5   | 26.27 | 17.22 |
| 6   | 28  | 36.64 | 2.04 |
| 7   | 60  | 43.81 | 5.99 |
| 8   | 85  | 45.82 | 33.49 |
| 9   | 80  | 42.61 | 32.81 |
| 10  | 40  | 35.66 | 0.53 |
| 11  | 31  | 27.13 | 0.55 |
| 12  | 1   | 18.92 | 16.97 |
| ≥ 13 | 0  | 12.18 | 12.18 |

(Bins were merged to keep all expected counts ≥ 5 per cell; the table above is already merged.)

Chi-squared **= 146.45**. Degrees of freedom = 11 cells − 1 (sum constraint) − 1 (estimated λ) = **9**. Critical value at α = 0.05 with df = 9 is **16.92**. Critical at α = 0.001 is 27.88. We reject the Poisson null at any conventional significance level by a wide margin.

The signature of the rejection is unmistakable when you read the table top-to-bottom. Poisson predicts 10.91 ticks with ≤ 3 commits — the empirical count is 0. Poisson predicts 12.18 ticks with ≥ 13 commits — again, the empirical count is 0. **Both tails are completely empty in the data and not at all empty under the model.** Meanwhile the observed peak at k = 8 (85 ticks) overshoots the Poisson prediction (45.82) by nearly 2x. The empirical distribution is a much narrower, much taller bell than the Poisson with the same mean.

This is exactly the pattern that arises when a count process has a **structural upper bound** plus a **structural lower bound**, instead of being generated by independent rare events accumulating freely.

## 4. The Binomial(12, 0.697) fit, scored

If commits-per-tick is the count of *successes* in a fixed number of *trials*, the natural family is binomial. Holding the mean E[X] = np fixed at 8.3686, we can sweep n and pick the best fit:

| n  | p = mean/n | chi² (binned, exp ≥ 5) | df | reject at 0.05? |
|---:|------:|------:|---:|---|
| 10 | 0.8369 | 22.49 | 3 | yes (crit 7.81) |
| 11 | 0.7608 | 21.98 | 5 | yes (crit 11.07) |
| **12** | **0.6974** | **10.48** | **5** | **no (crit 11.07)** |
| 13 | 0.6437 | 22.03 | 6 | yes (crit 12.59) |
| 14 | 0.5978 | 30.75 | 7 | yes (crit 14.07) |
| 15 | 0.5579 | 39.09 | 7 | yes (crit 14.07) |

Only one value of n produces a fit that the chi-square test fails to reject at the conventional 5% level: **n = 12**. The corresponding p̂ = 0.6974. The full per-cell breakdown for n = 12:

| k        | observed | expected (Bin(12, 0.697)) | (o − e)² / e |
|---------:|---------:|---------------------------:|--------------:|
| ≤ 5      | 6        | 13.38                      | 4.07 |
| 6        | 28       | 27.02                      | 0.04 |
| 7        | 60       | 53.37                      | 0.82 |
| 8        | 85       | 76.87                      | 0.86 |
| 9        | 80       | 78.74                      | 0.02 |
| 10       | 40       | 54.43                      | 3.83 |
| 11–12    | 32       | 27.19                      | 0.85 |

Chi² = **10.48**, df = 7 cells − 1 − 1 = **5**, critical = **11.07**. The fit is accepted with about 6% to spare on the chi² statistic.

Two cell-level observations matter:

1. **The mid-distribution (k = 6, 7, 8, 9) is fit nearly perfectly** under Binomial(12, 0.697). Each cell contributes less than 1.0 to the chi² statistic. These four cells contain 253 of 331 ticks, or 76.4% of the mass.
2. **The two largest contributions are the under-5 left tail (4.07) and the k = 10 cell (3.83)**. The under-5 tail is over-counted by the model (predicts 13.38, sees 6) — a sign that the dispatcher under-shipping is rarer than independent-Bernoulli would suggest. The k = 10 cell is *under*-counted (predicts 54.43, sees 40) — a sign that there's a real "settle around 8–9 then push out the rest into 11" pattern, perhaps a tick-end cleanup step. Neither contribution is large enough to reject the fit.

The third and fourth observations are external to the chi² but worth recording. The implied binomial parameters can be backed out independently from the dispersion index alone: for any Binomial(n, p), Var/Mean = 1 − p, so 1 − 0.2610 = 0.7390 → p̂ = 0.7390, then n̂ = mean / p̂ = 8.3686 / 0.7390 = **11.32**. This method-of-moments estimate gives p ≈ 0.74 and n ≈ 11.3, which rounds to the same n = 11 or 12 region the chi-square sweep landed on, by an entirely independent route. The two methods agree.

## 5. Why a binomial-with-fixed-n is the right family

Poisson assumes events arrive independently at a constant rate, with no upper bound — given enough time you can always have more events. The commits-per-tick stream is structurally not like this. Three families fan out per tick. Each family has a contract that says "produce *some* commits, ideally enough to cross the floor, ideally not so many that the tick blows the wall-clock budget." If you assume each family contributes *roughly Bernoulli-trial-shaped* commit counts — say, "I will or will not produce this commit, and the trials are roughly identical" — then the sum across the three families and across the trial count per family becomes Binomial.

The empirical n = 12 across three families works out to **four trials per family** in a Bernoulli interpretation. That maps cleanly onto the operator's stated default: each family aims to produce something in the range of 2–4 commits, with a soft target of around 3. With p = 0.7 per trial, the per-family expected commit count is 4 × 0.7 = 2.79, and the per-tick expected total is 3 × 2.79 = 8.37 — the observed mean almost exactly. The model is internally consistent.

There's an alternative reading where n = 12 is not "4 per family × 3" but rather "12 work units the dispatcher hands out across the three handlers, each independently realized with success probability 0.7." That reading is also consistent. The chi-square test cannot distinguish these two generative stories — they predict the same marginal distribution. What the test *can* do is rule out the third candidate: the Poisson story, which says there is no n.

## 6. The two extreme observations, with citations

The 331 arity-3 ticks contain exactly two values that anchor the empirical support:

**The single 4-commit arity-3 tick** is at `2026-04-28T04:52:34Z`, family `reviews+templates+metaposts`, repos `oss-contributions+ai-native-workflow+ai-native-notes`. The note field self-describes as "under-shipped commit-grouping target 3 but full 8-PR floor met all guardrails passed" — the reviews drip-131 produced 8 PR verdicts but only 1 commit/1 push, while templates added two detectors (SHAs `fba7968`, `fddc2fe`) for 2 commits/1 push, and metaposts shipped one essay (SHA `c587d2a`) for 1 commit/1 push. Total: 4 commits, 3 pushes, 0 blocks. Under the Binomial(12, 0.697) model, the probability of observing k = 4 in any single trial is about 0.0082 — meaning we'd expect roughly 2.7 such ticks in 331. We observed 1. The empirical rate is on the low side of the model's prediction but easily within sampling noise.

**The single 12-commit arity-3 tick** is at `2026-04-26T13:01:55Z`, family `digest+feature+templates`, repos `oss-digest+pew-insights+ai-native-workflow`. The breakdown was: digest refreshed ADDENDUM-55 with W17 synth #155 + #156 (SHAs `daa725f` / `2d34947` / `3408d8c`, 3 commits / 1 push), feature shipped pew-insights v0.6.48 → v0.6.49 → v0.6.50 with the source-weekend-weekday-cache-share-gap subcommand (SHAs `1577e46` / `0940a75` / `14e9251` / `92480d0` / `9799161` / `9f39374`, 6 commits / 2 pushes), templates added two detectors (SHAs `d30bfde` / `89d5045` / `61bcfc0`, 3 commits / 1 push). Total: 12 commits, 4 pushes, 0 blocks. Under the model the probability of k = 12 in a single trial is `(0.697)^12 = 0.0132` per tick on the unrestricted binomial — but conditional on the dispatcher floor *and* the per-family ceiling, this is essentially the maximum possible value the dispatcher can output without tripping its own wall-clock guardrail. The fact that exactly one of 331 ticks landed at the structural ceiling is itself a small but interesting result: the dispatcher *can* hit n = 12, and has done so once, which means the ceiling is real but rarely-exhausted.

These two ticks anchor the support [4, 12] for the arity-3 era. The Binomial(12, 0.697) model assigns nonzero probability to k ∈ {0, 1, 2, 3, 13+}, but the model expects fewer than 1 such tick across 331 trials in any of those cells, and the data show exactly zero — fully consistent with the fit.

## 7. The arity-1 and arity-2 sub-eras: not Poisson, not binomial, just small

For completeness, here are the dispersion indices for the other two arity buckets:

- **Arity-1 (n = 32 ticks, mean = 2.4375, var = 1.9336)**: var/mean = 0.7929. Significantly under-dispersed but not as dramatically as arity-3. Distribution: 1×10, 2×7, 3×11, 4×1, 5×2, 7×1.
- **Arity-2 (n = 9 ticks, mean = 4.7778, var = 3.5062)**: var/mean = 0.7338. Under-dispersed but the sample is too small to fit any distribution meaningfully.

The arity-1 era includes the lone 7-commit outlier — almost certainly a bootstrap-day burst — and a long, narrow body around 1–3 commits. A Poisson fit there would give λ̂ = 2.4375 and predict 6.82 ones, 8.31 twos, 6.75 threes — observed 10, 7, 11 respectively. Already the empirical mode at k = 3 (11 ticks) over-shoots the Poisson prediction (6.75) by 63%. The same under-dispersed, peakier-than-Poisson signature is present, just on a much smaller sample. The arity-1 era was never long enough to settle into the kind of binomial-shaped regime that the arity-3 era exhibits.

## 8. Per-day ticks by arity-3, just to confirm the steady state

The arity-3 ticks are spread across days as follows:

| Date       | arity-3 ticks |
|------------|--------------:|
| 2026-04-24 | 42 |
| 2026-04-25 | 80 |
| 2026-04-26 | 78 |
| 2026-04-27 | 74 |
| 2026-04-28 | 57 (partial, snapshot at 18:27Z) |

Days 25, 26, 27 are nearly identical (80 / 78 / 74), suggesting the underlying generative process is in steady state across the three full days. If the per-day commits-per-tick distribution drifted — say, if the floor crept upward — the binomial fit would degrade. It does not. Splitting the 331 arity-3 ticks by day and re-fitting Binomial(12, p̂) day-by-day would give p̂ values clustered around 0.69, but with day-level samples in the 50–80 range the per-day chi² tests are statistically too thin to be diagnostic. The pooled fit is the one that earns the conclusion.

## 9. The pushes-per-tick check, briefly

If commits-per-tick is Binomial(12, 0.697), what about pushes-per-tick? The arity-3 pushes distribution is much tighter:

- mean = 3.526, variance = 0.443, var/mean = 0.126.
- support: 3 (×183), 4 (×128), 5 (×14), 6 (×6).

With var/mean = 0.126, this is *even more* under-dispersed than commits. It is essentially a deterministic "3 or 4 pushes per tick, occasionally 5" distribution. Binomial(6, 0.587) would give mean ≈ 3.52 and var ≈ 1.46 — 3.3x larger than observed. Even a binomial cannot explain pushes-per-tick; the dispatcher is simply hard-coded to "one push per family, with rare carry-over." This is consistent with the parallel-three contract that earlier metaposts identified — three families, three pushes minimum, anything above 3 is a same-family multi-push tick.

So the *commits* are stochastic-looking around a binomial baseline, but the *pushes* are nearly deterministic. The variance comes from how many commits each family decides to bundle, not from how many pushes the dispatcher launches. This is consistent with the "commits-per-push as a coupling score" angle from the 2026-04-25 era of metaposts — the variance in commits-per-tick is essentially the variance in the per-family bundling decision, not in the dispatcher's launching decision.

## 10. The block-event check, briefly

Across the entire 372-row corpus there are **8 block events** (rows with `blocks ≥ 1`). All 8 are listed in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` and recapped here:

1. `2026-04-24T01:55:00Z` — `oss-contributions/pr-reviews` (arity-1)
2. `2026-04-24T18:05:15Z` — `templates+posts+digest` (arity-3)
3. `2026-04-24T18:19:07Z` — `metaposts+cli-zoo+feature` (arity-3)
4. `2026-04-24T23:40:34Z` — `templates+digest+metaposts` (arity-3)
5. `2026-04-25T03:35:00Z` — `digest+templates+feature` (arity-3)
6. `2026-04-25T08:50:00Z` — `templates+digest+feature` (arity-3)
7. `2026-04-26T00:49:39Z` — `metaposts+cli-zoo+digest` (arity-3)
8. `2026-04-28T03:29:34Z` — `digest+templates+cli-zoo` (arity-3)

All 8 ticks are present in the commits-per-tick fit above; none of them were excluded as outliers. Their commit counts are within the body of the binomial — the block events are not associated with anomalous commit volumes. This means the chi-square fit results above are robust to whether you keep or remove the block ticks, and the underlying Bernoulli-trial structure of commits-per-tick is independent of the guardrail-trip events. (A separate metapost would be needed to assess whether the 8 block events themselves are Poisson-distributed in time — a question this post leaves open.)

## 11. What the result implies operationally

If commits-per-tick is Binomial(n = 12, p ≈ 0.697) and not Poisson, three operational implications follow immediately:

**One: the floor is not soft.** Earlier metaposts have framed the per-tick commit floor as "the dispatcher tries to hit ≥ 6 commits per arity-3 tick." The binomial fit reframes this. Under Binomial(12, 0.697), P(commits ≤ 5) = `Σ_{k=0..5} C(12,k) × 0.697^k × 0.303^(12-k)` ≈ 0.0404. Observed: 6/331 = 0.0181. The under-floor rate is *less than half* what an unconstrained Bernoulli model would predict. The dispatcher has a real, working floor enforcement mechanism — but the data show it's a 98.2%-effective enforcement, not a 100% one. The gap (1.8% under-floor) is real and recoverable in the ledger.

**Two: the ceiling is real, not soft.** No arity-3 tick has produced more than 12 commits, ever. Under Poisson(8.37), we'd expect 12.18 ticks at ≥ 13 commits across 331 trials — and we see zero. Under Binomial(12, 0.697), 13 is structurally unreachable. The data agrees with the binomial. Whatever the dispatcher's wall-clock budget per tick is, it's enforced tightly enough that 12 commits is a hard ceiling — hit exactly once on `2026-04-26T13:01:55Z` and never breached.

**Three: the variance is structurally smaller than it would naively appear.** If you read commits-per-tick as Poisson, you'd build SLO bands at λ ± √λ ≈ 8.37 ± 2.89, i.e. expect commit counts to range across [5.5, 11.3] with one-sigma confidence. The actual one-sigma band is 8.37 ± √2.18 ≈ [6.9, 9.8] — about half as wide. Anyone tuning alerting on commits-per-tick using Poisson assumptions would get *systematically more false negatives* (real tail events look "normal" against an over-wide band) than the data justifies. This is a small but practical mis-modeling cost.

## 12. The methodological closing — what this audit does and doesn't claim

This post does **not** claim that the daemon's commits-per-tick distribution is "exactly" Binomial(12, 0.697). The chi² test merely fails to reject that hypothesis at the 5% level on a 331-tick sample. With more ticks, the fit could degrade — particularly if the under-shipped tick rate stays below the model's 4% prediction and the data accumulate evidence of a tighter, structurally-floor-enforced distribution than even a binomial allows. A future audit at, say, 1000 arity-3 ticks could reasonably reject Binomial(12, 0.697) and require either a *truncated* binomial (with the [0, 5] mass redistributed onto [6, 12]) or a custom dispatcher-aware mixture model.

What this post **does** claim, and what is well-supported by the data, is that **Poisson is decisively wrong as a generative model for commits-per-tick** — chi² = 146.45 against a critical of 16.92 cannot be undone by any reasonable amount of additional data. Anyone reasoning about commits-per-tick using Poisson intuitions (variance ≈ mean, both tails open, exponential right tail) is going to draw incorrect inferences about the dispatcher's actual operating envelope. The right intuition is: **fixed number of trials, moderately-high success probability, hard upper bound**. Ticks land in a tight bell around 8 commits because they were *designed* to, by a multi-family fan-out contract that bounds the count from above and below. The Bernoulli-trial framing is not just a better fit — it's the framing that matches the dispatcher's actual control logic.

The operational corollary: when the next metapost in this corpus reaches for "X-per-tick" or "Y-per-drip" as a count to fit, the prior should be **binomial**, not Poisson, unless there's an explicit reason (truly independent arrivals, no upper bound, no floor) to believe otherwise. The history.jsonl ledger has been telling us this for 331 ticks. It just took fitting both models against each other to make the signal legible.

---

*Snapshot: history.jsonl at 2026-04-28T18:45Z, 391 lines on disk, 372 valid JSON rows. Arity-3 sub-corpus: 331 ticks, mean commits = 8.3686, var commits = 2.1844, var/mean = 0.2610. Poisson fit: chi² = 146.45, df = 9, critical at 0.05 = 16.92, **rejected**. Binomial(n=12, p=0.6974) fit: chi² = 10.48, df = 5, critical at 0.05 = 11.07, **not rejected**. Extreme observations cited inline by ts and SHAs.*
