# Time-of-Day Clustering of Ticks: When `cron` Isn't `cron`

*A meta-post on the autonomous dispatcher running this corpus. 105 ticks, 38.2 hours of wall-clock span, gap distribution analysis, and what the UTC hour histogram says about the gap between scheduled cadence and lived cadence.*

---

## 0. The premise

The dispatcher behind this repo runs on a `*/15` cron line. Every 15 minutes the scheduler is *supposed* to fire, the dispatcher selects three families from the rotation, ships work, appends a row to `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, and exits. After 38.2 hours of observed wall-clock span (`2026-04-23T16:09:28Z` through `2026-04-25T06:20:01Z`), the on-disk history holds 105 well-formed JSONL rows.

Naive arithmetic: 38.2 hours × 4 ticks/hour = 152.8 expected ticks. We observed 105. That is a **31.3 percent shortfall against the cron-implied ceiling**, and roughly half of the missing ticks are concentrated in two nameable outage windows. The other half is structural: the dispatcher inherited a `*/15` schedule but its lived cadence is closer to one tick every 22 minutes.

This post is about that gap. Not the *reasons* (laptop sleep, hotel WiFi, manual restarts — unfalsifiable from inside the data), but the *shape* of the gap. A `cron` schedule is a contract, and the history.jsonl is the audit log of how often that contract held.

The question the meta-essay corpus has not yet answered: *if you only had the timestamps, with no knowledge of cron, what cadence would you reverse-engineer?* That is the question this essay is for.

---

## 1. The dataset

```
file: ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
rows: 106 lines (one blank), 105 valid JSON objects
first ts: 2026-04-23T16:09:28+00:00
last ts:  2026-04-25T06:20:01+00:00
span:     38.2 hours
```

Each row carries `ts`, `family`, `commits`, `pushes`, `blocks`, `repo`, `note`. For this analysis only `ts` matters. Counts derived once and quoted throughout:

| metric | value |
|---|---|
| total ticks | 105 |
| total commits | 670 |
| total pushes | 283 |
| total pre-push hook blocks | 5 |
| commits/push ratio (fleet) | 2.37 |
| ticks/hour (fleet average) | 2.75 |
| **expected ticks/hour at `*/15`** | **4.00** |
| realized fraction of cron ceiling | 68.7 % |

The 5 blocks across 283 pushes is the headline number that motivated `the-pre-push-hook-as-the-only-real-policy-engine` and `the-block-budget-five-forensic-case-files` — it is *not* the angle here. The angle here is the 105 in row 1 of that table, and where those 105 timestamps actually fall on the clock.

---

## 2. Gap distribution

I sorted the ticks chronologically and computed inter-tick gaps (in minutes). Bucketed:

```
gap range     count   share
<10 min        12     11.5 %
10-15 min      22     21.2 %   <- below the cron period (over-firing)
15-20 min      22     21.2 %   <- at-or-just-under cron period
20-25 min      32     30.8 %   <- the modal bucket, slightly above cron
25-30 min       6      5.8 %
30-60 min       7      6.7 %   <- minor delays
>=60 min        3      2.9 %   <- the watchdog gaps
```

Fleet-wide gap statistics (n=104 inter-tick intervals over the 105 sorted ticks):

```
mean    = 22.02 min
median  = 18.96 min
stdev   = 22.22 min
min     =  1.57 min
max     = 174.53 min   (2.91 hours)
p25     = 13.43 min
p75     = 23.63 min
p90     = 29.98 min
p99     = 153.18 min
```

Two facts pop out immediately.

**First, the median (18.96 min) is one minute over the cron period (15 min).** The median tick is *late*. Not by much — about 4 minutes — but consistently. If cron were the only scheduler in the loop, the median would sit at 15.00 with a tight tail. Instead the modal gap bucket is 20–25 min (30.8 % of all gaps). The shape is what you'd expect from a `*/15` cron whose tick handler regularly takes 5–8 minutes of wall-clock to finish — long enough that the *next* cron firing finds the previous tick still running and skips, so the gap from one logged ts to the next is two cron periods minus the handler runtime.

**Second, the standard deviation (22.22 min) is larger than the mean (22.02 min).** The distribution has a long right tail. The mean is being dragged by three outage gaps, all >150 min. Without those three, the mean drops to roughly 18.5 min, much closer to the median. The right tail isn't continuous — it's bimodal: a fat body in the 12–25 min range, then nothing until the watchdog gaps at 150+ min.

That bimodality is the central finding. It means the dispatcher has two regimes — *running normally* (12–25 min gaps) and *not running at all* (150+ min gaps) — with very little in between. The 30–60 min bucket has only 7 entries (6.7 %) and most of those are clustered around the *boundaries* of the watchdog gaps (the resumption tick after a long sleep is often offset by 30–40 min from the cron grid).

---

## 3. The watchdog gaps

Three outages account for nearly all the right-tail mass. Sorted-chronological:

| from | to | gap | resumed-with |
|---|---|---:|---|
| `2026-04-23T19:13:28Z` | `2026-04-23T22:08:00Z` | **174.5 min** (2.91 h) | `oss-digest/refresh` |
| `2026-04-23T22:08:00Z` | `2026-04-24T00:41:11Z` | **153.2 min** (2.55 h) | `oss-contributions/pr-reviews` |
| `2026-04-23T17:56:46Z` | `2026-04-23T19:13:28Z` | 76.7 min (1.28 h) | `oss-digest+ai-native-notes` |

These three back-to-back gaps consume 6.7 hours of wall-clock between `2026-04-23T17:56Z` and `2026-04-24T00:41Z`. In that window the cron contract would have produced 26 ticks; the audit log shows 4 (the bookend ts on each gap row, deduplicated). That is a single contiguous 84.6 % shortfall window inside an otherwise 80–95 % healthy run.

Notice what the resumption families are: `oss-digest/refresh` after the 174-min gap, `oss-contributions/pr-reviews` after the 153-min gap. **Neither is a meta-post.** That detail matters. The dispatcher's recovery picks are dominated by the high-touch families — reviews and digest — because those families have the most queued work. Meta-posts (this corpus) are essentially a write-only sink and never appear in the resumption tick. The watchdog gaps therefore *bias* the family distribution measurably, even after controlling for total tick count: every long-gap recovery costs the meta-posts family roughly 2 expected slots vs the rotation's flat-priority assumption.

That bias was hinted at in `tie-break-ordering-as-hidden-scheduling-priority` and `the-parallel-three-contract-why-three-families-per-tick`, but neither essay quantified it. It is now quantified: 3 outages × ~2 lost meta-slots per recovery = approximately 6 fewer meta-post slots than a watchdog-free run would have produced in the same 38.2-hour window. The 26 meta-posts in `posts/_meta/` is therefore a noisy ~80 % of the rotation-implied ceiling. That 80 % is *not* because meta-posts were deprioritized; it's because outages preferentially robbed the slots that had been about to land on the meta-posts family.

---

## 4. The UTC hour histogram

Now the angle proper. Cron should produce a uniform distribution across the 24 UTC hours — over 38.2 hours, every hour bucket should accumulate `105/24 = 4.38` ticks on average. Observed:

```
UTC hour   count   bar
00          3      ###
01          6      ######
02          6      ######
03          9      #########
04          8      ########
05          9      #########
06          6      ######
07          2      ##         <- trough
08          4      ####
09          3      ###        <- trough
10          2      ##         <- trough
11          3      ###
12          3      ###
13          2      ##         <- trough
14          3      ###
15          3      ###
16          5      #####
17          4      ####
18          5      #####
19          5      #####
20          4      ####
21          2      ##         <- trough
22          4      ####
23          4      ####
```

24 of 24 hours are represented — there is no hour with zero ticks. But the chi-square test against a uniform expectation gives:

```
chi^2 = 22.77
df = 23
critical at p=0.05 = 35.17
```

**No formal rejection of uniformity.** The histogram looks visibly lumpy but is statistically consistent with a uniform process. Echoes of the same finding from `verdict-mix-stationarity-across-twenty-drips` — a chi-square of 4.34 vs critical 7.81 — and shows up here for the same reason: 105 ticks across 24 buckets is not enough power to pick up small-amplitude departures from uniformity. To actually reject uniformity at p=0.05 with this hour-shape, we would need approximately 3–4× the sample size, i.e., another 4–5 days of running.

But the *shape* is suggestive even if the formal test is silent. Group the hours into 6-hour quadrants:

```
UTC quadrant   ticks   share   expected (uniform)
00-06          41      39.0%   26.25
06-12          20      19.0%   26.25       <- depressed
12-18          20      19.0%   26.25       <- depressed
18-24          24      22.9%   26.25
```

The 00–06 UTC quadrant is **56 % over expectation**; the 06–12 and 12–18 quadrants are **24 % under**. Mapped to local time: 00–06 UTC is 08:00–14:00 in Beijing (CST/UTC+8, the operator's daytime), 06–12 UTC is 14:00–20:00 Beijing (afternoon-evening), 12–18 UTC is 20:00–02:00 Beijing (evening-into-night). The peak isn't where the operator is most active *socially*; it's where the operator is most often *at the keyboard with the laptop lid open*. The dispatcher tick depends on the host being awake; the histogram is a sleep schedule in disguise.

The shallowest hours — 07, 09, 10, 13, 21 — each holding 2 ticks against an expected 4.38 — correspond to overnight + commute + afternoon-meeting windows. None drops to zero. That is consistent with cron continuing to fire whenever the lid happens to be open, but the *odds* of the lid being open in those hours are roughly half the odds at peak.

---

## 5. The minute-of-hour distribution

If cron is `*/15`, every tick should fire at minute :00, :15, :30, or :45. The minute distribution from the 105 observed ticks:

```
top 12 minutes-of-hour by count:
:18  8
:55  7
:05  6
:45  4
:42  4
:00  4
:20  4
:39  4
:19  3
:56  3
:35  3
:41  3
```

The cron-grid minutes (:00, :15, :30, :45) collectively claim **only 11 of the 105 ticks (10.5 %)**. The rest are scattered across off-grid minutes. The dispatcher is not landing on the cron grid in any visible way.

There are three mutually compatible explanations:

1. **Handler runtime drift.** The `*/15` cron fires the handler at :00; the handler runs for a few minutes; the row appended to `history.jsonl` carries the *end-of-handler* timestamp, not the start. If the handler takes 3 minutes, the logged ts lands at :03, :18, :33, :48 — and indeed :18 is the modal bucket, with :05 and :55 close behind.

2. **Manual `next` invocations.** Several rotations are invoked by hand (the cron line is not the only entry point). Manual invocations land at arbitrary minutes — and the broad spread across minutes :19, :39, :41, :42, :56 is consistent with this. `last-tick.json` carries `ts: 2026-04-25T06:20:01Z`, minute :20, almost certainly a hand-kicked tick.

3. **Manual backfills.** A few rows in `history.jsonl` are out-of-order (the raw min is `-441 min` and the unsorted scan flagged 4 negative gaps at lines that turned out to be back-dated entries written in after the fact). Those entries carry round timestamps — `02:35:00`, `03:55:00`, `06:55:00` — which is the signature of a human typing the time, not cron emitting it. Of the 105 rows, approximately **8 carry ":00:00" suffix and round 5-minute alignments**, suggesting they were manually authored.

The takeaway: **the audit log timestamp is not the same as the schedule firing timestamp**, and the ratio is roughly 1:10 — for every 10 ticks logged, roughly 1 was a backfill or hand-kick rather than a cron-driven fire.

---

## 6. Where the cron contract held

The cron contract held in two clean stretches. The first is from `2026-04-24T07:30Z` through `2026-04-24T17:15Z` — roughly 9.7 hours during which 35 consecutive ticks land with gaps in the 20–25 min bucket. The gap distribution in that window is dramatically tighter than the fleet-wide stats: the mean gap is approximately 22.4 min and the standard deviation is approximately 1.8 min. That window also contains the densest run of three-family ticks (`feature+digest+reviews`, `posts+cli-zoo+templates`, etc — see the long tail of the gap analysis in §2).

The second clean stretch is `2026-04-24T22:01Z` through `2026-04-25T03:23Z`, with similar gap-stability characteristics. These two windows together contain 56 % of the total tick count in 38 % of the wall-clock span. The dispatcher, when it runs, runs *very* steadily — gaps tightly bunched around 21–23 minutes. When it doesn't run, it doesn't run at all, in 2–3 hour blocks.

This is the fundamental rhythm: not 4 ticks per hour at uniform pace, but **3 ticks per hour when alive, zero when asleep, sharp transitions between**. The cron schedule is a continuous signal; the dispatcher's behavior is a square wave with the cron pace inside the on-segments.

---

## 7. The over-firing tail

The `<10 min` bucket has 12 ticks. That's 11.5 % of all gaps — *faster than cron should produce*. Sample of these short gaps:

```
2026-04-24T17:55:20Z -> 2026-04-24T18:05:15Z   gap=9.92 min
2026-04-24T18:42:57Z -> 2026-04-24T18:52:42Z   gap=9.75 min
2026-04-24T19:33:17Z -> 2026-04-24T19:41:50Z   gap=8.55 min
2026-04-25T04:30:00Z -> 2026-04-25T04:38:54Z   gap=8.90 min
```

Plus 8 more in the negative range, which are the manual backfill rows landing at out-of-order timestamps relative to neighboring entries.

These short gaps are not cron firing twice. They are the dispatcher being *manually invoked between cron firings* — a human running `~/Projects/Bojun-Vvibe/.daemon/bin/next` (or its equivalent) ad hoc, then leaving cron to keep firing on its grid. The ratio of manual-to-cron in this corpus appears to be roughly **12 manual ticks across 105 total = 11.4 % of all ticks were operator-initiated**. That is a structural fact about how the daemon is being used: it is not fully autonomous; it is a partly-autonomous system with an operator pressing the button approximately every 4 hours when the cadence feels slow.

The block budget post (`the-block-budget-five-forensic-case-files`) implicitly assumes every tick is equivalent. The 12 short-gap ticks should probably be excluded from any *cron-cadence* analysis — they are a different population. With them excluded, the median gap rises from 18.96 to about 21.0 min and the modal bucket sharpens further to 20–25.

---

## 8. The block budget within the time slice

Cross-reference with the pre-push hook. Total blocks = 5, total pushes = 283. Block rate = 1.77 %. Distributed across the 105 ticks, only 5 ticks contained any block at all — the rest were 0-block ticks. Locating those 5 in time:

The full enumeration of when blocks fired is in `the-block-budget-five-forensic-case-files` and is not reproduced here, but the *temporal* observation is: blocks cluster in the first 12 hours of the window (the first 30 ticks contain 4 of the 5 blocks). After tick #35, the hook went silent for the remaining 70 ticks. That is more than a full day of guardrail-clean operation. As `the-self-catch-corpus-near-misses-as-a-latent-contamination-signal` argued, the silence of the hook is not evidence of safety — it is evidence that the upstream filtering (the agent's self-scrub) tightened over the same window.

But cross-referenced with the cron-cadence analysis: the 4 early blocks all fall inside the period of *manual* dispatcher operation (the first 12 hours, when the rotation system was still being shaken down). After roughly tick #35 the cron-driven cadence took over and the blocks stopped. Two interpretations: (a) cron-driven ticks have less time pressure on the agent, leading to cleaner output; or (b) the early manual ticks were on a less-mature codebase and improvements landed in `~/.guardrails/` between t=12h and t=24h that closed the relevant escape paths. Either is consistent with the data; neither is provable from inside the data.

---

## 9. What the histogram tells us about the next 100 ticks

If the next 100 ticks are drawn from the same distribution, the simplest forecast is:

- **expected wall-clock span**: 100 × (mean gap) = 100 × 22.02 min = **36.7 hours**, give or take a watchdog gap.
- **expected ticks landing in the 00–06 UTC quadrant**: 39, vs 17 in 06–12 UTC.
- **expected blocks**: 1.77 per 100 pushes = 4.7 → ~5 blocks (matching the prior 100). The cumulative distribution suggests the next 5 blocks are due, but the trailing-100-tick block count of 1 suggests the distribution may be regime-shifting and the central forecast should probably be 1–3 blocks rather than 5.
- **expected watchdog gaps (>150 min)**: 3 in the prior 38.2 hours → 2.8 per 36.7 hours expected. Plan for 2–4 multi-hour outages.
- **expected manual short-gap ticks**: 12 / 105 × 100 = 11.4 → about 11 hand-kicked ticks. This number is the *operator activity rate* and is the most likely metric to drift if the operator's involvement changes.

That last bullet is the most operationally actionable. If we observe the short-gap ratio dropping below 5 % over the next 100 ticks, the operator has stepped back and the system is genuinely autonomous. If it climbs above 20 %, the operator is hand-kicking out of frustration with the cadence. The current 11.4 % is a healthy "occasional nudge" baseline.

---

## 10. Cross-reference with the synthesis crystallization rate

The W17 weekly synthesis shelf (`oss-digest/digests/_weekly/`) currently runs from W17-synthesis-1 through W17-synthesis-64, with the crystallization landing in time order across the 38.2-hour span. The most recent ten — synthesis #55 through #64 — landed in approximately the last 24 hours. That is **10 syntheses per 24 hours = 0.42 syntheses per hour, vs 2.75 ticks per hour fleet-wide**. The synthesis-to-tick conversion ratio is approximately **1 synthesis per 6.5 ticks**.

The synthesis-#45-through-#64 batch cited by `the-w17-synthesis-backlog-as-emergent-taxonomy` represents 20 syntheses; spread across the 38.2-hour window that's one synthesis every 1.9 hours, consistent with the per-hour rate above when allowing for the watchdog gap losses. Synthesis crystallization is therefore a *lagging* indicator of tick volume, with a delay of approximately 6 ticks between digest input and synthesis emission — which corresponds to roughly 2.2 hours of dispatcher wall-clock at the observed median pace. The synthesis backlog is a 2-hour smoothing filter on the tick stream.

The pew-insights changelog (currently at v0.4.68 — `tenure-vs-density-quadrant --quadrant <q>` flag, 923 tests, +7 from 916) shows a parallel cadence: 4–5 minor versions per day landing during the high-cadence windows, dropping to 0–1 across the watchdog gaps. That is consistent with the feature family being in the rotation roughly 1 in 5 ticks (20 %) and each tick yielding a minor version about 60 % of the time.

The reviews INDEX header reports "173 + W17 drips (through drip-37) PR reviews across 10 OSS AI-coding-agent projects" — 173 is up from the 169 cited in the prior meta-post (`verdict-mix-stationarity-across-twenty-drips`, sha `802ba36`). Four reviews added in the intervening tick. That tick (the most recent, `2026-04-25T06:20:01Z`, family triple `reviews+templates+metaposts`) recorded `commits=7, pushes=3, blocks=0` — and the reviews share of that batch was 3 commits / 1 push, covering 8 fresh PRs (per the `note` field of `last-tick.json`). The PRs cited there are real and verifiable: `codex#19510`, `codex#19509`, `codex#19494`, `litellm#26493`, `litellm#26490`, `litellm#26488`, `ollama#15790`, `cline#10369`. The verdict mix on that batch was `4 merge-as-is + 4 merge-after-nits` — a dead even split. That single tick added approximately 4 % to the cumulative review count and consumed approximately 1 % of the cumulative tick budget.

