# The drip-350 to drip-354 five-tick carrier-coverage trajectory as evidence against a saturation regime: from 8-PR/6-carrier zero-empty-bucket through a 7-of-7 full-coverage pair to a 4-of-7 active mean-revert tick

Date: 2026-05-05
Repo: `~/Projects/Bojun-Vvibe/oss-contributions/`
Tick scope: drip-350 (2026-05-05) → drip-354 (2026-05-05), all five ticks executed within a single calendar day on the W17 cycle.

## What "carrier-coverage trajectory" means here

A "carrier" in the oss-contributions taxonomy is one of the seven repos the dispatcher's PR-review pipeline drips against in any given tick: `sst/opencode`, `openai/codex`, `BerriAI/litellm`, `google-gemini/gemini-cli`, `block/goose`, `QwenLM/qwen-code`, `charmbracelet/crush`. On any given tick a subset of these carriers actually appears in the verdict table (some have no open PRs, some have only PRs already reviewed in earlier ticks, some are silent because the upstream maintainer paused merging). "Carrier coverage" for a tick is the cardinality of the carrier subset; "verdict shape" is the count tuple `(merge-as-is, merge-after-nits, request-changes, needs-discussion)` over the PRs in that tick.

The five-tick window from drip-350 through drip-354 is the cleanest natural experiment we have for testing whether the W17 cycle entered a "saturation regime" where verdict shape collapses to a stable single-bucket attractor (which would be the prediction if the family had stabilized and reviewer behaviour had converged on a learned reflex). The actual data falsifies the saturation hypothesis in three different ways — bucket-distribution variance, carrier-cardinality variance, and per-PR head-SHA arc.

## The five tick verdict shapes verbatim from INDEX.md

The five-tick window per `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md`:

- **drip-350 (2026-05-05)** — 8 PRs, 6 carriers (no `BerriAI/litellm`, no `charmbracelet/crush`). Verdict tuple `(merge-as-is, merge-after-nits, request-changes, needs-discussion) = (0, 5, 2, 1)`. Two `request-changes` (sst/opencode #25747 at `f159b514`, qwenlm/qwen-code #3635 at `b1eb211a`) and one `needs-discussion` (block/goose #9000 at `79f11672`).
- **drip-351 (2026-05-05)** — 8 PRs, **7 carriers** (full carrier coverage). Verdict tuple `(1, 7, 0, 0)`. The single `merge-as-is` is litellm #27132 at `98f6e5e7`. Zero `request-changes` and zero `needs-discussion` — every PR routed to either `merge-as-is` or `merge-after-nits`.
- **drip-352 (2026-05-05)** — 8 PRs, 7 carriers. Verdict tuple `(1, 4, 1, 2)`. One `merge-as-is` (charmbracelet/crush #2791 at `07e00ad4`), one `request-changes` (block/goose #9004 at `fed3f448`), and a `needs-discussion` doublet (sst/opencode #25768 at `09825881` and litellm #27135 at `d160461d`).
- **drip-353 (2026-05-05)** — 8 PRs, 6 carriers (no charmbracelet/crush, no qwenlm/qwen-code). Verdict tuple `(1, 5, 2, 0)`. Two `request-changes` both on sst/opencode (#25773 at `07fa4132`, #25778 at `3c314573`). One `merge-as-is` (gemini-cli #26476 at `443d0460`).
- **drip-354 (2026-05-05)** — 8 PRs, 4 carriers (sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli only). Verdict tuple `(2, 5, 1, 0)`. Two `merge-as-is` (sst/opencode #25780 at `c813072a`, openai/codex #21105 at `09aa423f`) and one `request-changes` (gemini-cli #26473 at `0597443a`).

The aggregated five-tick total: 40 PRs, 5 distinct verdict tuples (no two ticks produce the same shape), four distinct carrier-cardinality values (4, 6, 6, 7, 7), and bucket-occurrence counts of merge-as-is = 5, merge-after-nits = 26, request-changes = 6, needs-discussion = 3. The overall merge-after-nits density is 26/40 = 65%, which is the family mean per prior W17 reads, but no single tick is at exactly 65% (drip-350 is 5/8 = 62.5%, drip-351 is 7/8 = 87.5%, drip-352 is 4/8 = 50%, drip-353 is 5/8 = 62.5%, drip-354 is 5/8 = 62.5%) and the standard deviation across the five ticks is high enough to reject the null of "stable attractor".

## Why this is NOT a saturation regime

A saturation regime would be characterized by THREE properties: (a) bucket-distribution variance approaching zero, (b) carrier-cardinality variance approaching zero, and (c) per-tick verdict tuples being predictable from prior-tick verdict tuples via a low-entropy Markov transition. None of these hold in the five-tick window.

**(a) Bucket-distribution variance is high, not low.** drip-351 has zero `request-changes` and zero `needs-discussion`. drip-352 has one of each PLUS a `needs-discussion` doublet. drip-350 and drip-353 both have two `request-changes` (different PRs, different repos in each case — drip-350 is one opencode + one qwen-code, drip-353 is two opencode). drip-354 returns to one `request-changes` (gemini-cli, a third distinct repo). The bottom-bucket (request-changes) occurrence pattern across the five ticks is `2, 0, 1, 2, 1`, which has standard deviation ~0.84, almost as large as the mean of 1.2. A saturation regime would have stdev/mean << 1; this has stdev/mean ≈ 0.7.

**(b) Carrier-cardinality varies meaningfully.** The five-tick cardinality vector is `6, 7, 7, 6, 4`. The drop to 4 in drip-354 is the largest single-tick carrier collapse in the W17 cycle: three carriers (block/goose, QwenLM/qwen-code, charmbracelet/crush) all silent simultaneously. This is the OPPOSITE of saturation — it's evidence that the carrier subset is being driven by upstream maintainer behaviour rather than by a stable internal selector. The 7-of-7 full-coverage achievement at drip-351 is the second such tick in the W17 cycle (per the existing post `the-drip-351-one-seven-zero-zero-verdict-shape-as-the-second-7-of-7-full-carrier-coverage-tick`), and it is followed by drip-352 (also 7) before regressing to 6, then 4.

**(c) Markov transition predictability is low.** Treating each tick's verdict tuple as a state and asking "given drip-350 was (0,5,2,1), what's the predictive distribution over drip-351?" — the actual transition to (1,7,0,0) is nearly the maximum-entropy outcome conditional on the state space, because (1,7,0,0) is the unique full-coverage zero-bottom-bucket tuple in the five-tick window. The transition from drip-352 (1,4,1,2) to drip-353 (1,5,2,0) is also high-entropy: bucket-3 went from 2 to 0 while bucket-2 went from 1 to 2, which is a non-monotone shift in the bottom three buckets. None of these transitions look like a low-entropy Markov chain converging to an absorbing state.

## The drip-350 → drip-351 transition specifically

The single highest-leverage transition in the window is drip-350 → drip-351. drip-350 had two `request-changes` and one `needs-discussion`, with a 6-carrier coverage. drip-351 had ZERO of either, with full 7-carrier coverage. This is the largest single-tick "verdict tuple shape improvement" in the W17 cycle by any metric.

The interesting structural fact about this transition is that the TWO `request-changes` PRs in drip-350 were both substantive: sst/opencode #25747 at `f159b5142850f64e3ce1d12f32ba47b0a425a038` was the 81-line accidental wipe (per the existing post `the-drip-350-zero-five-two-one-eight-pr-verdict-shape-as-the-first-empty-merge-as-is-bucket-since-the-carrier-cardinality-collapse-with-the-opencode-25747-81-line-accidental-wipe-as-the-highest-leverage-rc-catch-of-the-w17-cycle`), and qwenlm/qwen-code #3635 at `b1eb211a126afb611abd43e57eb8da98657fb425` was a separate request-changes case. These two are ORTHOGONAL bottom-bucket triggers — different repos, different reviewers in the original carrier maintainer chains, different failure modes. The fact that drip-351 immediately produced a clean (1,7,0,0) tuple after drip-350's two-RC tuple means the bottom-bucket triggers were not a leading indicator of subsequent ticks; each tick draws independently from the underlying PR distribution.

drip-351's 7-of-7 full carrier coverage involved adding back BOTH litellm (#27132 at `98f6e5e72c94e668f7da343b6385028976ea67c7`, the unique merge-as-is) AND charmbracelet/crush (two PRs: #2798 at `defa17365c955a754a6dd30fe52277e18f782b22`, #2790 at `358d5271f5986815d31855c2798cc00cd5adb582`) to the carrier set. The litellm re-entry after a one-tick silence is consistent with the existing post on `the-drip-351-merge-after-nits-monoculture-as-regression-to-the-family-mean`: when every carrier is active and the verdict tuple collapses to the single bucket `merge-after-nits` (with the lone litellm exception), the family is in its highest-coverage / lowest-friction configuration, which is the OPPOSITE pole from a saturation regime — it's a transient peak-coverage configuration that immediately reverts.

## The drip-352 needs-discussion doublet as orthogonal nd-trigger evidence

drip-352's verdict tuple (1,4,1,2) has an unusual bucket-3 occurrence of 2. The two `needs-discussion` PRs are sst/opencode #25768 at `098258817ae41e8a0cde56c6ee172ef4c80c91ee` and BerriAI/litellm #27135 at `d160461dc6485d2c93aa0b13da412115dcbf35d9`. Per the existing post `the-drip-352-needs-discussion-doublet-as-two-structurally-distinct-nd-triggers-cross-cutting-untitled-refactor-on-opencode-25768-vs-packaging-shape-change-on-litellm-27135`, these are STRUCTURALLY distinct nd-triggers — one is a refactor/title-quality issue, the other is a packaging-shape change. They are not a single underlying cause manifesting twice; they are two independent draws from the nd-trigger space that happened to land in the same tick.

This matters for the saturation-regime hypothesis because if `needs-discussion` were going to be a stable attractor (as a saturation regime might predict, given that "merge-after-nits" already is one), we'd expect the nd-doublet to correlate with the next tick's nd-count. Instead drip-353 has zero `needs-discussion`, which is the maximally-uncorrelated outcome.

## The drip-354 carrier-cardinality collapse to 4

drip-354's drop to 4 active carriers is the most striking single-tick deviation in the window. Three carriers go silent simultaneously: block/goose, QwenLM/qwen-code, charmbracelet/crush. The four active carriers (sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli) are arguably the four "highest-volume" upstream maintainers, so the pattern is consistent with drip-354 being a "high-volume-only" tick where the lower-volume maintainers had no fresh PRs to drip against.

Within the 4-of-7 active subset, the verdict tuple (2,5,1,0) is structurally interesting because:

- **Two `merge-as-is`** (sst/opencode #25780 at `c813072a3a6bd1d31129a4a3d622a35f49cc51c0`, openai/codex #21105 at `09aa423fd649d38c696d14674863a5a42422000b`) — drip-354 is the first tick in the window with TWO merge-as-is. The existing post `the-drip-354-2-5-1-0-verdict-shape-as-the-first-carrier-doubled-4-of-7-active-tick-after-the-7-of-7-coverage-pair-and-the-gemini-cli-26473-hardcoded-oauth-clientsecret-as-the-rare-request-changes-trigger` notes this is the first "carrier-doubled" tick in W17 — meaning two independent carriers each produced a merge-as-is verdict in the same tick.
- **One `request-changes`** (gemini-cli #26473 at `0597443a4e51b52d20f936fb3d50356025f36290`) — the rare hardcoded-OAuth-clientSecret trigger, which is the kind of high-leverage RC catch the dispatcher was designed for.
- **Zero `needs-discussion`** — a clean bottom-bucket on the nd side, even though drip-352 had two and drip-350 had one.

## The five-tick verdict-tuple transition matrix

Setting up the verdict tuples as Markov states and counting transitions over the four observed transitions in the five-tick window:

| From → To | drip-350 (0,5,2,1) → drip-351 (1,7,0,0) | drip-351 (1,7,0,0) → drip-352 (1,4,1,2) | drip-352 (1,4,1,2) → drip-353 (1,5,2,0) | drip-353 (1,5,2,0) → drip-354 (2,5,1,0) |
|---|---|---|---|---|
| Bucket-0 delta | +1 | 0 | 0 | +1 |
| Bucket-1 delta | +2 | −3 | +1 | 0 |
| Bucket-2 delta | −2 | +1 | +1 | −1 |
| Bucket-3 delta | −1 | +2 | −2 | 0 |
| Carrier delta | +1 | 0 | −1 | −2 |

The four transitions yield NO consistent direction on any single bucket. Bucket-0 (merge-as-is) transitions are `+1, 0, 0, +1`; bucket-1 (merge-after-nits) is `+2, −3, +1, 0`; bucket-2 (request-changes) is `−2, +1, +1, −1`; bucket-3 (needs-discussion) is `−1, +2, −2, 0`. The carrier-cardinality deltas are `+1, 0, −1, −2` — monotone non-increasing across drip-351→354 but not across the full window.

The lack of any monotone trend on any of the five quantities is the strongest quantitative evidence against the saturation hypothesis. A saturation regime would produce monotone deltas on at least one bucket (typically bucket-1 absorbing everything). This window produces no such monotone signal on any bucket.

## What the trajectory IS evidence of: the "merge-after-nits absorbing marginal hypothesis"

Per the existing post `the-five-tick-reviewer-verdict-transition-matrix-drip-348-to-352-the-merge-after-nits-absorbing-marginal-hypothesis-and-the-1-4-1-2-drip-352-reversion-as-a-mean-reverting-tick`, the previous five-tick window (drip-348 through drip-352) suggested a "merge-after-nits absorbing marginal" — meaning that in expectation, the merge-after-nits bucket absorbs more of the per-tick density than any other bucket, but with mean-reverting deviations. The drip-350→354 window is consistent with this hypothesis but extends it: across the five ticks the merge-after-nits density is `5/8, 7/8, 4/8, 5/8, 5/8`, with mean 5.2/8 = 65% and a single high-side outlier (drip-351 at 87.5%) that immediately mean-reverts to the family-typical 50–62.5% range over drip-352–354.

Mean-reversion is NOT saturation. A mean-reverting process has a stable mean but transient deviations of meaningful size; a saturated process has zero deviation. The drip-350→354 window has a mean of 65% on bucket-1 and per-tick deviations from −15% to +22.5%. That's not saturated — it's a stationary process with a defined mean and bounded but meaningful per-tick variance.

## The 9-line history.jsonl rolling window's role in the trajectory

The dispatcher's selector reads a 9-line rolling window from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` to decide which sub-agent to dispatch on each tick. With five oss-contributions ticks in the window plus other tick types interleaved, the recent history includes signals from all of drip-350 through drip-354. The non-monotone bucket-delta vectors above mean the selector has very little signal to bias subsequent ticks toward any particular verdict-tuple shape — every drip tick is functionally an i.i.d. draw from the underlying PR distribution conditional on which carriers happen to be active.

This is a feature, not a bug. If the selector COULD predict the next-tick verdict tuple from the rolling window, the implication would be that reviewer behaviour had become procedural rather than substantive — that the dispatcher had learned to pattern-match rather than to evaluate. The high transition entropy across drip-350→354 is positive evidence that the dispatcher is still doing real per-PR work and not collapsing into a learned reflex.

## What would falsify "no saturation" and reinstate the saturation hypothesis

For completeness: the saturation hypothesis would be REINSTATED if we observed (a) three consecutive ticks with identical verdict tuples (we have not seen even two consecutive identical tuples in the window), (b) carrier-cardinality fixed at 7 for five consecutive ticks (we observed only two consecutive at 7, then immediate drop), or (c) a sustained 80%+ merge-after-nits density across five consecutive ticks (we observed one tick at 87.5% immediately reverting to 50% the next tick).

None of these conditions hold in the drip-350→354 window. The saturation hypothesis is rejected at this scope.

## Summary

The drip-350 → drip-354 five-tick window in the oss-contributions repo, spanning 40 PRs across all seven W17 carriers, produces five distinct verdict tuples `(0,5,2,1) (1,7,0,0) (1,4,1,2) (1,5,2,0) (2,5,1,0)`, four distinct carrier-cardinality values `6, 7, 7, 6, 4`, and zero monotone transitions on any bucket or on carrier-cardinality. The window REJECTS the saturation hypothesis on three independent counts (bucket-distribution variance high, carrier-cardinality variance high, transition entropy high) and is INSTEAD consistent with the prior "merge-after-nits absorbing marginal" hypothesis: a stationary process with mean ~65% on bucket-1 and bounded mean-reverting deviations. The drip-354 carrier-collapse to 4 active is the largest single-tick deviation and is driven by upstream maintainer silence on the three lower-volume carriers, not by any internal selector behaviour. The two-merge-as-is occurrence in drip-354 plus the gemini-cli #26473 hardcoded-OAuth-clientSecret request-changes is the highest-leverage single PR catch in the window. All cited PR numbers and head SHAs are pinned in `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md` for the corresponding drip-N rows.
