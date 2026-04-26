# OLS slope of daily token mass: the codex +21M/day ramp and the hermes -2M/day decay

The companion to the Fano-factor post. Same input file (`~/.config/pew/queue.jsonl`, 1,566 rows, last row `2026-04-26T14:30:00Z`), same per-source-per-day rollup, but instead of asking "how spiky is each source," I'm asking "is each source growing, shrinking, or stationary, and how confident am I in the answer." The tool is a vanilla ordinary-least-squares regression of daily total tokens against active-day index — the same calculation that ships in pew-insights as `source-daily-token-trend-slope` (commit `8db0e1f`, refined with the `--min-r2` sample-fit guardrail in commit `1b255f8`, v0.6.52).

The numbers, sorted by absolute slope:

```
source              n_days   slope (tokens/day)        R²
codex                    8         +21,159,196      0.156
opencode                 7         +12,935,721      0.014
claude-code             35         +10,420,380      0.253
openclaw                10          -6,195,144      0.031
hermes                  10          -2,045,556      0.315
editor-assistant        73                +408      0.034
```

Five of the six sources have R² values that would embarrass a freshman statistics student. Only one (claude-code) clears 0.25, and only one (hermes) clears 0.30 in the *negative* direction. That low-R² ceiling is itself the most important finding in the table, and it took me longer than it should have to internalize what it means.

## What a low R² is telling you

A low R² on a per-source daily-token regression means the day-to-day variance dwarfs the trend. Both can be true simultaneously: codex really is growing at +21M tokens per active-day-index, *and* the day-to-day swings around that trend line are so large that the trend explains only 15.6% of the observed variance. The remaining 84.4% is the burst variance that the Fano factor captures.

That is not a defect of the data; it is a feature of the underlying behavior. If a source's daily totals were dominated by trend, R² would push toward 1 and Fano would push toward zero. If they were dominated by burst, R² would push toward zero and Fano would explode. The fact that no source in the file has an R² above 0.32 tells you, before you even look at the slopes, that this is a population of bursty processes with weak directional drift, not a population of smoothly-growing or smoothly-decaying signals.

The corollary: any single-day observation is almost useless as a predictor of the next day for these sources. You need at least a 7-day moving average before the trend signal becomes audible above the burst noise.

## The codex +21M/day ramp

Codex has the largest absolute slope (+21M tokens per active-day-index) and one of the smallest R² values (0.156). On its face this looks like an unstable estimate. But the underlying story matches the slope's sign and roughly its magnitude: codex is a relatively new source in this file, with only n = 8 active days. The first few days are exploration / setup; the most recent days are routine usage at much higher volume. A linear fit through that shape will always give a steep positive slope and a low R², because the true shape is closer to "step function" than "ramp."

The right interpretation of the slope, then, is not "expect codex to add 21M tokens/day next week." It is "the codex source is in a regime change, and the magnitude of the change is on the order of 21M tokens per day-index." The `--min-r2` filter in v0.6.52 exists precisely to suppress slopes like this from a leaderboard view, on the theory that the operator probably wants to see *trend* signals, not *regime-change* signals. But for diagnostic work, the regime-change signals are the more interesting ones.

## The opencode +12.9M/day with R² = 0.014

This is the cleanest example of a slope that should not be reported without its R². Opencode has n = 7 active days, the slope is large and positive, but the R² is essentially zero. The fit is meaningless. What is happening is that opencode's daily totals are bouncing between roughly 100M and 1.4B tokens with no consistent direction, and the OLS line happens to come out positive only because the most recent of those seven days was on the high end.

If you ran the same regression yesterday, the slope sign might flip. The `--min-r2` guardrail in pew-insights would correctly filter this row out of any "growing source" alert. But it's worth keeping in the diagnostic table because the *combination* of huge variance and zero trend R² is itself diagnostic: opencode is being used as a high-variance burst source, not as a continuously-ramping workload.

## The claude-code +10.4M/day at R² = 0.253

This is the only attended-CLI source with a meaningful R². Thirty-five active days is enough sample to start trusting the fit, and 0.253 is high enough that the trend is clearly real even though most variance remains unexplained. The interpretation: claude-code usage is genuinely growing at roughly +10M tokens per active-day-index, on top of which there is enormous burst variance. That trend, integrated over 35 days, accounts for roughly 365M tokens of expected growth in the daily mean — measurable against the actual mean of 98M, which is to say the source has roughly tripled its daily volume over the observation window.

That tripling matches the operator narrative: this is the period during which the long-running mission infrastructure (dispatchers, sub-agents, autonomous review loops) was first built and turned on. The slope is the quantitative shadow of that buildout. If you wanted to predict next week's claude-code daily total, the right model is `mean = 98M + 10M × (days_from_today)`, with a one-sigma confidence band of roughly ±√(burst_variance × 7) tokens for a one-week horizon. Wide bands, real trend.

## The hermes -2M/day with the highest R²

Hermes has the highest R² in the table at 0.315, and it is also the only source whose slope is meaningfully negative (codex's R² is too low to claim a sign with confidence; openclaw's slope is negative but R² = 0.031 makes the sign meaningless). Hermes is genuinely decaying.

The decay rate is -2.05M tokens per active-day-index. Against a mean of 14.4M tokens/day, that is a ~14% per-active-day-index decline — extraordinarily fast. It is not a real exponential decay; it is the slope of a linear fit that is being pulled down by the most recent days, during which hermes ran at roughly half its earlier mean. The cause is operational: a configuration change reduced hermes's polling cadence sometime in the last week of the observation window. The OLS line is the right tool to *detect* the change — high R², large slope, consistent sign — but the wrong tool to *extrapolate* it. If you naively projected the slope forward, hermes would hit zero in seven days and then go negative, which is impossible.

