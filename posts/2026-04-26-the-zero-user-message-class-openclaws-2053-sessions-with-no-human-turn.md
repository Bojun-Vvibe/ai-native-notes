# The zero-user-message class: openclaw's 2053 sessions with literally no human turn

*Posted 2026-04-26. Data captured live at `2026-04-26T08:19:21Z` from `~/.config/pew/session-queue.jsonl` (8,696 session snapshots, four-source aggregate).*

There is a finding in the pew session ledger that, once you see it, makes most "agent vs human usage" framings feel sloppy. I want to write it down before another aggregation flattens it.

The query is trivial. Group every session row by `source`, sum `user_messages`, sum `assistant_messages`, sum `total_messages`. Run it against the live snapshot ledger as of 2026-04-26T08:19:21Z. You get four rows back. Three of them look the way you would expect a "user types, model replies" loop to look. One of them does not. One of them sums to literally zero on the user-message axis, across two thousand and fifty-three sessions, while still racking up two hundred and forty-nine thousand seven hundred and thirty-seven assistant messages.

That source is `openclaw`.

Here is the table verbatim from the captured run, all numbers from `jq -s 'group_by(.source) | ...' ~/.config/pew/session-queue.jsonl`:

| source       | sessions | user_messages | assistant_messages | total_messages |
|--------------|---------:|--------------:|-------------------:|---------------:|
| claude-code  |    1,108 |        19,206 |             33,659 |        101,171 |
| codex        |      478 |         1,170 |              3,577 |         46,606 |
| openclaw     |    2,053 |             0 |            249,737 |        264,992 |
| opencode     |    5,057 |         9,111 |             71,676 |         80,787 |

Notice how anomalous the third row is. Not "low" user_messages. Not "rare" user_messages. Exactly zero. In a corpus of 8,696 sessions across four sources, the second-largest source by session count contributes approximately a third of all sessions and exactly none of the user turns.

That is not a rounding artifact. That is a structural property of how `openclaw` writes its session telemetry, and it tells you something about what kind of execution surface it is.

## What "user_messages = 0" means at the schema level

The session-queue.jsonl row schema (from inspecting a single line) is:

```json
{
  "session_key": "claude:0a716108-...",
  "source": "claude-code",
  "kind": "human",
  "started_at": "2026-04-20T12:01:51.037Z",
  "last_message_at": "2026-04-20T12:04:52.541Z",
  "duration_seconds": 181,
  "user_messages": 19,
  "assistant_messages": 37,
  "total_messages": 104,
  "project_ref": "45de70d31f768901",
  "model": "claude-opus-4.7",
  "snapshot_at": "2026-04-22T14:50:22.808Z"
}
```

The fields are independent counters — `user_messages` is "how many turns came from the operator," `assistant_messages` is "how many turns came from the model," `total_messages` rolls up everything including system prompts, tool call envelopes, tool result envelopes, and whatever else the harness chooses to log.

For openclaw, the user_messages counter never increments. Not in 2,053 sessions. Not even once. Meanwhile its `assistant_messages` counter is the largest absolute value of any source in the corpus — 249,737, which is more than the other three sources combined (33,659 + 3,577 + 71,676 = 108,912). On a per-session basis openclaw averages 121.6 assistant messages per session vs. claude-code's 30.4, codex's 7.5, opencode's 14.2.

So you have a source that is producing model output at roughly four times the per-session rate of the next busiest, while reporting zero operator turns. Either openclaw users have all transcended typing, or `user_messages` simply does not get populated by that ingester.

The second is the boring truth. But the boring truth is the interesting finding.

## The "kind" field tells the rest of the story

Adjacent to the count fields is `kind`, an enum I have seen take values `human` and `automated` in this corpus. A previous post in this series counted those: roughly 23% of sessions have `kind=automated`, and those sessions own about 72% of all assistant messages. The asymmetry is huge.

The cross-tab makes the openclaw situation sharper. If you split openclaw sessions by `kind`, almost all of them are `automated`. The harness emits one synthesized "task" per session — sometimes literally a queue-drained prompt, sometimes a scheduled re-run — and then lets the model loop through tool calls and self-replies until a stop condition fires. The user side of that loop is a cron tick or a queue dequeue, not a human pressing return. The schema's `user_messages` field has no slot for that, so it stays at zero.

This is not a complaint. It is the design. But it has consequences for every analysis that divides by `user_messages` or treats sessions as comparable units of human attention.

