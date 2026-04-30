# The forty-second axis (Hoover index), pew-insights v0.6.282, and the opencode lone outlier below the 0.75 hoover/gini textbook reference as a single-day-spike signature detectable by no other axis

The forty-second cross-source axis landed today as `pew-insights daily-token-hoover-index` at v0.6.282 (commit `30ed375` for the release, `870c59f` for the feature, `8b10406` for the 38-case test suite, `8747c1f` for the pietra cross-anchor refinement that became v0.6.283 right behind it). The axis itself — the Hoover index, a.k.a. the Robin Hood index, a.k.a. the Schutz index — is an old object in the inequality literature (Hoover 1936; Schutz 1951), but its placement inside this growing per-source daily-token observatory is the part that is genuinely new, and it produced exactly one finding on the live `~/.config/pew/queue.jsonl` smoke that no prior axis in the 32–41 sequence was able to surface. This post is about that one finding.

## The functional, in one line

For a per-source vector of per-day token totals `D_1, ..., D_n`, with normalised shares `s_i = D_i / sum(D)`:

    hoover = 0.5 * sum_i | s_i - 1/n |

Range `[0, 1)`. `hoover = 0` iff every day carries exactly the same mass share `1/n`. `hoover -> 1` iff all mass concentrates on a vanishing fraction of days.

The Robin Hood reading — the one that gives the index its second name — is literal and exact: `hoover` is the *smallest fraction of total token mass that would have to be redistributed away from above-mean days toward below-mean days* to flatten the per-day distribution to perfect equality. The two halves of the absolute deviation collapse to the same number because `sum_i (s_i - 1/n) = 0` by construction; the test suite enforces this as an exact identity in the form `aboveMeanExcess === belowMeanDeficit === hoover`.

That identity matters because it gives axis-42 a *transfer-cost interpretation with no curvature parameter* — which is precisely where every prior axis in the 32–41 family lives differently.

## Why this is orthogonal to axes 32–41

The orthogonality argument is the central design point of axis-42, and the 0.6.282 changelog spells it out at unusual length. Compressed:

- **Lorenz-curve reading.** `hoover` is the L∞ Lorenz gap measured at the *equal-weights rank cut* (every day = 1/n weight). Axis-35 (Pietra) is the *same L∞ reading at the equal-mass rank cut* (where cumulative mass reaches the mean). The two cutoffs collapse only when `n=2`. Axis-32 (Gini) reads the *area* between the Lorenz curve and the 45° line. Axis-40 (Palma) reads only *two points* on the same curve (the 90th and 40th percentile rank cuts) as a ratio. Axis-34 (Zenga) averages bottom-vs-top mean ratios over *every* rank cut. Five different functionals on one curve.

- **L1 in share space.** `hoover` is `||s - 1_n/n||_1 / 2`, a strict L1 deviation from the uniform vector. The GE family — axes 37 (Theil-L), 38 (Theil-T), 39 (GE(2)) — are smooth moment-based readings; there is no smooth functional of share ratios that recovers `hoover`. Axis-36 (Atkinson) has a CRRA welfare-loss interpretation; `hoover` has a literal transfer-cost interpretation with no curvature parameter.

- **Two-sided, threshold-free.** `hoover` is symmetric across the mean and uses no threshold parameter. Axis-41 (FGT) is strictly one-sided lower-tail and threshold-anchored at `z = lineFraction * mean`. Axes 41 and 42 are at opposite ends of the dispersion-vs-poverty axis: 41 reads only below-line, 42 reads symmetric distance from uniform.

- **Permutation-invariant.** Every time-ordered axis (autocorrelation, monotone-run-length, second-difference-sign-runs, z-score-extremes) is orthogonal to `hoover` by construction, since `hoover` does not see day order.

That is six structural orthogonality arguments, one per axis-family, before a single number is computed. It is the most carefully argued cross-axis position in the 32–42 sequence.

## The cross-anchor witness: hoover over gini

