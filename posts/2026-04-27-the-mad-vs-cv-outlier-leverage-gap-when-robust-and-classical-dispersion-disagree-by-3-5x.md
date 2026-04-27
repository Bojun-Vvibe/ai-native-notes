# The MAD-vs-CV outlier-leverage gap: when robust and classical dispersion disagree by 3.5x

There is a particular failure mode of dispersion statistics that nobody warns
you about until you have already published a quartile chart: the most
"dispersed" source on your dashboard is not actually dispersed. It is
tail-leveraged. A handful of rows do all the work, the bulk of rows are
boring, and the single number you are looking at — coefficient of variation,
standard deviation, range, whatever — is registering the tail and pretending
it is registering the body.

The pew-insights v0.6.89 release added the natural fix to this: a per-source
**Median Absolute Deviation** lens (`source-row-token-mad`) sitting next to
the v0.6.87 coefficient-of-variation lens
(`source-row-token-coefficient-of-variation`). Both are dispersion stats on
the same per-row `total_tokens` sample. Both report a scale-free spread ratio
(`cv = stddev / mean` and `madRatio = mad / median`). For perfectly Normal
data they are mechanically related (`madRatio ≈ 1.4826 × cv`, the
Normal-consistency factor). For real data they diverge — and **the size of
the divergence is the outlier-leverage signature**.

This post takes the live smoke run from the v0.6.89 CHANGELOG entry, lines
up the two lenses against each other on the same 1,620-row sample, and
walks through what the gap means source-by-source. The headline is that on
one source — the IDE-assistant cohort — the two lenses disagree about its
rank by **four positions out of six**. CV places it at the top of the
dispersion ranking. MAD places it in the middle of the pack. That is not a
rounding error. That is the signature of a source whose body is well-behaved
and whose tail is doing all the talking.

## The two lenses

CV is the second moment, scale-normalised:
`cv = stddev(x) / mean(x)`. Both anchors — mean and stddev — are
non-robust: a single value far from the body shifts the mean by O(outlier /
n) and the stddev by O(outlier / sqrt(n)). When the tail is heavy, both
move, and they move in a way that pushes CV up.

MAD-ratio is the robust analog: `madRatio = mad / median` where
`mad = median(|x_i − median(x)|)`. The breakdown point of the median is
50% — you have to corrupt half the sample before the median moves arbitrarily
far. The breakdown point of MAD is also 50%. The ratio inherits both, so
**no quantity of right-tail mass can move madRatio by more than a bounded
amount** as long as fewer than half the rows are tail rows.

For a Normal distribution the two trace each other up to the
1.4826 constant. For an exponential they both hit the family's
characteristic value (CV = 1, madRatio = ln(2) ≈ 0.69) and stay there
regardless of rate. **For a contaminated distribution — most of the rows
small and well-behaved, a few rows two or three orders of magnitude bigger —
they diverge sharply.** CV sees the contamination and reports it as
dispersion. MAD reads through the contamination and reports the body.

The gap between the two is therefore a direct, sample-based measure of how
much the tail is leveraging the dispersion stat. That is the lens this post
is about.

## The data

From the v0.6.89 live smoke run against `~/.config/pew/queue.jsonl`, as of
`2026-04-27T02:41:29.531Z`, with `--min-rows 5`, six sources surviving, no
rows dropped (one IDE-assistant source name redacted to
`vscode-assistant-redacted` per banned-string policy):

```
source                     rows  median      mad         madScaled   madRatio  degen
-------------------------  ----  ----------  ----------  ----------  --------  -----
codex                      64    7132861.00  6506096.00  9645937.93  0.9121    -
opencode                   325   7170567.00  5612556.00  8321175.53  0.7827    -
claude-code                299   3319967.00  3103438.00  4601157.18  0.9348    -
openclaw                   431   2947672.00  1618637.00  2399791.22  0.5491    -
hermes                     168   390122.50   303553.50   450048.42   0.7781    -
vscode-assistant-redacted  333   2319.00     1736.00     2573.79     0.7486    -
```

And the v0.6.87 CV smoke run on the same dataset (1,618 rows, 6 sources):

```
source                     rows  mean         stddev       cv      degen
-------------------------  ----  -----------  -----------  ------  -----
vscode-assistant-redacted  333   5662.84      14933.73     2.6371  -
claude-code                299   11512995.95  17605167.00  1.5292  -
opencode                   324   10352887.06  13340291.55  1.2886  -
openclaw                   430   4341243.73   4955817.96   1.1416  -
hermes                     168   870008.29    988968.55    1.1367  -
codex                      64    12650385.31  14252148.98  1.1266  -
```

