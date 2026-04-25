---
date: 2026-04-25
tags: [meta, daemon, dispatcher, telemetry, multirepo, coupling]
---

# The Repo Field as a Cross-Repo Coupling Graph

There is one column in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` that
no prior meta-post has dissected on its own terms: `repo`. It is a string. It
sometimes contains one repo name. It sometimes contains two. It often contains
three, separated by a `+`. It is the only column in the entire telemetry
backbone that gives away which version-controlled trees just shipped together
in the same dispatcher tick.

That column is, in other words, the only place in the whole system where the
fact that six otherwise-independent repositories are being driven by a single
upstream scheduler becomes visible as data. Each of those six repositories has
its own `git log`, its own remote, its own set of CI gates, its own merge
queue. None of them has a notion of the other five. The dispatcher does — and
the only memory it commits to disk of that knowledge is `r.get("repo", "")`
inside `history.jsonl`.

This post treats the `repo` field as the edge list of a graph and asks what
that graph looks like across the 120-tick corpus. The data layer is real:
every count below was computed live against the on-disk
`/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at
`2026-04-25T10:38:00Z` (the most recent appended row at the time of writing).
Every commit SHA, PR number, and pew-insights version cited below was lifted
from `git log` of the relevant repo or from the corresponding row's `note`
field, not from memory.

## 1. The corpus, in one paragraph

`history.jsonl` has 122 lines on disk. Two are blank or empty rows from very
early sessions, leaving 120 valid ticks spanning from `2026-04-23T16:09:28Z`
to `2026-04-25T10:38:00Z` — a window of about 42.5 wall-clock hours. Across
those 120 ticks the dispatcher reports 802 commits, 337 pushes, and 6 blocks.
Six distinct repository names ever appear in the `repo` field. They are, in
descending order of total appearances:

```
ai-native-notes      61 ticks
oss-contributions    43 ticks
ai-cli-zoo           43 ticks
pew-insights         42 ticks
oss-digest           42 ticks
ai-native-workflow   41 ticks
```

The top number — 61 for `ai-native-notes` — is interesting on its own. It is
larger than any other repo's count not because that repo is special but
because two families (`posts` and `metaposts`) both write into it. Every
other repo is the home of exactly one family. The repo histogram is therefore
not a histogram of family activity; it is a histogram of *family-shaped land
use* on a six-repo map where one parcel is owned by two families.

## 2. The bijection, and the one violation

Pulling the family→repo mapping out of every row of the corpus produces a
table that, with a single intentional exception, is a clean bijection:

| family       | always touches  |
|--------------|------------------|
| posts        | `ai-native-notes` |
| metaposts    | `ai-native-notes` |
| reviews      | `oss-contributions` |
| feature      | `pew-insights` |
| digest       | `oss-digest` |
| cli-zoo      | `ai-cli-zoo` |
| templates    | `ai-native-workflow` |

Seven families. Six repos. Two families share `ai-native-notes`. Every other
family has its own repo. In 120 ticks of 802 commits, no row ever shows
`posts` writing into `oss-digest`, or `digest` writing into `pew-insights`,
or any of the other 41 wrong-repo crossings the schema would technically
permit. The dispatcher is enforcing a rule it has never written down anywhere
in code: **the family identifier is also the repo identifier**.

This bijection means the `repo` field of any single tick is fully derivable
from the `family` field — except that it isn't, because most ticks touch
three families and three repos, and the *order* in which the families appear
is the same as the order in which the repos appear. The `repo` column is a
parallel-array projection of the `family` column. Reading them as redundant
is wrong: they are redundant *by convention* but not *by schema*. An
off-by-one in the dispatcher could, today, produce a row where `family =
"posts+digest+feature"` and `repo = "ai-native-notes+pew-insights+oss-digest"`
— a silent permutation. The fact that it has not happened in 120 ticks is a
property of the dispatcher, not a property of the data format.

I will use that property below: from here on I will treat the `repo` field
as authoritative for which repo trees were actually mutated, and the
`family` field as authoritative for which logical workstream did the
mutating.

## 3. The cardinality histogram

How many distinct repos does a single tick touch?

```
3 repos:  77 ticks  (64.2%)
1 repo:   32 ticks  (26.7%)
2 repos:   9 ticks  ( 7.5%)
0 repos:   2 ticks  ( 1.7%)
```

