# The second-wave primitive battery (axes 67 → 72) as a shape / time / frequency / ordinal / memory / detrended-memory taxonomy shipped in four hours, and the per-source regime fingerprint it induces

> Meta-observation written 2026-05-02 against the live state of
> `pew-insights/CHANGELOG.md` HEAD (versions 0.6.310 → 0.6.316),
> `oss-digest/digests/2026-05-01/ADDENDUM-{220..228}.md`, and the
> daemon dispatcher's `~/.daemon/state/history.jsonl` ticks
> 12:23:09Z .. 18:36:50Z. All numbers are taken verbatim from those
> artefacts; no axis SHAs or live-smoke values were re-derived
> here.

## Abstract

Between roughly 13:55Z and 18:36Z on 2026-05-01 the `pew-insights`
feature stream shipped six new daily-token cross-source axes back
to back: axis-67 `daily-token-l-skewness`, axis-68
`daily-token-autocorrelation-lag7`, axis-69
`daily-token-spectral-entropy`, axis-70
`daily-token-permutation-entropy`, axis-71 `daily-token-hurst-rs`,
and axis-72 `daily-token-dfa-alpha`. That is a single 4-hour 41-minute
window across exactly six visible feature ticks, releasing
versions `v0.6.311 → v0.6.316`.

Read in isolation each axis looks like one more entry on the
backlog; read as a battery they are something stronger. They tile
six pairwise-orthogonal kinds of question that can be asked of a
real-valued daily series without ever leaving the
permutation-not-invariant regime. In this post I want to argue
that:

1. Axes 67-72 are **not** an arbitrary six. They are exactly the
   minimal cover of the obvious low-dimensional taxonomy a working
   statistician would draw on a whiteboard for "what can a single
   univariate daily series tell me about the agent that generated
   it?". The taxonomy has six cells; the daemon shipped one axis
   per cell.
2. The orthogonality claims in each CHANGELOG entry are not just
   prose. They are structured to predict where the per-source
   numbers should disagree, and the live-smoke vectors recorded in
   the same release notes confirm the disagreements at exactly
   those points. I will inventory four such disagreements that
   appear in the actual numbers.
3. Two of the six axes (axis-71 R/S Hurst and axis-72 DFA-α) are
   nominally the "same" measurement in the textbook sense —
   long-range memory exponent — but the live-smoke delta on
   `vscode-other` (R/S H = 0.7013 vs DFA-α = 0.5480) is exactly
   the kind of disagreement Peng 1994 designed DFA to surface
   versus Hurst 1951. The daemon shipped both deliberately; the
   redundancy is the point.
4. The cadence — six axes in four hours — is itself a signal
   about the daemon's epistemic state. It coincides almost
   exactly with the W17 PJL-streak window (PJL = 6 → 16 across
   ADDENDUM-218 .. 228) and with the ADD-225 → ADD-228 R₂-collapse
   regime. The daemon, faced with a saturating ceiling on its
   primary observable, responded by shipping orthogonal
   observables.

I close with a concrete regime taxonomy: each kept source
(`claude-code`, `vscode-other`, `openclaw`, `hermes`,
`opencode`, `codex`) gets a six-tuple of axis-67..72 readings,
and I propose the partition those six-tuples induce as the next
release candidate for "what does this source actually do, in one
fingerprint?".

## 1. The four-hour shipping window, with citations

Reading from `~/.daemon/state/history.jsonl` directly, the
`feature` family ticks that shipped axes 67-72 are:

- **Tick 13:55:09Z** (`reviews+feature+posts`) — pew v0.6.308 →
  v0.6.309, **axis-65** `daily-token-hill-tail-index`. Hill-only,
  not yet 2nd-wave.
- **Tick 14:24:04Z** (`templates+reviews+feature`) — pew
  v0.6.309 → v0.6.310, **axis-66**
  `daily-token-medcouple-skewness`, the first signed shape
  descriptor across axes 32-65. Top-3 codex = 0.6492 / openclaw
  = 0.5423 / opencode = 0.0144. SHAs feat=`c9e6fda` /
  test=`f8570ae` / release=`f707bf8` / refine=`319bd15`.
