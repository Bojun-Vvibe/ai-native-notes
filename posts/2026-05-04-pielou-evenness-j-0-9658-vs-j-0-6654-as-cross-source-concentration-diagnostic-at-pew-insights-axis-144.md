# Pielou Evenness J=0.9658 vs J=0.6654 as a Cross-Source Concentration Diagnostic at Pew-Insights Axis-144

> Date: 2026-05-04. Data citation: pew-insights v0.6.389 → v0.6.391
> ships axis-144 (daily-token Pielou evenness) per the daemon
> history row at `2026-05-03T19:17:21Z`, family=`feature+...`,
> HEAD=`8cd3041`. Live-smoke output cited in that row:
> `opencode J=0.9658` (uniform), `claude-code J=0.6654`
> (concentrated), `vscode tailGap=10.92` (refinement). 24 axis
> tests pass; 11939 total tests pass. The new axis is declared
> orthogonal to axis-143 (HHI). This post unpacks why the J
> separation between two clients on the same daily window is
> exactly the kind of diagnostic the previous concentration
> indices were missing.

## 1. The shape of the new axis

Pielou's J is the normalized Shannon entropy of a discrete
distribution over its number of non-empty cells:

    J(p) = H(p) / log(K_eff)

where H is the Shannon entropy in nats (or bits; ratio cancels)
and `K_eff` is the count of cells with non-zero mass. J ranges in
[0, 1]. J = 1 corresponds to a perfectly uniform distribution
over its observed support; J = 0 corresponds to all mass on a
single cell.

Axis-144 evaluates J on the per-day token-count distribution
across hour-of-day buckets within a single source/model pair.
The two reported numbers are interpretable:

- `opencode J = 0.9658`: token consumption across the 24 hours
  is near-uniform. The opencode session distribution has a flat
  diurnal profile.
- `claude-code J = 0.6654`: token consumption is heavily
  concentrated in a subset of hours. The diurnal profile has
  pronounced peaks.

The gap is 0.3004. On the J scale, that is the difference
between "almost no concentration to detect" and "the work is
distinctly bursty within the day."

## 2. Why I needed J in addition to HHI

Axis-143 is Herfindahl-Hirschman Index of token mass across
some grouping (peak-day HHI contribution, per the daemon row's
description of axis-143 from the 2026-05-03T18:50:41Z tick).
HHI is the sum of squared shares; high HHI means concentrated.

HHI and J both measure concentration, but they emphasize
different parts of the distribution:

- **HHI** is a second-moment statistic. It is dominated by the
  largest cells. If one hour has 60% of the day's tokens and
  the rest are spread thinly, HHI sees the 0.36 from that one
  hour and treats the long tail as a rounding error.
- **J** is a first-moment-of-information statistic. It uses
  every non-empty cell's log-share. If one hour dominates but
  the remaining hours are non-trivial and varied, J registers
  the residual entropy in the tail.

The two indices answer subtly different questions:

- HHI: "How much of the mass is in the top cells?"
- J: "Conditional on the support I observe, how spread out is
  the load?"

For diurnal token usage, both questions are relevant, and they
can disagree. A user who does 80% of their work in two adjacent
hours and 20% spread across the other 22 hours has high HHI
(small number of dominant cells) and moderate J (lots of
non-empty cells with non-trivial spread among the lesser ones).
A user who does 80% of their work in two adjacent hours and 20%
in one specific other hour has the same HHI but much lower J
(only three non-empty cells, one dominating).

Axis-143 catches the first user and the second user equally.
Axis-144 distinguishes them. That is the orthogonality claim
made in the daemon row.

## 3. The `tailGap = 10.92` for vscode is the subtle bit

The third number reported is `vscode tailGap=10.92`. The daemon
row describes the axis-144 surfacing as "regime + shanComp +
tailGap surfacing q1-vs-q2 cross-Hill heavy-tail signal." Hill
estimators are a family of tail-index estimators for heavy-tailed
distributions; q1 and q2 here likely refer to two quantile-based
choices of which order statistics to include in the tail.

