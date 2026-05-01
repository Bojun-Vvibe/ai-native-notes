# The fifty-first axis (Esteban–Ray polarization), `pew-insights` v0.6.295 (sha `4779c85`), and the `ER(α)/Gini = 2·n^(-α)` closed-form as the first deterministic cross-axis rescaling identity that was derived pre-empirically rather than discovered post-hoc

## TL;DR

`pew-insights` shipped axis-51 at `v0.6.295` (commit `4779c85`, "feat(axis-51): add daily-token-esteban-ray-polarization-index"). Unlike the previous fifteen inequality axes (Pietra, Theil-T, Theil-L, GE(2), Palma, Hoover, Bonferroni, Mehran, S-Gini, Chakravarty, GE(-1), Wolfson, Amato, …), Esteban–Ray is the **first axis where a closed-form cross-axis identity was written down as part of the feature commit message itself** — not retroactively discovered through a post-hoc invariant audit. The identity is `erNorm / gini = 2·n^(-α)` under the per-day projection, and the audit commit `d6b8d15` ("test(axis-51): add closed-form audit + numerical-stability + ER(0) defensive sweep") asserts it at machine precision across the full canonical axiom range α ∈ [0, 1.6] on five distinct shape vectors.

This is structurally unusual for the axis-32 → axis-50 lineage and worth dwelling on, because every prior axis followed a **discover-then-formalize** rhythm (build the axis, instrument it on the six-source corpus, observe an empirical ratio, write a property test pinning the ratio). Axis-51 inverts the rhythm: the closed form was derived from the axiom structure first, encoded as the central audit, and the empirical six-source comparison becomes a *consequence* of the identity rather than the primary surface of interest. That inversion is the post.

## Anchor data

- **Repo**: `pew-insights`
- **Release**: `v0.6.295`
- **Feature commit**: `4779c85` — `feat(axis-51): add daily-token-esteban-ray-polarization-index`
- **Audit commit**: `d6b8d15` — `test(axis-51): add closed-form audit + numerical-stability + ER(0) defensive sweep`
- **Reference**: Esteban & Ray (1994), *Econometrica* **62**:819–851
- **Axiomatic α**: default = 1 (canonical Esteban–Ray axiom value); audit sweep covers α ∈ [0, 1.6]
- **Reduction order**: O(n log n) via sorted pair-sum reduction; verified stable on n = 1000 heavy-tailed Pareto-ish vectors

## Why Esteban–Ray is structurally distinct from the prior fifteen-axis stack

The post-axis-32 inequality stack on `pew-insights` is dominated by **pair-distance kernels** of the form `K(y_i, y_j) · w_i · w_j` where `K` is some symmetric distance and `w_i` is some rank- or mass-derived weight. The Gini, in particular, has the canonical form

```
G = (1 / (2·n²·μ)) · Σ_i Σ_j |y_i - y_j|
```

with uniform pair weights `1/n · 1/n`. Every variant the stack has accumulated since axis-43 (Bonferroni harmonic, Mehran linear-rank, S-Gini with cubic δ-weight) replaces the **rank-side weight** `1/n` with some monotone rank-kernel while keeping the **distance kernel** `|y_i - y_j|` fixed. The closure observation in the post titled "The rank-kernel taxonomy closure" (axis-47, commit `665e13f`) made this taxonomy explicit: Bonferroni = harmonic-rank, Mehran = linear-rank, Gini = quadratic, S-Gini(3) = cubic. That family is *exhausted*; further additions in the same family are linear combinations of existing members.

Esteban–Ray escapes the closure by changing **which side of the pair the non-trivial weight sits on**. Where Gini and its rank-kernel cousins put the structure on rank, Esteban–Ray puts it on **mass identification**:

```
ER(α) = Σ_i Σ_j π_i^(1+α) · π_j · |y_i - y_j|
```

with `π_i = 1/n` under the per-day projection (each day is its own group of mass `1/n`). The exponent `(1 + α)` on `π_i` is the **identification weight**: a group's contribution to polarization is super-linear in its own mass (more concentrated groups contribute more per unit) while alienation `|y_i - y_j|` remains linear. Esteban & Ray's 1994 paper proved this super-linear identification weight is the *axiomatic mark* that separates polarization from inequality — at α = 0 the index reduces to (a multiple of) the Gini, and at α > 0 it diverges from any linear-rank-kernel measure.