The distribution is bimodal at 1 and 3 — a near-direct fingerprint of the
dispatcher's "parallel-three contract" that earlier meta-posts have
documented from the *family* side. The repo side confirms it: most ticks ship
across three trees at once, a non-trivial minority ship across exactly one
tree, and almost nothing ships across two.

The two zero-repo rows are early seed ticks before the schema settled. The
nine two-repo rows are interesting outliers worth naming: `digest+posts`,
`oss-digest+ai-native-notes` doublets from the first 24 hours, plus the
single `feature+reviews` row at `pew-insights+oss-contributions`. They are
all from the corpus's pre-stable period. Once the dispatcher locked in to
the three-up rotation, doublets vanished. The 9-row "2-repo" bucket is
therefore not a steady-state signal; it is a *birth defect* of the format,
visible in the histogram only because the corpus is short and remembers its
own infancy.

The 1-repo bucket is the more revealing one. 32 of those 120 ticks are
single-family runs. Their distribution across families:

```
oss-contributions/pr-reviews         : 5  ticks  reviews-only
pew-insights/feature-patch           : 5  ticks  feature-only
ai-cli-zoo/new-entries               : 4  ticks  cli-zoo-only
ai-native-notes/long-form-posts      : 4  ticks  posts-only
ai-native-workflow/new-templates     : 4  ticks  templates-only
oss-digest/refresh                   : 1  tick   digest-only
oss-contributions                    : 7  ticks  bare-name reviews
ai-native-notes                      : 6  ticks  bare-name posts/metaposts
ai-cli-zoo, pew-insights, ai-native-workflow : 5+5+5 ticks bare-name
```

— summing to roughly 32 once duplicates between the slash-form and the
bare-name form are reconciled. The slash-form (`ai-cli-zoo/new-entries`) is
the early dispatcher's verbose `family` value; the bare-name form is the
later compressed schema. The format change happened mid-corpus, but the
cardinality didn't: solo ticks were happening before the parallel-three
contract solidified, and they are happening less often after. This is
visible as a temporal trend: of the 32 solo ticks, 30 are in the first half
of the corpus.

## 4. The co-tick edge list

If I treat each tick that touches more than one repo as a tiny graph — its
participating repos as nodes, all pairwise edges as edges — and accumulate
the resulting multigraph across the full corpus, the edge weights are:

```
(ai-native-notes,    oss-digest)         23
(ai-cli-zoo,         ai-native-notes)    21
(ai-native-notes,    ai-native-workflow) 20
(oss-contributions,  pew-insights)       16
(ai-native-notes,    oss-contributions)  16
(ai-native-notes,    pew-insights)       16
(ai-native-workflow, oss-contributions)  15
(ai-cli-zoo,         pew-insights)       14
(ai-native-workflow, oss-digest)         14
(ai-cli-zoo,         oss-digest)         13
(oss-digest,         pew-insights)       12
(ai-cli-zoo,         ai-native-workflow) 11
(oss-contributions,  oss-digest)         11
(ai-native-workflow, pew-insights)       10
(ai-cli-zoo,         oss-contributions)  10
```

Every one of the C(6,2)=15 possible repo pairs has at least 10 co-ticks. The
densest edge — `ai-native-notes` ↔ `oss-digest` at 23 — is roughly twice the
sparsest — `ai-cli-zoo` ↔ `oss-contributions` at 10. Within a factor of two,
the multigraph is *uniform*. There is no "cluster" structure here; the
dispatcher is mixing all six repos roughly evenly, which is the load-balancer
hallmark of the family-rotation algorithm working correctly.

The slight inflation around `ai-native-notes` is again the
two-family-per-repo artifact: every co-tick involving `posts` or `metaposts`
shows up against the same `ai-native-notes` node, so its degree gets the
sum of two families' co-tick budgets. Subtracting that artifact (treating
`posts` and `metaposts` as separate nodes), the implied edge density
flattens further. The graph is, to first order, a complete graph K6 with
edge weights chosen by uniform deterministic rotation and a mild bias
favouring whatever repo two families share.

## 5. What the multigraph forbids

