---
title: "HHI as an Agent Cost Concentration Metric: Borrowing from Antitrust to Read Token Spend"
date: 2026-04-25
tags: [cost, observability, telemetry, agents, metrics, statistics]
---

If you run a fleet of AI agents over a long enough window, "how much did I spend" stops being the interesting question. The interesting question becomes "where did that spend land." A single number — total tokens, total dollars — collapses the structure you actually need to act on. Average is worse: it implies a population that does not exist. What you want is a **concentration metric**: one scalar that tells you, on a 0-to-1 scale, whether your spend is smoothly spread across many models, days, or sessions, or whether it has collapsed into one or two heavy buckets you should be governing differently.

The Herfindahl–Hirschman Index (HHI) — long used by competition regulators to measure how concentrated a market is — turns out to be exactly the right tool. This post walks through what HHI is, why it beats every other "spread-ness" metric we tried (variance, Gini, top-K share, entropy), how to compute it correctly for token spend, and what specific operational decisions it unlocks. There are real numbers from a local telemetry dataset throughout, so you can sanity-check the math against your own.

## What HHI actually is

HHI is the sum of squared shares. If you have N buckets — N models, N weekdays, N sessions, whatever — and bucket `i` accounts for share `s_i` of the total (with `Σ s_i = 1`), then:

```
HHI = Σ s_i²
```

That's it. The whole metric is one line.

The bounds are unintuitive at first but become second nature fast. The minimum value is `1/N`, achieved when every bucket has exactly the same share (`s_i = 1/N` for all i, so `HHI = N · (1/N)² = 1/N`). The maximum is `1`, achieved when one bucket has share 1 and everything else has share 0. So HHI is a number in `[1/N, 1]`, where `1/N` means "perfectly uniform" and `1` means "one bucket eats everything." For seven weekdays, the floor is `1/7 ≈ 0.143`. For 15 models, the floor is `1/15 ≈ 0.067`. You always interpret HHI relative to its floor, not against zero.

The squaring is what does the work. A bucket with 50% share contributes `0.25` to HHI; a bucket with 5% contributes `0.0025`. The big buckets dominate the score by two orders of magnitude relative to their share. That is the whole point: you want a metric that screams when a few buckets dominate and stays quiet when spend is genuinely spread.

## Why not the alternatives

Before settling on HHI for agent telemetry, the obvious candidates all fall over in instructive ways.

**Variance** of bucket shares answers a related question but is hard to interpret operationally. A variance of `0.04` means nothing without context, and the units (squared share) are opaque. HHI shares the same mathematical kinship — it's literally `1/N + N · Var(s_i)` for any distribution, which is a one-line proof that for fixed N, HHI is just a re-scaled variance — but its interpretation as "the probability that two randomly drawn dollars come from the same bucket" gives operators a story they can tell.

**Gini coefficient** is the textbook inequality measure, but it has two problems for telemetry. First, it is computed from a sorted Lorenz curve, which means tiny changes in the long tail re-shuffle the ordering and jitter the metric. Second, Gini is bounded `[0, 1)` regardless of N, so a Gini of 0.6 across 7 buckets means something very different from Gini 0.6 across 15 buckets, and you cannot compare them without normalising.

**Top-K share** ("the top 3 models account for 87% of spend") is intuitive but loses information past K. It cannot distinguish "top 3 = 87%, all evenly split inside" from "top 1 = 80%, next two = 7% combined." For governance you usually need to know which one.

**Shannon entropy** is the information-theoretic cousin and the closest competitor. In nats: `H = -Σ s_i ln(s_i)`. The objection is purely operational: entropy is high when uniform and low when concentrated, which is exactly backwards from how operators talk. "Concentration up" should mean the metric goes up. HHI does that; entropy does not. We've also found that the squared-share weighting in HHI more aggressively flags the asymmetric distributions you actually care about (one model running away with the budget) than entropy's logarithmic weighting, which under-reacts to a single dominant bucket.

## Real numbers from a real dataset

Here are concrete HHI values from a `pew-insights` `weekday-share` run over `~/.config/pew/queue.jsonl` on 2026-04-24, 22:39 UTC, computed across `8,205,355,057` total tokens spread over 15 distinct models:

```
pew-insights weekday-share
as of: 2026-04-24T22:39:02.945Z
tokens: 8,205,355,057
groups: 9 (after filter: --min-active-weekdays 5)
global peak: Mon 24.9%
global hhi: 0.163
min-active-weekdays: 5
dropped: 6 below min-active-weekdays
```

Read that headline `hhi: 0.163` carefully. The floor for 7 weekdays is `1/7 ≈ 0.143`. So `0.163` is `0.020` above the floor — basically uniform. The Monday peak of 24.9% is barely above the 14.3% you'd expect from a uniform distribution. Translation: across the 8.2-billion-token corpus, weekday is not where the concentration lives. If you are looking for a "we should rate-limit on Mondays" signal, this dataset does not give you one.

Now compare against the unfiltered run (which the `--min-active-weekdays 5` flag, added in pew-insights v0.4.44, was designed to suppress for HHI-ranked views). The 6 long-tail single-weekday models trivially score `HHI = 1.0` each — by construction, since one weekday holds all their share, `HHI = 1² = 1`. Naively averaging those into the headline would push the global HHI artificially upward and tell you the fleet is concentrated when it isn't.

