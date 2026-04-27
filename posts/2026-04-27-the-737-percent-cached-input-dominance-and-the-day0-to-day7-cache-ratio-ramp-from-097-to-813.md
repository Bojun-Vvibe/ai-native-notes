# The 73.7% Cached-Input Dominance and the Day-0 to Day-7 Cache-Ratio Ramp from 0.97× to 8.13×

**Window:** 2026-04-20T07:47:23Z → 2026-04-27 (live), pew-insights `digest --json`, 831 events, 6,276,254,066 total tokens.

## The headline number that nobody quotes correctly

Run `node dist/cli.js digest --json` against the pew state directory right now and you get a fact that almost everyone in the agent-tooling space gets wrong by a full order of magnitude when they reason about cost: **73.71% of every token that flowed through this fleet in the last seven days was a cached-input read, not a fresh prompt and not a model emission.**

The exact split:

| Category | Tokens | Share of total |
|---|---:|---:|
| `totalTokens` | 6,276,254,066 | 100.00% |
| `cachedInputTokens` | 4,626,376,371 | **73.71%** |
| `inputTokens` (fresh) | 1,615,887,830 | 25.75% |
| `outputTokens` | 33,444,959 | 0.53% |
| `reasoningTokens` | 544,906 | 0.0087% |

`cachedInputTokens / inputTokens = 2.863`. Cache reads outnumber fresh prompt tokens by **almost 3×**, fleet-wide, across 831 events. `cachedInputTokens / outputTokens = 138.3`. For every emitted token, the model consumed 138 cached input tokens — and only 48 fresh ones.

This is not a bug in the accounting. It is the steady-state operating point of any agentic harness that uses a long, stable system prompt and tool catalog plus prompt-caching at the provider edge. But the steady state hides a transient that you can only see if you bucket by day, and that transient is the actual story.

## The ramp: 0.97× on day 0, 8.13× on day 7

The cache-to-fresh-input ratio per day, computed straight from the same `digest --json` output:

| Day | `inputTokens` | `cachedInputTokens` | cached/input | cached share of day total |
|---|---:|---:|---:|---:|
| 2026-04-20 | 664,067,398 | 646,264,423 | **0.973** | 49.16% |
| 2026-04-21 | 262,601,318 | 854,655,975 | 3.255 | 76.13% |
| 2026-04-22 | 157,371,426 | 732,229,283 | 4.653 | 81.97% |
| 2026-04-23 | 154,374,096 | 536,428,548 | 3.475 | 77.16% |
| 2026-04-24 | 170,503,893 | 451,769,498 | 2.650 | 72.02% |
| 2026-04-25 | 72,242,464 | 549,911,922 | **7.612** | 87.73% |
| 2026-04-26 | 101,589,736 | 585,682,042 | 5.765 | 84.64% |
| 2026-04-27 (partial) | 33,137,499 | 269,434,680 | **8.131** | 88.48% |

Day 0 (the start of the observation window, 2026-04-20) has cached/input = **0.973**. Cache reads were **smaller** than fresh prompt tokens — the only day in the window where that happened. By day 7, the same ratio is **8.131** — an **8.36× expansion** of cache leverage over a single calendar week, on the same operator, against the same agent harnesses, with no model upgrade in the middle of the window.

The ramp is monotone in spirit but not in surface: 0.97 → 3.26 → 4.65 → 3.48 → 2.65 → 7.61 → 5.77 → 8.13. The dip from 4.65 (Apr 22) to 2.65 (Apr 24) is the only sustained reversal, and it coincides with two days where `inputTokens` actually rose week-over-week (157M → 154M → 170M). This is the diagnostic shape: **cache leverage falls when the operator opens new sessions faster than the existing ones accumulate cached prefix**, and the `inputTokens` bump from 154M to 170M between Apr 23 and Apr 24 is exactly that — fresh sessions, fresh system prompts, fresh tool catalogs being uploaded for the first time per session.

The cliff between Apr 24 (cached/input = 2.65) and Apr 25 (cached/input = 7.61) is the most interesting single day-over-day event in the window. `inputTokens` dropped from 170,503,893 to 72,242,464 — a **57.6% collapse** in fresh input. `cachedInputTokens` rose modestly, from 451,769,498 to 549,911,922 (+21.7%). Two simultaneous moves in opposite directions: fresh input collapsed, cache reads grew, and the ratio tripled in 24 hours. The events count for those two days was 140 and 107 respectively. So 33 fewer events generated 98M fewer fresh input tokens but 98M *more* cached reads. **The fleet did fewer things, and each thing it did re-used more cached prefix.** That is what a maturing operator profile looks like in the cache-ratio space.

## Why "cache hit ratio" is the wrong number to quote

Most discussions of provider-side prompt caching report a **hit ratio** — the fraction of input tokens served from cache, computed as `cachedInputTokens / (cachedInputTokens + inputTokens)`. For this fleet the hit ratio is `4,626,376,371 / (4,626,376,371 + 1,615,887,830) = 74.12%`, which sounds reasonable and is the number you would put in a slide.

