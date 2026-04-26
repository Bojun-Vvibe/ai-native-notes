---
title: "Cold-row vs cold-mass gap: codex's 119x ratio between mean cold and mean warm input, and the ide-assistant-A 100% cold floor"
date: 2026-04-26
tags: [pew-insights, cache, cold-warm, source-cold-warm-row-ratio, telemetry]
data_source: ~/.config/pew/queue.jsonl via pew-insights v0.5.6 source-cold-warm-row-ratio
generated_at: 2026-04-26T18:23:01.480Z
window_total_tokens: 9,351,749,311
---

## Two ways to count "cold"

Every cache-effectiveness lens I have seen on the dashboard collapses to a single ratio per source — `cached_input_tokens / input_tokens`, summed across rows, reported as a percentage. That ratio is mass-weighted. It tells you what fraction of the *input mass* hit the cache. It tells you nothing about what fraction of the *requests* hit the cache, because heavy rows dominate the numerator and denominator.

The `pew-insights source-cold-warm-row-ratio` subcommand splits those two questions apart by reporting, per source: `coldShare` (count of rows with `cached_input_tokens === 0` divided by all rows with positive `input_tokens`), `coldInputTokenShare` (input mass on those cold rows divided by total input mass), and a derived `coldRowMassGap = coldShare - coldInputTokenShare`. A positive gap means cold rows are smaller than the per-source average. A negative gap means the *big jobs* are the ones missing the cache. Zero means cold and warm rows weigh the same on average.

Run against `~/.config/pew/queue.jsonl` on `2026-04-26T18:23:01.480Z` (pew-insights v0.5.6, 9,351,749,311 input-positive tokens after dropping 327 non-positive-input rows), the full breakdown is:

| source           | nRows | cold rows | coldShare | coldInputTokenShare | gap (rows − mass) | mean cold input | mean warm input | warm/cold ratio |
|------------------|------:|----------:|----------:|--------------------:|------------------:|----------------:|----------------:|----------------:|
| claude-code      |   299 |        49 | 16.39%    |  6.85%              | +9.54 pp          | 2,564,317       | 6,835,848       |  2.66×          |
| opencode         |   308 |         3 |  0.97%    |  0.25%              | +0.73 pp          |   170,629       |   672,632       |  3.94×          |
| openclaw         |   414 |         3 |  0.72%    |  0.08%              | +0.64 pp          |   254,060       | 2,291,196       |  9.02×          |
| codex            |    64 |         2 |  3.13%    |  0.03%              | +3.10 pp          |    55,606       | 6,623,709       | **119.1×**      |
| hermes           |   164 |         1 |  0.61%    |  0.03%              | +0.58 pp          |    15,400       |   340,078       | 22.08×          |
| ide-assistant-A  |     6 |         6 | 100.00%   |  100.00%            |   0.00 pp         |    96,848       |     0           |  —              |

Every source with at least one warm row has a *positive* gap. That alone is a finding — there is no source in this corpus where the big jobs are the ones missing the cache. The cache is uniformly hitting the heavy requests. But the *magnitude* of the gap, and especially the warm-to-cold mean ratio, is wildly source-dependent.

## codex's 119x ratio: cache hits are not just larger, they are 119 times larger

The headline number is codex's `meanWarmInput / meanColdInput = 6,623,709 / 55,606 = 119.1×`. That is *not* a typo. On the 64 codex rows in the window, two of them have zero cached input and they average 55,606 tokens of prompt. The other sixty-two have non-zero cached input and they average 6,623,709 tokens of prompt. The cold rows are, on average, just over 0.8% the size of the warm rows.

There are two ways to read that.

The first reading is that **the codex harness packs a giant cached prefix into every steady-state call**. The 6.6M-token mean warm prompt is consistent with this corpus's earlier finding (the "1M+ prompt tier" post from yesterday) that codex routinely sends multi-million-token contexts. If the harness deterministically prepends the same project prefix on every call, the cache will hit on substantially all of it, and `cached_input_tokens` will be in the millions. The two cold rows would then be the *first* call of a session — the cache-priming row — where there is no prior context to replay.

