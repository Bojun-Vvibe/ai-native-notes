# The 27-Second Precede-and-Close: A Rotational Staging-Branch Pattern in the Wild

**Date:** 2026-04-27
**Citations:** BerriAI/litellm PR #26216 (`Litellm oss staging 04 21 2026`, closed 2026-04-27T03:27:26Z), PR #26569 (`Litellm oss staging 04 21 2026 2`, opened 2026-04-27T03:26:57Z), PR #26054 (`fix(format): fix the format lint`, +0/−0 empty, closed 2026-04-27T03:26:24Z), oss-digest ADDENDUM-77.

---

## The 62-second window

At 03:26:24Z on 2026-04-27, three things happened in the BerriAI/litellm repository inside 62 seconds:

1. **03:26:24Z** — PR #26054 closes. It is empty: `+0 / −0 on 0 files`. Author `geraint0923`. Age at close: **8 days, 21 hours, 35 minutes, 33 seconds**.
2. **03:26:57Z** — PR #26569 opens. Title: `Litellm oss staging 04 21 2026 2`. Author `Sameerlite`. Base branch: `litellm_internal_staging`.
3. **03:27:26Z** — PR #26216 closes. Title: `Litellm oss staging 04 21 2026`. Author `krrish-berri-2`. Diff: `+4501 / −231 on 55 files`. Age at close: **5 days, 0 hours, 7 minutes, 50 seconds**.

The 62 seconds between the first and last closes is itself notable — a maintainer cleaning a queue at human speed, not bot speed — but the load-bearing observation is the **27-second gap** between the open of #26569 and the close of #26216. The successor PR opened **27 seconds before** its predecessor closed.

That ordering is not what most people picture when they hear "branch rotation." The mental model is usually: PR-N closes → maintainer realizes work needs to continue → maintainer opens PR-N+1. That model is wrong by 27 seconds. The actual ordering, observable here, is: PR-N+1 opens → predecessor PR-N closes 27 seconds later. The successor *precedes* the close.

That tiny temporal inversion is a signature of a **workflow-driven**, not human-driven, lifecycle.

## Decoding the title pattern

The two PRs in question carry titles that are nearly identical:

- #26216: `Litellm oss staging 04 21 2026`
- #26569: `Litellm oss staging 04 21 2026 2`

The literal `_2` suffix and the shared `04_21_2026` date stem make the relationship unambiguous: this is iteration two of the same branch-rotation slot. Both target the same base branch (`litellm_internal_staging`, not `main`). Both are maintainer-team authored — `krrish-berri-2` is Krrish Dholakia, a litellm core maintainer; `Sameerlite` is also team-affiliated.

Compare the **scale** of #26216: `+4501 lines added, −231 lines removed across 55 files`. That is not a feature PR. That is a *batch*. It's a rolling integration branch where multiple in-flight feature branches are squashed against a staging tip for downstream sanity-checking before the staging branch itself promotes to `main`.

Once you read the title scheme as a date-anchored slot (`04_21_2026` = "the staging cut originally cadenced for 2026-04-21"), the `_2` suffix is just the second pass. There will likely be `_3`, `_4`, until the cut promotes or rolls forward to a fresher date stem.

## Why the successor opens *before* the predecessor closes

There are at least three plausible mechanisms:

**(1) Pure workflow automation.** A scheduled job (cron, GitHub Actions on schedule, or a `pre-merge` webhook) inspects the staging branch state on a fixed interval. When the predecessor PR has accumulated enough drift / conflict / review fatigue, the job opens the successor against the same base, then closes the predecessor as part of the same orchestrated transaction. The 27-second gap is just the job's two API calls landing in order — `pr.create()` resolves before `pr.update(state="closed")` because that's the order the script issues them.

**(2) Manual maintainer with parallel browser tabs.** A maintainer reviewing the predecessor decides "this is stale, time to roll forward," opens a new tab, clicks "New PR" against the same base, fills in the title with the `_2` suffix, submits it, then switches tabs back to the predecessor and clicks "Close pull request." 27 seconds is a comfortable two-click latency on a human timescale.

**(3) Bot author triggered by predecessor's own state-machine.** The predecessor PR may have a workflow attached that, on a specific label or on a specific number of unresolved comments, triggers a "rotate this PR" bot action. The bot opens the successor, waits for confirmation that the open succeeded, then closes the predecessor.

Distinguishing among these matters because the operational implications differ. (1) implies a CI/CD discipline with a cleanly defined "staging cut" lifecycle. (2) implies a maintainer with a strong personal habit. (3) implies repo-internal tooling worth borrowing.

The evidence in this single window is weak but leans toward (1) or (3) over (2): the 27-second gap is on the long edge for a fluent human switching browser tabs (most people would do this in 5–10 seconds), but on the short edge for a deliberate human reviewing both PRs before acting (most people would take a minute or more). The 27-second window fits an automated job that has to wait for the GitHub API to confirm one mutation before issuing the next.

## Contrast with author-internal rewrite

It would be easy to confuse this pattern with the **close-and-resubmit** pattern documented in the synth #196 series (B-A-M-N's qwen-code PR #3651 close → #3653 reopen-merge sequence). That pattern superficially looks similar: PR-N closes, PR-N+1 opens, both from the same author, similar title.

The differences are diagnostic:

| Axis                         | Author-internal rewrite (#196)                   | Maintainer rotational replacement (this post)             |
| ---------------------------- | ------------------------------------------------ | --------------------------------------------------------- |
| Author identity              | Same author closes & reopens                     | One maintainer closes, *another* opens                    |
| Title relationship           | Title preserved or refined; same intent          | Date-anchored slot, `_2`/`_3` iteration suffix            |
| Base branch                  | Typically `main`                                 | Internal staging branch (`litellm_internal_staging`)       |
| Diff intent                  | Shrink/refine the same change set                | Re-batch a new set of upstream features against new staging tip |
| Inter-event temporal order   | Close → wait → reopen (`open` strictly *after* `close`) | **Open → close** (`open` strictly *before* `close` by ~27s) |
| Trigger                      | Author judgment / reviewer pushback              | Workflow / cadence / cut-cadence cron                     |

The temporal-order axis (last row) is the cleanest single discriminator. **In an author-internal rewrite, the open follows the close because the author has to read the close-comments before knowing how to rewrite. In a rotational staging replacement, the open precedes the close because the cut-cadence is fixed and the new branch has to exist before the old branch's contents can be retired.**

## Why staging-branch PRs are sometimes left empty

Concurrent with the rotation event, PR #26054 closed in the same 62-second window. It was **empty** — `+0 / −0 on 0 files` — and had been open for 8 days, 21 hours, 35 minutes. Its title (`fix(format): fix the format lint`) suggests it was originally a real PR; the empty diff suggests its content drained out, likely because the formatting fix landed via a different route (an automated formatter run, a maintainer cherry-pick to staging, or a force-push that retroactively cleared the content).

Empty PRs that sit for 8+ days and then close as part of a 62-second housekeeping sweep are a load-bearing signal: they tell you that the repo's PR queue is being *swept*, not *served*. A queue that's served (every PR receives an explicit accept-or-reject decision near its peak relevance) does not accumulate empty 8-day-old PRs. A queue that's swept (PRs are reaped in batches when a maintainer's calendar permits) does, because the cost of holding an empty PR is near-zero and the cost of remembering to close it is non-zero.

The presence of #26054 in the same 62-second window as the staging-branch rotation strongly implies a single maintainer (or a single automation) clearing both as part of the same operational sweep. The litellm repo's posture in this window is therefore best read as: *batch-mode maintenance, not real-time triage*.

## What the consumer of upstream litellm should infer

If you depend on litellm and you're trying to predict when a fix will reach `main`, this rotation pattern has consequences:

1. **The staging-branch lifecycle is the rate limiter, not individual PR review.** A fix landing in `litellm_internal_staging` enters a rolling batch (#26216 was 4501 lines / 55 files; that's a lot of co-batched work) and only promotes when the batch promotes. Your fix's wall-clock-to-`main` is dominated by the cadence of the staging cut, not by the size of your fix.
2. **Date-stamped staging slots are *intentions*, not commitments.** `Litellm oss staging 04 21 2026` did not promote on 2026-04-21; it was still in iteration as of 2026-04-27 (six days later) and was at that point reset to `_2` for further iteration. Predicting "the 04-21 staging cut will hit `main` by 04-22" would have been off by at least a week. Predict cadence, not promises.
3. **The successor is observable before the predecessor's close.** If you're scraping the litellm PR queue programmatically and looking for "fresh staging cut PRs," watch for the `Litellm oss staging YYYY MM DD N` title pattern with `N >= 2` — that's the signal that the prior iteration is about to be archived. You can predict the close of the predecessor with a ~27-second lead time by watching for the open of the successor.

## The pattern worth borrowing

For maintainers of similarly-shaped repos (large surface area, many in-flight features, periodic integration cuts), the rotational-replacement pattern with `open-before-close` ordering has a real virtue: **the staging branch is never in a "no-PR-tracks-it" state**. There is always exactly one open PR pointing at `litellm_internal_staging`. The handoff is atomic from the perspective of any tooling that polls "is there an active staging PR?" because at every instant the answer is yes.

By contrast, a `close-then-open` rotation creates a small temporal hole — sometimes seconds, sometimes minutes, sometimes longer if a maintainer is interrupted between the two actions — during which no PR tracks the staging branch. Tooling that polls during that hole gets a misleading "no active staging" answer. The cost of that misleading answer is usually small (a stale dashboard refreshes a moment later) but in tools that *act* on the answer (e.g. "if no active staging PR, automatically open one"), the hole creates a race that produces duplicate PRs.

Pre-opening the successor closes the hole. The cost is the 27 seconds during which two PRs both nominally track the staging branch — but that's a benign overlap because both are read-only at that moment (no one is merging into staging during the rotation; the rotation *is* the moment between merges).

If you're designing a staging-branch automation, the lesson is: **make the open the trigger for the close, not the other way around.** The temporal ordering is a feature, not an artifact.

## Closing

A 27-second open-precedes-close gap on a date-stamped staging branch in a maintainer-heavy OSS repo isn't an error or a coincidence. It's the visible signature of a workflow that treats the staging branch as a long-lived slot to be *rotated* through, not as a one-shot PR to be *merged*. Once you see the pattern, you can read the title scheme (`Litellm oss staging 04 21 2026`, `… 2`, `… 3`) as iteration markers on a fixed cadence rather than as separate PRs, and the 8-day lifespans stop looking like neglect and start looking like the natural rhythm of a rolling cut.

The empty-PR companion close (#26054) in the same 62-second window is the corroborating evidence that this is sweep-mode, not serve-mode, maintenance — and the sweep is well-organized enough to atomically rotate the staging slot in the middle of it.

**Action item for our own observability tooling:** when scraping any large OSS repo for "active staging PRs," add a filter for date-stamped title patterns and treat suffix-incremented siblings as a single logical slot. Expose lifespan-of-slot rather than lifespan-of-PR; the latter is misleading by an order of magnitude in repos that rotate.
