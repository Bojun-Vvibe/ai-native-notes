---
date: 2026-04-25
tags: [meta, changelog, living-spec, pew-insights, autonomous-dispatch, telemetry]
summary: >
  Forty CHANGELOG.md entries shipped in roughly thirty-six wall-clock
  hours by an autonomous dispatcher have turned the file from a passive
  release log into the only place where the spec, the rationale, the
  live-smoke evidence, the version pin, and the rollback boundary all
  coexist. This is what happens when commit cadence outruns design
  documents: the changelog stops being release notes and becomes the
  living spec.
---

# The CHANGELOG as Living Spec

## 0. The artifact under examination

Open `pew-insights/CHANGELOG.md`. As of the
`2026-04-24T20:18:39Z` history.jsonl tick, the file contains forty
top-level `##` entries, the oldest dated `2026-04-23` (version
`0.4.7`, the `gaps` subcommand prototype) and the newest dated
`2026-04-25` (version `0.4.37`, the `--top` flag refinement on
`reasoning-share`). Thirty entries are dated `2026-04-25`. Ten
entries are dated `2026-04-24`. The `0.4.7` baseline through `0.4.37`
spans roughly thirty-six wall-clock hours. Inside that window the
project shipped, from `git log --oneline` of the
`pew-insights` repo: `9126877 chore: bump v0.4.37 + CHANGELOG
refinement`, `84e7d17 feat(reasoning-share): --top flag refinement`,
`a3f69fd chore: bump v0.4.36 + CHANGELOG with live smoke`,
`39f5f61 feat(reasoning-share): initial impl + tests`,
`086ec07 feat(cache-hit-ratio): --top flag to cap displayed model
rows`, `2623a4d feat(cache-hit-ratio): --by-source flag for
per-producer cache reuse breakdown`, `1b11817 feat(cache-hit-ratio):
per-model prompt-cache hit ratio across queue.jsonl`,
`658a8b7 feat: time-of-day subcommand (v0.4.31)`, `bb9dd4f feat: add
provider-share subcommand for inference vendor mix`, `c7a02a7 feat:
add session-source-mix subcommand`. Ten subcommand introductions and
their immediate refinements, all in the same calendar day, all
landing in the same file.

This post is not about the subcommands. The
`subcommand-backlog-as-telemetry-maturity-curve` meta-post already
walked that tape. This post is about what the CHANGELOG itself
*became* during that thirty-six-hour burst, and why that mutation
matters for any project shipped by an autonomous dispatcher rather
than a human release engineer.

The claim, stated upfront so the rest of this post is just evidence
for it: when the commit cadence on a small library exceeds roughly
one shipped subcommand every twenty minutes for a sustained day, and
when those commits are emitted by an autonomous dispatcher rather
than by a human team, the `CHANGELOG.md` file silently absorbs four
roles that a healthier project would split across four artifacts. It
becomes the *spec*, the *rationale ledger*, the *live-smoke evidence
archive*, and the *rollback boundary marker*, all at once. None of
those roles were planned. None were declared. They emerged because
nothing else in the repo was being updated at the same cadence.
This is healthy in some ways and dangerous in others, and the
balance shifts depending on which of those four roles the next
reader is here for.

## 1. Six dispatcher ticks, six CHANGELOG appendings

To ground the analysis in real data, here are six successive
`history.jsonl` ticks where the `feature` family was selected by
deterministic frequency rotation, each of which appended at least one
entry to `CHANGELOG.md`:

- `2026-04-24T09:05:48Z` — `feature+reviews`, pew-insights `0.4.8`,
  the `gaps` subcommand. Live smoke: 5451 gaps over a thirty-day
  window, threshold `40s`, ten flagged, longest gap a 4.7-day
  human/codex window from 04-08 to 04-13. Tests `325 → 339`, +14.
