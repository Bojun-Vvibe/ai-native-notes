# Cache hit ratio above 100% — when ratios stop being ratios

There is a moment, the first time you wire up token telemetry and
print a per-device or per-model "cache hit ratio" column, where the
number comes back at 146% and you assume you have a bug. You go look
at the divisor. You go look at the dividend. You add a clamp. You
add a warning. You add a unit test that asserts the value is in
`[0, 1]`. And somewhere in that ritual you have quietly thrown away
the most informative number on the screen.

This post is about why the ratio is allowed to exceed 100%, why
clamping it is the wrong move, and what it tells you about the
underlying accounting model. The motivating data point is concrete:
a real per-device telemetry slice on this host currently shows

```
device_id     tokens         input          cached         output      cache%
dev-37533d9d  8,237,898,930  3,329,072,339  4,873,378,646  34,349,311  146.4%
```

— `8.24B` total tokens across 869 active hours, and the "cache hit
ratio" column reports `146.4%` (`cached_input / input` = `4,873,378,646
/ 3,329,072,339`). That is a real number from a real queue and it is
not a bug. It is a load-bearing signal.

## What the ratio is actually computing

The instinctive mental model for "cache hit ratio" is the one from
HTTP caches and CPU caches: a request comes in, you check the cache,
it either hits or misses, and the ratio is `hits / (hits + misses)`.
That is bounded in `[0, 1]` by construction, because each request is
either a hit or a miss and never both.

LLM prompt caching does not work that way. The unit of accounting is
not a request — it is a token, and each token can be billed under
multiple columns at once depending on the provider's cost model.
There are at least three common shapes:

1. **Subset model.** `cached_input` is a strict subset of `input`.
   Total prompt bytes shipped on the wire equal `input`; some of
   those bytes happened to land on a cache hit and got the discount.
   Ratio is bounded `[0, 1]`. Most people's mental model.

2. **Disjoint-columns model.** `input` counts uncached prompt tokens
   only. `cached_input` counts cached prompt tokens, which the
   provider bills at a different rate (often 10× cheaper). Total
   prompt = `input + cached_input`. Ratio `cached_input / input` is
   unbounded above and is not a probability — it is a *mass ratio*
   between two non-overlapping populations.

3. **Read-vs-write model.** `cached_input` is the number of tokens
   read from a cache that was previously *written* during a prior
   call, and a separate `cache_creation_input` field counts the
   write side (often billed at a premium, e.g., 1.25× normal input).
   Three columns total. The "ratio" you usually want here is read /
   (read + write + uncached), not read / uncached.

The data shape on this host — and the data shape that ships in
`pew-insights` v0.4.46–v0.4.48 — is shape 2. `input_tokens` and
`cached_input_tokens` are *parallel disjoint counters*, not a parent
and a child. So `cached / input` of 1.464 means "for every uncached
prompt token billed, the workload also burned 1.464 cached prompt
tokens." That is enormously useful — it tells you cache is
load-bearing and is dominating prompt mass — and it is wrong by 46%
the moment you clamp it.

## Why providers diverged from the subset model

There are two real reasons, both tied to billing rather than
correctness.

**Distinct unit prices.** If cached input is billed at 0.1× of
uncached input, the simplest invoice is one column per price tier.
Subsetting `cached ⊂ input` would force the invoice to either show
the same token in two columns (confusing) or back out the cached
portion from `input` (an arithmetic step that has burned every
finance team that ever did it by hand). Disjoint columns make the
invoice trivially reconcilable: total cost = `input × p_input +
cached × p_cached + output × p_output`, no inclusion-exclusion.

**Cache write events.** When the provider runs a separate
"cache creation" tier, the same prompt token can legitimately be
counted three times across its lifetime: once as uncached input on
the first call (priced at 1.0×), once as a cache write (priced at
1.25× by the major U.S.-brand provider with the longest-running
cached-prompt API), and N times as cache reads on subsequent calls
(priced at 0.1×). A subset model literally cannot represent this —
the reads happen on calls where there is no parent `input` event.
The disjoint model just adds another column.

So the "ratio above 100%" is a downstream consequence of a billing
model that prioritized invoice arithmetic over taxonomic neatness.
Your telemetry tooling inherits that choice whether you like it or
not.

## What the 146% number actually tells you

Once you accept the disjoint accounting, the unbounded ratio
becomes one of the most diagnostic numbers you can read off a
workload. Three readings, all valid simultaneously:

**Cache is dominant prompt mass.** A ratio of 1.464 means cached
prompt bytes are ~59% of total prompt mass (`1.464 / (1 + 1.464)`).
The agent loop is reusing context — system prompt, tool schemas,
prior turns — far more often than it is shipping fresh context.
That is the *intended* shape for an agentic workload and the *wrong*
shape for a one-shot completion workload, so the same number tells
you what kind of workload you are looking at.

**Cache is doing real economic work.** Multiplying the disjoint
ratio by the price differential gives the realized discount. If
cached input is billed at 10% of uncached input, then the workload
is paying `1.0 × p + 1.464 × 0.1 × p = 1.146 × p` for prompt mass
that *would have cost* `(1.0 + 1.464) × p = 2.464 × p` at uncached
rates. Realized prompt discount: 53.5%. That is the number to put
in a cost dashboard, not the raw 146%.

**Cache is not yet "saturated."** In the subset model you can ask
"what fraction of prompts hit cache" and get a probability. In the
disjoint model you can ask the equivalent dual question — "for each
uncached token, how much cached mass am I getting?" — and the
answer in this slice is `1.464`. There is no a-priori ceiling, but
empirically a long-running agent loop with a stable system prompt
and growing conversation history will trend toward 5×–20× over
weeks. A ratio of 1.464 across 869 active hours is a workload that
is bursty and short-lived per turn, not a long-running session.
That is also actionable.

## The clamp is the bug

The seductive fix when you see 146% is to write

```ts
const cacheHit = Math.min(1.0, cached / input);
```

This is not a fix. It deletes information. Specifically it deletes
the *magnitude* of cache reuse — the most important variable in your
prompt economics — and replaces it with a saturated `1.0` that tells
you nothing about whether the workload is reusing context 1.05× or
20×. Worse, the clamp is not idempotent across aggregation: if you
clamp per-row and then sum, you get a different number than if you
sum and then clamp, and reviewers will not catch the difference
because both look "in range."

The defensive move people reach for next is *renaming*: call it
`cacheReuseRatio` or `cachedToUncachedRatio` and document that it
is unbounded. That is correct, but only goes halfway. The deeper
fix is to recognize that you have two distinct metrics that the
single name "cache hit ratio" was conflating, and to expose both:

- `prompt_cache_share = cached / (input + cached)`, bounded in
  `[0, 1]`, answers "what fraction of prompt mass is cached." This
  is the metric that behaves like a probability and is safe to
  compare across workloads of very different sizes.
- `cache_reuse_ratio = cached / input`, unbounded, answers "for
  each uncached prompt token, how much cached mass do I get." This
  is the metric that is sensitive to the *intensity* of reuse and
  saturates above 1.0 only in the trivial sense that it is a
  ratio of unequal populations.

`pew-insights` exposes the second under the column name `cache%`
and accepts that some readers will be confused on first encounter.
The accompanying `provider-share` and `device-share` reports also
let you back out the first by hand. The CHANGELOG is explicit
about the convention — see the v0.4.47 entry, which calls out
`cacheHitRatio = cached_input / input` and warns that it can exceed
1.0 because pew accounts uncached and cached as separate columns
rather than as a subset (`pew-insights/CHANGELOG.md`).

## Aggregation hazards

Even if you accept the unbounded ratio, aggregating it across rows
is its own minefield. Three traps that are easy to fall into:

**Mean-of-ratios vs ratio-of-sums.** `mean(cached_i / input_i)` is
*not* `sum(cached) / sum(input)`. The first weights each row
equally regardless of size; the second weights by `input_i`. For a
workload with a long tail of tiny rows (which is normal — most
queue rows are sub-1k-token), the first is dominated by noise from
zero-input rows and rows where the prompt was almost entirely
cached. The second is the only one that recovers the fleet-level
economics. Default to ratio-of-sums; expose mean-of-ratios only if
the user explicitly asks for a per-row distribution view.

**Zero-input rows.** When `input_i = 0` (entire prompt was a cache
hit and the provider rounded uncached input to zero), the per-row
ratio is undefined. The instinct to drop those rows from the
denominator is wrong — you have just thrown away the rows where
cache was *most* effective. The instinct to substitute `+1` to the
denominator (Laplace smoothing) is also wrong, because the
denominator is in tokens, not events, and `+1 token` does not
correspond to anything in the underlying domain. Correct treatment:
keep the row, contribute `cached_i` to the numerator and `0` to the
denominator, and report ratio-of-sums on the aggregate. The result
is then mathematically well-defined even when individual rows are
not.

**Cross-provider aggregation.** Different providers use different
shapes (subset vs disjoint vs read-vs-write). If you aggregate
`cached / input` across a mixed-provider workload, you are summing
populations that were not collected under the same schema. The fix
is to normalize at ingest, not at aggregation: pick one canonical
shape (disjoint columns are the most expressive — they can
represent the others losslessly) and convert everything to it on
the way in. This is what makes the per-device cross-provider view
on this host coherent: the queue normalizes all six observed
sources into the disjoint shape before any rollup happens.

## Reading the 146% number on this host

So, last reading: this host's per-device slice on 2026-04-24 shows
`cache% = 146.4%` over `8,237,898,930` total tokens spread across
`1,391` rows and `869` active hours, single-device, six distinct
sources, fifteen distinct models. Translating into the framework
above:

- Prompt cache share: `4,873,378,646 / (3,329,072,339 +
  4,873,378,646) = 59.4%`. Roughly three of every five prompt
  tokens shipped were on a cache hit. That is high but not
  saturating; comparable single-device long-running setups
  routinely run 70%+ once a stable system prompt and tool schema
  are in place.
- Realized prompt discount (assuming 10% cached pricing): `(1 +
  0.146 × 10) / (1 + 1.464) = ~99.4%` of the uncached-only cost
  becomes `~46.5%`. Order of magnitude: cache is roughly halving
  prompt cost on this workload.
- Output mass is `34,349,311` tokens against `8,202,450,985` prompt
  tokens — a `~239:1` prompt-to-output ratio, which dovetails with
  the broader observation that completion-tail mass is a tiny
  fraction of total spend on agentic workloads. (See the existing
  `completion-tail-economics-where-output-mass-lives` post for the
  general argument.)

None of those readings are accessible if the column got clamped at
100%. The 146% is not noise to be hidden behind a `Math.min`; it is
the signal.

## Practical recommendations

1. **Do not clamp.** If your dashboard or report shows a "cache hit
   ratio" above 100%, surface it. Add a footnote about the
   accounting model the first time the user sees the column;
   readers learn the convention quickly and the magnitude carries
   information you cannot recover after the fact.

2. **Expose two columns, not one.** Prompt-cache-share (bounded,
   probability-shaped) and cache-reuse-ratio (unbounded,
   intensity-shaped) answer different questions. Picking one
   silently is the choice that produces the most confused users
   downstream. Picking both is mildly redundant and far more
   defensible.

3. **Normalize at ingest.** If you ever plan to aggregate across
   providers, decide on the canonical shape (disjoint columns are
   the safest target) and convert during ingestion. Aggregating
   raw provider-shape data is a class of bug that surfaces months
   later as "the dashboard numbers don't tie out to the invoice."

4. **Use ratio-of-sums for fleet aggregates.** Default the headline
   metric to `sum(cached) / sum(input)`. Reserve mean-of-ratios for
   per-row distributions and label it as such. The two will diverge
   substantially on any workload with heavy-tailed row sizes, which
   is essentially every real workload.

5. **Treat zero-input rows as data, not edge cases.** They are the
   rows where caching worked best. Keep them in the numerator,
   keep them out of the denominator, and report on the aggregate.
   Per-row "undefined" handling is a display concern, not an
   arithmetic one.

The throughline: when a number you expected to be a probability
turns out to be a mass ratio, the answer is not to make it a
probability. The answer is to admit you have a mass ratio and
report what mass ratios actually tell you. Cache hit ratios above
100% are the canonical case where the ratio is more honest than
the name.

## Citations

- `pew-insights/CHANGELOG.md` v0.4.47 — 2026-04-25 — live-smoke
  output against `~/.config/pew/queue.jsonl`: `8,235,256,800`
  tokens across 1,391 rows, single device, `cache%` reported as
  `146.3%` with explicit note that "the real fleet shows `cache%`
  at 146.3% — `cached_input_tokens` exceeds `input_tokens` because
  pew's queue accounts uncached prompt bytes and cached prompt
  bytes as separate columns rather than as a subset."
- `pew-insights/CHANGELOG.md` v0.4.48 — 2026-04-25 — same workload
  one day later: `8,237,898,930` tokens, `cache% = 146.4%`,
  identical column shape (the post-redact run is byte-identical on
  every column except `device_id`).
- Test suite count cited above (753 total at v0.4.48, 750 at
  v0.4.47, 737 at v0.4.46) is from the same CHANGELOG entries and
  confirms the disjoint-column accounting is exercised under unit
  test, including the `cacheHitRatio = 0` edge case for zero-input
  rows.
