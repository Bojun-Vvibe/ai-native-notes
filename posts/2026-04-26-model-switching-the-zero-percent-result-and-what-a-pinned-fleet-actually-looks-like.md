# Model-switching: the 0.0% result, and what a pinned fleet actually looks like

A lot of agent-fleet metrics are interesting because of what they reveal. This one is interesting because of what it *doesn't* reveal — and the absence is, itself, a strong signal about how this fleet is wired.

I ran `pew-insights model-switching` (`pew-insights` v0.5.6, captured at `2026-04-25T21:47:03.354Z`) and got the following top line:

```
sessions: 6,159    switched: 1 (0.0%)
transitions: 24 across 2 pairs
mean models / switched session: 2.00
p50 distinct models: 1
p90 distinct models: 1
p99 distinct models: 1
max distinct models: 2
```

That is not a near-zero. It is a *categorical* zero with a single anomalous unit. Out of `6,159` sessions whose snapshots were inspected for any change in the `model` field, exactly **one** session crossed a model boundary, and that session bounced between two values — `delivery-mirror` and `gpt-5.4` — for `24` recorded transitions, `12` in each direction.

This is worth unpacking, because most fleet-design intuitions implicitly assume model-switching is at least *occasional*. Routers exist, fallbacks exist, A/B harnesses exist. A 0.0% switched-share on a fleet that is otherwise running at billions of tokens per day means something specific about the architecture — and it has consequences I didn't anticipate when I started writing this post.

## What `model-switching` actually measures

The lens is per-session, not per-bucket. `pew-insights` inspects the snapshot stream attached to each session in `session-queue.jsonl`, projects out the `model` field on each snapshot, and asks: did this set ever exceed cardinality 1 within a single session? It is not asking "did the operator pick a different model next time." It is asking "did the model change *during* a single user-driven conversation."

That distinction matters because there are two very different ways a fleet can have apparent model diversity:

1. **Inter-session diversity.** Operator opens session A on model X, then session B on model Y. Each session is internally pinned, but the population shows multiple models.
2. **Intra-session diversity.** A single session crosses models — typically because of a router, a fallback after a 429/5xx, an explicit `/model` command, or a tool-mediated handoff to a different agent.

`model-switching` ignores type 1 entirely. The `1 (0.0%)` is a statement strictly about type 2. So the right reading isn't "this fleet only uses one model" — earlier posts have already shown this fleet runs `gpt-5.4`, several Claude variants, `delivery-mirror`, and others. It's "once a session begins, the model it is using is fixed for the duration."

## Why this is not the default

If you have ever wired up an OpenAI Chat Completions client that catches `RateLimitError` and retries on a smaller model, your sessions probably have nonzero switched-share. Same if you use a router like LiteLLM with fallback chains, or any framework whose runtime config gets reread between turns. The default behavior of *most* agent stacks is to allow the model identity to drift inside a session, because the model identity is treated as configuration rather than as part of the session's identity.

What this 0.0% number tells me is that every CLI surface feeding `session-queue.jsonl` here treats the model as **bound at session creation**. Once the session record is opened, the snapshot stream's `model` field is whatever was negotiated up front, and nothing in the request path mutates it. The single switched session — `delivery-mirror ↔ gpt-5.4`, `24 transitions` evenly split — is almost certainly an instrumentation artifact rather than a real switch: `delivery-mirror` is the recently-introduced shipping/staging label that some sources tag onto outgoing requests when the model identity is being remapped at the edge. That session likely had its model field oscillating because the *labeller* changed, not because the underlying inference target changed.

In other words, the 0.0% may actually be 0.0% — the lone anomaly is plausibly a renaming event, not a real intra-session switch.

## Distinct-model histogram, in full

```
distinct models  count  share
---------------  -----  ------
1                6,158  100.0%
2                1      0.0%
3                0      0.0%
4+               0      0.0%
```

`p99 distinct models: 1` means even at the 99th percentile of session-shape, sessions stay on a single model. This is the histogram of a fleet whose configuration is **immutable per session**. Agentic frameworks that do mid-session model switching tend to produce a noticeable bump at distinct-models = 2 (the "fallback after one failure" pattern) and a long thin tail out to 4 or 5 (the "router that escalates on every retry" pattern). Neither of those bumps exists here.

The implication is that any cost-routing or capability-routing this fleet does happens **before** the session is opened, not during it. The pre-flight decision — "I'm going to talk to gpt-5.4 about this" or "I'm going to ask Claude to do that" — is sticky for the session's whole lifetime. If the operator wants a different model, they start a new session. The data shows they do exactly that, ~6,159 times in this window.

## The single transition pair, read carefully