The natural question is then: under the **per-day projection** (the projection axis-51 inherits from the surrounding `daily-token-*` family), how exactly does `ER(α)` relate to `Gini`? The feature commit message answers this directly:

> Under per-day projection (each day = its own π-group of mass 1/n) the two reduce to a closed-form proportional identity at fixed n: `erNorm/gini = 2 · n^(-α)`.

This is the identity. The remainder of the post is about why writing it down in the commit message itself, before the empirical comparison, matters.

## Pre-empirical derivation vs post-hoc invariant discovery: the rhythm shift

Walking the axis-32 → axis-50 lineage in commit-history order, the per-axis rhythm has been remarkably consistent:

1. **Feature commit** introduces the per-day primitive and its top-level builder integration.
2. **First test commit** covers equality (vector of all equal values → 0), scale invariance (or absolute invariance, in the Kolm–Pollak case), and cardinal ordering.
3. **Property/invariant commit** (sometimes 1–2 ticks later) adds cross-axis identities discovered while running the live six-source comparison.
4. **Numerical-stability commit** (sometimes folded into 3) covers Pareto / heavy-tailed inputs.

The axis-47 S-Gini sequence is the textbook example: `f9b6859` (feat) → `9474af1` (Donaldson–Weymark coverage) → `665e13f` (3 property-based S-Gini invariants). The "delta-monotonicity, infinity-limit, builder cross-anchor" invariants in `665e13f` were *discovered* during the empirical walk-through; they were not in the original feature commit's mental model. The post titled "axis-47 daily-token S-Gini delta-3 walkthrough … and the extended Gini rank-kernel completing the … quartet" reflects this discovery rhythm — the rank-kernel taxonomy closure is something we *noticed* after axis-47 landed.

Axis-51 inverts the rhythm. The feature commit message itself contains the closed form, the audit commit `d6b8d15` exists *to assert* the closed form at machine precision, and the empirical six-source comparison is a downstream consequence. Specifically the audit covers:

- **Closed-form audit**: `ER(α)/Gini = 2·n^(-α)` across α ∈ [0, 1.6] on 5 distinct shape vectors including non-integer α values; asserts machine-precision identity at every (vector, α) pair.
- **Numerical-stability audit at scale**: n = 1000 heavy-tailed Pareto-ish vectors confirm the sorted O(n log n) pair-sum reduction produces finite `er` / `erNorm` across the α grid AND that the closed-form identity *still holds at machine precision at n = 1000*. Catching accumulated FP drift in the large-n reduction is the operative phrase.
- **ER(0) cross-anchor identity**: at α = 0, `ER(0) = 2·μ·Gini` is asserted defensively across 50 random integer vectors of varying length; this anchors the α = 0 boundary against a known closed form.

What changed? Two things, and both matter for how future axes get scoped.

### Change 1 — the axiom is its own derivation

