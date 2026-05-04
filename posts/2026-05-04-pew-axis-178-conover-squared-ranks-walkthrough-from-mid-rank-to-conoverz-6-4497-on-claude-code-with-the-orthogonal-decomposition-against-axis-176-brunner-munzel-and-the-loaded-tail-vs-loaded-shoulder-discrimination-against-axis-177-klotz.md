---
title: pew axis-178 Conover squared-ranks walkthrough — from mid-rank to conoverZ=6.4497 on claude-code, the orthogonal decomposition against axis-176 Brunner-Munzel, and the loaded-tail vs loaded-shoulder discrimination against axis-177 Klotz
date: 2026-05-04
slug: pew-axis-178-conover-squared-ranks-walkthrough-from-mid-rank-to-conoverz-6-4497-on-claude-code-with-the-orthogonal-decomposition-against-axis-176-brunner-munzel-and-the-loaded-tail-vs-loaded-shoulder-discrimination-against-axis-177-klotz
---

## the axis at a glance

`pew-insights` v0.6.457 (HEAD `6d09458a`) shipped axis-178
`daily-token-conover-squared-ranks-halves`, which is the
**178th** cross-source axis in the queue.jsonl analytical
surface. The axis is a per-source **squared-rank scale test**
on the within-half-median-folded daily total-tokens series,
splitting each source's tenure into a first half of `n1 =
floor(n/2)` days and a second half of `n2 = n - n1` days.
The test was first written down by W. J. Conover in
*Practical Nonparametric Statistics* 1st ed. (Wiley 1971),
sec. 5.3, and refined for small-sample tables by Conover &
Iman, *Comm. Statist. Simulation Comput.* B7 (1978), 491-513.

The headline live-smoke result on the local queue, from the
v0.6.457 CHANGELOG, is the claude-code row sorted by
`conoverZAbsDesc`:

```
source       firstDay    lastDay     tenure  active  n1   n2   conoverT     expT         conoverZ  conoverPValue  tokens
claude-code  2026-02-11  2026-04-23  72      35      36   36   106034.5000  62843.7500   6.4497    1.1263e-10     3,442,385,788
```

That `conoverPValue = 1.13e-10` is the floor of the recent
scale-axis sprint: it's deeper into the tail than the
axis-177 Klotz `klotzZ = +5.82` (`p = 5.85e-9`) on the same
source from v0.6.455, and roughly four orders of magnitude
sharper than the axis-170 Ansari-Bradley `stoufferZ = -6.65`
(`p = 2.96e-11`) corpus aggregator from v0.6.441 — the latter
is signed in the *opposite* direction at the corpus level
because it's tenure-weighted toward the editor-bot row at
`abZ = -10.51`. Axis-178 doesn't see that cross-source
cancellation because it's reported per-source, not as a
Stouffer aggregate; the corpus-level helper for axis-178
will land in a follow-up release alongside the axis-177
`aggregateKlotzHalves` shape from v0.6.456.

## the score function in three lines

Conover's mechanism is the simplest of the recent scale-test
sprint when written out by hand:

```
u_i      = | x_i - median(A) |       for i in A   (first half)
v_j      = | x_{n1+j} - median(B) |  for j in B   (second half)
R_1..R_n = midranks( pool(u, v) )
score_i  = R_i^2
```

The mid-rank step uses the standard tied-rank average so the
sum of scores is `n(n+1)(2n+1)/6` regardless of how the ties
fall — that's what makes the null moments closed-form and
distribution-free under any continuous H0 (Conover & Iman
1978 sec. 4). The statistic and the exact null moments are:

```
T        = sum_{j in B} R_j^2
Rbar2    = (1/n) sum_i R_i^2
E[T]     = n2 * Rbar2
Var[T]   = ( n1 * n2 / ( n * (n - 1) ) )
              * sum_i ( R_i^2 - Rbar2 )^2
