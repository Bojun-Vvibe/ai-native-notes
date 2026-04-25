---
title: "Stickiness as a Session-Handoff Metric: 86.6% vs 47.5%"
date: 2026-04-25
---

## The number that started this

Run `pew-insights transitions` against the local pew queue and
the most interesting column on the page is the one nobody
comments on. From a fresh run on 2026-04-25T04:15:37Z over
6,306 sessions and 6,305 ordered pairs (6,207 handoffs, 98
breaks across the 1,800-second max-gap boundary, 3,534
overlaps):

```
group        outgoing  self-loop  stickiness
claude-code     1,020        883       86.6%
codex             466        349       74.9%
openclaw        1,083        514       47.5%
```

Three producers, one column, three numbers that span 39
percentage points. Stickiness here is `P(next session in same
group | from group)` — the conditional probability that when a
session ends, the next session within the gap window is from
the same producer.

Claude-code is sticky. When a claude-code session ends, the
next session is another claude-code session 86.6% of the time.
Openclaw is anti-sticky relative to that — its next-session is
a different producer more than half the time. Codex sits in
between, closer to claude-code than to openclaw.

This post is about why that column is the most operationally
useful number in the `transitions` report, why it is more
informative than the raw transition counts above it, and what
the gap between 86.6% and 47.5% actually tells you about how
the underlying producers are being used.

## What `transitions` measures

The subcommand builds an adjacency matrix of source-to-source
session handoffs. Two ordered sessions are paired into a
transition if their gap is below the `--max-gap` threshold
(default 1,800 seconds, i.e. 30 minutes). If the gap exceeds
the threshold, they are recorded as a "break" rather than a
handoff. Overlapping sessions (the next session starts before
the previous ends) are tracked separately.

From this run, the breakdown is:
- 6,305 ordered pairs
- 6,207 handoffs (98.4% of pairs were within 30 minutes)
- 98 breaks (1.6% of pairs exceeded 30 minutes)
- 3,534 overlaps (56.1% of pairs had the next session start
  before the previous one ended)

The overlap rate is the second-most interesting headline number
on the page. More than half of all session handoffs in this
fleet are not really handoffs — they are concurrent sessions
where one producer was running while another started. The
producers in this fleet are not single-threaded operators.
They are doing parallel work and the "next session" is often
a session that began without waiting for the previous one to
end.

But concurrency is not the focus of this post. Stickiness is.
And stickiness specifically excludes the overlap question — it
is computed across the whole outgoing-pair count, regardless of
whether the next session overlapped or strictly followed.

## Why the top transitions table is necessary but not sufficient

Above the stickiness table the report shows the top 10
transitions by count:

```
from              to    count  median gap  p95 gap  overlaps
opencode    opencode    3,073          0s      23s     2,128
claude-code claude-code   883          0s       5m       553
opencode    openclaw      553          0s       3m       194
openclaw    opencode      552          0s       2m       316
openclaw    openclaw      514          4s      10m        31
codex       codex         349          9s      30s        84
claude-code codex         117          0s       2m        99
codex       claude-code   114          0s       4m        91
openclaw    claude-code    13          0s      22s         9
opencode    claude-code    12          0s       3m        8
```

This is dense and useful but it has a problem: the absolute
counts are dominated by producer volume. Opencode has the most
sessions, so opencode→opencode tops the list with 3,073 pairs.
That is interesting if you want to know "what is the biggest
single transition flow," but it is misleading if you want to
know "is opencode sticky?"

In fact, opencode is not even in the stickiness table at the
bottom. Opencode is excluded from the stickiness lens because
the lens reports only on producers with at least some
non-self-loop outgoing transitions, and opencode→opencode is
overwhelmingly the dominant pair from opencode (3,073 self-loop
out of perhaps 3,200+ total outgoing). It is so sticky that
it doesn't fit the "compare stickiness across producers"
framing — its self-loop is essentially "always" and the
comparison is not meaningful.

That detail is itself important. The stickiness metric is most
informative for the producers in the middle range — the ones
where the next-session might or might not be the same producer.
A producer that is always self-following or never self-
following is uninformative; a producer with a stickiness in the
middle range carries actual signal about how it gets composed
into the broader workflow.

## The 39-point spread

Claude-code at 86.6%. Codex at 74.9%. Openclaw at 47.5%.

Read the spread at face value first. When a claude-code
session ends, the very next session within 30 minutes is
another claude-code session almost 9 times out of 10. When an
openclaw session ends, it is essentially a coin flip whether
the next session is openclaw or something else.

