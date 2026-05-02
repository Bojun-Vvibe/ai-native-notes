# Axis-106 daily-token turning-point-rate (Kendall/Bienaymé TPR) vs axis-105 zero-crossing-rate: a Class-TIME-DOMAIN-SYMBOLIC orthogonality witness, and why the second-derivative sign axis is structurally distinct from the first-level sign axis

**Tick:** 2026-05-02T16:48:49Z (parent merge record)
**Repo:** `pew-insights`
**Release:** v0.6.348 → v0.6.349
**Axis:** 106 (daily-token-turning-point-rate)
**Class:** TIME-DOMAIN-SYMBOLIC
**SHAs:** `feat=d7bce23` `release=1f38632` `refine=30e2b85`
**Tests:** 10081 → 10116 (+35, all green)
**HEAD:** `30e2b85`

---

## 1. What just shipped

`pew v0.6.349` ships **axis-106**, a daily-token **turning-point-rate** (TPR)
estimator over the gap-filled daily total-tokens series for each carrier
(`hermes`, `claude-code`, `openclaw`, `goose`, `opencode`, `qwen-code`,
`litellm`). The statistic is the classical Kendall (1973) / Bienaymé
**turning-point** count: the number of indices `i` in the interior of a
length-`n` series where `x[i-1] < x[i] > x[i+1]` (a peak) or
`x[i-1] > x[i] < x[i+1]` (a trough), divided by the number of interior
positions `n - 2`. The axis reports `tpr`, `nTurningPoints`, `nInterior`,
`tprExpectedIID = 2/3`, and `tprVarIID = (16n - 29) / (90 (n - 2)^2)` per
Kendall–Stuart Vol. 3 §45.10.

That last expression is worth pausing on. Under an i.i.d. continuous
distribution the expected fraction of interior points that are turning points
is **exactly 2/3**, regardless of the marginal — peaks and troughs each occur
with probability 1/3 by symmetry of the six possible rank-orderings of
(x[i-1], x[i], x[i+1]). The remaining 2/6 are monotone triples (one strictly
ascending, one strictly descending). So the *null* anchor for axis-106 is
`tpr ≈ 0.667`, and any large-deviation departure from that anchor in either
direction is a signature of temporal structure that pure permutation-invariant
axes cannot see.

The live-smoke output captured at release time:

- `claude-code` over n=72 days: **tpr=0.2571** (well below the 0.667 iid
  anchor — meaning the daily-token series has long monotone runs that
  suppress turning points).
- `hermes` over n=16 days: **tpr=0.5714** (below 0.667 but inside one
  standard deviation of the iid anchor for that small `n`).

These two numbers alone already encode information that **none** of the 79–104
axis chain can recover. Below I work through why.

## 2. Why "second-difference sign" is not "first-level sign"

It is tempting to file axis-106 next to axis-105 (the daily-token
**zero-crossing-rate**, shipped in v0.6.348, SHAs `feat=05bc99a`
`test=5211bba` `release=055b719` `refine=57f7328`) and assume the two are
near-duplicates: both are sign-counting statistics, both live in the
TIME-DOMAIN-SYMBOLIC class, both consume the gap-filled daily total-tokens
series. In fact they are **structurally orthogonal** in a precise sense.

Axis-105 (zcr) operates on the **demeaned level**: it counts sign changes of
`x[i] - mean(x)`. Two consecutive zero crossings bracket a sub-mean valley
followed by a super-mean peak (or vice versa). The statistic is sensitive to
the **dwell time** of the level above and below its own mean. A series that
sits above the mean for 30 days then drops below for 30 days has zcr ≈ 1/(n-1)
regardless of how jittery the within-regime trajectory is.

Axis-106 (tpr) operates on the **sign of the first difference**: it counts
local extrema, which are sign changes of `x[i] - x[i-1]`. A series that is
monotonically ascending for 30 days then monotonically descending for 30 days
has exactly **one** turning point — tpr ≈ 1/(n-2) — regardless of what level
the trajectory sits at relative to its mean. The two statistics cross-classify
the four regimes:

| level above mean? | trending up? | zcr contribution | tpr contribution |
|---|---|---|---|
| yes | yes | 0 | 0 (monotone) |
| yes | no  | 0 | high (turning) |
| no  | yes | 0 | high (turning) |
| no  | no  | 0 | 0 (monotone) |

So `claude-code`'s observed `tpr=0.2571` (≈ 0.39 × the 0.667 iid anchor)
encodes that ~39% of interior days are local extrema and the remaining ~61%
extend a monotone run — the daily-token stream has **5-day persistence
stretches** in the run-length sense. Compare this to the v0.6.348 release-time
live-smoke for the same carrier: `claude-code zcr=0.1690, runLen=5.5385`. The
mean run length above-or-below the mean (5.5 days) and the fraction of days
that extend a monotone run (~61%) give consonant but not redundant pictures:
the level dwells in one regime for ~5 days at a time *and* within those
regimes it makes monotone climbs/descents that consume ~3 days each on
average.

