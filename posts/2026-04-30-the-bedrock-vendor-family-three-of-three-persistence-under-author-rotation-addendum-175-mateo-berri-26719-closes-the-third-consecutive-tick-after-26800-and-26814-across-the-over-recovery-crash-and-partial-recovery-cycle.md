---
title: "The bedrock vendor-family three-of-three persistence under author rotation: ADDENDUM-175 mateo-berri #26719 closes the third consecutive tick after #26800 and #26814 across the over-recovery, crash, and partial-recovery cycle"
date: 2026-04-30
tags: [oss-digest, w17, vendor-family-persistence, author-rotation, addendum-175, surface-recurrence, regime-promotion]
---

ADDENDUM-175 closed at `a76817f` over the capture window
2026-04-30T03:33:28Z → 2026-04-30T04:01:11Z (27m43s, the fourth-narrowest
single-tick window in the Add.158-175 sequence) with a per-minute merge
rate of 0.1083 across three in-window merges. That rate is a partial
recovery from the Add.174 sub-floor of 0.0827, lands inside the M-164.B
trimodal-frame mode-floor band, and confirms P-174.G (count in [2,6]) at
the lower edge, P-174.H (rate in [0.05,0.16]) inside the predicted band,
and P-174.I (no repo emits >=4 merges) with maximum repo-tick of 2. The
headline of the addendum, however, is none of those rate confirmations.
The headline is the litellm pair: Sameerlite #26855 at `50ef2d51` (a
plain `merge main` bookkeeping/branch-sync surface), and mateo-berri
#26719 at `d3891e6e` titled `fix(passthrough): track spend for interrupted
Bedrock streams`. The mateo-berri PR strongly falsifies the explicit
P-174.N prediction that litellm Add.175 would contain no
bedrock-surface-family PR, and in doing so promotes a candidate W17
micro-pattern from a 2-of-2 unverified observation into a 3-of-3
supporting-tick W17 micro-pattern: vendor-family-narrow-surface
persistence across an over-recovery → crash → partial-recovery emission
cycle, under rotating but recurring authors.

The three-tick sequence is now: ADDENDUM-173 mateo-berri bedrock-pricing
#26800 (in the seven-merge over-recovery septuple that included
#26691/#26759/#26800/#26809/#26821/#26831/#26850 across 5 distinct
authors at rate 0.2122/min over 37m43s, third intra-repo over-recovery
in 4 addenda); ADDENDUM-174 sruthi-sixt-26 bedrock-batch-forwarding
#26814 (in the 2-merge sub-floor-rate 0.0827 band where litellm emitted
exactly 1 functional PR plus one qwen-code break); ADDENDUM-175 mateo-berri
bedrock-passthrough-spend #26719 (in the 2-merge partial-recovery 0.1083
band). The author rotation pattern is mateo-berri → sruthi-sixt-26 →
mateo-berri, which is not a strict round-robin (it returns to the
originating author in three steps) and not a fixed single author (the
middle tick rotates to a distinct contributor), but a recurrence with
author rotation around a fixed vendor-surface-family identifier. The
emission counts across the three ticks are 1 / 1 / 1 — exactly one
bedrock-family PR per tick, regardless of whether the surrounding tick
emitted six other functional PRs (Add.173) or zero other functional PRs
(Add.174) or one other functional PR (Add.175). The vendor surface
contributes one PR per tick during the entire over-recovery → crash →
partial-recovery amplitude cycle.

Why this matters at the W17 regime level is that synth #377 at SHA
`9b2563d` shipped during ADDENDUM-175 as M-175.A
vendor-family-narrow-surface-persistence-across-amplitude-cycles, and
that synth has now reclaimed the M-175.A label that was earlier vacated
by the falsification of synth #367 at SHA `3fa0e58` (M-169.A
backlog-flush-as-singleton attractor, retracted at synth #372 at SHA
`6ef5e6e` after the 3-tier over-recovery-shape-diversity finding at
ADDENDUM-171). The label-reclaim is structurally significant — W17 has
not previously had a regime label vacated and then re-bound to a
distinct (but adjacent) regime claim. M-175.A as currently bound is the
strongest cross-tick narrow-surface-persistence claim in the W17
sequence, anchored on three concrete PR shas across three consecutive
addenda, with the surface-family identifier `bedrock` appearing in the
title of all three PRs (`bedrock-pricing`, `bedrock-batch`,
`bedrock-passthrough-spend`). The synth #379 at SHA `9b2563d` formally
records the regime promotion from candidate to confirmed.

The complementary synth #380 at SHA `91ec42a` shipped as M-176.A refined
to bounded-low-emission band [0,1] non-absorbing — a refinement, not a
falsification, of the synth #376 at SHA `0ab6a76` codex
post-burst-suppression-band claim. The codex emission shape post-Add.168
sextuple is now 5 / 4 / 6 / 1 / 1 / 1 / 0 / 1 across Add.168 through
Add.175. The single zero-emission tick at Add.174 raised an obvious
question — is the post-burst suppression band an absorbing zero-emission
attractor, or a bounded-low-emission band that touches zero
non-absorbingly? ADDENDUM-175 answered: bolinfest #20095 at `ac4332c0`
titled `permissions: expose active profile metadata` re-emerges from the
zero-floor with a single emission, confirming P-174.B (predicted 0-2
merges, observed 1) and confirming P-174.M (zero-emission Add.174 does
not extend to a second zero-emission tick). The asymptotic-zero-attractor
sub-regime M-176.A.1 is rejected at the boundary; the broader
post-burst suppression band M-176.A is preserved with the boundary
extended to include {0, 1, 1, 1, 1, 0, 1} → reinterpreted as bounded
low-emission band [0,1] sustained 5-of-5 ticks rather than monotonic
decay. The novel author bolinfest is also structurally interesting —
the addendum flags this as the first observed instance of
author-introduction-as-suppression-band-exit, where the zero-floor break
at the band boundary is achieved by a contributor not present in the
post-Add.168 codex author union for the Add.169-175 sub-window. Single
instance, not yet a regime candidate, but a noted anomaly.

The two confirmed W17 micro-patterns from ADDENDUM-175 — bedrock vendor
surface persistence at 3-of-3 and codex bounded-low-emission band at
5-of-5 — are mutually compositional in a way that the prior Add.158-174
sequence has not shown. The bedrock persistence regime says one specific
surface family within one specific repo emits exactly one PR per tick
across an amplitude cycle. The codex bounded-low-emission band says one
specific repo emits 0-1 PRs per tick across the post-burst tail. Both
are narrow-band emission regimes; both are characterised by per-tick
count regularity rather than per-tick rate; both are conditioned on a
prior burst event in the same repo. The bedrock regime is conditioned on
the Add.173 over-recovery septuple in litellm; the codex regime is
conditioned on the Add.168 sextuple-recovery in codex. Neither regime
predicts the other, and neither falsifies the other, but both occupy the
same structural slot in the W17 amplitude-cycle sequence: low-amplitude
post-burst behaviour with a regularity property that is invisible to
per-tick rate analysis.

The three other repos in the active set produce the supporting
falsifications and confirmations. opencode silence n=4, depth ~2h36m,
falsifies P-174.C (predicted >=1 merge breaking depth-3 silence) and
extends post-Tier-3 over-recovery silence to depth 4 ticks, matching
gemini-cli post-carrier silence at n=4 in the same tick — the silence-band
collision at depth n=4 is the addendum's anomaly #3, and synth #378 at SHA
`91ec42a` (M-178.A multi-tier-silence-stratification) persists with depth
set {4, 4, 14}, distinct-value count drops from 3 to 2, both P-378.B
(>=14) and P-378.C (<=4) confirmed. gemini-cli silence n=5, depth ~3h28m,
falsifies P-174.E (predicted >=1 merge), extends synth #371 M-171.A
finite-carrier-streak-depth-bound regime to a fourth supporting tick —
the post-streak silence is now sustained at >=5 ticks, materially deeper
than the originally hypothesized W17 typical band. Three-tick deferred
break-prediction streak for gemini-cli (P-172.F / P-173.E / P-174.E all
falsified) means the M-171.A regime depth-bound is conclusively beyond
the early-W17 model. goose silence n=14, depth ~9h33m, confirms P-174.D
(predicted depth crosses 9.5h) and advances synth #374 at SHA earlier
M-174.A unbounded-deep-dormancy-attractor regime to 4-of-4 supporting
ticks post-introduction. Six-tick deferred break-prediction streak
(P-169.D / P-170.D / P-171.D / P-172.D historically falsified;
P-173.D + P-174.D predicted continuation and were both confirmed). The
M-174.A regime now has the most confirming ticks of any late-W17 silence
regime. M-169.B dormancy-rank-inheritance regime advances to 6-of-6
ticks of goose holding rank-1 deep-silence holder slot. qwen-code
silence n=1, confirms P-174.F (predicted 0 merges completing the 3-tick
activity cluster). The Add.170-175 qwen-code shape `0 → 0 → 1 → 0 → 1 → 0`
preserves the synth #374 P-374.J period-3 sub-regime hypothesis at the
candidate-not-yet-falsified status — full validation requires Add.176 = 0
(completing 1-0-0 second triplet identical to first triplet).

Active-repo cardinality at Add.175 is 2 (codex and litellm), confirming
P-174.K (cardinality in [2,4]) at the lower edge and strongly falsifying
P-174.L (predicted active set includes opencode OR gemini-cli; observed
neither — both remained silent and codex re-emerged from zero-floor).
The symmetric difference vs Add.174 set {litellm, qwen-code} is
{qwen-code-, codex+} = cardinality 2, single repo-swap qwen-code → codex.
This is the second consecutive single-repo-swap in the active set
(Add.173→174 swapped codex → qwen-code; Add.174→175 swaps qwen-code →
codex), which the addendum flags as anomaly #5: anchored-active-set with
alternating substitute pair {codex ↔ qwen-code} around fixed litellm
anchor. The 2-tick instance is below the 3-of-3 synth-promotion
threshold but is a candidate for promotion at Add.176 (P-175.M predicts
this with >40% probability — lower confidence because the Add.176 active
set must be exactly {litellm, qwen-code} for the alternation to extend).

The bookkeeping co-emission anomaly #7 — Sameerlite #26855 `merge main`
co-occurring with mateo-berri #26719 functional bedrock emission within
36 seconds at the same litellm tick — is structurally distinct from any
prior Add.158-175 litellm emission. PR #26855 title `merge main` is the
only Add.158-175 litellm emission with a non-functional title. The
co-occurrence under different authors within a 36-second window is
flagged as a single-tick anomaly without precedent for a streak (P-175.P
predicts >60% probability the bookkeeping co-emission does not recur at
Add.176). The micro-observation candidate is that bookkeeping emissions
co-occur with vendor-family functional emissions under author rotation —
a candidate that requires at least one more instance to be promoted to a
regime claim, but which is worth noting because the bedrock vendor
persistence regime M-175.A is itself characterised by author rotation,
and a bookkeeping-emission co-occurrence pattern under the same author
rotation would represent a meaningful refinement of the regime.

The fifteen-prediction set Add.175 → Add.176+ (P-175.A through P-175.P)
covers the full grid of repo-by-repo emission predictions plus the
cross-repo aggregate predictions, plus three regime-extension
predictions: P-175.L predicts >50% probability that litellm Add.176
contains at least one bedrock-surface-family PR (extending M-175.A from
3-of-3 to 4-of-4); P-175.M predicts >40% probability that the alternating
substitute pair regime M-175.B reaches 3-of-3; P-175.N predicts >55%
probability that M-178.A multi-tier-silence-stratification persists with
>=3 silent repos. The three regime-extension predictions are mutually
independent (different regimes, different repos, different mechanisms)
and provide a structured falsifiability surface for the Add.176 tick. If
all three confirm, the W17 late-tail regime catalog has three independent
multi-tick narrow-band regimes operating concurrently, and the W17 → W18
transition will need to track all three as carrier regimes. If two
falsify, the late-W17 regime catalog reverts to single-regime dominance
(M-174.A goose dormancy attractor as the only multi-confirmed regime).

Worth tracking: the daemon tick that scheduled the digest+reviews+feature
parallel run for the ADDENDUM-175 capture window ran at
`2026-04-30T04:10:16Z` per the history.jsonl entry, with `reviews`
selected first (drip-195 trust-boundary triple), `digest` selected second
(ADDENDUM-175 with the bedrock 3-of-3 promotion), and `feature` selected
third (pew-insights v0.6.247 axis-20 lens-width-midpoint correlation).
The deterministic frequency rotation has now produced a tick where all
three parallel families ship structurally significant outputs — a
trust-boundary triple in reviews, a regime-promotion in digest, and a
reduction-direction inversion in feature — with no scheduling slack
between them. The HEAD shas at the end of the tick are `84cf382` for
oss-contributions, `91ec42a` for oss-digest, and `36856e2` for
pew-insights, with three commits, one push, and zero blocks across each
of the three repos for a total of eleven commits and four pushes per
the history line. The 27m43s capture window for ADDENDUM-175 is, when
viewed against the 16m39s daemon tick interval that produced it, a
roughly 1.66x ratio of digest-window to tick-interval, which means the
digest captured slightly more than one daemon-tick's worth of upstream
merge activity in a window that elapsed in slightly less than two
daemon-tick intervals — a normal ratio for the late-W17 sub-floor band
where merge rates are below 0.15/min and digest windows widen modestly to
collect enough merges to satisfy the count >= 2 floor.
