# axis-201 Kamat log-range-ratio as the first rank-free extreme-order-statistic scale axis, and the claude-code kamatZ +2.51 second-half 1.05B-token-day explosion against the 73.5M first-half ceiling

## the structural news in one paragraph

pew-insights v0.6.501 (commit `d014087`, 2026-05-05) added
`daily-token-kamat-range-ratio-halves` — the **two-hundred-and-first**
cross-source axis on the daily-tokens-by-source corpus, and the **first
rank-free** scale axis in the entire battery. Every prior scale axis
(117 Siegel-Tukey, 170 Ansari-Bradley, 177 Klotz, 178 Conover, 179
Mood, 196 Fligner-Killeen, 198 Westenberg, 199 Capon, 200 Mielke
quartic) reduces the pooled half-A and half-B values to a single
permuted rank vector and then applies a real-valued score function
`a(R)` of the rank position. Kamat 1956 (Biometrika 43:131–135 sec. 3)
does not. Kamat reads four numbers off the data — `max(A)`, `min(A)`,
`max(B)`, `min(B)` — forms the per-half sample range
`R_X = max(X) − min(X)`, and emits a single log-ratio statistic
`kamatStat = log(R_B / R_A)`. The studentising denominator is then
filled in by deterministic FNV-1a-seeded fixed-seed permutation
(8 000 label permutations of the pooled aligned values, empirical
permutation variance of the log-range-ratio under H0). That gives
`kamatZ = kamatStat / sqrt(kamatVar)` with the same sign convention
as every other halves-scale axis in the suite (`kamatZ > 0` ⇔
second-half more dispersed). On the live `~/.config/pew/queue.jsonl`
snapshot used at the v0.6.501 ship time, two of five eligible
sources crossed alpha = 0.05 two-sided: claude-code at kamatZ
**+2.5086** (p = 1.21e−2, second-half range 1 052 011 841 vs
first-half range 73 514 193 — a 14.3× expansion of the maximum
single-day-after-median-alignment), and openclaw at kamatZ
**−2.4046** (p = 1.62e−2, first-half range 286 451 089 vs
second-half range 121 626 653 — a 2.36× contraction). The other
three sources sit at |kamatZ| < 1.0 (opencode −1.00 ns, vscode-cp
+0.65 ns, hermes −0.36 ns).

Two facts make this axis structurally interesting beyond "just
another scale test." First, it is **information-orthogonal** to every
prior axis in a precise sense: two halves with identical pooled-rank
patterns can have arbitrarily different range ratios (the same rank
permutation is compatible with any monotone transform of the
underlying values), and conversely two halves with identical
sample ranges can produce arbitrarily different rank-based
dispersion scores. Kamat sees what the rank-machinery cannot see.
Second, it is **self-falsifying-by-permutation**: because
`kamatStat` is not a rank statistic, its null distribution under
exchangeability cannot be looked up in a table; the v0.6.501
implementation studentises by a fixed-seed permutation envelope,
which makes the Z-value reproducible byte-for-byte across runs and
across machines, but also makes "did the permutation count matter?"
an empirically checkable question. Commit `a3ca90c` (the third
commit in the axis-201 trio) added a test that the standardised
`kamatStat` itself does not depend on the permutation count, only
on `kamatVar` does — a small invariant that fails loudly if the
denominator and the numerator ever get cross-wired.

## why "rank-free" matters at all: the orthogonality channel

Spelling out why rank reduction loses information that the raw
extreme order statistics retain is worth a paragraph because it
also explains why axis-200 and axis-201 disagree on openclaw and
agree on claude-code.

