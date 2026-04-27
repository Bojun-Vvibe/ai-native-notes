---
title: "The Fano factor by source: vscode-XXX 2.64 vs hermes 1.14 — when variance eats its mean six thousand times over, and what dispersion-relative-to-magnitude tells you that CV cannot"
date: 2026-04-28
tags: [telemetry, statistics, dispersion, fano-factor, pew, queue-jsonl, variance, tokens]
est_reading_time: 14 min
---

## The problem

Pull `~/.config/pew/queue.jsonl` (1,721 rows as of this snapshot) and ask the obvious question: how variable is per-row token usage *within* each source, relative to what that source typically emits? The first instinct is to reach for the coefficient of variation — `stdev / mean` — because it is dimensionless, easy to teach, and shows up in every introductory statistics text. The CV table for the six sources currently in `queue.jsonl` looks comforting at first:

| source         | n   | mean total_tokens | CV    |
|----------------|-----|------------------:|------:|
| openclaw       | 467 |       4,102,379.7 | 1.178 |
| opencode       | 361 |      10,647,637.6 | 1.212 |
| vscode-XXX     | 333 |           5,662.8 | 2.641 |
| claude-code    | 299 |      11,512,995.9 | 1.532 |
| hermes         | 197 |         838,641.6 | 1.144 |
| codex          |  64 |      12,650,385.3 | 1.136 |

Read straight, this says: vscode-XXX is the wild outlier (CV 2.64 — variance more than two and a half times the mean²), and the four heavyweight sources cluster sensibly between 1.14 and 1.53. Comforting, but misleading. CV is the wrong dispersion metric for this dataset, because it normalizes by the *square* of the mean and therefore collapses six orders of magnitude of absolute variance into a tight 2× spread. The interesting story — the one that actually tells you which sources will torture your downstream rolling-window aggregations and which ones will sit politely inside their confidence intervals — is the **Fano factor**: variance divided by mean, not by mean squared. And on Fano, the spread is not 2×. It is approximately **24,000×**, with vscode-XXX at 39,501 and opencode at 15,641,768 on the same `total_tokens` column. Same dataset. Same row count order of magnitude. Same nominal "tokens" units. The dispersion-relative-to-magnitude story is dramatically different from the dispersion-relative-to-magnitude-squared story, and almost everyone defaults to the wrong one.

## The setup

- **Data source.** `~/.config/pew/queue.jsonl`, the per-hour-per-model-per-source rollup that the pew collector writes. Schema fields available: `cached_input_tokens`, `device_id`, `hour_start`, `input_tokens`, `model`, `output_tokens`, `reasoning_output_tokens`, `source`, `total_tokens`. No error column, no latency column — this is a pure usage table.
- **Snapshot.** 1,721 rows total. Six distinct `source` values, with row counts ranging from 64 (codex) to 467 (openclaw). Distribution is reasonably balanced for the four heavyweight sources (197–467) and the two outliers behave differently for different reasons (vscode-XXX has 333 rows but a *tiny* mean; codex has only 64 rows but a *huge* mean).
- **Reference SHA.** Computed against pew-insights HEAD `4574e08` ("docs(changelog): cross-link v0.6.146 base entry to v0.6.147 refinement") which is the same base I used for the spectral-rolloff lens this morning. Same data, different lens.
- **The two ratios under comparison.**
  - **CV** = `stdev / mean` = `sqrt(var) / mean`. Dimensionless. Squared, it equals `var / mean²`.
  - **Fano factor** = `var / mean`. Has units of "tokens" in this case (because the numerator is tokens² and the denominator is tokens). Not dimensionless.

The choice between dividing by `mean` versus dividing by `mean²` sounds like a pedantic accounting move. It is not. It changes which sources look "well-behaved" by a factor of about 700.

## The Fano-factor table

Computed in plain Python over `queue.jsonl`:

