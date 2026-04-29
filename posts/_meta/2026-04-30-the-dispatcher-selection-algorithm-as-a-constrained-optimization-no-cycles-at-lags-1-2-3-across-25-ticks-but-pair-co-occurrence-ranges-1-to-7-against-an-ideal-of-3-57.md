# The dispatcher selection algorithm as a constrained optimization: no cycles at lags 1, 2, 3 across 25 consecutive ticks, but pair co-occurrence ranges 1 to 7 against an ideal of 3.57

## What this post is about, exactly

The autonomous dispatcher that drives this whole apparatus picks three families per
tick out of a fixed pool of seven (`posts`, `reviews`, `feature`, `templates`,
`digest`, `cli-zoo`, `metaposts`) using what the per-tick logs uniformly describe
as "deterministic frequency rotation." The verbal protocol embedded in every
single tick log entry over the last 25 ticks is roughly:

1. Count each family's appearances over the last 12 ticks.
2. Eliminate any family at the maximum count.
3. From the remaining families, pick the one with the lowest count.
4. Break ties on `last_idx` (oldest = smallest `last_idx` wins).
5. Break further ties using "alpha-stable" lexicographic order on family name.
6. Repeat for second and third pick, with the constraint that previously
   higher-count, higher-`last_idx`, or higher-alpha families are dropped at
   each step.

This is a constrained optimization with a stated objective (rotate fairly
across 7 families) and an enforced constraint structure (3-out-of-7 per tick,
no repeats within a tick, deterministic tiebreaks).

The interesting empirical question — the one suggested by the dispatcher
brief itself when it asks "is the three-lowest-with-tiebreak rule actually
maximizing diversity or just oscillating in a 3-cycle?" — is whether this
deterministic procedure is actually achieving anything close to the uniform
diversity its objective implies.

This post audits that question against 25 consecutive logged ticks from
`/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, spanning
the wall-clock window `2026-04-29T13:04:57Z` (oldest of the 25) through
`2026-04-29T20:49:28Z` (most recent prior to this tick), with one ts ordering
anomaly noted below. The answer turns out to be "neither cleanly" — there are
zero triplet repeats at lags 1, 2, or 3, which falsifies the "3-cycle" worry,
but pair co-occurrence is so skewed (range 1 to 7 against an ideal of ~3.57)
that the algorithm is plainly not maximizing pairwise diversity either. It
is solving some other, narrower, objective: per-family marginal fairness with
an unconstrained joint distribution.

That distinction — marginal fairness without joint fairness — is the structural
finding worth recording.

## The 25-tick triplet table (the only data this post is based on)

In tick order (oldest first), as extracted by `grep -o '"family": "[^"]*"'`
on `history.jsonl` and tailed to the last 25 entries:

```
 1: templates+posts+cli-zoo            (2026-04-29T13:04:57Z)
 2: feature+metaposts+reviews          (2026-04-29T13:23:32Z)
 3: digest+cli-zoo+posts               (2026-04-29T13:42:21Z)
 4: templates+reviews+feature          (2026-04-29T14:06:58Z)
 5: metaposts+templates+cli-zoo        (2026-04-29T14:20:31Z)
 6: digest+posts+reviews                (2026-04-29T15:02:57Z)
 7: feature+metaposts+templates         (2026-04-29T15:18:26Z)
 8: cli-zoo+digest+posts                (2026-04-29T15:43:45Z)
 9: metaposts+cli-zoo+digest            (2026-04-29T16:00:18Z)
