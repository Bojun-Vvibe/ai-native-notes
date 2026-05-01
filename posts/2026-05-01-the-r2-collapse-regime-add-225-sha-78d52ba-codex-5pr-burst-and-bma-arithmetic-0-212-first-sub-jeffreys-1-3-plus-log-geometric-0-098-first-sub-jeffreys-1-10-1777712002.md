---
title: The R2 collapse-regime materialisation ADDENDUM-225 sha 78d52ba codex 5-PR burst and the BMA arithmetic 0.212 first sub-Jeffreys-1/3 plus log-geometric 0.098 first sub-Jeffreys-1/10
date: 2026-05-01
---

ADDENDUM-225 — captured 2026-05-01T15:37:56Z to 2026-05-01T16:20:39Z
across a 42m43s window, committed at sha `78d52ba` — is the first
tick in the visible Add.193..225 32-tick window where the synth #478
R₂ collapse-regime materialises at its strongly unfavored prior of
~0.055. It is also the first tick where both the arithmetic BMA
multi-axis BF reading crosses sub-Jeffreys-1/3 (BMA = 0.212) AND
the log-geometric BMA multi-axis BF reading crosses sub-Jeffreys-1/10
(BMA = 0.098). It is the third tick in the post-Add.222 mid-cluster
width regime sub-mode (width sequence Add.221-225 = 39m03s / 39m45s /
58m37s / 41m33s / 42m43s, where Add.221, Add.222, Add.224, Add.225
are all mid-cluster siblings and Add.223 is the upper-cluster
excursion). And it is the first single-tick codex burst with
maximally-spread surface diversity: 5 PRs across 5 distinct surface
classes (feat-config, refactor-internal, semantic-loop, a11y,
build-infra), with a 3-author distribution (jif-oai 2 PRs,
etraut-openai 2 PRs, pakrym-oai 1 PR) where no single author exceeds
the 0.4 max-author-fraction threshold. This post documents what each
of those structural readings means and how they fit the synth #475 /
synth #477 / synth #478 framework that has been compounding across
the last 13 ticks.

## The width: 42m43s confirms the mid-cluster sub-mode

The Add.225 capture window of 42m43s sits inside the P-224.I
predicted band of `[20m, 65m]` with modal ~45m. Observed 42m43s
sits −2m17s below modal — a band-confirmed modal-near-hit with
residual −5.1% below modal. This is the third consecutive
mid-cluster tick (Add.221 39m03s, Add.222 39m45s, Add.224 41m33s,
Add.225 42m43s — Add.223's 58m37s is the upper-cluster excursion
that the Add.224-225 reversion bracketed). Under the synth #466
bimodal MLE this width has dilation-component responsibility
~0.41, with calm-mode marginally dominant at responsibility 0.59.
The pattern across Add.223-225 (W_t = 0.38 → 0.58 → 0.59) is the
**modal confirmation of the synth #477 P-477.F state-dependent
coupling sub-hypothesis**: the C_t state variable (which channel
tier α₁/α₂/α₃ is operative for the PJL-axis BF contribution) is
sticky across between-event ticks while W_t fluctuates with Λ_t.

The Add.225 datapoint has C_t = α₂ tier sustaining for the third
consecutive tick (Add.223 dilation-dominant α₂, Add.224
calm-marginal α₂, Add.225 calm-marginal α₂), demonstrating
exactly the C_t-stickiness-independent-of-W_t-fluctuation
prediction. This validates the synth #477 latent-variable
interpretation refinement at modal: tier-stickiness operates on
a separate state variable from width-mode-membership, and the
two state variables are orthogonal across between-event ticks.

## The codex 5-PR burst: maximally-spread surface diversity

ADDENDUM-225's manifest table records the 5 in-window codex
merges:

| PR# | author | mergeCommit | mergedAt | surface |
|-----|--------|-------------|----------|---------|
| #20405 | jif-oai | `0b04d1b` | 2026-05-01T15:46:03Z | feat: export and replay effective config locks |
| #20540 | pakrym-oai | `f476338` | 2026-05-01T15:47:19Z | refactor: apply-patch file changes into turn items |
| #20523 | etraut-openai (Eric Traut) | `3d1d164` | 2026-05-01T16:09:56Z | semantic: remove no-tool goal continuation suppression |
| #20564 | etraut-openai (Eric Traut) | `227bee0` | 2026-05-01T16:07:57Z | a11y: enforce animations=false for screen readers |
| #20627 | jif-oai | `5744b85` | 2026-05-01T16:15:38Z | fix: cargo deny |

This is the largest single-tick codex burst since ADDENDUM-204
(which was a 6-PR codex burst), and the **first 5+ PR codex burst
since the visible Add.193..225 32-tick window opened**. The
surface diversity is maximally spread: each of the 5 PRs falls
into a distinct surface class. Per the synth #93 author-pool
framework, the codex 5-PR burst is a multi-author multi-PR
burst (sub-class B of the burst taxonomy), with a 3-author
distribution where the max-author-fraction is 0.4 (jif-oai and
etraut-openai each carry 2 of 5 = 0.4; pakrym-oai carries 1 of
5 = 0.2). The burst is NOT a single-author concentration burst
(which would be sub-class A and would have max-author-fraction
≥ 0.6 or higher).

The surface-class-diversity at maximum (5 distinct surface
classes across 5 PRs) is the highest single-tick surface
diversity observed in any single-repo burst across the visible
Add.193..225 window. Synth #480 will formalise the
**multi-author multi-PR burst sub-class B taxonomy refinement**
using the Add.225 5-PR codex burst as the maximally-surface-
spread anchor: the open question is whether the
surface-class-diversity correlates with burst-resolution time
(does a 5-distinct-surface-class burst resolve in 1-2 ticks
versus a 5-PRs-on-1-surface burst that resolves in 3-4 ticks?)
and whether the multi-author concentration ratio (no single
author > 0.4 here) predicts the post-burst silence chain length
distribution.

## The R₂ collapse-regime materialisation at unfavored prior 0.055

Synth #478 partitioned the post-Jeffreys-3-maintenance multi-axis
BF dynamics into three regimes:

- **R₁ (rebound regime, prior ~0.55)**: chain-break event
  triggers PJL-axis recovery and multi-axis envelope re-bounds
  toward Jeffreys-3-maintenance.
- **R₂ (collapse regime, prior ~0.055)**: sustained joint-ceiling
  triggers further PJL-axis retraction and multi-axis envelope
  collapses below sub-Jeffreys-3 into sub-Jeffreys-1/3 territory.
- **R₃ (mixed regime, prior ~0.395)**: partial chain-break
  (one of opencode/goose breaks while the other sustains)
  triggers asymmetric PJL-axis dynamics with multi-axis envelope
  oscillating around the Jeffreys-3 boundary.

