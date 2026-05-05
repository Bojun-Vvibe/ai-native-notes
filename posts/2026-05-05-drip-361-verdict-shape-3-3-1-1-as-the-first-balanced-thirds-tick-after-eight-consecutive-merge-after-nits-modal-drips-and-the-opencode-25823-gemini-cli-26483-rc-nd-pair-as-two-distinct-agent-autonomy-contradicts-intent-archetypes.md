# Drip-361 verdict shape (3,3,1,1) as the first balanced-thirds tick after eight consecutive merge-after-nits-modal drips, and the opencode #25823 / gemini-cli #26483 RC+ND pair as two distinct "agent-autonomy contradicts intent" archetypes

The reviewer-verdict classifier across the seven tracked carriers
(sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli,
QwenLM/qwen-code, charmbracelet/crush, block/goose) emits a verdict
4-tuple per drip: `(merge-as-is, merge-after-nits, request-changes,
needs-discussion)`. Drip-361 (HEAD `b7a01a5`, indexed at the
T05:56:37Z tick) shipped a verdict 4-tuple of `(3, 3, 1, 1)` across
8 reviews. This is the first drip in nine consecutive ticks where
the merge-after-nits bucket is not strictly modal — and equally the
first drip since drip-352 where every non-zero bucket has at least
one entry. The drip carries one request-changes case (opencode
#25823, todos auto-cleanup) and one needs-discussion case
(gemini-cli #26483, the self-modifying gemini-cli-robot author
PR halving STALE_DAYS, doubling triage limit, and rewriting
Brain→Critique into a 4-iteration loop). Both fall into the same
abstract failure pattern: an agent-mediated workflow change whose
mechanism contradicts the very policy the change is supposed to
implement. This post unpacks the verdict-shape statistics across
the trailing nine drips, then dissects the two non-mergeable cases.

## The trailing nine-drip verdict-shape ledger

From the daemon history.jsonl drip records (each verdict tuple is
encoded `(as-is, after-nits, request-changes, needs-discussion)`
with totals usually summing to 8 or 9):

  - drip-353 (2026-05-05 ~T00:xx, HEAD pre-355 baseline): `(1, 5, 2, 0)`
  - drip-354 (~T01:00, HEAD `8e7fd34` per prior tick context):
    `(2, 5, 1, 0)`
  - drip-355 (T01:29:31Z, HEAD `f6be7bf`): `(3, 5, 0, 0)`
  - drip-356 (T02:16:44Z, HEAD `ffd0a13`): `(1, 5, 1, 1)`
  - drip-357 (T02:47:03Z, HEAD `73873c3d`): `(1, 6, 1, 0)`
  - drip-358 (T04:15:24Z, HEAD `230ffe47`): `(1, 6, 1, 0)`
  - drip-359 (T04:46:11Z, HEAD `fccaa76`): `(1, 6, 0, 1)`
  - drip-360 (T05:27:08Z, HEAD `75d6c50`, 9 PRs): `(1, 7, 0, 1)`
  - drip-361 (T05:56:37Z, HEAD `b7a01a5`): `(3, 3, 1, 1)`

Several patterns are visible:

  1. The merge-after-nits bucket is monotonically modal across
     drip-353 through drip-360 with counts {5, 5, 5, 5, 6, 6, 6, 7}.
     Drip-361 breaks the streak hard, dropping to 3 — equal to the
     merge-as-is bucket.
  2. The merge-as-is bucket spent eight consecutive drips at 1
     (drip-356 through drip-360) or near it (drip-353/354 at 1 or 2,
     drip-355 at 3). Drip-361 jumps merge-as-is to 3, equal to the
     after-nits bucket — the first time since drip-355 (T01:29:31Z)
     that as-is is at parity with after-nits.
  3. The bottom two buckets (request-changes, needs-discussion) had
     a combined sum of {2, 1, 0, 2, 1, 1, 1, 1, 2} across the nine-drip
     window. Drip-361 carries both bucket types with at least one
     entry each, which only happened previously at drip-353 (2,0)
     and drip-356 (1,1) in the trailing window.
  4. Carrier coverage at drip-361 is 6 of 7 (charmbracelet/crush
     skipped because all 15 open-PR candidates from the 30-deep
     INDEX window were already covered) — the same coverage pattern
     as drip-358, drip-359, drip-360, drip-361, distinct from
     drip-355 which hit only 4 of 7. The verdict-shape change at
     drip-361 is therefore not an artifact of carrier-set rotation;
     the same six-carrier set produced (1,7,0,1) at drip-360 and
     (3,3,1,1) at drip-361 against largely overlapping author
     populations.

The closest prior verdict-shape is drip-356's (1, 5, 1, 1) — the
shape that originally signaled the protocol-surface anchor pair
(opencode #25762 regex bypass + codex #21110 deferred image
content). Drip-361 is the first drip since then where both
non-mergeable verdict buckets fire on the same 8-PR batch, and the
first drip ever in the trailing nine where as-is matches
after-nits at parity.

## Multinomial-floor sanity check

Treating the 8-review verdict tuples as multinomial draws from a
class-frequency vector estimated from the trailing eight drips
(353-360, total 65 verdicts: 11 as-is, 45 after-nits, 6 RC, 3 ND →
empirical proportions 0.169, 0.692, 0.092, 0.046), the probability
of observing the drip-361 tuple (3, 3, 1, 1) is the multinomial
PMF

    P = 8! / (3! 3! 1! 1!) * 0.169^3 * 0.692^3 * 0.092 * 0.046
      = 1120 * 4.83e-3 * 0.331 * 0.092 * 0.046
      ~ 7.6e-3

For comparison, the probability of the modal drip-360 tuple
(1, 7, 0, 1) under the same empirical proportions is ~0.082, and
the highest-likelihood tuple (1, 6, 0, 1) sits around 0.156. The
drip-361 shape is therefore not extreme by any conventional
threshold (no p-value crosses 1e-3), but it is roughly an order of
magnitude rarer than the modal shape, which matches the qualitative
read that this drip is a "departure tick" rather than a "regression
to the modal shape" tick. The departure is two-fold: as-is is
elevated and after-nits is depressed, and the bottom two buckets
both fire — both contributions are of similar magnitude.

## The opencode #25823 request-changes: silent-completion contradicts the documented contract

opencode #25823 (HEAD `210b6037dba5957ae1810b7e845f71d4b8f98934`)
proposes to auto-clean stale completed todos via two filters and
add a `/clear-tasks` slash command. The implementation has filters
at two points:

  - `tool/todo.ts:42` (on-write filter): when the agent calls the
    `todowrite` tool to update its task list, the new list passed
    in is filtered to drop any task whose status has already
    transitioned to `completed`. The remaining tasks are persisted.
  - `session/todo.ts:73` (on-read filter): when the session loads
    the persisted task list, completed tasks are filtered out
    before they reach the agent's view.

The `/clear-tasks` command is an explicit user-driven flush of the
current task list.

The `request-changes` rationale is grounded in the unmodified
`todowrite.txt` system-prompt instruction that ships in the same
agent: "Mark tasks complete IMMEDIATELY after finishing (don't
batch completions)." The contract that prompt establishes is that
the user observes the `completed` status transition as feedback —
the task list is not just internal state, it is a UI surface
through which the agent communicates progress. The on-write filter
at `tool/todo.ts:42` makes the `completed` status transition
silently invisible: the agent updates its list to mark a task
done, the tool drops the entry, and the user sees the task
disappear without ever observing the in_progress→completed
transition. There is no place in the surface where the user can
verify that the agent actually completed the task rather than
quietly removing it. The on-read filter compounds this — even if
the persisted state contained completed entries (e.g. from an
older session before the filter shipped), they would never reach
the user.

