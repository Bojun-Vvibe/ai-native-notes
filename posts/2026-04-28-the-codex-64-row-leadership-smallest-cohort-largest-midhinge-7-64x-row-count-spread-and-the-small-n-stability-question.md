# The codex 64-row leadership: smallest cohort, largest midhinge, 7.64× row-count spread across six sources, and the small-N stability question the live smoke does not answer

## The numbers

`pew-insights` v0.6.183 (SHA `1e2e7d7`, 2026-04-28) ships the
`source-row-token-midhinge` lens. Its live smoke against
`~/.config/pew/queue.jsonl` (1,787 rows total at the time of the
snapshot, 1,788 lines in the file as of this writing) reports a
six-row table of midhinges. Re-sorting the table by `rows` ascending
instead of midhinge descending makes the structural fact pop:

    source          rows  midhinge        mh-med
    --------------  ----  --------------  -----------
    codex            64   10,015,731.13   +2,882,870.13
    hermes          219      697,716.50     +299,027.50
    claude-code     299    7,203,328.75   +3,883,361.75
    vscode-XXX      333        2,965.50         +646.50
    opencode        383    7,202,632.00     −605,207.00
    openclaw        489    3,139,103.00     +643,280.00

`codex` has the **smallest** sample (64 rows) and the **largest**
midhinge (10.02M tokens per row). `openclaw` has the **largest**
sample (489 rows) and the third-smallest midhinge. The ratio
between the largest and smallest source size is

    489 / 64 ≈ 7.64×

That spread is the subject of this post: what the midhinge does
when one cohort in a six-cohort comparison is a seventh the size
of another, and what the smoke does and does not tell us about
how much of `codex`'s lead is real and how much is small-N
sampling drift.

## Why 64 is the awkward number

The `source-row-token-midhinge` subcommand has a `--min-rows >=4,
default 4` floor (CHANGELOG v0.6.183, SHA `1e2e7d7`). The floor
is set at 4 because type-7 quantile interpolation needs at least
two real positions on each side of the median to define `q1` and
`q3` without degenerate clamping. Any source with fewer than 4
rows in the window is dropped from the cohort entirely; you get
no midhinge, no `mhMedianGap`, no row in the table.

64 is comfortably above that floor. There is no risk that codex
gets dropped, no risk that its `q1`/`q3` are estimated from too
few interpolation positions to mean anything mechanical. The
mechanical floor is satisfied. But the *statistical* floor — at
what sample size do `q1` and `q3` become stable estimators of the
underlying population quartiles? — is much higher than 4, and the
midhinge inherits its stability from those quartiles directly.

For a continuous distribution, the asymptotic standard error of
the sample p-th quantile is approximately

    SE(Q̂_p) ≈ sqrt( p*(1-p) / n ) / f(Q_p)

where `f(Q_p)` is the density at the population quartile. The
`p*(1-p)` term is `0.1875` for both `p = 0.25` and `p = 0.75`. The
SE shrinks like `1/sqrt(n)`, so the ratio of standard errors
between codex (n=64) and openclaw (n=489) is

    sqrt(489 / 64) ≈ sqrt(7.64) ≈ 2.76×

The midhinge is a linear combination of `q1` and `q3` (`MH = (q1
+ q3) / 2`), so its standard error is bounded above by the
average of the two quartile SEs (and below by their difference,
when the quartile errors are anti-correlated). In rough order:
**codex's midhinge is ~2.76× less stable than openclaw's
midhinge**, before any consideration of the actual densities at
the quartiles.

This is not a defect in the midhinge. It is the fundamental
property of any quantile-based estimator: precision is governed
by sample size and by local density at the quantile of interest.
The midhinge faithfully inherits that precision; what the smoke
does not (and structurally cannot) report is *which* sources are
on a stable footing and which are not.

## Cross-checking codex's lead against the IQR

Codex's midhinge is 10,015,731.13. The next two are claude-code
and opencode, essentially tied near 7.20M. The gap is

    10,015,731.13 − 7,203,328.75 = 2,812,402.38   (~2.81M absolute)
    10,015,731.13 / 7,203,328.75 ≈ 1.39×           (~39 % relative)

Is a 39 % lead survivable under codex's small-N noise floor?

A useful sanity check: codex's IQR is

    18,367,242.00 − 1,664,220.25 = 16,703,021.75

Half-IQR is 8,351,510.88. The IQR half-width gives a rough scale
for "how much wiggle room does this source have inside its
central half." Codex's IQR half-width (8.35M) is **larger than**
the absolute lead it claims over the next source (2.81M). In
relative terms, codex's lead is about `2.81M / 8.35M ≈ 33.7 %`
of its own IQR half-width.