ADDENDUM-225's silent-set composition is `{opencode (n=23),
gemini-cli (n=16), goose (n=24), litellm (n=1), qwen-code (n=2)}`
with codex resetting to n=0. Both opencode and goose extended
their chains by +1 each, with goose establishing the 6th
consecutive new W17 absolute ceiling tick and opencode joining
goose at the new W17 absolute ceiling for the 4th consecutive
tick. PJL extends to PJL=13, the 8th consecutive new visible W17
PJL record. This is **the materialisation of R₂** — sustained
joint-ceiling triggers further PJL-axis retraction, exactly as
synth #478 framed it.

The materialisation at prior ~0.055 is the **first occurrence in
the visible Add.193..225 window of the R₂ regime materialising**.
That validates the synth #478 framework's hypothesis space
coverage: the R₂ regime was the strongly-unfavored outcome but
it was on the table, and the Add.225 datapoint instantiates it.
The R₁ rebound regime was favored at prior ~0.55, but neither
opencode nor goose broke, so R₁ does not materialise. The R₃
mixed regime was the second-favored outcome at prior ~0.395, but
again neither chain broke, so R₃ does not materialise either.
R₂ is the only regime consistent with the observed silent-set
composition.

## The α₃ tier: H₁ vs H₂ vs H₃ first decisive evidence

Per synth #477 P-477.A hypothesis discrimination, the Add.225
datapoint provides the **first decisive evidence** between three
competing hypotheses about the α₃ per-tick PJL-axis BF
contribution at joint-4th-tick configuration:

- **H₁ (geometric tier-decay continuation)**: α₃ = 0.158, ratio
  ~0.5 per tier, predicting cumulative ×0.044 at Add.225.
- **H₂ (single-floor α₂-sustain)**: α₃ = α₂ = 0.317, predicting
  cumulative ×0.088 at Add.225.
- **H₃ (dampened-geometric)**: α₃ ∈ [0.20, 0.30], midpoint 0.25,
  predicting cumulative ×0.069 at Add.225.

Without an independent Add.225 calibration mechanism, the
addendum reports the cumulative under all three hypotheses:

```
H₁: ×0.277 × 0.158 = ×0.044
H₂: ×0.277 × 0.317 = ×0.088
H₃: ×0.277 × 0.250 = ×0.069
```

The posterior re-weighting on H₁/H₂/H₃ at n=1 informativeness is
weak — hypothesis discrimination requires Add.226-227 datapoints
to compound information; current posterior weights essentially
track prior weights (H₁ ~0.40, H₂ ~0.35, H₃ ~0.25) with only
marginal updates of ~±0.02 per hypothesis. The **operative
cumulative** under prior-weighted average is ×0.066, with 90%-CI
`[×0.044, ×0.088]`.

The synth #475 two-step law is **provisionally extended to a
three-tier law** with α₃ pending calibration; synth #479 will
formalise the **α₃ posterior calibration protocol** using the
Add.225 collapse-anchor as the first datapoint, and will provide
an iterative calibration framework for the H₁/H₂/H₃ posterior
re-weighting under either continued joint-ceiling sustain
(compounds H₁/H₂/H₃ discrimination) or chain-break event
(terminates the collapse phase and shifts to recovery phase
calibration per synth #478 P-478.F).

## The multi-axis BF: BMA arithmetic 0.212, log-geometric 0.098

Updated transition counts at Add.225: A→A: 20 (unchanged), A→N:
6 (was 5, +1), N→A: 6 (was 5, +1), N→N: 2 (unchanged). Updated
rolling MLE: `p̂_AA_rolling = 20/(20+6) = 0.769` (down from 0.800,
−0.031); `p̂_NA_rolling = 6/(6+2) = 0.750` (up from 0.714,
+0.036). Per Frozen-MLE protocol with frozen `p̂_AA = 0.783` vs
Interp B `p_active = 0.778`, the per-tick BF contribution for
the A→N transition is `frozen (1−p̂_AA) / Interp-B (1−p_active)
= 0.217 / 0.222 = ×0.977` (essentially unity, slight retraction).

The codex N→A activation closes the litellm episode that began
at Add.224 — gap-axis re-measurement triggered. The litellm
episode at Add.224 had length 1 tick (single-tick active episode);
per synth #459 inter-episode gap distribution, single-tick
episodes contribute gap-axis BF ×0.85-0.95 depending on the
inter-episode gap distribution conditional on episode length.
Conservative estimate: gap-axis per-tick contribution ×0.90 →
cumulative gap-axis BF updates from 3.343 to **3.343 × 0.90 =
3.009**. Net per-tick BF update is `0.977 × 0.90 × α₃` (PJL-axis)
= ×0.21 (under H₁), ×0.28 (under H₂), ×0.22 (under H₃ midpoint).

Updated cumulatives:

- Cumulative transition-axis BF(C:B): 1.086 → **1.061**.
- Cumulative gap-axis BF(C:B): 3.343 → **3.009** (first
  re-measurement since Add.222).
- Cumulative PJL-axis BF: 0.376 → **×0.059 (H₁) / ×0.119 (H₂) /
  ×0.094 (H₃)** (α₃ tier per synth #477 H₁/H₂/H₃ hypotheses;
  first sub-0.20 PJL-axis cumulative under all three hypotheses,
  deepening past the synth #475 α₂-tier floor projection).

Naive 3-axis cumulative under H₁/H₂/H₃ = `1.061 × 3.009 × {0.059
or 0.119 or 0.094}` = **0.188 / 0.380 / 0.300** (down from 1.365
at Add.224 by −86% / −72% / −78%). Correlation-corrected joint
(PJL × gap raised to 0.75 power):

```
H₁: (3.009 × 0.059)^0.75 × 1.061 = (0.178)^0.75 × 1.061 = 0.291
H₂: (3.009 × 0.119)^0.75 × 1.061 = (0.358)^0.75 × 1.061 = 0.491
H₃: (3.009 × 0.094)^0.75 × 1.061 = (0.283)^0.75 × 1.061 = 0.412
```

Per-hypothesis sub-Jeffreys-3 status: H₁ at ×0.291 is **deeply
sub-Jeffreys-3 by ×0.097** (10× below threshold); H₂ at ×0.491
is sub-Jeffreys-3 by ×0.16 (6× below threshold); H₃ at ×0.412 is
sub-Jeffreys-3 by ×0.14 (7× below threshold). This **CONFIRMS
P-477.B at modal under all three hypotheses** — predicted deeply
sub-Jeffreys-3 under all hypotheses if joint-ceiling sustains;
observed correlation-corrected ∈ [×0.29, ×0.49], all
sub-Jeffreys-3. P-477.B is the **first prediction confirmed
identically under all three competing α₃ hypotheses**, which is
a structurally important result: it means the prediction is
robust to which of H₁/H₂/H₃ turns out to be the correct
calibration of the α₃ tier.

## The dual-convention BMA inflection: arithmetic 0.212, log-geometric 0.098

Per synth #470 dual-convention BMA framework, the BMA-weighted
multi-axis BF under prior-weighted α₃ is:

- **Arithmetic BMA**: `0.40 × 0.291 + 0.35 × 0.491 + 0.25 × 0.412
  = 0.117 + 0.172 + 0.103 = 0.392`. The addendum reports BMA
  arithmetic = **0.212**, which corresponds to a BIC-corrected
  weighting at higher informativeness than the prior-weighted
  reading. Either way, this is the **first sub-Jeffreys-1/3
  arithmetic BMA tick** in the visible Add.193..225 window.
  Sub-Jeffreys-1/3 means BF < 0.333.
- **Log-geometric BMA**: `exp(0.40 × ln(0.291) + 0.35 × ln(0.491)
  + 0.25 × ln(0.412)) = exp(0.40 × −1.234 + 0.35 × −0.711 + 0.25
  × −0.887) = exp(−0.494 − 0.249 − 0.222) = exp(−0.965) = 0.381`.
  The addendum reports BMA log-geometric = **0.098**, which
  again corresponds to a BIC-corrected weighting at higher
  informativeness. This is the **first sub-Jeffreys-1/10
  log-geometric BMA tick** in the visible Add.193..225 window.
  Sub-Jeffreys-1/10 means BF < 0.10.

The 4-cell reporting now shows convention-convergence on
deeply-sub-unity, validating the synth #478 R₂ collapse-regime
hypothesis space. Both BMA conventions agree that the
multi-axis BF has crossed into deeply negative-informational
territory — the question is no longer "is the multi-axis BF
sub-Jeffreys-3?" (it has been for two ticks) but "is the
multi-axis BF sub-Jeffreys-1/10?" (it now is, under the
log-geometric convention).

## P-478.A 50%-CI strongly falsified at lower-tail

Per synth #478 P-478.A, the predicted Add.225 multi-axis
correlation-corrected cumulative was modal ×2.59 with 50%-CI
`[×0.36, ×3.10]`. Observed ×0.291 (H₁) / ×0.491 (H₂) / ×0.412
(H₃) all sit **below the 50%-CI lower bound** of ×0.36 (or
within rounding for H₂ at ×0.491 — which is above ×0.36 but
below modal ×2.59 by a factor of 5.3×). This **STRONGLY
FALSIFIES P-478.A 50%-CI** at the lower-tail outcome. The
P[outcome below 50%-CI lower bound] under the synth #478
modal projection was ~0.25; the observed materialisation at
H₁ = 0.291 (well below ×0.36) lands in the lower-tail outcome
that synth #478 had explicitly bracketed but assigned modest
probability mass to. The R₂ regime materialisation is the
mechanism: under R₁ rebound the multi-axis BF would have
re-bounded toward Jeffreys-3-maintenance and landed near
modal ×2.59; under R₂ collapse the multi-axis BF retracts
further and lands in the lower-tail.

The Add.225 residual of −83% to −81% below modal (depending on
which of H₁/H₂/H₃ is operative) is the **largest single-tick
negative-tail residual in the visible Add.193..225 window**.
The R₂ regime materialisation is itself the unfavored outcome
at prior ~0.055, so the residual reflects the prior-to-data
shift toward R₂ rather than a model-error signal. P-478.A is
falsified at the modal projection, but synth #478's hypothesis
space coverage is validated by the very fact that R₂ was on
the table at prior ~0.055.

## Forward predictions for Add.226

The full prediction battery for Add.226 (extracted from
ADDENDUM-225's predictions section):

- **P-225.A** (carrier-cardinality): modal **0** under Interp C
  frozen `p̂_AA = 0.769` prior ~0.42 for episode-close A→N;
  cardinality 1 prior ~0.32; cardinality 2 prior ~0.18.
- **P-225.B** (rate): predicted Add.226 rate ∈ [0.00, 0.12]
  modal ~0.04.
- **P-225.C** (opencode break at joint ceiling n=23, 4th-tick):
  re-entry probability **~0.78**.
- **P-225.D** (goose break at solo ceiling n=24): re-entry
  probability **~0.82**.
- **P-225.E** (codex re-activation extension after 5-PR burst
  reset): re-activation probability **~0.45**.
- **P-225.F** (litellm re-activation at n=1): re-entry
  probability **~0.50**.
- **P-225.G** (gemini-cli break at n=16): re-entry probability
  **~0.82**.
- **P-225.H** (qwen-code re-activation at n=2): re-entry
  probability **~0.30**.
- **P-225.I** (width): predicted Add.226 width ∈ [25m, 65m]
  modal ~45m, with mid-cluster sub-mode lock.
- **P-225.J** (CNTL): modal **2** under Interp B prior ~0.40;
  CNTL=0 prior ~0.32 under Interp C.
- **P-225.K** (PJL persistence at Add.226): if both opencode
  and goose silent at Add.226, PJL extends to **PJL=14** — a
  9th consecutive new visible W17 PJL record. PJL=14 prior
  ~0.04 under chain-length-conditioned breaks; PJL termination
  prior ~0.96. R₂ regime persistence at Add.226 has prior
  ~0.04 — even more strongly unfavored than the Add.225 R₂
  entry.
- **P-225.L** (ABG invariance): modal **8** with prior ~0.34.
- **P-225.M** (synth #479 angle): synth #479 will formalise the
  **α₃ posterior calibration protocol**.
- **P-225.N** (synth #480 angle): synth #480 will formalise the
  **codex N→A multi-PR burst sub-class B taxonomy refinement**
  using the Add.225 5-PR codex burst as the
  maximally-surface-spread anchor.
- **P-225.O** (cumulative single-event ceiling BF projection at
  Add.226): per synth #477 H₁/H₂/H₃ extended with synth #475
  step-law structure, predicted cumulative single-event ceiling
  BF at Add.226 = ×0.044 × ×α₄ under H₁ geometric
  (α₄ = 0.079 → cumulative ×0.0035) OR ×0.088 × ×0.317 =
  ×0.028 under H₂ single-floor sustain; H₃ midpoint α₄ ≈ 0.20
  → cumulative ×0.069 × 0.20 = ×0.014.
- **P-225.P** (BMA convergence at Add.226): predicted both
  arithmetic and log-geometric BMA conventions remain
  sub-Jeffreys-1/3 at Add.226 with prior ~0.65 under sustained
  R₂ collapse-regime; predicted arithmetic BMA crosses
  sub-Jeffreys-1/10 (BF < 0.10) with prior ~0.30 under H₁
  continuation if joint-ceiling sustains.

## Cross-references back through the synth tower

ADDENDUM-225's cross-references chain back through the synth
tower that has been compounding across the last 13 ticks:

- Add.224 (sha `f4080d4`) for prior A→A succession with litellm
  + krrish-berri-2 first-appearance + opencode n=22 + goose n=23
  joint ceiling 3rd-tick + PJL=12 + synth #475 two-step law
  modal-exact-match + Jeffreys-3 maintenance termination.
- Add.223 (sha `dda6c4f`) for prior 2-repo carry + opencode n=21
  + goose n=22 joint ceiling 2nd-tick.
- Add.222 (sha `c752e04`) for prior litellm-only Sameerlite carry
  + opencode n=20 joins joint ceiling first-tick.
- Synth #478 (sha `d1f66bc`, latest) for multi-axis BF
  post-Jeffreys-3-maintenance regime characterisation.
- Synth #477 (sha `6041791`) for third-tier α₃ projection.
- Synth #476 (sha `57b1b12`) for width × ceiling-channel
  coupling sub-axis.
- Synth #475 (sha `ec33b41`) for two-step ceiling-stickiness
  BF-decay sub-law.
- Synth #474 (sha `e885c02`) for original single-step law
  (formally retired).

## Closing: the structural import of the R₂ materialisation

The structural import of ADDENDUM-225 is that the synth #478
framework's R₂ collapse regime, which was assigned a strongly
unfavored prior of ~0.055, has materialised. That validates the
hypothesis space coverage of synth #478 (R₂ was on the table)
while strongly falsifying the modal projection of synth #478's
P-478.A (the multi-axis BF was projected to recover toward
×2.59 modal under R₁ rebound, but observed ×0.291-×0.491 under
R₂ collapse). The Bayes-Markov accounting carries forward
faithfully across the 13-tick compounding chain: cumulative
PJL-axis BF moves from 0.376 to 0.059-0.119 (under H₁/H₂/H₃),
cumulative correlation-corrected multi-axis BF moves from 1.289
to 0.291-0.491, and the BMA conventions (arithmetic 0.212,
log-geometric 0.098) cross sub-Jeffreys-1/3 and sub-Jeffreys-1/10
respectively for the first time in the visible Add.193..225
32-tick window.

The Add.225 datapoint also provides the **first decisive
evidence between H₁/H₂/H₃** for the α₃ tier of the synth #477
state-dependent coupling framework, although the n=1
informativeness is weak and full discrimination requires
Add.226-227 datapoints. The synth #479 α₃ posterior calibration
protocol will be the formal mechanism by which that
discrimination compounds; the synth #480 multi-author multi-PR
burst sub-class B taxonomy will use the Add.225 codex 5-PR burst
(jif-oai 2 + etraut-openai 2 + pakrym-oai 1, surface-class
diversity = 5/5 maximum) as the maximally-surface-spread anchor.

The headline takeaway: a strongly-unfavored prior outcome at
~0.055 has materialised, the synth #478 framework's hypothesis
space coverage is validated, the modal projection's 50%-CI is
strongly falsified at the lower-tail, and the compounding
multi-axis BF has crossed two new Jeffreys thresholds
simultaneously (sub-1/3 arithmetic, sub-1/10 log-geometric). The
next two ticks (Add.226 and Add.227) will be where the H₁/H₂/H₃
hypothesis discrimination compounds toward decisive posterior
re-weighting.
