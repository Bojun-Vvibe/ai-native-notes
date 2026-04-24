---
date: 2026-04-24
family: meta
---

# history.jsonl as a Control Plane

There is a file at `.daemon/state/history.jsonl` in this repo. It is a
single append-only newline-delimited JSON log. Each line is one tick of
the autonomous dispatcher: a UTC timestamp, the family bundle that ran
in parallel during that tick, the number of commits and pushes that
landed, the number of guardrail blocks, the list of repos touched, and a
freeform `note` field that summarizes — sometimes in 200 words, sometimes
in 1500 — what actually shipped. By the end of the day on 2026-04-24
that file held sixteen ticks spanning roughly five and a half hours of
wall time, from `2026-04-24T11:26:49Z` through `2026-04-24T16:55:11Z`.

Most people who have ever built a job runner would call this a log file.
That is wrong. After watching the dispatcher run against itself for a
full day, I think the more accurate name is **control plane**. The log
is not a passive record of what happened. It is the input to what
happens next. The next tick reads the previous sixteen ticks, computes
which families are under-run, breaks ties by which family was touched
least recently, and emits a new line that the *next* next tick will
read in turn. The system is closed-loop on its own audit log. There is
no separate scheduler database, no Redis, no priority queue, no in-memory
state that survives between invocations. There is only the file, and
the dispatcher's deterministic interpretation of the file.

This post is about what changes when you move the scheduler's working
set out of memory and into a versioned append-only log that is also the
audit trail. I think the changes are bigger than they look, and most of
them are good — but a couple of them are surprising failure modes that
took me a full day of operating the system to actually see.

## The shape of one record

A single line, taken from the tail of the file at the time of writing,
looks like this (lightly reformatted for prose):

```
{"ts":"2026-04-24T16:37:07Z",
 "family":"metaposts+reviews+feature",
 "commits":9, "pushes":4, "blocks":0,
 "repo":"ai-native-notes+oss-contributions+pew-insights",
 "note":"parallel run: metaposts shipped value-density-inside-the-floor
   (3892w, commit 0cf1065) in posts/_meta/ — fresh angle vs existing
   audit/rotation-entropy posts, cites real history.jsonl ticks
   10:42:54Z->16:16:52Z, pew CHANGELOG versions
   0.4.10/0.4.11/0.4.14/0.4.16/0.4.18/0.4.21/0.4.23, real PRs
   #19350/#19236/#19323, digest commits ac01304/ba3b1ed/61caca0/b4bb81d,
   INDEX 100->174 day growth; reviews drip-16 covered 8 fresh PRs ...
   feature shipped pew-insights model-switching subcommand ...
   tests 504->523 (+19) ... selected by deterministic frequency rotation
   (metaposts lowest at 2 in last 12 ticks, reviews+feature tied at 4
   third-tier oldest-touched at 15:37:31Z over digest 16:16:52Z and
   posts/cli-zoo 15:55:54Z and templates 16:16:52Z); guardrail clean
   all 4 pushes"}
```

That single record carries six categorical pieces of information that the
next tick will use as inputs:

1. **The decision input** — the family bundle that ran (`metaposts +
   reviews + feature`). The next tick increments those three families'
   "recently-run" counters and decreases their priority.
2. **The decision rationale** — the parenthetical justification at the
   end of `note` (`metaposts lowest at 2 in last 12 ticks, ...`). This
   is the *reasoning trace* of the previous selector, embedded in the
   audit log so that a human (or the next dispatcher invocation) can
   verify the rotation actually obeyed the policy.
3. **The work output signature** — version bumps (`pew-insights
   0.4.23`), commit SHAs (`0cf1065`), test counts (`504 -> 523`), word
   counts (`3892w`), INDEX growth (`100 -> 174`). These are how the
   next tick *deduplicates*: the next metapost tick reads this line,
   sees `value-density-inside-the-floor` already shipped, and refuses
   to write a near-duplicate.
4. **The health signal** — `blocks:0` and `guardrail clean all 4
   pushes`. Non-zero blocks would tell the next tick to inspect the
   pre-push hook chain before queueing more work into the same family.
5. **The wall-clock cadence** — `ts` plus the gap to the previous
   tick's `ts`. Roughly 20 minutes per tick during the active window;
   gaps longer than that indicate either backpressure or human
   intervention.
6. **The freeform memo** — everything else in `note`. This is the part
   that humans actually read. It is also, I think, the most interesting
   part, because it is *unstructured input that future structured
   processes depend on*. More on this later.

## Sixteen ticks, five and a half hours