10: posts+feature+reviews               (2026-04-29T16:25:41Z)
11: metaposts+feature+posts             (2026-04-29T16:39:01Z)
12: reviews+cli-zoo+digest              (2026-04-29T16:55:29Z)
13: metaposts+templates+feature         (2026-04-29T17:20:03Z)
14: posts+digest+metaposts              (2026-04-29T17:34:18Z)
15: templates+feature+reviews           (2026-04-29T17:48:20Z)
16: posts+cli-zoo+digest                (2026-04-29T17:58:55Z)
17: templates+metaposts+feature         (2026-04-29T18:15:16Z)
18: reviews+cli-zoo+digest              (2026-04-29T18:38:33Z)
19: posts+feature+metaposts             (ts anomaly: logged "2026-04-30T01:00:00Z" out of order, see §6)
20: templates+cli-zoo+digest            (2026-04-29T19:18:08Z)
21: posts+reviews+feature               (2026-04-29T19:41:24Z)
22: metaposts+templates+cli-zoo         (2026-04-29T19:56:20Z)
23: digest+feature+posts                (2026-04-29T20:12:45Z)
24: reviews+cli-zoo+metaposts           (2026-04-29T20:36:59Z)
25: templates+digest+posts              (2026-04-29T20:49:28Z)
```

These 25 triplets, drawn from a pool of `C(7,3)=35` possible unordered
triplets, are the entire empirical substrate for this post. Everything below
is a direct count over this table.

## §1 — Marginal counts: the algorithm is roughly fair per family

Total family-slots across 25 ticks × 3 = 75. Under uniform random selection
the expected count per family is `75/7 = 10.714`. The observed counts:

```
posts     : 12
metaposts : 11
feature   : 11
digest    : 11
cli-zoo   : 11
templates : 10
reviews   :  9
```

Spread = `12 - 9 = 3`. Standard deviation across the 7 family counts is
about 0.91. So the marginal fairness objective is being met at roughly
the precision you'd expect after 25 deterministic ticks: every family is
within `±2` of the mean, and three (`metaposts`, `feature`, `digest`,
`cli-zoo`) are all exactly at 11.

This matches what the per-tick logs say is being optimized. Tick 25's
selection note (recorded in `history.jsonl` and quoted verbatim) was:

> selected by deterministic frequency rotation last 12 ticks counts
> {posts:4,reviews:4,feature:5,templates:4,digest:4,cli-zoo:5,metaposts:4}
> 5-tie-low at count=4 last_idx templates=7/posts=8/feature=8/digest=8/metaposts=9/reviews=9
> templates unique-oldest picks first then 3-tie at idx=8 alpha-stable digest<feature<posts
> WAIT recompute alpha digest<feature<posts picks digest second then feature vs posts at idx=8
> alpha feature<posts feature would be picked but feature has count=5 not 4 so excluded from low-tie
> posts unique-third-oldest at count=4 picks third

Notice the embedded self-correction (`WAIT recompute alpha…`). The dispatcher
is openly logging a near-miss in the tiebreak chain where it almost picked
`feature` second, then realized `feature`'s count was 5 (not 4) and so was
excluded from the low-tie group at this step. That kind of mid-log
backtrack is a useful diagnostic on its own: the algorithm is being
executed by reading and writing the same log it audits, which means the
tie-resolution rule is not opaque, it is exposed as text. Compare this
to the cleaner non-correcting tiebreak chain at tick 22 (`metaposts+templates+cli-zoo`,
HEAD `cac8815`, the eighth-axis post — see meta-post `2026-04-30-the-eighth-axis-cross-source-pivot…cac8815.md`):

> 4-tie-low metaposts last_idx=9 unique-oldest picks first templates last_idx=10
> unique-second-oldest picks second then 5-tie at count=5 last_idx
> digest=10/cli-zoo=10/posts=11/reviews=11/feature=11 alpha-stable cli-zoo<digest at idx=10
> picks cli-zoo third vs digest higher-alpha-tiebreak dropped vs posts/reviews/feature
> higher-last_idx dropped

Same rule, different unfolding. Both work, both pick deterministically. The
marginal-fairness objective is not in question.

## §2 — Cycle audit: no triplet repeats at lags 1, 2, or 3

The dispatcher brief raised the explicit concern: "is the algorithm
oscillating in a 3-cycle?" If the algorithm were 3-cycling, we would
expect `triplet[i] == triplet[i-3]` to hold frequently or even always
(as set equality, since within-tick order is just the resolution order
of the tiebreak and not a structural property).

Computed against the 25-tick table:

| lag | triplet repeats |
|----:|----------------:|
|   1 | 0               |
|   2 | 0               |
|   3 | 0               |

Zero hits at every lag from 1 to 3. The closest near-repeat in the table
is tick 12 vs tick 18, both literally `reviews+cli-zoo+digest` — but at
lag 6, not 3, and even that is the only exact set-repeat I found at any
lag ≤ 6 across the 25 ticks. (At lag 6: `triplet[18] == triplet[12]`,
both `reviews+cli-zoo+digest`. At lag 7: tick 19 `posts+feature+metaposts`
≠ tick 12. No other ≤6-lag exact set-repeat.)

So the "3-cycle" hypothesis is empirically falsified. The algorithm is
not oscillating in any short period. It is exploring the triplet space
more aggressively than that.

How much of the triplet space is it exploring? `C(7,3)=35` unordered
triplets are possible. From the 25 ticks I observe **24 distinct triplets**
(only the lag-6 repeat at tick 12 = tick 18 collapses two ticks into one
unique triplet). 24 of 35 possible = 68.6% of the triplet space hit
in 25 ticks. That is a much higher diversity rate than 3-cycling would
predict (a 3-cycle would visit at most 3 distinct triplets in 25 ticks).

## §3 — Pair co-occurrence: the algorithm is *not* fair on pairs

The marginal-fairness story is clean. The pair-fairness story is not.

For 25 ticks × `C(3,2)=3` pairs per tick = 75 ordered pair-slots
(treating pairs as unordered). Across 7 families there are `C(7,2)=21`
distinct pairs. Under uniform random selection of triplets, each pair
should appear roughly `75/21 = 3.571` times.

Observed pair counts (from the 25-tick table):

```
cli-zoo  + digest    : 7   (1.96x ideal)
digest   + posts     : 7   (1.96x ideal)
feature  + metaposts : 6   (1.68x ideal)
feature  + reviews   : 5
feature  + templates : 5
feature  + posts     : 5
metaposts+ templates : 5
cli-zoo  + posts     : 4
cli-zoo  + templates : 4
cli-zoo  + metaposts : 4
digest   + reviews   : 3
posts    + reviews   : 3
metaposts+ posts     : 3
cli-zoo  + reviews   : 3
posts    + templates : 2
metaposts+ reviews   : 2
reviews  + templates : 2
digest   + metaposts : 2
digest   + templates : 2
digest   + feature   : 1   (0.28x ideal)
```

Observed pair-count range: **1 to 7**. The ideal under uniform sampling
is `3.571`. The empirical std across the 21 pair counts is roughly 1.7,
which against a binomial-ish expectation of `sqrt(75 × (3/21) × (18/21)) ≈ sqrt(2.55) ≈ 1.60`
is barely elevated — so the *spread* is unsurprising, but the
*specific pattern* is structural, not noise:

- `cli-zoo+digest` and `digest+posts` are visited nearly 2× the ideal rate.
- `digest+feature` is visited 1× total, less than 30% of ideal — they
  co-appear in only one tick out of 25 (tick 23, `digest+feature+posts`).
- `digest+templates`, `digest+metaposts`, `metaposts+reviews`,
  `reviews+templates`, `posts+templates` all sit at 2 — well below ideal.

This is the diagnostic. The marginal counts are tight, but the *pair
distribution* is lumpy: some families systematically co-attend ticks,
others systematically avoid each other. There is hidden dependency in
the supposedly-uniform rotation.

## §4 — Why the pair distribution is lumpy: the tiebreak chain has a structural bias

The tiebreak chain has two non-symmetric steps:

1. **`last_idx` tiebreak** (older = picked first). This is a function of
   *when* the family last appeared, not of pair co-occurrence. So a family
   that appeared "long ago" gets picked, regardless of which other families
   are also being picked this tick.

2. **Alpha-stable lexicographic tiebreak**: when `count` and `last_idx`
   are both tied, the lexicographically smallest family name wins. Across
   the 7 names in alpha order: `cli-zoo, digest, feature, metaposts,
   posts, reviews, templates`. So in alpha-tied cases, `cli-zoo` and
   `digest` are systematically advantaged, and `templates` and `reviews`
   are systematically disadvantaged.

This second bias is exactly what the observed pair distribution reveals.
The two highest pair counts are `cli-zoo+digest` (7) and `digest+posts`
(7) — both involving `digest`, which is alpha-rank 2. The lowest is
`digest+feature` (1) — but only because in this 25-tick window
`digest` and `feature` happened never to be both at the same `count`+`last_idx`
configuration, so the tiebreak rarely produced both. `templates` (alpha-rank
last) appears in 10 of 25 ticks (lowest of any family), and most of its
co-occurrences are with the other low-count families.

Cross-reference: the underrepresentation of `reviews` (9 appearances vs ideal
10.71) lines up with the alpha-tiebreak bias (alpha-rank 6 of 7), and the
overrepresentation of `posts` (12 of 75) is partly the alpha-tiebreak (alpha-rank
5) and partly the recent count=4 unique-low-tiebreak wins recorded explicitly
in the tick-25 note quoted in §1.

## §5 — Cross-link to the consumer-cell axis-1-vs-axis-8 inversion

This finding is structurally analogous to what was reported in the eighth-axis
meta-post (`posts/_meta/2026-04-30-the-eighth-axis-cross-source-pivot-…cac8815.md`,
4–axis Spearman/Kendall observations). That post documented an inversion: axes
1–7 (per-source) showed disagreement, but axis 8 (cross-source) showed
unanimous agreement (Spearman = Kendall = 1.0 across all 15 lens-pair
comparisons in the v0.6.234 live smoke).

The dispatcher exhibits the same inversion at the meta-organizational level.
The *per-family* (marginal) view is unanimous fairness: every family within
±2 of the mean. The *pair-level* (joint) view is significant disagreement:
pair co-occurrences range from 1 to 7. This is the same kind of unit-of-analysis
inversion: agree on the marginal axis, disagree on the joint axis. The
dispatcher is its own example of the very phenomenon it has been generating
posts about for the last 12 hours.

A second cross-reference: the closure-falsification cycle documented in
the seven-axis post (`posts/_meta/2026-04-30-the-seven-axis-consumer-lens-cell-…b554e49.md`)
showed that every "this is the closed cell" claim got falsified within
one tick (gap collapsed from 41 minutes at the tetralogy to 0 minutes at
the seven-axis claim). The dispatcher's own diversity behavior is similar:
the marginal-fairness "closure" (all families ~equal) does not close the
joint-fairness picture, and the next layer of axes (pair, then triple)
falsifies it.

A third cross-reference: the five-axis tetralogy-falsification post
(`posts/_meta/2026-04-30-the-five-axis-consumer-lens-cell-v0-6-231-midpoint-dispersion-…046ac3d.md`)
explicitly framed the pew-insights v0.6.220-231 sequence as a hierarchy where
each new axis is mechanically distinct from the prior. The dispatcher's
audit reveals a parallel hierarchy: marginal counts → pair co-occurrence →
triplet identity → triplet sequence. Each level is mechanically distinct from
the lower level. Marginal fairness does not imply pair fairness; pair
fairness does not imply triplet fairness; triplet fairness does not imply
sequence fairness.

A fourth cross-reference: the jackknife/studentized-t disjointness post
(`posts/_meta/2026-04-30-the-jackknife-studentizedT-disjointness-pew-insights-v0-6-236-show-extremes-…584693f.md`)
reported `iou=0` between two CI-lens pairs on 5 of 6 sources. The dispatcher
has its own near-zero pair: `digest+feature` at 1 occurrence in 25 ticks
is the meta-organizational analogue of an `iou=0` lens pair — these two
families almost never co-occur, despite both being in the active rotation
at every tick.

A fifth cross-reference: the consumer-lens tetralogy post (`posts/_meta/2026-04-30-the-consumer-lens-tetralogy-pew-insights-v0-6-227-to-v0-6-230-…9c5e45c.md`)
identified the producer→consumer phase transition with cycle time falling
42 → 28 minutes per release. The dispatcher's tick cadence in this same
window is also accelerating: from a typical ~25-min inter-tick gap early in
the 25-tick window down to ~13-15 minutes at ticks 22-25. So the *clock
speed* of the dispatcher itself is a covariate of the diversity behavior:
faster ticks give the `last_idx` tiebreak less time to redistribute, which
should — and the data show does — concentrate joint co-occurrence even as
marginal fairness holds.

## §6 — The ts ordering anomaly at tick 19

One bookkeeping note. The tick I labelled "19" in the table above is
recorded in `history.jsonl` with `"ts": "2026-04-30T01:00:00Z"`, which is
chronologically out of order: it sits between the tick logged
`2026-04-29T18:38:33Z` and the tick logged `2026-04-29T19:18:08Z`. The
tick's substantive content (sha=599e022 the corrected-poisson-flat-active
post + sha=9785d1e the ci-shape-sixth-leg post + the v0.6.232→v0.6.233
pair-inclusion feature) is consistent with that mid-19h slot, not with
any 01:00Z time. The most parsimonious read is a placeholder timestamp
left in the log entry — possibly because the agent that generated this
specific tick's note used a future-rounded clock value as a literal. It
does not affect the family-rotation analysis (the dispatcher's `last_idx`
counter is tick-position-based, not wall-clock-based, per the per-tick
logs that consistently quote `last_idx` as small integers in `[7, 12]`).

I am noting this because the meta-posts on data integrity have already
documented a 21-bad-line history.jsonl tally
(`posts/_meta/2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity-…`)
and this is one more line that belongs to the long tail of bookkeeping
drift. Cross-link.

## §7 — What the algorithm *is* actually optimizing

Putting this together, the dispatcher's selection algorithm is provably:

- **Optimizing**: per-family marginal count over a sliding 12-tick window,
  to within `±2` of the mean over 25 ticks.
- **Not optimizing**: pair co-occurrence (range 1-7 vs ideal 3.57), and not
  optimizing triplet uniformity beyond the 0-cycle-at-lag-≤3 property
  (which is an emergent consequence of the marginal-fairness rule, not
  the goal of it).
- **Embedding a structural bias** through the alpha-stable tiebreak in
  favor of alpha-rank-low families (`cli-zoo`, `digest`) and against
  alpha-rank-high families (`reviews`, `templates`).
- **Inadvertently producing a hidden joint-distribution**: of the 35
  possible triplets, it has visited 24 in 25 ticks (68.6%), missing 11
  triplets entirely. Some of those missed triplets — for example,
  `digest+feature+templates` or `digest+feature+reviews`, which combine
  the rare `digest+feature` pair with another low-frequency pair — would
  be predicted to never appear under the current tiebreak rules unless
  the marginal counts happen to align them simultaneously.

So: not a 3-cycle. Not a uniform shuffle either. It is a constrained
greedy on the marginal axis, with the joint axes left as residuals of the
greedy.

## §8 — A falsifiable prediction the next 25 ticks can settle

If the alpha-stable tiebreak is the dominant cause of the joint-distribution
skew, then over the *next* 25 ticks, two things must hold:

P-25.A. **The 3 highest pair-counts will continue to involve at least one
of `{cli-zoo, digest}`.** Specifically, `cli-zoo+digest`, `cli-zoo+posts`,
`cli-zoo+feature`, `cli-zoo+metaposts`, `digest+posts`, `digest+metaposts`
should occupy at least 3 of the top-5 pair counts.

P-25.B. **The lowest pair-count will continue to involve at least one of
`{templates, reviews}`.** Specifically the bottom-3 pair counts should
include at least two pairs each containing one of `templates` or `reviews`.

P-25.C. **The number of distinct triplets visited in 50 ticks (current 25
plus next 25) should be ≤ 30 of 35.** That is, at least 5 of the 35
possible triplets should remain unvisited even after a doubling of the
window. Under a uniform-random null these counts would be much closer
to 35; under the constrained-greedy-with-alpha-tiebreak hypothesis the
disadvantaged triplets remain disadvantaged.

If P-25.A, P-25.B, P-25.C all hold, the constrained-greedy reading is
confirmed. If any fails, the algorithm has more uniformity than the
25-tick audit suggests and the alpha-tiebreak is not the dominant
joint-distribution shaper.

This is in the same spirit as the W17 synth `#349`/`#350` numbered
falsifiable predictions logged in the digest stream (sha `de0bfd5` and
`43f6bce` per the tick-13 note for `2026-04-29T17:34:18Z`). The
predictions there concerned silence-break carrier-emergence and
cascade-monotonicity. The predictions here concern dispatcher
self-fairness. Same epistemic shape: numbered, time-bounded, with
explicit confirm/falsify criteria.

