---
title: "Lag-1 autocorrelation per source: hermes at 0.402, opencode at -0.188, and what serial dependence says about workflow"
date: 2026-04-26
tags: [pew-insights, autocorrelation, time-series, telemetry, observability]
---

# Lag-1 autocorrelation per source: hermes at 0.402, opencode at -0.188, and what serial dependence says about workflow

A `pew-insights daily-token-autocorrelation-lag1` snapshot captured at `2026-04-26T04:38:21.606Z` against the local queue (`~/.config/pew/queue.jsonl`, 8,983,505,226 tokens across 6 sources) gives a per-source serial dependence picture that is more interesting than I expected:

```
pew-insights daily-token-autocorrelation-lag1
as of: 2026-04-26T04:38:21.606Z    sources: 6 (shown 6)    tokens: 8,983,505,226    min-days: 3    sort: tokens

per-source lag-1 autocorrelation (sorted by tokens desc)
source            tokens         nActive  nFilled  mean         stddev       rho1Active  rho1Filled  first       last
----------------  -------------  -------  -------  -----------  -----------  ----------  ----------  ----------  ----------
claude-code       3,442,385,788  35       72       98353879.7   209106432.8  0.258       0.332       2026-02-11  2026-04-23
opencode          2,872,651,300  7        7        410378757.1  247317923.4  -0.188      -0.188      2026-04-20  2026-04-26
openclaw          1,714,075,855  10       10       171407585.5  104563777.2  0.198       0.198       2026-04-17  2026-04-26
codex             809,624,660    8        8        101203082.5  122703257.3  -0.019      -0.019      2026-04-13  2026-04-20
hermes            142,881,896    10       10       14288189.6   10669930.8   0.402       0.402       2026-04-17  2026-04-26
editor-assistant  1,885,727      73       265      25831.9      46558.5      0.218       0.145       2025-07-30  2026-04-20
```

Six sources, six different stories about how today's token usage depends on yesterday's. The values span from `-0.188` (opencode, mildly anti-correlated) to `+0.402` (hermes, modestly persistent). The dominant source by mass — claude-code — sits in the middle at `0.258` active, `0.332` filled. This post is about why the values diverge and what each one says about the underlying usage process.

## What lag-1 autocorrelation actually measures

The metric is one number: the Pearson correlation between the daily token total at day `t` and day `t-1`, computed across all valid `(t-1, t)` pairs. Two variants are reported:

- **rho1Active**: only consecutive *active* days are paired. If the source was used Monday, skipped Tuesday, and used Wednesday, the (Mon, Wed) pair is treated as adjacent.
- **rho1Filled**: missing calendar days inside `[first, last]` are filled with zeros. The (Mon, Wed) pair becomes (Mon→Tue=0) and (Tue=0→Wed).

The two answer different questions. `rho1Active` asks "when I use this tool, does today's intensity predict tomorrow's intensity?" `rho1Filled` asks "treating idle days as zeros, does the daily usage process show momentum?" The first is conditional on activity; the second is unconditional.

For five of six sources the two values are identical, because the source was used every day inside its `[first, last]` window — `nActive == nFilled`. The two sources where they diverge are claude-code (35 active, 72 filled) and editor-assistant (73 active, 265 filled). Those are the only sources with idle days inside their lifetime windows, so they're the only ones where the choice of metric matters.

## The headline: hermes at 0.402

`hermes` shows the highest positive autocorrelation in the dataset: `0.402`. Mean daily tokens 14.3M, stddev 10.7M, 10 active days from 2026-04-17 to 2026-04-26 with no gaps. Coefficient of variation is `10.67 / 14.29 ≈ 0.75`, so the day-to-day variance is real, not just noise around a flat line.

A 0.402 lag-1 means about 16% of yesterday's variance carries forward into today (`rho^2 = 0.16`). That's modest but not trivial. In a 10-day series, the standard error on Pearson rho with no real correlation is roughly `1/sqrt(n-2) ≈ 0.354`, so `0.402` is barely outside the noise band — suggestive, not significant. With 10 days you cannot make a strong statistical claim; you can make a directional one.

The directional claim is: hermes' usage shows *persistence*. A heavy hermes day tends to be followed by another heavy hermes day. This is consistent with hermes being a project-bound tool — once a project pulls it in, it gets used over a multi-day stretch, then the project ends and usage tapers. This is the textbook signature of a workflow-bound process: AR(1)-like behavior with positive momentum.

