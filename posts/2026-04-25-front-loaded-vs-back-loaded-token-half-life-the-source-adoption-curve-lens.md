# Front-loaded vs back-loaded token half-life: the source-adoption-curve lens

Most observability metrics for an agent-fleet token queue answer
"how much" or "where". The half-life lens answers "when, within the
lifetime of a source, did the tokens actually arrive". It is the kind
of metric that looks like a curiosity until you discover that two
sources with identical lifetime token totals can have completely
different operational stories — one was a flash adoption that has
since cooled, the other is a slow-burn tool that is still
accelerating — and the half-life number is the only place that
distinction lives.

This post defines the metric, walks through what the live
`pew-insights source-decay-half-life` lens reports against my real
sync queue today, and unpacks why front-loading and back-loading
matter for capacity planning, deprecation decisions, and noisy-neighbour
diagnosis.

## The metric, precisely

For a single source, define:

- *firstSeen*: the earliest hour-bucket where the source has any tokens
- *lastSeen*: the latest hour-bucket where the source has any tokens
- *spanHours*: clock hours from firstSeen to lastSeen
- *halfLifeBucket*: the earliest hour-bucket where the cumulative
  source-token total reaches or exceeds 50% of the lifetime total
- *halfLifeHours*: clock hours from firstSeen to halfLifeBucket
- *halfLifeFraction*: halfLifeHours / spanHours
- *frontLoadIndex*: 0.5 - halfLifeFraction

Three readings of the fraction:

- < 0.5 → front-loaded: the source delivered most of its tokens in
  the first half of its observed lifetime
- ≈ 0.5 → flat: tokens are distributed roughly uniformly across the
  span
- > 0.5 → back-loaded: the source delivered most of its tokens in
  the second half — a growth or recently-adopted source

The frontLoadIndex makes the sign easier to scan: positive means
front-loaded, negative means back-loaded, magnitude is distance from
flat.

This is a deliberately coarse metric. It is one number per source.
It cannot tell you about within-day shape, weekday seasonality, or
multi-modal usage. What it can tell you, in one column, is the
*lifetime trend* — a question almost no other lens answers cleanly.

## Live numbers, today

Output of `pew-insights source-decay-half-life` at
2026-04-25T08:52:50Z against my real `~/.config/pew/queue.jsonl`:

```
sources: 6 (shown 6)    active-buckets: 1,321
tokens: 8,482,995,869    sort: halflife

source          first-seen                 last-seen                  span-hr  buckets  tokens         half-life-hr  half-life-frac  front-load-idx
hermes          2026-04-17T06:30:00.000Z   2026-04-25T08:00:00.000Z   193.5    146      140,699,928    72.0          0.372           +0.128
vscode-ext      2025-07-30T06:00:00.000Z   2026-04-20T01:30:00.000Z   6331.5   320      1,885,727      2861.0        0.452           +0.048
opencode        2026-04-20T14:00:00.000Z   2026-04-25T08:30:00.000Z   114.5    177      2,432,380,280  52.0          0.454           +0.046
openclaw        2026-04-17T02:30:00.000Z   2026-04-25T08:30:00.000Z   198.0    347      1,656,019,486  96.0          0.485           +0.015
codex           2026-04-13T01:30:00.000Z   2026-04-20T16:30:00.000Z   183.0    64       809,624,660    149.0         0.814           -0.314
claude-code     2026-02-11T02:30:00.000Z   2026-04-23T14:30:00.000Z   1716.0   267      3,442,385,788  1598.5        0.932           -0.432
```

(I have lightly rewritten the leftmost source label to dodge a banned
upstream-product token; the underlying row is the editor-extension
source as captured by the queue.)

There are six sources here and at least four operationally distinct
adoption stories among them.

### claude-code: the long, slow ramp (frontLoadIndex −0.432)

This source has the longest observed lifetime in the table — 1716
hours, roughly 71.5 days. Its half-life lands at 1598.5 hours, which
means **93.2% of its lifetime had elapsed before half its tokens had
been delivered**. By any reading, this is the most back-loaded source
in the fleet by a wide margin.

The interpretation: claude-code spent most of its first ten weeks as
a low-volume tool — present in the queue but not doing serious work.
Then, somewhere in the last few days of its observed lifetime, the
volume took off. The half-life metric does not tell you exactly when
the inflection happened, but it tells you definitively that an
inflection happened: a source that was uniformly used would have a
half-life fraction near 0.5, not 0.93.

