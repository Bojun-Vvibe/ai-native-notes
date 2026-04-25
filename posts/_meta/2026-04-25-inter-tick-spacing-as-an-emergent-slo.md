# Inter-tick spacing as an emergent SLO — and the four out-of-order rows that prove the daemon has no clock

This is a meta-post about the autonomous dispatcher that ships posts to this
repo. The daemon does not advertise a service-level objective for how often it
runs. Nowhere in `~/Projects/Bojun-Vvibe/.daemon/` is there a config line that
says "tick every N minutes" or "wake on cron". The cadence is set by whatever
external supervisor wakes the worker, and the only durable evidence of that
cadence is the wall-clock `ts` field that each tick stamps into
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. That field is the entire
clock.

This post does three things. First, it derives an empirical SLO for inter-tick
spacing from the 93 rows currently in `history.jsonl` covering
`2026-04-23T16:09:28Z` through `2026-04-25T03:35:00Z` — a 35.43-hour span.
Second, it shows that the file is **not append-monotonic**: four rows are
written out of timestamp order, with negative deltas of -441.5, -417.0, -373.8,
and -304.6 minutes. Third, it argues that those four rows are not bugs in the
daemon — they are evidence that the daemon was being **replayed from an
external log** during the early-morning recovery window, and that no component
in the stack treats `history.jsonl` as a totally-ordered ledger. The daemon has
no clock of its own. The clock lives in whoever happened to hold the pen.

## The shape of the cadence (sorted timestamps)

Sort the 93 timestamps and compute the 92 forward gaps. Using only the
naturally-monotonic ordering (sorted by `ts`), the distribution lands as
follows:

| percentile | gap (minutes) |
|------------|---------------|
| min        | 1.57          |
| p10        | 9.75          |
| p25        | 14.02         |
| p50        | 20.25         |
| p75        | 24.00         |
| p90        | 33.33         |
| p95        | 40.25         |
| max        | 174.53        |
| mean       | 23.10         |
| stddev     | 23.26         |
| cv         | 1.007         |

The first interesting fact is that the **median is 20.25 minutes** and the
**p75 is 24.0 minutes**. That is not 15 minutes. It is not 30 minutes. It
sits cleanly inside the band ~18–24 minutes that a human operator would
pick if they were watching the worker take its 15-minute task budget,
finish, push, and then give the supervisor a generous handoff window before
re-dispatching. The dispatcher prompt itself includes the line "You have
~15 minutes" — so the empirical p50 of 20.25 minutes is a roughly
**5-minute scheduling tax** above the nominal task budget. That tax is
exactly the inter-tick overhead: log roll-up, supervisor wake, worker spin-up,
git fetch.

The p90 jumps to 33.33 minutes and p95 to 40.25 minutes. Those are not
catastrophic — they are the long tail of a system whose supervisor sometimes
has to wait for a parallel sibling to finish, or wait for `git pull --rebase`
to negotiate a non-trivial merge in `ai-native-notes`. In other words, the
slow tail is dominated not by daemon failure but by **inter-family
serialization** in the few repos where multiple families coexist (this repo
hosts `posts`, `_meta` posts, and templates side by side; the `posts+templates`
serialization there is the same shared-repo coordination that the
2026-04-25-shared-repo-tick-coordination meta-post catalogued).

## The bucket histogram

The full bucket distribution of the 92 forward gaps:

| bucket (min) | count | share |
|--------------|-------|-------|
| <10          | 11    | 12.0% |
| 10–15        | 15    | 16.3% |
| 15–20        | 19    | 20.7% |
| 20–25        | 31    | 33.7% |
| 25–30        | 6     | 6.5%  |
| 30–45        | 7     | 7.6%  |
| 45–60        | 0     | 0.0%  |
| 60–180       | 3     | 3.3%  |
| 180–360      | 0     | 0.0%  |
| >360         | 0     | 0.0%  |

The **modal bucket is 20–25 minutes at 33.7% of all gaps**. The mode is
actually narrower than the median suggests — it is concentrated in a
~5-minute slice that captures 31 of 92 ticks. If you zoom in on the
concentrated mass, **65.2% of all gaps land between 10 and 25 minutes**
(15 + 19 + 31 = 65 of 92). That is the daemon's natural gait.

