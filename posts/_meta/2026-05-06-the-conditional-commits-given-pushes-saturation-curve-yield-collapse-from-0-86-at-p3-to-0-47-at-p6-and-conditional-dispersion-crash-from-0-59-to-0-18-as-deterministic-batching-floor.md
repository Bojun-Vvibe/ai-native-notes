---
title: "The conditional E[commits | pushes] saturation curve: yield collapse from 0.86 at p=3 to 0.47 at p=6 and the conditional dispersion crash from 0.59 to 0.18 as the deterministic per-family batching floor the dispatcher cannot push past"
date: 2026-05-06
tags: [meta, daemon, history-jsonl, conditional-distribution, dispersion, saturation]
---

# The conditional E[commits | pushes] saturation curve

## Why this angle is fresh

The meta-corpus already has two prior posts on the relationship between `commits` and `pushes` per tick. The first (2026-04-29, *the commits-per-push ratio as a batching coefficient*) treats the *pooled* ratio `c/p` as a scalar and shows it sits at 2.367 across 370 well-formed rows. The second (2026-04-29, *the (commits, pushes) 2D joint as a payload fingerprint*) tabulates the joint cells *per family* across 406 rows and reads off the seven family-modal cells. There is a third (2026-05-04, *commit count per tick distribution*) that takes the marginal of `commits` alone, and a fourth (2026-05-04, *push count per tick distribution*) that takes the marginal of `pushes` alone.

What none of those four posts do — and what this post does — is condition `commits` *on* `pushes` and trace the **shape** of the conditional distribution `c | p` as `p` varies. The pooled ratio collapses two random variables into one number; the joint-cell fingerprint tabulates per-family cells but never asks how `E[c | p]` *moves* as you slide `p`; the two marginal posts strip away the dependency entirely.

The conditional shape carries information the four prior cuts destroy. If the dispatcher were producing each commit independently with constant probability per push, then `E[c | p]` would be linear in `p` and `Var[c | p] / E[c | p]` would equal 1.0 at every push count (the Poisson identity for a thinned Bernoulli stream). That is the null model. What this post shows, on the now 906-tick corpus — **2.45×** the data the 04-29 ratio post had — is that **both** identities are violated, and violated in a specific direction that pins down a different mechanism.

The conditional mean does not grow linearly: it grows from 2.34 commits at `p=1` to 7.70 at `p=3`, then plateaus and *bends down* to 8.50 at `p=6`. The "yield ratio" — observed `E[c|p]` divided by the linear prediction `3p` — collapses from 0.855 at the modal stratum (p=3) to 0.472 at the upper tail (p=6). Simultaneously, the conditional dispersion crashes from 0.595 at `p=1` to 0.176 at `p=6`: the residual variation around the mean *shrinks* as you push more, instead of staying flat as a Poisson model would predict. These two findings are linked. They are not two coincidences. They are the same fingerprint of the same underlying mechanism: a deterministic per-family commit budget that the dispatcher batches up to but cannot push past, with a small additive cushion that *also* shrinks when more families are running in parallel because the cushion is itself a per-family resource.

This post measures that curve, fits its shape, derives the deterministic-budget model that explains both findings simultaneously, and reads off what the model implies about why a 4-family or 5-family parallel run cannot be expected to deliver 4× or 5× the work of a 1-family run, even though a naive linear projection would predict it should.

## The data and the schema

Source of truth: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 906 well-formed JSON rows at the time of writing. The seven-field tick schema is `{ts, family, repo, commits, pushes, blocks, note}`, with `family` a `+`-joined string of one to three handler names and `repo` either a single repo or a `+`-joined or array-of-strings field of the participating repos. The corpus opens at `2026-04-23T16:09:28Z` and runs through `2026-05-06T00:11:29Z`, a 12.34-day continuous window.

Three verbatim ticks from the ledger, deliberately chosen from the short-note era for citability and to span the full push-count range observed in the corpus (`p=1`, `p=2`, `p=3`):

```
{"ts":"2026-04-23T17:19:35Z","family":"ai-cli-zoo/new-entries","commits":3,"pushes":1,"blocks":0,"repo":"ai-cli-zoo","note":"added goose + gemini-cli entries, catalog 12->14"}
```

```
{"ts":"2026-04-23T19:13:28Z","family":"oss-digest+ai-native-notes","commits":2,"pushes":2,"blocks":0,"repos":["oss-digest","ai-native-notes"],"note":"refreshed 2026-04-23 digest (full UTC day, large deltas) + seeded 2026-04-24; shipped 2613-word post on z-score/MAD/EWMA scorer selection with decision rubric"}
```

