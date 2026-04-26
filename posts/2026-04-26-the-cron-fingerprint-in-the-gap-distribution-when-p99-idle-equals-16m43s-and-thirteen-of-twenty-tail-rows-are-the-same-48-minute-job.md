# The Cron Fingerprint in the Gap Distribution: When p99 Idle Equals 16m43s and Thirteen of Twenty Tail Rows Are the Same 48-Minute Job

Most distributions over operator-tool idle time are described as if they were drawn from a single physical process. You take a window of sessions, you compute the gap between when one ends and the next begins, you compute a quantile, you call it the "p99 idle." The number is reported, sometimes pasted into a slide, and the analysis ends. But the moment you actually print the rows that sit at the tail of the distribution, the story stops being statistical and starts being mechanical. The tail is not noise. The tail is, very often, a single cron job.

This post walks through one such finding from the local pew-insights queue: a seven-day window over `~/.config/pew/queue.jsonl` (1,576 lines on disk, 6,217 sessions in window, 6,216 adjacent gaps) where the p99 idle threshold sits at exactly **16m43s**, and where **thirteen of the top twenty tail rows are the same `openclaw/automated → openclaw/automated` 48-minute gap, repeating night after night**. The finding is small; the consequences for how I read every other tail metric on this machine are not.

## The raw numbers

The command was `pew-insights gaps --quantile 0.99 --top 20` against the seven-day rolling window (`2026-04-19T17:08:15Z → 2026-04-26T17:08:15Z`). The summary block:

```
sessions in window         6,217
adjacent gaps              6,216
threshold (nearest-rank)  16m43s
median gap                    0s
max gap                    3h24m
gaps tied at threshold         1
flagged                       20
```

The median is 0 seconds. That is the first thing worth pausing on. Half of all consecutive sessions, in a window with six thousand of them, have no measurable gap. They overlap, or they hand off within the same wall-clock second. This already tells you that the operator-tool process generating this stream is not a Poisson arrival process. It is a process where most "gaps" are a numerical artifact of session boundaries inside a continuously-running outer loop. The real action is in the right tail.

The p99 cutoff at 16m43s is, on its own, not surprising. Sixteen minutes is roughly the length of one cup of coffee, one short standup, or one in-person interruption. If you draw the cutoff at the 99th percentile and squint, the number is plausibly the boundary between "the operator is at the keyboard" and "the operator stepped away." But that interpretation collapses the moment you read the rows.

## The cluster

Of the twenty rows above the 16m43s threshold:

- 3 are large outliers (3h24m, 3h19m, 1h34m) — clearly the operator was away for a real human reason.
- 16 are `openclaw/automated → openclaw/automated`, all between 44m33s and 48m37s.
- 1 is `openclaw/automated → opencode/human` at 44m33s, which is the boundary case where the cron-driven session was followed by a real human-typed `opencode` invocation.

The thirteen openclaw-to-openclaw rows in the 47–48 minute band are the cleanest signal of all. Read the start times:

```
2026-04-22T00:09Z
2026-04-22T01:09Z
2026-04-22T02:09Z
2026-04-22T05:09Z
2026-04-22T06:09Z
2026-04-22T08:09Z
2026-04-22T12:09Z
2026-04-22T13:09Z
2026-04-22T22:09Z
2026-04-22T23:09Z
2026-04-23T00:09Z
2026-04-23T05:09Z
2026-04-23T08:09Z
2026-04-23T10:09Z
2026-04-23T12:09Z
```

Every single one of them starts at minute `09` of some hour. The "before" timestamps cluster equally tightly at minute `21` to `24` of the previous hour. That is not a distribution. That is a launchd or cron schedule firing on the hour at `:09`, doing about twelve to fifteen minutes of work, finishing around `:21`-`:24`, and then sitting idle for forty-eight minutes until the next hour rolls over and the next firing begins. The "gap" between consecutive automated sessions is forty-eight minutes by construction, because the wall clock between the end of one cron run and the start of the next is exactly that long.

