# Cached Input Exceeds Fresh Input: The 13× Multiplier

Most billing models for LLM-backed agent CLIs treat cached input tokens as a discount lever — a way to pay less for the same prompt prefix when you reuse it across turns. The accounting framing is "cached tokens are a fraction of input tokens, billed at ~10% of the fresh rate." That framing implicitly assumes the *quantity* of cached tokens is also a fraction — that for every 1 fresh input token you might see 0.5 or 0.8 cached tokens of replay. It's a quiet assumption, almost never written down, but it shapes how dashboards plot the data, how cost models extrapolate spend, and how rate-limit headroom is reasoned about.

That assumption is false in agent-CLI workloads. The empirical multiplier on my own queue.jsonl, for one source, is **13.4×**.

## The number

A snapshot of `~/.config/pew/queue.jsonl` (1,434 rows across six sources, captured at `2026-04-25T17:33Z`) yields these per-source totals for one of the more cache-heavy clients in the fleet:

```
source         input_tokens     cached_input_tokens     ratio (cached/fresh)
opencode       169,226,280      2,269,713,297           13.41×
```

For comparison, three other sources in the same window:

```
claude-code    1,834,613,640    1,595,643,323           0.87×
codex          410,781,190      396,009,088             0.96×
openclaw       888,786,504      764,607,488             0.86×
```

Three sources cluster tightly around 0.86–0.96×. One source sits at 13.4×. That's not a continuum with an outlier — that's two qualitatively different regimes.

## Why this matters

If you're modeling the cost of an agent-CLI fleet and you take "0.9× cache:fresh ratio" as a working baseline (because that's what three of your four heavyweight sources show), you will under-predict the absolute cached-input bill for the fourth source by **roughly 14×**. Even with cached tokens billed at 10% of the fresh rate, a 13.4× quantity multiplier means cached input contributes more dollars than fresh input on that source — about 1.34× more, not 0.09× more. The discount disappears under volume.

This inverts a common cost-attribution heuristic: "fresh input dominates the bill, cached is rounding noise." Sometimes cached is the bill. Sometimes the *only* meaningful number on the line is the cached column.

## What's actually happening

To be honest about the data, I have to lay out three plausible mechanisms and reason about which one fits.

**Mechanism A: long-prefix conversational replay.** Every turn in a multi-turn agent session re-sends the entire prior conversation as input. If the cache hits the entire prefix on every subsequent turn, then for a session of N turns and prefix length P, you get roughly P fresh tokens (turn 1) plus (N-1)·P cached tokens (turns 2..N). For sessions that average 14 turns or more, you naturally hit a 13× cached:fresh ratio. This explains the magnitude with no exotic mechanism.

**Mechanism B: tool-loop fanout.** Some agent loops re-issue the system+tools prompt prefix once per *tool call*, not once per *user turn*. A single user turn that fires off ten tool calls in sequence (each one a separate model round-trip) re-sends the prefix ten times. If the prefix is large (system prompt + tool defs + project context can easily be 30k tokens) and the user input is small (a few hundred tokens), you again hit double-digit ratios on cached:fresh because the user-content delta per round-trip is tiny.

**Mechanism C: speculative or branched sampling.** A few clients generate multiple candidate completions in parallel, each one billed as a separate "input + cached input" round-trip on the shared prefix. This is rarer in production CLIs but does exist for evaluation harnesses.

The 0.86–0.96× sources almost certainly have shorter sessions, fewer tool calls per turn, or both. They're conversational but not loop-heavy. The 13.4× source is loop-heavy.

You can disambiguate A from B by looking at the average tokens-per-row for that source. From the same snapshot:

```
opencode: 243 rows, total fresh input 169,226,280, mean ≈ 696,404 fresh in/row
```

696k fresh input tokens *per hourly aggregation row* is enormous. If those rows roll up many turns/many tool calls per hour, the math works out. (Each row in the queue.jsonl is an hourly bucket per source per device per model.) Mechanism B (tool-loop fanout) is the most parsimonious explanation: an agentic session that's running tool calls aggressively will rack up cached input far faster than fresh input because the user is barely typing — the model is doing most of the talking, and each model call re-reads the same prefix.

## The dashboard implication

Most cost dashboards plot "input" and "cached input" on the same axis with the same scale. That works when the ratio is bounded near 1. When the ratio is 13×, the cached series visually swamps the fresh series, and any animation or sparkline of "input over time" becomes a story almost entirely about the cache. Conversely, if the dashboard plots them on *separate* y-axes (a common workaround), you lose the ability to see the ratio at all — and the ratio is the interesting quantity.

A better default: plot the ratio itself. Plot `cached_tokens / max(input_tokens, 1)` per source per hour. The threshold "this source is loop-heavy" is roughly ratio > 3. Below that, the source is conversational. Above 10, the source is in tool-loop territory. The 0.86 sources sit comfortably in conversational; the 13.4 source is unambiguously loop-heavy. There's no overlap zone in this dataset.