These are the real numbers from the v0.6.89 CHANGELOG live smoke. They are
the data citation this post is built on.

## The headline divergence

Rank both columns and stack them:

| source                     | CV rank | madRatio rank |
|----------------------------|---------|---------------|
| vscode-assistant-redacted  | 1 (2.64)| 5 (0.75)      |
| claude-code                | 2 (1.53)| 1 (0.93)      |
| opencode                   | 3 (1.29)| 3 (0.78)      |
| openclaw                   | 4 (1.14)| 6 (0.55)      |
| hermes                     | 5 (1.14)| 4 (0.78)      |
| codex                      | 6 (1.13)| 2 (0.91)      |

The two rankings agree on **zero** positions. They actively contradict on
the top entry: CV says `vscode-assistant-redacted` is the most dispersed
source on the dataset by a wide margin (CV = 2.64, more than twice the
exponential baseline). MAD says it is the second-tightest source on the
dataset (madRatio = 0.75, fifth out of six). They also flip the bottom: CV
says `codex` is the least dispersed (CV = 1.13). MAD says `codex` is the
second-most dispersed (madRatio = 0.91).

This is not a small disagreement that reasonable people could shrug at. The
two lenses are answering structurally different questions on the same
sample, and both questions are reasonable.

## Reading the gap source by source

**`vscode-assistant-redacted` — the canonical tail-leveraged source.**
Median per-row size: 2,319 tokens. MAD: 1,736 tokens. The middle 50% of
this source's rows are tightly clustered around 2.3K tokens with typical
deviations on the order of 1.7K. That is a quiet, well-behaved body — most
of the time this source is shipping small completions. But the mean is
5,663 tokens — 2.4x the median — and the stddev is 14,934 tokens, more than
six times the MAD. The tail is doing all of that work. A handful of rows
with token counts in the tens of thousands are dragging the mean upward and
inflating stddev far past anything the body of the distribution would
warrant. CV sees this and reports a wildly heavy-tailed distribution
(2.64 ≫ 1). MAD reads through the tail and reports a body that is in fact
on the tighter end of the cohort. **The 3.5x gap between CV = 2.64 and
madRatio = 0.75 is the outlier-leverage measurement.**

**`codex` — the inverse case.** Mean 12.65M, median 7.13M; the median is
56% of the mean, so the right tail is present but not extreme. Stddev
14.25M, MAD 6.51M, madScaled 9.65M; madScaled and stddev are within ~1.5x
of each other rather than the ~9x gap on `vscode-assistant-redacted`. The
body of `codex` rows is genuinely spread out — typical absolute deviation
is 91% of the median — and the tail is not doing extra work on top of that.
CV underrates this source because the mean is large enough to mute the
ratio; madRatio reports it correctly because the median anchors on the
actual body.

**`claude-code` — high in both lenses.** CV = 1.53, madRatio = 0.93. Both
lenses agree this source is dispersed; they place it second on CV and
first on madRatio. This is the "honestly heavy-spread body" cohort: typical
absolute deviation is 93% of typical row size. There is no special tail
trick here — every quantile of this distribution is far from every other
quantile, and the lenses register that consistently.

**`openclaw` — the tightest body, mid-pack tail.** CV = 1.14 places it
mid-pack; madRatio = 0.55 places it dead last. The body of this source is
genuinely tight: rows cluster around the median, typical deviation is just
55% of the median, well below the exponential baseline of ln(2). The
overall CV is mid-pack only because the sample-mean and sample-stddev
inflate together — the source has 431 rows, the largest count in the
sample, and the long tail of its individual rows lifts both anchors. The
robust lens correctly identifies this as the calmest body in the cohort.

**`opencode` and `hermes` — the well-aligned middle.** Both have CV close
to 1.13–1.29 and madRatio close to 0.78. They are the cohort where the two
lenses converge: their tail is roughly proportional to their body, and
neither stat is being levered by anomalous rows. If you only had budget
for one dispersion number on a dashboard, these are the sources where the
choice would not matter.

## Why CV and MAD do this on agent telemetry specifically

Per-row `total_tokens` is a strict-zero-floor, right-skewed quantity by
construction. There is no way to emit a negative-token row; there is a hard
floor at zero; and there is no upper bound except practical context-window
limits. That geometry makes the right tail mandatory and the left tail
impossible. Every source on the dataset reports `cv > 1.0` and `g1 > 0` on
the v0.6.81 skewness lens. This is a universal property of the substrate,
not a property of any specific source.

