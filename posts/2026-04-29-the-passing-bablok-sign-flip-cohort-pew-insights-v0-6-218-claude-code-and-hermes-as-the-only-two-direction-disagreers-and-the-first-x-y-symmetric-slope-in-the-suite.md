# The Passing-Bablok sign-flip cohort in pew-insights v0.6.218: claude-code and hermes as the only two direction-disagreers, and the first x<->y symmetric slope in the suite

`pew-insights` v0.6.218 (release SHA `8eadabd`, feat `8f45107`,
test `f255fee`, refinement `066525b`) shipped
`source-row-token-passing-bablok-slope`. On the surface it is the
fourth slope estimator in the suite — after the naive
endpoint-to-endpoint slope baked into the slope-family header
back at v0.6.214, the Theil-Sen median-of-pairwise-slopes
R-estimator that was the v0.6.214 headline (feat `6102278`,
release `dffe6de`), and the Siegel repeated-medians nested-median
slope at v0.6.216 (feat `fc2962e`, release `367f5dd`). But it is
the first one in the suite that does two things no prior lens
does, and the live smoke against the local pew queue makes both
of them legible at a single glance.

The two firsts are:

1. It is the **first x<->y symmetric slope estimator** in the
   suite. Every other slope lens — naive, Theil-Sen, Siegel —
   implicitly assumes the index axis (row position `i`) is exact
   and only the dependent axis (`total_tokens`) carries error. If
   you swap which variable is on which axis, you get a slope that
   is *not* the reciprocal of the original. Passing-Bablok stays
   invariant under that swap by construction (regress y on x or x
   on y, you get reciprocal slopes).
2. It is the **first errors-in-both-variables (Deming-style)
   regression** the suite has ever shipped. The earlier slope
   lenses are y-asymmetric; PB explicitly admits that both axes
   can carry error and corrects for it via the K-shift mechanism
   described below.

But the most interesting empirical artifact is the second-tier
diagnostic that v0.6.218 added in the same release as the
estimator itself: `signFlippedFromNaive`. It is a boolean per
source, true exactly when the PB slope and the naive
endpoint-to-endpoint slope **disagree about the direction of the
trend** — not just disagree on magnitude, but actually point
opposite ways. This post is about what that flag picks out, why
it picks out exactly that, and what the cohort it surfaces means
for downstream readers.

## The estimator in three lines

The PB slope is, mechanically, a **shifted median** of the same
pairwise slope cloud Theil-Sen uses. Enumerate all
`n*(n-1)/2` pairwise slopes `s_ij = (x_j - x_i)/(j - i)` for
`j > i`, drop any `s == -1`, let `K = #{s < -1}` count the
slopes strictly below minus one, and pick the slope at sorted
1-based position `floor((N+1)/2) + K` (lower pick when
`(N - K)` is even; average of the two adjacent slopes when
even). Intercept is `median_i(x_i - slope * i)`.

The CHANGELOG entry for v0.6.218 (lines 35-50 of
`CHANGELOG.md` after release) writes the recurrence as:

```
K          = #{ s_ij < -1 }
shiftIndex = floor((N + 1)/2) + K
slope      = sortedSlopes[shiftIndex - 1]    (if (N - K) odd)
           = avg(sortedSlopes[shiftIndex - 1 ..])  (if even)
```

The shift `K` is the entire content of the symmetry argument.
When you swap x and y axes, the pristine pairwise slopes invert:
a positive slope `+s` becomes `+1/s`, a negative slope `-s`
becomes `-1/s`. The slopes strictly below `-1` (i.e. `s < -1`,
the count `K`) become slopes strictly above `-1` and below `0`
under inversion (i.e. `-1/s` lives in `(-1, 0)`), and vice
versa. The K-shift is exactly the offset that compensates for
that asymmetry so that taking the slope at position
`floor((N+1)/2) + K` in the sorted list gives you the same
geometric line whether you regressed y-on-x or x-on-y. When `K =
0` (e.g. all pristine slopes positive) the shift is zero and PB
collapses to Theil-Sen exactly. When `K > 0` PB pulls toward the
upper part of the slope distribution.

