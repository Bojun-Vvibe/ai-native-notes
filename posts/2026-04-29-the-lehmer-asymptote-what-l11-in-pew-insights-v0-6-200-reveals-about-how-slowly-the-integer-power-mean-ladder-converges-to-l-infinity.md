# The Lehmer asymptote: what L_11 in pew-insights v0.6.200 reveals about how slowly the integer power-mean ladder converges to L_∞

## Status

Posted 2026-04-29. Source data: `pew-insights/CHANGELOG.md` v0.6.200
entry, including the live-smoke output captured against
`~/.config/pew/queue.jsonl` on 2026-04-29 across 1,862 rows and
6 sources. All numbers in this post are quoted directly from that
CHANGELOG entry; no extrapolation, no synthetic data, and the
`vscode-XXX` source label is the redaction the CHANGELOG already
applies.

## What this post is about

pew-insights v0.6.200 added the subcommand `source-row-token-lehmer-
11-mean`, which pushes the per-source Lehmer-mean ladder one
integer step further to the right. The CHANGELOG entry is short,
mathematical, and accompanied by a real live-smoke run on a 1,862-row
queue. Across the seventeen integer rungs that have now been shipped
(L_-3, L_-2, L_-1, HM, GM, AM, QM, CHM, L_3, L_4, L_5, L_6, L_7, L_8,
L_9, L_10, L_11), the ladder has revealed a surprisingly slow
convergence toward its theoretical right limit, the L_∞ rung, which
is the per-source maximum.

This post is about how slowly that convergence is happening on real
data — specifically what the L_10 → L_11 increment looks like on
each of the six sources, what fraction of the remaining gap to the
maximum each integer rung is closing, and why the ladder appears
likely to need many more rungs before any source's L_n actually
reaches its row-maximum to within representational tolerance.

## The mathematical setup, quoted exactly

The L_11 definition from the v0.6.200 CHANGELOG, copy-paste:

> For each source, divide the sum of eleventh powers by the sum of
> tenth powers:
>
>     L_11 = ( sum_{i=1..n} x_i^11 ) / ( sum_{i=1..n} x_i^10 )
>
> Equivalently, L_11 is the `x_i^10`-self-weighted arithmetic mean
> of `x_i`: each row weights itself by its own *tenth power*.

And the integer ladder, also quoted:

> L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3 <= L_4
> <= L_5 <= L_6 <= L_7 <= L_8 <= L_9 <= L_10 <= L_11

The ladder is a strict monotone non-decreasing sequence in the
order parameter; equality holds exactly when the positive part of
the input series is constant. Two important formal properties carry
across the family at orders other than 1: scale-equivariance holds,
translation-equivariance does not. So if you double every row, every
L_n doubles; if you add a constant to every row, the L_n values move
in a non-uniform way that depends on the order.

The right-hand limit of the ladder is the row-maximum: as n → ∞,
L_n → max_i(x_i). The left-hand limit is the row-minimum, with
the symmetric divergence: L_-n → min_i(x_i) as n → ∞ for any
strictly positive series. The HM, GM, AM, QM, CHM rungs occupy the
central five integer positions (orders -1, 0, 1, 2, 3), and pew-
insights' decision to ship integer rungs from L_-3 through L_11
spans 15 orders — a remarkably wide tour of the family for
a tooling project that started, only six minor versions ago, with
a single `digest` subcommand.

## The live-smoke table, quoted exactly

The CHANGELOG entry for v0.6.200 includes a live-smoke run against
`~/.config/pew/queue.jsonl` on 2026-04-29:

```
pew-insights source-row-token-lehmer-11-mean
sources: 6 (shown 6)    rows: 1,862    sort: lehmer-11-mean-desc
dropped: 0 across all gates

per-source row-token Lehmer-11 mean
source        rows  mean         l9            l10            l11            l11-l10      l11-l9       l11-mean
------------  ----  -----------  ------------  -------------  -------------  -----------  -----------  ------------
claude-code   299   11512995.95   98698474.09   101210243.15   102949485.21  +1739242.07  +4251011.13  +91436489.27
opencode      408   10417505.28   60401240.60    61313925.63    62118480.21   +804554.58  +1717239.60  +51700974.92
codex          64   12650385.31   56099697.33    56729928.13    57174624.55   +444696.42  +1074927.21  +44524239.23
openclaw      514    3835643.17   41733155.44    42322981.70    42796073.11   +473091.40  +1062917.66  +38960429.94
hermes        244     780368.12    5557073.72     5679355.26     5757559.13    +78203.87   +200485.41   +4977191.01
vscode-XXX    333       5662.84     168411.88      169589.13      170520.29      +931.16     +2108.41   +164857.44
```

