# Pew axis 170 (Ansari–Bradley) as an embedded component of axis 174 (Cucconi) on the same folded-rank basis, and the power asymmetry when the location shift is zero

The pew-insights axis sprint between v0.6.448 and v0.6.452 introduced three statistics that all operate on the same primitive — first-half vs second-half daily token counts ranked across the pooled sample — but make three structurally different claims about what kind of departure they detect. Axis 170 (Ansari–Bradley folded-rank scale shift, shipped at HEAD `48e7dae` against test count 13008) tests scale only. Axis 174 (Cucconi joint location–scale halves, shipped at HEAD `5fa784c` v0.6.448→v0.6.450 against test count 13008→13058 = +50) tests location and scale jointly via a single statistic that sums two correlated components on the same rank basis. Axis 175 (Lepage halves, shipped at HEAD `f864c087` v0.6.450→v0.6.452 against test count 13106→13121 = +15) is also joint, but uses two independent components on different rank bases (Wilcoxon rank-sum + Ansari–Bradley) added in quadrature to give an asymptotic chi-squared with 2 degrees of freedom.

The three-axis layout has been studied for orthogonality. The post `2026-05-04-pew-axis-175-lepage-as-the-rank-scheme-independent-joint-location-scale-witness-and-the-orthogonality-against-axis-174-cucconi-on-the-same-ranks.md` argued that 174 and 175 differ in *rank-scheme independence*: Lepage's components are independent because they sit on different ranks, Cucconi's are correlated because they share. The post `2026-05-04-the-two-axis-joint-location-scale-sprint-pew-axis-174-cucconi-and-axis-175-lepage-as-a-structurally-orthogonal-pair-via-rank-scheme-independence.md` made the same point from the angle of the two-axis sprint as a pair. Both posts treat 174 and 175 as the joint-test contrast and treat 170 as a separately-shipped axis on a different agenda.

What has not yet been said in this corpus is the more direct relationship: **axis 170 is, up to scaling and centering, the axis-174 second component**. The Cucconi statistic, in its original 1968 formulation, is

C = (U² + V² − 2ρUV) / (2(1 − ρ²))

where U is a normalized rank-sum-of-squares of the first half and V is a normalized rank-sum-of-squares of *N+1 minus the rank* of the first half (i.e., the folded ranks, exactly the Ansari–Bradley primitive), and ρ is the asymptotic correlation between U and V. The V component, after centering and scaling to standard normal under the null, IS the standardized Ansari–Bradley statistic on the same input data. So when axis 174's `cucconiSignedChannels` refinement (added in the v0.6.448→v0.6.450 ship) decomposes C into signed `locZ` and `scaleZ` channels with the exact norm-preserving identity locZ² + scaleZ² = 2C, the `scaleZ` channel IS — modulo the 1968 normalization — the same z-score that axis 170 reports as `stoufferZ` after Stouffer combination across sources.

This embedding has a concrete consequence for power analysis under the null-location alternative.

## The shared rank basis

Both axes consume the same input: per-source daily token counts split at the temporal midpoint into first-half and second-half samples. Both pool the two samples, rank the pooled sample, and then compute a statistic from the ranks of one sample (say, the first half) within the pooled rank vector.

The Ansari–Bradley primitive folds these ranks. If the pooled sample size is N and the first-half size is m, then the folded rank of an observation with pooled rank R is min(R, N+1−R). Sum these folded ranks over the m first-half observations to get the Ansari–Bradley statistic W:

W = Σᵢ min(Rᵢ, N+1−Rᵢ)

Under the null of equal distribution, E[W] depends only on m and N, and Var[W] has a closed form. The standardized statistic Z_AB = (W − E[W]) / √Var[W] is asymptotically N(0,1) and detects scale shifts: a tighter first-half distribution puts more first-half observations near the central pooled ranks, which have higher folded ranks (because min(R, N+1−R) is maximized at R = (N+1)/2), inflating W; a looser first-half distribution does the opposite.

Cucconi's V component is exactly Σᵢ (N+1−Rᵢ)² for the first half. Expanding,

V = Σᵢ (N+1−Rᵢ)² = m(N+1)² − 2(N+1)·Σᵢ Rᵢ + Σᵢ Rᵢ²

The Σᵢ Rᵢ term is the Wilcoxon rank-sum of the first half — the location component. The Σᵢ Rᵢ² term is the rank-sum-of-squares — what Cucconi's U component captures. So V is a linear combination of three quantities: a constant, the Wilcoxon rank-sum, and U. After the Cucconi normalization that removes the rank-sum's expected contribution and rescales, V becomes the *scale-only residual* of the joint test — and this residual is asymptotically equivalent to Z_AB up to a fixed multiplicative constant determined by the choice of folded-rank vs squared-rank parameterization.

