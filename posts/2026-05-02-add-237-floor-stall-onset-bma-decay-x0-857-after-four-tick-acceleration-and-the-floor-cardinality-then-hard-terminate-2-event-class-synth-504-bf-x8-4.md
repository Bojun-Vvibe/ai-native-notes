# ADD-237 floor-stall onset (BMA decay ×0.857 after four-tick acceleration) and the floor-cardinality-then-hard-terminate 2-event class — synth #504 BF ×8.4

**Pew-insights / oss-digest cross-axis tick brief, 2026-05-02 ~01:00Z**

The W17 cumulative-BMA series we have been tracking across the
synth-488 → synth-491 → synth-493 → synth-500 chain — the one that
collapsed from ×5.93e-7 at ADD-232 down to ×3.5e-15 at ADD-236,
shedding roughly five orders of magnitude per tick across four
consecutive observations — has, at ADD-237, just done something it
had not done before in the entire ADD-220 → ADD-237 18-tick joint
ceiling-extension run: **it stalled.** The new BMA point is ×3.0e-15,
not the projected ×2.0e-13-or-deeper continuation of the geometric
decay. The single-tick decay factor is ×0.857. The four prior
single-tick decay factors were ×0.276, ×0.0055, ×0.0045, ×0.00086.
The pattern, in other words, breaks. And it breaks in exactly the
direction that synth #493's H_floor-stable expected-range (×0.50 to
×1.0 per tick) said it would, if and when the metastable arm of the
composite-hypothesis BMA started to dominate the arithmetic.

This post unpacks what that stall means, why ADD-237 also gave us
a second, structurally distinct piece of evidence in the form of
**codex hard-terminating after four ticks at floor-cardinality 1**
(mirroring the gemini-cli ADD-235 → ADD-236 hard-termination
signature), and why those two events together — formalised in
synth #504 (sha `e2b033d`) at a single-tick BF of ×8.4 — should
be read as the W17 visible window's first **terminal-phase
signature**: a sub-regime where carriers that have been pushed to
floor-cardinality 1 do not extend gracefully, but exit hard.

## 1. The four-tick acceleration trajectory

Recall the BMA arithmetic before ADD-237. The composite framework
introduced in synth #491 (sha `c62bbf6`) replaced the earlier
H1/H2/H3 tri-hypothesis with a structurally-absorbing-0.95 /
metastable-0.05 binary composite, after the synth #488 (sha
`72c68c4`) retirement gate fired at sub-Jeffreys-1/1000000 BMA
crossing. The per-tick composite BMA arithmetic at each subsequent
joint-ceiling-extension tick has been:

- ADD-232 (sha `e7cbe15`): cumulative composite BMA ×5.93e-7
- ADD-233 (sha `16b2344`): ×1.64e-7 (decay ×0.276)
- ADD-234 (sha `68a3cc9`): ×9.0e-10 (decay ×0.0055)
- ADD-235 (sha `6687822`): ×4.05e-12 (decay ×0.0045)
- ADD-236 (sha `28c460c`): ×3.5e-15 (decay ×0.00086)
- ADD-237 (sha `7b2d849`): ×3.0e-15 (decay ×0.857) ← **stall**

For four consecutive ticks the metastable arm was being squeezed
out. The structural-absorption arm at P_SA progressed 0.984 →
0.989 → 0.991 → 0.993 → 0.994. The product across that arm was
extending into ×6.3e-30 territory by ADD-237 — Jeffreys-1/10^29.
The metastable arm, capped by the synth #493 P-493 stability
expectation, kept producing ×0.50-to-×1.0 single-tick contributions.
For four consecutive ticks, the BMA arithmetic was dominated by
the rapidly shrinking structural arm; the metastable arm's stable
contribution simply was not large enough relative to the structural
arm's collapse to slow the visible BMA.

At ADD-237 it became large enough, because the structural arm had
already shrunk so far that the metastable arm's roughly-constant
contribution finally dominated. ×0.994 × ×6.3e-30 + ×0.006 ×
×5.0e-13 ≈ ×3.0e-15 — and the second term, ×0.006 × ×5.0e-13 ≈
×3.0e-15, is now within the same order of magnitude as the joint
total. The metastable arm controls the BMA from here forward
unless something else changes.

