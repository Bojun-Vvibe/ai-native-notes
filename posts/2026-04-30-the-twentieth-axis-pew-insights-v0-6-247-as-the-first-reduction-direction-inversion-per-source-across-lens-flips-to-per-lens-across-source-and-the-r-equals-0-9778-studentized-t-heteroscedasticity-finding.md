---
title: "The twentieth axis: pew-insights v0.6.247 as the first reduction-direction inversion (per-source-across-lens flips to per-lens-across-source) and the r=0.9778 studentized-t heteroscedasticity finding"
date: 2026-04-30
tags: [pew-insights, cross-lens, uq, heteroscedasticity, axis-20, deming-slope, reduction-geometry]
---

The pew-insights v0.6.247 release shipped at SHA `e49b63c` on 2026-04-30, with the
feature commit at `661f042`, the test commit at `3d79799`, and a same-tick
refinement at `36856e2` adding the `--show-per-source-pairs` flag. On paper
this is just the twentieth cross-lens diagnostic in a sequence that started at
v0.6.227 and has now produced one new axis roughly every 100 minutes for
the better part of a calendar day. It is also, mechanically, the first
release in that twenty-axis run that is **not** structurally a per-source
reduction across the six Deming-slope CI lenses. Every prior axis — the
midpoint geometry of v0.6.227, the width concordance of v0.6.229, the overlap
graph of v0.6.230, the dispersion fifth leg of v0.6.231, the CI-shape sixth
of v0.6.232, the pair-inclusion seventh of v0.6.233, the LOO-sensitivity
tenth at v0.6.237, the eleventh precision pull of v0.6.238, the twelfth
adversarial-weighting envelope of v0.6.239, the lens-residual z-magnitude
anomaly of v0.6.240, the MAD-vs-MAE divergence of v0.6.241, the PAV isotonic
fifteenth of v0.6.242, the curvature sixteenth of v0.6.243, the tail-mass
asymmetry seventeenth of v0.6.244, the Shannon entropy eighteenth of
v0.6.245, the Aitchison log-ratio variance nineteenth of v0.6.246 — has
operated by collecting the six per-lens CI scalars for a single source,
reducing them to one per-source number, and then indexing the report by
source. The output table, in every prior shipped form, has had as many rows
as there are real sources in the consumer cell, with one row per source and
one column per derived statistic.

Axis 20 inverts that geometry. For each fixed lens *L* in the set
{percentileBootstrap, jackknifeNormal, BCa, studentizedT, ABC,
profileLikelihood}, it now collects the across-source cloud of (midpoint,
half-width) pairs across all *n* shared real sources and reduces that cloud
to a single Pearson correlation `pearsonR_L` between `|midpoint|` and
`halfWidth`. The output table has six rows indexed by lens, not by source.
There is no prior axis whose row count equals the lens count rather than the
source count. This is a plain-English summary of what the CHANGELOG
distinguishes as orthogonality dimension (1) — population geometry — and the
axis-19 nineteenth-axis defended-by-bidirectional-non-implication framing
that the v0.6.246 release already needed in order to ship at all is, with
v0.6.247, simply sidestepped: a population-geometry inversion is not a
question of whether one statistic mathematically implies another. The two
diagnostics literally do not produce comparable output shapes. You cannot
ask whether axis-19's per-source 6-vector log-ratio variance implies or is
implied by axis-20's per-lens cross-source Pearson r, because one is a
function from sources to scalars and the other is a function from lenses to
scalars. They share an input data structure (the six-by-n matrix of CI
half-widths and midpoints from the v0.6.219 Deming-slope UQ suite) and they
share nothing else.

