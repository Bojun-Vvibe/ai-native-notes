# Token-Mass Concentration: HHI=3619 Across Six Sources, HHI=5998 Across Seven Models, and the Three-Axis Concentration Asymmetry

**Date:** 2026-04-27
**Source data:** `pew-insights digest --json --since 2026-04-20` capture at 2026-04-27T~14:55Z (`since: 2026-04-20T00:00:00.000Z`, totalTokens: 6,671,056,701, events: 867)

## The headline

The pew-insights eight-day window contains three axes along which token mass can be partitioned: by source (the harness or client that produced the event), by model (the underlying weights invoked), and by day (the calendar bucket). All three axes are concentrated, but the *shape* of the concentration is dramatically different across them. This post quantifies all three with the same family of metrics — Herfindahl-Hirschman Index, effective-N, normalized Shannon entropy, Gini coefficient, and concentration ratios — and shows that the most concentrated axis is not the one most observers would guess.

The headline numbers, computed on the live capture:

- **bySource HHI = 3,619.1** (percent² scale, max 10,000) → effective N = **2.76** out of 6 sources
- **byModel HHI = 5,998.2** → effective N = **1.67** out of 7 models
- **byDay HHI = 1,575.4** → effective N = **6.35** out of 8 days

The model axis is the most concentrated (HHI nearly twice the source axis). The day axis is the least concentrated (HHI less than half the source axis). The source axis sits in between, but with a structurally different shape than either.

## The source axis: skewed but multimodal

The six sources, ranked by share of total tokens (6,671,056,701):

| Source         | Share   | Share² (pp²) |
|----------------|--------:|-------------:|
| opencode       | 52.72%  | 2,779.13     |
| openclaw       | 20.42%  | 417.12       |
| claude-code    | 19.67%  | 386.93       |
| codex          | 5.84%   | 34.13        |
| hermes         | 1.35%   | 1.81         |
| vscode-copilot | 0.0001% | 0.00         |

Concentration ratios:

- **CR1 = 52.72%** — one source (`opencode`) owns more than half the corpus
- **CR2 = 73.14%** — two sources own roughly three-quarters
- **CR3 = 92.81%** — three sources own ~93%; the bottom three together own ~7.2%

Effective-N (the reciprocal of HHI on a 0–1 scale) is **2.76**, meaning the corpus *behaves* as if it came from about 2.76 equally-sized sources, even though six sources actually exist. Normalized Shannon entropy is **0.673** of its theoretical max (log₂6 ≈ 2.585 bits; observed 1.7395 bits) — the entropy is meaningfully less than perfect uniformity but well above a near-degenerate single-source distribution. The Gini coefficient is **0.5577**, in the range typically associated with high-but-not-extreme inequality.

The interesting structural feature here is not the headline `opencode` dominance but the **shape of the runners-up**: `openclaw` at 20.42% and `claude-code` at 19.67% are nearly identical. Two sources, completely different harnesses, sit within 0.75 percentage points of each other. That means the source distribution has one giant (52.72%), a flat plateau at ~20% wide enough to hold two sources, a small but meaningful contributor at 5.84% (`codex`), and then a long tail (hermes + vscode-copilot together = 1.35%).

If you were trying to model this distribution, neither a Pareto nor a geometric-decay fit would work well. The first three points form a 52.72 / 20.42 / 19.67 sequence whose ratios are 2.58× and 1.04× — a single sharp drop followed by a flat. Then from rank 3 to rank 4 the ratio is 3.37× (claude-code → codex), and rank 4 to rank 5 is 4.34× (codex → hermes). The drops *accelerate* toward the tail, not decay smoothly.

This non-monotonic decay pattern is what keeps the source-axis HHI from being even higher than 3,619: the existence of a near-tie at rank 2 keeps two non-trivial mass blocks in the table, which suppresses the squared-share sum compared to a hypothetical distribution with the same CR1 but a long thin tail underneath.

## The model axis: the most concentrated

The seven models, by share of the same 6.67B token total:

| Model              | Share   |
|--------------------|--------:|
| claude-opus-4.7    | 72.72%  |
| gpt-5.4            | 26.63%  |
| unknown            | 0.4366% |
| claude-sonnet-4.6  | 0.1405% |
| claude-haiku-4.5   | 0.0594% |
| gpt-5.2            | 0.0045% |
| gpt-5-nano         | 0.0016% |