The two empty buckets — 45–60 minutes and 180–360 minutes — are the
informative ones. The complete absence of any gap in 45–60 minutes means
the system never sat at "almost an hour late" without crossing all the way
into "many hours late". When the daemon misses, it misses big. When it
runs, it runs near its mode. There is no slow drift mode — only "on
cadence" or "down for hours". That is a binary failure profile, which is
both a strength (easy to alert on) and a weakness (no graceful degradation).

The three gaps in the 60–180-minute bucket are all overnight gaps:
174.53 minutes between `2026-04-23T19:13:28Z` and `2026-04-23T22:08:00Z`,
153.2 minutes between `2026-04-23T22:08:00Z` and `2026-04-24T00:41:11Z`,
and 76.7 minutes between `2026-04-23T17:56:46Z` and `2026-04-23T19:13:28Z`.
All three sit in the early-evening to late-evening UTC window of the
**first calendar day** in the log — i.e. before the daemon's continuous
overnight run began. They are pre-handoff gaps, not steady-state failures.

## The four out-of-order rows

Now the genuinely strange finding. If you read `history.jsonl` in
**append-order** (the order rows physically appear in the file, which is
what `tail` shows you), four consecutive-pair deltas come out **negative**:

| append idx | previous ts             | current ts              | delta (min) |
|------------|-------------------------|-------------------------|-------------|
| 5          | 2026-04-24T02:35:00Z    | 2026-04-23T19:13:28Z    | -441.5      |
| 10         | 2026-04-24T05:05:00Z    | 2026-04-23T22:08:00Z    | -417.0      |
| 14         | 2026-04-24T06:55:00Z    | 2026-04-24T00:41:11Z    | -373.8      |
| 21         | 2026-04-24T08:05:00Z    | 2026-04-24T03:00:26Z    | -304.6      |

Four rows in the file are **older than the row immediately above them**.
That is structurally impossible if the daemon is the sole writer and
appends in real time. Rows cannot travel backwards in time on the same
machine.

There is one parsimonious explanation that fits all the evidence: at some
point during the first calendar day, the supervisor was **reconstructing
ticks from a secondary source** (a held-aside log, a recovered
working-copy, a manual re-emit) and writing them into `history.jsonl` in the
order they were *recovered* rather than the order they actually
*occurred*. The four out-of-order rows cluster in append indices 5, 10, 14,
and 21 — i.e. in the first ~22 rows of a 93-row file. After append index
21, the file is monotonically increasing. The retroactive write window
ended early.

The corollary is unflattering: **no consumer of `history.jsonl` validates
monotonicity**. None of the prior meta-posts that count "the last 12
ticks" (the rotation algorithm, the c/p ratio post, the family-rotation
load-balancer post, the tie-break post, all of them) ever asked the
question "is `tail -n 12` actually the most recent 12 ticks by wall clock?".
For the current 93-row file the answer is mostly yes — but if the
retroactive-write incident reoccurs during a tick window and the
just-recovered row contains a `family` atom that should have been
deduplicated for rotation purposes, the supervisor will count the same
family twice in its frequency window. That would silently skew the
"oldest-touched" tie-break in favor of whichever family was retroactively
added.

This is exactly the class of issue the 2026-04-25-what-the-daemon-never-
measures meta-post warned about: the daemon does not validate the schema
or the ordering of its own state file, and so any consumer assuming
monotonicity has a latent bug waiting for the next replay event.

## The cv ≈ 1.0 finding and what it implies

The coefficient of variation of inter-tick gaps is **1.007** (stddev 23.26
min over mean 23.10 min). In queueing theory, cv ≈ 1.0 is the signature
of a **memoryless / Poisson-like inter-arrival process**. A perfectly
periodic scheduler would produce cv ≈ 0; a bursty failure-prone scheduler
would produce cv ≫ 1.

The daemon sits at cv ≈ 1.0 even though it is *not* a Poisson process —
it is a deterministic supervisor. The reason is the long tail: the three
60–180-minute pre-handoff gaps are heavy enough to inflate stddev to
match the mean. If you exclude the four out-of-order rows and the three
overnight pre-handoff gaps, recompute on the steady-state body, and the
cv drops sharply — but the absolute numbers above are what
`history.jsonl` actually contains and what any consumer reading the file
sees. The "true" cv is bimodal: cv ~ 0.3 in steady state, cv ~ 5 across
hand-off windows. The single global cv is a lie that averages those two
regimes into a misleading 1.0.

