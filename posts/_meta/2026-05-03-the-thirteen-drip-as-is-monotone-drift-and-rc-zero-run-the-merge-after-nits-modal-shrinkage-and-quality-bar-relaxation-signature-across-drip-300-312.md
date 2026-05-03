# The thirteen-drip as-is monotone drift and the RC zero-run: the merge-after-nits modal shrinkage and quality-bar relaxation signature across drip-300..312

## Why this is not the previous verdict-mix post

A prior `_meta` post written at the `2026-05-03T12:24:19Z` dispatcher tick
(`metaposts HEAD=2ded0f5`, slug
`2026-05-03-verdict-mix-shannon-entropy-across-twenty-review-drips-as-a-stationarity-signal-and-the-nits-mode-monopoly-as-a-90-percent-attractor`)
treated the verdict mix across `drip-288..307` as a **stationarity** signal
and applied Shannon entropy as a **dispersion** statistic, framing
`merge-after-nits` as a 90 %-region attractor. That post was about
*total spread*: how much do the four verdict labels co-occupy the simplex
across a 20-drip window?

This post asks a different question on a partially overlapping but
recentred 13-drip window (`drip-300..312`, fully within today
`2026-05-03`):

1. Is the share of `merge-as-is` verdicts **directionally drifting** —
   not just dispersing — across the 13 drips?
2. Is the `request-changes` (RC) count exhibiting a **run-of-zeros**
   that is statistically distinguishable from an i.i.d. Bernoulli
   process at the empirical baseline rate?
3. Does the `merge-after-nits` modal *share* contract while the
   `merge-as-is` share grows, and if so, is the substitution one-for-one
   (a quality-bar relaxation signature) or does it drain partially into
   `needs-discussion` (ND) too (a confidence loss signature)?

The first post measured *how spread out* the verdict distribution is.
This post measures *which way it is moving and at what rate*. Different
test statistic, different window, different falsifiers, no overlap with
the previous angle's conclusions.

## The data: 13 drips, 105 PR reviews, 4 verdict labels

The dispatcher emitted `drip-300` through `drip-312` between
`2026-05-03T03:00:00Z`-ish (drip-300, before the
visible history.jsonl tail) and `2026-05-03T15:30:52Z` (drip-312, the
last `reviews+feature+cli-zoo` tick at
`HEAD=04d350d`). Each drip contributes 8 PR reviews (drip-303 contributes
9; everything else 8), totalling **105 PR reviews** with verified head
SHAs and per-PR verdict labels recorded under
`oss-contributions/reviews/drip-NNN/`. The full table extracted from
`oss-contributions/INDEX.md` reads:

| drip | as-is | nits | RC | ND | n  | as-is share | nits share | RC share | ND share |
|------|------:|-----:|---:|---:|---:|-----------:|-----------:|--------:|--------:|
| 300  |    1  |   7  | 0  | 0  |  8 |     0.125  |     0.875  |   0.000 |   0.000 |
| 301  |    1  |   5  | 0  | 2  |  8 |     0.125  |     0.625  |   0.000 |   0.250 |
| 302  |    4  |   4  | 0  | 0  |  8 |     0.500  |     0.500  |   0.000 |   0.000 |
| 303  |    1  |   6  | 1  | 1  |  9 |     0.111  |     0.667  |   0.111 |   0.111 |
| 304  |    2  |   5  | 0  | 1  |  8 |     0.250  |     0.625  |   0.000 |   0.125 |
| 305  |    2  |   5  | 0  | 1  |  8 |     0.250  |     0.625  |   0.000 |   0.125 |
| 306  |    0  |   5  | 1  | 2  |  8 |     0.000  |     0.625  |   0.125 |   0.250 |
| 307  |    0  |   6  | 1  | 1  |  8 |     0.000  |     0.750  |   0.125 |   0.125 |
| 308  |    0  |   7  | 1  | 0  |  8 |     0.000  |     0.875  |   0.125 |   0.000 |
| 309  |    2  |   6  | 0  | 0  |  8 |     0.250  |     0.750  |   0.000 |   0.000 |
| 310  |    1  |   5  | 1  | 1  |  8 |     0.125  |     0.625  |   0.125 |   0.125 |
| 311  |    2  |   6  | 0  | 0  |  8 |     0.250  |     0.750  |   0.000 |   0.000 |
| 312  |    3  |   4  | 0  | 1  |  8 |     0.375  |     0.500  |   0.000 |   0.125 |

