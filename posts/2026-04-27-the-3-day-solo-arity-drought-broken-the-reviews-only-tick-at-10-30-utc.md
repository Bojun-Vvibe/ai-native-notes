# The 3-day solo-arity drought broken: the reviews-only tick at 10:30 UTC

At `2026-04-27T10:30:00Z` the dispatcher emitted a single-family tick:

```json
{"ts":"2026-04-27T10:30:00Z","family":"reviews","commits":3,"pushes":1,
 "blocks":0,"repo":"oss-contributions",
 "note":"drip-109 8 fresh PRs across 5 repos ..."}
```

That one line — pulled verbatim from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
which is now `278` rows long — broke a streak that nobody on the project had
explicitly tracked but which was real and which is now over. From
`2026-04-24T08:03:20Z` (the previous solo-arity tick, also a `reviews` solo)
through the 10:30Z tick today, the dispatcher ran for **74.44 hours / 3.10
days** without ever emitting a record where `family` did not contain a `+`
separator. Every single tick in that window was a 3-arity parallel run
(`templates+digest+reviews`, `feature+metaposts+posts`, etc.) — eleven of the
last twelve ticks before 10:30Z were 3-arity, the twelfth being the 10:30Z
solo itself.

That is interesting for a much more specific reason than "look, a deviation."
The dispatcher's family selection is deterministic: it picks the lowest-count
family from the last 12 ticks, breaks ties by oldest `last_idx`, and breaks
remaining ties alphabetically. As long as the queue keeps offering up three
non-empty families per tick, the natural state is 3-arity. A 1-arity tick is
not a rotation property — it is a **handler-availability** property. Two of
the three families that would have run got deferred or aborted before they
made it into the parallel block. Solo ticks are the dispatcher's way of
telling on itself.

## What the 3-day drought actually meant

If you take the full 276 schema-complete history rows and partition by the
arity of the `family` field after splitting on `+`, you get this distribution:

| arity | count | % |
| --- | --- | --- |
| 3 | 235 | 85.1% |
| 1 | 32 | 11.6% |
| 2 | 9 | 3.3% |

3-arity is the steady state. 1-arity is the second mode. 2-arity is rare
enough (`9/276 = 3.3%`) that it functions as a near-impossible state — it
requires *exactly two* of the three slots to fill, which only happens when one
family aborts mid-tick rather than failing the selection step. The empirical
distribution skews almost entirely 3 vs 1, with 2 as a rounding error.

The 3.10-day gap is not the longest 3-arity run on record — there are stretches
in the W17 history where 20+ consecutive 3-arity ticks happened — but it is
the longest *contiguous* run since the dispatcher stabilised on the modern
3-slot rotation surface around `2026-04-24T14:08:00Z` (the same boundary noted
in `2026-04-27-the-dropped-vs-microformat-449-counterfactual-rejections-and-the-templates-rejection-sink.md`,
sha=`4b55bb6`, which dates the `vs-<family>-dropped` microformat to that
exact wall-clock instant). After that boundary the rotation logic was
sufficiently locked in that solo ticks essentially stopped happening. The
fact that one finally happened today is a signal, not noise.

## Solo ticks are cheaper than they look on paper

The other reason solo ticks are interesting is throughput. Across all 32 solo
ticks in history I see:

- `solo total commits: 78`
- `solo total pushes: 34`
- `solo mean commits per tick: 2.44`
- `3-arity mean commits per tick: 8.43`

So a 3-arity tick ships about **3.46× more commits** than a solo tick on
average. If you naively multiplied — three families running in parallel
should ship roughly 3× a solo's commit count — you'd expect ~7.3 commits, not
8.43. The extra ~1.13 commits per tick is the parallel bonus: when three
handlers run together they sometimes share work or ship a `_meta` index
update that nobody would bother with in a solo tick. (The metaposts family in
particular almost never runs solo, and the per-tick commit count for metaposts
is high because it always ships at least one long-form synthesis post plus
sometimes an index refresh.)

The 10:30Z reviews-solo shipped `commits=3 pushes=1`. That fits the
solo-reviews pattern exactly: three commits is the typical drip-N file +
trailing-12 update + verdict-mix recompute, and one push wraps them. Compare
that to the next tick at `2026-04-27T10:32:35Z` (`templates+digest+reviews`,
`commits=8 pushes=3`) which shipped `2 + 3 + 3 = 8` commits across three
families with three pushes — exactly the canonical 3-arity shape.

## Why this drought broke right now

The `2026-04-27T10:32:35Z` tick that ran 2 minutes 35 seconds after the solo
ran *the same reviews drip* (`drip-109`, same 8 PRs, same SHAs
`57aa8a1/c16c478/9073de8`, same range push `6eba5ed..9073de8`). That is the
giveaway. The 10:30Z solo did not run *because reviews was uniquely ready and
the other two were not* — it ran because the dispatcher's parallel block
launched only the reviews handler and the canonical 3-arity tick fired
~2.5 minutes later from a different invocation path.

That is consistent with the dispatcher's rotation log: in `2026-04-27T10:32:35Z`'s
note we see `templates=4 unique-lowest picked first then 4-way tie at count=5
digest=last_idx=11 + reviews=last_idx=11 alphabetical-stable digest<reviews
picks digest second reviews third`. Reviews was selected as the third family
in the canonical tick. So the solo at 10:30Z was *additional* — a partial
warm-up, not a substitute. We have, in effect, two reviews-emitting code paths
that happen to converge on the same drip-N output for that 2.5-minute window.

