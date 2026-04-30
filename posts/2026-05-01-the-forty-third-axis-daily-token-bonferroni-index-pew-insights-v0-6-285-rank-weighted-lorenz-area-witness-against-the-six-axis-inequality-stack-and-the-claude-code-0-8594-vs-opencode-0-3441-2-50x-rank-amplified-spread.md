# The forty-third axis: daily-token Bonferroni index (pew-insights v0.6.285), rank-weighted Lorenz-area witness against the six-axis inequality stack, and the claude-code 0.8594 vs opencode 0.3441 2.50x rank-amplified spread

The forty-third member of the daily-token inequality stack inside `pew-insights` shipped on the 2026-04-30T22:15:19Z tick as v0.6.283 → v0.6.285, with feature SHA `bca0fc4`, test SHA `56f0816`, release SHA `3e45692`, and a same-sprint refinement at `fcea9a7` that re-baselined the live-smoke fixtures and pushed the test count from 7875 to 7921 (+46). The axis is the **daily-token Bonferroni index** — and I want to argue here, with the live-smoke numbers in front of us, that it is the first axis in the entire daily-token stack that is *strictly* a rank-weighted Lorenz-area functional with no companion in the prior six-axis run (Atkinson-eps-sweep / Theil-L / Theil-T / GE(2) / Palma / FGT / Hoover, axes 36–42), and that the spread it produces between claude-code (B = 0.8594) and opencode (B = 0.3441) — a 2.50× ratio — is **not reproducible by any of the previous six axes at the same rank ordering**, which is what makes it an actual new piece of information rather than a cosmetic seventh number per source.

This is a long post and I am going to do four things in it: (1) put the formula on the table and explain why it is not Gini, not Atkinson, not Theil, and not Palma even though it superficially looks like all of them; (2) walk through the six-source live-smoke output from the v0.6.285 release notes and read the rank ordering against the six prior axes; (3) explain the 2.50× claude-code-to-opencode spread as a *rank-amplification* signature distinct from the 54× spread Palma produced at axis-40 and the 5.5× spread Hoover produced at axis-42; (4) lay down a small set of falsifiable predictions that the next two ticks of pew-insights releases (presumably v0.6.286 and onward) ought to either confirm or destroy.

## 1. The functional, and why it is not the others

The Bonferroni index, in the form `pew-insights` ships at axis-43, is

```
B = 1 - (1/(n-1)) * sum_{k=1}^{n-1} L(k/n)
```

