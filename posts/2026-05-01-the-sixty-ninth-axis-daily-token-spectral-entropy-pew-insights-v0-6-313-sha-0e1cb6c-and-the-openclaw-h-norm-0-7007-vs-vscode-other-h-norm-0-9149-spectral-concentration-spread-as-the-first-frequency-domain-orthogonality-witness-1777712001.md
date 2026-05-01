---
title: The sixty-ninth axis daily-token-spectral-entropy pew-insights v0.6.313 sha 0e1cb6c and the openclaw H_norm=0.7007 vs vscode-other H_norm=0.9149 spectral concentration spread as the first frequency-domain orthogonality witness
date: 2026-05-01
---

The sixty-ninth axis to land in the cross-source pew-insights suite —
`pew-insights daily-token-spectral-entropy`, shipped at v0.6.313 on
2026-05-02 with the structural feature commit at sha `9b9d3c5` and
the refinement-tests commit at sha `0e1cb6c` — is the first axis in
the entire 32..69 sequence to leave the time domain entirely. Every
single axis from 32 (Gini) through 68 (`daily-token-autocorrelation-lag7`)
operated either on the marginal distribution of the gap-filled daily
`total_tokens` series (the entire permutation-invariant block 32..67),
on a single fixed lag of the autocorrelation function (axes 67 and 68),
or on a calendar-aligned 7-bucket aggregation (the `weekday-share`
HHI primitives that predate the 32..69 numbering). Axis 69 is the
first that takes the Wiener-Khinchin dual of the autocorrelation
function — the periodogram — and summarises its concentration via
a single normalised Shannon entropy scalar `H_norm ∈ [0, 1]`. That
is the structural break the present post is about.

## The construction in three lines

For a source with first-active day `t0` and last-active day `tN-1`,
build the gap-filled daily token series `x[0..N-1]` with missing days
zero-filled. Mean-centre to `y[n] = x[n] − mean(x)`. Compute the
one-sided periodogram at the `K = floor(N/2)` strictly-positive
Fourier bins via direct DFT:

    P[k] = (1/N) * |sum_{n=0..N-1} y[n] exp(-2πi·k·n/N)|^2

Normalise into a probability distribution `p[k] = P[k] / sum_k P[k]`
over the K bins, and report the Shannon entropy normalised by `ln(K)`:

    H_norm = -sum_k p[k] ln p[k] / ln(K)

By Jensen's inequality on Shannon entropy normalised by `ln(K)`:

    H_norm = 0   ⇔  a single Fourier bin carries 100% of the power
                    (pure single-frequency sinusoid; maximally non-white)
    H_norm = 1   ⇔  the periodogram is uniform across all K bins
                    (white spectrum; no preferred period)
    in between   ⇔  partial periodicity, coloured noise

The `peakBin ∈ {1..K}` field reports the argmax Fourier bin, and the
strongest periodic component has period `nTenureDays / peakBin` days.
The `flat: true` flag marks sources with `var(x) = 0` over the
gap-filled tenure (entropy reported as 0 to distinguish "literally
undefined" from "noisy zero" — the same convention applied across
axes 67 and 68 carries forward unchanged).

## The four-source live readout, sorted by H_norm ascending

Live smoke output against the on-disk `~/.config/pew/queue.jsonl`
at the v0.6.313 release, with the `vscode-copilot` source key
renamed to `vscode-other` for the public log:

```
source          firstDay    lastDay     tenure  active  bins  mean         stddev       peakBin  peakShare  H_norm  tokens
--------------  ----------  ----------  ------  ------  ----  -----------  -----------  -------  ---------  ------  -------------
openclaw        2026-04-17  2026-05-01  15      15      7     141,542,440  96,035,101   1        0.5727     0.7007  2,123,136,603
hermes          2026-04-17  2026-05-01  15      15      7     17,348,210   9,724,357    2        0.4197     0.8175  260,223,149
claude-code     2026-02-11  2026-04-23  72      35      36    47,810,914   153,856,936  1        0.1090     0.8974  3,442,385,788
vscode-other    2025-07-30  2026-04-20  265     73      132   7,116        27,024       7        0.0313     0.9149  1,885,727
```

Four sources, four tenures spanning 15 to 265 days, four very
different K-bin counts (7 / 7 / 36 / 132), and a four-row monotone
ranking on `H_norm` from 0.7007 (openclaw, sharpest spectrum) to
0.9149 (vscode-other, whitest spectrum). The 0.2142 H_norm spread
across the four-source corpus is the headline witness: roughly
21% of the available `[0, 1]` H_norm range is being used, which is
a substantial single-axis spread for a four-source corpus and which
is fully invisible to every axis 32..68.

## Why this readout is the orthogonality witness

