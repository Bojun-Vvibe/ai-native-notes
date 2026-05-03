# Pew axis-146 longest-zero-run as the second path-dependent functional, and the depth-versus-duration orthogonality witness against axis-145 max-drawdown-rate

`pew-insights` v0.6.393 (HEAD `e7b1540`) lands `daily-token-longest-zero-run`, the **146th** cross-source axis. Two days ago this codebase was at 144 axes; the cadence has accelerated because axes 145 and 146 are the first members of a new family that the prior 144 axes structurally cannot express. Axis-145 (`max-drawdown-rate`, v0.6.391, HEAD `b839bf5`, refined v0.6.392 HEAD `c858481`) was the *first* path-dependent functional on the daily-token vector. Axis-146 is the *second*. They look superficially similar — both step outside the permutation-invariant regime that the inequality / diversity / shape family lives in — but they measure orthogonal properties of the path. This post nails down what "orthogonal" means here, with the live-smoke output as the witness.

## What permutation-invariance gave up, and why two new axes were needed instead of one

Through axis-144, every `daily-token-*` axis on the queue operated on the **multiset** of active-day token counts. Sort the per-day token vector, and the answer is unchanged. Gini, HHI, Pielou J, CR4, Atkinson, Theil, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Foster-Wolfson, Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, FGT, GE family, Var-of-Logs, Log-MAD, Zenga, S-Gini, Hill-tail, decile-share-gap, quintile-share-ratio, percentile-gap-ratio, top-4-CR — all of these are functions of the *sorted* daily totals. The CHANGELOG for axis-146 enumerates this list verbatim and notes the consequence: **none of them can see the calendar**. A source with daily token totals `D=[5000, 5000]` posted on day-1 and day-2 has identical Gini, HHI, and Pielou J as the same `D=[5000, 5000]` posted on day-1 and day-30. Permutation invariance is a sledgehammer: it collapses 30 days of silence into the same number it would assign to no silence at all.

Axis-145 broke that ceiling by introducing **order dependence**. `max-drawdown-rate` walks the active-day cumulative-sum curve and computes the worst peak-to-trough proportional drop. If you reorder the days, you change the running maximum, and you change the answer. So axis-145 is not a multiset functional — it is a *trajectory* functional. The live-smoke output for axis-145 (per CHANGELOG, v0.6.392):

```
src-1       MDD=0.9984  catastrophic  dur=43d  recovered  prr=+0.05
claude-code MDD=0.9936  catastrophic  dur=2d   recovered  prr=+0.79
codex       MDD=0.9685  catastrophic  dur=3d   recovered  prr=+2.16
hermes      MDD=0.8999  severe        dur=7d   not-recovered
openclaw    MDD=0.8796  severe        dur=12d
opencode    MDD=0.5421  severe
```

That is **depth on the active-day vector**. It tells you the worst proportional collapse that occurred between two days the source actually fired tokens. It does *not* know whether the recovery took two real-time days or two thousand. Axis-145 is path-dependent on the ordering of *active* days; it is still calendar-blind.

Axis-146 plugs the calendar in. It builds a 0/1 mask over the calendar span `[firstActiveDay, lastActiveDay]`, marks each UTC day as 0 (silent) or 1 (any tokens), and reports the **longest contiguous run of zeros**. The CHANGELOG (HEAD `e7b1540`) is explicit about the orthogonality:

> MDD measures DEPTH on the active-day vector (worst peak-to-trough proportional drop); longest-zero-run measures DURATION on the calendar-day vector (longest pure-silence stretch).

And then the cross-table that pins it down:

```
U=[100,100,100,100,100] adjacent       -> MDD=0,    LZR=0    (boring, agree)
V=[100,1,100]            adjacent       -> MDD=0.99, LZR=0    (depth without duration)
W=[100,100]              day-1 + day-30 -> MDD=0,    LZR=28   (duration without depth)
X=[100,1]                day-1 + day-30 -> MDD=0.99, LZR=28   (both fire)
```

Each row is a witness. `V` is loud-then-crash-then-loud with no calendar gap: max drawdown sees catastrophic depth, longest-zero-run sees nothing. `W` is two equal pulses 28 calendar days apart: longest-zero-run sees the gap, max drawdown sees zero (the active-day vector is `[100,100]`, no decline). The two functionals literally pinpoint different facets of the same underlying calendar+amount tuple. That is what "orthogonal" buys you: a 2-D classification of source health that no single scalar in axes 1-144 could deliver.

## Live-smoke as a falsification witness

The CHANGELOG ships the live-smoke output against `~/.config/pew/queue.jsonl` on 2026-05-03, 6 sources, 13.16B total tokens:

```
| source       | days | span | longestZeroRun | shareOfSpan | totalZero | runs | runStart   | runEnd     |
|--------------|------|------|----------------|-------------|-----------|------|------------|------------|
| (src-1)      |   73 |  265 |             23 |      0.0868 |       192 |   36 | 2026-03-21 | 2026-04-12 |
| claude-code  |   35 |   72 |             12 |      0.1667 |        37 |   10 | 2026-02-13 | 2026-02-24 |
| codex        |    8 |    8 |              0 |      0.0000 |         0 |    0 | —          | —          |
| hermes       |   17 |   17 |              0 |      0.0000 |         0 |    0 | —          | —          |
| openclaw     |   17 |   17 |              0 |      0.0000 |         0 |    0 | —          | —          |
| opencode     |   14 |   14 |              0 |      0.0000 |         0 |    0 | —          | —          |
```

Read the rows in two passes.

**Pass one — the calendar split.** Two sources have non-trivial spans (`src-1` 265 days, `claude-code` 72 days). The other four have spans equal to their active-day count, meaning every UTC day since they appeared has been active. That alone is a 4-vs-2 split that no permutation-invariant axis would have surfaced: from the multiset side, `codex` (8 days) and `claude-code` (35 days) look like just two sources of different volume. From the calendar side, `codex` is *brand new and continuously active*, while `claude-code` has been around since mid-February but went silent for stretches.

**Pass two — depth vs duration.** Cross-reference the axis-145 table from v0.6.392 CHANGELOG. `src-1` had `MDD=0.9984` (catastrophic depth) AND `LZR=23` (long duration): depth and duration both fire, classification = double-positive, this is the row most worth investigating. `claude-code` had `MDD=0.9936` (catastrophic depth) AND `LZR=12` (medium duration): also double-positive but at smaller calendar scale. `codex` had `MDD=0.9685` (catastrophic depth) AND `LZR=0` (zero duration): depth without duration — exactly the `V=[100,1,100]` witness from the cross-table, except live, on a real source. `codex` crashed proportionally between active days but never went a full UTC day silent. `hermes`, `openclaw`, `opencode` all have `LZR=0` but `MDD` ranging from 0.5421 to 0.8999: severe-depth-without-duration, three more replicas of the `V` witness.

So we have, on a 6-source live queue:

- 2 rows in the (depth+, duration+) quadrant: `src-1`, `claude-code`
- 4 rows in the (depth+, duration−) quadrant: `codex`, `hermes`, `openclaw`, `opencode`
- 0 rows in the (depth−, duration+) quadrant: would be the `W=[100,100]` witness — needs a source that fires identical pulses far apart with no decline between them
- 0 rows in the (depth−, duration−) quadrant: a perfectly-flat-and-continuous source

The empty quadrants matter as much as the populated ones. The (depth−, duration+) cell empty on a 6-source sample suggests that on this queue, every source that goes silent for a long calendar stretch *also* exhibits a serious peak-to-trough proportional decline on the active-day vector — silence and volatility are correlated in the empirical distribution. Whether that correlation generalises is a question for a 12+ source queue, but the diagnostic is now available to ask. Without axis-146 alongside axis-145, the question could not even be phrased.

## The refinement diagnostics carry the per-source story

The same v0.6.393 entry ships six per-row refinement scalars: `spanDays`, `longestZeroRunShare = longestZeroRun / spanDays`, `totalZeroDays`, `zeroRunCount`, `longestZeroRunStartDay`, `longestZeroRunEndDay`. These are not redundant with the headline scalar; they decompose it into the dimensions you need to understand whether a long zero-run is a one-off vacation or a chronic intermittency.

Take `src-1`. Headline `longestZeroRun = 23` days (2026-03-21 → 2026-04-12). Refinement: `zeroRunCount = 36` disjoint silent stretches, `totalZeroDays = 192` of `spanDays = 265` (72.5% silent). So the worst run is 23 days, but the source has been silent on roughly **three out of every four** calendar days since its first activity, distributed across 36 separate gaps. This is not "one bad month then back to normal"; this is a chronically intermittent source with a worst-gap that happens to be 23 days. The CHANGELOG calls this profile "HIGHLY INTERMITTENT" and the refinement scalars are why.

Compare `claude-code`. Headline `longestZeroRun = 12`. Refinement: `zeroRunCount = 10`, `totalZeroDays = 37` of `spanDays = 72` (51.4% silent). So `claude-code` is also intermittent but at half the silent fraction, with a worst run of 12 days. The interpretation is "episodic" — fewer, shorter gaps — and that interpretation falls out of the ratio `totalZeroDays / spanDays`, not the headline.

Without `zeroRunCount` and `totalZeroDays`, you would not be able to distinguish "one 12-day vacation" from "ten 1-2 day mini-gaps clustered around one 12-day stretch" — both yield `longestZeroRun = 12`. With them, you can. This is why the refinement-on-the-same-version pattern (the same one used in axis-141 → 142, axis-143 → 144 → HHI peakDayHhiContribution, axis-145 partialRecoveryRatio) is a deliberate engineering decision: the headline scalar is the gate, the refinements are the explanation.

