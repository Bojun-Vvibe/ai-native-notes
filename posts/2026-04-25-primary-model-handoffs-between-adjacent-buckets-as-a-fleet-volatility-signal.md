# Primary-model handoffs between adjacent buckets as a fleet volatility signal

There is a class of metrics in agent-fleet observability that nobody asks
for until they save you. Bucket-handoff frequency is one of them. The
question it answers is small and slightly weird: in your time-bucketed
token stream, how often does the model that owns the largest share of
tokens in bucket *t* differ from the model that owns the largest share
in bucket *t+1*? Phrased that way it sounds like a curiosity. In
practice it is one of the cleanest possible signals for "is your
routing layer doing work, or is it pinning on one model and pretending
to be a router".

This post unpacks the metric, walks through the live numbers from the
local pew-insights `bucket-handoff-frequency` subcommand against my own
sync queue, and shows what the two interesting failure modes look like
when you read the table the right way.

## The shape of the data

The input is the same hour-bucketed (here: half-hour-bucketed) token
table every other pew-insights subcommand consumes. Each row is keyed
on `(source, model, hour_start)` and carries a token sum. The
bucket-handoff lens collapses model out of the key by picking, per
bucket, the model with the maximum token count — call that the
*primary model* for the bucket. Ties are broken lexicographically so
the lens stays deterministic, which matters more than it sounds: a
non-deterministic primary-picker would let two adjacent ties produce a
spurious handoff, inflating the volatility headline.

After collapsing, you have a single sequence of (bucket → primary
model) pairs. Walking that sequence pairwise gives you the handoff
count: increment whenever the primary in bucket *t+1* differs from the
primary in bucket *t*. Divide by the number of adjacent pairs and you
have the headline percentage.

That is the whole metric. It hides nothing exotic. The interesting
part is what it correlates with.

## Live numbers from my own queue

Here is the actual output of `pew-insights bucket-handoff-frequency` at
2026-04-25T08:52:45Z, run against my real `~/.config/pew/queue.jsonl`:

```
as of: 2026-04-25T08:52:45.952Z
active-buckets: 887    pairs: 886    handoffs: 132 (14.9%)
minHandoffs: 1    topHandoffs: 10
split: 32 contiguous pairs (5 handoffs), 854 gapped pairs (127 handoffs)
stickiest model: gpt-5.4 (primary in 204 of 887 buckets)

top model handoffs (sorted by count desc)
from-model                 to-model                   count  share
-------------------------  -------------------------  -----  -----
claude-opus-4.7            gpt-5.4                    34     25.8%
gpt-5.4                    claude-opus-4.7            34     25.8%
claude-opus-4.6-1m         gpt-5.4                    6      4.5%
gpt-5.4                    claude-opus-4.6-1m         5      3.8%
claude-sonnet-4            gpt-5                      4      3.0%
gpt-5                      claude-sonnet-4            4      3.0%
claude-haiku-4-5-20251001  gemini-3-pro-preview       3      2.3%
claude-opus-4-7            gpt-5.4                    3      2.3%
claude-sonnet-4.5          gemini-3-pro-preview       3      2.3%
gemini-3-pro-preview       claude-haiku-4-5-20251001  3      2.3%
```

Five things in those numbers deserve attention.

### Thing 1: 14.9% is the headline, but it is not the right number

A 14.9% handoff rate sounds modest. The honest read is that across 886
adjacent-bucket pairs, the primary model changed 132 times. If you
stop reading there you walk away thinking the fleet is mostly pinned.

The next line dismantles that read. Of those 886 pairs, only 32 are
*contiguous* (the two buckets are exactly one bucket-width apart). 854
are *gapped* — there is at least one empty bucket between them. The
gapped pairs are doing 96.2% of the handoff work (127 of 132). The
contiguous pairs handed off 5 times out of 32, a rate of 15.6% — almost
identical to the headline.

The reason the two rates land in the same neighbourhood is that the
adjacency structure of this workload is heavily dominated by gaps. In
other workloads (a single long-running session, a high-frequency batch
job, a chat-only fleet) the contiguous-pair rate diverges sharply from
the gapped-pair rate, and the divergence is the part that matters.
Reporting only the global percentage would hide it.

