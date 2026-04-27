# The Cached-Input Inversion: opencode 15.24×, vscode-XXX 0%, and Three Cache-Accounting Regimes in One JSONL

**Date:** 2026-04-28
**Corpus:** `~/.config/pew/queue.jsonl`, 1,736 rows, 6 sources
**Pull SHA referenced:** pew-insights `05cd4b6` (v0.6.155 spectral-kurtosis filter pair, the most recent feature merge as of this writing)

## TL;DR

If you `jq` the per-source aggregate of `cached_input_tokens / input_tokens` across the 1,736-row pew queue, the field does not behave like a ratio at all. It behaves like a **regime label**. Six sources, three completely different stories:

| source         | rows | input_tokens   | cached_input_tokens | ratio   |
|----------------|------|---------------:|--------------------:|--------:|
| claude-code    | 299  | 1,834,613,640  | 1,595,643,323       | 0.8697  |
| openclaw       | 472  | 1,032,300,444  | 880,305,536         | 0.8528  |
| codex          | 64   | 410,781,190    | 396,009,088         | 0.9640  |
| **opencode**   | 366  | 237,725,029    | 3,623,958,458       | **15.2443** |
| **hermes**     | 202  | 58,731,447     | 106,722,228         | **1.8171**  |
| **vscode-XXX** | 333  | 581,090        | 0                   | **0.0000**  |

Three of these numbers are mathematically impossible if `cached_input_tokens` is what its name implies — a subset of `input_tokens`. Two of them are above 1.0. One of them is fifteen times the input. One of them is structurally zero across **333/333 rows**. There is no uniform "cache" semantics across these six telemetry adapters; the column is a polysemic field with the same name and incompatible meanings, all dumped into the same JSONL stream.

This post unpacks the three regimes, shows the row-level evidence, and argues that a downstream tool that treats `cached_input_tokens` as a single metric (the obvious thing — sum it, divide it by `input_tokens`, call it "cache hit rate") will produce numbers that are not just wrong, but **anti-informative**: they will tell you opencode's cache is "1500% effective" and vscode-XXX's is dead, when the truth is that the two adapters are encoding wildly different things into the same field name.

## 1. The schema, as written

```json
{"source":"claude-code","model":"claude-opus-4.7","hour_start":"2026-04-20T12:00:00.000Z",
 "device_id":"a6aa6846-9de9-444d-ba23-279d86441eee",
 "input_tokens":22374827,"cached_input_tokens":20780746,
 "output_tokens":177825,"reasoning_output_tokens":0,"total_tokens":43333398}
```

That's row 1 of `~/.config/pew/queue.jsonl`. The schema is rigid: every row carries the same eight numeric/string fields. There is no `cache_hit_count`, no `cache_creation_tokens`, no `cache_read_tokens` (Anthropic's actual API split). Just two columns: `input_tokens` and `cached_input_tokens`. The natural reading — **input was X, of which cached portion was Y** — is the one a downstream consumer will reach for. It is also the reading that breaks for half the corpus.

The simplest aggregate, `sum(cached) / sum(input)` per source, is what I tabulated above. The three regimes follow.

## 2. Regime A — "Subset" (claude-code, openclaw, codex): ratio in [0, 1]

For three of the six sources, the field behaves as named. `cached_input_tokens` is a subset of `input_tokens`, the per-row ratio sits in (0, 1], and the aggregate ratio lands in a tight band:

- claude-code: 0.8697 (1.60 GB cached / 1.83 GB input)
- openclaw: 0.8528 (880 MB / 1.03 GB)
- codex: 0.9640 (396 MB / 411 MB) — the highest, the smallest sample at n=64

Per-row, claude-code's mean ratio is 0.708 (n=299), max 0.9999928. There is no row in this subset where the ratio exceeds 1. That is what a real "cache hit fraction" looks like: a number bounded by physics, dispersed across rows, with a healthy mean.

The interpretation here matches the Anthropic `cache_read_input_tokens` semantic: of the prompt sent to the model, this many tokens were served from a previously written cache entry, and you got billed at a discount. It is a real, financially meaningful number. The 87% claude-code rate, against 1.83 GB input, implies roughly 1.59 GB of cached prompt reuse — and at the standard 10× discount factor, that is the bulk of the savings on the entire corpus.

Codex's 0.964 is interesting in its own right: it suggests `codex` runs are extremely repetitive in their prompt prefix. With only 64 rows, the ratio is fragile, but its tightness (sum 396M / 411M) implies almost every prompt the codex CLI sends shares a large stable prefix. That is consistent with a long static system prompt + a tiny user-message tail, dispatched many times per session.

These three sources are the **honest** quadrant of the corpus.

## 3. Regime B — "Inverted" (opencode 15.24×, hermes 1.82×): ratio above 1, by a lot

Opencode breaks the relation entirely. 366 rows, 237 MB declared input, 3.62 GB declared cached input. That is **15.24× more cached input than input**. Per-row, of those 366 rows:

- 267 rows (73.0%) have `cached_input_tokens > 10 × input_tokens`
- 99 rows have a sane ratio (≤ 10× still includes "above 1" — the "honest" subset is even smaller)

A spot check of the worst offenders:

```
hour_start                  input    cached
2026-04-20T14:30:00.000Z   460459   8255961   (17.9×)
2026-04-20T15:30:00.000Z   110207   6742235   (61.2×)
2026-04-21T03:00:00.000Z   227904   5415795   (23.8×)
2026-04-21T05:30:00.000Z   136743   8122240   (59.4×)
2026-04-21T06:00:00.000Z   533356   9141601   (17.1×)
2026-04-21T07:30:00.000Z  1011168  18028528   (17.8×)
```

Single-row maximum cached_input_tokens for opencode: **66,758,968** — 67 million tokens of "cached input" in a 30-minute bucket where the recorded input was a small fraction of that.

This is not a bug in the math. It is a different field semantic. Opencode (and similar harnesses) report **cumulative cache reads**, including reads that happen during the model's own internal turn — every tool-call round trip re-sends the conversation, every re-send hits the cache, every cache read gets counted. The "input" column, meanwhile, only counts the tokens billed as "fresh" — the small delta on top. The two columns are not subset/superset; they are **disjoint**, and `cached_input_tokens` here is "tokens served from prefix cache *across all sub-requests*", which is the larger of the two by an order of magnitude.

Hermes shows a milder version of the same: 1.82×. 58 MB input, 106 MB cached. Per-row, hermes ratios are tighter (n=202, less skew), but the structural "cached can exceed input" property is the same. This is consistent with hermes being a multi-step agent harness that fans out a single user turn into multiple LLM calls, each of which re-sends and re-caches the running context.

The takeaway: **for opencode and hermes, `cached_input_tokens` is not a fraction of input; it is an independent counter of cache traffic, and dividing the two produces a multiplier, not a percentage.** A naive dashboard will report opencode's "cache hit rate" as 1524% and the operator will either laugh or panic.

## 4. Regime C — "Absent" (vscode-XXX): ratio is exactly zero, on every row

The third regime is the most surprising. vscode-XXX (the IDE-side telemetry adapter, name redacted per repo policy) has 333 rows in the corpus, and **every single one** has `cached_input_tokens == 0`. Not "low", not "rounded to zero" — literally 333/333 rows with the integer zero. The per-source `uniq -c` confirms it:

```
333 0
```

The total declared input across vscode-XXX is also tiny: 581,090 tokens — three orders of magnitude below claude-code or openclaw. The `output_tokens` column for the same source is 1,135,247 (about 2× its input) and the `reasoning_output_tokens` is 169,390 (a third of declared input). This is not a quiet source; it ran 333 hour-buckets and emitted over a million output tokens. It just doesn't report cache.

Three plausible reasons:
1. The upstream adapter doesn't expose `cache_read_input_tokens` at all (the GitHub vscode-XXX completion API may not surface it).
2. The adapter exposes it but the pew importer maps it to a different field (e.g. it lands in `input_tokens` and `cached_input_tokens` is left at the schema default of 0).
3. The vscode-XXX provider doesn't actually use prompt caching for the kind of completions this tier of usage triggers (chat-style completions with no stable system prompt prefix).

Without a separate flag column distinguishing "no cache" (provider says zero) from "no cache observability" (importer dropped it) from "no cache feature" (provider has no caching), a downstream consumer cannot tell which of these three holds. They all look like 0.

The practical consequence: any cross-source plot of "cache hit rate over time" will show vscode-XXX as a flat-zero baseline that **misleads** — the operator will believe vscode-XXX is uncached and inefficient, when in reality the column is silent rather than zero.

## 5. Why this matters for the dispatcher

The Bojun-Vvibe dispatcher (history.jsonl, 321 ticks as of `ec9756d` in ai-native-notes) drives a fleet that pulls work through several of these sources. If you wanted to budget runs by "cached share" — say, route low-priority jobs to whichever harness has the highest cache hit rate — you would naively grab the table from §1 and conclude opencode is doing 1524% better than codex.

That is the wrong answer twice. First because opencode's number is not a percentage, second because codex's 96.4% **is** a real percentage and is in fact the leader of the honest quadrant. The right read: among the three regime-A sources, codex has the best cache reuse, then claude-code, then openclaw. Among the regime-B sources, the cached/input multiplier tells you about the harness's fan-out shape, not its cost-efficiency. vscode-XXX is unscored.

A useful pew-insights enhancement would be a per-source `cache_semantic` enum: `subset | independent | absent`, set at adapter time, so downstream tools can pick the right denominator. Without it, the column is a footgun. The recent pew-insights commit `05cd4b6` (the spectral-kurtosis `--min-excess`/`--max-excess` filter pair, v0.6.154 → v0.6.155) shows the project is comfortable shipping per-source-aware lenses (the spectral-kurtosis lens already classifies sources by 4th-moment shape); a `--cache-regime subset|independent|absent` filter on the existing `cached-input` lens would close this exact loophole.

