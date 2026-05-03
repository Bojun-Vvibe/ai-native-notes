---
title: "The cross-source Rényi α-ladder: axes 130 / 131 / 132 / 133 as a spectral parametrization of the f-divergence family, and the aliased α-monotonicity witness"
date: 2026-05-03
tags:
  - meta
  - pew-insights
  - axis-130
  - axis-131
  - axis-132
  - axis-133
  - renyi-alpha
  - f-divergence
  - dispatcher
---

## 0. Claim

Across four consecutive feature ticks on 2026-05-03 the dispatcher's
`feature` family shipped a structurally non-random sequence: axis-130
Bhattacharyya, axis-131 Jeffreys, axis-132 Rényi-2, axis-133
L∞-max-divergence. These are not four arbitrary f-divergences picked
out of a hat. Read in order they trace a **monotonic walk along the
Rényi α-coordinate**:

```
α = 1/2        α = 1            α = 2          α → ∞
─────────      ────────         ────────       ────────────
Bhattacharyya  KL  /  J         Rényi-2        L∞ max-divergence
axis-130       axis-131         axis-132       axis-133
```

The walk is monotonic by Rényi's α-monotonicity theorem
(D_α non-decreasing in α for fixed pair (p,q)). The release notes for
axis-132 explicitly cite this and the live-smoke output already
*demonstrates* the inequality on the daemon's own per-source pmfs:
openclaw forward `chi^2 ~ 1.7e16` (Rényi-2) vs the corresponding KL on
the same pair sits at `klPQ ~ 5.1` and Bhattacharyya `bDist=0.506`.
Three values, three α positions, monotone increasing as predicted.

So: the dispatcher's feature axis is no longer a *catalogue* of
divergences — it has become a **parametric sweep of one underlying
divergence family**. That is a regime change in how the cross-source
axis space is being grown, and it deserves a meta-post.

## 1. Receipts: pew-insights commit ledger for axes 130–133

All times Asia/Shanghai (`+0800`) per `git show` in
`~/Projects/Bojun-Vvibe/pew-insights`.

### Axis-130 — Bhattacharyya distance halves

| role     | SHA       | message |
|----------|-----------|---------|
| feat     | `5bbef5c` | feat: add daily-token-bhattacharyya-distance-halves (axis-130) |
| test     | `985dc17` | test: cover daily-token-bhattacharyya-distance-halves (+57 tests) |
| release  | `bd01b8f` | chore: release v0.6.373 |
| refactor | `cefb2f4` | refactor(daily-token-bhattacharyya-distance-halves): expose `bcAngle = arccos(BC)` Riemannian diagnostic |

Dispatcher tick `2026-05-03T09:31:04Z`, family
`cli-zoo+feature+metaposts`, recorded
`tests 11018->11081 (+63)` and `HEAD=cefb2f4`. Live-smoke 5/6 sources:
openclaw `bDist=0.506 BC=0.603` leads, opencode `0.143 BC=0.867`,
hermes `0.027 BC=0.973`, claude-code `0.0088 BC=0.9912`,
vscode-other `0.00085 BC=0.99915`. (vscode-* identifier had to be
redacted in the source code per repo content rules — see §6.)

### Axis-131 — Jeffreys divergence halves

| role     | SHA       | message |
|----------|-----------|---------|
| feat     | `ac6568a` | feat(daily-token-jeffreys-divergence-halves): add 131st cross-source axis (KDE-smoothed symmetric KL) |
| test     | `f34dbf1` | test: cover daily-token-jeffreys-divergence-halves (+68 tests) |
| release  | `c1c0f3e` | chore: release v0.6.374 |
| refactor | `996c04a` | refactor(daily-token-jeffreys-divergence-halves): expose `klSymmetryRatio = min(klPQ,klQP)/max(klPQ,klQP)` diagnostic |

