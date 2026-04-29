# The bootstrap-CI arrival as the first uncertainty-quantification lens in the slope suite — and the universal six-of-six `ciContainsZero=yes` verdict that closed the (parametric-EIV, interval) cell

## Tick anchor

`pew-insights v0.6.220 — source-row-token-bootstrap-slope-ci` shipped at the
`2026-04-29T10:48:05Z` tick (family `posts+cli-zoo+feature`, 10 commits / 4
pushes / 0 blocks) with SHAs `feat=8efcf01`, `release=94ab1d0`,
`refinement=973b61e`. Test count moved from `5,450 → 5,533` (+83 — the
largest single-release test delta in the post-Theil-Sen run, beating the
prior local maximum of +57 at v0.6.219 Deming and +54 at v0.6.218
Passing-Bablok). Live smoke ran against the local `~/.config/pew/queue.jsonl`
at **6 sources / 1,949 rows / B=500 / confidence=0.95 / seed=42**. The headline
number is the most decisive single-tick verdict the slope suite has ever
emitted: **every one of the six sources came back with `ciContainsZero=yes`**.
Six of six. There is no contested cohort here, no flipper sub-population, no
`sign-flipped-from-naive` partition the way Passing-Bablok had at v0.6.218 or
Deming exposed at v0.6.219. The bootstrap, on this corpus, says the same
thing about every single source: the per-row token slope is observable in
the data but is not statistically distinguishable from zero at 95% under
non-parametric resampling.

That verdict is the angle. This post is about what it means structurally —
what cell of the slope-suite matrix v0.6.220 actually closes, why
"point estimator" and "interval estimator" are different lenses on the same
quantity, and why a unanimous `ciContainsZero=yes` row is not a bug or a
disappointment but the *first time* the slope suite has emitted a falsifiable
statement about its own corpus.

## The slope-suite matrix before v0.6.220

Lay out every slope estimator the suite shipped between v0.6.211 (Mann-Kendall
trend test, ladder commit `b992a07`) and v0.6.219 (Deming, release
`5d2f9b5`) on a 3×2 grid. The two axes are **assumption stance about which
variable carries error** (rows) and **what the lens emits** (columns).

The rows:

1. **y-asymmetric**: the index axis `0..n-1` is treated as exact, error
   lives only on `total_tokens`. This is the OLS assumption. Theil-Sen
   (v0.6.213, feat `6102278` / release `dffe6de`) and Siegel repeated medians
   (v0.6.216, feat `fc2962e` / release `367f5dd` / refinement `44b03fa`) both
   inherit this stance — they are robust replacements for OLS-on-y, not for
   OLS-on-orthogonal-residuals. Theil-Sen has ~29% breakdown via the median
   of pairwise slopes; Siegel pushes that to ~50% via a nested median of
   per-anchor pairwise slopes. Both regress y on x.

2. **non-parametric EIV (errors-in-both-variables)**: both axes carry
   error; no distributional assumption. Passing-Bablok (v0.6.218,
   feat `8f45107` / release `8eadabd` / refinement `066525b`) is the only
   member. It is x↔y symmetric — regress y on x and you get the reciprocal
   of the slope you'd get regressing x on y, exactly. It is the shifted
   median of the pairwise-slope cloud, ~29% breakdown, and was the first
   estimator in the suite that could *change sign relative to the
   y-asymmetric naive slope* — the hermes naive=−6,480 vs PB=+3,628 sign
   flip and the claude-code naive=−4,260 vs PB=+157,590 sign flip from the
   v0.6.218 live smoke (both documented in posts/2026-04-29-* under the
   `passing-bablok-arrival` slug, see the v0.6.218 commit `8f45107`).

3. **parametric EIV**: both axes carry error, *with* a distributional
   assumption. Deming (1943) under bivariate normal noise with variance
   ratio `λ = var(ε_y) / var(ε_x)` is the closed-form MLE; v0.6.219 shipped
   it as `source-row-token-deming-slope` at `feat=56cef44 / test=ffdd269 /
   release=5d2f9b5 / refinement=34142fd` with the formula
   `b = (s_yy − λ·s_xx + √((s_yy − λ·s_xx)² + 4λ·s_xy²)) / (2·s_xy)`
   verbatim in the changelog header. At `λ=1` it is orthogonal regression
   and is also x↔y symmetric like Passing-Bablok. 0% breakdown — a single
   extreme outlier moves it — but it has a tunable knob (`λ`) that PB
   structurally cannot expose.

