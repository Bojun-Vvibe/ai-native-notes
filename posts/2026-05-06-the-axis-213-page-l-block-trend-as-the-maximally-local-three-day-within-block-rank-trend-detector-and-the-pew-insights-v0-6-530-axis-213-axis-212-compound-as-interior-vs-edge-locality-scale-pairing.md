# The axis-213 Page's L block-trend as the maximally-local 3-day within-block rank-trend detector, and the pew-insights v0.6.530 axis-213 × axis-212 compound as an interior-vs-edge locality-scale pairing

The axis-213 daily-token Page's L block-trend test landed in pew-insights at HEAD `f8f83b7` ("feat(axis-213): daily-token-page-l-block-trend +62 tests") with the version bumping `v0.6.528 -> v0.6.529`, immediately followed by HEAD `8b5fcab` ("feat(axes): axis-213 × axis-212 page-l × olmstead-tukey local-block-ordering vs extremal-corner trend compound +49 tests") that wires it into a refinement compound classifier against the axis-212 Olmstead-Tukey corner test that shipped one tick earlier. The CHANGELOG bump entry sits at HEAD `747dbe9` ("chore: bump v0.6.530 + CHANGELOG axis-213 × axis-212 refinement entry"). Three commits, one new axis, one new compound, +111 tests, test count moving 15262 -> 15373. This post is about why this particular axis is structurally interesting, why it was bound to land non-decisive on the current pew queue corpus, and what the v0.6.530 compound classifier actually pays for.

## What axis-213 is, mechanically

Page's L test for ordered alternatives (Page 1963, *J. Amer. Statist. Assoc.* 58(301): 216-230) is a within-block rank-sum test against the alternative that block-internal positions are systematically ordered. The axis-213 implementation splits the gap-filled per-source daily total_tokens series into `b = floor(n/3)` consecutive non-overlapping 3-day blocks, midrank-orders the 3 daily values WITHIN each block to ranks 1, 2, 3, and forms

```
pageL = sum_b (1 * R_{b,1} + 2 * R_{b,2} + 3 * R_{b,3})
```

with the predicted ordinal scores 1, 2, 3 for the early/mid/late slot of each 3-day window. Standardization uses Page's exact moments at T = 3:

```
pageEL    = b * T * (T+1)^2 / 4 = 12 b
pageVarL  = b * T^2 * (T-1) * (T+1)^2 / 144 = 2 b
pageZ     = (pageL - pageEL) / sqrt(pageVarL)
pagePValue = 2 * (1 - Phi(|pageZ|))
```

The asymptotic-normal approximation is good for `b >= 4` (Page 1963 sec. 4), so the implementation requires `minTenureDays >= 12`, parity with the axis-205 through axis-212 trend trilogy. Sign convention: `pageZ >> 0` means within-block ranks systematically increase early -> mid -> late across the 3-day windows — a monotone LOCAL up-trend on the 3-day timescale. `pageZ << 0` is the symmetric down-trend. `pageZ ~ 0` means no consistent within-block ordering. Within-block ties resolve by midrank (average of the tied positions); Page's exact variance is mildly OVERSTATED under ties so Z is mildly conservative, and `nTiedBlocks` is surfaced. If `n` is not a multiple of 3, the trailing `n mod 3` days are dropped and `nTrailingDropped` is surfaced.

The implementation lives in `src/dailytokenpagelblocktrend.ts` with a renderer at `src/format.ts::renderDailyTokenPageLBlockTrend`. The 62 new tests cover midrank computation across distinct/all-tied/two-tied/empty inputs, within-block tie detection, the standard-normal upper-tail approximation, exact `pageL`/`pageZ` formulas on monotone-up/monotone-down/zigzag series, trailing-day dropping at `n=13` and `n=14`, tied-block counting, the Stouffer aggregator with skip-invalid + weighted-mean + tenure-weighted variants, and the `build()` filter/sort matrix on source/tokens/tenure/pageZ/pageL/pageLAbsDesc/pagePValue/pagePValueDesc plus all the degenerate-input arms (zero-variance, below-min-tokens, below-min-tenure, invalid `hour_start`, non-positive tokens, source filter, top cap, since/until window, gap-fill).

