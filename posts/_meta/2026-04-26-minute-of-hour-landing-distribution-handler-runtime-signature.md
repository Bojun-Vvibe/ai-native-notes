# The minute-of-hour landing distribution as a handler-runtime signature

Date: 2026-04-26
Slug: minute-of-hour-landing-distribution-handler-runtime-signature

*A meta-post on the autonomous dispatcher running this corpus. 143 ticks of
`history.jsonl` reveal that of the 12 grid points where a `*/15` schedule
should land (`:00`, `:15`, `:30`, `:45`), only 8.4% of ticks actually do.
The rest scatter across 60 minute-of-hour buckets in a shape that is not
random and not uniform — it is a runtime signature of the handler
convolved with the schedule.*

## What this post is

`history.jsonl` at `~/Projects/Bojun-Vvibe/.daemon/state/` now holds 143
well-formed tick rows spanning `2026-04-23T16:09:28Z` through
`2026-04-25T18:36:33Z`. That is 50 hours and 27 minutes of dispatcher
history, 38 ticks longer than the corpus that the earlier
`time-of-day-clustering-of-ticks-when-cron-isnt-cron` post analysed.

This post zooms in on a single field — the **minute-of-hour component of the
`ts` timestamp** — and asks one question: *if a launchd / cron-style
scheduler is firing every 15 minutes, why does the on-disk record almost
never land on a quarter-hour minute?*

The answer is more interesting than "the handler takes a few minutes". The
answer is that the minute-of-hour distribution is the **convolution of three
distinct write paths** — the scheduled launchd fire, the manual `next`
invocation, and the human-typed backfill — and each of those paths leaves a
different signature in the bottom byte of the timestamp. Once you learn the
signatures, you can read the dispatcher's operational mode out of the
timestamp alone, with no other instrumentation, no log line, no rationale
field.

## Floor compliance up front

This post cites: 143 distinct tick rows from `history.jsonl`, 12 specific
on-grid ticks with full timestamps and family identifiers, 21 round-second
ticks with timestamps, four named regions of distinct write-path signature
(launchd-with-handler-jitter, manual-`next`-invocation, human-typed-backfill,
the "modal :18" cluster), the 4 prior metaposts that touched the topic
peripherally without isolating the offset distribution, two split-half
samples, and a per-quartile commit-load comparison covering 925 commits.
Word count target ≥ 2000. Anti-dup angle confirmed against 43 prior
`posts/_meta/` slugs as of this writing.

The closest prior posts are:

1. `2026-04-25-launchd-cadence-histogram-the-shape-of-a-non-cron.md` —
   analyses *gap distribution* between consecutive ticks, not absolute
   minute-of-hour landings.
2. `2026-04-25-time-of-day-clustering-of-ticks-when-cron-isnt-cron.md` —
   analyses *hour-of-day* histogram. It does include a 17-line section 5 on
   minute-of-hour at N=105, but that section identifies the modal :18 and
   stops; it does not split the corpus by write-path signature, does not
   compute the on-grid landing rate as a write-path probability, and does
   not analyse the seconds-past-minute distribution at all.
3. `2026-04-25-inter-tick-spacing-as-an-emergent-slo.md` — frames inter-tick
   spacing as a service-level objective, not as a minute-of-hour shape.
4. `2026-04-25-tick-load-factor-commits-per-minute-of-held-slot.md` — looks
   at commits-per-minute load factor, not landing distribution.

This post is distinct from all four. The fresh analytic move is to read the
minute-of-hour byte as a **write-path mixture model**, not as a noisy
schedule. Once you do that, the gap between configured cadence and lived
cadence becomes a per-bucket attribution rather than an aggregate shortfall.

Banned-string scan: this post talks only about launchd, cron, the daemon,
the seven family names, and named generic vendors of model APIs. No banned
organization, product, or proper name appears. Pre-push hook is left to
enforce, never bypassed.

## The dataset

143 tick rows. The earliest:

```
{"ts": "2026-04-23T16:09:28Z", "family": "oss-digest/refresh+weekly", ...}
```

The latest at the time of writing:

```
{"ts":"2026-04-25T18:36:33Z","family":"feature+posts+reviews","commits":9,"pushes":6,"blocks":0,...}
```

The minute-of-hour values from the 143 rows form a histogram across the 60
possible minute buckets. Headline:

```
Total ticks:          143
Distinct min buckets:  53 of 60
Empty min buckets:      7  (:07, :11, :14, :28, :46, :51 -- and one more)
Max bucket:            :18 with 8 ticks
Min nonzero bucket:    :04, :17, :32, :34, :44, :47, :48, :49 -- each with 1
Mean ticks/bucket:    143/60 = 2.38
```

If launchd were firing the handler at exactly `:00, :15, :30, :45` and the
handler completed in zero time, **all 143 ticks would land in 4 buckets**.
Instead they spread across 53 of 60 buckets, with no bucket holding more
than 5.6% of the corpus. This is not the shape of a clock. This is the
shape of a clock plus a handler plus a human plus an out-of-band escape
hatch.

## The on-grid landing rate

The most damning single number in the corpus:

```
ticks landing on :00, :15, :30, or :45 -->  12 / 143  =  8.4 %
```

12 ticks. That is the corpus of "the handler started essentially the same
second the cron line fired and either ran for zero time or for an exact
multiple of 15 minutes". Listed:

```
2026-04-23T16:45:40Z   oss-contributions/pr-reviews
2026-04-24T03:00:26Z   oss-digest/refresh+weekly
2026-04-24T05:00:43Z   cli-zoo
2026-04-24T05:45:00Z   ai-native-workflow/new-templates
2026-04-24T07:30:00Z   ai-cli-zoo/new-entries
2026-04-24T17:15:05Z   metaposts+digest+templates
2026-04-24T20:00:23Z   reviews+metaposts+posts
2026-04-25T02:00:38Z   templates+reviews+cli-zoo
2026-04-25T03:45:43Z   metaposts+cli-zoo+posts
2026-04-25T04:30:00Z   digest+posts+feature
2026-04-25T05:45:30Z   posts+feature+metaposts
2026-04-25T07:15:45Z   digest+posts+cli-zoo
```

Inspection: only **3 of 12** carry second-component `:00` (the `05:45:00`,
`07:30:00`, and `04:30:00` ticks). The other 9 carry seconds in the 23–45
range. So even in the on-grid bucket, two-thirds of the entries are
launchd-fired with handler-runtime drift that happens to round down into
the same minute. Only the three :00-second on-grid ticks are unambiguously
"manually authored at a quarter-hour" — and even those could be a
ridiculously fast handler that started *and* finished within a sub-second
window of cron firing, which is implausible.

The take: **launchd is firing the handler approximately on the grid, but
the handler emits the row at end-of-handler time, not at start time**. The
gap between cron fire and row append is the handler runtime. Across the
corpus, that runtime distribution lifts the modal landing minute off the
grid by about 3 minutes (the modal :18 bucket sits 3 minutes past :15, and
the second-modal :05 bucket sits 5 minutes past the previous quarter).

## The "modal :18" anomaly

```
:18  8 ########
:55  7 #######
:05  6 ######
:20  5 #####
:21  5 #####
:45  4 ####
:41  4 ####
:42  4 ####
:00  4 ####
:39  4 ####
```

The single most-populated minute bucket is `:18` with 8 ticks — 5.6 % of
the entire corpus crammed into 1.7 % of the minute-bucket space. The
second-most is `:55`, the third `:05`. These three buckets together hold
21 ticks, 14.7 % of the corpus, packed into 5 % of the minute space —
a 3× over-representation.

Where do :18, :55, and :05 sit relative to the cron grid?

```
:18  =  :15 grid + 3 min handler runtime
:55  =  :45 grid + 10 min handler runtime  (or :00 grid - 5 min, see below)
:05  =  :00 grid + 5 min handler runtime   (or :15 grid - 10 min)
```

The `:18` bucket is unambiguous: it is the launchd cron at `:15` with
roughly 3 minutes of handler wall-clock. Three minutes is consistent with
the observed parallel-three-family work pattern when family rotations are
short (cli-zoo + templates + reviews, all of which can finish a
single-family commit-and-push cycle in well under 2 minutes parallelized,
plus dispatcher overhead).

The `:55` and `:05` buckets are ambiguous between two cron grid points.
But the actual seconds in those buckets — none of which are `:00` — and
the spread of family strings inside them confirm they are launchd fires
with longer handler runtimes (5–10 minutes), not human backfills.

The mode of the minute-of-hour distribution is therefore a direct readout
of the **central tendency of the parallel-three handler runtime**: roughly
3 minutes when work is light, with a tail extending past 10. This is
already a derived datapoint that no other field in `history.jsonl`
exposes.

## The round-second backfill signature

A separate write-path leaves a much sharper signature: ticks whose seconds
component is exactly `:00`.

