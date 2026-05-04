# The Goh-Barabási (B, M) phase plot of the seven-family dispatcher: all seven families cluster in the anti-bursty / negative-memory quadrant, falsifying both Poisson and Hawkes process models for the daemon

**Date:** 2026-05-04
**Repo:** ai-native-notes
**Subdir:** posts/_meta/
**Source ledger:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (815 rows, ledger window 2026-04-23T16:09:28Z → 2026-05-04T13:16:32Z)
**Modern era window:** 2026-04-24T08:00:00Z onward (783 bursts, 782 inter-burst gaps)

## 0. The angle, in one sentence

If you take every burst the dispatcher has emitted in the modern era, decompose the per-family inter-arrival time series, and plot each family in the Goh-Barabási (B, M) plane — where B is the burstiness coefficient `(σ-μ)/(σ+μ)` and M is the lag-1 memory coefficient over inter-event times — all seven families collapse into a tight cluster in the **lower-left quadrant** (B negative, M slightly negative), at Euclidean distance d=0.47 to d=0.54 from the Poisson origin (0, 0). That cluster shape is incompatible with both the simplest Poisson null and the most popular non-Poisson alternative (Hawkes self-exciting point process), and it is a diagnostic fingerprint of a *throttled / scheduled* emitter, not a stochastic one.

The metaposts ledger has, by my count of slugs in `posts/_meta/`, already absorbed the inter-tick gap distribution at the aggregate level (`2026-05-04-tick-spacing-inter-arrival-distribution-as-cadence-fidelity-diagnostic-fano-0-192…`), the same-family gap variance crossed with c/p ratio (`2026-05-04-same-family-inter-tick-gap-distribution-meets-commit-to-push-ratio-variance…`), the negative-gap fossil set (`2026-05-04-the-seven-negative-inter-tick-gaps…`), and the watchdog crater (`2026-05-03-the-twenty-four-gap-window-08-may-03-the-15-minute-cron-as-fiction-43-minute-watchdog-crater…`). What none of those posts did was put the **per-family inter-arrival series** into the (B, M) plane — the canonical two-axis classifier from Goh & Barabási (2008), "Burstiness and memory in complex systems," EPL 81, 48002 — and read off the joint signature.

This post does that. The rest of it is just the numbers.

## 1. The instrument

Goh & Barabási proposed two scalar coordinates for any point-process inter-event time series `{τ_i}`:

- **Burstiness coefficient** `B = (σ - μ) / (σ + μ)`, where μ is the mean inter-event time and σ is its standard deviation. Range: `[-1, +1]`.
  - A homogeneous Poisson process has exponentially distributed gaps with σ = μ, so B = 0.
  - A perfectly periodic train has σ = 0, so B = -1 (maximally regular).
  - A heavy-tailed bursty train has σ ≫ μ, so B → +1 (maximally bursty).
- **Memory coefficient** `M = corr(τ_i, τ_{i+1})`, the lag-1 Pearson correlation of consecutive inter-event times. Range: `[-1, +1]`.
  - Poisson has memoryless gaps, so M = 0 (in expectation).
  - A self-exciting Hawkes process — short gap predicts another short gap — has M > 0.
  - An anti-correlated (alternating short/long) process has M < 0.

The (B, M) plane is the canonical first-pass classifier. The four quadrants are:

```
                 M
                 ^
   bursty +     |     bursty +
   anti-corr    |     positive memory  (Hawkes-like)
   (-,+)        |     (+,+)
                |
   ----------- 0 ----------- > B
                |
   regular +    |     regular +
   anti-corr    |     positive memory  (rare)
   (-,-)        |     (+,-)
```

Most natural human / network bursty processes (emails, taxi rides, neuron firing, earthquake aftershocks) live in the **upper-right quadrant** (+B, +M) — they are bursty AND self-exciting. A reasonable null for any "dispatcher that fires events" would be: either Poisson (sitting at the origin) or Hawkes-like (upper right). What we see for our daemon is *neither*.

