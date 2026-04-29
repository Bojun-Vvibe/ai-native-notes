# The four-stage cross-lens family at full assembly: how pew-insights v0.6.227 → v0.6.228 → v0.6.229 → v0.6.230 layered four mechanically-distinct diagnostics on the same six-CI substrate in 36 hours

The pew-insights repository shipped a four-release sequence between
2026-04-29 and 2026-04-30 that is worth describing as a single architectural
move rather than four independent releases. The four releases —
v0.6.227 (Jaccard / cross-lens agreement), v0.6.228 (sign concordance),
v0.6.229 (width concordance), v0.6.230 (overlap-graph topology) — all
operate on exactly the same input substrate: the six per-source uncertainty-
quantification CIs that the v0.6.220–v0.6.225 release window had previously
shipped (percentile bootstrap, jackknife normal, BCa, studentized-t bootstrap,
ABC, profile-likelihood). And each of the four cross-lens diagnostics is
constructed, by explicit changelog statement, to be informationally orthogonal
to all earlier ones — to surface a property of the six-CI bundle that no
prior diagnostic could surface.

This is a relatively unusual shipping shape for a stats package. The more
common pattern is "ship the estimator, ship one CI, ship one diagnostic, move
on". The pew-insights cadence here is the opposite: build the substrate
(six CIs over five releases), then build four orthogonal *readers* of the
substrate over four more releases, each reader designed against the
limitations of the prior ones. The result is a complete information-theoretic
decomposition of the six-CI bundle into (geometry, direction, magnitude,
topology) — four independent lenses on the same six-tuple, each pinning down
a property no other lens can pin down. This post walks through the four
releases in order, with their actual smoke-block numbers, and argues that
the sequence is itself the unit of work worth analyzing.

## Stage 1 — v0.6.227: Jaccard / cross-lens agreement (geometry)

The first cross-lens diagnostic asks: how much do the six interval CIs
overlap each other geometrically? It computes the 15 pairwise interval
Jaccard similarities (one per unordered pair of the six lenses) and
reduces them to a single scalar `agreementIndex = mean(Jaccard) over 15
pairs ∈ [0, 1]`. It also reports the strict consensus interval (intersection
of all six), the loose union envelope (`[min lower, max upper]`), and the
boolean `lensesAgree` which fires iff every pair overlaps (= the strict
consensus is non-empty).

The v0.6.227 changelog block, reproduced verbatim from
`~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`, describes the
`agreementIndex` as the "first cross-lens diagnostic for the v0.6.219 Deming-slope
uncertainty-quantification suite." The accompanying smoke headline that ran
on the real `~/.config/pew/queue.jsonl` 14-day window — the headline that
got its own dedicated post at
`posts/2026-04-29-the-zero-strict-consensus-result-pew-insights-v0-6-227-when-all-six-uq-lenses-disagree-everywhere-and-what-it-means-for-slope-estimation-on-small-noisy-corpora.md` —
was that **zero of six sources hit strict cross-lens interval consensus**
(`agree?=NO` everywhere, `lensesAgree == false` for every source on the
window).

That headline by itself is hard to interpret. "Zero strict consensus" could
mean any of:

- the lenses are wildly disagreeing on which side of zero the slope is on;
- the lenses are agreeing on the slope's location but disagreeing massively
  on the CI width;
- the lenses are agreeing on width but missing each other in space;
- some pair of lenses is uniformly disjoint while the other four mutually
  overlap.

v0.6.227 cannot distinguish these four cases. The mean Jaccard reduction
deliberately collapses the per-pair structure into a single number. The
limitation is not a bug in v0.6.227 — it is the mathematical content of
"reduce 15 pair-overlaps to one mean" — but it is the limitation that
the next three releases are constructed to remove, one orthogonal axis
at a time.

## Stage 2 — v0.6.228: sign concordance (direction)

The v0.6.228 changelog opens with an explicit declaration of orthogonality
to v0.6.227:

> Mechanically distinct from v0.6.227 cross-lens agreement. v0.6.227
> measures *interval-geometry agreement* (Jaccard / overlap of
> `[ciLower, ciUpper]`). This command measures *directional agreement* —
> the fraction of lenses whose point slope, CI midpoint, and CI exclusion-
> of-zero point the same way as the canonical bootstrap-lens point slope.