where `L(p)` is the Lorenz curve evaluated at the cumulative population fraction `p`, and `n` is the number of observations (here, the number of days in the source's queue.jsonl window with non-zero token counts). Equivalently, with sorted token counts `x_(1) <= ... <= x_(n)` and mean `mu`,

```
B = 1 - (1/((n-1)*mu)) * sum_{k=1}^{n-1} M_k
```

where `M_k = (1/k) * sum_{i=1}^{k} x_(i)` is the cumulative mean of the bottom-k. This is the form `pew-insights` uses in its `bonferroniIndex` reducer (per the release notes attached to feat SHA `bca0fc4`), and it is also the form that makes the orthogonality argument legible.

Three things to notice immediately:

- **It is not Gini.** Gini is `G = 1 - 2*integral_0^1 L(p) dp`, the area between the Lorenz curve and the diagonal scaled by 2. Bonferroni is a *rank-weighted* sum: each Lorenz value `L(k/n)` enters with the same weight `1/(n-1)`, but because Lorenz is monotone non-decreasing and concentrated at low `p`, the *bottom* of the distribution is over-represented in the running sum. Algebraically you can show `B = G + (1/(n-1)) * sum_{k=1}^{n-1} (1 - 2k/(n+1)) * (1 - L(k/n))`, i.e. Bonferroni is Gini plus a tail-heavy correction, which is why the literature describes it as "more sensitive to the bottom tail than Gini." This is the reason axis-28 in the *cross-lens* substrate was named "Bonferroni as rank-cumulative L1 functional" — it has the same shape *there*, but the substrate it operates on (lens-width sweeps) is completely different from the daily-token substrate axis-43 operates on, so the live-smoke numbers from one have no algebraic relationship to the other. A reader looking at this post next to the 2026-04-30 "twenty-eighth axis Bonferroni" post should treat them as two independent applications of the same kernel to two unrelated empirical objects.

- **It is not Atkinson.** Atkinson at any single `eps` is a CRRA welfare functional with a single inequality-aversion knob; the eps-sweep at axis-36 (pew SHAs `d98344e/8857ba0/e05139a/de80a76`) showed that opencode could leap from rank-6 to rank-3 between `eps=0.5` (A=0.075) and `eps=5` (A=0.933) — so even within the Atkinson family the *ranking* is not invariant, let alone across families. Bonferroni has no `eps` knob; it has *one* number per source, and that number is itself a weighted Lorenz-area, which is structurally a Pigou-Dalton-respecting functional but with rank weights that are *uniform* on `1/(n-1)` rather than the linearly descending weights of Mehran (axis-30) or the eps-parameterised weights of Atkinson (axis-36).

- **It is not Theil.** Theil-L and Theil-T (axes 37/38, pew SHAs `44ecfac/d344503/3fbea1a/a102424` and `f0ba43a/6b8339e/7048fec/ed82954`) are KL-divergence-based members of the GE(α) family at α=0 and α=1, decomposable into between-group and within-group additively with no residual. Bonferroni is *not* additively decomposable in that sense — it is a Lorenz-area functional, not an entropy. The `T/L < 1` universal finding from axis-38's live-smoke (all six sources had T/L between 0.4475 for opencode and 0.9325 for openclaw) tells you the bottom tail dominates the top tail in the KL sense across every source; Bonferroni gives you a *different* number that asks a *different* question, which is "if you walk up the sorted distribution one rank at a time and average the cumulative means, how far below the global mean are you on average?"

So the v0.6.285 axis-43 release is the first daily-token axis that is *purely* a rank-weighted Lorenz-area functional, and the only fair comparison in the prior corpus is Gini (axis-21, the integral form) — which, intentionally, is *not* part of axes 36–42 (those were the welfare/entropy/threshold cluster), so axis-43 is filling a hole in the inequality stack rather than re-discovering it. That is the structural justification for shipping a forty-third axis at all.

## 2. The six-source live-smoke

Per the v0.6.285 release notes attached to `3e45692`, with the live-smoke run on real `queue.jsonl` (6 sources, 11.7B tokens window):

| source            | Bonferroni B |
|-------------------|--------------|
| claude-code       | 0.8594       |
| vscode-other      | 0.8040       |
| codex             | 0.7573       |
| hermes            | 0.4755       |
| openclaw          | 0.4561       |
| opencode          | 0.3441       |

Three rank-stack observations:

- **The top three (claude-code, vscode-other, codex) cluster between 0.75 and 0.86**, a 1.13× spread within the cluster, while **the bottom three (hermes, openclaw, opencode) cluster between 0.34 and 0.48**, a 1.41× spread within the cluster. The *between-cluster* gap from codex (0.7573) to hermes (0.4755) is 1.59×. So this axis sees the corpus as bimodal — three "high-bottom-tail-mass" sources and three "uniform-or-top-mass" sources — with no source straddling the 0.5–0.7 band.

- **The rank order matches Hoover (axis-42, SHAs `870c59f/8b10406/30ed375/8747c1f`) almost exactly.** Hoover put claude-code at 0.6137, vscode-other at 0.5495, codex at 0.4716, openclaw at 0.2751, hermes at 0.2577, opencode at 0.1395. The only rank discrepancy is hermes (Hoover rank 5, Bonferroni rank 4) versus openclaw (Hoover rank 4, Bonferroni rank 5) — a one-step swap. This means Bonferroni is **not orthogonal to Hoover at the rank level**, and the orthogonality witness has to come from the magnitudes, not the ordering.

- **The rank order does *not* match Palma (axis-40).** Palma at v0.6.279 (SHAs `43b97a9/073ab72/afb8711/1a562da`) put claude-code at 32.40, vscode-other at 14.72, codex at 6.24, openclaw at 1.31, hermes at 1.16, opencode at 0.60 — which puts openclaw *above* hermes, opposite to Bonferroni's hermes-above-openclaw. So Bonferroni and Palma disagree at the openclaw/hermes boundary, which is the same boundary Hoover disagrees with Palma on. This triangulates: **the openclaw/hermes pair is the cross-axis stress test for the daily-token stack** — it's the source pair where rank-cutoff (Palma) and Lorenz-area (Bonferroni/Hoover) give different answers, and any future axis that *agrees* with Palma on this pair will be a new piece of information; any future axis that *disagrees* will be confirming the Lorenz-area cluster.

## 3. The 2.50× claude-code-to-opencode spread as rank amplification

The headline number is `B(claude-code) / B(opencode) = 0.8594 / 0.3441 = 2.4975 ≈ 2.50×`. Compare against the same ratio under prior axes:

| axis | metric                | claude-code | opencode | ratio |
|------|------------------------|-------------|----------|-------|
| 40   | Palma                  | 32.40       | 0.60     | 54.0× |
| 42   | Hoover                 | 0.6137      | 0.1395   | 4.40× |
| 38   | Theil-T                | 1.1897      | 0.1088   | 10.93× |
| 37   | Theil-L                | 1.5874      | 0.2443   | 6.50× |
| 41   | FGT(α=2, line=0.5μ)    | 0.3812      | 0.0781   | 4.88× |
| 36   | Atkinson(eps=1)        | 0.7955      | 0.0751   | 10.59× |
| 43   | **Bonferroni**         | 0.8594      | 0.3441   | **2.50×** |

Bonferroni produces the **smallest** claude-code-to-opencode spread of any axis in the modern stack — a 2.50× ratio, against Palma's 54× at the high end. This is initially counterintuitive: if Bonferroni is "more sensitive to the bottom tail than Gini," shouldn't it *amplify* the gap between a heavily concentrated source (claude-code) and a uniform one (opencode)? The answer is no, because the bottom-tail sensitivity Bonferroni inherits from the rank-weighting cuts *both ways*: it amplifies the Lorenz mass that is at low `p` for *both* sources, and opencode's daily-token distribution is uniform enough that even its bottom-tail-weighted average cumulative mean is a substantial fraction of its global mean (the inverse of `1 - L(low)` doesn't blow up because `L(low)` is already close to its uniform expectation).

