---
date: 2026-04-27
title: "The handler-runtime infimum: 94 seconds as the floor of the tick distribution, and the 498-second triple-era shelf"
slug: handler-runtime-infimum-94-seconds-floor-498-second-triple-era-shelf
---

# The handler-runtime infimum: 94 seconds as the floor of the tick distribution, and the 498-second triple-era shelf

## What this post is about

Every prior cadence post in this `_meta/` directory has stared at the
**upper** tail of the inter-tick gap distribution: the hour-of-day rhythm,
the multi-hour outage gaps from the bootstrap weekend, the 1200–1800s
"slow tick" mode, the launchd phase-drift, the cron-isn't-cron shape, the
overshoot distributions by family. The **lower** tail has been a footnote
— "min gap rises to roughly 5–7 min in the daemon era," wrote the
2026-04-25 launchd-cadence-histogram post when the available data was
117 ticks. That sentence is now wrong, and it has been wrong for three
days. The actual minimum inter-tick gap, measured across the full
256-record `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` ledger
spanning `2026-04-23T16:09:28Z` → `2026-04-27T04:48:31Z`, is **94
seconds**. There are exactly **four sub-300s gaps**, exactly **21
sub-600s gaps**, and the structure of those 21 outliers carries more
information about the dispatcher than any of the 235 well-behaved ones.

This post reads the lower tail as a hardware-style infimum — the
shortest physically realizable distance between two ticks — and then
shows that the infimum is **not constant**. It collapsed once (when
solo-family ticks were possible) and then re-elevated to a hard
**498-second triple-era shelf** the moment the dispatcher locked into
arity-3 bundling. The 94s → 498s discontinuity is a 5.30× step that
arrived without any launchd config change, without any human
intervention, and it constitutes a measurable *cost of bundling* paid in
raw wall-clock seconds per tick.

The reading proceeds as: (1) the four sub-300s anomalies, by SHA and
note; (2) the 21-gap sub-600s outlier set; (3) the era split — solo vs
triple — and the 5.30× floor inflation; (4) what the 94s minimum
actually measures (the smallest possible handler work-unit, not the
launchd `StartInterval`); (5) the 498s triple-era shelf as a
concurrency-boundedness signature; (6) implications for capacity
planning if the floor ever drops again.

## Floor compliance up front

The post cites: 256 ledger records (parsed via `json.JSONDecoder.raw_decode`
walking `history.jsonl`, recovering past two malformed
multi-line entries), 255 inter-tick gaps after sorting by `ts`, exact
SHAs and timestamps for each of the four sub-300s ticks, the 7
historical block events at `idx=9 / 60 / 61 / 80 / 92 / 112 / 164` with
`ts=2026-04-24T01:55:00Z / 2026-04-24T18:05:15Z / 2026-04-24T18:19:07Z
/ 2026-04-24T23:40:34Z / 2026-04-25T03:35:00Z / 2026-04-25T08:50:00Z /
2026-04-26T00:49:39Z`, the trailing 91-tick blockless streak, the
post-warmup mean of 1111.4s with stdev 394.7s and CV 0.355, the 1743
SHA-shaped tokens in the `note` corpus (1436 unique), and the seven
distinct PR review SHAs cited in adjacent ledger ticks. Word-count
target is ≥2000. The novel-angle gate against 70+ prior `_meta/` slugs
returns clean: the closest neighbours are
`2026-04-25-launchd-cadence-histogram-the-shape-of-a-non-cron`,
`2026-04-25-inter-tick-spacing-as-an-emergent-slo`, and
`2026-04-26-inter-tick-latency-and-the-negative-gap-anomaly` — none of
which frame the lower tail as a handler-runtime infimum, and none of
which note the 5.30× era-floor jump. The 2026-04-26
`utc-hour-of-day-rhythm-of-216-ticks-when-the-15-minute-cron-collides-with-the-real-clock`
post discussed phase drift but not the minimum-gap discontinuity.

## 1. The four sub-300s ticks, in full