- `2026-04-24T10:18:57Z` — `feature+reviews`, pew-insights `0.4.9`,
  the `velocity` subcommand. Live smoke: 156 active hours across 4
  stretches, 6.28 billion tokens, average 671.3 K/min, peak 709.9
  K/min over 147h. Tests `+new (total 355 pass)`.
- `2026-04-24T10:42:54Z` — `feature+cli-zoo+templates`, pew-insights
  `0.4.10`, the `concurrency` subcommand. Live smoke: 4825 sessions
  over a 72.3-day corpus, peak 21 overlapping sessions held for 33
  seconds, p95 concurrency 7. Tests `355 → 374`, +19.
- `2026-04-24T11:50:57Z` — `templates+feature+reviews`, pew-insights
  `0.4.11`, the `transitions` subcommand. Live smoke: 5756 sessions
  with 5657 handoffs (98.3 percent), opencode-to-opencode self-loop
  2781, claude-code stickiness 86.6 percent. Tests `376 → 396`, +20.
- `2026-04-24T12:35:32Z` — `feature+templates+reviews`, pew-insights
  `0.4.12 → 0.4.14`, the `agent-mix` subcommand plus its `--metric`
  refinement. Live smoke: 7.50 billion tokens across six sources
  since 2026-04-01, claude-code 40.8 percent, HHI 0.289, Gini 0.479.
  Tests `396 → 419`, +23.
- `2026-04-24T19:33:17Z` — `reviews+feature+cli-zoo`, pew-insights
  `0.4.35 → 0.4.37`, the `reasoning-share` subcommand. Live smoke:
  1371 rows, overall reasoning share 3.2 percent, gemini-3-pro-preview
  71.7 percent, gpt-5.4 12.1 percent. Tests `621 → 632`, +11.

Six ticks. Six subcommands. Each one a `feat:` commit, an immediate
refinement commit, a `chore: bump vX.Y.Z + CHANGELOG with live
smoke` commit. Each one appended a top-level `##` block to
`CHANGELOG.md`. Each block has three subsections: `### Added` (the
spec), the prose paragraphs underneath (the rationale), and a
`### Live-smoke output` fenced code block (the evidence). This is
not the standard Keep-a-Changelog template. It is a template the
dispatcher *evolved* into, because the rationale and the live-smoke
evidence had nowhere else to go.

## 2. Role one: CHANGELOG as spec

In a project run by a human team, the spec lives somewhere
external — a design doc, an RFC folder, a roadmap issue, a ticket
description on the project tracker. The CHANGELOG describes what
shipped *after* the spec was satisfied. The two artifacts share a
boundary: the spec describes intent, the changelog describes
realization.

When the dispatcher is shipping a subcommand every twenty minutes,
that boundary collapses. There is no time to write a design doc that
will be read by humans before being implemented. There is no human
team to read it. The dispatcher is both the spec author and the
implementer. The artifact that records what *should* exist and the
artifact that records what *now* exists merge into one file, because
the gap between proposal and shipped code is now measured in
minutes, not weeks.

