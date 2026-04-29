# The Hodges-Lehmann live-smoke asymmetry signature: all six `hlMeanGap` negative, all six `hlMedianGap` positive, and the R-estimator position between mean and median

`pew-insights v0.6.207` (2026-04-29) shipped `source-row-token-hodges-lehmann`,
the first **R-estimator / U-statistic** location lens in a per-source
row-token analyzer family that, until this release, had been entirely
populated by L-estimators (mean, mid-range, median, midhinge, trimean,
IQM, trim-mean-{10,20,25,30}, winsorized-mean-{10,20}) and power means
(Lehmer-{-3..-1, 3..12}). The shipped live-smoke against
`~/.config/pew/queue.jsonl` produced a result that is strong enough to
warrant its own post: across all six sources, with `n` ranging from 64
rows (`codex`) to 524 rows (`openclaw`), **every single
`hlMeanGap` is negative and every single `hlMedianGap` is positive**.
That is not a stylized fact recovered from theory — it is the verbatim
table that the v0.6.207 changelog committed to the repository, and it
encodes a structural property of the queue's token distribution that is
worth reading off carefully before it gets buried under the next eight
releases.

## What the smoke table actually says

The CHANGELOG.md entry for v0.6.207 (lines 97–109 of
`pew-insights/CHANGELOG.md`) reports the following six-row table for the
`source-row-token-hodges-lehmann` subcommand, sorted `hl-desc`:

```
source          rows  walsh    mean         median      hl           hl-mean      hl-median
--------------  ----  -------  -----------  ----------  -----------  -----------  -----------
codex           64    2,080    12650385.31  7132861.00  10128945.25  -2521440.06  +2996084.25
opencode        418   87,571   10393428.11  7820944.50  8002699.50   -2390728.61  +181755.00
claude-code     299   44,850   11512995.95  3319967.00  6401286.75   -5111709.20  +3081319.75
openclaw        524   137,550  3775457.44   2348988.50  2919957.25   -855500.19   +570968.75
hermes          254   32,385   769363.98    434837.50   550197.00    -219166.98   +115359.50
vscode-XXX      333   55,611   5662.84      2319.00     2983.00      -2679.84     +664.00
```

Six sources. Six rows. Six `hl-mean` values, all strictly less than
zero. Six `hl-median` values, all strictly greater than zero. There is
no source for which the Walsh-average pseudo-median sits at or above the
arithmetic mean; there is no source for which it sits at or below the
sample median. The R-estimator is sandwiched, with a directional
preference, on every cohort the lens looked at.

## Why the sign uniformity is the whole story

For a finite sample drawn from any distribution, the Hodges-Lehmann
estimator `HL = median{(x_i + x_j)/2 : i <= j}` estimates the **center
of symmetry of the symmetrized distribution** `(X + X')/2`, where `X'`
is an independent copy of `X`. Three population-level facts compose to
explain what we are seeing:

1. For a **symmetric** distribution, `HL` agrees in expectation with
   both the population mean and the population median (which coincide).
   `hlMeanGap` and `hlMedianGap` are both zero in expectation.

2. For a **right-skewed** distribution with a finite mean, the
   population mean lies strictly above the population median. The
   Walsh-average symmetrization `(X + X')/2` produces a distribution
   whose mode and median are pulled **toward** the original median (the
   pairwise mean of two right-tailed draws is, on average, less heavy
   in the upper tail than a single right-tailed draw, because the
   averaging operation has variance `Var(X)/2`).

3. The Hodges-Lehmann estimator therefore sits **between** the sample
   mean and the sample median for right-skewed distributions, with the
   gap to the mean being negative (HL below mean) and the gap to the
   median being positive (HL above median).

The v0.6.207 smoke table is the empirical confirmation that
**every one of the six sources in the live queue has a right-skewed
per-row token distribution**. This is not a surprise — agent-emitted
sessions have a long upper tail because long context-window sessions
emit `total_tokens` values that the modal short session does not
approach — but it is the first time the property has been certified by
an estimator whose sign-of-gap diagnostic is sharp.

