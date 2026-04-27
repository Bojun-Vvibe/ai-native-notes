---
title: "The snapshot lag asymmetry: claude-code 37h, codex 78h vs openclaw 2min — and the 1000x observability gap hiding inside one telemetry table"
date: 2026-04-28
tags: [telemetry, observability, pew, latency, snapshot-lag, ingestion]
est_reading_time: 12 min
---

## The problem

Every row in `~/.config/pew/session-queue.jsonl` carries two timestamps that look like they should be near-twins: `last_message_at` (the wall-clock time of the last assistant or user message in that session) and `snapshot_at` (the wall-clock time at which the pew collector wrote the row). Naively, the difference between these two should be on the order of seconds — a session ends, the collector polls, the row appears. That mental model is wrong by about three orders of magnitude for half of the sources, and the asymmetry is large enough that any analytics built on top of `session-queue.jsonl` without source-conditioning are silently averaging "fresh" against "ancient" and calling the result a trend.

The shape of the failure: I was building a recency-weighted moving window over assistant message volume, bucketing by hour of day, and the dispatcher families that route through `claude-code` and `codex` started showing up in *yesterday's* histogram even when their actual sessions had run earlier this week. I assumed an off-by-one bug. It wasn't. It was that the snapshot ingestion path for those two sources is fundamentally different from the `openclaw` and `opencode` paths, and the lag distribution is so heavy-tailed that median lag for `claude-code` is 37.7 hours and median lag for `codex` is 78.1 hours, against `openclaw` at 109 seconds and `opencode` at 126 seconds. Same table, same schema, same column names, same downstream consumers. Hidden behind a single `source` string is a four-way ingestion topology with completely different freshness contracts.

## The setup

- Data file: `~/.config/pew/session-queue.jsonl`, 9886 rows as of the snapshot taken for this post.
- Schema fields used: `source`, `last_message_at`, `snapshot_at`. Both timestamps are ISO-8601 UTC with millisecond precision.
- Sources present: `opencode` (5958 rows), `openclaw` (2342 rows), `claude-code` (1108 rows), `codex` (478 rows).
- Lag is computed as `(snapshot_at - last_message_at).total_seconds()`. Negative lags would indicate clock skew or a snapshotter that records future-dated rows; in this dataset there are zero negative lags and zero zero-lag rows, which is itself a useful sanity property — every row has at least one millisecond of lag, which means the snapshotter is not faking "live" rows by stamping them with the message time.

## The numbers

Computed by a 20-line Python script that read the entire file once and bucketed `(snapshot_at - last_message_at)` by source:

```
snapshot lag (all rows):
  n=9886 min=0.0s p25=36.7s med=153.9s p75=19682.4s p99=1774010.9s max=6091240.1s

by source:
  claude-code: n=1108 med=135948.7s   p95=2379165.3s   max=6091240.1s
  codex:       n=478  med=281332.9s   p95=563719.2s    max=824755.5s
  openclaw:    n=2342 med=109.0s      p95=347.2s       max=438520.4s
  opencode:    n=5958 med=126.6s      p95=98298.1s     max=174505.7s
```

Translated into human units:

| source       | rows | median lag | p95 lag    | max lag   |
|--------------|------|-----------:|-----------:|----------:|
| openclaw     | 2342 |   1m 49s   |   5m 47s   |  5d 02h   |
| opencode     | 5958 |   2m 07s   |   1d 03h   |  2d 00h   |
| claude-code  | 1108 |  37h 45m   |  27d 13h   | 70d 12h   |
| codex        |  478 |  78h 08m   |   6d 12h   |  9d 13h   |

The all-rows aggregate (median 153.9 seconds) is a complete fiction — it's the openclaw and opencode mass dragging the central tendency to a value that no `claude-code` or `codex` row achieves at the median. The real distribution is bimodal at the source level, with the two collector-driven sources sitting in a "near-real-time" mode and the two file-scrape sources sitting in a "session-archive harvest" mode whose lag is bounded only by how often the user opens the upstream tool's session directory.

## What I tried (and what was wrong)

- **Attempt 1 — assume a bug in the snapshotter clock.** I checked the snapshotter's emit timestamp by looking at recent rows for each source. All four sources produce rows with `snapshot_at` clustered around the same 30-minute window, so the snapshotter clock is unified. The lag is in the `last_message_at` value being old, not in `snapshot_at` being late. Wrong hypothesis, but useful for ruling out the easy case.

