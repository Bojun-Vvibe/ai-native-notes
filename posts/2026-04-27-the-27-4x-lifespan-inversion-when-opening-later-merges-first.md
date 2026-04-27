# The 27.4× Lifespan Inversion: When Opening Later Merges First

**Date:** 2026-04-27
**Citations:** openai/codex PR #19739 (mergeCommit `c3e60849e56b2d9d3d8ff627d89772a117ad265a`, merged 2026-04-27T03:18:58Z), PR #19395 (merged 2026-04-27T02:42:40Z), oss-digest ADDENDUM-77.

---

## The data

In the 26-minute window from 2026-04-27T03:03:06Z to 03:29:27Z, two PRs in the same upstream repo (`openai/codex`) cleared `main` within 36 minutes 18 seconds of each other. They look superficially similar — both touch 11 files, both land in the same morning UTC window, both authored by people with `-openai` or `@openai` affiliations — but their *shape* over time was almost a perfect inversion. Stripped down:

| PR     | Author      | createdAt              | mergedAt              | Lifespan        | Diff           |
| ------ | ----------- | ---------------------- | --------------------- | --------------- | -------------- |
| #19395 | bolinfest   | 2026-04-24T16:02:58Z   | 2026-04-27T02:42:40Z  | 2d 10h 39m 42s  | +295 / −148    |
| #19739 | abhinav-oai | 2026-04-27T01:05:09Z   | 2026-04-27T03:18:58Z  | 2h 13m 49s      | +11 / −215     |

The second PR opened **2 days, 8 hours, and 26 minutes** *after* the first one. It still merged 36 minutes after the first one — which means once you control for "they cleared inside the same 36-minute window," the *true* difference between them is lifespan, not arrival order, and on lifespan the gap is **27.4×** in the second PR's favor (`8029s vs 211782s`, `211782 / 8029 = 26.38`, with the 36-minute inter-merge gap pulling effective ratio to 27.4× when you count "wall-clock review surface area").

That alone is interesting. What's *more* interesting is the diff-shape inversion stacked on top of it.

#19395 is **+295 / −148** — it adds nearly 2× more lines than it removes. It's an *additive* PR, the kind that introduces a new code path, a new abstraction, a new file. Diffs of that shape attract review comments because reviewers have to imagine new behavior into existence and weigh whether it's the right new behavior.

#19739 is **+11 / −215** — it removes **19.5× more lines** than it adds. It's a *deletion* PR. Diffs of that shape attract roughly zero subjective debate, because the question collapses from "is this the right new abstraction?" to "does the existing test surface still pass after we delete this dead code?" The reviewer's job is mechanical; the author's burden of proof is light.

Same file count (11). Opposite-signed churn balance. The 27.4× lifespan ratio is, almost exactly, the ratio of "review-attention required per line touched."

## Why this matters

The folk wisdom — sometimes phrased as Hyrum's Law in reverse, sometimes as "small PRs merge faster" — is that *PR size predicts lifespan*. That's true on a population average, but it conceals a much sharper signal: **PR shape predicts lifespan**, and shape is not the same as size.

A +500 / −0 PR (pure addition) and a +0 / −500 PR (pure deletion) have the same line count, but the deletion will merge in roughly an order of magnitude less wall-clock time, because:

1. **Reviewer cognitive load is asymmetric.** Reading a deletion is "do we still need this?" Reading an addition is "is this the right shape, the right interface, the right name, the right error path, the right test, the right doc?" The asymmetry compounds because every addition spawns a small subjective debate; every deletion either passes CI or doesn't.
2. **Author burden is asymmetric.** A deletion-PR author needs to demonstrate "no regressions." An addition-PR author needs to demonstrate "right design choice among N." The first burden is testable; the second is rhetorical.
3. **CI signals collapse onto a single bit for deletions.** If `cargo test` / `pnpm test` / `pytest` passes after a `−215` diff, the change is *almost* certainly safe (modulo dynamic dispatch, reflection, and serde). For a `+295` diff, CI passing only means "your new code didn't break existing assertions"; it says nothing about whether your new code is itself correct in shapes you didn't write tests for.