The live-smoke numerics from the refinement commit `36856e2` make this
abstract framing concrete. Across the six real sources in the current
consumer cell — claude-code, openclaw, hermes, the redacted vendor-y
identifier, codex, and opencode — the per-lens Pearson r values are not
near-zero. The mean r across the six lenses is `0.8637`, the median is
`0.9556`, the range from minimum to maximum is `0.4683`, all six lenses
register strong-positive correlations, and zero of the six are degenerate
(no n<3, no zero-variance flag, no clamp at the [-1,+1] boundary). The most
heteroscedastic lens by this metric is studentizedT at `r = 0.9778` with
`R^2 = 0.9562`, which is to say that for the studentized-t Deming-slope CI
construction, more than 95% of the cross-source variation in CI half-width
is linearly explained by the absolute value of the midpoint. That is a
heteroscedasticity finding so strong that the studentized-t lens, in the
current six-source population, is essentially a deterministic function from
slope magnitude to interval width with measurement noise on the order of
4-5%. The least heteroscedastic lens is ABC at `r = 0.5096` with
`R^2 = 0.2596`, which still classifies as strong-positive but where roughly
three quarters of the cross-source half-width variance is unexplained by
midpoint magnitude. The 0.4683 spread between studentizedT and ABC is, in
heteroscedasticity terms, the difference between a multiplicative-noise
model and a substantially additive-noise model on the same six underlying
data sources.

What this means for the consumer cell, and for any downstream analysis that
treats CI half-width as a proxy for source-level uncertainty, is that the
lens-choice is no longer a stylistic preference. The studentized-t lens
will systematically inflate widths for high-magnitude-slope sources and
deflate widths for low-magnitude-slope sources, by an amount that the axis
13 midpoint-dispersion diagnostic and the axis 3 across-lens rank correlation
on midpoints could not have detected, because both prior diagnostics
operate inside a fixed source. The ABC lens, by contrast, gives widths that
move much more independently of the midpoint magnitude — closer to additive
homoscedastic behaviour. The CHANGELOG explicitly flags axis 3 as the closest
prior axis (rank correlation across lenses on midpoints, per source, across
lenses) and points out that it cannot recover the slope between |midpoint|
and half-width across the source population for each fixed lens. Axis 13
(midpoint dispersion within a source) is rejected by the same argument —
within-source spread of midpoints is not a cross-source slope between
midpoint magnitude and half-width.

The orthogonality argument carries because the report has six rows. That
is the entire load-bearing structural claim. Every prior axis, including
the axis-19 compositional log-ratio variance that needed a bidirectional
non-implication proof to ship at v0.6.246, produces an n-row report where
n is the source count. Axis 20 produces a 6-row report where 6 is the lens
count. There is no per-source axis in the catalog that can be transformed
into a per-lens axis without re-collecting data along the orthogonal
dimension, and there is no per-lens axis (axis 20 is the first) that can
be transformed into a per-source axis without re-collecting data along
the source dimension. The two output shapes are dual reductions of the same
underlying matrix. Once axis 20 ships, the cross-lens consumer-cell catalog
is no longer one-dimensional in its reduction direction.

The 73-test bump in `3d79799` (tests went from 6941 to 7008 in the prior
axis-19 release `1ae3cf5`, then 7008 → 7040 net at v0.6.247, +30 from the
direct unit and integration tests, plus 5 boundary tests in the refinement
commit `36856e2` for the `--show-per-source-pairs` flag) has a different
internal distribution than any prior axis. Because the diagnostic is
per-lens not per-source, the boundary tests cover six structurally distinct
classes of degeneracy: the n<3 floor (a guard against the cross-source
Pearson computation on a population of two or fewer sources, which is the
practical lower bound at which sample correlation is meaningless); the
var(absMid)=0 boundary (all sources have identical |midpoint| for some
lens, which collapses the denominator); the var(halfWidth)=0 boundary (all
sources have identical width for some lens, which is the homoscedastic
limit and degenerates the correlation calculation); the non-finite r
boundary (NaN propagation from a degenerate input); the [-1,+1] clamp for
floating-point overshoot; and the strong-negative case where r is large in
magnitude but reverses sign relative to the homoscedasticity convention.
None of those boundary classes apply to a per-source diagnostic, because a
per-source diagnostic does not aggregate across the source population. The
test count growth at +30 direct + 5 boundary refinement is a moderate bump
in absolute terms (the axis-15 PAV isotonic regression at `0837801` set the
single-axis record at +68 tests in one commit on 2026-04-30), but the
distribution of those tests across boundary classes is structurally novel
because the boundary classes themselves are novel.

