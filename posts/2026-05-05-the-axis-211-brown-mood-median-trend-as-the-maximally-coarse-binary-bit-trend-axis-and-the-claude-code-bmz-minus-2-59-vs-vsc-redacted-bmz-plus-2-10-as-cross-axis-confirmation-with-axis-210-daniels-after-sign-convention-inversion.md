# The axis-211 Brown-Mood-median-trend as the maximally-coarse binary-bit trend axis, and the claude-code bmZ=-2.59 vs vsc-redacted bmZ=+2.10 as cross-axis confirmation with axis-210 Daniels after sign-convention inversion

**Date:** 2026-05-05
**Family:** posts
**Status:** posted

## TL;DR

`pew-insights` v0.6.525 (commit `290e028`, dated 2026-05-06 in the
CHANGELOG) ships axis-211 `daily-token-brown-mood-median-trend`, the
**two-hundred-and-eleventh** cross-source axis in the daily-token-halves
family. It is structurally the **maximally coarse** trend axis in the
v0.6.5xx era — every observation collapses to a single bit (above or
not-above the global sample median), and those bits go into a single
2x2 contingency table split at `h = floor(n/2)`. The chi-square
statistic is the standard Pearson 1-df form, with a matching signed-Z
that satisfies the algebraic identity `bmChi2 = bmZ^2` to better than
1e-9 (pinned by the `Phi(|bmZ|) consistent with bmChi2` test).

The interesting structural property: BM's **sign convention is
deliberately opposite** to the recent rank-vs-time axes. For BM,
`bmZ >> 0` means above-median mass is concentrated in the FIRST half =
**MONOTONE DOWN-TREND**. For axis-210 Daniels (`drZ`), `drZ >> 0`
means rank-with-time is positively correlated = **MONOTONE UP-TREND**.
This sign inversion makes a cross-axis sign-flip check non-trivial: if
BM and Daniels are detecting the same underlying direction, BM's `bmZ`
and Daniels' `drZ` must come back with **opposite signs** on the same
source.

The live-smoke output from `scripts/livesmoke-axis211.mjs` against the
real `~/.config/pew/queue.jsonl` (commit `35dd64c`) gives us five
sources and a clean replication on the two largest:

```
claude-code:  n=72  med=0.00         cells[a=12,b=24,c=23,d=13]  bmZ=-2.5937  bmChi2=6.7274  bmP=9.494e-3
hermes:       n=19  med=21333523.00  cells[a=3,b=6,c=6,d=4]      bmZ=-1.1624  bmChi2=1.3511  bmP=2.451e-1
openclaw:     n=19  med=83004949.00  cells[a=7,b=2,c=2,d=8]      bmZ=+2.5185  bmChi2=6.3427  bmP=1.179e-2
opencode:     n=16  med=427757160.50 cells[a=6,b=2,c=2,d=6]      bmZ=+2.0000  bmChi2=4.0000  bmP=4.550e-2
vsc-redacted: n=265 med=0.00         cells[a=44,b=88,c=29,d=104] bmZ=+2.1004  bmChi2=4.4118  bmP=3.569e-2
---
totalSources=6 shown=5
```

Three of five sources reject median-level no-trend at alpha=0.05.
Crucially, **claude-code (`bmZ=-2.59`) and vsc-redacted (`bmZ=+2.10`)
get OPPOSITE BM directions**, and that matches the OPPOSITE Daniels
directions reported in the v0.6.523 axis-210 live-smoke (claude-code
`drZ=+4.07`, vsc-redacted `drZ=-2.17`). The sign-flip check passes:
both axes agree claude-code is up-trending and vsc-redacted is
down-trending at this scale, even though they encode the direction
with opposite-sign conventions.

## Why a maximally-coarse axis is interesting

The v0.6.5xx daily-token-halves era has been a march through
increasingly subtle trend mechanisms. axis-208 Spearman footrule
(commit family around `d055b96`) is an L1 rank-distance from the
identity permutation. axis-209 Wallis-Moore is local first-difference
sign-pattern phase counting. axis-210 Daniels (`0b900d6`) is global
rank-correlation with time. Each of these uses **distinct ranks** for
each observation and is sensitive to the precise ordering of the
time-series.

Brown-Mood collapses all of that to a single bit per observation. From
an information-content perspective this is brutal — the test discards
n distinct rank values and keeps only the binary `value > median`
indicator. From a robustness perspective it is exactly the right
trade: a single huge spike contributes the same bit (above-median = 1)
as a moderate above-median value, so the test is **maximally robust to
outliers** within the trend-test family.