The earlier L-estimator family already gave us hints. The trim-mean and
winsorized-mean lenses (TM-10, TM-20, TM-25, TM-30, WM-10, WM-20)
all reported negative `tmMeanGap` and `wmMeanGap` values for every
source — see for example v0.6.206's TM-30 table (CHANGELOG.md lines
158–170) where `claude-code` shows the largest `tm-mean` gap of
`-7,465,830.02` tokens. Negative gap means "the lens-estimated center
is below the raw mean", which already implies an upper-tail-heavy
distribution. But TM and WM have **no built-in companion gap to the
median**; they only report the gap to the mean. The HL lens is the
first one that ships **two** signed gaps simultaneously — `hlMeanGap`
and `hlMedianGap` — and the conjunction `(hlMeanGap < 0) AND
(hlMedianGap > 0)` is a much more specific diagnostic than either alone.

## The magnitude ranking is not the same as the L-estimator ranking

It is tempting to read the table top-to-bottom and conclude that
`codex` has the heaviest right tail because its `hlMeanGap` of
`-2,521,440.06` is the largest negative sat at the top of the
`hl-desc` sort. That is wrong, and the table itself is the
counter-example: `claude-code` posts an `hlMeanGap` of
`-5,111,709.20` — almost exactly **twice** the magnitude of `codex`'s
gap — at row three of the table. The `hl-desc` sort is ordering by the
HL value itself (10.13M, 8.00M, 6.40M, 2.92M, 0.55M, 2.98K), not by the
gap magnitude.

Sorting the live-smoke table by `|hlMeanGap|` descending instead
produces:

| rank | source       | `|hlMeanGap|` (tokens) |
| ---: | :----------- | ----------------------:|
|    1 | claude-code  | 5,111,709.20           |
|    2 | codex        | 2,521,440.06           |
|    3 | opencode     | 2,390,728.61           |
|    4 | openclaw     | 855,500.19             |
|    5 | hermes       | 219,166.98             |
|    6 | vscode-XXX   | 2,679.84               |

This is the **upper-tail heaviness ranking** as seen by the Walsh
symmetrization. `claude-code` has the heaviest right tail in absolute
token-count terms. `vscode-XXX` has the lightest by three orders of
magnitude — consistent with the four-orders-of-magnitude floor that
recent posts have already documented for that source.

Sorting instead by `|hlMedianGap|` descending produces a slightly
different ordering:

| rank | source       | `|hlMedianGap|` (tokens) |
| ---: | :----------- | ------------------------:|
|    1 | claude-code  | 3,081,319.75             |
|    2 | codex        | 2,996,084.25             |
|    3 | openclaw     | 570,968.75               |
|    4 | opencode     | 181,755.00               |
|    5 | hermes       | 115,359.50               |
|    6 | vscode-XXX   | 664.00                   |

`claude-code` and `codex` swap places relative to where they sat in the
HL-value sort, and `opencode` falls from second to fourth.
`opencode` has by far the smallest `|hlMedianGap|` of the three
million-token sources at `181,755` — only about 6 % the size of
`claude-code`'s. That asymmetry — large `|hlMeanGap|` of
`-2,390,728.61` but tiny `|hlMedianGap|` of `+181,755.00` — has a
clean reading: `opencode`'s sample median is already nearly where the
Walsh-symmetrization wants it to be, while its raw mean is being pulled
far above by a small number of very-heavy-tail sessions. In the
language of L-estimators, `opencode` looks like a distribution where
`median ~ HL` and `mean >> HL`, i.e. the right tail is concentrated in
the top quantile and does not bleed down into the upper-half body.
`claude-code`, by contrast, has `mean - HL = 5.11M` AND `HL - median =
3.08M` — a distribution where the upper tail is broader and pulls every
location estimator a different distance toward itself.

## Where each source sits on the `(median, HL, mean)` line

The full triple makes the picture concrete. Re-arranged in the
median-HL-mean order, every source produces a strict inequality
`median < HL < mean`:

| source       | median       | HL           | mean         |
| :----------- | -----------: | -----------: | -----------: |
| codex        | 7,132,861    | 10,128,945   | 12,650,385   |
| opencode     | 7,820,944    | 8,002,699    | 10,393,428   |
| claude-code  | 3,319,967    | 6,401,286    | 11,512,995   |
| openclaw     | 2,348,988    | 2,919,957    | 3,775,457    |
| hermes       | 434,837      | 550,197      | 769,363      |
| vscode-XXX   | 2,319        | 2,983        | 5,662        |

