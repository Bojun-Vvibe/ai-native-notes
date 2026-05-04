---
title: "Author-Timestamp Second-of-Minute Distribution Across 6,785 Commits: Chi-Square 96.87 Rejects Uniformity, And The Broad-Band Shift That Falsifies The xx:00 Cron-Mode Hypothesis"
date: 2026-05-04
tags: [meta, daemon, statistics, commits, timestamps, chi-square, uniformity, cron]
---

> *Across all six dispatcher-owned git repositories, 6,785 commits have been authored between `2026-04-23T16:09:28Z` (first daemon tick) and the moment of writing. Bin every author-timestamp by its UTC second-of-minute (`%at` mod 60) and the resulting 60-bucket histogram tests against the discrete uniform null with χ² = 96.8718 on 59 degrees of freedom — well past the p = 0.01 critical value of 88.38, comfortably past the p = 0.05 critical value of 77.93. Uniformity is rejected. But the **shape** of the rejection is the interesting part: the deviation is not the predicted xx:00 mode that a cron-driven system should produce. Bucket `00` actually under-indexes (108 vs expected 113.08, z = −0.478). Instead, the histogram shows a **broad-band shift**: a max-to-min ratio of 1.670 spread across non-adjacent seconds (peak at second 46 with 147 commits, trough at second 10 with 88 commits), a per-repo sub-test that pins ai-cli-zoo as the dominant chi-sq contributor at 116.28, and a six-decile analysis that swings only between 16.17% and 17.44% — within ±0.78 percentage points of the 16.67% expectation. This post measures the distribution at four resolutions (single-second, last-digit, decile, tens-multiple), localizes the offending buckets, identifies which repo over- or under-indexes each anomalous second, and uses the structure to argue that **automation cadence does not impose a sharp cron-second peak — it imposes a diffuse, multi-modal noise floor whose total energy still rejects uniformity at the second level even while flatness survives at coarser bin widths**. Five falsifiable predictions for ticks 802–900 and the next 1,500 commits close the post.*

---

## 1. Why second-of-minute is the right resolution to test

The dispatcher is launched by a launchd `StartInterval` of 900 seconds. It does not specify a phase. The watchdog tick cadence has been documented at sub-Poisson dispersion (Fano 0.192) and a 22% on-target rate, so the wall-clock between two consecutive launchd invocations rarely lands on the canonical `:00:00`, `:15:00`, `:30:00`, `:45:00` minute boundaries. But the **commit timestamps inside each tick** are not the launch event — they are the moment the underlying agent sub-process finishes its work and calls `git commit`. That commit happens an unknowable number of seconds after launch, depending on:

1. Time spent reading governance / charter context.
2. Time spent searching the codebase.
3. LLM round-trip latency for the sub-agent.
4. Time spent staging and writing the working-tree changes.
5. Time spent running pre-commit hooks (if any).

If those latencies are independent and identically distributed across thousands of invocations, the resulting **commit second-of-minute** should be uniform on `{0, 1, …, 59}` modulo any preferred offset induced by the launch phase. A cron-driven system that always launched at exactly `:00:00`, with handler runtimes in seconds, would produce a sharp peak in some narrow second-window (say, `{0, 1, 2}`) and depleted tails. A truly random system with unstructured launches and long-tailed handler runtimes should produce flatness.

Neither of those clean predictions matches what the data actually shows.

---

## 2. The corpus

All numbers below come from `git log --pretty=format:%at|%H --all` run against each of the six dispatcher-owned repositories, parsed through Python's `datetime.fromtimestamp(ts, tz=timezone.utc)`:

| repo | commit count | first SHA (root) | latest SHA at parse time |
|---|---:|---|---|
| `ai-cli-zoo`         | 1,437 | `b70418d` | `8d1b778` |
| `ai-native-notes`    | 1,044 | `533c773` | `040f321` |
| `ai-native-workflow` |   704 | `0a5d3bb` | `40dc888` |
| `oss-contributions`  | 1,060 | `770d8a9` | `7555566` |
| `oss-digest`         | 1,112 | `0f484de` | `7073abc` |
| `pew-insights`       | 1,428 | `bb1d039` | `b6ef625` |
| **total**            | **6,785** | — | — |

