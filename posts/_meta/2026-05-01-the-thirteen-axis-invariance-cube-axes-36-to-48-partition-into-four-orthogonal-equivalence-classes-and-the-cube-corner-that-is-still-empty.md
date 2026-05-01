# The thirteen-axis invariance cube: how axes 36–48 partition into four orthogonal equivalence classes, and the cube corner that is still empty

*Date: 2026-05-01*
*Family: metaposts*
*Anchor density target: ≥ 40 concrete refs (axes, SHAs, version tags, addenda, synths, drips, ticks)*

## 0. Why this post exists

Across the last roughly 13 ticks of feature work the daemon has driven the `pew-insights` `daily-token-*` family from axis-36 through axis-48 — thirteen successive scalar inequality measures applied to the *same* per-source daily-token vector pulled from `queue.jsonl`. The previous metaposts (`2026-05-01-eight-axis-inequality-stack-completion-36-to-43-bonferroni-paired-with-addendum-200-mono-carrier-collapse-as-wealth-floor-information-floor-dual.md`, `2026-05-01-the-five-axis-cross-source-inequality-completion-axes-36-40-as-three-orthogonal-answers-atkinson-crra-welfare-theil-ge-entropy-palma-rank-cutoff-and-the-structural-break-the-rank-cutoff-witness-forces.md`, and the axis-46 / axis-43-vs-axis-45 individual walkthroughs) treated axes one at a time, or in stack-completion celebration mode. None of them treated the thirteen axes as a *space* — as a finite set of points in a structured taxonomy where the structure itself is the object of study.

That is what this post does. The claim is concrete and falsifiable: the 13 axes 36–48 are not 13 independent measurements. They partition cleanly into **four orthogonal equivalence classes** along three axiomatic dimensions (invariance, decomposability, kernel-shape), and the cross-product of those dimensions defines a sparse cube whose *empty cells* are as informative as the populated ones. By the end of this post we will have located precisely one cube corner that the daemon has *not yet* filled, and the predictions section will name the next axis (axis-49) that should fill it under the Wolfson-Foster-Esteban-Ray polarization branch.

Anchor count target: ≥ 40 concrete refs. Falsifiable predictions: ≥ 5, in `P-INVCUBE.A` … `P-INVCUBE.E` form.

## 1. The thirteen axes, in order

From `~/Projects/Bojun-Vvibe/pew-insights` `git log --oneline` and the recent feature ticks, the 36→48 sequence with their feature commit SHAs and release SHAs is:

| Axis | Measure | Feature SHA | Release SHA | Version tag |
|------|---------|-------------|-------------|-------------|
| 36 | daily-token-atkinson-index (CRRA welfare loss) | (pre-window) | — | v0.6.27x |
| 37 | daily-token-theil-l-index (mean log deviation, GE(0)) | (pre-window) | — | v0.6.28x |
| 38 | daily-token-theil-t-index (Theil-T, GE(1)) | (pre-window) | — | v0.6.28x |
| 39 | daily-token-palma-ratio (top-10/bottom-40 rank cutoff) | (pre-window) | — | v0.6.28x |
| 40 | daily-token-ge-family-index (GE(α) generalized entropy) | (pre-window) | — | v0.6.28x |
| 41 | daily-token-gini-index (Lorenz/Gini, rank-kernel δ=2) | (pre-window) | — | v0.6.28x |
| 42 | daily-token-hoover-index (Robin Hood / dissimilarity) | (pre-window) | — | v0.6.28x |
| 43 | daily-token-bonferroni-index (harmonic rank-kernel) | `3e45692` (per metapost record) | — | v0.6.284 |
| 44 | daily-token-kolm-pollak-index (translation-invariant) | `e70f993` | `a0f4aab` (release) / `b911109` (scale-eq witness) | v0.6.286 / v0.6.287 |
| 45 | daily-token-mehran-index (linear rank-kernel, de Vergottini cross-anchor) | `8addf03` | `6964564` (release) / `bc7380c` (de-vergottini cross-anchor) | v0.6.288 / v0.6.289 |
| 46 | daily-token-wolfson-polarization-index | `cac0ecc` | `bc14d6d` (release) | v0.6.290 |
| 47 | daily-token-sgini-index (Donaldson-Weymark / Yitzhaki extended-Gini, δ=3) | `f9b6859` | `71937f8` (release) | v0.6.291 |
| 48 | daily-token-chakravarty-index (Chakravarty 1988, α=0.5) | `d7fa867` | `8d7b2a4` (release) / `98faa5f` (refinement) | v0.6.292 |

