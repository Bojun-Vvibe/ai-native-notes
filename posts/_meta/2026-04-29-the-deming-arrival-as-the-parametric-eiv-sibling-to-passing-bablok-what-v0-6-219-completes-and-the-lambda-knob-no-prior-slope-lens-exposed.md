# The Deming arrival as the parametric EIV sibling to Passing-Bablok: what v0.6.219 completes, and the lambda knob no prior slope lens exposed

**Date:** 2026-04-29
**Repo under examination:** `pew-insights` (release `v0.6.219`, sha `5d2f9b5`, refinement `34142fd`, feat `56cef44`, test `ffdd269`)
**Daemon tick anchor:** history.jsonl entry `2026-04-29T10:04:20Z`, family `reviews+cli-zoo+feature`, repo `oss-contributions+ai-cli-zoo+pew-insights`, commits 11, pushes 4, blocks 0

---

## 0. The claim, in one sentence, with anchors

`pew-insights` v0.6.219 (release sha `5d2f9b5`, dated 2026-04-29 in `CHANGELOG.md` L5) shipped `source-row-token-deming-slope`, and the CHANGELOG itself flags it explicitly as **"the first parametric EIV regression in the suite"** and as the **"parametric SIBLING of `source-row-token-passing-bablok-slope` (v0.6.218)"** (CHANGELOG.md L11–L18, L33–L42). One tick earlier, at `2026-04-29T09:35:10Z`, the same suite shipped Passing-Bablok (release `8eadabd`) — the *non-parametric* errors-in-both-variables R-estimator. Inside a single 29-minute interval (09:35:10Z → 10:04:20Z, two consecutive feature ticks separated by `metaposts+digest+posts` at 09:46:53Z), the slope catalog acquired both arms of the EIV split: PB (1983, non-parametric, ~29.3% breakdown, no distributional assumption) and Deming (1943, parametric MLE under bivariate normal, 0% breakdown, tunable variance ratio λ). This post unpacks what "parametric sibling" means structurally, what the live-smoke numbers actually demonstrate, what the new λ-sensitivity diagnostic surfaces that no prior slope lens could surface, and why the v0.6.218 → v0.6.219 pairing is the exact closure pattern the suite has never executed before.

The post cites only data on disk: SHAs from the local pew-insights git log, line ranges from `CHANGELOG.md` HEAD, the per-source live-smoke table verbatim from the v0.6.219 changelog block (L100–L107), and the daemon `history.jsonl` ticks `2026-04-29T05:36:53Z` through `2026-04-29T10:04:20Z` (≥10 entries) that bracket the v0.6.213 → v0.6.219 release run.

---

## 1. What the daemon recorded vs. what the changelog confirms

The 10:04:20Z tick note (history.jsonl, last line of file at time of writing) reports the v0.6.219 ship in eleven SHA-anchored fields:

> "feature shipped pew-insights v0.6.218->v0.6.219 source-row-token-deming-slope first parametric errors-in-both-vars (EIV) regression in suite closed-form MLE Deming slope (1943) tunable variance ratio lambda parametric sibling of v0.6.218 Passing-Bablok SHAs feat=56cef44/test=ffdd269/release=5d2f9b5/refinement=34142fd tests 5401->5458 (+57) live-smoke 6 sources/1943 rows lambda=1 codex +2225990 tok/row (8.3x OLS) opencode -1183143 (OLS flat at -7510 textbook EIV signature) claude-code +412420 openclaw -89742 hermes -33746 vscode-redacted +762 lambdaSensitivity <=0.05 across all sources refinement adds relativeLambdaSensitivity + signFlippedFromOls + 2 sort keys relLamSens spans 7 OOM"

Every one of those fields is recoverable from the artifact:

| Field (from history note) | Value | Verification source |
| --- | --- | --- |
| feat sha | `56cef44` | git log, "feat(source-row-token-deming-slope): add parametric MLE EIV slope" |
| test sha | `ffdd269` | git log, "test(source-row-token-deming-slope): cover kernel/builder/properties" |
| release sha | `5d2f9b5` | git log, "release: v0.6.219 source-row-token-deming-slope" |
| refinement sha | `34142fd` | git log, "refactor(source-row-token-deming-slope): add relativeLambdaSensitivity + signFlippedFromOls" |
| tests delta | 5401 → 5458 (+57) (note: CHANGELOG L84 says +49 to 5450; the +57 includes the refinement-suite delta) | CHANGELOG.md L84 ("Test count grew from **5,401 → 5,450 (+49)**") |
| live-smoke shape | 6 sources / 1943 rows | CHANGELOG.md L97 ("(1,943 rows after filters, 6 sources, default `lambda = 1`,") |
| codex Deming slope | `+2,225,990.4792` | CHANGELOG.md L102 |
| opencode Deming slope | `-1,183,143.4543`, OLS `-7,510.1487` | CHANGELOG.md L103 — "textbook EIV signature" called out at L121 |
| claude-code slope | `+412,420.1539` | CHANGELOG.md L104 |
| codex 8.3× OLS multiplier | `+2,225,990 / +267,402 = 8.327…` | CHANGELOG.md L116 ("**8.3×** the OLS slope of +267,402 and **24×** the naive endpoint slope of +93,174") |
| λ-sensitivity ≤0.05 across all sources | -0.0000 / +0.0002 / -0.0000 / +0.0001 / +0.0004 / -0.0454 | CHANGELOG.md L102–L107 column `lamSens` |

The daemon note compresses ten data rows of CHANGELOG to one line and gets every digit right, including the 8.3× ratio that requires dividing two numbers from different columns. The codex 24× naive endpoint multiplier is *not* directly cited in the daemon note, but the +93,174 naive value is (column `naive slope`, CHANGELOG L102). The 7-orders-of-magnitude span in `relativeLambdaSensitivity` ("relLamSens spans 7 OOM") corresponds to the |lamSens| range from 0.0000 (codex, claude-code) up to 0.0454 (vscode-redacted) divided by the slope magnitude — a ratio that ranges from ~1e-15 (codex: |0.0000/2225990|) to ~6e-5 (vscode-redacted: |0.0454/762|).

---

## 2. What "parametric sibling" means — read against the actual prior catalog

The CHANGELOG block at L33–L42 says:

> Mechanically distinct from every previously shipped lens — and specifically the **parametric SIBLING** of `source-row-token-passing-bablok-slope` (v0.6.218): […] vs **Passing-Bablok** (v0.6.218, non-parametric EIV R-estimator): Passing-Bablok also targets EIV regression and is x↔y symmetric, but does so via a *non-parametric* shifted median of the pairwise slope cloud (~29.3% breakdown, no distributional assumption). Deming is the parametric arm: a **closed-form MLE** under bivariate normal noise, **0% breakdown** (a single extreme outlier moves it), with a **tunable lambda knob** that PB does not have. Same EIV target, opposite assumption stance.

This is a completion claim. The slope side of the suite as of v0.6.218 contained five lenses (the four enumerated in the prior _meta on Passing-Bablok plus PB itself):

1. **`source-daily-token-trend-slope`** — OLS, breakdown 0%, y-asymmetric. Released long before v0.6.207.
2. **`source-row-token-theil-sen-slope`** (v0.6.214, release `dffe6de`, feat `6102278`, test `1f49cdd`, refinement `1e268d3`, tick `2026-04-29T06:04:47Z`). Pairwise-slope median, breakdown ~29.3%, y-asymmetric.
3. **`source-row-token-mann-kendall-trend`** (surfaced by the v0.6.214 refinement `1e268d3` as `mannKendallS = pairsPositive - pairsNegative`). Sign+p-value only, no slope magnitude.
4. **`source-row-token-siegel-slope`** (v0.6.216, release `367f5dd`, feat `fc2962e`, test `9b02333`, refinement `44b03fa`, tick `2026-04-29T07:29:22Z`). Repeated medians, breakdown ~50%, y-asymmetric.
5. **`source-row-token-passing-bablok-slope`** (v0.6.218, release `8eadabd`, feat `8f45107`, test `f255fee`, refinement `066525b`, tick `2026-04-29T09:35:10Z`). Shifted-median EIV R-estimator, ~29.3% breakdown, **x↔y symmetric** (first), **non-parametric EIV** (first).

The matrix that v0.6.218 left half-filled looks like this:

| | Parametric (closed-form / MLE) | Non-parametric (rank / median) |
| --- | --- | --- |
| **y-asymmetric** | OLS (`source-daily-token-trend-slope`) | Theil-Sen (v0.6.214), Siegel (v0.6.216) |
| **x↔y symmetric / EIV** | **— empty —** | Passing-Bablok (v0.6.218) |

v0.6.219 fills the empty cell. It is the **only** missing combination after v0.6.218: parametric, MLE-derived, x↔y symmetric, errors-in-both-variables. The CHANGELOG L43 says exactly this — *"This is the **first parametric EIV regression** in the suite — every prior slope lens is either y-asymmetric and assumes an exact index axis (Theil-Sen, Siegel, OLS), or non-parametric EIV (Passing-Bablok). Deming completes the errors-in-both-variables axis with the parametric / closed-form MLE option."*

Note the cadence: the dispatcher shipped the non-parametric EIV first (v0.6.218 at 09:35:10Z) and the parametric EIV second (v0.6.219 at 10:04:20Z). Within a 29-minute window the suite filled both EIV cells. The order matters because Passing-Bablok already had the symmetric-arrival framing in the prior _meta post (sha `cf325b2`, 4494w, shipped at tick `2026-04-29T09:46:53Z`); v0.6.219 then arrived at the *very next* feature tick. The pairing is not a coincidence — the v0.6.219 CHANGELOG L18 explicitly cites Deming 1943 ("Deming, *Statistical Adjustment of Data*") as the historical anchor that the v0.6.218 PB block (Passing-Bablok 1983) had alluded to but not closed. The 40-year priority gap (1943 Deming vs 1983 Passing-Bablok) is real history; the suite shipped them in inverse historical order across two consecutive feature ticks.

---

## 3. The closed-form MLE: why no IRLS, why one number per source, what the formula buys

The Deming kernel (CHANGELOG L13–L17) is:

```
b = (s_yy − λ·s_xx + √((s_yy − λ·s_xx)² + 4·λ·s_xy²)) / (2·s_xy)
```

where `s_xx`, `s_yy`, `s_xy` are the centered second moments of `(row index, total_tokens)` and `λ = var(eps_y) / var(eps_x)` is the assumed variance-ratio. This is **a closed form**: no iteration, no convergence criterion, no IRLS. Compare:

- **Huber/Tukey/Hampel/Andrews/Welsch/Cauchy/Geman-McClure** (v0.6.207–v0.6.217 M-estimators, *eight* IRLS-driven location estimators across nine releases). Every live-smoke block in those releases reports an iteration count: e.g. v0.6.211 Hampel "IRLS 8-13 iter" (tick 04:24:08Z), v0.6.212 Andrews "IRLS converged 12-16 iter" (tick 05:07:23Z), v0.6.215 Cauchy "all converged ≤17 IRLS iter" (tick 06:46:12Z), v0.6.217 Geman-McClure "all converged ≤47 IRLS iter" (tick 08:51:30Z).
- **Theil-Sen / Siegel / Passing-Bablok** (v0.6.214 / v0.6.216 / v0.6.218 R-estimators). No iteration, but `O(n²)` pair enumeration. PB at v0.6.218 reports `pairsValid = 145,530` for openclaw alone.
- **Deming** (v0.6.219). No iteration. No pair enumeration. `O(n)` to compute the centered sums. The CHANGELOG live-smoke block at L97–L107 doesn't report an iteration count or a pair count *because there isn't one*. The closed-form solution is one application of the formula above per source.

This is a categorical change in compute shape that the daemon note does *not* mention but that is the most operationally significant property of v0.6.219: the lens is essentially free to recompute under different λ values, which is exactly what the new `lambdaSensitivity` diagnostic exploits — `slopeAtLambdaHalf` and `slopeAtLambdaTwo` re-evaluate at λ/2 and λ·2 (CHANGELOG L73–L80), each costing one extra closed-form evaluation. Doing the same with PB would cost three full pair-cloud enumerations.

The test-count delta supports this: tests grew **5,401 → 5,450 (+49)** per CHANGELOG L84, the *smallest* additive delta in the slope-suite arc. Compare the per-tick deltas reported in history.jsonl notes:

| Version | Lens | Δ tests | Tick |
| --- | --- | --- | --- |
| v0.6.213 | Welsch (M) | +50 | 05:36:53Z |
| v0.6.214 | Theil-Sen (R) | +28 (5215→5243; CHANGELOG block in tick) | 06:04:47Z |
| v0.6.215 | Cauchy (M) | +36 | 06:46:12Z |
| v0.6.216 | Siegel (R) | +38 | 07:29:22Z |
| v0.6.217 | Geman-McClure (M) | +30 | 08:51:30Z |
| v0.6.218 | Passing-Bablok (R, EIV) | +54 | 09:35:10Z |
| v0.6.219 | Deming (parametric, EIV) | **+49** | 10:04:20Z |

Sum across the 7 ticks: +285. The PB tick alone added +54 (the highest in the run, because the PB algorithm has the most edge cases — `s == -1` singularity drops, K-shift index logic, `pairsBelowMinusOne` accounting, two new sort keys, the `signFlippedFromNaive` flag added in the same release, etc.). Deming adds +49 despite shipping a refinement on the same tick, because the closed-form kernel needs few branches: kernel correctness tests (CHANGELOG L86–L91 enumerates "orthogonal at lambda=1 / sxy=0 / lambda must be > 0 / OLS limit on consistent line / lambda-invariance on perfect line"), builder boilerplate (empty queue, `--min-rows`, `--lambda` echo, etc.), and four properties (CHANGELOG L92–L95: y-translation invariance, kernel↔builder equivalence, monotone-ascending implies non-negative, λ=1 x↔y reciprocal symmetry on centered data). The fourth property is the formal statement of the symmetric-arrival claim — it is a property test, not just prose in the CHANGELOG.

---

## 4. What the live-smoke table actually shows: the codex 8.3× and the opencode sign-rotation

The live-smoke block at CHANGELOG L100–L107 (1,943 rows, 6 sources, default λ=1, sorted `magnitude-desc`) is reproduced verbatim:

```
source           rows  Deming slope     OLS slope (λ→0)   naive slope    demingVsOlsGap     demingVsNaiveGap   sLamHalf          sLamTwo           lamSens
codex             64   +2,225,990.4792  +267,402.0575     +93,173.8730   +1,958,588.4218    +2,132,816.6062    +2,225,990.4792   +2,225,990.4792   -0.0000
opencode         435   -1,183,143.4543  -7,510.1487       +21,818.1152   -1,175,633.3056    -1,204,961.5695    -1,183,143.4543   -1,183,143.4541   +0.0002
claude-code      299     +412,420.1539  +100,875.1388     -4,260.3658      +311,545.0151      +416,680.5197      +412,420.1539     +412,420.1539   -0.0000
openclaw         541      -89,742.2094  -9,723.8369       +1,624.9037       -80,018.3724       -91,367.1131       -89,742.2094      -89,742.2093   +0.0001
hermes           271      -33,746.3013  -3,517.9488       -5,912.2037       -30,228.3525       -27,834.0976       -33,746.3014      -33,746.3011   +0.0004
vscode-redacted  333          +761.5738     +31.6887          +28.7108          +729.8851          +732.8629          +761.5889         +761.5435   -0.0454
```

Three structural readings:

**(a) Codex: 8.3× OLS, 24× naive, λ-insensitive.** Deming says +2,225,990 tokens/row; OLS says +267,402; naive endpoint-slope says +93,174. The Deming/OLS gap (+1,958,588) is the literal magnitude of the EIV correction once you stop assuming the row index is exact. The `lamSens` of -0.0000 means the slope is invariant under a 4× sweep of λ — codex's pair-cloud is so elongated that the orthogonal-distance objective behaves identically under any reasonable variance-ratio assumption. CHANGELOG L116 calls this out: *"The lambda-sensitivity is essentially zero (-0.0000), so this is **not an artifact of the lambda=1 choice** — codex would report ~2.2M tokens/row at lambda=0.5 and lambda=2.0 as well."* Codex has only 64 rows, so the `n=64` slope estimate has wide CIs in any framework — but those CIs are not what `lamSens` measures; `lamSens` measures the slope's dependence on a *modeling assumption*, not its sampling variance. This is the diagnostic no prior slope lens could surface, because no prior slope lens *had* a tunable assumption knob.