```python
import json, statistics
from collections import defaultdict
rows = [json.loads(l) for l in open('/Users/bojun/.config/pew/queue.jsonl')]
by_src = defaultdict(list)
for r in rows: by_src[r['source']].append(r['total_tokens'])
for src, vals in sorted(by_src.items(), key=lambda x: -len(x[1])):
    m = statistics.mean(vals); v = statistics.variance(vals)
    print(f"{src:15s} n={len(vals)} mean={m:12.1f} Fano={v/m:14.1f} CV={(v**0.5)/m:.3f}")
```

Output (unmodified):

| source         | n   | mean total_tokens | Fano (var/mean) | CV    |
|----------------|-----|------------------:|----------------:|------:|
| openclaw       | 467 |       4,102,379.7 |       5,696,609 | 1.178 |
| opencode       | 361 |      10,647,637.6 |      15,641,768 | 1.212 |
| vscode-XXX     | 333 |           5,662.8 |          39,501 | 2.641 |
| claude-code    | 299 |      11,512,995.9 |      27,011,386 | 1.532 |
| hermes         | 197 |         838,641.6 |       1,096,956 | 1.144 |
| codex          |  64 |      12,650,385.3 |      16,311,593 | 1.136 |

CV ranks the sources: hermes < codex < openclaw < opencode < claude-code < vscode-XXX. Fano ranks them: vscode-XXX < hermes < openclaw < opencode < codex < claude-code. **The most "stable-looking" source under CV (hermes at 1.144) is the second-most-stable under Fano. The most "unstable" source under CV (vscode-XXX at 2.641) is the *most* stable source under Fano (39,501).** They almost completely disagree on the right tail.

The reason is mechanical and clean: CV² = Fano / mean. So if two sources have very different means, their CV and Fano rankings only agree when their variances scale linearly with their means. They don't. The token distribution emitted by vscode-XXX (a code-completion-style harness with tight per-call usage) has a tiny mean and a tiny variance — but the variance/mean² ratio is large because the mean is small enough that even modest fluctuations look "huge in proportion." Meanwhile claude-code (a long-form conversational harness) has a 2,000× larger mean and a much larger absolute variance, but the variance does not scale with mean², so the variance/mean ratio (Fano) is what actually quantifies its instability per emitted token.

## What Fano is, mechanically

Fano factor was originally defined for counting processes — the variance in the number of events you observe in a fixed window, divided by the mean number of events you observe. For a Poisson process, Fano = 1 exactly: variance equals mean. Sub-Poisson processes have Fano < 1 (regular, pacemaker-like). Super-Poisson processes have Fano > 1 (bursty, clustered).

When you apply Fano to a non-counting quantity like `total_tokens`, the literal interpretation breaks (tokens are not events; they have units), but the *normalization principle* is still the right one for "how many of my own units of magnitude do I jitter by?" If a source emits a typical 10M tokens per row and its variance is 15M tokens², then a one-standard-deviation row jitters by sqrt(15M) ≈ 3,873 tokens. The Fano of 15M tells you that the variance burns through 15M of the source's own magnitude-units per row. That is the number you want when you are asking "how big a margin do I need around this source's mean to capture 1σ?" CV does not give you that — it gives you the answer rescaled by the mean again, which is fine for cross-domain comparison but actively misleading for "how much absolute variance is each source contributing to my joined dataset?"

## The breakdown across the four token sub-fields

`queue.jsonl` carries four token columns: `input_tokens`, `output_tokens`, `reasoning_output_tokens`, `cached_input_tokens`. They behave very differently. Computed Fano factors:

**input_tokens (Fano):**

| source       | n   |     mean   |       Fano  |
|--------------|-----|-----------:|------------:|
| openclaw     | 467 | 2,208,381  |   2,931,867 |
| opencode     | 361 |   651,922  |   1,417,659 |
| vscode-XXX   | 333 |     1,745  |     117,547 |
| claude-code  | 299 | 6,135,831  |  13,258,233 |
| hermes       | 197 |   296,256  |     724,286 |
| codex        |  64 | 6,418,456  |   8,140,963 |

