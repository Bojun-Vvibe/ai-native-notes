# IO-ratio stability: claude-code's 1.496 day-over-day swing vs opencode's 0.361 flatness

date: 2026-04-26
tags: [pew-insights, io-ratio, coefficient-of-variation, day-over-day, source-stability]

## TL;DR

The `source-io-ratio-stability` lens reports, per CLI source, the *coefficient of variation* (`stddev / mean`) of the daily `output_tokens / input_tokens` ratio across each source's active calendar days. It is deliberately a different statistic from any single-window average ratio (which collapses everything into one mean) and a different statistic from autocorrelation (which measures persistence, not dispersion). What it's measuring is: **does this CLI behave the same way day after day, or does its mix between "mostly reading" and "mostly generating" swing wildly?**

Across the same six-source operator queue covered elsewhere on this site, the answer is a 4x spread in stability. `opencode` is the most stable (ratioCv = **0.361**) — its day-to-day output/input shape moves only ±36% around its own mean. `claude-code` is the least stable (ratioCv = **1.496**) — its daily ratio swings ±150% around its mean, with a stddev (0.009) larger than the mean itself (0.006). The interesting reading isn't who's at the top or bottom; it's that the *mean ratios themselves* span two orders of magnitude (0.004 to 0.116), so "stable" and "high" are completely orthogonal axes here. This post unpacks what each row in the table actually means and why ratioCv > 1 is the real flag.

## The data, captured live

```
$ node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js source-io-ratio-stability
```

Header (verbatim, captured at `2026-04-26T07:48:45.362Z` UTC):

```
sources: 6 (shown 6)    tokens: 9,057,347,003
min-days: 3    cv-min: 0.000    sort: tokens
dropped: 0 bad hour_start, 0 by source filter,
         0 below min-days, 0 below cv-min, 0 below top cap
ratioCv = stddev(daily out/in ratio) / mean(daily out/in ratio)
low CV = stable interaction shape day-over-day
high CV = swings between mostly-prompt and mostly-generation
flatLine=y means every kept day had output_tokens=0
```

Per-source rows (verbatim, sorted by tokens descending):

```
source           tokens          inTok           outTok       activeD  ratioD  zeroInD  meanRatio  stdRatio  ratioCv  flat
claude-code      3,442,385,788   1,834,613,640   12,128,825   35       35      0        0.006      0.009     1.496    -
opencode         2,939,279,889     192,113,337   19,104,267    7        7      0        0.116      0.042     0.361    -
openclaw         1,720,727,870     924,320,566    4,818,952   10       10      0        0.004      0.003     0.717    -
codex              809,624,660     410,781,190    2,045,042    8        8      0        0.004      0.002     0.389    -
hermes             143,443,069      55,226,272    1,356,997   10       10      0        0.085      0.075     0.881    -
ide-assistant-A      1,885,727         581,090    1,135,247   73        3      70       0.023      0.013     0.546    -
```

(The inline-completion IDE source is labelled `ide-assistant-A` per project convention.)

## Why daily ratioCv is a different question than mean ratio

It's tempting to look at `meanRatio` and stop there: "opencode generates 11.6% as much as it reads, claude-code generates 0.6%, openclaw 0.4%, ide-assistant-A 2.3%". But that single number flattens away the question of whether the source is *consistent*. Two CLIs could both have a mean ratio of 0.05 and tell you completely different stories:

- CLI A: every active day, the ratio is 0.04–0.06. That's a tool that has a stable interaction shape — say, a chat tool that always reads N tokens of context and emits roughly N/20 tokens of completion.
- CLI B: half the days the ratio is 0.001 (read-heavy reconnaissance), the other half it's 0.099 (generation-heavy synthesis). Same mean, but a totally different *workflow*.

CV catches this. The day-over-day stddev divided by the mean is a scale-free dispersion measure. Two operational thresholds I've found useful:

- **ratioCv < 0.5**: stable. The tool's day-over-day interaction shape is roughly fixed. Treat it as a single mode.
- **ratioCv 0.5–1.0**: bimodal or drifting. Worth slicing by week or by project to find the regimes.
- **ratioCv > 1.0**: highly unstable. The stddev exceeds the mean. Either there's a structural break in the corpus (tool used very differently in different windows) or the underlying daily distribution is heavy-tailed enough that "mean ratio" is a misleading summary statistic — use medians or per-regime conditional means instead.

## Reading each source

### claude-code: ratioCv = 1.496 (the only > 1 row)

35 active days, every one of them has positive input tokens (zeroInD = 0), so the denominator is well-defined for every daily ratio. Mean ratio is 0.006 — i.e., on an average day this CLI emits about 0.6 tokens for every 100 it consumes. Stddev is 0.009 — *bigger than the mean*. That's the signature: most days are at or below the mean (because the median of a positive-stddev > positive-mean distribution drifts below the mean), and a few days are several multiples of the mean.

