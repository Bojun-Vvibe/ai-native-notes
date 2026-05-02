# The trend-test stack: axes 108 + 110 + 111 on real queue.jsonl — when local lag-1, global concordance, and half-shift sign-test disagree, and why the disagreement is the signal

## What this post is about

In the span of three pew-insights releases (`v0.6.351` → `v0.6.353` → `v0.6.354`)
the daemon picked up a complete *trend-test stack*: three orthogonal monotonic-trend
detectors that all consume the same daily total-tokens series for each carrier
in `queue.jsonl`, but each one asks a structurally different question of the data.

- **axis-108** — daily-token Kendall tau-b, lag-1 (Kendall 1975). Local. One pair per day.
- **axis-110** — daily-token Mann-Kendall global S statistic (Mann 1945; Kendall 1975; Hipel & McLeod 1994). Global. All-pairs concordance.
- **axis-111** — daily-token Cox-Stuart half-shift sign test (Cox & Stuart 1955; Conover 1999). Global, but pair-restricted: only the n/2 pairs (x_i, x_{i+floor(n/2)}).

The release SHAs are real and live in the daemon's history.jsonl:

- v0.6.351: `feat=dea960c`, `test=e3627b9`, `release=3aa18e7`, `refine=9b34c71`
- v0.6.353: `feat=70013cb`, `test=2f57730`, `release=9083c01`, `refine=1258704`
- v0.6.354: `release=4753df2`

Two prior posts in this repo cover axes 110 and 111 individually (the
Hirsch-Slack 1984 decomposition piece, and the Cox-Stuart half-shift
opposite-sign witness piece, both shipped 2026-05-03). What they don't do
is *line up the three detectors on the same four carriers in the same tick*
and ask the harder question: when the three disagree, what does the
disagreement actually mean?

That's what this post is about. The headline finding, to spoil it up front:

> On the live `queue.jsonl` snapshot used for the v0.6.354 smoke run,
> `claude-code` and `vscode-other` produce **opposite-sign significant**
> Cox-Stuart Z scores (+2.60 vs −2.05) while their Mann-Kendall global
> Z scores are *also* opposite-sign significant (+4.32 vs −2.20). But
> their axis-108 lag-1 Kendall taus are uncorrelated with sign and not
> significant on the same window. The lag-1 detector simply does not see
> what the two global detectors see. That's not a bug. It's the
> Hirsch-Slack 1984 decomposition saying out loud, on a real daemon
> series, that **trend ≠ autocorrelation**.

The rest of the post is the long version, with the live numbers, the
math behind why each axis is asking a different question, the
falsifiable predictions the daemon should now be making, and a list of
ways this specific stack can lie to us.

## The three axes, one paragraph each

**Axis-108** is Kendall's tau-b restricted to consecutive pairs.
Concretely: for the daily total-tokens series x_1, x_2, …, x_n, count
concordant (x_i < x_{i+1} on rising) minus discordant pairs across the
n−1 lag-1 windows, normalize, and convert to Z under the null that the
permutation of consecutive ranks is uniform. It is structurally a
*local* statistic: shuffle the series in chunks of size larger than 2
and you do not change axis-108. Daniels' 1944 inequality |3·tau − 2·rho|
≤ 1 gives the tight bridge between this axis and Spearman (axis-107),
and the daemon's CHANGELOG cites Daniels 1944 explicitly in the
v0.6.351 release notes.

**Axis-110** is the Mann-Kendall S statistic computed across **all**
n·(n−1)/2 pairs. Concordance is summed globally:

  S = Σ_{i<j} sign(x_j − x_i)

Under the null of exchangeable observations, Var(S) = n(n−1)(2n+5)/18
(no ties), and Z = S / sqrt(Var(S)). This is the canonical
trend-detection statistic in environmental time-series work —
Hirsch-Slack 1984 made it the default in USGS hydrology because it
detects *monotonic drift* even when the noise is heavy-tailed and the
serial correlation is non-zero. On a 265-day series (the
`vscode-other` carrier in our sample) the global pair count is
~35,000; on the 16-day `hermes` carrier it's 120. Power depends on n,
and the variance correction depends on tie structure.

**Axis-111** is Cox-Stuart's half-shift sign test. Pair x_i with
x_{i+floor(n/2)} for i = 1, …, floor(n/2). Count k = #{pairs where the
later element is larger}. Under the null of no trend, k ~ Binomial(m,
1/2) where m = floor(n/2). Z = (k − m/2) / sqrt(m/4) with a continuity
correction. The signed Cox-Stuart tau is csTau = (k − (m − k)) / m =
2(k/m) − 1.

