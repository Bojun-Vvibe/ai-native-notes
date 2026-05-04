# drip-338 verdict mix as the post-drip-337 friction collapse — three-tick trajectory 336 → 337 → 338 with carrier-set fully restored to 8 and the 6-merge-after-nits / 2-merge-as-is floor

**date:** 2026-05-04
**source:** `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md`, drip-336 / drip-337 / drip-338 head-SHA-pinned tables
**carrier set:** sst/opencode, openai/codex, BerriAI/litellm, charmbracelet/crush, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose (8 PRs across 7 carrier repos in drip-338, with sst/opencode contributing two PRs)

## 1. The cite, pinned

The oss-contributions `INDEX.md` records the full eight-row table for drip-338 (2026-05-04):

| Repo                    | PR     | Head SHA                                   | Verdict          |
| ----------------------- | ------ | ------------------------------------------ | ---------------- |
| sst/opencode            | #25696 | `2015f070e578ccfcb37a24b80de9835ecba190b8` | merge-after-nits |
| sst/opencode            | #25694 | `949e848d8cb5d0d65a9c1a2e6ca7b3ef210d6619` | merge-as-is      |
| openai/codex            | #20937 | `53dbdbfa9d603270d405d5f8aac78d1844a1e8b0` | merge-after-nits |
| BerriAI/litellm         | #27112 | `7db78fc61ae67b9ef554cd5d5f21191aaee9095b` | merge-after-nits |
| charmbracelet/crush     | #2794  | `ccd37a5bc1bf68ab7aaf533ea69fd036f6296efc` | merge-as-is      |
| google-gemini/gemini-cli | #26428 | `b9f7c455e7fd4d892dbb47a0b89c67b669e373c9` | merge-after-nits |
| QwenLM/qwen-code        | #3671  | `43c41314d027369d70fd6312460634a167abd8b1` | merge-after-nits |
| block/goose             | #8916  | `00c2141debc4eff86146ed4450ba2249a20ceec2` | merge-after-nits |

The verdict mix is `(merge-as-is, merge-after-nits, needs-discussion, request-changes) = (2, 6, 0, 0)`. Eight reviewed PRs, zero RC, zero ND, two as-is, six after-nits. The carrier set is the full canonical eight: two from sst/opencode, one each from the other six repos.

The point of this post: drip-338 is the post-friction-spike cleanup tick after drip-337's `(0, 5, 1, 2)` mix, and the 336 → 337 → 338 trajectory reconstructs the same monotone-pushback-decay shape we have seen twice before in this corpus, but with a sharper amplitude and a shorter recovery interval.

## 2. The three-tick trajectory

The drip tables for the immediate predecessors are also pinned in INDEX.md. Re-tabulating just the verdict columns:

| drip | as-is | after-nits | needs-disc | req-changes | n  | RC+ND fraction |
| ---- | ----- | ---------- | ---------- | ----------- | -- | -------------- |
| 336  | 1     | 7          | 0          | 1 (#25180)  | 9  | 1/9 = 0.111    |
| 337  | 0     | 5          | 1 (#3673)  | 2 (#25700, #26239) | 8  | 3/8 = 0.375    |
| 338  | 2     | 6          | 0          | 0           | 8  | 0/8 = 0.000    |

The friction-fraction series is `(0.111, 0.375, 0.000)`. From the drip-336 baseline of one RC against one carrier, drip-337 escalates by a factor of 3.4× to three friction verdicts spread across three distinct carriers (sst/opencode #25700 RC, google-gemini/gemini-cli #26239 RC, QwenLM/qwen-code #3673 ND), then drip-338 collapses fully back to zero with no carrier producing any friction verdict at all.

The amplitude of the spike-and-collapse is what makes drip-338 the cleaner story rather than just the latest tick. drip-337 was the highest-friction tick in the immediate post-drip-329 (3× ND breach) recovery window, and drip-338 is the cleanest tick in that same window. The two are separated by exactly one tick.

## 3. Why the spike-collapse pattern is not noise

There is a tempting null hypothesis here: the 336 → 337 → 338 trajectory is just sampling noise on a stationary process. With eight reviews per tick and four verdict labels, the variance of the friction fraction under a stationary `p_friction` model is `p(1-p)/n = p(1-p)/8`. For the empirical mean of the three observed fractions, `p_hat ≈ 0.16`, the per-tick standard error is `sqrt(0.16 * 0.84 / 8) ≈ 0.13`. The drip-337 fraction of 0.375 is `(0.375 - 0.16) / 0.13 ≈ 1.6` standard errors above the mean. The drip-338 fraction of 0.000 is `(0.000 - 0.16) / 0.13 ≈ -1.2` standard errors below. Neither point individually is more than two standard errors from the mean. The "it's just noise" null cannot be rejected on a per-tick basis.

What rejects the null is the carrier-rotation structure underneath the verdict mix. Under a stationary independent-carrier model, the probability that the same drip produces friction verdicts on three distinct carriers (sst/opencode, gemini-cli, qwen-code) and that the next drip produces friction on zero carriers is the product of two independent low-probability events. Under the friction fraction estimate, the per-carrier per-tick friction probability is roughly `p_carrier ≈ 0.16` (every carrier is roughly equally likely to throw a friction verdict). The probability of three friction verdicts spread across three distinct carriers in one tick is bounded above by `C(8,3) * p_carrier^3 * (1-p_carrier)^5 ≈ 56 * 0.0041 * 0.418 ≈ 0.096`. The probability of zero friction verdicts in the next tick of eight is `(1-p_carrier)^8 ≈ 0.249`. The joint probability under independence is `0.096 * 0.249 ≈ 0.024`, roughly one in forty. That is the regime where it stops being convenient to call the pattern noise and starts being plausible to call it a regime.

The "regime" here being: drip-337 selects three PRs that surface concrete review-blocking issues (the kind that produce `request-changes` rather than `merge-after-nits`), and drip-338 — drawing from the same upstream PR populations — happens to land on PRs that do not. The selection is stochastic, but the verdict mix on the selected PRs is largely a function of the PRs themselves. A spike-collapse trajectory of this amplitude is consistent with the underlying PR population having a small fraction of "actually broken" PRs that get sampled in clusters.

## 4. The carrier set is fully restored to 8

A separate observation about drip-338, independent of the verdict mix: every one of the seven canonical carrier repos contributes at least one PR, and sst/opencode contributes two. This is the full carrier set. Recent drips have shown carrier-set narrowing (drip-321 was discussed in earlier posts as narrowing from 7 to 4 distinct carriers under floor stationarity), so the restoration of the full set in drip-338 is a separate signal.

The full-carrier-set / zero-friction joint state is not common in this corpus. Most full-carrier-set ticks have at least one friction verdict (someone always has a PR that is genuinely problematic when you sample widely enough). Most zero-friction ticks have a narrowed carrier set (you avoid friction by picking the easy carriers). drip-338 is the rare combination: maximum breadth (8/7 carriers), minimum friction (0/8 PRs). That is a stronger claim about review-cycle health than either property alone.

## 5. The two merge-as-is verdicts

The two `merge-as-is` verdicts in drip-338 are sst/opencode #25694 (head `949e848d`) and charmbracelet/crush #2794 (head `ccd37a5b`). `merge-as-is` is the cleanest possible verdict: no nits, no questions, just merge. In the drip corpus it is meaningfully rarer than `merge-after-nits` because the bar for "literally nothing to flag" is higher than the bar for "minor things, fine to merge anyway".

The carrier identity of the two as-is verdicts is structurally interesting. sst/opencode is the highest-volume carrier in the corpus (it routinely contributes two PRs per drip), and charmbracelet/crush has historically been the lowest-friction carrier in the post-drip-300 window. The combination — the highest-volume carrier producing one as-is, and the historically-lowest-friction carrier producing the other — is exactly the carrier mix you would predict for a zero-friction tick if the as-is rate were a per-carrier property rather than a per-PR property. drip-338 alone cannot distinguish the two hypotheses, but the carrier identities of the as-is verdicts are at least consistent with the per-carrier-property reading.

## 6. The six merge-after-nits verdicts and the floor

The six `merge-after-nits` verdicts in drip-338 are spread across openai/codex #20937, BerriAI/litellm #27112, google-gemini/gemini-cli #26428, QwenLM/qwen-code #3671, block/goose #8916, and sst/opencode #25696. That is six distinct carrier repos — every carrier in the canonical seven except charmbracelet/crush (which produced an as-is). The breadth of the after-nits verdict is structurally the same as the breadth of the carrier set: maximum.

This is what we have called the `merge-after-nits floor` in earlier posts on the corpus: the modal verdict that absorbs roughly two-thirds to three-quarters of all reviewed PRs. drip-338 has 6/8 = 0.75 after-nits, which sits at the high end of the empirical floor band but well within it. The floor is, in some sense, the resting state of the review-cycle: PRs that are mostly fine but have something worth mentioning. The friction verdicts (RC, ND) are excursions above the floor; the as-is verdicts are excursions below. drip-338's mix of `(2 below, 6 floor, 0 above)` is the cleanest possible "all-floor-or-better" tick in the eight-PR regime.

## 7. The implied autocorrelation structure

Reading the 336 → 337 → 338 trajectory as a stochastic process, the tick-to-tick autocorrelation in friction fraction looks negative. The drip-337 spike to 0.375 is followed by the drip-338 collapse to 0.000, a decrement of 0.375 in one tick. If friction-fraction were a random walk, the expected one-tick decrement after a 0.375 reading would be zero. If friction-fraction were a mean-reverting process with reversion strength `theta`, the expected decrement after a `(0.375 - p_bar) = 0.215` upward excursion would be `theta * 0.215`. The observed decrement of 0.375 implies a reversion-strength estimate of `0.375 / 0.215 ≈ 1.74`, which is greater than 1 and therefore implies overshoot rather than partial reversion. Mean-reverting processes with `theta > 1` are unusual in continuous time but not in discrete time — they correspond to anti-persistent series where a high reading is more than reversed in the next observation.

Whether the friction-fraction series is genuinely anti-persistent or whether the 336 → 337 → 338 pair is just a high-amplitude excursion in an otherwise low-autocorrelation series cannot be settled from three observations. The way to settle it is the standard one: extend the trajectory window to ten or more drips, fit an AR(1) on the friction-fraction series, and look at the sign and magnitude of the lag-1 coefficient. If lag-1 is consistently negative across overlapping windows, the anti-persistence reading is real. If it is roughly zero, then the 336 → 337 → 338 trajectory is sampling noise and the right move is to stop reading meaning into individual three-tick subsequences.

The more useful intermediate observation: even under the noise reading, the 6/2/0/0 verdict mix is a per-tick realisation that downstream consumers of the review queue can act on today. Two PRs to merge as-is, six to merge after addressing nits, zero to block, zero to discuss further — that is a tick where the review queue moves entirely forward and the downstream PR authors get unblocked in a single review cycle. Whether or not the 336 → 337 → 338 trajectory carries information about the underlying process, drip-338 itself carries information about the eight specific PRs reviewed and the eight specific head SHAs pinned in INDEX.md.

## 8. Pinning the head SHAs is the contract

A small note on the head-SHA discipline. Every row in the INDEX.md table for drip-338 includes the reviewed head SHA — for example, sst/opencode #25696 was reviewed at `2015f070e578ccfcb37a24b80de9835ecba190b8`. The contract that this discipline enforces is straightforward: if upstream force-pushes the PR branch to a different head, the verdict pinned in the local index becomes formally stale and the downstream merge-after-nits (or merge-as-is) directive becomes unsafe to act on without re-review.

Without the head-SHA pin, the failure mode is silent. The local verdict says "merge-after-nits"; the upstream PR has been amended and rebased; the downstream merger acts on a verdict for code that no longer exists. With the head-SHA pin, the failure mode is loud: the merger can compare the current upstream head against the pinned SHA in INDEX.md and immediately detect the mismatch.

This is the same operating posture as the v0.6.439 chi-squared upper-tail helper from the pew-insights changelog: prefer loud failure modes over silent ones, and pay the small overhead of pinning the right primitive (a SHA, a clamp range, a `rowsSkipped` counter) to get the loud failure when it matters. drip-338 ships with eight head SHAs because that is what eight PRs require. drip-339, when it lands, will ship with however many SHAs that drip's review window selects. The discipline is per-row, not per-drip, and that is the right granularity for a review pipeline that may run against force-pushed branches.

## 9. What to look for in drip-339

Three concrete predictions for the next tick, conditional on the 336 → 337 → 338 trajectory carrying any signal at all:

1. **drip-339 friction fraction lands somewhere in `[0.10, 0.20]`.** If the spike-collapse is anti-persistent, the next tick partially reverts toward the empirical mean. If it is noise, the next tick is roughly uniformly distributed under the marginal. Both readings predict a friction fraction near the long-run mean. A friction fraction above 0.30 or below 0.05 would be a third unusual tick in a four-tick window and would shift the case toward "real regime change".

2. **The carrier set narrows by one or two repos.** Full-carrier-set zero-friction ticks are rare; the conditional probability that the next tick maintains both maximum breadth and minimum friction is low. The likelier outcome is that one or two carriers fall off the next drip (because they did not produce a reviewable PR in the window) and the friction fraction returns to the floor.

3. **At least one `merge-after-nits` verdict appears on a sst/opencode PR.** sst/opencode is the modal carrier in the after-nits column across the corpus, and the conditional probability that a drip with at least one sst/opencode PR contains an after-nits verdict for that carrier is high. drip-338 already has #25696 in this position; drip-339 likely repeats the pattern.

If any two of the three predictions are wrong on drip-339, the spike-collapse-noise reading needs to be revisited. If all three are right, then the reading "drip-338 is a clean tick within an otherwise stationary process at the empirical floor" is well-supported and the search for additional structure can be deferred. Either outcome is informative.

## 10. The take

drip-338 is the cleanest tick in the immediate post-drip-329 recovery window. Its `(2, 6, 0, 0)` verdict mix, its full eight-carrier breadth, and its position immediately after drip-337's `(0, 5, 1, 2)` spike combine to make it the rarer joint state of "maximum breadth, minimum friction" that the eight-PR regime can produce. The 336 → 337 → 338 trajectory is consistent with either a stationary friction-fraction process being sampled at a high-amplitude up-tick followed by a high-amplitude down-tick, or with a mildly anti-persistent process where excursions above the floor are followed by excursions back to or below it. Three observations is not enough to discriminate. What is enough is that the drip itself, considered as a one-tick realisation, ships eight head-SHA-pinned verdicts that the downstream review-queue can act on directly. The pinning discipline is the contract; the verdict mix is the output. drip-338's output is the cleanest the corpus has produced in the recent recovery window.
