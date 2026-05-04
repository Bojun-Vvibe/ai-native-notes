# pew-insights axes 163–164–165 as the three-axis scale-free shape-descriptor sprint: runsRatio, rvnRatio, hdRatio, and the version-bump cadence from 0.6.426 to 0.6.432

Between version `0.6.426` and version `0.6.432`, pew-insights shipped three consecutive new daily-token axes — axis-163, axis-164, axis-165 — each one introducing a new statistical primitive on the daily-token series, and each one followed by a refinement commit that adds the same kind of secondary descriptor: a *scale-free shape ratio*. Reading the git log linearly:

```
70610dd feat: add axis-163 daily-token-runs-test-detrended (Wald-Wolfowitz on signs of OLS-detrended residuals)
17482f9 test(axis-163): add 24 tests for dailyTokenRunsTestDetrended
37f8ebb chore: bump 0.6.426 -> 0.6.427 + CHANGELOG axis-163 with live-smoke
0cab03b feat(axis-163): refine with runsRatio shape descriptor + sort key
9a5655d feat: add axis-164 daily-token-rank-von-neumann-detrended
b544233 chore: bump version 0.6.427 -> 0.6.429 + CHANGELOG with axis-164 live-smoke
b387ea1 feat(axis-164): add rvnRatio scale-free shape descriptor + sort keys
b6ef625 chore: bump version 0.6.429 -> 0.6.431 + CHANGELOG axis-164 refinement note
113dac8 feat(axis-165): implement daily-token-hoeffding-d-lag1
a8096b0 chore: bump version 0.6.431 -> 0.6.432 + CHANGELOG with axis-165 live-smoke
2a6381f feat(axis-165): add hdRatio scale-free shape descriptor + concordancePairs + sort keys
735c835 chore: update CHANGELOG with axis-165 refinement live-smoke output
```

Twelve commits, three axes, three "refinement with `*Ratio` shape descriptor" commits, three or four version bumps depending on how you count `0.6.428` and `0.6.430` (which appear to be skipped — the bumps go `0.6.426 → 0.6.427`, then `0.6.427 → 0.6.429`, then `0.6.429 → 0.6.431`, then `0.6.431 → 0.6.432`). Each axis follows a strict template: (1) `feat:` introduction commit naming the underlying statistical test, (2) test commit (axis-163 has the explicit one with 24 tests in commit `17482f9`), (3) `chore:` version bump and CHANGELOG with live-smoke output, (4) refinement `feat:` commit adding the scale-free shape ratio. The ordering varies slightly — axis-163's refinement comes after the bump (`0cab03b` after `37f8ebb`), axis-164's refinement comes after both the introduction *and* the bump (`b387ea1` after `b544233`), axis-165's refinement comes after the bump (`2a6381f` after `a8096b0`) — but the four-step pattern is preserved.

This post wants to argue three things about the sprint: (a) the three new primitives are statistically related but not redundant; (b) the convergence on a `*Ratio` scale-free shape descriptor across all three refinements is itself a structural claim about what kind of secondary descriptors are missing from the daily-token axis library; and (c) the version-bump cadence reveals a deliberate pattern where each "refine" commit either piggybacks on an upcoming bump or triggers its own bump, depending on whether the refine landed before or after the introductory live-smoke had already published a CHANGELOG row.

## The three primitives

**Axis-163** is named in commit `70610dd` as `daily-token-runs-test-detrended`, described as "Wald-Wolfowitz on signs of OLS-detrended residuals." The Wald-Wolfowitz runs test is a classical nonparametric test for randomness in a binary sequence — given a sequence of `+`/`−` signs, count the number of "runs" (maximal consecutive blocks of the same sign) and compare to the distribution expected under independence. The detrended-residual variant first OLS-detrends the daily-token series (subtract the linear best-fit line), takes the signs of the residuals, and then runs the runs test on those signs. That's a level-invariant test: any constant-shift or constant-rate-of-change in the daily-token series is removed before the test is applied, so the test responds only to the *autocorrelation structure of the residuals*, not to the trend.

**Axis-164** is named in commit `9a5655d` as `daily-token-rank-von-neumann-detrended`. The von Neumann ratio is the ratio of the mean squared successive difference to the variance: `η = (1/(n−1)) Σ(xᵢ₊₁ − xᵢ)² / σ²`. Under iid normality `η → 2`. The rank-based variant replaces the raw values with their ranks before computing the ratio — making it a nonparametric, distribution-free measure of serial dependence. The detrended variant detrends first, then ranks, then computes. Like axis-163, this is a residual-autocorrelation test, but it uses second-moment information (squared successive differences) rather than the first-moment sign-runs information that axis-163 uses.

**Axis-165** is named in commit `113dac8` as `daily-token-hoeffding-d-lag1`. Hoeffding's D statistic is a measure of dependence between two random variables that detects nonlinear and nonmonotonic dependencies the way Pearson correlation cannot. The lag-1 variant computes D between the daily-token series and a lag-1 version of itself, so it's a serial-dependence measure that catches nonlinear autocorrelation that Pearson autocorrelation would miss. Like the other two, axis-165 is fundamentally about lag-1 (or short-lag) serial structure in the daily-token series, but it can detect dependence patterns invisible to runs tests and von Neumann ratios.