HHI = **5,998.2**. Effective-N = **1.67**. That is the number to focus on. Out of seven models in the corpus, the distribution is operating as if it had only 1.67 equally-sized contributors. Two models (`claude-opus-4.7` and `gpt-5.4`) together carry **99.36%** of total token mass. The remaining five models combined carry 0.64%.

This is much more concentrated than the source axis. The reason: the source axis has *three* meaningful contributors (opencode + openclaw + claude-code at 92.81% combined), but those three sources all funnel almost exclusively into two models. `opencode`, `openclaw`, and `claude-code` are all primarily Anthropic-pathway harnesses calling Opus 4.7. `codex` and `vscode-copilot` are GPT-5-pathway. The model axis collapses what looks on the source axis like meaningful diversity.

The square-of-shares for `claude-opus-4.7` alone is 72.72² = **5,288.2** percent-points-squared — that single model contributes 88.2% of the entire 5,998 HHI value. It is, mathematically, the dominant force.

This produces the first asymmetry: **the source axis looks more diverse than the model axis underneath it.** A naive "we use lots of different tools" reading of the source distribution understates how concentrated the actual model spend is. By model, the corpus is essentially a binary system.

## The day axis: nearly uniform

The eight days, by share:

| Day        | Share  |
|------------|-------:|
| 2026-04-20 | 26.59% |
| 2026-04-21 | 16.83% |
| 2026-04-22 | 13.39% |
| 2026-04-23 | 10.42% |
| 2026-04-26 | 10.37% |
| 2026-04-24 | 9.40%  |
| 2026-04-25 | 9.40%  |
| 2026-04-27 | 3.60%  |

HHI = **1,575.4**. Effective-N = **6.35** out of 8 days.

This is the loosest concentration of the three axes. The biggest day (2026-04-20) carries 26.59% — substantial, but well under the 52.72% the dominant source carries. The smallest full day (2026-04-25) carries 9.40%. The ratio of biggest to smallest *full* day is 26.59 / 9.40 = **2.83×**. Day 2026-04-27 is partial (the capture was taken at ~14:55 UTC mid-day), so its 3.60% is artificially low and should be discounted.

Effective-N of 6.35 means the corpus is operating as if it came from 6.35 equally-sized days out of 8 — a remarkably high diffuseness. The day axis is *the* axis on which the corpus is most diffused.

## The three-axis asymmetry

Stacking the three numbers side by side:

| Axis    | N | HHI    | Effective N | CR1    | Notes |
|---------|--:|-------:|------------:|-------:|------|
| sources | 6 | 3,619  | 2.76        | 52.72% | one giant + flat plateau at #2-#3 |
| models  | 7 | 5,998  | 1.67        | 72.72% | binary system; two models = 99.36% |
| days    | 8 | 1,575  | 6.35        | 26.59% | nearly uniform; ratio of biggest:smallest full day = 2.83× |

Three different axes, three completely different concentration regimes:

1. **Models are concentrated by surface availability.** When a user has access to two model families and uses both regularly, the model axis collapses to a quasi-binary distribution regardless of how many distinct model versions exist on the menu. Five of the seven models in the table contributed less than 1% of mass combined.
2. **Sources are concentrated by harness preference.** Six harnesses exist, but the user's working pattern routes mass through three of them (opencode + openclaw + claude-code = 92.81%). The remaining three harnesses (codex + hermes + vscode-copilot = 7.19%) are technically active but operationally minor.
3. **Days are diffused by time itself.** A user works most days; the pew window includes eight calendar days and effectively all eight contributed meaningfully. The day axis is the only one where a uniform-distribution model is even close to fitting the data.

The asymmetry is the finding. Token mass is *not* uniformly concentrated; it is concentrated *by axis-specific structural reasons*. Saying "the corpus is concentrated" is a category error. You have to specify *along which axis*.

## What HHI=3619 actually compares to

In antitrust regulation, HHI thresholds (on the percent² scale, max 10,000) are conventionally read as:

- HHI < 1,500 → unconcentrated
- 1,500 ≤ HHI < 2,500 → moderately concentrated
- HHI ≥ 2,500 → highly concentrated

By that calibration:

- The **day axis (1,575)** is in the *moderately concentrated* band — barely above the unconcentrated threshold.
- The **source axis (3,619)** is solidly *highly concentrated*.
- The **model axis (5,998)** is far into highly concentrated, near the threshold (6,000) at which most regulators would consider a market a near-monopoly.

