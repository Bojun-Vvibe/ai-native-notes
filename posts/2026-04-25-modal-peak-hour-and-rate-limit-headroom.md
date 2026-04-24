---
title: "Modal Peak Hour and Rate-Limit Headroom: Capacity Planning for Bursty Agent Workloads"
date: 2026-04-25
tags: [cost, capacity, rate-limits, observability, agents, telemetry]
---

Provider rate limits do not care about your daily totals. They care about what you do in any given minute, hour, or rolling window. So a per-day or per-week token report — the kind every billing dashboard ships by default — tells you almost nothing about whether you are about to get throttled. The number you actually need is **the share of your day's spend that lands in your single busiest hour**, and the number you need beside it is **which hour that usually is**. Together these two numbers tell you whether your workload is batch-shaped or smooth-shaped, where the burst lives in wall-clock time, and how much rate-limit headroom you have before the next provider-side capacity limit bites.

This post is about computing those two numbers correctly, what they mean operationally, and how a real dataset of 8+ billion tokens across 15 models splits cleanly into "smooth workhorses" and "spiky batch jobs" once you look at it through the peak-hour-share lens.

## The metric

For each `(group, day)` pair — where a group is a model, a source, a project, whatever your aggregation axis is — sum total tokens by hour-of-day (UTC), find the single hour with the largest sum, and record its share of the day's total. That single number is the day's peak-hour-share for that group. For a perfectly uniform 24-hour day, peak-hour-share is `1/24 ≈ 0.0417`. For a day where everything happened in one hour, it is `1.0`. Real workloads land somewhere between, and the *somewhere between* is what tells you the workload's shape.

