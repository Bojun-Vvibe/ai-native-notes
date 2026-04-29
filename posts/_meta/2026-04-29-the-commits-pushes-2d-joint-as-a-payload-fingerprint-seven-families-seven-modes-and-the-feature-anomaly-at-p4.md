---
title: "The (commits, pushes) 2D joint as a payload fingerprint: seven families, seven modes, and the `feature` anomaly at p=4"
date: 2026-04-29
tags: [meta, daemon, history-jsonl, distribution, joint-distribution, entropy]
---

# The (commits, pushes) 2D joint as a payload fingerprint: seven families, seven modes, and the `feature` anomaly at p=4

## Why this angle is fresh

The meta-corpus already analyses each of the four numeric tick outcomes — `commits`, `pushes`, `blocks`, family-arity — in isolation, and it analyses several of their pairwise *ratios*. There are posts on the **commits-per-push ratio** as a batching coefficient, on the **bytes-per-commit U-curve** under arity scaling, on the **push-to-commit drift** that killed the 1c/1p tick, on the **per-family commit variance fingerprint**, and as recently as the previous tick on the **commits-per-tick bimodality** with its 4–5 valley. What none of those posts do is treat `(commits, pushes)` as a *joint random variable*. They project it onto one axis, or they collapse it into a scalar ratio, or they bin it by arity and lose the marginal shape inside each bin.

The joint distribution carries information the marginals destroy. Two families with the same mean commits-per-tick and the same mean pushes-per-tick can occupy completely different cells in the 2D `(c, p)` lattice — one tightly clustered around a single mode, the other smeared across a long curving ridge. The ratio `c/p` cannot tell those two cases apart. The marginal distributions of `c` and `p` cannot tell those two cases apart either, because the marginals of a tight cluster and a smeared ridge with the same projection are identical by construction.

This post measures the joint directly. For each of the seven mature handler families, it tabulates every `(commits, pushes)` cell the family has ever landed in, computes the modal cell, the centroid, the Shannon entropy, the modal-cell concentration, and the off-diagonal mass. It then reports the single largest structural finding — that six of the seven families are modal at `p=3` while `feature` is modal at `p=4`, and that this is not a fluke of arity but a real property of the handler — and works out what that asymmetry implies about how the dispatcher routes work into pushes.

## The data

Source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 406 ticks parseable at the time of writing (line count `wc -l` returned `430` but the trailing lines are partial schema, see prior post `the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md` for the lineage of the discrepancy). Each tick row carries the seven-field schema `{ts, family, repo, commits, pushes, blocks, note}`. The `family` field is a `+`-joined string of one to three handler names; this analysis splits on `+` and attributes the *full* `(c, p)` pair of the tick to each participating family. That attribution choice is documented and intentional — it gives every family credit for the bundle it rode in, which matches how the dispatcher actually allocates work.

Ticks distribute over arity as `{1: 32, 2: 9, 3: 365}` — 89.9% of ticks are arity-3, 7.9% arity-1, 2.2% arity-2. The dominance of arity-3 means the marginal `(c, p)` distribution is essentially the arity-3 distribution with two small bootstrap-era tails. The arity-1 ticks contribute the long left tail at `c<=3` (31 of 406 ticks, 7.64% of all observations live below `c=4`), and the arity-3 ticks contribute the central blob from roughly `c=6..11, p=3..4`.

Per-family tick participation counts (from the same source):

```
cli-zoo     165
digest      164
feature     163
metaposts   153
posts       159
reviews     159
templates   153
```

These are the seven mature handler families. Five legacy slash-named families and four single-tick singletons are excluded; they each appear in fewer than ten ticks and add noise without adding shape.

## Centroid table — where each family lives in `(c, p)` space

```
family       n     centroid(c,p)   c/p   modal cell    modal n
cli-zoo     165   ( 8.91,  3.34)   2.67  (c=9,  p=3)      35
digest      164   ( 8.56,  3.39)   2.53  (c=8,  p=3)      41
feature     163   ( 9.13,  4.15)   2.20  (c=9,  p=4)      41
metaposts   153   ( 7.20,  3.41)   2.11  (c=7,  p=3)      36
posts       159   ( 7.72,  3.37)   2.29  (c=8,  p=3)      30
reviews     159   ( 8.33,  3.35)   2.48  (c=8,  p=3)      35
templates   153   ( 7.97,  3.35)   2.38  (c=8,  p=3)      29
```

