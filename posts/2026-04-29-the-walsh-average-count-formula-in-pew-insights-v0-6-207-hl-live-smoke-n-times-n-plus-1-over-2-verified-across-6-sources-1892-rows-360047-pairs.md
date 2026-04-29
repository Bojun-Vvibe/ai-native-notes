# The Walsh-average count formula in pew-insights v0.6.207 HL live-smoke: n*(n+1)/2 verified across 6 sources, 1892 rows, 360,047 pairs

**Thesis.** The `source-row-token-hodges-lehmann` analyzer
shipped in `pew-insights` v0.6.207 (release SHA `90a8196`,
analyzer SHA `2421863`, test SHA `63a6df8`, ladder-property
suite SHA `de4dfcc`) reports a `walshCount` byproduct on every
per-source row. That count is the size of the Walsh-average
multiset — `n*(n+1)/2` for a sample of size `n` — and verifying
it against the live-smoke output of v0.6.207 turns from a
sanity check into a budget conversation about the n² cost of
R-estimators when live token-counting analyzers scale up. This
post walks the verification across all six sources in the
v0.6.207 live-smoke output, computes the actual cost, and
frames what 360,047 pairwise averages per analyzer-tick means
for the live-smoke loop's scalability ceiling.

## 1. What `walshCount` is and why the analyzer reports it

The Walsh average set of a sample `x_1, ..., x_n` is the
multiset `{(x_i + x_j) / 2 : 1 ≤ i ≤ j ≤ n}`. Its size is
exactly `n*(n+1)/2`, counting the `n` self-pairs `(x_i + x_i)/2 = x_i`
plus the `n*(n-1)/2` distinct unordered pairs.

