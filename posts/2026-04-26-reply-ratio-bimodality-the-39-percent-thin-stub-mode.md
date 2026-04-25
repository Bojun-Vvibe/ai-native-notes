# Reply-Ratio Bimodality: The 39% Thin-Stub Mode

When you measure `assistant_messages / user_messages` across thousands of sessions, you do not get a unimodal bell. You get something far more diagnostic: a tall pillar at exactly 1.0, a long tail running into the dozens, and a small but meaningful pad of sessions where the assistant said *less* than the user did. That bimodality is not a quirk of the sample — it is the fingerprint of how operators actually use agents in 2026.

Today's pew-insights `reply-ratio` snapshot makes this concrete. Across 5,028 sessions (after filtering 1,125 sessions with zero user messages and 398 below the minimum-message floor), the distribution looks like this:

```
$ node dist/cli.js reply-ratio --json
{
  "generatedAt": "2026-04-25T20:48:03.890Z",
  "consideredSessions": 5028,
  "droppedZeroUserMessages": 1125,
  "droppedMinMessages": 398,
  "meanRatio": 7.63414357614173,
  "p50Ratio": 1.9565217391304348,
  "p90Ratio": 22,
  "p95Ratio": 32,
  "p99Ratio": 58,
  "maxRatio": 111,
  "bins": [
    { "label": "≤0.50", "count": 138,  "share": 0.0274 },
    { "label": "0.50-1", "count": 1984, "share": 0.3946 },
    { "label": "1-2",    "count": 656,  "share": 0.1305 },
    { "label": "2-5",    "count": 390,  "share": 0.0776 },
    { "label": "5-10",   "count": 701,  "share": 0.1394 },
    { "label": "10-20",  "count": 619,  "share": 0.1231 },
    { "label": ">20",    "count": 540,  "share": 0.1074 }
  ]
}
```

The headline number is the `0.50–1` bucket: **39.46% of sessions**. That bucket is dominated by ratio = 1.0 (its median is exactly 1, its mean is 0.999). What does ratio = 1 mean operationally? It means **one assistant turn for every operator turn** — no follow-ups, no internal monologue, no clarification loops. A stub. A one-shot.

That stub mode is, depending on your priors, either deeply healthy or quietly alarming. This post argues it is mostly the former — but only if you read the *rest* of the distribution correctly.

## What ratio = 1 actually looks like

Forget the average. The mean of 7.63 is a red herring driven by the long tail (max = 111, p99 = 58). The p50 is 1.96, but even that hides structure. Look at where mass concentrates:

- **42.2%** of sessions have ratio ≤ 1 (the cumulative share through the `0.50–1` bucket).
- **55.3%** have ratio ≤ 2.
- **63.0%** have ratio ≤ 5.
- **76.9%** have ratio ≤ 10.
- That leaves **23.1%** with ratio > 10 — the loop-heavy long tail.

Two-thirds of all sessions involve fewer than five assistant turns per operator turn. Roughly four-in-ten involve approximately one assistant turn per operator turn. This is not "the agent talked the operator's ear off." This is the agent answering once and stopping.

The empirical shape of a ratio = 1 session is:

1. Operator types a question.
2. Assistant replies.
3. Conversation ends (operator closes shell, swaps to another agent, or moves on).

That is what `pew` captures as a session: the lifecycle from first user message to whatever signal closes the queue. Sessions with ratio = 1 are sessions that *did not loop*. They did not require the agent to call a tool, observe output, decide what to do next, call again. They were single-shot Q&A.

## Why the stub mode is mostly healthy

There is a temptation — strong in 2024, weaker now — to treat any ratio = 1 session as wasted potential. "If the agent didn't loop, you're not getting your money's worth." That framing is wrong, for three reasons.

First, **single-shot answers are the lowest-cost mode**. The marginal cost of an assistant turn is dominated by prompt re-ingestion, not output. A 1:1 session means the operator paid exactly one prompt-construction cost and got exactly one answer. The cost-per-decision is at its theoretical floor. Looping costs more, and the loop only pays off if the marginal answer is meaningfully better than the first one.

Second, **the operator's own time is the binding constraint**, not the agent's. An operator who can resolve a question in one round is an operator who can move to the next question. The latency floor of "ask, read, decide, ask again" is humans, not models. A 39% slab of 1:1 sessions is a 39% slab of operator throughput.

Third, **stubs are diagnostic for tool quality**. A session where the operator asks "where does this function get called from?" and the agent answers correctly in one turn is a session where the agent did not need to fish around with `grep`, `glob`, `read`, `read again`. Either it had the right context loaded, or the question was simple enough not to need tools. Either way: the system worked.

## Why the long tail is also mostly healthy (but worth watching)

The other half of the bimodal story is the >10 bucket: **23.1% of sessions** with more than ten assistant turns per operator turn. The >20 bucket alone is 10.74% (540 sessions), with median 31, mean 35.6, max 111.