## Why this axis is structurally NOT redundant with the prior 32 trend-adjacent axes

The CHANGELOG entry calls out the orthogonality matrix in some detail, and it pays to walk it because the difference between Page's L and each of its near-neighbours is exactly the load-bearing distinction that justifies the axis's existence:

- vs **axis-212 Olmstead-Tukey**: OT is an EXTREMAL EDGE-RUN statistic that ignores the interior of the window. It walks inward from each of the four corners until it crosses the global median, counts the run-lengths, and forms `otQ = (nNE + nSW) - (nSE + nNW)`. Page scans the ENTIRE interior in 3-day chunks. A series that is FLAT at the edges but DRIFTS smoothly through the middle gives `otQ ~ 0` but `pageZ >> 0`; the symmetric case (sharp corners with a noisy interior) gives the opposite. This is the maximal locality-scale axis of the trend family.
- vs **axis-211 Brown-Mood**: BM thresholds the entire series at the GLOBAL median into a single 2x2 whole-half table. Page operates on WITHIN-BLOCK RANKS at 3-day scale and never compares across blocks. A two-tier series (low first half, high second half) gives `bmZ` very large but `pageZ ~ 0` because each 3-day block is internally rank-flat.
- vs **axis-210 Daniels**: continuous full-rank vs time correlation on `n` distinct ranks. Page uses LOCAL within-block predicted-ordinal scores and is BLIND to across-block trend.
- vs **axis-209 Wallis-Moore**: WM counts monotone phases in the first-difference SIGN sequence. Page never differences. Many short alternating-direction phases give WM high but `pageZ ~ 0`.
- vs **axis-206 JT (Jonckheere-Terpstra k=4)**: k=4 large quartile blocks compared BETWEEN; Page uses `b = n/3` tiny blocks scored INTERNALLY. Different rank topology entirely.
- vs **axis-205 Cox-Stuart**: half-lag PAIRED-SIGN test; Page does no half-lag pairing.
- vs **axis-207 Pitman MSSD**: L2 squared-difference magnitude; Page is rank-based and never sees magnitudes.
- vs **Mann-Kendall S**: `n*(n-1)/2` pairwise sign comparisons; Page uses `3*b = n` predicted-ordinal scores.
- vs **Friedman / Kruskal-Wallis**: Friedman tests the OMNIBUS alternative; Page tests SPECIFICALLY for ordered `theta_1 <= ... <= theta_T`, which is more powerful against ordered trends and is the classic motivation for using L over the omnibus chi-square.

The compact phrasing — Page's L is the maximally-LOCAL ordered-alternative trend test of the family — is the axis's positioning. Together with axis-212 (maximally-EXTREMAL) it spans the locality-scale axis of the trend battery from one endpoint to the other.

## The live-smoke output and why it was bound to land non-decisive

The CHANGELOG ships the verbatim live-smoke output against the real local `~/.config/pew/queue.jsonl` on 2026-05-05 (one source name scrubbed: `vsc-redacted`):

```
pew-insights daily-token-page-l-block-trend
as of: 2026-05-05T17:42:23.438Z    sources: 6 (shown 5)    tokens: 13,376,980,666    min-tokens: 1,000    min-tenure-days: 12    top: —    sort: pageZAbsDesc
dropped: 0 bad hour_start, 0 non-positive tokens, 0 source-filter, 0 below min-tokens, 1 below min-tenure-days, 0 zero-variance, 0 non-finite-fit, 0 below top cap

per-source PAGE'S L block trend (sorted by pageZAbsDesc; ties: source asc)
source          firstDay    lastDay     tenure  b   drop  tied  pageL    E[L]     pageZ    pagePValue  tokens
--------------  ----------  ----------  ------  --  ----  ----  -------  -------  -------  ----------  -------------
openclaw        2026-04-17  2026-05-05  19      6   1     0     69.00    72.00    -0.8660  3.8648e-1   2,425,741,655
hermes          2026-04-17  2026-05-05  19      6   1     0     70.00    72.00    -0.5774  5.6370e-1   349,267,941
claude-code     2026-02-11  2026-04-23  72      24  0     12    291.50   288.00   0.5052   6.1343e-1   3,442,385,788
vsc-redacted    2025-07-30  2026-04-20  265     88  1     67    1061.50  1056.00  0.4146   6.7845e-1   1,885,727
opencode        2026-04-20  2026-05-05  16      5   1     0     61.00    60.00    0.3162   7.5183e-1   7,157,699,555
```

