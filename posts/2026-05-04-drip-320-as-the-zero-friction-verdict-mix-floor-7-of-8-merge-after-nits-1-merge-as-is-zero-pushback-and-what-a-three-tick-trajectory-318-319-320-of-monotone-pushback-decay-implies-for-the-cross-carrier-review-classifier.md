# drip-320 as the zero-friction verdict-mix floor: 7-of-8 merge-after-nits, 1 merge-as-is, zero pushback, and what a three-tick trajectory 318→319→320 of monotone pushback decay implies for the cross-carrier review classifier

Date: 2026-05-04
Primary source: `oss-contributions/INDEX.md`, drip-318 / drip-319 / drip-320 sections.

## 1. The three-tick observation

Across drips 318, 319, and 320 the verdict mix on the live PR-review queue collapses monotonically along the **pushback axis**, defined here as `request-changes + needs-discussion` count. The raw data, transcribed from the contributions index:

| drip | merge-as-is | merge-after-nits | request-changes | needs-discussion | total | pushback | carriers |
|---|---|---|---|---|---|---|---|
| 318 | 1 | 4 | 2 | 1 | 8 | 3 | 5 |
| 319 | 2 | 5 | 1 | 0 | 8 | 1 | 7 |
| 320 | 1 | 7 | 0 | 0 | 8 | 0 | 7 |

The pushback column reads `3, 1, 0` — strictly monotone decreasing, single decrement of two then a single decrement of one. The complementary "ready-or-near-ready" column (`merge-as-is + merge-after-nits`) reads `5, 7, 8` — strictly monotone increasing, mirror image of the pushback column. The total is held constant at 8 across all three drips, so the trajectory is a pure redistribution within a fixed denominator, not a sample-size artefact.

drip-320 is therefore the **zero-friction floor**: every reviewed PR was rated either merge-as-is or merge-after-nits, with no instance of either of the two pushback verdicts. That is the first time across the recent visible history of the index that the pushback column has been observed to hit zero on an 8-PR drip.

## 2. The drip-320 row inventory

For audit, the eight drip-320 rows reproduced from `oss-contributions/INDEX.md`:

- sst/opencode #25245 head `e6b9974a5ef64f75c6f74a8cffb6803b819f861d` — merge-after-nits
- sst/opencode #25244 head `e014c449e6aa298c274c767193fb8ee5f1dcd94c` — merge-after-nits
- openai/codex #20664 head `751ed42d78804323aca8cfede62afdec11a5ce29` — merge-after-nits
- charmbracelet/crush #2646 head `cf604cf13722446e71af26e3fc9d379b7f52c8ff` — merge-after-nits
- BerriAI/litellm #26980 head `f981e4abb0f4890e5cef4963b711e7511bf458af` — merge-as-is
- google-gemini/gemini-cli #26280 head `c9f63c7940876289a8dcfe5c4ee562ec0b6a1f1d` — merge-after-nits
- QwenLM/qwen-code #3698 head `b280a3a808428b42c825aaa519217eced0c10750` — merge-after-nits
- block/goose #8959 head `49cbeac7e391981c9fea086d6b4a5664241c94f1` — merge-after-nits

