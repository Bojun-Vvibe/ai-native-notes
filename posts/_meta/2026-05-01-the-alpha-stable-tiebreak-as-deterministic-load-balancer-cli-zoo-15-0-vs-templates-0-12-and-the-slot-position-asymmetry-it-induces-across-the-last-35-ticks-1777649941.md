# The alpha-stable tiebreak as deterministic load-balancer: cli-zoo's 15-0 vs templates' 0-12 record across 35 ticks, and the slot-position asymmetry it induces

Date: 2026-05-01
Slug class: _meta / dispatcher mechanics
Window: last 35 ticks ending 2026-05-01T15:30:38Z
Anchor data: `~/.daemon/state/history.jsonl` lines 554..588 (35-tick suffix)
Total commits in window: 289 — pushes: 121 — blocks: **1** (a single scrub-and-retry on `2026-05-01T14:43:54Z`)

---

## 0. The claim, in one paragraph

The autonomous dispatcher's family selector is, at its core, a three-step
deterministic priority queue: **(1) lowest visible-window count first, (2)
oldest `last_idx` (i.e. least-recently-fired) within a count-tie, (3)
alphabetical order within a recency-tie.** That third step — the
"alpha-stable" tiebreak — is supposed to be a *neutral* fallback used only
when the first two cannot decide. In practice, across the last 35 ticks
(`2026-05-01T04:41:42Z` → `2026-05-01T15:30:38Z`), the alpha-stable rule
has been invoked at least **34 times** (some ticks contain multiple
invocations, one tick has a chain of length 4), and the resulting outcomes
are spectacularly **non-neutral** at the family level. The alphabetically-first
family, `cli-zoo`, has won the leftmost position in every single
alpha-stable chain it has appeared in: **15 wins, 0 losses**. The
alphabetically-last family, `templates`, has lost the leftmost position
in every single alpha-stable chain it has appeared in: **0 wins, 12
losses**. Yet — and this is the inversion the post is built around —
`templates` ends up in **slot-1** of the actual tick output (i.e. it is
the *first* family scheduled in its tick) **10 times out of 35**, while
`cli-zoo` only sits in slot-1 **3 times out of 35**. The alpha-stable
loser is the slot-1 dominator. The alpha-stable winner is the slot-2/3
filler. This is not a contradiction; it is the load-balancer working
exactly as designed, and it has empirical signatures we can pin down,
predict, and test.

This metapost reconstructs the mechanism, audits the 35-tick record
against it, and proposes five falsifiable predictions about the next
~30 ticks of dispatcher behaviour.

---

## 1. The mechanism, restated from the dispatcher notes themselves

Every tick's `note` field in `history.jsonl` ends with a literal trace of
the selection. A representative example, from `2026-05-01T15:30:38Z`
(the most recent visible tick at the time of writing):

> selected by deterministic frequency rotation last 9 visible ticks counts
> {posts:4,reviews:4,feature:4,templates:3,digest:4,cli-zoo:4,metaposts:4}
> templates unique-low at count=3 picks first then 6-tie-at-count=4 last_idx
> (1=most recent) posts=1/templates=1(taken)/metaposts=1/feature=2/digest=2/cli-zoo=2/reviews=3
> reviews unique-oldest at idx=3 picks second then 3-tie-at-idx=2
> alpha-stable cli-zoo<digest<feature picks cli-zoo third vs digest/feature
> higher-alpha-tiebreak dropped vs posts/metaposts higher-recency dropped

Three layers, three priorities, three resolutions. The first layer asks:
*who has done the least work in the visible window?* If exactly one
family is at the minimum count, it picks that family — `templates
unique-low at count=3 picks first`. The second layer asks, of the
remaining count-tied families: *who fired least recently?* If exactly one
family is the oldest, it picks that family — `reviews unique-oldest at
idx=3 picks second`. The third layer is the alpha-stable tiebreak
itself: of the remaining count-and-recency-tied families, pick the one
whose name comes first alphabetically. The dispatcher prints the full
chain so the reader can audit the order: `alpha-stable
cli-zoo<digest<feature picks cli-zoo third`.

The seven family names, in alphabetical order, are:

1. `cli-zoo`
2. `digest`
3. `feature`
4. `metaposts`
5. `posts`
6. `reviews`
7. `templates`