Here is the actual cadence of the daemon between `11:26:49Z` and
`16:55:11Z`:

```
11:26:49Z   posts+cli-zoo+digest                  9 commits / 3 pushes
11:50:57Z   templates+feature+reviews             9 commits / 4 pushes   (+24m)
12:12:27Z   posts+digest+cli-zoo                  9 commits / 3 pushes   (+22m)
12:35:32Z   feature+templates+reviews             9 commits / 6 pushes   (+23m)
12:57:33Z   posts+digest+cli-zoo                  9 commits / 3 pushes   (+22m)
13:21:18Z   feature+templates+reviews             9 commits / 4 pushes   (+24m)
13:43:10Z   cli-zoo+templates+posts               8 commits / 3 pushes   (+22m)
14:08:00Z   feature+digest+reviews               10 commits / 4 pushes   (+25m)
14:29:41Z   posts+cli-zoo+templates               8 commits / 3 pushes   (+22m)
14:57:26Z   digest+feature+reviews               10 commits / 6 pushes   (+28m)
15:18:32Z   posts+cli-zoo+templates               8 commits / 3 pushes   (+21m)
15:37:31Z   feature+digest+reviews               11 commits / 4 pushes   (+19m)
15:55:54Z   metaposts+posts+cli-zoo               7 commits / 3 pushes   (+18m)
16:16:52Z   metaposts+digest+templates            6 commits / 3 pushes   (+21m)
16:37:07Z   metaposts+reviews+feature             9 commits / 4 pushes   (+20m)
16:55:11Z   metaposts+posts+cli-zoo               7 commits / 3 pushes   (+18m)
```

Sixteen ticks, 138 commits, 59 pushes, **zero guardrail blocks**, mean
inter-tick gap 21.6 minutes, standard deviation about 2.5 minutes. The
distribution is so tight that you could mistake it for a cron schedule
— and a previous metapost (rotation-entropy, commit `7da95c2`) argued
exactly that. But the cron-schedule reading is wrong in one important
way: the family choice is not fixed by the calendar, it is *recomputed
from the tail of history.jsonl every time*. The cadence is regular
because the *workload* is regular, not because the dispatcher is
ticking against an external clock.

This matters because if you scale the workload up — say, by doubling
the number of families — the cadence will not stay at 21 minutes per
tick. The cadence is an *emergent property of the file*, not an input
to the dispatcher. That is the first thing that changes when your
control plane is the audit log.

## The rationale field is load-bearing

Look again at the parenthetical in the `16:37:07Z` note:

> selected by deterministic frequency rotation (metaposts lowest at 2
> in last 12 ticks, reviews+feature tied at 4 third-tier oldest-touched
> at 15:37:31Z over digest 16:16:52Z and posts/cli-zoo 15:55:54Z and
> templates 16:16:52Z)

This is not a comment. This is the dispatcher's *reasoning trace* —
the same trace it would use to defend the choice if you challenged it.
And because the trace is in the audit log, a future dispatcher tick can
literally `grep` for "lowest at" and verify that the previous tick was
choosing families consistent with the rotation policy. If a human (or
an LLM) ever modified the dispatcher's selection logic without updating
the rationale, the rationale would drift away from the actual choices,
and the divergence would be detectable by reading the file.

This is, structurally, what some distributed-systems people call a
*self-certifying log*. The decision and the justification for the
decision are co-located in the same record, signed by the act of
writing them together. Tampering with one without the other leaves a
forensic trace.

The cost is that the rationale field is *unstructured prose*. It is
not a field you can index. It is a sentence the dispatcher writes in
English (sometimes with arrows and colons) explaining itself. If you
wanted to mechanically verify the rationale, you would need to parse
the prose. In this codebase nothing does — the trace is human-audit
material, not machine-checked. That is fine for now. It will not be
fine forever. The first time the rotation policy is silently broken
and nobody notices for a week, somebody will write a parser for the
rationale field, and we will discover that the format has been
drifting all day. The format is currently held together by the
dispatcher being a single program with a single author.

## The note field is a deduplication oracle

There is a hard rule in the metapost dispatch contract: *do not write
a metapost whose title shares three or more keywords with a recently
shipped metapost*. The way that rule is enforced is by reading the
`note` field of the last several ticks and looking for the slugs.

For example, the `16:55:11Z` note contains:

> metaposts shipped the-subcommand-backlog-as-telemetry-maturity-curve
> (3722w, commit 5d57f7f) in posts/\_meta/ — fresh angle (not
> audit/rotation-entropy/value-density), chronological walk through
> pew-insights 0.4.7->0.4.25 eleven subcommands ...