The second reading is that **the codex harness occasionally issues a probe-class request** — a small, one-off prompt unrelated to the heavy session, perhaps a model-info query or a tool-validation probe, with no project context attached. Those rows would naturally be cold (no prior cache key to match) and small (no project prefix to send). Both readings are consistent with a 119× warm-to-cold mean ratio. They differ only in whether the cold row is *the priming step* (causal precursor to warm rows) or *an independent species* (a different request shape entirely).

Either way, the operational implication is the same: **for codex specifically, the row-count cache hit rate (96.9%) is materially less impressive than the mass-weighted cache hit rate (~99.97%)**, because the 3.1% of rows that miss the cache are 119× lighter than the rows that hit it. If you are budgeting on input-token cost, codex looks essentially fully cached. If you are budgeting on per-call latency or per-call rate-limit pressure, you have a 1-in-32 cold-call population that you must size around.

## claude-code's 9.5-percentage-point gap: the largest absolute coldShare in the corpus

claude-code is the only source with a *double-digit* coldShare: 49 of 299 rows (16.39%) had zero cached input. Its coldInputTokenShare is 6.85%, giving a gap of 9.54 percentage points — the largest absolute gap in the corpus. The mean cold input is 2.56M tokens and the mean warm input is 6.84M, a 2.66× warm/cold ratio. So unlike codex's harness-priming-only pattern, claude-code's cold rows are *not* tiny: they are still million-token-scale prompts. They just happen to land outside whatever cache window the provider is using.

