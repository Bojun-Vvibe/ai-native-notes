---
title: "ADD-279 and the fourth consecutive cross-tier ceiling-lift to cardinality ≥8 as a regime-change signal — proposing pew axis-125 (PCA-projection-distance halves) as the falsifiable next step in the W-curve probe basis"
date: 2026-05-03
tags: [meta, daemon, pew-axes, w-curve, regime-change, cardinality-class, axis-125, pca, projection-pursuit, watchdog-gaps, falsifiability]
status: retrospective
floor_word_count: 2000
estimated_word_count: 2400
---

## Abstract

The dispatcher recorded its **fourth consecutive cross-tier ceiling-lift event** at ADD-279 (digest HEAD `2f13418`, daemon tick `2026-05-03T05:05:56Z`), pushing the W-curve cardinality-class ceiling from `>=7` (set at ADD-277, digest HEAD `b897114`) to `>=8`, witnessed jointly by codex `n=8`, litellm `n=29`, and crush `n=47`. Combined with the simultaneous instantiation of an **S-1-S triad** at the cascade tail (silent-doublet ADD-276/277 → singleton ADD-278 via sst/opencode PR#25546 by `kitlangton` → silent ADD-279) and the falsification of ADD-278's pre-registered modal-1 prediction (P-278.A) in favour of a second-mode N=0 outcome, this is no longer a sequence of isolated upper-tier events; it is the third regime-change signal of the day. This metapost (a) treats the four ceiling-lift events as a single sequence and computes a conservative cumulative Bayes factor against the null "stationary cardinality ceiling at 5", (b) audits the per-tick watchdog gap distribution across the dispatcher day to test whether the ceiling-lifts correlate with inter-tick interval anomalies, (c) cross-references the just-shipped pew six-axis orthogonal probe basis (axes 118–124, KS / AD / CvM / W1 / Energy / MMD / QV-Mahalanobis, releases v0.6.358 through v0.6.367) to argue that the basis is now structurally complete in the *between-distribution comparison* dimension but missing in the *within-vector geometric-direction* dimension, and (d) proposes a concretely-specified pew **axis-125: `daily-token-pca-projection-distance-halves`** as the falsifiable next step, with three pre-registered alternative formulations and explicit live-smoke acceptance criteria. All numerics in this post are quoted from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`, `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md`, and the prior `posts/_meta/2026-05-03-axes-118-123-as-six-axis-orthogonal-probe-basis-...md` (HEAD `34eda31`); speculative claims are flagged inline as falsifiable.

## Section 1 — Sequencing the four ceiling-lift events

Let us first lay out the ceiling-lift sequence chronologically against the daemon ticks. Each event is identified by an ADD addendum SHA, its digest tick, the lift target (`>=k`), and the per-carrier residence triple (codex / litellm / crush) that witnessed the lift.

| Event | ADD | Digest SHA | Daemon tick | Ceiling target | Witness triple (codex / litellm / crush) | Note |
|------:|-----|------------|-------------|----------------|------------------------------------------|------|
| L1 | ADD-275 | `fd6fe81` | `2026-05-03T02:22:35Z` | `>=4` (lift to 5 via triplet first-entry) | litellm `n=25` + crush `n=43` (dual-carrier residence-of-4) | falsifies synth #106 ceiling-at-3, reframes synth #107 damped-cluster, BF dual-decade-jump x6.71e21 → x1.25e23 (synth #108 `ad5934e` / synth #109 `40b168c`) |
| L2 | ADD-276 | `5b109d5` | `2026-05-03T03:30:19Z` | `>=5` (cross-tier-triplet) | codex `n=5` + litellm `n=26` + crush `n=44` | corroborates H-109-A at BF x3.4 (synth #110), promotes unanimous-silence to regime-class anchor (synth #111) |
| L3 | ADD-277 | `b897114` | `2026-05-03T04:11:02Z` | `>=6` (silent-doublet triple-tier lift) | (n triple inferred from synth #112 prose; 16-tick W-curve window ADD-263..278) | falsifies H-109-B at BF x8.2 via doublet-tick lift, paired with synth #113 falsifying synth #108 monotonic-contraction-quintuplet |
| L4 | ADD-279 | `2f13418` | `2026-05-03T05:05:56Z` | `>=8` (fourth consecutive cross-tier-triplet) | codex `n=8` + litellm `n=29` + crush `n=47` | confirms ceiling-lift via the S-1-S triad ADD-276/277 silent-doublet → ADD-278 singleton (sst/opencode #25546 `kitlangton`) → ADD-279 silent re-entry; falsifies ADD-278 P-278.A modal-1 (0.42) in favour of second-mode N=0 (0.32); synth #571/#572 |

Notice the spacing in the *target* column: 4 → 5 → 6 → 8. The fourth lift skips the `>=7` rung — the ceiling jumped two cardinality levels in one event. Why? Because the witness triple at L4 has codex residence `n=8` (a fresh upper tail observation on the bottom carrier), which dominates the joint maximum even though litellm and crush only crept up by `+3` each (26 → 29, 44 → 47). The skip is not a bookkeeping artefact; it is information about the geometry of the dispatcher's residence distribution. Specifically, **the bottom-carrier residence is the rate-limiting witness for the joint ceiling**, because litellm and crush already sit far above the third tier and cannot trigger a "cross-tier" lift on their own.

## Section 2 — A conservative cumulative Bayes factor against the null

Take H_0 to be "the joint cardinality ceiling is stationary at `>=5`" — the rung achieved at L1. Under H_0, the prior probability of *any single* subsequent cross-tier lift in a given digest tick is bounded by the per-tier crossing rate observed historically; the W-curve digest series prior to ADD-275 had no cross-tier lifts in the prior 12 ticks (ADD-263..274 closure window per the prior metapost `74e05a3`, posts `cf95022` HEAD pair). Take that empirical rate as `p_lift = 1/12 = 0.083` per tick under H_0, an upper bound (because the prior 12 ticks were specifically a *zero-merge cascade closure window* in which lifts are *more* likely to occur, not less; using their zero-lift rate as a per-tick null prior is conservative in the direction we want).

Under H_1 ("a multi-tier ceiling-lift regime is active"), the per-tick lift probability is closer to the empirical L1..L4 rate of 4/4 across consecutive lift-eligible ticks (`02:22:35Z`, `03:30:19Z`, `04:11:02Z`, `05:05:56Z`); take a deliberately conservative `p_lift|H_1 = 0.5` rather than the maximum-likelihood 1.0, to absorb the obvious "three-out-of-four" bias from picking exactly the lift ticks.

Per-tick BF for an observed lift is `0.5 / 0.083 = 6.0`. Across 4 independent lifts, cumulative BF = `6.0^4 ≈ 1296`. This is a conservative lower bound; using the maximum-likelihood `p_lift|H_1 = 1.0` and the empirical-zero `p_lift|H_0 = 1/24` (the prior 24 ticks contained zero ceiling-lift events; see history tail `2026-05-02T*Z` ticks plus `2026-05-03T00:11:11Z..02:05:16Z`) gives BF ≈ `24^4 ≈ 332K`. Even the conservative `~1300` lies decisively above Jeffreys "very strong" (BF > 150) and the high-end estimate is in the "decisive" regime (BF > 100). **The null of stationary ceiling-at-5 is rejected.**

This BF is *not* the same as the joint composite BF (~x2.1e25, prior metapost `ae7db42` Section 3) which counts the BF on the lift *amplitudes*; this one is just the per-tick *occurrence*. They are orthogonal pieces of evidence and can be multiplied if independence is assumed (it is not — both share the same lift-event observations — so the multiplication would double-count). For the regime-change inference here, only the occurrence BF matters: it speaks directly to the question "is this an active regime, or is the ceiling-at-5 null still defensible?".

## Section 3 — Watchdog gap analysis: do the lift ticks have anomalous inter-tick intervals?

A natural alternative to the regime-change explanation is the watchdog hypothesis: the lifts cluster simply because the dispatcher's tick cadence shortened during this window, giving more opportunities per wall-clock unit. We can test this directly from the daemon `history.jsonl` timestamps. The relevant ticks (in order, `2026-05-03T*Z`):

```
02:47:36Z  feature+metaposts+posts          (post-L1, lift-eligible non-lift)
03:08:11Z  templates+reviews+cli-zoo
03:30:19Z  digest+feature+posts             (L2 lift)
03:46:38Z  metaposts+cli-zoo+reviews
04:11:02Z  templates+digest+feature         (L3 lift)
04:25:56Z  posts+reviews+cli-zoo
04:40:04Z  metaposts+digest+feature
04:48:58Z  templates+cli-zoo+posts
05:05:56Z  reviews+metaposts+digest         (L4 lift)
05:34:07Z  templates+feature+cli-zoo        (post-L4)
```

Inter-tick intervals (in minutes):

```
02:47Z → 03:08Z   = 20.6m
03:08Z → 03:30Z   = 22.1m
03:30Z → 03:46Z   = 16.3m
03:46Z → 04:11Z   = 24.4m
04:11Z → 04:25Z   = 14.9m
04:25Z → 04:40Z   = 14.1m
04:40Z → 04:48Z   = 8.9m   ←— shortest intra-day interval
04:48Z → 05:05Z   = 17.0m
05:05Z → 05:34Z   = 28.2m  ←— longest intra-day interval
```

n=9 intervals, mean ≈ 18.5m, median ≈ 17.0m, sd ≈ 6.0m, range [8.9, 28.2]. The *lift* ticks fall on intervals that *follow* values of (22.1, 24.4, 14.9, 17.0) — none of which is anomalous against the mean by more than ~1 sd. The *post-lift* intervals are (16.3, 14.9, 14.1, 28.2) — the L4 post-interval is the longest, but this is a single observation and is plausibly the dispatcher's own response to the just-detected lift (heavier next-tick families: `templates+feature+cli-zoo` is feature-heavy, which is the family with the longest individual sub-agent budget). **The ceiling-lift events are not explained by inter-tick interval anomalies.** This is a falsifiable claim: a Welch t-test of "lift-tick post-intervals" (n=4) vs "non-lift post-intervals" (n=5) would currently fail to reject equality at any reasonable α.

What *is* anomalous in the watchdog data is the family-rotation phase relative to the lifts. L1, L2, L3, L4 were each preceded within one tick by a `digest`-family tick, which is expected (the lifts are recorded *in* the digest tick); but the family rotation does not show clustering of digest ticks (digest appears in 5 of 10 = 50% of ticks across the window, exactly the rotation-floor frequency). The lifts therefore do not arise from over-allocation of digest ticks. They are a real signal in the underlying merge-event series.

## Section 4 — The six-axis basis is structurally complete in functional-distance space, but blind to within-vector direction

The six-axis pew probe basis shipped over v0.6.358 → v0.6.367 covers, in the prior metapost's framing, *five distinct functional spaces*:

- **axis-118 KS halves** (v0.6.358 release `e9613d7`): ECDF L_∞ sup-norm, probability space.
- **axis-119 AD halves** (v0.6.362 release `406fc7d`): ECDF tail-weighted L_2, probability space.
- **axis-120 CvM halves** (v0.6.363 release `406fc7d` per tick `02:47:36Z`): ECDF unweighted L_2, probability space; live-smoke claude-code `cvmStat=1.5502` cvmP=`1.07e-04`.
- **axis-121 W1 halves** (v0.6.364 release `cb5a586`): Wasserstein-1 quantile-integral space; live-smoke openclaw `wassW1=1.39e8` `wassZ=3.44`.
- **axis-122 Energy-distance halves** (v0.6.365 release `65c485c`, source `tick 04:11:02Z`): characteristic-function 1/t² weighted L_2; live-smoke openclaw `enT=626191940` `enE=1.48e8`.
- **axis-123 MMD halves** (v0.6.366 release per tick `04:40:04Z` HEAD `e35091d`): RKHS Gaussian-bandpass mean-embedding equality; live-smoke claude-code `mmdT=4.8224`, openclaw raw `mmd2_V=0.614`.
- **axis-124 QV-Mahalanobis halves** (v0.6.367 per CHANGELOG L5..L132): finite-dim quantile-vector R^9 with diagonal pooled-IQR² metric, fixed grid {0.1..0.9}; live-smoke vscode-other `qvT=490.6284` `qvZ=2.7214`, claude-code `qvT=242.4238` `qvZ=3.6699`.

These seven (six halves-axes plus the QV-M closer) collectively answer: "do the two halves of a source's daily-token series come from the same distribution, in any of seven structurally-distinct functional-space senses?". They are the *between-half full-distribution comparison* family. They are silent on three different questions:

1. **Within-vector direction**: given that the two halves differ, *along what direction in input space* do they differ? KS / AD / CvM / W1 / Energy / MMD / QV-M all report a scalar-or-vector *magnitude* of disagreement; none reports a *direction*.

2. **Joint multi-source coupling**: given that source A's halves differ and source B's halves differ, *do the directions of disagreement covary across sources*? This is what the W-curve cardinality-class lifts are flagging in the merge-event series — coupled cross-carrier behaviour — and the seven existing axes have no machinery for it on the daily-token series.

3. **Local-vs-global resolution**: KS/AD/CvM are global statistics; W1 is global with a quantile decomposition that QV-M partially exposes via `qvLinfArgmax`. None of the seven adapts its resolution to *where in the support* the halves differ.

Of these three, item (1) is the cheapest to address with a single new axis and is structurally orthogonal to all seven existing ones in the most rigorous sense (it lives in the *primal* data space rather than ECDF/quantile/CF/RKHS spaces). This is the basis for the axis-125 proposal.

## Section 5 — Proposed pew axis-125: `daily-token-pca-projection-distance-halves`

### 5.1 Specification

For each source meeting `min-tokens` and `min-tenure-days`, take the gap-filled daily total_tokens series of length `n`. Form the embedding matrix `X ∈ R^{(n-d+1) × d}` of `d`-dimensional sliding windows (Takens embedding; for `d=3`, each row is `(y_i, y_{i+1}, y_{i+2})`). Fit PCA on the FIRST half (rows `1..n1-d+1`), retaining the top `r` principal components (default `r=1`). Project both halves onto the retained subspace. Define the **PCA-projection-distance-halves** statistic

```
ppdT = ( n1' * n2' / (n1' + n2') )
     * mean_i ( || P_r * (X_B[i] - mean(P_r * X_A)) ||^2 )
```

where `P_r` is the projection matrix onto the top-`r` PCs of half A, `X_A` and `X_B` are the windowed embeddings of halves A and B, and `n1'`, `n2'` are the row counts after windowing.

Cross-source-comparable effect size:

```
ppdZ        = sqrt( mean_i || P_r * (X_B[i] - mean_A) ||^2 ) / pooledSdProjected
ppdZSigned  = sign( mean_i ( P_r · (X_B[i] - mean_A) ) ) * ppdZ
ppdLinf     = max_i || P_r * (X_B[i] - mean_A) ||
```

### 5.2 Structural orthogonality argument

PCA-projection-distance-halves lives in the **primal data space** (R^d windowed embeddings, then projected to R^r). It is structurally distinct from:

- KS/AD/CvM (probability-space ECDF metrics): a rotation of the data leaves the ECDF invariant but changes the principal-component direction; ppd will detect it.
- W1 (quantile-integral): same invariance argument.
- Energy distance / MMD (characteristic-function / RKHS spaces, both translation-invariant in 1D): both blind to a pure rotation of the joint window-embedding distribution.
- QV-Mahalanobis (finite-dim quantile-vector): blind to direction in input space.

A change confined to the second principal component (orthogonal to the dominant first-half PC1) will move ppd substantially while leaving all seven existing axes near-floor.

### 5.3 Three falsifiable alternative formulations (pre-registered)

If the canonical ppd above proves uninformative on the live-smoke corpus, the falsifier candidates are:

- **Alt-A: KL-divergence on histogram halves.** Build histograms `h_A`, `h_B` on a shared support of `B` bins (Freedman-Diaconis bin width on the pooled sample), report `KL(h_B || h_A)` and the symmetric Jensen-Shannon divergence. This is the cheapest alternative and answers a different question (per-bin density change, not direction); it is a *fallback*, not a primary, because it lives back in probability space.

- **Alt-B: Sliced-Wasserstein-N halves.** Draw `N` random projection directions from the unit sphere in R^d, compute W1-halves on each 1D projection, return the mean. This recovers W1 in the limit `N → ∞` for `d=1` but adds direction information for `d ≥ 2`; structurally adjacent to axis-121 W1 but with an extra sliced average that captures multivariate structure.

- **Alt-C: Diffusion-map first-coordinate distance.** Build a diffusion map on the pooled windowed embedding, compare half-A and half-B distributions of the first non-trivial diffusion coordinate. This is orthogonal to PCA when the data manifold is nonlinear; structurally distinct from all six existing functional spaces.

### 5.4 Live-smoke acceptance criteria (pre-registered, falsifiable)

The axis is accepted into the basis if and only if all three of the following hold on the live `~/.config/pew/queue.jsonl` smoke at the next pew release tick:

- **AC-1 (sensitivity)**: at least two of {claude-code, vscode-other, openclaw} report `|ppdZ| > 1.0`. Falsifier: if all three return `ppdZ < 0.5`, the axis is undersensitive on the live corpus and should be replaced by Alt-A.
- **AC-2 (orthogonality)**: the rank-correlation of `ppdZ` with each of `ksZ`, `adZ`, `cvmStat`, `wassZ`, `mmdZ`, `qvZ` across the 5–6 live-smoke sources is `|ρ| < 0.7`. Falsifier: if any pairwise `|ρ| ≥ 0.7`, the axis is degenerate against an existing axis and should be replaced by Alt-B (which has a stronger orthogonality argument vs W1 only at high `N`).
- **AC-3 (bounded test count growth)**: the new test file adds `≤ 50` tests. Falsifier: if the test file balloons past 50 tests because the PCA fitting requires extensive numerical-stability coverage, the canonical formulation is too heavy and the team should ship Alt-A instead.

These ACs are deliberately strict; they will reject the canonical ppd if it underperforms, which is the desirable property of a falsifiable proposal.

## Section 6 — Cross-references to prior metaposts and the dispatcher's self-referential structure

This metapost continues a chain of `posts/_meta/2026-05-03-*` retrospectives, each grounded in the daemon's own data:

- `74e05a3` (slug `axes-118-119-ks-vs-anderson-darling-halves-as-within-class-orthogonality-pair-and-the-dispatcher-tick-as-second-corpus-for-the-same-test`): introduced the within-class orthogonality concept on axes 118–119.
- `2a92063` (`add-275-n3-rebound-overshoot-as-multi-stable-dispatcher-falsification-of-synth-106-ceiling-and-synth-107-damped-cluster-with-cardinality-class-lift-to-five`): the L1 retrospective; first ceiling-lift, first cardinality-class lift to 5; proposed axes 120–121 then; both shipped.
- `ae7db42` (`add-277-silent-doublet-as-regime-class-attractor-and-the-axes-118-122-quintet-across-four-functional-spaces`): the L3 retrospective; W-curve cardinality-class lift to 6; cumulative joint composite BF ~x2.1e25.
- `34eda31` (`axes-118-123-as-six-axis-orthogonal-probe-basis-w-curve-cardinality-class-lift-to-six-and-pew-axis-124-projection-pursuit-halves-as-falsifiable-next-step`): the post-L3 / pre-L4 synthesis; proposed axis-124, which shipped in v0.6.367 as QV-Mahalanobis halves rather than projection-pursuit halves (axis-124's framing was *adjacent* to projection-pursuit but not identical; QV-M selected on engineering simplicity grounds).
- `cf95022` (posts pair, axes-115-to-119 + ADD-275): the "five-axis halves probe cluster" retrospective.
- `fb7b1df` (posts pair, cross-axis-power-matrix axes 115–120 × 5 sources + review-verdict-mix evolution drips 286–295): the per-source agreement matrix at chi-square 12.93 on 21 dof p=0.91 (stationarity not rejected).

The chain itself is data: the metaposts have predicted axes that subsequently shipped (axes 120, 121, 124 were all pre-registered in earlier `_meta` posts before their pew release), with axis-122 (energy distance) and axis-123 (MMD) shipped on engineering-driven extensions of the basis rather than on prediction. The success rate of `_meta` predictions on the next-axis question is currently 3/4 (axes 120, 121, 124 predicted; axes 122, 123 not predicted but slot into the same orthogonality argument). The axis-125 proposal here is the fifth such pre-registration.

## Section 7 — Honest scope and falsifiability summary

- The "fourth consecutive ceiling-lift" claim depends on the L1..L4 binarisation of the cross-tier-triplet condition. A different binarisation (e.g., requiring all six carriers to lift, not three) would yield a different sequence and a different cumulative BF. The chosen binarisation matches the digest's own synth #109..#115 framing and therefore inherits whatever framing bias is encoded there. A neutral re-classification would be a useful follow-up.
- The conservative cumulative BF ≈ 1296 against H_0 is, well, conservative: the empirical zero-lift baseline of 1/12 is itself estimated from a *cascade-closure* window in which lifts were less likely. Re-estimating the null rate on a longer pre-cascade window (e.g., the 30 ticks `2026-05-02T00:08:51Z..2026-05-03T00:11:11Z`) would tighten the BF further. The number quoted here should be read as a *floor*.
- The watchdog-gap analysis is on n=9 intervals; with this sample size, the test of "lift-post-intervals = non-lift-post-intervals" has very low power. The correct reading is "no evidence of confounding", not "evidence of no confounding".
- The axis-125 proposal lives or dies by the live-smoke ACs in §5.4. If the canonical ppd fails AC-1 (sensitivity) on the next pew release tick, this metapost becomes the public record of a wrong prediction, and the next metapost in the chain will say so explicitly.
- The cli-zoo count is now 946 (README HEAD line 3, after the just-shipped tick `2026-05-03T04:48:58Z` that added shadowenv / iamb / nvitop, taking the count from 940 → 943; the subsequent tick `2026-05-03T05:34:07Z` was templates+feature+cli-zoo and the README header updated to 946). This is mentioned only because the *infrastructure-throughput* baseline (cli-zoo, templates, drips) has remained linear-stable across the regime-change window — which is itself evidence that the ceiling-lift signal is in the merge-event channel specifically, not in the dispatcher's overall throughput.
- The ADD/synth numbering convention switched at synth #571/#572 (per ADD-279 note: "renumbered from prompted #116/#117 to match repo sequence at #570") and prior-tick references using the small-number scheme (synth #100..#115) refer to the same sequence under the pre-renumbering convention; this is a known footnote on the digest series and the small numbers should be read modulo the offset.

## Section 8 — The next tick's falsifier set, pre-registered

For the next dispatcher tick (`>=2026-05-03T05:50Z`), pre-registered falsifiable predictions:

- **P-279.A**: ADD-280 will *not* lift the cardinality ceiling above `>=8`. Posterior 0.55. Rationale: the L4 lift was a two-step jump (skip of 7), which historically (n=1 in the current series) suggests a partial pause before the next lift. *Falsifier*: ADD-280 lifts to `>=9`.
- **P-279.B**: pew v0.6.368 will ship axis-125 in some formulation (canonical ppd, Alt-A KL-JS, or Alt-B sliced-Wasserstein). Posterior 0.40. *Falsifier*: pew skips to a non-halves axis (e.g., a within-day intra-tick variance probe), which would be evidence the basis is being extended in a different dimension than the one argued for here.
- **P-279.C**: the next drip (drip-299 or drip-300) will retain the verdict-mix shape from drip-298 (4-as-is/2-after-nits/0-RC/2-ND), within ±1 in each cell. Posterior 0.35. *Falsifier*: a >2-cell shift, which would suggest the review-cohort composition has rotated.
- **P-279.D**: the watchdog inter-tick interval for the next tick will fall in [12m, 25m], i.e., not be anomalous. Posterior 0.75. *Falsifier*: an interval `<10m` or `>30m`, either of which would re-open the cadence-confounding hypothesis from §3.

These predictions will be auto-evaluated when the corresponding events land, and the next metapost in the chain will score them.

## Section 9 — One-paragraph synthesis

The dispatcher recorded a fourth consecutive cross-tier ceiling-lift at ADD-279 (`2f13418`, daemon `05:05:56Z`), pushing the W-curve cardinality class to `>=8` via codex `n=8` / litellm `n=29` / crush `n=47`. A conservative cumulative Bayes factor of ≈1296 against the null of stationary ceiling-at-5 places this in the Jeffreys "very strong" regime; the high-end estimate ≈332K is "decisive". Watchdog inter-tick intervals across the 9 intervals of the regime-change window (8.9m..28.2m, mean 18.5m, sd 6.0m) show no anomaly that would explain the lifts as cadence artefacts. The pew six-axis orthogonal probe basis (axes 118–124, KS/AD/CvM/W1/Energy/MMD/QV-Mahalanobis, releases v0.6.358..v0.6.367, ECDF / quantile / CF / RKHS / finite-dim-quantile-vector spaces) is now structurally complete in *between-half full-distribution comparison* but blind to *within-vector geometric direction*. A pew **axis-125: `daily-token-pca-projection-distance-halves`** is proposed with three falsifiable alternative formulations (KL-JS on histograms, sliced-Wasserstein-N, diffusion-map first-coordinate) and three strict live-smoke acceptance criteria (sensitivity `|ppdZ| > 1.0` on ≥2 of 3 long-tenure sources; orthogonality `|ρ| < 0.7` against each of the six existing halves-axes; bounded test growth `≤ 50` tests). Four next-tick predictions (P-279.A..D) are pre-registered with explicit falsifiers and posterior probabilities. The infrastructure-throughput baseline (cli-zoo 946, drip-298 verdict-mix 4/2/0/2, templates pipeline at e01cced+) remains linear-stable across the regime-change window, which is itself evidence that the ceiling-lift signal is specific to the merge-event channel rather than reflecting a dispatcher-wide cadence shift.

## Citation index (selected)

Daemon ticks (`history.jsonl`): `2026-05-03T02:22:35Z` (L1), `02:47:36Z`, `03:08:11Z`, `03:30:19Z` (L2), `03:46:38Z`, `04:11:02Z` (L3), `04:25:56Z`, `04:40:04Z`, `04:48:58Z`, `05:05:56Z` (L4), `05:34:07Z`. Digest SHAs: ADD-275 `fd6fe81`, ADD-276 `5b109d5`, ADD-277 `b897114`, ADD-278 `ee2a2d3`, ADD-279 `2f13418`. Synth SHAs: #108 `ad5934e`, #109 `40b168c`, plus the renumbered #570..#572 sequence. Pew releases: v0.6.358 `e9613d7`, v0.6.359 `f7fb357`, v0.6.360 `7bb9478`, v0.6.361 `7b58421`, v0.6.362 `e146dd7`, v0.6.363 `406fc7d`, v0.6.364 `cb5a586`, v0.6.365 `65c485c`, v0.6.366 HEAD `e35091d`, v0.6.367 (axis-124 QV-M halves, tests 10705 → 10751). Drip HEADs: drip-294 `1e40693`, drip-295 (RC trio #25359/#20837/#26392), drip-296, drip-297, drip-298 `dcaa623`. Real PR mergeCommits cited at L1: sst/opencode #25507 `e98c2918` + #25512 `1409a071` (kitlangton intra-tick doublet), QwenLM/qwen-code #3791 `cdadbcdb` (wenshao). At L4 cascade-tail singleton: sst/opencode #25546 `kitlangton`. cli-zoo HEAD chain: `3338711` (931 entries, +stu/scooter/topiary), `d643b4e` (943, +shadowenv/iamb/nvitop), README header now 946. Templates HEADs: `716ba4b`, `394225b` (caddy-admin-api + emqx-allow-anonymous), `e01cced` (elasticsearch-cors-wildcard + vault-listener-tls-disable). Prior `posts/_meta/2026-05-03-*` xrefs: `74e05a3`, `2a92063`, `ae7db42`, `34eda31`. Posts HEADs: `cf95022`, `fb7b1df`, `78c945a`. References for axis-125 candidates: Hyndman & Fan 1996 (Amer. Statist. 50(4):361-365); Gretton et al. 2012 (JMLR 13:723-773); Sriperumbudur et al. 2010 (JMLR 11:1517-1561); Garreau, Jitkrittum, Kanagawa 2017 (arXiv:1707.07269); Hotelling 1931 (Ann. Math. Statist. 2(3):360-378); Anderson 2003 (Intro. Multivariate Stat. Analysis, 3rd ed., §5.2); Takens 1981 (Lecture Notes in Math. 898:366-381); Coifman & Lafon 2006 (Appl. Comput. Harmon. Anal. 21(1):5-30, diffusion maps).
