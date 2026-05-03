---
title: "pew-axis-145 max-drawdown-rate as the first path-dependent cross-source functional, and the permutation-invariance barrier it breaks on the 144 prior axes"
date: 2026-05-03
---

## The structural claim, said up front

`pew-insights` shipped axis-145 — `daily-token-max-drawdown-rate` — at version `0.6.392` (commit `c858481` "feat: refine axis-145 with partialRecoveryRatio scalar field", on the back of `b839bf5` "feat: add axis-145 daily-token-max-drawdown-rate" and the version bump `f59ec8f` "chore: bump version 0.6.391 -> 0.6.392"). The `CHANGELOG.md` entry for `0.6.392 — 2026-05-04` opens with the structural orthogonality claim verbatim:

> `axis-145` is the first **PATH-DEPENDENT** cross-source functional surfaced. ALL prior cross-source daily-token axes (Gini, HHI, Pielou, CR4, Atkinson, Theil, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Foster-Wolfson, Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, FGT, GE family, Var-of-Logs, Log-MAD, Zenga, S-Gini, Hill-tail, decile-share-gap, quintile-share-ratio, percentile-gap-ratio, top-4-CR, ...) are PERMUTATION-INVARIANT on the day vector — they cannot distinguish "smooth ramp followed by a 90% crash" from the same multiset reshuffled into "crash first, ramp later". MDD separates them.

That single paragraph is doing more work than it looks. It is asserting that the previous 144 axes — every concentration / inequality / diversity / dispersion / tail-shape functional that pew-insights has accreted across roughly six weeks of axis-shipping — collectively live inside a quotient space, the space of finite multisets `{x_1, ..., x_n}` over the source's per-day token totals, with the day-index *forgotten*. Axis-145 is the first axis that lives one level up, on the *sequence* itself: it cares about which day came before which other day. The `CHANGELOG` makes that concrete with the witness triple `A=[100, 90, ..., 10]`, `B=[10, 20, ..., 100]`, `C=[10, 100, 20, ..., 90]`, all three identical as multisets, and notes that every permutation-invariant axis returns identical values on all three, while `MDD(A) = 0.9`, `MDD(B) = 0`, `MDD(C) = 0.8` — a 90 percentage-point spread no permutation-invariant axis can express.

This post is about why that ledger-line in the changelog is the most important structural event in the axis programme since the f-divergence quartet (axes 126-129) closed the cross-source distance-family, and what it implies about which axes the queue can and should reach for next.

## The definition, the bounds, and the live witness

The functional, copied from the source-of-truth `CHANGELOG` and the module reference `src/dailytokenmaxdrawdownrate.ts`:

```
MDD = max over (i < j) of (D_i - D_j) / D_i
```

where `D_i` is the running peak of the per-day token totals series ordered by ascending UTC day, and `D_j` is the post-peak trough that follows it. The range is `[0, 1)`: `MDD = 0` is the monotone non-decreasing case (no peak ever decreases below itself), and `MDD → 1` is the post-peak-collapse-to-near-zero case. The functional is scale-invariant — multiplying the whole series by any positive constant leaves MDD unchanged — and crucially it is *not* translation-invariant: shifting the series by a positive constant compresses MDD, because it shrinks the *fractional* drop while leaving the absolute drop unchanged. The references the changelog cites are Magdon-Ismail & Atiya 2004 ("Maximum drawdown", *Risk Magazine* 17(10): 99-102) and Chekhlov, Uryasev & Zabarankin 2005 ("Drawdown measure in portfolio optimization", *International Journal of Theoretical and Applied Finance* 8(1): 13-58), placing axis-145 in the portfolio-risk literature, where MDD is the standard non-parametric pathwise loss measure on equity curves.

The CHANGELOG ships a live-smoke table. Reproduced verbatim:

| source       | days | mdd    | regime       | peak (tok)        | trough (tok)    | dur | recovered |
|--------------|------|--------|--------------|-------------------|-----------------|-----|-----------|
| (src-1)      | 73   | 0.9984 | catastrophic | 2025-10-13 (181k) | 2026-02-24 (299)| 43d | yes       |
| claude-code  | 35   | 0.9936 | catastrophic | 2026-03-04 (12.3M)| 2026-03-06 (78k)|  2d | yes       |
| codex        |  8   | 0.9685 | catastrophic | 2026-04-13 (183M) | 2026-04-16 (5.8M)|  3d | yes       |
| hermes       | 17   | 0.8999 | severe       | 2026-04-19 (34.7M)| 2026-04-26 (3.5M)|  7d | no        |
| openclaw     | 17   | 0.8796 | severe       | 2026-04-19 (354M) | 2026-05-01 (42.6M)| 12d | no       |
| opencode     | 14   | 0.5449 | severe       | 2026-04-21 (724M) | 2026-05-03 (330M)| 12d | no       |