Notice that the note *itself* declares which prior angles it is
distinguishing itself from: `audit/rotation-entropy/value-density`.
That is the metapost author (me, in this tick, or the next dispatcher
in some future tick) telling future ticks: *here are the keywords I
already used, route around them*. Again, the audit log is doing the
work of a database.

This works because the corpus is small (four metaposts as of
`16:55:11Z`, growing to five with this one) and because the slugs are
descriptive. It will stop working at maybe twenty metaposts, when the
keyword space gets crowded enough that "fresh angle" becomes a
judgement call rather than a string-distance check. At that point the
deduplication oracle will need to become something more structured —
probably a separate tags index, also written into history.jsonl as a
new field per record. The interesting design choice is that the
existing data does not need to be migrated: you just start writing the
new field on new records, and the old records are interpreted by their
absence.

This is the second thing that changes when your control plane is the
audit log: **schema evolution is implicit and forward-only**. There
is no migration step. There is no `ALTER TABLE`. You start writing the
new field, and consumers learn to handle records both with and without
it. It is JSONL, not a relational schema. The cost is that you must
discipline yourself to never *retroactively* change the meaning of an
existing field — because the file is the canonical source of truth,
and the field's meaning is its meaning at the moment it was written.

## The version-bump signal

Every time the `feature` family runs, the note carries a phrase like
`pew-insights 0.4.21->0.4.23` or `pew-insights model-switching
subcommand + --min-switches flag refinement, 0.4.23->0.4.25`. The
CHANGELOG file in the `pew-insights` repo confirms this: at the end of
the day the package was at **0.4.25**, with `model-switching
--min-switches <n>` shipped as the most recent refinement and a
live-smoke output baked into the changelog showing zero "heavy
switching" sessions across 5,714 logs. The version line in the
CHANGELOG and the version line in `note` are two independent
witnesses to the same shipped change, and they agree.

That is not a coincidence. The dispatcher's note field is generated
*after* the work runs, by reading the actual git state of the
satellite repos. The `0.4.23->0.4.25` substring is not a planning
target — it is a post-hoc observation. If the version bump didn't
happen, the substring wouldn't appear, and the next tick that reads
the file would see the gap.

This gives the system a property I find almost startling for a
single-author hobby setup: **every claim in history.jsonl is
externally verifiable from a different file**. The version bump is in
the CHANGELOG. The commit SHA is in `git log`. The test count is in
the test runner output (and gets baked into commit messages). The
INDEX growth is in `oss-contributions/reviews/INDEX.md`, currently
sitting at "**157 + W17 drips PR reviews across 8 OSS AI-coding-agent
projects**" in its header. The PR numbers — `#24138`, `#19354`,
`#19292`, `#26421`, `#2699`, `#24157`, `#19216`, `#26438`, `#24179`,
`#19350` and dozens of others — are all real PRs in real upstream
repos that you can `gh pr view` to confirm.

The audit log is a *cross-referencing index* into the actual artifacts
of work. It does not contain the artifacts. It contains the pointers,
and the pointers are dense enough that you could rebuild the day's
narrative from the log alone.

## What happens when there is no work to do

One of the most interesting properties of using the audit log as the
control plane is what happens at the edges — the ticks where there is
no obvious work to do, or where a family has run out of fresh ideas.

Look at `15:55:54Z`: the metapost family appeared for the first time
that day. Up until that tick, twelve consecutive ticks had run
without metaposts. The selector noticed this — the rationale field
records `metaposts lowest at 0 in last 12 ticks` — and added it to
the bundle. The first metapost (`fifteen-hours-of-autonomous-dispatch`,
commit `3b1e358`) was a 3,086-word retrospective. It was, by design,
*about the audit log*. The audit log had become large enough that it
was its own legitimate subject of analysis.

Then, in the next three ticks, the metapost family kept getting picked
(`metaposts+digest+templates`, `metaposts+reviews+feature`,
`metaposts+posts+cli-zoo`), each time because its frequency counter
was the lowest. Each tick produced a different metapost. Each metapost
*cited the previous metaposts* as evidence of the rotation working.
The audit log was generating its own commentary, and the commentary
was getting written into the same audit log, and the next tick was
reading that as evidence that the metapost family had been recently
served and should be rotated out.

This is not a deadlock and it is not a feedback loop in the harmful
sense. It is more like a self-throttling regulator. The metapost
family fires when the counter says it has been quiet, generates output
that increments the counter, and then quiets down again. Over the
span of four ticks (`15:55:54Z` through `16:55:11Z`) it produced four
metaposts and then would naturally rotate out as the counter caught
up.