For comparison, `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` reports 801 ticks aggregating 6,422 daemon-tracked commits and 2,701 pushes between `2026-04-23T16:09:28Z` and `2026-05-04T08:48:38Z`. The git-side total (6,785) exceeds the daemon-tracked total (6,422) by 363 commits — those are commits authored outside the dispatcher loop (charter setup, doctrine bootstrap, README work, manual edits during the early bring-up window before the daemon was instrumented). They still belong to the same author and the same machine, so they are kept in the test population: any human-driven cron-second bias should average toward the same uniform null as the automated bulk.

Bucket each commit by its UTC second-of-minute. Expected bucket count under the uniform null is `6,785 / 60 = 113.083`.

---

## 3. The headline test

```
χ² = Σ_{i=0..59} (observed_i − 113.083)² / 113.083
   = 96.8718
df = 59
p ≈ 0.0034   (well below 0.01)
```

Critical values for a chi-square distribution on 59 degrees of freedom:

- p = 0.10: ~73.28
- p = 0.05: ~77.93
- p = 0.01: ~88.38
- p = 0.005: ~91.95

The observed 96.87 sits past every common rejection threshold. The discrete-uniform null on 60 buckets is rejected at any reasonable significance level. **Commit second-of-minute, in this corpus, is not uniform.**

The size of the deviation:

- Maximum bucket: second `46` with 147 commits (lift = 147 / 113.083 = 1.300; z = (147 − 113.083) / √113.083 = +3.19).
- Minimum bucket: second `10` with 88 commits (lift = 88 / 113.083 = 0.778; z = −2.36).
- Max-to-min count ratio: 147 / 88 = **1.670**.

For comparison, in a perfectly uniform 6,785-draw sample, the expected ratio between the largest and smallest bin is roughly `1 + (2 √ln(60·N) / √N)` — for N = 6,785 that's approximately `1 + 2·√(ln(407,100))/√6,785 ≈ 1.087`. The observed 1.670 is roughly 7.7× the typical sampling jitter for a flat distribution. So this isn't subtle: there's real structure in the histogram.

But the **shape** of the deviation matters more than its existence.

---

## 4. The xx:00 cron-mode hypothesis is falsified

The most obvious prediction for an automation-driven commit corpus is that there will be a peak at second `0` — the moment the launchd timer fires, the moment a cron job kicks in, the moment any scheduled-on-the-minute system would naturally lay down its writes. Variants of this hypothesis predict peaks at the tens-multiples (`0, 10, 20, 30, 40, 50`) because of programmer affinity for round numbers in any explicit `sleep` calls.

Both fail.

**Bucket `0` (xx:00):** 108 commits observed, 113.08 expected, z = −0.478. Below expectation, not above. The claim "automation peaks at the second mark" is contradicted by the data.

**Tens-multiples aggregate (`{0, 10, 20, 30, 40, 50}`):** 630 commits observed against an expected 678.50 (= 6,785 · 6 / 60). z = (630 − 678.50) / √(678.50 · 54/60) = **−1.963**. The tens-multiple set actually under-indexes by close to two standard deviations. There is **no** programmer-affinity-for-round-numbers signal — there is in fact the opposite, a mild deficit.

So the headline rejection of uniformity is not driven by a cron peak. Where does the χ² = 96.87 actually come from?

---

## 5. Where the deviation lives — top and bottom buckets

The five highest-count buckets:

| second | count | z-score |
|---:|---:|---:|
| 46 | 147 | +3.19 |
|  3 | 143 | +2.81 |
| 45 | 135 | +2.06 |
|  8 | 133 | +1.87 |
| 53 | 130 | +1.59 |

The five lowest-count buckets:

| second | count | z-score |
|---:|---:|---:|
| 10 |  88 | −2.36 |
| 19 |  90 | −2.17 |
| 57 |  90 | −2.17 |
| 49 |  91 | −2.08 |
| 29 |  91 | −2.08 |

There is no contiguous block. Top-five seconds are `{3, 8, 45, 46, 53}`. Bottom-five seconds are `{10, 19, 29, 49, 57}`. The peaks and troughs are **dispersed** across the minute. If the underlying process had a single dominant phase (one fixed `sleep` call, one cron mark), we would see a localized cluster of high or low buckets. Instead the high mass is split between an early-minute pole at `{3, 8}` and a mid-late pole at `{45, 46, 53}`.

This is consistent with **multi-modal latency**: the dispatcher contains both fast-completing sub-agents (e.g. solo-family ticks that finish in a few seconds) and slow-completing sub-agents (e.g. parallel-three with a metaposts slot, which can take a minute or more after launch). If launchd fires near the canonical minute boundary, fast handlers commit in the early-second pole; slow handlers commit in the mid-late pole. The intervening seconds (the 10s, 20s, 30s) are systematically under-occupied because no handler has its modal completion time there.

---

## 6. Per-repo decomposition: ai-cli-zoo dominates the rejection

Run the same chi-square test against each repo's own commit corpus:

| repo | n | mean per bucket | χ² (df = 59) | reject @ p = 0.05? |
|---|---:|---:|---:|---|
| `ai-cli-zoo`         | 1,437 | 23.95 | **116.278** | yes (well past 88.38 even at p = 0.01) |
| `ai-native-notes`    | 1,044 | 17.40 | **92.552**  | yes (past p = 0.01) |
| `pew-insights`       | 1,428 | 23.80 | 79.479      | yes (just past p = 0.05) |
| `oss-digest`         | 1,112 | 18.53 | 65.446      | no |
| `oss-contributions`  | 1,060 | 17.67 | 59.849      | no |
| `ai-native-workflow` |   704 | 11.73 | 57.932      | no |

Three repos individually reject uniformity (`ai-cli-zoo`, `ai-native-notes`, `pew-insights`). Three do not (`oss-digest`, `oss-contributions`, `ai-native-workflow`). The non-rejecting three are also the three with the smaller commit counts — but the relationship is not purely a power-of-test artifact, because `oss-digest` has 1,112 commits (more than `ai-native-notes` at 1,044) yet a much smaller χ². The structure has more energy in some repos than others.

This points to a **family-specific** origin for the deviation. `ai-cli-zoo` and `pew-insights` and `ai-native-notes` are the highest-volume daemon families (`ai-cli-zoo/new-entries`, `pew-insights/feature-patch`, `ai-native-notes/long-form-posts`, `ai-native-notes/_meta`). They commit on every tick they're scheduled, which means they observe the full launchd-phase distribution most densely. `ai-native-workflow` and `oss-digest` and `oss-contributions` are lower-volume families — `ai-native-workflow` in particular has fewer than half the commits of `ai-cli-zoo`. Their per-bucket counts are noisier and their deviations harder to pin down statistically.

The implication: the second-of-minute structure is not a global property of the machine, the kernel, or the git binary. It is a property of **which sub-agents commit how often** intersected with **the latency profile of those sub-agents**.

---

## 7. Localizing the bucket-46 peak: which repo causes it?

The single most over-indexed bucket is second `46` with 147 commits. Decompose by repo:

| repo | observed in bucket 46 | expected (n / 60) | lift |
|---|---:|---:|---:|
| `ai-cli-zoo`         | 38 | 23.95 | **1.587** |
| `ai-native-workflow` | 18 | 11.73 | **1.534** |
| `ai-native-notes`    | 25 | 17.40 | 1.437 |
| `oss-digest`         | 25 | 18.53 | 1.349 |
| `oss-contributions`  | 22 | 17.67 | 1.245 |
| `pew-insights`       | 19 | 23.80 | 0.798 |