Six sources, 13.14B total tokens at axis-launch. Three sources land in the `catastrophic` band (MDD ≥ 0.90), three in `severe` (0.50 ≤ MDD < 0.90), zero in `flat / shallow / moderate / degenerate`. The five-band `drawdownRegime` classifier fixes cutoffs at `0`, `< 0.25`, `< 0.50`, `< 0.90`, `≥ 0.90` plus a sixth `degenerate` band for `n < 2` (drawdown is undefined for fewer than two days). The bands are not arbitrary: `flat` is the bit-exact monotone-non-decreasing case, `shallow` is the band where a typical week-to-week oscillation lives, `moderate` is where a clear regime shift lives, `severe` is "the source did most of its work in one window and then mostly stopped", and `catastrophic` is "the source had one peak day and then collapsed".

The refinement commit `c858481` adds `partialRecoveryRatio` as a scalar field per row — a compact way to ask, "after the trough, did the post-trough series climb back partway, all the way, or above the original peak?". The dispatcher's history note for the feature run records `prr = +0.05` for src-1, `+0.79` for claude-code, `+2.16` for codex, "not-recovered" for hermes / openclaw / opencode. The codex `+2.16` is striking: codex collapsed by 96.85% over three days (peak 183M tokens on 2026-04-13, trough 5.8M on 2026-04-16) and then ran more than three times the original peak in subsequent days. That is exactly the kind of pathwise structure that no Gini / HHI / Pielou / CR4 row could ever surface — the multiset of 8 daily totals is the same regardless of which day held the 183M peak, but `recovered = yes` and `prr > 1` say the source was not just "concentrated on one day" but specifically "had a brief outage and rebounded".

## Why "permutation-invariant" is the right cleavage

The 144 prior cross-source axes are not a random pile. They cluster into seven structural families, and each family is permutation-invariant by construction:

1. **Concentration / inequality** — Gini (axis-N), HHI (axis-143), Theil-T, Atkinson, Hoover, Pietra, Bonferroni, Wolfson, Palma, Kolm-Pollak, Chakravarty, Esteban-Ray, FGT, GE family. These all map a vector to a scalar via sums and products of order statistics or share-of-total quantities. Permuting the input permutes the order statistics identically, and shares of total are invariant.

2. **Diversity / evenness** — Pielou-J (axis-144), Shannon-H, Hill numbers q1/q2, Simpson, Renyi-alpha, Tsallis-q. Functions of the share vector `p = x / sum(x)`. Permuting `x` permutes `p`. Sums over `p` are invariant.

3. **Tail / spread** — Var-of-Logs, Log-MAD, Zenga curves, S-Gini, Hill-tail-index, decile-share-gap, quintile-share-ratio, percentile-gap-ratio, top-4-CR (axis-142), top-k-share. Order-statistic functionals.

4. **Distribution-shape moments** — Pearson-second-skewness (axis-141), kurtosis, L-moments, mean-median-displacement. Moment-equivalent under permutation.

5. **f-divergence pairs** — JSD (axis-118 / axis-126), TV (axis-127), Hellinger (axis-128), Triangular-Discrimination (axis-129), Bhattacharyya (axis-130), Jeffreys (axis-131), Symmetric-Chi-Squared (axis-134), Clark (axis-135), Kumar-Johnson (axis-137), Neyman-halves (axis-139), K-divergence-halves (axis-140), Topsoe (axis-138). All are functions of the *empirical* share distributions of two sources, with day labels ignored.

6. **Two-sample full-distribution equality** — KS (axis-118), AD (axis-119), CvM (axis-120), W1 (axis-121), Energy (axis-122), Mann-Whitney / Brown-Forsythe / Siegel-Tukey (axes 115-117). These are explicitly permutation-tests; their null distributions are *defined* by invariance under joint permutation of the two pooled samples.

7. **Trend / temporal hints (NOT permutation-invariant, BUT not cross-source)** — Mann-Kendall, Cox-Stuart, lag-1 / lag-7 autocorrelation, sign-test, half-shift trend, Hjorth, Higuchi / Katz / Petrosian / Sevcik fractal dimensions, sample / permutation entropy, Hurst R/S, DFA-alpha. These *do* depend on the order of the day vector, but in the existing axis programme they are scoped per-source, not as cross-source rankings. The ranking they induce on sources is a ranking by "how trended is each source on its own time axis", not "how do the sources compare on a single pathwise quantity".

Axis-145 is the first axis to land in the cell `(cross-source, path-dependent)` — a cell that was empty in the 144-axis matrix.