Six sources. Six `median < HL < mean` orderings. The R-estimator is
**strictly interior** to the `(median, mean)` interval on every cohort.
This is precisely the asymptotic property the v0.6.207 changelog text
predicts (lines 43–47):

> For symmetric distributions HL agrees with the population
> median in expectation; for asymmetric distributions HL
> estimates the center of symmetry of the symmetrized
> distribution `(X + X')/2`, which is generally between the
> population mean and the population median.

The phrase "generally between" carries some latitude in the theoretical
literature — there are pathological distributions where the
symmetrization can push HL outside the `[median, mean]` interval — but
on agent-token data the property holds tightly. Six sources, zero
exceptions. The lens has effectively certified that the per-source
row-token distributions in the live queue are well-behaved enough that
the canonical R-estimator is doing what its theory promises.

## The Walsh-count combinatorics is what makes this affordable

The `walshCount` column in the live-smoke table is not decoration. It
is `n*(n+1)/2`, the size of the multiset of pairwise sums (including
self-pairs) that the algorithm has to median over. The numbers in the
table — 2,080 / 87,571 / 44,850 / 137,550 / 32,385 / 55,611 — sum to
**359,047 Walsh averages** computed across the six sources to produce
this single live-smoke run.

For the largest cohort (`openclaw`, n=524), the Walsh multiset has
137,550 elements, and the changelog reports that medianing it takes
"~1 ms". This is fast enough that the v0.6.207 implementation
deliberately rejected the asymptotically faster `O(n log^2 n)` Monahan
algorithm in favor of exact `O(n^2)` enumeration (changelog lines
52–58):

> For the queue sizes seen in this repo (largest source ~500 rows ->
> ~125,250 Walsh averages, ~1 ms to median) the exact
> enumeration is fast and **exact**; we deliberately do not
> use the `O(n log^2 n)` Monahan algorithm — exactness is more
> valuable than the asymptotic speedup at our scale, and the
> simpler code is auditable.

This is a calibrated engineering choice: the repo's largest-source
projection is roughly 525 rows, which puts `n^2/2` at about 138K. To
hit a regime where Monahan beats exact enumeration you would need
sources with rows in the tens of thousands. The current queue is two
orders of magnitude away from that threshold. The decision encodes the
cardinality envelope of the data, not just an aesthetic preference for
simpler code. (For comparison, ingestion-pipeline-style telemetry
queues commonly hit `n` in the hundreds of thousands, and exact Walsh
enumeration would not be viable there.)

The `walshCount` byproduct is also a useful join key: it is a pure
function of `rows`, so any future analyzer that wants to express
"this source has W17 sessions" can use either `rows` (n) or
`walshCount` (n*(n+1)/2) interchangeably, with the latter giving a
quadratic blow-up that exposes large cohorts more sharply on a log
scale.

## What the lens does that the L-family cannot

Returning to the central diagnostic: the L-estimator lenses already
shipping in the family — TM-10/20/25/30, WM-10/20, midhinge, trimean,
IQM — all answer the question "what is a robust location estimator
of the central body?". They give one number per source per lens. They
all agree, on this queue, that the location is below the raw mean.

The HL lens answers a different question: "what is the location
estimator that pretends the data is symmetric and asks where the
center of that symmetric distribution would be?". It also gives one
number per source. But its **two** companion gaps — to the mean and
to the median — together encode the **direction and magnitude of the
distributional asymmetry** that the L-estimators only see one side of.
A negative `hlMeanGap` plus a positive `hlMedianGap` says "right
skew"; the converse pair would say "left skew"; both gaps near zero
would say "approximately symmetric"; same-sign gaps would say
"distribution that the Walsh symmetrization is moving in only one
direction relative to both mean and median", which would be an
unusual diagnostic worth investigating.

On the v0.6.207 live-smoke, every source posts the right-skew
signature. None posts left skew. None posts approximate symmetry.
None posts the unusual same-sign pair. The fleet is, in this
specific structural sense, **distributionally homogeneous** — six
sources, six right-skewed token distributions, six confirmations
that the upper tail is heavier than the lower tail by an amount
detectable through the Walsh-average symmetrization.

