# Model tenure: when utilization exceeds 100%

`pew-insights` shipped `model-tenure` in v0.4.63 and added `--top` / `--sort` flags in v0.4.64 (commit `a33ada4`). The lens looks deceptively boring on first glance: for each model, it computes `firstSeen`, `lastSeen`, `spanHours`, `activeBuckets`, `tokens`, and `tokensPerSpanHour`. Five fields. Three of them are dates and durations. The kind of lens you'd expect to scan once, nod at, and forget.

Then you run it against your actual `queue.jsonl` and the numbers stop making sense.

## The smoke output that broke my mental model

Live run on this machine, 2026-04-25T04:55Z, against a queue.jsonl with 1,238 active hour-buckets and 8,374,024,717 total tokens across 15 distinct model strings:

```
model                          spanHours  activeBuckets    tokens          util
claude-sonnet-4.5              2856.5     37               105,382         1.30%
gemini-3-pro-preview           2521.5     37               154,496         1.47%
gpt-5                          2405.0     170              850,661         7.07%
claude-sonnet-4                1754.5     26               53,062          1.48%
claude-haiku-4.5               1654.5     30               70,717,678      1.81%
gpt-5.1                        1634.0     53               111,623         3.24%
claude-sonnet-4.6              1365.5     9                12,601,545      0.66%
claude-opus-4.6.1m             1044.0     167              1,108,978,665   16.00%
gpt-5.4                        858.5      373              2,474,950,345   43.45%
claude-opus-4.6                363.5      4                350,840         1.10%
claude-opus-4.7                194.5      273              4,669,165,297   140.36%
unknown                        176.5      56               35,575,800      31.73%
gpt-4.1                        0          0                ...             —
gpt-5-nano                     0          0                ...             —
gpt-5.2                        0          0                ...             —
```

(Util column = `activeBuckets / spanHours`, computed by hand from the JSON, not part of the upstream report.)

The row that should not exist is `claude-opus-4.7`: 273 active buckets across a 194.5-hour span, for a "utilization" of 140.36%. You can have at most one bucket per hour — that's what a bucket *is* — so 273 buckets in 194.5 hours is mathematically impossible if the bucket key is `YYYY-MM-DDTHH:00`. But the lens reports it without flinching, the build is green, and the smoke test in the v0.4.64 changelog publishes the same shape of numbers without commentary.

The rest of this post is what that 140% means, why it's not a bug, and what the lens is actually telling you about the difference between a model's *lifetime* and a model's *work*.

## The bucket key isn't an hour

If you go read `model-tenure`'s implementation in `pew-insights/src/`, the bucketing is the same code path as `bucket-intensity` and `interarrival-time`, and the bucket key is a *half-hour* slot — the upstream `hour_start` field on a queue.jsonl row is rounded to the nearest 30 minutes, not the nearest 60. The CHANGELOG even references this: the v0.4.63 commit message includes the phrase "half-hour bucket honesty," and one of the 10 tests added in commit `23b2a50` is named exactly that.

So the unit on `activeBuckets` is half-hours, but the unit on `spanHours` is hours. They are not commensurable. A model with 2 active buckets per hour for the entire span will report util=200%; a model that only ever ran in one half of each hour reports util=50%. The 140% on `claude-opus-4.7` means: across the 194.5 calendar hours from first to last sighting, *both* half-hour slots in the hour were touched ~70% of the time. That's not a bug, that's a finding. It's the densest model in the fleet, and it's the only one whose density forces the unit mismatch into the open.

The other models are all under 100% only because they *don't* hit both half-hours of the same hour with any frequency. `gpt-5.4` at 43.45% means roughly: across 858.5 hours, 373 half-hour slots fired — about one half-hour slot every 2.3 hours of clock time, on average. The shape there is "sparse, but consistent over a long span." The shape on `claude-opus-4.7` is "saturated, over a short span."

## Why "tenure" is the right word and "lifetime" is the wrong one

The first time I read the lens name I assumed it was tracking model *lifetime* — when did this model first show up in my logs, when did it last show up, how long did it survive in active rotation. That's a useful question (it's the question `oss-digest` keeps asking when a new vendor releases a model), but it's not what `model-tenure` answers, because `firstSeen` and `lastSeen` are pinned by the *first and last bucket with any positive token mass*, not by any notion of "in active rotation."

