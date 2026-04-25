---
date: 2026-04-26
tags: [pew-insights, handoff-latency, source-rotation, openclaw, opencode, multi-agent]
---

# Inter-source handoff latency: the 0.5h median and the openclaw/opencode twins

There is a metric I have been quietly waiting to have, and now I have it:
the wall-clock time between the moment one CLI tool stops being primary
in an hourly bucket and the moment a different CLI tool takes over.
`pew-insights inter-source-handoff-latency` materialises exactly that —
not "how often I switch tools" (that is the handoff *count* lens), and
not "which tools follow which" (that is the *transitions* matrix), but
the *latency* between the two adjacent buckets that disagree on the
primary source. The number that came back surprised me, and the shape
of the surprise is worth pulling apart.

## The headline

```
pew-insights inter-source-handoff-latency
as of: 2026-04-25T19:47:53.421Z    active-buckets: 909    pairs: 908
handoffs: 99 (10.9%)
latency: median 0.50h    mean 6.52h    min 0.50h    max 292.00h
contiguous-handoffs (1h): 1    gapped: 98
dominant source: vscode-copilot (primary in 312 of 909 buckets)
```

909 active hour-buckets across the queue, 908 adjacent pairs, and only
99 of those pairs are handoffs — a 10.9% handoff rate. That is the
first number that is not what naive intuition would say. With six
distinct CLI sources in rotation (`claude-code`, `codex`, `hermes`,
`openclaw`, `opencode`, `vscode-copilot`), and a workday that visibly
juggles between them in the moment, you might expect handoffs to be
the *common* case and same-tool stretches to be the exception. The
data says the opposite: 89.1% of consecutive active buckets keep the
same primary source. The tools cluster, they don't interleave.

## Median 0.5h, mean 6.52h — the gap-vs-contiguous split

The gap between median (0.5h) and mean (6.52h) is the second number
that matters. A 13× median-to-mean ratio is a strong tail signal.
The breakdown explains it: of the 99 handoffs, exactly **1 is
contiguous** (the next active bucket is the immediately following UTC
hour, latency = 0.5h, which is the bucket grain) and **98 are gapped**
(the next active bucket is some number of hours later, anywhere from
0.5h up to 292h — i.e. a 12-day idle stretch). The "contiguous-handoffs
(1h): 1" line is doing a lot of work: it tells us that the *vast
majority* of source rotations are not "I closed `claude-code` and
opened `opencode` in the next hour." They are "I closed `claude-code`,
walked away, came back later — possibly hours, possibly days — and
when I came back I happened to start with a different tool."

That changes the interpretation of "handoff" in a real way. If 98 of
99 handoffs are gapped, then handoffs are mostly **session boundaries
re-cast as tool boundaries**. The handoff metric is partially a proxy
for "did I take a break long enough that my tool choice on return is
effectively a fresh draw from the source distribution?" The
contiguous handoff — the one — is the only case where a tool was
actively swapped without a break. That is the rare event.

## The openclaw/opencode twin

The top of the handoff table is dominated by one pair, in both
directions:

```
from-source     to-source       count  share-of-handoffs  median-latency
--------------  --------------  -----  -----------------  --------------
openclaw        opencode        23     23.2%              0.50h
opencode        openclaw        22     22.2%              0.50h
```

Combined: **45 of 99 handoffs (45.4%) are between `openclaw` and
`opencode`**, and the directional split is essentially 1:1 (23 vs 22).
Both directions have a median latency of 0.5h — meaning when this
pair swaps, it swaps in the very next bucket. They are not handing
off across breaks; they are handing off in real time. This is the
signature of two tools being used **as a doublet** in the same work
session. (This is not the source-pair *cooccurrence* lens, which
asks whether two sources are both active in the same bucket — that
was the subject of the openclaw/opencode-doublet post earlier in the
day. This is the *sequential* lens: even when only one is primary at
a time, the next-primary is the other one, repeatedly.)

The third-largest handoff is `claude-code → openclaw` at 10 (10.1%).
Add the four pairs involving openclaw, opencode, claude-code: 23 + 22
+ 10 + 9 + 9 + 7 = 80 handoffs out of 99. **80.8% of all source
handoffs in the entire queue happen inside a 3-tool subset.** The
remaining 19 handoffs spray across 4+ tail pairs.

## The 292-hour outlier

`max 292.00h` is the tail. 292 hours is 12.17 days. The table shows
it lives in the `claude-code → vscode-copilot` row (max 292.00h vs
median 15.50h vs min 5.00h on a count of 3 — wide). What that almost
certainly is: a `claude-code` bucket from one work-week, then no
active buckets at all for ~12 days, then `vscode-copilot` as the next
active bucket. With `vscode-copilot` having the longest tenure of any
source (`firstSeen 2025-07-30`, per the `source-decay-half-life` lens
on the same snapshot), it is the natural "background" tool that
re-emerges when nothing else has been active. The handoff statistic
treats that as a single transition, but it is really a 12-day silence
sandwiched between two unrelated work episodes. The mean of 6.52h is
heavily inflated by exactly this kind of long-gap re-emergence.

## Why "contiguous-handoffs (1h): 1" is the most interesting cell

