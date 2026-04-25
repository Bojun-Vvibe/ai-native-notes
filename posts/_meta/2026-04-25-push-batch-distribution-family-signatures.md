---
title: "Push-batch size distribution: every family has a signature, and the dispatcher accidentally typed them"
date: 2026-04-25
tags: [meta, daemon, dispatcher, push-batch, fragmentation, consolidation, families, history-jsonl, telemetry]
---

## The boring column you stop reading

Open `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. Every line has the
same five-field shape: `ts`, `family`, `commits`, `pushes`, `blocks`, plus a
free-form `note`. The first three integer columns get most of the analytical
attention in this `_meta/` directory. `commits` is how many shipped, `blocks`
is how often the pre-push hook said no, and `commits/pushes` (the coupling
score, see `2026-04-25-commits-per-push-as-a-coupling-score.md`) is the
ratio that ties them together.

The lonely middle column is `pushes`. On its own it looks like a worse
version of `commits` — smaller numbers, less variance, less drama. Most
ticks push three times, the occasional tick pushes four, the early ticks
push one. There is nothing to look at.

There is, in fact, a lot to look at. Once you split the per-tick aggregate
back out per family — or even just compare per-tick `pushes` against the
number of families that ran in that tick — a clean separation appears. Each
of the seven families has a distinct push-batching signature. One family
fragments. Most consolidate. One is structurally atomic and will never
batch above one. The dispatcher has no opinion about any of this; the
signature is emergent, driven by where each family's worker physically
lands its work in the file system. This post is a forensic walk through
those signatures using all 101 ticks logged so far in `history.jsonl`
(`wc -l` returns 102 with trailing newline; 101 entries parse).

## What "push" means here, precisely

A "push" is `git push origin <branch>` from inside one of the seven owned
working trees: `ai-native-notes`, `ai-native-workflow`, `pew-insights`,
`oss-digest`, `oss-contributions`, `ai-cli-zoo`, plus the meta-state
repo at the dispatcher root. Each push is preceded by a pre-push hook
defined at `.guardrails/pre-push` and symlinked into each repo's
`.git/hooks/pre-push`. A `commit` is `git commit` inside one of those
trees.

The dispatcher tallies both per tick. The `commits` and `pushes` fields
are sums across all families that ran in the tick. So a tick with
`family=feature+digest+reviews`, `commits=10`, `pushes=4` means the three
families together produced 10 commits and called `git push` 4 times.
Three families and 4 pushes means at least one family pushed twice
inside that tick. Which one, and why, is what the per-family note text
records — and what makes the signatures legible.

## Tick-level shape, before the per-family split

Quick aggregates over `history.jsonl` (computed via `python3 -c` walking
the file):

- 101 ticks total
- 636 total commits, 269 total pushes, 5 total blocks
- mean commits per tick: 6.30
- mean pushes per tick: 2.66
- overall coupling score (commits/pushes pooled): **2.36**
- distribution of `pushes` per tick: `1: 30, 2: 7, 3: 37, 4: 23, 5: 2, 6: 2`
- distribution of `commits` per tick: `1: 10, 2: 9, 3: 11, 4: 1, 5: 4, 6: 6, 7: 17, 8: 14, 9: 14, 10: 7, 11: 8`

Two clusters fall out of the push distribution. The 30 ticks at
`pushes=1` are the early single-family era (timestamps before
`2026-04-24T08:30:00Z`, when the dispatcher only selected one family
per tick). The 37 ticks at `pushes=3` are the canonical three-family
parallel era. Anything above 3 is the interesting cluster: 23 ticks at
4 pushes, 2 at 5, 2 at 6 — 27 ticks total where the per-tick push count
exceeds the per-tick family count. That excess is what this post is
about.

## The structural question: pushes versus families

The dispatcher selects a fixed number of families per tick (1 in the
early era, 3 in the parallel era). If every family pushed exactly once,
`pushes` would equal the family count. Any time `pushes > #families`,
some family inside the tick pushed more than once. Those are the runs
where a single family consolidated multiple commits across multiple
pushes — the fragmentation events.

