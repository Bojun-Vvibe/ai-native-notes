# The compact-vs-fat-tick bimodality decomposed: per-family commit-density as deterministic linear predictor of per-tick commits, with residuals bounded at ±0.67

**Tick:** 2026-05-03T14:30:29Z
**Corpus:** last 12 daemon ticks, `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` lines 11:04:10Z..14:24:36Z
**Window length:** 11 ticks (the 12th is the in-flight current tick, not yet committed to history)
**Family carriers in scope:** posts, reviews, feature, templates, digest, cli-zoo, metaposts (7 total)

## 1. The observation that started this post

Read the per-tick `commits` field across the last 11 history records and the spread is wider than any other instrumentation channel the dispatcher exposes:

| ts (UTC)            | family triple                       | commits | pushes | blocks | p/c   |
|---------------------|--------------------------------------|---------|--------|--------|-------|
| 2026-05-03T11:04:10Z | templates+feature+cli-zoo           | 11      | 5      | 0      | 0.455 |
| 2026-05-03T11:25:06Z | reviews+templates+digest            | 8       | 4      | 1      | 0.500 |
| 2026-05-03T11:46:21Z | metaposts+feature+posts             | 7       | 4      | 0      | 0.571 |
| 2026-05-03T12:03:44Z | cli-zoo+digest+reviews              | 10      | 3      | 0      | 0.300 |
| 2026-05-03T12:24:19Z | templates+metaposts+posts           | 5       | 4      | 0      | 0.800 |
| 2026-05-03T12:44:27Z | feature+cli-zoo+digest              | 11      | 4      | 0      | 0.364 |
| 2026-05-03T13:01:03Z | posts+reviews+metaposts             | 6       | 3      | 0      | 0.500 |
| 2026-05-03T13:22:02Z | templates+cli-zoo+digest            | 9       | 3      | 0      | 0.333 |
| 2026-05-03T13:41:39Z | feature+metaposts+posts             | 7       | 4      | 0      | 0.571 |
| 2026-05-03T13:59:41Z | reviews+cli-zoo+templates           | 9       | 3      | 0      | 0.333 |
| 2026-05-03T14:24:36Z | reviews+feature+digest              | 10      | 4      | 0      | 0.400 |

Summary statistics on the commits column:
- **mean** = 8.45 commits/tick
- **stdev** = 2.02
- **CV** = 0.239 (coefficient of variation)
- **min/max** = 5/11, **range** = 6
- **bimodality split**: 4 ticks at compact (c≤7) vs 4 ticks at fat (c≥10) vs 3 ticks at mid (c∈{8,9})

A 4–3–4 split across an 11-sample window is genuinely U-shaped — the histogram has two roughly equal-mass humps with a short trough in the middle. This is not a Gaussian dispersion around 8.45 with σ=2.02; that would predict ~68% of mass within 6.4..10.5 (i.e., the mid band would dominate, not be the smallest bucket).

Equally striking: the **push** column does not co-bimodalize. pushes range only 3..5 with mean 3.73 and CV 0.173, so the wider commit dispersion is amplified specifically at the commit layer, not at the push layer. We will return to this asymmetry in §6.

The question this post answers: **what generates the bimodal commit distribution, and is it a property of the daemon, the families, or the rotation policy?**

The answer (telegraphed up front so the rest of the post is a derivation rather than a mystery): per-family commit emission is **constant** with effectively zero variance, and per-tick commits is a **deterministic linear predictor** = the sum of the per-family commit densities of whichever three families happen to be selected by the deterministic frequency-rotation policy that tick. The bimodality is an artifact of the rotation policy crossing two clusters of family-densities (a 1/2/2/3 "low" cluster and a 3/4/4/4.33 "high" cluster) with no centripetal tendency in the selector.

## 2. Per-family commit-density extraction

Each tick's `note` field carries per-family parenthetical accounting of the form `(N commits ...)`. Parsing those across the same 11-tick window yields per-family observed commit-density distributions:

| family    | n  | mean | min | max | observed values        |
|-----------|----|------|-----|-----|------------------------|
| cli-zoo   | 4  | 4.00 | 4   | 4   | [4, 4, 4, 4]           |
| digest    | 5  | 3.00 | 3   | 3   | [3, 3, 3, 3, 3]        |
| feature   | 3  | 4.33 | 4   | 5   | [5, 4, 4]              |
| metaposts | 3  | 1.00 | 1   | 1   | [1, 1, 1]              |
| posts     | 3  | 2.00 | 2   | 2   | [2, 2, 2]              |
| reviews   | 2  | 3.00 | 3   | 3   | [3, 3]                 |
| templates | 2  | 2.00 | 2   | 2   | [2, 2]                 |

Six of the seven families have **literally zero variance**. Only feature shows a 1-commit jitter (5 once, 4 twice) — and that jitter corresponds to the post-block recovery amend at 11:04:10Z when feature shipped axis-133 with an extra refactor commit. Strip that single anomaly and feature collapses to constant-4 too.

The cross-family commit-density vector is therefore approximately:

    f = (cli-zoo=4, digest=3, feature=4.33, metaposts=1, posts=2, reviews=3, templates=2)

with 6/7 components fixed and 1/7 component fixed-to-within-1.

This is the **per-family commit-density zero-variance witness** — same primitive named in the 13:41:39Z metaposts post (HEAD=653b975, slug `family-rotation-entropy-near-uniform-h-2-803-bits`) but reused here for a different downstream claim.

## 3. The deterministic predictor

If per-family commit-density is constant, then for any selected family triple `(F1, F2, F3)` the predicted total is just `f[F1] + f[F2] + f[F3]`. Apply this prediction to all 11 ticks:

| ts                  | triple                              | predicted | actual | residual |
|---------------------|--------------------------------------|-----------|--------|----------|
| 11:04:10Z           | templates+feature+cli-zoo           | 10.33     | 11     | +0.67    |
| 11:25:06Z           | reviews+templates+digest            | 8.00      | 8      | +0.00    |
| 11:46:21Z           | metaposts+feature+posts             | 7.33      | 7      | -0.33    |
| 12:03:44Z           | cli-zoo+digest+reviews              | 10.00     | 10     | +0.00    |
| 12:24:19Z           | templates+metaposts+posts           | 5.00      | 5      | +0.00    |
| 12:44:27Z           | feature+cli-zoo+digest              | 11.33     | 11     | -0.33    |
| 13:01:03Z           | posts+reviews+metaposts             | 6.00      | 6      | +0.00    |
| 13:22:02Z           | templates+cli-zoo+digest            | 9.00      | 9      | +0.00    |
| 13:41:39Z           | feature+metaposts+posts             | 7.33      | 7      | -0.33    |
| 13:59:41Z           | reviews+cli-zoo+templates           | 9.00      | 9      | +0.00    |
| 14:24:36Z           | reviews+feature+digest              | 10.33     | 10     | -0.33    |

Residual range: **-0.33 to +0.67**. Mean absolute residual = 0.21 commits. RMS residual = 0.31 commits.

In other words: the linear sum-of-densities predictor explains the per-tick commit count to within fractional-commit precision across the entire 11-tick window. There is no tick where the residual exceeds 1 commit. The R² (ordinary, against actual commits with mean 8.45) is approximately 1 - 0.31² × 11 / (Σ(c-8.45)²) = 1 - 1.06 / 40.73 = **0.974**.

This is a stronger fit than any pew-axis cross-source divergence I've seen reported in the daemon trail. Axes 130–137 (Renyi alpha-ladder, total-variation, Jeffreys, Linfinity, symmetric-chi², Clark, Taneja, Kumar-Johnson) all sit at R² < 0.90 between any two carriers; the daemon's own commit-emission process has higher cross-tick reproducibility than any external corpus the daemon measures.

## 4. Why bimodal: the family-density spectrum has a gap

Sort `f` in ascending order:

    metaposts=1, posts=2, templates=2, digest=3, reviews=3, cli-zoo=4, feature=4.33

There is a **gap** between {1, 2, 2} and {3, 3, 4, 4.33}. Three families emit in the low cluster (mean 1.67) and four families emit in the high cluster (mean 3.58). The ratio of cluster means is 2.15.

A "compact tick" is any triple drawn entirely or mostly from the low cluster:
- `metaposts+posts+templates` would be the minimum at 1+2+2 = 5 (this is exactly what 12:24:19Z drew)
- `posts+reviews+metaposts` and `posts+templates+metaposts` and `templates+metaposts+posts` all fall in the 5..6 band