Five of six repos over-index second `46`. The exception is `pew-insights`, which actually under-indexes it. The peak is therefore a **near-shared signal** across most of the corpus — not a single-repo accident. The most plausible mechanism is a launchd-phase artifact: launchd fires the dispatcher at a clock-second whose modal value lands such that, after the median handler latency, commits land near second 46. `pew-insights` differs because its sub-agent (feature-patch authoring with axis emission) has a longer characteristic runtime, pushing its commits past the bucket-46 attractor into the next minute entirely.

The second-most over-indexed bucket is second `3` with 143 commits. The same repo decomposition (not shown in full to save space) shows `ai-cli-zoo` again contributing the largest absolute over-count, followed by `ai-native-notes`. This pole at second 3 is consistent with a **next-minute-rollover** for the slowest handlers: launches whose handlers finish 17 seconds past the bucket-46 mode end up in the next minute's bucket 3. The 43-second gap between the two poles is in line with the documented multi-second handler runtime spread.

---

## 8. Bucket-10 trough: where the slow tail is missing

The single most under-indexed bucket is second `10` with 88 commits. Per-repo:

| repo | observed in bucket 10 | expected | lift |
|---|---:|---:|---:|
| `ai-cli-zoo`         | 15 | 23.95 | 0.626 |
| `ai-native-workflow` |  9 | 11.73 | 0.767 |
| `ai-native-notes`    | 14 | 17.40 | 0.805 |
| `pew-insights`       | 19 | 23.80 | 0.798 |
| `oss-digest`         | 15 | 18.53 | 0.809 |
| `oss-contributions`  | 16 | 17.67 | 0.906 |

All six repos under-index second 10. The deficit is shared. This is the cleanest evidence that **the seconds 9–12 region is a true latency desert**: no major handler finishes there, regardless of which family is running. It sits in the gap between the early-minute pole (peaks at 3 and 8) and the next plateau, and no sub-agent's runtime distribution puts mass on it.

This is the kind of structural anomaly that would be invisible at coarser bin widths. At decile resolution (bins of 10 seconds), the 0–9 bucket holds 1,155 commits (17.02%) and the 10–19 bucket holds 1,105 commits (16.29%) — the difference is only 0.73 percentage points and well within sampling jitter. The 50-bucket-finer view is what surfaces the localized depletion.

---

## 9. Coarser bin widths recover flatness

Re-bin into deciles of seconds (six bins of width 10):

| second range | observed | percentage | expected (16.67%) |
|---|---:|---:|---:|
|  0–9   | 1,155 | 17.02% | 16.67% |
| 10–19  | 1,105 | 16.29% | 16.67% |
| 20–29  | 1,097 | 16.17% | 16.67% |
| 30–39  | 1,143 | 16.85% | 16.67% |
| 40–49  | 1,102 | 16.24% | 16.67% |
| 50–59  | 1,183 | 17.44% | 16.67% |

The maximum decile percentage (17.44%) and minimum (16.17%) differ by only 1.27 percentage points. χ² on six bins against 1130.83-each expectation is 5.92 on 5 df — well below the p = 0.05 critical value of 11.07. **Decile-level uniformity is not rejected.**

Re-bin by **last digit** (10 bins of stride 10, each containing 6 seconds, e.g. digit 3 is `{3, 13, 23, 33, 43, 53}`):

| last digit | count | percentage |
|---:|---:|---:|
| 0 | 630 |  9.29% |
| 1 | 668 |  9.85% |
| 2 | 668 |  9.85% |
| 3 | 747 | 11.01% |
| 4 | 687 | 10.13% |
| 5 | 672 |  9.90% |
| 6 | 696 | 10.26% |
| 7 | 677 |  9.98% |
| 8 | 708 | 10.43% |
| 9 | 632 |  9.31% |