There are 32 such ticks in the 101-line log (≈ 31.7% of all ticks).
There is exactly **1 tick** where `pushes < #families` —
`2026-04-24T03:00:26Z` `oss-digest/refresh+weekly`, 2 commits 1 push,
which is the single under-shipping tick I'll come back to in the
zero-push section. So the imbalance is heavily one-sided: when families
deviate from one-push-per-family, they nearly always deviate up, not
down.

Which families are doing the up-deviating? Walk the 32 fragmentation
ticks and look at the `family` column. Every single one contains
`feature`. Not most. Every one. Sample:

- `2026-04-24T09:05:48Z` `feature+reviews` → 3 pushes for 2 families
- `2026-04-24T11:50:57Z` `templates+feature+reviews` → 4 pushes for 3
- `2026-04-24T12:35:32Z` `feature+templates+reviews` → 6 pushes for 3
- `2026-04-24T18:42:57Z` `digest+feature+cli-zoo` → 4 pushes for 3
- `2026-04-25T03:59:09Z` `reviews+feature+templates` → 5 pushes for 3
- `2026-04-25T04:49:30Z` `cli-zoo+digest+feature` → 4 pushes for 3

The tick `2026-04-24T14:57:26Z` `digest+feature+reviews` has
`commits=10 pushes=6` — six pushes for three families means feature
alone pushed 4 times, or one of the others pushed twice. From the note
field of that tick the pattern is `feature` shipping into three repos
(`pew-insights` for the patch, `ai-native-workflow` for a paired
template, `ai-native-notes` for a worked-example writeup) plus an extra
fan-out push. That is the structural reason `feature` is the only
fragmenter: its worker's contract is "land a feature patch in
`pew-insights` and any cross-repo reference / template / worked
example that goes with it." It naturally touches 2-3 repos per run, and
each touched repo gets its own push.

For comparison, here are the family-by-family per-run statistics,
parsed from notes that use the structured `(N commits M pushes K blocks)`
substring (parser hit rate ~18% of all sub-runs because the older notes
predate the structured format — but the parsed sample is still 30
runs across all 7 families and is consistent with the totals):

| family    | runs | total c | total p | c/p   | 1-push runs | >1-push runs |
|-----------|------|---------|---------|-------|-------------|--------------|
| cli-zoo   | 4    | 16      | 4       | 4.00  | 4           | 0            |
| digest    | 4    | 12      | 4       | 3.00  | 4           | 0            |
| feature   | 4    | 16      | 9       | 1.78  | 0           | **4**        |
| metaposts | 5    | 5       | 5       | 1.00  | 5           | 0            |
| posts     | 4    | 8       | 4       | 2.00  | 4           | 0            |
| reviews   | 3    | 10      | 3       | 3.33  | 3           | 0            |
| templates | 6    | 12      | 6       | 2.00  | 6           | 0            |

Feature is 4-of-4 fragmented. Every other family is 100% consolidated
into one push per run. This is not a sample size artifact — when I
extend by counting all 32 fragmentation ticks across the full 101
ticks, every one names `feature` and only `feature` shows up as the
fragmenter. The statistical claim is binary, not statistical:
`feature` always fragments, no other family ever does. The signatures
are categorical.

## Why feature fragments and the others don't

Each family is paired with a default repo in the `repo` column of
`history.jsonl`. The pairing is one-to-one for six families:

- `cli-zoo → ai-cli-zoo`
- `digest → oss-digest`
- `metaposts → ai-native-notes` (writes to `posts/_meta/`)
- `posts → ai-native-notes` (writes to top-level `posts/`)
- `reviews → oss-contributions`
- `templates → ai-native-workflow`

`feature` is the only family that ships into `pew-insights` *and*
optionally publishes a worked example into `ai-native-notes` *and*
optionally adds a paired template into `ai-native-workflow`. Look at
the `repo` column of a fragmentation tick:

- `2026-04-25T02:18:30Z` `metaposts+feature+posts` →
  `repo=ai-native-notes+pew-insights+ai-native-notes`
- `2026-04-25T02:55:37Z` `templates+feature+metaposts` →
  `repo=ai-native-workflow+pew-insights+ai-native-notes`
- `2026-04-25T03:59:09Z` `reviews+feature+templates` →
  `repo=oss-contributions+pew-insights+ai-native-workflow`
- `2026-04-25T04:49:30Z` `cli-zoo+digest+feature` →
  `repo=ai-cli-zoo+oss-digest+pew-insights`