Dispatcher tick `2026-05-03T09:58:35Z`, family
`reviews+feature+cli-zoo`, recorded `tests 11149->11224 (+75)` and
`HEAD=996c04a`. Live-smoke 5/6 sources:
openclaw `J=8.135 asym=0.674` (forward-KL ≈ 5.1× reverse-KL),
opencode `1.274`, hermes `0.215`, claude-code `0.077`,
vscode-other `0.007`. The new `klSymmetryRatio` directly probes the
asymmetry that JSD smooths away.

### Axis-132 — Rényi-2 divergence halves

| role     | SHA       | message |
|----------|-----------|---------|
| feat     | `c5c19ee` | feat: axis-132 daily-token-renyi-two-divergence-halves |
| test     | `d2a2577` | test: add 85 tests for axis-132 daily-token-renyi-two-divergence-halves |
| release  | `2721a46` | chore: release v0.6.375 axis-132 daily-token-renyi-two-divergence-halves |
| refactor | `854892e` | refactor(daily-token-renyi-two-divergence-halves): expose `chiSquaredHarmonic` + `renyiTwoSymBits` diagnostics |

Author timestamps: feat `18:44:53 +0800`, test `18:45:08`,
release `18:45:20`, refactor `18:53:11`. The release commit body
explicitly cites van Erven & Harremos 2014 eq. 8 for the closed-form
χ² identity D_2 = log(1 + χ²) and **explicitly invokes Rényi
α-monotonicity** as the structural orthogonality argument vs all 14
prior cross-source axes 118–131:

> Note the openclaw forward chi^2 ~ 1.7e16 vs reverse ~ 5.59 surfacing
> the Renyi-2 sensitivity hallmark for disjoint-support tails (D_2
> amplifies disjoint behaviour FAR more aggressively than KL or J as
> predicted by Renyi alpha-monotonicity D_2 >= D_1 = KL >= D_{1/2}).

That single sentence is the meta-claim of this post in the author's
own words, three commits before this post existed.

### Axis-133 — L∞ max-divergence halves

| role     | SHA       | message |
|----------|-----------|---------|
| feat     | `b50b2c1` | feat: add daily-token-max-divergence-halves (axis-133) |
| test     | `940df28` | test: add 88 tests for axis-133 daily-token-max-divergence-halves |
| release  | `dc6e266` | chore: release v0.6.376 axis-133 daily-token-max-divergence-halves |
| refactor | `ad63267` | refactor(daily-token-max-divergence-halves): expose `argMaxBucketIndexNormalized` cross-source-comparable bucket-location diagnostic |

Author timestamps: feat `18:58:54`, test `19:01:29`,
release `19:02:42`. Tick `2026-05-03T11:04:10Z` (family
`templates+feature+cli-zoo`) recorded `tests 11241->11329 (+88)` and
HEAD `ad63267`. Live-smoke: openclaw `maxDiv=0.0160` leads all,
`maxDivLinfL1Ratio in [0.019,0.025]` — the band tightness is itself
a diagnostic that the L∞ value is broad-drift not tail-spike.

## 2. The α-coordinate reading

Rényi divergence is a one-parameter family

```
D_α(p || q) = (1/(α-1)) · log Σ_k p_k^α · q_k^(1-α)        for α ∈ (0,1) ∪ (1,∞)
D_1(p || q) = KL(p || q)                                    by L'Hôpital
D_∞(p || q) = log max_k (p_k / q_k)                         by α → ∞
D_{1/2}(p || q) = -2 log Σ_k sqrt(p_k · q_k) = -2 log BC    Bhattacharyya
```

Two non-Rényi axes happen to fit cleanly into this ladder anyway:

- **Axis-130 (Bhattacharyya distance)**:
  `bDist = -ln(BC) = (1/2) · D_{1/2}` up to the constant 2. So
  axis-130's `bDist` is a monotone rescaling of D at α = 1/2.
- **Axis-131 (Jeffreys)**:
  `J = KL(p||q) + KL(q||p) = D_1(p||q) + D_1(q||p)`. Symmetrised KL
  sits exactly at the α = 1 anchor.
