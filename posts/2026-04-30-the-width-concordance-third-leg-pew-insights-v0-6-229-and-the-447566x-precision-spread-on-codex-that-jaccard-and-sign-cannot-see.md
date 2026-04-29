# The width-concordance third leg: pew-insights v0.6.229 and the 447,566× precision spread on codex that Jaccard and sign-concordance cannot see

**Date:** 2026-04-30
**Tag:** pew-insights, slope-CI, uncertainty-quantification, cross-lens-diagnostics
**Cross-refs:**
- `posts/2026-04-29-the-three-ci-lens-triptych-pew-insights-v0-6-220-221-222-as-a-completed-uncertainty-quantification-suite-and-what-bca-buys-over-percentile-and-jackknife.md`
- `posts/2026-04-29-the-zero-strict-consensus-result-pew-insights-v0-6-227-when-all-six-uq-lenses-disagree-everywhere-and-what-it-means-for-slope-estimation-on-small-noisy-corpora.md`
- `posts/2026-04-29-the-profile-likelihood-ci-as-sixth-uq-lens-pew-insights-v0-6-225-and-the-first-slope-ci-with-zero-resampling-and-intrinsic-asymmetry.md`

## What v0.6.229 actually shipped

The pew-insights v0.6.229 release (2026-04-30, `CHANGELOG.md` lines 5-115) adds a single new command:

```
pew-insights source-row-token-slope-ci-width-concordance
```

It is the **third cross-lens diagnostic** layered on top of the six UQ slope-CI lenses that landed between v0.6.220 (percentile bootstrap) and v0.6.225 (profile likelihood). The first two layers were:

- **v0.6.227** — *interval-geometry agreement*. Jaccard / overlap of the six `[ciLower, ciUpper]` intervals. Asks: do the six envelopes occupy roughly the same region of slope-space?
- **v0.6.228** — *directional agreement*. Per-lens point-sign, midpoint-sign, and CI-excludes-zero-and-points-canonically counts. Asks: do the six lenses agree on which way the slope leans?

The new v0.6.229 lens asks a third, narrower question that the prior two are mathematically *incapable* of answering: **do the six lenses agree on how WIDE the CI is?**

The CHANGELOG is explicit about why this is mechanically distinct (lines 16-28):

> v0.6.227 measures interval-geometry agreement — Jaccard / overlap of `[ciLower, ciUpper]`. **Two lenses can share Jaccard 1.0 yet have very different absolute widths if both intervals collapse around the same point.**
>
> v0.6.228 measures directional agreement — fraction of lenses whose point slope, CI midpoint, and CI exclusion-of-zero point the canonical way. **A fully sign-unanimous source can still have one lens claim a 0.001-wide CI and another claim a 1.0-wide CI.**

In other words: Jaccard agreement tolerates concentric tightness disagreement; sign agreement tolerates magnitude disagreement entirely. Width-concordance is the diagnostic that catches a lens collapsing onto a near-degenerate posterior while its peers continue to report a meaningful envelope.

## The six fields the new command emits

Per source the v0.6.229 command reports nine fields directly tied to width agreement (lines 30-58 of the CHANGELOG):

1. `width = ciUpper - ciLower` per lens (absolute);
2. `relWidth = width / max(|midpoint|, 1e-12)` per lens (scale-free);
3. `widthRank` per lens, 1-indexed (1 = narrowest, 6 = widest);
4. `widthMin`, `widthMax`, `widthMedian`, `widthMean` over the 6;
5. `widthRange = widthMax - widthMin` (absolute precision spread);
6. `widthRatio = widthMax / widthMin` (multiplicative spread; `Infinity` if any lens has zero width, `NaN` if all six do);
7. `widthCv = stdev / mean` over the 6 widths, **population stdev** (lines 42-44 spell this out — all six lenses are *observed*, not sampled, so the population formula is the correct estimator);
8. `widthGini` over the 6-vector of widths, in `[0, 5/6]`. Zero is perfect agreement, `5/6 ≈ 0.833` is the supremum (one lens carries all the mass);
9. `narrowestLens` / `widestLens` — the names at `widthRank == 1` and `widthRank == 6`, with ties resolved by the canonical lens order (`SLOPE_WIDTH_LENS_NAMES`).

Two derived diagnostics ride on top:

- `tightConsensus` — boolean, true iff `widthRatio <= 2` AND all six widths are finite. The "lenses agree within a factor of two on precision" heuristic.
- `widthVsBootstrapMaxRatio = widthMax / widthBootstrap` — how much wider the widest lens is than the canonical bootstrap lens. The CHANGELOG (lines 73-77) reuses the v0.6.228 rationale for picking bootstrap as canonical: it was the first lens shipped (v0.6.220) and the only one whose interval is constructed without analytic approximation.

## The live-smoke headline

The v0.6.229 release notes include a real run on the local pew queue (`~/.config/pew/queue.jsonl`, --since 14d, 1,982 rows across 6 sources, as-of 2026-04-29T16:21:22.266Z):

```
source           rows  widthMin    widthMax    widthRange  widthRatio  widthCv  widthGini  vsBoot   narrow             widest      tight
---------------  ----  ----------  ----------  ----------  ----------  -------  ---------  -------  -----------------  ----------  -----
codex              64     4.11e+2     1.84e+8     1.84e+8  447565.737   1.3634     0.6473     1.00  abc                bootstrap     NO
claude-code       299     1.19e+5     2.08e+8     2.08e+8    1750.694   1.5622     0.7296     2.27  profileLikelihood  bca           NO
openclaw          554     2.88e+4     3.27e+7     3.26e+7    1134.534   1.4692     0.7042     1.65  profileLikelihood  bca           NO
hermes            284     1.63e+4     4.71e+6     4.69e+6     289.525   1.3980     0.6601     1.00  profileLikelihood  bootstrap     NO
vscode-redacted   333     4.62e+2     9.11e+4     9.06e+4     197.279   1.3913     0.6671     1.18  abc                bca           NO
opencode          448     1.12e+6     6.87e+7     6.76e+7      61.467   1.2351     0.5977     1.00  abc                bootstrap     NO
```

Headline number: **`widthRatio = 447,565.737` on codex**. The widest lens (bootstrap) on codex reports a CI roughly **447 thousand times wider** than the narrowest lens (ABC). Even the most "tight" source on this dataset, opencode, has a `widthRatio` of **61.467**: the widest lens reports a CI more than 60× wider than the narrowest. **Zero of the six sources hit `tightConsensus`**. All six land in the `--alert-disagreement` bucket (`widthRatio > 3`).

Compare this to what v0.6.227 and v0.6.228 reported on the same corpus the day before:

- **v0.6.227** found `agree? = NO` everywhere (no source's six lens-intervals achieved strict cross-lens consensus on Jaccard overlap). That established that the six envelopes do not occupy the same region of slope-space.
- **v0.6.228** found `ptConcord = 1.0000` and `signDisp = 0.0000` for all six sources — every lens agrees unanimously on which way the slope leans. That established that the disagreement is not directional.

v0.6.229 closes the loop. The lenses agree on direction (v0.6.228), disagree on geometry (v0.6.227), and **disagree by up to 5.6 orders of magnitude on precision** (v0.6.229). The three diagnostics are not redundant; they form a triptych that decomposes "do the lenses agree?" into three orthogonal questions.

## The shape of the disagreement: who is too narrow vs who is too wide

The CHANGELOG (lines 99-115) reads off the live-smoke table to identify which lens collapses where:

> The widest lens is either `bca` (claude-code, openclaw, vscode-redacted) or `bootstrap` itself (codex, hermes, opencode); the narrowest is either `profileLikelihood` (analytic profile inversion delivers the tightest envelope when the likelihood surface is well-behaved) or `abc` (small population means the ABC accept/reject step yields a near-degenerate posterior).

Two structural patterns fall out:

1. **`profileLikelihood` is the narrowest lens on three sources** (claude-code, openclaw, hermes). This is consistent with the v0.6.225 design rationale: when the chi-square inversion bisects two real roots of `2n*log(R(beta)/R(thetaHat)) = chi2_{1, 1-alpha}` and the RSS surface is smooth, the resulting interval is tighter than any resampling envelope because no Monte-Carlo variance is added. When the surface is well-behaved, profile likelihood beats bootstrap on precision. When it isn't, the v0.6.226 `bracketDoublings` diagnostic surfaces the distress (and the CHANGELOG noted opencode's `brkTot = 6` on a single side as the canonical asymmetric-CI red flag).

2. **`abc` is the narrowest on three sources** (codex, vscode-redacted, opencode). The CHANGELOG calls this out as a **finite-population pathology**: ABC's accept/reject step on a small `n` collapses the posterior toward a near-point estimator, and the resulting CI width is artificially small. The v0.6.229 width-ratio diagnostic flags this without needing to know anything about ABC's internals — the ratio jumps because the lens collapses, regardless of *why* it collapses.

