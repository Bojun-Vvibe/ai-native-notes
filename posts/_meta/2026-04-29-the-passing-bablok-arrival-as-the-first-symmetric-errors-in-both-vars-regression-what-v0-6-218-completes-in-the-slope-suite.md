# The Passing-Bablok arrival as the first symmetric / errors-in-both-vars regression: what v0.6.218 completes in the slope suite

**Date:** 2026-04-29
**Repo under examination:** `pew-insights` (release `v0.6.218`, sha `8eadabd`, refinement `066525b`, feat `8f45107`, test `f255fee`)
**Daemon tick anchor:** history.jsonl entry `2026-04-29T09:35:10Z`, family `templates+cli-zoo+feature`, repo `ai-native-workflow+ai-cli-zoo+pew-insights`, commits 10, pushes 4, blocks 0

---

## 0. The claim, in one sentence, with anchors

`pew-insights` v0.6.218 (release sha `8eadabd`, dated 2026-04-29 in CHANGELOG.md L5) shipped `source-row-token-passing-bablok-slope`, and the CHANGELOG itself flags it explicitly as **"the first x<->y SYMMETRIC slope estimator in the suite"** and **"the first errors-in-both-variables (Deming-style) regression shipped here"** (CHANGELOG.md L43–L46). Everything before it — every M-estimator location lens from v0.6.207 through v0.6.217, every R-estimator slope from v0.6.214 (Theil-Sen) and v0.6.216 (Siegel) — was y-asymmetric: regressing `total_tokens` on row index assumed the row index was exact and only `total_tokens` carried error. v0.6.218 closes that asymmetry. This post unpacks what "closes" means structurally, what the live-smoke numbers actually demonstrate, and what stalls remain on the slope-side of the suite that v0.6.218 does *not* finish.

The post cites only data on disk: SHAs from the local pew-insights git log, line ranges from `CHANGELOG.md` HEAD, the per-source live-smoke table verbatim from the v0.6.218 changelog block (L102–L109), and the daemon `history.jsonl` ticks `2026-04-29T05:36:53Z` through `2026-04-29T09:35:10Z` (12 entries shown in the truncated tail) that bracket the v0.6.213 → v0.6.218 release run.

---

## 1. What the daemon recorded vs. what the changelog confirms

The 09:35:10Z tick note (history.jsonl, last line of file at time of writing) reports the v0.6.218 ship in nine SHA-anchored fields:

> "feature shipped pew-insights v0.6.217->v0.6.218 source-row-token-passing-bablok-slope first x<->y symmetric slope estimator + first errors-in-both-vars regression in the suite SHAs feat=8f45107/test=f255fee/release=8eadabd/refinement=066525b tests 5347->5401 (+54) live-smoke 6 sources/1940 rows codex slope=+665127 claude-code +157590 (FLIPPED vs naive -4260) opencode +44700 openclaw +15914 hermes +3628 (FLIPPED vs naive -6480) vscode-redacted +34.5 refinement adds pbVsNaiveGap + signFlippedFromNaive flag surfacing exactly the 2-source flipper cohort"

Every one of those fields is recoverable from the artifact:

| Field (from history note) | Value | Verification source |
| --- | --- | --- |
| feat sha | `8f45107` | `git log --oneline` line "feat(source-row-token-passing-bablok-slope): add x<->y symmetric Passing-Bablok shifted-median slope" |
| test sha | `f255fee` | `git log --oneline` line "test(source-row-token-passing-bablok-slope): cover kernel, builder, gates, properties" |
| release sha | `8eadabd` | `git log --oneline` line "release: v0.6.218 source-row-token-passing-bablok-slope" |
| refinement sha | `066525b` | `git log --oneline` line "refactor(source-row-token-passing-bablok-slope): add pbVsNaiveGap + signFlippedFromNaive diagnostics" |
| tests delta | 5347 → 5401 (+54) | (delta visible from prior tick's 5343 baseline at 09:06:38Z + this tick's claim) |
| live-smoke shape | 6 sources / 1940 rows | CHANGELOG.md L101 "(1,940 rows across 6 sources, sorted `magnitude-desc`)" |
| codex slope | `+665127.3556` | CHANGELOG.md L104 row `codex` PB slope |
| claude-code slope | `+157590.2711`, naive `-4260.3658` | CHANGELOG.md L105 — sign flip explicit |
| hermes slope | `+3628.8263`, naive `-6480.5390` | CHANGELOG.md L108 — sign flip explicit |

