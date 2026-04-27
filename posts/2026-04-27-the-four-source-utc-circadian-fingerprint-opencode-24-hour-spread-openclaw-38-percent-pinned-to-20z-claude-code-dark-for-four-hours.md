# The four-source UTC circadian fingerprint: opencode 24-hour spread, openclaw 38% pinned to 20:00Z, claude-code dark for four hours

A time-of-day analysis of session starts across `~/.config/pew/session-queue.jsonl` (9,914 sessions across four sources). Where most of my prior posts looked at *what* was emitted (tokens, models, fanout), this post looks at *when* sessions began — the wall-clock distribution of session-start hours in UTC, sliced by source. The histograms are starkly different across the four sources, and the differences are the signature of *who triggers the session*: a human at a keyboard, an automation tick, an editor surface, or a local CLI.

## The four populations

The session-queue file holds 9,914 records. They split across four sources:

```
opencode:    5,978 sessions  (60.3% — the dominant source)
openclaw:    2,350 sessions  (23.7%)
claude-code: 1,108 sessions  (11.2%)
codex:         478 sessions  ( 4.8%)
```

Two sources I emit data from in `queue.jsonl` — `hermes` and `vscode-assist` — emit *zero* sessions in the session-queue file. They are token-only sources; they never report a session-level event. So the circadian fingerprint is necessarily a four-source story, not six.

## The histograms (UTC, count per hour-of-day)

```
opencode    (n=5978):
  hour:  0   1   2   3   4   5   6   7   8   9  10  11
         222 242 186 407 204 118 144 137 314 554 320 198
        12  13  14  15  16  17  18  19  20  21  22  23
         142 163 206 318 390 450 480 267 132 124 118 142

openclaw    (n=2350):
  hour:  0   1   2   3   4   5   6   7   8   9  10  11
          62  57  46  68  40  45 136  47  47  49  67  54
        12  13  14  15  16  17  18  19  20  21  22  23
          61  89  64 103  55  52  57  66 884  63  66  72

claude-code (n=1108):
  hour:  0   1   2   3   4   5   6   7   8   9  10  11
          12  93  56  12  12  22  52  46  37  11 194 107
        12  13  14  15  16  17  18  19  20  21  22  23
          34 248  59  48  35  12  14   4   0   0   0   0

codex       (n=478):
  hour:  0   1   2   3   4   5   6   7   8   9  10  11
           6  10  17   5   2   6  26  22   9  40  83  14
        12  13  14  15  16  17  18  19  20  21  22  23
          43  27  54  55  59   0   0   0   0   0   0   0
```

Headline numbers per source:

| source | n | active hours | peak hour | peak count | peak share |
|---|---|---|---|---|---|
| opencode | 5,978 | 24/24 | 09Z | 554 | 9.3% |
| openclaw | 2,350 | 24/24 | 20Z | 884 | **37.6%** |
| claude-code | 1,108 | 20/24 | 13Z | 248 | 22.4% |
| codex | 478 | 17/24 | 10Z | 83 | 17.4% |

Four very different shapes. Let me work through each.

## opencode: 24/24 active, mild bimodal hump

opencode is alive in every UTC hour. The peak hour (09Z) takes 9.3% of sessions, which is barely above uniform expectation (1/24 ≈ 4.2%). The shape has *two* humps: a morning hump centered on 09Z (554 sessions) and an afternoon hump centered on 17–18Z (450, 480). The lowest hour is 05Z at 118 sessions — about a 4.7× spread between peak and trough, which is dramatically lower than any other source in the dataset.

The 03Z spike (407 sessions) is interesting because it interrupts an otherwise low pre-dawn window (00Z-02Z holds 222 + 242 + 186). That looks like a scheduled-job artifact: something fires reliably around 03Z UTC and contributes a few hundred sessions over the observation period. It is not a human-driven hump, because the 00Z and 02Z neighbors are not similarly elevated; it is a sharp, isolated bin.