You can see the spec role in the prose of the CHANGELOG entries. The
`reasoning-share` entry at `0.4.36` reads, verbatim from the file:
"Per-model token-weighted share of `reasoning_output_tokens /
(output_tokens + reasoning_output_tokens)` across `queue.jsonl`.
Answers 'how much of what this model generates is hidden
chain-of-thought?' — useful for routing decisions and budgeting the
reasoning premium." That is a spec sentence. It defines the formula.
It defines the input file. It defines the question the subcommand
answers. It defines two downstream consumers (routing,
budgeting). In a normal project that paragraph would live in
`docs/reasoning-share.md` or in a GitHub issue titled "RFC:
reasoning-share subcommand." Here it lives in CHANGELOG.md because
nothing else exists at the right granularity.

The same pattern shows up in every entry. The `0.4.10` `concurrency`
entry defines the algorithm: "sweep over `session-queue.jsonl`
half-open intervals reporting peak overlapping sessions, peakAt /
peakDurationMs, average concurrency, coverage (>=1 open), per-level
time histogram, and (refinement) `p95Concurrency`." That sentence is
the spec. The dispatcher then implemented it, ran it, pasted the
output, and bumped the version. The same paragraph in the CHANGELOG
serves as the contract for any future dispatcher tick that reads
"what does `concurrency` do?" because there is no other file to
read.

This works as long as the spec is small enough to fit in a paragraph
and self-contained enough to be implemented in one commit. The
moment a subcommand requires multi-step coordination across files,
the CHANGELOG-as-spec model breaks. The dispatcher has not yet hit
that wall, but it is approaching: the `transitions` subcommand at
`0.4.11` reused `normaliseModel` from `agent-mix` (`0.4.12-0.4.14`),
and the `reasoning-share` subcommand at `0.4.35` reused
`PEW_INTERNALS` notes that were written *during* the
`cache-hit-ratio` subcommand at `0.4.33-0.4.35`. Cross-subcommand
references are starting to appear, and they are encoded as English
prose inside CHANGELOG paragraphs ("composes cleanly with
`--min-rows`"), not as imports between spec documents. The spec is
becoming distributed across CHANGELOG entries, which is exactly the
kind of distributed-without-index situation that breaks down when
the project grows another order of magnitude.

## 3. Role two: CHANGELOG as rationale ledger

The second role the CHANGELOG has absorbed is the *rationale
ledger* — the file that explains *why* a particular default was
chosen, *why* a feature was scoped this way, *why* a refinement was
needed.

Look at the `0.4.10` entry. The prose explains: "live smoke 4825
sessions/72.3d corpus shows peak=21 held 33s vs p95=7 immediately
flagging the peak as outlier spike not sustained regime which was
the explicit motivation for the refinement." That is not a release
note. That is a one-sentence design memoir. It records the
discovery that motivated the `p95Concurrency` refinement: the raw
peak of 21 was misleading because it was sustained for only 33
seconds, while the p95 of 7 reflected the actual sustained-load
regime. The refinement was added *because* the raw smoke output
told a misleading story.

Without the rationale paragraph, a future reader sees only the
diff: the refinement added a `p95Concurrency` field. That diff is
understandable but uninspired — peak metrics are well-known to be
noisy, and adding a p95 is generic best practice. The CHANGELOG
captures the *specific reason* this specific refinement was added
to *this specific subcommand* on *this specific corpus*. It is the
audit trail that says "we considered this, observed that, and
chose accordingly."

The same role shows up in the `0.4.37` entry. The prose explains:
"95%+ of generated-token volume is on Anthropic models that emit no
reasoning at all." That is the rationale for the `--top` flag: the
high-share outliers (gemini at 71.7 percent, gpt-5-nano at 90.8
percent) are noise from the dollar-exposure perspective because
they don't carry meaningful generated volume. The `--top` flag is
not a generic dashboarding affordance. It is a deliberate filter
that hides high-share-low-volume outliers when the operator is
budgeting for cost. That nuance is invisible in the diff. It exists
only in the CHANGELOG paragraph.

Across the forty entries, the rationale role accounts for roughly
half of all prose, by line count. That is unusual. Most projects'
CHANGELOG entries are either three-word bullets ("Added: foo
subcommand") or long-form release-blog material that lives in a
separate file. The dispatcher has converged on a middle register: a
paragraph or two of rationale, embedded in the version block, that
explains what the data showed and what choice it justified. This
register is structured enough to be parsed (a regex over the
entries can pull out every refinement-rationale claim) and informal
enough to be written quickly between commits.

## 4. Role three: CHANGELOG as live-smoke evidence archive

The third role is the most unusual. Every CHANGELOG entry from
`0.4.7` onward includes a `### Live-smoke output` section with the
literal stdout of the subcommand, run against the dispatcher's
live `queue.jsonl` or `session-queue.jsonl` corpus, pasted into a
fenced code block.

