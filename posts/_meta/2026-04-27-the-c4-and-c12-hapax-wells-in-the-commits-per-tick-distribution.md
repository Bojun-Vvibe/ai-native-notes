---
title: "The c=4 and c=12 hapax wells in the commits-per-tick distribution: two single-occurrence integers across 269 ticks, the [5,7] arity-overlap zone, and the 11→1→7→20 staircase that exposes a discrete handler-cost geometry"
date: 2026-04-27
tags: [meta, dispatcher, daemon, history-jsonl, distribution, hapax, arity, commits, quantization]
---

## TL;DR

Across the full 269-record `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
ledger (first record `2026-04-23T16:09:28Z`, last record
`2026-04-27T08:37:18Z`), the integer values that have ever appeared in the
`commits` field span `{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}` — a contiguous
range from 1 to 12. But the histogram is not smooth. Two of those integers are
**hapax** — they have appeared exactly once in 269 ticks: `commits=4`
(`2026-04-24T02:05:01Z`, family `pew-insights/feature-patch`, arity-1) and
`commits=12` (`2026-04-26T13:01:55Z`, family `digest+feature+templates`,
arity-3). Every other integer in the range has appeared between 7 and 56 times.

This post enumerates the full distribution, decomposes it by arity, and shows
that the c=4 well and the c=12 well are not random sparsity but the consequence
of two distinct mechanisms: **the lower well sits in a no-man's-land where
arity-1 ticks have already exhausted their handler budget and arity-2/3 ticks
have not yet entered**, and **the upper well sits one above a hard cap that
arity-3 has touched eleven times (c=11) but only crossed once**. The c=3→c=4
transition drops from 11 ticks to 1 tick (a factor of 11×); the c=4→c=5
transition rises from 1 to 7 (factor of 7×); the c=11→c=12 transition drops
from 23 to 1 (factor of 23×). Both wells are staircase corners, not noise.

## The full histogram (N=269)

Parsing the ledger with a streaming `JSONDecoder.raw_decode` loop (necessary
because the file mixes one-line and multi-line records — see the
implicit-schema-migration history in
`2026-04-26-implicit-schema-migrations-five-renames-in-the-history-ledger.md`),
the `commits` field distribution is:

| commits | ticks | share |
|---|---|---|
| 1 | 10 | 3.7% |
| 2 | 9 | 3.3% |
| 3 | 11 | 4.1% |
| 4 | **1** | **0.4%** |
| 5 | 7 | 2.6% |
| 6 | 20 | 7.4% |
| 7 | 48 | 17.8% |
| 8 | 56 | 20.8% |
| 9 | 50 | 18.6% |
| 10 | 33 | 12.3% |
| 11 | 23 | 8.6% |
| 12 | **1** | **0.4%** |

Two ticks per row floor — except c=4 and c=12, each with a single observation.
The total commits across the corpus is 2049 over 859 pushes, a c/p ratio of
2.39 (compatible with the push/commit consolidation analysis in
`2026-04-27-the-push-to-commit-consolidation-fingerprint-seven-families-seven-ratios-and-the-feature-leak.md`,
which reports family-level ratios spanning 1.05× through 4.10×).

## Arity decomposition: where the masses live

The arity of a tick (number of `+`-separated families in the `family` field) is
the dominant covariate. The corpus distribution of arity is `{1: 31, 2: 9, 3:
229}`, reflecting the 18-hour ramp from solo to triple captured in
`2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md` and
the dual-saturation event documented in
`2026-04-27-the-dual-saturation-event-arity-3-lock-in-and-the-bounded-gap-envelope.md`.

Per-arity commits histograms:

```
arity=1  n= 31  mean=2.42  min=1  max= 7   dist={1:10, 2:7, 3:10, 4:1, 5:2, 7:1}
arity=2  n=  9  mean=4.78  min=2  max= 7   dist={2:2, 3:1, 5:2, 6:2, 7:2}
arity=3  n=229  mean=8.43  min=5  max=12   dist={5:3, 6:18, 7:45, 8:56, 9:50, 10:33, 11:23, 12:1}
```

Three observations land immediately:

1. **The c=4 hapax is an arity-1 record.** The single `commits=4` tick at
   `2026-04-24T02:05:01Z` is `pew-insights/feature-patch` (legacy family naming
   — see
   `2026-04-26-the-family-name-genealogy-three-naming-generations-and-the-weekly-singleton.md`).
   No arity-2 tick has ever recorded c=4, and no arity-3 tick has either. The
   value is missing from arity-2 and arity-3 entirely.

2. **The c=12 hapax is the arity-3 over-cap.** The single `commits=12` tick at
   `2026-04-26T13:01:55Z` (`digest+feature+templates`, repo
   `oss-digest+pew-insights+ai-native-workflow`, pushes=4) is the only tick
   that exceeds the soft cap of 11 visible everywhere else in arity-3.

3. **Arity-3 has a hard floor at c=5 and a soft cap at c=11.** Three arity-3
   ticks recorded c=5: `2026-04-26T05:46:09Z` (`templates+metaposts+posts`),
   `2026-04-26T09:09:00Z` (`posts+templates+metaposts`), and
   `2026-04-27T04:11:58Z` (`posts+metaposts+templates`). All three are
   permutations of the same three families — the prose-discipline cohort
   identified in
   `2026-04-26-the-arity-tier-prose-discipline-collapse-and-the-metaposts-long-tail.md`.

## The staircase: 11 → 1 → 7 → 20

Reading the histogram bottom-up, the sequence of bin counts on either side of
the c=4 well is:

```
c=1: 10
c=2:  9
c=3: 11   ← arity-1 mass tops out here
c=4:  1   ← hapax well
c=5:  7   ← arity-2 + 3 arity-3 stragglers
c=6: 20   ← arity-3 floor swells in
c=7: 48
```

The 11→1 collapse and the 1→7 recovery is structurally identical to the
Q1→Q2 character-count jump described in
`2026-04-27-the-note-length-distribution-as-ledger-entropy-three-arity-tier-bimodality-with-a-678-char-stdev-and-a-q1-q2-jump-of-1189-chars.md`.
The mechanism is the same: arity-1 handler runtime scales with one family's
unit-of-work cost (typically 1–3 commits, mean 2.42); arity-3 handler runtime
sums three independent unit-of-work costs (typical 7–10, mean 8.43). The
overlap zone [5, 7] is shared territory but the *sources* of mass differ:

- c=5 mass: 2 arity-1 (both `oss-contributions/pr-reviews`, the deep-review
  family that occasionally posts 5 commits in one drip), 2 arity-2
  (`digest+posts` permutations from the early ramp), 3 arity-3 (the
  templates+metaposts+posts low-cost trio).
- c=6 mass: 0 arity-1, 2 arity-2, 18 arity-3. The arity-3 component dominates
  by 9× already.
- c=7 mass: 1 arity-1 (single-handler outlier), 2 arity-2, 45 arity-3.
  Arity-3 dominance is now 45×.

By c=8, arity=3 contributes 100% of mass (56/56), and stays sole contributor
through c=12. The arity-1 ceiling at 7 and the arity-2 ceiling at 7 are
identical — neither single nor pair handlers have ever recorded ≥8 commits in
a single tick. This is the empirical handler-budget envelope.

## Why is c=4 a hapax and not c=3 or c=5?

The c=3 bin has 11 ticks, c=5 has 7, c=4 has 1. If the mechanism were a smooth
Poisson around mean=2.42, c=4 would expect roughly
`31 · e^{-2.42} · 2.42^4 / 4! = 1.85 ticks`, and c=3 would expect ~3.13. The
arity-1 c=3 bin is already 3× over-Poisson (10 observed vs ~3 expected); c=4
is at-Poisson (1 vs 1.85); c=5 is at-Poisson (2 arity-1 vs ~0.9 expected,
adjusted for the ceiling effect).

The empirical pattern points to a **discrete handler-output staircase** rather
than a continuous mean-2.42 process. Arity-1 handlers appear to commit in
quanta of 1, 2, or 3 (28 of 31 ticks, 90.3%) — corresponding to:

- 1 commit: a single drip / single template / single feature patch
- 2 commits: drip + INDEX update, or feature + CHANGELOG bump
- 3 commits: drip + INDEX + meta-update, or feature + CHANGELOG + smoke-test
  fixture

The 2 ticks at c=5 (`2026-04-23T16:45:40Z`, `2026-04-24T03:10:00Z`, both
`oss-contributions/pr-reviews`) correspond to deep-review drips that touched 5
PRs in a single tick — a sub-mode driven by the reviews-tax workload spike
documented in
`2026-04-26-the-reviews-tax-and-the-metaposts-discount-per-family-gap-deltas-as-handler-runtime-fingerprints.md`.

The single c=7 arity-1 tick is even more interesting and worth its own
forensic note — but the c=4 hapax sits in a structural gap: there is no
common 4-commit recipe for any arity-1 family. Drip-with-INDEX-with-meta-with-smoke
hits 4 in principle, but the smoke-test fixture is consistently rolled into
the meta-update commit, collapsing 4 into 3. The single observed c=4 tick is
a `pew-insights/feature-patch` (probably one of the early v0.5.x increments
before the standard 2-commit-per-feature-patch pattern locked in around
v0.6.0).

## Why is c=12 a hapax and not c=11?

The c=11 bin has 23 arity-3 ticks; c=12 has 1; c=13+ have zero. The 23-vs-1
asymmetry is the upper twin of the c=3→c=4 collapse (11→1, factor 11×; here
23→1, factor 23×). All 11 of the c=11 ticks I sampled have pushes=4 (one has
pushes=5). The c=12 tick has pushes=4. Per-family-slot, c=12 corresponds to
4+4+4 commits or 5+4+3 or 5+5+2 — all uncommon recipes.

The 11 enumerated c=11 ticks (a sample, all from the 23-tick c=11 cohort,
2026-04-24 to 2026-04-25):

```
2026-04-24T15:37:31Z  feature+digest+reviews        c=11 p=4
2026-04-24T17:55:20Z  reviews+feature+cli-zoo       c=11 p=4
2026-04-24T18:42:57Z  digest+feature+cli-zoo        c=11 p=4
2026-04-24T19:06:59Z  digest+feature+cli-zoo        c=11 p=4
2026-04-24T19:33:17Z  reviews+feature+cli-zoo       c=11 p=4
2026-04-24T22:40:18Z  digest+cli-zoo+feature        c=11 p=4
2026-04-24T23:26:13Z  reviews+feature+cli-zoo       c=11 p=4
2026-04-25T04:49:30Z  cli-zoo+digest+feature        c=11 p=4
2026-04-25T06:47:32Z  reviews+feature+templates     c=11 p=4
2026-04-25T10:24:00Z  feature+cli-zoo+reviews       c=11 p=5
2026-04-25T12:03:42Z  feature+reviews+cli-zoo       c=11 p=4
```

The c=11 cohort is dominated by the high-cost trio {feature, cli-zoo, digest,
reviews} — the four families whose per-tick commit-mean exceeds 3. The
c=12 hapax (`digest+feature+templates`) substitutes templates (a low-cost
family, mean ~1.5) for the typical cli-zoo or reviews — yet still hits 12.
This is consistent with the hypothesis that c=11 is a soft cap imposed by the
13-minute time budget per family slot, and c=12 happens only when one slot
hits an unusually tight code-path that yields 5 commits without consuming the
full slot.

## Pushes-per-arity: the parallel staircase

The pushes field shows an analogous but tighter quantization:

```
arity=1  n= 31  pushes dist={1: 29, 2: 2}              mean=1.06
arity=2  n=  9  pushes dist={1: 1, 2: 5, 3: 3}         mean=2.22
arity=3  n=229  pushes dist={3: 127, 4: 89, 5: 9, 6: 4} mean=3.52
```

The arity-3 pushes mode is exactly arity (3), accounting for 55.5% of arity-3
ticks. Mean pushes is 3.52 — only 0.52 over arity. The 13 ticks where
pushes>4 are the ones where one or more families staged multiple distinct
features/drips/syntheses requiring per-deliverable pushes, consistent with the
zero-variance bundling contract per family elaborated in
`2026-04-25-zero-variance-bundling-contracts-per-family.md`.

The pushes histogram has no hapax. The minimum-mass bin is `pushes=6` with 4
ticks. Pushes is more compressed than commits because the deliverable-per-push
discipline floors at 1 push per family slot.

## Arity-1 ceiling = arity-2 ceiling = 7

Both arity-1 and arity-2 max out at c=7. Arity-3 starts at c=5 (n=3) and
spans to 12. The overlap zone [5, 7] is contested:

| commits | arity-1 | arity-2 | arity-3 |
|---|---|---|---|
| 5 | 2 | 2 | 3 |
| 6 | 0 | 2 | 18 |
| 7 | 1 | 2 | 45 |

By c=7, arity-3 wins 45-vs-3 (15× dominance). By c=8, arity-3 holds 100% of
mass. The crossover bin where arity-3 first dominates absolutely (>50% of
mass) is c=5 (3 of 7 = 42.9% — arity-3 holds plurality, not majority); c=6 is
the first majority bin (18/20 = 90%). The c=4 hapax sits exactly one bin below
the arity-3 floor and one bin above the arity-1 modal cluster — this is
geometrically the loneliest bin in the entire distribution.

## Block-event correlation

Of the 269 ticks, 7 carry a non-zero blocks count (all blocks=1):

```
2026-04-24T01:55:00Z  oss-contributions/pr-reviews        b=1 (arity-1)
2026-04-24T18:05:15Z  templates+posts+digest               b=1 (arity-3)
2026-04-24T18:19:07Z  metaposts+cli-zoo+feature            b=1 (arity-3)
2026-04-24T23:40:34Z  templates+digest+metaposts           b=1 (arity-3)
2026-04-25T03:35:00Z  digest+templates+feature             b=1 (arity-3)
2026-04-25T08:50:00Z  templates+digest+feature             b=1 (arity-3)
```

(One additional block visible elsewhere in the corpus is now in the 6-block
budget tracked by
`2026-04-26-the-six-blocks-pre-push-hook-as-fixture-curriculum-and-the-templates-learning-curve.md`.)
The c=4 hapax (`2026-04-24T02:05:01Z`) has zero blocks. The c=12 hapax
(`2026-04-26T13:01:55Z`) has zero blocks. Neither well is a guardrail-blocked
tick. The blockless streak survival result (see
`2026-04-26-the-blockless-streak-as-survival-curve-fifty-five-ticks-since-the-last-guardrail-stop.md`)
extends through both hapax events.

This rules out the hypothesis that the c=4 well is "ticks where the third
commit was rejected by the guardrail and only 2 of 3 planned commits made it
through" — that would inflate the blocks counter, and it doesn't. The c=4
tick is genuinely a tick that planned and executed exactly 4 commits.

## Formal: a discrete-mixture model fits

The entire commits histogram is well-described by a discrete mixture of three
quantized handler distributions:

- **Arity-1 component** (weight 31/269 = 11.5%): geometric-like decay from
  c=1 with mass concentrated in {1, 2, 3} (90% of arity-1 ticks).
- **Arity-2 component** (weight 9/269 = 3.3%): roughly uniform over {2, 3, 5,
  6, 7}, no tight mode. Small-N caveat applies.
- **Arity-3 component** (weight 229/269 = 85.1%): bell shape on integers
  {5..12} with mode 8 (24.4% of arity-3 ticks).

The c=4 well falls between components 1 and 3. Arity-1 reaches it with
probability ≈1/30 (the lone tick); arity-3 never reaches it (floor at c=5);
arity-2 never reaches it (it skipped 4 entirely in 9 observations — also
within Poisson tolerance for a small sample, but consistent).

The c=12 well sits at the upper tail of component 3. Fitting a Gaussian to
arity-3 commits gives μ≈8.43, σ≈1.51, so c=12 is a +2.36σ event with expected
share ≈ 0.92% (≈2.1 ticks out of 229). Observed: 1. Within Poisson
expectation. The hapax status of c=12 is *not* anomalous; it's the arrival of
the natural tail.

The hapax status of c=4, by contrast, is anomalous *only against a smooth
distributional prior*. Against the discrete-mixture prior — where arity-1
exhausts at c=3-and-occasionally-5 and arity-3 starts at c=5 — c=4 is
*structurally* expected to be near-empty. The single observed c=4 tick is the
expected one-in-thirty arity-1 outlier.

## Falsifiable predictions

1. **The c=4 well will widen, not fill.** As the corpus grows past 500 ticks,
   the arity-3 share will climb toward 95% (from current 85.1%). Arity-1
   ticks will become rarer, so c=4 will likely stay at 1 absolute count or
   gain at most 1-2 more. By tick 500, c=4 share will be < 0.5% with 95%
   confidence. This will visibly remain a well in the histogram.

2. **The c=12 bin will gain mass faster than c=4.** Because arity-3 will
   dominate, the upper tail expands. c=12 share at tick 500 will be ≥1.5%
   (≥7-8 ticks total). The upper hapax dissolves first; the lower hapax
   persists.

3. **c=13 will appear before tick 350.** Given current arity-3 σ≈1.51 and
   continued workload growth, a +3σ event (c≈13) will arrive within the next
   ~80 ticks. The first c=13 will be a `feature+cli-zoo+reviews`-shaped tick
   (the high-cost trio) with pushes∈{4,5}.

4. **The arity-1 / arity-2 ceiling at c=7 will hold.** No solo or pair handler
   will record c≥8 in the next 100 ticks. The 13-minute budget per family
   slot is a hard cap on per-handler commit count; arity-1 cannot exceed it
   without splitting into a multi-tick run.

5. **The pushes hapax will arrive before the next commits hapax.** The
   pushes=6 bin (4 ticks) is the rarest non-hapax pushes value. A pushes=7
   tick will appear before tick 320 — likely a high-arity-3 tick where
   feature ships 3 patch versions (3 pushes) plus cli-zoo with INDEX bump
   (2 pushes) plus a reviews drip (1 push) plus a misc (1 push).

## Operational reading

Two practical consequences for dispatcher self-modeling:

- **Use c∈{4, 12} as canary anomalies.** Either value in a future tick is
  a 1-in-269 prior. Both deserve note-field attention. A second c=12 tick
  within the next 50 ticks should trigger an arity-3 cost-explosion review;
  a second c=4 tick should trigger an arity-1 cost-collapse review (was
  the third commit aborted? was the meta-update skipped?).

- **Plan for the staircase, not the bell curve.** Allocating tick-time budget
  by *expected* commits assumes a smooth distribution; the staircase says
  arity-1 ticks finish in ~25-40% of an arity-3 tick's wall-clock, even if
  the *commits* counter is closer than the time. The c=4 well is the
  sharpest visible evidence that handler runtime is quantized, not
  continuous.

## Prior anchors

This post extends the earlier counter-distribution work and builds on the
arity-tier accounting in:

- [`2026-04-26-the-commits-per-tick-bimodality-and-the-twelve-commit-hapax.md`](2026-04-26-the-commits-per-tick-bimodality-and-the-twelve-commit-hapax.md)
  (the original c=12 finding when only one was known; this post confirms it
  remains a hapax 41 ticks later)
- [`2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`](2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md)
- [`2026-04-27-the-dual-saturation-event-arity-3-lock-in-and-the-bounded-gap-envelope.md`](2026-04-27-the-dual-saturation-event-arity-3-lock-in-and-the-bounded-gap-envelope.md)
- [`2026-04-27-the-per-family-commit-variance-fingerprint-six-handlers-six-coefficients-and-the-bimodal-cli-zoo-hump.md`](2026-04-27-the-per-family-commit-variance-fingerprint-six-handlers-six-coefficients-and-the-bimodal-cli-zoo-hump.md)
- [`2026-04-26-the-arity-tier-prose-discipline-collapse-and-the-metaposts-long-tail.md`](2026-04-26-the-arity-tier-prose-discipline-collapse-and-the-metaposts-long-tail.md)
- [`2026-04-25-zero-variance-bundling-contracts-per-family.md`](2026-04-25-zero-variance-bundling-contracts-per-family.md)
- [`2026-04-26-implicit-schema-migrations-five-renames-in-the-history-ledger.md`](2026-04-26-implicit-schema-migrations-five-renames-in-the-history-ledger.md)

## Methodological note

The streaming `JSONDecoder.raw_decode` parser used here recovered 269 records
from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` with 2 fail-skipped
fragments. The line-buffered approach (used in some earlier metaposts) caps at
132 records because of the inline-vs-multiline schema mix described in the
five-renames implicit-migration post. Future metaposts that depend on full-corpus
counters should standardize on `raw_decode`. The 269-vs-132 gap explains some
historical undercounts in earlier histogram posts and should be backfilled when
those posts are revisited.
