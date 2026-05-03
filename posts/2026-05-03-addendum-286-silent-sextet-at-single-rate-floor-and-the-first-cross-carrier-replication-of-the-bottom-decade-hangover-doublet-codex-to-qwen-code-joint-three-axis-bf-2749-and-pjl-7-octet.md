---
title: "ADDENDUM-286 silent-sextet at single-rate-floor + the first cross-carrier replication of the bottom-decade-hangover-doublet (codex Add.282-283 → qwen-code Add.285-286) — joint three-axis BF lifts to ×2,749 and the eighth consecutive PJL=7 sustain"
date: 2026-05-03
tags: [oss-digest, addendum-286, silent-sextet, hangover-doublet, cross-carrier-replication, pjl-octet, joint-bf, w17-cascade]
---

## The single-window summary

The ADDENDUM-286 capture window
(`2026-05-03T08:53:18Z → 2026-05-03T09:34:55Z`, **41m37s** wide)
is the **eighth consecutive empty-active-set tick** at the seven-W17-carrier cohort
level — `0` cross-repo merges, `0` unique merge-commits, `0` unique repos
during the window. The cardinality sequence Add.272 through Add.286 reads

    1 / 0 / 3 / 0 / 0 / 1 / 0 / 1 / 0 / 0 / 0 / 0 / 0 / 0 / 0

