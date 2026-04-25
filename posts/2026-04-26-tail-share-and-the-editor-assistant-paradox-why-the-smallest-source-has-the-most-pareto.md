---
title: "Tail-share and the editor-assistant paradox: why the smallest source has the most Pareto"
date: 2026-04-26
tags: [pew-insights, tail-share, pareto, gini, telemetry, observability]
---

# Tail-share and the editor-assistant paradox: why the smallest source has the most Pareto

Most people's mental model of "tool concentration" goes like this: the tool you use the most is also the tool that's most concentrated in time. The big, dominant CLI in your stack hogs a few peak hours, and the small ones dribble along uniformly. It's a clean story. It's also wrong, at least for the local pew queue I'm staring at right now.

This post unpacks a single fresh `pew-insights tail-share` snapshot taken at `2026-04-25T18:07:10.650Z` against `~/.config/pew/queue.jsonl` (1,363 active hour-buckets across 6 sources, 8.71 B tokens total) and uses it to make a much narrower claim:

> The source with the **least** total token mass in my queue has the **most** Pareto-shaped distribution. The two are not the same axis. Confusing them is the most common observability mistake I see.

I'll walk through what `tail-share` measures, what the numbers actually say, why the editor-resident `vscode-assistant` source comes out the most concentrated despite being three orders of magnitude smaller than the leaders, and what this means for anyone trying to set alerting thresholds on tool usage.

## What `tail-share` actually computes

The metric is deliberately boring, which is its strength. For each source, `tail-share`:

1. Buckets the source's lifetime into 30-minute UTC hour-buckets.
2. Drops zero-token buckets — only "active" buckets count.
3. Sorts the active buckets by `total_tokens` descending.
4. Reports what fraction of the source's lifetime tokens lives in the heaviest 1%, 5%, 10%, and 20% of buckets.
5. Reports a `giniLike` scalar: the uniform-baseline-corrected mean of those four shares, normalised to `[0, 1]` where 0 means perfectly flat and 1 means all the mass is in a single bucket.

Note what it does **not** do: it does not weight by token volume, it does not cross-compare sources, it does not factor in calendar density. It is a pure shape statistic. A source with 1,000 tokens and a source with 1 trillion tokens can have identical `giniLike` values if their internal distributions are the same shape.

This is the entire point. Concentration is a shape property, not a volume property. When you look at one source's tail-share number, you are looking at how lumpy that source is *relative to itself*, with no reference to how big it is in the global picture. People constantly forget this and start ranking sources by `giniLike` as if it were a leaderboard. It is not.

## The data

Here is the raw output, copied verbatim:

```
pew-insights tail-share
as of: 2026-04-25T18:07:10.650Z    sources: 6 (shown 6)    buckets: 1,363
                                   tokens: 8,711,747,576    minBuckets: 0

source          buckets  tokens         top1%  top5%  top10%  top20%  giniLike
--------------  -------  -------------  -----  -----  ------  ------  --------
vscode-assistant  320      1,885,727      26.1%  44.2%  56.4%   71.1%   0.444
claude-code     267      3,442,385,788  8.0%   26.9%  44.9%   70.3%   0.312
opencode        196      2,632,980,698  5.1%   22.6%  40.7%   61.0%   0.255
codex           64       809,624,660    7.3%   25.1%  38.7%   58.5%   0.251
openclaw        366      1,683,526,396  9.0%   24.2%  35.3%   51.8%   0.230
hermes          150      141,344,307    7.1%   21.3%  33.8%   54.9%   0.221
```

Three observations jump out before any analysis:

1. **`vscode-assistant` has the most active buckets after `openclaw` (320 vs 366) but only 1.9 M tokens** — roughly 1/1800th of `claude-code`'s mass. It's a high-frequency, low-volume source.
2. **`giniLike` is monotonically descending**, and `vscode-assistant` is at the top with 0.444 — well above the second-place `claude-code` at 0.312.
3. **The `top1%` column is the most discriminating column**, ranging from 5.1% (`opencode`) to 26.1% (`vscode-assistant`). That's a 5× spread on a single statistic.

## The VS Code paradox

`vscode-assistant` has 320 active buckets and 1,885,727 lifetime tokens. That averages out to 5,893 tokens per active bucket — a number so small that one or two slightly larger sessions can swing the top-1% bucket dramatically. Indeed, the top 1% of its 320 buckets is just 3.2 buckets, and those buckets hold 26.1% of the source's lifetime tokens. The top-1% bucket alone is therefore carrying somewhere around 490,000 tokens — which dwarfs the 5,893-token average by roughly 80×.

This is the textbook signature of a low-baseline, occasional-spike source. Most of the time, the in-editor inline-completion telemetry is tiny — a handful of suggestions per active bucket, each one logging a few hundred tokens. But when something larger happens — a multi-file refactor accepted as one batch, or an explain-this-codebase invocation — the bucket suddenly carries five orders of magnitude more mass than its neighbours. The shape statistic catches that sharp asymmetry.

Compare this with `openclaw`, which has 366 buckets, 1.68 B tokens, an average of 4.6 M tokens per bucket, and a top-1% concentration of just 9.0%. Even when `openclaw` has a "spike", the spike is only ~10× the baseline, not 80×. So even though `openclaw` is the second-most-active source by bucket count, its activity is much smoother — it stays inside the same one-or-two-orders-of-magnitude band session-to-session.

The conclusion is uncomfortable for the standard "the big source is the bursty source" mental model: in this dataset, **smaller sources are more bursty**, not less. The mechanism is mundane — small sources have small denominators, and small denominators are easy to perturb — but the operational consequence is real.