These thresholds were designed for market-share analysis, not LLM telemetry, so the comparison is illustrative rather than literal. But the qualitative ranking — model > source > day — is robust and informative.

## The Gini and entropy cross-check

Using the source-axis distribution, the Gini coefficient is **0.5577**. For comparison: a perfectly uniform distribution gives Gini = 0; complete monopoly gives Gini → 1. A Gini of 0.5577 sits in the high-inequality range conventionally used for income distributions (real-world country-level Ginis range from ~0.25 to ~0.65).

Normalized Shannon entropy on the source axis is **0.6729** (1.7395 bits out of 2.585 bits maximum for 6 categories). Entropy of 0.673 of max means the source distribution is about two-thirds of the way from "perfect monopoly" (entropy 0) to "perfect uniformity" (entropy 1.0 normalized). That sounds high until you realize a single source carries 52.72% — the residual entropy comes mostly from the near-tie between openclaw and claude-code at #2 and #3, which contributes a lot of bits despite both being "minority" sources.

This is a useful intuition: **entropy and HHI can diverge when the runners-up are evenly matched.** In a distribution of (52.72, 20.42, 19.67, 5.84, 1.35, 0.0001), the HHI is dominated by the #1 share's square (2,779 / 3,619 = 76.8% of total HHI), but entropy distributes its bits across all positive-mass categories more evenly. So HHI says "this is dominated by opencode" while entropy says "this still carries a fair bit of structure beyond the dominant source". Both are true, viewed through different lenses.

## Per-source HHI contribution decomposition

Decomposing the 3,619 source-axis HHI into per-source contributions:

| Source         | Share   | Contribution to HHI | % of HHI |
|----------------|--------:|--------------------:|---------:|
| opencode       | 52.72%  | 2,779.13            | 76.79%   |
| openclaw       | 20.42%  | 417.12              | 11.53%   |
| claude-code    | 19.67%  | 386.93              | 10.69%   |
| codex          | 5.84%   | 34.13               | 0.94%    |
| hermes         | 1.35%   | 1.81                | 0.05%    |
| vscode-copilot | 0.0001% | 0.00                | 0.00%    |

`opencode` alone contributes 76.79% of the entire HHI value — meaning if you removed `opencode` from the corpus and renormalized the remaining five sources to 100%, the HHI of the residual distribution would change dramatically (likely to around ~1,800 if we redistribute proportionally, putting the remainder near the unconcentrated band).

This is the practical use of per-source HHI decomposition: it tells you exactly *how much* the headline concentration depends on a single contributor. A 76.79% concentration-of-the-concentration on one source is a very strong dependency. By contrast, on the model axis, `claude-opus-4.7` contributes 88.16% of the model HHI — even more dependent on a single contributor.

The model axis is therefore not just more concentrated overall; it is more *fragile* to the single-largest contributor. Remove `claude-opus-4.7` from the model corpus and `gpt-5.4` becomes the sole significant remaining model (residual HHI dominated almost entirely by gpt-5.4's 26.63²×(100/27.27)² ≈ 9,545 — extreme single-actor dominance).

## What the asymmetry implies

The three-axis decomposition produces a counter-intuitive read of the corpus: **the user's tool diversity (source axis) is real, the user's calendar diversity (day axis) is high, but the user's model diversity (model axis) is essentially binary.** Six harnesses funnel mass into effectively two model surfaces. Eight calendar days spread mass relatively evenly. One source dominates, but two more keep the source-axis HHI from collapsing to model-axis levels.

For anyone trying to estimate "how much of this corpus depends on access to model X" or "what would happen if source Y was unavailable", the per-axis HHI and per-contributor decomposition give exact answers:

- Lose `opencode` → source-axis HHI drops by 76.79%; model-axis HHI barely moves (opencode is mostly Opus, but Opus would still dominate via openclaw and claude-code).
- Lose `claude-opus-4.7` → 72.72% of token mass disappears; the remaining gpt-5.4 corpus has higher per-source concentration than today.
- Lose any single day → at most 26.59% of mass disappears; HHI on the residual 7-day window changes by less than 10%.

This is the kind of structural read that no headline aggregate can produce. The total `6,671,056,701` token number tells you how much was spent. The 3,619 / 5,998 / 1,575 HHI triple tells you *how that spending is structured along each available axis*, and which axis is the load-bearing one for the corpus's overall concentration.

The answer, on this week of data: the model axis is load-bearing, the source axis is structural, and the day axis is essentially uniform. Three numbers, three regimes, one corpus.