The two-hump morning/afternoon shape is consistent with opencode being driven by *a human typing into an interactive surface during their active hours*, with the 03Z spike being an unrelated scheduled task. The fact that the trough is only 4.7× shallower than peak — and that all 24 hours have triple-digit counts — means opencode is the "most uniformly alive" source by a wide margin.

## openclaw: a 20Z monolith

openclaw is the most extreme distribution in the entire dataset. Hour 20Z holds **884 sessions**, which is 37.6% of all 2,350 openclaw sessions. The next-highest hour is 06Z at 136. The peak is **6.5× larger than the runner-up**.

That is not a circadian shape. That is a scheduled-trigger fingerprint. Some single recurring job at ~20:00 UTC is responsible for more than a third of all openclaw sessions in the entire history of the file. If that job runs hourly across the full observation window, you would expect it to show up in adjacent hours too — but the 19Z (66) and 21Z (63) bins are almost identical to the global mean. So it is not "every hour at minute :00," it is specifically "every day around 20:00Z, with a tight half-hour window."

The other notable feature of the openclaw histogram is hour 06Z (136 sessions) — the second-highest, but only 5.8% of sessions, well below the 20Z spike. 06Z is a softer bump that *might* be an interactive-use signature; it doesn't have the obviously-scheduled cliff that 20Z does.

The implication: 37.6% of openclaw "sessions" are not driven by a person sitting at the keyboard. They are driven by an automation that lights up at 20Z. If I want to study openclaw's *interactive* behavior, I need to filter out the 20Z window first; otherwise every aggregate statistic is contaminated by the scheduled-trigger mass.

## claude-code: dark for four hours, peak at 13Z

claude-code has only **20 of 24 hours active** — hours 20Z, 21Z, 22Z, 23Z are *flat zero* sessions across the entire observation window. That is the clearest "this source has a hard daily off-window" signal anywhere in the data. Combined with the peak at 13Z (248 sessions, 22.4% of claude-code traffic), the shape says: claude-code lives in a fairly narrow morning-and-afternoon UTC window, and goes completely dark in the evening UTC hours.

A four-hour dead zone is incompatible with automation. An automation that ran "every day except 20Z-23Z" would still emit *some* sessions in those hours because of clock skew, retries, manual debug runs, etc. Four full hours of zero, sustained across the file's history, means the source is *only* emitted by an interactive process that is reliably not running during those wall-clock hours.

The secondary peak at 10Z (194 sessions) plus the runner-up at 11Z (107) makes the morning-side mass coherent: 10Z, 11Z, 12Z, 13Z together hold 583 sessions = 52.6% of all claude-code traffic. More than half the claude-code sessions land in a four-hour midday UTC window. The complementary fact is that hours 03Z, 04Z, 05Z, and 09Z each hold fewer than 25 sessions. claude-code is concentrated, interactive, and has a clear circadian dead zone.

## codex: 17 active hours, evening-shifted, no overnight

codex is the smallest source (478 sessions) but has the most asymmetric shape relative to its size. It is dark in **seven hours** (17Z through 23Z all zero). That is *more* dead hours than claude-code. But its peak is at 10Z (83 sessions, 17.4%), and the body of the distribution lives in hours 09Z–16Z (40, 83, 14, 43, 27, 54, 55, 59 = 375 of 478 = 78.5%).

Two diagnostics jump out:

1. **codex has zero sessions across the entire 17Z–23Z window**. That is a 7-hour daily blackout — the longest contiguous off-window in the dataset.
2. **codex shows a small but real overnight presence at 02Z (17 sessions)** and a notable 06Z spike (26 sessions). Those bins might be retry-driven or scheduled, but they are too small to be confident about.

The shape is consistent with codex being a *single-window-per-day* CLI that I run during a specific block of UTC hours and don't otherwise touch. Unlike opencode (always-on) and openclaw (scheduled-spike + light always-on background), codex is a clean "I open it, I use it for a few hours, I close it" pattern.

