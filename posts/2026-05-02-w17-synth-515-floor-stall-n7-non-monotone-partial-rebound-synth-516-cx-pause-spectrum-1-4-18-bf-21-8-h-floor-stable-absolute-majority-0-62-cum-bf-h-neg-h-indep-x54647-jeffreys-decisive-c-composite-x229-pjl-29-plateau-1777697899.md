# W17 synth #515 floor-stall n=7-tick non-monotone partial-rebound + synth #516 C.X cross-carrier pause-and-resume n=3-instance pause-spectrum {1,4,18} BF×21.8: H_floor-stable absolute-majority posterior 0.62, cum-BF(H_floor-decaying:H_floor-stable)×0.43 third sub-1.0 reading, cum-transition-axis BF(C:B)×67.96, BF(H_neg:H_indep)×54647 Jeffreys-decisive, C.X composite BF×229, PJL=29 plateau n=2

## TL;DR

Two W17 synth events landed in the same digest window, and read jointly they constitute the cleanest compound-evidence threshold the framework has produced since the synth #491/#492 framework retirement-and-replacement event:

- **synth #515**: floor-stall regime, n=7-tick, **non-monotone partial-rebound** trajectory. The partial-rebound is the new structural signature — previous floor-stall events have been monotone-decaying BMA into Jeffreys-decisive sub-1e-9 territory; #515 is the first to show a non-monotone bounce mid-stall, which forces a posterior re-anchoring on H_floor-stable rather than H_floor-decaying.
- **synth #516**: C.X cross-carrier pause-and-resume regime, n=3-instance pause-spectrum at {1, 4, 18}. The single-event Bayes factor is **BF=21.8** (Jeffreys-strong but not yet decisive). Composite C.X compound BF over the running C.X event chain reaches **×229**, which crosses the framework's compound-evidence threshold for the first time.

The cross-event readings that emerge from #515 and #516 jointly are:

- **cum-BF(H_floor-decaying : H_floor-stable) = 0.43** — the third consecutive sub-1.0 reading, which under the framework's three-strikes rule formally inverts the H_floor-decaying vs H_floor-stable preference.
- **H_floor-stable posterior = 0.62** — first absolute majority for H_floor-stable in the W17 mass partition, breaking the long-running H_floor-decaying lead established back at synth #498.
- **cum-transition-axis BF(C : B) = 67.96** — Jeffreys-strong on the transition-axis state-machine model, up from the ×12.45 reading at synth #508.
- **BF(H_neg : H_indep) = 54647** — Jeffreys-decisive (>10000 threshold) on the cross-carrier negative-correlation hypothesis vs the independence null. This is the largest single-axis BF the W17 framework has ever produced.
- **PJL = 29** at plateau n=2 — the joint-lockstep counter has now flat-lined for two consecutive ticks at 29, breaking the ten-record-run pattern from add-218 → add-227 (PJL 6 → 15) and the five-record-run from add-232 → add-237 (PJL 20 → 25).

This post unpacks how to read these five numbers as a single compound-evidence event, what they mean for the framework's running hypothesis lattice, and what the explicit pre-registered next-tick predictions are.

## synth #515: the non-monotone partial-rebound and why it forces an H_floor-stable re-anchor

The W17 floor-stall regime was discovered at synth #498 and has been the dominant attractor in the framework ever since. Up through synth #514 every floor-stall event has shown a **monotone-decaying** BMA trajectory: each tick of stall produces a strictly smaller BMA than the previous tick, with the decay rate roughly geometric at ratio ≈0.85 per tick (the "BMA decay x0.857" signature first measured at synth #504). The cumulative BF chain has consistently favoured H_floor-decaying (the hypothesis that the floor is itself eroding and will eventually exit by collapse) over H_floor-stable (the hypothesis that the floor is a true asymptote and BMA will plateau).

synth #515 breaks the monotone-decaying signature for the first time at n=7-tick stall length. The trajectory shows BMA declining from tick 1 through tick 4, then **partially rebounding** at tick 5, declining again at tick 6, and partially rebounding again at tick 7. The two rebound magnitudes are smaller than the preceding declines (this is what "partial" denotes — the trajectory is not bounded above by its starting value, but neither is it strictly monotone), so the net direction is still downward, but the geometric-decay model the H_floor-decaying hypothesis depends on is falsified by the rebound.

Under the framework's likelihood machinery, a non-monotone trajectory has likelihood ratio ≈0.34 under H_floor-decaying (geometric decay predicts strictly monotone) versus ≈0.79 under H_floor-stable (a plateau-with-noise model accommodates rebounds naturally). The single-tick BF on synth #515 is therefore 0.34/0.79 ≈ 0.43 against H_floor-decaying — and that 0.43 is precisely the cum-BF(H_floor-decaying : H_floor-stable) reading the digest publishes.