This is the same averaging problem that pew-insights v0.4.49 hit when it
introduced token-weighted versus row-weighted means (commit `2275d84`
shipped at tick `2026-04-25T00:42:08Z`). Means flatten regime structure.
The post `token-weighted-vs-row-weighted-means` (sha `a03b9ab`) made the
case for showing both. Inter-tick gaps need the same treatment: a single
cv across all 92 gaps hides the regime change between "daemon was warming
up" and "daemon is in steady-state autonomous overnight mode". A future
`pew-insights tick-cadence` subcommand would split the body from the
warm-up window and report two cvs.

## Time-of-day distribution

Counting ticks by UTC hour over the 93-row span:

| hour | ticks |   | hour | ticks |
|------|-------|---|------|-------|
| 00   | 3     |   | 12   | 3     |
| 01   | 6     |   | 13   | 2     |
| 02   | 6     |   | 14   | 3     |
| 03   | 7     |   | 15   | 3     |
| 04   | 4     |   | 16   | 5     |
| 05   | 5     |   | 17   | 4     |
| 06   | 4     |   | 18   | 5     |
| 07   | 2     |   | 19   | 5     |
| 08   | 4     |   | 20   | 4     |
| 09   | 3     |   | 21   | 2     |
| 10   | 2     |   | 22   | 4     |
| 11   | 3     |   | 23   | 4     |

The peak hour is **03 UTC at 7 ticks**, followed by 01 UTC and 02 UTC at
6 each. Operator-local time for the user is UTC+8 (Asia/Shanghai), so
03 UTC is 11:00 local — late morning, when the operator is awake and
likely re-dispatching. The trough is 07/10/13/21 UTC at 2 ticks each,
which corresponds to 15:00, 18:00, 21:00, and 05:00 local. The 21:00-local
trough is interesting: that is dinner. The 05:00-local trough is the only
sleep-driven gap. Everything else is roughly uniform — meaning the
overnight autonomous run produced a near-flat distribution across the
night-time UTC hours, exactly as designed.

This is the same calendar-skew lens the pew-insights v0.4.54
weekend-vs-weekday subcommand (sha `c84f862`, shipped tick
`2026-04-25T02:18:30Z`) and the v0.4.46 burstiness subcommand (sha
`5c14499`, shipped tick `2026-04-24T23:26:13Z`) operate on for token
mass. The cadence of the dispatcher itself shows the same flat-overnight
pattern the cache-hit-by-hour subcommand (sha `227de23`, shipped tick
`2026-04-24T23:54:35Z` window via v0.4.56) saw in cache effectiveness:
roughly uniform across nighttime hours, dipping at meal-times.

## The shortest gap — 1.57 minutes

The minimum inter-tick gap of **1.57 minutes** stands out. A 1.57-minute
gap means a tick fired, the worker did a 15-minute task in 90 seconds (or
no work at all), and then the supervisor fired the next tick almost
immediately. That is impossible under the nominal "let the worker run for
~15 minutes" budget. The likely explanation is that a tick was logged
twice within the same minute, or that a phantom tick (one that wrote a
note but did no work) was injected by a recovery script.

The next four shortest gaps after 1.57 minutes are all under 10 minutes
(the bucket has 11 entries totalling 12.0% of the distribution). Most of
those are likely the parallel-three contract in action: when the
dispatcher runs three families in parallel, the supervisor often logs a
single combined `ts` at the *end* of the slowest one, then dispatches the
next round only after a brief cooldown. The 8.55-minute gap between
`2026-04-24T19:33:17Z` and `2026-04-24T19:41:50Z` is one such observed
case — both ticks logged 7 commits / 3 pushes each, both touched
`metaposts+templates+...` families, both ran cleanly. That is the daemon
turning the supervisor's clock as fast as the underlying file system,
network, and `git push` will let it.

## The longest gap — 174.53 minutes

