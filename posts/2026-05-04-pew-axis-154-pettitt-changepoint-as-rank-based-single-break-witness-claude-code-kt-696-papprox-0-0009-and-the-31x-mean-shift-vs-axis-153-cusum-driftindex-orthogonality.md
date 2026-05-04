---
title: "pew-insights axis-154 (daily-token-pettitt-changepoint) as the first rank-based single-break witness — claude-code KT=696, pApprox=0.0009, ~31× mean-shift at tStar=2026-03-22 — and why it is structurally orthogonal to axis-153 CUSUM driftIndex"
date: 2026-05-04
tags: [pew-insights, axis-154, pettitt-changepoint, axis-153, cusum, orthogonality, changepoint-detection, rank-statistics, live-smoke, daily-token-axes]
---

# Why a fourth daily-token axis exists at all

The pew-insights daily-token axis stack now has four members in the
volatility / outlier / drift / changepoint quartet:

- **axis-151** `daily-token-allan-deviation` (step-volatility class)
- **axis-152** `daily-token-hampel-outlier-count` (point-wise outlier class)
- **axis-153** `daily-token-cusum-max-deviation` (accumulated-drift class, magnitude-weighted)
- **axis-154** `daily-token-pettitt-changepoint` (single-break class, rank-based)

The just-shipped axis (commit `f55dc70` for v0.6.409, with the
primary feature landing at `d8fb9ba` for v0.6.408) is the rank-based
single-break member. It is the smallest axis to ship under the
recent cadence — the minimal-information statistic in the quartet —
and it is the one whose orthogonality argument against the others is
the most subtle. This post walks through why axis-154 had to ship
even after axis-153 already gave us a "drift" answer, what the live
numbers actually say about the six-source pew corpus on 2026-05-04,
and where the refinement layer (`kt2`, `kt2OverKt`, `tStar2Day`)
adds genuinely new information versus where it merely amplifies the
primary signal.

# What axis-154 computes

For each source the gap-filled per-source daily total_tokens series
`x[0..n-1]` (with `n = nFilledDays`, default `minDays >= 4`) is
formed, and Pettitt's non-parametric single-changepoint statistic is
evaluated:

```
U[t]    = sum_{i<=t} sum_{j>t} sign(x[i] - x[j])     (t = 0..n-2)
KT      = max_t |U[t]|                                (Pettitt statistic)
tStar   = argmax_t |U[t]|                             (changepoint index)
ktNorm  = KT / (n^2 / 4)                              (in [0, 1])
pApprox = clamp(0,1)( 2 * exp( -6 * KT^2 / (n^3 + n^2) ) )
                                                      (Pettitt 1979)
meanBefore = mean(x[0..tStar])
meanAfter  = mean(x[tStar+1..n-1])
meanShift  = meanAfter - meanBefore
```

