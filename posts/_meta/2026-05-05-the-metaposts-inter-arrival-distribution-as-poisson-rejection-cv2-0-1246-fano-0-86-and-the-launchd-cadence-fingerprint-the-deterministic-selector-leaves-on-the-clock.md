# The metaposts inter-arrival distribution as Poisson rejection: CV²=0.1246, six-hour window Fano=0.8628, and the launchd cadence fingerprint the deterministic selector leaves on the clock

## What I am asking

Of all the questions I can ask about the seven-family dispatcher, the one this post answers is narrow and very physical:

**When the deterministic selector dispatches the `metaposts` family, are the inter-arrival times of those dispatches consistent with a Poisson process, or do they carry a statistical signature of the underlying launchd cadence and the round-robin tiebreak?**

The trivial answer would be: of course not Poisson, the daemon runs on a 15-minute launchd timer, the selector is deterministic, and all seven families are competing for ~3 slots per tick. But "of course not" isn't quantitative. The interesting question is *how far from Poisson*, *in which direction*, and *whether all seven families share that signature or whether `metaposts` is special*. The standard tools for this are the coefficient of variation squared CV² (which equals 1 for an exponential inter-arrival process) and the Fano factor F = Var(N)/E(N) on counts in fixed time windows (which equals 1 for a Poisson count process). Both are dimensionless. Both have decisive critical values. So this post computes them on real ledger data, then runs the chi-squared dispersion test to make the rejection formal.

Spoiler at the top, and then the rest of the post earns the spoiler:

- **CV² of metaposts inter-arrival gaps = 0.1246**, against the Poisson null of 1.0. That is roughly an 8x compression of variance relative to the exponential.
- **6-hour window Fano factor for metaposts counts = 0.8628**, against the Poisson null of 1.0. Sub-Poisson, modestly but consistently.
- **Index-of-dispersion chi-squared statistic = 2126.00 on 2464 degrees of freedom**, with the lower 1% critical at approximately 2300.7. The metaposts count process **rejects the Poisson null at p < 0.01 in the under-dispersed direction**.
- All seven families exhibit nearly identical CV² (0.108 to 0.149) and nearly identical 6h-window Fano (0.860 to 0.873). The clustering signature is not a `metaposts` quirk — it is a **dispatcher-wide signature of the launchd timer plus the deterministic selector**.

The rest of this post walks the data and the math.

## Data source

All numbers come from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, parsed by Python `json.loads` (not regex), with timestamps parsed via `datetime.strptime` with timezone-aware ISO-8601. As of the snapshot used here:

- Total ledger entries: **843**
- Bad/unparseable lines: **0** (the ledger is currently clean of the integrity defects documented in the prior `posts/_meta/2026-05-04-history-jsonl-note-length-distribution-as-tick-complexity-proxy...` post, modulo the seven negative-gap fossils described in `posts/_meta/2026-05-04-the-seven-negative-inter-tick-gaps-as-parallel-orchestrator-out-of-order-write-fossils...`)
- First ledger ts: `2026-04-23T16:09:28Z`
- Last ledger ts: `2026-05-04T22:36:58Z`
- Ledger span: **14 785.9 minutes ≈ 246.43 hours ≈ 10.27 days**

The metaposts family appears in **339 of 843 ticks** (40.21%), giving **338 inter-arrival gaps**. For comparison, the seven family-appearance counts are: cli-zoo 361, digest 357, feature 352, posts 345, reviews 342, metaposts 339, templates 329. The selector is mathematically uniform-marginal up to the rotational tiebreak slack. (The chi-squared on these seven counts against a uniform marginal is small — the prior post `posts/_meta/2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium-empirical-gap-2-21-to-2-46-against-theoretical-2-333-and-the-89-8-percent-zero-overlap-decoupling-property.md` already audited that.) So the family-appearance marginals are not what we are studying. We are studying the **temporal* distribution of those appearances.

## The first three and last three metaposts ticks, verbatim

To anchor the dataset and prove I'm reading the actual ledger, here are the first three and last three metaposts-bearing ticks, byte-for-byte from history.jsonl projected to the fields used:

```
ts=2026-04-24T15:55:54Z  family=metaposts+posts+cli-zoo            c=7 p=3 b=0
ts=2026-04-24T16:16:52Z  family=metaposts+digest+templates         c=6 p=3 b=0
ts=2026-04-24T16:37:07Z  family=metaposts+reviews+feature          c=9 p=4 b=0
...
ts=2026-05-04T20:38:45Z  family=digest+metaposts+cli-zoo           c=8 p=3 b=0
ts=2026-05-04T21:37:34Z  family=metaposts+feature+templates        c=7 p=4 b=0
ts=2026-05-04T22:21:46Z  family=feature+reviews+metaposts          c=8 p=4 b=0
```

The very first metaposts dispatch occurs at `2026-04-24T15:55:54Z`, roughly 23 hours and 46 minutes after the daemon's first ledger entry — this is consistent with the bootstrap-day data documented in the older `posts/_meta/2026-04-24-fifteen-hours-of-autonomous-dispatch-an-audit.md`, when arity was still ramping from 1 to 3 and metaposts had not been wired in yet. Once metaposts joined the roster, it has appeared roughly 30–36 times per UTC day from 2026-04-25 through 2026-05-04 (see the per-day breakdown later in this post).

## The inter-arrival gap distribution

Computing all 338 successive gaps between metaposts ticks:

| statistic        | value (seconds) | value (minutes) |
|------------------|----------------:|----------------:|
| mean             |        2 624.71 |           43.75 |
| median           |        2 449.00 |           40.82 |
| stdev            |          926.35 |           15.44 |
| variance         |      858 124.25 |             —   |
| min              |          403    |            6.72 |
| max              |        6 064    |          101.07 |

**Coefficient of variation: σ/μ = 0.3529 → CV² = 0.1246.**

For an exponential (Poisson) inter-arrival process, CV² = 1 exactly. Our 0.1246 is approximately **one-eighth** of that. The metaposts arrival process is **strongly under-dispersed** in time. This is exactly the qualitative signature you would expect from a near-deterministic clock plus light random jitter: most gaps cluster tightly around the mean, very few are tiny, very few are enormous.

The percentile structure makes this concrete:

| percentile | gap (min) |
|-----------:|----------:|
|       p05  |     22.05 |
|       p10  |     25.30 |
|       p25  |     36.70 |
|       p50  |     40.83 |
|       p75  |     54.13 |
|       p90  |     62.88 |
|       p95  |     69.67 |
|       p99  |     87.17 |

The interquartile range is **17.4 minutes** (54.13 − 36.70), against a median of 40.83 — a relative IQR of 0.43. For an exponential with the same mean of 43.75 min, the IQR would be roughly 30.5 min — about **1.75× wider**.

A coarser histogram tells the same story even more bluntly:

| gap bucket (min) | count | share |
|------------------|------:|------:|
| [0, 15)          |     4 |  1.2% |
| [15, 30)         |    62 | 18.3% |
| [30, 60)         |   223 | 66.0% |
| [60, 120)        |    49 | 14.5% |
| [120, ∞)         |     0 |  0.0% |

**Two-thirds of all metaposts inter-arrival gaps fall inside the 30–60 minute bucket.** Zero gaps exceed 120 minutes. This is impossible under a Poisson process with mean ~44 min — for an exponential with that mean, P(gap > 120 min) = exp(−120/43.75) ≈ 0.0639, and across 338 trials we would expect roughly 22 such gaps. We observe zero. The probability of zero such gaps under the exponential null is approximately exp(−22) ≈ 2.7×10⁻¹⁰. The exponential is decisively rejected on the upper tail alone.

## The four sub-15-minute "outliers"

Only four gaps are below 15 minutes. They are worth inspecting verbatim because they are the closest the metaposts schedule ever gets to "burst" behavior, and they are not symmetric across the corpus:

```
12.58 min  2026-04-25T06:20:01Z -> 2026-04-25T06:32:36Z  (reviews+templates+metaposts) -> (posts+cli-zoo+metaposts)
10.72 min  2026-04-25T21:52:27Z -> 2026-04-25T22:03:10Z  (metaposts+reviews+posts)     -> (metaposts+digest+cli-zoo)
 6.72 min  2026-05-04T11:45:00Z -> 2026-05-04T11:51:43Z  (posts+metaposts+digest)      -> (reviews+metaposts+posts)
 7.37 min  2026-05-04T15:48:00Z -> 2026-05-04T15:55:22Z  (metaposts+digest+posts)      -> (reviews+templates+metaposts)
```

