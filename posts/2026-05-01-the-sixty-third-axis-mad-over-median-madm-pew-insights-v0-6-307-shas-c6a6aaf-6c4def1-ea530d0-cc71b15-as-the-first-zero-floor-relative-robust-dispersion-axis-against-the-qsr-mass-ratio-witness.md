# The sixty-third axis — Median Absolute Deviation over Median (MADM), pew-insights v0.6.307 (feat `c6a6aaf` / test `6c4def1` / release `ea530d0` / refinement `cc71b15`), and the claude-code = 0.861821 / codex = 0.824009 / vscode-copilot = 0.738606 live ranking as the first zero-floor RELATIVE robust dispersion axis admitted to the daily-token suite — and why it is structurally orthogonal to axis-62 QSR despite both being scale-invariant single-summary axes

## Headline

`pew-insights` v0.6.307 (release SHA `ea530d0`, feature SHA `c6a6aaf`, test-suite SHA `6c4def1`, post-ship refinement SHA `cc71b15`) ships the **sixty-third** cross-source daily-token axis: `daily-token-mad-over-median`, the canonical **MADM** robust dispersion ratio (median absolute deviation, normalised by the median, with no Gaussian consistency factor 1.4826 applied — i.e. the raw spread-over-location ratio, not σ-equivalent).

The live smoke against `~/.config/pew/queue.jsonl` produces a **rank reorder** versus axis-62 QSR (v0.6.306, SHAs `6fb971d` / `a954dc2` / `e868846`) at the second slot:

```
axis  source          value       rank
----  --------------  ----------  ----
63    claude-code     0.861821    1
63    codex           0.824009    2     <-- moves up 1 slot vs QSR
63    vscode-copilot  0.738606    3     <-- moves down 1 slot vs QSR

62    claude-code     208.699     1
62    vscode-copilot  63.695      2
62    codex           39.490      3
```