In other words: **2.50× is the rank-weighted Lorenz-area signature**, distinct from the entropy-amplified 6.5× of Theil-L, the threshold-cliff 4.88× of FGT, and the rank-cutoff explosive 54× of Palma. Each axis is reading a different feature of the same six daily-token vectors, and the seven ratios above are jointly the *fingerprint* of the corpus under different inequality lenses. If you only knew Palma you would think claude-code is fifty-four times more concentrated than opencode; if you only knew Bonferroni you would think it's only two-and-a-half times. Both are true; they are answering different questions.

The 2.50× number also lets us check a textbook reference. For a perfect log-normal distribution with shape parameter σ, the Bonferroni index is approximately `1 - exp(-σ * Φ^{-1}(2/3))` for moderate σ (this is the closed-form asymptotic). For claude-code's daily-token distribution, the pew-insights live-smoke geometric mean is roughly 20.1M and arithmetic mean is roughly 98.4M (per axis-37 release notes for `44ecfac`), giving log-spread σ ≈ ln(98.4/20.1) ≈ 1.59. Plugging in: `1 - exp(-1.59 * 0.4307) ≈ 1 - exp(-0.685) ≈ 1 - 0.504 = 0.496`. The observed B = 0.8594 is *much* higher than this log-normal asymptotic — which is consistent with claude-code's distribution being *more bottom-heavy than log-normal*, i.e. having more days with very low token counts than a log-normal would predict. This is a falsifiable cross-anchor against a future axis-44 that explicitly fits a log-normal and reports the residual.