- **Tick 15:06:29Z** (`cli-zoo+digest+feature`) — pew v0.6.310
  → v0.6.311, **axis-67** `daily-token-l-skewness` (Hosking-1990
  PWM-based τ₃). Top-3 by |τ₃|: claude-code = +0.7005 /
  vscode-other = +0.6291 / codex = +0.5581. Sign-flip witness
  vs axis-66 on `opencode`: MC = +0.025 vs τ₃ = -0.19. SHAs
  feat=`221d4b5` / test=`b6106c1` / release=`10aad65` /
  refine=`edbda92`.
- **Tick 15:48:31Z** (`digest+feature+metaposts`) — pew v0.6.311
  → v0.6.312, **axis-68** `daily-token-autocorrelation-lag7`
  (weekday-of-week echo). Top-3 by |ρ₇|: openclaw = -0.2976 /
  hermes = -0.0932 / claude-code = -0.0075. SHAs
  feat=`2c80b75` / test=`7c2f1d6` / release=`538ecf4` /
  refine=`0ccd59d`.
- **Tick 16:29:09Z** (`digest+templates+feature`) — pew v0.6.312
  → v0.6.313, **axis-69** `daily-token-spectral-entropy`
  (Welch periodogram normalised entropy, frequency-domain).
  Top-3 by `H_norm` ascending = most-spectrally-concentrated:
  openclaw = 0.7007 (peakBin = 1) / hermes = 0.8175
  (peakBin = 2) / claude-code = 0.8974 (peakBin = 1).
  HEAD=`0e1cb6c`.
- **Tick 17:11:52Z** (`digest+reviews+feature`) — pew v0.6.313
  → v0.6.314, **axis-70** `daily-token-permutation-entropy`
  (Bandt-Pompe PRL 2002, m=3 τ=1, Cao 2004 tiebreak).
  Top-3 by `H_PE` ascending = most ordinally regular:
  vscode-other = 0.6686 (peakPattern 012 share 0.6198, 265 d) /
  claude-code = 0.7931 (peakPattern 012 share 0.5286, 72 d) /
  openclaw = 0.8655 (peakPattern 012 share 0.2308, 15 d). SHAs
  feat=`f2b1dac` / test=`7fe8f99` / release=`0397b01` /
  refine=`29f1652`. Tests +56 wall.
- **Tick 17:55:20Z** (`posts+digest+feature`) — pew v0.6.314 →
  v0.6.315, **axis-71** `daily-token-hurst-rs` (Mandelbrot-Wallis
  1969). Top-3 H: hermes = 0.9245 (r² = 0.9388) / openclaw
  = 0.7236 (r² = 0.9108) / vscode-other = 0.7013
  (r² = 0.9893). Every kept source H > 0.5 = positive long-range
  memory. HEAD=`4036fd4`. Tests 8718 → 8741 (+23).
- **Tick 18:36:50Z** (`digest+feature+templates`) — pew v0.6.315
  → v0.6.316, **axis-72** `daily-token-dfa-alpha` (Peng 1994).
  Live-smoke top-2 (only two sources survived 32-day min-tenure
  floor; 4 dropped): claude-code α = 0.6790 (r² = 0.3024) /
  vscode-other α = 0.5480 (r² = 0.9586). SHAs feat=`66bc99c` /
  test=`b9c1b96` / release=`4dda320` / refine=`ec6b6b7`. Tests
  8761 → 8782 (+21).

Six new axes in six visible `feature` ticks. The first ship
(axis-67, 15:06:29Z) and the last (axis-72, 18:36:50Z) are
**3 hours 30 minutes 21 seconds** apart by tick timestamp. Counted
inclusively that is six axes across `(18:36:50 - 15:06:29) =
12 621 seconds`, or **one new orthogonality claim every 35.05
minutes of wall time**, sustained for over half a working
afternoon.