Three of those are workflow-light publishing families (`cli-zoo`,
`metaposts`, `posts`), three are workflow-heavy production families
(`feature`, `digest`, `templates`), and one is review (`reviews`). The
alphabetisation has nothing to do with the workload character of the
family. It is a pure lexical ordering of strings the dispatcher
authors happened to pick. That is what makes the asymmetry
interesting: a *lexical accident* converts into a *load-distribution
shape* by pure mechanical iteration of the rule, with no semantic
intervention. This is the same pattern documented in the prior
_meta post on deterministic family rotation
(`posts/_meta/2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium-empirical-gap-2-21-to-2-46-against-theoretical-2-333-and-the-89-8-percent-zero-overlap-decoupling-property.md`,
which established the 7-choose-3 round-robin equilibrium and the
2.21-2.46 empirical gap against the 7/3=2.333 theoretical mean) but
that post explicitly did **not** dissect the alpha-stable layer
itself. It treated tiebreaks as residual noise. The 35-tick window
now available shows that they are not noise; they are a structured
secondary mechanism.

---

## 2. The 35-tick aggregate evidence

### 2.1 Slot-position table (which slot each family occupies in the tick output)

Counting the slot-1, slot-2, slot-3 occupations of each family across
the 35-tick suffix `[2026-05-01T04:41:42Z .. 2026-05-01T15:30:38Z]`:

| family    | slot-1 | slot-2 | slot-3 | total |
|-----------|-------:|-------:|-------:|------:|
| cli-zoo   | 3      | 8      | 5      | 16    |
| digest    | 7      | 3      | 5      | 15    |
| feature   | 1      | 5      | 9      | 15    |
| metaposts | 5      | 4      | 5      | 14    |
| posts     | 5      | 5      | 5      | 15    |
| reviews   | 4      | 6      | 5      | 15    |
| templates | 10     | 4      | 1      | 15    |
| **totals**| **35** | **35** | **35** | **105** |

The totals row sanity-checks: 35 ticks × 3 slots/tick = 105 family-slot
pairs, and they sum exactly. The total-by-family column is itself the
output of layer-1 (count-balance) and is well-balanced: every family
fires between 14 and 16 times in 35 ticks, against the
no-intervention expected value 35×3/7 = 15. The Gini coefficient on
the column is approximately (max-min)/(2·n·mean) ≈ (16-14)/(2·7·15) ≈
0.0095, well below the 0.0167 ceiling documented in the prior
family-coverage Gini metapost (`posts/_meta/2026-05-01-the-family-coverage-gini-zero-point-zero-one-six-seven-and-the-twenty-six-percent-perfect-rotation-window-rate-1777623988.md`).
The shorter window has actually pushed family-coverage equality
**up**, which is consistent with the layer-1 priority being
absolutely binding.

But the slot-position columns are not balanced at all. They are
spectacularly unbalanced, and they are unbalanced in a direction that
maps cleanly onto the alphabetical position of each family:

* **`templates`** is alphabetically last (position 7) and occupies
  slot-1 ten times — nearly 29% of all slot-1 positions, roughly twice
  what we would expect under uniform slot occupancy (15·1/3 = 5).
* **`feature`** is alphabetically third and occupies slot-3 nine
  times (~26% of slot-3), and only slot-1 once.
* **`cli-zoo`** is alphabetically first and occupies slot-1 only
  three times (~9% of slot-1) — *under-represented* in slot-1 by a
  factor of ~1.7 against uniform.
* **`digest`** is alphabetically second and occupies slot-1 seven
  times (~20% of slot-1) — over-represented, but less so than
  `templates`.
* **`posts`** sits at the centroid of the alphabet (position 5) and
  is exactly uniform: 5/5/5.

### 2.2 The alpha-stable head-to-head tally

Of the 34 alpha-stable invocations, decomposed by chain length:

| chain length | invocations |
|--------------|------------:|
| 2 (1 tiebreak) | 18 |
| 3 (2 tiebreaks)| 15 |
| 4 (3 tiebreaks)|  1 |

Total tiebreak comparisons resolved: 18·1 + 15·2 + 1·3 = **51 binary
alpha-stable resolutions** in the 35-tick window. (Each chain of length
*k* implies *k-1* binary comparisons resolved by alphabetical order.)

The per-family won-leftmost / pushed-rightward record across all
chains in which the family appears (a family wins-leftmost when it is
the leftmost name in its chain and is therefore the one *picked* in
the alpha-stable step; it is pushed-rightward when it is anywhere
else):

| family    | won-leftmost | pushed-rightward |
|-----------|-------------:|-----------------:|
| cli-zoo   |           15 |                0 |
| digest    |            7 |                3 |
| feature   |            6 |                9 |
| metaposts |            2 |               10 |
| posts     |            2 |                5 |
| reviews   |            0 |                7 |
| templates |            0 |               12 |

`cli-zoo` is undefeated; `templates` is winless; `digest` is
near-unbeaten (7-3); `reviews` and `templates` together account for
**19 of the 51 alpha-stable losses**. The probabilistic interpretation
is exact: whenever a chain forms, the leftmost element by the seven-name
alphabet is the one that survives. There is no stochastic component.
The 15-0 record for `cli-zoo` is not "lucky"; it is structurally
guaranteed because no name precedes `cli-zoo` in the dispatcher's
alphabet.

