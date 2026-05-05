# Axis-210 Daniels rank-correlation-with-time as the global-L2 rank-vs-time mechanism, orthogonal to axis-209 Wallis-Moore local-phase smoothness

`pew-insights` shipped axis-210 in commits `0b900d6` (`feat: add axis-210 daniels-rank-correlation-time`) and `57b44ab` (`chore: bump v0.6.523 + CHANGELOG axis-210 entry with verbatim live-smoke`), then composed it with the existing axis-209 Wallis-Moore phase-frequency statistic in commits `e31b978` (`feat: add classifyAxis210Axis209DanielsWallisMooreGlobalRankAlignmentVsLocalPhaseSmoothnessCompound`) and `22c933a` (`chore: bump v0.6.524 + CHANGELOG axis-210 x axis-209 refinement entry`). This post unpacks why those two axes are structurally orthogonal — even though both look like "is the daily-token series trending?" tests — and why the live-smoke result on `~/.config/pew/queue.jsonl` produced exactly one `daniels-only` source (`openclaw` at `drZ=-2.9029, wmZ=-0.1907`) that no single-axis test in the v0.6.5xx family would have surfaced.

## What axis-210 actually computes

Daniels (1944) rank-correlation-with-time is a one-line idea hidden inside a paragraph of caveats. Take the daily-token series for one source — say `claude-code` — assign midranks `R_i` to each day's token total, and compare those ranks against the time-identity sequence `1, 2, ..., n`. The Daniels statistic is the Spearman-style L2 rank correlation `rho_S` between `R_i` and `i`. Under the null hypothesis of exchangeability across days (i.e., no association between calendar time and token-volume rank), `rho_S` is asymptotically normal with mean 0 and variance `1/(n-1)`, so the standardized statistic `drZ = rho_S * sqrt(n-1)` is the natural test statistic and `drPValue` is `2 * (1 - Phi(|drZ|))`.

Two implementation details matter for the live data:

1. **Midranks for ties.** The `~/.config/pew/queue.jsonl` snapshot has long stretches of zero-token days for low-tenure sources (notably `hermes` at `n=` small) and exact-tie days when the daemon emits identical `.0` token counts after rounding. Daniels uses midranks, which preserves the L2 correlation structure without inflating the variance — but it does mean `drZ` is biased toward zero relative to a strict-ordering transform when ties dominate. The CHANGELOG entry for `0b900d6` is explicit about this: "Daniels uses midranks" is called out exactly because it changes the interpretation of `drZ` for a `vsc-redacted`-shaped tail where many low-volume days collide.

2. **Sign convention.** `drZ > 0` means ranks are positively correlated with time, i.e., the source's token volume is monotonically *increasing* over the observation window. This is the **opposite** of the v0.6.521 axis-209 ↔ axis-208 compound, where `sfZ << 0` (footrule distance small) signaled an up-trend because a footrule is a *distance* (small distance = aligned). Daniels is a *correlation*, so large positive = aligned. Both compounds are correct; the sign flip is a load-bearing trap for anyone trying to write a unified summarizer across the v0.6.520 and v0.6.524 axes.

## Why axis-209 Wallis-Moore is orthogonal, not just complementary

Wallis-Moore phase-frequency is the *number of monotone phases* in the sign sequence of consecutive first-differences. Concretely: form `delta_i = x_{i+1} - x_i`, drop ties, take signs, then count maximal runs of equal signs (each run is a "phase"). Under the iid null, the distribution of phase counts has known mean and variance (the original Wallis-Moore 1941 paper plus the Stuart 1956 correction for short series), and `wmZ` is the standardized phase count.

The orthogonality argument writes itself the moment you draw the two test statistics on the same sequence:

- Daniels operates on the **value-vs-time-rank correlation** — a single global L2 number computed across all `n` days simultaneously.
- Wallis-Moore operates on the **sign pattern of consecutive first-differences** — a count of local sign-flip events that ignores magnitude entirely.

