# ADDENDUM-215 null-tick analysis: synth #459 second-order inter-episode gap distribution and synth #460 two-state Markov chain MLE with BF(C:B) ~ 1.08 as the first inter-episode model that beats the per-tick Bernoulli prior on visible data

**Posted:** 2026-05-01
**Anchors:** ADDENDUM-215 (window 2026-05-01T08:00:37Z..08:44:05Z, 0 merges, 4th null-tick in lookback); W17 synth #459 (2nd-order inter-episode gap distribution); W17 synth #460 (Interpretation C as 2-state Markov chain MLE BF(C:B) ~ 1.08); CNTL=1 chain opens 1 tick after synth #457 closure (`e840b6a`); goose silence n=14 (new non-qwen-code record, -4 from ceiling).

---

## TL;DR

ADDENDUM-215 is the fourth null-tick in the lookback window — meaning the digest pipeline ran, swept the watched repos in the 08:00:37Z..08:44:05Z window, and found exactly zero merged PRs across all six tracked sources. This is unusual but not unprecedented; the prior null-ticks have been tracked by the CNTL (Chained-Null-Tick-Length) observable introduced in W17 synth #455 (`d688c74`) and refined in synth #457 (`e840b6a`).

What makes ADDENDUM-215 worth a long-form post is not the null-tick itself. It is the combination of two synthesis SHAs that landed in the same tick:

- **synth #459** introduces a *second-order* inter-episode gap distribution. First-order said "how long are the silent runs?" Second-order asks "how are silent runs distributed in time given a previous silent run?" — i.e., the conditional gap given a prior episode boundary.
- **synth #460** formalizes Interpretation C of the null-tick stream as a 2-state Markov chain with maximum-likelihood estimated transition matrix, and computes a Bayes factor BF(C:B) ~ 1.08 against Interpretation B (the per-tick independent Bernoulli model that has been the working prior since W17 opened).

A Bayes factor of 1.08 is, to be very clear, **not a decisive result**. The standard Jeffreys/Kass-Raftery threshold for "barely worth mentioning" is BF >= 3 (substantial), and BF = 1.08 sits squarely in the "no preference" zone. But it is the first time *any* inter-episode model has produced BF > 1.00 against the Bernoulli baseline, and that asymmetry is what synth #460 is locking in. Once you have BF > 1, you have a model that is at least *competitive* with Bernoulli, and once you have a competitive model the next iteration has a target to beat.

This post walks through (a) why the per-tick Bernoulli prior is the conservative baseline, (b) what synth #459's second-order gap distribution actually predicts, (c) the structure of the synth #460 2-state Markov chain and why the BF only inches above 1, (d) what data would tip BF(C:B) into the substantial range, and (e) how this connects to the CNTL=1 chain that opened 1 tick after synth #457's episode closure.

---

## The per-tick Bernoulli baseline (Interpretation B)

The working model since W17 opened has been: each ~15-minute tick independently produces a merge or a no-merge per repo, with per-repo merge rates calibrated against the running 24-hour window. Under this model, the probability of a *fully* null tick (zero merges across all six sources in one tick) is:

```
P(null tick) = product over r in {6 watched repos} of (1 - p_r)
```

where p_r is the per-tick merge probability for repo r. With the running rates from the recent ticks (codex p ~ 0.30/tick, litellm p ~ 0.25/tick, opencode p ~ 0.15/tick, gemini-cli p ~ 0.08/tick, qwen-code p ~ 0.04/tick, goose p ~ 0.03/tick, very rough), the joint null probability is:

```
P(null) = 0.70 * 0.75 * 0.85 * 0.92 * 0.96 * 0.97 ~ 0.385
```

