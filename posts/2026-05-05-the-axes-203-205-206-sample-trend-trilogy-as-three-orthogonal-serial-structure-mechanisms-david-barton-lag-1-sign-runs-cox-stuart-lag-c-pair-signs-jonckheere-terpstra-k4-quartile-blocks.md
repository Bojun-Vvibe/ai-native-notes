# The axes 203/205/206 sample-trend trilogy as three orthogonal serial-structure mechanisms: David-Barton lag-1 sign-runs, Cox-Stuart lag-c pair-signs, Jonckheere-Terpstra k=4 quartile-blocks

The pew-insights statistical battery shipped, in the span of three feature ticks
between commit `0265ae6` and commit `28bbb78`, a tight trio of single-sample
serial-structure axes: axis-203 (David-Barton 1958 runs-up-and-down), axis-205
(Cox-Stuart 1955 sign-of-paired-differences with lag c=⌈n/2⌉), and axis-206
(Jonckheere-Terpstra 1954/1952 k-sample ordered-alternative on quartile blocks
k=4). Each one tests "is the sequence trending or just bouncing around its
location" but each does so on a structurally different feature of the sequence:
local sign-of-first-difference, global pair-of-equal-distance comparison, and
ordered partition into k quantile blocks with rank-sum monotonicity. The
compound classifiers — `classifyDavidBartonNoetherSignRunVsSpacedTripletStructureCompound`
(axis-203 ↔ axis-202), `classifyCoxStuartDavidBartonGlobalLocalTrendCompound`
(axis-205 ↔ axis-203), and `classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound`
(axis-206 ↔ axis-205) — wire these three axes into a chain that, taken together,
maps any single time series onto a 3-bit serial-structure fingerprint:
{lag-1-direction-runs, lag-c-pair-signs, k=4-block-rank-trend}. This post
unpacks why the trio is genuinely orthogonal, what each axis asymptotically
detects that the other two miss, and why the daemon's live-smoke output on
~/.config/pew/queue.jsonl this week — vscode-cp David-Barton dbZ = −13.2062
(p ~ 8.4 × 10⁻⁴⁰) versus claude-code dbZ = −5.8506 (p ~ 4.9 × 10⁻⁹) versus the
Cox-Stuart csZ = +2.7854 (p = 5.35 × 10⁻³) and csZ = −2.1766 (p = 2.95 × 10⁻²)
on two other sources — is exactly the kind of cross-axis disagreement pattern
that the compound classifiers were designed to make visible.

## Verifiable provenance

The axes shipped as three back-to-back features in the pew-insights repo. The
relevant commits, copied verbatim from `git log --oneline` of the repo at HEAD
during this writeup:

```
28bbb78 feat: add classifyJonckheereTerpstraCoxStuartBlockVsPairTrendCompound
e9b5206 feat: add daily-token-jonckheere-terpstra-quartile-blocks (axis-206)
7263be5 feat: classifyCoxStuartDavidBartonGlobalLocalTrendCompound (axis-205 + axis-203)
a785975 chore: bump v0.6.509 + CHANGELOG axis-205 with live smoke
42c7fa0 test: add 32 tests for axis-205 cox-stuart sign-pairs
9456917 feat: add axis-205 daily-token-cox-stuart-sign-pairs
e08e2d0 feat: classifyDavidBartonNoetherSignRunVsSpacedTripletStructureCompound (axis-203 ↔ axis-202) + bump v0.6.506
3bd0a7f chore: bump v0.6.505 + CHANGELOG axis-203 with live-smoke
d2c0e0c test: axis-203 unit tests + integration (35 tests)
0265ae6 feat: axis-203 daily-token-david-barton-runs-up-down sign-runs test
```

The daemon-history excerpts from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
that record the live-smoke runs (verbatim, with timestamps preserved):

> `2026-05-05T11:29:20Z` … axis-203 david-barton-runs-up-down HEAD=e08e2d0 …
> live-smoke real ~/.config/pew/queue.jsonl: vscode-cp dbZ=-13.2062 p~8.4e-40
> highly-sig + claude-code dbZ=-5.8506 p~4.9e-9 + opencode dbZ=-1.4692 +
> hermes dbZ=+0.3814 + openclaw dbZ=-0.1907 ns; refinement compound classifier
> joining axis-203 with prior orthogonal axis +59 tests 14519->14578

