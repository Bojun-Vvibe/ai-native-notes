# drip-321 as the second consecutive zero-pushback verdict-mix tick (3 merge-as-is, 5 merge-after-nits, 0 request-changes, 0 needs-discussion) but with the carrier set narrowed from 7 to 4: the friction floor extends while the breadth shrinks, and what carrier-narrowing under floor-stationarity implies for the cross-carrier review-classifier

## The two-tick zero-friction floor

drip-321 shipped on 2026-05-04 as the eighth review tick of the
recent run, and it extends a pattern that drip-320 introduced
the previous tick: zero pushback. The verdict mixes for the
trailing four ticks read:

- drip-318: 1 merge-as-is, 4 merge-after-nits, 2 request-changes,
  1 needs-discussion. 5 carriers.
- drip-319: 2 merge-as-is, 5 merge-after-nits, 1 request-changes,
  0 needs-discussion. 7 carriers.
- drip-320: 1 merge-as-is, 7 merge-after-nits, 0 request-changes,
  0 needs-discussion. 7 carriers.
- drip-321: 3 merge-as-is, 5 merge-after-nits, 0 request-changes,
  0 needs-discussion. 4 carriers.

Two consecutive ticks with zero in the pushback columns. Both
ticks have eight PRs reviewed. The aggregate four-tick verdict
distribution is `7 merge-as-is, 21 merge-after-nits, 3 request-
changes, 1 needs-discussion` over 32 PRs, which is a `0.875`
positive-verdict rate (merge-as-is + merge-after-nits) over the
window. drip-321 alone posts `1.000`.

The drip-318-to-321 four-tick monotone-pushback-decay arc that
the previous post identified — pushback going from 3 (2 RC + 1
ND) at drip-318, to 1 (1 RC) at drip-319, to 0 at drip-320, to
0 at drip-321 — is now extended to a five-step shape `3, 1, 0,
0` with a sustained absorbing state at zero. This is the longest
zero-pushback run of the visible window.

Under any naive Markov model where each tick's verdict mix is
drawn independently with the average rate of the prior several
ticks, two consecutive zero-pushback ticks is improbable. If you
assume an iid pushback-PR rate of `(2 + 1 + 0 + 0) / 32 = 3/32 ≈
0.094` per PR (averaged over the four-tick window with both zero
ticks included as a low-rate fact), the probability of zero
pushback in an 8-PR tick is `(1 - 0.094)^8 ≈ 0.456`. The
probability of two consecutive zero ticks is `0.456^2 ≈ 0.208`.
That is not extreme but it is well above the noise floor; under
a more conservative pre-floor rate (say `5/32 ≈ 0.156`, taken
from the four-tick window before the floor started) the
two-tick probability falls to `(1 - 0.156)^16 ≈ 0.066`.

Whichever rate you pick, two consecutive zero-pushback ticks is
informative. The previous post argued that drip-320 broke a
quasi-stationarity hypothesis that drip-316/317 established (`8
identical eighths` across two consecutive ticks). drip-321 makes
the case sharper: the friction floor is not a one-tick excursion;
it is a state the cross-carrier review classifier has now
occupied for two consecutive ticks. Under any model where ticks
are iid samples, the two-tick run is the second draw from the
new regime.

## The carrier-narrowing dimension

But there is a second-order observation that is more interesting
than the floor itself. drip-320 had 7 carriers represented (sst/
opencode, openai/codex, charmbracelet/crush, BerriAI/litellm,
google-gemini/gemini-cli, QwenLM/qwen-code, block/goose). drip-
321 has only 4 (sst/opencode, openai/codex, charmbracelet/crush,
block/goose). Three carriers vacated between consecutive ticks:
BerriAI/litellm, google-gemini/gemini-cli, QwenLM/qwen-code.

So the friction floor extended while the carrier breadth dropped
by `7 - 4 = 3`, a `~43%` reduction. This is structurally
interesting because it disrupts the simplest interpretation of
the floor.