I want to dwell on the n=1 contiguous handoff. In 909 buckets of
real activity, with 99 source handoffs, only **one** instance exists
where the source changed *and* the next active bucket was the very
next UTC hour (no gap). That is the rate at which I actually swap
tools without a break. Roughly 1 in 909 hours of activity, or 1 in
99 handoffs. Effectively zero. Tool-swapping in this queue is an
**asynchronous** behaviour, not a synchronous one. There is no
"and then I switched to a different agent for the next problem" in
the data — there is "I stopped, life happened, I started again, and
the second start happened to use a different agent."

This has a secondary implication: the cost of a handoff in this queue
is not measurable by latency alone, because the latency is mostly
break-time, not switching-time. The actual *cognitive* cost of moving
from `openclaw` to `opencode` (or back) — re-orienting context,
re-pasting state, re-confirming permissions — is invisible to this
metric, because by the time the new tool's first bucket fires, the
operator has already done the swap mentally during the gap.

## Cross-citing today's other lenses

The 909 active buckets here is the same population that the
`bucket-density-percentile` and `tail-share` lenses partition by
mass. The 312-bucket dominance of `vscode-copilot` (primary in
312/909 = 34.3% of buckets) is itself a striking number when you
recall that `vscode-copilot` accounts for only **0.022%** of total
tokens (1,885,727 of 8,764,798,864 per the source-decay-half-life
output of the same snapshot). It is primary in a third of all
buckets while contributing essentially zero mass. That is the
"editor-assistant paradox" from the tail-share post: tiny per-bucket
intensity, but enormous bucket-presence. The handoff matrix corroborates
that picture — `vscode-copilot` shows up as a handoff endpoint
mostly through long-gap re-emergence (`vscode-copilot → claude-code`
median 21.5h with min 0.5h, max 116h), not as part of the
real-time openclaw/opencode rotation.

## What the metric is *not* telling us

Three things this lens cannot answer, which would require companion
metrics:

1. **Within-bucket source mix.** The "primary" is the max-tokens
   source per bucket, ties broken lexicographically. A bucket where
   `openclaw` does 51% and `opencode` does 49% is a primary-`openclaw`
   bucket and contributes nothing to handoff counts even though both
   tools were heavily used. The cooccurrence lens covers that gap.
2. **Sub-hour swaps.** The bucket grain is 30-minute hour_starts
   (the data shows half-hour `hour_start` values like
   `2026-04-25T19:30:00.000Z`). Swaps faster than that are invisible.
   Given that `pew` itself emits per-snapshot rows, a finer-grain
   handoff lens is buildable.
3. **The reason for the swap.** A handoff from `claude-code` to
   `codex` (9 occurrences, 9.1% share, median 0.5h, max 112.5h) could
   be a deliberate routing decision (different problem class) or an
   outage workaround (one provider was rate-limited) or a UX whim.
   The metric is shape-only.

## Operational reading

If 80.8% of handoffs happen inside the openclaw/opencode/claude-code
triangle, then any tool-orchestration heuristic that focuses on those
three pairs covers the overwhelming majority of cases. The remaining
~20 handoffs are scattered across pairs that occur 1–3 times each in
~209 hours of openclaw tenure, ~125 hours of opencode tenure, and
~1716 hours of claude-code tenure (per the source-tenure lens) — i.e.
they are noise.

The 0.5h-median, 6.52h-mean, 292h-max distribution argues against
treating "handoff latency" as a smooth metric. It is bimodal: one
mode at the bucket grain (the active openclaw↔opencode rotation), and
a long tail of break-induced gaps. A useful future split is:
"contiguous handoff" vs "gapped handoff" as two separate metrics,
because they are measuring different phenomena that the current lens
collapses into one number.

## Data

- **Snapshot:** `as of 2026-04-25T19:47:53.421Z`
- **Source:** `pew-insights inter-source-handoff-latency` against
  `~/.config/pew/queue.jsonl`
- **Population:** 909 active buckets, 908 adjacent pairs, 99 handoffs
  (10.9% handoff rate).
- **Latency:** median 0.50h, mean 6.52h, min 0.50h, max 292.00h.
  Contiguous (1h next-bucket): 1. Gapped: 98.
- **Top pair:** `openclaw ↔ opencode`: 23 + 22 = 45 of 99 handoffs
  (45.4%), both directions median 0.50h.
- **Triangle dominance:** openclaw / opencode / claude-code pairs
  account for 80 of 99 handoffs (80.8%).
- **Dominant primary source:** `vscode-copilot` (312 of 909 buckets,
  34.3%) despite contributing 0.022% of total tokens
  (1,885,727 / 8,764,798,864 from the source-decay-half-life lens on
  the same snapshot).
- **Cited tooling:** `pew-insights` v0.6.1 (CHANGELOG entry dated
  2026-04-26, repo `~/Projects/Bojun-Vvibe/pew-insights/`).
- **Cross-citation:** the `vscode-copilot` low-mass / high-presence
  behaviour matches the editor-assistant tail-share observation in
  `posts/2026-04-26-tail-share-and-the-editor-assistant-paradox-...md`
  on the same snapshot population.
