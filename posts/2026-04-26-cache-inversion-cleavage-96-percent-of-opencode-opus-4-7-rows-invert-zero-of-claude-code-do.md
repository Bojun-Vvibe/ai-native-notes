# The cache-inversion cleavage: 96.7% of opencode|opus-4.7 rows report cached>input, 0% of claude-code|opus-4.7 do

There is a published post in this notes repo on the aggregate
cache:input ratio — the observation that on the local pew queue,
the per-source ratios for prompt-cached input vs raw prompt input
go up to 13.9× when measured at the source level. That post made
its argument from totals. This post is the per-row, per-pair
follow-up that the totals were hiding.

The headline result, computed from the same `~/.config/pew/queue.jsonl`
snapshot at `2026-04-26T06:26:27Z` (1527 rows total):

- Of 214 `opencode|opus-4.7` rows, 207 (96.7%) report
  `cached_input_tokens > input_tokens`.
- Of 81 `claude-code|opus-4.7` rows, 0 (0.0%) do.
- Of 153 `hermes|opus-4.7` rows, 74 (48.4%) do.
- Of 64 `codex|gpt-5*` rows, 0 (0.0%) do.
- Of 390 `openclaw|gpt-5*` rows, 0 (0.0%) do.
- Of 333 `ide-assistant-A` rows, 0 (0.0%) do.

The phenomenon — `cached_input_tokens` exceeding `input_tokens` —
is not random and is not uniform. It is concentrated in two
sources (`opencode` and `hermes`). The other four sources never
exhibit it, even though some of them are calling the same model
weights. The cleavage is not at the model level. It is at the
*driver* level, and it tells you what each driver thinks
"cached_input_tokens" actually means.

## Why the inversion happens at all

In a strict-arithmetic accounting of the prompt, you cannot have
`cached_input_tokens > input_tokens`. The cached portion is a
subset of the prompt. The set arithmetic is `cached ⊆ input`,
which forces `|cached| ≤ |input|`. So when the field exceeds the
parent set, you are not looking at a subset relation. You are
looking at two different *event counts* that have been put in
fields with overlapping names.

Two distinct accounting models can produce the inversion:

**Model A (Anthropic explicit prompt cache, opencode/hermes
style).** When a harness uses Anthropic's prompt-caching feature,
the API distinguishes `cache_creation_input_tokens`,
`cache_read_input_tokens`, and `input_tokens`. The `input_tokens`
field becomes the *un-cached* portion only — the new tokens
appended after the cache breakpoint. A cached prefix of 100K
tokens, plus a 5K append for this turn, produces
`input_tokens = 5000` and `cache_read_input_tokens = 100000`.
If a logger flattens `cache_read_input_tokens` into a generic
`cached_input_tokens` field and leaves `input_tokens` as the
un-cached suffix, the recorded ratio is 100000/5000 = 20×. That
is an entirely correct accounting; the field names just lost
their parent-child relationship in transit.

**Model B (OpenAI auto-cache, codex/openclaw style).** OpenAI's
prompt-caching is automatic and the API reports
`prompt_tokens_details.cached_tokens` as a *count of tokens that
were served from cache*, where `prompt_tokens` is the *total* of
cached + un-cached. The set relation holds: `cached ≤ prompt`.
Hit ratios live in [0, 1] and are interpretable as cache hit
rates. The maximum value in the local data for these sources is
0.96 (`codex|gpt-5*`), which reads as "96% of prompt tokens were
served from cache" — perfectly bounded.

The presence or absence of inversion is therefore a *signature
of which API the harness is talking to* and *how it serializes
the response*. It is not a bug, and it is not a feature. It is a
schema fact.

## Per-source, per-pair detail

The per-source counts:

```
source         | rows | inverted | inv_share
---------------+------+----------+----------
claude-code    |  299 |        0 |    0.00%
codex          |   64 |        0 |    0.00%
hermes         |  157 |       77 |   49.04%
openclaw       |  390 |        0 |    0.00%
opencode       |  284 |      221 |   77.82%
ide-assistant-A|  333 |        0 |    0.00%
```

The cleavage is sharp. Three sources never invert; three sources
invert often. There is no in-between. No source sits at, say, 5%
inversion or 30% — the floor is 0.0% and the next observation is
48.4%.

This is what a *schema cleavage* looks like in observational
data. If the inversion were caused by model behavior (e.g., a
model occasionally returning a misreport), you would expect a
noisy distribution: a few percent here, a few percent there,
correlated weakly with traffic. The actual distribution is
binary at the source level. Either the source is plumbed into
the inverting accounting, or it is not.

## Same model, different report

The cleanest test is to fix the model and watch what the
harnesses do. `claude-opus-4.7` (and the legacy hyphenated form
`claude-opus-4-7`) appears in three sources with enough rows to
be meaningful:

```
source       | model    | rows | in_tok       | cached_tok    | ratio | inv_share
-------------+----------+------+--------------+---------------+-------+----------
opencode     | opus-4.7 |  214 |   163794592  |  2664647439   | 16.27 |   96.7%
hermes       | opus-4.7 |  153 |    54660626  |    80824327   |  1.48 |   48.4%
claude-code  | opus-4.7 |   81 |  1159017656  |  1091902015   |  0.94 |    0.0%
```

Same model. Three sources. Three different paradigms.

`opencode` reports the un-cached suffix as `input_tokens`, so the
denominator is small and the ratio explodes. Per-row inversions
are the rule (96.7%), not the exception. The mean per-row
inversion magnitude on this pair is ~16×; the maximum recorded
is 69× (a 2026-04-23T17:30Z row with `input_tokens = 45383` and
`cached_input_tokens = 3132549`).

`hermes` reports the same model with about a 1.5× aggregate
ratio and a 48.4% per-row inversion share. So `hermes` is using a
*partially* inverting accounting: some rows report the cached
suffix style, others do not. This is consistent with `hermes`
being a router or proxy that aggregates calls from multiple
upstream agents, some of which report Anthropic-style and some
of which do not.

`claude-code` reports the same model with a 0.94 aggregate ratio
and 0% inversion. Even though it is calling Anthropic's API, it
records the *total* prompt size (cached + un-cached) in
`input_tokens` and reports the cache as a subset. Aggregate
ratios in [0, 1] only. This is the bounded paradigm.

## What the cleavage tells you about each harness

The 0% sources cluster cleanly:

- **`claude-code`** — Anthropic API, but the harness is reporting
  the bounded paradigm. Either it is using a custom serialization
  layer that re-inflates `input_tokens` to include cached, or it
  is using a SDK version that flattens the cache breakpoint into
  the prompt total before logging. Either way, the consumer of
  this telemetry sees a ratio in [0, 1] that is meaningfully a
  hit-rate.
- **`codex`**, **`openclaw`** — OpenAI API. OpenAI's
  `cached_tokens` field is naturally bounded by `prompt_tokens`,
  so the inversion cannot occur. `codex|gpt-5*` reports a ratio
  of 0.96 (a real 96% hit rate) and `openclaw|gpt-5*` reports
  0.86. These are interpretable.
- **`ide-assistant-A`** — input-side accounting is hidden inside
  the IDE context engine and does not surface, so both
  `input_tokens` and `cached_input_tokens` come through as 0 for
  the gpt-5*, gemini, and sonnet rows. The ratio is undefined,
  not zero. This source is omitted from the inversion analysis
  on the grounds that the data is not present.

The high-inversion sources cluster cleanly the other way:

- **`opencode`** — Anthropic API with explicit prompt-caching, and
  the harness reports the unmodified shape: `input_tokens` is the
  un-cached suffix, `cached_input_tokens` is `cache_read_input_tokens`.
  The ratio is unbounded above. Hit rates here are *not*
  interpretable as fractions; they are interpretable as the size
  of the cached prefix relative to the new turn.
- **`hermes`** — partial inversion. The per-row distribution
  (48.4% inverted) suggests a heterogeneous upstream: some calls
  flow through inverting accounting and some through bounded
  accounting. Interpreting the ratio without first separating the
  two regimes will give you a number that is wrong on both sides.

## Why this matters more than the aggregate post let on

The previous post on aggregate inversion concluded: "the
accounting is lying to you." This per-row decomposition is a
stronger version of that claim. If you are computing *average
hit rate* across all sources by summing `cached_input_tokens`
and dividing by `sum(input_tokens)`, the answer is wrong by
construction. You are summing apples (a hit rate, bounded by 1)
and oranges (a prefix-size ratio, unbounded above) and dividing.
The result is uninterpretable.

The right pre-aggregation move is to *partition by source* (or
better, by `(source, model)` pair) and to choose a separate
metric for each partition based on the schema:

- For sources where `cached ≤ input` always holds: report
  `cached / input` as a hit rate.
- For sources where `cached > input` is the rule: report
  `cached / (cached + input)` as a hit rate (since `cached + input`
  is the closest available proxy for the *total* prompt size in
  the inverting paradigm), and report `cached / input` separately
  as a "prefix amplification factor."
- For sources where the cleavage is heterogeneous (`hermes`):
  partition further by some upstream identifier if available, or
  acknowledge that the metric is a mixture of two regimes.

Doing it this way for the 2026-04-26 snapshot gives:

```
source         | model     | metric                | value
---------------+-----------+-----------------------+-------
opencode       | opus-4.7  | prefix amplification  | 16.27
opencode       | opus-4.7  | hit rate (proxy)      | 0.942
hermes         | opus-4.7  | prefix amplification  |  1.48
hermes         | opus-4.7  | hit rate (proxy)      | 0.596
claude-code    | opus-4.7  | hit rate              | 0.942
codex          | gpt-5*    | hit rate              | 0.964
openclaw       | gpt-5*    | hit rate              | 0.857
```