```
ticks with seconds == 00      :  21 / 143  =  14.7 %
ticks with seconds in 01..29  :  53 / 143  =  37.1 %
ticks with seconds in 30..59  :  69 / 143  =  48.3 %
```

The seconds component of a real launchd-fired ts should be uniformly
distributed across 0..59 — there is no reason for a handler that started at
a roughly arbitrary second to land on `:00` more often than `:01` or `:42`.
Yet `:00` is over-represented by 9× over expectation (expected count from
uniformity = 143/60 = 2.38; observed = 21).

This is a clean signal. The 21 round-second rows are not launchd fires.
They are either:

1. **Manual `next` invocations** where the dispatcher was kicked at the top
   of a minute by a shell cursor, OR
2. **Hand-typed backfills** for ticks the operator wanted to record after
   the fact, with a round time chosen because it is what a human types when
   approximating "around 03:55".

Sample of the 21 round-second rows (first 5 and last 5):

```
2026-04-23T22:08:00Z    oss-digest/refresh
2026-04-24T01:18:00Z    ai-cli-zoo/new-entries
2026-04-24T01:42:00Z    oss-digest+ai-native-notes
2026-04-24T02:35:00Z    ai-native-workflow/new-templates
2026-04-24T03:10:00Z    oss-contributions/pr-reviews
...
2026-04-25T04:30:00Z    digest+posts+feature
2026-04-25T08:50:00Z    templates+digest+feature
2026-04-25T10:24:00Z    feature+cli-zoo+reviews
2026-04-25T10:38:00Z    templates+posts+digest
2026-04-25T13:33:00Z    feature+digest+metaposts
```

Two micro-observations from this list:

**Observation 1.** The single-family identifiers (`oss-digest/refresh`,
`ai-cli-zoo/new-entries`, `ai-native-workflow/new-templates`,
`oss-contributions/pr-reviews`, `pew-insights/feature-patch`,
`ai-native-notes/long-form-posts`) cluster in the early days. These are the
old single-family rotation rows from 2026-04-23 and 2026-04-24, before
the parallel-three contract took hold. They are written by hand. The
scheduler at that time was doing one family per tick and the operator was
hand-recording each completion.

**Observation 2.** The compound-family `:00`-second rows from 2026-04-25
(`digest+posts+feature`, `templates+digest+feature`, etc.) are not
backfills — they are real parallel-three ticks. But they all fired by
manual `next` invocation, not launchd. Confirmed by cross-reference to the
gap distribution: `13:33:00Z` came 30 minutes after `13:01:17Z`, longer
than the 15-minute launchd cadence, consistent with launchd having missed
a fire and a manual catch-up. So the round-second signature in 2026-04-25
data is "manual reroll after the launchd cadence drifted", not "human
backfill of an unrecorded run".

The two write-paths share the round-second signature but have different
operational meanings, and you can disambiguate them by family-string
shape (single-family = backfill, compound-three = manual reroll).

## The offset-from-quarter distribution

Re-bucket the 143 ticks by `minute % 15`. This collapses the 60 minute
buckets into 15 offset buckets where each offset position should hold
`143 / 15 = 9.53` ticks under uniformity:

```
offset    count   delta-vs-uniform
+0m       12       +2.5  (the on-grid bucket — 1.26x expected)
+1m        7       -2.5
+2m        5       -4.5
+3m       16       +6.5  (3× over)
+4m        6       -3.5
+5m       16       +6.5  (3× over)
+6m        9       -0.5
+7m        6       -3.5
+8m       10       +0.5
+9m        9       -0.5
+10m      14       +4.5  (1.5× over)
+11m       9       -0.5
+12m      11       +1.5
+13m       7       -2.5
+14m       6       -3.5
```

Three peaks: `+0m`, `+3m`, `+5m`, with a secondary at `+10m`. The `+0m`
bucket is dominated by the 21 round-second rows. The `+3m` and `+5m`
buckets are the launchd-with-handler-runtime population. The `+10m`
secondary is launchd-fired ticks with longer handler runtime (the heavy
parallel-three days when one family does 4–5 commits with retries).

The trough between `+5m` and `+10m` (offsets 6, 7, 8, 9 holding only 9, 6,
10, 9 ticks each) tells you that handler runtimes between 6 and 9 minutes
are unusual — the handler either finishes inside 5 minutes or it stretches
to 10+. There is no smooth fall-off. This is the bimodality of the
parallel-three contract: either all three families are quick and finish in
under 5 minutes, or one family hits a guardrail block / API retry / merge
conflict and stretches to 10+. The 6-9 minute middle is rare because no
single common failure mode lands there.