If I had 30 days of hermes data instead of 10, a `0.402` would be at the edge of statistical significance (SE ≈ 0.19). At 100 days it would be solidly significant (SE ≈ 0.10). Worth re-running this metric monthly to see if 0.402 is the steady-state value or just a 10-day fluctuation.

## opencode at -0.188: anti-persistence or just seven days?

`opencode` shows `-0.188` — the only negative value worth discussing (codex's `-0.019` is indistinguishable from zero). It's also based on the smallest sample: 7 active days, 2026-04-20 to 2026-04-26, no gaps. With 7 data points, the SE on rho is `1/sqrt(5) ≈ 0.45`, which makes `-0.188` deeply within noise. You cannot conclude anything from this number on its own.

But there's a structural reason to expect mild anti-correlation in a high-intensity source: **mean reversion under capacity constraint**. Opencode's mean daily total is 410M tokens with a stddev of 247M — a CV of 0.60. If there's a soft daily ceiling (a budget, a context-window limit, a fatigue limit), then a heavy day can leave you "depleted" the next day, producing negative serial dependence. Conversely, if there's a soft floor (you have to do *something* every day to keep the project moving), then a light day pulls you toward the mean the next day. Either pattern produces negative rho.

The fact that opencode is the highest-mean source (410M/day) and shows negative rho while hermes is the lowest-mean source among the recent ones (14M/day) and shows positive rho is consistent with this story: **light-usage sources cluster (positive rho) because their bursts are project-bound; heavy-usage sources mean-revert (negative rho) because they're capacity-bound.** Two more weeks of data will tell whether this pattern holds.

## claude-code: the active vs filled divergence

`claude-code` is the only source with a meaningful gap between `rho1Active` (0.258) and `rho1Filled` (0.332). The filled version is *higher*, which is initially counterintuitive — you'd expect adding zero-days to dilute correlation, not amplify it.

The explanation is in the `nActive=35, nFilled=72` numbers. Half the calendar days inside claude-code's `[2026-02-11, 2026-04-23]` window are idle. When you fill those with zeros, you create long runs of zeros punctuated by bursts. Long runs of zeros are *highly autocorrelated with each other* (zero begets zero). The filled rho is being inflated by the structural autocorrelation of the idle periods, not by anything about the actual usage process.

This is a well-known pitfall of zero-filled time-series autocorrelation: the metric conflates "is the process autocorrelated when active?" with "are the idle periods clustered?" For claude-code, the active-only number (`0.258`) is the more interpretable one. The filled number (`0.332`) is contaminated by idle-day clustering.

The directional read on `0.258` over 35 active days: there is real positive serial dependence in active claude-code intensity. SE on rho with n=35 is `1/sqrt(33) ≈ 0.174`, so `0.258` is only ~1.5 SE from zero — still suggestive rather than significant, but with much more data behind it than the 7- and 10-day series. If I had to commit to one autocorrelation number for claude-code, I'd report `0.258 ± 0.17`.

## codex at -0.019: white noise

`codex` shows `-0.019` over 8 active days. This is as close to zero as any real-data correlation gets. Codex's daily process, on this slice, is indistinguishable from independent draws.

Why might codex be uncorrelated when hermes (`+0.402`) and opencode (`-0.188`) are not? One hypothesis: codex usage is *task-driven without project-stickiness*. Each codex invocation is short, atomic, and not tied to a multi-day project arc. Yesterday's codex day doesn't predict today's because codex is being used for one-off automation tasks that arrive randomly.

This matches the role codex plays for me operationally: it's a CLI agent I reach for in short bursts to run a specific automation, then close. There's no "project" that uses codex for three days in a row the way there's a project that uses claude-code for three weeks in a row. So an AR(1) model of codex daily totals should be near-white-noise, which is what `-0.019` reports.

## editor-assistant: the 265-day filled window

`editor-assistant` is the only source with a multi-month window: 73 active days inside a 265-day calendar span from 2025-07-30 to 2026-04-20. That gives `nActive=73, nFilled=265`. The filled rho (`0.145`) is *lower* than the active rho (`0.218`), reversing the claude-code pattern.

Why? Because editor-assistant's idle days are spread *thinly* across the 265-day window, not clustered into long stretches. Most days are idle (192 of 265, or 72.5%), and active days are scattered. When you fill with zeros, you don't create long runs of zeros — you create a mostly-zero series with occasional spikes. The lag-1 correlation of such a series is dominated by the zero-zero pairs (which contribute zero variance and zero covariance — they neither help nor hurt the correlation) and the active-zero pairs (which usually pull the correlation toward zero, since active days don't predict adjacent zero days well). The active-zero pairs dilute the rho, hence `0.145 < 0.218`.

This is the *opposite* contamination pattern from claude-code's. Claude-code's idle days come in *clusters*, which inflates filled rho. Editor-assistant's idle days come *interspersed*, which deflates filled rho. The metric is highly sensitive to whether idle days are clustered or scattered, in opposite directions.

For editor-assistant, the more-interpretable number is again `rho1Active = 0.218` over 73 active days. SE is `1/sqrt(71) ≈ 0.119`, so `0.218` is about 1.8 SE from zero — the strongest statistical signal in the table after hermes' 10-day result, but in the modest-correlation regime.

## openclaw at 0.198: the new normal-looking source

`openclaw` shows `0.198` over 10 active days, 2026-04-17 to 2026-04-26, no gaps. Mean 171M tokens, stddev 105M (CV 0.61). It looks structurally similar to opencode (CV 0.60, recent week-plus window) except the rho is positive. With 10 days the SE is 0.354, so neither 0.198 nor opencode's -0.188 is statistically distinguishable from zero or from each other.

But the sign difference, taken at face value, would suggest openclaw has the same project-stickiness as hermes and the opposite character from opencode. Two paired sources with opposite serial dependence — if it holds with more data — would be a real finding about how those two surfaces are used differently. Right now it's a 10-day hint.

## What the table says when you read it as a whole

Three patterns emerge if you trust the directional reads despite the small-sample noise:

**1. There's no universal autocorrelation regime.** The six values span from -0.19 to +0.40. There is no single "AR(1) coefficient for daily agent usage." Each source has its own dynamics driven by its own use case. Modeling daily agent usage as a single time series with a single rho is wrong.

**2. The sources cluster by usage character, not by token mass.** Hermes (small) and claude-code (large) both show modest positive rho. Opencode (large) and codex (small) both show near-zero or negative rho. The split correlates with whether usage is project-bound (positive) or task-bound (zero/negative), not with raw scale.

**3. Zero-filled metrics are dangerous when idle-day distribution differs by source.** The `rho1Filled` numbers diverge from `rho1Active` in opposite directions for claude-code (inflate) and editor-assistant (deflate), driven by whether idle days cluster or scatter. Reporting only one of the two would systematically mislead. Reporting both, as this CLI does, is correct.

## Sample size caveats

Five of the six sources have between 7 and 10 active days. At those sample sizes, the noise on rho is ±0.35 or worse, and almost every value in the table is within one standard error of zero. **You cannot use this snapshot to make point claims about specific autocorrelation values.** What you can do is:

- Track the values over time. Re-run weekly for 3 months, watch which sources show stable rho and which are pure noise.
- Compare *signs* across sources at one snapshot, since sign agreement across multiple weeks is a much stronger signal than any single rho value.
- Use the number to *generate hypotheses* about the underlying process (project-bound vs task-bound, capacity-constrained vs demand-driven), not to test them.

A version of this metric with bootstrap confidence intervals — resample days with replacement, recompute rho, report the 95% CI — would be a much more useful default. The current point-estimate-only output invites overinterpretation. (Filing this as a feature request to my own tool.)

## Reproduction

```bash
# in pew-insights repo
node dist/cli.js daily-token-autocorrelation-lag1
```

Snapshot captured `2026-04-26T04:38:21.606Z`. Queue file: `~/.config/pew/queue.jsonl`, 1,519 lines at capture. Total tokens across all sources: 8,983,505,226. Default `min-days: 3`. All 6 sources passed the min-days filter and are shown.

The two columns to read first are `nActive` (sample size for the active-only rho) and the difference `nFilled - nActive` (which tells you how much idle-day contamination is in the filled rho). If `nActive < 15`, treat the rho as a directional hint, not a measurement. If `nFilled - nActive` is large, prefer `rho1Active`.
