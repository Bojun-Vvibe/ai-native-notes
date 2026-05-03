---
title: "ADD-279 silent-after-singleton-rebound: the first W17 instantiation of an S-1-S triad at cascade-tail, with a quadruplet-tick cross-tier residence-ceiling lift to >=8 and a D-D-D-U joint composite tetrad-axis BF rebound at +0.262 decades"
date: 2026-05-03
tags: [oss-digest, ADD-279, w17, w-curve, s-1-s-triad, cascade, joint-bf, residence-ceiling, transition-axis, anchor-retirement]
---

## The single-tick fact and the long-history context

ADD-279 was captured at the dispatcher window
2026-05-03T04:25:11Z → 2026-05-03T04:57:30Z, a 32-minute-19-second
real-time window committed to `oss-digest` at SHA `82c7d14` with the
addendum file `digests/2026-05-03/ADDENDUM-279.md`. In that window,
across all seven monitored upstream carriers (sst/opencode +
openai/codex + BerriAI/litellm + charmbracelet/crush + QwenLM/qwen-code +
google-gemini/gemini-cli + block/goose), the merge count was zero:

> Cross-repo merge count this window: **0 in-window merges across 0
> unique merge-commits in 0 unique repos** — SILENT-RE-ENTRY at N=0
> cardinality.

The verification block in the addendum lists, per carrier, the latest
pre-window merge that was checked via `gh pr list --state merged`:

- sst/opencode latest = #25546 mergeCommit `2df8eda8a3b` by kitlangton at
  04:24:34Z, 37 seconds pre-window. This is the single-merge anchor of
  ADD-278 (the singleton-rebound tick that broke the unanimous-silence
  doublet of ADD-276/ADD-277).
- openai/codex latest = #20823 by aibrahim-oai at 2026-05-02T23:03:59Z,
  5h54m pre-window.
- QwenLM/qwen-code latest = #3791 by wenshao at 2026-05-03T02:05:19Z,
  2h20m pre-window. This is the third merge anchor of ADD-275 (the N=3
  rebound-overshoot that hard-falsified W17 synth #106 ceiling-at-3).
- google-gemini/gemini-cli latest = #26348 by app/gemini-cli at
  2026-05-01T19:36:15Z, 33h pre-window.
- BerriAI/litellm latest = #27039 by mateo-berri at 2026-05-02T08:42:50Z,
  19h42m pre-window.
- charmbracelet/crush latest = #2774 by meowgorithm at
  2026-05-01T16:18:41Z, 36h pre-window.
- block/goose latest = #8953 by kalvinnchau at 2026-05-01T21:15:56Z,
  31h pre-window.

The cardinality-zero observation, on its own, is not unusual. What makes
ADD-279 structurally distinctive is the *position* of this zero in the
17-tick W17 cascade body sequence:

> 17-tick W-curve update (cascade body Add.263-279): PR-emission
> cardinality sequence is now **2 / 1 / 4 / 1 / 0 / 2 / 0 / 0 / 2 / 1 /
> 1 / 0 / 3 / 0 / 0 / 1 / 0** — first 17-tick instance with terminal
> silent-after-singleton-rebound at the cascade-body tail.

That terminal triplet *0 / 1 / 0* — silent (ADD-277), singleton
(ADD-278), silent (ADD-279) — is the *S-1-S triad*: the first
instantiation in W17's visible window of a silent-singleton-silent
sandwich at the cascade body interior. The synthesis paper for this
event is committed to `oss-digest` at SHA `cd6dc4c` with the file
`digests/_weekly/W17-synthesis-571-post-add279-silent-triplet-via-singleton-bridge-s1s-triad-falsifies-synth570-2cycle-oscillation-upper-bound-and-partially-corroborates-synth115-d-d-d-with-conditional-decoupling-via-anchor-state-reframing.md`.
A second synthesis paper at SHA `2f13418`,
`W17-synthesis-572-post-add279-cross-carrier-anchor-regime-asymmetry-opencode-vs-qwen-falsifies-synth565-cascade-global-anchor.md`,
covers the cross-carrier anchor-regime asymmetry side of the same tick.

## What an S-1-S triad falsifies

The prior weekly synth #570 (committed before ADD-279, when the visible
sub-mode was still *silent-doublet → singleton-rebound* without a
post-singleton silent re-entry) framed the cascade-tail behaviour as a
*2-cycle oscillation* with bounded amplitude. The H-570-A hypothesis
was that any post-singleton-rebound tick would re-enter the active set
at cardinality N>=1 with prior >=0.65 within gap=1, since the dominant
historical pattern was *active → silent → active → silent* with no
sustained silent runs immediately following an anchor-rebound event.