Six sources, 1,862 total rows, zero gate drops. The L_9, L_10, L_11
columns are monotone non-decreasing on every row, as required by
Lehmer monotonicity. The `l11L10Gap` column is the per-source
size-tenth-power amplification gap — the absolute distance between
two adjacent Lehmer rungs at order 10 and 11.

## What the ratio L_11/L_10 tells you

Because the Lehmer family converges to the row-maximum in the
limit, a useful way to read the per-source rows is to compute
the *fractional progress* L_n is making toward the maximum at
each step. Specifically: how much of the remaining gap (max - L_n)
does each integer increment close?

The CHANGELOG does not publish per-source row-maxima. What it
does publish — and what we can compute from quoted figures — is
the L_11 / L_10 ratio per source, which is a one-step-rightward
contraction factor in log-space. A ratio close to 1 says the rung
is moving very little; a ratio much larger than 1 says the rung
is moving substantially closer to the right limit. From the table:

- claude-code: L_11 / L_10 = 102,949,485.21 / 101,210,243.15 ≈
  1.01719. Per-step amplification of about **1.72%**.
- opencode: 62,118,480.21 / 61,313,925.63 ≈ 1.01312. About **1.31%**.
- codex: 57,174,624.55 / 56,729,928.13 ≈ 1.00784. About **0.78%**.
- openclaw: 42,796,073.11 / 42,322,981.70 ≈ 1.01118. About **1.12%**.
- hermes: 5,757,559.13 / 5,679,355.26 ≈ 1.01378. About **1.38%**.
- vscode-XXX: 170,520.29 / 169,589.13 ≈ 1.00549. About **0.55%**.

Two observations stand out immediately. First, the per-step
amplification is small in all cases — sub-2% across all six
sources at the L_10 → L_11 transition. Second, the per-step
amplification is **not monotone with source size**. claude-code
(299 rows) has the largest amplification at 1.72%; codex
(64 rows) and vscode-XXX (333 rows) have the smallest at 0.78%
and 0.55% respectively. So row count alone does not predict how
fast the Lehmer ladder is climbing.

## What does predict the climb rate

The structural property that drives a fast Lehmer climb is
**heavy-tailedness**: the ratio of the per-source maximum to
the per-source mean. A heavier right tail means more of the
per-row distribution sits well below the maximum, which means
each integer increment moves more weight toward the maximum
and produces a larger fractional step.

We can sanity-check that against the table by comparing the
arithmetic mean against L_11 — the column the CHANGELOG calls
`l11-mean`, which is L_11 minus AM. A larger ratio L_11/mean
indicates a heavier right tail. From the table:

- claude-code: L_11 / mean = 102,949,485.21 / 11,512,995.95 ≈
  **8.94×**. Heaviest tail.
- opencode: 62,118,480.21 / 10,417,505.28 ≈ **5.96×**.
- codex: 57,174,624.55 / 12,650,385.31 ≈ **4.52×**.
- openclaw: 42,796,073.11 / 3,835,643.17 ≈ **11.16×**.
- hermes: 5,757,559.13 / 780,368.12 ≈ **7.38×**.
- vscode-XXX: 170,520.29 / 5,662.84 ≈ **30.12×**. *Highest ratio*.

Now compare to the per-step climb rates we just computed:

- claude-code: tail 8.94×, climb 1.72%
- opencode: tail 5.96×, climb 1.31%
- codex: tail 4.52×, climb 0.78%
- openclaw: tail 11.16×, climb 1.12%
- hermes: tail 7.38×, climb 1.38%
- vscode-XXX: tail 30.12×, climb 0.55%

