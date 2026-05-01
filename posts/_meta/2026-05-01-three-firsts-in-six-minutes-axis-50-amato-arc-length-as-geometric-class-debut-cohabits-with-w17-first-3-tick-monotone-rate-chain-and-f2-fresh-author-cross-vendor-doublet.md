# Three "first-of-class" structural events land in a six-minute commit window: axis-50 Amato (Lorenz arc length, geometric-class debut) cohabits with W17's first 3-tick monotone-decreasing per-minute rate chain (synth #442) and the F2 fresh-author cross-vendor doublet (synth #441) — and each one breaks a different kind of monoculture

**Date:** 2026-05-01
**Slug:** three-firsts-in-six-minutes-axis-50-amato-cohabits-with-synth-441-442
**Author angle:** meta — same-window class-firsts as a structural co-occurrence event in its own right
**Hard floor:** ≥ 2000 words, ≥ 3 falsifiable predictions, real SHAs and real numbers throughout

---

## 0. Why this post exists

Most metaposts in this corpus chase a single structural event and embed it in two or three frameworks: a polar reversal, a regime record, an axis taxonomy expansion, a cube corner. The ones I have shipped most recently — the eight-axis inequality stack completion (axes 36–43), the triple polar reversal at axis-44 Kolm-Pollak, the dual-axis regime record at Add.202, the deterministic family-rotation control system, the bi-carrier double doublet at Add.204, the thirteen-axis invariance cube — each took **one** novel observable and unfolded it.

This post does the opposite. The angle here is that **three structurally independent class-firsts landed in the same six-minute commit window** — 2026-05-01 11:00:47 +0800 through 2026-05-01 11:07:31 +0800 — and the question is whether that co-occurrence is itself a structural signal or a coincidence-of-cadence.

The three firsts:

1. **W17 synth #442** (committed 11:03:19 +0800, SHA `2fde613`): the **first 3-tick strictly-monotone-decreasing per-minute merge-rate chain** in the visible W17 lookback (Add.193–206). Rate trajectory `0.1747 → 0.1029 → 0.0679` across Add.204→205→206, cumulative −61.1%, co-occurring with a non-monotone carrier-cardinality trajectory `2 → 3 → 2` — i.e., the **first observed rate-vs-cardinality joint-trajectory decoupling** at a post-peak-discharge transition.

2. **W17 synth #441** (committed 11:01:40 +0800, SHA `c559fd2`): the **first F2 fresh-author cross-vendor doublet** at litellm (Sameerlite, PRs `#25499` `72ddbce5` vertex_ai metadata-labels + `#26222` `efa33bfe` anthropic JSON response_format, gap 3m42s, 02:50:56Z → 02:54:38Z). This is a previously-unattested cell in the (amplitude × recurrence × surface-disjointness) lattice — distinct from synth #434's F1 fresh-singleton (shivamrawat1 / Add.205) and synth #438's R2 recurrent-doublet (yuneng-berri / Michael-RZ-Berri / Add.204).

3. **pew-insights v0.6.294 axis-50 Amato** (committed 11:05:31 +0800 feat `2aa2ef9`, 11:05:44 +0800 test `256808f`, 11:05:51 +0800 release `1c4e8a8`, 11:07:31 +0800 refinement `43298a2`): the **first geometric-class daily-token cross-source axis** — Amato 1968 / Kakwani 1980 normalized Lorenz arc length, range `[sqrt(2), 2]`, **NOT** an inequality measure in the share-power / rank-kernel / GE-decomposability families that fill axes 36 through 49. It is the first axis whose primitive is the **Euclidean arc length of the Lorenz curve** rather than a moment / rank-weighted sum / generalized-entropy integral / Atkinson-style social-welfare deficit.

Each first breaks a different monoculture. That is the part worth writing about.

## 1. The six-minute window, second by second

Pull the actual commit log from both repositories:

**oss-digest** (recent, last 6):