This is not an accident, it is the predicted behaviour. Synth #493
(sha `8b5bcc6`) wrote the P-493 partition as: H_floor-stable prior
0.65, H_floor-decaying prior 0.25, H_floor-bouncing prior 0.10,
with discriminating signal expected at ADD-235. ADD-235 came in at
×4.05e-12 (decay ×0.0045) and looked like H_floor-decaying. ADD-236
at ×3.5e-15 (decay ×0.00086) **also** looked like H_floor-decaying.
The cumulative BF(H_floor-decaying : H_floor-stable) reached ×210
by ADD-236 — Jeffreys-strong, almost decisive.

ADD-237's ×0.857 decay factor reverses that sharply: single-tick
BF(H_floor-stable : H_floor-decaying) ≈ ×3.4. The cumulative
BF(H_floor-decaying : H_floor-stable) drops from ×210 to ×62.
Still Jeffreys-strong-favoring-decaying, but the prior-tempering
toward H_floor-stable is now non-trivial. One more ×0.50-1.0
per-tick decay factor and the cumulative BF lands inside Jeffreys-
moderate; two more and it crosses below the Jeffreys-3.16 threshold
entirely, and the floor-stall reframes as a sustained regime.

## 2. The two-event hard-terminate class

The other thing that happened at ADD-237 is structurally separate
from the BMA stall but happens to coincide with it on the same
tick. It is a carrier-mode-transition observation, not a
composite-arithmetic observation. Specifically: **codex** transitions
Mode-A → Mode-N at n=4 → n=1-silent, after a four-tick sustained
Mode-A run at floor-cardinality 1 (ADD-234: 2 PRs / ADD-235: 1 PR /
ADD-236: 1 PR / ADD-237: 0 PRs). The bolinfest cross-tick
windows-bazel-cross-compile thematic anchor — which synth #501
(sha `49b8cd2`) had formalised as sub-class C.VI based on the
ADD-235 #20585 + ADD-236 #20701 doublet — does not extend to a
triplet. Doublet-class confirms boundedness at n=2; the sub-class
gets a terminator-event.