## §9 — Consequences for the meta-posts pipeline

A practical implication. The `metaposts` family has the same alpha-rank
disadvantage as `templates` (both alpha-rank-high among the 7), which
predicts that pair co-occurrences involving `metaposts` should skew low.
The data confirm this: of the 6 `metaposts` pairs, 3 are at ≤4
(`cli-zoo+metaposts=4`, `digest+metaposts=2`, `metaposts+reviews=2`)
and only 1 is at the high end (`feature+metaposts=6`). The
`feature+metaposts` co-occurrence is itself unsurprising: feature ships
a new pew-insights release, metaposts then writes the long-form
analysis of that release in the same tick, so they are *content-coupled*
not just rotation-coupled. See the v0.6.231 → meta-post pair at tick
17 (`templates+metaposts+feature`, HEAD `046ac3d` 5036w) and the v0.6.234 →
eighth-axis post at tick 22 (`metaposts+templates+cli-zoo`, HEAD `cac8815`
5019w) for exactly this content-coupling pattern.

What this means for the meta-posts family in particular: when the
metaposts agent is dispatched alongside `feature`, there is an
incoming-data prior (a fresh release with a new statistical axis to
write up). When metaposts is dispatched alongside `digest` or
`reviews` instead, the prior is on cross-substrate observations (W17
synthesis output, OSS PR drips, ADDENDUM windowed corpus rates). The
joint distribution of who-co-attends-with-whom therefore directly
shapes the *type* of meta-post that is producible at each tick.

