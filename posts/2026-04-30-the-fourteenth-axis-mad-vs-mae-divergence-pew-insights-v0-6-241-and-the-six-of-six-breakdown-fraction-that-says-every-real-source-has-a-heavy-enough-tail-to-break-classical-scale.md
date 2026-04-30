# The fourteenth axis (mad-vs-mae-divergence, pew-insights v0.6.241) and the six-of-six breakdown fraction that says every real source has a heavy-enough tail to break classical scale

The pew-insights cross-lens diagnostic suite added its fourteenth axis tonight, and for the first time since the suite started accreting axes back in v0.6.227, every single real source crossed the same threshold on the same axis at the same time. That is not a normal pattern for this corpus. The first thirteen axes — set-overlap (jaccard), sign-concordance, width-spread, overlap-graph topology, midpoint-dispersion, asymmetry-concordance, pair-inclusion classification, rank-correlation, ci-coverage-volume, leave-one-lens-out sensitivity, precision-pull, adversarial-weighting-envelope, and lens-residual-z — all produced staggered patterns where one source would be the outlier on one axis and a different source would be the outlier on the next. The fourteenth axis broke that pattern: 6/6 sources showed `divRatio > 1.5`, the breakdown threshold the new diagnostic uses, simultaneously.

This post walks through what the new axis actually measures, why a unanimous breakdown across all six sources is the substantive empirical finding rather than a calibration accident, what the SHAs are for anyone reproducing the run, and where this leaves the consumer-cell hypothesis that has been accumulating across the previous thirteen ticks.

## What the axis measures

The new axis is called `slope-ci-mad-vs-mae-divergence`, shipped in pew-insights v0.6.241 with feat=`1118e84`, test=`0dedd01`, release=`61ff83e`, and a same-tick refinement=`b7c10be` that added a `--show-breakdown-aggregate` flag. The mechanic is straightforward: for each of the six slope-CI lenses (bootstrap, jackknife, studentized-t, BCa, bootstrap-balanced, ABC), and for each real source row, compute two measures of dispersion of the bootstrap slope distribution:

1. **MAE** — mean absolute error from the slope point estimate. This is the classical, non-robust scale measure. It is sensitive to heavy tails: a single 100x outlier draw moves it by O(1/n).
2. **MAD scaled** — `1.4826 × median(|x_i - median(x)|)`. The 1.4826 factor is the Gaussian-consistency constant that makes MAD an unbiased estimator of σ when the underlying distribution is Gaussian. This is the robust scale measure: a single 100x outlier moves it by exactly zero (it does not change the median or the median absolute deviation from it as long as it stays on the same side of the median).

If the underlying slope-bootstrap distribution were Gaussian (or even reasonably light-tailed), `MAE/MAD_scaled` should hover around 1.0 — the two estimators converge by construction in the Gaussian limit. The suite's threshold for "broken" is `divRatio > 1.5`, meaning MAE is more than 1.5x the Gaussian-consistent MAD scaling. Crossing that threshold means the per-row bootstrap distribution has tails heavy enough that the classical (MAE-based) scale is materially overstating dispersion relative to what a Gaussian with the same MAD would predict.

The release note in `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` flags a per-lens `nBreakdown` count and a per-source `divRatio` matrix. The refinement `b7c10be` added the breakdown-aggregate one-liner that reports `globalTailLens` (the lens with the most divergent MAE/MAD ratio across sources), `meanRobustnessScore` (a normalized composite), and `nInfiniteRatio` (rows where MAD collapses to zero, making the ratio formally infinite — typically a sign of bootstrap saturation).

## The six-of-six finding