The hit-rate-proxy column for `opencode|opus-4.7` is 0.942 —
*identical* to the hit rate for `claude-code|opus-4.7` (also
0.942). That coincidence is the validation. Both harnesses are
calling the same model, doing the same kind of work, and getting
the same effective cache-hit performance. The naïve metric
(`cached / input`) would have told you they were 17× apart. The
partition-aware metric tells you they are operationally
identical.

That's the entire reason this post exists: the cleavage is a
schema artifact, and once you correct for it, the underlying
operational behavior is *much* more uniform than the raw numbers
suggest. The harnesses are not actually performing differently.
They are reporting differently.

## A second-order observation: the per-row inversion magnitudes are bursty

The top five inverting rows in the entire 1527-row corpus
(filter: `input_tokens > 1000`):

```
ratio  | source   | model      | in     | cached    | hour
-------+----------+------------+--------+-----------+-----
69.02  | opencode | opus-4.7   |  45383 | 3132549   | 2026-04-23T17:30Z
66.02  | opencode | opus-4.7   | 106375 | 7023164   | 2026-04-22T08:30Z
65.87  | hermes   | opus       |  30632 | 2017731   | 2026-04-17T06:30Z
60.39  | opencode | opus-4.7   |   8348 |  504124   | 2026-04-24T00:00Z
58.74  | opencode | opus-4.7   |  19698 | 1157100   | 2026-04-23T22:00Z
```

The 69× row had a cached prefix of ~3.1M tokens served against
a new-turn append of ~45K tokens. That is, plausibly, a long
agent conversation where the cache breakpoint covered the entire
accumulated history and only the most recent user turn flowed
through as un-cached. The 60× row at `2026-04-24T00:00Z` is
notable for a small absolute cached number (504K) divided by an
even smaller un-cached number (8.3K) — same shape, different
size.

The bursty nature matters because it means the per-row ratio is
a function of the *conversation length at that moment*, not of
the harness's steady-state caching policy. A long-running session
will accumulate ratio over time; a short session will show low
ratios even from the same harness. The aggregate ratio (16.27 on
`opencode|opus-4.7`) is therefore a session-length-weighted
average, not an intrinsic property of the harness. If your
sessions are systematically longer than mine, your aggregate will
be higher; if shorter, lower.

This is why "cache hit ratio" is not the right name for this
metric in the inverting paradigm. The number is at least three
things multiplied together: cache-policy effectiveness,
session-length distribution, and harness reporting paradigm. The
schema cleavage captured here is just the third factor — the
easiest one to detect, the easiest one to correct, but on its
own, the smallest of the three.

## Predictions falsifiable by next snapshot

Three predictions, each falsifiable in the next ~24h of data:

1. The 0%-inversion sources (`claude-code`, `codex`, `openclaw`,
   `ide-assistant-A`) will continue to show 0% inversion in
   tomorrow's snapshot. If any of them flip to a non-zero
   inversion share, it is a harness or SDK upgrade that should
   be visible in the source's commit log.
2. `opencode|opus-4.7`'s inversion share will stay above 90%.
   The 96.7% measurement here is high enough that ordinary
   sampling variance can move it ±2pp without crossing 90%.
3. `hermes`'s inversion share will *not* converge to either 0%
   or 100% within the next 30 days, because `hermes` is a
   genuinely heterogeneous mixture of upstream calls. If it
   does converge, the cause will be a routing change in the
   harness, not a model or API change.

If any of these predictions fail, the explanation will live in
the harness source tree, not in the model.

## The takeaway

The aggregate cache-inversion phenomenon noted in the prior post
turns out to have a sharp per-source structure. Three sources
exhibit it constantly (`opencode` 77.8%, `hermes` 48.4%,
`opencode|opus-4.7` 96.7%). Three sources never exhibit it. The
cleavage is binary at the source level: there is no source in
the corpus with an inversion share between 0.1% and 48%.

The cleavage is a schema artifact, not an operational difference.
Once you partition by source and apply the right denominator for
each schema, the effective cache-hit rate for `opencode|opus-4.7`
converges to 0.942 — the same as `claude-code|opus-4.7` (also
0.942) on the same model weights. The 16× headline ratio
disappears. What looked like a giant operational gap was a giant
*reporting* gap.

The lesson is the lesson the prior post tried to teach but could
not, because it was looking at totals: never aggregate a metric
across sources without first checking that the metric is
*denominated the same way* in each source. For prompt caching,
it is not. The sources fall cleanly into two paradigms, and
mixing them produces a number that is neither a hit rate nor a
prefix-amplification factor — just an arithmetic mean of two
incompatible quantities.

---

*Cited data: `~/.config/pew/queue.jsonl`, 1527 rows, snapshot at
2026-04-26T06:26:27Z. Per-source inversion shares computed
via `jq` group_by on a row-level boolean
`cached_input_tokens > input_tokens`. Top per-row ratios filtered
to `input_tokens > 1000` to avoid divide-by-near-zero blowups.
The label `ide-assistant-A` redacts one internal source name per
local content policy; the underlying source has all-zero
input/cached fields and is excluded from the inversion analysis
on those grounds.*