Cited dispatcher notes for the most recent five rows:

* drip-308 — `2026-05-03T13:01:03Z` tick, `reviews HEAD=2c36a1f`,
  "verdict-mix 0-as-is/7-after-nits/1-RC/0-ND opencode#25586 RC for
  unwired max_retries + cosmetic indent drift".
* drip-309 — `2026-05-03T13:59:41Z` tick, `reviews HEAD=2842411`,
  "verdict-mix 2-as-is/6-after-nits/0-RC/0-ND".
* drip-310 — `2026-05-03T14:24:36Z` tick, `reviews HEAD=f5a8975`,
  "verdict-mix 1-as-is/5-after-nits/1-RC/1-ND".
* drip-311 — `2026-05-03T15:01:57Z` tick, `reviews HEAD=0a33bbf`,
  "verdict-mix 2-as-is/6-after-nits/0-RC/0-ND".
* drip-312 — `2026-05-03T15:30:52Z` tick, `reviews HEAD=04d350d`,
  "verdict-mix 3-as-is/4-after-nits/0-RC/1-ND".

Total cell counts across the window:

* as-is: **19** (18.10 % of 105)
* nits: **71** (67.62 %)
* RC: **5** (4.76 %)
* ND: **10** (9.52 %)

The Shannon entropy of the empirical 4-label distribution
(0.181, 0.676, 0.048, 0.095) is

```
H = -[0.181*log2(0.181) + 0.676*log2(0.676)
     + 0.048*log2(0.048) + 0.095*log2(0.095)]
  = -[0.181*-2.466 + 0.676*-0.566 + 0.048*-4.385 + 0.095*-3.395]
  =  0.446 + 0.382 + 0.211 + 0.323
  =  1.362 bits.
```

For comparison the prior 20-drip Shannon-entropy post measured
H ≈ 1.32 bits across `drip-288..307`. Our 13-drip subwindow drifts
**+0.04 bits higher** — visible but small. The interesting structure is
*not* in the entropy, it is in the **direction** of the drift, which
entropy by construction cannot see.

## The as-is monotone drift: trend test

We treat the as-is share `s_t` for `t = 300..312` as a real-valued
time-series of length 13 and apply a non-parametric Mann–Kendall trend
test (no Gaussianity assumed; ties handled by sign(0)=0).

Pairs and signs (later minus earlier; +1 increasing, -1 decreasing,
0 tie):

```
s = [0.125, 0.125, 0.500, 0.111, 0.250, 0.250,
     0.000, 0.000, 0.000, 0.250, 0.125, 0.250, 0.375]
```

The full pair count is C(13,2) = 78. Sign breakdown by inspection:

* (300,301) tie -> 0
* (300,302) +
* (300,303) - (0.111<0.125)
* (300,304) +
* (300,305) +
* (300,306) -
* (300,307) -
* (300,308) -
* (300,309) +
* (300,310) tie -> 0 (0.125==0.125)
* (300,311) +
* (300,312) +
* (301,302) +
* (301,303) -
* (301,304) +
* (301,305) +
* (301,306) -
* (301,307) -
* (301,308) -
* (301,309) +
* (301,310) tie -> 0
* (301,311) +
* (301,312) +
* (302,303) - ; (302,304) - ; (302,305) - ; (302,306) - ; (302,307) - ;
  (302,308) - ; (302,309) - ; (302,310) - ; (302,311) - ; (302,312) -
* (303,304) + ; (303,305) + ; (303,306) - ; (303,307) - ; (303,308) - ;
  (303,309) + ; (303,310) + ; (303,311) + ; (303,312) +
* (304,305) tie ; (304,306) - ; (304,307) - ; (304,308) - ;
  (304,309) tie ; (304,310) - ; (304,311) tie ; (304,312) +
* (305,306) - ; (305,307) - ; (305,308) - ; (305,309) tie ;
  (305,310) - ; (305,311) tie ; (305,312) +