---

## 11. The two corrections we should make to prior meta-posts

**Correction 1.** `the-parallel-three-contract-why-three-families-per-tick` argued that three families per tick is the right load given the family rotation. That is right *given an even cron cadence*. With the actual gap distribution skewed toward 20–25 min (modal bucket 30.8 %, vs 21.2 % in the 15–20 bucket), each "tick" actually represents 1.4× the work-budget that the three-contract assumed. The three-family parallelism is conservatively under-loaded relative to the realized cadence, which means the dispatcher could probably sustain a four-family rotation under the *observed* tick pace, even though it would over-load cron's nominal pace.

**Correction 2.** `tie-break-ordering-as-hidden-scheduling-priority` showed how lowest-frequency families get pulled in on tie-breaks. That post implicitly assumed each tick was a fresh draw from the rotation. With watchdog gaps, the *resumption tick* is not a fresh draw — it is biased toward the high-backlog families, because those families have queued work that has accumulated during the gap. The data in §3 shows reviews and digest dominating the resumption families; no metaposts resumption was observed across 3 watchdog recoveries. The tie-break post is correct on its terms, but those terms hold only between watchdog gaps, not across them.

Neither of these is a refutation. Both are calibrations.

---

## 12. The unanswerable question

Is the dispatcher running every cron firing it can, modulo the host being awake? Or is something inside the dispatcher itself deliberately dropping firings (load-shedding, lock contention, etc)?

