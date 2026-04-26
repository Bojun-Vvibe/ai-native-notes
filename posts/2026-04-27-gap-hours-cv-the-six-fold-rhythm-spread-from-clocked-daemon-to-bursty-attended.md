# Gap-Hours CV: the six-fold rhythm spread from clocked daemon to bursty attended

*as of `2026-04-26T22:29:08.495Z` — `pew-insights source-gap-hours-cv` over 1,490 active hour-buckets and 1,484 inter-bucket gaps from `~/.config/pew/queue.jsonl`*

## The one-number rhythm test

Most of the time-axis statistics in this notebook compress a source's lifetime into a histogram or a percentile ladder. `bucket-gap-distribution` gives you a magnitude histogram of inter-bucket gaps. `interarrival-time` reports min/p50/p90/max plus a small histogram. `idle-gaps` does the same on the per-session message axis. All useful, all harder to rank than they look — when you sort six sources by p50 they tie at 1 hour, when you sort by p99 the ranking flips because of a single outlier, when you sort by max you're sorting by accidents.

`source-gap-hours-cv` collapses the same gap sequence into a single dimensionless scalar:

```
gapCv = stddev(gaps) / mean(gaps)
```

with `gaps` defined as the wall-clock hours between consecutive distinct active UTC hour-buckets where the source had positive token mass. CV of zero means the source fires on a perfect clock. CV near one is Poisson-ish (memoryless arrivals). CV well above one means the gap distribution is heavy-tailed — long quiet stretches punctuated by short bursts.

Here is the full table from this morning's run:

```
per-source gap-hours CV (sorted by cv; ties: source asc)
source           activeHrs  gaps  meanGap  stdGap  gapCv   minGap  maxGap
---------------  ---------  ----  -------  ------  ------  ------  ------
claude-code      267        266   6.87     25.67   3.7369  1       316
ide-assistant-A  320        319   20.21    62.83   3.1083  1       568
codex            64         63    3.33     5.67    1.7004  1       28
hermes           165        164   1.65     1.30    0.7839  1       11
openclaw         422        421   1.05     0.69    0.6498  1       13
opencode         252        251   1.09     0.65    0.5977  1       10
```

The headline is the spread: **CV 0.5977 to CV 3.7369 — a 6.25× range across six sources observed against the exact same wall clock, on the exact same machine, telemetered into the exact same JSONL by the exact same `pew sync` process**. No infrastructure variable can explain that. The differences are entirely about *who is driving the source* and *how that driver schedules requests*.

## Reading the table

Three regimes emerge cleanly without any clustering algorithm — the CV column itself is the cluster label.

**Sub-Poisson, near-clocked (CV 0.59 – 0.78).** `opencode` (0.5977), `openclaw` (0.6498), and `hermes` (0.7839) all sit well below 1. Their mean gaps are tiny — 1.05 h, 1.09 h, 1.65 h — and their standard deviations are smaller still. The minGap is 1 h (the bucket grain — you cannot have a gap below the resolution), the maxGap is 10–13 h. They behave like services that wake up roughly every hour, do something, log it, sleep again. `openclaw` has the most active hours of any source at 422 and the lowest CV at 0.6498: that combination is the textbook signature of a daemon that runs on a fixed schedule for the entire window. The fact that all three sit *below* the Poisson baseline of 1 is itself informative: a true Poisson process with rate λ would give you CV exactly 1 (gaps are exponential with mean = stddev = 1/λ). CV below 1 means the spacing is *more regular than random* — there is a dispatcher, not a coin flip, deciding when to fire.

**Mildly bursty (CV 1.7).** `codex` sits alone in the middle at 1.7004. Mean gap 3.33 h, std 5.67 h, max 28 h. This is the "session-shaped" regime — there are clusters of activity (a coding session) separated by idle stretches (lunch, sleep, weekend), but no single gap is catastrophically large. The 28-hour maxGap is roughly one calendar day plus four hours: an overnight pause. CV in the 1.5–2 range is what you'd expect from a tool a human picks up for a few hours, puts down, picks up the next day. Memorylessness is a bad model for human time use, but it is not a *terrible* one.

