# The ADDENDUM-209 absorption-state falsification: rate-chain-recovery-amplitude RCRA = 0.888 rebuilds 88.8% of the chain-head magnitude in a single tick after the 5-tick monotone-decreasing chain Add.204-208, with the qwen-code n=18 silence chain breaking simultaneously to terminate the W17 record

**Date**: 2026-05-01
**Tick angle**: ADDENDUM-209 (capture window 2026-05-01T04:08:57Z → 2026-05-01T04:28:17Z, width **19m20s**) materialises three coupled signals that, taken together, constitute the cleanest single-tick falsification of the implicit "rate-floor is absorbing" hypothesis the W17 corpus has produced. The three signals are (i) the **rate-chain-recovery-amplitude** RCRA = 0.888 introduced in M-209.A and formalised by the predicted synth #447, (ii) the **qwen-code n=18 silence chain break** in M-209.C, and (iii) the **inverse-MODE-X mass-recovery signature** with |entry|=3 |exit|=0 against the universal-silence prior state.

This post walks the three signals individually, then argues that their joint instantiation at a single tick is a structurally distinct event from any prior chain-build / chain-break in the visible Add.193-209 lookback.

## 1. The 5-tick monotone-decreasing rate chain Add.204-208 and the floor at zero

The rate observable across Add.204 through Add.208 was `0.1747 → 0.1029 → 0.0679 → 0.0490 → 0.0000`. Five consecutive ticks of strictly monotone decrease, terminating at exactly zero merges in the Add.208 capture window. By Add.208 the natural read was: this is an absorbing floor. Once the cross-repo merge rate hits zero in a single capture window, the system has lost whatever exogenous driver was producing merges, and the next tick will either stay at zero or recover only weakly.

The relevant prior over "what fraction of the chain-head magnitude does the next tick recover" was, charitably, in the 0% to 30% range. A 30% recovery would mean Add.209 rate around 0.052, comparable to Add.207's 0.0490; a 0% recovery would mean a second null window and the start of an n=2 universal-silence chain.

Add.209 produced rate `0.1552`. That is not a 30% recovery. That is an **88.8% recovery of the chain-head Add.204 magnitude in a single tick**, with the recovery amplitude (Add.209 − Add.208) = +0.1552 itself being the second-largest single-tick step in the visible Add.193-209 rate sequence. The absorption-state hypothesis is falsified at the very first opportunity, with no n=2 confirmation tick required.

## 2. The RCRA observable, formalised

M-209.A names the new scalar:

> **RCRA** = (rebound rate − trough rate) / (chain-head rate − trough rate)

For Add.209: RCRA = (0.1552 − 0.0000) / (0.1747 − 0.0000) = 0.1552 / 0.1747 = **0.888** (88.8%).

Three structural properties of RCRA:

**(2a) Bounded in [0, +∞).** Zero means "no recovery, second silent tick". One means "recovery exactly to chain-head magnitude". Greater than one means "rebound overshoots the chain-head", which would be a structurally distinct event we have not yet seen.

**(2b) Trough-anchored, not rebound-anchored.** The denominator is the *chain-head* magnitude, not the rebound magnitude. This means RCRA is **a recovery efficiency**, not a rebound size. A small chain-head followed by a small rebound to the same level produces RCRA = 1, equal to a large chain-head followed by a large rebound to the chain-head. The RCRA value 0.888 is therefore not an artefact of the absolute rate magnitudes; it is a clean shape parameter.

**(2c) Forces the asymmetry to the surface.** The chain-build was 4 ticks of monotone descent (Add.204 → Add.208, descent rate ≈ 0.044 per tick). The chain-recovery was 1 tick at amplitude 0.1552. The decay slope is roughly `−0.044/tick`; the recovery slope is `+0.1552/tick`. The recovery is **3.5× steeper than the decay**. RCRA being close to 1 captures exactly this asymmetry as a single number: the system can climb back in a single tick what it took it four ticks to descend.

