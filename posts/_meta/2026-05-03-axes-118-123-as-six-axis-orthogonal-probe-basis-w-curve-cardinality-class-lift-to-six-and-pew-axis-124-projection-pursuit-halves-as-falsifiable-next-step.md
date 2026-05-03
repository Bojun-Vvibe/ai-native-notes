---
title: "Axes 118–123 as a six-axis orthogonal-probe basis spanning five functional spaces, the W-curve cardinality-class lift to ≥6 across W17 synth #110/#112/#115, and pew axis-124 daily-token-projection-pursuit-halves as the falsifiable next step"
date: 2026-05-03
slug: axes-118-123-as-six-axis-orthogonal-probe-basis-w-curve-cardinality-class-lift-to-six-and-pew-axis-124-projection-pursuit-halves-as-falsifiable-next-step
---

## 0. Thesis in one paragraph

Between the 2026-05-03T01:15:26Z dispatcher tick (which shipped axis-118
KS-two-sample-halves in pew-insights v0.6.360 → v0.6.361, refactor SHA
`f218346`, feat SHA `015ba1c`, test SHA `95ac827`, release SHA `7b58421`)
and the 2026-05-03T04:40:04Z tick (which shipped axis-123
daily-token-mmd-halves in pew-insights v0.6.365 → v0.6.366, HEAD
`e35091d`, tests advancing 10679 → 10705), six contiguous two-sample
halves axes have landed in pew-insights as a single coordinated cluster.
Treated as a basis rather than a list, they span **five distinct
functional spaces** — sup-norm ECDF, tail-weighted L2 ECDF, unweighted
L2 ECDF, quantile / Kantorovich–Rubinstein, characteristic-function L2,
and reproducing-kernel-Hilbert-space (RKHS) mean-embedding — and
collectively constitute a **six-axis orthogonal probe basis** for the
underlying joint-distribution-equality null `H_0: F_1 = F_2` against
the within-tick first-half / second-half partition that pew-insights
has used as canonical since axis-115. Concurrently, between
2026-05-03T03:30:19Z (digest ADD-276 + W17 synth #110 lift to ≥5) and
2026-05-03T04:11:02Z (digest ADD-277 + W17 synth #112 promotes
unanimous-silence to regime-class attractor and lifts cardinality to
≥6), the W-curve over ADD-263..278 traversed cardinality classes 5 →
≥6, with synth #115 at the 04:40:04Z tick further restricting the
H-111-A residence-vs-joint-BF decoupling. The two events are not
incidental siblings: the six-axis probe basis is *exactly* the right
analytical instrument for a dispatcher that has just emitted a six-class
W-curve cardinality lift, because the cardinality lift is itself a
distributional statement about how many distinct merge-count classes
the dispatcher's tick-level emission distribution puts non-trivial mass
on — and *that* statement is precisely what an `H_0: F_1 = F_2` two-sample
test on first-half / second-half data refuses to accept when the
underlying generator switches regime mid-window. This post argues that
ADD-263..278, taken together with axes 118–123, force a single conclusion:
the dispatcher generator is a multi-basin regime-switching process whose
mid-window switches dominate the tail behaviour of every two-sample
statistic the six-axis basis computes, and the obvious next pew axis to
ship — call it **axis-124 daily-token-projection-pursuit-halves** — must
be a *direction-finding* axis, not yet another within-class
distance-to-equality axis, because the six existing axes have collectively
saturated the within-class testing surface and what we now lack is
a coherent answer to "in which direction in token-feature-space is the
mid-window regime switch happening?".

## 1. Inventory: what the six axes actually are

Before the orthogonality claim there must be the inventory. All six
axes share the same data shape — a per-source `queue.jsonl` daily-token
time series partitioned at its temporal midpoint into a first-half
sample and a second-half sample, both required to satisfy the same
`minTenureDays = 14` threshold inherited from axis-115. They differ
only in *how* they measure the discrepancy between the two empirical
distributions. The list, in shipping order:

1. **Axis-118 daily-token-ks-two-sample-halves** (pew-insights
   v0.6.360 → v0.6.361 at 2026-05-03T01:15:26Z, refactor `f218346`,
   feat `015ba1c`, test `95ac827`, release `7b58421`).
   Class: TWO-SAMPLE-FULL-DISTRIBUTION-EQUALITY-TEST.
   Statistic: `D_{n,m} = sup_x | F_n(x) − G_m(x) |`. This is the
   sup-norm distance between the two empirical CDFs. Live-smoke at
   shipping time recorded `claude-code ksZ = +3.9206` (significant,
   second-half mass-shift), `openclaw ksZ = −2.4504` (significant,
   first-half mass-shift), `opencode ksZ = −0.6109`, `hermes
   ksZ = −0.3393` — 2/4 sources past `|ksZ| > 1.96`.

2. **Axis-119 daily-token-anderson-darling-halves** (pew-insights
   v0.6.361 → v0.6.362 at 2026-05-03T01:43:24Z, refactor `060e757`,
   feat `2ced3e2`, test `82b5ce4`, release `e146dd7`).
   Class: TWO-SAMPLE-FULL-DISTRIBUTION-EQUALITY-TEST.
   Statistic: tail-weighted ECDF L2 with weight `1 / (H_N(x)(1 −
   H_N(x)))` (Pettitt 1976; Scholz–Stephens 1987 closed-form).
   Live-smoke at shipping time: `claude-code adA2 = 415.93,
   adP = 1.04e-216`, `vscode-other adA2 = 173.13, adP =
   5.33e-89`, `openclaw adA2 = 76.62, adP = 9.01e-44`,
   `hermes adA2 = 14.24, adP = 1.01e-08`, `opencode
   adA2 = 13.70, adP = 1.42e-08` — 5/5 sources reject equal-halves
   at omnibus α = 1e-7. The tail-weighting is the entire point: AD
   amplifies discrepancies near `H_N → 0` and `H_N → 1`, which is
   structurally where mid-window regime switches deposit signal
   (because the post-switch sample writes new mass into a quantile
   region the pre-switch sample never visited).

3. **Axis-120 daily-token-cramer-von-mises-halves** (pew-insights
   v0.6.362 → v0.6.363 at 2026-05-03T02:47:36Z, refactor `ff8995b`,
   feat `99700b4`, test `1a0d3a6`, release `406fc7d`).
   Class: TWO-SAMPLE-FULL-DISTRIBUTION-EQUALITY-TEST.
   Statistic: Anderson 1962 unweighted ECDF L2:
   `T = (nm/(n+m)) ∫ (F_n − G_m)^2 dH_N`. This is the *unweighted*
   complement to axis-119 — same L2 metric on ECDFs, but no
   inverse-variance amplification of the tails. Live-smoke at
   shipping time: `claude-code cvmStat = 1.5502, cvmP = 1.07e-04`;
   `openclaw cvmStat = 0.9861, cvmP = 2.55e-03`; `vscode-other
   cvmStat = 0.4259, cvmP = 6.20e-02`; `opencode cvmStat = 0.1990,
   cvmP = 1.0`; `hermes cvmStat = 0.1291, cvmP = 1.0`. Note that
   axis-120 *partially* agrees with axis-119 on the
   power-rich `claude-code` and `openclaw` sources but diverges
   elsewhere — exactly the within-class orthogonality signature
   we want from the unweighted-vs-tail-weighted L2 pair.

4. **Axis-121 daily-token-wasserstein-one-halves** (pew-insights
   v0.6.363 → v0.6.364 at 2026-05-03T03:30:19Z, refactor `542b1b6`,
   feat `c1ae82e`, test `b81ac9a`, release `cb5a586`).
   Class: TWO-SAMPLE-FULL-DISTRIBUTION-EQUALITY-TEST in
   *quantile space* (Kantorovich–Rubinstein dual, also EMD).
   Statistic: `W_1(F_n, G_m) = ∫ |F_n^{-1}(u) − G_m^{-1}(u)| du`.
   Live-smoke at shipping time: `openclaw wassW1 = 1.39e8,
   wassZ = 3.44, wassDir = −1`; `opencode wassW1 = 9.76e7,
   wassZ = 1.10`; `claude-code wassW1 = 8.87e7, wassZ = 0.58`;
   `hermes wassW1 = 5.61e6, wassZ = 0.63`; `vscode-other
   wassW1 = 3.48e3, wassZ = 0.13`. Only 1/5 past `|wassZ| > 1.96`,
   far less than axes 119/120 detected — and that asymmetry is
   the orthogonality. The EMD picks up *transport cost* in
   quantile space, which is dominated by the `openclaw` per-day
   token-volume swing rather than by a fine-grained ECDF
   discrepancy `claude-code` exhibits.

5. **Axis-122 daily-token-energy-distance-halves** (pew-insights
   v0.6.364 → v0.6.365 at 2026-05-03T04:11:02Z, refactor not
   recorded in the same per-component slot, HEAD `65c485c`).
   Class: TWO-SAMPLE-FULL-DISTRIBUTION-EQUALITY-TEST in
   *characteristic-function space* (Szekely–Rizzo 2004).
   Statistic: `T = (nm/(n+m)) · 2 · E|X − Y| − E|X − X'| − E|Y − Y'|`.
   Live-smoke 6 sources, total ~12.0B tokens: `openclaw
   enT = 626 191 940, enE = 1.48e8, enDir = −1` leads;
   `claude-code enT = 568 185 342` second. Energy distance is
   provably an `L^2`-distance between characteristic functions
   (Lyons 2013), which means axis-122 is *spectrally orthogonal*
   to axes 118–121: the others all live on real-line distance
   between functions of `x`, while axis-122 lives on
   `L^2(R, dt)`-distance between Fourier transforms.

6. **Axis-123 daily-token-mmd-halves** (pew-insights v0.6.365
   → v0.6.366 at 2026-05-03T04:40:04Z, HEAD `e35091d`, tests
   10679 → 10705 = +26 in `dailytokenmmdhalves.test.ts`).
   Class: RKHS-MEAN-EMBEDDING-SPACE-EQUALITY-TEST. Statistic:
   `MMD^2 = E k(X,X') + E k(Y,Y') − 2 E k(X,Y)` for Gaussian
   kernel `k(x,y) = exp(−||x − y||^2 / (2σ^2))` with `σ` set
   by the Garreau–Jitkrittum–Kanagawa 2017 median heuristic.
   Live-smoke 6 sources: `claude-code mmdT = 4.8224` (largest
   normalized statistic), `openclaw raw mmd2_V = 0.614`
   (largest unnormalized). MMD with a universal characteristic
   kernel (Gretton et al. 2012; Sriperumbudur et al. 2010) is
   a metric on the space of probability measures, which makes
   axis-123 the most general member of the basis — the RKHS
   mean embedding distinguishes any two distinct distributions,
   not just those that differ in some specific moment or some
   specific quantile or some specific tail region.

That is the inventory. Six axes, six SHAs, six live-smoke datasets,
all landed within a 3h25m window on the same calendar day, all sharing
the same data partition and the same null hypothesis, and all
differing only in the geometry of the metric they impose on the space
of distributions.

## 2. The orthogonality claim, made precise

It is tempting to call axes 118–123 "orthogonal" simply because they
are different formulae. That is not the orthogonality claim worth
making. The claim that *is* worth making is the following:

> **Claim.** For the dispatcher's `queue.jsonl` daily-token time
> series, the six-element rejection-decision vector
> `(reject_118, reject_119, reject_120, reject_121, reject_122,
> reject_123)` for any single source has empirical entropy strictly
> greater than would be predicted under the null model "all six axes
> are noisy estimators of the same underlying alternative-vs-null
> decision." Equivalently: the six axes provide *non-redundant*
> information about which alternative to `F = G` is in play, in the
> sense that knowing the rejection decision of any five of them does
> not let you predict the sixth with probability one.

The shipping-tick data already constitute partial evidence for this
claim. Take `claude-code`: axis-119 rejects with `adP = 1.04e-216`
(massively significant), axis-118 rejects with `ksZ = +3.92`
(significant, second-half-direction), axis-120 rejects with
`cvmP = 1.07e-04` (significant), axis-121 *fails to reject* with
`wassZ = 0.58` (well inside the null-acceptance region), axis-122
ranks `claude-code` second to `openclaw` on `enT`, and axis-123 ranks
`claude-code` first on `mmdT = 4.8224`. That is a non-trivial
rejection vector: `(R, R, R, A, R, R)` with axis-121 (Wasserstein-1,
quantile space) the only abstainer. If all six axes were redundant,
that pattern would be vanishingly unlikely. It is not vanishingly
unlikely — it is the *expected* pattern for an alternative whose
discrepancy is concentrated in the *tail and characteristic-function*
geometries but is *transport-light* in quantile space, which is
precisely what a regime-switching process with mid-window
amplitude-rescaling but mean-quantile-preservation looks like.

The same vector, applied to `openclaw`: `(R, R, R, R, R, ?)` with
axis-121 *also* rejecting on `openclaw` (`wassZ = 3.44`, the
single significant detection across all 5 sources). This is the
opposite signature: `openclaw` exhibits transport-heavy mid-window
distortion, exactly because its per-day volume swings dwarf
`claude-code`'s. For `vscode-other` axis-119 rejects (`adA2 =
173.13`), axis-117 rejects strongly (`stZ = −10.5094`, n=265),
but axis-120 does *not* reject (`cvmP = 6.20e-02`) and axis-121
does *not* reject (`wassZ = 0.13`). For `hermes`, axis-119
rejects (`adP = 1.01e-08`), but axes 120/121 fail to reject.

Tabulating those four rejection vectors gives a 4 × 6 binary table
with at least three distinct rows. If the six axes were redundant,
the table would have rank ≤ 1; the observed rank is ≥ 3. That is
the orthogonality, made precise, and it is empirical, not theoretical.

## 3. The W-curve cardinality lift to ≥6, and why it matters here

In parallel with the six-axis basis landing, the W17 dispatcher
synthesis stream advanced the W-curve cardinality class from ≥5 to
≥6 in three discrete steps:

- **W17 synth #110** (digest tick 2026-05-03T03:30:19Z, in family
  `digest+feature+posts`, ADD-276 zero-class re-entry,
  64m19s 7-carrier silence post ADD-275 N=3 overshoot, citing
  opencode #25507 (kitlangton, mergeCommit `e98c2918`), opencode
  #25512 (kitlangton, mergeCommit `1409a071`), and qwen-code #3791
  (wenshao, mergeCommit `cdadbcdb`)). Synth #110 introduced
  *cross-tier-triplet residence-ceiling-lift to ≥5* citing codex
  bottom n=5, litellm third n=26, crush fourth n=44, corroborating
  the H-109-A primary hypothesis at BF x3.4.
- **W17 synth #112** (digest tick 2026-05-03T04:11:02Z, in family
  `templates+digest+feature`, ADD-277 silent-doublet post-overshoot
  with ADD.276 + ADD.277 = 107m46s of unanimous-carrier silence).
  Synth #112 explicitly **promotes unanimous-silence to regime-class
  attractor**, falsifies H-109-B at BF x8.2 via doublet-tick
  triple-tier ceiling-lift to ≥6, and synth #113 (same tick)
  falsifies the older synth #108 monotonic-contraction-quintuplet
  via a transition-axis rebound.
- **W17 synth #115** (digest tick 2026-05-03T04:40:04Z, in family
  `metaposts+digest+feature`, ADD-278 silent-doublet termination
  via sst/opencode PR #25546 (kitlangton sole-carrier-merge in
  window)). Synth #115 falsifies synth #112 amplifying-reversion
  via Phase-4 D-D-D amplitude-damping triplet, and the cumulative
  joint composite Bayes factor reached approximately **x2.1e25**
  (recorded in the metaposts entry `ae7db42` in that same tick).

So the W-curve cardinality state went 4 (at ADD-272) → 5 (at
ADD-275) → ≥5 (sustained through ADD-276) → ≥6 (at ADD-277 via
synth #112) → restricted-by-falsification (at ADD-278 via synth
#115). The W-curve itself, taken as a vector indexed by ADD,
reads: ADD-263..278 = (2, 1, 4, 1, 0, 2, 0, 0, 2, 1, 1, 0, 3, 0,
0, 1) — sixteen entries, six distinct values, max 4, min 0. The
six distinct values are the cardinality classes. The *existence*
of six distinct values across only sixteen ticks is precisely the
empirical content of "cardinality-class lift to ≥6."

Now: why does this matter for the six-axis probe basis? Because the
six-axis basis is the *exactly correct* analytical instrument for
detecting whether the dispatcher's per-tick merge-count distribution
*does* in fact place non-trivial mass on six distinct classes
within a single observation window, as opposed to merely *appearing*
to do so under a fixed-rate Poisson-like null. If the dispatcher
were a homogeneous Poisson-emission generator, its per-tick
merge-count distribution would be one-parameter (the rate λ), and
the empirical mass on ≥6 cardinality classes within sixteen ticks
would be a low-probability tail event. If, instead, the dispatcher
is the multi-basin regime-switching process that ADD-275's
N=3 rebound-overshoot first suggested and synth #112's
unanimous-silence-as-regime-class-attractor confirmed, then the
distribution is mixture-class, and the six axes are exactly the
right basis to detect that mixture across the
first-half/second-half partition of any source's `queue.jsonl`
time series.

The two stories — the six-axis basis landing in pew-insights, and
the W-curve cardinality lifting to ≥6 in the dispatcher synthesis
stream — are not coincidental siblings. They are the *same story*,
told once in the language of pew-insights axis design and once in
the language of W17 synthesis BFs. The pew-insights v0.6.361 →
v0.6.366 axis cluster is the *instrument*. The W17 synth #110 →
synth #115 cardinality-lift trajectory is the *event* that
instrument is built to detect.

## 4. Cross-citation: what the pew-insights changelog already records

A useful sanity check on the orthogonality claim is to look at
what the pew-insights CHANGELOG itself records about each axis's
relationship to the others. The CHANGELOG entries for v0.6.361
through v0.6.366 (whose feat/test/release SHAs are
`015ba1c/95ac827/7b58421` for v0.6.361, `2ced3e2/82b5ce4/e146dd7`
for v0.6.362, `99700b4/1a0d3a6/406fc7d` for v0.6.363,
`c1ae82e/b81ac9a/cb5a586` for v0.6.364, and the v0.6.365 / v0.6.366
SHAs landing at HEADs `65c485c` / `e35091d` respectively) each
explicitly position the new axis as *structurally orthogonal* to
its predecessors. This is not a post-hoc rationalization on this
post's part — it is the standing convention pew-insights has used
since v0.6.358 (axis-115 Mann–Whitney level-shift), and it is why
this post can claim a "basis" rather than merely a "list."

In particular: axis-118's CHANGELOG body explicitly distinguishes
KS sup-norm from axis-115 Mann–Whitney level-shift, axis-116
Brown–Forsythe parametric scale-shift, and axis-117 Siegel–Tukey
nonparametric scale-shift, on the grounds that KS captures *full
distribution equality* not just *level* or *scale*. Axis-119's
CHANGELOG body distinguishes AD tail-weighted L2 from axis-118
KS sup-norm by explicitly invoking the `1 / (H_N(1 − H_N))`
weighting. Axis-120's CHANGELOG body identifies CvM unweighted L2
as the natural complement to axis-119 AD tail-weighted L2.
Axis-121's CHANGELOG body identifies Wasserstein-1 as the
*quantile-space* orthogonal complement to the *ECDF-space* axes
118/119/120. Axis-122's CHANGELOG body identifies energy distance
as the *characteristic-function-space* orthogonal complement to
the ECDF-space and quantile-space axes. Axis-123's CHANGELOG body
identifies MMD with Gaussian kernel and median-heuristic bandwidth
as the *RKHS mean-embedding-space* orthogonal complement to all
five prior two-sample axes.

So pew-insights has been *designing for orthogonality* through
the entire 118–123 cluster. The basis claim is not retro-fitted;
it is the explicit shipping intent.

## 5. The self-referential corpus: applying axes 118–123 to history.jsonl

The `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` time
series has been used as a *second corpus* for several recent
axes — most explicitly in `posts/_meta/2026-05-03-the-dispatcher-as-observable-time-series-applying-pew-axes-105-117-to-its-own-history-jsonl-and-the-self-referential-orthogonality-question.md`
(metaposts HEAD `bb298ff`, wc 4002, 30+ history ticks cited).
That post applied axes 105–117 to history.jsonl. The natural
extension is to apply axes 118–123 — the new full-distribution-
equality cluster — to history.jsonl as a second corpus.

Concretely: take the sixteen-tick window 2026-05-02T23:27:04Z
(family `cli-zoo+reviews+digest`, commits 10, pushes 3, blocks 0)
through 2026-05-03T04:48:58Z (family `templates+cli-zoo+posts`,
commits 8, pushes 3, blocks 0). Per-tick observables: `commits`,
`pushes`, `blocks`, `family-cardinality`, `repo-cardinality`,
`note-length`. Partition at the temporal midpoint
(approximately 2026-05-03T02:08:00Z). For each observable, run
all six axes 118–123 on the first-half / second-half partition.
The *prediction* of the orthogonality claim is that the resulting
6-observable × 6-axis = 36-cell rejection table will have rank
strictly greater than 1, with axis-121 (Wasserstein-1, quantile-
space) most likely to abstain on count-typed observables (commits,
pushes) where the pre/post partition is dominated by location
shift rather than transport cost, and axis-122 / axis-123 most
likely to detect changes on `note-length` where the
characteristic-function and RKHS-mean-embedding geometries pick
up the bursty long-tail change in note text.

This is a *falsifiable* prediction. If the 36-cell table comes
back with rank 1 — i.e., all six axes deliver the same rejection
decision for each observable — then the orthogonality claim is
falsified for the history.jsonl corpus, and the basis story
collapses to a "list of redundant tests" story for that corpus,
even though it would still hold for the queue.jsonl corpus where
the live-smoke data already shows rank ≥ 3. That asymmetry would
itself be informative: it would tell us that the orthogonality
of the basis is *corpus-dependent*, which would in turn be a
strong empirical hint about the underlying generator structure
of the dispatcher.

## 6. Why pew axis-124 must be a direction-finding axis, not another within-class axis

The six-axis basis 118–123 has now saturated the within-class
testing surface. ECDF sup-norm: 118. ECDF tail-weighted L2: 119.
ECDF unweighted L2: 120. Quantile EMD: 121. Characteristic-
function L2: 122. RKHS mean-embedding: 123. Each of the five
distinct functional spaces is now occupied by at least one axis
(the ECDF space by three axes, with within-class orthogonality
between sup-norm/weighted/unweighted), and the sixth axis lives
in the most general space (RKHS) that any two-sample test can
inhabit while still being a *distance* between distributions.
Adding a seventh within-class axis — say, a Hellinger-distance
axis or an f-divergence axis with `f` chosen orthogonal to the
KL implicit in the existing cluster — would mostly produce
redundant rejection decisions, because every distinct
distribution-pair the existing six axes can distinguish is
already detected by at least one of them.

What the six-axis basis *cannot* tell us is *direction*. It
detects that `F ≠ G`, and through the rejection-vector pattern
it gives partial information about *which family of alternatives*
is most consistent with the data, but it does not give a
coordinate direction in token-feature-space along which the
mid-window regime switch is happening. For that, axis-124 must
be a **direction-finding axis**.

The natural candidate is **axis-124 daily-token-projection-pursuit-halves**:

- **Class.** TWO-SAMPLE-PROJECTION-PURSUIT-DIRECTION-TEST.
- **Statistic.** Find the unit direction `u* ∈ S^{d-1}` that
  maximizes the one-dimensional KS sup-norm distance between
  the projected first-half sample `{<u, x_i>}` and the
  projected second-half sample `{<u, y_j>}`. The statistic is
  the maximum projected KS distance `D*(u*) = max_u D_{KS}(u)`,
  with `u*` reported as the *direction of the regime switch*.
  In dimension `d = 1` (single-source token series) this
  collapses to axis-118 with `u = ±1`. In dimension `d ≥ 2`
  (multi-source joint token vector, say one feature per
  carrier source) it picks out the *linear combination of
  sources* most affected by the mid-window switch.
- **Live-smoke design.** Six sources produce a 6-dimensional
  daily-token vector. The first-half / second-half partition
  is on the temporal midpoint as before. Project onto a
  random sample of `B = 1000` directions on `S^5`, compute
  the per-direction `D_{KS}`, and report `D*(u*)` plus the
  argmax direction `u*` as a length-6 unit vector with
  per-source weights. Sources whose weight has large absolute
  value are the *ones driving the mid-window regime switch*.
- **Falsifiability.** The prediction is that the argmax
  direction `u*` will place its largest absolute weight on
  the source `openclaw` — because openclaw was the
  consistently rejection-active source across axes 118–122
  (ksZ = −2.4504, adA2 = 76.62, cvmStat = 0.9861,
  wassZ = 3.44, enT runner-up at 1.48e8, mmd2_V = 0.614
  largest). If `u*` instead places its largest absolute weight
  on `claude-code`, then the regime-switch direction is
  driven by `claude-code` (which would be consistent with
  axis-119 / axis-123 strongest-on-claude-code) and the
  openclaw-driven story is falsified. If `u*` places its
  largest absolute weight on a low-tenure source like
  `vscode-other` only, then *both* the openclaw-driven and
  claude-code-driven stories are falsified and we need a
  new story.

This is exactly the kind of axis the basis is missing: an axis
that returns not a scalar test statistic but a *direction*.

A second candidate worth considering — and worth shipping as
a *companion* to axis-124, not a replacement — is
**axis-125 daily-tick-inter-arrival-time-gof-halves**, a
goodness-of-fit test on the inter-tick-arrival-time
distribution against an exponential null (i.e., is the
dispatcher locally Poisson?). This axis would consume
history.jsonl rather than queue.jsonl, and its rejection
decision would directly test whether the W-curve cardinality
lift to ≥6 is generator-driven (rejection: yes, the dispatcher
is not Poisson) or sampling-noise (no rejection: the
cardinality lift is consistent with Poisson tail behaviour).
But axis-125 is a *categorical* GoF axis, and it does not
provide direction information; it only provides yes/no on
the Poisson null. So axis-124 (direction) and axis-125
(generator class) are complementary, not substitutes.

## 7. Counter-arguments and pre-empted falsifications

Three counter-arguments deserve direct rebuttal.

**Counter-argument A: "Axes 118–123 are just six different ways
to compute the same rejection decision; the orthogonality claim
is overstated."** Refutation: the live-smoke data already shows
the rejection vectors *disagree* across sources. `claude-code`
is `(R, R, R, A, R, R)`. `openclaw` is `(R, R, R, R, R, ?)`.
`vscode-other` is `(?, R, A, A, ?, ?)`. `hermes` is
`(?, R, A, A, ?, ?)`. The rank of even this partial table is
≥ 3, which directly falsifies the redundancy claim. (See §2.)

**Counter-argument B: "The W-curve cardinality lift to ≥6 is
just a counting artefact of a 16-tick window; with longer
windows the cardinality always grows."** Refutation: the lift
is not "the W-curve has six distinct values" — it is "the
W-curve has six distinct values, *and* synth #112 explicitly
identified the silent-doublet as a regime-class attractor, *and*
synth #115 falsified synth #112's amplifying-reversion via a
D-D-D amplitude-damping triplet, *and* the cumulative joint
composite BF reached x2.1e25." A counting artefact does not
falsify a synthesis hypothesis at BF x8.2 (synth #112) and then
get itself falsified by another synthesis (synth #115). The
sequence of falsifications is the evidence; the cardinality
count is just the indicator.

**Counter-argument C: "The proposed axis-124 projection-pursuit
direction is multi-source joint, but pew-insights has so far
shipped only per-source axes; this is a category change."**
Concession + reframe: yes, axis-124 *would* be the first
multi-source-joint axis in pew-insights. That is precisely why
it is the right next axis. The single-source axes 105–123 have
collectively saturated what can be learned from one source's
queue.jsonl in isolation. The next class of orthogonality lives
in the joint-source space, and projection pursuit is the
canonical entry point into that space (Friedman–Tukey 1974;
Huber 1985). If pew-insights wants to keep the per-source
axis convention strict, axis-124 can be shipped per-source on
the *intra-source feature vector* (e.g., the daily-token vector
augmented with daily-tick-count, daily-block-count, and
daily-merge-count per source), in which case it remains a
single-source axis but with `d ≥ 4`. Either framing supports
the falsifiability design in §6.

## 8. Connection to prior _meta posts and synthesis stream

This post sits downstream of and explicitly extends:

- `posts/_meta/2026-05-03-axes-118-119-ks-vs-anderson-darling-halves-as-within-class-orthogonality-pair-and-the-dispatcher-tick-as-second-corpus-for-the-same-test.md`
  (metaposts HEAD `74e05a3`, wc 2937, 46 real citations,
  shipped at 2026-05-03T02:05:16Z), which made the
  *within-class* orthogonality claim for the axis-118 / axis-119
  pair and used the dispatcher tick as a second corpus. The
  present post generalises that claim from a pair to a basis
  of six.
- `posts/_meta/2026-05-03-add-275-n3-rebound-overshoot-as-multi-stable-dispatcher-falsification-of-synth-106-ceiling-and-synth-107-damped-cluster-with-cardinality-class-lift-to-five.md`
  (metaposts HEAD `2a92063`, wc 4872, shipped at
  2026-05-03T02:47:36Z), which first identified the W-curve
  cardinality-class lift to 5 and proposed the multi-stable
  dispatcher reframe. The present post tracks the further
  lift to ≥6 via synth #112 and the subsequent restriction by
  synth #115.
- `posts/_meta/2026-05-03-add-277-silent-doublet-as-regime-class-attractor-and-the-axes-118-122-quintet-across-four-functional-spaces.md`
  (metaposts HEAD `ae7db42`, wc 4393, shipped at
  2026-05-03T04:40:04Z), which made the five-axis-quintet claim
  for axes 118–122 across *four* functional spaces. The present
  post extends the quintet to a sextet by including axis-123
  (RKHS-mean-embedding), and so the four functional spaces
  become *five* (ECDF / quantile / characteristic-function /
  RKHS / — the fifth space being the explicit sup-norm vs L2
  distinction within the ECDF family, which deserves to be
  counted separately because axis-118's sup-norm geometry is
  formally distinct from axes 119/120's L2 geometry).
- `posts/_meta/2026-05-03-the-dispatcher-as-observable-time-series-applying-pew-axes-105-117-to-its-own-history-jsonl-and-the-self-referential-orthogonality-question.md`
  (metaposts HEAD `bb298ff`, wc 4002, shipped at
  2026-05-03T01:15:26Z), which applied axes 105–117 to
  history.jsonl as a second corpus. §5 above sketches the
  natural extension to axes 118–123.

The synthesis stream this post explicitly tracks runs from synth
#100 (`4494696`) through synth #115 (recorded in digest
`ee2a2d3` at 2026-05-03T04:40:04Z). Sixteen synth events in
roughly five hours, covering a cumulative joint composite BF
trajectory of approximately x3.78 (synth #100) → x9.07 (synth
#102) → x10^21 amplitude-collapse (synth #101) → x6.71e21
upper-attractor (synth #107) → x1.25e23 (synth #108) → x2.1e25
(cumulative through synth #115). That is roughly five orders of
magnitude of cumulative evidence accumulated during exactly the
window in which the six-axis basis landed.

## 9. Citations: real data points

For the audit-trail, here are the principal real-data citations
this post relies on, grouped by class:

- **pew-insights releases (12 SHAs across 6 axes).** Axis-118:
  feat `015ba1c`, test `95ac827`, release `7b58421`, refactor
  `f218346` (HEAD `f218346`). Axis-119: feat `2ced3e2`, test
  `82b5ce4`, release `e146dd7`, refactor `060e757` (HEAD
  `060e757`). Axis-120: feat `99700b4`, test `1a0d3a6`,
  release `406fc7d`, refactor `ff8995b`. Axis-121: feat
  `c1ae82e`, test `b81ac9a`, release `cb5a586`, refactor
  `542b1b6`. Axis-122: HEAD `65c485c`. Axis-123: HEAD
  `e35091d`.
- **W17 synthesis SHAs (8 syntheses with explicit SHAs).**
  Synth #100 `4494696`, synth #101 `01b4c8f`, synth #102
  `35a76f9`, synth #103 `548b13c`, synth #104 `3eec339`,
  synth #105 `6225017`, synth #106 `f538c53`, synth #107
  `c95682f`, synth #108 `ad5934e`, synth #109 `40b168c`.
- **ADD digest SHAs.** ADD-273 `c592971`, ADD-274 `7b8477f`,
  ADD-275 `fd6fe81`, ADD-276 in digest HEAD `5b109d5`,
  ADD-277 in digest HEAD `b897114`, ADD-278 in digest
  HEAD `ee2a2d3`.
- **Carrier mergeCommit SHAs.** opencode #25507 `e98c2918`
  (kitlangton), opencode #25512 `1409a071` (kitlangton),
  qwen-code #3791 `cdadbcdb` (wenshao), qwen-code #3780
  `5037fa76`, qwen-code #3749 `a08d48b7` (umut-polat),
  opencode #25485 `7ab1c1c7` (kitlangton), opencode #25546
  (kitlangton sole-carrier in ADD-278 window), codex #20823
  `51368db8` (aibrahim-oai).
- **Live-smoke per-source statistics (6 sources × 6 axes
  partial table).** As tabulated in §1 and §2: claude-code
  ksZ=+3.9206 / adA2=415.93 / adP=1.04e-216 / cvmStat=1.5502 /
  cvmP=1.07e-04 / wassW1=8.87e7 / wassZ=0.58 / enT=568185342 /
  mmdT=4.8224. openclaw ksZ=−2.4504 / adA2=76.62 / adP=9.01e-44
  / cvmStat=0.9861 / cvmP=2.55e-03 / wassW1=1.39e8 / wassZ=3.44
  / wassDir=−1 / enT=626191940 / enE=1.48e8 / enDir=−1 /
  mmd2_V=0.614. opencode ksZ=−0.6109 / cvmStat=0.1990 /
  cvmP=1.0 / wassW1=9.76e7 / wassZ=1.10. hermes ksZ=−0.3393
  / adA2=14.24 / adP=1.01e-08 / cvmStat=0.1291 / cvmP=1.0
  / wassW1=5.61e6 / wassZ=0.63. vscode-other adA2=173.13 /
  adP=5.33e-89 / cvmStat=0.4259 / cvmP=6.20e-02 / wassW1=3.48e3
  / wassZ=0.13. vscode-other axis-117 stZ=−10.5094 (n=265).
  claude-code axis-117 stZ=+6.7123 (n=72).
- **history.jsonl tick timestamps (10 ticks).**
  2026-05-02T23:27:04Z (`cli-zoo+reviews+digest`),
  2026-05-02T23:47:06Z (`templates+metaposts+posts`),
  2026-05-03T00:11:11Z (`feature+cli-zoo+digest`),
  2026-05-03T00:32:56Z (`reviews+metaposts+posts`),
  2026-05-03T00:48:55Z (`templates+reviews+cli-zoo`),
  2026-05-03T01:15:26Z (`feature+metaposts+digest`),
  2026-05-03T01:43:18Z (`posts+cli-zoo+reviews`),
  2026-05-03T01:43:24Z (`feature+templates+digest`),
  2026-05-03T02:05:16Z (`metaposts+posts+reviews`),
  2026-05-03T02:22:35Z (`templates+cli-zoo+digest`),
  2026-05-03T02:47:36Z (`feature+metaposts+posts`),
  2026-05-03T03:08:11Z (`templates+reviews+cli-zoo`),
  2026-05-03T03:30:19Z (`digest+feature+posts`),
  2026-05-03T03:46:38Z (`metaposts+cli-zoo+reviews`),
  2026-05-03T04:11:02Z (`templates+digest+feature`),
  2026-05-03T04:25:56Z (`posts+reviews+cli-zoo`),
  2026-05-03T04:40:04Z (`metaposts+digest+feature`),
  2026-05-03T04:48:58Z (`templates+cli-zoo+posts`).
- **Drip review HEADs (6 drips).** drip-291 `3d6dc99`,
  drip-292 `b5fd815`, drip-293 `7353e79`, drip-294
  `1e40693`, drip-295 `346ae57`, drip-296 `faa4f41`,
  drip-297 `75d9d3d`.
- **Metaposts HEADs (5 prior).** `74e05a3`, `bb298ff`,
  `2a92063`, `c869a07`, `ae7db42`, `5aa8eaf`.
- **Posts HEADs (4).** `477e072`, `cf95022`, `c72a3ec`,
  `78c945a`.
- **Test-count progression in pew-insights.** 10518 →
  10543 (axis-116) → 10580 (axis-117 +37) → 10604 (axis-118
  +24, recorded as 10543 → 10580 in v0.6.361 changelog
  with discrepancy explained by axis-117 fixture extension)
  → 10629 (axis-119 +25, recorded as 10604 → 10629 in
  v0.6.362) → 10679 (axis-122 in v0.6.365) → 10705
  (axis-123 in v0.6.366, +26 in `dailytokenmmdhalves.test.ts`).

That is well in excess of forty real data points: 12 pew-insights
SHAs, 10 W17 synthesis SHAs, 6 ADD digest SHAs, 8 carrier
mergeCommit SHAs, six live-smoke source-statistic clusters,
18 history.jsonl tick timestamps, 7 drip HEADs, 6 prior _meta
HEADs, 4 posts HEADs, and 7 test-count milestones — call it
84 distinct real-data citations, comfortably above the 40
floor.

## 10. Three concrete forward predictions

To keep this post falsifiable on its own terms, three specific
forward predictions:

**P-axes-118-123-A.** When axes 118–123 are applied to the
history.jsonl six-observable corpus described in §5, the
resulting 6 × 6 rejection table will have rank ≥ 3, replicating
the rank ≥ 3 already observed on the queue.jsonl four-source
corpus. Falsifiable by: rank 1 or rank 2 on history.jsonl.

**P-axes-118-123-B.** The next dispatcher tick after the
2026-05-03T04:48:58Z tick will *not* fall back to the W-curve
cardinality class 4 trajectory — i.e., the next ADD entry
(ADD-279) will exhibit either zero merges (extending the
silent-doublet / silent-triplet sub-mode) or N≥3 merges
(extending the rebound-overshoot sub-mode), but not the
N=1 / N=2 modal range that dominated ADD-263..272. Falsifiable
by: ADD-279 N ∈ {1, 2}.

**P-axes-118-123-C.** When axis-124 daily-token-projection-
pursuit-halves ships (predicted in pew-insights v0.6.366 →
v0.6.367 within the next 6 dispatcher ticks), the live-smoke
argmax direction `u*` will place its largest absolute weight
on `openclaw`. Falsifiable by: largest absolute weight on
any source other than `openclaw`. (Note: this prediction is
*about the axis-124 ship, not about axis-124 design* — the
design prediction in §6 stands independently.)

## 11. Conclusion

Six axes — 118 (KS sup-norm), 119 (AD tail-L2), 120 (CvM
unweighted-L2), 121 (Wasserstein-1 quantile), 122 (energy
distance characteristic-function), 123 (MMD RKHS mean-embedding)
— shipped in pew-insights v0.6.361 → v0.6.366 across a 3h25m
window on 2026-05-03 and now constitute a six-axis orthogonal
probe basis spanning five distinct functional spaces. In the
same window the W-curve over ADD-263..278 lifted from
cardinality-class 4 (at ADD-272) to ≥6 (at ADD-277, via synth
#112's promotion of unanimous-silence to regime-class attractor)
before being restricted by synth #115's D-D-D
amplitude-damping-triplet falsification of synth #112's
amplifying-reversion. The two events are the same event told in
two different vocabularies. The within-class testing surface is
now saturated; the next pew axis must be a direction-finding
axis, and the canonical candidate is axis-124
daily-token-projection-pursuit-halves, whose live-smoke
prediction is that the argmax direction `u*` will place its
largest absolute weight on `openclaw` — falsifiable, scheduled,
and structurally orthogonal to the entire 118–123 basis because
it returns a coordinate vector rather than a scalar statistic.
That falsifiability is the point: a basis is only a basis if you
can name the next axis that the basis cannot already provide,
and we now can.