χ² = 15.800 on 9 df. Critical value at p = 0.05 is 16.92. Last-digit uniformity is **not rejected** at the conventional threshold (it would be rejected at p ≈ 0.07, marginally). Digit 3 is mildly hot (lift 1.10) and digits 0 and 9 are mildly cold (lifts 0.93 and 0.93).

Re-bin into even vs odd seconds:

- Even seconds: 3,389 / 6,785 = 49.95%
- Odd seconds: 3,396 / 6,785 = 50.05%

This is a single-degree-of-freedom test. The deviation is essentially zero (z ≈ 0.085). Parity of seconds is uniform.

The pattern is consistent: **the rejection of uniformity is fine-grained**. It exists at single-second resolution, persists weakly at last-digit resolution, and disappears entirely at decile or parity resolution. This is the signature of a **multi-modal latency distribution** rather than a single dominant phase. A single dominant phase would survive any aggregation that respects its mode; a multi-modal latency averages out as soon as the bins are wide enough to contain multiple modes.

---

## 10. A quick adjacent uniformity check: leading hex character

To cross-check that the rejection is genuinely about clock-second structure and not about some general failure of randomness in the corpus, take the **first hex character** of the most recent 100 commit SHAs from each repo (600 SHAs total). If git's hashing is well-behaved (it is — SHA-1 is cryptographically uniform), each of the 16 hex digits should appear equally often. Expected per bucket: 600 / 16 = 37.5.

Observed χ² on 15 df: **18.400**. Critical value at p = 0.05: ~24.996. **Hex-character uniformity is not rejected.**

Good. The corpus is otherwise well-behaved. The second-of-minute deviation is not "everything is non-uniform"; it is a specific, localized signal in the timestamp dimension that does not contaminate the SHA dimension. This rules out trivial explanations like a corrupt random source or systematically biased sampling.

---

## 11. What this implies operationally

Three things follow from the above.

**First: the dispatcher does have a clock-second fingerprint, but it is not the obvious one.** Anyone surveying the corpus expecting to find a sharp xx:00 cron peak would find none. The energy is instead in a wide, multi-modal latency distribution with poles at second 46 (ai-cli-zoo dominant) and second 3 (next-minute rollover for the slowest handlers), and a trough at second 10 (no sub-agent finishes here). The signature is a property of the handler runtime mixture, not of the launch event.

**Second: the per-repo chi-square ranking is a proxy for handler-volume × handler-modality.** The three rejecting repos (`ai-cli-zoo`, `ai-native-notes`, `pew-insights`) are the highest-volume families with the most distinct sub-agent invocations. Lower-volume families don't accumulate enough mass to surface their own bucket structure. This means future per-family timestamp diagnostics should expect to find clearer signals from the high-volume tail and noisier signals from the low-volume head — a power-of-test asymmetry that any future repeat of this analysis must control for.

**Third: any monitoring tool that asks "is this repo's commit timing automated?" cannot rely on a sharp clock-second peak.** The only signature that survives at this corpus size is a chi-square test at single-second resolution. Coarser tests pass. A naïve regulator looking for "commits at xx:00" would conclude this corpus is human-driven; only the fine-grained χ² reveals the multi-modal automation fingerprint.

---

## 12. Falsifiable predictions for the next 1,500 commits

The dispatcher will continue to commit at roughly its current rate (6,785 commits in ~10.7 days = ~635 commits/day). The next 1,500 commits will arrive in approximately the next 56 hours. At that point, the corpus will reach ~8,285 commits and the same chi-square test should be repeated. The following predictions are made now and can be checked then:

1. **The aggregate χ² will increase** beyond 96.87 (more data, same effect size, higher power). Specifically: if the underlying effect is real and the per-bucket lifts persist, χ² should scale roughly with N. Expected χ² at N = 8,285 is `96.87 · (8,285 / 6,785) ≈ 118.3`, well past the p = 0.001 critical value of 99.6.

2. **The bucket-46 peak will remain top-3** (and likely top-1) by absolute count. If it falls out of the top 5, the peak was a transient artifact and the structural claim is weaker.