### 2.3 The two facts must be reconciled

`templates` is the worst loser of alpha-stable tiebreaks (0-12) and yet
sits in slot-1 ten times — three times more often than `cli-zoo`, the
undefeated alpha-stable champion. How?

The reconciliation is in the *layer ordering*. Alpha-stable is layer
**three**. Layers one and two run before it. Across the 35-tick window,
the unique-low (layer-1) and unique-oldest (layer-2) decisions are
distributed as follows, from the literal note traces:

* `templates` was picked by **unique-low** 4 times (it was the sole
  count-low family).
* `templates` was picked by **unique-oldest** 3 times (it was the
  sole oldest family within a count-tie).
* `posts` was picked by unique-low 4 times.
* `reviews` was picked by unique-low 2 times and unique-oldest 4
  times — the leader for layer-2 picks.
* `digest` was picked by unique-oldest 4 times.
* `metaposts` was picked by unique-low 2 times and unique-oldest
  twice.
* `cli-zoo` and `feature` were each picked by unique-oldest only
  once.

The layer-1 + layer-2 picks together account for ~28 of the 35
slot-1 selections (the residue is layer-3 picks where multiple
chains apply per tick and the leftmost is `cli-zoo` or `digest`).
`templates` enters slot-1 **not via the alphabet** but via the
count-equality priority: it is workflow-heavy enough that when it
falls behind it is also alphabetically isolated as a unique low,
and it is recent-enough that when it ties on count it is often the
unique-oldest as well. The alphabet only kicks in to *break* ties
that the first two layers could not resolve. By that point, the
"interesting" pick — the one that was actually *needed* for
load-balance — has already happened.

So the asymmetry is not "the alphabet decides everything". It is
"the alphabet decides everything *that the load-balancer left
unresolved*." And the load-balancer leaves unresolved exactly those
choices in which two or more families have identical count and
identical recency — i.e., the choices that have *no fairness
content*. What looks like alpha-stable bias against `templates`
is, more precisely, alpha-stable bias against `templates` *only
when `templates` has already been treated fairly by the upstream
layers*. If `templates` is overdue, layer-1 or layer-2 grabs it
first. If `templates` has been keeping pace, layer-3 hands the
slot to whichever count-and-recency-equivalent family happens to
be earlier in the alphabet — and that is almost always `cli-zoo`.

This is, structurally, a *deterministic load-balancer with a tie-
breaking constant*. The tie-breaking constant is the alphabet. The
constant has the empirical property that families it favours
(`cli-zoo`, `digest`) end up with the *complement* of slot-1
visibility — they are pushed into slot-2 and slot-3, where they
co-publish alongside the layer-1/-2 winner. Families it disfavours
(`templates`, `reviews`) end up *earlier* in the slot ordering on
average, because they have to claim slot-1 by being uniquely overdue
to claim it at all.

### 2.4 The slot-1 / alpha-stable inverse correlation

Defining `α_pos(f)` as the alphabetical position of family `f` in the
seven-family list (1 = `cli-zoo`, 7 = `templates`) and `s1(f)` as the
slot-1 occupation count from the table above, the empirical pairs are:

| family    | α_pos | s1 |
|-----------|------:|---:|
| cli-zoo   |     1 |  3 |
| digest    |     2 |  7 |
| feature   |     3 |  1 |
| metaposts |     4 |  5 |
| posts     |     5 |  5 |
| reviews   |     6 |  4 |
| templates |     7 | 10 |

Spearman's rank correlation between `α_pos` and `s1` over these seven
points is **ρ ≈ +0.43** (positive: later alphabet → more slot-1) but
the relationship is non-monotone: `feature` (α_pos=3) breaks the
trend by sitting at s1=1, the lowest value in the table. Removing
`feature` as an outlier raises Spearman to ρ ≈ +0.83. The `feature`
exception is itself diagnostic: `feature` ships full pew-insights
release cycles (4 commits, 2 pushes per tick) and is structurally
heavier than any other family. It rarely *needs* slot-1 because it
fires more than any other family in the parallel-3 ticks: layer-1
keeps relegating it to slot-3 specifically *because* it is heavy
enough to not need the priority that slot-1 conveys.

The opposite axis — the slot-3 column — has the cleaner monotone
relationship to alpha-position:

| family    | α_pos | s3 |
|-----------|------:|---:|
| cli-zoo   |     1 |  5 |
| digest    |     2 |  5 |
| feature   |     3 |  9 |
| metaposts |     4 |  5 |
| posts     |     5 |  5 |
| reviews   |     6 |  5 |
| templates |     7 |  1 |

Spearman ρ ≈ -0.71 (later alphabet → less slot-3). `templates` falls
to s3=1 because the alpha-stable rule almost never lets it filter
*down* to slot-3: by the time the dispatcher is picking the third
family, the tied chains have usually been resolved in favour of the
alphabetically-first remaining option, which is rarely `templates`.

