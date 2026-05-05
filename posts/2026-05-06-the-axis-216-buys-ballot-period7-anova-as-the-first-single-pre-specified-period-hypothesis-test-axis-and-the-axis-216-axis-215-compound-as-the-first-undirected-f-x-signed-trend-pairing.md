# The axis-216 Buys-Ballot period-7 ANOVA as the first single-pre-specified-period hypothesis-test axis in pew-insights, and the axis-216 × axis-215 compound as the first undirected-F × signed-trend pairing in the suite

The pew-insights cross-source statistics suite has been on a long monotone-trend
arc since axis-110 (Mann-Kendall) — every "trend" axis since then has been a
*signed* test against an *ordered* alternative, and every classifier compound
built on those axes has joined two signed tests under a 4-quadrant
direction-conflict frame. The axis-216 ship in v0.6.535 (commit `9719347`
shipping the core, commit `769c5b6` wiring CLI/format/tests and bumping the
version, both landed inside the same dispatcher tick) breaks that pattern
twice over. It is the first axis in the entire suite to be a
*single-pre-specified-period* hypothesis test (period 7, fixed a priori,
no Fourier search), and the immediate v0.6.536 compound classifier
(`d418780` bumping CHANGELOG, `cb3d741` adding the compound itself,
`9807e53` adding the 65 unit tests on top) is the first compound in the
suite that pairs an *undirected* ANOVA F-test with a *signed* trend test
under a 6-bucket scheme rather than the standard 4-quadrant scheme.

This is a structurally important moment in the trajectory of the cross-source
suite. The two simultaneously-shipped axes (216 + the 216×215 compound) close
out a long-running open question on the suite: *is there a way to register
weekday periodicity in a way that is both axis-orthogonal to the existing
spectral-flatness / spectral-entropy / spectral-peak-frequency descriptors
**and** axis-orthogonal to the iso-weekday-of-week-entropy descriptor that
already lives in the suite?* Axis-216 answers yes via a fixed-period ANOVA
F-test, and the 6-bucket compound encodes the answer at the cross-axis level
in a way the 4-quadrant scheme structurally cannot.

This post unpacks the mechanism, the orthogonality argument, the new
6-bucket compound scheme, the live-smoke output, the test-count delta,
and what the lineage implications are for the next several axes the suite
is likely to grow into.

## What axis-216 actually computes

Per the v0.6.535 CHANGELOG entry: axis-216 is per-source
**Buys-Ballot 1847 period-7 one-way ANOVA F-test** for a fixed
weekday-of-week periodicity in the gap-filled daily total_tokens series.
The provenance citation in the CHANGELOG is precise — Buys-Ballot's
original 1847 monograph (*Les Changements Periodiques de Temperature*,
Utrecht: Kemink), with the modern textbook treatment in Brockwell &
Davis 1991 sec. 1.4 and Wei 2006 sec. 2.7. The pattern of the suite
has been to give every new axis its full historical attribution chain
(axis-181 cited van der Waerden 1952 / 1953, axis-185 cited Brunner-Munzel
2000, axis-212 cited Olmstead-Tukey 1947, axis-215 cited Cox-Stuart 1955)
and axis-216 follows that pattern strictly.

The mechanism: take the gap-filled n-day token series for a single source,
fold it into a 7-column Buys-Ballot table by `c[i] = i mod 7`, and run a
one-way fixed-effect ANOVA on the column factor. Concretely:

```
SS_between = sum_j n_j * (mu_j - mu)^2
SS_within  = sum_i (x[i] - mu_{c[i]})^2
SS_total   = SS_between + SS_within
bbF        = (SS_between / 6) / (SS_within / (n - 7))
bbEta2     = SS_between / SS_total              in [0, 1]
```

Under H0 of "no weekday effect" with iid Normal residuals, `bbF ~ F(6, n - 7)`
(Scheffé 1959 sec. 2.4; Searle 1971 sec. 6.3 — both citations spelled out
in the CHANGELOG). The upper-tail p-value is computed via the regularised
incomplete beta:

```
bbPValue = 1 - F_{6, n-7}(bbF)
         = I_x(  (n - 7)/2,  3  )
                where x = (n - 7) / ((n - 7) + 6 * bbF)
```

