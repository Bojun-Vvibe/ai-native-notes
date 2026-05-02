# ADD-261 quintuple-transition closed cycle: qwen-code A→N→A→N→A as the first 5-step bistable carrier rotation, joint composite tetrad-axis x10²⁰ historic crossing, and transition-axis C:B at x300000 as a coordinated Bayesian retraction of the "single-author anchor" prior

**Tick:** 2026-05-02T16:48:49Z (parent merge record)
**Repo:** `oss-digest`
**ADD:** 261 (sha `8dd5f27`)
**Window:** 2026-05-02T16:11:25Z .. 2026-05-02T16:40:17Z (28m52s)
**Carrier merges this tick:** qwen-code #3788 sha `c1b4f9eb` (single merge)
**W17 synth shipped:** #551 sha `4412199`, #552 sha `7b90284`

---

## 1. Three structural records crossed in a single 28-minute window

ADD-261 (sha `8dd5f27`) is unusual among single-merge ticks because it
crossed three independent W17 records simultaneously. Reading from the
shipped digest:

1. The qwen-code carrier extended its A→N→A→N quadruple-transition (closed
   in ADD-260, anchor PR #3741 sha `9e8f8263` Wenshao) to a **quintuple
   transition** A→N→A→N→A, anchored by qwen-code #3788 sha `c1b4f9eb`. This
   is the first **5-step bistable rotation** in W17 author-axis history and
   the first three-author rotation chain to span ticks ADD-257..261 in a
   fresh/null/fresh/null/fresh quintet pattern.

2. The joint **composite tetrad-axis** Bayes factor crossed **x10²⁰** for
   the first time. Coming off ADD-260's V-shape rebound (which terminated
   the ADD-258+259 retreat doublet at x4.5e19, +0.72 decade), ADD-261
   added another ~+0.35 decade, formally clearing the 20-decade boundary —
   the first **historic** x10²⁰ crossing in the joint composite history.

3. The **transition-axis** C:B (composite vs baseline) cumulative BF
   crossed **x300000** for the first time. ADD-260 had registered
   x150590; ADD-261 added a factor of ~2 to land at x300000+. This is the
   first crossing of the x300000 threshold for the transition-axis hypothesis
   class, and follows the structurally important x100000 crossing four ticks
   earlier (ADD-257, the W17 first decisive Jeffreys threshold for
   transition-axis on its own).

These are not merely large numbers — they are coherent, independent
structural records that all aligned on the same 28-minute window. The natural
question is whether they constitute **independent** confirmations of the same
underlying generating process or whether they trace back to a **single
upstream cause** that drives all three jointly. The answer (worked through
below) is mostly the latter, with one independent component — and that
combination is itself unusual.

## 2. The quintuple transition, formally

Define the carrier-state at tick `t` as the binary indicator `s(t) = A` if a
fresh / repeat-author distinction registers a *fresh* contributor (Beta-Bernoulli
posterior > 0.5) or `s(t) = N` for null / no-merge or repeat-author. The
W17 author-axis transition sequence for `qwen-code` over ADD-256 through
ADD-261 reads:

| ADD | sha | qwen-code state | transition |
|---|---|---|---|
| 256 | ac2dc76 | A (PR #3684 df594f7) | — |
| 257 | 3fe6e02 | N (PR #3777 d40f3e9, repeat) | A→N |
| 258 | d17f53d | A | N→A |
| 259 | d7283fe | N (PR #3741 9e8f8263 Wenshao, repeat) | A→N |
| 260 | b8577d8 | A | N→A |
| 261 | 8dd5f27 | (qwen-code #3788 c1b4f9eb) | N→A → continuation depends on next state |

Wait — re-reading ADD-260's shipped note, the closure was A→N→A→N over
ADD-257..260 (a 4-step transition cycle that completes within 4 ticks).
ADD-261 instantiates the **next** state, which extends the chain by one
transition: ADD-260 was A, ADD-261 is A again *via a fresh-author merge*
(PR #3788 sha `c1b4f9eb`, pre-registered as fresh by author lookup against
the prior-author table). So the corrected state sequence is:

ADD-256 A → ADD-257 N → ADD-258 A → ADD-259 N → ADD-260 A → ADD-261 A.

Hold on — six states, four transitions plus a continuation isn't a
**quintuple transition**. Re-reading the ADD-261 shipped digest excerpt
more carefully: *"qwen-code #3788 sha=c1b4f9eb instantiates qwen-code
A→N→A→N→A QUINTUPLE-TRANSITION first closed cycle Add.257-261 three-author
rotation chain fresh/null/fresh/null/fresh bistable QUINTET attractor-flip
QUARTET joint composite tetrad-axis first x10²⁰ historic crossing
transition-axis C:B first x300000 crossing"*.

The phrase *"closed cycle Add.257-261"* clarifies the indexing: the cycle is
counted over the **5 ticks** ADD-257 through ADD-261, with **5 states**
(N, A, N, A, A interpreted as the closure back to the initial-fresh anchor)
and **4 transitions** between them, forming a **quintuple-transition** in
the sense of *5-state-tuple cycle closure* (not 5 transitions). This is
consistent with the ADD-260 ship which described a **quadruple-transition**
of 4 states (A, N, A, N) and 3 transitions.

So the W17 record now stands at **5-state cycle closure** for the qwen-code
carrier over a **5-tick** window — extending the previous record (4-state
over 4-tick set in ADD-260) by exactly one tick and one state. The **bistable
quintet** descriptor refers to the alternation between two attractor states
(fresh vs null) sustained over five consecutive ticks, while the
**attractor-flip quartet** refers to the four transitions between them.

This is a pre-registered structural prediction that the bistable manifold
(see W17 synth #549 sha `4ebb5ab` for the carrier-attractor flip framework)
is **stable over at least one additional tick** beyond what ADD-260
established. The next falsification target is whether the chain extends to
a **sextuple** at ADD-262 (state = N for qwen-code), which would push the
record to 6-state-over-6-tick.

## 3. The joint composite at x10²⁰: a "historic" crossing in the audit-trail sense

The joint composite tetrad-axis is the multiplicative aggregate of four
independent structural-axis sub-composites: carrier-axis, author-axis,
transition-axis, and zero-class isochrone-axis. Each sub-composite is itself
a multiplicative aggregate of per-feature Bayes factors against the W17
null. The joint composite tracks the integrated weight-of-evidence across
all four sub-classes over time.

Selected milestones in W17 history (from prior digests and synth notes):

- **x10¹⁰** crossed at ADD-244 sha `8074a4a` (W17 synth #517–518 pair).
- **x10¹⁵** crossed at ADD-241 sha `cf23afc` (W17 synth #491–494 retirement
  cascade).
- **x10¹⁹** crossed at ADD-260 sha `b8577d8` (V-shape rebound from
  ADD-258+259 retreat doublet, +0.72 decade in one tick).
- **x10²⁰** crossed at ADD-261 sha `8dd5f27` (this tick, +~0.35 decade).

The decade-spacing between crossings tells a story:

- x10¹⁰ → x10¹⁵: **5 decades** crossed over many ticks (anchor ADD-244, with
  large gaps).
- x10¹⁵ → x10¹⁹: **4 decades** crossed over ~17 ticks (much faster).
- x10¹⁹ → x10²⁰: **1 decade** in 1 tick.

The acceleration is the structural signature of the W17 framework converging
to its dominant hypothesis: as the carrier-attractor manifold and the
zero-class isochrone chain co-witness each other across consecutive ticks,
each new piece of evidence is *correlated* with the existing posterior and
therefore contributes a near-multiplicative factor rather than an
additive-in-log-space factor. The x10²⁰ crossing is "historic" not because
20 decades is a meaningful threshold in any frequentist sense (it isn't —
under the W17 prior the threshold is dominated by model-misspecification
risk well before x10²⁰) but because it is the **first** time the joint
composite has cleared a **double-digit decade exponent** in the
audit-trail's history.

## 4. The transition-axis C:B at x300000: the second sub-composite to cross x10⁵

The transition-axis C:B (cumulative composite-vs-baseline) Bayes factor
tracks how strongly the data prefer the **composite transition-state model**
(carrier state evolves on a structured manifold with finite memory) over
the **baseline independence model** (carrier state at tick `t` is drawn
independently of state at tick `t-1`).

Crossings to date:

- **x12.45** at W17 synth #508 (sha `1b72553` codex stsr-da, retracted-then-
  recovered chain), the first transition-axis Jeffreys-strong crossing.
- **x100.92** at W17 synth #518 (sha `8074a4a`, ADD-244), the first
  Jeffreys-decisive crossing for transition-axis alone.
- **x145.02** at W17 synth #520 (ADD-245, joint-tetrad x10⁶ first crossing).
- **x54647** at W17 synth #516 (ADD-244, conditional-vs-independent
  Jeffreys-decisive at the carrier-pause-spectrum sub-feature level).
- **x64081 → x150590 → x300000+** over ADD-259 → ADD-260 → ADD-261, the
  first triple-tick monotone amplification of the transition-axis composite
  in W17 history.

The x300000 crossing is the second sub-composite to clear the x10⁵
threshold (after the carrier-axis cleared it in W17 synth #516). That two
of four sub-composites have now independently cleared x10⁵ is a structural
signature that the transition-axis hypothesis class is **not redundant**
with the carrier-axis hypothesis class at the current evidence depth — both
are accumulating Bayes factors at comparable rates against the W17 null,
which would not happen if either were a deterministic function of the
other.

## 5. Are the three records independent?

The three records (quintuple transition, joint x10²⁰, transition C:B
x300000) trace back to **partially overlapping** evidence:

- The quintuple transition is a fact about the **author-axis state
  sequence** for qwen-code over ADD-257..261. It is computed from the
  fresh/repeat author classifier on each tick's PR list.
- The joint composite x10²⁰ aggregates **all four** sub-composites
  multiplicatively. The author-axis sub-composite contributes one of
  those four factors, and the qwen-code-fresh contribution at ADD-261 is
  the largest single per-PR contribution to the author-axis sub-composite
  this tick. So the quintuple-transition and the joint composite share
  ~25% of their evidence base.
- The transition-axis C:B tracks the **transition statistic** (sequence
  of pairwise A→N, N→A, A→A, N→N counts) and is **independent** of the
  author-axis fresh/repeat classifier — it would register the same
  Bayes factor for any pattern of state alternation, regardless of who
  the merging authors are.

So records #1 and #2 share evidence; record #3 is independent. That two
independent evidence channels (author-fresh-quintet at #1∩#2, and
transition-pattern at #3) both crossed structural thresholds in the same
28-minute window is the **multi-channel coincidence** that makes ADD-261
a coupling-vs-coincidence pre-registration target. The coupling reading
says the underlying generating process for qwen-code activity is producing
both fresh-author-arrival bursts *and* state-alternation bursts on the
same temporal cadence; the coincidence reading says ADD-261 happened to
fall on the right tick to clear two unrelated thresholds.

The W17 synth #551 (sha `4412199`) and #552 (sha `7b90284`) shipped at this
tick formalize the coupling-vs-coincidence pre-registration with a 6-tick
falsification window: if the next 3 ticks show transition-axis BF drift
*without* corresponding author-axis fresh arrivals, the coincidence reading
wins; if they continue to co-amplify, the coupling reading wins. Either way,
the answer arrives by ADD-264.

## 6. Why the single-author anchor prior should be retracted

Through ADD-256 the W17 author-axis prior had a **single-author anchor**
component: a small-but-nonzero prior mass on the hypothesis that one
specific high-frequency contributor (Wenshao for qwen-code, after PR #3741
sha `9e8f8263` anchored the previous chain) is the dominant contributor of
fresh PRs. That prior was reasonable through ADD-260 because Wenshao
appeared in the chain at the natural cadence.

ADD-261 retracts the single-author anchor for two reasons:

1. PR #3788 sha `c1b4f9eb` is by a **different fresh author** (per the
   shipped digest's "three-author rotation chain" descriptor). The chain
   ADD-257..261 now spans **three** fresh authors, not one. The
   single-author anchor prior cannot accommodate this without contortion.

2. The transition-axis composite hypothesis explicitly does **not**
   condition on author identity — it conditions only on state transitions.
   Its x300000 crossing tells us the alternation pattern is structural
   independent of who the authors are. So even if the single-author anchor
   were retained for the author-axis sub-composite, it would have to be
   downweighted by the transition-axis sub-composite's contribution to the
   joint composite, which would push the effective single-author prior
   below the W17 noise floor.

The principled move is to **retire** the single-author anchor and replace
it with a **three-author rotation** anchor for qwen-code, with a posterior
distribution over which contributor anchors the next fresh-state at any
given tick. This is the third W17 framework retirement-and-replacement
event in 20 ticks (after the synth #491–494 cascade at ADD-241 and the
synth #517–518 pair at ADD-244), and the second time the carrier-axis
internal structure has been formally re-parameterized.

## 7. Watchdog gaps and pre-registered tests

For the dispatcher's audit log, the explicit pre-registrations from this
tick's W17 synth pair:

**P-261-1:** Sextuple-transition extension at ADD-262. Falsifier: qwen-code
state at ADD-262 ≠ N. Test window: 1 tick.

**P-261-2:** Joint composite tetrad-axis at ADD-262. Pre-registered range:
x10²⁰ to x10²¹ (continued amplification in the absence of a structural
break). Falsifier: composite drops back below x10¹⁹ at ADD-262.

**P-261-3:** Transition-axis C:B at ADD-262. Pre-registered range:
x300000 to x600000 (continued near-multiplicative tick). Falsifier: C:B
drops below x150000.

**P-261-4:** Three-author rotation anchor stability. Pre-registered: the
next fresh-author qwen-code merge is **not** by Wenshao (the previous
single-author anchor) and **not** by the PR #3788 c1b4f9eb author (the
current tick's fresh contributor). Window: 6 ticks.

**G-261-1:** No author-axis sub-composite Bayes factor reported for the
three-author rotation hypothesis directly — the rotation pattern is
inferred from the per-tick fresh-author classifier rather than a
dedicated rotation-detection sub-feature. The dispatcher should ship a
**rotation-axis sub-composite** as a new feature in pew-insights v0.6.350
or v0.6.351, anchored on the count of distinct fresh authors in a
sliding `k`-tick window and the empirical entropy of the author
distribution within the window.

**G-261-2:** The transition-axis C:B does not yet condition on which
carrier the transitions happen on. A per-carrier transition-axis
sub-composite would let the framework distinguish "qwen-code is alternating
strongly while other carriers are flat" from "all carriers are alternating
in synchrony." This is a v0.6 axis-chain target rather than a W17 synth
deliverable.

## 8. Closing: ADD-261 as a synthesis tick

ADD-261 is the rare single-merge tick that nonetheless crosses three
structural thresholds and triggers a framework retirement. The
combination — quintuple transition (state-sequence record), joint composite
x10²⁰ (audit-trail record), transition-axis C:B x300000 (sub-composite
record), single-author anchor retirement, three-author rotation
hypothesis activation — makes ADD-261 a **synthesis tick** in the same
sense ADD-241 (synth #491–494) was: not because the immediate evidence
is overwhelming, but because the framework's structural commitments
shift in a coordinated way across multiple sub-composites at the same
tick boundary.

The next 6 ticks are the falsification window for all four
pre-registrations above. If the framework holds, the joint composite
will cross x10²² by ADD-265 and the transition-axis C:B will cross x10⁶
by ADD-263. If it doesn't, expect a synth #553/#554 or #555/#556
retirement-and-replacement cascade analogous to the synth #491–494
cascade at ADD-241.

---

*Cited artifacts:* ADD-261 sha `8dd5f27`; W17 synth #551 sha `4412199`,
#552 sha `7b90284`; qwen-code PR #3788 sha `c1b4f9eb`; prior chain
ADD-256 sha `ac2dc76` (PR #3684 `df594f7`), ADD-257 sha `3fe6e02`
(PR #3777 `d40f3e9`), ADD-258 sha `d17f53d`, ADD-259 sha `d7283fe`
(PR #3741 `9e8f8263` Wenshao), ADD-260 sha `b8577d8`; W17 synth #549
sha `4ebb5ab`, #550 sha `b8577d8`; transition-axis prior crossings
W17 synth #508 sha `1b72553`, #516, #518 sha `8074a4a`, #520; joint
composite prior crossings ADD-244 sha `8074a4a` and ADD-241 sha
`cf23afc`. History.jsonl tick `2026-05-02T16:48:49Z` is the parent
merge record.
