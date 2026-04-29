---
title: The Siegel-vs-Theil-Sen breakdown jump from ~29 % to ~50 % — pew-insights v0.6.216 on the same six-source 1928-row corpus, and what nesting medians actually buys
date: 2026-04-29
---

The pew-insights `0.6.214 → 0.6.216` step is one of the cleanest
controlled comparisons the suite has shipped to date. Both
releases ship a per-source robust slope estimator of per-row
`total_tokens` against row index. Both run against the same real
local `~/.config/pew/queue.jsonl`. Both report on six sources.
Both expose the same `--since`, `--until`, `--source`,
`--min-rows`, `--min-slope-magnitude`, `--max-pairs`, `--top`,
`--sort`, `--json` flag surface. Both report the same byproducts
(`mean`, `median`, `firstX`, `lastX`, `naiveSlope`).

The only thing that changes between `0.6.214` and `0.6.216` is the
robust slope formula. `0.6.214` (Theil-Sen, breakdown ~29.3 %)
takes a single median over the `n*(n-1)/2` unordered pairwise
slopes. `0.6.216` (Siegel repeated medians, breakdown ~50 %)
takes a per-anchor median first, then medians the per-anchor
medians. Theil-Sen does one median pass over `C(n,2)` slopes;
Siegel does two median passes — outer over `n` anchors, inner
over `n-1` slopes per anchor. Same data, two different
median-collapse structures, two different breakdown points. The
result on the live-smoke corpus is interesting because it
tightens some signals and produces one outright sign reversal
relative to Theil-Sen — and the sign reversal happens on the
single source where it most matters operationally.

This post is about what the ~29 % vs ~50 % breakdown gap means in
practice, what the live-smoke tables for v0.6.214 and v0.6.216
show side-by-side, why nesting medians doubles the breakdown,
where the two estimators agree, and where they disagree.

## What "breakdown ~29 %" and "breakdown ~50 %" actually mean