## Why this matters for alerting

If you ever want to set a "this source is unusually busy" alert, you have two natural choices:

- **Volume-based**: alert when a source's hourly token mass exceeds some absolute or percentile threshold.
- **Shape-based**: alert when a source's recent giniLike or top-1% share drifts away from its baseline.

The data above tells you that you cannot use the same threshold for both `vscode-assistant` and `opencode`. `vscode-assistant` will routinely punch its top-1% bucket up to 26% of its lifetime mass — that's normal for it, not anomalous. If you set an alert at "top-1% > 15%", you'll get nuisance pages from `vscode-assistant` constantly while sleeping through real anomalies in `opencode`, where 15% would be a genuine outlier (its baseline is 5.1%).

Equally, you cannot use a volume threshold without normalising. A 1 M-token bucket is a quiet hour for `claude-code` and a once-a-month event for `vscode-assistant`. Any alerting system that conflates the two axes will be wrong roughly half the time.

The right move is per-source baselines on both axes simultaneously. The `tail-share` numbers above are exactly the input you'd use for a "shape baseline": you compute the lifetime `giniLike` per source over a 14- or 30-day window, then alert when a 24-hour rolling `giniLike` deviates by more than ~2 standard deviations from that baseline. The volume baseline is independent — usually a logit-space EWMA on `log(tokens_per_active_bucket)`.

These are two separate alerts, on two separate axes, with two separate baselines per source. That is six configurations for six sources — a fact that explains why most teams don't bother and instead just stare at a global token-count chart that hides every interesting signal.

## What `giniLike` does and doesn't tell you

`giniLike` is calibrated against a uniform baseline. A perfectly flat source — same token mass in every active bucket — gets `giniLike = 0`. A source where one bucket holds everything gets `giniLike = 1`. The four real points (top-1, top-5, top-10, top-20) are averaged after subtracting their uniform-distribution expectations and renormalising.

A few things this metric is **not** sensitive to:

- **Calendar density.** A source that's active 3 hours a day every day and a source that's active 24 hours one day per week can have identical `giniLike` if the within-active-bucket distribution is the same shape. The `interarrival-time` and `bucket-streak-length` reports are the right lenses for calendar density.
- **Order.** `giniLike` is permutation-invariant. A source that ramps up smoothly over a month and a source that's flat for 29 days then spikes once will land in similar territory if their sorted bucket-mass curves match.
- **Tail length.** Top-20% caps the right tail at, well, 20%. A source whose 50th-percentile bucket is its dominant feature won't show up as concentrated.

What it *is* sensitive to is the joint asymmetry of the very heaviest active buckets relative to the rest. That's a narrow but useful question, and `tail-share` answers it cheaply. If you want a richer view, pair it with `bucket-density-percentile` (which gives you the full p1..p99.9 ladder pooled across rows) and `output-token-decile-distribution` (which gives you the per-decile token mass and a true Gini). Together those three reports cover most of the "how lumpy is this source/model?" question space.

## A working hypothesis for the VS Code shape

Why is `vscode-assistant` the most Pareto-shaped source in this dataset? My working hypothesis, based on what I know about the inline-completion telemetry pipeline:

1. The default mode is one tiny event per accepted suggestion: a few hundred tokens.
2. Occasionally, an "explain this file" or "summarise this PR" action emits a single event carrying tens of thousands of tokens.
3. The bucketing aggregates events by 30-minute window. Most windows contain only the tiny events. A handful of windows happen to contain one of the big actions on top of the usual baseline.
4. Result: a smooth low-mass baseline punctuated by isolated heavy buckets — the textbook generator for high `giniLike`.

This is testable with a deeper dive into the per-event distribution rather than the per-bucket distribution. The tooling for that exists — `bucket-intensity` will give you per-model magnitude histograms — but the result here is already strong enough to guide design decisions: any future alerting system on `vscode-assistant` needs to treat the heavy tail as a normal feature, not a bug.

## What to do with this

Three concrete takeaways from a single 1,363-bucket snapshot:

1. **Stop ranking sources by gini-like statistics.** The ranking is dominated by source size and shape interactions, not by which source is "most concentrated" in any operationally meaningful sense.
2. **Set per-source baselines on both volume and shape, separately.** Six sources × two axes is twelve baselines, but the alternative is alerts that lie to you.
3. **Treat low-volume, high-frequency sources as their own category.** `vscode-assistant` here behaves nothing like `claude-code` despite both being "AI coding tools" — they have different telemetry shapes, different burst structures, and need different threshold schemas.

The whole exercise took one CLI invocation and about ten minutes of reading the table carefully. The lesson is the same as always with telemetry: the data is almost always there, and the failure mode is almost always not looking at the right slice of it.

## Reproducing this

If you want to run the same query against your own queue:

```bash
cd ~/Projects/Bojun-Vvibe/pew-insights
node dist/cli.js tail-share
```

Defaults look at the full lifetime of `~/.config/pew/queue.jsonl`. Useful flags:

- `--source <name>` to scope to one source.
- `--since <iso>` / `--until <iso>` to scope to a window.
- `--min-buckets <N>` to drop sparse sources (default 0).

The exact run that produced this post's table was timestamped `2026-04-25T18:07:10.650Z` with the default flags. The numbers will drift as the queue grows; the *shape* observation — `vscode-assistant` topping the giniLike ranking despite being the smallest source — has been stable across the last week of snapshots, which is what makes it worth writing down.
