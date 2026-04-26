---
title: "Rolling-bucket CV: globalCv 2.60 vs window-meanCv 0.88, and the variance-time-scale gap"
date: 2026-04-26
tags: [pew-insights, coefficient-of-variation, rolling-window, variance, telemetry, observability]
---

# Rolling-bucket CV: globalCv 2.60 vs window-meanCv 0.88, and the variance-time-scale gap

A `pew-insights rolling-bucket-cv` snapshot captured at `2026-04-26T04:41:54.661Z` against the local queue (`~/.config/pew/queue.jsonl`, 1,344 windows, 8,986,315,443 tokens, 6 sources, window-size 12 active buckets) gives a per-source view of token-per-bucket variability that is easier to misread than to read:

```
pew-insights rolling-bucket-cv
as of: 2026-04-26T04:41:54.661Z    windows: 1,344    tokens: 8,986,315,443    window-size: 12 active buckets

per-source rolling-window CV distribution (sorted by tokens desc)
source            buckets  windows  globalCv  minCv  p50Cv  p90Cv  maxCv  meanCv  peak window start
----------------  -------  -------  --------  -----  -----  -----  -----  ------  ------------------------
claude-code       267      256      1.422     0.309  0.954  1.326  1.752  0.937   2026-04-08T09:00:00.000Z
opencode          217      206      1.132     0.190  0.516  1.127  2.013  0.598   2026-04-23T14:30:00.000Z
openclaw          387      376      1.161     0.185  0.484  1.108  1.660  0.625   2026-04-18T16:00:00.000Z
codex             64       53       1.127     0.522  0.963  1.295  1.675  0.981   2026-04-14T07:30:00.000Z
hermes            155      144      1.098     0.409  0.899  1.689  2.150  1.017   2026-04-22T16:00:00.000Z
editor-assistant  320      309      2.600     0.477  0.863  1.125  2.380  0.883   2026-02-05T06:00:00.000Z
```

The key structural observation: every single source has `globalCv > meanCv`. For some sources the gap is small (claude-code: 1.422 vs 0.937, ratio 1.52). For one source it's huge (the editor-assistant: 2.600 vs 0.883, ratio 2.94). This isn't a bug in the metric. It's the metric correctly telling you something specific about the time-scale of variance — and people consistently misread it.

This post is about what the gap between `globalCv` and the rolling-window CV distribution actually means, why the editor-assistant's gap is the largest, and how to use the two numbers together rather than treating them as competing measurements of the same thing.

## What rolling-bucket CV measures

Pick a source. Bucket its activity into 30-minute active buckets (idle buckets are skipped). Take a sliding window of 12 consecutive active buckets — six hours of "active time," which is *not* six hours of wall-clock time because the active buckets aren't necessarily contiguous. Compute the population coefficient of variation `stddev / mean` of the token-per-bucket values inside that window. Slide the window forward by one active bucket, recompute. Keep going until you've covered every valid window.

Each window gives you one CV number. Across all windows for a given source you get a distribution: `minCv`, `p50Cv`, `p90Cv`, `maxCv`, `meanCv`. That's the right-hand side of the table.

Separately, `globalCv` is computed once: `stddev / mean` over all token-per-bucket values for that source across the entire history. It does not slide. It does not window. It's the CV of the whole series.

These are answering two different questions:

- **globalCv** asks: "How variable is this source's bucket-token distribution over its full lifetime?"
- **window-CV distribution** asks: "When you look at any local 6-hour-active stretch, how variable is it inside that stretch?"

The first includes long-run drift, regime changes, and the gap between busy weeks and quiet weeks. The second strips all of that out — it only measures within-window variance.

## Why globalCv is always larger

The decomposition is mathematically forced. Total variance equals within-window variance plus between-window variance (law of total variance, with windows as the conditioning variable):

```
Var(global) = E[Var(window)] + Var(E[window])
```

Translated: the global variance is the average of the within-window variances *plus* the variance of the window means. The within-window CVs only see the first term. The global CV sees both.

So `globalCv ≥ meanCv` is a structural lower bound (modulo some technicality from CV being a ratio rather than a raw variance, but it holds in practice for any non-pathological series). The *gap* between them is informative: it tells you how much of your source's variability comes from "the bursts vary in size over time" rather than from "each individual burst is internally variable."

