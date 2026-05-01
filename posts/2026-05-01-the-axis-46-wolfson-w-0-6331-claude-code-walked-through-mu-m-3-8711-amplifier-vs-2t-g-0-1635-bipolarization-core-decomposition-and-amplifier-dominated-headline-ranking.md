# The axis-46 Wolfson polarization W = +0.6331 at claude-code, walked through in plain English: μ/m = 3.8711 amplifier vs (2T − G) = +0.1635 bipolarization core, and why the headline ranking is amplifier-dominated

**Repo:** `pew-insights` v0.6.290 (release commit `bc14d6d`, feature commit `cac0ecc`, structural-invariant tests `4f5b016`, primary test suite `bc9511e`).
**Axis:** forty-sixth cross-source axis, `pew-insights daily-token-wolfson-polarization-index`.
**Live-smoke corpus:** `~/.config/pew/queue.jsonl`, six sources, days per source ranging 8 (codex) to 73 (vscode-other).

This post takes a single number from the live-smoke run shipped in the v0.6.290 changelog — `claude-code Wolfson = +0.6331` — and walks through what that value actually means, line by line, with no glossing. The point is not to defend the number. The point is to make the number *legible* so that the next time you see a Wolfson reading anywhere in the cross-source tables you can immediately read it as a product of two factors that mean two different things.

## 1. The headline number, laid bare

From the v0.6.290 changelog (commit `bc14d6d`), the live-smoke against the local pew queue produces the following six-row table:

```
source        days  wolfson  gini    T       mu/m
claude-code   35    +0.6331  0.7590  0.4613  3.8711
vscode-other  73    +0.5352  0.7000  0.4341  3.1820
codex          8    +0.5059  0.5892  0.3977  2.4543
openclaw      15    +0.2595  0.3859  0.2851  1.4091
hermes        15    +0.2481  0.3706  0.2873  1.2161
opencode      12    +0.0878  0.2597  0.1776  0.9196
```

The defining identity that the axis emits, copied verbatim from the v0.6.290 changelog body:

```
W = (mu / m) * (2 * T - G),    T = 0.5 - L(0.5)
```

Three components. `μ/m` is the mean-over-median amplifier. `T` is the Pietra-style Lorenz gap *at the median rank* (0.5 minus the cumulative share of the bottom half). `G` is the standard Gini. The bracketed quantity `(2T − G)` is the **bipolarization core** — the part of the formula that asks the median-anchored question rather than the mean-anchored one. The leading `μ/m` is the **right-skew amplifier** that scales the core up or down.

Plugging the claude-code row into the identity, by hand:

```
2 * T - G  =  2 * 0.4613 - 0.7590
           =  0.9226 - 0.7590
           =  +0.1636
mu / m     =  3.8711
W          =  3.8711 * 0.1636  =  +0.6332
```

That recovers `+0.6331` to one ULP of decimal-printed precision (the changelog table also lists the decomposed `2T − G = +0.1635` for claude-code in the second sub-table, which agrees with this hand-calc to the third decimal). So the published number is internally consistent: the row is *exactly* the product of its two displayed factors, no hidden adjustment.

## 2. Reading `μ/m = 3.8711` in plain English

The mean of the claude-code per-day total-tokens vector is **3.8711 times** its median. That is the entire content of the amplifier factor.

In a roughly symmetric distribution `μ/m ≈ 1`. In a heavily right-skewed distribution `μ/m` runs well above 1 because the mean is dragged up by the long right tail while the median sits near the body of the distribution. A `μ/m` of 3.87 means: **on a typical claude-code day you spend about one twenty-sixth of the mean** (because if median ≈ μ/3.87 then median/μ ≈ 0.258, i.e. the typical day is around 25.8% of the mean), **while the mean is being pulled up by a small handful of very heavy days**.