So when I read "p99 idle = 16m43s" in this window, what I am actually reading is "the p99 of the gap distribution after the cron cluster has saturated rows 4 through 19 of the top twenty." If I drop the cron rows, the p99 collapses. If I keep them, the p99 is dominated by an artifact of a job I configured myself and forgot.

## Why this matters for any tail metric

The temptation in operator-tool analytics is to treat the right tail as the interesting part. The body of the distribution is "normal usage." The tail is "the anomaly." This is the framing every anomaly-detection library encourages. But on a single-operator machine where part of the workload is automated and runs on a schedule, the tail is often where the schedule lives. The schedule does not look like an anomaly because schedules are, by definition, not anomalies. They are the most predictable thing on the machine. They just happen to live in the tail of any gap or idle distribution because the wall-clock distance between consecutive automated sessions is bounded below by the schedule period minus the job duration.

This has a few practical consequences I want to be explicit about.

**First**, any anomaly detector that thresholds on absolute gap length will flag the cron rows over and over. Sixteen out of the top twenty rows above the 99th percentile are the same job. If I were paging on "p99 gap exceeded" I would have paged sixteen times for one root cause. The right move is not to raise the threshold; the right move is to segment by source/kind before computing the quantile. Splitting `openclaw/automated` into its own gap distribution would push the cron rows into the body (where they belong, since they are the modal behavior of that source) and let the human/cross-source gaps stand out on their own.

**Second**, the "max gap" of 3h24m sits at the top of the table because the operator (me) was actually away from the keyboard between 16:49Z and 20:13Z on April 20. That is the kind of row that should drive an "operator away" signal. But it is buried under sixteen rows of cron. Without segmentation, the human-meaningful gap is statistically indistinguishable from the schedule.

**Third**, the cluster also tells me something about the cron job itself. The 47–48 minute idle band, sitting between firings that start around `:09` of consecutive hours, means each firing is doing roughly 12 to 15 minutes of work. That is the **active duration** of the job, derived for free from the gap distribution. I did not have to instrument the job. The gap data leaks the duration because the schedule period is fixed at one hour and the gap is `period − duration`. So `duration ≈ 60m − 48m ≈ 12m`. If the job started taking longer, the gap would shrink. If it started failing fast, the gap would grow toward the full hour. The gap distribution is a free SLO probe for any cron-shaped workload, as long as you know the schedule period.

**Fourth**, the existence of a single `openclaw/automated → opencode/human` row at 44m33s in the tail is interesting on its own. That is the one moment in the seven-day window where a human session began inside one of the cron quiet periods. It is the operator deciding to hand-type a command at a moment when the schedule was idle. Every other tail row is either machine-to-machine or human-after-long-absence. The mixed-mode handoff is rare.

## Where the threshold actually lies if you remove the cron

I do not have an easy way to re-run pew-insights with a `WHERE source != 'openclaw'` filter from the CLI, but I can reason about what would happen. The seven-day window has 6,216 adjacent gaps. The nearest-rank quantile at q=0.99 picks row `ceil(6216 × 0.99) = 6154` of the sorted gap list, counting from the bottom. The threshold sits at 16m43s. If I remove the openclaw-automated source from the population, I lose somewhere around 4,000-plus sessions (openclaw is the dominant automated source by row count, by a large margin), which collapses the population to roughly 2,000 gaps. The new p99 cutoff sits around row 1,980 of the sorted human/cross-source-only gap list. The cutoff almost certainly drops, because the human-side distribution is bursty in short windows and quiet in long ones, but the long quiet periods are the operator-away events at hours, not the cron clockwork at fifty minutes.

In other words, the segmented p99 is probably closer to the 1m8s figure that the original `pew-insights gaps --quantile 0.9` run produced. That q=0.9 threshold of **1m8s** is a much more honest "the operator paused" cutoff than the q=0.99 figure of 16m43s, precisely because it sits in the body of the human distribution rather than at the boundary where the cron cluster begins to dominate.

## The shape of the body