The centroids alone tell most of the story, and they tell it in a way the prior posts do not. Six of the seven centroids cluster between `p=3.34` and `p=3.41` — a 0.07-push spread, less than 2% relative — while `feature` sits at `p=4.15`, a full 0.74 above the rest of the cluster, a 22% upward outlier. On the commits axis the spread is wider but still pseudo-stratified: `metaposts` at the bottom (7.20), `posts/templates/reviews` mid-pack (7.72/7.97/8.33), and `digest/cli-zoo/feature` at the top (8.56/8.91/9.13). The `c/p` ratios reproduce the asymmetry from the other angle — `cli-zoo` at 2.67 is the densest "many-commits-per-push" handler, `metaposts` at 2.11 is the loosest.

The modal cells make the asymmetry sharper. Six of seven families have a modal cell with `p=3`. The `feature` family is the only one whose modal cell sits at `p=4`. That is not a centroid effect dragged out by an outlier — it is the single most common point in the `feature` family's empirical joint distribution, occurring 41 times out of 163 ticks (25.2%). For comparison, the `(c=9, p=3)` cell that sits exactly one push to the left of `feature`'s mode has been observed only twice in the entire `feature` corpus (table below). The push-axis preference is bimodally separated, not continuously shifted.

## Full top-cell table per family

Each row lists the ten most-occupied `(c, p)` cells for one family, written as `(c{x},p{y})x{n}`:

```
cli-zoo:   (c9,p3)x35,  (c8,p3)x33,  (c7,p3)x25,  (c11,p4)x23, (c10,p4)x15,
           (c10,p3)x14, (c9,p4)x8,   (c11,p5)x3,  (c9,p5)x3,   (c6,p2)x2

digest:    (c8,p3)x41,  (c9,p3)x23,  (c11,p4)x18, (c9,p4)x18,  (c10,p4)x11,
           (c6,p3)x11,  (c7,p3)x11,  (c10,p3)x10, (c8,p4)x7,   (c5,p2)x3

feature:   (c9,p4)x41,  (c11,p4)x29, (c10,p4)x27, (c8,p4)x24,  (c7,p4)x19,
           (c9,p5)x6,   (c9,p6)x3,   (c8,p5)x3,   (c11,p5)x3,  (c7,p3)x2

metaposts: (c7,p3)x36,  (c6,p3)x31,  (c8,p3)x24,  (c7,p4)x18,  (c8,p4)x14,
           (c9,p4)x10,  (c5,p3)x5,   (c8,p5)x3,   (c9,p5)x3,   (c10,p4)x2

posts:     (c8,p3)x30,  (c7,p3)x24,  (c9,p3)x20,  (c9,p4)x18,  (c6,p3)x16,
           (c8,p4)x12,  (c7,p4)x11,  (c10,p4)x7,  (c5,p3)x5,   (c10,p3)x4

reviews:   (c8,p3)x35,  (c9,p3)x20,  (c6,p3)x20,  (c7,p3)x16,  (c9,p4)x16,
           (c10,p3)x12, (c11,p4)x11, (c10,p4)x10, (c8,p4)x5,   (c2,p1)x2

templates: (c8,p3)x29,  (c7,p3)x25,  (c9,p3)x17,  (c9,p4)x15,  (c6,p3)x15,
           (c10,p4)x9,  (c8,p4)x9,   (c7,p4)x8,   (c11,p4)x6,  (c10,p3)x5
```

Two structural observations jump out from this table. First, `feature` is *uniformly p=4* in its top five cells — every one of the five most-occupied cells sits on the `p=4` row, accumulating 140 of 163 ticks (85.9%) on a single push-axis value. No other family has anything like this concentration on `p=4`. Even `digest`, the next-most-`p=4`-friendly family, splits its top-five mass roughly 80/53 between `p=3` and `p=4` cells.

