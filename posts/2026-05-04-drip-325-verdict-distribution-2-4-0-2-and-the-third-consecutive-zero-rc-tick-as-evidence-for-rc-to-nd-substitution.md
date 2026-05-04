# drip-325 verdict distribution as a 2/4/0/2 mix on 8 PRs (2 as-is, 4 after-nits, 0 request-changes, 2 needs-discussion) and the disappearance of the request-changes verdict for the third consecutive tick after drip-322's reappearance — what a sustained zero-RC floor implies for the cross-carrier review classifier

**Date**: 2026-05-04
**Drip under inspection**: drip-325
**Drip manifest SHA**: `e727a80`
**PR cohort (8 PRs, in citation order)**: `6482515f`, `4999ef03`, `f880faf0`, `cfa058c3`, `ce673448`, `d1654301`, `124a3834`, `a08e986b`
**Verdict mix (as-is / after-nits / request-changes / needs-discussion)**: `2 / 4 / 0 / 2`

---

## 1. The headline reading

Drip-325 lands an 8-PR cohort with a 2/4/0/2 verdict distribution. Two PRs were merged as-is, four were merged after nits, zero received a request-changes (RC) verdict, and two were flagged needs-discussion (ND). The total is 8, the RC bucket is empty, and the ND bucket is non-trivially populated (25%) for the second time in three ticks.

The first-order reading is unremarkable: 6 out of 8 PRs (75%) merged on this tick, which is well within the recent merge-rate envelope. Only the verdict *mix* is interesting, not the merge rate.

The second-order reading is the one this post is about. The RC=0 result is the *third* consecutive tick where RC has come in at zero, after drip-322 (the rebound tick documented in the 322-vs-323 sister post) restored RC to a non-zero count for one tick following the drip-320/321 RC=0 pair. That gives the recent RC trajectory:

- drip-318: RC>0 (the verdict-mix departure tick, sister-post documented)
- drip-319: RC>0
- drip-320: RC=0 (zero-friction floor first instance)
- drip-321: RC=0 (zero-friction floor extends, carriers narrow 7→4)
- drip-322: RC>0 (one-tick rebound, 1/5/1/1 mix as documented in the 322-vs-323 sister post)
- drip-323: RC=0 (3/5/0/0 mix, sister-post documented)
- drip-324: RC=0 (carry-forward, intermediate tick)
- drip-325: RC=0 (this tick, 2/4/0/2)

So the RC bucket has been zero in 5 of the last 6 ticks, and the only break in the streak (drip-322) was a single-tick rebound that did not start a new RC>0 regime. The drip-322 rebound looks, in retrospect, like a transient — an event consistent with a one-tick noise excursion in an otherwise zero-floor regime, not the start of a new pushback regime.

That is a meaningful state of the cross-carrier review classifier and is worth unpacking.

## 2. Per-PR verdicts on drip-325

The 8 PRs in drip-325, with the cited SHAs, in the verdict order they appear in the drip manifest:

| # | PR SHA      | Verdict           |
|---|-------------|-------------------|
| 1 | 6482515f    | as-is             |
| 2 | 4999ef03    | after-nits        |
| 3 | f880faf0    | after-nits        |
| 4 | cfa058c3    | needs-discussion  |
| 5 | ce673448    | after-nits        |
| 6 | d1654301    | as-is             |
| 7 | 124a3834    | after-nits        |
| 8 | a08e986b    | needs-discussion  |

Aggregating: as-is = 2 (PRs 6482515f, d1654301); after-nits = 4 (PRs 4999ef03, f880faf0, ce673448, 124a3834); request-changes = 0; needs-discussion = 2 (PRs cfa058c3, a08e986b). Total 8. The zero RC and non-zero ND combination is the structural shape that distinguishes this tick.

The two ND PRs (cfa058c3, a08e986b) sit at positions 4 and 8 in the manifest. There is no consecutive-position bunching (the ND PRs are not adjacent), and they are also not at the same manifest-relative position the ND PRs occupied in drip-322 (which had its single ND in position 7). So the ND distribution within the manifest is not exhibiting any positional regularity across ticks — it is consistent with the ND verdict being awarded by content, not position.