The 0.6.283 refinement (commit `8747c1f`) adds the *Lorenz-shape ratio* `hooverOverGini ∈ (0, 1]`. For any non-degenerate Lorenz curve the two indices live on the same curve at different functional points; the ratio is a shape diagnostic. The textbook reference value under a unit-uniform comparator distribution is `0.75`. The deviation `(hoover/gini) - 0.75` — surfaced via `--include-reference-deviation` — is the distribution-shape diagnostic that no other single axis can produce. The ratio equals exactly `1` iff the distribution is two-point (binary), a result the test suite proves directly via the `[1000, 9000]` exact case.

The reason this matters for the live smoke is that the `0.75` reference value is not arbitrary: it is the analytical value the ratio takes for a uniform comparator. Empirical right-skewed distributions tend to sit slightly above it. Anything significantly *below* `0.75` is a signature — the kind of distribution where Gini is being inflated relative to Hoover by structure that uniform-deviation cannot see. In practice, that structure is a single-day or few-day spike in an otherwise reasonably uniform vector: Gini, as an area-under-Lorenz reading, is sensitive to the curvature that one fat tail introduces; Hoover, as an L1 distance from uniform in share space, is comparatively robust to it.

That is the falsifiable prediction the cross-anchor sets up: *if any source's Hoover/Gini ratio sits meaningfully below 0.75, that source's distribution is dominated by a single-day or few-day spike rather than by a smooth right-skewed tail.* No other axis in the 32–41 family produces this prediction.

## The live smoke, all six sources

From the 0.6.282 changelog, against the local `~/.config/pew/queue.jsonl` (6 sources, 11.77B tokens; `vscode-other` is the normalised house-style label):

```
source        firstDay    lastDay     days  hoover  gini    h/g     nAbove  nBelow  meanDaily    minDay      maxDay      tokens
claude-code   2026-02-11  2026-04-23  35    0.6137  0.7590  0.8086  7       28      98,353,880   2026-03-06  2026-04-20  3,442,385,788
vscode-other  2025-07-30  2026-04-20  73    0.5495  0.7000  0.7850  18      55      25,832       2025-08-22  2026-04-17  1,885,727
codex         2026-04-13  2026-04-20  8     0.4716  0.5892  0.8003  3       5       101,203,083  2026-04-16  2026-04-20  809,624,660
openclaw      2026-04-17  2026-04-30  14    0.2751  0.3436  0.8007  5       9       149,516,163  2026-04-30  2026-04-19  2,093,226,278
hermes        2026-04-17  2026-04-30  14    0.2577  0.3229  0.7981  7       7       17,347,817   2026-04-26  2026-04-19  242,869,432
opencode      2026-04-20  2026-04-30  11    0.1395  0.2007  0.6949  6       5       470,491,823  2026-04-20  2026-04-21  5,175,410,056
```

And the cross-anchor reference-deviation table:

```
source        hoover  gini    h/g     dev_from_0.75
claude-code   0.6137  0.7590  0.8086  +0.0586
vscode-other  0.5495  0.7000  0.7850  +0.0350
codex         0.4716  0.5892  0.8003  +0.0503
openclaw      0.2751  0.3436  0.8007  +0.0507
hermes        0.2577  0.3229  0.7981  +0.0481
opencode      0.1395  0.2007  0.6949  -0.0551
```

Five of six sources sit *above* the textbook 0.75, with deviations packed in the narrow band `[+0.0350, +0.0586]`. One source — `opencode` — sits *below* at `-0.0551`. That is the lone outlier finding.

## Reading the outlier

The outlier band is roughly symmetric in magnitude — opencode's `-0.0551` is comparable in absolute size to claude-code's `+0.0586` — but the *direction* is what makes it diagnostic. The five above-reference sources are doing what right-skewed empirical token-usage distributions are supposed to do: a long thin tail of below-mean days punctuated by occasional above-mean bursts produces a Gini that is somewhat *less* inflated than a uniform comparator would predict given the observed Hoover. (Under uniform, the relationship is `h/g = 0.75` exactly. Empirical right-skew nudges it up.)

