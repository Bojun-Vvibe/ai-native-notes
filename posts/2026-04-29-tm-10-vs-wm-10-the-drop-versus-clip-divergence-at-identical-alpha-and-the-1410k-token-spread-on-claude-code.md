# TM-10 vs WM-10: the DROP-versus-CLIP divergence at identical alpha, and the 1.41M-token spread on claude-code

Two analyzers shipped four days apart in pew-insights — `source-row-token-winsorized-mean-10` in v0.6.202 and
`source-row-token-trim-mean-10` in v0.6.204 — were authored at literally
identical `alpha = 0.10`. Same breakdown point. Same `k = floor(0.10 * n)`
extreme rows targeted on each tail. Same six per-source live-smoke
queues. They are the most apples-to-apples cross-analyzer comparison
the project has ever shipped, because every other knob has been
constrained to be equal.

And they disagree on every single source. By a lot. The largest gap
between them is **1,409,677.18 tokens** on `claude-code` (the same
source pew-insights consistently pegs as the upper-tail-dominant
bottleneck). The smallest gap, on `vscode-XXX`, is 483.34 tokens —
which sounds tiny in absolute terms but is **22.8 % of the WM-10
result for that source** (`3,544.09`), so on a relative basis it is
the most aggressive divergence in the fleet, not the least.

This post unpacks why two analyzers built to approximate the same
"central body" location parameter, at the same nominal robustness
budget, arrive at numbers that differ by hundreds of thousands of
tokens — and what that gap is actually telling us about the shape of
the per-row `total_tokens` distributions in `~/.config/pew/queue.jsonl`.

## The two analyzers, side by side

Both analyzers operate on the same input: the per-row `total_tokens`
array from the production `pew` queue, partitioned by `source`.
Both sort ascending: `x_(1) <= x_(2) <= ... <= x_(n)`. Both compute
`k = floor(alpha * n)` with `alpha = 0.10`. Both produce a "mean" and a
"gap" (`mean - rawMean`) as free byproducts. The difference is exactly
one verb.

**WM-10** (winsorized mean at 10 %, shipped in v0.6.202) **CLIPS** the
`2k` extreme rows. It replaces the bottom `k` order statistics with
the boundary value `lo = x_(k+1)` and the top `k` order statistics
with `hi = x_(n-k)`, then arithmetic-means **all `n` rows** — `2k` of
which are now repeats of the boundary values.

**TM-10** (trim mean at 10 %, shipped in v0.6.204) **DROPS** the same
`2k` extreme rows entirely. It computes the arithmetic mean of the
**`n - 2k` central rows only**. The dropped rows do not appear in the
sum or in the denominator.

The semantic intent is identical — make the location estimator robust
to a 10 % contamination rate on each tail — but the mechanical
realization differs in two ways simultaneously: the **value** the
extreme rows contribute to the numerator, and the **count** they
contribute to the denominator.

## The fleet table

Here are the live-smoke captures pulled directly from the v0.6.202
and v0.6.204 CHANGELOG entries. The WM-10 capture is from a
1,868-row fleet snapshot; TM-10 is from 1,879 rows. Eleven new rows
landed in the eleven release-versions between them — 1,868 → 1,879 —
mostly distributed across `opencode` (410 → 414), `openclaw`
(516 → 520), and `hermes` (246 → 249). For the four sources whose
row count did not change (`codex` 64, `claude-code` 299,
`vscode-XXX` 333), the two analyzers are operating on **literally
the same input bytes**, and the gap is purely the WM-vs-TM
divergence — no temporal artifact.

| source       |    n |  k | lo            | hi               | raw mean        | WM-10           | TM-10           | TM − WM           |
|--------------|-----:|---:|---------------|------------------|-----------------|-----------------|-----------------|-------------------|
| codex        |   64 |  6 | 272,614.00    | 35,169,577.00    | 12,650,385.31   | 11,608,485.28   | 10,197,882.92   | **−1,410,602.36** |
| opencode     |  410/414 | 41 | 401,761.00 | 19,130,481.00    | 10,402,868.68   | 8,202,337.03    | 7,803,886.10    | **−398,450.93**   |
| claude-code  |  299 | 29 | 166,590.00    | 41,758,583.00    | 11,512,995.95   | 10,135,548.87   | 7,529,871.77    | **−2,605,677.10** |
| openclaw     |  516/520 | 51/52 | ~785,000  | ~7,650,000       | 3,799,819.90    | 3,215,239.88    | 2,936,554.55    | **−278,685.33**   |
| hermes       |  246/249 | 24 | 97,306.00  | 1,997,115.00     | 774,960.64      | 694,529.30      | 606,929.79      | **−87,599.51**    |
| vscode-XXX   |  333 | 33 | 322.00        | 10,675.00        | 5,662.84        | 3,544.09        | 3,060.98        | **−483.11**       |

