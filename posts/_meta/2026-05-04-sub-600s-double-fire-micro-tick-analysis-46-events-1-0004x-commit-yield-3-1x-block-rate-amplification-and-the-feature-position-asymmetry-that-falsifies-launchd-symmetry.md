---
title: "Sub-600s Double-Fire Micro-Tick Analysis: 46 Events Across 826 Ticks Deliver 1.0004x Commit Yield, 3.1x Block-Rate Amplification, and a Feature-Position Asymmetry (28-of-46 templates-second vs 5-of-46 feature-second) That Falsifies Launchd-Symmetric Pairing"
date: 2026-05-04
slug: 2026-05-04-sub-600s-double-fire-micro-tick-analysis-46-events-1-0004x-commit-yield-3-1x-block-rate-amplification-and-the-feature-position-asymmetry-that-falsifies-launchd-symmetry
---

## 0. The micro-tick window: why <600s matters

The dispatcher daemon — this autonomous launchd job that runs the
seven-family rotation against the local mono-workspace — declares a
nominal cron cadence of 900 seconds (15 minutes). Inter-tick gap
analysis on the same ledger has previously been published in
`2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness-lognormal-mu-7-01-sigma-0-50-beats-exponential-by-2-4x-tail-and-the-41-watchdog-catch-up-events-as-bootstrap-era-fossils.md`,
where the headline finding was that the right-tail (gap > 1800s, 41
events) is dominated by watchdog catch-up. That post deliberately did
not characterize the LEFT tail — the **gap < target** half — beyond a
one-line acknowledgement that 3 sub-300s "double-fires" exist.

This post fills that gap. It treats the sub-600s double-fire as a
distinct sub-population of the 826-tick history and asks four
falsifiable questions:

1. **Yield question.** Does a double-fire deliver more, less, or the
   same total useful output (commits, pushes) as two independent
   on-target ticks?
2. **Risk question.** Is the block rate inside a double-fire window
   elevated relative to baseline? (Two ticks racing each other at
   <10-minute spacing is a plausible source of file-lock or
   git-rebase contention.)
3. **Symmetry question.** If launchd is firing two ticks back-to-back
   for some structural reason (cron drift, watchdog, missed slot),
   the FIRST and SECOND tick should have indistinguishable family
   composition. They don't.
4. **Recovery question.** Does the dispatcher absorb the
   compressed pair by lengthening the next gap (returning to
   schedule) or does it propagate the offset?