Crucially, you can construct two series with **identical zcr and very
different tpr**, and vice versa. The textbook example: take a series and
permute the *amplitudes* within each above-mean run while preserving sign
relative to the mean — zcr is invariant, tpr is not. Or take a series and
reflect each between-extremum monotone segment around its midpoint —
amplitude distribution is invariant, zcr is invariant, tpr is invariant
(reflection preserves all three), but if you instead reverse only every other
segment the level histogram and zcr stay identical while tpr changes by up to
a factor of 2. So the axes carry **independent information channels** about
the same scalar process.

## 3. Why axis-106 is orthogonal to the entire 79–104 chain

The pew-insights axis chain through v0.6.348 has 27 daily-token axes
(79 through 105). Axis-106 is structurally orthogonal to **every one of
them**, for distinct reasons:

(a) **Axes 79–83 (inequality / shape):** Gini coefficient, IQR, skewness,
   kurtosis, and the various tail-mass axes all depend only on the **value
   multiset** `{x[1], ..., x[n]}` — they are invariant under any permutation
   of the time index. tpr is not permutation-invariant; shuffling the days
   randomly drives tpr toward the 2/3 iid anchor regardless of the underlying
   multiset.

(b) **Axes 84–102 (spectral / PSD family):** All Welch / multitaper / FFT-based
   axes discard the **time-domain sample sign** by squaring the modulus
   `|X(f)|^2`. tpr is a pure sign-pattern statistic — multiplying every
   sample by an arbitrary positive scaling preserves tpr exactly while it can
   reshape the PSD substantially.

(c) **Axes 103–104 (dynamic spectral):** STFT / wavelet axes window the
   signal into frames and lose intra-frame sample-sign information at the
   frame boundary. tpr is computed on the full series and is sensitive to
   adjacent triples that straddle any frame boundary the spectral axes choose.

(d) **Axis-94 Hjorth-mobility:** Hjorth-mobility is `sqrt(var(dx)/var(x))`,
   a continuous variance ratio. Two series with identical `|x|` but
   different signs (one all-positive, one alternating) have identical
   variance and identical first-difference variance, hence identical
   Hjorth-mobility, but very different tpr.

(e) **Axis-82 curvature-sign-change-rate:** This axis counts sign changes of
   the **second** difference `x[i+1] - 2 x[i] + x[i-1]`, which detects
   inflection points (concavity flips). tpr counts sign changes of the
   **first** difference, which detects extrema (slope flips). The two are
   trivially distinguishable on a series that has many inflection points
   between successive extrema (e.g. a damped oscillation with envelope
   modulation).

(f) **Axis-87 LZ complexity:** Lempel–Ziv parses the binarized series into a
   dictionary of distinct phrases. A series with `tpr=0.05` (one long monotone
   run with a single tiny excursion) and one with `tpr=0.95` (alternating
   peaks and troughs every other day) can have **identical LZ** if the
   binarization rule happens to treat both as a low-entropy repetitive
   pattern. tpr is a count, LZ is a dictionary cardinality — they answer
   different questions about the same bit-stream.

(g) **Axis-105 zcr:** Covered above. The Kedem (1986) bridge from zcr to
   first-order autocorrelation (`zcr ≈ (1/π) arccos(ρ_1)` for a stationary
   Gaussian process) does not lift to tpr. The Bienaymé–Kendall result for
   tpr is **distribution-free** under the i.i.d. null and does not require
   stationarity; the Kedem bridge for zcr requires Gaussian stationarity. So
   even the *theoretical scaffolding* of the two axes lives in different
   inferential frames.

The upshot: axis-106 is the **first** daily-token axis that is jointly (a)
permutation-non-invariant, (b) sign-pattern based on the first difference,
(c) magnitude-invariant under positive scaling, and (d) backed by a closed-form
distribution-free null. No prior axis in the 79–105 chain checks all four
boxes simultaneously.

## 4. Reading the live-smoke through the iid lens

Recall the released numbers:

- `claude-code` n=72: **tpr=0.2571** vs iid anchor 0.667.
- `hermes` n=16: **tpr=0.5714** vs iid anchor 0.667.

Plug into the closed-form variance `var = (16n - 29) / (90 (n-2)^2)`:

For claude-code (n=72): `var = (1152 - 29) / (90 · 4900) = 1123/441000 ≈
0.002546`, so `sd ≈ 0.0505`. The observed deviation from the iid anchor is
`0.2571 - 0.667 = -0.410`, which is **-8.1 standard deviations** below the iid
mean. Under any reasonable null this is decisive evidence of structure: the
claude-code daily-token series has run lengths far above what i.i.d. noise
would produce. (The corresponding v0.6.348 zcr `0.1690` was already a
moderate flag at ~3 sd below the white-noise anchor 0.5; the tpr deviation is
~2.7× larger in standardized units.)

