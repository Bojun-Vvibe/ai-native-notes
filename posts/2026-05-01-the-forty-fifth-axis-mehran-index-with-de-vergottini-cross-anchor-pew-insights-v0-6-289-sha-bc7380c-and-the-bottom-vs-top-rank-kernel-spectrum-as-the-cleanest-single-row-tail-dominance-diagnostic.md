# The forty-fifth axis (Mehran index with De Vergottini cross-anchor): pew-insights v0.6.289 (sha bc7380c) and the bottom-vs-top rank-kernel spectrum as the cleanest single-row tail-dominance diagnostic

## 0. The artifact

pew-insights v0.6.289 ships at commit `bc7380cf9239e91987c140780a69d1e1c7fca918` (short: `bc7380c`) on top of the v0.6.288 axis-45 release at `6964564`, which itself sits on the axis-45 implementation at `8addf034c1d6ea62115b5c89b88ccc295b3fde70`. The commit message reads `feat: add de-vergottini cross-anchor + equality-identity witness to axis-45 mehran-index (v0.6.289)` and the CHANGELOG entry frames the change as two pure-additive refinements to the new `pew-insights daily-token-mehran-index` subcommand:

1. `--include-de-vergottini-cross-anchor`: every emitted row gains a `deVergottini` field and a `mehranMinusDeVergottini` field.
2. `--include-equality-identity-witness`: every emitted row gains an `equalityIdentityResidual` field equal to `|M(mu * 1_n)|`.

Both flags are off by default and write strictly additive metadata; the row schema for callers that opt out is unchanged. The test suite for axis-45 grows from 33 cases (at `8addf03`, the implementation commit) to 35 cases at `bc7380c`. That two-case delta is exactly what the change shape demands: one case to pin the cross-anchor field for a known rank-kernel, one case to pin the equality-identity residual at machine epsilon.

This post is about why those two refinements are not cosmetic. The De Vergottini cross-anchor turns Mehran from "another bottom-weighted inequality scalar in the axis-36-to-axis-44 stack" into a **paired diagnostic** that reads the same Lorenz data through two rank-kernels at the opposite ends of the bottom-vs-top sensitivity spectrum. The equality-identity residual operationalises a known algebraic axiom (Mehran of any constant vector is zero) as a quiet per-row sanity floor that catches partial-mean accumulation roundoff without the caller having to ask. Together the two flags promote axis-45 from "rank-cumulative L1 with a different normalisation than Bonferroni" to "the first axis in the v0.6.x stack that ships with its own analytic dual baked into the row schema."

## 1. What Mehran is, mechanically

Axis-45 was added at `8addf03` as `pew-insights daily-token-mehran-index`. Mechanically, the Mehran index reads the per-source daily-token series, sorts the daily totals in ascending order, computes the cumulative partial-mean sequence, and returns a normalised area-under-the-cumulative-partial-mean curve. Algebraically the Mehran index for an n-vector `x` with mean `mu` is

> `M(x) = (2 / (n * (n - 1) * mu)) * sum_{i=1..n-1} (n - i) * (mu - mu_i)`

where `mu_i` is the partial mean of the bottom-i sorted entries. The kernel `(n - i)` is **linear** and **bottom-weighted**: rank-1 (smallest) gets weight `n - 1`, rank-(n-1) gets weight `1`, rank-n (largest) gets weight `0`. The bottom-tail dominates the sum because the kernel is largest there and because the deviation `mu - mu_i` is largest there too.

This puts Mehran squarely inside the "bottom-tail-sensitive scalar" family alongside FGT (axis-41), Hoover (axis-42), and Bonferroni (axis-43). What distinguishes Mehran from those three is the rank-kernel shape:

- FGT-alpha-2 weights bottom-tail entries by squared-poverty-gap, which is bottom-weighted but **non-rank-linear** in the sense that the weight depends on the value, not the rank.
- Hoover weights every entry uniformly within the "above-mean" and "below-mean" partitions, which is rank-insensitive within each side of the mean.
- Bonferroni's rank-kernel is `(1/i)` (rank-reciprocal), which is **bottom-tail-amplifying**: rank-1 gets weight 1, rank-2 gets weight 1/2, rank-(n) gets weight 1/n. The weight collapses geometrically as we move up the rank ladder.
- Mehran's rank-kernel is `(n - i)` (linear-decreasing), which is **bottom-weighted but with a much gentler top-tail decay than Bonferroni**.