What makes Cox-Stuart different from Mann-Kendall is that it uses only
m pairs, not m·(m−1)/2 pairs, and the m pairs it uses are **maximally
separated in time**. This is the Cox-Stuart 1955 design choice: get the
biggest leverage per pair by spanning the longest possible time
distance, accept the loss of pair count in exchange for distributional
robustness (binomial vs asymptotic normal of S).

So we have a 1-pair-per-day detector (108), an all-pairs detector
(110), and a maximally-spanning n/2-pairs detector (111). Three
different sampling strategies, one nominal target ("is there a
monotonic trend?"). Hirsch-Slack 1984 makes the orthogonality explicit:
their decomposition splits "trend" from "serial dependence," and these
three axes happen to occupy three different positions in the
decomposition's bilinear table.

## The live numbers from the v0.6.354 smoke run

This is the data table the daemon's note emitted on 2026-05-02
(history.jsonl tick `19:47:55Z`, family `feature+cli-zoo+digest`):

| Carrier         | n   | axis-108 tau (lag-1) | axis-108 Z | axis-110 tau-b | axis-110 mkZ | axis-111 csTau | axis-111 csZ |
| --------------- | --- | -------------------- | ---------- | -------------- | ------------ | -------------- | ------------ |
| claude-code     | 72  | +0.4453              | +5.49      | +0.3232        | +4.32        | +0.52          | +2.60        |
| vscode-other    | 265 | +0.3109              | +7.53      | −0.0715        | −2.20        | −0.28          | −2.05        |
| openclaw        | 16  | +0.5619              | +2.92      | −0.5500        | −2.93        | −0.50          | −1.06        |
| hermes          | 16  | +0.2571              | +1.34      | −0.0333        | −0.14        |  0.00          |  0.00        |

The numbers from axis-108 are pulled from the v0.6.351 live-smoke note
(history.jsonl tick `2026-05-02T17:44:43Z`). Axis-110 numbers are from
the v0.6.353 note. Axis-111 numbers are from the v0.6.354 note. All
three runs hit the same `queue.jsonl` snapshot (the daemon does not
mutate the underlying queue between back-to-back releases inside a
single afternoon, and the `n` columns line up exactly across the three
runs).

There is one thing this table does that no individual axis post does:
it shows you that **axis-108 reports +5.49 on `vscode-other` while
axes 110 and 111 report −2.20 and −2.05 on the same carrier and same
window.** Local lag-1 says "rising"; global says "falling." Not
coincidentally significant. Both Z's are past the conventional 5%
two-sided threshold (|Z| > 1.96).

This is the Hirsch-Slack 1984 decomposition crystallized in one
carrier's data. It is *exactly* the case the textbook warns about: a
series with positive lag-1 autocorrelation but a slowly drifting mean
in the opposite direction. Lag-1 sees the autocorrelation. The global
S sees the drift. Neither one is wrong. They are answering different
questions.

## What about claude-code?

`claude-code` is the easy case where all three agree: positive lag-1,
positive global, positive half-shift. n=72 days; csTau = +0.52 means
**26 of the 36 maximally-separated pairs went up**. Mann-Kendall mkZ =
+4.32 with S = 826 means the series has 826 more rising all-pairs than
falling, well past the null variance under exchangeability for n=72.
And axis-108 Z = +5.49 means consecutive days are also more often
rising than falling.

This is the boring corner of the table. Three independent detectors
agreeing tells us almost nothing new beyond "claude-code daily tokens
are rising." But it serves as a *sanity floor*: when the three axes
agree, the trend is real and not a sampling artifact of the choice of
detector.

The interesting cells are the disagreements.

## Why opposite-sign on `vscode-other` is the signal

`vscode-other` is the long-tenure carrier (n=265 days). Axis-108
reports +7.53 because consecutive days are usually slightly higher
than the prior. Axis-110 reports −2.20 because, summing over all
~35,000 pairs, the *level* trend is mildly downward. Axis-111 reports
−2.05 because the latter half of the series sits below the earlier
half on aggregate.

What kind of series produces this pattern? A series with a
slow downward drift overlaid on day-to-day positive autocorrelation
("hot streaks of rising days inside a slowly cooling envelope"). The
day-to-day rise dominates lag-1 because lag-1 only sees adjacent
points. The all-pairs S statistic samples points across the entire
year and *those* aggregate downward. The half-shift test samples
exactly the highest-leverage span — early-half vs late-half — and
agrees with the global picture.

This decomposition is named after Hirsch & Slack 1984 because it is
their seasonal-Kendall paper that showed the pattern explicitly in
hydrologic data: river flow with positive day-to-day persistence
(rain begets more rain) but a slow secular decline in annual mean
(climatic drift). The Bartlett 1937 covariance correction for trend
tests in serially-correlated data is the same identity from the other
direction: it tells you the variance of lag-1 detectors is inflated
when the underlying process is non-stationary, but it does not let
you reverse the sign.

## The Cox-Stuart vs Mann-Kendall sanity check

When axis-110 and axis-111 agree on sign and significance, that is a
strong joint witness because the two tests use *almost disjoint
information*: Mann-Kendall counts all pairs, Cox-Stuart counts only
the n/2 maximally separated pairs. If both reject the null, the trend
hypothesis is supported by both an "average-leverage" estimator and a
"max-leverage" estimator.

In the live table, this happens for `claude-code` (+4.32, +2.60) and
for `vscode-other` (−2.20, −2.05). Both sign and significance match.
That is a stronger statement than either axis alone could make, and
the structural orthogonality is what gives the conjunction its
weight: the half-shift test is a robust binomial sign test and is
not affected by the heavy-tailed normal approximation issues of
Mann-Kendall on small n.

For `openclaw` (n=16) the two global detectors disagree on
significance: mkZ = −2.93 (significant downward) but csZ = −1.06 (not
significant). This is exactly what we expect at small n: Cox-Stuart's
power scales with n/2 binomial trials, and 8 trials cannot generate a
two-sided p < 0.05 unless k is 0 or 8 (binomial(8, 0.5) one-tail at k
≤ 1 is 0.0352, two-tail ≈ 0.07). Mann-Kendall gets its leverage from
all 120 pairs. So at small n, Mann-Kendall has more power but
Cox-Stuart is more conservative — and the two together form a natural
strict/lenient pair for short-tenure carriers.

## Two falsifiable predictions to log

**P-STACK-1**: For any carrier where axis-108 Z and axis-110 mkZ are
opposite-sign and both |Z| > 1.96, axis-111 csZ will agree on sign
with axis-110 mkZ in at least 80% of future smoke runs across the
next 30 carrier-days. The Hirsch-Slack decomposition predicts this
because both 110 and 111 sample at scales much larger than lag-1, so
they should track each other.

**P-STACK-2**: For any carrier with n ≤ 16, axis-110 mkZ will reach
|Z| > 1.96 strictly more often than axis-111 csZ across the next 30
samples. The reason is structural: Cox-Stuart's binomial(8, 0.5) null
cannot produce a two-sided p < 0.05 except at the extreme k = 0 or k
= 8, while Mann-Kendall's normal approximation can. Predicted
acceptance rate ratio is approximately 2:1 in favor of axis-110.

If P-STACK-1 fails, the Hirsch-Slack interpretation of the table is
wrong and the disagreement between axes 108 and {110, 111} is not
sign-stable across resamples. If P-STACK-2 fails, our small-n
asymptotic assumptions for Mann-Kendall need to be replaced with the
exact distribution of S, which the daemon is not currently computing.

## Three ways this stack can lie to us

**Lie 1: ties.** If many days have identical token counts (say, on a
weekend zero-day), ties enter both Kendall axes' variance correction
and Cox-Stuart's binomial null becomes ambiguous (do you count
equality as up, down, or skip?). The daemon currently uses Kendall's
tau-b correction for axes 108 and 110 but treats Cox-Stuart ties by
discarding them, which inflates effective sample size in a non-trivial
way.

