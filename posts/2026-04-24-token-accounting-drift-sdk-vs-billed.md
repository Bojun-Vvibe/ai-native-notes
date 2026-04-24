# Token Accounting Drift: When the SDK Number Doesn't Match the Bill

There is a quiet little disaster that happens to every team that runs an LLM
workload at scale for more than a quarter. They build a dashboard. The
dashboard reads `usage.prompt_tokens` and `usage.completion_tokens` off every
SDK response, sums them across the day, multiplies by a price per million,
and prints a number. Engineering trusts the number. Finance trusts the
number. Capacity planning trusts the number. Then the invoice arrives at the
end of the month and the invoice number is between 4% and 35% higher than
the dashboard number. Someone asks why. Nobody can answer for two weeks. By
the time anyone has a defensible answer, the next invoice has already
landed.

This post is about why the SDK-reported token count and the provider-billed
token count disagree, what the drift actually looks like in practice, and
how to design your accounting so the disagreement is observable instead of
mysterious.

## The naive model

The naive model is that the response object is the source of truth. Every
chat completion comes back with a `usage` block. The block has three or
four integers. You record them. Later you sum them. Tokens have a price.
Multiply. Done.

This model is wrong in roughly seven different ways and the wrongness
compounds. Let me walk through them.

## Drift source 1: cached input tokens

The first big source of drift is prompt caching. When the provider supports
prefix caching, a long stable prefix that was recently sent gets billed at a
fraction of the per-token rate — often 10% to 25% of the uncached price.
The SDK response will sometimes report this as a separate field
(`cached_tokens`, `prompt_tokens_details.cached_tokens`, or similar), and
sometimes it just folds it into the main `prompt_tokens` count. If your
dashboard is just summing `prompt_tokens` and multiplying by the
non-cached price, you will systematically over-estimate cost. If you are
multiplying by the cached price, you will systematically under-estimate
cost on requests that did not actually hit cache.

The drift here is bidirectional and depends on cache hit rate, which itself
depends on traffic shape. This is the worst kind of drift: it moves with
your workload, so a refactor that improves cache hit rate looks like a
revenue event in your dashboard while the provider's invoice barely budges.

## Drift source 2: reasoning tokens billed but not always reported

Reasoning models bill for the hidden chain-of-thought tokens. The SDK
sometimes reports these as `reasoning_tokens` inside a details block,
sometimes folds them into `completion_tokens`, and sometimes — depending
on which version of which SDK and which model — leaves them out of the
client-visible accounting entirely while still billing for them. A request
that visibly returns 200 tokens of answer might cost 4000 tokens of bill
because the model thought for 3800 tokens before answering.

If you don't have reasoning token accounting in your client-side meter,
your dashboard is going to underestimate completion cost by an order of
magnitude on reasoning-heavy traffic. The fix is to read the
`completion_tokens_details.reasoning_tokens` field if it exists, default
to "unknown, log a warning" if it doesn't, and never assume that visible
output bytes correlate with billed output tokens.

## Drift source 3: tool definitions are part of the prompt

Tool definitions get serialized into the prompt before it hits the model.
The SDK shows you the messages array. The messages array does not include
the serialized tool schema. The provider tokenizes the schema and bills
you for it. On a small message with twelve tools defined, the tools can
be 60% of the prompt tokens.

This drift is particularly nasty because tool schemas tend to grow over
time. Someone adds a new tool, someone else adds three more parameters to
an existing tool with verbose descriptions, and suddenly the average
prompt token count climbs 20% with no change in user-facing message
length. Your dashboard does not see this if it is computing tokens by
re-tokenizing the messages array client-side. The provider sees it. The
invoice reflects it.

The fix is to either (a) trust `usage.prompt_tokens` from the response
and never re-tokenize client-side for accounting purposes, or (b) include
the serialized tool schema in your client-side tokenizer pipeline. The
former is simpler. The latter is necessary if you want to do cost
budgeting before the call instead of after.

## Drift source 4: system prompt overhead added by the provider