## The editor-assistant: globalCv 2.60, meanCv 0.88

The editor-assistant has the largest gap in the table by a wide margin. `globalCv = 2.60`, `meanCv = 0.883`, ratio 2.94. The peak window started `2026-02-05T06:00:00.000Z`.

What does this mean? The editor-assistant's bucket-token stream, viewed globally, is wildly variable — `globalCv = 2.60` is enormous, indicating that some buckets have token counts dozens of times larger than the mean. But viewed inside any rolling 12-bucket window, it's only modestly variable — `meanCv = 0.88` is comparable to the other sources, and the `p90Cv = 1.125` is the *lowest* p90 in the table. So the inside-window experience is actually one of the *least* bursty in the dataset.

The reading: the editor-assistant has *long-run regime variance* but *short-run consistency*. Within any 6-active-hours stretch, it produces a fairly uniform stream of small token counts. Across months, those streams vary by huge amounts — quiet stretches average a few hundred tokens per bucket, peak stretches average tens of thousands. The metric is correctly identifying this.

This matches the operational picture. The editor-assistant runs continuously in the background of the editor; its per-bucket token output is dominated by inline completion suggestions, which are individually small and locally consistent. But how many completions get triggered varies enormously by what I'm doing in the editor that week — a heavy refactoring week looks completely different from a documentation week. Hence: low within-window CV, very high global CV.

The 73-active-day, 265-calendar-day window for this source (from the `daily-token-autocorrelation-lag1` companion metric) reinforces this: the source has the longest history in the dataset, so it has the most opportunity to accumulate regime variance.

## Claude-code: globalCv 1.42, meanCv 0.94, ratio 1.52

Claude-code has the *smallest* ratio between global and mean CV in the table. That means most of its variability is already captured inside any 12-bucket window. The local experience is bursty (meanCv 0.94, p90 1.33, max 1.75 — all high) and the global experience is only moderately more bursty.

Translated: claude-code is high-variance at every time scale. There's no "calm baseline" stretches and "busy regime" stretches that you'd see in the editor-assistant. Every 6-active-hour window of claude-code looks roughly as bursty as every other one, and the lifetime distribution doesn't add much on top.

This is consistent with claude-code's role as the primary interactive coding tool. Inside any session, prompt sizes vary enormously (a one-line question vs a 100K-context refactor request) but the *distribution* of prompt sizes is roughly stationary — it doesn't shift across weeks the way the editor-assistant's rate does.

The peak window (`2026-04-08T09:00:00.000Z`) at maxCv 1.75 is moderate — it's not a wild outlier from the typical p90 of 1.33. Compare to opencode whose maxCv 2.013 is much further above its p90 of 1.127, or hermes whose maxCv 2.150 vs p90 1.689 also shows a more extreme tail. Claude-code's worst window isn't that much worse than its 90th percentile window. The shape is high-mean-low-spread.

## Opencode and openclaw: paired sources, paired distributions

Opencode (`globalCv 1.132, meanCv 0.598`) and openclaw (`globalCv 1.161, meanCv 0.625`) have nearly identical CV statistics across every column. They're indistinguishable in this metric.

Both have low meanCv (~0.6), the lowest in the table among interactive sources. Both have moderate globalCv (~1.15). Both have minCv around 0.19, also the lowest in the table. The local experience of these two sources is *flat* — long stretches where bucket-to-bucket variance is small.

This pairing is consistent with what other metrics have shown about these two sources: they appear to be linked surfaces. Whether that's two clients to the same agent, or two operating modes that share a backend, the bucket-token signature is the same shape.

The interesting question is *why* they're so locally flat. One hypothesis: when these tools are active, they're being driven by an automated loop with a relatively uniform per-step token budget. A loop that does "fetch → analyze → respond → repeat" with similar context sizes on each iteration would produce exactly this shape — low within-window CV, moderate between-window CV (because different loops at different times use different context sizes). The fact that both sources have minCv 0.19 — meaning some windows have CV well below 0.5, which is structurally low for any real-world variable — is the strongest signal that these are partly machine-driven workloads.

## Codex and hermes: high meanCv, the bursty small sources

Codex (`globalCv 1.127, meanCv 0.981`) and hermes (`globalCv 1.098, meanCv 1.017`) have the highest within-window CVs in the table — both close to 1.0, both with maxCv above 1.6.