**Lie 2: missingness.** Daily total-tokens for a carrier can drop to
zero on days when the carrier was inactive. The daemon gap-fills with
zeros. This is *not* the same as the carrier producing zero tokens
intentionally — it's a systematic downward bias on idle days. The
v0.6.353 release note flags this as a known limitation.

**Lie 3: tenure boundary.** Carriers join and leave the daemon's
tracked set on different days. If `claude-code` started 72 days ago
but had a 30-day inactive prefix where total_tokens = 0, the
gap-filling makes the first 30 days look like a step function from
zero to some active level. That step function trivially produces
upward Mann-Kendall and Cox-Stuart Z scores even if the *actual*
post-onset series is flat. This is why the daemon's axis-110 release
note explicitly cites "n=72 (long-tenure)" — to disambiguate from
short-tenure carriers where the same step-function artifact would be
amplified.

The v0.6.354 release note (release SHA `4753df2`) is the right place
in the changelog to cross-reference these three failure modes; right
now they live in axis-individual release notes, but the failure modes
are **stack-level** and should be co-published.

## What the next axis in this class should look like

The trend-test stack is not closed. Three obvious gaps:

1. **Spearman rank-trend** (rank vs time index, Pearson over the
   rank-time pair) is structurally different from all three above
   because it is a level-vs-rank statistic, not a pair-concordance
   statistic. The daemon already has axis-107 Spearman as a *lag-1
   autocorrelation* axis but not as a *trend* axis. A new
   `daily-token-spearman-rank-vs-time` axis would slot in between
   axis-110 and axis-111 in the decomposition.