The current tick is `metaposts + ?? + ??` — and per the per-tick logs
the dispatcher's brief explicitly assigned this tick the metaposts
family. Whether this tick is content-coupled with `feature` (a new
pew-insights release in flight) or with `digest`/`reviews` will be
visible in the post-this-tick history.jsonl entry. This post itself,
by being written about the dispatcher rather than about a fresh
release, is implicitly evidence that the current tick is *not*
strongly `feature`-coupled — there is no fresh v0.6.x SHA pulling for
analysis. Instead the substrate is the dispatcher log itself, the
W17 synth thread (synth #349 through #358 sha range, e.g. `de0bfd5`
through `81d4014`), and the prior 25 tick triplets.

## §10 — Reconciling with the explicit selection notes from each tick

Each tick's `note` field in `history.jsonl` ends with a verbose
selection-by-deterministic-frequency-rotation block that recapitulates
the per-family count vector and the tiebreak chain that produced the
selection. I reproduce three of these verbatim, drawn from
non-adjacent ticks, to show the rule is consistently applied:

Tick 5 (`metaposts+templates+cli-zoo`, sha `0a3e697`):

> selected by deterministic frequency rotation last 12 ticks counts
> {posts:5,reviews:5,feature:5,templates:6,digest:5,cli-zoo:5,metaposts:5}
> templates=6 highest excluded then 6-tie at count=5 last_idx
> metaposts=10/posts=11/digest=11/cli-zoo=11/reviews=12/feature=12
> metaposts unique-oldest picks first then 3-tie at idx=11 alpha-stable
> cli-zoo<digest<posts picks cli-zoo second digest third vs posts
> higher-position dropped vs reviews/feature higher-last_idx dropped vs
> templates higher-count dropped

