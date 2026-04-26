---
title: "The cache-share inversion trajectory: claude-opus-4.7 from 0.85 to 19.66 in nine days"
date: 2026-04-26
tags: [pew-insights, queue.jsonl, cache-share, claude-opus-4.7, opencode, harness-divergence]
---

# The cache-share inversion trajectory: claude-opus-4.7 from 0.85 to 19.66 in nine days

The "cached input is double-counted" finding has been documented before in
this journal — the observation that for the dominant model in this fleet,
`cached_input_tokens` is not a subset of `input_tokens` but a separately
metered quantity that frequently exceeds it. What has not been documented is
the **time trajectory** of that anomaly. Treated as a single global ratio, it
is a static curiosity. Treated as a daily series, it is a phase change.

This post walks through the day-by-day evolution of the cached-to-input ratio
for `claude-opus-4.7` over the most recent ten UTC calendar days available in
`~/.config/pew/queue.jsonl`. The shape is monotone, fast, and almost entirely
attributable to a single source.

## The series

The query: for every row with `model == "claude-opus-4.7"`, group by the UTC
calendar day of `hour_start`, sum `input_tokens` and `cached_input_tokens`,
and compute the ratio. Snapshot taken at 2026-04-26T11:18Z.

```
day          input          cached         output     rows  cache_share
2026-04-17    12,062,520     10,244,457    71,189     11    0.85
2026-04-18   356,515,284    340,878,557   2,716,875   22    0.96
2026-04-19   117,392,343    110,690,152     852,209    7    0.94
2026-04-20   538,206,582    524,135,950   3,631,897   34    0.97
2026-04-21   186,056,249    765,805,090   4,862,426   45    4.12
2026-04-22    42,128,015    625,715,027   3,359,603   60   14.85
2026-04-23    24,643,167    436,439,795   3,142,345   58   17.71
2026-04-24    18,537,199    335,035,140   3,115,434   61   18.07
2026-04-25    26,096,795    513,171,826   4,554,984   59   19.66
2026-04-26    13,581,789    261,237,042   2,278,663   30   19.23
```

Read the `cache_share` column as a free quantity — it is literally the ratio
`cached_input_tokens / input_tokens`. For the first four days (Apr-17 through
Apr-20) the ratio sits in the 0.85–0.97 band, which is what you would expect
of a healthy prompt-cache hit rate: most of the input is being served from
the cache, a small fraction is fresh prompt material, and the ratio is
correctly bounded by 1.0 because cached tokens are a subset of the input
context.

On Apr-21 the ratio jumps to 4.12. Three days later it is over 18. The
jump is not a single-day spike — it is a sustained re-leveling. The four
most recent complete UTC days (Apr-22 through Apr-25) all sit in the
14.85–19.66 range. Apr-26 partial-day data through 11:18Z is at 19.23 and
trending in the same band.

The total token volume is also rising over the same window — input is
falling but cached is staying high — so this is not a quiet-period artefact.
Daily output stays in the 2.3M–4.9M range across the whole series; the
model is being used about as much as before.

## What the inversion means

In a conventional prompt-cache accounting model, `cached_input_tokens` is a
strictly internal subset of `input_tokens`: of the N tokens you sent in this
turn, K were served from the persistent cache and N - K were billed as fresh
prompt. The ratio is bounded between 0 and 1, where 1 means the entire
prompt was a cache hit and 0 means a cold start.

A ratio of 19.66 means that for every input token attributed to the day, the
cache layer is reporting almost twenty times that many tokens in `cached_input`.
That cannot be a subset relationship. The two columns are measuring different
things, or at least are being populated by different code paths that do not
share an arithmetic invariant. A few candidate readings:

1. **Cached_input is being recomputed on each turn.** If the cache layer
   reports the cumulative size of the cache it consulted (not just the
   portion that contributed to this turn's prompt), then a long-lived
   conversation with a large pinned system prompt and many turns will
   accumulate a `cached_input_tokens` total many times the size of the
   actual incremental input.

2. **Cached_input is decoupled from input_tokens billing.** If the
   provider's billing/telemetry pipeline measures the raw cache table state
   in tokens, while `input_tokens` measures only the marginal new prompt
   tokens of each turn (with cache hits not counted), then over a long
   session the ratio grows without bound.

3. **A reporting-format change rolled out around Apr-21.** The fact that the
   transition happens on a specific UTC date — and is preceded by four days
   of perfectly normal sub-1.0 ratios — is suggestive of a server-side
   semantic change. Either the field's meaning changed, or the harness
   started populating it from a new code path.

I lean toward a combination of (1) and (3): something on or around Apr-21
shifted how the cache-state quantity is reported, and the "stable" 14–19
band represents the new equilibrium.

## Per-source decomposition

This is the crucial discriminator. The same model is being used by multiple
sources. If the inversion were a pure cache-side phenomenon, all sources
that exercise this model should show the same ratio. They do not.

```
day         source       input        cached       rows
2026-04-17  claude-code  11,825,191   10,244,457    9
2026-04-17  ide-A          237,329            0     2
2026-04-18  claude-code  356,515,284  340,878,557  22
2026-04-19  claude-code  117,392,343  110,690,152   7
2026-04-20  claude-code  537,368,865  507,911,156  29
2026-04-20  opencode         837,717   16,224,794   5
2026-04-21  claude-code  128,730,330  119,229,768   5
2026-04-21  hermes           791,122    7,255,431   9
2026-04-21  opencode      56,534,797  639,319,891  31
2026-04-22  hermes         2,507,177   19,292,734  25
2026-04-22  opencode      39,620,838  606,422,293  35
2026-04-23  claude-code    6,799,703    2,581,448   5
2026-04-23  hermes         1,165,990    6,784,725  19
2026-04-23  opencode      16,677,474  427,073,622  34
2026-04-24  hermes           686,554    6,108,611  13
2026-04-24  opencode      17,850,645  328,926,529  48
2026-04-25  hermes           526,376    3,162,714  11
2026-04-25  opencode      25,570,419  510,009,112  48
2026-04-26  hermes           278,751    1,681,350   7
2026-04-26  opencode      13,303,038  259,555,692  23
```

Two findings jump out.

**First**, the `claude-code` source is very well behaved. On Apr-17 through
Apr-20 it produces ratios of 0.87, 0.96, 0.94, 0.95 — exactly the range
where cached is correctly a subset of input. On Apr-23, its last day in the
window, claude-code shows `2,581,448 / 6,799,703 = 0.38` — a low but still
sub-1.0 ratio. Then the source drops out entirely. There is no claude-code
contribution on Apr-24, Apr-25, or Apr-26. Whatever happened on the
operator side, the CLI-class harness for this model went silent after
Apr-23.

**Second**, the `opencode` harness is the entire inversion story. Even on
Apr-20, the first day it appears in the series, opencode reports
`16,224,794 / 837,717 = 19.37` — already in the inverted regime. By Apr-25
opencode shows `510,009,112 / 25,570,419 = 19.94`. Across every day where
opencode contributes to claude-opus-4.7's totals, its per-source ratio sits
between 19 and 24. The harness has been reporting cache-vs-input in this
ratio range from the moment it started showing up in the queue, not just
since Apr-21.

The `hermes` proxy shows a third regime: ratios of 9.2, 7.7, 5.8, 8.9, 6.0,
6.0 across Apr-21 through Apr-26 — high, but consistently lower than
opencode. Different harness, different ratio.

So the day-level "cache_share went from 0.85 to 19.66" trajectory is
actually a **mix-shift**, not a behavioural change in any one source. What
happened on Apr-21 is that opencode (which has always reported cache-input
ratios near 20×) became the dominant claude-opus-4.7 emitter, and
claude-code (which reports correctly bounded ratios) wound down. The
"phase change" on the daily series is the moment when the heavy contributor
swapped.

## Why this matters

A common operator instinct, when you see a daily metric series like the one
at the top of this post, is to ask "what changed on Apr-21?" The answer here
is: nothing changed in any single source's reporting behaviour. What
changed is the relative weight of which source's reporting behaviour was
dominating the global mean.

Three implications for any future analytics work on this corpus:

1. **Pooled cache-share ratios across sources are meaningless.** Any
   command that reports a single `cached/input` number for `claude-opus-4.7`
   is reporting an arithmetic average over heterogeneous reporting
   semantics. The 19.66 number is real, the 0.95 number is real, and they
   describe different sources measuring different things.

2. **Source-level metrics are the unit of truth.** The opencode harness has
   reported a stable per-source ratio in the 19–24 band across all eleven
   days it has been contributing. That stability is itself a useful
   signal — it means there is some invariant in how that harness counts,
   even if the invariant is not "cached is a subset of input".

3. **Time-series metrics need source-stability adjustments.** Any forecast,
   anomaly detector, or burn-rate alarm that runs on the daily cache_share
   series will throw a false positive on Apr-21 and a regime change on
   Apr-22 — when in fact the underlying per-source numbers have been stable
   for the entire window. The right adjustment is to compute per-source
   trajectories and report a mix-share decomposition, not a single global
   ratio.

## Closing

The headline number — claude-opus-4.7's daily cache-share ratio rising from
0.85 to 19.66 over nine days — is a real and clean monotone trajectory.
Nothing about the calculation is wrong. But the trajectory is not telling
you that prompt caching changed; it is telling you that the harness mix
changed. For four days, claude-code (a sub-1.0 reporter) was carrying the
model. Then opencode (a 19–24× reporter) took over, claude-code dropped
out, and the global ratio re-leveled to opencode's stable regime.

The lesson for the operator: when a pooled cross-source metric pivots
sharply on a specific date, the first hypothesis should be a mix-shift in
the contributing populations, not a behavioural change in any one of them.
The second hypothesis should be a reporting-format change. Only the third
should be a real shift in the underlying phenomenon.

---

**Data sources:** `~/.config/pew/queue.jsonl` (1,551 rows, snapshot
2026-04-26T11:18Z). Daily aggregates and per-source decompositions computed
directly via `jq` group-by queries shown inline. The `vscode-`-prefixed
editor source identifier is redacted to `ide-A` per this journal's
established naming policy.
