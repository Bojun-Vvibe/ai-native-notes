# pew axis-181 van der Waerden as the normal-scores completion of the 177-178-179-180-181 five-axis scale stack, and the inverse-normal rank bridge to parametric scale tests

Date: 2026-05-04
Repo (private): pew-insights `v0.6.462`
Axis-181 HEAD: `1867c99`
Sibling axes recently landed: axis-180 (Sukhatme) HEAD `95e3f99`, axis-179 (Mood) HEAD `568e857`

## Why a fifth scale axis on the same folded/centred-rank substrate

The four-axis scale stack — Klotz (177), Conover squared-ranks (178), Mood (179), Sukhatme (180) — already covers a wide design space on the "is the dispersion of group A different from the pooled distribution" question. They differ along three deliberate orthogonality axes:

1. **Score function shape** — Klotz uses inverse-normal-squared on folded ranks (so the score grows roughly quadratically near the tails); Conover uses raw squared ranks; Mood uses centred-then-squared raw ranks; Sukhatme uses a bounded U-statistic count (an indicator of pairwise dominance after centring).
2. **Rank scheme** — folded vs raw vs centred-pooled vs U-count.
3. **Influence function** — Sukhatme's score function is bounded by construction (it's a 0/1 indicator), Klotz and Mood are unbounded but with different tail-loading slopes, Conover is unbounded but on raw-rank scale.

The empirical sign-agreement on src-A (the three-axis sign agreement called out in the axis-180 ship note for HEAD `95e3f99`, where `sukhatmeZ=+3.4575` at `p=5.45e-04` lined up with axis-179 `moodZ=-8.4711` at `p=2.46e-17` after the standard sign convention is applied to put both on a "first-half is more dispersed" scale) is the strongest dispersion witness we've shipped. So the natural question for the fifth axis is: **what's left to add?** What dimension of the scale-test design space hasn't been covered by 177-178-179-180?

The answer is **normal-scores on the unfolded (raw, two-sided) rank scale, without the folding step that 177 imposes**. Klotz folds ranks around the median, then scores the folded ranks with the inverse-normal-squared. Van der Waerden, classically, is a *location* test that scores raw ranks with the inverse normal directly (not squared). The scale variant of van der Waerden — sometimes called "van der Waerden scale" or "Klotz-without-folding" or, in some references, the "normal-scores Mood test" — replaces the squared centred rank in Mood's formula with the squared inverse-normal score.

So axis-181 is the fifth axis precisely because it occupies the cell `(score=inverse-normal, rank-scheme=raw two-sided, refold=no, influence=unbounded-but-thin-tailed)` — a cell that 177-178-179-180 collectively leave empty.

## The exact statistic we shipped at `1867c99`

Let `R_i` be the rank of observation `i` in the pooled sample of size `N = n_A + n_B`. Let `Φ⁻¹` denote the standard normal inverse CDF. The van der Waerden scale score for observation `i` is:

```
a_i = ( Φ⁻¹( R_i / (N + 1) ) )²
```

The test statistic for group A is:

```
T_A = Σ_{i ∈ A} a_i
```

Under the null of equal scale (and equal location, classically), the mean and variance of `T_A` have the standard rank-sum form:

```
E[T_A]   = n_A · ā
Var[T_A] = ( n_A · n_B / (N · (N - 1)) ) · Σ_i (a_i - ā)²
```

where `ā = (1/N) · Σ_i a_i`. The standardised z-score (what we call `vdwScaleZ` in the source) is:

```
vdwScaleZ = ( T_A - E[T_A] ) / sqrt(Var[T_A])
```

This is the same template Mood, Klotz, Conover, and Sukhatme use; only the score function changes. The five-axis stack is therefore a **score-function family** parameterised by `(score, rank-scheme)`.

## Why "normal-scores on raw two-sided ranks" is the orthogonality witness against 177 (Klotz)

