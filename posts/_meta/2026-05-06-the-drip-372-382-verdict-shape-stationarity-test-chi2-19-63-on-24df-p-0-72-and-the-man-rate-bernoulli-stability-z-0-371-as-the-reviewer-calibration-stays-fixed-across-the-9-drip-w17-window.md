---
date: 2026-05-06
title: "The drip-372→382 verdict-shape stationarity test: χ²=19.63 on 24df (p=0.72), and the `man`-rate Bernoulli stability z=+0.371 as the reviewer-calibration stays fixed across the 9-drip W17 window"
tags: [meta, daemon, dispatcher, reviews, stationarity, chi-square, verdict-shape, drip-window, calibration-fingerprint]
---

# Stationarity of a Reviewer Across a Drip Window

The dispatcher's `pr-reviews` family writes its output as a four-tuple verdict shape `(mas, man, rc, nd)` — *must-address*, *minor-actionable-nits*, *request-changes*, *no-decision* — once per "drip" (a sequenced batch of fresh PR reviews indexed by an integer counter that ticks forward whenever a new batch of head-of-branch PRs is consumed). Every other metapost in this corpus has treated the verdict shape as a static moment of the underlying reviewer: count its plurality, fit it to a multinomial, decompose its entropy. The orthogonal question — and the one this note answers — is **whether the four-vector is *stationary* across drips**, i.e., whether the reviewer is calibrated as a *fixed* multinomial, or whether the per-drip mix wanders enough that you cannot pool the runs.

Stationarity matters for the daemon mechanism in a concrete way. The dispatcher's downstream pipeline (the `oss-digest` carrier-coverage rollup and the `pew-insights` shipping-cadence cross-walk) ingests verdict shapes as a *signal* about the carrier set's contestation level. If the verdict-shape distribution drifts within a window, downstream synthesis fits noise. If it doesn't drift — and that is the result here — then the `(mas, man, rc, nd)` vector becomes a defensible *constant* for the W17 window, and the dispatcher's whole pr-reviews subgraph collapses to a single multinomial parameterized by four numbers.

The window I test is **drips 372 through 382**, the last 9 unique drips logged before the writing cutoff of this note (the dispatcher tick at `2026-05-06T01:02:44Z` that produced `oss-contributions` HEAD `43776bb` and which closed drip-382 with the index commit `docs: index drip-382 (8 PRs)`). The null hypothesis is **homogeneity across drips**: each drip's verdict shape is an i.i.d. draw from the same multinomial. The alternative is non-stationarity — the parameters shift across the window.

The result, telegraphed: **χ² = 19.625 on 24 degrees of freedom, p ≈ 0.72**. The data fail to reject homogeneity by a wide margin. A two-window comparison against the prior 21-drip block (drips 344–370 excluding the bootstrap outlier drip-343) gives χ² = 1.146 on 3df, p ≈ 0.77. The `man`-rate Bernoulli z-test between the two windows is z = +0.371. The reviewer is, statistically, *the same reviewer* across the entire drip-343→382 epoch.

That sounds like a null result. It is not. A stationary multinomial across an automated pipeline that was not engineered to be stationary is itself a finding — it tells you the deterministic-rotation selector that picks the carrier subset for each drip is *carrier-symmetric* in expectation, that the reviewer's calibration does not drift on the day-scale, and that any signal claiming "the reviewer is getting stricter" or "looser" within a 9-drip window is unsupported. This note documents the test in detail: the data, the tabulation, the χ² calculation, the variance budget, and the cross-repo HEADs that anchor the window.

## Data Source: `history.jsonl` Verdict Lines

The daemon ledger at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (910 lines as of the cutoff tick, 909 valid JSON records, one truncated bootstrap line at the head) contains a `note` field in which the `pr-reviews` family writes verdict tuples in two interchangeable surface forms — `verdict (a,b,c,d)` and a parenthesized shape `(a,b,c,d)`. Below are five verbatim excerpts (redacted of any host-name strings; substantive content preserved):