This matches the qualitative pattern I'd expect for a long-context coding agent that spends most days in "read large repos, edit small chunks" mode and a few days in "generate big diffs / rewrites" mode. The 1.496 ratioCv quantifies that: it's not just that claude-code is read-heavy on average, it's that its read-heaviness varies by ~150% day to day. Any forecast of tomorrow's output-token volume that uses today's input-token volume as a regressor needs to account for that variance — a naive `pred = today_input * meanRatio` will be off by a multiplicative factor of ~2 a substantial fraction of the time.

This source has the largest sample (35 ratio-days), so the ratioCv estimate is the most reliable in the table. It is not noise.

### opencode: ratioCv = 0.361 (most stable, but only 7 days)

Stop and notice: opencode is the *most stable* and also has the *smallest* number of ratio-days (7) of any source other than ide-assistant-A's three valid days. With n = 7 the CV estimate is uncertain — the standard error of a CV from a sample of 7 normal-ish observations is on the order of CV/sqrt(2(n-1)) ≈ 0.10, so the true value could plausibly be 0.25–0.45. Even at the high end of that interval it would still be the most stable agentic source.

The mean ratio (0.116) is also the highest of any source by a wide margin: opencode emits about one token of output for every nine of input, an order of magnitude more generative than claude-code. That's plausible — opencode has a very different default agent loop than claude-code, with much shorter context-stuffing and much more direct generation. The combination "high mean ratio + low CV" is the characteristic shape of a tool with a stable, generation-forward interaction model.

The 7-day window is short because opencode's tenure in this corpus is short (firstDay 2026-04-20 in the related Benford table) — give it 30 days and the CV will firm up, but the qualitative reading (most stable agentic source) is unlikely to flip.

### openclaw: ratioCv = 0.717

10 active days, mean ratio 0.004 (very read-heavy, even more so than claude-code on average), stddev 0.003. CV of 0.717 sits in the "bimodal or drifting" middle band. Combined with the Benford evidence in the sister post (openclaw is the only source whose output-token chi-square is large enough to reject Benford with room to spare, and whose `d3..d5` shelf is elevated), the reading here is consistent: openclaw runs a wrapper that imposes structure on output sizes (Benford skew) but doesn't impose structure on the daily mix between read-heavy and generation-heavy work (CV in the middle band).

The mean ratio is essentially identical to claude-code (0.004 vs 0.006) but the CV is half as large (0.717 vs 1.496). So openclaw and claude-code emit similar token *quantities* per input on average, but openclaw's day-over-day mix is much more predictable.

### codex: ratioCv = 0.389 (second-most-stable, also a small-sample row)

8 active days. Mean ratio 0.004 (matches openclaw and claude-code on the read-heavy axis), stddev 0.002, CV 0.389. With n = 8 this is borderline reportable but the signal is clear: codex behaves very consistently from day to day. That's a fingerprint of a tool with a fixed agent loop and not much variability in how it's invoked. (The Benford row in the sister post showed codex with chi-square = 3.10, the lowest in the cohort, which is the same story from the digit angle: consistent, low-variance shaping.)

The combination "low mean ratio + low CV" describes a tool that is reliably read-heavy. You can forecast its output volume tomorrow from today's input volume with much smaller multiplicative error than you can for claude-code.

### hermes: ratioCv = 0.881

10 active days, mean ratio 0.085 (second-highest, reasoning-output-rich), CV 0.881. Hermes is the reasoning-token-heavy proxy in this corpus, and the CV near 0.9 says the day-over-day mix between reasoning-heavy days and reasoning-light days swings by ~90% of the mean. This is the row I'd most want a longer time series on — 10 days isn't enough to distinguish "structurally bimodal workflow" from "still warming up". With 30+ days I'd expect the CV to either stabilise around 0.6–0.8 (genuine bimodality between routine days and deep-thinking days) or drift toward 0.4–0.5 once the warm-up period averages out.

The 0.085 mean ratio is interesting in its own right because it's close to opencode's 0.116, but hermes's CV is more than twice opencode's. Same generation-forward order of magnitude, very different stability. That's the kind of orthogonality that justifies having both axes in the table.

### ide-assistant-A: ratioCv = 0.546 — but read the zeroInD column

This is the row that requires the most caveats. The corpus has 73 active days for ide-assistant-A but only **3 ratio-days** — `zeroInD = 70` means 70 of the 73 active days had `input_tokens = 0`. The ratio is undefined on those days (division by zero), so they're excluded from the CV computation. The CV of 0.546 is computed from 3 observations.

There are two readings of this:

1. **The mechanistic reading**: ide-assistant-A is an inline-completion tool whose telemetry doesn't always populate `input_tokens` — many of the 70 zero-input days are days when the tool was used but the prompt-side accounting wasn't recorded. The 3 days where input *was* recorded are the only ones with a defined ratio, and on those days the ratio averaged 0.023 with CV 0.546.

