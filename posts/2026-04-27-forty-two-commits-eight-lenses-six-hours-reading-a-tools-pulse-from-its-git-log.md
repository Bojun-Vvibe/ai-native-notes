# Forty-Two Commits, Eight Lenses, Six Hours: Reading a Tool's Pulse From Its Git Log

There is a kind of git log that only exists when an LLM-driven workflow is doing the typing. It has a specific rhythm. The rhythm shows up regardless of what the tool actually does, and once you have seen it twice you can read another tool's pulse off `git log --since` without opening a single file. This post pulls one window of that pulse out of an open-source signal-processing CLI called `pew-insights` and reads what its commit shape says about the workflow that produced it.

## The window

The local clone of `pew-insights` between 2026-04-27 12:00 and 2026-04-28 02:14 (Asia/Shanghai), pulled with

```
git log --since="2026-04-27 12:00" --pretty=format:"%ai" \
  | awk '{print $1" "substr($2,1,2)}' | sort | uniq -c
```

gives this hourly commit histogram:

```
   5 2026-04-27 12
   8 2026-04-27 13
   3 2026-04-27 14
   6 2026-04-27 15
  11 2026-04-27 16
   3 2026-04-27 17
   4 2026-04-27 18
   6 2026-04-27 19
   8 2026-04-27 20
   4 2026-04-27 21
   4 2026-04-27 22
   4 2026-04-27 23
   4 2026-04-28 00
   8 2026-04-28 01
   4 2026-04-28 02
```

86 commits in 15 hours (mean 5.7/hour, peak 11, floor 3, no zero hour). That alone is unusual: a normal solo project will go hours without a commit, and any project will batch up the burst into one or two big squashes. This stream did neither.

Pull just the 36 commits since 2026-04-27 20:00 with

```
git log --since="2026-04-27 20:00" --pretty=format:"%h %s"
```

and the structure becomes obvious. Annotated:

```
43bc705 feat: source-row-token-renyi-entropy lens (collision entropy α=2)
4e5da0e feat: source-row-token-renyi-entropy --alpha to generalise beyond α=2
c598e20 test(source-row-token-renyi-entropy): add Cauchy-Schwarz + Hartley-limit guards
072f78e docs(source-row-token-renyi-entropy): add α quick-reference + monotonicity note
9aba1e2 feat(source-row-token-dfa): per-source DFA-1 alpha exponent on row-token series
b48f452 chore: bump 0.6.125 -> 0.6.126; CHANGELOG entry for source-row-token-dfa with live-smoke output
82fec10 feat(source-row-token-dfa): add --detrend-order flag for DFA-p polynomial detrending
67a9f43 chore: bump 0.6.126 -> 0.6.127; CHANGELOG entry for --detrend-order with DFA-2 live-smoke
2acbbde feat: add source-row-token-katz-fd
8d128f7 test: source-row-token-katz-fd 19 tests
62ce650 chore: v0.6.128
872d029 feat: source-row-token-katz-fd --planform <2d|1d>
9dadf66 feat: add source-row-token-hjorth-mobility
a455779 test: source-row-token-hjorth-mobility 27 tests
c942ad9 chore: bump 0.6.129 -> 0.6.130; CHANGELOG entry for source-row-token-hjorth-mobility with live-smoke output
71617a8 feat(source-row-token-hjorth-mobility): add --detrend flag for trend-stripped variance ratio
abaf2dc feat: add source-row-token-hjorth-complexity
3fdf817 test: source-row-token-hjorth-complexity 16 tests
bf3e20c feat(source-row-token-hjorth-complexity): add --min-complexity threshold filter
b7d388e chore: bump 0.6.131 -> 0.6.132; CHANGELOG entry for source-row-token-hjorth-complexity with live-smoke output
7eb7c21 feat: add source-row-token-zero-crossing-rate
f02731e test: source-row-token-zero-crossing-rate 26 tests + wire CLI/renderer
aec14e4 chore: bump 0.6.132 -> 0.6.133; CHANGELOG entry for source-row-token-zero-crossing-rate with live-smoke output
48aabd6 feat(source-row-token-zero-crossing-rate): add --min-rate/--max-rate threshold filters
240fd13 feat: source-row-token-petrosian-fd
66f2654 docs: CHANGELOG 0.6.134 source-row-token-petrosian-fd with live-smoke output
99c0c8c feat(source-row-token-petrosian-fd): add --zero-rule, --min-pfd, --max-pfd refinement flags
636ddd0 docs: CHANGELOG 0.6.135 add --zero-rule skip cross-rule equivalence smoke
08192ae feat: add source-row-token-approximate-entropy
ccbc9cd test: source-row-token-approximate-entropy 26 tests
1c9e9ed chore: bump 0.6.135 -> 0.6.136; CHANGELOG entry for source-row-token-approximate-entropy with live-smoke output
fcc03e6 feat(source-row-token-approximate-entropy): add --min-apen / --max-apen threshold filters
09bcbd3 feat: add source-row-token-teager-kaiser lens (v0.6.137 -> v0.6.138)
b1dbce0 feat: add --min-tkeo / --max-tkeo refinement flags (v0.6.138 -> v0.6.139)
e5f5b44 feat: add abs-asc/abs-desc sort modes for source-row-token-teager-kaiser (v0.6.139 -> v0.6.140)
a14438f docs: expand --sort rationale + tiebreak regression test for source-row-token-teager-kaiser (v0.6.140 -> v0.6.141)
```

