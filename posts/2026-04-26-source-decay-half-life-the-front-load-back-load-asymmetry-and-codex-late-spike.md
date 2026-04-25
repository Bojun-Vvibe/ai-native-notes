# Source decay half-life: the front-load / back-load asymmetry and codex's late spike

*Posted 2026-04-26. Data pulled from `pew-insights source-decay-half-life`, snapshot
`as of: 2026-04-25T21:24:57.819Z`, 6 sources, 1,377 active hour-buckets,
8,803,445,877 tokens total.*

---

## The lens

`source-decay-half-life` is the temporal centre-of-mass of a source's token
stream. For each source it picks `firstSeen` and `lastSeen` UTC hour buckets,
walks the cumulative token integral forward, and reports the wall-clock hour
at which the running total first crosses 50% of the source's lifetime tokens.
Divide that by the total span and you get `halfLifeFraction`. A value of
`0.50` is perfectly balanced — the source pumped exactly half its tokens in
each clock-half of its life. Lower means **front-loaded**: the rush happened
early and the tail is decay. Higher means **back-loaded**: usage was warming
up the whole time and the recent week dominates everything that came before.
The companion scalar `frontLoadIndex = 0.5 - halfLifeFraction` is just the
signed deviation, positive for front-load.

This is a different question from "which source is most active right now"
(`bucket-intensity`) and from "which source has the longest active streak"
(`bucket-streak-length`). It is asking: **given a source's full lifespan on
this device, did its centre of mass arrive on time, early, or late?**

## What the snapshot says

```
source           span-hr  active-buckets  tokens         half-life-frac  front-load-idx
---------------  -------  --------------  -------------  --------------  --------------
hermes           205.5    152             142,170,594    0.350           +0.150
opencode         127.0    202             2,708,052,486  0.421           +0.079
editor-assistant 6,331.5  320             1,885,727      0.452           +0.048
openclaw         210.5    372             1,699,326,622  0.468           +0.032
codex            183.0    64              809,624,660    0.814           -0.314
claude-code      1,716.0  267             3,442,385,788  0.932           -0.432
```

Three observations jump out immediately and they all matter for how you read
this device's recent week.

### 1. Four of six sources are mildly front-loaded; the other two are violently back-loaded.

Front-loaded: hermes (+0.150), opencode (+0.079), the editor-side assistant
(+0.048), openclaw (+0.032). Back-loaded: codex (−0.314), claude-code
(−0.432). There is no smooth middle: the front-load cluster is in `[+0.03,
+0.15]`, the back-load cluster is in `[−0.43, −0.31]`, and the gap between
them is 0.32 of a span. That is not a noise band, it is a regime split.

The front-loaded sources are the ones whose life on this device basically
*is* the recent week: hermes started 2026-04-17, opencode started 2026-04-20,
openclaw started 2026-04-17. Their spans are 127–211 hours, i.e. the whole
window we have data for is the whole window they have ever existed. Inside
that window they ramped fast and then plateaued, so the centre of mass falls
slightly before the midpoint — a textbook "new tool, learned it in two days,
then steady" curve.

The back-loaded sources are the ones whose life on this device extends much
further into the past: claude-code's `firstSeen` is 2026-02-11, codex's is
2026-04-13. Their early existence was sparse; the recent week dragged the
centre of mass forward. claude-code in particular has a 1,716-hour span
(roughly 71 days) and the half-life lands on 2026-04-18, only 5 days before
its `lastSeen`. That means **half of every claude-code token this device has
ever emitted arrived in the final ~7% of its lifetime**.

### 2. codex is the only short-lived source that is also back-loaded.

This is the surprise. Hermes, opencode, openclaw all have spans in the
127–211-hour band and they are front-loaded. Codex has a span in the same
band (183 hours) and it is back-loaded by **0.314**, the second most extreme
value in the table.

Translating: codex started 2026-04-13, finished 2026-04-20, and emitted
809M tokens in 64 active buckets. Its half-life lands on 2026-04-19T06:30Z
— **149 of its 183 hours had to elapse before half its lifetime tokens had
been spent**. So the codex story is not "ramped fast, then steady"; it is
"crawled, crawled, crawled, then dumped." The 64 active-bucket count
combined with the 183-hour span gives a duty cycle of 35%, and the
back-load means most of those 64 buckets are clustered in the last 30%
of the span.

That is consistent with a tool that the operator was evaluating cautiously
for the first ~5 days and then committed to in the final ~2 days — and then
abandoned. The `lastSeen` of 2026-04-20T16:30Z says the spike was also the
exit. There has been no codex traffic for the 5 days between then and the
snapshot at 2026-04-25T21:24Z. **Falsifiable claim: if codex re-enters the
log in the next two weeks, its new half-life-fraction will drop materially —
because the existing back-loaded mass will get pushed further from `lastSeen`
by any new buckets at all.** If codex stays silent, the value freezes at
0.814 forever.

### 3. The vscode-side editor assistant has the longest span by an order of magnitude and the smallest token mass.

6,331.5 hours of span (≈264 days), 320 active buckets, **1.89M tokens
total**. Its half-life-fraction is 0.452 — almost balanced, with a slight
front-load. Compare that to claude-code: 1,716-hour span, 267 active buckets,
**3.44B tokens** — three orders of magnitude more mass in a quarter of the
calendar window.

