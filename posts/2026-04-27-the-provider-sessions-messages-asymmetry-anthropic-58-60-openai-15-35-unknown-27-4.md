# The provider sessions/messages asymmetry: anthropic 58→60%, openai 15→35%, unknown 27→4%, and what the second axis actually measures

**As of 2026-04-27T03:16:09.169Z**, the `pew-insights provider-share` lens over my local `~/.config/pew/queue.jsonl` returned a table that, on first read, looks like three different stories happening on the same population:

```
provider   sessions  sess.share  messages  msg.share  models
anthropic  4,096     58.4%       145,440   60.3%      4
unknown    1,900     27.1%       10,211    4.2%       2
openai     1,018     14.5%       85,436    35.4%      5
```

Three providers, three completely different relationships between **how many sessions you start with them** and **how many messages those sessions carry**.

If you only looked at sessions, anthropic dominates at 58% and openai is a distant third at 15%. If you only looked at messages, anthropic still leads at 60% but openai is now a strong second at 35%, and the "unknown" category has collapsed from 27% to 4%. The session-leaderboard and the message-leaderboard agree on the winner, disagree about by how much, and disagree completely on third place.

This isn't noise. It's a real second axis — **messages-per-session** — and each provider is sitting at a structurally different point on it. Below is what each ratio means, why the ratios are so different, and what they tell you about how to read any "provider share" chart in the future.

## The three ratios

The cleanest way to see the asymmetry is to compute messages-per-session for each provider and compare to the corpus mean.

- Corpus mean: 241,087 / 7,014 = **34.4 messages/session**
- anthropic: 145,440 / 4,096 = **35.5 messages/session** (≈1.03× corpus mean)
- openai: 85,436 / 1,018 = **83.9 messages/session** (≈2.44× corpus mean)
- unknown: 10,211 / 1,900 = **5.4 messages/session** (≈0.16× corpus mean)

So **a single openai session is, on average, 15.5× longer than a single unknown session, and 2.4× longer than a single anthropic session.** That's a 15-fold spread on the same axis, in a corpus where most other distributional widths are 3× to 10× at most.

The session-share and message-share columns aren't telling two different stories — they're telling one story projected onto two axes whose ratio is itself the metric. Plot them on a 2-D plane (sessions on x, messages on y) and you get three points that fall on three different lines through the origin.

## Why anthropic sits at the corpus mean

Anthropic at 35.5 messages/session, against a corpus mean of 34.4, is essentially the same number. That makes sense: anthropic is 58% of the session count, so it dominates the corpus average — wherever anthropic sits, the corpus mean has to be pulled toward.

The interesting thing is that anthropic does **not** drag the average up or down hard. It sits right on it. That tells you the anthropic sessions in this corpus are basically "average sessions" by any conversational length metric — typical reply count, typical turn cadence, typical multi-turn tool-use patterns.

This is consistent with the earlier finding that `claude-opus-4.7` is the dominant model in the queue (3,643 anthropic sessions out of 4,096) and that its turn-cadence and message-volume distributions look like the corpus median rather than any extreme. **Anthropic is the baseline.** The other two providers are deviations from it.

## Why openai sits at 2.44× the mean (the long-session signature)

openai sessions average 83.9 messages — more than twice the corpus mean. With only 1,018 sessions producing 85,436 messages, openai is the **long-session provider**: fewer starts, deeper tails.

Three plausible structural reasons, and I think all three are simultaneously true:

1. **Reasoning-trace amplification.** The earlier reasoning-share analysis put codex (an openai-fronted harness) at 38.6% reasoning-token share. Reasoning tokens get emitted as a stream of `reasoning_output_tokens`, and depending on how the harness records snapshots, each reasoning chunk can land as its own message-shaped event. When a model that emits visible chain-of-thought writes 200 tokens of "thinking" before each 50-token answer, you get more discrete message units per turn than a model that doesn't expose reasoning at all. anthropic's `claude-opus-4.7` and `claude-sonnet-4.6` don't emit chain-of-thought as a separate message stream in the same way; openai's gpt-5.x family does.

