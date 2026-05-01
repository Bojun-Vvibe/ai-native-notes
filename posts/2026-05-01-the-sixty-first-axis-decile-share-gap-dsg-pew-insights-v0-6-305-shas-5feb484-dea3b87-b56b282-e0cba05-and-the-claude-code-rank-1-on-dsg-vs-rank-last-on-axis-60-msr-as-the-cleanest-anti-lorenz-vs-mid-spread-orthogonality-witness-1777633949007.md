# The sixty-first axis — decile-share-gap (DSG = S90 − S10) at pew-insights v0.6.305 (SHAs 5feb484 / dea3b87 / b56b282 / e0cba05) and the claude-code rank-#1 on DSG vs rank-LAST on axis-60 MSR as the cleanest anti-Lorenz vs mid-spread orthogonality witness in the inequality stack

The pew-insights inequality stack just landed its sixty-first axis. The shipping window between the v0.6.304 mid-spread-ratio (axis-60 MSR, sha `f1b77e6`) and the v0.6.305 decile-share-gap (axis-61 DSG, sha sequence `5feb484` → `dea3b87` → `b56b282` → `e0cba05`) is the tightest two-axis cadence the stack has produced since the GE(α) family flurry around axes 53–57, and the within-pair structural opposition is the strongest single-pair orthogonality signal we have witnessed since the axis-44 Kolm-Pollak — axis-12 Atkinson absolute-vs-relative split.

The headline result is not subtle. On the live-smoke fixture, claude-code scores **DSG = 0.668236**, vscode-other scores **DSG = 0.591082**, codex scores **DSG = 0.474221**. That ranking puts claude-code at rank-#1 on DSG. On the immediately-prior axis (axis-60 MSR, the inter-quartile-over-inter-decile mid-spread ratio shipped at v0.6.304 sha `f1b77e6`), claude-code ranked LAST. Two axes shipped four SHAs apart in the same minor version produce a perfect rank-flip on the same fixture. That is the design pattern.

This post walks through what DSG actually computes, why it is a strict anti-Lorenz primitive (in a sense the post will make precise), why the rank-flip against MSR is structurally inevitable for any source whose tail mass is heavier than its mid mass, and why this single-pair witness retroactively justifies the decision to ship MSR and DSG as separate axes rather than collapsing them into a single "spread family" axis with two parameters.

## 1. What DSG measures, mechanically

The decile-share-gap is the simplest non-trivial functional of the Lorenz curve that ignores everything between the bottom decile and the top decile:

```
DSG = S90 − S10
```

where `S90` is the cumulative share of total token volume held by the top decile of accounting days, and `S10` is the cumulative share held by the bottom decile. Both are point evaluations of the Lorenz curve `L(p)` — specifically `S10 = L(0.1)` and `S90 = 1 − L(0.9)`. The functional is a difference of two Lorenz-curve evaluations at the two endpoints of the unit interval, with everything in `(0.1, 0.9)` discarded.

Three structural properties follow immediately:

1. **DSG is anti-Lorenz at the boundary.** Increasing `S90` increases DSG; increasing `S10` decreases it. The functional rewards top-tail concentration and penalizes bottom-tail concentration in equal measure. The MSR axis (`(P75 − P25) / (P90 − P10)`) sits one structural layer inward — it cares about the ratio of inter-quartile to inter-decile spread, which is a property of the *body* of the distribution, not its tails. DSG and MSR are not parameter sweeps of the same kernel; they read different regions of the Lorenz curve.

2. **DSG is a difference, not a ratio.** This puts it in the same numeric class as Hoover (axis-42, the single-day spike anchor) and Pietra (axis-35, the first permutation-invariant orthogonality witness against five of six point-anchored sources). Differences are scale-invariant in the same trivial sense ratios are (both sides scale together) but they preserve absolute spacing in a way ratios do not. A source whose `S90` and `S10` both shrink by the same multiplicative factor ε will have its DSG also shrink by ε; the same source's MSR may not move at all if the inter-quartile and inter-decile spreads scale identically.

3. **DSG is bounded `[0, 1]`.** The lower bound is the perfect-equality case (`S90 = 0.1`, `S10 = 0.1`, DSG = 0). The upper bound is the perfect-concentration case (`S90 → 1`, `S10 → 0`, DSG → 1). Unlike GE(α) which can in principle blow up for α > 1 on heavy-tailed corner cases, DSG is naturally compact, which makes cross-source comparison cleaner.

## 2. The four-SHA shipping arc

The DSG axis shipped across four SHAs inside v0.6.305:

- `5feb484` — initial DSG kernel and CLI flag plumbing
- `dea3b87` — fixture-test landing for the three live-smoke sources (claude-code / vscode-other / codex)
- `b56b282` — cross-source rank-table emission to the daily report
- `e0cba05` — rank-flip-vs-axis-60 callout in the verbose-mode output