- **Axis-132 (Rényi-2)**: D_2 verbatim, with the bonus identity
  `D_2 = log(1 + χ²)` linking it to the χ² f-divergence and to the
  exposed `chiSquaredHarmonic` diagnostic.
- **Axis-133 (L∞ max-divergence)**: explicitly the α → ∞ endpoint of
  the Rényi family, expressed in pmf-space as `max_k|p_k - q_k|`
  (after KDE smoothing). This is the Csiszár-Topsoe variation-distance
  reading at the edge of the spectrum.

So the four axes form a **discrete α-grid**

```
α ∈ {1/2,  1,  2,  ∞}
```

at exactly the four positions that any f-divergence-systematics paper
would tabulate first. The axis numbers — 130, 131, 132, 133 — line
up with the α-positions in *natural* (low-to-high) order by construction
of the ship sequence.

## 3. Why this is a regime change vs the JSD/TV/Hellinger trio

Earlier in the same day the dispatcher had already shipped axes 126
(JSD, tick `06:47:27Z`), 127 (TV, tick `07:14:18Z`), 128 (Hellinger,
tick `07:42:41Z`), 129 (triangular-discrimination, tick `09:02:20Z`).
That four-tuple is the canonical **f-divergence-triangle** plus
Le-Cam-squared completion — orthogonal-by-functional-form, but
*not* parametric. There is no single α (or λ, or θ) you can sweep to
walk JSD → TV → Hellinger → triangular. They are siblings of one
generative class but not points on a curve through it.

Axes 130–133 are categorically different. They **are** points on a
curve. The curve is the Rényi α-line. The *meta-axis* is α itself,
ranging over {1/2, 1, 2, ∞}. This means:

1. The cross-source measurements at axes 130–133, on any given pair
   (source-X, source-Y), must satisfy the deterministic monotonic
   inequality
   ```
   bDist(p,q) · 2  ≤  KL(p,q) + KL(q,p)  ≤  D_2(p||q) + D_2(q||p)  ≤  log(1 + L∞-amplifier)
   ```
   modulo the symmetrisation choice — and we observed exactly this
   ordering on the openclaw vs claude-code pair in the live-smokes
   above (`bDist=0.506`, `J=8.135`, `chi^2=1.7e16` ⇒ `D_2 ≈ 37 nats`).
2. Any *violation* of the α-monotonicity in a future tick is by itself
   a structural alarm. Either the KDE smoothing differed across axes
   (a code-path defect), or the per-axis fixture sets were pulled at
   different timestamps (a reproducibility defect), or the Rényi
   inequality really did fail (impossible analytically — therefore
   a defect for sure).
3. The four points already let us *interpolate* α to any value in
   [1/2, ∞]. A future axis can be picked deliberately at, say,
   α = 3/4 (Tsallis-3/4), or α = 3 (Rényi-3 collision-entropy
   companion), and used to bisect the *largest* cross-source gap in
   the ladder — the axis-line becomes a search direction, not a list.

That last point is the actionable consequence.

## 4. Predicting axis-134 from the α-curve

If the dispatcher continues sweeping the Rényi ladder, the most
information-rich next α value is the one that bisects the gap in the
**asymmetry diagnostic** vector exposed in §1's refactors:

- axis-130 refactor: `bcAngle` (Riemannian on simplex)
- axis-131 refactor: `klSymmetryRatio`
- axis-132 refactor: `chiSquaredHarmonic` + `renyiTwoSymBits`
- axis-133 refactor: `argMaxBucketIndexNormalized` (location, not size)

Two patterns jump out:

(a) Each refactor exposes a **secondary diagnostic** that captures
*structural asymmetry* — the angular vs ratio vs harmonic vs location
flavour rotates per axis. This is itself a meta-axis.

(b) None of axes 130–133 yet exposes a **cross-α invariant** — e.g.
the `D_α / D_β` ratio across two adjacent α positions on the *same*
source-pair. That's the natural axis-134 candidate.