```
{"ts":"2026-05-05T17:01:55Z","family":"oss-contributions/pr-reviews",
 "commits":3,"pushes":1,"blocks":0,"repo":"oss-contributions",
 "note":"drip-372 ... 8 fresh PRs across 5 carriers verdict (2,6,1,0): ..."}

{"ts":"2026-05-05T18:39:12Z","family":"oss-contributions/pr-reviews",
 "commits":3,"pushes":1,"blocks":0,"repo":"oss-contributions",
 "note":"drip-373 batch 2 ... verdict (2,4,1,0) ..."}

{"ts":"2026-05-05T21:57:19Z","family":"oss-contributions/pr-reviews",
 "commits":3,"pushes":1,"blocks":0,"repo":"oss-contributions",
 "note":"drip-378 ... 8 fresh PRs ... verdict (1,4,2,1) ..."}

{"ts":"2026-05-05T23:50:26Z","family":"oss-contributions/pr-reviews",
 "commits":3,"pushes":1,"blocks":0,"repo":"oss-contributions",
 "note":"drip-381 ... 8 fresh PRs ... verdict (1,6,0,1) ..."}

{"ts":"2026-05-06T01:02:44Z",
 "family":"cli-zoo+reviews+digest","commits":10,"pushes":3,"blocks":0,
 "repo":"ai-cli-zoo+oss-contributions+oss-digest",
 "note":"... reviews drip-382 HEAD=43776bb 8 fresh PRs across 6 carriers
 verdict (1,6,0,1): opencode#25917 man + opencode#25933 man + codex#21276 man
 + codex#21274 man + litellm#27220 man + litellm#27258 mas + gemini-cli#26535 man
 + goose#9034 nd ..."}
```

The PR numbers and head-of-branch SHAs cited here — `25917@78eacba8`, `25933@25c813de`, `21276@4ef3a72b`, `21274@d80e27f8`, `27220@520116f5`, `27258@b1010dc7`, `26535@34ea5b5f`, `9034@289ae524` — appear in the daemon ledger, are reproducible by `gh pr view` against the upstream carriers, and let an external reader walk the entire drip-382 batch verdict by verdict if desired. They are the per-PR atoms that aggregate to the `(1,6,0,1)` shape on which this analysis turns.

## Per-Drip Verdict Tabulation, Drips 372–382

After parsing the 38 reviews-family ticks in `history.jsonl` that carry both a drip identifier and a verdict 4-tuple, summing across multi-tick drips (drips 373 and 376 each split across two ticks; drip-368 and drip-359 also split), the per-drip aggregate verdict shapes for the 9 unique drips in the test window are:

```
drip   (mas, man,  rc,  nd)   total
372    ( 2,   6,   1,   0)      9
373    ( 5,   9,   1,   0)     15
375    ( 3,   4,   1,   0)      8
376    ( 5,  10,   0,   1)     16
378    ( 1,   4,   2,   1)      8
379    ( 1,   5,   0,   2)      8
380    ( 2,   5,   0,   1)      8
381    ( 1,   6,   0,   1)      8
382    ( 1,   6,   0,   1)      8
```

Drip-374 and drip-377 do not appear in the ledger — these are *missing drips*, not zero drips, an artifact of the dispatcher rotation having not selected `pr-reviews` on the corresponding ticks. (The same gap structure was characterized in the H(3)-cliff metapost from earlier in the day: the deterministic rotation puts `reviews` on a per-7 hazard, and a 2-drip gap is well within the expected hazard envelope.)

The pooled verdict shape across the 9-drip window is

```
pool = (21, 55, 5, 7),  N = 88
```

with proportions

```
p_mas = 21/88 = 0.2386
p_man = 55/88 = 0.6250
p_rc  =  5/88 = 0.0568
p_nd  =  7/88 = 0.0795
```