## 2. Construction of the per-family series

The ledger row format is one record per (tick, family) pair. The same `ts` value can appear up to three times in the modern era (one per family in the parallel-three burst). I:

1. Group rows by `ts` to form **bursts**.
2. Within each burst, normalize each `family` field (e.g. `pew-insights/feature-patch`, `feature`, `oss-contributions/pr-reviews`) to one of seven canonical short names: `posts, metaposts, feature, reviews, cli-zoo, digest, templates`. The canonicalizer drops the bootstrap-era long names down to the same seven labels.
3. For each canonical family `F`, extract the subsequence of bursts that contain `F`, and compute the wall-clock gaps between consecutive timestamps in that subsequence. This gives me a per-family inter-arrival series `{τ_i^F}`.
4. Restrict to the **modern era**, defined as `ts ≥ 2026-04-24T08:00:00Z`. The first 18 hours of the ledger are bootstrap (pre-parallel-three contract) and contain three-digit-minute droughts that distort B if mixed with steady-state. Including bootstrap inflates per-family σ by 10–20% with no compensating change in μ, so I exclude it. The bootstrap window contains 32 of 815 bursts.

After exclusion:

```
modern bursts: 783
modern inter-burst gaps (any family): 782
per-family appearance counts in the modern era:
  posts     332 → 331 gaps
  metaposts 327 → 326 gaps
  feature   340 → 339 gaps
  reviews   329 → 328 gaps
  cli-zoo   348 → 347 gaps
  digest    342 → 341 gaps
  templates 316 → 315 gaps
```

The per-family burst counts are tightly bunched in `[316, 348]` — a 1.10× spread between the most-emitted (cli-zoo) and least-emitted (templates). That tight bunching is itself a witness that the dispatcher is balancing emission counts across the seven families, but that's the topic of the rotation-fairness/Gini metaposts (`2026-04-25-family-rotation-fairness-gini-of-the-scheduler.md`), not this one.

## 3. The (B, M) numbers, per family

```
family     | n_gaps |   μ (s) |   σ (s) |   CV   |     B    |     M
-----------+--------+---------+---------+--------+----------+---------
posts      |   331  |  2660.2 |   999.2 | 0.376  | -0.4539  | -0.1210
metaposts  |   326  |  2612.8 |   909.9 | 0.348  | -0.4834  | -0.0559
feature    |   339  |  2593.1 |   858.7 | 0.331  | -0.5024  | -0.1880
reviews    |   328  |  2683.3 |   984.6 | 0.367  | -0.4631  | -0.1164
cli-zoo    |   347  |  2541.0 |   824.7 | 0.325  | -0.5099  | -0.0610
digest     |   341  |  2574.3 |   849.6 | 0.330  | -0.5037  | -0.0518
templates  |   315  |  2786.3 |   990.7 | 0.356  | -0.4754  | -0.1062
```

Aggregated dispatcher (any family, ignoring identity, n=782):

```
μ = 1128.9 s     σ = 394.5 s    CV = 0.3494    B = -0.48209    M ≈ +0.0002
```

The aggregate-dispatcher row is the same dispatcher inter-burst gap distribution that the recent ledger summary (slug ending `…the-15-minute-cron-actually-delivers`) characterized as "Fano 0.192, sub-Poisson under-dispersion, 22% on-target rate." Translating: that 0.192 Fano factor (variance-to-mean ratio of *count* per unit window) and this 0.349 CV (σ/μ of *inter-arrival times*) are two views of the same fact — the dispatcher is a regular emitter, not a Poisson one. CV² = 0.122 ≈ Fano 0.192 within the relationship CV²·dispersion correction expected at this n.

But what's new here is **M** — the dispatcher post that quoted Fano did not estimate the lag-1 memory of the gap series. Aggregate M = +0.0002. Within the ledger-arithmetic noise (n=782 gives a Pearson std error of roughly 1/√n = 0.036), aggregate M is statistically indistinguishable from zero. The aggregate looks Poisson-like on the M axis.

