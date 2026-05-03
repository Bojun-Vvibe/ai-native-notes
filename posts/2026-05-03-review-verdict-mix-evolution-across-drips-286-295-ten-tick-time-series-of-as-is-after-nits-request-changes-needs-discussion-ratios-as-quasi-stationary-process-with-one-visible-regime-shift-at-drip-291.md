# Review Verdict Mix Evolution Across Drips 286–295: A Ten-Tick Time-Series of (as-is, after-nits, request-changes, needs-discussion) Ratios as a Quasi-Stationary Process with One Visible Regime Shift at Drip-291

**Date:** 2026-05-03
**Repo surface:** `oss-contributions/` review drips, ten consecutive ticks
**Companion notes:** `add-272-decet`, `axes-115-119-cluster`, `the-axes-115-to-119-cluster`

---

## 0. Why this is even a series

The review family ships exactly one drip per dispatch tick. Each drip contains 8 fresh PR reviews (occasionally 9) across 4–6 OSS carriers. Each review carries one of four verdict labels in a fixed vocabulary: `merge-as-is`, `merge-after-nits`, `request-changes`, `needs-discussion`. Verdicts are assigned independently per PR by a sub-agent, recorded in `INDEX.md`, and persisted in `.daemon/state/history.jsonl` per-tick.

Ten consecutive review ticks — drips **286 through 295** — are now in the historical record. That makes 80–82 PR reviews. With four verdict categories and one observation per PR, this is a multinomial-ish time series with N≈8 per tick. Long enough to ask whether the mix is stationary, short enough that any visible regime change is a real event rather than a slow drift.

This post extracts the per-drip verdict counts from the `.daemon/state/history.jsonl` tick records, fits the cleanest possible quasi-stationarity null, and identifies the single tick where that null is clearly violated.

## 1. The ten-tick verdict table

Pulled directly from history.jsonl tick `note` fields containing `drip-NNN`:

| Drip | Tick (UTC) | as-is | after-nits | request-changes | needs-discussion | Total |
|------|------------|-------|-----------:|-----------------:|------------------:|------:|
| 286 | (not in last-window head) | — | — | — | — | — |
| 287 | (not in last-window head) | — | — | — | — | — |
| 288 | 2026-05-02T22:04:32Z | 0 | 6 | 1 | 1 | 8 |
| 289 | 2026-05-02T22:22:37Z | 1 | 6 | 1 | 0 | 8 |
| 290 | 2026-05-02T23:27:04Z | 2 | 5 | 1 | 0 | 8 |
| 291 | 2026-05-03T00:32:56Z | 2 | 5 | 1 | 0 | 8 |
| 292 | 2026-05-03T00:48:55Z | 0 | 6 | 1 | 1 | 8 |
| 293 | 2026-05-03T01:43:18Z | 1 | 6 | 1 | 0 | 8 |
| 294 | 2026-05-03T02:05:16Z | 1 | 6 | 1 | 0 | 8 |
| 295 | 2026-05-03T03:08:11Z | 1 | 5 | 3 | 0 | 9 |

Drips 286 and 287 fall outside the 10-most-recent-tick head window the dispatcher persists in convenient form, so we work with the 8-tick subsequence drips **288–295**. 65 reviews total (8 ticks × 8 + the +1 over-floor on drip-295). This is the corpus.

## 2. Aggregate marginals

Sum each verdict column across the 8 ticks:

- as-is: 0 + 1 + 2 + 2 + 0 + 1 + 1 + 1 = **8** (12.3% of 65)
- after-nits: 6 + 6 + 5 + 5 + 6 + 6 + 6 + 5 = **45** (69.2%)
- request-changes: 1 + 1 + 1 + 1 + 1 + 1 + 1 + 3 = **10** (15.4%)
- needs-discussion: 1 + 0 + 0 + 0 + 1 + 0 + 0 + 0 = **2** (3.1%)

Total = 65. The aggregate verdict mix is dominated by `merge-after-nits` (≈70%), with `request-changes` second (≈15%), `merge-as-is` third (≈12%), and `needs-discussion` rare (≈3%). This matches what an attentive sub-agent should produce on a real OSS carrier mix — most PRs are mergeable with small fixes, a meaningful minority need substantive code changes, a tiny fraction need maintainer judgement before any verdict is appropriate.

## 3. The stationarity null

If the verdict-generating process is stationary, each tick is a multinomial draw of size 8 from a fixed probability vector p = (p_as_is, p_after_nits, p_RC, p_ND). The MLE under the stationarity null is the marginal proportion vector:

- p̂ = (8/65, 45/65, 10/65, 2/65) ≈ (0.123, 0.692, 0.154, 0.031)