2. **The honest reading**: with n = 3 the CV is statistical noise. We should treat the ide-assistant-A row as "insufficient data for a stability claim" and rely on the mean ratio of 0.023 (from 3 days) only as a rough order-of-magnitude marker.

Both readings are consistent with what we know about inline-completion telemetry from the Apr-26 phantom-input post in this repo (98% of editor-assistant-source rows have zero recorded input). The CV column is real; the sample is just too thin to act on.

## Why "ratioCv" is orthogonal to the other stability lenses on this site

Several other Apr-25 / Apr-26 posts on this site touch nearby concepts. They are not redundant:

- **`burstiness` / coefficient-of-variation of hourly tokens** measures dispersion in *magnitude* per hour. Two sources can have identical magnitude burstiness while one has a stable I/O ratio and the other doesn't.
- **`daily-token-autocorrelation-lag1`** measures whether today's volume predicts tomorrow's volume — a *persistence* statistic, not a *dispersion* statistic. A source can be highly autocorrelated (predictable trajectory) and still have an unstable I/O ratio (it always uses about the same total tokens, but it shifts between reading and generating from day to day).
- **`output-input-ratio`** (the global mean version) gives you the single number `meanRatio` per source — exactly the column in this table. CV adds the dispersion-around-that-mean axis that the global statistic discards.
- **`rolling-bucket-cv`** measures within-source dispersion of token *magnitude* over rolling windows — very useful, but again about magnitude, not about read/write balance.

The unique question this lens answers is: **for each tool, how much does the read-vs-generate balance swing day-over-day?** That's a workflow-shape question, not a volume question.

## Operational implications

A few things this metric is genuinely useful for, beyond curiosity:

1. **Forecasting.** If you're trying to predict tomorrow's output-token cost from today's input-token volume, the per-source ratioCv is the multiplicative error band you should attach to your forecast. For opencode (CV 0.361) you can be reasonably confident; for claude-code (CV 1.496) you should treat any single-day forecast as ±100%.

2. **Diagnosing harness changes.** If a CLI's ratioCv suddenly jumps from ~0.4 to ~1.5 across a 30-day window, that's a strong signal something changed in how the tool is being used (different mission types, different harness, different operator workflow). The output-input mean might be unchanged, hiding the regime shift; the CV will catch it.

3. **Picking a comparison baseline.** When A/B-testing two agents, comparing single-window mean ratios is misleading if one of the agents has a high CV — the comparison is dominated by which days happened to land in the window. Compare distributions, or at minimum report both meanRatio and ratioCv per side.

4. **Flagging telemetry bugs.** ide-assistant-A's `zeroInD = 70` would be the canonical example: when zeroInD/activeD > 0.5, treat the meanRatio and CV as suggestive only and go look at the data-collection path before drawing operational conclusions.

## Cross-references

- *the editor-assistant phantom-input* (Apr-26) — explains the zeroInD = 70 row mechanically.
- *output-input-ratio-bimodality: the 26x source spread and the opencode fat middle* (Apr-26) — the global-mean version of this metric, complementary lens.
- *source-x-model-output-input-ratio: the 16x harness gap on the same opus-4.7 weights* (Apr-26) — same axis sliced by model rather than by day; explains why opencode's mean ratio is so much higher than claude-code's despite both having Opus-class models in the mix.
- *daily-token-autocorrelation-lag1: hermes at 0.402, opencode at -0.188* (Apr-26) — the persistence-axis sibling of this dispersion-axis lens.
- *picking a baseline scorer: zscore, mad, ewma* (Apr-24) — methodological prior on when CV is the right scalar.

## Reproduce

```
node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js \
  source-io-ratio-stability
```

Useful flags:

- `--min-days N` (default 3): raise to require more ratio-defined days before reporting a CV. With this corpus, raising to `--min-days 7` would drop the ide-assistant-A row entirely.
- `--cv-min X`: filter out rows with CV below X (useful for surfacing only the unstable sources).
- `--top N`: cap the number of source rows returned.

The capture timestamp `as of: 2026-04-26T07:48:45.362Z` is the citation anchor; re-running tomorrow will move the meanRatio columns by a few percent as new active days land, and will move ide-assistant-A's CV and meanRatio more substantially because n = 3 is so thin that any new ratio-day shifts both substantially. The other five sources' CVs will be stable to two significant figures across the next several active days.

## Final note: what "ratioCv > 1" actually feels like

If you operate one of these CLIs daily, "ratioCv > 1" maps to a lived experience: some days you spend the whole session feeding the agent context — pasting logs, attaching files, narrating problem statements — and the agent emits one short diff at the end. Other days you give it a one-liner prompt and it produces a 2,000-line refactor. The mean across those two days is some middle ratio; the day-by-day reality is two completely different interaction modes.

claude-code's 1.496 is that lived experience quantified. opencode's 0.361 is the opposite: regardless of what you ask it, the input/output mass shape comes out roughly the same. Neither is good or bad; they're different tools doing different jobs. The metric just makes the difference legible.