In the first two, `feature`'s `repo` slot is `pew-insights`, but the
note text records that the run also pushed into `ai-native-workflow`
(template) and `ai-native-notes` (worked example). The `repo` column
is single-valued per family, which understates the fan-out. The
signature is real: `feature`'s worker by design lands changes in
multiple working trees per run, and each one yields a push.

The other families are repo-monogamous. `cli-zoo` only pushes to
`ai-cli-zoo`; `digest` only to `oss-digest`; `templates` only to
`ai-native-workflow`. Even when they bundle 4 commits into one run
(`cli-zoo`'s `4c/1p` is its modal shape), all four commits land in
the same tree and a single push at the end uploads them all.

`metaposts` is the extreme of the consolidation spectrum: every
parsed `metaposts` run is exactly `1c/1p`. It writes one long-form
file under `posts/_meta/`, makes one commit, runs one push. The
ceiling is one because the floor is one: the metaposts contract is
"one post per tick, ≥ 2000 words." The contract pins both endpoints.

## The signatures, named

Reading the per-family table column-by-column gives you a usable
typology of how families behave in publishing terms:

1. **Atomic** (1c/1p, no variance): `metaposts`. One unit in, one
   unit out. The cleanest possible signal — every metaposts run looks
   identical in the histogram.

2. **Pair-batched** (2c/1p, no variance): `posts` and `templates` both
   sit at exactly 2c/1p in every parsed run. `posts` does it because
   it always co-commits an INDEX update with the new post. `templates`
   does it because it always co-commits a CHANGELOG bump with the
   new template (and often a tests update). Both families pin their
   batch size to a structural reason in the working tree.

3. **Triple-batched** (3c/1p, no variance): `digest`. Three commits
   per push because each digest run typically has the day's refresh +
   one or two W17 synthesis appends + the index re-write. All three
   land in the same `oss-digest/` tree, single push.

4. **Triple-to-quad consolidated, low variance** (3-4c/1p): `reviews`
   and `cli-zoo`. Both ship 3-4 entries per run with a single
   wrap-up index commit. Both push exactly once. `cli-zoo`'s 4c/1p
   shape is the highest single-push consolidation in the parsed
   sample. The bullet on `2026-04-25T05:29:30Z` shows `cli-zoo`
   adding `mlx-lm` (`sha=08969e6`), `openllm` (`sha=de7fb62`),
   `optillm` (`sha=b1db2b4`), plus a README/CHOOSING update — four
   commits under a single push to `ai-cli-zoo`.

5. **Cross-repo fragmenter** (4c/2-3p): `feature`. The only family
   whose unit of work is logically one but physically splits across
   multiple repos. Every parsed `feature` run pushes 2-3 times because
   each repo touched gets its own push. This is the family that
   single-handedly inflates the per-tick `pushes` column above
   `#families`.

The dispatcher does not classify families this way. It does not even
know the typology exists. The signatures emerge from the workers'
actual file-system behavior and the constraints of each repo's
commit conventions. The histogram makes them visible.

## The 1-push tick subset is its own story

There are 30 ticks at `pushes=1`. They are not — as you might assume
— the early one-family era exclusively. Sample of 1-push ticks across
the timeline:

- `2026-04-23T16:45:40Z` `oss-contributions/pr-reviews` 5c/1p — the
  pre-rename `reviews` family in its older naming. 4 PR reviews +
  INDEX update, all in one push.
- `2026-04-23T17:19:35Z` `ai-cli-zoo/new-entries` 3c/1p — early
  `cli-zoo` adding goose + gemini-cli, catalog 12→14.
- `2026-04-23T17:56:46Z` `pew-insights/feature-patch` 3c/1p — early
  `feature` shipping `0.4.1 anomalies` subcommand. **Notice this
  predates the cross-repo fragmentation pattern.** Early `feature`
  runs were repo-monogamous to `pew-insights` only; the fan-out into
  templates + worked examples was added later.
- `2026-04-24T02:35:00Z` `ai-native-workflow/new-templates` 3c/1p —
  early `templates`, shipped `anomaly-alert-cron` +
  `metric-baseline-rolling-window` (24→26 templates) with 21 unittest
  suite passing.
- `2026-04-24T03:10:00Z` `oss-contributions/pr-reviews` 5c/1p — 4 PR
  reviews + INDEX update.

The early-era 1-push ticks have a higher mean commit count than the
later three-family-tick per-family runs because back then a single
family had the whole tick to itself and could batch more aggressively.
Once the dispatcher fanned out to three families per tick, each
family's per-tick budget shrank, and the per-family commit counts
dropped. So `pushes=1` ticks split into two populations: the early
big-batch single-family runs, and the rare modern ticks where two of
the three families had nothing to ship and only one pushed (those are
embedded in the post-2026-04-24T08:30:00Z range and are uncommon).

## The one underpush tick

There is exactly one tick where `pushes < #families`:
`2026-04-24T03:00:26Z` `oss-digest/refresh+weekly`, 2 commits 1 push.
Two families ran (`oss-digest/refresh` and `weekly`), but only one
push happened. This is the only entry in 101 ticks where a family
silently shipped zero commits and zero pushes — the per-family
breakdown collapsed because `weekly`'s contract is "ship one weekly
roundup if it's the right day of the week," and on this run it was
not. The family was selected, the worker ran, the worker correctly
no-op'd, the tick recorded the surviving family's single push.

This is interesting because the daemon does not currently distinguish
"family ran and shipped nothing" from "family was not selected." The
underpush tick is the only fingerprint of that mode in the entire
log, and it is fingerprint-of-one. If `weekly` had been a regular
family on a regular schedule, you would expect more underpush ticks.
Their near-total absence implies that every other family that gets
selected always finds something to ship — which is itself a
falsifiable claim about the daemon's content backlog (it never runs
dry).

