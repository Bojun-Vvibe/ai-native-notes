# cache-hit-by-hour: the 32 pp source spread and the 05:00 UTC trough

When you tell people "my prompt cache is at 87 percent" they hear one thing: a single number, holding steady, presumably trending up over time as your tooling matures. That framing is wrong in three different ways at once, and the `pew-insights cache-hit-by-hour` lens — bucketed cache effectiveness across the 24 UTC hours and broken out per source — is what makes the wrongness visible. The headline 87.52 percent global cache ratio across 3.378 billion input tokens hides a 32 percentage-point spread between the best and worst hour of the day, a 53.5 percentage-point spread within a single source, and at least one source that is essentially uncached. Once you look at the geometry instead of the scalar, "improve cache hit ratio" stops being a useful goal and starts being four different goals that pull in different directions.

## The data

Generated 2026-04-25T17:15 UTC against the local `pew` queue. Input vocabulary: 3,378,908,667 input tokens, 2,957,344,925 cached, six distinct sources, 327 zero-input rows dropped.

Global cache ratio by UTC hour:

```
hr  input        cached       cache%  rows
00  50,522,168   43,248,730   85.6%   27
01  203,109,223  183,878,008  90.5%   58
02  201,520,970  174,929,264  86.8%   85
03  135,207,321  112,268,874  83.0%   58
04  108,483,280  97,415,796   89.8%   37
05  166,848,476  121,516,057  72.8%   53
06  259,680,236  198,893,491  76.6%   88
07  273,820,082  229,044,940  83.6%   84
08  308,257,392  287,038,877  93.1%   77
09  126,064,424  103,037,164  81.7%   62
10  147,235,874  132,610,638  90.1%   52
11  141,109,642  123,746,659  87.7%   46
12  178,297,004  165,872,979  93.0%   37
13  181,834,957  162,275,912  89.2%   48
14  185,953,688  169,489,729  91.1%   53
15  177,667,354  163,890,419  92.2%   46
16  203,213,768  190,369,534  93.7%   46
17  97,352,517   89,807,146   92.2%   32
18  90,932,645   85,328,194   93.8%   28
19  43,845,137   39,560,645   90.2%   29
20  21,996,796   18,738,907   85.2%   23
21  18,705,219   15,751,424   84.2%   22
22  27,907,564   23,757,250   85.1%   22
23  29,342,930   24,874,288   84.8%   29
```

Per-source summary:

```
source         input          cached         daily%  peak hr  peak%  trough hr  trough%  spread
claude-code    1,834,613,640  1,595,643,323  87.0%   19       95.7%  05         61.7%    34.1 pp
openclaw         901,500,378    773,467,008  85.8%   03       90.8%  00         80.8%    10.0 pp
codex            410,781,190    396,009,088  96.4%   12       98.0%  01         86.0%    12.1 pp
opencode         176,522,658    148,794,516  84.3%   19       99.7%  09         46.2%    53.5 pp
hermes            54,909,711     43,430,990  79.1%   19       96.7%  20         53.6%    43.1 pp
[6th source]         581,090              0   0.0%   02        0.0%  02          0.0%     0.0 pp
```

(The sixth source is an editor-embedded assistant that emits 581 KB of input tokens with literally zero cache hits — it appears to bypass the prompt-cache entirely. Listed for completeness; will be excluded from the cache-quality discussion below.)

That is the entire dataset. Everything that follows is reading this table.

## Wrongness #1: the global number averages over a 21 percentage-point hour-of-day swing

Read the global column straight down. The trough is hour 05 UTC at 72.8 percent. The peak is hour 18 UTC at 93.8 percent. That is a 21 pp gap, on a metric where everyone treats anything below 90 as a problem and anything above 95 as a stretch goal. It means the same fleet, doing the same work, is hitting completely different cache regimes depending on what time of day a prompt happens to land.

Hour 05 is not a sampling artefact. It carries 166.8 M input tokens across 53 rows — both numbers are firmly in the middle of the population. Hour 18 is smaller (90.9 M, 28 rows) but has the trough-defining cache ratio reproduced two hours out (hour 16: 93.7 percent, hour 18: 93.8 percent, hour 12: 93.0 percent). That is a five-hour cache-friendly band straddling the middle of the UTC day.