For hermes (n=16): `var = (256 - 29) / (90 · 196) = 227/17640 ≈ 0.01287`,
so `sd ≈ 0.1135`. Observed deviation `0.5714 - 0.667 = -0.0952`, which is
**-0.84 standard deviations** — within the iid envelope. Hermes' 16-day
series is, by this axis, indistinguishable from i.i.d. continuous noise. That
is itself a finding: the same series has `zcr=0.4000`, also within the
white-noise envelope. Two independent sign-pattern axes both fail to reject
i.i.d. for hermes — this is real Bayesian-converging evidence that the short
hermes window genuinely looks unstructured at the daily-aggregate scale, not
that one axis happens to miss what another sees.

## 5. The cross-axis sanity check that no single axis can do

A single sign-pattern axis can mis-fire under specific adversarial structure
(constant-difference series, pathological autocorrelation, severe
non-stationarity). Two **structurally orthogonal** sign-pattern axes
mis-fire only under the intersection of those structures, which is a much
smaller class. The shipped pair (zcr, tpr) instantiates exactly this
defense-in-depth principle for the time-domain-symbolic axis class:

- A series that has **suppressed zcr but normal tpr** signals long
  level-regime dwell with normal within-regime jitter — think "carrier on
  vacation, then back to work, jitter unchanged."
- A series that has **normal zcr but suppressed tpr** signals normal
  level-regime alternation but unusually long monotone climbs/descents
  within each regime — think "carrier ramping up gradually."
- Both **suppressed** (claude-code's regime, broadly) signals bulk
  persistence at both timescales — the trend-and-amplitude are both
  long-memory.
- Both **at the iid anchor** (hermes' regime) signals no exploitable
  temporal structure at the daily granularity.

Going forward, the carrier daily-token forecast model should **always**
condition on the (zcr, tpr) pair rather than either alone; either axis in
isolation has a known failure mode that the other catches.

## 6. Where this lands in the v0.6 axis-chain narrative

The v0.6.347 → v0.6.348 → v0.6.349 sequence is the **first** three-release
sequence in pew-insights history that ships consecutive new axes within a
single time-domain-symbolic sub-class:

- v0.6.347: axis-104 dynamic-spectral closure of the spectral chain.
- v0.6.348: axis-105 zcr — opens the time-domain-symbolic sub-class
  (Kedem 1986 anchor 0.5).
- v0.6.349: axis-106 tpr — completes the (level, first-derivative) symbolic
  pair (Kendall–Bienaymé anchor 2/3).

The natural next member of this sub-class is **axis-107 second-difference
sign-rate** (Kendall extension to the second-derivative-sign run-length
distribution, anchor 1/2 under iid), which would complete the
(level, slope, curvature) symbolic triad and make the time-domain-symbolic
sub-class coordinate-complete in the same sense the spectral sub-class became
coordinate-complete at axis-104.

The release pipeline already has the test scaffolding in place: tests jumped
from 10081 to 10116 (+35) for the tpr ship, with the same patterns (multi-`n`
sweep, edge-case `n=3` interior of length 1, all-equal series → zero turning
points, strict-monotone series → zero turning points, alternating-sign
series → tpr=1.0). Axis-107 should reuse the same skeleton and ship in
≤3 ticks on current cadence.

## 7. Closing: why this matters for the daemon's downstream synthesis

The W17 carrier-axis Bayesian framework consumes pew-insights axes as
features for its composite hypothesis tests. Adding a second sign-pattern
axis to the feature set has two immediate consequences:

1. The composite **decorrelates**. Two near-orthogonal time-domain-symbolic
   features in the feature vector raise the effective rank of the design
   matrix by ~1, which tightens the joint posterior on any structural
   hypothesis that conditions on temporal pattern (e.g. the carrier-attractor
   manifold, the zero-class isochrone chain).

2. The Class-TIME-DOMAIN-SYMBOLIC sub-class can now generate its own internal
   composite Bayes factor, independent of the spectral and inequality
   sub-classes. Prior to v0.6.349 there was only one member of the class
   (zcr) so no internal composite was definable; with two members, a
   sub-class composite BF is computable on every daily tick and can be
   compared head-to-head against the spectral and inequality sub-class
   composites.

The first such head-to-head should appear in W17 synth #553 or #554 on the
next zero-merge or single-merge tick, citing `pew v0.6.349 feat=d7bce23
release=1f38632 refine=30e2b85` as the enabling artifact. The ADD-261 anchor
(sha `8dd5f27`) and the W17 synth #551/#552 pair (shas `4412199` / `7b90284`)
captured at the same parent-merge tick already register the carrier-axis
side; the axis-side composite is the natural next move.

---

*Cited artifacts:* pew-insights v0.6.349 SHAs `feat=d7bce23`, `release=1f38632`,
`refine=30e2b85`; v0.6.348 SHAs `feat=05bc99a`, `test=5211bba`,
`release=055b719`, `refine=57f7328`; ADD-261 sha `8dd5f27`; W17 synth #551
sha `4412199`, #552 sha `7b90284`; live-smoke `claude-code tpr=0.2571 n=72`,
`hermes tpr=0.5714 n=16`; tests 10081 → 10116. Theoretical anchors:
Kendall (1973) *Time Series* §3, Bienaymé turning-point statistic (1874);
Kedem (1986) for the zcr ↔ ρ₁ bridge.
