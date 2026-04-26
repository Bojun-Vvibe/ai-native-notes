# Session prompt-count distribution mode: the one-prompt spike, the zero-prompt shadow, and the four-source shape divergence

There is a tempting fiction that an AI-assisted coding session looks like a conversation. You open a tool, you send a few prompts, you get a few replies, and over the course of fifteen or twenty minutes you converge on something useful. Pictured this way, the natural distribution of prompts-per-session ought to look like a hump — perhaps a gamma curve with a mode somewhere around three or four user messages, a thin left tail of abandoned starts, and a long right tail of marathons.

The data does not look like that. Across **8,608 sessions** captured in the local session ledger as of 2026-04-26, the empirical distribution of `user_messages` per session is:

```
n=8608  mean=3.42  min=0  max=398
p10=0   p25=0   p50=1   p75=1   p90=2   p95=19   p99=62
```

The median is one. The 75th percentile is also one. The 90th percentile is two. Then between p90 and p95 the distribution detonates from 2 to 19, and between p95 and p99 it goes from 19 to 62, and the tail finishes at 398. This is not a hump. This is a spike at one, a shadow at zero, and a power-law tail. And once you decompose it by `source`, the shape is not even *one* shape — it is at least four distinct shapes living under the same field name.

This post pulls those four shapes apart and argues that the global histogram of `user_messages` is a meaningless number. Each source is generating a different process; mixing them produces the zero-and-one fortress that dominates the global view.

## The raw histogram, before any decomposition

The first 25 cells of the histogram, by integer value of `user_messages`, are:

```
value  count
    0   2485
    1   4913
    2    486
    3     48
    4     20
    5     15
    6     14
    7     10
    8      8
    9     12
   10     15
   11     12
   12     13
   13     17
   14     20
   15     19
   16     26
   17     23
   18     16
   19     23
   20     16
   21     26
   22     26
   23     14
   24     21
```

Read that as a histogram and you see something almost unbelievable: 7,398 of 8,608 sessions — 86.0 percent — sit in the {0, 1} pair. A further 5.6 percent are at value 2. Everything from value 3 upward, all the way to 398, is the remaining 8.4 percent, and inside that 8.4 percent the count is roughly flat from value 13 onward (15, 19, 26, 23, 16, 23, 16, 26, 26, 14, 21…) until the tail starts to thin.

A flat tail past a sharp mode is the signature of a mixture distribution. The flat region is *not* the same population as the spike at one. Something is generating long sessions at roughly equal rates per length bucket, and something else is generating one-prompt sessions in volume. They are not the same process.

## Decomposing by source

Cutting the same field by `source` produces this:

```
source            n     zero     one        p50  p90  p99   max   mean
openclaw         2034  100.0%   0.0%        0    0    0     0     0.00
opencode         4988    9.0%  88.1%        1    1   22    195    1.81
claude-code      1108    0.0%  47.0%        4   44  130    398   17.33
codex             478    0.0%   0.0%        2    2   11     69    2.45
```

(Sources that appeared in `queue.jsonl` but not as their own row in `session-queue.jsonl` — for instance the editor-side IDE assistant and the workflow proxy — do not contribute sessions in the same shape and are excluded here.)

Four sources, four entirely different processes. None of them looks like the global mixture. Each one tells you what its substrate is actually doing.

### openclaw: the zero-prompt shadow

`openclaw` posts 2,034 sessions and *every single one* has `user_messages = 0`. The mean is exactly zero, the max is zero, the p99 is zero. There is no distribution to speak of — it is a delta function at the origin.

What this means in practice is that `openclaw` is not actually emitting "user prompts" in any meaningful sense. It is emitting *session shells* — records where a session existed, presumably with assistant-side activity counted under `assistant_messages` and `total_messages`, but where no item in the transcript was tagged as user-typed. Those sessions are real (they have `started_at`, `last_message_at`, durations, models), but their contribution to the distribution of "human prompts per session" is zero by construction.

When you mix 2,034 zeros into the global histogram, you create the value-0 pillar. Of the 2,485 zero-prompt sessions globally, openclaw alone contributes 2,034. The remaining 451 zeros all come from one other source — opencode — which we will get to in a moment. Together they fully explain the value-0 cell.

This is the first reason the global histogram is a fiction: 28.9 percent of the "sessions" being averaged in are sessions that do not have a notion of a user prompt at all.

### opencode: the one-prompt monolith

`opencode` is the largest contributor to the session ledger by row count, with 4,988 of 8,608 sessions — 57.9 percent of the total. Its distribution is:

- 9.0 percent (n=451) at zero user messages,
- 88.1 percent (n=4,392) at exactly one user message,
- the remaining 2.9 percent (n=145) spread across values 2 through 195,
- median 1, p90 1, p99 22, max 195, mean 1.81.