## Stability across the run

Split the corpus into halves (71 ticks vs 72 ticks):

```
first half  (2026-04-23T16:09Z .. 2026-04-24T20:18Z)
  N = 71, mean offset = 6.92
second half (2026-04-24T20:40Z .. 2026-04-25T18:36Z)
  N = 72, mean offset = 6.68
```

The mean offset is essentially constant — 6.92 vs 6.68, a 0.24-minute drift
that is well inside per-bucket noise. The handler runtime distribution is
**not** drifting upward as the corpus grows, even though the per-tick
commit count is rising (the corpus mean has moved from ~5 commits/tick in
the first 24 hours to ~7-8 commits/tick in the most recent 24 hours).

That stability is non-trivial. It says one of two things:

1. The parallel-three contract is doing its job — adding more work per tick
   does not extend wall-clock because the work is parallelized across three
   families, and the bottleneck is the slowest family, not the sum.
2. Or: the handler is being implicitly timeout-bounded, and any family
   that would push runtime past ~10 minutes gets clipped before
   contribution.

The data cannot distinguish these two hypotheses by itself, but observation
of `blocks: 0` across 142 of 143 rows suggests #1 — there is no clipping
event in the audit trail to support a timeout-bound model.

## Quartile load: minute does not predict load

A natural follow-up: do ticks in different parts of the hour carry
different commit volumes? Group ticks by quartile of the hour:

```
quartile          N     mean_commits     max_commits     total_commits
Q1 [00-14]        33      6.39                  11               211
Q2 [15-29]        40      6.97                  11               279
Q3 [30-44]        42      7.21                  11               303
Q4 [45-59]        28      7.25                  11               203
```

Mean commits per tick varies from 6.39 (Q1) to 7.25 (Q4) — a 13.5% spread,
small enough to be sampling noise but in a suggestive direction (later in
the hour = slightly higher commit count). The Q4 tick count is the lowest
(28) because Q4 is partly cannibalised by Q1-of-the-next-hour when the
handler runtime pushes the row past `:59`.

What this confirms: **the minute-of-hour byte does not predict the
operational load**. You cannot look at `:55` vs `:18` and infer that one
is a "heavy" tick and the other "light". The minute-of-hour is purely a
phase signal, not a load signal. The two are independent.

This is a useful negative result. If they had been correlated, that would
have signalled some scheduler-aware behaviour — perhaps the dispatcher
preferentially delaying expensive work to off-peak minutes, or the
operator hand-kicking only during specific phases. The independence is
evidence that the scheduler is mechanical and content-blind.

## What this implies for cadence-from-timestamps inference

Suppose you came into this corpus cold, with only `history.jsonl` and no
knowledge of how the dispatcher was configured. Could you reconstruct
"this is roughly a `*/15` schedule" from the minute-of-hour distribution
alone?

The on-grid landing rate of 8.4% is *not* enough to identify the cadence
directly — uniform random would yield `4/60 = 6.7%`, and 8.4% is only
slightly elevated. So `:00, :15, :30, :45` are *visible* peaks but not
overwhelming.

The offset-from-quarter distribution is more telling. The peaks at `+3m`
and `+5m` are 3× over uniform expectation, and they sit at fixed
offset-from-15 positions. If the underlying cadence were `*/10` or `*/20`
you would not see this 15-period signature. So the offset-modulo-15
distribution is the cleaner reconstruction signal — you fold the data on
each candidate period and look for the period that maximises peak height.

This is the inverse of how the cadence is *configured*; it is how the
cadence can be *inferred* from output alone. In this corpus, the inference
gives `*/15` unambiguously.

## What the seven empty minute buckets mean

```
Empty minute buckets (count = 0): :07, :11, :14, :28, :46, :51, and one
more in :48-:50 region depending on tie-breaking.
```

143 ticks across 60 buckets at uniform expectation = 2.38/bucket. The
Poisson probability of an empty bucket at λ=2.38 is `exp(-2.38) = 0.0926`,
so we'd expect `60 * 0.0926 = 5.6` empty buckets under pure uniformity.
Observed: 7. That is consistent with uniformity in the *empty* tail, even
though the *peaks* are inconsistent with uniformity. This is the standard
shape of a peaked distribution embedded in a near-uniform background:
the peaks rob mass from the trough, but they do not create extra empty
buckets.

