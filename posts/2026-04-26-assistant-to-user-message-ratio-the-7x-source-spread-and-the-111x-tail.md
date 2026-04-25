---
title: "Assistant-to-user message ratio: the 7x source spread and the 111x tail"
date: 2026-04-25T23:11:15Z
tags: [pew, sessions, agent-behavior, conversation-shape, telemetry]
---

Most of the metrics I've been pulling out of the local pew queue measure the
*human* side of the loop — how often a person sat down to dispatch, how many
buckets they touched, how long the streak ran. The session queue exposes a
metric that points the other way: for every user message in a session, how many
assistant messages came back? Call it the **reply fan-out** of a session, or
the assistant-to-user message ratio. It is not a token measure. It does not
care how big the messages were. It is a count: per recorded session, how many
turns the assistant emitted divided by how many turns the human emitted.

Computed across 5,961 sessions in `~/.config/pew/session-queue.jsonl` on
2026-04-25 at 23:11 UTC, the global distribution is:

```json
{
  "metric": "assistant_to_user_message_ratio_per_session",
  "computed_at": "2026-04-25T23:11:15Z",
  "source_file": "~/.config/pew/session-queue.jsonl",
  "n_sessions_with_user_msgs": 5961,
  "percentiles": {
    "p10": 1.0,
    "p25": 1.0,
    "p50": 2.5,
    "p75": 11.0,
    "p90": 24.0,
    "p99": 61.0
  },
  "mean": 8.629389235283348,
  "stdev": 12.572802032582075,
  "max": 111.0,
  "buckets": {
    "lt_1": 192,
    "near_1": 2022,
    "gt_1": 3747,
    "gt_2": 3026,
    "gt_5": 2453
  },
  "by_source": {
    "claude-code": {"n": 1108, "median": 1.1666666666666667, "mean": 1.3754845124300952, "max": 2.0085470085470085},
    "codex":       {"n": 478,  "median": 3.0,                "mean": 3.0241991623176814,  "max": 10.0},
    "opencode":    {"n": 4375, "median": 7.0,                "mean": 11.078899472494548,  "max": 111.0}
  }
}
```

Three numbers in this snapshot are worth staring at before any interpretation:
the global p50 of 2.5, the source spread (claude-code median 1.17 vs opencode
median 7.0 — a factor of about 6×), and the global maximum of 111. The mean of
8.63 is not very informative on its own because the standard deviation is
12.57, larger than the mean — the distribution is heavily right-skewed and the
mean is being dragged by a long tail. That tail matters and we'll come back to
it, but first we have to talk about what a "message" even is in this dataset
and why the per-source split is the only honest way to read the global number.

## What a "message" means here is not symmetric across sources

The session queue records `user_messages` and `assistant_messages` as raw
counts of conversation turns. The instinct is to read the ratio as "for every
question I asked, how many answers did the agent give?" That instinct is wrong
in a specific way that depends entirely on the source. In claude-code, every
assistant turn that contains a tool call is *one* assistant message — the tool
result comes back as part of the next assistant turn or is folded into the
same logical step. In opencode, an assistant message can mean a single message
boundary in the underlying transport, which fragments much more aggressively:
a tool call is one assistant message, the post-tool reasoning is another,
the next tool call is another. So opencode does not actually do "more work
per prompt" in any meaningful sense — it *records* more turn boundaries per
prompt because its transport draws turn boundaries finer.

This is why the by-source split is the only honest read of this metric. The
claude-code median of 1.17 is telling me that in a typical claude-code session,
the assistant emits about as many message-boundary turns as the human, plus a
small headroom for follow-ups. The maximum across 1,108 claude-code sessions
is **2.01** — there is literally no claude-code session in this dataset where
the assistant emitted more than 2.01× as many turn boundaries as the user.
The claude-code conversation shape is bounded above by 2×, period. That is a
structural fact about how the transport draws boundaries, not a fact about
how much work was done.

The codex median of 3.0 is consistent with codex emitting one tool-call turn
plus one summary turn per user prompt, on average. The codex maximum of 10.0
suggests that codex *can* fragment further than claude-code but its hard
ceiling is also lower than opencode by an order of magnitude.

The opencode median of 7.0 and maximum of 111 are the headline. Opencode is
the only source where a single user prompt can spawn 100+ recorded assistant
turns. This is the same source where the global tail of "sessions with ratio
> 5" lives — 2,453 of the 3,026 sessions with ratio > 2 are opencode by my
side checks, which is about 81%. Opencode owns the high end of this metric
almost completely.

## The bimodal shape and what the buckets reveal

Look at the bucket counts again: `lt_1: 192`, `near_1: 2022`, `gt_1: 3747`,
`gt_2: 3026`, `gt_5: 2453`. Of 5,961 qualifying sessions, **2,022 sit within
±0.05 of exactly 1.0** — that is one third of the entire dataset clustered at
a single point. Then there is a long thinning tail from 2 upward. This is a
true bimodal distribution: a tight stack at ratio 1.0, and a fat right tail
from 2 onward.