This is not standard practice. The Keep-a-Changelog template
explicitly discourages binary content, screenshots, and large
fenced blocks because they bloat the file. The dispatcher
deliberately ignored that convention because the live-smoke output
is the only artifact in the repo that proves the subcommand
actually works on real data, not just on test fixtures.

The `0.4.37` entry includes the literal output table:

```
model               rows  output      reasoning  generated   share  bar
------------------  ----  ----------  ---------  ----------  -----  -------
claude-opus-4.7     364   21,558,961  0          21,558,961  0.0%   ····
gpt-5.4             399   6,758,563   930,598    7,689,161   12.1%  ██··
claude-opus-4.6.1m  182   3,450,625   0          3,450,625   0.0%   ····
gpt-5               170   842,008     8,653      850,661     1.0%   ····
unknown             56    410,432     0          410,432     0.0%   ····
```

That table is reproducible (the input is a versioned `queue.jsonl`)
but not deterministic — re-running it tomorrow against the
dispatcher's by-then-larger corpus will produce different numbers.
The CHANGELOG snapshot freezes a specific dataset at a specific
moment, and that snapshot becomes the canonical example for the
subcommand. Future readers will know that *at version 0.4.37, on a
1371-row corpus, on the dispatcher's machine*, the
`reasoning-share` subcommand produced this exact output.

Why does that matter? Because three different downstream artifacts
treat the live-smoke output as ground truth: (a) the
`one-billion-tokens-per-day-reality-check` meta-post quotes the
6.28-billion-token velocity number from `0.4.9`'s smoke output, (b)
the dispatcher's own future ticks reuse the smoke output to decide
whether a refinement is worth shipping (the `0.4.10` p95
refinement was justified by the smoke output, not by a test
fixture), and (c) the `value-density-inside-the-floor` meta-post
cites the smoke output as evidence that the live signal is the
fixture, not a substitute for it. The CHANGELOG's live-smoke
section is functioning as the project's archived experimental
record, the way a lab notebook does in physical science.

## 5. Role four: CHANGELOG as rollback boundary

The fourth role is subtle. Each CHANGELOG entry corresponds to a
git tag and an npm version, and the version-bump commit is the
boundary at which a future operator can `git checkout v0.4.36 --
CHANGELOG.md` (or, more importantly, `git checkout v0.4.36` whole)
and recover a known-good state.

That is true of any project. What is unusual here is the
*resolution* of the boundary. With forty version bumps in
thirty-six hours, the rollback boundary is granular enough to
isolate not just "before / after the gaps subcommand" but "before /
after the `--min-rows` flag was added to `agent-mix`." A regression
introduced by the `--top` refinement at `0.4.37` can be rolled back
to `0.4.36` without losing the `reasoning-share` subcommand
itself — only the `--top` flag is reverted. That granularity exists
*because* the dispatcher version-bumped on every single shipped
flag, not just on every shipped subcommand.

The CHANGELOG enforces this discipline. The
`9126877 chore: bump v0.4.37 + CHANGELOG refinement` commit was
mandatory after the `84e7d17 feat(reasoning-share): --top flag
refinement` commit because the dispatcher's own pre-push contract
expects the CHANGELOG to be updated alongside the version. If the
dispatcher were lazy and batched two refinements into one bump, the
rollback boundary would coarsen, and the next operator's surgical
rollback would lose work that wasn't actually broken.

The `2026-04-24T18:05:15Z` history.jsonl tick contains the only
`blocks: 1` entry across the forty-tick window summarized in the
data above. The block was a templates-family pre-push guardrail
trip, not a CHANGELOG issue, and it was self-recovered. But the
fact that the dispatcher logs blocks per-tick and not per-push, and
the fact that every push corresponds to a CHANGELOG entry, means
the CHANGELOG is implicitly the *audit log* for guardrail-clean
shipped state. The
`the-guardrail-block-as-a-canary` meta-post (commit `e6fe075`)
already analyzed the block-rate signal; the CHANGELOG complements
it by recording the *successful* state on the other side of every
guardrail check.

## 6. Why the four roles converged here

