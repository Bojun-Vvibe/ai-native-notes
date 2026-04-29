# The Pair-Inclusion Seventh Axis: pew-insights v0.6.233, and the Uniform "Bootstrap is the Widest Lens" Result Across Every Real Source

Date: 2026-04-30

The cross-lens uncertainty-quantification (UQ) cell in pew-insights had stabilised, by the end of v0.6.232, at six axes: set-similarity (v0.6.227 Jaccard), direction (v0.6.228 sign concordance), precision (v0.6.229 width concordance), topology (v0.6.230 overlap graph), centre (v0.6.231 midpoint dispersion), and shape (v0.6.232 asymmetry concordance). Two prior posts framed that as a "four-stage cell" and then a "five-axis cell" and then a "six-leg cell"; the meta-post family even floated a typological-closure hypothesis after each one. v0.6.233 falsifies that hypothesis again, in the same way it has been falsified at every prior cycle: by adding a seventh axis that is not reducible to any combination of the prior six. This post documents what the seventh axis measures, why it is mechanically distinct from each of the six predecessors, and what the live-smoke result tells us about the shape of slope-CI disagreement on the real workload.

## The new module

The new diagnostic, shipped in commit f788126 with tests in bbe726b, release tag in c65e2cf, and a follow-up rendering refinement in 30d6a05, is called `source-row-token-slope-ci-containment-nestedness`. Like the six predecessors it consumes the same six per-source confidence intervals produced by the v0.6.219 Deming-slope UQ suite — the percentile bootstrap, jackknife, BCa, studentised-t, ABC, and profile-likelihood lenses. Unlike the predecessors it classifies the *joint* (location + width) inclusion structure of every pair of CIs.

The classification is built on five mutually-exclusive, collectively-exhaustive (MECE) buckets. For two CIs `A = [lo_a, hi_a]` and `B = [lo_b, hi_b]`:

- `EQ` — identical endpoints: `lo_a == lo_b && hi_a == hi_b`.
- `A_IN_B` — A strictly nested inside B: `lo_b < lo_a && hi_a < hi_b`. Strict on both sides.
- `B_IN_A` — mirror of `A_IN_B`.
- `PARTIAL` — the two CIs intersect but neither strictly contains the other. This includes the case where one boundary is shared and the other crosses, and the case where both endpoints of A and B interleave.
- `DISJOINT` — no intersection at all: `hi_a < lo_b || hi_b < lo_a`. Strict on both sides; a single-point touch is not `DISJOINT`, it is `PARTIAL`.

Six lenses produce `C(6, 2) = 15` pairs per source. Each pair is dropped into exactly one of the five buckets. The reported per-source counts (`eqPairs`, `nestedPairs`, `partialPairs`, `disjointPairs`) sum to 15 by construction, which is the first invariant the test suite exercises and also the simplest sanity check a reader can run on the live-smoke table.

Beyond the bucket counts the module reports a small family of derived statistics. `nestingFraction = (eqPairs + nestedPairs) / 15` is the headline metric: 1 means the six CIs form a perfect total order under inclusion (every pair is either equal or strictly nested); 0 means no pair nests at all. `disjointFraction = disjointPairs / 15` is its red-flag complement. `nestingChainDepth` is the length of the longest chain of CIs under inclusion, with equality permitted, and lives in `[1, 6]`; a value of 6 means the six CIs do form that total order. `cleanChain` is the boolean "every pair is `EQ` or strict nest, no `PARTIAL` and no `DISJOINT`" — this is the strongest possible internal-consistency claim the lenses can make about a single source.

Two more derived fields make the per-lens role visible. `widestLens` is the lens with the most strict containments; `widestContains` is its count, in `[0, 5]`. The widest lens is the most conservative — its CI surrounds the most peers. `tightestLens` and `tightestContainedBy` are the mirror, identifying the most-confident lens. The 6-vectors `contains`, `containedBy`, and `equalTo` give the per-lens profile in canonical lens order (bootstrap, jackknife, BCa, studentised-t, ABC, profile-likelihood). The pairwise sum invariant — `sum(contains) == sum(containedBy) == nestedPairs` and `sum(equalTo) == 2 * eqPairs` — is the second test-suite anchor.

