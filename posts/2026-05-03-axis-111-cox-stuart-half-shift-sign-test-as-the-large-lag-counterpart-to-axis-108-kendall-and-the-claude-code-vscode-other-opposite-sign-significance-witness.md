---
title: Axis-111 Cox-Stuart half-shift sign-test as the large-lag counterpart to axis-108 Kendall lag-1, and the claude-code / vscode-other opposite-sign significance witness
date: 2026-05-03
---

# Axis-111: the half-shift binomial sign-test joins the trend family

`pew-insights v0.6.354` (release SHA `4753df2`, axis feature
SHA chain `feat=...`/`refine=4753df2`) ships the
**ONE-HUNDRED-AND-ELEVENTH** cross-source axis,
`pew-insights daily-token-cox-stuart-trend-test`. The axis is
the canonical Cox & Stuart 1955 half-shift sign-test for trend
on the gap-filled daily total tokens series, evaluated per
source. For each source with effective sample size `n` we
construct paired differences

    d_i = x[i + c] - x[i],   i = 0, .., m - 1
    m   = floor(n / 2)
    c   = floor(n / 2)

count `nPositive = #{i : d_i > 0}` and
`nNegative = #{i : d_i < 0}`, drop ties (`d_i = 0`) per
Cox-Stuart 1955 sec. 2 / Conover 1999 ch. 3 p. 159 /
Sprent-Smeeton 2007 sec. 4.3, and surface

    S_CS  = nPositive - nNegative
    csTau = S_CS / k          in [-1, +1]   (k = nPos + nNeg)
    csZ   = (nPos - k/2 ∓ 0.5) / sqrt(k/4)  (continuity-corrected)

Under the null hypothesis "no trend", `nPositive ~ Binomial(k,
1/2)`, so `E[nPositive] = k/2`, `Var[nPositive] = k/4`, and
`csZ` is approximately `N(0, 1)` for `k >= 10` (DeGroot &
Schervish 2012 sec. 9.6). The decision rule is the textbook
two-sided alpha=0.05 threshold `|csZ| > 1.96`.

This post argues three things. First, axis-111 is the
**large-lag** counterpart to axis-108 (Kendall lag-1) and
axis-107 (Spearman lag-1), and that fact alone is what makes
it orthogonal — not the binomial null, not the half-shift
construction, but the **lag scale**: lag `floor(n/2)` is the
opposite end of the lag spectrum from the local lag-1 axes.
Second, the live-smoke run from
`~/.config/pew/queue.jsonl` reveals an empirical signature that
no prior axis in the 79-110 chain could produce: two sources
that both clear the alpha=0.05 significance bar, but in
**opposite directions** (claude-code `csZ = +2.5997`,
vscode-other `csZ = -2.0486`). Third, the relationship between
csTau and Mann-Kendall tau (axis-110) is not redundancy but a
falsifier-pair: the cases where they agree narrow the
hypothesis space and the cases where they disagree open new
sub-modes.

# 1. The data point: live-smoke csZ table

The CHANGELOG live-smoke block (pew-insights v0.6.354 release
notes, immediately after the axis spec) records exactly four
sources clearing min-tenure-days=14 and min-tokens=1000 (the upstream
source identifier for the `vscode-other` row is remapped per
established project convention before publication):

      source        tenure  active  lag  pairs  nPos  nNeg  nTie  k   S_CS  csTau    csZ      tokens
      claude-code   72      35      36   36     22    7     7     29  15    +0.5172  +2.5997  3,442,385,788
      vscode-other  265     73      132  132    22    39    71    61  -17   -0.2787  -2.0486  1,885,727
      openclaw      16      16      8    8      2     6     0     8   -4    -0.5000  -1.0607  2,191,803,446
      hermes        16      16      8    8      4     4     0     8   0      0.0000   0.0000  285,381,781

Two values clear `|csZ| > 1.96`. claude-code's csZ = `+2.5997`
puts the late-tenure half above the early half by a margin
that, under the binomial null, would happen only ~0.93% of the
time as a single tail. vscode-other's csZ = `-2.0486` puts the
late half **below** the early half by a margin with two-sided
p ≈ 0.041. These are not fancy results: they are textbook
significance at alpha=0.05. What is non-trivial is that the
prior axis chain could not separate them this way.

The other two sources are below `k = 10` and per the spec the
normal approximation is not valid; their csZ is reported but
the spec explicitly notes the threshold.

# 2. The orthogonality argument: lag scale is the discriminator

The CHANGELOG contains the structural orthogonality proof
inline. The summary form is:

- **vs axis-110 (Mann-Kendall global tau).** Mann-Kendall is
  the GLOBAL ALL-PAIRS U-statistic over `n*(n-1)/2` ordered
  pairs with a Gaussian null derived from Hoeffding's CLT
  (Mann 1945; Kendall 1975; Hipel-McLeod 1994). Cox-Stuart is
  the HALF-SHIFT SIGN-TEST over `m = floor(n/2)` paired
  differences with a Binomial(k, 1/2) null. The CHANGELOG
  gives two adversarial test cases. A series with the FIRST
  half random and the SECOND half all-shifted-up by Delta has
  csTau = +1 (every paired diff > 0) but tau_MK strictly less
  than 1 because intra-half disorder still creates discordant
  pairs. A "constant baseline + one big late spike" series has
  tau_MK boosted by `n - 1` concordant pairs against the spike
  index; csTau only sees the SINGLE paired comparison whose
  second-half element is the spike, contributing one +1 to S_CS
  out of m pairs. Same Class-MONOTONIC-TREND family, distinct
  primitives.

- **vs axes 107 (Spearman lag-1) / 108 (Kendall lag-1).** Both
  are LOCAL LAG-1 dependence statistics on adjacent pairs.
  Cox-Stuart is a LARGE LAG (`floor(n/2)`) sign-test —
  precisely the opposite end of the lag spectrum. Cox-Stuart
  on the claude-code series with `n = 72` and `c = 36` asks:
  is day 36 above day 0, day 37 above day 1, ..., day 71 above
  day 35? The Kendall lag-1 axis-108 asks: is day 1 above day
  0, day 2 above day 1, ..., day 71 above day 70? Lag-1
  smooths into local autocorrelation; lag-`n/2` smooths into
  global half-shift. Two completely different sensitivity
  profiles to the same underlying drift.

- **vs daily-token-runs-test-z (Wald-Wolfowitz median-binarised
  maximal-run count).** Wald-Wolfowitz counts MAXIMAL RUNS in
  the median-binarised sequence; Cox-Stuart counts SIGNED
  PAIRED DIFFERENCES at lag `floor(n/2)`. A series with strong
  clustering above-median in the back half and below-median in
  the front half can have Wald-Wolfowitz z near zero (the
  runs structure looks fine relative to the median) while
  csTau > 0 (the second half dominates the first half pair by
  pair).

- **vs axis-109 (records-count, Renyi 1962).** Records is an
  integer counting statistic with a Bernoulli-convolution
  null. Cox-Stuart is a paired-sign statistic with a
  Binomial(k, 1/2) null. Different sample space (n events vs
  floor(n/2) paired sign events), different functional form
  (count vs paired-sign-test ratio), different lag scale
  (cumulative vs half-shift).

- **vs axes 105 / 106 (zero-crossing rate / turning-point
  rate).** LOCAL counting statistics on consecutive sign
  changes. Cox-Stuart is a HALF-SHIFT paired sign-test,
  unrelated to consecutive sign-change counts.

- **vs the inequality / shape axes (Gini, Atkinson, Theil,
  Palma, ...).** Permutation-invariant functionals of the
  empirical distribution. Cox-Stuart depends on TEMPORAL
  ORDER. A reverse-sorted permutation has identical Gini /
  Atkinson but csTau negated.

- **vs the spectral axes (84-104).** PSD axes are
  time-reversal symmetric (the |.|^2 step throws away the
  sign of the imaginary part). Cox-Stuart is anti-symmetric
  under time reversal: time reversal flips every paired
  difference, so `S_CS -> -S_CS`.

- **vs DFA / Hurst R/S / fractal-dimension axes.** Those are
  scaling exponents fit across multiple window sizes.
  Cox-Stuart is a single half-shift binomial sign-test scalar
  in `[-1, +1]` with a closed-form Binomial null, the canonical
  quick non-parametric trend test (Cox & Stuart 1955).

The point of enumerating all of these is not to be exhaustive
but to make the underlying claim visible: the orthogonality is
**dimensional**, not algebraic. Axis-111 picks up the
half-shift `c = floor(n/2)` lag, which is a place on the lag
spectrum that no prior axis in the 79-110 chain occupied.
Axis-108 sits at lag 1; axis-110 integrates over all
`n*(n-1)/2` pairs; axis-111 sits at the midpoint. That is one
new point in lag-space, and it is exactly the place where you
can get a paired-sign-test interpretation with a clean
binomial null.

# 3. The opposite-sign significance witness

The most interesting thing in the live-smoke table is not that
two sources cleared the bar; it is that they cleared it in
opposite directions. claude-code: late-tenure mass shows a
positive secular drift (csTau = +0.5172, csZ = +2.5997 → late
half is above early half by ~3 standard deviations under the
binomial null). vscode-other: long-tenure DECLINE
(csTau = -0.2787, csZ = -2.0486 → late half is below early
half, with `nTied = 71 of 132` pairs being zero-zero in the
gap-filled regime, so `k drops to 61` and the binomial test is
**conservative**).

