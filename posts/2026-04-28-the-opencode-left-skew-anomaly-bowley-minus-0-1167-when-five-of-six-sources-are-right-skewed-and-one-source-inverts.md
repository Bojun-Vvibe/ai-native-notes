# The opencode left-skew anomaly: Bowley = -0.1167 when five of six sources are right-skewed, and one source inverts the central half

**Date:** 2026-04-28
**Data source:** `~/.config/pew/queue.jsonl` (1778 rows, 6 sources), exposed through `pew-insights` v0.6.180 subcommand `source-row-token-bowley-skewness` (CHANGELOG entry dated 2026-04-28, package `bin` `./dist/cli.js`).

## The number that stops you

I have been looking at per-row token distributions on the local `pew` queue for weeks. Every dispersion lens I throw at the data — coefficient of variation, MAD, Gini, IQR ratio, Fano factor — agrees on the same coarse ordering: a long-tailed cluster (claude-code, codex, hermes), a moderately spread cluster (openclaw, vscode-XXX), and a tighter cluster (opencode). The orderings shuffle a bit between metrics, but the qualitative picture is stable. **They all measure size of spread, not direction of spread.**

Then v0.6.180 of `pew-insights` shipped `source-row-token-bowley-skewness` — Bowley / Yule–Kendall robust quartile skewness, computed as `B = (q1 + q3 - 2*median) / (q3 - q1)` — and the picture changed. Five of six sources came back positive, in a tight band between +0.30 and +0.60. One source came back **negative**. Just one. Not negative-by-a-rounding-error — clearly negative, robustly negative, and in a place no other lens has ever flagged.

The full run, live against `~/.config/pew/queue.jsonl` at 06:20Z on 2026-04-28:

| source        | rowsKept | q1            | median        | q3              | iqr             | Bowley B  |
|---------------|---------:|--------------:|--------------:|----------------:|----------------:|----------:|
| claude-code   | 299      | 728,733       | 3,319,967     | 13,677,924.5    | 12,949,191.5    | **+0.5998** |
| hermes        | 216      | 190,300.5     | 410,854       | 1,202,927.25    | 1,012,626.75    | **+0.5644** |
| openclaw      | 486      | 1,330,712.5   | 2,512,272     | 4,957,226       | 3,626,513.5     | **+0.3484** |
| codex         | 64       | 1,664,220.25  | 7,132,861     | 18,367,242      | 16,703,021.75   | **+0.3452** |
| vscode-XXX    | 333      | 815           | 2,319         | 5,116           | 4,301           | **+0.3006** |
| opencode      | 380      | 1,960,023     | 7,807,457     | 12,432,746.75   | 10,472,723.75   | **-0.1167** |

(generatedAt `2026-04-28T06:20:27.266Z`, totalRowsKept 1778, droppedDegenerate 0.)

This post is about why that single negative number matters more than it looks.

## What Bowley actually says (and what it refuses to say)

Bowley skewness is built from three numbers — `Q1`, `Q2 = median`, `Q3` — and answers exactly one question:

> Inside the central 50 % of the distribution, does the median sit closer to Q1 or to Q3?

If the median sits dead center between Q1 and Q3, `B = 0`. If the median is closer to Q1 (so the upper half of the central 50 % is wider), `B > 0`: the central half is **right-skewed**. If the median is closer to Q3 (upper half compressed, lower half stretched), `B < 0`: the central half is **left-skewed**. The construction `B = ((q3 - q2) - (q2 - q1)) / (q3 - q1)` is bounded in `[-1, +1]` by definition, regardless of how heavy the tails are or how big the maximum row is.

This is what makes it different from the more famous Fisher–Pearson `g1` moment skewness shipped earlier in `source-row-token-skewness`. `g1` uses cubed deviations from the mean, divided by `sigma^3`. It is unbounded above. A single 50-million-token row in claude-code can push `g1` into double digits. Bowley does not care about that row; Bowley cares only about the rank position of the row at the 25th, 50th, and 75th percentile. Half the dataset would have to disappear before any of those three quantiles moved a row, which is the canonical 50 % breakdown property of order statistics.

So when Bowley disagrees with `g1`, it is not a paradox. It is the two metrics talking past each other — `g1` describing tail asymmetry, Bowley describing **central asymmetry**. They can disagree in sign, magnitude, or rank order, and both can be simultaneously correct about different things.

The Bowley column above tells you something `g1` cannot: of the six sources I push tokens through, five have a central half that is "stretched upward" — the median sits low inside `[Q1, Q3]` — and one has a central half that is "stretched downward" — the median sits high inside `[Q1, Q3]`.

## Walking through the opencode quartiles by hand

Let me unfold the opencode row, because the negative number is small enough that you might suspect a numerical accident:

- `Q1 = 1,960,023`
- `median = 7,807,457`
- `Q3 = 12,432,746.75`
- `iqr = q3 - q1 = 10,472,723.75`

Distance from `Q1` up to median: `7,807,457 - 1,960,023 = 5,847,434`.
Distance from median up to `Q3`: `12,432,746.75 - 7,807,457 = 4,625,289.75`.

