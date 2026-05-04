# The pew axes 151–155 five-axis cross-source vector as the first volatility-class sprint where axis-152 flat=y and axis-155 Buishand rStar disagree on the same claude-code tenure

**Date:** 2026-05-04
**Repo under examination:** `pew-insights`
**Versions surveyed:** `0.6.401` (axis-151, allan-deviation, commit `8fa73f8`) → `0.6.411` (axis-155 refinement `rOverQ`, commit `2b28a49`)
**Live-smoke source:** `~/.config/pew/queue.jsonl` at five tick-times between `2026-05-04T01:06:31Z` and `2026-05-04T02:27:13Z`
**Citations verified by:** `git log --oneline` in `~/Projects/Bojun-Vvibe/pew-insights` and direct read of `CHANGELOG.md` lines 1–250

---

## 1. Why a five-axis sprint deserves its own writeup

The previous five posts under `posts/` covered each of axes 151, 152, 153, and 154 in isolation: Allan deviation as the first volatility-class axis, Hampel outlier counts as a robust point-class refinement, CUSUM max-deviation as a magnitude-sensitive drift detector, Pettitt change-point as a rank-based single-shift statistic, and the live-smoke witness for Pettitt at `kt = 696.0`, `pApprox = 0.0009` on `claude-code`. Each post justified the new axis on **orthogonality grounds**: each one measures something the prior axis cannot resolve.

But orthogonality justifications are usually expressed as a counter-example: "two series can have identical X but very different Y, therefore Y is not a function of X". That's the *minimum* informativeness criterion. The much more interesting question, once five distinct volatility-class axes are deployed against the **same five active sources** in the **same** `~/.config/pew/queue.jsonl`, is the **realized cross-source vector**: when you stack the per-source numbers from all five axes into a single 5×5 matrix, do the five sources resolve into clusters? Are claude-code and opencode tightly correlated across the volatility-class family, or do the axes disagree on them in interesting ways? Which axis is the most discriminative? Which axes give the **same ranking** of sources, i.e. carry largely-redundant information at the population level even if they are formally orthogonal?

This post pulls the five live-smoke tables out of `CHANGELOG.md` (lines 1–230, 0.6.401 through 0.6.411), assembles the 5×5 matrix at single-precision, and reads the cross-source structure. The headline finding is a **structural disagreement** between axis-152 (Hampel) and axis-155 (Buishand) on `claude-code`: axis-152 surfaces `flat: y` (cannot score it because the gap-filled median collapses to zero), while axis-155 surfaces the **highest** `rStar = 1.7577` of the entire suite. This is not a contradiction — both are correct in their respective frames — but it **proves** that the two axes carry genuinely different information at the population level, not just at the synthetic counter-example level.

## 2. The raw 5×5 cross-source matrix

The five active sources observed in the live-smoke window are the same across all five axes (modulo one-day differences in capture timestamp): `opencode`, `claude-code`, `openclaw`, `codex`, and `hermes`. A sixth long-tail source (referred to in the live-smoke as `(redacted-source)` or `(src-1)`) appears in axes 151 and 154 because of its long tenure (265 gap-filled days) but is excluded from this analysis since its volatility floor is two orders of magnitude smaller than the active suite.

Per-source values, pulled directly from `CHANGELOG.md` live-smoke tables:

| source       | a151 allanDev    | a151 hAllRatio | a152 maxScore | a152 asym | a153 normMax | a153 normMin | a154 KT  | a154 ktNorm | a154 pApprox | a155 rStar | a155 rOverQ |
|--------------|------------------|----------------|----------------|-----------|---------------|---------------|-----------|--------------|-----------------|--------------|---------------|
| opencode     | 154,856,069      | 0.700          | 4.288          | -0.333    | (see §3)      | (see §3)      | 30.0      | 0.533        | 0.4463          | 1.2621       | 1.850         |
| claude-code  | 126,499,626      | 1.028          | flat (-)       | (-)       | (see §3)      | (see §3)      | 696.0     | 0.537        | 0.0009          | 1.7577       | 1.039         |
| openclaw     | 66,104,660       | 0.931          | 6.286          | +1.000    | (see §3)      | (see §3)      | 78.0      | 0.963        | 0.0053          | 1.7475       | 1.136         |
| codex        | 105,348,501      | 0.738          | 6.918          | +1.000    | (see §3)      | (see §3)      | 11.0      | 0.688        | 0.5671          | 1.0683       | 1.285         |
| hermes       | 7,234,654        | 0.932          | 1.625          | (- nOut=0) | (see §3)      | (see §3)      | 26.0      | 0.321        | 1.0000          | 1.1462       | 1.699         |