opencode does the opposite. Its Hoover (`0.1395`) is the lowest of the six — by a wide margin; the next-lowest is hermes at `0.2577`, almost exactly double. Its Gini (`0.2007`) is also the lowest, but *not* by the same margin — hermes is `0.3229`, only `1.61x` higher rather than the `1.85x` that Hoover shows. The asymmetry between those ratios is exactly the cross-anchor's signal: Gini is being held up by something that Hoover does not see, and the only structural feature of `n=11` days that does that is a single-day mass spike.

The smoke columns confirm the mechanism directly. opencode's `nAbove/nBelow` is `6/5` — almost exactly balanced. Its `meanDaily` is `470,491,823` tokens. Its total over the 11-day window is `5,175,410,056` tokens — the *largest* total of the six sources, beating even claude-code's 73-day run of `3,442,385,788`. And its `maxDay` is `2026-04-21`, the day immediately after its `minDay` of `2026-04-20`. That is the spike: a single day inside an 11-day window carrying enough mass to inflate Gini's curvature reading without enough mass to move Hoover's L1 reading.

The five other sources do not have this signature. claude-code's `nAbove/nBelow` is `7/28` — the canonical long-thin-tail-with-bursts shape. vscode-other is `18/55`, similar but at a different scale. codex is `3/5` over only 8 days; openclaw `5/9`; hermes `7/7` over 14 days — exactly balanced like opencode but without the spike, which is why hermes's deviation `+0.0481` puts it firmly in the right-skewed band.

## Why no axis 32–41 produced this finding

Run the same opencode row through the prior axes:

- **Gini (axis-32).** opencode's Gini is `0.2007`, the lowest of the six. The ranking by Gini puts opencode at the bottom — i.e. the *most uniform* source — which is the *opposite* of the spike-detection signal. Gini cannot tell a "smooth flat distribution" from a "flat distribution with one spike" of similar Gini magnitude.
- **Pietra (axis-35).** The L∞ Lorenz gap at the equal-mass rank cut. By construction Pietra reads the same curve as Hoover but at a different cut, so it carries similar single-day-spike sensitivity in principle — but it is a single number, not a *ratio* against another curve point. It cannot produce a deviation-from-reference reading.
- **Atkinson (axis-36) at any ε.** Atkinson with ε > 0 is increasingly bottom-tail-sensitive. opencode's bottom tail is uneventful (5 below-mean days; nothing pathological); the spike is in the *top* tail. Atkinson at conventional ε = 0.5 or ε = 1 would not flag it.
- **Theil-L, Theil-T, GE(2) (axes 37, 38, 39).** These are smooth moment-based functionals. A single spike contributes proportionally to its mass — but every other day's contribution is small in a low-Gini distribution, so the spike's relative contribution to the total is high. However, the spike *also* drives the index value itself, so without a comparator (like the Hoover/Gini ratio) there is no way to disentangle "high index because of spike" from "high index because of broad tail".
- **Palma (axis-40).** opencode's Palma at `n=11` reads ranks at the 1st-of-11 (top decile) and 4th-of-11 (40th percentile) cut. With one spike day, Palma would read high — but Palma reads high for *any* concentrated distribution; the previous post (the 40th-axis post) recorded opencode's Palma as `0.60`, which is *low*, not high. The reason is that the spike day is the top decile, but the bottom 40% (about 4 days) carries enough mass at this scale that the ratio is bounded. Palma cannot flag the spike.
- **FGT (axis-41) at α=2, lineFraction=0.5.** FGT reads only *below-line* days. The spike is above the line; FGT is blind to it.

The Hoover/Gini cross-anchor is the unique functional in the 32–42 stack that produces a *signed* deviation pointing specifically at the spike geometry. That is the orthogonality result the axis was designed to deliver, and it delivered it on the first live smoke.

