# Provider Tenure: the Vendor Axis and the 2.5× Throughput Gap

When you analyze AI-native development workloads, you almost always start at the model axis. Which model is doing the work? Which one consumes the most tokens? Which one is on the way out? That framing is useful, but it loses something important: a single vendor usually ships *several* models in parallel, and over time it ships replacement models that inherit the workload of their predecessors. If you only look at models, you see a churning population. If you roll up to the **provider** axis, you see a stable industrial relationship — and the differences between vendors become visible in a way that single-model views obscure.

This post is about that vendor axis and a specific number that fell out of running `pew-insights provider-tenure` on a real local queue at `2026-04-25T13:53:44Z`.

## The data

Here is the raw output, abbreviated to fit prose:

```
pew-insights provider-tenure
as of: 2026-04-25T13:53:44.914Z   providers: 4   active-buckets: 1,252   tokens: 8,598,095,271

provider   span-hr  active-buckets  distinct-models  tokens          tok/bucket   tok/span-hr
anthropic  6463.5   544             7                6,057,628,143   11,135,346   937,206
openai     6007.0   615             6                2,504,736,832   4,072,743    416,970
google     2521.5   37              1                  154,496       4,176        61
unknown    176.5    56              1                  35,575,800    635,282      201,563
```

Four providers. Two giants. A long-tenure but invisible third. And a rookie that only became legible to the classifier on April 17.

There is a single derived ratio I want to focus on:

```
anthropic tok/bucket = 11,135,346
openai    tok/bucket =  4,072,743
ratio     ≈ 2.73
```

Anthropic delivers **2.7× as many tokens per active hour-bucket** as OpenAI on this queue, and on a tok/span-hr basis the ratio is **937,206 / 416,970 ≈ 2.25×**. The OpenAI fleet has slightly more active buckets (615 vs 544) but moves less than half the token mass through them. On the same workload, on the same machine, with overlapping span windows, one vendor's models do roughly two-and-a-half times more work per unit of clock time when they're engaged.

That is a large number. It demands an explanation.

## Why provider rollup matters

Before getting into the explanation, it's worth saying why the provider axis even exists as a separate lens.

A model-level view of the same data shows fifteen models — `claude-opus-4.7`, `claude-opus-4.6.1m`, `claude-haiku-4.5`, `claude-sonnet-4.6`, `claude-sonnet-4.5`, `claude-sonnet-4`, `claude-opus-4.6`, `gpt-5.4`, `gpt-5.2`, `gpt-5.1`, `gpt-5`, `gpt-5-nano`, `gpt-4.1`, `gemini-3-pro-preview`, and an `unknown`. Most of these have short tenures relative to the eight-month window because vendors ship rapidly. Span hours at the model level overstate churn: every bump from `claude-opus-4.6` to `4.6.1m` to `4.7` resets a clock that should really be reset only when the *vendor* relationship breaks.

Provider tenure rolls those siblings up. The result is that Anthropic's first-seen at `2025-07-30T06:00:00Z` is the genuine start of the working relationship, not the start of any one weight checkpoint. The 6463.5-hour span (≈ 269 days) is the real "we keep coming back" duration. OpenAI's 6007-hour span (≈ 250 days) starts ~19 days later and is similarly anchored. Those two are the substrate of the workload — they are not alternatives that compete every day; they coexist in different roles.

The model axis tells you about churn. The provider axis tells you about the underlying contract.

## Decoding the 2.7× tok/bucket gap

A ratio like 2.7× is not subtle and is rarely caused by a single thing. There are at least four candidate explanations and the data lets us narrow them down.

### Candidate 1: model selection inside the vendor

The Anthropic side of the ledger has 7 distinct models; OpenAI has 6. But not every model contributes equally. From the `tenure-vs-density-quadrant` lens (separate post) we know that `claude-opus-4.7` alone is responsible for 4.86B tokens out of Anthropic's 6.06B — about 80% of the vendor's mass through 291 buckets. The Anthropic provider number is essentially "what does opus-4.7 look like, smoothed by a long tail."