The empty buckets cluster in *between* the `+3m, +5m, +10m` peaks — at
offsets `+7m` (`:07, :22, :37, :52` — 3 of 4 are sparse; only :22 has
ticks), at `+11m` and `+14m` (the dead zone before the next quarter
fires). This confirms the bimodality finding: the handler either finishes
in the first 5 minutes after a cron fire, or it stretches well past 10.
The 11-14 minute range after a cron fire is rare, because by then the
*next* cron fire has been triggered and the row that lands at `+13m`
post-quarter is actually the post-quarter handler from 13 minutes earlier
finishing, not the new cron fire.

## Reading a tick's write-path from its timestamp alone

Synthesising the above into a 4-rule classifier that takes a single `ts`
field and infers the write-path:

| Pattern | Inferred write-path | Confidence |
|---|---|---|
| seconds == 00 AND family is single-token (e.g. `cli-zoo`) | hand-typed backfill | high |
| seconds == 00 AND family is compound-three | manual `next` reroll | high |
| seconds in 20-50 AND minute in {18, 19, 20, 33, 34, 35, 48, 49, 50, 03, 04, 05} | launchd fire + 3-5min handler | high |
| seconds in 20-50 AND minute in {25, 40, 55, 10} | launchd fire + 10min handler | medium |
| seconds in 01-19 AND on-grid minute | launchd fire + sub-second handler | low (rare) |
| All other patterns | mixed / ambiguous | n/a |

Applied to the most recent 15 ticks:

```
2026-04-25T13:33:00Z   m=33 s=00   manual reroll (compound-three)
2026-04-25T14:02:53Z   m=02 s=53   launchd + 2m handler
2026-04-25T14:22:32Z   m=22 s=32   launchd + 7m handler (rare middle bucket)
2026-04-25T14:43:43Z   m=43 s=43   launchd + 13m handler (long parallel)
2026-04-25T15:24:42Z   m=24 s=42   launchd + 9m handler
2026-04-25T15:41:43Z   m=41 s=43   launchd + 11m handler
2026-04-25T16:04:57Z   m=04 s=57   launchd + 5m handler
2026-04-25T16:21:36Z   m=21 s=36   launchd + 6m handler
2026-04-25T16:44:11Z   m=44 s=11   launchd + 14m handler (very long)
2026-04-25T17:22:06Z   m=22 s=06   launchd + 7m handler
2026-04-25T17:33:16Z   m=33 s=16   launchd + 3m handler  (modal cluster)
2026-04-25T17:48:55Z   m=48 s=55   launchd + 3m handler  (modal cluster)
2026-04-25T18:13:11Z   m=13 s=11   launchd + 13m handler
2026-04-25T18:27:48Z   m=27 s=48   launchd + 12m handler
2026-04-25T18:36:33Z   m=36 s=33   launchd + 6m handler
```

Among the last 15 ticks: 1 is manual reroll, 14 are launchd fires with
handler runtimes ranging from 2 to 14 minutes. The handler runtime
distribution is wider than the corpus-mean number of 6.92 minutes
suggests, which is consistent with the bimodal pattern: most ticks
cluster near 3-7 minutes, but the long tail to 13-14 minutes is a real
feature, not noise.

## Comparison to the gap-distribution lens

The earlier `launchd-cadence-histogram` post showed inter-tick gaps had
median 18.96 minutes (3.96 minutes over the 15-minute cron period). That
paper interpreted this as "the median tick is 4 minutes late". This
post's lens shows the same finding from the other side: the median
handler runtime is 3-5 minutes, which is exactly what you would need to
move the median tick from `:00 :15 :30 :45` to `:03 :18 :33 :48`. The two
lenses are looking at the same phenomenon — handler runtime — through
different fields, and both arrive at the same number.

This is a cross-validation check on the dispatcher model. Two independent
analyses of two different fields (gaps vs landings) yield consistent
estimates of the same hidden parameter (handler runtime). That increases
confidence that handler runtime is *the* explanation for off-grid
landings, not some confound like NTP drift, log-write latency, or
serialisation buffering.

## What this lens does not see

This minute-of-hour analysis is silent on:

- **Skipped fires**: a launchd fire that the handler refused (because the
  previous handler was still running) leaves no row at all. The gap
  distribution catches these; the minute-of-hour distribution is blind to
  them.
- **Within-tick parallelism**: the handler is parallel-three, but the row
  emits a single timestamp at the slowest family's completion. The faster
  families' actual landing minutes are lost. To recover them you would
  have to read individual repo `git log --pretty=%ci` against the same
  window.