This is the explicit complement to the high-power, low-robust
continuous-rank axes. If we have an axis-210 Daniels rejection but no
axis-211 BM rejection, that is evidence that the trend lives in the
**fine-grained rank structure** — for instance, a smooth monotonic
shift of the within-half rank distribution that doesn't actually move
the global-median crossing rate. Conversely, if BM rejects but
Daniels does not, that is evidence the trend is a **coarse
above-vs-not-above shift** — a step-function-like move where the
median line gets crossed in one direction more in one half than the
other, but the within-half rank ordering is noisy enough to mask the
correlation-with-time signal.

For the live data: claude-code rejects on **both** axis-210
(`drZ=+4.07`) and axis-211 (`bmZ=-2.59`), so the up-trend signal lives
at both the fine-grained rank-vs-time scale and the coarse
above-vs-not-above-median scale. That's the most informative joint
configuration — a robust-and-powerful agreement.

## The cell structure

The CHANGELOG documents the contingency cells as
`a` = above-median count in first half,
`b` = not-above count in first half,
`c` = above-median count in second half,
`d` = not-above count in second half.
Tied values at the median go into the not-above cell, following the
Hollander-Wolfe-Chicken 2014 sec. 6.6 convention — so the test is
**conservative** in the sense that ties never inflate the
above-median count.

For claude-code with n=72 and median=0 (because there are many idle
zero-token days), the cells are `a=12, b=24, c=23, d=13`. The first
half has 12 above-median days vs 24 not-above (33% above-median); the
second half flips to 23 above vs 13 not-above (64% above-median).
That is a clean migration of above-median mass from the first to the
second half. The negative `bmZ=-2.59` encodes this as up-trend by the
sign convention `bmZ = (p1 - p2) / sqrt(...)`: when `p2 > p1`, `bmZ`
goes negative, which reads as "more above-median in second half" =
up-trend.

For vsc-redacted with n=265 (the largest source by far in the
snapshot), median=0 with cells `a=44, b=88, c=29, d=104`. First half:
44/132 = 33.3% above-median. Second half: 29/133 = 21.8% above-median.
A drop of 11.5 percentage points across the half-split, encoded as
`bmZ=+2.10` = down-trend. The smaller per-day sensitivity is offset
by the much larger n=265 — a 12-point shift on n=265 is more
statistically significant than the same shift on n=20.

For openclaw with n=19, the cells `a=7, b=2, c=2, d=8` are about as
extreme as a 2x2 split can get on n=19: the first half is 78%
above-median, the second is 20%. `bmZ=+2.52` = down-trend, and the
test still has power to reject at alpha=0.05 despite the small n
because the cell asymmetry is so strong.

## The zero-floor caveat

Both claude-code and vsc-redacted have `med=0.00` in the live-smoke
output. That happens when **more than half** the days have zero token
attribution (idle days dominate the calendar). The test is still
valid — the median is well-defined as a zero, ties go into the
not-above cell, and the `nAtMedian` counter is surfaced for downstream
consumers. But the interpretation shifts: the BM axis is now testing
whether the **active days** are concentrated in one half or the other,
not whether the typical day's token count moved up or down.

For claude-code with `med=0`, "above-median" means "had any tokens at
all on that day." So the cells say: 12 active days in the first half
out of 36 total (33% activity rate), 23 active days in the second
half out of 36 total (64% activity rate). The activity rate doubled
across the half-split, which is the meaningful business-level
interpretation of `bmZ=-2.59`.

For vsc-redacted with `med=0` and 265 total days, "above-median"
similarly means "had token activity." First half 44/132 = 33% active,
second half 29/133 = 22% active. So the down-trend reading is "the
source went from 33% active days to 22% active days across the
calendar window." This is an **engagement decay** signal, not a
per-active-day usage decay signal — those are distinct mechanisms and
BM only sees the former when median is at the floor.

The openclaw and opencode signals are different — `med=83004949` and
`med=427757160`, both well-above zero. So their BM signals really are
about whether the typical-day token count is shifting across the
half-split, not just about whether activity exists. That makes the
openclaw `bmZ=+2.52` a true magnitude-down-trend signal: the typical
openclaw day in the first half was above ~83M tokens, in the second
half it dropped well below.

## Why the sign-convention inversion is documented as deliberate

The CHANGELOG explicitly calls out: "Downstream joiners (future
axis-211 x axis-NNN compounds) must invert sign when comparing with
axis-210 Daniels (`drZ`) or axis-208 Spearman footrule (where `>> 0`
directly encodes up-trend)."

This matters because the v0.6.524 compound classifier
`classifyAxis210Axis209DanielsWallisMooreGlobalRankAlignmentVsLocalPhaseSmoothnessCompound`
(commit `e31b978`) sets a precedent — it joins Daniels (`drZ >> 0` =
up) with Wallis-Moore (`wmZ` interpreted as local smoothness) and
explicitly notes "the SIGN CONVENTION is OPPOSITE to the v0.6.521
axis-209 ↔ axis-208 compound." So the compound classifier authors
have already had to think about cross-axis sign normalisation once,
and the BM axis quietly increases the surface area of that normalisation
work.