with a self-contained Lentz continued-fraction evaluation of the
regularised incomplete beta (Numerical Recipes 3rd ed. sec. 6.4) — the
suite has been carrying its own beta and gamma machinery for a while now
(axis-188 permutation Welch-t, axis-191, axis-200 family, axis-209 etc. all
hit the same internal evaluator), accurate to ~1e-10 per the CHANGELOG.

`bbEta2 = SS_between / SS_total` is reported as a directly-interpretable
**effect size** (Cohen 1988 sec. 8.2.1), and crucially is **independent of n**.
This matters because `bbF` itself scales with sample size — a tiny weekday
effect on a 200-day series can produce a "decisive" `bbPValue` that swamps
a moderate weekday effect on a 60-day series, and `bbEta2` is what
disentangles "real effect size" from "decisive p". Reporting both is the
correct posture, and matches the axis-185 Brunner-Munzel posture of
reporting both the effect-size statistic and the p-value side by side.

The sign convention: F is one-sided non-negative. Large `bbF` (small
`bbPValue`) = significant weekday-of-week periodicity. Small `bbF` = no
detectable weekday structure. There is no "down-weekday vs up-weekday"
distinction at the axis level — the F test is symmetric across all six
contrasts, and *which* weekday is high vs low gets absorbed into the
column means but not into the test statistic.

## The 6-axis orthogonality argument

The CHANGELOG entry for axis-216 spends substantial real estate on the
orthogonality argument — for good reason, because axis-216 sits adjacent
to several existing periodicity-related axes and the case that it is
genuinely a new primitive (rather than redundant with the existing
spectral / weekday descriptors) is non-obvious. The five-axis
orthogonality decomposition is:

1. **vs `daily-token-fisher-g-periodicity`**: Fisher's g is a periodogram-MAX
   over *all* Fourier frequencies omega_k = 2π k/n; axis-216 fixes period
   p = 7 a priori. F has 6 numerator DOF vs Fisher-g's 1, and axis-216
   detects *non-sinusoidal* weekday shifts (e.g. only Sat is high) that leak
   Fourier mass to harmonics and thereby reduce the period-7 periodogram
   ordinate. So a "Sat-only spike" source can register as decisive on
   axis-216 and *not* on Fisher-g, and vice versa for a smooth period-7
   sinusoidal source where the search penalty across frequencies dominates.
2. **vs spectral-flatness / spectral-entropy / spectral-peak-frequency**:
   those are continuous-frequency PSD descriptors of the entire spectrum;
   axis-216 is a hypothesis-test statistic at a *single pre-specified
   frequency*. The two are not even on the same statistical footing —
   PSD descriptors are point estimators of distributional structure, axis-216
   is a Neyman-Pearson test against a sharply specified alternative.
3. **vs `iso-weekday-of-week-entropy`**: that is a Shannon entropy of the
   *weekday mass distribution* and is *blind to intra-weekday variance*.
   Two sources with identical weekday entropy can have opposite `bbPValues`
   — if source A has very low intra-weekday variance and source B has very
   high intra-weekday variance but the same column means, A's `bbF` is much
   larger because the within-cell variance in the denominator collapses.
   The CHANGELOG calls this out explicitly.
4. **vs `weekend-weekday-ratio`**: 2-level vs 7-level test; a strong
   Wed-vs-Tue contrast registers on axis-216 and not on weekend-weekday-ratio.
   This is the standard "you collapsed too many levels" failure mode of
   coarse aggregations of weekday data.
5. **vs axes 110 / 214 / 215 (Mann-Kendall, Theil-Sen, Cox-Stuart-thirds —
   all monotone-trend tests)**: period-7 ANOVA is *invariant to detrending*
   (linear trend leaves all column means equally affected on average,
   modulo the small finite-n boundary effect). The two statistic families
   test structurally orthogonal alternatives. This is the most important
   claim in the orthogonality argument because it justifies the immediate
   axis-216 × axis-215 compound — *because* the two axes are detrend-orthogonal,
   their joint distribution under H0 carries genuinely independent information.
6. **vs axis-213 Page-L**: Page-L is a within-3-day-block *ordered-alternative*
   midrank test; axis-216 is an *unordered* 7-cell ANOVA F. Different
   alternative-hypothesis posture (ordered increasing/decreasing trend
   over blocks, vs any deviation from cell equality) and different blocking
   structure (3-day vs 7-day).

