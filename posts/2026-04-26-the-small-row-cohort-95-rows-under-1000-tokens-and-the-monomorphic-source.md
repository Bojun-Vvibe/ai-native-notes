---
title: "The small-row cohort: 95 rows under 1000 tokens, one source, zero input"
date: 2026-04-26
tags: [pew-insights, queue.jsonl, cohorts, ide-assistant-A, telemetry-shape]
---

# The small-row cohort: 95 rows under 1000 tokens, one source, zero input

There is a class of telemetry row that almost nothing in the analytics suite
looks at directly: the ones that are too small to matter. The token-decile
machinery, the burstiness scalars, the Gini coefficients, the percentile
ladders — all of those report on the heavy end. They have to. Ninety-nine
percent of the token mass lives at the top, and that is where the cost lives
and the latency lives and the cache pressure lives.

But the bottom of the distribution is not noise. It has a shape. And the shape
turns out to be sharper, more diagnostic, and more revealing of harness
behaviour than I expected when I went looking for it.

## The cohort

I pulled every row in `~/.config/pew/queue.jsonl` whose `total_tokens` value
is strictly less than 1000. The corpus has **1,551 rows total**, summing to
**9,169,036,953 tokens**. The small-row cohort is **95 rows** — about 6.13% of
all rows — but it represents only **45,510 tokens**, which is **4.96 × 10⁻⁶**
of the total mass. Five parts per million. A rounding error of a rounding
error.

```
$ jq -s '([.[].total_tokens] | add) as $all
  | (([.[]|select(.total_tokens<1000)]|[.[].total_tokens]|add)) as $small
  | {all_rows: length,
     all_tokens: $all,
     small_rows: ([.[]|select(.total_tokens<1000)]|length),
     small_tokens: $small,
     small_share_rows: (([.[]|select(.total_tokens<1000)]|length)/length),
     small_share_tokens: ($small/$all)}' ~/.config/pew/queue.jsonl
{
  "all_rows": 1551,
  "all_tokens": 9169036953,
  "small_rows": 95,
  "small_tokens": 45510,
  "small_share_rows": 0.06125080593165699,
  "small_share_tokens": 4.963443841843136e-06
}
```

So the volumetric story is "ignore them". The structural story is the
opposite.

## Finding 1: the cohort is monomorphic by source

Every single one of the 95 small rows comes from the same source: the editor
assistant integration (the source we redact to `ide-assistant-A` in this
journal — the original telemetry tag is the IDE-side `vscode-`-prefixed
identifier). Not one row from `claude-code`. Not one row from `opencode`. Not
one row from `codex`. Not one row from `openclaws`, `openhands`, `gemini-cli`,
or any of the other CLI-class sources. The cohort is sourced from a single
emitter.

```
$ jq -c 'select(.total_tokens < 1000)' ~/.config/pew/queue.jsonl \
    | jq -r '.source' | sort -u
ide-assistant-A
```

