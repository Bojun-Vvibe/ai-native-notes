---
title: "Project concentration across sessions: Gini 0.83, top-1 holds 31.8%, and the 1320-project long tail"
date: 2026-04-25T23:11:15Z
tags: [pew, sessions, project-concentration, gini, telemetry, long-tail]
---

The session queue at `~/.config/pew/session-queue.jsonl` carries a `project_ref`
field on every recorded session — an opaque hex identifier that groups sessions
by working directory or project root. I had not pulled this dimension before,
and it turns out to be the most concentrated distribution I have seen come out
of this dataset: across 8,393 sessions distributed over 1,320 distinct
projects, a single project owns nearly one third of all sessions, and the top
five own two thirds. Computed at 23:11 UTC on 2026-04-25:

```json
{
  "metric": "project_concentration_across_sessions",
  "computed_at": "2026-04-26",
  "source_file": "~/.config/pew/session-queue.jsonl",
  "n_distinct_projects": 1320,
  "n_sessions_with_project": 8393,
  "gini": 0.8321,
  "top1_share": 0.3177648040033361,
  "top5_share": 0.6724651495294889,
  "top10_share": 0.7271535803645895,
  "top15": [
    {"project_ref": "8001c27439650c5c", "sessions": 2667, "share": 0.31776},
    {"project_ref": "0d6e4079e36703eb", "sessions": 1347, "share": 0.16049},
    {"project_ref": "62f8e1ec095e1857", "sessions": 634,  "share": 0.07554},
    {"project_ref": "f5756e707027c9b7", "sessions": 513,  "share": 0.06112},
    {"project_ref": "cb12e97eab40781e", "sessions": 482,  "share": 0.05743},
    {"project_ref": "d6de2e6f059eb485", "sessions": 216,  "share": 0.02573},
    {"project_ref": "c56cabc8c1f4b974", "sessions": 91,   "share": 0.01084},
    {"project_ref": "557bda4cc2181730", "sessions": 58,   "share": 0.00691},
    {"project_ref": "8ab30e9dab447af3", "sessions": 50,   "share": 0.00596},
    {"project_ref": "45de70d31f768901", "sessions": 44,   "share": 0.00524},
    {"project_ref": "863b5d10dd33e07f", "sessions": 33,   "share": 0.00393},
    {"project_ref": "986f2e5881ca1703", "sessions": 31,   "share": 0.00369},
    {"project_ref": "8e2208edc21154d9", "sessions": 31,   "share": 0.00369},
    {"project_ref": "ca1c1ef301d1efef", "sessions": 31,   "share": 0.00369},
    {"project_ref": "77365f6667aedc51", "sessions": 31,   "share": 0.00369}
  ]
}
```

The Gini coefficient of **0.8321** puts this distribution somewhere between the
income concentration of a kleptocracy and a textbook power-law tail — Gini
above 0.8 is roughly what you see for citation counts of academic papers, for
GitHub repository stars, or for Twitter follower counts. Whatever a project
is to this user, the choice of which project to work on is not anything close
to uniform. It is a savagely top-heavy decision distribution.

## What concentration this severe actually means

Three numbers describe the shape better than the Gini does on its own:

**Top 1 share = 31.8%.** Nearly one in three of all sessions in this 8,393-row
dataset went to a single project. For perspective, the second project holds
16.0% — exactly half the share of the leader. The third project holds 7.5%,
exactly half again. The top three projects follow an almost perfect 4:2:1
ratio, which is a textbook Zipf-1 head: the rank-k frequency is approximately
proportional to 1/k. After rank 3 the curve flattens but remains heavy.

**Top 5 share = 67.2%.** Two thirds of *all* sessions touched only five
distinct project roots. If you wanted to predict where a new session would
land, predicting "one of the top five" would be right two thirds of the time
without knowing anything else.

**Top 10 share = 72.7%.** Note how little is added between top-5 and top-10:
the marginal contribution of projects ranked 6 through 10 is only 5.5
percentage points combined, an average of 1.1 pp each. The top of the
distribution is brutally short — a head of 5, a shoulder of 5 more, and then
a 1,310-project tail.

The 1,310-project tail is the surprise. With 8,393 sessions and 1,320 projects,
the mean sessions-per-project is 6.36. The median sessions-per-project is 1.
Most projects in this dataset got recorded for a single session. They are
one-time cd-into-a-directory events that the session recorder caught and the
human never returned to. They might be quick experiments, downloaded
repositories briefly inspected, scratch directories, demo clones — projects
that exist as a session-queue entry and as nothing else in the user's
attention.

## The 4:2:1 head and what Zipf usually means

When the top-3 of a distribution falls in a 4:2:1 ratio, you are almost
always looking at a Zipf-like distribution where rank-k frequency goes as
1/k^s with s close to 1. For this dataset the top-three sessions are
2667, 1347, 634 — call them ~2700, ~1350, ~675 and the 4:2:1 is exact to
within rounding. The 4th-ranked project (513) and 5th-ranked (482) break the
pattern slightly — they cluster near each other rather than continuing the
halving sequence — which suggests the top-3 are categorically different from
the rest. They are the user's *primary* working contexts; the 4th and 5th
are secondary but stable; everything below rank 5 is sporadic.