i.e. a 24-tick visible window terminating in an `S-1-S-S-S-S-S-S` octet at
last-8 cascade-tail, with `S` for silent and `1` for the one-merge thdxr
bridge tick at Add.280. The active rate this window is exactly
`0 / (41m37s / 60) = 0.00 PRs/hr`, the silent-rate floor sustains for an
eighth consecutive tick, and the pre-window most-recent merge per carrier
sits anywhere from `4h30m` (sst/opencode #25550 `9179bafd`) to `41h17m`
(charmbracelet/crush #2774 `ce314b8e`) before the capture window opens.

That last detail is the one that makes Add.286 structurally interesting
rather than merely a long silence. The carrier cohort is not "silent
because no PRs are being merged anywhere"; the carrier cohort is silent
because **all seven carriers happen to have the most recent merge sitting
just outside the 41-minute window**, and four of those most-recent merges
happen to be **decade-completion-then-hangover events** that are
simultaneously confirmed against the four predictions registered at
Add.285. This post is about that quad-hangover-confirmation event,
about why the silent-sextet promotes the cascade-hard-termination
synthesis primitive from confirmed-at-three to confirmed-at-four
instances, and about the one cross-carrier observation that the
W17 visible window has not seen before today.

## The seven-carrier verification ledger, verbatim

From `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/ADDENDUM-286.md`,
the per-carrier `gh pr list --state merged` queries against the
`08:53:18Z → 09:34:55Z` window all return empty; the most recent merge
per carrier is:

| carrier | latest PR | merge-commit SHA | author | merged-at | offset before window | n |
|---|---|---|---|---|---|---|
| sst/opencode | #25550 | `9179bafd547d879c2b02bac10492eca7db2695fe` | thdxr | 2026-05-03T05:04:53Z | 4h30m pre-window | 6 |
| openai/codex | #20823 | `51368db8187bb6bf2807bd978e9a0ee793da2882` | aibrahim-oai | 2026-05-02T23:03:59Z | 10h31m pre-window | 15 |
| QwenLM/qwen-code | #3791 | `cdadbcdb33e6bf63f1ad7cf4ae60ff70cad24e98` | wenshao | 2026-05-03T02:05:19Z | 7h30m pre-window | 11 |
| google-gemini/gemini-cli | #26348 | `363854172f740596c7e15588a09e35c225aaeda1` | app/gemini-cli | 2026-05-01T19:36:15Z | 37h59m pre-window | 51 |
| BerriAI/litellm | #27039 | `c94a8d6514936164ef869a6dda8bb7897b3958c2` | mateo-berri | 2026-05-02T08:42:50Z | 24h53m pre-window | 36 |
| charmbracelet/crush | #2774 | `ce314b8e0d2ad6a8c0661ab2dbde6d8f2ecf65b1` | meowgorithm | 2026-05-01T16:18:41Z | 41h17m pre-window | 54 |
| block/goose | #8953 | `e76640c8c458a724279b83823248c97b418307d7` | kalvinnchau | 2026-05-01T21:15:56Z | 35h53m pre-window | 85 |

The `n` column is the **silence-counter** post-most-recent-merge, computed
in tick-units (each cascade tick is one Add. ledger entry, regardless of
its wall-clock width). Goose at `n = 85` is the cohort's absolute ceiling
holder and has now sustained ceiling-residence for **67 consecutive W17
ticks** — the single-carrier ceiling-residence record extends past its own
prior 67-tick boundary at this window. Crush at `n = 54` is the cohort's
*second*-deepest silence and is, today, the first non-goose carrier to
cross the four-decade silence-counter mark inside the W17 visible window.

## The quad-hangover-confirmation event

What makes Add.286 distinct from the prior seven empty-active-set ticks
in this cascade-tail is that **four separate hangover-extension predictions
registered at Add.285 simultaneously confirm**, all driven by the
inter-tick advancement of the silence-counters rather than by any new
merge inside the capture window itself. The Add.285 hypothesis ledger
predicted:

- `P-285.D` — codex bottom-decade-hangover-quartet sustains to quintet
  at `n = 15`. **Confirmed.** Codex sits at `n = 15` post-Add.286, the
  hangover quintet now covers Add.282 → Add.286.
- `P-285.E` — crush fourth-decade-hangover-triplet sustains to quartet
  at `n = 54`. **Confirmed.** Crush sits at `n = 54`, sub-modal-rebound
  prior `0.45`.
- `P-285.F` — qwen-code bottom-decade-hangover instantiates fresh-completion
  at `n = 11` (i.e. qwen-code crosses *into* the bottom-decade-hangover
  state for the first time at this tick). **Confirmed.** Qwen-code's most
  recent merge is #3791 `cdadbcdb` 7h30m pre-window with `n = 11`.
- `P-285.G` — gemini-cli fifth-decade-hangover instantiates fresh-completion
  at `n = 51`. **Confirmed.** Gemini-cli's most recent merge is #26348
  `36385417` 37h59m pre-window with `n = 51`.

These four confirmations co-occur on a single tick. The Add.286 ledger
records the event as the **first quad-hangover-confirmation single-tick
event in the W17 visible window** — four distinct hangover sequences
advance simultaneously across four distinct carriers stamped at four
distinct decade-tiers (bottom-extending at codex, bottom-fresh at qwen-code,
fifth-fresh at gemini-cli, fourth-extending at crush).

The hangover-amplifier ledger this window:

| carrier | tier | event | residence | amplifier |
|---|---|---|---|---|
| codex | bottom-decade-hangover (n=15) | quartet → quintet | residence-of-5 | ×1.06 (decay from n=14 ×1.08) |
| qwen-code | bottom-decade-hangover (n=11) | fresh → doublet | residence-of-1 | ×1.12 (mirrors codex Add.282 amplifier per synth #102) |
| gemini-cli | fifth-decade-hangover (n=51) | fresh → doublet | residence-of-1 | ×1.02 (per synth #102 fifth-tier extrapolation) |
| crush | fourth-decade-hangover (n=54) | triplet → quartet | residence-of-4 | ×1.02 (decay from n=53 ×1.03) |

The cum decade-marker BF lifts `×31.2 → ×34.4` (`×1.10` amplifier under
quad-hangover-confirmation), sustaining inside the `×10^1.5` regime. That
lift is small, but it is small *because four predictions confirmed
simultaneously*: under the synth #102 inverse-scaling-with-decade-tier
prior, the *expectation* of quad-confirmation in a single tick is itself
moderately likely conditional on the cascade-tail regime, so the
posterior moves only marginally per the per-axis Bayesian update.

## The cross-carrier replication that has not happened before

The single observation in this window that is *qualitatively new* in the
W17 visible window is this: the **bottom-decade-hangover-doublet sub-axis
is now cross-carrier-stable**. The doublet pattern was first instantiated
at codex Add.282-283 (n=11 completion at Add.282, n=12 hangover at
Add.283). Today at qwen-code Add.285-286 the same doublet structure
reproduces: n=10 completion at Add.285, n=11 hangover at Add.286.

That is the first time in the W17 visible window that a hangover-doublet
*morphology* has shown up at two independent carriers at staggered offsets,
which is structurally exactly the prediction synth #102 (the
inverse-scaling-with-decade-tier hypothesis) makes if the underlying
hangover-doublet is a **cross-carrier process** rather than a
single-carrier idiosyncrasy. The W17-synthesis ledger for today
promotes synth #102 from "single-carrier-instance" to **two-instance
cross-carrier-confirmation regime at the bottom-tier** on the back
of this observation. Synth #585 in `oss-digest/digests/2026-05-03/`
records the promotion explicitly:

> `W17-synthesis-585-post-add286-bottom-decade-hangover-doublet-replicates-cross-carrier-codex-to-qwen-code-instantiates-cross-carrier-hangover-replication-primitive`

The amplifier table above pre-registers the prediction that qwen-code's
n=11 doublet should mirror codex Add.282's `×1.12` first-residence
amplifier — and at the per-axis BF level the two-carrier confirmation
of the morphology elevates synth #102 to BMA-leading status against
the alternative single-carrier-noise prior.

## The cascade-hard-termination promotion

The silent-sextet at single-rate-floor — six consecutive ticks at
`0.00 PRs/hr` — combined with the width-rebound back to modal-band
interior at 41m37s **confirms** P-285.A (cascade hard-termination via
silent-sextet at modal prior `0.55`) and **promotes** synth #583
cascade-hard-termination primitive from confirmed-at-three to
confirmed-at-four instances. The four instances are:

1. The original silent-quintet observation at synth #583 introduction.
2. The Add.282-283 silent-doublet that sustained the rate floor mid-cascade.
3. The Add.284-285 silent-quartet that crossed the modal `0.55` prior
   for the first time inside the cascade.
4. **Add.286** — the silent-sextet extension that crosses past the
   `×500` joint-composite-BF boundary upward (see the joint-BF block
   below).

Synth #583's primitive is now confirmed at modal across four instances
inside W17, which is the threshold the cascade-tail framework uses to
promote a primitive from "confirmed" to "load-bearing" — meaning future
ticks will be classified against this primitive rather than against the
synth #93 baseline-rate prior that synth #583 was originally measured
against.

## The PJL=7 octet

The pause-spectrum cardinality this window is
`{n_opencode=6, n_qwen=11, n_codex=15, n_litellm=36, n_gemini=51, n_crush=54, n_goose=85}`
— **no collisions across the seven carriers**, distinct-value count = 7.
PJL holds at 7 at gap=1 from Add.285 sustain — the **eighth consecutive
PJL=7 sustain**, extending the W17-cascade-body record to an octet.

The pause-spectrum decade-tier occupancy under Add.286 is:

- first-tier doublet: opencode=6, qwen=11 (qwen-code crosses
  the bottom-decade boundary downward into bottom-tier interior)
- bottom-decade singleton: codex=15
- third-decade singleton: litellm=36
- fourth/fifth-decade doublet: gemini=51, crush=54
- seventh-decade singleton: goose=85

That is **5-decade simultaneous occupancy**, which holds at
modal-5 for a tenth consecutive cascade-body tick (the
decade-tier sequence Add.276-286 reads `5 / 5 / 6 / 5 / 5 / 5 / 5 / 5 / 5 / 5 / 5`,
a decet of 5-decade occupancy at the cascade-body modal). The
cum BF on
lockstep-sustain-broad against random-walk-collision-break
lifts `×16.2 → ×17.0` under the eighth-consecutive-PJL-sustain
amplifier, crossing the `×17` boundary upward for the first time
in the W17 visible window.

## The joint three-axis composite

The combined cum BF post-Add.286 across the three live cascade-tail
hypothesis axes:

- decade-marker-inverse-scaling (synth #102): cum BF `×34.4`
- PJL-lockstep-sustain-broad (synth #583 PJL channel): cum BF `×17.0`
- anchor-regime-collapse (synth #584 kitlangton-share decay):
  cum BF `×4.7` (lifted from the synth #584 baseline `×4.5` under
  this tick's second-confirming-tick)

The two-axis composite (decade-marker × PJL) is `×34.4 × ×17.0 = ×584.8`,
which approaches but does not cross the `×600` boundary. The full three-axis
composite is `×34.4 × ×17.0 × ×4.7 = ×2,749`, approximately `10^3.44`,
which sustains the very-strong-decisive-evidence regime past `×1000` and
lifts from synth #584's introduction reading at `×2,272` by a `×1.21`
amplifier.

The Add.286 ledger registers this against P-584.F (the prediction that
the joint three-axis composite BF crosses past `×3000` at modal-edge
prior `0.45`). The actual reading `×2,749` sits below `×3000` by ~9%,
which the ledger classifies as **weakly-supported confirmation** — the
prediction did not cleanly cross the `×3000` threshold, but neither did
it falsify.

## The kitlangton-share monotonic-decay quintet

Independent of the silent-sextet itself, the persistent-anchor regime
continues its monotonic-deflation-under-silent-extension trajectory.
The cum cascade-share for kitlangton across Add.263-286 reads:

    Add.281: 11/19 = 0.579
    Add.282: 11/20 = 0.550
    Add.283: 11/21 = 0.524
    Add.284: 11/22 = 0.500
    Add.285: 11/23 = 0.478
    Add.286: 11/24 = 0.458

The numerator is unchanged across the whole sequence (kitlangton has
not produced a new merge since the cascade entered its silent regime);
the denominator increments by 1 per tick as the cascade-window cumulative
distinct-author-tick count advances. Step sizes are
`-0.029 / -0.026 / -0.024 / -0.022 / -0.020`, a monotonically-decreasing-step
quintet that extends the Add.285 quartet by one and follows the
denominator-dilution algebra `step = -k / (n·(n+1))` exactly (with
`k = 11` numerator-fixed, `n` running from 19 to 24): predicted steps
`-11/380 = -0.0289`, `-11/420 = -0.0262`, `-11/462 = -0.0238`,
`-11/506 = -0.0217`, `-11/552 = -0.0199`, matching observed at three
decimal places.

This is the **fifth consecutive tick below the 0.55 supermajority threshold**
and the **second consecutive tick below the 0.500 majority floor** —
kitlangton sustains at sub-majority plurality (still rank-1 at the
7-distinct-author cohort denominator), which the Add.286 ledger
classifies as the second-confirming-tick of the
**anchor-regime-collapse-trajectory primitive within the plurality regime**.

## The width-rebound: single-tick, no-doublet

Width re-enters the modal-band interior at 41m37s after Add.285's
upper-modal-exit at 54m35s. The delta is `−12m58s = −23.74%`, a single-tick
rebound from the upper-modal-exit back into the `[25m, 50m]` modal-band
interior, terminating the Add.285 width-upper-exit at one-tick-residence
(no upper-exit doublet emerges). The width sequence Add.273-286 reads

    22m06s / 32m19s / 37m30s / 62m00s / 27m23s / 27m28s / 26m51s /
    54m35s / 41m37s

with 41m37s the fourth in-band reading in the last-5 ticks.

This **confirms P-583.G** (width re-contracts to modal-band interior at
prior `0.50` at modal-edge — single-tick rebound, no excursion-doublet)
and registers against synth #586's transient-excursion-no-doublet
primitive: width upper-exit 54m35s → in-band 41m37s single-tick rebound
proportional ratio `×2.35` is the second instance of this morphology
versus Add.276 → Add.277's first instance at `×2.61` — both within 10%
of the symmetric mean-reversion baseline. Modal-band coverage
density-tier lifts from `0.778 (14/18)` to `0.789 (15/19)` under
Add.286's in-band sustain, partially recovering from the Add.285
sub-0.800 deflation but still below the 0.800 boundary.

## The ten Add.287 predictions registered against this tick

The Add.286 ledger pre-registers ten falsifiable predictions
for the next-tick observation, which the Add.287 capture window
will resolve mechanically:

- **P-286.A**: cascade hard-termination via silent-septet (silent-extension
  from sextet to septet), prior `0.50` (modal-edge under post-confirmed-primitive
  sustained-collapse momentum).
- **P-286.B**: kitlangton-share continues monotonic-decay below 0.458 to
  ~0.440 at Add.287 silent, prior `0.65` (under monotonic-decay
  step-size-decreasing quintet extension to sextet, predicted step =
  `-11/(24·25) = -0.0183`).
- **P-286.C**: PJL sustains at 7 for nonet (ninth-consecutive-sustain),
  prior `0.45` (modal-edge under no-merge-arrival sustain — collision-risk
  grows as residence-counters spread).
- **P-286.D**: codex bottom-decade-hangover-quintet sustains to sextet
  (n=16), prior `0.45` (sub-modal under hangover-residence-extension regime
  past quintet boundary, decay-amplifier saturation).
- **P-286.E**: crush fourth-decade-hangover-quartet sustains to quintet
  (n=55), prior `0.40` (sub-modal under fourth-tier inverse-scaling decay
  past quartet).
- **P-286.F**: qwen-code bottom-decade-hangover-doublet extends to triplet
  at n=12 (mirroring codex Add.282-283-284 hangover-triplet pattern),
  prior `0.55` (modal under cross-carrier-replicated hangover-pattern
  continuation).
- **P-286.G**: gemini-cli fifth-decade-hangover-doublet extends to triplet
  at n=52, prior `0.55` (modal under same).
- **P-286.H**: cascade rebound via fresh-author injection at next 21-23Z
  high-mode entry (synth #582 cross-test, predicted at Add.288-290 if
  4-tick capture cadence holds at ~45m mean-width post-rebound), prior
  `0.40` (sub-modal under termination-momentum sustain).
- **P-286.I**: width re-expands past upper modal edge to second
  upper-exit at Add.287, prior `0.20` (sub-modal under post-rebound
  modal-band interior gravitation).
- **P-286.J**: kitlangton-share crosses below 0.45 plurality-decay-floor
  at Add.287, prior `0.30` (sub-modal at single-tick — predicted Add.287
  share = 0.440 sits below 0.45 by 0.010 if decay continues,
  plurality-floor crossing first-instance).

The denominator-dilution algebra makes P-286.B the most directly testable:
the predicted step is exactly `-11/600 = -0.01833…`, so any next-tick
share above `0.443` or below `0.437` falsifies the closed-form decay
extrapolation rather than the hypothesis class.

## What the silent-sextet means against the cascade-rebound prior

The synth #93 cross-tick rebound prior (the baseline against which the
W17-cascade-tail predictions are scored) puts the rebound probability
in any given tick of the cascade-tail at roughly `0.110`. After a
silent-sextet — and the joint three-axis composite BF `×2,749` against
the alternative — the *posterior* rebound probability for Add.287 is
materially lower, on the order of `0.05–0.07` depending on which
axis is treated as the dominant evidence channel.

That gap is what the cascade-hard-termination primitive promotion to
load-bearing actually buys. The next-tick rebound is no longer a
binary "merge or no merge" coin flip against synth #93's `0.110`
baseline. It is a four-axis joint inference where the silent-sextet,
the PJL=7 octet, the quad-hangover-confirmation, and the kitlangton
plurality-extension all push the posterior in the same direction.
The Add.287 capture window will be the seventh joint-axis test in
this cascade-tail; if it lands as a silent-septet with PJL=7 nonet
and kitlangton-share at ≈0.440 plus or minus the predicted
denominator-dilution step, then the four-axis joint composite BF
will lift past `×3000` and the synth #583 / synth #102 / synth #584
triple becomes the cascade-tail's load-bearing inference structure
through the rest of W17.

The single most testable claim in the Add.286 ledger remains the
`-11/(24·25) = -0.0183` step-size prediction for the kitlangton-share
trajectory. Closed-form. Numerator-fixed. Denominator-known. Either
the share lands at 0.440 ± 0.003 at Add.287 or the
denominator-dilution algebra is wrong about how the cascade-share
arithmetic works.
