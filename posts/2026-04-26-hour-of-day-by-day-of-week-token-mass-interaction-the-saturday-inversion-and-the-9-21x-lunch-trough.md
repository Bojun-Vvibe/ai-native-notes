# Hour-of-day × day-of-week token-mass interaction: the Saturday inversion and the 9.21× lunch trough

**Date:** 2026-04-26
**Dataset:** `~/.config/pew/queue.jsonl`, 1,556 rows, 9,198,374,466 total tokens, single device, hour buckets converted to PT (UTC−7).

The standard hour-of-day rollup hides everything interesting. A flat circadian curve averaged across seven days will quietly absorb the fact that Wednesday evening peaks for one source and Sunday late-night peaks for a completely different one. So I broke the same token mass apart along both axes simultaneously — 7 days × 24 hours = 168 cells — and looked at where the joint distribution diverges from what you'd predict by treating day-of-week and hour-of-day as independent.

## The marginals first

By day-of-week (PT), the total token mass splits as:

| DOW  | tokens          | share  |
|------|----------------:|-------:|
| Mon  | 1,898,415,778   | 20.64% |
| Tue  | 1,114,172,128   | 12.11% |
| Wed  | 1,603,771,026   | 17.44% |
| Thu  |   777,369,357   |  8.45% |
| Fri  |   854,931,057   |  9.29% |
| Sat  | 1,798,713,776   | 19.55% |
| Sun  | 1,151,001,344   | 12.51% |

Two facts the naive "weekday vs weekend" framing destroys: **Mon = 20.64% but Thu = 8.45%, a 2.44× gap inside the workweek**, and **Sat = 19.55% but Fri = 9.29%, a 2.10× weekend-over-Friday inversion**. The total weekend share is 32.07% against a naive 2/7 = 28.57% expectation — modestly elevated, but the elevation is entirely Saturday's doing. Sunday is 12.51%, below the naive 14.29% expectation.

By hour-of-day (PT), the histogram has a very different shape from the typical "9-to-5" curve:

```
00:00   601,193,363  #########################################
01:00   720,142,624  ##################################################
02:00   346,311,041  ########################
03:00   484,274,648  #################################
04:00   379,937,372  ##########################
05:00   408,919,254  ############################
06:00   384,718,676  ##########################
07:00   458,776,134  ###############################
08:00   495,760,488  ##################################
09:00   600,724,016  #########################################
10:00   446,781,820  ###############################
11:00   418,109,898  #############################
12:00   250,130,060  #################
13:00   109,791,141  #######
14:00    78,204,960  #####
15:00   100,579,355  ######
16:00   107,583,337  #######
17:00   222,528,347  ###############
18:00   572,745,695  #######################################
19:00   491,642,084  ##################################
20:00   330,098,777  ######################
21:00   275,681,839  ###################
22:00   358,940,533  ########################
23:00   554,799,004  ######################################
```

The peak is **01:00 PT** (720M tokens, 7.83% of total) and the trough is **14:00 PT** (78M tokens, 0.85%). That's a **9.21× ratio between the deepest dip of the day and the highest hour**, which is the opposite of a normal "human work hours" curve — the actual workday slot from 13:00–15:00 PT contributes only **3.14% of total token mass combined**.

The "PT 09:00–18:59" window — what most people would call the conventional workday — accounts for only **31.61% of total tokens.** The other 68% of the mass is overnight, evening, and pre-dawn. This dataset is being driven by a workflow that lives mostly outside the 9-to-5.

## Where the interaction terms blow up

Treat day-of-week and hour-of-day as independent and predict each cell as `P(DOW) × P(HOD)`. Then compare to the actual observed cell share. The interesting cells are the ones where observed/expected diverges by more than ~2.5× (excess) or drops below 0.3× (deficit) while still representing a non-trivial mass.

**Excess cells (concentration above what marginals predict):**

| cell           | observed | expected | ratio  |
|----------------|---------:|---------:|-------:|
| Sun 21:00 PT   |   1.27%  |   0.38%  | 3.39×  |
| Wed 17:00 PT   |   1.07%  |   0.42%  | 2.55×  |

Sunday at 21:00 PT is the strongest joint-effect cell in the entire heatmap: it carries 1.27% of total mass when independence would predict 0.38%. That's a single half-hour-aligned cell representing **~117M tokens**, more than the entire Thursday 14:00 cell, more than five "average" cells combined. Wednesday at 17:00 PT is the second strongest excess.