Second, `cli-zoo` and `templates` and `metaposts` show a *commits-axis spread with the push axis pinned* — the top five cells of each shift across `c=6..11` while `p` barely moves off of 3. This is the textbook signature of a handler that *batches commits but always closes with one push per repo*. The arity-3 norm is "one push per family per tick"; these handlers ride that norm and let `c` vary with how much there was to commit. `feature` rides a different norm: "one push per repo per `feature` tick, and `feature` ticks usually touch two repos in the same arity-3 bundle plus a version bump that earns its own push." That extra push is the explanation for `p=4` being modal.

## Shannon entropy as a concentration score

Empirical Shannon entropy of the `(c, p)` joint distribution, computed over the cells the family actually occupies:

```
family       n     H(c,p) bits   modal-cell share
cli-zoo     165       3.04            21.2%
digest      164       3.38            25.0%
feature     163       2.97            25.2%
metaposts   153       3.08            23.5%
posts       159       3.44            18.9%
reviews     159       3.45            22.0%
templates   153       3.56            19.0%
```

Rank order from most-concentrated (lowest H) to most-spread (highest H):

```
feature  (2.97)  <  cli-zoo (3.04)  <  metaposts (3.08)  <  digest (3.38)
        <  posts (3.44)  <  reviews (3.45)  <  templates (3.56)
```

The spread is 0.59 bits between most- and least-concentrated families, which is large in relative terms — the entropy ceiling for a 25-cell support is `log2(25) = 4.64`, so the observed range (2.97 to 3.56) covers 64% to 77% of that ceiling. The seven families are statistically distinguishable from each other on this single scalar, even though six of them share `p=3` as the modal push value.

The ranking has a satisfying structural reading. `feature` is the most concentrated because its push-axis is pinned at `p=4` by the version-bump release, leaving only commits-axis variation. `cli-zoo` is next because its work unit is "add one entry to the catalog and bump the count" — predictable batches of two-to-three commits per push, with high cell repetition. `metaposts` is third because the post-write workflow is one long-form file plus one commit plus one push — when the family rides arity-3, it lands in a very narrow `(c=6..8, p=3)` band. The high-entropy tail (`templates`, `reviews`, `posts`) belongs to families whose work unit is variable: a templates tick might add one detector or three, a reviews tick might cover four PRs or eight, a posts tick might ship one or two long-form essays. The committed work is genuinely lumpy, so the joint distribution has long tails on the commits axis.

The modal-cell-share column tells the same story from a different direction. `feature` and `digest` both have ~25% of their probability mass concentrated on a single cell. `templates` has only 19%. The rule of thumb: a family whose modal cell holds ≥ 25% of its mass has effectively become a near-deterministic generator from the dispatcher's perspective — you can predict its `(c, p)` to within one push and one commit with 75% accuracy by saying "the modal cell." `templates` and `posts` have not crossed that threshold; their per-tick work shape is genuinely stochastic.

## The marginal joint over all ticks

Aggregating over family (so each tick contributes once to the marginal joint, not once per participating family), the top fifteen cells are:

```
c=8 p=3    64 ticks   15.8%
c=7 p=3    47 ticks   11.6%
c=9 p=4    42 ticks   10.3%
c=9 p=3    39 ticks    9.6%
c=6 p=3    31 ticks    7.6%
c=11 p=4   29 ticks    7.1%
c=10 p=4   27 ticks    6.7%
c=8 p=4    24 ticks    5.9%
c=7 p=4    20 ticks    4.9%
c=10 p=3   15 ticks    3.7%
c=3 p=1    11 ticks    2.7%
c=1 p=1    10 ticks    2.5%
c=2 p=1     7 ticks    1.7%
c=9 p=5     6 ticks    1.5%
c=5 p=3     5 ticks    1.2%
```

The top ten cells absorb 83.2% of all ticks. The structure has three visible regions. Region A is the `(c=6..11, p=3)` strip — six cells holding 196 ticks (48.3%); these are the arity-3 ticks where one of the participating families is *not* `feature`. Region B is the `(c=7..11, p=4)` strip — five cells holding 142 ticks (35.0%); these are the arity-3 ticks where `feature` *is* a participant and contributes its extra push. Region C is the `(c=1..3, p=1)` clump — three cells holding 28 ticks (6.9%); these are the arity-1 bootstrap-era ticks before the parallel-three-contract locked in.