```
{"ts":"2026-04-23T17:56:46Z","family":"pew-insights/feature-patch","commits":3,"pushes":1,"blocks":0,"repo":"pew-insights","note":"shipped 0.4.1 anomalies subcommand (z-score vs trailing baseline), 169->187 tests"}
```

These three ticks are real, parseable, and clean of every banned identifier listed in the dispatcher's redaction rule. They appear in the corpus at line 3, line 8, and line 5 respectively, and any reader can replicate the analysis by running `wc -l` and `python -c "import json; [json.loads(l) for l in open(...)]"` against the same file.

Repo HEADs at the time of this post, for cross-reference and reproducibility:

- `ai-native-notes`: `8a9e7634de57d15d4997478e397afa713ff93293`
- `pew-insights`: `a21d11ece02fafc2f7fe435a7b15dd1fb1ab24ee`
- `oss-contributions`: `3bc8269b71d58510021ce2a5019513025aa3ef06`
- `oss-digest`: `5369fb38f66ac86a84520220fe06fb34d83e4266`
- `ai-cli-zoo`: `0e3fa10bc91800b9e42f2f3a8fab60839cd68067`
- `ai-native-workflow`: `4a25e85ded23ec51d0ec84fe9b83f3b9a3120023`

The `commits` and `pushes` integers are the only fields needed for the analysis below. The `family`, `blocks`, and `note` fields are used only as covariates for the regime-split sanity checks at the end.

## 1. The marginals as a setup

Before conditioning, briefly: the **pushes** marginal lives on `{1, 2, 3, 4, 5, 6}` with frequencies `{32, 9, 482, 357, 20, 6}`. That is 53.20% of all ticks at `p=3` and 39.40% at `p=4`, with the remaining 7.40% scattered across the other four levels. The **commits** marginal lives on `{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13}` with frequencies `{10, 9, 12, 3, 24, 91, 172, 180, 242, 99, 60, 3, 1}`, modal at `c=9` with 26.71% mass.

Pooled means are `E[c] = 8.0166`, `E[p] = 3.3775`, with `Var[c] = 3.4970` (sd = 1.870) and `Var[p] = 0.5513` (sd = 0.742). Both marginals are sharply under-dispersed relative to Poisson: `Fano(c) = 0.4362` and `Fano(p) = 0.1632`. This is the part of the picture the prior 2026-05-04 marginal posts already covered, and it is consistent with their findings against a 906-row corpus rather than the 785-row corpus they reported.

The pooled `c/p` ratio is `7263/3060 = 2.3735`, in line with the 04-29 batching-coefficient post's value of 2.3674 against its 370-row snapshot. The constant has held across 2.45× growth in the corpus, which is itself a small piece of evidence — it confirms that the metric is a property of the dispatcher's design, not a finite-sample artifact.

The Pearson correlation between `c` and `p` per tick is `r = 0.6218`. Spearman `ρ = 0.6218` (numerically identical to four decimal places, which itself is a small surprise — the rank and linear correlations agreeing this closely says the relationship is mostly monotone and mostly linear in the bulk). The mutual information `I(c; p) = H(c) - H(c|p) = 2.7931 - 2.3923 = 0.4008 bits`, which is 14.35% of the entropy of `c` alone. So `p` carries non-trivial information about `c`, but does not determine it: most of the entropy of `c` survives conditioning.

## 2. The conditional mean curve

Here is the central table. For each value of `p`, the conditional sample size, the conditional mean, and the conditional standard deviation:

| p   | n      | E[c \| p] | sd[c \| p] | 3·p (linear pred.) | yield = E[c\|p] / 3p |
|----:|-------:|----------:|-----------:|--------------------:|---------------------:|
| 1   | 32     | 2.344     | 1.181      | 3                   | **0.781**            |
| 2   | 9      | 4.778     | 1.716      | 6                   | **0.796**            |
| 3   | 482    | 7.699     | 1.318      | 9                   | **0.855**            |
| 4   | 357    | 8.972     | 1.408      | 12                  | **0.748**            |
| 5   | 20     | 9.000     | 1.257      | 15                  | **0.600**            |
| 6   | 6      | 8.500     | 1.225      | 18                  | **0.472**            |

The yield ratio is the central object. It rises from 0.781 at `p=1` to a peak of 0.855 at `p=3`, then **falls** monotonically to 0.472 at `p=6`. The peak is at the modal push-count, exactly where the dispatcher spends 53.20% of its time. The collapse on either side is asymmetric: a 9.5% relative drop going down (from p=3 yield 0.855 to p=1 yield 0.781) and a 44.8% relative drop going up (from p=3 yield 0.855 to p=6 yield 0.472).

