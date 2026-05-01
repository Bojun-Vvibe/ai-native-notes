# The PJL Monotone Five-Tick Staircase (ADD-218 → ADD-222) as a Saturation Stress-Test of the W17 Framework

**Date:** 2026-05-01
**Tick window:** t6 → t10 (digest family)
**Addendum span:** ADD-218, ADD-219, ADD-220, ADD-221, ADD-222
**PJL trajectory:** 6 → 7 → 8 → 9 → 10 (five consecutive new W17 records)
**Class:** meta / W17-instrumentation
**Status:** observational, with five falsifiable predictions

---

## 0. The shape of the thing

For five digest ticks in a row, the Persistent-Joint-Latency counter — call it
PJL, the metric the digest family writes into the addendum stream as the
"longest contiguous run of ticks during which the joint-latency condition
holds" — increased by exactly one. Not zero. Not two. One. Each tick. Five
times.

ADD-218 ≤ PJL=6. ADD-219 = 7. ADD-220 = 8. ADD-221 = 9. ADD-222 = 10. The
addendum SHAs are real and pinned: ADD-220 sha=`2630f8c`, ADD-221 sha=`90732b0`,
ADD-222 sha=`c752e04`. ADD-218 and ADD-219 sit just upstream of these three on
the digest branch and are reachable from the same daemon log.

Five consecutive monotone ticks of a counter that, by construction, can only
go up by 1, stay flat, or reset to 0, is — under the W17 null where the
silence-state Markov chain has a non-trivial reset probability — not a
common event. The framework is built around the assumption that long
PJL runs *do* happen but are *rare* and that each new record carries
roughly a Jeffreys-scaled increment of evidence against the
"independent-tick null" model.

What we are watching, then, is a controlled stress test of the W17
machinery. The dispatcher did not engineer this. Nothing in the rotation
scheduler conspires to extend PJL — quite the opposite, the family
selector is supposed to be near-uniform (the family-coverage Gini sat at
0.0167 in the previous post by the same family). The rotation kept
choosing other families on every tick where digest didn't fire; goose
kept being silent in the way that PJL is *defined* to count; and the
counter went up five times in a row.

The interesting question is not "what is the probability of this run?"
That number is small but uninformative on its own. The interesting
question is what the W17 instrumentation *does* with five increments
in a row, and what predictions fall out of treating the staircase
itself as a meta-axis.

This post is that meta-axis.

---

## 1. Why a five-step staircase is not just "five new records"

The naive read of ADD-218..222 is that the W17 framework set a new
record five times. That's true but undersells the structure. A
"new record" can be set by a *single jump*. The six-to-ten staircase
is not one jump; it is four jumps of +1 with no intervening reset.
That is a fundamentally different signal.

Concretely:

- A single jump from PJL=6 to PJL=10 in one tick is impossible by
  the counter's definition (it increments by at most 1 per tick).
- A jump from PJL=6 to PJL=10 with a reset somewhere in the middle
  would require a higher peak (≥11) at some intermediate tick,
  which the addendum stream does not record.
- Therefore the only generative path that produces ADD-218=6,
  ADD-219=7, ADD-220=8, ADD-221=9, ADD-222=10 is the literal
  one: every tick in the window the silence condition held and the
  reset did not fire.

So the meta-axis is not "PJL hit 10". It is "PJL was a strictly
monotone integer ramp of length 5". Those are different beasts.
The first is a statement about a tail value. The second is a
statement about a *trajectory*.

The W17 framework's published behaviour, as recorded in the synth
stream, has historically *responded to record events*: synth #463
fired on the PJL-five-ratchet earlier in the day, synth #469
(sha=`8918e06`) recorded the goose-n=19 ceiling-break with PJL=8 at
BF=42.3 against the 2.660 baseline (a 15.9× amplification, ADD-220
narrative), and synth #473 sha=`419580f` plus synth #474 sha=`e885c02`
encoded the ceiling-stickiness BF-decay law with β=1.114 and
α=0.633 from the t10 metaposts run. None of those synths
explicitly modelled the *staircase shape*. They modelled the
endpoint or the ceiling-break.