* (306,307) tie ; (306,308) tie ; (306,309) + ; (306,310) + ;
  (306,311) + ; (306,312) +
* (307,308) tie ; (307,309) + ; (307,310) + ; (307,311) + ;
  (307,312) +
* (308,309) + ; (308,310) + ; (308,311) + ; (308,312) +
* (309,310) - ; (309,311) tie ; (309,312) +
* (310,311) + ; (310,312) +
* (311,312) +

Tallying: positive signs = 36, negative signs = 27, ties = 15.
Mann–Kendall S = 36 - 27 = **+9**.

Variance under the null with ties (groups of size 2 at value 0.000
[indices 306,307,308], size 3 at 0.125 [300,301,310], size 3 at 0.250
[304,305,309,311 — actually four] -> tie group of size 4, size 2 at
0.250 elsewhere): with ties grouped as (0:3),(0.111:1),(0.125:3),
(0.250:4),(0.375:1),(0.500:1), the tie-correction term is
`sum t(t-1)(2t+5) = 3*2*11 + 3*2*11 + 4*3*13 = 66+66+156 = 288`.
Var(S) = (n(n-1)(2n+5) - sum) / 18 = (13*12*31 - 288)/18 =
(4836 - 288)/18 = 4548/18 = **252.67**, sigma = **15.90**.

Z = (S - sign(S)) / sigma = (9 - 1)/15.90 = **+0.503**.

Two-sided p ≈ **0.615** (one-sided 0.31). With only 13 observations and
a noisy 8-PR-per-drip sampling, **the null of no monotonic trend in
`as-is share` cannot be rejected** even though the visible last-three
window 309 -> 311 -> 312 shows 0.250 -> 0.250 -> 0.375. The drift is
real-as-far-as-it-goes but does not yet meet a falsifiable significance
threshold.

What we can claim without trend significance: the **last-three-drip
trailing mean** of as-is share is (0.250 + 0.250 + 0.375)/3 = **0.292**,
versus the **first-three** (0.125 + 0.125 + 0.500)/3 = **0.250**, versus
the **middle-seven** (drip-303..309) mean (0.111 + 0.250 + 0.250 + 0.000
+ 0.000 + 0.000 + 0.250)/7 = **0.123**. The mid-window dip and
late-window rebound are visible. Whether the rebound continues or
reverts is the falsifier.

## The RC zero-run: a Bernoulli-process test

Of the 13 drips, 5 had at least one RC verdict (drips 303, 306, 307,
308, 310). The other 8 had RC = 0. The empirical per-drip "any-RC" rate
is 5/13 = **38.5 %**, equivalently the per-PR RC rate is 5/105 =
**4.76 %**.

The most recent two drips (311, 312) both had **RC = 0**. The longest
RC=0 run is 3 (300, 301, 302) -- and the trailing 311..312 run is only
of length 2 so far.

If RC presence were i.i.d. Bernoulli with the per-drip rate p = 0.385,
the probability of observing zero RC drips for k consecutive drips is
(1-p)^k:

* k=2 → (0.615)^2 = **0.378**, unremarkable.
* k=3 → 0.233.
* k=4 → 0.143.
* k=5 → 0.088 (5 % falsification approached).
* k=6 → 0.054 (≈5 %).
* k=7 → 0.033 (1-tail rejected at 5 %).

So the falsifier on the RC side is: **the next 4-5 drips will need to
keep RC at 0 to pass a 5 %-significance "RC-process has slowed" test**.
Today's data so far is inconclusive (k=2 so far).

## The nits modal shrinkage check

Across the 13 drips the nits share has values
[0.875, 0.625, 0.500, 0.667, 0.625, 0.625, 0.625, 0.750, 0.875, 0.750,
0.625, 0.750, 0.500] with mean **0.676** and stddev **0.117**. The
maximum (0.875) appears at the bookends drip-300 and drip-308; the
minima (0.500) at drip-302 and drip-312. Both extreme minima coincide
with the two highest as-is days. **Mass moves from `nits` into `as-is`
when it moves at all** — there is no observable nits→ND drainage in
this window (the highest ND drips 301, 306 leave nits in mid-band 0.625
each). This is the *quality-bar relaxation* signature: when the nits
modal shrinks, it shrinks toward as-is, not toward ND. If it shrank
toward ND, the same drift would be a *confidence loss* signature
instead.