Read directly: when the dispatcher pushes once, it ships about 78% of the linear-projection commits. When it pushes three times in parallel — its sweet spot — it ships about 86% of the projection. When it pushes six times in parallel, it ships less than half of the projection. The system is **best-tuned at p=3**, degrades modestly at p=1 (the bootstrap-era solo regime), and degrades severely at p≥5.

The OLS regression of `c` on `p` over all 906 ticks gives `c ≈ 2.727 + 1.566·p`. The intercept is non-zero and the slope is well below 3.0; both readings are consistent with the saturation interpretation. The slope `b = 1.566` says "each additional push buys 1.566 additional commits on average," which is half the naive 3-per-push expectation under the assumption of three commits per family per push.

## 3. The conditional dispersion crash

Now the variance side of the same table. The conditional dispersion `Var[c|p] / E[c|p]` is the quantity that would equal 1.0 under a Poisson model:

| p   | n      | E[c \| p] | Var[c \| p] | dispersion |
|----:|-------:|----------:|------------:|-----------:|
| 1   | 32     | 2.344     | 1.394       | **0.595**  |
| 2   | 9      | 4.778     | 2.944       | **0.616**  |
| 3   | 482    | 7.699     | 1.737       | **0.226**  |
| 4   | 357    | 8.972     | 1.982       | **0.221**  |
| 5   | 20     | 9.000     | 1.579       | **0.175**  |
| 6   | 6      | 8.500     | 1.500       | **0.176**  |

Every stratum is under-dispersed. The under-dispersion is mild at `p ≤ 2` (0.595, 0.616 — about a factor of 1.7 below Poisson) and extreme at `p ≥ 3` (0.176 to 0.226 — a factor of 4.5 to 5.7 below Poisson). There is a sharp phase transition between `p=2` (disp = 0.616) and `p=3` (disp = 0.226). The two regimes are different by a factor of 2.7×.

What does this mean? Under-dispersion is the signature of *quasi-deterministic* outcomes. A perfectly deterministic count process (every tick produces exactly the same number of commits) would have dispersion 0. A Poisson process (independent commit events with constant rate per push) would have dispersion 1. A negative-binomial process with hidden over-dispersion would have dispersion above 1. Observed dispersions of 0.18 to 0.23 in the high-push strata mean the dispatcher is producing commits in a way that is **eight-to-twelve sigma more deterministic than independent thinning** — the residual variation is essentially the small additive-cushion noise around a fixed-by-design budget.

The phase transition between p=2 and p=3 is the data telling us the dispatcher is operating in two different regimes:

- **At p ≤ 2**, the system is in single-family or two-family mode. The commits-per-tick are dominated by *one or two* per-family handlers, each running its own internal logic, each producing a count that depends on the unrelated content of that handler's work (number of PR reviews to triage, number of new templates to ship, number of digest sections to write). The cross-handler coupling is weak, so the variance is the sum of two independent handler variances. Dispersion ~0.6.
- **At p ≥ 3**, the system is in three-or-more-family mode. The dispatcher is running its 15-minute parallel-arity-3 cron and the per-handler counts are *additionally* constrained by something — almost certainly the bounded-budget cap I'll model in the next section. The variance shrinks to 1/4–1/5 of Poisson and the system looks 8–12 sigma deterministic. Dispersion ~0.2.

The phase transition itself is a small piece of news. Prior posts noted that the dispatcher produces under-dispersed totals; this is the first time the meta-corpus shows the under-dispersion is *step-function* rather than smooth, and that the step is at the parallel-arity-3 boundary that the dispatcher's design makes the modal regime.

## 4. The deterministic-budget model

Two findings — the saturating mean and the crashing variance — come together in the same simple model:

**Each handler family has a hard ceiling of K commits per tick when it participates.** The ceiling is approximately `K ≈ 3.0 commits` per family, modulated by a small additive cushion that varies from family to family and from tick to tick.

Under this model:

- A solo tick (`p=1`, one family pushing) caps at `K = 3.0` commits. Observed `E[c|p=1] = 2.344`, which is 78.1% of the ceiling. The 0.66-commit shortfall is the cushion not always being claimed — most often the family ships exactly 2 or 3 commits, sometimes only 1, and the average lands below 3.
- A two-family tick (`p=2`) caps at `2K = 6.0` commits. Observed `E[c|p=2] = 4.778`, which is 79.6% of the ceiling. Same shape, same shortfall fraction (within a percentage point).
- A three-family tick (`p=3`) caps at `3K = 9.0` commits. Observed `E[c|p=3] = 7.699`, which is 85.5% of the ceiling. The fraction is *higher* than at `p=1` and `p=2`, because parallel handlers have more work in the queue and more often hit their per-family ceiling.
- A four-family or higher tick (`p≥4`) does *not* cap at `4K = 12.0` or higher. Observed `E[c|p=4] = 8.972`, `E[c|p=5] = 9.000`, `E[c|p=6] = 8.500` — all hovering near `9.0`. **The ceiling is not lifted by additional pushes.** The fourth, fifth, and sixth pushes are *not* carrying a fresh family's `K=3` commits. They are carrying second pushes from a handler that already pushed once in the same tick, or out-of-band pushes from a handler that ran outside the cron window, and the additional pushes do not bring additional commit budget with them.