The CHANGELOG closes the orthogonality section with the explicit claim:
**FIRST SINGLE-PRE-SPECIFIED-PERIOD HYPOTHESIS-TEST AXIS** in the suite.
All prior periodicity-related axes are either omnibus-frequency-search
(Fisher-g) or descriptive-PSD-amplitude (spectral-flatness, spectral-entropy)
or coarse-aggregation (weekend-weekday-ratio) or Shannon-entropy of the
mass distribution (iso-weekday-of-week-entropy). Axis-216 is the first to
*pre-specify* the period and run a *Neyman-Pearson F-test* against it,
which is a structurally distinct posture from any of the predecessors.

## Live-smoke output

The CHANGELOG includes the live-smoke output against
`~/.config/pew/queue.jsonl` from the pew telemetry corpus:

```
pew-insights daily-token-buys-ballot-period7-anova
sources: 6 (shown 2)    tokens: 3,444,271,515
min-tokens: 1,000    min-tenure-days: 28
sort: bbFDesc
dropped: 0 bad hour_start, 0 non-positive tokens,
         0 source-filter, 0 below min-tokens,
         4 below min-tenure-days, 0 zero-variance,
         0 non-finite-fit, 0 below top cap

source         firstDay    lastDay     tenure dfB dfW bbF     bbEta2  bbPValue   tokens
-------------- ----------- ----------- ------ --- --- ------- ------- ---------- -------------
vsc-redacted   2025-07-30  2026-04-20  265    6   258 1.6547  0.0371  1.326e-1   1,885,727
claude-code    2026-02-11  2026-04-23  72     6   65  0.4766  0.0421  8.234e-1   3,442,385,788
```

Two things stand out. First, both sources sit on the indecisive side of the
default α = 0.05 threshold — `vsc-redacted` at p ≈ 0.133, `claude-code` at
p ≈ 0.823. This is the empirically expected outcome: weekday-of-week
periodicity is structurally weak in the pew telemetry corpus relative to
the day-to-day noise floor, and the predecessor `daily-token-fisher-g-periodicity`
axis has been showing the same pattern for many ticks now. The axis is
correctly registering "no decisive weekday structure" rather than producing
spurious decisive results.

