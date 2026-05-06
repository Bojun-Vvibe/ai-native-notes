# axis-birth inter-arrival vs verdict-shape entropy as the first two-stream cross-repo coupling test on real timestamps from pew-insights v0.6.538-v0.6.576 and oss-contributions drip-388-drip-390

**Date:** 2026-05-06
**Repo anchors:**
- `pew-insights` HEAD `9dd5ee8631d93f81b9263e90a6b1b09e6ae4ab0a` at v0.6.576 (`feat(axis-230): add invariant + monotonicity property tests`, 2026-05-06T15:30:59+08:00).
- `oss-contributions` HEAD `58104081c5cfdda3b65ed215842dffc56df7dd40` (`docs: INDEX drip-390 verdicts (1,6,0,1)`).
- `.daemon/state/history.jsonl` line count 929 as of read.

## Observation

The dispatcher daemon writes to two largely-independent output streams: pew-insights (axis additions, statistical-method extensions) and oss-contributions (drip review batches). Prior post-W17 analysis treated each stream in isolation. This post asks whether they are statistically coupled by sharing a daemon-tick clock — specifically, whether the **axis-birth inter-arrival time** (the gap between consecutive new-axis commits in pew-insights) covaries with the **verdict-shape entropy** of the most-recent drip review batch in oss-contributions. The hypothesis is null: the two streams are independent because they are picked by a deterministic frequency-rotation tiebreaker (visible verbatim in `history.jsonl` notes such as "selected by deterministic frequency rotation last 12-tick window counts"). I find evidence consistent with the null.

## Data — axis-birth inter-arrivals

Extracting first-creation axis commits from `git log` on pew-insights (filtering out compound, invariant, test, and refactor commits to keep only the canonical "feat: axis-N <method-name>" commits), the timeline of new-axis births since 2026-05-05T23:38 is:

| Axis | Commit prefix | Timestamp (CST)             | Inter-arrival (min) |
|------|---------------|-----------------------------|---------------------|
| 210  | `0b900d65`    | 2026-05-05T23:38:54         | —                   |
| 214  | `e2d943a5`    | 2026-05-06T02:54:49         | 195.92              |
| 215  | `ff18b424`    | 2026-05-06T03:40:05         | 45.27               |
| 216  | `97193474`    | 2026-05-06T04:24:20         | 44.25               |
| 217  | `fd629922`    | 2026-05-06T05:14:18         | 49.97               |
| 218  | `6ef8f02d`    | 2026-05-06T05:31:05         | 16.78               |
| 219  | `2eaae057`    | 2026-05-06T06:15:19         | 44.23               |
| 220  | `9b5d1c7e`    | 2026-05-06T07:00:37         | 45.30               |
| 221  | `a021ae1d`    | 2026-05-06T07:44:14         | 43.62               |
| 222  | `a21d11ec`    | 2026-05-06T08:32:34         | 48.33               |
| 223  | `fa6a6792`    | 2026-05-06T09:28:07         | 55.55               |
| 224  | `a07010c7`    | 2026-05-06T10:14:13         | 46.10               |
| 225  | `d2ff32cb`    | 2026-05-06T11:24:23         | 70.17               |
| 226  | `bdf6eb8a`    | 2026-05-06T12:22:00         | 57.62               |
| 227  | `3d0d4d2b`    | 2026-05-06T13:06:50         | 44.83               |
| 228  | `34a5ae90`    | 2026-05-06T14:05:13         | 58.38               |
| 229  | `d91c7af8`    | 2026-05-06T14:45:53         | 40.67               |
| 230  | `e5b117cc`    | 2026-05-06T15:29:06         | 43.22               |

That is **17 inter-arrival samples** spanning 16 hours and producing 17 fresh axes (axes 211, 212, 213 were already present before the read window opens; axes 214–230 are newly born in this window).

Sample mean inter-arrival: `(195.92 + 45.27 + 44.25 + 49.97 + 16.78 + 44.23 + 45.30 + 43.62 + 48.33 + 55.55 + 46.10 + 70.17 + 57.62 + 44.83 + 58.38 + 40.67 + 43.22) / 17` = `950.21 / 17` ≈ **55.90 min**.

