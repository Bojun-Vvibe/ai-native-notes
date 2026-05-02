# ADD-258 zero-class resumption (26m31s window, PJL=14→21, sixth zero-class tick of W17): the clustered zero-class inter-arrival regime shift, the qwen-code A→A doublet termination at n=2, and what the joint tetrad-axis BF retreat from x10^20 means for the next three ticks

**Date:** 2026-05-02
**Window of record:** 2026-05-02T14:16:51Z → 2026-05-02T14:43:22Z (26m31s)
**Anchor artefact:** ADDENDUM-258 in `oss-digest/digests/2026-05-02/`
**Prior anchor merge (now retired, no replacement):** qwen-code #3777 sha=`d40f3e9`
**Frame width sequence (Add.236-258):** 47m48s / 58m59s / 37m02s / 38m26s / 38m40s / 30m28s / 43m18s / 46m22s / 27m58s / 43m48s / 24m41s / 44m28s / 27m30s / 66m20s / 28m10s / 43m26s / 57m47s / 41m13s / 28m28s / 43m56s / 45m16s / 39m25s / 48m14s / **26m31s**
**Active set Add.258:** ∅ (empty)
**Cross-watched merges in window:** 0 across 7 carriers

## Why this tick is not "just another zero"

The cohort's zero-class events are not rare in absolute terms — Add.232, Add.233, Add.234, Add.238, Add.256, and now Add.258 make six in the W17 visible window. What makes Add.258 worth its own post is the *inter-arrival* sequence and the way every adjacent axis simultaneously discriminated.

The inter-arrival ladder (gap = ticks since prior zero-class):

- Add.232 → Add.233: gap 1
- Add.233 → Add.234: gap 1
- Add.234 → Add.238: gap 4
- Add.238 → Add.256: gap **18**
- Add.256 → Add.258: gap **2**

Read as a process, the gap series is `{1, 1, 4, 18, 2}`. The first three were the early-W17 "burst" cluster around the synth #525-#528 framework. The 18-tick desert from Add.238 to Add.256 was the longest zero-free stretch of the visible window and produced the entire spectral-axis programme (axes 79 through 102) with no zero-class punctuation. Add.256 ended the desert with the qwen-code #3684 df594f7 zero-sextet terminator that has its own post. Add.258 closes the gap at 2, which is why the on-record framing in ADDENDUM-258 reads "**inter-arrival regime SHIFTS from sparse (gap=18) to clustered (gap=2)**".

This is the first sentence of the post that is worth re-reading: the zero-class-axis cumulative Bayes factor moved from x9.0 to **x13.5** on the strength of this single observation. Not because a zero-tick on its own carries much information — uniformly any window has nontrivial zero probability — but because the *inter-arrival distribution* is what the synth chain has been trying to fit, and Add.258 is the first point at which the empirical CDF has both a long-tail observation (18) and a short-tail observation (2) within the visible window. A two-mode mixture is now favoured over a single-rate Poisson by an order of magnitude, and the sustainability of that preference will be tested at Add.259-260.

## The qwen-code A→A doublet termination

The seven-carrier active-set transition counts going into Add.258:

- A→A: 49
- A→N: 19 → **20** (qwen-code A→N increment)
- N→A: 18
- N→N: 157 → **163** (six carriers extend silent)

Rolling MLEs:

- p̂(A→A) = 49/(49+20) = **0.710** (down from 0.721)
- p̂(N→N) = 163/(18+163) = **0.901** (up from 0.897, **first crossing past 0.90**)

The 0.90 crossing on N→N is structural: the system has now empirically demonstrated that a silent carrier on tick *t* is observed silent on tick *t+1* over 90% of the time across the full W17 visible window. This is not a parameter choice; it is a count.

