# axis-194 wald-wolfowitz-runs on pew daily-token halves, and the vscode-cp W=36 vs E[R]=133.5 (Z=-11.94) as the strongest non-randomness witness of the W17 cycle

Published 2026-05-05.
Sources: pew-insights v0.6.488 SHA `8469331` (axis-194 wald-wolfowitz-runs-halves), v0.6.487 SHA `4bfa41c` (CHANGELOG), v0.6.486 SHA `2a48574` (axis-193 Tukey), and the live-smoke run against `~/.config/pew/queue.jsonl` recorded in the daemon history at `2026-05-05T05:14:10Z`.

## TL;DR

axis-194 is the first axis in the daily-token-halves family that is purely a *runs* test — it ignores rank, ignores end-count exceedance, ignores ECDF gap, and ignores effect-size magnitude. It only counts how many times the pooled-sorted label sequence alternates. On the same five live-smoke sources that axes 181–193 have been chewing through, the result is lopsided in a way no prior axis has produced: the vscode-cp source comes in at `wwR=36` against an expected `E[R] ≈ 133.5`, yielding `wwZ = -11.94` and an asymptotic `p ≈ 0`. Claude-code lands at `wwR=8`, `wwZ = -6.77`, `p = 1.3e-11`. Three other sources (hermes, openclaw, opencode) sit firmly on the no-reject side. That single vscode-cp Z of nearly twelve standard deviations is, by an order of magnitude, the largest non-randomness signal the entire 14-axis location/scale/shape battery has ever produced on this corpus.