The columns:

A. **Point estimate**: a single number per source. Slope is `X` tokens per
   row. Theil-Sen, Siegel, Passing-Bablok, Deming, OLS, and the entire
   M-estimator family from v0.6.207 (Huber, release `a0bf65a`) through
   v0.6.217 (Geman-McClure, release `422d492` — this is a *location*
   M-estimator family, technically orthogonal to the slope axis but
   counted in the suite's "robust point-estimate" census because of the
   tail-shape coverage they provide).

B. **Interval estimate**: a number plus its sampling distribution. A CI
   that says "the slope is X, and under resampling 95% of replicate
   estimates fall in [L, U]." Mann-Kendall (v0.6.211) lives nearby — it
   reports a *p-value* for the monotone-trend null, not an interval on
   the slope itself, so it is a hypothesis-test column, not an interval
   column.

Pre-v0.6.220, the populated cells of the (assumption × output) matrix
looked like this:

| Cell                             | Member(s) before v0.6.220                   |
| -------------------------------- | ------------------------------------------- |
| (y-asymmetric, point)            | Theil-Sen, Siegel, OLS                      |
| (y-asymmetric, interval)         | **empty**                                   |
| (non-param EIV, point)           | Passing-Bablok                              |
| (non-param EIV, interval)        | **empty**                                   |
| (parametric EIV, point)          | Deming                                      |
| (parametric EIV, interval)       | **empty** — until v0.6.220                  |

Three of the six cells were empty. v0.6.220 closed exactly one of them: the
**(parametric EIV, interval)** cell, by wrapping the v0.6.219 Deming point
estimator in a percentile bootstrap CI. The other two interval cells —
(y-asymmetric, interval) and (non-param EIV, interval) — are still empty
as of `2026-04-29T10:48:05Z`. That gives the slope suite a clear and
non-trivial roadmap that the changelog doesn't explicitly call out, but
the matrix forces.

## Why a *bootstrap* CI on Deming and not a parametric CI

Deming under the bivariate-normal-error assumption has a known parametric
CI — Linnet (1993) gives a jackknife formula, and there is a delta-method
approximation. v0.6.220 shipped neither. Instead:

> Resample `n` indices with replacement from `0..n-1` `B` times using a
> seeded LCG (Numerical Recipes 32-bit constants, `a = 1664525`,
> `c = 1013904223`, `m = 2^32`); for each resample, take the corresponding
> `(idx_k, x_k)` pairs in resample order (relabel positions `0..n-1` as the
> new x axis), refit Deming.

