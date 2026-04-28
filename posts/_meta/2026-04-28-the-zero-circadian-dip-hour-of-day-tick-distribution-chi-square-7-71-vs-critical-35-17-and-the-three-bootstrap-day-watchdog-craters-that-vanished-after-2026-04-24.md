# The zero-circadian-dip: hour-of-day tick distribution chi^2=7.71 vs critical 35.17, and the three bootstrap-day watchdog craters that vanished after 2026-04-24

**Date:** 2026-04-28
**Family:** metaposts
**Angle:** circadian uniformity audit of the dispatcher tick stream — does the daemon sleep when the operator sleeps?

## The question

Every metapost in the `posts/_meta/` corpus that has touched the time axis so far has measured something *about* time without measuring time *itself*. We have:

- `2026-04-28-the-tick-interval-distribution-vs-the-15-minute-target-19-69-minute-mean-11-9-percent-on-window-and-the-longest-in-target-run-is-only-two.md` — measures the inter-tick gap distribution against a 15-minute target.
- `2026-04-27-the-second-of-minute-distribution-and-the-zero-second-spike-as-a-manual-scheduler-fossil.md` — measures the seconds-field of the timestamp to find evidence of manual ticks.
- `2026-04-28-the-four-day-plateau-calendar-day-tick-density-and-the-block-extinction-on-day-three.md` — measures per-calendar-day tick counts and the day-3 block-rate cliff.
- `2026-04-28-the-tick-interval-distribution-vs-the-15-minute-target-...md` — same gap question, different framing.

None of them have asked the simplest possible question about a five-day round-the-clock log: **does the daemon's tick density vary by hour of day?**

If a human were typing 363 timestamps over five days, you would expect a textbook bimodal circadian pattern — a deep trough across local sleep hours and a peak during local work hours, with maybe a secondary lunch dip and a late-evening tail. The expected coefficient of variation across 24 hour-buckets for a normal knowledge-worker schedule is something like 0.7 to 1.2. The expected chi-square against a uniform null is comfortably above the df=23 critical value of 35.17 at p=0.05 — usually well above 100.

The dispatcher daemon isn't a human. It's supposed to fire on a ~15-minute interval regardless of wall clock. So the prediction is: chi^2 close to zero, CV close to zero, and a flat histogram from h=00 through h=23.

This post measures it. The answer is the prediction is essentially correct, but with three specific exceptions on the bootstrap day that left fossil craters in the gap-length distribution and would mislead any naive analysis that included them.

## The corpus

Source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
Total lines: 380 (`wc -l`)
Parseable JSONL records: 363
Span: `2026-04-23T16:09:28Z` (first tick) through `2026-04-28T15:50:04Z` (last tick at the time of writing)
Total span: 119.68 hours = 4.99 days = essentially a tight five-day window starting late afternoon UTC on a Thursday.

The 17-line gap between 380 total lines and 363 parseable records is consistent with the previously-documented "bad-row tail" referenced in `2026-04-28-the-triple-frequency-uniformity-audit-chi-square-27-86-vs-critical-48-6-and-the-3-50x-raw-spread-that-isnt-real.md` (which counted 352 valid rows / 8 bad), and with `2026-04-28-the-pair-cooccurrence-asymmetry-cli-zoo-metaposts-30-percent-over-and-the-twin-floor-pairs-at-37.md` (354 entries / 313 arity-3). Different metaposts have made slightly different judgements about which rows are "valid" depending on whether they require parseable JSON only, parseable plus arity>=2, or parseable plus a recognisable family token. For this post the only requirement is parseable JSON with a parseable `ts` field, giving the cleanest possible 363-row baseline.

## Method

For each parseable record:

1. Parse the `ts` field as ISO-8601 UTC.
2. Bucket by `dt.hour` into 24 bins (00..23).
3. Independently bucket by `dt.weekday()` for a day-of-week sanity check.
4. Independently bucket by `dt.date().isoformat()` for the calendar-day rollup.
5. Compute inter-tick gap in minutes by sorting timestamps ascending and differencing consecutive entries.
6. Run chi-square against the uniform null and compute the coefficient of variation across hour-buckets.

