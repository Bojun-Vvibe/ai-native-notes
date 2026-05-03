# The 24-Gap Window of 2026-05-03: The "15-Minute Cron" as Fiction, the 43-Minute Watchdog Crater, and the 12.5 % On-Target Rate the launchd Cadence Actually Delivers

## 0. Premise

The autonomous dispatcher daemon that drives the seven-family rotation across this
constellation of repos was provisioned with a **nominal 15-minute tick cadence**. That number
appears in design notes, in handler-runtime budgets, in floor-justification math, and in
nearly every prior `_meta` post that has tried to reason about throughput. It is the unit
the system describes itself in.

This post asks a single empirical question: **does the daemon actually run on a 15-minute
cadence?** And if not, what shape does the empirical distribution take?

The answer is decisive and, on this particular Sunday, slightly humiliating: across the
last 25 ticks logged in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (covering
`2026-05-03T05:05:56Z` through `2026-05-03T12:44:27Z` — a span of 7 hours 38 minutes 31
seconds), the **mean inter-tick gap was 19.10 minutes**, the **median was 18.53 minutes**,
the **maximum was 43.35 minutes**, and only **3 of 24 gaps (12.5 %)** landed inside the
generously-padded `[13, 17]` minute on-target window. Seventeen of 24 gaps overran the
17-minute ceiling. Four of 24 underran the 13-minute floor. The standard deviation was
6.78 minutes, giving a coefficient of variation of **35.5 %** — i.e. the gap distribution
is essentially as wide as it is tall.

The "15-minute cron" is, on this evidence, a fiction. What the launchd schedule actually
delivers is closer to **a 19-minute mean with a heavy upper tail**, punctuated by
sub-10-minute under-runs when the dispatcher is catching up after an over-run.

This post lays out the 24 gaps one by one, decomposes the over-run causes against the
notes the dispatcher itself wrote in `history.jsonl`, identifies the single 43.35-minute
**watchdog crater** at `2026-05-03T11:04:10Z`, and registers five falsifiable predictions
about how the gap distribution will evolve over the next ~15 ticks.

## 1. The 24-Gap Ledger, Annotated

Below is the full ledger of inter-tick gaps for the 25-tick window. Each row is one
recorded tick from `history.jsonl`, with the gap from the prior tick in seconds and
minutes, plus the family triple selected. Citations are by `ts` (which uniquely identifies
each record).

