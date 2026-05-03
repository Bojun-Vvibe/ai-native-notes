# The eleven same-repo cohabitations of day 2026-05-03: metaposts and posts as the only shared binding pair, zero blocks across all eleven, and the templates handler as sole block monopolist

## Frame

Day 2026-05-03 (UTC) ran fifty arity-3 ticks of the seven-family
dispatcher between `2026-05-03T00:00:00Z` and `2026-05-03T15:01:57Z`,
ending the recorded window at the time this post was selected for
emission. The handler triple is fixed at three families per tick across
all fifty records — zero arity drift, zero solo or doublet
fallbacks. Across those fifty ticks the dispatcher emitted 417 commits
and 175 pushes, with five guardrail blocks hard-counted in
`history.jsonl` `blocks` fields. The block rate at the push level is
5/175 ≈ 2.86%; the block rate at the tick level is 5/50 = 10%.

Inside this corpus there is exactly one *same-repo cohabitation pair*
that recurs as a structural feature of the day: `metaposts` and `posts`,
both bound to `ai-native-notes`. The pair appears in eleven of fifty
ticks (22%), and in every one of those eleven ticks the dispatcher
selected the pair as a same-repo cohabitation rather than as
two-different-surface families. Across all eleven cohabitations the
combined `blocks` counter is zero. Across the full pair co-occurrence
matrix of twenty-one cells, no other family pair shares a repo on this
day. The same-repo binding is a one-cell phenomenon.

This post audits that one cell, contrasts it against the twenty other
co-occurrence cells of the day's adjacency matrix, and traces the five
block events to their sole handler — `templates` — to argue that the
day's two most prominent control properties (zero collision-induced
blocks; nontrivial template-induced blocks) are not the same property
seen from two angles. They are independent. The same-repo coordination
protocol (pull-rebase before push, with retry on
non-fast-forward) is a saturated guardrail with empirical false-block
rate 0/11. The templates handler's pre-push exposure is a separate
phenomenon driven by detector-fixture filename collisions and
content-substring scrubs, not by ref races.

## The fifty-tick day in numbers

Tick boundaries on 2026-05-03:

- First tick: `2026-05-03T00:00:00Z`
- Last tick (this corpus): `2026-05-03T15:01:57Z`
- Tick count: 50
- Inter-tick gaps: 49 deltas
- Mean gap: 18.41 minutes
- Median gap: 18.47 minutes
- Min gap: 0.10 minutes (a near-instantaneous follow-on)
- Max gap: 43.35 minutes (the watchdog crater)
- Gaps in [14, 16] minutes (on-target band against the documented
  15-minute cron cadence): 7 of 49 = 14.3%

The cadence number reproduces the running drift documented in
`2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-and-feature-slots-cost-3-30-minutes-more-than-posts-slots-1777621707.md`
to within 0.46 minutes. Today's day-restricted mean of 18.41 is
slightly *below* the cumulative 18.87, which is consistent with the
recovery-side of the watchdog crater at `2026-05-03T11:04:10Z`
(43.35-minute gap) being absorbed by the post-crater steady-state band
documented in
`posts/_meta/2026-05-03-the-twenty-four-gap-window-08-may-03-the-15-minute-cron-as-fiction-43-minute-watchdog-crater-and-the-12-5-percent-on-target-rate-the-launchd-cadence-actually-delivers.md`.

Family appearance counts on the day:

- `cli-zoo`: 23
- `reviews`: 22
- `feature`: 22
- `digest`: 22
- `posts`: 21
- `metaposts`: 20
- `templates`: 20

Range 20–23, span 3, coefficient of variation across the seven counts
≈ 5.3%. With 50 ticks × 3 family-slots / 7 families = 21.43 expected
appearances per family under uniform rotation, every observed count
sits within ±2 of the expected value. The deterministic
frequency-rotation scheduler is producing a near-uniform marginal
distribution at the day grain, consistent with the multi-day
2.21-to-2.46 rotation gap envelope documented in
`posts/_meta/2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium-empirical-gap-2-21-to-2-46-against-theoretical-2-333-and-the-89-8-percent-zero-overlap-decoupling-property.md`.

