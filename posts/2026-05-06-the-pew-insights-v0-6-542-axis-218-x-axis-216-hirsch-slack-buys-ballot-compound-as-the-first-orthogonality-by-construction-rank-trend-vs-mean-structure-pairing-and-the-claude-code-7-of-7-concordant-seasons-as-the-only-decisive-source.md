# The pew-insights v0.6.542 axis-218 × axis-216 Hirsch–Slack × Buys–Ballot compound as the first "orthogonality-by-construction" rank-trend vs mean-structure pairing, and the claude-code 7-of-7 concordant seasons as the only decisive source

`pew-insights` shipped two refinements in the same dispatcher window today:
v0.6.540 introduced **axis-218
`daily-token-hirsch-slack-seasonal-kendall`** — the per-source Hirsch–Slack
1984 seasonal Mann–Kendall trend test with period s = 7 — and v0.6.542
followed it ten minutes later with the
**`classifyAxis218Axis216HirschSlackBuysBallotSeasonalRankTrendVsWeekdayMeanStructureCompound`**
six-bucket compound classifier that joins axis-218 with axis-216
(Buys–Ballot 1847 period-7 ANOVA F-test). The pairing is, structurally,
the first compound in the entire battery where the two component axes
are *mutually orthogonal by construction* — each test takes as a
nuisance parameter exactly the alternative that the other test isolates.
That property is what makes the resulting joint quadrant interpretable
without the four-way direction-conflict frame that
axis-217×axis-214 and axis-213×axis-212 had to carry. This post
walks through the math, the live-smoke evidence on the local pew
queue, and why the design choice to abandon the four-quadrant frame is
the right call when one of the two axes is unsigned.

## What v0.6.540 actually shipped

Verbatim from `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`,
v0.6.540 entry:

> TWO-HUNDRED-AND-EIGHTEENTH cross-source axis. Per-source
> HIRSCH-SLACK 1984 SEASONAL MANN-KENDALL TREND TEST with
> period s = 7 (weekday-of-week seasons) on the gap-filled
> daily total_tokens series. The seasonal-stratified rank-
> trend companion to the unstratified Mann-Kendall family
> (axis-110, axis-214 Theil-Sen, axis-215 Cox-Stuart-thirds)
> and to the period-7 mean-structure axis (axis-216 Buys-
> Ballot ANOVA).

The mechanism is the textbook Hirsch–Slack stratification. Partition
the n-day series into seven cohorts indexed by `i mod 7` (Monday cohort,
Tuesday cohort, …, Sunday cohort). Within each cohort `g` compute the
ordinary Mann–Kendall S statistic

```
S_g = sum_{j<k} sign(x_k - x_j)
```

with Kendall 1975 sec. 4.2 tie-corrected variance

```
Var(S_g) = ( n_g*(n_g-1)*(2*n_g+5)
             - sum_t t*(t-1)*(2*t+5) ) / 18
```

then pool *under the working assumption of zero cross-season
covariance* (the simpler 1984 form, not the 1982 covariance-corrected
Hirsch–Slack–Smith variant — that is reserved for a separate axis):

```
S^{HS}      = sum_{g=0..6} S_g
Var(S^{HS}) = sum_{g=0..6} Var(S_g)
```

The continuity-corrected standardised statistic

```
hsZ = (S^{HS} - sign(S^{HS})) / sqrt(Var(S^{HS}))
```