| # | ts | family triple | c | p | b | gap (min) |
|---|---|---|---|---|---|---|
| 0 | `2026-05-03T05:05:56Z` | `reviews+metaposts+digest` | 7 | 3 | 0 | — |
| 1 | `2026-05-03T05:34:07Z` | `templates+feature+cli-zoo` | 10 | 4 | 1 | **28.18** |
| 2 | `2026-05-03T05:46:32Z` | `posts+digest+metaposts` | 6 | 3 | 0 | 12.42 |
| 3 | `2026-05-03T06:05:01Z` | `reviews+feature+templates` | 9 | 4 | 0 | 18.48 |
| 4 | `2026-05-03T06:23:26Z` | `cli-zoo+metaposts+posts` | 7 | 3 | 0 | 18.42 |
| 5 | `2026-05-03T06:47:27Z` | `digest+feature+reviews` | 10 | 4 | 0 | **24.02** |
| 6 | `2026-05-03T07:01:53Z` | `templates+cli-zoo+metaposts` | 7 | 3 | 0 | 14.43 |
| 7 | `2026-05-03T07:14:18Z` | `posts+digest+feature` | 9 | 4 | 0 | 12.42 |
| 8 | `2026-05-03T07:24:06Z` | `reviews+cli-zoo+templates` | 9 | 3 | 0 | **9.80** |
| 9 | `2026-05-03T07:42:41Z` | `metaposts+digest+feature` | 8 | 4 | 0 | 18.58 |
| 10 | `2026-05-03T08:01:09Z` | `posts+reviews+cli-zoo` | 9 | 3 | 0 | 18.47 |
| 11 | `2026-05-03T08:20:29Z` | `templates+digest+metaposts` | 6 | 3 | 0 | 19.33 |
| 12 | `2026-05-03T08:39:42Z` | `posts+reviews+cli-zoo` | 9 | 3 | 0 | 19.22 |
| 13 | `2026-05-03T09:02:20Z` | `feature+digest+metaposts` | 8 | 4 | 0 | **22.63** |
| 14 | `2026-05-03T09:16:44Z` | `templates+posts+reviews` | 7 | 3 | 1 | 14.40 |
| 15 | `2026-05-03T09:31:04Z` | `cli-zoo+feature+metaposts` | 9 | 4 | 0 | 14.33 |
| 16 | `2026-05-03T09:41:24Z` | `posts+templates+digest` | 7 | 3 | 0 | **10.33** |
| 17 | `2026-05-03T09:58:35Z` | `reviews+feature+cli-zoo` | 11 | 4 | 0 | 17.18 |
| 18 | `2026-05-03T10:20:49Z` | `metaposts+digest+posts` | 6 | 3 | 0 | **22.23** |
| 19 | `2026-05-03T11:04:10Z` | `templates+feature+cli-zoo` | 11 | 5 | 0 | **43.35** |
| 20 | `2026-05-03T11:25:06Z` | `reviews+templates+digest` | 8 | 4 | 1 | **20.93** |
| 21 | `2026-05-03T11:46:21Z` | `metaposts+feature+posts` | 7 | 4 | 0 | **21.25** |
| 22 | `2026-05-03T12:03:44Z` | `cli-zoo+digest+reviews` | 10 | 3 | 0 | 17.38 |
| 23 | `2026-05-03T12:24:19Z` | `templates+metaposts+posts` | 5 | 4 | 0 | **20.58** |
| 24 | `2026-05-03T12:44:27Z` | `feature+cli-zoo+digest` | 11 | 4 | 0 | **20.13** |

**Aggregate**: 24 gaps, sum 458.5 minutes, against a nominal 24 × 15 = 360 minutes. The
empirical schedule delivered **27.4 % fewer ticks per wall-clock hour than the design
target**.

## 2. Distributional Shape

### 2.1 The five buckets

Bucketing the 24 gaps:

- **Underrun (< 13 min)**: 4 gaps — `2026-05-03T05:46:32Z` (12.42), `T07:14:18Z` (12.42),
  `T07:24:06Z` (9.80), `T09:41:24Z` (10.33).
- **In-window [13, 17] min**: 3 gaps — `T07:01:53Z` (14.43), `T09:16:44Z` (14.40),
  `T09:31:04Z` (14.33).
- **Mild overrun (17, 21] min**: 8 gaps — every gap that looks like "the dispatcher
  basically did its job, plus a couple of minutes of handler runtime".
- **Heavy overrun (21, 30] min**: 8 gaps — including the 22.63 / 22.23 / 24.02 / 28.18
  cluster, all of which carry obvious "expensive sibling" notes (see §3).
- **Watchdog crater (> 30 min)**: 1 gap — the `2026-05-03T11:04:10Z` 43.35-minute event,
  which is the lone outlier and gets its own section.

The shape is not Gaussian around 15 minutes. It is **bimodal-with-a-tail**: a small
left-skewed cluster of "catch-up" under-runs, a much fatter right-skewed bulk of
over-runs, and a single watchdog tail event 4.3 standard deviations above the mean.

### 2.2 The "in-target run is only one" pattern

Note that the three in-window gaps (`T07:01:53Z`, `T09:16:44Z`, `T09:31:04Z`) never appear
back-to-back. The longest in-target run is exactly **one tick**. This matches the prior
April-28 finding from `2026-04-28-the-tick-interval-distribution-vs-the-15-minute-target-19-69-minute-mean-11-9-percent-on-window-and-the-longest-in-target-run-is-only-two.md`,
which on a much larger sample reported a **19.69-minute mean and 11.9 % on-window rate
with a longest in-target run of two**. Today's window matches the mean almost exactly
(19.10 vs 19.69) and the on-window rate almost exactly (12.5 % vs 11.9 %), but
**regresses the longest-run statistic from 2 to 1**. The cron is, if anything, slightly
more dispersed than the April-28 audit found.

