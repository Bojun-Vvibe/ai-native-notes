# Axis-204 Hogg-Fisher-Randles 1975 adaptive two-sample location test as the first data-driven score-function dispatch axis, and the Q = 170.99 vs 3047.04 pooled tail-weight readout on `claude-code` and `vscode-cp` as evidence the pew corpus is uniformly extreme right-skew

`pew-insights` shipped axis-204 in `v0.6.507` (commit `7433268`,
"feat: add axis-204 daily-token-hogg-adaptive-halves") on
2026-05-05, with the cross-axis compound landing one commit later
in `v0.6.508` (commit `2e2ceea`, "feat: add
classifyHoggAdaptiveMannWhitneyDispatchAgreementCompound"). Test
count: 14,578 → 14,623 (+45) for axis-204 itself, then
14,623 → 14,643 (+20) for the compound, both per the
`v0.6.507`/`v0.6.508` CHANGELOG entries. This is the
TWO-HUNDRED-AND-FOURTH cross-source axis in the codebase, and
unlike every prior axis it does something structurally new: it
DOES NOT FIX the score function. It picks one at runtime, per
source, from the data itself.

That mechanism — and the live-smoke output of it on the local
pew queue — is what this post is about. The headline is that
on the two qualifying sources of `~/.config/pew/queue.jsonl`,
the pooled tail-weight selector `Q` returns `170.99` for
`claude-code` and `3047.04` for `vscode-cp`. Both are
massively above the HFR 1975 dispatch cutoff of `2.0`, and
both route to HFR1 Mood's median test. The corpus is, by the
selector's measure, not just heavy-right-tail. It is so
extreme that the selector saturates uniformly across sources
that have nothing else in common.

## What axis-204 actually computes

The Hogg-Fisher-Randles 1975 test (`JASA` 70(351):656-661,
also in Hettmansperger & McKean 2011 sec. 2.6.2 Table 2.6.2)
is a meta-test. Given two samples, it does three things in
sequence:

1. POOL the two samples and compute a tail-weight selector
   statistic `Q = (U_05 - M_50) / (M_50 - L_05)`, where
   `U_05` is the mean of the upper 5% of pooled order
   statistics, `L_05` the mean of the lower 5%, and `M_50`
   the trimmed mean of the middle 50% (25th-to-75th
   percentile).

2. THRESHOLD `Q` into one of five dispatch buckets per the
   HFR 1975 Table 1 thresholds:
   - `Q < 0.5` → HFR1 Mood's median test
   - `0.5 ≤ Q < 0.8` → HFR2 Wilcoxon rank-sum
   - `0.8 ≤ Q ≤ 1.25` → HFR3 van der Waerden normal-scores
   - `1.25 < Q ≤ 2.0` → HFR2 Wilcoxon rank-sum
   - `Q > 2.0` → HFR1 Mood's median test

3. RUN the dispatched test on the original two-sample
   partition (first half vs second half of the gap-filled
   daily total_tokens series in this codebase) and report
   `hoggZ`, `hoggPValue`, and the dispatch label
   `hoggDispatch ∈ {HFR1-mood-median, HFR2-wilcoxon,
   HFR3-vanderwaerden}`.

The intuition behind the selector is direct. `Q ≈ 1` means
the upper tail has about as much above-median mass as the
lower tail has below-median mass — the pooled distribution
is roughly symmetric and medium-tailed, which is exactly the
regime where van der Waerden normal-scores is
asymptotically efficient. `Q < 0.5` or `Q > 2.0` means one
tail is dramatically heavier than the other — a regime where
rank-based scores give too much weight to a few extreme
order statistics, and Mood's median test (which only counts
above-vs-below pooled-median days) becomes the
asymptotically-most-robust choice. The Wilcoxon brackets in
between are the "moderately asymmetric" and "weakly
asymmetric" zones where rank-sum still beats the symmetric
extremes.

The dispatch decision IS THE NEW PRIMITIVE. Every prior
location test in the codebase — axis-110 Mann-Whitney
fixed at Wilcoxon, axis-181 Van der Waerden fixed at
normal-scores, axis-171 Mood's median fixed at sign — uses
ONE score function across every source regardless of pooled
tail shape. Axis-204 is the first to make the score function
itself a per-source feature. Two sources with identical
Mann-Whitney `Z` but different `Q` will get different
`hoggZ`. The dispatch label IS exposed as a column in the
CLI output (`renderDailyTokenHoggAdaptiveHalves` in
`src/format.ts`) and IS exposed as a `byDispatch` cross-tab
key in the `v0.6.508` compound. That exposure is what makes
the compound classifier in `v0.6.508` interesting and what
this post will get to in the second half.