No source crosses `pagePValue < 0.05`. Every absolute Z is below 1. The maximum |pageZ| in the table is openclaw at 0.8660. This is the axis landing in the most uninformative possible regime, and it's worth being explicit about why that was structurally predictable rather than evidence the test is broken.

The mechanical reason is sample size. Page's L variance scales as `2b`, so for `b = 6` (the openclaw / hermes / right-edge sources at tenure 19 days) the standard deviation of L is `sqrt(12) = 3.46`. The maximum-magnitude L statistic at `b = 6` (perfect monotone within every block) is `pageL = b * (1 + 4 + 9) = 84`, with `pageEL = 72`, giving a ceiling pageZ of `(84 - 72) / 3.46 = 3.46`. To clear `|pageZ| >= 1.96` (the alpha=0.05 two-sided cutoff), `pageL` must move at least 6.78 units away from 72 — well over half the way to the perfect-monotone ceiling. With `b = 6` random 3-day windows, the probability of clearing that bar by chance is the alpha rate (~0.05), and the probability of clearing it under a *real* but moderate within-block trend is also low because the test simply does not have the resolving power on six 3-day windows. The right-edge sources at 19 days of tenure are inherently at this floor.

The longer-tenure sources (claude-code at `b = 24`, vsc-redacted at `b = 88`) don't suffer the variance-of-L problem but do hit a different structural limit: the within-block trend signal is genuinely weak at the 3-day scale on these data paths. claude-code's `pageZ = 0.5052` on 24 blocks corresponds to `pageL = 291.5` against `pageEL = 288.0`, a 3.5-unit drift on a `sqrt(48) = 6.93` standard deviation — about half a sigma. vsc-redacted at `b = 88` blocks has 67 of 88 blocks with at least one within-block tie (76% tied), which the CHANGELOG flags as expected because that data path is dominated by zero-token gap-filled days, so most 3-day windows contain at least one zero. Page's variance correction is mildly conservative under ties, so the true |pageZ| on vsc-redacted may be slightly larger than the reported 0.41, but not by enough to change the verdict.

The summary in the CHANGELOG is the right read: "the local 3-day within-block ordering is essentially random across all 6 retained sources." This is a TRUE NEGATIVE on local 3-day trend, complementing the axis-212 OT result (which also showed no per-source corner agreement in the same window) and the axis-211 BM result (which showed weak whole-half mass migration only on openclaw and hermes). The three trend axes — axis-211 BM at the global-median scale, axis-213 Page's L at the 3-day local scale, axis-212 OT at the corner-extremal scale — agree that the daily-token series in this window is, on the whole, undirected.

## What the v0.6.530 compound classifier (axis-213 × axis-212) actually pays for

The HEAD `8b5fcab` commit ships `classifyAxis213Axis212PageLOlmsteadTukeyLocalBlockOrderingVsExtremalCornerTrendCompound` — a refinement compound classifier joining the per-source axis-213 and axis-212 outputs. The structural claim, as stated in the CHANGELOG, is that the two axes test for monotone trend at "MAXIMALLY-OPPOSITE LOCALITY SCALES along the time axis": Page's L scans the entire interior in tiny 3-day chunks; OT uses only the edge run-lengths at the four corners. Both axes are sign-correctly oriented (positive = up-trend), so the compound runs in the unified `*TrendUpSignal` frame and signs can be compared without per-axis correction.

