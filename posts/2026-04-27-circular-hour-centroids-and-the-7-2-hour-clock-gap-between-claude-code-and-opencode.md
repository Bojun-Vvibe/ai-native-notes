---
title: "Circular hour centroids and the 7.2-hour clock gap between claude-code and opencode"
date: 2026-04-27
tags: [pew-insights, time-of-day, circular-statistics, source-analysis]
est_reading_time: 11 min
---

## The problem

Every time-of-day chart I have ever built for this stack has lied in the same way. You take a source's token mass, bin it into the 24 UTC hours, average the bin index weighted by mass, and call the resulting number the source's "centroid hour." It is a perfectly defensible thing to do — except that hour 23 and hour 0 are next-door neighbors on the actual clock, and the linear mean treats them as 23 hours apart. So a source whose work clusters around midnight UTC gets a centroid somewhere near hour 11 — the literal worst possible answer — and your dashboard cheerfully informs you that this midnight-loving tool is actually a midday tool.

I had been ignoring this because the obvious top-of-mind tools (`claude-code`, `opencode`) live during ordinary daylight hours, so the linear mean and the circular mean agree to within a fraction of an hour. But this week I finally ran the `source-token-mass-hour-centroid` lens on the full corpus, and the resultant lengths — the `R` value that comes out of the circular-mean computation — told a much more interesting story than the centroid hours themselves. Six sources, six wildly different answers about how concentrated their day is, and one source whose `R = 0.0996` is so close to "uniform across the 24-hour clock" that calling it a centroid at all is borderline misleading.

## The setup

The lens is `pew-insights source-token-mass-hour-centroid`, which treats hour-of-day as a point on the unit circle, weights it by per-bucket `total_tokens`, and reports three things: the angular mean (re-mapped to a 0–24 hour clock as `centroidHour`), the resultant length `R` ∈ [0, 1] (1 = perfectly concentrated at one hour, 0 = uniform around the clock), and a circular standard deviation in hours computed from `R`.

Run on `~/.config/pew/queue.jsonl` at `2026-04-26T21:07:43.718Z`, six sources cleared the 1000-token floor. Total mass: **9,415,159,661** tokens across **6 sources**, no rows dropped for invalid hour-start, sparseness, or filter — clean data. Here is the full table:

| source           | tokens (B)  | nBuckets | centroidHour | R        | spreadHours | peakHour | peakHourTokens |
| ---------------- | ----------- | -------- | -----------: | -------: | ----------: | -------: | -------------: |
| claude-code      | 3.442       | 20       |        8.474 |  0.39693 |       5.193 |        8 |    466,586,192 |
| opencode         | 3.252       | 24       |       15.663 |  0.18690 |       6.996 |       17 |    311,032,700 |
| openclaw         | 1.764       | 24       |        6.315 |  0.09963 |       8.204 |        1 |    128,895,936 |
| codex            | 0.810       | 16       |       11.126 |  0.43592 |       4.922 |       12 |    103,439,933 |
| hermes           | 0.145       | 24       |        7.943 |  0.30310 |       5.902 |       14 |     15,687,670 |
| editor-assistant | 0.00189     | 14       |        6.099 |  0.72739 |       3.048 |        2 |        410,051 |

(`nBuckets` is the count of distinct UTC hours-of-day with positive mass, out of 24 possible. `editor-assistant` is the only source with `nDays = 73` — its tenure runs from `2025-07-30` to `2026-04-20` — versus 7–35 days for everyone else.)

## What I tried

- **First attempt: just sort the centroids and report the order.** This gives `editor-assistant ≈ openclaw ≈ hermes ≈ claude-code (6–8) → codex (11) → opencode (15.6)`. Nice clean ladder from "morning workers" to "afternoon worker." But it ignores the resultant length entirely. A centroid hour with `R = 0.10` is almost meaningless — it is the average of a near-uniform distribution. Two sources can have the same centroid hour and one can be tightly bunched while the other is smeared evenly across the clock.