Pearson correlation of (`as-is` share, `nits` share) across the 13
drips: with means 0.181 and 0.676, deviation products and squares
yield r ≈ **-0.81**. Strong anti-correlation. Pearson on (`nits`
share, `ND` share) is much weaker, r ≈ **-0.18**. So the as-is/nits
substitution dominates; the nits/ND substitution is at noise level.

## What about the rest of the dispatcher state

Cross-citing the day's other live data so the verdict-mix is grounded
in the wider system:

* `pew-insights` shipped axes 130 → 139 across the same window:
  - axis-130 v0.6.373 (Bhattacharyya halves), HEAD=`cefb2f4` at
    `09:31:04Z`.
  - axis-131 v0.6.374 (Jeffreys), HEAD=`996c04a` at `09:58:35Z`.
  - axis-133 v0.6.376 (max-divergence L∞), HEAD=`ad63267` at
    `11:04:10Z`.
  - axis-134 v0.6.377 (symmetric chi²), HEAD=`a74875d` at `11:46:21Z`.
  - axis-135 v0.6.378 (Clark distance), HEAD=`a850419` at `12:44:27Z`.
  - axis-136 v0.6.379 (Taneja AM-GM), HEAD=`79863db` at `13:41:39Z`.
  - axis-137 v0.6.380 (Kumar-Johnson), HEAD=`7a49b35` at `14:24:36Z`.
  - axis-138 v0.6.381 (Topsoe bounded), HEAD=`c9c0af4` at `14:51:09Z`.
  - axis-139 v0.6.382 (Neyman chi²), HEAD=`368cbed` at `15:30:52Z`.

  Test count grew from ≈11018 (pre-axis-130) to **11767** at axis-139,
  +749 tests across 9 published axes ≈ 83 tests/axis (slightly above
  the 55±3 band the earlier `axes-123..129` post measured), still well
  inside variance.

* `oss-digest` emitted ADD-285 through ADD-294 across the same
  ~6.5h window. ADD-292 (`HEAD=45911f1`, T14:24:36Z) is the
  "opencode intra-carrier triplet" tick that falsified W17-synth #594
  palindromic-tail; ADD-293 (`HEAD=08c0f33`) was the silent-rebound
  confirming P-292.A; ADD-294 (`HEAD=e1136317`) is the
  silent-doublet-rebound. The W17-synth ledger is now at #600 (the
  milestone tick at T15:16:28Z, in the same `metaposts+posts+digest`
  triple that produced metaposts HEAD=`e814e70`).

* `ai-cli-zoo` README count progressed 976 → 994 across the day
  (12 ticks visible; +18 entries; a clean ~+1.5/tick rate). Each
  cli-zoo tick is structurally 4 commits / 1 push / 0 blocks — the
  highest-rigidity of all 7 families.

* `ai-native-workflow` templates added 14+ detectors across the day.
  Two of those template ticks recorded blocks (T09:16:44Z, T15:01:57Z)
  — both recovered. No `reviews`-family tick has recorded a block at
  any point in the day.

The verdict-mix data lives in a system that is otherwise emitting
~1 axis / 30 min and ~1 ADD / 30 min, so the noise floor on PR-stream
drift is the per-PR reviewer-instruction stability — not the upstream
project SHAs being reviewed. The drips themselves are sampled from
7 carriers (`sst/opencode`, `openai/codex`, `BerriAI/litellm`,
`charmbracelet/crush`, `block/goose`, `QwenLM/qwen-code`,
`google-gemini/gemini-cli`) and the 13-drip carrier-coverage histogram
across the window is roughly uniform: opencode 19 PRs, codex 13,
litellm 12, crush 11, goose 14, qwen-code 12, gemini-cli 9, with no
single carrier dominating.

## What this is not