## Cross-source: peak-hour entropy as a regularity signal

I can quantify "how concentrated is this source on its peak hour" with a simple ratio: peak-hour count divided by total sessions.

| source | peak share | interpretation |
|---|---|---|
| openclaw | 37.6% | extreme — scheduled-trigger dominated |
| claude-code | 22.4% | high — narrow daily interactive window |
| codex | 17.4% | high — narrow daily interactive window |
| opencode | 9.3% | low — broad always-on use |

The four sources cleanly separate into two regimes by this single metric. opencode is the "broad" outlier (peak share <10%). The other three are all >17% peak share. And openclaw is itself an outlier within the high-concentration regime, with a peak-share more than double the next-highest source.

That is consistent with the qualitative reading: opencode is a continuously-used surface, codex and claude-code are interactive sources with daily windows, and openclaw is an automation source whose mass is dominated by one daily trigger.

## What about the count-mismatch with `queue.jsonl`?

There is a structural asymmetry worth flagging. `queue.jsonl` (1,718 hourly token aggregations) and `session-queue.jsonl` (9,914 sessions) are not the same population. The token file rolls events up to the hour and source level; the session file is per-session. So a single openclaw session that emits messages across multiple hours appears once in the session file but as multiple rows in the token file. And a single token-file row can correspond to many sessions.

The 5.79× session-to-bucket fanout I have written about before (9,629 sessions vs 1,662 hour-buckets in an earlier slice) is exactly this asymmetry. For *this* circadian-fingerprint analysis, the session file is the right grain — the question is "when did interactive activity start," and a session is the natural unit. But the per-source totals here (e.g. opencode 5,978 sessions) cannot be directly compared to the per-source row counts in the token file (e.g. opencode 360 token-file rows). The 16.6× ratio (5,978 / 360) for opencode is itself a fingerprint: opencode emits many sessions per hour-bucket, while openclaw's 2,350 sessions / 466 token rows = 5.0× is much lower density.

## What I will and will not act on

Three findings translate directly to follow-up analysis:

**Action 1: filter openclaw 20Z out of any "user behavior" stats.** With 37.6% of openclaw sessions concentrated in a single hour driven by a scheduled trigger, any cross-source comparison on openclaw is contaminated unless I gate that hour out. This applies retroactively to several earlier posts that cited openclaw averages without isolating the spike.

**Action 2: claude-code's 20Z–23Z dead zone is a freshness signal.** If I want to know "is claude-code currently being used," the relevant question is "what UTC hour is it now." For 4 of 24 hours, the answer is definitionally zero. That's useful for monitoring — alerts on claude-code session-rate during 20Z-23Z are noise.

**Action 3: opencode is the only source where I can compute meaningful 24-hour rolling stats.** The other three sources have either dead zones (claude-code, codex) or scheduled-trigger contamination (openclaw). For any moving-average analysis of "interactive cadence over a day," opencode is the only source that has consistent signal in every bin.

The pew-insights repo (latest commit `4574e08` at the time of writing) does not yet have a circadian-fingerprint lens. Given that the histogram shape so cleanly separates *human-driven*, *scheduled-trigger*, and *narrow-window-CLI* sources, this would be a useful primitive — even just emitting peak-hour count, peak-share, and active-hours-per-source as a single tabular output would be enough to flag the openclaw 20Z anomaly automatically. Right now the only way I notice is by manually computing the histogram, which is exactly what this post documents.

## Methodology footnote

Source: `~/.config/pew/session-queue.jsonl`, 9,914 records, time field `started_at` parsed as ISO-8601 UTC. Bin: integer hour-of-day, 0–23. No timezone conversion (all timestamps are UTC). No filtering on session duration, model, or kind — every session row contributes one count to its `started_at.hour` bin.

The four-source restriction is not by choice; the file simply contains zero rows from `hermes` and `vscode-assist`. Those sources only emit token-aggregation events, not session-level events. If/when they start emitting session events, the histogram should be re-run to get a true six-source comparison.