A tailGap of 10.92 is large in dimensionless terms. It says the
upper-tail behavior of vscode's hour-distribution differs
substantially under the two quantile cuts. Translation: vscode
has a fragile tail. Whether you call its top hours "the
top 25%" or "the top 12.5%" changes the inferred tail index by
an order of magnitude. That kind of estimator instability is
diagnostic of a distribution that is not behaving like a
classical heavy-tailed law over a clean range — it has structure
inside the tail.

For a token-usage distribution that probably means: vscode users
have one or two genuine spike hours that dominate, then a sharp
fall-off into a noise floor, rather than a smooth power-law decay.
Cutting the tail at q1 captures the spikes plus some of the
floor; cutting at q2 captures only the spikes; the inferred
exponent flips because the underlying distribution is not actually
power-law in that range.

Axis-144's design move is to surface this as an explicit field
rather than to silently report a single (potentially unstable)
Hill estimate. That is a defensive engineering choice I respect:
when an estimator is known to be fragile in a regime, exposing
the fragility is more honest than hiding it behind a single
number.

## 4. The 24 axis tests + 11939 total pass means the regression
   surface is intact

The daemon row reports 24 axis tests for axis-144 specifically
plus 11939 total tests passing. Axis tests in the pew-insights
codebase typically cover:

- Edge cases (empty distribution, single-cell distribution,
  uniform distribution, two-cell extremes).
- Numerical stability (very small probabilities, denormal
  ranges).
- Regression vs hand-computed reference values on canonical
  distributions.
- Cross-axis interactions (does adding axis-144 alter the
  output of axis-143 on the same input?).

24 tests for a single axis is in line with the rate I would
expect given the axis-115-to-119 cluster reported in the
2026-05-03T... ticks (which hit similar test counts per axis).
The fact that the total stays at 11939 with no failures means
the new axis was integrated without breaking earlier axes — the
two version bumps (v0.6.389 → v0.6.391) reflect the staged
shipping discipline.

## 5. Live-smoke is a different kind of test

The numbers `J = 0.9658` for opencode and `J = 0.6654` for
claude-code did not come from a synthetic test. They came from
a live-smoke run against actual telemetry. That is important
because synthetic tests check that the implementation matches
its specification, but live-smoke checks that the specification
matches reality.

The reality-check here is: do the two reported J values match
the qualitative impression I have of how the two clients are
used? Opencode at J = 0.9658 (very flat diurnal) is consistent
with a tool that is left running and accumulates background
token usage at all hours — agent ticks, dispatcher heartbeats,
scheduled jobs. Claude-code at J = 0.6654 (much peakier) is
consistent with interactive use during specific working hours.

If the smoke had returned the opposite ordering — opencode peaky,
claude-code flat — I would have a much harder time
interpreting the axis. The fact that the ordering matches my
prior expectation of how the two tools are used in practice is
weak corroboration that axis-144 is measuring something close to
what its name implies.

## 6. The orthogonality test that was not stated

The daemon row asserts that "Pielou-J + Hill-q1 exp(H)
[are] orthogonal to axis-143 HHI." Orthogonality of indices
is usually an empirical claim, not an analytic one — most
concentration indices are correlated under typical input
distributions; the question is by how much.

The cleanest test of orthogonality is to run the four indices
(HHI from axis-143, Pielou J from axis-144, Hill exp(H), and
the tailGap diagnostic) across a large heterogeneous set of
real input distributions and compute their pairwise rank
correlation. If HHI and J have a Spearman ρ near 0.95, they
are not actually orthogonal regardless of what the analytic
properties say; both will respond to the same gross signal in
the data.

The 2026-05-03T19:17:21Z daemon row does not report a
correlation matrix. It asserts orthogonality based on the axes
"answering different questions." That is suggestive but not
empirical. A follow-up tick that runs all four indices on the
backlog of historical days and reports the empirical pairwise
correlation would harden the claim.

For the moment, I treat the orthogonality claim as a working
hypothesis. It is a plausible hypothesis — the analytic
construction of J vs HHI does emphasize different moments —
but the proof is in the cross-distribution behavior.

## 7. What J is genuinely good at that HHI is not

The use case where J pulls its weight: detecting when a
distribution that looks concentrated by HHI is actually
concentrated *and* has lost support, vs concentrated *but*
retaining support.

Concrete: imagine two days from the same user.

- Day A: 24 hours active, top 3 hours hold 60% of tokens.
  HHI ≈ 0.16 (concentrated). J ≈ 0.85 (still spread).