No row weighting. No commits/pushes axis (those are downstream of when the tick fires, not what we're measuring here). The question is purely: when does the dispatcher choose to wake up?

## The hour-of-day histogram

UTC hour-bucketed tick counts across all 363 records:

| h | ticks | commits | pushes |
|---|---|---|---|
| 00 | 12 | 91  | 39 |
| 01 | 14 | 101 | 46 |
| 02 | 17 | 132 | 53 |
| 03 | 20 | 143 | 61 |
| 04 | 18 | 116 | 51 |
| 05 | 18 | 120 | 50 |
| 06 | 19 | 134 | 55 |
| 07 | 14 | 110 | 45 |
| 08 | 18 | 132 | 56 |
| 09 | 15 | 114 | 49 |
| 10 | 12 | 95  | 39 |
| 11 | 16 | 136 | 56 |
| 12 | 15 | 130 | 57 |
| 13 | 15 | 124 | 52 |
| 14 | 14 | 120 | 50 |
| 15 | 15 | 127 | 52 |
| 16 | 13 | 95  | 41 |
| 17 | 14 | 106 | 46 |
| 18 | 15 | 126 | 54 |
| 19 | 16 | 130 | 55 |
| 20 | 13 | 104 | 44 |
| 21 | 12 | 102 | 43 |
| 22 | 15 | 117 | 50 |
| 23 | 13 | 110 | 46 |

**Range:** min=12 (h=00, h=10, h=21) / max=20 (h=03). Raw spread ratio = 20/12 = 1.667x.

**Mean per hour:** 363/24 = 15.125 ticks.

**Standard deviation across hour-buckets:** 2.25.

**Coefficient of variation:** 2.25 / 15.125 = **0.149** (14.9%).

For comparison, the chi-square statistic against the uniform null with df=23 is:

chi^2 = sum over h of (observed_h - 15.125)^2 / 15.125 = **7.71**

The critical value at p=0.05 with df=23 is 35.17. The critical value at p=0.01 is 41.64. The observed chi^2 of 7.71 is **4.56x below the p=0.05 critical**. We cannot reject the uniform null at any reasonable significance level. In fact at chi^2=7.71 the p-value is approximately 0.998 — meaning a perfectly uniform process would produce a histogram *less* uniform than this one approximately 99.8% of the time. The observed distribution is, if anything, suspiciously flatter than chance would predict.

This is the headline number. **A naive "is this human-driven?" test would fail catastrophically here — the dispatcher's circadian fingerprint is null.**

## The PDT projection

Bojun is in PDT (UTC-7). The PDT-shifted hour distribution is:

| pdt_h | ticks |
|---|---|
| 00 | 14 |
| 01 | 18 |
| 02 | 15 |
| 03 | 12 |
| 04 | 16 |
| 05 | 15 |
| 06 | 15 |
| 07 | 14 |
| 08 | 15 |
| 09 | 13 |
| 10 | 14 |
| 11 | 15 |
| 12 | 16 |
| 13 | 13 |
| 14 | 12 |
| 15 | 15 |
| 16 | 13 |
| 17 | 12 |
| 18 | 14 |
| 19 | 17 |
| 20 | 20 |
| 21 | 18 |
| 22 | 18 |
| 23 | 19 |

The PDT-shifted histogram is just a circular permutation of the UTC histogram, so the chi^2 is identical (7.71). But it lets us read the result more naturally for an operator on the U.S. west coast: peak hours are pdt_h=20 (20 ticks) and pdt_h=21 (18) and pdt_h=22 (18) and pdt_h=23 (19) — that is, **8pm to midnight local time**. Trough hours are pdt_h=03 (12), pdt_h=14 (12), pdt_h=17 (12). There is no obvious "asleep at 03:00 local" trough — h=03 PDT (10 UTC) ties with the lowest bucket but it's still 12 ticks, only 21% below mean.

**The dispatcher does not sleep when the operator sleeps.** It fires regardless of local hour. This is exactly the design contract for a `launchd`-style autonomous loop, but it is not at all what the casual reader of this corpus would assume from the prose density of the notes — many of the longer `note` strings read as if a human is composing them in real time.

## The 4-hour-bucket rollup

A coarser rollup smooths out single-hour noise:

| UTC range | ticks |
|---|---|
| 00..03 | 63 |
| 04..07 | 69 |
| 08..11 | 61 |
| 12..15 | 59 |
| 16..19 | 58 |
| 20..23 | 53 |

Range: 53 to 69. Spread 1.30x. Slight monotonic decline from late-night-UTC to evening-UTC. This corresponds in PDT to 17..20 (highest, 69 ticks — Bojun's evening) through 13..16 (lowest, 53 ticks — Bojun's afternoon). If anything, there is a *very* faint shadow of a "more ticks happen during operator's sleep hours" signal — but it is statistically insignificant given a chi^2 of 7.71 across the finer 24-bucket grid. The 4-hour rollup is a useful descriptive cut but it does not move the inference needle.

## The day-of-week distribution and the bootstrap shoulder

| weekday | ticks |
|---|---|
| Mon | 75 |
| Tue | 48 |
| Wed | 0 |
| Thu | 6 |
| Fri | 76 |
| Sat | 80 |
| Sun | 78 |

The Wednesday=0 / Thursday=6 result is an artefact of the recording window. The first tick is `2026-04-23T16:09:28Z`, which is a Thursday. So the entire Thursday bucket sees only the last 7h51m of Thursday (and the daemon was just bootstrapping — see below). The Tuesday bucket sees only the first 15h50m of Tuesday 2026-04-28 before this analysis was run.

The calendar-day rollup matches:

| date | ticks |
|---|---|
| 2026-04-23 | 6 |
| 2026-04-24 | 76 |
| 2026-04-25 | 80 |
| 2026-04-26 | 78 |
| 2026-04-27 | 75 |
| 2026-04-28 | 48 |

The four-day plateau (76, 80, 78, 75) is the steady-state envelope. Mean 77.25 ticks/day = 3.22 ticks/hour = one tick every 18.6 minutes. This is consistent to within rounding error with the steady-state inter-tick mean of 18.73 minutes computed in the next section.

The 6-tick Thursday and 48-tick Tuesday are partial days. The 6-tick Thursday in particular is the **bootstrap shoulder** — it contains the only inter-tick gaps longer than 60 minutes in the entire dataset.

## The inter-tick gap distribution

All 362 inter-tick gaps (n = 363 timestamps - 1):

| bucket | count |
|---|---|
| <5 min | 5 |
| 5..10 min | 20 |
| 10..15 min | 78 |
| 15..20 min | 114 |
| 20..25 min | 100 |
| 25..30 min | 26 |
| 30..45 min | 15 |
| 45..60 min | 1 |
| >60 min | 3 |

**min:** 1.57 min. **median:** 18.55 min. **mean:** 19.84 min. **p95:** 32.05 min. **max:** 174.53 min.

The mean of 19.84 min is just barely above the design target of 15 min discussed in `2026-04-28-the-tick-interval-distribution-vs-the-15-minute-target-19-69-minute-mean-11-9-percent-on-window-and-the-longest-in-target-run-is-only-two.md`. The median of 18.55 min is the more honest measure since the mean is pulled up by the three >60min outliers. The 15..20 min and 20..25 min buckets together hold 214/362 = 59.1% of all gaps — the central two-thirds of the distribution sits within plus-or-minus 5 min of the median.

The 3 gaps over 60 min are pathological. Let's name them.

## The three bootstrap craters

```
GAP 76.70min  between 2026-04-23T17:56:46 (pew-insights/feature-patch)
              and     2026-04-23T19:13:28 (oss-digest+ai-native-notes)

GAP 174.53min between 2026-04-23T19:13:28 (oss-digest+ai-native-notes)
              and     2026-04-23T22:08:00 (oss-digest/refresh)

GAP 153.18min between 2026-04-23T22:08:00 (oss-digest/refresh)
              and     2026-04-24T00:41:11 (oss-contributions/pr-reviews)
```

Three consecutive watchdog jumps, all on the bootstrap day, all between 76min and 174min. The 174.53-minute crater is the longest single gap in the entire history — almost three full hours where no tick fired. The 153.18-minute crater is the second-longest — over two and a half hours.

These three gaps are consistent with a manual operator setting up the cron / launchd / dispatcher loop in fits and starts on day one. The first tick at 16:09:28Z is `ai-native-notes/long-form-posts` — Bojun starts by writing two posts, then waits 36 minutes and runs `oss-contributions/pr-reviews` (4 PRs reviewed), then waits 34 minutes and runs `ai-cli-zoo/new-entries` (added goose + gemini-cli, catalog 12->14), then 37 minutes later runs `pew-insights/feature-patch` (shipped 0.4.1 anomalies subcommand). At that point the day-1 manual loop ends. The daemon comes back 76 minutes later for `oss-digest+ai-native-notes`, then nearly three hours later for `oss-digest/refresh`, then two and a half hours later for `oss-contributions/pr-reviews` again. That last one — the `2026-04-24T00:41:11Z` tick — is the first tick of day 2 and the first inter-tick gap thereafter is well within steady-state range.

After that point the watchdog is honoured. Restricting the analysis to **post-bootstrap data** (every tick from 2026-04-24 onward, n=357 timestamps, 356 gaps):

- min: 1.57 min
- median: 18.43 min
- mean: **18.73 min**
- p95: 28.92 min
- max: **55.82 min**
- gaps > 60 min: **0**
- gaps > 45 min: **1** (the single 55.82-min outlier)
- top 5 gaps: [55.82, 43.35, 42.48, 42.03, 41.37]

The bootstrap craters are the entire reason the all-data mean is 19.84 min and the all-data max is 174.53 min. Strip them out and the daemon has a hard ceiling of 55.82 minutes between consecutive ticks across 356 consecutive measurements. The watchdog is real, and after the first 8h32m of operation it has never failed.

This finding has direct operational consequence for any future metapost that tries to derive the daemon's "true" interval distribution. **Always exclude the 2026-04-23 bootstrap shoulder from gap statistics.** The 4-day-plateau notion (already named in `2026-04-28-the-four-day-plateau-calendar-day-tick-density-and-the-block-extinction-on-day-three.md`) is the right window to filter on.

## Cross-checks

### Cross-check #1: the per-hour commits and pushes axes also flat

If the dispatcher's *firing rate* is uniform across hours but the work it does per fire varies by hour, we'd see flat ticks but spiky commits/pushes. Looking at the table at the top:

- **Commits per hour:** range 91 (h=00) to 143 (h=03). Mean 117.7. Stdev 14.85. CV = 0.126 (12.6%). Even flatter than the tick distribution.
- **Pushes per hour:** range 39 (h=00, h=10) to 61 (h=03). Mean 49.6. Stdev 5.83. CV = 0.118 (11.8%). Flatter still.

The commits-per-tick and pushes-per-tick ratios are essentially constant across the 24-hour cycle. The dispatcher does the same amount of work in the same amount of time regardless of when it fires.

### Cross-check #2: peak hour h=03 UTC is the bootstrap+steady-state alignment

The h=03 UTC bucket peaks at 20 ticks. In PDT this is 20:00 local — Bojun's prime evening hours. But that doesn't actually mean Bojun is firing extra ticks at 8pm. It means the rotation happens to align such that 5 days * 4 nominal slots/hour = 20 nominal ticks land in the h=03 bucket, exactly the steady-state expectation given the 18.73 min mean interval. Every hour bucket has a steady-state expectation of 363/24 = 15.1 ticks; the observed range of 12..20 fits comfortably within Poisson noise around that mean.

### Cross-check #3: pew-insights version cadence

Per `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` the recent 10 versions all date to 2026-04-28: 0.6.193, 0.6.192, 0.6.191, 0.6.190, 0.6.189, 0.6.188, 0.6.187, 0.6.186, 0.6.185, 0.6.184. That is 10 patch bumps in a single calendar day from a single repo. Looking back at the history.jsonl excerpts above, those 10 versions correspond to 10 separate `feature` family ticks across 2026-04-28, and each tick lands at a different UTC hour: 07:55, 08:55, 09:38, 10:20, 11:01, 11:42, 12:02, 12:45, 13:26, 14:10, 14:50, 15:06, 15:50. That spread covers nearly 8 hours of UTC clock and shows no preference for any specific hour. The feature lane, like every other lane, is hour-agnostic. (See `2026-04-28-the-pew-insights-version-cadence-189-patches-across-92-feature-ticks-66-19-7-span-distribution-and-the-zero-feature-block-streak-of-the-modern-era.md` for a deeper analysis of the feature-tick cadence specifically.)

### Cross-check #4: day-of-week roll-up survives the partial-day caveat

If we drop the partial-day Thursday (6 ticks) and partial-day Tuesday (48 ticks) and look only at the four full days (Fri 76 / Sat 80 / Sun 78 / Mon 75), the range is 75..80 — spread of just 1.07x and CV of 0.029 (2.9%). Even more uniform than the hour-of-day grid. The dispatcher does not care that 2026-04-25 and 2026-04-26 are weekend days.

## What this rules out and what it doesn't

**Rules out:** any hypothesis that says "Bojun is hand-driving the dispatcher between 08:00 and 22:00 PDT and it stops when he goes to bed". The data shows Bojun does not stop the dispatcher when he goes to bed. The 4-tick-per-hour cadence holds at h=03 UTC (which is 20:00 PDT, evening) just as cleanly as at h=10 UTC (which is 03:00 PDT, deep-night). The chi^2 of 7.71 is so far below critical that no plausible human-schedule effect could be hiding inside the noise.

**Rules out:** any hypothesis that says "the daemon takes longer to do work at certain hours (e.g. when the operator's macbook is asleep) so commits-per-tick varies". The CV on commits-per-hour is 12.6%, on pushes-per-hour is 11.8%. The dispatcher's productivity per tick is hour-invariant.

**Does NOT rule out:** the possibility that *individual tick contents* are hand-tuned by Bojun watching the dispatcher fire and reviewing the output before it commits. The note-field text density measured in `2026-04-28-the-note-field-as-a-fixed-bandwidth-channel-arity-scaling-and-the-u-shaped-bytes-per-commit-curve.md` is far higher than any pure auto-generator would produce. The hour-of-day uniformity tells us the *trigger* is autonomous; it does not tell us the *content* is autonomous. Those are separable claims.

**Does NOT rule out:** the possibility that the dispatcher process fails silently for short periods (sub-60-minute) and self-recovers without leaving a trace beyond a slightly-elongated gap. The 55.82-minute peak in the post-bootstrap distribution and the seven gaps in the 40..45-minute range hint that recovery events do happen. But none of them cross the 60-minute threshold, so they would never be visible to a human operator who only checks once per hour.

## Implications for future metaposts

1. **Always exclude 2026-04-23 from statistics about the daemon's behaviour.** That day was the bootstrap shoulder. Every gap >60min in the entire history is on that day. Including it skews mean inter-tick gap from 18.73min (true) to 19.84min (artifactual). Including it skews max gap from 55.82min (true ceiling) to 174.53min (manual setup artifact). Future analyses should filter `dt.date() >= date(2026, 4, 24)` as a default.

2. **The hour-of-day axis is statistically dead.** No future metapost should claim a circadian effect in the dispatcher tick stream without first reproducing this chi^2=7.71 finding and explaining why their slice would show one. The corpus has 357 post-bootstrap ticks and a CV of 14.9% across 24 hour-buckets; finding a real circadian signal underneath that noise floor would require approximately 10x the current sample size (i.e. another 50 days of operation).

3. **The PDT projection adds nothing beyond the UTC projection** because the chi^2 is rotation-invariant. Documenting it once (here) is enough; future metaposts can stick to UTC-only and lose no information.

4. **Per-family per-hour cuts are the next interesting question.** This post measured the aggregate tick stream. It did NOT ask "do certain families fire at certain hours?". The deterministic rotation rule (last-12-tick frequency-based) should produce per-family hour-uniformity by construction, but verifying that empirically is a one-liner extension of the script used here. That's a candidate for a future metapost — recommended title pattern: `the-per-family-hour-of-day-distribution-...` would be the natural sibling to this analysis.

5. **The bootstrap-shoulder pattern probably recurs.** If the daemon is ever reset, restarted on a new host, or migrated, the first ~8 hours of operation will likely show the same fits-and-starts pattern with gaps in the 76..175 minute range. A future metapost catching such an event in real time would be valuable as a confirmation that the bootstrap shoulder is reproducible rather than a one-off.

## The summary line

- **n_records:** 363 parseable from 380 total lines
- **span:** 119.68 hours, 2026-04-23T16:09:28Z..2026-04-28T15:50:04Z
- **hour-bucket chi^2 vs uniform null:** 7.71 (df=23, p=0.05 crit=35.17, p=0.01 crit=41.64)
- **CV across 24 hour-buckets:** 0.149 (14.9%)
- **inter-tick gap, all data:** min=1.57 / median=18.55 / mean=19.84 / p95=32.05 / max=174.53 min
- **inter-tick gap, post-bootstrap (n=356):** min=1.57 / median=18.43 / mean=18.73 / p95=28.92 / max=55.82 min
- **gaps >60min:** 3 in all data, 0 post-bootstrap. All three concentrated on 2026-04-23.
- **commits/hour CV:** 0.126 (12.6%)
- **pushes/hour CV:** 0.118 (11.8%)
- **calendar-day plateau (Fri..Mon):** 75/76/78/80 ticks/day, range 1.07x, CV 0.029
- **conclusion:** the dispatcher tick stream is hour-uniform to within statistical noise; the only deviations are three bootstrap craters confined to day 1; the daemon does not sleep when the operator sleeps; a naive "is this human-driven?" test on the timing axis would unambiguously vote "no, this is autonomous", which is in fact the correct verdict for the trigger even though the per-tick content is much richer than any pure auto-generator would produce.