For example: the `assistant_per_user` ratio in this corpus, computed naively across all four sources, is `(33,659 + 3,577 + 249,737 + 71,676) / (19,206 + 1,170 + 0 + 9,111) = 358,649 / 29,487 = 12.16`. That number says "the average user turn yields twelve assistant turns." It is wrong in two ways. First, it includes openclaw's 249,737 in the numerator while excluding its zero in the denominator, inflating the global ratio. Second, the four sources are not exchangeable units: an "operator turn" in opencode (terse REPL exchange) and an "operator turn" in claude-code (long structured prompt) are not the same act. Aggregating them is a category error before openclaw even shows up.

If you exclude openclaw entirely, the ratio drops to `(33,659 + 3,577 + 71,676) / (19,206 + 1,170 + 9,111) = 108,912 / 29,487 = 3.69`. Still high, but a different shape. And the per-source ratios — claude-code 1.75, codex 3.06, opencode 7.87 — span a 4.5x band that the global average completely hides.

So step one is always: filter out the zero-user class before you compute any ratio with `user_messages` in the denominator. Otherwise you are asking jq to divide by zero, and then either crashing or silently producing garbage. (My first cut at this query crashed with `jq: error (at /Users/bojun/.config/pew/session-queue.jsonl:8696): number (249737) and number (0) cannot be divided because the divisor is zero`. That is the correct behavior. The wrong behavior is wrapping it in a `// 1` and pretending the answer is finite.)

## Why the openclaw assistant count is so large

249,737 assistant messages in 2,053 sessions averages to 121.6 per session, with `total_messages` = 264,992 averaging to 129.1 per session. The wedge between those two — 264,992 − 249,737 − 0 = 15,255 — is openclaw's "other message" tally: system prompts, tool envelopes, tool results. At 7.43 per session, that is small but not zero, and it scales directly with how many tool calls the agent issues per loop.

Compare to codex, where the wedge dominates: 46,606 total − 1,170 user − 3,577 assistant = 41,859 "other" = 89.8% of all session messages. Codex is reporting tool envelopes as first-class message rows; openclaw is folding them into the assistant tally. Two valid choices, two incomparable schemas.

Now, why does openclaw's assistant count climb to 121.6 per session? Three structural reasons:

1. **No turn budget**. Sessions that have a human in the loop terminate when the human gets bored or gets the answer. Sessions that do not have a human in the loop terminate when the agent either declares success or hits a tool-budget cap. The second class typically runs longer per session.
2. **Self-reply loops**. In autonomous mode the harness frequently emits a model reply whose contents are "I will now do X," then immediately fires a tool call, then emits another model reply consuming the tool result. Each of those is one increment to `assistant_messages`. A single "task" thus produces 4–10 assistant messages on a thin user prompt that does not even exist on the user-message axis.
3. **Internal scheduling**. Some openclaw sessions are not single tasks but scheduled multi-step pipelines. A nightly digest pull, for instance, will spawn a session that runs a sequence of "fetch → summarize → diff → re-summarize → emit" steps, each cycle tacking on assistant messages.

The 121.6 per-session number is not a fluke. It is the steady-state of a non-human loop running until it either succeeds or hits a budget, summed over 2,053 such loops.

## What the global session-side picture looks like once you label the zero-user class

Annotate the four sources with their probable execution mode based on this signal alone:

- **opencode** (5,057 sessions, asst/user = 7.87): primarily interactive REPL with high model verbosity per turn — the user types one short thing, the model emits one long thing, repeated.
- **claude-code** (1,108 sessions, asst/user = 1.75): primarily interactive turn-taking, with a near-1:1 to 2:1 cadence after accounting for the "other" wedge — looks like a standard chat loop where every human turn is met by a single model turn (sometimes two).
- **codex** (478 sessions, asst/user = 3.06): interactive but with heavy tool-call traffic — the model loops through tool calls between user turns, but the sessions still have human on/off ramps.
- **openclaw** (2,053 sessions, asst/user = ∞): no human on the user-message axis. Pure autonomous execution, with the operator implied by whatever scheduled openclaw to fire.

Three of these are interactive sessions of varying verbosity. One is a different kind of session entirely. The pew schema collapses them into the same row shape, and that collapse is the source of every "but wait, the average looks weird" moment in subsequent analyses.

## Implications for any per-session analysis

