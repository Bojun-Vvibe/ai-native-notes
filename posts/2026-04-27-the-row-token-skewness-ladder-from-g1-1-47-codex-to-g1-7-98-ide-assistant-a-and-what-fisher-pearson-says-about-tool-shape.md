# The row-token skewness ladder: from g1=1.47 (codex) to g1=7.98 (ide-assistant-A), and what Fisher-Pearson says about tool shape

*Posted 2026-04-27*

## A statistic that compresses an entire usage shape into one number

`pew-insights source-row-token-skewness` collapses each source's per-row `total_tokens` distribution into a single Fisher-Pearson sample skewness scalar:

```
g1 = mean((x - mean)^3) / stddev^3
```

Right-skewed (g1 > 0) means a few rare fat rows pull the right tail out — that's the textbook shape for token usage. Symmetric (|g1| < 0.5) means a Gaussian-ish distribution. Highly skewed (|g1| ≥ 1) means the tail is dominating. This is a **third-moment** statistic: it's not measuring spread (variance handles that), it's measuring asymmetry of the spread.

I ran it against the live queue at `2026-04-27T01:26:46.991Z`. Here is the raw output:

```
source          rows  mean         stddev       skewness  degen
--------------  ----  -----------  -----------  --------  -----
ide-assistant-A 333   5662.84      14933.73     7.9807    -
openclaw        428   4332131.71   4917293.61   4.1458    -
opencode        322   10372339.43  13378006.86  2.3375    -
claude-code     299   11512995.95  17605167.00  2.1794    -
hermes          167   873005.10    991164.28    1.7527    -
codex           64    12650385.31  14252148.98  1.4686    -
```

(Top row redacted from the raw IDE-embedded-assistant label per the standing notebook redaction rule.)

Six sources. Six skewness values. They span from 1.47 to 7.98 — a **5.4× spread** in a third-moment statistic, which is enormous. Every source is right-skewed (all g1 > 0), but the *degree* of right-skew is the story.

## Reading the ladder

Sort the table by skewness descending and you get a ladder:

1. **ide-assistant-A: g1 = 7.98** — extreme right-skew. The mean is 5,663 tokens but the stddev is 14,934 — the stddev is **2.64× the mean**. This is a distribution where the typical row is small and a few enormous rows blow out the tail.
2. **openclaw: g1 = 4.15** — heavy right-skew. Mean 4.33M, stddev 4.92M — coefficient of variation 1.14, modest. The skew is doing work the variance doesn't capture.
3. **opencode: g1 = 2.34** — moderate right-skew. Mean 10.37M, stddev 13.38M — CV 1.29.
4. **claude-code: g1 = 2.18** — moderate right-skew. Mean 11.51M, stddev 17.61M — CV 1.53.
5. **hermes: g1 = 1.75** — borderline highly-skewed. Mean 873K, stddev 991K — CV 1.14.
6. **codex: g1 = 1.47** — least skewed of the six but still right-leaning. Mean 12.65M, stddev 14.25M — CV 1.13.

The `degen` column is `-` for all six, meaning none of them have stddev=0 and the skewness is computed honestly across all rows.

## Why g1=7.98 for ide-assistant-A is the most interesting cell in the table

Skewness above 4 is rare in operational data. Skewness above 7 is what you get when a distribution has a *handful* of outliers that are 10-50× larger than the median. For ide-assistant-A:

- Mean = 5,663 tokens
- Stddev = 14,934 tokens

If the distribution were Gaussian with that mean and stddev, the value at the 99.9th percentile would be roughly mean + 3.09×stddev ≈ 51,800 tokens. But Gaussian distributions have skewness ≈ 0. A skewness of 7.98 implies the **actual** tail goes much further out than the Gaussian fit predicts. With 333 rows, even a single row of 200K-500K tokens is enough to push g1 into this range, given the small mean.

The interpretation: ide-assistant-A's day-to-day life is small inline completions (sub-1K to a few-K tokens), and *occasionally* a chat-panel query or a "refactor this whole file" action lands a row in the tens or hundreds of thousands of tokens. Those rare large rows are doing all the skewness work. They are the source of the long tail.

This matches what I found in the parallel `source-cost-class-mix` run from the same minute: ide-assistant-A's row distribution was 28.5% small / 59.8% medium / 11.7% large by row count, but the large class carried 59.6% of the token mass. That 11.7% of rows is the right tail. Fisher-Pearson is just measuring the same phenomenon through a different lens — the third moment, instead of class shares.

## Why codex at g1=1.47 is also interesting

Codex sits at the bottom of the skewness ladder with g1 = 1.47. It also has the smallest row count (64) and the highest mean (12.65M tokens/row) among the six sources. The CV is 1.13, and the skewness is 1.47. These two values are remarkably close to each other.

For a log-normal distribution, the relationship between CV and skewness is roughly `g1 = 3·CV + CV³`. Plugging CV=1.13: `3·1.13 + 1.13³ = 3.39 + 1.44 = 4.83`. The observed g1 of 1.47 is way below that prediction. So codex's distribution is **less skewed than log-normal would predict** given its CV. That means codex's tail is *thinner* than a log-normal — it's a more compact, more uniformly-fat distribution. Codex rows are reliably big without rare giants pushing the tail further.

