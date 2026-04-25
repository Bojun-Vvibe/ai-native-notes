# Token-velocity percentiles: the 4-order-of-magnitude p50 spread and the editor-assistant outlier

There is a metric I keep coming back to that doesn't quite fit anywhere in the standard "tokens per day" / "tokens per session" framing, because the *unit of measurement* sits at neither the day grain nor the session grain. It sits at the bucket grain — one UTC hour — and asks: when this source was actively producing tokens, *how fast was it producing them*?

The output below comes from `pew-insights token-velocity-percentiles --json` against `~/.config/pew/queue.jsonl`, captured at `2026-04-25T22:34:37.942Z`:

```json
{
  "generatedAt": "2026-04-25T22:34:37.942Z",
  "totalSources": 6,
  "totalBuckets": 1382,
  "totalTokens": 8836955120,
  "sources": [
    { "source": "claude-code",   "buckets": 267, "tokens": 3442385788,
      "min": 99.6,    "p50": 75072.22,  "p90": 722223.30, "p99": 1304497.35, "max": 1815870.88, "mean": 214880.51 },
    { "source": "opencode",      "buckets": 205, "tokens": 2739435973,
      "min": 796.48,  "p50": 141736.82, "p90": 695837.37, "p99": 1004629.15, "max": 1158406.95, "mean": 222718.37 },
    { "source": "openclaw",      "buckets": 374, "tokens": 1701452378,
      "min": 1779.32, "p50": 50550.85,  "p90": 149306.15, "p99": 524151.40,  "max": 751226.03,  "mean": 75822.30 },
    { "source": "codex",         "buckets":  64, "tokens":  809624660,
      "min": 788.62,  "p50": 102559.70, "p90": 586159.62, "p99": 980675.87,  "max": 980675.87,  "mean": 210839.76 },
    { "source": "hermes",        "buckets": 152, "tokens":  142170594,
      "min": 387.98,  "p50": 7632.02,   "p90": 36841.48,  "p99": 69381.90,   "max": 98311.88,   "mean": 15588.88 },
    { "source": "vscode-*",      "buckets": 320, "tokens":    1885727,
      "min": 0.33,    "p50": 40.05,     "p90": 179.80,    "p99": 1101.75,    "max": 2910.42,    "mean": 98.21  }
  ]
}
```

The unit is *tokens-per-minute averaged over a one-hour bucket*. So `p50 = 75072` for claude-code means: in a typical active hour, claude-code produced about 75,072 tokens per minute (~4.5M tokens per active hour). Multiply by 60 to get tokens-per-hour if that's a more natural mental unit; the p50/p90/p99 ladder works either way.

I want to walk three things: the 4-order-of-magnitude spread between the slowest source and the fastest, the *intra-source* skew where p99 is 17× the p50, and what this metric stops short of telling you that you'd need a second pass to recover.

## The cross-source spread: 1875× from the bottom to the top

Order the p50s:

1. vscode-* — 40.05 t/min
2. hermes — 7,632.02 t/min
3. openclaw — 50,550.85 t/min
4. claude-code — 75,072.22 t/min
5. codex — 102,559.70 t/min
6. opencode — 141,736.82 t/min

The ratio from top to bottom is `141736.82 / 40.05 = 3,540×`. From hermes (the next-slowest) to opencode it's still 18.6×. From openclaw (the median pew-source) to opencode it's 2.8×.

These are not subtle differences. They reflect what each source *is*:

- **vscode-* is an editor assistant** that produces tab-completion-grade tokens at a slow trickle. The whole point of the source is short, low-latency interactions; high token velocity would actually be a bug. A bucket that produces 40 tokens per minute means roughly two completions per minute, each a few tokens long. That's healthy for the use-case.
- **hermes is a router/proxy class of source.** It produces metadata and small structured emits. Its p99 (69k t/min) is still smaller than the p50 of the actual coding agents. That's the right shape for a control-plane source.
- **openclaw, claude-code, codex, opencode are coding agents.** Their p50s sit between 50k and 142k tokens per minute. That is consistent with sustained generation: the model is responding to a long-context prompt with a long-form answer, possibly with reasoning tokens included, and the bucket is dominated by output throughput.
- **opencode and codex have the highest p50s** but very different bucket counts (205 vs 64). The p50 ranking would invite the read "opencode is the fastest source" — but with only 205 active buckets vs claude-code's 267, opencode's median is computed over a smaller, possibly more selected, sample.

The bucket-count column is a sample-size sanity gate. With 64 buckets, codex's p99 *equals* its max — meaning at that quantile there's only one observation, and you can't distinguish "extreme tail" from "unique outlier." For codex, only the p50 and the min/max have any sample-size discipline. This is one of those metrics where reading the bucket count is non-negotiable.

## The intra-source skew: every source is a power tail

Compute p99 / p50 per source:

| source | p99 / p50 |
|---|---|
| vscode-* | 27.5× |
| hermes | 9.1× |
| openclaw | 10.4× |
| claude-code | 17.4× |
| codex | 9.6× |
| opencode | 7.1× |

Even the *least* skewed source (opencode) has a p99 that is 7× its p50. The most skewed (vscode-*) has a p99 that is 27.5× its p50 — driven not by big absolute numbers but by the long-tail behaviour of having occasional bursts an order-of-magnitude above the typical 40 t/min trickle.

The right reading: **the unit "active hour" hides a lot of variance.** A typical claude-code active hour is around 75k t/min, but the top 1% of active hours are at 1.3M t/min — a 17× difference. Whatever workload triggers that tail (probably a long auto-applied refactor or a code-review batch) is qualitatively different from the median active hour, and would deserve its own metric ("burst hours") rather than being lumped in.

The p90 column gives a less hostile slice: p90/p50 ranges from 4.5× (vscode-*, 179.8 / 40.05) to 9.6× (claude-code, 722k / 75k). So even at the 90th percentile, claude-code is still doing 9.6× the median, which means the tail is broad, not just a single freak hour.

## The mean-vs-median gap as a tail indicator

| source | mean / p50 |
|---|---|
| vscode-* | 2.45× |
| hermes | 2.04× |
| openclaw | 1.50× |
| claude-code | 2.86× |
| codex | 2.06× |
| opencode | 1.57× |

Mean > median in every source — as expected for a power-law-ish distribution — but the gap is biggest for claude-code (2.86×) and vscode-* (2.45×). Both of those have the highest p99/p50 ratios too. The mean is being pulled up by the right tail, which is consistent with most active hours being ordinary and a few being extreme.

This is exactly the situation where a *cost forecast* built on `mean × hours_per_day × days` will systematically over-predict the modal day and under-predict the burst day. The right shape of forecast is the *median* for the steady-state estimate and a *tail-adjusted* surcharge for the burst probability. The token-velocity percentiles give you both numbers in the same call.

## The vscode-* row deserves its own paragraph

vscode-* is the strangest row in this table because it's the only source where the *entire distribution* sits in a different unit-class:

- p50 is 40 tokens per minute.
- p99 is 1,101 tokens per minute.
- max is 2,910 tokens per minute — still smaller than the *minimum* observed velocity for opencode (796) or hermes (388).

In raw token mass it produces 1.89M tokens across 320 buckets, which is the *most buckets* of any source in the table. So the source is *active* in more hours of the day than any other (320 of 1,382 buckets, 23.2% of all activity), but each of those hours is barely contributing to the cumulative token total.

The right way to think about this is: vscode-* is *background*. It's the source that's quietly co-resident in nearly every active hour, contributing a low-tokens-per-minute trickle whose presence-or-absence is more meaningful than its volume. If you wanted to detect "did the operator have an editor open today?" — vscode-*'s bucket-presence would be a near-perfect proxy. If you wanted to detect "how much real generative work is happening?" — vscode-*'s token volume is essentially noise.