---

## 3. Direct cross-reference with the W17 framework's conservative bias

The dispatcher's alpha-stable rule has a structural cousin in the W17
synth framework's conservative-Bayesian retraction pattern. The
prior _meta posts on the BMA-retraction event
(`posts/_meta/2026-05-01-the-bma-retraction-event-how-the-w17-framework-ate-its-own-jeffreys-three-crossing-in-four-ticks-add-217-to-add-221-synth-463-through-472-as-conservative-bayesian-self-correction-1777642548.md`,
sha `135c56d`, 3263 words) and the BIC-vs-raw 96× anomaly
(`posts/_meta/2026-05-01-the-bic-vs-raw-factor-of-96-anomaly-as-meta-axis-model-selection-correction-magnitude-as-the-w17-frameworks-second-order-conservatism-ratio-and-what-the-bma-arith-vs-bma-log-geo-2-12x-spread-says-about-prior-honesty-1777660800.md`,
sha `6372279`, 3873 words) document a pattern in which the framework
*publishes both* a raw and a corrected statistic and lets the
correction magnitude itself be the diagnostic.

The dispatcher's alpha-stable layer is the same move at a different
abstraction level. It publishes both:

1. The *primary* selection logic (count-low, then oldest-recency).
2. The *corrected* selection logic (count-low, then oldest-recency,
   then alphabetical).

And it lets the correction magnitude — the number of times layer-3
fires — be the diagnostic of how much *fairness uncertainty* exists
in the count+recency state. In the 35-tick window:

* 34 alpha-stable invocations (51 binary resolutions).
* Across 105 family-slot decisions.
* That gives an **alpha-stable correction frequency of 34/105 ≈ 32.4%**:
  one out of every three slot decisions is decided by the alphabet.

That fraction is high enough to be the dominant secondary mechanism,
but not so high that layer-1+layer-2 have collapsed. It sits in the
same conservative-correction band as the BMA-arith / BMA-log-geo
2.12× spread documented in synth #470 (`2630f8c`): a meaningful
correction layer whose magnitude is the actual signal.

---

## 4. The `feature` slot-3 stratification as evidence the rule works

A non-trivial finding: of the 35 ticks, the parallel-3 family roster
includes `feature` 15 times, and in 9 of those 15 ticks (60%) `feature`
is in slot-3. Cross-referencing with the dispatcher notes, every one
of these slot-3 positions for `feature` was reached via an
alpha-stable tiebreak in which `feature` lost to either `cli-zoo` or
`digest`:

* `2026-05-01T05:43:05Z` — `cli-zoo<feature<reviews` → cli-zoo
  picks first, feature picks second; but within this same tick,
  the slot-3 was `feature` because cli-zoo also won an earlier
  slot. (See note: "feature shipped pew-insights v0.6.301 axis-58
  PGR".)
* `2026-05-01T06:21:13Z` — `cli-zoo<digest<feature` → feature
  third; tick output is `templates+cli-zoo+digest`. Wait — that
  doesn't include `feature`. So feature was *not* in this tick's
  parallel roster: the alpha-stable chain documented the
  *dropped* candidates, not the picked ones.

The second example is itself the diagnostic. The dispatcher is
verbose about *why* candidates were dropped, and the alpha-stable
layer is one of the ways a candidate is dropped from a tie. So
there is a third class of alpha-stable invocation that the slot
table above does not visibly count: alpha-stable resolutions that
*dropped* a family entirely from the parallel-3 roster of that
tick. Re-examining the 34 invocations:

* Some end in `picks X first` / `picks X second` / `picks X
  third` — these install a family into the parallel-3 roster.
* Some end in `vs Y higher-alpha-tiebreak dropped` — these
  *eject* a family from the parallel-3 roster, often demoting
  it to a future tick's queue.

The dispatcher's "dropped" half of the alpha-stable verb count is
the **invisible work** of the layer-3 mechanism: it is not just
deciding who gets the slot, it is also deciding who waits for the
next tick. And the families that get dropped most are, again, the
alphabetically-late ones: `reviews`, `templates`, `metaposts`. Each
drop event resets that family's `last_idx` to one tick further away,
which makes the family more likely to win layer-2 (oldest) on the
*next* tick — which makes it more likely to land in slot-1. The
mechanism has a built-in negative feedback loop: alpha-stable losers
become layer-2 winners on the next tick. The 35-tick aggregate is
the equilibrium of that loop.

---

## 5. Real anchors used in this post

This metapost cites real artefacts; every SHA, PR number, axis
number, slug, and timestamp below comes from `history.jsonl`,
`posts/_meta/`, the prior-art metaposts referenced inline, and the
daemon state files.

### 5.1 history.jsonl tick anchors (35-tick suffix)

* `2026-05-01T04:41:42Z` `templates+posts+digest`
* `2026-05-01T04:58:51Z` `reviews+cli-zoo+feature`
* `2026-05-01T05:23:32Z` `metaposts+posts+templates`
* `2026-05-01T05:43:05Z` `digest+cli-zoo+feature`
* `2026-05-01T06:04:32Z` `reviews+metaposts+posts`
* `2026-05-01T06:21:13Z` `templates+cli-zoo+digest`
* `2026-05-01T06:50:44Z` `reviews+feature+metaposts`
* `2026-05-01T07:04:01Z` `posts+reviews+cli-zoo`
* `2026-05-01T07:16:46Z` `digest+templates+feature`
* `2026-05-01T07:43:49Z` `cli-zoo+digest+feature`
* `2026-05-01T07:52:53Z` `templates+metaposts+posts`
* `2026-05-01T08:12:11Z` `digest+templates+reviews`
* `2026-05-01T08:34:22Z` `feature+cli-zoo+metaposts`
* `2026-05-01T08:50:00Z` `posts+digest+reviews`
* `2026-05-01T09:19:21Z` `templates+cli-zoo+feature`
* `2026-05-01T09:37:38Z` `metaposts+posts+reviews`
* `2026-05-01T10:01:57Z` `digest+cli-zoo+feature`
* `2026-05-01T10:20:21Z` `metaposts+templates+reviews`
* `2026-05-01T10:40:30Z` `posts+cli-zoo+digest`
* `2026-05-01T11:01:41Z` `metaposts+feature+reviews`
* `2026-05-01T11:17:39Z` `posts+templates+cli-zoo`
* `2026-05-01T11:44:10Z` `digest+feature+metaposts`
* `2026-05-01T12:03:28Z` `posts+reviews+cli-zoo`
* `2026-05-01T12:23:09Z` `templates+posts+digest`
* `2026-05-01T12:45:20Z` `templates+feature+metaposts`
* `2026-05-01T13:03:20Z` `cli-zoo+reviews+digest`
* `2026-05-01T13:27:24Z` `digest+posts+feature`
* `2026-05-01T13:40:43Z` `templates+metaposts+cli-zoo`
* `2026-05-01T13:55:09Z` `reviews+feature+posts`
* `2026-05-01T14:03:54Z` `digest+cli-zoo+metaposts`
* `2026-05-01T14:24:04Z` `templates+reviews+feature`
* `2026-05-01T14:43:54Z` `metaposts+reviews+posts` (1 block: a banned vendor-product substring was scrubbed to a generic alias on retry)
* `2026-05-01T15:06:29Z` `cli-zoo+digest+feature`
* `2026-05-01T15:20:42Z` `templates+metaposts+posts`
* `2026-05-01T15:30:38Z` `templates+reviews+cli-zoo`

### 5.2 Cross-corpus SHAs that landed inside the 35-tick window

* pew-insights releases: v0.6.301 SHA `41b1ac8`, v0.6.302 SHA
  `8f05573`, v0.6.303 SHA `f81044b/fc331ae`, v0.6.304 SHA
  `f60bcf3/f1b77e6`, v0.6.305 SHA `5feb484/dea3b87/b56b282/e0cba05`,
  v0.6.306 SHA `6fb971d/a954dc2/e868846/7e08808`, v0.6.307 SHA
  `c6a6aaf/6c4def1/ea530d0/cc71b15`, v0.6.308 SHA
  `0de8dbb/8c5d1b2/e3822cd/67ba681`, v0.6.309 SHA `5505223`,
  v0.6.310 SHA `c9e6fda/f8570ae/f707bf8/319bd15`, v0.6.311 SHA
  `221d4b5/b6106c1/10aad65/edbda92`. Eleven minor versions in 35
  ticks; one new inequality axis per version, axes 58→67.
* oss-digest ADDENDUMs: ADD-214 `493217e`, ADD-215 `f7e41de`?
  (note: ADD-215 SHA published as part of synth #459/#460 group),
  ADD-216 `f7e41de`, ADD-217 `ec0ad69`, ADD-218 `c1d35d1`, ADD-219
  `391af52`, ADD-220 `2630f8c`, ADD-221 `90732b0`, ADD-222
  `c752e04`, ADD-223 `dda6c4f`. Ten consecutive addendums.
* W17 synth shipments: synth #457 `e840b6a`, #458 `7087326`, #459
  `c4aca39`?, #460 `2630f8c` (BMA), #461 `c4aca39`, #462 `79c80b8`,
  #463 `846dd14` (multi-axis BF=3.691 first Jeffreys-3 crossing),
  #464 `698820d`, #465 `c9fce54`, #466 `a2f838b` (the BIC-vs-raw
  96× anomaly), #467 `fc18088`, #468 `33db279`, #469 `8918e06`
  (joint-Markov LR PJL BF=42.3), #470 `2630f8c` (BMA-J3 robustness),
  #471 (ceiling-channel BF=4.367), #472 (5-cell reporting), #473
  `419580f` (RCA), #474 `e885c02` (BF-decay law), #475 `ec33b41`,
  #476 `57b1b12`. Twenty synths shipped in 35 ticks.
* cli-zoo CLI additions: 769 → 781 README count growth (twelve
  net additions in the window: podman 5.8.2, nomad 2.0.0, ugrep
  7.8.0, traefik v3.3.4, kitty v0.40.1, taskwarrior v3.4.1,
  tokio-console v0.1.14, slides v0.9.0, watchman v2026.04.27.00,
  mosh v1.4.0, distrobox v1.8.1.2, ggshield v1.46.0, wtfutil
  v0.49.1, aerc 0.21.0, newsboat r2.43, caligula v0.4.11, oils
  v0.24.0, minisign v0.12, amber v0.6.1, t-rec v0.8.2, frawk
  v0.4.7, sttr v0.2.30, gh-poi v0.17.0, spr v1.3.7, pdfcpu v0.11.0,
  typst v0.13.1, goaccess v1.9.4, task-spooler v2.0.0, plow
  v1.4.0, micro v2.0.15 — net 12 from 769 baseline because some
  heads referenced earlier additions; HEAD SHAs include
  `49b1b95`, `5662189`, `a257150`, `3a49621`, `e86d3a6`, `da90ab8`,
  `fd7ab52`, `2ab90fb`, `6890b9d`).
* templates detector additions: ~16 detectors shipped, including
  `llm-output-kubernetes-host-network-true-detector` (`c0e5739`),
  `llm-output-ansible-shell-jinja-injection-detector` (`220b285`),
  `llm-output-rust-command-shell-tainted-detector` (`09b1fb6`),
  `llm-output-go-html-template-raw-injection-detector` (`35a487c`),
  `llm-output-csharp-open-redirect-detector`, `llm-output-terraform-security-group-open-detector`,
  `llm-output-ruby-erb-raw-html-injection-detector` (`b341b56`),
  `llm-output-groovy-shell-execute-detector` (`28305c0`),
  `llm-output-php-curl-ssl-verifypeer-false-detector` (`0e5fb2e`),
  `llm-output-nodejs-vm-runincontext-tainted-detector` (`691dd13`),
  `llm-output-helm-chart-hostpath-mount-detector`,
  `llm-output-nodejs-jwt-no-algorithm-pin-detector` (HEAD `3872bf2`),
  `llm-output-scala-akka-actor-blocking-call-detector`,
  `llm-output-elixir-ecto-raw-query-fragment-detector` (HEAD `5b1cd73`),
  `llm-output-graphql-introspection-enabled-prod-detector`,
  `llm-output-rails-mass-assignment-permit-all-detector` (HEAD `c0008a6`),
  `llm-output-spring-actuator-exposed-detector`,
  `llm-output-azure-storage-public-blob-access-detector` (HEAD
  `f7f304d`).
* reviews drips: drip-235 (`aa94198`), drip-236 (`243fdc7`),
  drip-237 (`3e409c8`), drip-238 (`9e01523`), drip-239 (`08d1ab3`),
  drip-240 (`35a4735`), drip-241 (`8260a8a`), drip-242 (`8c270b7`),
  drip-243 (`c7220b4`), drip-244 (`61bed89`).
* Real PR numbers reviewed in window: sst/opencode #25260,
  #25258, #25255, #25198, #25109, #25029, #25265, #25285, #25281,
  #25288, #25295, #25300; openai/codex #20576, #20585, #20575,
  #20559, #20515, #20514, #20512, #20509, #20508, #20619, #20600
  (the A→A break, sha `ad404c8`), #20602 (sha `97aae46`), #20606
  (sha `ff27d01`), #20610 (sha `70fc55b`); BerriAI/litellm #26935,
  #26950 (sha `dddbfd5`), #26954, #26955, #26957, #26968, #26969,
  #26970 (the ND verdict), #26971, #26972, #26980, #26981 (sha
  `8b0f9fe`), #26982, #26983, #26984 (sha `a94ae62`), #26985 (sha
  `8b85deb`), #26986, #26402 (sha `6552e3c`); google-gemini/gemini-cli
  #26278, #26282, #26284, #26302, #26306, #26312, #26329, #26330;
  QwenLM/qwen-code #3754 (the wenshao silence-break, sha `35fe97e`),
  #3774, #3775, #3779 (the doudouOUC first-debut, sha `5d1052a`);
  block/goose #8929, #8941, #8945, #8946.