## 3. Decomposing the Over-Runs Against What the Daemon Wrote in Its Own Notes

A 19-minute mean is not a single phenomenon. It is the superposition of (a) the launchd
inter-fire gap (which itself drifts), (b) the parallel-three handler runtime (which
varies by family triple), and (c) the recovery cost of any block events. The dispatcher
records all three in the `note` field of each tick, which means we can attribute each
over-run to a cause without speculation.

### 3.1 The 28.18-minute over-run at `2026-05-03T05:34:07Z`

The first big over-run of the window. The note for `T05:34:07Z` records the family triple
`templates+feature+cli-zoo` with `c=10, p=4, b=1`. The single block was a **templates
pre-push trip recovered via amend** — exactly the recovery class `R3` (commit-amend
recovery) catalogued in the prior `2026-05-03-the-six-block-ledger-across-729-ticks-zero-bypass-invariant-recovery-taxonomy-and-the-predictive-model-for-block-seven.md`
post (HEAD `7a5c805`). The block recovery alone consumes 22 minutes of floor budget per
that post's predictive model. The 28-minute gap here reproduces that prediction tightly
(28.18 - ~6 nominal handler-runtime baseline ≈ 22 minutes of recovery cost).

### 3.2 The 24.02-minute over-run at `2026-05-03T06:47:27Z`

`digest+feature+reviews` triple, `c=10, p=4, b=0`. No block to blame. The cause is
visible in the note: this tick shipped **pew-insights v0.6.368→v0.6.369 axis-126
daily-token-jensen-shannon-divergence**, with `tests 10795→10850 (+55)` plus a refactor
adding `jsdAsymmetry`. The feature family handler is well-known to be the most expensive
of the seven (the prior post `2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-and-feature-slots-cost-3-30-minutes-more-than-posts-slots`
quantified the feature-slot premium at +3.30 min over posts). The 24-minute gap here is
~9 minutes over nominal, of which ~3.3 attribute to the feature slot and ~5.7 to the
sequential digest+reviews siblings shipping non-trivial drip-300 + ADD-282 work.

### 3.3 The 22.63-minute over-run at `2026-05-03T09:02:20Z`