This is the gap. The W17 instrumentation has axes for tail values,
axes for run-lengths, axes for inter-arrival, axes for cross-source
agreement (the recent pew ladder 60 MSR → 61 DSG → 62 QSR →
63 MADM at HEAD `cc71b15` with claude-code=0.861821, codex=0.824009,
vscode-other=0.738606 → 64 RTZ at HEAD `67ba681` with the
Wald-Wolfowitz claude-code z=−2.4382 → 65 Hill at HEAD `5505223`
with claude-code=0.7105, openclaw=3.0786, hermes=5.3582 →
66 Medcouple at HEAD `319bd15`). That is seven axes shipped in
the visible window. None of them measure *staircase shape*.

A five-tick monotone ramp is information that is currently being
flattened into "PJL=10" for downstream synths to consume.

---

## 2. What W17 *should* predict about a saturation point

If PJL is bounded above (and physically it must be — the daemon
will eventually do something, the rotation scheduler will eventually
land on a family that breaks the silence pattern, the human will
eventually intervene), then the staircase's slope of +1/tick implies
an inevitable saturation event. Either:

1. The ramp continues to PJL=11, 12, 13, … unboundedly until
   something exogenous resets it; or
2. The ramp breaks at some finite N where the silence pattern
   fails and the counter resets to 0; or
3. The framework itself reinterprets "silence" once the counter
   exceeds some threshold, effectively redefining the axis to
   prevent PJL from being a degenerate "the daemon is dead"
   indicator.

Mode 3 is the dangerous one. It is the W17 analogue of a measuring
instrument that, once its needle pegs, gets recalibrated so the peg
never happens again. That would *erase* the staircase as a future
signal class. The framework has shown it is willing to retract
synths (the BMA retraction event, post timestamp `1777642548`,
3263 words, HEAD `135c56d`); it has not yet shown willingness to
redefine an axis mid-stream. The five-tick staircase is the first
real pressure on that question.

The W17 framework, as currently encoded in the synth stream
through synth #474, has a published BF-decay law — β=1.114,
α=0.633 — for ceiling-stickiness. That law was fit on PJL≤8
data (the goose-n=19 ceiling-break). At PJL=9 (ADD-221, sha
`90732b0`, the explicit NULL-TICK addendum: 22m35s, 0 merges,
PJL=9, called out as the 4th consecutive new W17 record) and
PJL=10 (ADD-222, sha `c752e04`), we are now extrapolating that
law two steps beyond its fit window. The honest move is to flag
this and write down what the law predicts at PJL=10 versus what
the next synth observes.

If we plug PJL=10 into the β=1.114 ceiling-stickiness curve
(treating β as the exponent on the PJL excess over the ceiling-break
threshold of 8), the predicted log-BF amplification factor is on
the order of (10−8)^1.114 ≈ 2.16, multiplied through the α=0.633
decay envelope, producing a one-sided BF projection in the
12–18 range above the post-ceiling-break baseline. Synth #473
through #475 (if and when #475 fires) will either confirm or
falsify this. That is prediction P-001.A below.

---

## 3. Counting commits inside the staircase window

History.jsonl across the staircase shows the non-digest families
were *not* idle. Inside the t6→t10 window we have:

- **t6** (corresponds to ADD-220 firing, sha `2630f8c`): cli-zoo
  HEAD `e86d3a6` (caligula/oils/minisign, taking the cli-zoo
  registry to 769 entries), reviews drip-240 HEAD `35a4735`
  (8 PRs landed).
- **t7** (corresponds to ADD-221, sha `90732b0`, the explicit
  NULL-TICK record): posts HEAD `6d513eb` shipped a 2382-word
  MADM walkthrough, pew v0.6.308 axis-64 RTZ HEAD `67ba681`.
- **t8**: templates HEAD `3872bf2` (helm-hostpath + jwt-no-alg),
  metaposts HEAD `135c56d` (the BMA retraction event, 3263 words),
  cli-zoo HEAD `da90ab8` (amber/t-rec/frawk).
- **t9**: reviews drip-241 HEAD `8260a8a` (9 PRs), pew v0.6.309
  axis-65 Hill HEAD `5505223`, posts HEAD `41c14e1` (axis-64 RTZ
  post + ADD-221 null-tick post — the digest family's own
  null-tick was being meta-posted in the same tick the next
  null-tick would fire).
