# The single-model session lock: 6,603 of 6,604 sessions stay on one model, and the only switch is a mirror

**As of 2026-04-27T03:16:09.090Z**, the `pew-insights model-switching` lens over my local `~/.config/pew/queue.jsonl` returned a number that, the first time I read it, I thought was a bug:

```
sessions                        6,604
switched                        1
switched share                  0.0%
transitions                     24 across 2 pairs
mean models / switched session  2.00
p50 distinct models             1
p90 distinct models             1
p99 distinct models             1
max distinct models             2
```

One. Out of 6,604.

And the single switched session is not a human picking up a new model mid-conversation. It is an entirely synthetic loop: 12 transitions from `delivery-mirror` to `gpt-5.4` and 12 transitions back, on the same session, alternating across snapshots. A mirror is not a switch.

So the operational reading is: **in this corpus, no session ever changes its underlying model.** P50 = P90 = P99 = max = 1. The distribution is a degenerate point mass at 1, with one outlier whose 2 is a measurement artifact.

This is a weird number, and most of the writing about model-switching assumes the opposite — that operators try multiple models inside a single conversation, that "fall back to a cheaper model when the cache is hot" is a real session-shape, that "escalate to opus when sonnet refuses" happens. None of that shows up here. Below is what I think the number means and what it definitely doesn't.

## Why 0.0% is a structural number, not a behavioural one

The first instinct is to read 0.0% as "Bojun never switches models," which would be a behavioural claim. But the number is structural — it falls out of how a session is constructed in the first place — and reading it that way makes the whole rest of the analysis hang together.

A "session" in this telemetry is not a UI conversation. It is a contiguous run of snapshots emitted by a single CLI / harness invocation against a single configured model. When you launch `claude-code` with `claude-opus-4.7` selected, the session inherits that model and emits snapshots all carrying the same model field. When the harness exits and you re-invoke it with a different model, that's a new session, with a new `session_id`, and the model-switching lens would never connect the two.

Said differently: **most of the surfaces I use route their model choice at process-launch time, not at message-send time.** The model is a startup flag, not a conversational setting. So "switching models inside a session" would require the harness itself to choose a different model for a single message — a tool-call sub-task, a fallback, a router decision — and then emit that under the same `session_id`. That's the only path to a non-1 distinct-model count, and in 6,604 sessions exactly one ever took it.

The only path is `delivery-mirror ↔ gpt-5.4`. The other 6,603 are all "session = single model."

## Why this is the opposite of what 2024-era takes predicted

Two years ago there was a steady drumbeat of "multi-model orchestration" think pieces predicting that production agents would routinely fan out to multiple models per session: small for routing, big for reasoning, tiny for embedding-time classification, big again for synthesis. The session would be a multi-model graph; each model would handle the slice of work where it dominated on cost-per-correct-answer.

The data from a single heavy operator's queue says: that did not happen at the session granularity. It might be happening at the *deployment* granularity — different sessions go to different models — but inside a session, there is one model and that model handles every turn.

Three reasons I think this is structurally true and not just a Bojun-corpus artifact:

1. **State is expensive to migrate.** Switching models mid-session means re-priming the system prompt, re-warming the prompt cache (anthropic has a 5-minute cache TTL — switching providers throws all of that away), and re-establishing tool schemas. A 60-second context-rebuild penalty per switch dominates almost any latency or cost win you'd hope to get from the switch itself.

2. **Cache geometry punishes switching even more than latency.** I've written before about the 29.4% cache-hit-ratio outlier on `claude-opus-4.7` — when one in three input tokens is served from cache, the dollar-per-token cost is heavily front-loaded onto the *first* call of a session. Every subsequent call free-rides on prefix-cache. Switching providers means paying that front-load again. The cache penalty for switching scales with how cache-hit-friendly the original session was, which means **the more efficient your session is, the more expensive a switch becomes** — exactly backwards from what naive cost-routing logic predicts.

3. **Tool schemas are model-specific.** Anthropic, OpenAI, and Google all expose tool-calling as JSON-schema-shaped, but the prompt-injection of those schemas, the way refusals get formatted, and the way tool errors surface in the response are different enough that "swap model mid-session" tends to break harness state machines. The harness vendors solved this by simply not supporting it.

So 0.0% is what you'd predict from the engineering, even before you look at the data.

## The mirror artifact: what `delivery-mirror ↔ gpt-5.4` actually is

The one switched session has 24 transitions across 2 pairs, distributed exactly 12/12. That's not a switch — that's a delivery shadow. `delivery-mirror` is what the local pew tagging logic calls a snapshot whose model field is set to a routing alias rather than a concrete model, and `gpt-5.4` is the resolved alias. Some snapshot writers tag a row as `delivery-mirror` while it's in flight and re-tag it as `gpt-5.4` once the response lands. Or the reverse — the resolved row goes first and the mirror row gets retroactively tagged.

You can see that this is mechanical, not conversational, from two clues:

- **The transition count is exactly 2× the pair count.** 12 from-mirror, 12 to-mirror, 24 total. Each "switch" is paired with a return. In a real conversation the operator would not undo every model choice immediately.
- **The mean-models-per-switched-session is exactly 2.00.** Not 2.1, not 2.5, not 3.0 — exactly two distinct strings appear, alternating. A real switching pattern would have at least some sessions reaching 3+ distinct models if the operator was actively choosing.

So the corpus reports zero real switches. The mean of 2.00 is the ceiling of the artifact, not a sign that switching happens "at least sometimes."

## What this tells you about session boundaries

If a session can never contain more than one model in practice, then **the session is, operationally, the unit of model commitment.** Every other lens we run — turn cadence, reply-ratio, cache-hit-ratio, reasoning share — gets to assume that the model is constant within its window. We've been making that assumption implicitly. Now we know we were right.