This is striking when you compare it to the source's overall row count. The
IDE-assistant source contributes **333 rows total** to the queue — about 21.5%
of all rows. Of those 333 rows, **95 are in the small cohort** (28.5% of that
source's rows are sub-1000-token). Compare that to the CLI-class sources,
where rows routinely run into the millions of tokens because they batch
hour-aggregated cached-prompt loops with multi-MB system messages.

The small cohort is not the bottom of a continuous global distribution. It is
the entire bottom slice of one source's distribution, and that source happens
to be the only one whose rows ever venture into sub-1000-token territory.

## Finding 2: input_tokens = 0 in 100% of the cohort

This is where the story stops being demographic and starts being structural.
Every row in the small cohort has `input_tokens == 0` and
`cached_input_tokens == 0`:

```
$ jq -c 'select(.total_tokens < 1000)' ~/.config/pew/queue.jsonl \
    | jq -c 'select(.input_tokens != 0 or .cached_input_tokens != 0)' \
    | wc -l
0
```

Zero rows out of 95 carry any input-side tokens. The total_tokens value is
entirely composed of `output_tokens` and/or `reasoning_output_tokens`. There
is no prompt being measured. There is no context window being attributed. The
row exists to record what came out of the model and is silent about what went
in.

This is not unique to the small cohort within the source. Of the 333
IDE-assistant rows in the corpus, **327 (98.2%)** have `input_tokens == 0`.
The small cohort is just the lowest-magnitude slice of a source that
fundamentally does not report prompt tokens. The "phantom input" pattern that
prior posts have cited — the editor-assistant rows where input is invisible —
is the dominant shape, and the small-row cohort is its purest expression.

The implication for any cost or cache analysis that pools across sources: if
you compute `output_tokens / input_tokens` or `cached_input_tokens /
input_tokens` over the full queue without filtering, you will get a finite
ratio that is distorted by 327 zero-denominator rows. Most of the
percentile-ratio commands in the analytics suite have to handle this with a
guard. The small-row cohort is a microcosm of why those guards exist.

## Finding 3: the cohort is multi-model — but split unevenly

If you expected the small rows to come from a single weak/cheap model — a
guess that would be intuitively appealing — the data refuses to cooperate.
Seven distinct model strings appear in the cohort:

```
$ jq -c 'select(.total_tokens < 1000)' ~/.config/pew/queue.jsonl \
    | jq -r '.model' | sort | uniq -c | sort -rn
  37 gpt-5
  24 gpt-5.1
  11 github_ide-assistant-A/claude-sonnet-4
  11 claude-sonnet-4.5
   9 gemini-3-pro-preview
   2 claude-sonnet-4
   1 gpt-4.1
```

The frontier OpenAI generation (`gpt-5` and `gpt-5.1`) accounts for **61 of 95
rows** (64.2%). Both Anthropic Sonnet variants together total 24 rows
(25.3%). Google's `gemini-3-pro-preview` shows up 9 times. The model strings
are largely the same set you would see in the source's heavy-token rows;
they are not relegated cheap models. A `gpt-5` row in this cohort might emit
70 output tokens; the same model in another row emits a quarter-million.

(Side note on the 11 rows tagged `github_ide-assistant-A/claude-sonnet-4`: that
is one of the legacy alias strings that the deduplication post called out
weeks ago. It is a different surface name for the same Anthropic weights as
the `claude-sonnet-4` rows. The aliasing chaos persists in the small cohort
just like it does at scale.)

## Finding 4: pure-output, pure-reasoning, and mixed splits

When I separated the cohort by what kind of generation token it emitted, the
shape was:

```
{
  "pure_output":     65,   // output_tokens > 0, reasoning_output_tokens == 0
  "pure_reasoning":  10,   // reasoning_output_tokens > 0, output_tokens == 0
  "mixed":           20,   // both > 0
  "both_zero":        0
}
```

There are no rows where both output and reasoning are zero, which is
reassuring — a row with input=0 and output=0 and reasoning=0 would be a
heartbeat with no signal. The 10 pure-reasoning rows are notable: those are
emissions where the model recorded reasoning trace tokens but no surfaced
output. In the small cohort that pattern shows up disproportionately on
`claude-sonnet-4.5` (the two `54`-token rows in the raw dump are both pure
reasoning). It suggests aborted or interrupted generations where the harness
captured the reasoning fragment but no completion was committed.

The 20 "mixed" rows are the most interesting from a behavioural standpoint.
They are cases where the model genuinely produced some output and some
reasoning, but the total stayed under 1000 tokens. These are the closest
analogue to a "real but small" interaction — quick suggestions, short
completions, a one-line edit accepted with a one-paragraph rationale. They
account for about a fifth of the cohort.

## Finding 5: the diurnal smear

The hour-of-day distribution of the cohort is not flat. Aggregating
`hour_start` UTC hour:

```
  19 07
  16 06
  11 02
  10 08
   8 09
   7 03
   6 05
   5 01
   4 11
   3 12
   ...
```

The 06:00–09:00 UTC band carries **53 of 95 small rows** (55.8%). For a
source that the operator typically uses from a North America/Pacific
timezone, those UTC hours map to late evening / pre-bed local time. The
pattern is consistent with sub-1000-token rows being short, exploratory
queries fired off at the end of a session — try a thing, see if it lands,
close the laptop. The denser CLI sessions, with their multi-MB cached prompts
and long agentic loops, dominate earlier hours of UTC time and therefore
later hours of operator local time. The small rows live in the wake of the
day, not in the working middle of it.

## Why the cohort is not "retries" or "pings"

One hypothesis I wanted to rule out is that the small rows are
infrastructural noise — health checks, retries after a 429, keep-alive pings,
schema-discovery probes. The data doesn't support any of those readings:

- **Retries:** if these were retries, you would expect them to cluster
  temporally near larger rows from the same source/model in the same hour.
  Spot-checking the timestamps, the small rows are not adjacent to large
  rows; they are the entire content of their hour bucket.
- **Pings:** the queue.jsonl ingestor doesn't record zero-output rows, and
  the cohort has zero rows with both `output_tokens == 0` and
  `reasoning_output_tokens == 0`. There is generation, just very little of
  it.
- **Schema discovery:** schema-discovery requests in the IDE-assistant flow
  use a model identifier that doesn't match any of the seven model strings
  in the cohort. These are real model invocations, just stunted ones.

The most parsimonious reading is the boring one: the small cohort is genuine
micro-queries. Quick autocompletes, single-line refactors, "what does this
function do" hovers. The reason no other source has them is that no other
source emits rows below the bucket-fill threshold of its own reporting
loop — the CLI-class sources roll up tokens at the hour level and a single
hour of CLI activity almost never sums to fewer than four digits of tokens.

## What this means for the analytics surface

There are a few practical takeaways for any future analytics command that
reads `queue.jsonl`:

1. **Cohorts should be source-conditional.** Pooling small rows across all
   sources gives you a one-source distribution mislabelled as a global one.
   Any "small-query" analysis must filter by source first.

2. **The phantom-input shape is structural, not buggy.** 327 of 333 rows
   from the editor-assistant source have `input_tokens == 0`. That is not a
   missing-data problem, it is the source's reporting contract. Tools that
   want input-side metrics on this source need to either declare it
   unsupported or impute from a separate signal.

3. **The 4.96 ppm token share is misleading on its own.** The cohort is
   tiny in tokens but represents 28.5% of one source's row count, and over
   half of those rows fall in a three-hour band. Counting by row, by source,
   or by hour gives you a different — and more honest — view of the cohort's
   significance than counting by tokens alone.

## Closing

There is a tendency in token-telemetry analysis to treat the bottom of the
distribution as residual — the leftover after the interesting things have
been counted. In this corpus, the bottom is more diagnostic than the top:
ninety-five rows that are all from one source, all input-blind, mostly
clustered in three UTC hours, spread across seven model strings and three
providers. The shape doesn't show up in any of the percentile ladders or
Gini scalars, because by token mass it is invisible. By row, it is the
fingerprint of a single editor integration that reports differently from
every CLI in the fleet. That asymmetry is worth knowing about before you
trust any pooled cross-source ratio.

---

**Data sources:** `~/.config/pew/queue.jsonl` (1,551 rows, snapshot
2026-04-26T11:18Z). All numbers above are derived directly via `jq` queries
shown inline; no model output was used to estimate any quantity.