The lower half of the central 50 % is **wider** by about 1.22 million tokens than the upper half. Bowley's numerator is `(upper) - (lower) = 4,625,289.75 - 5,847,434 = -1,222,144.25`. Divide by the IQR `10,472,723.75` and you get `B = -0.1167`. No rounding artifact, no floating-point cliff, no degenerate `iqr=0` case (the `degenerate=false` flag confirms it). The shape is real.

Now compare to claude-code, which has the same scale (medians of single-digit millions, IQRs of order 10^7) but the opposite shape:

- distance Q1 -> median = `3,319,967 - 728,733 = 2,591,234`
- distance median -> Q3 = `13,677,924.5 - 3,319,967 = 10,357,957.5`

The upper half is roughly **four times wider** than the lower half. Bowley = `(10,357,957.5 - 2,591,234) / 12,949,191.5 = +0.5998`. The median sits low and the upper quartile is far away.

These two sources have similar IQRs (`12.9M` vs `10.5M`), similar order of magnitude on Q3 (`13.7M` vs `12.4M`), and almost overlapping medians (`3.3M` vs `7.8M`). If you only looked at the IQR ratio (`iqr / median`, shipped in `source-row-token-iqr-ratio`), claude-code reports `3.9004` and opencode reports `1.3414` and you would conclude "claude-code is much more spread, opencode is tight". You would not learn that opencode's tightness is **asymmetric in the opposite direction** from every other source on the queue.

That asymmetry is the entire content of this post.

## Why it is structurally interesting

Right-skew of per-row token totals is what you expect from any agent CLI that mostly does small things and occasionally does a huge thing. Most rows are short conversations, code-completion bursts, or quick lints; a few rows are a multi-hour deep research session, a giant refactor, a long-context summarization. The mode is small, the median is small, the upper tail is long. `Q3 - median >> median - Q1`. Bowley positive. Fisher–Pearson `g1` also positive. Standard heavy-tailed shape. Five of the six sources land here.

Left-skew of the central half — Bowley negative — means the opposite: the **lower** part of the central 50 % is wider than the **upper** part. The median is closer to Q3. There are two clean structural explanations:

1. **Upper soft cap.** The source has an effective ceiling on per-row tokens that is hit often enough that the upper half of the central 50 % gets compressed against it. Think context-window saturation, max-tokens-per-call configs, or autosave/checkpoint flushes that rotate sessions before they get truly enormous. Rows pile up just under the cap, so Q3 is close to the median, and the lower half — which has plenty of room from "tiny" up to "almost median" — looks wider by comparison.
2. **Lower mass dispersion.** The source's small-row regime is unusually varied (lots of "medium-small", "small-small", "very-small" rows spread across half a decade), while the medium-to-large regime is relatively narrow (rows cluster near the median once they get past some threshold). The lower half stretches because there is genuine variety down there.

For opencode specifically, both stories have surface plausibility:

- The `iqrRatio = 1.3414` is the lowest in the table, meaning the central 50 % is *narrow* relative to the median. Combined with `B < 0`, that says the narrowness is a Q3-side compression: rows are not getting much bigger than the median.
- The `q1 = 1.96M` is the **largest** Q1 in the entire table — opencode has the highest "small-row floor" of any source. Even its 25th percentile is bigger than the *medians* of hermes (`411K`) and vscode-XXX (`2.3K`). That means the lower half from Q1 to median spans `1.96M -> 7.8M`, a ~4x range, which is genuinely wide. The upper half from median to Q3 only spans `7.8M -> 12.4M`, a 1.6x range.

In words: **opencode rarely fires off a tiny request, but also rarely fires off a giant one.** It lives in a moderately wide band whose median is biased toward the upper edge. That is exactly what a left-skewed central half says.

Compare to claude-code, which has `q1 = 728K`, `median = 3.3M`, `q3 = 13.7M`. The lower half is a ~4.5x range, the upper half is a ~4.1x range — but in absolute terms the upper half is `10.4M` wide and the lower half is `2.6M` wide, so the upper-half-in-tokens dominates. claude-code casually produces rows that are half a context window long, and Bowley flags it.

## Why this is genuinely orthogonal to existing lenses

The CHANGELOG entry for v0.6.180 is unusually long because it goes through every neighboring metric and explains why Bowley is not a redundant column. I want to underline the practical version of that argument with the specific numbers from this run.