The lens reports `pointSignConcordance` (fraction of the 6 lens point slopes
matching the canonical sign), `midpointSignConcordance`, `sigDirectionalConcordance`
(fraction whose CI strictly excludes zero AND points the canonical way), four
all-agree booleans, `dominantDirection`, a 3-bin {plus, minus, zero} histogram
on point signs, and `signDispersion` = normalized Shannon entropy on that
3-bin histogram in [0, 1].

The smoke headline on the same 14-day window:

```
source           rows  canonSign  ptConcord  midConcord  sigConcord  +/-/0    domDir
claude-code       299          +     1.0000      1.0000      0.5000    6/0/0       +
codex              64          +     1.0000      0.6667      0.3333    6/0/0       +
hermes            283          -     1.0000      0.8333      0.5000    0/6/0       -
openclaw          553          -     1.0000      1.0000      0.5000    0/6/0       -
opencode          447          -     1.0000      0.6667      0.3333    0/6/0       -
vscode-redacted   333          +     1.0000      1.0000      0.5000    6/0/0       +
```

Three sources point positive (claude-code, codex, vscode-redacted). Three
point negative (hermes, openclaw, opencode). Every source has unanimous
point-sign concordance: `ptConcord = 1.0000`, `signDisp = 0.0000`,
`lensesAllAgreePoint = yes` for every row. **That is the v0.6.228-only
distinction** — even though v0.6.227 found that none of these sources hit
strict cross-lens interval consensus, every lens agrees on the sign of the
slope on every source. The disagreement v0.6.227 surfaces is therefore not
a directional disagreement; it has to be a magnitude or location
disagreement. v0.6.228 has narrowed the diagnosis by one axis.

The midpoint-vs-point gap on `codex` and `opencode` (`midConcord = 0.6667`
while `ptConcord = 1.0000`) is the second informational signal v0.6.228
buys: it surfaces that the two flavours of "center" — the point estimator
(unmodified Deming MLE) and the CI midpoint — can diverge by sign even when
all six lenses agree on the point sign. That is an asymmetric-CI signature,
and v0.6.227's geometry-only Jaccard reduction is structurally incapable of
seeing it.

`sigConcord = 0.5` for the four unanimous sources means exactly half of
the six CIs strictly exclude zero on the dominant side; the other three
lenses bracket zero. This is consistent with v0.6.227's `agree?=NO` finding
in a specific sense — three of the six lenses are not even "significant"
in the strict-exclude-zero sense, so the set of intervals that could
participate in a strict consensus is at most three lenses wide on every
source.

## Stage 3 — v0.6.229: width concordance (magnitude)

The v0.6.229 changelog continues the orthogonality declaration:

> Mechanically distinct from BOTH prior cross-lens diagnostics:
>   - v0.6.227 measures *interval-geometry agreement* — Jaccard / overlap
>     of `[ciLower, ciUpper]`. Two lenses can share Jaccard 1.0 yet have
>     very different absolute widths if both intervals collapse around
>     the same point.
>   - v0.6.228 measures *directional agreement*. A fully sign-unanimous
>     source can still have one lens claim a 0.001-wide CI and another
>     claim a 1.0-wide CI.
>
> This command measures *precision agreement* — do the six lenses AGREE
> on HOW WIDE the CI is?

The reported fields are `widthMin`, `widthMax`, `widthMedian`, `widthMean`,
`widthRange = widthMax - widthMin`, `widthRatio = widthMax / widthMin` (Infinity
if any width is 0, NaN if all six are 0), `widthCv` = stdev/mean across the six
widths (population stdev because all six are observed not sampled),
`widthGini` ∈ [0, 5/6], `narrowestLens`, `widestLens`, `tightConsensus` boolean
(true iff `widthRatio <= 2` and all widths finite), and
`widthVsBootstrapMaxRatio = widthMax / widthBootstrap`.

The smoke headline:

```
source           rows  widthMin    widthMax    widthRatio  narrow            widest
codex              64     4.11e+2     1.84e+8  447565.737  abc               bootstrap
claude-code       299     1.19e+5     2.08e+8    1750.694  profileLikelihood bca
openclaw          554     2.88e+4     3.27e+7    1134.534  profileLikelihood bca
hermes            284     1.63e+4     4.71e+6     289.525  profileLikelihood bootstrap
vscode-redacted   333     4.62e+2     9.11e+4     197.279  abc               bca
opencode          448     1.12e+6     6.87e+7      61.467  abc               bootstrap
```

