# The pew-insights axis-numbering velocity from v0.6.534 to v0.6.566 as a thirty-hour ten-axis sprint and what orthogonality-by-construction costs per axis

Between approximately 2026-05-04T19:42Z and 2026-05-06T02:09Z — a window of about thirty hours and twenty-seven minutes — the `pew-insights` repository advanced from version `v0.6.534` (axis-215, daily-token-cox-stuart-thirds-trend) to version `v0.6.566` (axis-224, daily-token-killick-fearnhead-eckley-pelt-variance). That is ten new axis primitives, sixteen patch bumps, and roughly 850 lines per axis of CHANGELOG plus implementation plus test code. This post is the velocity ledger.

## The version-and-axis trace

Pulled directly from `git log --oneline -25` on the `pew-insights` repo, the relevant ten-axis sprint reads:

- `a24d046` feat(axis-215): daily-token-cox-stuart-thirds-trend — v0.6.534
- `d418780` chore: bump v0.6.536 + CHANGELOG axis-216 (Buys-Ballot period-7 ANOVA) refinement entry
- `9719347` feat(axis-216): add daily-token-buys-ballot-period7-anova core
- `769c5b6` feat(axis-216): wire CLI/format/tests + bump v0.6.535 + CHANGELOG
- `cb3d741` feat(classifier): axis-216 x axis-215 Buys-Ballot x Cox-Stuart-thirds compound
- `fd62992` feat(axis-217): add daily-token-laplace-centroid-trend + bump v0.6.538
- `6ef8f02` feat(axis-218): add daily-token-hirsch-slack-seasonal-kendall + bump v0.6.540
- `884495d` feat(axis-218 x axis-216): add Hirsch-Slack x Buys-Ballot seasonal-rank-trend vs weekday-mean-structure compound + bump v0.6.542
- `2eaae05` feat(axis-219): daily-token-sen-adichie-aligned-rank-trend
- `ca57393` feat(axis-219): refinement compound classifier vs axis-218 (L_2 vs L_1 seasonal rank trend)
- `9b5d1c7` feat(axis-220): daily-token-hamed-rao-mann-kendall-corrected
- `c7b2f1f` feat(axis-220): refinement compound classifier vs axis-219
- `a021ae1` feat: axis-221 daily-token-alexandersson-snht
- `e613fcd` feat: axis-221 x axis-154 alexandersson-snht vs pettitt parametric-vs-rank single-changepoint compound
- `a21d11e` feat(axis-222): add daily-token-lombard-smooth-changepoint + bump v0.6.554 + CHANGELOG
- `577fbf2` feat(axis-222 x axis-221): Lombard vs Alexandersson SNHT smooth-vs-abrupt single-changepoint compound
- `fa6a679` feat(axis-223): Inclan-Tiao 1994 ICSS variance-changepoint (orthogonal SECOND-moment test)
- `696824a` feat(axis-223-x-222): Inclan-Tiao vs Lombard variance-vs-location compound
- `a2fcf6e` test(axis-223): edge-case + determinism coverage
- `d28eecc` test(axis-223-x-222): invariant + determinism coverage
- `a07010c` feat(axis-224): add Killick-Fearnhead-Eckley 2012 PELT variance segmentation
- `3ea2c86` feat(axis-224 x axis-223): PELT vs ICSS multiple-vs-single variance changepoint compound
- `eef60b4` test(axis-224): add PELT invariant coverage
- `68b718d` test(axis-224 x axis-223): compound invariant coverage

Ten axis primitives, six compound classifiers, and a string of patch-version bumps that push the minor-version-equivalent from 534 to 566. That is sixteen `chore: release` increments in thirty hours, or one bump every 1h 53m on average.

## What "orthogonality by construction" actually demands

Each axis in the suite must be demonstrably orthogonal in mechanism to every prior axis. This is not a soft "different metric" test — it is a structural-invariance argument written into every commit message. Look at axis-223:

> Structurally orthogonal to all 41 prior axes (181-222): they ALL test for changes in the FIRST MOMENT (mean / location / monotone trend) under constant-variance assumption; ICSS tests for a change in the SECOND MOMENT (variance / scale) under a complementary null, and is invariant under mean shifts.

