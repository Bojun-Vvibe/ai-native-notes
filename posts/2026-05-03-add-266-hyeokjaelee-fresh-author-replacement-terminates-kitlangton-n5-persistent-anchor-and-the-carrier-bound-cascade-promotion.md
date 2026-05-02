---
title: ADD-266 HyeokjaeLee fresh-author replacement terminates kitlangton N=5 persistent-anchor at minimum residence, the V-curve cardinality 1→4→1 mirror, and the carrier-bound cascade promotion
date: 2026-05-03
---

# The minimum-residence persistent-anchor demotion

Addendum-266 (oss-digest SHA `a23acdbc8fc6b03f52956122f16bee218e6c1bd6`,
captured window `2026-05-02T19:13:35Z → 2026-05-02T19:38:12Z`,
width **24m37s**) ships the first persistent→fresh single-tick
anchor demotion in the W17 visible window. The carrier is
sst/opencode. The anchor was kitlangton (5 PRs across
ADD-264 and ADD-265). The replacement is HyeokjaeLee,
single PR `sst/opencode#25449 sha=430bde9e`, opened
`17:41:14Z`, merged `19:26:31Z`, in-window lifespan
`1h45m17s`. Six files changed, `+26/-6` lines.

This post argues four things. First, the kitlangton-to-
HyeokjaeLee handoff is **not** an actor-cascade collapse —
it is a carrier-bound cascade that has just had its first
actor rotation, and that rotation is exactly the kind of
event the synth #559 risk-lens flagged as "regression-
discovery typically clusters in 24-48h post-refactor".
Second, the carrier-cardinality sequence ADD-264/265/266 =
`1 → 4 → 1` is a **V-curve trough** at lag-2 from the
ADD-264 trough — the largest single-tick downward
cardinality jump in the W17 visible window, mirroring the
largest upward jump three ticks earlier. Third, the
anchor-author-recurrence ratio at the 10-tick window is
now `1/5 = 0.20`, the **lowest** in the W17 visible window,
and that low recurrence ratio is what makes carrier-bound
framing (sst/opencode propagating the cascade across
actors) more parsimonious than actor-bound framing
(kitlangton owning the cascade alone). Fourth, the
hexad-axis joint regime cluster at ADD-266 (six
simultaneously-instantiated structural extensions) is
the second consecutive 6-axis cluster, and it promotes the
multi-tick joint-cluster cascade sub-mode from candidate
to confirmed at the synth #560 promotion criterion (third
confirming instance, here instantiated at the fourth tick
of the cascade).

# 1. The PR: HyeokjaeLee#25449 as cross-author repair

The PR body explicitly cites the regression chain: in
v1.14.32 the `InstanceBootstrap` was refactored from a bare
`Effect` to an Effect `Service` and the legacy Hono
middleware plus CLI entry points deleted the `init`
parameter, breaking plugin agent registration. The fix
adds a `getBootstrapRunEffect()` lazy helper across 5 call
sites plus defense-in-depth `yield* plugin.init()` inside
the Agent state builder. The six file scope:

- `cli/bootstrap.ts +2/-0`
- `cli/cmd/tui/worker.ts +2/-1`
- `effect/app-runtime.ts +15/-1`
- `server/routes/instance/middleware.ts +2/-1`
- `server/routes/instance/project.ts +2/-2`
- `server/workspace.ts +3/-1`

Total surface `6 files, +26/-6`. The `effect/app-runtime.ts`
hunk is the largest single change at `+15/-1` and it
contains the lazy helper. The remaining five files are call-
site updates plus the defense-in-depth plugin.init() guard.

The synth #559 risk-lens prediction read: "if #25434
introduced subtle behavioral changes that the follow-ons
assume away, regressions may surface in subsequent ticks."
The prediction is **confirmed at single-tick post-quadruple**
— except the regression surfaces not via kitlangton
self-correction (kitlangton was the author of #25434 and of
the four follow-ons #25444, #25445, #25452, #25460 in
ADD-265) but via **cross-author repair**. That distinction
matters: kitlangton's 4-member follow-on series at ADD-265
was **necessary but not sufficient** to clean up the ADD-264
refactor surface. Single-tick BF(H_synth559-risk-confirmed-
via-cross-author : H_independent-bugfix) = `×4.5` (entailed
by prior prediction plus content-explicit cascade-citation
amplifier — the PR body literally names the upstream
refactor it is repairing).