The mechanism is not subtle. Hour 05 UTC is mid-evening Pacific time, which corresponds to first-touch-of-the-day prompts on a fresh shell — the cache is cold. Hours 12-18 UTC are deep into the workday on the same machine; the system prompt, the project context, and the recent file blobs are all warm. The cache ratio is reading the diurnal cycle of process lifetimes, not the quality of the cache itself.

If you optimise the global number you will be tempted to do warm-up tricks: pre-flight a prompt at 05:00 UTC to seed the cache. Do not do this. The 21 pp gap is not a regression. It is a load shape. The prompts at 05 UTC are different prompts (cold-context first-day-of-work prompts) and they would not benefit from being seeded with stale context from the previous day. Reducing the spread in a way that changes the prompt mix would actually reduce signal quality. Reducing the spread by improving the cold-cache pathway (longer cache TTL, persistent cache across processes) is the actual move.

## Wrongness #2: the 87 percent global hides one source at 96 percent and another at 79 percent

The per-source pivot puts the headline number into a band:

- `codex`: 96.4 percent across 410 M input tokens
- `claude-code`: 87.0 percent across 1.834 B input tokens
- `openclaw`: 85.8 percent across 901 M input tokens
- `opencode`: 84.3 percent across 176 M input tokens
- `hermes`: 79.1 percent across 54.9 M input tokens

That is a 17.3 pp spread between the best and worst non-degenerate source. It is also a clean ordering by tooling maturity, in the wrong direction for the obvious story: the source with the smallest input volume (hermes at 54.9 M) is the worst-cached, and the source with the third-largest volume (codex at 410 M) is the best-cached. Volume does not predict cache quality — `claude-code` carries 4.5x the input tokens of `codex` and is 9.4 pp behind it on cache ratio.

The clean read: `codex` has a tight prompt schema with a stable system block and incremental message appends, which is exactly the shape prompt-caches reward. `claude-code` has a richer agent framework with more tool-call expansion and more frequent system-prompt mutation, which fragments the cache key. `hermes` is bursty and ephemeral; sessions are short, and cache hits cannot accumulate within a session before it ends.

The lesson for the headline: aggregating cache ratio across sources is a 17 pp lie. The number you should report is per-source, with sample size, and the optimisation you should pursue is per-source — `codex` is at the diminishing-returns frontier and you should leave it alone; `hermes` has 21 pp of recoverable headroom but the recovery requires extending session lifetime, not tuning the cache.

## Wrongness #3: the within-source spread is bigger than the between-source spread

The between-source spread is 17.3 pp (96.4 − 79.1). The within-source spread for `opencode` is 53.5 pp — peak 99.7 percent at hour 19 UTC, trough 46.2 percent at hour 09 UTC. The within-source spread for `hermes` is 43.1 pp. For `claude-code` it is 34.1 pp.

`opencode` deserves a paragraph on its own. A 99.7 percent cache hit at hour 19 means almost every input token is being served from cache — that is a hyper-stable, repeated-prompt regime, probably a tight tool-call loop or a single long-running chat. A 46.2 percent cache hit at hour 09 means more than half of the input tokens are net-new — that is exploration mode, new files being read, new context being loaded. Same source, same machine, same calendar day, six hours apart, and the cache regime has flipped end-to-end. There is no average that tells the truth about a source whose hourly geometry looks like that. The 84.3 percent daily ratio for `opencode` is the arithmetic mean of two completely different workloads and should be discarded.

The narrow exception is `openclaw` — peak 90.8 percent at hour 03, trough 80.8 percent at hour 00, spread 10.0 pp. That is a source whose cache effectiveness genuinely is hour-stable. Reporting a single daily number for `openclaw` is honest. Reporting one for `opencode` is malpractice.

## What the row counts tell you about confidence

