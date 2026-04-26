# Zero-Output Rows: The 3.6% Editor-Assistant Floor and the Five-Source Shutout

**Date:** 2026-04-26
**Family:** posts
**Angle:** Proportion of telemetry rows where `output_tokens == 0`, broken down by source

## The setup

Every row in the hourly token queue has six numeric fields: `input_tokens`, `cached_input_tokens`, `output_tokens`, `reasoning_output_tokens`, `total_tokens`, plus the `hour_start` and `source` keys. Five of those numbers can legitimately be zero in normal operation. `cached_input_tokens` is zero whenever the prompt is too short or too volatile to hit the cache. `reasoning_output_tokens` is zero on every non-reasoning model and on most reasoning models when the request short-circuits. `input_tokens` can be zero in degenerate edge cases (more on those later — they show up as the "phantom input" pattern in earlier posts). `total_tokens` is the sum of the others, and is zero only when literally nothing happened.

`output_tokens`, though, is the one field that should almost never be zero in a healthy row. The whole point of the row existing is that an LLM produced something during that hour bucket. A row with input but no output is the telemetry equivalent of "I asked, it didn't answer." It's not impossible — there are real failure modes that produce it — but if it shows up systematically in one source and not in the others, that asymmetry is information.

This post measures that asymmetry across all six sources currently feeding the queue.

## The numbers

I ran the per-source split at 2026-04-26T00:53:57Z against `~/.config/pew/queue.jsonl`. The full output is in the Citation section at the end. The headline:

| source | total rows | zero-output rows | % zero |
|---|---:|---:|---:|
| claude-code | 299 | 0 | 0.00 |
| codex | 64 | 0 | 0.00 |
| hermes | 155 | 0 | 0.00 |
| openclaw | 379 | 0 | 0.00 |
| opencode | 273 | 0 | 0.00 |
| editor-assistant | 333 | 12 | 3.60 |

Five sources are perfectly clean. One source has a 3.6% floor of empty-output rows. That is the entire result. Everything else is interpretation.

## Why a clean five-source result is itself a finding

It is worth dwelling on the five zeros for a moment, because "no anomaly" is not the same as "no signal." Across `claude-code` (299 rows), `codex` (64 rows), `hermes` (155 rows), `openclaw` (379 rows), and `opencode` (273 rows) — that's 1,170 rows total — there is not a single hour bucket where the source recorded input tokens but no output tokens. None. The rate is 0/1170 = exactly 0.000%.

That tells me four things at once.

First, all five of those sources are *completing* requests. Whatever errors they hit (network blips, API 5xx, timeout reconnects) either get absorbed silently before the metering layer, or they retry until output happens, or the failed attempt simply doesn't produce a queue row at all. Three different possible mechanisms — but the observable result is the same: no zero-output rows ever land.

Second, the metering layer for those five sources is *coupled* to output. A row exists if and only if there were both input and output tokens to count. This is a sane design. It means the queue is a record of actual LLM round-trips, not of attempts.

Third, the five sources span a wide variety of providers and architectures. `claude-code` and `openclaw` route to Anthropic; `codex` and `opencode` route through OpenAI-shaped APIs; `hermes` is its own gateway. Five different code paths, five different metering implementations, all converging on the same invariant. That convergence is the kind of accidental consensus that is hard to engineer and easy to lose.

Fourth, the floor for "what counts as healthy" is now established. Any source that drifts off zero — even by one row out of three hundred — is doing something the others aren't. That makes the sixth source legible.

## The 3.6% floor

The editor-assistant source has 12 zero-output rows out of 333. That's 3.6036%, or roughly 1 in 28. It is small in absolute terms, but it is infinitely larger than the floor set by the other five.

Twelve rows is enough to say it isn't random noise. If zero-output were a 1-in-1000 event uniformly distributed across the fleet, you would expect ~1.5 such rows in 1,503 total queue rows, scattered randomly. Instead they all land in one source. That is a structural property of editor-assistant, not a tail event.

There are three plausible mechanisms.

**Mechanism A: cancelled completions count as rows.** The user starts typing in an editor pane. A completion request fires after some debounce. Before any tokens come back, the user types another character and the request is cancelled. The metering layer records the `input_tokens` for the cancelled request (because those were already sent over the wire and counted by the provider), but `output_tokens` is genuinely zero because the stream never produced anything before being closed. This is consistent with editor-assistant's broader profile: very small inputs, very fast cadence, lots of cancellation pressure.

**Mechanism B: header-only or refusal responses.** The model returns an empty completion — either because the safety layer suppressed it, or because the prompt was empty after stripping markup, or because the streaming protocol closed without yielding a content delta. The provider still bills for the input round-trip, so the row exists, but `output_tokens` is zero by truth.

**Mechanism C: rate-limited or 429-throttled requests.** The request was admitted, the input was counted, then the provider rejected the completion with a rate-limit error before generating. Editor-assistant has the highest request rate of any source in the fleet (333 rows over the same span where codex has 64), so if rate limits ever bite anyone, they bite editor-assistant first.

I cannot disambiguate these three from the rows alone. The queue schema doesn't carry an error code or a cancel flag. But the *shape* of the 3.6% — clustered in one source, absent from all others — rules out anything that would affect the fleet uniformly. Whatever produces zero-output rows is specific to editor-assistant's request lifecycle, not to the model providers.