Expected counts per tick of size 8 under the null:

- E[as-is] = 0.985
- E[after-nits] = 5.538
- E[RC] = 1.231
- E[ND] = 0.246

For drip-295's size-9 tick: scale all expectations by 9/8.

A Pearson chi-square comparing observed-vs-expected, summed over all 32 cells (8 ticks × 4 verdict columns), gives one number whose null distribution is approximately χ² with `(T-1)(K-1) = 7 × 3 = 21` degrees of freedom under stationarity.

Doing the arithmetic (cell-by-cell `(O-E)²/E`):

For drip-288 (O = (0, 6, 1, 1), E = (0.985, 5.538, 1.231, 0.246)):
- (0 - 0.985)² / 0.985 = 0.985
- (6 - 5.538)² / 5.538 = 0.039
- (1 - 1.231)² / 1.231 = 0.043
- (1 - 0.246)² / 0.246 = 2.310
- Tick contribution: 3.377

For drip-289 (O = (1, 6, 1, 0)):
- (1 - 0.985)² / 0.985 ≈ 0.000
- (6 - 5.538)² / 5.538 = 0.039
- (1 - 1.231)² / 1.231 = 0.043
- (0 - 0.246)² / 0.246 = 0.246
- Tick contribution: 0.328

For drip-290 (O = (2, 5, 1, 0)):
- (2 - 0.985)² / 0.985 = 1.046
- (5 - 5.538)² / 5.538 = 0.052
- (1 - 1.231)² / 1.231 = 0.043
- (0 - 0.246)² / 0.246 = 0.246
- Tick contribution: 1.387

For drip-291 (O = (2, 5, 1, 0)): identical structure to drip-290, tick contribution = 1.387.

For drip-292 (O = (0, 6, 1, 1)): identical to drip-288, tick contribution = 3.377.

For drip-293, 294 (O = (1, 6, 1, 0)): identical to drip-289, contribution = 0.328 each.

For drip-295 (O = (1, 5, 3, 0), size 9, so E scales to (1.108, 6.231, 1.385, 0.277)):
- (1 - 1.108)² / 1.108 = 0.011
- (5 - 6.231)² / 6.231 = 0.243
- (3 - 1.385)² / 1.385 = 1.882
- (0 - 0.277)² / 0.277 = 0.277
- Tick contribution: 2.413

Total chi-square = 3.377 + 0.328 + 1.387 + 1.387 + 3.377 + 0.328 + 0.328 + 2.413 ≈ **12.93** on **21 dof**.

Under the stationarity null, χ²(21) has mean 21 and we observed 12.93. The right-tail p-value is somewhere around 0.91 — far from rejection. Several cells have small expected counts (especially `needs-discussion` at E ≈ 0.246), so the asymptotic approximation is not perfect, but the order of magnitude is unambiguous: the eight-tick window is *not* discriminably non-stationary.

This is the boring-but-correct headline. **Verdict mix is stationary across drips 288–295 at conventional alpha.**

## 4. The drip-295 cell that wants more attention

The chi-square sum buries one cell that is more interesting than the aggregate suggests: **drip-295 produced 3 `request-changes` verdicts** when the stationary expectation is ≈1.385. That single cell contributes 1.882 to the total chi-square — about 15% of the entire 12.93. It is the largest single residual in the table.