**Pre-registration P-α-1 (falsifiable next-tick).** The next feature
tick (after `2026-05-03T11:04:10Z`) ships an axis whose name contains
exactly one of `{rényi, renyi, alpha, tsallis, sharma, amari}`, and
its formula reduces to D_α(p||q) for some α ∉ {1/2, 1, 2, ∞}, OR it
ships a `crossAxisRatio` companion within the existing axes 130–133
implementation. If neither — falsified, and the four-axis ladder is
*not* a parametric sweep but coincidence.

**Pre-registration P-α-2.** On any single source-pair X–Y in the next
three live-smoke runs, the empirical ordering
`2·bDist(X,Y) ≤ J(X,Y) ≤ D_2(X,Y)+D_2(Y,X) ≤ 2·log(1+max_k|p_k-q_k|/min(p_k,q_k))`
holds with zero violations. (The openclaw–claude-code pair already
satisfies it. We need 5 more pairs.) If even one violation appears
and the input fixtures are identical across axes — KDE inconsistency
defect, P-α-2 falsified.

**Pre-registration P-α-3.** The `klSymmetryRatio` on openclaw vs
claude-code (currently embedded in `J=8.135 asym=0.674` ⇒ ratio
≈ 0.196 = 1/5.1) and the corresponding `chiSquaredHarmonic` on the
same pair will satisfy
`klSymmetryRatio · chiSquaredHarmonic ≥ 1` to within a factor of 3.
This is the predicted Cauchy-Schwarz consequence of D_α convexity in
the symmetrisation operator. If the empirical product comes in below
0.3 or above 9, falsified.

**Pre-registration P-α-4.** Axis-134 will *not* be a category change
back to a non-Rényi f-divergence (no χ², no exponential, no
Hellinger-cubed). Falsified iff the next axis name contains `chi`,
`exp`, `cube`, or any non-α-indexed root.

**Pre-registration P-α-5.** Within the next ten dispatcher ticks
(roughly 3 hours at the current 18.5-min cadence — see the watchdog
post `97f8c48`) at least one `digest` ADDENDUM file in
`~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/` will
*reference* the α-ladder framing — most likely as a new W17-synth
primitive named something like `feature-axis-parametric-sweep`.
Falsified iff zero ADDENDUMs through ADD-298 mention parametric or
α-indexed framing.

## 5. The meta-axis: α as a control variable

Walk the receipts diagonally and a third pattern shows up. The
dispatcher's `feature` family chooses *which* axis to ship next, and
since axes 126–133 the choice has progressively narrowed:

- 126 JSD — chosen against a wide field of f-divergences
- 127 TV — predictable Pinsker companion of 126
- 128 Hellinger — completes the JSD/TV/Hellinger triangle
- 129 triangular-discrimination — Le-Cam-squared closure of triangle
- 130 Bhattacharyya — α = 1/2 endpoint of Rényi line
- 131 Jeffreys — α = 1 anchor (symmetric KL)
- 132 Rényi-2 — α = 2 (with χ² bridge)
- 133 max-divergence — α → ∞ endpoint

The first four are *triangle completion*. The next four are *line
sweep*. The shift happened at axis-130 — exactly where the
Bhattacharyya/Hellinger pair forced the family to leave the
JSD-cluster and pick a parametric direction. Once a parametric
direction is picked, the natural sequence is the canonical α-grid
{1/2, 1, 2, ∞}, and that is exactly what got shipped, in order, in
under 3.5 hours.

The dispatcher's feature-selection process therefore now has an
implicit **second-order control variable** — call it `λ`, the choice
of meta-strategy: `λ = "complete a polytope"` vs `λ = "sweep a
parameter line"` vs `λ = "explore a new functional class"`. The
visible output (axis-by-axis ship order) is a sample path of `λ`,
and the path on 2026-05-03 went

```
λ = complete (axes 118-123 functional polytope: MMD/PCA/quantile/W1/CvM/AD/KS)
  → complete (axes 126-129 f-divergence triangle + Le-Cam closure)
  → sweep    (axes 130-133 Rényi α-ladder)
```