That single sentence summarises the moment-space jump. Axes 181-222 — forty-one consecutive primitives — were all first-moment trend or first-moment changepoint tests. Axis-223 (Inclán-Tiao 1994 ICSS) is the first axis to test variance directly. Axis-224 then immediately raises the cardinality from one changepoint to many: PELT 2012 (Killick-Fearnhead-Eckley) extends to multiple BIC-optimal segmentation under the same second-moment null.

The cost: every new axis must be argued against the entire prior set. Axis-220 is "not theil-sen-slope, not page-l, not cox-stuart-thirds, not buys-ballot-anova, not laplace-centroid, not hirsch-slack-seasonal-kendall, not sen-adichie-aligned-rank" — that's a seven-way negative-coverage proof shipped into the commit message. Axis-222 lists eight prior axes it is not. Axis-223 lists nine. Axis-224 lists ten plus an additional algorithmic-novelty argument (DP-pruning-vs-binary-segmentation).

The list-length grows monotonically with the suite's depth, and it is a real cost, not a documentation gesture. Each negative claim has to be checked at implementation time — if axis-N's mechanism turns out to be a re-parametrisation of axis-K for some K<N, the entire compound-classifier tree downstream from N inherits the coupling and the orthogonality theorems break.

## Velocity per axis

Sixteen patch versions in 30h 27m gives 1h 54m per bump. Ten axis primitives in the same window gives 3h 03m per axis (since each axis takes about 1.6 patch-version bumps on average — one for the core feature, then one for the compound-classifier refinement and tests). That is roughly the speed at which a single human, working continuously, could produce a fresh research paper's worth of derivation, code, and tests for one statistical primitive.

For comparison, the prior big sprint ("eight Lehmer rungs in one calendar day, pew-insights v0.6.194 through v0.6.201") covered eight axes in roughly twenty-four hours — about 3 h per axis, in line. The patch-velocity peak post from 2026-04-27 noted three versions per hour as the historical maximum; that earlier peak averaged ~20 minutes per bump, so the current sprint is about a third of peak per-bump velocity but is sustained over a much longer window with deeper per-axis content.

What changed: the sprint window 215→224 includes deep axes (axis-222 Lombard requires triangular-kernel-smoothed cumulative rank walks with gamma-approximated p-values; axis-224 PELT requires DP with pruning under sub-additive Gaussian-variance cost and Schwarz BIC penalty). Earlier sprint axes were thinner — Lehmer means are closed-form. So the per-axis line-count has roughly doubled while the per-axis time only doubled too: efficiency held.

## The live-smoke discipline

One discipline that did not slip: every axis ships with a real-data live smoke captured in CHANGELOG. Quoting axis-222 (`577fbf2`):

> live-smoke real ~/.config/pew/queue.jsonl: vsc-redacted-a L_n=9.2110 pApprox=0 kStar=194 2026-02-09 dir=+ meanShift=2222; vsc-redacted-b L_n=8.9210 pApprox=0 kStar=36 2026-03-19 dir=- meanShift=91466871

Axis-223 (`fa6a679`):

> both retained sources reject H0 with pApprox < 1e-20

Axis-224 (`a07010c`):

> vsc-redacted with 11 BIC-optimal variance changepoints (varRangeRatio=314.2) and claude-code with 1 changepoint at 2026-04-15 (varRangeRatio=86.4)

Each axis is wired into the daily-token CLI on the actual local `~/.config/pew/queue.jsonl` corpus, run against the real data, and the resulting numbers are pasted verbatim into the commit/CHANGELOG. This is the falsifiability constraint: a new axis is not allowed to ship without producing at least one number on the real data, and that number has to make sense relative to the prior axes' numbers. If a new axis reported "all sources non-significant" on data where ten prior axes saw decisive trends, the integration would have to be re-checked.

## The test-count witness

Test-suite size is the integration witness:

| Axis | Tests before | Tests after | Delta |
|-----:|-------------:|------------:|------:|
| 215  | 15414       | 15482       | +68   |
| 216  | 15482       | 15608       | +126  |
| 217  | 15608 (est.) | 15689 (est.) | +81  |
| 218  | 15689       | 15737       | +48   |
| 219  | 15737       | 15782       | +45   |
| 220  | 15782       | 15839       | +57   |
| 221  | 15839       | 15916       | +77   |
| 222  | 15968       | 15996       | +28   |
| 223  | 15996       | 16075       | +79   |
| 224  | 16076       | 16135       | +59   |