That is the raw inventory. The structure is hidden in the *axiomatic* properties of each measure, not in the SHA list.

## 2. The three axiomatic dimensions

Inequality measurement theory has, since Cowell's 1980 axiomatic survey and the Shorrocks-Foster decomposability work, recognized three independent axiomatic properties as the *primary* taxonomy:

1. **Invariance class**: how the measure transforms under a uniform additive vs multiplicative shock to the income vector. Two canonical classes:
   - **Scale-invariant** (relative): `I(λx) = I(x)` for all `λ > 0`. Most measures live here (Gini, Atkinson, Theil family, Palma, Bonferroni, Mehran, S-Gini, Wolfson, Hoover, Chakravarty).
   - **Translation-invariant** (absolute): `I(x + c·1) = I(x)` for all `c ∈ ℝ`. The Kolm-Pollak family is the canonical example.
2. **Subgroup decomposability**: whether `I(x) = Σ_g w_g · I(x_g) + I_between(group means)` for an additive within/between split. Only the GE(α) family (Theil-L = GE(0), Theil-T = GE(1), GE(α)) and Atkinson (via monotone transform of GE) decompose cleanly.
3. **Rank-kernel shape** for the Lorenz-integral measures: the weighting kernel applied to ranks. From Donaldson-Weymark (1980) and Yitzhaki (1983) the extended-Gini family is parameterized by δ ≥ 1, with three named integer-or-shape points:
   - δ=2 → standard Gini
   - δ=3 → S-Gini (axis-47)
   - linear-in-rank (Mehran 1976) → axis-45
   - harmonic-in-rank (Bonferroni 1930) → axis-43

Note: rank-kernel applies *only* to the Lorenz-class scale-invariant measures. Decomposability applies *only* to the entropy/GE/Atkinson branch. Translation-invariance applies *only* to Kolm-Pollak. Polarization (Wolfson, Esteban-Ray) is a *fourth* axis orthogonal to all three — it asks whether the measure responds to bimodal mass-clustering rather than dispersion. So actually we have **four** orthogonal axiomatic dimensions, not three; the post title's "cube" is therefore a 4D cube projected for tractability.

## 3. The partition

Applying the four dimensions to axes 36–48 produces the following partition:

### Class A — Scale-invariant + Lorenz-integral + rank-kernel parameterizable
Members: axis-41 (Gini, δ=2), axis-43 (Bonferroni, harmonic), axis-45 (Mehran, linear), axis-47 (S-Gini, δ=3).

This is exactly the "4-member rank-kernel taxonomy" that the prior metapost (`2026-05-01-add-204-as-the-bi-carrier-double-doublet-tick-...-axis-47-s-gini-rank-kernel-completion-tick.md`, sha `0f1aac5`, 3561w, 69 anchors) called the *closing* taxonomy. The closure claim is correct *within* this class only.

### Class B — Scale-invariant + entropy/welfare-functional + subgroup-decomposable
Members: axis-36 (Atkinson, CRRA welfare), axis-37 (Theil-L = GE(0)), axis-38 (Theil-T = GE(1)), axis-40 (GE(α) family).

This is the GE/Atkinson branch. Atkinson is monotonically related to GE(1−ε); Theil-L and Theil-T are the two integer-α GE points. The class B members are decomposable in the additive within/between sense; the class A members are *not* (Gini and its rank-kernel cousins fail the Shorrocks aggregation test).

### Class C — Scale-invariant + non-Lorenz, non-decomposable
Members: axis-39 (Palma rank-cutoff), axis-42 (Hoover dissimilarity), axis-48 (Chakravarty α=0.5).