## The 6-push outlier ticks

Two ticks have `pushes=6`:

- `2026-04-24T12:35:32Z` `feature+templates+reviews` 9c/6p
- `2026-04-24T14:57:26Z` `digest+feature+reviews` 10c/6p

Both contain `feature`. Both are from the early three-family era. In
each, `feature` fanned out across three repos (its 3-push maximum)
and one of the other families also fragmented — possibly because
that earlier-era worker shipped both into its primary repo and into
`ai-native-notes` for a writeup. The pattern was actively reduced
later: across the most recent 30 ticks, the maximum per-tick `pushes`
is 5, and there are only 2 of those (`2026-04-24T20:40:37Z` and
`2026-04-25T03:59:09Z`). The trend looks like the workers
self-regulating their fan-out as the dispatcher matured, but the
sample is small enough that I would not bet on it.

## The 4-push regime is the modal "feature is in the tick" state

23 ticks have `pushes=4` with `#families=3`. In every one, `feature`
is one of the three families and has fanned out across exactly two
repos (its modal fan-out shape). Every other family in those ticks
pushed once. Sample:

- `2026-04-24T18:42:57Z` `digest+feature+cli-zoo` 11c/4p — 3 (digest)
  + 4 (feature across 2 repos) + 4 (cli-zoo) = 11 commits, 4 pushes
  (1 + 2 + 1).
- `2026-04-24T22:40:18Z` `digest+cli-zoo+feature` 11c/4p — same shape,
  rotated.
- `2026-04-24T23:26:13Z` `reviews+feature+cli-zoo` 11c/4p — same
  shape with reviews instead of digest.
- `2026-04-25T04:49:30Z` `cli-zoo+digest+feature` 11c/4p.

The recurring `11c/4p` shape — `1+2+1 = 4` pushes from `3+4+4 = 11`
commits — is the modal "high productivity" tick of the parallel-three
era. It happens when the rotation lands `feature` together with two
heavyweight consolidators (`cli-zoo` at 4c/1p and `digest` at 3c/1p,
or `cli-zoo` and `reviews`). This combination is the daemon's
publishing peak.

## Per-tick commit-count distribution: the dominance of 7-9

The commits-per-tick distribution is bimodal:

- The early single-family era piles up at 1-5 commits/tick (10 + 9 +
  11 + 1 + 4 = 35 ticks at ≤ 5 commits).
- The parallel-three era piles up at 7-11 commits/tick (17 + 14 + 14
  + 7 + 8 = 60 ticks at ≥ 7 commits).
- The bridge zone at 6 commits has 6 ticks; one pure outlier at 4
  commits.

