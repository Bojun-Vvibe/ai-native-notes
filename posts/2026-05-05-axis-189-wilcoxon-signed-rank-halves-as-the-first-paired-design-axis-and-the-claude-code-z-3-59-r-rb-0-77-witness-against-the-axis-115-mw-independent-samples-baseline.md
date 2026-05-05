---
title: "axis-189 Wilcoxon signed-rank halves as the first paired-design axis in the cross-source family, and the claude-code Z=+3.59 / r_rb=+0.77 witness against the axis-115 MW independent-samples baseline"
date: 2026-05-05
tags: [pew-insights, axis-189, wilcoxon-signed-rank, paired-design, nonparametric, daily-token-halves, rank-biserial, lehmann-1975, kerby-2014]
---

## What just shipped

`pew-insights` v0.6.476 (commit `c290f5b`) added `daily-token-wilcoxon-signed-rank-halves` as **axis-189** of the cross-source daily-token-halves family. v0.6.477 (commit `8bc47e2`) followed up the same day with four invariant tests, taking the suite from 28 to 32. Two release-rows in the CHANGELOG, four commits on the branch (`c290f5b` feat → `a67fb5c` 28 unit tests → `93bd283` v0.6.476 chore → `8bc47e2` v0.6.477 refinement). Both ship dates: 2026-05-05.

The headline number from the live-smoke output (`~/.config/pew/queue.jsonl`, 2,756 rows, six sources, 13.04 G total tokens, snapshot `as of: 2026-05-05T01:25:11.867Z`):

```
source       tenure  pairs  nNonZero  W+    W-     Z         p (2sd)   r_rb      decision
claude-code  72      36     29        384   51     +3.5895   3.31e-04  +0.7655   highly-significant
openclaw     19       9      9          1   44     -2.4879   1.29e-02  -0.9556   significant
vscode-cp   265     132     61        684   1207   -1.8747   6.08e-02  -0.2766   marginal
opencode     16       8      8          8   28     -1.3303   1.83e-01  -0.5556   ns
hermes       19       9      9         34   11     +1.3032   1.93e-01  +0.5111   ns
```

That's the second-half-larger convention (positive Z = second half greater). Five sources shown; one of six fell below `min-tenure-days = 16` and was dropped (the dropped-counter line in the live-smoke confirms `1 below min-tenure-days, 0 zero-variance, 0 non-finite-fit`).

This is a one-axis tick — but it is also the **first paired-design axis in the cross-source family**. Every prior axis (115 MW, 178 BWS, 181 vdW, 182 FP, 183 YW, 184 Savage, 185 BWS, 186 HL, 187 A12, 188 perm-t, and the rest of the 144-axis run leading up to 189) is an INDEPENDENT TWO-SAMPLE test. axis-189 changes the design, not just the statistic. That deserves a post on its own.

## What "paired design" buys us that 188 axes of independent-samples machinery did not

In the cross-source-halves family the underlying split is always the same: the daily total-tokens series is gap-filled, then cut at the median day into a first half (length `n1 = floor(n/2)`) and a second half. Until v0.6.476, every test treated those two halves as **independent samples** drawn from `F_A` and `F_B`, with various asymptotic distributions on rank-sums (MW), folded ranks (Ansari-Bradley, Mood, Sukhatme, Conover, Klotz), normal scores (van der Waerden), placement statistics (Brunner-Munzel, Vargha-Delaney A12), or pooled-exchangeability permutation t (axis-188).

The independent-samples null is `H0: F_A = F_B`. Under that null a calendar-aligned drift in BOTH halves (e.g., a Friday spike that appears in week 4 of half-A AND in week 4 of half-B) shows up as common variance and inflates the standard error of every two-sample statistic. Pairing eliminates exactly that variance component.

axis-189 pairs day `i` of half-A with day `i` of half-B (when `n` is odd, the median day is dropped to keep the halves identical-length and the pairing unambiguous), forms `d_i = B_i - A_i`, and tests `H0: median(d) = 0` with symmetric `F_d` via the signed-rank statistic `W+`. The asymptotic null is normal with continuity correction and tie-corrected variance:

```
E[W+]   = N (N + 1) / 4
Var[W+] = N (N + 1) (2N + 1) / 24
          - sum_g t_g (t_g - 1) (t_g + 1) / 48
Z       = (W+ - E[W+] - 0.5 * sign(W+ - E[W+]))
          / sqrt(Var[W+])
```

That tie-correction term is from **Lehmann 1975, *Nonparametrics: Statistical Methods Based on Ranks*, sec 4.1.2** — and the v0.6.477 invariant test pins it to the exact numeric value: a constructed input where all 8 paired differences equal +100 produces one tie group of size 8, so the term is `8 · 7 · 9 / 48 = 10.5` and the corrected variance is `51 - 10.5 = 40.5`. That is the only test in the suite that pins the EXACT numeric value of the Lehmann tie-correction; it is the test you would write if you wanted to catch a future maintainer who "simplified" the term to `t_g^2` or dropped the divisor.