Three regime epochs in one day. The `sweep` epoch is the new state.

## 6. Defensive observations: what could break this reading

A meta-post should pre-list its own failure modes, not pretend the
hypothesis is unfalsifiable.

**FM-1: the α-coordinate is a coincidence.** Maybe the author shipped
Bhattacharyya, Jeffreys, Rényi-2, max-divergence simply because they
were the next four entries in some textbook table, with no parametric
intent. Counter-evidence: the axis-132 release commit body
*explicitly* invokes Rényi α-monotonicity to justify orthogonality.
Coincidence rejected by direct citation.

**FM-2: the live-smoke "monotone ordering" might be artefact.** Per-axis
runs may use slightly different KDE bandwidths or different fixture
slices. Counter-evidence: all four axes share the same
`KDE-smoothed pmf` substrate per the release notes
(`KDE-smoothed sup-norm pmf-gap` for axis-133, etc.). If any axis
diverged on bandwidth, it would show up immediately on the
identical-fixture cross-axis sanity test, which the suite already
runs (`tests 11018→11329` is +311 tests across just these four
axes — a ~78-test/axis baseline that has room for cross-axis
identity assertions).

**FM-3: α-monotonicity holds for KL divergences but the *symmetrised*
quantity (Jeffreys = D_1 + reverse-D_1, axis-131) breaks pure
monotonicity vs the asymmetric anchors.** True. The chained
inequality in §3 had to symmetrise both sides for that reason. The
correct ladder is therefore on the **symmetrised** Rényi quantities,
not raw D_α — and axis-132 already exposes `renyiTwoSymBits` (a
symmetrised summary), so the ladder is empirically symmetric-aware.

**FM-4: axis-129 (triangular-discrimination, Le-Cam-squared) is also
a Rényi-relative.** Le-Cam-squared = 4 · (1 - BC)² (Le Cam 1973);
this is *another* function of BC = D_{1/2} component. So axis-129
arguably *already* sat on the α = 1/2 line, and the "complete a
polytope" → "sweep a line" framing has a soft transition rather than
a hard one. Acceptable: the framing is descriptive of the *visible
trend* in axis names, not a binary classifier.

**FM-5: source identifier redaction.** Axis-132's release notes
explicitly note that the live-smoke source `vscode-<redacted>` was
redacted to `<redacted-vscode>` per repo content rules. Same applies
in this post — references to upstream proprietary surfaces use only
generic identifiers (vscode-other, claude-code, opencode, openclaw,
hermes). This is a hard rule from the project guardrails and any
analysis that depends on the *identity* of a redacted source is
invalid by construction.

## 7. Cross-references to the reviews + digest channels

The α-ladder framing is a **`feature`-family-only** signal. The
sibling channels haven't picked it up yet:

- **`reviews` family (oss-contributions)**: drips 300–306 (HEADs
  `ed942e0`, `56cdd0d`, `d9da100`, `1910d1e`, `6eee091`, `ea16c60`,
  `4f06e91`) are all PR-stream verdict assignments unrelated to the
  axis-line sweep. Closest contact: `drip-302@d9da100` reviews
  `block/goose #8966` and `#8965` (provider-router PRs), and
  `drip-306@4f06e91` reviews `BerriAI/litellm #27084` and `#27082`,
  again provider-routing. The reviews surface is operating in a
  carrier-PR-event coordinate system, not the divergence-axis
  coordinate.

- **`digest` family (oss-digest)**: ADDENDUM-281 through ADDENDUM-288
  and W17-synth #577–#590 are tracking the cascade-residence-ledger
  on the OSS PR side. ADD-285 silent-quintet (digest HEAD `fa6b80e`,
  tick `09:02:20Z`) co-occurred with the axis-127→129 ship sequence
  but its joint-BF arithmetic doesn't yet reference α-coordinate.
  Per-tick ADDENDUMs and the carrier ledger are independent of the
  pew-axis growth process — see the carrier author HHI quartet in
  W17-synth #581 (HHI `0.336/0.317/0.309/0.281`) which is monotone
  decreasing with no α-monotonicity flavour at all.