Zipf distributions arise naturally from preferential-attachment processes:
projects you have already worked in are more likely to attract your next
session, because attention has switching costs. The longer you have spent
in a project, the more context you carry forward, the cheaper the next
session in that same project is, the more likely you are to choose it
again. This is the same dynamic that produces Zipf in citations, web
traffic, and book sales — popular things get more popular at a rate
proportional to current popularity.

The shape of the project-session distribution is then evidence that the
user is being economically rational about context-switching costs. They
return to known contexts more often than they explore new ones, in a ratio
that obeys an attachment law.

## Where the long tail comes from and why it matters

If 1,320 distinct projects appear in 8,393 sessions, and 5 projects account
for 5,643 of those sessions (67.2%), then the remaining 1,315 projects share
the remaining 2,750 sessions. A back-of-envelope split: of those 1,315 tail
projects, the count of singleton projects (projects with exactly one session)
is large — likely around 800-900 based on the rate of decay. These singletons
are not a defect; they are the dataset's record of every place the user
briefly touched.

This matters in three ways:

1. **Storage and indexing.** Any system that aggregates by project_ref pays
   a fixed cost per distinct project. With 1,320 distinct project_refs, the
   per-project overhead — a row in a projects table, an entry in a cache,
   an index bucket — is non-trivial. A lot of that overhead is being spent
   on projects with one or two sessions.
2. **Rollups and dashboards.** Any "top projects" leaderboard converges
   after the top 5 — there is essentially no information in showing ranks
   6 through 50 because they all hover near 1% or below. A dashboard that
   pretends to show "top 20 projects" is mostly showing noise.
3. **Privacy and footprint.** The session queue is a complete record of
   every working directory the user has touched on this device. If the
   project_ref is a hash of an absolute path, the long tail is also a
   privacy footprint — every clone, every demo, every scratch dir is
   recorded as a distinct hash. An auditor reading this file does not
   know what's behind each hash, but they know how many distinct project
   contexts existed.

## The Gini number on its own

Gini = 0.8321 is the most concentrated distribution I have computed from
this dataset so far. For reference, prior posts have measured:

- Token concentration across models: top model held 79% of cost.
- Source concentration across sources: top source about 60%.
- Bucket-density percentile concentration: top decile held 60% of buckets.

Project concentration at 0.83 sits at the high end of those, and unlike
those metrics it is computed across more than 1000 buckets rather than 3-10.
Gini values are sensitive to the number of categories — distributions with
many categories naturally trend higher because there is more room for
inequality. So the 0.83 figure should not be compared 1:1 with the model
or source Ginis. But within the family of long-tail distributions over
many categories, 0.83 is high. It is well into "small handful of dominant
categories plus a heavy tail" territory and not anywhere near
"distributed-evenly-across-projects" territory.

## What the metric does not tell you

The session count per project does not weight by session size. A project
that contains 100 short two-minute sessions is counted as 100; a project
that contains one 15-hour session is counted as 1. The session-weighted
view I just computed is not the same as the duration-weighted view or the
token-weighted view. Those would each tell a different story:

- Duration-weighted concentration would likely be similar (the top
  projects are long-lived) but the rank-1 share might shift.
- Token-weighted concentration would heavily favor projects where the
  agentic loops are longest per session, not where session count is
  highest.

A future post could compute all three concentration measures side by side
and see which decomposition is most informative.

## What I'd watch over time

The single most informative number to track here is the **top-1 share**.
At 31.8% today, it captures how much of the user's attention is consumed
by their primary working context. If it climbs above 40%, the user has
become more focused (or has stopped touching scratch directories). If it
falls below 25%, the user has either started working on more diverse
projects or has been doing more exploratory cd-into-strange-dirs activity.

The second most informative number is the **rank-1 to rank-2 ratio**,
currently 2667 / 1347 ≈ 1.98, almost exactly 2.0. A perfect 4:2:1 head
implies a Zipf-1 process (s ≈ 1). If the rank-1/rank-2 ratio drifts above
2.5, the head is steepening — the leader is pulling away. If it drifts
below 1.5, the leader is being challenged by a second focus area. Either
way, that ratio is a leading indicator of focus-area shift before the
leader actually changes.

## A small confession about session_count vs project_count

When I first ran this, I was surprised that 8,393 sessions distributed
over 1,320 projects had a Gini as high as 0.83. The intuition that pulls
me toward a lower Gini comes from "the average project has 6.36 sessions,
that doesn't sound that uneven." The intuition that gets me to the right
answer is the *median* sessions-per-project, which is 1. Mean ≫ median
is the universal signal for a heavy-tailed distribution, and when mean
is 6× the median you should expect Gini in the 0.7-0.9 range.

This is the kind of metric where median and mean disagree by enough to
flip your priors about what the distribution looks like. The right move
is always to compute both and trust the median's read on shape.

The data is in the queue. The metric is one group-by and a sort. The
asymmetry it reveals about how this user's attention is allocated across
contexts is the most concentrated signal in the file.