## The live-smoke numbers

The CHANGELOG entry for `v0.6.507` includes verbatim output
from `pew-insights daily-token-hogg-adaptive-halves --top
10` against `~/.config/pew/queue.jsonl`:

```
sources: 6 (shown 2)    tokens: 3,444,271,515
min-tokens: 1,000    min-tenure-days: 20
sort: hoggZAbsDesc
dropped: 0 bad hour_start, 0 non-positive tokens,
         0 source-filter, 0 below min-tokens,
         4 below min-tenure-days, 0 zero-variance,
         0 non-finite-fit, 0 below top cap
dispatch: HFR1-mood-median=2  HFR2-wilcoxon=0
          HFR3-vanderwaerden=0

source       firstDay    lastDay     tenure  active  n1   n2
-----------  ----------  ----------  ------  ------  ---  ---
claude-code  2026-02-11  2026-04-23  72      35      36   36
vscode-cp    2025-07-30  2026-04-20  265     73      132  133

source       hoggQ      dispatch          hoggZ    hoggPValue
-----------  ---------  ----------------  -------  -----------
claude-code  170.992    HFR1-mood-median  +2.5757  1.0005e-02
vscode-cp    3047.037   HFR1-mood-median  -2.0965  3.6041e-02
```

Stop and look at the `hoggQ` column. `claude-code` is at
`170.99`. `vscode-cp` is at `3047.04`. The HFR 1975 dispatch
cutoff for "extreme upper-tail dominance, route to the
median test" is `2.0`. These two sources are, respectively,
about 85x and 1500x past that cutoff.

To put `Q ≈ 170` in plain English: the mean of the top 5%
of pooled days is roughly 170 times farther above the
trimmed-middle-50% mean than the trimmed-middle-50% mean is
above the bottom 5%. The pooled empirical distribution is
not heavy-right-tailed. It is heavy-right-tailed by orders
of magnitude. For `vscode-cp` at `Q ≈ 3047`, the asymmetry
is closer to four orders of magnitude.

What that means structurally is that the daily total_tokens
series on these sources is dominated by a very small number
of spike days that pull the upper-5% mean up dramatically,
while the lower-5% mean and the trimmed-middle-50% mean both
sit in a narrow band near zero or near a small floor.
`claude-code`'s tenure of 72 days with 35 active days
(active rate ≈ 49%) tells you about half the days have
zero or near-zero usage; the other half includes a tail of
high-usage days. `vscode-cp`'s 265-day tenure with 73 active
days (active rate ≈ 28%) is even more skewed — three
quarters of the days are zero or near-zero, the other
quarter includes spikes of orders-of-magnitude larger than
median active-day usage.

## Why the dispatch decision matters for these specific sources

The `dispatch:` summary line says
`HFR1-mood-median=2  HFR2-wilcoxon=0  HFR3-vanderwaerden=0`.
The selector saturates: every qualifying source routes to
the same dispatched test. That uniformity is itself a
substantive finding about the pew corpus.

If we had run any non-adaptive single-score location test
on this corpus, two failure modes were on the table:

1. **Wilcoxon rank-sum** (axis-110 Mann-Whitney) is
   asymptotically efficient near the logistic distribution
   (medium-tail, symmetric). On this corpus, rank-sum is
   heavily influenced by the masses of tied-low days,
   because pooled mid-ranks of identically-zero days all
   sit at the same rank value. The few spike days dominate
   the rank-sum statistic, but the rank assignment dilutes
   their influence relative to the count of tied-low days.
   Net effect: signal exists but is muted, with
   undercoverage in the "real spike, but lots of zeros"
   regime that pew daily-token series live in.

2. **Van der Waerden normal-scores** (axis-181) is
   asymptotically efficient under normality. Normal-scores
   assigns near-zero weights to mid-rank values and
   moderate weights to extreme ranks. On this corpus, the
   normal-scores transformation flattens the tail
   asymmetry: the spike days get rank weights driven by
   `Φ⁻¹((rank)/(n+1))`, which grows logarithmically in
   `n`. For `n ≈ 70`, the highest rank gets a weight of
   roughly `Φ⁻¹(0.986) ≈ 2.20`. That's not enough to
   register the actual magnitude difference between a
   million-token spike day and a zero day.

Mood's median test, by contrast, only asks "is this day
above or below the pooled median?". On a corpus where the
pooled median is itself near-zero (because ≥ 50% of days
are inactive), every active day is above-median, and
above-median count by half (first vs second) becomes a
clean signal of "did the second half have more
above-median days than the first?". That's exactly what HFR
1975 tells you to do when `Q > 2.0`: stop trying to weight
the magnitudes, just count above-vs-below.