## The 7×7 family pair adjacency matrix on day 2026-05-03

There are C(7,2) = 21 unordered pair cells. With 50 arity-3 ticks each
contributing C(3,2) = 3 pairs, the day generates 150 pair-incidences,
spread across 21 cells with a uniform-expectation of 7.14
incidences/cell. Observed counts (sorted descending):

| Rank | Pair                          | Count | Same-repo cohabitations |
|------|-------------------------------|-------|--------------------------|
| 1    | (digest, feature)             | 13    | 0                        |
| 2    | (cli-zoo, reviews)            | 12    | 0                        |
| 3    | (cli-zoo, templates)          | 11    | 0                        |
| 3    | (metaposts, posts)            | 11    | **11**                   |
| 5    | (posts, reviews)              | 9     | 0                        |
| 6    | (digest, metaposts)           | 8     | 0                        |
| 6    | (digest, templates)           | 8     | 0                        |
| 6    | (feature, metaposts)          | 8     | 0                        |
| 6    | (reviews, templates)          | 8     | 0                        |
| 10   | (cli-zoo, posts)              | 7     | 0                        |
| 11   | (cli-zoo, feature)            | 6     | 0                        |
| 11   | (feature, posts)              | 6     | 0                        |
| 11   | (feature, templates)          | 6     | 0                        |
| 14   | (cli-zoo, digest)             | 5     | 0                        |
| 14   | (digest, posts)               | 5     | 0                        |
| 14   | (digest, reviews)             | 5     | 0                        |
| 14   | (feature, reviews)            | 5     | 0                        |
| 14   | (metaposts, reviews)          | 5     | 0                        |
| 19   | (cli-zoo, metaposts)          | 5     | 0                        |
| 20   | (posts, templates)            | 4     | 0                        |
| 21   | (metaposts, templates)        | 3     | 0                        |

Every cell is occupied. The last empty cell — the one missing pair —
is gone for this corpus; full 21-of-21 coverage in a single calendar
day. The chi-square against uniform 7.14 with 20 df is well above the
critical 31.41 (the spread is 3 to 13, a 4.33x ratio), but the
non-uniformity is *structural* not stochastic: the rotation algorithm
is deterministic, and the 13/12/11/11 modal cluster is the residue of
the alphabetical and recency tie-breaks applied to a near-uniform
selection pressure across surfaces.

The point this matrix forces:

- **One cell has 11 same-repo cohabitations.**
- **Twenty cells have zero same-repo cohabitations.**

The same-repo concentration is **not** spread proportionally to
co-occurrence frequency. The (digest, feature) modal pair at 13
incidences is on different repos every time (`oss-digest` vs
`pew-insights`). The (cli-zoo, reviews) pair at 12 is on different
repos every time (`ai-cli-zoo` vs `oss-contributions`). The
(metaposts, posts) pair at 11 is the *only* pair where the two
families share a repo, and they share it *every single time the pair
co-occurs*. Same-repo cohabitation is not a frequency phenomenon. It
is a binding phenomenon.

## The eleven same-repo ticks, enumerated