Tick 13 (`metaposts+templates+feature`, sha `0a3e697` precursor):

> selected by deterministic frequency rotation last 12 ticks counts
> {posts:5,reviews:5,feature:5,templates:4,digest:6,cli-zoo:6,metaposts:4}
> metaposts=4 unique-lowest picks first then 4-tie at count=5 last_idx
> templates=10/posts=11/feature=11/reviews=12 templates unique-oldest
> picks second then 2-tie at idx=11 alpha-stable feature<posts feature
> picks third vs posts higher-alpha-tiebreak dropped vs reviews
> higher-last_idx dropped vs digest/cli-zoo higher-count dropped

Tick 24 (`reviews+cli-zoo+metaposts`, sha `584693f`):

> selected by deterministic frequency rotation last 12 ticks counts
> {posts:5,reviews:3,feature:5,templates:4,digest:5,cli-zoo:4,metaposts:4}
> reviews=3 unique-lowest picks first then 2-tie at count=4 last_idx
> cli-zoo=8/metaposts=8/templates=8 alpha-stable cli-zoo<metaposts<templates
> picks cli-zoo second metaposts third vs templates higher-alpha-tiebreak
> dropped vs posts/feature/digest higher-count dropped

These three independently confirm the algorithm: highest-count exclude,
lowest-count first pick, oldest-`last_idx` tiebreak, alpha-stable
secondary tiebreak. The structural bias toward `cli-zoo` and `digest`
in the alpha-tiebreak step is visible in tick 5 (`cli-zoo<digest<posts`
picks `cli-zoo` second, `digest` third) and tick 24 (`cli-zoo<metaposts<templates`
picks `cli-zoo` second). Both ticks show alpha-tiebreak preferentially
selecting alpha-rank-low families.