Some providers wrap your prompt in a small envelope before tokenization:
a turn marker, a role marker, a few tokens of formatting. Some don't.
Some used to and stopped. Some do for chat endpoints and don't for
completion endpoints. This adds 4 to 40 tokens per request that you
cannot see in your message array. At a million requests a day, 20 invisible
tokens per request is 20 million tokens a day of phantom prompt that
shows up on the bill but never in your dashboard.

## Drift source 5: tokenizer version skew

The tokenizer used by the model is occasionally updated, especially across
model versions. If you pinned `tiktoken` to a specific version in 2024 and
the provider rolled out a new tokenization variant in 2025, your local
counts will be off by single-digit percentages on most strings and by
double-digit percentages on strings with a lot of recently-added special
tokens (think emoji, certain CJK sequences, code fence markers).

Re-tokenizing client-side for budgeting is fine if you treat the result as
an estimate within ±5% and never as an accounting source of truth. Re-
tokenizing client-side and feeding it into your billing pipeline guarantees
drift from day one.

## Drift source 6: retries and partial responses

Streaming responses that get cut off mid-way might or might not bill for
the tokens the provider already generated. SDK-level retries that succeed
on the second attempt typically get reported as a single response in your
application logs but billed as two attempts on the provider side. If you
have retry logic that runs above the SDK and below the application, the
"single response" the application sees can have hidden multiplier of 2,
3, or 4 depending on how your retry budget was configured.

The fix here is brutal but necessary: instrument at the lowest layer that
makes a network call. Every request to the provider is a billing event,
regardless of whether it ultimately succeeded or whether your application
ever saw it. If you only count successful responses, your accounting will
under-count by exactly the failure rate times the retry depth.

## Drift source 7: rounding and minimum bills

Some pricing tiers have minimum bills per request, or round up to the
nearest 1000 tokens, or charge for a minimum context length even on tiny
requests. Most teams ignore this because it sounds small. On a workload
dominated by a few high-token requests, it is small. On a workload
dominated by millions of tiny requests (think classification, routing,
embeddings, light tool calls), the rounding can be 5-15% of the bill.

## What the drift looks like in practice

Let me walk through a real-shaped example with fake numbers.

Suppose your dashboard says you spent $42,000 last month. The invoice says
$48,500. That is 15.5% drift. Where does it come from?

- Reasoning tokens not accounted: +6%. Your reasoning model traffic is 30%
  of requests and the average reasoning expansion is 3x completion tokens,
  but you were only counting visible completion.
- Tool schema tokens not accounted: +3%. Your tool catalog grew 40% in the
  quarter and you are tokenizing only the messages array.
- Cache hit rate over-credited: +2.5%. You assumed 70% cache hit rate
  based on a one-week sample. Actual hit rate is 58% because of a traffic
  shift toward novel prompts on weekends.
- Retries not accounted: +2%. Your retry layer adds 2% to request volume
  and you only count successful responses.
- Rounding and minimums: +1.5%. You have a high-volume classifier that
  hits the per-request minimum a lot.
- Tokenizer skew: +0.5%. Small but nonzero.

Total: +15.5%. Each piece is small. Together they are a budget hole.

## How to design accounting that doesn't lie

The principle is simple: **the bill is the source of truth, the SDK is a
preview, and your client-side count is an estimate**. Your accounting
pipeline should reflect this hierarchy.

In practice this means three layers:

**Layer 1: budgeting estimate.** Computed before the call, used for rate
limiting, cost guards, and "should I even make this call" decisions. This
is allowed to be wrong by up to 10% as long as the wrong direction is
known. I prefer over-estimates here so that the budget guard fails safe.

**Layer 2: SDK-reported usage.** Computed from the response object after
the call. This is what your real-time dashboards read from. It will drift
from the bill but it is fast, free, and per-request, which makes it the
right input for things like per-tenant cost attribution and live alerts.
Always log every field the response provides — not just the top-level
totals — because drift sources 1 and 2 only become observable if you
preserve the breakdowns.