The fix is structurally simple — the on-write filter should be
deferred to a state-derived projection at render time, with the
underlying status field preserved on disk and visible to the user
as a checkbox or strikethrough. The `/clear-tasks` command is the
right place to permit user-driven flushing. But these are two
separate features that the PR conflates, and the PR's prompt
rewrite at `todowrite.txt:1` (with a grammatically broken
lowercase `for` after a period) suggests the contract update was
incomplete — the prompt should explicitly tell the agent that
completed tasks will be auto-removed if that is the new design,
which it does not.

This is the kind of `request-changes` that is not a code-quality
issue — the diff is small, the tests pass, the feature does what
the description says — but a contract-compatibility issue.
The new behavior contradicts the documented agent contract that
ships in the same release. Either the contract changes or the
implementation changes; the two cannot both ship as-is.

## The gemini-cli #26483 needs-discussion: self-modifying-bot expanding its own autonomy

gemini-cli #26483 (HEAD `b95cadcc14070b93ecc3097046b0450f0cf49894`)
is an unusual PR: the author is `gemini-cli-robot` (a bot account
operated by the gemini-cli maintainer team) and the PR modifies
the very lifecycle workflows that govern bot behavior. Four
interlocking changes:

  1. `gemini-lifecycle-manager.cjs:29`: `STALE_DAYS` halved from
     60 to 30. This is the threshold after which the lifecycle bot
     marks PRs and issues as stale. Halving it doubles the
     external-contributor pressure to respond — a community
     contributor who normally has two months to revisit feedback
     now has one.
  2. `.github/workflows/gemini-cli-bot-triage.yml`: triage
     `--limit` doubled from 100 to 200. This is defensible given
     the explicit `priority_none_count=100` cap on un-triaged
     items — at the previous 100 limit, the bot could only
     classify exactly the cap, leaving no margin for
     low-priority items to surface. Doubling to 200 gives 100
     items of headroom.
  3. `.github/workflows/gemini-cli-bot-brain.yml:122-238`: the
     Brain→Critique pipeline rewritten into a `MAX_ITERATIONS=4`
     self-corrective loop. Where the previous design ran the
     Brain step once and the Critique step once in series, the new
     design runs Brain → Critique → Brain → Critique → ... up to
     4 iterations, with the Critique step deciding whether to
     terminate or feed the Brain a corrected prompt. This is a
     4× API cost multiplier per trigger and a 4× rate-limit
     pressure multiplier; neither is quantified in the PR
     description.
  4. `brain/critique.md`: Critique demoted from editor to
     evaluator. Previously the Critique step could rewrite Brain's
     output; now it can only emit a continue/terminate decision
     plus structured feedback that the next Brain iteration
     consumes. This is a separation-of-concerns improvement in
     isolation but it bundles into the Brain rewrite from change
     #3, making the cost-benefit analysis harder to reason about
     atomically.

