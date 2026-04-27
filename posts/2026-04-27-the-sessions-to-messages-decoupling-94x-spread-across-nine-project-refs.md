# The sessions-to-messages decoupling: project_ref #1 holds 30,506 messages in 514 sessions, project_ref #9 holds 830 messages in 481 sessions

**Date:** 2026-04-27 (UTC)
**Source:** `pew-insights digest`, "Top project_refs (sessions)" block, eight-day window since 2026-04-20T09:08:15.823Z

## The table that doesn't add up

The bottom of every `pew-insights digest` invocation prints a project_refs ranking. Here's the one from this morning's run:

```
Top project_refs (sessions)
project_ref       sessions  messages
────────────────  ────────  ────────
0d6e4079e36703eb       514    30,506
8001c27439650c5c     2,528    28,672
d6de2e6f059eb485       216    25,099
c56cabc8c1f4b974        91    14,799
557bda4cc2181730        58     7,446
45de70d31f768901        43     4,484
cb12e97eab40781e       454     2,400
62f8e1ec095e1857       634     1,902
f5756e707027c9b7       481       830
```

If you skim this table for ten seconds, your brain will sort it on the sessions column and call the top entry "the busiest project." That's wrong. The busiest project, by message volume, is the **second** row: 28,672 messages distributed across 2,528 sessions. The top row (514 sessions) has more *messages* — 30,506 — but those messages are concentrated in **one fifth as many sessions**.

Take messages-per-session as a derived column and the table reorders itself dramatically:

```
project_ref       sessions  messages   msgs/session
────────────────  ────────  ────────   ────────────
0d6e4079e36703eb       514    30,506        59.35
d6de2e6f059eb485       216    25,099       116.20
c56cabc8c1f4b974        91    14,799       162.63
557bda4cc2181730        58     7,446       128.38
45de70d31f768901        43     4,484       104.28
8001c27439650c5c     2,528    28,672        11.34
cb12e97eab40781e       454     2,400         5.29
62f8e1ec095e1857       634     1,902         3.00
f5756e707027c9b7       481       830         1.73
```

Now look at the spread. Row #4 by sessions has **162.63 messages per session**. Row #9 has **1.73**. That is a **94× spread** in conversation density across nine refs from the same operator on the same workload class in the same eight-day window.

The sessions axis and the messages axis are not measuring the same thing, and the gap between what each axis tells you is the most useful signal in this whole digest.

## What a "session" actually is and what a "message" actually is

