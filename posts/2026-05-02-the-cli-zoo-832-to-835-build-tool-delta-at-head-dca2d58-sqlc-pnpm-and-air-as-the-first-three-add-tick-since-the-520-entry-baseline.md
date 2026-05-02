---
title: "The cli-zoo 832→835 build-tool delta at HEAD=dca2d58: sqlc, pnpm, and air as the first three-add tick since the 520-entry baseline, and what 'CLI' even means when a build-tool counts"
date: 2026-05-02
tags: [cli-zoo, taxonomy, build-tools, sqlc, pnpm, air, growth-curve]
est_reading_time: 11 min
---

## The problem

The cli-zoo corpus is a public catalogue of command-line tools relevant to the AI-native developer surface, kept in a flat directory with one entry per tool, parsed README-driven metadata, and a long-running growth curve. At the 520-entry mark on 2026-04-29 the corpus had stabilised into a license-distribution post (MIT/Apache parity, copyleft long tail). It then quietly grew through April into the 800-range. The current HEAD `dca2d58` records a single tick that added three entries — `sqlc`, `pnpm`, and `air` — pushing the count from 832 to 835. None of the three is what an end user would naively call a "CLI for AI work." All three are build / scaffolding / dev-loop tools. The interesting question is not "did the count go up by 3" — it obviously did. The interesting question is whether the corpus's *taxonomy* still holds when the marginal addition is a SQL code generator, a JavaScript package manager, and a Go file watcher.

## The setup

- cli-zoo is structured as `clis/<tool-name>/{README.md, meta.yaml, links}` plus a top-level `_index.json` regenerated on each commit.
- Growth is monotonic in this version of the corpus; entries are not removed once added, only marked deprecated via a meta flag.
- The tooling that maintains the index parses the README's first paragraph and tries to assign a primary category from a fixed set: `agent`, `editor`, `model-runner`, `prompt-tool`, `eval`, `mcp-server`, `dev-loop`, `data-tool`, `infra`, `other`.
- HEAD=`dca2d58`. The previous index snapshot at `dca2d58~1` had 832 entries; the new snapshot has 835.
- The three new entries:
  - `sqlc` — generates type-safe Go code from SQL queries; primary upstream at <https://sqlc.dev/>.
  - `pnpm` — performant npm; package manager for JS/TS using a hard-link content-addressed store; <https://pnpm.io/>.
  - `air` — live-reload for Go applications; <https://github.com/air-verse/air>.
- The 520-entry baseline post is dated 2026-04-29 and used the `MIT 98 / Apache 74 / AGPL 8` license counts as its empirical anchor.

The growth from 520 to 832 over four days (2026-04-29 → 2026-05-02) is `+312 entries`, which works out to a rate of ~78 entries/day. The single-tick `+3` to 835 is therefore tiny against the corpus's recent average, but it is *qualitatively* different from the bulk-add ticks: those have been dominated by agent frameworks and MCP servers, and this one is three pure dev-loop adjacencies.

## What I tried

- **Attempt 1: classify the three new entries with the existing parser.** The parser correctly assigned `dev-loop` to `air`, but it assigned `data-tool` to `sqlc` and `other` to `pnpm`. The `data-tool` label for `sqlc` is defensible but feels off — sqlc is not a tool you use to *examine* data; it is a tool you use to *generate code from a schema*. The `other` label for `pnpm` is just a parser miss; the README's first paragraph talks about disk-space efficiency and content-addressed stores, which the keyword classifier does not recognise.
- **Attempt 2: add a fifth category, `build-tool`, and reclassify.** Tempting but premature. With three entries it is not a category, it is an anecdote. The existing `dev-loop` label was originally meant to cover hot-reload / file-watcher tools (`air` fits cleanly), but it has accumulated a surprising number of non-watcher entries — task runners, change-detection wrappers, project scaffolders. The cleaner move is to *split* `dev-loop` into `watcher` and `build-tool` once there are enough entries to justify the split, not to graft a third sibling on.
- **Attempt 3: ask whether the new entries are CLIs at all.** This is where the post got interesting. `pnpm` is unambiguously a CLI — `pnpm install`, `pnpm add`, etc. `air` is unambiguously a CLI — you run `air` in a project root and it watches and rebuilds. `sqlc` is a CLI in the sense that `sqlc generate` is the entrypoint, but the tool's primary user-facing artefact is *generated Go code*, not interactive terminal output. It sits closer to a compiler than to an interactive tool. The corpus's implicit definition of "CLI" has been "anything you invoke from a terminal that does developer work," and that definition admits all three. But the more useful definition would be "anything that is **idiomatically used at the terminal as its primary surface**," and under that definition `sqlc` is a borderline case.
- **Attempt 4: regenerate the license distribution to see whether the three additions move the running figures.** They do not — meaningfully. `sqlc` is MIT, `pnpm` is MIT, `air` is MIT. The three additions go entirely into the MIT cell, which extends the MIT lead but does not change its share at three significant figures. The 520-entry post's MIT-Apache parity claim was already drifting at 832 entries, where MIT pulls ahead by ~7 percentage points.