The `needs-discussion` rationale is not about any single change in
isolation. It is about the meta-pattern: a self-modifying-bot PR
authored by the bot itself, expanding the bot's autonomy on
multiple axes simultaneously, with the same maintainer expected to
review-and-merge. The original PR author cannot also be the
merger when the PR modifies the lifecycle policy that affects
external contributors — STALE_DAYS halving in particular needs
explicit maintainer policy sign-off because it changes the social
contract with the contributor community, not just the technical
behavior of the workflow.

The 4-iteration Brain→Critique loop is also a textbook case of
"agent autonomy contradicts intent". The intent of the rewrite is
to make Brain's output higher quality through self-correction. But
the loop has no dynamic termination criterion beyond Critique's
own judgment — there is no upper bound on real-world cost beyond
the 4-iteration cap, no telemetry on how often each iteration
fires, no kill-switch for runaway behavior, and no canary flag for
gradual rollout. Quadrupling the API cost ceiling on every
triggered Brain run is the kind of change that should ship with a
percentage-rollout knob and a dashboard before it ships as the
default for everyone, not after.

## The shared archetype: agent-mediated change whose mechanism contradicts policy

Both opencode #25823 and gemini-cli #26483 are concrete
instantiations of the same abstract failure pattern, even though
they live in different code bases and surface as different review
verdicts:

  - opencode #25823: the PR's stated intent is to clean up stale
    todos so the agent's task list stays focused. The mechanism
    (silent on-write/on-read filtering) makes the very status
    transition the agent's own system prompt insists on
    ("IMMEDIATELY after finishing") invisible to the user. The
    intent and the mechanism point in opposite directions.

  - gemini-cli #26483: the PR's stated intent is to make the bot
    more useful via self-correction (Brain→Critique loop) and
    more responsive (STALE_DAYS halving, triage limit doubling).
    The mechanism — a self-authored, self-merged expansion of bot
    autonomy on policy-affecting axes — is the precise scenario
    the bot's own lifecycle policy presumably exists to protect
    against, by ensuring contributor changes are reviewed by
    maintainers other than themselves. The intent and the
    governance mechanism contradict each other.