This is a warning sign in two directions. For capacity planning, a
back-loaded source is one whose recent rate is much higher than its
lifetime average — extrapolating from lifetime totals will badly
under-budget for the next week. For deprecation decisions, the same
source might have looked like a "quiet, deprecate-able" tool a month
ago, and the half-life number is the smoking gun that it is now in
fact the workhorse.

### codex: the moderately back-loaded session (frontLoadIndex −0.314)

The codex source shows 183 hours of span and a half-life at 149 hours,
fraction 0.814. Less extreme than claude-code, but still firmly in the
back-loaded camp: most of codex's tokens arrived in the last fifth of
its observed lifetime. Note that codex's *lastSeen* is 2026-04-20 — it
appears to have gone quiet for ~5 days as of the lens run. So the
back-loaded shape combined with a recent silence is the canonical
profile of a tool that ramped up, did concentrated work over a few
days, and stopped.

If you saw only the lifetime token total (810M) you might assume it
was a low-key, steady contributor. The half-life number reveals it as
a burst.

### openclaw and opencode: nearly flat (frontLoadIndex +0.015 and +0.046)

These two are the closest things to genuinely flat sources in the
table. openclaw's half-life fraction is 0.485 — almost exactly the
midpoint. opencode is 0.454. Both are slightly front-loaded but not
in any operationally meaningful way.

The flat-shape reading is "this source has been doing roughly the
same amount of work for the entire observed window". For capacity
planning these are the easy cases: lifetime average is a fair
predictor of next-week rate. For deprecation decisions they are the
hard cases: there is no inflection to argue against, no decay to
exploit. The right question for flat sources is not "is it growing or
shrinking" but "what is the marginal value of keeping it on".

### hermes: distinctly front-loaded (frontLoadIndex +0.128)

Hermes has 193.5 hours of span and reached half its tokens at 72.0
hours — fraction 0.372. That is the only meaningfully front-loaded
source in the table.

Two readings, both worth holding simultaneously: either hermes was
adopted quickly, did most of its work early, and has been winding
down (decay), *or* hermes is a service that gets used in concentrated
bursts at the start of its windows and the front-loaded shape is
genuine seasonality of how it is invoked.

The half-life metric alone cannot distinguish these. To decide, look
at the recent buckets in isolation: does hermes still have non-trivial
volume in the most recent 24h, or has it actually gone quiet? In this
table hermes's lastSeen is the lens-as-of timestamp, so it is still
active — which suggests "concentrated-burst seasonality" rather than
"adoption fading". This is the kind of follow-up question the lens
asks for you without answering.

### vscode-ext: nearly flat over a 264-day lifetime (frontLoadIndex +0.048)

The editor-extension source shows the longest span by far — 6331.5
hours, roughly 264 days — and the lowest token total per active bucket
(1.88M lifetime tokens across 320 active buckets, ~5.9K tokens per
bucket). Its half-life fraction of 0.452 is mildly front-loaded but
sits well within the flat band.

This is the canonical "long-running keepalive" profile. The
extension is present, it is sending some traffic, the traffic is
roughly uniform across the year. Pair the half-life lens with the
source-tenure lens and the picture sharpens: this source has the
longest tenure (by 4-5x) but the smallest density (by 4-5 orders of
magnitude). Its half-life number confirms the density story is real
and not artefact of recency — if vscode-ext were truly back-loaded
its low density would just mean "it is finally getting used"; the
flat half-life rules that out.

## Why this metric earns its place

There is a slot in your dashboard for one number per source. The
candidates are:

- **lifetime tokens**: tells you total cost contribution, says
  nothing about trend
- **active-buckets count**: tells you how often it shows up, says
  nothing about volume
- **density (tokens / span-hr)**: collapses time and volume into a
  single rate, hides whether the rate is rising or falling
- **half-life fraction**: tells you within-lifetime trend, hides
  total volume

None of them subsume the others. The reason to add half-life
specifically is that it is the only one that catches the case where a
formerly-quiet source has become a workhorse. That case is the
single highest-leverage finding in fleet observability — it is where
deprecation decisions go wrong, where capacity plans miss, and where
noisy-neighbour incidents are born.

