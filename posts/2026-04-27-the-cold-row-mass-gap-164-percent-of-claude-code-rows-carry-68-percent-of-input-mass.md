# The Cold-Row Mass Gap: 16.4% of claude-code Rows Carry 6.8% of Input Mass and What That Says About Cache-Miss Geometry

**Run:** `pew-insights source-cold-warm-row-ratio --json ~/.config/pew/queue.jsonl`
**Generated at:** 2026-04-26T23:06:20.895Z
**Corpus:** 6 sources, 9.47B total tokens, 327 rows dropped for non-positive input

---

There is a metric in this corpus that is *only* legible if you split rows by whether they hit the prompt cache or missed it, and that the global cache-hit ratio collapses to garbage. It is the gap between *how many of your rows were cold* and *how much of your input-token mass those cold rows carried*. Call it the **cold-row mass gap**: `coldShare − coldInputTokenShare`. The latest pew-insights snapshot gives the per-source values directly:

| source            | rows | cold rows | coldShare | coldInputTokenShare | mass gap | mean cold input | mean warm input |
|-------------------|------|-----------|-----------|----------------------|----------|------------------|------------------|
| claude-code       | 299  |   49      | 0.16388   | 0.06849              | +0.0954  | 2,564,318        | 6,835,848        |
| codex             |  64  |    2      | 0.03125   | 0.00027              | +0.0310  |   111,212        | 6,623,709 (warm) |
| opencode          | 318  |    4      | 0.01258   | 0.00266              | +0.0099  |   139,412        |   666,436        |
| openclaw          | 424  |    4      | 0.00943   | 0.00104              | +0.0084  |   248,403        | 2,270,703        |

(`vscode-assistant-A` and `hermes` rows from the same subcommand are omitted from the table for brevity but follow the same direction.)

Every source in this corpus has a *positive* cold-row mass gap. That means: in every tool, the cold rows are *systematically smaller* than the warm rows, in input-token mass per row. The gap is the row-share of cold calls minus the input-token-mass-share of cold calls, and across the entire stack here the cold rows carry a smaller fraction of the bill than they do of the call count.

This is not the obvious finding. The obvious finding would be that big calls miss the cache (because they have unique gigantic contexts that nothing matched) and small calls hit it (because they re-use the same template prompt). The data says the *opposite*: the cold calls are the small ones, and the big calls are warm.

That is structurally important.

## Why the cold rows are small in this corpus

`claude-code` has the cleanest signal. 299 rows, 49 of them cold (no `cached_input_tokens`), 250 warm. 16.4% of rows are cold but only 6.8% of input mass is cold. The mass gap is +0.095 — about 9.5 percentage points of "cold rows are lighter than average". The mean cold input is 2.56M tokens; the mean warm input is 6.84M tokens. Warm rows are ~2.67× heavier on the input axis than cold rows.

Why? Because in modern coding-assistant harnesses, the cache hits *are* the long-running sessions. Every turn after turn 1 in a `claude-code` conversation is fed roughly the same accumulated context with a small delta — and that accumulated context is exactly what gets cached. By turn 5 of a session, the prompt is a 5-turn-deep replay; almost all of the input tokens are warm and a small tail at the end are the new turn. By turn 30, the prompt may be 30 turns deep and *almost all* of those input tokens are warm.

The cold rows, then, are the *short* sessions: a one-turn invocation that never built up reusable context. Or the *first* turn of a session, before there is anything to cache. Both of those classes have much smaller input than the long-running, cache-saturated sessions, because they have not had time to accumulate replay context. So cold ↔ short; warm ↔ long. And that is exactly what the +0.095 mass gap reads as.

`codex` makes the same point with even more extreme numbers. 64 rows, only 2 cold. The cold mass share is *0.027%* — those 2 cold rows carry essentially zero of the input bill. The mean cold input is 111K tokens; the mean warm input is in the multi-million-token range. The two cold `codex` calls in this corpus are tiny by `codex`'s own standards, and the 62 warm calls are the giant repo-context turns that define the tool's normal mode.

`opencode` and `openclaw` agree, with smaller absolute numbers. `opencode`: 4 cold rows out of 318, `coldShare = 0.0126`, `coldInputTokenShare = 0.0027`, gap +0.0099. `openclaw`: 4 cold out of 424, `coldShare = 0.0094`, `coldInputTokenShare = 0.0010`, gap +0.0084. Both have mean cold input roughly 5× smaller than mean warm input.

The pattern is uniform: across these four sources, cold rows are a small minority by count *and* a much smaller minority by mass. The mass gap is positive everywhere.

## The misleading global statistic this exposes

If you ran `cache-hit-ratio` globally on this corpus you would get a single number — the fraction of total input tokens that came from the cache. That number is mass-weighted by construction. Because the warm rows dominate the input mass (often by 90%+), the global cache-hit ratio looks excellent on every source: roughly 1 − coldInputTokenShare. For `claude-code` that is 1 − 0.0685 = 93.2% mass-weighted cache hit. For `opencode` it is 99.7%. For `codex` it is 99.97%.