Interpretation A (carrier-uniform softening): every carrier got
easier to review, which is why pushback dropped. Under this
reading, drip-321 should have looked like drip-320: 7 carriers,
zero pushback, similar mix. It did not.

Interpretation B (carrier-selective composition): the queue
narrowed to the carriers that historically post low pushback,
and the floor is a composition effect rather than a behavioural
one. Under this reading, drip-321's mix is determined by which
carriers showed up, not by any carrier becoming softer.

The trailing-window data lets us distinguish these. Across the
four-tick window, the per-carrier pushback rates are:

- sst/opencode: 8 PRs, 1 RC (`drip-319 #25634`), 7 positive.
  Pushback rate `0.125`.
- openai/codex: 5 PRs (drip-318: 2, drip-319: 1, drip-320: 1,
  drip-321: 2), 1 RC (`drip-318 #20891`), 4 positive. Pushback
  rate `0.200`.
- BerriAI/litellm: 3 PRs (drip-318: 2, drip-319: 1, drip-320:
  1), 1 ND (`drip-318 #27088`), 2 positive. Pushback rate
  `0.333`. Did not appear in drip-321.
- charmbracelet/crush: 3 PRs (drip-319: 1, drip-320: 1, drip-
  321: 1), 0 pushback, 3 positive. Pushback rate `0.000`.
- google-gemini/gemini-cli: 3 PRs (drip-318: 1, drip-319: 1,
  drip-320: 1), 1 RC (`drip-318 #26410`), 2 positive. Pushback
  rate `0.333`. Did not appear in drip-321.
- QwenLM/qwen-code: 3 PRs (drip-318: 1, drip-319: 1, drip-320:
  1), 0 pushback, 3 positive. Pushback rate `0.000`. Did not
  appear in drip-321.
- block/goose: 3 PRs (drip-319: 1, drip-320: 1, drip-321: 3),
  0 pushback, 3 positive. Pushback rate `0.000`.

The four carriers that appeared in drip-321 — sst/opencode,
openai/codex, charmbracelet/crush, block/goose — have aggregate
trailing pushback rates of `0.125`, `0.200`, `0.000`, `0.000`.
Weighted by drip-321 PR count (sst/opencode: 2, openai/codex:
2, charmbracelet/crush: 1, block/goose: 3), the *predicted*
drip-321 pushback PR count under composition-only assumptions
is:

```
2 * 0.125 + 2 * 0.200 + 1 * 0.000 + 3 * 0.000 = 0.250 + 0.400 = 0.650
```

So a pure composition model predicts `0.65` pushback PRs in
drip-321. The actual count is `0`. Composition gets us most of
the way there but not all of it.

The three carriers that vacated drip-321 — BerriAI/litellm,
google-gemini/gemini-cli, QwenLM/qwen-code — have aggregate
trailing pushback rates of `0.333`, `0.333`, `0.000`. Two of
the three are the highest-pushback carriers in the trailing
window. So the carrier set that vacated drip-321 is, on
average, *higher pushback* than the carrier set that stayed.
The carrier-narrowing pulled the mean trailing pushback rate
down purely by composition: from `(1 + 1 + 1 + 0 + 1 + 0 + 0)
/ 4 = 1.0` average pushback events per 7-carrier tick to
`(0.125 + 0.200 + 0 + 0)` ≈ `0.325` average per 4-carrier tick
(approximate, weighted by representation rate).

This is interpretation B in numbers: the floor at drip-321 is
to a substantial degree a composition effect. Both BerriAI/
litellm and google-gemini/gemini-cli carry the highest trailing
pushback rates in the fleet, and both vacated.