This is consistent with the model name aliasing chaos finding from earlier in the week (the "23 strings, 10 models, five-way collision on claude-opus-4.7" post): claude-code routes through several model aliases over its 35-day tenure, and a model switch is one of the few clean ways to invalidate the prompt cache mid-session. A 16.4% cold rate on claude-code is therefore *not* surprising in light of the aliasing data — every model handoff is a cold-start by construction. What is interesting is that the cold rows still average 2.56M input tokens, meaning they are full-context first-calls into the new model rather than minimal probes. The cache misses are *expensive* in absolute terms (49 × 2.56M = 125.6M tokens of cold input, or 6.85% of claude-code's total 1.83B input tokens), even though the gap reading says they are below-average size.

## openclaw's 9× warm/cold ratio with a tiny 0.7% cold rate

openclaw has the second-largest warm/cold ratio in the corpus at 9.02× (mean warm 2.29M vs mean cold 254K), but only 0.72% of rows are cold (3 out of 414). That combination is the cleanest "harness with a deterministic cache-priming step" signature in the dataset:

- Almost every row hits the cache (warmRows = 411, coldRows = 3).
- The cold rows are systematically smaller — about 1/9 the size — so they look like setup or validation calls.
- The mass impact of the misses is microscopic: 0.08% of total input tokens.

If I were measuring openclaw against the codex pattern, I would say openclaw has a *cleaner* harness shape: the misses are concentrated in 3 rows, they are uniformly small, and they cost essentially nothing in mass. codex has 2 misses but they sit underneath a much heavier average warm-row size, and the absolute warm-cold ratio is 13× larger than openclaw's. Different tools, different cache-priming geometries, both visible from the same 5-column report.

## opencode's 0.97% cold rate is the lowest in the corpus

opencode shipped 308 input-positive rows over 7 active calendar days and missed the cache on exactly 3 of them. coldShare = 0.974%, coldInputTokenShare = 0.249%, gap = +0.73 percentage points. This is the steadiest cache profile in the corpus by every metric: lowest row-count miss rate, lowest mass miss rate, smallest absolute gap. The mean warm input is 672K tokens — substantially smaller than codex (6.6M) or claude-code (6.8M) — and the mean cold input is 170K tokens, a warm/cold ratio of just 3.94×. opencode's pattern looks like a tool that streams steady-sized requests and only ever cold-starts on session boundaries.

This dovetails with opencode's CV of 0.469 from the Fano-factor analysis — it is the steadiest source on this dataset by daily-token-volume CV, *and* it is the steadiest by cache-state row distribution. Two orthogonal lenses, same conclusion: opencode is the calmest harness in the queue.

## ide-assistant-A: the 100% cold floor, by construction

ide-assistant-A is, again, the corpus's outlier. All 6 of its input-positive rows are cold. coldShare = 1.0, coldInputTokenShare = 1.0, coldRowMassGap = 0.0 — the gap is mechanically zero because there is no warm denominator to compare to. The mean cold input is 96,848 tokens; the mean warm input is undefined (zero rows). This source has *never*, in the recorded queue window, sent a request that the provider could match against an existing cache key.

That is consistent with what the "cache-input asymmetry" post documented earlier this week (where the same source posted a cache-to-input ratio of exactly 0.000 over a fundamentally larger row sample), and it is the strongest evidence in this dataset that ide-assistant-A's harness either does not opt into prompt caching or sends prompts in a shape that defeats the provider's cache key derivation. Either way, the operator-visible consequence is that **every single ide-assistant-A call costs the full input-token rate**, with no cache discount on any row, ever.

## Why coldRowMassGap is the right index

The naïve cache report on any of these sources is just `cached_input_tokens / input_tokens`. That single ratio rolls up the mass-weighted view and hides the row-count view entirely. With coldRowMassGap split out, I can read:

- `gap > 0` → cold rows are smaller-than-average → the cache is hitting the heavy requests, missing the light ones. **Every source with non-trivial sample size in this corpus looks like this.**
- `gap < 0` → cold rows are larger-than-average → the cache is missing the heavy requests. **No source in this corpus exhibits this, but it would be a five-alarm signal if any did.** A negative gap means the dollar-per-token cost is going to land disproportionately on the misses.
- `gap ≈ 0` → cold and warm rows are the same size on average → the cache state is independent of request size. Either you are getting unlucky uniformly, or your harness has no project-prefix priming pattern.

The coldRowMassGap is, in effect, a row-vs-mass *covariance signal* between cache state and request size. Every source in this corpus reports it positive, which says the prompt-caching layer is working as a "discount on the heavy stuff" — exactly what I would want from it operationally. The size of that discount, expressed as a `meanWarmInput / meanColdInput` ratio, ranges from 2.66× (claude-code) up to 119× (codex), with no obvious clustering. The harness shape, not the model, is the dominant factor.

## What this implies for cost dashboards

If I run a single-line cache effectiveness gauge on the operator dashboard, it should be the mass-weighted ratio (which is what every cost model already computes). But for *anomaly detection*, I now have a better candidate: **the trailing 7-day coldRowMassGap, per source**. A sudden swing from +9 percentage points to –2 on claude-code would mean the heavy jobs have started missing the cache — a cost regression that the mass ratio alone might mask if total volume also dropped.

The follow-up subcommand I would want is `source-cold-warm-row-ratio --rolling 7d` — emit one (coldShare, coldInputTokenShare, gap) per (source, trailing-7-day-window) so I can plot the gap series and trip an alert when it turns negative. The current subcommand reports a single window; a rolling version would turn it into a real monitor. Filing that as the obvious extension while the current scalar reading still gives me the headline numbers I needed for this post.

## Citation footer

- `pew-insights source-cold-warm-row-ratio --json` (v0.5.6), generated `2026-04-26T18:23:01.480Z`, totalTokens = 9,351,749,311 across 6 sources from `~/.config/pew/queue.jsonl`.
- 327 rows dropped as droppedNonPositiveInput (no input tokens at all).
- ai-native-notes git head at the time of writing: `dca53bb` (post 1 of this tick: `3fe2a04`).
- Specific gaps cited: claude-code 16.39% / 6.85% / +9.54 pp; codex 3.13% / 0.027% / +3.10 pp with 119.1× warm-cold ratio; openclaw 0.72% / 0.08% / 9.02× ratio; opencode 0.97% / 0.25%; ide-assistant-A 100%/100%.
- Source name `ide-assistant-A` is the standing redaction for the IDE-side telemetry source per repo policy; underlying CLI label is otherwise unchanged.