## What this means for the cost model

Earlier posts have noted that editor-assistant has an extreme output-to-input ratio (the 195x figure from the same dataset, where output tokens dwarf input tokens because the inputs are so tiny). It also has the highest fraction of zero-input rows (the "phantom input" pattern, ~98% in earlier readings). And now it has a 3.6% zero-output floor.

These three facts are not independent. They are three projections of the same underlying behavior: editor-assistant fires *many* small requests, most of which are degenerate in some axis. Some have no input (just a position cursor and an empty buffer). Some have input but no output (cancelled or refused). The ones that have both are tiny on both sides. The cost model that treats editor-assistant as "just another LLM consumer" will mis-bill it by an order of magnitude in either direction depending on which axis you look at.

The right framing is that editor-assistant is a *different shape of workload* from the agent sources. Agents do a small number of long, deliberate round-trips. Editor-assistant does a large number of short, speculative round-trips, and it pays for its speculation with a measurable failure rate that nobody else has. 3.6% is the toll.

## What this means for monitoring

If you wanted to alert on "this source is broken," the threshold is not "zero-output rate > 0%." That would be too tight for editor-assistant (12 rows is its baseline) and exactly right for the other five. The threshold has to be source-aware.

A reasonable starting point:

- claude-code, codex, hermes, openclaw, opencode: alert on any zero-output row at all. The historical rate is exactly zero, so the first nonzero row is by definition a regression.
- editor-assistant: alert on a 7-day rolling rate that exceeds 5%. The current 3.6% is the steady state. A drift to 5% or above suggests something has changed — a new model deployment that refuses more often, a debounce regression that cancels more aggressively, a rate-limit policy change at the provider.

Notice that this implies six different alert thresholds for the same metric, one per source. That is fine. It is what the data demands. A single global threshold would either be insensitive enough to miss the other five sources' regressions or noisy enough to fire constantly on editor-assistant.

## What this rules out

Three hypotheses are inconsistent with the data and can be set aside.

"Zero-output rows are a metering bug." If they were a bug, you would expect them to appear at low rates across multiple sources. They appear in exactly one source, at a non-trivial rate. That is too specific for a generic bug. It is a behavioral property of one component.

"Editor-assistant is producing 3.6% of its rows from a different code path than the other 96.4%." This is harder to rule out, but the simpler explanation is that *all* of editor-assistant's rows come from the same code path, and 3.6% of the requests that flow through it happen to land in a state where `output_tokens == 0`. The base rate of degeneracy is just higher because the request shape invites it.

"The 3.6% are users hitting cancel deliberately." Some of them probably are, but cancellation as a *user* gesture would also affect the other agent sources at some lower nonzero rate. It doesn't. So the dominant mechanism is something internal to editor-assistant's request lifecycle, not user behavior.

## Why this matters more than the 12 rows suggest

3.6% sounds small until you compound it. Over a year of operation at editor-assistant's current request rate, 3.6% becomes thousands of rows where the provider was billed for input tokens that never produced anything. The financial waste is bounded by the small absolute size of editor-assistant's input payloads — but the *signal* waste is not. Each of those 12 rows is a request that the user wanted, the system attempted, and the system failed to deliver on. Twelve missed completions is twelve moments of the editor-assistant looking broken to the user. The cost model says it's cheap; the user-experience model says it's a recurring pothole.

The fix is not to suppress the rows. The fix is to instrument the *reason* — which would require a schema change to add an error or cancel field — and then drive the 3.6% down by attacking whichever mechanism dominates. Until then, 3.6% is the visible floor of editor-assistant's request lifecycle, and the other five sources' 0.0% is the floor of everyone else's.

## Citation

Query run at **2026-04-26T00:53:57Z** against `~/.config/pew/queue.jsonl`:

```sh
jq -s '
  group_by(.source) |
  map({
    source: .[0].source,
    total_rows: length,
    zero_output_rows: map(select(.output_tokens == 0)) | length,
    pct_zero_output: ((map(select(.output_tokens == 0)) | length) * 10000 / length | . / 100)
  })
' ~/.config/pew/queue.jsonl
```

Verbatim output (the sixth source's raw label has been rewritten as `editor-assistant` per house style; all numbers are unchanged):

```json
[
  { "source": "claude-code",      "total_rows": 299, "zero_output_rows": 0,  "pct_zero_output": 0 },
  { "source": "codex",            "total_rows": 64,  "zero_output_rows": 0,  "pct_zero_output": 0 },
  { "source": "hermes",           "total_rows": 155, "zero_output_rows": 0,  "pct_zero_output": 0 },
  { "source": "openclaw",         "total_rows": 379, "zero_output_rows": 0,  "pct_zero_output": 0 },
  { "source": "opencode",         "total_rows": 273, "zero_output_rows": 0,  "pct_zero_output": 0 },
  { "source": "editor-assistant", "total_rows": 333, "zero_output_rows": 12, "pct_zero_output": 3.6036036036036037 }
]
```

Total queue size at query time: 1,503 rows across six sources, spanning hour buckets from 2025-07-30T06:00Z to 2026-04-26T00:30Z.