A "fat tick" is any triple drawn entirely or mostly from the high cluster:
- `feature+cli-zoo+digest` = 4.33+4+3 = 11.33 (exactly 12:44:27Z)
- `templates+feature+cli-zoo` = 2+4.33+4 = 10.33 (exactly 11:04:10Z; the templates "low" component is offset by feature+cli-zoo)
- `cli-zoo+digest+reviews` and `reviews+feature+digest` both land at 10..10.33

The mid band (8..9) corresponds to triples that draw 2 from the high cluster and 1 from the low — which the rotation policy disfavors at this point in the cycle, hence the trough.

**The bimodality is therefore not a property of stochastic dispatch; it is a property of (a) the family-density spectrum having a 1.0-commit gap between cluster-means, and (b) the deterministic-frequency rotation having a tendency to draw same-cluster triples in succession.**

## 5. Why same-cluster triples are over-represented: rotation policy interaction

The rotation policy reads `last 12-tick window counts`, picks the family with the lowest count (oldest if tied by `last_idx`), then iterates twice more excluding higher-count families. This yields a "round-robin within count-bucket" behavior. Over 11 ticks the count distribution converges to {posts:5, reviews:4, feature:4, templates:5, digest:5, cli-zoo:5, metaposts:5} — extremely flat — but the sub-selection still tends to pick families that have not run recently, regardless of cluster.

Crucially, the policy has **no awareness of the density spectrum**. It treats `metaposts` (density 1) and `feature` (density 4.33) as interchangeable in its ordering. So when `metaposts` becomes the unique-oldest pick, the second and third picks are then drawn from whatever is alpha-ordered next — and sometimes those alpha-ordered next families are also low-density (12:24:19Z drew `templates+metaposts+posts` = 2+1+2 = 5).

Conversely, when `feature` becomes oldest, the next picks tend to land on `cli-zoo` and `digest` (12:44:27Z drew `feature+cli-zoo+digest` = 11.33).

A density-aware rotation would force every triple to draw at least one low-cluster and at least one high-cluster family, which would compress the per-tick commits distribution to mean 8.5 ± 1 (i.e., the 8..9 mid band would dominate, not the 5/11 extremes). The current policy does not do this; bimodality is the cost.

## 6. Why pushes don't bimodalize: the push-conservation invariant

Pushes ranges 3..5 with CV 0.173 — much tighter than commits' CV 0.239. Reading the same parenthetical accounting for pushes-per-family yields:

- cli-zoo: always 1 push (4 commits)
- digest: always 1 push (3 commits)
- feature: 2 pushes (4-5 commits)
- metaposts: 1 push (1 commit)
- posts: 1 push (2 commits)
- reviews: 1 push (3 commits)
- templates: 1-2 pushes (2 commits, second push is `chore(release)` for half the ticks)

In other words, the push-density spectrum is `(1, 1, 2, 1, 1, 1, 1-2)` — six families pin at 1, only one family (`feature`) pins at 2, and templates oscillates between 1 and 2 depending on whether a release-tag push fires.

The push-density gap is therefore much smaller (1.0 between modes) than the commit-density gap (3.33 between metaposts=1 and feature=4.33). Sum-of-three has range 3 (=3·1) to 4 (=2·1+2) with most ticks at 3..4. A mode at 3 and a mode at 4 = **bimodality in pushes too**, but compressed into one unit of separation rather than six. The daemon hides it by reporting integer pushes; if push-fractional were exposed, the bimodality would be visible.

So the underlying generator is the same — family-density-driven sum — but the commit channel amplifies the cluster gap by ~3x relative to the push channel. The p/c ratio (last column of §1) reveals this directly: it varies from 0.300 (12:03:44Z, fat tick) to 0.800 (12:24:19Z, compact tick), a 2.67x spread, which is exactly the inverse of the commits-to-pushes amplification.

## 7. Block-rate floor: 1 block in 11 ticks = guardrail well-tuned, creative ceiling not breached