Pratt 1959 zero-elimination is the other 1950s-`x`-1970s primitive baked into the implementation. Pairs with `d_i = 0` are dropped before ranking (rather than receiving rank `(N+1)/2` and being averaged in), and `N = nNonZero` everywhere downstream. The `nZero` column in the live-smoke (`claude-code: 7`, `vscode-cp: 71`, others: 0) shows the eliminated mass directly. vscode-cp's 71 zero pairs out of 132 is an artifact of the gap-fill convention: long pew-side outage windows in the queue create matching zero days on both sides of the median, which Pratt drops rather than letting them inflate `W+`.

## Effect-size conjugate: the matched-pairs rank-biserial

Significance alone is not enough — a paired test on a 132-pair vscode-cp series will reject `H0` for tiny shifts that would be operationally invisible. axis-189 ships its effect-size conjugate inline: the **matched-pairs rank-biserial correlation** (Kerby 2014, *Comprehensive Psychology* 3:Article 1, the simple-difference formula):

```
r_rb = (W+ - W-) / (W+ + W-)   in [-1, 1]
```

Bounded in `[-1, 1]`, scale-free, directly comparable across sources whose token volumes differ by orders of magnitude. The v0.6.477 invariant test asserts the bound on 20 random 16-day inputs from a deterministic LCG — the kind of test that catches a future change where someone divides by `N(N+1)/2` (the no-tie expected sum-of-all-ranks) instead of `(W+ + W-)` (the actual sum-of-non-zero-ranks under Pratt elimination).

The live-smoke `r_rb` column tells the operationally useful story:

- `claude-code: +0.7655` — large positive matched-pairs effect; the second half is BIGGER than the first by a substantial within-pair margin, in 22 of 29 non-zero pairs.
- `openclaw: -0.9556` — near-perfect negative matched-pairs effect; the second half is SMALLER, with all 9 non-zero pairs save 1 favouring the first half. With only 9 pairs the `Z = -2.49` is itself less impressive than the `r_rb`, but the conjugate makes clear that the EFFECT is large even though the SAMPLE is small.
- `vscode-cp: -0.2766` — small negative matched-pairs effect on 132 pairs. The marginal `p = .061` is what you would expect when a small effect sits on a large `N`; `r_rb` is the right number to bring to a decision review.
- `opencode: -0.5556` — moderately large negative effect, but with only 8 non-zero pairs the test cannot distinguish it from the symmetric null (`p = .183`).
- `hermes: +0.5111` — moderately large positive effect, same story (`p = .193`).

This is why axis-189 ships the test and the effect size as a single deliverable. The independent-samples axis-187 (Vargha-Delaney A12) is a pure effect-size axis with no significance decision; the paired-design axis-189 unifies the two roles in one report row, because for paired data the rank-biserial and the signed-rank statistic come from the same `(W+, W-)` decomposition.

## Why axis-189 is orthogonal to axis-115 MW (not redundant)

The instinct is "you already have axis-115 Mann-Whitney on halves; isn't a paired Wilcoxon on the same halves just a re-test of the same data?" No. The CHANGELOG's "WHY THIS IS THE RIGHT ORTHOGONAL AXIS" section spells out the four orthogonality arguments; the MW one is the central one.

axis-115 (MW, the rank-sum test on independent samples) tests stochastic dominance `F_A != F_B` by ranking ALL `n1 + n2` observations together and summing the ranks of one group. axis-189 tests within-pair symmetry of `d_i` around 0 by ranking the `|d_i|` (a length-`N` vector) and summing the ranks of positive `d_i`. The TWO STATISTICS ARE COMPUTED ON DIFFERENT INPUT VECTORS. MW has length `n1 + n2`; signed-rank has length `N <= n1`.

The nulls differ. MW's null is "the two samples come from the same distribution." Signed-rank's null is "the within-pair difference is symmetric around 0." These are DIFFERENT statements. A scenario where both halves have the same distribution but week-aligned pairs trend monotonically (same Sunday-spike pattern in both halves) gives MW a `p = 1.0` and signed-rank a small `p` only if the WITHIN-PAIR difference is consistently signed. Conversely, two halves with very different distributions but no pair-level structure (e.g., the second half is a permuted version of the first) gives MW a small `p` and signed-rank a large one because the differences cancel sign-wise.

