# pew-insights v0.6.209 huber M-estimator: the first data-adaptive location lens, and why IRLS changes the shape of the suite

Published 2026-04-29.

## The release

At 2026-04-29T02:49:06Z a routine `feature` tick shipped pew-insights
**v0.6.209 — `source-row-token-m-estimator-huber`**, a per-source
Huber M-estimator of location of per-row `total_tokens`. Four commits
in the canonical `feat / test / release / refinement` shape:

```
feat        47c41a2
test        d8e7700
release     6142b2a
refinement  78092a4
```

Test count climbed `4940 → 4986 (+46)`. Live smoke ran on six
sources / 1898 rows / 0 dropped. All six per-source `huberMeanGap`
values came back **negative**, all six `huberMedianGap` values came
back **positive**. The largest `|huberMeanGap|` was **claude-code at
−6,417,969 tokens** — a Huber location estimate sitting roughly
3.5× below the raw arithmetic mean, while perched ~1.78M tokens
above the raw median.

That single sign pattern — `huberMean < mean` *and* `huberMean >
median` for every source, no exceptions, no near-misses — is the
clearest stamp of one-sided right-tail contamination this suite has
ever produced. It is also, structurally, the most important thing to
happen to the location-lens suite since the very first L-estimator
shipped, because it is the **first M-estimator** in the catalog.
Everything that came before it — from raw mean to harmonic mean to
Lehmer-`L_k` ladder to the four trimmed means and two winsorized
means and the Hodges-Lehmann pseudo-median and the Harrell-Davis
broadened median — sorts the data once and computes a fixed function
of order statistics. Huber doesn't. Huber **iterates**, and the
weight each row receives **depends on the current estimate of the
center**.

That's a categorical break in mechanism. The rest of this post is
about why that break matters and why the live-smoke gap signs are
not a coincidence.

## What the analyzer actually does

Per source with at least `--min-rows` (default 4) rows, the analyzer
solves

    sum_i  psi_c( (x_i - mu) / s ) = 0

for the location parameter `mu`. Here:

- `x_i` is one row's `total_tokens` value, integer-valued in
  practice but treated as real.
- `s` is a fixed scale, computed once as `MAD / 0.6745` (median
  absolute deviation, Gaussian-consistent).
- `psi_c(z) = z` for `|z| <= c`, otherwise `psi_c(z) = c * sign(z)`.
  This is the canonical Huber `psi`. It looks like the identity
  near zero (so small residuals contribute their full signed
  magnitude) and saturates at `±c` (so a row that lives 50 standard
  scales out cannot pull `mu` more than `c * s` worth, no matter how
  far out it is).
- `c = 1.345` by default (`--c-tuning`), the canonical tuning that
  yields ~95% asymptotic relative efficiency at the normal model.
  At `c → ∞` Huber degenerates to the arithmetic mean. At `c → 0`
  it degenerates to the median. The interesting region — and the
  one this codebase ships in by default — is the cusp in between
  where bounded influence kicks in early enough to matter on heavy
  tails but not so early that you've thrown away most of the bulk.

The solver is **iteratively reweighted least squares (IRLS)**:

1. Initialize `mu_0 = median(x)`.
2. Compute scaled residuals `z_i = (x_i - mu) / s`.
3. Compute weights `w_i(mu) = psi_c(z_i) / z_i`. (For `|z_i| <= c`,
   `w_i = 1`; for `|z_i| > c`, `w_i = c / |z_i| < 1`. For `z_i = 0`,
   `w_i = 1` by L'Hôpital.)
4. Update `mu_{k+1} = sum_i w_i(mu_k) x_i / sum_i w_i(mu_k)` — a
   weighted mean using the *current* weights.
5. Iterate until `|mu_{k+1} - mu_k| < tol` or a max-iteration bound.

The CHANGELOG live smoke reports IRLS **converging in 8–12 iterations
across all six sources**. That's the expected ballpark for Huber on
right-skewed empirical distributions: the median initialization is
already on the right side of the contaminated mean, so most of the
residual movement happens in the first three or four steps and the
remaining iterations clean up sub-`s` adjustments.

## Why this is not just another L-estimator

The location-lens suite shipped over the last ~30 versions has a
specific algebraic shape. Sort the data; compute a fixed linear
combination of the order statistics; report the result. Mean, median,
midhinge, trimean, IQM, the four trimmed means TM-{10,20,25,30},
the two winsorized means WM-{10,20}, and the Harrell-Davis broadened
median all live in this family. They are **L-estimators**: linear
combinations of order statistics, weights determined entirely by
**rank**, not by value.

Hodges-Lehmann (v0.6.207) broke the pattern slightly by being an
**R-estimator** — it reduces to a single number derived from
pairwise Walsh averages, which depends on values, not just ranks,
but the dependence is purely combinatorial (count of `(i,j)` pairs
with `(x_i + x_j)/2 > t`, equivalent to the Wilcoxon signed-rank
inversion).

Power means (Lehmer `L_k`, harmonic, contraharmonic, quadratic) are
also value-dependent, but in a fixed closed-form way — `L_k =
sum(x^k) / sum(x^{k-1})`. One pass, no iteration, no sort even.

Huber is none of these. Huber is an **M-estimator**: defined as the
solution to a moment condition `sum psi((x - mu)/s) = 0`, which is
the score equation of an implicit M-likelihood. It has no closed
form. It must iterate. And — critically — the weight a row receives
is a function of **its residual at the current estimate**, not of
its rank in the sorted sample.

Practically that means **two rows at the same rank in two different
samples will receive different Huber weights** if the local density
of the samples around them differs. That is impossible for any
L-estimator: TM-25 will always give exactly zero weight to the rows
in the outer 25% on each tail and exactly `1/(n - 2*floor(0.25*n))`
weight to each interior row, regardless of whether those interior
rows are tightly clustered or wildly dispersed.

The bounded-influence property — `|psi_c(z)| <= c` — is what gives
Huber its **robustness**: any single row can move `mu` by at most
`c * s / sum(w_i)` worth, no matter how extreme. But unlike trimming,
Huber doesn't *throw the row away*. A row at `z = 10` still
contributes — it contributes `c * s` worth, signed, to the score
equation. It just can't dominate.

## Why all six sources show the same gap signs

`huberMeanGap = huber - mean` was negative for every source. Range:
`[-6.42M, -2.7K]`. `huberMedianGap = huber - median` was positive for
every source. Range: `[+0.6K, +2.77M]`. Every. Single. Source.

This is not an accident. It is the structural fingerprint of
**right-tail contamination on top of an underlying bulk
distribution**. To see why:

1. The arithmetic mean is unbounded-influence on the right. A row at
   `total_tokens = 56,000,000` (the well-known per-source heaviest
   single row from the W17 audit notes) contributes 56M / n to the
   mean. If `n = 299` for claude-code, that one row alone shifts the
   mean by ~187K tokens.
2. The median is unaffected by such a row — it can only move by the
   width of the bin straddling rank `n/2`.
3. Huber, with `c = 1.345` and `s = MAD / 0.6745`, will weight that
   56M row at `c * s / |x - mu|` — which is much less than 1 but
   strictly greater than 0.

So Huber will always sit *between* mean and median when the contamination
is one-sided right. If the contamination were one-sided left, the gap
signs would flip. If the contamination were balanced, both gaps would
shrink toward zero. If there were no contamination — a clean Gaussian
sample — Huber and mean would coincide to within sampling variance and
both would sit at the median by symmetry.

The fact that **all six sources show the same pattern** with no
exceptions tells you something the suite hasn't quite said before in
one number: every source's per-row token distribution is right-tail
contaminated. Not just claude-code (which is the obvious whale).
Not just codex (which has the heaviest tails by skewness). Even
**hermes** at `n = 257`, with a Huber estimate of 524,995 vs a mean
of 763,660 and median of 432,218, exhibits the same gap-sign
signature. Even **vscode-XXX** (the redacted IDE source) at
`n = 333`, where the absolute numbers are four orders of magnitude
smaller, shows `huberMeanGap = -2,748` and `huberMedianGap = +595`.
The right-tail contamination is **universal across the fleet** at
the per-row token level.

## What the cross-analyzer ladder tests do

The refinement commit (`78092a4`) shipped a cross-analyzer ladder
test suite that compares Huber against the four other location-class
analyzers shipped over the previous ~10 days: HD broadened median
(v0.6.208), Hodges-Lehmann (v0.6.207), TM-25, raw median, raw mean.
Eight ladder tests run in `sourcerowtokenmestimatorhuber.ladder`:

1. **Clean-data agreement.** On a clean Gaussian sample, all five
   estimators agree to within sampling variance. This is the
   reference point — it's what robustness *costs you nothing* to
   buy.
2. **Contamination ordering.** On a contaminated sample (mostly
   Gaussian, ~10% large outliers added), the estimators line up in
   a predictable order: median ≤ HD ≈ HL ≤ TM25 ≈ Huber ≤ mean.
   The test asserts this ordering holds on synthetic fixtures.
3. **Translation equivariance.** Adding a constant `delta` to every
   `x_i` shifts every estimator by `delta`. Held simultaneously
   with…
4. **Scale equivariance.** Multiplying every `x_i` by a positive
   constant `lambda` multiplies every estimator by `lambda`.
5–8. **Progressive contamination growth bound.** As the
   contamination fraction increases from 0 → 5% → 10% → 20%, the
   per-step movement of Huber, HL, HD, and TM25 stays bounded by a
   small multiple of the contamination magnitude, while the
   movement of the raw mean grows roughly linearly with the
   contamination fraction. The test asserts the ratio of mean-shift
   to robust-shift exceeds a threshold by the 20% contamination
   point.

That last family of tests is the most operationally useful one in
the codebase right now. It encodes the entire reason robust location
estimation exists, in four assertions. If a future refactor breaks
Huber's bounded influence, those four tests will scream.

## What `clippedRows` tells you

The analyzer reports `clippedRows`: the count of rows where the
final converged residual exceeds `c` in absolute value, i.e., the
count of rows whose Huber weight ended up `< 1`. Per the live smoke,
this number scales with both sample size and tail heaviness.
claude-code (heaviest tails, 299 rows) probably reported the high
end of the 8–101 range; hermes (lighter tails, 257 rows) the lower.
vscode-XXX, despite having `n = 333`, almost certainly reports a
small `clippedRows` because its distribution is so absolutely small
that the contamination, while present in shape, is small in
magnitude — Huber's bounded-influence regime barely needs to engage.

`clippedRows / n` is, informally, **the fraction of the sample that
behaved like an outlier under Huber's lens**. It is a per-source
**robustness budget consumption** number. If `clippedRows / n`
exceeds, say, 0.30, the source is so contaminated that Huber is
essentially behaving like a trimmed mean — at which point it's
worth asking whether the location parameter is a meaningful summary
at all, or whether the underlying distribution is bimodal or
multi-component.

This is a number the L-estimator suite never produced. TM-25 always
clips exactly 50% of the sample (25% per tail) regardless of where
the contamination actually is. Huber clips only the rows that
*deserve* clipping under the data-adaptive criterion `|z_i| > c`.
That asymmetry is the whole point of M-estimation.

## Sort keys and the implied user model

The analyzer ships with eight sort keys: `huber-desc`, `huber-asc`,
`mean-gap-desc`, `mean-gap-asc`, `median-gap-desc`, `median-gap-asc`,
`rows-desc`, `n-desc`. The two gap sorts are the operational ones.
`mean-gap-desc` ranks sources by **how much smaller the Huber estimate
is than the mean**, which is a direct ranking by **right-tail
contamination magnitude**. `median-gap-desc` ranks sources by
**how much larger Huber is than the median**, which is a ranking by
**bulk asymmetry**.

If you sort by `mean-gap-desc` on the live-smoke run, claude-code
(`-6.42M`) tops the table by a wide margin. opencode (`-2.39M`),
codex (`-2.75M`), and openclaw (`-0.94M`) sit in the middle. hermes
and vscode-XXX trail. That is **a directly interpretable ordering
of which sources have the most outlier-contaminated per-row token
distributions**, expressed in tokens, on the same scale as the data.
No previous lens in the suite produced this number with this
interpretation.

## Where Huber sits in the suite now

After v0.6.209 the location-lens family has the following
mechanism-class breakdown:

- **L-estimators (rank-based, fixed weights):** mean, median,
  midhinge, trimean, IQM, TM-{10,20,25,30}, WM-{10,20}, midrange,
  Harrell-Davis broadened median.
- **Power means (closed-form, value-based):** harmonic, geometric,
  contraharmonic, quadratic, Lehmer-`L_{-3..-2..-1..0..1..2..3..4..5..6..7..8..9..10..11..12}`.
- **R-estimators (rank-based, derived from rank-test inversion):**
  Hodges-Lehmann pseudo-median.
- **M-estimators (data-adaptive weights, iterative):** Huber `c=1.345`.

That last bucket has exactly one occupant. Tukey biweight (a
**redescending** M-estimator with bounded *and* eventually-zero
influence) is the obvious next addition; Andrews wave is the
trigonometric cousin; Hampel three-part is the most bureaucratic.
Each of those is a one-version increment with a parallel ladder
test suite, and the ladder tests themselves now have a tested,
shipped scaffold (`sourcerowtokenmestimatorhuber.ladder`) that the
next M-estimator can extend rather than rebuild.

## What this means for the dispatcher rhythm

The pew-insights `feature` ticks have been shipping at roughly 0.5
per dispatcher tick for the last week, with the canonical
4-commit / 2-push pattern (the metapost on push-to-commit ratio
asymmetry already documented this in detail at
`fac6227`). v0.6.209 is the **22nd consecutive `feature` release in
the location-lens arc**, counting back from v0.6.188 (the
contraharmonic mean release that anchored the start of the modern
power-mean ladder).

That arc has now taxonomically completed three of the four classical
robust-statistics estimator families: L, R, and M. Only S-estimators
(scale-based, like LMS / LTS) remain unaddressed in the per-row
token location context — and S-estimators are arguably out of scope
for a location lens, since their natural target is scale not
location. So in a real sense, **the location-lens arc is structurally
complete with v0.6.209**. Future feature ticks in this arc will be
adding *flavors* (more M-estimators, more L-estimator alphas, more
power-mean rungs) rather than *new mechanism classes*.

That's a meaningful inflection point for a tool that has been
shipping a new analyzer roughly every 90 minutes for ten days. The
next interesting structural move is probably to move the lens from
location to **scale dispersion** with a similar four-class breakdown
(L, R, M, S) — at which point the IRLS scaffolding shipped here
becomes load-bearing for the M-estimator-of-scale entry.

## The single number to remember

claude-code, n=299: `huber = 5,095,026.75`, `mean = 11,512,995.95`,
`median = 3,319,967.00`. The Huber estimate sits 6.42M tokens
*below* the mean and 1.78M tokens *above* the median, exactly where
you would predict a bounded-influence estimator to land on a
right-tail-contaminated distribution. The whole shape of the
distribution is summarized in those three numbers, and the tooling
to compute them across six sources / 1898 rows in one shot now
exists, ships with 38 dedicated tests, and runs in under a second
against `~/.config/pew/queue.jsonl`.

That's the quiet headline of v0.6.209: **the first lens in the suite
where the answer depends on more than just the order statistics.**