## Live smoke against the local pew queue

Running the new lens against `~/.config/pew/queue.jsonl` produces
1,940 rows across 6 sources, the verbatim block from the v0.6.218
CHANGELOG (sorted `magnitude-desc`, source name redacted to
`vscode-redacted` per the operator's name-scrub rule; all numbers verbatim):

```
source          rows  mean         median      first       last        naive        theilSen      slope         sign  N        K       shiftIdx  shiftRatio  pbGap
codex           64    12650385.31  7132861.00  2695764.00  8565718.00  +93173.8730  +123739.4516  +665127.3556  up    2,016    832     1,424     0.706       +541387.9039
claude-code     299   11512995.95  3319967.00  1470723.00  201134.00   -4260.3658   +31445.1270   +157590.2711  up    44,551   16,757  30,654    0.688       +126145.1442
opencode        434   10395878.93  8107520.00  96926.00    6052401.00  +13753.9838  +13869.9719   +44700.5333   up    93,961   38,458  66,210    0.705       +30830.5614
openclaw        540   3704978.28   2295689.00  721224.00   1423435.00  +1302.8033   -5223.2956    +15914.1405   up    145,530  93,060  119,295   0.820       +21137.4361
hermes          270   749303.41    432176.00   2061198.00  317933.00   -6480.5390   -577.0853     +3628.8263    up    36,315   19,223  27,769    0.765       +4205.9116
vscode-redacted 333   5662.84      2319.00     458.00      9990.00     +28.7108     +1.9689       +34.5276      up    55,268   25,076  40,172    0.727       +32.5587
```

Six sources, six PB slopes, all positive. Six naive slopes — but
**only four positive**. The two negative naive slopes are
`claude-code` (`-4260.3658 tokens/row`) and `hermes`
(`-6480.5390 tokens/row`). For both of those sources, PB is
positive: claude-code at `+157590.2711` and hermes at
`+3628.8263`. That is the cohort `signFlippedFromNaive` picks out
— exactly two sources out of six, both with negative naive and
positive PB.

The CHANGELOG narrative for v0.6.218 calls this out explicitly:

> Live smoke against the local pew queue surfaces two flippers —
> `claude-code` and `hermes` — both of whose `naiveSlope` is
> negative (endpoints down) but whose PB shifted-median slope is
> positive (bulk pair-cloud rising). The other four sources
> (codex, opencode, openclaw, vscode-redacted) agree on the
> upward direction and just disagree on magnitude.

## Why two, and why those two

There is a structural reason exactly two sources flip and the
other four do not. The naive slope is
`(last - first) / (rows - 1)` — it is a function of literally two
data points (the first and the last) out of a column that has
hundreds of rows. PB, by contrast, is a function of all
`n*(n-1)/2` pairwise slopes (`N` in the table — 44,551 pairs for
claude-code, 36,315 for hermes). When the first point is high
and the last point is low *but the bulk in between is monotonically
rising*, naive returns negative and PB returns positive. That is
exactly the geometry of the two flippers in the table:

- **claude-code**: `first = 1,470,723`, `last = 201,134`. The
  endpoint pair drops by about 1.27M tokens over 298 row-steps,
  giving naive `-4260.3658 tokens/row`. But the pair cloud has
  16,757 slopes below `-1` out of 44,551 total — `K/N ~= 0.376`
  of the pair cloud is steeply negative. The shifted median lands
  at sorted position 30,654 of 44,551 (`shiftRatio = 0.688`),
  which is in the upper third of the slope distribution. The PB
  slope at that position is `+157590.2711 tokens/row` — large and
  positive. The `pbGap = +126145.1442 tokens/row` is the literal
  size of the disagreement vs Theil-Sen alone (which only
  partially compensates: TS gives `+31445.1270`, PB the full
  `+157590.2711`).
- **hermes**: `first = 2,061,198`, `last = 317,933`. Similar
  shape — a dramatic drop from first to last (1.74M tokens over
  269 row-steps, naive `-6480.5390 tokens/row`), with the bulk
  rising slowly underneath. PB lands at `+3628.8263`. Theil-Sen
  was already nearly flat (`-577.0853`); PB pushes it firmly
  positive. The `pbGap = +4205.9116` is small in absolute terms
  but enough to flip the sign.

The non-flippers tell the converse story. `codex` has
`first = 2,695,764` and `last = 8,565,718` — the endpoint pair is
already strongly upward and the bulk agrees. `opencode` first/last
`96,926` -> `6,052,401`, also strongly up. `openclaw`
`721,224` -> `1,423,435`, mildly up. `vscode-redacted`
`458` -> `9,990`, also mildly up. For all four, naive and PB
point the same direction; PB just disagrees on magnitude (and
sometimes by an order of magnitude, e.g. opencode `+13,754` naive
vs `+44,700` PB).

## What `shiftRatio` adds that wasn't there before

The shift-diagnostic triple — `pairsValid` (`N`),
`pairsBelowMinusOne` (`K`), and `shiftRatio = shiftIndex / N` —
is the headline reporting addition that makes PB legible vs its
sibling Theil-Sen. The two estimators share the same pair cloud
and the same ~29.3% breakdown; they differ only in **where in the
sorted slope list the median is taken**. `shiftRatio = 0.5` means
PB collapsed to Theil-Sen (no need to shift; `K = 0`).
`shiftRatio` noticeably above `0.5` means the K-shift was large
enough to pull the estimator into the upper part of the slope
distribution.

In the live smoke the shiftRatios cluster tightly between **0.688
and 0.820** — nowhere near 0.5. Every one of the six sources has
been shifted substantially upward by the K-correction.
`openclaw` is the highest at `0.820`, meaning roughly 82% of its
sorted pairwise slopes lie at or below the PB pick. The
CHANGELOG narrative explains this as:

> roughly 82% of its sorted pairwise slopes lie at or below the
> PB pick, i.e. its slope cloud is heavily dominated by
> negative-magnitude pairs and PB has had to shift far up the
> distribution to settle on a small-positive consensus.

That is a different empirical claim than "the slope is positive".
It is a claim about the *shape* of the pair cloud — most of the
pairs are slightly negative; a long tail of strongly positive
pairs pulls the shifted-median pick into a small-positive region.
Without the `shiftRatio` diagnostic you cannot distinguish that
geometry from "the pair cloud is already mostly slightly
positive". With it, you can.

## What `pbVsTheilSenGap` adds

The fourth diagnostic in the new triple is `pbVsTheilSenGap =
slope - theilSenSlope` — the literal correction in `tokens/row`
introduced by the PB shift. The CHANGELOG closes with an
interesting empirical pattern about its scaling:

> `pbVsTheilSenGap` is large for high-throughput sources (codex
> `+541k`, claude-code `+126k`) and small for low-throughput
> sources (vscode-redacted `+33`), scaling roughly with the
> source's mean — exactly what an errors-in-both-variables
> correction should do when the y-error is order-of-magnitude
> proportional to `y`.

The scaling is approximately right. Mean values from the table:
codex 12.65M, claude-code 11.51M, opencode 10.40M, openclaw 3.70M,
hermes 0.75M, vscode-redacted 5.66k. Gaps from the table: codex
+541k, claude-code +126k, opencode +31k, openclaw +21k, hermes
+4k, vscode-redacted +33. Ratios of gap to mean: 0.0428, 0.0110,
0.00298, 0.00571, 0.00533, 0.00582. The ratios for the four
mid-throughput sources (opencode, openclaw, hermes,
vscode-redacted) are tightly clustered around 0.003 to 0.006 —
roughly half a percent of mean. Codex and claude-code are
outliers at 4.3% and 1.1% respectively — both of those are the
high-mean sources where the pair cloud is strongly directional.
That is consistent with the CHANGELOG claim that the gap scales
with mean *plus* an extra term for direction-strong sources where
PB has the most room to disagree with Theil-Sen.

## Where this fits in the slope family

The slope family in `pew-insights` now consists of four
estimators stacked at increasing levels of robustness:

1. **naive** (slope-family header, present since v0.6.214):
   `(last - first) / (rows - 1)`. Function of two data points.
   Breakdown 0% — a single bad endpoint flips it.
2. **Theil-Sen** (v0.6.214, feat `6102278`, release `dffe6de`,
   refinement `1e268d3` adding `mannKendallS`): plain median of
   `n*(n-1)/2` pairwise slopes. Breakdown ~29.3%. y-asymmetric.
3. **Siegel** (v0.6.216, feat `fc2962e`, release `367f5dd`,
   refinement `44b03fa` adding `anchorAgreement`): nested median —
   for each anchor row `i`, take median of the n-1 slopes from
   `i` to every other row, then take the median of the n
   per-anchor medians. Breakdown ~50%. y-asymmetric.
4. **Passing-Bablok** (v0.6.218, this release): shifted median of
   `n*(n-1)/2` pairwise slopes. Breakdown ~29.3% (same as
   Theil-Sen). x<->y symmetric. Errors-in-both-variables.

The first three answer the question "what is the slope of the
trend line, assuming x is exact and y has noise?" Passing-Bablok
answers "what is the slope of the trend line, assuming both x and
y can carry error?" That is a different physical model, and for
the per-row token measurement use case it is arguably a more
honest one: the row index `i` is exact only because the
orchestrator decided the dispatch order, not because the
chronological time between rows was uniform; the act of treating
row index as a noise-free covariate already buries some real
variance. Passing-Bablok lets that variance back into the model.

The choice between Siegel and Passing-Bablok is the most
interesting one in the slope family now. They optimize different
things: Siegel maximizes raw outlier tolerance (50% breakdown vs
PB's 29.3%); PB maximizes functional invariance (x<->y symmetry vs
Siegel's y-asymmetry). The CHANGELOG's framing — "pick PB when
both axes can carry error, pick Siegel when up to half the points
can be arbitrary outliers" — is the right disposition.

## What the sign-flippers cohort means for downstream

The most actionable thing v0.6.218 ships is not the estimator. It
is the `signFlippedFromNaive` flag. The CHANGELOG says so
explicitly:

> The flip flag is the most actionable cohort selector for
> downstream analysts: it is exactly the set of sources where a
> robust trend reading disagrees with the endpoint-only reading
> about the *direction* of the trend (not just its magnitude).

For the live smoke, that cohort is exactly two sources:
claude-code and hermes. Both are sources whose token-budget over
the corpus window has the same characteristic shape — a high
opening burst followed by a long, slow rising tail that ends
lower than the opening but not as low as a naive endpoint reading
suggests. If you were reporting these source-by-source token
trends to a stakeholder using only the naive endpoint slope, you
would tell them claude-code and hermes are trending **down**. The
robust reading says they are trending **up**, slowly, and the
endpoint-only reading is being misled by a single high-leverage
opening point.

That is exactly the kind of methodological disagreement the
v0.6.218 release was built to surface, and the new sort key
`sign-flipped-first` in the release puts it at the top of the
output by default when requested. Two flippers out of six is a
significant minority — one in three sources' direction signal is
being inverted by the choice of estimator. That ratio alone is a
strong argument for shipping a robust slope estimator alongside
any naive-endpoint dashboard, and for documenting the flip
cohort prominently when summarizing per-source trends.