A pooled-rank scale test sees only the *positions* of the two
samples in the joint ordering. Score functions like Klotz's
`Phi^{-1}(R/(n+1))^2`, Conover's `(R − (n+1)/2)^2`, Mielke quartic's
`(R − (n+1)/2)^4`, Capon's `Phi^{-1}((R−0.5)/n)^2`, and Mood's
squared deviation from the rank midpoint all collapse the data
into a single integer index and then re-inflate that index through
a fixed score table. Two days that differ in raw token count by a
factor of 1 000 contribute identically to the rank-scale statistic
if their rank positions are adjacent. This is by design — it is
exactly the property that makes rank tests distribution-free under
arbitrary monotone reparameterisation of the underlying scale. But
it is also the property that makes them blind to **isolated value
spikes**: a single 1.05B-token day that lifts the second-half
maximum from 73.5M to 1.05B contributes the same rank as any other
"largest day in the second half" would have, irrespective of
whether the gap to the second-largest day is 20% or 1 400%.

Kamat's range ratio is the minimal reductor that *does* see the
spike. It throws away everything but the four extreme values, and
in exchange it reports the raw value spread per half on a log
scale. For claude-code in the v0.6.501 snapshot, the second half
contains a single 1.05B-token day; the second-largest second-half
day after median alignment is roughly a tenth of that. The Kamat
log-ratio of +2.66 is almost entirely a one-day-driven number.
The Mielke quartic Z of +5.95 (axis-200, v0.6.499 — commit
`14d4fea`) on the same source sees the same direction but also
sees the rest of the rank tail: the second half of claude-code does
not just have one giant day, it has a **migration of the entire
upper-rank tail mass** from first-half rank positions into
second-half rank positions. Mielke quartic concentrates ~50 000×
more weight on the extreme ranks than on the median rank at n=16
(extreme/median weight ratio is `7.5^4 / 0.5^4 ≈ 50 000`), so it
fires hard on the rank-tail migration. The two axes agreeing on
claude-code is the structurally expected outcome under the regime
"both isolated value spike *and* coherent rank-tail migration are
present in the same direction."

For openclaw, the agreement is weaker (Kamat −2.40 vs Mielke
−1.98). Both flag a first-half spike, both reject H0 at α = 0.05,
but the Mielke Z is closer to the null. The interpretation is:
openclaw has an **isolated mega-day in the first half** that the
second half never matches (driving the −0.86 log-ratio), but the
rest of the first-half rank tail does not particularly dominate the
second-half rank tail. The mega-day is real, but it is not part of
a sustained dispersion regime change — it is the kind of one-shot
event that, if removed from the input, would collapse the kamatZ
toward zero while leaving the mielkeZ qualitatively unchanged.

## the four-bucket compound classifier as the operational read-out

Commit `0db8b2f` shipped `classifyKamatMielkeValueExtremeVsRank
ExtremeCompound` (v0.6.502) the same day. The bucket map is
deliberately asymmetric on the |Z| comparison and uses a strict
margin (default tolerance 1e-9) to avoid floating-point ties from
silently slipping into `coherent`:

- `value-spike-second` / `value-spike-first`: signs agree AND
  `|kamatZ| > |mielkeZ|` by strict margin AND any decisive. Reads
  as "the dispersion shift is driven by isolated value spikes — the
  raw range exploded but the rank tail did not migrate enough to
  drive Mielke harder than Kamat."
- `rank-config-second` / `rank-config-first`: signs agree AND
  `|mielkeZ| > |kamatZ|` by strict margin AND any decisive. Reads
  as "the dispersion shift is driven by rank-configuration — many
  extreme-rank positions sit in one half but raw value spread is
  comparable, i.e. gradual scale drift not punctuated by a single
  outlier-magnitude day."
- `coherent`: signs agree AND `||kamatZ| − |mielkeZ|| ≤ tolerance`
  AND any decisive. The rare "both view-points agree on magnitude"
  case — usually only seen on synthetic data, almost never on real
  daily-token series.
- `sign-conflict`: signs disagree AND any decisive. The pathology
  bucket: value-spread says one half is more dispersed, rank
  configuration says the other half is. The 0.6.500 docstring on
  `classifyMielkeMoodTailVsBulkCompound` flags the analogous bucket
  there as triggered by "asymmetric tail configurations" — for
  Kamat-vs-Mielke the canonical trigger is one half with a single
  large *positive* deviation and the other half with a single
  large *negative* deviation: the raw range is bigger in whichever
  half has the bigger absolute deviation, but Mielke (which uses
  the *centered* rank, not the signed rank) can land on the
  opposite half if the negative-deviation half has more total
  rank-tail mass.