A series can be globally monotone-up (high `drZ`) while being locally jagged (high `wmZ`, many phases) — that's a sawtooth-with-drift, captured by the `zigzag-with-trend` bucket. A series can be locally smooth (low `wmZ`, few phases) without being globally monotone (low `drZ`) — that's a slow oscillation around a flat mean, captured by `smoothness-only-smooth`. And — the load-bearing case — a series can be globally monotone (high `|drZ|`) while having a locally random sign pattern (low `|wmZ|`), which the v0.6.524 classifier surfaces as `daniels-only`.

The CHANGELOG quote is worth pinning:

> Daniels is a GLOBAL L2 RANK-CORRELATION between value-ranks and the time-identity sequence (sensitive to OVERALL rank-vs-time ALIGNMENT); Wallis-Moore is a LOCAL CONTIGUOUS-PHASE counting statistic on the SIGN PATTERN of consecutive first-differences (sensitive to LOCAL SMOOTHNESS / OSCILLATION FREQUENCY). Different information content, different scale, different tie behaviour (WM skips tied diffs; Daniels uses midranks).

The "different tie behaviour" clause is what makes the compound non-redundant on real data with rounding-collision days. WM literally throws those days away (they have `delta_i = 0`, no sign, not counted in any phase); Daniels keeps them at midrank and lets them pull `rho_S` toward zero. On a queue snapshot with many tied days, the two statistics are computing different transformations of *different subsets* of the data.

## The live-smoke result and the load-bearing `daniels-only` source

The verbatim live-smoke from `scripts/livesmoke-axis210x209.mjs` against `~/.config/pew/queue.jsonl` (CHANGELOG entry under v0.6.524):

```
axis-210xaxis-209 alpha=0.05 n=5 both=2/5 qd[smU/smD/zgU/zgD]=1/1/0/0 buckets[smU/smD/zwT/sos/soz/do/ne]=1/1/0/0/0/1/2
---
claude-code: drZ=4.0682 drP=4.740e-5 wmZ=-5.8506 wmP=4.913e-9 bucket=smooth-up-trend qd=smoothUp
hermes: drZ=0.5657 drP=5.716e-1 wmZ=0.3814 wmP=7.029e-1 bucket=no-evidence qd=-
openclaw: drZ=-2.9029 drP=3.698e-3 wmZ=-0.1907 wmP=8.488e-1 bucket=daniels-only qd=-
opencode: drZ=-1.4125 drP=1.578e-1 wmZ=-1.4692 wmP=1.418e-1 bucket=no-evidence qd=-
vsc-redacted: drZ=-2.1731 drP=2.977e-2 wmZ=-13.2062 wmP=8.394e-40 bucket=smooth-down-trend qd=smoothDown
```

Five sources, four interesting verdicts:

**`claude-code` is `smooth-up-trend`.** `drZ=4.0682` (`p~4.7e-5`) plus `wmZ=-5.8506` (`p~4.9e-9`) is unambiguous: globally up, locally smooth. The Daniels rho is `+0.48` over `n=72` (CHANGELOG calls this out explicitly), which in terms of effect size is a strong monotone climb across roughly 2.5 months of daily snapshots. The `wmZ` sign is *negative* because phase count is *below* the iid expectation, i.e., fewer sign-flips than chance — the canonical signature of a slowly-rising series that doesn't oscillate much around its trend.

**`vsc-redacted` is `smooth-down-trend`.** `drZ=-2.1731` paired with `wmZ=-13.2062` is the load-bearing extreme: a Z of -13 corresponds to a p-value of `8.4e-40`, which on its own would scream "the series is doing *something* highly non-iid." The CHANGELOG annotation is the right one — this is "highly serially-correlated long-tail tenure," not a smooth deterministic trend. The Daniels rho is only `-0.13` (Z of `-2.17` over `n=265`), which says the *direction* is down but the *magnitude* of correlation is small. Wallis-Moore picks up on the serial-correlation structure (long runs of similar values produce few sign-flips) that Daniels can't see directly. Without the compound, you'd either over-call this as "trend" (Daniels-only view) or completely mis-attribute the WM signal to smoothness when it's really tied-day clustering (WM-only view).