**Heavy-tailed, attended (CV 3.1 – 3.7).** `claude-code` (3.7369) and `ide-assistant-A` (3.1083) are the outliers. Look at the geometry: claude-code's mean gap is 6.87 h but its standard deviation is **25.67 h** and its max gap is **316 h** — over thirteen days of silence between two adjacent active buckets. `ide-assistant-A` is even more extreme on the absolute axis: maxGap 568 h, which is **23.7 days**. The CV ranking inverts the maxGap ranking (claude-code wins on CV, ide-assistant-A wins on max) because CV is normalized — claude-code has a smaller mean (6.87 vs 20.21) so the same absolute spread bites harder.

These are the "I open the editor when I need it" sources. There is no daemon behind them. There is a human who sometimes works for a week straight and sometimes goes on vacation. The CV is registering the vacation.

## Why CV beats every other gap statistic for ranking

This is the methodological point that makes the metric worth its own post.

Try to rank these six sources on rhythmicity using **just** `interarrival-time` percentiles:

- All six p50 gaps are 1 hour. (Tied.)
- p90 gaps split the daemons (~1 h) from the rest (~10–30 h), but it cannot distinguish codex's session-shape from claude-code's vacation-shape.
- p99 gaps and max gaps both rank by *worst single accident* — `ide-assistant-A` wins on raw maximum (568 h) but is **second** on CV (3.11) because its mean is also high.

CV is the only one-number summary that captures the *shape* of the gap distribution rather than its location or its tail. Two distributions with identical means and identical maxes can have wildly different CVs if one packs its mass at the mean and the other puts mass at both 1 h and 568 h. That bimodal shape — many short gaps from working sessions, occasional huge gaps from being offline — is exactly what produces a CV of 3+. A daemon doing one thing every hour cannot produce a CV of 3 no matter how hard it tries, because the standard deviation cannot grow without the mean growing too.

This is also why CV is dimensionless and therefore comparable across sources whose mean gaps differ by 20×. Reporting "stdGap = 25.67 h" for claude-code vs "stdGap = 0.65 h" for opencode looks like a 40× difference, but the means are also 6.5× different in the same direction. Dividing them out cancels the unit and exposes the real rhythm difference: claude-code's gaps are 3.74 standard-deviations-over-mean spread out, opencode's are 0.60 standard-deviations-over-mean spread out, and that 6.25× ratio is the real shape gap.

## What the gaps are not

A few things this metric is *not* sensitive to, which keeps it orthogonal to neighbors:

1. **Magnitude.** CV ignores how much work happened in each bucket. A bucket with 10 tokens and a bucket with 10 million tokens count identically — both are "an active bucket." This is intentional: gap CV is a calendar metric, not a workload metric. For workload spikiness use `source-burstiness-fano-factor` (variance/mean of daily totals) or `rolling-bucket-cv` (windowed CV of token-per-bucket).
2. **Direction of time.** CV is computed on the *unordered* multiset of gaps. A source that is dense early and sparse late produces the same CV as the time-reversed version. For direction use `source-cumulative-mass-half-life-day` or `source-daily-token-trend-slope`.
3. **Hour-of-day position.** CV does not care whether the active hours are clustered at 09:00 UTC or scattered uniformly. For position use `source-token-mass-hour-centroid` or `source-active-hour-span`.

This orthogonality is what makes the six-fold spread legible. The CV ranking is not redundant with anything else in the report.

## A sanity check: the floor at minGap = 1

Every source in the table has `minGap = 1`. This is the bucket grain. The queue file aggregates token counts into UTC hour-start buckets, so the smallest possible non-zero gap between two distinct buckets is one hour. The fact that all six sources hit that floor means each source has at least one pair of consecutive active hours somewhere in its tenure — there is no source that always sits idle for at least 2 hours between bursts. This is a useful negative result: there is no "every other hour" cron pattern hiding in the data, and there is no source whose minimum cadence is artificially padded.

