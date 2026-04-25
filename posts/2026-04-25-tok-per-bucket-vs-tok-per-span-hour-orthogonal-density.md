# Two density numbers, two different questions: tok/bucket vs tok/span-hr

*2026-04-25 — pew-insights v0.4.64, `model-tenure` subcommand (commit `a33ada4`)*

The `model-tenure` lens prints two density columns next to each other and
they are easy to confuse. They are not the same number divided by different
constants. They answer two genuinely different questions about the same
model, and once you separate them, a lot of "which model is heaviest?"
debates become well-defined.

This is a short note about what those two columns mean, why they
disagree, and how to use the disagreement.

## The two columns

From the smoke output of `node dist/cli.js model-tenure --top 5 --sort
density` against my local `~/.config/pew/queue.jsonl`, repository at
HEAD `a33ada4`, version `0.4.64`:

```
as of: 2026-04-25T05:37:24.828Z
models: 15 (shown 5)    active-buckets: 1,240    tokens: 8,387,867,906
sort: density

model               span-hr  active-buckets  tokens          tok/bucket  tok/span-hr
claude-opus-4.7     195.0    274             4,679,970,507   17,080,184  23,999,849
gpt-5.4             859.0    374             2,477,988,324   6,625,637   2,884,736
claude-opus-4.6.1m  1044.0   167             1,108,978,665   6,640,591   1,062,240
gpt-5.2             0.0      1               299,605         299,605     299,605
unknown             176.5    56              35,575,800      635,282     201,563
```

`tok/bucket` is `tokens / activeBuckets`. It answers: when the model was
active, how many tokens did it consume per active hour? Call this
**intensity-when-on**.

`tok/span-hr` is `tokens / spanHours`. It answers: amortized across the
entire calendar window from first sighting to last sighting, how many
tokens per clock hour? Call this **amortized-density**.

The two columns coincide only when the model was active in every hour of
its span (no idle gaps). The ratio `tok/span-hr ÷ tok/bucket` is exactly
the duty cycle: the fraction of the span during which the model was
actually doing work.

## Three regimes the columns expose

**claude-opus-4.7: 195 hours of life, 274 active buckets.** That's
already the impossible math from the previous note (buckets are
half-hour, span is hour, so 274 buckets in 195 hours means the model
was active in roughly 70% of all available half-hour slots, or
equivalently it had a duty cycle above 1.0 if you naively divide). The
two density numbers reflect this: tok/bucket is 17.1M, tok/span-hr is
24.0M. The amortized number is *higher* than the per-bucket number.
Read this as "this model has been so consistently on, that even
spreading its tokens over every wall-clock hour of its life still
gives you a number bigger than what it does in a typical active hour."
That's the signature of a model with no real downtime.

**gpt-5.4: 859 hours of life, 374 active buckets.** tok/bucket is
6.63M, tok/span-hr is 2.88M. The amortized number is roughly
43% of the per-bucket number. So gpt-5.4 has been around for
4.4× longer than claude-opus-4.7 (859 vs 195 hours) but is only
active in about 22% of half-hour slots within that span. Lots of
gaps, decent intensity when on. This is the "long resident, sparse
work" pattern.

**claude-opus-4.6.1m: 1044 hours of life, 167 active buckets.**
tok/bucket is 6.64M (basically identical to gpt-5.4 — the models
do similarly-sized work when they do it), but tok/span-hr collapses
to 1.06M. So the amortized number is ~16% of the per-bucket number.
This model has been around the longest but is mostly idle now — its
`lastSeen` was eight days before the snapshot, which `model-tenure`
doesn't surface but you can derive from `firstSeen + spanHours`.
Given the snapshot date 2026-04-25 and `firstSeen` 2026-03-04T14:00,
its span ends 2026-04-17T02:00, exactly the day claude-opus-4.7's
span begins. So we're looking at a clean handoff: opus-4.6.1m
deprecated, opus-4.7 picked up the work.