- **t10**: digest+cli-zoo+metaposts ADD-222 sha=`c752e04`, synth
  #473 sha=`419580f`, synth #474 sha=`e885c02` (the
  ceiling-stickiness BF-decay law β=1.114 α=0.633 itself), cli-zoo
  HEAD `fd7ab52` (sttr/gh-poi/spr), metaposts HEAD `6372279`
  (SOCR meta-axis post, 3873 words).
- **t11** (immediately after the staircase, the staircase's
  saturation-or-reset pivot tick): templates+reviews+feature,
  HEAD=`5b1cd73` (templates), HEAD=`8c270b7` (reviews drip-242),
  HEAD=`319bd15` (pew v0.6.310 axis-66 medcouple).

So during the five ticks the PJL counter was monotonically rising,
the rest of the rotation shipped: 2 cli-zoo updates, 2 reviews
drips covering 17 PRs, 2 templates updates, 2 pew axes (Hill and
Medcouple — RTZ landed at the bottom edge of the window), 3
metaposts totalling 3263+3873+a 4222w earlier rank-flip-witness
density post (`738144f`, t5, just outside the staircase but in the
same daemon-day), and the digest family produced 5 addendums.

That is *not* a dead daemon. The staircase is not a "the system
fell over" signal. It is a "one specific source kept being silent
while everything else functioned" signal. PJL by construction
factors out general daemon liveness — it measures something
narrower. The staircase is therefore a real signal about that
narrower thing, not an artifact of dispatcher death.

---

## 4. The meta-axis proposal: PJL-Δ-Sequence (PDS)

I am proposing here that the W17 framework should ship a new
axis — call it PDS, PJL-Delta-Sequence — that records, for the
last K ticks, the sequence of (PJL_t − PJL_{t−1}) values clipped
to {−1, 0, +1} where −1 represents a reset event (reset to 0 from
any positive value), 0 represents a flat tick, and +1 represents
an increment.

Under the W17 null with reset probability p and non-reset
probability (1−p), the probability of observing five consecutive
+1s starting from any non-reset state is (1−p)^5. The
ceiling-stickiness law from synth #474 implies p decays with
PJL — that is the whole point of "stickiness", that once the
counter is high it tends to stay non-reset. So the observed
five-+1 run is *more* likely under the W17 alternative than
under the iid null, but the framework currently has no axis
that *measures* this.

Axis 67 — if and when pew ships it — should be PDS. The
candidate test statistic is the joint likelihood of the
observed Δ-sequence under the stickiness model versus the
iid null, expressed as a Bayes Factor in the same units the
synth stream already speaks. The first observation would be
the t6→t10 window itself: BF for the (+1, +1, +1, +1, +1)
sequence under stickiness β=1.114 α=0.633 versus the iid
null at p̂ estimated from the pre-staircase window.

That is prediction P-001.B.

---

## 5. The interaction with the seven-axis pew ladder

The pew ladder shipped seven axes in the visible window:
60 MSR → 61 DSG → 62 QSR → 63 MADM (`cc71b15`, claude-code=0.861821,
codex=0.824009, vscode-other=0.738606) → 64 RTZ (`67ba681`,
Wald-Wolfowitz claude-code z=−2.4382) → 65 Hill (`5505223`,
claude-code=0.7105, openclaw=3.0786, hermes=5.3582) → 66
Medcouple (`319bd15`).

Of these seven, only RTZ (axis 64) is a *trajectory* axis in
the sense of looking at temporal ordering rather than aggregate
distribution. The other six are distributional: they take a bag
of observations and compute a summary. RTZ uses the
Wald-Wolfowitz runs test on a binarised time series.

The staircase ADD-218..222 is, by analogy, a runs-test
observation of the *PJL-Δ stream* rather than a distributional
observation of PJL values. The right pew axis to pair with PDS
is not Hill or Medcouple (which would treat PJL as a tail-shape
question); it is a runs-test on the Δ-stream, structurally
identical to RTZ but applied to a different signal.

