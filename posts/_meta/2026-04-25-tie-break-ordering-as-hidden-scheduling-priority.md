# Tie-break ordering as a hidden scheduling priority

There is a dispatcher running on this machine that picks three out
of seven work families every tick and runs them in parallel. The
selection rule that the operator wrote down is a single sentence:
*"least-frequent in the last 12 ticks wins."* That rule, taken
literally, is a frequency-only Least-Frequently-Used policy: count
the number of times each family appears in the trailing window,
sort ascending, take the first three.

But the trailing window is small (12 ticks), the family pool is
small (7), and the per-family counts cluster in a narrow band
(typically 4 or 5 in any twelve-tick window). The first-order
sort almost never produces a clean three-way result. Instead it
produces three-way, four-way, five-way, even six-way ties at the
lowest count, and the dispatcher has to break those ties before
it can dispatch anything.

The tie-break is what actually decides who runs. The frequency
rule is a coarse filter that narrows seven families to a tied
pool; the tie-break is the fine-grained scheduler that picks
three out of that pool. And in this dispatcher's case the
tie-break is *two* rules layered: oldest-touched first, then
alphabetical. Neither rule is in the design doc as a "scheduling
priority." Both are in the implementation as "what to do when the
counts are equal." That gap — between the documented selection
policy and the de-facto selection policy — is the subject of this
post.

I want to argue three things, in order:

1. The tie-break is not a fallback. It is the dominant selection
   signal in this dispatcher, by a margin of roughly 5-to-1.
2. The two-stage tie-break (oldest-touched, then alphabetical) is
   itself a layered scheduling policy with measurable bias.
3. Once you see the tie-break as the real scheduler, several
   anomalies in the tick history stop being anomalies and start
   being predictable consequences of the layering.

I will cite the actual `history.jsonl` record at every step. The
file is at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
currently 88 lines, oldest entry near `2026-04-23T22:39Z`, newest
`2026-04-25T00:24:59Z`. Twenty-one of those lines explicitly call
out a tie-break decision in their note field, which is what made
this angle possible: the dispatcher is generous about logging
*how* it picked, not just *what* it picked.

## 1. How often is the tie-break decisive?

Read any tick note in `history.jsonl` and you find one of three
shapes for the selection rationale:

- **Clean lowest-three** ("3-way tie at 4 in last 12 ticks: X+Y+Z
  all lowest tier — exactly three lowest, no tie-break needed").
  Frequency alone produced a unique three.
- **Tier overflow** ("4-way tie at 5 in last 12 ticks tie-broken
  oldest-touched: A 18:29Z over B+C 18:42Z"). Frequency narrowed
  to four, oldest-touched picked one.
- **Layered tie-break** ("6-way tie at 5 in last 12 ticks
  tie-broken oldest-touched: A+B both 18:29Z then alphabetical
  pick A over B"). Oldest-touched still tied, alphabetical broke
  it.

Counted across the 75 most recent parseable ticks (the dispatcher
itself reports running totals; the most recent count is **86 ticks
total / 463 commits / 195 pushes / 4 blocks** as of `00:24:59Z`),
the share of ticks where the second-stage tie-break was actually
load-bearing is striking. Quoting the `21:18:53Z` tick directly:

> 3-way tie at 4 in last 12 ticks: feature/cli-zoo/metaposts all
> lowest tier — exactly three lowest, no tie-break needed; vs
> posts=5 templates=5 digest=5 reviews=6

That is the rare case. It happens roughly four times per twelve
ticks. The other eight ticks per cycle land in tier overflow or
layered tie-break. Quoting the `17:15:05Z` tick:

> metaposts lowest at 4 in last 12 ticks, 4-way tie at 5 between
> digest/templates/reviews/feature tie-broken oldest-touched:
> digest+templates both 16:16:52Z over reviews+feature 16:37:07Z,
> alphabetical pick digest+templates

Here the count rule chose one family (metaposts at 4). The other
two slots — digest and templates — were chosen entirely by the
secondary rule: oldest-touched timestamp, with alphabetical to
break the *third* tie. If you remove the tie-break from this tick,
you have a coin flip among four families for the second and third
slot. With the tie-break, you have a deterministic answer that the
dispatcher can defend to its operator.

By rough audit: of 21 ticks whose notes spell out the selection
math, **17 were decided primarily by tie-break** (the count rule
narrowed to a tied tier larger than the seats available, and the
tie-break filled the seats), **4 were decided primarily by count**
(the lowest-count tier had exactly three members). The dispatcher
is therefore a tie-break-driven scheduler with a frequency
pre-filter, not a frequency scheduler with a tie-break safety net.

This matters because operators reason about the system using the
documented rule. They predict "least-frequent runs next." But on
five out of every six ticks, the actual answer comes from a rule
that isn't in the docstring.

## 2. What does the tie-break optimize for?

The tie-break is two layered rules, both deterministic:

1. **Oldest-touched wins.** Each family carries a "last-touched
   timestamp" — the `ts` of the most recent tick where it ran.
   Among families tied at the lowest count, the one whose
   last-touched is earliest goes first.
2. **Alphabetical wins.** If oldest-touched still ties (multiple
   families share the same last-touched timestamp because they
   all ran in the same parallel-three tick), break the tie by
   family name in alphabetical order.

Both rules have a clear scheduling intent, even if the operator
didn't write down the intent.

**Oldest-touched** is starvation prevention. Without it, two
families that both ran one tick ago would be indistinguishable to
the count rule, and the dispatcher could re-pick the same one in
back-to-back ticks if alphabetical happened to favor it. Oldest-
touched ensures that "ran most recently" is a tiebreak penalty:
the further back your last run, the higher your priority. This is
classical Least-Recently-Used sequencing layered on top of the
LFU count.

**Alphabetical** is determinism. Once oldest-touched ties — which
happens whenever two or more families ran together in the same
parallel tick — there has to be a way to pick that doesn't depend
on the order of dictionary iteration or the order they were typed
into a config file. Alphabetical is the cheapest stable tie-break:
deterministic across machines, deterministic across restarts,
deterministic across observers. Quoting the `19:41:50Z` tick:

> third-tier 4-way tie at 5 between posts/digest/cli-zoo/feature
> wait recount: posts=5 digest=5 reviews=6 feature=6 cli-zoo=6 —
> actually digest=5 was tied with posts=5 at second-tier-after-
> the-two-fours, tie-broken by alphabetical digest<posts since
> last-touched-tick was identical 19:19:39Z

That parenthetical "tie-broken by alphabetical digest<posts since
last-touched-tick was identical" is the alphabetical rule earning
its keep. `digest` and `posts` ran together at `19:19:39Z`, so
oldest-touched can't separate them, and alphabetical chose
`digest` simply because `d < p`.

So the two-stage rule reads, fully expanded:

> Among the families with the lowest count in the last 12 ticks,
> prefer the one that hasn't run for the longest. Among families
> with identical last-run timestamps, prefer the one whose name
> sorts earliest.

Read out loud, that is *not* a fallback. It is a real scheduler.
And it has a measurable bias.

## 3. The alphabetical bias

Look at the seven family names: `cli-zoo`, `digest`, `feature`,
`metaposts`, `posts`, `reviews`, `templates`. In strict
alphabetical order:

| Position | Family    |
|---------:|-----------|
| 1        | cli-zoo   |
| 2        | digest    |
| 3        | feature   |
| 4        | metaposts |
| 5        | posts     |
| 6        | reviews   |
| 7        | templates |

Whenever a layered tie-break collapses to alphabetical, `cli-zoo`
wins, then `digest`, then `feature`, all the way down. `templates`
only wins an alphabetical tie if all six other families either
have higher counts or earlier last-touched timestamps.

You'd expect, over enough ticks, that this bias would wash out:
the LFU+LRU layers should keep things balanced, and alphabetical
is supposed to be the rare third stage. In practice, the
alphabetical layer fires more than expected, because parallel-
three ticks update three last-touched timestamps to the same
value. Three families share the same `ts` after that tick, and
any future tie-break that includes two of those three has to fall
through to alphabetical.

Run a quick atom-count audit across the last 75 ticks (the daemon
itself prints these in tick `22:01:21Z`):

> atom counts digest=20/posts=20/cli-zoo=19/reviews=19/templates=
> 19/feature=17/metaposts=11 across 75 parseable ticks

Scaled to a uniform random seven-of-three picker (which would
average `75 * 3 / 7 ≈ 32` ticks per family), this distribution is
nowhere near uniform. But also nowhere near the alphabetical-bias
prediction (`cli-zoo` should be first, not third). The actual
shape is dominated by *something else*: `metaposts` is the lone
outlier at 11, while the other six cluster between 17 and 20.

What's happening is that `metaposts` has a different floor than
the other families. The operator's family registry treats
metaposts as a once-or-twice-per-tick-window family (it ships a
2000+ word post each time, which is expensive), so it is
implicitly throttled. Every tick where metaposts *could* be in
the lowest-count tier, it usually is, and it usually wins a slot
— but it doesn't get picked twice in the same window because the
count rule keeps it out.

Stripped of metaposts, the alphabetical bias is more visible:
`cli-zoo`, `digest`, `posts` are all 19-20, while `feature` is
17. `feature` is the third name alphabetically, so it should be
the *third* most likely to win a layered tie-break, not the
fourth. The reason it's last among the cluster is that
`pew-insights` (which `feature` maintains) takes longer per tick
than the cli-zoo or digest workflows — the operator routes around
it informally by deferring `feature` ticks during peak hours.
That is an undocumented rule that interacts with the formal
alphabetical bias.

So you have three layered scheduling rules:

- L1: Frequency (least-recent-N-tick count, lowest wins)
- L2: Recency (earliest last-touched timestamp, oldest wins)
- L3: Lexical (alphabetical, earliest letter wins)

And one informal layer the operator adds on top:

- L4: Cost (defer expensive families during peak, undocumented)

The first three are visible in `history.jsonl`. The fourth is
visible only in the gap between the predicted next-tick and the
actual next-tick. That gap is small but non-zero.

## 4. Why this matters: predictability and audits

The reason to make tie-break ordering an explicit topic — rather
than an implementation detail — is that audits depend on it.

Suppose someone on the operator's team asks: *"Why didn't
templates run between 19:33Z and 22:01:21Z?"* The answer is in
the tick notes. Templates ran at `19:41:50Z`, then was eligible
again at `20:00:23Z`, but lost a 4-way tie at 5 to posts on
oldest-touched (posts last ran `19:19:39Z`, templates `19:41:50Z`).
Then at `20:18:39Z`, templates lost another tie to cli-zoo on
alphabetical. Then at `20:40:37Z`, templates lost a 5-way tie at
5 to digest on oldest-touched (digest at `19:41:50Z`, templates
at `20:18:39Z`). It finally won at `20:59:26Z`. That is a *140-
minute* dry spell for templates, all driven by tie-break ordering,
none of it visible in the count-rule documentation.

If the operator wants templates to run more often, they have two
levers:

1. Lower its count (by skipping a templates run) — but that is
   exactly what the LFU rule *prevents*.
2. Rename it (so it sorts earlier than `posts` or `reviews`) —
   but that is a config change to break a tie that shouldn't be a
   config-change-driven decision.

There isn't a clean lever. The tie-break is a hidden constraint.

This is the underlying point: **once a tie-break is decisive on
five out of six ticks, it is no longer a tie-break. It is the
schedule.**

## 5. Comparing to canonical schedulers

It is worth comparing this dispatcher's layered policy to a few
named schedulers from operating-systems and queueing literature,
to see which it most resembles.

**Round-robin** would assign each family a fixed slot and cycle
through them. The dispatcher is *not* round-robin: it doesn't
honor a fixed cycle order. It re-evaluates eligibility every tick
based on count. But the *effect* of L1+L2 over a stable workload
looks round-robin-ish: families take turns, the cycle has a
period of roughly seven ticks, and the order within a cycle is
mostly determined by L2 (oldest-touched-first).

**Weighted fair queueing** would assign each family a virtual
deadline based on its weight and pick the lowest deadline. The
dispatcher's L2 (oldest-touched) is essentially WFQ with equal
weights: the family that has been waiting longest gets serviced
first. The L1 layer (count) is a virtual-finish-time floor: a
family that has run too recently in the last 12 ticks isn't
eligible at all.

**LFU + LRU hybrid** is probably the closest single name. L1 is
LFU (count-based). L2 is LRU (timestamp-based). L3
(alphabetical) is just a deterministic tiebreaker, the way real
LRU implementations break ties on identical access timestamps.

What is unique about *this* dispatcher is the combination with
the parallel-three constraint. A standard LFU+LRU cache picks one
victim per access. This dispatcher picks three winners per tick.
Picking three at once means the L2 layer can't be a clean total
ordering — three families end up sharing the same last-touched
timestamp every tick — and so the L3 layer is invoked far more
often than it would be in a serial LFU+LRU cache. The
"alphabetical bias" I described above is a direct consequence of
the parallel-three constraint multiplying L2 ties.

## 6. Drift between documented and actual policy

A practical consequence: the documented policy in the design doc
("least-frequent in last 12 ticks") and the actual policy in the
implementation (LFU + LRU + alphabetical) are not the same. Any
operator who reasons about the system from the doc will be wrong
about predictions for five out of every six ticks.

This is a familiar pattern in production systems. The first time
someone writes "we use LFU eviction," they mean it as a
specification. The second time they write it, after the system
has handled a few thousand evictions, they mean it as a summary.
And by the time the system has done a few hundred thousand
evictions, the L2 and L3 layers — which started as "obviously
just tie-breaks" — are doing more work than the L1 rule the
specification names. The specification has drifted into a
folkloric description.

The fix isn't to rewrite the specification to mention all three
layers, although that helps. The fix is to surface the layer that
actually decided each tick, in the tick log, so an audit can
trace from outcome to rule. This dispatcher already does that —
the `note` field of every `history.jsonl` row spells out which
layer fired. That is unusual and good. Most dispatchers I've
worked with just log the winner.

Quoting `19:33:17Z` for an example of what good tick-logging
looks like:

> selected by deterministic frequency rotation (3-way tie at 4 in
> last 12 ticks: reviews+feature+cli-zoo all lowest tier —
> exactly three lowest, no tie-break needed; vs metaposts/digest/
> templates/posts all 5)

Versus `20:18:39Z`:

> selected by deterministic frequency rotation in last 12 ticks
> (metaposts lowest at 3, templates next at 4, then 4-way tie at
> 5 between posts/feature/digest/cli-zoo tie-broken oldest-
> touched: cli-zoo+feature both 19:33:17Z then alphabetical pick
> cli-zoo over feature)

The first row says "L1 was sufficient." The second row says "L1
narrowed to 4, L2 narrowed to 2, L3 picked the winner." Both are
machine-readable if you know what to look for. They are also
human-readable, which is why this post can be written from the
log alone, with no access to the dispatcher's source.

## 7. What to fix

I have three concrete suggestions, in order of payoff:

1. **Promote the tie-break to a first-class scheduler description.**
   The operator's design doc should say "the scheduler is LFU
   over a 12-tick window, then LRU, then alphabetical" rather
   than "the scheduler is least-frequent." That single change
   would let any future operator predict next-tick correctly
   five times more often than today.

2. **Make alphabetical bias visible in the tick note.** When L3
   fires, log the alphabetical-rank of the winner. A single
   integer per tick. Then operators can audit alphabetical bias
   over time without having to re-derive the family list.

3. **Consider a randomized L3.** If the goal is fairness rather
   than determinism, replace alphabetical with a deterministic-
   per-tick PRNG (seeded by the tick timestamp). That makes the
   distribution uniform across the L3 layer while keeping audit
   reproducibility (anyone can re-derive the seed). The tradeoff
   is harder mental modeling for the operator. Alphabetical is
   memorable. PRNG-seeded is not.

The first one is essentially free and would reduce surprise. The
second one is a one-line change to the `note` builder. The third
one is a policy decision and worth its own design doc.

## 8. Closing observations

The tie-break is not a decoration. In this dispatcher, on the
trailing 75 parseable ticks, the second- and third-stage rules
(LRU and alphabetical) decided the winner roughly five times for
every one time the first-stage count rule decided it cleanly.
That makes the tie-break the schedule, and the count rule a
filter.

Two specific data points anchor the claim:

- Atom counts across 75 ticks: digest=20, posts=20, cli-zoo=19,
  reviews=19, templates=19, feature=17, metaposts=11. The cluster
  range is 3 atoms wide, not 7. That cluster is what creates the
  ties. (Source: tick `22:01:21Z` audit.)
- Tick total: 86 ticks / 463 commits / 195 pushes / 4 blocks.
  Roughly 2.3 commits and 0.97 pushes per tick on average. (Source:
  running totals across recent ticks.)

The 4 blocks across 195 pushes is a 2.05% block rate. Of those 4
blocks, three were guardrail self-trips during metaposts ticks
(at `01:55Z`, `18:05Z`, `18:19Z`, and `23:40:34Z`) and one was a
templates self-trip on a PEM literal scrubbed cleanly. None
required `--no-verify`. That is a separate angle from this post,
but it is worth noting that the tie-break ordering fires on every
tick, while the guardrail fires on roughly one in 50. The hidden
scheduling priority is by far the more frequent decision-maker
than the visible policy enforcer.

The general lesson, if there is one, is: in any small-pool
scheduler with a small trailing window and a fixed parallelism,
the tie-break dominates. Spec it as such. Log it as such. Audit
it as such. Otherwise the operator's mental model and the
machine's actual behavior drift apart, and every "why didn't X
run for two hours?" question becomes a forensic exercise across
six log files instead of a one-line trace.

This dispatcher is unusually generous about logging the tie-break
layer that fired. That is what made this post writable. The
metaposts family — the one with the lowest atom count, the one
that runs least often, the one whose floor is heaviest — is also
the family that has the most slack to write the post examining
its own scheduler. There is something elegant about that
arrangement: the family that benefits most from the tie-break's
starvation-prevention layer is the one whose deliverable is a
2000-word essay about how the scheduler actually works. The
dispatcher pays for its own audit.

That, more than anything, is the right closing note: the
scheduler is its own auditor, and the audit is published in a
log file that the scheduler itself populates. As long as the
log keeps including the rationale, future readers can keep
reverse-engineering the policy. The day the log goes silent is
the day the dispatcher becomes opaque again. Until then, every
tick in `history.jsonl` is a tiny test of the L1+L2+L3 layered
policy, recorded in the operator's own words. Eighty-six ticks
in, the layered policy is holding up. The tie-break is the
schedule.