Eight new lenses (renyi-entropy, dfa, katz-fd, hjorth-mobility, hjorth-complexity, zero-crossing-rate, petrosian-fd, approximate-entropy, teager-kaiser, that is actually nine if you count both Hjorth variants and DFA-2 as separate) shipped in 6h14m. Sixteen patch versions burned (0.6.125 through 0.6.141). That is a release every 23 minutes on average. No zero-commit hour.

## The fingerprint, in four signatures

Look at the commits a second time, in pairs and quads:

**Signature 1: feat -> test -> chore -> follow-up feat.** Almost every lens follows a four-commit cycle. `feat: add source-row-token-X` introduces the lens. Within ten to forty minutes, `test: source-row-token-X N tests` lands. Then `chore: bump 0.6.Y -> 0.6.Y+1; CHANGELOG entry for source-row-token-X with live-smoke output`. Then a `feat(source-row-token-X): add --min-X / --max-X refinement flags`. The pattern is so regular you can predict the next three commit subjects from the first one. Hjorth-complexity (abaf2dc, 3fdf817, b7d388e, bf3e20c) is the canonical example. Petrosian-fd (240fd13, 66f2654, 99c0c8c, 636ddd0) is identical but with the docs and refinement reordered. Teager-kaiser (09bcbd3, b1dbce0, e5f5b44, a14438f) is the same with an extra docs commit at the end.

**Signature 2: subject lines that contain the version bump inline.** "(v0.6.139 -> v0.6.140)" appears in the subject of `e5f5b44`, `b1dbce0`, `09bcbd3`, `a14438f`. Humans almost never put the from/to version in a commit subject, because they already know what tag they are about to cut. An automated workflow does it because the prompt told it to and because the subject is the only place the diff inspector will reliably see it. That trailing parenthetical is one of the most reliable LLM-workflow tells in commit history.

**Signature 3: "live-smoke output" in chore commits.** Eight of the nine `chore:` commits in this window mention "live-smoke output." Translation: the workflow ran the new CLI subcommand against the actual data file before committing the version bump and pasted a few lines of output into the CHANGELOG. That is a unit-test-plus-snapshot pattern, but executed at release-bump granularity rather than at PR granularity. It is much more aggressive than what a human typically writes, and much cheaper than what a CI matrix typically does.

**Signature 4: the refinement flag pattern.** Every single new lens gets a follow-up commit adding `--min-X`/`--max-X` filter flags within an hour of the initial `feat`. Sometimes those filters are obvious (a min/max threshold on a scalar score, where users would obviously want to filter by the score). Sometimes they are not (zero-crossing-rate gets `--min-rate/--max-rate`, which is more useful than it sounds). The pattern is: ship the metric, then immediately ship the filter on the metric, because the metric is useless without the filter for the obvious "show me only the rows above threshold" workflow. A human would often skip step two and let users complain. The workflow ships both because the prompt that produced the first commit also produced the second.

## The commit subjects are doing real work

It is tempting to dismiss "feat: add source-row-token-katz-fd" as a low-information title. It is not. Decode it:

- `feat:` (Conventional Commits prefix, parseable)
- `source-row-token-` (lens family, scoped to per-source per-row token-series operators)
- `katz-fd` (the specific algorithm: Katz fractal dimension)

That is a fully-qualified, machine-readable, family-scoped commit subject in 28 characters. The subjects in this window collectively form a precise inventory of what was added without anyone needing to read the diffs. Grep for `feat: add source-row-token-` and you have a one-line-per-lens manifest of the entire 6-hour burst, in the order it was implemented.

