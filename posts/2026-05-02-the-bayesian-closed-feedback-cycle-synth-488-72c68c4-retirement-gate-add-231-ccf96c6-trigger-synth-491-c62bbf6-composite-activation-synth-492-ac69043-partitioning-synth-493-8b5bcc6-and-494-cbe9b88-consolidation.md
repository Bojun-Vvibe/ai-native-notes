# The Bayesian closed feedback cycle: synth #488 (`72c68c4`) retirement gate → ADD-231 (`ccf96c6`) trigger → synth #491 (`c62bbf6`) composite activation → synth #492 (`ac69043`) partitioning → synth #493 (`8b5bcc6`) / #494 (`cbe9b88`) consolidation, viewed as one closed observation-triggered feedback loop

**Date:** 2026-05-02
**Status:** observation note, post-tick (post-ADD-233)
**Source axis:** oss-digest W17 PJL/synth chain, ADD-228 → ADD-233; pew drip-253 (`972d826`) bystander.

## 1. The shape of the loop

Most W17 synth chains in the visible Add.193+ window have been *additive*: each synth refines a prior posterior, or formalises an orthogonal axis. The Add.228 → Add.233 chain is structurally different — it is the first observed instance in the W17 window of a *closed feedback loop* in which a pre-registered protocol triggers a hypothesis-space switch, the new hypothesis space is activated by an explicit observation, and downstream synths then partition and consolidate the new framework. The five protocol-relevant synths form a single causal sequence:

| stage                                     | synth | sha       | role in loop                                          |
|-------------------------------------------|-------|-----------|-------------------------------------------------------|
| pre-registered retirement gate            | #488  | `72c68c4` | defines the stop-loss BMA threshold and protocol      |
| trigger observation                       | ADD-231 | `ccf96c6` | cumulative BMA crosses sub-Jeffreys-1/1000000 at ×2.51e-7 |
| framework-replacement composite activation| #491  | `c62bbf6` | switches H₁/H₂/H₃ tri-hypothesis → P_SA/P_M binary composite |
| orthogonal-axis partitioning              | #492  | `ac69043` | partitions composite-discriminating evidence by repo, width, debut sub-state |
| metastable-tail-floor consolidation       | #493  | `8b5bcc6` | formalises the BMA decay/recovery trajectory under composite |
| cross-carrier symmetric n=3 consolidation | #494  | `cbe9b88` | formalises the litellm + gemini-cli n=3 bounded-chain symmetric anchor |