- **Second attempt: compute the difference between centroid and peak hour as a "drift" indicator.** This was promising in the abstract — if the centroid is far from the peak, the distribution is multi-modal or skewed. The numbers:

  - `claude-code`: centroid 8.47, peak 8 → drift 0.47 (essentially co-located)
  - `opencode`: centroid 15.66, peak 17 → drift 1.34 (afternoon distribution skewed earlier than peak)
  - `openclaw`: centroid 6.31, peak 1 → drift 5.31 (massive — the peak hour is 5 h earlier than the centroid, meaning there's a heavy late-morning tail dragging the centroid forward)
  - `codex`: centroid 11.13, peak 12 → drift 0.87
  - `hermes`: centroid 7.94, peak 14 → drift 6.06 (centroid sits 6 hours *before* the peak, meaning the early-morning mass outweighs the afternoon spike)
  - `editor-assistant`: centroid 6.10, peak 2 → drift 4.10

  This is interesting — `openclaw` and `hermes` are the two "drifty" sources, both with peak/centroid divergence above 5 hours — but it conflates two different things: distributional skew, and the fact that the peak hour is itself a noisy point estimate from a small bucket count.

- **Third attempt: rank by `R` and treat it as the real signal.** This is what finally made sense. `R` has a clean interpretation: it is the magnitude of the vector sum of all hour-of-day positions weighted by token mass, normalized to [0, 1]. `R = 1` means every token landed in the same hour. `R = 0` means perfect uniformity around the clock. Anything below ~0.2 is functionally "no preferred hour-of-day."

## What worked

Sort by `R` descending and read it as a "diurnal coherence" score:

| rank | source           | R       | reading                                        |
| ---: | ---------------- | ------: | ---------------------------------------------- |
| 1    | editor-assistant | 0.72739 | tightly clustered around hour 6 (UTC)          |
| 2    | codex            | 0.43592 | clearly clustered around late morning UTC      |
| 3    | claude-code      | 0.39693 | moderate morning cluster, broad shoulders      |
| 4    | hermes           | 0.30310 | weak preference, multi-modal                   |
| 5    | opencode         | 0.18690 | barely-there afternoon lean                    |
| 6    | openclaw         | 0.09963 | functionally uniform across the clock          |

That last row is the headline finding. `openclaw` has 1.76B tokens spread across all 24 UTC hours (`nBuckets = 24`), and the resultant length is **0.0996**. Its centroid hour (6.315) is mathematically correct but operationally meaningless — there is no "morning openclaw." The tool runs whenever it runs, and the angular center of mass happens to land at 6 a.m. UTC because the noise around the clock canceled almost perfectly.

The 7.2-hour gap in the headline is `15.663 − 8.474 = 7.189` hours between `opencode` (centroid 15.66) and `claude-code` (centroid 8.47). On the surface this looks like the most dramatic finding — these are the two largest sources by token mass, and they sit a third of a day apart on the clock. But the gap is *only* meaningful because both `R` values, while modest (0.40 and 0.19), are above the floor where the centroid still encodes something. For `openclaw` (`R = 0.10`) the centroid hour is decoration, not data.

The fix in code is one line — instead of reporting `centroidHour`, you report a tuple `(centroidHour, R)` and gate any downstream "what hour does this source prefer?" logic on `R > 0.2` or similar. Everything below that floor gets the answer "no preferred hour."

```bash
# Re-rank by diurnal coherence rather than centroid:
pew-insights source-token-mass-hour-centroid --json \
  | jq -r '.sources | sort_by(-.resultantLength) |
           .[] | "\(.source)\t\(.resultantLength | . * 1000 | floor / 1000)\t\(.centroidHour | . * 100 | floor / 100)"'
```

## Why it worked (or: my current best guess)

The reason linear means break on hour-of-day is the same reason they break on compass bearings, lunar phase, and any other modular quantity: the mean of `[23, 0, 1]` is 8, which is neither the visual nor the structural center of those three values. The circular mean treats each hour as a 2D unit vector (`cos θ`, `sin θ`) where `θ = 2π · hour / 24`, sums them with weights, and reads the angle of the resulting vector. The magnitude `R` of that resulting vector falls out for free, and it happens to be exactly what you want as a confidence indicator.

The `R` interpretation has a sharp cutoff in practice. Above `R ≈ 0.7` (only `editor-assistant` here, and that is on a tiny absolute mass — 1.89M tokens), you can talk about "the hour" the source prefers and it will be a tight, defensible window. Between `0.3` and `0.7` (`codex`, `claude-code`, `hermes`), you have a legitimate diurnal pattern but with broad shoulders — the mean is informative but a histogram is more honest. Below `0.2` (`opencode`, `openclaw`), the source simply does not have a preferred hour. Reporting one is making up structure that is not in the data.

The reason `editor-assistant` has the highest `R` of any source despite having the lowest absolute mass is, I think, mostly artifact. It only ran in 14 of 24 hours, and its 1.89M total tokens are dominated by a small number of large rows (the same 487K-row that other posts have flagged). When most of your mass is in one bucket of one hour, `R` shoots up regardless of whether the underlying tool has a "real" diurnal preference. The lens is honest about this — `nBuckets = 14` is right there in the output — but the headline metric does not penalize you for it.

The reason `openclaw` is so close to uniform is the more interesting half. `openclaw` has only 10 calendar days of tenure (`firstDay 2026-04-17`, `lastDay 2026-04-26`) and 1.76B tokens spread over all 24 hours. That is roughly 73M tokens per hour-of-day on average — which sounds like a lot, but the per-hour variance is small enough that the angular sum nearly cancels. This matches the "always-on background batch" character that other lenses have flagged for `openclaw`: it does not have a wake-up clock or a shutdown clock, it has a process loop.

The peak-vs-centroid drift I computed in the second attempt is actually decomposable. For `hermes`, the peak hour is 14 but the centroid is 7.94 — that is the centroid being pulled toward a heavy early-morning shoulder. For `openclaw`, the peak is 1 and the centroid is 6.31 — the centroid is pulled toward a midday tail. In both cases, the peak hour is a single noisy bucket and the centroid is a smoothed estimate that captures the actual mass distribution. When `R` is low, the smoothed estimate is itself noise. When `R` is high (`codex`, `editor-assistant`), the peak and centroid agree to within an hour or two.

A subtler point: the `spreadHours` column is derived from `R` via the standard circular-SD formula `√(−2 ln R) × (24 / 2π)`, so it is not independent information. `R = 0.10` gives `spreadHours = 8.20` (more than a third of the day), `R = 0.73` gives `spreadHours = 3.05`. The ranking by `spreadHours` ascending is the inverse of the ranking by `R` descending. Either column will do; I prefer `R` because the [0, 1] scale is easier to threshold.

## What I would do differently

Two changes to how I read this lens going forward:

1. **Always sort by `R`, not by `centroidHour` or by `tokens`.** Then read centroid hours only for sources above an `R` floor. The default `--sort tokens` is fine for "which sources are biggest," but it puts `openclaw` (one of the most diffuse sources in the dataset) right next to `claude-code` (one of the most coherent ones) and invites apples-to-oranges comparisons.

2. **Stop quoting hour-of-day numbers for `openclaw` entirely.** The lens has run twice this week, both times against ~10 days of `openclaw` tenure, both times producing `R < 0.11`. It is not a diurnal source. Time-of-day analysis on `openclaw` is the wrong lens; the right lens is `interarrival-time` (gap distribution between consecutive active buckets) or `bucket-streak-length` (how long each run lasts before pausing). Diurnal analysis assumes there is a wake/sleep cycle to detect, and `openclaw` does not have one.

The second-order question this lens raises but does not answer is: *why* is `openclaw` so close to uniform? Three hypotheses, none verified yet:

- It is being driven by a 24/7 cron-like loop with a fixed cadence, so its hour-of-day distribution is the marginal of a process that is operationally hour-blind.
- It is being driven by multiple humans in different timezones (or one human with a very broken schedule), and the per-operator diurnal signals cancel in aggregate.
- The 10-day tenure is just too short to see any signal — `R` would rise as the corpus grew.

The clean way to disambiguate would be to re-run the lens daily and watch `R` over time. If `R` is stable around 0.1 across multiple weeks, hypothesis 1 or 2 is correct. If `R` rises monotonically as more days accumulate, hypothesis 3 wins. Neither is hard; I just have not done it yet.

## Links

- `pew-insights source-token-mass-hour-centroid --help` (run locally; no public docs URL)
- `~/.config/pew/queue.jsonl` (the source data, local)
- prior posts on `openclaw` shape: `2026-04-26-source-decay-half-life-...`, `2026-04-26-the-stickiness-asymmetry-...`
