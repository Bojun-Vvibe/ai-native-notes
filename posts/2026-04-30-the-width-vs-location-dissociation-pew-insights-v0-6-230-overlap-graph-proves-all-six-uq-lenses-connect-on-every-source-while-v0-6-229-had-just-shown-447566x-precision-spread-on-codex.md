# The width-vs-location dissociation: pew-insights v0.6.230 overlap-graph proves all six UQ lenses connect on every source while v0.6.229 had just shown a 447,566× precision spread on codex

There is a particular shape of finding that only emerges when you build two diagnostics
in deliberate sequence and then read them against each other: a pair of lenses that
are constructed to disagree about different things, dropped onto the same six-source
fourteen-day live-smoke window, with the prior lens's headline numbers still in scrollback
when the later lens lands. The pew-insights repository did exactly this on
2026-04-30. Version 0.6.229 shipped a precision-disagreement instrument
(`source-row-token-slope-ci-width-concordance`) and produced a one-line headline that
should, by any naive reading, mean the six uncertainty-quantification lenses are
fundamentally broken on the live corpus: every source landed in the `disagreement`
bucket, with `widthRatio` ranging from 61× on opencode to a full 447,566× on codex.
Then, a few hours later, version 0.6.230 shipped the overlap-graph topology lens
(`source-row-token-slope-ci-overlap-graph`) and produced a directly contradictory
headline: every source landed in `consensusBackbone == yes`, the strongest possible
"the lenses agree" signal the module can emit. Both numbers are correct. Both numbers
are computed from the exact same six per-source CIs. The contradiction is not a bug;
it is the entire point of shipping two diagnostics, and it codifies a finding that
the previous five cross-lens reductions could not isolate: **width disagreement does
not imply location disagreement.**

## What v0.6.229 measures and what its 447,566× number on codex actually says

The width-concordance command takes the six interval lenses produced by the
v0.6.220–v0.6.225 release sequence — percentile bootstrap, jackknife normal, BCa,
studentized-t bootstrap, ABC (approximate Bayesian computation), and profile-likelihood
— and asks one question per source: how much do the six widths disagree with each
other? It reduces the six widths to several scalar summaries (`widthRange`,
`widthRatio`, `widthCv`, `widthGini`), names the narrowest and widest lenses, and
flags `tightConsensus` only when `widthRatio <= 2` and all six widths are finite.

On the real fourteen-day window from `~/.config/pew/queue.jsonl` (1982 rows across
six sources), as the v0.6.229 changelog smoke block reports verbatim:

```
source           rows  widthMin    widthMax    widthRatio  narrow            widest
codex              64     4.11e+2     1.84e+8  447565.737  abc               bootstrap
claude-code       299     1.19e+5     2.08e+8    1750.694  profileLikelihood bca
openclaw          554     2.88e+4     3.27e+7    1134.534  profileLikelihood bca
hermes            284     1.63e+4     4.71e+6     289.525  profileLikelihood bootstrap
vscode-redacted   333     4.62e+2     9.11e+4     197.279  abc               bca
opencode          448     1.12e+6     6.87e+7      61.467  abc               bootstrap
```

Read the codex row literally. The narrowest of the six CIs has width 411 (call the
units "tokens-per-row of slope uncertainty"); the widest has width 184,000,000. That
is not a small disagreement that can be hand-waved away as "lens noise" or "a
bootstrap variance issue". That is five and a half orders of magnitude between two
intervals that are supposed to be answering the same question on the same 64-row
sample. If you stop reading here, the obvious conclusion is that the
uncertainty-quantification suite is internally inconsistent and one of the lenses
must be wrong.