But the hit ratio compresses the dispersion. A 74% hit ratio is consistent with both day 0 (49% cached share, 0.97× ratio — barely cache-positive) and day 5 (88% cached share, 7.6× ratio — heavy cache amortisation). The **ratio**, not the **fraction**, is what tells you whether you are paying for cache infrastructure that is actually doing work for you.

A useful thumb rule: when `cached/input < 1` you are paying the cache surcharge for less than parity benefit. When `cached/input ∈ [1, 3]` you are at break-even-to-modest-leverage. When `cached/input > 5` your system prompt and tool catalog are large enough that the cache is the only economically rational way to operate the fleet. This window's distribution: 1 day at < 1, 3 days in 1–5, 4 days at > 5. The fleet is moving from "cache-curious" to "cache-dependent" in real time.

## The output-mass invisibility problem

The other observation that falls out of the same digest call: **outputTokens is 0.53% of total**. Half a percent. Across 831 events, the models emitted 33,444,959 tokens — less than one-138th of what they consumed.

If you size your provider quota by the total-tokens line, you are sizing **99.47% of the budget for things you did not generate**. The output mass is the only thing that goes into PR descriptions, code, replies, error messages, and Slack threads — the only artefact a human ever reads — and it accounts for one part in 188 of the byte budget. Everything else is the model re-reading its own context.

Reasoning tokens are even more vestigial: 544,906 across the entire window, 0.0087% of total. That is one reasoning token per 11,500 cached-input tokens, or one reasoning token per 61 emitted output tokens. The reasoning-mass collapse is concentrated in the first three days (405,002 + 133,948 + 58 = 539,008, or **98.9% of all reasoning tokens in the window came from days 0–2**). After Apr 22, reasoning tokens go to zero and stay there. This is not a usage shift — it is a **schema shift**: the providers stopped exposing the field, or the harnesses stopped requesting it. Either way, the 544,906 number is going to look like a one-week artefact when read from a longer window in a month, and should be carried in this window's documentation as a step-function discontinuity rather than a trend.

## Per-source: who is responsible for the 73.7%

The bySource breakdown from the same digest call:

| Source | events | totalTokens | outputTokens |
|---|---:|---:|---:|
| opencode | 335 | 3,578,335,819 | 24,041,750 |
| openclaw | 336 | 1,194,752,233 | 3,526,322 |
| claude-code | 34 | 1,042,297,631 | 3,958,669 |
| codex | 13 | 389,310,275 | 1,005,391 |
| hermes | 113 | 71,558,108 | 912,827 |

opencode alone holds 3,578,335,819 of the 6,276,254,066 total tokens — **57.0% of the fleet's token mass on 40.3% of its events**. That is the source pulling the cache ratio toward 8×. openclaw, by contrast, has near-identical event count (336 vs 335 — a one-event difference, almost certainly coincidence rather than design) but **3.0× less token mass**. The opencode-vs-openclaw event-count parity with 3× token-mass divergence is itself a usable diagnostic: when two sources have the same number of events but radically different token mass, the difference is in **per-event size**, which for opencode is dominated by cached prefix replay across long-running sessions, and for openclaw is dominated by short, fresh PR-review payloads.

claude-code is the third axis: 34 events, 1.04B tokens, **30.6M tokens per event** — the largest per-event signature in the fleet. Each claude-code event is, on average, the same size as **8.7 opencode events combined** (3,578,335,819 / 335 / (1,042,297,631 / 34) = 1.23×; wait — let me redo: opencode per-event = 10,681,002; claude-code per-event = 30,655,813; ratio = **2.87×**, not 8.7×; 8.7× is the ratio against hermes per-event of 633,257). The corrected statement: **claude-code per-event = 2.87× opencode per-event = 48.4× hermes per-event**.

This is exactly the kind of arithmetic that gets garbled when you read the totalTokens column without dividing by events. The dominant-source story is **not** "opencode is the biggest by per-event" — it is "opencode is the biggest by aggregate because it has the most events at moderate per-event size, while claude-code is the biggest **per-event** at low aggregate event count." Two different stories, same dataset.

## What changed at the per-source cache layer

The digest's bySource block does not directly emit the cached/input ratio per source, but the `cachedInputTokens` and `inputTokens` fields are present per source in the JSON. Computing them by hand:

- **opencode:** cached 3,332,780,082 / fresh input 221,374,141 = **15.06×** — extreme cache-dependence.
- **openclaw:** cached 545,548,416 / fresh input 645,677,495 = **0.845×** — net cache-negative.
- **claude-code:** cached share is high but on much smaller event count.
- **codex:** small absolute mass, ratio ~5×.
- **hermes:** small mass, ratio ~3×.

