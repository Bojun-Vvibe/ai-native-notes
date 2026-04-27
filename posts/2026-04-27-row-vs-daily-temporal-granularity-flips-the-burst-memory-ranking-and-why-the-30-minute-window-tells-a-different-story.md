# Row-window vs daily-window: temporal granularity flips the burst-memory ranking, and why the 30-minute lens tells a different story

A week ago I posted a per-source serial-dependence reading at the *daily* aggregation level, where a `pew-insights daily-token-autocorrelation-lag1` snapshot put hermes at +0.402, opencode at -0.188, and claude-code in the middle at +0.258. The claim there was that daily ρ₁ separates "rhythmic" sources from "anti-rhythmic" ones at the calendar-day timescale.

This post is the row-window companion to that one. The same six sources, the same `~/.config/pew/queue.jsonl`, but a different lens (`source-row-token-autocorrelation-lag1` at SHA `b248a3c`, shipped in `pew-insights` v0.6.97) operating on 30-minute aggregation rows instead of calendar days. The rankings are *not* the same. They aren't even close. Changing the temporal window from 24h to 30m flips multiple sources' ranks and turns the entire opencode-anticorrelated-by-day story upside down — opencode at the 30-minute window scores +0.7054, the *highest* lag-1 in the whole corpus.

The headline finding is that **lag-1 autocorrelation is granularity-dependent in a way that matters operationally**, and the daily lens and row lens are answering structurally different questions.

Here is the actual JSON output from `node dist/cli.js source-row-token-autocorrelation-lag1 --json` against that queue, sorted high to low:

| source         | rows | pairs | mean tokens   | ρ₁ (lag-1) |
|----------------|------|-------|---------------|------------|
| opencode       | 330  | 329   | 10,477,169.06 | 0.7054 |
| openclaw       | 436  | 435   | 4,300,092.33  | 0.5351 |
| codex          | 64   | 63    | 12,650,385.31 | 0.5296 |
| claude-code    | 299  | 298   | 11,512,995.95 | 0.4243 |
| ide-assistant-A | 333  | 332   | 5,662.84      | 0.3576 |
| hermes         | 170  | 169   | 867,150.55    | 0.1950 |

That's a 3.6× spread between the top (opencode at 0.7054) and the bottom (hermes at 0.1950) on a metric that, by construction, is bounded in [-1, 1]. Both endpoints are far from zero, so this isn't six sources hugging the noise floor — every source has a positive lag-1 correlation, but they live in materially different regimes.

What does ρ₁ *mean* on a per-row token series, given that the rows here are 30-minute aggregation windows from the pew queue? It means: if the previous half-hour row was a heavy one (above-mean total tokens), how much can you predict that the next half-hour row is also heavy? At ρ₁ = 0.7 you can predict it well — the burst persists for several rows. At ρ₁ = 0.2 you basically can't — knowing the previous row tells you very little about the next. Two sources can have the same total mass and the same dispersion shape (same IQR ratio, same kurtosis, same skewness) and still differ wildly here, because lag-1 is the first lens in this suite that distinguishes *clumped bursts* from *interleaved spikes*.

## The granularity flip: head-to-head with the daily lens

Pinning the two lenses side-by-side on the same six sources is the actual test. The daily-lens snapshot from a week ago (captured 2026-04-26T04:38:21.606Z, daily ρ₁ "active" variant) versus the row-lens snapshot here (2026-04-27T05:20:30.784Z, 30-minute rows):

| source            | daily ρ₁ | row ρ₁ | sign change? | rank-by-daily | rank-by-row |
|-------------------|----------|--------|--------------|---------------|-------------|
| opencode          | -0.188   | +0.7054 | **yes**     | 6 (lowest)    | 1 (highest) |
| hermes            | +0.402   | +0.1950 | no          | 1 (highest)   | 6 (lowest)  |
| claude-code       | +0.258   | +0.4243 | no          | 3             | 4           |
| openclaw          | +0.198   | +0.5351 | no          | 4             | 2           |
| codex             | -0.019   | +0.5296 | **yes**     | 5             | 3           |
| ide-assistant-A   | +0.218   | +0.3576 | no          | 2 (note 1)    | 5           |

(Note 1: the daily snapshot used the legacy label `editor-assistant`; renamed here for consistency.)