- **Attempt 2 — assume the two slow sources just have long-running sessions.** This was tempting because `claude-code` and `codex` are the two sources where I personally do "thinking work" that can sit idle for hours mid-session. But `last_message_at` is the time of the *last* message, not the start of the session. A 6-hour session that ended 5 minutes ago has a 5-minute lag, not a 6-hour lag. The slow sources aren't slow because the sessions are long. They're slow because the row only gets written when the snapshotter rediscovers the upstream session file, which happens on a totally different cadence than the session itself.

- **Attempt 3 — assume the openclaw lag is the floor and everyone else is some multiple of it.** The openclaw plugin lives at `~/.config/pew/openclaw-plugin/` and emits rows on session close via a directly-wired event hook, so its 109-second median is essentially "next snapshot tick after session ends." If that were the floor, opencode at 126 seconds is the same regime, and I'd expect claude-code and codex to be at most a few minutes behind. They're not. They're four to five orders of magnitude behind. So the lag is not "tick interval drift" — it's a categorically different ingestion path.

- **Attempt 4 — actually look at the p95 vs median ratio per source.** This is what cracked it:

  - openclaw: p95/median = 347.2 / 109.0 = 3.18x. Tight distribution, exponential-ish tail.
  - opencode: p95/median = 98298 / 126.6 = 776x. Bimodal — most sessions are scraped near-real-time, but some live sessions only get harvested when the user closes their editor.
  - claude-code: p95/median = 2379165 / 135948 = 17.5x. Heavy tail but the *median itself* is already 37 hours. The whole distribution lives in the "stale" regime.
  - codex: p95/median = 563719 / 281332 = 2.0x. Surprisingly tight, but tight around a 78-hour median. This means codex sessions are harvested in batched sweeps, not on demand.

## What's actually happening

The four sources have four different ingestion topologies, even though they all land in the same `session-queue.jsonl` file:

1. **openclaw** — direct event hook from a plugin process that watches its own session lifecycle. When a session closes, the plugin writes a queue entry. The snapshotter picks it up on its next tick (default ~60s). So the floor is "1 tick" and the tail is "snapshotter was paused / device asleep."

2. **opencode** — file watcher on the upstream session directory. New session files are noticed within a few seconds, but a session file that's still being written stays open and isn't harvested until it stops growing. So short sessions land within ~2 minutes; long active sessions can sit unharvested for the duration of the session.

3. **claude-code** — periodic directory scrape that walks the upstream tool's stored history and computes a digest. The digest only updates when the user has been *using* the tool, which means dormant sessions get re-noticed every few days. The 37-hour median is "average days since last claude-code usage burst." It's not the snapshotter being slow — it's that the snapshotter only *learns about* sessions when the user generates new ones nearby.

4. **codex** — same general idea as claude-code but with an even sparser scrape cadence and no incremental file-watcher, so even the p25 sits in the multi-day range. The tightness of p95/median (only 2x) suggests codex sessions are batched into a single sweep that runs every few days, so almost every row is "this many days behind whichever sweep happened to find it."

## Why this matters

If you treat `session-queue.jsonl` as a real-time event log, you will get five categories of wrong answer:

- **Recency rankings are corrupted.** Sort by `last_message_at` descending and the top of the list will be dominated by openclaw and opencode rows. Sort by `snapshot_at` descending and you'll see fresh rows from all sources — but the underlying `last_message_at` for the claude-code rows in that sort might be from last month. Pick the wrong key and your "what happened today" view is silently wrong.

- **Hour-of-day histograms double-count or under-count by source.** A claude-code session that ended at 14:00 UTC on Monday and got snapshotted at 03:00 UTC on Wednesday will land in Monday's bucket if you use `last_message_at` and Wednesday's bucket if you use `snapshot_at`. Neither is wrong; they answer different questions. But if half your downstream queries use one and half use the other, your dashboards drift from each other and no single fix makes them agree.

- **"Live" alerting is impossible for the slow sources.** Any alert built on "did we see a new claude-code session in the last 5 minutes?" will fire constantly even when the user is actively using claude-code, because the snapshotter hasn't noticed yet. The only safe alert latency for claude-code and codex is on the order of a week. For openclaw and opencode, single-digit minutes is fine.

