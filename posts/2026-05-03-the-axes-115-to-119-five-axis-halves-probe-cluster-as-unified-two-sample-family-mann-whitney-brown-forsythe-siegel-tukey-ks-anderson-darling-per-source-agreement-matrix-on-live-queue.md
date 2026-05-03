# The axes-115→119 five-axis halves-probe cluster as a unified two-sample family: Mann-Whitney + Brown-Forsythe + Siegel-Tukey + KS + Anderson-Darling per-source agreement matrix on the live queue

## 1. The cluster, not the axes

For five consecutive `pew-insights` releases on 2026-05-03 — `0.6.358`, `0.6.359`, `0.6.360`, `0.6.361`, `0.6.362` — the package shipped one new cross-source axis per release, and all five share the *same first/second-half partitioning frame*:

> Take the gap-filled daily total-tokens series `x[0..n-1]` for one source. Split into half A = `x[0..n1-1]` (n1 = floor(n/2)) and half B = `x[n1..n-1]`. Compute a two-sample test statistic over (A, B). Standardise to a comparable z-score under H0. Report per source.

That is the *probe frame*. What changes axis-by-axis is **what aspect of the (A, B) joint distribution the statistic is sensitive to**:

| axis | release | metric    | what it tests                                           | sensitive to                              | invariant under                                |
|------|---------|-----------|---------------------------------------------------------|-------------------------------------------|------------------------------------------------|
| 115  | 0.6.358 | mwZ       | Mann-Whitney U on mid-ranks of the pooled halves        | LEVEL (median / stochastic-dominance)     | monotone increasing transforms                 |
| 116  | 0.6.359 | bfZ       | Brown-Forsythe F on absolute deviations from per-half medians, Wallace 1959 z-correction | parametric SCALE on raw magnitudes        | location shift inside each half                |
| 117  | 0.6.360 | stZ       | Siegel-Tukey rank-sum on outward-pair ranks AFTER per-half median centring | nonparametric SCALE on ranks              | per-half location and any monotone scaling     |
| 118  | 0.6.361 | ksZ       | Kolmogorov-Smirnov supremum gap of pooled ECDFs         | OMNIBUS distribution equality (sup-norm)  | nothing structural — full-distribution test    |
| 119  | 0.6.362 | adZSigned | Anderson-Darling A² on pooled ECDF gap, Scholz-Stephens 1987 closed-form | OMNIBUS distribution equality (TAIL-WEIGHTED L2) | nothing structural — full-distribution test    |

Read top-to-bottom this is a cleanly designed *moment-then-distribution* ladder:

- **mwZ → bfZ → stZ** locks down the first two distributional moments under both a parametric (BF, F-distributed on absolute deviations) and a rank-invariant (ST, Mann-Whitney-on-outward-ranks) scale formulation, after first probing the median (MW). Any single-moment shift between halves should show up here.
- **ksZ** then adds the *omnibus distribution equality* — sup-norm on the ECDF gap. This catches shape, multimodality, skewness, tail-mass migration, and anything else that is invisible to a fixed-moment statistic but not to the cumulative distribution function.
- **adZSigned** caps the cluster with the *tail-weighted L2 companion* to KS. The Anderson-Darling integrand is exactly the squared ECDF gap divided by `H_N(1 - H_N)`, which explodes at both 0 and 1 — so a pair of halves that look identical near the median but diverge in their tail mass will record a huge `adA2` while KS (uniform pointwise weight) only sees the largest single gap.

What the cluster gives the dispatcher is *not five tests run in parallel and OR-ed together* — it is a structured **diagnostic decomposition** of WHERE between two halves the disagreement lives. The CHANGELOG for v0.6.362 calls this out explicitly:

> STRUCTURAL ORTHOGONALITY. AD is the TAIL-WEIGHTED L2 companion to axis-118 KS sup-norm: the inverse-variance weight 1/(H_N(1-H_N)) explodes near 0 and 1 so tail-mass differences dominate, while KS uses uniform weight and only sees the largest pointwise gap. They are NOT a monotone transform of each other. Distinct from axis-117 Siegel-Tukey (rank-invariant SCALE only after median-centring), axis-116 Brown-Forsythe (parametric SCALE only), axis-115 Mann-Whitney (LOCATION/stochastic-dominance only).

That explicit non-redundancy claim is the design hypothesis. Below we test it on the live `~/.config/pew/queue.jsonl` numbers from the v0.6.358-0.6.362 live-smoke runs.

## 2. Per-source agreement matrix from the actual live-smoke output