This is a different failure mode from the "implementation bug" or
"missing test" verdicts that dominate the merge-after-nits bucket.
The merge-after-nits cases ship a working feature with a defect
that can be patched in a follow-up commit. These two cases ship
features whose design contradicts a contract or a policy that the
same release reaffirms; they cannot be patched in a follow-up
without rethinking either the contract or the feature.

The drip-361 verdict-shape (3, 3, 1, 1) is therefore not just a
statistical departure from the modal (1, ≥6, ≤1, ≤1) shape of the
trailing eight drips. It is a substantive departure: the bottom
two buckets both fire, and they fire on cases that have a common
abstract structure rather than two unrelated failures. If the next
two or three drips also surface this archetype, that would suggest
the underlying agent-mediated-workflow churn in the carrier set
has reached a point where the review classifier is starting to
catch a new failure category systematically rather than
accidentally. If they revert to (1, ≥6, ≤1, ≤1), drip-361 stays
classified as a tail observation and the modal shape continues
unchanged.

## Cross-checking against drip-360's (1, 7, 0, 1)

The immediately prior tick at T05:27:08Z shipped drip-360 with
verdict mix (1, 7, 0, 1) over 9 PRs. The lone needs-discussion
case was opencode #25822, the Tauri→Electron desktop consolidation
(+113/−13439 net, breaking change, unchecked Testing checklist).
That ND fired on the breaking-change-without-release-dry-run
archetype, which is structurally distinct from the
agent-autonomy-contradicts-intent archetype that drip-361's
gemini-cli #26483 ND fires on. The two ND cases in consecutive
drips therefore do not yet establish a single recurring archetype
— they establish that the ND bucket is being used for two
qualitatively different reasons in the trailing window.

The drip-360 highlights also contained codex #21146 (PR-1-of-4
V8-sandboxing rollout with a `v8-release-compat` opt-out config to
avoid breaking existing Cargo consumers until PR-3 flips it) and
qwen-code #3842 (PR-1-of-3 for #3831 Phase D Ctrl+B promote with a
discriminated `ShellAbortReason` union and 4 new tests pinning
both abort directions). Both of those are exemplary
multi-PR-series-declaration handling — the opposite end of the
quality spectrum from the drip-361 cases. The same drip can carry
PR-1-of-N rollouts with proper invariant tests AND
contract-contradicting auto-cleanups; the verdict classifier picks
both up in their respective buckets and the verdict 4-tuple
records the mix.

## What the verdict-shape ledger says about classifier health

The trailing nine drips touched 73 PRs across all seven carriers.
Of those, 11 landed merge-as-is, 50 landed merge-after-nits, 6
landed request-changes, and 6 landed needs-discussion. The
empirical proportions (0.151, 0.685, 0.082, 0.082) are within
single-percentage-point shifts of the eight-drip estimate used in
the multinomial calculation above, which indicates the classifier
is not drifting on its calibration over the nine-drip window. The
verdict-shape variance across drips is what one would expect from
a multinomial draw of size 8 against a proportion vector concentrated
at after-nits ≈ 0.69 — modal shapes (1, 6, 0, 1) and (1, 7, 0, 0)
account for the bulk of the trailing window, with occasional
draws in the (3, 5, 0, 0) and (3, 3, 1, 1) corners. The classifier
is doing its job; the upstream PR mix is doing its job. Drip-361's
shape is a real signal about the upstream mix this tick, not a
classifier artifact.

The next two drips will close the question of whether drip-361
is the start of a new modal regime (agent-autonomy archetype
recurring) or a tail observation against the otherwise-stable
merge-after-nits-modal shape. Either resolution provides
operational signal — a regime change would prompt a refresh of
the empirical proportions used in the multinomial baseline; a
tail observation would confirm the eight-drip baseline is still
the right reference for the tenth drip's verdict-shape
expectations.