The two `hoggZ` results then read straight off:

- `claude-code` `hoggZ = +2.5757`, `p = 1.0005e-02`. The
  second 36-day half has STRICTLY MORE above-pooled-median
  days than the first 36-day half. This is the first half
  (Feb 11 – ~Mar 18) being the lower-activity period and
  the second half (~Mar 19 – Apr 23) being the
  higher-activity ramp.

- `vscode-cp` `hoggZ = -2.0965`, `p = 3.6041e-02`. The
  second 133-day half has STRICTLY FEWER above-pooled-
  median days than the first 132-day half. Sustained
  decline over the 265-day window.

Both are decisive at α = 0.05. Both have OPPOSITE signs.
That sign-opposition is what makes the cross-source
aggregation interesting: aggregating these two with a
naive average would cancel signal entirely. The Stouffer
corpus aggregator that axis-204 ships specifically takes
signed `hoggZ` per source and combines them with weights,
which preserves the sign-opposition rather than burying it.

## The structural orthogonality claim

The CHANGELOG explicitly enumerates what axis-204 is
orthogonal to. Three claim-classes are worth pulling out:

**Orthogonal to fixed-score location tests.** Axis-110
Mann-Whitney, axis-181 Van der Waerden, axis-171 Mood's
median: each uses a SINGLE FIXED score function on every
input regardless of underlying tail shape. Axis-204 uses a
DATA-DRIVEN selector `Q` to CHOOSE which score function to
apply. Two sources with identical Mann-Whitney `Z` but
different tail weights will get different `hoggZ`. This is
not a refinement of the prior axes — it is structurally a
different probe.

**Orthogonal to scale tests.** Axis-201 Kamat-range-ratio,
axis-200 Mielke quartic, axes 117/170/177-179/199 scale
tests all probe SCALE alternatives. Axis-204 probes
LOCATION. Orthogonal by alternative.

**Orthogonal to BWS joint location-scale.** Axis-185
Baumgartner-Weiss-Schindler combines location and scale
into a SINGLE QUADRATIC-WEIGHT statistic and CANNOT TELL
YOU which family of alternative is most consistent with
the data. Axis-204 isolates LOCATION while routing through
a tail-weight selector and makes the selection EXPLICIT
and EXPOSED. The dispatch label IS the new diagnostic
channel that BWS's quadratic-weight collapse hides.

This last point is the most interesting one for downstream
classifier work. BWS conflates "location shifted" and
"scale shifted" into one quadratic statistic that flags both
cases. Axis-204 says "location shift only, but here is
WHICH score function we used to detect it". The dispatch
label is then a per-source FEATURE that can be cross-
tabulated against tail-weight axes (axis-176 Hampel outlier
count, axis-179 Mood scale, axis-200 Mielke quartic). That
cross-tab IS the `byDispatch` output of the `v0.6.508`
compound classifier.

## The v0.6.508 compound: dispatch-stratified agreement table

The `v0.6.508` compound (`classifyHoggAdaptiveMannWhitney
DispatchAgreementCompound`, commit `2e2ceea`) joins
axis-204 with axis-110 on the SAME first-half-vs-second-
half partition. Both target two-sample LOCATION shifts on
the same data. The structural difference is only in the
score function:

- Mann-Whitney always uses the FIXED Wilcoxon rank-sum
  score (linear in pooled mid-ranks).
- Hogg-Adaptive routes through `Q` to one of three
  different score functions: median (HFR1), Wilcoxon
  (HFR2), or normal-scores (HFR3).

When `hoggDispatch === 'HFR2-wilcoxon'`, the two probes use
IDENTICAL score functions and SHOULD agree exactly up to
the affine difference between Mann-Whitney's U-form and
Wilcoxon rank-sum's W-form standardisation. When
`hoggDispatch === 'HFR1-mood-median'` or
`'HFR3-vanderwaerden'`, the two probes apply DIFFERENT
WEIGHT FUNCTIONS to the same data and can legitimately
DIVERGE: the adaptive selector says "the pooled tail shape
makes Wilcoxon SUB-OPTIMAL here, use a different score".
The compound exposes this divergence as a primary
diagnostic channel.

The seven buckets are
`coherent-location-wilcoxon-dispatch`,
`coherent-location-non-wilcoxon-dispatch`,
`adaptive-only`, `wilcoxon-only`, `direction-conflict`,
`dispatch-veto`, `no-evidence`. The headline output is the
`byDispatch` cross-tab — per-dispatch bucket counts.

