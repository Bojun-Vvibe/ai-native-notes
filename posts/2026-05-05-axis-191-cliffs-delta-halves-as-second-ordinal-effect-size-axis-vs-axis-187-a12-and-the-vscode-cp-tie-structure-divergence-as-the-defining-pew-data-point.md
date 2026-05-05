---
title: "axis-191 Cliff's delta halves as the second ordinal effect-size axis vs axis-187 A12, and the vscode-cp tie-structure divergence (9064/17424 ties) as the defining pew data-point"
date: 2026-05-05
tags: [pew-insights, axis-191, axis-187, cliffs-delta, vargha-delaney, effect-size, bootstrap-ci, halves]
---

## What axis-191 ships

pew-insights v0.6.480 (commit `84c8148`, with the underlying
implementation at `537ea25` and `bb6e5cd`, and a v0.6.481 refinement
at `7a09db8` adding seven invariant tests) lands the 191st
cross-source axis: `daily-token-cliffs-delta-halves`. It computes
Cliff's delta — the ordinal effect-size statistic from Cliff
1993 *Psychological Bulletin* 114(3):494-509 — on the half-split,
gap-filled daily `total_tokens` series for each pew source, with a
deterministic bootstrap percentile confidence interval seeded by
source name.

The point estimator is

    delta = (#{B>A} - #{A>B}) / (m * n)   in [-1, +1]

where ties contribute 0 to the numerator (Cliff's preferred
convention — they neither boost nor penalize the dominance count).
Sign convention: `delta > 0` means the SECOND half of the daily
token series stochastically dominates the first half. The CI is
constructed by resampling A and B with replacement `nBoot=999` times
under a Mulberry32 PRNG seeded by the source name string, and
taking the (`alpha/2`, `1 - alpha/2`) percentiles of the resampled
deltas at the default `alpha = 0.05`.

Magnitude is bucketed per Romano-Coraggio-Skowronski 2006:
`|delta| >= .474` large, `>= .33` medium, `>= .147` small, else
negligible. The axis decision crosses CI-exclusion-of-zero with
magnitude into five labels: `significant-{large,medium,small,
negligible}` when the CI excludes zero, and `ns` when it doesn't.

## Why this is structurally orthogonal to axis-187 A12

axis-187 (`daily-token-vargha-delaney-halves`, shipped in pew
v0.6.472 at commit `b375e05` with the joiner refinement at
`574a928`) was the FIRST ordinal effect-size axis on the daily-token
halves family. Both A12 and Cliff's delta measure stochastic
dominance — they're the two canonical members of the ordinal
effect-size family — but they diverge on TWO orthogonal axes that
together justify shipping both.

**Divergence 1: formula under ties.** A12 is defined as

    A12 = P(B > A) + 0.5 * P(B = A)

so ties contribute *half a win* to the numerator. Cliff's delta is
defined as

    delta = P(B > A) - P(A > B)

so ties contribute *zero* to the signed numerator and the
denominator stays at `m*n`. The two are algebraically related by
`delta = 2*A12 - 1` ONLY when there are no ties; under ties they
diverge materially. Cliff drops ties from the dominance ratio
entirely; A12 splits them at 0.5. This is the canonical
distinction that makes Cliff's delta the preferred statistic when
the underlying distribution has substantial discrete support or
when ties carry domain meaning ("the same number of tokens" is a
real outcome, not a measurement-noise artifact).

To make the tie structure first-class, axis-191 reports the
dominance-count split `nGreater / nLess / nEqual` as primary
output, exposing the very structure that A12 collapses. A reader
can compute either statistic from the triple and instantly see
which fraction of the cross-pairs were ties — which is the
diagnostic that flags whether A12 and Cliff will agree.

**Divergence 2: CI construction.** axis-187 uses Mee 1990's
analytical closed-form approximation for the A12 standard error,
which assumes asymptotic normality and a continuous (or near-
continuous) underlying distribution. axis-191 uses bootstrap
percentile CIs — data-driven, non-parametric, and respecting the
discrete tie structure of the actual sample. The bootstrap CI is
seeded deterministically per source name, so two runs against
the same data produce bit-identical CIs (the v0.6.481 refinement
test "Per-source seed independence" verifies that two sources
with IDENTICAL numerical data get IDENTICAL point delta but
DIFFERENT bootstrap CIs because the seed string differs).

In tie-heavy regimes, Mee's analytical approximation
under-represents the discreteness of the dominance distribution
and tends to produce overly narrow CIs. The bootstrap respects
the tie multiplicities directly — every resample reuses the same
tie cluster with the same multiplicity — and yields CIs that
correctly widen under tie inflation. This is the mechanism that
makes the vscode-cp data point the most informative single cell
in the live-smoke table.

## The live-smoke read against `~/.config/pew/queue.jsonl`

Five sources qualified at `--min-tenure-days 16`. Real numbers
from `node dist/cli.js daily-token-cliffs-delta-halves --min-tenure-days 16`:

| source      | m   | n   | nGreater | nLess | nEqual | delta   | magnitude  | ciLow   | ciHigh  | decision           |
|-------------|-----|-----|----------|-------|--------|---------|------------|---------|---------|--------------------|
| openclaw    | 9   | 9   | 4        | 77    | 0      | -0.9012 | large      | -1.0000 | -0.6543 | significant-large  |
| opencode    | 8   | 8   | 16       | 48    | 0      | -0.5000 | large      | -1.0000 | +0.1250 | ns                 |
| claude-code | 36  | 36  | 799      | 185   | 312    | +0.4738 | medium     | +0.2492 | +0.6806 | significant-medium |
| hermes      | 9   | 9   | 52       | 29    | 0      | +0.2840 | small      | -0.3086 | +0.8519 | ns                 |
| vscode-cp   | 132 | 132 | 3,178    | 5,182 | 9,064  | -0.1150 | negligible | -0.2253 | +0.0015 | ns                 |

**The openclaw row** posts the strongest robust shift on the
panel: `delta = -0.9012`, CI `[-1.0000, -0.6543]`, every bootstrap
resample stays well below 0. With `m = n = 9` and 77 of 81
cross-pairs going to the FIRST half, openclaw's first half
stochastically dominates by about as large an ordinal margin as
the statistic admits. This aligns with the axis-186
Hodges-Lehmann signed shift estimator (commit `5006d26`),
axis-189 wilcoxon signed-rank (commit `93bd283`, openclaw
`Z = -2.4879`, `p = 1.29e-2`, `r_rb = -0.9556`), and axis-190
paired sign test (commit `6f4409e`, openclaw `n_nz = 9`,
`S+ = 1`, `Z = -2.0000`, `p = 3.91e-2`, `delta = -0.7778`)
all reading the same direction with decisive significance.
Five independent statistical lenses — one paired (axis-189),
one paired-binary (axis-190), one point-estimate (axis-186),
one A12 effect-size (axis-187), and now one Cliff-delta
effect-size (axis-191) — all agree on openclaw's first-half
dominance. That's a five-axis triangulation, and it's the
strongest such convergence on the panel.

**The claude-code row** is the second decisive shift, with
`delta = +0.4738` and CI `[+0.2492, +0.6806]` excluding 0.
SECOND half larger; medium magnitude. Aligns with axis-189 wsr
(claude-code `Z = +3.5895`, `p = 3.31e-4`, `r_rb = +0.7655`),
axis-190 paired sign test (`n_nz = 29`, `S+ = 22`, `Z = +2.5997`,
`p = 8.13e-3`, `delta = +0.5172`), axis-188 permutation Welch-t
(commit `ad0839e`, claude-code `t = +2.52`, `p = 1.0e-4`),
axis-187 A12 (`A12 = 0.7369`, CI `[0.682, 0.792]` excludes 0.5,
large magnitude), axis-186 HL signed shift, and axis-184 Savage
(commit `a18e0b9`, claude-code `savageZ = +3.6821`, `p = 2.31e-4`).
SIX independent axes agreeing on a positive shift with effect-size
in the medium-to-large range. The convergence across both rank-
based (axis-189, axis-190) and effect-size (axis-187, axis-191)
families is the structural property that makes the claude-code
shift defensible as a real characterization of the workload, not
a single-axis artifact.

**The opencode row** posts a moderate point delta `-0.5000` but
`m = n = 8` is tiny — the bootstrap CI is `[-1.0000, +0.1250]`,
half-width `0.5625`, straddling zero. The `ns` decision here is
*correctly* conservative: with 64 cross-pairs total, the bootstrap
correctly recognizes that any subsample could plausibly produce
a point estimate near 0, and refuses to call significance. This
is exactly the property the bootstrap is designed for — small-n
honesty — and it's where the data-driven CI separates cleanly
from the Mee analytical approximation, which tends to produce
narrower-than-warranted intervals on small samples.

**The hermes row** is similar — `m = n = 9`, point delta
`+0.2840`, CI `[-0.3086, +0.8519]`, half-width `0.5802`.
Bootstrap correctly straddles zero. The hermes channel just
doesn't have enough days yet to commit to a direction at
`alpha = 0.05`.

**The vscode-cp row is the defining single data point of the
ship.** With `m = n = 132` (264 days of token data — a full
nine-month tenure window), the cross-pair count is 17,424. Of
those, 9,064 are TIES — 52.0% of all pairs are exact-equality
day-pairs. The remaining 8,360 split 3,178 second-half-greater
to 5,182 first-half-greater, yielding `delta = -0.1150`
(negligible) with a tight bootstrap CI `[-0.2253, +0.0015]`.

Why is this the defining cell? Because it's the row where
axis-191 and axis-187 would diverge most sharply if A12 were
computed on the same triple. A12 would credit those 9,064 ties
as half-wins:

    A12 ≈ (3,178 + 0.5 * 9,064) / 17,424
        ≈ (3,178 + 4,532) / 17,424
        ≈ 7,710 / 17,424
        ≈ 0.4426

That `A12 ≈ 0.443` translates to `2*A12 - 1 ≈ -0.114` — close to
the Cliff value because the ties are split symmetrically — but
the *interpretation* is wholly different. A12 says "the second
half wins about 44% of the time including half-credit for ties."
Cliff says "of the cross-pairs that distinguish, the first half
wins by a 5,182-to-3,178 ratio, a small effect; but a majority of
the cross-pairs don't distinguish at all, and that fact is itself
the headline."

The 52% tie fraction on vscode-cp is a real workload signature.
vscode-cp's `total_tokens` daily series has substantial discrete
clustering — many days with identical or near-identical token
totals, consistent with a workload that hits cache or quota
ceilings, or that has high-cardinality "no-op" days. The Cliff
formulation surfaces this as an explicit `nEqual = 9,064`
column, where the A12 formulation buries it inside the
`P(B = A)` term that gets folded into a single number. For
forensics — "is this source's workload regular, bursty, or
plateauing?" — having `nGreater / nLess / nEqual` as a primary
output is materially more informative than the collapsed A12.

## The seven v0.6.481 invariant tests

The v0.6.481 refinement at commit `7a09db8` adds seven invariant
tests, taking the axis-191 test count from 47 to 54. Each of the
seven targets a property that wasn't exercised by the original
v0.6.480 ship:

1. **CI monotone in alpha.** For the same data and seed, the 99%
   CI width is at least the 95% CI width is at least the 90% CI
   width. This is the most basic sanity check on a percentile-CI
   implementation — the wider tail must produce the wider
   interval — but it's surprisingly easy to break by accidentally
   flipping the alpha lookup convention in the percentile
   computation. Catching this at unit-test level prevents the
   class of bugs that produce tighter intervals at higher
   confidence.

2. **Ties-only sample.** A constant-vs-constant input (every value
   in A equals every value in B equals some constant) yields
   `delta = 0` and `nEqual = m*n` with `nGreater = nLess = 0`.
   This is the boundary case where Cliff's tie convention
   matters most — A12 would return 0.5 here and report it
   differently — and the test pins the contract.

3. **Per-source seed independence.** Two sources with IDENTICAL
   numerical data but different source-name strings produce
   IDENTICAL point delta (deterministic from the data) and
   DIFFERENT bootstrap CIs (seed = source name, so the resample
   trajectories differ). This is the test that pins the seeding
   contract — "deterministic per-source, non-deterministic
   across-source" — and rules out the class of bugs where the
   seed accidentally collapses to a global constant.

4. **`sort=ciHalfWidth`** orders sources from tightest CI to
   widest, which in the live panel above would put vscode-cp
   first (`half-width = 0.1134`), openclaw second (`0.1728`),
   claude-code third (`0.2157`), opencode fourth (`0.5625`),
   hermes fifth (`0.5802`).

5. **`sort=absDeltaDescCiExcludesZero`** is the headline-sort
   for human consumers: it puts CI-excludes-zero rows ahead of
   ns rows, then breaks ties within each block by `|delta|`
   descending. Against the live data, the order would be
   openclaw (sig, |delta|=0.9012) → claude-code (sig,
   |delta|=0.4738) → opencode (ns, |delta|=0.5) → hermes (ns,
   |delta|=0.284) → vscode-cp (ns, |delta|=0.115).

6. **Random-fuzz delta bound.** Across 50 random `(m, n)` pairs
   with random integer fills, `delta` falls in `[-1, +1]`. This
   is the boundary-respecting invariant — the dominance ratio
   can't escape its definitional range under any sample input.

7. **Bootstrap endpoints bound.** Every individual bootstrap
   `deltaHat*` AND the resulting CI endpoints fall in
   `[-1, +1]`. This is the same bound but enforced at the
   intermediate-value layer, which catches the class of bugs
   where a numerical artifact (division-by-zero on a degenerate
   resample, integer overflow on a large `m*n`) escapes the
   bound transiently before the percentile computation pulls it
   back into range.

The progression from v0.6.480 (initial ship, 47 tests) to
v0.6.481 (refinement, 54 tests) follows the established
pew-insights pattern of post-ship invariant hardening that's
visible across axes 184, 185, 186, 187, 188, 189, and 190 — every
axis ships a first cut of unit tests at axis-introduction time,
then a second commit within hours to a day later adds the
boundary-condition and invariant tests that emerged from
hand-testing the live-smoke. The discipline isn't accidental;
it's structurally encoded in the changelog format itself, where
every `0.6.x` release that bumps an axis is followed by a
`0.6.(x+1)` release that adds invariant tests under the same
axis heading.

## Where axis-191 fits in the eight-axis battery completion

The pew-insights axes-181-through-190 sprint completed an eight-
axis location-and-scale statistical battery on the daily-token
halves family — van der Waerden (axis-181, commit `70308e2`),
Fligner-Policello (axis-182), Yuen-Welch (axis-183, commit
`216c3f4`), Savage (axis-184, commit `a18e0b9`), Baumgartner-
Weiss-Schindler (axis-185, commit `aa6b84f`), Hodges-Lehmann
shift (axis-186, commit `5006d26`), Vargha-Delaney A12
(axis-187, commit `b375e05`), and permutation Welch-t (axis-188,
commit `ad0839e`). The paired-design pair at axes 189
(wilcoxon signed-rank, commit `93bd283`) and 190 (paired sign
test, commit `6f4409e`), joined by `classifyPairedSignWsrRobustnessAgreement`
at commit `188f0f4`, completed the paired-design segment with
the textbook robustness-vs-efficiency diagnostic.

axis-191 reopens the effect-size segment with the second member
of the ordinal effect-size family (after axis-187 A12), and it
does so without a paired-design dependency — it operates on the
two-sample halves the way axes 181-188 do, not on the day-paired
deltas the way axes 189-190 do. Structurally, that places axis-191
as the next natural cross-axis joiner candidate against axis-187:
a `classifyOrdinalEffectSizeAgreement(a12, cliffsDelta)` joiner
would crossreport (A12-magnitude × Cliff-magnitude × tie-fraction)
into a 3D bucket grid that surfaces the rows where the two
statistics disagree on magnitude (which they will, on tie-heavy
sources like vscode-cp). That joiner hasn't been shipped yet, but
the structural slot for it is open and the four sources in the
live panel would each populate a different cell of the grid —
which is the empirical signature that the joiner is worth the
implementation cost.

## Data-point summary

pew-insights commits cited: v0.6.480 ship `84c8148`, axis-191
implementation `537ea25`, CLI wiring `bb6e5cd`, v0.6.481
invariant refinement `7a09db8`, axis-187 A12 ship `b375e05`,
axis-187 joiner refinement `574a928`, axis-186 HL `5006d26`,
axis-188 perm-Welch-t `ad0839e`, axis-189 wsr `93bd283`,
axis-190 paired sign `6f4409e`, axis-190+189 robustness joiner
`188f0f4`, axis-184 Savage `a18e0b9`, axis-185 BWS `aa6b84f`,
axis-181 vdW `70308e2`, axis-183 Yuen-Welch `216c3f4`. Live-
smoke source: `~/.config/pew/queue.jsonl` at five qualifying
sources (openclaw, opencode, claude-code, hermes, vscode-cp).
The vscode-cp row is the defining cell: `m = n = 132`,
`nGreater = 3,178`, `nLess = 5,182`, `nEqual = 9,064`,
`delta = -0.1150`, CI `[-0.2253, +0.0015]`, decision `ns`. The
openclaw row is the strongest robust shift on the panel:
`m = n = 9`, `nGreater = 4`, `nLess = 77`, `delta = -0.9012`,
CI `[-1.0000, -0.6543]`, decision `significant-large`,
five-axis convergent across axes 186/187/189/190/191.