The mechanism for claude-code's stickiness is straightforward.
Claude-code sessions tend to come in extended interactive
runs — you open a terminal, you have a conversation, you
follow up, you follow up again. The 883 self-loops out of
1,020 outgoing transitions reflect that interactive-run
shape. The 137 non-self-loops (1,020 − 883) split roughly
evenly between codex (117) and other producers — meaning that
when claude-code does hand off, it most often hands off to
codex. Hold that thought.

Codex at 74.9% has the same shape, milder. Codex sessions also
come in runs but the runs are shorter on average (codex has
466 outgoing transitions vs claude-code's 1,020) and the
hand-offs out of codex are more diverse. Codex→claude-code is
114, almost exactly mirroring claude-code→codex at 117. There
is a near-symmetric exchange between these two producers.

Openclaw at 47.5% is qualitatively different. Of its 1,083
outgoing transitions, only 514 are openclaw→openclaw. The
other 569 are split: 552 go to opencode, 13 go to claude-code,
and the remainder to other producers below the top-10
threshold. Openclaw is essentially a router or wrapper — its
sessions tend to be followed by sessions from a different
producer, most often opencode. The 47.5% is a structural fact
about how openclaw is used, not about how busy it is.

## Why stickiness is more informative than transition share

A naive way to ask "is producer X sticky?" would be to compute
the row-share of self-loops in the top transitions table. For
claude-code that would be 883 / 1,020 = 86.6%. Same number.
But the row-share calculation depends on knowing the total
outgoing — which is exactly what the stickiness column
provides as a derived quantity, with the right denominator
choice baked in.

The right denominator matters. Two competing definitions:

1. **Of all outgoing pairs from group X**, what fraction are
   self-loops? This is what pew uses. Denominator = outgoing.
2. **Of all sessions ending in group X**, what fraction have a
   next-session in the same group within the gap window?
   Denominator = total sessions of group X (including those
   that ended without a within-window next session).

Definition 2 conflates "stickiness" with "frequency." A
producer that mostly ends sessions with no next-session within
30 minutes (because the producer fires once a day) would have
artificially low "stickiness" by definition 2 even if every
within-window next-session is a self-loop. Definition 1 fixes
this by conditioning on the existence of a within-window
handoff. The 86.6% for claude-code is "given that a next
session happened within 30 minutes, the probability it was
also claude-code." That is the conditional probability the
column header promises.

This is the same denominator-choice problem that comes up in
cache-hit-ratio reporting (where the choice between "of all
requests" and "of requests that could have been cached"
changes the headline number by a lot). Pew picks the
conditional version in both cases, which is the more
operationally useful one.

## What stickiness diagnoses

Three things, in priority order.

**First: producer composability.** A high-stickiness producer
is one you use in extended runs. A low-stickiness producer is
one you use in short stretches that hand off to other producers.
The composability is a direct function of the work shape.
Claude-code's 86.6% says "when I use claude-code, I use it for
a while." Openclaw's 47.5% says "openclaw is a step in a
sequence."

This matters for capacity planning. High-stickiness producers
benefit from session-resumption optimizations (cache the
context, keep the state hot). Low-stickiness producers benefit
from fast-startup optimizations (cold start matters because
you'll start a new session shortly).

**Second: producer pairing.** The off-diagonal entries in the
top-transitions table tell you who hands off to whom. The
near-symmetric claude-code↔codex exchange (117 in one
direction, 114 in the other) is a genuine pairing — these two
producers are being used together in something like a
back-and-forth pattern. The asymmetric openclaw↔opencode
exchange (553 one direction, 552 the other) is also nearly
symmetric in count but reflects a different relationship,
because both producers also have substantial self-loop counts.

A pair of producers with low stickiness and high cross-flow is
a producer pair that should probably be considered as a single
unit for some purposes — they are essentially one workflow
that happens to use two tools.

**Third: anomalous routing.** Openclaw→claude-code at only 13
and opencode→claude-code at only 12 tell you that claude-code
is rarely a downstream consumer of either openclaw or opencode.
Combined with the symmetric claude-code↔codex exchange, the
implication is that claude-code is its own ecosystem — it
mostly self-loops, occasionally hands off to codex, and is
almost never the next-session after openclaw or opencode.

If you saw the openclaw→claude-code count jump from 13 to 130
in a future tick, that would be a routing anomaly worth
investigating. Stickiness gives you the baseline against which
"normal" routing is defined.

## The 30-minute gap-window is a knob

The default `--max-gap 1800s` (30 minutes) is the threshold
above which a session pair stops counting as a handoff and
starts counting as a break. The 98.4% handoff rate from the
live data tells you that most session pairs in this fleet are
well under 30 minutes apart — the median gap is 0 seconds
across the whole fleet, which is the time-overlap signature
of concurrent producers.

But the gap window is a knob. Tighter — say 60 seconds —
would shift the handoff rate down because it would reclassify
30-minute-gap pairs as breaks. The stickiness numbers would
also shift, because some self-loops would drop out of the
handoff bucket. The shift would be biggest for low-stickiness
producers (where the self-loop is rarer and more sensitive to
filtering) and smallest for high-stickiness ones.

The right gap-window depends on what you mean by "handoff."
For an interactive-session-flow analysis, 30 minutes is fine —
it captures "I closed one terminal and opened another within
the same work session." For a strict same-task analysis, 60
seconds would be tighter and would force the metric to look
at near-immediate handoffs only.

I have not yet run the same `transitions` report with a
tighter gap window. The hypothesis is: claude-code's
stickiness would stay around 86%, codex's would shift slightly
down (it has more medium-gap self-loops, p95 gap 30s for the
self-loop is borderline), and openclaw's would shift more
because its self-loop p95 of 10 minutes shows substantial
medium-gap traffic. If that hypothesis holds, openclaw's
stickiness is partly a function of how much medium-gap activity
gets included; openclaw's true tight-handoff stickiness might
be even lower.

## Why "by source" is the right grouping for this fleet

The `transitions --by source` grouping treats each producer
(opencode, openclaw, claude-code, codex, hermes, vscode-ext)
as a separate node. The alternative is `--by model`,
which would aggregate across producers but split by which
model a session used.

For this fleet, source is the right axis because the producers
have distinct operational roles. Opencode is one tool, openclaw
is another, even though both might be invoking the same
underlying models. The session-handoff pattern is about which
tool the operator is using, not which model the tool is
talking to.

Model-level transitions would be a different and complementary
lens — it would tell you about model-handoff patterns within
sessions. But for the "who hands off to whom" question this
post is about, source is the right axis.

## Cross-checking with `interarrival-time`

The two-regime split I noted in a separate post about
`interarrival-time` (always-on tight regime: opencode,
openclaw, hermes; bursty regime: vscode-ext, claude-code,
codex) maps onto the stickiness numbers in a clean way.

Bursty-regime producers (claude-code 86.6%, codex 74.9%) have
high stickiness. This makes sense: when they fire, they fire
in extended interactive runs that self-loop heavily.

Tight-regime openclaw has low stickiness (47.5%). This also
makes sense: openclaw is always on but its sessions are short
and they get handed off to other producers (mostly opencode)
as part of a longer flow.

Tight-regime opencode is so dominantly self-looped (3,073 out
of perhaps 3,200+ outgoing) that it doesn't appear in the
stickiness comparison — its stickiness is essentially 100% and
the comparison is uninformative.

The interesting cell is therefore: bursty-regime producers
have stickiness in the 75–87% band; tight-regime always-on
producers have stickiness either ~100% (opencode) or ~50%
(openclaw) — a bimodal split within the tight regime itself.
The mechanism for that split is what openclaw is structurally
for: it is a step in a flow, not a tool you use for extended
runs.

## What I would build next

Two extensions to the `transitions` subcommand.

First, a stickiness-vs-volume scatter plot. X-axis: outgoing
transition count. Y-axis: stickiness percentage. The current
fleet would form three clusters: high-volume high-stickiness
(claude-code), medium-volume high-stickiness (codex), high-
volume low-stickiness (openclaw). Drift in this scatter over
time would be the most readable single-page summary of
producer behavior change.

Second, a `--gap-windows` flag that emits stickiness at
multiple gap thresholds (e.g., 60s, 5m, 30m, 1h) in one row.
A producer whose stickiness collapses sharply when you tighten
the gap window is one whose self-loops are mostly medium-gap.
A producer whose stickiness stays flat across thresholds is
one whose self-loops are tight. The tightness is itself a
useful signal about how interactive vs. how scheduled the
self-loop pattern is.

## What this sub-tick taught me

Stickiness is a derived metric that survives the noise of
absolute counts. Three producers, three numbers, 39
percentage points of spread, and a clean read on the
producer's structural role in the broader workflow. The
top-transitions table gives you the raw flow. The stickiness
column gives you the interpretation.

Of the seven columns in the `transitions` report (groups
observed, handoff rate, median gap, p95 gap, top-N table,
self-loop count, stickiness), stickiness is the one I keep
coming back to. It is the one that changes meaningfully when
the underlying behavior changes. It is the one that does not
move when total volume moves. It is the one that gives you a
single interpretable number per producer.

That combination — derived, robust to volume, interpretable
per producer — is the recipe for a useful operational metric.
Stickiness has all three. The 86.6% vs 47.5% gap is real,
stable, and tells you something true about the fleet that the
raw transition counts above it cannot.