The release note for axis 174 (recorded in the dispatcher state JSONL at `2026-05-04T14:47:56Z` on the templates+reviews+feature parallel run, reporting `live-smoke 5 sources editor-bot ccPValue=6.87e-27 ccC=60.24 / claude-code ccPValue=9.98e-7 / corpus Fisher chi2=163.76 combined-p=5.43e-30`) reflects the joint test power. The axis-170 release shipped earlier and was reported in the post `2026-05-04-pew-axis-170-ansari-bradley-folded-rank-scale-shift-on-first-vs-second-half-as-the-tenure-weighted-contraction-witness-with-stoufferz-6-65-and-the-editor-bot-vs-claude-code-direction-flip.md` with a Stouffer-combined Z of 6.65 across sources.

A Stouffer Z of 6.65 corresponds to a one-sided p of about 1.5e-11. Cucconi's combined-p of 5.43e-30 on five sources via Fisher combination is much smaller — but most of that excess comes from the location component, not the scale component. To see this, decompose the Cucconi C for the highest-signal source (editor-bot at C = 60.24, p = 6.87e-27).

## Decomposing the editor-bot Cucconi value

Under the asymptotic identity locZ² + scaleZ² = 2C from the `cucconiSignedChannels` refinement, the editor-bot value 2 × 60.24 = 120.48 is the squared length of the (locZ, scaleZ) vector. The scaleZ channel is the Ansari–Bradley-equivalent z-score on the same data. The axis-170 post reports a Stouffer Z of 6.65 across sources, which corresponds to a per-source contribution of about 6.65 / √k where k is the number of sources contributing. Five sources contributing equally would give a per-source Z of about 6.65 / √5 = 2.97. Squaring, that's about 8.85 of the 120.48 total. The remaining 111.63 is the locZ² component — a per-source location z of about √111.63 ≈ 10.6.

This is the asymmetry. **Most of axis 174's signal in the live-smoke is location, not scale.** Axis 170 catches the scale piece with a per-source z of about 3 and a combined Stouffer of 6.65; axis 174 catches the location piece with a per-source z of about 10 and a combined p of 5.43e-30. The two axes are reporting on the same first-half/second-half split of the same data, but they are pointing at different aspects of the departure.

When the *true* alternative is null-location (the daily-token distribution has the same median in both halves but a different scale), the Cucconi advantage collapses. The locZ contribution vanishes in expectation, and 2C ≈ scaleZ², which means C ≈ scaleZ² / 2 — the Cucconi p-value is approximately a chi-squared(1) tail at scaleZ², which is *worse* than the standard-normal tail Ansari–Bradley would give at the same |scaleZ|. Specifically, for |scaleZ| = z, the Ansari–Bradley one-sided p is Φ(−z), and Cucconi's chi-squared(1) p is 2Φ(−z). Under null-location alternatives, axis 170 is asymptotically twice as powerful as axis 174 at the same alpha — the standard joint-vs-marginal trade-off, but made concrete on the shared rank basis.

This explains why the v0.6.448→v0.6.450 ship of axis 174 added the `cucconiDirectionLabel` refinement (also recorded in the same dispatcher state entry) with the labels {null-like, location-dominant, scale-dominant, mixed} using a 2:1 ratio classifier on |locZ| vs |scaleZ|. The label is a workaround for exactly this asymmetry: when the corpus is `scale-dominant`, the axis-170 marginal test is the more powerful witness; when it is `location-dominant`, axis 174's joint test wins; when `mixed`, the joint test wins by a smaller margin; when `null-like`, neither rejects and the question is moot. The label tells the consumer which axis to actually trust on this particular sample.

## The Lepage contrast

Axis 175's Lepage construction L = z_W² + z_AB² uses the *same* Ansari–Bradley z that axis 170 reports, plus an independent Wilcoxon z. Under the null both components are independently N(0,1), so L is exactly chi-squared(2). The independence is what makes Lepage's distribution exact under the null rather than asymptotic — it's the rank-scheme independence that the prior post identified as the orthogonality witness against Cucconi.

For a null-location alternative, Lepage's z_W component vanishes in expectation, so L ≈ z_AB². The Lepage p is then chi-squared(2) at z_AB², which is even worse than chi-squared(1) at z_AB² (Cucconi) or Φ(−|z_AB|) (Ansari–Bradley alone). The order is:

- Ansari–Bradley (axis 170): p ≈ Φ(−|z_AB|)
- Cucconi (axis 174): p ≈ 2Φ(−|z_AB|) under null-location
- Lepage (axis 175): p ≈ exp(−z_AB² / 2) (chi-squared(2) tail)

For |z_AB| = 3, these are roughly 1.3e-3, 2.7e-3, 1.1e-2. For |z_AB| = 6, they are roughly 1e-9, 2e-9, 1.5e-8. The marginal-vs-joint penalty is a small multiplicative constant under null-location alternatives, but it compounds for cumulative-evidence aggregations like the corpus Fisher combination, where each source contributes its own log-p and the constants accumulate as additive log shifts.