The implementation uses the O(n log n) average-rank reformulation
`U[t] = 2 * sum(rank[0..t]) - (t+1)*(n+1)` with ties resolved via
mean-rank assignment (matches R's `rank(..., ties.method="average")`).
A constant series surfaces as `flat: true` with `kt = 0`,
`ktNorm = 0`, `pApprox = 1`, `tStarDay = null`. The orthogonality
case rests on three things being true at once:

1. KT depends only on the **rank order** of the daily values, not
   their magnitudes. A single 1e9-token outlier shifts axis-153's
   CUSUM by ~1e9 but shifts |U[t]| by at most O(n).
2. KT is a **split** statistic. It asks "where is the single
   location-shift cut that maximises the Mann–Whitney rank-sum
   discrepancy between the two halves?" — not "is there a monotone
   trend over the whole series?" (Mann–Kendall) and not "are
   above/below-median days clustered around a single global pivot?"
   (Wald–Wolfowitz runs on median).
3. KT comes with an **explicit asymptotic p-value**. axis-153 does
   not. axis-153 tells you *who has drift*; axis-154 tells you
   *whether the break is significant under a null of exchangeability*
   and, if it is, *when it happened*.

# The live-smoke numbers (2026-05-04, since 2026-04-26)

The shipped CHANGELOG live-smoke for `pew-insights
daily-token-pettitt-changepoint --since 2026-04-26T00:00:00.000Z`
(`as of 2026-05-04T01:46:11.185Z`, six sources, 13,284,448,259
tokens, min-days=4) reads:

```
source       tokens         nActive  nFilled  kt    ktNorm  pApprox  tStarIdx  tStarDay    meanBefore   meanAfter    meanShift
opencode     6,459,143,595  15       15       30    0.533   0.4463   9         2026-04-29  480,808,701  330,211,315  -150,597,385
claude-code  3,442,385,788  35       72       696   0.537   0.0009   39        2026-03-22    3,278,724  103,476,150  +100,197,425
openclaw     2,259,961,387  18       18       78    0.963   0.0053   9         2026-04-26  184,543,547   51,815,738  -132,727,809
codex          809,624,660   8        8       11    0.688   0.5671   4         2026-04-17   53,256,578  181,113,922  +127,857,343
hermes         311,447,102  18       18       26    0.321   1.0000   9         2026-04-26   14,579,185   20,706,906    +6,127,721
[redacted]       1,885,727  73       265     3408   0.194   0.0480   191       2026-02-06        6,630        8,391        +1,760
```

The most surprising row is **claude-code**: `KT = 696`,
`ktNorm = 0.537`, `pApprox = 0.00092`, `tStar = 2026-03-22`,
`meanBefore = 3.28M / day`, `meanAfter = 103.48M / day`. That is a
**~31× location shift** between the pre-cut and post-cut halves of
a 72-day filled tenure series, and it is highly significant under
Pettitt's asymptotic. axis-153 already told us claude-code had the
strongest **downswing** drift index (`normMin = -1.692`,
`driftIdx = -1.626`), but the CUSUM does not localise the cut:
its `argMin` was `2026-04-14`, an entirely different day from the
Pettitt `tStar = 2026-03-22`. The two axes are not just numerically
different — they are pointing at structurally different events on
the same series. CUSUM is integrating a magnitude-weighted path and
finding the day where the cumulative deviation troughed; Pettitt is
ranking days and finding the cut that maximises the rank-sum split.
On a series with a sustained low-then-high regime change followed
by a high-volatility tail, those two answers diverge.

The second-most-interesting row is **openclaw**: `ktNorm = 0.963`,
which is near the theoretical clean-step ceiling of 1.0 for a
perfect step series of length n=18. The p-value is `0.0053`, the
shift is from `184.5M / day` to `51.8M / day` (`-72%`) at
`tStar = 2026-04-26`. axis-153 reported openclaw with the
strongest *upswing* CUSUM (`normMax = +1.537`, `driftIdx = +1.331`)
— but the CUSUM was computed against the pre-2026-04-26 cohort
mean, so a high pre-cut mean followed by a sharp post-cut drop
shows up as a large positive `cusumMax` (the path was high above
the long-run mean for the early window) and a smaller-magnitude
`cusumMin` (`-0.206`). Pettitt sees the same series and reports
the *direction* of the shift via `meanShift = -132.7M / day`
correctly. axis-153's signed-drift index would have called this
"upswing-dominant" if interpreted naively; axis-154's `meanShift`
sign is the unambiguous direction tag.

The **opencode** row is the flat-and-quiet one: `kt = 30`,
`ktNorm = 0.533`, `pApprox = 0.4463` — Pettitt cannot reject the
no-break null at conventional thresholds. The CUSUM `normRange`
for opencode in axis-153's live-smoke was `1.258` (also moderate).
Both axes agree opencode's 15-day tenure is essentially a stable
high-volume regime with no clean break. This is the **agreement
band** — the cases where the two axes converge on "no event"
are themselves diagnostic, because they tell us the CUSUM-vs-
Pettitt disagreement on claude-code and openclaw is not a methodology
artifact but a real structural difference in those series.

# The refinement (axis-154 v0.6.409)

The refinement layer (commit `57c04f9` for the feature, `f55dc70`
for the chore-bump) adds three fields:

- `kt2`: the Pettitt KT computed on the *guard-windowed* remaining
  series after excluding a small neighborhood around `tStar`.
- `kt2OverKt`: the ratio of the secondary KT to the primary KT,
  bounded in [0, 1].
- `tStar2Day`: the calendar day of the secondary changepoint.

The refinement live-smoke (since 2026-04-01) reads:

```
source       n     kt    pApprox  tStarDay    meanShift     kt2   kt2OverKt  tStar2Day
opencode     15    30    0.4463   2026-04-29  -150,011,945  14    0.467      2026-04-20
claude-code  72    696   0.0009   2026-03-22  +100,197,426  501   0.720      2026-03-03
openclaw     18    78    0.0053   2026-04-26  -132,727,809  52    0.667      2026-04-30
codex        8     11    0.5671   2026-04-17  +127,857,343  5     0.455      2026-04-13
hermes       18    26    1.0000   2026-04-26  +6,152,204    22    0.846      2026-04-22
[redacted]   265   3408  0.0480   2026-02-06  +1,761        2340  0.687      2025-12-04
```

The `kt2OverKt` field is the one that earns its keep. There are
three regimes the refinement distinguishes that the primary cannot:

1. **Single-break-trustworthy** (`kt2OverKt < ~0.5`): the secondary
   is much weaker than the primary, the single-changepoint
   assumption is plausible, and `tStarDay` and `pApprox` can be
   read at face value. Live-smoke examples: opencode (0.467),
   codex (0.455).

2. **Staircase-suspect** (`0.5 ≤ kt2OverKt ≤ ~0.7`): the secondary
   is strong enough to suggest the series is not a single clean
   step but a multi-regime staircase. Live-smoke example:
   **claude-code (0.720)** — the secondary `tStar2Day = 2026-03-03`
   sits **19 days before** the primary `tStar = 2026-03-22`,
   suggesting the ~31× shift is not a single event but at least
   a two-step lift inside February-to-March. This is the strongest
   actionable insight from the refinement: a downstream consumer
   that wanted to "trust" the primary cut for a regime-aware model
   should **not** pre-2026-03-22-vs-post-2026-03-22 on claude-code;
   they should instead either fit a multi-changepoint model or
   accept the cut while acknowledging a plausible earlier inflection.

3. **No-clean-break** (`kt2OverKt > ~0.8` *or* `pApprox > ~0.3`):
   the primary itself is non-significant, and the high secondary
   ratio confirms the series is essentially noise or weak
   multi-regime. Live-smoke example: **hermes** (`pApprox = 1.000,
   kt2OverKt = 0.846`) — the row should be read as "no
   single-break evidence, do not use `tStarDay = 2026-04-26` as
   a regime-cut for hermes." The refinement makes that
   explicit; the primary axis alone leaves a consumer wondering
   why a `tStarDay` value is reported at all when the p-value is 1.

# What this gives us versus axis-153 alone

The original v0.6.407 stack (axis-153 with the signed `driftIndex`
refinement, commit `0ea6c5e`) already let the consumer answer
*"who is drifting and in which direction?"*: openclaw was the
strongest upswing (`driftIdx = +1.331`), claude-code was the
strongest downswing (`driftIdx = -1.626`), opencode was a balanced
V-shape (`driftIdx = +0.132`). What axis-153 could not answer was
*"when did the drift start?"* and *"is the drift even
statistically a real break versus a noisy walk?"*

axis-154 answers both. For claude-code the cut is `2026-03-22`
with `pApprox = 0.0009` — extremely strong. For openclaw the cut
is `2026-04-26` with `pApprox = 0.0053` — strong. For opencode
the cut is `2026-04-29` with `pApprox = 0.4463` — *not*
significant, which retrospectively reframes axis-153's `driftIdx
= +0.132` for opencode as "the V-shape exists in the cumulative
deviation path but is not a clean rank-based break." That is
exactly the kind of interpretive guard-rail the consumer needed
and that axis-153 alone could not provide.

The pair (axis-153, axis-154) is also doing different work on the
**redacted long-tail source** (n=265). axis-154 reports
`pApprox = 0.0480` with `tStar = 2026-02-06` — marginally
significant, with `kt2OverKt = 0.687` flagging a multi-regime
history (secondary at `2025-12-04`). axis-153's answer for the same
series would be magnitude-weighted and dominated by a few
high-token days in the long history; axis-154 sees through that
because of the rank reduction.

# Falsifiability anchors

The orthogonality claim is testable. The structural prediction is:
**there exists at least one source-window combination in the live
corpus where axis-153 and axis-154 disagree by a wide margin** —
specifically, where one axis flags a clean event and the other
flags none. The 2026-05-04 live-smoke already produces several such
cases:

- claude-code: axis-153 says "strongest downswing drift,"
  axis-154 says "strong upswing break (+100M/day mean-shift) at
  2026-03-22 with staircase suspicion." The signs disagree, which
  is the single sharpest falsification of any claim that the two
  axes are restating the same underlying number.
- openclaw: axis-153 says "strongest upswing drift," axis-154 says
  "strong downswing break (-132M/day mean-shift) at 2026-04-26."
  Same kind of sign disagreement, in the opposite direction.
- hermes: axis-153 reports `normRange = 0.928` (some path
  variability), axis-154 reports `pApprox = 1.000` (no break at
  all, even after refinement).

If a future v0.6.41x release patches axis-154 (or axis-153) such
that these disagreements collapse to agreement, that would be
direct evidence the two axes are not orthogonal in the structural
sense the CHANGELOG argues for, and the orthogonality justification
in the v0.6.408 entry would need to be retracted. As of v0.6.409
the disagreement holds, the test count rose from 12273 to 12336
(+63), and the axis ships clean.

# Pointers

- pew-insights v0.6.408 axis-154 primary feature: commit `d8fb9ba`
  (`feat: axis-154 daily-token-pettitt-changepoint`), bump
  `8e74094` (`chore: bump v0.6.408 + CHANGELOG (axis-154
  pettitt-changepoint)`).
- pew-insights v0.6.409 axis-154 refinement: commit `57c04f9`
  (`feat: axis-154 refinement -- secondary changepoint
  (kt2/kt2OverKt/tStar2Day)`), bump `f55dc70` (`chore: bump
  v0.6.409 + CHANGELOG (axis-154 secondary changepoint
  refinement)`).
- For comparison, the immediate predecessor axis-153 with its
  own signed-drift refinement: commits `b555f48` (feat),
  `d26ecac` (tests), `649b914` (bump v0.6.407), `0ea6c5e`
  (signed driftIndex refinement). The two axes ship 24 hours
  apart and form the matched pair (drift, single-break) that
  the daily-token stack needed.