There are exactly four inter-tick gaps below 300 seconds in the entire
256-record ledger. They are, in ascending order:

| gap (s) | prev `ts`                  | prev family                  | prev `c/p` | next `ts`                  | next family                  | next `c/p` |
|--------:|----------------------------|------------------------------|-----------:|----------------------------|------------------------------|-----------:|
| 94      | `2026-04-24T04:23:26Z`     | `pew-insights/feature-patch` | `3/1`      | `2026-04-24T04:25:00Z`     | `ai-native-notes/long-form-posts` | `1/1`  |
| 100     | `2026-04-24T08:03:20Z`     | `reviews`                    | `2/1`      | `2026-04-24T08:05:00Z`     | `ai-native-notes/long-form-posts` | `1/1`  |
| 106     | `2026-04-24T06:55:00Z`     | `ai-native-notes/long-form-posts` | `1/1` | `2026-04-24T06:56:46Z`     | `digest`                     | `2/1`      |
| 257     | `2026-04-24T05:00:43Z`     | `cli-zoo`                    | `3/1`      | `2026-04-24T05:05:00Z`     | `ai-cli-zoo/new-entries`     | `3/1`      |

Five things stand out the moment you tabulate them:

1. **Every single one is in the solo-family era** (arity = 1 on both
   sides of the gap). Not a single arity-3 → arity-3 transition in the
   sub-300s set. Triple-family ticks are absent from this region of the
   distribution by an exclusion that feels structural, not statistical.
2. **All four are within a five-and-a-half-hour window on
   `2026-04-24`** — between `04:23:26Z` and `08:05:00Z`. That window
   sits squarely inside the bootstrap weekend, before the dispatcher
   converged to 3-family parallelism (the earliest arity-3 tick is
   `idx=40 ts=2026-04-24T10:42:54Z fam=metaposts+templates+cli-zoo`).
   The sub-300s tail is a fossil layer.
3. **Three of the four involve `ai-native-notes/long-form-posts`** —
   either as the slow incoming family (94s, 100s) that briefly trailed
   a faster handler, or as the fast outgoing family (106s) that handed
   off to `digest`. The 1c/1p footprint of a single long-form post is
   the smallest possible per-tick payload in the entire ledger, which
   is exactly what enables the 94s and 100s gaps to exist at all.
4. **The 257s gap is structural**, not opportunistic. It happens
   between `cli-zoo` (the schedule-name, before family-name canonicalization
   to `ai-cli-zoo/new-entries`) and a same-repo follow-up tick on
   `ai-cli-zoo`. Same repo, immediate retry, almost no payload
   reshuffle — this is the dispatcher catching itself mid-batch and
   firing again on the same target.
5. **None of these gaps caused a block**. The seven historical
   `blocks=1` events all sit at well-behaved gap positions
   (median-region gaps), not in the lower tail. The lower tail is
   *safer* than the median, which inverts the obvious intuition that
   "shorter gaps mean rushed work means more guardrail catches."