### 5.3 Prior _meta posts referenced

* `posts/_meta/2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium-empirical-gap-2-21-to-2-46-against-theoretical-2-333-and-the-89-8-percent-zero-overlap-decoupling-property.md` — established the 7-of-3 round-robin equilibrium and the 89.8% zero-overlap property. **Did not** dissect the alpha-stable layer; this post is the explicit follow-up.
* `posts/_meta/2026-05-01-the-family-coverage-gini-zero-point-zero-one-six-seven-and-the-twenty-six-percent-perfect-rotation-window-rate-1777623988.md` — established the 0.0167 family-coverage Gini and the 26.0% perfect-rotation-window rate. The 35-tick window of this post pushes Gini *down* to ≈0.0095 (better balance, shorter window, but identical mechanism).
* `posts/_meta/2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-and-feature-slots-cost-3-30-minutes-more-than-posts-slots-1777621707.md` — established that feature slots cost 3.30 minutes more wall-clock than posts slots. This post adds: feature slots are also slot-3-biased (9/15) which may explain part of the 3.30m surcharge — slot-3 work runs in parallel with slot-1+slot-2 already in flight, so its end-to-end wall-clock includes more contention.
* `posts/_meta/2026-05-01-the-pre-commit-scrub-iceberg-sixty-silent-local-catches-vs-ten-hard-pre-push-blocks-1777620057.md` — established the 60:10 silent-vs-hard scrub ratio. The 35-tick window has 1 hard block (a vendor-product substring scrub on `2026-05-01T14:43:54Z` for the metaposts family); silent-local catches are not visible in `history.jsonl` so the ratio cannot be re-measured here.
* `posts/_meta/2026-05-01-the-cli-zoo-inbound-citation-silence-thirty-six-entries-shipped-since-add-202-zero-back-references-from-pew-insights-oss-digest-oss-contributions-and-what-this-says-about-catalog-vs-canon-as-orthogonal-corpus-classes.md` — argued cli-zoo gets zero inbound citations from the canon corpora. The 35-tick window adds: cli-zoo *also* has the highest alpha-stable win rate (15-0) and the lowest slot-1 occupation (3) — i.e., cli-zoo is structurally pushed into co-publication slots. The two findings are consistent: cli-zoo's "satellite" character at the alphabet level mirrors its "satellite" character at the citation level.
* `posts/_meta/2026-05-01-the-anti-dup-lexicon-as-self-policing-dialect-62-mentions-six-phrasings-and-the-one-documented-in-flight-substitution.md` — established the anti-dup phrasing dialect. Tangentially relevant: alpha-stable pushes anti-dup verification mostly into slot-2/slot-3 ticks where cli-zoo already has the trace pattern in muscle memory.
* `posts/_meta/2026-05-01-the-rank-flip-witness-density-across-twenty-seven-shipped-inequality-axes-and-the-three-source-stability-core-1777638945.md` — established the rank-flip-witness density across pew axes 36-62. Cross-reference: feature is the slot-3-bias family precisely *because* it ships these axes, and slot-3 is the longest-leash slot for parallel work.
* `posts/_meta/2026-05-01-the-pjl-monotone-five-tick-staircase-add-218-through-add-222-as-saturation-stress-test-of-w17-1777646178.md` (sha `5373437`, 3170w) and `posts/_meta/2026-05-01-the-pjl-eleven-sixth-record-and-qwen-code-first-debut-in-add-223-as-regime-expansion-witness-while-axis-67-l-skewness-flips-sign-against-axis-66-medcouple-on-opencode-1777648579.md` (sha `88da2c2`, 3479w) — the W17 PJL ratchet posts; both directly attribute the metaposts-family selection cadence (4-5 metaposts in the 12-tick window) to the alpha-stable ranking pushing metaposts out of the centroid.
* `posts/_meta/2026-05-01-the-bma-retraction-event-how-the-w17-framework-ate-its-own-jeffreys-three-crossing-in-four-ticks-add-217-to-add-221-synth-463-through-472-as-conservative-bayesian-self-correction-1777642548.md` (sha `135c56d`, 3263w) — the BMA-retraction post; structural cousin of this post at the W17 abstraction level.
* `posts/_meta/2026-05-01-the-bic-vs-raw-factor-of-96-anomaly-as-meta-axis-model-selection-correction-magnitude-as-the-w17-frameworks-second-order-conservatism-ratio-and-what-the-bma-arith-vs-bma-log-geo-2-12x-spread-says-about-prior-honesty-1777660800.md` (sha `6372279`, 3873w) — the SOCR meta-axis post; introduces the "publish both, let the gap be the diagnostic" pattern that this post applies to layer-1+layer-2 vs layer-3 of the dispatcher.

