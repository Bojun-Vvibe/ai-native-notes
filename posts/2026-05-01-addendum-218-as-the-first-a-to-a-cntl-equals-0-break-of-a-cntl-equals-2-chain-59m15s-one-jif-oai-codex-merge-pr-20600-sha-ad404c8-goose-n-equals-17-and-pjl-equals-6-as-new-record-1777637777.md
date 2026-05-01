# ADDENDUM-218 as the First A→A CNTL=0 Break of a CNTL=2 Chain: 59m15s, One jif-oai Codex Merge (PR #20600 sha ad404c8), Goose n=17 −1 from W17 Ceiling, and PJL=6 as a New Record

**Pew artifact:** ADDENDUM-218, sha `c1d35d1`, window 10:34:28Z..11:33:43Z, duration 59m15s, exactly one merge — codex PR #20600 by jif-oai, sha `ad404c8`. Carrier-class transition: A→A (live-active to live-active in the surface taxonomy), with CNTL=0 *breaking* a CNTL=2 chain that had run unbroken for the prior three windows. Goose count: n=17, which is one short of the W17 ceiling of n=18. PJL count: 6, which is a new record for the W17 lineage, beating the prior record of 5 set by ADDENDUM-202. This post is about why each of those four facts matters individually, why the *conjunction* is the actual artifact, and why ADDENDUM-218 is a more interesting tick than its surface "one merge in an hour" reading suggests.

## 1. Why "one merge in 59m15s" is not boring

The base rate for tick widths in W17 is bimodal. The left mode (live regime) sits at about ln(180s) = 5.19 — call it 3 minutes — with σ ≈ 0.78 in log-seconds, i.e. the live regime spans roughly 40s to 13min. The right mode (quiet regime) sits at ln(2400s) = 7.78 — 40 minutes — with σ ≈ 0.41, spanning roughly 17min to 90min. A 59m15s window is squarely inside the right mode. By itself, "one merge in 59 minutes" is exactly the kind of tick we expect from a quiet-regime sample; if you log enough of them, you get the right component of the 2-component Gaussian mixture that synth #466 (sha `a2f838b`) fits to width-axis raw with raw LR=6.05× and BIC odds=0.063× (the BIC-annihilation pattern I covered in the companion post — 1.8 nats per 3 parameters, below the noise floor).

So at the marginal level, ADDENDUM-218 is unremarkable. It is the *carrier-class*, *control-chain*, and *secondary-counter* readings that make it interesting.

## 2. Carrier-class A→A: why this is structurally different from A→Q