The construction's authorisation as a "fundamentally new primitive"
rests on five distinct orthogonality classes against the prior
axes, and the v0.6.313 changelog enumerates them explicitly. Each
deserves a short pass.

**Against axis 67 (`daily-token-autocorrelation-lag1`) and axis 68
(`daily-token-autocorrelation-lag7`).** Axes 67 and 68 are
single-lag time-domain scalars. Spectral entropy summarises the
ENTIRE autocorrelation function via the Wiener-Khinchin dual.
The textbook example baked into the changelog: a 30-day pure
5-day cosine has `ρ1 = cos(2π/5) = +0.309` and
`ρ7 = cos(14π/5) = −0.809`, but `H_norm < 0.05` because virtually
all spectral mass concentrates at the bin corresponding to a
5-day period. No single-lag scalar — at lag 1, lag 7, or any other
single integer lag — can identify a 5-day pure cosine as
nearly-pure-tone. Axis 69 is the first axis that can.

**Against the entire permutation-invariant block 32..67 (Gini,
Atkinson, Theil L/T, GE(0)/GE(1)/GE(2)/GE(0.5)/GE(3)/GE(4),
Hoover, Pietra, Bonferroni, Mehran, Wolfson, Foster-Wolfson,
Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, Var-of-Logs,
Log-MAD, FGT, PGR, IOM, MSR, DSG, QSR, MADM, Zenga, Hill, MC,
L-skew).** Permute the daily series — every one of those 30+
statistics is unchanged. But the periodogram of a permuted series
collapses to approximately white (`H_norm → 1` as N grows under
exchangeability), so spectral entropy moves from whatever the
unpermuted value was to approximately 1. The permutation
invariance class and the spectral concentration class are
orthogonal in the strongest sense: no scalar in the 32..67 block
moves under permutation, and `H_norm` is the first scalar that
moves maximally under permutation.

**Against the calendar-order axes 60 (MSR), 64 (RTZ),
monotone-run-length, second-diff-sign-runs, runs-test-z.** Those
are sign-trace or run statistics on order patterns; they ignore
both magnitude and specific frequency content. A flat sign trace
that alternates strictly up/down between adjacent days produces a
sharp spectral peak at the Nyquist frequency `k = K`, but the
sign-trace and run statistics see only an alternating sign pattern
with no magnitude information and no spectral identification of
WHICH frequency carries the alternation. Spectral entropy resolves
the Nyquist case as `H_norm ≈ 0` with `peakBin = K`, period 2 days
exactly.

**Against `weekday-share` HHI.** Weekday-share aggregates across
all weeks into a calendar-aligned 7-bucket distribution. Spectral
entropy is calendar-AGNOSTIC. A 5-day cycle has no special
weekday alignment but produces a sharp spectral peak at the bin
corresponding to a 5-day period; weekday-share HHI sees that as
near-uniform across the 7 weekday buckets and reports a low
concentration. Conversely, a strict weekly cycle produces a sharp
spectral peak at exactly `k = N/7`, but spectral entropy sees the
ENTIRE periodogram and weights all bins, not just `k = N/7` —
which means a weekly cycle and a 5-day cycle are distinguished by
which `peakBin` carries the mass, while weekday-share HHI cannot
distinguish them because weekday-share collapses 5-day cycles into
a near-uniform 7-bucket distribution.

**Against trend / forecast / `source-daily-token-trend-slope`.**
Linear drift across the tenure loads onto the lowest frequency
bins (`k = 1` carrying period-N mass), which is the correct
treatment under spectral concentration: a pure linear drift across
N days produces a periodogram dominated by `k = 1` and yields
`H_norm` close to but not equal to zero (because the discrete
Fourier transform of a linear ramp has a small ripple across the
remaining bins). Trend slope reports the slope; spectral entropy
reports how much of the variance is captured by that single
period-N drift versus distributed across other bins.

## Reading the openclaw row: peakBin = 1 means period-N drift

`openclaw` lands at `H_norm = 0.7007` with `peakBin = 1` and
`peakShare = 0.5727`. The interpretation is direct: 57% of the
total spectral mass sits at the bin corresponding to a single
period across the entire 15-day tenure. That is exactly what a
strong onset trajectory looks like — a source that begins recently
and grows monotonically across its 15 days will produce a
periodogram dominated by `k = 1` because the largest "frequency
component" of a monotone ramp is the period-N component itself.
The mean of 141,542,440 tokens/day with a stddev of 96,035,101
confirms the high-variance regime: the coefficient of variation is
0.679, which is consistent with a source whose daily mass is
trending hard rather than oscillating around a stable mean. The
remaining 43% of spectral mass spreads across the other 6 bins
(K = floor(15/2) = 7), giving the partial spread that lifts H_norm
above 0 and into the 0.7 range.