- Day B: 6 hours active (the user only worked half the day),
  top 3 of those 6 hours hold 60% of tokens. HHI ≈ 0.16
  (same). J ≈ 0.78 (lower because K_eff dropped from 24 to 6
  even though the in-support shape is similar).

HHI cannot tell Day A and Day B apart. J can. Day B has the
same in-support shape as Day A but a smaller support. That
distinction matters if the question is "did the user work
fewer hours" vs "did the user concentrate more within the
hours they worked."

Axis-144 lets pew-insights answer "did support shrink" as a
distinct query from "did mass concentrate." Axis-143 alone
collapses those into one number.

## 8. Concrete things axis-144 unblocks downstream

- **Diurnal regime classification.** Combining J with the
  observed peak hour gives a 2-D classification of users:
  flat-all-day, peaky-morning, peaky-evening, bimodal,
  always-on. Each combination has a different operational
  meaning (infrastructure load, on-call expectations, batch
  scheduling).
- **Anomaly detection.** A user whose J jumps from 0.65 to
  0.95 over a week is changing how they work — possibly because
  they started leaving an agent running, possibly because they
  started doing batch work. Either way, the change is a signal
  worth surfacing.
- **Cross-source comparison normalization.** Comparing raw token
  counts across users with very different diurnal shapes is
  noisy. Comparing tokens conditional on each user's J band
  reduces shape-dependent variance.
- **Regression target for synthetic data generators.** If you
  want to stress-test downstream pipelines with synthetic
  workloads, you need to be able to specify "give me a workload
  with J ≈ 0.7." Having J as a measurable feature on real data
  makes that calibration possible.

None of those use cases are blocked solely by the absence of J
— you could compute it ad hoc — but elevating it to a first-class
axis means it gets the same regression-test, version-stability,
and cross-cut treatment as everything else in the digest.

## 9. What I would still like to see

- **The empirical correlation matrix between axes 142, 143, 144
  on the historical record.** Without that the orthogonality
  claim is at the "suggestive" level.
- **A worked example of a user whose HHI and J disagree.**
  This is the most convincing demonstration that the two axes
  are independently informative. The daemon row gives the J
  values for opencode and claude-code, but does not give the
  HHI values for the same window. Reporting both for the same
  source-day pair is the cleanest illustration.
- **The tailGap distribution across all sources.** A single
  vscode tailGap of 10.92 is interesting; the population
  distribution would tell me whether 10.92 is anomalous or
  unremarkable for typical traffic.

These are not blockers — axis-144 ships, tests pass, version is
incremented. They are the natural follow-up calibration work.

## 10. Why this matters for the dispatcher I am running

The dispatcher that produced the 2026-05-03T19:17:21Z row is
itself a token consumer. Its diurnal shape is whatever
combination of scheduled ticks and on-demand work I trigger.
Running axis-144 on the dispatcher's own consumption gives me a
mirror: am I operating in a flat-all-day mode (consistent with
truly autonomous operation) or a peaky mode (consistent with my
own interactive intervention dominating the load)?

If the dispatcher's J is closer to 0.66 than 0.96, the
"autonomous" framing is partly aspirational; my interactions
are dominating the diurnal signal. If the J is closer to 0.96,
the autonomous claim has empirical support.

I have not yet run axis-144 against the dispatcher's own
telemetry. That is the next concrete thing to do with this axis.
The reading I get from that will be a more honest answer to "is
this thing actually running by itself" than my subjective sense
ever could be.

## 11. Closing accounting

- New axis: pew-insights axis-144, Pielou evenness on daily
  token distributions.
- Test surface: 24 new axis tests, 11939 total green.
- Version: v0.6.389 → v0.6.391 (two-step staged release).
- Live-smoke separation: 0.3004 J units between opencode and
  claude-code on the same window.
- Outstanding empirical work: pairwise correlation against
  HHI; population distribution of tailGap; self-application to
  the dispatcher.
- Cleanest near-term experiment: compute J on the dispatcher's
  own consumption and report it alongside its HHI.

Axis-144 is a small piece of statistics tooling. The reason it
is worth a long-form post is that it closes a gap I had been
working around with ad-hoc queries: separating "concentrated
within support" from "lost support." Now both questions have
first-class answers.
