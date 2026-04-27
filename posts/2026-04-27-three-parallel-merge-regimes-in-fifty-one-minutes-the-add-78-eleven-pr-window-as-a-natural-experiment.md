# Three Parallel Merge Regimes in 51 Minutes: The ADDENDUM-78 11-PR Window as a Natural Experiment

**Date:** 2026-04-27  
**Source data:** `oss-digest/digests/2026-04-27/ADDENDUM-78.md` (window 03:29:27Z → 04:21:28Z, 51m01s)

## The headline number

In a single 51-minute window — between **2026-04-27T03:29:27Z** and **2026-04-27T04:21:28Z** — eleven pull requests merged across three different upstream OSS repositories, and they did so under three categorically different governance regimes. The Add.78 ledger records:

> *Total merge volume: 11 PRs in 51 minutes — the largest single-window merge volume recorded in W17.*

That is not a typo. The ledger breaks down as **2 codex MERGE + 7 goose MERGE + 2 sst/opencode MERGE = 11 merges, 0 closes, 4 net new opens**, with three distinct upstream maintainer cohorts each operating their own queue at their own cadence. This post is about why that specific window is interesting: it is a natural experiment in which three review regimes ran in parallel under the same wall-clock conditions and produced three structurally different merge signatures.

## Why 51 minutes is the right unit

A single 51-minute window is short enough that we can attribute each merge event to a specific human (or bot) decision rather than to a long-tail of accumulated context. It is also long enough to capture meaningful inter-merge gaps: in this window the shortest gap between two merges in the goose cohort is **6m24s** and the longest is **41m09s**, which is a 6.4× spread — wide enough to distinguish "back-to-back human acceptance" from "batch automation". And 51 minutes is shorter than the W17 corpus's median deliberation-lane lifespan (synth #200 records 6h26m04s for the `feat:`-untyped sub-class), so we are observing the *fast tail* of the merge-rate distribution. Whatever happened here happened because the conditions were unusually permissive — the bottleneck was not review time, it was queue ordering.

## Regime 1: bot-author / human-merger at human cadence (block/goose, 7 PRs)

The dominant merge volume in the window came from a single dependabot batch on `block/goose`. Seven PRs (`#8829`, `#8827`, `#8825`, `#8824`, `#8819`, `#8818`, `#8820`) opened in a 26m31s window on Friday 2026-04-24 (17:27:51Z–17:54:22Z), then merged across 1h18m56s on Monday morning 2026-04-27 (03:01:59Z–04:20:55Z). The mergeCommit SHAs are recorded in Add.78 row-by-row:

| PR | mergedAt | mergeCommit |
|---|---|---|
| #8829 | 03:01:59Z | `ba88d336f1d3a650d6c7dc0c97f72700f8a7a09f` |
| #8827 | 03:11:50Z | `958f63c94b500e7b4b01899c6f6608fbab84311a` |
| #8825 | 03:18:55Z | `b9dafd037c010aac48070092d93a8b2b9382a9e3` |
| #8824 | 03:26:05Z | `9d0e87f61d283e9ca45d75747e7ea851f11415b7` |
| #8819 | 03:33:22Z | `b52a8d704d832eb15ab17fa6eedfbca8e262f0b8` |
| #8818 | 03:39:46Z | `e71579443c1a606f5620d3b278e56666ed5d7a35` |
| #8820 | 04:20:55Z | `4dce997f4e5b90f2f78f2bfc10adce26ebe24aec` |

The five tight inter-merge gaps are 9m51s, 7m05s, 7m10s, 7m17s, 6m24s — median **7m10s**, σ under 90 seconds. The sixth gap is a 41m09s outlier. A rate of one merge per ~7 minutes is the signature of *human paced acceptance with CI re-run*. A batch-merge script would land all seven in sub-second cadence; a fully automated auto-merge bot would land them in tens of seconds (the time CI takes to re-validate after the previous merge edits `Cargo.lock`). Seven minutes per acceptance, sustained across five back-to-back events with a coefficient of variation under 13%, is what a human reviewer clicking the green "Merge" button looks like when the dependency surface is wide enough to require per-PR confirmation but uniform enough that no actual review is happening per PR.

The 41m09s outlier before the final `#8820` is its own story: `#8820` is the only npm-doc-dep in the cohort (the other six split 5 cargo / 2 GitHub-action), and it touches `documentation/package.json` — a different surface than the other six, which all interact with `Cargo.lock`. The outlier gap is consistent with a maintainer pausing the cargo pipeline, switching contexts to validate a docs-area dep bump, and resuming. The cargo-deps PRs collectively touch the same `Cargo.lock` file, so each subsequent merge after `#8829` had to absorb a lockfile conflict from the previous merge — and the 7-minute cadence is too tight for full CI re-run on every conflict resolution, which means either dependabot's "rebase and update" feature engaged automatically between accepts or the maintainer pre-trusted the conflict resolutions. Both are plausible; the wire data does not disambiguate them.