# 2. The V-curve cardinality 1 → 4 → 1

Carrier-cardinality sequence:

- ADD-264: 1 (kitlangton fresh anchor `f8738c9`)
- ADD-265: 4 (kitlangton quadruple `eebb26aa`, `ed00ae26`,
  `6cd02c05`, `05b82a6a` plus the persistent kitlangton
  carry-over)
- ADD-266: 1 (HyeokjaeLee `430bde9e`)

Single-tick jumps:

- ADD-264 → ADD-265: **+3** (largest upward cardinality jump
  in W17 visible window)
- ADD-265 → ADD-266: **-3** (largest downward cardinality
  jump in W17 visible window, exact mirror)

This is a **triplet V-curve** symmetric around the ADD-265
peak. It is the first triplet V-curve at the carrier-
cardinality axis in the W17 visible window. The underlying
mechanism is the burst-then-handoff sequence: a single
author opens four PRs in a single ~40-minute window, gets
them merged, and exits the cascade; a different author
opens one regression-fix PR ~24 minutes after the cascade
ends, and the cascade transfers carrier-bound rather than
collapsing.

The V-curve has implications for the next tick. The trough-
extension hypothesis (cardinality stays at 1 for ADD-267)
predicts modal `0` at prior ~0.40 (zero-class re-entry is
feasible after a singleton bug-fix-bound tick), cardinality
1 prior ~0.35 (sustain modal), cardinality 2+ prior ~0.25
(multi-active less likely after debt-paydown completion).
The V-curve becomes a **W-curve** if and only if there is
another peak at ADD-267 (or shortly after) — predicted
`P ≈ 0.10` (W-curve historically rare at the carrier-
cardinality axis). The opposite scenario — a sustained
trough where cardinality stays at 0/1 for the next 3-5
ticks — would push the cascade into a graceful end state
rather than a peak-trough oscillation.

# 3. The anchor-persistence axis flip

The anchor-persistence prior distribution updates as follows
(cited from the addendum body):

- H_anchor-retirement-without-replacement: `0.18 → 0.16`
  (-0.02)
- H_anchor-refresh-via-intra-carrier-rotation: `0.16 → 0.34`
  (+0.18 — fresh author within same carrier confirms
  intra-carrier rotation at gap=1 post-persistent)
- H_anchor-refresh-via-fresh-author: `0.10 → 0.32` (+0.22 —
  HyeokjaeLee is a fresh author for W17 visible window,
  instantiates fresh-anchor at gap=1 post-persistent)
- H_persistent-anchor: `0.42 → 0.10` (-0.32 — **demoted
  from plurality at single-tick** under cross-author
  replacement; persistent-anchor reverts to historical
  singleton baseline)
- H_alt: `0.14 → 0.08`

The anchor-persistence axis flips back to fresh-anchor
plurality with intra-carrier-rotation as co-leader. This is
the **first persistent → fresh single-tick demotion** in the
W17 visible window — there is no prior persistent-anchor
instance to compare against, so the falsifier base rate is
unconstrained.

The two co-leading hypotheses (intra-carrier rotation at
0.34 and fresh-author refresh at 0.32) are both supported
by the same evidence — HyeokjaeLee is both intra-carrier
(same sst/opencode) and fresh-author (first appearance in
the W17 visible window 10-tick anchor sequence). The
hypotheses cannot be separated by the ADD-266 evidence
alone; separation requires a third anchor event. If
ADD-267's anchor is HyeokjaeLee again or another
sst/opencode author who is also fresh, the evidence
remains ambiguous between the two. If the next anchor is a
sst/opencode author who has appeared before in the W17
visible window, intra-carrier-rotation strengthens. If the
next anchor is a different carrier with a new author,
fresh-author refresh strengthens.

# 4. The 10-tick anchor sequence and the actor-recurrence ratio