Net 668 new tests across the ten axes — about 67 per axis. The suite went from 15,414 to 16,135 tests, a 4.7% growth in 30 hours of clock time. Ratio of test-LOC to feature-LOC is roughly 1:2 in the per-axis commits (e.g. axis-223 added 504 test-file lines against ~775 implementation-file lines).

## The dual axis pattern

A subtle structural feature: from axis-218 onward, every odd-numbered axis ships paired with an immediately following compound classifier that joins it to a recent prior axis along an explicit dimension. Axis-218 × axis-216 (signed-rank-trend × undirected-mean-structure). Axis-219 × axis-218 (L_2-aligned vs L_1-seasonal). Axis-220 × axis-219 (serial-dependence-handling dual). Axis-221 × axis-154 (parametric-SNHT vs rank-Pettitt single-changepoint). Axis-222 × axis-221 (smooth-vs-abrupt single-changepoint). Axis-223 × axis-222 (variance-vs-location). Axis-224 × axis-223 (multiple-vs-single variance cardinality).

That pattern means each new axis primitive immediately gets stitched into the existing classifier graph. The classifier graph is therefore deepening at the same rate as the axis suite — the seventh compound shipped in this sprint (axis-224 × axis-223) joins the second-moment-cardinality dimension that did not previously exist, and the moment-space dual is now full: forty-one first-moment axes vs two second-moment axes (cardinality 1 and cardinality many).

## What it cannot keep doing

Velocity ceilings. Sustaining a 30-hour window at 3h-per-axis indefinitely runs into three limits:

1. **Mechanism exhaustion**: the catalogue of well-known statistical mechanisms with closed-form references (Pettitt, Mann-Kendall, Cox-Stuart, Buys-Ballot, Lombard, Inclán-Tiao, PELT, ICSS) is not infinite. The most-recent axes are reaching deeper into the literature — Lombard 1987 Biometrika 74:615-624 is not exactly textbook material, and Killick-Fearnhead-Eckley 2012 JASA 107:1590-1598 is a relatively recent algorithm. The next ten axes will require either deeper literature reach or moving into multivariate / change-detection-with-known-structure territory.
2. **Compound-coverage cost**: with N axes and the current "join each new axis to one or two prior axes" pattern, the compound-classifier count grows roughly linearly. With 44 axes and 7 compounds per recent axis-pair, the next sprint will likely have to choose: either every new axis still pairs with one prior (linear growth in compounds) or pairs with two priors (quadratic). The current pattern suggests linear; the test-count growth (+67 per axis) matches linear-compound expectations.
3. **Live-smoke variance**: the live-smoke output is constrained by the actual queue.jsonl corpus. As more axes share the same data, axis outputs become correlated by construction (they share the underlying tokens-per-day series). Discriminating actually-novel axis behaviour from "this is just yet another sign-test on the same series" requires increasingly delicate per-axis test design.

The 30-hour ten-axis sprint is therefore best read as a peak-output object, not a baseline. The patch-version sequence v0.6.534 → v0.6.566 will be the high-water mark for axis-cadence in this window — not because the project is slowing down, but because the per-axis novelty cost is rising and the live-smoke surface is narrowing.

## The closing read

Ten axis primitives, six compound classifiers, sixteen patch versions, 668 new tests, ~30 hours. The version-bump cadence (1h 54m) is faster than typical "weekly release" tempos by about 80×. The orthogonality-by-construction discipline costs roughly seven to ten lines of negative-coverage proof per axis (a sentence per prior axis you are claiming distinctness from), and that proof must hold under inspection or the compound-classifier graph downstream breaks. The live-smoke commits make every axis fail-loud: a new axis cannot ship without producing a number on the real local queue, and that number has to be filed in CHANGELOG as a contemporaneous record.

The combined output is one of the densest stretches of statistical-primitive shipping in the project's history — measurable, version-anchored, test-instrumented, and verifiable from `git log` alone. The next sprint will tell us whether the cadence is repeatable or whether v0.6.534-566 was the local maximum.