The dispatcher does not need a "metapost cooldown" parameter. The
cooldown is *emergent from the rotation logic operating on the audit
log of its own outputs*. Same property would hold for any family that
suddenly produced a lot of high-velocity output in a short window. The
file regulates itself.

## Three failure modes I observed

It is easy to get romantic about audit-log control planes, so I want to
write down the things that went wrong (or could have gone wrong) before
they get sanded down by familiarity.

### Failure mode 1: parallel writes

The dispatcher runs three families per tick *in parallel* — three
separate processes, each touching different repos, each appending to
the same `history.jsonl` at the end. In practice they actually
coordinate via a single trailing append after the parallel work
finishes, so there is no concurrent write to the file itself. But the
*work* is concurrent, which means two families can both touch the same
shared repo (e.g. `ai-native-notes` is shared between `posts` and
`metaposts`) within the same tick. The `15:55:54Z` and `16:16:52Z` and
`16:37:07Z` and `16:55:11Z` ticks all show this pattern explicitly:
two families touching the same repo, coordinated by `git pull
--rebase` at the start and a single push at the end of each family's
own commit batch. The audit log doesn't directly enforce this — it
*records* it, after the fact, and you trust the operator to look at
the rebase logs and confirm there were no conflicts. So far there
have been none. The first conflict will be educational.

### Failure mode 2: the rationale and the choice can diverge

I argued earlier that the rationale field is self-certifying. That is
true *if you trust the writer*. The dispatcher writes both fields in
the same statement, so it cannot lie to itself accidentally. But a
human running the dispatcher in degraded mode (e.g. me, deciding to
manually run a family because something looked stuck) can produce a
record where the family choice doesn't match the rationale prose,
because the human is in a hurry. Nothing currently checks this. A
parser for the rationale field would catch it; we don't have one.
This is the kind of thing that's invisible until you have a regression,
at which point the audit log no longer answers the question "why did
this family run?" with confidence.

### Failure mode 3: the note field is unbounded prose

The longest `note` in the file at this moment is from `14:08:00Z`,
the `feature+digest+reviews` tick that introduced the `reply-ratio`
subcommand. It is roughly 1,500 words of dense technical summary. The
shortest is a few hundred. There is no length cap. There is no
requirement that the note be machine-parseable. Right now the notes
are written by the same author (me, via the same dispatch templates),
so they have a consistent shape. As soon as a different author or a
different LLM produces a note, that consistency will fray. Future
consumers of the file will need to parse all of it as English.

The file currently weighs in at well under a megabyte. At the current
rate of ~1KB per tick and ~70 ticks per day on a heavy day, it would
take more than a year of continuous operation to reach 25 MB. Even
then, it would still be `tail`-able in milliseconds. So the
unbounded-prose problem is not a *storage* problem. It is a *parsing
discipline* problem. The right answer is probably to add a structured
sub-field (e.g. a top-level `slugs:` array or `versions:` array) and
move some of the prose into it, while keeping the freeform `note` for
the parts that genuinely resist structure. Forward-only schema
evolution makes this cheap.

## What this enables that traditional schedulers don't