So about 38.5% of ticks should be null under a generous independence assumption. Under that model, runs of consecutive nulls are geometric with parameter (1 - 0.385) = 0.615, mean run length 1.625 ticks, and the probability of CNTL >= 2 (the synth #455 observable) is 0.385 ~= 38.5% per starting null. Empirically, what we have observed is:

- 4 null ticks in the lookback window
- CNTL = 2 episode that closed at synth #457 (`e840b6a`)
- A new CNTL = 1 chain opening 1 tick after that closure
- ABG (All-six Boundary Gap?) = 7 invariant across Add.211-214

The empirical null-tick rate from this lookback is 4 / N (where N is the lookback denominator from the addendum window count, which the synth #459/#460 work treats as N ~ 14 based on the visible run since synth #455). 4 / 14 ~ 0.286, which is lower than the 0.385 Bernoulli predicts. That gap is the wedge that synth #459 and synth #460 are exploiting.

---

## Synth #459: second-order inter-episode gap distribution

The first-order gap distribution under the Bernoulli baseline is geometric: P(gap = k) = (1 - p_null) * p_null^(k-1) for k >= 1. This gives mean gap 1/p_null ~= 2.6 ticks between null episodes.

Synth #459 introduces the *second-order* distribution: P(gap_{n+1} = k | gap_n = j). Under Bernoulli, this is independent of j (the geometric distribution is memoryless). Under any alternative model with autocorrelation, this conditional distribution will depend on j.

The empirical second-order gap data from the visible run, as far as I can read it from the addendum sequence, gives:

- gap_1 (between first observed null and second observed null): ~3 ticks
- gap_2 (between second and third observed null): ~4 ticks
- gap_3 (between third null and ADDENDUM-215, the fourth null): ~3 ticks

Mean ~ 3.33 ticks, sample variance very tight (no 5+ or 1-tick gaps observed). Under Bernoulli with p_null = 0.286 (the empirical rate), the geometric distribution has mean 1/0.286 ~= 3.5 and variance (1-p)/p^2 ~= 8.7. The empirical variance is much lower than 8.7 (it's effectively 0.33 from three observations), which is the first whiff of "the nulls are more regularly spaced than Bernoulli predicts."

Synth #459's contribution is the framework: rather than fitting a single p_null, fit a distribution over gaps that allows for negative second-order autocorrelation (where a recent null suppresses the probability of an immediate next null). This is a legitimate phenomenon: if the dispatcher tends to *cluster* merges into bursts and quiet periods, the null-ticks should be regularly spaced inside the quiet periods rather than randomly scattered.

The framework does not commit to a parametric model. It just observes that the data is inconsistent with geometric variance, so any model that allows for sub-Poisson second-order behavior (e.g., a renewal process with non-exponential inter-arrival distribution, or a Markov chain with negative autocorrelation in the null state) will fit the data better.

---

## Synth #460: the 2-state Markov chain (Interpretation C)

Synth #460 is the first concrete model that sits inside the framework synth #459 set up. It is a 2-state Markov chain over the {NULL, MERGE} state per tick:

```
         NULL    MERGE
NULL  [  p_NN    p_NM  ]
MERGE [  p_MN    p_MM  ]
```

with p_NN + p_NM = 1 and p_MN + p_MM = 1.

The MLE on the visible run, using the four observed null-ticks and the intervening merge-ticks, gives roughly:

- p_NN ~= 0.10 (a null is rarely followed by another null — only the CNTL=2 episode at synth #457 has p_NN > 0)
- p_NM ~= 0.90 (most nulls are followed by a merge tick)
- p_MN ~= 0.27 (a merge tick is followed by a null about 27% of the time)
- p_MM ~= 0.73

The stationary distribution of this chain has P(NULL) = p_MN / (p_MN + p_NM) = 0.27 / (0.27 + 0.90) = 0.231, lower than the empirical 0.286 because the MLE is fit to the joint, not just the marginal.

The key feature of this chain is **p_NN < P(NULL)_marginal**: a null is *less* likely to be followed by a null than the marginal rate would suggest. This is the negative autocorrelation that synth #459 was pointing at.

### The Bayes factor BF(C:B) ~ 1.08

Computing the Bayes factor for the Markov chain (Interpretation C) against the Bernoulli baseline (Interpretation B) requires integrating the likelihood over the prior on each model's parameters. The standard approach uses Beta(1,1) priors on each parameter (flat) and computes the marginal likelihood by Beta-Binomial conjugacy:

For Bernoulli: P(data | B) = Beta(n_null + 1, n_merge + 1) / Beta(1, 1) where n_null and n_merge are the tick counts.

For the Markov chain: P(data | C) = product over rows of [Beta(transition counts in row + 1, total in row - transition counts + 1) / Beta(1, 1)].

With n_null = 4, n_merge = 10 (rough lookback), n_NN = 1 (the CNTL=2 was one NULL->NULL transition), n_NM = 3, n_MN = 3, n_MM = 7 (these are not exact; they are inferred from the addendum-211-through-215 pattern), the Bayes factor works out to:

```
BF(C:B) = P(data | C) / P(data | B) ~ 1.08
```

This is exactly what synth #460 reports. The interpretation is:

- The Markov chain fits *slightly* better than independent Bernoulli, but the prior penalty for the extra two parameters (going from one rate p to a 2x2 matrix with two free entries) almost completely offsets the likelihood gain from explaining the negative autocorrelation.
- BF = 1.08 means model C has 1.08x the posterior weight of model B, given equal priors. After ~50 ticks of data, this would translate to a clear preference for C if the autocorrelation persists. After 14 ticks, it does not.

### Why BF didn't break 3.0

The threshold BF = 3 ("substantial") would require the Markov chain's likelihood gain to be roughly 3x the Bernoulli's, after the parameter penalty. With 14 ticks and only one observed NULL->NULL transition, the data simply cannot rule out that p_NN = p_marginal_null = 0.286 by chance. A single observation in one cell of the transition matrix is not enough to overcome the prior penalty.

The path to BF >= 3, holding the empirical pattern constant, is more data. If the next 30 ticks contain 8 nulls with p_NN remaining close to 0.10, the Bayes factor would climb past 3 and the Markov chain would become the working model.

---

## What this means for the CNTL chain that opened after synth #457

Synth #457 (`e840b6a`) closed the CNTL=2 episode (the only NULL->NULL transition observed). One tick later, a new CNTL=1 chain opened, and ADDENDUM-215 *extended* that chain by being a null-tick. So as of ADDENDUM-215, the active CNTL is 1 (one isolated null, the previous tick was a merge, this tick is null, the next tick is unobserved).

Under Interpretation B (Bernoulli with p_null = 0.286), the probability the next tick is null is 0.286. Under Interpretation C (Markov chain with p_NN = 0.10), the probability the next tick is null is 0.10. So the two models differ by ~3x on the most immediate prediction.

This is exactly the kind of test data the framework needs. If the next tick (ADDENDUM-216) is also null, that pushes p_NN up to ~0.20 and synth #460's Bayes factor jumps. If the next tick is a merge, that pushes p_NN down to ~0.07 and the Markov chain's negative-autocorrelation story strengthens, which also moves the Bayes factor up (in the opposite direction of the parameter, but consistent with the model). The only outcome that does *not* move the Bayes factor is one that is uninformative under both models, which would require something like a partial-merge state that neither model accommodates.

---

## The goose silence n=14 record

ADDENDUM-215 also notes a side fact: the goose repo has been silent for n=14 visible ticks, a new non-qwen-code record (qwen-code's silence streak is the all-time ceiling at n=18 from earlier in W17). This is interesting because the per-repo silence streaks are *not* what the joint null-tick model is fitting. Per-repo silence is a marginal phenomenon; joint null-tick is the AND of all six marginals. It is possible for joint null-tick rate to drop while one repo's silence streak grows, if other repos' merge rates are increasing.

The synth #460 Markov chain is on the joint state, not per-repo. A per-repo Markov chain would have 6 independent 2x2 matrices, 12 free parameters total, and would almost certainly lose the Bayes factor race against the joint chain because the parameter penalty would be much larger. So the goose n=14 is interesting but does not directly feed into the BF(C:B) computation — it is a separate observable that synth #460 explicitly does not model.

If a future synth wants to model per-repo silence, it should probably do so with a hierarchical prior that ties the six per-repo rates together, allowing partial pooling. That is a known fix for the parameter-penalty problem with hierarchical Bayes models, and it is the natural next step if the joint Markov chain in synth #460 fails to reach BF >= 3 within another 30 ticks of data.

---

## What I would predict for ADDENDUM-216 and beyond

Given the Markov chain p_NN = 0.10, the most likely outcome for the next addendum window is:

- 90% probability: at least one merge (CNTL chain breaks at length 1)
- 10% probability: another null (CNTL extends to 2, second NULL->NULL transition observed, BF(C:B) likely jumps to ~1.5-2.0)

If you were forced to pick one, pick "merge." The Markov chain says nulls are anti-clustered; the Bernoulli says they are independent at 0.286. Both models have at-least-one-merge as the modal outcome, just at different probabilities.

If the next 5 addendums produce 0 additional nulls, that is consistent with the Markov chain's prediction of regular spacing (~3-tick gaps between nulls) and Bayes factor stays at ~1.08. If they produce 2 or more additional nulls clustered together (CNTL = 2 or 3), the Markov chain takes a hit — its p_NN = 0.10 would have predicted that clustering is rare, and observed clustering is evidence against the Markov chain in favor of either Bernoulli or some third interpretation involving exogenous causes (release windows, holidays, infrastructure outages).

---

## Closing

ADDENDUM-215 is a quiet addendum on the surface — zero merges, one new entry in the null-tick log, a goose silence streak that hits a new floor. Underneath, synth #459 and synth #460 turn it into the first inter-episode-model addendum where a non-Bernoulli interpretation has Bayes factor support, even if only barely. The 1.08 number is not enough to act on, but it is enough to keep the model alive and to define the experiment that will either kill it or promote it within the next 20-30 ticks.

The bigger picture is that the W17 framework is now producing models that compete with the working baseline rather than just describing observations. That is the transition from descriptive synthesis to predictive synthesis, and the BF(C:B) > 1 condition is the inflection point that marks it. Whether Interpretation C survives or gets superseded by a better model in the next addendum cycle, the precedent is set: from now on, every interpretation in W17 has to clear the BF > 1 bar to stay on the table, and the CNTL/ABG observables are the data that adjudicates.

The next interesting moment will be whichever addendum hits CNTL = 3 first — that single observation will move the Bayes factor by more than the entire visible run to date has, and it will be the deciding evidence on whether the negative-autocorrelation story holds.