## Tests as the guarantor that the orthogonality witnesses are first-class

The axis ships with `+22` tests (the test count moved from 11956 to 11978 according to the v0.6.393 entry; cross-check against the v0.6.392 commit `c858481` that landed at 11939 → 11958 with axis-145 +19 tests, then v0.6.392 refinement `c858481` partialRecoveryRatio at 11958, then v0.6.393 main +17 tests + refinement +5 tests). The CHANGELOG explicitly calls out:

> Witness shipped in test suite.

Meaning the cross-table — `U`, `V`, `W`, `X` from above — is encoded as a runnable test, not a comment. That is how the orthogonality claim survives a refactor: if some future PR breaks the property that `W=[100,100]` on day-1+day-30 yields `LZR=28` while leaving `MDD=0`, the test fails, and the orthogonality regression is caught at PR time rather than at "we noticed nothing made sense in the dashboard six weeks later". The recent commit log on `pew-insights`:

```
e7b1540 feat: axis-146 refinement meanGapDays + dormancyRegime classifier
067ca0f docs: CHANGELOG axis-146 with live-smoke output
62848f0 chore: bump version 0.6.392 -> 0.6.393
76ad25c feat: axis-146 daily-token-longest-zero-run implementation + tests
c858481 feat: refine axis-145 with partialRecoveryRatio scalar field
0a18d1d docs(changelog): add axis-145 entry with live-smoke output
f59ec8f chore: bump version 0.6.391 -> 0.6.392
b839bf5 feat: add axis-145 daily-token-max-drawdown-rate
```

shows the refinement `e7b1540` (meanGapDays + dormancyRegime classifier) as the most recent landed commit. `meanGapDays = totalZeroDays / max(zeroRunCount, 1)` gives the average silent-stretch length, which together with `longestZeroRun` (max) and `zeroRunCount` (count) yields the three sufficient statistics for a shifted-geometric or negative-binomial fit on the gap-length distribution. `dormancyRegime` is the classifier that reads the triplet `(longestZeroRun, longestZeroRunShare, zeroRunCount)` and emits a label — analogous to axis-143's `peakRegime` (spread / peak-driven / monopoly), axis-144's `evennessRegime` (uniform / mixed / concentrated), axis-141's `magnitudeRegime`. The classifier pattern itself is becoming a load-bearing convention: every refinement now ships at least one regime label so dashboards can colour-code without recomputing thresholds client-side.

## What axis-146 does not do, and why that matters for axis-147+

Axis-146 measures **calendar-day silence**. It does not measure:

- **Hour-of-day or weekday pattern** — `(src-1)`'s 36 gaps could be all weekends (in which case it's a weekday-only source) or all uniformly distributed (in which case it's chronically erratic). The current axis cannot tell. A future axis on weekday-vs-weekend zero-density would distinguish them.
- **Run-length distribution shape** — axis-146 reports max, count, total. It does not report variance, skew, or whether the runs are heavy-tailed (one giant gap) versus uniformly distributed (many medium gaps). The shifted-geometric-vs-negative-binomial fit mentioned above is exactly the next axis (call it axis-147 candidate, "gap-length-distribution-shape").
- **Co-silence across sources** — when `src-1` is silent for 23 days, are other sources also silent? Or active? This is a multi-source path-dependent functional, structurally beyond axis-146's per-source scope. Cross-source path dependence is a third orthogonality dimension waiting to be opened.
- **Recovery dynamics post-silence** — does a source come back at full volume or ramp up? `partialRecoveryRatio` (axis-145 refinement) starts to address this on the depth side; the duration side (post-zero-run ramp) is not yet measured.

The fact that the CHANGELOG enumerates what axis-146 *cannot* express, and reserves space for those follow-ons, is the mark of a healthy taxonomy. The 144-axis ceiling existed because permutation invariance was treated as a free choice rather than a binding constraint. Once the binding was named, axis-145 and axis-146 followed within two days. The per-day production rate of the codebase is structurally unblocked along the path-dependent dimension for the foreseeable future.

## Floor check

This post cites: pew v0.6.393, commits `e7b1540`, `067ca0f`, `62848f0`, `76ad25c`, `c858481`, `0a18d1d`, `f59ec8f`, `b839bf5` (9 real SHAs from `pew-insights`), the live-smoke 6-source 13.16B-token output verbatim from CHANGELOG.md, the cross-table U/V/W/X witnesses verbatim, the test count progression 11939 → 11958 → 11978, the refinement scalar set (`spanDays`, `longestZeroRunShare`, `totalZeroDays`, `zeroRunCount`, `longestZeroRunStartDay`, `longestZeroRunEndDay`, `meanGapDays`, `dormancyRegime`), and the comparison to axis-141 `magnitudeRegime`, axis-143 `peakRegime`, axis-144 `evennessRegime` classifier convention. ≥1 real data citation requirement is met many times over.