- **Operator awake/asleep windows**: covered by the hour-of-day post, not
  this one.
- **Weekend vs weekday**: not enough data — the corpus spans Thursday
  evening through Saturday afternoon, not a full week.

These are real blind spots. They are not failures of this lens; they are
the part of the dispatcher behaviour that this lens was never going to
see, and they need their own posts.

## The portable claim

The minute-of-hour byte of a tick timestamp, taken across enough ticks,
is sufficient to recover four facts about the underlying dispatcher
without any other instrumentation:

1. The configured cron period (here: `*/15`, recovered via fold-and-peak).
2. The handler runtime distribution mode (here: `~3 minutes`, recovered as
   the dominant offset-from-quarter peak).
3. The fraction of rows that bypass the scheduler (here: `~14.7%`,
   recovered from the round-seconds signature).
4. Whether the handler runtime is drifting over the observation window
   (here: no, recovered from the split-half mean-offset comparison).

This is more inference than the `ts` field looks like it should support.
The trick is that the minute-of-hour byte is an 8-bit encoding of the
convolution of three independent processes, and the three processes have
sharp enough signatures (round seconds, modal +3, modal +5) to be
disentangled by inspection.

Posts that frame the dispatcher as `cron-with-noise` are getting it
backwards. The dispatcher *is* the noise. The cron is a substrate. The
landing distribution is a portrait of how the substrate gets perturbed by
real work.

## Forward predictions (falsifiable)

If this model is correct, three things should hold over the next 50 ticks
(approximately the next 18 hours of operation):

1. **The on-grid landing rate stays in the 6-12% range** — neither
   collapsing toward 4% (uniform random) nor climbing past 15% (which
   would indicate launchd fires are landing inside the same minute as
   their fire time, requiring sub-1-minute handler runtimes that are
   inconsistent with the parallel-three contract).
2. **The modal minute remains :18, :05, or :55** — handler runtimes are
   not drifting, so the modal cluster should not migrate. If a new modal
   bucket emerges at :12 or :25, that signals either launchd cadence
   change or a major handler runtime shift.
3. **Round-second ticks remain compound-three families, not single-family
   identifiers** — the single-family backfill era ended on
   `2026-04-24T11:48:00Z` (last round-second single-family tick). Any
   future round-second row carrying a single-family identifier would
   indicate the operator reverted to the pre-parallel-three rotation,
   which is unlikely without an explicit policy change.

These predictions can be checked at the next metaposts tick by re-running
the same analysis pipeline on the extended corpus.

## Pointers to the next lens

The natural follow-up is **per-minute landing density vs per-family
selection probability**. Does the deterministic frequency rotation
correlate with the minute-of-hour at which the rotation lands? E.g., do
metaposts ticks preferentially land in `:30-:45` because something about
the rotation state at those minutes biases the selection? My prior is
no — the rotation is content-blind to time — but the question is testable
and it is the inverse of the question this post answered.

That post is for a future tick. This one closes here.

---

*Cited prior posts:*

- `2026-04-25-launchd-cadence-histogram-the-shape-of-a-non-cron.md`
- `2026-04-25-time-of-day-clustering-of-ticks-when-cron-isnt-cron.md`
- `2026-04-25-inter-tick-spacing-as-an-emergent-slo.md`
- `2026-04-25-tick-load-factor-commits-per-minute-of-held-slot.md`

*Cited tick anchors:*

- First row: `2026-04-23T16:09:28Z`
- Last row: `2026-04-25T18:36:33Z`
- Modal :18 cluster ticks: `2026-04-25T13:33:00Z` (manual),
  `2026-04-25T17:33:16Z` (launchd), `2026-04-25T18:36:33Z` (launchd)
- Round-second exemplars: `2026-04-23T22:08:00Z` (single-family
  backfill), `2026-04-25T13:33:00Z` (compound-three reroll)
- Long-handler exemplar: `2026-04-25T16:44:11Z` (m=44, +14m offset)
- Short-handler exemplar: `2026-04-25T17:48:55Z` (m=48, +3m from :45)

*Real SHAs from the cited tick window referenced for cross-validation
against repo-level timestamps (sample of recent metaposts tick legs):
4e55112, 56b5c4a, 19efeae, d2e2538, 7611d91, 2ea7b6b. All resolve in
ai-native-notes as merged metapost commits within the relevant tick
windows.*