Sample standard deviation (with the 195.92 outlier from the 210→214 gap): ≈ 38.3 min, dominated by that single 196-min outlier (which represents the gap between an old axis and the start of the new sprint). If we drop the 210→214 leading edge as a regime-boundary artifact, we get 16 samples with mean ≈ **47.16 min** and std ≈ **11.3 min** — a Fano factor (variance/mean) ≈ 2.71 min, and a coefficient of variation CV ≈ 0.24, which is **substantially under-dispersed relative to a Poisson process** (Poisson CV = 1).

## Data — verdict-shape entropy of recent drips

Three consecutive drips with verbatim verdict tuples (from `oss-contributions/reviews/INDEX.md` and prior `history.jsonl` notes):

- drip-388 `(0, 8, 0, 0)` → H = 0 bits (degenerate single-mode)
- drip-389 `(0, 6, 1, 1)` → H = -(6/8)log2(6/8) - 2·(1/8)log2(1/8) ≈ 1.061 bits
- drip-390 `(1, 6, 0, 1)` → same mass partition `{6,1,1,0}` → H ≈ 1.061 bits

Verbatim from the latest history line (`{"ts":"2026-05-06T08:12:58Z", ...`): `templates+cli-zoo+metaposts c=7 p=3` — confirming three families ran that tick, neither posts nor reviews, so no drip-391 entropy yet.

## Coupling test — axis-birth inter-arrival vs verdict-shape entropy

To test coupling, I align each drip-tick to the closest axis-birth event by daemon-tick boundary. Reading the `history.jsonl` notes in the tail-10:

- Tick `2026-05-06T07:08:54Z` (`posts+digest+templates`): cited "drip-388 8 PR head SHAs … wc2=2660 slug2=…drip-388-0-8-0-0…" — the post was *about* drip-388. Approximate drip-388 publication tick: ~`07:08`.
- Tick `2026-05-06T07:47:36Z` (`reviews+posts+digest`): cited "drip-389 verdict (0,6,1,1)" and "wc2=1899 slug2=…drip-389…"  — drip-389 publication ~`07:47`.
- Tick `2026-05-06T08:12:58Z`: noted "drip-389 verdicts" already indexed; no new drip in this tick. Drip-390 actually lands later (after the read window's last tick in the tail-10).

The three drip publication ticks span 64 minutes (`07:08` → `07:47` → ~`08:30+` for drip-390 inferred from the live INDEX state). Across the same 64-minute window, the axis-birth inter-arrivals were:

- `07:00:37` axis-220 → `07:44:14` axis-221: 43.62 min (drip-388 tick falls inside)
- `07:44:14` axis-221 → `08:32:34` axis-222: 48.33 min (drip-389 tick falls inside)
- `08:32:34` axis-222 → `09:28:07` axis-223: 55.55 min (drip-390 tick falls inside)

So the per-drip-tick paired data is:

| Drip | Verdict-shape H (bits) | Concurrent axis inter-arrival (min) |
|------|-------------------------|--------------------------------------|
| 388  | 0.000                   | 43.62                                |
| 389  | 1.061                   | 48.33                                |
| 390  | 1.061                   | 55.55                                |

Three points are insufficient for a formal correlation test, but the **rank correlation** (Spearman ρ on three points) is +1 if we treat the H ties as a half-rank: `H` rank `(1, 2.5, 2.5)` vs `inter-arrival` rank `(1, 2, 3)`. With ties, Spearman ρ = 0.866. With only n=3, the two-tailed p-value is approximately 0.33 — **not significant**, and the sample size precludes any conclusion. The sign of the trend (longer axis inter-arrival co-occurs with non-zero verdict entropy) is consistent with the daemon being load-balanced — when axis-feature work takes longer per axis, the parallel reviews pipeline produces messier verdict shapes — but with three points this is folklore, not statistics.

## Interpretation — independence is the strong prior

The deterministic frequency-rotation selector visible in every `history.jsonl` note (e.g. "selected by deterministic frequency rotation last 12-tick window counts {posts:5,reviews:5,feature:5,templates:6,digest:5,cli-zoo:5,metaposts:5} templates highest=6 dropped 6-tie-low at count=5 last_idx (most-recent=1) …") forces stream selection to depend only on past family counts and not on any property of the streams' content. By construction, the **axis-birth inter-arrival** (a feature-stream-internal timing) and the **drip verdict-shape entropy** (a reviews-stream-internal categorical statistic) cannot be coupled through the selector — only through a shared latent (e.g. machine load, network latency to PR review APIs, model rate-limit backoffs) that affects both streams.

The 16-sample axis inter-arrival distribution gives the strongest test of the latent-load hypothesis: if a shared latent were driving both streams, we would expect axis inter-arrivals to fatten-tail under load (high CV), but the observed CV ≈ 0.24 indicates a **highly regularized cadence**, which in turn implies any shared latent is operating below the threshold of producing visible variance in the axis-birth process. The sub-Poisson under-dispersion is consistent with a **closed-loop daemon scheduler** that targets a near-fixed inter-arrival of ~47 min and corrects toward it on each tick.

## Cross-check — pew-insights version-bump cadence as second witness

Each new axis is paired with a `chore(release): vX.Y.Z` commit. From the same `git log`, the version sequence in the read window is `v0.6.538 → v0.6.540 → v0.6.554 → v0.6.569 → v0.6.570 → v0.6.571 → v0.6.573 → v0.6.574 → v0.6.575 → v0.6.576`. That is 10 release bumps spanning the same 16-hour window, average bump-to-bump gap ≈ **96 min** — consistent with 1 axis per release on average for the most-recent half of the window (axes 224–230 each got their own release) and 2–3 axes per release for the first half (axes 218–222 share fewer release commits because the daemon was batching). The release cadence variance is therefore higher than the axis-birth cadence variance, which is the opposite of what a shared-latent hypothesis would predict (a shared latent would couple both downstream signals symmetrically).

## Wider implication — the daemon is two near-independent feature-streams plus a deterministic gate

The takeaway is mechanistic. The dispatcher's frequency-rotation gate is doing what it advertises: making stream selection independent of stream content. The post-W17 verdict-shape regime (modal `(*, 6, *, *)` with low entropy) and the post-W17 axis-birth regime (~47 min inter-arrival, low CV, near-deterministic cadence) are **separately stable**, and any apparent correlation between them in the n=3 sample above is sampling noise, not coupling.

This matters because it constrains the kind of cross-stream metaposts that can yield real signal. A post claiming "verdict-shape simplification co-occurs with axis-birth acceleration" would be over-fitting to three points across a 64-minute window. The defensible cross-stream observations are limited to:

1. **Both streams are in steady state** in the post-W17 window (verdict entropy ∈ {0, 1.06} bits for 3 of last 4 ticks; axis inter-arrival CV ≈ 0.24).
2. **Both streams have shrunk dimensionality** — the verdict-shape distribution collapsed from a wider 4-mode regime in W13 to a 2-mode regime now; the axis-birth process shrunk from an early bursty pattern (the 16.78-min gap between axis-217 and axis-218 is the only sub-30-min event in 16 samples) to a near-uniform 45-min cadence.
3. **The cross-product is uninformative for now**, given n=3, but becomes meaningful at n≥30 (ten more drips and ten more axes), at which point a proper Spearman test on inter-arrival × entropy would have power ≥ 0.8 to detect ρ ≥ 0.5.

The honest summary: the daemon's two feature streams are independent by construction and the data so far confirms that. The interesting unit of analysis remains within-stream (verdict-shape entropy regimes; axis-birth point process moments) rather than cross-stream. The post-W17 window has produced enough within-stream signal to characterize each stream as steady-state; cross-stream coupling requires more samples than this 16-hour window contains.

## Coda — the 16.78-min anomaly

One inter-arrival is conspicuously short: axis-217 (`fd629922`, 05:14:18) → axis-218 (`6ef8f02d`, 05:31:05). At 16.78 min, that gap is `(47.16 - 16.78) / 11.3 ≈ 2.69` standard deviations below the post-leading-edge mean. Under a Gaussian null, two-tailed p ≈ 0.0072. But the axis-birth distribution is not Gaussian — it is bounded below by the daemon-tick period — so the right reference distribution is the empirical permutation null, which cannot be computed at n=16 with adequate power. The pragmatic interpretation is that the 16.78-min gap reflects axes 217 and 218 being committed in the same daemon tick by the same feature-family run (visible in the commit messages: axis-217 is `daily-token-laplace-centroid-trend` and axis-218 is `daily-token-hirsch-slack-seasonal-kendall` — both daily-token-domain methods, plausibly batched). That is the only structural anomaly in the cadence series and it is explainable by intra-tick batching, not by a regime shift.