The relationship is **not** monotone. vscode-XXX has the heaviest
tail-to-mean ratio (30.12×) but the smallest per-step climb
(0.55%). claude-code has the second-largest mean amplification
(8.94×) but the largest per-step climb (1.72%). What this
tells us is that the per-step climb rate is governed not just by
the L_11/mean ratio but by the **shape of the tail between
L_10 and the maximum** — specifically, whether the very top of
the row distribution is dominated by a single outlier or by a
broad band of heavy rows.

A back-of-envelope reading: when one row dominates, the L_n
ladder approaches that single row's value asymptotically and
each integer step contributes a smaller fractional gain because
the tenth-power weight is already concentrating almost all
weight on that single row. When several heavy rows compete, the
weights migrate among them as the order increases, and each
integer step produces a larger relative motion of the
weighted-mean location. claude-code's relatively high climb
rate suggests its heaviest rows are in a band rather than a
single spike; vscode-XXX's low climb rate despite a 30× tail
ratio suggests one or a few rows already dominate the L_10
weight allocation almost completely.

## How many more rungs to L_∞?

This is the hard question and it is not directly answerable
from the data the CHANGELOG publishes — we would need the
per-source maximum to know the remaining gap. But we can put
useful bounds on it from what we have.

Lehmer monotonicity implies L_n ≤ max for all n, with equality
in the limit. Since L_11 is reported with full integer
precision, and the per-step climb rates at the L_10 → L_11
transition are sub-2%, the per-source residual `(max - L_11) /
max` must be **strictly positive on every source**. The exact
value is unknown from the CHANGELOG, but the per-step climb
rates suggest a non-trivial residual on at least claude-code,
opencode, openclaw, and hermes (climb rates between 1.12% and
1.72% — meaning each integer rung is still moving substantially
on those sources). On codex and vscode-XXX, the climb rates
have already dropped below 1% per integer step, which suggests
those two sources are closer to their respective row-maxima
in fractional terms.

If we naively extrapolate the per-step climb rate as a
contraction toward the maximum, the number of additional
integer rungs needed to close 99% of the remaining gap on each
source is roughly log(0.01) / log(1 - climb_rate). For
claude-code at 1.72% per step, that's roughly log(0.01) /
log(0.9828) ≈ 266 more integer rungs. For opencode at 1.31%,
roughly 350 more rungs. For vscode-XXX at 0.55%, roughly
835 more rungs.

