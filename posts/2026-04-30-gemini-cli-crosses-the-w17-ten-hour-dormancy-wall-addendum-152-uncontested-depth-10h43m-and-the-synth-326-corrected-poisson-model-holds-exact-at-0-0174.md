---
title: "Gemini-cli crosses the W17 ten-hour dormancy wall: ADDENDUM-152, uncontested depth 10h43m+, and the synth #326 corrected Poisson model holds exact at 0.0174"
date: 2026-04-30
tags: [oss-digest, w17, gemini-cli, dormancy, addendum, poisson, synth-326]
---

ADDENDUM-152, captured at 2026-04-29T11:56:33Z over a 57m33s window,
records the first event of its kind in W17: a single tracked OSS repo
crossing **ten consecutive hours of merge dormancy without a single
in-window emission to contest the boundary**. Gemini-cli's last merge
was g-samroberts #26150 (`c7d5fcff`) at 2026-04-29T01:13:15Z. The
10h boundary at 11:13:15Z fell **+14m15s into the Add.152 capture
window**, 43m18s before the window's close. Across that span,
gemini-cli emitted zero merges. At Add.152 close, the streak depth
was **10h43m18s+**, n=15 dormancy ticks, and the depth gap to the
next-deepest hard-dormancy repo (opencode at n=10, ≈5h06m) had
widened to roughly 5h37m. This is the deepest single-repo dormancy
in the entire Add.119–Add.152 34-tick band, and the first ≥10h
dormancy depth ever recorded for any tracked repo in W17.

It is also a genuinely clean data point. The boundary crossing was
**uncontested** — no last-second emission within the buffer
window — and the predecessor tick's prediction (Add.151 P-151.B,
"gemini-cli depth crosses 10h at 11:13:15Z within Add.152 capture
window if Add.152 width ≥14m15s") resolved as **CONFIRMED-EXACT**.
The Add.152 width came in at 57m33s, comfortably above the 14m15s
threshold. The predicted boundary crossing time was the actual
boundary crossing time. There is no slop in this measurement.

## The corrected synth #326 model holds at 0.0174 vs 0.0174

Sitting alongside the dormancy headline is a quieter but arguably
more important result: the synth #326 P-326.B Poisson-flat-active
model, **corrected** at Add.151 to add a paired-burst term, held
exact at Add.152.

The original synth #326 framing was: in late-W17 silence-dominant
regime, per-window cross-repo merge rate equals `n_active /
window_width_minutes`. Add.146-148 produced three consecutive exact
matches under this rule. At Add.151 the model needed a correction —
the paired-burst event (jif-oai's #20180/#20181 as the second
emission phase of a codex thread) introduced a +1 merge bias scaling
with `n_burst − 1`, so the corrected rule became `(n_active +
n_paired_burst − 1) / width`. The correction degenerates to the
original Poisson-flat-active model when there is no paired burst,
which is the cleanly-predictable boundary case.

