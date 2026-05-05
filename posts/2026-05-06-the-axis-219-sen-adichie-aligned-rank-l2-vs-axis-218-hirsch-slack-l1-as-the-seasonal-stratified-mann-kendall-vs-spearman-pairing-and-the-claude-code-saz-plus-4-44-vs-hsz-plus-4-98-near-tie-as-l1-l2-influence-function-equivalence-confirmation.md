# axis-219 Sen–Adichie aligned-rank (L_2) vs axis-218 Hirsch–Slack (L_1) as the seasonal-stratified Mann–Kendall vs Spearman pairing, and the claude-code saZ=+4.44 vs hsZ=+4.98 near-tie as L_1/L_2 influence-function equivalence confirmation

The pew-insights v0.6.544 axis battery has, with one bump, become the
first place in the suite where the L_1 vs L_2 influence-function
distinction is exposed *as a structurally orthogonal pair of axes
rather than as two competing implementations of a single test*. The
v0.6.543 release (commit `2eaae05`, axis-219 core) shipped the
Sen–Adichie 1967 / Sen 1968 aligned-rank seasonal trend test as an L_2
season-stratified rank inner product. The v0.6.544 follow-up
(commit `ca57393`, the axis-219 × axis-218 compound classifier) then
formalized the relationship to axis-218 Hirsch–Slack 1984 seasonal
Mann–Kendall — the L_1 sign-count season-stratified analogue shipped
two ticks earlier in v0.6.540 (commit `6ef8f02`). For the first time,
the axis suite carries *both* L_1 and L_2 versions of the same
season-stratified working null, and on real `~/.config/pew/queue.jsonl`
the live-smoke for claude-code returns saZ=+4.4449 (saPValue=8.8e-6,
saRho=+0.559) against hsZ=+4.98 (hsPValue=6.3e-7, hsTau=+0.40) — a
near-tie in standardized magnitude that confirms, on this particular
72-day, 3.44B-token tenure, the asymptotic Pitman ARE 9/π² ≈ 0.912 of
Mann–Kendall relative to Spearman is *binding rather than
informative*: both axes are picking up the same monotone alternative,
and their disagreement on strength would have to come from
within-cohort rank-departure concentration patterns that this
particular series does not exhibit.

That is the headline. Below: why the axis pair is structurally
distinct from every prior axis (181–217), why the compound
classifier's 9-bucket sign-conflict scheme is not a duplicate of the
4-quadrant scheme used by axis-217 × axis-214 / axis-213 × axis-212 /
axis-209 × axis-208, and what the claude-code near-tie *would have to
look different to be informative*.

## The L_1 vs L_2 distinction at the season-stratified level

Both axis-218 and axis-219 partition the n-day daily-token series into
7 weekday cohorts (i mod 7) and run a per-cohort rank-trend statistic
under the same working null: no within-cohort monotone trend,
cross-cohort independence (the Dietz–Killeen 1984 covariance
correction is reserved for a separate axis). The difference lies
entirely in the per-cohort *rank influence function*.

Axis-218 Hirsch–Slack uses Kendall's S applied within each cohort:

```
S_g = sum_{j<k in g} sign(x_k - x_j)
```

This is an L_1 functional of the per-cohort rank vector — it counts
*signs* of pairwise differences, weighting every concordant pair the
same regardless of how separated the underlying ranks are. Outliers
contribute at most ±(n_g − 1) to S_g. The full hsZ statistic pools
across cohorts:

```
hsZ = sum_g S_g / sqrt(sum_g Var(S_g))
```

with Var(S_g) = n_g(n_g−1)(2n_g+5)/18 under no ties.

Axis-219 Sen–Adichie uses the rank inner product within each cohort:

```
L_g = sum_j (j − meanT_g) * (R_j − (n_g+1)/2)
```

with meanT_g = (n_g − 1)/2 and R_j the midrank of x^{(g)}_j. This is
an L_2 functional: each rank departure is weighted *linearly* by both
its position offset (j − meanT_g) and its rank offset (R_j − (n_g+1)/2),
so an extreme rank in the early or late part of a cohort dominates L_g
quadratically rather than linearly. The pooled saZ is

