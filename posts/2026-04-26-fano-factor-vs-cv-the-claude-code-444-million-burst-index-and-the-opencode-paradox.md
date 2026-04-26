---
title: "Fano factor vs CV: the claude-code 444-million burst index and the opencode paradox of a steady source with a 100M Fano score"
date: 2026-04-26
tags: [pew-insights, dispersion, fano-factor, source-burstiness, telemetry]
data_source: ~/.config/pew/queue.jsonl via pew-insights v0.5.6 source-burstiness-fano-factor
generated_at: 2026-04-26T18:23:00.999Z
window_total_tokens: 9,353,043,468
---

## The shape of "bursty"

There are at least four different ways to call a usage source "bursty," and they almost never agree with each other. The most popular one in operator dashboards is the coefficient of variation — `cv = stddev / mean` — because it is dimensionless and easy to put in a sparkline tooltip. The most popular one in queueing-theory papers is the Fano factor — `F = variance / mean` — because it has a natural reference point at `F = 1` (the Poisson baseline). The two indices look almost the same on paper. They are not the same in practice, and the per-source breakdown of `pew-insights source-burstiness-fano-factor` against my local `~/.config/pew/queue.jsonl` (9,353,043,468 tokens across 6 sources, generated `2026-04-26T18:23:00.999Z` by pew-insights v0.5.6) makes the disagreement loud enough to be the entire post.

The full, unredacted-but-policy-redacted output:

| source           | active days | mean / day      | stddev / day    | Fano factor   | CV    |
|------------------|-------------|-----------------|-----------------|---------------|-------|
| claude-code      | 35          |  98,353,879     | 209,106,432     | 444,573,212   | 2.126 |
| codex            |  8          | 101,203,082     | 122,703,257     | 148,771,055   | 1.212 |
| opencode         |  7          | 457,211,628     | 214,255,310     | 100,402,822   | 0.469 |
| openclaw         | 10          | 175,342,900     |  98,919,851     |  55,805,721   | 0.564 |
| hermes           | 10          |  14,523,689     |  10,386,581     |   7,427,938   | 0.715 |
| ide-assistant-A  | 73          |      25,831     |      46,558     |      83,915   | 1.802 |

Two sources jump out for opposite reasons.

## The claude-code 444-million burst index

`claude-code` has the highest Fano factor in the corpus by a wide margin: **444,573,212**. That is "tokens of variance per token of mean," which is a slightly weird unit but the right one — Fano is dimensional in whatever count you feed it, and tokens-per-day is what the queue records. The raw mean is 98.35M tokens/day across 35 active calendar days; the raw stddev is 209.10M tokens/day. Stddev is 2.13× the mean — a CV of 2.13 — which is also the highest CV of any source with more than a thousand tokens of mean.

In queueing terms, `F ≈ 4.4 × 10⁸` is many orders of magnitude above the Poisson baseline of 1. This is not a Poisson process. The arrivals are **clustered**: a few extreme-volume days carry most of the mass, separated by stretches of light or zero activity. That matches the corpus shape — claude-code's tenure spans `2026-02-11` through `2026-04-23`, 71 calendar days, but only 35 of them registered any positive tokens (a 49% activity rate). On the active days, the daily total ranges from a few million tokens to several hundred million, and the variance term in `F = σ²/μ` blows up quadratically in the tail.

This is consistent with what other lenses on the same corpus have already said: the "max-over-mean outlier coefficient" post from earlier today reported a 33.9× spike on `ide-assistant-A` and a 4.6× floor on codex; the "stickiness asymmetry" post reported claude-code returns to itself 86.6% of the time after a session boundary; the "per-source heaviest single row" post called out a single 56-million-token claude-code outlier. Fano is just the second-moment summary of all of those phenomena. It is the magnitude of clustering, not its location, not its shape.

## The opencode paradox: huge Fano, tiny CV

The line that breaks the simple "Fano = bursty" reading is `opencode`:

- Fano factor: **100,402,822** (third-highest in the corpus, 22% of claude-code's value)
- CV: **0.469** (the *lowest* of any source)

If you had only the Fano factor and a single threshold ("F > 10⁶ = bursty"), you would file opencode as a clustered source. If you had only the CV ("CV < 0.5 = steady"), you would file it as the steadiest source on the dashboard. Both readings come from the same seven daily totals.

What is going on: opencode's mean is 457.2M tokens/day — by far the largest of any source. Its stddev is 214.2M, which is large in absolute terms but only 47% of the mean. Fano = σ²/μ = (214.2M)² / 457.2M ≈ 100.4M; CV = σ/μ = 214.2M / 457.2M ≈ 0.469. Fano scales with the mean (it has units of tokens), so a heavy source with even modest relative dispersion will out-Fano a light source with chaotic shape. CV strips out the units and gives you the shape directly.

For an operator, this means: **Fano is the right index when you are dimensioning capacity** (it tells you how much cushion above the mean you need in the same unit you are buying capacity in), and **CV is the right index when you are diagnosing behavior** (whether a tool's traffic pattern is volatile in a relative sense). Conflating them is how you end up either over-provisioning the steady tools or under-investigating the bursty ones.

opencode itself shipped 3,200,481,397 tokens over its first 7 active calendar days (`2026-04-20` to `2026-04-26`) — that is the entire trailing week, no off-days, which immediately tells you something about my own work pattern as the operator. It is on, every day, and the daily totals don't swing by more than a factor of ~2. By contrast claude-code has 35 active days inside a 71-day tenure and the daily totals span at least an order of magnitude.

## The ide-assistant-A inversion

The opposite extreme is the smallest source by mass: ide-assistant-A, with a Fano factor of just **83,915** but a CV of **1.802** — the second-highest in the corpus. Mean is 25,831 tokens/day across 73 active days inside a 9-month tenure (first seen `2025-07-30`, last seen `2026-04-20`). The stddev is 46,558 — almost double the mean — which is precisely the same shape as claude-code's distribution but at four orders of magnitude smaller absolute scale.

If you were ranking by Fano alone, you would dismiss ide-assistant-A as the most stable source on the dashboard. Its 83,915 is six orders of magnitude smaller than claude-code's 444M. But the CV says: relatively, this source is just as bursty as claude-code; it just has a tiny mean. Most of its 73 active days are under-mean noise; a handful of days carry visible mass.

This is the same paradox as opencode but flipped. opencode has a big Fano because it has a big mean, despite a calm CV. ide-assistant-A has a small Fano because it has a small mean, despite a wild CV. **Neither index alone is sufficient.** The pew-insights report ships both columns for exactly this reason; if you only print one, you get fooled by half the corpus.

## Calibrating against the Poisson baseline

All six sources are super-Poisson. The smallest Fano in the corpus is ide-assistant-A's 83,915 — still 4.9 orders of magnitude above `F = 1`. Even hermes, which is not a primary tool and only logs 14.5M tokens/day mean, comes in at F = 7.4M.

That is not surprising once you think about what a Poisson token process would look like: arrival rate constant in time, no clustering, no idle gaps that aren't independently sampled. The pew queue records *user-driven* tool usage, which is the canonical example of a non-Poisson process — humans batch their work into sessions, and sessions cluster around hours-of-day, days-of-week, and project deadlines. The interesting question is not "is it bursty?" (it always is) but "how much, in what unit, and ranked against what baseline?" That requires both Fano and CV side by side.

A cleaner reference would be a *negative-binomial* baseline rather than Poisson — negative binomial is the standard model for over-dispersed count data, and its dispersion parameter has a direct relationship with the Fano factor. I might try fitting one in a follow-up post; for now the pew-insights output gives me Fano and CV, which is enough to separate "bursty in absolute tokens" (claude-code) from "bursty relative to its own mean" (ide-assistant-A) from "looks bursty by Fano but isn't" (opencode).

## Why the gap between sort orders matters operationally

The pew-insights subcommand defaults to sorting by Fano descending, which puts claude-code, codex, opencode, openclaw, hermes, ide-assistant-A in that order. If I re-sort by CV descending, the order becomes claude-code, ide-assistant-A, codex, hermes, openclaw, opencode — a near-reversal at the bottom of the list. opencode falls from third place to last place; ide-assistant-A jumps from last place to second.

For capacity planning (how many TPM headroom should I leave for tool X?), the Fano sort is the right one. For tool-shape characterisation (which tools have the most volatile day-to-day pattern?), the CV sort is the right one. For *anomaly detection*, neither alone is right — you want to flag a source whose daily Fano or daily CV moves significantly from its trailing baseline, which is what `daily-token-zscore-extremes` and `rolling-bucket-cv` exist for. Fano and CV are *steady-state* dispersion summaries; they are not designed to spot a single weird day.

The overall corpus mean is 9.35B / (35+8+7+10+10+73) = ~63.4M tokens per source-active-day. Three of the six sources are above that mean (opencode at 457M, openclaw at 175M, codex at 101M); three are below it (claude-code at 98M, hermes at 14.5M, ide-assistant-A at 25.8K). That is its own concentration finding orthogonal to Fano and CV: the heaviest source by daily mean is also the steadiest by CV (opencode), and the source with the most active days is also the lightest by daily mean (ide-assistant-A). Big-and-steady vs small-and-spiky is a real dichotomy in this dataset.

## What the next index after Fano should look like

If I were to write a `source-overdispersion-rank` subcommand, I would want it to:

1. Report Fano (variance/mean) — absolute-tokens dispersion.
2. Report CV (stddev/mean) — relative dispersion.
3. Report a **Fano-rank delta** — `rank_by_fano(s) - rank_by_cv(s)` — to surface sources where the two disagree (opencode at +5, ide-assistant-A at –4 in this corpus).
4. Optionally report a **negative-binomial dispersion `k`** — `μ² / (σ² - μ)` — which is the natural overdispersion parameter and degrades to ∞ for Poisson.

The current 6-source output already gives me 1 and 2; rows 3 and 4 are the obvious follow-ups. For the moment, the takeaway is that I now have a quantitative way to say "claude-code is bursty in tokens, ide-assistant-A is bursty in shape, opencode is neither, and any single-index dashboard that doesn't show both Fano and CV is hiding one of the two stories from the operator."

## Citation footer

- `pew-insights source-burstiness-fano-factor --json` (v0.5.6), generated `2026-04-26T18:23:00.999Z`, totalTokens = 9,353,043,468 across 6 sources from `~/.config/pew/queue.jsonl`.
- ai-native-notes git head at the time of writing: `dca53bb`.
- Specific fano values cited: claude-code 444,573,212 (CV 2.126); opencode 100,402,822 (CV 0.469); ide-assistant-A 83,915 (CV 1.802).
- Source name `ide-assistant-A` is the standing redaction for the IDE-side telemetry source per repo policy; underlying CLI label is otherwise unchanged.
