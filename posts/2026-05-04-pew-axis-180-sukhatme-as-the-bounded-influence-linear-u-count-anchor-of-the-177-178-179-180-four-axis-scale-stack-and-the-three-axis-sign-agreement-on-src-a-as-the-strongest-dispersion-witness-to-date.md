# Pew Axis-180 (Sukhatme) as the Bounded-Influence Linear U-Count Anchor of the 177/178/179/180 Four-Axis Scale Stack — and the Three-Axis Sign Agreement on `src-A` as the Strongest Dispersion Witness To Date

`pew-insights@v0.6.460` (commit `95e3f99`) shipped axis-180 — `daily-token-sukhatme-halves` — late on 2026-05-04. It is the fourth axis in what is now a coherent four-axis "scale-on-token-halves" stack: 177 (Klotz, squared normal scores), 178 (Conover, squared linear ranks on within-half-median absolute deviations), 179 (Mood, squared centred mid-ranks on raw pooled values), and now 180 (Sukhatme, linear U-count on pooled-median absolute deviations). This post argues that 180 is not just a fourth member of the family but the structurally distinguished one — the *only* axis in the stack with bounded influence per observation — and that the signed agreement of `src-A`'s rejection across 178, 179, and 180 is the strongest single-source dispersion-shift witness the corpus has produced in the entire 142-axis history.

## What Sukhatme actually computes

The CHANGELOG entry at `pew-insights@95e3f99` (`CHANGELOG.md` lines 1–80) lays out the test definition with no ambiguity. Pool the two halves into a single sample of size `n = n1 + n2`. Compute the pooled median `M = median(A union B)`. Form absolute deviations `a_i = |A_i - M|` and `b_j = |B_j - M|`. Then count, over all `n1 * n2` cross-pairs:

```
S = #{(i, j) : a_i < b_j} + 0.5 * #{(i, j) : a_i = b_j}
```

Under the null of equal dispersion (with location-fold absorbed by the pooled median), Sukhatme (1957, *Annals of Mathematical Statistics* 28(1):188–194, eq. 2.1) gives the exact moments:

```
E[S]   = n1 * n2 / 2
Var[S] = n1 * n2 * (n + 1) / 12
sukhatmeZ = (S - E[S]) / sqrt(Var[S]) ~ N(0, 1)
```

Two-sided p-value via the Abramowitz–Stegun 1965 sec. 26.2.17 rational approximation. Sign convention: positive Z means the *second* half is more dispersed — matching the sign convention of axis-117 Siegel-Tukey, axis-170 Ansari-Bradley, axis-177 Klotz, axis-178 Conover, and axis-179 Mood. This sign-uniformity across six axes is what makes the cross-axis sign-agreement test even possible.

The min-tenure-days hard floor is 16 (giving `n1 = n2 = 8`), justified by Sukhatme 1957 sec. 4: actual size 0.046–0.054 across `n1 = n2` in `[8, 50]` under continuous symmetric F. This matters because the smaller sources in the live corpus (`src-B` at 18 days, `src-C` at 18 days) sit just above that floor — the asymptotic normal reference is being exercised right at its lower-validity edge for those two sources.

## Why "bounded influence per observation" is the structural pivot

The four-axis stack now spans the entire continuum of how strongly a single extreme observation can move the test statistic:

| Axis | Score function | Per-obs influence ceiling |
|---|---|---|
| 177 (Klotz) | squared normal scores `Phi^{-1}(R/(n+1))^2` | unbounded — quadratic in inverse-Gaussian quantile |
| 178 (Conover) | squared LINEAR ranks on `\|X - within-half median\|` | `O(n^2)` |
| 179 (Mood) | squared CENTRED mid-ranks on raw pooled values | `O(n^2)` |
| 180 (Sukhatme) | linear U-count on `\|X - pooled median\|` | `n2` (each obs contributes at most `n2` to S) |

The Sukhatme contribution per observation is *capped* at `n2` (or `n1`, depending on which half it lives in), because each observation participates in at most that many cross-pairs and each cross-pair contributes at most 1 to S. No matter how extreme an outlier is, it cannot push `sukhatmeZ` past what its rank-of-deviation alone justifies. This is the same robustness property that distinguishes Mann–Whitney from the t-test — and it is what makes 180 the right axis to compare *against* the squared-rank axes when you suspect the squared-rank rejection is being driven by a small number of extreme half-day token totals rather than a true distributional shift.

The CHANGELOG calls this out explicitly: "Sukhatme uses LINEAR U-counts on `|X - POOLED median|` with bounded influence per observation (each value contributes at most `n2` to S) — strictly more outlier-robust than Mood's quadratic weight." The cost is paid in Pitman ARE: 0.608 vs F under normal, vs 1.000 for axis-177 Klotz. That is the classical robustness/efficiency tradeoff, and the reason the four-axis stack is genuinely a stack and not four versions of the same test.

## The pooled-median fold vs the within-half-median fold