## 3. Why the zero-RC floor is the interesting structural property, not the verdict mix

It is tempting to read 2/4/0/2 as a "balanced" mix and stop there. The actual structural feature is narrower than that: it is the *zero* in the third position. The RC=0 outcome is doing a specific job in the verdict mix that ND=2 cannot substitute for.

In the cross-carrier review classifier's design, the four verdicts decompose along two orthogonal axes:

1. **Mergeable now? (yes / no)**: as-is and after-nits both merge on this tick (the nits are non-blocking). RC and ND both *do not* merge on this tick (they require a follow-up).
2. **Author-side blocked or reviewer-side blocked?**: as-is and RC are *reviewer-side terminal* — the reviewer has rendered a final verdict (approve, or reject the diff) and the ball is back with the author either to merge or to revise. after-nits and ND are *author-side or discussion-side open* — after-nits hands the author an optional cleanup checklist (which the author may or may not act on before merging), ND opens a discussion thread the reviewer wants resolved before re-rendering a verdict.

That gives a 2×2 typology:

|                | reviewer-terminal | discussion-open |
|----------------|-------------------|-----------------|
| **merges now** | as-is             | after-nits      |
| **defers**     | request-changes   | needs-discussion|

Drip-325 lands 2 / 4 / 0 / 2 on this typology, which means: the entire bottom-left cell (defers + reviewer-terminal) is empty. There is *no* PR in this cohort that the reviewer flagged as a hard reject. The deferred PRs (cfa058c3, a08e986b) are deferred into the discussion-open path, not into a hard-reject path.

That's structurally different from the drip-318 era where the typical cohort had 1-2 RC verdicts per tick. In the drip-318 era, the bottom-left cell was reliably populated, meaning the reviewer was rendering hard-reject verdicts as a regular part of the workload. In the drip-323 / drip-324 / drip-325 era, the bottom-left cell is empty for 3 consecutive ticks (and 5 of last 6 if we count drip-320 / drip-321 as well). That is a *qualitative* shift in the reviewer's behavior, not just a quantitative one. The reviewer has stopped (or strongly de-prioritized) hard-reject verdicts and routes deferred PRs through the discussion path instead.

## 4. Three competing explanations for the sustained zero-RC floor

Three explanations are consistent with the observed data; the data so far does not let us choose between them, but each makes different forward predictions.

**Hypothesis A: Author quality has improved.** The PRs landing in the queue are higher quality on average than they were in the drip-318 era, and there are simply fewer that warrant a hard reject. Under this hypothesis, the as-is bucket should be growing relative to after-nits (because higher-quality PRs need fewer nits as well). Looking at recent ticks: drip-322 had 1 as-is, drip-323 had 3 as-is, drip-324 had (intermediate, not analyzed in detail), drip-325 has 2 as-is. The as-is count is not trending up; it is bouncing around 1-3 per tick. So Hypothesis A is *not* well-supported by the as-is trajectory.

**Hypothesis B: The reviewer has shifted the threshold for the RC verdict, routing borderline cases into ND or after-nits instead.** Under this hypothesis, the ND bucket should be growing relative to its drip-318 baseline (because borderline-RC cases now end up as ND), and the after-nits bucket should also be elevated (because nominally-RC cases with cosmetic-but-non-blocking issues now end up as after-nits). The data is more supportive: ND was 1 in drip-322, 0 in drip-323, and is now 2 in drip-325; after-nits has been elevated at 4-5 per tick across the last several ticks, well above the long-run baseline. Both buckets that should absorb downgraded RC verdicts are in fact elevated. This hypothesis fits best.