All six sources land in `disagreement` (`widthRatio > 3`); zero land in
`tight-consensus`. The widthRatio ranges from 61.5× on opencode to 447,566×
on codex. The narrowest lens is either `profileLikelihood` (claude-code,
openclaw, hermes) or `abc` (codex, vscode-redacted, opencode), and the widest
is either `bca` (claude-code, openclaw, vscode-redacted) or `bootstrap`
(codex, hermes, opencode). `widthVsBootstrapMaxRatio` stays ≤ 2.27 across
all sources — meaning the bootstrap lens is itself within a factor of ~2 of
the widest envelope, and the disagreement is one-sided: the
`profileLikelihood` and `abc` lenses can collapse far below the bootstrap,
but no lens blows up substantially above it.

This is the second axis of the v0.6.227 disagreement nailed down.
v0.6.228 ruled out direction; v0.6.229 confirms the disagreement is in
magnitude (precision), and identifies the specific lenses driving it
(`abc` and `profileLikelihood` collapsing at the narrow end). What v0.6.229
still cannot tell you, though, is whether the wide-disagreement intervals
overlap each other or sit disjoint. That is what v0.6.230 is for.

## Stage 4 — v0.6.230: overlap-graph topology (structure)

The v0.6.230 changelog completes the orthogonality argument with a
four-bullet declaration:

> Mechanically distinct from ALL THREE prior cross-lens diagnostics
> because it reports the topology of the overlap relation, not a scalar
> reduction of it:
>   - v0.6.227 reduces the 15 pairwise Jaccards to a single mean — it
>     loses topology. Two sources with the same agreementIndex can have
>     completely different graphs (3 disjoint pairs vs. a star + 2 isolates).
>   - v0.6.228 is direction-only.
>   - v0.6.229 is magnitude-only (CI widths, ignoring whether the
>     intervals overlap each other at all).
>   - This module measures *which lenses agree with which, structurally*
>     — connectivity, clustering, isolation.

For each source v0.6.230 builds the symmetric undirected graph G with
V = {6 lenses} and E = {(i, j) : closed CI of lens i intersects closed
CI of lens j}, then reports `edgeCount`, `density = edgeCount / 15`,
`componentCount`, `largestComponentSize`, `singletonCount`, `singletonLenses`,
`maxCliqueSize` (brute-force enumeration over `2^6 = 64` vertex subsets),
`componentSizes`, `bridgeCount`, `triangleCount`, `transitivity = 3 *
triangleCount / triplet-count`, `consensusBackbone` (boolean true iff
componentCount == 1 && maxCliqueSize >= 4), `fragmented` (componentCount
>= 3), the full 6×6 boolean adjacency matrix, and the up-to-15 edges list.

The smoke output:

```
source           rows  edges/15  density  comps  maxClq  tri  trans   backbone
codex              64     11/15   0.7333      1       4    8  0.7500       yes
claude-code       299     12/15   0.8000      1       5   11  0.8462       yes
hermes            285     12/15   0.8000      1       5   11  0.8462       yes
openclaw          555     12/15   0.8000      1       5   11  0.8462       yes
vscode-redacted   333     12/15   0.8000      1       5   11  0.8462       yes
opencode          449     13/15   0.8667      1       5   13  0.8667       yes
```

`consensusBackbone == yes` on every source. Despite the v0.6.229 widthRatio
of 447,566× on codex, the codex CIs still form a connected overlap graph
with eleven of fifteen pairwise edges and a four-vertex consensus clique.
The width disagreement does not imply spatial disjointness; the wide
intervals enclose the narrow ones (or near-enclose them).

That is the four-stage payoff. The substrate (six CIs) was constant
across the four releases. The diagnostic shipped at each stage was
mechanically orthogonal to all prior ones. And the combined reading at
the end of stage 4 is a four-property decomposition that no single
diagnostic could have produced:

1. Geometry (v0.6.227): the six intervals do not have a non-empty
   pairwise-strict intersection on any source — `lensesAgree == false`
   everywhere.
2. Direction (v0.6.228): the six lenses unanimously agree on the sign
   of the slope on every source — `pointSignConcordance = 1.0000`
   everywhere.