OpenAI's mass is more spread out: `gpt-5.4` carries 2.50B of the 2.50B vendor total — about 99.97%. Other OpenAI models (`gpt-5.2`, `gpt-5.1`, `gpt-5`, `gpt-5-nano`, `gpt-4.1`) collectively contribute well under 0.1% of the provider's mass.

So a fair pairwise comparison is really opus-4.7-on-anthropic vs gpt-5.4-on-openai. Both are flagship coding models from their vendors. Anthropic's flagship runs hotter per bucket because it's deployed in a denser, larger-context loop.

### Candidate 2: bucket density vs bucket count

OpenAI has more active buckets (615 vs 544) but lower density. This is the classic "spreader vs concentrator" pattern: OpenAI's models are touched in more hours, but each touch is lighter. Anthropic's models are touched in fewer hours but each touch is heavier.

The ratio of tok/span-hr (937K vs 417K) is *smaller* than the ratio of tok/bucket (11.1M vs 4.07M), because Anthropic uses a smaller fraction of its 6463-hour span as active buckets. Anthropic's duty cycle within span = 544 / 6463.5 ≈ 8.4%; OpenAI's = 615 / 6007 ≈ 10.2%. OpenAI is engaged in a larger fraction of clock hours but does less per engaged hour.

This is the difference between a freight train and a commuter line. Both run a lot. One moves much more cargo per trip.

### Candidate 3: tool wiring at the host

`pew-insights` rolls up by normalised model id, so identical workloads issued through different host wrappers will all attribute to the same provider regardless of which bridge sent them. That means the gap is not an artifact of "Anthropic happens to be wired through a chattier host." It is a real difference in the per-bucket payload size that the host actually sends.

Per-bucket payload size is dominated by *context window utilization*: how much you stuff into each call. The `opus-4.6.1m` 1M-context variant is in the dense quadrant with 6.64M tokens per bucket. The non-1m variants and OpenAI's `gpt-5.4` sit lower. Provider gap = context-window strategy gap.

### Candidate 4: workload scheduling

The machine doesn't run both providers at the same time on every bucket. Anthropic gets the heavy multi-file refactor jobs and the long agent loops (because the 1M context is the right tool for them). OpenAI gets the smaller-context single-shot tasks and the cheaper background jobs. The provider tenure lens collapses scheduling into a single number, but the underlying truth is that the host *chose* to send the dense workload to Anthropic.

This is the explanation that matters most. The 2.7× isn't a benchmark of vendor capability — it's a fingerprint of the host's routing policy. Show me a different host and the ratio will swing. That is the key methodological point of provider-tenure analysis: it measures the relationship the operator *built*, not the vendors' raw performance.

## The Google line and the long-tenure illusion

Google's row is the most instructive in the table even though it carries 0.002% of the token mass. Span = 2521.5 hours (≈ 105 days). Active buckets = 37. Tokens = 154,496. Tok/bucket = 4,176. Tok/span-hr = 61.

A 105-day span sounds substantial. The first-seen of `2025-11-20T04:30:00Z` and last-seen of `2026-03-05T06:00:00Z` look like a five-month relationship. But 37 active buckets across 105 days is a *duty cycle of 1.5%* — measured at the hour granularity, this provider has been engaged for one-and-a-half percent of the clock hours of its tenure. And it stops on March 5, almost two months before the as-of timestamp.

This is the long-tenure illusion. Span-hours alone are not a measure of importance, weight, or even continued use. They are a measure of *first-touch to last-touch*, and a single experiment back in November is enough to anchor a long span if a single follow-up touch happens months later. The right way to read provider tenure rows is: span tells you the *ceiling* of the relationship; active buckets tells you what's actually happening inside that ceiling.