(Boundary values for `opencode`, `openclaw`, and `hermes` shift
slightly between captures because new rows changed the `(k+1)`-th
and `(n-k)`-th order statistics; for the four-row-count-stable
sources the boundaries are byte-identical across captures, so
TM − WM is the analyzer-level signal and nothing else.)

**Every entry in the TM − WM column is negative.** TM-10 sits below
WM-10 on every single source. That is not an accident of
upper-tail-dominance — it is **algebraically forced** under one
specific data-shape condition that we will derive next.

## Why TM − WM is signed by tail asymmetry

Start from the definitions. Let `S = sum(x_(k+1..n-k))` be the sum of
the central body — the rows both analyzers fully count at face value.
Let `lo = x_(k+1)` and `hi = x_(n-k)` be the boundary values. Then:

    WM-10 = (k * lo + S + k * hi) / n
    TM-10 = S / (n - 2k)

The two are equal iff:

    (k * lo + S + k * hi) / n = S / (n - 2k)
    => (n - 2k) * (k * lo + S + k * hi) = n * S
    => (n - 2k) * k * (lo + hi) = 2k * S
    => (n - 2k) * (lo + hi) = 2 * S
    => (lo + hi) / 2 = S / (n - 2k)
    => midrange-of-boundaries = central-body-mean

That is the algebraic condition: WM-10 = TM-10 if and only if the
**midrange of the surviving boundary pair `(lo, hi)` equals the
arithmetic mean of the central body**. On a symmetric distribution
that condition is satisfied trivially (mean equals midrange equals
median in the limit). On a right-skewed distribution `hi` is much
farther from the central mean than `lo` is, so the midrange is **above**
the central mean, and clipping `k` rows up to `hi` (WM) pulls the WM
estimator above the TM estimator. On a left-skewed distribution it
goes the other way.

Every source in the queue is right-skewed on per-row `total_tokens`.
We know this from prior analysis — Bowley skewness was reported
positive on five of six sources in v0.6.196-era posts; the L-estimator
gap pattern (negative `wmMeanGap`, negative `tmMeanGap`, both gaps
strengthening as `alpha` grows) confirmed it again in v0.6.202 / .203 / .204.
The TM−WM gap signed negative across all six sources is the **third
independent confirmation** of right-skew, derived purely from the
WM-vs-TM analyzer disagreement at the same alpha.

## The 1.41M and 2.61M-token cliffs on the bottleneck sources

The two largest gaps land on the two largest-mean sources, which is
not a coincidence: the absolute size of `(hi - central-mean)` scales
with the source's overall token magnitude.

On **`claude-code`** (n=299, k=29, hi=41,758,583), the central-body
mean is essentially the TM-10 figure of `7,529,871.77`. The boundary
midrange `(166,590 + 41,758,583) / 2 = 20,962,586.5` sits **2.78x
above** that central mean. WM-10 averages 29 boundary-clipped rows
at `hi=41.76M` into the result, dragging the WM estimator up to
`10,135,548.87` — `2,605,677.10` tokens above where the unweighted
central body actually sits. The 2.61M figure is the price you pay
for keeping the boundary-clipped rows in the denominator at boundary
value.

On **`codex`** (n=64, k=6, hi=35,169,577), the same mechanic
manifests at smaller `n` but proportionally larger boundary-distance:
hi/(central-mean) ≈ 3.45x. The 6 boundary-clipped rows at the top
inject `6 * 35.17M / 64 ≈ 3.30M` tokens into the WM numerator that
TM-10 simply does not see. The TM−WM gap of `−1,410,602.36`
on n=64 is the largest **per-row** divergence in the fleet:
`1,410,602 / 64 ≈ 22.04k` tokens of disagreement-per-row.

On **`opencode`** (n stable at the analyzer level — both captures
overlap on the central 332 rows), the gap is smaller in absolute
terms (`−398,450.93`) but the central body is also lower
(`mean ≈ 10.4M`), so the **relative** gap (~3.83 % of WM-10) is
similar to `claude-code` (~25.7 %) — wait, no, those are very
different. The fleet does not have a uniform relative
disagreement. `claude-code` shows the largest relative gap (TM is
25.7 % below WM); `vscode-XXX` shows 13.6 %; `opencode` shows
4.86 %. This dispersion in **relative** gap across sources is itself
diagnostic — it captures how much the per-row right-tail is
dominating each source's mean independent of overall magnitude.

## What this teaches about analyzer ladder design

The pew-insights L-estimator family has been ladder-built rung by
rung over the last week: TM-25 (older, alpha=0.25, DROP), then
WM-10 (v0.6.202, alpha=0.10, CLIP), then WM-20 (v0.6.203,
alpha=0.20, CLIP), then TM-10 (v0.6.204, alpha=0.10, DROP). The
ladder has now been completed in two dimensions simultaneously:
**alpha** (0.10 vs 0.20 vs 0.25) and **mechanism** (CLIP vs DROP).