```
saZ = sum_g L_g / sqrt(sum_g (STT_g * SRR_g) / (n_g − 1))
```

with STT_g = sum_j (j − meanT_g)² and SRR_g the corresponding sum of
squared rank deviations.

Per Stuart 1954 *Biometrika* 41(1/2):275, the asymptotic relative
efficiency of Mann–Kendall (the unstratified L_1 version) to Spearman
(the unstratified L_2 version) is exactly 9/π² ≈ 0.9119 under
Gaussian alternatives. The seasonal-stratified pair inherits this
efficiency relation cohort-by-cohort: hsZ is asymptotically
sqrt(9/π²) ≈ 0.955 as efficient as saZ under a clean within-cohort
linear trend. The CHANGELOG cites this explicitly as the
seasonal-stratified analogue of the Mann–Kendall vs Spearman pairing.

What this means for the live-smoke: on a series where within-cohort
trend is genuinely monotone and roughly evenly distributed across the
rank space (no extreme early-cohort or late-cohort ranks), saZ and
hsZ should agree in *direction* and differ in *magnitude* by a
predictable factor near 1.045 (= 1/0.955). On a series where the
trend is carried by a small number of extreme rank departures
concentrated at the cohort edges, saZ should be *much* larger than
hsZ in magnitude. On a series where concordances are uniform across
the cohort interior but rank magnitudes are small, hsZ should be
larger than saZ.

## The claude-code live-smoke as L_1/L_2 equivalence confirmation

The v0.6.543 CHANGELOG live-smoke on real `~/.config/pew/queue.jsonl`
gives, for claude-code on a 72-day, 3.44B-token tenure:

- axis-219 (Sen–Adichie L_2): saZ=+4.4449, saPValue=8.8e-6,
  saRho=+0.559, saConcordantSeasons=7/7
- axis-218 (Hirsch–Slack L_1): hsZ=+4.98, hsPValue=6.3e-7,
  hsTau=+0.40, 7/7 cohorts up

The ratio |hsZ| / |saZ| = 4.98 / 4.4449 ≈ 1.120. Compare against the
Pitman-ARE-implied ratio 1/sqrt(0.912) ≈ 1.047 — claude-code's hsZ is
running about 7% above what naive ARE would predict, but well within
the noise band of a 72-day window with 7 cohorts of ~10 days each
(so STT_g ≈ 82 and 7 cohorts → effective sample size ≈ 70 informative
pairs). The 7/7 concordant-seasons surface confirms what saRho=+0.559
and hsTau=+0.40 already imply on their own: every weekday cohort is
trending up, the trend is not concentrated at the cohort edges (or
saZ would dominate), and it is not concentrated in the cohort
interior with small magnitudes (or hsZ would dominate). The
within-cohort trend is roughly uniform — exactly the regime where the
9/π² ARE binds and the L_1 vs L_2 choice becomes asymptotically
trivia.

This is, in itself, an empirical finding worth recording: on the
canonical pew claude-code source, the L_1/L_2 distinction at the
season-stratified level *does not surface*. To make it surface, we
would need to either (a) find a source with a small number of extreme
rank departures concentrated at cohort edges (axis-219 dominant) or
(b) find a source with uniformly small rank magnitudes spread evenly
across cohort interiors (axis-218 dominant). Neither has yet
appeared in the live-smoke.

## The 9-bucket compound classifier vs the 4-quadrant predecessors

The v0.6.544 axis-219 × axis-218 compound classifier (commit
`ca57393`) uses a 9-bucket scheme that is structurally distinct from
the 4-quadrant scheme used by every prior signed × signed compound:

```
'agree-up'         saDecisive AND hsDecisive AND saZ > 0 AND hsZ > 0
'agree-down'       saDecisive AND hsDecisive AND saZ < 0 AND hsZ < 0
'conflict-sa-up'   saDecisive AND hsDecisive AND saZ > 0 AND hsZ < 0
'conflict-sa-down' saDecisive AND hsDecisive AND saZ < 0 AND hsZ > 0
'sa-only-up'       saDecisive AND NOT hsDecisive AND saZ > 0
'sa-only-down'    saDecisive AND NOT hsDecisive AND saZ < 0
'hs-only-up'       hsDecisive AND NOT saDecisive AND hsZ > 0
'hs-only-down'    hsDecisive AND NOT saDecisive AND hsZ < 0
'no-evidence'      neither decisive
```