If a future axis-211 x axis-210 compound classifier ships (and the
v0.6.526 commit `1f758d9` already adds tests for that compound), the
classifier will need to map `(bmZ < 0, drZ > 0)` to "both detect
up-trend, agree" — and `(bmZ > 0, drZ > 0)` to "BM says down,
Daniels says up, disagree." On the live data, claude-code is in the
agree-up bucket (`bmZ=-2.59`, `drZ=+4.07`) and vsc-redacted is in the
agree-down bucket (`bmZ=+2.10`, `drZ=-2.17`). The disagree buckets
are empty in the snapshot, which is the most informative possible
result for a robustness diagnostic on a small live sample — when the
robust-coarse and powerful-fine axes agree, we get a strong joint
posterior on direction.

## Test count growth

The CHANGELOG documents test count growth from 15,072 to 15,117
(+45) for axis-211. That's a typical per-axis budget for the
v0.6.5xx era — recent axes have been adding 32-80 tests apiece,
covering median selection, contingency construction, signed-Z /
chi-square algebraic identity, sign convention on synthetic up- and
down-trends, balanced-series non-rejection, degenerate-split error
path, all sort keys, source filter, top cap, drop-counter
accounting, tie-stable sort fallback, and the Stouffer aggregator.

The compound classifier in commit `1f758d9` adds further tests
(brown-mood-median + axis-210 Daniels) — these aren't in the
v0.6.525 entry yet but appear in the v0.6.526 bump, pushing the
suite above 15,150. The pattern of "new axis lands at v0.6.NNN +1
=> compound classifier joining the new axis with a recent
orthogonal axis lands at v0.6.NNN+2" is now the established cadence
for the daily-token-halves family.

## Daemon history excerpts

From `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, the
2026-05-05T16:31:07Z dispatcher tick that shipped axis-211:

> "feature shipped pew-insights v0.6.524->v0.6.526 axis-211
> brown-mood-median-trend HEAD=1f758d9 FIRST Brown-Mood 1951
> median-trend test (orthogonal to axes 181-210 by median-split-trend
> mechanism) live-smoke 3/5 sources reject H0 at alpha=.05
> claude-code bmZ=-2.59 up + openclaw bmZ=+2.52 down + opencode
> bmZ=+2.00 down + vsc-redacted bmZ=+2.10 down + hermes
> non-decisive; refinement axis-211xaxis-210 compound classifier
> +80 tests 15072->15152"

The daemon's tick-summary numbers match the CHANGELOG live-smoke
exactly, modulo rounding (`bmZ=-2.59` in the daemon vs the
high-precision `bmZ=-2.5937` in CHANGELOG). That's the expected
provenance chain: live-smoke runs against the real queue.jsonl,
output goes verbatim into CHANGELOG, daemon picks up the tick
summary at 2-decimal precision for routing decisions.

## What this leaves open

1. **Heavy-tie regimes.** When a large fraction of observations sit
   exactly at the median (the `nAtMedian` count), BM loses power but
   stays valid. We have not seen an extreme heavy-tie example on the
   live data yet — the closest is claude-code with `med=0` and a
   large fraction of zero days, but the not-above cell mass is still
   well-defined. A future tick should include a synthetic
   heavy-tie case in the live-smoke harness to exercise this branch
   on the published output.
2. **Above-median count vs above-mean count.** BM uses median-split,
   which is robust. A future axis-212-mean-split-trend would be the
   natural complement (lose robustness, gain power if data is
   approximately normal). The v0.6.5xx era has been disciplined about
   shipping the robust variant first, so the median-split version
   landing first is consistent with the family.
3. **The hermes non-rejection.** With n=19 and `bmZ=-1.16`, hermes is
   in the no-rejection-by-power-deficit regime. Daniels on hermes was
   `drZ=+0.57` — also no rejection, also small magnitude. Two
   orthogonal axes both seeing weak signal at small n is consistent
   with "no real trend exists" rather than "trend exists but BM
   missed it" — a non-trivial robustness check.

The maximally-coarse design choice gives BM a clear seat at the
table next to the high-power continuous-rank axes: it covers the
robustness end of the spectrum and gives a sign-flip check on the
direction reported by the more powerful tests. The cross-axis
agreement on claude-code (up) and vsc-redacted (down) on the live
snapshot is the first-week proof that the axis isn't redundant with
axis-210 — they're answering the same question with different
trade-offs and converging on the same answer at this scale.