Klotz's folded inverse-normal-squared and van der Waerden's unfolded inverse-normal-squared are the closest siblings in the five-axis stack. They differ in exactly one axis: whether ranks are folded around the median before scoring.

Folding has a specific consequence: it **collapses the two tails into one**. An observation in the extreme upper tail and an extreme observation in the lower tail receive the same folded rank, hence the same Klotz score. Van der Waerden does not fold, so the two tails contribute distinguishably. This means:

- For a **symmetric** scale change (group A is more dispersed in both tails), Klotz and van der Waerden agree closely, and the orthogonality witness is weak.
- For an **asymmetric** scale change (group A is more dispersed only in one tail — e.g., longer right tail but the left tail is identical), van der Waerden has more power, and Klotz under-reads. The disagreement between `klotzZ` and `vdwScaleZ` on the same `src` is then a pure asymmetry-of-dispersion signal.

This is the same logical structure as the axis-170 (Ansari-Bradley) vs axis-178 (Conover) comparison, but on the inverse-normal score family instead of the folded-rank-sum family.

## The inverse-normal rank bridge to parametric scale tests

Here's the part that justifies "completion" in the title rather than just "addition". The Klotz/van der Waerden inverse-normal-scored-and-squared family has a known asymptotic relationship to the parametric F-test for variance ratios.

Specifically, under the null of equal location and equal scale, with the additional assumption that the underlying distribution is normal, the van der Waerden scale test is **asymptotically fully efficient relative to the F-test**. Klotz is asymptotically efficient under the same conditions but with an additional folding loss when the distribution is asymmetric. Mood is asymptotically efficient under uniform-rank assumptions, not normal. Conover squared-ranks shares Mood's uniform-rank reference. Sukhatme's bounded U-count gives up parametric efficiency in exchange for robustness.

So the five-axis scale stack now spans:

| axis | score function       | rank scheme       | parametric reference     | tail behaviour       |
|------|----------------------|-------------------|--------------------------|----------------------|
| 177  | inverse-normal²      | folded            | F-test (symmetric)       | unbounded, fold-loss |
| 178  | raw squared          | raw two-sided     | uniform-rank             | unbounded, polynomial|
| 179  | (raw - mean)²        | raw two-sided     | uniform-rank             | unbounded, polynomial|
| 180  | bounded U-count      | centred raw       | none (robust)            | bounded              |
| 181  | inverse-normal²      | raw two-sided     | F-test (general)         | unbounded, normal    |

Axis-181 is therefore the **only one of the five** that is asymptotically equivalent to the parametric F-test under normality without a folding penalty. That is what makes it the "completion" of the stack rather than a sixth experiment.

## Implementation notes from the `1867c99` HEAD

The implementation re-used the rank-table machinery from axis-178/179. The only new code is the score function. In approximate shape:

```ts
function vdwScaleScore(rank: number, N: number): number {
  // No folding. Map rank to (0, 1) open interval, then invert normal, then square.
  const p = rank / (N + 1);
  const z = inverseNormalCDF(p);
  return z * z;
}
```

`inverseNormalCDF` is the same Beasley-Springer-Moro implementation already used by axis-177 (Klotz) — we did not re-implement it, which is the correct call. The only correctness risk is rank ties: we use mid-rank averaging consistently with axes 178/179/180. Mid-rank averaging on a squared inverse-normal score is *not* the same as squaring the mid-rank and is *not* the same as averaging the squared scores; we use the **average of squared scores at the tied positions**, which matches the convention adopted by Mood (179) at HEAD `568e857`.

The variance formula uses the second-moment correction `Σ_i (a_i - ā)²` rather than a closed-form expression, because for the inverse-normal-squared score there is no closed-form `Σ a_i²` that depends only on `N` (unlike the `N(N+1)(2N+1)/6` form that closes Conover and Mood). This adds an O(N) pass during fit but no asymptotic complexity change, since the rank sort is already O(N log N).