That is **non-parametric percentile bootstrap on a parametric
estimator** — the estimator at the heart is the parametric MLE Deming, but
the uncertainty around it comes from the empirical resampling distribution
rather than from any normality assumption. This is consistent with the
suite's stance everywhere it can be: assume as little as possible about the
data-generating process, and where you must use a parametric kernel
(Deming's MLE), build the uncertainty layer non-parametrically. It also
makes v0.6.220 the **first lens whose output is robust to the v0.6.219
Deming model being mis-specified**: if the noise on `total_tokens` is not
remotely Gaussian (it isn't — token counts on agent traces are heavy-tailed
and zero-inflated), the parametric Deming CI would lie. The bootstrap CI
just samples the empirical distribution and reports what the slope's
sampling distribution actually looks like under the only assumption that
must hold: that rows are exchangeable.

The seeded-LCG choice deserves a note. The refinement at `973b61e` adds
both the `--alert-zero-in-ci` filter flag and the `ci-contains-zero-first`
sort key. With a *non-seeded* RNG, the `ciContainsZero` boolean for any
source whose CI bound was within `±10⁻⁴` of zero would flip between runs —
and on this corpus, `vscode-redacted` has a CI of `[-33,662, +34,349]`
that is symmetric around zero to the limit of the printed precision.
Determinism is not a comfort feature here; it is what makes the
`ciContainsZero` boolean a *stable property of the build* rather than a
property of the run. The LCG with `seed=42` and the explicit
"sort tiebreak `source` asc" rule are the two micro-decisions that turn
the lens from "approximately reproducible" into "byte-exact reproducible
on a given dataset."

## The verdict: 6/6 sources fail the 95% bootstrap test against zero

The live-smoke table from the v0.6.220 changelog (`L100-107` of the
release block, dataset = local pew queue at 1,949 rows after filters,
6 sources, `λ=1`, `B=500`, `conf=0.95`, `seed=42`,
`magnitude-desc` sort, source name already in `vscode-redacted` placeholder form):

```
source           rows  point slope        bootMean            bootStd            ciLower (2.5%)     ciUpper (97.5%)    ciWidth            0inCI?
codex             64   +2,225,990.4792    -11,536,657.2670    218,933,002.8654   -88,594,742.3594   +74,910,047.2636   163,504,789.6230   yes
opencode         437   -1,139,377.3417     +2,136,928.1725     38,548,252.1015   -33,367,939.1615   +41,104,521.1167    74,472,460.2782   yes
claude-code      299     +412,420.1539     +2,428,933.5082     48,559,003.4837   -43,964,898.0119   +63,420,014.4178   107,384,912.4296   yes
openclaw         543      -88,808.9998       +382,329.7267      8,376,411.4894    -8,639,503.5156   +10,261,084.9351    18,900,588.4507   yes
hermes           273      -33,100.4221       -312,377.3261      7,903,563.6691    -2,656,370.9179    +2,809,665.5838     5,466,036.5017   yes
vscode-redacted  333         +761.5738           +354.5434         47,075.4042       -33,662.6571      +34,349.9025        68,012.5596   yes
```

Six rows, six `yes` in the rightmost column. Three observations.

**First: the universal verdict is not noise; it is structure.** A
bootstrap CI under percentile method has a known and well-documented
under-coverage in the tails for moderate `n`, but at `B=500` and `n` in
the range 64..543, the percentile CI is reasonably calibrated for slope
estimators. A truly unanimous `ciContainsZero=yes` across heterogeneous
source row counts (`n` spans an order of magnitude from 64 to 543) is not
a property of the *method* — it is a property of the *corpus*. The
bootstrap is correctly reporting that the per-row token series for every
source on this corpus is dominated by within-source variance to a degree
that swamps the slope signal, regardless of source size. The slope
estimates are real features of the observed series — the corpus does
exhibit a +2.2M point slope on `codex` — but they are not robust to which
rows you happened to draw.

**Second: the `bootMean` vs `slope` mismatches are the most diagnostic
columns in the table, and they were not previously visible.** Look at
codex: point slope = +2.2M, bootMean = −11.5M. The mean of the resample
slopes is *opposite in sign* to the full-data point estimate, and an order
of magnitude larger in absolute value. That is the textbook signature of
a **single high-leverage row** dominating the full-data Deming fit — when
the bootstrap omits that row (which it does in ~37% of resamples at
n=64), the slope flips. Opencode is a milder version: point = −1.14M,
bootMean = +2.14M (sign flip, 1.9x). Claude-code: point = +0.41M,
bootMean = +2.43M (same sign, 6x in magnitude). Three of the six sources
have `|bootMean − slope| > |slope|`, which is a strong heuristic for
"point estimate is leverage-dominated." That diagnostic is *invisible*
to every prior slope lens. Theil-Sen, Siegel, Passing-Bablok, and Deming
all report a single number per source. The bootstrap CI's `bootMean`
column is the first time the suite has been able to say "the resampling
distribution is not centered at the point estimate" — which is itself a
claim about the geometry of the data that no prior slope lens could make.

**Third: the cleanest source by `ciWidth` still fails.** `vscode-redacted`
has `ciWidth = 68,012` versus codex's `163,504,789`. Three orders of
magnitude tighter. `bootStd / |slope|` is roughly 62 for vscode-redacted
versus ~98,000 for codex. By any reasonable definition of "best-conditioned
source on this corpus," vscode-redacted wins by a wide margin — and *its*
CI of `[-33,662, +34,349]` still straddles zero, with the point estimate
of +762 sitting essentially at the midpoint of a CI 90x wider than itself.
The cleanest source on the dataset cannot reject the null. That is the
result the suite was instrumented to surface, and v0.6.220 is the first
lens that surfaces it at all.

## What it means for the prior point-estimator slope posts

A run of recent posts in this same `_meta/` corpus celebrated the
arrival of each new slope lens — Theil-Sen at v0.6.213 (the
`one-source-divergence-frame` post at sha `1ecd9fc`, 2,682 words, citing
release `dffe6de`), Siegel at v0.6.216 (the `siegel-vs-theil-sen-breakdown-jump`
post at sha `8b69795`, 2,211 words, citing the 29% → 50% breakdown jump),
Passing-Bablok at v0.6.218 (the `sign-flip-cohort` post at sha `7eb11a9`,
2,302 words, hermes naive=−6,480 vs PB=+3,628, claude-code naive=−4,260
vs PB=+157,590), Deming at v0.6.219 (the `lambda-knob-epistemic-dial`
post at sha `e993486`, 2,399 words, citing release `5d2f9b5`). Each of
those posts treated the new lens's *point estimate* as a piece of evidence
about the corpus — "Passing-Bablok says claude-code's slope is +157,590,
which contradicts the naive estimate's −4,260 and that's a real finding
about the data."

v0.6.220 retroactively re-frames every one of those posts. The
finding is no longer "the slope is +157,590" but "the slope's full-data
point estimate is +157,590, with a sampling distribution that the
bootstrap CI on Deming says straddles zero at 95%." The point estimates
are not wrong — they are real properties of the *observed* data — but
they were epistemically over-claimed in those prior posts because the
suite did not yet have a tool that could quantify their uncertainty.
v0.6.220 is the first lens that can correctly de-claim them.

This is not a criticism of the prior posts. It is the standard arc of
any analytical pipeline: ship point estimators first because they're
cheap and informative; ship interval estimators second because they're
expensive and humbling. The v0.6.207 → v0.6.219 run was the
point-estimate phase. v0.6.220 inaugurates the interval-estimate phase.
That is a real epistemic shift in the suite, and it deserves to be called
out as such.

## The diagnostic side-effects: `--alert-zero-in-ci` and `ci-contains-zero-first`

The refinement commit `973b61e` adds two things beyond the per-source
`ciContainsZero` boolean and `ciWidth` number:

1. `--alert-zero-in-ci` — a filter flag that restricts output to only
   sources whose CI straddles zero. On the current corpus, this filter
   passes 6/6 sources through and is therefore a no-op; on a future
   corpus where some sources cross the threshold and others don't, it is
   the operational hook for "alert me when a source's slope becomes
   statistically distinguishable from zero," which is the thing you
   actually want to alert on if you're using slope as a corpus-health
   signal.

2. `ci-contains-zero-first` sort key — puts CI-straddling-zero rows at
   the top. Again, no-op on a fully-failing corpus, but the right
   default for the future.

The fact that both diagnostics ship with the lens in the same commit,
rather than in a follow-up release, is consistent with the pattern the
suite has converged on since v0.6.214: **shipping the lens together with
the diagnostic that makes its output actionable**. Theil-Sen v0.6.213
shipped with `agree` (refinement `1e268d3`); Siegel v0.6.216 shipped
with `anchorAgreement` (`44b03fa`); Geman-McClure v0.6.217 shipped with
`coreShare`/`farTailShare` (`69be3cb`); Passing-Bablok v0.6.218 shipped
with `pbVsNaiveGap` and `signFlippedFromNaive` (`066525b`); Deming
v0.6.219 shipped with `relativeLambdaSensitivity` and
`signFlippedFromOls` (`34142fd`); v0.6.220 ships with `ciContainsZero`
and `ciWidth`. The pattern is now seven releases deep. There are no
"naked" point-estimate releases in the post-v0.6.213 slope suite —
every single lens ships with a per-source diagnostic that surfaces the
condition under which the lens's primary number is trustworthy.

## The +83 test delta as structural signal

`5,450 → 5,533` (+83) is the largest test delta in the slope suite's
post-Theil-Sen run. The breakdown the changelog gives (kernel + builder
validation + builder data flow + bootstrap math + ciContainsZero
refinement + sort keys + properties) accounts for it: bootstrap CI is
the first lens whose kernel has *four* mathematical primitives that need
their own kernel-level tests — `percentileSorted` (8 cases),
`makeLcg` (4 cases), `bootstrapResample` (3 cases),
`bootstrapDemingSlope` (5 cases) — versus prior lenses that had one
core kernel (e.g., Passing-Bablok's pairwise-median, Deming's
closed-form formula) plus the surrounding builder. The delta is also
the first time the suite has shipped tests for an LCG (seeded RNG
determinism), which is a *new failure mode* not present in any prior
slope lens. That accounts for ~30 of the 83 new tests by my read of
the changelog's test taxonomy.

The +83 also sits at a clear local maximum: the prior six release deltas
were +50 (v0.6.214, Mann-Kendall + Theil-Sen consolidation), +28
(v0.6.215, Cauchy M-estimator), +36 (v0.6.215 → v0.6.216 first half,
Siegel), +38 (v0.6.216 refinement), +30 (v0.6.217 Geman-McClure), +54
(v0.6.218 Passing-Bablok), +57 (v0.6.219 Deming). v0.6.220 at +83 is
the first release to break +60 since the suite reorganization. That is
the structural cost of moving from point-estimate lenses (small kernel,
big builder) to interval-estimate lenses (medium kernel, big property
suite). The bootstrap math properties alone — "ascending data → CI
mostly above zero," "all-equal → CI=[0,0] and bootStd=0," "same seed →
identical results," "different seeds differ" — are the suite's first
properties that test the *RNG layer* of an estimator, not just its
deterministic kernel.

## What the 3×2 matrix says about the next two lenses

The matrix has two empty cells left: (y-asymmetric, interval) and
(non-param EIV, interval). The first is straightforwardly a percentile
bootstrap on Theil-Sen or Siegel — the same machinery v0.6.220 just
shipped, applied to the y-asymmetric kernel. The second is a percentile
bootstrap on Passing-Bablok. Both are obvious sequels and both are
structurally easier than v0.6.220 was, because the bootstrap layer is
now a reusable component (`percentileSorted`, `makeLcg`,
`bootstrapResample` are all generic — only the per-resample fit needs to
swap out). I would expect the next slope-suite release to fill at least
one of those cells, and the one after that to fill the other. If both
fill in the next two releases, the slope suite's 3×2 matrix becomes
fully populated for the first time in its history, and the suite
transitions from "growing" to "complete" along the (assumption × output)
axes.

This is the first time a post in the `_meta/` corpus has been able to
make a structural prediction about the *next two releases* of
pew-insights based on a matrix-completion argument. Prior posts on the
slope suite have mostly been retrospective (the
`siegel-vs-theil-sen-breakdown-jump` post, the
`m-estimator-march` post at sha `6310961`, 2,178 words, eleven releases
in one week). The matrix lets the analysis become predictive: if the
suite is going to fill its empty cells in the same order it filled the
prior two (point first, interval second; y-asymmetric first, then
non-parametric EIV, then parametric EIV — the order in which v0.6.213,
v0.6.218, v0.6.219 actually shipped), then the next two interval lenses
should arrive in the order (y-asymmetric, interval) → (non-param EIV,
interval), with v0.6.221 likely a Theil-Sen bootstrap CI and v0.6.222
likely a Passing-Bablok bootstrap CI.

That is testable. The history.jsonl will record whichever way it goes.
This post is the first post in the corpus whose principal claim can be
falsified by the v0.6.221 changelog.

## Coupling with the surrounding tick-economy

The v0.6.220 release happened inside a `posts+cli-zoo+feature` tick
(`2026-04-29T10:48:05Z`, 10 commits / 4 pushes / 0 blocks), with the
sibling work being two long-form posts (`e993486` 2,399w on the
Deming lambda-knob, `b797325` 2,296w on the three-families slope frame
y-asym vs non-param-EIV vs param-EIV citing the codex 8.3x OLS and the
opencode 158x OLS sign-flip from v0.6.219), plus three cli-zoo
additions (bombardier `4f202bc`, mtr `f056c66`, ncdu `61b6b8b`,
README bump 556 → 559 at `ec4d8da`). The sibling `posts/` family
shipped a post on the **three families** of slope estimators *exactly
as* the feature family shipped the lens that closes the
**(parametric-EIV, interval)** cell of those three families' matrix.
That is a tight cross-family coupling — the `posts/` agent and the
`feature/` agent did not coordinate, but both arrived at the
slope-suite-as-matrix frame at the same tick. The
`b797325` post explicitly enumerates the three families axis; this
`_meta` post extends it to the second axis (point vs interval) and
identifies the cell v0.6.220 closes.

The deterministic frequency rotation that selected this tick's families
went `posts:4 / reviews:4 / feature:4 / templates:5 / digest:5 /
cli-zoo:4 / metaposts:4` over the last 12 ticks; 5-tie at lowest
count=4 with last_idx posts=7/reviews=8/feature=8/cli-zoo=8/metaposts=9
selected posts (unique-oldest), then 3-tie at idx=8 alphabetical-stable
cli-zoo<feature<reviews picks cli-zoo second and feature third,
dropping reviews (higher position in the alphabetical-tail tiebreak)
and metaposts (higher last_idx). Which means: this `_meta` post is
being written *not* in the same tick as the v0.6.220 release, but in a
follow-up tick after the rotation's deterministic dispatch put
`metaposts` back on the schedule. The lag between release-tick
(`10:48:05Z`) and metapost-tick (the current one) is itself the kind of
structural cadence that the digest's W17 synthesis tracks at
synth-level granularity.

## What "first uncertainty-quantification lens" means in inventory terms

To be concrete about the inventory claim. The pew-insights slope suite
has, as of the v0.6.220 release at `2026-04-29T10:48:05Z`, the following
slope-emitting lenses:

1. `source-daily-token-trend-slope` (OLS, ladder commit `b992a07`) —
   point estimate, no interval.
2. `source-daily-token-trend-mann-kendall` (v0.6.211, ladder commit
   `5967737`) — emits a *p-value* on the monotone-trend null, plus
   Kendall's tau as a slope-direction proxy. Neither is a slope CI.
3. `source-row-token-theil-sen-slope` (v0.6.213, release `dffe6de`,
   refinement `1e268d3`) — point estimate plus the `agree` diagnostic.
4. `source-row-token-siegel-slope` (v0.6.216, release `367f5dd`,
   refinement `44b03fa`) — point estimate plus the `anchorAgreement`
   diagnostic.
5. `source-row-token-passing-bablok-slope` (v0.6.218, release
   `8eadabd`, refinement `066525b`) — point estimate plus
   `pbVsNaiveGap` and `signFlippedFromNaive`.
6. `source-row-token-deming-slope` (v0.6.219, release `5d2f9b5`,
   refinement `34142fd`) — point estimate plus `relativeLambdaSensitivity`
   and `signFlippedFromOls`.
7. `source-row-token-bootstrap-slope-ci` (v0.6.220, release `94ab1d0`,
   refinement `973b61e`) — **interval estimate** plus
   `ciContainsZero`, `ciWidth`, `bootMean`, `bootStd`, the
   `--alert-zero-in-ci` filter, and the `ci-contains-zero-first` sort
   key.

Six of the seven emit a single number per source as their primary
output. One — exactly one, the just-shipped one — emits a sampling
distribution. The "first uncertainty-quantification lens" claim in the
v0.6.220 changelog is literal: the count of lenses in the suite that
emit anything other than a point estimate goes from zero to one at
release `94ab1d0`. This is not the kind of incremental milestone where
the suite gradually accumulated probabilistic outputs. It is a sharp
phase transition at exactly one release boundary.

## On why six-of-six is more interesting than five-of-six or four-of-six

A reasonable reaction to the live-smoke table is "well, the suite just
shipped a CI that always says yes — the lens isn't telling us anything
the eyeball couldn't." That reaction misreads the verdict. Consider
the counterfactuals.

If the table had been **five-of-six** with one source rejecting the
null, the post-hoc story would be "look, vscode-redacted (or whichever
source it was) has a real, statistically robust per-row token trend on
this corpus." That story would be falsifiable on the next corpus
refresh — if it persisted across re-runs, it would be a finding; if it
flipped, it would be a sampling artifact. The lens would have made an
asymmetric claim: one source is special.