**Layer 3: provider-billed reconciliation.** Daily or weekly, ingest the
provider's billing export and reconcile against the sum of layer 2. The
delta per day is your drift. Track the drift over time. A stable drift
of 12% is fine — you can multiply layer 2 by 1.12 for forecasting and
move on. A drift that swings between 4% and 28% week over week is a
signal that something in your traffic shape is changing in a way your
dashboard can't see, and you need to dig.

The reconciliation step is the one teams skip. It is also the one that
saves you from a quarter-end surprise. Build it on day one. It costs a
few hours of pipeline work and it pays for itself the first month it
catches a 10% drift you would otherwise have missed.

## Per-tenant attribution: the multiplier problem

If you run multi-tenant and you bill your customers based on token usage,
the drift becomes a margin problem instead of a forecasting problem.
Imagine you charge tenant A based on the SDK-reported tokens for tenant
A's requests. You apply a markup. You think you have 30% margin. The
provider invoice arrives 15% higher than the sum of all tenant
attributions. Your actual margin is 15%, not 30%, and you have no clean
way to claw back the missing 15% from individual tenants because you
already invoiced them.

The fix: every tenant attribution should include a "reconciliation
factor" that gets updated monthly based on actual drift. You attribute
SDK tokens to tenant A in real time, then once the bill lands, you
multiply each tenant's attributed cost by the observed drift factor and
that is the cost of record. Communicate this clearly in your pricing so
nobody is surprised when the post-reconciliation number is 12% above the
preview.

The alternative — pre-multiplying the SDK number by an estimated
drift factor — works fine until the drift factor changes, at which
point you are either over-billing or under-billing every tenant for a
month before you notice.

## Things that make the drift worse

A few patterns that I have learned to flag in code review because they
predictably blow up token accounting:

- **Re-tokenizing client-side and trusting the result for cost.** You
  will be wrong by tokenizer skew plus tool schema tokens plus envelope
  overhead. Use the response object.
- **Counting only "successful" requests.** Failed requests bill too. So
  do retries. Count at the network layer, not the application layer.
- **Aggregating to daily totals before logging.** You lose the ability
  to bucket by model, by tenant, by tool, by cache state. Log per-request,
  aggregate downstream.
- **Hard-coding prices in code.** Prices change. The price you encoded
  in February is not the price the provider charged in April. Pull from a
  config that is versioned and dated, and apply prices to billing periods
  not to clock time.
- **Trusting one provider's `cached_tokens` semantics across providers.**
  Each provider defines cache scope, cache TTL, and cache hit reporting
  differently. A multi-provider deployment needs a per-provider adapter
  that normalizes the fields, not a single struct that pretends they are
  all the same.

## A concrete checklist

If you are setting this up from scratch:

1. Log every numeric field in the `usage` block. Don't pre-aggregate.
2. Log per-request, per-model, per-provider, per-tool-call-count, per
   cache-status, per-tenant. The cardinality is fine if you keep the log
   short.
3. Instrument retries at the network layer, not the application layer.
4. Build the reconciliation pipeline in week one. Compare daily SDK
   totals against daily provider totals. Alert when the drift moves more
   than 3 percentage points week over week.
5. For each tenant, store an "attribution as of SDK" number and an
   "attribution post-reconciliation" number. Bill from the latter.
6. Maintain a price config that is dated and versioned. Run the
   reconciliation against the price that was active during the billing
   window, not the current price.

## Closing

The SDK number is not the bill. The SDK number is a preview of the bill,
and the preview is good enough for live dashboards and rate limiting and
budget guards. It is not good enough for invoicing your customers or
forecasting your quarterly spend. The gap between the SDK number and the
bill is a meaningful 5-20% on most workloads, and it changes as your
traffic shape changes.

Build the reconciliation early. Track the drift as a first-class metric.
When someone asks "how much did we spend yesterday," the answer is "the
SDK says X, and based on a 14-day rolling reconciliation factor of 1.12
we expect the bill to land near 1.12 * X." That is an honest answer. The
alternative — pretending the SDK number is the bill — works right up
until the month it doesn't, and the month it doesn't is the month
finance loses trust in the number forever.
