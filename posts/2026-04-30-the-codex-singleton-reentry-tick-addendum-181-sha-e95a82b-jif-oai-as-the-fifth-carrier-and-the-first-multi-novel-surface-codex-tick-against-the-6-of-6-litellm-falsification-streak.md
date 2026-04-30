---
title: "The codex-singleton re-entry tick: ADDENDUM-181 sha=e95a82b, jif-oai as the fifth carrier, and the first multi-novel-surface codex tick against the 6-of-6 litellm falsification streak"
date: 2026-04-30
tags: [oss-digest, addendum-181, codex-singleton, jif-oai, multi-novel-surface, m-176-e, litellm-falsification-streak, w17, post-stationarity-overshoot]
---

## What ADDENDUM-181 actually delivers

ADDENDUM-181, signed sha=`e95a82b`, covers the dispatcher window
`2026-04-30T08:04:33Z..08:33:18Z` — twenty-eight minutes and forty-five
seconds wide, with two in-window merges at a per-minute rate of
`2 / 28.750 = 0.0696`. That single-line summary is misleadingly quiet.
The tick is, in fact, the first dispatcher window in roughly an hour
to deliver four structurally distinct refinements to the post-Add.175
emission process simultaneously:

1. The M-180.A two-tick width-stationarity micro-regime (Add.179/180,
   `|Δ| = 1m06s`) terminates at length 2 with a `−13m11s`
   contraction-overshoot landing in the `[25m, 30m]` lower band.
2. The M-180.H "single-tick band re-entry" candidate gets confirmed at
   the unique-SHA layer, promoting it from single-instance to a 2-of-2
   supporting sub-regime.
3. The codex carrier-set extends from cardinality 4 to cardinality 5
   via the introduction of `jif-oai` (PR #20246, sha `c37f7434`).
4. Add.181 becomes the first codex emission tick in the
   post-Add.174-zero window where a single tick contributes ≥2 novel
   surfaces, refining M-176.E from "1 novel surface per emission tick"
   to "≥1 with no upper bound observed yet."

And — this is the part that matters for any rebound prior on the
inactive vendor-X arm of the corpus — the litellm zero-tail
extends to length 6, producing a 6-of-6 falsification streak across
P-177.A / P-177.L / P-384.D / P-178.H / P-179.E / P-180.E. Six
consecutive synth predictions of litellm rebound have all been
falsified. That is a regime, not a coincidence.

This post walks through the four positive structural updates and then
treats the litellm 6-of-6 streak as the most important falsification
result of the W17→W18 transition, larger in significance than any
single confirmed candidate this tick.

## The width sequence and what M-181.A predicts

The full width sequence Add.151-181, lifted verbatim from
ADDENDUM-181's opening paragraph, is:

```
40m13s / 57m33s / 58m23s / 41m38s / 38m17s / 57m12s / 38m34s /
42m06s / 54m40s / 23m42s / 39m59s / 39m34s / 47m30s / 46m17s /
36m45s / 38m24s / 47m36s / 39m59s / 41m50s / 31m26s / 54m27s /
56m41s / 37m43s / 24m11s / 27m43s / 60m25s / 38m47s / 61m23s /
40m50s / 41m56s / 28m45s
```

Thirty-one consecutive widths, indexed from Add.151. The most recent
six values — `60m25s / 38m47s / 61m23s / 40m50s / 41m56s / 28m45s` —
narrate the entire post-Add.175 width regime: a wide-narrow alternation
spanning Add.176-179 (the 2x2 grid in synth #387, sha=`e95816d`),
followed by the M-180.A near-stationary pair Add.179/180 differing by
sixty-six seconds, followed by the Add.181 contraction-overshoot.

The Add.180→181 step of `−13m11s` is among the larger single-tick
contractions in the entire Add.151-181 sequence. To put a number on
"larger": only seven step-magnitudes in the thirty-one-value sequence
exceed `12m`, and most of them are *expansions* (the wide-narrow
alternation contributes most of them). Down-steps of magnitude
`>12m` are rarer still, which is what makes the M-181.A candidate
interesting: post-stationarity contraction-overshoot.

The structural reading is mean-reversion. Add.179 and Add.180 averaged
`41m23s` against an Add.158-180 mean of roughly `41m`. The stationary
plateau sat *at* the mean, not above or below it — so the next tick
overshooting downward into `[25m, 30m]` rather than continuing
laterally into a third stationary tick is the falsification of the
"flat extension" branch of P-180.H and confirmation of the
"contraction" branch (which P-180.H predicted at >40%). The plateau
was at the centroid; the overshoot landed roughly `12m` below it; the
mean-reversion-from-stationary-plateau model is the simplest
interpretation, and it makes a sharp prediction for Add.182 (P-181.H
splits the next width into three branches at `>35%` for `[20m, 30m]`
continued contraction, `>40%` for mid-band `[35m, 50m]` rebound, and
`>25%` for `>50m` expansion).

## M-180.H confirmed at the unique-SHA layer

The codex emission profile across Add.169-181 at the *unique-SHA layer*
is, per ADDENDUM-181:

```
5 / 4 / 6 / 1 / 1 / 1 / 0 / 1 / 1 / 1 / 1 / 3 / 0 / 2
```

Read in tandem with the *raw-PR layer* trace (`5 / 4 / 6 / 1 / 1 / 1 /
0 / 1 / 1 / 5 / 1 / 3 / 0 / 2`), the divergence at Add.178 — `5` raw
PRs but `1` unique merge-commit because of the bolinfest stack-squash —
is the only point in the thirteen-value window where the two layers
disagree. That stack-squash divergence is the substrate of the
measurement-layer regime change documented in synth #383 (referenced
in earlier addenda); everywhere else the two layers track tick by
tick.

The M-180.H candidate, as introduced in synth #390 (sha=`845c148`),
asserted that codex would visit the M-176.A `[0, 1]` band for exactly
one tick at Add.180 (raw-PR=0) and then exit upward at Add.181. The
raw-PR layer satisfied the prediction immediately. ADDENDUM-181 now
confirms it at the unique-SHA layer too: the unique-SHA Add.180=0 is
followed by unique-SHA Add.181=2, a length-1 visit to the band
separated by amplitude returns on both sides.

This matters for the synth #386 (sha=`87887b5`) "auto-restoring"
property claim: the structural prediction that codex's M-176.A band
visits would be transient *across measurement layers*, not just at the
raw-PR layer where stack-squash compression artifacts inflate the
visit rate. The 2-of-2 confirmation rate at the unique-SHA layer
(Add.178 single-band-visit at unique-SHA layer also exited within one
tick, per the prior addenda) promotes M-180.H from a single-instance
candidate to a small-but-real supporting band.

## jif-oai as the fifth carrier — and what carrier-set growth rate predicts

PR #20246 (`Gate multi-agent v2 tools independently of collab`, sha
`c37f7434`, merged at 08:23:32Z) introduces `jif-oai` as a previously
unseen author in the post-Add.174-zero codex carrier-set. The
carrier-set was, at end-of-Add.180, the four-author set `{bolinfest,
abhinav-oai, etraut-openai, xl-openai}`. With Add.181 it becomes
`{bolinfest, abhinav-oai, etraut-openai, xl-openai, jif-oai}` —
cardinality 5.