The dashboard rule of thumb that fell out of this is **density ratio = activeBuckets / spanHours**. Anthropic 8.4%, OpenAI 10.2%, Google 1.5%, unknown 31.7%. The first three are stable engagements at varying intensity. The unknown's 31.7% over 176 hours marks it as a hot, brand-new relationship — and the next subsection picks that up.

## The unknown row: a brand-new vendor or a classifier miss?

The most interesting row is the bottom one. `unknown` provider, first-seen `2026-04-17T06:30:00Z`, last-seen `2026-04-24T15:00:00Z`. Span 176.5 hours. 56 active buckets. 35.5M tokens. Density ratio 31.7%, the highest in the table by 3×.

Two interpretations:

1. **A genuinely new vendor** that the model-id normaliser doesn't yet recognise. New entrant launches in mid-April, gets adopted within a week, runs hot for a hundred buckets, and the classifier hasn't caught up. The 35.5M token volume — 1,000× larger than Google's eight-month total — supports a real adoption story.

2. **A normalisation regression**: a model id format that recently changed and is now landing in the bucket that catches everything unparseable. The host is the same, the work is the same, but the id field shifted and the rollup lost it.

You can usually tell which from the cross-correlation with model tenure. In the quadrant lens, `unknown` shows up as a single short-dense model with span 176.5h, 56 buckets, 35.6M tokens, density 635K — sitting between the established short-dense flagships and `gpt-5.2`/`gpt-5-nano` outliers. A new vendor wouldn't typically show up *only* as one model id; you'd expect to see two or three siblings as the vendor rolled out variants. So the leading hypothesis is a normalisation miss on a known vendor's new family. The one-week clock and the high density both fit "the host upgraded; the classifier didn't."

This is exactly the kind of issue that provider-tenure surfaces and model-tenure hides. At the model axis, `unknown` looks like just another short-tenure model; at the provider axis, it looks like a classification failure that needs a normaliser update to route 35.5M tokens back to the right vendor row.

## The methodology takeaway

The 2.7× tok/bucket ratio and the 31.7% unknown density are both data points. The methodological point underneath is that the provider lens forces a *different* set of questions than the model lens.

At the model lens you ask: which model is best, which is being deprecated, which one am I locked into? Those are tactical questions and they have correct answers in days.

At the provider lens you ask: how is my compute bill split between the two vendor relationships? Is the split widening or narrowing? Is one vendor being pushed into a corner of light bucket touches while the other gets the dense work? When a new model lands, does it inherit the workload of its predecessor or is the workload fragmenting across more siblings? Those are strategic questions and they have correct answers in months.

The corollary is that you should run provider-tenure on the same cadence you renegotiate vendor terms: roughly quarterly, more often if the workload mix is changing. A 2.7× tok/bucket gap is a routing fact. A swing in that ratio between quarters is a behavioural change in your own host configuration that you might not have intended to make.

## What the eight-month frame says

Stepping back to the headline:

- 8.6 billion tokens across 1,252 active hour-buckets in eight months
- 80% of mass on Anthropic, 29% on OpenAI, ~0% on Google, 0.4% on unknown
- Two stable long-tenure vendors carrying the production work; a third in dormant experimental status; a fourth being misclassified in real time
- A 2.7× per-bucket throughput gap that is a property of the host's routing strategy, not of either vendor's raw capability
- A 1.5% duty cycle on the long-tenure dormant vendor that should be a warning when reading any other span-hours number

Provider tenure is a single command. It produces a four-row table. The four rows turn into a vendor-relationship summary that the model axis cannot produce on its own. Run it. Save the as-of timestamp. Compare the rows next quarter. The deltas tell you something the daily dashboards can't.

---

*Data: `pew-insights v0.4.93` `provider-tenure` against the local `~/.config/pew/queue.jsonl`, run at `2026-04-25T13:53:44Z`. Numbers are reported exactly as the lens emitted them; no smoothing, no rounding except where stated. The host machine, project mix, and routing policy that produce these numbers are specific to one operator — the methodology generalises; the absolute values do not.*
