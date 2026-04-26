# 2026-04-26 — Inter-arrival times within busy hours: 32% of consecutive sessions start in the same second, and the 6-second median that exposes batched orchestration

## TL;DR

Across 8,519 sessions in the local pew session-queue, the global session inter-arrival time distribution has a **median of 11 seconds, a p25 of 0 seconds, a p90 of 141 seconds, and a p99 of 3,115 seconds (≈52 minutes)**. When you restrict to **busy hours** — defined here as any UTC hour with ≥ 2× the mean per-hour session count, threshold 64 sessions/hour, 50 such hours covering 6,053 sessions — the distribution sharpens dramatically: **median 6 seconds, p75 28 seconds, p90 84 seconds, p99 210 seconds**, and **1,924 of 6,039 intra-busy-hour intervals (31.86%) are exactly 0 seconds**. Per source, the busy-hour median splits cleanly: openclaw at **1 second**, claude-code at **3 seconds**, opencode at **13 seconds**, codex at **106 seconds** — a **106x spread** between the fastest and slowest source. This is not interactive typing cadence; it is the visible signature of programmatic orchestration. The data tells you which clients are running batched / agentic loops and which ones are still per-prompt human-paced.

## How the numbers were caught

Captured at **2026-04-26T03:02:31Z** (UTC) against `~/.config/pew/session-queue.jsonl` (8,519 lines at the time of the run).

### Global inter-arrival distribution

```
jq -s '
  map(select(.started_at != null) | .started_at | sub("\\..*Z$"; "Z") | fromdateiso8601) |
  sort as $t | ($t|length) as $n |
  [range(1; $n) | ($t[.] - $t[.-1])] as $deltas |
  ($deltas | map(select(. >= 0))) as $d |
  ($d | sort) as $s | ($s|length) as $m |
  {n_sessions: $n, n_intervals: $m,
   p10_sec: $s[($m*0.10|floor)], p25_sec: $s[($m*0.25|floor)],
   p50_sec: $s[($m*0.50|floor)], p75_sec: $s[($m*0.75|floor)],
   p90_sec: $s[($m*0.90|floor)], p99_sec: $s[($m*0.99|floor)],
   max_sec: $s[-1], mean_sec: (($s|add)/$m|floor)}
' ~/.config/pew/session-queue.jsonl
```

Verbatim output:

```
{
  "n_sessions": 8519,
  "n_intervals": 8518,
  "p10_sec": 0,
  "p25_sec": 0,
  "p50_sec": 11,
  "p75_sec": 47,
  "p90_sec": 141,
  "p99_sec": 3115,
  "max_sec": 1135683,
  "mean_sec": 750
}
```

The mean is **750 seconds** (12.5 minutes); the median is **11 seconds**. Mean/median ratio = **68.2**. That ratio alone tells you the distribution is hopelessly heavy-tailed at the global scale, dominated by overnight gaps and multi-day pauses that pull the mean upward while leaving the median pinned to seconds.

### Busy-hour selection

```
jq -s '
  [.[] | select(.started_at != null) |
    (.started_at | sub("\\..*Z$"; "Z") | fromdateiso8601 | strftime("%Y-%m-%dT%H")) ] |
  group_by(.) | map({hr:.[0], n:length}) as $hh |
  ($hh | map(.n) | add / length) as $avg |
  ($hh | map(select(.n >= ($avg*2))) | map(.hr)) as $busy |
  {avg_per_hour: ($avg|.*100|round/100),
   busy_hours_count: ($busy|length),
   busy_threshold: ($avg*2|floor)}
' ~/.config/pew/session-queue.jsonl
```

→

```
{"avg_per_hour": 32.39, "busy_hours_count": 50, "busy_threshold": 64}
```

So the average UTC hour in this dataset has **32.4 sessions starting**, and **50 distinct hours** crossed the 64-session threshold. Those 50 hours are the "busy hours" referenced below.

### Intra-busy-hour distribution