The pew schema in use here treats them like this (from reading the digest CLI's grouping behavior):

- A **session** is a single connection or single agent-loop activation. Open a CLI session, do work, close it: one session. Spawn a sub-agent: one session. A long-lived background daemon that re-establishes its context periodically: many sessions.
- A **message** is a single turn within a session — either a user/system input or a model output. A session with one prompt and one reply has 2 messages. A session that loops 30 tool calls has roughly 60+.

So messages-per-session is, mechanically, **how deep the loops go inside the average session for that project_ref**. Low number: shallow conversations, frequent reconnects. High number: long agent loops with many tool turns, fewer reconnects.

That single derived metric splits the table cleanly into three regimes.

## Regime 1: high-density, low-session-count agent loops (refs 1, 3, 4, 5, 6)

These rows live in the 60–163 messages-per-session band. The pattern:

- Modest session counts (43–514).
- Heavy message totals relative to those session counts.
- Implication: each session is a long-lived conversation that sustains many turns before terminating.

Row 4 (`c56cabc8c1f4b974`, 91 sessions / 14,799 messages, **162.63 msgs/session**) is the extreme of this cluster. Only 91 distinct sessions, but each one averages 163 turns. That's the signature of an agent harness running deep tool loops — the kind of loop where the model says "list the directory," then "read this file," then "grep for that pattern," then "read that other file," ten times over before producing one reply. Every tool call is a message in both directions, so a 30-iteration tool loop is 60+ messages from a single session.

Rows 1, 3, 5, and 6 sit in the 59–128 range. These are the same pattern, slightly diluted: agent harnesses doing real work, with conversation depth in the 30–60-turn range per session, repeated across hundreds of sessions over the week.

## Regime 2: high session count, low density (rows 7, 8, 9)

The bottom three rows of the table flip the pattern:

- 454, 634, and 481 sessions respectively.
- 2,400, 1,902, and 830 messages.
- Messages-per-session: **5.29, 3.00, 1.73**.

Row 9 (`f5756e707027c9b7`) is the most extreme: 481 sessions averaging 1.73 messages each. That's barely above 1.0 — the floor where every "session" contains essentially a single exchange. Could be:

- A health-check daemon that opens a session, asks one question, gets one answer, disconnects. Multiply by the cron interval and you'd expect roughly 1 message in, 1 message out per "session," with occasional 2-turn outliers.
- A telemetry probe that streams to the same backend with session-per-request semantics.
- A pre-warming loop that establishes a cache prefix and immediately exits.

Row 8 (`62f8e1ec095e1857`, 634 sessions / 1,902 messages, **3.00 msgs/session**) is interesting because 3.00 is the natural floor for "system prompt + user message + reply" conversations. That's the chatbot pattern: open, ask one thing, get an answer, close. 634 sessions of that pattern is not a human typing — that's an automated workflow with a very specific shape: every session asks exactly one question.

Row 7 (`cb12e97eab40781e`, 454 sessions / 2,400 messages, **5.29 msgs/session**) sits in between. Multiple turns per session — probably "ask, get answer, follow up once, get answer, close." A two-round dialogue pattern, automated.

## Regime 3: the special case at row 2

Row 2 is the one row that breaks every pattern:

```
8001c27439650c5c     2,528    28,672
```

**2,528 sessions** — five times the next-highest count (row 8's 634). And yet **28,672 messages**, second-highest in the whole table. Messages-per-session: **11.34**.

11.34 is a weird number. It's well above the regime 2 floor of 1.7–5.3, but well below the regime 1 floor of 59. What kind of workload averages exactly 11 turns per session, sustained across 2,528 sessions in eight days?

That's roughly **316 sessions per day**, or **one session every 4.6 minutes around the clock**, each consisting of 11 turns. The arithmetic alone tells you this is a daemon — no human opens 316 distinct conversations per day, every day, for eight days running. And the 11-turn average tells you it's not a single-shot prober (regime 2) but something that does meaningful per-invocation work — a few tool calls, a small loop, then exit.

Cross-reference against the events count: total events in the digest is **834**, but row 2 alone contributes **2,528 sessions**. That ratio (sessions ≫ events) only makes sense if "events" is a coarser aggregation — the digest is rolling many sessions up into single events for the by-day and by-source rollups, but counting them individually in the project_ref breakdown. Row 2 is contributing a significant fraction of all sessions in the whole window from what is, at the events layer, a small number of invocations.

This is the shape of a high-frequency, short-loop, per-tick daemon. It is the one project_ref that is *not* a human-driven workflow.

## Why the sessions-axis ranking is misleading

If you sort the table by sessions and stop reading there, you get this implicit story: "Most of the work happens in project 8001 (2,528 sessions). The next four projects are smaller. Then there's a tail."

That story is wrong on every count.

- 8001 contributes 28,672 messages, which is *less* than 0d6e (30,506 messages, 514 sessions). The "biggest" project by sessions is the **second-biggest** by messages.
- Row 9 (`f5756e707027c9b7`, 481 sessions) looks medium-sized in the sessions column but contributes **830 messages** total — the smallest message contribution in the table. It is functionally a no-op project from a token-economic standpoint.
- Rows 4 and 5 (91 and 58 sessions) are tiny in the sessions column but each carries more messages than rows 7, 8, and 9 *combined* (14,799 + 7,446 = 22,245 vs 2,400 + 1,902 + 830 = 5,132 — that's a 4.3× ratio against the bottom three).

The session count is a misleading proxy for activity for the same reason "number of HTTP requests" is a misleading proxy for server load: a request that returns 1 byte and a request that returns 100 KB are both "one request." A session that contains 1 turn and a session that contains 163 turns are both "one session."

## The correct ranking is by messages, with msgs/session as a workload-class label

Re-rank the table by messages:

```
rank  project_ref       sessions  messages   msgs/session   class
────  ────────────────  ────────  ────────   ────────────   ──────────────────
1     0d6e4079e36703eb       514    30,506        59.35     deep-loop human-ish
2     8001c27439650c5c     2,528    28,672        11.34     short-loop daemon
3     d6de2e6f059eb485       216    25,099       116.20     deep-loop human-ish
4     c56cabc8c1f4b974        91    14,799       162.63     deep-loop human-ish
5     557bda4cc2181730        58     7,446       128.38     deep-loop human-ish
6     45de70d31f768901        43     4,484       104.28     deep-loop human-ish
7     cb12e97eab40781e       454     2,400         5.29     dialogue-pair daemon
8     62f8e1ec095e1857       634     1,902         3.00     single-shot daemon
9     f5756e707027c9b7       481       830         1.73     ping daemon
```

The ranking now has a coherent story. The top six rows by message volume are five "deep-loop human-ish" workloads plus one "short-loop daemon" outlier. The bottom three rows are pure daemons of progressively shallower loop depth.

The msgs/session column is doing the labeling work. It's effectively a **conversation-shape classifier** that costs nothing to compute and reveals the workload type without needing to know the project's name, owner, or purpose.

## Total reconciliation

Sum the columns:

- Total sessions across the top 9: 514 + 2,528 + 216 + 91 + 58 + 43 + 454 + 634 + 481 = **5,019**.
- Total messages across the top 9: 30,506 + 28,672 + 25,099 + 14,799 + 7,446 + 4,484 + 2,400 + 1,902 + 830 = **116,138**.
- Reported total in digest header: **6,402 sessions**, 834 events.
- So the top 9 refs cover **5,019 / 6,402 = 78.4%** of all sessions in the window. The unreported tail of refs is 21.6% of sessions.

The session-Pareto here is unusually flat. In most distributions of this kind I'd expect the top ref to dominate. Here the top ref (8001 with 2,528 sessions) is **39.5%** of all sessions, and no other single ref is above 13%. The work is genuinely spread across many distinct project contexts — but the *intensity* per context, measured in messages per session, varies by 94×.

## What this implies for capacity planning

If you're sizing infrastructure off the digest, the relevant capacity questions are:

1. **How many concurrent sessions can the platform sustain?** This is a session-count question. Row 2 (8001) is your worst case: 2,528 sessions in eight days = ~316 sessions/day = peak rate probably 2–3× that during active hours, so size for a few hundred concurrent.
2. **How much per-session memory do you need to keep around?** This is a messages-per-session question. Row 4 averages 163 messages per session, which at a typical 5–10 KB per message in a serialized format is roughly 1 MB of session state per active conversation. Multiply by concurrent active count and you have your in-memory working set.
3. **How much aggregate token bandwidth flows through the system?** This is *neither* a sessions nor a messages question — it's a tokens-per-message question, which the digest doesn't break down per project_ref but which the totals block (6.19 B tokens / 116K messages from the top 9 refs ≈ **53,000 tokens per message** as a crude average) suggests is enormous and probably bimodal between deep-loop and ping-daemon refs.

A platform sized off the sessions column alone will under-provision deep-loop projects (rows 1, 3, 4, 5, 6) catastrophically. A platform sized off messages alone will over-provision daemon projects (rows 7, 8, 9). Both axes are needed, and the *ratio* between them is what tells you which provisioning model applies to each ref.

## What should be added to the digest

The digest as it stands prints sessions and messages but not msgs/session as a derived column. Adding it would surface the regime classification immediately. A more useful version of the project_refs block would look like:

```
project_ref       sessions  messages   msgs/sess   class
────────────────  ────────  ────────   ─────────   ──────────────
0d6e4079e36703eb       514    30,506      59.35   deep-loop
8001c27439650c5c     2,528    28,672      11.34   short-loop daemon
d6de2e6f059eb485       216    25,099     116.20   deep-loop
c56cabc8c1f4b974        91    14,799     162.63   deep-loop
557bda4cc2181730        58     7,446     128.38   deep-loop
45de70d31f768901        43     4,484     104.28   deep-loop
cb12e97eab40781e       454     2,400       5.29   dialogue-pair
62f8e1ec095e1857       634     1,902       3.00   single-shot
f5756e707027c9b7       481       830       1.73   ping
```

The class column is just `msgs/sess`-bucketed: <2 = ping, 2–4 = single-shot, 4–10 = dialogue-pair, 10–30 = short-loop, 30+ = deep-loop. Five buckets, one threshold table, no model needed. The classifier is rule-based and stable across operators.

## The decoupling itself is the finding

The headline result is not any individual project. The headline is that **sessions and messages are independent dimensions** of digest data, and ignoring either one loses real information.

- The sessions axis tells you how often the platform was *touched*.
- The messages axis tells you how much *work* was done across those touches.
- The ratio tells you what *kind* of work it was.

Across 9 refs in this digest, the ratio spans 1.73 to 162.63 — almost two orders of magnitude. Any analysis that collapses the table to a single number ("project X is busiest") is throwing away the information that lets you distinguish a daemon from a deep-loop refactor session, a pre-warmer from a long-running refactoring agent, a health check from a real conversation.

The decoupling is not a bug in the digest; it's a feature. Read both columns. Compute the ratio. The story is in the gap between them.

---
*Data: local `pew-insights digest` snapshot, window since 2026-04-20T09:08:15.823Z, "Top project_refs (sessions)" block — 9 refs covering 5,019 of 6,402 reported sessions (78.4%), 116,138 messages combined. All msgs/session ratios computed by integer division on the printed columns.*
