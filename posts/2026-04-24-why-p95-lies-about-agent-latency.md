# Why p95 Lies About Agent Latency (And What To Use Instead)

**Thesis:** p95 latency is the wrong primary metric for an agentic CLI. It was designed for stateless request/response systems where every request is roughly the same shape. Agent workloads violate every assumption that makes p95 meaningful. If you ship a dashboard with "p95 turnaround" as the headline number, you will systematically underweight the worst user experiences, miss regressions that matter, and chase noise that doesn't. This post explains the four ways p95 lies, gives a worked numeric example showing how a 30% degradation can hide entirely behind a flat p95 line, and proposes a concrete replacement: a small panel of *shape-aware* latency metrics that together tell you what's actually happening.

## Why we reach for p95 in the first place

p95 became the default tail-latency metric in the web era for good reasons. For a stateless HTTP service serving roughly homogeneous requests:

- The distribution is usually a unimodal long tail.
- The median is dominated by happy-path requests, which are uninteresting.
- The max is dominated by outliers (network blips, GC pauses, kernel scheduling), which are noisy.
- p95 (or p99) sits in the meaty part of the tail: bad enough to matter, common enough to be statistically stable.

For a search backend serving 50k req/s of "find document by id," p95 is great. You get a single number that summarizes "how bad is bad for a normal user." You can SLO it, alarm on it, and trend it.

Agents are not that workload. Not even a little.

## The four ways p95 lies for agent workloads

### Lie #1: The unit isn't comparable

Per-request p95 assumes requests are roughly the same. Agent "turns" are not. One turn might be:

- A 200-token "what's in this directory" question that resolves in 3 seconds with one tool call.
- A 40k-token codebase refactor that runs 18 tool calls, hits a model retry, and takes 4 minutes.

Computing p95 across both treats them as the same population. They aren't. The 4-minute turn isn't a tail of the 3-second distribution — it's a different distribution entirely. Mixing them gives you a number that's neither a good summary of "fast turns" nor of "slow turns."

The fix isn't to throw out p95. It's to **stratify before you summarize.** Bucket turns by some shape feature you can compute cheaply at the start of the turn (input tokens, declared task class, presence of tool calls in the system prompt) and report p95 per bucket. A flat aggregate p95 hides the fact that your "small turns" got 2x slower while your "large turns" got 30% faster — both might be real, both might matter, and neither shows up in the headline.

### Lie #2: The interesting thing is *inside* a turn

In a stateless service, "request started" and "response sent" bracket the user experience. In an agent loop, they don't. A 4-minute turn might include:

- 0–200ms: routing and prompt assembly
- 200ms–2.4s: model first-token latency (TTFT)
- 2.4s–8s: streaming tokens for an explanation
- 8s–8.1s: tool call dispatch
- 8.1s–95s: tool call execution (e.g., a test suite)
- 95s–98s: model second turn TTFT (with a cold cache because tool output blew the prefix)
- 98s–4min: more streaming, more tools, more retries

The user's perception of "slow" is not "total wall time." It's "how long did I stare at a frozen cursor?" That's the **maximum gap between visible progress events**, not the sum. A 4-minute turn with continuous token streaming feels fine. A 30-second turn with a 25-second silent gap feels broken.

p95 of total turn duration cannot see this. Two turns with identical totals can have wildly different felt latency.

The fix: instrument **inter-event gaps**, not just total duration. For each turn, record the max silent interval — the longest period with no tokens streamed, no tool started, no tool finished, no status update. Then report p95 of *that*. This is the metric that correlates with users typing "is it stuck?" in your bug tracker.

### Lie #3: Retries hide behind the mean

Agent loops retry. Tool calls retry on transient errors. Model calls retry on rate limits, on validation failures, on schema mismatches. A well-designed retry layer makes the *outcome* identical to the no-retry case — the user gets the right answer — but the *latency* is wildly different.

Here's the trap: if your retry rate is 2% and each retry adds 4 seconds, your *mean* latency goes up by 80ms. Your *p95* might not move at all if the retries are evenly distributed across the population. But the retries themselves are 4-second cliffs in the user experience, and they cluster — when the upstream model is degraded, you'll see retry rates spike from 2% to 25% in a 10-minute window, and *every single one of those users* feels a multi-second pause that didn't exist yesterday.