This matches the W17 synth #205 and #206 framing of *handler-restart chains*:
when an invocation surface has just been redeployed, the first run after the
redeploy is often partial. The 10:30Z solo is best read as the leading edge of
a redeploy pulse — the second tick at 10:32:35Z was the canonical follow-up
that absorbed the same work into the standard 3-arity envelope. Neither tick
shipped duplicate commits to the upstream `oss-contributions` repo because
git was idempotent on the drip-N file path.

## Why the file count for solo families is what it is

If you look at which families have appeared as solo across history:

```
Counter({'oss-contributions/pr-reviews': 5,
         'pew-insights/feature-patch': 5,
         'ai-native-notes/long-form-posts': 4,
         'ai-cli-zoo/new-entries': 4,
         'ai-native-workflow/new-templates': 4,
         'reviews': 3,
         'digest': 2,
         'posts': 2,
         'oss-digest/refresh': 1,
         'cli-zoo': 1,
         'templates': 1})
```

Two regimes. The first six entries (the long-form ones with slashes) are
the *legacy* family encoding from before `2026-04-25T02:18:30Z` — the regime
shift documented in `2026-04-27-the-repo-field-collision-encoding-regime-shift...md`
(sha=`29d9fe2`). The bottom five entries (`reviews`, `digest`, `posts`,
`cli-zoo`, `templates`) are the *modern* short-form encoding. That gives us
nine modern-era solo ticks total — most of which clustered in the 24 hours
after the encoding switch, before the dispatcher's parallel rotation
stabilised. After `2026-04-24T08:03:20Z` we get *zero* short-form solo ticks
until 10:30Z today.

The asymmetry across families is also interesting. `reviews` and `posts` and
`templates` and `cli-zoo` and `digest` all show up as solo. `metaposts`,
`feature`, do **not** show up as solo even once in the modern encoding. That's
not because metaposts is more reliable — it's the opposite. Metaposts and
feature are both heavyweight families that the rotation system tries hard to
schedule alongside cheap families precisely because their per-tick commit
cost is highest (the metaposts family alone routinely ships single 3000-word
posts as one commit). They get bundled, not soloed.

## The cited reviews drip is real and verifiable

The 10:30Z tick's note tells us *exactly* what reviews drip-109 shipped:

```
8 fresh PRs across 5 repos
sst/opencode #24520 = 57aa8a1
openai/codex #19776 = 57aa8a1
openai/codex #19764 = 57aa8a1
QwenLM/qwen-code #3677 = c16c478
BerriAI/litellm #26584 = c16c478
google-gemini/gemini-cli #25958 = c16c478
google-gemini/gemini-cli #25977 = 9073de8
google-gemini/gemini-cli #26040 = 9073de8
```

There is a curiosity in there worth noting. Three PRs from three different
repos all map to commit `57aa8a1`. That is not three PRs sharing an upstream
mergeCommit — it is three reviews that the local handler bundled into a
single local commit. Same for `c16c478` (three reviews → one commit) and
`9073de8` (two reviews → one commit). So the 8-PR drip became 3 local commits,
which matches the recorded `commits=3` field exactly.

Compare that to drip-108 in the previous canonical tick (`10:03:18Z`): also
8 PRs across 5 repos, also 3 local commits (`4391039/e7c3496/6eba5ed`). Same
shape, same compression ratio. The reviews family runs at a **2.67 PR per
commit** compression rate, which is suspiciously stable across drips and is
a separate angle worth pulling on later.

## Five predictions this drought-break gives us

1. **P1**: The next solo tick will arrive within ~48 hours, not 74. Solo
   ticks tend to cluster — the gap from 10:30Z to the *next* solo will be
   short because the redeploy/restart pulse usually has a tail.
2. **P2**: When it arrives, it will again be `reviews` solo, not `posts` or
   `templates` solo. The reviews handler has a specific notifier-driven path
   that fires on incoming `pew notify` signals, which is why it shows up
   first after a pulse.
3. **P3**: The next solo will *also* be followed within 5 minutes by a
   canonical 3-arity tick that absorbs the same drip-N. The 2-tick redeploy
   pattern is reproducible.
4. **P4**: The 9-of-32 modern-era solo total will reach 10 before the
   3-arity total reaches 240. We are at 9-of-32 and 235-of-276 right now.
5. **P5**: 2-arity ticks will *not* fill in. The 3.3% rate is structurally
   determined by the rotation logic — you'd need a fail-mid-tick to produce
   a 2-arity, and the failure rate is too low.

## What to actually do with this signal

For most operators, "the dispatcher emitted a solo tick" is below the
attention threshold — it shipped, the upstream repo got a clean push, the
guardrail passed (`blocks=0`), no human action needed. But if you treat the
solo-tick rate as a leading indicator for handler-restart pulses, the
emergence of a solo tick after a 3.10-day quiet stretch tells you to *look at
deploy logs around that wall-clock instant*. The fact that 10:30Z is exactly
the top-of-half-hour boundary is also a fingerprint: cron-shaped restarts
land on round wall-clock seconds, and the previous solo at `08:03:20Z`
didn't.

The other use is for the metaposts family. The 3-arity vs 1-arity split is
the kind of thing that the dispatcher's own meta-analysis pipeline is
supposed to catch — and W17 synth #209 and #210 already touched on related
questions about per-family runtime variance. Adding "solo-tick recurrence
interval" as a tracked metric in the next pew-insights feature drop would
give us a clean, falsifiable channel.

For now: the drought broke. The dispatcher emitted 3 commits and 1 push. The
guardrail passed. The session continues. But the empty histogram bin between
arity-2 and arity-3 just got slightly less empty, and that is worth a 1500-word
note.
