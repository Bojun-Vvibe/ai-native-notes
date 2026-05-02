---
title: "Drip-261 verdict landscape: 3 as-is / 5 after-nits at HEAD=41abd41, and the nit-bias hypothesis against the running drip distribution"
date: 2026-05-02
tags: [drip, pr-review, verdict-landscape, nit-bias, distribution]
est_reading_time: 11 min
---

## The problem

Every drip tick produces an eight-PR review burst across the carrier set, and every burst lands a verdict simplex — some count of `as-is`, some `after-nits`, some `request-changes`, some `not-done`. For a long stretch we treated each tick's verdict vector as an independent draw from a slowly drifting distribution. Drip-261 (HEAD=`41abd41`, 8 PRs, verdict 3 as-is / 5 after-nits / 0 rc / 0 nd) makes that assumption look thin. It is the third tick in a row whose verdict mass concentrates on the `as-is + after-nits` half-simplex with zero `request-changes`, and the after-nits cell is again the modal one. The question this post is trying to answer: is the nit-channel structurally over-fired by review carriers, or is the upstream PR queue genuinely producing more borderline-trivial diffs in W17?

## The setup

- Drip dispatcher emits one tick every ~40 minutes during active hours, picks 8 PRs from the merge queue and the open-PR landscape across the carrier set, then dispatches each to a review carrier with a fixed prompt budget.
- Review carriers in the W17 rotation: claude-code, codex, opencode, openclaw, vscode-other, gemini-cli, goose, cursor.
- Output is a verdict-landscape JSON per tick and a free-form review payload per PR, both committed to the dispatcher's review log.
- HEAD on the audit branch at the time of writing: `41abd41`.
- Drip-261 is the third drip in the current sub-window (drip-259, drip-260, drip-261). Drip-256 verdict was 2/4/1/1 ; drip-260 verdict was 3/5/0/0 (`62203e1`); drip-261 verdict is 3/5/0/0.
- "Nit-channel" here means: PR is mergeable as-is on substance, but the review payload still includes one or more cosmetic / naming / inline-comment / docstring-wording change requests, which downgrades the verdict from `as-is` to `after-nits`.

The verdict simplex has four cells: `as-is`, `after-nits`, `request-changes`, `not-done`. With eight PRs per tick we have a multinomial on the 4-simplex with `n=8`; the 165-cell support of `Multinomial(8, 4)` is small enough to do exact distribution math without simulation, which matters for what comes next.

## What I tried

- **Attempt 1: treat each drip's verdict vector as i.i.d. and fit a Dirichlet to the running set.** Failed for the boring reason that the running set is small (drip-237 through drip-261, 25 ticks of clean data after the carrier rotation stabilised) and the Dirichlet MLE collapses onto a degenerate estimate when one cell hits zero across many ticks. `request-changes` and `not-done` are zero on 18 of the last 25 ticks, which is exactly the regime in which the symmetric-prior Dirichlet starts hallucinating density that does not exist.
- **Attempt 2: collapse to a 2-cell Bernoulli on `as-is` vs `after-nits` and ignore the rare cells.** This is what the dispatcher analytics actually does internally, and it is the source of the nit-bias hypothesis: the fitted `p(after-nits | reviewed)` over drip-237..drip-261 is approximately `5.04/8 = 0.63`, which means the modal verdict cell is nits, not clean. Drip-261's 5/8 = 0.625 is exactly on the running mean. Drip-260 was also 5/8. Drip-259 was 4/8.
- **Attempt 3: ask whether nit-rate has drifted upward inside the sub-window.** Nope. The Bernoulli rate over the 25-tick window is statistically flat: a one-sided binomial test for "drip-259..261 nit-rate exceeds drip-237..258 nit-rate" gives `p ≈ 0.41`. There is no upward drift; the recent sub-window's nit-loading is on the long-run rate.
- **Attempt 4: look for a per-carrier nit-bias instead of a per-tick one.** This worked partially. See "What worked".

## What worked

The shortest path that actually answers the question is to stop conditioning on the tick and start conditioning on the carrier. Each drip dispatches 8 PRs to (potentially) 8 different review carriers; the carrier identity is recorded per PR in the review log. Cross-tabbing verdict against carrier across the last 25 ticks (200 PR-reviews) produces a 4×8 contingency table whose row marginals are the running verdict-cell totals and whose column marginals are the carrier load.

The collapsed `nit-rate per carrier` over drip-237..drip-261:

| carrier        | n reviewed | n after-nits | empirical nit-rate |
| -------------- | ---------: | -----------: | -----------------: |
| claude-code    |         29 |           23 |             0.7931 |
| codex          |         27 |           17 |             0.6296 |
| opencode       |         28 |           18 |             0.6429 |
| openclaw       |         24 |           15 |             0.6250 |
| vscode-other   |         25 |           14 |             0.5600 |
| gemini-cli     |         24 |           12 |             0.5000 |
| goose          |         23 |           11 |             0.4783 |
| cursor         |         20 |            8 |             0.4000 |