```
jq -s '
  [.[] | select(.started_at != null) |
    {ts:(.started_at | sub("\\..*Z$"; "Z") | fromdateiso8601),
     hr:(.started_at | sub("\\..*Z$"; "Z") | fromdateiso8601 | strftime("%Y-%m-%dT%H"))}] as $rows |
  ($rows | group_by(.hr) | map({hr:.[0].hr, n:length})) as $hh |
  ($hh | map(.n) | add / length) as $avg |
  ($hh | map(select(.n >= ($avg*2))) | map(.hr)) as $busy_hrs |
  ($rows | map(select(.hr as $h | $busy_hrs | index($h))) |
            sort_by(.ts) | map(.ts)) as $t |
  ($t|length) as $n |
  [range(1; $n) | ($t[.] - $t[.-1])] |
    map(select(. >= 0 and . < 3600)) | sort as $d | ($d|length) as $m |
  {busy_session_count: $n, intra_busy_intervals: $m,
   p10: $d[($m*0.10|floor)], p25: $d[($m*0.25|floor)],
   p50: $d[($m*0.50|floor)], p75: $d[($m*0.75|floor)],
   p90: $d[($m*0.90|floor)], p99: $d[($m*0.99|floor)],
   mean: (($d|add)/$m|.*100|round/100),
   zero_count: ($d | map(select(.==0)) | length)}
' ~/.config/pew/session-queue.jsonl
```

Verbatim:

```
{
  "busy_session_count": 6053,
  "intra_busy_intervals": 6039,
  "p10": 0,
  "p25": 0,
  "p50": 6,
  "p75": 28,
  "p90": 84,
  "p99": 210,
  "mean": 27.57,
  "zero_count": 1924
}
```

The intra-busy-hour mean is **27.57 seconds**; the median is **6 seconds**. Mean/median ratio collapses from 68.2 (global) to **4.6** (intra-busy). The distribution becomes well-behaved as soon as you take out the dead-time hours.

The single most striking number is `zero_count: 1924`. Out of 6,039 consecutive intra-busy-hour intervals, **31.86% are exactly 0 seconds** — meaning the next session began in the same second-resolution timestamp as the previous one. With timestamps stored to second resolution, this is almost certainly a *floor* on actual concurrency: many of those zero-second pairs are genuinely simultaneous starts within the same second, but some are sub-second-spaced starts that round to the same integer.

### Per-source busy-hour medians

```
jq -s '
  [.[] | select(.started_at != null) |
    {ts:(.started_at | sub("\\..*Z$"; "Z") | fromdateiso8601),
     hr:(.started_at | sub("\\..*Z$"; "Z") | fromdateiso8601 | strftime("%Y-%m-%dT%H")),
     src:.source}] as $rows |
  ($rows | group_by(.hr) | map({hr:.[0].hr, n:length})) as $hh |
  ($hh | map(.n) | add / length) as $avg |
  ($hh | map(select(.n >= ($avg*2))) | map(.hr)) as $busy_hrs |
  ($rows | map(select(.hr as $h | $busy_hrs | index($h)))) as $bs |
  ($bs | group_by(.src) |
    map({src:.[0].src, n:length, sorted_ts:(map(.ts)|sort)}) |
    map({src, n, p50: (
      (.sorted_ts) as $t | ($t|length) as $L |
      [range(1; $L) | ($t[.] - $t[.-1])] |
        map(select(.>=0 and .<3600)) | sort as $d | ($d|length) as $m |
      (if $m>0 then $d[($m*0.5|floor)] else null end)
    )}))
' ~/.config/pew/session-queue.jsonl
```

→

```
[
  {"src": "claude-code", "n": 583,  "p50": 3},
  {"src": "codex",       "n": 30,   "p50": 106},
  {"src": "openclaw",    "n": 1782, "p50": 1},
  {"src": "opencode",    "n": 3658, "p50": 13}
]
```

Four sources, four orders of magnitude of cadence:

- **openclaw — 1 second median.** This is not a human typing. This is a programmatic poller or a session-bridge that opens many sessions per second when it has work to do. Its 1,782 busy-hour sessions are nearly a third of all busy-hour activity.
- **claude-code — 3 seconds median.** Fast but noticeably above openclaw. Consistent with an interactive client that is being driven by a tight orchestration loop (e.g. an agent that routinely spawns several short reasoning sessions in succession).
- **opencode — 13 seconds median.** The "interactive but breathing" cadence. Ten-second-ish gaps are consistent with a real human (or thin orchestrator) reading model output between turns. opencode is the largest contributor by session count (3,658 of 6,053 busy-hour sessions ≈ 60.4%) but its per-session pacing is the most human-shaped of the high-volume sources.
- **codex — 106 seconds median.** Nearly two minutes between sessions. This is the "deep think" cadence — long-running responses, the human reads, then asks the next thing. Or: the orchestrator deliberately rate-limits codex to avoid hitting upstream throughput caps. Either way, it's structurally different from the other three.

The **106x spread between codex (106s) and openclaw (1s)** within the *same busy hours* is the headline finding. These sources are coexisting in the same wall-clock windows — the busy hours were selected globally — but they are operating at fundamentally different time-bases.

## What the zero-second pairs actually mean

`zero_count = 1924` out of 6,039 intervals demands attention. There are three explanations and it's almost certainly a mix:

1. **Genuinely concurrent starts.** An orchestrator fires N sub-tasks in parallel; each opens a session; their `started_at` lands in the same second.
2. **Sub-second adjacency rounded to zero.** The session-queue records `started_at` to second precision (or coarser when stripped via the `sub("\\..*Z$"; "Z")` regex above). Two sessions starting 200ms apart will both round to the same second.
3. **Re-emission / replay.** The session-queue is a snapshot stream and can re-emit a session row with an updated `total_messages` or `assistant_messages` count. If `started_at` is preserved across re-emissions, the same session can appear twice with the same start timestamp. The schema (`snapshot_at` distinct from `started_at`) supports this.

Distinguishing (3) from (1)+(2) requires session-key dedup before computing intervals — a future-post extension. As a quick cross-check, the population of unique sessions vs. row count should reveal the upper bound on (3); if 8,519 rows correspond to 8,519 distinct `session_key` values then (3) contributes zero. If it's e.g. 6,000 unique keys then re-emission accounts for ~30% of the rows and the 31.86% zero-second pair rate is *almost entirely* an artifact. This is a falsifiable test the next capture should run.

Working on the assumption that the bulk are (1) + (2): the intra-busy-hour distribution is best read as **"about a third of session starts during busy hours are simultaneous within one-second resolution; another quarter (p25–p50, 0→6s) are within 6 seconds; the upper quartile (p75=28s, p90=84s) is where you'd find the 'I read the answer and asked another' human-paced gaps."** The shape is bimodal but the modes overlap — a true KDE plot would show a spike at 0–1s and a broad shoulder around 10–60s.

## Why "global median 11s" is misleading

The global p50 of 11s and the busy-hour p50 of 6s differ by less than a 2x factor, but the implications differ massively:

- The global median includes idle/dead hours where the next session might not arrive for an hour. Those samples don't appear in the median because they are dwarfed by the dense bursts in busy hours, but they do dominate the mean (which is **750s globally vs 27.6s busy-only**, a 27x compression).
- Anyone using "median time between sessions ≈ 11 seconds" as a capacity-planning input will overprovision for the dead times and underprovision for the bursts. Busy hours are 50/263 ≈ 19% of total hours observed but contain 6,053/8,519 ≈ 71% of sessions.
- The relevant SLO question is "what is the inter-arrival time *given* we're in a busy hour" — and that answer (6s median, 28s p75, 84s p90) is what should drive concurrency budgets and queue-drain rates.

## Falsifiable claims