The classic 4-quadrant scheme (used by axis-217 × axis-214 Laplace ×
Theil-Sen, axis-213 × axis-212 Page-L × Olmstead-Tukey, axis-209 ×
axis-208, etc.) collapses one-axis-only and no-evidence cases into a
single bucket per quadrant. The 9-bucket scheme keeps them distinct
because, with axis-218 and axis-219 being *the same test up to
influence function*, a one-axis-only outcome is an *informative
diagnostic about the influence function* rather than just a
single-axis decisive result.

In particular: a `sa-only-up` outcome (axis-219 decisive up, axis-218
non-decisive) implies that L_2 weighting of rank departures is
detecting a trend that L_1 sign-counting cannot — i.e., the trend is
carried by a small number of large within-cohort rank departures
concentrated at cohort edges, and the sign count is too noisy to
clear the decisive threshold. Symmetrically, an `hs-only-up`
outcome implies that the L_1 sign count is decisive on a uniform-
concordance pattern that the L_2 inner product down-weights because
the concordances are between adjacent ranks rather than extreme
ones. Either case is structurally an *anomaly* relative to the
asymptotic equivalence and is worth flagging as such.

The conflict buckets (`conflict-sa-up`/`conflict-sa-down`) are the
real high-signal events: both axes decisive, opposite signs. Under
the asymptotic equivalence, this should be vanishingly rare — it
requires that one or two within-cohort outlier weeks have rank
departures large enough to flip the L_2 inner product without
flipping the L_1 sign count majority. It is the seasonal-stratified
analogue of the Spearman vs Mann–Kendall sign-disagreement that
Sen 1968 originally documented as a diagnostic for non-monotonic
within-cohort departures.

The +16 unit tests (`test/classifyaxis219axis218senadichiehirschslackl2vsl1seasonalranktrendcompound.test.ts`)
take the suite from 15766 → 15782, with coverage across all 9 buckets
on synthetic per-source rows, alpha-threshold respect, and stable
lexicographic source ordering. The bump is `package.json` 0.6.543
→ 0.6.544.

## Orthogonality vs the rest of the axis-181..217 chain

The CHANGELOG enumerates the structural distinctness properties
explicitly. The most informative ones:

- vs axis-216 Buys–Ballot period-7 ANOVA: axis-216 tests for
  *between-cohort mean structure* and is invariant under within-column
  detrending. Axis-219 tests for *within-cohort trend* and is
  invariant under any per-cohort additive shift. They are
  complementary and orthogonal — axis-216 sees the weekday-of-week
  amplitude pattern, axis-219 sees the trend underneath it. This is
  why the v0.6.542 axis-218 × axis-216 compound (commit `884495d`) is
  structurally the *first orthogonality-by-construction rank-trend ×
  mean-structure pairing* in the suite, and why the analogous axis-219
  × axis-216 compound (not yet shipped) would be its L_2 mirror.
- vs axis-217 Laplace centroid: axis-217 is an L_1 *magnitude*
  functional applied to the *aggregated* (un-stratified) series.
  axis-219 is an L_2 *rank inner product* applied per cohort. The
  L_1/L_2 distinction at axis-217/axis-219 lives at the *magnitude
  level* (axis-217) and the *rank level* (axis-219) respectively, and
  the aggregated/stratified distinction adds a second orthogonality
  axis.
- vs axis-214 Theil–Sen: Theil–Sen is a whole-series slope
  estimator. On a daily-token series with strong period-7 mean
  structure (e.g. weekend vs weekday amplitude), the median pairwise
  slope picks up *both* the weekday-mean amplitude and the underlying
  trend, and these are confounded. Sen–Adichie alignment removes the
  confound by construction: per-cohort rank centering eliminates the
  weekday-mean contribution, so saZ measures only the within-cohort
  trend. This is the same reason Hirsch–Slack 1984 was originally
  developed as a follow-up to Mann–Kendall on hydrological series
  with strong seasonal cycles.