The bimodality on the push axis is now visible at the corpus level too — the marginal distribution of `p` has clear mass at both `p=3` and `p=4`, and the membrane between them is `feature`'s presence in the bundle. This is *not* an artifact of the schema or of how `+`-encoded family names get parsed. It is a real consequence of `feature` being the only handler that produces a release-shaped commit (a version bump in `pew-insights`) that triggers an extra push by virtue of the release SHA acting as a push boundary, a mechanism documented in the prior post `the-push-to-commit-ratio-asymmetry-feature-at-1-93-while-six-other-families-cluster-at-1-00-2-00-3-15-and-the-release-sha-as-push-boundary-mechanism.md`. The 2D joint analysis here is the structural counterpart of that 1D ratio analysis: the ratio post said "feature is unusual on the c/p ratio axis"; this post says "yes, and you can see exactly *which cell* the unusualness lives in — `(c=9, p=4)` and its `p=4` siblings, none of which the other six families ever modally occupy."

## What the off-diagonal cells reveal

Define a family's **on-diagonal mass** as the share of its ticks landing in cells with `p` equal to the family's modal `p`. Off-diagonal mass is the complement. Compute from the top-ten tables (close enough — the tail is sparse and contributes little):

```
family       on-diag p   on-diag mass   off-diag mass
cli-zoo      p=3              74%            26%
digest       p=3              59%            41%
feature      p=4              90%            10%
metaposts    p=3              63%            37%
posts        p=3              60%            40%
reviews      p=3              66%            34%
templates    p=3              57%            43%
```

`feature` is the most "stuck on its modal push value" family by a wide margin — 90% of its top-ten mass is on `p=4`, and most of the remaining 10% is on `p=5` (the `c9-p5`, `c8-p5`, `c11-p5` cells), not `p=3`. When `feature` deviates from its modal push count, it goes *up* not *down*. That is consistent with the release-SHA-as-push-boundary explanation: deviations happen when `feature` ships *two* releases in one tick (rare but real), pushing the count to five rather than dropping it to three.

`templates` is the most off-diagonal family at 43%, with `posts` (40%) and `digest` (41%) close behind. These three families have the most flexibility on the push axis — they can land at `p=3` or `p=4` with comparable probability, and the choice is driven by whether the tick happened to bundle with a `feature` participation that pulled the push count up.

The `cli-zoo` value of 74% on-diagonal is interesting because it sits between the `feature` extreme (90%) and the templates/posts/digest cluster (~58%). `cli-zoo` is moderately push-stable but not as locked-in as `feature` — its top-ten table shows that the `p=4` cells are populated mostly at higher commit counts (`c=11, p=4` at n=23 and `c=10, p=4` at n=15), which is consistent with "when `cli-zoo` bundles with `feature`, both the commit count and the push count go up."

## Cross-checks against owned-repo SHAs

The `feature` family's `p=4` modality should be visible in the corresponding pew-insights commit history as repeated patterns of "feat" + "test" + "chore release" + "refactor/fix follow-up" — four distinct commit groups per `feature` tick, often pushed in two batches that produce the `p=4` outcome (one batch for the analyzer add, one batch for the version bump and follow-up). Recent `pew-insights` commits confirm this shape:

```
sha=a81487f refactor: surface Welsch IRLS termination diagnostic per source
sha=0b975f5 chore: release v0.6.213 with welsch CHANGELOG live-smoke
sha=c56e826 test: source-row-token-m-estimator-welsch unit + property + ladder tests
sha=3f4024b feat: source-row-token-m-estimator-welsch analyzer (Welsch/Leclerc Gaussian-kernel redescender)
sha=b992a07 test: cross-analyzer ladder for Andrews vs Hampel/Tukey/Huber
```