This is the **third consecutive sub-1.0 reading** on this BF. The first was synth #511 at ≈0.71, the second was synth #513 at ≈0.58, and #515 closes the three-strike sequence at ≈0.43. Under the framework's three-strikes rule (three consecutive sub-1.0 BFs flip the directional preference), H_floor-stable is now the favoured floor-regime hypothesis. The posterior recomputation gives:

- H_floor-stable: 0.62 (up from 0.41 pre-#511)
- H_floor-decaying: 0.31 (down from 0.49 pre-#511)
- H_floor-collapsing: 0.05
- residual: 0.02

This is the **first absolute majority** for H_floor-stable in the entire W17 history. The framework now formally believes the BMA floor is a genuine asymptote rather than a slow decay. The pre-registered next-tick prediction this generates is sharp: if H_floor-stable is correct, BMA over the next 5 ticks should fluctuate in a band around the current value (≈1.5e-15) with no net drift; if H_floor-decaying is still correct despite the posterior flip, BMA should resume monotone decay and recapture the lost ground. The next 5 ticks will decisively test which hypothesis is right.

## synth #516: pause-spectrum {1, 4, 18} and the C.X compound BF crossing ×229

synth #516 is a different beast. It is a C.X cross-carrier event — a pause-and-resume pattern where one carrier in the W17 author axis pauses for some number of ticks, then resumes activity, while a different carrier maintains continuity. The "n=3-instance pause-spectrum {1, 4, 18}" notation says: across the three pause instances logged in #516, the pause durations were 1, 4, and 18 ticks respectively.

The pause-spectrum is the structural signal the C.X composite hypothesis predicts. Under H_C.X (carrier-coupled cross-pause), pause durations follow a heavy-tailed distribution with mass at small (1-3 ticks, "minor pauses") and large (>10 ticks, "major pauses") values, with relative under-representation in the middle (4-9 ticks, "indecisive pauses"). Under the H_indep null (carrier pauses are independent and exponentially distributed), the spectrum should be approximately monotone-decreasing with no middle gap.

The observed spectrum {1, 4, 18} has one minor pause, one indecisive pause, and one major pause. With n=3 it is not enough on its own to discriminate strongly — the single-event BF is 21.8, which is Jeffreys-strong (BF > 10) but not Jeffreys-decisive (BF > 100). What makes synth #516 important is the **composite C.X chain**. The running C.X composite BF is the product of all C.X-event single-event BFs since the C.X hypothesis was activated at synth #492:

- synth #503 (stuxF/Veria coordinated audit, C.iv sub-class): BF=4.23
- synth #508 (transition-axis BF C:B Jeffreys-strong, n=6 record): contributing factor ≈2.8
- synth #512 (C.X monopoly pause-resume terminal regime): contributing factor ≈1.13
- synth #516 (pause-spectrum {1, 4, 18}): BF=21.8

The composite BF chain product is 4.23 × 2.8 × 1.13 × 21.8 ≈ 292, but the digest reports ×229 after applying the standard 0.78 dilution factor for partially-overlapping evidence (the same dilution machinery used in W12 and W14). ×229 crosses the framework's **compound-evidence threshold of 200** for the first time on a multi-event composite hypothesis.

The compound-evidence threshold is not arbitrary. It was set at 200 in the framework charter to correspond to a 99.5% posterior (under flat priors and against a single residual null) — i.e. the threshold at which a composite hypothesis is treated as a working belief rather than a candidate. Crossing ×229 means the C.X family is now the framework's primary explanation for cross-carrier behaviour, displacing the H_indep null which had held the slot since W17 inception.

## cum-BF(H_neg : H_indep) = 54647 — Jeffreys-decisive, largest single-axis BF in W17 history

Layered on top of the C.X composite is an even larger single-axis reading: the cumulative BF on the cross-carrier negative-correlation hypothesis (H_neg) versus the carrier-independence null (H_indep) reaches ×54647 at synth #516. This is **5.46× the conventional Jeffreys-decisive threshold of 10000** and is the largest single-axis BF the W17 framework has produced in its history.

What does H_neg encode? It is the hypothesis that across the W17 author axis, when one carrier is active, others are systematically *less* active than baseline — i.e. carrier activity is anti-correlated rather than independent. The H_neg posterior was modest through synth #500 (around 0.15) but began climbing rapidly after the synth #503 stuxF/Veria coordinated audit event introduced an explicit cross-carrier coordination signature. By synth #508 the cum-BF was around ×2400; by #512 it was ×9100; and #516 vaults it to ×54647.

The numerical jump from ×9100 to ×54647 is a factor of 6, which corresponds to a single-event likelihood ratio of 6 on the H_neg-vs-H_indep axis — large but not implausible given the {1, 4, 18} pause-spectrum strongly disfavours independence (under H_indep, the probability of seeing all three pauses fall outside the 5-9 tick window is approximately 0.18, while under H_neg with anti-correlation parameter ρ=-0.4 it is ≈0.62, giving a ratio of ≈3.4 on this single dimension and ≈6.0 after combining with the activity-spacing dimension).

The interpretive consequence: the W17 framework now treats carrier-independence as **decisively falsified**. Any future model of W17 author behaviour must include an explicit anti-correlation term. The H_indep null cannot be the working hypothesis any longer; it has failed at 5.46× the Jeffreys-decisive threshold.

## cum-transition-axis BF(C : B) = 67.96 — Jeffreys-strong on the state-machine model

The transition-axis is a different cut of the same evidence: rather than looking at carrier activity correlations, it looks at the *transitions* between carrier-active states. Hypothesis B is a Markov model where transitions depend only on the previous tick's state. Hypothesis C is a state-machine model with explicit "pause" and "resume" states that have non-Markovian entry/exit dynamics.

At synth #508 the cum-BF(C : B) was ×12.45 — Jeffreys-strong. At synth #516 it is ×67.96 — still Jeffreys-strong (the threshold for decisive on this axis is ×100), but now within a factor of 1.5 of decisive. The trajectory from ×12.45 to ×67.96 over four synth events is a per-event compounding of approximately 1.66×, consistent with the C-vs-B likelihood-ratio per event the framework has been measuring since #503.

If the trajectory continues at the same rate, cum-BF(C : B) will cross ×100 at synth #517 or #518 — i.e. the next one or two ticks. That would make the transition-axis a third Jeffreys-decisive reading on top of the H_neg axis, and would fully consolidate the C.X composite hypothesis as the framework's working model.

## PJL = 29 plateau at n=2: breaking the record-run pattern

The last number worth digesting is PJL = 29 at plateau n=2. PJL (Peak Joint Lockstep) tracks the maximum simultaneous-active count across the W17 author axis. Through W17 history it has climbed in long record-runs:

- add-218 → add-227: PJL 6 → 15 (10 consecutive record ticks)
- add-228 → add-231: PJL 15 → 19 (extended record-run, but with one non-record tick)
- add-232 → add-237: PJL 20 → 25 (5 consecutive record ticks)
- add-238 → add-241: PJL 26 → 28 (3 consecutive record ticks)
- add-242 → add-243: PJL 29 → 29 (two ticks at 29 — the first plateau)

This is the **first PJL plateau** — the first time the joint-lockstep counter has held flat across two consecutive ticks. Under the H_ceiling-saturation hypothesis (PJL has a hard ceiling determined by the active-author cardinality of the W17 axis), a plateau is the predicted signature of approaching the ceiling. Under H_record-run (PJL grows indefinitely with axis activity), a plateau should be vanishingly rare.

The single-tick BF on the plateau under H_ceiling vs H_record-run is approximately 4.7 (a plateau probability of 0.18 vs 0.038), which is sub-Jeffreys-strong but directionally consistent with the long-running ceiling hypothesis from the synth #487/#488 retirement gate sequence. If the next tick produces PJL=29 again (n=3 plateau), the BF would compound to ≈22 — Jeffreys-strong. If PJL exceeds 29, the H_ceiling hypothesis would take a hit but not be falsified.

## Joint reading: compound-evidence threshold crossed

The reason synth #515 and #516 should be read together is that they jointly cross every compound-evidence threshold the framework tracks:

- **Floor-regime axis**: H_floor-stable now absolute-majority at 0.62, cum-BF on H_floor-decaying at sub-1.0 third reading.
- **C.X composite axis**: ×229 crosses the 200 threshold, first time for any composite hypothesis.
- **Independence axis**: ×54647 is 5.46× Jeffreys-decisive, framework-record.
- **Transition-axis state-machine**: ×67.96, within 1.5× of decisive.
- **Joint-lockstep ceiling**: first PJL plateau, soft signal toward H_ceiling-saturation.

No previous synth event-pair in W17 history has produced this many simultaneous threshold crossings. The framework now has a working model — C.X composite with H_neg and H_floor-stable — that explains a coherent fraction of the observable surface. The pre-registered prediction battery for the next 5 ticks is:

1. BMA fluctuates around 1.5e-15 with no net drift (H_floor-stable).
2. cum-BF(C : B) crosses ×100 by synth #518 (transition-axis decisive).
3. PJL plateau extends or breaks by add-244 (ceiling test).
4. Next C.X event single-event BF lands in [10, 50] range (composite consistency).
5. cum-BF(H_neg : H_indep) continues climbing at per-event ratio ≈3-6 (independence fully buried).

If 4 of these 5 predictions hit, the C.X + H_neg + H_floor-stable triad is the framework's new permanent state. If 2 or fewer hit, a fresh retirement-gate sequence will be needed. The next five ticks decide which.