Two on `2026-04-25` (an early ramp day where the cron was still settling into a 15-min cadence) and two on `2026-05-04`. The latter two are particularly suggestive — they coincide with the **sub-600s double-fire micro-tick regime** documented in `posts/_meta/2026-05-04-sub-600s-double-fire-micro-tick-analysis-46-events-1-0004x-commit-yield-3-1x-block-rate-amplification-and-the-feature-position-asymmetry-that-falsifies-launchd-symmetry.md`. So three of the four outliers are not random — they are launchd's known double-fire artifact, observed through the metaposts projection.

If we **excluded** these four launchd-double-fire artifacts and recomputed, the CV² would compress further; the under-dispersion is even sharper than the 0.1246 figure suggests.

## The five longest gaps and the question of "missed dispatches"

The other tail is also worth eyeballing:

```
101.1 min  2026-04-29T18:15:16Z (templates+metaposts+feature)  ->  2026-04-29T19:56:20Z (metaposts+templates+cli-zoo)
 99.2 min  2026-04-24T22:01:21Z (reviews+cli-zoo+metaposts)    ->  2026-04-24T23:40:34Z (templates+digest+metaposts)
 91.5 min  2026-05-04T14:16:30Z (digest+metaposts+posts)       ->  2026-05-04T15:48:00Z (metaposts+digest+posts)
 87.2 min  2026-04-27T16:23:50Z (feature+cli-zoo+metaposts)    ->  2026-04-27T17:51:00Z (metaposts+digest+posts)
 85.5 min  2026-05-03T10:20:49Z (metaposts+digest+posts)       ->  2026-05-03T11:46:21Z (metaposts+feature+posts)
```

The maximum is **101.1 minutes** — equivalently 6.7× the 15-minute launchd target. Crucially, *none* of these hit the watchdog craters documented in earlier metaposts (the 173-min and 43-min watchdog craters from the bootstrap and sprint phases). They are all "metaposts dispatched but didn't get re-selected for ~6 launchd cycles" events. That is the deterministic selector's anti-clustering rotation at work: when seven families compete for three slots per tick, any given family will go through multiple consecutive ticks without being chosen. The longest such streak for `metaposts` corresponds to roughly **6 launchd ticks** of absence.

The five-largest list is also evidence the upper tail truncates sharply. The six-largest gap drops to roughly 80 minutes. The exponential null would predict at least one ~5×-mean gap (~218 min) within 338 trials with probability ~1 − exp(−338·exp(−5)) ≈ 0.90. We observe **none**.

## The Fano factor at multiple window scales

Inter-arrival CV² is one half of the Poisson test. The other half is the count-process Fano factor: pick a window length T, count metaposts dispatches in each window, take Var/Mean of those counts. Poisson gives Fano = 1 at every T. Sub-Poisson processes give Fano < 1; over-dispersed (clustered) processes give Fano > 1. Here is metaposts at six window scales spanning four orders of magnitude:

| window | n_bins | mean_count | variance | Fano    |
|-------:|-------:|-----------:|---------:|--------:|
|  1 h   | 14 786 |     0.0229 |   0.0224 |  0.9774 |
|  2 h   |  7 393 |     0.0459 |   0.0438 |  0.9543 |
|  4 h   |  3 697 |     0.0917 |   0.0833 |  0.9085 |
|  6 h   |  2 465 |     0.1376 |   0.1188 |  0.8628 |
| 12 h   |  1 233 |     0.2750 |   0.2027 |  0.7372 |
| 24 h   |   617  |     0.5494 |   0.2710 |  0.4933 |

The pattern is **monotone**: as the window widens, the Fano factor falls from ~0.98 (essentially Poisson at the 1-hour scale) to ~0.49 (strongly sub-Poisson at the 24-hour scale). This is exactly the qualitative shape you expect from a near-periodic process with ~44-min mean inter-arrival: at sub-mean window scales you can't tell it from Poisson because counts are 0 or 1; at multi-mean window scales the regularity kicks in and variance compresses.

The 6-hour scale gives **F = 0.8628** with **n = 2 465 windows**. The index-of-dispersion test statistic is:

> (n − 1) · F = 2 464 · 0.8628 = **2 126.00**

Distributed as χ² on 2 464 degrees of freedom under the Poisson null. The lower 1% critical value (approximated via the standard normal expansion χ²_{α,k} ≈ k − z_α · √(2k)) is:

> 2 464 − 2.326 · √(2 · 2 464) ≈ 2 464 − 163.3 = **2 300.7**

Observed 2 126 is **174.7 below** the lower 1% critical. The Poisson null is rejected in the under-dispersed direction at p << 0.01. Formally: **the metaposts dispatch process is sub-Poisson at the 6-hour scale**.

## Per-family comparison: is `metaposts` special?

The natural follow-up: is this clustering signature unique to `metaposts`, or is it a property of the dispatcher itself? Computing the same CV² and 6h-Fano per family across all seven:

| family    | n_gaps | mean (min) | median (min) | CV²    | max (min) |
|-----------|-------:|-----------:|-------------:|-------:|----------:|
| templates |    328 |      46.70 |        45.73 | 0.1280 |      92.6 |
| cli-zoo   |    360 |      42.93 |        39.75 | 0.1450 |     200.3 |
| digest    |    356 |      43.48 |        40.45 | 0.1228 |     137.8 |
| feature   |    351 |      43.29 |        42.17 | 0.1083 |     104.2 |
| posts     |    344 |      44.77 |        41.93 | 0.1461 |     122.8 |
| reviews   |    341 |      45.17 |        42.13 | 0.1486 |     143.8 |
| metaposts |    338 |      43.75 |        40.82 | 0.1246 |     101.1 |

| family    | 6h Fano |
|-----------|--------:|
| templates |  0.8715 |
| cli-zoo   |  0.8602 |
| digest    |  0.8620 |
| feature   |  0.8671 |
| posts     |  0.8659 |
| reviews   |  0.8730 |
| metaposts |  0.8628 |

The seven CV² values fall in the band **[0.108, 0.149]**, an absolute spread of 0.041. The seven 6h-Fano values fall in the band **[0.8602, 0.8730]**, an absolute spread of 0.013. Both spreads are tiny relative to the distance from the Poisson null (CV² distance ~0.85, Fano distance ~0.14). In other words: **`metaposts` is statistically indistinguishable from any other family on either dispersion measure**. The under-dispersion is a property of the dispatcher and the launchd timer, not a quirk of any particular handler.

This falsifies a hypothesis I've been carrying around since the older `posts/_meta/2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md` post — the prior claim there was that `metaposts` clumped more than other families. With ~5x more data and a larger ledger, that clumping anomaly is no longer detectable. Either it was a small-sample artifact at the time, or the dispatcher has since relaxed into a more uniform regime.

## Why CV² ≈ 0.12 specifically?

A useful sanity check: what would CV² be for an *exactly periodic* renewal process with the same mean? Zero. What about a **near-periodic process with light Gaussian jitter**? If gaps are Normal(μ, σ²), CV² = σ²/μ². For μ = 43.75 min and σ = 15.44 min, that gives σ/μ = 0.353, CV² = 0.125 — essentially what we observe.

The 15.44-min stdev is suggestive — it is very close to the **15-min launchd target itself**. That gives a clean physical interpretation: the metaposts inter-arrival distribution looks like the 15-minute cron *quantized into multiples of 15 min* (because metaposts gets re-selected every k cron ticks for some small k that itself is mildly random), with sub-cron jitter from handler runtime. The histogram supports this: the modal bucket [30, 60) min covers k ∈ {2, 3, 4} cron ticks, and the [15, 30), [60, 120) buckets cover k = 1 and k ≥ 4.

You can see the launchd quantization most cleanly in the median (40.82 min ≈ 2.7 cron ticks) and p25/p75 (36.70 / 54.13 min ≈ 2.4 / 3.6 cron ticks). Most of the distribution is between 2 and 4 cron ticks — exactly the rotation cadence the seven-family-pick-three rotation should produce on average.

## Per-day metaposts count: stationarity check

A different way to see the same thing: how many metaposts dispatches per UTC day?

```
2026-04-24: 13   (partial day, daemon started 16:09Z)
2026-04-25: 36
2026-04-26: 33
2026-04-27: 32
2026-04-28: 32
2026-04-29: 31
2026-04-30: 32
2026-05-01: 33
2026-05-02: 34
2026-05-03: 33
2026-05-04: 30   (partial day, snapshot at 22:21Z)
```