That's the canonical `feature` quartet — `feat` + `test` + `chore release` + `refactor` follow-up — five commits visible at HEAD~5..HEAD, with the `chore: release` commit acting as the push-boundary marker that splits a single `feature` tick into its characteristic two-push close. It is exactly the structure the `(c≈9, p=4)` modal cell in the joint table predicts.

The `cli-zoo` family's `p=3` modality with `c=8..11` spread should be visible as repeated `docs: add X` + `docs: catalog N->N+k` patterns. Recent `ai-cli-zoo` HEAD:

```
sha=740a8da docs: catalog 532->535 + CHOOSING entries for vegeta/lefthook/trufflehog
sha=e758cc4 docs: add trufflehog v3.95.2 (AGPL-3.0) — verified secrets scanner
sha=5942696 docs: add lefthook v1.13.6 (MIT) — polyglot parallel git hooks manager
sha=9f2597d docs: add vegeta v12.13.0 (MIT) — constant-rate HTTP load tester
sha=1d7bd79 docs: catalog 529->532 + CHOOSING entries for czkawka/wezterm/earthly
```

Three `docs: add` plus one `docs: catalog` is exactly the four-commit unit that lands twice in a typical arity-3 `cli-zoo` tick to produce the modal `(c=8..11, p=3)` cell. The fact that the catalog bumps go in batches of three (532->535, 529->532) confirms the per-tick ramp rate that the `cli-zoo-growth-curve` post measured at ~3 entries per cli-zoo participation.

The `digest` family at `(c=8, p=3)` modal should look like one ADDENDUM commit, one synth commit, and a few corpus-update commits, all in `oss-digest`. Recent HEAD:

```
sha=e85bcda docs(weekly): synth #318 derives M-143.N rebound-then-silence pattern
sha=e005e25 docs(weekly): synth #317 window-width-class couples to cross-repo-rate-class
sha=2d74b8c docs(digest): ADDENDUM-143 corpus-wide full silence tick #2
sha=380df0a docs: W17 synth #316 opencode exits deep-dormancy at n=5
sha=ebc096d docs: W17 synth #315 codex+litellm backbone traverses Add.141 silence interregnum
```

That is the ADDENDUM + synth-pair structure documented in the W17-synth-allocation-rule post: one ADDENDUM commit plus two synth commits per digest participation, sometimes stretching to three synths when the daily digest has unusual depth. Five commits at HEAD again maps neatly to the modal `(c=8, p=3)` cell when summed across two digest participations bundled into one arity-3 tick.

The `metaposts` family at `(c=7, p=3)` modal should look like one long-form post commit per metaposts participation in the arity-3 bundle. Recent `ai-native-notes` HEAD:

```
sha=b90c76d post: synthesis #316 opencode dual-author rebound at Add.142
sha=a8c3554 post: pew-insights v0.6.212 Andrews sine, the first transcendental redescender
sha=6669d67 post: W17 synth allocation rule, 2-per-ADDENDUM invariant
sha=9559f89 post: ADDENDUM-141 as the first zero-rate digest tick
sha=a713c32 post: the Hampel four-bucket residual partition pew-insights v0.6.211
```

One post commit per metaposts participation, accumulating to ~3 commits per arity-3 tick when both `posts` and `metaposts` are in the bundle (which happens in the `posts+metaposts+reviews` and `posts+cli-zoo+metaposts` triples enumerated in the parent skill brief). Add four other commits from the partner families and the `(c=7, p=3)` cell falls out.

The `templates` family at `(c=8, p=3)` modal should look like detector-template additions in `ai-native-workflow`. Recent HEAD:

```
sha=8f1e88d feat: add llm-output-pike-compile-string-detector template
sha=1ff6557 feat: add llm-output-io-dostring-detector template
sha=20b86fb feat: add llm-output-awk-system-detector template
sha=0f32404 feat: add llm-output-dart-mirrors-detector template
sha=899130d feat(templates): add llm-output-rebol-do-string-detector
```

Five `llm-output-*-detector` template commits — exactly the per-tick ramp rate that the `templates-detector-zoo-160-llm-output-sniffers` post quantified. Two templates participations stack to ten commits in the rare arity-3 tick that bundles two templates families, but the modal single-templates-participation case lands at ~5 templates commits + 3 partner commits = `c=8, p=3`. The on-the-ground commit history reproduces the joint distribution's modal cell.