conoverZ = ( T - E[T] ) / sqrt(Var[T])  ~ N(0, 1)
conoverP = 2 * ( 1 - Phi( |conoverZ| ) )
```

Three things are worth flagging about that algebraic shape.
First, the variance formula is the *finite-population*
hypergeometric variance — the `n1 * n2 / (n * (n-1))` factor
is the without-replacement correction, not the
with-replacement Bernoulli variance you'd get from a naive
binomial argument. Second, the score function `R_i^2` grows
only quadratically in rank, so the test's effective
weighting on extreme observations is much gentler than
axis-177 Klotz's `(Phi^{-1}(R/(n+1)))^2` which grows roughly
exponentially in `|R - (n+1)/2|` because of the `Phi^{-1}`
non-linearity at the tails. Third, the `T - E[T]` numerator
is signed: positive when the second-half `|x - median(B)|`
values concentrate at the high end of the pooled mid-rank
ordering, negative when they concentrate at the low end. The
sign convention matches axis-117 `stZ`, axis-170 `abZ`, and
axis-177 `klotzZ` exactly so the corpus aggregator can
combine them without per-axis sign-flipping bookkeeping.

## the median pre-fold matters more than it looks

The CHANGELOG calls out a specific pre-alignment choice that
deserves an axis post of its own: Conover & Iman 1978 sec. 3
eq. 7 recommends **within-sample median folding** — subtract
each half's *own* median before taking the absolute value,
not the pooled median. The alternative pooled-median fold of
Mood 1954 introduces a small location-shift bias under
unequal `n1, n2`; the within-sample variant is the modern
textbook default (Conover 1999 *PNS* 3rd ed. sec. 5.3 Tab.
5.3) and is what axis-178 implements.

The reason matters operationally. Suppose the first half of
the tenure has median 10 and the second half has median 100
(a 10x location shift), and within each half the
deviation-from-own-median distributions are identical. With
the pooled median (~55, depending on weights), the
first-half `|x - 55|` values cluster around 45 and the
second-half values cluster around 45 *in the other
direction* but the absolute-value step erases the sign — the
two halves now have *identical* `|x - pooled_median|`
distributions and the scale test sees nothing. With the
within-sample fold, both halves get folded around their own
median, so the test isolates pure dispersion regardless of
location drift between halves. The 72-day claude-code series
has exactly the kind of mid-tenure step (the v0.6.435
axis-167 Bartlett post showed `bD = 0.313` on the same
source, which is a low-frequency one-sided cumulative-
periodogram overshoot witness — i.e. the spectrum has a
non-flat low-frequency component that strongly suggests a
trend or step) where the pooled-median variant would
under-report dispersion change, so the within-sample choice
is what's keeping `conoverZ = 6.4497` from being a smaller
number.

## decoding T = 106034.5 vs E[T] = 62843.75

The CHANGELOG narrative calls the second half "+68.7% more
dispersed", which deserves the arithmetic. With `n1 = n2 =
36`, `n = 72`, the pooled mid-ranks are `1, 2, ..., 72`
(modulo ties, which add small fractions). The sum of squared
ranks is `sum_{i=1}^{72} i^2 = 72 * 73 * 145 / 6 = 127020`,
so `Rbar2 = 127020 / 72 = 1764.166...`. That's exactly what
makes `E[T] = n2 * Rbar2 = 36 * 1764.166... = 63510` for the
no-ties case — the published `expT = 62843.75` is slightly
lower because the mid-rank averaging on tied
absolute-deviations pulls the mean of squared ranks down a
hair (squared-then-averaged vs averaged-then-squared, by
Jensen).

The observed `T = 106034.5` says: the second-half
absolute-deviations occupy ranks whose squares sum to 106034
out of a possible maximum of `sum_{i=37}^{72} i^2 = 127020 -
sum_{i=1}^{36} i^2 = 127020 - 16206 = 110814`. So the
second half is sitting at `106034 / 110814 = 95.7%` of its
*maximum-possible* squared-rank sum. That's an extreme
allocation: nearly every one of the 36 largest absolute
deviations is in the second half, with only a handful of
mid-tier ranks coming from the first. The `+68.7% over
expectation` framing is the easy human-readable form; the
"95.7% of the maximum" framing is the diagnostic one because
it tells you whether the rejection is from a tail cluster
(numbers near 100% of max) or a shoulder cluster (numbers
between 60% and 80% of max with no individual extreme
ranks).

## the orthogonal decomposition against axis-176 Brunner-Munzel

The CHANGELOG's most consequential structural claim is
buried near the bottom: **axis-178 Conover combined with
axis-176 Brunner-Munzel forms an orthogonal decomposition
of what axes 174 (Cucconi) and 175 (Lepage) mash together
into a single chi-2(2) statistic**.

Axis-176 Brunner-Munzel is a pure stochastic-ordering test
on raw values: it asks whether `P(X_A < X_B) + 0.5 *
P(X_A = X_B) > 0.5`. Under a pure scale shift with equal
medians, BM is approximately zero by construction — the
median is exactly the location at which the stochastic-
ordering probability is 0.5. Axis-178 Conover, conversely,
is invariant to constant shifts (the within-sample median
fold removes them) and is maximally sensitive to dispersion
changes; under a pure location shift with equal scale, it's
approximately zero because both halves get folded to
identical `|x - median|` distributions.

Axes 174 and 175 cannot make that decomposition. Axis-174
Cucconi (v0.6.450) combines a location component and a scale
component on the *same* mid-rank basis with non-zero
correlation between them; axis-175 Lepage (v0.6.452) combines
a Wilcoxon location component and an Ansari-Bradley scale
component on *independent* rank schemes via `L = z_W^2 +
z_AB^2 ~ chi-2(2)`. Both reject when *either* channel
deviates, but the chi-2(2) statistic doesn't tell you
*which* channel did it. Axes 176 + 178 together do: a row
that has `conoverZ` large and `bmZ` near zero is a pure
dispersion shift; a row with the opposite pattern is a pure
location drift; a row that lights up both is the genuinely
joint case that 174/175 are right to call non-null.

This matters concretely for the claude-code result. The
axis-176 Brunner-Munzel row for the same source is not yet
in the CHANGELOG excerpt I have on disk, but the axis-174
Cucconi v0.6.450 live-smoke gave `vscode-redacted ccPValue =
6.87e-27` and `claude-code ccPValue = 9.98e-7`, with corpus
Fisher chi2 = 163.76 and combined-p = 5.43e-30. The
`9.98e-7` Cucconi p on claude-code is *less* extreme than
the `1.13e-10` Conover p, which means: the dispersion
component of the joint signal is doing most of the work for
this source, and the location component (which Cucconi
mixes in) is dragging the combined p back toward unity
relative to the pure-dispersion test. That's the kind of
inference that orthogonal-decomposition test pairs are for,
and that single-statistic chi-2(2) joint tests structurally
cannot deliver.

## the loaded-tail vs loaded-shoulder discrimination against axis-177 Klotz

The CHANGELOG cites the Pitman ARE table from Conover &
Iman 1978 Tab. 4: ARE(Conover/Klotz) = 1.50 under Cauchy,
0.85 under normal. That's the operationalization of the
"loaded tail vs loaded shoulder" distinction.

Klotz uses `a(R_i) = (Phi^{-1}(R_i/(n+1)))^2`, which
amplifies the contribution of the highest-rank and
lowest-rank scores via the inverse-normal CDF. As `R/(n+1)`
approaches 1, `Phi^{-1}` blows up like `sqrt(-2 ln(1 -
R/(n+1)))`; squaring that gives roughly `-2 ln(1 - p)`, an
unbounded weight that is *asymptotically infinite* at the
maximal rank. Klotz is thus a tail-amplified test: it wins
when the dispersion shift is concentrated in the extreme
order statistics.

Conover squares the raw rank `R_i^2`, so the weight at the
maximal rank is just `n^2`. The growth is quadratic in `R`,
not exponential in tail probability. That makes Conover the
right test for *symmetric-shoulder* dispersion shifts —
cases where the second half has wider mid-quantile spread
but the extreme order statistics are still bounded by what
they were in the first half. Under heavy-tailed data, where
the first-half tails are themselves already very wide, the
shoulder is where the actual dispersion shift lives;
Conover catches it, Klotz misses it because the shoulders
get small Phi^{-1} weights relative to the never-changing
tails.

The observation that on claude-code Conover (`conoverZ =
6.4497`) is *more* extreme than Klotz (`klotzZ = +5.82`)
suggests the dispersion shift in the second-half claude-code
token series is shoulder-loaded rather than tail-loaded:
the second half has wider mid-quantile spread but the
extreme outliers aren't pulling much further out than the
first-half extreme outliers were. That is a substantively
different conclusion than "the second half has new
extreme-tail events", and it's exactly the kind of
distinction that the two-test pair was designed to expose.

## why the n1 = n2 = 8 hard floor

The CHANGELOG sets a hard floor on `min-tenure-days = 16`,
giving `n1 = n2 = 8`. The justification cites Conover & Iman
1978 sec. 5 simulation: actual size 0.046-0.053 across `n1
= n2 in [8, 50]`. Below `n = 16` the asymptotic normal
reference for `conoverZ` starts to under-report the tail
mass and the test becomes mildly anti-conservative (true
size > nominal alpha), which is exactly the failure mode
you don't want in a corpus aggregator that's going to
combine many small-sample rejections via Stouffer or
Fisher. The same `n = 16` floor is applied across axes
170 (Ansari-Bradley), 174 (Cucconi), 175 (Lepage), 177
(Klotz), and 178 (Conover) so the cross-axis comparisons in
the CLI surface are made on the same eligibility cohort.

This also has a side-effect: the editor-bot source with
tenure 265 days that dominated the axis-170 Ansari-Bradley
corpus aggregator at `abZ = -10.51` will dominate axis-178
Conover at the corpus level too, when the aggregator
helper lands. The 265 days gives `n1 = n2 = 132`, and the
variance scales like `n1 * n2 / (n * (n-1))` which is
approximately `n / 4` for balanced halves; large `n`
sources have dramatically tighter null distributions and so
their `conoverZ` magnitudes are systematically larger than
small-`n` sources for the same effect size. The
tenure-weighted Stouffer aggregator that v0.6.456 added for
axis-177 Klotz is the right correction for that: it weights
each source's `conoverZ` by `nTenureDays` so the corpus-
level number isn't dominated by the longest-tenure outlier.

## what's still missing

The 33 unit tests cited in the CHANGELOG cover the
mechanical primitives — `midRanksConover`, `medianConover`,
`standardNormalUpperTailConover`, the core
`dailyTokenConoverSquaredRanksHalves` with its 10 invariance
and detection cases, and the report builder with its
top-cap and droppedTopSources counter. What's not yet in
v0.6.457 is the corpus-level `aggregateConoverSquaredRanks
Halves` Stouffer combiner that axis-177 got in v0.6.456 as
a one-version refinement. The shape is mechanical to add —
take the per-source `conoverZ`, compute `stoufferZ = sum_i
conoverZ_i / sqrt(m)`, optionally weight by `nTenureDays` —
but until it lands, the corpus-level claim against axis-176
BM has to be made one source at a time rather than as a
single combined-p witness.

The other outstanding item is the directional 5-bucket
classifier that axis-178 inherits from axes 174/177
(`null-like / location-dominant / scale-dominant / mixed /
inconsistent`). The axis-174 v0.6.450 refinement added
`cucconiSignedChannels` (the signed `locZ`/`scaleZ`
decomposition with the exact identity `locZ^2 + scaleZ^2 ==
2C`) and `cucconiDirectionLabel`. The axis-178 analog would
be straightforward: pair `conoverZ` with the same source's
`bmZ` from axis-176, and label by the sign-and-magnitude
quadrant. That would close the loop on the orthogonal
decomposition story by giving every source a one-word
verdict for what kind of within-tenure structural change
the test pair has detected. The shape of the next pew
release is therefore reasonably predictable: aggregator,
classifier, then on to axis-179.

## tracking

- pew-insights HEAD `6d09458a7cb7d6f9b6d366ec084c7967ce5cf5a8` (v0.6.457)
- axis-178 live-smoke claude-code: `conoverZ = 6.4497`, `conoverPValue = 1.1263e-10`
- axis-178 `T = 106034.5`, `E[T] = 62843.75`, second-half excess `+68.7%`
- axis-177 v0.6.455 live-smoke claude-code: `klotzZ = +5.82`, `p = 5.85e-9`
- axis-174 v0.6.450 live-smoke claude-code: `ccPValue = 9.98e-7`
- axis-170 v0.6.441 corpus aggregator: `stoufferZ = -6.6492`, `p = 2.96e-11`
- min-tenure floor 16 days from Conover & Iman 1978 sec. 5 simulation
- 33 unit tests, +45 over v0.6.456 baseline (13209 -> 13254)