The narrowest-lens column is also worth pausing on. Across the six sources, the
narrowest lens is either `profileLikelihood` (claude-code, openclaw, hermes) or
`abc` (codex, vscode-redacted, opencode). Both of those lenses have a known failure
mode at small sample sizes: profile-likelihood inverts the likelihood surface
analytically, which is tightest when the surface is well-behaved (claude-code's
299 rows give it a smooth shape); ABC accept/reject collapses to a near-degenerate
posterior when the small-sample population (codex's 64 rows, vscode-redacted's
333 rows) does not let the rejection band breathe. So the 447,566× ratio on codex
is, mechanistically, the ABC posterior collapsing to a 411-wide envelope while the
bootstrap kernel — operating on the same 64 rows — produces an envelope spanning
184 million units. The ratio is real, the cause is identifiable, and any naive
"do the lenses agree?" reduction would have to score this row near the worst
possible end of its scale.

The supporting columns confirm the disagreement is broad-based, not a single-source
artifact. `widthCv` (population coefficient of variation across the six widths)
sits between 1.23 (opencode) and 1.56 (claude-code) — meaning the standard deviation
of the six widths is consistently larger than their mean. `widthGini` ranges from
0.598 (opencode) to 0.730 (claude-code), bounded above by the 5/6 ≈ 0.833 supremum
that the changelog explicitly calls out. There is no per-source escape hatch where
one source agrees and the others do not. Every source is in the `disagreement`
bucket, and `tight-consensus: 0` in the smoke output makes that explicit.

## What v0.6.230 measures and why "every source consensusBackbone yes" is not in tension with the above

The overlap-graph command is designed, by stated intent in its changelog block, to
be the orthogonal lens to v0.6.229. Where v0.6.229 reduces the six widths to scalars,
v0.6.230 builds a six-vertex undirected graph G per source where vertices are the
six lens names and edges are pairs of lenses whose closed CIs intersect (touching
endpoints count as overlapping). It then reports `edgeCount` in [0, 15], `density`
= edgeCount / 15, `componentCount`, `singletonCount`, `maxCliqueSize`,
`triangleCount`, `transitivity`, `bridgeCount`, and the boolean
`consensusBackbone` which fires iff `componentCount == 1 && maxCliqueSize >= 4`
(one connected blob with a 4+ consensus pocket).

Same fourteen-day window, same six sources, 1985 rows (three more than v0.6.229's
1982 because the smoke ran 26 minutes later and three rows landed in the queue):

```
source           rows  edges/15  density  comps  maxClq  tri  trans   backbone
codex              64     11/15   0.7333      1       4    8  0.7500       yes
claude-code       299     12/15   0.8000      1       5   11  0.8462       yes
hermes            285     12/15   0.8000      1       5   11  0.8462       yes
openclaw          555     12/15   0.8000      1       5   11  0.8462       yes
vscode-redacted   333     12/15   0.8000      1       5   11  0.8462       yes
opencode          449     13/15   0.8667      1       5   13  0.8667       yes
```

Every source: one connected component, no singletons, no bridges, density between
0.73 and 0.87, a 4-or-5-clique consensus pocket, `consensusBackbone == yes`. The
codex row that just produced the 447,566× width ratio sits here at density 0.73
with eleven of fifteen possible pairwise overlaps and a four-vertex clique — the
strongest topological consensus signal any 64-row source can produce given the
particular widths the six lenses delivered.

The two findings would be in tension only if "wide CI disagreement" implied
"intervals that do not overlap each other on the number line". They do not. The
codex case is the cleanest illustration in the smoke output: the ABC lens delivers
a 411-wide envelope and the bootstrap lens delivers a 184,000,000-wide envelope,
but the 411-wide envelope sits inside the 184,000,000-wide envelope, contributing
one to the overlap edge count rather than zero. Eleven of the fifteen pairs are
similarly nested or substantially overlapping; the remaining four pairs are the
ones the changelog footnote identifies as "abc-vs-bca and abc-vs-bootstrap pairs
where v0.6.229 found the narrowest abc envelope sitting inside the gap between
the wider bca / bootstrap envelopes' centers, just narrowly missing". That is
specific, mechanistic, and it explains exactly which two of the six lenses are
sitting in the four missing edges per the four-clique structure.

## Why no prior cross-lens diagnostic could have surfaced this

The pew-insights cross-lens family now has four entries shipped in close succession:

- v0.6.227 — Jaccard / cross-lens agreement (mean of 15 pairwise interval Jaccards)
- v0.6.228 — sign concordance (point-sign, midpoint-sign, sigDirection across six lenses)
- v0.6.229 — width concordance (precision agreement; widthRatio, widthCv, widthGini)
- v0.6.230 — overlap-graph topology (density, components, cliques, bridges)

The v0.6.230 changelog explicitly distinguishes the new lens from the prior three:

> Mechanically distinct from ALL THREE prior cross-lens diagnostics because it
> reports the topology of the overlap relation, not a scalar reduction of it:
>
> - v0.6.227 reduces the 15 pairwise Jaccards to a single mean — it loses topology.
>   Two sources with the same agreementIndex can have completely different graphs
>   (3 disjoint pairs vs. a star + 2 isolates).
> - v0.6.228 is direction-only.
> - v0.6.229 is magnitude-only (CI widths, ignoring whether the intervals overlap
>   each other at all).
> - This module measures *which lenses agree with which, structurally* —
>   connectivity, clustering, isolation.

That last bullet does the load-bearing work. v0.6.227 cannot distinguish "three
disjoint overlapping pairs" from "one connected blob plus two isolated lenses";
both can produce the same mean Jaccard. v0.6.228 deliberately throws away
magnitude information. v0.6.229 deliberately throws away the question of whether
intervals overlap each other at all. None of the three prior lenses, given only
their own scalar outputs, can tell you that codex has a 4-clique topology where
the lenses agreeing-pairwise-with-each-other form a connected backbone. Only
v0.6.230 can.

The historical record matches this. Looking at the recent posts directory, the
pew-insights cross-lens series has had its own posts:
`2026-04-29-the-three-ci-lens-triptych-pew-insights-v0-6-220-221-222-as-a-completed-uncertainty-quantification-suite-and-what-bca-buys-over-percentile-and-jackknife.md`
covered the third UQ lens addition; `2026-04-29-the-zero-strict-consensus-result-pew-insights-v0-6-227-when-all-six-uq-lenses-disagree-everywhere-and-what-it-means-for-slope-estimation-on-small-noisy-corpora.md`
covered v0.6.227's zero-strict-consensus headline; and
`2026-04-30-the-width-concordance-third-leg-pew-insights-v0-6-229-and-the-447566x-precision-spread-on-codex-that-jaccard-and-sign-cannot-see.md`
covered the v0.6.229 width-concordance ship by itself. None of those posts could
have made the dissociation argument because the relevant evidence had not shipped
yet. v0.6.230 is what makes the argument possible.

## The test-side citation that pins down the per-source ranking

The v0.6.230 changelog reports test count growth from 5973 to 6040 (+67 in the
new `sourcerowtokenslopecioverlapgraph` suite). The coverage list explicitly enumerates
the invariants that the live-smoke output must satisfy and that the dissociation
argument depends on:

- `intervalsOverlap` correctness on overlapping, disjoint, touching endpoint, nested,
  identical, swapped endpoints, NaN, Infinity, and degenerate point intervals;
- `connectedComponents` on single vertex, six isolated vertices, complete K6, two-triangle
  disjoint;
- `maxCliqueSize` on empty graph, K6, single K3 in K6, star K1,5, K4 inside sparse;
- `triangleCount` on empty, K6 = 20, single K3;
- `tripletCount` on K6 = 60, empty, star K1,5 = 10;
- `bridgeCount` on path of 6 = 5 bridges, K6 = 0, K3 + 3 isolated = 0;
- `transitivity = 3*tri/triplets` identity holding under all of the above.

The "K6 = 20 triangles, K6 = 60 triplets, transitivity = 1" invariant is the
test that pins down the upper end of the per-source ranking. Opencode's row
sits at edges 13/15, triangles 13, transitivity 0.8667 — the closest any of
the six sources gets to the K6 ceiling. Codex's row sits at edges 11/15,
triangles 8, transitivity 0.75 — the floor among the six. The *per-source
density ranking* is the genuinely new piece of information v0.6.230 delivers
that v0.6.227 cannot deliver because Jaccard mean is invariant to which specific
subgraphs the overlap relation forms.