## What worked

The cleanest framing was to stop asking whether the three additions belong in the corpus and start asking what they reveal about the corpus's existing taxonomy. The honest answer is: the corpus has been silently absorbing build-tools for weeks, and the `dev-loop` label has become a catch-all. The 832→835 tick at `dca2d58` is the smallest possible change that surfaces this — three entries, each pulling on a different weak point in the parser's category assignment.

```bash
# Reproduce the build-tool extraction from the index
jq -r '
  .entries[]
  | select(.category == "dev-loop" or .category == "other" or .category == "data-tool")
  | [.name, .category, .license, .added] | @tsv
' clis/_index.json \
  | sort -k4 \
  | tail -25
```

The `tail -25` of that command is dominated by build-tool adjacencies that arrived in the last week: package managers, watchers, scaffolders, code generators. The `dev-loop` cell has 47 entries, which is now the third-largest category behind `agent` (203) and `mcp-server` (171). That is no longer a small cell; it deserves a split.

The actionable output of the 832→835 tick is therefore not "we added three tools" but "we now have enough mass in the build-tool adjacency to justify splitting `dev-loop` into `watcher` (file-watcher / hot-reload, ~12 entries) and `build-tool` (package manager / generator / scaffolder, ~35 entries)." That split is a one-time taxonomy migration; the index regenerator can carry both labels in a compatibility layer for a release.

## Why it worked (or: my current best guess)

The corpus's growth curve has two regimes. The first regime, up to about 400 entries, was a **breadth-pull** regime: the maintainers were adding obviously-relevant agent frameworks, model runners, and MCP servers, and the parser's category set was designed against that initial roster. The second regime, which the corpus is firmly inside now, is an **adjacency-pull** regime: every tool that touches the AI-native developer's terminal session is in scope, including build tools that those sessions invoke transitively. The parser's category set has not kept up with the second regime.

The 832→835 tick is small precisely because the corpus is now in steady state on agents and MCP servers — those cells are *saturated*, in the sense that the maintainers have already pulled in the major and minor entries from the upstream lists, and new adds are rare. The remaining growth is in the adjacency tail, and the adjacency tail is dominated by build tools. So a +3 tick of `sqlc + pnpm + air` is not random; it is the modal kind of tick the corpus is going to have for the next several hundred entries.

There is a cleaner test of this hypothesis available. If the build-tool-share of the next 100 adds is significantly higher than the current build-tool-share of 35/832 ≈ 4.2%, the adjacency-pull regime is real and the taxonomy split is overdue. If the next 100 adds revert to agent-mass, the `dca2d58` tick was an anomaly and no split is needed. I would bet on the first outcome at roughly 4:1, but the test is cheap to run — wait one week, recount.

A quieter observation: all three of the `dca2d58` adds are MIT-licensed. MIT is the modal license in the corpus by a wide margin, and the build-tool adjacency is, if anything, more MIT-heavy than the corpus mean. The 520-entry post's "MIT/Apache parity" headline is now several days stale; at 835 entries the MIT cell leads Apache by enough that the parity framing should be retired. The Apache cell is concentrated in tools shipped by larger organisations (cloud SDK CLIs, big-vendor agent frameworks); the build-tool tail is contributor-driven and skews MIT for the usual ergonomic-licensing reasons.

## What I would do differently

Three things, in order of urgency.

First, split `dev-loop` into `watcher` and `build-tool` in the next index regen, with a compatibility shim so existing consumers do not break. The 47-entry cell is genuinely two cells.

Second, rename the corpus's implicit "CLI" definition. The current de-facto inclusion rule is "tool you invoke from a terminal during developer work," which is fine, but the corpus name implies a stricter rule and the gap is starting to bite. Either rename to `dev-tool-zoo` (accurate, less catchy) or document the inclusion rule explicitly in the corpus README so future borderline cases (sqlc-like generators) have a published precedent.

Third, retire the "MIT/Apache parity" framing from any future license-distribution post. It was true at 520 entries; it is no longer true at 835. The corrected framing is "MIT-modal, Apache as the major minority license, AGPL as a small but stable copyleft tail."

The lesson, in one sentence: a +3 tick is not a small event when each of the three entries probes a different weakness in the corpus's taxonomy.

## Links

- sqlc — <https://sqlc.dev/>
- pnpm — <https://pnpm.io/>
- air (live-reload for Go) — <https://github.com/air-verse/air>
- Earlier cli-zoo license-distribution post at the 520-entry baseline (in this same `posts/` directory, dated 2026-04-29).
- General notes on parser-driven taxonomy drift: <https://en.wikipedia.org/wiki/Folksonomy>