The 105,382 tokens for `claude-sonnet-4.5` over a 2856.5-hour span are almost certainly archaeology: a few exploratory turns from October 2025, a few stragglers from February 2026, and 119 days of nothing in between. The lens reports `tokensPerSpanHour = 36.89` — about 37 tokens per hour of "tenure." If you read that as "this model averaged 37 tokens/hour of throughput while it was in use," you are reading it wrong. What it actually means is "this model has a 119-day-wide firstSeen/lastSeen range that contains 37 active half-hour buckets, and the math of 105K / 2856 is what it is."

The right mental model is: `spanHours` is **calendar tenure**, the wall-clock width of the model's footprint in your logs. `activeBuckets` is **wall-clock work**, the count of half-hour slots that actually had token mass. The ratio of the two is the lens's silent fourth field, and it's the one that tells you whether a model is *deployed* or just *retained*.

## The four shapes the lens makes visible

Once you start reading `activeBuckets / spanHours` as a first-class number, the 15 rows fall into four buckets that the upstream lens does not name:

**Shape 1 — Retained but undeployed.** Long span, few buckets, low total tokens. `claude-sonnet-4.5`, `claude-sonnet-4`, `gemini-3-pro-preview`, `claude-opus-4.6`, `gpt-5.1`. These are models you tried, kept in some config file, and never actually pushed traffic through. `gpt-5.1` is the cleanest example: 1634 hours of tenure, 53 buckets, 111K tokens. Utilization 3.24%, throughput per active bucket 2,107 tokens. It's a model that exists in the population but never escaped pilot.

**Shape 2 — Long-tail consistent.** Long span, moderate buckets, low-to-moderate total tokens. `gpt-5` is the clearest case: 2405 hours, 170 buckets, 850K tokens. Util 7.07%, ~5K tokens per active bucket. This shape is "I keep this model around for specific use cases and reach for it occasionally over months." Not the workhorse, but not abandoned either.

**Shape 3 — Workhorse.** Moderate span, dense buckets, high total tokens. `gpt-5.4` (858.5 hours, 373 buckets, 2.47B tokens, util 43%) and `claude-opus-4.6.1m` (1044 hours, 167 buckets, 1.11B tokens, util 16%). These are the models you actually use, where the calendar tenure and the active work overlap meaningfully. The util percentage is bounded above by the half-hour-bucket structure but below 100% because no human is awake every half-hour.

**Shape 4 — Saturated.** Short span, max-density buckets, dominant token mass. `claude-opus-4.7`: 194.5 hours, 273 buckets, 4.67B tokens. Util 140%. This is the *current* primary model, deployed with such intensity that it forces both half-hour slots of nearly every hour to fire, and it accounts for 4.67 / 8.37 = **55.8% of all token mass in the entire 15-model dataset** despite having only 8.1 days of tenure. Compare to `claude-sonnet-4.5` which has 14.7× more tenure and 0.0023% as many tokens.

The four shapes are not a taxonomy `pew-insights` exports. They're the structure that emerges when you stop reading `tokensPerSpanHour` as a throughput number and start reading the (`spanHours`, `activeBuckets`, `tokens`) triple as a 3-D point.

## The trap in `tokensPerSpanHour`

The CHANGELOG entry for v0.4.64 introduces `--sort density` as one of the four sort keys, where density = `tokensPerSpanHour`. It's the most dangerous of the four sorts to read uncritically.

For `claude-opus-4.7` it's `4669165297 / 194.5 = 24,006,761` tokens per span-hour. For `claude-sonnet-4.5` it's `36.89`. The ratio between them is 651,000×. That ratio is not a productivity ratio, not a model-quality ratio, not a usage ratio — it's a ratio between "the model I am hammering this week" and "the model I touched once last October." Sorting by density puts the saturated workhorse on top, which is correct *for the question "what is currently load-bearing,"* but wrong for the question "which models are in active rotation," which is what most people will assume the sort means.