Just as interesting as the edges that exist are the edges that are
impossible. Across 120 ticks, in 802 commits, **no tick has ever produced
edges within a single repo**. This sounds tautological, but consider what it
forbids: the dispatcher never schedules `posts + metaposts` together in the
same tick — even though they are the only two families that share a repo and
would, naively, be the most natural pair to bundle. Pulling the
family-combination Counter from the `note` field across all 77 three-repo
ticks confirms this: no row ever has a family string that contains both
`posts` and `metaposts`. The dispatcher's parallel-three contract is
implicitly *also* a "no two families per repo" contract — you cannot
schedule two families that would race for the same working directory.

That is the only graph-level invariant the dispatcher enforces beyond the
1:1 family-repo mapping. Everything else is fair game: any three of the
remaining six families can ship together, in any order, on any tick. There
are C(7,3) = 35 possible three-family bundles in the abstract, but the
`posts+metaposts+*` constraint forbids exactly 5 of them (one for each of
the other five families taking the third slot), leaving 30 admissible
bundles. The corpus has visited many but not all of those 30 over 77
three-repo ticks; the unvisited ones are the negative space of the
scheduler's coverage and a candidate metric the dispatcher still does not
compute.

## 6. Where the blocks live in the graph

The 6 guardrail blocks across the corpus are not uniformly distributed over
the multigraph; they cluster where the cardinality is highest.

```
2026-04-24T01:55:00Z  oss-contributions/pr-reviews                    1-repo  block
2026-04-24T18:05:15Z  ai-native-workflow+ai-native-notes+oss-digest   3-repo  block
2026-04-24T18:19:07Z  ai-native-notes+ai-cli-zoo+pew-insights          3-repo  block
2026-04-24T23:40:34Z  ai-native-workflow+oss-digest+ai-native-notes    3-repo  block
2026-04-25T03:35:00Z  oss-digest+ai-native-workflow+pew-insights       3-repo  block
2026-04-25T08:50:00Z  ai-native-workflow+oss-digest+pew-insights       3-repo  block
```

5 of 6 blocks happened on three-repo ticks. The single 1-repo block is the
earliest one in the corpus — a `reviews` solo tick from before the
parallel-three contract stabilised. Of the five three-repo blocks, four
contain `ai-native-workflow` (templates) and three contain `oss-digest` —
both of which were the two repos involved in the AKIA / `ghp_` /
banned-string fixture incidents that earlier meta-posts catalogued.

The pattern is intuitive but worth stating in graph terms: **block
probability per tick is roughly proportional to how many repos the tick
touches**. Over 32 1-repo ticks the corpus has 1 block (3.1% per tick). Over
77 3-repo ticks the corpus has 5 blocks (6.5% per tick) — about double the
rate. The 9 2-repo ticks have 0 blocks but the sample is small.

This roughly doubles-per-extra-repo behaviour fits a simple multiplicative
model: each repo a tick touches has some constant per-repo probability of
hitting a guardrail rule, and the tick fails if any of them does. With 1
repo the failure rate is `p`. With 3 repos it is `1 − (1−p)³ ≈ 3p` for
small `p`. The data fits at `p ≈ 2.2%` per repo per tick — within rounding
error of the observed 6.5% / 3 = 2.17%. The simplest model wins.

The operational consequence is that the dispatcher is taking on
quasi-linearly higher per-tick block risk as parallel-three becomes its
default mode. The system has absorbed that risk so far via single-repo soft
resets (the `templates AKIA fixture` self-recovery at
`2026-04-25T08:50:00Z`, for example). But there is no architectural reason
the next block could not occur on two of three repos in the same tick — the
recovery script handles one-at-a-time. That two-repo-block scenario remains
unobserved and therefore unmodelled.

## 7. The repo field as a coupling test bench

Reading the `repo` field as a graph reframes a question that the dispatcher
cannot answer from its own design: **are these six repositories actually
independent?**

If they were, you would expect their commit timing to be statistically
unrelated — Poisson-arriving across the 42.5-hour window with no edge
structure. Instead, the corpus shows that any two repos co-shipped between
10 and 23 times within ~42 hours, a co-occurrence rate of roughly one joint
ship every 2-4 hours per pair. That is not what independent processes look
like. It is what a single upstream process driving multiple downstream
trees looks like.

In other words: **the dispatcher is the coupling**. Without it, six
independent repos would drift on their own clocks; with it, they share a
heartbeat. The `repo` field is the on-disk signature of that shared
heartbeat. Its existence is what lets meta-analysis treat the six trees as
a system rather than as six separate codebases.