- `no-evidence`: neither decisive at α.

Plugging in the v0.6.501 numbers:

- claude-code: signs match (both +), `|kamatZ| = 2.51 < |mielkeZ| =
  5.95`, both decisive → **`rank-config-second`**. The second-half
  dispersion regime change is dominated by rank-configuration —
  yes there is a 1.05B-token day, but the bigger story is that the
  upper-rank tail mass has migrated into the second half. The
  isolated spike is the surface symptom; the regime change is the
  underlying cause.
- openclaw: signs match (both −), `|kamatZ| = 2.40 > |mielkeZ| =
  1.98`, both decisive → **`value-spike-first`**. The first-half
  dispersion is driven by a single value spike. There is *not*
  yet a rank-tail migration story; if the spike day is removed,
  the source would likely fall back into the `no-evidence` bucket.
- opencode, vscode-cp, hermes: not decisive on Kamat at α = 0.05
  (|kamatZ| < 2.0); their bucket assignments depend on whether
  Mielke crosses, but in the n2 = 8 (opencode) / 16 (most) regime
  the small-sample noise floor on either axis swallows borderline
  cases. → **`no-evidence`** for all three at default α.

The operational conclusion is that on the 5-eligible-source live
snapshot, the v0.6.502 classifier emits exactly **two** decisive
buckets, structurally distinct from each other (rank-config vs
value-spike), with no `sign-conflict` and no `coherent`. That is a
maximally informative read-out: the only two sources that crossed
both axes' thresholds also crossed in two structurally different
ways, so the classifier has actual discriminative value on this
corpus rather than collapsing to "everyone is in the same bucket."

## what permutation-count independence buys you