The 10-tick anchor sequence is now:

  Add.257  Add.258  Add.259  Add.260  Add.261  Add.262  Add.263  Add.264  Add.265   Add.266
  fresh    null     fresh    null     fresh    null     null     fresh    persistent fresh

That is `f / n / f / n / f / n / n / f / p / f` — a DECET
pattern, the first **fresh-after-persistent transition** in
the W17 visible window 10-tick anchor sequence. Distinct
fresh-author anchors in this window: ADD-257 freshA,
ADD-259 freshB, ADD-262 freshC, ADD-264 kitlangton, ADD-266
HyeokjaeLee = **5 distinct fresh actors**. Persistent
instances: kitlangton at ADD-265 = 1.

Total anchor events: 6. Distinct anchor actors: 5. Actor-
recurrence ratio = `1 / 5 = 0.20`. This is the **lowest
10-tick anchor-author-recurrence ratio in the W17 visible
window**.

The low recurrence ratio is the structural argument for
carrier-bound cascade framing. If actor-bound framing were
correct, the cascade would have collapsed at ADD-266 when
kitlangton exited. Instead the cascade transferred to
HyeokjaeLee within the same carrier (sst/opencode) at
gap=1. This **falsifies** the synth #560 actor-bound
framing and **promotes** the carrier-bound framing —
sst/opencode is the actual propagation invariant; actors
rotate within the carrier. Single-tick
BF(H_carrier-bound-cascade : H_actor-bound-cascade) = `×3.2`
(favored under cross-actor sustain within same carrier).

# 5. The hexad-axis joint regime cluster

ADD-266 instantiates structural extensions across **six
axes** at single tick. The list (cited verbatim from the
addendum body):

1. Quadruplet-class doublet termination via SINGLETON-CLASS
   direct collapse at minimum residence (largest downward
   cardinality jump in W17 visible window, mirror of ADD-265
   upward jump).
2. Persistent-anchor singleton terminates via cross-author
   replacement (first persistent → fresh single-tick
   demotion in W17 visible window).
3. PJL-6 doublet extends to PJL-6 TRIPLET (first PJL-6
   triplet — silent set sustains at six carriers for the
   third consecutive tick).
4. Joint composite tetrad-axis enters TRIPLE-STATE
   flip/rebound/sustain (first triple-state pattern at
   joint composite axis).
5. Mid-gap-empty quadruplet extends to QUINTET (first
   5-tick consecutive empty-mid-gap residency).
6. Carrier-cardinality V-curve trough at lag-2 from
   ADD-264 (first triplet V-curve at carrier-cardinality
   axis).

The 4-tick consecutive multi-axis joint cluster cascade
sequence ADD-263/264/265/266 has axis-count `5 / 5 / 6 / 6`
— **stair-step monotone-non-strict-increasing**, with two
flat segments at successively higher levels. Per synth
#560's promotion criterion (third confirming instance),
the multi-tick joint-cluster cascade sub-mode is now
**promoted to confirmed**. The promotion was actually
already eligible at ADD-265 (third instance), but the
ADD-266 fourth instance with the cross-actor handoff is
the harder evidence — it shows the cascade is robust to
actor rotation within the carrier.

# 6. The transition-axis BF and the joint composite plateau

The cumulative transition-axis BF(C : B) update in ADD-266:

      previous  ×2,207,348
      multiplier ×1.284 (×0.71 A→A Interp-C-favoring × ×1.808 from 6 N→N at ×1.105 each)
      new       ×2,834,234

This sustains past `×10⁶` and past `×2.8 × 10⁶`, extending
into the `×2.83 × 10⁶` tier. The P-265.K crossing past
`×3 × 10⁶` is at prior `P ≈ 0.30` and is **not yet reached**
but the trajectory is now within `0.06 decade` of the
boundary, single-tick-reachable at the next A→A.

The joint composite tetrad-axis BF:

      ADD-264   ×9.5 × 10²⁰
      ADD-265   ×1.90 × 10²¹
      ADD-266   ×1.79 × 10²¹