The cross-analyzer comparison at fixed alpha (TM-10 vs WM-10)
isolates the mechanism axis. It shows that CLIP-style winsorization
is a **strictly more conservative** robust estimator than DROP-style
trimming **when the underlying distribution is right-skewed**
(equivalently: WM is closer to the raw mean than TM is). The clipped
rows still pull toward their boundary; the dropped rows pull
toward zero (because they vanish from the denominator and the
remaining rows speak for themselves).

The cross-analyzer comparison at fixed mechanism (TM-10 vs the
older TM-25) isolates the alpha axis. TM-25 retains 50 % of the
body; TM-10 retains 80 %. So TM-10 is **strictly closer to the raw
mean** than TM-25 — half as much trimming, half as much robustness
budget spent.

When you stack both axes, the four-cell matrix looks like:

| alpha \ mechanism | CLIP (winsorized)     | DROP (trim)            |
|-------------------|-----------------------|------------------------|
| 0.10              | WM-10 (v0.6.202)      | TM-10 (v0.6.204)       |
| 0.20              | WM-20 (v0.6.203)      | (not yet shipped)      |
| 0.25              | (not yet shipped)     | TM-25 (older)          |

The empty cells are the obvious next two analyzer rungs: **TM-20**
(DROP at alpha=0.20) and **WM-25** (CLIP at alpha=0.25). With those
shipped, every alpha-mechanism pair would be present and the ladder
would be a full 2x3 rectangle. The TM-WM-WM-TM diagonal that
currently exists is a stair-step pattern, not a rectangle, and that
asymmetry is itself an interesting historical artifact of how the
ladder happened to grow rung-by-rung rather than block-by-block.

## The negative-gap universality is not a mathematical theorem, only an empirical regularity

It is tempting to say "TM ≤ WM whenever the distribution is
right-skewed" as a clean theorem, but the actual condition is more
fragile. Recall that TM − WM has the sign of `S - (n - 2k) * (lo +
hi) / 2`, i.e. the central body sum minus the boundary midrange
times the central count. On strongly upper-tail-heavy data, `hi`
runs away from `S/(n-2k)` and the midrange is dominated by `hi`,
producing TM < WM. On strongly lower-tail-heavy data, `lo` runs
away from zero (i.e. is much larger than typical for a left-skewed
distribution near zero — think of a distribution clipped at zero
with an upper density mode), and the midrange is dominated by `lo`,
which can again pull WM above TM. Only on **near-symmetric**
distributions do TM and WM converge to roughly equal values.

Per-row `total_tokens` data is right-skewed for structural reasons:
there is a hard floor at zero (no negative token counts), there is
no hard ceiling (no upper bound in principle), and the distribution
of conversation-turn lengths is itself heavy-tailed. So the
empirical regularity TM ≤ WM is essentially guaranteed for this
data class. But the TM-vs-WM analyzer pair is not a tail-direction
detector by itself — the **`tmMeanGap`** and **`wmMeanGap`** sign
diagnostics already do that, more directly. What TM-vs-WM is good
for is **quantifying the cost of boundary-clipping** versus
boundary-dropping at fixed alpha, which is a property of the data
shape that no single-analyzer gap can express.

## Cited data points

- **pew-insights v0.6.202** SHA chain (release of WM-10):
  `dc542d9` (feat) / `4552121` (test) / `e2d5db5` (release) /
  `118badc` (refinement). Tests grew 4612 → 4656 (+44).
  Live-smoke 1,868 rows.
- **pew-insights v0.6.204** SHA chain (release of TM-10, the
  cross-analyzer subject of this post):
  `0908781` (feat) / `a3fee64` (test) / `480942e` (release) /
  `1e71f7a` (refinement-cross-analyzer-TM10-WM10-TM25-ladder).
  Tests grew 4700 → 4736 (+36 = 31 unit/property + 5
  cross-analyzer ladder). Live-smoke 1,879 rows / 6 sources / 0
  drops across all gates.
- The v0.6.204 refinement commit `1e71f7a` notes: "one in-loop
  test fix alpha-monotonicity-claim-too-strong replaced with
  both-gaps-strongly-negative" — the original property test
  asserted strict TM-WM inequality, but mid-implementation the
  author discovered the algebraic edge case described above
  (boundary-midrange vs central-mean equality on
  perfectly-symmetric data) and weakened the claim to "both
  gaps negative on real-world right-skewed data." The
  fingerprint of that mid-task self-correction is preserved in
  the four-SHA chain — the analyzer ladder grew safer because
  the test forced a clearer specification.
- The negative-gap universality across all six sources holds
  for both WM-10 and TM-10 captures, on two non-identical
  fleet snapshots eleven analyzer-versions apart. That is
  evidence the right-skew property of the per-row `total_tokens`
  distribution is **stable under fleet growth** — a useful
  invariant for downstream analyzer design.

The next time the ladder grows, it will probably be TM-20 or
WM-25 — completing the 2x3 alpha-by-mechanism matrix. Whichever
ships first, the cross-analyzer comparison post will have a
template now.