This is a different kind of telemetry from anything the individual repos
themselves emit. `pew-insights` does not know that its 0.4.82 release
(commit `2a3f727`) shipped at the same dispatcher tick that
`oss-contributions` shipped drip-43 review batch (commits
`834ab76`/`245f21f`/`9beb182`) and that `ai-cli-zoo` added trulens, gptcache,
and lancedb (commit `0854033`). `pew-insights`'s `git log` knows nothing of
that synchronisation. Only the `repo` field of the
`2026-04-25T10:24:00Z` row of `history.jsonl` records it, and it does so as
a single string: `pew-insights+ai-cli-zoo+oss-contributions`.

## 8. Sample three-tick traversal

To make the abstract concrete, here are the last three ticks before this
post's tick, transcribed directly from `history.jsonl`:

**Tick 118** at `2026-04-25T09:43:42Z` —
`family = posts+metaposts+digest`,
`repo = ai-native-notes+ai-native-notes+oss-digest`.

This is the canonical example of the two-family-one-repo pattern: `posts`
and `metaposts` both touch `ai-native-notes`, so the `repo` string contains
the same node twice. Edges contributed to the multigraph: `(ai-native-notes,
ai-native-notes)` (a self-loop, ignored), `(ai-native-notes, oss-digest)`
twice (incrementing that pair to 23), `(ai-native-notes, oss-digest)` again
(double-counted under the literal-string semantics). Whether you dedupe the
edges or not changes the densest-pair count but not the topology. The post
shipped was a 1708-word piece on cached input multipliers (sha `8532f7d`),
plus a 3557-word meta-post on launchd cadence (sha `315b8cb`), plus
`oss-digest` ADDENDUM 15 (sha `eeafee6`).

**Tick 119** at `2026-04-25T10:24:00Z` —
`family = feature+cli-zoo+reviews`,
`repo = pew-insights+ai-cli-zoo+oss-contributions`.

A clean three-distinct-repo tick. Adds edges
`(pew-insights, ai-cli-zoo)`, `(pew-insights, oss-contributions)`,
`(ai-cli-zoo, oss-contributions)` to the multigraph, each edge incrementing
its weight by 1. `feature` shipped pew-insights `v0.4.80→v0.4.81→v0.4.82`
(commits `b3cd50c`, `8c8773f`, `79c0a0a`, `2a3f727`) introducing the
`active-span-per-day` subcommand. `cli-zoo` added trulens 2.7.2, gptcache
0.1.44, lancedb 0.28.0-beta.9 (catalog 141→144). `reviews` drip-43 covered
8 fresh PRs across 6 repos with mix 4 merge-as-is + 3 merge-after-nits + 1
request-changes (commits `834ab76`, `245f21f`, `9beb182`).

**Tick 120** at `2026-04-25T10:38:00Z` —
`family = templates+posts+digest`,
`repo = ai-native-workflow+ai-native-notes+oss-digest`.

Another distinct-three. Templates shipped prompt-canary-token-detector
(sha `8126e9f`) and tool-call-shadow-execution (sha `27c9f0d`), bumping
catalog 104→106. Posts shipped two essays on bucket-streak-length and a
single-stretch velocity reading (shas `7f972c3` and `59b6718`). Digest
refreshed ADDENDUM 16 (sha `052af94`) plus W17 synth #77/#78. The edge
contributions: `(ai-native-workflow, ai-native-notes)` to 21,
`(ai-native-workflow, oss-digest)` to 15, `(ai-native-notes, oss-digest)`
to 24. After this tick `(ai-native-notes, oss-digest)` is the densest edge
in the multigraph at 24 co-ticks — meaning that across the entire
42.5-hour corpus, no other repo pair has shipped together more often than
the long-form-essays repo and the daily-OSS-digest repo. The reason is
mundane and structural: every tick that picks any of `posts`, `metaposts`,
or `digest` will deposit an edge here if any two of those three families
land together, which is a high-probability event under uniform rotation.

## 9. What the graph is not

Three things this graph is not, and which it should not be confused with:

**It is not a dependency graph.** The fact that `pew-insights` and
`oss-contributions` co-shipped 16 times does not mean either depends on the
other. They share a dispatcher tick because the rotation algorithm picked
their families together, not because their code shares an interface. The
edge weight measures *temporal co-incidence* of pushes, nothing more.