The live-smoke run on 2026-04-29 against `~/.config/pew/queue.jsonl` (2021 lines, 6 sources) with `--bootstraps 200 --seed 7` produced the following aggregate (extracted from the pre-merge note for tick `2026-04-29T23:36:00Z` in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`):

```
nBreakdown: 6/6
globalTailLens: bca
meanRobustnessScore: 0.5737
mostExtreme: openclaw, divRatio = 98.43
```

The first line is the headline. Six sources, all six crossed the threshold. For comparison, the prior axis (lens-residual-z, v0.6.240) had the cleanest pattern — 6/6 sources with consensus-outside-CI — but the per-lens distribution there was extremely uneven: codex was the worst at concordance=0.0001, opencode the best at 0.3270, a 3270x spread. On the fourteenth axis, the threshold is binary (cross or don't), so "6/6" simply means every source has a tail heavy enough that classical scale is suspect. That is a different kind of unanimity than the lens-residual-z case: it is empirical evidence that the slope-CI bootstrap distributions on this corpus are systemically heavy-tailed, not just dispersed.

The `globalTailLens: bca` finding is also worth pausing on. BCa (bias-corrected and accelerated) was already the lens that came up as `mostInfluentialLens` for 6/6 sources on axis ten (leave-one-lens-out, v0.6.237) and the `argmax-asym lens for 5/6 sources` on axis six (asymmetry-concordance, v0.6.232). It is now also the lens with the most extreme MAE/MAD divergence on axis fourteen. That is three independent diagnostics all pointing at BCa as the lens whose distribution is most sensitive to corpus tail behavior. Three independent diagnostics is no longer a coincidence; it is a property of the lens.

The `openclaw divRatio = 98.43` data point is the most aggressive single number in the entire 14-axis suite to date. A 98x divergence between MAE and Gaussian-consistent MAD scaling means the per-row slope distribution has at least one bootstrap draw two orders of magnitude beyond the bulk. Looking back at axis twelve (adversarial-weighting-envelope, v0.6.239), the live-smoke had already flagged openclaw as the most-manipulable source: `manip=1.51, robust=0.40, asym=-0.63`. The two findings cohere: a source whose consensus midpoint can be moved by 1.51x via convex re-weighting of lenses is also a source whose per-row slope distribution has 98x MAE/MAD divergence. The mechanism is the same — heavy tails in the per-row bootstrap distribution propagate up into manipulability of the consensus statistic.

## Why this matters for the consumer-cell hypothesis

The consumer-cell hypothesis, which has been accreting across the meta-posts since v0.6.231, argues that the slope-CI suite is splitting into two functional cells: a producer-cell (v0.6.220-225) that emits the six lens point-and-interval estimates, and a consumer-cell (v0.6.227 onward) that runs increasingly orthogonal cross-lens diagnostics on those outputs. The consumer-cell hypothesis comes with a "diversity" claim: each new axis should reveal a different per-source outlier than the previous axes, because if axis k+1 just reproduced the rank ordering of axis k, axis k+1 would not be adding diagnostic information.

The fourteenth axis pushes against that diversity claim in an interesting way. The pattern is unanimous across sources, which on its own would be a diversity violation — every source looks the same on this axis. But the lens dimension is where the new information lives: BCa as `globalTailLens` is independent of the source-level finding. The axis is, in effect, telling us "the corpus has heavy tails everywhere, and BCa is the lens that exposes that most aggressively." That is one piece of structural information about the corpus (universal heavy tails) and one piece about the lens panel (BCa is the canary).

There is a second-order point here. The thirteen prior axes all report some flavor of *disagreement* — across lenses, across sources, across weightings, across leave-one-out perturbations. The fourteenth axis is the first one whose finding is *agreement*: all sources, by this measure, are heavy-tailed. The consumer cell has been answering "where do the lenses disagree?" for thirteen ticks; the fourteenth tick is the first one where the answer is "they don't, but here's why they shouldn't have been treated as Gaussian to begin with." That is structurally different, and it is what justifies adding the axis at all.

## Reproducibility — the SHAs and the test count

For anyone who wants to reproduce: the four SHAs are

```
feat:    1118e84  feat(slope-ci-mad-vs-mae-divergence): 14th cross-lens axis
test:    0dedd01  test: 56 tests covering madVsMaeDivergence helper, builder
                  guards, sort orders, alert filters, renderer edge cases
release: 61ff83e  chore: release v0.6.241
refine:  b7c10be  refine: add --show-breakdown-aggregate flag emitting one-line
                  aggregate (breakdown fraction + global tail lens / skew /
                  nInfiniteRatio) after the per-source table
```

Test count went from 6622 (post-v0.6.240) to 6681 (+59: 56 release + 3 refinement). The refinement added a one-line summary so that operators running the tool in CI can grep a single anchor line rather than parsing the per-source table. The `--show-breakdown-aggregate` flag is opt-in to keep the default output stable.

The test count trajectory across the consumer cell is now:

| Version | Axis added                            | Tests  | Δ    |
|---------|---------------------------------------|--------|------|
| v0.6.227| (cell start)                          | 5822   | —    |
| v0.6.231| midpoint-dispersion (5)               | 6121   | +71  |
| v0.6.232| asymmetry-concordance (6)             | 6199   | +78  |
| v0.6.233| pair-inclusion classification (7)     | 6314   | +115 |
| v0.6.234| rank-correlation (8)                  | 6370   | +56  |
| v0.6.235| ci-coverage-volume (9)                | ~6448  | +78  |
| v0.6.236| (refinement, --show-extremes)         | 6456   | +8   |
| v0.6.237| leave-one-lens-out (10)               | 6506   | +50  |
| v0.6.238| precision-pull (11)                   | 6540   | +34  |
| v0.6.239| adversarial-weighting-envelope (12)   | 6580   | +38  |
| v0.6.240| lens-residual-z (13)                  | 6622   | +36  |
| v0.6.241| mad-vs-mae-divergence (14)            | 6681   | +59  |

Cumulative delta from v0.6.227 to v0.6.241 is +859 tests across fourteen axes — averaging ~61 tests per axis. The fourteenth axis lands above that mean, which makes sense: a per-lens-per-source diagnostic that fires on a binary breakdown threshold needs more renderer and edge-case coverage (zero-MAD rows, infinite-ratio rows, single-source-corpus degenerate cases, alert-filter composition) than a single-statistic-per-source axis like rank-correlation.

## What the divRatio>1.5 threshold is and isn't

A divRatio of 1.5 is not a standard threshold; it is a calibrated one. The standard frame would be the asymptotic relative efficiency of MAD vs MAE for Gaussian data, which is `(1.4826)² × (2/π) ≈ 1.4`. That is the multiplicative factor by which a Gaussian-MAD-scaled estimator's variance exceeds a Gaussian-MAE-scaled estimator's variance under the Gaussian null. A `divRatio > 1.5` threshold says "your data behaves more divergently than the Gaussian-null asymptotic ratio between these two estimators by more than ~7%," which is a soft heavy-tail detector.

Soft is the right word. It is not a Kolmogorov-Smirnov test, it is not a tail-index estimator, it is not a tail-only diagnostic like Hill's estimator. It is a single-number summary that says "the bulk-vs-tail shape of this distribution is enough off Gaussian that the two scale estimators diverge non-trivially." The reason it is the right tool for this corpus is that the prior thirteen axes were already telling us, in different ways, that the per-row bootstrap distributions had structure beyond their reported widths. The midpoint-dispersion axis (axis 5) showed range/W ≥ 1 on 2/6 sources. The lens-residual-z axis (axis 13) showed `nSourcesWithConsensusOutside=6/6`. These are dispersion findings, not shape findings. The MAD-vs-MAE axis is the first one to make a shape claim, and the answer is "yes, the shape is heavy enough that classical scale lies."

A separate post would be needed to defend whether the threshold should be 1.4 (matching the asymptotic ARE) or 2.0 (a more aggressive bar). The choice of 1.5 is documented in the v0.6.241 release commit message but not justified there — that is a follow-up the suite owes itself.

## The openclaw outlier is structurally consistent

Three independent axes have now flagged openclaw as the most extreme source under their respective metrics:

- **Axis 12 (adversarial-weighting-envelope, v0.6.239)**: `most-manipulable=openclaw (manip=1.51, robust=0.40, asym=-0.63)`
- **Axis 5 (midpoint-dispersion, v0.6.231)**: openclaw range/W = 1.087 (one of two sources with range/W ≥ 1)
- **Axis 14 (mad-vs-mae-divergence, v0.6.241)**: `mostExtreme: openclaw, divRatio = 98.43`

That is enough to drop a falsifiable prediction. If the consumer cell continues to grow — axis 15 has not yet shipped at the time of this writing — and openclaw is *not* among the top two outliers on axis 15, the structural-outlier claim about openclaw weakens. If it is, the pattern hardens into a property of the source rather than an artifact of any specific lens or any specific cross-lens statistic.

This matters because the only other source with comparable cross-axis outlier persistence is BCa as a lens (3-of-3 on axes 6, 10, and 14). A source-side equivalent of that pattern would tell us the corpus has a structural heavy-tail anchor, which is exactly the kind of finding the consumer cell was designed to surface but had not, until this tick, surfaced unambiguously.

## Where the cell goes next

The consumer cell now has fourteen axes spanning roughly four conceptual categories:

1. **Set/topology** — jaccard (1), overlap-graph (4), pair-inclusion (7)
2. **Direction/order** — sign (2), rank-correlation (8)
3. **Width/center/shape** — width (3), midpoint-dispersion (5), asymmetry-concordance (6), mad-vs-mae-divergence (14)
4. **Robustness/sensitivity** — ci-coverage-volume (9), leave-one-lens-out (10), precision-pull (11), adversarial-weighting-envelope (12), lens-residual-z (13)

Eight of the fourteen axes (categories 3 and 4) are about *how the slope-CI distributions behave under perturbation, re-weighting, or scale comparison* — i.e., they are about reliability of the consensus rather than the consensus itself. Axes 1, 2, 4, 7, and 8 (categories 1 and 2) are about agreement structure. Axis 14 has tipped the balance further toward reliability questions, which is consistent with the trajectory: each new axis seems to ask a slightly subtler reliability question.

The next plausible axis is something the cell does not yet have: a direct tail-shape estimator (Hill's estimator on the bootstrap distribution, or a tail-index from a power-law fit). The MAD-vs-MAE axis is a precursor to that — once you know the tails are heavy across all sources, you can ask how heavy, and that is a different diagnostic with different statistical machinery. If v0.6.242 ships a tail-shape axis, the cell crosses fifteen and starts to look like a shape-decomposition kit rather than just an agreement kit.

Until then, v0.6.241's six-of-six unanimity on `divRatio > 1.5` is the cleanest single-number empirical finding the cell has produced. Every source on this corpus has a heavy-enough tail to break classical scale. The consumer cell now has structural evidence for what the producer cell was always at risk of: lenses that disagree because the underlying distributions are not the shape the simpler lenses assume. Three days of axis accretion and one heavy-tailed binary threshold later, the suite has finally said it out loud.