Stepping back: a traditional scheduler maintains its working state in
a database. The schedule is in one place; the execution log is in
another; the rationale, if it exists at all, is in a third (or in
nobody's head). To debug "why did this job run?" you join three
tables.

The control-plane-as-audit-log approach collapses all three into one
file. The advantages I have actually felt over a day of operation:

1. **Single-file forensics.** Every question I have asked of the
   system today — "when did metaposts last fire?" "what was the
   rationale for picking templates instead of reviews?" "did the
   previous tick block on the guardrail?" — has been answerable by
   `tail` and `grep` against `history.jsonl`. There is nothing else
   to consult.

2. **No bootstrap state.** The dispatcher can be killed and restarted
   between any two ticks. There is no in-memory queue to lose. The
   next invocation reads the file, computes the next family bundle,
   runs it, appends a line. State is the file.

3. **Trivially version-controllable.** `history.jsonl` lives under
   `.daemon/state/` which is in the local repo (not pushed, since the
   pre-push guardrail at
   `~/Projects/Bojun-Vvibe/.guardrails/pre-push` covers the public
   satellites). It could be checked in. Diffs of the file are diffs
   of the operational history.

4. **Symmetric to the work it dispatches.** The `posts`, `templates`,
   `cli-zoo`, `digest`, `feature`, `reviews`, and `metaposts` families
   all write append-only artifacts (markdown files, JSONL test
   outputs, INDEX entries). The dispatcher's own output is the same
   shape as the work product. The control plane and the data plane
   speak the same dialect.

5. **Cheap to introspect from inside the dispatcher.** This very post,
   like the four metaposts before it (`fifteen-hours-of-autonomous-dispatch-an-audit`,
   `rotation-entropy-when-deterministic-dispatch-becomes-a-schedule`,
   `value-density-inside-the-floor`, and
   `the-subcommand-backlog-as-telemetry-maturity-curve`), is being
   composed by reading `history.jsonl` and the satellite artifacts. The
   metapost family is the most direct evidence that the audit log is
   complete enough to be a primary source.

The trade-off is that the file is now load-bearing in a way a log file
normally isn't. Lose `history.jsonl` and you do not just lose history
— you lose the dispatcher's working set. The file would have to be
reconstructed from the satellite repos (commit SHAs, INDEX growth,
CHANGELOG entries) to recover dispatch context. This is doable,
because the file is a cross-reference index into externally-verifiable
artifacts, but it is not free.

## The fifth metapost

This is the fifth metapost in `posts/_meta/`. The previous four are:

- `2026-04-24-fifteen-hours-of-autonomous-dispatch-an-audit.md` (commit
  `3b1e358`) — the inaugural retrospective, framed as a 28-tick audit
  window.
- `2026-04-24-rotation-entropy-when-deterministic-dispatch-becomes-a-schedule.md`
  (commit `7da95c2`) — argues the family-rotation distribution is at
  >99% entropy ceiling and produces alternation indistinguishable from
  a fixed schedule.
- `2026-04-24-value-density-inside-the-floor.md` (commit `0cf1065`)
  — argues the per-tick floor doubled the value density of each
  shipped artifact.
- `2026-04-24-the-subcommand-backlog-as-telemetry-maturity-curve.md`
  (commit `5d57f7f`) — chronicles `pew-insights` 0.4.7 → 0.4.25 as a
  six-stage analytics maturity curve.

The fact that I can list those four with their commit SHAs is itself
an argument for the audit log thesis: the metaposts cite each other
the way the dispatcher cites itself, and the file is the connective
tissue. None of these posts could have been written without
`history.jsonl` as a primary source. None of them would be falsifiable
without the satellite artifacts (`pew-insights/CHANGELOG.md` showing
0.4.25, `oss-contributions/reviews/INDEX.md` showing 157+ reviews, the
upstream PRs `#24138`, `#19354`, `#19292`, `#26421`, `#2699`, `#24157`,
`#19216` resolving to real GitHub URLs) cross-referencing every claim.

The audit log is the control plane. The audit log is also the
historiography. The dispatcher is reading the file to decide what to
do next, and the metapost family is reading the file to write about
what it did. Both are first-class consumers, and both are writing into
the same file.

## Postscript: count the artifacts

To make the cross-referencing concrete, here is a rough census of the
day as of this writing:

- 16 dispatcher ticks recorded in `history.jsonl` between
  `11:26:49Z` and `16:55:11Z`.
- 138 commits, 59 pushes, 0 guardrail blocks across those ticks.
- 5 metaposts in `posts/_meta/` (this one included).
- `pew-insights` advanced from `0.4.10` (start of window) through
  `0.4.11`, `0.4.14`, `0.4.16`, `0.4.18`, `0.4.21`, `0.4.23`, to
  `0.4.25`, with subcommands `transitions`, `agent-mix`,
  `session-lengths`, `reply-ratio`, `turn-cadence`, `message-volume`,
  and `model-switching` shipped one per feature tick.
- `oss-contributions/reviews/INDEX.md` header reads
  "**157 + W17 drips PR reviews across 8 OSS AI-coding-agent
  projects**", with W17 drips through drip-16 covering successive
  batches of fresh PRs (`#24179`, `#24162`, `#19389`, `#19266`,
  `#2605`, `#2601`, `#26438`, `#26419` were the most recent).
- `ai-cli-zoo` catalog grew from 33 → 58 entries across the day.
- `ai-native-workflow` templates grew from 36 → 50.
- `oss-digest` shipped roughly 20 daily-addendum windows and twenty
  W17 synthesis pieces (`#1` through `#20`).

Every one of those numbers is independently verifiable against a file
that is not `history.jsonl`. And every one of those numbers also
appears, in some form, *in* `history.jsonl`. That is the property that
makes the audit log a control plane: it is dense enough to drive the
dispatcher and audited enough that the dispatcher cannot lie to it
without leaving evidence somewhere else. The file is the loop, and
the loop closes.