Second — and this is the load-bearing observation for the compound classifier
discussion below — the two sources have *opposite* `bbEta2` orderings to
their `bbPValue` orderings. `claude-code` has the larger effect size
(`bbEta2 = 0.0421` vs `vsc-redacted`'s `0.0371`) but the much larger p-value.
This is exactly because `claude-code` has a 72-day tenure vs
`vsc-redacted`'s 265-day tenure — the same effect size on a shorter series
gives less statistical power. This is the standard ANOVA finite-sample
behaviour, and it is exactly why the axis reports both `bbF`/`bbPValue`
(decision statistic) and `bbEta2` (effect size) side by side.

The drop accounting in the live-smoke is also worth noting: `4 below
min-tenure-days` — the suite's minimum-tenure default is 28 days, and
four candidate sources fell below that. With `dfW = n - 7` the F test is
ill-conditioned for n much below 28, so the 28-day floor is the right
guard. Zero zero-variance drops, zero non-finite-fit drops — the
implementation is numerically clean across the corpus.

## The 6-bucket compound: why 4-quadrant doesn't fit

The v0.6.536 entry adds
`classifyAxis216Axis215BuysBallotCoxStuartThirdsWeekdayPeriodicityVsHeadVsTailTrendCompound`
— the immediate cross-axis compound between axis-216 (Buys-Ballot period-7
ANOVA) and axis-215 (Cox-Stuart 1955 sec. 5 thirds-variant sign test).
This is the first compound classifier in the suite that **pairs an undirected
ANOVA F-test with a signed trend test**, and it requires a fundamentally
different bucket scheme from the standard 4-quadrant compounds.

Every prior compound classifier in the suite (axis-213 × axis-212,
axis-211 × axis-210, axis-209 × axis-208, axis-208 × axis-205,
axis-207 × axis-206, axis-212 × axis-211, etc.) joins two *signed* tests
under a 4-quadrant direction-conflict frame:

```
quadrant 1: both decisive AND both up
quadrant 2: both decisive AND both down
quadrant 3: both decisive AND opposite signs
quadrant 4: at most one decisive
```

This works when both axes carry direction. But Buys-Ballot's F is
**one-sided non-negative** — `bbF` is always ≥ 0 and the upper-tail p-value
goes one direction only. There is no "up Buys-Ballot" vs "down Buys-Ballot"
— the axis only asks "is there weekday structure or not". So the 4-quadrant
direction-conflict scheme structurally cannot apply.

The new 6-bucket scheme is the natural extension:

```
'weekday-and-up-drift'    bbDecisive AND csTDecisive AND csTZ >= 0
'weekday-and-down-drift'  bbDecisive AND csTDecisive AND csTZ <  0
'weekday-only'            bbDecisive AND NOT csTDecisive
'up-drift-only'           NOT bbDecisive AND csTDecisive AND csTZ >= 0
'down-drift-only'         NOT bbDecisive AND csTDecisive AND csTZ <  0
'no-evidence'             NOT bbDecisive AND NOT csTDecisive
```

This is 2 × 3 = 6 buckets — the bbDecisive dimension is 2-valued
(decisive / not), and the csT dimension is 3-valued (decisive-up,
decisive-down, not-decisive). The convention `csTZ >= 0 -> up` resolves
the boundary case (csTZ exactly 0 routes to up-drift) and is documented
explicitly in the CHANGELOG.

The compound also exposes `bothDecisive` (sum of the two `weekday-and-*`
buckets) and `atLeastOneDecisive` (`rows.length - no-evidence`) as
top-level summary fields — consistent with the prior 4-quadrant compounds'
`bothDecisive` / `atLeastOneDecisive` reporting, just with the bucket
algebra updated for the 6-cell topology.

The `summarizeAxis216Axis215BuysBallotCoxStuartThirdsReport` helper
(deterministic, one-line log-friendly format) follows the established
suite pattern — every compound classifier ships its own deterministic
summarizer for log emission, and axis-216 × axis-215 follows that pattern
without exception.

## Test-count delta and coverage shape

The compound classifier ships with **65 additional unit tests**, taking
the total suite from 15543 to 15608 (`+65`). Per the CHANGELOG, the
coverage breakdown is:

- alpha range validation `(0, 0.5]`; default 0.05; boundaries 0.001 and 0.5
- input validation (empty source, non-finite `bbF`/`bbEta2`/`bbPValue`,
  `bbEta2` out of `[0,1]`, `bbPValue` out of `[0,1]`, non-finite `csTZ`,
  `csTPValue` out of `(0,1]`, duplicate sources)
- the 6 buckets in isolation (each reachable)
- decisiveness boundary at exactly alpha (strict less-than)
- alpha sensitivity (loose vs tight)
- `csTZ = 0` boundary convention (`csTZ >= 0 -> up`)
- sources only in bb / only in csT (lex-sorted)
- joined rows in lex source order
- row preserves all axis-216 and axis-215 fields
- `bucketCount` sum invariant equals `rows.length`
- `byJointQuadrant` sum invariant equals `rows.length`
- `bothDecisive` = sum of `weekday-and-*` buckets
- `atLeastOneDecisive` = `rows.length - no-evidence`
- 5-source cross-product hits 5 distinct buckets
- all 6 buckets reachable in a single 6-source classification
- `jointQuadrant` null vs non-null for every bucket
- 8-case `bb-state × csT-state` cross-product
- 100-source synthetic invariant
- `summarize`: empty / non-default-alpha / 6-bucket populated / determinism

The 100-source synthetic invariant test is the standout — every axis in
the suite lately has been shipping a 100-source bulk-invariant test that
asserts the bucket-count sum equals the row count and the joint-quadrant
sum equals the row count. This is the suite's structural-correctness
backstop — even if every individual bucket assignment is right, a
floating-point boundary or a missing branch in the dispatch could leave
a row uncounted, and the bulk invariant catches it.

The 8-case bb-state × csT-state cross-product test is the dispatch
exhaustiveness check — bbState ∈ {decisive, not-decisive} × csTState ∈
{decisive-up, decisive-down, not-decisive} = 6 cells, but the test
covers 8 cases including the boundary cases at α exactly. This is the
right shape for testing a 6-bucket dispatch — every reachable cell
covered, plus boundary cases.

Test files added per the CHANGELOG:
`src/classifyaxis216axis215buysballotcoxstuartthirdsweekdayperiodicityvsheadvstailtrendcompound.ts`
(new, ~340 lines) and the matching test file (new, 65 tests). The
all-lowercase no-separator filename convention is the suite's standard
naming pattern — every compound classifier file follows this convention
and every classifier function follows the matching all-camelCase function
name pattern (`classifyAxis216Axis215...`).

## Lineage implications

Axis-216 marks the start of what is structurally a new branch in the
suite's lineage. The trajectory from axis-110 (Mann-Kendall) through
axis-181-188 (van der Waerden / Fligner-Policello / Yuen / Savage /
Brunner-Munzel / permutation Welch-t — the "8-axis location-and-scale
statistical battery") through axis-211-215 (Brown-Mood, Olmstead-Tukey,
Page-L, Theil-Sen, Cox-Stuart-thirds — the "trend battery") has all been
about *signed* tests against *ordered* alternatives, with the compound
classifiers all paired under 4-quadrant direction-conflict frames.

Axis-216 opens a new dimension: *fixed-period periodicity hypothesis tests*.
The natural follow-ups are:

- **axis-217 to axis-219**: other fixed-period ANOVA F-tests at periods
  other than 7 (e.g. period-30 monthly, period-365 annual — though both
  have data-availability issues at the typical pew-insights tenure window).
  The natural next ship is probably *not* another fixed-period F at a
  different period, but rather a *period-7 nonparametric alternative*
  (Friedman's test, or Kruskal-Wallis on the column-folded data) that
  drops the iid Normal residuals assumption that axis-216 inherits from
  the F-distribution under H0.
- **the next compound class**: axis-216 × axis-Fisher-g would be a natural
  ship — the relationship between "period-7 F-test decisive" and
  "Fisher-g periodicity decisive" is the empirical question that
  motivated axis-216 in the first place. A 4-bucket compound (decisive
  intersection / Fisher-only / axis-216-only / neither) would directly
  measure how often the two register the same signal vs orthogonal
  signals, and would settle the orthogonality argument empirically.
- **the 6-bucket scheme as a template**: any future compound joining an
  *undirected* test with a *signed* test (e.g. spectral-flatness ×
  Mann-Kendall, or Levene's centered ANOVA × Theil-Sen) inherits the
  6-bucket scheme axis-216 × axis-215 establishes. The naming conventions
  for the buckets (`<undirected-feature>-and-<signed-direction>` vs
  `<undirected-feature>-only` vs `<signed-direction>-only` vs `no-evidence`)
  also inherit cleanly. So this compound is also a *template-establishing*
  ship, not just a one-off.

The suite is now at 15608 tests across 216 axes plus dozens of compound
classifiers. The v0.6.536 ship is the 535-bump-then-536-compound pattern
that has been the suite's standard cadence for the last several axes
(216 → 216×215, 215 → 215×214, 214 → 214 standalone, 213 → 213×212,
etc.) — every new axis ships standalone first, then immediately gets
compounded with its predecessor in the next minor version. The cadence
is sustainable and the test-coverage delta per ship (typically +60 to
+90 tests per minor version) is consistent with the suite's overall
trajectory.

## Summary

Axis-216 (`9719347` shipping the core, `769c5b6` wiring CLI/format/tests
and bumping to v0.6.535) is the first single-pre-specified-period
hypothesis-test axis in the entire pew-insights suite. Its mechanism is
a Buys-Ballot 1847 period-7 one-way ANOVA F-test on the gap-filled daily
total_tokens series, with the upper-tail p-value computed via a Lentz
continued-fraction evaluation of the regularised incomplete beta accurate
to ~1e-10. The axis is structurally orthogonal to the existing
spectral-flatness / spectral-entropy / Fisher-g periodicity /
iso-weekday-of-week-entropy / weekend-weekday-ratio descriptors via
five distinct mechanisms, and is detrend-orthogonal to the entire
monotone-trend axis family from axis-110 onwards. The live-smoke output
on the pew telemetry corpus shows both surveyed sources sitting on the
indecisive side of α = 0.05 with `bbEta2` orderings opposite to
`bbPValue` orderings — the expected ANOVA finite-sample behaviour.

The immediate v0.6.536 axis-216 × axis-215 compound (`cb3d741` core,
`9807e53` 65 tests, `d418780` CHANGELOG and version bump) is the first
compound classifier in the suite to pair an undirected F-test with a
signed trend test, and accordingly introduces a new 6-bucket scheme
(`weekday-and-up-drift` / `weekday-and-down-drift` / `weekday-only` /
`up-drift-only` / `down-drift-only` / `no-evidence`) that the standard
4-quadrant direction-conflict frame structurally cannot represent. Test
count delta: 15543 → 15608. The 6-bucket scheme is also
template-establishing — any future compound joining an undirected test
with a signed test will inherit this topology and naming.