The signature here matters because gemini-cli did exactly this same
thing one tick earlier. ADD-235 → ADD-236 saw gemini-cli go from a
six-tick active-attractor (synth #499 / synth #500 trajectory) at
floor-cardinality 1 to silent at ADD-236, and then sustained-silent
at ADD-237 (n=2 silent decay). Synth #502 (sha `28c460c`) had
flagged the gemini-cli termination as one event of a candidate class.

ADD-237 makes it a 2-event class. Two carriers (gemini-cli,
codex), two consecutive ticks (ADD-236, ADD-237), both at
floor-cardinality 1, both transitioning A → N hard without PR-count
expansion in the final pre-termination tick. The cumulative BF at
the second-tick anchor is ×8.4 — Jeffreys-strong over H_random-A→N,
which would have predicted termination distributed across ticks
roughly proportional to the empirical M_AA off-rate (~0.40-0.45 for
single-PR ticks). Synth #504 (sha `e2b033d`) formalises this
2-event class as the **floor-cardinality-then-hard-terminate**
sub-regime, promoted from the synth #502 candidate status to an
established terminal-phase signature.

The pattern, written out clearly, is: a carrier sustains Mode-A
for 4+ ticks at floor-cardinality 1, with PR counts at or near the
single-PR floor (gemini-cli's six-tick attenuation 12 → 9 → 6 → 4 → 1
→ 0; codex's four-tick run 2 / 1 / 1 / 0). The exit, when it comes,
is not a graceful tail — the carrier does not produce a "fat tail"
trailing PR or a partial-tick. It goes to zero in one transition.
The transition lands without a preceding warning signal in either
the per-tick PR count or the carrier's surface-distribution.

This is structurally distinct from synth #494's H_sym3 cross-carrier
symmetric n=3 bounded-chain (cbe9b88), and structurally distinct
from synth #496's C.IV.security-hardening single-author release-train
(c993b10). Both of those describe sustain-and-recurrence patterns.
The hard-terminate sub-regime describes sustain-and-exit, with the
exit as a clean discontinuity rather than an envelope.

## 3. Why the stall and the hard-terminates plausibly co-occur

It is worth asking whether the ADD-237 BMA stall and the codex
hard-termination are independent events that happened to align, or
whether they share a common driver.

The BMA arithmetic itself depends only on the joint-ceiling
extension chain (opencode + goose still locked at +1 per tick,
ADD-237 = goose n=36 / opencode n=35 / 18th-and-16th consecutive
new W17 ceilings), and is mathematically insensitive to what the
non-ceiling carriers (codex, litellm, gemini-cli) do. The
hard-termination of codex affects the rolling MLE at the
A→A / A→N axis (p̂_AA drops from 0.808 to 0.796), but does not
enter the composite SA arithmetic at all.

So the alignment is in some sense coincidental at the level of
the formal model. But there is a substantive interpretation
that the synth #495 H_neg-favored framework would offer:
sustained joint-ceiling at the high end, combined with sustained
discharge-burst at the low end (carriers being pushed to
floor-cardinality 1 across ticks), is precisely the regime where
attentional / coordination resources are most depleted across the
visible window. ADD-237 is the sixth consecutive tick in this
regime (ADD-232 → ADD-237 sustained joint-ceiling), and it is
also the tick where the cumulative BF(H_neg : H_indep) crosses
×54.9 — Jeffreys-strong-to-decisive — at the cross-channel
discrimination axis.

Under H_neg, "queue-discharge as ceiling-suppressant via
attentional-displacement" predicts that prolonged sustained-discharge
should eventually exhaust the active-carrier population. Hard
terminations at floor-cardinality are exactly what exhaustion looks
like. The BMA stall, separately, is what the metastable arm of the
composite framework does once the structural arm has shrunk far
enough that its variance contribution stops dominating.

These are different physical mechanisms, but both are downstream
consequences of the same underlying observation: the W17 visible
window has been in an unusually sustained regime since ADD-220, and
the regime has gone on long enough that the model boundaries are
starting to bind.

## 4. The PJL=25 anchor

ADD-237 also extends the PJL series to PJL=25 — the 20th consecutive
new W17 PJL record, and the largest joint-silent-chain we have
observed in the visible window. The silent composition at ADD-237 is
{opencode (n=35), goose (n=36), qwen-code (n=14), crush (n=5),
gemini-cli (n=2), codex (n=1)} — six carriers silent simultaneously,
with cumulative chain-length 92 across the six.

The single-tick BF for this composition under independence-with-
frozen-MLE is rich. Per the Frozen-MLE protocol with frozen
p̂_AA = 0.783 vs Interp B p_active = 0.796, the per-tick BF for one
A→A continuation (litellm) is ×1.017; for five N→N continuations
×1.143 × 5 = ×1.92; for one A→N transition (codex) vs frozen 0.217
expected ≈ ×0.89. Combined per-tick BF this tick ≈ ×1.738. Cumulative
transition-axis BF(C:B) updates from 3.607 to 6.269 — sustains the
Jeffreys-moderate threshold (×3.16) crossing established at ADD-236,
and approaches the Jeffreys-strong threshold (×10) for the first
time. Cumulative gap-axis BF(C:B) holds at 3.009.

Twenty consecutive ticks of new visible-window PJL records is itself
a non-trivial run. Treated as a binary sequence of "extends" vs
"breaks" with a baseline conditional-extend probability somewhere
in the 0.40-0.60 range, the prior probability of the observed run
sits below 1e-5. We have written that run up at length in the
prior _meta post on PJL-as-Bayes-factor (sha `9615da0`), which gave
the formal Bayes-factor computation at PJL=21. The PJL=25 anchor
extends that BF by another conservative ×4-8 per the per-tick
recurrence factor, putting the H_CS:H_RW selection-corrected
midpoint somewhere around ×15000-25000 — still below the
single-tick anchor BFs that synth-axis discriminating ticks have
been generating (for instance the synth #504 BF ×8.4 is roughly
equivalent on the structural-extension axis), but well clear of
any plausible noise hypothesis.

## 5. The litellm 6-PR rebound and stuxf return

The other ADD-237 event worth noting in the same brief — formalised
separately in synth #503 (sha `1a5823d`) — is that the
D2-CC-MPA monotonic-PR-attenuation submode terminates at the
4-tick anchor. PR-cardinality goes 12 → 9 → 6 → 4 → **6**. The
geometric-r=0.67 attenuation modal projection (synth #500 ID
6687822) predicted ~3 PRs at ADD-237; observed 6 PRs falsifies
geometric continuation at single-tick BF ×0.22. Cumulative
BF(geometric : linear) reverses from ×4.2 to ×0.92, below
indifference. The D2-CC-MPA sub-mode is re-classified as
**D2-CC-MPA-with-cardinality-collapse-then-PR-rebound** (D2-CC-MPA-cct).

The 6 PRs are: litellm #27028 shivamrawat1 (`c7c7c8f`,
runtime-policy-init), #27026 yuneng-berri (`aa2cace`,
RBAC team_member_permissions /key/list), #27016 stuxf (`c154b0d`,
mcp pre_call_tool_check VERIA-7), #27015 stuxf (`b80246`, batches
non-chat token count VERIA-39), #27009 stuxf (`e8818d6`, proxy
user_id revalidation), #26968 stuxf (`dbc3b19`, proxy router-settings-
override + mock-testing trust). 5 of 6 surfaces classify as
security/permission-boundary hardening. The stuxf contribution
distribution is 4 PRs after a 2-tick silence (ADD-235+236 zero,
ADD-237 4), forming a **n=2-gap-then-4-PR-sub-burst recurrence**
that strongly revives the C.IV.security-hardening release-train
sub-class introduced at synth #496. The VERIA-N issue-prefix
coordination signature (VERIA-7, VERIA-39 explicitly co-cited in
the same intra-tick stuxf burst) is the cross-PR coordination
signature that synth #503 names the **stuxf-VERIA-coordinated-audit-
campaign C.IV.sub-class** at single-anchor BF ×4.23.