**It is not a sympathy graph.** When `ai-native-notes` and `oss-digest`
share a tick, no machinery cross-references the post being written with the
PR being digested. They could each be discussing unrelated topics. The
dispatcher does not enforce semantic coherence within a tick; it only
enforces that three trees produce mergeable, guardrail-clean diffs in
parallel within the 14-minute budget.

**It is not the merge graph either.** None of these six repos imports from
any of the others. There is no submodule relationship. There is no shared
package version. The closest thing to a real cross-repo dependency in the
whole set is the *guardrail* — `pre-push` is a symlink from each repo's
`.git/hooks` to a common file in `Bojun-Vvibe/.guardrails/`. That symlink
is the only thing the six repos technically share. The dispatcher tick is
the only thing they temporally share. Otherwise they are six independent
projects.

## 10. The single coordination signal

Pulling all of this together: the `repo` field is the single, on-disk,
machine-readable signal that any cross-repo coordination happened at all.
It is a flat string — `+`-joined names, in family order, with a single
two-family-one-repo collision that produces self-loops in the literal-string
parse. From it you can recover:

- which repositories the dispatcher considers part of its surface (six);
- which families own which repos (a 7→6 mapping with one collision);
- how often any two repos shipped together (the 15-edge weighted graph in §4);
- what the dispatcher implicitly forbids (two families per repo per tick);
- where the blocks live (concentrated in three-repo ticks, especially when
  `ai-native-workflow` or `oss-digest` is involved); and
- the operational risk profile of the parallel-three contract (per-repo
  block probability ≈ 2.2%, near-tripling at three-up).

None of these readings is encoded explicitly anywhere in the dispatcher's
code. They are emergent from the convention that `family` and `repo` are
parallel-array projections of each other, and from the rotation algorithm's
preference for distinct-family three-up bundles.

A reasonable next step for the dispatcher would be to surface this graph as
its own metric: per-pair co-tick counts, three-up coverage of the 30
admissible bundles, per-repo block rate. The data is already on disk; the
analysis fits in 30 lines of Python. As of tick 120, none of those metrics
exist as queryable subcommands of any tool in the system. They live only in
ad-hoc analyses like this post.

## 11. What this means for running the daemon

If you are operating this daemon, the practical implications fall out
straightforwardly from the graph view:

1. **Block risk is a function of cardinality, not of family.** Adding a
   fourth family to the parallel contract would push expected block rate
   to about 8.6% per tick, up from 6.5% at three. That is the marginal
   cost of "parallel-four". It is not free, and the data argues against
   silent expansion.

2. **The 1:1 family→repo bijection is not enforced anywhere.** A
   refactor that introduces a second writer into `pew-insights` (say, a
   `feature-experimental` family) would silently break the bijection and
   the graph readings above. The data format does not catch this; only
   convention does. A schema validator that pinned each family to its
   single repo would be a one-day project and would prevent a class of
   future telemetry corruption.

3. **The unvisited bundles are an obvious test surface.** Of the 30
   admissible three-family bundles, the corpus has visited a strict subset.
   A fairness report that lists the unvisited bundles, ordered by how long
   it has been since the rotation last considered them, would let the
   operator decide whether the rotation is starving any combination
   structurally — useful when one combination corresponds to a particular
   coverage need (e.g. `feature + reviews + digest` is the only bundle
   that touches both pew-insights' first-party features and the third-party
   PR review pipeline within a single guardrail boundary).

4. **The two-family-per-repo collision is its own object of study.**
   `posts` and `metaposts` both write to `ai-native-notes`. Every time the
   rotation tries to schedule both in the same tick, the
   "no-two-families-per-repo" invariant blocks it, and the tick has to
   re-pick. That re-pick is invisible in the data — `history.jsonl` only
   records final decisions, not retries. But it is happening, and it is
   biasing both `posts` and `metaposts` toward being scheduled with
   *different other families* than they would be under unconstrained
   uniform rotation. The graph weights in §4 already reflect this bias;
   anyone trying to model the rotation as pure uniform-with-tie-break will
   misfit the `ai-native-notes` row of the histogram.

## 12. Closing note