### Thing 2: the top two pairs are mirror images and they are 51.6% of all handoffs

`claude-opus-4.7 → gpt-5.4` and `gpt-5.4 → claude-opus-4.7` each show
up 34 times. Together they account for 68 of 132 handoffs, or 51.6% of
all volatility in the system. The fleet is not a fleet. It is a
two-model duopoly with occasional appearances by other models, and the
"router" — to the extent there is one — is just oscillating between
the two dominants depending on which session is currently active.

This is exactly the kind of finding you cannot get from a token-mix
pie chart. The pie chart shows you that opus and gpt-5.4 dominate by
volume; it does not show you that they dominate by *taking turns*. The
distinction matters because turn-taking implies the fleet can survive
a single-model outage (the other carries the load) where pure
single-model dominance cannot.

### Thing 3: the stickiest model is not the largest

`gpt-5.4` is primary in 204 of 887 buckets — that is 23.0% of buckets,
not anywhere near a majority. By contrast, the bucket-intensity lens
will tell you that `claude-opus-4.7` carries far more *tokens* per
active bucket than gpt-5.4 does (4.7B-class total vs gpt-5.4's
roughly 1B-class), so the token-volume leaderboard and the
bucket-primary leaderboard disagree.

The reconciliation is that opus is dense — when it is active it pours
tokens — but gpt-5.4 is *frequent*: it shows up in more buckets, often
as the primary by token-count when opus is dormant. The handoff lens
exposes this by counting buckets, not tokens. A model that fires for
one bucket every hour and disappears looks like a high-stickiness
primary even if its lifetime token total is modest.

### Thing 4: the long tail is multi-vendor

Beyond the opus/gpt-5.4 axis, the table shows handoffs into and out of
`gemini-3-pro-preview`, `claude-haiku-4-5-20251001`,
`claude-sonnet-4`, and `claude-opus-4.6-1m`. Each of these contributes
3-6 handoffs — small in absolute terms, but they confirm that the
fleet is genuinely multi-provider rather than a Claude-only deployment
with a token AOI exception.

The presence of `claude-opus-4-7` as a separate row from
`claude-opus-4.7` (note the dash vs dot in the version suffix) is a
known pew-insights normalisation gap — different surfaces report the
opus version slightly differently and the model-name normaliser only
collapses some of them. That is a separate story; for this lens the
honest thing to do is leave the rows split because folding them
silently would conceal the source of the volatility.

### Thing 5: the dropped-counts row is the integrity check

`dropped: 0 bad hour_start, 0 zero-tokens, 0 by source filter, 0
empty-model buckets, 0 below min-handoffs, 23 below top cap`. The 23
"below top cap" entries are the long-tail handoff pairs that exist
once or twice in the data and were squeezed out by `--top-handoffs
10`. The other zero-counts confirm the sweep was clean — no
ill-formed timestamps, no empty buckets that would have produced phantom
handoffs, no models filtered out implicitly. Whenever you publish a
volatility number, publishing the integrity-check row underneath it is
how you keep yourself honest.

## What this metric is good for

The reason I keep coming back to this lens is that it answers three
operational questions that ordinary token charts answer badly.

**"Is my router actually routing?"** A router worth its name should
produce a handoff rate that tracks the diversity of incoming tasks. If
your task mix is genuinely mixed (code, chat, summarisation, image
analysis) and your handoff rate sits below 5%, your router is pinning.
If it sits above 50% in a workload with stable task mix, your router
is thrashing — probably round-robining or hash-bucketing on a key that
does not correlate with the right model. The 14.9% number above is in
the healthy band for a workload where opus owns code and gpt-5.4 owns
fast-turnaround turns; if the same workload produced 60% I would start
auditing the routing logic.

**"What is my single-model-outage radius?"** If two models account for
51.6% of handoffs as mirror images, an outage of either of them
collapses the handoff structure to a near-zero rate (whichever model
remains becomes primary almost everywhere). That is a different kind
of risk than total token-volume risk. The bucket-handoff number is
the only place this risk is legible without reconstructing the
sequence by hand.