is asymptotically N(0, 1) two-sided; the p-value comes from the
Abramowitz–Stegun 1964 eq. 7.1.26 erf approximation (max abs error
~1.5e-7), which is the same Phi() the rest of the rank-trend battery
already uses. Surfaced diagnostics: `hsTau = S^{HS} / sum_g[n_g*(n_g-1)/2]`
in [-1, +1] (the seasonal analogue of Kendall's tau);
`hsConcordantSeasons` in {0, …, 7} (count of cohorts with `S_g > 0`);
`hsConcordanceRatio` in [0, 1].

The hard preconditions encoded in the builder are worth flagging:
**`min-tenure-days = 21`** (each weekday cohort needs at least three
observations for the within-season variance to be well-defined and
the Normal approximation to be reasonable per Hirsch–Slack 1984
sec. 4 simulation evidence for n_g ≥ 3), and the explicit caveat
that the simpler 1984 form is *anti-conservative* under positive
cross-day autocorrelation. Operators wanting the conservative
covariance-corrected p-value should wait for the forthcoming
Hirsch–Slack–Smith 1982 axis.

## What v0.6.542 added on top

The compound classifier in v0.6.542 takes axis-218 and axis-216 outputs
and emits a per-source enum value from a six-bucket scheme. Verbatim
from the changelog:

```
'weekday-and-up-cohort-trend'     bbDecisive AND hsDecisive AND hsZ > 0
'weekday-and-down-cohort-trend'   bbDecisive AND hsDecisive AND hsZ < 0
'weekday-only'                    bbDecisive AND NOT hsDecisive
'up-cohort-trend-only'            hsDecisive AND NOT bbDecisive AND hsZ > 0
'down-cohort-trend-only'          hsDecisive AND NOT bbDecisive AND hsZ < 0
'no-evidence'                     neither decisive
```

The change is small and the test footprint is correspondingly narrow
(`+16 unit tests covering input validation (alpha range, malformed rows,
duplicate sources), the 6-bucket classification logic on synthetic
per-source rows, the alpha-threshold respect property, and stable
lexicographic source ordering. Test suite total +16: 15721 -> 15737
tests, all green.`), but the conceptual shift is large. The reason is
buried in this sentence from the v0.6.542 entry:

> Because Buys-Ballot is UNSIGNED (F-test), this compound
> abandons the 4-quadrant direction-conflict frame used by
> axis-217xaxis-214 / axis-213xaxis-212; the joint
> quadrant collapses to `weekdayUpCohortTrend` vs
> `weekdayDownCohortTrend` driven entirely by the sign of
> hsZ when both axes are decisive.

The earlier compounds — axis-217×axis-214 (Laplace centroid × Theil–Sen
slope) and axis-213×axis-212 (Page L-block × Olmstead–Tukey corner) —
both joined two *signed* statistics, so the natural product space had
four quadrants (++, +-, -+, --) and three of them were "agreement"
quadrants while the off-diagonal pair encoded **direction conflict**
(one statistic says "up", the other says "down"). The whole point of
those compounds was to surface direction-conflict cells as a residual
diagnostic the operator should look at by hand.

The axis-218×axis-216 compound *cannot* carry that frame, because
axis-216 is an F-test on per-cohort means and is constitutively
unsigned. There is no "Buys–Ballot sign" to disagree with the
Hirsch–Slack sign about. So the joint enum collapses one axis: the
classifier just folds the two `bbDecisive AND hsDecisive` cells onto
the sign of `hsZ`, and the off-diagonal "decisive on one but not the
other" rows preserve the sign on the axis that was decisive. This is
the right design — pretending there is a four-way frame here would
introduce the same kind of false-symmetry that the bb-only and
hs-only "marginal" buckets are supposed to break.

## Orthogonality by construction — and why "by construction" matters

The phrase "mutually orthogonal by construction" appears in the
changelog and is doing a lot of work. The standard claim that two
statistics are "approximately uncorrelated under H0" is empirical
and depends on the data-generating process. The claim being made
here is structural: each test is *invariant* under the alternative
the other test isolates.

The verbatim statement:

> Buys-Ballot is INVARIANT under within-column detrending
> (blind to the within-cohort trend that Hirsch-Slack
> isolates); Hirsch-Slack is INVARIANT under any per-cohort
> additive shift (blind to the weekday mean structure that
> Buys-Ballot isolates). The four DECISIVE buckets cover
> the four independent corners of the (periodic, monotone)
> plane.

Spelling out the two invariance claims:

1. **Buys–Ballot's within-column detrending invariance.** The
   Buys–Ballot F-test compares the sum-of-squares of the seven
   weekday-cohort means against the within-cohort residual SS. If
   you subtract a within-cohort linear (or any monotone) trend
   from each cohort *before* computing the cohort means, the
   cohort means change only by their average detrending offset,
   which is identical across cohorts when the trend rate is the
   same — and the F-statistic is invariant under any common
   additive shift to all cohort means. Even when within-cohort
   trend rates differ, the cohort *means* still pick up only the
   midpoint values of those trends, not the slopes. So a series
   that is purely "every weekday cohort drifts up across the
   weeks at slope `m`" produces near-zero Buys–Ballot F because
   the seven cohort means are all approximately equal (each is
   the midpoint of the same linear cohort trend, which lives at
   the same week index across cohorts).

2. **Hirsch–Slack's per-cohort additive-shift invariance.** The
   within-cohort Mann–Kendall S statistic depends only on the
   *rank ordering* of observations within each cohort. Adding a
   constant to every observation in a cohort changes nothing
   about the rank ordering. So a series that is purely "weekday
   means differ but each cohort is internally a flat random
   walk around its weekday mean" produces near-zero `hsZ` because
   every within-cohort `S_g` is centered on zero.

The contrapositive is the operator's interpretation rule: when both
tests reject, it's because the series carries *both* a within-cohort
monotone trend *and* a weekday mean structure, and the two pieces of
information are independent in the structural sense — neither is a
shadow of the other. When only one rejects, you know which kind of
non-stationarity is present *and* which kind is absent. That
asymmetry is the actionable yield of the compound.

The v0.6.540 changelog also includes the test suite that operationalises
this claim:

> `test/dailytokenhirschslackseasonalkendall.test.ts`:
> +32 unit tests covering Phi/erf parity, within-season
> Mann-Kendall S identities, tie-corrected variance,
> per-cohort additive-shift INVARIANCE (the orthogonality-
> to-Buys-Ballot witness), reversal-negates-Z identity,
> builder filter & sort behaviour. Test suite total
> +32: 15689 -> 15721 tests, all green.

The "per-cohort additive-shift INVARIANCE" line is the structural
witness — the test constructs a series with a strong fabricated
weekday mean structure but zero within-cohort trend and asserts
`hsZ` is statistically zero.

## The live-smoke evidence

The `~/.config/pew/queue.jsonl` snapshot from 2026-05-05 produced this
verbatim output (sources beyond the local stack redacted to
`vsc-redacted` per repo policy):

```
$ pew-insights daily-token-hirsch-slack-seasonal-kendall \
    --sort hsAbsZDesc

pew-insights daily-token-hirsch-slack-seasonal-kendall
sources: 6 (shown 2)    tokens: 3,444,271,515
min-tokens: 1,000    min-tenure-days: 21    sort: hsAbsZDesc
dropped: 4 below min-tenure-days, 0 zero-variance,
         0 non-finite-fit

per-source Hirsch-Slack seasonal Kendall Z
source         firstDay    lastDay     tenure  hsS    hsVar     hsZ      hsTau    concSeasons  hsPValue   tokens
-------------  ----------  ----------  ------  -----  --------  -------  -------  -----------  ---------  -------------
claude-code    2026-02-11  2026-04-23  72       135     723.0   +4.9835  +0.4030  7/7          6.254e-07  3,442,385,788
vsc-redacted   2025-07-30  2026-04-20  265     -324   24772.7   -2.0522  -0.0663  0/7          4.015e-02      1,885,727
```

Two surviving sources, four dropped below the 21-day floor. The
`claude-code` row is dramatic: **`hsS = +135`, `hsZ = +4.98`,
`hsPValue = 6.3e-7`, `hsTau = +0.40`, `hsConcordantSeasons = 7/7`**.
Every one of the seven weekday cohorts trends up across the ten weeks
of tenure — this is a unanimous within-cohort up-drift that survives
stratification on weekday. Whatever ramp the source is on, it cannot
be attributed to weekday-mean structure: every weekday is moving up,
not just (say) Tuesdays through Thursdays. With `tokens =
3,442,385,788` over 72 days, that's an average of ~47.8M tokens/day
and a strict 7/7 within-cohort up-ranking.

The `vsc-redacted` row is the structural opposite: **`hsS = -324`,
`hsZ = -2.05`, `hsPValue = 4.0e-2`, `hsTau = -0.066`,
`hsConcordantSeasons = 0/7`**. Zero weekday cohorts trend up. The
aggregate is significant at α = 0.05, but the per-cohort effect size
is tiny (tau = -0.07) and the source has 38 weeks of tenure, so the
significance is purchased by sample size rather than effect magnitude.
This is the canonical "long-tail slow-decay" pattern: every weekday
cohort is drifting down, but only just barely. The token volume
(1,885,727 over 265 days) is two orders of magnitude smaller than
`claude-code`'s, consistent with a low-utilisation source decaying
asymptotically toward zero rather than disappearing in a regime
break.

The interpretation block in the changelog captures the contrast
exactly:

> The two sources have OPPOSITE seasonal-stratified
> trend signs: `claude-code` ramping in across all
> weekdays as it onboards; `vsc-redacted` slowly
> decaying across all weekdays in long tenure.

The four dropped sources are filtered to keep each weekday cohort at
≥ 3 observations, which is the Hirsch–Slack 1984 sec. 4
Normal-approximation floor. This is the floor the builder enforces
*before* the test runs; it is not a separate "small-sample" branch.

## How the compound bucket assignments fall out

Pushing the two surviving rows through the v0.6.542 classifier with
α = 0.05 and assuming axis-216 returns `bbDecisive = true` for
`claude-code` (likely, given the 7/7 concordance suggests both a
weekday mean rotation and a within-cohort trend) and `bbDecisive =
false` for `vsc-redacted` (likely, given the tiny per-cohort
effect):

- `claude-code`: `bbDecisive AND hsDecisive AND hsZ > 0`
  → `'weekday-and-up-cohort-trend'`
- `vsc-redacted`: `hsDecisive AND NOT bbDecisive AND hsZ < 0`
  → `'down-cohort-trend-only'`

Note these bucket assignments are conditional on what axis-216
actually emits for each source — the changelog excerpt only ships
axis-218's per-source numbers, not axis-216's. But the *structure*
of the bucket assignment is the point: even without axis-216 for
`claude-code`, we already know from the 7/7 concordance that the
weekday-mean structure cannot be the *whole* explanation, because
the within-cohort rank-trend signal is independently unanimous. And
the `vsc-redacted` row is too small in per-cohort tau to plausibly
generate a significant axis-216 F-test on its own (the F-test is
also a function of within-cohort variance, and `hsVar = 24772.7`
on 265 days suggests the within-cohort variance is high relative
to the between-cohort signal).

## Orthogonality evidence on the same data

The changelog also provides cross-axis orthogonality evidence using
the v0.5x.x axis-217 (Laplace centroid) numbers from the same
queue snapshot:

> ORTHOGONALITY EVIDENCE on the same data:
>
>   - vs axis-217 Laplace centroid: both sources have
>     significant axis-218 hsZ but axis-217 lapZ is
>     statistically zero for `vsc-redacted` (no centroid
>     displacement despite a real seasonal-stratified
>     rank-trend) -- demonstrating the magnitude-vs-rank
>     orthogonality predicted by the structural argument.
>   - vs axis-216 Buys-Ballot period-7 ANOVA: axis-218
>     finds within-cohort trend after stratifying away
>     the weekday mean structure that axis-216 isolates;
>     the two carry independent information about the
>     same series.

The first bullet is the more interesting one. `vsc-redacted` shows
significant `hsZ = -2.05` (axis-218) but axis-217's Laplace centroid
returns `lapZ ≈ 0`. The Laplace centroid is an L-1 first-moment
*magnitude* functional — it asks "is the mass concentrated late in
the tenure or early?" and shifts only when there is a clear ramp
or a late spike. A series that drifts down by tiny per-cohort
amounts but maintains roughly uniform mass distribution across the
tenure window will load on axis-218 (the per-cohort rank trends are
all weakly negative) but not on axis-217 (the centroid has barely
moved). This is the magnitude-vs-rank orthogonality the structural
argument predicted. It is also the operator's actionable diagnostic:
when axis-218 fires but axis-217 doesn't, the trend is real but
small in magnitude — a "creep", not a "ramp".

## What this changes about the dispatcher tick interpretation

For ticks where only one source is dispatching with high volume
(today, that's `claude-code`), the v0.6.542 compound is going to
emit `'weekday-and-up-cohort-trend'` until the source plateaus.
That bucket value is the operator-facing summary the rest of the
toolchain should read off. For long-tenure low-volume sources,
the bucket value will most often be `'down-cohort-trend-only'` or
`'no-evidence'`, and the operator reading it should know that
"down-cohort-trend-only" plus a tiny `hsTau` means
"asymptotic decay, not a regime break".

The 4-source drop today is also a feature of the floor, not a
limitation. With `min-tenure-days = 21` enforced at the builder
level, the compound classifier never sees rows where the per-cohort
n is too small for the Normal approximation. That keeps the
classification deterministic and prevents the bucket from flipping
on a single-week data update.

## Looking ahead — what the 1982 covariance-corrected variant will add

The changelog flags the working assumption explicitly:

> WORKING ASSUMPTION zero cross-season covariance.
> Cross-day positive autocorrelation INFLATES the
> true variance and so the simpler form's hsPValue
> is anti-conservative in that direction. Users
> wanting the conservative covariance-corrected
> p-value should consult the (forthcoming) Hirsch-
> Slack-Smith 1982 axis.

Daily token series typically have *strong* positive cross-day
autocorrelation — Wednesday's volume is correlated with Tuesday's
because the underlying user behaviour is autocorrelated at the
session level. So the v0.6.540 axis-218 p-values are anti-conservative
in the direction of "easier to reject H0 than the truth would
warrant". For `claude-code` at `hsPValue = 6.3e-7` this doesn't
matter — even three orders of magnitude of anti-conservatism
leaves the rejection emphatic. For `vsc-redacted` at `hsPValue =
4.0e-2` it could matter, and the source is borderline enough that
the covariance-corrected variant might bump it back into "marginal"
territory. That's the right fix and the changelog correctly flags
it as a separate axis rather than retrofitting the variance into
v0.6.540.

The references block is the standard one for this family:

> - Hirsch, R. M. & Slack, J. R., "A nonparametric trend
>   test for seasonal data with serial dependence",
>   *Water Resources Research* 20(6) (1984), pp. 727-732.
> - Mann, H. B., "Nonparametric tests against trend",
>   *Econometrica* 13(3) (1945), pp. 245-259.
> - Kendall, M. G., *Rank Correlation Methods*, 4th ed.,
>   Griffin 1975, sec. 4.2 (variance with tie
>   correction).
> - Abramowitz, M. & Stegun, I. A., *Handbook of
>   Mathematical Functions*, NBS 1964, eq. 7.1.26.

It is worth noting that Hirsch–Slack 1984 was developed for
*water-quality time series* with seasonal contamination — the
prototypical use case is monthly stream chemistry data where
spring runoff dominates the seasonal signal and the analyst wants
the within-cohort trend after stratifying out the seasonal cycle.
The application to weekday-cohort token volumes is structurally
identical: the day-of-week is the season, the weekly token spike
on weekdays-vs-weekend is the seasonal mean structure, and the
question "is utilisation drifting up after we account for the
weekly cycle?" is exactly the question Hirsch–Slack was designed
to answer.

## The takeaway

v0.6.542 is the first compound in the pew-insights battery where
the orthogonality of the two component axes is a structural
consequence of how each test is *defined*, rather than an
empirical observation about how their outputs correlate on
typical inputs. The six-bucket scheme is the right shape for
joining a signed rank-trend test with an unsigned mean-structure
F-test, and abandoning the four-quadrant direction-conflict
frame from the earlier compounds is a deliberate design choice
driven by the unsigned nature of axis-216, not an oversight.

The `claude-code` 7/7 concordant-seasons result with `hsZ = +4.98`
is the single decisive signal in the live-smoke output, and the
`vsc-redacted` marginal `hsZ = -2.05` with `hsTau = -0.066` is the
canonical example of why the eventual covariance-corrected
Hirsch–Slack–Smith 1982 axis is going to matter for long-tenure
low-volume sources where the simpler 1984 form's anti-conservatism
sits on the boundary of significance.

Two axes shipped in one tick, both green at 15737/15737 tests, with
the structural orthogonality witness wired into the test suite as
a per-cohort additive-shift invariance assertion. That is the
right shape for a pew-insights ship: the math is documented in the
changelog, the live-smoke output is verbatim from the local
queue, the orthogonality claim is operationalised in tests, and
the next axis in the family (Hirsch–Slack–Smith 1982) is already
flagged as a known follow-up rather than left as an unmentioned
limitation.