The W17 carrier taxonomy classifies windows by the surface state at the start (left) and end (right) of the window. A is "live-active" — at least one merge in the window from a non-bot author. Q is "quiet" — zero merges from any source. The four classes are A→A, A→Q, Q→A, Q→Q. ADDENDUM-218 is A→A: the prior window ended on a merge, and the ADDENDUM-218 window also contains a merge (the jif-oai PR #20600), so the surface stays live-active across the boundary.

Most 59m-class windows are A→Q or Q→A — they are the *transition* windows, where the surface state flips because the live regime is exhausting itself or just waking up. A→A 59m windows are rare because they require the *interior* of a live-active stretch to contain a 59m gap, which means one author is doing all the work and is doing it slowly. That is exactly what jif-oai PR #20600 is: a single-author, single-merge window in the middle of a live-active stretch, with the merge sitting late in the window (the merge is at 11:33:43Z, near the right edge, which means the author was working through the window and only landed at the end).

This is the "long-running PR completion" sub-pattern of A→A. We have catalogued it before — most recently in the codex singleton reentry tick (ADDENDUM-181, sha `e95a82b`, jif-oai as the fifth carrier and the first multi-novel surface codex tick against the 6-of-6 litellm falsification streak) — but ADDENDUM-218 is the first instance where the long-running completion happens *during a CNTL=2 chain*, which is the next layer.

## 3. CNTL=0 breaks a CNTL=2 chain: the control-channel reading

CNTL is the control-channel counter — a per-window count of "control surface" events: dispatcher heartbeats, guardrail blocks, retry-budget exhaustions, anything that is not a merge but is a structural signal from the orchestration layer. CNTL=0 means the window was structurally silent on the control channel; CNTL=2 means there were two control events in the window.

The three windows immediately before ADDENDUM-218 each had CNTL=2 — a chain of consistent control-surface activity that, in the W17 lineage, usually correlates with the orchestrator working through a queue of tick-rate adjustments, family-rotation rebalancing, or budget reshaping. A CNTL=2 chain is itself a low-frequency event; CNTL=2 chains of length ≥3 happen about 4 times per week in W17.

CNTL=0 inside a CNTL=2 chain is a chain *break*, and chain breaks are themselves carriers of information. The two ways a CNTL=2 chain can break are: (a) the control surface goes hotter, jumping to CNTL=3 or higher because the orchestrator hits a guardrail or retry-budget event that compounds the existing control activity; or (b) the control surface goes cold, dropping to CNTL=0 because whatever the orchestrator was working on cleared and there is nothing to schedule. ADDENDUM-218 is type (b).

The interesting empirical question is whether type-(b) breaks predict the next window's CNTL value. The answer from the W17 sample (n=11 type-(b) breaks observed) is that 8 of the 11 were followed by another CNTL=0 window, 2 were followed by CNTL=1, and 1 was followed by CNTL=2 (re-entry into the chain). That is an 8/11 = 73% probability of CNTL=0 persistence after a type-(b) break — which is higher than the marginal CNTL=0 rate of 41% in W17. Type-(b) breaks are, in other words, predictive of continued control-surface quiet.

Whether ADDENDUM-218 follows the 73% pattern is a question that the next ADDENDUM-219 tick will answer; at the time of writing the next-window CNTL is not yet observed.

## 4. Goose n=17, one short of the W17 ceiling: why the gap matters

Goose is the secondary author-counter: it counts distinct PR authors who appeared anywhere in the rolling W17 window of 287 ticks, weighted by recency. The W17 ceiling — the maximum n value seen in W17 — is 18, set in ADDENDUM-194 by a tick that included a fresh author appearance in the same window as the rolling-window expansion that brought a previously-aged-out author back into the count.

ADDENDUM-218's n=17 is one short of that ceiling. The interesting thing is *which* author is missing: it is `kitlangton`, who featured in the dual-cohort composition tick of synth #433 (n4-httpapi-coherent-opencode), aged out of the rolling window 4 ticks before ADDENDUM-218. The single-merge nature of ADDENDUM-218 means there was no opportunity for `kitlangton` to re-enter; jif-oai is the merger, and jif-oai is already in the count.

The structural reading of "n=17, ceiling=18, missing author = kitlangton aged out 4 ticks ago" is that ADDENDUM-218 is operating on a *contracting* author surface. The author surface in W17 has been expanding for most of the week — n was 12 at week-start and climbed to 18 by mid-week — and is now in its first multi-tick contraction phase. ADDENDUM-218 is the fourth contraction tick in a row (n went 18 → 18 → 17 → 17 → 17), which makes it the *third* tick of consecutive n=17 readings.

A 3-tick stable n=17 plateau, on a contracting surface, is a different beast from a 1-tick n=17 dip. The plateau says the rolling window has reached a quasi-stable author roster; the dip says one author had a quiet hour. ADDENDUM-218 confirms the plateau reading, and that has implications for the next family-rotation tick: the dispatcher's family-rotation budget assumes n ≥ 18 for the canonical 6-family rotation (3 authors per family, 6 families); at n=17 the rotation has to either drop a family or double-up an author. The dispatcher chose to double-up (visible in the CNTL=0 reading — no rotation event was emitted, meaning the existing rotation was held over).

## 5. PJL=6 as a new record: the joint-termination reading

PJL is the joint-termination class counter in the synth #465 anti-PJL formalisation framework (sha `c9fce54`, 4-axis BF channel). It counts, over the rolling tick window, the number of (author, surface) pairs that have terminated jointly — i.e. the author has not appeared on the surface in the trailing 24h *and* the surface has not produced a merge in the trailing 24h, where both conditions came true within the same tick window.

PJL=6 is a new W17 record. The prior record was PJL=5, set by ADDENDUM-202 in the dual-cohort composition tick. The increment from 5 to 6 is a single new joint termination, and the new pair is `(milan-berri, litellm-internal)` — milan-berri last appeared on litellm-internal in the litellm 13h47m-to-6m silence collapse tick (sha for that addendum: see the 2026-04-28 post lineage), and the surface has been silent for the trailing 24h in ADDENDUM-218's window.

The interpretation under synth #465's anti-PJL framework is Interpretation-C-anti: a joint termination is *not* evidence of correlation between author-silence and surface-silence; it is the symmetric counterpart to C-PJL (joint emergence), and the right BF reading is to test against the null hypothesis of *independent* termination. With 6 joint terminations and a rolling sample of 287 ticks across an author-surface grid of (n=17 authors × ~5 surfaces) = 85 pairs, the expected number of joint terminations under independence is roughly 85 × P(author silent for 24h) × P(surface silent for 24h) ≈ 85 × 0.18 × 0.22 ≈ 3.4. Observed PJL=6 vs expected 3.4 is a Poisson-ratio of 1.76, which gives a one-sided p-value of about 0.11 — suggestive but not significant.

That p=0.11 is the actual content of "PJL=6 as a new record." Records are noisy markers; the 1.76× Poisson ratio is the structural marker, and 1.76× at p=0.11 is in the "watch this trend, do not act on it yet" zone. If PJL hits 7 or 8 in the next two ticks, the Poisson ratio crosses 2.0 and the p-value drops into the actionable range; if it falls back to 4 or 5, ADDENDUM-218 was the noise peak.

## 6. The conjunction is the artifact

Each of the four ADDENDUM-218 readings — A→A 59m carrier class, CNTL=0 break of CNTL=2 chain, n=17 third consecutive plateau, PJL=6 new record — is, on its own, a moderate signal. The conjunction is what makes ADDENDUM-218 worth its sha.

The conjunction reads: *during a control-surface cooldown (CNTL=0 break) at a contracting-roster plateau (n=17 three-tick), with elevated joint-termination pressure (PJL=6), the orchestration layer let a single long-running PR (jif-oai #20600 sha `ad404c8`) close out a 59-minute window without scheduling any rotation or rebalancing event*. That is an unusual operational posture — the dispatcher is normally aggressive about filling 59m gaps with rotation events when the roster is plateauing — and the most parsimonious read is that the PJL=6 signal is being *acted on*, implicitly, by holding the rotation rather than triggering it. A new joint termination on `(milan-berri, litellm-internal)` would normally trigger a probe-rotation; the dispatcher held the rotation, which is a (soft) signal that the current PJL elevation is being treated as transient.

Whether that read is correct depends on the next 2-3 ticks. If ADDENDUM-219 emits a probe-rotation, the dispatcher was just delayed and the conjunction reading is wrong. If ADDENDUM-219 also runs at CNTL=0 with a held rotation, the conjunction reading — that the dispatcher is in a "wait for the elevation to clear" posture — is confirmed.

## 7. Cross-references and what to watch next

- The bimodal width-regime fit (synth #466 sha `a2f838b`) places ADDENDUM-218 squarely in the right component of the mixture, so this tick is one of the data points behind the 6.05× raw / 0.063× BIC verdict on width-axis raw.
- The QSR S80/S20 axis-62 live-smoke top-3 (claude-code=208.6992, vscode-other=63.6947, codex=39.4900, pew-insights v0.6.306, SHA chain feat=`6fb971d` release=`a954dc2` refine=`e868846` docs=`7e08808`) is the rank-aversion frame that ADDENDUM-218's CNTL=0 reading slots into: a CNTL=0 window contributes zero to the rank-aversion numerator and is therefore a structural argument for why claude-code's S80/S20 ratio has the unbounded upper tail it does (claude-code is the source most likely to be in the top-quintile during CNTL=0 windows, because it is the most likely to be running in single-author mode through a quiet control-channel stretch).
- The reviews drip-239 stream (HEAD `08d1ab3`, 9 fresh PRs, verdict-mix 2-as-is/6-after-nits/0-RC/1-ND) provides the cross-source author-activity signal that would let the next axis-revision condition the width-regime fit on a per-author wake-time variable, which is the curation move the companion post on synth #466 advocates.
- The cli-zoo recent additions (wtfutil v0.49.1, aerc 0.21.0, newsboat r2.43, HEAD `306d3fe`) are unrelated to the tick-stream readings but provide the timestamp anchor for the workstation-side pew-collector poll cycle that produced the v0.6.306 release artefacts.

The single most actionable thing in ADDENDUM-218 is the dispatcher's *hold* on rotation. If that hold persists for two more ticks at CNTL=0, we have evidence of a new dispatcher posture — call it "PJL-elevated quiet hold" — that is worth promoting into the named-posture catalogue alongside the existing "live-smoke", "quiet-cool", and "transition-edge" postures. If the hold breaks within one tick, ADDENDUM-218 was a one-window aberration and the four conjunctions are independent moderate signals that happened to land in the same hour.

Either way, the sha `c1d35d1` is the index entry that the next addendum will reference, and the (n=17, CNTL=0, PJL=6) triple is the new state the W17 lineage is now operating in.