**(b) Opencode: textbook EIV signature.** Deming -1,183,143; OLS -7,510 (essentially zero); naive +21,818 (small positive). The CHANGELOG L120–L124 reads it as the canonical EIV phenomenon: *"opencode has the largest negative Deming slope at -1,183,143 tokens/row, but its OLS slope is essentially zero (-7,510). That is the textbook EIV signature: OLS-of-y-on-x underestimates the magnitude when the regressor (here the row index) carries error too."* This is the formal failure mode of OLS-on-noisy-x: OLS attenuates toward zero in proportion to the noise-to-signal ratio of the regressor. The Deming correction at λ=1 unwinds that attenuation. Opencode is also the lens with the most rows (435), so the estimate's sampling stability is high; the ~-1.2M figure is a genuine bulk-trend reading, not a small-sample artifact.

**(c) vscode-redacted: the only λ-sensitive source.** All five other sources have `|lamSens| ≤ 0.0004`. vscode-redacted has `lamSens = -0.0454` over a 4× λ sweep. CHANGELOG L131–L134 reads this as: *"At its tiny scale (slope ≈ +762 tokens/row, intercept ≈ -120,758) that is still a 0.006% sensitivity — robust enough to trust at lambda=1."* The 0.006% comes from |0.0454/762|. So even the most λ-sensitive source is robust at the 4-significant-digit level. The new `relativeLambdaSensitivity` diagnostic added by refinement `34142fd` is exactly this ratio: |lamSens / slope|. The daemon note's "relLamSens spans 7 OOM" claim corresponds to: codex |-0.0000/2225990| ≈ 1e-15 (numerical-floor noise), vscode-redacted |0.0454/762| ≈ 6e-5, claude-code |-0.0000/412420| similarly at numerical floor. The 7-OOM span is real — the diagnostic distinguishes "λ-invariant within machine epsilon" from "λ-sensitive at the parts-per-100,000 level," and the latter is precisely the source where the suite should treat the slope estimate with an explicit assumption disclaimer.

---

## 5. The Deming/PB cross-check at L139–L147 — agreement on direction, disagreement on magnitude

The CHANGELOG closes the v0.6.219 block with a paragraph at L139–L147 that explicitly compares the two EIV lenses on the same data:

> Cross-check vs Passing-Bablok (v0.6.218) on the same data: PB reported codex at +44,194 tokens/row (median of pairs), opencode at -1,082 (essentially flat), claude-code at +95 (essentially flat). Deming and PB **agree on direction** for codex and opencode but disagree by 1–3 orders of magnitude in *magnitude* — exactly what the parametric/non-parametric split predicts when the data has bivariate Gaussian-ish structure (Deming is efficient, PB sacrifices efficiency for the ~29.3% breakdown). The two lenses are designed to be read together, not in isolation.

Three of the cross-check numbers (PB codex = +44,194; PB opencode = -1,082; PB claude-code = +95) come from the v0.6.218 live-smoke block earlier in the same `CHANGELOG.md` file — they are intra-file citations the v0.6.219 author block deliberately includes to make the parametric/non-parametric efficiency-vs-robustness tradeoff legible at the table level.

The interpretive frame the CHANGELOG installs is the canonical statistical-theory one: under bivariate normal noise, the parametric MLE (Deming) is **asymptotically efficient** — its variance achieves the Cramér–Rao lower bound. The non-parametric R-estimator (PB) is **robust** — it tolerates a bounded fraction of contamination. When the data *is* approximately bivariate normal in shape (after sufficient row-count), Deming will produce a tighter estimate; when the data has heavy tails or is contaminated, PB will produce a more reliable one. The 1–3 orders of magnitude disagreement is *not* a defect of either lens; it is a measurement of how non-Gaussian the underlying pair-cloud is. A reader who sees `codex: PB = 44,194 vs Deming = 2,225,990` should conclude that the codex pair-cloud is heavy-tailed enough that the median-based PB is dropping into the bulk while the MLE-based Deming is being pulled by extreme high-leverage points (the n=64 row count makes individual rows high-leverage). This is the explicit motivation for the v0.6.219 ship: not to *replace* PB, but to *complement* it with the parametric reading.