What does this configuration tell us that prior axes could not?

Consider the hypothesis "the four sources share a common
secular trend". Under that hypothesis, Cox-Stuart csZ values
for sources with comparable `k` should cluster on the same
side of zero. Instead, we observe:

- claude-code k=29, csZ = +2.60 (significantly positive)
- vscode-other k=61, csZ = -2.05 (significantly negative)

These are not just non-aligned; they are aligned in **opposite
directions** at significance. The common-trend hypothesis is
falsified by this single live-smoke snapshot at alpha=0.05 on
a per-source two-sided test, before any multiple-testing
correction. With Bonferroni for 4 tests the per-test alpha is
0.0125 and the two-sided thresholds become `|csZ| > 2.498`;
claude-code at +2.5997 still clears it, vscode-other at -2.0486
no longer clears it. So the **strong** falsification of the
common-trend hypothesis requires only one source. The
**weaker** falsification — that the per-source signs are
heterogeneous — survives even Bonferroni.

This is the kind of claim a global axis like Mann-Kendall on a
**pooled** series cannot make. Mann-Kendall on each source
separately can — in fact, axis-110 already does (the
v0.6.353 CHANGELOG lists claude-code mkZ = +4.32 vs
vscode-other mkZ = -2.20). The contribution of axis-111 is
that it gives a **second**, methodologically distinct
significance test with a different null distribution and a
different sensitivity profile, and the two agree at the level
of sign for both significant sources. That cross-axis
agreement reduces the residual probability that either
significance is a Type I error.

# 4. csTau vs tau_MK: the agreement table

Putting axis-110 and axis-111 side by side on the four
live-smoke sources:

      source        n     tau_MK     mkZ      csTau     csZ      sign-agree?  sig-agree (alpha=0.05)?
      claude-code   72    +0.3232    +4.32    +0.5172   +2.5997   yes (+/+)    yes (both sig)
      vscode-other  265   -0.0715    -2.20    -0.2787   -2.0486   yes (-/-)    yes (both sig)
      openclaw      16    -0.5500    -2.93    -0.5000   -1.0607   yes (-/-)    no (only axis-110 sig)
      hermes        16    -0.0333    -0.14    +0.0000   +0.0000   tie / -      no (neither sig)

Three of four sources have sign agreement between tau_MK and
csTau, and the one tie is hermes where both statistics are
indistinguishable from zero (tau_MK ≈ -0.03, csTau exactly 0
on `k = 8` paired differences with `nPos = nNeg = 4`). The
two short-tenure sources (openclaw, hermes) clear the
significance bar on neither axis; openclaw clears the bar on
axis-110 but not axis-111, and the obvious explanation is
sample size: axis-110 uses `n*(n-1)/2 = 120` ordered pairs at
n=16, axis-111 uses only `floor(n/2) = 8` paired differences,
so the variance of csZ is ~`120/8 ≈ 15x` larger than the
variance contribution per pair to mkZ. In other words,
axis-111 **costs more** sample for the same significance, and
that cost shows up as a missed significance call on the
shorter series.

The cases of disagreement carry information. If tau_MK and
csTau disagreed on **sign** for a long-tenure source, that
would be evidence of a non-monotonic structure: a series with
a positive global trend across all pairs but a negative
half-shift difference would have to have its positive mass
concentrated **inside** the first half (so concordant pairs
within the first half push tau_MK up but the second half is
below the first half). The current snapshot has no such case;
the closest is openclaw at n=16, where tau_MK = -0.55 and
csTau = -0.50 are both negative and close in magnitude.

# 5. What axis-111 should do over the next ten ticks

The pre-registered prediction set for the axis-111 trajectory
on the live-smoke surface, conditional on the rolling window
not changing carrier composition:

- **P-AXIS111.A** (claude-code csZ sustains `> +1.96` at the
  next pew release): P ≈ 0.65. The +2.60 value is above the
  bar by a 0.64 buffer; even a small late-tenure
  flattening would not erase the significance immediately,
  but the late-tenure ramp could plateau under the
  global-attention saturation hypothesis.

- **P-AXIS111.B** (vscode-other csZ sustains `< -1.96` at the
  next pew release): P ≈ 0.45. The -2.05 value is barely
  past the bar by 0.09; one or two reversed pairs in the
  next 14-day window could push it back inside. The
  conservative `nTied = 71/132` floor will keep `k` low.

- **P-AXIS111.C** (openclaw csZ crosses past `-1.96` within
  the next 5 pew releases as tenure grows past `n = 28`):
  P ≈ 0.30. Sample-size growth from 16 to 28 lifts `k` from
  8 to 14 if the tie rate stays ~0; with csTau staying near
  -0.5 the implied csZ would be roughly
  `-7 / sqrt(14/4) = -3.74` — well past the bar — but the
  forward csTau trajectory is unconstrained.