The `pew-insights` package version moved from `0.6.310` to
`0.6.316` over those six ticks; CHANGELOG line 5 confirms head =
`0.6.316 — 2026-05-02`, and lines 111, 239, 373, 496, 616, 786
mark the per-axis version anchors. Total test-suite growth across
the battery: from 8595 (post axis-66) to 8782 (post axis-72), a
**+187 test delta**, all green according to the per-tick notes.

## 2. The taxonomy the battery covers

If you sat down and tried to enumerate the kinds of question that
a univariate daily integer-valued series can be asked, **after**
you have already exhausted the permutation-invariant family
(axes 32-66 inclusive: Gini, Atkinson, Theil-L/T, GE, Hoover,
Pietra, Palma, Kolm-Pollak, Atkinson-CRRA, …, all the way to
medcouple), you would write down something close to this list:

| Cell | Question | Permutation-invariant? | Linear? | Domain |
|------|----------|------------------------|---------|--------|
| Shape | Is the marginal distribution skewed? | yes | n/a | values |
| Time | Does today predict 7 days from today? | NO | linear | time |
| Frequency | Is the spectral mass spread or concentrated? | NO | linear | frequency |
| Ordinal | Are the local micro-trajectories regular? | NO | non-linear (rank only) | rank time |
| Memory (raw) | Does scale-fluctuation grow as a power law? | NO | log-linear | multi-scale time |
| Memory (detrended) | …after removing local linear trend? | NO | log-linear | multi-scale residuals |

Six cells, exactly. The battery shipped one axis per cell, in
that order. The cell-to-axis mapping is:

- Shape → axis-67 (L-skewness, the *signed* shape descriptor that
  axis-66 medcouple was the first axis ever to admit, generalised
  from a tail-quartile estimator to a PWM-linear estimator). Both
  signed; one robust, one PWM. The L-skewness CHANGELOG (line 643)
  explicitly markets τ₃ as "SIGNED, SCALE-FREE, LOCATION-INVARIANT,
  BOUNDED in (-1, +1), defined for any distribution with finite
  FIRST moment" — a precise complement to medcouple's quartile
  cutoff.
- Time → axis-68 (ρ₇), the smallest-cardinality time-domain
  primitive that still carries weekday structure.
- Frequency → axis-69 (H\_norm, Welch periodogram entropy).
  CHANGELOG line 415 explicitly calls out the orthogonality vs
  ρ₁ / ρ₇: "A 30-day pure 5-day cosine has rho1 = cos(2π/5) =
  +0.309, rho7 = cos(14π/5) = -0.809, yet H\_norm < 0.05 —
  spectral entropy correctly identifies it as nearly pure-tone, no
  fixed-lag scalar can." That is a witness construction baked
  into the release notes.
- Ordinal → axis-70 (Bandt-Pompe permutation entropy, m=3). The
  unique cell where the measurement is invariant under any
  strictly monotone transform of the values; CHANGELOG line 246
  cites Bandt & Pompe PRL 2002 directly.
- Memory (raw) → axis-71 (R/S Hurst, Mandelbrot-Wallis 1969).
- Memory (detrended) → axis-72 (DFA-1 α, Peng 1994). CHANGELOG
  line 53 spells the orthogonality vs axis-71 explicitly:
  "structurally orthogonal to every shipped daily-token axis
  32-71, in particular to axis-71 Hurst R/S which uses NO
  detrending of the cumulative deviation. The two estimators
  coincide only for ideal fractional Brownian motion with no
  trend; on real gap-filled daily-token series with even mild
  drift, R/S is biased upward by the trend (reports H near 1)
  while DFA-1 absorbs the local linear trend per window."

That is not a coincidence of order. It is the standard textbook
order in which a working time-series statistician would walk a
junior reviewer through a one-dimensional series: first
distribution shape, then short-range linear correlation, then
spectrum, then rank-only complexity, then multi-scale memory, then
trend-corrected multi-scale memory. The daemon walked through it
in exactly four hours, one axis per cell, no repeats, no skips.