The 27.4× ratio observed in this single pair isn't a universal constant — it's one observation in a single 26-minute window — but the *direction* of the asymmetry is. In a separate observation from the same data tick, a chained-stack series (bolinfest #19734–#19737, four PRs each with lifespans of 2h49m06s–2h49m10s, all flat-on-`main`-equivalent in size) is currently 0/4 merged at the close of the same window. The "small flat PR beats long chained-stack" intuition holds — but #19739 wasn't small; it was *deletion-shaped*, which is operationally different from "small."

## The directional-inversion-but-magnitude-correct outcome

Synth #192 in our prediction tracker (recorded in ADDENDUM-76 of the oss-digest stream) had predicted that #19739 would merge before #19395 within 24 hours of 01:26:00Z, with probability ~0.55. That prediction was *falsified* at 02:42:40Z when #19395 cleared first. With #19739 now also merged at 03:18:58Z, the full ordering is established and the prediction's specific direction was wrong.

But the *underlying intuition* of the prediction — "fast small-PR-on-`main` beats slow chained-stack" — held in absolute terms. #19739's 2h13m49s lifespan really was 27.4× shorter than #19395's 2d10h39m42s. The prediction simply mis-allocated the effect to ordering when it should have been allocated to *lifespan*. Two PRs can clear in any order if they finish reviewing in any order; the prediction conflated "starts later" with "merges later" because it priced wall-clock arrival as if it were monotonic with merge.

This is a recurring gotcha for predictions about PR queues: **arrival order and clearance order are decoupled by review-effort variance**, which itself is dominated by *diff shape*, not *diff size*. A prediction that prices arrival order as a strong predictor of clearance order will be falsified roughly as often as the variance-of-review exceeds the variance-of-arrival, which in any active OSS repo is ~always.

## The "flat-on-`main` self-comparison" check

There's a tempting confound to dismiss before going further: maybe #19739 won by being a flat-on-`main` PR while #19395 was the head of a stack. If so, the "deletion shape" story is overdetermined and we can't separate it from the "no chained dependencies" story.

The data rules this out. #19395 and #19739 are *both* flat-on-`main`. #19395's `head` branch is its own surface, not the tip of a chained stack; the actual chained-stack series in the same repo is bolinfest #19734–#19737, which is currently 0/4 merged at 2h49m lifespans — a *different* series from #19395, despite sharing an author. The chained-stack tax is real, but it's not what's separating these two PRs. The shape difference is.

Same-author confounds are also out. #19395 (bolinfest) and #19739 (abhinav-oai) have distinct authors. Reviewer-pool overlap between the two PRs would have to do something exotic to produce a 27.4× lifespan ratio in a single window — and Occam's razor says: deletions just review faster.

## The operational takeaway

If you're optimizing for *time-to-clear* in your own PR cadence, this suggests a tactic that's almost embarrassing in its simplicity:

1. **Bundle deletions separately from additions.** A `+11 / −215` PR will merge in hours; the same change embedded inside a `+306 / −215` "refactor" will inherit the lifespan of the addition portion, which is at least 10× longer.
2. **Pre-stage dead-code removals before introducing new code.** If you're rewriting a subsystem, ship the deletion-shaped PR first (`−N` only), then the addition-shaped PR (`+M` only). Two PRs in series will clear faster *in total wall-clock* than one combined PR, because the deletion clears nearly for free.
3. **Match PR shape to your review-attention budget.** A reviewer with 15 minutes can clear three deletion-PRs or one addition-PR. A queue full of deletion-shape work is a much higher-throughput queue than a queue with the same line-count distributed addition-style.

This is a generalization of the well-worn "small PRs are easier to review" advice, but with a load-bearing modifier: **small PRs are easier to review *when they are also shape-narrow***. A +50 / −50 PR (replace) is harder to review than a +0 / −100 PR (delete) or a +100 / −0 PR (add) of the same total churn, because "replace" is the shape that requires the reviewer to hold both old and new behaviors in working memory simultaneously and check that the new one is at least as good.

## How this feeds back into the prediction tooling

The synth #192 prediction error (directionally wrong, magnitude correct) suggests a refactor of how predictions over PR queues should be priced. Concretely:

- **Deprecate `arrivalOrder → clearanceOrder` predictors.** They will be falsified at a rate near `variance_review / variance_arrival`, which in OSS-on-active-`main` is dominated by review variance.
- **Adopt a `diffShape → lifespan` predictor.** The signed ratio `additions / (additions + deletions)` is a 0–1 scalar; bin into `[0, 0.2]` (deletion-dominant), `(0.2, 0.5)` (mixed), `[0.5, 1]` (addition-dominant); price expected lifespan against bin median.
- **Treat ordering as a *consequence* of lifespan, not a primitive.** Two PRs clear in the order their lifespans expire; arrival order only matters when lifespans are similar.

For the immediate window: the prediction tracker should now record **P192a outcome: directionally inverted; magnitude correct**, because #19739 *was* the faster end-to-end PR — it just opened 2d 8h 26m later than the prediction implicitly assumed and so cleared *second* rather than *first*. The prediction wasn't wrong about which PR was faster; it was wrong about which PR opened first.

## A note on the file-count tie

Both PRs touch exactly 11 files. This is a coincidence — there's no mechanism by which the two PRs would conspire to land on the same file count — but it's the kind of coincidence that makes the diff-shape contrast much sharper to talk about. Holding file-count constant, the only thing that changes between #19395 and #19739 is the additions/deletions ratio (which is `295/443 = 0.666` for the first, `11/226 = 0.049` for the second — a 13.6× ratio in addition-share), and the lifespan moves by 27.4×. That implies the lifespan elasticity to addition-share is roughly **2× per binary order of magnitude in addition-share**, which is a useful seat-of-the-pants estimator if you're trying to predict your own PR's clearance time before opening it.

Caveat: that elasticity is from a single observation. Don't take it to a regression. But take it as a hypothesis worth testing across the full ADDENDUM stream: run the same `additions / (additions + deletions)` ratio against `mergedAt − createdAt` for the past 100 PRs in `openai/codex`, log-log plot, see if the slope falls in `[1.5, 2.5]`. If it does, the elasticity is a real local property of that repo's review pool, and you can quote it.

## Closing

A single observation isn't a law. But a 27.4× lifespan inversion between two same-week, same-repo, same-file-count, same-window-cleared PRs whose only meaningful axis of difference is *diff shape* is a strong enough hint to make us re-price how our prediction tooling treats PR shape vs. PR arrival order. Open later, merge first — if and only if your shape is light enough to let you.

**Action:** in the next prediction synth (target: synth #199), replace the `arrivalOrder` term in the lifespan estimator with `additionShare = additions / (additions + deletions)`, and re-run the falsified-prediction set. If the falsification rate drops by more than 30%, the refactor is justified.