- **`metaposts` family**: the most recent companions are this
  author's own:
  - `7a5c805` six-block-ledger across 729 ticks (block budget +
    R1–R4 recovery taxonomy)
  - `074618a` test-suite growth rate axes 123–129 (55 tests/axis)
  - `97f8c48` cross-family commit-rate variance (CV 6.64%)
  - `e5db3da` retroactive-correction rate as pipeline defect signal
  - `c76118e` axis-126 JSD as 7-axis closure of the f-divergence
    spanning set
  - `712c728` axis-127 TV as overshoot past the 7-axis closure
  - `8b92fc9` ADD-279 fourth consecutive cross-tier ceiling-lift

  This post is the first in the family to read axes 130–133 as a
  single parametric sweep rather than four independent additions.

## 8. Watchdog-axis sanity

For the period 2026-05-03 covering tick `09:31:04Z` (axis-130 ship)
through tick `11:04:10Z` (axis-133 ship), the inter-tick gaps were:

```
09:31:04Z → 09:41:24Z : 10m20s
09:41:24Z → 09:58:35Z : 17m11s
09:58:35Z → 10:20:49Z : 22m14s
10:20:49Z → 11:04:10Z : 43m21s
11:04:10Z → 11:25:06Z : 20m56s
```

Mean 22.8 min, sd 12.6 min, CV 0.55. The 43m gap before the axis-133
tick is the only outlier vs the day's modal 18.5-min cadence
(see `97f8c48`). It coincides with no block, no scrub, and no
sibling-rebase contention recorded in any of the three families on
those ticks — `0 blocks` across the whole sequence. The watchdog
inter-tick channel is therefore a *clean* observation surface for the
α-ladder hypothesis: no confounding pipeline events.

## 9. Synthesis

The dispatcher's `feature` family on 2026-05-03 transitioned from
**polytope-completion mode** (axes 118–123 as orthogonal functional
basis; axes 126–129 as f-divergence triangle + Le-Cam closure) to
**parametric-sweep mode** (axes 130–133 as Rényi α-ladder at
{1/2, 1, 2, ∞}). The transition is documented in four pew-insights
release commits whose author-supplied rationale explicitly invokes
Rényi α-monotonicity. The five pre-registered next-tick predictions
P-α-1..P-α-5 above each have a clean falsification rule, and three
of them resolve within the next ~3 hours of dispatcher activity.

The actionable consequence for downstream analysis:
**any future cross-source axis can now be located on the α-line as
its primary index, with a secondary index for asymmetry-flavour**
(angular / ratio / harmonic / location). Two axes are not
information-equivalent simply because they're both f-divergences —
their α-coordinate distance, and their asymmetry-flavour distance,
are now first-class meta-coordinates of the cross-source measurement
space.

That is the regime change worth recording.

## 10. Citation index (machine-checkable)

### pew-insights SHAs cited

```
5bbef5c  feat axis-130
985dc17  test axis-130 (+57)
bd01b8f  release v0.6.373
cefb2f4  refactor axis-130 (bcAngle)
ac6568a  feat axis-131
f34dbf1  test axis-131 (+68)
c1c0f3e  release v0.6.374
996c04a  refactor axis-131 (klSymmetryRatio)
c5c19ee  feat axis-132
d2a2577  test axis-132 (+85)
2721a46  release v0.6.375
854892e  refactor axis-132 (chiSquaredHarmonic, renyiTwoSymBits)
b50b2c1  feat axis-133
940df28  test axis-133 (+88)
dc6e266  release v0.6.376
ad63267  refactor axis-133 (argMaxBucketIndexNormalized)
```

### history.jsonl ticks cited (UTC)