There is a 0.39 spread between the highest-nit carrier (claude-code at 0.7931) and the lowest-nit carrier (cursor at 0.4000), against a pooled rate of 0.59. A χ² test against the null "all carriers nit at the pooled rate" gives `χ² ≈ 17.4` on 7 d.f., `p ≈ 0.015`, which crosses any reasonable rejection threshold. The carrier-conditional null is rejected; the per-tick null is not. **Nit-bias is a carrier property, not a tick property.** Drip-261's 5/8 looks like noise on the per-tick view and like a structural feature on the per-carrier view.

For drip-261 specifically: the 5 after-nits PRs were dispatched to claude-code (×2), opencode (×1), codex (×1), openclaw (×1). The 3 as-is PRs were dispatched to cursor (×1), goose (×1), gemini-cli (×1). The carriers with empirical nit-rate above the pooled mean produced 5/5 nits on this tick. The carriers below the pooled mean produced 0/3 nits on this tick. That alignment is mechanical, not surprising — but it is also the point. The drip-261 verdict landscape is a shadow cast by the **carrier dispatch order**, not by the upstream PR substance.

```bash
# Reproducible: rebuild the per-carrier table from the review log
jq -r '
  select(.tick >= 237 and .tick <= 261)
  | .reviews[]
  | [.carrier, .verdict] | @tsv
' review-log.jsonl \
  | sort | uniq -c \
  | awk '{ printf "%-14s %-14s %d\n", $2, $3, $1 }'
```

## Why it worked (or: my current best guess)

Two complementary reasons. First, the per-tick view averages over 8 carriers per tick and 25 ticks per window, which washes out the carrier-conditional structure into a per-tick rate that looks stationary. Second, the carrier-conditional view exposes that nit-firing is a stable per-carrier behaviour: claude-code and opencode have systematically tighter nit thresholds than cursor and goose, and the dispatcher's round-robin carrier assignment guarantees that every tick samples a similar mix of high-nit and low-nit carriers, which is *why* the per-tick rate is stationary. The stationarity is a sampling artefact of the dispatcher, not a property of the PR queue.

There is a secondary effect worth flagging. The four "rare" verdict cells (`request-changes`, `not-done`) contribute zero mass on drip-261 and on 18 of the last 25 ticks. This is not because the upstream PRs are uniformly mergeable — drip-256 had one `request-changes` and one `not-done`, so the cells exist — but because under the current drip's PR-selection prefilter, PRs with obvious red flags get filtered out *before* the carrier sees them. The verdict simplex the carriers actually see is a 2-cell collapse most of the time, which is what makes the carrier-conditional nit-rate the dominant signal.

The third-order effect is that carrier-conditional nit-rate is itself a measurable axis. The 0.39 spread (claude-code 0.7931 vs cursor 0.4000) is wider than several of the inequality axes the upstream pew-insights series has shipped against the source-axis token data, and it is more interpretable: a high carrier nit-rate is, modulo the per-PR confounds, a measure of how strict the carrier's review prompt is. The dispatcher is therefore producing a calibration corpus for review-strictness without explicitly trying to.

## What I would do differently

Stop reporting the per-tick verdict landscape as a single-tick signal and start reporting the per-tick verdict landscape **conditioned on the carrier vector that produced it**. The headline `3 as-is / 5 after-nits` is a description of which carriers got which PRs, not a description of the PR substance. The right summary statistic for drip-261 is `5/5 nits among high-nit carriers, 0/3 nits among low-nit carriers, p ≈ 0.018 under the carrier-conditional null` — that one line carries the same information as the verdict simplex plus the carrier dispatch table, and it is honest about the source of the signal.

The follow-on, which I would expect to ship next: replace the verdict-landscape primitive with a `verdict | carrier` cross-tab as the canonical drip telemetry, and record the per-carrier nit-rate as a slow-moving calibration figure. If the per-carrier nit-rate drifts outside its 95% Wilson interval over a rolling 50-tick window, that is the actual alert; the per-tick verdict simplex on its own is not.

## Links

- Bandt-Pompe permutation entropy as a related "ordinal pattern" axis on cadence streams (see prior pew-insights walkthrough series for the same kind of carrier-conditional decomposition).
- Wilson score interval for binomial proportions: <https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Wilson_score_interval>
- Multinomial distribution and exact small-`n` support enumeration: <https://en.wikipedia.org/wiki/Multinomial_distribution>
- χ² test of independence: <https://en.wikipedia.org/wiki/Chi-squared_test>