That's a good worked example of why presence and velocity are *separate* signals. A purely token-weighted dashboard would have already collapsed vscode-* to a rounding-error sliver and missed the most ubiquitous behavioural signal in the data. A purely bucket-count dashboard would have inflated vscode-* into a dominant source and missed the fact that the actual token mass lives elsewhere.

## What the percentile ladder won't tell you

Three things this metric is structurally blind to, even though they are tractable from the same JSONL:

1. **Time-of-day modulation.** The percentiles pool every active hour together, regardless of whether it's 02:00 UTC or 14:00 UTC. If claude-code's p50 is 75k overall but 30k during night-buckets and 120k during peak-buckets, the pooled p50 is hiding the effect. The fix is to stratify by hour-of-day before computing the percentile ladder; that exists in the toolkit as `cache-hit-by-hour` and could be re-applied to velocity.
2. **Concurrency interaction.** A bucket where claude-code and openclaw are both active will get half the wall-clock per source, which inflates the *per-source rate* if you're not careful about how you allocated bucket time. The current report computes rate as `total_tokens / 60` (one bucket = 60 minutes), which assumes the source had the whole hour to itself. In high-concurrency hours that overstates true velocity. The correction needs concurrency data, which `concurrency` provides.
3. **Cache-hit confounds.** Tokens-per-minute counts cached input tokens the same as cold input tokens, but the latency profile is wildly different. A bucket where 90% of input is cached can sustain a much higher *output* velocity than a 0%-cached bucket. The velocity ladder is implicitly weighted toward cache-rich hours. Pairing this metric with `cache-hit-ratio` per bucket would let you decompose velocity into "this much is cache-amplified" vs "this much is wall-clock generation."

None of those are objections to the metric — they're invitations to layer it. The velocity ladder is the *univariate* view, and it does that well. The conditional views (velocity-given-hour, velocity-given-cache-rate, velocity-given-concurrency) are what you reach for once the univariate has shown you which sources are worth conditioning on.

## What I'd build on top of it

Two derived metrics I'd add, in priority order:

1. **Velocity stability index.** `(p50 / mean)` per source. The closer to 1.0, the more "steady" the source — its typical hour is its average hour. The further from 1.0 (always less than 1.0 for these distributions), the more bursty. Right now opencode is the steadiest (1/1.57 = 0.64) and claude-code is the spikiest (1/2.86 = 0.35). A dashboard tile that just shows this 0–1 scalar per source would tell you, at a glance, which sources you can capacity-plan against and which you can't.
2. **Cross-source velocity ratio.** Pick a "reference" source (probably the median-velocity coding agent — openclaw, here, with p50 = 50,550) and express every other source as a multiple of it. claude-code is 1.49× reference, opencode is 2.80× reference, codex is 2.03× reference, hermes is 0.15× reference, vscode-* is 0.0008× reference. Same-units, same-grain. That gives you a vendor-and-product-agnostic ladder of "how fast does this tool generate text" with the tooling differences normalised out.

Both are pure post-processing on the existing JSON. Neither needs new data. The hard part of the velocity metric was always the *grain selection* — picking the bucket as the unit, computing the rate against the bucket width, and reporting percentiles instead of a single mean. Once that's done, derived metrics are cheap.

## Short version

Across 1,382 active hour-buckets and 8.84B tokens, the per-source velocity p50 ranges over 4 orders of magnitude (40 to 142k tokens-per-minute). Every source is right-tailed: p99/p50 is between 7× and 28×. The vscode-* row is a presence signal, not a volume signal — most active buckets, least token mass per bucket. The right derivative is a velocity-stability scalar (p50/mean) and a normalised cross-source ratio against the median coding agent. Don't forecast costs from the mean; forecast from the median and surcharge for the tail.