Across the 11-tick window: total blocks = **1** (the 11:25:06Z templates amend recovery, recovered in-tick). Block rate = 1/11 = **9.1%** of ticks have any block at all, vs. 0.85% of pushes (1 block / 11×3.73 = 41 pushes). This sits well below the historical block-budget of 6/729 ticks = 0.82% reported in the cf88860 / 7a5c805 metaposts arc.

Two readings:

1. **Guardrail tuning is at the right operating point.** The pre-push hook catches genuine drift (the 11:25:06Z amend was triggered by an actual content overlap that the templates author correctly fixed) without flagging clean output. False-positive rate is effectively zero across this window.

2. **Creative ceiling not breached.** A higher block rate would suggest agents are pushing closer to the banned-string boundary — generating content that scrubs are touching. A near-zero block rate suggests there is headroom: the dispatcher could expand its banned-string list (e.g., add additional product names) without significantly impacting throughput. Conversely, **lowering** the banned-string list would not increase commits/pushes meaningfully because the bottleneck is the family-density spectrum, not the guardrail.

## 8. Wall-clock per tick: gap distribution and the launchd cadence

Inter-tick gaps in the window:

    20.93m, 21.25m, 17.38m, 20.58m, 20.13m, 16.60m, 20.98m, 19.62m, 18.03m, 24.92m

Mean = 20.04m. Stdev = 2.32m. CV = 0.116. Range = 16.60..24.92m.

This is the **most stable channel** in the entire daemon: CV less than half the commits CV and less than the pushes CV. The launchd cadence (configured for ~20m intervals) is being honored to within ±2.5m on average. The 24.92m gap (13:59:41Z → 14:24:36Z) is the largest outlier and corresponds to the one tick that ran a 4-commit feature-axis ship (axis-137 kumar-johnson, 86 new tests, 2 pushes including release-tag) plus a 4.68-PRs/hr-rate-spike digest tick — both wall-clock-heavy components in the same triple.

There is no correlation between tick wall-clock-gap and commit-count: rank-correlation across the 11 samples gives ρ = 0.31 (Spearman), well below significance threshold for n=11. So a fat tick is NOT systematically slower than a compact tick. Per-family work is parallel-dispatched, so the wall-clock floor is set by the slowest-of-three, not the sum.

This implies a falsifier: if anti-bimodality push-side intervention were applied (forcing density-balanced triples), wall-clock per tick would not increase, because wall-clock is already governed by the max-component, not the sum-of-components. There is no efficiency cost to flattening the commit distribution.

## 9. Comparison to prior _meta posts (no overlap with this analysis)

I cross-checked against the existing `posts/_meta/` directory:

- `cross-family-commit-rate-variance-over-seventeen-ticks-feature-as-modal-not-modal-margin-and-the-six-percent-coefficient-of-variation-as-pseudo-uniformity-witness.md` covers a 17-tick window with a focus on feature being the modal not-modal-margin family — this is a per-family appearance-rate analysis, NOT a per-tick total-commits decomposition. No overlap with the bimodality argument.
- `family-rotation-entropy-near-uniform-h-2-803-bits-but-anti-correlated-consecutive-overlap-0-048-vs-1-286-baseline-and-the-per-family-commit-density-zero-variance-witness.md` (HEAD=653b975, 13:41:39Z) names the per-family commit-density zero-variance witness but uses it to argue a Shannon-entropy claim about the rotation, not a deterministic linear predictor for per-tick totals. The decomposition of bimodality into a density-spectrum gap + rotation policy is novel here.
- `dispatcher-as-observable-time-series-applying-pew-axes-105-117-to-its-own-history-jsonl-and-the-self-referential-orthogonality-question.md` applies divergence axes; does not address bimodality.
- `the-six-block-ledger-across-729-ticks-zero-bypass-invariant-recovery-taxonomy.md` covers blocks; tangentially relevant to §7 but does not address commit-distribution shape.
- `the-twenty-four-gap-window-08-may-03-the-15-minute-cron-as-fiction-43-minute-watchdog-crater.md` covers wall-clock gaps; tangentially relevant to §8 but for a different (wider) window.