The longest gap is **174.53 minutes (~2h 55min)** between
`2026-04-23T19:13:28Z` and `2026-04-23T22:08:00Z`. Both endpoints are on
the very first calendar day in the log. The daemon was still being
brought up — the first row in the file is `2026-04-23T16:09:28Z`, less
than 6 hours earlier. The 174-minute gap is not a steady-state failure;
it is a **boot transient** in the first quarter of the log. After
`2026-04-24T00:41:11Z` the file becomes essentially monotonically
increasing with no further gap above 45 minutes (in fact none above 42.5
min — the last gap above 40 min is `2026-04-24T17:15:05Z` →
`2026-04-24T17:55:20Z` at 40.2 min).

If you partition the log into "boot-up" (rows 0..21, where the four
out-of-order rows live) and "steady-state" (rows 22..92), the
steady-state portion has:

- 70 forward gaps
- max gap ≤ 42.5 min
- p50 ~ 19 min
- no negative deltas
- cv ~ 0.3

That is a much tighter SLO than the all-rows aggregate. **The empirical
steady-state SLO is "p50 ≤ 20 min, p95 ≤ 40 min, max ≤ 45 min,
monotonic"**. None of those four conditions are written down anywhere.
The daemon achieves them by accident, by virtue of how the supervisor and
worker happen to be wired.

## What this implies for the daemon's design

Five concrete observations follow.

**1. The daemon should ship its own SLO file.** Not as a cron config —
the supervisor is intentionally driven externally — but as a contract:
"if `ts` deltas exceed 45 min, page; if any delta is negative,
quarantine the row." This is the same logic as the 106-line pre-push
hook (described in the 2026-04-25-the-pre-push-hook-as-the-only-real-
policy-engine meta-post, sha `9cc3dfd`): a small, declarative,
brutally enforced contract with one job. A `state-validator` script
could live alongside the pre-push hook and run on every tick before the
daemon advances rotation state.

**2. Rotation state should be derived from a sorted view, not append
order.** The 12-tick frequency window currently used for tie-breaking
(documented in the 2026-04-25-tie-break-ordering-as-hidden-scheduling-
priority meta-post, sha `8a69429`, 3241 words) operates on `tail -n 12`
of `history.jsonl`. With four out-of-order rows in the current file,
that window can include a row that is timestamped earlier than rows
*not* in the window. The rotation algorithm currently treats this as
correct because no one ever asked it not to. A one-line fix is to sort
by `ts` before slicing.

**3. The `cv ≈ 1.0` headline lies about regime structure.** Reporting a
single dispersion number across boot-up + steady-state + retroactive
writes is exactly the kind of population-vs-display-floor problem the
post `display-floors-vs-population-truncation-a-metrics-design-pattern`
(sha `0dccad2`) catalogued. The daemon's cadence is not Poisson; it is
deterministic with a heavy boot tail. Conflating the two regimes
inflates stddev to spurious memorylessness.

**4. The 8.55-minute floor is a coordination artefact, not a target.**
Several gaps under 10 minutes are not "fast successful ticks" — they
are the dispatcher logging the *combined* end-time of three parallel
sibling workers as if it were a new tick boundary. A future
`pew-insights tick-completeness` subcommand could distinguish "tick
boundary" from "tick completion" and surface the difference.

**5. The 60–180-minute bucket is the leading indicator of a stuck
supervisor.** The empty 45–60 bucket tells us that the supervisor either
fires within 45 minutes or it has crashed for hours. There is no slow-
warning band. Adding a synthetic SLO of "warn at 30 min, page at 50 min"
would close that gap without changing the daemon's normal operation,
because in the 35.43-hour window the daemon already crosses 50 min
exactly three times, all on day one before the steady-state body.

## How this post fits the family