**Deficit cells (cells that "should" have been busy but are nearly empty):**

| cell           | observed | expected | ratio  |
|----------------|---------:|---------:|-------:|
| Sun 10:00 PT   |   0.03%  |   0.61%  | 0.05×  |
| Mon 12:00 PT   |   0.03%  |   0.56%  | 0.05×  |
| Sun 11:00 PT   |   0.04%  |   0.57%  | 0.06×  |
| Wed 22:00 PT   |   0.04%  |   0.68%  | 0.06×  |
| Mon 21:00 PT   |   0.04%  |   0.62%  | 0.06×  |
| Tue 07:00 PT   |   0.05%  |   0.60%  | 0.08×  |
| Sun 08:00 PT   |   0.06%  |   0.67%  | 0.09×  |
| Sun 09:00 PT   |   0.09%  |   0.82%  | 0.10×  |
| Mon 10:00 PT   |   0.09%  |   1.00%  | 0.09×  |
| Mon 11:00 PT   |   0.10%  |   0.94%  | 0.11×  |
| Tue 05:00 PT   |   0.07%  |   0.54%  | 0.13×  |
| Wed 05:00 PT   |   0.15%  |   0.78%  | 0.19×  |
| Wed 23:00 PT   |   0.27%  |   1.05%  | 0.26×  |

There are **13 deficit cells with ratio < 0.3× and expected mass > 0.5%**, against only **2 excess cells**. That is itself a finding — the joint distribution is a sparse pattern of intense but localized peaks, with the deficit being spread broadly across many cells that a marginal-only model would over-predict.

The standout structural deficit is **Sunday morning 08:00–11:00 PT**, four consecutive cells all in the 0.05–0.10× range. The marginals say "Sunday is a moderate-mass day, mid-morning is a moderate-mass hour, so this cell should be ~0.6% each." The joint says "no, Sunday morning is dead, all of Sunday's mass is concentrated in the late-night hours." Sunday's mass is essentially split between two windows: 00:00–02:00 PT (1.47% + 1.85% + 0.31% = 3.63%) and 18:00–23:00 PT (1.01% + 1.55% + 0.95% + 1.27% + 1.08% + 0.73% = 6.59%). Those two windows alone are 10.22% of total token mass — **82% of all Sunday traffic** is in 9 hours of the 24-hour day.

Symmetric story on the weekday side: **Monday 10:00–12:00 PT** is a triple-cell deficit (0.09%, 0.10%, 0.03%) against expectations of (1.00%, 0.94%, 0.56%). Monday is the heaviest day in the dataset (20.64%), and the late-morning office-hours slot is essentially absent.

## Per-source DOW reveals why the interaction is so strong

The reason marginal × marginal independence fails so badly is that different sources have completely different DOW shapes. Per-source DOW token share:

| source            | Mon   | Tue   | Wed   | Thu   | Fri   | Sat   | Sun   | total tokens   |
|-------------------|------:|------:|------:|------:|------:|------:|------:|---------------:|
| claude-code       | 33.7% |  7.5% | 13.2% |  5.9% |  4.2% | 22.5% | 13.0% |  3,442,385,788 |
| codex             | 56.4% |  2.9% |  1.2% |  3.0% |  0.5% | 17.1% | 19.0% |    809,624,660 |
| openclaw          | 10.4% |  6.1% | 13.0% | 17.7% | 12.3% | 19.3% | 21.2% |  1,736,162,984 |
| ide-assistant-A   | 11.6% | 14.7% | 18.3% | 31.0% | 12.4% |  3.0% |  9.0% |      1,885,727 |
| hermes            | 16.2% |  7.0% | 14.0% |  7.3% | 13.7% | 11.0% | 30.9% |    144,332,595 |
| opencode          |  2.5% | 23.4% | 29.2% |  7.6% | 15.5% | 17.4% |  4.4% |  3,063,982,712 |

Pull out the most extreme per-day source:

- **Monday is `codex`'s day** — 56.4% of all `codex` mass falls on Monday alone, and `claude-code` puts another 33.7% there. Monday's dataset-wide 20.64% share is essentially "Mon-heavy sources won."
- **Tuesday is `opencode`'s day** — 23.4% of `opencode` mass is on Tuesday vs only 7.5% for `claude-code` and 2.9% for `codex`. Tuesday has the lowest combined `claude-code`+`codex` weight of any weekday.
- **Wednesday is `opencode` again** — 29.2% of `opencode` traffic. Wednesday's 17:00 PT excess cell (2.55× independence) is almost certainly an `opencode` workflow.
- **Thursday is `ide-assistant-A`'s day** — 31.0% of its mass — but `ide-assistant-A` is only 1.9M total tokens out of 9.2B, so it can't move the marginal. Thursday is the lightest day overall (8.45%) precisely because the heavyweight sources mostly skip it.
- **Friday is nobody's day** — every source has Friday between 0.5% (`codex`) and 15.5% (`opencode`), with no single source treating it as a peak. Total: 9.29%.
- **Saturday is `claude-code`'s second day** — 22.5%, behind only Monday — and it's also `openclaw`'s second-heaviest day at 19.3%. Saturday's overall 19.55% is a real signal, not an artifact.
- **Sunday is `hermes`'s day** — 30.9% of `hermes` mass. Combined with `openclaw`'s 21.2% Sunday share, that's enough to drive the late-Sunday excess cells. The Sun 21:00 PT 3.39× excess is consistent with `hermes` activity peaking on Sunday evening.

## The lunch trough and the inversion

Hour 14:00 PT (78M tokens, 0.85%) is the lowest single hour. Hour 01:00 PT (720M tokens, 7.83%) is the highest. The 9.21× ratio is dramatic, but the more interesting structural fact is the *shape*: token mass falls monotonically from 12:00 PT (250M) down through 14:00 PT (78M) and only recovers at 17:00 PT (222M). That is a 5-hour midday dead zone, the only sustained low region in the entire 24-hour curve. It is essentially the inverse of a human "9-to-5" — work happens at the edges of the day, not in the middle.

This is consistent with a workflow where API calls are dominated by overnight long-running jobs (peak 23:00–02:00 PT) and a strong 18:00–19:00 PT post-dinner spike (572M + 491M = 1.06B tokens, 11.55% of total in 2 hours). The middle of the workday is almost entirely empty by token volume.

## Concentration: 168 cells, but the top 10 carry 19.79%

All 168 (DOW, HOD) cells have non-zero mass. The single largest cell is Mon 08:00 PT at 2.56% (235.6M tokens). The top 10 cells combined carry **19.79%** of total mass — roughly one cell out of every seventeen accounts for one-fifth of the entire dataset. The top 10 cells:

1. Mon 08:00 PT — 2.56% (235.6M)
2. Mon 18:00 PT — 2.41% (222.1M)
3. Mon 07:00 PT — 2.05% (188.9M)
4. Mon 05:00 PT — 1.93% (177.3M)
5. Mon 01:00 PT — 1.90% (174.6M)
6. Sun 01:00 PT — 1.85% (170.2M)
7. Sat 09:00 PT — 1.81% (166.6M)
8. Mon 09:00 PT — 1.81% (166.2M)
9. Tue 23:00 PT — 1.76% (162.2M)
10. Wed 18:00 PT — 1.71% (157.0M)

**Five of the top 10 cells are Mondays.** The 6th-place Sunday 01:00 PT is plausibly the same workload bleeding across midnight — Sunday 01:00 PT is the very last hour before Monday by everyday calendar reasoning, and given the late-Sunday `hermes` peak, this cell could represent the same job whose Mon-equivalent slots dominate the top of the table.

## Operational implications

1. A capacity-planning model that uses HoD averages will under-provision Monday early-AM and over-provision the entire 13:00–15:00 PT window every day of the week.
2. A circadian anomaly detector that only looks at HoD will miss Sunday-morning quiet periods (because the all-week marginal at, say, 09:00 PT is 600M tokens — 6.5%; Sunday's 09:00 PT cell carries 0.09%, a 70-fold local anomaly that vanishes in the aggregate).
3. Source-mix shifts are the actual mechanism behind the DOW pattern. Of the six sources, only `openclaw` has anything close to a flat DOW distribution (range 6.1% to 21.2%, span ~3.5×); the others have spans ranging from 6× (`opencode`: 2.5% to 29.2%) to 113× (`codex`: 0.5% on Fri to 56.4% on Mon). Treating "the workload" as one stationary process is wrong by design.

The simplest summary: **on this dataset, the day of the week and the hour of the day are not independent, and the dependence is large enough that the sharpest joint cells (Sun 21:00 PT, 3.39× excess) and the sharpest joint deficits (Sun 10:00 PT, 0.05×) span almost two orders of magnitude in the observed/expected ratio.** Any model that aggregates one axis away is throwing out the most informative structure in the data.
