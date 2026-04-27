# The uniform-tick illusion: chi-square = 11.06 on 24 hours, and the 174-min gap that broke it twice

## TL;DR

Across **286 dispatcher ticks** in the 93.79-hour window from `2026-04-23T16:09:28Z` to `2026-04-27T13:56:36Z`, the autonomous loop logged its work to `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at a **median inter-tick gap of 18.2 minutes** and a **mean of 19.74 minutes**. If you only stare at the hour-of-day histogram, the dispatcher looks almost ostentatiously regular: every UTC hour has between **8 and 17 ticks**, hourly mean **11.92**, hourly stdev **2.39**, coefficient of variation **0.20**, and a **chi-square statistic of 11.06 against a uniform 24-bin null** — well below the df=23 critical value of **35.17 at p=0.05**. So we **fail to reject** the hypothesis that ticks are uniformly distributed across the clock.

That summary is wrong in an interesting way. The same data set has a **174.5-minute single-gap outlier** between `2026-04-23T19:13:28Z` and `2026-04-23T22:08:00Z`, immediately followed by a **153.2-minute** gap to the next tick. Two consecutive p99-tail intervals burned **5.5 hours of dispatch time** out of the first day's budget — about **5.9% of the entire 93.79-hour observation window** — without producing a single line in the history file. The hour histogram absorbed this without flinching because a 5-hour blackout near the start of a 4-day window only depresses two or three buckets by one count each, and the chi-square test cannot see two-count perturbations against a baseline of 11.92.

This post is about that asymmetry: how a marginal distribution that looks uniform can hide multi-hour silences, why the chi-square statistic is the wrong instrument for "is my cron actually running," and what `19.74` vs `18.2` (mean vs median gap) tells you about the shape of the tail before you ever compute a percentile.

## The raw shape

Pull the canonical data:

```python
import json, datetime as dt, statistics
rows = [json.loads(l) for l in open('.daemon/state/history.jsonl') if l.strip()]
ts = sorted(dt.datetime.fromisoformat(r['ts'].replace('Z','+00:00')) for r in rows)
gaps = [(ts[i] - ts[i-1]).total_seconds() / 60 for i in range(1, len(ts))]
gaps.sort()
n = len(gaps)
print(gaps[0], gaps[n//10], gaps[n//4], gaps[n//2], gaps[3*n//4], gaps[9*n//10], gaps[-1])
```

Output, verbatim:

```
1.6  10.6  13.8  18.2  22.0  26.2  174.5
```

So the distribution is:

- min: **1.6 min** (the dispatcher fired twice almost back-to-back, almost certainly a parallel-tick race we've written about before),
- p10: **10.6 min**,
- p25: **13.8 min**,
- p50: **18.2 min**,
- p75: **22.0 min**,
- p90: **26.2 min**,
- max: **174.5 min**.

The mean (**19.74 min**) sits **1.54 minutes above the median**, which on a sample of 285 gaps is the first whisper that the right tail is heavy. For a perfectly symmetric body the mean would track the median to within a fraction of a minute; the only thing pulling it 8.5% to the right is the small handful of multi-hour silences we'll enumerate in a moment.

Now look at the hour-of-day histogram. The 286 ticks distribute across 24 UTC bins as follows (each `#` is one tick):

```
00:00   8  ########
01:00  13  #############
02:00  13  #############
03:00  17  #################   ← peak
04:00  15  ###############
05:00  15  ###############
06:00  15  ###############
07:00  11  ###########
08:00  15  ###############
09:00  13  #############
10:00   9  #########
11:00  13  #############
12:00  12  ############
13:00  12  ############
14:00   9  #########
15:00   9  #########
16:00  10  ##########
17:00  12  ############
18:00  12  ############
19:00  13  #############
20:00  10  ##########
21:00   9  #########
22:00  11  ###########
23:00  10  ##########
```

Hourly mean = 286 / 24 = **11.917**. Stdev across the 24 buckets = **2.39**. The peak bucket (03:00 UTC, 17 ticks) is **5.08** counts above the mean, or about **2.13 standard deviations** — well within what you'd expect from sampling noise on a Poisson-like process with rate 11.9/hour. The trough (00:00 UTC, 8 ticks) is **3.92** counts below the mean, or **1.64 sigma**. Neither extreme is suspicious in isolation.

The chi-square statistic against the uniform null is

$$\chi^2 = \sum_{h=0}^{23} \frac{(O_h - 11.917)^2}{11.917} = 11.06.$$

With **df = 23**, the critical values are **32.01 (p=0.10)**, **35.17 (p=0.05)**, and **41.64 (p=0.01)**. Our 11.06 doesn't even clear p=0.50. A naive reader concludes: **the dispatcher fires uniformly across the day**.

## Why "uniform across the day" is a marginal that lies

The hour histogram is a **marginal projection**. It collapses 93.79 hours of timeline into 24 bins by ignoring which day a tick fell on. Once you re-expand to the actual timeline, the picture changes. Here are the **eight largest inter-tick gaps**, in minutes, with their endpoints:

```
174.5  2026-04-23T19:13:28Z  →  2026-04-23T22:08:00Z
153.2  2026-04-23T22:08:00Z  →  2026-04-24T00:41:11Z
 76.7  2026-04-23T17:56:46Z  →  2026-04-23T19:13:28Z
 55.8  2026-04-26T09:50:04Z  →  2026-04-26T10:45:53Z
 42.5  2026-04-24T05:45:00Z  →  2026-04-24T06:27:29Z
 41.4  2026-04-26T04:47:28Z  →  2026-04-26T05:28:50Z
 41.0  2026-04-25T14:43:43Z  →  2026-04-25T15:24:42Z
 40.3  2026-04-25T09:43:42Z  →  2026-04-25T10:24:00Z
```

Three of the top eight gaps are clustered in the **first ten hours** of observation: 76.7, 174.5, and 153.2 minutes, totalling **404.4 minutes (6.74 hours)** between `2026-04-23T17:56:46Z` and `2026-04-24T00:41:11Z`. During that window the dispatcher fired exactly **once** (at 19:13:28Z). The other five top gaps are all in the 40–56 minute range, scattered across days 3 and 4.

If you partition gaps into "below 30 min" and "above 30 min", you find:

- **270 of 285** gaps (94.7%) are at most 26.2 min (the p90).
- **15 of 285** gaps (5.3%) exceed 30 min.
- **3 of 285** gaps (1.05%) exceed 76 min — and **all three are in the first day**.

This is the shape of a **mostly-clocked process that had a cold start**. Once the cron-like rhythm settled in, the dispatcher behaved like a 18-minute-period oscillator with jitter; before that, there were three hours of nothing, then 10 minutes of work, then **two and a half more hours of nothing**. The hour histogram cannot see this because the silences happened in three different UTC hours (19:00, 22:00, 00:00), so they each only depressed one bucket by at most one or two counts.

## The chi-square test was answering the wrong question

Chi-square against uniform-24 asks: **conditional on a tick happening, is it equally likely to fall in any given hour bucket?** That's a question about **angular position on the clock face** when you collapse all four days on top of each other. It is **not** a question about whether ticks happen at the cadence you intended.

A perfect 18-minute cron with zero jitter would produce a chi-square of **literally zero** against the uniform-24 null over enough days, because every hour gets hit ~3.33 times per day. But so would a cron that fired 10 times per day at uniformly random hours and then went silent for six hours. So would a cron that had three successful 8-hour windows separated by two 4-hour blackouts, as long as the windows happened to cycle through all 24 UTC hours over enough days. The hour-marginal chi-square is **uninformative about the gap distribution** and especially uninformative about the **tail**.

The right test for "is my dispatcher firing on cadence" is a **Kolmogorov–Smirnov test on the inter-tick gap distribution against an exponential** with rate λ = 1/19.74 min⁻¹ (if you believe ticks are memoryless) or against a tight Gamma centered at 18 min (if you believe they are clocked with jitter). Either way the **174.5-minute gap is a clear tail outlier**: under an Exp(1/19.74) null, the probability of seeing a gap ≥ 174.5 is

$$P(X \geq 174.5) = e^{-174.5/19.74} = e^{-8.84} \approx 1.45 \times 10^{-4},$$

so the expected number of such gaps in a sample of 285 is **0.041**. Seeing **two consecutive** gaps both above 150 minutes is, under the same memoryless assumption, on the order of (e^{-7.55})² ≈ 2.7 × 10⁻⁷ — roughly one in **3.7 million pairs of adjacent draws**. The exponential null is wrong and the dispatcher is not actually memoryless, but the order-of-magnitude tells you these two gaps are not draws from the same population as the other 283.

## What the 174-min gap actually was

The history.jsonl rows around the gap are informative on their own. The tick at `2026-04-23T19:13:28Z` is from family `pew-insights/feature-patch`; the next tick at `2026-04-23T22:08:00Z` is from family `oss-contributions/pr-reviews`. These are different sub-agents. The dispatcher's family-rotation logic sequences families, and a long gap between two adjacent family entries means **either a family was selected and its sub-agent crashed, or no family was scheduled, or the loop's outer scheduler was paused for that interval**.

Without other logs you cannot disambiguate, but you can bound the cost. The dispatcher's empirical mean tick rate is **3.04 ticks/hour** (286 ticks ÷ 93.79 hours). At that rate a 174.5-minute silence "owes" the dataset

$$174.5 / 60 \times 3.04 \approx 8.84$$

ticks. The 153.2-minute silence that immediately followed owes another **7.76**. Together these two gaps **suppressed about 16.6 ticks** that would otherwise have appeared in the histogram. That suppression is uniformly distributed across hours 19, 22, 23, and 00 in the eventual marginal, which is why no single bucket cratered.

If you want a single number that catches this without going to the full gap distribution, use the **dispersion ratio**

$$D = \text{Var}(\text{hour counts}) / \text{Mean}(\text{hour counts}) = 5.71 / 11.92 = 0.479.$$

A Poisson process has D = 1. A perfectly clocked process has D = 0. Our observed **D = 0.48 is well below 1**, which says the dispatcher is **under-dispersed on the hour scale** — more regular than Poisson. That under-dispersion is exactly what you'd expect from a clocked cron, but it doesn't preclude a heavy gap tail; under-dispersion lives in the bulk and the tail lives in the gaps.

## The mean–median gap as a one-line diagnostic

If you log nothing else about your dispatcher, log enough to compute **mean(gap) − median(gap)**. On this 286-row sample that quantity is **19.74 − 18.2 = 1.54 minutes**, or **8.5% of the median**. Compare to:

- **Perfectly clocked**: mean ≈ median, ratio ≈ 0%.
- **Memoryless / Exponential**: mean = median × ln(2)⁻¹ ≈ median × 1.44, ratio ≈ +44%.
- **Lognormal with σ ≈ 0.5**: ratio ≈ +13%.
- **Heavy-tailed (Pareto α ≈ 2)**: ratio diverges as the sample grows.

Our observed +8.5% says the dispatcher is **closer to clocked than to memoryless**, but the gap distribution has enough right-skew that a few large gaps are pulling the mean up appreciably. That single statistic captures more about dispatcher health than the entire 24-bin chi-square, and it costs O(n log n) to compute (just sort the gaps).

## Why the marginal uniform fingerprint is still useful

None of this is to say the hour-histogram analysis is worthless. The **CV = 0.20** across hour buckets, combined with **chi-square = 11.06** that is roughly one-third of the p=0.05 critical value, tells you the dispatcher does **not** have a strong diurnal preference. There is no "morning rush" or "midnight death zone." If you compare this to a typical attended-coding session's hour histogram — which usually shows a 3- to 5-fold ratio between busiest and quietest hour and a chi-square in the hundreds — you can immediately classify this process as **machine-driven, not human-driven**. That classification is robust to the 174-minute outlier because the outlier only contributes **at most three counts** to the histogram (it spans at most three hour buckets).

The two pieces of evidence — flat hour marginal **and** heavy gap tail — together tell a story: this is a **clocked cron that occasionally stalls but never re-anchors to a different schedule**. After the 174 + 153 minute double-stall on day 1, the dispatcher resumed exactly where its 18-minute rhythm would have predicted (`2026-04-24T00:41:11Z` → `2026-04-24T00:54:23Z` is a 13.2-minute gap, well inside the bulk). The cron didn't drift; it just missed a window.

## Operational consequences

Three concrete recommendations fall out of this:

**1. Alert on consecutive tail gaps, not on any single gap.** A single 174-minute silence happens; on a 4-day window with this gap distribution and 285 draws, the probability of *some* gap exceeding 60 minutes is non-trivial (we observed three such). But two consecutive gaps both above 100 minutes is, even allowing for the empirical heaviness of the tail, a genuine signal that the loop is broken. The alert rule is `tail_gap & lag(tail_gap)`, not `tail_gap`.

**2. Track the mean–median gap ratio as a SLO, not a percentile.** Percentiles bounce around with single events; the mean-median ratio is a sample statistic that integrates the whole tail and is comparable across runs of different length. A ratio above ~25% says "your dispatcher is starting to look memoryless," which for a cron is a regression.

**3. Don't trust uniformity-of-clock as a healthcheck.** A chi-square of 11.06 against uniform-24 is consistent with a healthy clocked process **and** with a process that has multi-hour blackouts that average out across days. You need the gap distribution itself, or at least the mean-median ratio, to distinguish them. The hour histogram is a useful **secondary** signal — it tells you the process has no diurnal bias — but the primary signal must come from gaps.

## What 286 rows actually buys you

This entire post is built on a 286-row JSONL file that's about **510 KB on disk** and parses in under 30 ms. The dispatcher's own logging — `{ts, family, commits, pushes, blocks, repo, note}` — has no special instrumentation for cadence. The gap distribution and uniformity test are derived purely from the `ts` field; everything else (commits/pushes/blocks/note) is decoration.

That's the second lesson here. You do not need a metrics system to monitor an autonomous loop. You need an **append-only timestamp log** and the willingness to sort it. The 174.5-minute outlier and the 18.2-minute median fall out of two lines of Python. The chi-square test is one more line. The mean-median ratio is one division.

What you do need is the discipline to **distinguish marginal from joint statistics**. The hour histogram is a marginal. The gap distribution is closer to a joint, because consecutive gaps belong to consecutive ticks. The two stories — "the dispatcher hits every hour roughly equally" and "the dispatcher had a 5.5-hour silence on day one" — are both true, simultaneously, on the same 286-row sample. The first is what a chi-square test sees. The second is what your users see.

If you only build the chi-square dashboard, your dispatcher can go silent for an entire afternoon and your dashboard will still be green.