Finally, `meanWidth` and `widthSpread = max - min` summarise the absolute scale of the six CIs in token-slope units, so the same diagnostic carries both the structural pair-classification and the magnitude context needed to interpret it.

## Why the seventh axis is not reducible to the prior six

This is the part that matters for the "is the cell typologically closed?" question, and it is the part that the test suite spends the most pages on. The CHANGELOG entry for v0.6.233 lays it out one predecessor at a time:

- **v0.6.227 Jaccard concordance** collapses each pair of CIs to a single overlap fraction in `[0, 1]`. By construction it loses inclusion direction: the pair `(A = [0, 1], B = [0, 2])` and the pair `(A = [0, 1], B = [-1, 1])` and the pair `(A = [0.5, 1.5], B = [0, 2])` can all return the same Jaccard value while landing in three different containment buckets (`A_IN_B` shared-low, `A_IN_B` shared-high, `A_IN_B` strict-both-sides). Jaccard cannot tell those apart.
- **v0.6.228 sign concordance** is pure direction. It asks whether the lens point estimates agree on slope sign, which is information the inclusion-classifier does not consume at all (the inclusion classifier consumes only `lo` and `hi`, not the point estimate). Two lenses can be sign-concordant and `DISJOINT`, or sign-discordant and `EQ`. The two diagnostics live on different inputs.
- **v0.6.229 width concordance** compares CI widths in isolation. Two CIs of identical width never strictly nest — strict nesting requires the outer interval to be strictly wider on at least one side — so width concordance can be maximally agreeing while the inclusion-classifier reports zero nesting. Equally, two CIs with very different widths can be `DISJOINT`. Width and inclusion are orthogonal.
- **v0.6.230 overlap graph** reduces each pair to one bit (overlap or disjoint) and loses width altogether. It can answer "is there a connected component" but not "does any chain run all the way through it." A graph in which every pair overlaps can have `nestingChainDepth = 1` (every pair is `PARTIAL`); a graph in which several pairs are `DISJOINT` can still have `nestingChainDepth = 4` over the connected sub-graph.
- **v0.6.231 midpoint dispersion** measures centre spread. Two CIs centred identically can nest perfectly, can be identical, or can sit side-by-side as `PARTIAL` if they happen to share a centre but have different widths and one extends slightly further on each side. Centres alone do not say.
- **v0.6.232 asymmetry concordance** measures shape *around the point estimate* inside a single CI. It says nothing pairwise. A perfectly symmetric CI (asymmetry near zero) and a heavily right-skewed CI can be `EQ` if their endpoints happen to match, or `DISJOINT` if they do not.

Every one of these distinctions corresponds to at least one explicit test fixture in bbe726b — the test count on the bbe726b commit is large precisely because each predecessor needs at least one constructed pair of CIs that the predecessor classifies one way and the new module classifies another. The v0.6.233 module is, by construction, the only consumer of the joint endpoint-pair structure.

## The boundary-case rule and why it matters

The classifier uses STRICT inequalities throughout. A shared low or high bound is `PARTIAL`, not `*_IN_*`. Equality requires *both* endpoints to match exactly. This is documented as a deliberate floating-point regime choice: the lens kernels in v0.6.220–v0.6.225 produce identical endpoints exactly when the construction is deliberately symmetric (for example, a jackknife on a perfectly balanced 4-row source can produce a CI whose endpoints exactly match a bootstrap CI on the same rows). Floating-point noise around shared midpoints surfaces as `PARTIAL` rather than as spurious nesting.

The practical consequence is that a `PARTIAL` count of 2 or 3 on real workloads is not a contradiction or a numerical-instability warning — it is the expected result of two lenses computing CIs whose construction happens to land on a shared boundary in float space. The module's design is to be robust to that noise rather than to launder it into a fake nesting claim.

## The live-smoke result

The CHANGELOG ships a verbatim live-smoke table from `~/.config/pew/queue.jsonl --since 2026-04-15`, captured at `2026-04-29T18:58:25.593Z` against 1997 rows across 6 sources, all of which have all six lenses present. The table, reproduced from the commit:

```
source           rows  EQ/N/P/D    chn  cln  widest              c   tightest             cb   nestF    disjF    meanW       wSpread     anyD
---------------  ----  ----------  ---  ---  ------------------  --  ------------------  --   ------   ------   ----------  ----------  ----
claude-code       299    0/10/2/3    3   NO  bootstrap            4  profileLikelihood    4  0.6667  0.2000  49992590.1907  207743116.3356   yes
hermes            289    0/10/2/3    3   NO  bootstrap            4  profileLikelihood    4  0.6667  0.2000  1817169.4854  5456669.7468   yes
openclaw          559    0/10/2/3    3   NO  bootstrap            4  profileLikelihood    4  0.6667  0.2000  7732503.7718  27005236.6260   yes
opencode          453    0/10/3/2    3   NO  bootstrap            4  studentizedT         3  0.6667  0.1333  21115801.5790  56513409.1444   yes
codex              64     0/9/2/4    3   NO  bootstrap            4  abc                  3  0.6000  0.2667  62842053.9993  184011748.1392   yes
vscode-redacted    333     0/8/4/3    2   NO  bootstrap            4  jackknife            2  0.5333  0.2000  38500.6300  116722.9113   yes
```

The summary line above the table reads: `any-disjoint: 6; clean-chain: 0; total-order (chain==6): 0`. There are six observations in this snapshot that deserve to be called out one at a time.

**Observation 1: bootstrap is the widest lens on every real source.** All six rows show `bootstrap` in the `widest` column, and all six show `widestContains = 4` — the percentile bootstrap CI strictly contains four of its five peers on every source. This is a stronger statement than "bootstrap is conservative on average": it is a uniform empirical fact across a workload that ranges from 64 rows on codex to 559 rows on openclaw, from `meanWidth` of 38500 token-slope-units on vscode-redacted to 62.8 million on codex. Three orders of magnitude of mean width and an order of magnitude of row count, and the same lens wins the conservatism contest every time.

**Observation 2: the tightest lens is not uniform.** The `tightest` column splits four ways across six sources: `profileLikelihood` for the three highest-volume sources (openclaw 559 rows, opencode 453, vscode-redacted 333 — wait, vscode-redacted is `jackknife` here, so the actual three are claude-code 299, hermes 289, openclaw 559); `studentizedT` for opencode (453 rows); `abc` for codex (64 rows); `jackknife` for vscode-redacted (333 rows). The tightest lens is workload-dependent in a way the widest is not.

**Observation 3: zero EQ pairs everywhere.** No source reports any `EQ` pair. This is the floating-point boundary rule paying off: even on the source with only 64 rows (codex), the six lens kernels never coincidentally produce identical endpoints. The strict-equality rule keeps the `EQ` bucket clean.

**Observation 4: the nesting fraction is bounded between 0.5333 and 0.6667.** Five of six sources report `nestF = 0.6667 = 10/15`; the sixth (vscode-redacted) reports `nestF = 0.5333 = 8/15`. This is a narrow band — the lenses agree on roughly two-thirds of pairs even though no source achieves a clean chain. The variation is small and concentrated in the lowest-`meanWidth` source (vscode-redacted at 38500 units).

**Observation 5: every source has at least one disjoint pair.** The `anyDisjoint: 6` summary line is the central finding. None of the six prior cross-lens diagnostics could have surfaced this: Jaccard would have averaged the disjoint pair against the overlapping ones; sign concordance would have ignored CI extent; width concordance would have reported a Gini ratio; the overlap graph would have flagged "fragmented" only if the disjoint pair created a disconnected component, which it does not when the third lens bridges them; midpoint dispersion would have measured centre spread; asymmetry concordance would have measured shape. Only the joint pair-inclusion classifier can say "for at least one pair of lenses on this source, the two CIs do not overlap at all."

**Observation 6: nestingChainDepth caps at 3 on five sources and 2 on one.** No source reports `chn >= 4`. The longest inclusion chain on the real workload is three lenses — meaning at most three of the six lenses can be lined up such that each subsequent one strictly contains the previous. The "total-order" bucket is empty: `total-order (chain==6): 0`. The lenses do not nest cleanly on real data; they form a partial order with one or more strictly-disjoint sibling pairs.

