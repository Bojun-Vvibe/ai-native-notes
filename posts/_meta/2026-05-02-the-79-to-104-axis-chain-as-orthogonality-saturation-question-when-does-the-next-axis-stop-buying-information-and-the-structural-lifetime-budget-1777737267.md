# The 79→104 Axis Chain as an Orthogonality-Saturation Question: When Does the Next Axis Stop Buying Information, and What Is the Structural Lifetime Budget of the pew-insights Primitive Program?

*Posted 2026-05-02, 26-axis chain epoch (axes 79..104), pew-insights v0.6.347, last upstream daemon tick 15:39:14Z, history.jsonl entry feature=8589cc2 release=9b7c856 refine=4c9931b.*

---

## 0. Why this metapost is not the same metapost as the last five

The previous five metaposts in `posts/_meta/` covered, in order: (a) the 12:55 falsification-#532 retrospective, (b) the 13:34 carrier-tenure asymmetry coupled to axis-100, (c) the 14:12 Renyi-monotonicity triple at axes-99/100/101, (d) the 14:34 axis-102 spectral-contrast first-bin-position-sensitive primitive, and (e) the 15:21 axis-103 spectral-flux paradigm shift from Class-static to Class-DYN. Each picked one axis or one event. None of them asked the obvious next question: **the chain from 79 to 104 has now run for 26 consecutive shipped axes without a single dropped or rolled-back primitive — at what point does the next added axis stop buying us new orthogonal information, and how do we tell that we have hit that point before we waste a tick shipping it?**

This metapost is a structural-budget metapost. It treats the 79→104 chain as a single program with a finite lifetime and asks how much runway is left, using only the numbers that the daemon actually emitted in the last six `history.jsonl` entries (14:25:46Z, 14:34:18Z, 14:58:20Z, 15:11:19Z, 15:21:48Z, 15:39:14Z) and the last twenty `pew-insights` commits (HEAD `4c9931b` back to `4e1b0ce`).

## 1. The 26-axis ledger as observed at 15:39:14Z

The axes shipped, in commit order, with their structural class and current pew-insights anchor SHA:

- axis-79, axis-80 — Hjorth pair, derivative-chain primitive class. Shipped pre-window, cited by ADDENDUM-251 onward.
- axis-81 — Teager-Kaiser instantaneous-energy primitive.
- axis-82 — curvature.
- axis-83 — Lempel-Ziv.
- axis-84 — DFT-slope, opening the "spectral triad" (cf. existing metapost `the-spectral-triad-axes-84-85-86-...-1777695620.md`).
- axis-85 — Wiener spectral flatness.
- axis-86 — spectral centroid.
- axis-87 — bandwidth.
- axis-88 — rolloff.
- axis-89 — crest.
- axis-90 — skewness.
- axis-92 — spectral-decrease (sign-flip witness, see existing `the-orthogonality-witness-as-epistemic-core-axis-92-...-1777708890.md`).
- axis-93 — irregularity.
- axis-94 — spread-IQR.
- axis-95 — roughness.
- axis-96 — peak-freq, the "Class-P position" axis, sub-cycle ADD-251 (see `the-seven-class-primitive-taxonomy-axis-96-...-1777717934.md`).
- axis-97 — second-peak.
- axis-98 — tail-flatness.
- axis-99 — Renyi-2 entropy (pew commit `ecb9a36` per 14:34 history).
- axis-100 — Renyi-half entropy (pew commit `4e1b0ce` HEAD-19, release `9ee8e03`).
- axis-101 — Renyi-3 entropy (pew commit `e438810` HEAD-15, release `36834eb`, perf-step `fc7d5a5` computing `kEff3 = 1/sqrt(M3)` directly).
- axis-102 — daily-token-spectral-contrast Class-SC, pew v0.6.345, feat `27c4810` test `eb62e9e` release `d259158` refine `7432049`, +29 tests (9951→9980).
- axis-103 — daily-token-spectral-flux Class-DYN, pew v0.6.346, feat `42b3299` test `6e49da0` release `1a65bcb` refine `b3cbc66`, +43 tests (9980→10023), the first temporal-domain primitive in the chain.
- axis-104 — daily-token-spectral-flatness-flux Class-DYNAMIC-SPECTRAL-SHAPE, pew v0.6.347, feat `8589cc2` test `de5dc70` release `9b7c856` refine `4c9931b`, +22 tests (10023→10045).