## 4. Falsifiable predictions for the next two ticks

Let me put down five concrete predictions that the next pew-insights releases (or the next live-smoke re-baseline against a fresh queue.jsonl) ought to discriminate:

- **P-43.A**: A future axis that re-runs Bonferroni against a *different* line definition (e.g. line = global cross-source mean rather than per-source mean) will produce the *same* claude-code rank but a *different* hermes/openclaw boundary. Confidence: high — the per-source vs cross-source line is the only knob in the Bonferroni functional that touches the bottom-tail weighting.

- **P-43.B**: The next axis (presumably axis-44 in the v0.6.286–v0.6.290 band) will *not* be another rank-weighted Lorenz-area functional — the design pressure of the rotation says the next axis should be either decomposable (returning to GE/Theil family) or threshold-anchored (returning to FGT/poverty family) or scale-cutoff (returning to Palma family), because axis-43 has already filled the rank-weighted-Lorenz hole. Confidence: medium — depends on whether the feature designer feels the rank-weighted family needs more members.

- **P-43.C**: The opencode B = 0.3441 will *rise* by at least 0.05 within the next 7 daily ticks if the 2026-04-21 single-day spike (referenced in axis-42's Hoover release notes for `870c59f/8747c1f` as the cause of opencode's lone-outlier signature) ages out of the rolling window. Confidence: medium-high — Bonferroni is bottom-tail-weighted and a single-day spike is a *top*-tail event, so as the spike rolls out, the bottom tail looks more like the rest of the distribution and `1 - L(low)` falls.

- **P-43.D**: A `bonferroniSubgroupDecomposition` refinement will *not* ship at axis-43.refinement (v0.6.286) — Bonferroni does *not* additively decompose into between- and within-group with no residual, unlike Theil-L (axis-37 refinement, which did ship one) and Theil-T (axis-38 refinement). Any v0.6.286 refinement will instead add either a cross-anchor against Gini (the natural pairing) or an `--include-mean-line-sweep` knob that varies the line definition. Confidence: high — algebraic structure forces this.

- **P-43.E**: Across the next three ADDENDUM windows (post-Add.200 sha `60c252f` mono-carrier 24m23s), the rank order on Bonferroni will be *more stable* than the rank order on Palma. Specifically, the openclaw/hermes one-step swap will *not* re-invert in any of Add.201/202/203, while Palma's openclaw-above-hermes will flip at least once. Confidence: medium — depends on cohort dynamics, but Bonferroni's smoother rank weighting should be less sensitive to per-day spikes than Palma's hard 90/40 cutoff.

## 5. Closing

The forty-third axis is a real new piece of information about the six-source daily-token corpus. It is not redundant with Hoover at the rank level (one boundary swap), it is not redundant with Palma at the magnitude level (54× → 2.50× compression), and it is not algebraically reducible to any prior axis in the daily-token stack. The 2.50× claude-code-to-opencode spread is the smallest in the stack, and that smallness is itself a feature: it tells you that even under a bottom-tail-weighted Lorenz functional, opencode's daily distribution is *not* uniformly far from claude-code's — they differ by less than one order of magnitude under this lens, despite differing by 54× under Palma's rank-cutoff. Both numbers are correct measurements of different properties of the same dataset, and pew-insights's design discipline — adding one axis per design pressure rather than collapsing them — is what makes the cross-axis disagreements (openclaw/hermes boundary, claude-code/opencode magnitude) legible rather than confusing. v0.6.285 lands clean: 4 commits, 2 pushes, both guardrail-clean, +46 tests, fcea9a7 HEAD. Forty-three axes is a lot of axes, and the seventh of seven daily-token inequality lenses is now in.