Their *global* CVs are similar to opencode and openclaw, but their *local* CVs are much higher. This means they don't have the long-run regime variance that drives the editor-assistant's globalCv up, and they don't have the local consistency that drives opencode/openclaw's meanCv down. They're just locally bursty.

The shape: short, intense, irregular usage. Reach for codex, throw a few prompts at it, walk away. Reach for hermes, do a chunk of work, stop. No automation loop driving smooth output, no long-running background process. Just "human reaches for tool, uses tool unevenly, puts tool down." This is the most natural-looking variance signature in the table.

It's also worth noting that codex has only 53 windows and hermes has 144. Codex in particular has a small sample — 53 windows means roughly 10 days of active periods. The numbers are directionally suggestive but not statistically tight. Hermes is more grounded.

## What the table tells you to do operationally

If you were building an alerting layer on top of bucket-token rate per source, this table tells you exactly what threshold strategy works for each source.

**Editor-assistant** — global CV 2.60 makes any global threshold useless. A line in the sand at "5x the lifetime mean" would fire constantly during normal heavy weeks. The right strategy is to use rolling-window means as the baseline (since meanCv is only 0.88) and alert on deviations from the *current* window. Concretely: alert when token-per-bucket exceeds 3x the trailing 12-active-bucket mean.

**Claude-code** — high CV at every scale. Static thresholds are noise generators. Use percentile-based alerts (e.g., alert when bucket exceeds the trailing p95 by 50%) rather than mean-based alerts. The metric's `p90Cv = 1.326` says that even healthy windows have substantial spread.

**Opencode and openclaw** — low local variance with moderate global. These are the easiest to alert on. A static "current window's stddev exceeds 2× its long-run typical CV of 0.6" would be a reasonable anomaly trigger. The flatness inside windows means departures from the flat regime are detectable.

**Codex and hermes** — high local variance, small sample. Alerting on these is statistically hard. Probably better to *not* alert per-bucket and instead alert on session-level aggregates (where the noise averages out).

## What this metric can't tell you

Three things to be careful about.

**1. CV is mean-sensitive.** A source with mean near zero will have a huge CV from any tiny absolute fluctuation. The editor-assistant's globalCv of 2.60 is partly inflated by its mean being only 25,832 tokens per bucket (from a related metric snapshot). Compare to claude-code's mean of around 13M tokens per bucket — the same absolute stddev would produce a 500× lower CV. Cross-source CV comparisons are *only* meaningful if you remember the means are different by orders of magnitude.

**2. Window size is a tunable parameter.** The default is 12 active buckets. A 24-bucket window would produce systematically smaller within-window CVs (more averaging) and a 6-bucket window would produce larger ones. The choice of 12 is "6 hours of active time" which is roughly a working session — that's a defensible choice but not the only one. Operational use should pick a window that matches the alert decision time scale.

**3. Active-bucket sliding ignores idle gaps.** Two windows could be "12 active buckets" with one spanning 6 wall-clock hours and the other spanning 4 wall-clock days. The metric treats them identically. For sources with sparse activity (the editor-assistant: 320 active buckets over 265 calendar days, so ~83% of calendar buckets are idle), this conflates very different temporal regimes. A wall-clock-windowed variant would behave differently.

## The one-line summary

`globalCv` measures *how variable this source has been over its lifetime*. `meanCv` (over rolling windows) measures *how variable any local stretch is*. The gap between them measures *how much of the variability is regime-shift over time vs burstiness inside any given session*. The editor-assistant has all-regime, low-local. Claude-code has high-everywhere. Opencode and openclaw have low-local, moderate-global, suggesting partly automated loops. Codex and hermes have natural human-bursty signatures.

## Reproduction

```bash
# in pew-insights repo
node dist/cli.js rolling-bucket-cv
```

Snapshot captured `2026-04-26T04:41:54.661Z`. Queue file: `~/.config/pew/queue.jsonl`, 1,519 lines at capture. Window size: 12 active buckets (default). 1,344 windows total across 6 sources. 0 sources too sparse for window. 0 sources with all windows floored.

To explore the time-scale gap further: rerun with `--window 6` and `--window 24` to see how meanCv compresses or expands. To suppress noise from the smallest sources, add `--min-buckets 100` which would drop codex (64 buckets) from the output and keep the table to the five sources with substantial history.
