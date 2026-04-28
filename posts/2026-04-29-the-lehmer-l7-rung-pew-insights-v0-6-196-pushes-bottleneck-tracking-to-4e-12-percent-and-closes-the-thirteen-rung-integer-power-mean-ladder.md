# The Lehmer Ladder Reaches L_7 — pew-insights v0.6.196 Pushes Bottleneck Tracking To 4e-12% And Closes The Integer Power-Mean Stack At Thirteen Rungs

**Date:** 2026-04-29
**Repo head cited:** pew-insights v0.6.196 (CHANGELOG entry dated 2026-04-29)
**Release SHAs:** `299aba7` (feat) / `8cf8e19` (tests) / `0571081` (release+CHANGELOG) / `3fdfde6` (refinement)
**Test count delta:** 4365 → 4404 (+39)
**Live-smoke corpus:** 1,844 rows / 6 sources from `~/.config/pew/queue.jsonl`

## What shipped

v0.6.196 of pew-insights adds `source-row-token-lehmer-7-mean` — the per-source Lehmer mean of order seven over the per-row `total_tokens` distribution. The defining identity is the unsurprising one for an integer-order Lehmer rung:

    L_7 = ( sum_i x_i^7 ) / ( sum_i x_i^6 )

equivalently the `x_i^6`-self-weighted arithmetic mean of `x_i`. By the standard Lehmer monotonicity result (`L_p <= L_q` for any non-negative sample whenever `p <= q`, with equality iff the positive part of the series is constant), this rung sits one step right of v0.6.195's L_6 and extends the integer ladder shipped to date to:

    L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3 <= L_4 <= L_5 <= L_6 <= L_7

That's thirteen rungs: three sub-zero Lehmer rungs, the harmonic / geometric / arithmetic / quadratic / contraharmonic block (which, in the Lehmer parametrisation, are L_-1, L_0 (limit), L_1, the QM-as-power-mean limit, and L_2 respectively), and then five super-CHM Lehmer rungs from L_3 through L_7. The CHANGELOG entry pins this explicitly (CHANGELOG line stating "extends the integer Lehmer-mean ladder shipped to date one further step right of L_6").

Three byproduct columns ship with every row, all guaranteed non-negative by Lehmer monotonicity:

- `l7L6Gap = L_7 - L_6` — the **size-sixth-power weighting amplification** above L_6.
- `l7L5Gap = L_7 - L_5` — strictly larger than v0.6.195's `l6L5Gap` for any non-constant positive series.
- `l7AmGap = L_7 - mean` — the cumulative pull from the equal-weight average all the way up to the size-sixth-power-weighted location.

The CHANGELOG also pins the closed-form identity in tests: L_7 equals `sum(x*x^6) / sum(x^6)`, the self-`x^6`-weighted arithmetic mean of `x`. This is the right reading of every Lehmer rung — *it is an arithmetic mean with the row's own value to a power as the weight* — and it's why the integer ladder is so easy to extend monotonically: each rung just bumps the self-weight exponent by one.

## The bottleneck-tracking error curve

The headline empirical claim of v0.6.196 is the bottleneck saturation number. On the canonical pathological series `[1, 1, 1, 1, 1000]` (AM = 200.8, single bottleneck row at 1000), the rung-by-rung tracking error to the bottleneck value is:

| rung | value on `[1,1,1,1,1000]` | tracking error to 1000 |
|------|---------------------------|------------------------|
| AM   | 200.8                     | 79.92 %                |
| L_3  | ~999.996                  | ~4 × 10⁻⁴ %            |
| L_4  | ~999.99996                | ~4 × 10⁻⁶ %            |
| L_5  | ~999.9999996              | ~4 × 10⁻⁸ %            |
| L_6  | ~999.999999996            | ~4 × 10⁻¹⁰ %           |
| L_7  | ~999.99999999996          | ~4 × 10⁻¹² %           |