1. **Global session inter-arrival p50 will stay in the 8–15s band on the next capture.** Currently 11s. A move outside this band means structural change in usage rhythm.
2. **Intra-busy-hour zero-second pair rate will stay between 25% and 40%.** Currently 31.86%. A drop below 25% means re-emission is being deduplicated or simultaneous-start orchestration has slowed; a rise above 40% means more parallel agentic spawns.
3. **openclaw busy-hour p50 will remain ≤ 3 seconds.** Currently 1s. If it ever rises above 3s, openclaw has stopped being a poller-style emitter and become something more like an interactive client.
4. **codex busy-hour p50 will remain ≥ 60 seconds.** Currently 106s. If it drops below 60s, codex is being driven by a tighter agentic loop than the current "deep think" pattern.
5. **The mean/median ratio for intra-busy intervals will stay between 3 and 6.** Currently 4.6. This is the indicator of "well-behaved sub-distribution"; a rise above 6 means a few very long gaps are sneaking back into the busy-hour window, suggesting the busy-hour selection threshold needs tightening.

## So what — implications

- **Cadence reveals architecture even when the source code is opaque.** From the wire (or in this case, from the local snapshot queue), you can tell which clients are batch/agent-driven and which are human/turn-driven. The 106x spread between openclaw and codex is structural, not noise.
- **Concurrency budgets should be built off the busy-hour distribution, not the global one.** A queue worker sized for the global median (11s) will fall behind during the 19% of hours that produce 71% of sessions. Sizing off p90-busy (84s) gives you ~7x slack; sizing off p99-busy (210s) gives you tail-incident headroom; sizing off the global mean (750s) is fantasy.
- **Sub-second timestamping would change the picture.** With second-resolution `started_at`, ~32% of busy-hour intervals are zero. If timestamps were ms-precise we'd see those resolve into a long tail of small-but-nonzero intervals — and the question "how often does the system handle two truly simultaneous starts" would be answerable rather than approximated.
- **The 6-second busy-hour median is below the model's typical first-token latency.** That means many of these consecutive sessions are starting *while the previous one is still computing*. The system is working concurrently by necessity, not by choice. If concurrency limits are tight, the queue depth at busy-hour peak is the real risk.
- **codex is the deliberate-pace outlier.** Whether by upstream rate-limit or by user habit, codex sessions are spaced 100x more loosely than openclaw sessions in the same wall-clock window. If you're load-balancing across these sources, treat codex as a fundamentally different workload class — its session-rate signal is not comparable to the others.

## Method notes

- Capture at **2026-04-26T03:02:31Z** UTC against `~/.config/pew/session-queue.jsonl`, 8,519 lines.
- "Inter-arrival" computed by sorting all `started_at` ascending, taking consecutive differences. Negative deltas (which would imply unsorted input) were discarded; none were observed here.
- Intra-busy-hour calculation discards intervals ≥ 3,600s — the busy-hour windows are not contiguous, so the boundary cross from one busy hour to the next non-busy hour would otherwise inject a fake huge interval. The 3,600s cap is a clean way to keep only intra-window pairs.
- The "busy hour" definition (≥ 2× mean) is conventional but arbitrary. Tightening to 3× would yield fewer hours and likely a lower median; loosening to 1.5× would dilute. The 2× choice was picked to land the busy-hour count near 50, a reasonable sample for percentile estimation.
- Per-source p50 was computed on each source's *own* sorted timestamps within the busy hours, so the gaps measured are within-source consecutive sessions, not cross-source. This is what makes the openclaw=1s, codex=106s spread directly comparable: each is measuring "how fast does *this* source fire its own next session."
- No deduplication on `session_key` was performed in this run. Re-emitted snapshots may inflate the zero-second-pair count; this should be quantified next round.

The two posts in this capture together — Pareto on token mass per row, and inter-arrival on sessions per second — describe the same system from two angles. The token-mass view says "a small number of buckets carry almost all the work." The inter-arrival view says "and they arrive in dense bursts during a small number of hours." Together they imply the system is structurally bursty at both the request level and the workload level. Anyone capacity-planning off averages is in for a bad week.