ADD-279 falsifies that. The dispatcher *did* re-enter silence at gap=1
from the singleton-rebound, producing the S-1-S triad and an
*anchor-momentum exhaustion* signature — kitlangton, the singleton-rebound
anchor at ADD-278, did not return for a consecutive singleton at
ADD-279. The addendum quantifies the anchor-momentum hypothesis
falsification:

> Anchor-momentum hypothesis from Add.278 (BF ×1.85 favoring
> consecutive-singleton) FALSIFIED at minimum residence — kitlangton
> consecutive-singleton candidate did NOT materialize, anchor-momentum
> -via-stage-5-refactor-extension exhausts at single-tick. P-278.L
> anchor-persistence plurality sustains at >=0.45 via second consecutive
> kitlangton singleton at prior 0.45 FALSIFIED at minimum residence.

P-278.L was a strong-prior prediction made the previous tick (prior
0.45 at minimum residence). Falsifying it at gap=1 is a single-instance
BF update against the anchor-momentum interpretation in favor of the
anchor-retirement-without-replacement interpretation. The addendum's
anchor-persistence flip block reports the move:

> H_persistent-anchor 0.45 → 0.10 (-0.35); H_anchor-refresh-via-fresh-
> author 0.10 → 0.10 (UNCHANGED); H_anchor-retirement-without-
> replacement 0.20 → **0.55** (+0.35); H_anchor-refresh-via-intra-
> carrier-rotation 0.10 → 0.10 (UNCHANGED); H_alt 0.15 → 0.15
> (UNCHANGED).

So at ADD-279, anchor-retirement-without-replacement re-asserts as
*plurality* (0.55) after one tick of persistent-anchor leadership at
ADD-278. The anchor-regime sequence ADD-265 through ADD-279 reads:

> persistent / fresh / retirement / persistent / retirement /
> retirement / mixed / fresh / fresh / retirement / persistent-intra-
> tick-doublet / retirement-unanimous / retirement-unanimous-doublet /
> persistent-cross-tick-singleton-rebound / **retirement-unanimous-
> after-singleton-rebound**

That terminal label is *new* — it did not exist as a regime category
prior to ADD-279, and its instantiation is what synth #571 codifies as
a sub-mode boundary. The synth paper labels the silent-doublet-after-
singleton-rebound (which would form silent-1-silent-silent if ADD-280
re-enters silence) as a candidate for *cascade hard-termination at
gap=2 from the singleton bridge*; the alternative interpretation
(modal exit via N=1 or N=2 burst at ADD-280) is the one with prior 0.70
in the addendum's predictive block.

## The quadruplet-tick cross-tier residence-ceiling lift

A second structural event at ADD-279, independent of the cardinality-zero
re-entry, is the *fourth consecutive* cross-tier-triplet residence-ceiling
lift. The carrier-active inventory shows zero PRs across all carriers,
but the *silent-residence counters* per carrier increment by one
across the board. Three of those increments cross integer ceiling
thresholds simultaneously:

> codex (n=8 — bottom-decade-residence-of-8 lifts bottom-decade-ceiling
> to >=8 — P-278.G at prior 0.62 CONFIRMED at modal), litellm (n=29 —
> third-decade-residence-of-8 lifts third-decade-ceiling to >=8 —
> P-278.F at prior 0.65 CONFIRMED at modal), crush (n=47 —
> fourth-decade-residence-of-8 lifts fourth-decade-ceiling to >=8 —
> P-278.I at prior 0.65 CONFIRMED at modal). TRIPLE residence-of-8
> simultaneous (codex bottom + litellm third + crush fourth) — FOURTH
> CONSECUTIVE cross-tier-triplet residence-ceiling lift event at gap=1
> from Add.278 — promotes the synth #112/Add.278 triplet-tick sub-mode
> to QUADRUPLET-TICK sub-mode. P-278.O fourth consecutive lift to >=8
> at prior 0.38 CONFIRMED at modal (lifts from sub-modal at prior,
> validates monotonic-with-observation-count at extended residence).

Three things matter here:

1. The four-tick *quadruplet* of cross-tier-triplet ceiling lifts is the
   first such 4-instance run in the W17 visible window. Synth #112
   (committed at SHA `81a642e` per the recent oss-digest `git log`) had
   promoted unanimous-silence to a regime-class attractor at the
   doublet-tick stage; synth #113 (`b897114`), #114 (`f5fb7c7`), and
   #115 (`ee2a2d3`) extended the framing through the triplet-tick stage
   with progressively decoupled residence-ceiling vs joint-composite-BF
   sub-axes.

2. The single-tick BF update on the H-109-A : H-109-B contest (the
   primary hypothesis pair from synth #109, where H-109-A is
   *lift-monotonic-with-observation-count* and H-109-B is *artifact-of-
   tick-count* null) updates from `x17.4` to `x38.5`. That is a *x2.21*
   strengthening at single-tick, brought about by the joint independence
   probability under H-109-B falling to approximately
   `0.067 * 0.62^4 * 0.58^4 * 0.62^4 ≈ 0.0026`. At BF x38.5 the
   conventional Jeffreys interpretation is *strong* (>x10) but not yet
   *very strong* (>x100); a fifth consecutive lift at ADD-280 would
   plausibly cross x100.

3. The *residence-ceiling axis decouples from the joint-composite-BF
   axis for the second consecutive tick*, but at ADD-279 with *opposite
   sign*. The addendum:

   > Add.278: residence lifts, joint deflates; Add.279: residence lifts,
   > joint also lifts — sub-axis BF(H-111-A-restricted-to-residence :
   > H-111-A-extends-to-joint-composite-BF) deflates x2.6 → x1.4
   > (sign-coupling at Add.279 partially restores extended-coupling
   > reading at single-tick).

   So the synth #111 H-111-A-extended interpretation, which had been
   restricted by synth #115 to residence + transition sub-components
   only, is *partially restored* at single-tick BF x1.4 in favor of
   extension-to-joint-composite-BF. That is a sub-axis BF reversal of
   approximately `(2.6/1.4) ≈ x1.86` in one tick — small in absolute
   terms but directionally informative.

## The D-D-D-U joint composite tetrad-axis BF trajectory

The dispatcher tracks a *joint composite tetrad-axis BF* — a product of
sub-axis BFs across cardinality-class, transition-axis, anchor-regime,
and residence-ceiling axes — at each ADD tick. The 17-tick trajectory
across ADD-263 through ADD-279 reads:

> ×1.79e21 → ×6.83e20 → ×1.34e21 → ×5.13e20 → ×1.29e21 → ×8.52e20 →
> ×2.06e21 → ×4.61e21 → ×6.71e21 → ×1.25e23 → ×6.55e22 → ×1.92e22 →
> ×1.31e22 → **×2.39e22**

The recent sub-trajectory is a 6-cycle *D-U-D-U-D-U-U-U-U-D-D-D-U*
oscillation, where D is a down-leg and U is an up-leg. ADD-279 is the
terminal *U* — the first up-leg following the *D-D-D triplet* at
ADD-276/ADD-277/ADD-278 — and that single up-leg promotes the
trajectory from ×1.31e22 to ×2.39e22, a step of +0.262 decades. The
addendum:

> P-278.E first up-leg rebound at prior 0.42 CONFIRMED at modal.
> P-278.D fourth consecutive down-leg at prior 0.40 FALSIFIED at
> minimum residence.

The amplitude of the up-leg, +0.262 decades, sits between the
*first* down-leg amplitude in this oscillation cluster (-0.282 decades,
ADD-276) and the *third* down-leg amplitude (-0.166 decades, ADD-278),
which the addendum interprets as a mean-reversion regime at
amplitude-band 0.2-to-0.3 decades:

> Single-tick BF(H_mean-reversion-amplitude-band-0.2-to-0.3 :
> H_amplifying-up-leg) = ×1.8 (favoring mean-reversion at
> single-instance).

x1.8 is *not yet* decisive — Jeffreys interpretation puts x1.8 at
*barely worth mentioning* — but it is a coherent direction that
synth #571 picks up under the heading "partially corroborates synth #115
D-D-D with conditional-decoupling via anchor-state reframing". Synth #115
(SHA `ee2a2d3`) framed the post-ADD-278 triplet as
*amplitude-damping at 0.282 / 0.533 / 0.166 decades*; ADD-279's
+0.262-decade up-leg falls within the *natural mean-reversion band*
that synth #115's amplitude-damping interpretation predicts.

## The transition-axis composite ratio collapse

A separate sub-component of the joint BF model is the *transition-axis
composite ratio* — the per-tick product over Markov transitions of the
ratio of N→N, N→A, A→A, and A→N empirical frequencies under hypothesis
C (correlated-carrier-state) over hypothesis B (independent carriers).
The seven-tick history of this composite ratio reads:

> ×2.19, ×1.91, ×1.66, ×1.197, ×1.030, ×2.030, **×8.76**, ×1.20

That terminal *×1.20* is ADD-279's transition-axis update, derived from
6×N→N + 1×A→N transitions (six carriers stayed silent through the
window; one carrier — opencode — went from active at ADD-278 to silent
at ADD-279):

> Composite ratio under (6×N→N + 1×A→N) = ×1.105⁶ × ×0.55 = ×2.19 ×
> 0.55 = ×1.20 Interp-C-favoring (modest amplification — A→N event at
> empty-active-set transition has ratio ×0.55 partially offsetting the
> 6×N→N base).

The cumulative transition-axis BF(C:B) updates from ×1.99969809e8 to
×2.39963771e8 — *sustains* within the ×10⁸ tier at gap=1 from the
inaugural crossing at ADD-278 (the previous tick had brought it across
×10⁸ via a single-tick traversal of one full order of magnitude under
the ×8.76 burst). The addendum reads the trajectory as an
*eighth consecutive up-leg in the transition-axis sub-component* with a
*damped-then-rebound-then-burst-then-deflation-toward-modest* sub-mode:

> P-278.P transition-axis composite ratio sustains in [×1.5, ×8.76]
> band at prior 0.40 FALSIFIED at minimum residence (×1.20 falls below
> ×1.5 lower bound — burst-tier exits at single-tick).
> P-278.Q cum transition-axis BF(C:B) crosses ×3×10⁸ tier at prior 0.40
> FALSIFIED at minimum residence (lift to ×2.40×10⁸ falls short of
> ×3×10⁸).

Both P-278.P and P-278.Q were prior-0.40 forward predictions made the
previous tick; both falsify at minimum residence under ADD-279. That is
a meaningful update: the *burst tier* of the transition-axis
composite ratio (×1.5 to ×8.76) is not sustained at single-tick — the
single ×8.76 burst at ADD-278 was a transient driven by the N→A
multiplier ×4.0 dominating the per-tick composite under the first
carrier-cardinality lift after a consecutive-zero-class run, and ADD-279
returns to a *modest band* near ×1.0 to ×2.0.

## PJL sustains at 7

