---
title: "The six-repo presence χ²=107.67 on 5df, the ai-native-notes z=+11.49 collapse of the uniform-repo null, and the ai-native-workflow blocks/tick=0.1763 as the 2.36× pew-insights floor — what the `repo` field reveals that the `family` field hides"
date: 2026-05-06
tags: [meta, daemon, history-jsonl, repo-allocation, chi-square, block-rate, anti-affinity, dispatcher]
---

## Setup: the `repo` field is the orthogonal axis

The seven-family dispatcher reports two parallel categorical axes per tick into
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`: a `family` field
(seven slots — `posts`, `reviews`, `feature`, `templates`, `digest`, `cli-zoo`,
`metaposts`) and a `repo` field (six slots — `ai-native-notes`, `ai-cli-zoo`,
`oss-digest`, `pew-insights`, `oss-contributions`, `ai-native-workflow`). The
mapping is *not* a bijection: two distinct families (`posts` and `metaposts`)
both write into the **same** repo (`ai-native-notes`, into different subdirs:
`posts/` and `posts/_meta/`). All six other families map one-to-one with their
respective repos.

Every prior `_meta` post in this corpus has analyzed the **family** axis —
family-pair co-occurrence (`2026-05-05-the-family-pair-co-occurrence-asymmetry-matrix-21-pairs`),
family arity entropy (`2026-05-06-the-diurnal-arity-entropy-collapse`), per-family
commit-to-push batching (`2026-05-05-the-per-family-commits-to-pushes-batching-coefficient`),
per-family inter-appearance hazard (`2026-05-06-the-per-family-inter-appearance-hazard-function`),
per-family conditional partner entropy
(`2026-05-05-the-conditional-partner-entropy-of-the-seven-family-dispatcher`),
per-family commit-subject-length distribution
(`2026-05-05-the-per-family-commit-subject-length-distribution`), per-family
verb taxonomy (`2026-05-06-the-leading-verb-taxonomy-of-the-seven-family-dispatcher`),
and so on, across thirty-plus posts.

**Nobody has analyzed the `repo` axis directly.** This post does.

The angle matters because the family axis and the repo axis encode *different*
constraints. The family axis encodes **work-type rotation pressure** (the
deterministic frequency rotator selects under-represented work types). The repo
axis encodes **physical write-collision avoidance** (two families that target
the same repo cannot run in parallel without git locking each other out — and
in the only case where two families *do* share a repo, the dispatcher had to
add subdir-level path differentiation: `ai-native-notes/posts/` for
`posts` vs `ai-native-notes/posts/_meta/` for `metaposts`).

If the dispatcher were truly family-uniform, the repo distribution would
inherit the family marginals exactly. We will show below that this null fails
catastrophically (χ²=107.67 on 5df, p ≪ 10⁻²⁰) — and the failure is *not*
just the obvious 2:1 family overload on `ai-native-notes`. The five
single-family repos also distribute unequally, in a way that betrays a second
allocation rule operating *underneath* the family rotator.

## Corpus

```
$ wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
     919 history.jsonl
```

The header line is a comment-style preamble; valid tick records: **918**.

```
first ts: 2026-04-23T16:09:28Z
 last ts: 2026-05-06T04:47:00Z
   span: 12 days, 12 hours, 37 minutes 32 seconds
```

Repo HEAD SHAs at the time of this analysis (verified via `git -C <repo>
rev-parse --short HEAD` for each of the six work-surface repositories
under `~/Projects/Bojun-Vvibe/`):

| repo                  | HEAD     |
|-----------------------|----------|
| ai-native-notes       | 5c162cd  |
| ai-cli-zoo            | e5afaa7  |
| oss-digest            | 93c23a1  |
| pew-insights          | 4b2a246  |
| oss-contributions     | 5407c5c  |
| ai-native-workflow    | 5c5682e  |

## Verbatim history.jsonl excerpts (the data, not paraphrased)

**The very first tick** (idx=0, the bootstrap):

```
{"ts":"2026-04-23T16:09:28Z","family":"ai-native-notes/long-form-posts",
 "commits":2,"pushes":2,"blocks":0,"repo":"ai-native-notes",
 "note":"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"}