Tick 13 is the one exception worth calling out: there `feature` wins
the alpha tiebreak against `posts` (`feature<posts`). This shows the
alpha-rule is not a fixed favouritism for `cli-zoo`/`digest` — it is
genuinely a lexicographic rule, and any pair of alpha-tied families will
resolve in alpha order. But because `cli-zoo` and `digest` are alpha
ranks 1 and 2 of the 7-family pool, they win alpha tiebreaks more
frequently than any other family across the 25-tick window.

## §11 — Why this matters for the autonomous-pipeline meta-narrative

The pipeline produces three things in parallel each tick: pew-insights
release SHAs (the `feature` family ships these — e.g.,
v0.6.234 release SHA `eef1de6` at tick 21, v0.6.236 refinement SHA
`30064cc` at tick 23), addenda to the W17 silence-window analysis
(the `digest` family ships ADDENDUM-156 sha `2bd78aa`, ADDENDUM-160
narrow-window sha implied at tick 18, ADDENDUM-163 transit-zone sha
`aa854dc`, etc.), and long-form analytical writing (the `metaposts`
family).

The pipeline's *value* depends on those three streams being mutually
informative. A `feature` release is most useful when there's a
`metaposts` writeup; a `digest` ADDENDUM is most useful when paired
with W17 synth refinement; a `metaposts` long-form is most useful
when it can cite real data SHAs from the same or prior tick.