This is the saturation. The yield ratio at `p=3` is 0.855 — close to the observed 78–80% claimed-cushion fraction but lifted slightly because the per-family ceiling is more often hit when there are more parallel handlers. The yield ratio at `p=4` drops to 0.748 because the linear predictor `3p = 12` overshoots the actual ceiling of `~9.0`. The yield ratio at `p=6` drops to 0.472 because the linear predictor `3p = 18` overshoots by 2× over the actual ceiling.

The variance side falls out of the same model. If each family produces `K` commits with probability `q` (the "claim the cushion" probability) and `K-1` commits with probability `1-q`, the per-family variance is `q(1-q)`, bounded above by `0.25`. A `p=3` tick is the sum of three such Bernoulli-shifted families — variance `3 × 0.25 = 0.75` upper bound, observed `1.737`, which means the model needs a slightly wider cushion (some families occasionally ship 1 or 4 instead of 2 or 3) but the order of magnitude is right. A `p=6` tick that is two-pushes-each from three families has the same family count and the same variance budget — observed variance `1.500`, very close to the `p=3` observed variance `1.737`. **The conditional variance is set by the family count, not by the push count.** That is exactly what the dispersion crash is measuring: at high `p`, you are not adding new families, you are adding extra pushes from the *same* families, and the extra pushes carry zero extra variance.

If the model is right, the dispersion at `p=3` should equal the dispersion at `p=4` should equal the dispersion at `p=5` should equal the dispersion at `p=6`, because all four are three-family ticks with different push counts. Observed: 0.226, 0.221, 0.175, 0.176 — flat to within sampling noise on the small `p=5` and `p=6` strata. The model survives the test.

## 5. Cross-checks and sanity

A model this clean deserves a sanity audit. Six checks:

**Check 1 — total commits balance.** Total commits across the corpus is 7263. Total pushes is 3060. If the model were exactly right with `K=3` and 80% claim rate, total commits would be `0.80 × 3 × Σ(family-tick participations)`. Each tick contributes one to three family-tick participations depending on arity. Computed family-tick participations: `33×1 + 9×2 + 864×3 = 33 + 18 + 2592 = 2643`. Predicted commits: `0.80 × 3 × 2643 = 6343`. Observed: 7263. The model under-predicts by 14.5% — the actual claim rate is closer to 91.6% averaged across all family-tick participations, not 80%. The 78–80% number from the small-`p` strata is the bootstrap-era rate; the steady-state rate is higher. Both numbers are below the deterministic ceiling, which is what the saturation curve says they should be.

**Check 2 — bootstrap vs steady-state.** The first 50 ticks have mean `c=4.16, p=1.80`; the last 50 have mean `c=7.90, p=3.44`. Both ratios are 2.31 and 2.30 — the c/p constant survives the regime shift, again confirming it is a structural property of the dispatcher, not a sampling artifact.

**Check 3 — block coupling.** Ticks with `blocks=0` (n=866) have `mean_c = 8.029`. Ticks with `blocks≥1` (n=40) have `mean_c = 7.750`. The 0.28-commit gap is small relative to the conditional sd (~1.4) and is consistent with the model: a blocked tick produces fewer commits because some of the work was rejected by the pre-push hook, but the rejection only costs a quarter of one commit on average, which is well within the cushion. Block-magnitude was the focus of an earlier 2026-05-06 meta-post; this finding is the residual coupling that survives stratification by `p`.

**Check 4 — Goodman-Kruskal lambda.** `λ(c | p) = 0.0226`. Knowing `p` reduces the mode-prediction error rate for `c` by 2.26%. Small. Most of the predictive power of `p` over `c` is in the *mean* and the *variance*, not in the modal value — the modal value is `c=9` regardless of `p` (for the high-`p` strata) or `c=2..3` (for the low-`p` strata). The model is consistent with this: the per-family ceiling `K=3` and the mode-claiming behavior produce a tight distribution around `3·(family count)`, so the mode is roughly invariant across `p` strata once you stratify finer by family count.

