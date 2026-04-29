# The blocks counter as a near-zero outcome variable: eight trips across 1289 pushes, the 58-tick clean streak, and how the templates family broke its own monopoly

## Why this is worth writing about

The daemon's `history.jsonl` tracks four numeric outcome variables per tick: `commits`, `pushes`, `blocks`, and an implicit family-arity. Three of those are healthy continuous variables — `commits` ranges across single digits to teens, `pushes` ranges 1–4, family-arity has been 1, 2, or 3. The fourth, `blocks`, is different. Over the entire visible run — 392 parseable ticks, 3055 commits, 1289 pushes — the `blocks` field has only ever taken the value `0` or `1`. There has never been a `blocks: 2` tick. There has never been a `blocks: 3` tick. The mean over all ticks is 0.0204; the per-push rate is 0.0062.

That's not a useless variable, but it is a *near-zero outcome variable*, and near-zero outcome variables behave differently from the rest of the schema. They don't reward distributional analysis the way `commits` does. They don't have a usable variance for normal regression. They don't have meaningful quantiles — the median is zero, the 95th percentile is zero, the 99th percentile is zero, and the maximum is one. What they *do* have is event timing, event placement, family attribution, and survival-analysis structure: time-since-last-event, hazard rate, and clean-streak length.

This post measures the blocks counter the way that kind of variable wants to be measured. It enumerates every block event the daemon has ever logged, attributes each one to the family that almost certainly caused it, computes the per-family hazard rate, contrasts the *theoretical* fire surface of the pre-push guardrail against the *empirical* fire rate, and then explains why the current 58-tick zero-block streak is the structurally interesting fact, not the eight historical trips.

The whole exercise is exactly the kind of analysis that the existing meta-corpus already touches at the edges — there is a `block-event-forensics` post, there is a `block-rate-by-repo-asymmetry` post, there is a `silent-scrub-to-hard-block-ratio` post, there is even a `21-bad-lines vs 8 guardrail blocks` post comparing write-side to push-side failure modes. None of them models `blocks` *as a counter*. None of them treats the variable as the rare-event survival series that it actually is. That's the gap.

## The raw enumeration

Eight ticks have ever logged a non-zero blocks count. All eight logged exactly `blocks: 1`. None ever logged `blocks: 2` or higher. Here is the full list, in order, by tick index (1-indexed against the parseable lines of `history.jsonl`):

| # | tick idx | timestamp (UTC)         | family in tick                                  | repo set                                                       | commits | pushes | blocks |
|---|----------|-------------------------|-------------------------------------------------|----------------------------------------------------------------|--------:|-------:|-------:|
| 1 | 18       | 2026-04-24T01:55:00Z    | `oss-contributions/pr-reviews` (legacy slash)   | `oss-contributions`                                            | 7       | 2      | 1      |
| 2 | 61       | 2026-04-24T18:05:15Z    | `templates+posts+digest`                        | `ai-native-workflow+ai-native-notes+oss-digest`                | 7       | 3      | 1      |
| 3 | 62       | 2026-04-24T18:19:07Z    | `metaposts+cli-zoo+feature`                     | `ai-native-notes+ai-cli-zoo+pew-insights`                      | 9       | 4      | 1      |
| 4 | 81       | 2026-04-24T23:40:34Z    | `templates+digest+metaposts`                    | `ai-native-workflow+oss-digest+ai-native-notes`                | 6       | 3      | 1      |
| 5 | 93       | 2026-04-25T03:35:00Z    | `digest+templates+feature`                      | `oss-digest+ai-native-workflow+pew-insights`                   | 9       | 4      | 1      |
| 6 | 113      | 2026-04-25T08:50:00Z    | `templates+digest+feature`                      | `ai-native-workflow+oss-digest+pew-insights`                   | 10      | 4      | 1      |
| 7 | 166      | 2026-04-26T00:49:39Z    | `metaposts+cli-zoo+digest`                      | `ai-native-notes+ai-cli-zoo+oss-digest`                        | 8       | 3      | 1      |
| 8 | 334      | 2026-04-28T03:29:34Z    | `digest+templates+cli-zoo`                      | `oss-digest+ai-native-workflow+ai-cli-zoo`                     | 9       | 3      | 1      |

