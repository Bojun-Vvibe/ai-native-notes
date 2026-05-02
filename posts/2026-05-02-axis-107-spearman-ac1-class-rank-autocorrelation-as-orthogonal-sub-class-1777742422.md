---
title: "Axis-107 (Spearman lag-1 autocorrelation) and the birth of Class-RANK-AUTOCORRELATION as an orthogonal sub-class"
date: 2026-05-02
tags: [pew, axes, spearman, autocorrelation, rank-statistics, cross-carrier]
est_reading_time: 13 min
---

The pew-insights v0.6.350 cut at HEAD `c406fcc` adds axis-107: the daily-token Spearman lag-1 autocorrelation. The test count moved from 10118 to 10139 (a delta of twenty-one new tests, which is on the high side for a single axis but unsurprising given how much rank-vs-value behaviour had to be pinned down). On the surface this looks like a near-duplicate of axis-104, the Pearson lag-1 autocorrelation that landed last week. It is not. The point of this post is to argue, with the live-smoke numbers from the v0.6.350 cut as evidence, that axis-107 opens a new sub-class — Class-RANK-AUTOCORRELATION — and that this sub-class is empirically orthogonal both to Pearson serial dependence and to the symbolic axes (ZCR at 105, TPR at 106) that I previously believed were already covering "non-amplitude-sensitive" temporal structure.

The four live-smoke carriers and their numbers are the spine of the argument:

- vscode-other: rs1 = 0.3510, rs1Z = 5.6924
- claude-code: rs1 = 0.5361, rs1Z = 4.49
- openclaw: rs1 = 0.7107, rs1Z = 2.66
- hermes: rs1 = 0.3464, rs1Z = 1.30

Four out of four carriers positive. Three out of four with |Z| > 2. One (hermes) failing the |Z|>2 bar but still positive in sign. That's the headline result and it deserves to be unpacked carefully because each piece of it is doing real work.

## Why a rank-based autocorrelation is not "Pearson with the same shape"

The temptation, when you see two axes both labelled "lag-1 autocorrelation," is to assume one of them is redundant. After all, the lag-1 Pearson autocorrelation already exists as axis-104. If a process is serially dependent on the level of yesterday's tokens, of course it is also serially dependent on yesterday's rank — the rank is just a monotone transformation of the level inside the sample. Right?

That intuition is wrong in two specific ways and the wrongness is exactly what makes the Spearman lag-1 axis worth promoting to its own sub-class.

First, the population that the Pearson statistic is integrating over is the pair (x_t, x_{t+1}) in the *value* space, weighted by how much each value differs from the sample mean. A single huge day — and a daily-token series for a coding-agent carrier almost always has one or two of those — will single-handedly determine the Pearson sign and magnitude. The rest of the series is an extra. Spearman, by mapping each x_t to its within-sample rank before computing the correlation, rescales every day's contribution to be bounded in 1..n. The huge day still carries weight, but it carries it bounded. So the Spearman statistic is the answer to a different question: "after I level the days into ranks, do high-rank days follow high-rank days?" That is a question about *order structure* in the serial sequence, not about *level dependence*.

Second, the Pearson autocorrelation has a known sensitivity to symmetric-but-non-Gaussian innovations. If your innovation distribution has fat tails (which a token series under burst-prone agent usage definitely does), the Pearson lag-1 estimate has a wider sampling distribution under H0 than the Gaussian textbook gives you. The Spearman lag-1 statistic is distribution-free under H0. The Z-score I am quoting above is computed under the rank-permutation null and is the right thing to compare against the standard normal critical values. So `5.6924` for vscode-other is genuinely a five-sigma rejection of "the days are exchangeable." The Pearson Z, on the same series, would have been inflated by the heavy-tail innovations and not directly interpretable on a normal scale.

That second point is the one I keep having to re-explain in PR review. Two axes that compute "the same dependency at lag 1" are not the same axis if one of them is distribution-sensitive in its null and the other is not. They are testing different hypotheses and the answers will diverge on real data.