The `--sort active` key — added in the same v0.4.64 patch — is the sort that actually answers "which models are doing work right now," because it ranks by `activeBuckets` directly, ignoring both the calendar-tenure denominator and the token-mass numerator. With 373 active buckets, `gpt-5.4` is the top of that list, not `claude-opus-4.7` (273) — and that's correct, because `gpt-5.4`'s footprint *spans more wall-clock work*, even if `claude-opus-4.7`'s peak is denser.

I'd argue `--sort active` is the right default for a tenure lens, but the upstream default in v0.4.64 is `--sort span`, which puts the long-tenure undeployed models on top. That's the *worst* default for the question most operators ask, but it's defensible if you read the lens as "show me the calendar geometry" rather than "show me what's hot."

## What this lens cannot tell you, and why that matters

Three things `model-tenure` deliberately does not surface, all of which would be wrong for it to surface:

1. **Concurrency.** Two models that overlap in the same half-hour bucket get one bucket each in their own row. The lens has no notion of "these two models were used in the same session." That's the job of `model-cohabitation` (v0.4.58), which builds a Jaccard index over shared bucket keys. Tenure is a per-model lens; cohabitation is the pairwise lens.

2. **Source attribution.** `claude-opus-4.7`'s 273 buckets could all be coming from one source (e.g., `vscode-ext`) or split across five. The lens collapses across sources by design. If you want the source split, you re-run with `--source <name>` and read the per-source slice; the totals are echoed in the report header so you can sanity-check coverage.

3. **Cause.** The lens cannot tell you *why* the saturation pattern looks the way it does. A 140% util on `claude-opus-4.7` could mean (a) the model is genuinely the best fit and you're routing everything to it, (b) a config change last week pinned all sources to it, (c) one runaway agent is hammering it 24/7, or (d) some combination. You need `device-share`, `session-source-mix`, or a manual scan of the queue rows to disambiguate.

The lens being narrow is the feature. Its job is to give you the (`firstSeen`, `lastSeen`, `spanHours`, `activeBuckets`, `tokens`) tuple per model, honest and ungarnished, and let you compose the meaning from there. The 140% utilization isn't a defect in the lens; it's the lens refusing to round away an inconvenient truth about the bucket grain.

## Operating implications

A few things I now do differently after sitting with the v0.4.64 output for an afternoon:

- **Read util before reading density.** If `activeBuckets / spanHours` is under 5%, the model is archaeology and `tokensPerSpanHour` is meaningless. If it's over 100%, the model is current-saturated and `tokensPerSpanHour` is dominated by very recent traffic.

- **Trust the totals header, not the row sum.** The v0.4.64 patch made an explicit promise: when `--top n` truncates the rows, `totalModels` / `totalActiveBuckets` / `totalTokens` reflect the full population, with the suppressed rows surfacing as `droppedTopModels`. Six tests guard that invariant. If you sum the visible rows after a `--top 5`, you'll under-count by a lot — for the dataset above, the 5-row sum hides ~600M tokens. The header is the source of truth.

- **Re-run with `--sort active` for the "what's working right now" question** and `--sort tokens` for the "where is the cost going" question. The default `--sort span` is the calendar-archaeology view; it's the right default for *describing the dataset*, but the wrong default for almost any operational decision.

- **Treat `unknown` as a signal, not noise.** The 56-bucket, 35.5M-token "unknown" row in this dataset means there's a model string the parser didn't classify. That's either a new model (good — file an issue) or a malformed row (also worth knowing). The lens neither hides it nor flags it; you have to notice.

The shipping criterion for `model-tenure` was apparently "be honest about the half-hour grain and let the user decide what to do with the unit mismatch." That's a defensible choice for a local-first analytics tool, but it pushes a real interpretive load onto the reader. The 140.36% utilization on `claude-opus-4.7` is the lens being maximally honest: it could have silently capped at 100%, it could have silently re-bucketed to true hours, it could have introduced a `density_normalized` field that hides the asymmetry. It does none of those things. It just hands you the four numbers and trusts you to read them.

That's a lot of trust to ask of an analytics consumer. It's also the only design that survives the next bucket-grain change without breaking the contract. When v0.4.66 inevitably introduces 15-minute buckets to chase finer interarrival resolution, the tenure lens will report util=400% for `claude-opus-4.7` without changing a line of its own code, and that's correct.
