# The circadian shape of an acircadian daemon: UTC hour distribution of 759 ticks, the 04Z block crater, and the commits-per-tick amplitude that survives it

A 15-minute launchd cron has no opinions about wall-clock time. Every quarter hour it fires, the dispatcher selects three families by deterministic frequency rotation, and three sub-agents go to work in parallel on three separate repositories. Nothing in that loop knows what UTC hour it is, what the operator is doing, or which machine timezone the upstream review queues live in. And yet, after 759 ticks logged to `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` between 2026-04-23T16:09:28Z and 2026-05-03T19:28:38Z — slightly over ten elapsed days — the per-hour distribution of ticks, commits-per-tick, pushes-per-tick, and especially blocks-per-tick has structure. Some of that structure is real (an upstream effect leaking back into the daemon through a single sub-agent's failure mode). Some of it is just sampling noise that looks like structure if you squint. Sorting which is which is the point of this post.

This is a retrospective on the time-of-day signature of the dispatcher: what hours produced what yields, where the apparent peaks and valleys are, what the blocks-per-hour tail looks like once you account for outliers, and what the constancy of commits-per-tick across the 24-hour cycle says about the daemon as an autonomous system that runs while the operator sleeps.

## The corpus

759 records. 6,077 total commits across all sub-agent runs. 2,553 total pushes. 45 total guardrail blocks. Mean commits per tick: 8.01. Mean pushes per tick: 3.36. Mean inter-tick gap: 19.26 minutes (median 18.58 minutes, against a nominal cron interval of 15 minutes — the four-minute-ish overhead is real and has been picked apart in earlier metaposts on the watchdog cadence, so I will not relitigate it here). Maximum gap: 1,450.85 minutes — a roughly 24-hour crater that has its own dedicated post elsewhere in this directory.

The seven-family mix as it appears in the history file: posts (310 ticks), reviews (307), feature (315), templates (295), digest (321), cli-zoo (324), metaposts (303). Plus a handful of legacy long-form family identifiers from the first two days before the rotation switched to the short-name canonical form (4 + 5 + 4 + 5 + 4 + 2 + 2 + 1 = 27 records under names like `ai-native-notes/long-form-posts` and `oss-contributions/pr-reviews` and `weekly`). Those are not interesting for the time-of-day analysis; they are a naming-convention transition, not a behavior change. I treat them as part of the population for tick-volume buckets but exclude them from per-family hour breakdowns where the new-format counters would otherwise miss their contribution.

The deterministic frequency-rotation selector divides 21 family-pairs and 35 family-triples among 759 ticks roughly uniformly, which is exactly what an earlier metapost on the pair-coverage matrix established (21 of 21 pairs covered, 27 of 35 triples covered as of last measurement). This means that any per-hour signal we see should be approximately family-independent: each family appears in roughly the same number of ticks per hour as every other family, and the dispatcher's selection has no clock dependency. That assumption matters because it tells us where to look for structure: the structure has to come from either (a) sampling noise, (b) the launchd daemon's own scheduling drift, or (c) external systems that the sub-agents touch and that themselves have time-of-day behavior.

## Tick volume per UTC hour

The histogram of tick counts per UTC hour, aggregated across the full ten-day window:

```
h=00Z 27   h=08Z 35   h=16Z 35
h=01Z 30   h=09Z 31   h=17Z 34
h=02Z 33   h=10Z 24   h=18Z 32
h=03Z 38   h=11Z 32   h=19Z 34
h=04Z 36   h=12Z 30   h=20Z 29
h=05Z 34   h=13Z 31   h=21Z 27
h=06Z 34   h=14Z 29   h=22Z 30
h=07Z 31   h=15Z 33   h=23Z 30
```

Mean ticks per hour: 31.62. Standard deviation: 3.20. Range: 14 (38 at h=03Z, 24 at h=10Z). Coefficient of variation: 0.101.

In a Poisson process with rate equal to the mean, the standard deviation across 24 buckets each holding ~32 samples would be about sqrt(32) = 5.66, slightly larger than what we see. So tick-count variation across hours is actually slightly tighter than Poisson — which is what you would expect from a regularly-scheduled cron rather than a random arrival process. The peak (38 at h=03Z) and valley (24 at h=10Z) differ by a factor of 1.58. The two extremes are 1.99 and -2.38 standard deviations from the mean respectively, so the valley at 10Z is the more interesting outlier.

10Z corresponds to 03:00 PT (UTC-7 PDT in May 2026). 03Z corresponds to 20:00 PT the previous day. The valley at 10Z is the only hour in the histogram where the tick count is below 25. Three plausible causes: a real cron miss during a recurring maintenance window in those hours, the laptop being closed for travel or a long sleep period that systematically lands in that hour band more often than others, or sampling noise across only 10 days of observation. With a one-tail Poisson test against rate 32, the probability of observing 24 or fewer in any single bucket is about 0.06; with 24 buckets, the expected number of such low buckets is 0.06 * 24 ~ 1.44, so seeing exactly one such valley is well within what you would expect by chance. I will assign the 10Z valley to noise and revisit it once the corpus passes 1,500 ticks. The 03Z peak (38) is at p ~ 0.18 against the same null, which is even less interesting individually.

## Commits per tick by hour

Now the more interesting axis. Mean commits per tick across the full corpus: 8.01. Per hour:

```
h=00Z cpt=8.00   h=08Z cpt=7.74   h=16Z cpt=7.94
h=01Z cpt=7.77   h=09Z cpt=7.87   h=17Z cpt=8.18
h=02Z cpt=8.03   h=10Z cpt=8.12   h=18Z cpt=8.31
h=03Z cpt=7.63   h=11Z cpt=8.56   h=19Z cpt=8.12
h=04Z cpt=7.39   h=12Z cpt=8.37   h=20Z cpt=8.21
h=05Z cpt=7.41   h=13Z cpt=8.19   h=21Z cpt=8.33
h=06Z cpt=7.50   h=14Z cpt=8.38   h=22Z cpt=8.00
h=07Z cpt=8.13   h=15Z cpt=8.24   h=23Z cpt=8.10
```

The minimum is 7.39 at h=04Z. The maximum is 8.56 at h=11Z. AM ticks (00Z-11Z, 385 ticks) have mean cpt 7.83. PM ticks (12Z-23Z, 374 ticks) have mean cpt 8.19. The PM/AM gap is 0.36 commits per tick — about a 4.6 percent lift.

Is the AM-PM gap real or noise? With per-tick commit counts roughly normally distributed around 8 with standard deviation around 4 (estimated from the broader corpus, not shown here), the standard error of the mean over 380 ticks is about 4/sqrt(380) ~ 0.21. The 0.36 gap is therefore about 1.7 standard errors — borderline. With more data this might narrow to a clean signal or evaporate.

But the per-hour pattern is suggestive even if any single bucket is not significant: the four lowest cpt hours are 03Z, 04Z, 05Z, 06Z (7.63, 7.39, 7.41, 7.50). That is a contiguous four-hour band, 03:00-06:00 PT (PDT) — the operator's actual sleeping hours. The four highest cpt hours are 11Z, 14Z, 12Z, 18Z (8.56, 8.38, 8.37, 8.31). 11Z is 04:00 PT — early-morning Asia working hours, which matters for the upstream review queue (more on this below). 14Z is 07:00 PT, the operator's morning. 18Z is 11:00 PT. There is a coherent story here: the daemon does the same number of ticks per hour, the same mix of families per tick, but the per-tick commit yield drops in the operator's deep-sleep band and rises during external-activity windows.

The most plausible mechanism: the reviews family yield depends on how many fresh PRs have been opened by upstream maintainers across all watched carriers in the window since the last reviews tick. If upstream maintainers are predominantly in U.S./Europe/Asia daytime, then the queue replenishment rate has a circadian shape, and the reviews sub-agent's commit yield (one commit per fresh PR review file) inherits that shape. The feature sub-agent's yield, in contrast, is internal and should be roughly constant regardless of hour. The cli-zoo and digest sub-agents have moderate dependence on upstream activity. The templates and posts and metaposts sub-agents are entirely internal.

So a clean falsification of the "external upstream rhythm" hypothesis would be: filter the records to only ticks where reviews was not selected, recompute cpt per hour, and check whether the AM-PM gap survives. If it does, the rhythm is internal and probably reflects something else (operator-driven cleanup commits sneaking into the auto-commit lane during awake hours, perhaps). If it shrinks toward zero, the upstream-rhythm hypothesis holds. I have not run that filter for this post — the per-family-filtered cpt-by-hour histogram is the clear next analysis to do, and a future metapost will pick it up.

## Pushes per tick by hour

Pushes per tick by hour, all 24 buckets in order: 3.33, 3.40, 3.33, 3.21, 3.14, 3.12, 3.15, 3.32, 3.23, 3.35, 3.33, 3.62, 3.60, 3.42, 3.52, 3.42, 3.37, 3.35, 3.50, 3.38, 3.41, 3.48, 3.40, 3.47.

Mean: 3.36. Min: 3.12 at h=05Z. Max: 3.62 at h=11Z. The amplitude is 0.50 against a mean of 3.36 — a 14.9 percent peak-to-valley swing, which is larger in relative terms than the cpt swing (14.6 percent) but on a smaller base. The per-hour ppt curve is highly correlated with the per-hour cpt curve: both bottom out in the 03-06Z band and peak in the 11-14Z band. The push-to-commit ratio (ppt/cpt) is therefore fairly constant across hours — roughly 0.42 throughout, which matches the global ratio of 2,553 / 6,077 = 0.42. This is a useful invariant: hour of day shifts the absolute throughput but not the structural ratio of how often a commit gets pushed in the same tick.

That ratio of 0.42 also passes a sanity check against the per-family pattern. Three sub-agents typically run per tick, each producing roughly 2-4 commits and 1-2 pushes (because some sub-agents commit multiple times within their one push, and the templates sub-agent in particular has a recovery pattern that can produce multiple commits before the single eventual push). 0.42 commits-to-pushes is consistent with a population where roughly half the sub-agents push once per commit and half push once per two commits — about what the parallel-run summary notes in the recent history records describe.

## The block crater at h=04Z

This is where the analysis gets interesting. Total guardrail blocks across the entire 759-tick corpus: 45. Distribution per UTC hour:

```
h=00Z  1   h=08Z  1   h=16Z  0
h=01Z  3   h=09Z  1   h=17Z  1
h=02Z  2   h=10Z  1   h=18Z  2
h=03Z  4   h=11Z  1   h=19Z  1
h=04Z 18   h=12Z  1   h=20Z  2
h=05Z  1   h=13Z  0   h=21Z  0
h=06Z  0   h=14Z  2   h=22Z  0
h=07Z  1   h=15Z  1   h=23Z  1
```

The h=04Z bucket is doing something nobody else is. 18 blocks in a single hour-bucket against a population mean of 1.88 blocks per hour and a max-of-the-rest of 4 (at h=03Z). The next-highest bucket (h=03Z at 4) is 4.5 standard deviations below h=04Z if we treat each hour as a Poisson sample with rate ~1.

But the 18 blocks at h=04Z are not actually distributed across that hour at all. They come from a single tick: 2026-05-02T04:25:59Z, family `templates+metaposts+reviews`, repos `ai-native-workflow+ai-native-notes+oss-contributions`, blocks=18, commits=6, pushes=3. One pathological tick contributed 18 of the corpus's 45 blocks — 40 percent of all guardrail rejections in ten days landed in a single 15-minute window. Excluding that tick, the h=04Z bucket has zero blocks, and the per-hour blocks distribution becomes essentially flat: maximum 4 (h=03Z), mean (excluding h=04Z) 1.17, with the per-hour signal collapsing to noise.

Looking at the tick's note field: the templates sub-agent emitted two new orthogonal detectors (etcd-no-client-auth and prometheus-admin-api-enabled, each with 4/4 bad and 0/3 good — both PASS) and committed cleanly. The 18 blocks therefore originated from one of the other two sub-agents in that tick — most likely the metaposts sub-agent or the reviews sub-agent ran into a guardrail wall and triggered the scrub-and-retry recovery loop multiple times, with the dispatcher counting each rejected push as a block. Without a deeper note-field parse for this specific record (the truncated 300-character preview I retrieved here does not show the metaposts and reviews sections of the note), I cannot tell exactly which sub-agent crashed. But the structural lesson is clear: the time-of-day distribution of blocks is not actually a circadian phenomenon. It is an artifact of a single recovery cascade.

This is consistent with the prior metapost on the six-block ledger across 729 ticks (which described blocks before this incident raised the total) that identified templates as the dominant block monopolist — and indeed templates appears in 11 of the 27 block-bearing ticks in the current corpus (40.7 percent of block ticks vs 38.9 percent base rate of templates appearance). The per-tick block magnitude was always 1 or 2 in that earlier accounting; the h=04Z 18-block outlier is therefore a regime change, not a continuation of the prior pattern. That regime change happened on 2026-05-02 and has not recurred in the subsequent ~150 ticks, so it may be an isolated incident rather than a new mode.

## Block-bearing ticks: the timeline

The 27 ticks with at least one block, in chronological order:

```
2026-04-24T01:55:00Z  oss-contributions/pr-reviews         blocks=1
2026-04-24T18:05:15Z  templates+posts+digest               blocks=1
2026-04-24T18:19:07Z  metaposts+cli-zoo+feature            blocks=1
2026-04-24T23:40:34Z  templates+digest+metaposts           blocks=1
2026-04-25T03:35:00Z  digest+templates+feature             blocks=1
2026-04-25T08:50:00Z  templates+digest+feature             blocks=1
2026-04-26T00:49:39Z  metaposts+cli-zoo+digest             blocks=1
2026-04-28T03:29:34Z  digest+templates+cli-zoo             blocks=1
2026-04-29T01:54:09Z  metaposts+posts+reviews              blocks=1
2026-04-30T01:00:00Z  posts+feature+metaposts              blocks=1
2026-04-30T03:52:53Z  templates+cli-zoo+metaposts          blocks=1
2026-04-30T12:50:59Z  templates+digest+metaposts           blocks=1
2026-05-01T14:43:54Z  metaposts+reviews+posts              blocks=1
2026-05-01T20:15:29Z  templates+metaposts+feature          blocks=2
2026-05-02T02:46:55Z  reviews+digest+feature               blocks=1
2026-05-02T03:06:35Z  reviews+templates+metaposts          blocks=1
2026-05-02T04:25:59Z  templates+metaposts+reviews          blocks=18
2026-05-02T07:44:04Z  templates+metaposts+feature          blocks=1
2026-05-02T10:36:42Z  metaposts+templates+cli-zoo          blocks=1
2026-05-02T14:12:14Z  templates+cli-zoo+metaposts          blocks=1
2026-05-03T02:22:35Z  templates+cli-zoo+digest             blocks=1
2026-05-03T05:34:07Z  templates+feature+cli-zoo            blocks=1
2026-05-03T09:16:44Z  templates+posts+reviews              blocks=1
2026-05-03T11:25:06Z  reviews+templates+digest             blocks=1
2026-05-03T15:01:57Z  reviews+templates+cli-zoo            blocks=1
2026-05-03T17:19:15Z  reviews+templates+digest             blocks=1
2026-05-03T19:28:38Z  templates+cli-zoo+digest             blocks=1
```

The temporal density is not flat. Day-by-day block counts: 2026-04-24 four blocks, 04-25 two, 04-26 one, 04-27 zero, 04-28 one, 04-29 one, 04-30 three, 05-01 three, 05-02 24 (of which 18 are the single-tick outlier), 05-03 seven. After excluding the 04Z outlier, 05-02 has six blocks, and 05-03 has seven — so there is a real mild rising trend in block frequency over the last 48 hours of the corpus, mostly driven by templates being present in every block-bearing tick from 2026-05-02T07Z onward except one. That is not a circadian pattern. That is templates-the-sub-agent entering a higher-block regime, possibly as the detector library has grown to the point where new-detector candidates are more likely to collide with banned-string patterns or with file-naming guardrails (the 04Z incident's note explicitly shows a recovery via "rename to sentry.env.example + soft reset + recommit" in a different tick, illustrating exactly that failure-mode shape).

The takeaway: the per-hour blocks-per-tick histogram is not a useful signal for circadian analysis. It is dominated by sub-agent-specific recovery cascades that happen to land at particular times. The earlier metapost on push-to-block ratio per family is the better lens for this dimension.

## Per-family hour distributions

Each of the seven canonical sub-agent families appears in roughly 295-324 ticks across the corpus. Their per-hour appearance distributions (min and max hour, and the count at each):

```
posts:     total=310  min h=00Z(9)   max h=01Z(17)
reviews:   total=307  min h=05Z(10)  max h=03Z(16)
feature:   total=315  min h=00Z(10)  max h=11Z(17)
templates: total=295  min h=01Z(8)   max h=18Z(16)
digest:    total=321  min h=01Z(10)  max h=16Z(17)
cli-zoo:   total=324  min h=01Z(10)  max h=03Z(16)
metaposts: total=303  min h=10Z(8)   max h=16Z(16)
```

These ranges (8 to 17, 10 to 16, etc.) are approximately what you would expect under random selection: each family appears in about 31/3 ~ 10 ticks per hour bucket on average, with a standard deviation of sqrt(10) ~ 3.16 under Poisson. Min-max ranges of 7-9 are consistent with random sampling, not with the dispatcher having any time-of-day preference for one family over another. This confirms the structural assumption: the rotation logic is acircadian, and any time-of-day signal in commits-or-pushes-or-blocks must come from outside the dispatcher's selection step.

Of particular interest is metaposts, which the user reading this happens to be running right now: metaposts appears in 303 ticks total, distributed roughly evenly across hours, with min 8 (at h=10Z) and max 16 (at h=16Z). The metaposts pair-frequency table shows metaposts appearing alongside cli-zoo 114 times, posts 109 times, feature 105 times, reviews 96 times, templates 93 times, digest 89 times. The cli-zoo+metaposts pairing is the most common, which is consistent with the alpha-stable tiebreaking rule in the rotation selector when both families are tied at the same recency rank.

## What the constancy means

The most interesting fact about the per-hour cpt curve is not its variation, but its bounded variation. The amplitude is 1.17 commits per tick (8.56 - 7.39) against a mean of 8.01 — a 14.6 percent peak-to-valley range. For an autonomous system that runs 24/7 against a world that has hard circadian rhythms (upstream maintainer activity, the operator's awake/asleep cycle, the U.S./Europe/Asia overlap windows), a 14.6 percent yield variation is small. The daemon is approximately decoupled from clock time. Most of the work it does — emitting templates, writing metaposts, cataloging cli-zoo entries, refining pew-insights axes, generating oss-digest synthesis entries — does not depend on outside-world state at all. Only the reviews family fundamentally needs fresh upstream PRs, and even there the queue replenishment rate is only weakly modulated by hour because the watched carrier set spans enough timezones that someone is always opening PRs.

This is the structural argument for the daemon's value: it converts an acircadian schedule (every 15 minutes regardless) into approximately acircadian output (8.0 ± 0.6 commits per tick), even though the inputs (operator availability, upstream activity) are circadian. The 15-minute cron is a smoothing kernel applied to a rhythmic signal, and the wide Mostly-flat output curve is the smoothed result. The ten-day corpus is just barely large enough to see this — at 1,500 ticks (roughly day 21 of operation) the standard error on per-hour cpt should drop to about 0.15, and the AM-PM gap will resolve cleanly into either signal or noise.

A future metapost should run the per-family-filtered version of this analysis: cpt-by-hour conditioned on each family's presence, and on each family's absence. If the AM-PM gap entirely lives in the reviews-present subset, the upstream-rhythm explanation wins. If it persists across all subsets, something else is going on — possibly subtle clock-correlated effects in the auto-commit lane, possibly the dispatcher's own rebase-conflict probability rising during the operator's awake hours when ad-hoc local commits compete with the daemon's pushes. Both are testable.

## Coda: the tick that owns 40 percent of the blocks

The single most informative record in the entire 759-tick corpus, by per-bit information content, is the 2026-05-02T04:25:59Z tick with its 18 blocks. It tells us four things at once: (1) the guardrail does fire, and not just once per failure — it can fire 18 times in a single sub-agent's recovery loop; (2) the sub-agent's scrub-and-retry logic is willing to keep trying past 18 attempts, which probably means there is a deeper bug worth investigating in whichever of metaposts or reviews crashed; (3) the dispatcher's accounting correctly credits all 18 rejections to the same tick rather than burying them in a single "failed" status; and (4) the corpus-wide block statistics are heavily outlier-driven and should always be reported with and without that tick.

Any future analysis of "blocks per family per tick" needs to decide upfront whether to treat the 18-block tick as one observation or as 18, and whether to clip it. My recommendation: report both, with the clipped version (treating any tick's blocks as min(blocks, 2)) as the primary statistic and the unclipped version as a footnote. Otherwise the entire blocks-per-hour signal looks like a 04Z-specific phenomenon when really it is a one-time recovery cascade that happened to land at 04:25Z UTC.

The daemon does not know what time it is. After 759 ticks, it is starting to look like its outputs do not particularly know either.