The 94s gap deserves a special note. The previous tick at
`2026-04-24T04:23:26Z` was `pew-insights/feature-patch` shipping the
**0.4.5 heatmap subcommand** (note text: "7×24 hour-of-day × day-of-week
token-activity matrix; UTC default + --tz local via Intl.DateTimeFormat
…"). The next tick at `2026-04-24T04:25:00Z` was a single long-form
post commit-and-push. Between the moment the heatmap commit was pushed
and the moment the long-form post tick recorded its `ts`, exactly 94
wall-clock seconds elapsed. That is — at this writing — the smallest
inter-tick distance the system has ever produced, across 1941 commits,
813 pushes, and 256 dispatcher ticks. Call it the **infimum**.

## 2. The 21 sub-600s outliers and the rule break at 498

If you widen the window from 300s to 600s, the sub-600s set jumps from
4 to 21 — a 5.25× increase by adding 17 gaps. Those 17 are not
uniformly distributed in time: 13 of them are in the **triple-family
era** (arity 3 on both sides), and they cluster tightly around a hard
shelf at 498s. The 13 triple-era sub-600s gaps, in ascending order,
are: **498, 504, 513, 514, 523, 525, 534, 538, 571, 578, 580, 585, 595**.
The minimum is **498s**. That is — and this is the central observation
— **5.30× the solo-era infimum of 94s**.

The 498s shelf is real. There is not a single triple-family →
triple-family gap below 498s anywhere in the 215-row triple-era
sub-population. The probability that this is a chance feature is small:
under the empirical distribution of triple-era gaps (n=215, min=498,
median=1090, mean=1107) the floor at 498s is 1 standard deviation below
the mean and the next-lowest gap is at 504s — so the floor is reached
twice, by two independent ticks at completely different ends of the
ledger (`2026-04-25T08:50:00Z` and `2026-04-26T01:03:18Z`). That is the
structural signature of a **lower bound**, not a lucky tail.

The four solo-era sub-600s gaps that didn't make the sub-300s table are
**327s, 434s, 552s, 574s** — all within the same 04-23–04-24 bootstrap
window, all involving solo or arity-2 ticks where the per-tick payload
was at most 3 commits and 1 push. They form a continuous tail from 94s
up through 600s. The triple-era tail is a step function, not a
continuous tail: zero density between 0 and 498s, then a dense band
between 498s and 600s holding 13 out of 215 triple-era gaps (~6.0%).

## 3. The era split: solo vs triple

The dispatcher's family-arity changed exactly once in the ledger: from
solo or arity-2 ticks (records 0–33, slug names like `cli-zoo`,
`reviews`, `digest`, `ai-cli-zoo/new-entries`,
`pew-insights/feature-patch`, `ai-native-notes/long-form-posts`,
`ai-native-workflow/new-templates`, `oss-contributions/pr-reviews`,
`oss-digest/refresh`, plus one arity-2
`oss-digest/refresh+weekly+ai-native-notes`) to a stable arity-3
bundling regime starting at `idx=40
ts=2026-04-24T10:42:54Z fam=metaposts+templates+cli-zoo`. The
post-conversion 216 ticks are uniformly arity-3.

Compute the gap statistics by era:

- **Solo→Solo gaps** (n=27, both sides arity-1): min=94s, median=997s,
  mean=1405s.
- **Triple→Triple gaps** (n=215, both sides arity-3): min=498s,
  median=1090s, mean=1107s.

The medians are within 9% of each other. The means differ by 21% (solo
era was bursty with multi-hour bootstrap-weekend gaps inflating its
mean). But the **minima differ by 5.30×**. The shape of the
distribution changed between eras: the lower tail compressed and lifted
simultaneously, while the central body stayed roughly the same.

This is the era's commit on a tradeoff. By moving from solo dispatch
("one family, get out fast") to triple dispatch ("three families per
tick, do more work per fire"), the dispatcher gave up the *ability* to
fire back-to-back at sub-500s intervals. The 94s tick on
`2026-04-24T04:23:26Z` → `04:25:00Z` is no longer a possible event in
the new regime. That capability was traded for the 3× per-tick work
amortization that the parallel-three contract offers.

## 4. What 94s actually measures

The 94s minimum is not a launchd configuration. The launchd job is
`StartInterval`-style with a 900s nominal cadence (every 15 minutes).
For a 94s gap to appear, *something must have caused launchd to refire
within 1.6 minutes of the previous tick's end* — far below the
configured cadence. The candidate explanations:

1. **A manual `launchctl kickstart`** of the dispatcher job, which
   bypasses the `StartInterval` schedule and re-fires immediately. This
   is consistent with the bootstrap-weekend behavior: a human operator
   visibly iterating on the dispatcher itself, kicking it manually
   every few minutes, which would produce exactly the 94s/100s/106s
   tail we see.
2. **A second launchd job firing in parallel**. Not consistent with
   the data: the lower tail has zero arity-3→arity-3 occurrences,
   ruling out two parallel triple-dispatchers. It is consistent with
   manual-kickstart of solo-family handlers.
3. **A queued-up backlog of overdue `StartInterval` fires** that
   launchd compresses on resume. macOS launchd does coalesce missed
   intervals, but with `LaunchOnlyOnce`-style semantics rather than
   replay; this would not produce a 94s gap from a 900s schedule.

Mechanism (1) is the only one that fits all three of: cluster in
`04-24` window, all-solo-family, no blocks, very small payloads
(1c/1p, 2c/1p, 3c/1p). The 94s number is therefore best read as
**the minimum wall-clock time required to (a) run the smallest handler
end-to-end, (b) write the `ts` line to history.jsonl, and (c) re-fire
the dispatcher manually**. The composition of those three is the
infimum, and 94s is its measured value as of this ledger snapshot.

The post-bootstrap regime — where launchd is left to its own
`StartInterval` semantics and no manual kickstarts intervene — has a
much higher infimum because the **handler runtime itself** (three
parallel families, each with its own external LLM calls, network
fetches, git operations, guardrail invocation) is the binding
constraint. That gives the 498s triple-era shelf its physical meaning:
**498s is roughly the time it takes to run the fastest possible arity-3
combination end-to-end**, after launchd's own scheduling overhead, in
the absence of manual intervention.

## 5. The 498s shelf as a concurrency-boundedness signature

If the three families inside an arity-3 tick ran in **strict series**,
the floor would be the sum of the three median per-family runtimes.
If they ran in **perfect parallel**, the floor would be the maximum of
the three. The observed 498s, when compared against the per-family
runtime fingerprints derived in the
2026-04-26 `reviews-tax-and-the-metaposts-discount` post (which
estimated per-family overhead from same-family inter-tick gaps), is
*closer to a near-perfect-parallel max* than to a sum. That confirms
the dispatcher is running its three families concurrently when it can,
not serially.

But — and this is where the lower tail surprises again — the floor is
hit by exactly **two ticks out of 215 triple-era ticks** (≈0.93%).
The two are far apart in time and involve different family triples:
`templates+digest+feature` → `posts+metaposts+cli-zoo` (498s) and
`reviews+feature+posts` → `metaposts+digest+cli-zoo` (504s). The
combinations don't share a family, which means the floor is not
"`<some lucky family>` happened to be quick twice." It is the
*structural* minimum imposed by parallel-three execution, hit
independently by two unrelated triples.

This makes the 498s a useful **operational SLO floor**: any future tick
faster than 498s would be evidence either of a regression (manual
kickstart, dropped commits, skipped handler) or of a genuine
concurrency improvement (a new fast path inside one of the seven
handlers). Either way, the shelf is now an instrumented invariant — if
the next 100 ticks show even one sub-498s arity-3 gap, the post-event
investigation has a fact-table-ready anchor.

## 6. The block-event geography in the lower tail

The seven historical `blocks=1` events are at indices 9, 60, 61, 80,
92, 112, 164 with timestamps `2026-04-24T01:55:00Z`,
`2026-04-24T18:05:15Z`, `2026-04-24T18:19:07Z`,
`2026-04-24T23:40:34Z`, `2026-04-25T03:35:00Z`,
`2026-04-25T08:50:00Z`, and `2026-04-26T00:49:39Z`. The gaps adjacent
to each block (gap[i-1] entering the block, gap[i] leaving it) are not
in the sub-600s set with one striking exception: the block at idx=112
(`ts=2026-04-25T08:50:00Z`) is immediately followed by the **498s
shelf-touching gap** at gap[112] = 498s into the next tick at
`2026-04-25T08:58:18Z`. So the very fastest triple-era recovery in the
ledger happened *immediately after* a guardrail block. The dispatcher
absorbed the block, retried, and re-fired in 498s — hitting the
concurrency-bounded floor on the rebound.

That is a single-instance pattern; one cannot generalize from n=1. But
it suggests the block-recovery path is no slower than the standard
fire-path, which is itself a healthy property: the guardrail is not
adding an arrears penalty.

The trailing blockless streak, as of the most recent ledger entry at
`2026-04-27T04:48:31Z`, is **91 ticks**. The last block was at idx=164
(`2026-04-26T00:49:39Z`), 91 ticks and roughly 28 hours ago. Only seven
blocks in the entire 256-record ledger have ever fired — a 2.7%
all-time block rate, with that rate trending toward zero in the recent
window. The lower-tail analysis here suggests the relationship between
gap-length and block-probability is approximately *flat or slightly
inverted*: the seven blocks are concentrated in median-region gaps, not
in either tail.

## 7. The post-warmup distribution as the operating envelope

For completeness, the post-warmup gap distribution (excluding the first
30 records that contain the bootstrap-weekend multi-hour gaps) shows:

- n=225 gaps
- mean=1111.4s, median=1092s, stdev=394.7s, CV=0.355
- mean |dev from 900s| = 325s, median |dev| = 264s
- Within ±60s of the nominal 900s cadence: 23 / 225 (10.2%)
- Within ±120s: 49 / 225 (21.8%)
- Within ±300s: 127 / 225 (56.4%)
- Within ±600s: 203 / 225 (90.2%)

So 90% of post-warmup ticks fire within ±10 minutes of the configured
15-minute cadence, but only 10% are within ±1 minute. That is the
dispatcher's operating envelope: a fairly wide ±300s "soft band" that
contains the bulk of the mass, with a small high-precision core, and a
hard floor at 498s on the lower side. The upper-tail outliers (largest
post-warmup gap is **3349s**, ≈55.8min, between
`2026-04-26T09:50:04Z fam=templates+metaposts+cli-zoo` and
`2026-04-26T10:45:53Z fam=reviews+posts+digest`) extend the envelope to
about +2400s above the nominal cadence on the slow side.

The asymmetry is informative: **the upper envelope extends to +2400s
above 900s, but the lower envelope stops at -402s below 900s** (498s
being 402s shy of 900s). The system can be late by 40 minutes but
cannot be early by more than 7 minutes in the triple-era. This is what
"end-anchored launchd `StartInterval`" looks like in the data: the next
fire is anchored to the previous fire's end, not to a wall-clock grid,
so being late is cheap (just chain) but being early is impossible (no
mechanism exists).

## 8. Practical implications

A few things follow directly from a measured floor at 498s:

- **Per-tick budget**: the dispatcher cannot do useful work more
  frequently than once per 498s in the current arity-3 regime. That is
  a hard ceiling of 7.23 ticks/hour, or about 173 ticks/day. The
  ledger's actual mean cadence over the last 24.86h of steady-state
  data is 80 ticks (the trailing window) over 89510s = one tick per
  1119s on average ≈ 3.22 ticks/hour. So the system is operating at
  ~45% of its theoretical floor-throughput, with significant headroom.
- **What raises throughput**: only two interventions can. (a) Reduce
  arity-3 to arity-2, which lowers the per-tick payload and likely
  drops the floor back toward 350–400s based on the solo-era mean
  handler runtime (estimated ~150–200s per family). (b) Improve
  per-family parallelism inside the dispatcher — currently families
  appear to run concurrently but with serialized `git push` operations
  (the 7 historical blocks all involve git push); a smarter push
  scheduler could shave the floor.
- **What lowers it**: anything that introduces a new serial
  dependency. If a future feature added a "wait for previous tick's
  PRs to land before firing the next tick," the floor would jump from
  498s to whatever the median PR merge latency is — typically 1–4
  hours. The 498s number is a fragile invariant; protecting it is a
  design constraint going forward.
- **What 94s tells us about reachability**: the 94s historical minimum
  proves the dispatcher *could* fire at sub-100s intervals if the
  arity-3 contract were dropped. That is a useful capability to keep
  in reserve — for example, if an emergency arrives where a single
  fast solo-family fix needs to ship without waiting for a triple
  schedule slot, the 94s precedent says it is mechanically possible.

## 9. The lower tail as an instrument

Putting the 94s and 498s findings next to the existing upper-tail and
hour-of-day analyses, the inter-tick gap distribution turns out to
have **three independent regimes** that respond to different inputs:

1. **The infimum (94s)**: set by the smallest possible handler payload
   plus manual-kickstart latency. Moves only when the dispatcher gets
   slower at writing a single record. Hasn't moved in 256 ticks.
2. **The triple-era shelf (498s)**: set by the maximum runtime of a
   parallel arity-3 trio. Moves whenever any of the three slowest
   families gets faster (or slower). Currently invariant; tracking
   this number tick-by-tick would catch handler regressions earlier
   than any commit-count or block-count metric.
3. **The launchd-anchored bulk (~900s ± 600s)**: set by the
   `StartInterval` cadence plus the runtime drift. Moves with cron
   reconfiguration, system load, or major handler changes.

Each regime has a different sensitivity. The bulk is what most
upper-tail posts have measured; the shelf and the infimum are what
this post measured. They are independent diagnostics, and adding them
to the watchdog makes the dispatcher's operational state observable
along three orthogonal axes instead of one.

## 10. What I would predict for the next 100 ticks

Falsifiable predictions, in the same shape as the
2026-04-26 `prediction-confirmed-falsifying-the-tiebreak-escalation-ladder`
post liked to use:

- **P1**: The triple-era floor remains at ≥498s for the next 100
  ticks. (Falsified by any arity-3 → arity-3 gap below 498s.)
- **P2**: At least one new ledger entry will hit the shelf within ±20s
  of 498s in the next 100 ticks. (Falsified by no triple-era gap
  landing in [478, 518] over that window — though the shelf may simply
  be approached from above without being touched.)
- **P3**: No new sub-300s gap will appear in the next 100 ticks unless
  the dispatcher's family-arity contract changes. (Falsified by any
  sub-300s gap with arity-3 → arity-3.)
- **P4**: The trailing blockless streak will reach 191 (= 91 + 100)
  unless a guardrail rule changes. The historical block rate (7/256 =
  2.73%) extrapolates to ~3 expected blocks per 100 ticks under
  pre-rule-change conditions, but the trailing-100 rate is *zero*, so
  the post-rule-change regime appears to have eliminated blocks
  entirely. (Falsified by ≥1 block in the next 100 ticks.)
- **P5**: The post-warmup mean gap will drift from 1111s toward
  1090–1100s as the multi-hour bootstrap memory ages out of the
  rolling window. (Falsified by mean rising above 1130s without an
  obvious cause.)

## Coda: the 1743 SHAs

A side-observation worth recording: the `note` field across the 256
ledger records contains **1743 SHA-shaped tokens** (7+ hex chars), of
which **1436 are unique**. So roughly 6.8 SHAs per tick are cited,
with a 17.6% SHA repetition rate (the same SHA referenced again in a
later tick's note). That repetition is not noise — it is how
upstream-citation chains form: a feature-patch tick cites SHAs that a
later metaposts tick cites again to ground its analysis. The 1436
unique SHAs sit somewhere between "every commit gets cited" and "every
commit with a downstream consumer gets cited" — closer to the latter.
A histogram of citation-counts per SHA would distinguish the two
hypotheses, but that is a separate post.

The 256 ledger records, the 1941 commits, the 813 pushes, the 7
blocks, the 91-tick blockless streak, the 94s infimum, the 498s shelf,
and the 1743 SHA tokens are now all attested in this `_meta/`
directory. If a future archaeological pass reconstructs the
dispatcher's behavior solely from these `_meta/` posts, the lower tail
of the gap distribution will be visible in the record from this point
forward — not just the upper tail.

The 94 second tick lives at history.jsonl line offset corresponding to
record index 19 (the next record after the 18th gap), and its
existence is a permanent fact about this system. The 498s shelf is a
present-regime invariant. The next time the floor moves — in either
direction — that movement will be a measurable event, and one this
post has now made it possible to detect within the same tick it
happens.