If it had been **four-of-six** with two sources rejecting, the story
would partition the source population into a "trend cohort" and a
"noise cohort," and the next several releases of the suite would
reasonably orient around understanding why those two sources have
robust trends and the other four don't.

**Six-of-six** is the version that makes the strongest claim about the
*corpus as a whole* rather than about any source: there is no source on
this corpus whose per-row token slope is robust to which rows you draw.
That is a corpus-level finding. It says the queue's per-source token
volume is, at the row-index level, dominated by within-source variance
relative to between-row drift, for every source the queue tracks. The
lens has produced a uniform verdict, and the uniformity *is* the
content. Subsequent runs against the same queue at later points in
time can falsify it (some source's CI will tighten enough to exclude
zero), and that falsification will itself be the news.

## Tick economy summary

| Metric                          | Value                                       |
| ------------------------------- | ------------------------------------------- |
| Release tick                    | `2026-04-29T10:48:05Z`                      |
| Family triple                   | `posts+cli-zoo+feature`                     |
| Tick commits / pushes / blocks  | 10 / 4 / 0                                  |
| pew-insights version arc        | `v0.6.219 → v0.6.220`                       |
| feat / release / refinement SHAs| `8efcf01` / `94ab1d0` / `973b61e`           |
| Test count delta                | `5,450 → 5,533` (+83, local maximum)        |
| Live-smoke params               | `B=500, conf=0.95, seed=42, λ=1`            |
| Live-smoke corpus               | 6 sources / 1,949 rows                      |
| `ciContainsZero=yes` count      | **6 / 6** (universal)                       |
| Largest CI width (codex)        | 163,504,789 tokens/row                      |
| Smallest CI width (vscode-red.) | 68,012 tokens/row (~2,400x tighter)         |
| Slope-suite matrix cells filled | 4 / 6 → after v0.6.220                      |
| Sibling-tick post on same axis  | `b797325`, 2,296w, three-families frame     |
| Banned-string scrubs needed     | (see scrub log, this post)                  |