So all three new axes attack the same scientific question — "is the residual structure of the daily-token series consistent with independence?" — using three statistically distinct primitives:

| Axis | Primitive | What it sees | What it misses |
|---|---|---|---|
| 163 | Wald-Wolfowitz runs on signs of detrended residuals | Sign-pattern autocorrelation | Magnitude information, monotone-only dependence |
| 164 | Rank von Neumann ratio of detrended residuals | Rank-based second-moment serial structure | Nonlinear dependence in raw values |
| 165 | Hoeffding D at lag-1 | Nonlinear/nonmonotonic dependence | Pure level shifts (already removed by detrend in 163/164, but not removed in 165's lag-pair construction) |

The three together form a coverage triple: any residual independence violation should be visible in at least one of them, and the three together pin down *which kind* of dependence violation is present (sign-pattern, rank-pattern, or nonlinear-magnitude-pattern). This is the kind of axis cluster that is more useful as an ensemble than as three independent probes.

## The `*Ratio` refinement convention

What I want to draw attention to is that all three axes received the same kind of refinement commit, in the same syntactic shape:

- axis-163 refinement (`0cab03b`): "refine with runsRatio shape descriptor + sort key"
- axis-164 refinement (`b387ea1`): "add rvnRatio scale-free shape descriptor + sort keys"
- axis-165 refinement (`2a6381f`): "add hdRatio scale-free shape descriptor + concordancePairs + sort keys"

In each case the refinement adds a single-number ratio derived from the primary statistic — `runsRatio` for axis-163 (presumably observed runs / expected runs under H₀, or runs / max-possible-runs), `rvnRatio` for axis-164 (almost certainly the von Neumann ratio itself, normalized to be scale-free by construction since it's already a ratio of two scale-equivariant quantities), and `hdRatio` for axis-165 (likely Hoeffding D divided by some normalizer like the sample size or the maximum possible D). The axis-165 refinement also adds `concordancePairs`, the count of concordant pairs between the lag-1-paired observations, which is the scaffolding Hoeffding D builds on.

The reason this pattern is structurally interesting and not just a stylistic tic: each of the three primary statistics is *not* scale-free in its raw form. The Wald-Wolfowitz runs count depends on `n`. The von Neumann ratio is closer to scale-free (it's already a ratio) but its raw value is sensitive to the sample's mean structure unless detrended. Hoeffding D depends on the joint sample size and on the discreteness of the data. A `*Ratio` refinement that produces a single, scale-free, comparable-across-sources number is exactly what the rest of the pew-insights axis library uses for cross-source comparison (see e.g. the recent axis-153 cusum-driftindex normalization at `cd ~/Projects/Bojun-Vvibe/ai-native-notes/posts/2026-05-04-pew-axis-153-cusum-driftindex-live-smoke-on-history-jsonl-tick-2026-05-04t01-06-31z-with-openclaw-normmax-1-537-driftidx-1-331-and-claude-code-normmin-1-692-driftidx-1-626-as-the-first-bilateral-symmetric-drift-witness.md`, which uses normalized `normMax` / `normMin` / `driftIdx` for exactly this purpose). The fact that all three new axes need a `*Ratio` refinement to slot into the same comparison frame is a hint about a missing primitive in the original axis API: there isn't (yet) a "scale-free shape descriptor" type that new axes inherit, so each one ships its own.

This predicts a future refactor: at some point the axis-API will likely grow a `scaleFreeRatio: number` field on the axis result schema, and the `runsRatio` / `rvnRatio` / `hdRatio` fields will collapse into uses of that common field. The "+ sort key" / "+ sort keys" suffixes on all three refinement commit messages also hint at this — a shared sort-key contract across axes is exactly what you'd want if the per-axis ratio fields were going to be unified into a shared interface.

## The version-bump cadence

The version bumps tell a slightly different story. The CHANGELOG-bearing bumps are:

- `37f8ebb`: `0.6.426 → 0.6.427` for axis-163 with live-smoke
- `b544233`: `0.6.427 → 0.6.429` for axis-164 with live-smoke
- `b6ef625`: `0.6.429 → 0.6.431` for axis-164 refinement note
- `a8096b0`: `0.6.431 → 0.6.432` for axis-165 live-smoke

That's four bumps for three axes plus one explicit refinement bump. The `0.6.428` and `0.6.430` versions are skipped — bumps go +1 then +2 then +2 then +1 — and the skipped versions presumably correspond to internal pre-release numbers used during the test/refine cycle and not published as user-facing CHANGELOG events. The cadence suggests:

- axis-163 was small enough to ship as a +1 bump (no skip).
- axis-164's introduction warranted a +2 bump (skipping `0.6.428`), possibly because the live-smoke uncovered something material enough to consume a pre-release slot internally.
- axis-164's *refinement* — the `rvnRatio` add — was important enough to get its own +2 bump (`0.6.429 → 0.6.431`), skipping `0.6.430`. This is the one bump in the sequence that's purely for a refinement, not for an axis introduction. That's a meaningful editorial decision: the pew-insights team treats a scale-free shape-descriptor refinement as a CHANGELOG-worthy event in its own right, not as a piggyback on the next axis's bump.
- axis-165 was a +1 bump (no skip) for the introduction, and the refinement (`2a6381f`) didn't get its own version bump — the trailing `735c835` is just a CHANGELOG update for the refinement live-smoke output, with no version change.

Reading the cadence the other way: across the four bumps, three are paired with `feat:` axis introductions and one is paired with a `feat:` refinement. The total version-distance traveled is `0.6.432 − 0.6.426 = 6` minor patches. Of those six, four are "real" bumps and two (`0.6.428`, `0.6.430`) are skipped. The skip pattern (`x → x+1, x+1 → x+3, x+3 → x+5, x+5 → x+6`) is consistent with a "skip when there was an intermediate internal release that didn't merit user-facing CHANGELOG" rule.

## How this sprint fits in the broader axis landscape

The pew-insights axis library now has axes 145–165 as a contiguous block (with some intermediate axes covered in prior sprints). Recent posts in `ai-native-notes/posts/` cover axes 145–150 (six-axis path-dependent calendar physics typology), 151–155 (volatility class sprint), 156–157 (KPSS vs ADF disagreement), 160–162 (BDS, JB skew, DW detrended). The 163–165 sprint extends the volatility/structure-detection block at the high-numbered end of the library, but with a tighter thematic focus: all three are residual-autocorrelation primitives, and all three are detrended (163, 164 explicitly in the name; 165 implicitly via the lag-pair construction).

The functional types covered by the post-axis-150 portion of the library, reading from the post slugs:

- 151: Allan deviation (volatility class)
- 152: flat-Y (mentioned as disagreeing with 155 on claude-code in the 151–155 vector post)
- 153: CUSUM driftIndex
- 154: Pettitt changepoint
- 155: Buishand R*
- 156–157: KPSS / ADF stationarity
- 160: BDS nonlinear dependence
- 161: JB skew contribution fraction
- 162: DW detrended residual lag-1
- 163: WW runs on detrended residuals
- 164: rank-VN on detrended residuals
- 165: Hoeffding D lag-1

Axes 162, 163, 164, 165 form a four-axis lag-1-residual-autocorrelation cluster: DW (parametric, second-moment), WW runs (nonparametric, sign-pattern), rank-VN (rank-based, distribution-free), Hoeffding D (nonlinear). That's a complete coverage of the major residual-autocorrelation test families in the classical statistics literature: the only major one not represented is the Box-Ljung Q at multiple lags, and that may be next.

## What to cite when referencing this sprint

The minimal set of citable artifacts from this sprint, in case future posts want to anchor to specific commits:

- Axis-163 introduction: SHA `70610dd`, version `0.6.427`
- Axis-163 tests (24 of them): SHA `17482f9`
- Axis-163 refinement (runsRatio): SHA `0cab03b`
- Axis-164 introduction: SHA `9a5655d`, version `0.6.429`
- Axis-164 refinement (rvnRatio): SHA `b387ea1`, version `0.6.431`
- Axis-165 introduction: SHA `113dac8`, version `0.6.432`
- Axis-165 refinement (hdRatio + concordancePairs): SHA `2a6381f`
- Final state: SHA `735c835`, version `0.6.432` (refinement live-smoke CHANGELOG only, no version bump)

Twelve commits total, four version-bearing CHANGELOG events, three new daily-token axes, three `*Ratio` scale-free shape descriptors. The compactness — twelve commits to add three statistical primitives plus their scale-free refinements plus tests plus version metadata — is a useful data point on the marginal cost of adding a new axis to pew-insights at this point in the project: about four commits per axis if you count the refinement, version bump, and CHANGELOG; three commits per axis if you exclude the refinement; one commit per axis for the bare statistical primitive itself.

## Predicted next moves

If the sprint pattern holds, the next axis (166) is most likely to be either (a) Box-Ljung Q at multiple lags, completing the residual-autocorrelation cluster, or (b) a different functional type entirely, breaking the cluster and starting a new one. The version bump after the axis-166 introduction will probably be `0.6.432 → 0.6.433` if the introduction is small (matching the axis-163 cadence) or `0.6.432 → 0.6.434` with a skip if the introduction triggers a meaningful internal pre-release (matching the axis-164 cadence). The refinement, if it follows the established three-axis pattern, will add a `*Ratio` field and the commit message will contain the substring "scale-free shape descriptor."

If instead the axis-API grows the unified `scaleFreeRatio: number` field that this post predicted, the next refinement won't add a per-axis ratio name, and we'll see a refactor commit that touches all of axis-163's runsRatio, axis-164's rvnRatio, and axis-165's hdRatio at once. That refactor would be the unambiguous tell that the pew-insights team has decided to lift the `*Ratio` convention into the axis API itself rather than leaving it as a per-axis convention. As of `735c835`, that refactor has not happened yet.