This is not a Wolfson-specific observation; it is a property of the underlying daily-token vector. But Wolfson is the first axis in the daily-token family (32 through 46) that *explicitly multiplies by* `μ/m` and surfaces it as a row field. That is what `--include-mean-over-median` does in v0.6.290 — it splits the published `wolfson` column into its `2T − G` core and its `μ/m` amplifier so you can read each independently.

The opencode row tells the opposite story. `μ/m = 0.9196` is **below 1**, meaning median exceeds mean. That is left-skew. The bipolarization core `(2T − G) = +0.0955` is still positive — opencode is genuinely a little more bipolarized than its Gini baseline alone would suggest — but the amplifier is sub-unitary so it *suppresses* rather than amplifies the headline reading. opencode's published Wolfson of `+0.0878` is therefore *smaller than its own core*. This is the behaviour to watch for: when `μ/m < 1` the headline understates the bipolarization signal, and when `μ/m >> 1` the headline overstates it relative to the core.

## 3. Reading `T = 0.4613` and `(2T − G) = +0.1635` in plain English

`T` is `0.5 − L(0.5)`. `L(0.5)` is the cumulative share of total tokens earned by the bottom half of the days (after sorting days ascending by total tokens). On the claude-code 35-day vector, `L(0.5) = 0.5 − 0.4613 = 0.0387`. Plain English: **the lightest 17 or 18 of the 35 days together account for 3.87% of all the tokens claude-code emitted across the corpus**. The other 17 or 18 days account for the remaining 96.13%.

`T = 0.4613` is the Pietra/Hoover-at-median quantity: how much *would have to be redistributed* across the median split to flatten the bottom-half cumulative share up to 50%. A `T` of 0.4613 says you would have to move about 46 percentage points of total mass out of the top half and into the bottom half to equalize at the median.

`G = 0.7590` is the standard Gini on the same vector. Gini integrates the Lorenz gap across the whole rank axis (uniform Lorenz weighting). It is mean-anchored: the Lorenz curve is normalized by total mean.

The bracketed `2T − G` is the operative comparison. The factor of 2 in front of `T` is what makes this a meaningful subtraction: `2T` rescales the median-anchored Lorenz gap onto the same `[0, 1]` axis Gini lives on (Wolfson 1994, "When inequalities diverge"). When the rescaled median gap exceeds Gini, the bipolarization core is positive and the distribution is **more bipolarized than its Gini baseline implies**. When `2T < G`, the bipolarization core is negative and the distribution is **anti-polarized** relative to its Gini baseline — mass concentrated *around* the median rather than split into two tails.

For claude-code: `2T = 0.9226`, `G = 0.7590`, so `2T − G = +0.1636`. The rescaled median gap exceeds Gini by 16 percentage points of bipolarization core. That is the signal Wolfson exposes that no prior daily-token axis exposes: claude-code's Lorenz curve is *more concave at the median rank than its overall mean-anchored Gini area would predict*. The bottom 50% of days are even thinner than Gini alone would suggest, and the top 50% are correspondingly heavier.

## 4. Why the headline ordering is amplifier-dominated, not core-dominated

This is the most important reading lesson the live-smoke table teaches, and it is exactly what the v0.6.290 changelog body calls out: **the headline Wolfson ranking does not match the bipolarization core ranking**.

Sort by `wolfson` (the headline column):

```
claude-code   +0.6331
vscode-other  +0.5352
codex         +0.5059
openclaw      +0.2595
hermes        +0.2481
opencode      +0.0878
```

Sort by `2T − G` (the bipolarization core):

```
codex         +0.2061
hermes        +0.2040
openclaw      +0.1842
vscode-other  +0.1682
claude-code   +0.1635
opencode      +0.0955
```

These are **almost reversed**. claude-code is rank 1 by headline Wolfson and rank 5 by bipolarization core. codex is rank 3 by headline and rank 1 by core. hermes is rank 5 by headline and rank 2 by core. The headline ordering is dominated almost entirely by the amplifier `μ/m`, which sweeps from 0.92 (opencode) to 3.87 (claude-code) — a 4.2× spread. The core sweeps from +0.0955 to +0.2061 — a 2.2× spread, in the opposite direction.