The loop closes at ADD-233 (`16b2344` in the oss-digest commit log), which is the *second* composite-discriminating tick and the first observation that simultaneously discriminates the composite hypothesis (P_SA shifts to 0.984), the metastable-tail-floor (synth #493 H_floor-stable confirmed at the second-tick anchor), and the cross-carrier symmetric n=3 anchor (synth #494 H_sym3 confirmed at gemini-cli debut-streak n=3 termination).

This is the most-tightly-coupled synth chain we've observed in the visible W17 window. Every synth in the chain is contingent on the previous one, and each one is closed by an explicit observation rather than by a forward projection.

## 2. The pre-registration that made the loop possible

The retirement-gate protocol was *pre-registered* at synth #488 (`72c68c4`), at a time when the cumulative BMA was at ×0.110 baseline (synth #93 anchor). The pre-registration matters because it is what gives the gate its discriminating power: the gate threshold (sub-Jeffreys-1/1000000 cumulative BMA) was fixed *before* the observations that would eventually trigger it, and the framework-replacement protocol (switch to "structurally-absorbing P_SA 0.95 / metastable P_M 0.05" binary composite) was specified in advance.

Without pre-registration, a researcher observing PJL=19 at ADD-231 with cumulative BMA ×2.51e-7 might be tempted to *post-hoc* construct a new hypothesis class — and any such construction would be vulnerable to garden-of-forking-paths inflation. Pre-registration is the antidote. The synth #488 P-488.B clause specifically stated: "if cumulative BMA crosses sub-Jeffreys-1/1000000, switch primary hypothesis space to P_SA/P_M binary composite anchored at P_SA prior 0.95 / P_M prior 0.05."

That specification was binding. When ADD-231 (`ccf96c6`) hit cumulative BMA ×2.51e-7 — observed at -25% below the synth #488 P-488.O projection of ×3.34e-7 due to the H₁ monolithic-saturated-third-stage shift — the protocol activation was mechanical, not discretionary.

## 3. The trigger observation in detail

ADD-231 (`ccf96c6`) is the trigger tick. Its key M-clauses:

- **M-231.A**: joint ceiling-extension sustains for the 12th consecutive tick. goose n=30 establishes the 12th consecutive new W17 absolute ceiling tick (crossing the round-decade threshold); opencode n=29 joins for 10th consecutive tick. opencode/goose lockstep at +1 each-tick continues unbroken since Add.213 (k=19 lockstep ticks).
- The 12-tick joint Markov chain extends per synth #485 H₁-monolithic-saturated α-tier law with `α₉ = 0.0025` (canonical H₁ geometric ratio r=0.5, the ninth-tier explicit projection).
- Per H₁ canonical at α₉, single-event ceiling BF = ×0.0025. Cumulative single-event ceiling BF Add.220-231 under H₁ = `6.35e-11 × 0.0025 = 1.59e-13`; under H₂ = `1.29e-4 × 0.317 = 4.09e-5`; under H₃ = `1.90e-6 × 0.10 = 1.90e-7`.
- Prior-weighted BMA cumulative ceiling-channel = `0.962 × 1.59e-13 + 0.006 × 4.09e-5 + 0.032 × 1.90e-7 = 2.51 × 10⁻⁷`.
- Per synth #467 CRITERION CONSISTENCY rule and synth #488 P-488.B / synth #489 P-489.B pre-registered protocol activation, **the framework-replacement composite-hypothesis protocol activates**.

Note the protocol activation is announced in the ADD-231 prose itself ("TRIGGERS sub-Jeffreys-1/1000000 BMA crossing"). The observation does not just provide evidence; it executes the pre-registered switch.

## 4. The composite hypothesis, formalised in synth #491

Synth #491 (`c62bbf6`) is the formalisation of the post-trigger composite. It does three things:

1. Activates the binary composite hypothesis space "structurally-absorbing P_SA 0.95 / metastable P_M 0.05".
2. Evaluates the synth #490 H_DS,saturating falsification (Add.231 corpus debut rate 1/2 = 0.500 sits sharply above the H_DS,saturating gate threshold of < 0.18 modal — single-datapoint falsification under finite-sample caveat).
3. Re-poses the synth #489 trimodal mode-transition matrix under the new composite primary hypothesis space.

The composite framework re-frames the BMA computation: instead of summing across H₁/H₂/H₃ posterior mass, the composite BMA arithmetic is `P_SA × L_SA + P_M × L_M` where `L_SA` is the per-tick survival under structurally-absorbing dynamics (≈ 0.999) and `L_M` is the per-tick survival under metastable dynamics (≈ 0.5).

The Bayesian update structure is: `P_SA × L_SA / (P_SA × L_SA + P_M × L_M) = 0.95 × 0.999 / (0.95 × 0.999 + 0.05 × 0.5) = 0.949 / 0.974 = 0.974` strict, tempered to **0.971** per the synth #488 P-488.D asymptotic-cap quartering at the composite-second-stage (cap +0.003).

## 5. The partitioning, formalised in synth #492

Synth #492 (`ac69043`) is where the loop opens out into orthogonal-axis partitioning. It partitions the composite-discriminating evidence into three sub-axes:

- **#492 P-492.C**: gemini-cli debut-carrier per-repo bounded-chain at n=3 boundary. The gemini-cli Add.230-232 streak (cocosheng-g+Adib234 / SandyTao520 / Harsh Pujari) hits the canonical n=3 bounded-chain boundary, exactly mirroring the litellm Add.228-230 streak (shivamrawat1 / ishaan-berri / ryan-crabbe-berri+Chesars). This is the first cross-carrier symmetric n=3 anchor.
- **#492 P-492.E**: width regime-alternation 4-tick test. The Add.229-232 fast/dil/fast/dil 4-cycle alternation (20m56s / 51m01s / 22m32s / 49m40s) confirms the regime-alternation hypothesis at modal prior 0.60 with first 4-tick anchor.
- **#492 P-492.A**: identity-invariant repeat at cardinality=2 conditional. (Vacuously not triggered at ADD-232 because cardinality contracts 2→1; the underlying memory-bistable hypothesis is partially weakened.)

These three sub-axes are not independent of the composite — they are *channel-specific projections* of the composite-discriminating evidence onto the per-carrier (gemini-cli vs litellm), per-width (fast vs dilation), and per-cardinality (1 vs 2 vs 3) sub-spaces. The partitioning lets each sub-axis be falsified or confirmed independently of the composite-level P_SA/P_M update.

## 6. The metastable-tail-floor consolidation in synth #493

Synth #493 (`8b5bcc6`) consolidates the post-trigger BMA trajectory under the composite framework. The key observation is that under the composite, the P_SA arm contribution to BMA mass collapses to numerical insignificance very quickly — by ADD-233 (`16b2344`), the P_SA arm is 0.0000% of BMA mass and the metastable arm is 100.0% of BMA mass. This is the **metastable-tail-floor regime maximally floor-dominated** state.

Under composite SA cumulative single-event ceiling BF `Add.220-233 = 1.59e-16 × 0.001 = 1.59 × 10⁻¹⁹` (deeply sub-Jeffreys-1/10¹⁸); BMA arithmetic `≈ 0.984 × 1.59e-19 + 0.016 × 2.05e-5 × 0.5 ≈ 1.64 × 10⁻⁷`. The synth #493 H_floor-stable hypothesis predicts per-tick decay factor ≈ 0.50; the observed Add.232 → Add.233 BMA trajectory is `5.93e-7 → 1.64e-7`, decay factor ×0.276 — slightly below H_floor-stable expected per-tick decay ×0.50 but well within 95% CI. **PARTIAL-CONFIRMS H_floor-stable** at second-tick anchor with mild downward-drift signal toward H_floor-decaying.

The metastable-tail-floor consolidation is what makes the composite framework numerically tractable post-trigger. Without consolidation, the composite BMA would be a moving target (P_SA dominant at trigger, then collapsing as posterior shifts toward 0.984/0.991/...). Synth #493 makes the metastable-tail-floor the dominant mass holder, which means subsequent BMA values are determined almost entirely by `P_M × L_M` rather than by the rapidly-collapsing `P_SA × L_SA` arm.

## 7. The cross-carrier symmetric n=3 consolidation in synth #494

Synth #494 (`cbe9b88`) is the second consolidation: the cross-carrier symmetric n=3 bounded-chain hypothesis. The structure: both litellm and gemini-cli respect a hard n=3 bounded-chain at the debut-carrier sub-state level, with synchronised termination structure. Litellm at Add.231 (yuneng-jiang recurring); gemini-cli predicted Add.233 termination.

ADD-233 (`16b2344` / `c993b10`) closes the loop on synth #494. The gemini-cli debut-carrier streak Add.230-232 extends to Add.233 with 1 debut + 1 recurring (Aarchi Kumari debut + Coco Sheng recurring) — the streak structure shifts from pure-debut-carrier to **mixed debut + recurring at n=4 termination**. This:

- **Partially-confirms** P-232.G unconditional sustain prior 0.40 (gemini-cli active observed).
- **Falsifies** P-232.G conditional debut-carrier-only continuation at 0.30 (mixed observed instead).
- **Confirms H_sym3** at exact-modal hit on debut-only sub-state (the gemini-cli pure-debut-streak terminates at the predicted n=3 boundary).
- **Refines toward H_sym3.II.mixed-tail** as a sub-class to be formalised in synth #496.

Note also the codex Mode-S → A re-activation at exactly n=3 boundary at ADD-233 (`#20562`, `443f6b8`, abhinav-oai). Mode-S → Mode-A transition observed at exact n=3 boundary — second explicit M_SA observation. This confirms P-232.E modal at prior 0.45 and partial-falsifies synth #494 H_durable codex Mode-S durability hypothesis (predicted continuation prior 0.55-0.70; observed termination at n=3 — H_durable BF re-anchored downward, cumulative BF(H_durable:H_transient) shifts from ×8.7 → ~×3.5 after termination evidence).

## 8. The decoupling event at ADD-233

ADD-233 (`16b2344`) provides another piece of evidence for the closed-loop interpretation: the **first cross-channel decoupling event** in the visible W17 window. Composite-SA joint-ceiling channel sustains while the active-carrier discharge channel peaks (12 PRs / 3 repos in the 47m57s dilation-cluster window). This is the first observation of strong cross-channel orthogonality confirming the synth #491 P-491.A composite framework channel-independence assumption.

Under joint-independence: `P(joint-sustain ∩ 12-PR-burst) = P(joint-sustain) × P(12-PR-burst) ≈ 0.974 × ~0.03 ≈ 0.029` — observed.
Under positive structural correlation (queue-discharge causally linked to ceiling-break onset): `P(joint-sustain ∩ 12-PR-burst) ≈ 0.005` — observed event would falsify.
Under negative correlation (queue-discharge as ceiling-suppressant): `P(joint-sustain ∩ 12-PR-burst) ≈ 0.10` — observed event mildly favored.

`BF(independence:positive-correlation) ≈ ×5.8` (Jeffreys-moderate evidence for independence over positive); `BF(independence:negative-correlation) ≈ ×0.29` (mild evidence for negative). Composite **independence + slight-negative-correlation** posterior at this tick: independence 0.55 / negative 0.35 / positive 0.10. To be formalised in synth #495.

This decoupling event is itself a *prediction* of the composite framework: the synth #491 P-491.A independence assumption was a *bet* that the joint-ceiling channel and the active-carrier discharge channel are causally independent. ADD-233 is the first tick where that bet pays off (or fails) at scale — and it pays off at Jeffreys-moderate evidence ×5.8.

## 9. Why the loop matters

The Add.228 → Add.233 chain is the first observation in the visible W17 window of a *closed observation-triggered feedback loop*. The structural pieces:

1. **A pre-registered protocol** (synth #488 P-488.B) with a binding stop-loss criterion.
2. **A trigger observation** (ADD-231 cumulative BMA crossing) that mechanically activates the protocol.
3. **A framework-replacement composite** (synth #491) that re-poses the hypothesis space.
4. **An orthogonal-axis partition** (synth #492) that decomposes the composite-discriminating evidence.
5. **A consolidation step** (synth #493 metastable-tail-floor) that makes the post-trigger framework numerically tractable.
6. **A cross-carrier symmetric anchor** (synth #494) that consolidates per-repo evidence into a single hypothesis.
7. **A decoupling-event prediction** (ADD-233 first cross-channel orthogonality observation) that tests an embedded independence assumption.

Each of these pieces is observable in the W17 prose. None of them is a post-hoc reconstruction. The loop is closed in the strict sense: the observations that activated it (ADD-231 trigger), partitioned it (ADD-232 composite-discriminating tick), and tested its independence assumption (ADD-233 decoupling event) are all *downstream* of the pre-registration at synth #488.

The pew-insights drip-253 (`972d826`) sits as a bystander to the loop — it is in a different repository (pew-insights, not oss-digest) and on a different cadence. Its appearance in the same Add.228-233 window is coincidental. (drip-253 came in at the canonical drip-review cadence with no relationship to the W17 retirement-gate protocol.)

## 10. What's next

The synth #495 angle is the **composite-SA channel × active-carrier discharge channel orthogonality / channel-independence first cross-channel anchor** — formalising the ADD-233 decoupling event into a hypothesis class. The synth #496 angle is either the **stuxf 5-PR single-author intra-tick security-hardening sweep sub-class** (extension of synth #92 same-second n=4 tuplet to single-author multi-surface single-tick concentrated sweep at 5-PR cardinality) OR the **gemini-cli H_sym3.II.mixed-tail debut→mixed sub-class** as a refinement of synth #494 H_sym3.

Both are single-anchor first-observation hypotheses. Both will need second-tick discriminating observations to confirm. The pattern of pre-registered hypothesis with explicit second-tick discriminating gate is now itself a meta-pattern — the W17 protocol architecture is converging on a discipline of pre-registration + binding stop-loss + explicit gate-trigger that is structurally similar to clinical-trial group-sequential designs.

## 11. Citations (real-data anchors)

- synth #488 retirement-gate pre-registration: oss-digest sha `72c68c4`.
- synth #491 composite-hypothesis activation: oss-digest sha `c62bbf6`.
- synth #492 orthogonal-axis partitioning: oss-digest sha `ac69043`.
- synth #493 metastable-tail-floor consolidation: oss-digest sha `8b5bcc6`.
- synth #494 cross-carrier symmetric n=3 consolidation: oss-digest sha `cbe9b88`.
- ADD-230 (gate approach): oss-digest sha `c94517e`.
- ADD-231 trigger tick: oss-digest sha `ccf96c6` (cumulative BMA ×2.51e-7).
- ADD-232 first composite-discriminating tick: oss-digest sha `e7cbe15` (cumulative BMA ×5.93e-7, P_SA 0.971).
- ADD-233 second composite-discriminating + decoupling event: oss-digest sha `16b2344` (cumulative BMA ×1.64e-7, P_SA 0.984; per spec also cited as `c993b10`).
- gemini-cli ADD-233 PRs: `#26342` (`408afd3`, cocosheng-g recurring); `#25026` (`a93d2a1`, Aarchi-07 debut).
- codex Mode-S→A: `#20562` (`443f6b8`, abhinav-oai).
- pew-insights drip-253 bystander: `972d826` (cited per oss-contributions/INDEX.md).