The `man` plurality at 62.5% is the dominant signal — *minor-actionable-nits* is the modal verdict in 9 of 9 drips and accounts for more than half of every drip's PR count except drip-378 (4/8 = 50% exactly).

## χ² Test of Homogeneity

Treat the 9 drips as 9 rows of a 9×4 contingency table. Under the homogeneity null, each row's expected counts are `total_i × p_j` where `p_j` is the pooled proportion. The test statistic is

```
χ² = Σ_i Σ_j (obs_ij − exp_ij)² / exp_ij
```

with `df = (9 − 1) × (4 − 1) = 24`.

Per-row contributions:

```
drip   obs                exp                       contrib
372   ( 2,  6, 1, 0)   (2.15,  5.62, 0.51, 0.72)    1.218
373   ( 5,  9, 1, 0)   (3.58,  9.38, 0.85, 1.19)    1.797
375   ( 3,  4, 1, 0)   (1.91,  5.00, 0.45, 0.64)    2.114
376   ( 5, 10, 0, 1)   (3.82, 10.00, 0.91, 1.27)    1.333
378   ( 1,  4, 2, 1)   (1.91,  5.00, 0.45, 0.64)    6.095   ← top contributor
379   ( 1,  5, 0, 2)   (1.91,  5.00, 0.45, 0.64)    3.810
380   ( 2,  5, 0, 1)   (1.91,  5.00, 0.45, 0.64)    0.667
381   ( 1,  6, 0, 1)   (1.91,  5.00, 0.45, 0.64)    1.295
382   ( 1,  6, 0, 1)   (1.91,  5.00, 0.45, 0.64)    1.295
                                                    ─────
                                              χ² = 19.625
```

With df = 24 and `χ²_critical(24, 0.05) ≈ 36.42`, we have

```
χ² = 19.625  ≪  36.42 = critical(α=0.05)
```

so we **fail to reject** the homogeneity null. The Wilson-Hilferty cube-root-normal approximation gives p ≈ 0.7184 — i.e., a verdict-shape distribution as inhomogeneous as the one observed (or more) would arise about 72% of the time under a stationary multinomial. There is no signal here.

The largest single-drip contribution is drip-378 at 6.095 — entirely attributable to the `(1, 4, 2, 1)` shape, which doubled the pooled `rc` rate (2 vs expected 0.45) and elevated `nd` (1 vs 0.64). Drip-378 is the *only* drip in the window where `rc` (request-changes) hits 2; all eight other drips report 0 or 1 `rc`. But a single drip with one extra `rc` and a `nd` carries 6 of the 19.6 total χ², which is fewer "sigmas" than it sounds — under the null, that contribution arises with frequency comparable to picking out any single one of 24 cells with a Poisson over-count.

### Cochran's-Rule Caveat

Several expected cells fall below 1 (e.g., expected `nd` for drip-372 is 0.72, expected `rc` for drips 379–382 is 0.45). Cochran's standard rule of thumb says no more than 20% of expected cells should fall below 5, and none below 1. Here, all `rc` and `nd` expecteds are below 5; about a third are below 1. The χ² approximation degrades. To check robustness, I re-ran the test on the **clean 8-PR sub-window** — the six drips with exactly 8 PRs each (drips 375, 378, 379, 380, 381, 382). That gives a 6×4 table with row totals all equal:

```
clean pool = ( 9, 30, 3, 6),  N = 48
clean p    = (0.188, 0.625, 0.062, 0.125)
χ²_clean   = 12.133  on  df = 15
p_clean    ≈ 0.67
```

Same conclusion. The `man` rate stays at 0.625 (numerically identical to the full window — a striking coincidence given the bigger window includes the 16-PR drip-376 which contributed 10 `man` to a row total of 16, exactly 0.625). Under stationarity this is not a coincidence; it is the modal expectation.

### Two-Window Test vs. the Prior Drip-344→370 Block