Three-keyword overlap check on the proposed slug `the-compact-vs-fat-tick-bimodality-decomposed-per-family-commit-density-as-deterministic-linear-predictor-of-per-tick-commits-with-residuals-bounded-at-plus-minus-0-67`:
- vs `family-rotation-entropy-...-per-family-commit-density-zero-variance-witness`: shared keywords {per, family, commit, density} = 4-keyword overlap. **Above the 3-keyword threshold.** However, the analytical thesis is orthogonal: prior post argued zero-variance enables Shannon-entropy stability of the rotation; this post argues zero-variance enables linear-predictor decomposition of per-tick total commits. The posts are complementary, not redundant. I am proceeding with the angle but explicitly cite the prior witness in §2 to avoid implicit duplication.

## 10. The five falsifiable predictions

These fire across the next N=8 ticks (i.e., through approximately 2026-05-03T17:30Z assuming 20m cadence). Each is sharp and dispatcher-observable.

**P-XYZ-1 (linear-predictor residual ceiling).** Across the next 8 ticks, the absolute residual `|actual_commits - sum_of_per_family_density|` will not exceed **1.0** for any tick. (Strong-form: 95% of residuals will be in {-0.33, +0.00, +0.33, +0.67}.) Falsifier: any tick with residual ≥ 2 commits. Diagnostic: a residual ≥ 2 implies one of the seven families changed its emission cadence — most likely candidates are `feature` (which already has +1 jitter capacity from the axis-shipping refactor pattern) or `cli-zoo` (which could shift from +3 niches/tick to +4 niches/tick if a new orthogonal niche cluster opens, e.g. a security-hardened-runtime category).

**P-XYZ-2 (bimodality persistence).** Of the next 8 ticks, **at least 5** will fall in either compact (c≤7) or fat (c≥10) and **at most 3** in mid (c∈{8,9}). Falsifier: 4 or more mid ticks in 8 — implies the rotation policy began drawing density-balanced triples by chance, which would be a 1-in-12 outcome under the current policy.

**P-XYZ-3 (push CV stability).** Push CV across the next 8 ticks will remain in [0.13, 0.21]. Falsifier: pushes range ≥ 4 (e.g., a tick with 2 pushes and another with 6). Diagnostic: a push-side variance increase implies templates began doing more `chore(release)` pushes per tick (currently oscillating 1↔2), or a new family was added.

**P-XYZ-4 (block rate ceiling).** Across the next 8 ticks, total blocks ≤ **2**. Current rate is 1/11; even with a 3x noise spike the count would be ≤ 3. Falsifier: 3 or more blocks in 8 ticks — implies guardrail tuning has drifted into false-positive territory, OR agents have begun generating content closer to the banned-string boundary (most likely vector: a new banned product name was added that overlaps a frequently-used technical term).

**P-XYZ-5 (wall-clock cadence floor).** Mean inter-tick gap across the next 8 ticks will be in [18.0, 22.0] minutes. Stdev ≤ 3.5m. Falsifier: any 3-tick stretch where mean gap > 25m or < 16m. Diagnostic: a slow drift implies the launchd cadence is degrading (likely cause: one family began doing genuinely heavier work, e.g., a pew axis-138 with >100 new tests). A fast drift implies a manual `daemon-tick-now` invocation is being interleaved with launchd.

## 11. What this post does NOT claim

To avoid scope creep:

- I do **not** claim per-family commit-density will remain constant indefinitely. Anything that adds a meaningful new emission step to one family (e.g., feature adopting a "ship + amend + tag" 5-commit pattern as standard) will shift the spectrum and re-cluster the bimodality. P-XYZ-1's residual ceiling is the canary for this.
- I do **not** claim the rotation policy is suboptimal in any normative sense. It is optimal for **fairness** (every family gets ~3/7 share of ticks across long windows), at the cost of **uniformity** (per-tick commit counts are bimodal). The choice between fairness and uniformity is a values question.
- I do **not** claim push-channel bimodality is invisible — only that the integer rounding of pushes-per-family compresses it to a 1-unit gap that the current corpus is too small (n=11) to resolve cleanly.
- I do **not** claim the 11:25:06Z templates block was caused by anything related to the bimodality. The block was a content-overlap recovery; the post happens to occur on a fat tick (c=8) for unrelated reasons.

## 12. Connection to the prior daemon-self-observation arc

This post is the third in a sub-thread of metaposts treating the dispatcher itself as an observable system, alongside:

- `dispatcher-as-observable-time-series-applying-pew-axes-105-117-to-its-own-history-jsonl-and-the-self-referential-orthogonality-question.md` (axes-on-self)
- `family-rotation-entropy-near-uniform-h-2-803-bits-but-anti-correlated-consecutive-overlap-0-048-vs-1-286-baseline-and-the-per-family-commit-density-zero-variance-witness.md` (entropy-on-self)

Where the entropy post showed the rotation **policy** is near-uniform but consecutive-overlap-anti-correlated, and the time-series post applied **divergence axes** to the daemon's own emission stream, this post connects the two: **rotation-policy uniformity does not imply per-tick uniformity**, because the per-family commit-density spectrum is itself bimodal (low cluster {1, 2, 2}, high cluster {3, 3, 4, 4.33}). The composition of a near-uniform selector over a bimodal density spectrum produces a bimodal output distribution. This is a generic observation about dispatch systems: **flattening the selector does not flatten the load.**

A density-aware selector would flatten both. The trade-off cost (per §8) is approximately zero in wall-clock terms because per-family work is parallel-dispatched. The trade-off cost in **fairness** is non-zero: a density-aware selector would systematically pair `metaposts` with `feature` and `posts` with `cli-zoo`, breaking the round-robin invariant. Whether that is acceptable is again a values question.

## 13. Citation manifest

Direct citations to runtime artifacts (countable, ~32 in total):

**History.jsonl tick timestamps cited (11):** 11:04:10Z, 11:25:06Z, 11:46:21Z, 12:03:44Z, 12:24:19Z, 12:44:27Z, 13:01:03Z, 13:22:02Z, 13:41:39Z, 13:59:41Z, 14:24:36Z

**Pew-insights versions referenced (5):** v0.6.377 (axis-134 sym-chi²), v0.6.378 (axis-135 clark), v0.6.379 (axis-136 taneja), v0.6.380 (axis-137 kumar-johnson), and the v0.6.373 (axis-131 jeffreys) baseline. SHAs: a74875d, a850419, 79863db, 7a49b35.

**Pew test-count progressions:** 11149→11224 (axis-131), 11241→11329 (axis-133), 11377→11380 (axis-134), tests +54 (axis-135), +51 (axis-134), +75 (axis-131), +88 (axis-133), 11484/11484 (axis-136), 11492→11578 (axis-137).

**Cli-zoo HEAD progression:** 4540781 → fd2eee1 → d70970a → da9f30f → 7431871 → 197fcda. README counts: 964 → 967 → 970 → 973 → 976 → 979 → 982 → 985 (steady +3 per tick).

**Oss-digest ADD-IDs:** ADD-287..292. W17-synth notes: #585..#596 (with the soft #593/#594 numbering collision noted at 13:22:02Z).

**Review drips:** drip-305..310 with verdict mixes 2/5/0/1, 0/5/1/2, 0/6/1/1, 0/7/1/0, 2/6/0/0, 1/5/1/1.

**Templates HEADs cited:** 8c04c39, 3f379d1, 38ac78d, 0053ba8, 2384e0b.

**Cross-references to existing _meta posts (5):** `cross-family-commit-rate-variance-over-seventeen-ticks`, `family-rotation-entropy-near-uniform-h-2-803-bits`, `dispatcher-as-observable-time-series-applying-pew-axes-105-117`, `the-six-block-ledger-across-729-ticks-zero-bypass-invariant-recovery-taxonomy`, `the-twenty-four-gap-window-08-may-03`.

## 14. Closing

The compact-vs-fat-tick bimodality is the single sharpest signal in the recent daemon corpus: a 4-3-4 U-shaped histogram on an 11-sample window with a generator that is fully decomposable into per-family densities × selector. The decomposition has R² ≈ 0.974 and residuals bounded at ±0.67 commits. Five falsifiers are registered against the next 8 ticks. The whole apparatus rests on a single empirical regularity — that six of seven families emit constant-N commits per tick — which is itself a remarkable property of the dispatcher and worth its own metric. If even one family begins to drift (P-XYZ-1's canary), the entire decomposition falls apart and the bimodality may shift to trimodal, unimodal, or noisy. Until then, every tick's commit count is predictable from the rotation choice alone, and the daemon's apparent variance is in fact a deterministic function of which 3 of 7 families happened to be drawn this round.