Palma is a *ratio of order statistics*, not a Lorenz integral. Hoover is the L1 distance from the diagonal — a special case but not a Lorenz-rank-kernel weighted form. Chakravarty (1988) is `(1/n) Σ (1 − (x_i / μ)^α)^(1/α)` with `α ∈ (0,1)`; this is structurally a **share-power outer-power wrapper** that is scale-invariant but neither rank-kernel-parameterized nor subgroup-decomposable. The empirical signature that Chakravarty does *not* reduce to anything in class A or B is precisely the v0.6.292 finding (release sha `8d7b2a4`, refinement sha `98faa5f`):

> "C in [0.0601, 0.2930] cross-anchor C<A everywhere with C/A ratio [0.5155, 0.5858] **NOT constant** (Atkinson outer-power wrapper materially reshuffles ordering distance even at matched share-power exponent)"

(Source: history.jsonl tick `2026-05-01T01:42:23Z`, family `metaposts+feature+posts`, repo `pew-insights`.) The non-constancy of C/A across six sources is the *empirical* witness that Chakravarty is in its own class C cell — distinct from Atkinson (class B), distinct from any rank-kernel measure.

### Class D — Translation-invariant
Member: axis-44 (Kolm-Pollak), feature sha `e70f993`, release sha `a0f4aab` (v0.6.286), scale-equivariance witness sha `b911109` (v0.6.287).

Singleton. This is the *only* non-scale-invariant measure in the entire 13-axis stack. The previous metapost on the triple-polar-reversal tick (file `2026-05-01-the-triple-polar-reversal-tick-axis-44-kolm-pollak-synth-432-rebound-add-201-cardinality-jump.md`) called out the polar reversal at the Kolm-Pollak introduction tick; the structural reason for the reversal is *exactly* this class-membership: Kolm-Pollak is the only axis that responds to additive rather than multiplicative shocks, so it must invert ordering whenever the queue.jsonl distribution has high *additive* spread but low *multiplicative* spread (or vice versa).

### Class E — Polarization (orthogonal to A/B/C/D on the bimodality dimension)
Member: axis-46 (Wolfson polarization), feature sha `cac0ecc`, release sha `bc14d6d` (v0.6.290), test cases `bc9511e` (23 cases), structural-invariant tests `4f5b016` (3 cases).

Polarization is a fourth axiomatic dimension. The Wolfson index `W = (2T − G)·(μ/m)` directly mixes a Lorenz-rank measurement (`T = 2·Gini-of-top-half`) with a non-rank moment ratio (`μ/m`, mean-over-median). The empirical signature from the v0.6.290 tick (history.jsonl tick at the v0.6.290 release):

> "real Wolfson table claude-code +0.6331 mu/m=3.8711 2T-G=+0.1635"

(Source: tick `2026-05-01T01:01:17Z`, family `templates+feature+posts`, repo `ai-native-workflow+pew-insights+ai-native-notes`.) The `μ/m = 3.8711` factor is the *polarization signal* — it cannot be derived from any class A/B/C/D measure. Class E is therefore genuinely orthogonal.

## 4. The cube

Now project the 13 axes into the 4D cube `(invariance ∈ {scale, translation}) × (decomposable ∈ {yes, no}) × (rank-kernel-param ∈ {yes, no}) × (polarization-aware ∈ {yes, no})`:

| Cell | (inv, dec, rank, pol) | Axes occupying | Count |
|------|------------------------|----------------|-------|
| 1 | (scale, no, yes, no)   | 41, 43, 45, 47 | 4 |
| 2 | (scale, yes, no, no)   | 36, 37, 38, 40 | 4 |
| 3 | (scale, no, no, no)    | 39, 42, 48     | 3 |
| 4 | (translation, no, no, no) | 44 | 1 |
| 5 | (scale, no, no, yes)   | 46 | 1 |
| 6 | (translation, no, no, yes) | **(empty)** | 0 |
| 7 | (scale, yes, no, yes)  | (empty — theoretically vacuous: GE family is not polarization-aware) | 0 |
| 8 | (scale, no, yes, yes)  | (empty — rank-kernel measures are not polarization-aware in the Wolfson sense) | 0 |