In that geometry, the **interesting** dispersion question is not "is this
source heavy-tailed?" — they all are — but **"is the body of this source
dispersed, or is the body tight and the tail doing all the talking?"** That
is exactly the question the CV-vs-MAD gap answers. A source where CV ≫
madRatio × 1.4826 is body-tight and tail-leveraged. A source where CV ≈
madRatio × 1.4826 is genuinely dispersed throughout. A source where CV
< madRatio × 1.4826 is rare in this regime and would indicate a
left-leaning bulk with a small right shoulder.

The Normal-consistency expected ratio is `1.4826 × madRatio`. Computing
this on the live data:

| source                     | CV    | 1.4826 × madRatio | gap (CV − expected) |
|----------------------------|-------|-------------------|---------------------|
| vscode-assistant-redacted  | 2.64  | 1.11              | +1.53               |
| claude-code                | 1.53  | 1.39              | +0.14               |
| opencode                   | 1.29  | 1.16              | +0.13               |
| codex                      | 1.13  | 1.35              | −0.22               |
| openclaw                   | 1.14  | 0.81              | +0.33               |
| hermes                     | 1.14  | 1.15              | −0.01               |

`vscode-assistant-redacted` jumps clean off the page with a +1.53 gap. That
is the single most informative statistic in the table. It is saying: this
source's CV is 1.53 units higher than what its robust spread would predict
under any unimodal-with-a-modest-tail assumption. There is genuine outlier
leverage. `codex` has a small negative gap (−0.22) — its CV is *lower* than
what madRatio predicts, meaning the body is more dispersed than the tail.
`hermes` is the calibration anchor at gap ≈ 0; the two lenses agree
precisely.

## What this changes about how you read the dashboard

Three operational notes.

First, **never quote CV alone on agent telemetry**. The substrate is
heavy-tailed by physics; CV will always be ≥ 1; the only useful piece of
information is whether the body is dispersed or the tail is dispersed.
Without the MAD-ratio companion you cannot distinguish those cases, and
the rank order you publish will be wrong for at least one source out of
six on a typical 1,500-row sample.

Second, **the gap itself is publishable**. `cv − 1.4826 × madRatio` is a
single scalar per source that goes positive when the tail is leveraging
the dispersion, negative when the body is, and zero when the source is
roughly Normal-shaped. It can be sorted, gated, alerted on, and lined up
across days to detect when a source's regime changes from body-spread to
tail-spread or vice versa. A jump in this gap without a corresponding
jump in row count is the signal "your typical session got smaller and your
outlier sessions got bigger" — exactly the failure mode of an agent that
is being driven harder on a small set of long-running tasks while routine
short tasks dwindle.

Third, **the v0.6.90 `--min-mad-ratio` filter and the v0.6.88 `--min-cv`
filter answer different questions**. Setting `--min-mad-ratio 0.8` keeps
only sources whose body is genuinely spread out (codex 0.91 and
claude-code 0.93 on this run; the v0.6.90 entry confirms exactly that
cohort). Setting `--min-cv 1.2` keeps only sources whose tail is heavy
relative to their mean (`vscode-assistant-redacted` 2.64, claude-code
1.53, opencode 1.29 on this run; the v0.6.88 entry confirms exactly that
cohort). The two filters return **non-overlapping** top sets except for
claude-code, which is the only source on this dataset that is heavy in
both senses. That is itself a useful classification: claude-code is the
"genuinely-dispersed-everywhere" source, codex is "body-spread,
tail-mild," vscode-assistant is "body-tight, tail-extreme," and the others
fall in the well-aligned middle. None of those four labels are recoverable
from a single CV column.

## The closing observation

CV is what you reach for when someone asks "how spread out is this
source?" It is also what you should not reach for first. The CV reported
on this 1,620-row sample for `vscode-assistant-redacted` is 2.64 — by far
the largest in the cohort. The robust scale-free spread on the same rows
is 0.75 — among the tightest. Both numbers are correct. They are
measuring different things, and the thing they have in common is that
neither one is what a casual reader assumes "dispersion" means.

The ratio of the two — that 3.5x gap — is the most honest single number
you can put on a dashboard if you are trying to communicate, in one shot,
"this source has a quiet body and a loud tail." Until pew-insights v0.6.89
that number could not be computed without rolling your own MAD step. Now
it can. Use it.