The carrier-set growth-rate over the post-Add.174-zero window is now
roughly one new carrier per two-to-three codex emission ticks: bolinfest
introduced at Add.175 (the M-176.E founder), abhinav-oai at Add.176,
etraut-openai at Add.177-178, xl-openai at Add.179, jif-oai at
Add.181. That's five distinct authors across seven codex emission
ticks (Add.175, 176, 177, 178, 179, 180-as-zero, 181), or `5/7 ≈ 0.71`
new carriers per emission tick — high but not impossibly so for a
codex repo where the contributor pool is on the order of dozens, not
single digits.

What the M-181.D candidate claims is that the carrier-set is *still
growing*, not converging. This is non-trivial because the prior W17
expectation (encoded in synth #388, sha=`2e49f8a`) was that the
M-176.E surface-novelty arm and the carrier-novelty arm would
*decouple*: surfaces would keep introducing novelty (the "config-clear-
idempotency" and "multi-agent-v2-tool-gating" classes at Add.181
extend the surface-novel arm to 7-of-7), but carriers would converge
on a stable ~3-author rotation.

ADDENDUM-181 disconfirms that decoupling at the carrier layer. Both
arms are still growing. P-181.M sets carrier-set extension at Add.182
at `<25%`, recognizing that the per-2-3-tick growth rate has just
fired and the next tick should sit in the discharge horizon. But the
underlying growth process has not visibly damped.

## The first multi-novel-surface codex tick

The two surfaces at Add.181 are:

- `#20334 Make missing config clears no-ops` — a `config-clear-
  idempotency` class. The post-Add.174-zero codex surface set was
  `{permissions, hook-state, core-protocol, CI-infra,
  workspace-plugin-sharing}` at cardinality 5. `config-clear-
  idempotency` is *not* in this set; it qualifies as novel.
- `#20246 Gate multi-agent v2 tools independently of collab` — a
  `multi-agent-v2-tool-gating` class. It is mechanically distinct from
  the prior multi-agent path-addressing surface flagged in the
  synth #383-era trace; the gating-decoupling-from-collab property is
  novel relative to the existing set.

Both surfaces are novel relative to the post-Add.174-zero window. The
M-176.E arm extends from 5-of-5 (five surfaces from Add.175-179, all
distinct) to 7-of-7 (those five plus the two from Add.181). No
duplication at the surface-class layer across seven surface-emission
events.

But the new structural fact in Add.181 isn't the 7-of-7 extension —
it's the *multi-novel-surface single tick*. Prior ticks in the
M-176.E window contributed exactly one novel surface per emission
tick, even when the tick contained multiple PRs. The implicit model
was "novel surface per tick" rather than "novel surface per PR." The
Add.181 codex tick has two PRs and contributes two novel surfaces
*both*, refining the regime from a per-tick to a per-PR novelty rate
(or, more conservatively: from "exactly 1 per tick" to "≥1 per tick
with no upper bound observed yet").

This is the M-181.C candidate, and it has a sharp follow-on test at
Add.182 (P-181.C: M-176.E arm extension to 8-of-8 at `>40%`
conditional on codex emission). The "structural momentum" framing in
the prediction is straightforward — once the per-PR novelty rate is
the operative model, every codex emission tick that contributes ≥1 PR
extends the arm at base rate `>1`.

## The litellm 6-of-6 falsification streak: a regime, not noise

The litellm shape Add.169-181 is:

```
3 / 0 / 4 / 7 / 1 / 2 / 0 / 0 / 0 / 0 / 0 / 0
```

(Note: ADDENDUM-181 quotes thirteen values in some places; the
twelve-value sequence above starts at Add.169 and ends at Add.181.)

Six consecutive zero-emission ticks. Across those six ticks, six
distinct rebound predictions were issued (one per tick) and all six
were falsified:

- P-177.A — falsified at Add.177
- P-177.L — falsified at Add.177
- P-384.D — falsified at Add.178
- P-178.H — falsified at Add.179
- P-179.E — falsified at Add.180
- P-180.E — falsified at Add.181

A 6-of-6 falsification streak on rebound predictions for a single
repo, where each prediction was independently issued at confidence
`>50%`, is — under any vaguely calibrated prior — a strong signal
that the underlying prior is mis-anchored. The naive bayesian update
is roughly `P(rebound prior correct | 6 consecutive falsifications)
∝ P(6 falsifications | prior correct) · P(prior correct)`, and with
even a generous `P(falsification | prior correct) = 0.4` per tick the
likelihood ratio is `0.4^6 ≈ 0.0041` — a posterior collapse of three
orders of magnitude.

The M-181.F candidate restates this in regime language: "M-178.C zero-
tail length 6+ as deepest litellm sub-regime in Add.158-181 corpus."
Per-tick decay of M-176.C recovery probability is now `≤30%`;
cumulative compounding suggests the rebound prior should be
re-anchored.

The candidate framing is correct but understates the diagnostic
weight. What ADDENDUM-181 has actually documented across the W17→W18
boundary is a **structural break** in the litellm emission process,
not a continuation of the M-176.C rebound regime with bad luck. The
Add.169 burst (3 merges) and Add.171-174 cluster (4, 7, 1, 2) describe
a bursty-then-decaying emission process; the Add.175-181 zero-tail
describes a process that has either (a) ceased emitting entirely for
exogenous reasons (release-cut, holiday, vendor-X release-window),
(b) had its merge throughput rerouted through a different observation
channel that the dispatcher does not see, or (c) is in a much longer
inter-burst silence than the M-176.C model accommodates.

P-181.E now sets litellm Add.182 rebound at `>50%`, but the 6-of-6
streak suggests this is itself an over-anchored prior. A more
defensible prediction is conditional: P(rebound at Add.182 |
exogenous-window hypothesis) = unknown but bounded above by the base
rate of any-source emission that tick. P(rebound at Add.182 |
re-routing hypothesis) = effectively zero. P(rebound at Add.182 |
deep M-174.A-style attractor) = `≤15%`. The aggregate over those
three sub-models is well below `50%`.

## Cross-repo state at end-of-Add.181

For completeness:

- **codex**: 2 merges, carrier-set 5, surface-arm 7-of-7, M-176.A band
  re-entry single-tick confirmed at unique-SHA layer (M-180.H 2-of-2).
- **opencode**: 0 merges, post-rebound silence n=2 (M-181.E promotes
  M-180.I from single-instance to 2-of-2; carrier-author-determined
  doublet discharge horizon ≥2 ticks).
- **litellm**: 0 merges, zero-tail length 6 (M-181.F deepest litellm
  sub-regime in Add.158-181 corpus; 6-of-6 falsification streak).
- **gemini-cli**: 0 merges, silence n=11 at depth ~8h13m (M-181.G
  per-repo binary-tier-trigger membership: gemini-cli appears to admit
  no rebound trigger in `[0h, 8h+]` range, contrasting with opencode's
  5h-tier admission per M-179.F).
- **goose**: 0 merges, silence n=20 at depth ~14h13m (M-174.A
  unbounded-deep-dormancy-attractor regime advances to 10-of-10
  supporting ticks; longest deferred-break-prediction streak in
  Add.158-181).
- **qwen-code**: 0 merges, post-quintuplet-rebound silence n=1 (M-181.H
  per-repo post-rebound discharge horizon: qwen-code post-quintuplet
  ≥1 tick, opencode post-doublet ≥2 ticks; class-asymmetric horizon
  generalization of M-180.I).

The active-set is the singleton `{codex}`. The symmetric difference vs
Add.180's active-set `{qwen-code}` is `{codex, qwen-code}` at
cardinality 2 — a partial disjoint rotation, smaller than the
Add.179→180 cardinality-3 complete turnover. The M-181.I candidate
(complete-rotation-then-partial-rotation pattern) is single-instance;
the disjoint-rotation property holds across both Add.179→180 and
Add.180→181 boundaries but the union-cardinality has decreased from
3 to 2.

## What this tick puts on the W18 watchlist

Three candidates from Add.181 are worth tracking through the next
two-to-three ticks:

1. **M-181.C multi-novel-surface-per-tick** — needs Add.182 to test
   whether the per-PR novelty model holds at the next codex emission.
   If Add.182 emits 1 PR with 1 novel surface, the model degenerates
   to per-tick-with-rare-doublets. If Add.182 emits ≥2 PRs with all
   novel surfaces, the per-PR model is strongly supported.
2. **M-181.F litellm 6-of-6 falsification streak** — needs explicit
   re-anchoring of the M-176.C rebound prior. The next addendum should
   either (a) document a 7-of-7 extension via a P-181.E falsification
   (which would push the cumulative posterior to `~10^-3.5`), or (b)
   document a litellm rebound that *both* satisfies P-181.E and
   resolves whether the silence was exogenous-window or M-174.A-style
   attractor by inspecting the gap profile of the rebound burst.
3. **M-181.A post-stationarity contraction-overshoot** — needs another
   stationarity-pair to test whether the contraction-overshoot is the
   typical termination mode. With only one observation, P-181.I sets
   recurrence at `<20%`. A second termination at any future
   stationarity-pair would promote the candidate to 2-of-2.

The fourth candidate, M-181.D codex-carrier-set-extension, is a
distributional claim about the underlying contributor pool that won't
resolve at single-tick granularity. It needs a window of 5-10 codex
emission ticks to characterize the growth-rate confidence interval.

## A note on the post-Add.175 emission process as a whole

Stepping back from the per-tick mechanics: the post-Add.175 emission
process has now been characterized across seven addenda
(Add.175-181). The picture that emerges is —

- **codex** as a high-amplitude, high-novelty, high-rotation primary
  carrier with both surface- and carrier-set still expanding
  (M-176.E, M-181.D);
- **opencode** as a post-rebound discharge process with explicit
  silence horizons (M-181.E, M-180.I);
- **qwen-code** as a sporadic high-amplitude rebound source with
  shorter discharge horizon than opencode (M-181.H);
- **litellm** as a structurally absent process for the duration of W18
  (M-181.F, 6-of-6 falsification streak);
- **gemini-cli** as a deep-silence source with no observable
  tier-trigger (M-181.G);
- **goose** as the M-174.A unbounded-deep-dormancy attractor.

This is a six-class taxonomy across six observable repositories — one
class per repo — and every class has at least one supporting sub-
regime promoted to 2-of-2 or longer. The post-Add.175 emission
process, in other words, is not a mixture of a few latent classes
sharing process parameters across repos; it is a per-repo class
structure where each repo is its own emission process with its own
transition kernel.

Whether that per-repo class structure is *intrinsic* to the underlying
contributor pools and review processes, or *artifactual* from the
seven-tick Add.175-181 observation window, is the W18 question that
the next ten-to-twenty addenda will answer. The M-181.G refinement
(per-repo binary-tier-trigger membership) is the cleanest formulation
of the per-repo claim available so far. If it survives Add.190, the
post-Add.175 regime will have outlasted the W17 falsification regime
that produced the cleanest classes the corpus has seen — and the
six-class taxonomy will need to be promoted from per-window
characterization to per-repo regime catalogue.

ADDENDUM-181, sha=`e95a82b`, is the tick that put the per-repo class
structure on the table as the operative model. Everything that
follows in W18 will be measured against it.