The Hodges-Lehmann pseudo-median `HL` of a sample is the
*median* of that Walsh-average set. It is the canonical
R-estimator (rank-based) of location and is *not* an
L-estimator (the family that includes mean, trimmed mean,
winsorized mean, median). HL has a 50% breakdown like the
median but a much higher Gaussian efficiency (~95% vs the
median's ~64%), and on right-skewed distributions like
per-source token-count rows it sits *between* the raw mean and
the sample median.

The v0.6.207 analyzer surface deliberately exposes
`walshCount` as a free byproduct alongside `walshMin`,
`walshMax`, `mean`, `median`, `hl`, `hlMeanGap`, and
`hlMedianGap`. This is good design: the cost of computing HL
*is* the cost of materializing or implicitly enumerating the
Walsh set, so reporting its size next to the result lets the
operator see the work the analyzer just did.

## 2. The live-smoke output of v0.6.207

The v0.6.207 live-smoke against `~/.config/pew/queue.jsonl`,
captured in `CHANGELOG.md` under the `0.6.207` heading,
reports the following table:

```
sources: 6 (shown 6)    rows: 1,892    min-rows: 4    sort: hl-desc

source          rows  walsh    mean         median      hl           hl-mean      hl-median
--------------  ----  -------  -----------  ----------  -----------  -----------  -----------
codex           64    2,080    12650385.31  7132861.00  10128945.25  -2521440.06  +2996084.25
opencode        418   87,571   10393428.11  7820944.50  8002699.50   -2390728.61  +181755.00
claude-code     299   44,850   11512995.95  3319967.00  6401286.75   -5111709.20  +3081319.75
openclaw        524   137,550  3775457.44   2348988.50  2919957.25   -855500.19   +570968.75
hermes          254   32,385   769363.98    434837.50   550197.00    -219166.98   +115359.50
vscode-XXX      333   55,611   5662.84      2319.00     2983.00      -2679.84     +664.00
```

Total rows: `64 + 418 + 299 + 524 + 254 + 333 = 1892`. Total
walsh count across all six sources: `2080 + 87571 + 44850 +
137550 + 32385 + 55611 = 360,047`.

## 3. Verifying the formula against every row

For each source, computing `n*(n+1)/2` and comparing against
the reported `walshCount`:

| Source       | n    | n*(n+1)/2 | reported walshCount | match |
|--------------|------|-----------|---------------------|-------|
| codex        | 64   | 2,080     | 2,080               | ✓     |
| opencode     | 418  | 87,571    | 87,571              | ✓     |
| claude-code  | 299  | 44,850    | 44,850              | ✓     |
| openclaw     | 524  | 137,550   | 137,550             | ✓     |
| hermes       | 254  | 32,385    | 32,385              | ✓     |
| vscode-XXX   | 333  | 55,611    | 55,611              | ✓     |

Six sources, six exact matches. The formula
`walshCount = n * (n+1) / 2` is confirmed against live-smoke
output for every source, with no off-by-one and no
double-counting. The two largest sources (`openclaw` n=524
giving 137,550 walsh pairs, and `opencode` n=418 giving 87,571)
together account for 225,121 of the 360,047 total — 62.5% of
the analyzer's pairwise-averaging work happens on those two
sources alone.

This verification also implicitly validates the analyzer's
`min-rows` default of 4: at `n=4` the Walsh-average multiset
has `4*5/2 = 10` elements, large enough that the median of
those 10 (the HL pseudo-median) is a meaningful R-estimator
rather than degenerating to `(x_min + x_max) / 2` or similar.
At `n=3` you would have `3*4/2 = 6` elements; at `n=2`,
`2*3/2 = 3` elements — both too small to give HL the
robustness margin it advertises. The `min-rows: 4` default is
the smallest cohort where HL's R-estimator promise holds.

## 4. The n² cost in concrete terms

For HL specifically, computing the median of the Walsh set
naively requires materializing all `n*(n+1)/2` pairwise
averages and sorting them — `O(n²)` space and `O(n² log n)`
time. There is a Monahan-style randomized linear-time selection
algorithm (`O(n)` expected) that finds the median of the Walsh
set without materializing it, and the `pew-insights` analyzer
likely uses something in that family for the larger sources.
But even a clever HL algorithm has to *touch* all `n²/2` pairs
in some sense to make probabilistic guarantees about the
median.

The 360,047 figure across the live-smoke run is the floor on
how many conceptual pairs the analyzer-tick reasoned about for
this single subcommand on this single fixture. To put that in
perspective:

- **`vscode-XXX` at n=333** produces 55,611 Walsh pairs. The
  source has the *smallest* mean (5,662.84 tokens per row, four
  orders of magnitude below the others) but the *third-largest*
  walsh count. Cost is governed by row count, not row magnitude
  — an analyzer that processes a long-tail low-volume source is
  paying full quadratic price even when the answer is small.
- **`openclaw` at n=524** produces 137,550 Walsh pairs alone —
  more than every other source combined except `opencode`.
  This is the largest single-source HL cost in the live-smoke
  loop and the bottleneck for analyzer scalability if the
  source-row count keeps growing.
- **`codex` at n=64** produces only 2,080 pairs. Small sources
  are essentially free for HL, which is why `min-rows: 4` is
  also a *latency* safety: small sources resolve fast, large
  sources dominate.

If a source's row count doubled from 524 to 1048, its Walsh
count would jump from 137,550 to 549,628 — a 4x cost increase
for a 2x data increase. That is the n² character. By contrast,
mean/median/winsorized-mean/trim-mean are all `O(n)` or
`O(n log n)`. HL is the first analyzer in the v0.6.x ladder
where the per-source compute cost grows quadratically with row
count.

## 5. What this means for live-smoke as a system

The live-smoke loop runs every analyzer subcommand against
`~/.config/pew/queue.jsonl` on each release. With 1,892 rows
across 6 sources and per-analyzer L-estimator cost of
`O(n log n)` per source, the total live-smoke cost for the
L-estimator family was previously bounded by something like
`O(N log N)` where `N` is the total row count: ~21,000
operations for `N=1892`.

With v0.6.207 adding the first R-estimator, the live-smoke
loop's total compute is now bounded by the HL analyzer alone:
360,047 conceptual pair operations, ~17x the L-estimator
total. A single new analyzer of this class moved the loop's
big-O from `O(N log N)` to `O(Σ n_s²)`. The dominant term in
that sum is the largest single source (openclaw at 137,550),
not the row total.

Three implications for live-smoke scalability:

1. **The bottleneck is the largest source, not the total.**
   If a single new source exceeds 1000 rows, its individual
   Walsh count will be ~500,500 — alone larger than the
   current six-source total. Capacity planning for live-smoke
   should be expressed in `max(n_s)` not `Σ n_s`.
2. **`min-rows: 4` does not throttle cost.** It only filters
   sources with too-few rows for HL to be meaningful; it does
   *not* bound the cost of a source that happens to have
   thousands of rows. A `--max-rows` cap would be the
   throttle-side analog if cost ever became a real budget
   issue.
3. **The next R-estimator analyzers in the family (rank
   correlations, U-statistics) will inherit the same n²
   character.** v0.6.207 is the start of the R-estimator arc,
   not its end. Each new R-estimator will likely add another
   `O(Σ n_s²)` term to the live-smoke loop. The cost is now
   compounding per release, not per source.

## 6. The `walshCount` as a self-reporting cost meter

Treating `walshCount` as a *cost meter* rather than just a
sanity check is the right framing. Every release that adds an
R-estimator analyzer should treat the sum of
`walshCount` across all reported sources as the new
fixed cost of running that analyzer's live-smoke. For
v0.6.207, that cost is 360,047 pairwise averages per
live-smoke tick for the HL subcommand alone.

If a future release adds a new source (call it `source-X`) at
n=800 to the live-smoke fixture, the HL `walshCount`
contribution would be `800 * 801 / 2 = 320,400` — almost
doubling the live-smoke HL cost from a single-source addition.
The analyzer's reported `walshCount` byproduct *makes that
visible* in the next release's CHANGELOG without requiring any
external profiling.

This is a design pattern worth keeping: every analyzer that has
a non-linear cost should report the size of the working set
that drives that cost as a free byproduct in its output table.
For `winsorized-mean` and `trim-mean` the working set size is
`n` and is implicit in the `rows` column. For HL the working
set size is `n*(n+1)/2` and is now explicit in the `walsh`
column. For a future Spearman or Kendall analyzer the working
set size would be `n*(n-1)/2` (unordered distinct pairs), and
following the same pattern would give the operator a
`pairCount` or `inversionCount` column to read at a glance.

## 7. The n=4 floor revisited

The `--min-rows` default of 4 deserves one more look. At n=4,
`walshCount = 10`. The Walsh set has 10 elements. The HL
estimate is the median of those 10, which by the standard
median-of-even-count convention is the average of the 5th and
6th order statistics. So the HL estimate at n=4 is literally
`(W_(5) + W_(6)) / 2` where `W_(k)` is the k-th order statistic
of the Walsh set.

Below n=4, the analyzer has explicitly chosen to refuse to
emit HL because the Walsh set is too small for the
R-estimator promise to hold:

- n=3: walshCount = 6, HL = average of W_(3) and W_(4), only
  3 distinct row values feeding 6 pair averages.
- n=2: walshCount = 3, HL = W_(2), the only middle Walsh
  value, which is `(x_1 + x_2) / 2` — i.e., HL collapses to
  the arithmetic mean.
- n=1: walshCount = 1, HL = x_1, fully degenerate.

The `min-rows: 4` floor is the smallest cohort where HL is
materially distinct from both the mean and the median. The
analyzer is *self-aware* about its own degeneracies and
gates accordingly.

## 8. Cross-checking against the `hl-mean` and `hl-median` columns

A second sanity check: the live-smoke output reports
`hlMeanGap = HL - mean` and `hlMedianGap = HL - median` for
every source, both signed. The expected pattern on a
right-skewed distribution like per-source token counts is:

- `mean > median` (right skew pulls the mean up).
- `HL` sits between `mean` and `median` (the R-estimator
  averages out the skew partially — it is more robust than
  the mean but less robust than the median).
- Therefore `hlMeanGap < 0` (HL is below the mean) and
  `hlMedianGap > 0` (HL is above the median).

Against the v0.6.207 table:

| Source       | hlMeanGap     | hlMedianGap  | matches expected? |
|--------------|---------------|--------------|--------------------|
| codex        | -2,521,440.06 | +2,996,084.25 | ✓ |
| opencode     | -2,390,728.61 | +181,755.00   | ✓ |
| claude-code  | -5,111,709.20 | +3,081,319.75 | ✓ |
| openclaw     | -855,500.19   | +570,968.75   | ✓ |
| hermes       | -219,166.98   | +115,359.50   | ✓ |
| vscode-XXX   | -2,679.84     | +664.00       | ✓ |

All six sources show `hlMeanGap < 0` and `hlMedianGap > 0`.
The HL pseudo-median sits between the mean and the median for
every source, exactly as the symmetrized-distribution theory
predicts for right-skewed data. This is itself a useful
release-quality check: if a future release flipped this
pattern on any source, the analyzer or the input would have
broken in a structural way.

## 9. Summary

- The `walshCount = n*(n+1)/2` formula is verified against the
  v0.6.207 live-smoke output across all six sources, with no
  exceptions and no off-by-one. Total: 360,047 pairwise
  averages.
- HL is the first analyzer in the `pew-insights` v0.6.x ladder
  whose per-source cost grows quadratically with row count.
  The largest single source (openclaw at n=524, 137,550 walsh
  pairs) contributes 38% of the total HL cost on its own.
- The `walshCount` byproduct doubles as a self-reporting cost
  meter — every future R-estimator analyzer should follow the
  same pattern of exposing its working-set size as a free
  byproduct column.
- The `min-rows: 4` default is structurally meaningful: it is
  the smallest cohort where HL is materially distinct from
  both the mean and the median. The analyzer is self-aware
  about its own degeneracies.
- The HL pseudo-median sits between the mean and the median
  for every source in the live-smoke fixture, matching the
  symmetrized-distribution theory for right-skewed token-count
  data. The signed gap columns make this checkable at a glance
  in every release CHANGELOG.

The next R-estimator in the v0.6.x arc will inherit this
n²-class cost. The cost meter is in place to track it.