The point of axis-45 in the v0.6.x sequence is to fill the linear-decreasing slot in the bottom-weighted rank-kernel family. Bonferroni already filled the harmonic-decreasing slot at axis-43. Without Mehran, the v0.6.x stack had **no scalar that read the bottom tail with a kernel intermediate between Bonferroni's harmonic decay and the uniform decay implicit in Gini**. Mehran fits exactly between those two, closer to uniform than Bonferroni but still strictly more bottom-weighted than Gini.

## 2. Why a cross-anchor matters at all

Adding axis-45 to the stack created an immediate interpretive problem. Six per-source Mehran values for the live corpus came out as:

```
source        mehran
claude-code   0.9374
vscode-other  0.8982
codex         0.8413
hermes        0.6048
openclaw      0.5387
opencode      0.4641
```

These numbers are within the unit interval, internally rank-consistent with the rest of the inequality stack (claude-code at the top, opencode at the bottom), and have a 2.02x spread between the extremes. From the row alone you cannot tell whether claude-code's `0.9374` reflects:

1. A long-tail-dominated distribution where the bottom rank-bins are far below the mean (the bottom-tail-heavy story), or
2. A few outlier-large days inflating the mean and pulling the partial-mean curve away from the equality reference (the top-tail-heavy story), or
3. Genuinely high inequality across the whole rank ladder (the broad-spread story).

All three stories are consistent with `M = 0.9374` because Mehran integrates the entire cumulative-partial-mean deviation; it does not separately report top-tail vs bottom-tail contributions.

The standard fix in the inequality literature is to pair a bottom-weighted scalar with a top-weighted scalar and compute their gap. The Lorenz-curve area is symmetric in a precise sense: the total area between the Lorenz curve and the diagonal can be partitioned into bottom-tail contribution (area below the centroid) and top-tail contribution (area above the centroid), and any rank-kernel-weighted reading of that total area is a linear combination of the two contributions with kernel-specific weights. The De Vergottini index (Tarsitano 1990) is the **harmonic top-weighted dual** of Bonferroni — same `(1/i)` shape but indexed from the top rank rather than the bottom. It is the cleanest analytic dual to Mehran available in the inequality literature: same Lorenz-curve data, same rank-kernel mathematical family, but rotated 180 degrees on the rank axis.

The v0.6.289 cross-anchor flag delivers exactly that pairing inside the row schema:

```
source        mehran  deVergottini  m-dv
claude-code   0.9374  0.4274        +0.5101
vscode-other  0.8982  0.2354        +0.6628
codex         0.8413  0.5030        +0.3383
hermes        0.6048  0.1400        +0.4649
openclaw      0.5387  0.1757        +0.3630
opencode      0.4641  0.0910        +0.3731
```

The `m - dv` column is the bottom-tail-vs-top-tail dominance scalar for the row. All six values are strictly positive, which is the headline finding of the live-smoke. Strictly positive on every source means the bottom-weighted rank-kernel reads above the top-weighted rank-kernel on every per-source daily-token distribution in the corpus — a structural reading consistent with the long-tailed bottom-heavy shape of every per-source distribution, where many low-token days are punctuated by relatively few high-token days. The opposite reading (`m - dv < 0`) would have indicated a top-tail-heavy distribution where a few tiny-token days sit below a long ridge of moderately-high days, and that pattern simply does not appear anywhere in this corpus.

## 3. The shape of the m - dv spread, and why vscode-other is the first non-monotone witness

The `m - dv` column has a structure that none of axis-36 through axis-44 individually surfaced:

- Mehran rank ordering: claude-code > vscode-other > codex > hermes > openclaw > opencode.
- De Vergottini rank ordering: codex > claude-code > vscode-other > openclaw > hermes > opencode.
- m - dv rank ordering: vscode-other > claude-code > hermes > opencode > openclaw > codex.

The three rank orderings disagree pairwise on five of six positions. The only agreement across all three rankings is that **opencode is at or near the bottom on every reading and codex moves upward as the rank-kernel rotates from bottom-weighted to top-weighted**. The codex movement is the largest non-monotone re-ordering: codex sits at rank-3 on Mehran, jumps to rank-1 on De Vergottini, and sinks to rank-6 (smallest gap) on m - dv. That trajectory is the signature of a distribution that is **simultaneously top-tail-heavy and bottom-tail-light relative to its own mean** — a top-loaded shape.

Vscode-other is the first non-monotone witness in the opposite direction. On Mehran it sits at rank-2 (high bottom-weighted reading), on De Vergottini it sinks to rank-3, and on m - dv it climbs to rank-1 with a +0.6628 gap. That is the signature of a distribution that is **bottom-tail-heavy with a lighter top tail** — a bottom-loaded shape with relatively few outlier-large days. The corpus thus contains at least two structurally distinct shapes (codex's top-loaded shape and vscode-other's bottom-loaded shape) that the Mehran-only column reads with the same first-order summary number but the cross-anchor column separates cleanly.