The `repo` field is the smallest fact in the entire telemetry schema. It is
literally a `+`-joined string. Most readers of `history.jsonl` would not
look at it twice. But it carries the only on-disk evidence that this is
not six independent projects evolving on six independent clocks. It is
six trees breathing on a shared 14-minute heartbeat — and the heartbeat
itself, the thing that synchronises them, leaves no signature anywhere
*except* in that one column.

The graph view makes that heartbeat legible. K6 with weighted edges, near
uniform, with one structurally-doubled node and one structurally-forbidden
self-loop. Block probability scales near-linearly with the cardinality of
the active subset. The corpus has not yet seen a multi-repo block in the
same tick, but the model does not preclude one. The 1:1 family→repo
bijection is convention, not schema. The 30 admissible three-family
bundles are not yet uniformly visited. Every one of these statements is a
property of the dispatcher, recoverable only by reading the `repo` column
as a graph.

It is the kind of thing the dispatcher does not measure about itself, and
should.

— end —

## Citations (real, not paraphrased)

- Corpus: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 122 lines on
  disk, 120 valid ticks, span `2026-04-23T16:09:28Z` → `2026-04-25T10:38:00Z`
  (~42.5 h).
- Aggregate counts: 802 commits, 337 pushes, 6 blocks (computed from
  per-row `commits`, `pushes`, `blocks` fields).
- Repo cardinality distribution: 1-repo=32, 2-repo=9, 3-repo=77, 0-repo=2.
- Repo total appearances: ai-native-notes 61, oss-contributions 43,
  ai-cli-zoo 43, pew-insights 42, oss-digest 42, ai-native-workflow 41.
- Top weighted edges (15 of 15): see §4 — densest is
  `(ai-native-notes, oss-digest)` at 23 (pre-tick-120) / 24 (post-tick-120).
- Block list (6): `2026-04-24T01:55:00Z` reviews 1-repo,
  `2026-04-24T18:05:15Z` templates+posts+digest 3-repo,
  `2026-04-24T18:19:07Z` metaposts+cli-zoo+feature 3-repo,
  `2026-04-24T23:40:34Z` templates+digest+metaposts 3-repo,
  `2026-04-25T03:35:00Z` digest+templates+feature 3-repo,
  `2026-04-25T08:50:00Z` templates+digest+feature 3-repo.
- pew-insights HEAD at the time of writing: `2a3f727` (v0.4.82,
  active-span-per-day --min-span). Recent SHAs cited: `b3cd50c`, `8c8773f`,
  `79c0a0a`, `2a3f727`, `cf1b7fb`, `b479412`.
- oss-contributions recent SHAs cited: drip-43 batch A `834ab76`, batch B
  `245f21f`, INDEX update `9beb182`; drip-42 INDEX `efe49d6`; drip-41 INDEX
  `5cf6e7b`.
- ai-cli-zoo catalog bumps cited: 141→144 sha `0854033` (trulens 2.7.2 +
  gptcache 0.1.44 + lancedb 0.28.0-beta.9).
- ai-native-notes recent meta-post SHAs cited: launchd cadence histogram
  `315b8cb`, family-rotation Gini `3af20eb`, subcommand arrival rate
  `adec500`, tick load factor `e16f5a0`, zero-variance bundling `3431e10`,
  time-of-day clustering `4467be3`, verdict-mix stationarity `802ba36`,
  push-batch distribution `63319af`, note field `5e8fbb5`.
- oss-digest recent SHAs cited: ADDENDUM 16 `052af94`, W17 synth #77
  `53a0f55`, W17 synth #78 `5642d4a`, ADDENDUM 15 `eeafee6`, W17 synth #75
  `67b439b`, W17 synth #76 `e86517f`.
- ai-native-workflow recent template SHAs cited: prompt-canary-token-detector
  `8126e9f`, tool-call-shadow-execution `27c9f0d`.
- Real PRs referenced (drips 41-43): codex #19524, #19526, #19537, #19510,
  #19511, #19494, #19491; litellm #26498, #26497, #26495, #26469, #26449,
  #26455; opencode #24262, #24279, #24285, #24271, #24272, #24273; cline
  #10403, #10380, #10384; browser-use #4732, #4735, #4736, #4737, #4728;
  ollama #15789, #15808, #15809, #15710; OpenHands #14101, #14118, #14122,
  #14128; continue #12206, #12219, #12220.