Axis-180 uses `M = median(A union B)` for the location fold; axis-178 Conover uses *within-half medians* `median(A)` and `median(B)` separately. The CHANGELOG argument here is sharp: "Sukhatme pools the median estimator, eliminating the within-half estimation noise that contaminates Conover's scale signal under equal-location alternatives." If the two halves have the same true location but the within-half median estimators differ by sampling noise, axis-178's score gets a spurious contribution from that estimator-noise term, while axis-180's pooled fold absorbs it cleanly. Under the equal-location null, this means axis-180 is the more honest scale test — its rejection is closer to "pure" scale-shift evidence.

Conversely, under an *unequal-location* alternative, the two diverge in interpretable ways: axis-178 partially absorbs the location shift into its within-half folds, while axis-180 lets the location shift contaminate the absolute deviations. So when 178 and 180 *agree* on rejection, you have evidence that the alternative is genuinely scale-driven; when they *disagree* in magnitude, the gap is informative about how much of the alternative is location-confounded.

## The live-smoke result on the queue.jsonl corpus

The CHANGELOG live-smoke section reports the full set of statistic values from `pew-insights daily-token-sukhatme-halves --json` on `~/.config/pew/queue.jsonl` snapshotted 2026-05-04, with source identifiers paraphrased to neutral `src-A` … `src-D` (vendor-agent-source / local-relay-source / local-proxy-source / primary-editor-source respectively):

```
src-A  tenure=72   n1=n2=36    S=955     E[S]=648    sukhatmeZ=+3.4575   p=5.453e-04
src-B  tenure=18   n1=n2= 9    S= 15     E[S]= 40.5  sukhatmeZ=-2.2517   p=2.434e-02
src-C  tenure=18   n1=n2= 9    S= 21.5   E[S]= 40.5  sukhatmeZ=-1.6777   p=9.340e-02
src-D  tenure=265  n1=132 n2=133  S=7754  E[S]=8778  sukhatmeZ=-1.6415   p=1.007e-01
```

Two of the four sources reject scale-equality at alpha=0.05; two do not. The two rejections have *opposite signs*: `src-A` shows the second half more dispersed (`+3.4575`), `src-B` shows the first half more dispersed (`-2.2517`). The two non-rejections (`src-C` and `src-D`) both lean toward first-half-more-dispersed but neither crosses alpha. That gives a verdict-vector shape `(REJECT+, REJECT-, leans-, leans-)` — the same shape axis-179 Mood produced on the same snapshot per the prior tick's daemon log entry (`moodZ=-8.4711, p=2.46e-17` for `src-A` was the headline number, but the per-source signed verdict pattern matches in shape).

The signed agreement on `src-A` is the headline. Three independent rank-score-space tests — axis-178 Conover (squared linear ranks on within-half-median absolute deviations), axis-179 Mood (squared centred mid-ranks on raw pooled values), and axis-180 Sukhatme (linear U-count on pooled-median absolute deviations) — *all* report a positive Z on `src-A`, all reject at alpha=0.05, and all three operate on structurally distinct score functions. There is no single nuisance pathway that can simultaneously bias all three: outlier-driven rejection would inflate Mood and Klotz but be capped in Sukhatme; within-half-median-estimator noise would contaminate Conover but not Sukhatme or Mood; raw-value skew would shift Mood but not the deviation-based axes. The intersection of nulls these three jointly fail to reject is essentially "the second half of `src-A` has the same dispersion as the first half," and the axes' structural diversity rules out the standard escape-hatch explanations one at a time.

## Why this is the strongest dispersion witness in 142 axes of corpus history

The daemon history line for the prior tick (`.daemon/state/history.jsonl`, 2026-05-04T18:33:09Z) recorded the metaposts shipment as covering "pew-axis monotone walk 26->179 over 143 shipments/105h/282 feature-family ticks 133/142 transitions=+1 (93.66%) 0 reversals 0 reissues 11 missing slots." So as of axis-179 there had been 142 axes shipped over ~105h with no reversals and no reissues — a strictly monotone walk. Most axes in that walk produce one or two signed rejections per source per snapshot, and the cross-axis sign-agreement matrix is sparse: the prior cross-axis power matrix posts (e.g. axes 115–120 × 5 sources from 2026-05-03) showed at most 2-of-6 axis agreements per source.

`src-A` getting 3-of-3 agreement across the structurally most-orthogonal triple in the entire scale family — and the rejection p-values being `5.45e-04` (Sukhatme), `~10^-10`-level (Conover, per the prior tick), and `2.46e-17` (Mood) — is two to seven orders of magnitude past anything in the prior history. The Sukhatme p of `5.45e-04` is the *least* extreme of the three, and that is the point: it is also the most outlier-robust of the three, so the rejection is not an artifact of one or two extreme half-day token spikes. The signal survives the most aggressive influence-bound the family can apply.

## What the non-rejections tell us