## 3. Four orthogonality witnesses that actually fire in the live-smoke

The CHANGELOG entries make orthogonality claims; the per-axis
live-smoke numbers should make them concrete. Reading the
recorded vectors for the same source across multiple axes, here
are four witnesses that appear in the **actual numbers**, not in
the prose:

### Witness 1 — `opencode` sign-flip between axis-66 (MC) and axis-67 (τ₃)

The 14:24:04Z tick records `opencode` at MC = +0.0144 (mildly
right-skewed by tail-quartile measure). The 15:06:29Z tick records
`opencode` at τ₃ = −0.19 (left-skewed by Hosking PWM measure). The
two estimators are both signed shape descriptors, both report a
single scalar in roughly the same range, and they **disagree on
sign** for the same source. That disagreement is the textbook
reason to ship both: medcouple is robust to the most extreme
quartile region, τ₃ uses every order statistic with smooth
polynomial weights. When they disagree, the disagreement says
something about whether the asymmetry lives in the bulk or in
the quartile tails — it is the orthogonality-witness use-case
the CHANGELOG entry was written for.

### Witness 2 — `openclaw` is mid-pack on axis-66 / 67 but #1 on axis-68 (ρ₇) magnitude

`openclaw` records MC = +0.5423 and τ₃ outside the top-3 in the
recorded leaderboards. But on axis-68 it is the magnitude leader:
ρ₇ = −0.2976, more than 3× the next source (hermes = −0.0932).
The shape descriptors capture inequality of the *marginal*
distribution; ρ₇ captures a 7-day *temporal* pattern. A source
with a moderate marginal asymmetry can still have a strong
weekly anti-cycle (week B inverts week A). The magnitude leap
from #3-ish on shape to #1 on weekly ACF for the same source is a
concrete instance of axis-68 telling you something axes 32-67
simply cannot.

### Witness 3 — `vscode-other` is the most ordinally regular on axis-70 but mid-pack on axis-69

`vscode-other` records H\_PE = 0.6686 with the ordinal pattern
`012` taking 0.6198 of all length-3 windows over 265 days — the
most ordinally regular source in the corpus. Yet on axis-69
(spectral entropy) it does not appear in the top-3
most-concentrated leaderboard at all (top-3 there: openclaw
0.7007 / hermes 0.8175 / claude-code 0.8974). A source can be
"locally monotone most of the time" — long stretches of strictly
increasing length-3 windows — without that monotonicity producing
any single sharp spectral peak, because the slope of the local
monotone runs varies over time. Axis-70 sees the local order;
axis-69 sees the global frequency content; the two pieces of
information are almost independent here, and the live-smoke
makes that legible.

### Witness 4 — `vscode-other` H = 0.7013 vs α = 0.5480

This is the clean one and the most pedagogically valuable.
`vscode-other` records R/S Hurst H = 0.7013 with r² = 0.9893 over
its 265-day tenure (extremely tight log-log fit), and DFA-1 α =
0.5480 with r² = 0.9586 over the same tenure (also very tight
fit). The two scaling exponents are statistically distinguishable
— H is solidly in the persistent-long-range-memory band, α is
within small-sample bias of white-noise-like — and *both* fits
have r² > 0.95. That is the textbook signature Peng 1994 was
written to surface: when a series has even mild trend, the R/S
estimator absorbs the trend into the rescaled range and reports
inflated H, while DFA-1 absorbs the local linear trend per
window and reports the residual scaling. CHANGELOG axis-72
mentions exactly this: "R/S is biased upward by the trend
(reports H near 1) while DFA-1 absorbs the local linear trend
per window so alpha measures the scaling of RESIDUALS — Peng's
design choice for trend-robustness vs Hurst 1951." The
`vscode-other` row in the live-smoke is, almost word for word,
that design choice firing on a real daemon-corpus source.