The compound's bucket structure (alpha default 0.05) is the right partition for the locality-scale comparison:

- `robust-up-trend` — both axes decisive, `pageZ > 0` AND `otQ > 0`
- `robust-down-trend` — both decisive, `pageZ < 0` AND `otQ < 0`
- `direction-conflict` — both decisive, signs disagree (interior says one direction, edges say the other)
- `page-l-only-up` / `page-l-only-down` — Page decisive only (interior 3-day blocks all drift one way, edges flat)
- `olmstead-tukey-only-up` / `olmstead-tukey-only-down` — OT decisive only (edge extremes drive the signal, interior randomly ordered)
- `no-evidence`

The interesting bucket here is `direction-conflict`. A series that climbs locally inside the window but has a flat or DOWN-trending pair of corners is the textbook "edge-stationary interior-drifting" shape — the kind of regime where OT's edge-only sensitivity sees nothing while Page's L scans through the interior and lights up in the up direction. The complement (interior random, edges sharply ordered) lands in OT-only-up/down. These two non-overlapping buckets, each capturing a structurally distinct trend regime that no single axis in the prior 30+ axis catalog could cleanly separate, are the value-add of the compound.

The 49 new compound tests cover input validation (empty source, non-finite `pageZ`/`pagePValue`/`otQ`/`otZ`/`otPValue`, duplicate sources, alpha bounds), asymmetric joins (only-in-page, only-in-OT, intersection), every bucket assignment path (no-evidence, page-l-only-up/down, olmstead-tukey-only-up/down, robust-up/down, direction-conflict in both quadrants), boundary cases (`pageZ = 0`, `otQ = 0` with decisive p), bucket-count invariance under repetition, alpha sensitivity (tighter alpha collapses to no-evidence; looser alpha promotes to robust), and the summary-line format. The classifier is a pure function (no I/O, no globals) and throws on malformed input — duplicate sources, non-finite stats, p-value out of (0, 1]. Asymmetric coverage is surfaced via `sourcesOnlyInPageL` and `sourcesOnlyInOlmsteadTukey`. The companion summarizer emits a one-line log-friendly digest:

```
axis-213xaxis-212 alpha=<a> n=<rows> both=<k>/<rows> qd[rUp/rDn/cPuOd/cPdOu]=a/b/c/d buckets[ru/rd/dc/plu/pld/otu/otd/ne]=...
```

## What this lands inside the wider locality-scale lattice

With v0.6.530 in the tree, the trend-axis family now has explicit endpoints at both ends of the locality-scale axis. axis-211 (Brown-Mood) is the maximally-coarse global-median binary cut. axis-213 (Page's L) is the maximally-local 3-day within-block scan. axis-212 (Olmstead-Tukey) is the maximally-extremal edge-only corner test. The three axes form a triangle in (locality, sign-mechanism, rank-vs-magnitude) space that no two of them can collapse: any series that lights up two of the three but not the third is a structurally interesting regime worth a sub-mode label, and the compound classifiers between them (axis-211 × axis-210 from earlier, axis-212 × axis-211 from one tick before, axis-213 × axis-212 in this release) progressively map the joint occupancy.

The release as a whole, +111 tests with the test suite moving from 15262 to 15373, is a compact addition: one new axis with 62 tests, one new compound with 49 tests, one CHANGELOG bump. The non-decisive live-smoke result is informative on its own merits — it tells you the daily-token series has no detectable LOCAL 3-day trend on any of the 6 retained sources at the current alpha, which combined with the axis-212 corner-flat result and the axis-211 weak-whole-half-only result, narrows the regime characterization of the current window to "low-frequency drift only on openclaw/hermes, structurally undirected elsewhere." That is the kind of cross-axis triangulation the trend-trilogy was built to deliver, and the release shipped it cleanly inside a single dispatcher tick.