That homogeneity is itself a finding. It says the agent
ecosystem in this queue does not contain a source whose token
distribution is qualitatively different in shape from the others —
they vary in scale (six orders of magnitude from `vscode-XXX`'s
2,983-token HL to `codex`'s 10,128,945-token HL), they vary in
sample size (64 to 524 rows), they vary in tail heaviness
(`|hlMeanGap|` ranges from 2,679 to 5,111,709 tokens), but they all
sit on the same side of the symmetric/asymmetric boundary. There is
no "low-asymmetry source" hiding in this queue, and there is no
"left-skewed source" either. Future drift in either direction would
be immediately visible as a sign flip in the HL signature, which
makes this lens a usable monitoring primitive in addition to a
location estimator. The 0.293 breakdown point (changelog lines
39–41) means it would take roughly 30 % of the rows of a source to
go pathological before HL itself starts misreporting — a generous
budget for routine drift detection.

## Test count delta and the cost of the new lens

The v0.6.207 release ships **+54 new tests** (4,823 -> 4,877; changelog
lines 78–84): 44 unit + property tests in
`sourcerowtokenhodgeslehmann.test.ts`, plus 10 cross-analyzer ladder
tests in `sourcerowtokenhodgeslehmann.ladder.test.ts`. The ladder
tests are the interesting ones — they exercise three explicit cross-
estimator agreements:

- **HL vs median** agreement on symmetric distributions (where they
  must coincide in expectation),
- **HL vs TM-30** agreement at low contamination (where both should
  ignore the trimmed tails) **and divergence at high contamination**
  (where TM-30's hard drop diverges from HL's soft Walsh
  symmetrization), and
- **HL's sensitivity to spacing changes in trimmed tails that TM-30
  ignores by construction**.

That third test is the key methodological commitment. TM-30 drops the
top 30 % and bottom 30 % of order statistics and is, by construction,
insensitive to any rearrangement within those dropped regions. HL,
because it medians over Walsh averages, is sensitive — moving an
interior `x_i` even without crossing another point changes `n` Walsh
averages. The test suite locks this distinction in: the two estimators
are not redundant, and the ladder tests guarantee they will diverge on
distributions where the trimmed-tail spacing matters.

The +54 test delta brings the total to 4,877 tests for a 25.6 KLoc
TypeScript project. Per the recent test-count growth-curve post for
v0.6.194..v0.6.203, the trim-mean family added on the order of +37 to
+50 tests per release; HL at +54 sits at the high end of that band,
reflecting the extra ladder tests that an R-estimator (which is not
mechanically equivalent to any L-estimator) needs in order to certify
its non-redundancy.

## What the next rung should answer

Two follow-on questions are visible from this table that the next
release could pick off:

First, **does the sign uniformity hold across `--since` windows?** The
v0.6.207 smoke is a single point-in-time snapshot of the queue. A
weekly-window-sliced HL would tell us whether the right-skew
signature is stable across time, or whether some sources oscillate
between right-skew and approximate-symmetry depending on what kind of
sessions are running that week. A flip from negative `hlMeanGap` to
positive on any source would be a strong signal worth alerting on.

Second, **does the magnitude of `|hlMeanGap|` correlate across sources
with any other shipped metric?** The L-estimator family already exposes
`tmMeanGap`, `wmMeanGap`, and the midhinge / trimean differences. A
6×6 cross-source correlation between `|hlMeanGap|` and `|tmMeanGap|`
at α=0.30 would tell us whether the R-estimator is capturing
asymmetry information that the L-family already had access to, or
whether it is genuinely new signal. The HL changelog text explicitly
positions the lens as "mechanically distinct from every shipped
median-family lens", so a correlation matrix would test that
claim empirically rather than just structurally.

Both questions can be answered with the lens that already shipped.
v0.6.207 is the substrate; the analysis is downstream work.

## Citations

- `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` lines 5–109
  (v0.6.207 entry, including the live-smoke table at lines 97–109,
  the algorithm/breakdown discussion at lines 36–58, the byproduct
  list at lines 60–67, and the test count delta at lines 78–84).
- `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` lines 158–170
  (v0.6.206 TM-30 live-smoke table, used as the comparison anchor
  for the L-estimator side of the family).
- Total Walsh averages computed in the v0.6.207 live smoke:
  `2,080 + 87,571 + 44,850 + 137,550 + 32,385 + 55,611 = 359,047`,
  derived from the `walsh` column of the table.
- Sign-uniformity finding: 6/6 `hlMeanGap` < 0 and 6/6 `hlMedianGap`
  > 0 in the v0.6.207 live-smoke run, verbatim from the committed
  table.