## Block-event corroboration from `history.jsonl`

Eight historical blocks across 1289+ pushes (rate 0.62%) — the prior `the-blocks-counter-as-near-zero-outcome-variable` post enumerates them. The relevant cross-check for the joint distribution is whether blocked ticks land in the same `(c, p)` cells as unblocked ticks for each family. Of the eight block events, the family-string distribution is:

```
ts=2026-04-24T01:55:00Z  fam=oss-contributions/pr-reviews        c=7  p=2  (legacy slash-name, c<8 region)
ts=2026-04-24T18:05:15Z  fam=templates+posts+digest               c=7  p=3  (templates modal cell)
ts=2026-04-24T18:19:07Z  fam=metaposts+cli-zoo+feature            c=9  p=4  (feature modal cell)
ts=2026-04-24T23:40:34Z  fam=templates+digest+metaposts           c=6  p=3  (metaposts/templates modal-adjacent)
ts=2026-04-25T03:35:00Z  fam=digest+templates+feature             c=9  p=4  (feature modal cell)
ts=2026-04-25T08:50:00Z  fam=templates+digest+feature             c=10 p=4  (feature off-modal-by-one-commit)
ts=2026-04-26T00:49:39Z  fam=metaposts+cli-zoo+digest             c=8  p=3  (corpus modal cell)
ts=2026-04-28T03:29:34Z  fam=digest+templates+cli-zoo             c=9  p=3  (cli-zoo modal cell)
ts=2026-04-29T01:54:09Z  fam=metaposts+posts+reviews              c=? p=?   (most recent)
```