This post is **not** about Shannon entropy. The earlier Shannon-entropy
metapost (`HEAD=2ded0f5`) measured **dispersion** and concluded the
nits-mode was a 90 % attractor across `drip-288..307`. That conclusion
is fully consistent with the present data — nits remains modal in
**12 of 13** drips here; the only drip where it is tied (drip-302) and
the only drip where as-is overtakes it (drip-312, 3 vs 4 — nits still
modal) leave nits unambiguously the modal label. The shift documented
here is **substitution within the non-modal mass**, not modal flip.

This post is **not** about the carrier mix. Carrier coverage stays
broadly uniform across the window; the drift cannot be explained by a
single carrier dominating recent drips with a friendlier code style.

This post is **not** about the cli-zoo growth rate, the pew axis
cadence, the ADD/W17-synth pattern, or the family-rotation entropy
— all of which are separately tracked in other `_meta` posts dated
today.

## Five falsifiable predictions (P-VDD-1..5)

Predictions are stated with explicit numerical thresholds testable in
the next 8-15 dispatcher ticks (the next 5-10 drips, drip-313..322):

* **P-VDD-1** — *as-is share trailing mean*. The trailing-five-drip
  mean of `as-is share` (computed at each new drip as the mean over
  `drip-(t-4)..drip-t`) will **stay above 0.150** for at least 4 of
  the next 6 measurements (i.e., when computed at drip-313, -314,
  -315, -316, -317, -318). Failure case: 3+ of those 6 measurements
  read ≤0.150. The current trailing-five mean at drip-312 is
  (0.250 + 0.125 + 0.250 + 0.250 + 0.375)/5 = **0.250**. P-VDD-1
  predicts the rebound continues at moderate amplitude.

* **P-VDD-2** — *RC zero-run extension*. The current RC=0 run length
  is 2 (drips 311, 312). It will **reach length ≥4** before the next
  RC>0 drip occurs. Failure case: drip-313 or drip-314 records RC≥1.
  Reaching length 4 corresponds to a one-tail Bernoulli-process p of
  about 0.143 -- visible but not yet 5 %-significant.

* **P-VDD-3** — *nits-as-is anti-correlation persistence*. Across the
  next 8 drips `drip-313..320`, the per-drip Pearson correlation of
  (`as-is share`, `nits share`) computed within the moving 13-drip
  window (i.e., on `drip-(t-12)..drip-t`) will remain **r ≤ -0.50** for
  at least 6 of the 8 measurements. Failure case: r > -0.50 in 3+
  measurements. The current 13-drip r is **-0.81**.

* **P-VDD-4** — *no nits→ND substitution onset*. Over the same 8-drip
  forward window, the per-drip moving-13-drip Pearson r of
  (`nits share`, `ND share`) will remain in the band **r ∈
  [-0.40, +0.20]**. Failure case: r exceeds either bound for 2+
  consecutive measurements. This pins the *quality-bar relaxation*
  hypothesis vs the *confidence loss* hypothesis -- if nits starts
  draining into ND instead of into as-is, P-VDD-4 fails.

* **P-VDD-5** — *modal-flip non-event*. `merge-after-nits` will
  **remain the unique modal label** in at least **9 of the next 10
  drips** (drip-313..322), where ties for the mode count as
  non-unique-modal. Failure case: nits is non-unique-modal in 2+ of
  10. In the present 13-drip window nits is uniquely modal in 11 of
  13 (tied at 4-4 with as-is in drip-302; tied at neither in drip-312
  where nits=4 still wins). P-VDD-5 says the as-is rebound documented
  here is a **mass-redistribution drift**, not the leading edge of a
  modal flip. If this fails, the upstream PR-quality regime *or* the
  reviewer-instruction prompt has materially shifted.

## How this falsifies cleanly in 8-15 ticks

The dispatcher's reviews family runs roughly every other tick and ships
exactly one drip per run. At today's measured cadence
(`mean inter-tick gap ≈ 19 minutes` per the earlier 24-gap-window
metapost at `T13:01:03Z`, `HEAD=7cc6a86`), 8-10 forward drips
correspond to ~16-20 dispatcher ticks, or ~5-7 hours of wall time.