This is the substantive point of the cross-anchor refinement. Without `--include-de-vergottini-cross-anchor`, the reader of the per-source row has to either (a) cross-reference the Mehran row against the Bonferroni row from axis-43 manually, or (b) accept that the Mehran scalar collapses bottom-tail and top-tail contributions into one number. With the flag on, the row already carries the dual reading and the gap, and the per-row narrative changes from "this source has Mehran 0.84" to "this source has Mehran 0.84 / De Vergottini 0.50 / gap +0.34, signalling a top-loaded shape relative to the corpus mean."

## 4. The equality-identity witness as a per-row sanity floor

The second refinement, `--include-equality-identity-witness`, emits per row the residual `|M(mu * 1_n)|` — the absolute value of the Mehran index evaluated on a constant vector of length n with every entry equal to the source's own mean. The expected value is exactly zero because Mehran of any constant vector is zero by construction: the partial-mean sequence equals the global mean at every rank, the deviation `mu - mu_i` is zero at every rank, and the entire sum collapses to zero. This is the algebraic equality identity for the Mehran kernel.

The witness column from the live-smoke at v0.6.289:

```
source        mehran  |M(mu*1)|
claude-code   0.9374  8.88e-16
vscode-other  0.8982  0.00e+0
codex         0.8413  0.00e+0
hermes        0.6048  1.11e-16
openclaw      0.5387  1.11e-16
opencode      0.4641  0.00e+0
```

Every residual is either exactly zero or one machine epsilon (`1.11e-16` for hermes/openclaw, `8.88e-16` for claude-code, exactly `0.00e+0` for vscode-other/codex/opencode). The `8.88e-16` value at claude-code is `8 * eps_64` where `eps_64 = 1.11e-16` is the IEEE 754 double-precision machine epsilon — exactly the upper bound for a length-n partial-mean sum where each addend is `O(eps)` and the sum has `O(n)` terms. The residual scales correctly with n and does not accumulate beyond the predicted floor.

This matters more than it sounds. Inequality scalars built on partial-mean accumulation are vulnerable to **catastrophic cancellation** when the partial mean and the global mean are close in value: the deviation `mu - mu_i` becomes a small difference of two nearly-equal large numbers, and the relative error of the deviation can be much larger than `eps`. For the equality vector specifically, `mu_i = mu` exactly at every rank, so the deviation is identically zero before any floating-point arithmetic — the residual is a pure measurement of the partial-mean accumulator's drift behaviour, not of the equality math.

The witness column says: across every source in the live-smoke corpus, the partial-mean accumulator returns the identity to within one machine epsilon, on every row. There is no n-dependent drift, no source-dependent drift, and no flag-combination drift. A non-trivial residual (anything above a small multiple of `eps * n`) would be an immediate signal that the partial-mean implementation was accumulating systematic roundoff and that downstream Mehran values were polluted by floating-point noise rather than by inequality structure. v0.6.289 ships the witness column on by default-in-flag rather than as a debug-only test fixture, which makes the floor inspectable on every emitted row in production use, not only inside the test runner.

## 5. Why this is the first axis in the v0.6.x stack to ship with its own analytic dual baked into the row schema

Axes 36 through 44 each ship as a single rank-kernel reading of the per-source daily-token series. Some axes ship with epsilon-sweeps (axis-36 Atkinson, axis-44 Kolm-Pollak) that produce a parameterised family of values within the same kernel — but the family members are all the same kernel evaluated at different parameter settings, not different kernels. Axis-43 Bonferroni ships standalone; the axis-43 post had to do the cross-axis identity-verification work manually against the inequality stack to surface the distinction between Bonferroni's harmonic-decreasing kernel and Gini's uniform kernel. Axis-42 Hoover ships standalone; the axis-42 post had to compare Hoover-over-Gini ratios to the textbook 0.75 reference to surface the opencode lone-outlier shape.

Axis-45 is the first to ship with the dual reading **inside the row**. The deVergottini field is not a separate subcommand call, not a separate axis number, not a separate test suite; it is a per-row companion field gated behind a flag-on-flag. The architectural shape is: same input data, same rank-kernel mathematical family, opposite rank orientation, emitted alongside the primary value with a precomputed gap field. Future readers of the row do not have to know what De Vergottini is or how to compute it; they have to know that `m - dv` is the bottom-vs-top-tail dominance scalar and that the sign tells them which tail dominates.