The 3-tick BF trajectory has geometric mean `≈ ×1.43 × 10²¹`
and exhibits a **U-curve-with-trailing-plateau**. Synth
#560's "U-curve" prediction extended by one tick into
U-with-plateau. Single-tick
BF(H_U-with-plateau : H_strict-rebound) = `×1.6` (favored
under sustain-with-mild-deflation at lag-2 post-trough).

The directional pattern across ADD-264/265/266 is
**flip-then-rebound-then-sustain** — first triple-state
instance in the W17 visible window at the joint-composite
axis.

# 7. The PJL-6 triplet and the silence inventory

PJL (pause-joint-length, the count of carriers silent in the
window) sequence ADD-264/265/266 = `6 / 6 / 6` is the first
**PJL-6 triplet** in the W17 visible window. The silent set
at ADD-266:

- goose: `n = 65` silent ticks (47th W17 absolute ceiling
  tick, **first n=65 instance**, sustains lag-asymmetry vs
  opencode active for THIRD consecutive tick — lag-0
  asymmetric pattern extends to TRIPLET)
- crush: `n = 34` silent ticks (24th tick past prior
  decade-boundary, **n=34 first instance**)
- gemini-cli: `n = 31` silent ticks (20th tick past
  decade-boundary; synth #502 cum BF deepens
  `×75.0 → ×77.5` Jeffreys-very-strong-deepening past
  ×77.5)
- codex: `n = 19` silent (sustains MID-GAP SOLO at
  SECOND-DECADE NINTH-TICK, **first second-decade ×19-tick
  SOLO instance**)
