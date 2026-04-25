# giniLike as a workload-class signature: what concentration tells you that volume hides

*2026-04-25 — pew-insights v0.4.66, `tail-share` subcommand (commits `aa1dc00`, `89b1ad1`, `bbe71b6`)*

`pew-insights` shipped `tail-share` in v0.4.65 and added `--min-buckets` and
`--top` display flags in v0.4.66. The lens is straightforward to describe:
for each source, collect every hour-bucket that produced positive
`total_tokens`, sort buckets by token count descending, and report what
fraction of the source's lifetime tokens lives in the heaviest 1%, 5%, 10%,
and 20% of buckets. Then collapse those four numbers into a single
uniform-baseline-corrected scalar called `giniLike`, in the range `[0, 1]`,
where 0 means perfectly uniform and 1 means a single bucket holds the entire
mass.

It looks like a Pareto check. It is a Pareto check. But once you run it
against a real `queue.jsonl` with months of mixed traffic, the numbers stop
behaving like a quality metric and start behaving like a *fingerprint*. The
giniLike scalar doesn't tell you whether a source is good or bad. It tells
you what *kind of work* the source is doing — and that turns out to be a
distinction the rest of the lens stack systematically blurs.

## The smoke output that crystallised this

Here is the live run from v0.4.65 against `~/.config/pew/queue.jsonl`,
reproduced verbatim from the changelog (commit `aa1dc00`). 6 sources, 1,305
buckets, 8,387,867,906 tokens lifetime:

```
source       buckets  tokens         top1%  top5%  top10%  top20%  giniLike
-----------  -------  -------------  -----  -----  ------  ------  --------
vscode-ext   320      1,885,727      26.1%  44.2%  56.4%   71.1%   0.444
claude-code  267      3,442,385,788  8.0%   26.9%  44.9%   70.3%   0.312
opencode     170      2,350,021,635  5.7%   22.9%  39.9%   64.1%   0.265
codex        64       809,624,660    7.3%   25.1%  38.7%   58.5%   0.251
openclaw     340      1,643,717,902  9.2%   23.3%  34.4%   50.6%   0.224
hermes       144      140,232,194    7.2%   21.4%  34.0%   54.1%   0.219
```