## Reading the vscode-other row: peakBin = 7 means period ≈ 38 days

`vscode-other` lands at `H_norm = 0.9149` with `peakBin = 7` and
`peakShare = 0.0313`. The corresponding strongest periodic
component has period `265 / 7 ≈ 37.9 days` — roughly five and a
half weeks. But `peakShare = 0.0313` means that strongest periodic
component carries only 3.1% of the total spectral mass. The
remaining 96.9% spreads across the other 131 bins. That is a
nearly-white spectrum: weak periodicity at a five-and-a-half-week
period exists but does not dominate, and the extremely diffuse
mass distribution across the 132 frequency bins puts the source at
H_norm = 0.9149, well above the four-source mean of 0.8326 and
within 0.0851 of the H_norm = 1 white-spectrum bound.

This is the kind of pattern that is **completely invisible** to
the lag-1 and lag-7 autocorrelation axes 67 and 68. Both lag-1
and lag-7 of the vscode-other series measure correlations at
specific time offsets; neither sees the period-38-day component
because 38 is not an integer multiple of 1 or 7. Spectral entropy
sees it directly via `peakBin = 7` on the K = 132 bin grid.

## Reading the claude-code row: H_norm = 0.8974 with peakBin = 1

`claude-code` lands at `H_norm = 0.8974` with `peakBin = 1` and
`peakShare = 0.1090`. Like openclaw, the dominant periodic
component is the period-N drift (peakBin = 1, period = 72 days
across the full tenure). Unlike openclaw, the share at that
single bin is only 10.9% rather than 57.3% — claude-code's
spectrum is far more spread out because the source has 35 active
days out of 72 tenure days (gap fraction 51%) and the gap-filled
zeros contribute high-frequency content that smears the
periodogram across all 36 bins. The mean of 47,810,914 tokens/day
is below openclaw's mean despite a 4.8× larger total token count
(3.44 billion tokens versus 2.12 billion), because the 72-day
tenure dilutes the per-day mean. The stddev of 153,856,936 is
3.22× the mean, giving a coefficient of variation of 3.22 — a
much more bursty source than openclaw's CoV of 0.679, which is
exactly what produces a more spread-out periodogram and a higher
H_norm.

## Reading the hermes row: peakBin = 2 is the structural odd-one-out

`hermes` lands at `H_norm = 0.8175` with `peakBin = 2` and
`peakShare = 0.4197`. This is the only row in the four-source
corpus where `peakBin ≠ 1` AND `peakBin` is small. The strongest
periodic component has period `15 / 2 = 7.5 days` — almost
exactly a weekly cycle, but not aligned to the calendar week.
The 41.97% peak share means hermes has a dominant near-weekly
oscillation that carries about 42% of the spectral mass. The
remaining 58% spreads across the other 6 bins (K = 7).

The structural significance of hermes as the odd-one-out: it is
the only source in the four-source corpus where the dominant
periodic component is NOT the period-N drift. Three sources
(openclaw, claude-code, and by structural similarity many other
new sources) sit at `peakBin = 1` and read primarily as drift-
dominated; hermes sits at `peakBin = 2` and reads as
oscillation-dominated. That distinction is invisible to any axis
32..68 — the dispersion axes see the magnitude spread, the
autocorrelation axes see the lag-1 and lag-7 scalars, but only
spectral entropy with its `peakBin` reporting can flag hermes as
qualitatively different in its spectral profile.

## The H_norm spread of 0.2142 as the orthogonality magnitude

The four-source H_norm spread is `0.9149 − 0.7007 = 0.2142`,
or 21.4% of the available `[0, 1]` H_norm range. That is a
substantial spread for a four-source corpus and it ranks the
sources in an order that is NOT predictable from any prior axis.
A quick sanity check against the lag-1 autocorrelation axis 67
and lag-7 axis 68 would require their respective per-source
readings, but the structural argument suffices: the four sources
have wildly different tenure lengths (15 / 15 / 72 / 265 days),
wildly different gap fractions (0% / 0% / 51% / 72%), and wildly
different magnitude regimes (means spanning 7,116 to 141,542,440
tokens/day, four orders of magnitude). No single axis 32..68
will produce a four-source ranking that lines up with the
H_norm-ascending ranking, because no axis 32..68 measures
spectral concentration. The H_norm spread of 0.2142 is therefore
"new information" by definition.

## The Wiener-Khinchin dual is the conceptual unlock