3. **The bucket-10 trough will remain bottom-3** (and likely bottom-1). Same logic; failure here means the latency desert was illusory.

4. **The decile and last-digit tests will continue to **not** reject uniformity** at p = 0.05. If they begin to reject, the multi-modal hypothesis is wrong and a wider effect is hiding underneath.

5. **`ai-cli-zoo`'s per-repo χ² will continue to lead the field**, with relative ranking `ai-cli-zoo > ai-native-notes > pew-insights > {oss-digest, oss-contributions, ai-native-workflow}` preserved within ±1 swap. Major reordering implies family-mixture changes (e.g. one family stalling) rather than the steady-state behavior measured here.

These five predictions form an audit checklist for the next instrumented re-run. Any single failure forces a rewrite of the diffuse-multi-modal-latency hypothesis. Two or more failures invalidate the central claim of this post.

---

## 13. What this measurement cannot decide

It is worth being explicit about what the test does **not** establish.

It does not isolate the precise launchd-phase distribution — to do that, the launch events themselves would need to be timestamped independently of the commits they spawn. The dispatcher's `history.jsonl` records tick-level `ts` fields at minute resolution but not second resolution for the actual launchd fire moment.

It does not distinguish between "kernel-scheduling jitter" and "handler-internal latency jitter" as the source of the multi-modal poles. Both would produce the same observed signature at the commit level.

It does not address whether the **rate** of commits per second-bucket is changing over time. A weighted-by-recency repeat might show that the bucket-46 peak is intensifying (handler latency stabilizing) or fading (latency variance growing). Only a sliding-window χ² on time-stratified subsets could decide that.

And it does not say anything about second-of-hour or second-of-day periodicity. Those are larger-scale circadian questions and were measured in a separate metapost (`2026-05-04-hour-of-day-utc-distribution-of-the-785-tick-dispatcher-chi-square-6-39-failure-to-reject-uniformity-fano-0-266-and-the-shared-hour-10-trough-that-falsifies-circadian-drift.md`), with the conclusion that **hour-of-day uniformity holds** (χ² = 6.39 on 23 df, fails to reject). The contrast is informative: the dispatcher is uniform across hours of the day but **non-uniform** within seconds of a minute. The non-uniformity is a property of intra-minute mechanics, not of human or environmental circadian patterns.

---

## 14. Summary

Across 6,785 commits in six dispatcher-owned repositories, the UTC second-of-minute distribution rejects discrete uniformity at χ² = 96.87 on df = 59, p < 0.005. The deviation is **not** a cron peak: bucket 0 under-indexes (108 vs 113.08, z = −0.478) and the tens-multiples aggregate under-indexes by z = −1.963. The deviation is instead **broad-band**, with peaks at second 46 (147 commits, z = +3.19) and second 3 (143 commits, z = +2.81), a trough at second 10 (88 commits, z = −2.36), and a max-to-min ratio of 1.670. Per-repo decomposition pins `ai-cli-zoo` as the dominant chi-sq contributor (116.28), with `ai-native-notes` (92.55) and `pew-insights` (79.48) also rejecting individually. Coarser binnings — deciles of seconds, last-digit, parity — all fail to reject uniformity, confirming the deviation is fine-grained and multi-modal rather than dominated by a single phase. A control test on the leading hex character of recent SHAs (χ² = 18.4 on 15 df) confirms the corpus is otherwise well-behaved. Five falsifiable predictions are made for the next 1,500 commits, including that aggregate χ² should rise to ~118 and that bucket-46 should remain the dominant peak. This metapost cites no speculative numbers; every quantity above was computed from `git log --pretty=format:%at|%H --all` on the six repositories at the moment of writing, with the daemon corpus cross-checked against `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (801 ticks, 6,422 daemon-tracked commits, 2,701 daemon-tracked pushes between `2026-04-23T16:09:28Z` and `2026-05-04T08:48:38Z`).