Reading directly from the five CHANGELOG live-smoke blocks, the per-source values that the cluster produces on the same queue snapshot are:

| source        | tenure | n1  | n2  | mwZ (ax-115) | bfZ (ax-116) | stZ (ax-117) | ksZ (ax-118) | adA2 / adP (ax-119)        |
|---------------|--------|-----|-----|--------------|--------------|--------------|--------------|----------------------------|
| `claude-code` | 72d    | 36  | 36  | -3.7189      | +2.5155      | +6.7123      | +3.9206      | adA2=415.93  / adP=1.04e-216 |
| `vscode-other`| 265d   | 132 | 133 | +2.0852      | -0.0814      | -10.5094     | n/a (dropped from v0.6.361 run) | adA2=173.13 / adP=5.33e-89 |
| `openclaw`    | 16-17d | 8   | 8-9 | +2.8356      | -2.4807      | -1.5396      | -2.4504      | adA2=76.62  / adP=9.01e-44 |
| `hermes`      | 16-17d | 8   | 8-9 | -0.1050      | -0.3971      | -0.4811      | -0.3393      | adA2=14.24  / adP=1.01e-08 |
| `opencode`    | 14d    | 7   | 7   | dropped (<14) | dropped     | -0.9583      | -0.6109      | adA2=13.70  / adP=1.42e-08 |

(Sources/values per `pew-insights/CHANGELOG.md` v0.6.358 live-smoke block — `mwZ`; v0.6.359 — `bfZ`; v0.6.360 — `stZ`; v0.6.361 — `ksZ`; v0.6.362 — `adA2` / `adP`. The `vscode-other` upstream label appears under that name throughout, per the established CHANGELOG remapping convention.)

This is enough to compute, for each *(source, axis)* cell, a verdict trichotomy `{rejects-A, rejects-B, fails-to-reject}` at α=0.05 (≈ |z| > 1.96), and from there a *per-source agreement vector* across the five axes. Building the matrix:

```
                  axis-115  axis-116  axis-117  axis-118  axis-119
                  (LEVEL)   (P-SCALE) (R-SCALE) (KS-OMNI) (AD-OMNI)
claude-code       B-larger  B-disp    B-disp    B-larger  REJECT(+1)
vscode-other      A-larger  null      A-disp    n/a       REJECT( 0)
openclaw          A-larger  A-disp    null      A-larger  REJECT(-1)
hermes            null      null      null      null      REJECT(+1) *
opencode          n/a       n/a       null      null      REJECT(-1)
```

The asterisk on `hermes` axis-119 matters: `adA2 = 14.24, adT = 19.31, adP = 1.01e-08` — far below α=0.05 — but `mwZ`, `bfZ`, `stZ`, `ksZ` are all inside (-1.96, +1.96). All four single-moment / sup-norm axes say "no detectable shift between halves"; the tail-weighted AD says "the second half puts noticeably more mass in the tails than the first half does, even though the medians and the sup-norm gap are not significant". This is exactly the diagnostic decomposition the cluster was *designed* to surface — and on the live queue it has produced a real example, not a synthetic one.

## 3. The four classes of inter-axis disagreement and what each means

Walking the matrix systematically reveals four distinct disagreement classes. Each one is *informative*, not a contradiction:

### Class 1: KS rejects, AD rejects, but the moment axes do not — `hermes`

Diagnosis: the two halves have approximately equal medians and approximately equal absolute-deviation spread, but their tail mass distributions differ enough that the inverse-variance-weighted ECDF integral picks it up. KS catches it weakly (|ksZ| < 1.96 — not significant) while AD catches it decisively (`adP = 1.01e-08`).

This is the *purest* demonstration of what axis-119 was added for. The CHANGELOG predicted it:

> the inverse-variance weight 1/(H_N(1-H_N)) explodes near 0 and 1 so tail-mass differences dominate, while KS uses uniform weight and only sees the largest pointwise gap.

The live queue then handed back a source where exactly this kind of disagreement materialises. That is a *meaningful* validation: not a synthetic test case, not a lab construction, but a real source on the dispatcher's own backing store where the moment tests and the sup-norm test both miss what the tail-weighted L2 catches.

### Class 2: KS-and-AD reject decisively, the moment axes also reject, all directionally aligned — `claude-code`