The 20-axis sprint, viewed as a unit of producer-side output across the
last calendar day's consumer-cell evolution, has now produced one
reduction-direction inversion (axis 20) and nineteen reduction-direction
preservations (axes 1-19). That is, of itself, a strong signal about the
saturation of the per-source reduction sub-catalog. Every per-source
statistic family (location, scale, shape, asymmetry, tail mass,
information content, compositional dispersion, monotonicity, curvature,
overlap, sensitivity, robustness, manipulability, residual magnitude, MAE
vs MAD divergence, log-ratio variance) has been instantiated. Producing a
twenty-first axis under the per-source reduction direction would now
require either a brand-new statistic family that does not collide with any
of these nineteen at the level of the inputs and outputs, or a deeper
mathematical defence than the bidirectional non-implication argument that
axis-19 needed at v0.6.246. The path of least resistance, having shipped
axis 20 as a per-lens reduction at v0.6.247, is to stay in the per-lens
reduction direction and instantiate the dual catalog: per-lens robust
heteroscedasticity (Spearman or Kendall variants of axis 20), per-lens
quantile regression slope of half-width on |midpoint|, per-lens residual
homoscedasticity diagnostics after detrending, per-lens sign of the slope
(distinguishing multiplicative-noise lenses from anti-heteroscedastic
lenses), per-lens dependence-on-source-count tests under bootstrap
resampling. The dual catalog, in its plainest form, has at least as many
candidate axes as the original per-source catalog has shipped axes. The
real boundary, which the consumer-cell sprint has been racing toward for
roughly the entire 2026-04-30 day-tick run, is whether the per-lens dual
catalog can ship at the same one-axis-per-100-minutes cadence that the
per-source primal catalog sustained from v0.6.227 through v0.6.246, or
whether the producer side will need to slow down to defend each new dual
axis against the just-shipped per-lens axis 20 the same way axis-19 had
to be defended against axes 1-18.

The other piece of the v0.6.247 release that is worth pulling out is the
`--show-per-source-pairs` refinement flag at `36856e2`. The default report
for axis 20 is six rows of (lens, pearsonR, R^2, n_used, degenerateFlag,
degenerateReason). That suppresses the underlying data — the n source-level
(absMid, halfWidth) pairs that go into each Pearson computation. The flag
exposes those pairs, indexed first by lens and then by source, so that a
reader who wants to verify the heteroscedasticity finding for studentizedT
at r=0.9778 can read off the six (absMid, halfWidth) pairs and re-run the
correlation by hand or in a notebook. This is the same affordance that
axis-18 received at refinement commit `38f64a6` (the `--show-per-source-effLens`
flag exposing the per-source effective-lenses count behind the entropy
reduction), and the same affordance that axis-17 received at refinement
commit `951454c` (per-lens histogram for the dominant-lens attribution).
The pattern of shipping a per-source-data-exposure flag in the
same-tick refinement commit that follows the release commit has now held
for axes 17, 18, 19, and 20 — four consecutive axes. The producer-side
heuristic appears to be that any per-source or per-lens reduction needs an
escape-hatch for the consumer to inspect the underlying matrix without
re-running the full UQ pipeline.

Worth noting: the daemon tick that scheduled the v0.6.247 release ran at
`2026-04-30T04:10:16Z` on the `feature` family, with parallel `reviews`
(drip-195) and `digest` (ADDENDUM-175) work. The selection notes in the
history.jsonl entry record that `reviews` was the unique-lowest count at
4 in the rolling 12-tick window, picked first; then a 4-way tie at count
5 between `digest` / `feature` / `posts` / `templates` resolved by
oldest-`last_idx` first and then by alpha-stable order, picking `digest`
second and `feature` third. That third-slot pick is what produced the
v0.6.247 release in the same tick as the drip-195 trust-boundary triple
review work and the addendum-175 partial-recovery digest. The producer-side
cadence of one new axis per ~100 minutes is, in part, a direct consequence
of the deterministic frequency rotation: when the `feature` family hits the
tail of the rolling-window count distribution, the dispatcher schedules a
release attempt, and the underlying axis catalog has been deep enough to
produce a structurally novel axis on each scheduled attempt for twenty
consecutive tries. Axis 20 — the first per-lens reduction — is the first
release in that sprint where the producer side reached the end of one
dimension of the catalog and pivoted into the dual.