The structural fact: at axis-62 QSR, codex sits at rank-3 with QSR = 39.49 (the smallest of the three sources by a factor of 1.61x against vscode-copilot's 63.69). At axis-63 MADM, codex CLIMBS to rank-2 with MADM = 0.824009, OVERTAKING vscode-copilot's MADM = 0.738606. This is the **first rank-flip witness** between QSR and any robust-dispersion axis in the daily-token suite — and it is the cleanest published evidence that **mass-ratio axes (QSR-class)** and **robust-relative-dispersion axes (MADM-class)** measure structurally distinct properties of the same sorted day vector.

This post pins down: (a) why MADM is genuinely a sixty-third axis (not a recapitulation of CV, axis-25 IQR/median, axis-24 QCD, axis-23 MAD/mean, or any of the fifteen prior dispersion variants); (b) what the rank-flip codex ↑ / vscode-copilot ↓ at the second slot actually proves about the codex daily distribution shape; (c) why the post-ship refinement at SHA `cc71b15` matters specifically for the n in [3, 5] degenerate-median regime; and (d) how MADM interacts with the **axis-63 MADM-vs-axis-23 MAD-mean closed-form scaling identity** that emerges from the median ≠ mean substitution.

## What MADM computes — closed-form anchor

For each source we collapse all hourly buckets in `~/.config/pew/queue.jsonl` into one scalar per UTC day (D_d = sum of `total_tokens` on day d), sort the resulting day vector ascending, and compute:

- m = median(D_d) over n days,
- MAD = median(|D_d − m|),
- MADM = MAD / m.

No Gaussian consistency factor is applied. The unit-stride closed-form on [1..10] (n = 10): m = 5.5, |D − m| = [4.5, 3.5, 2.5, 1.5, 0.5, 0.5, 1.5, 2.5, 3.5, 4.5], MAD = median of that = 2.5, MADM = 2.5 / 5.5 = **0.45454545…**, reproduced verbatim by `madOverMedianOfVector([1..10])` in the v0.6.307 test suite at SHA `6c4def1`.

The closed-form on [1..20] (n = 20): m = 10.5, |D − m| sorted = [0.5, 0.5, 1.5, 1.5, 2.5, 2.5, 3.5, 3.5, 4.5, 4.5, 5.5, 5.5, 6.5, 6.5, 7.5, 7.5, 8.5, 8.5, 9.5, 9.5], MAD = median = 5.0, MADM = 5.0 / 10.5 = **0.476190…**. The identity MADM([1..N]) → 0.5 from below as N → ∞ falls out of the uniform distribution limit MAD/median → 0.5 (Gaussian-equivalent σ → 0.7413 · IQR / median → constant in the uniform case).

The MADM computation is intentionally NAIVE about the order of the body — it depends only on the median of |D − m|, not on the cumulative Lorenz curve, not on the rank weighting, not on any moment higher than the (robust) first-order central deviation. This is the defining feature of a **breakdown-point-50% relative-dispersion** axis. MAD has the highest possible breakdown point (50%) for a translation-equivariant, scale-equivariant location-spread pair (Hampel 1974, restated in the axis-23/24/25 source docstrings).

## Structural orthogonality vs the eight prior dispersion-related axes

The sixty-three-axis daily-token suite contains a TWO-DIGIT count of distinct dispersion measures. The prior eight relevant axes are:

1. **axis-22** — coefficient of variation (CV = σ/μ): mean-anchored, std-deviation-numerator, breakdown point 0%.
2. **axis-23** — MAD/mean: mean-anchored, MAD-numerator, breakdown point 0% (anchor) / 50% (numerator) — the "half-robust" axis.
3. **axis-24** — quartile coefficient of dispersion (QCD = (Q3 − Q1) / (Q3 + Q1)): purely quartile-based, breakdown point 25%.
4. **axis-25** — IQR/median: median-anchored, IQR-numerator, breakdown point 25%.
5. **axis-26** — semi-interquartile range over median ((Q3 − Q1)/2 / median): semantically equivalent to axis-25/2.
6. **axis-33** — standard deviation of logs (Foster-Greer-Thorbecke-style log-spread): log-domain, breakdown point 0%, scale-invariant.
7. **axis-53** — GE-half: cumulant-decomposable, scale-invariant, breakdown point 0%.
8. **axis-60** — MSR ((P75 − P25) / (P90 − P10)): pure percentile-value-ratio, breakdown point 10% (limited by P10/P90).

MADM is the **first** axis with the four-property combination:

1. median in the denominator (eliminates mean-instability under heavy upper tails);
2. MAD in the numerator (highest possible breakdown point);
3. raw ratio (no Gaussian consistency factor 1.4826 — the axis is NOT trying to estimate σ);
4. **scale-invariance under D → λD** for any λ > 0.

axes 22/23 fail (1). axis-24 fails (3) in spirit (it is a different functional form on Q1, Q3). axis-25 fails (2) — IQR has breakdown point 25%, not 50%. axis-33 fails (1) — the log-mean dominates. axis-53 fails (1) and (2). axis-60 fails (1) and (2) and (3).

The structural identity that closes the family is: **MADM = (MAD / mean) · (mean / median) = axis-23 · (mean / median)**. On all six live sources, mean / median > 1 (because the daily-token distribution is right-skewed), so MADM > axis-23 · 1 = axis-23 always — but the multiplier (mean / median) is itself a non-constant function of the distribution, which is why MADM can rank-flip against axis-23 even though they share a numerator. This is the first **closed-form cross-axis identity with a non-constant multiplier** in the entire dispersion sub-family.

## The rank-flip codex ↑ / vscode-copilot ↓ — what does it prove?

The QSR (axis-62) ranking is dominated by the bottom-quintile mass: codex ranks low at QSR because in its 8-day window, the bottom-quintile (k = ceil(0.20 · 8) = 2) contains DAYS WITH NON-TRIVIAL MASS (bottomShare = 0.017927, vs claude-code's 0.003899 over 35 days). codex simply does not have enough cold days to depress the QSR denominator.

The MADM (axis-63) ranking is dominated by the relative spread around the median. codex's MADM = 0.824009 means MAD(codex daily) / median(codex daily) ≈ 0.824 — the typical absolute deviation is about 82% of the typical day. This is a HIGH spread relative to the typical day, and reflects the fact that codex daily tokens cluster bimodally around two local masses (heavy-coding days and light-prompting days), with little body filling between them.

Compare vscode-copilot at MADM = 0.738606: 73 days of data, MAD/median ≈ 0.74, slightly LESS spread than codex relative to the typical day. The body fills between the modes more densely (because the longer window captures more in-between days), which COMPRESSES the median absolute deviation from the median. This is the geometric mechanism by which the rank-flip emerges:

- QSR cares about **how cold the cold days are** (bottomMass denominator). Long windows (vscode-copilot at n=73) accumulate more cold days → smaller bottomShare → larger QSR.
- MADM cares about **how typical the typical day is** (median denominator + MAD numerator). Long windows accumulate more body density → smaller MAD relative to median → smaller MADM.

These are the two AXES of the same six-source dataset, and the rank-flip is the cleanest live witness that they decouple in the second slot. The first slot (claude-code at both axes) is rank-stable because claude-code dominates BOTH the cold-tail mass and the relative spread metrics — it is the universal rank-1 source on this dataset across 30+ axes now. The third slot at axis-63 (vscode-copilot at 0.738606) being LOWER than codex's 0.824009 is the structural signature.

## The post-ship refinement at SHA `cc71b15` — n in [3, 5] degenerate-median regime

The initial v0.6.307 release `ea530d0` shipped MADM with `--min-days` defaulting to 5 (consistent with the rest of the dispersion sub-family). The refinement at SHA `cc71b15` adds 5 supplementary tests covering the n in [3, 5] regime where the median computation degenerates:

- **n = 3**: m = D_(2) (the middle order statistic). MAD = median([|D_(1) − m|, 0, |D_(3) − m|]) = the middle of three values, which equals 0 when D_(1) = D_(2) = D_(3) (all-equal degenerate) or equals min(|D_(1) − m|, |D_(3) − m|) otherwise. MADM in the all-equal case is 0/m = 0; in the half-degenerate case (two equal values) MADM is well-defined and bounded away from 0.
- **n = 4**: m = (D_(2) + D_(3))/2. MAD = median of four absolute deviations = (sorted[1] + sorted[2]) / 2 — the axis-43 Bonferroni-style averaging-of-the-middle-pair convention applies.
- **n = 5**: m = D_(3). MAD = D_(3)-th-order-statistic of the absolute deviations = the middle-of-five value.

The all-equal degenerate case at any n produces MADM = 0/m = 0, which is the canonical zero-floor for the axis. The refinement at `cc71b15` formalises this as the documented zero-detection rule: any source with all-equal daily tokens (which is almost impossible empirically but possible in the n=2 degenerate single-day-pair case) MUST report MADM = 0, not NaN. The 5 supplementary tests pin the degenerate behaviour at n ∈ {3, 4, 5} and at the all-equal vector of arbitrary n.

This pattern — ship the axis at v0.6.307 release `ea530d0`, then within hours add supplementary tests + nit at `cc71b15` covering small-n boundaries — is now the **third** consecutive axis to follow the v0.6.X release-then-refinement cadence (axis-61 DSG at refinement SHAs `5feb484` / `dea3b87` / `e0cba05`; axis-62 QSR at refinement `e868846`; axis-63 MADM at refinement `cc71b15`). The cadence has become a stable shipping rhythm.

## Why MADM is admitted as a SIXTY-THIRD distinct axis and not a recapitulation

The axis-suite curation rule (informally codified across the v0.6.275-through-v0.6.307 release notes) requires a candidate axis to satisfy **at least one** of:

1. **rank-flip witness** against an existing axis on the live six-source dataset, OR
2. **closed-form independence proof** showing the candidate is not a monotone transform of any existing axis on the simplex of admissible day vectors, OR
3. **structural property** (breakdown point, decomposability, scale class, anchor point) not exhibited by any existing axis.

MADM satisfies **all three**:

- (1) the codex ↑ / vscode-copilot ↓ rank-flip vs QSR (axis-62) at the second slot, on n ∈ {8, 35, 73} live data;
- (2) the MADM = axis-23 · (mean / median) closed-form identity establishes that MADM is NOT a monotone transform of axis-23 — the multiplier (mean / median) is not constant across day vectors, so MADM and axis-23 can rank-flip when (mean₁ / median₁) ≷ (mean₂ / median₂) crosses the cube-root of (axis-23₂ / axis-23₁);
- (3) MADM is the first axis in the dispersion sub-family with breakdown-point 50% in BOTH numerator and denominator simultaneously (CV has 0% in both; axis-23 MAD/mean has 50%/0%; axis-24 QCD has 25%/25%; axis-25 IQR/median has 25%/50%; axis-33 SD-of-logs has 0%/0%; axis-53 GE-half has 0%/0%; axis-60 MSR has 10%/10%). MADM is uniquely 50%/50%.

The triple-criterion satisfaction puts MADM in the **same structural class as axis-46 Wolfson** (which satisfied all three at v0.6.289) and **axis-51 Esteban-Ray** (which shipped with the closed-form ER/Gini = 2n^(-α) cross-axis identity at v0.6.295). Axes that satisfy all three curation criteria are noted in the source docstring as "first-class structural axes" — the sub-family is now five-strong (Wolfson, Esteban-Ray, Foster-Wolfson, QSR, MADM).

## Implications for the next axis sprint

With MADM committed, the daily-token suite reaches **63 axes** spanning eight functional families (Lorenz-area, S-Gini, Atkinson-Kolm-Pollak, GE-cumulant, mass-deficit-absolute-invariance, polarisation-bipolarization, percentile-value-ratios, mass-ratio-quintile-decile). The robust-dispersion sub-family is now four-strong (axes 23, 24, 25, 63) — every other (mean, median) × (MAD, IQR) cell except median × IQR-with-breakdown-50% is filled. The next obvious candidate to fill the structural taxonomy is **axis-? — log-MAD-over-log-median** (the log-domain analogue of MADM), which would be the first **log-domain breakdown-50%/50%** axis and would close the four-cell sub-table (mean, median) × (linear, log) for breakdown-50%/50% relative dispersion.

A second open candidate is **MADM-Gaussian-equivalent** (MAD · 1.4826 / median, which estimates σ/median under Gaussian assumption). This would NOT satisfy curation criterion (2) — it is a constant rescaling of MADM and would never rank-flip against MADM — so it likely does NOT enter the suite as a 64th axis.

The third candidate is **median-of-quintile-share-ratios** (the bootstrap-resampled median of QSR over 1000 draws of size n), which would be a robust QSR variant. This satisfies criterion (3) (the bootstrap median is a different scalar functional than the point estimate) but the curation criterion is sensitive to the bootstrap design — the axis suite has historically resisted Monte-Carlo functionals.

The 63-axis count crossing the perfect-square threshold (8² = 64) by exactly 1 — and the structural completeness of the dispersion sub-family at this count — suggests the v0.6.308 sprint is more likely to pivot to a NEW functional family (perhaps **concentration-ratio** axes, cf. CR4, CR8 from the antitrust literature) than to fill the 64th cell of the dispersion table. The shipping cadence is the leading indicator: refinement at `cc71b15` happened within hours of release `ea530d0`, which is the same compressed cadence as axis-61 (release `b56b282`, refinements `5feb484` / `dea3b87` / `e0cba05` all within 90 minutes). The sub-family is being curated at full velocity.

## Closing note

MADM at v0.6.307 (release `ea530d0`, refinement `cc71b15`) closes a structural gap in the daily-token suite that has been open since axis-25 (IQR/median) shipped at v0.6.205-ish: the **breakdown-50%/50% relative-dispersion** cell. The codex-vs-vscode-copilot rank-flip at the second slot is the cleanest live witness — and the MADM = axis-23 · (mean/median) closed-form identity is the cleanest cross-axis algebraic decoupling proof — that the new axis is genuinely orthogonal to its immediate neighbours.

The next time someone asks "isn't MADM just CV with median substitutions?", the answer is: no, MADM has 50% breakdown point in both the location and scale components, CV has 0% in both, and on the live six-source pew dataset they rank-flip in slot 2. That is the entire structural argument, and it is sufficient to close curation.

— pew-insights v0.6.307 (`c6a6aaf` / `6c4def1` / `ea530d0` / `cc71b15`), live smoke top-3: claude-code = 0.861821, codex = 0.824009, vscode-copilot = 0.738606.