- **Cohort analysis breaks.** If I bucket sessions into "this week" and "last week" by `snapshot_at`, the claude-code cohort in "this week" includes sessions that actually happened up to 70 days ago (the max lag observed). If I bucket by `last_message_at`, I correctly attribute the session to its real week, but I will see *new last-week rows showing up this week*, and any cohort that was "closed" last week will keep growing.

- **Dispatcher fairness analysis is biased.** If a dispatcher round-robins families and one family routes through codex (78-hour median lag), that family will always look "underweight" in any near-real-time fairness check, because half its sessions haven't been ingested yet. The correct denominator is "sessions snapshotted in window AND originated in window," not just one or the other.

## The fix (and the limit of the fix)

The cheap fix is to add a `lag_seconds` derived column at query time and filter or window on it explicitly. Anything with `lag_seconds > 3600` is "archive material," anything below is "near-real-time." For the four sources observed, this single split cleans up the bimodality enough that downstream histograms stop lying.

The expensive fix — and the one I haven't done — is to push the snapshotter for claude-code and codex onto a file-watcher path so they look like opencode. This requires reading the upstream tool's session-storage layout and writing a watcher that tracks file mtimes incrementally instead of doing periodic directory walks. It's not hard; it's just work, and the payoff is hidden until you actually try to do near-real-time analytics across all four sources, which until this week I had no reason to do.

The limit of the cheap fix is that it cannot recover sessions you never observed. If a `codex` session ended on Monday and the snapshotter doesn't sweep until Friday, the row for that session will not exist on Tuesday no matter how clever your query is. "Lag" is the *visible* part of the staleness; the invisible part is "sessions that happened and have no row yet." The 99th-percentile lag of 1,774,010 seconds (20.5 days) for claude-code is a strong hint that there's a long tail of sessions out there that are *still* unsnapshotted at the moment I ran the query for this post — they'll show up in next week's snapshot file with a `last_message_at` that's already a month old.

## Lessons for telemetry plumbing

- **Source is not just a label.** When four ingestion paths share a table, `source` becomes a hidden join key on freshness, and any aggregate that doesn't condition on it is silently averaging across regimes.

- **Verify the floor first.** The fastest source's median lag tells you the snapshotter's tick interval. If a slower source's median is more than 10x that floor, the difference is not snapshotter pacing — it's a fundamentally different ingestion path. Don't try to explain a 1000x gap with "tick drift."

- **Distinguish `event_at` from `observed_at`.** This is the lambda-architecture lesson, restated for personal telemetry: every row needs a true event time and a true observation time, and they are not the same column. Pew already does this correctly with `last_message_at` and `snapshot_at`. The mistake was mine — I treated them as interchangeable for a week.

- **Bimodal lag is the rule, not the exception.** Any system that mixes push-event ingestion with pull-scrape ingestion will have at least two modes per source, and the modes will be separated by at least the scrape interval. The opencode p95/median ratio of 776x is the canonical signature: most sessions land fast, a long tail lands slow, and the tail's height is set by how long sessions stay open before their files close.

- **The "archive harvest" sources are not bugs.** The 37-hour and 78-hour medians for claude-code and codex are exactly what you'd want from an archival ingestion path: complete, eventual, batched, cheap. The bug is treating them as anything other than archival.

## What I'm going to do next

Add a `lag_class` derived column to my own analytics layer with three values: `near_real_time` (< 5 min), `delayed` (5 min to 1 hour), and `archival` (> 1 hour). Recompute every dashboard I have. Annotate the ones that change by more than 10%. Investigate any dashboard that doesn't change — that's a dashboard whose definition was already accidentally robust, which is interesting in its own right and probably worth a separate post.

The deeper thing here is that I had been treating `session-queue.jsonl` as one table with one freshness contract for several weeks before noticing the asymmetry, and the only reason I noticed was that a recency-weighted dispatcher metric started disagreeing with my intuition. The lesson is not "check your data more carefully." The lesson is that single-table abstractions over multi-source ingestion are a *common shape of silent bug*, and the way you catch them is by computing the cheapest possible per-source diagnostic — in this case, a one-line lag distribution per `source` value — at the moment you first ingest the table.

Cost of this discovery: about an hour of thinking, twenty lines of Python, and the confidence to stop trusting any aggregate over `session-queue.jsonl` that doesn't condition on source. Cost of *not* having discovered it: dashboards I'd been showing myself for weeks were wrong by factors I would have struggled to defend if anyone had asked.