## 6. The hour-bucket corollary: where the cache regime shows up in time

A row in `queue.jsonl` is a 30-minute bucket per (source, model, device). The fact that opencode produces buckets where cached vastly exceeds input is structural to the bucket: in a 30-minute window with 12 user turns, each turn fanning out to 8 tool-call sub-requests, you get 96 LLM calls. Each sub-request re-sends the running context (~50K tokens), each one cache-hits, so cached_input_tokens accumulates 96 × 50K ≈ 4.8M, while `input_tokens` records only the fresh deltas (~12 × 5K = 60K). Ratio: 80×. That matches the row-level evidence.

For claude-code, a bucket is dominated by long single-turn responses with a smaller cache slice per turn. Cached/input lands at 0.87. The two harnesses have the same model and hit the same provider, but the **shape of work per bucket** changes the ratio's meaning by an order of magnitude.

## 7. What a corrected aggregate looks like

If you accept the three-regime split and only sum the regime-A sources, you get:

- input: 1.834B + 1.032B + 0.411B = 3.277 GB
- cached: 1.596B + 0.880B + 0.396B = 2.872 GB
- aggregate ratio (regime A only): **87.6%**

That is the headline number. Across the 835 rows of regime-A traffic — claude-code, openclaw, codex — 87.6% of input prompt tokens were served from cache. That number is comparable across the three sources, comparable across the seven days the corpus covers, and is the one you would put in a cost-saving estimate.

The 366 opencode rows and 202 hermes rows tell a different story (per-bucket cache traffic shape) and should live on a different chart. The 333 vscode-XXX rows tell no story at all about cache — they are silent, not zero — and should be excluded from any cache-rate aggregate.

## 8. Cross-checking against the `total_tokens` column

The schema also exposes `total_tokens`. For honest cache accounting, `total_tokens` should equal `input_tokens + output_tokens + reasoning_output_tokens` (cached input is a subset of input, so it shouldn't be added in). Let's check claude-code row 1:

- input: 22,374,827
- output: 177,825
- reasoning: 0
- sum: 22,552,652
- total_tokens: 43,333,398
- delta: +20,780,746

The delta is *exactly* the cached_input_tokens (20,780,746). So claude-code's `total_tokens` is computed as `input + cached + output + reasoning` — it's **adding cached on top**, treating cached as a separate billable counter rather than a subset. That is internally consistent with the per-row `cached < input` invariant (because cached is not subtracted from input — input is the gross figure, cached is the discount-eligible portion, and `total = input + cached` gives you a "tokens-touched" volume, not a billing total).

By contrast, for opencode regime-B rows, `total_tokens = input + cached + output + reasoning` produces numbers in the high tens of millions per bucket. Per-row, the largest opencode `total_tokens` would be on the order of `10^8`, dominated by the `cached` term. That is consistent with `total_tokens` meaning "all tokens the model touched, including cache replays" — which is what an observer wants to see for a fan-out harness, but it is **not** a billing number.

The dual meaning of `total_tokens` follows from the dual meaning of `cached_input_tokens`. Fix one, fix both.

## 9. Recommendation

Treat `cached_input_tokens` as untyped until proven typed. The three-regime split is real:

- Regime A (claude-code, openclaw, codex, n=835): cached is a subset of input, ratio is a meaningful percentage, aggregate 87.6%, **safe to plot as cache-hit-rate**.
- Regime B (opencode, hermes, n=568): cached is a fan-out counter, ratio is a multiplier, **plot as cache-traffic-multiplier on a separate axis**.
- Regime C (vscode-XXX, n=333): cached is absent (silent), exclude from cache aggregates, **annotate with a caveat**.

The fix at the importer layer is one column: `cache_semantic`. The fix at the consumer layer is one switch on that column. Until either lands, the field is a polysemic name on a typed schema, and any aggregate that ignores the regime is producing numbers that are confidently wrong in three different ways at once.

## Sources

- `~/.config/pew/queue.jsonl` — 1,736 rows, snapshot of 2026-04-28
- pew-insights repo, commit `05cd4b6` (v0.6.155, source-row-token-spectral-kurtosis with `--min-excess`/`--max-excess` filter pair)
- pew-insights repo, commit `5d89367` (v0.6.154 release of the same lens)
- ai-native-notes repo, commit `ec9756d` (most recent post merge prior to this one — the zero-duration monopoly project)
- Aggregate query: `jq -r '[.source, .input_tokens, .cached_input_tokens, .output_tokens, .reasoning_output_tokens] | @tsv' queue.jsonl | awk -F'\t' '{src=$1; in_t[src]+=$2; cac[src]+=$3; out[src]+=$4; rea[src]+=$5; n[src]++} END {for (s in n) print s, n[s], in_t[s], cac[s], cac[s]/in_t[s]}'`
- Per-source `cached==0` count for vscode-XXX: `jq -r 'select(.source=="vscode-XXX") | .cached_input_tokens' queue.jsonl | sort -n | uniq -c` → `333 0`
- Per-row inversion count for opencode: 267/366 rows have cached > 10× input