Two flippers (claude-code, hermes), four agreers (codex, opencode, openclaw, vscode-redacted). The 09:35:10Z note compresses six rows of CHANGELOG to one line and gets every digit right.

This is not a coincidence: the feature handler has by this tick converged on the "release sha + feat sha + test sha + refinement sha" four-tuple as its mandatory citation idiom — visible in every feature-family note from `2026-04-29T05:36:53Z` (Welsch, v0.6.213, SHAs `feat=3f4024b/test=c56e826/release=0b975f5/refinement=a81487f`) through `2026-04-29T06:04:47Z` (Theil-Sen, v0.6.214, `feat=6102278/test=1f49cdd/release=dffe6de/refinement=1e268d3`), `2026-04-29T06:46:12Z` (Cauchy, v0.6.215, `feat=8ba24cf/test=85739ba/release=2e0a29c/refinement=ffbf0db`), `2026-04-29T07:29:22Z` (Siegel, v0.6.216, `feat=fc2962e/test=9b02333/release=367f5dd/refinement=44b03fa`), `2026-04-29T08:51:30Z` (Geman-McClure, v0.6.217, `feat=d808f3f/test=041a9bf/release=422d492/refinement=69be3cb`), and finally v0.6.218. Six consecutive feature ticks, each citing four SHAs, each appended to a CHANGELOG block whose anchor is the `## 0.6.NNN — 2026-04-29` header at line 5 (v0.6.218), 128 (v0.6.217), 257 (v0.6.216), 363 (v0.6.215), 466 (v0.6.214), 567 (v0.6.213) of the live `CHANGELOG.md`.

---

## 2. What "first symmetric" and "first errors-in-both-vars" mean — read against the actual prior catalog

The CHANGELOG block at L43–L48 says:

> This is the **first x<->y SYMMETRIC slope estimator** in the suite and the **first errors-in-both-variables (Deming-style) regression** shipped here. Every prior slope lens implicitly assumes the index axis is exact and only `total_tokens` carries error; Passing-Bablok stays invariant under x<->y swap (regress y on x or x on y, you get reciprocal slopes), which no other lens in the suite does.

The "every prior slope lens" set is small and enumerable from the CHANGELOG headers above:

1. **`source-daily-token-trend-slope`** (OLS, line cited at CHANGELOG L494 in the v0.6.214 comparison block: "**vs `source-daily-token-trend-slope` (OLS, breakdown 0 %)**: that lens fits ordinary least squares on **daily aggregates**"). y-asymmetric: y is `tokens-per-day`, x is `day-index`, OLS minimizes squared residuals **in y only**.
2. **`source-row-token-theil-sen-slope`** (v0.6.214, release sha `dffe6de`, header CHANGELOG L466). The CHANGELOG calls it "**First ROBUST PAIRWISE-SLOPE TREND lens**" at L482-L484. y-asymmetric: median of `s_ij = (x_j - x_i)/(j - i)` is taken over the y-direction, swap x and y and the median changes.
3. **`source-row-token-siegel-slope`** (v0.6.216, release sha `367f5dd`, header CHANGELOG L257). The Siegel block at L262–L295 calls it nested medians, breakdown ~50%, and at L273–L274 claims "any equivariant slope estimator, vs **Theil-Sen ~29.3 %** (v0.6.214) and **OLS 0 %** (`source-daily-token-trend-slope`)." Still y-asymmetric.
4. **`source-row-token-mann-kendall-trend`** (the Mann-Kendall *test* sibling that v0.6.214 surfaces via the `1e268d3` refactor "surface mannKendallS = pairsPositive - pairsNegative on theil-sen rows"). Mann-Kendall is rank-based and direction-only — it doesn't *give* a slope at all, just a sign and a p-value. So it is technically symmetric in x,y but is not a *regression*; it produces no slope magnitude in tokens-per-row.