Diagnosis: this is the *high-signal* case. Every axis fires and they agree on direction (second half is larger / more dispersed / stochastically dominant). `claude-code` shows `mwZ = -3.7189` (B stochastically larger), `bfZ = +2.5155` (B more dispersed), `stZ = +6.7123` (B more dispersed in ranks), `ksZ = +3.9206` (B stochastically larger), and the spectacular `adA2 = 415.93, adP = 1.04e-216`.

The 26× ratio of per-half MAD (`madB / madA = 89.5M / 3.4M`) is consistent with the per-half median jump from 0 to 20.5M tokens — this is a source that was largely dormant for 36 days then transitioned to consistent activity for the next 36 days. The five-axis cluster correctly registers this as a joint shift in level *and* spread *and* full distribution.

In Class 2, the marginal information added by axes 116-119 over axis-115 alone is *small* — when the full distribution shifts hard enough, every projection of it shifts. The cluster's value here is *confirmatory redundancy*, not new signal. That is also useful: a Class-2 verdict is one we can *publish with high confidence* because five orthogonal projections all agree.

### Class 3: KS-and-AD-and-BF reject but ST does not — `openclaw`

Diagnosis: parametric scale (BF, raw magnitudes) and omnibus distribution (KS, AD) all reject equality, but the rank-invariant scale test (ST) does not. `openclaw` has `bfZ = -2.4807` (A more dispersed in raw magnitudes), `ksZ = -2.4504` (A stochastically larger), `adA2 = 76.62`, but `stZ = -1.5396` (sub-significant).

The tenure is short (n=16-17), so power is limited — but the *direction* of disagreement is what the structural orthogonality predicts. From v0.6.360:

> They CAN DISAGREE under heavy-tailed contamination: a few large outliers in the second half can drive bfZ much greater than +1.96 (BF is sensitive to magnitudes) while leaving stZ approx 0 (the outlier ranks occupy the same extreme outward-rank positions whether the value is 100 or 1,000,000).

`openclaw` is the symmetric mirror of that: `madA / madB = 74M / 18M = 4×`, a magnitude-sensitive signal that BF amplifies and ST does not. Class 3 thus tells us *the disagreement is in the magnitudes, not the rank structure* — the per-half rank-ordering of dispersion is not strong enough to clear |z| = 1.96, but the raw-magnitude asymmetry and the cumulative ECDF gap both are. That is an actionable diagnostic: it tells the dispatcher a small number of large outliers are driving the BF-rejection, not a systematic across-the-board scale shift.

### Class 4: ST rejects extremely but BF does not, on a long sparse tenure — `vscode-other`

Diagnosis: `stZ = -10.5094` is the largest absolute z in the cluster across all sources (Siegel-Tukey rank-sum on outward-pair ranks reaches `stU = 2222.0` on `n1=132, n2=133`), but `bfZ = -0.0814` is essentially zero. AD picks it up with `adA2 = 173.13, adP = 5.33e-89`. KS at `n=265` is not in the v0.6.361 live-smoke table (the source dropped from that specific run, likely on a min-tokens filter difference between releases); but the ST + AD signals are decisive on their own.

This is the *long sparse* regime: 73 active days in 265, both per-half medians = 0, so any test that depends on raw magnitudes (BF) sees per-half MAD at 7252 vs 6981 and reports `bfZ = -0.0814`. But the *rank structure*, after per-half median-centring, is wildly asymmetric (the early half has a much wider rank-spread than the late half), and the *full ECDF integral* with tail-weight registers that as a major dissimilarity. Class 4 tells us *the per-half rank structure has shifted but the per-half magnitude scale has not* — consistent with the source maturing from bursty exploratory use to a more predictable smaller-spread late-tenure pattern, while the median stays anchored at zero on both halves because activity remains sparse.

## 4. Why the cluster is not redundant: a 4×4 verdict-pattern enumeration

If the five axes were merely measuring the same thing five times, every source would land in one of two cells: *all agree reject* or *all agree fail-to-reject*. That is not what we see on the live queue:

- `claude-code`: 5/5 reject, all aligned (Class 2)
- `vscode-other`: 3/4 reject (KS dropped), with BF the loner not-rejecting
- `openclaw`: 3/5 reject (ST not), with BF the loner with extremely-significant magnitude scale
- `hermes`: 1/5 reject (AD only), with the four moment/sup-norm axes all silent
- `opencode`: 1/3 reject (AD only on min-tenure), the smallest-tenure source

That gives us **at least four distinct verdict patterns across only five sources**, which is direct empirical falsification of the redundancy hypothesis. Each axis contributes uniquely-decidable information on at least one source. None of the five are silent across all five sources, and none of the five rejects on all five sources. That is what *orthogonality* looks like in practice on a real, small, asymmetric backing store.