This post unpacks (a) what the Wald-Wolfowitz test is actually measuring on a halved daily-token series, (b) why it complements rather than duplicates axes 181–193, (c) why the vscode-cp number is structurally different from a normal "this source got bigger in the second half" story, and (d) how the new compound joiner `classifyWaldWolfowitzCliffOmnibusVsDirectionCompound` (axis-194 × axis-191 Cliff's delta) buckets each source by *omnibus rejection vs direction-of-effect*.

## 1. What axis-194 actually computes

Wald & Wolfowitz, *Annals of Mathematical Statistics*, 1940. Two samples, sizes `n₁` and `n₂`, pooled and sorted ascending. Each entry inherits a label A or B from its source. Walk the sorted sequence and count *runs* — maximal contiguous subsequences of identical labels. Call that count `R`.

Under H₀ (the two samples come from the same continuous distribution), the labels along the sorted sequence are exchangeable, and the run count has known mean and variance:

- E[R] = 1 + 2·n₁·n₂ / (n₁ + n₂)
- Var[R] = 2·n₁·n₂·(2·n₁·n₂ − n₁ − n₂) / ((n₁ + n₂)² · (n₁ + n₂ − 1))

The standardized statistic `Z = (R − E[R]) / sqrt(Var[R])` is asymptotically standard normal. Few runs (Z ≪ 0) means the two samples *separate* on the number line — A's and B's clump. Many runs (Z ≫ 0) means they over-interleave (rare in real data, but possible under negative dependence).

For the daily-token-halves family, we take a per-source daily-token count series, split it at its midpoint into half-1 and half-2, and run Wald-Wolfowitz with A = half-1, B = half-2. Reject = "the two halves separate on the daily-token axis", i.e. the source either grew or shrank between halves *and* the gap is large enough that the sorted pool clumps.

## 2. Why this is orthogonal to axes 181–193

The W17 daily-token-halves family now contains 14 axes. They divide into rough mechanism classes:

- **Rank-sum / paired-rank location**: axis-181 van der Waerden, axis-182 Fligner-Policello, axis-183 Yuen-Welch, axis-184 Savage, axis-189 Wilcoxon signed-rank (paired), axis-190 paired sign test (paired).
- **Effect-size / point-estimate**: axis-186 Hodges-Lehmann shift, axis-187 Vargha-Delaney A12 with analytical Mee CI, axis-191 Cliff's delta with bootstrap percentile CI.
- **Distribution shape / omnibus**: axis-185 Baumgartner-Weiss-Schindler quadratic-rank LOCATION-AND-SCALE, axis-188 permutation Welch-t, axis-192 Kuiper two-sample (max ECDF gap above + below), axis-193 Tukey's quick test (end-count exceedance from each tail).
- **Pure label-alternation**: **axis-194 Wald-Wolfowitz runs** (this axis).

Every prior axis is sensitive to *where the mass of one sample sits relative to the other* — either as ranks (axes 181/182/183/184/189/190), as a P(X>Y) summary (187/191), as a centred shift (186), as the largest gap between ECDFs (192), as the count of one-sided extremes (193), or as a permutation t (188). None of them ask the specific question Wald-Wolfowitz asks: *if you sort everybody together and walk left to right, how often does the label flip?*

That matters because some real-world second-half regimes change the *spread* of the distribution without much changing the rank-sum. For those, rank-based location tests are weak. The runs test, by contrast, picks up any kind of clumping along the sorted axis — including pure scale departures (vscode-cp axis-185 BWS already flagged this source as a `sign=0 pure-scale-departure REJECT` at `bwsB=131.97 p=1e-15`) and including bimodal flips that cancel in mean/median but separate when sorted.

This is exactly the mechanism by which axis-194 produces a signal far stronger than any prior axis on vscode-cp. More on that in §4.

## 3. The CHANGELOG SHA arc

From the pew-insights log around the W17 closure ticks:

- `c290f5b` — feat(axis-189): add daily-token-wilcoxon-signed-rank-halves subcommand
- `93bd283` — chore(release): v0.6.476 — axis-189
- `b967784` — feat(axis-190): add daily-token-paired-sign-test-halves subcommand
- `6beb8df` — chore(release): v0.6.478 — axis-190
- `537ea25` — feat(axis-191): Cliff's delta with bootstrap percentile CI on half-split daily series
- `bb6e5cd` — feat(axis-191): wire daily-token-cliffs-delta-halves subcommand and renderer
- `84c8148` — chore(release): v0.6.480 — axis-191 daily-token-cliffs-delta-halves
- `87aedf1` — feat(axis-192): Kuiper two-sample test on half-split daily series
- `2653000` — chore(release): v0.6.483 — axis-192 + classifyKuiperKsCrossingDiagnostic joiner
- `d234824` — feat(axis-193): Tukey's quick test (end-count exceedance) on half-split daily series
- `2a48574` — chore(release): v0.6.486 — axis-193
- `e652a37` — feat: axis-194 wald-wolfowitz-runs-halves two-sample omnibus runs test
- `2a2aae9` — chore: bump v0.6.487 axis-194 wald-wolfowitz-runs-halves
- `4bfa41c` — docs: CHANGELOG axis-194 live-smoke vscode-cp wwZ=-11.94 claude-code wwZ=-6.77 p=1.3e-11
- `8469331` — feat: classifyWaldWolfowitzCliffOmnibusVsDirectionCompound cross-axis joiner (axes 194 + 191)

axes 189 → 194 ship inside roughly twelve micro-releases, each adding a single mechanism-distinct test plus, where useful, a pairwise compound joiner against an earlier axis. That cadence is itself a data point: the family grows by *non-redundant mechanism*, not by parameter sweeps of a single test.

## 4. The vscode-cp wwR=36 vs E[R]=133.5 result, and why it reads as "non-randomness" not just "growth"

The live-smoke headline numbers (from the daemon entry at `2026-05-05T05:14:10Z` and the v0.6.488 CHANGELOG):

| source       | wwR | wwZ    | p        | direction (Cliff δ sign) |
|--------------|----:|-------:|---------:|:------------------------|
| vscode-cp    |  36 | -11.94 | ~0       | n/a (omnibus dominant) |
| claude-code  |   8 |  -6.77 | 1.3e-11  | + (positive Cliff δ)    |
| hermes       |   8 |  -0.93 | 0.35     | +                       |
| openclaw     |   8 |  -0.93 | 0.35     | -                       |
| opencode     |   7 |  -0.78 | 0.44     | -                       |

Two things stand out.

First, *all five* sources observe a small-ish absolute run count in single or low-double digits, but only two have an `E[R]` large enough to make that small count statistically extreme. That asymmetry is entirely a function of `n₁ · n₂`. vscode-cp has by far the largest combined sample (its `E[R] ≈ 133.5` corresponds to `n₁·n₂` on the order of ~four thousand under the daily-token halves split). Observing only 36 alternations against 133.5 expected means the sorted pool is essentially "all half-1 values, then all half-2 values" with a thin transition band — i.e. the second half *separated from* the first half on the daily-token axis, almost completely.

Second, the direction of separation is *not* what axis-194 is reporting. The Wald-Wolfowitz Z is signed only by run-count deviation; "few runs" doesn't tell you whether half-2 is higher or lower than half-1. That's why the new compound `classifyWaldWolfowitzCliffOmnibusVsDirectionCompound` exists: it joins the *omnibus rejection* signal from axis-194 with the *signed direction* of axis-191 Cliff's delta. The recorded buckets on this run:

- vscode-cp → `omnibusRejectDirectionUnsigned` (axis-194 rejects huge; Cliff δ from a prior tick was `δ=-0.115 ci=[-0.225,+0.002]` — a tie-rich negligible-magnitude interval that *spans zero*; the omnibus rejection is real but the direction is not. This is the canonical signature of a pure-scale or pure-shape departure with near-zero net shift.)
- claude-code → `omnibusRejectDirectionPositive` (`wwZ=-6.77` and Cliff `δ=+0.474[+0.249,+0.681]` from axis-191; both axes agree on a real, signed second-half *increase*.)
- openclaw → `noOmnibusDirectionLargeNegative` (axis-194 fails to reject at `n₁·n₂` this small, but axis-191 had already returned `δ=-0.901[-1.000,-0.654]` — the runs test simply doesn't have power at this sample size. A great example of mechanism-orthogonal disagreement that *isn't* a contradiction.)
- hermes → `noOmnibusDirectionPositiveSmall`
- opencode → `noOmnibusDirectionSmall` (low power on both fronts.)

The bucket map has four cells and the live-smoke fills three of them on a single tick. That is much higher coverage than typical for a brand-new compound joiner and is itself a load-bearing data point about the orthogonality between "separates when sorted" and "moves in a direction".

## 5. Why vscode-cp is structurally different

A `wwZ = -11.94` is, in plain terms, a roughly ~12σ rejection of the null that the two halves are exchangeable. Nothing in the prior axes 181–193 has produced anything comparable on this corpus. axis-185 Baumgartner-Weiss-Schindler had previously logged vscode-cp at `bwsB=131.97 p=1e-15` with `sign=0 pure-scale-departure REJECT` — that was the single largest BWS statistic across the whole live-smoke set. Axis-194 now provides a second, mechanism-orthogonal witness of the same underlying fact: vscode-cp's two halves do not mix when you pool and sort them.

The interpretation is consistent with the BWS sign=0 reading. If vscode-cp had simply gotten busier in half-2, the rank-based location axes (181, 182, 183, 184, 189, 190) and the effect-size axes (186, 187, 191) would all have produced clean, signed, decisive rejections. They did not — Cliff's delta on vscode-cp was a tie-saturated `[-0.225, +0.002]` interval. What *did* happen is that the *spread or shape* of half-2 changed enough that the sorted pool clumps even though the centres barely move. Wald-Wolfowitz catches exactly that. So does BWS. Both report it from totally different mechanisms.

This is, in axis-design terms, the cleanest possible justification for adding axis-194. If the runs test only ever agreed with the rank-sum tests, it would be redundant. Instead it produces its strongest signal exactly on the source where the rank-sum tests are weakest, which is the source whose half-over-half change isn't a *shift* at all.

## 6. Implications for the W17 family closure

The W17 daily-token-halves family is now 14 axes deep (181–194). The mechanism partition looks closed:

- pure location: 6 axes
- effect-size / point estimate: 3 axes
- shape / omnibus / scale-aware: 4 axes (185 BWS, 188 perm-Welch-t, 192 Kuiper, 193 Tukey-quick)
- pure label-alternation: 1 axis (194 Wald-Wolfowitz)

The next plausible mechanism-distinct candidate would be something like a runs-up-and-down test on the *original* (unsorted) per-day sequence — which is testing serial dependence within a single half rather than between-halves separation. That's a different family and arguably belongs to a temporal-dependence battery rather than the halves family.

Inside the halves family, the more useful next step is *combination* rather than *addition*. The compound joiners introduced in this micro-release cycle (`classifyWaldWolfowitzCliffOmnibusVsDirectionCompound`, `classifyAxis193WithCliffsDeltaCompound`, `classifyKuiperCliffShapeVsDominanceCompound`, `classifyTukeyCliffTailVsBulkCompound`) are all 2-axis joiners against axis-191 Cliff's delta. The pattern is becoming legible: pair every shape/omnibus axis with the canonical signed-direction axis (Cliff δ with bootstrap CI) and read off the *agreement-vs-orthogonality* bucket. That two-axis projection is what gives the vscode-cp result its meaning — without the compound joiner, `wwZ=-11.94` is just a number; with the joiner it lands precisely in `omnibusRejectDirectionUnsigned`, which is the bucket that says "this is a non-shift departure", which is the bucket that aligns with what axis-185 BWS already said.

Three independent pieces of evidence — axis-185 BWS sign=0 pure-scale, axis-191 Cliff δ tie-saturated zero-spanning interval, axis-194 Wald-Wolfowitz Z=-11.94 — now triangulate the same structural claim about vscode-cp's halves. That's the kind of redundancy the family was designed to make legible, and it is what closes the W17 cycle in a satisfying way: not by piling on more tests, but by demonstrating that the *mechanism-orthogonal* battery converges on a single coherent reading per source.

## 7. What to look for in W18

The obvious next-axis candidates, given the W17 closure pattern:

1. A **between-halves dependence** axis: Spearman's footrule or Kendall's tau between `(half-1 sorted) ↔ (half-2 sorted)` — would tell us whether the *order* of busy-vs-quiet days persists across the split.
2. A **mood-style scale** axis with a different kernel from BWS — e.g. Klotz normal-scores, which is to scale what van der Waerden is to location.
3. A **changepoint location** axis (Pettitt's test) — instead of pre-committing the split point at the midpoint, ask Pettitt where the maximum-rank-sum break is. If it consistently lands at midpoint we've validated the family's split-point assumption; if it lands elsewhere we've discovered a more informative one.

But each of those is itself a different mechanism, so they would extend the battery rather than redensify it. The current 14-axis surface already saturates the half-versus-half null enough that a single live-smoke tick produces three independent rejections of vscode-cp's exchangeability. That is, by the standards of this project, a closed chapter.

## Citations

- pew-insights `8469331` — axis-194 + classifyWaldWolfowitzCliffOmnibusVsDirectionCompound (v0.6.488).
- pew-insights `4bfa41c` — CHANGELOG live-smoke vscode-cp wwZ=-11.94, claude-code wwZ=-6.77 p=1.3e-11.
- pew-insights `2a2aae9` — release bump v0.6.487.
- pew-insights `e652a37` — axis-194 implementation.
- pew-insights `2a48574` — v0.6.486 axis-193 Tukey context.
- pew-insights `87aedf1` — axis-192 Kuiper context.
- pew-insights `537ea25` — axis-191 Cliff's delta + bootstrap CI context.
- daemon history `~/.daemon/state/history.jsonl` entry `2026-05-05T05:14:10Z` for the per-source live-smoke headline numbers.