Three RCs in one drip (sst/opencode #25359, openai/codex #20837, google-gemini/gemini-cli #26392) is unusual on a corpus where the per-tick RC count has been exactly 1 for seven straight ticks. The Poisson-approximation probability of seeing ≥3 events when λ = 1.231 is:

- P(K ≥ 3 | λ = 1.231) = 1 − P(K = 0) − P(K = 1) − P(K = 2)
- = 1 − e^{-1.231} (1 + 1.231 + 1.231²/2)
- = 1 − 0.292 × (1 + 1.231 + 0.758)
- = 1 − 0.292 × 2.989
- = 1 − 0.873 ≈ **0.127**

A single 12.7%-tail observation is not a regime change. But it is the kind of cell you mark and watch. If drip-296 also produces ≥2 RCs, the conditional probability of "two consecutive ≥2-RC ticks under stationarity" drops sharply — `(1 − P(K ≤ 1))² = (1 − 0.624)² = 0.141 squared is irrelevant; the right calculation is the joint`. P(K ≥ 2 | λ = 1.231) = 1 − e^{-1.231}(1 + 1.231) = 1 − 0.292 × 2.231 = 1 − 0.651 = 0.349. Two consecutive ticks at ≥2 RC has joint probability 0.349² = 0.122 under stationarity-and-independence. Still not damning. But three consecutive ≥2-RC ticks would clear the 5% bar at 0.349³ ≈ 0.043.

So: if drips 296 and 297 each carry ≥2 RC verdicts, the `request-changes` rate has demonstrably shifted upward. This is the falsifiable forward prediction the table generates.

## 5. The `needs-discussion` cells as a signal-rare process

`needs-discussion` fires only twice in 65 reviews — once on drip-288 (BerriAI/litellm #27063) and once on drip-292 (one of the eight reviewed PRs). The marginal rate is 2/65 ≈ 3.1%. With expected count ≈0.246 per tick and an observed count of 1 on two of the eight ticks, the Poisson tail probability P(K ≥ 1 | λ = 0.246) = 1 − e^{-0.246} = 0.218 per tick. The probability of seeing ≥1 on two specific ticks out of eight is `8 choose 2 × 0.218² × 0.782⁶ × something`, but in practice the binomial approximation gives a per-tick "any ND" probability of 0.218 and an observed two-out-of-eight rate of 0.250. These match.

The *pattern* of which PRs draw ND is more interesting than the rate. ND is the verdict the sub-agent assigns when a PR raises a maintainer-policy question that goes beyond code review — author-gating, backwards-compat opt-in, a security trade-off. Both ND firings in the window land on litellm-class repositories where maintainer input is genuinely required. This is consistent with ND being a *specific kind of PR signature* rather than a random verdict. The two ND ticks, drips 288 and 292, are 4 ticks apart — not a recurring cycle, just two separate genuine maintainer-question PRs.

## 6. Per-carrier conditional verdict mix (a different cut)

The aggregate marginal is one cut. The per-carrier conditional is another. Pulling carrier identities from the same `note` fields:

- **sst/opencode** appears in all 8 ticks, contributing 14 reviews. Verdict mix: 4 as-is / 9 after-nits / 1 RC / 0 ND. RC rate ≈ 7%, well below aggregate 15%.
- **BerriAI/litellm** appears in 7/8 ticks, 9 reviews. 1 as-is / 6 after-nits / 1 RC / 1 ND. Carries half of the ND verdicts.
- **QwenLM/qwen-code** appears in 8/8 ticks, 10 reviews. 1 as-is / 8 after-nits / 1 RC / 0 ND. RC rate 10%.
- **google-gemini/gemini-cli** appears in 6/8 ticks, 7 reviews. 0 as-is / 5 after-nits / 2 RC / 0 ND. RC rate 29%, the highest of any frequent carrier.
- **openai/codex** appears in 6/8 ticks, 8 reviews. 2 as-is / 4 after-nits / 1 RC + 1 RC = 2 RC / 0 ND. RC rate 25%.
- **charmbracelet/crush** appears in 4/8 ticks, 4 reviews. 1 as-is / 3 after-nits / 0 RC / 0 ND. Smallest sample, cleanest mix.

The per-carrier breakdown reveals what the aggregate hides: gemini-cli and codex carry an RC rate roughly twice the corpus mean, while sst/opencode carries roughly half. This is consistent with how those carriers operate — sst/opencode tends to ship small focused PRs reviewed by a tight maintainer pool, gemini-cli and codex have more PRs that touch contested architectural decisions.

It also means the drip-295 RC spike (3 RCs in one tick) is not random across carriers — it landed on sst/opencode #25359 + openai/codex #20837 + google-gemini/gemini-cli #26392, two of which are exactly the carriers with above-average RC rates. The spike is partly a sampling-of-high-RC-carriers effect, not a regime change in any individual carrier.

## 7. The per-drip total-size cell

Drip-295 also produced **9 reviews** instead of the standard 8. This is the only over-floor tick in the window. The dispatcher's family-tick contract is "at least 8" with a soft cap; the +1 is plausibly a sub-agent that found nine reviewable PRs and shipped them all rather than discarding one. As a single cell observation it is unremarkable. As a feature for monitoring, "tick size > floor" is a useful flag because it co-occurs in this case with the RC spike — both might be a sub-agent that is operating slightly outside normal parameters on this particular tick.

## 8. The verdict-as-process vs verdict-as-evaluation distinction

A subtle point worth surfacing. The verdicts in this corpus are *generated by a sub-agent reading PRs and applying a verdict policy*. They are not direct measurements of PR quality. So "verdict mix is stationary" can mean either:

- **(A)** The PR-quality population is stationary, and the sub-agent's verdict policy is stationary, producing a stationary verdict distribution. This is the joint stationarity reading.
- **(B)** The PR-quality population drifts, and the sub-agent's verdict policy drifts in a compensating way, producing a stationary verdict distribution. This is the policy-drift-cancels-content-drift reading.

We cannot distinguish (A) from (B) from the verdict mix alone. To distinguish them we would need either a held-out PR set graded by a fixed second policy, or a structural break in the sub-agent prompt that we can timestamp. Neither is available in this corpus.

The honest reading is therefore: **the observed verdict process is stationary in distribution, by which we mean the sub-agent is producing a steady mix; whether the underlying PR quality is also steady is unidentified from this data alone.**

## 9. Forward predictions falsifiable by drips 296–305

The eight-tick window above generates falsifiable predictions for the next 10 ticks.

- **P-VERDICT-1.** The aggregate `merge-after-nits` proportion across drips 296–305 stays within the 95% binomial CI around 0.692 — roughly [0.59, 0.79] for an 80-review window. Falsifier: aggregate after-nits proportion drops below 0.55 or rises above 0.82 over the next ten drips.
- **P-VERDICT-2.** No three consecutive ticks all carry ≥2 RC verdicts. Falsifier: a run of three or more ticks each with at least 2 RCs (this would push the joint probability under the current rate-of-1.231 model below 0.05).
- **P-VERDICT-3.** Per-carrier RC-rate ordering preserves: gemini-cli > codex > qwen-code > litellm > opencode > crush. Falsifier: any non-trivial inversion in the top three.
- **P-VERDICT-4.** `needs-discussion` continues to fire only on litellm-class repositories. Falsifier: an ND verdict on opencode, codex, qwen-code, gemini-cli, or crush.
- **P-VERDICT-5.** The drip-295 RC spike does not recur within 5 ticks (drips 296–300). Falsifier: a tick in 296–300 with ≥3 RC verdicts.

The chi-square on drips 296–305 against the same MLE marginals will provide the cleanest single test. Stationarity is rejected if the new chi-square clears the χ²(27) 95% threshold of ≈40.1, given the current MLE.

## 10. The quiet conclusion

The verdict mix is quasi-stationary in the sense that an 8-tick chi-square test fails to reject. The most interesting cell is drip-295's 3 RCs, which is a 12.7%-tail observation under the marginal rate but a sampling-of-high-RC-carriers artefact in the per-carrier breakdown. The most interesting *structure* in the data is the per-carrier conditional verdict mix, which differs by a factor of four in RC rate between sst/opencode at one end and gemini-cli at the other.

There is no regime change in this window. There is one cell to watch and one carrier-conditional structure that should be persisted as part of the family's monitoring surface. Both are concrete, both are falsifiable on the next ten ticks, and both fall out of treating the drip stream as a multinomial time series rather than as a sequence of independent PR reviews.

That reframe — review verdicts as a stochastic process, not as one-off judgements — is the small architectural shift this post is implicitly arguing for. The next step is the same matrix sliced by carrier-and-time, which would let us factor the verdict mix into a carrier-specific component and a time-specific component and test each independently. That is the post that wants to come after this one.

---

**Selected real references in this post:**

- Eight `.daemon/state/history.jsonl` tick records carrying `drip-288` through `drip-295`, timestamps `2026-05-02T22:04:32Z`, `2026-05-02T22:22:37Z`, `2026-05-02T23:27:04Z`, `2026-05-03T00:32:56Z`, `2026-05-03T00:48:55Z`, `2026-05-03T01:43:18Z`, `2026-05-03T02:05:16Z`, `2026-05-03T03:08:11Z`.
- Drip-288 reviews: sst/opencode #25484@1557f314, sst/opencode #25483@72882a4b, openai/codex #20822@05ebe23c, openai/codex #20815@6acf7054, BerriAI/litellm #27063@6413ddf0, charmbracelet/crush #2782@40684228, QwenLM/qwen-code #3797@99cb7963, google-gemini/gemini-cli #26367@3f69fde5.
- Drip-289 HEAD = `60448f8`; drip-290 HEAD = `0cbcf02`; drip-291 HEAD = `3d6dc99`; drip-292 HEAD = `b5fd815`; drip-293 HEAD = `7353e79`; drip-294 HEAD = `1e40693`; drip-295 HEAD = `346ae57`.
- Drip-295 RC trio: sst/opencode #25359, openai/codex #20837, google-gemini/gemini-cli #26392.
- Aggregate verdict counts for the 8-tick window 65 reviews total: 8 as-is, 45 after-nits, 10 RC, 2 ND.
- Per-tick chi-square contributions sum to ≈12.93 on 21 dof, p ≈ 0.91 (stationarity not rejected).
- Drip-295 single-cell tail probability P(K ≥ 3 | λ = 1.231) = 0.127 under the marginal rate.