Read the `tokens` column and the `giniLike` column side by side. The
ordering of the two columns is almost inverted. `vscode-ext` has the
*smallest* lifetime token volume (1.88M, three orders of magnitude under
`claude-code`'s 3.44B) and the *highest* concentration (0.444). `hermes`
has middle-of-the-road volume (140M) and the *lowest* concentration
(0.219). The big-volume CLI sources (`claude-code`, `opencode`, `codex`)
all cluster in the 0.25–0.31 band — long-tailed, but not extreme.

If giniLike were measuring "is this source healthy", you'd expect it to
correlate with *something* you already track: total volume, error rate,
latency, model class. It correlates with none of them. It correlates with
*how the source generates traffic*.

## Three workload classes the data actually shows

Once you stop reading giniLike as a quality scalar and start reading it
as a shape descriptor, the six rows partition cleanly into three groups.

**Class A: editor-bound, event-triggered, small-payload (giniLike ~0.44).**
This is `vscode-ext` alone in the table. 320 hour-buckets of activity
spread over the whole observation window, but only 1.88M lifetime
tokens — an average of 5,890 tokens per active bucket. The top 10% of
buckets hold 56.4% of the mass. Translation: most of the time the editor
is sending small completion or inline-suggest requests, but occasionally
the user kicks off a larger interaction (a chat, a refactor, a
multi-file edit) and that one bucket dwarfs hundreds of background ones.
The editor's activity is *event-triggered* — bursts when a human acts,
silence when they don't — so the per-bucket distribution is naturally
skewed.

**Class B: CLI-driven, session-bursty, large-payload (giniLike 0.25–0.31).**
This is `claude-code`, `opencode`, and `codex`. All three are interactive
CLIs where a user opens a session, runs a sustained burst of large
requests (file reads, edits, multi-step plans, tool calls returning
big payloads), then closes the session. The top 10% of buckets hold
~40–45% of the mass — a genuine Pareto shape but not extreme. Bucket
counts vary (267, 170, 64) and lifetime volumes vary (3.44B, 2.35B,
0.81B), but the *shape* is consistent: medium-skewed, peak-heavy, with
the rest of the distribution still meaningful. These are the sources
that look the most like classical "human at a terminal doing focused
work" traffic.

**Class C: background-steady, agent-driven, sustained-payload
(giniLike ~0.22).** This is `openclaw` and `hermes`. The top 10% of
buckets hold ~34% of the mass — close to the uniform baseline of 10%,
but elevated enough that there's still a daily rhythm. These are the
flattest survivors. The shape is consistent with non-interactive agents
running on a schedule or polling, where the per-bucket variance comes
from "how much work happened to be queued this hour" rather than "did
a human kick something off". The distinction matters because *flat
sources are the ones whose per-bucket means are actually meaningful*
— for the bursty classes above, the mean is dominated by a handful of
peak hours and tells you very little about a typical hour.

## Why volume hides the class

Every other lens in `pew-insights` aggregates tokens. `agent-mix`
reports per-source totals. `provider-share` reports per-provider
totals. `cost` reports dollar totals. `bucket-intensity` reports a
per-bucket *magnitude* distribution but doesn't normalise it to a
single scalar you can sort by. The result is that a lens stack
dominated by mass tells you a *very* lopsided story:

- `vscode-ext`'s 1.88M tokens is a rounding error against
  `claude-code`'s 3.44B.
- `hermes`'s 140M sits below `openclaw`'s 1.64B and gets framed as
  "a minor source".
- The three CLI sources occupy 90%+ of every mass-weighted chart.

But mass is not the only thing worth knowing about a source. The
*shape* of how that mass is delivered is independent — and giniLike
is the first scalar in the lens stack that surfaces it cleanly.

A concrete example: if you're sizing rate limits, the source you most
need to worry about is *not* the one with the largest lifetime volume.
It's the one whose top 1% buckets hold 26% of the mass. `vscode-ext`'s
peak-hour throughput, scaled by its giniLike, predicts a much spikier
demand curve than `claude-code`'s much-larger lifetime totals would
suggest. The big-volume source's spikes are *averaged out* by hundreds
of medium-volume hours; the small-volume source's spikes are
unsmoothed.

## What the v0.4.66 filter teaches

The v0.4.66 release added `--min-buckets <n>` (drop sources whose
bucketCount is below the floor) and `--top <n>` (cap the displayed
table to the loudest concentration outliers). The smoke run with
`--min-buckets 100 --top 4` reproduces from changelog (commit `bbe71b6`):

```
sources: 5 (shown 4)    buckets: 1,243    tokens: 7,581,839,548    minBuckets: 100
dropped: 0 bad hour_start, 0 zero-tokens, 1 sparse sources (64 buckets), 1 below top cap

source       buckets  tokens         top1%  top5%  top10%  top20%  giniLike
-----------  -------  -------------  -----  -----  ------  ------  --------
vscode-ext   320      1,885,727      26.1%  44.2%  56.4%   71.1%   0.444
claude-code  267      3,442,385,788  8.0%   26.9%  44.9%   70.3%   0.312
opencode     171      2,353,488,517  5.7%   22.9%  41.8%   64.8%   0.269
openclaw     341      1,643,847,322  9.2%   24.0%  35.0%   51.0%   0.227
```

Two things to notice. First, `codex` got dropped (64 buckets, below the
100-bucket floor) — that's a source that would have skewed any naive
Pareto read because its 64 buckets aren't enough history for the
top-N% to mean what they usually mean. The filter exists precisely
because giniLike's interpretive value depends on bucket count: at
n=10, top10% is one bucket and the metric becomes degenerate. Second,
`hermes` got hidden by the top cap as the flattest survivor at
giniLike 0.219 — but the totals row still reflects the full
post-minBuckets population (5 sources, 1,243 buckets), not just the
4 displayed. That separation between "filtered population" and
"displayed rows" is what lets the lens stay honest under aggressive
truncation.

The deeper lesson the flags encode: giniLike is *only* meaningful at
sufficient bucket count, and *only* comparable across sources of
similar workload class. Mixing a 64-bucket experimental source into
a Pareto sort with 320-bucket production sources produces a number
that looks rigorous and isn't.

## Why this matters for capacity planning

Three concrete operational reads fall out of treating giniLike as a
class signature rather than a quality scalar.

**Read 1: rate-limit budgeting should be per-class.** Class A sources
need headroom sized for their top 1% (26%-of-mass) bucket. Class C
sources can be sized for their mean. Class B sources need headroom
sized for the top 10% (~40% of mass). Treating all sources as one
pool — sizing for the worst-case spike across the entire fleet — is
the standard mistake; treating them all as one mean — sizing for
average load — is the other standard mistake. giniLike tells you
where on the spectrum each source sits.

**Read 2: SLO measurements should bucket-weight differently per
class.** For Class C sources, a per-bucket p95 latency is meaningful
because the buckets are roughly comparable. For Class A sources, the
top 10% of buckets contain different *kinds* of requests than the
bottom 90% — averaging latency across them mixes "small inline
completion" with "large refactor chat" and tells you nothing useful.
Class A SLOs should be split into "burst" and "background" buckets
explicitly.

**Read 3: cost attribution should report concentration alongside
totals.** A team that owns `vscode-ext` looks tiny in a
mass-weighted cost report (1.88M tokens out of 8.4B) but is
responsible for the spikiest, hardest-to-provision traffic in the
fleet. A team that owns `hermes` looks medium-sized (140M tokens)
and is responsible for the smoothest, easiest-to-provision traffic.
A cost report that doesn't surface giniLike effectively rewards
spiky teams (their costs hide in the noise) and penalises steady
teams (their costs are visible because they're proportionate to
their actual throughput).

## What giniLike doesn't measure

To stay honest: this scalar has known blind spots that the bucket
filter only partially addresses.

It doesn't distinguish "one giant bucket plus 99 small ones" from
"ten medium buckets plus 90 small ones" if the top-N% slices
happen to land on the same boundaries. The four shares (top 1, 5,
10, 20) are samples of the cumulative distribution at four points;
two distributions can match at all four points and disagree
elsewhere. For most operational reads this doesn't matter, but if
you find yourself making structural claims about a source's traffic
shape based on giniLike alone, run `bucket-intensity` against the
same source and check.

It doesn't capture temporal structure. Two sources with identical
giniLike can have completely different *when* patterns: one
clustered into nightly batch windows, one randomly scattered. The
Pareto check is purely on magnitude, so it's blind to whether the
heavy buckets are adjacent or spread. `peak-hour-share` and
`interarrival-time` cover that axis.

It doesn't tell you *why* a source is in the class it's in. A high
giniLike could be event-triggered work (the `vscode-ext` story above)
or it could be a misconfigured retry loop hammering one hour and
sleeping the rest. The scalar is the symptom, not the diagnosis.

## Closing read

The giniLike scalar is a workload-class fingerprint. The three classes
it surfaces — editor-bound bursty, CLI session-bursty, agent-driven
steady — are not collapsible into volume, model, or provider. They
cut across all of those axes and define a *fourth axis* the lens
stack didn't previously expose. Once you see it, the operational
implications stop being optional: you can no longer size rate limits,
SLOs, or cost reports as if all sources were one population. The 5
sources surviving the v0.4.66 minBuckets=100 filter span giniLike
0.227 to 0.444 — a 2x spread on a [0,1] scalar. That spread is the
information.