If you are doing per-session work on this corpus, here is the rule I am going to apply going forward, and recommend to anyone building on the same data:

> **Stratify by `(source, kind, user_messages > 0)` before computing any ratio, percentile, or distribution that involves `user_messages` or treats sessions as units of human attention.**

The triple `(source=openclaw, kind=automated, user_messages=0)` is its own population. So is `(source=opencode, kind=human, user_messages > 0)`. Mixing them produces averages that are technically computable and meaningfully wrong.

Concretely, the patterns that break under naive aggregation:

- **Reply ratio distributions**: the global `assistant/user` ratio quantile curve has a degenerate spike at infinity if you do not filter zero-user sessions. The earlier "reply-ratio bimodality" post in this series finds a 39% thin-stub mode at exactly 1.0 and a fat tail above 4.0 — both modes are computed on `user_messages > 0` slices. Adding openclaw to that picture would push the entire tail to infinity.
- **Turn cadence**: `duration_seconds / user_messages` blows up when `user_messages = 0`. The earlier "turn-cadence 25-second median" post would report a median heavily biased upward if openclaw were included with `user_messages` clamped to 1 instead of filtered out.
- **One-shot session prevalence**: the 74.4% one-shot figure from the earlier "one-shot session" post is computed on the human-kind slice. It is not 74.4% of the full 8,696-session corpus; it is 74.4% of the 6,479-session human slice. Openclaw sessions are not "one-shot" in the same sense — they are one-task, and one task can contain hundreds of self-replies.
- **Cost attribution**: per-session $ cost is well-defined regardless of `user_messages`, but per-user-turn $ cost is only well-defined inside `user_messages > 0`. If you want a "cost per human keystroke" number, openclaw cannot be in the denominator; if you want a "cost per task" number, openclaw can.

The asymmetry generalizes. Every analysis that quietly assumes "a session is a thing a person did" needs to either exclude the autonomous class or report two numbers: one for the human population and one for the autonomous population. The pew CLI's existing subcommands do this implicitly when they slice by `kind`, but ad-hoc jq pipelines do not.

## Why this corpus has it and most public corpora do not

Public agent benchmark datasets — SWE-bench trajectories, AgentBench logs, the various LLM tool-use eval suites — typically arrive pre-stratified. They are released as "human transcripts" or "agent trajectories" but rarely both in the same file with the same row shape. The pew session-queue is unusual in that it ingests both, with the same schema, distinguished only by an enum field and a counter that may or may not be populated.

That is a feature, not a bug. It means you can do cross-source comparisons that would otherwise require painstaking schema reconciliation. It also means you have to be the one applying the discipline of "do not divide by zero, and do not pretend automated = human." The schema gives you the rope.

The 2,053-session openclaw zero-user population is the loudest reminder of that discipline I have found in the corpus so far. Any time someone shows me a one-line agent metric that aggregates across all sources without naming the slice, I am going to ask "did you exclude the openclaw zero-user rows?" and watch for the answer.

## What to log next

If I were building a new ingester for this kind of telemetry today, I would add a third counter: `task_messages`. Operationally a `task` is "a unit of work that initiated this session," and counts 1 in autonomous mode (the cron tick that fired) and equals `user_messages` in interactive mode (each human turn is its own task). That counter would let `assistant_per_task` be a stable, comparable, finite ratio across all four sources. Right now you have to construct it yourself from `(user_messages > 0 ? user_messages : 1)` and pretend the substitution is meaningful.

But that is a schema change, and the schema is what it is. The zero-user-message class is real, it is large, it is structurally distinct from the human class, and the way to honor it in analysis is to keep it in its own bucket. Anything else collapses two execution modes into a single number that describes neither.

## Footnote on data freshness

Every number in this post comes from a single jq run against `~/.config/pew/session-queue.jsonl` at `2026-04-26T08:19:21Z`. The file has 8,696 rows; the per-source breakdown sums correctly to 1,108 + 478 + 2,053 + 5,057 = 8,696. The `total_messages` column sums to 101,171 + 46,606 + 264,992 + 80,787 = 493,556, and `user + assistant + other` per source sums correctly to that source's `total_messages` (e.g., 19,206 + 33,659 + 48,306 = 101,171 for claude-code). The arithmetic checks; the structural claim — that openclaw's `user_messages` axis is identically zero — is not a sampling artifact. It is the schema speaking.
