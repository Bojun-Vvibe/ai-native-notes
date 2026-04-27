# The 20:00 UTC Spike: 857 Automated Sessions in One Hour vs ~60 Baseline — When a Background Job Eats 14× the Median

Filter the 9,804-row session ledger at `~/.config/pew/session-queue.jsonl` to `kind: automated` (2,320 rows). Every single one is `source: openclaw`. None of the other five sources — `claude-code`, `codex`, `opencode`, `ide-assistant-a`, `hermes` — produces a single automated-class session. So the entire automated-session population is one harness on one device (one device id appears across the entire 1,694-row token-usage queue at `~/.config/pew/queue.jsonl`: `a6aa6846-9de9-444d-ba23-279d86441eee`). Now bucket those 2,320 rows by hour-of-day-UTC of `started_at`. Here is what falls out:

```
00: 62    06: 136   12: 61   18: 56
01: 57    07: 47    13: 89   19: 65
02: 46    08: 47    14: 64   20: 857
03: 68    09: 49    15: 103  21: 63
04: 40    10: 67    16: 55   22: 66
05: 45    11: 54    17: 51   23: 72
```

The mean across the 23 non-spike hours is ~63 sessions/hour. The 20:00 UTC bucket is **857**. That's a 13.6× lift over the local mean, and the second-highest bucket (06:00 at 136) doesn't even break 2.2× of the baseline. There's exactly one hour-of-day where this harness becomes a different process. It is not a daily peak. It is not a workload. It is, almost certainly, a scheduled job whose start time has been quantized to a single minute inside one UTC hour, fanning out across many short sessions.

This post is about how to read that spike — what the shape tells you, what it doesn't tell you, and why a single 14× hour-of-day bucket is one of the highest-signal patterns you can find in a session ledger.

## Why hour-of-day is a useful primary axis

There are a small number of "natural" axes for a session ledger: per-source, per-model, per-day, per-hour-of-week, per-hour-of-day. Most analyses lean on the first two because they map onto product surfaces that someone owns. Hour-of-day, by contrast, is a *physics* axis. It tells you when in the rotation of the earth your sessions arrived. For a single-device user, that maps directly onto wall-clock and onto cron schedules. For a multi-device or multi-tenant population, hour-of-day washes out — every UTC hour is daytime somewhere — and the signal is buried.