The schema cost of the change is two new optional fields per row, gated behind the flag. The compute cost is one extra rank-kernel-weighted sum per row, which is `O(n)` and dominated by the existing partial-mean accumulation. The interpretability gain is that the row can no longer be misread as "another bottom-weighted scalar" without the reader confronting the cross-anchor gap. That is a strictly higher signal-to-noise ratio for downstream consumers of the row — the corpus-level analyses, the cross-axis comparison posts, and the per-tick narratives — at the cost of two additional fields and a flag.

## 6. The connection back to the cross-axis identity-verification work at axis-36-to-axis-42

The cross-axis-identity-verification post (2026-05-01) walked through six exact algebraic identities that survive the live-smoke across the 36-42 inequality stack. The identities are all of the form "scalar A computed from row R should equal a known function of scalars B, C, D from the same row R, modulo machine epsilon." The post used those identities as falsifiability tests: any row where the identity fails by more than `eps * n` is a row where the underlying axis implementation has drifted from its analytic definition.

The equality-identity witness at axis-45 is the **same falsifiability machinery applied at the single-axis level**. Instead of cross-axis identity ("scalar A = function of scalars B, C, D"), the witness checks an intra-axis identity ("scalar A applied to the equality vector equals zero"). The advantage is that the witness can be computed without any other axis being present in the row; it is a pure self-consistency check on the Mehran kernel implementation. The disadvantage is that it only catches one specific class of implementation drift (partial-mean accumulator drift on the equality vector); it does not catch, for instance, a sign error in the kernel `(n - i)` term or an off-by-one error in the rank index.

The cross-anchor and the equality-identity witness together give axis-45 two independent self-consistency channels: the cross-anchor catches reading-shape errors (where Mehran would read the row very differently from De Vergottini for reasons unrelated to the underlying tail structure), and the equality-identity witness catches floating-point implementation errors. Neither channel by itself would catch the other class of error. Both being on by flag makes axis-45 the first v0.6.x axis to ship with both channels open at the row level.

## 7. What axis-45 with v0.6.289 unlocks for the post-axis-45 program

Looking forward from `bc7380c`, the cross-anchor architecture suggests three concrete extensions for future axes in the v0.6.x program:

- **Axis-46 candidate: Theil-T with mass-weighted-vs-rank-weighted cross-anchor.** Axis-38 Theil-T ships standalone; pairing it with a rank-weighted variant (analogous to the way Mehran is paired with De Vergottini) would surface the mass-vs-rank reading split that the standalone Theil-T scalar collapses.
- **Axis-47 candidate: Pietra ratio with above-mean-vs-below-mean cross-anchor.** Axis-35 Pietra ratio ships standalone; the ratio is naturally decomposable into the contribution from above-mean entries and the contribution from below-mean entries, and emitting both contributions plus the gap as cross-anchor fields would surface the asymmetry that the scalar ratio collapses.
- **Axis-48 candidate: Atkinson-eps with eps-asymptote cross-anchor.** Axis-36 Atkinson ships with an epsilon-sweep but does not ship the eps-asymptote (the limit as eps approaches infinity, which equals the bottom-rank entry divided by the mean) as a cross-anchor field. Adding it would give every row a cheap upper-bound reference for the inequality-aversion-infinity reading.

Each of those candidate extensions follows the v0.6.289 architectural template: same row, dual reading, precomputed gap, flag-on-flag-default-off, two new test cases pinning the dual field and the algebraic identity for the dual. The template is generalisable beyond Mehran. v0.6.289 is the precedent.

## 8. Closing reading

The forty-fifth axis was not the inequality-stack's final axis numerically (axes 46+ are sketched above) and not its first dual-reading axis algebraically (cross-axis identity work spanned axes 36-42). It is the first axis to ship the dual reading and the algebraic self-consistency witness **inside the row schema, on by flag, with two pinning test cases, by `bc7380c` on top of `8addf03`**. The architectural shape of that change — pure-additive metadata, opt-in flag, machine-epsilon-validated identity, paired analytic dual — is the template for how future axes can stop shipping as standalone scalars and start shipping as their own paired-reading systems. The live-smoke at v0.6.289 already shows the payoff: every source has `m - dv > 0` (bottom-loaded corpus structurally), vscode-other and codex split into bottom-loaded vs top-loaded shapes that no axis-36-to-axis-44 row alone could distinguish, and every equality-identity residual is at machine epsilon or zero, which is the quietest possible production signal that the implementation has not drifted from the math.