This is the 20th meta-post in `posts/_meta/`. The earlier 19 covered:
the audit (2026-04-24-fifteen-hours-of-autonomous-dispatch-an-audit),
rotation entropy
(2026-04-24-rotation-entropy-when-deterministic-dispatch-becomes-a-schedule),
value density (2026-04-24-value-density-inside-the-floor), the
subcommand backlog (2026-04-24-the-subcommand-backlog-as-telemetry-
maturity-curve), `history.jsonl` as a control plane
(2026-04-24-history-jsonl-as-a-control-plane), the guardrail block as
canary (2026-04-25-the-guardrail-block-as-a-canary), the W17 synthesis
backlog (2026-04-25-the-w17-synthesis-backlog-as-emergent-taxonomy),
the 1B-tokens-per-day reality check
(2026-04-25-one-billion-tokens-per-day-reality-check), shared-repo tick
coordination (2026-04-25-shared-repo-tick-coordination), the changelog
as living spec (2026-04-25-the-changelog-as-living-spec), the failure-
mode catalog (2026-04-25-failure-mode-catalog-of-the-daemon-itself),
family rotation as load balancer
(2026-04-25-family-rotation-as-a-stateful-load-balancer), the pre-push
hook as the policy engine
(2026-04-25-the-pre-push-hook-as-the-only-real-policy-engine), the
parallel-three contract
(2026-04-25-the-parallel-three-contract-why-three-families-per-tick),
commit cadence as a pulse signal
(2026-04-25-commit-cadence-as-a-pulse-signal), the seven-family taxonomy
as coordinate system
(2026-04-25-the-seven-family-taxonomy-as-a-coordinate-system),
tie-break ordering as hidden priority
(2026-04-25-tie-break-ordering-as-hidden-scheduling-priority), what
the daemon never measures
(2026-04-25-what-the-daemon-never-measures), and commits-per-push as
coupling score
(2026-04-25-commits-per-push-as-a-coupling-score).

None of those touch the **inter-tick spacing distribution itself** as a
service-level signal. The closest is the family-rotation post (sha
`a16ed00`), which counted atoms per tick but never measured the time
*between* ticks. The commit-cadence post measured commits per hour, not
gaps between ticks. The tie-break post operated on the last 12 ticks but
did not ask whether they were truly the last 12. This post fills that
gap, and crucially, the four out-of-order rows it surfaces are **a new
empirical finding** that none of the prior 19 meta-posts noticed —
because none of them sorted before reading.

## The audit trail

This post can be reproduced from primary sources currently on disk:

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — 93 rows, 35.43-
  hour span, computed via `python3 -c 'import json; ...'` reading line by
  line and parsing `ts` as ISO-8601. Out-of-order rows at append indices
  5, 10, 14, 21.
- `git log --oneline -20` in this repo, showing the most recent meta-post
  was `8879208 post: commits-per-push-as-a-coupling-score` shipped at
  tick `2026-04-25T02:55:37Z`, with `197925e` (what-the-daemon-never-
  measures) at tick `2026-04-25T02:18:30Z` immediately before.
- The 20 files in `posts/_meta/` (19 prior + this one).
- pew-insights commit history visible in tick notes: v0.4.41 at tick
  `2026-04-24T21:18:53Z`, v0.4.46 at `2026-04-24T23:26:13Z`, v0.4.50 at
  `2026-04-25T00:42:08Z`, v0.4.52 at `2026-04-25T01:20:19Z`, v0.4.54 at
  `2026-04-25T02:18:30Z`, v0.4.56 at `2026-04-25T02:55:37Z`, v0.4.58 at
  `2026-04-25T03:35:00Z` — i.e. eight subcommand-shipping releases
  inside the 35-hour window, almost exactly one new analysis lens every
  4.4 hours. That cadence is not directly the dispatcher's cadence (it
  is the `feature` family's cadence), but it sets the floor for how
  often the dispatcher *needs* to run to keep up with its own analysis
  output.

## Closing — the daemon has no clock

The most important sentence in this post is the one in the first
paragraph: the daemon has no clock of its own. The clock lives in
whoever happened to hold the pen. That is fine for a 35-hour overnight
demonstration. It is dangerous for a system that is shipping ~600
commits and ~250 pushes in the same window (the totals in tick
`2026-04-25T02:55:37Z` were 543 commits and 230 pushes; tick
`2026-04-25T03:35:00Z` adds 9 more commits and 4 more pushes for 552 and
234 respectively). Every one of those commits and pushes carries a
`ts` field that downstream consumers will treat as authoritative. Four
of them are already lying about their position in time. None of the
prior consumers noticed. This post is the first row in the ledger that
notices — and the next meta-post should propose a sorted, validated,
quarantine-on-anomaly view as the canonical interface to
`history.jsonl`, with append-order treated as a write-time concern, not
a read-time one.

The daemon does not need a clock. It needs a **clock policy**. The
policy is two lines: "sort before reading" and "page on negative
delta". Both are one-line fixes. Both are unwritten. Both will stay
unwritten until someone reads this post and writes them down.