2. **Tool-loop depth.** openai-fronted harnesses in this corpus are heavily codex-shaped — meaning long tool-using loops where the model proposes a command, the harness runs it, the model reads the output, proposes the next command, and so on. Each round-trip is two messages (assistant → tool, tool → assistant) and the loop runs deep before terminating. The 83.9 messages/session number is consistent with tool loops averaging in the 30–40 step range.

3. **Lower starting friction = more discrete sessions you'd otherwise call "one task."** This one cuts the other way. If openai sessions started cheaper (lower context-rebuild cost), I'd expect *more* sessions per task, not fewer messages per session. So this reason is a counterweight to the first two and helps explain why openai isn't 5× the mean — the message-per-session amplifier is partly offset by users starting fresh sessions for sub-tasks rather than threading them all into one.

The net is 2.44×, which sits about where you'd predict from "openai surfaces are codex-like long-tool-loop boxes that emit reasoning as separately-counted messages."

## Why "unknown" sits at 0.16× the mean (the daemon signature)

The "unknown" category at 5.4 messages/session is the one that surprised me on first read. It's 27% of sessions but only 4% of messages — by any naive read, it's a huge population that does almost no work.

Look at its top-3 models, though, and the picture sharpens:

```
unknown    unknown      1,266
           acp-runtime   634
           unknown_2       0
```

`acp-runtime` is a useful tell: it's the agent-control-protocol runtime, which is what mediates between a tool-using agent and the model behind it. Sessions tagged as `acp-runtime` are typically very short — they exist to broker a handoff or a tool-call response, not to host a long conversation. A typical acp-runtime session is something like 3–10 messages: open the channel, exchange a few protocol-level frames, close. That's exactly where 5.4 messages/session lands.

Sessions tagged just `unknown` are similarly short on average — these are rows where the model field never resolved (provider routing was ambiguous, or the snapshot got written before the model was selected). Those rows tend to come from the early-life of a session, before it stabilizes into a real conversation.

So the "unknown" category is **not a 27%-of-the-corpus story**. It is a daemon/control-plane story masquerading as a conversational one. If you filtered by message count instead of session count, the unknown bucket would shrink to its real economic footprint of 4%. **Sessions are not the right unit for measuring unknown's importance — messages are.**

## The general lesson: pick your axis to match the question

This is the third or fourth time in the last few weeks I've run into a metric where the session-axis and the message-axis disagree by a factor that exceeds the spread on either axis alone. The pattern is consistent enough that it deserves a name:

- **Session-axis dominance** = "how often do I commit to provider X" → who I sign up with, who I keep credentials for, who I depend on for availability.
- **Message-axis dominance** = "how much actual work does provider X do for me" → who I depend on for compute, who pays my bills, who needs to be reliable per-call.
- **Token-axis dominance** = a third axis we haven't crossed today, but which sits one level deeper still — how much *content* moves through each provider, weighted by message size.

For anthropic the three axes mostly agree (58/60/likely-in-the-50s for tokens). For openai they diverge sharply: 14.5% sessions, 35.4% messages, and given that gpt-5.x messages are typically denser than acp-runtime frames, the token share is probably even higher. For unknown they diverge in the opposite direction.

**Reading any provider-share chart on the wrong axis is a category error.** "Anthropic does 58% of my work" is true on sessions, defensible on messages, and probably wrong on tokens. "openai does 15% of my work" is true on sessions and badly understates the message-level reality. Both numbers are real; neither alone is the answer.

## How this interacts with the single-model-session lock

Yesterday I wrote about the model-switching lens reporting `switched_share: 0.0%` — within a session, the model never changes. That result combined with this provider asymmetry gives a precise statement:

> **The session is the unit of provider commitment, but the cost of that commitment is heterogeneous by provider.**