**output_tokens (Fano):**

| source       | n   |     mean   |     Fano    |
|--------------|-----|-----------:|------------:|
| openclaw     | 467 |    10,549  |      31,595 |
| opencode     | 361 |    72,930  |      67,649 |
| vscode-XXX   | 333 |     3,409  |       7,000 |
| claude-code  | 299 |    40,564  |     112,399 |
| hermes       | 197 |     8,860  |       6,513 |
| codex        |  64 |    31,953  |      44,277 |

**reasoning_output_tokens (Fano):**

| source       | n   |    mean   |    Fano    |
|--------------|-----|----------:|-----------:|
| openclaw     | 467 |       0   |   (all 0)  |
| opencode     | 361 |     387.4 |     19,213 |
| vscode-XXX   | 333 |     508.7 |      3,655 |
| claude-code  | 299 |       0   |   (all 0)  |
| hermes       | 197 |       0.3 |         58 |
| codex        |  64 |  12,333.4 |     20,415 |

**cached_input_tokens (Fano):**

| source       | n   |     mean   |       Fano  |
|--------------|-----|-----------:|------------:|
| openclaw     | 467 | 1,883,448  |   2,772,064 |
| opencode     | 361 | 9,922,396  |  15,088,809 |
| vscode-XXX   | 333 |       0    |   (all 0)   |
| claude-code  | 299 | 5,336,599  |  14,027,086 |
| hermes       | 197 |   533,524  |     809,160 |
| codex        |  64 | 6,187,642  |   8,116,674 |

Three observations fall out of these tables that you would not see from a CV-only view:

**1. Output is dramatically tighter than input across every single source.** For openclaw, input Fano (2.9M) is roughly 93× larger than output Fano (31.5K). For opencode, the ratio is 21×. For claude-code, it is 118×. For hermes, 111×. The asymmetry is consistent and large: assistants emit a much more constrained distribution of tokens per row than the prompts they consume. This is the natural shape of conversational harnesses — input is whatever the user feeds in plus accumulated context, while output is bounded by stop conditions, max_tokens, and the model's own length priors. The Fano view makes the asymmetry numerically vivid; the CV view does not, because input and output have different means and the normalization confuses you.

**2. Cached-input Fano is comparable to input Fano in the heavy-cache sources, and cached-input Fano is *zero* in the only source that does not cache.** vscode-XXX has zero cached-input tokens across all 333 rows — it is the only source in the dataset that does not participate in prefix caching. Every other source shows cached_input Fano of the same order as input Fano (within 2×), which is consistent with cache hits being roughly proportional to input volume per row, not anti-correlated with it. If cache utilization were a separate axis from input size, we would expect the two Fanos to decouple. They do not.

**3. The reasoning column has a binary failure pattern.** openclaw and claude-code have *zero* reasoning tokens in every row. opencode, vscode-XXX, hermes, and codex have nonzero reasoning. This is not noise — it is a model-routing decision baked into the harness: openclaw and claude-code currently route exclusively to non-reasoning model variants in this snapshot window, while the other four route at least sometimes to reasoning variants. Fano on reasoning_output_tokens is therefore best read as *conditional* on the harness routing to a reasoning model at all. For codex, where reasoning fires more reliably, mean reasoning is 12,333 and Fano is 20,415 — Fano is 1.65× the mean, which is super-Poisson but only modestly so. For opencode and vscode-XXX, where reasoning is sporadic, Fano is dramatically larger than the mean (50× and 7×) because most rows are zero and the few non-zero rows are huge.

## Why CV misled me first

I had been using CV as my default dispersion metric for the per-source token tables for about three weeks. The reason is structural: CV is what `pandas.DataFrame.describe()` and most spreadsheet pivot tables hand you for free. You compute mean and stdev, divide them, you get a number that looks like "1.2," and you write the conclusion "moderate variability."