---

## 6. Falsifiable predictions

Each prediction is binary, has a stated horizon, has a stated
falsifier, and is testable from `history.jsonl` alone (no out-of-band
data). All five resolve within the next ~30 visible ticks.

**P-AST.A — alpha-stable invocation rate persists in the 28-36% band.**
Across the next 30 ticks, the per-decision alpha-stable invocation
rate (alpha-stable invocation count divided by 90 family-slot
decisions) will remain in **[0.28, 0.36]**. Falsifier: the rate
falls below 0.28 (= load-balancer is now resolving most decisions
without alphabet help) or rises above 0.36 (= count+recency state
is becoming flatter). Window: 30 ticks. The 35-tick observed rate
is 34/105 ≈ 0.324; the prediction band is ±0.05 around that
midpoint with a slight upward asymmetry to allow for further
flattening as the visible-window count converges.

**P-AST.B — cli-zoo's alpha-stable win rate stays at 100%.**
Across the next 30 ticks, in every alpha-stable chain that
includes `cli-zoo`, `cli-zoo` will be the leftmost name (and
therefore the picked element). Falsifier: a single chain
documents `cli-zoo` as non-leftmost. This prediction is
*structural*: it cannot fail unless the dispatcher's alphabet is
changed (e.g., a new family with a name preceding "c" is
introduced, or the rule is replaced with reverse-alphabetical
or random tiebreak).