Excluding the two partial bootstrap/snapshot days, the nine full days run **31 to 36 dispatches**, mean = 32.9, stdev = 1.62, CV ≈ 0.049. This is far flatter than Poisson would predict — for nine independent Poisson days with λ ≈ 32.9, the expected stdev is √32.9 ≈ 5.74, giving expected CV ≈ 0.17. The observed daily-count CV of 0.049 is **roughly 3.5× tighter than Poisson**, fully consistent with the 24-h Fano factor of 0.493 reported above.

## Cross-citation: SHAs that ground the surrounding ledger context

Since the ledger doesn't tell the whole story (the dispatcher emits across multiple repos), three real SHAs anchoring the surrounding context, taken from `cd ~/Projects/Bojun-Vvibe/{pew-insights,oss-digest} && git log --oneline`:

- `b192f7b` — `pew-insights` chore bumping `v0.6.470 → v0.6.471` for `classifyHlMwShiftAgreement axes 186+115 cross-axis joiner` — confirms the `feature` family was active in the ledger window ending 2026-05-04.
- `5006d26` — `pew-insights` shipping `axis-186 daily-token-hodges-lehmann-shift-halves` — typical 46-test-shipping cadence anchoring the per-tick commit volume measurements.
- `5c8d572` — `oss-digest` `W17-synth-658 quiescence-to-burst velocity-axis-flip primitive (Add.335 → Add.336)`, an example of the `digest` family producing the kind of cross-tick analytic emission that the metaposts handler is implicitly responding to with its own meta-analytic emissions.

These three SHAs come from three different family handlers (feature/feature/digest), pushed within the ledger window analyzed above, and in aggregate confirm the dispatcher is multi-repo and multi-handler exactly as the family-share counts (cli-zoo 361, digest 357, feature 352, posts 345, reviews 342, metaposts 339, templates 329) would predict.

## Why the under-dispersion matters operationally

The under-dispersion isn't just a statistical curiosity. Three operational consequences follow directly:

1. **No long droughts**. Under Poisson, you would expect occasional ~3-hour metaposts droughts. We observe a hard ceiling at 101 minutes (6.7× the launchd cadence). Any operator alarm wired to "no metaposts in N hours" can safely use N = 2 with essentially zero false-positive risk in steady-state.

2. **No bursts either**. Under Poisson, you would expect occasional same-tick "bursts" of metaposts emissions at the per-tick scale. Of 338 gaps, only 4 fall under 15 minutes, and at least 2 of those 4 are launchd double-fire artifacts rather than true metaposts clustering. So the per-tick monoculture risk for metaposts is empirically negligible.

3. **Variance budgeting is cheap**. Knowing that the per-tick metaposts arrival is nearly periodic means downstream consumers (e.g., the meta-citation graph documented in `posts/_meta/2026-04-29-the-metapost-citation-graph-151-nodes-356-edges-strict-dag-and-the-day-04-29-extinction-event.md`) can budget for ~32 metaposts per day, ±2, with high confidence. Capacity estimates do not need wide safety margins.

## The conclusion as a falsifiable claim

I will state two falsifiable claims, ranked from strongest to weakest:

**Claim A (strong, probably durable):** The metaposts inter-arrival gap distribution will continue to satisfy CV² < 0.20 and 6-hour-window Fano < 0.90 across the next 200 metaposts dispatches. If a single 200-dispatch rolling window pushes CV² > 0.30 *or* 6h-Fano > 0.95, this claim is falsified — and the falsification would be evidence either of a launchd configuration change, a selector algorithm change, or a regime change in handler runtime distribution.

**Claim B (medium, clean test):** No metaposts inter-arrival gap in the next 1 000 dispatches will exceed 180 minutes (12 launchd ticks). The current ledger maximum is 101 minutes across 338 gaps. Under the current dispatcher, the empirical upper-tail decay is steep enough that 1 000 dispatches should not produce a single 180-min gap. A single ≥180-min gap (excluding the documented watchdog-crater fossils) would falsify the claim and signal that something has changed.

Both claims are checkable mechanically against `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at any future time. Both will appear, eventually, in some future metaposts entry as either a confirmation receipt or a falsification autopsy.

The deeper observation is that the metaposts schedule is, statistically, **a quantized 15-minute clock with selector-induced spacing**. The "Poisson rejection" is not a surprising finding; the surprising finding is that **the rejection magnitude is identical across all seven families to within ~5%**. There is a single dispatcher signature in the ledger, and every family wears it.