What CV hides: when sources have very different means, equal CVs do not mean equal absolute jitter, and unequal CVs do not mean unequal absolute jitter either. CV answers the question "what fraction of my mean is one stdev?" That question only matters if you care about *relative* error bars per source. If you are joining the sources into a unified dataset and asking how much absolute variance each one contributes to the pooled estimate, you need Fano (or, equivalently, raw variance with the mean factored back in for normalization context).

A worked example with the actual numbers: vscode-XXX has CV 2.641 and Fano 39,501. opencode has CV 1.212 and Fano 15,641,768. CV says vscode-XXX is the more variable source by 2.2×. Fano says opencode is more variable than vscode-XXX by 396×. Both ratios are correct — they are answering different questions. If you are deciding whether to trust vscode-XXX's per-row mean as a usable point estimate, CV's answer ("2.6× the mean is one stdev — don't trust it") is correct. If you are deciding which source dominates the pooled variance of your joined table, Fano's answer ("opencode swamps vscode-XXX by 400×") is correct.

The mistake I had been making was using CV's answer to inform pooled-variance decisions. Specifically: I had been treating vscode-XXX as the "main source of noise" in a hourly rollup that combined all six sources, and I had been excluding it from rolling-window aggregations to "stabilize" the signal. The numbers say the opposite: excluding vscode-XXX removes 39,501 units of variance per row from the pool, while leaving in opencode keeps 15.6M units of variance per row in the pool. The "stabilization" I thought I was getting was a rounding error against the actual variance contribution of the heavy-source rows.

## A diagnostic procedure

Going forward, when looking at any per-source token rollup from `queue.jsonl`, the right ladder is:

1. **Compute means first.** Sort sources by mean. If means span more than one order of magnitude, CV is going to lie to you for some pair of sources. The current dataset spans 5 orders of magnitude in mean total_tokens (5,663 for vscode-XXX up to 12,650,385 for codex), so CV is unreliable for almost every cross-source comparison.

2. **Compute Fano second.** This is your "absolute variance per unit of own-mean" number. It is the right number for pooled-variance reasoning, for Bayesian priors over per-row token usage, and for sample-size calculations.

3. **Compute CV third, and only as a within-source consistency check.** CV is fine for asking "is this source emitting a tighter distribution this week than last week?" because the mean is anchored. It is not fine for cross-source comparison.

4. **Always check for structural zeros.** If a column is all zeros for some sources (reasoning_output_tokens, cached_input_tokens), treat those sources as a separate population. Mixing them into a joint Fano calculation produces nonsense — the mean is dragged toward zero and the Fano blows up artificially.

The Fano-factor lens does not invent any new information; it is just `var / mean` instead of `var / mean²`. But that one-character difference in the denominator changes which source you blame for instability in your pooled telemetry table, and in this dataset it changes the answer by a factor of 400×. Worth getting right.

## What this changes downstream

Three concrete consequences for the analytics I am running on top of `queue.jsonl`:

- **Hourly rollups should weight by inverse Fano, not by inverse CV.** Inverse-variance weighting is the standard meta-analysis move when combining estimates of different precision. The right "precision" here is `1 / Fano` in the units of the source's own mean, not `1 / CV²`.
- **The vscode-XXX rows are not the noise floor.** They are the *quietest* rows in the table by absolute variance contribution. Removing them does almost nothing to the pooled variance. The noise floor is set by the heavy-source long-tail rows (claude-code 27M Fano, opencode 15.6M, codex 16.3M), and those are precisely the rows you want to keep for the analytics signal you actually care about.
- **Outlier detection thresholds need to be source-conditional.** A row with 50K total_tokens is a 9× mean for vscode-XXX (extreme outlier) and a 0.005× mean for codex (well below average). Any global outlier rule applied to the pooled table will simultaneously over-flag vscode-XXX and under-flag the heavy sources.

The CV table at the top of this post is the one I had been quietly looking at for weeks. The Fano table is the one I should have been looking at. Same data file, same 1,721 rows, same six sources — the dispersion lens I chose was hiding a 400× misallocation of where my actual variance was coming from.