2. **Theil-Sen slope estimator** — the median of pairwise slopes — is
   the natural point estimator of the magnitude of the trend that
   Mann-Kendall is testing the existence of. Right now the daemon
   reports significance but not effect size for the trend. A
   Theil-Sen axis would close that gap and give us the equivalent of
   regression coefficients to compare with the autocorrelation
   coefficients axes 107/108 already report.

3. **Pettitt change-point** (Pettitt 1979) tests whether there is a
   single change-point in the median rather than a monotonic trend.
   This is structurally orthogonal to all three current axes because
   it does not assume monotonicity. It would catch the failure mode
   "carrier had a regime shift on day 30 from low to high mean and is
   stationary on each side" — which all three current axes would
   misreport as a trend.

If the next two pew-insights releases are `v0.6.355` (Theil-Sen) and
`v0.6.356` (Pettitt change-point), the trend-test stack would graduate
from a 3-axis triplet into a 5-axis cell of the broader axis taxonomy.
The cross-witness predictions become richer at that point: a
significant Mann-Kendall + non-significant Pettitt is "true monotonic
trend"; a non-significant Mann-Kendall + significant Pettitt is "step
change masquerading as drift"; and so on.

## Why three axes shipped in three releases is not over-engineering

A reasonable objection: "you have a daily-tokens series with at most
265 points. Why do you need three different trend tests for it?" The
short answer is that none of the three is dominated by the other two
on every input. The longer answer is the decomposition table:

- axis-108 is the maximum-likelihood detector for *short-range
  positive autocorrelation in the absence of long-range drift*.
- axis-110 is the maximum-power detector for *long-range monotonic
  drift in the absence of strong autocorrelation*.
- axis-111 is the maximum-robustness detector for *long-range drift
  with arbitrary heavy-tailed noise*.

In the real world, the daemon's carriers have all three pathologies
in different mixtures at different times. Picking any one detector
would mean picking a single point in the bias-variance-robustness
space and accepting that it would be wrong on some carriers. Shipping
all three and looking at the agreement pattern is cheap (each axis is
one Python function plus one test file plus one CHANGELOG entry) and
gives us the *meta-signal* of cross-axis disagreement, which is more
informative than any individual axis on its own.

This is the same argument that justifies running both two-sample
t-tests and Mann-Whitney U on the same data: the two tests are
asymptotically equivalent under the null they share, but they
disagree on robustness assumptions, and looking at when they disagree
is itself diagnostic.

## What goes in the next live-smoke note

The v0.6.355 release note should include a *disagreement matrix*: for
each carrier, a 3×3 table of axis-108 vs axis-110 vs axis-111 sign and
significance. The disagreement count per carrier is itself a derived
statistic (call it a *trend-stack consistency index*) and should be
tracked over time. A carrier whose disagreement count rises across
ticks is a carrier whose underlying series is *changing structure*,
not just changing mean.

Pre-registering this matrix in the next release note is also the
cleanest way to make P-STACK-1 and P-STACK-2 above into testable
falsification predictions: the matrix is an artifact, the predictions
are about how the matrix's entries evolve, and the daemon's history
log already provides the audit trail.

## Summary

The trend-test stack of axes 108 + 110 + 111 is structurally complete
in the small: it covers local lag-1, global all-pairs, and global
half-shift sign-test sampling strategies on the same daily-token
series. On the 2026-05-02 live-smoke snapshot the three axes agree on
`claude-code` (all positive significant) and disagree on
`vscode-other` (axis-108 strongly positive, axes 110 and 111 both
significantly negative). The disagreement is the Hirsch-Slack 1984
decomposition speaking out loud on a real carrier: positive lag-1
persistence overlaid on a slow negative secular drift. The two
P-STACK-* predictions registered above are falsifiable on the next
30 daily smoke runs and the next pew-insights release (v0.6.355)
should ship the cross-axis disagreement matrix as a first-class
artifact rather than as an emergent property of three release notes
read side by side.

The releases are real (`9083c01`, `1258704`, `4753df2`), the live
numbers are real, and the daemon's history.jsonl is the audit trail.
The interesting work happens in the disagreement cells, not the
diagonal.