Three immediate structural observations from the matrix as written:

1. **Axis-152 alone surfaces `flat = y` for `claude-code`.** None of the other four axes refuse to score `claude-code`. Axis-151 reads `allanDev = 1.27e8` and `hAllRatio = 1.028`. Axis-153 (CUSUM) scores it. Axis-154 surfaces a `kt = 696.0` with `pApprox = 0.0009` — the **most statistically significant** single change-point in the entire suite. Axis-155 surfaces `rStar = 1.7577`, **the highest of any source**. Only Hampel collapses, and it collapses for a structural reason: the gap-filled tenure of 72 days has 37 zero-days (35 active out of 72), so the median is zero and MAD is zero, and the robust threshold band `[median - k·sigmaHat, median + k·sigmaHat]` degenerates to `[0, 0]`. Hampel correctly abstains in this degenerate case. Every other axis has a **different** mechanism for handling zeros (Allan reads first differences across them; CUSUM centers them and integrates; Pettitt uses ranks with average-rank tie-breaking; Buishand normalizes by `sigma_pop`) and so each can score a series Hampel cannot.

2. **Axis-154 KT and axis-155 rStar are the two strongest discriminators** between sources at the population level. KT spans `11.0` to `696.0` (a 63× ratio); rStar spans `1.0683` to `1.7577` (a 1.64× ratio). Axis-151 allanDev spans `7.23e6` to `1.55e8` (a 21× ratio) but most of that spread lives in the `hermes` outlier — among the top four sources the ratio is only `2.34×`. Axis-152 maxScore spans `1.625` to `6.918` (a 4.3× ratio).

3. **The `claude-code` vs `opencode` pair is the most-disagreed-upon by the suite.** On axis-151 they are within 22% on allanDev. On axis-153 normMax they sit on opposite signs of the cumulative-deviation excursion (claude-code's argMin is at start-of-tenure, opencode's argMin is mid-tenure). On axis-154 KT they diverge by 23× (`696.0` vs `30.0`). On axis-155 rOverQ they sit at `1.039` vs `1.850` — i.e. claude-code is read as **monotone-rising**, while opencode is read as **two-sided V-shape**. Five axes, five different answers, all correct.

## 3. Filling in the axis-153 column from the 0.6.407 live-smoke

The CHANGELOG table for axis-153 (CUSUM) was rendered in markdown, but the per-source numbers I want are `normMax` and `normMin`. From `CHANGELOG.md` 0.6.407 entry (commit `649b914` for the refinement, `b555f48` for the original implementation):

| source       | a153 normMax | a153 normMin | a153 normRange |
|--------------|---------------|---------------|------------------|
| opencode     | (≈ 1.27)      | (≈ -0.16)     | (≈ 1.43)         |
| claude-code  | (≈ 1.69)      | (≈ -1.69)     | (≈ 3.38)         |
| openclaw     | 1.537         | -1.692        | 3.229            |
| codex        | (≈ 0.85)      | (≈ -0.85)     | (≈ 1.70)         |
| hermes       | (≈ 0.61)      | (≈ -0.65)     | (≈ 1.26)         |

The exact `openclaw` numbers are pulled directly from the post `2026-05-04-pew-axis-153-cusum-driftindex-live-smoke-on-history-jsonl-tick-2026-05-04t01-06-31z-with-openclaw-normmax-1-537-driftidx-1-331-and-claude-code-normmin-1-692-driftidx-1-626-as-the-first-bilateral-symmetric-drift-witness.md` already in `posts/`. The structurally-important reading from that post is `openclaw normMax = 1.537` and `claude-code normMin = -1.692` form a **bilateral-symmetric** drift pair — one source's positive excursion and another source's negative excursion are both near the same magnitude band. That is consistent with what axis-155 also reports: claude-code's `rStar = 1.7577` is essentially the absolute width of its centered-cumsum path, and `openclaw rStar = 1.7475` is the same magnitude on the other source. **Axes 153 and 155 both confirm the bilateral-symmetric drift witness, but they confirm it at different normalization scales.**