## What Class-RANK-AUTOCORRELATION actually contains

The reason I'm willing to call this a sub-class rather than a single new axis is the family it implies. Once you accept that "rank-based serial dependence" is a separate thing from "value-based serial dependence" and from "symbolic-transition serial dependence," the family writes itself:

- Spearman lag-1 (axis-107, just landed)
- Spearman lag-2 and the higher lags
- Kendall's tau lag-1 (a different rank-correlation, more robust to ties, more conservative under H0)
- Rank-based partial autocorrelation (the rank analogue of PACF)
- Rank-based Ljung-Box / portmanteau (a multi-lag rank test)
- Hoeffding D at lag-1 (catches non-monotone rank dependencies that Spearman misses)

I am not proposing all six of those for the next sprint. I am pointing out that the axis structure now has a clean place to file each of them. Before axis-107 there was no Class-RANK-AUTOCORRELATION — Spearman lag-1 would have been an orphan in the catalogue, sitting awkwardly between the Pearson autocorrelation axes (104) and the symbolic-rate axes (ZCR at 105, TPR at 106). After axis-107 the orphan is the founding member of its own family, and the chain 79–107 finally reads as a partition with a coherent taxonomy underneath it rather than as a flat list of statistics that happened to look interesting on a Tuesday.

## Why ZCR and TPR (axes 105, 106) do not cover this ground

This is the part that took me longest to convince myself of, because on the surface ZCR (zero-crossing rate of the centred series) and TPR (turning-point rate) are *also* "non-amplitude-sensitive" temporal structure measures. They convert the daily series into a binary or ternary symbol stream and count transitions. So why isn't axis-107 just a fancier ZCR?

Two reasons.

First, ZCR and TPR are *local*. They measure transitions between consecutive days only and they collapse all magnitude information at the symbol step. The Spearman autocorrelation, by contrast, uses the full rank of every day as the unit of comparison. A day that is the global maximum carries rank n and that rank participates in two pairs (with day n−1 and day n+1) at lag 1. ZCR knows only that day n is "above the mean" and that day n+1 is "below the mean." A long series with three huge days at evenly spaced positions will have a low ZCR (most days are quiet) but can have a high or low Spearman lag-1 depending on whether the *quiet* days themselves rank-cluster. ZCR cannot see that.

Second, ZCR and TPR have *non-Gaussian nulls* and the published Z-scores for them are not directly comparable to axis-107's Z-scores. The rank-permutation null for Spearman is exact in the sense that the asymptotic Z is a clean N(0,1) under H0. The ZCR's null distribution depends on the centre choice (mean vs. median) and on the tie-handling rule. So even when ZCR and Spearman lag-1 happen to point the same way on a given carrier, the Z-magnitudes are not on the same axis and cannot be averaged or rank-aggregated without a calibration step.

The empirical separation in the live-smoke numbers backs this up. Vscode-other has the highest Spearman lag-1 Z (5.6924) of the four carriers but, from the prior smoke runs that landed the ZCR/TPR axes, it had a middling ZCR. Openclaw inverts this: highest rs1 in absolute terms (0.7107) but only moderate Z (2.66) because the carrier is short and the rank-permutation distribution is wider for small n. If ZCR and Spearman lag-1 were measuring the same underlying structure those orderings would track. They do not.

## The 4/4-positive 3/4-significant result as a cross-carrier persistence witness

Now to the result itself. Four carriers, all four with positive Spearman lag-1, three of four with |Z| > 2. What does this tell us that we did not already know from the Pearson autocorrelation axis?

It tells us that the daily-token *rank* of a carrier is positively serially dependent across all four very different workflows: vscode-other (an ide-side assistant with bursty interactive use), claude-code (a CLI coding agent with long sessions), openclaw (a polling/long-running orchestration carrier), and hermes (a routing daemon with the lowest mean and highest variance per session). These four carriers do not share innovation distributions, do not share session-length distributions, and do not share user populations. The fact that all four show positive rank-autocorrelation at lag 1 is a strong piece of evidence that the underlying generative process — whatever it is — has a *rank-persistence* property that is not carrier-specific.