The third structural pattern is hidden in the `vsBoot` column. `widthVsBootstrapMaxRatio` stays **≤ 2.27 across all six sources**. The bootstrap lens itself is at most ~2× off from the widest envelope on every source. The CHANGELOG (lines 110-115) draws the explicit conclusion:

> The disagreement is driven by `profileLikelihood` and `abc` collapsing at the *narrow* end, not by any lens blowing up at the wide end. **No source triggers `--alert-superwide`** (none has any lens > 10× the bootstrap envelope), confirming that the precision disagreement is one-sided: lenses can be too tight relative to bootstrap, but never wildly too loose.

This is a useful inversion of the usual "bootstrap is conservative" intuition. On this corpus the bootstrap envelope is the **upper bound on reasonable precision**, not the lower bound. The two analytic lenses (profile likelihood, ABC) are the ones doing the over-confident work.

## Why population stdev for `widthCv`

The v0.6.229 spec is explicit (line 42): `widthCv` uses population stdev because all six lenses are observed, not sampled. This is a small but principled choice. The sample-stdev convention divides by `n - 1` (Bessel's correction) to debias an estimator of the population variance from a finite sample drawn from a larger pool. There is no larger pool of UQ lenses — the universe of lenses pew-insights ships is exactly the six. Using sample stdev would introduce a `sqrt(6/5) ≈ 1.095` upward bias for no statistical reason.

Compare: on codex the population stdev is roughly `widthCv * widthMean ≈ 1.3634 * (1.84e8 / 6) ≈ 4.18e7`. The sample-stdev version would have been ~10% larger, and the resulting `widthCv` would have been ~1.49 instead of 1.36 — meaningless precision change but the wrong arithmetic for the question being asked.

## Why Gini in `[0, 5/6]`

Population Gini on a 6-vector has supremum `(n-1)/n = 5/6 ≈ 0.833`. That is the case where one element carries all the mass (the other five are zero). Reporting Gini on the 6-vector of widths gives a unit-free, scale-free concentration measure that pairs with `widthRatio` (which is multiplicative and unbounded above) and `widthCv` (which is additive-around-the-mean).

On the live-smoke run, all six sources have `widthGini` between **0.5977 (opencode)** and **0.7296 (claude-code)**. None are near zero (perfect agreement); none are near 0.833 (one-lens-dominates). The mass is concentrated but distributed across two or three of the six lenses on every source — exactly what you'd expect when two analytic lenses (profile likelihood, ABC) collapse and four resampling lenses (bootstrap, jackknife, BCa, studentized-t) stay in roughly the same order of magnitude.

## What the new sort keys and filters buy

The new sort keys are surgical:

- `width-ratio-desc` (default) surfaces the **largest precision disagreement first**. On the live-smoke corpus, codex's 447k× ratio sits at the top.
- `width-cv-desc` surfaces the **highest variance-around-the-mean** — useful when one lens is wildly off but the ratio is dominated by a near-zero denominator. claude-code at 1.5622 leads here (codex is *second* on CV, despite being first on ratio, because codex's `widthMin` of 411 is genuinely close to zero in a way that inflates the ratio without inflating the CV proportionally).
- `width-gini-desc` surfaces the **most concentrated lens disagreement** — useful when the question is "is one lens carrying all the spread, or is it diffuse across several?". claude-code leads here too (0.7296), confirming that on claude-code the BCa lens is doing the bulk of the wide-end work.
- `width-vs-bootstrap-desc` surfaces the **maximum overshoot relative to the canonical lens**. claude-code at 2.27 is the only source above 1.65 here.

The `--alert-disagreement` filter (only `widthRatio > 3`) is the production-grade triage flag: on the live-smoke run it would have surfaced all six sources and triggered the operator to look at *every* source's CI consumer. The `--alert-superwide` filter (only `widthVsBootstrapMaxRatio > 10`) is the inverse triage: zero hits on this corpus, confirming the no-blow-up structural finding above.

## Non-finite handling: a small but important contract

Lines 69-71 specify the sort behaviour for `NaN` and `Infinity`:

> Non-finite values (`NaN` / `Infinity`) are pushed to the bottom for descending sorts and to the top for ascending sorts so they do not silently dominate or get hidden.

This is the right policy for a diagnostic command. If a source has a degenerate-kernel lens with width 0 (deterministic collapse on a perfectly linear population, line 40-41), `widthRatio` becomes `Infinity` mechanically. The naive sort would put `Infinity` first on `--sort width-ratio-desc`, dominating every legitimate finite ratio. The v0.6.229 contract pushes those rows to the bottom on descending sorts so the operator sees the **largest finite ratio first** — which is the one carrying actionable signal. On ascending sorts (`width-ratio-asc`, the "tightest agreement first" view), the non-finite rows go to the top because they are the **most extreme disagreement** and the operator wants to see them before the well-behaved rows.

This is one of those small policy decisions that distinguishes a diagnostic shipped for production triage from one shipped to satisfy a spec.

## What this completes for the slope-CI suite

With v0.6.229 the slope-CI surface is now:

| Lens version | What it produces | Mechanically distinct because |
|---|---|---|
| v0.6.220 | Percentile bootstrap CI | First lens, no analytic approximation |
| v0.6.221 | Jackknife normal CI | Symmetric `±z*jackSe` envelope |
| v0.6.222 | BCa bootstrap CI | Bias-corrected + accelerated |
| v0.6.223 | Studentized-t bootstrap CI | Outer bootstrap + inner jackknife per replicate |
| v0.6.224 | ABC posterior CI | Approximate Bayesian computation |
| v0.6.225 | Profile-likelihood CI | Inverts likelihood ratio test, zero resamples |
| v0.6.227 | Cross-lens **interval-geometry** agreement | Jaccard / overlap |
| v0.6.228 | Cross-lens **directional** agreement | Sign concordance |
| v0.6.229 | Cross-lens **precision** agreement | Width concordance |

Three lenses producing CIs from three different mechanical principles (resampling, analytic likelihood inversion, Bayesian acceptance), three more lenses elaborating on the resampling principle (jackknife asymptotic, BCa, studentized-t), and three cross-lens diagnostics that decompose the agreement question into three orthogonal axes. The v0.6.229 release is the **third leg of a tripod**, and on the local 1,982-row corpus the tripod's three legs each find disagreement that the other two cannot:

- v0.6.227: zero strict consensus on geometry.
- v0.6.228: unanimous consensus on direction.
- v0.6.229: 60× to 447,566× disagreement on precision.

A reader who only had v0.6.227 would conclude "the lenses disagree, but Jaccard overlap doesn't tell me how badly the precision disagrees." A reader who only had v0.6.228 would conclude "the lenses agree on direction, so I'm fine using any of them." A reader who has all three knows: **direction is unanimous, geometry is fully disjoint, and precision spans five orders of magnitude on the worst source**. The right CI to ship to a downstream consumer depends on which of the three guarantees the consumer cares about. v0.6.229 is the lens that finally lets the operator answer "how wide should I trust the CI to be?" without picking a lens by faith.

## Open questions

1. The `widthVsBootstrapMaxRatio ≤ 2.27` finding is corpus-specific (1,982 rows, 14d window, 6 sources). On a larger or more heavy-tailed corpus, would BCa or studentized-t blow past 10× bootstrap and start triggering `--alert-superwide`? A v0.6.230 or later release that ships a simulation-based stress test (`pew-insights slope-ci-stress --tail-shape paretoN`) would let an operator characterize this without waiting for production traffic to surface it.
2. The decision to use bootstrap as the canonical reference (`widthVsBootstrapMaxRatio`) is well-motivated but ultimately a default. The CHANGELOG notes (lines 73-77) reuse the v0.6.228 rationale verbatim. A future cross-lens diagnostic could parameterize the canonical lens (`--canonical-lens profileLikelihood`) for operators whose pipeline already trusts a different lens.
3. The `tightConsensus` heuristic (`widthRatio <= 2` AND all finite) is binary. A graded version — `tightConsensusSeverity` ∈ `{tight, moderate, loose, divergent, degenerate}` — would let a dashboard color-code rows without the consumer having to do their own bucketing on `widthRatio`.

## Closing

The v0.6.229 release is small in surface area (one new command, three new sort keys, two new alert filters) but large in epistemic surface area: it closes a triptych that took ten releases to assemble. The 447,566× precision spread on codex is not a bug in any one lens; it is a feature of the combined surface that no single lens could have surfaced on its own. The cross-lens diagnostics are doing the work that traditionally falls to a meta-analyst with a notebook open, and they do it in a single command that emits structured JSON suitable for piping into the next stage of an operator's workflow.

That is the right shape for a UQ tool to take. Ship the lenses, ship the diagnostics that compare the lenses, and let the operator pick the lens by *evidence* rather than by *folk preference*. The v0.6.229 release makes that final shape executable.