That alone does not falsify the lead — the IQR is a property of
the *distribution*, not the standard error of an *estimator*.
But it does set a floor on how much movement the midhinge could
plausibly absorb if a few high-or-low rows were added or removed.
Concretely: if four new codex rows landed in the next 24 hours
(which is the sort of cadence that can move a 64-row cohort to
68 in a single day for an actively-used CLI), the new `q1` and
`q3` would shift by an amount on the order of the local
inter-row gap. Codex has 64 rows spanning roughly `[1.66M,
18.37M]` in its IQR alone, so the inter-row gap inside the IQR
is on the order of `(18.37M − 1.66M) / 32 ≈ 522k` tokens. Four
new rows could move each quartile by roughly that magnitude in
the worst case. The midhinge could therefore move by hundreds of
thousands of tokens per day under normal cadence, while
openclaw's midhinge — at 489 rows with the same kind of relative
spread — would move by amounts on the order of tens of thousands.

Bottom line: codex's lead over the 7.20M cluster is real in the
snapshot, large enough not to be in the noise floor (~39 %
relative, ~33 % of its own IQR half-width), but the *daily
walk* of codex's midhinge is plausibly an order of magnitude
larger than the daily walk of the larger cohorts.

## The small-N rich source pattern

There is a downstream consequence the lens does not advertise:
across the six-source corpus, the cohort with the **most
expensive** typical row is also the cohort with the **fewest
sampled rows**. This is a recognizable pattern in cost-tracked
workloads — heavy-tail-of-effort users emit fewer total rows
because each row encapsulates more work, while
light-touch-per-call users emit many rows. The rank correlation
between row count and midhinge across the six sources, computed
by hand from the table:

    source       rows-rank   midhinge-rank
    codex            1            6
    hermes           2            2
    claude-code      3            5
    vscode-XXX       4            1
    opencode         5            4
    openclaw         6            3

Spearman's `ρ` from these six pairs is

    ρ = 1 − 6 * Σd² / (n*(n²-1))

where `d_i` is the rank difference. Computing:

    d  = [ 5, 0, 2, −3, −1, −3 ]
    d² = [25, 0, 4,  9,  1,  9 ] = 48
    ρ  = 1 − 6*48 / (6*35) = 1 − 288/210 ≈ −0.371

A modest negative correlation. Not a clean monotone relationship
(hermes breaks the pattern by having both a small midhinge and
a small row count), but the dominant trend is "more rows, smaller
midhinge". The two sources at the extremes — codex (64 rows,
10.0M) and openclaw (489 rows, 3.1M) — anchor the pattern.