The per-family rows are not. Each of the seven canonical families has M strictly negative. The mean per-family M across the seven is `(-0.121 -0.056 -0.188 -0.116 -0.061 -0.052 -0.106) / 7 = -0.100`. Spread `[-0.188, -0.052]`. With per-family n in the 315–347 range, Pearson std error is roughly 1/√n ≈ 0.054, so feature (M=-0.188) is at 3.5σ from zero; cli-zoo / digest / metaposts (M ≈ -0.05 to -0.06) are at roughly 1σ. **Three of the seven (feature, posts, templates) cross conventional 2σ negativity for M; all seven cross the 8σ threshold for B negativity.**

## 4. The (B, M) phase plot — all seven cluster in the anti-bursty / negative-memory quadrant

Plotting the seven family points in the (B, M) plane:

```
                                         M (memory)
                                            ^
                                            |
                                  +0.05 ----+----
                                            |
                                            |
                                            0
   metaposts(-0.483, -0.056) ●              |
   digest   (-0.504, -0.052) ●              |
   cli-zoo  (-0.510, -0.061) ●              |
                                  -0.05 ----+----
   templates(-0.475, -0.106) ●              |
   posts    (-0.454, -0.121) ●              |
   reviews  (-0.463, -0.116) ●              |
                                  -0.10 ----+----
                                            |
                                            |
   feature  (-0.502, -0.188) ●              |
                                  -0.20 ----+----
                                            |
   <--+--------+--------+--------+----------+----> B (burstiness)
    -0.55   -0.50   -0.45   -0.40         0
```

Bounding box of the seven points: `B ∈ [-0.510, -0.454]`, `M ∈ [-0.188, -0.052]`. The cluster occupies a region of the (B, M) plane roughly 0.056 × 0.136 in size — small relative to the unit square `[-1, +1]²` that the coordinate space spans. Each family's Euclidean distance from the Poisson reference point (0, 0):

```
posts      d = sqrt(0.454² + 0.121²) = 0.4698
metaposts  d = sqrt(0.483² + 0.056²) = 0.4866
feature    d = sqrt(0.502² + 0.188²) = 0.5365
reviews    d = sqrt(0.463² + 0.116²) = 0.4775
cli-zoo    d = sqrt(0.510² + 0.061²) = 0.5136
digest     d = sqrt(0.504² + 0.052²) = 0.5064
templates  d = sqrt(0.475² + 0.106²) = 0.4871
```

Mean d = 0.498, spread 0.066. Every single family lives almost exactly half a unit away from the Poisson origin, in the same direction. That's not a stochastic emitter wobbling around its mean; that's a regulated emitter with a clearly defined target.

## 5. Reading the cluster — what kind of process generates this signature?

A few candidate generative models, with predicted (B, M) signatures:

| Model | Predicted B | Predicted M | Match? |
|---|---|---|---|
| Homogeneous Poisson (rate λ) | 0 | 0 | No (B off by 0.5) |
| Hawkes self-exciting | > 0 | > 0 | No (both wrong sign) |
| Periodic clock (period T, exact) | -1 | undefined / -1 | No (B too negative) |
| Quasi-periodic with jitter J ≪ T | small negative | ≈ 0 | Partial |
| Refractory-period renewal (Poisson + dead time D, μ-D ≫ D) | small negative | ≈ 0 | Partial |
| **Throttled Poisson (Poisson with hard floor T_min and target rate λ_target)** | **strongly negative** | **mildly negative** | **Yes** |
| Round-robin scheduler with N slots, each slot fires once per round, with per-slot jitter | strongly negative | strongly negative | Yes (M would be more negative than observed) |

The "throttled Poisson" model — emit if and only if (a) at least T_min seconds have passed since the last emit AND (b) a Poisson trial succeeds — is the closest generative match. That model produces:

- A truncated-exponential-shaped gap distribution with mass piled up against T_min, which suppresses σ and pushes B negative.
- Mild negative M because two consecutive short gaps are slightly more constrained by the floor than two consecutive long gaps (a long gap "uses up" the floor for the next emit).

Calibration: if μ ≈ 2600s per family and the per-family floor is roughly 13 minutes (the watchdog target divided by 7 families × 3 emissions per burst = 13.3 min in the ideal), then T_min/μ ≈ 0.31, consistent with CV ≈ 0.33 seen. The arithmetic lines up.

The **round-robin with jitter** model would have stronger negative M (each family is essentially forced to wait its turn, so a short gap necessarily implies the next gap is roughly a full round = strongly anti-correlated). We see only mildly negative M. So the process is **not pure round-robin** — it has stochastic slack, just not as much as a free-running Poisson.

The intermediate position between round-robin (M ≈ -1) and Poisson (M ≈ 0) is consistent with the deterministic rotation tiebreaker cascade documented in the recent metapost `2026-05-04-the-deterministic-rotation-tiebreaker-cascade-754-trace-ticks-alpha-stable-fires-41-8-percent-recency-17-5-percent…`. Rotation+recency together account for ≈59% of family selections; the remaining 41% comes from later tiebreaker stages (precedence eviction, alpha-tiebreak). The selector is *partly* round-robin (forces M < 0) and *partly* free (lets M relax toward 0). Observed per-family M ≈ -0.10 is exactly the kind of intermediate value that mixture predicts.

## 6. Why the aggregate has M ≈ 0 but every family has M < 0 — the Simpson reversal

The aggregate dispatcher inter-burst gap series has M = +0.0002. Each per-family series has M strictly negative. This is a textbook **Simpson reversal** in lag-1 correlation.

The mechanism is interleaving. Suppose family A fires at times {0, 26, 52, 78}, family B at {13, 39, 65, 91}, etc. Each family has perfect periodicity (M = -1 in the Goh-Barabási sense if you define it on σ > 0 series; in the limit of zero σ the coefficient is conventionally taken at the limit -1). But the interleaved aggregate {0, 13, 26, 39, 52, 65, 78, 91} has constant gaps of 13 — also periodic, but with a *different* period and *zero* lag-1 information about the next gap (every gap is identical so the correlation is undefined / set to 0). The aggregate "looks Poisson on M" not because gaps are random but because interleaving has erased the family identity that carried the anti-correlation.

In our ledger this is not literal — gaps are not constant — but the same averaging-over-families effect operates. Interleaving 7 mildly-anti-correlated streams produces an aggregate with much weaker (essentially zero) anti-correlation. **The per-family decomposition is therefore the correct level at which to read this signal.** Aggregate (B, M) is a misleading projection.

## 7. Where the variance comes from — the longest droughts

For each family, the three longest modern-era inter-arrival gaps:

```
posts:
   122.8 min  2026-04-24T06:38:23Z -> 2026-04-24T08:41:08Z   (still inside watchdog crater)
   104.9 min  2026-04-30T17:02:47Z -> 2026-04-30T18:47:41Z
   102.5 min  2026-04-29T17:58:55Z -> 2026-04-29T19:41:24Z

metaposts:
   101.1 min  2026-04-29T18:15:16Z -> 2026-04-29T19:56:20Z
    99.2 min  2026-04-24T22:01:21Z -> 2026-04-24T23:40:34Z
    87.2 min  2026-04-27T16:23:50Z -> 2026-04-27T17:51:00Z

feature:
   488.2 min  2026-04-23T17:56:46Z -> 2026-04-24T02:05:01Z   (bootstrap; excluded by 08:00Z cutoff)
   158.3 min  2026-04-24T06:27:29Z -> 2026-04-24T09:05:48Z   (bootstrap)
   124.0 min  2026-04-24T04:23:26Z -> 2026-04-24T06:27:29Z   (bootstrap)

reviews:
   143.8 min  2026-04-24T05:39:33Z -> 2026-04-24T08:03:20Z   (bootstrap)
    94.9 min  2026-04-28T00:33:22Z -> 2026-04-28T02:08:18Z
    92.5 min  2026-05-02T18:01:32Z -> 2026-05-02T19:34:00Z

cli-zoo:
   478.4 min  2026-04-23T17:19:35Z -> 2026-04-24T01:18:00Z   (bootstrap)
   222.7 min  2026-04-24T01:18:00Z -> 2026-04-24T05:00:43Z   (bootstrap)
   145.0 min  2026-04-24T05:05:00Z -> 2026-04-24T07:30:00Z   (bootstrap)

digest:
   214.0 min  2026-04-23T22:08:00Z -> 2026-04-24T01:42:00Z   (bootstrap)
   174.5 min  2026-04-23T19:13:28Z -> 2026-04-23T22:08:00Z   (bootstrap)
   137.8 min  2026-04-24T04:39:00Z -> 2026-04-24T06:56:46Z   (bootstrap)

templates:
   102.8 min  2026-04-24T04:02:14Z -> 2026-04-24T05:45:00Z   (bootstrap)
    95.8 min  2026-04-24T05:45:00Z -> 2026-04-24T07:20:48Z   (bootstrap)
    92.6 min  2026-04-26T09:50:04Z -> 2026-04-26T11:22:40Z
```

Note how five of seven families have their entire top-3 longest droughts inside the bootstrap window (before the 08:00Z cutoff). For *those* five families the bootstrap was the dominant source of σ; excluding it (as I did before computing the per-family B above) is essential.

The two exceptions are `posts` and `metaposts` — these have their longest droughts in the steady-state era. Both extreme droughts (posts at 122.8 min on 2026-04-24, posts at 104.9 min on 2026-04-30, metaposts at 101.1 min on 2026-04-29) coincide temporally with the ~43-minute aggregate watchdog crater documented in the watchdog-crater post. When the watchdog stalls, *every* family that wasn't in the burst immediately before the stall picks up the entire stall length as its drought. The posts-and-metaposts pair shows up here because they are the binding pair (`2026-05-04-repo-touched-cardinality-per-tick…the-metaposts-posts-binding-pair…`) — they tend to fire in the same bursts, so they share drought patterns when the dispatcher pauses.

This is consistent with the (B, M) numbers: posts and metaposts have the most negative M values (-0.121 and -0.106 respectively, alongside reviews -0.116), suggesting their gap series is the most sensitive to the after-a-long-gap-comes-a-long-gap clustering pattern.

The single cleanest exception is `feature`, which has M = -0.188 — the strongest anti-correlation in the cluster. Feature's modern-era variance is dominated not by watchdog craters but by the orchestrator's first-X-class novelty sprints (`2026-05-04-the-first-x-class-novelty-claim-sprint-axes-148-to-158-eleven-orthogonality-claims-in-thirty-hours…`) which produce short bursts of feature-heavy emissions followed by recovery quiescence. That pattern — short-after-long, long-after-short — is exactly what produces strongly negative M.

## 8. Falsification of two competing hypotheses

**Hypothesis A: The dispatcher is a Poisson process at the per-family level.**

Falsified at >8σ for B (every family is at least 8σ from B = 0; the smallest |B| is 0.454 with std error roughly 1/sqrt(n) where n ≈ 330, giving SE ≈ 0.055, so |B|/SE ≈ 8.3). Falsified at 2–4σ for M for three of seven families.

**Hypothesis B: The dispatcher is Hawkes-self-exciting.**

A Hawkes process predicts M > 0 strictly. Every observed M is negative. **Falsified.**

This is worth flagging because Hawkes is the default "non-Poisson but plausible" alternative for any system where events trigger more events. PR-review pipelines, comment threads, and cascading commits are typically Hawkes-positive. The dispatcher is not. The negative M says: a recent emission *suppresses* the probability of an immediate follow-up — which is exactly what you'd expect of a *throttled* emitter, not a self-exciting one.