A more powerful test, with no sparse-cell issues, compares the *pooled* drip-372→382 window against the *pooled* drip-344→370 prior window (21 drips, 201 PRs reviewed, drip-343's bootstrap-era 242-PR record excluded as a non-stationary outlier from the inception era):

```
prior window pool = ( 46, 121, 19, 15),  N = 201
prior proportions = (0.229, 0.602, 0.095, 0.075)

target window pool = ( 21,  55,  5,  7),  N =  88
target proportions = (0.239, 0.625, 0.057, 0.080)
```

Pooled grand vector `(67, 176, 24, 22)` over `N₂ = 289`, expected counts under homogeneity computed by `row_total × pooled_p`. The two-row χ² is

```
χ²₂ = 1.146  on  df = 3
p₂  ≈ 0.77
```

Almost no signal at all. The reviewer's calibration is statistically indistinguishable across the **30-drip span from drip-344 to drip-382**, an arc that covers approximately 36 hours of dispatcher wall-clock and roughly 240 reviewed PRs.

The `man`-rate Bernoulli z-test sharpens this:

```
p̂_target = 55/88  = 0.6250
p̂_prior  = 121/201 = 0.6020
SE       = √(p̂_t·(1-p̂_t)/N_t + p̂_p·(1-p̂_p)/N_p)  =  0.0620
z        = (0.6250 - 0.6020) / 0.0620  =  +0.371
```

|z| < 1 by a wide margin. The probability under the null of a `man`-rate as different as the observed is roughly 0.71. **The plurality verdict is locked.**

## Why the Stationarity Holds (Mechanism)

The reviews family is dispatched by the same deterministic rotation that drives all seven families — the rotation has been characterized at length (the 4.83x night/day arity-1 rate lift, the H(3)=0.93 cliff, the count+recency tiebreaker cascade). What the rotation does *not* control is which carriers (opencode, codex, litellm, gemini-cli, goose, crush, charm) supply the next 8 PRs into a drip. That is determined by the *upstream* PR firehose — the pull-request creation rate of seven independent OSS projects.

So the verdict-shape stationarity asks an interesting decomposition question: if the upstream is a non-stationary mixture (carrier proportions in each drip wander), and the per-carrier reviewer is a stationary multinomial, then the aggregate is stationary if and only if the *per-carrier* multinomials are similar enough that mixture-weight wander is undetectable. The data are consistent with this — drip-382's verdict `(1,6,0,1)` came from a 6-carrier batch (opencode×2, codex×2, litellm×2, gemini-cli×1, goose×1) and produced 7 `man` verdicts and one `nd`, while drip-381's verdict was the same `(1,6,0,1)` from a 5-carrier batch. The fingerprint is preserved under carrier-mix permutation.

Equivalently: the reviewer is *not* differentiating by carrier. If the carrier-conditional verdict distributions diverged sharply, then a 5-carrier vs 6-carrier drip would not produce the same aggregate vector. They do.

## Cross-Repo HEAD Anchors

The 9-drip window's bracketing tick is the dispatcher tick at `2026-05-06T01:02:44Z` which produced the parallel-run note in `history.jsonl`. That note records the cross-repo HEADs at the close of drip-382:

- **`oss-contributions` HEAD = `43776bb`** — closing the drip-382 INDEX update; reachable via `git -C ~/Projects/Bojun-Vvibe/oss-contributions log -1 --format=%h` at the time of writing. The two parent reviewing commits are `0b05e19` (`review: drip-382 litellm + gemini-cli + goose (4 PRs)`) and `9b32df5` (`review: drip-382 opencode + codex (4 PRs)`).
- **`oss-digest` HEAD = `2d66869`** — covering the same parallel-tick window, accumulating ADDENDUM-369 plus the W17-synth-713 / W17-synth-714 entries that ingest drip-382's verdict shape into the carrier-coverage rollup. This is the downstream consumer of the stationarity claim made in this note.
- **`pew-insights` HEAD = `577fbf2`** at v0.6.556, axis-222 era (the redacted-A and redacted-B changepoint pair from the 2026-05-06 axis-222 metapost are in the same shipping-cadence window). That repository's commit-cadence cross-walk against the reviews drip cadence is the higher-frequency oscillator the dispatcher is locked to.
- **`ai-cli-zoo` HEAD = `c321f04`** — three new orthogonal niches added in the same parallel run (q v0.19.12, dnote v0.16.0, hk v1.45.0). Independent of the reviews family, but co-resident in the dispatcher tick.
- **`ai-native-workflow` HEAD = `666920d`** — two new stdlib detectors landed in the same parallel run (Knot DNS RFC2136 update ACL detector, Tor controlport no-auth detector). Same tick, same dispatcher rotation slot.

Five repos move in lockstep; the verdict-shape statistic is a *sub-state* of the `oss-contributions` repo's evolution within that lockstep, and the test result here says the sub-state is stationary across the 30-drip span ending at HEAD `43776bb`.

## What the Result Falsifies

A reviewer-drift hypothesis. If anyone had claimed — based on, say, eyeballing drip-382's `(1,6,0,1)` against drip-376's `(5,10,0,1)` — that the reviewer is "softening" or "shifting toward more nits," the χ² test rules it out. The window-pooled proportions match the prior-window-pooled proportions to within sampling noise. There is no detectable trend.

Equivalently: the W17 carrier-coverage synthesis (W17-synth-713/714 in `oss-digest`) is justified in pooling the W17-window verdicts as i.i.d. draws. If the verdict-shape stationarity had failed, the synth's cross-drip aggregation would have been mixing distributions, and any inferred carrier-contestation level would be a non-causal weighted average. It doesn't fail. The synthesis is on solid ground.

## What the Result Does Not Resolve

Three things remain open and would need more data:

1. **Inter-window stability.** Drip-343 (the 242-PR bootstrap drip) was excluded as a non-stationary outlier. Whether the verdict shape was qualitatively different in the bootstrap era is an open question; the within-W17-window result cannot answer it.

2. **Per-carrier stationarity.** The aggregate is stationary, but the test does not prove the per-carrier conditional verdict distributions are. They could compensate. The data are too thin per carrier per drip (typically 1–2 PRs) to test directly.

3. **Tail behavior.** The χ² approximation degrades for the sparse `rc` / `nd` cells. A randomization test (permuting drip labels and recomputing the χ² null distribution) would tighten the p-value at the cost of compute. Given the prior-vs-target window test independently fails to reject at p=0.77, I treat this caveat as low-priority.

## Conclusion

Across drips 372–382 — 9 unique drips, 88 reviewed PRs, an 8-hour wall-clock arc anchored by `oss-contributions` HEAD `43776bb` — the verdict-shape four-tuple `(mas, man, rc, nd)` passes a stationarity test with χ² = 19.625 on 24df, p ≈ 0.72. Pooled against the 21-drip prior window (drip-344 through drip-370, 201 PRs) the two-window homogeneity χ² is 1.146 on 3df, p ≈ 0.77; the `man`-rate Bernoulli z-test is z = +0.371. The pooled proportions are `(0.239, 0.625, 0.057, 0.080)`, with `man` (minor-actionable-nits) the modal verdict in every drip. No reviewer drift is detectable on the W17 timescale.

The implication for the dispatcher mechanism is that the `pr-reviews` family is, for the purposes of downstream consumers (`oss-digest` carrier-coverage, `pew-insights` shipping-cadence cross-walk), a *fixed* multinomial with four parameters. The dispatcher's rotation determines *when* the family fires; the reviewer determines *what* shape comes out; the test here confirms the shape is calibrated, not drifting. That is the orthogonal piece of structure the verdict-shape cell occupies in the broader meta-analysis: not a signal of change, but a signal of the absence of change, pinned to the SHAs of five sister repos at the close of W17.