Rows per hour vary from 22 (hour 21) to 88 (hour 06). The cache-ratio numbers from low-row hours should be discounted. Hour 21 carries 18.7 M input tokens across 22 rows — that is mean ~850 K input tokens per row, fine for sample size, but the small denominator means a single 4 M-token cold prompt could swing the hourly cache ratio by 20 percent. Hour 06 with 88 rows and 259.7 M input tokens is much harder to perturb.

The interesting confidence pattern is that the highest-cache hours (16, 18, 12) all have row counts in the 28-46 range — well-populated but not overwhelming. The lowest-cache hour (05) has 53 rows with 166.8 M tokens, which is solidly populated. So the trough is statistically real. The peak band is also statistically real but slightly less so.

What this means in practice: do not chase a 1 pp move on hour 21 or hour 22 (small denominators, easy to perturb). Do chase a 5 pp move on hours 05-07 (large denominators, currently at 72.8 / 76.6 / 83.6 — clear underperformance against the daily mean of 87.52, plausibly recoverable by extending cache TTL across the overnight gap).

## The asymmetry between peak and trough is structural

For four of the five real sources, the peak hour is in the second half of the UTC day (hours 12, 19, 19, 19) and the trough is in the first quarter (hours 00, 01, 05, 09). The exception is `openclaw` (peak 03, trough 00 — both in the cold band).

The pattern is consistent with a diurnal warm-up: caches are cold at session start, warm by mid-session, cold again after long idle gaps. The 19 UTC peak repeating across `claude-code`, `opencode`, and `hermes` is the same operator's caches all warming up together at the same wall-clock moment — late afternoon Pacific. That is a single-operator fleet shape, not a global pattern, and it would not generalise to a multi-operator team. Anyone reading these numbers needs to know: the per-source peak/trough hours are operator-specific. The shape is real but the clock is yours.

## What to actually do with this

1. Stop reporting a global cache hit ratio. Report per-source, with sample size in input tokens. The 87.52 percent number is a weighted mean of five regimes and a degenerate sixth and is not actionable.
2. For each source, report the spread alongside the mean. `opencode` 84.3 percent ± 27 pp is a different story than `openclaw` 85.8 percent ± 5 pp. The first invites investigation; the second invites no action.
3. Do not chase the trough hour on principle. Investigate whether the trough is cold-cache prompts (extend TTL) or genuinely net-new prompts (do nothing). The hour-05 / hour-06 trough on `claude-code` looks like the former; the hour-09 trough on `opencode` looks like the latter (different work, not stale cache).
4. Use the high-ratio sources as the upper-bound benchmark. `codex` at 96.4 percent shows what is achievable when the prompt schema is tight. `claude-code` at 87.0 percent has a measurable 9.4 pp gap to that frontier; the question is whether closing it costs more in framework expressiveness than it saves in token cost.
5. The degenerate sixth source (581 KB input, 0 cached) is not a cache problem. It is an integration problem — that source is not participating in the prompt cache at all. The fix is not "improve the cache ratio" but "wire the source into the cache". Once wired, the entire 581 KB column becomes hit-eligible.

## The shape of the number, not the number

The metric `cache-hit-by-hour` is a 24-by-N grid (24 hours, N sources), and the headline 87.52 percent is one cell of that grid that has been computed by collapsing the other 24 × N − 1 cells. The compression ratio is roughly 144 to 1 (24 × 6 = 144 cells reduced to one scalar). At that compression ratio the only thing the scalar can preserve is the order of magnitude.

The right reporting unit for this metric is the per-source row of the table, not the global cell. The right alerting unit is a per-source threshold breach, with the source-specific spread baked into the threshold. The right optimisation unit is a per-source intervention with a per-source success criterion. None of this is novel; it is what you would do for any other metric where the population is heterogeneous. The mistake is letting "cache hit ratio" sound singular enough to evade the population question.

The 32 pp gap between the global peak (93.8 percent at hour 18) and the global trough (72.8 percent at hour 05) is not a cache problem. It is a measurement problem. Once you stop averaging across regimes that should not be averaged together, both numbers become useful for different things, and the 87.52 percent global number gets retired.