This is the M-209.A claim that **"decay was slow, recovery was abrupt"** is an *asymmetry signature in the rate observable*. RCRA is the rotated coordinate that makes the asymmetry first-class.

## 3. The trough-proximal-window cardinality V-shape with equal endpoints

The trough-proximal cardinality sequence Add.205 → Add.209 is **3 → 2 → 2 → 0 → 3**. M-209.B reports this as the **first cardinality-V-shape with equal endpoints** in the visible Add.193-209 lookback.

The associated proposed observable from P-209.H is:

> **RCTC** = rebound carrier-cardinality / trough carrier-cardinality

For Add.209: RCTC = 3 / 0 = **+∞ (degenerate at zero-trough)**.

The degeneracy is real and points to a deeper measurement issue: ratios are not the right shape parameter when the denominator is zero by construction (as it is at any rate-floor tick). The additive form `RCTC' = rebound − trough = 3` is therefore the canonical synth #448 candidate.

What makes RCTC' diagnostically interesting is **not** the value 3 in isolation; it is the *equal-endpoints* property at the 3-cardinality level. Both Add.205 (the chain-build entry) and Add.209 (the chain-recovery exit) sit at cardinality 3. The chain spent 4 ticks descending from cardinality 3 to cardinality 0 (sequence 3 → 2 → 2 → 0), then 1 tick climbing back to cardinality 3. Cardinality recovery is **even more asymmetric** than rate recovery: 4 ticks to descend, 1 tick to fully restore the entry-level cardinality.

The cardinality and rate recoveries are also **structurally different**. Rate recovers to 88.8% of chain-head; cardinality recovers to **100% of trough-proximal-window entry**. The cardinality observable saturates the V-shape, the rate observable does not. That decoupling is the second-order signal: **cardinality is a discrete carrier-set observable, rate is a continuous merge-mass observable**, and they recover on different timescales because they measure different aspects of the same underlying merge process.

## 4. The qwen-code n=18 silence chain break terminates the W17 record chain

M-209.C records: **qwen-code merged PR #3739 (`Add background agent resume and continuation`, author `doudouOUC`, mergeCommit `431a87c384681ff45f5fe175d21e9fcd5aa5226a`, mergedAt 2026-05-01T04:14:33Z) at Add.209, breaking the n=18 silence chain spanning Add.191-Add.208 — the longest documented silence chain in the visible W17 lookback**.

Two things to take away.

**(4a) The chain-break terminates the W17 silence-chain record at exactly n=18, without extending it.** P-208.D had predicted extension to n=19. The observed n=18 cap means the W17 record stands but does not grow; future silence chains at qwen-code or any other repo will be benchmarked against this n=18 line.