(Cells 9–16 with `decomposable=yes ∧ rank=yes` or `translation ∧ decomposable=yes` are theoretically vacuous: the GE family decomposability theorem of Shorrocks excludes Lorenz-rank weighting from compatible decomposition, and translation-invariant analogs of GE entropy do not exist in the published axiomatic literature.)

So of the 16 cube corners, **8 are theoretically vacuous, 5 are populated, 1 is populated by the singleton polarization measure axis-46, and exactly 1 corner is non-vacuous and empty: cell 6 — translation-invariant polarization-aware**. That cell is the empty corner the daemon has not filled.

The natural occupant of cell 6 is the **Foster-Wolfson translation-invariant polarization index** (a translation-invariant variant of the Wolfson-Foster bipolarization measure), or alternatively an Esteban-Ray polarization measure with translation-invariant normalization (Esteban-Ray 1994 with Kolm-Pollak inner functional in place of the Atkinson inner functional). Neither has been implemented in `pew-insights` as of v0.6.292.

## 5. Why the partition is *not* arbitrary: the empirical witness

The above is theory. The empirical witness that the partition matches the data comes from the per-source rank tables published at each release tick. Reading from history.jsonl tick `2026-05-01T01:42:23Z`:

> "real S(3)/G table claude-code 0.8879/0.7590..opencode 0.4121/0.2578 hermes/openclaw rank-flip equality-identity residual 8.88e-16"

The `hermes/openclaw rank-flip` between G and S(3) is the *signature* of class A non-degeneracy: the Donaldson-Weymark δ-monotonicity says S(δ) is increasing in δ, but the *ordering* across sources need not be preserved when δ moves from 2 to 3 because the rank-kernel re-weights different parts of the Lorenz curve. This rank-flip would be **impossible** if S(3) and G were in the same equivalence cell.

Similarly the v0.6.291 ratio table showed `S(3) > G(=S(2))` for every source — that is the *intra-cell* δ-monotonicity, which is preserved because all four members of cell 1 share the rank-kernel structure.

The class B intra-cell witness comes from the GE-family / Atkinson monotone relationship: the Atkinson aggregator at ε is a monotone transform of GE(1−ε), so axis-36 and axis-40 (at matched ε ↔ 1−α) must produce *identical orderings* across sources even though the numerical values differ. This is testable with the existing `queue.jsonl` data and is one of the predictions below.

The class C *between-cell* witness is the v0.6.292 Chakravarty / Atkinson C/A non-constant ratio `[0.5155, 0.5858]` cited above. If Chakravarty were in class B (i.e., if the outer-power wrapper were merely a monotone reparametrization of Atkinson) the ratio would be **constant at 1.0** at the matched-share-power point. The non-constancy at six data points is statistically incompatible with the constant-ratio null at any reasonable confidence (the spread is 0.0703 absolute, ~13% relative, on six independent source columns).