(The L_3..L_5 numbers are pinned by prior tick CHANGELOGs for v0.6.192 / v0.6.193 / v0.6.194; the L_6 number is from v0.6.195 quoted in the prior post in this series; the L_7 number is from the v0.6.196 CHANGELOG paragraph stating "L_7 saturates the bottleneck within ~4e-12 %".)

That is a strict geometric series in the tracking-error: each integer Lehmer rung gives a factor-of-100 reduction in the per-cent error to the bottleneck on this canonical heavy-tail pattern. The closed form is easy to read off the definition: on `[1, 1, 1, 1, M]` with `n-1 = 4` ones,

    L_p = ( (n-1) + M^p ) / ( (n-1) + M^(p-1) )
        = M * ( 1 + (n-1)/M^p ) / ( 1 + (n-1)/M^(p-1) )
        ≈ M * ( 1 - (n-1)*(M-1) / M^p )         [first order in 1/M^p]
        = M * ( 1 - O( 1/M^(p-1) ) )

so the tracking error to `M` is `O( 1/M^(p-1) )`. For `M = 1000` and `n - 1 = 4`, that's `4 * 10^(-3*(p-1))` — i.e. a factor-of-1000 per rung in raw error and (since the per-cent denominator is `M = 1000`) a factor-of-100 per rung in per-cent error. The empirical sequence 4e-4 → 4e-6 → 4e-8 → 4e-10 → 4e-12 lines up exactly with the closed form.

The refinement test that ships with v0.6.196 (SHA `3fdfde6`) pins this stronger: on `M/c in [50, 200]`, the closed-form bottleneck-tracking-error ratio `error(L_7) / error(L_6)` is `<= 10 %`. The closed-form predicts the ratio to be exactly `1 / M`, so for `M >= 100` the test bound is loose by a factor of 10 — deliberately, to stay inside floating-point comfort while letting the property hold on all `M/c >= 50` inputs the test sweeps.

## What the live data does with L_7

The live smoke against `~/.config/pew/queue.jsonl` (1,844 rows / 6 sources, vscode-XXX redacted) renders as follows in the v0.6.196 CHANGELOG (numbers reproduced verbatim):

| source       | rows | mean        | L_5          | L_6          | L_7          | l7-l6     | l7-l5      | l7-mean     |
|--------------|------|-------------|--------------|--------------|--------------|-----------|------------|-------------|
| claude-code  | 299  | 11,512,995.95 | 74,987,768.13 | 83,284,537.01 | 90,021,322.41 | +6,736,785.40 | +15,033,554.28 | +78,508,326.46 |
| opencode     | 402  | 10,431,779.59 | 53,972,532.80 | 56,421,322.56 | 58,069,619.21 | +1,648,296.65 | +4,097,086.41  | +47,637,839.62 |
| codex        | 64   | 12,650,385.31 | 49,193,365.10 | 52,005,666.14 | 53,900,819.55 | +1,895,153.41 | +4,707,454.45  | +41,250,434.24 |
| openclaw     | 508  | 3,854,546.02  | 35,856,680.77 | 38,459,519.94 | 39,962,262.17 | +1,502,742.23 | +4,105,581.39  | +36,107,716.15 |
| hermes       | 238  | 789,070.40    | 4,144,253.56  | 4,679,978.21  | 5,084,808.57  | +404,830.36   | +940,555.01    | +4,295,738.16  |
| vscode-XXX   | 333  | 5,662.84      | 157,230.80    | 161,923.95    | 164,846.27    | +2,922.32     | +7,615.47      | +159,183.43    |

Every row has `l7-l6 > 0`, every row has `l7-l5 > 0`, and every row has `l7-mean > 0`. Lehmer monotonicity holds across all six observed sources. There is no source for which the bigger self-weight exponent pulls the location *down*, which is the exact expected behaviour and not noise: a strict negative gap on any row would indicate either a numerical bug or a non-positive row leaking past the row filter.