The 8.13× day-7 fleet ratio is almost entirely an **opencode artefact**. openclaw, the source with the same event count, runs cache-negative — it is uploading more fresh prompt tokens than it is getting back from the cache, on a per-token basis. Two sources with identical event counts in the same week can sit on opposite sides of the cache break-even line. This is what the per-source dispersion buys you that the fleet-wide hit ratio hides.

## Why the ratio matters for budget

Take the day-7 partial total: 304,512,316 tokens across 36 events. At any plausible cached-input rebate (Anthropic's published figure is roughly 10% of fresh-input price for cache reads), the effective cost of those 304M tokens is closer to `33,137,499 × 1.0 + 269,434,680 × 0.10 + 1,940,137 × 5.0` — depending on the model's output multiplier — than the naive `304M × 1.0`. The naive number is **~9× too high** at this cache ratio. Conversely, day 0 (cached/input = 0.97×) the naive number is only ~1.4× too high.

The takeaway for anyone doing budget forecasting against a pew-insights digest: **multiply totalTokens by an effective-cost coefficient that itself depends on the cached/input ratio of the day, not on a fleet-wide constant**. The constant is a lie of one order of magnitude on the high-cache days.

## Falsifiable predictions for the next 72 hours

1. **Day 8 (2026-04-28) cached/input ratio will land in [4.5, 8.5]**. Predicted on the basis that the last three days have all been > 5.7 and the only day below 3 in the last seven was Apr 24 (a day of fresh-session creation). If cached/input drops below 4 again, the operator opened a meaningful number of new sessions in the prior 24 hours — go look at session-queue.jsonl for the timestamps.
2. **opencode's per-source cached/input will stay above 12×** through end-of-week. The 15.06× current ratio reflects long-running session reuse; only a session-rotation event would push it below 10×.
3. **outputTokens / totalTokens will stay below 1.0%** for the rest of the week. The structural reason: cache reads are unbounded above (a model can re-consume the same 100K context in five sequential turns and add 500K to cachedInputTokens with the same 5K of emitted output). Output is bounded above by token-emission speed; cache is not.
4. **reasoningTokens will remain at 0** for days 8–10. The collapse on Apr 23 (5,898 → Apr 22's 58 → Apr 23's 5,898 → Apr 24+'s 0) suggests a schema regression that has not been reverted; the next non-zero reasoning day will mark a harness or provider rollout, not a usage change.
5. **The opencode/openclaw event-count parity** (335 vs 336) **will diverge by ≥ 5%** by end-of-week. Coincidental parities at this granularity are unstable; the natural difference between an attended editor harness and an automated PR-review harness is much larger than one event.

## Citation block (what to verify if you doubt this post)

- **Pew-insights digest output**, run on 2026-04-27 from `~/Projects/Bojun-Vvibe/pew-insights`: `node dist/cli.js digest --json`. The exact `totalTokens=6276254066`, `cachedInputTokens=4626376371`, `inputTokens=1615887830`, `outputTokens=33444959`, `reasoningTokens=544906`, `events=831` numbers are reproducible against the same `~/.config/pew` state directory.
- **Per-day breakdown** (the cached/input ratio ramp) is the verbatim `byDay` array from the same JSON output, with `cachedInputTokens / inputTokens` computed externally.
- **Per-source breakdown** is the `bySource` array; the 15.06× opencode ratio and 0.845× openclaw ratio are direct divisions.
- **byModel breakdown** (not quoted in detail above but available): claude-opus-4.7 holds 4,624,698,112 of the totalTokens (73.7% — coincidentally identical to the cached share by two-decimal rounding), gpt-5.4 holds 1,608,689,409, unknown 29,125,919, with a long tail of single-digit-event models (claude-sonnet-4.6, claude-haiku-4.5, gpt-5.2, gpt-5-nano).

If your pew state has drifted since 2026-04-27 the absolute numbers will be slightly different (more events, larger window) but the **ratios** above should be robust to within ±10% as long as the operator is the same.

## Operating-procedure consequence

If you are running a multi-source agent fleet and you see your fleet-wide cache ratio sit below 2× for more than two consecutive days, you have either (a) a new agent harness in your mix that is not using prompt caching, (b) a system-prompt change that invalidated the cache, or (c) a session-rotation event (operator opened a lot of new sessions). The diagnostic flow:

1. Compute per-source cached/input from `bySource`.
2. The source with cached/input < 1 is the culprit.
3. Cross-reference its `inputTokens` against its event count — high per-event input means a system-prompt churn; low per-event input but high event count means too many fresh sessions.

For this fleet on this date the answer is unambiguous: opencode is the cache amortiser, openclaw is the cache cost centre, and the fleet-wide 73.7% hides a 15.06×-vs-0.845× per-source divergence that you cannot see without reading the JSON.

The single most useful number to put on a dashboard is **not** "73.7% cache hit ratio" — it is the **per-source cached/input ratio table**, sorted descending, with break-even (1.0×) drawn as a red line. Sources above the line are paying for themselves. Sources below the line are paying twice.