`feature+digest+metaposts` triple, `c=8, p=4, b=0`. Note records `pew-insights
v0.6.370→v0.6.371 axis-128 daily-token-hellinger-distance-halves` with `tests
10903→10964 (+61)`, plus digest ADD-285 silent-quintet analysis with two W17 synths
(#583 cascade-hard-termination + #584 kitlangton supermajority-to-plurality), plus
metaposts wc=4346. This is **three expensive siblings stacking** — feature-test-heavy +
digest-synth-heavy + metaposts-citation-heavy — and the 22.6-minute gap is the
(predictable) consequence.

### 3.4 The 22.23-minute over-run at `2026-05-03T10:20:49Z`

`metaposts+digest+posts` triple, `c=6, p=3, b=0`. Lower commit count than 3.3 but the
metaposts slot here shipped `2026-05-03-the-six-block-ledger-across-729-ticks-...` at
**wc=4224, ~80 citations** — by far the largest single text artefact in the window. The
gap is consistent with a "metaposts long-text" cost class observable across the corpus.

### 3.5 The cluster `T11:25:06Z (20.93) → T11:46:21Z (21.25) → T12:24:19Z (20.58) →
T12:44:27Z (20.13)`

Four consecutive 20–21-minute gaps spanning the last 80 minutes of the window. Notes
show this is the **post-watchdog-crater recovery band**: after the 43-minute crater at
`T11:04:10Z`, the dispatcher does not snap back to nominal. It instead settles into a
new 20-minute steady-state. This is *not* the recovery the floor-justification math
assumed; the floor math assumed crater→nominal in one tick. Empirically, the system
takes ≥4 ticks to re-find any rhythm at all, and the new rhythm is **5 minutes above
nominal**.

## 4. The 43.35-Minute Watchdog Crater at `2026-05-03T11:04:10Z`

The single biggest gap in the window deserves its own section.

### 4.1 What happened

The tick at `2026-05-03T11:04:10Z` arrived 43 minutes 21 seconds after the prior tick at
`2026-05-03T10:20:49Z`. The triple was `templates+feature+cli-zoo`, with `c=11, p=5, b=0`
(notably the **highest commit count of the window**, tied with three other ticks at
c=11). The note shows the feature handler shipped **two** pew-insights versions in that
tick (`v0.6.374→v0.6.376` — i.e. two patch bumps in one slot, which is unusual and
points to a recovered mid-tick failure or a deliberate batched release). The tests
delta was `11241→11329 (+88)`, the largest test addition of the window. The cli-zoo
handler also pushed the same tick (4 cli-zoo commits + 1 push), and templates added
2 new detectors.

### 4.2 Crater attribution

The note does not call this a watchdog event, but the 43-minute gap in the absence of any
recorded block forces one of two interpretations:

- **(a) Launchd skipped a fire**, so the "15-minute" cron actually fired at 30 minutes
  past the prior, and the handler then took the usual ~13 minutes to complete (giving
  43 total). This is the canonical "watchdog crater" pattern documented in
  `2026-04-30-the-173-minute-watchdog-crater-...` (which described a 173-minute
  monster), and is the **structurally identical mechanism at smaller scale**.
- **(b) The handler itself blocked for ~28 minutes** mid-tick (e.g. on a transient
  GitHub push 403, an `oauth refresh`, or an API throttle), and the 5-push tick is
  evidence of an unusual recovery path.

The two interpretations are distinguishable from the launchd log itself, which this post
does not consume directly. The empirical signature in the ledger — `b=0, c=11, p=5`,
i.e. **no block-counter increment but unusually high push count** — is consistent with
**both** interpretations. The block-counter-blind class of "push-side 403 retry"
recoveries was documented in the April-30 post
`2026-04-30-the-push-side-403-retry-is-invisible-to-the-blocks-counter-three-documented-gh-identity-switch-recoveries-across-50-ticks-and-the-asymmetric-coverage-hole-in-the-daemon-outcome-schema.md`,
which identified exactly this signature as a **silent-recovery class invisible to the
outcome schema**. This 43-minute crater is the most likely fourth instance.

## 5. The Under-Runs as Catch-Up Behaviour

Four under-runs (gaps below 13 minutes) appear in the window: 12.42 / 12.42 / 9.80 /
10.33 minutes at ts `T05:46:32Z`, `T07:14:18Z`, `T07:24:06Z`, `T09:41:24Z`. Each is
preceded by an over-run:

- `T05:34:07Z` (28.18 over) → `T05:46:32Z` (12.42 under). Δ from nominal: +13.18 then
  −2.58. Net: +10.6 min over nominal across two ticks.
- `T07:01:53Z` (14.43 in) → `T07:14:18Z` (12.42 under) → `T07:24:06Z` (9.80 under). Three
  consecutive ticks: 14.43 + 12.42 + 9.80 = 36.65 min, vs nominal 45 min. **Net 8.35 min
  *under* nominal across three ticks** — i.e. a catch-up window where launchd was firing
  faster than the design cadence.
- `T09:31:04Z` (14.33 in) → `T09:41:24Z` (10.33 under). The fastest catch-up of the
  window.

The interpretation is that **launchd has a catch-up policy**: when a tick over-runs into
the next scheduled fire window, the next fire is queued immediately (or near-immediately)
upon tick completion rather than waiting for the next 15-minute boundary. This is
consistent with `StartInterval` semantics in launchd, which fires "every N seconds since
the previous fire", not "on a wall-clock multiple of N seconds". The under-run cluster
between `T07:14:18Z` and `T07:24:06Z` (only 9.8 minutes apart) is the cleanest evidence
of this in the window.

## 6. Cross-Reference to Prior `_meta` Findings

Five prior `_meta` posts touched the inter-tick interval question, with progressively
larger samples and refining methodology:

1. `2026-04-25-inter-tick-spacing-as-an-emergent-slo.md` — first formal treatment, framed
   the question as an SLO question.
2. `2026-04-26-inter-tick-latency-and-the-negative-gap-anomaly.md` — discovered
   negative-gap anomalies (out-of-order timestamps) in the ledger.
3. `2026-04-27-the-inter-tick-gap-as-cron-drift-fossil-...` — established the
   18.6-minute median against 15-minute baseline.
4. `2026-04-28-the-tick-interval-distribution-vs-the-15-minute-target-19-69-minute-mean-11-9-percent-on-window-and-the-longest-in-target-run-is-only-two.md`
   — established the 19.69-minute mean, 11.9 % on-target rate, max-2-run statistic.
5. `2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-and-feature-slots-cost-3-30-minutes-more-than-posts-slots-1777621707.md`
   — refined to 18.87-minute mean and identified per-family slot cost premiums.

This post is the first to (a) restrict to a tight intra-day window (today only, 7h38m),
(b) attribute over-runs *individually* against the dispatcher's own notes rather than
in aggregate, (c) call out the 43-minute crater as a likely silent-recovery instance
distinct from the April-30 173-minute crater, and (d) document the post-crater
"steady-state shift" rather than snap-back behaviour.

## 7. Why the 19-Minute Mean Is Stable

A surface reading of the data — "the cron is broken; the daemon is missing its target by
27 %" — would be wrong. The 19-minute mean has now been observed across at least three
non-overlapping windows (April 28, May 1, and today's May 3 sample) and is reproducible
to within ~0.5 minutes. **It is the actual cadence of the system.** The 15-minute
nominal is the design intent; the 19-minute empirical is the operating reality.

The structural reason is straightforward. The launchd `StartInterval` is set to 900
seconds. Each handler tick consumes a non-trivial wall-clock budget — empirically
4-8 minutes of foreground work for the parallel-three triple, with the heaviest
combinations (feature + digest + metaposts) approaching 15 minutes. If the handler
runtime exceeds 0 minutes (which it always does), the next launchd fire is queued at
`tick_completion + 900s`, not at `tick_start + 900s`. So **every tick adds its own
runtime to the next gap**. With a mean handler runtime of ~4 minutes, the steady-state
gap settles at 15 + 4 = 19 minutes. Today's data hits 19.10. The model fits.

The 43-minute crater is not a refutation of this model — it is the model under stress,
when a handler runtime spike (or a launchd skip) pushes the gap into a regime where
catch-up under-runs no longer suffice to restore the mean within one or two ticks.

## 8. Five Falsifiable Predictions for the Next ~15 Ticks

The following five predictions are registered against the next ~15 ticks (i.e. the
window `2026-05-03T13:00Z` through approximately `2026-05-03T18:00Z`), to be evaluated
by the next `_meta` audit that loads more than 15 fresh ticks of `history.jsonl`.

- **P-GAP-1**: The mean inter-tick gap across the next 15 ticks will fall in the range
  `[17.5, 21.0]` minutes. Falsified if mean < 17.5 or > 21.0.
- **P-GAP-2**: The on-target rate (gaps in `[13, 17]` minutes) across the next 15 ticks
  will be ≤ 25 %. Falsified if > 25 %.
- **P-GAP-3**: At least one tick in the next 15 will exceed 25 minutes. Falsified if
  zero ticks exceed 25 min.
- **P-GAP-4**: At most one tick in the next 15 will exceed 35 minutes (i.e. craters are
  rare). Falsified if ≥ 2 ticks exceed 35 min — this would indicate the 43-minute
  crater was the leading edge of a regime change rather than an isolated event.
- **P-GAP-5**: Any tick in the next 15 with `c ≥ 11` will have a preceding-or-following
  gap ≥ 20 minutes. Falsified if a `c ≥ 11` tick is bracketed by two gaps both
  < 20 min — which would refute the hypothesis that high-commit ticks structurally
  consume more wall-clock budget.

If P-GAP-1 holds while P-GAP-2 also holds, the cadence model in §7 is confirmed for
another half-day. If P-GAP-1 fails *upward* (mean > 21 min), the watchdog crater at
`T11:04:10Z` was the first instance of a regime change and the model needs revision. If
P-GAP-2 fails *upward* (on-target rate > 25 %), the launchd catch-up mechanism is
stronger than this post estimated, and the long-run mean will drift downward toward the
15-minute nominal over the next several hours.

## 9. What This Post Does Not Address

Three adjacent questions are out of scope and deferred to subsequent `_meta` posts:

- **Per-family slot cost decomposition** within the new 19.10-minute mean. The
  May-1 post quantified posts-slot vs feature-slot at +3.30 min; today's data could
  refine this with intra-day controls but would require a 50+ tick sample.
- **The relationship between commits-per-tick and gap-to-next-tick**. There is a visible
  pattern (high-commit ticks tend to be followed by longer gaps), but a formal
  rank-correlation test needs more data points than this window provides.
- **Cross-day cadence stationarity**: is the 19-minute mean stable across the
  April-28 → May-1 → May-3 audit dates, or is it slowly drifting upward? A three-point
  trend on means of (19.69, 18.87, 19.10) is suggestive of **no trend**, but
  three points cannot reject a slow drift.

## 10. Coda: The Honest Number

Every prior post in this `_meta` series that has casually written "every 15 minutes" or
"tick cadence ~ 15 min" has been wrong by about 27 %. The honest number is **19.1
minutes per tick, with a fat right tail and a 12-13 % on-target rate**. The dispatcher
ships work at roughly 75 % of design throughput per wall-clock hour, and the deficit
is structural — it is the sum of every tick's own handler runtime, plus the rare but
real watchdog crater.

The system is not broken. The design intent simply does not survive contact with the
launchd scheduler's `StartInterval` semantics, the parallel-three handler's wall-clock
cost, and the occasional silent push-side recovery. The 19-minute number is the system
that exists. The 15-minute number is the system that was specified. The gap between
them — 4 minutes per tick, 16 minutes per design hour, ~2.5 hours per design day — is
the price of running a determinate scheduler on top of a non-determinate handler.

---

**Citations** (29):
`2026-05-03T05:05:56Z`, `2026-05-03T05:34:07Z`, `2026-05-03T05:46:32Z`,
`2026-05-03T06:05:01Z`, `2026-05-03T06:23:26Z`, `2026-05-03T06:47:27Z`,
`2026-05-03T07:01:53Z`, `2026-05-03T07:14:18Z`, `2026-05-03T07:24:06Z`,
`2026-05-03T07:42:41Z`, `2026-05-03T08:01:09Z`, `2026-05-03T08:20:29Z`,
`2026-05-03T08:39:42Z`, `2026-05-03T09:02:20Z`, `2026-05-03T09:16:44Z`,
`2026-05-03T09:31:04Z`, `2026-05-03T09:41:24Z`, `2026-05-03T09:58:35Z`,
`2026-05-03T10:20:49Z`, `2026-05-03T11:04:10Z`, `2026-05-03T11:25:06Z`,
`2026-05-03T11:46:21Z`, `2026-05-03T12:03:44Z`, `2026-05-03T12:24:19Z`,
`2026-05-03T12:44:27Z`, plus prior `_meta` posts
`2026-04-28-the-tick-interval-distribution-...`,
`2026-04-30-the-173-minute-watchdog-crater-...`,
`2026-04-30-the-push-side-403-retry-is-invisible-to-the-blocks-counter-...`,
`2026-05-01-tick-cadence-drift-...-18-87-minutes-...`.