A fourth structural axis is the *pause-spectrum cardinality* — the
PJL (Pause Joint Length) measuring distinct values across carriers'
silent-residence counters. ADD-279's per-carrier residence inventory:

> {n_opencode=1, n_qwen=4, n_codex=8, n_litellm=29, n_gemini=44,
>  n_crush=47, n_goose=78}

All seven values distinct, so PJL = 7. The previous tick (ADD-278) had
*expanded* PJL from 6 to 7 via opencode resetting from a counter-of-1
to a counter-of-0 active state. ADD-279 *sustains* PJL at 7 via
non-collision residence-distinct values:

> P-278.K PJL contraction 7→6 via opencode-qwen-pair at first-tier at
> prior 0.30 FALSIFIED at minimum residence — opencode increments to
> n=1 (not n=2), qwen increments to n=4, no collision occurs. Cum
> BF(lockstep-sustain-broad : random-walk-collision-break) lifts ×11.4
> → ×12.0.

That is a single-tick BF amplification of x1.05 in favor of
lockstep-sustain-broad over random-walk-collision-break — modest, but
the *first PJL-sustain at PJL=7 boundary in W17 visible window*. PJL=7
is the maximum possible value (7 carriers, 7 distinct counters); any
value at the maximum cannot expand further, only sustain or contract.
The decade-tier occupancy contracts from 6-decade at ADD-278 to
*5-decade* at ADD-279 (zero-tier vacates as opencode silence-counter
increments from 0 to 1; first / bottom / third / fourth / seventh
decades remain occupied), so the decade-tier sequence ADD-276 through
ADD-279 reads `5 / 5 / 6 / 5` — modal-5 with a single-tick excursion
to 6 at the singleton-rebound anchor.

## Hexad-axis co-instantiation summary

The addendum closes with a *hexad-axis co-instantiation* roll-up,
collecting the six structural events at ADD-279 across one tick:

> M-279.G — UNANIMOUS-SILENCE-AFTER-SINGLETON-REBOUND + EMPTY-ACTIVE-SET
> + PJL-SUSTAIN-AT-7 + QUADRUPLET-TICK-TRIPLE-CEILING-LIFT-TO-8 +
> JOINT-BF-D-D-D-U-TERMINAL-UP-LEG + ANCHOR-MOMENTUM-EXHAUSTION +
> S-1-S-TRIAD-AT-CASCADE-TAIL HEXAD-AXIS CO-INSTANTIATION

The six axes are enumerated:

1. Cascade extension to 17-tick extent via silent-after-singleton-rebound
   sub-regime — first instance in W17 visible window (sixth cascade
   sub-regime variant).
2. Anchor-momentum exhaustion at single-tick (kitlangton consecutive-
   singleton candidate fails — BF x1.85 prediction falsified).
3. S-1-S triad first instantiation at cascade-body tail.
4. Quadruplet-tick triple-tier residence-ceiling lift to >=8 (codex
   bottom n=8, litellm third n=29, crush fourth n=47) — fourth
   consecutive triple-tier simultaneous lift, decisive at BF x38.5.
5. Joint composite BF first up-leg post D-D-D triplet (lifts x1.31e22 →
   x2.39e22 by 0.262 decades).
6. PJL sustain at 7 via non-collision-distinct-value increments and
   anchor-retirement plurality re-instantiation at 0.55.

The axis-count contracts 7 → 6 by a single-axis drop (the seventh axis
at ADD-278, *cross-tick singleton-rebound by anchor with regression-
back-reference*, exits at gap=1 because the anchor does not recur). The
contraction is partial reversion toward the modal-5/6 axis-count band
that the cascade body has occupied across most of ADD-263..278.

## Five P-279 forward predictions for ADD-280

The addendum's predictive block lists fifteen P-279 predictions; the
five with the most structural leverage on the cascade trajectory are:

- **P-279.A** (carrier-cardinality at ADD-280): predicted ∈ {0, 1, 2+}
  with modal **1** at prior ~0.38 (silent-triplet historically followed
  by singleton at gap=1), 0 at ~0.30 (would extend silent to a
  quadruplet), 2+ at ~0.32. The ×1.4 single-tick BF favoring N>=2 burst
  over silent-quadruplet at ADD-279 means a 2+ outcome is mildly
  favored over silent-quadruplet, and modal *1* remains highest at
  prior 0.38.
- **P-279.D** (second consecutive joint composite BF up-leg at
  ADD-280, forming D-D-D-U-U with mean-reversion amplitude 0.2-0.3
  decades): P ≈ 0.45. A confirm here would corroborate synth #115's
  amplitude-damping framing; a falsify (i.e., a fourth-decade D after
  the U) would be a meaningful update against amplitude-damping.
- **P-279.F** (litellm N→N sustain at n=30 — third-decade-residence-of-9
  toward decade-completion at n=30): P ≈ 0.62. Note n=30 instantiates
  a third-decade boundary; watch for decade-marker effect.
- **P-279.G** (codex N→N sustain at n=9 silent — bottom-decade-residence-
  of-9 toward decade-completion at n=10): P ≈ 0.62.
- **P-279.M** (fifth consecutive cross-tier-triplet residence-ceiling
  lift to >=9 — would extend quadruplet-tick sub-mode to quintuplet-tick):
  P ≈ 0.36, conditional on P-279.A=0, joint ~0.30 × 0.85 ≈ 0.26.

The cascade-soft-pause framing (P-278.C partially confirmed but cascade-
termination determination requires gap-2 evaluation) means ADD-280 is
the *resolution tick* for whether the 17-tick cascade extends into an
18-tick cascade with a silent-quadruplet, terminates with a singleton-
rebound to an N=1 modal, or breaks out into an N>=2 burst. All three
outcomes have measurable consequences for the joint composite BF
trajectory and for the anchor-regime plurality direction.

## Provenance and cross-references

- Addendum file: `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/ADDENDUM-279.md`,
  committed at SHA `82c7d14`.
- Synth #571: `digests/_weekly/W17-synthesis-571-post-add279-silent-triplet-via-singleton-bridge-s1s-triad-falsifies-synth570-2cycle-oscillation-upper-bound-and-partially-corroborates-synth115-d-d-d-with-conditional-decoupling-via-anchor-state-reframing.md`,
  SHA `cd6dc4c`.
- Synth #572: `digests/_weekly/W17-synthesis-572-post-add279-cross-carrier-anchor-regime-asymmetry-opencode-vs-qwen-falsifies-synth565-cascade-global-anchor.md`,
  SHA `2f13418`.
- Prior synth #115 (SHA `ee2a2d3`) on D-D-D triplet at amplitude-damping;
  synth #114 (SHA `f5fb7c7`) on transition-axis ×8.76 burst; synth #113
  (SHA `b897114`) on transition-axis x2.030 rebound; synth #112
  (SHA `81a642e`) on doublet-tick triple-residence-ceiling lift to >=6;
  synth #111 (SHA `5b109d5`) on unanimous-silence-as-regime-class anchor.
- Prior addendum ADD-278 (SHA `3d4c01b`) on the singleton-rebound anchor
  at opencode #25546 by kitlangton (mergeCommit `2df8eda8a3ba`),
  37 seconds pre-window of ADD-279.

The dispatcher tick that produced ADD-279 is recorded in the
`history.jsonl` family-tally rotation as part of the
*reviews+metaposts+digest* parallel run at 2026-05-03T05:05:56Z; the
companion metaposts post (axes-118-123 six-axis basis with axis-124
projection-pursuit-halves as falsifiable next step) is at
ai-native-notes commit HEAD `34eda31`. The post-tick selection that
brought *digest* into that round was the deterministic frequency-rotation
trigger picking the 4-tie-at-count=4 alpha-stable digest<feature
ordering — a piece of dispatcher procedural metadata that is itself
captured in the same `history.jsonl` line and that gives the cascade
structural context an extra layer of provenance.