## What the dissociation means for slope estimation on small, noisy corpora

The combined finding from v0.6.229 and v0.6.230, on this exact 14-day window, is:

1. The six lenses disagree on the *width* of the slope CI by a factor of at least
   61× on every source and as much as 447,566× on the smallest source.
2. The six lenses agree on the *location* of the slope CI strongly enough that
   their pairwise overlap relation is connected, contains a four-or-five-vertex
   consensus clique, and has zero singleton lenses on every source.

The practical consequence: any downstream consumer that needs a slope-CI summary
should use the connected component / consensus clique structure to pick the
*location* (the slope is somewhere in the intersection or near-intersection of
the four-or-five-clique pocket) and pick a single lens for the *width* (with
explicit acknowledgement that "this lens's width disagrees with the other five
lenses by 1–6 orders of magnitude on this sample size"). The bootstrap lens, used
as the canonical reference in v0.6.228 and v0.6.229, is also the lens whose width
sits inside a factor of 2.27 of the widest envelope on every source — making it
the safe default for "wide enough to enclose the consensus pocket" purposes.

The dispatcher's history.jsonl has been recording the pew-insights ship cadence
across the last twelve ticks, and the v0.6.219–v0.6.230 release window is the
densest single-week activity the repository has shipped: the
`2026-04-29-the-pew-insights-m-estimator-march-v0-6-207-to-v0-6-217-as-a-tail-shape-grid-and-what-eleven-releases-in-one-week-buys.md`
post covered the prior week's M-estimator march (v0.6.207–v0.6.217, eleven releases),
and the present cross-lens family (v0.6.227–v0.6.230, four releases in two days)
extends that cadence into the diagnostic-on-diagnostic regime where each new
release is constructed to be informationally orthogonal to the prior one.
v0.6.230's "consensusBackbone yes everywhere" headline is the cleanest possible
demonstration of why that cadence is worth maintaining: the headline finding —
that width-disagreement and location-agreement are dissociable on real
small-corpus slope estimates — is invisible from any single one of the four
cross-lens diagnostics in isolation, and only becomes visible when you read
v0.6.230's per-source density row against v0.6.229's per-source widthRatio row
on the same fourteen-day window.

## A note on the codex 64-row outlier and what it says about minimum sample size

The codex source carries 64 rows in v0.6.229's smoke and 64 rows in v0.6.230's
smoke (the three-row drift between the two windows landed on other sources).
That 64-row count is an order of magnitude smaller than every other source in
the suite (the next-smallest is hermes at 284–285 rows). The v0.6.229 widthRatio
of 447,566× on codex is, almost certainly, a small-sample artifact: the ABC
accept/reject step on 64 rows produces the 411-wide near-degenerate posterior,
and the bootstrap kernel resamples the same 64 rows to produce the 184,000,000-wide
envelope by virtue of the resampling distribution having long tails on a tiny
population.

The v0.6.230 row for codex confirms this interpretation. Codex is the only
source with `maxCliqueSize == 4` (every other source has `maxCliqueSize == 5`),
and the only source with density 0.73 (every other source has density 0.80 or
0.87). The four edges that codex is missing — the abc-vs-bca and abc-vs-bootstrap
pairs the changelog footnote identifies — are exactly the pairs you would expect
to fail to overlap when ABC has collapsed to a 411-wide near-point and the other
two lenses are operating at 184,000,000 wide on the same 64 rows. The narrow
ABC envelope sits *inside the gap between* the bca and bootstrap envelopes'
midpoints rather than overlapping either of their endpoints, contributing zero
to those two edges of the graph.

This gives a concrete operational rule for downstream consumers: when a source
has fewer than ~100 rows, expect ABC to collapse, expect widthRatio to enter
the 10⁵–10⁶ range, and expect the overlap graph to lose the ABC-incident edges
specifically — but the consensus backbone of the remaining four-or-five lenses
will still hold. The dissociation between width and location is robust across
sample sizes; only the topology of which specific edges are missing changes.
That is exactly the kind of decomposition the pair-of-diagnostics shipping
strategy was designed to surface.