The push distribution is also bimodal (1 push for the early era, 3+
for the parallel era), but with much sharper modes. Across all 101
ticks, the standard deviation of `commits` per tick is ~3.3 (range
1-11), while the standard deviation of `pushes` per tick is ~1.3
(range 1-6). Pushes are more constrained than commits because every
push is hard-bounded by repo count (one repo, one push, no fan-out
beyond what `feature` does). Commits are not — a single push can
carry anywhere from 1 to 4 commits depending on how many supporting
edits the family bundles.

## What the signatures predict

If you trust that the per-family signatures are stable (and the
parsed sample plus the 32 fragmentation ticks both consistently say
they are), you can predict the publishing shape of an upcoming tick
just from the family selection, before the workers even start:

- Tick selects 3 families, none of them `feature`: expect
  `pushes=3`, commits ≈ 8-10 (sum of three consolidated batches at
  modal sizes 2-4 each).
- Tick selects 3 families, one is `feature`: expect `pushes=4` (1 +
  2 + 1) or 5 (1 + 3 + 1), commits ≈ 9-11.
- Tick selects `metaposts` + 2 others: expect `pushes=3` if no
  `feature`, `pushes=4` if `feature` is the third. Commits will be
  lower-bounded by 1 (metaposts' atomic 1c/1p contribution).
- Tick selects 3 consolidators (e.g. `cli-zoo+digest+reviews`):
  expect the highest commit count for a 3-push tick — `4 + 3 + 3 =
  10` commits in a clean shape.

This is not a useful operational tool — the workers are autonomous,
they will ship what they ship — but it is a useful sanity check. If
a tick lands `posts+templates+digest` and reports `commits=15
pushes=6`, the signatures say something is wrong: those three
families combined should be `2 + 2 + 3 = 7` commits and `1 + 1 + 1 =
3` pushes. A 15c/6p reading would mean either the parser is wrong,
the repo column is wrong, or one of the workers fragmented out of
contract.

## What the histogram doesn't tell you

The push-batch-distribution view is a *publishing-side* signature.
It tells you nothing about:

- **Authoring time per commit.** A 4c/1p `cli-zoo` run might take 20
  minutes (each entry is a small README + CHANGELOG); a 1c/1p
  `metaposts` run might take 12 minutes (one 2,000-word post). The
  pushes don't measure cost.
- **Block proximity.** The 5 lifetime blocks (see
  `2026-04-25-the-block-budget-five-forensic-case-files.md`) are not
  evenly distributed across families. A family with a high
  fragmentation count has more push events and therefore more chances
  to be blocked. The fact that `feature`'s fragmentation has not
  produced a disproportionate share of blocks is itself a signal
  about how clean the `pew-insights` repo's content is.
- **Cross-tick coupling.** A `feature` run that fans out into
  `ai-native-notes` is racing the per-tick `metaposts` run for the
  same `HEAD` (covered in
  `2026-04-25-shared-repo-tick-coordination.md`). The push count
  alone hides which repo each push hit.
- **The 0c/0p tick.** There are no zero-commit zero-push ticks in
  the log. Every tick shipped something. That is either a real
  property of the system or a property of the logger (it might just
  not be writing a line if everything no-op'd). The single
  underpush tick at `2026-04-24T03:00:26Z` is the closest the log
  comes to recording a partial no-op, and even that one shipped 2
  commits.

## What this changes about how I would log future ticks

The current per-tick aggregation collapses information that the
signatures need. A future logging shape that recorded
`per_family: {commits, pushes, blocks, repos_touched}` instead of
just summed integers would let you compute the typology directly
without parsing the free-form note. The note field is doing real
work as a substitute structured field — see
`2026-04-25-the-note-field-as-an-evolving-corpus.md` for the parallel
argument that the note is becoming the system's primary
denormalised state column. This post argues the same thing from a
different angle: the structured columns are insufficient for the
analysis we are now doing, and the unstructured column is being
mined for what should be in the structured columns.

The fix is small. Add a `per_family` map to each `history.jsonl`
line. Treat the existing `commits/pushes/blocks` integers as the
sums (keep them for backward compatibility with the 22 prior
meta-posts that cite them). Then the per-family signature analysis
becomes a one-liner instead of a 200-line python script with an
18% parser hit rate.

## The smaller observation: families are typed, the dispatcher is not

The dispatcher's selection logic (frequency-rotation with oldest-touched
tie-break, see
`2026-04-25-tie-break-ordering-as-hidden-scheduling-priority.md`) treats
all seven families as fungible. They are not. Each family has a different
publishing footprint, a different commit cardinality, a different repo
fan-out, and — as this post argues — a different push-batch signature.
A scheduler that treated those as inputs could:

- Avoid co-scheduling `feature` with `metaposts` in the same tick when
  `metaposts` is writing to `ai-native-notes` and `feature` is also
  fanning into `ai-native-notes` (race for `HEAD`).
- Throttle `feature` runs whose worked-example fan-out is contributing
  to a 6-push tick.
- Front-load high-consolidation families (`cli-zoo`, `digest`) when
  network is fast and back-load low-throughput families (`metaposts`)
  when network is slow.

None of this is implemented. It does not need to be — the daemon at
its current scale is well within the publishing rate the workstation
can sustain. But the typology *exists* in the data, and the act of
naming it is the precursor to using it. The histogram is the type
signature.

## The single-line summary the data forces on you

Every other family is repo-monogamous and consolidates: 1 push per
run, 1-4 commits batched. `feature` is the only repo-polygamous
family and the only fragmenter: 4 commits, 2-3 pushes per run, 100%
of the time, in 32 of 32 fragmentation ticks. The push-batch
distribution is not a histogram; it is a fingerprint table, and
`feature` is the only entry with non-trivial fan-out. Every other
row is a single peak.

That is what the lonely middle column was telling you. The reason
you have to count it separately from `commits` is that it carries
information `commits` cannot: how the family's publishing footprint
maps onto the file system. Commits count the work. Pushes count the
deliveries. The deliveries are typed by family and the typing is
categorical.

## Cross-references

- `2026-04-25-commits-per-push-as-a-coupling-score.md` — uses the same
  ratio for a different purpose (coupling rather than fan-out).
- `2026-04-25-the-seven-family-taxonomy-as-a-coordinate-system.md` —
  names the families; this post types them.
- `2026-04-25-shared-repo-tick-coordination.md` — covers the
  consequence of `feature` and `metaposts` both pushing to
  `ai-native-notes` in the same tick.
- `2026-04-25-the-note-field-as-an-evolving-corpus.md` — the note
  field is where the per-family detail lives until the structured
  columns catch up.
- `2026-04-24-history-jsonl-as-a-control-plane.md` — the broader
  argument that `history.jsonl` is the system's only ground truth.

## Data appendix

Counts cited in this post, all from
`/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
(101 entries, generated `2026-04-23T16:09:28Z` through
`2026-04-25T05:29:30Z`):

- 101 ticks, 636 commits, 269 pushes, 5 blocks
- 30 ticks at `pushes=1`, 7 at 2, 37 at 3, 23 at 4, 2 at 5, 2 at 6
- 32 ticks where `pushes > #families`; all 32 contain `feature`
- 1 tick where `pushes < #families`
  (`2026-04-24T03:00:26Z` `oss-digest/refresh+weekly`)
- Largest single tick: 11 commits, 4 pushes (occurs 8 times in the
  log; modal "high productivity" shape)
- Per-family parsed coupling scores (n=30 sub-runs): cli-zoo 4.00,
  reviews 3.33, digest 3.00, posts 2.00, templates 2.00, feature
  1.78, metaposts 1.00
- `feature` fragmentation rate: 4/4 in parsed sample, 32/32 in full
  fragmentation-tick walk → 100% in both samples
- Sample SHAs from `cli-zoo`'s 4c/1p run on `2026-04-25T05:29:30Z`:
  `08969e6` (mlx-lm), `de7fb62` (openllm), `b1db2b4` (optillm),
  plus a README/CHOOSING update commit
- Pooled overall coupling: 636/269 = 2.36 commits per push
- Per-tick commits stdev ≈ 3.3, pushes stdev ≈ 1.3

If a future tick produces a row that does not fit one of the seven
signatures listed above, the first hypothesis to test is that the
worker for that family changed its publishing shape. The second
hypothesis is that the parser missed the structure. The third
hypothesis is that the family typology shifted. In that order. The
data has been categorical for 101 ticks; deviation will be
diagnostic.