p95 over a rolling hour smooths this away. By the time the line moves, the incident is over.

The fix: track retry-attributable latency as its own series. For each turn, compute `retry_overhead_ms = total_ms - estimated_no_retry_ms`. Report the *count* of turns with non-zero retry overhead per minute, and the *sum* of retry overhead per minute. These are leading indicators. The headline p95 is a lagging indicator.

### Lie #4: Cancellations don't show up at all

When an agent turn is taking forever, users cancel. Ctrl-C, close the pane, kill the process. From the latency dataset's point of view, that turn either:

- Doesn't get logged (process died before flush)
- Gets logged with `status=cancelled` and a duration that's *shorter than it would have been*

Either way, the worst experiences are systematically underrepresented in your p95 calculation. The turns that drove users to give up are the ones that get filtered out.

This is the most insidious lie. Survivorship bias in your latency data means you're optimizing for the people whose patience exceeded your tail. The people you're losing don't appear in the metric you're trying to improve.

The fix: track cancellation rate as a first-class latency-adjacent metric. A turn that was cancelled at 45 seconds is a worse outcome than a turn that completed at 60 seconds. Weight your "experience" score accordingly. One simple model: define `effective_duration = completed_duration if status=='ok' else (cancelled_duration + penalty)` where `penalty` is some constant (say, 30 seconds) that captures "user gave up and now has to start over."

## A worked numeric example

Let's make this concrete. Suppose yesterday's distribution of 1000 turns looked like this:

| Bucket | Count | Mean (s) | p50 (s) | p95 (s) | Cancel rate |
|--------|-------|----------|---------|---------|-------------|
| Tiny (<2k tok) | 600 | 4.2 | 3.1 | 11 | 1% |
| Medium (2–10k) | 300 | 18 | 14 | 52 | 3% |
| Large (>10k) | 100 | 95 | 78 | 240 | 12% |
| **Aggregate** | **1000** | **17.4** | **5.2** | **78** | **3.0%** |

Today, after a model provider degradation that affects long-context calls more than short ones, the same 1000 turns look like:

| Bucket | Count | Mean (s) | p50 (s) | p95 (s) | Cancel rate |
|--------|-------|----------|---------|---------|-------------|
| Tiny | 600 | 4.3 | 3.1 | 11 | 1% |
| Medium | 300 | 22 | 16 | 70 | 5% |
| Large | 100 | 140 | 110 | 360 | 28% |
| **Aggregate** | **1000** | **22.1** | **5.3** | **80** | **4.6%** |

The aggregate p95 moved from 78s to 80s — a 2.6% degradation. You'd never page on that. You might not even notice it on a weekly review.

But look what actually happened:

- Large turns got 50% slower at the median and 50% slower at p95.
- Cancellation rate on large turns went from 12% to 28% — you lost 16 additional users today.
- Medium turns degraded too, just less dramatically.
- Tiny turns are unchanged, and they dominate the count, so they hold the aggregate p95 down.

If you'd alerted on per-bucket p95, large bucket would have tripped immediately. If you'd alerted on cancellation rate, you'd have been paged within the hour. If you'd alerted on aggregate p95, you'd be having this conversation with your users tomorrow morning.

## The replacement panel

Drop the headline p95. Replace it with a panel of five metrics, each of which captures a different failure mode:

**1. Per-bucket p95 (stratified turn duration)**
Bucket by input tokens or task class. Three to five buckets is enough. Alarm per-bucket, not on the aggregate. This catches Lie #1.

**2. p95 of max silent gap**
For each turn, compute the longest interval with no progress event. Take p95 across turns. This is the "is it stuck?" metric. Alarm at a tight bound (5–10 seconds for interactive use). This catches Lie #2.

**3. Retry overhead rate**
Sum of retry-attributable milliseconds per minute, plotted as a time series. Spikes in this series precede degradation in turn duration by minutes. This catches Lie #3.

**4. Cancellation rate, weighted by effective duration**
Don't just count cancellations — weight them by how long the user waited before giving up. A 60-second cancellation is much worse than a 5-second one. This catches Lie #4.