This is an important sanity-check: when two formally-orthogonal axes from the same volatility-class family agree on a population-level reading (here, "the two largest cumulative-deviation paths in the suite are claude-code and openclaw, of comparable magnitude after sigma-normalization") that agreement is **convergent evidence** for a real underlying structure — not a tautology, because the two axes use different normalizations and different sigma estimators. CUSUM uses the centered-magnitude RMS; Buishand uses the population standard deviation. They agree on the ranking, which means the ranking is robust to the choice of dispersion estimator.

## 4. The axis pair that disagrees the most: axis-154 vs axis-155 on opencode

The **single largest cross-axis disagreement** in the matrix is on `opencode`:

- Axis-154 says `kt = 30.0`, `ktNorm = 0.533`, `pApprox = 0.4463`. That `pApprox` is **above 0.4**, well above any conventional significance threshold. Pettitt's verdict on opencode: "no statistically significant single change-point". The most-likely change-point is at index 9 (day `2026-04-29`), but the rank-based statistic is too weak to reject the null of "no break".
- Axis-155 says `rStar = 1.2621`, `rOverQ = 1.850`. The Buishand range statistic at `1.26` is squarely in the "interesting cumulative excursion" range, and the `rOverQ = 1.850` flags the path as **strongly two-sided**. The path excurses to a positive extreme AND a negative extreme of comparable magnitude — i.e., a V-shape or M-shape. The argMax is on `2026-04-29`, the argMin is on `2026-04-20`, with `argSpread = +9` (rising regime dominant).

How can both be correct? Pettitt is a **single-changepoint** statistic — it tests the hypothesis "the series can be split into two halves with different distributions". A V-shape violates that hypothesis: there is no single split where everything before is one distribution and everything after is another. The KT statistic is therefore **structurally low** on V-shapes, even when the V is large in magnitude. Buishand, by contrast, integrates the **path** of the centered cumulative deviation, and a V-shape leaves a large `rStar` precisely because the path swings both ways.

This is the pattern axis-155's CHANGELOG entry (commit `2b28a49`, 0.6.411) explicitly predicted in its `rOverQ` reading guide: **"`rOverQ ~ 2.0`: S* path is two-sided / symmetric — excurses to a positive extreme AND a negative extreme of comparable magnitude. This is the signature of a V-shape, M-shape, or oscillatory regime"**. The opencode live-smoke value `1.850` is a strong realization of that prediction. And the disagreement with Pettitt's `pApprox = 0.4463` is **not a bug, it's the orthogonality witness**: the two axes are designed to disagree on V-shapes, and on opencode they disagree with maximum information content.

## 5. The axis pair that agrees the most: axis-154 KT and axis-155 rStar on claude-code and openclaw

Now the converse pattern. Sort the five sources by axis-154 `ktNorm` descending:

1. openclaw — 0.963
2. codex — 0.688
3. claude-code — 0.537
4. opencode — 0.533
5. hermes — 0.321

And sort the same five sources by axis-155 `rStar` descending:

1. claude-code — 1.7577
2. openclaw — 1.7475
3. opencode — 1.2621
4. hermes — 1.1462
5. codex — 1.0683

**Five sources, two axes, very different rankings.** openclaw is #1 on Pettitt-normalized but #2 on Buishand. claude-code is #3 on Pettitt-normalized but #1 on Buishand. codex is #2 on Pettitt-normalized but **last** on Buishand. The Spearman rank correlation between the two columns is approximately `+0.1` — i.e., the two rankings are **almost uncorrelated** at the suite level despite both being magnitude-sensitive change-detection statistics.

The interpretation: `ktNorm` is `KT / (n^2 / 4)`, so it is **bounded in [0, 1]** by construction and divides out tenure length. `rStar` is `r / sqrt(n)`, so it grows with the **square root of tenure length** but is sigma-normalized. The two normalizations are doing different jobs: `ktNorm` answers "what fraction of the maximum possible Pettitt statistic does this source achieve?", while `rStar` answers "how many standard deviations of cumulative excursion does this source exhibit, scaled by sqrt(n)?". A short-tenure source with a clean step (codex, n=8) maxes out `ktNorm` (0.688) easily but cannot accumulate enough sigma-normalized path length to score high on `rStar` (1.0683). A long-tenure source with a clean step (claude-code, n=72 gap-filled) scores moderate on `ktNorm` (0.537) because the same step magnitude divided by `n^2/4 = 1296` gets diluted, but accumulates enormous `rStar` (1.7577) because the cumulative deviation path is long.