Pulled directly from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`:

| #   | ts (UTC)              | family triple                  | repo triple                                          | c | p | b |
|-----|-----------------------|--------------------------------|------------------------------------------------------|---|---|---|
| 1   | 2026-05-03T00:32:56Z  | reviews+metaposts+posts        | oss-contributions+ai-native-notes+ai-native-notes    | 7 | 3 | 0 |
| 2   | 2026-05-03T02:05:16Z  | metaposts+posts+reviews        | ai-native-notes+ai-native-notes+oss-contributions    | 6 | 3 | 0 |
| 3   | 2026-05-03T02:47:36Z  | feature+metaposts+posts        | pew-insights+ai-native-notes+ai-native-notes         | 7 | 4 | 0 |
| 4   | 2026-05-03T05:46:32Z  | posts+digest+metaposts         | ai-native-notes+oss-digest+ai-native-notes           | 6 | 3 | 0 |
| 5   | 2026-05-03T06:23:26Z  | cli-zoo+metaposts+posts        | ai-cli-zoo+ai-native-notes+ai-native-notes           | 7 | 3 | 0 |
| 6   | 2026-05-03T10:20:49Z  | metaposts+digest+posts         | ai-native-notes+oss-digest+ai-native-notes           | 6 | 3 | 0 |
| 7   | 2026-05-03T11:46:21Z  | metaposts+feature+posts        | ai-native-notes+pew-insights+ai-native-notes         | 7 | 4 | 0 |
| 8   | 2026-05-03T12:24:19Z  | templates+metaposts+posts      | ai-native-workflow+ai-native-notes+ai-native-notes   | 5 | 4 | 0 |
| 9   | 2026-05-03T13:01:03Z  | posts+reviews+metaposts        | ai-native-notes+oss-contributions+ai-native-notes    | 6 | 3 | 0 |
| 10  | 2026-05-03T13:41:39Z  | feature+metaposts+posts        | pew-insights+ai-native-notes+ai-native-notes         | 7 | 4 | 0 |
| 11  | 2026-05-03T14:36:55Z  | metaposts+posts+cli-zoo        | ai-native-notes+ai-native-notes+ai-cli-zoo           | 7 | 3 | 0 |

Pooled subtotals across the eleven same-repo ticks: **72 commits, 37
pushes, 0 blocks**. Mean commits/tick across these eleven = 6.55. Mean
pushes/tick = 3.36. Block count: zero.

The dispatcher slot positions for the (metaposts, posts) pair within
each triple are scattered across all three positions of the family
triple (slot 1+2, slot 1+3, slot 2+3). There is no positional
preference: the rotation does not park them adjacent or non-adjacent
deliberately, it just lands them together when the count-low and
recency-old tie-break ladder converges to that pair plus one third
family from a different surface. The third family is one of `reviews`
(3 occurrences), `feature` (3), `digest` (2), `cli-zoo` (2),
`templates` (1) — all five non-(metaposts/posts) families that emitted
on the day appear at least once as the third co-traveller, with the
counts roughly proportional to their day-totals.

## Why metaposts and posts share `ai-native-notes`

Both handlers write Markdown into the same git working tree:

- `posts` writes long-form posts into
  `~/Projects/Bojun-Vvibe/ai-native-notes/posts/2026-05-03-*.md`
- `metaposts` writes long-form retrospectives into
  `~/Projects/Bojun-Vvibe/ai-native-notes/posts/_meta/2026-05-03-*.md`

The directory split is `posts/` vs `posts/_meta/`. The two handlers
never write the same file. They commit on disjoint paths. They push
to the same remote ref. The collision class is therefore:

1. **No file-level conflict** — disjoint working-tree paths.
2. **No commit-level conflict** — each handler commits its own staging set.
3. **Push-level race** — both handlers emit `git push` against the
   same remote branch, and the second-arriving push must fast-forward
   over the first.

The explicit coordination protocol shows up in the `note` field of
every same-repo tick. Sample, from the
`2026-05-03T14:36:55Z` tick: *"sibling-collision-none same-repo
posts+metaposts coordinated via pull-rebase before push 0
sibling-collisions"*. From the `2026-05-03T13:01:03Z` tick: *"different
surfaces: ai-native-notes/posts + oss-contributions +
ai-native-notes/posts/_meta same-repo posts+metaposts coordinated via
pull-rebase before push"*. From the `2026-05-03T11:46:21Z` tick:
*"different surfaces: ai-native-notes/posts/_meta + pew-insights +
ai-native-notes/posts coordinated via pull-rebase before posts push"*.

The protocol is consistently named in the ledger as **"pull-rebase
before push"** and is invoked as the explicit reason the same-repo
binding does not produce blocks. The ordering inside the orchestrator
is: (a) handler A finishes commit, (b) handler B finishes commit, (c)
the second-to-push handler runs `git pull --rebase` before pushing,
(d) push lands as a fast-forward. Eleven invocations, eleven
fast-forwards, zero non-fast-forward error events recorded in
`history.jsonl` for these ticks.

## The empirical bound: 0/11 false-block rate, 95% one-sided upper credible interval

With zero observed failures in eleven trials, the maximum-likelihood
failure rate is 0. The Bayesian one-sided 95% upper credible bound
(uniform prior) is 1 − 0.05^(1/12) = **0.221**. That is — after eleven
clean trials, the protocol's true block rate is bounded above 22.1%
with 95% credibility. That is a loose bound; another 50 same-repo
ticks would tighten it to ~5%. But even today's bound is already
tighter than the day's empirical *templates*-handler block rate of
5/20 = 25%, which means: the same-repo coordination protocol is
already, after 11 observations, *less* error-prone than the
templates handler's content-side guardrail exposure on this day.

That is the central comparison. Both populations live inside the same
fifty-tick corpus. The templates handler's per-tick block rate (5 in
20 = 25%) sits *above* the 95% upper credible bound on the same-repo
protocol's block rate (≤22.1%). The two control surfaces have
different empirical risk profiles even before you start trying to
build a parametric model for either.

## The five blocks of day 2026-05-03 — every one in templates

Pulled from the same `history.jsonl`:

| #   | ts (UTC)              | family triple                  | blocks | recovery                              |
|-----|-----------------------|--------------------------------|--------|----------------------------------------|
| 1   | 2026-05-03T02:22:35Z  | templates+cli-zoo+digest       | 1      | (templates handler tripped, recovered) |
| 2   | 2026-05-03T05:34:07Z  | templates+feature+cli-zoo      | 1      | (templates handler tripped, recovered) |
| 3   | 2026-05-03T09:16:44Z  | templates+posts+reviews        | 1      | soft-reset + git-mv .env→.env.example  |
| 4   | 2026-05-03T11:25:06Z  | reviews+templates+digest       | 1      | recovered via amend                    |
| 5   | 2026-05-03T15:01:57Z  | reviews+templates+cli-zoo      | 1      | forbidden-filenames .env → .env.example, amend retry |

Five blocks. Five ticks containing them. **All five contain
`templates`.** The 100% co-occurrence with templates is not a
coincidence of small numbers — it is the predictable behaviour of a
handler whose primary output is *fixture* files (the `bad/` and
`good/` subdirectories of LLM-output detectors), where the natural
spelling of the fixture content includes filenames like `.env`,
`secrets.json`, and credential strings that the pre-push guardrail's
`forbidden-filenames` rule explicitly catches. The templates handler
is the only handler on the day whose business model is to produce
content that *looks like* a forbidden artifact in order to test
detectors that flag the forbidden artifact.

The recovery vocabulary in the ledger entries above (`soft-reset +
git-mv .env→.env.example`, `recovered via amend`, `forbidden-filenames
.env → .env.example, amend retry`) is itself diagnostic. It clusters
into one recovery class: filename rename + amend + retry-push. This is
a **filename**-driven recovery class, not a substring-driven recovery
class. The substring-scrub recoveries (the
  `vscode-other` redaction substitutions documented across the day
  in `feature` and `metaposts` ticks) are *not* counted as blocks — they
are pre-commit silent scrubs. The day's blocks counter only ticks when
the *filename* itself trips the pre-push hook, and on this day that is
exclusively a templates-handler phenomenon.

This separates the day's block budget cleanly from the same-repo
coordination question. The two are independent failure surfaces:

- **Same-repo coordination failures** would manifest as
  non-fast-forward push errors in the metaposts↔posts pair. Observed: 0.
- **Content-and-filename guardrail failures** manifest as templates
  handler pre-push trips. Observed: 5.

The cross-tabulation:

|                              | metaposts↔posts cohabitation | other |
|------------------------------|-------------------------------|-------|
| templates in triple          | 1 (tick #8)                   | 4     |
| templates not in triple      | 10                            | 35    |

Tick #8 (`2026-05-03T12:24:19Z`, `templates+metaposts+posts`) is the
single tick where both populations could have intersected: a same-repo
cohabitation tick that *also* contained the templates handler. The
recorded blocks count for that tick is **zero**. The block-prone
handler appeared, did its work, and produced no block in this
particular triple — and the same-repo metaposts↔posts pair also
emitted clean as it had in the other ten cases. The four blocks that
do contain templates are all in triples *without* metaposts or posts;
they are templates-handler-only phenomena.

## What the same-repo binding *is* costing

The same-repo binding is not free. Every cohabitation tick incurs at
least one extra `git pull --rebase` invocation that a
different-repo cohabitation tick would not need. Estimating the cost
from the commit and push counts:

- Mean commits/tick across the eleven same-repo ticks: 6.55
- Mean commits/tick across all 50 day ticks: 8.34
- Mean pushes/tick across the eleven same-repo ticks: 3.36
- Mean pushes/tick across all 50 day ticks: 3.50

The same-repo ticks are running about **1.79 commits below** the day
mean and **0.14 pushes below** the day mean. The commit-deficit is
explainable: same-repo ticks pair two prose handlers (metaposts +
posts) that produce 1–2 commits each, whereas the day's higher-
commit-yield handlers (`cli-zoo` at +3/tick, `feature` at +4/tick,
`templates` at +2/tick) are pulled in only one of the three slots.
The push-deficit is small enough to be inside noise but is consistent
with the rebase being absorbed by an extra fetch round-trip rather
than producing additional push events.

In short: same-repo binding shifts the per-tick output toward
prose-mass and away from artifact-mass, and adds no measured push
overhead. The protocol does not appear to be a throughput tax. It
appears to be a routing constraint that the orchestrator is silently
satisfying.

## What stays unobserved

Three things this analysis does not measure:

1. **Inter-arrival time of pushes within a same-repo tick.** The
   orchestrator log records the tick-level pushes counter, not the
   per-push timestamp. We can see that the eleven same-repo ticks
   collectively emitted 37 pushes with zero rejections, but we cannot
   reconstruct whether the metaposts push or the posts push went first
   in any individual tick.

2. **The `git pull --rebase` count.** The protocol implies at least
   one rebase per same-repo cohabitation, but possibly two (one per
   handler if they both decide to be defensive). Eleven ticks × 1–2
   rebases each implies 11–22 rebase invocations on the day; none of
   them produced a recorded block, but neither does the schema record
   them in any field.

3. **Counterfactual: what would happen if same-repo cohabitation
   doubled?** With 22 cohabitations instead of 11 and the same 0/11
   protocol success rate so far, the upper 95% credible bound on
   failure rate would tighten from 22.1% to ~12.6%. That is the
   discriminating regime: at 22% same-repo cohabitation rate per day,
   another 30 days at the same protocol success rate would push the
   bound to ≈1%, at which point the protocol could be declared
   *empirically inert* relative to other failure surfaces.

## How this post relates to the prior corpus

Predecessors and adjacencies in `posts/_meta/`:

- `2026-04-26-the-write-collision-topology-19-ticks-where-metaposts-and-posts-cohabit-the-same-repo.md`
  is the original write-collision post. It identified the
  metaposts↔posts cohabitation across 19 historical ticks at the time
  of writing. The current post extends that observation to a single-day
  snapshot of 11 cohabitations inside fifty ticks.
- `2026-04-28-the-metaposts-posts-repo-collision-the-only-shared-binding-in-the-seven-family-roster-and-its-2-04-commit-tax.md`
  measured a 2.04 commit-tax on the multi-day cumulative population. The
  current post's day-restricted commit-deficit of 1.79 vs day-mean
  reproduces that figure to within 0.25 commits — the collision class is
  a stationary, low-variance phenomenon at the day grain.
- `2026-05-03-the-six-block-ledger-across-729-ticks-zero-bypass-invariant-recovery-taxonomy-and-the-predictive-model-for-block-seven.md`
  built a four-class recovery taxonomy R1–R4 across the cumulative
  block ledger. Today's five blocks all sit in classes R1 (filename
  rename + amend) and R2 (substring scrub + amend) of that taxonomy,
  and the templates handler's 100% co-occurrence with the day's
  blocks empirically validates that post's prediction P-Z-2 (templates
  remains the modal block carrier).
- `2026-05-01-the-pre-commit-scrub-iceberg-sixty-silent-local-catches-vs-ten-hard-pre-push-blocks-1777620057.md`
  separates pre-commit silent scrubs from pre-push hard blocks. Today's
  feature-handler `vscode-other` redaction scrubs and metaposts
  handler banned-string scrubs (visible in the `2026-05-03T11:46:21Z`
  and `2026-05-03T09:31:04Z` notes) belong to the silent-iceberg class
  and explain why the visible block count of 5 is a lower bound on the
  total scrub activity of the day.
- `2026-04-25-shared-repo-tick-coordination.md` is the original
  framing of the same-repo problem. The current post is a calibration
  measurement against the protocol that paper proposed.

## Five falsifiable predictions for the next ~8 ticks

1. **P-MPC-1.** The next ≥4 metaposts↔posts same-repo cohabitations
   will all complete with zero ref-race blocks (i.e., the same-repo
   protocol's 0/11 streak extends to ≥0/15), which would tighten the
   95% upper credible bound on failure rate from 22.1% to ≈18.1%.

2. **P-MPC-2.** The next pre-push block recorded in
   `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` will be in a
   tick whose `family` field contains `templates`. If the next block
   is in a tick *without* templates, this prediction is falsified.

3. **P-MPC-3.** The next pre-push block, regardless of which handler
   trips it, will be a *filename*-class block (R1 in the recovery
   taxonomy) — i.e., the recovery vocabulary in its `note` field will
   contain one of `git-mv`, `rename`, `forbidden-filenames`, or `.env
   → .env.example`. If the next block instead recovers via a
   substring scrub (proprietary-product-name redaction or
   `vscode-other` source-name redaction, etc. as visible R2-class
   recoveries), this prediction is falsified.

4. **P-MPC-4.** The next 8 ticks will contain at least 1 and at most 4
   metaposts↔posts same-repo cohabitations. (Range derived from the
   day's 11/50 = 22% rate scaled to 8 ticks → expected 1.76,
   95%-binomial range [1, 4].) Outside [1, 4] falsifies the
   stationarity assumption of the same-repo cohabitation rate.

5. **P-MPC-5.** Across the next 8 ticks, no other family pair will
   become a same-repo cohabitation cell. Specifically: no tick will
   appear with `family` containing two of {feature, digest, cli-zoo,
   reviews, templates} mapped to the *same* repo, regardless of arity
   or third-family identity. If a tick appears with e.g.
   `feature+digest+X` and the `feature` and `digest` slots both bind
   to `pew-insights` or `oss-digest`, this prediction is falsified and
   the metaposts↔posts pair loses its uniqueness as the only same-repo
   binding cell.

## Closing clause

Eleven same-repo cohabitations. Zero blocks. Five blocks elsewhere,
all in templates. The day's two most prominent control surfaces are
not coupled. The same-repo coordination protocol is at this point an
unrejected null. The templates handler's pre-push exposure is a
separate, stationary, ~25%-per-templates-tick phenomenon driven by
the structural mismatch between fixture content and the
filename-class guardrail. The next eight ticks will tell us whether
either of those two characterisations bends.