The two columns together expose this handoff. tok/bucket alone
would say "they do the same work" (6.64M vs 6.63M, near-identical).
tok/span-hr alone would say "opus-4.6.1m is dying" (1.06M vs 23.99M).
Together they say: the work was the same per active hour; the
schedule changed.

## Why the difference matters for capacity questions

Suppose a finance partner asks: "how much capacity should we
provision for claude-opus-4.6.1m next quarter?" Two density numbers
give two very different answers.

If you use **tok/span-hr (1.06M)** — the amortized number — you'd
size for ~1M tokens per hour, sustained. That's a small reservation
and it would be totally inadequate any time the model actually got
work, which is 16% of the time at 6.64M tokens per active hour.

If you use **tok/bucket (6.64M)** — the intensity-when-on — you'd
size for ~6.6M tokens per hour. That's enough for active hours
but is six times overprovisioned during the 84% idle gap. You'd
buy reserved capacity that sits dark.

The right answer depends on the contract. A **reserved-throughput
contract** (you pay for hours, you get the headroom) should price
against tok/bucket — you need that capacity when you need it,
period. A **pay-per-token contract** should think in tok/span-hr
because that's what your monthly bill amortizes to. The columns
are not interchangeable; they price against different cost
structures.

For the active model (claude-opus-4.7) this matters less because
the duty cycle is already saturated — both numbers are
approximately the right capacity. For the dying model
(claude-opus-4.6.1m) it matters enormously because the gap
between them is 6×.

## The duty cycle is the missing column

The lens does not print duty cycle as its own column, but it's
trivial to derive: `activeBuckets / (2 × spanHours)` if the bucket
width is half an hour, or `activeBuckets / spanHours` if it's a
full hour. From the help text, the bucket width is "whatever pew
emits" — the upstream queue.jsonl rows determine it, not the lens.
For my queue, the bucket key is half-hourly (per the test name
"half-hour bucket honesty" in commit `23b2a50`), so:

- claude-opus-4.7: 274 / (2 × 195) = 70.3%
- gpt-5.4: 374 / (2 × 859) = 21.8%
- claude-opus-4.6.1m: 167 / (2 × 1044) = 8.0%

These are the duty cycles. They are the bridge between the two
density numbers. `tok/span-hr = duty_cycle × tok/bucket × 2` (the
2 accounts for the half-hour vs hour unit mismatch).

For claude-opus-4.7: 0.703 × 17.08M × 2 = 24.01M. Close to the
reported 24.0M. ✓

For gpt-5.4: 0.218 × 6.63M × 2 = 2.89M. Reported 2.88M. ✓

For claude-opus-4.6.1m: 0.080 × 6.64M × 2 = 1.06M. Reported 1.06M. ✓

So the lens doesn't print duty cycle because you can recover it
exactly from the columns it does print. But: very few people will
recover it by inspection. Showing both density numbers is the
lens's way of forcing the reader to notice that duty cycle is the
hidden third axis.

## What "sort: density" actually sorts on

The `--sort density` flag (added in commit `a33ada4`) sorts on
`tok/span-hr`, the amortized-density column, not on `tok/bucket`.
This is a defensible choice — amortized density is closer to "how
much does this model cost me on the calendar" than "how heavy is
this model when it works" — but it's worth knowing because if
you're trying to find your peakiest model, sorting by density
will mislead you. You want to sort by `tokens` or by `tok/bucket`,
not by `tok/span-hr`.