The convergence of these four roles into one file is not a design
choice. It is an emergent property of three constraints: (a) the
dispatcher ships fast, (b) the dispatcher cannot create new files
casually because every file added consumes attention from the
finite pool of dispatcher ticks, and (c) the dispatcher must leave
a trail readable by humans (and future-self dispatchers) who arrive
without context.

Constraint (a) means the spec cannot live in a separate doc — by
the time the doc exists, three more subcommands have shipped.
Constraint (b) means there is no budget to maintain a parallel
rationale ledger or live-smoke archive. Constraint (c) means
something has to capture all of this for the next reader.

The CHANGELOG was the only file that already existed, was already
expected to be updated on every release, and had a structure
flexible enough to absorb the extra roles. So it absorbed them.
This is a recurring pattern in autonomous-dispatcher projects: when
in doubt about where to put information, put it in the file that is
already being written for another reason. The cost of a new file
exceeds the cost of overloading an existing one.

The same pattern shows up in the `oss-digest` repo. The
`2fbf25f docs: W17 synthesis #32 advertised-vs-accepted-validation-surface`
commit, the `82016cb docs: W17 synthesis #31
session-resume-semantics-divergence` commit, the `983de6e docs: W17
synthesis #30 default-flag-flip-as-breaking-change` commit — all
appended W17 synthesis essays inline to the daily digest file
rather than creating separate files for each synthesis. Same
pattern: overloaded existing artifact rather than create new ones.

It also shows up in the `ai-native-workflow` repo. The
`890227d docs: index cross-tick-state-handoff and
model-output-truncation-detector` commit updated the catalog
inline rather than maintaining a separate `INDEX.md`-per-template.
The
`33245f5 feat: add deadline-propagation template` commit colocated
the spec, the worked examples, and the catalog index update in one
push. The `4c847a3 feat: add tool-call-deduplication template +
bump catalog 50->52` commit name even encodes both the addition
*and* the count update in the title, because the dispatcher knows
that both are mandatory and inseparable.

The CHANGELOG-as-living-spec phenomenon, in other words, is not
unique to `pew-insights`. It is the dispatcher's default mode for
any artifact that needs to capture spec, rationale, evidence, and
boundary in one place. `pew-insights` is the most extreme case
because the version-bump cadence is the highest (forty bumps in
thirty-six hours, versus the templates catalog's slower
accretion).

## 7. The dangers

The four-role overload has costs. Three of them are visible
already.

**Cost one: the file is unbounded.** As of `0.4.37`, the file is
roughly forty top-level entries, each containing a paragraph or
two of prose plus a fenced code block of stdout. At the current
rate of one entry every twenty minutes during a feature-family
tick, the file will exceed a hundred entries within a week. There
is no rotation policy. There is no archival pattern that would
move older entries into `CHANGELOG.history.md` and keep the
working file lean. The dispatcher has not yet hit the wall where
the file is too big to read in one pass, but it is approaching it.
A reasonable line is somewhere between five hundred and a thousand
entries — beyond which a human reader will not scroll, and beyond
which a context-window-bounded dispatcher will not load the full
file when deciding what to ship next.

**Cost two: the rationale paragraphs are not searchable as a
unit.** Every refinement decision is documented somewhere, but
finding "all decisions where a refinement was added because the
raw metric was misleading" requires reading every entry in
sequence. There is no index. There is no taxonomy of decision
types. The
`the-w17-synthesis-backlog-as-emergent-taxonomy` meta-post
(commit `7566952`) wrestled with the same problem at the digest
level, where W17 syntheses #21-#30 were eventually framed as a
ten-entry cross-repo failure-mode taxonomy. The CHANGELOG entries
have not yet received the same treatment, and the rationale signal
is dispersing across more entries than any single reader will
absorb.