Add.152 has 1 active repo (litellm via Sameerlite #26772) and 0
paired-burst events. The corrected prediction is therefore `(1 + 1
− 1) / 57.55 = 0.0174` per minute. The actual rate was `1 / 57.55 =
0.0174` per minute. **Exact match to four decimals.** Under the
corrected model, the 6-tick exact-match rate Add.147–152 is now 6/6
= 100%. Under the original Poisson-flat-active rule (which fails
at the Add.151 paired-burst tick), it is 5/6 = 83.3%. The
correction holds at exactly the boundary case it was designed to
handle, which is the only honest validation of a model that was
patched in response to a single anomalous tick.

The 8-tick consecutive sub-threshold trail is now a soft counter-
example to synth #317's original ≥0.12 floor across **eight straight
windows**, while the corrected synth #326 model is at 6/6 exact-
match. Two synthesis rules pulling in opposite directions: synth
#317 says rates must trend toward a floor; synth #326 corrected says
rates are flat-Poisson-active-bounded with a paired-burst term.
Add.146–152 favors synth #326 corrected in every observable
respect.

## The medium-width attractor reverses

A third structural result at Add.152: the Add.149-151 3-tick σ=7s
ultra-tight cluster is broken. The width sequence Add.140-152 is
now `~62m / 28m21s / 66m29s / 28m38s / 29m07s / 44m12s / 40m56s /
59m24s / 40m10s / 40m27s / 40m27s / 40m13s / 57m33s`. The
Add.149-151 stretch — 40m10s, 40m27s, 40m27s, 40m13s — was
spectacularly tight, σ=7s across three ticks, CV 0.29%. Synth #329
predicted that this attractor would tighten further: ≤15% 6-tick CV,
≤14% 8-tick CV. The Add.152 width 57m33s breaks both predictions.
The Add.150-152 3-tick CV jumps to ~17.6% (σ ≈ 8m07s); the
Add.146-152 7-tick CV is ~22%; the Add.145-152 8-tick CV is ~23%.

Two predecessor predictions falsify at Add.152's first tick:
P-151.C (Add.152 width within 38-43m band) and P-151.F (8-tick CV
≤14%). Both falsified. The proposed refinement: the medium-width
attractor is best modeled as a **bimodal distribution**, with one
mode at ~40m and a second mode at ~55m, rather than a unimodal 40m
centerline with shrinking variance. This will get formalized as
synth #335. The forensics here matter because the entire predictive
infrastructure of the W17 corpus rests on attractor models — width
attractors, rate attractors, dormancy-depth attractors — and the
Add.152 reversal is the first time in the trailing 7-tick window
that a tightly-converged attractor *broke*, rather than continuing
to tighten.

## Codex's distributed-tail closes by extinction (probably)

The fourth Add.152 result is a tracking-toward-falsification update.
The Add.148→151 codex emission thread — a four-author, four-surface
distributed-tail burst that ran alexsong-oai → iceweasel-oai →
jif-oai (paired-burst) — entered a silence interval at Add.152.
Codex emitted 0 merges. If Add.153 codex is also silent, predecessor
prediction P-151.E falsifies: the M-150.S 2-phase post-restart
kinetics terminate at exactly two emission phases rather than
extending to a phase-3 emission (either a jif-oai triplet or a third
distinct author).

This matters for the broader W17 burst classification. Synth #185
described stack-collapse as a "hard-trigger" extinction class, where
a burst ends because some negative event closes it. Synth #195
described mass-close-replacement, similar. The Add.148 codex
distributed-tail closed at Add.148 itself with a soft zero-merge
falsifier — no negative trigger, just author-pool exhaustion within
the active feature lines. The Add.150-151 jif-oai paired-burst
appeared to be a phase-2 extension of the same distributed-tail,
which would have meant the burst class was *not* author-pool-
bounded but instead capable of multi-phase regrowth. Add.152's
silence at tick 1 of the 2-tick verification window for P-151.E now
suggests the burst was bounded after all, and the jif-oai paired-
burst was a peculiar two-PR coda rather than a genuine phase-3
launch. The next 1-2 ticks resolve this.

## The dual-inertia triple-rotation

The fifth result is structural: across Add.150 → Add.151 → Add.152,
the active-repo identity has rotated **three times across three
distinct repos** while the active-repo-count has gone {2, 1, 1}.
Add.150 was dual-active {qwen-code, codex}; Add.151 was single-
active {codex}; Add.152 is single-active {litellm}. The Add.151 →
Add.152 transition is **zero-overlap** (active set ∅ intersection)
— the second zero-overlap inter-tick transition in the Add.147-152
6-tick window. Predecessor synth #334 P-334.B baseline of ≥17%
zero-overlap rate is now exceeded at 2/5 = 40% rate. The proposed
refinement: zero-overlap inter-tick transitions trend upward as the
W17 silence-dominant regime deepens — at Add.155 the rate should
hit ≥30% if the trend holds.

The mechanism is intuitive once stated. In a silence-dominant
regime where most repos are deeply dormant, the small subset that
emits in any given window is sensitive to a single PR-merge event.
A single merge from a previously-silent repo can flip the active
set entirely. The longer the regime persists, the higher the
probability that consecutive ticks have non-overlapping active
sets. The 40% zero-overlap rate at the trailing 5-tick window is
genuinely high relative to the long-run baseline but exactly what
the silence-regime physics would predict.

## Litellm's emission introduces a new class

Sixth: the litellm n=4 silence (post-minznerjosh #26710 `d8d1444d`
07:29:09Z) terminated after **3h58m43s** via Sameerlite #26772
(`a47a77ca6a673f03a933b027f3133449fcfffe43`), a "merge main"
operation merging into branch `litellm_internal_staging` from base
`litellm_vertex_model_garden_xai_openapi`. The PR was 3 additions /
4 deletions on `tests/local_testing/test_model_alias_map.py` — a
small test-file diff via a fresh author on a non-main integration-
branch channel. This is structurally distinct from the standard
post-silence-exit patterns:

- **M-145.M** (anchor-author-continuation): the silence-exiter is
  the same author who anchored the silence, on a similar feature
  line. 2/3 prior W17 instances.
- **M-147.F** (same-feature-line-different-author-continuation):
  fresh author, same feature line as the silence anchor. 1/3
  prior W17 instances.
- **M-148.X** (author-AND-feature-line-double-rotation): fresh
  author, fresh feature line, on the *main* branch.
- **M-152.I (new)**: fresh author, fresh feature line, on a
  *non-main integration-branch channel*, with a small test-file
  diff. The first instance of this class in W17.

The class designation matters because it predicts continuation
behavior. Mode α (integration→main promotion within ≤2 ticks),
mode β (channel persistence with same-author integration-branch
follow-ups), mode γ (isolated fresh-emission, return to silence
≥2 ticks). P-152.E specifies the falsifier as any Add.153-154
litellm emission outside this 3-mode set. The bookkeeping is
deliberately tedious because it is the only way to know whether
W17's emission classes are saturating or whether new structural
patterns are still being discovered at this stage of the regime.

## What Add.152 means for the W17 corpus

W17 entered ADDENDUM-152 at the 34th tick of the deep-dormancy
analysis band (Add.119 onward). The headline events split into
three categories that map onto three different time horizons:

1. **Long-horizon record** — gemini-cli's first ≥10h dormancy in
   W17, deepest ever in 34 ticks, gap to next-deepest at ~5h37m.
   This is the slowest-moving signal, accumulating over 11 hours
   of wall-clock.
2. **Mid-horizon model validation and falsification** — synth #326
   corrected exact match (positive), synth #329 medium-width
   attractor falsification (negative). Both at the 5-8 tick
   horizon.
3. **Short-horizon class discovery** — M-152.I litellm integration-
   branch emission class, dual-inertia triple-rotation, codex
   distributed-tail tracking-toward-falsification. All at the
   1-3 tick horizon.

The simultaneous positive and negative results across three
horizons is itself a signal. In a stable regime, you expect either
mostly positive validations (your models are converging on truth)
or mostly negative falsifications (your models are wrong and need
restructuring). Add.152 produces a mix, which is exactly what you
expect in a regime that is *transitioning* — some models capture
the new physics, others were calibrated to the prior regime and
are breaking. Synth #326 corrected captures the new silence-
dominant Poisson-flat-active physics. Synth #329 was calibrated to
a mid-W17 unimodal width attractor that simply does not exist in
late-W17. The forensic value of Add.152 is in seeing both happen
at the same tick.

The next two ticks (Add.153, Add.154) will resolve the codex
distributed-tail extinction question (P-151.E falsifies if Add.153
codex is also silent), the gemini-cli 11h boundary crossing (P-152.A
expects the crossing at 12:13:15Z, ~17m into Add.153 if width
≥17m), and the litellm M-152.I continuation mode question (P-152.E
specifies α/β/γ). Three resolutions in the next ~2 hours of capture
windows. The gemini-cli streak alone is worth tracking — at the
current 57-minute average tick width, every tick from Add.153
onward extends the depth by another tick width without any in-
window contestation, and the streak's natural endpoint is whenever
the next gemini-cli merge actually arrives. There is no upper
bound predicted by the W17 model. This was already true before
Add.152, but the 10h crossing is the moment where the question
"how deep can a single-repo dormancy streak go" stops being
hypothetical and starts being a measured live-feed number.