The five other sort keys (`span`, `active`, `tokens`, plus
`firstSeen` and `lastSeen` if I'm reading the changelog right)
each answer a different question:

- `--sort span` ranks by how long the model has existed in your
  data. Useful for cohort analysis: which models have been around
  the longest?
- `--sort active` ranks by how many distinct hour buckets the
  model touched. Useful for footprint: which models show up the
  most often?
- `--sort tokens` ranks by total token consumption. The spend
  ranking. The one most people want by default.
- `--sort density` ranks by amortized tok/span-hr. The
  cost-per-calendar-hour ranking, which is closer to a billing
  rate than to a workload signal.

The default sort (no flag) is `tokens`. The fact that `density`
is its own sort key is a hint that the maintainers think the
distinction is important enough to be a first-class option, not
just a derived column you eyeball.

## How this composes with the other lenses

`bucket-intensity` (the histogram lens) tells you the *shape* of
work in active buckets. `model-tenure` tells you how that shape is
distributed across calendar time. They are orthogonal in exactly
the way "what kind of work" and "when does it happen" are
orthogonal.

For claude-opus-4.7 specifically, the joint reading is:
heavy-lift bucket histogram (43% of buckets in the `[10M, +inf)`
band per the previous note) plus 70% duty cycle plus tok/span-hr
≈ tok/bucket. Translation: this model is on most of the time,
and when it's on, it's doing big work. There is no exploratory
mode in evidence; there is barely an idle mode.

For claude-opus-4.6.1m the joint reading is: similarly heavy-lift
histogram (35 of 167 buckets in the top band) but 8% duty cycle
and the amortized density 1/6 of the active density. Translation:
when this model is on, it does similar work to claude-opus-4.7,
but it's mostly off. Aging out.

The lens combination is the point. Either lens alone could
mislead you. `bucket-intensity` alone would say opus-4.6.1m and
opus-4.7 are similar (both heavy-lift). `model-tenure` alone
would say opus-4.6.1m is dying (low density) but wouldn't tell
you whether the small amount of work it still does is exploratory
or heavy. Together, they say: opus-4.6.1m is in maintenance mode,
doing occasional heavy jobs, while opus-4.7 has inherited the
sustained heavy load.

## What I'd add if I were extending the lens

A duty-cycle column. Yes, it's derivable, but no, no one will
derive it. Adding it would make the unit mismatch (half-hour
buckets vs hour spans) impossible to gloss over and would let
people sort on it directly — "show me the busiest models by
fraction of active hours" is a cleanly different question from
"show me the highest amortized density."

A `--bucket-width` flag (or auto-detect) so the half-hour
versus full-hour ambiguity is stated, not implicit. Right now
the lens header documents the convention parenthetically but
the column units are silent on it.

A second density variant: `tok/active-hour` (treating each
active *bucket* as an hour-equivalent). For a half-hour-bucket
queue, this would be `tokens / (activeBuckets × 0.5)`, which
for opus-4.6.1m would be 13.28M — twice the printed tok/bucket.
That number answers the question "if I extrapolated active
half-hours to active hours, what's the rate?" which is what
people usually mean when they ask "how fast does this model
burn tokens when it's working?" It's not currently on the
table; right now you have to think about whether tok/bucket is
in tokens-per-half-hour or tokens-per-hour, and the answer is
the former, but most readers will assume the latter.

## Reproduce

```
git -C ~/Projects/Bojun-Vvibe/pew-insights rev-parse --short HEAD
# a33ada4
node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js model-tenure \
  --top 5 --sort density
```

The `--top` and `--sort` flags landed in v0.4.64 (commit `a33ada4`).
The lens itself shipped in v0.4.63 (commit `93ab01a`). The "half-hour
bucket honesty" tests are in commit `23b2a50`. If you want to verify
the duty-cycle derivation, JSON output mode is on the same subcommand
and gives you `firstSeen` / `lastSeen` / `spanHours` / `activeBuckets`
as raw numbers — divide and check.

The single takeaway: when a model-tenure-style report shows you two
density columns, *the gap between them is the story*. tok/bucket near
tok/span-hr means the model is on most of the time. tok/bucket much
greater than tok/span-hr means the model is mostly idle. The columns
are telling you a duty-cycle truth that the totals row will smear.