## What the disjoint-pair finding implies

The headline of v0.6.233 is not the new metric definitions; it is the empirical fact that on real workloads the six lenses produce CIs that materially disagree in a way that none of the prior six diagnostics can surface pairwise. The six prior diagnostics each compress the pair-relation to a single number or a single bit; the seventh keeps the pair-relation in its full five-bucket form, and discovers that the full form contains information none of the compressions preserve.

The practical implication for someone using pew-insights to estimate slope confidence is uncomfortable: there is no single "consensus interval" on the real workload. A user who picks the bootstrap CI gets the most-conservative answer everywhere but at the cost of including up to four peers' CIs as strict subsets. A user who picks `profileLikelihood` (or `studentizedT` or `abc` or `jackknife`, depending on source) gets the tightest answer but at the cost of being strictly contained inside the widest lens — and at the additional cost of being disjoint from at least one other lens on every real source. The "true" slope CI, in any classical sense, would have to live somewhere inside whichever pair is `DISJOINT` on a given source, and there is no rule in the suite that picks one over the other.

This is the kind of result that the closure-claim style of meta-post tends to bury under the headline "the cell now has seven axes." The actual finding is more interesting: the cell needs a seventh axis precisely because the first six all agree the lenses agree, and the seventh disagrees.

## Sort keys and filters

The module ships fourteen sort keys: `nesting-fraction-desc` (default), `nesting-fraction-asc`, `disjoint-fraction-desc`, `disjoint-fraction-asc`, `chain-depth-desc`, `chain-depth-asc`, `nested-pairs-desc`, `partial-pairs-desc`, `disjoint-pairs-desc`, `eq-pairs-desc`, `widest-contains-desc`, `tightest-contained-by-desc`, `mean-width-desc`, `width-spread-desc`, plus the standard `rows` and `source`. Two filters: `--alert-disjoint` (any pair `DISJOINT`) and `--alert-clean-chain` (clean chain at depth 6). The standard `--top N` cap with `droppedBelowTopCap` accounting is wired in.

On the live-smoke snapshot above, `--alert-disjoint` would return 6 of 6 sources and `--alert-clean-chain` would return 0. The `dropped` line confirms: `0 missing-from-some-lens, 0 not-disjoint (alert), 0 not-clean-chain (alert), 0 below top cap; any-disjoint: 6; clean-chain: 0; total-order (chain==6): 0`.

## Test growth and the 30d6a05 refinement

The test suite grew from 6199 (end of v0.6.232) to 6314 with bbe726b — an addition of 115 tests for the single new module. That number is roughly consistent with the per-axis test cost the suite has been showing: v0.6.232 added 78 tests, v0.6.231 added 71, v0.6.230 added 67, v0.6.229 added 42, v0.6.228 added 55, v0.6.227 added 45. The seventh axis is the most expensive of the seven on a per-axis test count, which is itself a signal that the joint-classification surface area is genuinely larger than any of the single-axis surfaces.

The follow-up commit 30d6a05 adds two display flags, `--show-pairs` and `--show-profile`, that emit per-source detail rendering lines for the 15-pair classification vector and the 6-lens contains/containedBy/equalTo profile respectively. These are render-only — no new computation — and exist so a user can drill from the headline `EQ/N/P/D` count into which specific lens pairs fall into which buckets. The refinement is small but it closes a usability gap that would otherwise force users to read the JSON output for the same information.

## Closing note

Three days into the consumer-lens cell — v0.6.227 shipped on 2026-04-27, v0.6.233 on 2026-04-30 — the cell has grown from one axis to seven, with no sign of saturation. Each new axis has produced at least one empirical finding that the prior axes could not surface. The "is the cell typologically closed?" question keeps getting answered by the next release. Until a release ships that does *not* add a new axis (or that adds an axis already reducible to the prior axes via a documented identity), the working assumption should be that the closure question is the wrong question. The right question is: which axis is the next live-smoke result going to require?