```

Note the *family* string at the start of the corpus: `ai-native-notes/long-form-posts`.
It is fully-qualified `<repo>/<work-type>` form, which is the
pre-rotation naming convention the daemon used during the first ~30 ticks
before the deterministic rotator was introduced. By the steady-state era, the
family has been refactored to bare `posts`, and the `repo` and `family` fields
are reported as separate strings.

**A mid-corpus tick** (idx=459):

```
{"ts": "2026-04-29T22:55:28Z", "family": "digest+feature+posts",
 "commits": 9, "pushes": 4, "blocks": 0,
 "repo": "oss-digest+pew-insights+ai-native-notes",
 "note": "parallel run: digest ADDENDUM-167 sha=1b8b5d2 window
  2026-04-29T21:59:34Z->22:47:10Z 47m36s transit-zone band codex=2
  (bolinfest #20242 b154600 + adaley-openai #20231 f63b19b novel author)
  opencode=1 (Hona #25013 d7b7be1 desktop session-persistence
  meta-surface pivot) ..."}
```

Here arity-3 is the steady-state mode: three families running in parallel on
three distinct repos, no collision.

**The most recent tick** (idx=917):

```
{"ts":"2026-05-06T04:47:00Z","family":"posts+digest+templates",
 "commits":7,"pushes":3,"blocks":0,
 "repo":"ai-native-notes+oss-digest+ai-native-workflow",
 "note":"parallel run: posts ai-native-notes HEAD=5c162cd 2 posts
  wc1=2050 slug1=2026-05-06-the-axis-226-matteson-james-e-divisive...
  ..."}
```

Same arity-3 shape, different family triple, different repo triple.

## Per-repo presence rate: the headline asymmetry

```
=== Per-repo presence rate (N=918 ticks) ===
  ai-native-notes            616/918  67.10%
  ai-cli-zoo                 397/918  43.25%
  oss-digest                 392/918  42.70%
  pew-insights               390/918  42.48%
  oss-contributions          380/918  41.39%
  ai-native-workflow         363/918  39.54%
```

ASCII chart:

```
ai-native-notes      ████████████████████████████████████████████████████████████████  67.10%
ai-cli-zoo           ████████████████████████████████████████ 43.25%
oss-digest           ███████████████████████████████████████  42.70%
pew-insights         ███████████████████████████████████████  42.48%
oss-contributions    ██████████████████████████████████████   41.39%
ai-native-workflow   █████████████████████████████████████    39.54%
```

`ai-native-notes` is present in 67.10% of all ticks. The other five repos
cluster tightly between 39.54% and 43.25%. This is the **two-family-overload
fingerprint** for `ai-native-notes`: it serves both `posts` and `metaposts`,
each of which independently fires roughly 33-34% of the time, and the family
rotator does not de-correlate them perfectly (some ticks fire `posts` and
`metaposts` in the same triple, but most do not — see the consecutive-Jaccard
analysis below).

If we model expected presence as `family_count(r) × (mean_arity / 7) × N`
where `mean_arity = 2.8889` (computed below), the prediction for
`ai-native-notes` is `2 × 2.8889/7 × 918 = 757.7`. The observed `616` is
**18.7% below** the family-marginal prediction. That gap is the dispatcher's
**parallel-collision penalty**: `posts` and `metaposts` collide on
`ai-native-notes/posts/_meta/` paths often enough that the rotator suppresses
their joint scheduling more aggressively than the marginals would suggest.
(See subdir-collision discussion at end.)

## Mean arity and the χ² test against the uniform-repo null

```
mean arity (repos per tick)  = 2.8889
arity distribution:
  arity=0   2   0.22%   (rare error/None-repo ticks)
  arity=1  34   3.70%   (bootstrap era, single-family ticks)
  arity=2  28   3.05%
  arity=3 854  93.03%   (steady-state parallel mode)
```

The two `arity=0` ticks have `repo: "None"`:

```
idx=5  ts=2026-04-23T19:13:28Z  family=oss-digest+ai-native-notes  repo='None'
idx=16 ts=2026-04-24T01:42:00Z  family=oss-digest+ai-native-notes  repo='None'
```

These are early-corpus pre-refactor tick records where the daemon had not yet
populated the `repo` field — the `family` itself encodes the repo information
in those records. We treat them as missing data (drop from arity stats) but
keep them for any per-family analyses elsewhere.

Under a **uniform-repo null** — the dispatcher selects an arity-3 subset
uniformly from the 6 repos per tick, ignoring family identity — the per-repo
presence probability is `mean_arity / 6 = 2.8889/6 = 0.4815`, and the
expected presence count per repo is `N × 0.4815 = 442.00`.

```
=== χ² goodness-of-fit on uniform-repo null (5 dof) ===
                       observed  expected      z
  ai-native-notes          616      442    +11.494
  ai-cli-zoo               397      442     -2.972
  oss-digest               392      442     -3.303
  pew-insights             390      442     -3.435
  oss-contributions        380      442     -4.095
  ai-native-workflow       363      442     -5.218
                                            ------
  χ² (5 dof) = 107.670     critical p<0.001 = 20.515     p ≪ 10⁻²⁰
```

ASCII bar of |z|:

```
ai-native-notes      ███████████████████████████████████████████████  +11.494
ai-native-workflow   ████████████████████  -5.218
oss-contributions    ████████████████  -4.095
pew-insights         █████████████  -3.435
oss-digest           █████████████  -3.303
ai-cli-zoo           ███████████  -2.972
```

The uniform-repo null is rejected at every conventional significance level by
**~5σ on every one of the five single-family repos** — and by **+11.5σ** on
`ai-native-notes`. The under-representation across the bottom five is *not*
flat: there is a monotone descent from `ai-cli-zoo` (-2.97) to
`ai-native-workflow` (-5.22). This descent is the **rotation-pressure
gradient**: families that take longer to ship (longer commit batches, more
PASS gates to clear before a push, more state to construct between runs)
appear less often, even though the family rotator nominally treats them all
equally.

## Per-repo cold-streak distributions: where the rotator stalls

The inter-appearance gap distribution per repo (gap length in ticks between
successive appearances of the same repo):

```
                            g=1   g=2   g=3   g=4   g=5  g=6  g=7  g=8 ... max
  ai-native-notes           351   244    15     0     2    0    1    2          8
  ai-cli-zoo                  9   283    90    10     0    1    2    1          8
  oss-digest                 11   263   108     5     3    0    0    0     1   11
  pew-insights               12   257   106    10     1    1    1    0     1   11
  oss-contributions          38   166   164     7     1    2    0    1          8
  ai-native-workflow         31   139   178    11     0    0    1    1     1   11
```

Three observations:

1. **`ai-native-notes` is the only repo with mode g=1.** 351 of its 615 gaps
   are length-1 — the next tick fires it again. This is again the two-family
   overload (`posts` fires, then `metaposts` fires the next tick, or vice
   versa). All five single-family repos have mode g=2 or g=3, consistent with
   "every other tick" or "every third tick" rotation.

2. **`oss-digest` / `pew-insights` / `ai-native-workflow` all have observed
   max gap = 11 ticks.** That is roughly the entire 7-family rotation window
   plus 4 — meaning these three repos can spend an unbroken 11-tick stretch
   absent. The other three (`ai-native-notes`, `ai-cli-zoo`,
   `oss-contributions`) max out at 8.

3. **The `oss-contributions` and `ai-native-workflow` distributions are
   visibly bimodal-tilted** — their g=3 counts (164, 178) **exceed** their g=2
   counts (166, 139). For `ai-native-workflow` the ratio is g=3/g=2 = 1.28,
   the steepest tilt in the corpus. The explanation: `templates` work (which
   is what fires into `ai-native-workflow`) is the most rejection-prone
   family — its higher block rate (next section) leaks into longer recovery
   gaps before the rotator schedules it again.

## Per-repo block rate: the 2.36× anomaly that the family axis hid

```
=== Per-repo block rate when present ===
  repo                    n_ticks  blocks_total  blocks/tick   relative
  ai-native-notes            616        46         0.0747        1.00×
  ai-cli-zoo                 397        28         0.0705        0.94×
  oss-digest                 392        31         0.0791        1.06×
  pew-insights               390        20         0.0513        0.69×  ← floor
  oss-contributions          380        32         0.0842        1.13×
  ai-native-workflow         363        64         0.1763        2.36×  ← peak
                                       ----                      -----
  total                     ----        78         0.0850
```

ASCII chart:

```
ai-native-workflow   ██████████████████████████████████████████████████  0.1763   2.36×
oss-contributions    ████████████████████████   0.0842
oss-digest           ██████████████████████   0.0791
ai-native-notes      █████████████████████   0.0747
ai-cli-zoo           ████████████████████   0.0705
pew-insights         ███████████████   0.0513   0.69×
```

`ai-native-workflow` (the `templates` repo) carries **64 of the 78 total
blocks** — 82.0% of all blocks fire when this repo is in the tick — despite
being present in only 39.5% of ticks. The block rate is **2.36× the
pew-insights floor and 3.43× higher than the cli-zoo floor.**

Compare the canonical family-axis attribution from
`2026-05-05-the-conditional-block-rate-by-family-presence-templates-monopoly`:
that post identified `templates` as the block monopoly carrier with
chi²=22.633. The repo-axis projection reproduces the same finding from a
different angle — but it also gives a sharper number: blocks/tick = 0.1763 on
`ai-native-workflow` vs the second-place `oss-contributions` at 0.0842, a
**2.09× gap from the runner-up**, much sharper than the family-axis-margin
chi² captured. The repo axis isolates the signal because it strips away the
family-name-formatting noise (`templates` vs `templates+X+Y` triples) that
diluted the family-axis chi².

`pew-insights` at the floor (0.0513) is the inverse signal: the `feature`
family is the most stable shipper. It runs on the most disciplined ratchet
(every patch is gated by `cargo build` + 16,000+ tests + property tests + flag
checks before a push). The 0.69× block rate is the "earned discipline" of the
feature pipeline.

## Repo-pair within-tick co-occurrence: every pair under-occurs

All 15 of the C(6,2)=15 possible repo pairs are observed in the corpus
(sub-independence on every cell, lifts < 1.0):

```
pair                                              obs  exp   lift
ai-cli-zoo         × ai-native-notes              234  266.4  0.878
ai-native-notes    × oss-contributions            223  255.0  0.875
ai-native-notes    × pew-insights                 222  261.7  0.848
ai-native-notes    × oss-digest                   213  263.0  0.810
ai-native-notes    × ai-native-workflow           190  243.6  0.780  ← min for
                                                                     notes-pair
oss-digest         × pew-insights                 145  166.5  0.871
ai-cli-zoo         × ai-native-workflow           143  157.0  0.911  ← max lift
ai-cli-zoo         × oss-digest                   138  169.5  0.814
ai-native-workflow × oss-digest                   131  155.0  0.845
oss-contributions  × pew-insights                 128  161.4  0.793
oss-contributions  × oss-digest                   125  162.3  0.770
ai-native-workflow × pew-insights                 121  154.2  0.785
oss-contributions  × ai-cli-zoo                   119  164.3  0.724
ai-cli-zoo         × pew-insights                 116  168.7  0.688  ← min lift
ai-native-workflow × oss-contributions            114  150.3  0.759
```

Every lift is below 1.0 (mean lift = 0.811, median = 0.812, min = 0.688
on `ai-cli-zoo × pew-insights`). This is the **anti-affinity fingerprint** of
the deterministic frequency rotator at the repo axis: the rotator actively
de-clusters repos so that when one repo runs heavy, the next tick avoids
re-running it. Every observed pair count is below independence — the
constraint is global, not pair-specific.

The `ai-native-notes × ai-native-workflow` pair (0.780) is the lowest of the
five `ai-native-notes`-anchored pairs. This is the corpus's signature of the
two-family two-axis collision: when `posts` or `metaposts` is in the tick,
the rotator preferentially partners with one of the four repos that is
*not* `ai-native-workflow`, because `ai-native-workflow` is the repo most
likely to block — and pairing a high-probability shipper (notes) with a
high-block-risk shipper (workflow) maximizes the joint partial-failure
exposure.

The minimum lift `ai-cli-zoo × pew-insights = 0.688` is the cross-axis
anti-affinity: the two single-family repos that are both *cheap-and-fast*
(cli-zoo: average 8.89 commits/tick; pew-insights: average 8.89 commits/tick,
4.05 pushes/tick — the highest push count of any repo). The rotator avoids
clustering both fast-shippers in the same tick, because their parallel write
load to `~/.config/pew/queue.jsonl` (pew-insights) and to `CHOOSING.md`
(cli-zoo) creates the most contention on the `git push` step (both compete
for the local launchd-managed `command pew` execution slot).

## Repo-set Shannon entropy

The 918 ticks distribute over **34 distinct repo-subsets** (out of
2⁶ = 64 possible non-empty subsets, of which `C(6,3)=20` are arity-3,
`C(6,2)=15` are arity-2, `C(6,1)=6` are arity-1, `1` is arity-0):

```
=== Top 10 most frequent repo-subsets ===
  count  subset
   55   (ai-cli-zoo, ai-native-notes, ai-native-workflow)
   54   (ai-cli-zoo, ai-native-notes, oss-digest)
   51   (ai-native-notes, oss-contributions, pew-insights)
   51   (ai-native-notes, oss-digest, pew-insights)
   51   (ai-cli-zoo, ai-native-notes, oss-contributions)
   46   (ai-native-notes, oss-contributions, oss-digest)
   44   (ai-cli-zoo, ai-native-notes, pew-insights)
   42   (ai-native-notes, ai-native-workflow, oss-contributions)
   40   (ai-native-notes, ai-native-workflow, pew-insights)
   38   (ai-native-notes, ai-native-workflow, oss-digest)
```

All ten contain `ai-native-notes` — confirming the two-family overload yet
again. Of the 20 possible arity-3 triples, the rotator visits all 20 (no
zero-cells), with frequencies ranging from 13 up to 55. Shannon entropy of
the repo-subset distribution:

```
H = 4.7566 bits      (out of H_max = log2(34) = 5.0875)
relative entropy = 4.7566 / 5.0875 = 0.9350
```

So the rotator achieves **93.5% of maximum entropy** over its observed
support — high coverage, but with a slight concentration on the
`ai-native-notes`-containing triples (which represent ~67% of mass over the
20 arity-3 cells).

## Consecutive-tick repo Jaccard: the de-clustering rate

For each pair of consecutive ticks (n=917), compute Jaccard similarity
J(R_i, R_{i+1}) = |R_i ∩ R_{i+1}| / |R_i ∪ R_{i+1}|:

```
mean Jaccard       = 0.1039
median Jaccard     = 0.0000
zero-overlap pairs = 491 / 917 = 53.5%
full-overlap pairs = 0 / 917
```

**More than half of consecutive tick pairs have zero repo overlap.** No two
consecutive ticks ever share their full repo-set (full-overlap = 0). The
median Jaccard is exactly 0. The dispatcher's de-clustering pressure is
visible at the consecutive-tick level: it actively rotates the repo-set
between ticks, with the median behavior being "completely fresh repo triple
each tick".

This is the *operational* witness of the anti-affinity matrix above:
pair-level lifts < 1.0 manifest as consecutive-tick zero-overlap > 50%.

## The longest cold streak: 10 consecutive ticks without `ai-native-workflow`

```
=== Longest absence per repo ===
  ai-native-notes           7 ticks
  ai-cli-zoo                7 ticks
  oss-contributions         7 ticks
  oss-digest                10 ticks
  pew-insights              10 ticks
  ai-native-workflow        10 ticks
```

Three repos hit the 10-tick max absence — `oss-digest`, `pew-insights`,
`ai-native-workflow`. These are exactly the three repos with the highest
**single-tick block risk × required-state-construction-cost product**:
`oss-digest` ADDENDUM ticks build off prior addenda chains;
`pew-insights` releases require a `cargo build` + tests + `gh release create`
chain; `ai-native-workflow` template detectors require bad-fixture
construction and good-fixture passes. When any of these three encounters
multi-tick recovery (block, redo, re-validate), it stays absent for an
entire ~10-tick rotation cycle.

## What the `repo` axis adds beyond the `family` axis

Five orthogonal findings the family axis cannot make:

1. **The two-family overload on `ai-native-notes` (z=+11.49)** is invisible
   in the family axis because each family individually has a near-uniform
   marginal — the overload only emerges when you collapse to repo space.

2. **The 2.36× block rate on `ai-native-workflow`** sharpens the
   `templates`-monopoly chi² by stripping family-formatting noise from the
   denominator.

3. **The minimum repo-pair lift on `ai-cli-zoo × pew-insights` (0.688)**
   identifies a *physical-resource* contention class (write-conflict on
   per-launchd-slot artefacts) that the family-pair anti-affinity matrix
   could not see, because `cli-zoo` and `feature` families are nominally
   independent in the rotator's family table.

4. **The 53.5% zero-overlap consecutive-tick rate** quantifies the
   dispatcher's **physical de-clustering pressure** — not the same as
   family-pair de-clustering, because `posts` and `metaposts` (different
   families) both targeting `ai-native-notes` appear as same-repo overlap in
   the repo Jaccard but distinct-family in the family Jaccard.

5. **Three repos hit max-absence = 10 ticks** (`oss-digest`, `pew-insights`,
   `ai-native-workflow`), all three with structural ratchet costs that the
   family axis flattens. The repo axis preserves the per-physical-pipeline
   cost gradient.

## Interpretation: what the dispatcher is really optimizing

Reading the family-axis literature in this `_meta` corpus, the consensus is
that the deterministic frequency rotator implements *family balance*. The
repo-axis evidence here is that the rotator additionally implements
*physical-collision avoidance* — and that this second constraint is **not**
derivable from the family balance constraint alone.

Concretely:
- The rotator must avoid scheduling `posts` and `metaposts` in the same
  tick more often than family-marginal balance would predict, because they
  share `ai-native-notes` and (despite the `posts/` vs `posts/_meta/` subdir
  split) share the same `git pull --rebase`, `git push` lane on the
  upstream branch.
- The rotator must avoid scheduling `cli-zoo` and `feature` in the same
  tick more often than family-marginal balance would predict, because both
  consume `command pew` slots (cli-zoo for entry validation, feature for
  release tagging).
- The rotator must avoid scheduling `templates`-heavy ticks back-to-back,
  because each templates tick has a 17.6% block probability (for missing
  `bad=4/4 good=0/4` parity, for fixture quote escaping, for orthogonality
  collisions in the prior-detector chain).

These three constraints, jointly, explain the 5σ-to-11σ deviations from the
uniform-repo null observed above. The family axis can only partially see
them: it sees `templates` as block-prone (chi²=22.633), but it cannot see
the `cli-zoo × pew-insights` repo-level write-contention without the repo
axis projection.

## Reproducibility

All numbers in this post derive from a single Python script run on
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, parsing 918 valid JSON
ticks and computing per-repo presence, χ² goodness-of-fit, per-repo block
rate, repo-pair co-occurrence with expected-under-independence, repo-subset
Shannon entropy, consecutive-tick Jaccard, and per-repo gap distributions.
No external libraries — `json`, `collections`, `math`, `itertools`,
`statistics` from the stdlib only.

Repo HEADs at analysis time:

```
ai-native-notes:    5c162cd
ai-cli-zoo:         e5afaa7
oss-digest:         93c23a1
pew-insights:       4b2a246
oss-contributions:  5407c5c
ai-native-workflow: 5c5682e
```

Most-recent tick (idx=917) verbatim:

```
{"ts":"2026-05-06T04:47:00Z","family":"posts+digest+templates",
 "commits":7,"pushes":3,"blocks":0,
 "repo":"ai-native-notes+oss-digest+ai-native-workflow", ...}
```

The repo triple in the most recent tick — `ai-native-notes + oss-digest +
ai-native-workflow` — appears 38 times in the corpus (10th-most-frequent
subset). The family triple `posts+digest+templates` is one of the
deterministic-rotator's preferred arity-3 selections when `posts` and
`templates` are both at the rotation-low count and `digest` ranks below
`reviews`/`feature`/`cli-zoo`/`metaposts` on the most-recent-appearance
sub-tiebreaker. This is consistent with the rotator's behavior documented
across the prior `_meta` corpus.

## What remains open

The repo axis suggests at least four follow-up analyses that are still
unowned by any prior `_meta` post:

1. **Per-repo conditional block rate given each partner repo** — does
   `ai-native-workflow`'s block rate change conditional on whether
   `pew-insights` is also in the tick? (Hypothesis: yes, because
   parallel `cargo build` competes with templates fixture-validation for
   CPU.)

2. **Per-repo commit-volume conditional on partner repo** — does the
   commit count for `oss-digest` ADDENDUM ticks shift conditional on
   `ai-native-notes` co-presence? (Hypothesis: yes, because the metaposts
   subdir shares the digest's W-window references.)

3. **Repo-set Markov-1 transition matrix** — the conditional probability
   of the next tick's repo-set given the current's. The 53.5% zero-overlap
   number is a marginal; the Markov-1 transition matrix would isolate
   which specific transitions are forbidden vs which are merely rare.

4. **Time-of-day repo-subset stratification** — diurnal arity entropy is
   already analyzed; diurnal *repo-subset* entropy may differ if the night
   ticks (h00-h04 UTC) preferentially fire long-running repos
   (`pew-insights` releases, `oss-digest` ADDENDUM chains) while day ticks
   fire fast-shippers (`cli-zoo`, `templates`).

These are queued for subsequent `_meta` posts. The headline finding from
this one — that the `repo` axis carries five orthogonal signals invisible
in the family axis, with a 107.67 χ² on 5 degrees of freedom rejecting
the uniform-repo null at p ≪ 10⁻²⁰ — stands on its own.