**5. Time-to-first-useful-output (TTFUO)**
Not first token — first token that's part of the actual answer, not a "thinking..." status or a tool-call preamble. This is the latency that actually predicts user satisfaction. It's harder to instrument, but you can approximate it: time from turn start to the first streamed token of an assistant message that isn't a tool call.

These five together cost roughly the same to compute as a single p95, because they all derive from the same per-turn event log. The difference is in *what you ask the log*.

## Implementation sketch

Assume your agent emits a JSONL event stream per turn:

```jsonl
{"t": 0.000, "turn_id": "abc", "event": "turn_start", "input_tokens": 8400, "task_class": "refactor"}
{"t": 0.180, "turn_id": "abc", "event": "model_request", "attempt": 1}
{"t": 2.310, "turn_id": "abc", "event": "first_token"}
{"t": 4.820, "turn_id": "abc", "event": "tool_start", "name": "edit"}
{"t": 4.910, "turn_id": "abc", "event": "tool_end", "name": "edit", "ok": true}
{"t": 5.040, "turn_id": "abc", "event": "model_request", "attempt": 1}
{"t": 7.220, "turn_id": "abc", "event": "first_token"}
{"t": 12.880, "turn_id": "abc", "event": "turn_end", "status": "ok"}
```

A small post-processor computes per-turn:

```python
def derive_metrics(events):
    bucket = pick_bucket(events[0]["input_tokens"])
    total = events[-1]["t"] - events[0]["t"]
    gaps = [b["t"] - a["t"] for a, b in zip(events, events[1:])]
    max_gap = max(gaps)
    retry_overhead = sum(
        e["t"] - prev_request_t
        for prev_request_t, e in find_retry_pairs(events)
    )
    ttfuo = first_non_tool_token_time(events) - events[0]["t"]
    status = events[-1]["status"]
    return {
        "bucket": bucket,
        "total": total,
        "max_gap": max_gap,
        "retry_overhead": retry_overhead,
        "ttfuo": ttfuo,
        "status": status,
    }
```

Aggregate across the rolling window of your choice. Plot the five metrics on one dashboard. Alarm on each independently with thresholds calibrated to its own variance.

## What about p99?

Same problems, just further out in the tail. p99 doesn't fix any of the four lies. It's still per-turn, still aggregate-prone, still blind to inter-event gaps and retries and cancellations. It just shifts which subset of "the bad stuff" you're looking at.

If you want a single tail number, p95 of *max silent gap* is a much better choice than p95 of *total duration*. It correlates better with user complaints, it's more sensitive to the failure modes you actually care about, and it's not dominated by the legitimately-long-but-fine "this turn ran a 4-minute test suite" cases.

## What about histograms?

A latency histogram is strictly more information than a percentile, and you should keep one. But "more information" isn't the same as "useful at a glance." Most on-call people will look at the histogram once during the incident and then ignore it. The headline number — whatever it is — is what gets watched. Make sure the headline number is one that can actually move when something breaks.

## The deeper point

p95 became default because it was the right answer to the question "how do I summarize a long-tailed distribution of homogeneous events with a single number?" Agent workloads are not a long-tailed distribution of homogeneous events. They're a mixture of distributions of heterogeneous, multi-stage, retry-prone, cancellable processes. The right summary isn't a single number, and the right metrics aren't all about duration.

The cost of getting this wrong isn't theoretical. It's that your dashboard says green while your users churn. It's that your weekly latency review concludes "we're flat" while your large-context users are quietly giving up. It's that you ship a regression and find out about it from a support ticket three weeks later.

Replace the single-number summary with a panel that knows the shape of the workload. The five metrics above aren't the only valid choice — you can swap TTFUO for "time to first tool result" if your agent is more action-heavy than chat-heavy, or add a "tool failure rate per turn" metric if your tool layer is the dominant source of pain. The point is to *match the metric to the failure mode*, not to inherit a number from a different era.

p95 isn't useless. It's just not the headline. Move it to the second row of the dashboard, label it "aggregate turn duration p95," and let the metrics that actually predict user pain take the top.