Of those 4,392 one-prompt opencode sessions, this one contributor alone is responsible for *89.4 percent of every one-prompt session in the entire ledger* (4,392 of 4,913). Without opencode, the value-1 cell would shrink from 4,913 to 521 — the value-1 cell would no longer dominate.

So the global "median is 1" finding is, at root, an opencode finding. opencode runs in a mode where each session typically contains one user message and then the assistant takes over. The 88.1 percent one-prompt rate is not just high; it is structural. The substrate is built so that the user fires a single instruction and then the agent works without further interjection.

Crucially, opencode also has a long thin tail: a p99 of 22 and a max of 195. Three of those long sessions are themselves notable. Looking at the four longest opencode sessions in the ledger:

```
src       u_msgs  t_msgs  duration_s  started_at                last_msg_at
opencode  195     1031    261729      2026-04-21T08:23:29.828Z  2026-04-24T09:05:39.426Z
opencode  194     1025    261660      2026-04-21T08:23:29.828Z  2026-04-24T09:04:30.374Z
opencode  183      968    260715      2026-04-21T08:23:29.828Z  2026-04-24T08:48:45.705Z
```

These three sessions all started at the *same* timestamp — 2026-04-21T08:23:29.828Z — and ran for 261,729, 261,660, and 260,715 seconds respectively. That is a duration of roughly 72.7 hours per session. They are essentially the same session emitted three times with very slightly different terminal timestamps and slightly different counts (195, 194, 183 user messages). This is a snapshotting artifact — the session-queue is dumping interim states of the same long-running session as separate rows. The "max 195" data point in the opencode column is therefore not three independent 195-prompt sessions; it is one session being snapshotted at three different moments.

The implication for distribution-shape claims is real: opencode's right tail has duplicated mass. Even acknowledging that, opencode is overwhelmingly a one-prompt-per-session source, with a thin and partly-overcounted tail.

### claude-code: the only source with a real conversation distribution

If you want a source whose `user_messages` field looks like an actual gamma-shaped conversation distribution, claude-code is it:

- 0.0 percent at zero (no shells),
- 47.0 percent at one (n=521),
- median 4, p90 44, p99 130, max 398,
- mean 17.33.

That mean of 17.33 is roughly five times the global mean of 3.42. The p90 of 44 is twenty-two times the global p90 of 2. The p99 of 130 is more than double the global p99 of 62. claude-code is single-handedly producing the global tail.

The longest sessions in the entire ledger are claude-code sessions. The very top of the distribution:

- 398 user messages, 2,394 total messages, 23,063 seconds duration, 2026-04-03 02:53:41 UTC start to 09:18:05 UTC same day.
- 283 user messages, 1,683 total, 7,778 seconds, 2026-03-23 06:51:50 to 09:01:28 — a hair over two hours.
- 252 user messages, 1,129 total, **1,221,212 seconds** duration, started 2026-04-02 and last message 2026-04-16. (The 14-day "duration" almost certainly indicates a session that was left open across days; the 252 prompts likely cluster.)
- 239 user messages, 17,431 seconds, 2026-03-25.
- 234 user messages, 17,431 seconds, 2026-03-26.

The 398-prompt session is the headliner. 23,063 seconds is about six hours and twenty-four minutes; 398 prompts in that window is roughly one user message every 58 seconds, sustained for the full window. That is not "a conversation" in any normal sense; it is a fast back-and-forth driving a tool, the human side typing more or less continuously.

claude-code is also where 47 percent of the *one-prompt* sessions outside of opencode live (521 of the 521 non-opencode one-prompt sessions). So even though claude-code has the heaviest tail, almost half of its sessions are still one-shots. The substrate supports both.

### codex: the narrow-band source

codex is the smallest of the four with sessions, at n=478, and its shape is the strangest:

- 0.0 percent at zero,
- 0.0 percent at one,
- median 2, p90 2, p99 11, max 69, mean 2.45.

codex never produces a one-prompt session. Its modal value is 2 — both the median and the p90 are 2. So roughly speaking, the codex norm is "one user message, one follow-up, done." Whatever the substrate is doing internally, it counts user messages such that the minimum sensible session contains two of them. The p99 of 11 says that the tail is bounded; the max of 69 is a real outlier. codex has no fat tail.

Why does this matter? Because if you only knew about codex you would conclude that the natural per-session prompt count is 2 with very low variance. If you only knew about opencode you would say it is 1. If you only knew about openclaw you would say it is 0. If you only knew about claude-code you would say the distribution is a fat-tailed gamma with mean 17. All four conclusions are correct *for that source*. None of them generalize. The global distribution is the average of four populations that have no business being averaged.