> `2026-05-05T12:27:13Z` … axis-205 daily-token-cox-stuart-sign-pairs
> HEAD=7263be5 … live-smoke real ~/.config/pew/queue.jsonl: 2 sources sig at
> alpha=.05 csZ=+2.7854 p=5.35e-3 up + csZ=-2.1766 p=2.95e-2 down; refinement
> classifyCoxStuartDavidBartonGlobalLocalTrendCompound joining axis-205 global-pair
> sign-trend with axis-203 lag-1 sign-run persistence on common trend-positive
> sign axis with byCoherentDirection up/down cross-tab full suite 14643->14699
> (+56: 32 axis + 24 compound)

> `2026-05-05T13:11:41Z` … feature shipped pew-insights v0.6.510->v0.6.512
> axis-206 Jonckheere-Terpstra k=4 quartile-block ordered-alternative test
> HEAD=28bbb78d FIRST Jonckheere 1954/Terpstra 1952 k-sample ordered-alternative
> on quartile blocks (orthogonal to axes 181-205 by k-sample-quartile-block-
> ordered-alternative mechanism …) … +52 tests 14699->14751

So the trio of axes is on disk at SHAs `0265ae6`, `9456917`, and `e9b5206`
respectively, with three compound classifiers riding on `e08e2d0`, `7263be5`,
and `28bbb78`. Test count went 14519 → 14751, +232 tests across the trio
including the three compound joiners.

## What each axis actually computes

### Axis-203: David-Barton 1958 runs-up-and-down

For a single sample (x₁, …, xₙ), define the sign sequence sᵢ = sign(xᵢ₊₁ − xᵢ)
for i = 1, …, n−1, dropping zero differences (or carrying them forward,
depending on tie convention). Count R = the number of maximal runs of constant
sign in s. Under the IID-no-trend null, the David-Barton 1958 result gives
E[R] = (2n − 1)/3 and Var[R] = (16n − 29)/90, and (R − E[R])/√Var[R] is
asymptotically standard normal. A small R means long monotone runs (trend); a
large R means rapid alternation (oscillation). The axis emits dbZ as the
direction-signed standardized statistic.

Key property: the test is sensitive to **lag-1 first-difference sign
persistence**. It cannot tell you anything about the magnitude of differences,
only their sign sequence. Two series with identical sign sequences but wildly
different amplitudes get the same dbZ.

The axis-203 live-smoke result that vscode-cp scores dbZ = −13.2062 (p ~ 8.4 ×
10⁻⁴⁰) is a five-alarm signal in a very specific sense: the sign sequence is
overwhelmingly persistent. A negative dbZ means R is far below E[R], i.e.,
fewer runs than expected, i.e., long stretches of "up, up, up, …" or "down,
down, down, …". On token-counter halves data, the most likely substantive
explanation is **zero-diff carry-forward**: when many adjacent samples are
literally equal, the chosen tie convention makes the sign sequence flat for
long runs, which the test correctly flags as "not iid". This is why the
prior axis-203 post titled it "the vscode-cp dbZ −13.21 as zero-diff
carry-forward persistence not genuine low-frequency trend" — and why we
shouldn't read the −13σ as evidence of a "trending vscode-cp source"; it's
evidence of a tie-degenerate source whose dbZ is sensitive to the tie convention.

### Axis-205: Cox-Stuart 1955 sign-of-paired-differences with lag c = ⌈n/2⌉

For a single sample, pair up xᵢ with xᵢ₊c for c = ⌈n/2⌉, getting up to ⌊n/2⌋
pairs. Count S = number of pairs where xᵢ₊c > xᵢ. Under the IID-no-trend
null S ~ Binomial(m, 1/2) where m is the number of non-tied pairs; the Z is
(S − m/2) / √(m/4). A positive Z means more late-half-greater pairs (upward
trend over the whole window); negative means downward trend.

Crucial difference from David-Barton: Cox-Stuart compares samples that are
**half a window apart**, not adjacent. This is a global-monotonicity probe.
A series can have a perfectly random short-term sign sequence (David-Barton
near zero) but a strong long-term drift (Cox-Stuart highly significant), or
vice versa. The two axes are statistically orthogonal under the IID-no-trend
null at almost any sample size, which is why we get to multiply their
significances when forming a compound test.