**"Is the workload changing under me?"** Compute this metric over a
rolling 7-day window and watch the value. A drift from 15% to 25%
without a code change is a workload shift — probably a new caller, a
new task class, or a model deprecation that is forcing failover paths.
A drift from 15% to 5% is consolidation — possibly desirable, possibly
the symptom of one model swallowing all the work because the others
are silently rate-limited.

## How to read the contiguous/gapped split

The contiguous/gapped distinction is the part of the lens that took
me longest to get right. Naively you treat every adjacent pair the
same way, but a one-bucket gap and an eight-bucket gap are not the
same thing.

A *contiguous* pair (gap = 0) tells you about within-session model
swaps. If the same caller is hitting the proxy continuously and the
primary model changes between bucket *t* and *t+1*, something inside
the routing decision changed during that bucket. Maybe a function-call
result triggered a different model, maybe a long-context request
spilled to a higher-context tier, maybe a rate limit forced fallback.

A *gapped* pair (gap ≥ 1) tells you about between-session model
shifts. The previous active bucket ended, the fleet went quiet, and
when traffic resumed the new session happened to land on a different
primary. That is not really a "handoff" in the operational sense —
nothing flipped under load — but it is still a fleet-level change of
who-runs-things.

Reporting both rates separately lets the reader pick the one that
matches their question. Reporting only the global rate forces a wrong
answer when the gap distribution is skewed.

## The deterministic-tie-breaking guarantee

Tie-breaking deserves a paragraph because it is where this lens earns
its right to be trusted. If two models tie for primary in bucket *t*,
which one wins matters: a non-deterministic pick can produce a
spurious handoff out of pure noise.

The pew-insights implementation breaks ties by picking the
lexicographically-smaller model name. That is arbitrary but
deterministic — `claude-opus-4.7` beats `gpt-5.4` not because it
should but because `c < g`. The consequence is that ties never
contribute to the handoff count *unless they are followed by a non-tie
that flips the leader*. In a workload with many ties this would
distort the headline; in this workload it does not, because exact
half-and-half token splits in a single bucket are vanishingly rare
when the bucket is half an hour wide.

If you ever build your own version of this metric, the temptation
will be to "do the right thing" and average over tied possibilities.
Resist it. Determinism beats fairness here. You want to be able to
re-run the lens tomorrow and get the same number, and tied-pair
averaging produces drift the moment a token count shifts by a single
unit.

## What it does not tell you

Three honest limitations.

It does not tell you about latency — a handoff between models with
wildly different time-to-first-token will look identical to a handoff
between two equally-fast models. Pair this with the
time-to-first-meaningful-output lens if latency is what you care about.

It does not tell you about cost — a handoff from a 10x-cheaper model
to a 10x-more-expensive one shows up as one event, the same as the
reverse. Pair with the cost-attribution lens.

It does not tell you about *why* the handoff happened. The lens is
post-hoc and structural. Causal attribution requires per-call
metadata the queue does not carry.

## A note on bucket width

Everything above assumes the bucket width is the same across the whole
window. If you change bucket width — say, from 1h to 30m — the absolute
handoff count will rise even if nothing about the underlying routing
changed, because finer buckets give the primary more chances to flip.
The fix is to report bucket-width alongside the headline (the
implementation does: it is implicit in the `active-buckets: 887` count
relative to the time span). When comparing two windows always normalise
to the same bucket width before you trust the percentage.

## Closing

The metric is shallow. The interpretation is not. A single number —
"14.9% of adjacent buckets see a primary-model handoff" — opens onto
five distinct operational questions: is the fleet a duopoly, is the
router pinning, is the stickiest model the largest, is the workload
multi-provider, and what does the dropped-counts row look like. None
of those questions are answered by a token-volume pie chart. All of
them are answered cheaply by a single linear pass over the bucketed
queue.

Build the metric. Add it next to your token totals. Watch what it
does over time. The first time it drifts unexpectedly will be the
first time you catch a workload shift before it becomes a
post-incident review.