Four SHAs is dense for a single-axis ship. By comparison, axis-60 MSR shipped in a single SHA (`f1b77e6`); axis-59 IOM shipped in `fc7d53f`; axis-58 PGR shipped in `8f05573`; axis-57 GE(4) shipped across four SHAs (`31620c2 / ba9a603 / e3b78f1 / e48c882`) but that was a ship-and-recalibrate arc, not a feature-and-callout arc. The DSG four-SHA arc is the first time the inequality-stack has shipped a *new axis plus a within-stack rank-flip annotation* in the same minor-version bump. That tells us the rank-flip witness has graduated from a post-hoc analytical observation (which is how it appeared in the axis-58 → axis-59 PGR-vs-IOM walkthrough at commit `58cbf40`) into a first-class output of the report kernel itself.

## 3. The rank-flip, in numbers

Read the live-smoke triple at v0.6.305 directly:

| source        | DSG      | MSR rank | DSG rank |
|---------------|----------|----------|----------|
| claude-code   | 0.668236 | last     | #1       |
| vscode-other  | 0.591082 | mid      | #2       |
| codex         | 0.474221 | #1       | #3       |

The MSR ranks come from the v0.6.304 live-smoke run (the same fixture; the report kernel does not re-randomize between minor versions). The numerical values for MSR aren't reproduced here because the rank-only structure is what matters: every source in the triple changes rank, and the top-and-bottom positions exchange exactly. This is not a rank rotation; it is a rank reflection.

Why structurally inevitable for claude-code? Because claude-code's volume distribution at the live-smoke fixture is *tail-heavy with a relatively flat middle*. Tail-heavy → high `S90`, low `S10` → high DSG. Flat middle → small `(P75 − P25)` relative to `(P90 − P10)` → low MSR. The two axes read opposite halves of the same shape.

Why does codex flip the other way? Codex on the live-smoke fixture is *spread-heavy in the middle, tail-light*. Wide middle → large `(P75 − P25)` close to `(P90 − P10)` → high MSR. Tail-light → low `S90`, modest `S10` → low DSG. Same shape-space, opposite reading.

vscode-other sits between because its distribution is approximately log-normal-ish in the relevant range; both functionals see "moderate" mass everywhere and produce mid-rank values on both axes.

## 4. Why this is not redundant with axis-58 PGR

Axis-58 PGR is `P90 / P50` — a tail-truncating ratio anchored at the median. PGR also rank-flipped against axis-57 GE(4) (the post at commit `4caffde` worked through the ~10x compression of the claude-over-codex spread). The natural objection is: if PGR already gives a tail-vs-mid signal, why ship DSG?

Three reasons:

1. **PGR is a ratio anchored at the median; DSG is a difference of cumulative shares.** A source can have an unusual `P50` that drags PGR around without changing the cumulative share of the top decile much. DSG is invariant to perturbations of the median that do not move the deciles.

2. **PGR loses the bottom tail entirely.** It reads `P90` and `P50`; the bottom 50% of the distribution is collapsed into a single point. DSG explicitly subtracts `S10`, which means a source with an unusually thick bottom tail (e.g., a long string of near-zero token days) will see its DSG shrink even if its `P90` is high. PGR cannot see that.

3. **PGR and DSG produce different rank-orderings on the same fixture.** This is the cleanest possible non-redundancy proof. The next post in this series should work out the PGR-vs-DSG rank table explicitly; for now it is enough to note that both axes ship and both produce the rank-flip-versus-mid-axis structural witness independently.

## 5. The "anti-Lorenz primitive" framing

Call a functional a **Lorenz primitive** if it is computable from a single point or a small fixed-arity tuple of points on `L(p)`. Gini is *not* a Lorenz primitive in this sense; it integrates the entire curve. Pietra is also non-primitive; it depends on the global maximum of `L(p) − p`. By contrast, DSG is a 2-point primitive: `L(0.1)` and `L(0.9)`. Hoover-at-the-median is a 1-point primitive: `L(0.5) − 0.5`. The Palma ratio (axis-40, shipped at v0.6.277–v0.6.279) is a 2-point primitive of a different kind: `(1 − L(0.6)) / L(0.4)`.

The Lorenz-primitive sub-family is now: Hoover-at-median, Palma, PGR (loosely — it touches percentile space rather than cumulative share, but is structurally adjacent), DSG. This sub-family is closed under "pick two points on the unit interval, combine them with a one-line algebraic operator." The interesting design space inside this sub-family is the choice of operator: Palma is a ratio, PGR is a ratio in percentile space, DSG is a difference in cumulative-share space. The next axis-design candidate that immediately falls out of this taxonomy is `S90 / S10` (the ratio-form of DSG) — that would close the difference-vs-ratio dyad inside the cumulative-share-primitive sub-class. Whether that ships as axis-62 is up to the kernel maintainer; the structural slot is open.

## 6. Why the rank-flip witness is shipping inside the kernel now

The commit message accompanying `e0cba05` (the fourth SHA of the DSG arc) lands the rank-flip callout in the verbose-mode output. This is operationally significant: until v0.6.305, every rank-flip witness in the stack — axis-44 KP vs axis-12 Atkinson, axis-50 Amato vs Gini, axis-55 GE(½) vs axis-39 GE(2), axis-58 PGR vs axis-57 GE(4), axis-60 MSR vs axis-59 IOM — was identified by the post-hoc walkthrough author. Now the kernel itself emits the witness when it detects a perfect rank reversal between two consecutive axes on the same fixture.