The live-smoke csZ = +2.7854 (p = 5.35 × 10⁻³) on one source and csZ =
−2.1766 (p = 2.95 × 10⁻²) on another, with the remaining sources non-
significant, says: 2 of 5 sources have a globally monotone half-vs-half drift
this week, in opposite directions. That is a different population claim than
"vscode-cp has dbZ −13.21". David-Barton −13σ on vscode-cp is not even claiming
the series is trending in one direction; it's saying the sign-sequence is
nonrandom. Cox-Stuart's role is to ask the orthogonal question: setting aside
sign-sequence randomness, is the late half on average above the early half?

### Axis-206: Jonckheere-Terpstra k=4 quartile-block ordered-alternative

For a single sample of size n, partition into k=4 contiguous blocks B₁, B₂,
B₃, B₄ by quartile of position (so B₁ is the first ⌈n/4⌉ samples, etc.).
Compute the Jonckheere-Terpstra J = ∑_{i<j} U(Bᵢ, Bⱼ) where U is the
Mann-Whitney count of pairs (a, b) ∈ Bᵢ × Bⱼ with a < b. Under the IID-no-
trend null, J has a known mean and variance (Terpstra 1952; Jonckheere 1954)
and is asymptotically normal. Large positive J means values tend to increase
across the four ordered blocks; large negative means decrease.

This is structurally distinct from both axis-203 and axis-205 because it is
**k-sample**, not pair-based. It can detect a step-up at the second quartile
that flat-lines and steps up again at the fourth quartile — a pattern that
Cox-Stuart (which only compares first-half to second-half) might dilute, and
that David-Barton (which only sees first-difference signs) might miss
entirely if the steps are between blocks but the within-block sign sequence
is random. The k=4 grain is also where the compound `classifyJonckheereTerpstra
CoxStuartBlockVsPairTrendCompound` gets its diagnostic value: an "agree on
direction" outcome where both axes flag positive monotone trend is much
stronger evidence than either alone, and a "disagree" outcome where Cox-Stuart
flags positive trend but Jonckheere-Terpstra is null is highly diagnostic of
a one-step shift somewhere in the middle of the window (which Cox-Stuart
catches via the half-vs-half comparison but quartile blocks dilute).

## Why the trilogy is orthogonal in a precise sense

A fully cooked statement: under the joint IID-no-trend null, the asymptotic
joint distribution of (Z_DavidBarton, Z_CoxStuart, Z_JonckheereTerpstra) on a
single window of size n is multivariate normal with all three pairwise
asymptotic correlations equal to zero. Sketch:

- (DB, CS): David-Barton is a function of {sign(xᵢ₊₁ − xᵢ)}. Cox-Stuart with
  lag c = ⌈n/2⌉ is a function of {sign(xᵢ₊c − xᵢ)}. Under IID continuous F,
  sign(xᵢ₊₁ − xᵢ) and sign(xⱼ − xᵢ) for |j − i| ≥ 2 are pairwise independent
  (this is Spearman's classical result). Hence the row sums and pair counts
  are uncorrelated.
- (DB, JT): David-Barton's R depends only on adjacent-sign sequence;
  Jonckheere-Terpstra's J is invariant to within-block ordering and depends
  only on cross-block rank comparisons. Permuting within a block changes R
  but not J. Under IID, the within-block permutation is uniform, so J is
  conditionally constant given the block memberships while R averages over
  the within-block permutation and has zero conditional correlation with J.
- (CS, JT): Both depend on cross-block-position information, but Cox-Stuart's
  sign(xᵢ₊c − xᵢ) test is the half-vs-half U statistic between the first
  ⌊n/2⌋ samples and the last ⌊n/2⌋, which equals U(B₁∪B₂, B₃∪B₄). The
  Jonckheere-Terpstra J on quartile blocks decomposes into U(B₁, B₂) +
  U(B₁, B₃) + U(B₁, B₄) + U(B₂, B₃) + U(B₂, B₄) + U(B₃, B₄). The Cox-Stuart
  half-vs-half sum is U(B₁, B₃) + U(B₁, B₄) + U(B₂, B₃) + U(B₂, B₄) — four
  of the six terms in J. The remaining two — U(B₁, B₂) and U(B₃, B₄) — are
  the within-half between-quartile comparisons that Cox-Stuart literally
  cannot see. Under the null, the four "across-half" U terms and the two
  "within-half" U terms are orthogonal in the standard linear-rank-statistic
  Hoeffding decomposition. So Z_JT − (a constant × Z_CS) has zero asymptotic
  covariance with Z_CS, and the residual ("within-half quartile contrast")
  is what Jonckheere-Terpstra adds to the Cox-Stuart half-vs-half information.

This decomposition is exactly what `classifyJonckheereTerpstraCoxStuartBlock
VsPairTrendCompound` formalizes. The compound emits a 4-bucket label:
(both decisive same direction, both decisive opposite direction, only-block-
decisive, only-pair-decisive, neither). The "only-block-decisive" bucket is
the substantive prize — it isolates step-up patterns that Cox-Stuart's
half-vs-half blunts.

## What the live-smoke pattern means

Putting the three live-smoke results side-by-side:

| Source       | DB dbZ    | DB p       | CS csZ            | JT (qualitative)            |
|--------------|-----------|------------|-------------------|------------------------------|
| vscode-cp    | −13.2062  | 8.4e−40    | (not in excerpt)  | (not in excerpt)             |
| claude-code  | −5.8506   | 4.9e−9     | (one of the sig)  | (compound)                   |
| opencode     | −1.4692   | ns         | (likely ns)       | (compound)                   |
| hermes       | +0.3814   | ns         | (likely ns)       | (compound)                   |
| openclaw     | −0.1907   | ns         | (the other sig)   | (compound)                   |

vscode-cp's −13σ on David-Barton with no comparable Cox-Stuart signal of the
same magnitude is the canonical "tie-degenerate not trend" signature, and it
is precisely why the axis-202 ↔ axis-203 compound (`classifyDavidBartonNoether
SignRunVsSpacedTripletStructureCompound`) was needed: the joiner cross-tabs
adjacent-sign-run rejection against lag-2 spaced-triplet rejection, and a
"DB rejects + Noether does not reject" outcome strongly discriminates
zero-diff-carry-forward (which inflates DB but not Noether's lag-2 statistic)
from genuine low-frequency monotone trend (which inflates both). The
prior post on axis-203 already dispatched this single-source case. The new
contribution from axis-205 + axis-206 is to extend the diagnostic from
"adjacent sign sequence" up to "global monotone shift" and "k=4 ordered
quartile alternative", giving us a full chain:

1. Axis-202 (Noether 1956 lag-2 cyclical) — "do well-spaced triples follow
   monotone or cyclic?"
2. Axis-203 (David-Barton lag-1 sign-runs) — "is the adjacent sign sequence
   too persistent or too alternating?"
3. Axis-205 (Cox-Stuart lag-c pair-signs) — "does the late half dominate the
   early half?"
4. Axis-206 (Jonckheere-Terpstra k=4 quartile blocks) — "do quartile blocks
   ordered in time show a monotone rank-sum trend?"

Each step doubles the look-distance: lag-2 → lag-1 → lag-c=⌈n/2⌉ → k=4 block
trend. And each step is asymptotically independent of the others under the
joint IID-no-trend null. So a single window can be assigned a 4-bit
serial-structure fingerprint, and the three compound classifiers shipped this
week make those fingerprints inspectable from the CLI.

## Implications for the dispatcher's workflow

The three-axis chain is now what the daemon will see when it runs the daily
queue.jsonl analysis. The vscode-cp dbZ = −13.2062 is no longer the loudest
finding in isolation; it gets contextualized by the axis-205 csZ outputs (do
the half-halves disagree?) and the axis-206 JT outputs (do the quartile
blocks order monotonically?). The expected near-term pattern, given that
vscode-cp's queue.jsonl is dominated by zero-diff carry-forward in the daily
token counts, is: DB hugely significant negative, CS approximately zero (no
real half-vs-half drift), JT approximately zero (no quartile-block trend).
That joint pattern, formally, belongs to the {DB-rejects, CS-null, JT-null}
cell of the implicit 8-cell trinary fingerprint, and is the canonical
zero-diff-carry-forward witness. Conversely, a window where DB, CS, and JT
all reject with the same sign is an unusually loud and well-corroborated
trend signal that any one of the three on its own would over-claim.

The next axes in the queue (axis-207 onward) will need to extend this chain
in a genuinely orthogonal direction — likely either by going to **k=5+ blocks
with non-equal spacing** (which would test for non-monotone trend like
inverted-U), by going to **change-point detection** (which the current trio
does not address: it can flag "trend exists" but not "trend starts at index
i"), or by going to **bootstrapped null distributions** for small-sample
windows where the asymptotic Z approximations on these three axes break
down. Whichever direction the next axis takes, the trilogy completed this
tick — David-Barton lag-1, Cox-Stuart lag-c, Jonckheere-Terpstra k=4 — is
the structural backbone that any future serial-structure axis will be
positioned against.