That's a non-trivial guarantee. It means:

- **You can attribute session-level cost to a single model SKU** without doing per-row weighting. Sum input tokens × that model's input rate, sum output × output rate, that's the bill. No need to track per-snapshot model.
- **Cache-hit-ratio is well-defined per session.** Every cache hit lives in the same model's cache namespace as every cache miss in that session. You can write `cache_hit_ratio = sum(cached_input) / sum(input)` over the session and it's not lying to you.
- **Reasoning-share, when present, is attributable to one model.** The earlier analysis where I saw codex at 38.6% reasoning share and IDE-assistant-A at 14.9% was implicitly assuming session-level attribution; this lens confirms that assumption holds.
- **Provider-mix is a between-session phenomenon, not a within-session one.** If you want to ask "what fraction of my work goes to anthropic vs openai," you have to ask it at the session level. The within-session answer is always 100%-or-0%.

This last point is the one I find most interesting, because it implies that **the right granularity for "do I depend on provider X" questions is sessions, not tokens.** A heavy single-session run on anthropic and a heavy single-session run on openai look very different on a per-token chart even if they represent the same level of operator commitment. The session count is the commitment unit; the token count is the consumption volume. Those are two axes, not one.

## The rank-1 model dimension and what fits inside it

If `distinct_models_per_session` is a rank-1 distribution, then any feature you derive from "how does the model change inside a session" is also rank-1 — i.e., dead. There is no "model-volatility-per-session" axis. There is no "first-model-vs-last-model" axis. There is no "mean dollar-per-token across the session given heterogeneous models" axis.

What survives:

- **First-model = only-model.** So the model field of any snapshot in a session is, with overwhelming probability (6,603/6,604 ≈ 99.985%), the model field of every snapshot in that session. You can derive session-level model from any single row.
- **Switched-share is bounded above by the harness-routing-rate.** If a future tool ships true mid-session model routing — say, a router that escalates to a bigger model on hard tool-calls — switched-share will go from 0.0% to whatever fraction of sessions that router fires in. Watching this number move is therefore a sensitive indicator of "did one of my tools start doing real model routing."
- **Distinct-models-per-session is a step function on harness updates, not on operator behaviour.** The number won't move because I changed how I work. It'll move because some harness vendor shipped a feature.

That third point is what makes this metric worth keeping in the dashboard even when it's a flat zero. **A flat zero with a clear mechanical explanation is a high-signal monitor.** Any deviation is news.

## Comparing to message-volume to confirm the floor is real

`pew-insights provider-share` for the same `as of` window reports 7,014 sessions and 241,087 messages — so mean ≈ 34 messages/session. Some of those are user turns, some assistant, some tool calls. If even 1% of sessions had a real model-switch event happening on, say, half their messages, you'd expect roughly 70 switched sessions and a few hundred transitions. We have 1 switched session and 24 transitions, all on the same mirror artifact.

The discrepancy isn't just "0% rounded down." It's three orders of magnitude below where you'd start to see real switching. The floor is real.

## What I'd want to see before declaring the floor universal

Caveats I'm aware of:

- **N = 1 operator.** This is one user's queue. A team with a router-fronted gateway (e.g., LiteLLM doing model-cost-aware routing in front of every call) would presumably show non-trivial switched-share, because the router is making per-message decisions. If you put 50 such operators in one queue, switched-share might run 5–20%.
- **Session definition is harness-specific.** If a future harness redefines "session" to include router-level fallbacks as same-session events (instead of starting a new session each time the model resolves differently), the number will jump without any underlying behaviour change.
- **Mirror artifacts can multiply.** If more rows start getting tagged with routing-alias names and then resolved, switched-share will inflate without any real mid-session model change. Worth filtering known mirror pairs out of the lens. The current report's `delivery-mirror ↔ gpt-5.4` 12/12 pattern is a good signature to denylist.

But within those caveats, the floor is exactly where I'd expect it from first principles: at zero, plus one mechanical artifact.

## Practical takeaways

1. **Treat session-level model attribution as exact, not approximate.** Cost reports, cache analyses, reasoning-share analyses, and any per-model latency benchmark can use session-level aggregation without penalty. We've now confirmed within-session model is constant to ~5 nines.

2. **Watch `switched_share` as a low-bandwidth canary.** It should stay at 0.0%. Any time it moves to even 0.1%, something interesting just happened in tooling — either a new harness shipped a router, or a mirror tag started leaking into a new pair. Both are worth chasing.

3. **Stop treating "multi-model orchestration" as a session-level concept.** The session is the unit of model commitment. Multi-model orchestration is a between-session phenomenon: I run claude for one session, gpt for another, gemini for a third. That is a real and observable behaviour (the provider-share lens shows anthropic at 58.4% sessions, openai at 14.5%, unknown at 27.1%) — but it lives at the session-mix level, not the within-session level. Conflating those two has been a low-grade error in how the multi-model literature talks.

4. **Quote `delivery-mirror ↔ gpt-5.4` when the mirror story comes up again.** It's a clean example of how routing aliases produce phantom switches that look like behaviour but aren't. It will come up again the moment any other routing alias lands in the queue.

5. **The corpus's answer to "should I be a multi-model operator within a session?" is "you literally cannot, the harnesses don't support it."** This is a constraint you can stop trying to optimize against. Spend the optimization budget on between-session routing instead — choose the right harness for the task, then let the harness commit to its model.

The headline number is `switched share: 0.0% (1 / 6,604)`. The only switch is a mirror. The rest is silence.

That silence is the signal: **the session is the model.**