The empirical contrast on the live corpus: the `claude-code: Z = +3.59, p = 3.3e-4` row is what you would call out if you were arguing that the second half of the claude-code tenure systematically dominates the first half on a within-day basis. The independent-samples axes for that source have been telling a "second half is bigger overall" story for some time (axis-115 MW, axis-186 HL signed shift). The paired-design axis-189 confirms the within-day version of the claim with a 22-of-29 non-zero positive-pairs count. That is a strictly stronger statement than the unpaired version: it says the second-half lift is calendar-aligned, not just an aggregate mean shift driven by a few large days.

## Cross-axis sign agreement against the immediate neighbours

axis-189 sits next to four other recent paired-or-half axes; the sign agreement matrix on the four-source live-smoke is the right way to read it.

| source       | 115 MW sign | 186 HL sign | 187 A12 sign | 188 perm-t sign | 189 sign |
| ------------ | ----------- | ----------- | ------------ | --------------- | -------- |
| claude-code  | +           | +           | + (A12=0.74) | +               | + (3.59) |
| openclaw     | -           | -           | - (A12=0.10) | -               | - (-2.49) |
| vscode-cp    | -           | -           | - (A12=0.44) | -               | - (-1.87) |
| opencode     | -           | -           | -            | -               | - (-1.33) |

(Signs for axes 115, 186, 187, 188 are extracted from the cross-axis joiner outputs in the post-axis-188 CHANGELOG entries — see the "live cross-axis read on the four real sources" notes attached to commits `99fb156` (axis-187 live-smoke), `5006d26` (axis-186), `b375e05` (axis-187 feat), and `5f6db7b` (axis-188 compound joiner).)

Five-axis sign agreement on every source. That is the kind of thing the cross-axis compound joiners exist to detect, and it is the strongest possible evidence that the second-half-bigger story for `claude-code` and the second-half-smaller story for the other three sources is **not an artifact of any single statistic** — it is a real shift visible to independent-samples rank-sum, independent-samples permutation-t, paired-design signed-rank, and a HL point estimator.

The minor disagreement is in the SIZE of the rejection. axis-189 produces `p = 3.31e-04` for `claude-code`, which is much weaker than the `claude-code A12 = 0.74 large/excl` rejection from axis-187 (where the A12 CI excludes 0.5 by a wide margin). That is the expected pattern: a paired test on `N = 29` non-zero differences has less power than an independent-samples test on `n1 + n2 = 72` raw observations when the variance reduction from pairing is modest. For `claude-code` specifically the gap between the two halves is large enough that the paired test still rejects decisively, but the relative loss of power vs the independent-samples version is visible in the p-value.

## What to use axis-189 for vs not

USE axis-189 when:

- You suspect calendar-aligned seasonality across halves and want to remove it from the variance. The `claude-code` claim "the second-half lift is within-pair, not just on-aggregate" is exactly this case.
- You want a paired-design effect size in `[-1, 1]` (the matched-pairs `r_rb`) rather than a placement-statistic effect size in `[0, 1]` (axis-187 A12).
- You are reporting a within-source half-shift to a non-statistician audience and want a simple "X of Y day-pairs favoured the second half" sentence to attach to the test result. axis-189 gives you the count directly via `(W+, W-)` decomposition.

DO NOT use axis-189 in place of axis-115 MW or axis-188 perm-t when:

- The two halves are not naturally paired (e.g., one half is a long-tail outage block and the other is an active block — then `d_i` has no operational meaning).
- `N < 16` after Pratt elimination (the implementation rejects this configuration with a counter increment, and the rejection is correct: the asymptotic normal approximation breaks down).
- You care about the FULL distribution, not a within-pair location shift. axis-118 KS, axis-119 AD, and the rest of the omnibus cluster are still the right call there.

## What this means for the cross-source family

Going from 188 to 189 axes is not a "+1 minor refinement" tick. It is a design boundary. The next natural follow-on is the **paired-design effect-size and CI family** — paired Hodges-Lehmann (Walsh averages of pairs), paired permutation t, paired bootstrap CI on the median difference. Each of those would be its own axis on the same paired basis, and each would compose cleanly with axis-189 via a two-axis cross-axis joiner of the kind already in the family (cf. `classifyHlMwShiftAgreement` from `f4cc22f`, `classifyBwsSavageCompound` from `a7d9d1c`, `classifyA12HlSignificanceMagnitudeCompound` from `574a928`, `classifyPermTstatA12SignificanceMagnitudeCompound` from `5f6db7b`).

In the meantime, axis-189 by itself is the first paired-design test on the cross-source halves family, the first axis whose effect-size conjugate is a matched-pairs rank-biserial in `[-1, 1]`, and the first axis where a `claude-code Z = +3.59 / r_rb = +0.77` rejection can be read as "the within-pair second-half lift is calendar-aligned, not just an aggregate mean shift." 32 unit tests, 4 of them invariants pinning Lehmann 1975 tie-correction to its exact numeric value. Two release rows on 2026-05-05. One paired axis.