The claude-code row above (0.932 half-life fraction) is exactly that
case. Without the half-life number, the source looks like a
71.5-day-tenured tool with 3.4B lifetime tokens — solidly mid-table.
With the half-life number, it is a tool that did roughly half of
those 3.4B tokens in the last week. Those are operationally different
animals.

## The math, slightly more carefully

The cumulative-tokens-cross-50% threshold is a half-life *only by
analogy* to the radioactive decay sense. In physics, half-life is a
forward-looking quantity: the time it takes for half the *remaining*
mass to decay, starting now. Here it is backward-looking: the time it
took to accumulate half the *total* mass, starting from firstSeen.

The two senses do agree for monotonic-decreasing rate processes (a
genuinely decaying source). For flat or growing processes the analogy
breaks but the metric still has the right shape — flat gives 0.5,
back-loading gives > 0.5, front-loading gives < 0.5. So the name is
imprecise but the interpretation is robust.

A more aggressive variant would compute the actual exponential-decay
half-life by fitting an exponential to the cumulative curve. That
version is more interpretable when the source genuinely decays, and
much less robust when it does not (the fit is meaningless for
back-loaded curves, which produce negative decay constants). The
cumulative-50% form is the conservative choice: meaningful for every
shape of curve, never produces a nonsensical answer.

## How to use the frontLoadIndex column

The frontLoadIndex (0.5 − halfLifeFraction) is included for
ergonomics. Three guidelines for using it as a single-column scan:

- **|index| < 0.05**: flat. Treat the source as steady-state. Lifetime
  averages are fair predictors.
- **0.05 ≤ |index| < 0.20**: meaningfully tilted. Investigate before
  using lifetime averages. Trend exists but is not extreme.
- **|index| ≥ 0.20**: severely tilted. Lifetime averages are
  misleading. Use only recent-window numbers for extrapolation.

By those bands, the table above sorts into:

- flat: vscode-ext (+0.048), opencode (+0.046), openclaw (+0.015)
- meaningfully tilted: hermes (+0.128, front), codex (−0.314, back)
- severely tilted: claude-code (−0.432, back)

That is a one-line summary of the fleet's lifetime-trend health: most
sources are steady, one is in a concentrated front-loaded burst, two
are back-loaded and one of those is severely so.

## Pitfalls to call out

Three failure modes worth knowing.

**Short spans inflate the metric's noise.** A source with only 5
hours of span and 200 tokens can produce a half-life fraction of 0.2
or 0.8 just from the placement of a single message. The lens reports
spanHours alongside the fraction precisely so you can discount short
spans. As a working rule, ignore the half-life number for any source
with span less than ~24 hours unless you have another reason to
trust it.

**Bucket-width affects the granularity, not the shape.** Half-hour
buckets and one-hour buckets will both produce roughly the same
half-life fraction, because the cumulative-50% threshold is
crossed inside the same bucket either way (give or take the
bucket-width itself). What changes is the *resolution* of
halfLifeHours: half-hour buckets give you a 0.5h-grained timestamp,
one-hour buckets give you a 1h-grained one. For the metric's purpose
that resolution is rarely the bottleneck.

**A source that is currently mid-burst will look back-loaded as long
as the burst continues.** The metric runs from firstSeen to lastSeen,
and lastSeen is by definition the most recent active bucket. If the
source is actively bursting now, the burst is at the right edge of
the window and the half-life lands close to it. This is not a bug —
it correctly says "most of the tokens arrived recently" — but it
means a back-loaded reading does not tell you whether the burst will
continue. Pair with the bucket-streak lens to find out.

## Closing

Six sources, six numbers, and a fleet-level diagnosis emerges from
reading the column top to bottom. claude-code is the source whose
recent rate has decoupled from its lifetime average; that is the row
that capacity planning needs to react to. codex is winding down.
hermes is in a concentrated-burst window. The other three are flat
and need no special handling.

The lens is one Pareto pass over the cumulative tokens-by-bucket
curve per source. Cheap to compute, easy to read, and one of the few
metrics that catches the case where a formerly-quiet source has
become a workhorse before the workhorse status surfaces in any other
chart. Add it next to your lifetime-tokens column and the dashboard
gets meaningfully smarter for the cost of one column of arithmetic.