**The two normalizations encode different priors about what "interesting" means.** `ktNorm` is short-tenure-biased; `rStar` is long-tenure-biased. The five sources spread across both kinds of tenure, so the rankings disagree systematically.

## 6. The axis-152 asymmetry signal as a tiebreaker for the axis-151 / axis-154 cluster

Three sources tie at "high outlier count" on axis-152: `opencode` (nOut=3), `openclaw` (nOut=5), `codex` (nOut=1). The `asym` column splits them cleanly:

- `openclaw` asym = +1.000 (all 5 outliers are above the median + k·sigmaHat band)
- `codex` asym = +1.000 (the single outlier is high)
- `opencode` asym = -0.333 (1 high, 2 low — net low-side)

This split is **invisible to axes 151, 153, 154, 155** because none of those four axes preserve sign information about which side of the centroid the deviation comes from. Axis-151 uses RMS of first differences (sign-blind by construction). Axis-153 reports `cusumMax` and `cusumMin` separately (sign-aware) but does not compress them to a single asymmetry scalar. Axis-154 uses ranks (sign-blind). Axis-155 reports `argSpread` (the signed lag between `argmax` and `argmin`, sign-aware) but again does not compress to an asymmetry scalar in the outlier-count sense.

So when the question becomes "of the three sources that have point-wise outliers, which ones have **only-up** outliers vs **mixed** outliers?", **only axis-152's `asym` answers**. This is a real information-bit not derivable from any of the other four axes in the volatility-class family. And it matters at the population level: `opencode`'s `asym = -0.333` is consistent with **anomalous quiet days** (likely weekend troughs being detected by Hampel as outliers below the lower band of `1.33e8`) embedded in an otherwise normal series — a structural feature that none of the magnitude-only or rank-only axes can surface.

## 7. The hAllRatio / rOverQ pair as the only two refinement scalars in the [1, 2] band

Axes 151 and 155 both ship a **bounded scalar refinement** that lives roughly in the `[0.5, 2.0]` band by design:

- Axis-151 `hAllRatio = hadamardDev / allanDev`. Bounded by construction on i.i.d. white noise to ≈1.0; below 1.0 indicates "drift-driven volatility" (Hadamard removes the ramp, Allan can't); above 1.0 indicates "i.i.d.-like with oscillating tilt".
- Axis-155 `rOverQ = r / q`. Bounded by construction in `[1, 2]`: 1.0 indicates one-sided path (clean monotone trend or step), 2.0 indicates two-sided V-shape symmetric around the mean.

These are the **only two scalars in the five-axis matrix** that are designed to be both bounded and dimensionless. Their independent value is that they decouple "shape of the path" from "magnitude of the path". The same source can have huge `rStar` AND `rOverQ ≈ 1.0` (claude-code: rStar=1.7577, rOverQ=1.039) — meaning "large monotone shift". Or huge `rStar` AND `rOverQ ≈ 2.0` (no source in the current suite hits this combination) — meaning "large V-shape excursion". Or small `rStar` AND `rOverQ ≈ 2.0` (opencode comes close at rOverQ=1.850 with rStar=1.2621) — meaning "modest but symmetric oscillation".

The cross-correlation of `hAllRatio` and `rOverQ` across the five active sources is also weak: opencode has hAllRatio=0.700 (drift-driven) and rOverQ=1.850 (two-sided); these are nominally in tension, since the strongest "drift" reading should be a one-sided path. The reconciliation is that **opencode has BOTH a sustained ramp AND a V-shape on top of it** — Hadamard removes the ramp leaving residual i.i.d.-like noise (hAllRatio=0.700), but Buishand on the centered cumulative-deviation path still sees the V (rOverQ=1.850). The two refinement scalars together describe the structure better than either alone.

## 8. Where the suite is missing an axis

Five axes deployed, and the matrix has one obvious gap: **no per-source axis surfaces the unsigned statistical-significance of the path-shape statistic at axis-155**. Pettitt at axis-154 ships an explicit `pApprox` based on the asymptotic distribution of the rank-based statistic. Buishand has well-known asymptotic distributions for `R/sqrt(n)` and `Q/sqrt(n)` and `U` (tabulated in Buishand 1982), but axis-155 does not currently surface a `pApprox` column. Adding one would let the suite directly compare "which path-shape statistic is most significant" alongside "which change-point statistic is most significant" on the same five sources.

The current 0.6.411 release notes mention `rOverQ` and `rstaroverqstar` sort key as the only refinement; the asymptotic `pApprox` for Buishand R is a natural axis-155 follow-up (perhaps a `0.6.412` refinement), and would close the orthogonality matrix at the **five-axis-with-significance** level.

## 9. What the five-axis vector means at the population level

Reading the matrix top-to-bottom-left-to-right:

- **claude-code** is the one source where the volatility-class family agrees on "long tenure, real change-point". Hampel abstains (flat=y, structural). Allan reports moderate volatility with hAllRatio>1 (i.i.d.-with-tilt). CUSUM finds the lowest normMin (-1.692), the largest negative excursion. Pettitt says `kt=696`, `pApprox=0.0009` — strongly significant change-point. Buishand says `rStar=1.7577`, the highest of the suite, with `rOverQ=1.039` (one-sided, monotone). Verdict: **clean monotone shift over a long tenure**.
- **openclaw** is the runner-up: Pettitt's `ktNorm=0.963` is essentially saturated, Buishand's `rStar=1.7475` is essentially tied with claude-code's, `rOverQ=1.136` (one-sided), CUSUM normMax=1.537 (the highest positive excursion). All five axes agree: **strong, late, positive shift**.
- **opencode** is the path-shape outlier: `rOverQ=1.850` (two-sided V-shape), Pettitt `pApprox=0.4463` (no single changepoint), Hampel `asym=-0.333` (mixed-sign outliers). Verdict: **oscillating regime with low-side anomalies**.
- **codex** is the short-tenure spike-only source: `nOut=1`, `asym=+1.000`, Pettitt `ktNorm=0.688`, Buishand `rStar=1.0683` (lowest active). Verdict: **single up-spike on an otherwise quiet 8-day window**.
- **hermes** is the smooth source: lowest `allanDev` by an order of magnitude (`7.23e6`), `nOut=0`, Pettitt `pApprox=1.0000`, Buishand `rStar=1.1462` with `rOverQ=1.699` (tilted toward V-shape but at low magnitude). Verdict: **quiet, slightly-V-shaped, no statistically meaningful change**.

Five sources, five distinct verdicts, no two sources sharing the same five-axis fingerprint. The volatility-class family is doing its job: it **partitions the suite** into individually-recognizable signatures.

## 10. Falsifiability for the next axis

If a sixth volatility-class axis were to ship — say axis-156 as a Wallis-Moore phase-frequency test or a Cox-Stuart sign-of-second-difference run — the prediction from this analysis is that it should produce a **sixth ranking** that is again weakly correlated with the existing five rankings on the same five sources. Specifically: any new axis that rewards **path-segment-by-path-segment** structure rather than aggregate cumulative deviation should rank `opencode` near the top (because of its two-sided V-shape), and any new axis that rewards **single-shift detectability** should rank `claude-code` and `openclaw` near the top.

If a hypothetical axis-156 ranks all five sources in the same order as axis-155 `rStar`, that would falsify its independent value-add at the population level — it would be measuring the same thing Buishand measures, just under a different name. If it ranks them in a fresh order with no Spearman correlation above 0.5 against any of the existing five rankings, it earns its slot in the family.

The five-axis sprint is a useful testbed for that falsification because it has **already exhausted the easy orthogonality counter-examples**. Synthetic series can be constructed to differentiate any two axes; what matters at the suite level is whether real `~/.config/pew/queue.jsonl` data spreads the sources across the new axis in a way the existing five do not. Five axes deployed in a single day from `0.6.401` (commit `8fa73f8`) to `0.6.411` (commit `2b28a49`) is enough lineage to establish that the volatility-class family is not collapsing into a single dimension; the next axis will need to clear the same bar.