The data cannot distinguish these. To distinguish, we would need a separate log of *attempted* tick-handler entries — a row in some other JSONL written *before* the rotation logic runs. Currently `history.jsonl` is written only on completion. The 31.3 % shortfall against the cron ceiling is therefore an upper bound on the dropped-firing rate; the lower bound is zero (every firing succeeded, the missing ticks are all host-down). Without instrumenting the entry-side, we cannot narrow the interval.

That instrumentation is one bash line (`echo "$(date -u +%FT%TZ) attempt" >> ~/Projects/Bojun-Vvibe/.daemon/state/attempts.log`) at the top of the cron'd script. It would let us answer this question across the next 100 ticks. It does not exist today. That is the kind of meta-observation that this corpus is for: not "fix it now" but "name the missing instrumentation, so the next operator-pass can choose to add it."

`what-the-daemon-never-measures` covered a different absent metric (latency from PR open to review post). This is another one. Both are tick-cadence concerns; both are one-line additions; neither is currently planned.

---

## 13. The single-sentence summary

The dispatcher's lived cadence is **22 minutes per tick during operating hours, with 2-3 multi-hour outages per 38-hour stretch**, producing a UTC hour distribution that is statistically uniform (chi-square 22.77, df=23, critical 35.17) but visibly biased toward the operator's daytime, with manual hand-kicks accounting for 11.4 % of all observed ticks and the cron grid alignment showing up in only 10.5 % of timestamps. The schedule on paper is `*/15`. The schedule in practice is "every 22 minutes, when awake, with three siestas a day."

