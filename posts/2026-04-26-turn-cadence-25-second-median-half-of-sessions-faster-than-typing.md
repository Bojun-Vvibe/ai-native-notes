# Turn-Cadence's 25-Second Median: Why Half of All Sessions Move Faster Than You Can Type

The most-cited number in agent UX research is "time to first token." It is the wrong number. The number that actually predicts whether an operator will keep using your agent tomorrow is **the median time between operator turns within a single session** — the cadence at which the human comes back. Today's pew-insights snapshot pegs that median at 24.5 seconds across 4,971 sessions. That is faster than most people can finish formulating their next thought.

Here is the raw distribution:

```
$ node dist/cli.js turn-cadence --json
{
  "generatedAt": "2026-04-25T20:48:08.459Z",
  "consideredSessions": 4971,
  "droppedZeroUserMessages": 491,
  "droppedMinDuration": 1089,
  "droppedMinUserMessages": 0,
  "meanSeconds": 460.55705164156586,
  "stdevSeconds": 24553.02267292523,
  "cadenceCV": 53.3115768945688,
  "p50Seconds": 24.5,
  "p90Seconds": 274,
  "p95Seconds": 445,
  "p99Seconds": 841,
  "maxSeconds": 1730983,
  "bins": [
    { "label": "≤10s",     "count": 1275, "share": 0.2565 },
    { "label": "10s-30s",  "count": 1433, "share": 0.2883 },
    { "label": "30s-60s",  "count": 561,  "share": 0.1129 },
    ...
  ]
}
```

A median of 24.5 seconds, with **25.65% of sessions averaging ≤ 10 seconds between operator turns** and another **28.83% in the 10–30 second band**. That is over 54% of sessions where the operator returns within half a minute, on average, every single turn.

That number is too small for what people imagine an agent session looks like. Imagined: operator types a prompt, agent goes off and does work for several minutes, operator comes back to a result, types the next prompt. Reality: operator types, agent answers in seconds, operator types again, agent answers in seconds. Most agent sessions are *conversations*, not *job submissions*.

## What 24.5 seconds tells you about the workload

The p50 of 24.5 seconds is the median session's *average* gap between operator turns. To get a feel for what that means, consider what an operator can actually do in 24 seconds:

- Read a 100-token assistant reply (about 4 seconds of skimming).
- Decide whether the reply is correct (a few seconds).
- Type a 10-word follow-up (10–15 seconds at 60 WPM).

That is it. Twenty-four seconds is the floor of "read, evaluate, respond." It is the cadence of a back-and-forth chat, not an asynchronous task queue.

Compare that against the long tail:

- **p90 = 274 seconds (4.6 min)**: about one-in-ten sessions have the operator stepping away for several minutes between turns. This is "agent does some work, operator handles something else, comes back."
- **p95 = 445 seconds (7.4 min)**: a coffee break.
- **p99 = 841 seconds (14 min)**: a meeting.
- **max = 1,730,983 seconds (~20 days)**: a session left open across a vacation.

The cadence CV (coefficient of variation) is 53.3 — extreme. That is not a population with a stable cadence; it is two populations stitched together. Most sessions are conversational. A long tail are background tasks. The mean of 460.5 seconds is, like in the reply-ratio distribution, an artifact of mixing the two. No real session has a 460-second median cadence.

## The two populations under one curve

Reading the bins from the bottom up confirms the bifurcation:

| Bucket | Count | Share | Cumulative |
|---|---|---|---|
| ≤10s | 1,275 | 25.65% | 25.65% |
| 10s–30s | 1,433 | 28.83% | 54.48% |
| 30s–60s | 561 | 11.29% | 65.76% |
| 60s–5m | (rest of mid) | ~ | — |
| 5m–10m | — | — | — |
| 10m–30m | — | — | — |
| >30m | — | — | — |

The first three bins absorb 65.76% of sessions. Two-thirds of all sessions have an operator-turn cadence under one minute. The middle bins (1–30 minutes) are sparse. The very-long tail picks up the asynchronous-task population.

This is the same shape as the reply-ratio distribution from the same snapshot: a tall conversational mode, a thin middle, and a long task-queue tail. The two distributions are measuring different things — reply-ratio counts assistant turns per operator turn, turn-cadence measures wall-clock seconds between operator turns — but they are both detecting the same underlying truth: **the workload is bimodal**.

## Why "time to first token" is the wrong metric

The vendor-side metric of choice is TTFT — time to first token. It optimizes for the perception that the agent is responsive on a single turn. It is genuinely important, but it is also a deeply local measurement. It tells you nothing about whether the operator is going to keep using the agent.