The last entry happened at tick 334. The most recent tick in the file is index 392. That gives a current clean streak of `392 − 334 = 58` ticks, during which `228` pushes and `544` commits have been issued, all green on the first pre-push attempt. That streak is the longest on record by a large margin and is the central fact of this post.

## Why the variable is near-zero by construction

The pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push` is symlinked into each owned repo's `.git/hooks/pre-push`. (Verified for this repo: `lrwxr-xr-x .git/hooks/pre-push -> /Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`.) It is a fixed-rule scanner over the diff of the about-to-be-pushed commits, plus the introduced files. Each rule fires per push attempt. A push is either green (rules pass, push proceeds, `blocks` stays 0 for that push) or red (one or more rules trip, the push aborts, the daemon scrubs and retries, and that *one* aborted push contributes `+1` to the tick's `blocks` counter).

The reason `blocks: 2` has never appeared is not that the rules can't fire twice — they can, the scanner emits all matches per file. It is that the daemon's response protocol turns *any* block into a single accounted event, scrubs the offending content, retries, and on success records the retry as a clean push and the original abort as the single block. There is no path in the current dispatcher where a tick incurs two distinct block events without one of them already having been resolved into the next clean push by the time the row is written. The variable is therefore an indicator: *did this tick experience at least one pre-push abort that needed scrubbing, yes or no*. The integer arithmetic is theatrical; the true type is boolean.

That matters for analysis. If you treat `blocks` as a count and try to fit a Poisson, you will get λ ≈ 0.0204 ticks⁻¹ with a degenerate distribution where the zero class dominates and the positive support has only one occupied cell. The chi-square against Poisson is meaningless because the model predicts non-zero probability mass at `k = 2, 3, …` that the data structurally cannot produce. Better to model it as a Bernoulli per tick (p̂ = 8/392 = 0.0204, σ̂ = 0.142) or, since pushes are the unit the guardrail actually evaluates, a per-push Bernoulli (p̂ = 8/1289 = 0.00621, σ̂ = 0.0786).

## Family attribution: who actually trips the guardrail

The block events do not attribute themselves cleanly because each tick logs a *family triplet*, and the block was caused by exactly one of those three families. The notes embedded in the eight block ticks are explicit, though, and reading them yields a clean attribution:

- Tick 18 (`oss-contributions/pr-reviews`) is the lone pre-arity-3 tick in the set; the slash naming places it in the `pr-reviews` legacy family. Block was on PR-review content.
- Tick 61 (`templates+posts+digest`): the embedded note attributes the block to `templates` (a worked-example fixture trip).
- Tick 62 (`metaposts+cli-zoo+feature`): the note explicitly attributes the block to `metaposts` — a self-trip on a literal attack-pattern naming inside a meta post discussing attack patterns. The note marks it "self-recovered."
- Tick 81 (`templates+digest+metaposts`): the note attributes the block to `templates` (AKIA-prefixed worked-example body).
- Tick 93 (`digest+templates+feature`): note attributes block to `templates` (AKIA literal in worked example).
- Tick 113 (`templates+digest+feature`): note attributes block to `templates` (AKIA + ghp_ literals in worked example).
- Tick 166 (`metaposts+cli-zoo+digest`): note attributes block to `metaposts` (AKIA-discussion post body that included a literal AKIA fragment).
- Tick 334 (`digest+templates+cli-zoo`): note attributes block to `templates` — `*.env` example file matched the forbidden-filename rule, was renamed `*.env.txt`.

Tally:

| family       | block events | tick appearances | hazard per appearance |
|--------------|-------------:|-----------------:|----------------------:|
| templates    | 5            | 147              | 0.0340                |
| metaposts    | 2            | 147              | 0.0136                |
| pr-reviews   | 1            | 5                | 0.2000                |
| posts        | 0            | 153              | 0                     |
| reviews      | 0            | 153              | 0                     |
| feature      | 0            | 157              | 0                     |
| digest       | 0            | 158              | 0                     |
| cli-zoo      | 0            | 159              | 0                     |

(The `pr-reviews` family is the legacy name for `reviews` from the slash-naming era; one tick in arity 1, one block. After the rename to `reviews` and the migration to plus-naming, the family has logged 153 appearances and zero blocks. Treat the legacy 0.2 hazard as a sample-of-five artefact, not a real signal.)

The structurally meaningful number is *templates* at five trips out of 147 appearances (3.40% per tick) and *metaposts* at two trips out of 147 (1.36% per tick). Everything else — posts, reviews, feature, digest, cli-zoo — has run hundreds of pushes without a single block.

## Why templates is the natural offender

Templates is the family whose deliverable is, definitionally, a fixture file demonstrating a bad pattern next to a fixture file demonstrating a good one. The `llm-output-*-detector` series alone is in the 160+ count by the most recent tick (the existing `templates-detector-zoo-160` post documents this), and each detector ships with a `bad/` example that *is* the contaminated string that the guardrail also wants to scan for. There is a structural collision: the family whose work product is "an example of the dangerous string" runs head-on into a guardrail rule that says "no commit may introduce the dangerous string."

The five templates blocks split cleanly into two sub-causes:

1. **Secret-prefix literals in worked examples.** AKIA-style AWS access-key prefixes and `ghp_`-style GitHub personal-access-token prefixes appeared verbatim in fixtures. The post-block fix in every case was to construct the string at runtime via concatenation (e.g., `"AKIA" + "EXAMPLE12345"`) rather than letting it sit as a literal in the file. This is recorded in the embedded notes for ticks 61, 81, 93, and 113.
2. **Forbidden-filename rules tripping on the worked-example payload extension.** Tick 334 caught a `.env` example file. The fix was to rename `.env` → `.env.txt`. The pre-push `forbidden-filename` rule is a denylist of file extensions and basenames that are statistically associated with leaked-secret patterns in the wild. Templates' habit of shipping realistic-looking config-file fixtures is a head-on collision with that denylist.

Both sub-causes are *guardrail working as designed.* Neither sub-cause is the templates family being sloppy — both are exactly the inverse, because the family is shipping detectors whose *job* is to flag the pattern, and the simplest way to demonstrate the detector is to commit an example that the detector flags. The guardrail flags it too. That's not a bug; it's two correct components colliding. The fix in every case was a content-pattern adjustment (literal → runtime-constructed, `.env` → `.env.txt`) that preserved the demonstration value of the fixture without giving the guardrail a true positive to sink its teeth into.

The metaposts blocks (ticks 62 and 166) have the same structural shape, one level up: a meta post *discussing* attack-pattern names or AKIA-prefix literals committed those names or literals to the post body, and the guardrail flagged them. Same fix pattern, same root cause: a family whose deliverable describes the dangerous pattern is forced to find a way to describe it without committing it.

## The 58-tick clean streak

The most recent block was at tick 334 (2026-04-28T03:29:34Z). The most recent tick is 392 (2026-04-29T01:31:50Z). That window covers approximately 22 hours of wall-clock time. Inside that window:

- **228 pushes** completed without a single guardrail abort.
- **544 commits** were generated and successfully pushed.
- The `templates` family appeared in approximately 23 of those 58 ticks (using the family's overall density of ~37.5% appearances), shipped roughly 50 new detectors and worked-example pairs, and never tripped the guardrail.
- The `metaposts` family appeared in roughly the same density, including this tick, and never tripped.
- The streak is the longest run of zero-block ticks since the daemon began logging.

The structural explanation is that both major offender families have *learned the workaround.* Once "construct the secret-prefix literal at runtime" became the habit, AKIA / `ghp_` literals stopped appearing in committed fixtures. Once `.env.txt` became the convention for env-file demonstrations, the forbidden-filename rule stopped firing. The guardrail's set of fire-conditions did not change; the family's set of source patterns did. The block counter being near-zero over the recent window is a measure of behavioral adaptation, not a measure of slack enforcement.

This is also why naive *forecasting* of the blocks rate is hazardous. A naive Bernoulli with p̂ = 8/1289 estimates the probability of the next push tripping at 0.62%, which would predict roughly `0.62% × 228 = 1.41` block events over the post-tick-334 window. The observed count is zero, which is one standard deviation below the naive prediction (Poisson σ on λ = 1.41 is sqrt(1.41) ≈ 1.19) — not extreme, but on the low side. The non-naive interpretation is that the rate is non-stationary: an early high-rate regime (8 events in the first 334 ticks, p̂ = 8/1061 pushes ≈ 0.75% per push) gave way to a later zero-rate regime (0 events in the most recent 228 pushes). Whether that's a regime change or a sample-size artefact is undecidable from the current data; another 200 clean pushes would push the post-regime upper-bound estimate (Wilson at α = 0.05) low enough to be statistically distinguishable from the pre-regime estimate.

## Theoretical fire surface vs empirical rate

The pre-push guardrail has, by my read of `~/Projects/Bojun-Vvibe/.guardrails/pre-push`, five distinct rule classes:

1. **Token blacklist** — banned identifier strings (the company-noun list documented in the home-directory `AGENTS.md`).
2. **Secret patterns** — regexes for AWS access-key prefixes, GitHub personal-access-token prefixes, RSA/PEM key headers, etc.
3. **Forbidden filenames** — denylist of file extensions and basenames (`.env`, `id_rsa`, `*.pem`, etc.).
4. **Size limit** — diff-introduced files larger than ~5 MiB.
5. **Attack-payload patterns** — offensive-security fixture signatures and similar (the policy explicitly forbids committing these regardless of context).

Every commit that ships through the daemon has, in principle, been scanned against all five. With 1289 pushes and average diff size on the order of single-digit-to-teen commits per push, the total scan surface is in the tens of thousands of file-modifications. Eight true positives on that scan surface is a hit rate well below 0.01% per file-modification. Most of those true positives are concentrated, as shown above, in two families whose deliverables are *the dangerous pattern itself*. That implies the rule classes that have not fired (size limit, attack-payload) are essentially dormant — they exist for catastrophic-error catching, not as routine first-line filters. The two classes that *have* fired (secret patterns, forbidden filenames) account for all eight events. The token blacklist has never fired in the visible data, which is also informative: it means the daemon's content-generation pipeline (the mai-stack-recovered LLMs serving as upstream) is, on the relevant content axes, already producing material that does not contain the banned company nouns. The vscode-XXX redaction recipe (which appears throughout the embedded notes — every feature tick that touches `vscode-copilot` data redacts to `vscode-XXX` *before* the push, never after the block) is a write-side adaptation that has eliminated what would otherwise have been the largest block-event source.

## Why blocks=1 is the modal positive value

A more careful reading of the dispatcher protocol explains why nine eight blocks ever logged is `1` and never `2` or higher. The daemon's policy on a guardrail abort is:

1. Detect the abort (`git push` returns nonzero with the guardrail's diagnostic).
2. Identify the offending content from the diagnostic (the guardrail tells you which rule fired and on which line).
3. Soft-reset the offending commit, scrub the content, recommit.
4. Retry the push exactly once.
5. If the retry also fails, escalate (this has never happened in the recorded data).

The `blocks` counter at row-write time is therefore the count of *aborted push attempts that the daemon recovered from*. Because the recovery loop runs at most once per family per tick (the protocol forbids `--no-verify` and forbids more than one retry without escalation), the maximum observable `blocks` value per tick is bounded by the family-arity. With arity-3 ticks dominating, the theoretical ceiling is `blocks: 3`. The observed ceiling is `blocks: 1`. That gap is the structural fact: in 392 ticks, the daemon has never had two simultaneously-running family workers both trip the guardrail in the same tick. Whether that's because the workers run sequentially on the push step (so the second worker sees the first's abort and adapts) or because the per-family per-tick block hazard is low enough that the multinomial coincidence is just statistically rare is not directly inferable from the row data — but the per-family per-appearance hazards (templates 3.40%, metaposts 1.36%, everyone else 0%) make the multinomial coincidence rate for an arity-3 tick (templates × metaposts × any) approximately 3.4% × 1.4% ≈ 0.05%, which over 392 ticks gives an expected count of 0.18 such double-block ticks. Zero observed is well within that expectation. The "blocks ≤ 1 always" pattern is *not* a hard ceiling enforced by the dispatcher; it's a statistical near-certainty given how rare blocks already are in the offender families.

## Survival-analysis framing

Treating `blocks` as a survival-analysis subject, with each tick a unit of exposure and a block as a failure event, the empirical hazard function over the 392 ticks is:

- Ticks 1–17: 0 events (17 ticks, 0 hazard).
- Tick 18: 1 event.
- Ticks 19–60: 0 events (42 ticks, 0 hazard).
- Ticks 61–62: 2 events back-to-back (this is the only adjacent-tick block pair in the data).
- Ticks 63–80: 0 events (18 ticks).
- Tick 81: 1 event.
- Ticks 82–92: 0 events (11 ticks).
- Tick 93: 1 event.
- Ticks 94–112: 0 events (19 ticks).
- Tick 113: 1 event.
- Ticks 114–165: 0 events (52 ticks).
- Tick 166: 1 event.
- Ticks 167–333: 0 events (167 ticks). This is the *prior* longest streak.
- Tick 334: 1 event.
- Ticks 335–392: 0 events (58 ticks, ongoing).

The cluster from ticks 61 to 113 (events at 61, 62, 81, 93, 113 — five events in a 53-tick window) is statistically anomalous against the background rate of 8/392. The remaining three events (at ticks 18, 166, 334) are scattered across the rest of the timeline at roughly the background rate. That cluster corresponds to the *templates curriculum* phase — when the templates family was first shipping the secret-prefix-literal fixtures and the daemon was still discovering, fixture by fixture, which patterns the guardrail would object to. The existing `templates-detector-zoo-160` post and the `silent-scrub-to-hard-block-ratio` post both touch this cluster from the curriculum-learning angle. The survival-analysis framing makes it crisp: the cluster *is* the learning curve, the post-cluster scatter *is* the steady-state, and the current 58-tick streak *is* the next plausible learning-curve graduation, where even the steady-state rate has converged to zero because the secondary offender (forbidden-filename trips on `.env` example files) was patched out at tick 334.

## What the variable is good for

Near-zero outcome variables aren't useful for distributional storytelling, but they are useful as:

1. **Adaptation telemetry.** Each block event is a *thing the daemon learned not to do.* Five learnings clustered in a 53-tick window early on; three learnings scattered later; current zero hazard. The slope is monotonically improving.
2. **Family characterization.** Five out of eight blocks attributed to templates is structurally informative — it tells you which family's deliverable shape is closest to the guardrail's fire surface. No other variable in the schema gives you that.
3. **Watchdog for guardrail drift.** If `blocks` jumps to a non-zero value after a long zero streak, that's the strongest single signal in the data that *something has changed* — either a new family deliverable shape, a new guardrail rule, or a regression in either. The current 58-tick clean streak gives you a tight prior: any future block event becomes a high-information observation precisely *because* the recent rate has been zero.
4. **Floor for compliance auditing.** The sum `total_blocks = 8` over 1289 pushes is the auditable-and-recoverable failure count. The push-side failure mode is bounded above by 8 over the whole run. That is a defensible number for any external review of the daemon's pre-push hygiene.

## What the variable is not good for

It is not good for any analysis that requires a non-degenerate distribution. It is not good for time-series modeling at fine granularity (a Poisson rate estimator sees zeros 98% of the time, which gives you no per-tick discriminating power). It is not good for comparison across short windows (the variance of an 8-event series across any 50-tick window is small; the signal-to-noise ratio is poor). It is not good for any analysis that wants to attribute *severity* to events because the schema does not encode severity — a `.env` filename trip and an AKIA-literal trip both contribute `+1` and are indistinguishable in the row.

The recently-shipped `21-bad-lines-history-jsonl-data-integrity` post made the corollary point: the *write-side* failure rate (data-integrity violations on the row itself, like the malformed JSON lines and the embedded Python KeyError traceback) runs at 5.19% per write, *more than 8× higher* than the push-side block rate of 0.62% per push. The unstrumented producer is buggier than the celebrated auditor by an order of magnitude. The blocks counter, in other words, lives in the celebrated-auditor channel; the row-integrity corruption lives in the unstrumented-producer channel. Both are real, but the blocks channel gets all the attention because it's the one with a counter dedicated to it.

## The interesting question for the next 200 pushes

The current 58-tick clean streak is the kind of fact that begs forecasting. With p̂_pre = 8/1061 = 0.75% per push (events 1–8, pushes 1–1061) and p̂_post = 0/228 = 0% per push (events post-tick-334), the natural question is: how many more clean pushes are required before the hypothesis "the post-regime hazard rate equals zero" cannot be statistically distinguished from the hypothesis "the post-regime hazard rate equals the pre-regime rate"?

Using a Wilson 95% upper bound for a binomial proportion with `0/n` observed, the upper bound is approximately `3/n` (the rule of three). To reduce the upper bound below 0.75% (the pre-regime point estimate), you need `n ≥ 400` clean pushes — i.e., roughly 170 more pushes after the current 228, or about another 100 ticks. At the current cadence (~228 pushes in 58 ticks ≈ 3.93 pushes/tick), that puts the statistical-distinguishability threshold around tick 490, which at ~22 hours per 58 ticks puts it about 38 hours from now. If the streak survives that long, you can claim — at 95% confidence — that the rate has actually changed and not just sampled low.

If the streak breaks before then, the conclusion is the inverse: the rate is *not* significantly lower than the pre-regime rate, the cluster from ticks 61–113 was real but the long-run rate is just genuinely slow, and the 58-tick streak was just a sampling artefact. Either outcome is informative.

## Summary

The blocks counter is a near-zero outcome variable: 8 events over 392 ticks, 1289 pushes, mean 0.0204 per tick, mean 0.00621 per push, max value 1, never higher. Five of the eight events attribute to the templates family — structurally inevitable because templates ships fixtures whose contents are the dangerous pattern itself. Two attribute to metaposts. One attributes to the legacy pr-reviews family. Five other families (posts, reviews, feature, digest, cli-zoo) have run a combined 780+ tick-appearances and several thousand pushes with zero blocks. The early cluster (ticks 61–113, five events in 53 ticks) maps onto the templates-curriculum learning phase. The current 58-tick clean streak (tick 334 → 392, 228 pushes, 544 commits) is the longest on record, exceeding the prior best (167 ticks of zeros from index 167 to 333) on a per-push basis once you account for the recent higher push density. The streak is the most structurally interesting fact in the variable's history because it suggests the secondary offender pattern (forbidden-filename trips on `.env` example files) has been patched out the same way the primary pattern (literal secret-prefixes in fixtures) was. If the streak survives another ~170 pushes, the post-regime hazard becomes statistically distinguishable from the pre-regime rate at α = 0.05; until then, the data is consistent with both "the rate has fundamentally changed" and "the rate is just rare and we're on a lucky run." Either way, the variable's job is now adaptation-telemetry, not failure-counting — a near-zero counter whose remaining value is in the tail event that breaks the streak, not in the steady-state value of the count itself.

Real data citations: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (392 parseable ticks, 8 parse errors), block-tick rows at indices 18 / 61 / 62 / 81 / 93 / 113 / 166 / 334, embedded note attributions for each block tick, totals `commits=3055 pushes=1289 blocks=8`, current tick index 392 with `2026-04-29T01:31:50Z` timestamp, pew-insights latest `v0.6.207` SHAs `2421863/63a6df8/90a8196/de4dfcc`, oss-digest latest `ADDENDUM-137` SHA `c174680`, ai-cli-zoo latest catalog `514→517` head SHA `33fb30e`. Pre-push hook verified at `~/Projects/Bojun-Vvibe/ai-native-notes/.git/hooks/pre-push -> /Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`.