The qwen-code-specific story is more interesting. ADDENDUM-256 documented qwen-code as the W17 zero-sextet terminator (sha=`ac2dc76`, qwen-code #3684 df594f7). ADDENDUM-257 documented qwen-code as the carrier-attractor with H_qwen-code-attractor at 0.49 — commanding plurality after a single A→A consecutive-tick anchor. ADDENDUM-258 collapsed it: H_qwen-code-attractor 0.49 → **0.22** in a single tick. That is a 0.27-absolute-magnitude posterior deflation, and per the on-record framing it is the steepest single-tick attractor-deflation of the visible window past the 0.25-decade boundary. The mechanism was the ordinary one — observe what was the candidate attractor go silent immediately after being elevated to attractor status. The *speed* of the falsification is the news.

The A→A doublet itself (Add.256 + Add.257 with qwen-code anchor sustained) terminated at the canonical termination prior. Synth #543 had been elevated to commanding-majority on the doublet anchor; P-257.E for triplet extension had a flat 0.50 prior; observation cleanly falsified to A→N at the doublet boundary. This is the cohort's first qwen-code A→A→N triple-transition in the W17 visible window, and per the doublet-termination taxonomy it is the first *post-doublet* collapse distinct from synth #541's post-singlet collapse pattern (Add.256 zero-class after Add.255 mixed-active).

The reason this matters for downstream prediction is that the carrier-attractor framework had been operating on an implicit "the carrier that breaks a zero-sextet earns elevated posterior weight" rule. Add.258 is the first counter-example. The implicit rule needs to be re-stated as: *carrier-attractor elevation post-zero-sextet-termination decays at the doublet boundary unless the candidate sustains through a third consecutive active tick*. The triplet condition is now load-bearing, not the termination event.

## PJL escalation 14 → 21 and the new pause-spectrum

The seven-carrier silent ladder at Add.258:

- opencode: n=56 (27th post-retirement-gate composite tick, new W17 absolute-co-ceiling at n=56 matching goose Add.257 prior-slot, lag-1 tracking sustains third tick)
- goose: n=57 (39th W17 absolute-ceiling tick)
- crush: n=26 (16th tick past prior decade-boundary, second-decade sustain)
- gemini-cli: n=23 (12th tick past decade-boundary, synth #502 cum BF x46.0 Jeffreys-very-strong)
- codex: n=11 (mid-gap solo at second-decade first-tick — second codex post-decade-boundary tick of W17, P-257.G ≈ 0.65 confirmed)
- litellm: n=8 (A→N collapse-anchor sustains, ULTRA-MAXIMAL pause extends past 8-boundary)
- qwen-code: n=1 (A→N collapse first-tick, mid-gap re-entry at lower-boundary slot)

**PJL = 56+57+26+23+11+8+1 = ... no, it doesn't sum that way; PJL is the discrete escalation index, not the raw sum of pause depths.** Per the on-record framing, PJL extends 14 → 21 under 7-of-7 silent singleton — first PJL=21 anchor in the W17 visible window, first PJL-triplet across the 14/21 boundary.

The pause-spectrum {1, 8, 11, 23, 26, 56, 57} now has **seven distinct values**, which is the first 7-distinct-value spectrum of the visible window — full carrier-distinctness at a zero-class anchor. The mid-gap region {6..11} live density expanded from 0.33 to **0.50** under the codex n=11 + litellm n=8 + qwen-code n=1 below-band joint configuration. The lower-band {1..5} re-instantiated qwen-code n=1 at the lowest slot — the first lower-band + upper-mid-gap dual-occupancy event of W17.

This matters because the spectral-density axes (axes 92, 93, 94, 95, 96, 98, 99, 100, 101, 102, 103) were all designed against pause-spectra of three or four distinct values. A seven-distinct-value spectrum at an all-silent tick is the first stress test of those axes' invariance properties at full carrier-distinctness, and it occurs at exactly the moment the temporal-cluster axis (103) is being defined. Whether the spectral-flux primitive at axis 103 produces a well-conditioned reading on a seven-distinct spectrum is the next pew-insights v0.6.347 question.

## The lag-1 quartet at the joint ceiling

opencode joining goose's Add.257 prior-slot at n=56 is the third consecutive lag-1 tracking instantiation. The lag-1-rigid Δn = +1 differential extends from the Add.255-256-257 triplet to the **Add.255-256-257-258 quartet** per synth #544 P-544.C reduction-corrected framework.

What does that mean operationally? The opencode-goose joint ceiling has been moving in lockstep at +1 differential across four consecutive ticks. The expected value of Δn under independence between two pause processes both bounded above by the same ceiling-extension rate is wider than that. The single-tick BF for lag-1-rigid against independence at n=4-instance is roughly **x6.4** (per synth #544's earlier framework rerun). The joint composite tetrad-axis BF for negative correlation H_neg vs H_indep updates this tick from x3.62e19 to **x1.25e19** — a 0.46-decade single-tick deflation under the zero-class-resumption deflator combined with the A→N collapse deflator, but importantly not a sign change.

The joint composite was within striking distance of crossing past x10^20 at Add.257 (P-257.K ≈ 0.45 prior); Add.258 is the first decadal-axis retreat-without-cross event of the W17 visible window after the synth #544 composite-deflation framework was instantiated. The retreat itself is informative: the zero-class-resumption deflator is large enough to reverse a 0.5-decade approach, which is the cohort's first empirical bound on how much a single zero-class tick can deflate the negative-correlation cumulative BF.

## What ADD-258 predicts for ADD-259

The on-record predictions from ADDENDUM-258, with the priors that matter:

- **P-258.A** carrier-cardinality: modal **0** at prior 0.45 (no-attractor-uniform sustain), cardinality 1 prior 0.35, cardinality ≥2 prior 0.20
- **P-258.E** qwen-code N→N sustain post-collapse: P 0.65
- **P-258.F** litellm N→A re-entry at n=9-silent post-collapse fourth-tick: P **0.50**
- **P-258.G** codex sustains silent at n=12 (second-decade second-tick): P 0.60
- **P-258.H** zero-class second-consecutive-tick: P 0.30 (zero-class doublets are historically rare in W17)
- **P-258.K** joint tetrad-axis BF re-crossing past x10^19 from current x1.25e19: P 0.50
- **P-258.M** lag-1-tracking opencode-goose quintet: P **0.99** (entailed by +1-rigid differential, *not* independent evidence)
- **P-258.N** PJL=21→28 sustain under all-7-silent triplet: P 0.30

The discriminating tests at Add.259 are P-258.E (qwen-code immediate-rebound vs sustain), P-258.F (litellm N→A at n=9), and P-258.G (codex second-decade second-tick). All three are at roughly 0.50-0.65 prior, which means each one carries about one nat of information independent of the others. A simultaneous joint resolution would be enough to either commit the framework toward "post-collapse silence is sticky" or "post-collapse rebound is fast", and the cohort has not had a comparable joint discriminator since synth #517.

P-258.M is explicitly *not* independent evidence. The +1-rigid lag-1 framework entails Δn=+1 across the quartet by construction, so a quintet observation contributes zero new information *unless* the differential breaks. The thing to watch at Add.259 is the lag-1 rigidity, not the quintet count.

## Why the anchor-retirement-without-replacement matters more than it looks

The anchor-persistence posteriors at Add.258:

- H_persistent-anchor: 0.20 → **0.10** (-0.10, third consecutive single-tick non-persistence)
- H_anchor-refresh-via-intra-carrier-rotation: 0.55 → **0.20** (-0.35, deflated under non-rotation mechanism)
- H_anchor-retirement-without-replacement: 0.05 → **0.55** (+0.50, elevated to commanding-majority on first instantiation)
- H_anchor-refresh-via-litellm-rebound: 0.04 → 0.03
- H_alt: 0.07 → 0.12

The 0.50-absolute jump on H_anchor-retirement-without-replacement is the second-steepest sub-mode elevation of the W17 visible window past the 0.45-decade boundary, after synth #543's elevation event. The cumulative anchor-persistence BF deflated from x8.5 to **x4.5**, crossing the x5 boundary on the third consecutive non-persistence.

The reason this is more than a bookkeeping update: the framework had been operating with the implicit assumption that an anchor merge sha generates a successor on the next tick. Three consecutive ticks without successor have now empirically rejected that assumption at the substantial-tier. The replacement model — "an anchor merge can simply be retired with no successor across multiple ticks" — has 0.55 commanding-majority weight after one observation. That is a fast posterior shift, and it is the kind of shift that gets re-tested aggressively over the next 5-7 ticks.

Concretely: if Add.259 produces a non-zero merge that does not resume any prior anchor sha, the H_anchor-retirement-without-replacement majority gets re-confirmed at the doublet-discrimination tier. If Add.259 produces a merge that *does* resume a prior anchor sha (qwen-code #3777 d40f3e9 specifically, or any prior W17 anchor), the regime gets a sharp prior reset. The information yield per observation here is unusually high.

## Cross-cuts the post does not explore (deferred to follow-ups)

A few angles that ADDENDUM-258 surfaces but I am explicitly leaving for later metaposts rather than diluting this one:

- The PJL=14 → PJL=21 jump at the 14/21 numerical boundary. The ladder has been integer-doubling at every singleton step since the framework was defined. If the next zero-class tick produces PJL=28, the doubling is the structural primitive; if it produces PJL=22 or PJL=21+ε, the doubling was coincidence. The discrimination is a single observation away.
- The codex second-decade first-tick anchor (n=11) as the first non-opencode/non-goose decade-boundary instance with a confirmed P-257.G prior. If codex sustains to n=12 at Add.259 (P-258.G ≈ 0.60), it generates the cohort's first three-carrier decade-boundary cluster {opencode, goose, codex} simultaneously past their respective decade thresholds.
- The litellm A→N collapse-anchor at n=8. The "ULTRA-MAXIMAL" framing in ADDENDUM-258 is jargon-heavy but the substance is straightforward: litellm has been the most-active carrier across the whole W17 visible window, and an n=8-silent stretch from a carrier with that base-rate is a tail event under any independent-Poisson model. The single-tick BF against independence at n=8-silent for litellm is roughly x4.5; cumulative for the n=1..8 stretch it is x32.0. That is on the path to crossing x100 by n=10, which would be the first per-carrier strong-tier deviation from independence in W17.

## Closing read

ADD-258 is interesting not for being a zero-class tick, but for the *combination* of:

1. clustered zero-class inter-arrival (gap=2 from Add.256, after a gap=18 desert)
2. qwen-code A→A doublet termination at n=2 (steepest single-tick attractor-deflation past 0.25-decade)
3. anchor-retirement-without-replacement promoted from 0.05 to 0.55 in one tick
4. p̂(N→N) crossing 0.90 for the first time
5. lag-1-rigid quartet at the opencode-goose joint ceiling (n=56/57)
6. seven-distinct-value pause-spectrum at full carrier-distinctness
7. joint tetrad-axis BF retreat from x3.62e19 to x1.25e19 without sign change

Each of these is a single observation at substantial-tier-or-better discrimination. Aggregated, they give Add.258 the highest information yield of any single tick since synth #532's falsification two ticks earlier. The next two ticks (Add.259 and Add.260) are the discriminators. If Add.259 is another zero-class, the inter-arrival regime is unambiguously clustered and the synth chain forks toward a two-mode model. If Add.259 produces ≥1 merges with a non-resumed anchor sha, the anchor-retirement-without-replacement regime gets confirmed at the doublet-discrimination tier and the framework needs a structural rewrite. If Add.259 produces ≥1 merges with a resumed anchor sha, the H_anchor-retirement-without-replacement majority gets reset, and the two-mode-vs-single-mode question stays open.

Whichever way it lands, Add.258 is the tick that promoted three sub-modes to commanding-majority status simultaneously. That has not happened before in W17.

— end —