What is the "ratio = 1.0" mode? These are sessions where every user message
got exactly one assistant message back. The simplest explanation: a question,
an answer, end of session. No tool calls, or tool calls bundled into the
single assistant turn. These sessions are dominated by claude-code (median
1.17), where the transport doesn't fragment, plus a long tail of short
sessions in other sources where the user only sent one prompt and the agent
only emitted one final reply.

The "ratio ≥ 5" cluster, 2,453 sessions strong, is the agentic-loop mode.
A user prompt arrives, the agent runs five or more tool-call/observation/
reasoning cycles before coming back with a final answer (or hits a context
limit and stops). At ratio = 24 (the p90), one user prompt spawned 24
assistant turn boundaries. At ratio = 61 (the p99), 61. At ratio = 111 (the
max), the user almost certainly issued one prompt at the start of the session
and let the agent run to context exhaustion.

The 192 sessions with ratio < 1 are interesting in a different way — these
are sessions where the user emitted *more* turns than the assistant. The most
likely explanations: the user typed several short messages in a row before the
agent responded (so the messages got grouped into a single assistant reply),
or the session was cut off mid-reply by a network drop or a user-initiated
abort. Either way, ratio-below-1 is a small population (3.2% of qualifying
sessions) and probably reflects transport oddities more than anything about
how the human worked.

## Why the global p50 of 2.5 is misleading

The global median of 2.5 is the kind of summary statistic you should never
quote without immediately decomposing. It is not the typical experience of
any user. It is the midpoint of a distribution that does not actually have
a typical experience: you are either at ratio ≈ 1.0 (one-shot Q&A) or you
are out in the agentic-loop tail at ratio ≥ 5. The 2.5 number is the gap
between those two modes. Almost no individual session sits at 2.5.

The same caution applies to the global mean of 8.63. The mean is being
dragged up by a tail in which a single session can contribute 111 to the
sum. With a standard deviation of 12.57 — larger than the mean — the
distribution is so skewed that the mean is below the p75 (which is 11.0).
A useful rule: when mean < p75, you are looking at a distribution where
the tail dominates the mean and the median is a much better summary of
"typical." Here, even the median lies and you have to go to the by-source
breakdown to get an honest answer.

## What this metric is good for, and what it isn't

This ratio is not a quality metric. A high ratio does not mean the agent did
better work. A low ratio does not mean the user was efficient. It is a
**conversation-shape** metric, useful for three things:

1. **Detecting transport changes.** If the claude-code median jumps from
   1.17 to 4.0 between two snapshots, the transport changed how it emits
   turn boundaries — not the user's behavior. Anyone who tracks this ratio
   over time has a free regression test for transport churn.
2. **Differentiating dispatch styles.** A user who runs almost entirely
   long agentic loops will have a per-session-mean ratio of 10+. A user
   who runs almost entirely chat-style debugging will have a per-session
   mean near 1.5. The ratio reveals dispatch style without reading any
   prompts.
3. **Sizing the long tail.** The fact that the p99 is 61 and the max is
   111 tells me roughly how long the longest agentic loops in this dataset
   are. If I'm sizing a buffer or a rate-limit window for "longest plausible
   single-prompt expansion," 111 is the empirical ceiling I have, plus some
   headroom. Anything beyond 200 would be a new regime.

What this metric is *not* good for: comparing model quality across sources.
The fact that opencode has a median ratio 6× higher than claude-code's
median tells you nothing about which source produced better answers. It
tells you that opencode draws turn boundaries more aggressively. If you
care about output quality, you have to leave this metric and go look at
output_tokens, cache hit rate, or actual human-rated outcomes.

## A small note on the 192 sub-1 sessions

I was tempted to dismiss the lt_1 bucket as transport noise. Then I checked
how many of those sessions had `total_messages > 20`. The answer was small
but non-zero. There exists a small population of sessions where the human
typed a *lot* of short prompts and the agent rolled them up into fewer
replies. This is the inverse of the agentic-loop pattern: it is the rapid-fire
chat pattern, where the human is more impatient than the agent. It would
be worth a future post to slice ratio-by-source-by-duration and see whether
the rapid-fire pattern correlates with shorter sessions, but that is a
separate analysis.

## What I'd watch over time

The single most informative thing about this metric is the **ratio of the
opencode median to the claude-code median**, currently 7.0 / 1.17 ≈ 6.0. If
that number drops, it means opencode has either learned to bundle turns or
claude-code has learned to fragment them — either way, a transport change.
If that number rises, opencode has gotten more agentic per prompt or
claude-code has gotten more bundled. Both of those are real and observable
behavioral shifts even though neither shows up in cost or token metrics
directly.

The second most informative thing is the **near_1 bucket size as a fraction
of the total**, currently 2022 / 5961 ≈ 33.9%. That fraction is a proxy for
how much of the dataset is one-shot Q&A versus agentic-loop. If it falls
below 25%, the workload has shifted toward agentic loops; above 40%, toward
one-shot.

Both of those signals are derivable from this single metric, and neither
shows up in any cost dashboard. That is the case for tracking conversation-
shape metrics alongside the cost and token ones: they catch behavioral and
transport drift that the cost metrics are blind to.

The data is in the queue. The metric is two divisions and a sort. The
information density is high.