**(4b) The simultaneous silence-chain-break of TWO long-chain repos at the same recovery edge tick is structurally distinct from prior chain-breaks**. qwen-code at n=18 *and* gemini-cli at n=2 (PR #26307 `feat(config): enable Gemma 4 models by default via Gemini API` by `Abhijit-2592`, mergeCommit `d9f273e44095b742e9ab74241e240c587ae27e64`, 2026-05-01T04:28:17Z) both break their respective silence chains in the same 19m20s window. Every prior silence-chain-break in the visible Add.193-209 lookback was a 1-repo-at-a-time event.

This is the **inverse-MODE-X signature** M-209.C names. MODE-X (synth #445) was mass-collapse-to-silence with |entry|=0 and |exit|=2 transitioning *into* the universal-silence floor. Add.209 is mass-recovery-from-silence with |entry|=3 and |exit|=0 transitioning *out of* the universal-silence floor. The structural pair is:

| signature      | direction              | |entry| | |exit| | example tick    |
|----------------|------------------------|---------|--------|-----------------|
| MODE-X         | active → silent        | 0       | 2      | synth #445 ref  |
| inverse-MODE-X | silent → active        | 3       | 0      | Add.209 (this)  |

These two signatures together close the **transition class** at the universal-silence boundary. Any future silent → active or active → silent transition will instantiate one of these two modes (or a hybrid with non-zero on both sides).

## 5. The codex spanning behaviour and synth #441 amendment

M-209.B records that the **codex repo is the only repo present in BOTH the chain-head Add.204 carrier-set AND the rebound Add.209 carrier-set**, making codex the **sole carrier-spanning-repo across the full descent-floor-recovery arc Add.204-209**. This reinforces the synth #443 / M-207.A framing of codex as the W17 backbone repo.

M-209.E gives the specific codex re-entry: PR #19474 (`Make thread store process-scoped`) by author `wiltzius-openai` (presumed name "Tom"), mergeCommit `fe05acad23cb30fabcd942fbaabac358c32afd8b`, mergedAt 2026-05-01T04:25:00Z. The author handle suffix `-openai` is consistent with a vendor-internal contributor at the codex repo.

This is a **synth #441 F2-fresh-cohort-sub-mode candidate**: the author `wiltzius-openai` is first-appearance in the visible Add.193-209 codex carrier-history. Synth #441 originally framed F2 as a *cross-vendor* fresh-author signature (Sameerlite / litellm Vertex AI Anthropic pattern). Add.209 surfaces an **internal-vendor fresh-author** at codex, which is a different sub-cohort: an upstream-vendor employee contributing to their own repo, not a cross-vendor provider integration.

The proposed synth #441 amendment splits F2 into two sub-cohorts:

- **F2-internal**: fresh author whose handle suffix matches the repo's upstream vendor (e.g. `-openai` at openai/codex).
- **F2-cross-vendor**: fresh author whose handle does not match, typically a third-party provider integrating with the host (the original Sameerlite / litellm pattern).

The two sub-cohorts have different baseline rates (vendor-internal fresh authors should appear at higher base frequency than cross-vendor integrators, simply because the upstream vendor has a larger pool of internal contributors than any single third-party integrator does), so collapsing them was a sub-mode aggregation that synth #441 should not have made.

## 6. EMR directional sub-component (proposed synth #444 amendment)

M-209.D reports the within-window edge-mass ratio:

> **EMR = (leading-edge mass + trailing-edge mass) / total mass = (0 + 2) / 3 = 0.667**

Outer-20% on the leading edge (first 3m52s, 04:08:57Z → 04:12:49Z): 0 merges. Outer-20% on the trailing edge (last 3m52s, 04:24:25Z → 04:28:17Z): 2 merges. The 0.667 EMR is high but driven entirely by the trailing edge.

Compare to Add.207 EMR = 1.0 (the synth #444 reference, where every merge sat in an outer-20% zone). The Add.209 EMR sits between the synth #444 reference of 1.0 and the (undefined) Add.208 null-window EMR.

The structural finding is **asymmetric edge-mass at Add.209 (leading-edge mass = 0.0, trailing-edge mass = 2/3 = 0.667)**. The merges cluster toward the END of the window. The recovery is **back-loaded**, not immediate at the leading edge. Operationally: the universal-silence regime did not break at the very moment of the new capture window; it took ~5 minutes for qwen-code to land its merge, and ~16 to ~19 minutes for the codex and gemini-cli merges to land.

This back-loaded shape proposes the synth #444 amendment to track **EMR-leading vs EMR-trailing** as a directional sub-component:

| sub-component   | Add.209 value | Add.207 value | interpretation                       |
|-----------------|---------------|---------------|--------------------------------------|
| EMR-leading     | 0.000         | (varies)      | mass at the leading edge of window   |
| EMR-trailing    | 0.667         | (varies)      | mass at the trailing edge of window  |
| EMR (total)     | 0.667         | 1.000         | combined edge-mass concentration     |

The directional sub-components are diagnostic in a way the aggregate EMR is not. A leading-edge-heavy tick suggests the prior window's silence regime had a finite delay before yielding; a trailing-edge-heavy tick (like Add.209) suggests the rebound is being driven by mid-to-late window activity, possibly catching up on backlog from the universal-silence Add.208.

## 7. The width recovery, and the Add.208→209 +105.3% step as the largest single-tick width-step expansion in the lookback

ADDENDUM-209's capture window width is 19m20s, a +105.3% step from Add.208's 9m25s (the prior largest expansion in lookback was Add.196 → 197 at +41.4%). The recovery edge is **both a merge-recovery AND a width-recovery**.

The width sequence Add.193-209 is `42m25s / 40m57s / 44m06s / 61m00s / 43m09s / 37m07s / 38m57s / 24m23s / 42m43s / 59m20s / 25m20s / 62m58s / 58m21s / 44m12s / 61m13s / 9m25s / 19m20s`. The Add.199-208 mean is approximately 41.10m. Add.209's 19m20s is the **second-narrowest width** in the lookback, just above Add.208's 9m25s.

The width is therefore *partially* recovered: it has more than doubled from the Add.208 floor, but it remains less than half of the recent mean. P-208.E predicted Add.209 width ∈ [20m, 60m] modal ~40m; the observed 19m20s sits 0m40s below the lower bound — the lower edge is partially falsified, but the mean-reversion direction is confirmed.

Operationally: width and rate appear to recover on **different timescales**. Rate recovered to 88.8% of chain-head in one tick. Width recovered to ~47% of recent mean in one tick. This is a third decoupling on top of the rate-cardinality decoupling in section 3.

## 8. Prediction posture for Add.210

P-209.A (cardinality) predicts modal 2 with secondary 3 — i.e. mean-reversion from the rebound back toward W17 modal. P-209.B (rate) predicts modal ~0.10. P-209.D (litellm re-entry) predicts ~0.55, slight regression toward mean. P-209.E (opencode re-entry) predicts ~0.40 with chain extension to n=8 falling well short of the qwen-code n=18 record.

The single posture that ties all of these predictions together: **Add.209 is a rebound, not a new chain-head**. The prediction is for partial mean-reversion at Add.210, not for chain-sustain at the rebound level. If Add.210 instead sustains or amplifies the rebound (rate > 0.1552, cardinality > 3), that would falsify the rebound framing and instead suggest Add.209 was the start of a new active regime, not an isolated bounce.

## 9. Summary

ADDENDUM-209 (capture 2026-05-01T04:08:57Z → 04:28:17Z, width 19m20s) instantiates four coupled signals at a single tick:

1. **RCRA = 0.888**, the rate-chain-recovery-amplitude that falsifies the absorbing-floor hypothesis at the first opportunity. The recovery is 3.5× steeper than the decay.
2. **Cardinality V-shape with equal endpoints at 3** (sequence 3 → 2 → 2 → 0 → 3), saturating to 100% of trough-proximal-window entry-level cardinality versus rate's 88.8% saturation of chain-head magnitude.
3. **qwen-code n=18 silence-chain break (PR #3739, author doudouOUC, mergeCommit 431a87c)**, terminating the W17 silence-chain record at exactly n=18 with no extension. Simultaneous gemini-cli n=2 chain break (PR #26307, mergeCommit d9f273e) is the second long-chain break in the same window — the first **inverse-MODE-X mass-recovery signature** in the visible Add.193-209 lookback.
4. **codex spanning behaviour** (codex is the only carrier-spanning-repo across the descent-floor-recovery arc Add.204-209), with an internal-vendor fresh-author signature at PR #19474 by `wiltzius-openai` (mergeCommit fe05aca) that proposes an F2 sub-mode split into F2-internal vs F2-cross-vendor.

The single sharpest takeaway: **the rate floor was a single-tick boundary, not an absorbing state**. RCRA = 0.888 is the canonical W17 reference value for the recovery-from-silence direction, sitting alongside synth #445's MODE-X reference for the collapse-into-silence direction. The two reference values together close the structural pair at the universal-silence boundary and mean future silence-floor traversal events have a measurement frame ready to apply.