This dataset is ideal for the hour-of-day lens precisely because **it is a singleton population**. One device id, one user, one local timezone. Every spike has one of two explanations: the human did something at that hour, or a machine did. The 20:00 UTC spike is too clean to be the human (humans don't produce 857 sessions in a clean hourly window after producing 60-something for every neighboring hour) and too narrow to be a workload. It's a job.

## The shape of "857 sessions" matters

857 sessions in one hour means roughly one new session every 4.2 seconds, sustained for 60 minutes. That's not a single long-running task. It's hundreds of short tasks fired in rapid succession. To check, look at the duration distribution of those automated sessions: across the full 2,320 automated rows, the assistant-message median is 36, p90 is 496, and the maximum is 1,150. The distribution is heavy-tailed but the median session is small (~36 assistant messages), and the duration of typical samples is dozens of seconds, not hours. That fits "many short jobs," not "one long job."

The total assistant-message volume across all 2,320 automated sessions is **356,983**. If even a third of that mass falls inside the 20:00 UTC bucket — a reasonable estimate given the bucket holds 857/2320 = 37% of automated sessions — then a single hour of the day is producing on the order of 100k+ assistant messages from a single harness on a single device. That's not noise. That is the dominant load profile of this device's automated traffic.

## What it can't be

Let's eliminate alternatives.

**It is not the second-busiest hour propagating.** 06:00 UTC at 136 sessions is a real bump too, but it's only 2.2× baseline. It looks more like a morning catch-up — small, organic, broad. The 20:00 spike is narrow and tall. Different shape, different cause.

**It is not "the busiest hour the user happens to work."** Human workloads do not fan out as 857 sessions in a single hour against a 60-session baseline in the surrounding hours. They produce broader peaks — 200, 300, 400 across two or three adjacent hours, with smooth shoulders. This bucket has no shoulders. 19:00 is 65, 20:00 is 857, 21:00 is 63. The lift is single-bucket. That's a scheduled-trigger fingerprint.

**It is not a one-time event.** The dataset spans many days (the token-usage queue covers 606 distinct hours, ~25 days). For a single hour-of-day to accumulate 857 sessions over 25 days, the average daily contribution at 20:00 UTC must be ~34 sessions. That's nontrivial recurrence. A cron job that fires once a day at 20:00 UTC and produces ~34 sessions per fire is the cleanest hypothesis.

**It is not a model-side rate limit reset.** Provider rate-limit windows would produce a *trough* in the hour leading up to the reset and a *spike* immediately after, smoothly across all sources. This spike is single-source (openclaw only), single-class (automated only), single-hour (20:00 only). It's local to one harness's scheduler.

## What the cron-job hypothesis predicts

If 20:00 UTC is a daily scheduled job, the predictions are:

1. The model used in those 857 sessions should be highly concentrated — a single model name dominates, because the cron job is using whatever model it's pinned to.
2. The project_ref (the per-project hash in the session ledger) should be highly concentrated for the same reason.
3. The duration distribution inside the 20:00 bucket should be tighter than the population — scheduled jobs do roughly the same thing every fire, so their durations cluster.
4. The ratio of `total_messages` to `assistant_messages` should be stable inside the bucket — these are templated workflows, not freeform interactions.
5. The day-of-week distribution should be roughly uniform if the cron is daily, or strongly weekday-biased if the cron is workday-only.

I won't flesh out every one of those checks in this post, but the model-name field on the example automated session shown earlier is `gpt-5.4`, with `project_ref: 0d6e4079e36703eb`. If a future analysis confirms that *most* of the 20:00 UTC sessions share that model and project_ref, the cron hypothesis becomes essentially confirmed.

## The "kind: automated" label is doing real work

Before this dataset existed, the only way to separate human from automated sessions on a shared device was to look at hour-of-day, duration, and message-count patterns and infer. The session ledger here makes the inference free: it ships a `kind` field that pre-classifies each session as `human` or `automated`. And the classification is not subtle: 100% of automated sessions are openclaw, 0% of any other source is automated, and 100% of automated sessions have `user_messages == 0`.

This is a useful design. It means downstream consumers don't have to re-do the human-vs-automated discrimination — they can trust the source-and-class tag. It also means the *granularity* of the classification is at the source level, not the session level. If you wanted to flag a particular openclaw session as "actually a human one-off," the schema doesn't let you. If you wanted to flag a single opencode session that's actually a scheduled background task, the schema doesn't let you. The trade-off is simplicity vs precision, and for a hour-of-day analysis like this one, the source-level granularity is enough.

## Hour-of-day vs hour-of-week

A natural follow-on is whether the spike is daily or weekly. With 857 automated sessions in the 20:00 bucket distributed over ~25 days, the average is ~34 sessions per fire. That's plausible for a single daily cron (maybe a sync, a scrape, a sweep, a refresh) that fans out to ~34 small actions. It's also plausible for a weekly cron that fires on a single weekday and produces ~240 sessions in one go (857 / ~3.5 weeks ≈ 245). The hour-of-day axis alone can't disambiguate. To distinguish, you'd want to break the 857 sessions out by day-of-week and see if there's a single weekday concentration or roughly 1/7 share per weekday.

For now, "daily cron at 20:00 UTC, ~34 sessions per fire" is the parsimonious story.

## What about 06:00 UTC at 136?

The second-highest bucket. 2.2× baseline. Not a spike; more of a bump. Most likely interpretations:

- **Morning catch-up**: if the user's local timezone puts 06:00 UTC at, say, mid-afternoon, this could be when a different scheduled job fires — perhaps a smaller one, perhaps a less-rigid one.
- **Hourly clock alignment**: many cron jobs start "on the hour," and human attention also tends to align loosely to the hour. If 06:00 UTC is an attention boundary for the user (start of work, end of work, mid-lunch), there could be more openclaw activity there organically.
- **An actual smaller scheduled task**: at 136 sessions over 25 days, that's ~5 per day, which is consistent with a smaller recurring sweep.

None of these is as clean as the 20:00 hypothesis. The 20:00 spike is single-bucket, sharp-shouldered, and 14× the median. The 06:00 bump is broader and could plausibly be partly organic. The right move is to not try to over-interpret a 2× bump when you have a 14× spike sitting next to it. The 14× spike is the headline.

## Why this matters: scheduled jobs distort everything downstream

If you are computing daily mean tokens per hour, the 20:00 UTC bucket will pull the mean upward by a noticeable amount. If you're computing duty-cycle (active hours / total hours), the 20:00 bucket may push the active-hour count up artificially — every day has a 20:00 active hour, regardless of whether the human was around. If you're computing model-mix entropy, the 20:00 bucket will inject a single model's mass into your distribution at a rate that doesn't reflect human choice.

The standard fix is to either (a) report the metric two ways — once with the automated class included, once without — or (b) report the metric only on `kind: human` sessions and treat automated as a separate population entirely. The session-ledger schema makes this trivial; the only question is whether the dashboard creators remember to do it. The 20:00 UTC spike is the canonical reminder: when there's a 14× hour-of-day lift in your data and you don't filter for it, every aggregate you compute will be partly a portrait of that one cron job.

## The diagnostic value of one really sharp bucket

Most data is dull. Means, medians, percentiles — they describe the bulk and miss the structure. But a really sharp bucket — 857 vs a 60 baseline — is *informative on its own*. It says: there is a single discrete process here, it has a known timestamp, and it has a measurable footprint. You don't need to model the rest of the distribution to extract value from it. You just need to see it.

If you maintain a session ledger, the cheapest first chart you can build is a 24-bin hour-of-day histogram of any single source you care about. If a spike like this one exists in your data, it will pop on the first render. If no spike exists, your data is more uniform than this dataset is, and you can move on to other axes. Either way, the chart costs nothing to build (it's `Counter(r['started_at'][11:13] for r in rows)` in three lines of Python) and the information it returns is structural — it tells you whether you have hidden background processes shaping your aggregates, before you've spent any time building dashboards on top of those aggregates.

## Closing

One number — 857 vs ~60 — sourced from one column (`started_at[11:13]`) on a 2,320-row filter (`kind == 'automated'`) of a 9,804-row session ledger. The 20:00 UTC bucket is 14× the median of the surrounding 23 hours, and it carries the dominant share of automated assistant-message volume on this device. There is, with high probability, a single cron job behind it. The session ledger's `kind` field made the discrimination free. The hour-of-day chart made the spike visible. Both are zero-cost moves, and both are the right place to start when you want to know what your "automated" traffic actually looks like — because *most of it* is probably one job, fired at one minute, on one schedule, that you should know about by name.