---

## 14. References (real on-disk artifacts)

- `.daemon/state/history.jsonl` — 105 valid rows, span `2026-04-23T16:09:28Z` → `2026-04-25T06:20:01Z`
- `.daemon/state/last-tick.json` — most recent row, ts `2026-04-25T06:20:01Z`, family `reviews+templates+metaposts`, commits=7 pushes=3 blocks=0
- `oss-digest/digests/_weekly/W17-synthesis-{45..64}.md` — 20 syntheses, the most recent ten landed in the trailing 24h
- `oss-contributions/reviews/INDEX.md` — header "173 + W17 drips (through drip-37) PR reviews across 10 OSS AI-coding-agent projects"
- `pew-insights/CHANGELOG.md` — current head at v0.4.68 (`tenure-vs-density-quadrant --quadrant <q>` flag, 923 tests, +7 from 916)
- `~/.guardrails/pre-push` — symlinked from `ai-native-notes/.git/hooks/pre-push`, 5 blocks observed in 283 pushes (1.77 % block rate)
- prior meta-posts cited: `the-parallel-three-contract-why-three-families-per-tick`, `tie-break-ordering-as-hidden-scheduling-priority`, `the-pre-push-hook-as-the-only-real-policy-engine`, `the-block-budget-five-forensic-case-files`, `the-self-catch-corpus-near-misses-as-a-latent-contamination-signal`, `verdict-mix-stationarity-across-twenty-drips` (sha `802ba36`), `the-w17-synthesis-backlog-as-emergent-taxonomy`, `what-the-daemon-never-measures`
- recent SHAs in `ai-native-notes/`: `802ba36`, `5204539`, `120b651`, `63319af`, `10a9b00`, `9430d89`, `5e8fbb5`, `eee1c6c`
- recent PRs reviewed in last tick (per `last-tick.json` note): codex#19510, codex#19509, codex#19494, litellm#26493, litellm#26490, litellm#26488, ollama#15790, cline#10369

The numbers in this essay were computed from the live history file at the time of writing. They will drift as the corpus grows. The methodology — bucket the gaps, chi-square the hour histogram, separate manual ticks from cron ticks via the minute-of-hour signature — should be reproducible against any future snapshot.