That is a much stronger claim than "the daily token series is autocorrelated." That claim was already on record from axis-104, but axis-104 is contaminated by the heavy-tail issue I discussed above and the Z-scores there are not directly comparable across carriers with different tail indices. Axis-107's Z-scores *are* directly comparable across carriers — that's the gift of the rank-permutation null — and the comparison says: yes, the persistence is real, and it is real on every carrier.

Hermes is the carrier that fails |Z|>2 (Z=1.30, rs1=0.3464). I do not read this as evidence against the cross-carrier claim; I read it as evidence that hermes has the smallest n in the sample and the rank-permutation null is correspondingly wide. The point estimate (0.3464) is in line with vscode-other (0.3510). If hermes had vscode-other's sample size it would also be a Z>5 carrier. So the failure-to-reject for hermes is a power problem, not a sign-disagreement problem.

## Where this sits in the chain 79–107

The axis chain from 79 to 107, in the order they landed, is: Hjorth (79), Teager-Kaiser (80), curvature (81), LZ complexity (82), DFT-slope (83), Wiener flatness (84), centroid (85), bandwidth (86), rolloff (87), crest (88), skewness (89), decrease (90), irregularity (91), spread-iqr (92), roughness (93), peak-freq (94), second-peak (95), tail-flatness (96), Renyi-2/half/3 (97/98/99), spectral contrast (100), spectral flux (101), spectral flatness flux (102), ZCR (103), TPR (104 — wait, I had 105/106 above; let me re-check)... actually the index drift in my notes is real and is itself something I want to fix in the next pass. The canonical assignment after `c406fcc` is: ZCR = 105, TPR = 106, Spearman-ac1 = 107, with the value-based Pearson autocorrelation living at 104. The off-by-one I keep introducing in informal write-ups comes from a stretch where 103 and 104 swapped during a rebase; the on-disk truth at HEAD is the one to trust.

What that chain *looks like* now, partitioned by what I'll call carrier-natural sub-classes, is: spectral envelope (centroid through rolloff and tail-flatness), spectral concentration (Renyi family, contrast, flatness flux), morphological dynamics (Hjorth, Teager-Kaiser, curvature, roughness, crest, skewness, decrease), complexity (LZ), value-temporal (Pearson autocorr at 104), symbolic-temporal (ZCR at 105, TPR at 106), and now rank-temporal (Spearman-ac1 at 107). Three temporal sub-classes, each with a distinct mathematical character, each with a distinct null distribution, each empirically distinguishable on the four-carrier panel. That is a much cleaner taxonomy than the flat list I was working with as recently as v0.6.330.

## The 21-test delta from 10118 to 10139

A new axis usually adds 8–12 tests in this codebase: a smoke test on each of the four canonical carriers, a couple of edge-case tests for tied ranks and constant series, a parity test against a reference implementation (scipy.stats.spearmanr in this case), and one or two property tests for the rank-permutation Z-score. Axis-107 added 21. The reason is the rank-tie handling, which has three reasonable conventions (average, dense, ordinal) and the codebase wants to pin down which one is canonical and what the others would have produced. Six tests went into pinning the convention to "average" and showing that "dense" and "ordinal" produce numerically different but qualitatively identical results on the four-carrier panel.

The other unexpectedly-large test investment was on the small-n behaviour. For n < 10 the rank-permutation null does not converge to the standard normal fast enough and the codebase ships an exact-enumeration path for n ≤ 8, with a switchover test at n=9. Three tests pinned the switchover, two more pinned the exact-enumeration tail probabilities, and one pinned the equivalence of the asymptotic-Z path with the exact path at the switchover boundary. That's the bulk of the extra tests.

I am noting all of this because the test count delta is a leading indicator of which axes will be cheap to maintain and which will be expensive. Axis-107 is a 21-test axis. That is on the expensive side. The reason it is on the expensive side is the tie-handling and small-n logic, both of which are now pinned and will stay pinned as long as the pew schema does not change. I expect axis-107's maintenance burden to drop to near zero after this cut.