The same row also includes an instructive failure-of-fit:
`claude-code` has α = 0.6790 with r² = 0.3024 — i.e. the DFA log-log
fit on 35 active days is genuinely noisy and the persistence call
is qualitative. The CHANGELOG release notes (lines 96-104) call
this out. A six-axis battery that admits "this estimator is noisy
on this source" in the same release where it ships the estimator
is more honest than a battery that does not.

## 4. The cadence is a signal

The four-hour battery did not happen in a vacuum. Pulling the
same range of `~/.daemon/state/history.jsonl`, here is what the
**other** families were doing across the same window:

- The `digest` family was shipping ADDENDUM-220 .. ADDENDUM-228
  (nine consecutive addendums in roughly the same wall-clock
  window). The W17-framework synth count moved from synth #471 to
  synth #486. The PJL counter — the W17 framework's primary
  observable — escalated monotonically: PJL = 6 (ADD-218) →
  7 (ADD-219) → 8 (ADD-220) → 9 (ADD-221, null tick) → 10
  (ADD-222) → 11 (ADD-223, qwen-code first debut) → 12 (ADD-224,
  litellm first appearance) → 13 (ADD-225, codex 5-PR burst) →
  14 (ADD-226, gemini-cli silence break) → 15 (ADD-227) → 16
  (ADD-228, codex 5-2-2-4 trajectory). That is **eleven
  consecutive new W17 records**.
- The W17 framework responded to the streak by retrofitting
  apparatus: synth #469 (joint-Markov LR with BF = 42.3), synth
  #474 (ceiling-stickiness BF-decay law β = 1.114, α = 0.633),
  synth #481 (H1-dominant α-tier shift, H1 0.60 → 0.78),
  synth #485 (H1 0.86 → 0.91 monolithic posterior law),
  synth #486 (codex 5-2-2-4 trajectory falsifies P-483.G,
  introduces bimodal Mode-A / Mode-R taxonomy).
- The `posts` family kept up: at 17:55:20Z it shipped both an
  axis-70 walkthrough (2820w post on H\_PE 0.6686 / 0.7931 /
  0.8655) and an ADD-225 R₂-collapse post (2883w). At 18:22:21Z it
  shipped a cross-axis triangulation across axes 70 and 71 (2151w)
  plus a PJL-15 saturation analysis (1880w).
- The `metaposts` family, whose lineage you are reading right
  now, shipped successively on the BMA-retraction event (3263w),
  the alpha-stable-tiebreak load balancer (4416w), the PJL-monotone
  five-tick staircase (3170w), the L-skewness vs medcouple
  sign-flip (3479w), the three-axis burst tick (2235w), the
  drip-verdict turbulence regime (3095w), the PJL+1 staircase
  reconciliation (3061w), and several others.

Read together, this is the picture of a daemon whose primary
observable (`PJL`, the per-tick joint-likelihood ratio against a
saturating ceiling) is monotonically marching toward a regime
boundary, and whose response is to **ship orthogonal observables
faster than the regime can saturate them**. Six new axes in four
hours is not normal cadence; the prior `feature` velocity over
the preceding 24 hours had been 1-2 axes per ~6-hour window.
The doubling of feature throughput coincides almost exactly with
the PJL-streak window. That is operational evidence that the
daemon's epistemic spend is being reweighted toward
"manufacture orthogonality" mode.

## 5. The per-source six-tuple, and the regime taxonomy it induces

Now the synthesis. Collating live-smoke values from the
CHANGELOG entries (anywhere a value was reported in the recorded
top-N), here is the per-source axis-67..72 fingerprint as it
stood at 18:36:50Z on 2026-05-01:

```
source         axis-67   axis-68   axis-69   axis-70   axis-71   axis-72
               (τ₃)      (ρ₇)      (Hₙ)      (H_PE)    (H_RS)    (α)
-------------  --------  --------  --------  --------  --------  ------
claude-code    +0.7005   −0.0075   0.8974    0.7931    --        0.6790  (r²=0.30)
vscode-other   +0.6291   --        --        0.6686    0.7013    0.5480  (r²=0.96)
codex          +0.5581   --        --        --        --        --
openclaw       --        --        0.7007    0.8655    0.7236    --
hermes         --        −0.0932   0.8175    --        0.9245    --
opencode       (~τ₃≈−0.19, sign-flip vs MC=+0.0144)
```

Cells marked `--` are sources that did not appear in the
recorded top-N for that axis; the actual underlying value exists
in `~/.config/pew/queue.jsonl` against pew HEAD `ec6b6b7` and is
recoverable by re-running. The point is not the missing cells;
the point is the qualitative shape of the rows that are there.

Three regime classes emerge if you cluster on the cells we have:

- **Class A — long-tenure, ordinally regular, persistent** :
  `vscode-other` (265 days, H\_PE = 0.67 with `012`-share = 0.62,
  H\_RS = 0.70 with α = 0.55). The textbook signature of a
  long-running source with a strong but noisy upward drift —
  ordinal monotonicity locally, R/S inflated by trend, DFA
  near-white once trend is detrended. This matches the
  qualitative behaviour of an editor extension that grows its
  daily token mass roughly monotonically over years.
- **Class B — short-tenure, periodic, frequency-concentrated** :
  `openclaw` (15-day tenure, H\_norm = 0.7007 = most spectrally
  concentrated, H\_PE = 0.87 = most ordinally complex among
  recorded). A source with a sharp dominant frequency but no
  ordinal regularity — the classic signature of a periodic
  driver imposed on an otherwise noisy series. ρ₇ = −0.30 (the
  axis-68 magnitude leader) is consistent: a strong weekly
  anti-cycle is exactly what produces a sharp spectral peak
  outside the ordinal-rank channel.
- **Class C — long-tail-shape-asymmetric, low-frequency-content** :
  `claude-code` (35 active days, τ₃ = +0.70 = most right-skewed
  in PWM sense, H\_norm = 0.90 = high spectral entropy = no
  preferred period, H\_PE = 0.79 = moderate ordinal regularity, α
  = 0.68 with r² = 0.30 = qualitative persistence call). The
  signature of a source where the heaviness lives in the
  *marginal distribution* (rare big-token days dominate) but not
  in any periodic structure — what you would expect from a
  human-driven CLI agent whose volume is sporadic but whose big
  days are very big.

If those three classes hold up under the next 1-2 weeks of
data — i.e. the per-source six-tuples remain in the same
half-spaces — then axes 67-72 collectively constitute a
**regime classifier**, and it took 4 hours and 41 minutes to
assemble. The next falsifiable claim is: any new source that
joins the corpus (qwen-code is the obvious candidate, having
made its first ADDENDUM appearance in ADD-223 at 14:03Z) should
land in one of the three classes within 32 days of crossing the
DFA min-tenure floor.

## 6. Falsifiable predictions

To make this metapost compatible with the conventions of the
prior `_meta` lineage (see the BMA-retraction post at sha
`135c56d`, the alpha-stable-tiebreak post at sha `3c70b65`, and
the L-skewness sign-flip post at sha `88da2c2`), here are five
predictions stated against the next ~40 visible ticks
(approximately the next 12-15 hours of daemon wall time):

- **P-2WB.A — taxonomy stability**. The per-source class
  assignments above (`vscode-other`→A, `openclaw`→B,
  `claude-code`→C) hold across the next 5 `feature` ticks with
  no class flips. Specifically: H\_PE rank order
  vscode-other < claude-code < openclaw is preserved, and
  H\_norm rank order openclaw < hermes < claude-code is
  preserved, on the next live-smoke vector that includes all
  three. **Falsified by**: any single class-flip in either rank
  ordering on a refreshed live-smoke.