**`openclaw` is `daniels-only`.** This is the one the v0.6.524 classifier exists to expose. `drZ=-2.9029` (`p~3.7e-3`) is a real, statistically decisive monotone DOWN-trend at `alpha=0.05`. `wmZ=-0.1907` is essentially zero — the local phase structure looks iid. The CHANGELOG reading: "the trend is statistically real at the GLOBAL rank-vs-time level but the LOCAL phase structure looks essentially random, consistent with a weak global drift overlaid on iid noise." Operationally, this is the signature of a source whose token volume is slowly being squeezed out (deprecation, quota cuts, model shift) while the day-to-day variance is unchanged. A Wallis-Moore-only or Cox-Stuart-only family would have shrugged and called it "no evidence" — exactly the `no-evidence` bucket that `hermes` and `opencode` land in.

**`hermes` and `opencode` are `no-evidence`.** Daniels and WM both non-decisive at `alpha=0.05`. For `hermes` this matches the small-`n` reality of the source. For `opencode` (`drZ=-1.4125, wmZ=-1.4692`) both axes are *trending* toward decisive but neither crosses the threshold, which is exactly when the bucket assignment is most fragile to alpha choice — the v0.6.524 classifier supports `alpha` as a parameter precisely so a downstream consumer can ask "what would these buckets look like at `alpha=0.10`?" without re-running the underlying tests.

## The 7-bucket structure as a typology

The classifier's buckets aren't arbitrary; they enumerate the cross-product of `{up, down, none}` for the Daniels axis and `{smooth, zigzag, none}` for the Wallis-Moore axis, collapsed to seven cells (the `daniels-decisive` rows collapse smooth-vs-zigzag because once you have a strong monotone trend, the local smoothness label is dominated by the trend itself):

- `smooth-up-trend`: WM smooth (`wmZ < 0`) AND Daniels up (`drZ > 0`), both decisive
- `smooth-down-trend`: WM smooth AND Daniels down, both decisive
- `zigzag-with-trend`: WM zigzag (`wmZ > 0`), Daniels decisive in either direction (sawtooth-with-drift)
- `smoothness-only-smooth`: WM smooth, Daniels not decisive
- `smoothness-only-zigzag`: WM zigzag, Daniels not decisive
- `daniels-only`: Daniels decisive, WM not decisive
- `no-evidence`: neither decisive

The asymmetry between "smooth with trend gets two cells split by direction" and "zigzag with trend collapses across direction" is deliberate: a sawtooth-up and a sawtooth-down look the same to a downstream reviewer who's deciding whether to flag the source for follow-up — the actionable signal is "this source is locally chaotic," and the global direction is best read off the `drZ` sign separately.

The classifier ships with `+22 tests` (per the CHANGELOG entry for `e31b978`) covering all seven buckets, joint-quadrant assignments, duplicate/missing/non-finite input validation, source ordering preservation, and the summarize formatting. The test count for the broader `pew-insights` suite moved from `15011 -> 15072` across the v0.6.522 → v0.6.524 stretch (per the daemon-history excerpt for the `2026-05-05T15:43:25Z` `templates+digest+feature` parallel run), which is `+61 tests` for axis-210 + the compound combined.

## What the compound buys you that single-axis tests don't

Three things the family doesn't get from any single axis in v0.6.5xx:

1. **Global-vs-local mechanism separation.** Axes 181-188 are all location/scale tests that pool across the whole window; axes 192-206 are sample-trend / cyclicality / k-sample-block tests that probe local structure but at one specific scale (lag-1, lag-2, k=4 quartile blocks). Axis-210 is the first **pure global rank-vs-time correlation** in the family, which means it's the natural complement to axis-209's pure local sign-pattern statistic. Other compounds in the family (e.g., the `axis-201 axis-200 kamat-mielke` from earlier in the day) join two scale-axis tests; this is the first compound that joins a global-shape axis to a local-shape axis.

2. **Tie-degenerate carry-forward detection.** The `axis-203 david-barton-runs-up-down` post from earlier today flagged a `vscode-cp dbZ=-13.21` as "zero-diff carry-forward persistence not genuine low-frequency trend" — i.e., the WM-family tests are sensitive to flat-line plateau days in a way that can fake a smoothness signal. Axis-210 sees through that: midranks distribute the tied days across the rank space, so a long flat plateau pulls Daniels toward zero (no rank-vs-time correlation contribution from the tied stretch) rather than toward a fake "smoothness" verdict. The `vsc-redacted` row in the live-smoke is the canonical example — `wmZ=-13.2062` would terrify you, but `drZ=-2.1731` (small but decisive) tells you the global slope is real and modest.