The asymptotic breakdown point of an estimator is, informally,
the largest fraction of arbitrarily-corrupted observations the
estimator can absorb without its output being driven to infinity.
For a slope estimator of n points, breakdown 0 % means a single
contaminated row can swing the slope by an arbitrary amount —
this is the OLS regime, which is what `source-daily-token-trend-
slope` (the original v0.6 lens) gives you. Breakdown 50 % is the
maximal possible breakdown for any equivariant slope estimator;
you cannot do better, by an information-theoretic argument
(Donoho's work on this is the standard citation).

Theil-Sen sits at ~29.3 %. The intuition is that for the median
of `C(n,2)` pairwise slopes to flip sign, you need to corrupt
roughly the `1 - sqrt(1/2) ≈ 0.293` fraction of rows so that more
than half of the pairwise slopes have at least one corrupted
endpoint. Siegel's nested-median construction defeats this by
requiring two breakdowns to compose. To corrupt the outer median,
you need to corrupt at least half the anchor-medians. To corrupt
each anchor-median, you need to corrupt at least half the
per-anchor pairwise slopes. The arithmetic gives ~50 %.

The price is computational and conceptual. Theil-Sen materializes
`C(n,2)` numbers and medians them once. Siegel materializes
`n*(n-1)` ordered slopes (the inner medians use only `j != i`,
not `i < j`), takes `n` per-anchor medians, then medians those
`n` numbers. Cost goes from one big median to one big set of
medians plus an outer median. The CHANGELOG flags this at
`pew-insights/CHANGELOG.md:48`: `pairsTotal = n * (n - 1)` is the
ordered slope evaluation count. So Siegel uses 2x the slope
evaluations of Theil-Sen on the same `n`, plus extra median
overhead.

## The two live-smoke tables, side by side

Here is the v0.6.214 (Theil-Sen) live-smoke table reproduced from
`pew-insights/CHANGELOG.md:288-296`:

```
source            rows  mean         median      first       last         naive        slope         sign  +pairs  -pairs  0pairs
codex             64    12650385.31  7132861.00  2695764.00  8565718.00   +93173.8730  +123739.4516  up    1,184   832     0
claude-code       299   11512995.95  3319967.00  1470723.00   201134.00    -4260.3658   +31445.1270  up    27,793  16,758  0
opencode          427   10428028.63  8078254.00    96926.00  12884867.00  +30018.6408   +14597.1786  up    53,941  37,010  0
openclaw          533    3726644.61  2310411.00   721224.00    600569.00    -226.7951    -5297.7660  down  51,286  90,492  0
hermes            263     758778.88   432218.00  2061198.00    705178.00   -5175.6489     -586.2500  down  16,254  18,199  0
vscode-redacted   333       5662.84     2319.00      458.00      9990.00      +28.7108      +1.9634  up    29,290  25,977  11
```

And the v0.6.216 (Siegel) live-smoke table from
`pew-insights/CHANGELOG.md:67-74`:

```
source            rows  mean         median      first       last        naive        slope         intercept   sign  mMin          mMax           mRange        +anch  -anch  0anch  pairs
codex             64    12650385.31  7132861.00  2695764.00  8565718.00  +93173.87    +105180.27    2591691     up    -872705.53    +1651905.63    2524611.15    46     18     0      4032
claude-code       299   11512995.95  3319967.00  1470723.00    201134.00   -4260.37    +22324.89    1636118     up      -89650.73    +664247.27     753898.00    241     58     0     89102
opencode          430   10414154.00  8018557.00     96926.00   4942077.00  +11294.06    +22305.49    2255706     up    -272666.22    +137936.21     410602.42    344     86     0    184470
openclaw          536    3724538.00  2305940.00    721224.00    883626.00     +303.56    -6369.11    4294402     down -159332.03      +59351.10    218683.14     87    449     0    286760
hermes            266      753798.00   432176.00  2061198.00    189746.00   -7062.08       +20.37    428337     up     -41384.45     +19631.28      61015.73    134    132     0     70490
vscode-redacted   333         5663.00     2319.00      458.00     9990.00      +28.71        +0.62      2247     up        -194.45     +982.22       1176.68    171    162     0    110556
```

The row counts shift slightly (1,919 rows under Theil-Sen at
v0.6.214, 1,928 rows under Siegel at v0.6.216) because the
production queue grew between the two release runs. That is fine
for the comparison — both runs are on the live local production
queue at their respective release moments, and the cohort
composition (six named sources) is identical.

## The slope numbers, side-by-side, with sign-agreement annotation

Here is the same six-source comparison reduced to just the two
slope columns:

| source | Theil-Sen slope (v0.6.214) | Siegel slope (v0.6.216) | sign agreement | magnitude ratio (Siegel/Theil-Sen) |
|---|---:|---:|:---:|---:|
| codex | +123,739.45 | +105,180.27 | agree (up) | 0.85 |
| claude-code | +31,445.13 | +22,324.89 | agree (up) | 0.71 |
| opencode | +14,597.18 | +22,305.49 | agree (up) | 1.53 |
| openclaw | -5,297.77 | -6,369.11 | agree (down) | 1.20 |
| hermes | -586.25 | +20.37 | **DISAGREE** | flip (negative → positive) |
| vscode-redacted | +1.9634 | +0.62 | agree (up) | 0.32 |

Five of six sources agree on slope sign between Theil-Sen and
Siegel. The one that disagrees is `hermes`, and it disagrees in a
specific way: Theil-Sen reports a small negative slope (-586
tokens/row), Siegel reports an essentially-zero positive slope
(+20 tokens/row). The sign disagreement is real but the
practical interpretation under both lenses is the same: hermes is
flat. The Siegel anchor counts make this very clear:
`hermes` has 134 anchors with positive inner-median and 132
anchors with negative inner-median, an essentially perfect 50/50
split. The CHANGELOG flags this directly at line 92-95: "exactly
the signature of noise around zero trend, even though the naive
endpoint slope reads -7,062." Theil-Sen does not surface this
50/50 anchor split because it doesn't compute per-anchor inner
medians at all — it just reports the median of the global pair
pool.

This is the first concrete payoff of the nested-median
construction visible on real data: Siegel exposes the per-anchor
disagreement structure that Theil-Sen flattens into a single
number. On `hermes`, the per-anchor disagreement is so symmetric
that the outer median lands essentially on zero — which is a
truer description of "this source has no trend" than Theil-Sen's
"this source has a small negative trend" reading.

## Where the two estimators agree but Siegel adds a diagnostic

The four other up-trending sources (`codex`, `claude-code`,
`opencode`, `vscode-redacted`) all agree on the up sign. The
magnitudes shift a bit. Some get smaller (codex 0.85x,
claude-code 0.71x, vscode-redacted 0.32x), one gets larger
(opencode 1.53x). The single down-trending source (`openclaw`)
gets slightly steeper under Siegel (1.20x) and the anchor-sign
counts make this concrete: 449 of 536 anchors (84 %) report a
negative inner median. The CHANGELOG describes this at line 88-91
as "a strong consensus that Theil-Sen alone would have flagged
less crisply" — the point being that Theil-Sen's pair counts
under v0.6.214 (51,286 positive vs 90,492 negative on openclaw)
are a 36/64 split, while Siegel's anchor counts are a 16/84 split.
The Siegel split is more decisive because the per-anchor median
collapses local variation before the outer count is taken.

The `perAnchorMedianMin / Max / Range` columns are a diagnostic
that Theil-Sen literally cannot produce, because Theil-Sen does
not compute per-anchor inner medians at all. They surface the
heterogeneity of trend across the series. `codex` has a per-anchor
range of 2,524,611 — the widest in the cohort — which the
CHANGELOG flags at line 80-84 as "anchor-sensitive": codex's
trend is real but a single-anchor estimate (which is essentially
what some bootstrap or jackknife approximations do) would have
landed somewhere within a 2.5M-token-per-row window depending on
which anchor was chosen. `vscode-redacted`, by contrast, has a
per-anchor range of 1,176 — narrow relative to its slope, meaning
the trend is locally consistent throughout the series.

## Cost of doubling breakdown

Theil-Sen on 533 rows materializes `C(533,2) = 141,778` slopes,
then takes one median (CHANGELOG row for openclaw at v0.6.214
shows 51,286 + 90,492 = 141,778 pairs). Siegel on 536 rows
materializes `536 * 535 = 286,760` ordered slopes — exactly 2x
the unordered count, as expected — then takes 536 inner medians,
then medians those. So Siegel does 2x the slope evaluations and
~537x more median operations than Theil-Sen on the same data.
For n = 536 this is fine; the CHANGELOG sets `--max-pairs` to
`5,000,000` by default which corresponds to roughly `n = 2236`,
where Siegel's `n*(n-1)` would land at ~5M.

The pair counts in the live-smoke tables are 4,032, 89,102,
184,470, 286,760, 70,490, and 110,556 — all comfortably under
the 5M cap. The CHANGELOG explicitly calls this out at line 78-79.

What the cost buys: a slope estimator that absorbs corruption
into nearly half of the rows before the output is destroyed. For
the production queue this is mostly insurance — the queue is
not adversarially contaminated — but it shows up in two visible
ways on the v0.6.216 table:

1. The `hermes` flip (-586 → +20). Theil-Sen's small negative
   reading was driven by a minority of anchors in the local
   pair-pool; Siegel's nested median says the per-anchor
   medians split 134/132 around zero, so the slope is zero. The
   Siegel reading is closer to the truth that hermes has no
   trend.
2. The `openclaw` anchor consensus (449/87 anchors negative,
   ratio 84:16) is sharper than the Theil-Sen pair consensus
   (90,492/51,286 negative, ratio 64:36). Same direction, but
   Siegel's nested-median structure says "even after taking
   the per-anchor inner median, 84 % of anchors still see a
   negative slope" — a stronger claim than "across all pairs
   with at least one corrupted endpoint, 64 % are negative."

## What this lens slot in the suite now looks like

After v0.6.216 the slope/trend family has three distinct lenses:

- `source-daily-token-trend-slope` (OLS, breakdown 0 %, daily
  aggregates).
- `source-row-token-mann-kendall-trend` (rank-correlation test,
  reports tau and p-value, not slope magnitude).
- `source-row-token-theil-sen-slope` (R-estimator, breakdown
  ~29.3 %, single median over `C(n,2)` pair pool, reports slope
  magnitude in tokens/row).
- `source-row-token-siegel-slope` (R-estimator, breakdown ~50 %,
  nested medians, reports slope magnitude plus per-anchor
  diagnostic).

Two of the four (Mann-Kendall and Theil-Sen) ship together and
are mechanically related — Theil-Sen's `pairsPositive -
pairsNegative` is exactly Mann-Kendall's `S` statistic. Siegel
adds a third axis to that picture: not just the test (Mann-
Kendall) and the point estimator (Theil-Sen), but a maximally-
robust point estimator with the per-anchor median spread as a
free diagnostic.

The CHANGELOG at line 24-27 calls Siegel "the first ~50 %-
breakdown slope estimator in the suite, and the first NESTED-
MEDIAN (median-of-medians) estimator in the suite." The first
claim is a fact about robustness; the second is a fact about
algorithmic structure. Both matter, and both are visible in the
live-smoke output: the breakdown jump shows up in the `hermes`
sign flip, and the nested-median structure shows up in the
per-anchor `mMin / mMax / mRange` columns that no previous lens
in the suite produces.

## Closing observation: the same-corpus comparison is rare and
valuable

Most of the pew-insights releases compare a new lens to its
nearest sibling on a single live-smoke run from the new release.
What makes the `0.6.214 → 0.6.216` step interesting is that the
two lenses ship two minor versions apart and both have published
live-smoke tables on essentially the same corpus (1,919 rows
versus 1,928 rows, six sources, same names, same local
`~/.config/pew/queue.jsonl`). That makes the comparison much
crisper than a typical "we shipped a new lens, here is the
output" release note. You can read both tables, line up the six
sources, and see exactly where the two estimators agree and
disagree.

The summary: Theil-Sen and Siegel agree on slope sign for five of
six sources. They disagree on the one source (`hermes`) that is
flat enough that the disagreement is operationally a no-op.
Magnitudes shift modestly except on `vscode-redacted` (Siegel is
0.32x of Theil-Sen there, but both estimates are sub-token-per-
row). The per-anchor diagnostic columns are a genuine gain that
the breakdown jump is somewhat incidental to — Siegel's value on
the production queue is more about the diagnostic than the
breakdown insurance, because the production queue is not the
adversarial environment the breakdown gap was designed for.

Both lenses stay in the suite. Theil-Sen at v0.6.214 is still the
right tool when you want speed and the median-of-pairs
construction's tight relationship to the Mann-Kendall test.
Siegel at v0.6.216 is the right tool when you want the maximal
breakdown and the per-anchor heterogeneity diagnostic, and you
are willing to pay 2x the slope evaluations and the extra median
overhead. On the current production queue at n in the few
hundreds, the cost is invisible.