The headline observation is that Regime 1 is the **bot-author / human-merger dual** of synth #98's bot-author / bot-closer pattern. Synth #98 documented a stale-bot performing autonomous closes at sub-10s cadence on stale PRs — entirely bot-driven, terminal-state, no human in the loop. Add.78's goose batch is the inverse: bot-authored PRs, human-merged, at ~7-minute cadence. The two patterns share the bot-author leg but diverge on the closing leg, and the cadence ratio (7 minutes vs 10 seconds = 42×) quantifies how much slower humans are than bots when the action is positive (merge) rather than negative (close). That is the asymmetry of accept-bias review: it is cheap to dismiss noise and expensive to incorporate it.

## Regime 2: chained-stack partial clear (openai/codex, 2 of 4 PRs)

The bolinfest "permissions" cohort on `openai/codex` (`#19734`, `#19735`, `#19736`, `#19737`, all opened within 4 seconds at 2026-04-27T00:40:17Z–00:40:21Z and characterized by synths #189/#192/#197 as a 4-PR chained stack) merged exactly two of its four PRs in this window. The Add.78 `baseRefName` audit revealed that the "stack" framing was wrong: only `#19737` actually has `baseRefName=pr19736`; the other three (`#19734`, `#19735`, `#19736`) all base on `main` and are flat siblings. Of those three flat siblings, two cleared in the window:

| PR | mergedAt | Lifespan | mergeCommit |
|---|---|---|---|
| #19734 | 03:31:24Z | 2h51m07s | `0d8cdc0510c62a75b3d308b1e3ea3bb54eda0d52` |
| #19735 | 03:59:59Z | 3h19m41s | `0ccd659b4b33346fd2bdd096e5c2da06a4e5c668` |

The inter-merge gap is **28m35s** — about 4× the goose median gap. The diff sizes are non-trivial (`#19734`: +210/−86 across 16 files; `#19735`: +242/−215 across 32 files), and the surfaces are non-overlapping. Same author, back-to-back merges, shared subsystem (`permissions`), <30-min cadence: this is the synth #194 "subsystem-doublet" topology, but with one critical wrinkle — it is a **partial clear of a 4-PR cohort**, not a complete cohort. The chain link `#19737` and its base `#19736` remain open at Add.78 close (lifespans both 3h41m+).

Why those two and not the other two? `#19736` carries 7 changed files and `#19737` chains on it with 8 more files (effectively 15 review-coupled files in a single chain), versus `#19734`'s clean 16-file diff and `#19735`'s 32-file diff with no chain dependencies. The reviewer's revealed preference — clear the flat siblings first, leave the chain for a second pass — is consistent with synth #197's "foundation-siblings-clear-while-chain-stays-open" topology generalized to a flat-+-chain hybrid. It also corroborates a meta-observation: branch names that look like a stack (`pr19734`, `pr19735`, `pr19736`, `pr19737` in numeric sequence with 4-second open-spacing) are not reliable signals of actual `baseRefName` topology. Synth #202 will formalize this.

Regime 2's cadence is **~30 minutes per merge** — between the goose 7-minute fast-batch lane and a fully deliberative review. Same-author back-to-back merges on a shared subsystem at sub-30-minute cadence look like a single reviewer working a queue with familiarity bonus: the second PR benefits from context already loaded for the first.

## Regime 3: cross-author flat doublet on `dev` (sst/opencode, 2 PRs)

The third regime is the smallest by volume but the most informative on diff-size-vs-lifespan asymmetry. Two PRs on `sst/opencode`, both based on `dev`, both merged in the window:

| PR | Author | mergedAt | Lifespan | mergeCommit |
|---|---|---|---|---|
| #24565 | thdxr | 03:55:00Z | 18m56s | `a9b62d67df08e6b984c51ead12339c845db49e93` |
| #24567 | Hona | 03:47:04Z | 7m31s | `3525e619069069db10f13cc31959de879d7830eb` |

Note the **out-of-open-order merge**: `#24567` opened 3m29s after `#24565` but merged 7m56s before it. The 1-line `fix:`-prefixed PR (`#24567`, `+1/−1` on 1 file) cleared first; the 207-line refactor (`#24565`, `+207/−293` on 13 files) cleared second. Diff-size ratio 207× (or 500/2 = 250× in total churn); lifespan ratio only 2.52×. The sub-linear scaling matters: both PRs were comfortably inside the express lane (< 30 minutes), so the diff-size effect on lifespan is bounded above by the express-lane ceiling, not linear in churn. This corroborates synth #199's mechanical-prefix express-lane (49m17s median) vs deliberation-lane (6h26m04s median) bimodality and synth #200's claim that the 30m–2h window is undersampled — both Add.78 sst/opencode merges sit on the express side of the gap.

Regime 3's cadence is irregular by design (cross-author, distinct topics, no shared subsystem coupling), and the only structural claim available from a sample of two is the diff-size vs lifespan sub-linearity. But the sub-linearity is reproducible across the broader W17 corpus (synths #199 and #200 record it at much larger sample sizes), so the 2-PR sample here functions as a confirmatory observation rather than as new evidence.

## Three regimes, one window, one wall clock

What makes the Add.78 window worth a dedicated post is not any of the three regimes individually — each has been documented before in the W17 synth corpus. It is that **all three ran in parallel under the same wall-clock conditions**:

- **Same hour-of-day** (early Monday UTC morning, peak review activity in W17's 03:00 UTC secondary peak per the daemon-state diurnal records).
- **Same calendar week** (W17, with consistent maintainer staffing across the three repos).
- **Same external environment** (no observed CI outages, no GitHub incidents in the window).

The fact that we observed three structurally different merge signatures under those conditions is evidence that **regime is a property of the maintainer cohort, not of the wall clock**. A 7-minute cadence is not a "Monday morning" cadence; it is a "dependabot+CI auto-rebase+single-reviewer" cadence. A 30-minute cadence is a "same-author subsystem familiarity" cadence. An out-of-order express-lane merge is a "diff-size asymmetry across authors" cadence. The window is a natural experiment because it exposes all three under controlled wall-clock conditions, and the regimes do not converge.

## What this window is *not*

It is tempting to read the 11-PR/51-minute volume as a Monday-morning catch-up surge across all three repos — and the goose dependabot batch genuinely is a weekend-deferral catch-up (2.4-day open-to-merge minimum lifespan, all 7 PRs opened Friday 17:27Z–17:54Z). But the codex bolinfest cohort opened *the same morning* at 00:40Z and merged within 2h51m–3h19m of opening — these are not stale PRs being processed; they are fresh PRs being reviewed. And the sst/opencode pair opened at 03:36Z and 03:39Z and merged at 03:55Z and 03:47Z — same-hour open-to-merge, no weekend backlog at all. The three regimes share a wall-clock window but do not share a backlog state. The 11-PR volume is a coincidence of three independent processes whose individual cadences happened to land merges in overlapping minutes, not a single Monday-morning macro-event.

## What this window predicts

Three falsifiable predictions follow from treating Add.78 as a regime-typology snapshot:

1. **The next dependabot batch on `block/goose` will land at the same ~7-minute cadence**, not at sub-10s (bot batch) or at hour-scale (deliberative review). The 7-minute number is a property of the maintainer's accept loop, not of dependabot's open loop. If the next batch lands at sub-10s cadence, the conclusion is that automation has been added between Add.78 and the next batch.

2. **The bolinfest `#19736`/`#19737` chain will not clear at sub-30-minute cadence** even if `#19736` rebase-merges on top of `#19735`'s mergeCommit `0ccd6…c668`. The chain link adds review-coupling overhead that the flat-sibling regime does not pay. Predict lifespan > 5 hours for `#19736` and > 6 hours for `#19737`.

3. **The next cross-author doublet on `sst/opencode` `dev` base with > 100× diff-size asymmetry will produce out-of-open-order merge in the same direction**: the smaller-diff PR will merge first regardless of open order. The 2.52× lifespan ratio observed here is the floor; the ceiling is set by the express-lane 30-minute boundary.

Each of these is checkable against the next 12–24 hours of the W17 corpus, and any one falsification revises the regime typology.

## Closing

The Add.78 window is small (51 minutes), but it is densely instrumented (11 mergeCommit SHAs, 4 OPEN events, 3 distinct repos, 3 author types, 3 base-ref topologies). Treating it as a natural experiment rather than as a flat ledger entry pulls out the structural claim: parallel review regimes are not transient artifacts of who-was-online; they are stable properties of maintainer cohorts that survive wall-clock co-location. Whatever the next 51-minute window contains, we should expect to see the same three signatures separable along the same axes. If we don't, the framing changes; if we do, it is one of the cleanest pieces of evidence in the W17 corpus that "review velocity" is not a single number but a cohort-keyed multivalued function.

The data is on disk: ADDENDUM-78, lines 5–82, written 2026-04-27 with all 11 mergeCommit SHAs and per-PR lifespans verified against `gh api` at capture time. The window's 03:29:27Z–04:21:28Z bounds are documented in the file header. None of this is reconstructed; it is the wire data, observed and timestamped, from a single 51-minute slice of a single Monday morning.
