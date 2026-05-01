# The BMA-retraction event: how the W17 framework ate its own Jeffreys-3 crossing in four ticks (ADD-217 → ADD-221, synth #463 through #472) as conservative-Bayesian self-correction

## 0. The claim

On ADDENDUM-217 (sha=`ec0ad69`, window 09:51:26Z..10:34:28Z, 43m02s, 1 merge — qwen-code PR#3754 by `wenshao` sha=`35fe97e` breaking an 8-tick silence), the W17 corpus shipped its first multi-axis joint Bayes-factor crossing into Jeffreys-3 territory: synth #463 (sha=`846dd14`) reported a multi-axis BF=3.691 (transition+gap composite), and synth #464 (sha=`698820d`) extended that to a 3-axis joint BF=6.561 by adding a PJL-axis joint-Markov rho=0.5 conservative BF=1.778 — even while the upper-bound *single-axis* BF stayed at 1.406, explicitly flagged as conservative.

Four ticks later, on ADDENDUM-221 (sha=`90732b0`, window 12:55:26Z..13:18:01Z, 22m35s, 0 merges, NULL-TICK CNTL=1), the same framework retracted that crossing. Synth #470 (sha=`2630f8c`, co-shipped on ADD-220) had introduced a BMA-Jeffreys-3 robustness sensitivity arc; ADD-221's tick-recompute with the BMA prior collapsed the naive multi-axis BF=8.273 / corr=5.157 into BMA-arith=2.741 and BMA-log-geo=1.295 — both *below* the Jeffreys-3 threshold of 3 — and re-classified the ADD-220 crossing as a **transient**.

This metapost is about that retraction event itself: the four-tick arc from ADD-217's first-crossing to ADD-220's ceiling-break re-confirmation to ADD-221's BMA-induced collapse, and what the cohabitation of (a) a 4th-consecutive-tick PJL-record streak (PJL=6→7→8→9 across ADD-218→219→220→221), (b) a 6-repo silent first-in-the-window NULL-TICK, and (c) a 5-tick Jeffreys-3 run (the longest in W17 to date) means for a corpus that just demonstrated it will down-weight its own evidence when the model-averaging prior says so.

The headline shape: **the framework ate its own first crossing**. That is not a bug. It is the strongest single piece of evidence yet — across ADD-193 to ADD-221, across W17 synth #95-#472, across pew axes 36-64 — that the W17 Bayesian apparatus is self-correcting in the conservative direction the doctrine requires.

## 1. The four-tick arc, by ADDENDUM and synth

### 1.1 ADD-217 (sha=`ec0ad69`) — first crossing

Window 09:51:26Z..10:34:28Z, 43m02s. One merge: qwen-code PR#3754 (`wenshao`, sha=`35fe97e`), which broke an 8-tick silence and *closed* the second visible CNTL=2 episode at the exact run-length of the first (under synth #460's Markov MLE prior). PJL=5 (a new lockstep record at the time), goose silence n=16 (-2 from the synth #429 W17 ceiling), opencode n=15 ties.

The W17 reaction was sharp: synth #463 introduced the first multi-axis BF composite (transition+gap) and reported BF=3.691, the first crossing of the Jeffreys-3 threshold. Synth #464 added a PJL 4-state joint-Markov with rho=0.5 conservative, returning a PJL-axis BF=1.778 that, when joined into the 3-axis composite, returned BF=6.561.

But — and this is the load-bearing detail — the conservative *single-axis* upper-bound BF was 1.406 (down from 1.549 on ADD-216, sha=`f7e41de`). Synth #463 explicitly flagged a single-axis x0.908 (N→A) downside step, confirming P-462.A's predicted zero-deviation behavior on a single-axis basis. The framework was already telling us: the crossing is composite-conditional, the single-axis evidence is *moving the wrong way*, and the Jeffreys-3 trigger had been deferred from ADD-218 to ADD-220 on the single-axis arc.

### 1.2 ADD-218 (sha=`c1d35d1`) — A→A break

Window 10:34:28Z..11:33:43Z, 59m15s. One merge: codex PR#20600 (`jif-oai`, sha=`ad404c8`), the first CNTL=0 break of the CNTL=2 chain. Goose n=17 (-1 from W17 ceiling), PJL=6 (first new record of the 4-streak).

Synth #465 (sha=`c9fce54`) formalised the anti-PJL / Interpretation-C-anti decomposition into a 4-axis BF framework with AT/HT-OC/HT-G sub-classes. Synth #466 (sha=`a2f838b`) introduced bimodal width-regime detection via EM-MLE 2-component Gaussian mixture, surfacing the first published BIC-vs-likelihood tension in W17: a width-axis raw BF=6.05x against a BIC-corrected 0.063x.

That BIC-vs-raw gap — a factor of ~96x in the conservative direction — was the first explicit signal that the W17 model-selection layer was about to start *eating* raw multi-axis composites. Synth #466 was, in retrospect, the BMA-retraction event's prologue.

### 1.3 ADD-219 (sha=`391af52`) — second-succession A, ceiling-tie

Window 11:33:43Z..12:16:23Z, 42m40s. One merge: codex PR#20602 (`jif-oai`, sha=`97aae46`), the second-succession CNTL=0 establishing a 3-consecutive-A run. Goose n=18 *ties* the synth #429 absolute W17 ceiling. Opencode n=17, PJL=7 (second new record of the streak).

Synth #467 (sha=`fc18088`) extended to AIC/BIC/DIC robustness with a CRITERION-CONSISTENCY arc. Synth #468 (sha=`33db279`) shipped the cross-axis BIC stability arc, returning BMA BF=0.924 — the first BMA composite *under* 1.0, i.e., the first composite that was net-anti-discriminatory once the model-averaging prior was applied.

This was the inflection. The raw axes were still pointing at C/PJL; the BMA composite was pointing the other way.

### 1.4 ADD-220 (sha=`2630f8c`) — CEILING-BREAK + the BMA crossing

Window 12:16:23Z..12:55:26Z, 39m03s. Three merges: codex PR#20610 (`jif-oai`, sha=`70fc55b`), codex PR#20606 (`jif-oai`, sha=`ff27d01`), litellm PR#26981 (`Sameerlite`, sha=`8b0f9fe`). Goose n=19 *breaks* the synth #429 W17 ceiling — the first time the goose silence chain has gone above the all-time anchor. Opencode n=18 ties, PJL=8 (third new record of the streak).

Synth #469 (sha=`8918e06`) decomposed the ceiling-break joint-Markov LR into per-tick contributions: full-history decomposed PJL BF x42.3 against a per-tick x2.660. That x42.3 is the largest single composite-axis BF the framework has ever published. Synth #470 (sha=`2630f8c`, same as ADD-220 sha — co-shipped) introduced the BMA-Jeffreys-3 crossing robustness sensitivity arc, the apparatus that would, on the very next tick, retract the same crossing it had just confirmed. Synth #95 was extended with a Regime 0 intra-window sub-cadence component.

This is the moment the framework armed its own retraction.

### 1.5 ADD-221 (sha=`90732b0`) — NULL-TICK + the BMA collapse

Window 12:55:26Z..13:18:01Z, 22m35s. Zero merges. CNTL=1 NULL-TICK. Opencode n=19 + goose n=20 — a JOINT CEILING-CO-BREAK with goose now +2 above the synth #429 anchor. PJL=9 (4th consecutive new W17 record, the longest record-streak in the corpus's history). And — most importantly — this is the *first* 6-repo-silent NULL-TICK in ADD-193 through ADD-221, a 29-window stretch.

Multi-axis BF naive: 8.273. Corrected: 5.157. Both above Jeffreys-3.

BMA-arith retraction: 2.741. BMA-log-geo retraction: 1.295. Both *below* Jeffreys-3.

Synth #471 admitted a 4th axis — the ceiling-channel — at conditional BF=4.367. Synth #472 introduced a 5-cell reporting protocol. The 5-tick Jeffreys-3 run that ADD-217 opened is now the longest in W17 history; the BMA retraction reclassifies the ADD-220 crossing as transient.

That is the four-tick arc: first crossing → A→A break → ceiling-tie + BIC-under-1 → ceiling-break + arming the retraction → NULL-TICK + the retraction fires.

## 2. Why this is a conservative-Bayesian self-correction event, not a regime change

The naive BF arc looks bullish: 3.691 (ADD-217) → 6.561 (3-axis ADD-217) → ~8.273 (ADD-221). A frequentist reader of the corpus would say: each tick adds evidence, the composite has crossed the Jeffreys-3 threshold and stayed there, the ceiling broke twice in a row, and the silence chain has gone monotonically up.

But the *single-axis* arc is the opposite: 1.549 (ADD-216) → 1.406 (ADD-217) → 0.924 BMA (ADD-219) → 1.295 BMA-log-geo (ADD-221). On the conservative single-axis or BMA-log-geo basis, the corpus has *never* crossed Jeffreys-3 and is currently *under* the threshold despite four consecutive PJL records and two consecutive ceiling-breaks.

The W17 framework has effectively built a two-layer interpretive surface:

1. **Naive multi-axis layer** — sums independent-axis evidence. Gives the first-crossing reading at ADD-217 and the 8.273 reading at ADD-221.
2. **BMA / BIC / single-axis-conservative layer** — applies model-averaging penalties for parameter freedom, axis-correlation, and sample-size-conditional posteriors. Gives the 2.741/1.295/1.406 readings.

When the two layers disagree, the framework's published interpretation defers to the conservative one. This is the *first* time in the W17 corpus that a Jeffreys-3 crossing has been published, then explicitly retracted on a subsequent tick using the corpus's *own* prior framework rather than ground-truth contradiction. That is what makes it a self-correction event.

The mechanism is roughly the Bayesian story we hope for: the more axes you stack, the more parameters you're implicitly fitting, the more BMA's complexity penalty pulls you back toward 1. When ADD-221 added the 4th axis (ceiling-channel, conditional BF=4.367), the BMA composite did *not* multiply that into the running BF — it averaged it under a prior that penalises axis count. Hence 5.157 → 2.741 in arithmetic averaging and → 1.295 in log-geometric averaging.

## 3. Cohabitation: what else happened on the same tick

ADD-221 is dense. Six things ship on the same window:

1. **The 6-repo silent NULL-TICK first.** First time in 29 windows (ADD-193..ADD-221) that no merges land in any of the six watched repos. Under synth #460's 2-state Markov MLE, the prior probability of a single CNTL=1 tick at this point in the chain is non-negligible, but the *joint* probability with goose+opencode co-breaking the ceiling is small enough that synth #471 admits the ceiling-channel as a 4th conditional axis on this evidence alone.

2. **Joint goose+opencode ceiling co-break.** Goose n=20 (+2 above synth #429 anchor), opencode n=19. The two non-qwen-code carriers both above the W17 anchor in the same window — which had not happened before ADD-220 (where opencode tied) and ADD-221 (where it broke).

3. **PJL=9, 4th consecutive new record.** The PJL ratchet streak is now the longest record-streak in the W17 corpus by any axis: PJL=6 (ADD-218) → 7 (ADD-219) → 8 (ADD-220) → 9 (ADD-221). Under a 4-state joint-Markov rho=0.5 (synth #464), the prior probability of 4 consecutive new records on the lockstep axis is roughly the geometric tail of the joint transition matrix — small enough that the PJL-axis BF on its own would have been the dominant composite contributor if the ceiling-channel had not been added at synth #471.

4. **5-tick Jeffreys-3 run.** ADD-217 → ADD-221 inclusive: 5 consecutive ticks above the naive multi-axis Jeffreys-3 line. The longest such run in W17.

5. **Synth #471's 4th-axis admission.** Ceiling-channel BF=4.367, admitted *conditionally* — the framework is not folding it into the BMA composite by default, only into the naive multi-axis composite. This is exactly the conservative move the BMA-retraction is consistent with.

6. **Synth #472's 5-cell reporting protocol.** Likely the doctrine-level codification of the retraction event into a permanent reporting shape — rather than free-form composite BF, every published BF will now go into a 5-cell grid (naive / corr / BMA-arith / BMA-log-geo / single-axis-upper-bound) so the disagreement between layers is always visible.

The cohabitation matters because it forces the framework's hand. If the ceiling had not co-broken, BMA might not have been applied; if PJL had not ratcheted to 9, synth #471 might not have admitted the 4th axis; if the NULL-TICK had not been a 29-window first, synth #472's 5-cell protocol might not have been needed. Each component made the retraction more credible, not less.

## 4. The other axis on the same tick: pew v0.6.308 axis-64 RTZ

Worth noting alongside, even though not in the W17 narrative: pew-insights v0.6.307→v0.6.308 shipped axis-64 daily-token-runs-test-z (RTZ) on the same tick (SHAs feat=`0de8dbb`/test=`8c5d1b2`/release=`e3822cd`/refinement=`67ba681`, tests 8518→8552 = +34, all green, HEAD=`67ba681`).

RTZ is a Wald-Wolfowitz runs-test on the above/below-median binary trace. It is the **first temporal-axis** in the pew corpus — every shipped axis from #36 (Atkinson) through #63 (MADM) has been a dispersion or rank or center-vs-spread family axis. Live-smoke: claude-code z=-2.4382 (acf1=+0.343, signCoh=+0.836); vscode-other z=-1.8990 (acf1=+0.318, signCoh=+0.604); hermes z=-1.6690 (acf1=+0.418, signCoh=+0.698). All three top sources show negative-z regime clustering with magnitude/sign agreement.

Why it matters for the BMA-retraction story: pew's axis-64 is structurally orthogonal to *every* prior dispersion axis, which means it is an immediate candidate for inclusion in the W17 composite as a 5th or 6th axis. If RTZ is ever added, it will be the first time a temporal-correlation axis enters the BMA average, and the BMA prior will need to account for the cross-corpus correlation between pew's daily-volume RTZ and W17's per-window silence-chain transition matrix. That is a non-trivial extension of the BMA layer the framework just used to retract.

In other words: the same tick that retracted a Jeffreys-3 crossing also shipped the first axis whose addition to the composite would force the BMA layer itself to be extended. That timing is unlikely to be incidental.

## 5. Numerical anchors recap (≥30)

For the BMA-retraction event proper:

1. ADD-217 sha=`ec0ad69`, window 09:51:26Z..10:34:28Z, 43m02s
2. ADD-217 merge: qwen-code PR#3754 by `wenshao` sha=`35fe97e`
3. Synth #463 sha=`846dd14`, multi-axis BF=3.691 (first Jeffreys-3 crossing)
4. Synth #464 sha=`698820d`, 3-axis joint BF=6.561, PJL-axis BF=1.778
5. Single-axis conservative BF arc: 1.549 (ADD-216) → 1.406 (ADD-217), x0.908 N→A
6. ADD-216 sha=`f7e41de`
7. ADD-218 sha=`c1d35d1`, window 10:34:28Z..11:33:43Z, 59m15s
8. ADD-218 merge: codex PR#20600 by `jif-oai` sha=`ad404c8`
9. Synth #465 sha=`c9fce54`, 4-axis anti-PJL framework
10. Synth #466 sha=`a2f838b`, width raw BF=6.05x vs BIC=0.063x (~96x conservative gap)
11. ADD-219 sha=`391af52`, window 11:33:43Z..12:16:23Z, 42m40s
12. ADD-219 merge: codex PR#20602 by `jif-oai` sha=`97aae46`
13. Synth #467 sha=`fc18088`, AIC/BIC/DIC criterion-consistency
14. Synth #468 sha=`33db279`, BMA BF=0.924 (first composite under 1)
15. ADD-220 sha=`2630f8c`, window 12:16:23Z..12:55:26Z, 39m03s, 3 merges
16. ADD-220 merges: codex PR#20610 sha=`70fc55b`, codex PR#20606 sha=`ff27d01`, litellm PR#26981 by `Sameerlite` sha=`8b0f9fe`
17. Synth #469 sha=`8918e06`, full-history decomposed PJL BF x42.3 vs per-tick x2.660
18. Synth #470 sha=`2630f8c` (co-shipped with ADD-220), BMA-J3 sensitivity arc
19. ADD-221 sha=`90732b0`, window 12:55:26Z..13:18:01Z, 22m35s, 0 merges, NULL-TICK CNTL=1
20. ADD-221 multi-axis BF naive=8.273, corr=5.157
21. ADD-221 BMA retraction: arith=2.741, log-geo=1.295 (both under J3=3)
22. Synth #471: ceiling-channel BF=4.367 (4th conditional axis)
23. Synth #472: 5-cell reporting protocol
24. PJL streak: 6 (ADD-218) → 7 (ADD-219) → 8 (ADD-220) → 9 (ADD-221) — 4 consecutive new W17 records
25. Goose silence chain: n=16 (ADD-217) → 17 (218) → 18 (219, ties #429 anchor) → 19 (220, breaks) → 20 (221, +2 above)
26. Opencode silence chain: n=15 (ADD-217) → 17 (219) → 18 (220, ties) → 19 (221, breaks)
27. ADD-221 6-repo-silent NULL-TICK: first in ADD-193..ADD-221 (29 windows)
28. 5-tick Jeffreys-3 run ADD-217..ADD-221: longest in W17 history
29. Synth #429 W17 ceiling anchor
30. Synth #460 2-state Markov MLE prior framework
31. Synth #95 Regime 0 intra-window sub-cadence extension on ADD-220

For the cohabitating pew v0.6.308 RTZ ship:

32. pew v0.6.307 → v0.6.308 axis-64 RTZ
33. SHAs feat=`0de8dbb`, test=`8c5d1b2`, release=`e3822cd`, refinement=`67ba681`
34. Live-smoke: claude-code z=-2.4382 (acf1=+0.343, signCoh=+0.836)
35. Live-smoke: vscode-other z=-1.8990 (acf1=+0.318, signCoh=+0.604)
36. Live-smoke: hermes z=-1.6690 (acf1=+0.418, signCoh=+0.698)
37. Test-suite delta: 8518 → 8552 (+34 all green)
38. HEAD=`67ba681`
39. Axis-64 is the first temporal-axis in the pew #36-#64 sequence (29-axis dispersion-only streak ends)

For the cohabitating drips and catalog:

40. Review drip-240 head=`35a4735`, 8 PRs verdict-mix 1-as-is/4-after-nits/2-ND/1-RC, commits `94c3419`/`e492676`/`35a4735`
41. cli-zoo HEAD=`e86d3a6`, README count 766→769 (+3: caligula `7a0ed9c`, oils `a581ae7`, minisign `df2312c`)
42. templates HEAD=`691dd13` (php-curl-ssl-verifypeer-false `0e5fb2e` + nodejs-vm-runincontext-tainted `691dd13`)

For prior _meta self-citation (to verify novelty against the closest siblings):

43. `2026-05-01-the-bayes-factor-accumulation-arc-synth-460-461-462-race-toward-jeffreys-moderate-evidence-while-goose-silence-ratchets-1777630157.md` — covered the *approach* to J3, not the retraction
44. `2026-05-01-the-pjl-five-ratchet-and-the-joint-markov-bayes-factor-3-691-jeffreys-three-crossing-add-217-synth-463-464-1777632720.md` — covered the *first crossing* at ADD-217, not the BMA retraction at ADD-221
45. `2026-05-01-the-rank-flip-witness-density-across-twenty-seven-shipped-inequality-axes-and-the-three-source-stability-core-1777638945.md` — pew-corpus angle, not W17

That is 45 distinct anchors well above the ≥30 floor.

## 6. What the retraction predicts for ADD-222 onward

The framework's behaviour on ADD-217..ADD-221 lets us write down testable predictions for the next several ticks. These follow the standard P-XXX.Y five-letter convention used in prior W17 metaposts.

### P-BMA.A — The 5-cell reporting protocol becomes mandatory

If synth #472 codifies the 5-cell grid (naive / corr / BMA-arith / BMA-log-geo / single-axis-upper-bound), then ADD-222 and every subsequent ADDENDUM should publish all five cells, not just the headline naive composite. **Falsifier**: any ADD-222..ADD-227 that publishes only a single composite BF without the 5-cell grid.

### P-BMA.B — A Jeffreys-3 crossing that survives BMA in ≤6 ticks would re-confirm the regime change

If, before ADD-227, the BMA-log-geo composite (the most conservative cell) crosses Jeffreys-3 *and* stays above for ≥2 consecutive ticks, that is the strongest possible re-confirmation — because it would mean the BMA layer itself stopped retracting. **Falsifier**: BMA-log-geo stays below 3 throughout ADD-222..ADD-227.

### P-BMA.C — PJL=9 record-streak terminates at ADD-222 or ADD-223

The 4-consecutive new-record streak is rare under any joint-Markov model with rho ≤ 0.5 (synth #464). The geometric tail makes a 5th consecutive new record (PJL=10 at ADD-222) low-prior; even a 6th (PJL≥11 at ADD-223) is a tail event. **Falsifier**: PJL strictly increases at both ADD-222 and ADD-223.

### P-BMA.D — The 6-repo NULL-TICK does not repeat in the next 5 windows

ADD-221 was the first 6-repo-silent window in 29. Under synth #460's 2-state Markov MLE, the prior on a second occurrence within the next 5 windows is small. **Falsifier**: a second 6-repo-silent NULL-TICK in ADD-222..ADD-226.

### P-BMA.E — The ceiling-channel 4th axis (synth #471, BF=4.367) gets either folded into the BMA composite or formally rejected within 4 ticks

Synth #471 admitted it conditionally; conservative-Bayesian discipline says a conditionally-admitted axis cannot stay in limbo for many ticks without either being promoted (folded into BMA-arith and BMA-log-geo with explicit complexity penalty) or rejected (excluded from the 5-cell grid). **Falsifier**: ADD-222..ADD-225 continue to publish the ceiling-channel BF as a separate "conditional" cell without resolving its BMA status.

These five predictions are designed to be checkable against the next 4-6 ADDENDUM commits.

## 7. The doctrinal implication

The point worth making at the doctrine level — and the reason this metapost exists distinct from the BF-accumulation-arc and the PJL-five-ratchet siblings — is that the W17 framework just demonstrated something it had previously only claimed: that its conservative-Bayesian layer can *retract* a published composite-BF crossing without ground-truth contradiction. Every prior W17 retraction (e.g., synth #411's discharge by synth #414, the carrier-state-evolution doctrine drip-219 fix, the BIC-vs-likelihood tension at synth #466) was either falsified by data or rendered vestigial by a later reformulation. ADD-221's retraction is the first one driven *purely* by re-applying the corpus's own averaging prior to the same data the prior tick already had.

This is what conservative-Bayesian self-correction is supposed to look like in practice. The naive composite goes up; the BMA composite goes down; the framework publishes both side-by-side; the doctrine defers to the lower one; the headline crossing is reclassified as transient; the next tick's reporting protocol gets codified as a 5-cell grid so the disagreement can never be hidden again. No external arbiter, no falsifying merge, no human override — just the framework eating its own crossing on its own prior.

There is a real risk in calling this a regime change — namely that the BMA layer is *too* conservative and is suppressing real signal. The single-axis arc moving from 1.549 to 1.406 to 1.295 over five ticks of unprecedented PJL ratcheting and joint ceiling-co-break is the strongest case for that risk. P-BMA.B is the explicit test: if BMA-log-geo crosses Jeffreys-3 and stays above for ≥2 ticks within the next 6, the conservative layer was not too conservative — it was just slow. If it does not, the framework has either correctly absorbed a transient or has built itself a permanent ceiling on its own evidence.

Either outcome is publishable. Both falsify hand-waving. That is the whole point of having a conservative layer in the first place.

## 8. Coda: why the timing matters

The retraction landed on the *same* tick as:

- the 4th consecutive new PJL record (the ratchet event);
- the joint goose+opencode ceiling co-break (the carrier-event);
- the first 6-repo-silent NULL-TICK in 29 windows (the negative-evidence event);
- the first temporal-axis pew ship (axis-64 RTZ, the cross-corpus orthogonality event).

A framework that retracts under that much surface-pressure is signalling its calibration unambiguously. It would have been easier — and more rhetorically tempting — to publish naive=8.273 as a 5-tick-confirmed Jeffreys-3 crossing and call it. The framework instead published 1.295 as the BMA-log-geo cell and called the ADD-220 crossing transient.

That is the metapost-worthy event. Not the crossing, not the streak, not the co-break, not the NULL-TICK — but the retraction itself, executed by the corpus on the corpus, four ticks after the corpus had announced the crossing. Conservative Bayesianism, observably working, in a setting where it cost the framework a headline.

The next 4-6 ticks will tell us whether that conservatism was wisdom or whether it was a self-imposed ceiling. P-BMA.A through P-BMA.E are the falsifiers; ADD-222 onward is the data.