When I commit a session to anthropic, I get a corpus-mean-shaped session: ~35 messages, balanced reply ratio, normal cache geometry. When I commit a session to openai, I am implicitly committing to a 2.44× longer session, on average — more messages, deeper tool loops, more reasoning-trace messages. When the broker tags a session as unknown, I'm committing to a 0.16× session, short and protocol-shaped.

The session is uniform within itself but the population of sessions per provider is structurally different. **The choice of provider at session-launch is therefore a much bigger commitment for openai sessions than for anthropic sessions** — once that 84-message session starts, it carries on for 84 messages. You cannot exit halfway and go to anthropic, because that would be a switch and switches don't happen (the previous post's 0.0% number).

This is, I think, the answer to "why don't I see more 'switch to anthropic when openai gets stuck' patterns in production agents." The answer is: by the time you'd want to switch, you've already paid most of the session cost, and the cost is paid in messages, not in tokens. Switching only saves you the *remaining* messages, which on an average openai session is whatever fraction of 84 you haven't yet emitted. On an average anthropic session, you'd be saving even less, because you're already most of the way through 35.

Switching is unattractive at every point on the cost curve. Which is why nobody bothers, which is why `switched_share: 0.0%` is what we observe.

## What changes if these ratios shift

The three ratios are stable enough right now that they read like permanent constants. They are not. Three things would move them measurably:

1. **A new harness that emits anthropic reasoning as separate messages.** If anthropic ships visible chain-of-thought and one of my surfaces starts logging it as discrete message events, anthropic's messages-per-session will drift up from 35.5 toward openai's 83.9. The session count won't move; the message count will. This would be a very specific signature: anthropic msg-share rising while sess-share holds.

2. **An openai harness that batches tool calls into one message instead of two.** If a future codex variant collapses tool-call-and-response into a single emitted message, openai's messages-per-session will drop sharply, possibly halving from 83.9 to ~42. session count holds; message count drops. Again, a specific signature.

3. **A change in how `unknown` sessions get tagged.** If every snapshot eventually resolves to a concrete model, the unknown bucket disappears — those 1,900 sessions get re-attributed to their real provider, mostly anthropic and openai based on the model field. Anthropic sess-share would climb from 58% toward 70%; openai from 15% toward 22%; the messages totals barely move because unknown is only 4% of messages. **The session leaderboard would shift dramatically; the message leaderboard would not.** That alone is a useful sanity check on which axis is reporting structural reality.

I'll know this happens because the dashboard tracks all three numbers. The asymmetry between the axes is itself a metric, and it's the kind of metric that drifts slowly and meaningfully — exactly the shape of monitor I want.

## Practical takeaways

1. **When someone shows you a "provider share" pie chart, ask which axis.** If they don't know, the answer is usually sessions, because sessions are easier to count. The session axis is the one that flatters anthropic and the unknown bucket, and disadvantages openai relative to its real footprint.

2. **For cost projections, use messages or tokens, not sessions.** Provider bills are token-priced. Sessions are a startup-cost proxy at best.

3. **For dependency analysis, use sessions.** If anthropic goes down, 58.4% of my session-starts fail; if openai goes down, only 14.5% fail. Sessions are the right unit for "how often does X have to be up for me to do my work."

4. **For per-call latency / reliability tuning, use messages.** A 1% error rate on openai (35.4% of messages) is operationally larger than a 1% error rate on anthropic (60.3% of messages) only when you weight by call volume — which messages do, sessions don't.

5. **Treat `unknown / acp-runtime` as a control-plane bucket, not a provider.** It's neither anthropic nor openai; it's the plumbing that connects them to my harness. Filter it out before computing anything that's supposed to be about model behaviour.

6. **The 15.5× messages-per-session spread between openai and unknown is the real second axis to monitor.** When it compresses, providers are converging on a common session shape — probably because protocols are stabilizing. When it widens, providers are diverging — probably because someone shipped a new reasoning surface or a new daemon. Both directions are interesting.

Three providers. Three completely different relationships between sessions and messages. Same population, same window, same `as of` timestamp.

That asymmetry is the whole story.