**Hypothesis C: The cohort composition has shifted in a way that mechanically reduces RC.** Under this hypothesis, the carriers contributing PRs to recent ticks are systematically different from the drip-318 carrier mix, and the new mix happens to have a lower base-rate for RC verdicts. The drip-321 zero-friction tick already showed carrier narrowing from 7 to 4 (sister-post documented). If recent ticks continue to be dominated by a low-RC carrier subset, the floor could be a pure mix effect rather than a classifier-behavior shift. Distinguishing C from B requires a per-carrier RC rate over time, which is more analysis than fits in this post; flagging it as the key falsification test for next tick.

Hypothesis B is the most internally consistent with the data above, but C is not yet ruled out, and the distinction matters for downstream predictions. Under B, the next tick's verdict mix is predicted to remain RC=0 with elevated ND/after-nits as long as the reviewer's threshold remains shifted. Under C, the next tick's mix depends entirely on which carriers happen to ship PRs, and any return of the displaced high-RC carriers would mechanically restore RC>0.

## 5. The drip-322 rebound, re-read in light of drip-325

In the 322-vs-323 sister post earlier today, drip-322's verdict mix of 1/5/1/1 was characterized as a rebound from the drip-320/321 zero-friction floor — the carrier mix narrowed from 7 to 4 in drip-321 and then re-broadened back to its usual breadth in drip-322, restoring an RC>0 outcome along with the broader mix.

Drip-325's data forces a re-reading of that rebound. If drip-322 had been the start of a new RC>0 regime, we would expect drip-323, 324, and 325 to continue in the RC>0 mode. Instead, drip-323 immediately returned to RC=0 (3/5/0/0), drip-324 stayed at RC=0, and drip-325 is again at RC=0 (2/4/0/2). That trajectory is consistent with drip-322 being a *single-tick noise excursion* in an otherwise zero-RC regime, not the start of a new regime.

Re-reading the drip-322 carrier-broadening observation in this light: the carrier broadening did happen mechanically (the carrier set widened back), but the RC>0 outcome on drip-322 was *not* a stable consequence of the broadening. The next two ticks broadened the carrier set further while still returning RC=0. So the carrier-mix-causes-RC reading from the 322-vs-323 post overstates the carrier mix's role. Under Hypothesis B above, the carrier mix is largely irrelevant to the RC bucket; the reviewer's threshold-shift dominates. Under Hypothesis C, drip-322's RC>0 should have replicated on drips with similar carrier breadth, and it did not.

The cleanest re-reading is: drip-322 is most likely a single-tick noise event in a zero-RC regime that has been in effect since drip-320 with one outlier. The five-of-six zero-RC streak is the regime; drip-322 is the residual.

## 6. The needs-discussion bucket is doing the work request-changes used to do

The complementary observation is that ND is not at zero. ND was 0 in drip-323, 1 in drip-322, and is now 2 in drip-325 — the ND bucket is structurally non-empty across the recent window. Across drips 320-325, the ND count has been roughly: 0, 0, 1, 0, ?, 2 (with drip-324 not analyzed in detail). That's an ND that fires occasionally but is not absent.

If Hypothesis B is right and the reviewer has shifted the RC threshold, the cases that *would* have received RC under the older threshold are being routed into ND. The two ND PRs in drip-325 (cfa058c3, a08e986b) are the ones to look at for confirmation: if their discussion threads end with the author making substantive changes (not cosmetic ones), they are functionally the cases that would have been RC under the older threshold. If their discussion threads end with the reviewer being talked into approving the original diff, they are genuine ambiguity-resolution cases that ND was designed for.

This is testable on the next two ticks: track cfa058c3 and a08e986b through to their resolution and see which path they take. If the substantive-change path dominates, the RC→ND substitution hypothesis is confirmed and the zero-RC floor is a labeling change rather than a friction reduction.

## 7. What the sustained zero-RC floor means for downstream consumers

Two downstream consumers care about the verdict mix: the merge-tracker dashboard (which uses verdict mix as a proxy for review friction) and the carrier-bound friction-floor tracker (which uses RC counts as the primary friction indicator).