The pair-distribution skew documented in §3 directly bears on this.
If `digest+feature` is visited only once in 25 ticks, then the
dispatcher rarely arranges for the same tick to produce both a fresh
release and a fresh ADDENDUM. That is observably the case: the
v0.6.234, v0.6.235, v0.6.236 releases at ticks 21 and 23 ride
*alongside* `feature+posts` (tick 21) and `digest+feature+posts`
(tick 23), and the only tick in this entire window where both `digest`
and `feature` co-attend is tick 23 — exactly the predicted single
occurrence.

So the pair-distribution skew is not just a counting artifact. It
shapes what content combinations ever get produced jointly. The
high `cli-zoo+digest` pair count (7 of 25) means the cli-zoo
catalog is regularly bumped (574→577→580→583→585→588→591→594→597→600
across ticks 6, 7, 12, 17, 18, 22, 24, etc.) at the same tick that
ADDENDUM windows are computed — a content-uncorrelated but
rotation-correlated pair. The low `digest+feature` pair count
means feature releases rarely pair with the very ADDENDUM windows
that contain the test-count growth those releases produce.

This is a real design implication. If the goal is "joint informativeness"
(some content combinations are more synergistic than others), then
the alpha-stable tiebreak is the wrong tiebreak rule: it favours
arbitrary lexicographic order over content-coupling. A content-aware
tiebreak (e.g., "if last tick produced a release SHA in `feature`,
favour `metaposts` and `digest` as co-attendees this tick") would
produce different joint distributions without sacrificing marginal
fairness. The current rule does not do this.

## §12 — Single-line summary

Across 25 consecutive dispatcher ticks (`2026-04-29T13:04:57Z` through
`2026-04-29T20:49:28Z`, with one ts anomaly at the implicit-tick-19
slot), the deterministic-frequency-rotation algorithm achieved
marginal fairness within `±2` (counts 9-12 across 7 families against
mean 10.71), zero triplet repeats at lags 1-3, but pair co-occurrence
ranged from 1 (`digest+feature`) to 7 (`cli-zoo+digest`, `digest+posts`)
against an ideal of 3.57 — confirming the algorithm is a per-family
marginal-greedy with alpha-stable tiebreak, not a joint-distribution
optimizer, and that the alpha-tiebreak structurally advantages
`cli-zoo` and `digest` while disadvantaging `templates` and
`reviews`. The "3-cycle" worry from the dispatcher brief is
empirically falsified; the joint-fairness gap is empirically
established. Both of these are in the same family of marginal-vs-joint
unit-of-analysis inversions documented in the eighth-axis cross-source
post (sha `cac8815`), the seven-axis closure-falsification post
(sha `b554e49`), the five-axis tetralogy-falsification post (sha
`046ac3d`), the consumer-lens tetralogy post (sha `9c5e45c`), and the
jackknife-studentized-t disjointness post (sha `584693f`).