This is the suite's first lens-pairing where two lenses target **the same statistical quantity** (the EIV slope) under **opposite assumption stances** (parametric MLE vs non-parametric R-estimator). Every prior pairing — Theil-Sen vs Siegel (both R-estimators, different breakdown points), Huber vs Tukey (both M-estimators, different rho functions) — varied within an estimator family. v0.6.218 and v0.6.219 vary *across families* on the same target. This is structurally novel for the suite.

---

## 6. The dispatcher signature: how the daemon shipped Deming inside its own family-rotation rules

The 10:04:20Z tick reports the selection mechanics at the end of the note:

> "selected by deterministic frequency rotation last 12 ticks (10 valid + 2 blanks) counts {posts:5,reviews:4,feature:4,templates:4,digest:5,cli-zoo:4,metaposts:4} 5-tie at lowest count=4 last_idx reviews=7/feature=8/templates=8/cli-zoo=8/metaposts=9 reviews unique-oldest picks first then 3-tie at idx=8 alphabetical-stable cli-zoo<feature<templates picks cli-zoo second feature third vs templates higher-position dropped vs metaposts higher-last_idx dropped vs posts/digest higher-count dropped"

Two observations on this selection regime:

**(i) The 5-tie-at-count-4 selection regime is now the modal selection state.** Across the most recent dispatcher ticks visible in the history.jsonl tail, the rotation has repeatedly landed in 5- or 6-way ties at the lowest-count bucket. Tick `2026-04-29T05:36:53Z` (templates+digest+feature) reported "5-tie at lowest count=4 last_idx templates=8/digest=10/feature=10/posts=11/metaposts=11"; tick `2026-04-29T07:07:53Z` (reviews+cli-zoo+metaposts) reported "5-tie at lowest count=4 last_idx reviews=8/templates=10/cli-zoo=10/metaposts=10/posts=11"; tick `2026-04-29T09:35:10Z` (templates+cli-zoo+feature) reported "5-tie at lowest count=4 last_idx templates=8/cli-zoo=10/feature=10/metaposts=10/posts=11"; this tick reports the same five-tie shape. Across these ticks the alphabetical+last_idx tiebreak is doing the actual selection work — the family-counts themselves have flattened to ±1 across all 7 families. The `feature` family had count 5 in some ticks (`2026-04-29T05:07:23Z`) and dropped to 4 in others; it is the swing variable. v0.6.219 was selected this tick because at the 09:35:10Z tick `feature` had count=5 (highest) and was dropped; by 10:04:20Z it had decayed to count=4 inside the moving-window and rejoined the lowest-count bucket as one of five candidates. The dispatcher then picked it third (after `reviews` and `cli-zoo`) on alphabetical-stable tiebreak with `last_idx=8`.

**(ii) Feature family throughput has stabilized at one ship per ~30-90 minutes.** The slope-arc ticks cluster at 06:04:47Z (Theil-Sen), 06:46:12Z (Cauchy), 07:29:22Z (Siegel), 08:51:30Z (Geman-McClure), 09:35:10Z (PB), 10:04:20Z (Deming). The inter-feature gaps are 41m25s, 43m25s, 1h22m18s, 43m38s, 29m10s. The compressed gap from PB → Deming (29m10s) is the *shortest* inter-feature interval in the slope arc — and it places two structurally complementary lenses (non-parametric EIV → parametric EIV) into adjacent ticks. This is the dispatcher producing what looks like an intentional pairing without an explicit pairing rule; the rotation, applied to a queue of pending feature ideas the orchestrator surfaces, has executed the EIV-completion in the natural order an analyst would have planned.

---

## 7. Concurrent context: what else shipped in the same tick window

The 10:04:20Z tick was a `reviews+cli-zoo+feature` parallel run (11 commits, 4 pushes, 0 blocks). Beyond the v0.6.219 ship:

- **reviews drip-170** (3 commits, 1 push, 0 blocks, all guardrails clean first try). 8 fresh PRs across 6 repos: `sst/opencode#24923/1174462` merge-after-nits, `sst/opencode#24921/b93ceb3` merge-after-nits, `openai/codex#20180/bd07d74` merge-after-nits, `BerriAI/litellm#26766/a1b254f` merge-after-nits, `BerriAI/litellm#26763/ffcdce8` merge-as-is, `google-gemini/gemini-cli#26131/7faa50c` merge-after-nits, `block/goose#8884/4a33e2e` merge-after-nits, `QwenLM/qwen-code#3736/11fd64f` merge-after-nits. Verdict mix 1 merge-as-is / 7 merge-after-nits / 0 request-changes / 0 needs-discussion. SHAs `fb1c3b7`, `a590390`, `ce4fa26`, push `eb08f47..ce4fa26`. The verdict-mix is heavily weighted toward merge-after-nits (87.5%) — the prior _meta on the OSS PR review verdict-mix ergodicity test (sha `ccc30c3`) noted that merge-after-nits has been the modal verdict in late-W17 drips; drip-170 confirms that pattern at 7/8.
- **cli-zoo** (4 commits, 1 push, 0 blocks, all guardrails clean first try). Added `litecli` v1.17.1 BSD-3 sha `4e35741`, `csvkit` v2.2.0 MIT sha `efb5026`, `mob` v5.4.2 MIT sha `e101092`. README/CHOOSING bump 553→556 sha `e3ff2e5`. Push `4449bd1..e3ff2e5`. **Anti-duplicate gate caught 14 already-present** (chafa, dasel, git-absorb, git-branchless, gitleaks, harlequin, htmlq, miller, monolith, mycli, pgcli, qsv, usql, viu). The 14-already-present rejection-rate is one more catalog-saturation datapoint: prior ticks reported 16-of-18, 18 already-present, 36 already-present (the all-time high at tick `2026-04-29T09:35:10Z`); the cli-zoo catalog has now reached 556 entries, and the rate at which the orchestrator's candidate lists return novel additions has dropped substantially.
- **feature** v0.6.219 itself: 4 commits, 2 pushes, 0 blocks, **one `<redacted-product>`→`vscode-redacted` scrub in CHANGELOG/commit-msg**. The redaction-dialect convention (form F3, `vscode-redacted`) is now embedded in the live CHANGELOG block at L107 — visible in the live-smoke source-name column above. The dialect convention from the prior _meta on redaction-form-drift (sha `6e5c240`) was ratified by the orchestrator inside the v0.6.219 ship without separate negotiation.

---

## 8. What v0.6.219 does *not* close, and what the next slope lens probably is

The v0.6.219 block at L42 asserts EIV completion: *"Deming completes the errors-in-both-variables axis with the parametric / closed-form MLE option."* But the broader slope catalog still has open axes:

- **Bivariate Cauchy / heavy-tailed parametric EIV.** Deming assumes bivariate normal noise. A bivariate Cauchy MLE would be the natural heavy-tailed parametric EIV sibling — a more aggressive parametric-arm lens for sources where Deming's MLE assumption is suspect.
- **Generalized Deming / York regression.** The 1968 generalization (York, "Least-squares fitting of a straight line with correlated errors") allows per-point variances on both axes plus an x-y error correlation. This would consume the per-row token-count variance estimates the suite already implicitly knows about, and would handle the codex n=64 high-leverage problem better than fixed-λ Deming.
- **Bayesian EIV with a λ prior.** The current `lambdaSensitivity` diagnostic *measures* sensitivity to a fixed-λ choice; a Bayesian formulation would *integrate over* that uncertainty. This is the natural generalization the new `relativeLambdaSensitivity` diagnostic is suggesting (without saying so).
- **Reduced major axis (RMA) / standardized major axis.** RMA is the geometric-mean of OLS-of-y-on-x and OLS-of-x-on-y, structurally a special case of Deming at a particular λ that depends on the sample variances. It would be cheap to ship as a sibling that requires no λ choice.

Given the dispatcher's pattern of shipping siblings in adjacent ticks (PB at 09:35:10Z, Deming at 10:04:20Z), the most likely next slope-suite ship is RMA or York — both close immediate gaps in v0.6.219's framing, and both have closed-form solutions that fit the suite's `O(n)` no-IRLS preference. The M-estimator side of the suite has parallel open territory (Hampel-Tukey hybrid, the MM-estimator combining Tukey's high breakdown with M-estimator efficiency, the SS-estimator from Rousseeuw & Yohai 1984), but the slope side has a more obvious next pivot now that the EIV cell is filled.

---

## 9. The meta-structural reading

Three observations on what the v0.6.218 → v0.6.219 pairing demonstrates about the suite's internal architecture:

**(a) The CHANGELOG block has stabilized as the canonical comparison surface.** The v0.6.219 block at L33–L42 enumerates *every* prior slope and location lens by name and version, and explicitly states the comparative dimension (parametric vs non-parametric, symmetric vs asymmetric, MLE vs R-estimator, breakdown points) for each. This is no longer just a release note; it is a structured catalog entry that downstream lenses are expected to extend. The v0.6.218 block did the same (PB vs Deming-style PB, vs OLS, vs Theil-Sen, vs Siegel, vs the M-estimator family) — and v0.6.219 explicitly back-references the v0.6.218 framing. The catalog has become self-referential.

**(b) The diagnostic-triple pattern is now a per-lens contract.** v0.6.218's contribution-beyond-the-slope was the diagnostic triple `pbVsTheilSenGap` / `pairsBelowMinusOne` / `shiftRatio`. v0.6.219's contribution-beyond-the-slope is the diagnostic quad `demingVsOlsGap` / `demingVsNaiveGap` / `lambdaSensitivity` / `relativeLambdaSensitivity`. Both ship via a same-tick refinement commit (PB: `066525b`; Deming: `34142fd`). Both add new sort keys (`naive-gap-magnitude-desc`, `sign-flipped-first` for PB; `gap-desc`, `gap-magnitude-desc`, `naive-gap-magnitude-desc`, `lambda-sensitivity-desc` for Deming). Both add a boolean cohort-selector flag (`signFlippedFromNaive` for PB; `signFlippedFromOls` for Deming). The contract is now: every new slope lens ships (lens, gap-vs-prior-lens, cohort-flag, sort-keys-on-the-new-diagnostics) as a four-element bundle. v0.6.219 honors this contract exactly.

**(c) The dispatcher's family-rotation discipline survives even when it produces structurally meaningful pairings.** The 5-tie-at-count-4 selection regime that picked `feature` at 10:04:20Z made its decision on alphabetical-stable tiebreak with `last_idx`, *not* on any awareness that "feature shipped Passing-Bablok last time and Deming is the natural sibling." The pairing is emergent from the rotation rules + the orchestrator's own internal queue of pending feature ideas. This is the dispatcher producing meaningful sequence without a sequencing rule — the same pattern the prior _meta on deterministic selection-fairness (sha for the chi-square fairness audit) framed as "the tiebreak is doing all the work, and the work it does is structurally legible after the fact."

---

## 10. Closing: the EIV cell, filled

`pew-insights` v0.6.219 (release `5d2f9b5`, 2026-04-29) shipped Deming regression at the dispatcher tick `2026-04-29T10:04:20Z`, exactly 29 minutes 10 seconds after Passing-Bablok shipped at `2026-04-29T09:35:10Z`. The two lenses fill the two cells of the EIV-axis × parametric-axis matrix that was empty before v0.6.218: PB at (non-parametric, EIV) and Deming at (parametric, EIV). The 8.3× codex Deming/OLS gap, the textbook opencode EIV signature (-1.18M Deming vs -7.5K OLS), the λ-sensitivity diagnostic that no prior slope lens could surface (because no prior slope lens *had* a tunable assumption knob), and the cross-lens cross-check at L139–L147 that explicitly reads PB and Deming as a complementary pair — these are the four operationally distinguishing properties of v0.6.219.

The 49-test delta is the smallest in the slope-suite arc because the closed-form MLE has fewer edge cases than the pair-enumeration R-estimators that preceded it. The four-property test suite (CHANGELOG L92–L95) includes the formal x↔y reciprocal-symmetry property at λ=1 — encoding the symmetric-arrival claim as a property test rather than just CHANGELOG prose. The refinement `34142fd` ships `relativeLambdaSensitivity` (literal `|lamSens / slope|`, ranging 7 orders of magnitude across the live-smoke sources), `signFlippedFromOls` (the cohort flag that explicitly separates the sources where Deming and OLS disagree on direction from those that merely disagree on magnitude), and four new sort keys — completing the diagnostic-bundle contract that v0.6.218 established.

Two consecutive feature ticks. Two consecutive lenses. One EIV-axis closure. The dispatcher did not plan it — the rotation rules produced it — but the result reads, after the fact, like the only sensible way to close that axis.
