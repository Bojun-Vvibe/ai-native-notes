---
title: "Reading session-queue.jsonl: what longest-session and chattiest-session actually tell you"
date: 2026-04-24
tags: [observability, agents, jsonl, distributions, telemetry]
est_reading_time: 11 min
---

## The problem

Every agent session I've run in the last four months has appended a record to `session-queue.jsonl`. The record has a session id, a start time, an end time, a turn count, a token count, and the agent name. About 2,400 entries by now. For most of those four months I treated this file the way most people treat their browser history: as an audit trail to grep when something went wrong, not as data.

Then last week I wrote two cheap CLI tools, `longest-session` and `chattiest-session`, that just sort the file and print the top entry. They take maybe forty lines of code each. I thought they would be useless toys. Instead they have become two of the most informative pieces of telemetry I have, because the answers they produce force you to confront a fact about agent sessions that I had been hiding from: **the distribution is fat-tailed, the mean is a lie, and the operator's behavior is encoded in the tail much more than in the body.**

This post is about what those two tools actually tell you, why p95 vs. max is the interesting comparison rather than mean vs. median, what a 66-hour session means, and how I now use these numbers to debug not the agents but myself.

## The setup

`session-queue.jsonl` is one record per session, appended at session end. Sessions can be from any of the agents I run: a few CLI assistants, a couple of orchestrators, one long-running watcher. Schema is small:

```json
{"session_id": "...", "agent": "primary", "started_at": 1745000000, "ended_at": 1745003600, "turns": 42, "tokens_in": 18000, "tokens_out": 6500}
```

Two CLIs read this file:

- `longest-session` — sorts by `ended_at - started_at` descending, prints the top one.
- `chattiest-session` — sorts by `turns` descending, prints the top one.

Both have a `--top N` flag for the top N rather than just the max, and both have a `--by` flag (`duration`, `turns`, `tokens`, `tokens_per_turn`) so they're really the same tool wearing two hats. I keep them as two commands because I look at them at different times for different reasons.

## What I tried

### Attempt 1: just look at the mean

The first thing I did with the file was compute means. Mean session duration: 14 minutes. Mean turn count: 9. Mean tokens per session: 12k. These numbers are accurate and useless. They are accurate because I computed them correctly. They are useless because the distribution they describe has the shape of a power law with a long, heavy tail, and the mean of a heavy-tailed distribution is dominated by a handful of extreme values.

I learned this the hard way when I tried to use the mean to budget compute. "If the mean session is 14 minutes, then 100 sessions per day is 1,400 minutes, so I can fit them in 24 hours." This logic is fine on paper and wrong in practice, because on any given day, two or three sessions will be 4+ hours each and the rest will be 1–2 minutes. The mean smears that across the population and you end up with a number that describes no actual session.

### Attempt 2: use the median

The median of session duration on this file is about 90 seconds. That is also useless, in the opposite direction. If I budget for 90-second sessions, I will be wrong every time the agent does anything interesting. The median describes the *most common* session — the ones where I asked a quick question and exited. The interesting work happens in the tail.

### Attempt 3: p95, p99, and max

This is where it got useful. The p95 of session duration is about 47 minutes. The p99 is about 4 hours. The max, as of this morning, is **66 hours and 11 minutes.** That single session is from a watcher process I left running over a long weekend. It dominates the entire distribution by itself.

The interesting question is not "what is the max" — that's a stunt. The interesting question is **how much daylight is between p95, p99, and max**. If p99 is 4 hours and max is 66 hours, that ratio of ~16x is telling you something about the *shape* of the tail, not just its location.

In a normal-ish distribution, p99 and max should be close — within a factor of 2 or so for any reasonable sample size. When max is 16x p99, you have one of two things:

1. A genuine outlier that should not be in the data (a stuck process, a crash that didn't get logged, a session left open during sleep).
2. A real mode of usage that the rest of the distribution doesn't capture (a long-running watcher, a batch job, an overnight run).

For me it was #2. The 66-hour session is real. It is a watcher I run intentionally. But it shouldn't be in the same distribution as the 90-second "what does this error mean" sessions, because they're different *things*. Once I tagged sessions by mode (`interactive`, `watcher`, `batch`), the distributions per-mode looked sane and the cross-mode aggregate looked like the bimodal mess it actually was.

## What worked

The thing that made these tools genuinely useful, rather than cute, was adding **deterministic tie-breaks** and **per-agent breakdowns**.

### Deterministic tie-breaks

Without a tie-break, `longest-session --top 5` is a non-deterministic selection whenever two sessions have the same duration to the second. This sounds rare, but it isn't, because I have a lot of sessions that exit immediately after a startup error and they all have duration 0 or 1. Asking for "the top 5 longest" when half the file has duration 0 is fine; asking for "the top 5 shortest" with no tie-break gives you a different answer every run.

I now sort by `(duration_desc, started_at_asc, session_id_asc)`. The session id is a UUID, so it's a stable last resort. The `started_at` ordering means that when I run the tool twice, I get the same answer, which means I can cite a number in a note and trust that it'll still be true tomorrow. (It also means the answer is *interpretable* — older sessions sort first within a tie — which I find more useful than UUID order alone.)

Deterministic tie-breaks are a small thing that has an outsized effect on whether you trust your own observability. If two runs of the same query return different answers, you stop quoting the query, even if the difference is in a tie-breaker that doesn't matter. The fix is cheap; the cost of not having it is that you lose confidence in the tool.

### Per-agent breakdowns

The aggregate distribution is bimodal because I have multiple agents with different usage patterns. The watcher dominates the duration tail. The orchestrator dominates the turn-count tail. The interactive agents dominate the body of the distribution.

`chattiest-session --by turns --group-by agent` told me something I hadn't noticed: the orchestrator has, on average, 3x more turns per session than any interactive agent, but its tokens-per-turn are 5x lower. It is having lots of short conversations with sub-agents. That is the *correct* shape for an orchestrator, but I had been mentally lumping it in with the interactive agents and worrying about why my "average" turn count looked low. It looked low because the orchestrator was pulling it down with lots of cheap turns.

This is the kind of thing that's invisible if you only look at the aggregate. P95 turns across all agents is 38. P95 turns for the orchestrator alone is 92. Different number, different story, different action.

## What a 66-hour session actually means

I want to dwell on the longest session because it taught me something about myself.

That session was a watcher I started Friday afternoon and forgot about until Monday morning. It was doing useful work the whole time — polling a queue, running short tasks, idling between them — but it was also accumulating context, and by the end it had ~140k tokens of conversation history, which means every new turn was paying for that history in the prompt cache (or worse, not in the prompt cache). The session cost about 4x what it would have cost if I had restarted the watcher every 6 hours.

The lesson is not "don't run long sessions." The lesson is **the existence of a 66-hour session in your queue is a signal that the operator (me) does not have a session-lifetime policy.** I was making a decision by not making one. The agent did exactly what I asked: keep going until told to stop. It would have kept going for another month if I hadn't noticed.

I now have a launchd job that SIGTERMs any watcher session older than 12 hours, and the watchers are written to checkpoint their state to disk so they can be restarted cheaply. The tail of the duration distribution dropped immediately. The p99 went from 4 hours to about 75 minutes. The max went from 66 hours to ~12 hours, exactly the cap.

The shape of the tail is a mirror. If your max is much larger than your p99, you have a policy gap. The number isn't telling you about your software; it's telling you about your operating discipline.

## p95 vs. max as a workflow

I now check these two numbers — p95 and max, for both duration and turns — about once a week. The check takes 30 seconds:

```bash
longest-session --top 1 --by duration
longest-session --top 1 --by duration --percentile 95
chattiest-session --top 1 --by turns
chattiest-session --top 1 --by turns --percentile 95
```

The thing I'm looking for is not the absolute number. It's the *ratio* of max to p95. When that ratio is roughly 2x, the system is behaving normally. When it goes above 5x, something is in the tail that shouldn't be — usually a stuck process, occasionally a real new use case I haven't accounted for.

I have started doing the same check on a few other JSONL files in the same project (`tool-calls.jsonl`, `errors.jsonl`, `commits.jsonl`). The same shape shows up everywhere. **Append-only logs of human-or-agent-driven activity are almost always fat-tailed**, because both humans and agents have rare expensive operations mixed in with frequent cheap ones. The mean lies. The median lies. The p95-to-max ratio tells the truth.

## What I would do differently

I would have written `longest-session` and `chattiest-session` on day one of running a JSONL log, not month four. They are two of the cheapest tools I have ever written and two of the most informative. The barrier was that they felt too simple to bother with — "I can just `jq`" — and the answer is: yes, you can, but you won't, because `jq` doesn't enter your muscle memory the way `longest-session` does. A named CLI is a *commitment* to look at the number; a shell one-liner is a thing you write once and forget.

The other thing I would do differently is tag sessions by mode from the start. Without mode tags, the aggregate distribution is a bimodal mess, and the tools have to work harder to be useful. With mode tags, the per-mode distributions are clean and the cross-mode aggregate becomes obviously meaningless, which is the right outcome — you stop computing it.

## Links

- The Pareto distribution Wikipedia page, which is the right mental model for session duration even if your data isn't strictly Pareto
- Gil Tene's "How NOT to Measure Latency" talk, which is the canonical reference on why means and medians lie about latency. Replace "latency" with "session duration" and the entire talk applies.
- `man jq` — every JSONL tool you write is a more committed version of a `jq` one-liner you already know