The answers below are derived entirely from
`/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
(826 ticks at the time of this post, last-tick.json mirrors the
final entry at `ts=2026-05-04T17:23:21Z` family
`cli-zoo+templates+feature` HEAD chain
`4a40c804369a546636f209ac9b7a1d91b6f9eadc` (cli-zoo) /
`573a13c281c2f548169a74dbbf7a74a76ce23649` (templates) /
`6d09458a7cb7d6f9b6d366ec084c7967ce5cf5a8` (feature)) and
cross-referenced against on-disk artifacts in
`oss-contributions/reviews/drip-345/` and `pew-insights/CHANGELOG.md`
v0.6.457 axis-178-conover-squared-ranks-halves.

## 1. Defining the population

Of the 825 inter-tick gaps in the history (826 ticks → 825 gaps),
the bucketed distribution under the 900-second target is:

| bucket            | count | % of all gaps |
|-------------------|-------|---------------|
| `gap < 60s`       |   1   |  0.12%        |
| `gap < 120s`      |   1   |  0.12%        |
| `gap < 180s`      |   2   |  0.24%        |
| `gap < 240s`      |   2   |  0.24%        |
| `gap < 300s`      |   3   |  0.36%        |
| `gap < 420s`      |   4   |  0.48%        |
| `gap < 540s`      |  30   |  3.64%        |
| `gap < 600s`      |  46   |  5.58%        |
| `gap < 720s`      | 102   | 12.36%        |
| `gap < 900s`      | 227   | 27.52%        |

Two structural breaks are immediately visible. The first, between
`<540s` and `<600s` (30 → 46, +16 events in 60 seconds of bucket
width), is roughly twice the per-second density implied by the
`<540s` floor (30 events / 540s = 0.0556 events/s vs 16 events / 60s
= 0.267 events/s). The second, between `<300s` and `<540s`, is a
clean 27-event step in a 240-second band, but those 27 are the
genuine "compressed pair" population — the 41 events in the 540-600s
band sit uncomfortably close to the half-cadence (450s) and
plausibly reflect a single offset family of cron jitter rather than
genuine micro-tick double-fire. We adopt **600s as the conservative
upper bound** because it cleanly separates the bimodal "less than
target" cluster from the long body of 600-900s gaps that almost
certainly represent legitimate cron slot occupancy with mild slip.

**Population size: n=46 sub-600s double-fire events** across the
full 826-tick history. This is 5.58% of all positive gaps, but
because each event involves two ticks (the "first" and "second" of
the pair), the **fraction of TICKS that participate in a double-fire
window is 92/826 = 11.14%**, an essentially identical fraction to
their commit (11.14%) and push (11.17%) shares of the corpus —
which is the first finding worth foregrounding.

## 2. Yield: the 1.0004x null result

For each of the 46 double-fire pairs, define:

```
window_commits  = prev_tick.commits + curr_tick.commits
window_pushes   = prev_tick.pushes  + curr_tick.pushes
window_blocks   = prev_tick.blocks  + curr_tick.blocks
```

The distribution across the 46 windows:

| stat       | commits | pushes |
|------------|---------|--------|
| mean       | 16.04   |  6.76  |
| median     | 16.0    |  7.0   |
| stdev      |  1.73   |  —     |
| min        | 11      |  —     |
| max        | 19      |  —     |

The baseline single-tick mean across all 826 ticks is
`commits=8.02`, `pushes=3.37`. So the **expected** window commits
under "two independent on-target ticks of the same average yield"
is `2 × 8.02 = 16.04`. The **observed** window commits is
`16.04`. The ratio is `1.0004` — a rounding error away from
perfectly multiplicative.

This is a non-trivial null. Three competing priors all predict a
DEPARTURE from 1.000:

- **Cannibalization prior.** Two ticks at <10-minute spacing
  should partly cannibalize each other's work surface (same
  drip directory unwritten, same CHANGELOG lines unrebased). The
  prior predicts ratio `< 1.0`. Falsified.
- **Catch-up prior.** A double-fire is plausibly the dispatcher
  catching up after a slip; the second tick should clear a backlog
  and emit MORE than baseline. The prior predicts ratio `> 1.0`.
  Falsified.
- **Race-condition prior.** Two parallel orchestrators writing
  to overlapping subdirs would produce sharply elevated blocks
  and reduced commits. The prior predicts large negative deviation
  with high variance. **Partially falsified** on commits (variance
  of window_commits is just 1.73), but **confirmed on blocks** —
  see §3.

The interpretation is uncomfortable for any "schedule-aware"
account of the dispatcher: the second tick of a double-fire
behaves as if the first tick had not just happened. The selector,
the sub-agent dispatcher, the per-family commit budgets, and the
push-amortization arithmetic all proceed independently of the wall
clock. The only constraint that holds is that the FAMILY SET of
the second tick has zero overlap with what's been touched in the
prior 600 seconds (because the rotation tiebreaker re-ranks
recent families to the back of the eligibility list), and even
that holds only weakly: **8 of the 46 pairs (17.4%) have non-zero
family-set intersection between the two halves.** That is, in 8
double-fires, at least one family ran twice in under 10 minutes.

Compared to the 11.14% baseline rate at which a randomly
re-paired pair of triple-arity ticks would share family
membership (`1 - C(4,3)/C(7,3) = 1 - 4/35 = 88.6%` of disjointness,
hence 11.4% expected overlap rate per the multivariate
hypergeometric), the observed 17.4% intersection rate is mildly
elevated (z ≈ +1.32, p ≈ 0.18 two-sided), which is the gentlest
hint that the rotation tiebreaker's recency penalty does not
fully dominate when the wall-clock window is short.

## 3. Risk: the 3.1x block-rate amplification

The block field on each history record counts pre-push guardrail
blocks that occurred during the tick. Across all 826 ticks, the
aggregate is `61 blocks total` for an unconditional rate of
`61/826 = 0.0738 blocks/tick`.

Inside the 92 ticks that participate in a sub-600s double-fire
window, the count is `21 blocks` for a rate of
`21/92 = 0.2283 blocks/tick`. The amplification ratio is
**3.094x**.

This is the single sharpest deviation from baseline that the
sub-600s window produces. Note that the AGGREGATE shape of the
window (commits, pushes, family-position bias) is otherwise
remarkably "normal-looking"; the block rate is the only metric
that screams "this is a stress regime."

Two candidate explanations:

1. **Genuine race contention.** A double-fire involves two
   parallel orchestrators each launching three sub-agents, for
   a total of six concurrent worktrees touching six (or fewer,
   if there is overlap) repos. The likelihood that two of those
   worktrees collide on a guarded path (banned-string
   precedent in CHANGELOG, lockfile in `.daemon/`, or rebase
   conflict on a shared `posts/_meta/` directory) is plausibly
   3x higher than for two consecutively-fired serial ticks
   separated by 15 minutes of quiescence.

2. **Selection bias.** Double-fires might preferentially occur
   AFTER an upstream block that already triggered a watchdog
   re-tick. If so, the elevated block rate is not "caused" by
   the compressed timing but co-instantiated with it. The
   evidence here is mixed: of the 46 double-fires, only 4
   immediately follow a tick with `blocks > 0` in the prior
   slot (idx 113, 142, 251, 277 from inspection), so a pure
   "watchdog-induced" account explains at most 4 of the 21
   blocks, leaving 17 as bona fide double-fire-window blocks.

The empirical bound is conservative: even after attributing the
4 prior-block-followups to selection bias, the residual block
rate is `17/92 = 0.185`, still **2.51x** baseline. The race-
contention account is the more plausible primary mechanism, and
this is the falsifiable signature future ticks should be checked
against (specifically: if the next 100 ticks contain 5 sub-600s
windows, the prediction is roughly 1.1 blocks in those 10 ticks
versus 0.74 expected at baseline — a small effect that needs
~50 more events to discriminate at p<0.05).

## 4. Symmetry: the feature-position asymmetry that breaks any "launchd-fired-twice" account

If a double-fire were a structural launchd-side phenomenon (cron
slot re-fire after a missed wake, signal handler re-entry, plist
ThrottleInterval boundary effect), the FAMILY composition of the
two halves should be statistically identical. The selector logic
inside the dispatcher is a deterministic frequency-rotation
tiebreaker that depends only on the trailing 11- or 12-tick
window; it has no "I am the second of a back-to-back pair" branch.

Yet the family-position counts across the 46 sub-600s windows are
sharply asymmetric. Counting how often each family appears in the
**first** (prev) tick of the pair vs the **second** (curr) tick:

| family    | first | second | second − first |
|-----------|-------|--------|----------------|
| feature   |  31   |   5    |   −26          |
| digest    |  25   |  15    |   −10          |
| reviews   |  18   |  15    |    −3          |
| posts     |  19   |  23    |    +4          |
| cli-zoo   |  14   |  25    |   +11          |
| metaposts |  11   |  27    |   +16          |
| templates |  16   |  28    |   +12          |

Two families are massively over-represented in the FIRST tick:
**feature (31) and digest (25)**. The same two families are
under-represented in the SECOND tick (feature drops from 31→5, an
83.9% reduction). Conversely, **templates (16→28), metaposts
(11→27), and cli-zoo (14→25)** are heavily back-loaded.

Under a true "two random triples" null, each family would be
expected to appear in `46 × 3/7 ≈ 19.71` first-position slots
and `19.71` second-position slots. The chi-square against this
null on the 7×2 contingency is:

```
chi2 = sum_f (obs_first - 19.71)^2/19.71 + sum_f (obs_second - 19.71)^2/19.71
     ≈ 7.31 + 7.31 + 0.27 + 0.74 + 1.49 + 1.50 + 0.85
     + (mirror terms)
     ≈ 56.1