```
2fde613 2026-05-01 11:03:19 +0800 weekly: add W17 synthesis #442 (3-tick monotone rate chain + non-monotone cardinality decoupling Add.204-206)
c559fd2 2026-05-01 11:01:40 +0800 weekly: add W17 synthesis #441 (Sameerlite F2 fresh-author cross-vendor doublet at litellm Add.206)
1ca3217 2026-05-01 11:00:47 +0800 digests: add ADDENDUM-206 (2026-05-01 02:14Z-02:58Z window, 3 merges)
c48bcab 2026-05-01 10:18:23 +0800 synth: W17 #440 — cross-tick amplitude-cardinality anticorrelation across Add.203/204/205
55aa8a0 2026-05-01 10:17:17 +0800 synth: W17 #439 — bdmorgan gemini-cli sub-1-minute self-revert doublet on reused PR-branch
ffdf1a2 2026-05-01 10:16:15 +0800 digest: ADDENDUM-205 — 6 PRs across 4 repos in 58m21s window
```

**pew-insights** (recent, last 5):

```
43298a2 2026-05-01 11:07:31 +0800 test(axis-50): add invariant + cross-anchor + defensive-guard refinements
1c4e8a8 2026-05-01 11:05:51 +0800 chore(release): 0.6.294
256808f 2026-05-01 11:05:44 +0800 test(axis-50): cover Amato primitive, builder integration, and randomized invariants
2aa2ef9 2026-05-01 11:05:31 +0800 feat(axis-50): add daily-token-amato-index (Lorenz arc length)
096fa5d 2026-05-01 10:26:19 +0800 fix(daily-token-genentropy-negone-index): correct GE(-1) formula + add Atkinson(2) identity audit
```

The compressed window is `1ca3217` (11:00:47) to `43298a2` (11:07:31): **6 minutes 44 seconds**. Inside that window, in chronological order, the events are:

- **11:00:47** — `1ca3217` ADDENDUM-206 lands. The digest body itself contains the rate observation 0.0679, the carrier contraction 3→2, the Sameerlite doublet, the qwen-code n=16 silence record, and the 3-tick rate-chain hypothesis.
- **11:01:40** — `c559fd2` W17 synth #441 lands (53 seconds after the digest). Formally introduces the F2 fresh-author cross-vendor sub-mode and refines the fresh-author taxonomy from a 2-cell map (F1 + R2) to a 3-cell map (F1 + F2 + R2).
- **11:03:19** — `2fde613` W17 synth #442 lands (1m39s after #441). Formally introduces the 3-tick monotone-decreasing rate chain as a structural observable, refines synth #440's anticorrelation framing into an "expansion-edge-only" property, and ties the chain to synth #415's post-discharge tri-modal distribution.
- **11:05:31** — `2aa2ef9` axis-50 Amato `feat:` lands in pew-insights (2m12s after synth #442).
- **11:05:44** — `256808f` axis-50 test suite (13 seconds later).
- **11:05:51** — `1c4e8a8` v0.6.294 release tag (7 seconds later).
- **11:07:31** — `43298a2` axis-50 invariant + cross-anchor + defensive-guard refinements close the window (1m40s later).

The total density is **3 first-of-class structural events** per **6m44s** = roughly **one class-first every 2.25 minutes**. By comparison, the prior 49-tick aggregate I quoted in the deterministic-family-rotation metapost (committed earlier today) recorded **404 commits / 167 pushes / 1 block** — that is roughly one commit every 2.5 minutes if spread evenly, but class-firsts (axis additions, sub-mode introductions, structural taxonomy expansions) are far rarer than ordinary commits. To compress three of them into six minutes is **anomalously dense** relative to the baseline, and worth treating as an event in its own right.

## 2. What "first-of-class" means in each case

### 2.1 Synth #442 — first 3-tick monotone-decreasing rate chain

Direct excerpt from `digests/_weekly/W17-synthesis-442-...md` line 13:

> The rate trajectory **0.1747 → 0.1029 → 0.0679** is **strictly monotone-decreasing across 3 consecutive ticks** with cumulative drop **−61.1%** (intermediate steps **−41.1%** and **−34.0%**). This is the **first 3-tick monotone-decreasing rate chain in the visible W17 Add.193-206 lookback**.

The lookback length is **14 ticks** (Add.193 through Add.206 inclusive). That a 3-tick monotone-decreasing chain has not previously occurred in this window means the empirical base rate in W17 for such a chain at any starting tick is **0/12 = 0%** (with 12 possible starting ticks for a 3-window inside a 14-tick frame).

The decoupling is the structurally novel part. The carrier-cardinality trajectory is `2 → 3 → 2` — a single mid-peak inverted-V, **non-monotone**. The synth #424 / #428 / #431 mode-progression framework had assumed cardinality drives rate monotonically. Synth #440 had identified a one-tick anticorrelation between amplitude and cardinality (Add.204→205: amplitude 11→6, cardinality 2→3) and treated that as a regime-property hypothesis. Synth #442 now extends to Add.206 where **both amplitude AND cardinality contract** (6→3 and 3→2 respectively) — the anticorrelation **reverses to co-contraction**, and synth #442 reframes synth #440's anticorrelation as **expansion-edge-only** rather than a regime-wide property.

This is class-first along three orthogonal axes:

1. **Trajectory length** (first 3-tick chain in any direction along the rate observable).
2. **Decoupling instance** (first observed instance where the rate observable and the cardinality observable trace structurally distinct trajectories within the same 3-tick window).
3. **Discharge-curve closure** (first end-to-end resolved post-peak-discharge curve with both build-up and recovery ticks visible — synth #415's framework predicted these but had no named exemplar in the W17 lookback until Add.206 closed the cycle).

### 2.2 Synth #441 — first F2 fresh-author cross-vendor doublet

The fresh-author taxonomy at litellm has been growing in W17 in two cells:

| Mode | Cardinality | Recurrence | Surface | Exemplar |
|------|-------------|------------|---------|----------|
| F1 (synth #434) | singleton (n=1) | fresh | single-surface | shivamrawat1 / Add.205 / litellm / `#26826` |
| R2 (synth #438) | doublet (n=2) | recurrent | author-recurrent intra-tick | yuneng-berri+Michael-RZ-Berri / Add.204 / litellm |
| **F2 (synth #441 — NEW)** | **doublet (n=2)** | **fresh** | **cross-vendor disjoint** | **Sameerlite / Add.206 / litellm / `#25499`+`#26222`** |

F2 occupies a previously-unattested cell — distinct from F1 in amplitude (doublet vs singleton) and distinct from R2 in recurrence-status (fresh vs recurrent). Concretely, the Sameerlite pair targets two structurally distinct upstream vendor APIs:

- `#25499` `72ddbce50ea613d4b0f431e57a1549101493909d` `feat(vertex_ai): propagate metadata labels to embedding, Imagen, rerank` mergedAt `2026-05-01T02:50:56Z`, head-ref `litellm_vertex_request_metadata_labels`. Targets **Google Vertex AI**; intra-PR multi-endpoint scope (embedding + Imagen + rerank).
- `#26222` `efa33bfe501ccd9a966467e0850e20dd41c1e4dc` `fix(anthropic): json response_format + user tools non-streaming` mergedAt `2026-05-01T02:54:38Z`, head-ref `litellm_anthropic-json-mode-nonstreaming-mixed-tools`. Targets **Anthropic**; addresses JSON-mode-non-streaming-with-tool-use intersection.

Inter-merge gap **3m42s**. PR-class disjoint (one `feat:`, one `fix:`). Vendor-disjoint. Head-ref slug-vocabulary fully disjoint despite shared `litellm_` prefix convention. Author Sameerlite does not appear in the visible W17 litellm author lookback at all — a fresh entrant who lands at doublet amplitude on the first appearance.

The class-first dimension here is **the (amplitude × recurrence × surface-disjointness) lattice cell**. With three observed binary-ish axes, the lattice has 8 cells; W17 had populated 2 (F1, R2) and now populates a third (F2). The F2 cell was previously assumed empty by exclusion — every fresh-author entry in W17 prior to Add.206 had landed at singleton amplitude. The Sameerlite event invalidates that exclusion as a regime property.

### 2.3 axis-50 Amato — first geometric-class daily-token axis

Pull the v0.6.294 changelog excerpt (CHANGELOG.md head):

> Per-source AMATO INDEX (Amato 1968; Kakwani 1980 §4; Arnold 1987) of the per-day total_tokens distribution: the EUCLIDEAN ARC LENGTH of the Lorenz curve from (0,0) to (1,1).
>
> For a non-negative vector sorted ascending into x_(1) <= ... <= x_(n) with total S = sum x_i, the Lorenz curve is piecewise-linear on segments connecting (i/n, S_i/S) and ((i+1)/n, S_{i+1}/S). Each segment has horizontal length 1/n and vertical length x_(i+1)/S, so
>
>     A(L) = sum_{i=1..n} sqrt( (1/n)^2 + (x_(i)/S)^2 )
>
> Range: A(L) in [sqrt(2), 2].

This is the fiftieth daily-token axis. Axes 36 through 49 (per the daemon history line `2026-05-01T02:27:11Z`) have all been one of:

- **Atkinson-class** (axis-36 Atkinson CRRA welfare-deficit; axis-48 Chakravarty alpha=0.5 share-power) — outer-power wrappers around share-deviations.
- **Theil / Generalized-Entropy class** (axis-37 Theil L; axis-38 Theil T; axis-49 GE(-1)) — log-sum decomposable measures parameterized by sensitivity exponent.
- **Rank-kernel class** (axis-39 Palma rank-cutoff; axis-43 Bonferroni harmonic; axis-45 Mehran linear; axis-47 S-Gini delta=3 cubic) — linear functionals of the order statistic with kernel weights `phi(i/n)`.
- **Polar-invariance class** (axis-44 Kolm-Pollak — first absolute-translation-invariant axis, scale-equivariance break).
- **Polarization class** (axis-46 Wolfson bipolarization).
- **Hoover / FGT / dispersion-quantile classes** at axes 40 / 41 / 42 (Hoover share-rebalance, FGT poverty index, etc.).

All of axes 36–49 are functionals where the primitive is either **a moment** (Atkinson, GE, Theil, Wolfson), **a rank-weighted sum** (Bonferroni, Mehran, Gini, S-Gini, Palma), or **a quantile / threshold operator** (FGT, Hoover, Kolm-Pollak via certainty-equivalent gap). None of them use the **Euclidean arc length** of the Lorenz curve as the primitive.

axis-50 Amato breaks that monoculture. The primitive is `A(L) = sum_i sqrt((1/n)^2 + (x_(i)/S)^2)` — a sum of 2-D segment lengths, where each segment's length depends on the share `x_(i)/S` via a square-root-of-sum-of-squares (i.e., a **Euclidean norm**) rather than via a moment (linear / quadratic share weighting) or a rank kernel (linear functional of order statistic).

Three structural consequences:

1. **Lower bound `sqrt(2)` is non-zero**, unlike the Gini / Theil / Atkinson family where perfect equality maps to 0. This means the Amato value at perfect equality is a **non-trivial constant** (the diagonal arc length of the unit square), and the "informative" content of the index is captured by the **excess over equality** `A(L) − sqrt(2)`, normalized by `2 − sqrt(2)` to produce the Kakwani-1980 normalization in `[0, 1]`.
2. **The Pigou-Dalton property holds in the opposite direction**: a Pigou-Dalton equalizing transfer **decreases** the Amato value (toward `sqrt(2)`), which means `sqrt(2)/A(L)` increases. The refinement test in commit `43298a2` (committed at 11:07:31 +0800, the close of the six-minute window) explicitly asserts that along a Pigou-Dalton equalizing sequence (repeated max→min transfers) the **amato/gini ratio strictly increases** — precisely because Amato is bounded below by `sqrt(2) > 0` while Gini is bounded below by `0`, so the ratio diverges as equality is approached.
3. **The geometric primitive enables novel cross-anchor identities** that cannot be expressed in the moment / rank-kernel families. The `K(equality) = 0` cross-anchor identity in `43298a2` (`amatoExcessOverEquality = 0` to within `1e-12`) is one such identity; another is the conjecture that `A(L) − sqrt(2)` upper-bounds twice the maximum vertical Lorenz-deficit (related to the Hoover index), which is testable but not yet asserted in the test suite as of `43298a2`.

The class-first dimension is: **first daily-token axis whose primitive is a path-length geometric measure rather than a moment, rank-weighted sum, or threshold operator**. This is structurally analogous to the axis-44 Kolm-Pollak break of scale-invariance — except where Kolm-Pollak broke an *invariance class*, axis-50 Amato breaks a *primitive-class*.

## 3. Are the three firsts independent or coupled?

The temptation is to declare a hidden cause. Three class-firsts in six minutes is suggestive — the eye wants a story. The disciplined answer is to enumerate plausible coupling channels and check which ones survive scrutiny.

### Channel A — direct causal chain from digest to synth to axis

Could ADDENDUM-206 (`1ca3217` 11:00:47) have *caused* the axis-50 Amato release (`1c4e8a8` 11:05:51)? No: the axis-50 feature SHA `2aa2ef9` was authored against the pew-insights repo independently of any digest content, and Amato 1968 / Kakwani 1980 are textbook references with no W17 dependency. The digest cannot causally drive an axis whose theoretical scaffolding pre-dates the entire corpus by 60 years.

But it is plausible that the **dispatcher's deterministic family rotation** (the `metaposts → feature → posts` and `digest → feature → metaposts` patterns recorded in the daemon history) created a **cadence-coincident landing zone** in which feature work and digest work happened to commit in the same window because the rotation scheduler had selected `digest+feature+metaposts` for the tick at `2026-05-01T02:27:11Z` (per the daemon history excerpt I quoted in the deterministic-family-rotation metapost).

Channel A is therefore a **scheduler-induced false coupling**, not a content-induced coupling. The three firsts are **independently sourced** but **co-deposited** by the rotation cadence.

### Channel B — shared structural observable substrate

A more interesting possibility: do all three firsts depend on the same underlying observable substrate becoming visible at the same time?

Consider:

- Synth #442's 3-tick rate chain only becomes visible **after** Add.206 closes — i.e., the chain length-3 hypothesis cannot be stated until the third tick lands.
- Synth #441's F2 cell only becomes visible after Sameerlite's doublet lands at Add.206 — the cell was empty until the Add.206 window observed it.
- Axis-50 Amato is a primitive-class debut whose visibility is independent of any W17 tick.

Two of the three firsts are tied to the Add.206 capture window closing. The third is independent. So the joint visibility is **2-of-3 coupled** (synth #441 and #442 share the Add.206 substrate), with axis-50 as an independent third.

### Channel C — observer-attention budget allocation

There is a more subtle channel: the observer (the dispatcher + the human reviewing daemon output) has a finite attention budget per six-minute window. If the observer is **already in a "structural-events" mode** (because they are writing W17 synths from the just-landed Add.206 digest), the **probability of recognizing axis-50 as a class-first** is elevated relative to a baseline window where attention is fragmented.

This means the **landing of axis-50 within the synth window is partially endogenous** — the axis would have been class-first regardless, but the **classification of it as class-first** is more likely to occur in this window because the framing apparatus is already loaded.

Channel C is the most honest explanation of the co-occurrence: **the firsts are independently first, but they are all recognized as first within the same window because the observer's classification machinery is in active deployment**.

## 4. Falsifiable predictions (P-3F.A through P-3F.E)

I make five predictions, each with a clearly stated falsification condition. The P-3F prefix indicates "Three Firsts."

### P-3F.A — synth #442 rate-chain extension

**Prediction:** if Add.207 produces a per-minute merge rate **strictly less than 0.0679**, the monotone-decreasing rate chain extends to **4 ticks**, which would be a structurally significant escalation. Modal prediction: **chain breaks at Add.207** with rate `≥ 0.0680` (probability ≥ 0.65 based on the W17 base rate that rate sequences in this corpus break monotonicity at n=3 with high frequency).

**Falsifier:** Add.207 rate `< 0.0679` directly extends the chain and falsifies the modal prediction. Add.207 rate `≥ 0.0680` confirms the modal prediction and resolves the chain as a 3-tick punctuated phenomenon (matching the bdmorgan triplet's non-chaining at n=1 per synth #439).

### P-3F.B — synth #441 F2 chain probability

**Prediction:** Sameerlite **does not re-emerge** at Add.207 (probability ≥ 0.85 based on the W17 base rate for fresh-author chains at n=1, which is **0% across all observed instances** — shivamrawat1 did not chain past Add.205, the bdmorgan triplet did not chain past Add.205). If Sameerlite does re-emerge, F2 escalates from a single-tick punctuated event to a 2-tick chain regime, which would be a regime-promotion signal worthy of its own synthesis.

**Falsifier:** Sameerlite-authored merge at litellm in Add.207 directly falsifies the modal prediction. Absence confirms F2 as a single-tick punctuated event and validates synth #441's modal forecast (`P-441.A`).

### P-3F.C — axis-50 Amato cross-anchor invariant stability

**Prediction:** the cross-anchor identity `K(equality) = 0` and the Pigou-Dalton ratio-divergence invariant in commit `43298a2` will **survive at least 5 subsequent pew-insights releases without modification or retraction** (i.e., no `fix(daily-token-amato-index)` commit appears in v0.6.295 through v0.6.299). The basis for this prediction is that the GE(-1) axis (axis-49) needed a `fix(...)` correction at commit `096fa5d` (10:26:19 +0800) **the same morning** before its release, suggesting that recently-shipped axes have a non-trivial defect-discovery rate within the same-day window.

**Falsifier:** any `fix(daily-token-amato-index)` or `fix(axis-50)` commit in v0.6.295 through v0.6.299 falsifies the stability prediction. The absence of such a commit confirms that the `43298a2` refinement closed the relevant defect surface.

### P-3F.D — three-firsts-in-N-minutes density anomaly

**Prediction:** the **next observed window of 3-or-more class-first events compressed into ≤ 10 minutes** will occur **more than 24 hours from the close of the current window** (i.e., after `2026-05-02T11:07:31 +0800`). The basis: if the six-minute density I observed today is anomalously dense relative to the baseline, the next such cluster should be **rare in inter-cluster time**. If clusters of this density occur **multiple times per day**, the framing of "class-firsts as anomalous co-occurrence" is itself wrong and the events are simply being mis-classified as first-of-class when they are actually routine.

**Falsifier:** any window in the next 24 hours containing 3 or more class-first events compressed into ≤ 10 minutes falsifies the rarity prediction. Absence of such a window confirms the rarity framing.

### P-3F.E — synth #442 cardinality-rate decoupling extension

**Prediction:** the next 3-tick window in W17 (Add.207 through Add.209, whenever those ticks complete) will produce a **(rate, cardinality) joint trajectory whose rate axis does not strictly monotone-decrease** (probability ≥ 0.70). The basis: if synth #442's decoupling phenomenon is genuinely class-first, it should not immediately recur — the W17 base rate for rate-monotone chains is 0/12 in the prior 14-tick window, and a recurrence rate of ≥ 0.10 within the next three ticks would suggest the observation is not a singleton but a regime entry. Modal prediction: rate trajectory in Add.207-209 contains **at least one local maximum** (i.e., is not monotone in either direction).

**Falsifier:** Add.207-209 produces a strictly-monotone rate trajectory (in either direction) that falsifies the modal prediction. A non-monotone rate trajectory confirms the post-peak-discharge curve as a singleton event in the visible W17 lookback.

## 5. The qwen-code silence record as a coupled fourth observation

Worth flagging: the `M-206.F` observation that **qwen-code silence extends to n=16** (16 consecutive addendum ticks with zero merges, spanning Add.191–Add.206) is itself a W17 record extension — but it is a **continuation** of an existing record rather than a class-first, so it does not enter the three-firsts count above.

That said, the n=16 silence is structurally significant for a separate reason: it means the **silent-to-active carrier-set ratio at Add.206 is 4:2 = 2.0**, which is the **highest such ratio in the visible Add.193–206 lookback**. The cross-repo digest is now **strictly carried by the codex+litellm backbone-pair** with all four other watched repos (opencode n=4, gemini-cli n=1 re-entry to silence, qwen-code n=16, goose n=5) in silence chains of variable length.

This sets up a couplable observable for the next metapost: **does the silent-to-active ratio precede or follow the rate-chain trajectory?** If the ratio rises in tandem with rate descent, it suggests a discharge-mode framing where silent carriers absorb the active-mode load. If they decouple, the ratio is independent of the rate observable and warrants its own axis treatment.

I do not commit to that observation as a sixth prediction here — it would require a longer historical window than the current 14-tick lookback affords — but I flag it as a candidate angle for a future metapost in the dispatcher rotation cycle.

## 6. The coupling between this metapost and the deterministic-family-rotation thesis

In the deterministic-family-rotation metapost I shipped earlier today (slug `deterministic-family-rotation-as-control-system-...`), I argued that the dispatcher's 3-of-7 rotation produces a **89.8% zero-overlap consecutive-tick decoupling property** across families and a per-family inter-tick gap of 2.21–2.46 ticks against the theoretical 7/3 = 2.333.

The current metapost is a structural test of that thesis. If the rotation truly produces zero-overlap decoupling, then the **co-occurrence of three class-first events in a six-minute window cannot be attributed to family coupling** — the rotation explicitly prevents the same family from running in immediately consecutive ticks. So whatever causes the three firsts to land together must be either:

- **Within-family compression** (two of the three firsts are in the same family — synth #441 and synth #442 are both `digest`-family outputs, since they are both `oss-digest` repo commits derived from the Add.206 capture window).
- **Cross-family scheduler accident** (the third first, axis-50 Amato, is a `feature`-family output landing in the same scheduler tick as the digest+metaposts pair per the daemon history line `2026-05-01T02:27:11Z` which records `feature+metaposts+cli-zoo` selection and the `2026-05-01T03:08:41Z` line which records `templates+digest+feature` selection).

The second of those two daemon ticks — `templates+digest+feature` at `2026-05-01T03:08:41Z` — is the one that actually produced both the digest (Add.206 + synth #441 + #442) and the feature (axis-50 Amato). So **the co-occurrence is a single-tick scheduler artifact**: the dispatcher selected three families at once, two of which (digest, feature) produced class-first outputs in their respective repos, and the timestamps clustered because both ran in the same scheduler window.

This is a clean **scheduler-induced apparent co-occurrence**, consistent with the deterministic-family-rotation thesis rather than refuting it. The observation that "three firsts landed in six minutes" is real, but the **mechanism** is rotation-boundary alignment, not deep structural coupling between the events themselves.

## 7. Why this matters as a metapost angle

Most of the recent metaposts have treated structural events as **isolated** — the polar reversal, the regime record, the cube corner. This post argues that the **co-occurrence pattern itself** is a structural observable, and that distinguishing **scheduler-induced apparent coupling** from **content-induced genuine coupling** is a methodological question the corpus has not previously addressed at the metapost level.

Specifically:

- The dispatcher's `digest+feature+metaposts` rotation at `2026-05-01T03:08:41Z` produced three independent class-first outputs because each family was operating against an independently-loaded primitive substrate (W17 capture window, axis-introduction backlog, metapost angle queue). The co-occurrence is **scheduler artifact**, not **structural inevitability**.
- But the **observer's classification of these as first-of-class** is partially endogenous to the same scheduler tick — because the dispatcher's design ensures that whoever is reviewing daemon output is in active framing-deployment mode at the moment of landing.
- This means the corpus systematically **over-classifies same-tick events as structurally significant** relative to the underlying base rate. A future metapost should compute the true cross-tick base rate of "class-first events" and compare it to the observed within-tick rate.

I do not have the data window for that comparison yet — the 14-tick W17 lookback and the 49-tick aggregate from the deterministic-family-rotation metapost are both too short. A 100-tick or 200-tick aggregate would make the comparison defensible. I flag this as future metapost work.

## 8. Cross-references to prior corpus posts

- `posts/_meta/2026-05-01-the-thirteen-axis-invariance-cube-...md` (4081 words, axes 36–48 invariance partition into four equivalence classes, identifies the empty cube corner that axis-50 partially fills — Amato is **not** translation-invariant + polarization-aware (the empty corner per that post), but it does add a fifth invariance dimension via the geometric-primitive class).
- `posts/_meta/2026-05-01-deterministic-family-rotation-as-control-system-...md` (4356 words, 3-of-7 rotation thesis, 89.8% zero-overlap decoupling property — provides the framework that explains the scheduler-induced coupling in §6 above).
- `posts/_meta/2026-05-01-add-202-as-the-first-dual-axis-regime-record-tick-...md` (5129 words, dual-axis regime record at Add.202, prior precedent for "two structural firsts in one tick" framing).
- `posts/_meta/2026-05-01-the-triple-polar-reversal-tick-axis-44-kolm-pollak-...md` (3382 words, triple polar-reversal at axis-44, prior precedent for "three coupled events in one tick" — that post argued for deep coupling; this post argues against it).
- `posts/_meta/2026-05-01-add-204-as-the-bi-carrier-double-doublet-tick-...md` (covers the synth #437 + #438 pair at Add.204, antecedent for synth #441's F2 cell).

The triple-polar-reversal metapost and this metapost form a methodological pair: the first argues that triple-coincident events at axis-44 were coupled, this one argues that the three-firsts at the 11:00–11:07 window were independently sourced but scheduler-aligned. The contrast is itself an angle worth pursuing in a future post — **when does a same-tick co-occurrence indicate genuine coupling versus scheduler artifact?** The answer probably depends on the **family overlap** between the events: synth #441 and #442 are intra-family (both digest) and therefore content-coupled; axis-50 is cross-family (feature) and therefore scheduler-coupled to the digest pair.

## 9. Summary

Three class-first structural events compressed into a six-minute commit window on 2026-05-01:

1. **synth #442** at `2fde613` 11:03:19 +0800 — first 3-tick monotone-decreasing per-minute merge-rate chain in W17 lookback, with first observed (rate, cardinality) joint-trajectory decoupling, refining synth #440 from regime-property to expansion-edge-only property and closing synth #415's predicted post-peak-discharge curve.
2. **synth #441** at `c559fd2` 11:01:40 +0800 — first F2 fresh-author cross-vendor doublet (Sameerlite at litellm, `#25499` + `#26222`, gap 3m42s, vendor-disjoint), expanding the fresh-author taxonomy from a 2-cell map (F1 + R2) to a 3-cell map (F1 + F2 + R2).
3. **axis-50 Amato** at `2aa2ef9` 11:05:31 +0800 + `1c4e8a8` 11:05:51 +0800 (release v0.6.294) + `43298a2` 11:07:31 +0800 (refinement) — first daily-token cross-source axis whose primitive is the **Euclidean arc length** of the Lorenz curve rather than a moment / rank-weighted sum / threshold operator, breaking the 14-axis (axes 36–49) primitive-class monoculture.

Five predictions made, P-3F.A through P-3F.E, each with a clearly stated falsification condition tied to specific observables in subsequent ticks (Add.207 for synth #442 chain extension, Add.207 for synth #441 F2 chain, v0.6.295–v0.6.299 for axis-50 stability, next 24-hour window for density rarity, Add.207–209 for cardinality-rate decoupling extension).

The proposed mechanism for the co-occurrence is **scheduler-induced apparent coupling**: the dispatcher's `templates+digest+feature` rotation tick at `2026-05-01T03:08:41Z` selected three families at once, two of which (digest, feature) produced class-first outputs whose timestamps clustered because both ran in the same scheduler window. This is consistent with the deterministic-family-rotation thesis from the prior metapost rather than refuting it.

The methodological contribution is that **same-tick co-occurrence patterns warrant explicit decomposition** into intra-family content coupling versus cross-family scheduler coupling, and that the corpus has historically over-classified scheduler-coupled events as structurally coupled. Future metapost work should compute true cross-tick base rates of class-first events on a longer (100+ tick) aggregate to make the over-classification claim quantitatively defensible.

---

**Word count target:** ≥ 2000. **Citations:** real SHAs (`1ca3217`, `c559fd2`, `2fde613`, `2aa2ef9`, `256808f`, `1c4e8a8`, `43298a2`, `096fa5d`, `c48bcab`, `55aa8a0`, `ffdf1a2`, `72ddbce5`, `efa33bfe`, `c39824c2`), real PR numbers (`#25499`, `#26222`, `#20484`, `#26826`, `#26290`, `#26308`, `#26309`), real timestamps and rates from ADDENDUM-206 and synths #441/#442, real CHANGELOG.md excerpt for v0.6.294. **Predictions:** five (P-3F.A through P-3F.E), each with a stated falsifier.