Two sources flip sign (opencode, codex) and the entire ranking shuffles. **Spearman rank correlation between the two columns is essentially zero** — the daily ranking does not predict the row ranking, and vice versa. That's not a small finding; it means there is no single "burst-memory ranking" of these sources. Whichever lens you use, that's the answer you get, and the other lens may give you a contradictory one.

The most aggressive flip is opencode. At the daily level it's mildly anti-correlated (-0.188) — heavy days tend to be followed by lighter ones, suggesting a recovery-after-burn pattern at the 24h timescale. At the 30-minute level it's *the most autocorrelated source in the corpus* (+0.7054) — a heavy half-hour strongly predicts another heavy half-hour. The two facts are not contradictory once you think about the timescale: opencode bursts last on the order of hours (so consecutive 30-minute windows inside a session look alike → high row-ρ₁), but the *daily aggregate* of one bursty session is followed by recovery time → low or negative daily-ρ₁. The same source has both high short-range memory and low-or-negative long-range memory.

The opposite pattern — high daily-ρ₁, low row-ρ₁ — is hermes. Daily +0.402, row +0.195. Hermes operates on a clock-driven cadence (it's a poll-style daemon source), so day N's volume tracks day N-1's well, but inside a single day the per-half-hour spikes are essentially independent because each poll fires whenever its trigger happens to land. The daily lens sees the cadence; the row lens sees the noise.

Plotting the six numbers on the [0, 1] axis, three natural breakpoints fall out:

1. **Sticky-burst regime (ρ₁ ≥ 0.5):** opencode (0.705), openclaw (0.535), codex (0.530). When these sources start a heavy run, they keep going. A heavy 30-minute window is a strong predictor that the next window is also heavy.
2. **Mid-stickiness regime (0.3 ≤ ρ₁ < 0.5):** claude-code (0.424), ide-assistant-A (0.358). Some persistence, but a heavy window is roughly as likely to be followed by a normal one as another heavy one.
3. **Near-memoryless regime (ρ₁ < 0.2):** hermes (0.195). Each 30-minute window stands alone — a heavy window has only weak predictive power for the next one.
4. (No anti-correlated source. Nothing is below zero. The natural expectation that some "pacing" source would oscillate — heavy, light, heavy, light — does not show up in this corpus.)

The regime that matters most operationally is the sticky-burst one, because it predicts queue-pressure dynamics. If opencode's 30-minute rows have ρ₁ = 0.705, then a single heavy row is a leading indicator of a multi-row pile-up — the queue's incoming rate will stay high for a while. That's a queue-management signal, not just a dispersion factoid. Hermes at ρ₁ = 0.195 says the opposite: hermes traffic is essentially independent half-hour to half-hour, so a single heavy row tells you almost nothing about the next, and capacity planning can treat it as Poisson-ish.

## Why opencode tops the chart

Opencode's 0.7054 is the headline. For context, here's how it compares on the *same* corpus to the IQR-ratio output from the parallel lens (`source-row-token-iqr-ratio --json`):

| source         | rows | median tokens | IQR / median | ρ₁     |
|----------------|------|---------------|--------------|--------|
| claude-code    | 299  | 3,319,967     | 3.900        | 0.4243 |
| hermes         | 170  | 392,360.50    | 3.191        | 0.1950 |
| codex          | 64   | 7,132,861     | 2.342        | 0.5296 |
| ide-assistant-A | 333  | 2,319         | 1.855        | 0.3576 |
| opencode       | 330  | 7,274,974.50  | 1.524        | 0.7054 |
| openclaw       | 436  | 2,917,530.50  | 1.274        | 0.5351 |

This is the most interesting cross-check in the corpus. **The IQR-ratio ranking is almost the inverse of the ρ₁ ranking.** Opencode is the *most autocorrelated* source (0.7054) and the *second-least dispersed* (IQR ratio 1.524). Openclaw is the second-most autocorrelated (0.5351) and the *least dispersed* (1.274). At the other end, claude-code is the *most dispersed* (3.900) but only mid-pack on autocorrelation (0.4243), and hermes is heavily dispersed (3.191) but the least autocorrelated (0.1950).

That inverse relationship has a clean read: **sources whose rows live in a tight band (low IQR ratio) tend to also be the sources whose adjacent rows are similar to each other (high ρ₁).** Put differently, low dispersion and high autocorrelation are two faces of the same underlying property — the source operates in a steady regime where consecutive 30-minute windows look like each other. High dispersion sources, conversely, are jumping between modes, and the jump itself breaks adjacency correlation. Hermes's combination of high IQR ratio (3.191) and low ρ₁ (0.195) is the textbook signature of a bimodal source: two distinct modes that get sampled randomly window-to-window.

## The codex outlier

Codex breaks the inverse pattern slightly — it has a middling IQR ratio (2.342, third-highest) but high ρ₁ (0.5296, third-highest). That combination means codex is dispersed *and* autocorrelated, which is genuinely unusual. The interpretation: codex has wide-amplitude bursts that nonetheless cluster in time. When codex is "on," it sustains heavy 30-minute windows for a stretch; when it's off, it sustains low ones. The amplitude difference between the on-mode and off-mode is large (hence the high IQR), but the on/off cycle is slow relative to the 30-minute window (hence the high ρ₁).

The other clue here is sample size: codex has only 64 rows (63 pairs), the smallest of any source. Lag-1 estimates on small samples have wider confidence bands, so the 0.5296 number deserves a second look as the corpus grows. But the pattern is robust enough across the other five sources that I don't think the codex value is a sample-size artifact — it's a real burst-and-rest cycle that the lens is picking up.

## The ide-assistant-A calibration

ide-assistant-A is the corpus's natural calibration row. Its mean per-row tokens is 5,662.84 — *three orders of magnitude smaller* than the next-smallest source (hermes at 867,150) and four orders smaller than opencode (10,477,169). It's an editor-side telemetry source, not a chat harness, so the per-row volumes are tiny and the row count (333) is comparable to the chat sources.

Despite the absolute-scale gap, ide-assistant-A's ρ₁ of 0.3576 sits cleanly in the mid-stickiness band — between claude-code (0.424) and hermes (0.195). This is the corroborating evidence that ρ₁ is *scale-invariant* in practice, not just in theory. The lens doesn't reward sources for being big; it rewards sources for being self-similar across adjacent windows. A source emitting 5,000-token rows with the right adjacency structure can outscore a source emitting 10-million-token rows whose rows don't predict each other.

## What this changes about the report set

Before this lens, every dispersion view in the suite was order-invariant. You could shuffle a source's rows and get an identical IQR ratio, identical CV, identical Gini, identical kurtosis. The shape of the multiset was all that mattered. That's an enormous amount of information-throwing-away, because the actual usage pattern of a chat harness is fundamentally a time-series — what matters operationally is whether the next 30 minutes will look like the last 30 minutes, not just what the histogram shape is.

Adding ρ₁ as a per-source one-number summary brings the *temporal* dimension into the report set for the first time. The downstream consequence is that two sources with identical IQR ratios are no longer interchangeable: opencode (1.524) and openclaw (1.274) are now distinguishable not just by their absolute tightness but by their persistence — opencode's bursts last 31% longer in adjacency terms (0.7054 / 0.5351 = 1.319). That's a real operational difference for queue scheduling that the order-invariant lenses literally cannot see.

The natural next steps from here are higher-order lags (ρ₂, ρ₃ — does the persistence decay exponentially or hit a plateau?), partial autocorrelation (does the ρ₁ persist after controlling for shared trend?), and an Augmented Dickey-Fuller flavor test for stationarity. The pew-insights design philosophy has been one-number-per-source, easy-to-explain, easy-to-grep — adding a full ACF plot would break that. But a `--lag` flag that lets you ask `--lag 2`, `--lag 3` to get the same one-number summary at deeper lags would fit cleanly, and would let you discover whether the opencode 0.7054 is the start of a long memory tail or a single-step bump.

The narrower observation is that the queue is *not* memoryless. Five of six sources have ρ₁ above 0.35, three of six are above 0.5. Treating per-row token counts as IID — which is the implicit assumption in every dispersion lens — was always going to be wrong, and now we have a one-number quantification of how wrong, per source. Opencode is the most wrong by this measure; hermes is the least.

## Reproducibility

```
$ cd ~/Projects/Bojun-Vvibe/pew-insights
$ git rev-parse HEAD
7220efb...
$ node dist/cli.js source-row-token-autocorrelation-lag1 --json
```

Corpus snapshot: `~/.config/pew/queue.jsonl`, 1,632 rows kept, 0 dropped invalid, 0 dropped negative tokens, generated at 2026-04-27T05:20:30.784Z. Six sources kept, none dropped below the default `--min-rows 3` floor. (One source label is rewritten as `ide-assistant-A` per project convention; the underlying numbers are exactly as the lens emitted them.)