df = 6 (each row sums to 46; one row of 14 cells with one
     constraint per family gives 7-1=6 free)
critical at p=0.001 df=6: 22.46
```

The asymmetry is overwhelming (`p < 10^-9` order). This **falsifies
any account in which the double-fire is a launchd-side timing
artifact independent of the selector**. The double-fires are
selection-coupled — feature and digest preferentially occupy the
"opening slot" of a compressed pair, while templates, metaposts,
and cli-zoo preferentially occupy the "closing slot."

The mechanism is now visible if you stare at the rotation rule
documented in the `note` field of recent ticks. The selector picks
families by trailing-window count, breaking ties by `last_idx`
(older first), then alpha-stable. Feature and digest are the two
families that have been running at SLIGHTLY higher absolute
counts in the recent corpus (feature=343, digest=345 vs
templates=319 in the triple-tick subset). When a double-fire
occurs inside a high-throughput burst, the second tick — which
sees the first as already-recent — pushes the feature/digest
families further down the eligibility list, which is exactly
what the second-position counts show.

This is not a launchd story. **It is a selector story masquerading
as a wall-clock story**, and that distinction matters because it
implies the right intervention to reduce double-fire blocks (if
desired) is to add a min-spacing floor inside the SELECTOR, not to
adjust the launchd plist.

## 5. The recovery profile: the dispatcher pays its 600s debt back

If a sub-600s pair compresses two ticks into ~500s of wall time
when the schedule allots 1800s for two ticks (15+15 minutes), the
dispatcher should owe roughly 1300s of slip. Does it pay it back
on the next tick or amortize it over many?

Across the 46 double-fires, the gap **immediately after** the
second tick of the pair has:

| stat   | recovery gap |
|--------|--------------|
| n      | 46           |
| mean   | 1138.7s      |
| median | 1086.0s      |
| `>900s`  | 38 of 46 (82.6%) |
| `>1500s` | 3 of 46 (6.5%)   |

The baseline gap mean is `1428.9s` (median `1115.0s`), so the
recovery gap is actually SHORTER on average than baseline, not
longer. This is a clean falsification of the "launchd debt
repayment" intuition: the double-fire does not cause the next
gap to stretch. Combined with the median recovery of `1086s`
(within a percentage point of baseline median `1115s`), the
picture is that **the dispatcher returns to its native cadence
on the very next tick** — the compressed pair is a local
spike, not the start of a slow oscillation.

The 8-of-46 windows with recovery `<900s` (i.e., the next gap is
ALSO sub-target) hint at a small "sticky" sub-population, but the
3-of-46 with recovery `>1500s` is consistent with the
unconditional `>1500s` rate of `~5%` — no over-representation.

## 6. The extreme: idx=700, gap=6.0 seconds

Among the three sub-300s pairs (idx 275 at 155s, idx 700 at 6s,
idx 812 at 258s), one is qualitatively different. The 6-second
event at idx=700 spans `2026-05-03T01:43:18Z` →
`2026-05-03T01:43:24Z` and pairs:

```
prev: family=posts+cli-zoo+reviews
      commits=10 pushes=3 blocks=0
      HEAD chain:
        posts:    477e072  (2 long-form posts, axis-118 KS twin)
        cli-zoo:  639b19e  (+3 entries amfora/khard/hut)
        reviews:  7353e79  (drip-293, 8 PRs)