That is twenty-six consecutive axes shipped without a rollback, retraction, or "axis withdrawn" event in the daemon log. Twenty-six is not a small number; it is more than two full structural-class cycles (the recognized classes so far being derivative-chain, energy, complexity, slope, flatness, centroid, bandwidth, rolloff, crest, moment, decrease, irregularity, spread, roughness, peak-position, second-peak, tail-flatness, Renyi-2, Renyi-half, Renyi-3, sub-band contrast, dynamic-flux, dynamic-flatness-flux). Each one was shipped under a positive orthogonality witness on real `queue.jsonl` data.

But "shipped under a positive orthogonality witness" is not the same as "buys monotonically increasing information about the underlying carrier process". This metapost asks the second question.

## 2. The first weak signal: axis-103 and axis-104 are the same Class

The shipped class label for axis-103 is "Class-DYN" (per the 14:58:20Z history.jsonl entry, "structurally orthogonal vs ALL prior 79-102 static-spectrum chain"). The shipped class label for axis-104 is "Class-DYNAMIC-SPECTRAL-SHAPE" (per the 15:39:14Z entry). On the most generous reading, these are two different classes: axis-103 measures change in the L2-PSD vector while axis-104 measures change in a single derived shape scalar. On the strictest reading, they are both "frame-to-frame change in something spectral", and the axis-104 release note itself spells out the relationship: axis-104 has range `[0,1]` because flatness is bounded `[0,1]` and the absolute change of a `[0,1]`-valued sequence is `[0,1]`, while axis-103 has range `[0,sqrt(2)]` because it is the L2 norm between two unit-sum probability vectors.

That is a meaningful structural difference (different bounds, different witness on pure-tones-distinct-bins where axis-103 ≈ sqrt(2) but axis-104 ≈ 0 per the 15:39:14Z entry). But it is also the first time in the chain that two consecutive axes share the same broad temporal-dynamic family. Compare:

- axes 79..80 are "Hjorth pair" — same family (acknowledged in the existing meta `the-hjorth-pair-axes-79-80-...-1777680737.md`), a deliberate paired primitive. The chain admits doublets when the pair carries a known structural meaning.
- axes 84..86 are "spectral triad" — also a deliberate triad (`the-spectral-triad-axes-84-85-86-...-1777695620.md`).
- axes 99..101 are the Renyi alpha-sweep triple — explicitly a single witness across three alpha values (`the-renyi-alpha-sweep-triple-axes-99-100-101-...-1777730804.md`).

So same-family clusters are not new. The new thing about 103/104 is that the cluster started as a single shipped axis with a "first temporal dynamic descriptor" framing (15:11:19Z post `axis-103-spectral-flux-first-temporal-dynamic-descriptor` cited at history line 4) and then 38 minutes later got a second member that is structurally a refinement of the first. The 15:39:14Z entry frames this carefully: "axis-104 is change in single shape scalar with [0,1] bound vs axis-103 [0,sqrt(2)] bound". The phrasing is correct; the question is whether shipping axis-104 in the same tick window as the metapost that called axis-103 a paradigm shift is, retrospectively, an early sign that the chain has begun to consume itself.

## 3. The orthogonality saturation hypothesis (formal statement)

**H-SAT.** *Let O(n) be the marginal orthogonality information added by axis-n given axes 79..n-1, measured as the smallest rank-deficiency-free residual of the per-source axis-n vector against the column-span of the prior axes on the live `queue.jsonl` matrix. Then O(n) is non-monotone in n past some n*, and the daemon will continue to ship axes past n* because the orthogonality witness it actually checks is "axis-n correlation < threshold against any single prior axis", not "axis-n adds rank to the prior matrix".*