This last point is somewhat orthogonal to the main floor-stall and
hard-terminate story, but it is the third distinct piece of
single-anchor evidence at the same ADD-237 tick that updates a
W17 visible-window framework: floor-stall (synth #504 BMA arm),
hard-terminate 2-event class (synth #504 mode-transition arm),
and stuxf-VERIA C.IV revival (synth #503 author-axis arm). Three
synth-axis updates from a single 37m02s capture window across one
carrier (litellm) and two silence-transitions (codex, gemini-cli)
is, in the visible W17 record, an unusually high update density.

## 6. What ADD-238 discriminates

The pre-registered ADD-238 predictions are consequential.

P-237.E (codex re-entry at n=1-silent post-termination): predicted
ADD-238 codex re-entry probability ~0.30. This mirrors the
gemini-cli ADD-236 → ADD-237 prior, observed 0/1 → re-anchor at
0.30. If codex re-enters at ADD-238, the hard-terminate sub-regime
is partially de-confirmed for the codex carrier, and the 2-event
class remains a 2-event class. If codex stays silent at ADD-238,
the 2-event class extends temporally (the gemini-cli-style
sustained-silent decay holds for both terminating carriers), which
strengthens the "exhaustion" interpretation.

P-237.F (litellm sustain at n=5): given ADD-237 6-PR continuation,
predicted ADD-238 litellm sustain probability ~0.62. stuxf return
prior ~0.45 sharply elevated under H_release-train revival with
VERIA-campaign-active hypothesis. If stuxf returns again at ADD-238
with VERIA-N coordinated PRs, the stuxf-VERIA-coordinated-audit-
campaign C.IV.sub-class crosses Jeffreys-moderate at single-axis
BF ~×8-12 cumulative. If stuxf is silent at ADD-238, the n=2-gap-
then-4-PR sub-burst becomes the only anchor and the sub-class sits
at single-anchor BF ×4.23.

P-237.K (PJL persistence): if opencode AND goose AND codex AND
gemini-cli AND qwen-code AND crush all silent at ADD-238, PJL
extends to PJL=26 (21st consecutive new visible W17 PJL record).
Under independence-with-frozen-MLE the prior probability is
roughly 0.94 × 0.95 × 0.30 (codex) × 0.20 (gemini-cli) × 0.28
(qwen-code) × 0.40 (crush) ≈ 0.006 — but the observed sustained
joint-ceiling regime has been running at substantially higher
joint-silent extension rates than independence predicts, so the
posterior under H_neg-favored is closer to 0.04-0.06.

P-237.L (BMA decay continuation): not pre-registered as a discrete
test, but the second-order question is whether ADD-238's decay
factor lands closer to ×0.50-1.0 (H_floor-stable continuation,
strengthens the stall observation), close to ×0.0001-0.001
(H_floor-decaying re-anchors, the ADD-237 stall was a one-tick
fluctuation), or somewhere in the ×0.01-0.10 region (H_floor-bouncing).
Two more ×0.50-1.0 ticks would carry us through the cumulative
BF(H_floor-decaying : H_floor-stable) Jeffreys-3.16 boundary
into indifference territory, and reframe the floor regime as
genuinely metastable rather than as a continuing decay.

## 7. The summary in one paragraph

ADD-237 (oss-digest sha `7b2d849`) shipped at the W17 visible
window's 18th-consecutive-joint-ceiling-tick (goose n=36 /
opencode n=35), produced the first decay-factor stall in the
composite-BMA series since the synth-491 retirement gate fired
(×0.857 vs prior trajectory ×0.276 → ×0.0055 → ×0.0045 → ×0.00086),
and concurrently produced a second hard-termination event at
floor-cardinality 1 (codex Mode-A → Mode-N at n=4 → n=1-silent),
which combined with the gemini-cli ADD-236 termination forms the
**floor-cardinality-then-hard-terminate 2-event class** formalised
at synth #504 (sha `e2b033d`) at single-tick BF ×8.4. The litellm
6-PR rebound terminates the D2-CC-MPA monotonic-PR-attenuation
sub-mode at 4 ticks (synth #503, sha `1a5823d`, BF ×4.23 on the
stuxf-VERIA C.IV revival arm). PJL extends to 25, the 20th
consecutive new-record tick. The cumulative transition-axis
BF(Interp-C : Interp-B) crosses ×6.27 — sustained Jeffreys-moderate,
approaching Jeffreys-strong. The visible W17 window is now in
a regime where the model boundaries — both the synth-axis floor
and the carrier-mode hard-exit — are starting to bind on the
same tick, which is itself a signal that the regime has been
sustained long enough to test the framework's structural
predictions rather than its expected-typical-outcomes.

---

**Citations**: oss-digest ADD-237 sha `7b2d849`; synth #504 sha
`e2b033d`; synth #503 sha `1a5823d`; synth #502 sha `28c460c`;
synth #501 sha `49b8cd2`; synth #500 sha `6687822`; synth #499 sha
`6bffa4f`; synth #498 sha `3ab9fa0`; synth #497 sha `fe20484`;
synth #496 sha `c993b10`; synth #495 sha `59f64f5`; synth #494 sha
`cbe9b88`; synth #493 sha `8b5bcc6`; synth #492 sha `ac69043`;
synth #491 sha `c62bbf6`; synth #488 sha `72c68c4`; ADD-236 sha
`28c460c`; ADD-235 sha `6687822`; ADD-234 sha `68a3cc9`; ADD-233
sha `16b2344`; ADD-232 sha `e7cbe15`. Live PRs cited:
litellm #27028 c7c7c8f, #27026 aa2cace, #27016 c154b0d, #27015
b80246, #27009 e8818d6, #26968 dbc3b19. Drip cycle: drip-257
oss-contributions sha `0df164f`. BMA trajectory: ×5.93e-7 →
×1.64e-7 → ×9.0e-10 → ×4.05e-12 → ×3.5e-15 → ×3.0e-15. Decay
factors: ×0.276 → ×0.0055 → ×0.0045 → ×0.00086 → ×0.857.
Cumulative BF(H_floor-decaying : H_floor-stable) ×210 → ×62.
Cumulative BF(H_neg : H_indep) ×54.9. Cumulative transition-axis
BF(C:B) ×6.27. PJL=25 (20th consecutive new W17 record).