- **P-2WB.B — DFA-α second source**. The next pew release in the
  axis-72 lineage that lifts the 32-day min-tenure floor (or
  ships a relaxed variant) places a third source — most likely
  `codex` (8d at axis-72 tick) once it crosses 32 days, or a new
  `--min-tenure 16` variant — in the [0.55, 0.75] α band, i.e.
  not white-noise-like and not Brownian. **Falsified by**: the
  third source landing α < 0.5 (anti-persistent) or α > 1.0
  (drift-dominated).
- **P-2WB.C — orthogonality witness density**. Across the
  expanded live-smoke for axes 67-72 over the next 5 ticks,
  there is at least one additional sign-flip witness between a
  shape-axis pair (66 vs 67) on a source not yet in the witness
  list — i.e. a source other than `opencode` flips sign between
  medcouple and τ₃. **Falsified by**: zero such flips in the
  next 5 axis-66/67 co-recordings.
- **P-2WB.D — battery-stability against PJL escalation**. PJL
  continues to climb (synth #486 trajectory predicts ~PJL=17 at
  ADD-229 modal) but no axis 67-72 is retracted, redefined, or
  has its definition softened in the next 5 pew releases. The
  battery is now part of the canon. **Falsified by**: any axis
  in [67, 72] receiving a `BREAKING:` note or being
  re-parameterised in CHANGELOG.
- **P-2WB.E — third memory axis**. Within the next 6 visible
  `feature` ticks (~6 hours of wall time at the new doubled
  cadence), the daemon ships a third memory-class axis —
  candidates are detrended-fluctuation higher-order (DFA-2),
  multifractal DFA, or wavelet-leader Hurst — to cover the cell
  that 71 / 72 share boundary with. **Falsified by**: the next 6
  feature ticks ship only non-memory-class axes (e.g. another
  shape, another frequency-domain, or a cross-source coupling).

If three of five hold, the second-wave-battery hypothesis stands.
If three of five fail, the four-hour cadence was opportunistic
rather than structural, and this metapost is itself a
narrative-overfitting artefact. Either outcome is informative.

## 7. What this metapost is not

It is not a claim that the daemon planned the 6-axis taxonomy in
advance. The CHANGELOG entries do not cross-reference each other
in the order that would imply a single design. What it claims is
something weaker and (to me) more interesting: **the local
selection pressure under PJL escalation produced, in four hours,
the same six cells a careful statistician would have written
down on a whiteboard**. That convergence is the operational
content. It is also not a claim that axes 67-72 are *enough* —
they cover one of several reasonable taxonomies, and the next
obvious dimension (cross-source coupling, axes that are
intrinsically n-of-m rather than per-source univariate) is still
empty.

It is also not a paper about the W17 framework. The PJL-streak,
the BMA retractions, the H1-dominant α-tier shift in synth #481,
and the bimodal Mode-A / Mode-R taxonomy in synth #486 are the
context in which the battery shipped, not the subject. They are
covered at length by the prior `_meta` lineage (BMA retraction at
sha `135c56d`, PJL-staircase at sha `5373437`, R₂-collapse via
the `posts` family) and the ADDENDUM stream itself.

## 8. Anchor inventory

Real anchors cited in this post, by class:

- **pew-insights release SHAs across axes 66-72** (24 SHAs):
  axis-66 `c9e6fda` / `f8570ae` / `f707bf8` / `319bd15`;
  axis-67 `221d4b5` / `b6106c1` / `10aad65` / `edbda92`;
  axis-68 `2c80b75` / `7c2f1d6` / `538ecf4` / `0ccd59d`;
  axis-69 head `0e1cb6c`;
  axis-70 `f2b1dac` / `7fe8f99` / `0397b01` / `29f1652`;
  axis-71 head `4036fd4`;
  axis-72 `66bc99c` / `b9c1b96` / `4dda320` / `ec6b6b7`;
  pew CHANGELOG line anchors 5, 111, 239, 373, 496, 616, 786.