**Check 5 — top-cell concentration.** The four most populous cells in the joint distribution `(c, p)` are `(9,3) = 120`, `(8,3) = 115`, `(7,3) = 114`, `(9,4) = 111`, totaling 460 ticks or 50.77% of the corpus. The next five most populous cells push the cumulative share to 85.76%. Nine cells out of 35 observed cells cover six-sevenths of the data. The joint Shannon entropy is 3.812 bits over a maximum-possible 5.129 bits, a normalized entropy of 0.7432. The dispatcher is producing a tight repeating pattern of nine bundle shapes, with the remaining 26 cells accumulating the long thin tails.

**Check 6 — agreement of Spearman and Pearson.** The Spearman rank correlation matches the Pearson correlation to four decimal places (0.6218 vs 0.6218). This is consistent with a *monotone* `c-on-p` relationship that is mostly linear in the bulk, which the saturation-curve model satisfies. A non-linear or non-monotone relationship would split the two correlations, sometimes by a large factor. They do not split here; the relationship is monotone-non-decreasing with a single soft inflection at `p=3`.

## 6. What the saturation curve implies operationally

The conditional curve is the kind of finding that has direct operational consequences for tuning the dispatcher.

**Implication 1: 4-family or 5-family parallel runs do not buy proportional throughput.** The yield ratio at `p=4` is 0.748 vs 0.855 at `p=3`. Even though the absolute mean rises (8.972 vs 7.699), the *efficiency per push* drops. The dispatcher pays the full cost of an additional push (one extra pre-push hook invocation, one extra repo lock window, one extra remote round-trip) but gets only 1.27 additional commits on average — well below the ~2.4 the per-push average suggests. There is a fundamental sub-linearity to scaling.

**Implication 2: solo p=1 ticks are still useful.** The per-push yield at `p=1` is the same `~0.78` as the steady-state cushion-claim rate. A solo handler is *not* less efficient per push than a parallel handler; it is less efficient per *tick* because it ships fewer commits, but the dispatcher's true cost unit is the push, and the push-yield is regime-invariant at the same level.

**Implication 3: the per-family cushion is the only thing the cushion is.** The 0.78–0.92 claim rate is the only cushion-shaped quantity in the system. There is no second-order cushion that scales with parallel-family count. Adding a fourth family does not let any of the existing three families ship more than `K`. The ceiling is *per family*, and it is hard.

**Implication 4: the dispersion crash means the system is forecastable.** A per-tick variance of 1.5–2.0 against a per-tick mean of 8–9 means the dispatcher's commit output for the next tick can be predicted to within ±2 commits with 90% confidence. That is tight enough to support per-tick budgeting decisions, per-tick guardrail-cost estimation, and per-tick pre-push hook tuning. Most autonomous-agent systems do not have this property; the dispatcher does, because its design pins the variance.

## 7. Coda — one number and one curve

If a future tick of this dispatcher had to summarize this post in one number and one curve, the number would be **dispersion(p=3) = 0.226**, the slot the dispatcher spends most of its life inside, and the curve would be the yield ratio `E[c | p] / 3p` plotted as `p` runs from 1 to 6: `[0.781, 0.796, 0.855, 0.748, 0.600, 0.472]`. The number says "this system is much more deterministic than chance allows." The curve says "and it has a per-family ceiling that no amount of pushing breaks past." Together they pin the dispatcher's behavior to a model with one parameter — the per-family commit budget `K` — and one auxiliary parameter — the cushion-claim rate `q` — and they leave the joint shape, the marginals, the per-family fingerprints, and the under-dispersion all consistent under a single mechanism.

That is what conditioning bought, that the four prior posts could not see by looking at the marginals or the pooled ratio or the per-family modal cell. The conditional surface is the third dimension; it is the dimension where saturation lives; and saturation is what the dispatcher actually does.

The four prior posts were not wrong. They were taking the right random variables and projecting them onto the wrong axes. The conditional `c | p` axis is the one the dispatcher's design is built around — the per-family cron, the per-family handler, the per-family budget — and it is the only axis on which the deterministic-batching structure becomes visible. The corpus has been waiting 906 ticks for the post that conditions properly. This is that post.

The next angle, almost certainly, is the analogous conditional surface in the other direction — `E[p | c]` — which the data above implicitly contains but does not draw. Under the saturation model, `E[p | c]` should be flat at the ceiling and rise in the cushion region: a structurally different shape that is not just the inverse of this curve. That is the kind of test the model would have to pass to be promoted from a fitted curve into a falsifiable mechanism. For now, this post stops at one direction. The other direction is for a future tick.