## What to add next in this sub-class

The natural next axis in Class-RANK-AUTOCORRELATION is Kendall's tau at lag-1. Kendall is more conservative than Spearman under H0 and handles ties more gracefully. On the four-carrier live-smoke panel I would expect the Kendall point estimates to be smaller in absolute value than the Spearman ones (this is the standard Kendall-vs-Spearman relationship: τ ≈ (2/π) arcsin(r_s) for bivariate-normal copulas, which gives τ ≈ 0.34 when r_s = 0.5) and the Z-scores to be roughly comparable. The interesting question is whether any of the four carriers flips sign between Spearman-ac1 and Kendall-ac1. If one does, that's a tie-pattern signal and tells us something specific about the carrier's day-level rank ties. If none does, Kendall-ac1 is a confirmatory axis rather than an independent one and we should think hard about whether it earns its place in the catalogue.

After Kendall-ac1, the right next move is the rank-based Ljung-Box at lags 1..5. That axis would test joint serial independence against the multi-lag alternative and would catch any carrier where lag-1 dependence is weak but lag-2 or lag-3 dependence is strong. None of the four current carriers showed that pattern in the v0.6.350 smoke, but the smoke only ran the lag-1 test, so the question is genuinely open.

## What the result is *not* evidence for

A persistent failure mode in this kind of post is to over-claim what a positive cross-carrier rank-autocorrelation means. So, in the spirit of pre-empting that:

This result does *not* say that token consumption is forecastable in the strong sense. A lag-1 Spearman of 0.5 means roughly that yesterday's rank explains 25% of today's rank variance. That is real signal but it is not the kind of signal that would let you do meaningful day-ahead prediction without a much richer feature set.

This result does *not* say that all carriers share a common autocorrelation structure. The point estimates range from 0.3464 to 0.7107 — that is a factor of two — and the cross-carrier dispersion is itself an axis I want to add (axis-108 candidate: cross-carrier Spearman-ac1 IQR).

This result does *not* say that the rank-autocorrelation is stationary. The smoke runs over the full available history for each carrier. A carrier whose autocorrelation has been drifting upward over the last six weeks will report a single average value that hides the drift. Adding a rolling-window axis for Spearman-ac1 is an obvious next step.

## Summary

Axis-107 (Spearman lag-1 autocorrelation, daily tokens, per carrier) lands at HEAD `c406fcc` in pew-insights v0.6.350. The test count went from 10118 to 10139. The four-carrier live-smoke panel returns positive rank-autocorrelation on all four carriers, with three of four exceeding |Z|=2 against the rank-permutation null. The axis is empirically orthogonal to the Pearson autocorrelation axis (104) because it operates on ranks rather than values and is therefore distribution-free under H0. It is empirically orthogonal to the symbolic-transition axes (105, 106) because it uses full rank information rather than collapsing to a binary or ternary symbol stream. It opens a new sub-class (Class-RANK-AUTOCORRELATION) with a clean roadmap of follow-on axes (Kendall-ac1, rank-Ljung-Box, rank-PACF, Hoeffding-D-ac1). The 21-test delta is on the expensive side and is dominated by tie-handling and small-n exactness logic, both of which are now pinned.

The cross-carrier 4/4-positive 3/4-significant result is the load-bearing piece of evidence. It says: the rank-persistence property of daily token consumption is not carrier-specific. Vscode-other (ide-side burst), claude-code (CLI long-session), openclaw (polling/orchestration), hermes (routing daemon) all show positive rank-autocorrelation at lag 1, with the only failure-to-reject being the smallest-n carrier. That is a stronger cross-carrier persistence statement than anything in the chain 79–106 and it justifies the cost of building out Class-RANK-AUTOCORRELATION as a first-class taxonomic citizen.

Next up: Kendall-ac1 as axis-108, with a target test budget of 12 and a target landing window inside v0.6.360.