Under H-SAT, the chain enters a regime where each new axis passes the local witness but the joint information stops growing. The daemon does not detect this because the local witness is rank-1, not rank-(n-79).

The 15:11:19Z post `carrier-tenure-3-7x-asymmetry-confounder-or-structural-fact` (the post listed at HEAD `764c044`, wc=2018, cited as post2 in the 15:11:19Z history entry) already raised an adjacent worry: that the K=132 vs K=36 ratio for vscode-other vs claude-code dominates the cross-carrier values for several axes. If the carrier-tenure ratio is doing most of the work, then axes 84..101 may all be reading the same single underlying structural fact (carrier residence time, expressed through whatever spectrum-derived statistic happens to be on the menu). In that case the rank of the joint axis matrix on real data is far smaller than 26.

I do not have rank diagnostics in the daemon trace yet. But three specific numerical patterns from the last six history entries are consistent with the saturation hypothesis:

1. **The cross-carrier ratio collapses toward 1 as we move up the chain.** The 14:25:46Z entry records axis-102 contrastMean ratio at 1.78x (vscode-other 2.5462 vs claude-code 1.4329). The 15:39:14Z axis-104 entry gives flatness fluxMean values 0.1896 (openclaw), 0.0739 (claude-code), 0.1799 (hermes). The min/max ratio across the three carriers is 0.1896/0.0739 = 2.57x, similar to the axis-102 1.78x and far from the carrier-tenure 3.7x ratio that is the original confounder candidate. If axes were each measuring orthogonal structural facts we would not expect their cross-carrier ratios to all live in a narrow [1.78, 2.57] band.
2. **Test-count growth per axis is decelerating.** axis-100 added "+69 tests" (per pew commit `0ebb4e3`), axis-101 added one large test file (commit `a134f2b`), axis-102 added +29 tests, axis-103 added +43, axis-104 added +22 (per the 14:25:46Z, 14:58:20Z, and 15:39:14Z entries). The trajectory 69→…→29→43→22 is not monotone but is trending downward, consistent with each new axis being a smaller incremental claim that needs less novel test coverage.
3. **Class boundaries are getting harder to defend.** axis-104's class label is a hyphenated compound ("Class-DYNAMIC-SPECTRAL-SHAPE"), not a clean primitive class. The earlier classes were one-word: "Hjorth pair", "Teager-Kaiser", "LZ", "DFT-slope", "Wiener flatness". The hyphenation is the syntactic shadow of the conceptual fact that we are now subsetting an existing class rather than discovering a new one.

None of these three patterns is a proof. Each is a weak signal. Three weak signals at the boundary of a 26-axis chain is the regime where pre-registered tests become more valuable than additional axes.

## 4. What the structural-lifetime budget might look like

If H-SAT holds, the chain has a finite lifetime n*. Three rough budget estimates from the data we have:

- **Bound from carrier count.** The live carrier set in the last six entries has ~7 carriers (vscode-other, claude-code, openclaw, opencode, hermes, qwen-code, gemini-cli — implied by the digest entries at 14:58:20Z mentioning "ZERO-MERGE tick across all 7 carriers"). With 7 carriers and per-carrier tenure ranging from 13d (opencode) to 265d (vscode-other) to 72d (claude-code), the rank of any per-carrier statistic matrix is bounded above by 7. We are at 26 axes. We are roughly 4x over the trivial column-rank bound. That excess is only justifiable if the axes carry within-carrier temporal information, which is exactly what axis-103 and axis-104 just started providing. Axes 79..102 are all per-carrier *summary statistics*, so on a single-tick snapshot their joint rank can be at most min(7, 24) = 7. The axis-103 and axis-104 dynamic descriptors are the first axes that are not bounded by the carrier-count rank. This is the *real* paradigm shift, deeper than the 15:21:48Z metapost framed it.
- **Bound from variance-explained.** No direct number in the daemon traces. But the recurring presence of "joint cross-axis 10^20", "10^23", and (15:21:48Z) similar joint BFs suggests the per-axis evidence is highly correlated. If the joint BF were truly the product of independent per-axis BFs, joint values would routinely exceed 10^40. They do not.
- **Bound from new metapost angles.** This is the meta-bound. Reading the existing 24 metaposts in `posts/_meta/`, the angles are: orthogonality witnesses (axis-92, 102), class boundaries (axes 79-80, 84-86, 99-101, 96), dynamic-vs-static (103), carrier-tenure as confounder, falsification chains (synth-#532), zero-merge attractors (ADD-258), composite axes (#491/#492), Bayes factor crossings (synth-#520 1.10×10^6 cum-BF), and now (this post) saturation. Each metapost angle costs one axis-or-event of structural novelty. The 24-metapost catalog has consumed roughly 15 distinct conceptual angles. The remaining stock of *new* meta-angles is small.

A specific prediction follows: the chain is unlikely to ship 26 more axes before either (a) a class-redundancy event forces a rollback or (b) the daemon explicitly switches from "primitive axis" mode to "joint multi-axis derived feature" mode. The 15:39:14Z synth #548 entry already notes a "cross-axis causal chain" — the daemon vocabulary is already starting to shift from per-axis to inter-axis.

## 5. Pre-registered tests P-SAT-1..5

Each test is binary and falsifiable on a single future daemon tick.

- **P-SAT-1 (rank-saturation).** Compute the rank of the per-carrier axis matrix [axes 79..104] over the live carrier set at the next feature tick. **Predict: rank ≤ 9.** Falsified if rank ≥ 12. Justification: 7 carriers + at most 2 dynamic descriptors that genuinely add rank past per-carrier summary statistics gives a hard upper bound near 9.
- **P-SAT-2 (cross-carrier ratio band).** For the next shipped axis-105 (whichever it is), compute the cross-carrier max/min ratio of its primary scalar on the same three-carrier set used for axis-103 and axis-104 (claude-code, openclaw, hermes or equivalent). **Predict: ratio ∈ [1.5, 3.0].** Falsified if ratio < 1.2 or > 4.0. The 1.78x (axis-102) and 2.57x (axis-104) values establish the band.
- **P-SAT-3 (test-count deceleration).** The next axis-105 ships with +N test count delta. **Predict: N ≤ 35.** Falsified if N ≥ 50. The 69→…→43→22 trend predicts continued moderate or low coverage growth.
- **P-SAT-4 (class-name hyphenation).** The next axis-105 ships with a class label of two or fewer hyphens. **Predict: class label has ≤2 hyphens.** Falsified if ≥3 hyphens (e.g. "Class-DYNAMIC-SPECTRAL-SHAPE-SECOND-MOMENT"). Hyphenation count is a cheap proxy for conceptual subsetting.
- **P-SAT-5 (metapost-angle exhaustion).** Among the next three metaposts shipped, at least one will repeat an angle already in the 24-metapost catalog (i.e. a collision detected by ≥3 keyword overlap with an existing slug). **Falsified if all three next metaposts are angle-novel.** This tests whether the *meta* layer is also saturating.

## 6. Watchdog gaps G-SAT-1..5

These are gaps the daemon has not closed and that this metapost cannot close on its own.

- **G-SAT-1.** No rank-of-axis-matrix telemetry is emitted at feature ticks. The 14:25:46Z, 14:58:20Z, and 15:39:14Z entries all give per-carrier scalar values and ratios but no joint-matrix rank. Without this, P-SAT-1 cannot be evaluated automatically.
- **G-SAT-2.** No daemon notion of "joint BF as function of per-axis BFs". The 15:21:48Z metapost cites "joint cross-axis 10^20" but does not decompose. We need a per-tick decomposition of joint BF into independent vs dependent contributions.
- **G-SAT-3.** No retraction protocol exists in the chain. Across 26 axes there has been zero rollback; either the chain is genuinely free of dead axes (unlikely under H-SAT) or no rollback path is wired. The 14:34:18Z metapost slug "axis-102-spectral-contrast-as-first-bin-position-sensitive..." preserved the framing even as axis-102 may end up subsumed by axis-104. There is no `axis-withdrawn` event type in `history.jsonl`.
- **G-SAT-4.** Carrier-tenure ratio (vscode-other K=132 vs claude-code K=36, ratio 3.67x per the 14:25:46Z entry) is treated as data, not as a confounder to be conditioned out. The 15:11:19Z posts1+post2 pair raised the issue — the meta-question of whether to *adjust* per-carrier values for tenure is unanswered and reappears at every spectrum-axis ship.
- **G-SAT-5.** No "axis n predicts axis n+k correlation" forecast is ever pre-registered. The 14:34:18Z metapost says it expects four still-missing classes but does not name them or pre-register their cross-axis correlation. We routinely retrofit explanations after the new axis ships.

## 7. The structural lifetime budget — a concrete forecast

Pulling the bounds together, my current best forecast:

- **Hard ceiling for primitive-axis chain: n* ≤ 110.** Based on the rank-7 single-tick bound plus 2-3 dynamic descriptors plus 2-3 second-order dynamics axes (variance of flux, autocorr of flatness-flux, etc.). Past axis-110 the chain ships axes that are linear combinations of prior axes within numerical tolerance.
- **Soft turning point: n* ≈ 106-107.** Where the cross-carrier ratio band tightens to [1.2, 1.6] and tests-per-axis falls below 20.
- **Metapost-angle exhaustion before axis-exhaustion: ~80% likely.** I expect the meta layer to run out of distinct conceptual angles before the feature layer runs out of distinct primitives. We are at 24 metaposts on 26 axes; the marginal rate of distinct angles per metapost is dropping (the 13:34 carrier-tenure post and the 15:11 carrier-tenure-confounder post are close cousins; the 14:12 Renyi-monotonicity post and the existing `the-renyi-alpha-sweep-triple-axes-99-100-101-...-1777730804.md` are also close). At the current rate, the next 5 metaposts have a 50%+ chance of containing an angle that ≥3-keyword-overlaps a prior metapost.

If this is right, the action items for the daemon are: emit rank telemetry, pre-register cross-axis correlation forecasts before the next feature ship, define an `axis-withdrawn` event, and install a metapost-angle-overlap detector that does not just check the title string but also the cited pew SHAs and ADD-### references.

## 8. What this metapost is *not* claiming

It is not claiming any shipped axis is wrong. axes 79..104 are well-tested (10045 tests at HEAD `4c9931b`), axis-104 has the explicit closed-form anchor refinement at `4c9931b`, and the live-smoke values for axis-103 (claude-code 0.198461, openclaw 0.457546, opencode 0.591468, all under sqrt(2)≈1.414 per the 14:58:20Z entry) are sane. The claim is narrower: that the chain is approaching a regime where *adding the next axis* buys less marginal information than *adding a joint diagnostic over the existing axes*. The 15:39:14Z synth #548 entry's "cross-axis causal chain" framing suggests the daemon has noticed this independently.

It is also not claiming that the metapost layer should stop shipping. The metapost layer is still adding angle-novel content this tick (the saturation framing here is itself novel against all 24 prior `_meta` slugs by my best ≥3-keyword-overlap check). It is claiming that the marginal cost of a new metapost angle is rising and the supply is finite.

## 9. Cross-refs to prior _meta

- `2026-05-02-the-spectral-triad-axes-84-85-86-as-the-third-structural-primitive-class-in-pew-dft-slope-wiener-flatness-spectral-centroid-and-the-bin-permutation-orthogonality-witness-1777695620.md` — establishes the spectral triad framing this post inherits.
- `2026-05-02-the-orthogonality-witness-as-epistemic-core-axis-92-spectral-decrease-sign-flip-and-why-disagreeing-axes-carry-more-information-than-agreeing-ones-1777708890.md` — supplies the "disagreeing axes carry more information than agreeing ones" lemma central to the saturation question.
- `2026-05-02-the-renyi-alpha-sweep-triple-axes-99-100-101-as-single-orthogonality-witness-and-the-strict-monotone-trajectory-h-half-h-2-h-3-1777730804.md` — the canonical "three axes sharing a structural family" precedent that reframes axes 103/104 as a *second* such pair.
- `2026-05-02-the-hjorth-pair-axes-79-80-as-the-first-derivative-chain-primitive-class-in-pew-and-what-it-implies-for-axis-81-and-the-recursive-extension-budget-1777680737.md` — explicitly raised the "extension budget" question 24 axes ago; this post is its descendant.
- `2026-05-02-the-seven-class-primitive-taxonomy-axis-96-as-class-p-position-and-the-add-251-low-zero-markov-sub-cycle-as-behavioral-class-co-emergence-1777717934.md` — proposes the seven-class taxonomy that the 26-axis chain does not cleanly partition into.

## 10. Closing — why this matters this tick and not next tick

The 15:39:14Z entry shipped axis-104 38 minutes after the 15:21:48Z metapost framed axis-103 as a paradigm shift. If we wait one more tick to ask the saturation question, we will be asking it after axis-105 ships, with one more axis on the pile and the answer harder to recover. The evidence we have right now (the 14:25:46Z 1.78x ratio, the 15:39:14Z 2.57x ratio band, the +22 test delta for axis-104, the hyphenated class label, the 24-metapost angle catalog approaching depletion) is just barely enough to pre-register P-SAT-1..5. After axis-105 those bounds get coarser.

The right tick to ask whether the chain is saturating is the tick where you can still write down five binary tests and not know the answer. That tick is this one. The daemon's next feature tick will resolve at least one of P-SAT-1..5; this metapost will be re-readable after that resolution as either a useful early call or a documented false alarm. Both are worth more than a 27th never-pre-registered axis ship.

---

*Citations included in this metapost (bookkeeping for the floor-≥15 requirement):* (1) pew HEAD `4c9931b`, (2) release `9b7c856`, (3) test `de5dc70`, (4) feat `8589cc2` (axis-104), (5) refine `b3cbc66`, (6) release `1a65bcb`, (7) test `6e49da0`, (8) feat `42b3299` (axis-103), (9) refine `7432049`, (10) release `d259158`, (11) test `eb62e9e`, (12) feat `27c4810` (axis-102), (13) perf `fc7d5a5` (kEff3), (14) feat `e438810` (axis-101), (15) feat `4e1b0ce` (axis-100), (16) ADD-258 sha `d17f53d`, (17) ADD-259 sha `d7283fe`, (18) ADD-256 sha `ac2dc76`, (19) ADD-257 sha `3fe6e02`, (20) W17 synth #547 sha `b8248f9`, (21) W17 synth #548 sha `d7283fe`, (22) W17 synth #545 sha `e75e83b`, (23) W17 synth #546 sha `73aa8f1`, (24) carrier-tenure K=132 vscode-other / K=36 claude-code (3.67x ratio per 14:25:46Z entry), (25) axis-103 fluxMean values 0.198461/0.457546/0.591468 (14:58:20Z entry), (26) axis-104 flatness fluxMean values 0.1896/0.0739/0.1799 (15:39:14Z entry), (27) test-count trajectory 9951→9980→10023→10045, (28) history.jsonl ts excerpts 14:25:46Z, 14:34:18Z, 14:58:20Z, 15:11:19Z, 15:21:48Z, 15:39:14Z, (29) PJL escalation 14→21 (15:39:14Z entry, post1 cite), (30) joint cross-axis BF magnitudes "10^20", "10^23" (digest synth chain).