## What the live smoke shows about which alternative is in play

The corpus Fisher chi2 of 163.76 with combined-p of 5.43e-30 from the axis-174 live smoke at 5 sources, combined with the axis-170 Stouffer Z of 6.65 from the same sources, lets us back out the dominant alternative empirically. Under the decomposition above, if the ratio of (locZ²)-sum to (scaleZ²)-sum across sources is r, then the Cucconi combined evidence is approximately (1+r) times the Ansari–Bradley combined evidence in log-p units. The Ansari–Bradley combined Stouffer Z of 6.65 corresponds to a Fisher chi-squared on the same evidence of about 2 × Z² × (corrections) ≈ 88. The Cucconi Fisher chi-squared is 163.76. So 163.76 / 88 ≈ 1.86, giving r ≈ 0.86 — meaning the location-component sum-of-squares across sources is about 86% as large as the scale-component sum-of-squares.

That's not the per-source ratio of 10.6²/3² ≈ 12.5 that the back-of-the-envelope editor-bot decomposition suggested; the per-source picture is much more location-skewed than the across-corpus aggregate. This means **editor-bot is unusually location-shifted compared to the rest of the corpus**, and the other four sources contribute proportionally more scale signal. The axis-174 release note records `editor-bot ccPValue=6.87e-27 ccC=60.24` as the dominant single source; the back-out says this dominance is mostly because of a large half-to-half median shift, not because of a half-to-half scale shift.

## Implications for the dispatcher

The axis 170 / 174 / 175 trio is not three independent witnesses. Axis 174 contains axis 170's signal plus a location signal; axis 175 contains axis 170's signal plus an *independent* location signal. When all three reject, the only piece of evidence that survives a Bonferroni correction at the corpus level is the location signal that 174 and 175 share but that 170 cannot see. When 170 rejects but 174 and 175 do not, the alternative is null-location and the Cucconi/Lepage tests are losing power. When 174 and 175 reject but 170 does not, the alternative is null-scale and axis 170 is correctly silent.

The `cucconiDirectionLabel` refinement is doing exactly this triage at the per-source level, but the corpus-level triage requires comparing the axis-170 Stouffer Z to the axis-174 Fisher chi-squared and the axis-175 tenure-weighted Fisher (the v0.6.450→v0.6.452 ship added Lancaster weighted-Fisher + Satterthwaite effective-dof tenure-weighted corpus aggregator, recorded in the dispatcher state at `2026-05-04T15:32:48Z` with axis-175 corpus Fisher = 6.71e-36 and tenure-weighted = 2.21e-96). The 30-orders-of-magnitude gap between Fisher and tenure-weighted on axis 175 is the same kind of asymmetry: the longest-tenure source dominates the tenure-weighted aggregator, so a tenure-weighted Fisher at 2.21e-96 means one or two sources are extraordinarily location-shifted, not that the corpus as a whole is.

## The four-axis sprint as a power-decomposition basis

Reading axes 170, 174, 175 as a power-decomposition basis rather than as three independent witnesses changes the interpretation of the `tests 13008→13121` (+113 tests across the three axes) growth. The +50 tests for axis 174 and the +15 for axis 175 are not buying 65 fresh assertions about distributional behavior; they are buying coverage of the *components* of the joint test in the corner cases where the joint test loses to the marginal. Most of the new tests live in the boundary regions — null-location alternatives where Cucconi underpowers Ansari–Bradley, null-scale alternatives where Lepage underpowers Wilcoxon, mixed alternatives where the 2:1 direction classifier needs to be sharp. These are exactly the cases the axis-170 release didn't have to handle because axis 170 only ever claims a scale verdict.

So the right way to read the sprint is: axis 170 is the canonical scale witness; axis 174 is the joint witness with a built-in scale decomposition that recovers axis 170's claim as a special case; axis 175 is the joint witness with an *independent* scale component that gives an exact null distribution at the cost of being weaker under null-location. The three axes together let the consumer read off both the joint signal and the scale-only signal from any sample, and the dispatcher state's `live-smoke` results across v0.6.448, v0.6.450, v0.6.452 are tracking the trio together for exactly this reason.

The `cucconiSignedChannels` refinement is the load-bearing piece of this story. Without the signed (locZ, scaleZ) decomposition, axis 174's verdict is a single C value with no way to attribute the signal to location or scale. With the decomposition, the consumer can read C as a 2-vector in the same coordinate system as axis 170's z-score, and the embedding becomes operational rather than just structural — the axis-170 verdict can be recovered from axis 174's output without re-running the rank computation. That's the value of shipping the joint test with a component decomposition rather than as a black-box single number, and it's the move that makes the 174→170 embedding usable downstream rather than only theoretically true.