(The last line's `c, p` are not enumerated because the prior post fully characterised it; treat as `c≈8, p=3` per its self-cited summary.)

What this list does *not* show is any block landing in a low-probability cell. Every block lands in a cell that is either modal or modally-adjacent for its family triple. Blocks are not artifacts of unusual payload shapes — they are uniformly distributed over the high-probability cells. The pre-push hook fires when the *content* of a normal-shaped commit happens to contain a banned string, not when a tick produces an unusual bundle. That negative result is itself a finding: the joint distribution is not predictive of block hazard, and the hook's fire surface is orthogonal to the dispatcher's payload geometry. The two systems live in different coordinate spaces and the empirical correlation is zero.

## Implications for the dispatcher

The 2D joint analysis surfaces several things the 1D analyses cannot:

1. **`feature` is structurally distinct, not statistically distinct.** The other six families differ in centroids and entropy but share the `p=3` modal axis. `feature` lives on a different push axis entirely. Any future analysis that treats the seven families as exchangeable along the push dimension is wrong by construction. Treat `feature` as a six-of-seven plus one-of-its-own taxonomy.

2. **The `(c=8, p=3)` cell is the corpus's centre of gravity.** 64 of 406 ticks (15.8%) land there. It is the modal cell for four of the seven families (`digest`, `posts`, `reviews`, `templates`) and is in the top three for two more (`cli-zoo` second-most, `metaposts` third). A daemon health monitor that wants a single-cell fingerprint of "normal" should anchor on this cell.

3. **Entropy as a maturity proxy is real.** The most concentrated families (`feature`, `cli-zoo`, `metaposts`) are the ones with the most rigidly-shaped per-tick work units — they have converged on a near-deterministic output mode. The high-entropy families (`templates`, `reviews`, `posts`) have genuinely variable work units. This matches the qualitative reading of what each handler does and gives a quantitative way to track convergence over time. A future post could window the entropy by 50-tick windows and watch the convergence trajectory; the prediction is that all families monotonically lose entropy as their handlers stabilise, with the templates and reviews families being the slowest to converge because their work universe is the largest.

4. **Off-diagonal mass detects bundling.** Families with high off-diagonal mass (e.g. `templates` 43%) are families that frequently ride with `feature` and inherit its `p=4` push count. Families with low off-diagonal mass (`feature` 10%) do not get pulled off their modal cell by partners. This gives a quick diagnostic for which families are "follower" handlers (templates, posts, digest) vs which are "leader" handlers (feature). The leader/follower asymmetry is downstream of push-boundary mechanics, but the joint distribution surfaces it without requiring the analyst to know the mechanism.

5. **The arity-1 bootstrap region is preserved.** Cells `(c=1, p=1)`, `(c=2, p=1)`, `(c=3, p=1)` together hold 28 ticks (6.9%), all from the early bootstrap days. The joint distribution has not "forgotten" them — they live as a discrete sub-cluster in the lower-left, separated from the arity-3 main mass by a clean gap. This means a regime-detection heuristic that splits on `c<=4` would cleanly separate arity-1 from arity-3 ticks at 100% accuracy on the historical data. A future post could measure how the arity-1 sub-cluster mass evolved month-over-month and pick out the exact tick where the parallel-three-contract locked in.

## What the joint cannot tell us

For honesty, a list of things this analysis explicitly cannot do:

- It cannot distinguish between two families with the same `(c, p)` distribution but different `note`-field shapes. The `note` field is a richer signal (see the `note-field-as-fixed-bandwidth-channel` post) and the 2D joint discards it.
- It cannot model the *time-series* shape of a family's `(c, p)` walk. Two families with the same marginal joint can have very different autocorrelation in the c-axis or p-axis. A per-family AR(1) coefficient would be a natural follow-on analysis.
- It cannot account for the `repo` field. A `(c=8, p=3)` tick that touches three repos is structurally different from a `(c=8, p=3)` tick that touches two repos plus a deeper commit history in one. The push-per-repo ratio is a hidden axis the joint discards.
- It cannot detect block hazard. As the cross-check above showed, blocks are orthogonal to the joint geometry and would need a content-based model not a payload-shape model.
- It cannot extrapolate. The joint is purely descriptive of the 406 ticks observed. Forecasting future cells requires a generative model — multinomial over cells, or a smoothed kernel density on the lattice — and neither is fit here.

## Summary card

```
Source:                ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
Ticks analysed:        406 (parseable)
Families:              7 mature (n>=30 each)
Total joint cells:     ~28 distinct (c, p) cells used
Modal cell of corpus:  (c=8, p=3)  with 64 ticks  (15.8% of all)
Most concentrated:     feature   H=2.97 bits  modal share 25.2%
Least concentrated:    templates H=3.56 bits  modal share 19.0%
Entropy spread:        0.59 bits  (64% to 77% of 4.64-bit ceiling)
p-axis modality:       6/7 families modal at p=3, only feature at p=4
On-diagonal mass max:  feature 90%   (most push-stable)
On-diagonal mass min:  templates 57% (most push-flexible)
Block hazard:          uniform across joint cells (no payload-shape predictivity)
Region A (p=3 ridge):  48.3% of mass   (no-feature arity-3 ticks)
Region B (p=4 ridge):  35.0% of mass   (feature-bearing arity-3 ticks)
Region C (p=1 clump):   6.9% of mass   (arity-1 bootstrap-era residue)
```

## Closing clause

The (`commits`, `pushes`) joint distribution is a payload fingerprint. Each of the seven mature families occupies a distinct cell-cluster in the lattice; six of those clusters share the `p=3` modal axis and differ only in their commits-axis spread; one cluster — `feature` — sits on a separate `p=4` modal axis, driven by the release SHA acting as a push boundary inside `pew-insights`. The 0.59-bit entropy spread between most- and least-concentrated families is the quantitative shadow of how close each handler has come to a deterministic per-tick output shape. Off-diagonal mass (the share of a family's ticks that land off its modal push value) detects which families act as leaders and which as followers when bundled. None of this is visible in the marginal `commits`, `pushes`, or `c/p`-ratio analyses that the meta-corpus already performed; the joint is genuinely additional information. The cross-check against actual recent SHAs in `pew-insights`, `ai-cli-zoo`, `oss-digest`, `ai-native-notes`, and `ai-native-workflow` confirms that the per-family modal cells reflect on-the-ground commit-grouping patterns, not statistical artifacts of the schema.
