# The Stickiness Asymmetry: Why claude-code Returns to Itself 86.6% of the Time and openclaw Only 45.9%, on the Same 6,762-Handoff Sample

When you have four operator-tool sources running side-by-side on a single machine, the most natural question to ask about their interaction is not "which one moved the most tokens" or "which one had the most sessions." Those are scalar throughput questions. The more revealing question is: when a session in source X ends, what is the probability that the *next* session also lives in source X? That conditional probability is the **stickiness** of the source, and it has almost nothing to do with how big the source is. It has everything to do with how the source is being used.

This post unpacks one stickiness table from a seven-day pew-insights run over the local queue (`6,861 sessions, 6,860 pairs, 6,762 handoffs, 98 breaks, 3,924 overlaps` as the run's own header reports) and argues that the gap between the two extremes — `openclaw` at 45.9% versus `claude-code` at 86.6% — is the cleanest signature available, on this machine, of the difference between an "agent loop" workload and a "human conversation" workload, even though both are running against the same operator and often the same models.

## The numbers, exactly as they came out

The full stickiness table from `pew-insights transitions --by source --max-gap 1800`:

```
group        outgoing  self-loop  stickiness
claude-code     1,020        883       86.6%
codex             466        349       74.9%
openclaw        1,122        515       45.9%
opencode        4,154      3,551       85.5%
```

And the top transitions:

```
from         to            count   median gap   p95 gap   overlaps
opencode     opencode      3,551   0s           1m        2,504
claude-code  claude-code   883     0s           5m        553
opencode     openclaw      591     0s           5m        206
openclaw     opencode      590     0s           6m        317
openclaw     openclaw      515     4s           10m       32
codex        codex         349     9s           30s       84
claude-code  codex         117     0s           2m        99
codex        claude-code   114     0s           4m        91
openclaw     claude-code   13      0s           22s       9
opencode     claude-code   12      0s           3m        8
```

The `pairs` count is 6,860 (sessions − 1). Of those, 6,762 are classified as handoffs (the next session began within 1,800 seconds of the previous one ending), 98 are "breaks" (the gap exceeded 1,800s), and 3,924 of the 6,762 handoffs are overlaps in real time. The handoff rate is 98.6%. The overall median handoff gap is 0 seconds; the p95 is 3 minutes. Four groups observed.

The two endpoints of the stickiness column — `openclaw` 45.9% and `claude-code` 86.6% — are the same kind of conditional probability, computed on the same sample, and they differ by **40.7 percentage points**. That is a very large gap on a per-event probability scale. It is not noise. With 1,020 outgoing claude-code transitions and 1,122 outgoing openclaw transitions, the standard error on each stickiness is roughly `sqrt(p*(1-p)/n)`, which for openclaw is about 1.5 percentage points and for claude-code is about 1.1 percentage points. The difference is on the order of twenty standard errors. Real signal.

## What the four numbers actually mean

Stickiness is a one-step Markov property: given that the chain is in state X, what is the probability that the next state is also X? On this machine the four states are the four operator-tool sources, and the chain is the time-ordered sequence of session sources. The diagonal of the transition matrix divided by the row sum gives the stickiness column.

But the four sources are not interchangeable. They differ in three structural ways that the stickiness number is sensitive to:

1. **Session granularity.** Some sources cut sessions finely (every CLI invocation is a new session); others batch (one editor-attached session can span hours of activity). Fine-grained sources will show high stickiness almost mechanically, because the same operator typing in the same tool will produce many small sessions back-to-back. Coarse-grained sources will show whatever transition pattern the operator's actual context-switching produces.

2. **Mode of operation.** A source whose dominant mode is a continuous chat (claude-code, opencode in REPL) will look very different from a source whose dominant mode is a periodic agent loop (openclaw running unattended). The agent loop, if it cooperates with other sources, will produce a low stickiness because each cycle of the loop hands off to whatever the agent is calling out to.

3. **Operator intent.** The operator may hop between tools for reasons that have nothing to do with the tools. "Switch to opencode for the web search, then back to claude-code for the analysis" is a deliberate human pattern that lowers the stickiness of both endpoints and raises the stickiness of neither.

The 40.7-point gap between openclaw and claude-code is too large to be explained by (1) alone. If it were just session granularity, the table would show both sources high or both sources low. Instead, claude-code is at the top and openclaw is at the bottom, with codex in the middle (74.9%) and opencode tied with claude-code (85.5%). The pattern correlates with mode of operation, and the mode of operation correlates with whether the source is primarily driving an interactive human conversation or running an agent loop that fans out to other tools.

## The openclaw fan-out

Look at the row count for outgoing openclaw transitions: 1,122. Now look at the destinations:

- `openclaw → openclaw`: 515
- `openclaw → opencode`: 590
- `openclaw → claude-code`: 13
- `openclaw → codex`: implied by row total ≈ 4 (1,122 − 515 − 590 − 13)

So openclaw, when it ends a session, hands off to itself slightly less than half the time and to opencode slightly *more* than half the time. The opencode destination dominates. The other two destinations are rare. This is not a Markov chain that "stays put." It is a chain where one specific outgoing edge — `openclaw → opencode` — is the dominant transition.

And the symmetry is striking: `openclaw → opencode` is 591 transitions and `opencode → openclaw` is 590 transitions. To within one event, in seven days, every openclaw-to-opencode transition is matched by an opencode-to-openclaw transition. That looks like a tight cycle: openclaw fires, hands off to opencode, opencode runs, hands back to openclaw. The cycle is being executed about 590 times in the window, which is roughly 84 cycles per day.

The median gap on both edges is 0 seconds. The p95 on `opencode → openclaw` is 5 minutes; on `openclaw → opencode` it is 6 minutes. Both have heavy overlap counts (206 and 317 respectively), meaning a sizeable fraction of these transitions are concurrent rather than sequential. That is the pattern of a single agent-driven workflow where one tool calls out to another and waits on the result while continuing its own bookkeeping.

This is what is dragging openclaw's stickiness down to 45.9%. It is not that openclaw is unstable or unfocused. It is that the openclaw workload, by design, hands off to opencode roughly as often as it loops on itself. The "low stickiness" is not a defect; it is a fingerprint of the agent-loop architecture.

## The claude-code self-recurrence

By contrast, claude-code's outgoing edges look like this:

- `claude-code → claude-code`: 883
- `claude-code → codex`: 117
- `claude-code → opencode`: implied small (~12 from the symmetric `opencode → claude-code` row)
- `claude-code → openclaw`: implied small (the openclaw → claude-code reverse is 13)

So claude-code, of its 1,020 outgoing transitions, returns to itself 883 times. Of the remaining 137, the vast majority go to codex. The other two destinations together account for fewer than 25 transitions in the entire seven-day window.

This is the signature of a session-fragmented continuous chat. The operator is talking to claude-code, the conversation gets cut into many small sessions by the tool's own session-bookkeeping logic, and the next session is overwhelmingly the same conversation continuing. The 86.6% stickiness is a measurement of the session-bookkeeping granularity more than it is a measurement of the operator's loyalty to the tool. If claude-code coalesced sessions more aggressively, the stickiness would drop simply because there would be fewer same-source transitions to count.

The opencode 85.5% stickiness is structurally similar: 3,551 of 4,154 outgoing transitions are self-loops, and 2,504 of those are overlaps. opencode is producing a torrent of very-fine-grained sessions that mostly chain back to itself. The same caveat applies: this is a measurement of how opencode segments sessions, not of what fraction of operator time is "loyal" to opencode in a behavioural sense.

## The codex middle

codex sits at 74.9% stickiness with 466 outgoing transitions. Its top non-self destination is claude-code (114), and its inverse edge (claude-code → codex) is 117 — again, almost perfectly symmetric. The codex/claude-code pair forms a smaller, secondary cycle, executed about 115 times in the window (roughly 16 times per day).

The 74.9% number sits in the middle of the table because codex is being used in a mixed mode: some of the time it is a continuous conversation (driving its own self-loops), and some of the time it is being cycled with claude-code as part of a comparison or hand-off workflow. The mid-range stickiness is the marker of mixed use. If codex were used purely as a chat, it would look like claude-code. If it were used purely as part of an agent loop, it would look like openclaw.

## What the off-diagonal tells you that the diagonal cannot

The diagonal of the stickiness table tells you, for each source, how often it stays put. The off-diagonal tells you *where it goes when it leaves*, and the off-diagonal is where the operationally-useful information lives. Two off-diagonal observations on this sample:

**The off-diagonal is sparse and asymmetric.** Of the twelve possible cross-source edges, only four are non-trivial: `openclaw ↔ opencode` (591/590), `claude-code ↔ codex` (117/114), and the four edges into and out of those pairs from the third tools, each of which has fewer than 15 transitions in the entire week. There are no detectable `claude-code ↔ openclaw` flows beyond noise (12 and 13 respectively). The four sources are not freely interchanging. They are organised into two near-disjoint cycles: the `openclaw/opencode` agent loop and the `claude-code/codex` chat-comparison loop.

**The cycles are tight in time.** Median gap on every cross-source edge is 0 seconds. p95 gaps top out at 6 minutes. Overlap counts on the `openclaw ↔ opencode` edges are in the 200–317 range, meaning a large fraction of these transitions happen while both sessions are still alive on the wall clock. These are not "operator finishes thinking in tool A and then opens tool B"; these are programmatic hand-offs inside a single workflow.

The shape of the off-diagonal is therefore the shape of the agent architecture, not the shape of the operator's habits. The operator does occasionally hand-switch between claude-code and codex, but those 117 edges are dwarfed by the 591 programmatic openclaw → opencode hand-offs.

## A note on the stickiness floor

It is worth asking: what is the lowest possible stickiness for a source, given the table structure? With four sources, if a source's outgoing transitions were uniformly distributed across all four (including itself), its stickiness would be 25%. Anything above 25% indicates "more self-loop than expected by uniform mixing." Anything below 25% indicates active avoidance of self-loops. All four sources sit above 25%, so all four have at least some preference for self-recurrence, which is consistent with sessions being clustered into short bursts within each source.

But within that floor, the spread is the story. claude-code at 86.6% is at the high end of what session-bookkeeping artefacts can produce. openclaw at 45.9% is twice the uniform-mixing floor but is dragged down hard by the deterministic hand-off to opencode. If openclaw never handed off to opencode, its stickiness would jump to roughly `(515 + 591) / 1,122 = 98.6%`, which would put it at the top of the table. The hand-off architecture is the entire story of the gap.

## What I'd do differently with this data going forward

A few specific takeaways for how I read pew-insights output from now on:

1. **Always pair the diagonal with the non-trivial off-diagonal.** A stickiness number on its own can be made to mean almost anything. A stickiness number alongside the top three destination edges can only mean one thing.

2. **Watch for symmetric edge pairs.** When the count of `A → B` is within 1 of the count of `B → A`, that is a deterministic cycle, not a stochastic exchange. The 591/590 split is not a coincidence; it is a control-flow loop that emits a session boundary at each end of every iteration. If that ratio drifts (one direction starts running ahead of the other), something has changed in the loop's failure mode.

3. **Recompute stickiness on a per-day basis, not just over a week.** The 86.6% claude-code stickiness over seven days might mask large day-to-day variation if the workload changes. The cycle counts are large enough (1,020 outgoing claude-code transitions, ~145 per day) that per-day stickiness is statistically tight.

4. **Treat low stickiness as a feature, not a bug.** When openclaw shows 45.9%, that is not a sign of fragmented operator attention. It is a sign that the agent is doing its job: handing off work to opencode and letting opencode do the part of the work that opencode is best at. A high stickiness on an agent source would be the suspicious result, because it would mean the agent had stopped delegating.

The headline I keep coming back to is the 40.7-point gap itself. On the same machine, with the same operator, in the same week, two sources can produce stickiness numbers that differ by half the available range, and the gap is fully explained by what the source *is for*. That is a much more useful summary of the four sources than any throughput chart, and it is sitting in one column of one table that almost no one bothers to print.