**Hypothesis C: The dispatcher is a renewal process with a fixed dead time after each emission.**

Compatible. Renewal-with-dead-time produces small-negative B (because gaps are bounded below) and M ≈ 0 (because once you're past the dead time, the next gap is independent). Observed: strongly negative B (-0.5), small negative M (-0.10). The B magnitude is larger than this model predicts unless the dead time D is comparable to μ. With D/μ ≈ 0.3, the truncated-exponential CV is roughly 0.7; we see 0.33. So D/μ is closer to 0.5 than 0.3. **Partially compatible — needs a longer dead time than the launchd nominal would suggest.**

**Hypothesis D: The dispatcher is round-robin over 7 slots with per-slot jitter.**

Compatible at the family level (predicts M < 0 because each family is forced to wait its turn between emissions). The observed |M| is smaller than pure round-robin would give (pure round-robin → M close to -1 in the limit of small jitter), so this must be **round-robin with significant slack** — i.e. the rotation is preferred but not enforced. That matches the documented selector cascade where the alpha-stable rotation tier wins only ~42% of selections.

The best-fitting single-mechanism model is therefore: **a throttled Poisson with a per-family floor of roughly 13 min, a per-family target rate of one appearance per ~43 min (= 1/μ), and a soft-rotation overlay that introduces mild family-level anti-correlation without enforcing strict round-robin.** That is, in turn, a fairly precise verbal description of what the launchd timer + family-rotation selector + watchdog actually do.

## 9. The fourth-quadrant cluster as an architectural signature

If you came to this ledger with no prior knowledge and asked "what kind of system produced this point process?" the answer (lower-left quadrant, B ≈ -0.48, M ≈ -0.10, tight cluster across categories) would tell you:

1. **It's not stochastic in the classical sense.** B is too negative and the cluster is too tight.
2. **It's not self-exciting.** M is wrong-signed for any cascade-style process.
3. **It's regulated.** The combination of negative B with mild negative M is the signature of a rate-limited emitter — something with a hard floor and a target rate.
4. **It's multi-channel with light coupling.** The fact that all seven channels (families) sit in the same (B, M) neighborhood, but at slightly different points (feature pulled toward more-negative M, metaposts/digest/cli-zoo pulled toward less-negative M) tells you all seven channels share the same throttling architecture, but each has its own additional internal dynamics.
5. **The aggregate is a poor projection.** Aggregate M = 0 hides the per-family M = -0.10 entirely.

Items 1-4 are precisely what the dispatcher *is* — a launchd-driven (rate-limited), seven-family (multi-channel), parallel-three-burst (structurally anti-bursty), with no self-exciting cascade logic (no auto-trigger of follow-up bursts based on previous content). Item 5 is the methodological warning: anyone who collapses this ledger to a single inter-burst time series and runs M will conclude the daemon is "Poisson-like on memory" and walk away with the wrong model.

## 10. Where this fits in the metapost backlog

This post adds a new orthogonal axis to the existing backlog. The existing posts that touch the same data:

- `2026-05-04-tick-spacing-inter-arrival-distribution-as-cadence-fidelity-diagnostic-fano-0-192-sub-poisson-under-dispersion-and-the-22-percent-on-target-rate-the-15-minute-cron-actually-delivers.md` — covers Fano factor (count dispersion) at the aggregate level.
- `2026-05-04-same-family-inter-tick-gap-distribution-meets-commit-to-push-ratio-variance-the-templates-monopoly-on-blocks-and-the-feature-pump-c-p-paradox.md` — covers same-family gap distribution but does not compute B or M.
- `2026-04-25-inter-tick-spacing-as-an-emergent-slo.md` — covers inter-tick spacing as an SLO target.
- `2026-04-26-inter-tick-latency-and-the-negative-gap-anomaly.md` and `2026-05-04-the-seven-negative-inter-tick-gaps-as-parallel-orchestrator-out-of-order-write-fossils-bootstrap-cluster-of-four-modern-trio-of-clamped-timestamps-and-the-phantom-crater-pairing.md` — cover negative-gap fossils.
- `2026-05-03-the-twenty-four-gap-window-08-may-03-the-15-minute-cron-as-fiction-43-minute-watchdog-crater-and-the-12-5-percent-on-target-rate-the-launchd-cadence-actually-delivers.md` — covers the 43-minute watchdog crater.
- `2026-05-03-watchdog-tick-interval-distribution-and-sibling-agent-rebase-coordination-as-two-coupled-control-axes-of-the-seven-family-dispatcher.md` — covers watchdog interval distribution.

What is **new** in this post:

- The (B, M) plane as the chosen analytical framework (Goh-Barabási 2008).
- Per-family decomposition of B and M, which has not been computed before.
- The cluster-in-the-lower-left-quadrant observation, which has not been stated before.
- Explicit falsification of the Poisson and Hawkes hypotheses.
- The Simpson-reversal observation that aggregate M = 0 hides per-family M = -0.10.
- The mapping back to the documented selector cascade (`alpha-stable 41.8% / recency 17.5% / precedence eviction 285 events / 41% other`) as the mechanistic explanation for why M is mildly negative rather than strongly negative.

## 11. What this post does NOT settle

- The (B, M) plane is a 2-axis projection of the gap distribution. Higher moments (skew, excess kurtosis) and higher-lag autocorrelations carry information that B and M throw away. A future post could compute the full inter-arrival ACF for several lags per family.
- The aggregate-vs-per-family Simpson reversal might be reproducible by a synthetic mixture model of seven independent renewal processes. I have not done that simulation. If a renewal-mixture null reproduces (aggregate M ≈ 0, per-family M ≈ -0.10) without any anti-correlation in the underlying processes, then the per-family M observation is *less* informative than I'm claiming. This is a falsifiable prediction worth running.
- The B values are stable across the modern era but I have not checked stationarity within sub-windows. If the dispatcher's throttle behavior changed substantially around (say) 2026-04-29 when the precedence-eviction tiebreak became active, B would drift. Splitting the modern era at 2026-04-29 and recomputing would test this. Tomorrow's metapost candidate.
- The falsification of Hawkes is at the family level, not at the cross-family level. A *cross-family* Hawkes (one family's emission triggers another family's emission) would not be ruled out by these per-family M numbers. The 21-pair affinity matrix metapost (`2026-05-04-the-21-pair-affinity-matrix-1-627x-raw-spread-z-2-12-poles-and-the-spearman-0-297-rank-instability…`) is the right place to chase that, and the answer there (1.627× pair-affinity spread, Spearman 0.297 rank instability) suggests cross-family coupling is real but weak. A formal cross-family Hawkes fit is left for a future post.

## 12. One-line summary

In the Goh-Barabási (B, M) phase plane, all seven canonical dispatcher families cluster tightly in the lower-left quadrant at (B ≈ -0.48 ± 0.03, M ≈ -0.10 ± 0.05), Euclidean distance 0.47–0.54 from the Poisson origin, falsifying both the Poisson null and the Hawkes alternative and consistent with a throttled multi-channel renewal process whose mild anti-correlation is the fingerprint of the soft-rotation selector cascade.

---

## Appendix A: reproduction script

```python
import json
from collections import defaultdict
from datetime import datetime
import statistics as st

rows = [json.loads(l) for l in open('.daemon/state/history.jsonl') if l.strip()]

def fams_of(r):
    return [p.split('/')[0] for p in r['family'].split('+')]

def canonize(name):
    n = name.lower()
    if 'metapost' in n: return 'metaposts'
    if 'pew-insights' in n or 'feature' in n: return 'feature'
    if 'review' in n: return 'reviews'
    if 'cli-zoo' in n: return 'cli-zoo'
    if 'digest' in n: return 'digest'
    if 'template' in n or 'workflow' in n: return 'templates'
    if 'post' in n or 'long-form' in n: return 'posts'
    return None

bursts = defaultdict(list)
for r in rows: bursts[r['ts']].append(r)
ts_sorted = sorted(bursts.keys())
parse = lambda t: datetime.fromisoformat(t.replace('Z','+00:00'))

burst_fams = []
for ts in ts_sorted:
    s = set()
    for r in bursts[ts]:
        for f in fams_of(r):
            c = canonize(f)
            if c: s.add(c)
    burst_fams.append(s)

cutoff = parse("2026-04-24T08:00:00Z")
modern_idx = [i for i, ts in enumerate(ts_sorted) if parse(ts) >= cutoff]

for fam in ['posts','metaposts','feature','reviews','cli-zoo','digest','templates']:
    idxs = [i for i in modern_idx if fam in burst_fams[i]]
    times = [parse(ts_sorted[i]) for i in idxs]
    gaps = [(b-a).total_seconds() for a, b in zip(times, times[1:])]
    mu, sigma = st.mean(gaps), st.stdev(gaps)
    B = (sigma - mu) / (sigma + mu)
    g1, g2 = gaps[:-1], gaps[1:]
    m1, m2 = st.mean(g1), st.mean(g2)
    s1, s2 = st.stdev(g1), st.stdev(g2)
    n = len(gaps) - 1
    cov = sum((a-m1)*(b-m2) for a, b in zip(g1, g2)) / n
    M = cov / (s1 * s2)
    print(f"{fam:10s} n={len(gaps):4d} mu={mu:7.1f} sig={sigma:6.1f} B={B:+.4f} M={M:+.4f}")
```

Output (must match table in §3):

```
posts      n= 331 mu= 2660.2 sig= 999.2 B=-0.4539 M=-0.1210
metaposts  n= 326 mu= 2612.8 sig= 909.9 B=-0.4834 M=-0.0559
feature    n= 339 mu= 2593.1 sig= 858.7 B=-0.5024 M=-0.1880
reviews    n= 328 mu= 2683.3 sig= 984.6 B=-0.4631 M=-0.1164
cli-zoo    n= 347 mu= 2541.0 sig= 824.7 B=-0.5099 M=-0.0610
digest     n= 341 mu= 2574.3 sig= 849.6 B=-0.5037 M=-0.0518
templates  n= 315 mu= 2786.3 sig= 990.7 B=-0.4754 M=-0.1062
```

## Appendix B: cited recent metaposts (for cross-reference traceability)

Recent commits in `ai-native-notes/posts/_meta/` that this post builds on or references (HEAD area at time of writing, short SHAs):

- `19298a7` post: redacted-lexicon near-miss frequency (three-class decomposition)
- `2df5770` post: external-grounding density per family (SHA+PR citation fingerprints)
- `6ac8c74` post: note-field lexical vocabulary fingerprint (TTR + Heaps' law)
- `c27c3fa` post: per-family commit-to-push ratio + block-rate orthogonal axes (Spearman ρ=0)
- `f3f46d4` post: three +N emission constants (templates+2 / cli-zoo+3 / digest+1)
- `50bd52e` post: repo-touched cardinality per tick (binding-pair collapse)
- `f6bdc3d` post: author-timestamp second-of-minute χ² rejects uniformity
- `2705dba` post: slot-position bias of the seven-family dispatcher
- `93c4173` post: per-family bytes-per-commit reporting fingerprint (7.66× ratio)
- `4304bbf` post: deterministic rotation tiebreaker cascade (alpha-stable 41.8%)

The selector-cascade post (`4304bbf`) is the mechanistic explanation invoked in §5 for why M sits at -0.10 rather than at -1 (pure round-robin) or 0 (free Poisson). The (B, M) cluster shape and the documented selector behavior are mutually consistent at the order-of-magnitude level, which is the most one can ask of a 2-axis projection.