Those numbers are correct. They are also *not what an operator would notice*. What the operator notices is the *experience* of cold turns — the latency, the cost spike, the eviction. And the experience axis is not mass-weighted, it is row-weighted. Row-weighted, `claude-code` has 16.4% cold rows. One out of every six turns is a cold-prompt experience. That feels very different from the 6.8% mass-weighted number.

The mass gap names this discrepancy as a single scalar. Whenever it is large and positive (>0.05 or so), the global cache-hit ratio is materially understating how often the operator actually hits a cold turn. Whenever it is near zero, mass and rows agree and either statistic gives the same picture. Whenever it is *negative* — the rare case where cold rows are bigger than warm rows — you have an inverted regime: the big ad-hoc one-shot calls miss, the small repeated calls hit. None of the sources in this corpus are in the negative regime; all of them are in the warm-is-bigger regime.

## Why the directionality matters for cache-design decisions

Two practical consequences fall out of "cold rows are systematically smaller than warm rows":

**(1) The marginal cost of a cache miss is bounded.** If your cold rows are ~2.5× smaller than your warm rows on input, missing the cache on a cold row costs less in absolute tokens than the warm-row baseline. In `claude-code`'s case the mean cold input is 2.56M tokens vs 6.84M for warm. A cache miss on a typical cold call is a 2.56M-token full-input bill — which is large in absolute terms but smaller than the typical cached-input number. The cache geometry is doing what it's supposed to: the most expensive prompts are the ones being deduplicated.

**(2) Cache-policy tuning has asymmetric upside.** Improving the hit rate on warm rows (e.g., by extending TTL, increasing cache size, or using semantic-key cache extensions) gives you leverage on the rows that already carry most of the input mass. Improving the hit rate on cold rows (e.g., by warming the cache with template prompts before first turn) buys you a smaller absolute reduction because the cold rows are individually smaller. If you have 10 hours of cache-engineering time, the warm-side investment dominates.

The mass gap quantifies the asymmetry. A source with mass gap near zero has no asymmetric leverage — cold and warm rows are roughly the same size, so optimizing either axis has comparable payoff. A source with mass gap +0.10 (close to `claude-code`) has the cold/warm asymmetry baked in, and the warm-side investment is roughly 2-3× more efficient per cache-engineering hour.

## What "cold" actually means in this dataset

The subcommand defines a row as cold when `cached_input_tokens === 0` *and* `input_tokens > 0`. The second condition is the filter that drops 327 rows from the analysis (per the report's `droppedNonPositiveInput: 327` count) — these are rows where the model was invoked with no input-side cache slot exposed at all, either because the provider does not report cache stats for that model or because the row is a degenerate metadata-only event. They cannot be classified cold or warm and are correctly excluded.

A row is warm when `cached_input_tokens > 0` *and* `input_tokens > 0`. Note that this is a binary classification, not a continuous one — a row with cache hit ratio of 0.05 is still "warm" by this definition, even though most of its input came from the live path. If you want the continuous picture, the `cache-hit-ratio` subcommand gives you per-row hit ratios. The cold/warm split exists specifically because *some* operator decisions (latency budgets, eviction expectations, "did this turn pay the full prompt cost") are binary in nature: either the cache hit at all, or it did not.

## The dataset boundaries

`firstDay` and `lastDay` per source vary. `claude-code` spans `2026-02-11` → `2026-04-23` (71 days of activity). `opencode` is much more recent: `2026-04-20` → `2026-04-26` (7 days). `openclaw`: `2026-04-17` → `2026-04-26` (10 days). `codex` does not report firstDay/lastDay in the trimmed table above but spans roughly the full corpus window.

The point: the cold/warm regime described above is *stable across time windows of very different lengths*. The 71-day `claude-code` window and the 7-day `opencode` window both produce the same directional finding: cold rows are systematically lighter than warm rows. This is not an artifact of one bad week or one anomalous session. It is a structural property of how these tools use their context windows — the cache hits when there is something to cache, the cache misses when the operator is doing something fresh and small, and "fresh and small" is the dominant cold-row geometry.

## A note on the totals

`totalTokens` in the report is 9,468,474,722 across the six sources, of which `claude-code` alone contributes 3.44B and `opencode` 3.29B. These two sources between them are >70% of the total token mass in the corpus. Their cold/warm splits, then, dominate any mass-weighted statistic. The operator-feel statistics (row-weighted) tell a different story because `vscode-assistant-A` (333 rows) and `openclaw` (424 rows) carry a large fraction of *rows* despite being a smaller fraction of *mass*. The mass gap is exactly the bridge between those two views.

## Summary

The cold-row mass gap is a one-number scalar that asks: *do my cache misses cluster on the small calls or on the big ones?* Across all six sources in this corpus, the answer is **on the small calls**. The largest gap is `claude-code` at +0.095 — about 16% of rows are cold but only 7% of input mass is. The geometry is consistent everywhere: cold ↔ short, warm ↔ long, and the warm rows carry almost all of the input bill.

This is the right shape for a healthy cache, and the wrong shape for a cache that's trying to deduplicate one-off ad-hoc prompts. If you want to know which kind of cache regime your stack is operating in, the cold-row mass gap is the cleanest single statistic to look at. The global hit ratio will tell you the cache is "working"; the mass gap will tell you *what it is working on*.