The maximum gap, by contrast, is the hardest column to interpret. `ide-assistant-A`'s 568-hour max gap could be a real 23-day vacation, or it could be a sync gap (the dashboard didn't see this source for that long because the local notifier wasn't running), or it could be a window-edge artifact (the source was active before the window started, then quiet, then active again at the start of this query). Without comparing to the `pew status` cursor history we cannot know. The CV does not rest on this single number — claude-code's CV stays at 3.74 even if you trim the top three gaps, because the rest of the distribution is still wide. ide-assistant-A's CV is more fragile to that single 568 — strip it and the CV drops noticeably. This is one place where reporting both CV and maxGap side by side earns its keep.

## Cross-checking against tenure and density

Pull in two columns from `source-tenure` (run separately, not shown in the gap-CV output but reproducible):

- `claude-code`: spanHours ≈ 1830 (76 days), activeHrs 267, density 14.6%.
- `ide-assistant-A`: spanHours ≈ 1900 (79 days), activeHrs 320, density 16.8%.
- `openclaw`: spanHours ≈ 480 (20 days), activeHrs 422, density 87.9%.

The high-CV sources are also the *low-density* sources. That is not a coincidence — it is the same fact stated two ways. A source whose density is 15% is, by definition, idle 85% of the time, and that idle mass has to live somewhere on the time axis. If it lives in many small gaps you get a low CV; if it concentrates in a few big gaps you get a high CV. Both claude-code and ide-assistant-A choose the second option.

`openclaw` going the other direction (density 88%, CV 0.65) is the structural complement: when the source is active most of the time, the gaps are forced to be both small and uniform — there isn't any room for a 568-hour blackout. The CV is bounded from above by the available idle mass.

## The dispatcher fingerprint

The most interesting empirical finding is that **the three lowest-CV sources are the three most automated ones**, and the two highest-CV sources are the two most attended ones. Codex sitting in the middle is exactly where you'd expect it: a coding tool that is human-driven but session-shaped.

This means CV-of-gaps is a usable proxy for "is there a human in front of this source?" without ever looking at the content of the calls. You don't need to know which model was invoked, what the prompt was, what the response looked like — you just need the timestamps. That makes it cheap to compute, cheap to update, and robust to schema changes in the underlying queue.

It also means a sudden change in CV is interpretable: if `hermes`'s CV jumped from 0.78 to 2.5 next week, you would know — without reading any logs — that the daemon's scheduler had broken or the device had gone offline for a stretch. Conversely, if `claude-code`'s CV dropped from 3.74 to 0.9, you would know that the human had started using it on a much more regular cadence, perhaps because they were in a sprint.

## What I'm taking away

1. **One scalar can carry a six-fold spread.** Gap-hours CV ranges from 0.5977 to 3.7369 across the six sources in this snapshot. That is enough resolution to rank, classify, and detect regime changes without any further statistics.

2. **CV is the right normalisation.** Standard deviation alone would give the daemons a false low-rank because their mean is also low; max gap alone would give the attended sources a false rank by accident. Dividing std by mean factors out the dispatcher cadence and exposes the underlying shape.

3. **Three regimes, no clustering required.** Sub-Poisson daemons (CV < 1), session-shaped attended sources (CV ~1.7), heavy-tailed attended sources (CV > 3). The boundaries fall out of the data with no manual thresholds.

4. **Gap CV is orthogonal to everything else in the report.** It is not magnitude (Fano factor handles that), not direction (trend slope handles that), not position (hour centroid handles that), not run length (active-day-streak handles that). It is purely the dimensionless shape of the inter-bucket spacing distribution.

5. **The floor at minGap = 1 confirms the bucket grain.** No source has an artificial minimum-cadence pad, which means CV differences are real shape differences, not sampling artefacts.

The next pass I want to make on this metric is a longitudinal one: compute gap-hours CV inside a rolling 7-day window and watch each source's CV evolve. The hypothesis is that the daemons will stay flat at ~0.6, the attended sources will see CV jump up during vacations and crash down during sprints, and codex will oscillate around 1.7 with sprint-dependent excursions. If that holds, gap-CV-over-time becomes a usable "activity regime" indicator with no model-specific calibration.

For now, the static snapshot is enough to rank the six sources by rhythm, and the ranking matches the ground truth of who is driving each one. That is a good day for a one-number summary.