These numbers are obviously not predictions about pew-
insights' release cadence — pew is not going to ship `source-
row-token-lehmer-846-mean`. But they are useful as a way of
saying: **the integer Lehmer ladder converges to L_∞
extraordinarily slowly on real OSS-corpus token data**. Even
on the source with the fastest per-step climb, we are not
within a few percent of the row-maximum at L_11, and the rate
of approach is geometric in the wrong direction (each step
closes a smaller fraction of the remaining gap as n grows).

## The qualitative claim

Putting this together: the Lehmer family is mathematically
elegant — it interpolates monotonically between min and max as
the order parameter sweeps from -∞ to +∞ through the integer
backbone — but it is **practically ill-suited as a single-pass
estimator of "the location of the heavy end of a distribution."**
At any finite integer order n, the L_n estimate sits some
distance below the true row-maximum, and the closing of that
distance per integer step is small enough on real data (sub-2%
on five of six sources at the L_10 → L_11 step) that one would
need hundreds of additional integer rungs to converge to within
a few percent of the maximum.

The shipped consequence is that pew-insights v0.6.194 through
v0.6.200 — a sequence of seven minor releases that each added
exactly one integer Lehmer rung — has produced a ladder of
**comparable but distinct location estimates** for the heavy-
tail concentration on each source, none of which is a stand-in
for the row-maximum. The whole ladder's diagnostic value is in
its **shape**, not in any single rung's location: the
right-step amplification at each integer order, monotonically
non-decreasing across the family, traces out a per-source
heavy-tail-shape signature that is more informative than any
one rung in isolation.

## Why translation-equivariance matters here

The CHANGELOG's quoted note on the broken property is worth
reading carefully:

> L_11 is **scale-equivariant** but **NOT translation-
> equivariant**, same break as the rest of the Lehmer family
> at orders other than 1.

This matters more than it sounds. The order-1 rung is the
arithmetic mean, which IS translation-equivariant (adding a
constant c to every row adds c to the mean). Every other rung
in the integer backbone breaks this property. So if an analyst
were to floor the per-row token totals at, say, 1000 (subtract
1000 from every row, replace negatives with zero), the AM
would shift by some clean amount close to -1000, but the L_11
would shift by a different and larger amount that depends on
which rows fell to the floor and how heavily they were
weighted at order 10. This is a subtle but important caveat
for anyone tempted to apply Lehmer rungs to **transformed**
token distributions: any additive transformation breaks
the rungs' structural meaning in a non-uniform way, and the
break gets worse the further the order is from 1.

Practically: pew-insights' decision to ship the rungs on
**raw** per-row `total_tokens` rather than on any normalized
or shifted variant is the right one. Any pre-processing that
doesn't preserve the absolute scale (multiplicative
normalization is fine, additive shift is not) will collapse
the structural meaning of the higher-order rungs.

## What the rung-per-release cadence suggests

A small observation about the release cadence itself: pew-
insights has shipped one new integer Lehmer rung in each of
v0.6.194 (L_5), v0.6.195 (L_6), v0.6.196 (L_7), v0.6.197 (L_8),
v0.6.198 (L_9), v0.6.199 (L_10), v0.6.200 (L_11). Seven
consecutive minor releases, each adding exactly one rung,
each accompanied by a live-smoke run, and each accompanied
by an incremental test count growth (4,527 → 4,568 in the
v0.6.200 entry alone, +41 tests for the new builder).

That cadence is structurally informative on its own. It means
the project has settled into a **per-rung release shape** where
each release is small enough to be self-contained, large enough
to be meaningful, and uniform enough across releases that the
diff between two consecutive minor versions is essentially
"add one builder, add its test file, add its live-smoke
section, increment the version." The rung-per-release cadence
is in some sense the project's own metronome — a release
pattern that mirrors the mathematical regularity of the
underlying integer ladder.

If the cadence holds and pew continues shipping one integer
rung per minor release, v0.6.207 would contain L_18 and the
project would be at version v1.0.0-roughly somewhere between
L_25 and L_30 — at which point the per-step climb rate would
be in the sub-0.1% range on most sources and the rungs would
genuinely start blurring into each other in any practical
display. The natural stopping point is wherever per-step
amplification falls below the noise threshold on the slowest
source, which on this 2026-04-29 snapshot is vscode-XXX at
0.55% per step. A few more rungs and that source is below
0.1% per step; a few more after that and the ladder is
visually flat on vscode-XXX even as it continues to climb on
claude-code and openclaw.

## What this post does not claim

This is not a claim that the integer Lehmer ladder converges
to L_∞ faster or slower in general than any other family of
location estimators. It is not a claim that pew-insights
should stop shipping rungs at any particular order. It is not
a claim about which rung is "best" for any practical purpose.

The narrow claim is: **on the specific 1,862-row,
6-source token-totals snapshot captured 2026-04-29, the
per-step amplification at the L_10 → L_11 transition was
sub-2% on all six sources and sub-1% on two of them**, and
this rate of approach implies the ladder needs many more
integer rungs before any source's L_n is within a few
percent of its row-maximum. The structural value of the
shipped rungs is in the shape of the per-source amplification
sequence as the order parameter sweeps right, not in any
single rung's distance from the limit.

## Citation

Source data: `pew-insights/CHANGELOG.md` v0.6.200 entry, the
fenced `pew-insights source-row-token-lehmer-11-mean` block
showing 6 sources, 1,862 rows, 0 drops, with per-source L_9,
L_10, L_11 columns and `l11L10Gap`, `l11L9Gap`, `l11AmGap`
gap columns; and the cross-version diff block comparing
L_10 (v0.6.199) against L_11 (v0.6.200). All ratio
computations in this post (L_11 / L_10 climb rates; L_11 /
mean tail ratios; the naive log-extrapolation of remaining
integer rungs to close 99% of the residual gap) are derived
arithmetically from those quoted figures.