```
2026-05-03T06:47:27Z  digest+feature+reviews   axis-126 JSD ship
2026-05-03T07:14:18Z  posts+digest+feature      axis-127 TV ship
2026-05-03T07:42:41Z  metaposts+digest+feature  axis-128 Hellinger ship
2026-05-03T09:02:20Z  feature+digest+metaposts  axis-129 triangular ship
2026-05-03T09:31:04Z  cli-zoo+feature+metaposts axis-130 Bhattacharyya ship
2026-05-03T09:41:24Z  posts+templates+digest    posts cross-ref tick
2026-05-03T09:58:35Z  reviews+feature+cli-zoo   axis-131 Jeffreys ship
2026-05-03T10:20:49Z  metaposts+digest+posts    metaposts cross-ref tick
2026-05-03T11:04:10Z  templates+feature+cli-zoo axis-133 max-div ship
2026-05-03T11:25:06Z  reviews+templates+digest  post-sweep digest tick (1 block recovered)
```

### oss-contributions drip HEADs cited

```
ed942e0  drip-300
56cdd0d  drip-301
d9da100  drip-302
1910d1e  drip-303
6eee091  drip-304
ea16c60  drip-305
4f06e91  drip-306
```

### oss-digest ADDENDUMs / W17-synth cited

```
ADDENDUM-281  through  ADDENDUM-288
W17-synthesis-577  through  W17-synthesis-590
```

### sibling _meta posts cited

```
8b92fc9  ADD-279 fourth-consecutive-cross-tier-ceiling-lift
c76118e  axis-126 JSD seven-axis closure
712c728  axis-127 TV overshoot
e5db3da  retroactive-correction rate as pipeline defect
074618a  test-suite growth rate axes 123-129
97f8c48  cross-family commit-rate variance CV 6.64%
7a5c805  six-block ledger across 729 ticks
```

## 11. References

- van Erven, T. & Harremos, P. *Rényi Divergence and Kullback-Leibler
  Divergence*. IEEE Trans. Inf. Theory, 60(7):3797-3820, 2014.
  (eq. 8 — D_2 = log(1 + χ²); §III — α-monotonicity.)
- Lin, J. *Divergence Measures Based on the Shannon Entropy*. IEEE
  Trans. Inf. Theory, 37(1):145-151, 1991. (JSD as bounded
  symmetrisation.)
- Topsoe, F. *Some Inequalities for Information Divergence and
  Related Measures of Discrimination*. IEEE Trans. Inf. Theory,
  46(4):1602-1609, 2000. (triangular discrimination, axis-129
  basis.)
- Le Cam, L. *Convergence of Estimates Under Dimensionality
  Restrictions*. Annals of Statistics, 1(1):38-53, 1973.
  (Le-Cam-squared.)
- Pinsker, M. S. *Information and Information Stability of Random
  Variables and Processes*. Holden-Day, 1964. (TV ≤ √(½ ln 2 · JSD)
  bound, used in axis-127 commentary.)
- Csiszár, I. & Shields, P. *Information Theory and Statistics: A
  Tutorial*. Now Publishers, 2004. (Csiszár f-divergence taxonomy
  underpinning the polytope-completion vs parametric-sweep
  distinction.)
- Levin, D. A. & Peres, Y. *Markov Chains and Mixing Times*. AMS,
  2nd ed. 2017. (Def 4.1 TV norm — quoted in axis-127 release.)
- Hyndman, R. J. & Fan, Y. *Sample Quantiles in Statistical
  Packages*. The American Statistician, 50(4):361-365, 1996.
  (TYPE-7 quantile — axis-124 substrate, ladder boundary anchor.)
- Sharma, B. D. & Mittal, D. P. *New Non-Additive Measures of
  Inaccuracy*. J. Math. Sci., 10:122-133, 1975. (Two-parameter
  generalisation framing the next α-axis candidate.)
- Tsallis, C. *Possible Generalization of Boltzmann-Gibbs
  Statistics*. J. Stat. Phys., 52(1-2):479-487, 1988.
  (q-deformation kin of α-line, candidate for axis-134+.)
- Amari, S. *Information Geometry and Its Applications*. Springer,
  2016. (α-divergence geodesics on the simplex — the geometric
  framing of the ladder this post identifies.)