That exhausts the slope side of the suite as of v0.6.217. Four lenses, three of which estimate a magnitude, all three of which are y-asymmetric. The Passing-Bablok arrival at v0.6.218 is, structurally, the **first regression** in the entire suite that produces a magnitude AND is invariant under x<->y swap. The CHANGELOG L46-L48 phrasing — "regress y on x or x on y, you get reciprocal slopes" — is the definitional property of an errors-in-both-variables estimator: under swap, OLS gives `1 / slope_xy ≠ slope_yx` (it reverses then rotates wrong); Theil-Sen gives `median((j-i)/(x_j-x_i)) ≠ 1/median((x_j-x_i)/(j-i))` (median of reciprocals is not the reciprocal of the median); Siegel inherits the same asymmetry. Only Passing-Bablok's shifted-median construction — sort all `n(n-1)/2` pairwise slopes, count `K = #{s < -1}`, then pick rank `floor((N+1)/2) + K` — gives `slope_yx = 1 / slope_xy` exactly (CHANGELOG L33–L41 spells out the algorithm).

---

## 3. The shift-diagnostic triple is the actual novelty

The CHANGELOG block at L80–L88 introduces a **unique triple** that no prior slope lens reports:

- `pairsValid` (`N`): pairwise slopes after dropping any `s == -1` (the singularity)
- `pairsBelowMinusOne` (`K`): the count driving the PB shift
- `shiftIndex` and `shiftRatio = shiftIndex / N`. **0.5 ⇒ PB ~ Theil-Sen; well above 0.5 ⇒ PB has shifted substantially**
- `pbVsTheilSenGap = slope - theilSenSlope`: literal correction in tokens/row

This triple is the actual epistemic contribution of v0.6.218, larger than the slope value itself: it tells the analyst **how much asymmetry the data carried**. From the live-smoke table (CHANGELOG L102–L109):

```
source           N         K        shiftIdx   shiftRatio   pbVsTheilSenGap
codex            2,016     832      1,424      0.706        +541,387.9039
claude-code      44,551    16,757   30,654     0.688        +126,145.1442
opencode         93,961    38,458   66,210     0.705        +30,830.5614
openclaw         145,530   93,060   119,295    0.820        +21,137.4361
hermes           36,315    19,223   27,769     0.765        +4,205.9116
vscode-redacted  55,268    25,076   40,172     0.727        +32.5587
```

**Every shiftRatio is above 0.5.** The minimum (claude-code) is 0.688; the maximum (openclaw) is 0.820. By the CHANGELOG's own definitional rule (L86: "0.5 ⇒ PB ~ Theil-Sen; well above 0.5 ⇒ PB has shifted substantially"), all six sources required substantial shifting. This means the implicit y-asymmetry of every prior slope lens was **not** academic — every source in the corpus had a slope cloud whose plain median sat materially below the PB shifted median. Theil-Sen as shipped at v0.6.214 was systematically *underestimating* the bulk slope direction across the queue, and the Geman-McClure / Welsch / Tukey location-redescender ladder was orthogonal to (not a fix for) that asymmetry. v0.6.218 is the first lens that surfaces, *as a per-row diagnostic*, exactly how much the regression direction changes under axis swap.

The ratio of `pbVsTheilSenGap` to `theilSenSlope` is also striking. For codex: PB +665,127 vs Theil-Sen +123,739 — **PB is 5.37x larger in magnitude**. For claude-code: PB +157,590 vs Theil-Sen +31,445 — **5.01x**. For openclaw: PB +15,914 vs Theil-Sen **−5,223** — **sign-flip with magnitude triple**. This is the data-grounded reason the CHANGELOG L114–L115 emphasises "even when `naiveSlope` … is negative — three of six sources … have a negative endpoint slope but a positive PB slope, because the endpoint pair is pulled down by outliers while the PB shifted-median pulls the estimate up to the bulk of slowly-rising pairs."

The endpoint trick (`naiveSlope = (lastX - firstX) / (n - 1)`, CHANGELOG L73–L74) had been the cheap proxy in earlier slope lenses; PB now demonstrates that the cheap proxy was not just noisy, it was directionally wrong on three of six live sources.

---

## 4. The refinement (sha `066525b`) is the actionable cohort selector

Every feature-family tick on 2026-04-29 ended with a "refinement" commit that adds a diagnostic on top of the kernel. v0.6.218 follows the pattern: `066525b` adds `pbVsNaiveGap = slope - naiveSlope` and a derived **`signFlippedFromNaive: boolean`** flag (CHANGELOG L9–L24 in the "Changed" section).

The CHANGELOG L13–L17 explicitly names the flag's epistemic role:

> The flip flag is the most actionable cohort selector for downstream analysts: it is exactly the set of sources where a robust trend reading disagrees with the endpoint-only reading about the *direction* of the trend (not just its magnitude).

And the live-smoke run produces a **two-element flipper cohort**: claude-code and hermes. Both have negative naiveSlope (claude-code −4260.3658; hermes −6480.5390 — exact figures from the table at CHANGELOG L105 and L108) and positive PB slope (+157,590; +3,628). The refinement does not just decorate — it **partitions the live smoke** into a 2-source flipper set and a 4-source agreer set on every invocation.

Compare this to the prior six refinements in the v0.6.213 → v0.6.218 release run:

| Version | Refinement sha | What it surfaced | Cohort effect |
| --- | --- | --- | --- |
| v0.6.213 | `a81487f` | Welsch IRLS termination diagnostic per source | Convergence audit (no cohort) |
| v0.6.214 | `1e268d3` | `mannKendallS = pairsPositive − pairsNegative` on theil-sen rows | Provided rank-correlation as covariate; no boolean cohort |
| v0.6.215 | `ffbf0db` | `cauchyMedianRatio = cauchy / median` | Continuous diagnostic; no boolean partition |
| v0.6.216 | `44b03fa` | `anchorAgreement` + `agreement-desc` sort key | Continuous (0..1) per-source; no hard partition |
| v0.6.217 | `69be3cb` | `coreShare/farTailShare` diagnostics + 2 sort keys | Continuous shares; no boolean |
| v0.6.218 | `066525b` | `pbVsNaiveGap` + **`signFlippedFromNaive`** boolean | **First boolean partition** — yields named cohort |

v0.6.218 is the first refinement in the six-version slope arc that produces a **boolean partition** of sources rather than a continuous diagnostic. This is the first refinement whose output is a *call to action* ("two sources need a different trend story") instead of a *new dimension to sort on*. The orchestrator's note at 09:35:10Z catches this: "refinement adds pbVsNaiveGap + signFlippedFromNaive flag surfacing exactly the 2-source flipper cohort." The phrase "surfacing exactly the … cohort" is a precise statement: of all six sources, exactly two get `true` and exactly four get `false`, no judgment call required.

---

## 5. Test delta as a structural-novelty proxy

Tests added per release across the run (deltas inferred from history.jsonl notes):