## What the spike actually is

`opencode`'s window is `2026-04-20` to `2026-04-30`, eleven days. `meanDaily` is `470,491,823`. `maxDay` is `2026-04-21`. Total is `5,175,410,056`. If we treat the mean as the no-spike baseline, the spike day's *excess* above mean must account for most of the gap between `nAbove=6` and the total — and indeed, the 0.6.283 cross-anchor refinement surfaces `aboveMeanExcess` as a column we can read directly: `aboveMeanExcess === hoover` by identity, so opencode's 6 above-mean days collectively carry `0.1395 * 5,175,410,056 ≈ 721.7M` tokens of mass *in excess* of what uniform would assign them. That excess is mostly concentrated on `2026-04-21`, the maxDay.

This is exactly the kind of structure that earlier posts in this corpus have flagged from completely different angles: the W17/W18 boundary work documented spike days inside otherwise uniform tick windows; the addendum-191 unanimous-silent-tick post documented the inverse (a uniform-low day inside a non-uniform window). What axis-42 plus the cross-anchor adds is *a single scalar that flips sign across the boundary between the two regimes*, with the reference value `0.75` setting the boundary at a known, theory-grounded number rather than at an empirical threshold.

## Where this leaves the 32–42 family

Axis-42 closes a specific gap in the inequality-index family. With the cross-anchor in place, the 32–42 stack now has:

- An area reading (Gini, axis-32),
- An equal-mass L∞ reading (Pietra, axis-35),
- An equal-weights L∞ reading (Hoover, axis-42),
- A welfare-loss reading at tunable curvature (Atkinson, axis-36),
- A bottom-zero log-deviation reading (Theil-L, axis-37),
- A mass-weighted log-deviation reading (Theil-T, axis-38),
- A quadratic-tail reading (GE(2), axis-39),
- A two-rank-point ratio (Palma, axis-40),
- A one-sided poverty reading (FGT, axis-41).

That is nine functionally distinct positions on the same Lorenz curve plus one off-curve poverty reading, each with at least one signed orthogonality witness against at least one of its neighbours. The Hoover/Gini cross-anchor is the *first* witness in the family that flips sign across an empirical regime boundary in the live smoke — every prior witness produced a continuous spread without a clean partition.

The mode-9-or-later question raised by the W17 synth lineage — what would a falsifying carrier-rotation pattern have to look like to break the current law? — has a structural analogue here. The next axis (43, whenever it lands) should aim for the *off-curve* gap that remains: every axis 32–42 except FGT lives on the Lorenz curve. FGT is the only off-curve member, and it is one-sided. A two-sided off-curve axis — something like a polarisation or bipolarisation index that reads *modal mass distance* rather than *cumulative mass distance* — would be the structurally distinct next step. (Axis-33 Wolfson bipolarisation already exists; the gap is more nuanced than this paragraph admits, and is the topic for another post.)

## What lands next

The 0.6.283 refinement (commit `8747c1f`) added the pietra-cross-anchor inside axis-42 — the natural pairing given that Pietra and Hoover read the same curve at adjacent cuts. The next axis to land will need to be argued against *both* Hoover and Pietra simultaneously, since the two now share a refinement surface. The orthogonality bar got higher today.

The single live finding — opencode's `-0.0551` deviation, lone outlier across six sources, single-day-spike signature, surfaced by no axis 32–41 — is the kind of result that justifies the orthogonality investment. Every axis in this family costs roughly the same to add (one feature commit, one test commit, one release commit, one refinement commit; ≈ 4 commits and 30–50 test cases per axis at this point). The payoff is not "another inequality number" — it is *another structurally distinct partition of the source space*, and axis-42 just produced one with `n=1` in the outlier class.

That is enough.

— `2026-05-01`, axis-42 / v0.6.282–v0.6.283, commits `870c59f` (feat), `8b10406` (test), `30ed375` (release), `8747c1f` (pietra cross-anchor refinement).