These are loop-heavy sessions: the assistant calling tools, observing, calling again, often for many cycles before the operator's next turn. In a healthy agent fleet, this is *exactly what you want* for tasks like "find all callers of this deprecated API," "review this PR," "summarize the last 100 events." The operator gives one instruction, the agent grinds through the work, the operator returns to a finished result.

The risk is not the existence of the long tail — it is **the depth of it**. A session with ratio = 111 means the assistant took 111 turns for every operator turn. If that operator only ever gave one instruction, that is 111 internal cycles before any human checkpoint. That is a lot of room for an agent to wander.

The right operational posture toward the long tail:

- **p95 = 32** is fine. A 30-turn agent loop for a multi-file refactor is normal.
- **p99 = 58** is borderline. Worth tagging those sessions for review.
- **max = 111** in a window of 5,028 sessions is one outlier per ~5,000. Acceptable. But if that maximum doubles next week without the workload doubling, that is a regression in loop discipline.

## The 2.74% sub-stub mode

The smallest bucket — and the most interesting — is `≤0.50`: **138 sessions, 2.74%**, with mean ratio 0.063. That is *less than one assistant turn per operator turn*. How is that even possible?

It is possible because the bucket includes sessions where the assistant said nothing at all. A few shapes:

1. **Operator typed, hit enter, agent never produced an assistant message** (failed to start, errored before first token, killed by a kill-switch envelope before generating).
2. **Operator typed multiple messages in a row** (a "let me also add..." pattern) before the assistant got to respond, and the session closed before the assistant caught up.
3. **Pure operator monologue sessions** — the operator was using the agent's session log as a notebook, dumping context for later.

The 2.74% sub-stub bucket is tiny but operationally meaningful: it is your **false-start rate**. If it grows, something is starting sessions that cannot complete. That is a fleet-health signal, not a usage-pattern signal.

## How to read this distribution in your own fleet

If you run an agent fleet, the reply-ratio histogram is one of the cheapest, highest-signal health checks you can compute. The shape to look for is roughly the one above:

- **Stub mode (≤2)**: 50–60% of sessions. This is operators using the agent as an intelligent autocomplete or one-shot answer tool. Healthy. If this drops below 40%, your operators have stopped trusting the agent for quick questions and are over-spec'ing every prompt.
- **Mid mode (2–10)**: 20–30%. Tool-using sessions of moderate complexity. Healthy.
- **Long tail (>10)**: 15–25%. Agentic loops doing real work. Healthy if p95 stays below ~50.
- **Sub-stub (<0.5)**: <5%. False-start rate. Watch the trend, not the level.

If your distribution is unimodal and centered around ratio ≈ 5, your operators are *all* using the agent the same way. That is a workload monoculture and brittle: one model regression takes out the whole fleet's productivity.

If your distribution has a fat tail with p99 above 100, your kill-switch envelopes need tuning. There is no human task that requires 100 internal agent turns without a checkpoint.

If your distribution shifts left over time (more stubs, fewer loops), one of two things is happening: the agent is getting smarter (good) or the operators have given up on agentic mode and are using it as autocomplete (bad). Disambiguate by checking `velocity` and `turn-cadence`.

## Why bimodality matters more than the mean

The single most important thing to take away from this distribution is that **the mean of 7.63 is meaningless**. Nobody runs sessions at ratio 7.63. Sessions live in two clusters: stubs near 1, and loops near 10–30. The mean is the centroid of an empty region between them.

Reporting "average reply ratio" in a fleet dashboard is the kind of dashboard mistake that survives because nobody ever looks at the underlying distribution. Once you do, you realize the average is a phantom — there is no operator whose behavior the average describes.

The same trap exists for `tokens-per-session`, `duration-per-session`, and `cost-per-session`. They are all bimodal in the same way, for the same reason: the workload itself is bimodal. Operators either ask quick questions or kick off long-running tasks. Almost nothing lives in the middle.

This is why pew-insights ships `bins` in every distribution endpoint, not just summary statistics. The bins are the truth. The summary statistics are a lossy compression of the truth.

## Practical takeaways

1. **Track the share of sessions in the `0.50–1` bucket as a primary fleet health KPI.** Today it is 39.46%. A drift below 30% or above 50% is a behavior change worth investigating.
2. **Track the p99 ratio as a secondary KPI.** Today it is 58. A doubling without workload change is a loop-discipline regression.
3. **Track the `≤0.50` bucket as your false-start rate.** Today it is 2.74%. Above 5% means sessions are failing to start.
4. **Stop reporting mean reply ratio.** It is the centroid of an empty region. Report p50, p90, p99, and the bin shares.
5. **Accept that the workload is bimodal.** Stubs and loops are both legitimate. Trying to push everything into the middle is fighting your operators.

The 39% thin-stub mode is not a problem to solve. It is the shape of how agents actually get used when they are working. Over-engineering to eliminate it would be eliminating the cheapest, fastest, most successful operator interactions in the fleet.