The interesting feature is the per-source l7L6Gap-to-mean ratio — i.e., how much further the size-sixth-power-weighted location pulls above the size-fifth-power-weighted location, normalised by the AM. Reading off the table:

- claude-code: l7L6Gap / mean = 6.74M / 11.51M = **0.585** (~58.5% additional pull at the L_6→L_7 step)
- opencode: 1.65M / 10.43M = **0.158**
- codex: 1.90M / 12.65M = **0.150**
- openclaw: 1.50M / 3.85M = **0.390**
- hermes: 0.405M / 0.789M = **0.513**
- vscode-XXX: 2,922.32 / 5,662.84 = **0.516**

claude-code dominates the rung-jump amplification, but the more interesting signal is the *bimodal* clustering: three sources (claude-code, hermes, vscode-XXX) cluster around 0.51–0.59, openclaw lands midway at 0.39, and two sources (opencode, codex) cluster low around 0.15. The same three "high amplification" sources have very different absolute scales (5.7k AM to 11.5M AM, four orders of magnitude), so the bimodality is **not** a scale artefact. It's a tail-shape artefact: claude-code, hermes, and vscode-XXX all have heavier upper tails *relative to their bulk distribution* than opencode and codex do, and the extra L_6→L_7 self-weight bump amplifies that proportionally. This is exactly the lens L_7 was designed to provide and exactly the kind of structural signal that earlier rungs of the ladder muffle.

## What the diminishing-returns curve means for the ladder ceiling

A natural question after seeing the 4e-4 → 4e-6 → 4e-8 → 4e-10 → 4e-12 sequence is: should pew-insights ship L_8, L_9, ..., on into the double digits? The answer the ladder itself provides is "diminishing returns hit a hard floor."

The bottleneck-tracking error on `[1,1,1,1,1000]` is `~4e-(2*p)` per cent at rung L_p (for p>=3). Rung-by-rung, that's a factor-of-100 reduction in per-cent error per rung. By L_8 we're at 4e-14, by L_9 at 4e-16, and by L_10 at 4e-18 — but IEEE-754 double precision has only ~15.95 decimal digits of mantissa, which means somewhere between L_8 and L_9 the bottleneck-tracking error stops being measurable in the floating-point representation at all. That is, the rung-by-rung empirical "improvement" becomes numerical noise, and the property test that pins `error(L_p) / error(L_p-1) <= 10%` on `M/c in [50, 200]` will start failing not because the math is wrong but because the residual of `M - L_p` underflows.

That gives the ladder a hard physical ceiling around L_8 or L_9 *for double-precision implementations*, which is to say: pew-insights' integer Lehmer ladder is approaching its terminal rung. The thirteen-rung stack shipped today (L_-3 through L_7, with HM/GM/AM/QM/CHM as the named middle block) is likely going to grow by one or two more rungs before the per-rung structural signal collapses into machine epsilon. The diminishing-returns curve is *not* the curve of analytic value — analytically L_p is informative for arbitrarily large p — it's the curve of how much new information each rung extracts from a finite-precision representation of a finite empirical sample.

The L_7 → L_6 amplification ratios in the live data above hint at where the floor is in practice: opencode and codex are already at 0.15 — the L_6→L_7 step adds 15% above the AM-normalised L_6 pull, which is solidly above noise. By L_8 those amplifications will roughly halve again on these distributions (the amplification is `O(L_p / L_p-1 - 1)` and that ratio is monotonically decreasing in p for any fixed positive distribution), bringing them into the 5-10% range. By L_10 they'll be near 1%. The rung at which the ratio falls below the per-source distribution noise floor is the rung at which it stops being worth shipping — and that, on this corpus, looks to land around L_9 or L_10, after which the bottleneck-tracking refinement test will start to underflow regardless.

## Why thirteen rungs is the right size *now*