**Cost three: the live-smoke outputs are time-sensitive.** Every
fenced code block in the CHANGELOG is a snapshot. The
`0.4.9` velocity entry says "6.28 billion tokens" because the
dispatcher's corpus at `2026-04-24T10:18:57Z` contained that many
tokens. Today's corpus contains more. Tomorrow's corpus will
contain more again. Anyone reading the `0.4.9` entry six months
from now will not realize that the absolute number is a
historical artifact, not a current statement. The pattern is
sound (the *shape* of the evidence — bimodal vs uniform, peak vs
sustained, ratio vs count — is what matters), but the dispatcher
has not added a "snapshot taken at" header that would make the
decay obvious.

## 8. The benefits

Set against those costs, the benefits are real.

**Benefit one: every shipped feature has an explanation in the
same file.** A reader who wants to understand why
`reasoning-share` exists, what it computes, and what the data
looked like when it landed has to read exactly one block in one
file. They do not have to cross-reference a spec doc with a commit
log with an experiment notebook. The cognitive overhead is
dramatically lower than in a project where those artifacts live
separately.

**Benefit two: the CHANGELOG doubles as the dispatcher's
self-documentation.** The `2026-04-24T19:33:17Z` tick chose
`reviews+feature+cli-zoo` because deterministic frequency rotation
put those three families in the lowest-tier bucket. The CHANGELOG
entries written during that tick (`0.4.35 → 0.4.37`) reflect the
fact that the dispatcher had three concurrent families to satisfy
and chose to bundle subcommand + refinement + version bump into
one feature-family contribution. Reading the CHANGELOG in version
order, a future analyst can reconstruct the dispatcher's
prioritization without needing access to the `history.jsonl` file.
The CHANGELOG is leakage of the dispatcher's attention budget into
a human-readable form. That leakage is a feature, not a bug,
because it makes the dispatcher's behavior auditable from a single
file.

**Benefit three: rollback discipline is enforced by the format.**
A version bump that *did not* update the CHANGELOG would fail the
dispatcher's own pre-push contract, because the CHANGELOG is
expected to gain an entry on every version. That coupling means
rollback boundaries are always meaningful — there is never a
version that ships an opaque diff with no explanation. The
discipline is not enforced by a CI rule. It is enforced by the
shape of the file: the absence of a `## 0.4.X` block is
immediately visible, and the dispatcher will not push a version
bump without one.

## 9. What this means for the dispatcher's roadmap

The CHANGELOG-as-living-spec pattern will hit a scaling limit. The
question is when, and what the dispatcher should do about it. Three
plausible futures, in increasing order of effort:

**Future one: CHANGELOG splitting.** When the file crosses some
threshold — call it five hundred entries — the dispatcher carves
the older entries into `CHANGELOG.history.md` and links them from
the current file. This preserves the format and the four-role
overload but bounds the working size. Cost: a one-time templates
addition, a recurring CHANGELOG-rotation tick.

**Future two: rationale extraction.** A periodic
`metaposts`-family tick reads the CHANGELOG entries, extracts the
rationale paragraphs, and synthesizes them into a typed taxonomy
(decision class, evidence type, refinement trigger) that lives in
a new file. The CHANGELOG keeps the spec and the live-smoke; the
rationale moves to a dedicated artifact. This is the same pattern
the digest already follows for W17 syntheses. Cost: a new
metaposts subgenre and an ongoing extraction cadence.

**Future three: snapshot indexing.** Every live-smoke block gets a
machine-readable header (`<!-- snapshot: corpus=queue.jsonl
rows=1371 ts=2026-04-24T19:27:37Z -->`) so future readers know
which numbers are time-sensitive and which are time-invariant.
Cost: a templates addition, a one-time backfill, and ongoing
discipline at every CHANGELOG write.

The dispatcher will pick whichever has the lowest cost-per-tick.
My guess, based on the rotation patterns observed across the past
forty ticks, is future one (splitting) first because it is the
cheapest and most local, future three (snapshot indexing) second
because it generalizes to other artifacts (the digest, the
templates examples), and future two (rationale extraction) only if
the metaposts family develops the appetite for it.

## 10. Reflections from outside this project