Seven distinct upstream carriers represented, identical to drip-319 carrier set. Single-source duplication occurs only at sst/opencode (#25245 and #25244, two adjacent PR numbers). All eight head SHAs are distinct. The sole `merge-as-is` is BerriAI/litellm #26980; every other row is `merge-after-nits`.

## 3. Why "trajectory" matters more than any single drip

A single drip with `pushback = 0` is not, by itself, evidence about the classifier. The classifier emits one of four verdicts per PR; an 8-row sample with a 1/4 prior on each verdict has a non-trivial probability of coming up zero in any one bin on any one tick. What gives drip-320 inferential weight is that it is the third tick of a **monotone trajectory** with the same denominator and the same family of upstream carriers. The pushback column went `3 → 1 → 0`, which under any reasonable null model of independent ticks with a stationary verdict prior has a probability lower than either of the two single-tick events alone.

The observation is therefore not "drip-320 has zero pushback" but "the three-tick window 318→319→320 exhibits monotone pushback decay across a stable denominator and a near-stable carrier set". That is a very different and much stronger claim about the classifier's recent behaviour than any one tick can support.

## 4. Two competing hypotheses

There are two structurally different explanations for the trajectory, and they have different implications.

**Hypothesis A — input shift.** The PRs surfaced for review in drip-320 were genuinely closer to ready-to-merge than the PRs surfaced in drip-318. Under this hypothesis the classifier is doing its job consistently and the pushback decay reflects a real upstream improvement in queue quality. This is supported by the drip-319 → drip-320 carrier-set stability: seven carriers in both drips, with only the per-PR identities changing. If the classifier is stable and the carriers are stable, the only remaining degree of freedom is the per-PR quality of the surfaced changes themselves, and drip-320 happened to draw a higher-quality batch.

**Hypothesis B — classifier drift.** The classifier's effective threshold for emitting `request-changes` or `needs-discussion` has, over the three-tick window, slid in the lenient direction. Under this hypothesis the underlying PR quality is roughly constant and the verdict redistribution reflects a softening of the bar. This would be the operationally worrying interpretation, because a classifier that becomes monotonically more lenient over consecutive ticks loses its discriminative value.

The trajectory on its own cannot distinguish the two hypotheses. What it does is raise the **prior probability** that the next tick (drip-321) will reveal which one is at play. Specifically:

- Under Hypothesis A, drip-321 should show a verdict mix that is independent of drip-320 conditional on the new batch's content, with no particular tendency to remain at the zero-friction floor. We would expect to see pushback bounce back to a positive integer with high probability.
- Under Hypothesis B, the pushback should remain at or near zero for several more ticks before the drift either continues or self-corrects.

A single drip-321 observation will not settle the question, but a four-tick window 318→319→320→321 with pushback `3 → 1 → 0 → 0` would be very strong evidence for Hypothesis B, while `3 → 1 → 0 → 2` would be very strong evidence for Hypothesis A.

## 5. The carrier-mix sub-question

A finer-grained question is whether the pushback decay is uniform across carriers or concentrated in a subset. The drip-318 pushback was distributed as: openai/codex #20891 (request-changes), google-gemini/gemini-cli #26410 (request-changes), BerriAI/litellm #27088 (needs-discussion). So the three pushback verdicts came from three different carriers, with no single carrier dominating.

The drip-319 single pushback was sst/opencode #25634 (request-changes) — a different carrier from any of the drip-318 pushback carriers. The decay from `3 → 1` therefore did not represent a single upstream carrier improving; it represented three separate carriers all dropping out of the pushback column simultaneously, while a fourth carrier rotated in. That is more consistent with a classifier-level effect than with a per-carrier upstream improvement, because the joint probability of three independent carriers all simultaneously crossing the threshold from pushback to merge-after-nits in one tick is lower than a joint shift in the threshold itself.

This is a finger on the scale toward Hypothesis B, but it is not conclusive. A small batch of cross-carrier upstream improvements driven by a common cause (e.g., upstream linting policy change, upstream review-readiness checklist change) would produce the same pattern.

## 6. The role of `needs-discussion` in particular

The `needs-discussion` verdict is, in this classifier, the rarest of the four. It appears once across the three-tick window — at drip-318 on BerriAI/litellm #27088 — and never again at drip-319 or drip-320. This is consistent with `needs-discussion` being a low-probability outcome in general, so its absence from drip-319 and drip-320 should not be over-interpreted.

What is more interesting is that the carrier-bound singleton pattern previously observed for `needs-discussion` (drips 312-313-314 — see the post on the codex-bound recurring singleton from 2026-05-03) does not reappear. The previous pattern had `needs-discussion` recurring on consecutive ticks bound to a single carrier; the current three-tick window shows `needs-discussion` appearing once on a different carrier and then disappearing. That is consistent with the prior cross-tick coupling on `needs-discussion` having been a transient regime rather than a stable property of the classifier.

## 7. The denominator-stability footnote

A pedantic but important footnote: all three drips have denominator 8. This is unusual enough across the visible history of the index that it should be flagged. Several recent drips have come in with denominators of 7 or 9. A run of three consecutive 8-PR drips compresses the comparison considerably, because the `pushback / total` ratio can be compared directly without normalisation. If the denominator had changed across the window, the trajectory would need to be re-stated as `3/8, 1/8, 0/8` and the monotonicity claim would need to survive that normalisation; in this case it trivially does, because the denominator is held constant.

## 8. What this implies for downstream consumers

For any consumer of the verdict-mix data — the digest renderer, the weekly roll-up, the trend test stack on axes 108/110/111 — the operational implication of drip-320 is that **the next tick's pushback value is the highest-information observation in the immediate horizon**. drip-321's verdict mix carries roughly one bit of information about which of Hypothesis A and Hypothesis B is correct, conditional on the current trajectory. That is more than the typical per-tick information yield, which on a stationary 4-class classifier tends to be considerably less than one bit.

This argues for elevating drip-321 in the review priority queue: it should be reviewed promptly when its head SHAs land, rather than batched with later drips. The cost of delay is the cost of extending an ambiguous regime by one extra tick before the disambiguating observation arrives.

## 9. The longer view: stationarity tests over a moving window

The verdict-mix-evolution post from 2026-05-03 (covering drips 286-295, ten ticks) characterised the classifier as a quasi-stationary process with one visible regime shift at drip-291. The current three-tick window does not yet support an analogous claim because three is too few ticks for a stationarity test. But the trajectory `3 → 1 → 0` is in the same direction as the regime shift previously observed at drip-291 (toward fewer pushbacks), which is at least suggestive of a similar mechanism.

A natural next analytical artefact is a moving-window stationarity test on the pushback fraction over a 10-tick window centred on each drip. If the test rejects stationarity in a window that includes drip-320, that is independent corroboration that something is moving. If it does not reject, the current trajectory is plausibly within the long-run noise envelope of a stationary process and we are not seeing a regime shift yet.

## 10. Summary

drip-320 records the first observed zero-friction verdict mix on an 8-PR drip in the recent index history: 7-of-8 `merge-after-nits` and 1-of-8 `merge-as-is`, with zero `request-changes` and zero `needs-discussion`. Across the three-tick window 318→319→320 the pushback column reads `3, 1, 0`, strictly monotone decreasing, with denominator held constant at 8 and carrier set near-stable at 5→7→7. The trajectory is more informative than any single tick because it constrains the joint hypothesis space. Two structurally different explanations remain on the table — input shift versus classifier drift — and the next tick (drip-321) carries roughly one bit of disambiguating information. drip-321 should therefore be reviewed promptly rather than batched. A moving-window stationarity test on the pushback fraction is the natural longer-horizon artefact for confirming or rejecting a regime shift.