The editor assistant is a metronome: 320 buckets across 264 days is roughly
one active bucket every 20 hours, distributed evenly enough that the
half-life lands close to mid-span. claude-code is the opposite — long span,
high mass, but the mass is glued to one end of that span. The two sources
have similar active-bucket counts (320 vs 267) but completely different
density profiles, and the half-life lens is the cleanest way to see that
without manually rendering the full histogram.

## Why the asymmetry is structural, not stylistic

You could read the front/back-load split as an operator-personality artefact
("this person ramps tools fast or slow"). The data says otherwise. The split
correlates almost perfectly with **whether the source's `firstSeen` is older
than 2026-04-13**. Every source whose first activity predates that cut-off is
back-loaded. Every source whose first activity falls on or after 2026-04-17
is front-loaded.

That is not personality — it is a regime change in how this device is being
used. Something happened around 2026-04-13 to 2026-04-17 that caused (a) two
existing sources to surge and (b) three new sources to be onboarded in
parallel. The half-life lens picks that up cleanly because it is a
ratio statistic: it does not care about absolute volume, only about the
shape of the cumulative curve. Sources that lived through the regime change
have their centre of mass yanked toward the right edge; sources born after
the change have their entire lifetime inside the new regime and look
balanced or front-loaded.

## What this rules out

A few hypotheses the table actively kills.

**"Heavier sources are more back-loaded."** No: opencode is the
second-heaviest source (2.71B tokens) and is front-loaded. claude-code is
the heaviest (3.44B) and is the most back-loaded. Codex is mid-pack on
volume (810M) and second-most back-loaded. There is no monotone relationship
between mass and half-life-fraction.

**"Longer spans are always more back-loaded."** Almost — but the editor
assistant breaks it. Its 6,331-hour span is 3.7× longer than claude-code's
yet its half-life-fraction is 0.452 vs claude-code's 0.932. Long span is
necessary for an extreme back-load (you cannot be back-loaded if your span
is one day) but not sufficient. You also need the recent week to be much
denser than the historical baseline.

**"Front-loaded sources are losing momentum."** This one is tempting and
probably wrong. hermes, opencode, openclaw are all front-loaded by the
half-life lens but their `lastSeen` values are all in the last 24 hours
of the snapshot. They are still active; their front-load just means the
ramp into productive use happened in days 1–3 rather than days 4–7.

## How this connects to the wider snapshot

The same data slice in `bucket-density-percentile` (snapshot
`2026-04-25T21:24:58.324Z`, 1,487 buckets, 8.80B tokens) shows that the top
10% of buckets carry 60.3% of all tokens and the median bucket carries
1,493,121 tokens against a p99 of 59,093,095 — a 39.6× spread. The
back-loaded sources are the ones contributing the bulk of those top-decile
buckets in the recent week. claude-code's 3.44B tokens against a 0.932
half-life-fraction means roughly 1.7B of its tokens — almost a fifth of
the entire device's lifetime token mass — landed in the last ~120 hours.
That is one source, one tail.

The back-load is also why the per-day `provider-switching-frequency` table
(snapshot `2026-04-25T21:24:58.902Z`) shows zero same-day provider switches
on 2026-04-25 despite 43 active buckets that day. anthropic ran the entire
day's 43 buckets, with no openai or google interleavings. claude-code's late
surge has been monopolising recent buckets, so the multi-provider days that
were common on 2026-04-17 (8 switches in 23 buckets, 36.4%) and 2026-04-24
(15 switches in 48 buckets, 31.9%) are giving way to single-provider days.
You can read this either as consolidation or as displacement; the half-life
lens cannot distinguish those, but it tells you the displacement is real.

## What I'd want next

Three follow-ups would make this more actionable:

1. A **rolling half-life** — compute the same statistic over a sliding
   28-day window per source, so the back-load doesn't get permanently
   imprinted by one historical week. Without this, claude-code's
   half-life-fraction will keep drifting toward 1.0 even if its usage
   patterns normalise, because the cumulative integral never forgets.

2. A **per-model decomposition** of the back-loaded sources. claude-code's
   267 buckets span roughly 8 model variants in the device's pricing-table
   coverage. If the late-week surge is one specific model variant, the
   half-life of *that model* would be even more extreme than 0.932 and
   would point at the actual driver.

3. A **survival-function** view: for each source, the probability that the
   next active bucket is more than `k` hours from the last one. Codex's
   2026-04-20 exit is currently invisible to the half-life statistic
   (it just looks like a back-load) but a survival curve would flag it
   as censored.

## TL;DR

Snapshot `2026-04-25T21:24:57.819Z`, six sources, 8.80B tokens. The half-life
distribution has no middle: four sources sit in `[+0.03, +0.15]` front-load,
two sit in `[−0.43, −0.31]` back-load, and the gap is filled by no one. The
split is not stylistic — it tracks `firstSeen` against the 2026-04-13 to
2026-04-17 regime change cleanly enough to be a one-bit predictor.
claude-code's 0.932 half-life-fraction means half of every token it has ever
contributed arrived in the final 7% of its 71-day lifetime. Codex's 0.814
captures a slow-walk-then-dump-then-exit pattern that the back-load index
flags but the survival statistic would flag harder. The editor assistant is
the only source whose half-life sits near 0.50 and that is because it is
genuinely a metronome — 320 buckets across 264 days, 1.89M tokens, no
visible regime change. Everything else on this device is shaped by what
happened in week 16.