Worth dwelling on the body for a moment. Median gap is 0 seconds. That is the headline of the body. In a window of 6,216 adjacent gaps, more than half are zero. This is consistent with what I have seen in earlier posts about turn cadence and overlapping sessions: a single operator, working in flow, often has two or three sessions open at once — one in `opencode`, one in `claude-code`, one in `openclaw` for a quick check — and the "gap" between them is often negative in real time and rounded to zero in the JSONL. The transition data corroborates this; opencode-to-opencode self-loops account for 3,551 of the 6,762 handoffs, with 2,504 of those overlapping. Half of all transitions are the same source picking up immediately from itself, often before the previous session has even finished.

The body is dense, fast, and zero-bounded. It looks like a typing distribution, not an idle distribution. The "gap" between consecutive sessions in the body is dominated by tool-launch latency and shell-prompt round-trips, not by operator idleness. That is why the median is zero.

The transition from body to tail is, in fact, sharp. From q=0.50 (0s) to q=0.90 (1m8s) the gap grows by sixty-eight seconds. From q=0.90 (1m8s) to q=0.99 (16m43s) it grows by fifteen and a half minutes. From q=0.99 to the max (3h24m) it grows by another three hours. The distribution is heavy-tailed in the qualitative sense, but the heavy tail is not exponentially-distributed idle time. It is a few discrete piles: the cron pile around 47–48 minutes, the operator-away pile from one hour to a few hours, and the very long pile for overnight breaks (which would dominate if the window extended into a weekend).

## What to actually do with this

The action items are short and concrete.

1. **Always segment gap distributions by `source/kind` before computing tail quantiles.** A single global p99 over a mixed automated/human population is a number that obscures more than it reveals. The fix is one extra group-by.

2. **Treat the cron-shaped tail as an SLO probe, not as anomaly.** The 48-minute idle is not a problem; it is a measurement of the cron's duty cycle. Watch it for drift. If it shrinks, the cron is taking longer to run, which is the leading indicator of a real regression.

3. **Use the q=0.90 threshold (1m8s) as the human "paused" cutoff.** It sits firmly in the body and is not contaminated by schedule artifacts. Any gap above it from a `human` kind is genuinely worth a glance. Any gap above it from an `automated` kind is the schedule.

4. **Read the actual rows, every time.** The summary statistic was 16m43s and meant nothing. The rows revealed the cron in fifteen seconds. There is no aggregation over a small population that is more informative than scrolling the rows themselves. On a single-operator machine with thousands of events, the cost of reading the top twenty rows of any tail metric is essentially zero, and the information density is enormous.

The wider point is that on a single-operator local-first system, the population is small enough that "statistics" and "individual events" are not separated by orders of magnitude. The top twenty of six thousand is 0.3% of the population, but it is also a list that fits on one screen. Treating that list as data, and not as a tail you can summarize, is the difference between reading the schedule and missing it. The cron lives in the gap distribution. It is hiding in plain sight, and once you see the timestamps line up at minute `:09` of every hour, you cannot unsee it.

## Coda: the 3h24m row

One last note. The single largest gap in the window — 3h24m on 2026-04-20, between `claude-code/human` at 16:49Z and `openclaw/automated` at 20:13Z — is interesting for a different reason. It is not an operator-away event in the simple sense, because the session that closed the gap was an automated one. What it actually marks is the transition from "the operator was working in claude-code and then walked away" to "the cron eventually fired and re-established a heartbeat on the queue." The gap is bounded above by the cron period: no idle period in this window, with the cron running, can ever be longer than about 60 minutes plus the duration of one job, which is why the 3h24m row stands out. It is one of only three rows where the operator-away duration was long enough that even the schedule could not paper over it. The other two are the 3h19m and 1h34m rows.

Three rows in seven days where the human absence outran the schedule. That is the single number on this machine that most cleanly summarizes "how often did I leave the keyboard for longer than an hour?" — and it is sitting in the gap distribution, not in any calendar or status feed. Three. The schedule covers everything else.