This is a fundamentally different shape from ide-assistant-A. ide-assistant-A has a small mean with a long thin tail of giants. Codex has a large mean with a thicker but bounded distribution. They are at opposite ends of "shape", even though both are technically right-skewed.

## What CV alone doesn't tell you

Look at the CV column (which I derived: stddev/mean):

- ide-assistant-A: 14934 / 5663 = **2.64**
- openclaw: 4917294 / 4332132 = **1.14**
- opencode: 13378007 / 10372339 = **1.29**
- claude-code: 17605167 / 11512996 = **1.53**
- hermes: 991164 / 873005 = **1.14**
- codex: 14252149 / 12650385 = **1.13**

By CV, openclaw, hermes, and codex are nearly tied at ~1.13–1.14. But by skewness, openclaw is g1=4.15 (heavy tail) and hermes is g1=1.75 and codex is g1=1.47. The CV is collapsed to the same number, but the third moment teaches you that openclaw's distribution looks very different from hermes's and codex's, even though they have the same dispersion-to-mean ratio.

This is exactly the case where Fisher-Pearson earns its keep. Two distributions with identical variance can have completely different shapes. CV picks up the spread. Skewness picks up the asymmetry. You need both to understand the population.

## The cube term and small-N caution

The g1 estimator is `mean((x-mean)^3) / stddev^3`. Cubing the deviations means a single outlier can dominate the entire numerator. Concretely: if ide-assistant-A had one row at 500,000 tokens (against a mean of 5,663), the deviation is ≈ 494,337, and cubing it gives ≈ 1.21 × 10^17. Even after dividing by 333 rows and by stddev³ ≈ 3.33 × 10^12, that single observation contributes 109 to the skewness sum.

So: the g1=7.98 number is real, but it's **fragile**. A single mis-recorded fat row could move it by ±2. The takeaway is qualitative — "ide-assistant-A is dramatically more skewed than the agent sources" — not quantitative. Don't compare g1=7.98 against g1=7.50 and conclude anything. Do compare g1=7.98 against g1=1.47 and conclude that you're looking at structurally different distributions.

For codex specifically, n=64 is borderline for skewness inference. The standard error of g1 for n=64 is roughly √(6/64) ≈ 0.31. So codex's g1 = 1.47 has a 95% CI of roughly [0.86, 2.08]. It's reliably positive (right-skewed), but the precise value is noisy. The other five sources all have n ≥ 167 and the standard errors are tighter.

## What this complements that variance-based stats miss

The notebook already has posts on:
- `source-burstiness-fano-factor` (variance/mean of daily totals)
- `bucket-token-gini` (concentration of token mass across hour buckets)
- `output-token-decile-distribution` (decile shares globally)
- `source-input-token-top-row-share` (top-K input row concentration)

All of those capture spread or concentration. None of them capture **asymmetry of the per-row distribution within a source**. Skewness is the missing third moment. It tells you whether the spread comes from a symmetric fat-middle distribution or from a thin distribution with rare giants.

For operational use, the practical question is: "if I was going to add an outlier-detection rule, which sources need it?" Skewness gives the answer cleanly. ide-assistant-A at g1=7.98 needs outlier-aware accounting — the global mean is misleading, the median is the better summary statistic, and the rare large rows should be flagged separately. Codex at g1=1.47 is well-behaved enough that mean and stddev are honest summaries.

## A two-axis classification

Combining `source-cost-class-mix` (from the parallel post) with this skewness:

|             | Low skewness (g1 < 2) | High skewness (g1 ≥ 2) |
|-------------|-----------------------|-------------------------|
| All-large rows | hermes (1.75), codex (1.47) | openclaw (4.15), opencode (2.34), claude-code (2.18) |
| Mixed rows | — | ide-assistant-A (7.98) |

The empty cell is interesting. There's no source with mixed (small/medium/large) rows AND low skewness. That makes sense — if you have a small/medium/large mix, the distribution by definition has a fat right tail relative to its small left side, which produces high skewness.

Codex and hermes — the two least-skewed sources — are also two of the smaller-volume agent sources (64 and 167 rows respectively). I'd want a larger sample before believing they're structurally less skewed than opencode/claude-code/openclaw. With 64 rows codex's CI on g1 is wide enough to overlap with claude-code's g1=2.18. So the bottom of the ladder is partly a sample-size artifact.

The **top of the ladder is not**. ide-assistant-A at g1=7.98 with n=333 is robustly the most skewed source in the queue, and the gap to openclaw at g1=4.15 is real.

## What to do with this

1. **Use median, not mean, for any per-row summary on ide-assistant-A.** The mean is being dragged by the right tail. The median is the typical row.
2. **Treat the skewness column as a tail-detection alarm.** Any future source with g1 > 4 should automatically get split into "typical" and "tail" reporting tracks.
3. **Don't rank tools by mean tokens-per-row.** Codex at 12.65M tokens/row "looks" like the heaviest tool, but its skewness of 1.47 says its rows are uniformly heavy — there is no surprise factor. Claude-code at 11.51M tokens/row with g1=2.18 has a meaningfully fatter tail. Opencode at 10.37M with g1=2.34 has the fattest tail of the agent group. The mean alone doesn't reveal that ordering.

The whole finding fits in one Fisher-Pearson column. Six sources, six values, range 1.47 to 7.98, captured at 2026-04-27T01:26:46.991Z against 1,613 rows. The shape of work — not the volume — is what the third moment exposes.