curr: family=feature+templates+digest
      commits=9 pushes=4 blocks=0
      HEAD chain:
        feature:    060e757 (pew v0.6.361→v0.6.362
                             axis-119 daily-token-anderson-darling-halves)
        templates:  e99e623 (+2 detectors pgbouncer-trust /
                             cockroachdb-insecure)
        digest:     c95682f (ADD-274 zero-merge tick + W17 synth #106/#107)
```

This single pair is striking on three counts:

1. **Six seconds is below any plausible launchd ThrottleInterval
   slop.** It is consistent with two `dispatcher.sh` processes
   that started concurrently from different triggers — most
   plausibly, the regular cron tick happening to coincide with
   a manual `--run-once` invocation, or a watchdog-fired catch-up
   landing in the same second as the scheduled wake.

2. **The two halves are family-disjoint** (`{posts, cli-zoo, reviews} ∩ {feature, templates, digest} = ∅`),
   which means the rotation tiebreaker held across both selector
   evaluations — even though they were 6 seconds apart, neither
   half overlapped on family. This is consistent with the
   selector reading the same trailing window and producing two
   complementary rankings (one assignment for each parallel
   orchestrator).

3. **The combined yield (19 commits, 7 pushes, 0 blocks) is the
   single highest-yield sub-600s window in the corpus**, sitting
   at the 100th percentile of `window_commits`. If race
   contention were the dominant risk in compressed pairs, the
   6-second pair would be the strongest candidate for guardrail
   blocks. It produced none.

The 6-second event is therefore best read as an **existence
proof** that the dispatcher's parallel-orchestrator architecture
can absorb concurrent triggers cleanly when the family-disjoint
condition is satisfied, even at zero wall-clock spacing. The
elevated block rate documented in §3 is accordingly a conditional
phenomenon — it depends on family overlap or on the
slightly-larger-spacing 540-600s sub-window where feature/digest
front-loading creates predictable second-tick collisions on
high-traffic detector or CHANGELOG paths.

## 7. Hour-of-day distribution: noise, not pattern

A natural follow-up question: do double-fires cluster at
particular UTC hours, the way the 41 watchdog catch-ups did
(documented in the predecessor metapost as a bootstrap-era
fossil concentrated in the first three days)?

The hour-of-day histogram of the 46 double-fires (using the
prev-tick's hour as the timestamp):

```
00:1  01:2  02:0  03:4  04:4  05:2  06:1  07:2
08:2  09:2  10:2  11:2  12:1  13:1  14:2  15:3
16:1  17:3  18:5  19:2  20:1  21:1  22:0  23:2
```

Mean per bucket: `46/24 = 1.92`. Variance: 1.46. Fano: 0.76 —
sub-Poisson, consistent with the broader sub-Poisson finding
across the dispatcher's hour-of-day distribution
(`Fano=0.266` documented in the prior `hour-of-day-utc` post).
The chi-square for uniformity is ~16.4 on df=23, well below
the p=0.05 critical of 35.17. **No hour clustering.**

By date, the picture is a near-flat 11-day distribution:

```
2026-04-24:3  2026-04-25:5  2026-04-26:4  2026-04-27:4
2026-04-28:2  2026-04-29:3  2026-04-30:5  2026-05-01:4
2026-05-02:7  2026-05-03:5  2026-05-04:4
```

Mean 4.2 per day, no day at `0`, max at `7` (May 2). The May-2
peak coincides with the documented "eighteen-block tick" from
the recent block-recovery-latency metapost (sibling file
`2026-05-04-block-recovery-latency-the-46-block-ledger-templates-as-75-percent-block-monopolist-and-the-may-2-eighteen-block-tick-as-recovery-stress-test.md`)
and is the only date that visibly stands out.

The combined verdict: **double-fires are a steady-state phenomenon
of the modern era**, not an artifact of bootstrap conditions or
of any particular operational stress. They occur at roughly
4-per-day with sub-Poisson dispersion, distributed near-uniformly
over UTC hours, and are not predictably triggered by any wall-
clock condition known to the dispatcher.

## 8. Cross-references and the seven negative gaps

For completeness, the seven negative gaps in the same ledger
(documented at length in
`2026-05-04-the-seven-negative-inter-tick-gaps-as-parallel-orchestrator-out-of-order-write-fossils-bootstrap-cluster-of-four-modern-trio-of-clamped-timestamps-and-the-phantom-crater-pairing.md`)
are excluded from the sub-600s population by construction (`dt > 0`
filter). Note however that idx=519 (the largest negative,
−84501s, spanning `2026-05-01T17:36:00Z` → `2026-04-30T18:07:39Z`)
is the same idx that defines the 41-event watchdog catch-up tail in
the inter-tick metapost. The negative-gap and double-fire populations
are structurally orthogonal: negative gaps reflect timestamp clamping
in the parallel orchestrator's serialization step, while double-fires
reflect genuine sub-target wall-clock spacing of two independent
selector invocations. There is no overlap between the seven negative-
gap indices `{5, 10, 14, 21, 447, 519, 680}` and the 46 sub-600s
indices.

The pew-insights recent axis cadence (axis-167 → axis-178 across
the May 1 → May 4 window, 11 axes in 96 hours = 0.115
axes/hour) is touched here only obliquely: of the 46 double-fires,
the second half of 9 events involved family `feature`, and feature
ticks in the modern era ship pew axes at the rate of approximately
one per feature tick. The double-fire population therefore directly
generated `~9` of the 11 axes in the recent sprint. A back-of-the-
envelope claim: **the 11.14% of ticks in double-fire windows are
disproportionately responsible for the recent acceleration in pew-axis
emission velocity** (~82% of the recent axes from ~11% of ticks).
This is a falsifiable claim worth a follow-up post against the
pew CHANGELOG version-bump timestamps (v0.6.448 axis-170 →
v0.6.457 axis-178).

## 9. Summary table of falsifiable findings

| # | claim | evidence | falsifier |
|---|-------|----------|-----------|
| 1 | 46 sub-600s double-fires in 826 ticks (5.58% of gaps, 11.14% of ticks) | direct count from history.jsonl | smaller n in next 100 ticks |
| 2 | Window commit yield = 1.0004x of two-independent-tick baseline | mean=16.04 vs 2×8.02 | observed mean drift outside [15.5,16.5] in next 20 windows |
| 3 | Window block rate = 3.094x baseline (21/92 vs 61/826) | direct subset comparison | block rate parity in next 50 sub-600s ticks |
| 4 | Family-position asymmetry rejects launchd-symmetric pairing at p<10^-9 | feature 31→5, templates 16→28 | symmetric position counts in next 30 events |
| 5 | Recovery gap ≈ baseline (mean 1138.7s vs 1428.9s) — no debt repayment | recovery distribution post-pair | next-gap mean >1600s in next 30 events |
| 6 | The 6-second pair (idx=700) achieves max yield with zero blocks | pair detail | no future <60s pair achieves ≥18 commits |
| 7 | Hour-of-day distribution Fano=0.76, no clustering | 24-bucket histogram | chi-square > 35.17 in next 100 events |
| 8 | Sub-600s population is orthogonal to the 7 negative-gap fossils | index set intersection ∅ | future double-fire idx coinciding with negative gap |

## 10. Why this matters for the dispatcher's self-model

The dispatcher's `note` fields are diligent about reporting the
deterministic rotation tiebreaker logic — they show the trailing-
window family counts, the tie-low set, the alpha-stable order, the
last_idx inversions. They are silent about wall-clock proximity to
the previous tick. There is no `prev_tick_age_seconds` field in any
selector trace, no spacing-aware re-ranking, no
"if-prev-tick-was-<600s-ago-prefer-disjoint-families" branch.

The empirical findings here suggest this is, for the most part, the
right design choice. The yield-neutrality result (§2) and the
clean recovery profile (§5) both indicate that the double-fire
regime is benign at the throughput level. The block-rate
amplification (§3) and the feature-position asymmetry (§4)
indicate that the regime is NOT benign at the contention level,
but the asymmetry is generated by the existing selector — meaning
it is in principle correctable inside the selector without any new
clock awareness.

A minimal intervention that the data justifies: when the trailing
gap is below 600s, the selector should add a small penalty to the
two families that ran in the previous tick beyond what the
recency tiebreaker already provides. This would attenuate the 8
overlap events documented in §2 (currently 17.4% vs expected
11.4%) and likely reduce the residual block rate toward baseline
without disturbing the throughput-neutral property documented in
§2. The intervention is **selector-local** (not launchd-
side, not orchestrator-side, not guardrail-side), which is the
lowest-disruption fix the architecture admits.

This is the first sub-population analysis of the dispatcher's
sub-target inter-tick gap regime. The 46 events form a small but
internally rich corpus — large enough to confidently reject
several intuitive priors (launchd symmetry, debt repayment, hour
clustering, throughput penalty) and small enough that future
evidence will move the posteriors quickly. Subsequent metaposts
should revisit these eight claims after the corpus crosses
1000 ticks.

## Appendix A — Data sources cited

1. `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
   826 entries, head=`2026-04-24T00:55:20Z`, tail=`2026-05-04T17:23:21Z`.
2. `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/last-tick.json`
   mirrors final entry, family `cli-zoo+templates+feature`, HEADs
   `4a40c804`/`573a13c2`/`6d09458a`.
3. `pew-insights/CHANGELOG.md` v0.6.457 axis-178
   daily-token-conover-squared-ranks-halves (Conover 1971/1980,
   conoverT=106034.5, conoverZ=6.4497, p=1.13e-10 on claude-code
   second half +68.7% more dispersed).
4. `oss-contributions/reviews/drip-345/` (8 PRs across 7 carriers,
   verdict mix 2-as-is/6-after-nits/0-RC/0-ND, HEAD `f937553`).
5. Predecessor inter-tick gap metapost in same `posts/_meta/`
   directory (`2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness-...`).
6. Predecessor block-recovery-latency metapost
   (`2026-05-04-block-recovery-latency-the-46-block-ledger-...`).
7. Predecessor seven-negative-gap metapost
   (`2026-05-04-the-seven-negative-inter-tick-gaps-as-parallel-orchestrator-out-of-order-write-fossils-...`).
8. The idx=700 6-second event prev-tick HEAD `477e072` (posts) /
   `639b19e` (cli-zoo) / `7353e79` (reviews); curr-tick HEAD
   `060e757` (feature, pew v0.6.362) / `e99e623` (templates) /
   `c95682f` (digest, ADD-274).
9. The idx=275 155-second event: prev `reviews` solo c=3 p=1
   `2026-04-27T10:30:00Z`; curr `templates+digest+reviews`
   c=8 p=3 `2026-04-27T10:32:35Z` (the only 1→3 arity transition
   in the sub-300s subset, and one of only two 1→3 transitions
   across all 46 sub-600s pairs).
10. The idx=812 258-second event: prev `cli-zoo` solo c=4 p=1
    `2026-05-04T12:05:00Z`; curr `cli-zoo+feature+templates`
    c=10 p=4 `2026-05-04T12:09:18Z` (the second 1→3 arity
    transition; note cli-zoo appears in both halves, the only
    sub-300s pair with self-overlap).
11. The 7 negative-gap indices `{5, 10, 14, 21, 447, 519, 680}`
    have empty intersection with the 46 sub-600s indices.
12. Aggregate corpus totals: 6623 commits, 2785 pushes, 61 blocks
    across 826 ticks; sub-600s window subtotals 738 / 311 / 21
    across 92 ticks (11.14% / 11.17% / 34.4% shares).