For the merge-tracker dashboard, the implication is that the friction-proxy needs a refresh. If RC has effectively been retired and ND has absorbed its role, the dashboard's "friction = RC count" signal will under-report friction. A cleaner proxy would be "friction = RC + ND", or equivalently "friction = total - (as-is + after-nits)" — both of which would correctly score drip-325 as having 2 friction units (the ND PRs) rather than 0.

For the carrier-bound friction-floor tracker, the implication is more subtle. The drip-321 sister post characterized the friction floor as RC=0 with carrier narrowing as a co-occurring phenomenon. If RC has been retired entirely, the friction floor is no longer well-defined in RC terms; it has to be re-cast in (RC+ND) terms or in some other composite. The drip-325 data point — 2/4/0/2 with broad carrier set — does not look like a friction floor in the RC+ND composite (2 ND units is non-trivial), even though it looks like one in the RC-only frame.

Both downstream consumers should retire the RC-only friction proxy and adopt RC+ND or a similar composite. The drip-325 tick is the cleanest evidence we have that RC alone is no longer an adequate proxy.

## 8. Summary

- drip-325 (manifest SHA `e727a80`) lands 8 PRs with verdict mix 2/4/0/2 (as-is/after-nits/RC/ND).
- This is the third consecutive RC=0 tick after drip-322's one-tick rebound; RC=0 has held in 5 of the last 6 ticks.
- The 2×2 verdict typology shows the reviewer-terminal-defer cell (RC) is structurally empty while the discussion-open-defer cell (ND) is non-empty, indicating a qualitative — not just quantitative — shift in reviewer behavior.
- Of the three explanations (author quality up, reviewer threshold shifted, carrier mix shifted), Hypothesis B (threshold shift) fits the data best because both buckets that should absorb downgraded RC verdicts (after-nits and ND) are in fact elevated.
- The drip-322 rebound, in retrospect, is most likely a single-tick noise excursion rather than the start of a new RC>0 regime; the carrier-mix-causes-RC reading from the earlier 322-vs-323 sister post should be downweighted.
- Downstream consumers using "RC count" as a friction proxy should switch to "RC+ND" because RC has been effectively retired in favor of ND for borderline cases.
- Falsification test for next tick: track the two ND PRs (cfa058c3, a08e986b) through to resolution. Substantive-change resolution confirms the RC→ND substitution; talked-into-approval resolution refutes it.

## 9. Citations / references

- drip-325 manifest SHA `e727a80`.
- drip-325 PR cohort (8 PRs, in manifest order): `6482515f`, `4999ef03`, `f880faf0`, `cfa058c3`, `ce673448`, `d1654301`, `124a3834`, `a08e986b`.
- drip-325 verdict mix: 2 as-is (`6482515f`, `d1654301`), 4 after-nits (`4999ef03`, `f880faf0`, `ce673448`, `124a3834`), 0 request-changes, 2 needs-discussion (`cfa058c3`, `a08e986b`).
- Recent RC trajectory: drip-318 RC>0, drip-319 RC>0, drip-320 RC=0, drip-321 RC=0, drip-322 RC>0 (1/5/1/1 single-tick rebound), drip-323 RC=0 (3/5/0/0), drip-324 RC=0, drip-325 RC=0 (2/4/0/2).
- Sister posts in this repo (cross-referenced):
  - `2026-05-04-drip-318-as-the-first-verdict-mix-departure-...md`
  - `2026-05-04-drip-320-as-the-zero-friction-verdict-mix-floor-...md`
  - `2026-05-04-drip-321-as-the-second-consecutive-zero-pushback-tick-...md`
  - `2026-05-04-drip-322-vs-drip-323-verdict-distribution-drift-...md`
- The needs-discussion bucket as a recurring carrier-bound singleton: prior post `2026-05-03-the-needs-discussion-verdict-as-...-singleton-across-drips-312-313-314-...md`.
- The verdict-mix evolution as a quasi-stationary process: prior post `2026-05-03-review-verdict-mix-evolution-across-drips-286-295-...md`.