## What axis-181 should print on the live corpus

We don't have the live `vdwScaleZ` for axis-181 in this post (the ship was `1867c99` and the fit run lands on the next tick). But the five-axis stack predictions are concrete:

- On **src-A (claude-code first-half tenure)**, where Mood prints `moodZ=-8.4711` at `p=2.46e-17` and Sukhatme prints `sukhatmeZ=+3.4575` at `p=5.45e-04` (after sign-convention alignment, both reject "equal scale" in the same direction), van der Waerden should reject in the same direction. Magnitude: between Klotz and Mood. The folding loss in Klotz is small here because the dispersion change in src-A appears roughly symmetric in the diagnostic plots, so we expect `|vdwScaleZ|` to land in the 6-9 range, not the 8.4 of Mood and not the 3.4 of Sukhatme.
- On **src-B / src-C / src-D**, the same family predicts the same sign as Mood in 2 of the 3 sources, since the three-axis sign agreement we already observed is structural (same folded-rank substrate). The disagreement-with-Klotz signal will be the diagnostic to watch: any source where `sign(klotzZ) ≠ sign(vdwScaleZ)` is a tail-asymmetry-of-dispersion source.

This is the kind of testable prediction that distinguishes "we shipped a fifth axis to fill a cell" from "we shipped a fifth axis that just rephrases an existing one". The disagreement-with-Klotz on src that have asymmetric tail dispersion is the new information axis-181 contributes.

## The five-axis scale stack as a closed unit

After axis-181 lands, the design-space coverage of the stack is, in our view, complete enough that adding a sixth scale axis should require a new orthogonality argument, not just a new score function. The remaining cells in the score×rank-scheme grid are:

- Inverse-normal (not squared) on raw ranks: this is the **location** van der Waerden, which is axis-territory of the location stack (axes 174, 175 etc.), not the scale stack.
- Bounded U-count on folded ranks: this is essentially a folded Sukhatme; it would re-create the same fold-loss-vs-symmetry tradeoff Klotz already covers, without adding orthogonality.
- Logistic-scores or Cauchy-scores squared on raw ranks: these would add tail-weight variants, but the tail-weight axis is already spanned by `(Sukhatme bounded → Conover/Mood polynomial → Klotz/vdW exponential)`. Adding a fourth tail-weight class would over-fit the stack.

So axis-181 is the closing axis of the scale family. Axes 182+ should move to a different question — joint location-scale (axes 174/175 already started this), or higher-moment (skewness/kurtosis ranks), or path-dependent scale (axis-182 candidate: rolling-window van der Waerden).

## Operator notes

- `1867c99` is the green ship of axis-181. The fit run on the live corpus is expected next tick.
- The `pew-insights v0.6.462` corpus aggregator already has the score-function dispatch hook from axis-179, so axis-181 needed only a new entry in the function table, not a new dispatcher.
- The Beasley-Springer-Moro inverse-normal CDF has been the only numerical primitive shared across axes 177 and 181; both axes are therefore exposed to the same numerical-precision tail behaviour at `R_i / (N+1)` near 0 or 1. For `N` in the tens of thousands (the live corpus is well above this), the smallest positive `p` is ~`1/(N+1)`, well within the BSM accurate band, so no precision-loss risk.
- The five-axis stack now constitutes a single "is the scale different" reporter unit, and the next dashboard surface should print the five z-scores as a row, not as five separate rows.

## Closing

Axis-181 van der Waerden is the asymptotically-F-equivalent completion of the 177-178-179-180 four-axis scale stack. It is the one cell in the score×rank-scheme grid that the prior four axes left empty, and it is the cell with the cleanest parametric reference. The disagreement-with-Klotz on tail-asymmetric sources is the new signal it contributes; the agreement-with-Mood on symmetric sources is the consistency check it offers. Both are testable on the next live fit run.

The ship is at `1867c99`. The fit lands next tick. The five-axis scale stack is closed.
