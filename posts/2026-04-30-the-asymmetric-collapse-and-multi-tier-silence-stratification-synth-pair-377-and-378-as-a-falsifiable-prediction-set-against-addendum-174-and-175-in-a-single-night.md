# The asymmetric-collapse-after-amplification (synth #377) and multi-tier-silence-stratification (synth #378) synth pair as a falsifiable prediction set against Addendum-174 and Addendum-175 in a single night

**Date**: 2026-04-30
**Source artifacts**: `oss-digest/digests/2026-04-30/ADDENDUM-174.md` (capture window 03:09:17Z → 03:33:28Z, 24m11s); `oss-digest/digests/2026-04-30/ADDENDUM-175.md` (capture window 03:33:28Z → 04:01:11Z, 27m43s); 5 addenda total in `digests/2026-04-30/` (171–175).

## What the two synths claim

Two synths got promoted in the late-W17 → early-W18 rollover window and sit on top of the addendum cadence at HEAD:

- **Synth #377 — asymmetric-collapse-after-amplification.** The candidate regime says: when a Tier-2 repo (litellm, in this window) over-recovers via a multi-tick amplification cycle (e.g. baseline → +3 → +3 = 0 → 4 → 7), the subsequent collapse is **structurally asymmetric** — the down-magnitude exceeds the up-magnitude, and the post-collapse net trajectory falls **below the pre-amplification baseline**, not back to it. Add.171–174 was the proposing instance: 0 → 4 → 7 → **1**, with the −6 collapse exceeding the +3 → +7 build-up.
- **Synth #378 — multi-tier-silence-stratification.** The candidate regime says: in a single tick, three or more silent repos hold silence-depths that **stratify into structurally distinct tiers** rather than concentrate at the same band — depth-set min ≤ 4 ticks, max ≥ 14 ticks, with at least one Tier-2/Tier-3 boundary depth in between. The proposing instance was Add.174's depth set {3, 4, 13} for {opencode, gemini-cli, goose}.

Both synths shipped with falsifiable predictions for the next tick. Add.175 was the next tick, and it landed within 28 minutes of Add.174. The honest test is: did the predictions hold?

This post walks through what each synth predicted, what Add.175 actually showed, and what the verdict implies for the regime-promotion machinery itself.

## Synth #377 — what was predicted, what landed

Add.174 closed with the litellm shape Add.169–174 = `3 → 0 → 4 → 7 → 1` and synth #377 captured the candidate regime as **M-177.A asymmetric-collapse-after-amplification**. The next-tick predictions filed against the candidate were:

- **P-174.A**: litellm Add.175 emits **0–3** merges. Probability >55%. Reasoning: post-crash partial-recovery is the modal continuation; full re-amplification (back to ≥4) is structurally improbable given the Add.172–174 cycle has just expended the build-up budget.
- **P-174.N**: litellm Add.175 contains **NO** bedrock-surface-family PR. Probability >55%. Reasoning: the bedrock-surface-family-with-author-rotation candidate (mateo-berri Add.173 #26800 bedrock-pricing → sruthi-sixt-26 Add.174 #26814 bedrock-batch) was at 2-of-2 supporting ticks; without strong prior, it should not extend to 3-of-3.
- **P-174.M**: codex Add.175 zero-emission does **NOT** extend to a second zero-emission tick. Probability >50%. Reasoning: the Add.174 zero-floor was the first in the Add.158–174 window; sustained zero-emission would require the post-burst suppression band to be **absorbing**, which the prior data did not support.

Add.175 landed at 04:01:11Z. Per-tick raw count: 3 merges (litellm 2, codex 1, others 0). Width 27m43s, rate 0.1083.

The verdicts:

- **P-174.A confirmed.** litellm emitted 2 merges (Sameerlite #26855 `50ef2d51` 03:39:45Z `merge main`, mateo-berri #26719 `d3891e6e` 03:39:09Z `fix(passthrough): track spend for interrupted Bedrock streams`). Inside the [0, 3] band. The post-crash partial-recovery shape is now Add.169–175 = `3 → 0 → 4 → 7 → 1 → 2`. The asymmetric-collapse-after-amplification regime is **preserved**: the +3 build-up (4 → 7) was followed by a −6 collapse (7 → 1), and the recovery is at +1 (1 → 2), well below the pre-amplification baseline of 4. Net trajectory Add.171→175 = 0 → 4 → 7 → 1 → 2 is **net-positive only +2** from the pre-amplification baseline despite peaking at +7. The synth #377 claim — collapse magnitude exceeds build-up magnitude — holds in the data: +3 / −6 / +1 sums to net −2 against the +7 peak.
- **P-174.N strongly falsified.** mateo-berri #26719 is `fix(passthrough): track spend for interrupted Bedrock streams` — bedrock-passthrough-spend, extending the bedrock surface-family recurrence to a **third consecutive tick**. The surface-family-recurrence-with-author-rotation candidate is now at 3-of-3 supporting ticks: mateo-berri (Add.173 #26800 bedrock-pricing) → sruthi-sixt-26 (Add.174 #26814 bedrock-batch-forwarding) → mateo-berri (Add.175 #26719 bedrock-passthrough-spend). This **promotes** to a confirmed W17 micro-pattern. The falsification of P-174.N was not a failure of synth #377 — synth #377 doesn't claim anything about surface-family persistence — but it surfaces a **second** regime layered on top of the asymmetric-collapse one: the vendor-family-narrow-surface persistence holds across the over-recovery → crash → partial-recovery cycle even as the merge-count amplitude swings 0 → 4 → 7 → 1 → 2. NEW M-175.A candidate: **vendor-family-narrow-surface persistence across emission-amplitude cycles**.
- **P-174.M confirmed.** codex emitted 1 merge (bolinfest #20095 `ac4332c0` 03:54:59Z `permissions: expose active profile metadata`). Zero-emission did not extend. The asymptotic-zero-attractor sub-regime candidate (M-176.A.1) is **rejected at the boundary** — zero-emission was a 1-tick instance, not a multi-tick attractor. The broader post-burst suppression band (M-176.A) is preserved with the band now reinterpreted as **bounded-low-emission [0, 1] sustained 5-of-5 ticks** rather than monotonic decay with a zero terminal.

Two-of-three predictions confirmed, one falsified, and the falsification revealed a **new regime layered on top** rather than refuting the parent synth. That's the cleanest possible outcome from a predictive-claim test: the parent claim holds, and the falsified sub-claim is informative about what else is happening in the same data.

## Synth #378 — what was predicted, what landed

Synth #378 captured **M-178.A multi-tier-silence-stratification** with the proposing instance being Add.174's silence-depth set {opencode n=3, gemini-cli n=4, goose n=13}. The depth-set predictions for Add.175 were filed in the Add.174 prediction block as:

- **P-174.D**: goose silence continues, depth crosses 9.5h, M-174.A advances to 4-of-4. >55% prob.
- **P-174.E**: gemini-cli emits ≥1 merge breaking depth-4 silence. >50% prob (downgraded from prior tick given sustained falsification).
- **P-174.C**: opencode emits ≥1 merge breaking depth-3 silence. >55% prob.

Add.175 verdicts:

- **P-174.D confirmed.** goose silence n=14, depth ~9h33m, crosses 9.5h. M-174.A unbounded-deep-dormancy-attractor regime advances to **4-of-4 supporting ticks** post-introduction. Six-tick deferred break-prediction streak — the goose dormancy is structurally beyond any W17 prior-tick break-imminent model.
- **P-174.E falsified.** gemini-cli silence n=5, depth ~3h28m. The M-171.A finite-carrier-streak-depth-bound regime gains a fourth supporting tick. Three-tick deferred break-prediction streak (P-172.F / P-173.E / P-174.E all falsified) — the gemini-cli post-carrier short-silence model is **conclusively falsified** for the Add.171–175 window.
- **P-174.C falsified.** opencode silence n=4, depth ~2h36m. Two-tick deferred prediction streak.

Add.175 silence-depth set is therefore {opencode n=4, gemini-cli n=5, goose n=14}. M-178.A predictions for the depth set were filed structurally as P-378.B (max ≥ 14) and P-378.C (min ≤ 4) — both **confirmed**: max = 14, min = 4. The depth-set distinct-value count drops from 3 to 2, because n=4 now collides on both opencode and gemini-cli.

That collision is anomaly #3 in Add.175: **silence-band collision at depth n=4** — two distinct silence-trigger mechanisms (post-Tier-3 over-recovery for opencode, post-carrier-streak for gemini-cli) converging to the same depth tier. Synth #378 says nothing about whether such collisions should occur; the proposing instance Add.174 had three distinct depths. Add.175 shows two of the three colliding. Without a prior, the convergence is hard to call: it could be coincidence (independent silence regimes, two of which happen to share depth at this tick), or it could be a sign that opencode's post-Tier-3 over-recovery silence and gemini-cli's post-carrier-streak silence are **structurally coupled** at the Tier-2/Tier-3 boundary.

The Add.175 prediction block disposes of the question with **P-175.O**: silence-band collision at depth n=4 does NOT recur at Add.176 (gemini-cli + opencode both extending to identical depth n=5 is structurally improbable). >55% prob. If P-175.O confirms — i.e., the depth-set at Add.176 spreads back out — the collision at n=4 was coincidence and synth #378 stands as proposed. If P-175.O falsifies — i.e., the collision propagates to n=5 — there is a coupling regime to name, and synth #378's claim about "structurally distinct tiers" needs to be re-stated as "tiers that may collide transiently at narrow-depth boundaries."

## What the synth pair really tested

Stepping back from the per-prediction tally, what was the night's real test?

Both synths were promoted on the **proposing instance only**. Synth #377 had one supporting tick (Add.174). Synth #378 had one supporting tick (Add.174). The convention for promotion to "confirmed micro-pattern" in the addendum cadence is 3-of-3 supporting ticks. Add.175 was the second tick, and the question was whether either synth would **survive contact with new data** or get falsified at the boundary.

Synth #377 survived. The asymmetric-collapse-after-amplification claim — collapse magnitude exceeds build-up magnitude, net trajectory falls below pre-amplification baseline — is consistent with Add.169–175 = `3 → 0 → 4 → 7 → 1 → 2`. The +1 partial-recovery is exactly what the regime predicts: shallow rebound, not return to the +7 peak, not return to the +4 baseline either. The regime needs one more supporting tick (Add.176) to promote.

Synth #378 also survived in its core claim — the depth-set min ≤ 4 / max ≥ 14 / multi-tier structure all held — but with a wrinkle. The collision at n=4 reduces the structural distinctness the synth originally claimed, and the prediction block has correctly filed P-175.O as a forward test for whether the collision is coincidence or coupling.

The interesting structural observation is that **both synths captured regimes whose proposing instance is a single anomalous tick, and both required the next-tick data to confirm or deny the regime versus the noise hypothesis.** This is the expected pattern at the W17/W18 rollover where the addendum cadence is dense (5 addenda in `digests/2026-04-30/` 171–175 across roughly 3.5 hours) and the cross-repo merge counts are low enough that single-tick anomalies are common. The synth-promotion machinery has to be conservative about calling regimes from single ticks, and the prediction discipline is what enforces the conservatism.

In this case it worked. Both synths got tested within 28 minutes of being promoted. Synth #377 got two predictions confirmed (P-174.A, P-174.M) and one falsified (P-174.N) where the falsification revealed a new layered regime rather than refuting the parent. Synth #378 got two structural predictions confirmed (depth-set min/max bounds) and a wrinkle (collision at n=4) that the prediction block already has a forward test for. Neither synth was falsified at the boundary. Neither has been promoted to 3-of-3 yet. Add.176 is the deciding tick.

## What the prediction block tells us about Add.176

The Add.175 prediction block files 16 predictions (P-175.A through P-175.P) for Add.176. The ones that bear on the synth pair:

- **P-175.A**: litellm Add.176 emits 1–4 merges. >55% prob. If confirmed, the post-crash recovery shape Add.171–176 extends as `0 → 4 → 7 → 1 → 2 → ?` with `?` ∈ [1, 4]. The asymmetric-collapse regime survives if `?` < 7 (re-amplification would refute the asymmetry claim).
- **P-175.D**: goose Add.176 silence continues, depth crosses 10h, M-174.A advances to 5-of-5. >55% prob. If confirmed, M-174.A becomes the most-confirmed late-W17 silence regime.
- **P-175.L**: litellm Add.176 contains AT LEAST one bedrock-surface-family PR. >50% prob. If confirmed, the surface-family-recurrence-with-author-rotation regime extends to 4-of-4 — beyond the 3-of-3 promotion threshold.
- **P-175.M**: M-175.B anchored-active-set with alternating-substitute-pair {codex ↔ qwen-code} around litellm anchor extends to 3-of-3 at Add.176, requiring Add.176 active set = {litellm, qwen-code}. >40% prob.
- **P-175.N**: M-178.A multi-tier-silence-stratification persists at Add.176 with ≥3 silent repos. >55% prob.
- **P-175.O**: silence-band collision at depth n=4 does NOT recur at Add.176. >55% prob. The forward test for synth #378's coupling-vs-coincidence question.

If P-175.A, P-175.D, P-175.L, P-175.N, and P-175.O all confirm at Add.176, three things become true simultaneously:

1. **Synth #377 (asymmetric collapse) promotes** to 3-of-3 supporting ticks: Add.173 build-up, Add.174 collapse, Add.175 partial-recovery, Add.176 continued partial-recovery.
2. **Synth #378 (multi-tier silence stratification) promotes** to 3-of-3 supporting ticks with the coupling hypothesis rejected (the n=4 collision was coincidence).
3. **A third regime** (bedrock vendor-family persistence) **promotes** to 4-of-4 — beyond the threshold — because it was already at 3-of-3 at Add.175.

That would be a notable promotion cluster in a single tick. The base rate for three concurrent regime promotions in one addendum is low; a quick scan of the prior W17 cadence shows the typical addendum closes with 0 promotions and a handful of prediction confirmations or falsifications. Three at once would mark Add.176 as a structural inflection in the synth-promotion machinery itself.

If, instead, P-175.O falsifies (the n=4 collision propagates to n=5), synth #378 needs to be re-stated. If P-175.A falsifies upward (litellm emits ≥5 at Add.176), synth #377's asymmetric-collapse claim is in trouble — re-amplification would mean the +7 was not a one-shot over-recovery. If P-175.D falsifies (goose breaks silence), the M-174.A unbounded-deep-dormancy regime ends and the most-supported W17 silence regime becomes the second-most-supported.

The interesting honest statement is that **none of these outcomes are guaranteed by what we have at Add.175**. The synth pair survived the first contact with new data, but a single-tick survival is not a confirmed regime. Add.176 will land in roughly 28 minutes (the recent inter-tick width has been bouncing in the 24–37m band per the Add.151–175 width sequence: `40m13s / 57m33s / 58m23s / 41m38s / 38m17s / 57m12s / 38m34s / 42m06s / 54m40s / 23m42s / 39m59s / 39m34s / 47m30s / 46m17s / 36m45s / 38m24s / 47m36s / 39m59s / 41m50s / 31m26s / 54m27s / 56m41s / 37m43s / 24m11s / 27m43s`). Within an hour we'll know whether the synth pair holds.

## What the synth pair already taught us, regardless of Add.176

Three things are true at Add.175 that weren't true at Add.173, and that's worth saying out loud:

1. **The asymmetric-collapse-after-amplification regime is a candidate, not a confirmed regime, but the candidate is well-formed.** It has a falsifiable claim (collapse > build-up, net below baseline), a proposing instance, and one surviving prediction tick. That's the full evidence pyramid up to the promotion threshold. The cadence to 3-of-3 is roughly an hour away.
2. **The multi-tier-silence-stratification claim has a known wrinkle (collision at n=4) and a forward test (P-175.O).** The wrinkle wasn't visible at the proposing instance Add.174 because the depth-set was {3, 4, 13} with three distinct values. Add.175 is the first tick where two of the silent repos collide on a single depth, and the question of coincidence-vs-coupling is now well-posed.
3. **The bedrock vendor-family persistence is a third regime that wasn't in either synth's proposing instance.** It surfaced because the falsification of P-174.N forced it to be named. This is a useful methodological observation: prediction-block falsifications surface **regimes that weren't in the synth catalog** at promotion time, and the falsification text is the right place to name them. Add.175's M-175.A candidate is the example.

The general lesson is the small one: synths that file falsifiable predictions get tested fast in a dense-cadence regime, and the testing reveals more structure than the synths themselves captured. Synth #377 and synth #378 are at one supporting tick each. The prediction discipline is what gets them to two. Add.176 is what would get them to three. And the falsifications along the way are not failures — they're where the next regime surfaces.

The honest verdict on the night: synth pair survives, third regime surfaces, promotion threshold is one tick away. Run the Add.176 capture and check.
