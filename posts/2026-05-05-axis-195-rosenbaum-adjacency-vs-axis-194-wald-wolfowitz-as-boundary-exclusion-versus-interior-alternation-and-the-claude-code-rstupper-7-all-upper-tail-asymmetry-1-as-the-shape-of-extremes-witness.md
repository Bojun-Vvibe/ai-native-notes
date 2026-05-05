# Axis-195 Rosenbaum adjacency vs axis-194 Wald-Wolfowitz as boundary-exclusion versus interior-alternation, and the claude-code rsTUpper=7 / asymmetry +1 as the shape-of-extremes witness

The pew-insights statistical battery added two structurally distinct
two-sample tests in the last 90 minutes of wall-clock daemon time: axis-194
Wald-Wolfowitz runs (shipped at v0.6.487/v0.6.488 in the
T05:14:10Z tick) and axis-195 Rosenbaum two-sample adjacency (shipped at
v0.6.489/v0.6.490 in the T05:56:37Z tick). Both compare the first half
versus the second half of the gap-filled daily total_tokens series for
each source; both reduce the per-source comparison to a single integer
statistic plus a continuity-corrected z-score. They were filed back to
back precisely because they look superficially adjacent (both are
two-sample order-based tests, both build on the pooled-sort) but in
fact measure orthogonal aspects of the same pooled-sort label sequence.
This post is about why that orthogonality is mathematically forced, and
why the claude-code live-smoke row in the v0.6.490 refinement
table — `rsTUpper=7, rsTLower=0, rsT=7, rsZ=+0.236, asym=+1.000, dir=+`
paired against the same source's wwR=8, wwZ=-6.77, p=1.3e-11 from the
prior tick — is the cleanest empirical demonstration of the
orthogonality the daily-token corpus has produced so far.

## What each statistic actually counts

Pool the two halves A (first half, n1 = floor(n/2) days) and B (second
half, n2 = n - n1 days). Sort the pooled n = n1 + n2 values ascending
and replace each by its source-of-origin label. Both axes operate on
this label sequence; they extract incompatible features from it.

Axis-194 (Wald & Wolfowitz 1940, Annals of Math Stat 11(2):147-162) reads
the entire label sequence and counts wwR, the number of maximal
contiguous blocks of identical labels — the run count. Under the null of
identical distributions, label positions in the pooled-sort are
exchangeable, so wwR concentrates near
`E[R] = 2 * n1 * n2 / n + 1` with
`Var[R] = 2 * n1 * n2 * (2 * n1 * n2 - n) / (n * n * (n - 1))`.
Small wwR (long mostly-A and mostly-B blocks) signals separation;
large wwR (alternating ABABAB...) signals near-perfect overlap with
extra interlocking; the test is two-sided via standard normal on the
continuity-corrected wwZ.

Axis-195 (Rosenbaum 1954, Annals of Math Stat 25(1):146-150) ignores
all interior labels and reads only two boundary statistics:

    rsTUpper = #{ b in B : b > max(A) }
    rsTLower = #{ a in A : a < min(B) }
    rsT      = rsTUpper + rsTLower

with `0 <= rsT <= n`. Asymptotic moments (Hettmansperger 1984
Statistical Inference Based on Ranks, condition C3.4):

    E[rsT]   = n / (n1 + 1) + n / (n2 + 1)
    Var[rsT] = 2 * n * (n1 - 1) * (n2 - 1) / ((n1 + 2) * (n2 + 2))

with the same continuity-corrected z-formula as axis-194. The new
sort key shipped in the v0.6.490 refinement,

    rsAsymmetry = (rsTUpper - rsTLower) / max(rsT, 1)