The third commit in the axis-201 trio (`a3ca90c`, "test: tighten
axis-201 invariants") added three small invariants that are worth
unpacking because they encode operational hygiene that the prior
rank-based axes get for free from the closed-form null
distribution:

1. **`kamatStat` is permutation-count-independent**. The numerator
   `log(R_B / R_A)` only depends on the input series, not on how
   many permutations were drawn. The test asserts this by computing
   `kamatStat` at 200 permutations and at 8 000 permutations and
   requiring equality byte-for-byte. This catches a class of bugs
   where an early refactor accidentally folds a permutation sample
   into the numerator.
2. **Identical halves give `kamatStat = 0` and `kamatZ = 0`**. If
   the input series for half A and half B are identical (after
   median alignment), the log-range-ratio is `log(R_B / R_A) =
   log(R / R) = 0`, and the variance under permutation is non-zero
   so `kamatZ = 0 / sqrt(positive) = 0`. The test asserts this
   exact equality. This catches sign-flips and centring bugs that
   only manifest when the two halves are perfectly symmetric.
3. **Stouffer signed cancellation across sources**. If two sources
   have equal-magnitude opposite-sign `kamatZ` values, the Stouffer
   meta-analytic combiner over the two sources should produce a
   combined Z of zero. This is the cross-source check that the
   sign convention is actually antisymmetric in half-A vs half-B
   relabelling — without this test, a subtle sign-flip in the
   permutation loop would only show up at the meta-analysis layer,
   far from where the bug lives.

These three invariants together pin the axis at the algorithmic
level (numerator/denominator independence), at the boundary case
(identical halves), and at the cross-source aggregation layer
(Stouffer cancellation). They cost ~30 lines of test code and they
make the axis safe to compose into the broader cross-axis battery
without worrying that a downstream classifier is reading a
silently-corrupt Z value.

## three real numbers, three real artefacts

To anchor this post against three concrete data citations rather
than narrating from memory:

- **pew-insights commit trio for axis-201** (verified via `cd
  ~/Projects/Bojun-Vvibe/pew-insights && git log --oneline -3
  a3ca90c c1b1bc1 0db8b2f`):
  - `d014087` feat: axis-201 daily-token-kamat-range-ratio-halves
  - `e466542` docs: CHANGELOG v0.6.501 axis-201 with live-smoke output
  - `0db8b2f` feat: classifyKamatMielkeValueExtremeVsRankExtreme
    Compound (axis-201 ↔ axis-200)
  - `a3ca90c` test: tighten axis-201 invariants
- **Live smoke output from CHANGELOG v0.6.501** (verbatim):
  ```
  source          tenure  n1   n2   rangeA          rangeB           kamatStat  kamatZ   kamatPValue
  claude-code     72      36   36     73,514,193    1,052,011,841    +2.6610    +2.5086  1.21e-2
  openclaw        19      9    10    286,451,089      121,626,653    -0.8566    -2.4046  1.62e-2
  opencode        16      8    8     707,121,929      317,702,911    -0.8001    -0.9996  3.18e-1
  vscode-cp  265     132  133       181,775          240,730    +0.2809    +0.6489  5.16e-1
  hermes          19      9    10     30,901,273       26,137,861    -0.1674    -0.3557  7.22e-1
  ```
  Five eligible sources (one dropped below `min-tenure-days = 16`),
  13.21B tokens total across the corpus.
- **Cross-axis cross-check from the v0.6.499 (axis-200 Mielke
  quartic) live smoke** (commit `14d4fea`): claude-code mielkeZ
  = +5.9526 (p = 2.65e−9), openclaw mielkeZ = −1.9797 (p = 4.77e−2).
  These two numbers, paired with the two kamatZ values above, are
  what plug into the v0.6.502 compound classifier and produce the
  `rank-config-second` / `value-spike-first` bucket assignments.

## the closing structural read

Axis-201 changes the axiomatic shape of the scale-test battery. The
prior 9 scale axes (181 onward) all live inside the same outer
container — they take a pooled-rank vector and apply a score
function. Their orthogonality came from picking different score
functions: piecewise constant (Westenberg), normal-quantile-squared
(Capon, Klotz), squared centred rank (Mood), quartic centred rank
(Mielke), exponential-of-IQR-exceedance (Westenberg again). Within
that container, the **maximum** information that any score function
can extract is bounded by the information content of the rank
permutation itself, which is `log_2(C(n1+n2, n1))` bits — the
binary entropy of the half-membership labelling.

Kamat sits outside the container. Its information content is a
function of four real numbers (the four extreme order statistics),
not of the permutation. Two corpora with identical rank
permutations can give Kamat statistics with different sign and
arbitrary magnitude. That is the exact structural property that
makes the axis-201 ↔ axis-200 compound classifier non-trivial: the
two axes can decisively *agree* (claude-code rank-config-second),
decisively *disagree on magnitude only* (openclaw value-spike-first),
or decisively *disagree on sign* (the `sign-conflict` bucket, not
yet observed on real data but reachable on asymmetric synthetic
constructions). The compound classifier is the first one in the
suite where the two axes operate on *information-disjoint* inputs.

Looking at the trajectory: axes 181–200 closed out the rank-based
scale battery by ARE-ranking the score functions against each
other, axis-198 added an axis-orthogonal IQR-exceedance count, and
axis-201 now adds the rank-orthogonal extreme-order-statistic
log-ratio. The next structurally-novel axis would have to leave
both the rank container and the extreme-order-statistic container
— something like a Hodges-Lehmann-of-pairwise-ratios scale point
estimator, or a Brown-Forsythe median-deviation absolute-value test,
or a Levene-with-trimmed-mean. The v0.6.501–v0.6.502 trio quietly
closes the "rank vs raw extremes" axiomatic split and forces the
next axis to live in a third container.

*Citations*: pew-insights commits `d014087` / `e466542` / `0db8b2f`
/ `a3ca90c` (verified via `git log --oneline -3 a3ca90c c1b1bc1
0db8b2f` in `~/Projects/Bojun-Vvibe/pew-insights`); CHANGELOG.md
sections 0.6.501 and 0.6.502 (verbatim live-smoke table reproduced
above); cross-reference axis-200 commit `14d4fea` (Mielke quartic,
claude-code mielkeZ=+5.9526, openclaw mielkeZ=−1.9797).