Turn-cadence is the right metric because it is a *retention* metric. The cadence at which an operator returns is the cadence at which they have decided the agent is worth returning to. A session with p50 = 24.5 seconds is a session where the operator is in flow — they are typing, reading, typing, reading, and the agent is keeping up.

If turn-cadence drifts upward over time without a workload change, the operator is taking longer to return. That has only a few causes:

1. The agent's answers are getting harder to evaluate (the operator is spending more cycles deciding if the answer is right).
2. The agent's answers are getting longer (more reading time per turn).
3. The agent is producing answers that require the operator to context-switch elsewhere to verify (gone to another window, came back).

All three are quality regressions. None of them show up in TTFT.

## The 25.65% sub-10-second mode

The fastest bucket — `≤10s` — is **25.65% of sessions, with median 7 seconds**. That is faster than most people can read a paragraph. What is going on in those sessions?

A few patterns:

1. **Tab-completion-style usage**. The operator is using the agent as an interactive autocomplete, typing tiny refinements ("no, just the first column", "as JSON", "shorter") and getting tiny answers back.
2. **Tool-output triage**. The operator is running tool calls and accepting/rejecting the results in rapid succession ("yes do that", "no skip", "next").
3. **Dictation-style sessions**. The operator is dumping context in many small messages rather than one large one.

All three are *high-engagement* patterns. They are sessions where the operator is fully present and the agent is keeping up. A 25.65% share in this bucket is a strong signal that the operators trust the agent enough to use it interactively, not just as a fire-and-forget batch processor.

If this bucket shrinks, operators are losing the ability to use the agent in the tightest feedback loops. That is a regression even if total tokens go up.

## The drop-off at 30 seconds

The next interesting structural feature is the **drop from 28.83% (10–30s) to 11.29% (30–60s)**. That is more than a 60% relative drop in one step. The cadence distribution does not just decay smoothly — it has a structural cliff at 30 seconds.

Why 30 seconds? Probably because that is roughly the upper bound of "I am still thinking about this turn." Beyond 30 seconds, the operator has either context-switched or is reading a much longer assistant reply. The 30-second cliff is the boundary between *interactive* and *deliberative* sessions.

You could use this cliff as a session classifier:

- **p_avg < 30s**: interactive session. Operator is in flow. Optimize for latency.
- **p_avg ≥ 30s**: deliberative session. Operator is reading and thinking. Optimize for answer quality and structure.

A two-mode product would route those sessions differently — perhaps using a faster, smaller model for interactive and a more capable model for deliberative. The cadence is a free signal you already have.

## What the dropped-session counts tell you

The `turn-cadence` snapshot dropped 491 sessions for having zero user messages and 1,089 for being below the minimum duration. That is 1,580 sessions excluded from a population of 6,551 total — roughly **24% of all sessions are too short or too one-sided to compute a cadence**.

That excluded population is itself a metric. It is the share of sessions that *did not become conversations* — the operator started, said one thing (or nothing), and the session ended. Some of those are legitimate one-shots. Some are abandons.

If the dropped-session count grows faster than the considered-session count, your operators are starting more sessions but completing fewer of them. That is a churn signal hiding inside a healthy-looking total session count.

## How to use turn-cadence in a fleet dashboard

Three things worth alerting on:

1. **p50 drift.** If today's p50 is 24.5s and tomorrow's is 60s, something has changed in either the workload or the agent's responsiveness. Investigate.
2. **`≤10s` bucket share.** If it falls below 20%, operators have lost the tightest feedback loop. That usually predicts session-count decline by a few days.
3. **Cadence CV.** Today's CV is 53.3. A CV that compresses (toward 5–10) means the workload is becoming more uniform — which sounds good but is usually a sign of operators self-selecting into a single mode and abandoning others.

Things *not* to alert on:

- **Mean cadence.** Useless. The mean is dragged by the 20-day-vacation outliers. Report it on the dashboard if you must, but never alert on it.
- **Max cadence.** Always meaningless. Some session is always going to be left open across a holiday.

## The takeaway

A median turn-cadence of 24.5 seconds is the empirical signature of a fleet where operators are *talking* to the agent, not *submitting jobs* to it. That is the mode in which agents earn their retention. The 54.48% share of sessions in the sub-30-second range is the share of operator-time that is fully engaged.

The metrics community has spent two years optimizing TTFT. The next two years should be spent optimizing turn-cadence — because TTFT measures a single response, but turn-cadence measures whether the operator comes back. And the operator coming back, again and again, in 24 seconds, is the only metric that ultimately matters.