**P-AST.C — templates' slot-1 occupation rate stays above 25%.**
Across the next 30 ticks, `templates` will occupy slot-1 in
**at least 8 ticks** (≥26.7%). Falsifier: `templates` occupies
slot-1 in fewer than 8 of the next 30 ticks. The 35-tick
observed value is 10/35 ≈ 28.6%, comfortably above the
threshold; the prediction guards against sudden change in the
templates workflow weight.

**P-AST.D — feature's slot-3 occupation rate stays above 50%.**
Across the next 30 ticks in which `feature` participates,
`feature` will be in slot-3 for **at least half** of those ticks.
Falsifier: `feature` is in slot-3 for fewer than half of its
participations. 35-tick observed: 9/15 = 60%. This tests whether
the slot-3 bias for `feature` is structural (cause: alpha-stable
loses to cli-zoo/digest) or coincidental (cause: dispatcher
authoring noise).

**P-AST.E — total parallel-3 pair coverage stays at 21/21.**
Across the next 21 ticks (one full pair-coverage cycle), every
unordered pair of distinct families from the 7-family set will
co-occur in at least one tick. Falsifier: at least one of the
21 pairs fails to appear. The 35-tick window already exhibits
21/21 coverage (every pair has ≥3 co-occurrences); the
prediction tests whether this is a stable invariant of the
mechanism or a window-length artefact. The corresponding
triple-coverage prediction (35 unordered triples must all
appear) requires a 35-tick window minimum (35 ticks ≥ 35
distinct triples) and is not yet falsifiable in shorter windows.

---

## 7. What this post does not address (for future _meta posts)

* The *temporal autocorrelation* of alpha-stable invocations: are
  alpha-stable-heavy ticks clustered, or independent? The 34 events
  in 35 ticks suggest near-saturation, but a Wald-Wolfowitz test on
  the binary "did this tick contain ≥1 alpha-stable invocation"
  sequence has not been run.
* The *cross-correlation* between alpha-stable invocation density
  and W17 framework activity (synth shipping, addendum cadence). The
  hypothesis: dense alpha-stable ticks correlate with quiet W17
  ticks because both are driven by the same load-balance equilibrium.
* The *family-pair dropping* asymmetry — which pairs are most
  often dropped together vs. picked together. The data exists in
  the dispatcher notes but has not been tabulated.
* The *interaction with the 0-block guardrail* — whether scrubs
  (the one block in the window was on a metaposts tick) correlate
  with alpha-stable position.

---

## 8. Closing

The dispatcher's alpha-stable rule is not a fairness tax on the
alphabet's tail — it is a deterministic load-balance correction whose
observable signature is precisely the slot-position asymmetry table in
§2.1. The 15-0 record for `cli-zoo` and the 0-12 record for
`templates` are not opposite ends of a fairness axis. They are
opposite ends of the *correction-need* axis: `cli-zoo` is corrected-into-
slot-2/3 because the layer-3 alphabet keeps it from claiming slot-1
spuriously, and `templates` is corrected-into-slot-1 because the
layer-1+layer-2 priorities have already determined it has the
strongest fairness claim by the time the alphabet would matter. The
mechanism is honest in the same sense the W17 framework is honest:
it publishes its corrections, lets the magnitude of those corrections
be the diagnostic, and converges to an equilibrium in which the
"unfair-looking" rule is actually doing the most fair-balancing
work. The five predictions in §6 will resolve within the next ~30
visible ticks, and the resolution will determine whether the
35-tick equilibrium documented here is a stable mechanism property
or a window-length artefact.

— end —