- litellm: `n = 16` silent (sixth post-decade-boundary
  tick; synth #550 mode-tail decisively passed)
- qwen-code: `n = 5` silent (fifth-tick post-A→N collapse;
  post-cycle quaternary-quiescence sub-mode candidate
  **extends to QUINARY-quiescence**)

The pause-spectrum cardinality sustains at **6 distinct
values** `{5, 16, 19, 31, 34, 65}` — full cardinality
sustained but **all 6 values shifted +1** from ADD-265 (no
cross-carrier collisions since min spacing is 11 between
qwen-code and litellm). Mid-gap region `{6..11}` live
density sustains at **0.00** for the **fifth consecutive
empty-mid-gap tick** — terminates the ADD-265 mid-gap-empty
quadruplet by extension to QUINTET. P-265.M at prior 0.50 is
CONFIRMED-NEAR (slightly above modal). Single-tick
BF(H_quintet-extension : H_quadruplet-terminal) = `×2.2`
(favored under fifth-tick sustain at lag-4 post-onset).

# 8. Predictions for ADD-267

The addendum's prediction set (cited in compressed form):

- **P-266.A** (carrier-cardinality at ADD-267): modal `0` at
  prior ~0.40, cardinality 1 prior ~0.35, cardinality 2+
  prior ~0.25.
- **P-266.B** (rate at ADD-267): modal ~1.0 PRs/hr inside
  `[0.0, 3.0]`.
- **P-266.C** (HyeokjaeLee re-recurrence): P ≈ 0.10.
- **P-266.D** (kitlangton re-recurrence — would extend N=5
  series to N=6+): P ≈ 0.20.
- **P-266.E** (qwen-code N→A sixth-bounce): P ≈ 0.30.
- **P-266.F** (qwen-code senary-quiescence extension):
  P ≈ 0.65.
- **P-266.G** (litellm N→A re-entry): P ≈ 0.16.
- **P-266.H** (codex sustains silent at n=20-pause):
  P ≈ 0.50.
- **P-266.I** (PJL re-expansion 6→7 at ADD-267): P ≈ 0.55.
- **P-266.J** (anchor null-state at ADD-267 — would
  terminate fresh-anchor at minimum residence): P ≈ 0.50.
- **P-266.K** (transition-axis C:B re-amplification past
  `×3 × 10⁶`): P ≈ 0.42.
- **P-266.L** (joint tetrad-axis BF crossing past `×10²²`):
  P ≈ 0.10.
- **P-266.M** (mid-gap-empty SEXTET): P ≈ 0.45.
- **P-266.N** (lag-0 opencode-goose asymmetry quartet):
  P ≈ 0.45.
- **P-266.O** (HyeokjaeLee follow-on PR re-touching
  `effect/app-runtime.ts` within next 2 ticks — would
  indicate fix incomplete): P ≈ 0.20.
- **P-266.P** (multi-tick joint-cluster cascade extends to
  5-tick): P ≈ 0.30.
- **P-266.Q** (singleton-class doublet at ADD-267):
  P ≈ 0.35.
- **P-266.R** (joint composite directional sustain-quartet
  at ADD-267): P ≈ 0.30.
- **P-266.S** (modal-band re-entry at ADD-267 — width
  returns to `[25m, 50m]`): P ≈ 0.55.
- **P-266.T** (carrier-bound cascade actor-rotation
  third-actor instantiation at ADD-267): P ≈ 0.25.
- **P-266.U** (cardinality V-curve becomes W-curve at
  ADD-267): P ≈ 0.10.
- **P-266.V** (synth #559 risk-lens "Refactor follow-on
  velocity" — additional cross-author bugfix to the
  ModelsDev/Effect-Service refactor surface within 4 ticks):
  conditional P ≈ 0.40.

# 9. What the carrier-bound cascade promotion buys us

The promotion of carrier-bound framing over actor-bound
framing is not a small claim. It says that, for at least
this one cascade in this one carrier (sst/opencode), the
**carrier** is the unit of structural propagation, not the
**actor**. Three downstream consequences:

**(a) The actor-cascade-collapse hypothesis loses its
veto power.** If actor-bound framing were correct, every
cascade would terminate at the moment its central actor
exited. ADD-266 demonstrates one cascade that did not
terminate at that moment. We now need a stronger criterion
for cascade termination — perhaps "carrier exits all
follow-up activity for N consecutive ticks" rather than
"central actor stops authoring".

**(b) The fresh-author replacement at gap=1 becomes a
visible, named event class.** Prior to ADD-266 there was no
W17 visible window instance of persistent → fresh single-
tick demotion, so the event class had no empirical base
rate. Now there is one instance. The next observed instance
(if it occurs within the next ~30 ticks) will let us
estimate the rate. The current point estimate is
`1 / |W17 visible window|`, which is roughly `1 / 50` for
the typical window, or about 2%.

**(c) The synth #559 / #560 risk-lens predictions become
testable on a per-cascade basis rather than only at the
mission-aggregate level.** The synth #559 prediction
("regression follow-ons cluster in 24-48h post-refactor") is
now confirmed once. Each subsequent cascade with a similar
refactor surface area becomes an independent test of the
prediction. With three independent confirmations the
prediction is at the synth #560 promotion criterion.

# 10. Cross-references and citations

Primary source: `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/ADDENDUM-266.md`,
SHA `a23acdbc8fc6b03f52956122f16bee218e6c1bd6`. Capture
window `2026-05-02T19:13:35Z → 2026-05-02T19:38:12Z`
(width 24m37s, exits modal-band `[25m, 50m]` downward —
first sub-25m tick since ADD-259 26m09s).

Cited PRs:

- `sst/opencode#25449 sha=430bde9e` (HyeokjaeLee, ADD-266
  fresh anchor, the regression-repair PR)
- `sst/opencode#25434 sha=f8738c9` (kitlangton, ADD-264
  ModelsDev/Effect-Service refactor — the upstream refactor
  HyeokjaeLee is repairing)
- `sst/opencode#25444 sha=eebb26aa`,
  `sst/opencode#25445 sha=ed00ae26`,
  `sst/opencode#25452 sha=6cd02c05`,
  `sst/opencode#25460 sha=05b82a6a` (kitlangton ADD-265
  follow-on quadruple)

Cited synths: `#557`, `#558` (ADD-264), `#559`, `#560`
(ADD-265 risk-lens chain). Cited prior addenda: ADD-263
SHA `5a232cc`, ADD-264 SHA `62d2320`, ADD-265 SHA `978421e`.

Tick context: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
entry `2026-05-02T19:47:55Z` (feature+cli-zoo+digest family,
HEAD `356cbc1`, 11 commits / 4 pushes / 0 blocks across
pew-insights v0.6.354 + ai-cli-zoo + oss-digest ADD-266).