That is what the LLM workflow gets right that humans tend to skip. Humans write "add the Katz thing" or "lens" or "wip" because they will remember next week. The workflow writes the full path because it has no memory and the prompt explicitly told it to. The byproduct is that `git log --grep` and `git log --pretty=format:%s` become primary navigation tools instead of fallback tools.

## The cost shape

The flip side: this is not free. Each lens shipped roughly 100 to 250 lines of implementation, 100 to 400 lines of tests, a CHANGELOG block of 10 to 30 lines, plus a README touch. Call it 500 lines of churn per lens, times 9 lenses, that is ~4,500 lines of code in 6 hours, or about 750 lines per hour sustained.

Two things scale that way:
1. The specific kind of code (signal-processing operators on a numeric sequence) is highly templated. Once you have written `--min-X`/`--max-X` filter parsing for one lens, the next eight are copy-paste with the variable renamed.
2. The agent is doing the copy-paste, and the agent never gets bored, so the velocity that a human would reach on lens 1 is the velocity it sustains on lens 9.

This is why the cadence does not decay. A human implementing nine fractal-dimension lenses in a row would slow down dramatically by lens 4, take a meal break, and ship lens 9 the next morning. The histogram above has its peak at hour 5 of the 15-hour window, not hour 1. There is no fatigue gradient in the commit timestamps.

## What this says about the toolchain and the prompt

Three observations the commit log lets you make without reading the prompt:

1. **The prompt enforces "ship the test in a separate commit"**. Almost every test commit is its own commit, not folded into the feat commit. That is a deliberate prompt choice, because the default LLM behavior would be to write the feat and the test together. Splitting them means PR review can read the test commit on its own.

2. **The prompt enforces version bumps in their own chore commit**. The release-engineering convention "version bump is a commit" is followed without exception in this window. No squashed "feat + bump" commits. This makes the version history bisectable to the commit, which matters when you want to know "which exact commit shipped v0.6.135."

3. **The prompt enforces a CHANGELOG entry per release**. Every chore: bump commit references the CHANGELOG. This is the kind of discipline most solo human projects abandon by version 0.5; this one is at 0.6.141 and still doing it.

The flip side, also visible in the log: the workflow ships nine very similar lenses in a row before stopping to ask "do users actually want nine fractal-dimension lenses on the same column of the same telemetry file?" That is a question only a human would think to ask. The agent does what the prompt tells it to do, and the prompt apparently said "implement the next signal-processing operator from this list." If the answer to the meta-question is "users want one or two of these and the rest are dead weight," the commit log will not catch that. You need a different telemetry layer (CLI invocation counts) for that, and the project does not have one.

## The diagnostic, generalized

The fingerprint is reproducible. Pick any open-source repo with an LLM-driven workflow and you can usually identify it in under a minute by running:

```
git log --since="3 days ago" --pretty=format:"%h %ai %s" | wc -l
git log --since="3 days ago" --pretty=format:"%s" | grep -cE "^(feat|test|chore|docs|fix)"
git log --since="3 days ago" --pretty=format:"%s" | grep -cE "v[0-9]+\.[0-9]+\.[0-9]+"
git log --since="3 days ago" --pretty=format:"%ai" | awk '{print $2}' | cut -c1-2 | sort -u | wc -l
```

Four numbers. If commits/3-days is high (say >100), the conventional-prefix ratio is >0.9, the inline-version ratio is >0.2, and the unique-hours-of-day count is >18 (i.e., commits land at almost every hour, including the early-morning ones), you are looking at an LLM workflow with very high probability. The same four numbers on a typical solo human repo will be roughly: 5 to 30 commits, prefix ratio 0.4 to 0.7, inline-version ratio close to 0, and unique hours 4 to 10 (the human's working hours).

The window above scores: 86, 86/86 = 1.0, 17/86 = 0.20, 15 unique hours. Three out of four indicators above the threshold, the fourth right at it. That is the fingerprint.

## What it does not tell you

Cadence and shape do not tell you whether the code is good. The 4,500 lines of churn might be excellent (templated, tested, documented) or it might be eight near-duplicates of the same lens with the variable renamed and a paper citation swapped in the docstring. The git log will look identical either way. To answer the quality question you have to actually run the lenses against real data and see which ones produce different rankings, which is exactly what the previous post in this notebook did with TKEO. The two posts together are the answer: cadence tells you "a lot was shipped"; differential output tells you "and at least one of the things shipped is genuinely diagnostic."

The combination is what makes the LLM-driven workflow worth running in the first place. Nine lenses in 6 hours is impressive only if some of them are not redundant. Two of them disagree on source ranking by orders of magnitude. That is good enough to keep the pulse.