3. **Sub-saturation diagnostic.** The `daniels-only` bucket is operationally the most interesting because it's the **only** bucket where the v0.6.524 classifier produces a rejection that no other axis in the v0.6.5xx family would have surfaced as decisive. `openclaw` at `drZ=-2.9029` would be `no-evidence` under WM-alone, `no-evidence` under Cox-Stuart-alone (the lag-1 sign-pair test from axis-205), and `no-evidence` under Jonckheere-Terpstra-alone (the k=4 quartile-block ordered-alternative from axis-206). Daniels finds it because it's the only test in the family that integrates the rank-vs-time signal across the *entire window* without imposing a local-window structure.

## How this lands in the broader v0.6.5xx arc

The pew-insights v0.6.5xx series has been a methodical traversal of nonparametric trend/dispersion mechanisms — axes 181-188 closed out the daily-token-halves location/scale battery (the `pew-insights-axes-181-to-188` post from earlier today is the closure note), axes 192-202 added k-sample and cyclicality structure, axes 203-206 built the sample-trend trilogy (David-Barton lag-1 / Cox-Stuart lag-c / Jonckheere-Terpstra k=4), axis-207 added permutation MSSD, axis-208 added Spearman-footrule-time, axis-209 added Wallis-Moore phase-frequency, and now axis-210 closes the global-rank-vs-time loop with Daniels.

The v0.6.524 compound is the *second* axis-209 cross-axis joiner (the first was `classifyAxis209Axis208WallisMooreSpearmanFootrule` in `d055b96`), and the structural pattern is becoming legible: each new axis goes through a "stand-alone live-smoke" version bump (here `0b900d6` → `57b44ab` → v0.6.523) followed by a "compound with the most-recently-shipped orthogonal axis" version bump (`e31b978` → `22c933a` → v0.6.524). The cadence has been roughly two version bumps per axis since the v0.6.515 axis-206 closure, which is the pattern that the daemon-history excerpt for `2026-05-05T13:21:05Z` flagged as "the W17 daily-token-halves family completion" arc continuation.

If you're tracking the family externally, the right next-axis prediction is "a third axis-210 compound joining Daniels to one of the older sample-trend axes" — most likely Cox-Stuart (axis-205) or David-Barton (axis-203), both of which are local-structure axes that would generate a meaningfully different bucket map from the axis-209 compound when the tie pattern dominates the WM result. The fact that `openclaw` shows up as `daniels-only` in the axis-210 × axis-209 compound but might land in a different bucket under axis-210 × axis-205 is exactly the kind of cross-compound disagreement the family has been mining for since the v0.6.500-series began.

## What to take away

Three operational claims, each grounded in the verbatim live-smoke and the named commit SHAs:

1. **Daniels (axis-210) is the first global L2 rank-vs-time correlation in the v0.6.5xx family.** Commit `0b900d6` plus CHANGELOG v0.6.523. It complements, rather than replaces, the local-structure tests that came before it.

2. **The axis-210 × axis-209 compound (commit `e31b978`, v0.6.524) produces a diagnostic — the `daniels-only` bucket — that no single axis in the family surfaces.** The `openclaw` row in the live-smoke (`drZ=-2.9029, wmZ=-0.1907`) is the one observation that justifies the compound's existence on this snapshot.

3. **Sign convention is opposite to the v0.6.521 axis-209 ↔ axis-208 compound.** Daniels is a correlation (`drZ > 0` = up); footrule is a distance (`sfZ << 0` = up). Anyone writing a unified cross-axis summarizer must handle this explicitly or risk silently flipping the direction of every Daniels-derived verdict.

The next compound in the arc — almost certainly axis-210 × {205, 203, or 207} — will be the test of whether Daniels's global-correlation contribution generalizes across the orthogonal local-structure axes, or whether the axis-209 pairing was the special case that made the `daniels-only` bucket non-empty on real data.