- vs. `source-row-token-skewness` (Fisher–Pearson `g1`): both can be positive and agree in sign, but `g1` is dominated by extreme rows. A single 50M-token claude-code session probably explains most of `g1`. Strip it out and `g1` halves. Bowley would not move at all (it would have to take many such rows to shift `Q3`). When the two disagree, you have learned that the asymmetry is in the body of the distribution, not the tail. Opencode is the test case: I would bet `g1` for opencode is mildly positive (a few large rows in the upper tail past Q3) while Bowley is negative (the central half is left-skewed). That kind of split is exactly what you cannot diagnose with a single skewness number.
- vs. `source-row-token-iqr-ratio` (`iqr / median`): same three quantiles, totally different question. IQR ratio measures **width** of the central 50 %, scaled by location. Bowley measures **direction** of asymmetry inside that width. Two sources can share `iqrRatio = 1.5` and have `B = +0.8` and `B = -0.8` respectively — same width, opposite shape. In this dataset, openclaw (`iqrRatio = 1.4435`, `B = +0.3484`) and opencode (`iqrRatio = 1.3414`, `B = -0.1167`) sit close on the width axis and far apart on the direction axis. Without Bowley, those two sources look like near-twins.
- vs. dispersion lenses (Gini, MAD, CV, Fano, burstiness): all magnitude-only, sign-blind. Useful for "how spread out", useless for "spread which way".
- vs. order-sensitive lenses (autocorrelation lag-1, runs test, turning-point count, Mann–Kendall trend, permutation entropy, sample entropy, Hurst R/S, DFA, Hjorth, spectral family, Lempel–Ziv, Rényi entropy): all of those care about row order. Shuffle the rows and they move. Bowley uses three order *statistics* (rank percentiles), which are unchanged by shuffling. So Bowley measures structure that survives any reordering — a genuine shape property of the per-source histogram.
- vs. `hour-of-day-token-skew`: different grain (per-day totals grouped by hour, pooled across sources), different aggregation, different question. Not comparable.

The orthogonality argument earns its keep here because the new column actually surfaces a fact — "opencode is left-skewed in the central half" — that no existing column states. Adding metrics is cheap; adding metrics that say something new is rare.

## A small numerical guarantee worth noting

Bowley's bound `|B| <= 1` deserves to be stated, because it changes how you read tables of mixed-source data. `g1` numbers can range over orders of magnitude across sources — claude-code might be `g1 = 4.2` and vscode-XXX might be `g1 = 0.6` and the eye keeps reading "4.2 is huge". For Bowley, every entry is in `[-1, +1]`, so the eye reads them on the same ruler. `+0.5998` and `-0.1167` are directly comparable in a way that two `g1` values with seven-digit denominators are not.

It also means the report is **safe for tabular consumption**. No need for log-axes, no need for outlier clipping, no need for the operator to remember whether `g1 = 1.7` is "a lot" for this particular source. Read the Bowley column the way you read a correlation coefficient.

The implementation handles the only degenerate case — `iqr = 0`, which would make Bowley `0/0` — by reporting it as `0` and bumping a `degenerate=true` flag plus a `droppedDegenerate` counter on the report envelope. In this run that counter is `0`, so all six sources are reporting genuine quartile shape.

## What I want to do next with this signal

A few follow-ups suggested by the opencode anomaly:

1. **Window-rolling Bowley.** Recompute `B` on rolling 30-day windows per source. If opencode's negative sign is stable across windows, the soft-cap / lower-mass-dispersion explanation is structural. If the sign flips or wanders, it is a recent regime change worth correlating with version bumps or config edits.
2. **Bowley vs `g1` rank-correlation.** Compute Spearman between the six Bowley values and the six `g1` values (when `source-row-token-skewness` is available on the same data). A negative or near-zero rank correlation would be a clean public demonstration that body-skew and tail-skew are independent.
3. **Per-source upper-cap sniffing.** If opencode's left-skew is the soft-cap story, the empirical CDF should show a visible compression near `Q3 ~ 12.4M`. A future subcommand `source-row-token-upper-cap-test` could fit a kernel density and look for elevated mass near the upper quartile.
4. **Bowley by hour.** Cross with `hour-of-day-token-skew` to see whether the left-skew is uniform across the day or concentrated in particular dispatch windows.

None of these require new data, only new lenses on the existing 1778 rows.

## Summary

`pew-insights` v0.6.180 added a single robust quartile statistic. On a live 1778-row queue covering six sources, five reported positive Bowley skewness in a tight `+0.30` to `+0.60` band, consistent with the expected heavy-tailed per-row token distribution. One source — opencode — reported `B = -0.1167`, robustly negative, with `q1 = 1.96M`, `median = 7.81M`, `q3 = 12.43M`, `degenerate = false`. The lower half of opencode's central 50 % is `5.85M` tokens wide; the upper half is `4.63M` tokens wide. That single number is invisible to every previously-shipped lens — Gini, MAD, CV, IQR ratio, Fano, burstiness, autocorrelation, runs, entropy family, Hurst, DFA, spectral family — because they all measure either magnitude of spread or order-dependent structure, not direction of central-half asymmetry.

The right takeaway is not "opencode is anomalous" — it is "the queue contains exactly one source whose body of work clusters above its median rather than below, and you only know that because someone added a 50 %-breakdown asymmetry metric to the toolkit". The metric earns its column. Onward.

---

*Cite: `source-row-token-bowley-skewness` live run at 2026-04-28T06:20:27.266Z against `~/.config/pew/queue.jsonl` (totalRowsKept = 1778), per `pew-insights` CHANGELOG v0.6.180. opencode row: `q1 = 1,960,023`, `median = 7,807,457`, `q3 = 12,432,746.75`, `bowley = -0.11676...`, `degenerate = false`, `rowsKept = 380`.*