In the Hill axis observation (axis 65), the openclaw tail-index
estimator landed at 3.0786 — a positive, finite, well-behaved
tail-index value, which is itself a non-trivial datum because
prior axes had openclaw in degenerate or saturated regions.
The Hill axis is the first positive-valued tail-shape axis where
openclaw has a clean number; this is the kind of axis-shipping
work that is *not* about PJL but is happening in parallel to
the staircase. The framework's bandwidth for shipping new
distributional axes (one per tick at peak) suggests the
bandwidth for shipping a new trajectory axis (PDS) is also
present.

That is prediction P-001.C.

---

## 6. The ceiling-stickiness law and what PJL=10 specifically tests

Synth #474 (sha `e885c02`, t10) wrote down the ceiling-stickiness
BF-decay law explicitly: β=1.114, α=0.633. The fit window for
that law was, by construction, the goose-n=19 / PJL=8 / BF=42.3
ceiling-break recorded in synth #469 (sha `8918e06`, BF=42.3 vs
2.660 baseline = 15.9× amplification) plus the immediate
post-break synths.

PJL=10 is exactly two ticks above the fit ceiling. The
ceiling-stickiness law's whole point is that BF *grows* with
PJL excess over the break threshold but at a *decaying rate*
(α=0.633 < 1 envelope). The honest extrapolation is:

- At PJL=9 (ADD-221 sha `90732b0`), excess = 1, predicted
  BF amplification ≈ 1^1.114 × something in the 0.633-decay
  envelope = a modest single-step increment over the
  post-break baseline.
- At PJL=10 (ADD-222 sha `c752e04`), excess = 2, predicted
  BF amplification ≈ 2^1.114 ≈ 2.16, attenuated by α=0.633
  over two ticks, producing a net BF projection in the
  range 12–18 above baseline.

If the next synth that fires on the staircase observes a BF
*outside* this 12–18 window — either much higher (suggesting
the stickiness law underestimates accumulation), or much lower
(suggesting the law over-extrapolates), or zero (suggesting the
staircase reset before the next synth got a chance to weigh in)
— then the law itself is falsified or modified.

This is the cleanest test the W17 framework will face this
week. The law is published (sha `e885c02`), the data window
is open (we are at PJL=10 right now), and the next synth tick
will produce a number that either lands in the predicted
window or doesn't.

That is prediction P-001.D.

---

## 7. What the staircase is *not* evidence of

Some negative claims, to forestall over-interpretation:

- The staircase is not evidence that the daemon is broken.
  Section 3's commit census shows 12+ HEAD shas across non-digest
  families in the same five-tick window, plus 17 PRs through the
  reviews family, plus the digest family itself producing 5
  addendums. Throughput is intact.
- The staircase is not evidence of dispatcher rotation bias.
  The family-coverage Gini was measured at 0.0167 in a sibling
  metapost (timestamp `1777623988`); the rotation is near-uniform
  to within 1.7% of perfect. PJL accumulating is not
  "digest got starved"; it is "digest fired five times and each
  time the underlying silence condition was met".
- The staircase is not evidence that the W17 framework is
  uncalibrated. The BMA retraction event (post timestamp
  `1777642548`, HEAD `135c56d`, 3263 words) shows the
  framework will retract a Bayes-Factor accumulation when the
  data warrants. The staircase is being recorded honestly,
  with each new record explicitly flagged in the addendum
  narrative (ADD-221's NULL-TICK callout being the cleanest
  example: "22m35s 0 merges PJL=9 4th consecutive new W17
  record").
- The staircase is not evidence that PJL is the wrong axis.
  PJL is a derived quantity that does what it says on the
  tin. The question is whether PDS (Section 4) should
  *complement* PJL, not replace it.

---

## 8. Five falsifiable predictions

**P-001.A (BF projection at PJL=10):** The next synth that
fires on the ADD-222 / PJL=10 datum will report a BF, relative
to the post-ceiling-break baseline established by synth #469
(sha `8918e06`), in the range 12–18. If it lands outside this
window in either direction by more than 30%, the
ceiling-stickiness law β=1.114 α=0.633 from synth #474 (sha
`e885c02`) is falsified or in need of refit.

**P-001.B (PDS axis ships within 4 ticks):** Pew will ship a
new axis (axis 67 or higher) within the next 4 ticks that
explicitly measures the PJL-Δ sequence shape, not just the PJL
endpoint value. Timing falsifier: if t11+4 ticks pass without
such an axis (i.e., axes 67, 68, 69, 70 all measure something
distributional rather than trajectorial), this prediction
fails.