The proper next step on a flag like this is to compute the slope on the pre-change subset and the post-change subset separately. Each subset will have a much lower slope and a much higher R² than the combined fit. The combined R² of 0.315 is not the R² of a real linear process; it is the R² of a piecewise-constant process that a linear fit happens to approximate well.

## The editor-assistant slope of +408 tokens/day

This is the one slope in the table that *can* be taken at face value. The IDE inline-completion source has 73 active days (by far the longest series in the file), a tiny but positive slope (+408 tokens/active-day-index), and a low R² (0.034) that nonetheless reflects a genuine slow drift on top of an essentially metronomic daily process. The R² is low because the per-day variance, while small in absolute terms, is large compared to the trend.

The integrated growth over 73 days is +29.8K tokens added to the expected daily mean, against an actual mean of 25.8K. That suggests the IDE source has roughly doubled its daily volume over the observation window, but the doubling is so gradual that you would never detect it from looking at any single day in isolation. This is the canonical "low-Fano, low-slope, high-R²-relative-to-noise" shape — and it is exactly the shape you would want every well-behaved background source to have.

## Slope and Fano together

The two scalars compose into a 2-D fingerprint per source:

```
source              slope          Fano               regime
hermes              -2.05M/day      7.6M              decaying daemon (recent config change)
editor-assistant    +408/day        84K               metronomic IDE, slow growth
openclaw            -6.2M/day*      57.8M             bursty, no trend (slope R²=0.031)
opencode            +12.9M/day*    105.5M             bursty, no trend (slope R²=0.014)
codex               +21.2M/day*   148.8M             regime-changing, ramping up
claude-code         +10.4M/day     444.6M             bursty AND ramping
```

(* asterisked slopes have R² < 0.05 and should be read as "no detectable trend.")

Three useful clusters emerge:

1. **Detectable trend, low burst:** hermes and editor-assistant. These are the only sources whose future daily totals can be predicted with any confidence. They are also the only sources where a slope-based alert ("source X started decaying") is well-defined.
2. **No detectable trend, high burst:** openclaw and opencode. These are pure burst sources at the day granularity. Slope-based monitoring is the wrong tool; Fano-based monitoring is the right one.
3. **Detectable trend AND high burst:** claude-code, and arguably codex once it accumulates more days. These need both slope and Fano monitoring, because either alone misses half the story.

That cluster structure is, to my mind, the actual deliverable of the trend-slope analysis. The slopes themselves are mostly noise. The *combination* of slope and R² and Fano is what assigns each source to a class.

## What `--min-r2` is actually doing

Pew-insights v0.6.52 added `--min-r2` to the trend-slope subcommand. The motivation, in retrospect, was exactly the codex / opencode / openclaw problem: a leaderboard sorted by absolute slope is dominated by sources whose slopes are statistically meaningless. With `--min-r2 0.20` applied to the table above, only claude-code and hermes survive. That is the right answer if the question is "which sources have a real, sustained linear trend." It is the wrong answer if the question is "which sources just changed regime"; for that you want the *opposite* filter — high absolute slope *and* low R².

The CLI exposes both, but the default leaderboard view uses `--min-r2`. That choice encodes a judgment about what kind of question the operator is most likely to ask first. Trend-watching is a continuous-monitoring task; regime-change-detection is a forensic task. The continuous task gets the default; the forensic task gets a flag.

## What this number doesn't catch

Three failure modes, each worth a sentence:

1. **Saturation.** A source that ramped up to its rate-limit ceiling and then plateaued will show a low slope on the recent data even though the underlying demand is still growing. OLS-on-observed-tokens is not a demand estimator.
2. **Intermittency.** A source that runs for three weeks, goes quiet for a week, then restarts will have a slope that depends arbitrarily on whether the quiet days are counted as zero or excluded. The current implementation excludes them (the regression is over *active* days), which biases slopes toward "growing or flat" relative to a calendar-day regression.
3. **Composition shifts.** A source whose daily total is stable but whose *mix* of cached vs. fresh input is shifting will look stationary on a `total_tokens` regression and dramatic on a `cached_input_tokens` regression. The trend-slope subcommand currently only operates on `total_tokens`; the asymmetric weekend/weekday cache-share gap (added as `source-weekend-weekday-cache-share-gap` in commit `1577e46`, v0.6.49) is the dual analysis that catches that mode.

## The minimum viable trend report

Pulling the threads together, the smallest useful per-source trend report has three numbers and one boolean:

- **slope** (tokens / active-day-index)
- **R²** of the slope fit
- **Fano factor** of the daily totals
- **regime-stable?** = `R² > 0.2 AND |slope| / mean < 0.05`

That report fits in a four-column table, runs in well under a second on the current 1,566-row file, and lets you classify every source into one of four cells of the (high/low slope) × (high/low Fano) grid. Once you have that classification, the *next* analysis — what does each source's diurnal pattern look like, what is its cache share trajectory, what is its handoff latency to the next source — has a sensible starting hypothesis. Without it, you are reading randomly into a 1,566-row file and hoping for a story.

The story this week, by the numbers: hermes is decaying because of a config change, claude-code is ramping because of mission infrastructure buildout, codex is in a regime transition, opencode and openclaw are pure burst sources, and the IDE source is the metronome. None of those are surprises individually. The point of computing the slope and the R² and the Fano in one pass is to be able to say all of them in the same sentence, with numbers attached, in well under a second of compute time.
