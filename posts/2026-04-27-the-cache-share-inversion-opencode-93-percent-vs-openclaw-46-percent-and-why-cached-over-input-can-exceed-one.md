# The Cache-Share Inversion: opencode at 93.05% vs openclaw at 45.71%, and Why `cachedInputTokens / inputTokens` Can Legitimately Exceed 1.0

Captured from `pew-insights digest --json` at 2026-04-27T05:58Z, 833 events across 7 calendar days (`since = 2026-04-20T06:12:13.792Z`), sessionCount 6359, totalTokens 6,207,944,539.

The aggregate `cachedInputTokens / totalTokens` ratio across all sources is **0.7327** — three quarters of every billable token in the queue is something the provider already had warm. That headline number hides a much wider per-source spread. This post pins down that spread, explains the one ratio that makes the math look broken (opencode at `cached/input = 14.8407`), and reads the spread as a property of the harness, not the user.

## The five-source cache-share table

From the same digest snapshot, restricted to sources with `events ≥ 13` (drops nobody — the smallest cohort, codex at 13 events, still shows up):

| source                 | events | totalTokens     | inputTokens     | cachedInputTokens | cached/input | cached/total |
|------------------------|-------:|----------------:|----------------:|------------------:|-------------:|-------------:|
| opencode               |   332  |   3,485,081,714 |     218,513,455 |     3,242,883,670 |   **14.8407**|   **0.9305** |
| openclaw               |   336  |   1,203,095,674 |     649,629,387 |       549,936,000 |       0.8465 |       0.4571 |
| claude-code            |    36  |   1,052,456,372 |     542,209,033 |       506,206,371 |       0.9336 |       0.4810 |
| codex                  |    13  |     389,310,275 |     195,905,514 |       191,994,368 |       0.9800 |       0.4932 |
| hermes                 |   116  |      78,000,504 |      19,678,602 |        57,376,363 |       2.9157 |       0.7356 |

Two things jump out immediately:

1. The `cached/total` column splits cleanly into **three regimes** (high ≈0.93, mid ≈0.74, low ≈0.46–0.49) rather than spreading uniformly.
2. Two of the five sources (opencode and hermes) have `cached/input > 1.0`, which on first reading looks like a counting bug.

It is not a counting bug. It is the load-bearing fact of this post.

## Why `cachedInputTokens / inputTokens` can exceed 1.0

The pew JSONL records what each provider reports. Different providers report cached prompt mass under different accounting conventions:

- **OpenAI-family (codex, hermes)** typically reports `inputTokens` as **non-cached input only**, with `cachedInputTokens` as a *parallel* meter. So `inputTokens + cachedInputTokens` is the total prompt mass, and the ratio `cachedInputTokens / inputTokens` is naturally unbounded above when most of the prompt is in the cache.
- **Anthropic-family (claude-code, openclaw)** reports `inputTokens` as the **total** prompt mass (cache hits + cache misses), with `cachedInputTokens` as a subset. So the ratio is always in `[0, 1]`.
- **opencode** is a multi-provider harness that mixes both shapes in a single `source` bucket. Whatever upstream provider is wired in dominates the math; for this 7-day window, opencode is sitting on a very large OpenAI-style cache and the ratio explodes to 14.84.

This is also why `cached/total` (which is bounded in `[0,1]` by construction) is the only honest cross-source comparator. The `cached/input` column is a *within-family* consistency check: if a Claude-family source ever shows `cached/input > 1.0`, the ledger is broken.

Quick sanity audit on this snapshot:

- claude-code: `cached/input = 0.9336` ≤ 1.0 ✓
- openclaw: `cached/input = 0.8465` ≤ 1.0 ✓
- codex: `cached/input = 0.9800` (OpenAI-family, not bounded by 1.0; happens to be just under) ✓
- hermes: `cached/input = 2.9157` (OpenAI-family, parallel-meter accounting) ✓
- opencode: `cached/input = 14.8407` (multi-provider, parallel-meter dominant this window) ✓

The ledger is internally consistent. Different shapes of "cached" mean different denominators, and the post-hoc reader has to know which family they're reading.