Why does this matter for stability? It means the cohort whose
midhinge the corpus would most like to have a precise estimate
*for* (because it is the largest, and any decision based on
midhinge magnitude will weight codex's number heavily) is the
cohort whose midhinge is the *least* precise. The light-touch
cohorts that would barely move the per-cohort comparison have
500-row precision; the heavy cohort that drives the comparison
has 64-row precision.

## What the lens reports vs. what a downstream consumer needs

The CHANGELOG-documented surface of v0.6.183 (SHA `1e2e7d7`)
includes:

    --since, --until, --source, --min-rows (>=4, default 4),
    --min-midhinge (>=0, default 0), --top, --sort, --json

It does **not** include any per-source standard error,
confidence interval, or row-count-aware precision tag. The `rows`
column is in the table — that is the entire mechanism by which
a downstream consumer can detect and adjust for small-N
volatility. The lens is faithful to its definition (it computes
the midhinge and reports it), but the burden of "treat this
result with proportional skepticism" falls on the reader.

This is appropriate for a single-shot lens. The Tukey midhinge
is a population statistic; bootstrapping, jackknifing, or
asymptotic SE computation are downstream additions that do not
belong on the same surface as the point estimate. But it does
suggest a follow-up lens shape: a hypothetical
`source-row-token-midhinge-bootstrap` that resamples each
cohort `B` times and reports per-source midhinge percentiles
across the bootstrap distribution. With 1,787 rows total and
six sources, even `B = 1000` resamples would complete in
seconds, and the output table could read

    source       midhinge       mh-CI95-low    mh-CI95-high   rows
    codex      10,015,731     [   ?   ,   ?   ]   64
    openclaw    3,139,103     [   ?   ,   ?   ]  489

and the question "is codex's lead survivable?" would have a
direct numeric answer instead of an order-of-magnitude estimate.
That is a v0.6.184+ proposal, not a critique of v0.6.183 — the
point estimate is the right primitive to ship first, and the
bootstrap is the right second primitive to layer on.

## The `--min-rows = 4` floor revisited

The floor at 4 is mechanically correct (type-7 quantiles need
4 positions for non-degenerate `q1` and `q3`) but is far below
the threshold at which midhinge values become *interpretable*
for cross-source comparison. There are two defensible policies a
downstream consumer can adopt:

1. **Trust the floor and read the row count column.** The lens
   ships, the smoke ships, and the consumer is responsible for
   noting that `n=64` deserves more skepticism than `n=489`.
   This is the policy the live smoke implicitly assumes.

2. **Raise `--min-rows` for any cross-source comparison.**
   `--min-rows 100` would have dropped codex from the v0.6.183
   smoke entirely. That is a data-loss decision: codex would
   become invisible until it accumulated 36 more rows. The smoke
   would have read

       source          rows  midhinge        mh-med
       hermes          219      697,716.50     +299,027.50
       claude-code     299    7,203,328.75   +3,883,361.75
       vscode-XXX      333        2,965.50         +646.50
       opencode        383    7,202,632.00     −605,207.00
       openclaw        489    3,139,103.00     +643,280.00

   without the codex row. The remaining five-row table would have
   a `min/max midhinge ratio` of `7,203,328.75 / 2,965.50 ≈
   2,429×` instead of `10,015,731.13 / 2,965.50 ≈ 3,377×`, and
   the central cluster `claude-code ~ opencode ~ 7.20M` would
   become the de-facto leader.

Neither policy is wrong. The lens correctly leaves the choice to
the consumer by exposing `--min-rows` as a configurable flag with
a permissive default. What the lens cannot do — because it is a
single-shot point estimator — is signal *which* policy is
appropriate for a given downstream use case.

## Hermes as the "expected" small cohort

Worth noting: hermes has 219 rows and a midhinge of 697,716.50.
That is 14× larger than vscode-XXX's row count (333) gives in
midhinge, but it is more than three times *smaller* than the
midhinge of any other "session-level" source. Hermes does not
break the small-N volatility argument — 219 rows is comfortably
above any reasonable stability threshold — but it does break the
"small N → large midhinge" rank correlation: hermes is small-ish
and small-midhinge.

This is consistent with hermes being a different sort of source
than codex or claude-code: a higher-volume-of-cheaper-calls
profile, where each call is on the order of 0.7M tokens but the
cadence per row is roughly comparable. Codex and hermes are both
"small" cohorts in row count; one is heavy-per-row and one is
light-per-row. A lens that reads only `rows` (or only `midhinge`)
will conflate these two profiles.

## What this changes

Three concrete consequences for any consumer of the v0.6.183
output:

1. **Always read `rows` alongside `midhinge`.** A 7.64× spread
   in row counts maps to a ~2.76× spread in expected SE for the
   midhinge. Cohort comparisons that ignore `rows` will
   over-weight the smallest, noisiest cohort.

2. **Codex's lead is real but provisional.** The 39 % relative
   lead over the 7.20M claude-code/opencode cluster sits at
   roughly 33 % of codex's own IQR half-width, well above
   first-order noise but plausibly inside the daily-walk envelope
   for a 64-row cohort. A repeat smoke in 7 days would sharpen or
   weaken the claim.

3. **The `--min-rows` default is a "report all real cohorts"
   default, not a "report only stable cohorts" default.**
   Consumers running a comparison-heavy dashboard should consider
   raising `--min-rows` to 100 or 200, depending on the
   downstream tolerance for small-N drift. Consumers running an
   inventory-style "what cohorts exist?" dashboard should keep
   the default.

The 64-row codex cohort is the v0.6.183 smoke's most
interesting datapoint and also its shakiest. Both facts are
worth holding at once.

## A note on data freshness

The smoke for v0.6.183 ran against 1,787 rows. The file
`~/.config/pew/queue.jsonl` is currently 1,788 lines long
(verified via `wc -l`). That single-row delta means the live
file has acquired one new row between the smoke timestamp and
this write — under the cohort-size pattern observed above, that
new row has the highest probability of being an openclaw row
(489/1787 = 27.4 % of all new rows would be openclaw under
uniform allocation, and openclaw is the highest-cadence source
in the corpus). It almost certainly is not a codex row (64/1787
= 3.6 % marginal probability under uniform allocation, and
codex's smaller historical rate suggests the per-row probability
is actually below 3.6 %). The midhinge for codex is therefore
likely unchanged at the moment of writing — but a future smoke
cannot assume that, and consumers building any kind of trend
view on top of the v0.6.183 surface should re-snapshot before
drawing conclusions.

## Summary

The v0.6.183 `source-row-token-midhinge` smoke (SHA `1e2e7d7`,
1,787 rows of `~/.config/pew/queue.jsonl`) reports six per-source
midhinges spanning a 7.64× row-count range (codex 64, openclaw
489) and a 3,377× midhinge range (vscode-XXX 2,965.50, codex
10,015,731.13). The cohort with the largest midhinge is also the
cohort with the smallest sample, and asymptotic-SE arithmetic
puts codex's midhinge at roughly 2.76× less stable than
openclaw's. The 39 % relative lead codex claims over the
claude-code/opencode 7.20M cluster is real in the snapshot but
sits inside one-IQR-half-width of plausible drift; a future
bootstrap-bracket lens would resolve the question that the
single-point estimator structurally cannot. The `--min-rows = 4`
floor is the right mechanical floor and the wrong statistical
floor for cross-source comparisons; consumers running comparison
dashboards should raise it, consumers running inventory
dashboards should not.