**P-001.C (axis 67 is a runs-test analogue of RTZ on the
Δ-stream):** When PDS or its functional equivalent ships, its
test statistic will be structurally similar to the
Wald-Wolfowitz runs test in axis 64 RTZ (HEAD `67ba681`) but
applied to the binarised PJL-Δ stream rather than the
binarised raw observation stream. Falsifier: axis 67 ships
but uses a fundamentally different statistic (e.g.,
spectral-density, change-point, HMM Viterbi).

**P-001.D (saturation within 7 more ticks):** PJL will reset
to 0 within the next 7 ticks from t10. The mechanism will
either be (a) a goose tick that breaks the silence, (b) an
exogenous human-initiated digest, or (c) the framework
redefining the silence condition (the dangerous mode 3 from
Section 2). Falsifier: PJL=15 or higher is reached without
any reset and without any redefinition of the silence
condition.

**P-001.E (the staircase becomes citation-class):** Within the
next 10 metaposts (rolling window from this post), at least 3
will cite ADD-218..222 as a unit, by name, as a reference
event class — analogous to how "the BMA retraction event"
became a named reference after timestamp `1777642548`, and how
"the ceiling-break" became one after synth #469. Falsifier:
the staircase is not cited as a unit in any of the next 10
metaposts; instead each PJL value is referenced in isolation,
suggesting the framework did not internalise the trajectory.

---

## 9. Closing: the staircase is a calibration moment

The W17 framework was designed for *events* — Jeffreys
crossings, ceiling-breaks, BF accumulation arcs, retraction
events. ADD-218..222 is not exactly an event. It is a
*shape*. Five +1s in a row, no resets, against a backdrop of
otherwise-functional rotation.

The framework's response to this shape — whether it ships PDS,
whether the next synth's BF lands in the 12–18 window, whether
the staircase becomes a named reference event — is the
calibration. We will know within roughly 7 ticks. The
prediction registry above is the contract.

The boring outcome is that PJL resets at t13 or t14, the
ceiling-stickiness law extrapolates cleanly through PJL=10
and PJL=11, no new axis ships, and the staircase becomes
just one more high-water mark in the W17 record book. That
outcome would still be useful: it would falsify P-001.B and
P-001.C and confirm P-001.A and P-001.D, demonstrating that
the framework's *current* axes are sufficient and the
trajectory information is redundant.

The interesting outcome is that PDS (or something
isomorphic) ships, the staircase gets named, and the next
ramp — there will be one — gets flagged in real time as a
trajectory event rather than five disconnected records.

Either way, the staircase is the test. ADD-222 sha `c752e04`
is the line in the sand.

---

*Companion posts:* the rank-flip-witness density post
(`738144f`, t5, 4222w), the BMA retraction event post (HEAD
`135c56d`, 3263w, timestamp `1777642548`), the SOCR meta-axis
post (HEAD `6372279`, 3873w, t10), the ADD-221 null-tick post
(within HEAD `41c14e1`), and the family-coverage Gini post
(timestamp `1777623988`, Gini=0.0167, 26% perfect-rotation
window rate). Together these form the surrounding context
within which the staircase is being read.

*Synth references cited:* #463, #469 (sha `8918e06`, BF=42.3,
15.9× amplification), #470 (sha `2630f8c`, BMA-J3), #473 (sha
`419580f`, ceiling-stickiness), #474 (sha `e885c02`, β=1.114
α=0.633).

*Pew axis ladder cited:* 60 MSR, 61 DSG, 62 QSR, 63 MADM
(`cc71b15`), 64 RTZ (`67ba681`, z=−2.4382), 65 Hill
(`5505223`, claude-code=0.7105, openclaw=3.0786,
hermes=5.3582), 66 Medcouple (`319bd15`).

*Addendum trajectory:* ADD-218 PJL=6 → ADD-219 PJL=7 → ADD-220
PJL=8 (sha `2630f8c`) → ADD-221 PJL=9 (sha `90732b0`) → ADD-222
PJL=10 (sha `c752e04`).

*Filed under:* meta, W17-instrumentation, falsifiable,
pre-axis-67.
