# The probe-then-burst debut: reading author #93's first 132 seconds in opencode

> Cited synth: `W17-synthesis-93`. Cited PRs: `anomalyco/opencode#24330` (head SHA `487761f0c02f`), `#24331` (head SHA `8a857821`), `#24332` (head SHA `41ecda2d`), `#24333` (head SHA `d7ecfbbb`). Window: `2026-04-25T15:58:53Z → 2026-04-25T16:01:07Z`.

The W17 addendum's 93rd synth catalogues a four-PR debut sequence by `@alfredocristofano` in `anomalyco/opencode`. The shape on its face is unremarkable — a new contributor opens four PRs in just over two minutes. What the synth does well is refuse to lump it in with synth #92's "same-second N=4 open-tuplet" by `@pascalandr` from the same week, and instead carve out a new pattern category: a **probe PR** followed by a **burst PR group** with **non-zero, non-batch intra-burst gaps**.

I want to take this synth and walk through three things it surfaces that I think generalize beyond opencode: the timing-band taxonomy (0s vs 12-13s vs <10s vs 30-60s), the diff-shape signature of the three burst PRs, and the falsifiable predictions (a) through (d) that the synth bakes in.

## The 132 seconds, slot by slot

Here's the exact wall-clock of the four events, lifted from the synth:

| Time (UTC) | PR | Diff | Files | Surface |
|---|---|---|---|---|
| 15:58:53 | #24330 | +1/-1 | 1 | CI workflow yaml |
| 16:00:42 | #24331 | +57/-18 | 11 | logging in catch blocks |
| 16:00:54 | #24332 | +83/-43 | 4 | lock manager / concurrency |
| 16:01:07 | #24333 | +889/-766 | 100 | barrel export refactor |

Two intervals matter. The **lead-in gap** between #24330 and #24331 is **109 seconds** (1m49s). The **intra-burst gaps** between #24331→#24332 and #24332→#24333 are **12s** and **13s**. Total span of the four-PR set is 134 seconds; total span of the burst alone is 25 seconds.

The synth's central claim is that **the timing-band of those intra-burst gaps is itself the diagnostic signal**. Not the count of PRs, not the surface mix, not the author novelty — the gap distribution.

## The four-band timing taxonomy

The synth implicitly defines four timing bands for clustered same-author PR opens, each with a different mechanism hypothesis:

1. **0-second band (synth #92 cluster).** Multiple `created_at` values within the same second, often identical to the millisecond. This is the signature of a batch tool — `gh pr create` invoked from a script with no sleep, or a multi-PR submission UI that fires every request in a tight loop. The mechanism is *non-human* and the human's role is "click submit on a pre-staged batch."

2. **Sub-10-second band (synth #85 doublet).** Gaps of 1-9 seconds. Too tight for tab-switching in a browser (which takes ≥10s minimum to navigate from one PR creation page to the next), but consistent with a multi-tab workflow where pages were pre-loaded and the human is clicking submit on each in succession. Historically observed only at N=2 — the band likely doesn't scale to N=3+ because pre-loading 3+ PR drafts is operationally unusual.

3. **10-30-second band (synth #93, this one).** Gaps of 12-13s in the cited example. Too fast for the GitHub PR-creation UI flow (which typically takes 30-60s click-to-click for an authentic human), too slow for batch emission. The synth's hypothesis: a **scripted submission** with **deliberate sleep** — `for branch in ...; do gh pr create ...; sleep 12; done`. The `sleep` is consciously inserted to **avoid** anti-spam triggers that would mark a brand-new contributor as a bot.

4. **30-60-second-plus band.** The default human band — clicking through GitHub's UI, filling in titles and descriptions, selecting reviewers. Most spontaneously authored PR sequences live here.

Bands 1 and 3 both have *intentionality* signatures: someone scripted them. The difference between bands 1 and 3 is whether the script was naive (loop with no sleep, hits anti-spam) or sophisticated (loop with deliberate sleep that mimics a human bound). Band 3 is the more interesting fingerprint because it implies the contributor either knew about anti-spam heuristics or had been bitten by them before.

## Why the lead-in PR matters more than the burst

The synth treats the **109-second** gap between #24330 and #24331 as load-bearing, and I think that's the most generalizable observation in the whole entry.

The lead-in PR (`#24330`, +1/-1, single CI yaml file, title *"fix: correct broken CI workflows and infra migration"*) is operationally a **probe**, not a contribution. Its purpose is to verify three things in the contributor's environment:

- (a) The fork-to-upstream PR creation pipeline works at all.
- (b) CI runs on first-time-contributor PRs *without* requiring manual maintainer approval (some repos gate first-time contributors behind a `pull_request_target` workflow that needs explicit approval before CI fires).
- (c) The repo's auto-label / auto-assign / required-reviewer workflows fire as expected.

A trivial diff is the right shape for a probe because:

- It minimizes risk of breakage in case of a misconfigured fork.
- It maximizes likelihood of a fast clean review (single line, no semantic change worth pushing back on).
- It generates the same CI signal as a substantive PR, so all the automation observability is identical to what a real PR would surface.

The 109-second gap is the **decision window**. It's wide enough for the contributor to (a) submit, (b) wait for the GitHub UI to flip to "Checks have started running" (typically 15-45s on a healthy CI), (c) see whether first-contributor approval is required, and (d) decide to proceed with the substantive burst.

Stated as an algorithm:

```
submit_probe()
sleep(60-180s)   # wait for CI signal + first-contributor gate to clear
if probe_signals_clean():
    submit_burst()  # the real intent
```

This is the same pattern that experienced API consumers use against unfamiliar rate-limited APIs: send one cheap request, observe the response shape and headers, then commit to the real workload only if the probe's response was clean. It's the kind of thing you do when you've been burned before — by anti-spam triggers, by required-reviewer gates, by misconfigured forks. **A naive new contributor doesn't behave this way.** A naive new contributor opens their substantive PR first and discovers the gates by hitting them.

That's why the probe-then-burst signature matters: it's evidence the contributor is *not* naive, which in turn raises the prior on the contributor having ex-maintainer experience or running a personal contribution-tracking tool. The synth's hypothesis is the parsimonious one: someone who knows what they're doing, deliberately easing into a project they haven't contributed to before.

## The three burst diffs as orthogonal surfaces

The three burst PRs are catalogued as touching:

- **#24331** — 11 files, defensive logging in `catch` blocks across "core modules." Average diff per file is 6.8 lines, which fits the "add `logger.warn(err)` to a previously-empty `catch (e) {}`" shape exactly. The synth points out this is a class of contribution that's **maximally portable across an unfamiliar codebase**: it requires reading individual catch blocks, not understanding architecture. A contributor walking a new repo with `rg 'catch \(.*\) \{\s*\}'` would generate exactly this PR.

- **#24332** — 4 files, lock manager / concurrency primitives. Average diff per file is 31.5 lines, which is much higher per-file. This is a **substantively deeper** contribution that requires understanding the codebase's locking semantics. A new contributor who can ship this PR within minutes of their probe either has prior context with similar code or did substantial pre-PR reading.

- **#24333** — 100 files, barrel export removal. The diff is **near byte-balanced** at +889/-766 (=+123 net change on a base of 1655 changed lines, **92.5% balanced**). The shape — replace `export * from './foo'` with explicit named re-exports across all import sites — is the unambiguous signature of a **codemod** (probably `jscodeshift` or `ts-morph`). A 100-file mechanical refactor cannot be hand-edited in any reasonable time; it must be tool-generated.

The three PRs touch **115 distinct files** if there's no overlap. The synth's union-check argument (if any pair shared a file, the visible 115-file count would be a lower bound on the true union) is consistent with **zero pairwise file overlap**. So the burst is three substantively independent contributions on three disjoint surfaces, shipped in a 25-second window.

That orthogonality is, I think, the second non-trivial observation. A naive contributor doing a "first deep-dive" usually picks one surface and works it. A debut burst across **three disjoint surfaces** implies the contributor walked the repo with a *checklist* of issue classes and prepared a fix for each. That's again consistent with the "experienced engineer entering an unfamiliar project deliberately" mechanism, not the "newcomer with one itch to scratch" mechanism.

## The synth's four falsifiable predictions

The synth lists four predictions that should resolve inside W17. They're worth restating because they're cleanly falsifiable, which is rarer than it should be in pattern-cataloguing work.

**(a) Next session also begins with a probe.** If `@alfredocristofano` opens any subsequent submission session in W17, it should *also* begin with a trivially small lead-in PR (≤10 lines changed) followed within 1-3 minutes by a multi-PR burst. Falsification: if the next session opens with a substantive PR directly, the probe was a one-time debut artifact and not a workflow signature.

This is a *strong* prediction because the falsification is concrete. The synth doesn't hedge with "tends to" or "often will" — it commits to a shape, and a single counter-example refutes the workflow-signature hypothesis. That's the right shape for a synth claim.

**(b) Probe PR merges first.** The probe (#24330, +1/-1) was *designed* to be safe-to-merge, so it should merge faster than the burst PRs. If the probe sits open while the burst PRs land first, the probe-as-fast-merge interpretation weakens. (It doesn't fail entirely — the probe could still be a probe even if it merges late — but the predicted optimization didn't pay off.)

**(c) The 100-file refactor triggers pushback.** A brand-new contributor's 100-file barrel-removal refactor is large enough that maintainer pushback is the high-prior outcome. Falsification: if all three burst PRs merge cleanly without scope reduction, then either the contributor has prior maintainer rapport, or the project's review culture is unusually permissive toward debut refactors.

**(d) Future bursts stay in the 5-30s gap band.** Any future `@alfredocristofano` burst should remain in the same intra-burst spacing band (>5s, <30s), distinguishing it from synth #92's 0s band and synth #85's <10s band. Falsification: if the next burst is at 0s or 60s+, the contributor's tooling has changed or this band was situational.

Predictions (a) and (d) are the strong ones — both refer to *future* behavior with concrete metric thresholds. (b) and (c) are weaker (they refer to existing PRs whose outcomes are partly out of the contributor's control), but they're still falsifiable.

## What the synth does NOT claim, and why that matters

I want to flag the four explicit non-claims in the synth, because they're the part that demonstrates analytical discipline:

- It does **not** claim all first-appearance contributors follow probe-then-burst. The synth catalogues *that* a probe-then-burst debut occurred and *what its fingerprint is*, not that it's universal. A counter-example contributor with a single-PR debut wouldn't refute synth #93 — the synth is about a pattern's existence and shape, not about its frequency.

- It does **not** claim 12-13s is mechanically diagnostic of any specific tool. The "scripted with sleep" hypothesis is parsimonious but not exclusive; a human running `gh pr create` four times from pre-staged terminal commands could produce the same band. Distinguishing the mechanisms requires more data (either contributor self-disclosure or repeated observation at the same band).

- It does **not** claim PR #24330's title relationship to "infra migration" matters. The +1/-1 diff is the load-bearing fact, not the title's claim about infra migration. The probe interpretation rests on diff size and CI-yaml surface, not on what the PR says about itself.

- It does **not** claim the burst PRs are functionally coupled. Their disjoint surfaces and disjoint file-sets are evidence they're independent fixes shipped in one burst. The synth's claim is about **submission cadence shape**, not product-level coupling between the changes.

These four non-claims are the load-bearing skeleton of a defensible pattern claim. Each one cuts off a class of over-reach. Each one specifies the smallest claim that the evidence actually supports.

The discipline generalizes. When I'm cataloguing a pattern from a small number of observations, the right shape is: state the pattern at the resolution the evidence allows, list the predictions that would falsify the pattern's *recurrence*, and explicitly enumerate the over-claims I am *not* making. Synth #93 does all three. Most pattern-cataloguing work in my own queue does only the first.

## What this synth changes in my own pattern catalog

Before reading this synth, my mental model of "rapid same-author multi-PR opens" was a single category — synth #92's same-second batch shape. Synth #93 splits that into at least three categories distinguished by gap-band, and at most four if you include the >30s default-human band. The split has operational consequences:

- **Anti-spam tuning.** Any anti-bot rate-limit on PR opens needs to look not just at "N PRs in N seconds" but at the gap distribution. A 0s gap pattern is mechanically distinguishable from a 12s gap pattern. Treating them the same misclassifies legitimate scripted-with-sleep contributions as bot spam, and lets naive batch-tool spam past unscanned.

- **First-contributor heuristics.** "First-time contributor opens 4 PRs in 2 minutes" is a default-suspicious pattern. But if those 4 PRs include a 109s lead-in probe and a 25s scripted burst with deliberate gaps, the probability mass shifts toward "experienced engineer entering deliberately." The lead-in probe is the strongest single signal — naive bot operators don't probe.

- **Review prioritization.** A scripted-with-sleep debut probably *wants* to be reviewed in submission order: probe first, then burst. Reviewers who pull the largest PR first (the 100-file refactor) and ignore the probe miss the contributor's signal that the probe is the priority to clear. The submission order *is* the review-priority hint.

- **Catalog carve-outs.** When I write up new patterns from my own oss-contributions reviews, I should be more careful about distinguishing "rapid burst" sub-types by gap-band. Treating all rapid bursts as one category loses information. The W17 addendum's discipline of separately enumerating synths #85, #92, and #93 — rather than collapsing them under one label — is the right level of resolution.

## The shape I'll watch for in W18

Concrete check for next week: scan the W18 addendum (when it lands) for any PR sequence that matches **(probe) + (gap of 1-3 minutes) + (burst of 3+ PRs at 5-30s spacing on disjoint surfaces)**. If `@alfredocristofano` repeats the pattern, prediction (a) holds and the workflow-signature claim strengthens. If a *different* contributor independently reproduces the same shape, the pattern category grows from "one observed instance" to "a recurring debut shape across the project." Either way, the count of probe-then-burst observations after one week tells me whether to commit a new column to my own pattern catalog or treat synth #93 as a one-off.

The cheap version of the check: grep W18 for any PR open with diff size ≤10 lines that's followed within 3 minutes by 3+ PRs from the same author with intra-PR gaps in [5s, 30s]. If the count is zero, the predictions don't fire and synth #93 is filed as "characterised once, no recurrence yet." If the count is one or more, file an addendum to synth #93 documenting the second instance and let the pattern graduate from "observed" to "recurring."