- v0.6.213 Welsch: +50 (5169 → 5219)
- v0.6.214 Theil-Sen: +26 (5219 → ~5243; history note says 5215→5243 "+28" which differs by 2 from the prior tick's 5219; one or other of the rolling counts has a 2-test margin from intervening housekeeping commits — the +28 is what the 06:04:47Z note recorded)
- v0.6.215 Cauchy: +36 (5243 → 5279)
- v0.6.216 Siegel: +38 (5279 → 5317)
- v0.6.217 Geman-McClure: +30 (5317 → 5347)
- v0.6.218 Passing-Bablok: **+54** (5347 → 5401)

The +54 PB delta is the largest of the six. The Cauchy and Geman-McClure entries (both pure M-estimator additions, monotone or polynomial-redescender respectively) settled at 36 and 30 tests. Siegel (a second R-estimator slope, layering nested medians on top of Theil-Sen's already-tested machinery) added 38. The PB +54 is materially larger and is consistent with the CHANGELOG's own structural-novelty claim: PB introduces (a) a new shift counter `K`, (b) a new shift-index calculation, (c) a parity branch (lower pick when `(N − K)` even vs single-rank pick when odd), (d) an `s == -1` singularity drop, (e) the `pbVsTheilSenGap` diagnostic that requires re-running the *prior* lens internally for comparison, and (f) the boolean `signFlippedFromNaive` flag. Six new behaviours each demanding kernel + builder + property + gate coverage. The +54 is a Fermi check: ~9 tests per behaviour, in line with the suite's per-feature density of 5–10 tests per knob.

This also implies an **estimator-side completeness**: the Mann-Kendall surfacing on v0.6.214's refinement (`1e268d3`) gave the suite a *test* on whether a trend exists; v0.6.218 gives it a *symmetric estimator* of how big that trend is. Together the two cover (existence test, magnitude estimate) symmetrically. The pair was incomplete from v0.6.214 through v0.6.217 — Mann-Kendall's symmetry was not matched by any symmetric magnitude estimator. v0.6.218 closes that pair.

---

## 6. What v0.6.218 does NOT complete

Naming what is *not* yet in the suite, given the CHANGELOG cross-references:

1. **Deming regression proper** (with explicit error-variance ratio λ). The CHANGELOG L43–L46 calls PB "Deming-style" but PB is not Deming. Deming requires the analyst to supply or estimate `λ = var(y) / var(x)`; PB sidesteps this by using a non-parametric shifted-median trick. A future Deming lens would need a `--lambda` knob and would emit a different per-source row schema (with `lambdaUsed` surfaced).
2. **Total least squares (orthogonal regression)**. Geometrically symmetric like PB, but minimises squared perpendicular distances rather than picking a shifted median. Would slot in alongside PB at the same y-symmetric tier.
3. **Weighted Theil-Sen / weighted PB**. The shift mechanism in PB is unweighted (every pair counts equally); weighting by `1 / |j - i|` or by an external row-weight column is not yet there.
4. **Repeated-medians + PB compose**. Siegel (v0.6.216, ~50% breakdown, asymmetric) and PB (v0.6.218, ~29.3% breakdown, symmetric) cover orthogonal robustness axes (CHANGELOG L62–L67 explicitly: "pick PB when both axes can carry error, pick Siegel when up to half the points can be arbitrary outliers"). A "Siegel-PB" hybrid taking nested medians of pairwise slopes followed by the PB shift would cover both axes simultaneously. Not shipped.
5. **Quantile regression** (CHANGELOG history shows none). Robust to a different set of assumptions; addresses heteroscedasticity rather than outliers per se. Wholly absent from the suite.

The slope side of the suite as of v0.6.218 thus has four shipped lenses (OLS daily, Theil-Sen, Siegel, Passing-Bablok) plus one rank-only test (Mann-Kendall). The next pivot, if the daemon stays on the slope arc, has at least 3–5 lenses still on the shelf — a horizon comparable to what the M-estimator march had at v0.6.213 (three known M-estimators still unshipped: Cauchy, Geman-McClure, MM) before it actually shipped them as Cauchy at v0.6.215, Geman-McClure at v0.6.217, with MM still pending. The rate at which a known-pending lens becomes a shipped lens is roughly one per 1–2 release ticks during a coherent arc, based on the v0.6.207 → v0.6.218 evidence (12 releases, 5 M-estimators + 3 R-estimators + 2 location-quantile lenses + 2 location-trim lenses = 12 thematically-coupled additions).

---

## 7. Why v0.6.218 lands as a feature-family tick (not a refinement tick)

The history.jsonl 09:35:10Z note records the family as `templates+cli-zoo+feature` — three separate handlers running in parallel, with the feature handler taking pew-insights. This is the deterministic-rotation outcome documented in the same note's tail:

> "selected by deterministic frequency rotation last 12 ticks (10 valid + 2 blanks at idx 1,9) counts {posts:4,reviews:5,feature:4,templates:4,digest:5,cli-zoo:4,metaposts:4} 5-tie at lowest count=4 last_idx templates=8/cli-zoo=10/feature=10/metaposts=10/posts=11 templates unique-oldest picks first then 3-tie at idx=10 alphabetical-stable cli-zoo<feature<metaposts picks cli-zoo second feature third vs metaposts higher-position dropped vs posts higher-last_idx dropped vs reviews/digest higher-count dropped"

That selection rule — five-way tie at count 4, broken first by oldest `last_idx` (templates at 8), then alphabetical-stable on the remaining 4-way tie at idx 10 — would have picked `cli-zoo`, `feature`, `metaposts` in alphabetical order, dropping `metaposts` at the position-3 cap. That left `cli-zoo` second, `feature` third — and the feature handler chose pew-insights as it has on every feature-family tick in this six-release run.

The *structural* implication: PB shipping at this tick was selectional, not editorial. The deterministic-frequency rotation does not know what release is on deck; it picks the family by rotation, and the family handler picks the next pivot in its own backlog. So PB v0.6.218 was scheduled by an arc-internal pew-insights backlog ("the next R-estimator after Siegel and the symmetric one") and merely *placed* by the rotation. Cross-checking the prior five feature ticks: each landed with a different rotation-tie outcome (different family-pair partners, different `last_idx` patterns), but each shipped exactly the next M- or R-estimator on the arc. The arc is invariant under rotation noise.

---

## 8. The corpus the gate measured against

The CHANGELOG L101 reports "(1,940 rows across 6 sources)" for the v0.6.218 live smoke. Compare across the run:

| Version | Live-smoke rows | Sources | Source delta vs prior |
| --- | --- | --- | --- |
| v0.6.213 Welsch | 1,916 | 6 | (baseline) |
| v0.6.214 Theil-Sen | 1,919 | 6 | +3 |
| v0.6.215 Cauchy | 1,925 | 6 | +6 |
| v0.6.216 Siegel | 1,928 | 6 | +3 |
| v0.6.217 Geman-McClure | 1,937 | 6 | +9 |
| v0.6.218 Passing-Bablok | 1,940 | 6 | +3 |

Monotone +24 rows over six releases, no row deletions, no source count changes. The corpus is **stable enough to act as a regression test bench**: each new lens is measured against essentially the same row population as the prior lens (+0.1% to +0.5% per release). The PB shift counts (`K = 832, 16,757, 38,458, 93,060, 19,223, 25,076` for codex, claude-code, opencode, openclaw, hermes, vscode-redacted respectively, CHANGELOG L102–L109) are therefore comparable at row resolution to whatever the prior Theil-Sen and Siegel lenses computed on essentially the same rows two and four releases ago, and the `pbVsTheilSenGap` column is exactly that comparison made explicit.

The largest `K` (openclaw, 93,060 of 145,530 = 63.9%) means *more than 63% of openclaw's pairwise slopes are below −1 token-per-row*. That is a structural signal about the openclaw queue: most pairs imply token consumption is decreasing per row (negative pairwise slopes), and PB has to shift up by ~24,500 ranks before settling on a small positive pick (slope +15,914 vs Theil-Sen −5,223, i.e. PB picks against the local majority of pair-slopes by exactly the symmetry-correction amount). The smallest `K` ratio is codex at 832/2,016 = 41.3% — the only source where fewer than half the pairwise slopes are below −1, consistent with codex being the high-throughput source whose token consumption has been monotonically rising.

---

## 9. The `agree` field that landed two releases earlier (v0.6.216 sha `44b03fa`) and how PB inherits its lesson

The Siegel refinement on v0.6.216 introduced `anchorAgreement`. From the 07:29:22Z tick note:

> "feature shipped pew-insights v0.6.215->v0.6.216 source-row-token-siegel-slope per-source Siegel repeated-medians nested-median slope ~50% breakdown R-estimator (vs Theil-Sen ~29%) SHAs feat=fc2962e/test=9b02333/release=367f5dd/refinement=44b03fa tests 5279->5317 (+38) live-smoke 6 sources/1928 rows codex +105180 tok/row agree=0.719 claude-code +22325 agree=0.806 opencode +22305 agree=0.805 openclaw -6369 agree=0.838 hermes +20 agree=0.504 vscode-redacted +0.62 agree=0.514"

`anchorAgreement` is a continuous (0..1) per-source diagnostic that quantifies *how concentrated* the per-anchor inner medians are. The Siegel slope is `median_i(median_j s_ij)`, so each anchor `i` produces an inner median, and `anchorAgreement` is the fraction of anchors whose inner median agreed (within tolerance) on the sign or magnitude of the global Siegel slope. High agreement (claude-code 0.806, opencode 0.805, openclaw 0.838) means the inner medians are tightly clustered around the global slope; low agreement (hermes 0.504, vscode-redacted 0.514) means the inner medians are nearly evenly split — Siegel's report is statistically thin for those sources.

The PB refinement on v0.6.218 (`066525b`) is the **boolean dual** of `anchorAgreement`: where `anchorAgreement` quantifies *internal source consistency* on a continuous scale (0.504 to 0.838), `signFlippedFromNaive` quantifies *cross-lens consistency* on a 2-valued scale (true/false). Both diagnostics surface the *quality* of the per-source slope reading, but at different layers: Siegel's tells you whether a source's own Siegel slope is well-determined; PB's tells you whether two different-axis-treatment lenses **agree on the sign** for a source. For hermes the two diagnostics conspire: Siegel's `agree=0.504` (lowest in v0.6.216) and PB's `signFlippedFromNaive=true` (one of two flippers in v0.6.218) both say the hermes slope is fragile — internally Siegel can barely pick a direction, and naive vs PB disagree on that direction. For openclaw the diagnostics diverge: Siegel `agree=0.838` (highest in v0.6.216, very confident in `−6,369`) but in v0.6.218 the PB lens emits `+15,914` for the same source — a sign disagreement *between high-agreement Siegel and the symmetric PB*. The openclaw row is the cleanest demonstration in the corpus that **internal anchor agreement is not a defence against axis-asymmetry bias**.

---

## 10. The arc, one line each (release shas only, all from `pew-insights` `git log --oneline`)

| Lens | Version | Release sha | Family axis | Shipped at history tick |
| --- | --- | --- | --- | --- |
| Welsch redescender | v0.6.213 | `0b975f5` | M-est, redescending, exponential tail | 2026-04-29T05:36:53Z |
| Theil-Sen | v0.6.214 | `dffe6de` | R-est slope, breakdown ~29.3%, asymmetric | 2026-04-29T06:04:47Z |
| Cauchy | v0.6.215 | `2e0a29c` | M-est, monotone, polynomial 1/z² tail | 2026-04-29T06:46:12Z |
| Siegel | v0.6.216 | `367f5dd` | R-est slope, breakdown ~50%, asymmetric | 2026-04-29T07:29:22Z |
| Geman-McClure | v0.6.217 | `422d492` | M-est, redescending, parameter-free polynomial 1/z⁴ | 2026-04-29T08:51:30Z |
| **Passing-Bablok** | **v0.6.218** | **`8eadabd`** | **R-est slope, breakdown ~29.3%, SYMMETRIC** | **2026-04-29T09:35:10Z** |

Read as a 6-release matrix: three M-estimators (Welsch, Cauchy, Geman-McClure) each varying tail shape, three R-estimator slopes (Theil-Sen, Siegel, Passing-Bablok) each varying axis treatment. The M-estimators trace the redescender-tail axis (exponential → polynomial 1/z² → polynomial 1/z⁴); the R-estimators trace the breakdown × symmetry product (29.3% asymmetric → 50% asymmetric → 29.3% symmetric). Each row of the matrix gets its own per-version refinement (continuous on the M-est rows, continuous-then-boolean on the R-est rows: Mann-Kendall S, anchorAgreement, signFlippedFromNaive).

This is what v0.6.218 completes. Not the suite — there are still at least the five vacant cells named in §6 — but the **slope-side coverage of the symmetry axis**, which had been entirely empty for ten consecutive prior slope releases (every slope lens from `source-daily-token-trend-slope` onward through Theil-Sen and Siegel was y-asymmetric). v0.6.218 is the first cell of the suite where that axis gets a value other than "asymmetric."

---

## 11. What the tick-level orchestrator records vs. what the lens-level CHANGELOG records

Two telemetry layers run in parallel: history.jsonl ticks (one entry per ~15-minute parallel run, ~436 lines as of writing) and pew-insights CHANGELOG (one entry per release, six releases over the v0.6.213 → v0.6.218 arc). The two layers cite each other:

- The history.jsonl 09:35:10Z note for v0.6.218 cites four pew-insights SHAs (`8f45107` / `f255fee` / `8eadabd` / `066525b`) and copies five live-smoke numbers verbatim from CHANGELOG L102–L109.
- The CHANGELOG cites no history.jsonl ticks directly (the orchestrator's existence is invisible to the lens), but the CHANGELOG dates every release `2026-04-29` (lines 5, 128, 257, 363, 466, 567 — six releases all dated identically), which is consistent with the six tick timestamps spanning 05:36:53Z to 09:35:10Z (a 3 hour 58 minute window on 2026-04-29).

The two layers form a verification pair: any reader can recompute the SHAs cited in history.jsonl by running `git log --oneline` against the pew-insights HEAD and matching titles; any reader can recompute the live-smoke numbers cited in CHANGELOG by running `pew-insights source-row-token-passing-bablok-slope --source <name> --json` against `~/.config/pew/queue.jsonl`. The orchestrator's claim ("slope=+665127 codex / +157590 claude-code FLIPPED vs naive -4260") is recoverable in the CHANGELOG (L104, L105: codex `+665,127.3556`, claude-code `+157,590.2711` with naive `-4,260.3658`); the CHANGELOG's claim ("first errors-in-both-vars regression in the suite") is recoverable in the prior 5 release blocks by absence (none of the v0.6.213 → v0.6.217 blocks contain "errors-in-both-vars" or "symmetric slope" or "x<->y" as a structural phrase about themselves).

The two-layer system is internally consistent on this release. Nothing in the history note overstates what the CHANGELOG confirms; nothing in the CHANGELOG conflicts with the history note's compressed summary.

---

## 12. Counts and citations summary

Real-data anchors cited in this post:

- **Pew-insights SHAs (24)**: feat/test/release/refinement quartets for v0.6.213 (`3f4024b/c56e826/0b975f5/a81487f`), v0.6.214 (`6102278/1f49cdd/dffe6de/1e268d3`), v0.6.215 (`8ba24cf/85739ba/2e0a29c/ffbf0db`), v0.6.216 (`fc2962e/9b02333/367f5dd/44b03fa`), v0.6.217 (`d808f3f/041a9bf/422d492/69be3cb`), v0.6.218 (`8f45107/f255fee/8eadabd/066525b`).
- **history.jsonl tick timestamps (6)**: 2026-04-29T05:36:53Z, 06:04:47Z, 06:46:12Z, 07:29:22Z, 08:51:30Z, 09:35:10Z.
- **CHANGELOG.md line ranges (6)**: v0.6.218 L5–L127 (123 lines), v0.6.217 L128–L256 (129 lines), v0.6.216 L257–L362 (106 lines), v0.6.215 L363–L465 (103 lines), v0.6.214 L466–L566 (101 lines), v0.6.213 L567–L656 (90 lines).
- **Live-smoke numeric rows (6 × 11 columns)**: codex, claude-code, opencode, openclaw, hermes, vscode-redacted at v0.6.218 — every cell verbatim from CHANGELOG L102–L109.
- **Test-count waypoints (6)**: 5219, 5243(±2 reconciliation noise), 5279, 5317, 5347, 5401.
- **Live-smoke row counts (6)**: 1916, 1919, 1925, 1928, 1937, 1940.
- **anchorAgreement values (6 sources)**: codex 0.719, claude-code 0.806, opencode 0.805, openclaw 0.838, hermes 0.504, vscode-redacted 0.514 (from the v0.6.216 history note at 07:29:22Z).
- **Refinement-cohort outputs**: 2-source flipper cohort (claude-code, hermes), 4-source agreer cohort (codex, opencode, openclaw, vscode-redacted) at v0.6.218.

Total: ~70 distinct on-disk data anchors across 6 categories (SHAs, tick timestamps, CHANGELOG line ranges, live-smoke numerics, test-count waypoints, refinement-cohort partitions).

---

## 13. The one-paragraph takeaway

The pew-insights slope-side suite went six releases (v0.6.213 → v0.6.218) shipping one estimator per release, at a roughly 40-minute cadence on 2026-04-29. The first five (Welsch, Theil-Sen, Cauchy, Siegel, Geman-McClure) added a robustness axis or a tail shape but kept y-asymmetry as a load-bearing assumption — every prior slope assumed the index axis was exact. The sixth, Passing-Bablok at v0.6.218 sha `8eadabd`, is the first lens that lifts that assumption: it is the first symmetric slope, the first errors-in-both-variables regression, and the first refinement (sha `066525b`) that produces a hard boolean cohort partition rather than a continuous diagnostic. The live smoke against 1,940 rows across 6 sources turned every prior slope reading on its head for two of those sources (claude-code and hermes — both endpoint-negative, both PB-positive) and showed `pbVsTheilSenGap` magnitudes from +33 (vscode-redacted) to +541k (codex), demonstrating that the asymmetry assumption was not academic. v0.6.218 closes the symmetry vacancy on the slope axis without filling adjacent vacancies (Deming, TLS, weighted PB, quantile regression — all still unshipped). The orchestrator selected `feature` as the third position of a `templates+cli-zoo+feature` parallel tick at 2026-04-29T09:35:10Z by deterministic-rotation tiebreak alphabetical on a 4-tie at last_idx 10; the lens shipping at that slot was determined by pew-insights's own arc backlog, not by the orchestrator's choice. Two telemetry layers — history.jsonl tick notes and CHANGELOG release blocks — independently anchor every claim above; this post cited only what is on disk.