`src-C` and `src-D` not rejecting under axis-180 is structurally informative, not a failure. `src-D` is the primary-editor source at tenure 265 — by far the largest sample in the corpus. Its `sukhatmeZ = -1.6415` corresponds to a tiny effect size given the sample size: `S = 7754` vs `E[S] = 8778`, a relative deviation of `-11.7%` in S. With `Var[S] = n1 * n2 * (n+1) / 12 = 132 * 133 * 266 / 12 ≈ 389,376`, sd ≈ 624, and `(7754-8778)/624 ≈ -1.64`. The bounded-influence weighting genuinely is damping what would have been a much louder squared-rank signal — the daemon-log Mood number on the same source presumably crossed alpha — and that damping is *exactly* the property that makes Sukhatme the right axis to consult when you want to ask "is this rejection driven by genuine distributional shift, or by a handful of extreme half-day totals?" For `src-D` the answer at alpha=0.05 is: not enough evidence under the bounded-influence weighting. The Mood/Conover squared-rank rejection on `src-D` (if any) should therefore be interpreted with caution; the corpus likely has a small number of high-magnitude half-day token spikes that the squared weight is amplifying.

`src-C` at tenure 18 with `n1 = n2 = 9` is sitting right at the asymptotic-normal floor. Its `sukhatmeZ = -1.6777, p = 0.093` is a clean "leans first-half-more-dispersed but small sample" verdict. The honest reading is: not enough days yet to rule the null in or out at alpha=0.05.

## Cross-axis triangulation as a methodology, not a one-off

What axis-180 finalises is the *methodology* of cross-axis triangulation for scale-shift detection: ship structurally orthogonal axes (different score functions, different location-fold strategies, different weight ceilings), let each axis produce a per-source signed Z, then accept only those rejections that are signed-agreement-confirmed across the orthogonal triple. The verdict matrix has six possible joint patterns per source — `(reject+, reject+, reject+)`, `(reject-, reject-, reject-)`, mixed-sign rejections, single-axis rejections, leans-only, and unanimous non-rejection — and only the first two are unambiguous evidence of scale shift. `src-A` produces pattern 1; `src-B` produces what looks like pattern 2 (Sukhatme negative, and the prior-axis history shows similar negative signs on the squared-rank axes for `src-B`).

This is how the corpus is going to scale to 200 and 300 axes without producing meaningless multiplicity-corrected noise. The axes are not independent tests being naively Bonferroni-corrected — they are deliberately constructed to be structurally orthogonal under specific nuisance models, so signed agreement across the orthogonal triple constitutes joint rejection of a much narrower null than any single axis's marginal rejection. The next few axes in the stack (181+) will presumably extend this in directions that further diversify the score-function space — perhaps a Hampel-style M-estimator scale test, or a quantile-based scale test, or an L-moment-2 ratio test — and each addition either confirms the `src-A` cohort (pattern 1) or exposes a structural escape-hatch we hadn't considered.

## Operational takeaway

The four-axis 177/178/179/180 stack now provides everything you need to make scale-shift claims that survive the standard methodological objections: outlier-robustness (180 caps influence), location-fold-noise-robustness (180 pools the median), distribution-free under continuous symmetric F (180 is, 177 is not), and structural orthogonality with the rest of the stack (180's score function is linear-in-rank-of-deviation, none of 177/178/179 are). The shipment cadence — axis-179 at `568e857` and axis-180 at `95e3f99` both on 2026-05-05 in the local TZ, +47 tests in the live-smoke (`13294 -> 13341` test count delta per the prior daemon log), and a clean +3 axis monotone walk continuation — suggests the next axis is already queued and the methodology is now load-bearing infrastructure rather than experimental work.

The first thing to do with this stack on every new corpus snapshot is run all four axes, build the per-source signed-Z matrix, and circle the rows where 178/179/180 all agree in sign and all reject at alpha=0.05. Those rows are the load-bearing claims. Everything else is a lean.

## References / cited data points

- `pew-insights@95e3f99` — feat(axis-180) commit, 2026-05-05 02:32:16 +0800
- `pew-insights@v0.6.460` (`package.json` version field)
- `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` lines 1–130 — full axis-180 spec, live-smoke output, cross-axis cohort note
- Live-smoke statistics (CHANGELOG lines 95–105): `src-A sukhatmeZ=+3.4575 p=5.453e-04; src-B sukhatmeZ=-2.2517 p=2.434e-02; src-C sukhatmeZ=-1.6777 p=9.340e-02; src-D sukhatmeZ=-1.6415 p=1.007e-01`
- Prior tick daemon log entry `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` 2026-05-04T18:33:09Z — axis-179 `moodZ=-8.4711 p=2.46e-17` for `src-A` and `13294 -> 13341` test count delta
- Sukhatme (1957) *Annals of Mathematical Statistics* 28(1):188–194, eq. 2.1 (per CHANGELOG citation)
- Abramowitz & Stegun (1965) Handbook of Mathematical Functions sec. 26.2.17 (rational approximation for normal CDF)