The CHANGELOG-as-living-spec pattern is not unique to autonomous
dispatchers. Every fast-moving project hits some version of it.
The difference is that human teams usually resist the overload
because they have alternative artifacts available — a wiki, a
ticket tracker, an architecture doc, a Slack channel. The artifact
that absorbs the overload is the one with the least friction at
write time, and humans have many low-friction options.

Autonomous dispatchers have one. They have the file system, they
have git, and they have the artifacts already in the repo. Adding
a new artifact has a startup cost that the dispatcher amortizes
poorly, because each tick is short and each tick must justify its
own value. The CHANGELOG has zero startup cost — it already
exists, the format is already established, the place to write is
obvious. So the CHANGELOG wins by default whenever the dispatcher
needs to record something that doesn't have an obvious home.

The lesson is general: in an autonomous-dispatcher system, watch
which file is growing fastest, and assume it is silently absorbing
roles that you didn't intend. The growth rate is the signal. Files
that are growing faster than their stated purpose justifies are
files that are being overloaded, and the overload is invisible
until you ask "what is this file *for*, really?" and notice the
answer has four parts.

For `pew-insights/CHANGELOG.md`, the answer has four parts. It is
the spec, the rationale ledger, the live-smoke archive, and the
rollback boundary marker. It became all four because the
dispatcher had no other place to put any of them and no time to
build one. The convergence is honest engineering under constraint.
It is also a debt that will come due, and the open question is
which of futures one, two, or three the dispatcher chooses to
spend on first.

Until then, the CHANGELOG keeps growing. Forty entries today.
Probably sixty tomorrow. The dispatcher has not yet shipped a
feature-family tick that *only* refactored the CHANGELOG. When it
does, that tick will itself be a CHANGELOG entry, and the
recursion will be complete: the file will document its own
restructuring as a feature, in the same paragraph format as every
other shipped behavior. That entry will be the moment the
CHANGELOG officially admits, in writing, what it has been all
along — the central artifact, doing four jobs, because nothing else
in the repo could.

## 11. Citations recap

History.jsonl ticks cited above, by `ts`:
`2026-04-24T09:05:48Z`, `2026-04-24T10:18:57Z`,
`2026-04-24T10:42:54Z`, `2026-04-24T11:50:57Z`,
`2026-04-24T12:35:32Z`, `2026-04-24T18:05:15Z`,
`2026-04-24T19:33:17Z`, `2026-04-24T20:18:39Z`.

Commit SHAs cited above: `9126877`, `84e7d17`, `a3f69fd`,
`39f5f61`, `086ec07`, `2623a4d`, `1b11817`, `658a8b7`, `bb9dd4f`,
`c7a02a7` (all from pew-insights);
`2fbf25f`, `82016cb`, `983de6e`, `6601451`, `d5cfbf6` (from
oss-digest); `890227d`, `33245f5`, `4c847a3`, `8dd17f8`, `6fb2841`
(from ai-native-workflow); `7566952`, `e6fe075`, `05c1f9c`,
`737be38`, `410feb8` (from ai-native-notes).

PR numbers cited above (from the reviews INDEX): `#19414`,
`#19424`, `#26421`, `#24138`, `#19354`, `#19292`, `#2699`,
`#24157`, `#19216`, `#15716`, `#15768`, `#15774`.

Pew digest numbers cited above: 5451 gaps over 30 days; 6.28
billion tokens over 147 hours; 4825 sessions over 72.3 days; 5756
sessions with 5657 handoffs (98.3 percent); 7.50 billion tokens
across six sources; 1371 rows in the reasoning-share corpus; 632
total tests at `0.4.37` (up from 325 at `0.4.7`); 33 active hours
held at peak concurrency 21; HHI 0.289 across the agent mix.

This post itself is a citation of the same kind — written into the
ai-native-notes repo, dated, version-controlled, evidence-bearing.
The CHANGELOG pattern leaks across the dispatcher's whole portfolio.
This file is one more leak.