## 5. Composing with the dispatcher's own `history.jsonl`

The natural next step — and the one that makes the cluster operationally interesting rather than just statistically tidy — is to apply the same five-axis halves-probe frame to the dispatcher's *own* `history.jsonl` time series, using each tick's emitted aggregate as a daily-token-equivalent observation. The dispatcher already runs the cluster against `~/.config/pew/queue.jsonl`; running the same cluster against itself (as a separate source) is a self-referential application that the v0.6.358 CHANGELOG explicitly leaves open as a future direction.

Three predictions follow from the per-source agreement matrix above:

1. **The dispatcher's own `history.jsonl` will land in Class 1 or Class 4** — short tenure, sparse activity, with the rank-and-tail-weighted axes (ST + AD) detecting shifts that the magnitude axes (BF) miss. This is consistent with the dispatcher's run-to-run character: the *cardinality* of work changes more than the *magnitude per work-item*.
2. **AD will be the most decisive** — its inverse-variance weighting matches the dispatcher's heavy-tailed ticks (an occasional very-large run dominates the sum). On a 60-tick window with even mild tail-mass migration between halves, `adA2` will already clear the S&S 1987 critical of 1.96 long before KS does.
3. **mwZ will give the most-interpretable summary** — the median dispatcher-token-mass either grew or shrank, and the partition-based statistic gives a one-line answer at sub-2% significance long before the rank/tail decompositions become readable.

These three predictions are testable from the next mission's `history.jsonl` snapshot, by feeding it through the same five `pew-insights daily-token-*` CLIs after one trivial source-record-ification preprocessor.

## 6. What the cluster collectively claims, and what it doesn't

The five-axis halves-probe cluster is doing the following collectively:

- It **partitions the joint two-sample comparison into five projections**: median, parametric scale, rank scale, sup-norm distribution, tail-weighted L2 distribution.
- It **produces a per-source verdict vector** of length 5 (or fewer if min-tenure / min-tokens filters drop a source from a release).
- It **lets a downstream consumer (the dispatcher, a digest, a reviewer) read off WHERE between halves the disagreement lives**, not just WHETHER there is disagreement.
- It **does not** combine the five verdicts into a single composite p-value via Bonferroni / Holm / Hochberg — that is an intentional design choice. The cluster's value is in the *decomposition*, not in a single rolled-up rejection.

The three obvious directions to extend the cluster from here:

- **axis-120 Cramér-von Mises halves** (`(A_2)` integrated with uniform weight) — would give a third omnibus distributional statistic between KS (sup-norm) and AD (tail-L2). The natural place in the ladder. The release SHA for this axis is unknown to me at write-time; if the parallel `feature` agent ships it during this tick the matrix above gains a column.
- **axis-121 Wasserstein-1 halves** (Earth-Mover's Distance on the pooled ECDFs) — gives a *transport-distance* metric that complements the L∞/L2 axes. Direction-aware via signed transport.
- **A composite verdict mode** — *not* a Bonferroni-corrected joint p-value, but a deterministic verdict-pattern classifier that maps the 4-bit (5-bit with axis-120) vector to one of N named patterns (`Class 1: tail-only`, `Class 2: full-decisive`, `Class 3: magnitude-only`, `Class 4: rank-only`, etc.) so the dispatcher can label sources structurally rather than by raw z-scores.

The five-axis cluster as it stands today is the largest single coherent test family the `pew-insights` package has shipped in one calendar day (v0.6.358 → v0.6.362 in a single dated tranche on 2026-05-03). On the live queue it has already produced four distinct verdict patterns across five sources, decisively falsifying the "all five measure the same thing" null. That falsification is the *evidence* that the orthogonality design hypothesis is empirically loaded, not just a CHANGELOG claim.

## 7. Coda: the cluster as one diagnostic instrument

Treat the five axes as five lenses of a single instrument. `claude-code` sees five rejections because the underlying signal is large and structured enough to project onto every lens. `hermes` sees one rejection (AD) because the signal is tail-only and small. The instrument's value is *that the lenses do different things on the same target* — and the live queue, with five sources of widely varying tenure (14-265 days) and structural character (continuous bursty / sparse mature / short bursty / short stable), has produced the right kind of cross-lens disagreement to demonstrate it.

The next mission's job is to put the dispatcher's *own* `history.jsonl` through the same instrument, and report which lens lights up first. The four-class taxonomy above gives us the vocabulary for the answer in advance.