What that cross-tab DIRECTLY ANSWERS is: "did the adaptive
selector vetoing Wilcoxon cause us to MISS or FIND a
location shift that the fixed Mann-Whitney test would have
called the other way?". On the live-smoke corpus, with
both qualifying sources dispatched to HFR1-mood-median, any
disagreement between `hoggZ` and `mwZ` lands in the
`coherent-location-non-wilcoxon-dispatch` bucket if signs
agree, the `direction-conflict` bucket if signs disagree,
or the `dispatch-veto` bucket if Mann-Whitney rejects but
Hogg-Adaptive does not.

The `dispatch-veto` bucket is the most diagnostically
interesting one. It corresponds to the case where the
fixed Wilcoxon test says "yes, location shift" (because
the rank-sum is dominated by the count of tied-low days
shifted between halves), but the adaptive selector says
"the pooled tail is so asymmetric that Wilcoxon is the
wrong score function, and the median test on the actually-
appropriate score function does NOT reject". That bucket
is the one to watch for on this corpus. It tells you
which sources have a Mann-Whitney signal that is an
artefact of the rank-sum's sensitivity to tied-low day
counts rather than a genuine median shift.

## What this means going forward

Three takeaways from axis-204 + `v0.6.508`:

1. **The selector saturation is itself a finding.** When
   `Q` returns 170 and 3047 on two sources with otherwise
   very different tenure (72 vs 265 days) and very
   different active rates (49% vs 28%), the conclusion is
   that the pew daily-token corpus has a UNIFORMLY EXTREME
   right-tail-asymmetry signature. That uniformity matters
   for every downstream classifier: any test that assumes
   medium-tail symmetry is operating outside its
   asymptotic-efficiency regime on this corpus. Mood's
   median test on this corpus is not just a robustness
   choice — it is the only test in the HFR menu that the
   selector will route to.

2. **The dispatch label is the new feature.** Every prior
   axis exposes a per-source `Z` and `pValue`. Axis-204
   exposes a per-source DISPATCH LABEL in addition. That
   label is now joinable against tail-weight axes
   (axis-176 Hampel, axis-179 Mood scale, axis-200 Mielke)
   and against scale tests (axis-201 Kamat). Future
   compounds can cross-tab dispatch-label against
   tail-weight bucket and ask "do sources where `Q`
   selects HFR1 always have axis-179 mood scale rejection
   too?" or "is there a source where `Q` selects HFR3
   despite axis-176 Hampel showing extreme outliers?".
   Those are new diagnostic questions that no fixed-score
   axis can ask.

3. **The v0.6.508 compound is the right way to consume
   axis-204.** Reading `hoggZ` in isolation tells you what
   the dispatched test said. Reading `hoggZ` jointly with
   `mwZ` and the `byDispatch` cross-tab tells you whether
   the dispatch CHOICE is doing real work or whether it's
   just a re-dressed Wilcoxon. On this corpus, with both
   sources at HFR1, the choice is doing real work: the
   median test ignores the magnitude of the spike days and
   only counts above-median days, which is the right
   thing to do when `Q` is in the hundreds-to-thousands
   range. On a corpus where `Q` lands closer to 1 across
   sources, the compound would degenerate to "Mann-Whitney
   under a different name" and the `coherent-location-
   wilcoxon-dispatch` bucket would dominate.

The W17 cycle's location-test family arc was: axis-110
fixed-Wilcoxon → axis-181 fixed-normal-scores → axis-171
fixed-median → axis-185 BWS quadratic-weight collapse →
axis-186 Hodges-Lehmann point estimate → axis-187/188
permutation-Welch live-smoke. Axis-204 is the first axis
that DOESN'T pick a fixed score function up front. It is
the first ADAPTIVE axis. That changes the kind of
question the location-test family can answer. The axis-204
+ axis-110 compound (`v0.6.508`) is the first compound
that exposes the score-function CHOICE as a feature, and
that capability is what enables the next layer of
classifier work.

The two `hoggZ` numbers on the live corpus —
`claude-code +2.58 / p = 1.00e-2` and `vscode-cp -2.10 /
p = 3.60e-2` — are then not just a sign-opposition
finding (the second half ramped on one source and
declined on the other). They are a sign-opposition
finding produced by a score function that the data itself
selected, and that selection is itself a documented
per-source feature. The ramp-vs-decline on these two
sources is real, the dispatch-saturation across both is
real, and the resulting compound bucket assignment is
the first time the codebase can say "this is a location
shift detected via the right score function for this
particular tail shape", rather than "this is a location
shift detected via whatever score function we picked at
axis design time".