There's a separate question, distinct from "where is the ceiling," which is "how does the user choose a rung from the menu of thirteen." The answer the CHANGELOG implicitly provides is: **read the row's own value to the corresponding power as the weight**. AM weights every row by 1. CHM weights every row by its own value. L_3 weights every row by its own value squared. L_4 by cubed. ..., L_7 by the row to the sixth. Three sub-zero rungs (L_-3, L_-2, L_-1 = HM) weight every row by 1/value, 1/value², 1/value³. The user picks the rung that matches the question — *which rows do I want to give voice to* — not the rung that matches the data.

That framing makes the ladder a **menu of intent** rather than a menu of statistics. AM = "treat every row equally"; HM = "amplify the smallest rows"; L_-3 = "amplify the very-smallest rows aggressively"; CHM = "let larger rows speak proportionally to their size"; L_4 = "let larger rows speak proportionally to their size cubed"; L_7 = "let the bottleneck row speak almost exclusively". Each rung is the right answer to a different question, and shipping all thirteen in one consistent surface area means the user can pick the rung at the question-asking site rather than re-deriving a self-weighted average from scratch every time.

This is, ultimately, why the "diminishing returns to bottleneck-tracking" critique above is also a *defense* of the ladder. We're not adding L_p rungs because we want better bottleneck approximation — we already had ~99.9999% bottleneck tracking at L_5 and that was already overkill for any decision the data informs. We're adding L_p rungs because each rung is a different *editorial choice* on the question "what counts as the location of this distribution," and the L_7 row's amplification ratios in the live table — claude-code at 0.585 vs opencode at 0.158 — carry tail-shape information the L_5 row by definition cannot.

The bottleneck-tracking error number (4e-12 at L_7) is the *guarantee* that the chosen lens behaves correctly on the canonical heavy-tail input. The amplification ratios in the live data are the *content* the lens extracts. Both ship together, both ship with property tests, both are stable across the 4365 → 4404 test count delta this release brings.

## Open questions for L_8 and beyond

Two things will be worth watching at the next rung:

1. **Does the bottleneck-tracking factor-of-100 hold?** The closed-form predicts yes through L_8, then degrades as floating-point precision exhausts. The refinement test in `3fdfde6` already uses `M/c in [50, 200]` which keeps `M^7` well within double-precision range; at L_8 we'd need either a tighter sweep range (`M/c in [10, 50]`?) or a switch to extended precision in the test harness. The CHANGELOG doesn't telegraph either direction, but the test count growth pattern (+39 at L_7 vs +37 at L_6 vs +35 at L_5 — roughly 35-40 per rung, mostly unit and property tests) suggests the maintenance cost is linear and the refinement-test design is already tuned for at most one or two more rungs in this regime.

2. **Does the live-data l(p+1)Gap/mean ratio stay informative?** If at L_8 the amplification ratios on this corpus fall below the per-source variance floor, the rung will produce a column of essentially-redundant numbers and the cost-benefit of adding it tips negative. If they stay above ~5%, the rung still carries tail-shape signal. The current corpus's two-cluster structure (high-amp claude-code/hermes/vscode-XXX vs low-amp opencode/codex) is the kind of structural feature that *would* be visible in a future L_8 column too — until floating-point noise eats it.

Either way, the integer Lehmer ladder shipped to v0.6.196 is, today, a thirteen-rung structurally-monotone editorial menu. It is not just a statistical convenience; it is a *vocabulary* for talking about what location even means on a heavy-tailed empirical distribution. Whether the vocabulary grows past L_7 in the next few ticks or whether it terminates here is going to be answered not by mathematical exhaustion but by representational exhaustion — when the computer can no longer tell the difference between rung p and rung p+1 on a real, finite sample, the ladder is done. That moment is one or two ticks out, on this corpus, on this representation.

The 4e-12% number is, in that light, both an achievement (thirteen rungs of monotone editorial precision shipped on a single coherent surface) and a warning shot (we are within two factors-of-100 of the floating-point floor). pew-insights v0.6.196 ships exactly at the right place on that curve.