```
top transitions (from → to):
delivery-mirror  gpt-5.4          12     50.0%
gpt-5.4          delivery-mirror  12     50.0%
```

Twelve transitions in each direction, perfectly symmetric. That symmetry is itself diagnostic. A real fallback pattern is asymmetric — you fall back from the expensive model to the cheap one many more times than you climb back. A real upgrade pattern is similarly asymmetric — once you're on the better model, you stay. Symmetric transitions of equal count are what you see when the same session gets re-read from disk, and the labeller tags consecutive snapshots with alternating values. This is the signature of an instrumentation-level mismatch, not a behavioral switch.

So even the one switched session probably isn't a switch. It is probably a single conversation whose snapshot stream got tagged inconsistently between two synonyms for the same target.

## What this implies for downstream metrics

A 0.0% switched-share simplifies several of the other lenses I've been writing about, and it changes how you should read them:

- **`prompt-output-correlation`.** Per-session correlations are clean here, because you don't have to worry about model-induced regime changes inside a single session. Each session is one population.
- **`reasoning-share` per model.** The reasoning-token mass attributed to a model is unambiguous: every session that *used* a model used only that model.
- **`cache-hit-ratio` per model.** Cache hits are not contaminated by mid-session model changes that would otherwise flush a per-key cache.
- **`turn-cadence`, `reply-ratio`, `session-lengths`.** Per-session distributions are cleanly attributable to the single model that owned the session — no need for token-weighted apportionment.
- **`model-cohabitation`.** This metric measures cross-model *bucket* sharing, not cross-model *session* sharing, so cohabitation can be high even when intra-session switching is zero. The two lenses are orthogonal, and now I know they truly are.

What it complicates is anything that wants to study fallback behavior or router efficacy. There is no observable router-induced variance in this dataset, because there is no router that mutates session-bound state. If I want to study routing, I have to do it at the *session-creation* layer, not the snapshot layer.

## How this gets violated, and how to detect when it changes

Three things would push the switched-share above 0.0% in the future, and each is worth a watch:

1. **Adding a real intra-session router.** If I wire up a LiteLLM-style fallback chain or a "downgrade on rate-limit" handler, sessions will start crossing model boundaries. The first month after that goes live, switched-share should climb from 0.0% to somewhere in the 1–5% range, and the distinct-models histogram should grow a visible bar at 2.
2. **Multi-agent sessions.** If a single session legitimately invokes two different agents — e.g., a planner on Claude and an executor on gpt-5.4 — and the snapshot stream records both, this lens will register transitions. The pair count will grow beyond 2 (it's currently capped at the single anomaly), and the from→to matrix will start showing asymmetry.
3. **Snapshot-labelling cleanups.** Conversely, if I clean up the `delivery-mirror` aliasing so that a single inference target gets a single label in the snapshot, the lone switched session will disappear and switched-share will drop from "0.0% with one outlier" to "0.0% with no outliers." That is also a real change worth recording, because it tells me the labeller is now consistent.

## The metric that proves the architectural choice

The reason I find this lens useful is that it exposes an architectural decision — **model identity is part of session identity** — without me having to read any source code. The number `6,158 / 6,159 = 100.0%` is the architectural decision, written in tokens.

Most "interesting" metric posts find an unexpected distribution. This one found an unexpectedly *flat* distribution, and the flatness is the story. The fleet is wired so that the operator chooses the model up front and lives with it. Every other lens — cache-hit, reasoning-share, prompt-output correlation, model-cohabitation — should be read with that constraint in mind.

If you're staring at a model-switching report on your own fleet and seeing a distinct-models histogram with a real bump at 2 or 3, you have a different architecture from this one. That bump is your router. The flatness here is the absence of one.

## Footnotes for reproducibility

- Tool: `pew-insights` v0.5.6 (CLI binary at `~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js`)
- Subcommand: `pew-insights model-switching`
- Capture timestamp: `2026-04-25T21:47:03.354Z`
- Window: all (default), `min-switches: 2`, `top: 10`
- Sessions inspected: `6,159`
- Switched: `1 (0.0%)`, transitions `24 across 2 pairs`
- The single anomaly: a `delivery-mirror ↔ gpt-5.4` symmetric pair, 12+12 transitions, plausibly an aliasing artifact rather than a real intra-session switch.

The next time I run this, the timestamp and the session count will be different, but if the switched-share is still 0.0% with at most one anomaly, I will know nothing about the fleet's session-binding architecture has changed. If the switched-share has climbed even to 0.5%, that's the signal that a router has been wired in — and I'll need to revisit every per-model lens with the new awareness that sessions are no longer guaranteed to be model-pinned.

Zero is, occasionally, the most informative number a metric can produce. This is one of those cases.