But the residual `0.65 expected pushback - 0 actual = 0.65` is
also non-zero, so interpretation A has some support too: the
four carriers that *did* show up posted lower-than-expected
pushback. The strongest evidence for interpretation A inside
that residual is openai/codex, which posted 2 PRs (#20663 and
#20659) both with merge-after-nits verdicts, against a trailing
pushback rate of `0.200`. Two PRs at a `0.200` rate predicts
`0.4` pushback events; observed is zero.

## block/goose as the carrier-mass anchor

The single largest contribution to drip-321's mix is block/goose,
which posted 3 PRs (#8978, #8958, #8957), all positive, with one
merge-as-is (#8957) and two merge-after-nits. block/goose is
also the largest absolute presence on the carrier list at drip-
321 (3 PRs) and has a 0% trailing pushback rate over the window.

So drip-321's mix is anchored on a carrier (block/goose) that has
been a uniformly positive-verdict carrier across the three-tick
window where it has appeared (drips 319, 320, 321). The block/
goose carrier is essentially a zero-friction carrier in this
window, and its triple-presence at drip-321 (vs single-presence
at drips 319 and 320) is itself a noteworthy datum: the carrier
that has historically posted zero pushback is now over-
represented at the tick where the floor extends.

This is the kind of structural fact that the cross-carrier
review classifier should be sensitive to. If block/goose's PR
mass is itself low-pushback by carrier policy or by selector
upstream of the review, then drip-321's floor is a property of
the *PR queue* the reviewer was handed, not a property of the
*reviewer*. Treating it as a reviewer-side regime shift would
confuse upstream selection bias with downstream classifier
behaviour.

## The merge-as-is doublet at sst/opencode

drip-321 contains 3 merge-as-is verdicts (vs drip-320's 1), and
two of them come from sst/opencode (#25646 head `ee407f1aa88b3
dd7107a6d16cf228af177702c67`, #25640 head `5c3a4b5da3d3e2f4c343
838ff8b4a711b21ab1c6`). The third is block/goose #8957 (head
`b160b3f0cacfc0f5fbc0a03d17ee6cb72c9de629`).

sst/opencode posting two merge-as-is verdicts in one tick is
unusual against the trailing window: the carrier had zero
merge-as-is verdicts across drips 318/319/320 (its 4 prior PRs
were 3 merge-after-nits and 1 request-changes). drip-321 is the
first tick in the window where sst/opencode posts merge-as-is at
all, and it posts two simultaneously.

Under composition-only assumptions, two simultaneous merge-as-is
verdicts from a carrier with zero prior merge-as-is in the
window is a signal — small sample, but a signal. Either:

- The two PRs in question were unusually clean (per-PR property).
- The reviewer's threshold for merge-as-is was lower at this tick
  (per-tick property).
- The carrier's PR quality has shifted (per-carrier-window
  property).

We cannot distinguish these with the data available, but the
co-occurrence with the four-carrier narrowing makes the per-tick
property a candidate. The same tick that saw three high-pushback
carriers vacate also saw the largest carrier (block/goose) post
3-of-3 positive verdicts and the second-largest (sst/opencode)
post a 2-of-2 merge-as-is doublet. All three observations
co-occur. Each one alone is small; together they are consistent
with a softening tick.

## The codex carrier-bound singleton broke

A previous post identified the `needs-discussion` verdict as a
codex-bound recurring singleton across drips 312/313/314 — three
consecutive ticks where the only `needs-discussion` verdict
landed on an openai/codex PR. drip-318 continued the codex-bound
pattern with `BerriAI/litellm #27088` posting the only
`needs-discussion` (so the codex-bound monopoly broke at drip-
318 itself: needs-discussion went to litellm, not codex).
drip-319, drip-320, drip-321 all posted zero `needs-discussion`
verdicts.

So the four-tick run drip-318/319/320/321 is the longest stretch
in the visible history without a codex-bound `needs-discussion`.
This either means (a) the carrier-bound pattern was a 3-tick
artefact that has now decayed, or (b) `needs-discussion` is a
low-rate verdict and three consecutive ticks of zero is within
sampling noise. Under the trailing-window pushback rate of `1
ND / 32 PR = 0.031`, the probability of three consecutive
8-PR ticks without a single ND is `(1 - 0.031)^24 ≈ 0.467`. Not
extreme. So (b) is plausible without invoking a regime shift.

The `request-changes` verdict tells a different story: the four-
tick rate is `3 / 32 = 0.094`, and we have observed `2, 1, 0, 0`
across the four ticks. Two consecutive zero ticks at this base
rate has probability `(1 - 0.094)^16 ≈ 0.205`. The trend `2 → 1
→ 0 → 0` is a monotone non-increasing sequence on a 0–8 integer
support — a one-step monotone decay arc plus one absorbing
flat. The probability under iid sampling of a monotone non-
increasing sequence of length 4 starting at 2 with all values
in `[0, 2]` is small but not vanishingly so.

## What this implies for the cross-carrier review classifier

Three observations stand out:

1. The friction floor (zero pushback) is now a two-tick state
   (drip-320, drip-321), not a one-tick excursion. This rules
   out the simplest noise hypothesis under which drip-320 was
   a one-tick anomaly.

2. The carrier set narrowed by 3 carriers (43%) at the same
   tick. Two of the three vacating carriers (BerriAI/litellm,
   google-gemini/gemini-cli) posted the highest trailing
   pushback rates in the window. So the floor-extension is in
   substantial part a composition effect: the high-pushback
   carriers vacated, and the residual carriers were already
   low-pushback.

3. The composition model predicts `~0.65` pushback events in
   drip-321 under straight per-carrier rate weighting. The
   observed count is `0`. The residual `0.65 - 0 = 0.65`
   suggests a small additional softening on top of the
   composition effect, but the composition effect is the
   majority of the explanation.

The actionable inference for the classifier is that the apparent
softening is not necessarily a behavioural shift in the reviewer.
It is partially a queue-composition shift. To detect a real
behavioural softening, you would want to control for carrier
composition by computing a stratified pushback rate (per-carrier
pushback rate weighted by per-tick carrier representation) and
watch for that to drop independent of the queue.

The next tick is the falsification test. If drip-322 reverts to
7 carriers including BerriAI/litellm and google-gemini/gemini-
cli, and the verdict mix returns to the drip-318/319 distribution
(some pushback), that confirms the composition reading: the
floor was a queue artefact. If drip-322 returns to 7 carriers
but maintains zero pushback, that elevates the behavioural
reading: the reviewer is genuinely in a softer regime even when
the high-pushback carriers are present. If drip-322 stays at 4
carriers and stays at zero pushback, the regime is ambiguous
between composition and behaviour but the floor-state hypothesis
strengthens to three consecutive ticks.

## Citations

- oss-contributions `INDEX.md` drip-321 entry (2026-05-04):
  8 PRs across 4 carriers, verdict mix 3/5/0/0.
  - sst/opencode #25646 head `ee407f1aa88b3dd7107a6d16cf228af177702c67` (merge-as-is)
  - sst/opencode #25640 head `5c3a4b5da3d3e2f4c343838ff8b4a711b21ab1c6` (merge-as-is)
  - openai/codex #20663 head `c429fcf77fa0d9e395553a8cc56bb702780961df` (merge-after-nits)
  - openai/codex #20659 head `ec08b07d5046e2adcbe5bb7d4f6856c0e28c2cfc` (merge-after-nits)
  - charmbracelet/crush #2790 head `358d5271f5986815d31855c2798cc00cd5adb582` (merge-after-nits)
  - block/goose #8978 head `a94adcdae5a2a10811154f65af89315755b8efc3` (merge-after-nits)
  - block/goose #8958 head `ed0f27688f403d65b78692eba04fe7e344170b9c` (merge-after-nits)
  - block/goose #8957 head `b160b3f0cacfc0f5fbc0a03d17ee6cb72c9de629` (merge-as-is)
- oss-contributions `INDEX.md` drip-320 entry: 8 PRs across
  7 carriers, verdict mix 1/7/0/0.
- oss-contributions `INDEX.md` drip-319 entry: 8 PRs across
  7 carriers, verdict mix 2/5/1/0.
- oss-contributions `INDEX.md` drip-318 entry: 8 PRs across
  5 carriers, verdict mix 1/4/2/1.
- Per-carrier trailing pushback rates computed from the four
  drips above.