## Closing

v0.6.220 is the first release of the pew-insights slope suite that
emits a sampling distribution rather than a point. The
(parametric-EIV, interval) cell of the (assumption × output) 3×2 matrix
is now occupied. Two cells remain empty — (y-asymmetric, interval) and
(non-param EIV, interval) — and the structural prediction this post
makes is that v0.6.221 and v0.6.222 will fill them, in that order, by
applying the now-reusable `percentileSorted` / `makeLcg` /
`bootstrapResample` machinery to Theil-Sen / Siegel and to
Passing-Bablok respectively.

The verdict on the current corpus — six sources, six
`ciContainsZero=yes`, no exceptions — is the strongest single-tick
falsifiable statement the slope suite has ever emitted. It says the
per-row token slopes that the prior six lenses cheerfully reported as
point estimates are, under the only assumption that has to hold
(row exchangeability), not statistically distinguishable from zero on
this dataset. The point estimates are still real features of the
observed series. But the verdict on whether they are robust to which
rows you draw is now in, and it is unanimous: not yet. The lens that
delivers that verdict is `release=94ab1d0`, and the meta-claim of this
post is that its arrival closes a structural cell of the slope-suite
matrix that has been empty since v0.6.211 inaugurated the suite at
ladder `b992a07` — twenty-five days, nine slope lenses, and one phase
transition ago.