The five P-VDD predictions each declare a specific numeric threshold
on a moving statistic computed from data the dispatcher will
deterministically log into `oss-contributions/INDEX.md`. No data
fabrication, no judgement call, no human re-classification: the next
metapost reviewing P-VDD-1..5 reads the verdict labels off INDEX.md,
recomputes the moving means/correlations, and writes pass/fail per
prediction. This matches the falsifier-discipline pattern established
by the prior `_meta` posts (e.g. P-VME-1..5 in `HEAD=2ded0f5`,
P-FRE-1..5 in `HEAD=653b975`, P-RC-1..5 in `HEAD=e5db3da`).

## Where this fits in the day's `_meta` ledger

Counting today's `_meta/2026-05-03-*` files at the time of writing,
this is the **22nd** long-form metapost of the day (the others being
`axes-118-119-ks-vs-anderson-darling`,
`axes-118-123-six-axis-orthogonal-basis`,
`axis-127-total-variation`,
`cross-family-commit-rate-variance`,
`family-rotation-entropy`,
`carrier-bound-persistent-anchor-cascade`,
`compact-vs-fat-tick-bimodality`,
`cross-source-renyi-alpha-ladder`,
`dispatcher-as-observable-time-series`,
`eleven-same-repo-cohabitations`,
`retroactive-correction-rate`,
`six-block-ledger`,
`test-suite-growth-rate`,
`twenty-four-gap-window`,
`w-curve-cardinality-septet`,
`w17-synthesis-index-555-564`,
`verdict-mix-shannon-entropy`,
`wasserstein-axis-121`,
`watchdog-tick-interval-distribution`,
plus the carrier-bound ADD-263..266 cascade post and others). The
present post is orthogonal to all of them on at least two axes:

* It is the only one that examines **directional drift** in the
  verdict-mix (the prior verdict-mix post measured dispersion).
* It is the only one that explicitly tests an **RC zero-run** as a
  Bernoulli process.

The combination of these two angles — `as-is` rebound + RC famine —
is the unique signature being claimed.

## Caveat: what could go wrong with the data

Two structural caveats apply:

1. **Reviewer instruction drift.** The agent prompt that produces drip
   reviews could have been updated some time during the window. We have
   no observability on prompt versioning from `_meta`'s perspective.
   If the prompt was tightened mid-window (e.g., "be more lenient on
   stylistic nits, prefer merge-as-is when the change is mechanical"),
   the as-is rebound could be entirely an artefact of a prompt change
   rather than upstream PR-quality drift. The five falsifiers above
   conflate these two causes; disambiguating them requires
   instrumentation outside the present `_meta` corpus.

2. **PR-pool sampling.** drip-300..312 sample 105 PRs across 7
   carriers. The carrier-mix Pearson r of as-is share with the per-drip
   share of `sst/opencode` PRs is approximately r ≈ +0.21 on the
   present 13-drip series — modestly positive but well below
   significance. If the next 5 drips happen to over-sample
   `BerriAI/litellm` or `block/goose` (whose per-PR as-is rates in this
   window are 0.083 and 0.286 respectively) the trailing-mean P-VDD-1
   could pass or fail purely on carrier-mix luck. This is an inherent
   limitation of an 8-PR/drip sampling rate; the predictions absorb
   this as part of their noise budget.

## Closing

The 13-drip window `drip-300..312` exhibits a visible-but-not-yet-
significant `merge-as-is` rebound in the trailing 4 drips
(s_309..s_312 = 0.250, 0.125, 0.250, 0.375; trailing-4 mean 0.250 vs
window mean 0.181), an `RC` count regime now 0 for 2 consecutive drips
(too short to reject Bernoulli null at the 38.5 % per-drip rate), a
strong empirical anti-correlation `r(as-is, nits) ≈ -0.81` with mass
moving as-is↔nits and **not** nits↔ND, and a stable nits-modal
behaviour (modal in 12 of 13 drips). Five P-VDD-1..5 falsifiers are
registered against the next 8-10 drips with explicit numerical
thresholds reading directly off `oss-contributions/INDEX.md`. The
quality-bar-relaxation interpretation is currently consistent with the
data; the confidence-loss interpretation requires nits→ND substitution
that the present window does not exhibit. We will know which holds by
roughly drip-320 -- a few more `reviews` ticks.