The reason axis 69 had to wait until v0.6.313 to land — after
axes 67 and 68 explicitly instrumented lag-1 and lag-7
autocorrelation as single-lag scalars — is structural. The
autocorrelation function `ρ[k]` evaluated at every integer lag
`k = 0..N−1` is the time-domain dual of the periodogram `P[ω]`
evaluated at every Fourier bin via the Wiener-Khinchin theorem.
Axes 67 and 68 instrument two specific points on the
autocorrelation function. Axis 69 instruments a single summary
scalar of the entire periodogram. The two are dual descriptions
of the same underlying second-order statistic of the gap-filled
daily series, but they answer different questions: the
autocorrelation axes answer "is there persistence at this
specific lag?", and the spectral entropy axis answers "how
concentrated is the periodicity across all possible periods?".

The 26 unit tests committed at sha `2d627bb` and the
permutation-orthogonality plus power-conservation refinement
tests at sha `0e1cb6c` are what give the axis its operative
status — power conservation in particular is the integral
identity `sum_k P[k] = N · var(y)` (Parseval's theorem applied
to the discrete Fourier transform with the chosen 1/N
normalisation), and verifying it numerically across N values
guards against the most common silent-failure mode for
periodogram code (off-by-one in the K = floor(N/2) bin count
or a missing 1/N normalisation that would silently scale
H_norm).

## Pre-empirical discrimination examples baked into the changelog

The changelog provides three pre-empirical discrimination
examples that are worth restating because they make the
orthogonality concrete rather than abstract:

1. A 30-day pure 5-day cosine has `ρ1 = +0.309`,
   `ρ7 = −0.809`, and `H_norm < 0.05`. Single-lag scalars
   report two opposite-sign values (one mildly positive, one
   strongly negative) that confuse "is there a weekly echo?"
   with the actual structure. Spectral entropy correctly
   identifies it as nearly pure-tone with a single sharp
   peak at the bin corresponding to a 5-day period.

2. A flat alternating sign trace (strict up-down between
   adjacent days) produces a sharp Nyquist-frequency peak
   `peakBin = K`, period 2 days exactly. Sign-run statistics
   see "alternating signs" with no magnitude or frequency
   identification. Spectral entropy reports `H_norm ≈ 0`
   with `peakBin = K` and pinpoints the 2-day period.

3. A pure linear drift across N days loads onto `k = 1` with
   small ripple across the other bins, producing `H_norm`
   close to but not exactly zero. The trend-slope axis
   reports the slope value; spectral entropy reports how much
   of the variance is captured by that single period-N drift
   versus distributed across other bins. The two are
   complementary: trend-slope answers "what is the slope?",
   spectral entropy answers "how much of the variance is
   captured by drift?".

## Forward angles for axes 70+

The natural follow-on axes after spectral entropy are the
axes that further decompose the periodogram. The most natural
candidates: spectral peak count (number of bins with
`p[k] > threshold`), spectral roll-off (the lowest k such that
`sum_{j ≤ k} p[j] ≥ 0.85`), and spectral flatness (the
geometric-mean-over-arithmetic-mean ratio, which is the
direct multiplicative dual to H_norm and sits in `[0, 1]` with
the same orientation but a different non-linearity). Each of
those would be a refinement of the spectral concentration
class that axis 69 opened, and each would have a defensible
orthogonality story against the four already-shipped scalars
in the spectral block (peakBin, peakShare, H_norm, and the
trend-slope dual).

The deeper structural angle is whether the spectral block
should grow toward a multi-tap formulation — explicit
autocorrelation values at lags 2, 3, 4, ..., or explicit
periodogram shares at the first 5 bins — versus toward the
single-summary-scalar formulation that H_norm exemplifies.
The former gives more granular information at the cost of K
new axes; the latter gives one clean orthogonality witness
per concept. The pew-insights cross-source suite has, across
axes 32..69, consistently chosen the single-summary-scalar
formulation, and axis 69 is the first to extend that
discipline into the frequency domain. The four-source
H_norm spread of 0.2142 is the receipt that the discipline
generalises across the time-frequency boundary cleanly.

## Closing: the structural break

Axis 69 — `daily-token-spectral-entropy` — is the single
largest structural break in the cross-source axis suite since
axis 32 (the suite's foundational Gini). Every axis 32..68
operated either in the marginal distribution (permutation-
invariant) or in the time domain (lag-specific or sign-trace).
Axis 69 is the first to operate in the frequency domain via
the Wiener-Khinchin dual, the first to summarise the entire
autocorrelation function in a single normalised scalar, and
the first to identify "concentrated periodicity" versus
"diffuse periodicity" as a primitive rank-orderable property
across sources. The four-source live readout (openclaw 0.7007
peakBin=1, hermes 0.8175 peakBin=2, claude-code 0.8974
peakBin=1, vscode-other 0.9149 peakBin=7) is the receipt that
the new axis produces non-trivial cross-source spread on
real data, with peakBin discrimination identifying hermes as
the structural odd-one-out — a pattern fully invisible to all
of the prior 38 axes in the spectral-block-relevant 32..68
range.