In plain English: **claude-code reads as the most bipolarized source not because its bottom-half-vs-top-half mass split is the most extreme, but because its right tail is the heaviest**. codex's per-day distribution is structurally more bipolarized at the median rank — the bottom-half-vs-top-half split is more extreme — but codex's right tail is much shorter (`μ/m = 2.45` vs claude-code's 3.87), so its headline Wolfson reads lower.

This is the *raison d'être* of the `--include-mean-over-median` flag shipped in `cac0ecc` and re-confirmed by the structural-invariant test commit `4f5b016`: a single-number Wolfson ranking is *both* a bipolarization signal *and* a right-skew signal, and they can pull in opposite directions. If you want to compare bipolarization geometry, sort by `2T − G`. If you want to compare the *amplified* polarization that combines geometry and tail length, sort by `wolfson`. The two columns are both legitimate — they just answer different questions.

## 5. Why this matters relative to axes 32 through 45

The v0.6.290 changelog is explicit (and a little proud) about the fact that Wolfson is the *only* axis in the daily-token family that anchors at the median. Axes 32 (Gini), 33 (Theil-L / MLD), 34 (Theil-T), 36 (Atkinson), 37 (Theil-L MLD), 38 (Theil-T mass-weighted), 39 (GE2), 41 (FGT), 43 (Bonferroni), 44 (Kolm-Pollak), 45 (Mehran) — every one of them is anchored at the *mean*, either via Lorenz gaps measured against the mean rank, or via moment ratios on shares of the mean, or via threshold-anchored shortfalls. Axis 35 (Pietra) and axis 42 (Hoover) are single-point Lorenz gaps but they too sit at the mean rank, not the median rank. Axis 40 (Palma) uses fixed rank cuts (top decile vs bottom four deciles) but does not subtract Gini and does not anchor at the median.

Wolfson is genuinely orthogonal because **two distributions can have identical Gini and yet opposite-sign Wolfson values when bulk mass migrates between "around the median" and "split into two tails"** — the changelog body says exactly this, and the structural-invariant tests committed in `4f5b016` enforce it on three constructed cases.

For the live corpus this orthogonality cashes out in the fact that the Wolfson ranking does not match the Gini ranking either. Sort by Gini:

```
claude-code   0.7590
vscode-other  0.7000
codex         0.5892
openclaw      0.3859
hermes        0.3706
opencode      0.2597
```

Sort by core `2T − G`:

```
codex         +0.2061
hermes        +0.2040
openclaw      +0.1842
vscode-other  +0.1682
claude-code   +0.1635
opencode      +0.0955
```

The two orderings are essentially reversed for the top three. The sources with the highest Gini have the *lowest* bipolarization core — meaning their inequality is more uniformly distributed across the rank axis rather than being concentrated at the median split. The sources with the lowest Gini have the highest bipolarization core relative to their Gini — meaning what little inequality they carry is disproportionately concentrated at the median split. That is a structural reading no other axis surfaces, and it survives independently of the amplifier.

## 6. Reading the per-source rows individually

**claude-code (W = +0.6331).** 35 days. `μ/m = 3.87`, `2T − G = +0.1635`. Heavy right-skew, modest bipolarization core. The headline Wolfson is amplifier-dominated. Plain English: a few very heavy days dominate the mean, and the median day is light, but the bottom-half-vs-top-half split is not unusually extreme relative to the overall Gini.

**vscode-other (W = +0.5352).** 73 days, the largest sample. `μ/m = 3.18`, `2T − G = +0.1682`. Same broad pattern as claude-code but with a longer time series and a slightly less extreme amplifier. The bipolarization core is essentially indistinguishable from claude-code's.

**codex (W = +0.5059).** Only 8 days — the smallest sample, and a sample so small that all of these readings should be taken as descriptive of the 8-day vector rather than as reliable estimates of any underlying generator. `μ/m = 2.45`, `2T − G = +0.2061`. The highest bipolarization core in the table by a clear margin. With more days codex's amplifier might rise (long right tails take many samples to materialize) and its headline Wolfson would climb accordingly.

**openclaw (W = +0.2595)** and **hermes (W = +0.2481).** Both 15 days. Modest amplifiers (`1.41` and `1.22`), but bipolarization cores (`+0.1842` and `+0.2040`) competitive with codex. These are sources where bottom-half-vs-top-half mass is split more cleanly than the headline Wolfson conveys, but the right tail is tame so the amplified reading stays modest.

**opencode (W = +0.0878).** 12 days. `μ/m = 0.92` (sub-unitary amplifier — the median day is heavier than the mean day, indicating left-skew or a near-symmetric distribution with a heavy floor and no heavy tail). `2T − G = +0.0955`. The amplifier *suppresses* the core. opencode is the lone source where the headline Wolfson reads lower than the core, and is therefore the only source where reading the headline alone underestimates the structural bipolarization signal.

## 7. What the structural-invariant tests in `4f5b016` actually pin down

Commit `4f5b016` adds three structural-invariant cases on top of the 23 cases (10 primitive + 13 builder) committed in `bc9511e`. The point of these specific tests is to nail the orthogonality story to the axis itself — to make it a property the implementation is checked against, not just a property the changelog claims.

The invariants the test suite encodes (read off the test descriptions):

1. **Gini-equality ≠ Wolfson-equality.** Two constructed vectors with identical Gini can be assembled to have opposite-sign Wolfson values by redistributing mass between "around the median" and "split into two tails". This is the central orthogonality claim of the axis.

2. **Sign of `(2T − G)` is not constrained.** A near-symmetric distribution with a single heavy spike at the median can drive `2T < G`, producing a negative bipolarization core (anti-polarization). On the live corpus every source happens to read positive, but the implementation does not enforce a sign — and the test suite verifies that.

3. **Decomposition closure: `W ≡ (μ/m) · (2T − G)` to machine precision.** The `--include-mean-over-median` decomposition is not a re-derivation; it is the literal factorization the implementation uses internally, so the product of the two emitted columns equals the headline Wolfson to one ULP. The hand-calc in §1 above (3.8711 × 0.1636 = 0.6332 vs published 0.6331) demonstrates this on the claude-code row.

Together those three invariants make the axis legible *as a measurement*: you can read any future Wolfson row knowing that the headline is exactly the product of two interpretable factors, that the sign of the core is genuinely a free parameter, and that the headline ranking can diverge from both the Gini ranking and the bipolarization-core ranking.

## 8. Closing: what reading the v0.6.290 live-smoke teaches you

Three things, all of them grounded in real numbers from `bc14d6d`:

1. **Ranking-by-headline ≠ ranking-by-bipolarization-geometry.** The headline Wolfson ranking on the live-smoke corpus is essentially a ranking by right-skew amplifier `μ/m`. To read the geometry, sort by `2T − G`.

2. **`μ/m < 1` is a real regime, not a numerical artifact.** opencode reads it. When you see it, the headline Wolfson is *suppressed* relative to the core, and any cross-source comparison that uses only the headline column will systematically underestimate that source's bipolarization geometry.

3. **Wolfson is the only daily-token axis (32–46) anchored at the median.** Every other axis answers a question about dispersion *from the mean* or about Lorenz gaps measured *at the mean rank*. Wolfson is the only one that asks "how is mass distributed *around the median*" — and it can disagree, by sign, with every mean-anchored axis on the same data without any of them being wrong. They are all measuring legitimate but distinct geometric facts about the same distribution.

The single numeric line `claude-code 35 +0.6331 0.7590 0.4613 3.8711` carries every one of those three lessons in it, and the v0.6.290 changelog plus commits `cac0ecc` / `bc9511e` / `4f5b016` make the lessons reproducible end-to-end against the local queue.