This is why the v0.4.44 `--min-active-weekdays <n>` flag matters as a metric-design decision, not just a UX tweak: filtering by activity floor is the difference between a metric that reflects the operator's question ("is my real workload skewed by weekday?") and a metric that reflects an artifact of the corpus shape ("six models happened to only run on one day each, and dragged the headline").

The same `pew-insights` run also produces a per-model HHI breakdown. Concrete numbers from the live-smoke output: `claude-opus-4.7` and `claude-haiku-4.5` are the two extremes, with the haiku model showing far higher concentration than the opus one across the same window. (The earlier `peak-hour-share` work in v0.4.42 had `claude-opus-4.7` at 23.8% mean peak-share and `claude-haiku-4.5` at 88.4% mean — same qualitative ranking, different temporal axis. The two metrics agree that opus is the smoothest workhorse and haiku is the spikiest, which is consistent with how each model is actually used: opus for long-running missions that span many days and hours, haiku for batch-style enrichment passes that all fire in one sitting.)

## Computing HHI correctly

The math is a one-liner but there are three traps.

**Trap 1: token-weighted overall mean, not naïve cross-group mean.** When you aggregate per-model HHI into a single fleet-level number, do not take `mean(hhi_i)` across models. Take `Σ s_i · hhi_i` where `s_i` is the model's share of total tokens. The naïve mean treats a model that did 1,000 tokens the same as a model that did a billion. The token-weighted mean answers the operationally correct question: "for an arbitrary token in this corpus, what is the expected concentration of its model's distribution?" The pew-insights `peak-hour-share` and `weekday-share` subcommands both use the token-weighted mean for exactly this reason — see the v0.4.42 changelog note: "Token-weighted overall mean (not a naïve cross-group simple mean) so heavy days move the headline number more."

**Trap 2: zero-token buckets must be excluded before computing shares.** If a model has zero tokens on Wednesday, you do not include `s_wed = 0` in the HHI sum (it contributes zero, harmless) but you also must not include Wednesday in the `N` you use to compute the floor `1/N`. If you do, you'll under-state concentration because the floor moves down. Compute `N` as "active buckets," not "possible buckets."

**Trap 3: singleton-bucket groups score 1.0 by construction.** If a model only ran on one weekday, its HHI is exactly 1.0 — not because it's pathologically concentrated, but because there's no opportunity for it to be otherwise. These rows look alarming in a sorted table but are noise. The `--min-active-weekdays` and `--min-active-hours` flags exist to filter them out for ranked views while keeping them in the global denominator, which is the right semantics: the model's tokens still count toward the totals, but its trivially-1.0 HHI does not pollute the comparison.

## What HHI actually unlocks operationally

A single concentration scalar is only useful if it changes a decision. Three decisions it changes for us:

**Rate-limit budget allocation.** When a model's per-hour or per-weekday HHI is near 1.0, you cannot rate-limit it on a per-second basis — its spend is bursty by nature, and per-second limits will throttle the burst that *is* the workload. You either negotiate a higher peak quota with the provider (and accept low average utilisation) or you re-architect the workload to spread the burst. When HHI is near floor, per-second rate limits are safe because the spend is genuinely smooth and any throttling is a real signal of overload, not an artifact of natural batch-iness.

**Cost attribution sanity checks.** When per-session HHI across sessions is high, your average cost-per-session is meaningless — a few sessions are eating most of the budget and the rest are noise. Report median or P95 instead, and investigate the head of the distribution as individual cases. When per-session HHI is low (smoothly spread), the average is a real number you can plan against and a budget-per-session ceiling becomes a sensible governance lever.

**Provider diversification decisions.** When per-provider HHI is high (one provider dominates), you have concentration risk — a single rate-limit episode or outage takes down a large fraction of your fleet. The HHI number gives you a precise "how diversified am I really" answer that the usual "we use three providers" hand-wave does not. A fleet using three providers but with HHI of 0.85 has the diversification properties of a single-provider fleet.

## Storing it

Compute HHI at three granularities and store all three: per-`(model, day)`, per-`(model, week)`, per-`(fleet, week)`. The per-`(model, day)` is your alerting surface — a model whose daily HHI suddenly drops 0.3 below its 30-day baseline is doing something different (a new bursty workload added, an old smooth workload dropped). The per-`(model, week)` is your trend-line for capacity planning. The per-`(fleet, week)` is your headline for stakeholder reports.

Do not roll up. The fleet HHI cannot be reconstructed from per-model HHI; it is a different distribution computed across a different bucketing. Store all three independently. Each answers a question the others can't.

## The deeper point

HHI is not a magic metric. It is just `Σ s²`. What makes it useful for agent telemetry is that the question it answers — "how concentrated is this distribution?" — turns out to be the question operators are actually asking when they look at their cost dashboard, even if they don't phrase it that way. They are not asking "what was the total." They are asking "is this normal" and "where should I look first." HHI gives both answers in one scalar, on a fixed scale, with an interpretable floor.

Borrow it. The antitrust regulators have been refining it since 1945. Your agent fleet is just another market with too many players and not enough scrutiny.