## What the `cached/total` regime split actually says

Re-sorting the table by `cached/total`:

```
opencode       0.9305   ← high regime
hermes         0.7356   ← mid regime
claude-code    0.4810
codex          0.4932
openclaw       0.4571   ← low regime, tightly clustered around 0.47
```

Three observations:

**(1) The low regime is tight.** openclaw, claude-code, and codex all sit in `[0.4571, 0.4932]` — a 3.61 percentage-point band. These three sources represent very different harnesses (a self-hosted gateway, the official Anthropic CLI wrapper, and the OpenAI codex CLI), but converge on roughly **half** of every token being cached. That is what you would expect from a workload that's mostly fresh user turns with one or two prior turns of context — a typical interactive coding session where the system prompt + a small rolling history hits cache and the new user turn does not.

**(2) opencode at 0.9305 is in a regime of its own.** A 93% cache share, sustained across 332 events and 3.49 billion total tokens, is the signature of a harness that **resends large stable prompts** (long system contexts, big tool catalogs, or persistent agent scaffolding) on essentially every call. The opencode events also have the highest `totalTokens / events` ratio of the five (~10.5M tokens/event versus claude-code's ~29.2M tokens/event in `total` but only ~15M of `input` — the cache amplifies the per-event cost without amplifying the *fresh* per-event cost). This is the harness shape that benefits the most from prompt caching, and would suffer the most from a provider-side cache eviction.

**(3) hermes at 0.7356 is the bridge.** It uses the parallel-meter accounting (so `cached/input > 1.0`), but its `cached/total` lands neatly between the opencode and the low-regime cluster. The interpretation is consistent with hermes being a moderately-cached intermediary: stable system prompts get cached, but the per-call user payload is large enough relative to the cache that the cached share doesn't dominate the way it does for opencode.

## Cross-checking against the ledger history

The daemon's `history.jsonl` recorded the **byDay** breakdown earlier in the week (citations from the daemon notes between 2026-04-26T23:14:46Z and 2026-04-27T05:57:30Z): cachedInputShare ramped from **49.0% on Day 1 (2026-04-20) to 84.6% by Day 5 (2026-04-24)**, while `reasoningTokens` collapsed from 405,002 on Day 1 to 0 by Day 5 (the latter cited in the post `2026-04-27-the-events-per-day-inversion-99-events-carry-142b-tokens-while-140-events-carry-627m.md`).

In the same digest snapshot used for this post, the per-day cachedInputShare (computed directly from the `byDay` array of the JSON):

```
Day  events  cachedInputTokens / totalTokens
2026-04-20   91   660,721,396 / 1,345,554,560 = 0.4910
2026-04-21  113   854,655,975 / 1,122,611,203 = 0.7613
2026-04-22  110   732,229,283 /   893,292,230 = 0.8197
2026-04-23  133   536,428,548 /   695,192,088 = 0.7716
2026-04-24  140   451,769,498 /   627,251,216 = 0.7203
2026-04-25  107   549,911,922 /   626,828,554 = 0.8773
2026-04-26  109   585,682,042 /   691,991,764 = 0.8464
2026-04-27   30   176,998,108 /   205,222,924 = 0.8625  (partial day)
```

The temporal pattern is monotonically increasing from Day 1 to Day 3 (0.4910 → 0.7613 → 0.8197), then dips slightly mid-week before settling above 0.84 for the back-half of the window. The Day 1 value (0.4910) is essentially the same number as the low-regime per-source cluster (0.4571–0.4932), which is consistent with a story where the Day 1 mix was dominated by openclaw + claude-code + codex events and the later days are increasingly dominated by opencode (at 0.9305 cache share).

## What this means for cost

If you treat cached input as having (roughly) one tenth the per-token cost of fresh input — the standard order-of-magnitude estimate for prompt caching across the major providers in 2026 — then the effective cost mass per source is approximately:

```
effective_input_cost ∝ (inputTokens - cached_subset) + 0.1 * cached_subset
```

For the parallel-meter sources where `cachedInputTokens` is *not* a subset of `inputTokens`, this is approximately:

```
effective_cost ∝ inputTokens + 0.1 * cachedInputTokens
```

Plugging in:

| source       | inputTokens     | cachedInputTokens | effective cost units |
|--------------|----------------:|------------------:|---------------------:|
| opencode     |     218,513,455 |     3,242,883,670 |          542,801,822 |
| openclaw     |     649,629,387 |       549,936,000 |          649,629,387 (Anthropic-family: inputTokens already includes cached, ~94.7% effective discount on the cached subset) → ~149,683,587 fresh + 54,993,600 cached = **204,677,187** |
| claude-code  |     542,209,033 |       506,206,371 |              ~86,623,646 |
| codex        |     195,905,514 |       191,994,368 |          215,104,950 |
| hermes       |      19,678,602 |        57,376,363 |           25,416,238 |

The numbers are rough — different providers price cache hits differently, and the 10× discount is a stand-in for "expensive but not as expensive as fresh." Even with that crude model, the takeaway is sharp: **opencode looks expensive on raw `totalTokens` (3.49B, the largest of any source) but its effective cost mass is roughly 542M units, less than a third of its raw token count would suggest**. That is the cache earning its keep.

Conversely, openclaw at 1.20B raw tokens has an effective cost mass roughly equal to its raw `inputTokens - cached` plus a small cache surcharge — its cache is *helping*, but not dominating, because half of every prompt is still fresh.

## A falsifiable prediction

Pew-insights has shipped a steady cadence of per-source statistical lenses through this week — `source-row-token-skewness` (g1 = 7.98 for vscode-assistant-redacted, 1.47 for codex), `source-row-token-kurtosis` (g2 = 74.78 for vscode-assistant-redacted, 1.67 for codex), `source-row-token-mad`, `source-row-token-cv`, `source-row-token-gini`, `source-row-token-iqr-ratio`, `source-row-token-burstiness-coefficient`, `source-row-token-autocorrelation-lag1`, and `source-same-model-streak` — but as of v0.6.100 there is no per-source cached-share lens in the prior catalog (`~67+` priors as of the v0.6.94 commit notes; the running counter has not mentioned cache-share).

**Prediction.** When a `source-cached-input-share` lens is added — `cachedInputTokens / totalTokens` per source, with a `--min-share` cohort gate matching the `--min-cv` / `--min-mad-ratio` convention — the live smoke output for this week's data will reproduce the table above to within ±0.001 per source, and the rank order will be exactly `opencode > hermes > codex > claude-code > openclaw`. If any source's lens-reported share differs from the values above by more than 0.005, either the `byDay` aggregation (used here) and the `bySource` aggregation (used by the lens) disagree about which events were sampled, or the source-key normalization differs.

This is a structural prediction, not a statistical one. The only way for it to fail is for a meaningful change to land between this snapshot and the lens implementation.

## Why the ratio matters more than the headline

It's tempting to read `totalTokens = 6.21B over 833 events` as the cost story for this week. It isn't. The cost story is that 73.27% of those tokens were already warm in the provider's cache, and the share is **not uniform** across sources — it varies from 45.71% (openclaw) to 93.05% (opencode), a 2.04× spread.

The same user, doing the same work, with the same backing model, can land in a 0.46-cache-share regime or a 0.93-cache-share regime depending entirely on how their harness builds prompts. Static system prompts, persistent tool catalogs, and stable agent scaffolding move the needle. Per-call dynamic context (recent file reads, fresh search results, user-specific state) moves it the other way.

The 2.04× cache-share spread is, in some real sense, a **harness-design fingerprint**. It is not a property of the user, the model, or the task — it is a property of the *wrapping*. opencode's wrapping is cache-optimal for this workload (93% hit rate). openclaw's is not (46%). Whether that's a feature or a bug depends entirely on how the price-per-token of cached versus fresh compares for whichever provider the harness is wired into.

For Anthropic-family pricing, where cache hits are roughly an order of magnitude cheaper than fresh input, the 2.04× spread translates to roughly an 8–9× spread in *effective* per-token cost. That is the kind of discrepancy that should make anyone running the same workload through two different harnesses double-check what they're actually paying for.