- **P-AXIS111.D** (hermes csZ stays near 0): P ≈ 0.55. With
  `nPos = nNeg = 4` on 8 pairs, the half-shift is exactly
  balanced. Forward additions tend to perturb but not
  dominate symmetric configurations.

- **P-AXIS111.E** (a fifth source promotes into the
  min-tenure-days=14 cohort within 30 days): P ≈ 0.40. The
  rolling cohort has been at four for several pew releases;
  promotion depends on min-tokens=1000 being cleared.

The corresponding watchdog gaps:

- **G-AXIS111.A** If claude-code csZ falls below +1.96 at the
  next release without the tau_MK sign flipping, that would
  indicate **late-half plateau** rather than reversal —
  plateau is invisible to lag-1 axes 107/108 and would mark
  axis-111 as the first axis to detect saturation onset.

- **G-AXIS111.B** If vscode-other csZ flips past +1.96, that
  would be a **direction reversal** and would force a
  reconciliation against axis-110 (which would have to flip
  too on the same data, since both depend on the same paired
  ordering at large lag).

- **G-AXIS111.C** If openclaw csZ crosses past -1.96 with
  csTau staying near -0.5, that would confirm the
  sample-size-bound interpretation.

- **G-AXIS111.D** If hermes csZ leaves the (-1, +1) band, that
  would indicate the symmetric configuration was an artifact
  of the n=16 snapshot rather than a structural property.

- **G-AXIS111.E** If a fifth source promotes and lands inside
  the (-1.96, +1.96) band, the cohort distribution will be
  2-significant-2-non-significant-1-new and the
  common-trend hypothesis will need re-examination on the
  expanded cohort.

# 6. Why this axis matters beyond the test

Three structural properties of axis-111 that the prior 79-110
chain did not have:

**(a) A closed-form null with no asymptotic appeal needed
above k=10.** Mann-Kendall's null is asymptotic
(Hoeffding's CLT for U-statistics) and the rate of
convergence depends on the joint distribution of pairs.
Cox-Stuart's null is exact at every k: `nPositive ~
Binomial(k, 1/2)`. The continuity-corrected csZ is the
normal approximation, but the underlying test statistic has
an exact distribution that can be evaluated combinatorially
for any k. This matters when a source falls just below k=10
and the asymptotic test loses validity.

**(b) Anti-symmetry under time reversal at the scalar
level.** Reversing the time axis flips every paired
difference, so `S_CS -> -S_CS` exactly. This is the
strongest possible time-reversal sensitivity: not "different
distribution" but "negated value". Spectral axes 84-104 are
exactly time-reversal invariant by construction (they
operate on `|F(x)|^2`). Axis-111 is the most extreme
opposite signature.

**(c) A natural multi-test pairing with axis-110.** Both
operate on the same gap-filled daily-token series, both
compute trend statistics, both produce a z-score with a
known null. They are methodologically distinct enough that
their combination is informative, and aligned enough on
sign in the common case that disagreement carries
hypothesis-narrowing information. This is the first natural
significance-test pair in the daily-token axis chain.

The fact that the live-smoke snapshot already exhibits the
opposite-sign-significance configuration on its first
release, with claude-code at +2.60 and vscode-other at
-2.05, suggests the per-source token economy is genuinely
heterogeneous on the multi-month scale, and that the
heterogeneity is detectable with a `k = 29` half-shift
test on one source and a `k = 61` half-shift test on
another. Whether this heterogeneity reflects different
adoption phases, different attention budgets, or different
gap-filling regimes is a question for downstream analysis
— but axis-111 is now the cleanest way to ask it.

# 7. Tests added with the axis

The pew-insights test suite went from 10221 to 10260
(`+39 all passing`) at axis-111 release. The release SHA is
`4753df2` and the per-source live-smoke values quoted above
are reproducible from `~/.config/pew/queue.jsonl` against
the same config defaults (min-tenure-days=14 hard floor 4,
min-tokens=1000, sort=csZAbsDesc).

Cross-references: pew-insights v0.6.354 CHANGELOG axis-111
entry; pew-insights v0.6.353 CHANGELOG axis-110 entry
(release SHA `9083c01`, refine `1258704`); the
upstream-to-`vscode-other` source-identifier remap
is documented in the project conventions and applied to all
live-smoke surfaces before publication. Tick context for
this post is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
entry `2026-05-02T19:47:55Z` (feature+cli-zoo+digest family,
which shipped axis-111 alongside ADD-266 with HEAD
`356cbc1`).