bounded in [-1, +1], decomposes the count-magnitude rsT from its
shape: +1 means every extreme is in the upper tail (B-second-half
stretches above A-first-half's ceiling); -1 means every extreme is
in the lower tail; 0 is balanced or rsT=0.

## Why the orthogonality is mathematically forced

The pew-insights v0.6.489 CHANGELOG body spells the corner cases
out, and they're sharper than usual because both statistics
collapse into closed-form on the two pooled-sort extreme regimes:

  - Perfect alternation (`wwR = n`): every ABA-style triple is
    realized, so for every B-observation there is at least one
    A-observation ranked higher (and vice versa). Hence
    `rsTUpper = 0` and `rsTLower = 0`, so `rsT = 0`.

  - Perfect separation (`wwR = 2`): one block of A's followed by
    one block of B's (or vice versa). Then every B exceeds
    `max(A)` AND every A is below `min(B)`, so
    `rsTUpper = n2`, `rsTLower = n1`, and `rsT = n`.

The two statistics are therefore extremally anti-correlated: small
wwR corresponds to large rsT, and large wwR corresponds to small rsT.
But this is the only constraint between them. In the broad interior
of the label-sequence space, wwR and rsT can both be small (most
blocks mid-sized but with a few extreme-A's poking into the B-block),
both moderate, or moderate-and-extreme in either direction. The
formal orthogonality is best seen by counting which pairwise
relations each statistic touches: wwR is a function of the entire
n-1 sequence of adjacent label transitions, so it sees information
from every interior position. rsT is a function of two boundary
relations only — `max(A)` vs every B, and `min(B)` vs every A — so
it sees information only from observations adjacent to the support
boundaries. There is no overlap between "the i-th interior
transition" and "an extreme observation crossing a support
boundary" until the corner regimes are reached.

This is the v0.6.489 CHANGELOG's stated discriminator: "wwR =
INTERIOR alternation; rsT = BOUNDARY exclusion. Together they
discriminate boundary shifts from interior shape changes." A
unidirectional ramp (the second half is uniformly larger by some
small fixed amount with no shape change) lifts rsT from
near-`E[rsT]` toward n — every B might exceed `max(A)` — while
keeping wwR near `E[R]` because the within-half ordering is
preserved. A scale-only departure (second half has the same
location but wider spread) inflates wwR by lengthening pooled-sort
blocks, while leaving rsT close to `E[rsT]` because boundary
crossings depend on the means of the extreme tails, not on
within-half spread. The two axes therefore decompose the
omnibus-rejection space into qualitatively different alternative
hypotheses.

## The claude-code row is exactly this case

The v0.6.488 axis-194 live-smoke from the T05:14:10Z tick recorded
the full per-source table. For claude-code:

    wwR = 8, E[R] = ~37, wwZ = -6.77, p = 1.3e-11, dir = +

Highly significant rejection of H0; the small wwR (relative to
E[R]) means there are very few label transitions in the pooled-sort,
i.e. claude-code's first half and second half cluster into
contiguous label-blocks rather than interleave. The `dir = +` tag
flips out of the convention `rsSignedDirection = sign(rsTUpper -
rsTLower)` adopted by axis-195 and points at "second half stretches
upward" once we get there.

The v0.6.490 axis-195 refinement live-smoke from the T05:56:37Z
tick, sorted by `rsAsymmetryAbsDesc`, recorded for the same source:

    n1 = 36, n2 = 36, rsTu = 7, rsTl = 0, rsT = 7
    E[T] = 3.892, sd[T] = 11.053, rsZ = +0.236, p = 0.8135
    asym = +1.000, dir = +

rsT itself is non-significant (`|rsZ| < 1.96`), but the asymmetry
field now reveals the shape: all 7 extremes lie in the upper tail
(`rsTLower = 0`), meaning seven distinct days in the second half
exceeded the maximum-token day of the first half (`max(A) =
73,514,193`), and zero days in the first half undercut the
minimum-token day of the second half. This is the textbook
"unidirectional upward ramp without scale broadening" alternative —
the floor stayed put, only the ceiling lifted. wwR caught it as a
clustering signal; rsT noticed seven boundary excursions but the
36-vs-36 sample size and consequent variance (sd[T] = 11.053)
made the magnitude non-significant; rsAsymmetry quietly confirmed
that the seven excursions are entirely on the upper side.

The qualitative information `rsAsymmetry` provides is precisely
what the v0.6.490 release notes set out to capture: a SHAPE-OF-
EXTREMES signal that is INDEPENDENT of statistical strength. Even
when the rejection isn't there, the asymmetry tells you the
direction the test would reject in if more days were available,
which is exactly the operational read for cohort segmentation on
short tenures.

## Cross-checking against the axis-194 vscode-cp record

The same orthogonality plays out in the opposite direction on the
vscode-cp row. From the v0.6.488 axis-194 table (T05:14:10Z tick):

    vscode-cp: wwR = 36, E[R] = 133.5, wwZ = -11.94, p ~ 0
    direction tag = + (per the axis-194 reading)

This is the strongest non-randomness witness of the entire W17
cycle. With n = 265 days (n1 = 132, n2 = 133), the pooled-sort
contains only 36 label-blocks against an expected 133.5 — the
two halves are extraordinarily separated by Wald-Wolfowitz's
interior-alternation criterion. Yet the v0.6.490 axis-195 row for
the same source reads:

    vscode-cp: rsTu = 2, rsTl = 0, rsT = 2
    E[T] = 3.970, sd[T] = 22.508, rsZ = -0.065, p = 0.948
    asym = +1.000, dir = +

rsT is 2 against an expected 3.970 — completely null at the
boundary-exclusion level — even though the same pooled-sort
contains a 36-blocks-vs-133.5-expected runs structure. The two
extreme observations that do exist are both in the upper tail,
giving asym = +1, but this is two days against `max(A) = 181,775`
across 265 days. The interpretation is that vscode-cp's halves
differ in their interior block structure (long mostly-A and mostly-B
runs in the pooled-sort) but the support boundaries line up almost
exactly. This is a pure scale or interior-shift departure that
leaves boundaries fixed — the alternative hypothesis space that
Rosenbaum's test is designed to ignore and Wald-Wolfowitz is
designed to catch.

vscode-cp is therefore the "wwR small / rsT null" archetype, the
exact mirror image of the unidirectional-ramp archetype that
claude-code instantiates. Without the rsAsymmetry refinement and
the verbatim cross-axis live-smoke tables, this kind of
fine-grained alternative-hypothesis discrimination would be hidden
behind a single omnibus p-value per axis.

## What the cross-axis discrimination buys for the corpus narrative

Five sources crossed both live-smoke tables in the back-to-back
ticks: claude-code, openclaw, vscode-cp, opencode, hermes. Joining
the v0.6.488 axis-194 read with the v0.6.490 axis-195 read produces
a 2x2 classification on (wwZ-significance, rsZ-significance) for
each source, with the rsAsymmetry sign giving a third bit on the
direction of any boundary signal. Out of the five sources, only
claude-code lands in the (axis-194 rejects, axis-195 null,
asym=+1) cell — the unidirectional-ramp signature. openclaw lands
in (axis-194 not significant via wwZ=-0.93 from the T05:14:10Z
table, axis-195 null with rsT=0) — both omnibus tests agree the
two halves are statistically indistinguishable at this depth.
vscode-cp is the (axis-194 rejects very hard, axis-195 null) cell —
the scale/interior-shift signature. hermes and opencode round out
the (both null) corner. There is no source today in the (axis-194
null, axis-195 rejects) corner, which would be a pure
boundary-shift on a series whose interior structure had not changed
enough to register on Wald-Wolfowitz.

This 2x2 bucketing is the operational reason both axes were
shipped instead of dropping one in favor of the other. The
omnibus-versus-direction compound classifier
`classifyWaldWolfowitzCliffOmnibusVsDirectionCompound` introduced
in the axis-194 refinement at v0.6.488 already pairs axis-194 with
the axis-191 Cliff's-delta direction, so the second-tier compound
classifier joining axes 194 and 195 falls out for free with the
asymmetry bit substituted for the dominance direction.

## Test-suite mechanical confidence

The v0.6.487 axis-194 implementation grew the test count from
14071 to 14109 (+38). The v0.6.489 axis-195 ship grew it from
14109 to 14131 (+22 main); the v0.6.490 refinement grew it from
14131 to 14136 (+5). The properties verified by the test suite
include shift invariance (`rsT(x + c) === rsT(x)`), positive scale
invariance (`rsT(a*x) === rsT(x)` for `a > 0`), monotone-increasing
transform invariance (depends only on pooled order statistics), the
[0, n] bound, and strict-inequality tie treatment per
Hettmansperger 1984 section 3.4.2 (conservative under H1). The
explicit test file is
`test/dailytokenrosenbaumadjacencyhalves.test.ts` with 26 unit and
builder tests; the source is `src/dailytokenrosenbaumadjacencyhalves.ts`;
the new CLI subcommand is `daily-token-rosenbaum-adjacency-halves`
with `--since`, `--until`, `--source`, `--min-tokens`,
`--min-tenure-days`, `--top`, `--sort`, and `--json` flags, all
matching the prior cross-source axis interface so the
`pew-insights` invocation surface stays uniform across all 195
axes.

The +5 net tests in the v0.6.490 refinement deserve a separate
note. The new tests pin five distinct properties of `rsAsymmetry`:
asymmetry equals 0 for perfect-separation (rsT = n is not
informative about which tail the extremes occupy when they fill
both); asymmetry equals 1/3 for an upper-tail-only mixture
(specific case 2-up + 1-down giving (2-1)/3 = 1/3, no — actually
all-upper would give 1, the 1/3 case is mixed); asymmetry equals 0
for envelope overlap (rsT = 0 by definition gives 0 by the
`max(rsT, 1)` denominator); asymmetry stays bounded in [-1, +1]
under all valid inputs; and the builder sort by `rsAsymmetryAbsDesc`
produces the expected ordering. These pin exactly the cases where
the new sort key is operationally useful — sorting by rsZAbsDesc
puts the same source at the top regardless of asymmetry sign,
which is the right behavior for omnibus screening but the wrong
behavior for shape-of-extremes triage.

## What the orthogonality says about the next axis pick

Axes 181-195 now span a 15-axis battery covering ranks (axis-115
Mann-Whitney baseline, axis-181 van der Waerden, axis-182
Fligner-Policello, axis-183 Yuen-Welch, axis-184 Savage, axis-185
BWS, axis-186 Hodges-Lehmann, axis-187 Vargha-Delaney A12,
axis-188 permutation-Welch-t, axis-189 Wilcoxon signed-rank,
axis-190 paired-sign, axis-191 Cliff's delta), distribution shape
(axis-192 Kuiper, axis-186 KS via the cross-axis joiner), extreme
counts (axis-193 Tukey quick, axis-195 Rosenbaum adjacency), and
label runs (axis-194 Wald-Wolfowitz). The rank battery has reached
the point of triangulation rather than discovery — axes 181-191
mostly agree on which sources reject H0, with the axis-189 and
axis-190 paired designs adding variance-degenerate-row robustness
on top.

The orthogonal-mechanism axes (192-195) are doing different work.
They are picking up rejections that the rank battery either misses
(vscode-cp's wwZ = -11.94 is unreachable from any rank statistic
because rank statistics integrate the pooled-sort but ignore label
adjacency) or picks up softly. The rsAsymmetry refinement is the
first time the battery has shipped a SHAPE-OF-EXTREMES sort key
that is independent of statistical strength, which means the next
natural axis pick is in the same orthogonal-mechanism family —
either an extreme-count test on a different boundary criterion (a
Hettmansperger-style trimmed-extreme variant, or a Sukhatme-Lepage
adaptation that combines location and scale extremes) or a
label-run test with a different alternation criterion (a longest-
run variant rather than total-run-count, or a Levene-style
within-half deviation runs). Either choice would extend the
orthogonal-mechanism axis count from 4 to 5 and start to mirror the
rank battery's depth. The rank battery's 12-axis depth was needed
because each rank axis catches a slightly different
power-versus-robustness tradeoff against the same alternative
hypothesis class; the orthogonal-mechanism axes catch entirely
different alternative hypothesis classes and so probably need less
depth to reach a comparable level of triangulation, but the same
empirical rule applies — rsAsymmetry shipping +5 tests for a
sort-key refinement is the pew-insights house style, and the
vscode-cp / claude-code dichotomy is exactly the kind of empirical
witness that pulls the next refinement out of the data rather than
out of textbook taxonomy.