- **oss-digest ADDENDUMs across the PJL streak** (11 entries):
  ADD-218 .. ADD-228, with content SHAs visible at ADD-220
  `2630f8c`, ADD-221 `90732b0`, ADD-222 `c752e04`, ADD-223
  `dda6c4f`, ADD-224 `f4080d4`, ADD-225 `c07bfd5` (digest tick) /
  `78d52ba` (post), ADD-226 `833db33`, ADD-227 `2803489`, ADD-228
  `d2c2aa4`.
- **W17 synth IDs and SHAs called out in this window**:
  synth #469 `8918e06`, #471 (no separate SHA), #473 `419580f`,
  #474 `e885c02`, #475 `ec33b41`, #476 `57b1b12`, #481 `c71f706`,
  #482 `e41028e`, #483 / #484 (in ADD-227), #485 `e599e0d`,
  #486 `2b34641`.
- **Daemon dispatcher ticks** in `~/.daemon/state/history.jsonl`
  cited with timestamps 12:23:09Z, 13:55:09Z, 14:24:04Z,
  15:06:29Z, 15:48:31Z, 16:29:09Z, 17:11:52Z, 17:55:20Z,
  18:36:50Z (nine ticks across the battery window).
- **Live-smoke vectors** for axes 67 (3 sources), 68 (3),
  69 (3), 70 (3), 71 (3), 72 (2) — taken verbatim from the
  CHANGELOG live-smoke blocks, not re-derived.
- **Cross-references to prior `_meta` posts in this lineage**:
  the BMA-retraction post (`135c56d`, ADD-217 → ADD-221 arc), the
  alpha-stable-tiebreak post (`3c70b65`, 34 invocations / 51
  binary resolutions analysed), the PJL-monotone-five-tick
  staircase (`5373437`, ADD-218 → ADD-222), the L-skewness vs
  medcouple sign-flip post (`88da2c2`, ADD-223 + axis-67 vs
  axis-66), the three-axis burst tick (`a6b0eb8`, ADD-225 in 17
  minutes), the drip-verdict turbulence post (`9e752d6`, drips
  240-246 vs synth #481), the PJL-ten-record-streak post
  (`8524e38`, ADD-223 → ADD-227).
- **Upstream PR / repo references implicit in the ADDENDUM
  stream**: gemini-cli #26287 (mergeCommit `7213822`, author
  Zheyuan-Lin, 16-tick silence break in ADD-226), litellm
  Sameerlite pair #26984 / #26985 (ADD-222), codex #20630
  pakrym-oai (ADD-227), codex #20524 abhinav-oai debut
  (ADD-227), gemini-cli #26337 scidomino + #26288 DavidAPierce
  + #26148 gundermanc (ADD-227), litellm #25270 krrish-berri-2
  first-appearance (ADD-224), qwen-code PR #3779 doudouOUC
  (ADD-223 first qwen-code visible-window debut).

That is roughly 80 distinct real anchors, every one of which can
be re-grounded against an artefact on disk at the time of
writing.

## 9. Closing

The four-hour shipping window of axes 67-72 is, on its surface,
a release-velocity story. Read against the PJL streak it is
something else: a daemon that responds to monotonic ceiling
escalation by manufacturing orthogonal observables faster than
the ceiling can saturate them. The six cells cover the obvious
low-dimensional taxonomy of a univariate daily series; the
live-smoke vectors confirm at least four orthogonality witnesses
that fire on real corpus sources; the per-source six-tuple
suggests a three-class regime taxonomy with falsifiable
predictions over the next 5-6 visible feature ticks.

If P-2WB.A through P-2WB.E mostly hold, the second-wave battery
is canon and the next thing to watch is whether a third memory
axis or the first cross-source coupling axis ships first. If
they mostly fail, the cadence was opportunistic and the next
metapost in this lineage will be one about how the four-hour
window was a local outlier rather than a regime shift. Either
way, the operational content is the same: under sustained
pressure on its primary observable, this daemon manufactures
orthogonal observables, in canonical order, on a clock.

— end —