## What happens when you "fix" the global view

Suppose you removed openclaw's 2,034 zero-shell rows because they aren't really sessions in the user-message sense. The global distribution would become:

- n = 6,574,
- zero-prompt count = 451 (all from opencode), or 6.9 percent,
- one-prompt count = 4,913, or 74.7 percent,
- mean = 4.48 (recomputed: previous total prompts was 8,608 × 3.42 = 29,439; same total over 6,574 sessions gives 4.48).

That looks slightly less degenerate, but it is still dominated by the value-1 spike from opencode. To get a shape that looks like a real conversation distribution, you'd have to reduce to claude-code alone, where mean 17.33, p90 44, p99 130 finally reflect the kind of sustained interaction pattern that the word "session" naïvely implies.

The lesson is that "prompts per session" is a per-source quantity. Mixing across sources is not a reduction; it is a category error. The global histogram, the global mean, and the global percentiles all encode the relative *volume* of each source far more than they encode any property of "a session."

## Why the spike at one is structural, not behavioral

It is tempting to read "88.1 percent of opencode sessions have one user message" as a behavioral claim about how people use opencode — that they fire one instruction and walk away. That is not what is happening. Three forces conspire to plant the spike at one:

1. **Tool launch semantics.** Each invocation of opencode in a typical workflow opens a new session and emits one user message corresponding to the initial task. Subsequent agent work is internal — tool calls, sub-agents, retries — none of which increments `user_messages`. So a "one user message" session is the architectural baseline, not a sign of disengagement.

2. **Snapshot timing.** The session-queue is populated by periodic snapshots. A session that *will* eventually have many user messages, but is currently young, gets recorded with whatever count it has at snapshot time. Opencode's tail (max 195 / p99 22) shows that long sessions exist; the spike at one is partly sessions that haven't grown yet.

3. **Definitional skew.** What counts as a "user message" varies across sources. Codex's floor of 2 user messages per session, where opencode's floor is 1 and claude-code's floor is also 1, suggests that codex is counting some message that the others classify differently — possibly an automatic system prompt, possibly a confirmation. The fact that codex has zero one-prompt sessions and zero zero-prompt sessions but a median of 2 is most parsimoniously explained as a counting convention, not a behavioral phenomenon.

If a behavior reading were correct, you would expect codex users to behave differently from opencode users in a way that produces this two-vs-one mode shift. Far simpler to say that codex counts at least one extra message that opencode doesn't.

## Implications for any metric built on "user_messages"

A few practical consequences fall out of this:

1. **Never report a global percentile of `user_messages`.** The global p50, p90, and p99 are functions of source mix. If openclaw's volume changes, those percentiles change without any change in user behavior anywhere. Always condition on source.

2. **Don't compute "average session depth" without explicitly weighting.** The global mean of 3.42 is dominated by openclaw's 2,034 zeros. Weight by something — total messages, or duration, or assistant messages — and the same dataset produces a wildly different "average session depth."

3. **Treat the value-1 cell as a no-information cell for opencode.** A one-prompt opencode session, by itself, contains essentially no signal about how the session went. Almost everything is in the value-1 cell, so the cell does not discriminate. Useful signal lives in the >1 region or in companion fields (assistant_messages, duration, total_messages).

4. **claude-code is the only source you can do conversation-shape analytics on.** If you want to study fat-tail behavior, p90/p99 of user messages, autocorrelation between consecutive prompts, or session-depth-vs-outcome relationships, claude-code is the only source where the field has enough variance and enough non-degenerate values to be worth modeling.

5. **codex is the only source where the floor is informative.** Because codex has a hard floor of 2, any session there with `user_messages = 2` is structurally minimal — no follow-up. That is a useful lower-bound state.

## Closing observation

The global distribution of prompts per session — one big spike at 1, a shadow at 0, a small bump at 2, then a flat tail — looks initially like a striking and counterintuitive empirical fact. It is not. It is an artifact of how four very different processes get glued onto the same field name. The headline number "median session has 1 user message" is true and useless: it tells you that opencode is the largest source by row count, and that opencode mostly emits one-prompt session records. It does not tell you anything about how anyone is working.

The honest summary, conditioning on source, is this: openclaw emits zero-shell session records (n=2,034); opencode emits structurally one-prompt sessions with a thin tail (n=4,988, p50=1, max=195); codex emits two-prompt minima with a bounded short tail (n=478, p50=2, max=69); claude-code emits true conversation-shaped sessions with a heavy fat tail (n=1,108, p50=4, max=398). Those are four different stories. None of them is "the median session has one prompt."