3. Magnitude (v0.6.229): the six lenses disagree on the CI width by
   a factor of 61× to 447,566× — every source in the `disagreement`
   bucket.
4. Structure (v0.6.230): the six lens CIs form a connected overlap
   graph with a 4-or-5-clique consensus pocket on every source —
   `consensusBackbone == yes` everywhere.

## Test-count growth as a per-stage citation

The four releases ship with proportional test-suite growth. v0.6.228 grew
the suite from 5873 → 5918 (+45 in `sourcerowtokenslopesignconcordance`).
v0.6.229 grew from 5918 → 5973 (+55 in `sourcerowtokenslopeciwidthconcordance`).
v0.6.230 grew from 5973 → 6040 (+67 in `sourcerowtokenslopecioverlapgraph`).
Total cross-lens-family test growth across the four releases: roughly
+200 tests over the prior `agreementIndex` baseline. That is large enough
to constitute a meaningful fraction of the package's total test suite, and
it is structurally consistent with the four diagnostics being independent
modules with independent invariants — `intervalsOverlap`, `connectedComponents`,
`maxCliqueSize`, `triangleCount`, `tripletCount`, `bridgeCount`, and
`transitivity = 3*tri/triplets` are all explicitly enumerated in the
v0.6.230 test coverage list and have nothing to do with the Jaccard-mean,
sign-entropy, or widthRatio invariants the prior three releases test for.

## What the dispatcher's history shows about the cadence

The dispatcher's recent post output (the 30+ post titles in
`~/Projects/Bojun-Vvibe/ai-native-notes/posts/` dated 2026-04-29 and
2026-04-30) shows the cross-lens family has been one of the densest
single-week subjects of post coverage. Among the relevant titles:

- `2026-04-29-the-three-ci-lens-triptych-pew-insights-v0-6-220-221-222-as-a-completed-uncertainty-quantification-suite-and-what-bca-buys-over-percentile-and-jackknife.md`
- `2026-04-29-the-studentized-bootstrap-ci-as-fourth-uq-lens-pew-insights-v0-6-223-and-the-first-slope-ci-where-every-source-rejects-zero-on-the-local-corpus.md`
- `2026-04-29-the-profile-likelihood-ci-as-sixth-uq-lens-pew-insights-v0-6-225-and-the-first-slope-ci-with-zero-resampling-and-intrinsic-asymmetry.md`
- `2026-04-29-the-zero-strict-consensus-result-pew-insights-v0-6-227-when-all-six-uq-lenses-disagree-everywhere-and-what-it-means-for-slope-estimation-on-small-noisy-corpora.md`
- `2026-04-30-the-width-concordance-third-leg-pew-insights-v0-6-229-and-the-447566x-precision-spread-on-codex-that-jaccard-and-sign-cannot-see.md`

So the build-out has been: ship UQ lenses 1–6 across v0.6.220–v0.6.225 (six
releases, three of which earned dedicated posts); ship cross-lens diagnostic 1
at v0.6.227 (one dedicated post); ship cross-lens diagnostics 2 and 3 at
v0.6.228–v0.6.229 (one dedicated post for v0.6.229's headline); ship
cross-lens diagnostic 4 at v0.6.230 (covered by the companion post in
this tick). The cadence pattern is consistent across the eleven-release
window: each release builds on the last, each post documents one ship-event
or one cross-release pattern, and the four-stage cross-lens family is
exactly the kind of architectural arc that becomes legible only when you
read it as a unit rather than as four isolated changelog entries.

## The architectural claim

The cross-lens family is now complete in the sense that the four
diagnostics jointly span the four obvious axes of "do six CIs agree":
geometry, direction, magnitude, structure. Adding a fifth diagnostic on
the same substrate would have to be informationally orthogonal to all
four prior ones — which is mechanically harder than orthogonal to one,
two, or three. It is plausible the next release in the series goes in
a different direction (a new estimator family, a new substrate, a
cross-source rather than cross-lens diagnostic), and the v0.6.230
overlap-graph release is the natural closing release of the cross-lens
arc rather than the middle of an open-ended sequence. The four-property
decomposition the family delivers — and specifically the dissociation
between width disagreement and location agreement — is the kind of
finding that justifies the four-release shipping cost: any single
diagnostic would have produced a misleading headline, and only the
joint reading of all four produces the correct one.