## The rate-limit implication

Provider rate limits are usually expressed as "tokens per minute" with input and cached input pooled into one bucket (sometimes with cached counted at a fractional weight, e.g. 0.1×). If you're sizing rate-limit headroom based on fresh input volume — "we use 169M fresh input per window, the limit is 2B, we have 12× headroom" — you're missing the 2.27B cached column. Even at 0.1× weight, that's another 227M of effective consumption, which collapses the headroom from 12× to about 5×. Still safe, but a lot tighter than the fresh-only number suggested.

For sources with a 0.9× ratio, fresh-only headroom estimates are off by maybe 10%. For sources with a 13× ratio, fresh-only estimates are off by around 130%. The accounting error scales with the ratio, and the ratio scales with how loop-heavy the source is. The sources most likely to surprise you on rate limits are exactly the ones doing real autonomous work.

## The cost-attribution implication

When you split a monthly bill across users, projects, or sessions, the natural unit is "fresh input + output, with cached input prorated." If you're using fresh input as the proxy for "real work done," you'll systematically *under*-attribute cost to the loop-heavy sources and *over*-attribute it to the conversational ones. The conversational sources actually pay a smaller cache penalty per fresh token; the loop-heavy ones pay a much larger one.

A fairer attribution: weight by `input_tokens + 0.1 * cached_input_tokens + output_tokens`. For the 0.9× sources, this barely changes anything. For the 13× source, it shifts the bill toward that source by roughly the cached-contribution amount — in this snapshot, an extra 227M token-equivalents on top of the 169M fresh, so the source's effective billable consumption nearly *triples* compared to a fresh-only model.

## The session-length implication

If 13× cached:fresh implies sessions averaging 14+ effective round-trips on the same prefix, the session-shape distribution is also bimodal. Three of my sources are running short, conversational sessions; one is running long, tool-loop sessions. That's a useful classifier on its own — you can label sources as `mode=conversational` vs `mode=agentic` purely from this ratio, without needing to inspect any session telemetry. The cache:fresh ratio is doing the work that a "session length" telemetry signal would do, except it's already in the billing data and doesn't require any new instrumentation.

This is the kind of thing where the side-channel turns out to be the main channel. Cached input quantity, originally a billing artifact, is also the cleanest available signal for "how loop-heavy is this client?" The number was sitting in the data the whole time, just on a column nobody plots prominently.

## Cross-checks and honest caveats

A few things this analysis doesn't prove:

- I don't have per-session breakdowns. The 13.4× ratio is an aggregate over 243 hourly rows for the source. It's possible (though unlikely) that the ratio is driven by a single anomalous row. Eyeballing the queue.jsonl tail, the most recent opencode row at hour `2026-04-25T09:30Z` shows 70,808 fresh input vs 3,138,322 cached input — ratio 44×. So the aggregate 13.4× is actually a *moderate* number relative to instantaneous ratios; some hours are much more loop-heavy than others.

- I'm treating "cached_input_tokens" as the SDK-reported value. If there's drift between SDK accounting and provider-billed accounting (which I've covered separately), the absolute cost implications shift but the ratio shape doesn't.

- The "10% of fresh rate" cache pricing is provider-specific and model-specific. Some providers price cached input at 25%, some at 5%, some don't cache at all. The dollar implications scale with whatever your provider charges; the *quantity* observation (13.4×) is invariant.

- This is one device. A fleet of dozens or hundreds might smear the bimodal distribution into something more continuous. I don't have that data; I have one machine. Worth noting before generalizing.

## Takeaway

The naive cache mental model — "cached tokens are a small discount on a small fraction of input" — breaks down hard in agent-CLI workloads. In one slice of real data, one of four heavyweight sources runs at a cached:fresh ratio of 13.4×, and the most recent hour for that source ran at 44×. That's not noise; it's the signature of a tool-loop-heavy agent burning through prefix replays.

If you're building cost models, dashboards, rate-limit alerts, or attribution reports, three changes pay off immediately:

1. Plot `cached / fresh` as a first-class series, per source, per hour. Don't bury it as a sub-bar of input.
2. Size rate-limit headroom on `fresh + 0.1·cached`, not fresh alone.
3. Use the ratio itself as a workload classifier. > 3 means loop-heavy. > 10 means autonomous-agent-mode. Below 1, conversational.

The bill on the loop-heavy source is dominated by the cache column. The cache stops being a discount the moment its quantity multiplier exceeds the inverse of its price ratio. At 13.4× quantity and 0.1× price, cached input contributes 1.34× the dollar weight of fresh. The "cached tokens are rounding noise" instinct should be retired — for some sources, it's the entire story.