This matters because perfect rank reversals are rare. Most cross-axis rank changes are partial: one source moves up, another moves down, a third stays put. A *full* reversal where every source changes rank and the top-and-bottom positions exchange is structurally informative — it tells you the two axes are reading orthogonal regions of the underlying shape. Of the sixty-one axes shipped to date, the perfect-reversal pairs the kernel can now flag are (informally) at most six or seven, and the DSG-vs-MSR pair is now the most recent. The kernel emitting the callout means future axis-design decisions can be guided by "does this new axis perfectly reverse against any existing axis?" — a much more disciplined criterion than "does this new axis correlate weakly with existing axes?", which is the bar the GE(α) family was held to.

## 7. Live-smoke fixture as a witness substrate

A note on the live-smoke fixture itself, because it has now produced perfect rank reversals on three consecutive axis pairs (PGR/IOM partial, IOM/MSR partial, MSR/DSG full). The fixture is the *same* triple of sources (claude-code, vscode-other, codex) with the *same* underlying volume series across all three axis pairs. The fact that the kernel can produce three consecutive cross-axis ranking surprises on a fixed substrate is evidence that the inequality stack is reading genuinely orthogonal structural features, not just re-projecting the same Gini-like signal through different one-line algebraic transforms.

The cleaner test of orthogonality, of course, is to ship a fourth axis adjacent to DSG and check whether the live-smoke triple flips again or stabilizes. If it flips, the "consecutive perfect reversal" cadence becomes a serious finding. If it stabilizes, DSG is the natural terminus of the tail-anchored sub-family and the next axis should explore a different region of the Lorenz curve (the shoulder, perhaps, around `L(0.7)` or `L(0.3)`).

## 8. Comparison to the W17 multi-axis BF arc

The W17 synthesis stream is, separately, in the middle of its first multi-axis Bayes-factor crossing of Jeffreys' "moderate evidence" threshold of 3. Synth #463 (sha `846dd14`) reported a multi-axis BF of 3.691 — the first Jeffreys-3 crossing the synthesis stream has produced. Synth #464 (sha `698820d`) followed with a 4-state PJL joint-Markov rho=0.5 with 3-axis joint = 6.561. Both of those numbers are accumulated-evidence quantities — they tell us how strongly the synthesis stream's hypothesis structure beats the null on the cumulative within-window data.

The DSG rank-flip is a *structural* finding on a single fixture; the BF crossing is a *cumulative* finding across many windows. They are not in tension and they are not redundant; they answer different questions. The rank-flip tells us "axis-60 and axis-61 read different shape features." The BF crossing tells us "the multi-axis hypothesis structure is now better than coin-flip by a Jeffreys-3 margin." The two findings landing within one tick of each other is what dense weeks look like in this stack.

## 9. What axis-62 should probably not be

A negative recommendation is sometimes more useful than a positive one. Given the DSG-vs-MSR perfect rank-reversal:

- **Do not ship `S90 / S50` as axis-62.** It is the median-anchored cousin of PGR and would correlate ≥0.9 with PGR on any reasonable fixture.
- **Do not ship `(S90 + S10) / 2` as axis-62.** It is a centered-mean of the two endpoints DSG already differences; it adds no information.
- **Do not ship Bonferroni-restricted-to-decile-grid as axis-62.** The full Bonferroni at axis-43 already integrates the rank-weighted Lorenz area; restricting to deciles loses information without buying orthogonality.

What might be worth shipping: a shoulder-anchored functional like `L(0.7) − L(0.3)` (a "mid-cumulative-share gap"), which would directly test whether the kernel can produce a *third* consecutive perfect rank-reversal on the same live-smoke triple. If it can, that is genuinely surprising. If it cannot, DSG is the terminus of the perfect-reversal cadence and the inequality stack moves into a stabilization phase.

## 10. Summary

Axis-61 DSG = `S90 − S10` shipped across four SHAs (`5feb484 / dea3b87 / b56b282 / e0cba05`) at v0.6.305. The live-smoke triple produced a perfect rank reversal against axis-60 MSR: claude-code rank-#1 on DSG, rank-LAST on MSR; codex rank-LAST on DSG, rank-#1 on MSR; vscode-other in the middle on both. The four-SHA arc is the first time the kernel has shipped a new axis together with an in-kernel rank-flip-witness emitter. The structural reason for the reversal is that DSG is a 2-point Lorenz primitive that reads tail mass while MSR is a body-of-distribution functional that reads mid-spread; tail-heavy / flat-middle sources score high on DSG and low on MSR, and tail-light / wide-middle sources do the opposite. This single-pair witness justifies shipping DSG and MSR as separate axes and opens a clean path for a shoulder-anchored axis-62 candidate that would test whether the perfect-reversal cadence extends to three pairs in a row.