Aggregate across days for a group as four numbers: mean, p50, p95, max. Then add two metadata fields: the **modal peak hour** (which clock-hour most often holds the daily peak across the group's days) and the **dominance ratio** (`modalPeakHourCount / days` — how often the modal hour wins).

That's six numbers per group. They give you a picture per-day of any other metric.

## Real numbers, two extremes

From a `pew-insights peak-hour-share` run (the v0.4.42 release) against `~/.config/pew/queue.jsonl`, two models bracket the spectrum cleanly:

- `claude-opus-4.7` — **mean peak-share 23.8%**. Smooth workhorse. On an average day, the busiest hour holds only 23.8% of the day's tokens, leaving 76.2% spread across the remaining 23 hours. This is what a long-running interactive workload looks like.

- `claude-haiku-4.5` — **mean peak-share 88.4%**. Spiky batch job. On an average day, 88.4% of the day's tokens land in a single hour. This is what a scheduled enrichment pass looks like — fire at the same time, drain in one window, idle the rest of the day.

The 64.6-percentage-point gap between these two models, on the same fleet, on overlapping days, is the entire point. They are technically "the same kind of thing" (LLM inference on the same provider) but their *operational shape* is so different that any single rate-limit policy applied to both will be wrong for at least one. If you set a per-second cap that's safe for opus's smooth pattern, haiku's batch will hit it instantly and shed work. If you set a peak-hour cap that's safe for haiku's burst, opus's smooth flow will never come close to using the budget.

## Generalising: the top-K-hours-by-mass refinement

The single-busiest-hour view is the leading-order signal, but it can mislead in cases where the burst is wider than 1 hour but narrower than the day. Concrete from the same pew-insights live smoke: at `K=3` (top three hours), `claude-opus-4.7`'s peak-share rises from 23.8% to **51.6%**. That is a meaningful jump — it says about half of opus's day actually does cluster into a 3-hour window, the work is just smooth enough within that window that no single hour stands out. Haiku at K=3 is essentially saturated (close to 100%, since the bulk was already in 1 hour and the next two hours pick up the residual).

So the right operational question is not "what's K=1" but "how does peak-share change as you sweep K from 1 to 24." A workload where peak-share is 25% at K=1 and rises smoothly to 50% at K=3 and 75% at K=12 has a "rolling-on" pattern. A workload where peak-share is 88% at K=1 and stays near 88% at K=2 and K=3 has a "tight burst" pattern. They demand different rate-limit strategies and different reservation windows.

The pew-insights v0.4.42 changelog notes this directly: the `--peak-window <k>` refinement generalises peak-hour-share to top-K-hours-by-mass. Use K=1 for "where's the spike," use K=3 to K=6 for "where's the campaign," and use the full 24-hour cumulative curve for capacity planning.

## Modal peak hour: where in wall-clock time the burst lives

The mean peak-share answers "how spiky," but operators also need to answer "spiky *when*." The modal peak hour is the most-frequently-winning clock-hour across the group's days, and the dominance ratio is how strong that mode is.

A model with modal peak hour `15` (UTC) and dominance ratio `0.85` has a strong daily 3pm UTC peak — 85% of days, 3pm holds the daily max. You can plan rate-limit headroom against 3pm. You can pre-warm caches at 2:55pm. You can schedule low-priority work for the off-peak windows.

A model with modal peak hour `09` and dominance ratio `0.20` has no real mode — its peak hour is scattered, and the "modal" hour just happens to be the most common in a noisy distribution. You cannot plan against a specific clock-time for this model; you have to plan against the worst-case rolling-hour budget.

The dominance ratio is what tells you which case you're in. Modes only matter when they're tight.

## Why per-day, not per-week

A common mistake is to compute peak-share over a longer window — "what fraction of this week's tokens hit in the single busiest hour of the week?" That number is always lower (because more hours dilute the denominator) and tells you nothing useful. Provider rate limits do not stretch across days. A one-hour burst that uses 95% of the daily TPM (tokens-per-minute) budget has no relationship to whether it uses 4% of the weekly budget — the rate-limit episode happens or doesn't happen entirely within that one hour.

Compute peak-share per day, then aggregate the per-day shares as a distribution (mean, p50, p95, max). The p95 is your rate-limit planning target. The max is your "what's the worst day we've ever had" sanity bound. The mean is the headline.

## Singleton-hour days

Edge case worth being explicit about: a day where the model only ran in one hour scores peak-share = 1.0 by definition. These are not pathologically spiky — they're just days where the model did one thing once. Keep them in the default view (they reflect real operator behaviour: "I fired off one batch today and went home"), but provide a `--min-active-hours <n>` flag that filters them out when you want to compare *true* spikiness across models with different baseline activity. The pew-insights v0.4.42 implementation notes call this out: "Singleton-hour days score 1.0 by definition and are kept by default; `--min-active-hours` scopes to multi-hour days when comparing 'true' spikiness across models with different baseline activity."

This is the same design pattern as the `--min-active-weekdays` flag added in v0.4.44 for HHI computation: keep all data in the global denominator, but allow ranked-comparison views to filter trivially-extreme rows. The principle is "a metric that flags something as concentrated should be reflecting workload structure, not corpus shape."

## Rate-limit headroom: the actual decision

Here is how the metric drives a concrete decision. Suppose your provider gives you 1M TPM (tokens per minute) of capacity. That is `60M` tokens-per-hour budget at sustained peak. If your daily total for one model is `120M` tokens and its peak-hour-share is 25% (smooth, opus-like), then peak-hour usage is `120M × 0.25 = 30M` tokens — 50% of the hourly cap. You have 50% headroom. You can scale this workload 2x before peak-hour starts brushing the limit.

If your daily total is the same `120M` tokens but peak-hour-share is 88% (spiky, haiku-like), then peak-hour usage is `120M × 0.88 = 105.6M` tokens — 176% of the hourly cap. You are *already* exceeding the limit during the peak hour and either getting throttled, getting silently degraded, or paying for burst credits you didn't budget for. You cannot scale this workload at all without re-shaping it.

Same daily total. Same model. Wildly different operational verdict. The peak-hour-share is what told you which case you're in. Total tokens did not. Daily average TPM did not. Only the peak-hour view did.

## What to alert on

Three alert rules that work in practice:

1. **Peak-hour-share regression.** Alert when a group's daily peak-hour-share crosses its 30-day p95. This catches "today is much spikier than usual" — a new burst pattern, a runaway loop, a stuck retry storm. Use p95 not max as the threshold so a single anomalous historical day doesn't desensitise the alert.

2. **Modal-hour drift.** Alert when a group's modal peak hour shifts by more than 2 hours week-over-week and the dominance ratio is above 0.5 (i.e., the mode was real). This catches "the workload is moving in time" — usually a scheduled job's cron changed, or a cache is now expiring at a different time and forcing a refill burst.

3. **Headroom alert.** Compute `peak_hour_tokens / hourly_provider_cap` daily and alert when it crosses 0.7. This is your "we're going to start hitting the rate limit soon if growth continues" early warning. By the time you cross 1.0 you're already shedding work; 0.7 gives you a runway to either negotiate higher capacity or shape the workload.

## The summary

Daily totals are for finance reports. Per-day peak-hour-share is for capacity planning. They are not the same metric and they answer different questions. Add a top-K-hours-by-mass curve to distinguish "tight burst" from "rolling campaign," add modal peak hour and dominance ratio to know *when* in wall-clock the burst lives, and you have a rate-limit telemetry view that actually corresponds to how providers think about capacity. Without it, you are flying blind into a TPM ceiling you can't see until you've already hit it.