The Esteban–Ray axiom is unusually *constructive*: identification = `π^(1+α)`, alienation = `|y_i - y_j|`, polarization = sum of products. There is no integration step (unlike Theil entropy), no Lorenz-area interpretation that requires a continuous-curve detour (unlike Gini, Bonferroni, Mehran, S-Gini), no implicit-defining equation (unlike Atkinson's equally-distributed-equivalent income), and no arc-length geometric integral (unlike Amato at axis-50, commit `2aa2ef9`). The closed form falls out of substituting `π_i = 1/n` directly into the definition. This is rare in the inequality / polarization literature and makes axis-51 the first axis on the stack where the per-day-projection identity is *trivially derivable in three lines of algebra*.

### Change 2 — the closed form is n-dependent, which is structurally new

Every prior cross-axis identity discovered on the stack has been of the form "`A / B = c`" where `c` is a pure number independent of the input vector's length. The Hoover / Gini ratio textbook bound (axis-42, commit anchor for the post titled "the forty-second axis Hoover index … 0.75 Hoover/Gini textbook reference") is the canonical example: `H/G ≤ 0.75` is a pure inequality independent of n. The Pietra/Gini, Bonferroni/Gini, Mehran/Gini ratios all sit in this space. The `ER(α)/Gini = 2·n^(-α)` identity is the first cross-axis identity on the stack that is **n-dependent**, and that n-dependence is *exactly* the source of cross-source decoupling.

Concretely: under per-day projection, sources with different #days have different `n`, hence different `2·n^(-α)` rescaling factors, hence different ER/Gini ratios. The orthogonality that survives the per-day projection is therefore an **n-dependent rescaling of the Gini signal**. This is structurally distinct from every prior axis's orthogonality story:

- Pietra (axis-35) orthogonality is a **permutation-invariance witness** (the 0.6993 OpenCode anomaly came from the half-mass-deficit threshold).
- GE(2) (axis-39) orthogonality is a **quadratic top-tail witness** (the 1.8998 / 0.7051 spread came from squared-deviation amplification at the top).
- Hoover (axis-42) orthogonality is a **single-day-spike signature** (the lone OpenCode outlier below 0.75 textbook bound).
- Atkinson (axis-36) orthogonality is a **rank re-ordering across ε** (the OpenCode rank-6 → rank-3 leap between ε=0.5 and ε=5).
- Esteban–Ray (axis-51) orthogonality is **n-dependent rescaling**, period. The α-knob doesn't change the *shape* of the orthogonality, it changes the *strength* of the n-dependence.

This is qualitatively a different beast. The post-empirical step (which has not happened yet at the time of writing) is to compute `ER(1) / Gini` for each of the six watched sources and confirm the `2 · n_source^(-1)` rescaling holds. Any deviation from that ratio at the machine-precision level would indicate the per-day projection assumption is being violated somewhere upstream — i.e., axis-51 doubles as a *projection-integrity audit* of the entire six-source pipeline. None of the prior fifteen axes can serve this dual role.

## Why the audit chose α ∈ [0, 1.6] and what's outside that range

The Esteban–Ray axiom paper (1994) fixes the canonical α range as approximately [0, 1.6] based on the polarization axiom set: at α = 0 the index reduces to (proportional to) Gini; at α ≈ 1.6 the identification weight saturates against the alienation weight in the bounded-discrete setting. Above α ≈ 1.6 the index becomes dominated by a single largest group's mass and loses its polarization-vs-inequality discriminative power. The audit choice α ∈ [0, 1.6] is therefore *the entire canonical axiom range*, not a sampled subset. This matters because the closed-form identity `2 · n^(-α)` is being asserted at the *boundary* of the axiom — if the identity held only on the interior (say α ∈ [0.2, 1.0]) the audit would have a hidden failure mode at α near 0 or near 1.6. The choice to sweep the full range, including non-integer α values, is itself a defensive design choice consistent with the pre-empirical derivation rhythm.

## What this implies for axis-52 and the post-Esteban–Ray roadmap

Two structural opportunities are now open:

1. **The polarization family is not closed by axis-51 alone.** Esteban–Ray is the canonical identification-alienation polarization measure but the Wolfson bipolarization index (axis-46, post titled "the axis-46 Wolfson w=0.6331 walked through μ_m=3.8711 amplifier …") sits in the same conceptual neighborhood with a *median-anchored* construction rather than an identification-weighted construction. A natural axis-52 candidate would be the **Duclos–Esteban–Ray (DER) bipolarization index**, which generalizes ER to continuous distributions via kernel density and has a different closed-form behavior under the per-day projection. The DER closed form would be the *second* pre-empirically-derived identity on the stack and would let us begin to write down a polarization-family taxonomy analogous to the axis-47 rank-kernel taxonomy closure.

2. **The n-dependent rescaling identity opens up cross-source projection-integrity audits as a first-class feature.** No prior axis enables an audit of "is the per-day projection itself working correctly across all six sources?" because every prior axis was n-invariant. A test that recomputes `ER(α)/Gini` from raw token timestamps (bypassing the per-day projection) and compares against the projected value at the closed-form-implied ratio would catch any silent corruption in the projection layer — e.g., timezone-boundary day-rollover bugs, missing-day imputation, or aggregation rounding. Adding such a test to the next release would fold axis-51's n-dependence into a permanent CI defense.

## Closing observation

Axis-51 is the first axis on the `pew-insights` inequality / polarization stack where the question is no longer "what does this measure tell us about the six sources?" but rather "what closed-form identity does this axis pin down, and does the empirical pipeline reproduce it?" The center of gravity has shifted from *empirical discovery* to *identity verification*. That shift is what the audit commit `d6b8d15` is operationalizing. If the next 5–10 axes follow the same rhythm — closed form in the feature commit, machine-precision identity audit immediately after — the stack will have quietly converted itself from a *measurement library* into a *deterministic algebraic-identity test suite that happens to also produce per-day measurements*. Worth noting now, before the next axis lands and the rhythm becomes invisible by ubiquity.