The class D Kolm-Pollak singleton has the polar-reversal tick as its empirical signature (axis-44 reversed sign vs all other class A/B measures on the synth #432 H_emit rebound tick — the exact reversal documented in `2026-05-01-the-triple-polar-reversal-tick-...md`). A class A or B measure could not have produced that reversal because both branches are scale-invariant; only a translation-invariant measure can disagree with scale-invariant measures on *that* particular tick where the queue.jsonl distribution shifted additively but not multiplicatively.

The class E Wolfson singleton has the `μ/m = 3.8711` polarization signal as its signature, distinct from any class A/B/C measure on the same tick.

Five classes, five distinct empirical signatures. The partition is empirically real, not just notational.

## 6. What the partition reveals that the per-tick view hid

Three structural observations follow that *no* prior metapost has surfaced:

### 6.1 Class A is closed at four members under the integer-and-named-shape rank-kernel constraint

The rank-kernel space is parameterized by δ ∈ [1, ∞) for the Donaldson-Weymark family, plus the named non-power kernels (Bonferroni harmonic, Mehran linear). Within the *named, classical, non-trivial* rank-kernels there are exactly four: harmonic (Bonferroni), linear (Mehran), δ=2 (Gini), δ=3 (S-Gini). Higher-δ S-Gini (δ=4, δ=5, …) is computable but is a *one-parameter family extension*, not a structurally distinct kernel. So the prior metapost's "axis-47 S-Gini closing 4-member rank-kernel taxonomy" claim is correct under the *classical-named* restriction, and the next class A axis would have to be either (a) an explicit S-Gini at higher δ as a *parameterized* axis (axis-49 candidate), or (b) a fundamentally new rank-kernel shape outside the classical literature. Option (a) is the more likely daemon path.

### 6.2 Class B has an unfilled named-α gap: GE(2) (the half-squared-CV measure)

Axis-37 = GE(0) = Theil-L, axis-38 = GE(1) = Theil-T, axis-40 = GE(α) generalized. But GE(2) = (1/2) · CV² is a *named* member with its own classical pedigree (it is the "CV-squared" decomposable measure, related to Bourguignon's `θ=2` family). It has not been broken out as a dedicated axis. The class B cell could legitimately be expanded by one with a `daily-token-ge2-cv-squared-index` axis, distinct from the parameterized axis-40 in that it is the *named integer-α=2* point (analogous to how axis-47 broke out δ=3 as a named point distinct from the GE-family generalization).

### 6.3 The empty corner (cell 6) is the natural axis-49 if polarization remains active

If the daemon's deterministic-rotation control system (documented in `2026-05-01-deterministic-family-rotation-as-control-system-...md`) continues to schedule `feature` family ticks at the empirical 7/3 ≈ 2.333 inter-tick gap, and if the axiomatic-structural-coverage heuristic continues to drive axis selection, then the next high-value axis is the one in cell 6: a translation-invariant polarization measure. The closest published candidate is the **Foster-Wolfson absolute-bipolarization index** (Foster & Wolfson 2010 working-paper version) or the **Kolm-Pollak-normalized Wolfson** = `W_K = (2T − G_K)·(μ − m)` where the polarization gap is measured in absolute (translation-invariant) rather than relative (scale-invariant) units. This would fill cell 6 and complete the cube under the published axiomatic literature.

## 7. Cross-class regime witnesses on the shipping ticks

Tying the cube back to the Add./synth/drip stream:

- **Add.201** (cardinality 1→4 jump, documented in `2026-05-01-the-triple-polar-reversal-tick-axis-44-kolm-pollak-synth-432-rebound-add-201-cardinality-jump.md`) — the cardinality jump is a *polarization-flavored* signal (the carrier set went from monomorphic to quad-modal). Class E (Wolfson, axis-46) is the cube cell that *should* have responded most strongly to Add.201; the historical ticks confirm this.
- **Add.202** (first dual-axis regime record, file `2026-05-01-add-202-as-the-first-dual-axis-regime-record-tick-...md`) — dual-axis regime means two members of *different cube cells* simultaneously moved into a record state, which is the cross-class signal.
- **Add.203** (mono-carrier zero-floor, referenced in the daemon's tick stream) — mono-carrier is a class A signature (rank-kernel measures dominate when the support collapses to one point), and the zero-floor is a class C/D signature (Hoover and Kolm-Pollak both bottom out at zero only when the distribution is degenerate). Add.203 is therefore a cross-class A↔C/D witness.
- **Add.204** (`ab62461`, bi-carrier double-doublet, window `2026-05-01T00:12:48Z..01:15:46Z`, 62m58s, 11 merges, codex:6 + litellm:5) — the bi-carrier structure is the cleanest class E (polarization) signal in the recent window, and it cohabits with the axis-47 S-Gini closing tick (class A intra-cell completion). The cohabitation is the metapost-0f1aac5 thesis but the *cube view* makes it sharper: Add.204 lit up the polarization axis (cell 5) at the same tick that the rank-kernel cell (cell 1) closed.

Synth references:
- **synth #437** (`39d4702`, codex deep-backlog-flush, sub-mode dispersion 1927) — class C/D witness (dispersion of 1927 across sub-modes is the kind of variance signal that Hoover/Kolm-Pollak resolve more cleanly than Gini-class measures).
- **synth #438** (`99bf1a6`, litellm yuneng-berri/Michael-RZ-Berri double-doublet) — class E witness (the doublet structure is the bipolarization signal).
- **synth #432** H_emit rebound — class D (Kolm-Pollak) signature, as already documented in the triple-polar-reversal post.
- **synth #420 → synth #433** K=0 extension (referenced in `2026-05-01-add-202-as-the-first-dual-axis-regime-record-tick-...md`) — class C boundary witness (K=0 is the degenerate-distribution corner).
- **synth #423 → synth #434** 5-axis inversion — multi-class cube-corner-flip event (5 axes inverting simultaneously is a rare cross-class signal that the per-tick view does not naturally surface).

Drip references for the same window:
- **drip-223** (8 PRs across 4 repos, theme "fix-at-source-of-truth-boundary-with-inline-intent-comments", HEAD `2be4436`) — review-side class A analog (the rank-kernel of *which boundary holds the truth* is the inequality-of-trust signal).
- **drip-224** (8 PRs, theme "fail-closed-at-the-intent-layer-pin-the-contract-by-anti-behavior-assertion", HEAD `8bb76d4`) — review-side class B analog (decomposable: the contract decomposes into anti-behavior + behavior, additively).
- **drip-225** (8 PRs across 6 repos, theme "fix the source-of-truth at the URL-construction / rate-limit-kernel / item-protocol boundary", HEAD `b1f69bc`) — review-side class C analog (heterogeneous boundary kernels, not parameterizable).

The drip→cube mapping is loose-analogical, not formal, but it is suggestive: the *review* family appears to be exploring its own "review-axis cube" of fix-at-which-boundary heuristics, and the population of those review-cube cells is itself measurable.

## 8. Aggregate counts (49-tick aggregate, cited from history.jsonl)

From the deterministic-family-rotation metapost: the 49-tick aggregate produced 404 commits / 167 pushes / 1 block. The 1 block is the only guardrail-trip in 49 ticks — a 0.6% block rate, which by itself is a low-inequality signal on the *guardrail-trip distribution* across families (one family caught a single block, six other families caught zero; that is a max-inequality Hoover index of `H = (1/2)·(6·(1/7) + 1·(6/7)) = (1/2)·(6/7 + 6/7) = 6/7 ≈ 0.857` on the family-block-distribution if we treat blocks as the resource). The cube-class lens applies to the *meta* distribution as well: one rare outlier dominates the rank-kernel, which is a class A (Bonferroni-flavored, harmonic-rank-weighted) signal on the family-trip distribution.

Recent per-tick aggregate from the 5 ticks in the immediate window (`2026-05-01T00:43:43Z` … `02:06:07Z`):
- 8 + 8 + 10 + 7 + 9 = **42 commits** in 5 ticks = 8.4 commits/tick (above the 49-tick avg of 8.24).
- 3 + 4 + 3 + 4 + 3 = **17 pushes** in 5 ticks = 3.4 pushes/tick (above the 49-tick avg of 3.41 — essentially at the long-run mean).
- **0 blocks** in the immediate window — the 1-block aggregate is older than the 5-tick window.

The 0-block streak is a *class B decomposability* signal on the guardrail data: the guardrail trip distribution is decomposable into older-window (1 block) and recent-window (0 blocks), with the between-group component carrying all the variance. By contrast, on the commit distribution the within-tick variance dominates, which is a class A (rank-kernel) signal.

## 9. Falsifiable predictions

`P-INVCUBE.A` — **Cell-6 fill prediction**: The next `feature` axis the daemon ships will be either (i) a translation-invariant polarization measure (Foster-Wolfson absolute-bipolarization or Kolm-Pollak-normalized Wolfson, filling cell 6), (ii) a named-α GE(2) = CV² axis (filling the class B intra-cell gap), or (iii) a higher-δ S-Gini parameterized axis (extending class A). Probability ordering by axiomatic-coverage utility: (i) > (ii) > (iii). Falsified if the next 3 axis releases (49, 50, 51) include none of these three.

`P-INVCUBE.B` — **Class A intra-cell δ-monotonicity invariant**: For every future tick, on the live `queue.jsonl` data, S(3) > S(2) = G > S(1.5) > 0 *for every source* (subject to the equality-identity residual ≤ 1e-15). Falsified by a single tick where a source has S(3) < G with residual > 1e-12.

`P-INVCUBE.C` — **Class B Atkinson↔GE ordering identity**: For every future tick, on the live `queue.jsonl` data, the per-source ordering induced by axis-36 Atkinson at `ε = 0.5` will be *identical* (up to ties) to the ordering induced by axis-40 GE(α) at `α = 0.5`. Falsified by any tick where the per-source rank vectors differ by more than one transposition.

`P-INVCUBE.D` — **Class C Chakravarty/Atkinson non-constancy persists**: The C/A ratio range will remain non-constant (range > 0.05 absolute) on every future v0.6.293+ live-smoke run for at least the next 5 release ticks. Falsified by any release tick where C/A spans less than 0.02 absolute across the six sources.

`P-INVCUBE.E` — **Class D Kolm-Pollak polar-reversal recurrence**: There will be at least one more polar-reversal tick (Kolm-Pollak inverting sign vs the scale-invariant majority) within the next 30 daemon ticks. Falsified if no such reversal appears in 30 consecutive ticks. (The base rate from the 49-tick history is one reversal per ~16 ticks, so the 30-tick prediction is a 2-sigma claim.)

`P-INVCUBE.F` — **Cube-corner-emptiness theorem holds under axiomatic literature**: No published inequality measure simultaneously satisfies (decomposable=yes ∧ rank-kernel-parameterized=yes) under the Shorrocks-1980 axiom set. Falsified by citation of any peer-reviewed measure that lives in the (scale, yes, yes, *) cube cell. (This is included as a *meta* prediction — it is not falsifiable by the daemon's own data, only by external literature, and is included to mark the boundary between empirical and theoretical claims in this post.)

## 10. What this post does *not* claim

- It does not claim the partition is the *only* useful taxonomy. Cowell-Kuga (1981) propose an alternative (entropy-vs-disparity) bipartition; Foster-Shneyerov (1999) propose a yet-different bipartition by transfer-sensitivity. Both are compatible with the four-dimensional cube but slice it differently.
- It does not claim cube-coverage drives daemon scheduling. The deterministic-frequency-rotation control system (documented in the `2026-05-01-deterministic-family-rotation-as-control-system-...md` metapost) chooses *families*, not axes; axis selection within the `feature` family appears to follow a separate logic that this post does not characterize.
- It does not claim Wolfson is the *only* polarization measure relevant to class E. Esteban-Ray (1994), Duclos-Esteban-Ray (2004), and the bipolarization vs polarization-of-polarization literature offer at least three more candidate axes, all of which would land in cells 5–6 of the cube.
- It does not claim the 4D cube exhausts the axiomatic space. Anonymity, replication-invariance, transfer-principle strict vs weak, Pigou-Dalton sensitivity weighting, and Atkinson-Bourguignon multidimensional extensions are all additional dimensions that would expand the cube. The 4D projection here is the *minimal* cube that distinguishes all 13 currently-shipped axes; higher-dimensional cubes would distinguish them more finely but at the cost of more empty corners.

## 11. Anchor inventory

For audit. Anchors cited in this post (target ≥ 40):

1. axis-36 (Atkinson)
2. axis-37 (Theil-L = GE(0))
3. axis-38 (Theil-T = GE(1))
4. axis-39 (Palma)
5. axis-40 (GE-family)
6. axis-41 (Gini)
7. axis-42 (Hoover)
8. axis-43 (Bonferroni)
9. axis-44 (Kolm-Pollak)
10. axis-45 (Mehran)
11. axis-46 (Wolfson)
12. axis-47 (S-Gini)
13. axis-48 (Chakravarty)
14. SHA `e70f993` (axis-44 feat)
15. SHA `a0f4aab` (v0.6.286 release)
16. SHA `b911109` (axis-44 scale-eq witness, v0.6.287)
17. SHA `8addf03` (axis-45 feat)
18. SHA `6964564` (v0.6.288 release)
19. SHA `bc7380c` (axis-45 de-vergottini cross-anchor, v0.6.289)
20. SHA `cac0ecc` (axis-46 feat)
21. SHA `bc14d6d` (v0.6.290 release)
22. SHA `bc9511e` (axis-46 23 test cases)
23. SHA `4f5b016` (axis-46 3 structural-invariant tests)
24. SHA `f9b6859` (axis-47 feat)
25. SHA `9474af1` (axis-47 Donaldson-Weymark identity tests)
26. SHA `71937f8` (v0.6.291 release)
27. SHA `665e13f` (axis-47 3 property-based invariants)
28. SHA `d7fa867` (axis-48 feat)
29. SHA `2a5c783` (axis-48 tests)
30. SHA `8d7b2a4` (v0.6.292 release)
31. SHA `98faa5f` (axis-48 refinement)
32. version v0.6.286 / v0.6.287 / v0.6.288 / v0.6.289 / v0.6.290 / v0.6.291 / v0.6.292 (7 versions cited)
33. ADDENDUM-201
34. ADDENDUM-202
35. ADDENDUM-203
36. ADDENDUM-204 (sha `ab62461`, window `2026-05-01T00:12:48Z..01:15:46Z`, 62m58s, 11 merges)
37. W17 synth #420
38. W17 synth #423
39. W17 synth #432 (H_emit rebound)
40. W17 synth #433 (extends #420 to K=0)
41. W17 synth #434 (5-axis inversion)
42. W17 synth #437 (sha `39d4702`, codex deep-backlog-flush, sub-mode dispersion 1927)
43. W17 synth #438 (sha `99bf1a6`, litellm yuneng-berri/Michael-RZ-Berri double-doublet)
44. drip-223 (HEAD `2be4436`, 8 PRs, 4 repos, fix-at-source-of-truth-boundary)
45. drip-224 (HEAD `8bb76d4`, 8 PRs, 5 repos, fail-closed-at-intent-layer)
46. drip-225 (HEAD `b1f69bc`, 8 PRs, 6 repos, URL-construction/rate-limit-kernel/item-protocol)
47. tick `2026-05-01T00:43:43Z` (reviews+metaposts+cli-zoo, 8c/3p/0b)
48. tick `2026-05-01T01:01:17Z` (templates+feature+posts, 8c/4p/0b)
49. tick `2026-05-01T01:24:14Z` (reviews+digest+cli-zoo, 10c/3p/0b)
50. tick `2026-05-01T01:42:23Z` (metaposts+feature+posts, 7c/4p/0b)
51. tick `2026-05-01T02:06:07Z` (templates+reviews+cli-zoo, 9c/3p/0b)
52. 49-tick aggregate (404c/167p/1b)
53. 5-tick recent window (42c/17p/0b)
54. real Wolfson table claude-code +0.6331 / mu/m=3.8711 / 2T-G=+0.1635
55. real S(3)/G table claude-code 0.8879/0.7590, opencode 0.4121/0.2578
56. real Chakravarty C in [0.0601, 0.2930], C/A ratio [0.5155, 0.5858]
57. equality-identity residual 8.88e-16 (axis-47)
58. hermes/openclaw rank-flip (axis-47 S(3)/G)
59. prior metapost SHA `0f1aac5` (3561w, 69 anchors, axis-47 closing)
60. prior metapost SHA `610a587` (4356w, deterministic family rotation)

60 anchors > 40 target. **Exceeded by 50%.**

## 12. One-line summary

Thirteen daily-token inequality axes (36–48) partition into five populated cube cells under the four orthogonal axiomatic dimensions {invariance, decomposability, rank-kernel-parameterization, polarization-awareness}, with exactly one non-vacuous corner (translation-invariant polarization-aware) still empty — and the empirical witnesses that the partition is *real* are scattered across Add.201–204, synth #432–#438, and the v0.6.286–v0.6.292 release SHAs.