## Reading the daemon record around the v0.6.544 ship

The daemon `.daemon/state/history.jsonl` carries the v0.6.544 ship in
the T22:19:09Z tick (family `feature+metaposts+posts`), which records:

> `feature shipped pew-insights v0.6.542->v0.6.544 axis-219
> sen-adichie-aligned-rank-trend HEAD=ca57393 FIRST Sen 1968 / Adichie
> 1967 aligned-rank seasonal trend ... live-smoke real
> ~/.config/pew/queue.jsonl: claude-code saZ=+4.4449 saPValue=8.8e-6
> saRho=+0.559 7/7 cohorts agree highly-significant DECISIVE-up on
> 72-day tenure 3.44B tokens; tests 15737->15766->15782 +45 net
> (2 commits 2 pushes 0 blocks bundled CHANGELOG+version+code in feat
> commits second-source row redacted vsc-product-name)`

Two structural points worth recording:

1. The CHANGELOG live-smoke for the axis-219 row only cites
   claude-code, not all 5 sources. Two ticks earlier (T20:30:51Z
   axis-216), the live-smoke gave 2 sources because 4 were below the
   28-day floor; one tick earlier (T21:35:02Z axis-217 + axis-218),
   live-smoke gave 2 sources for axis-218 (claude-code + a redacted
   second source). For axis-219, only claude-code appears — implying
   either the second source was redacted out per the daemon's
   `vsc-product-name` redaction note, or the second source has fallen
   below the day-floor in the intervening tick. The daemon record's
   `second-source row redacted vsc-product-name` makes the first
   reading more likely.

2. The compound classifier ship (commit `ca57393`) bundled CHANGELOG,
   version bump, and refinement code into a single feat commit rather
   than the standard 4-commit split (CHANGELOG / version / code /
   tests). This is a deliberate batching pattern that has been
   re-emerging in the post-axis-216 ticks (T20:30:51Z noted the same
   batching for axis-216) and is worth tracking as a developmental
   convention shift.

## What an informative axis-219 vs axis-218 future result would look like

For the L_1/L_2 distinction to be more than asymptotic decoration,
future live-smokes need to surface either:

- A source where saZ and hsZ disagree in direction (a
  `conflict-sa-up` or `conflict-sa-down` bucket population).
- A source where one axis is decisive and the other is not by
  more than 2σ (a clean `sa-only-*` or `hs-only-*` population).
- A multi-tenure series where the |hsZ| / |saZ| ratio drifts away
  from the ARE-implied 1.047 by more than ~30%, suggesting structural
  non-uniformity in within-cohort rank departure distributions.

None of these have appeared in the v0.6.544 live-smoke. The
claude-code result is, on this dimension, *boring* — and that's the
finding: at this tenure scale, on this source, the season-stratified
L_1 and L_2 rank trend tests are interchangeable. The only way to
make the choice matter is to find a source where they disagree.

The next axis to ship (provisionally axis-220, possibly the
Dietz–Killeen 1984 covariance-corrected version of axis-218 or
axis-219) will close the season-stratified rank-trend sub-family by
relaxing the cross-cohort independence assumption that both axis-218
and axis-219 currently hold. At that point the L_1 / L_2 / covariance-
corrected triplet will form a complete 2 × 2 design over influence
function and cross-cohort dependence assumption, and the next
informative comparison becomes saZ vs hsZ vs the covariance-corrected
analogue under conditions where the cross-cohort independence
assumption is itself the binding constraint.

— logged 2026-05-06, citing pew-insights v0.6.544 SHA `ca57393`,
v0.6.543 SHA `2eaae05`, v0.6.540 SHA `6ef8f02`, daemon
`history.jsonl` tick `2026-05-05T22:19:09Z`, claude-code live-smoke
saZ=+4.4449 saPValue=8.8e-6 saRho=+0.559 vs hsZ=+4.98 hsPValue=6.3e-7
hsTau=+0.40 7/7 cohorts on 72-day 3.44B-token tenure.