The `src/dailytokenmaxdrawdownrate.ts` module makes this orthogonality argument explicit, including against every entry in family 7. The argument is short: trend tests answer "is there a monotone tendency", and pathwise smoothness tests answer "is the autocorrelation high / fractal-dimension low / entropy low". MDD answers a different question — "regardless of trend or smoothness, what is the worst peak-to-trough fractional descent the series ever traced". A series can be Mann-Kendall-non-trending (S ≈ 0), have moderate lag-1 autocorrelation, and still have MDD = 0.99 if it had one all-time peak followed by a single deep trough. Conversely, a series can be Mann-Kendall-strongly-trending-down (S ≪ 0) with MDD only 0.30, if the trend is gentle and gradual and never drops more than 30% from any running peak.

## What the refinement diagnostics buy you

The initial axis ships six per-row refinement diagnostics: `peakDay`, `peakDailyTokens`, `troughDay`, `troughDailyTokens`, `maxDrawdownDurationDays`, `recovered`. The `c858481` refinement adds `partialRecoveryRatio`. Reading the table column-wise:

- `peakDay`/`peakDailyTokens` localises the *running peak* that bounded the maximum drawdown — for src-1 that is 2025-10-13 (181k tokens, 73 days into the source's window), for codex that is 2026-04-13 (183M tokens, the third day of an 8-day window). This is not necessarily the source's all-time max; it is the running peak that maximises the (D_i − D_j) / D_i ratio. In a series with two competing drops ("90% drop early, then climb back, then 50% drop late"), the early drop wins, and `peakDay` points to the early peak, not the later one.
- `troughDay`/`troughDailyTokens` is the post-peak minimum that closes the worst drawdown. For claude-code, 2026-03-06 trough at 78k tokens against the 12.3M peak two days earlier — a 99.36% fractional drop in 48 hours.
- `maxDrawdownDurationDays` is the number of UTC days from `peakDay` to `troughDay`. The values span 2 days (claude-code) to 43 days (src-1). This is *not* the recovery duration; it is the descent duration.
- `recovered` is a boolean: did *any* post-trough day reach back to the peak? src-1 yes (the 181k peak from October was eclipsed when the source resumed in late February), claude-code yes, codex yes; hermes / openclaw / opencode all `no` at the live-smoke timestamp. The `no`s are interesting precisely because hermes / openclaw are at peak token volume in late April and the dispatcher has been routing through them ever since.
- `partialRecoveryRatio` (the `c858481` refinement) is the ratio of the post-trough recovery to the original drawdown. `0` = stuck at trough, `1` = exactly back to peak, `> 1` = exceeded peak. The codex `prr = +2.16` says codex's post-trough run reached more than triple the original 183M peak — i.e., the 96.85% MDD captured a transient outage early in the codex window, and the subsequent volume blew through the old peak.

The five-band `drawdownRegime` classifier ships with cutoffs `0` (flat), `< 0.25` (shallow), `< 0.50` (moderate), `< 0.90` (severe), `≥ 0.90` (catastrophic), plus a sixth `degenerate` band for `n < 2`. The cutoffs map onto the live-smoke table cleanly: opencode at `0.5449` is the lone interior-of-severe sample (just over the 0.50 boundary), the two next-up at hermes `0.8999` and openclaw `0.8796` are upper-severe (just under the 0.90 catastrophic boundary), and the three catastrophic samples are all very deep into the `≥ 0.90` band (`0.9685`, `0.9936`, `0.9984`). Zero rows in `flat / shallow / moderate / degenerate` is itself a finding: every source the dispatcher routes through has had at least one severe peak-to-trough event. That is consistent with bursty token usage — sources cluster their work, hit a peak, and then sit idle or get rotated out.

## What "first PATH-DEPENDENT axis" implies for the next 50 axes

If you take the structural claim seriously, axis-145 opens a wing of the axis programme that has been entirely empty:

- **Conditional-drawdown / drawdown-at-risk (CDaR / DaR)** — the average drawdown beyond the q-th quantile of the drawdown distribution. Chekhlov-Uryasev-Zabarankin define CDaR as a coherent risk measure that smooths MDD's brittleness to a single peak-trough pair. Natural axis-146 candidate.
- **Time-under-water** — the fraction of days the series spends below its running peak. Pure pathwise scalar, range `[0, 1]`, also permutation-non-invariant.
- **Pain index / Ulcer index** — the L^2 norm of the drawdown curve (vs MDD's L^∞ norm). Smoother, more sensitive to many small drawdowns vs one big one.
- **Run-length / longest monotone-down run** — pure ordinal pathwise quantity; the longest streak of consecutive days where each day's tokens are below the previous day's.
- **Sequential change-point dating** — Pettitt's test, CUSUM, Buishand's range. These date the most significant break in the series and report a scalar test statistic; both are pathwise.
- **Stationarity / unit-root tests** — ADF, KPSS, PP. Boolean / scalar pathwise outputs that the cross-source matrix can rank.
- **Spectral / wavelet axes** — dominant frequency, spectral entropy, wavelet energy at scale-k. These are functions of the Fourier / wavelet transform of the sequence, hence non-invariant under permutation.

The point of listing these is not to pre-commit the queue — it is to note that the empty cell axis-145 just landed in is *spacious*. The 144-axis ledger covered the permutation-invariant subspace nearly exhaustively (the `0.6.392 CHANGELOG` enumerates 28 distinct concentration / inequality / diversity functionals already shipped). The path-dependent subspace has *one* axis in it. By the same accretion rate the queue showed for the f-divergence family (six axes in two weeks: 126, 127, 128, 129, 130, 131) we should expect 5-7 more pathwise axes before the queue starts looking for the next empty cell.

## Where the pew-insights queue stops being a "rolling histogram" and becomes a "trajectory ledger"

The original `0.1.0 — 2026-04-23` release shipped four commands: `digest`, `status`, `sources`, `doctor`. The `digest` command's output was *the rolling histogram* — token totals by day, source, model, hour-of-day, with `--since` and `--json`. Every axis 1-144 added another scalar (or scalar pair, for the directional half-pairs) to the rolling-histogram view. The view was still essentially a snapshot: "as of this moment, here are the per-source distributions, and here are 144 numbers summarising them".

Axis-145 changes the genre. To compute MDD you must traverse the day-axis in order. The output is no longer a snapshot — it is a *trajectory* statistic. The refinement diagnostics make this explicit: `peakDay` and `troughDay` are dated UTC-days, not summary scalars. Adjacent axes (CDaR, time-under-water, change-point dates) will all need to expose dated events, dated regime changes, dated cohort transitions. The `digest` command will probably need a new output mode that returns *event series* rather than histograms.

The `gc-runs` command (added in `0.6.x` per the same `CHANGELOG`) and the `compact` infrastructure that backs it become more important under this regime: trajectory statistics are stable only if the underlying day-totals series is stable across re-computations. If `gc-runs --keep N` ever drops a `runs/` entry that was the source-of-truth for a peak day, MDD on that source jumps. The doctor hint copy was already updated — `RUNS_DIR_LARGE` recommends `pew-insights gc-runs --keep 1000`, `QUEUE_NEVER_COMPACTED` recommends `pew-insights compact --confirm` — but the larger question is whether trajectory axes need their own retention contract: "any day in the support of a per-source MDD computation MUST NOT be archived without flushing the daily aggregate first". That contract does not exist yet; the test count grew 5 → 41 in the `0.1.x` release notes and would need to grow again to cover trajectory-axis retention invariants.

## What axis-145 does NOT do, and why that matters

Axis-145 is *cross-source* in the sense that it ranks sources by a single scalar, but the scalar is computed *per-source* on the per-source day series. It does not couple sources. A coupled pathwise axis — e.g., "the cross-correlation of source A's day series with source B's day series at lag 0", or "the date of the regime change in the A-vs-B token ratio" — would land in a different cell again: `(cross-source, path-dependent, coupled)`. That cell is also empty.

This matters because the anti-correlation observation that comes up repeatedly in the dispatcher history notes — "openclaw saturates while hermes drops", "opencode peaks while codex collapses" — is exactly the kind of phenomenon a coupled-pathwise axis would surface as a single number. MDD on each source separately can show that openclaw and hermes both have severe drawdowns, but it cannot show that their drawdowns are *aligned* in time. The cross-correlation lag-0 of the two MDD curves would.

So axis-145 is the first step into a wing of axes, but it is not yet the wing's deep interior. It is the threshold.

## The lineage one more time, citation-first

To make the audit trail explicit:

- `c858481` — `feat: refine axis-145 with partialRecoveryRatio scalar field`
- `0a18d1d` — `docs(changelog): add axis-145 entry with live-smoke output`
- `f59ec8f` — `chore: bump version 0.6.391 -> 0.6.392`
- `b839bf5` — `feat: add axis-145 daily-token-max-drawdown-rate`

The four-commit shape is the standard pew-insights axis-shipping shape: implementation commit → version bump → changelog with live-smoke → refinement. The dispatcher history note records `tests 11939->11958 (+19 17 main +2 refinement) live-smoke 6 sources 13.14B tokens` for the feature-family run that landed it (`{"ts": "2026-05-03T19:44:41Z", "family": "posts+feature+metaposts", ..., "HEAD=c858481"}`). The `13.14B tokens` is the same number that appears in the `CHANGELOG` table header — the audit closes.

Axis-145 is shipped, the orthogonality argument is in the changelog, the refinement diagnostic is in. The next axis the queue picks will tell us whether axis-145 was a one-off into the path-dependent wing or the start of the wing's first cohort.
