# Cost concentration: 79% of spend on one model, and the 63% cache rebate

## Lede

A week of `pew` queue data, priced through the local rate table, shows something
the per-event token counters never quite expose: the dollar geometry is wildly
more concentrated than the event geometry. 369 events on one model burned
$22,035.52. 371 events on another model burned $5,853.52. Same order of
magnitude on event count, almost exactly a 4× separation in dollars. And
underneath that, a prompt-cache that is doing more pricing work than the
model picker is — a 63% rebate against the no-cache baseline that, if it
ever regresses, would silently triple the bill before anyone notices the
event count never moved.

## The data

Snapshot from `pew-insights cost`, run against `~/.config/pew/queue.jsonl`
at `2026-04-25T18:29:45Z`, window since `2026-04-18T18:29:44Z` (7-day
trailing):

```
metric                  value
─────────────────  ──────────
estimated cost     $27,911.53
no-cache baseline  $76,214.10
cache savings      $48,302.58

By model
model                   total       input     cached     output  reasoning     $/1M  events
─────────────────  ──────────  ──────────  ─────────  ─────────  ─────────  ───────  ──────
claude-opus-4.7    $22,035.52  $15,395.93  $4,891.22  $1,748.36      $0.00    $5.11     369
gpt-5.4             $5,853.52   $5,282.89    $474.93     $86.64      $9.06    $2.91     371
claude-sonnet-4.6      $22.16      $20.41    $0.7499    $0.9989      $0.00    $2.36       4
gpt-5.2               $0.3132     $0.2264    $0.0511    $0.0127    $0.0231    $1.05       1
gpt-5-nano            $0.0114   $0.007561  $0.001375  $0.000230  $0.002253  $0.1041       1

Unpriced models (add to ~/.config/pew-insights/rates.json)
model             tokens  events
────────────────  ──────  ──────
unknown           29.13M      53
claude-haiku-4.5   3.96M       2

rates source: built-in defaults  (5 models priced)
```

Five priced models, two unpriced families that together account for 33.09M
tokens across 55 events. The headline number — `$27,911.53` — is therefore
already an undercount, and we can bound that undercount: at the cheapest
priced rate ($0.10/1M for `gpt-5-nano`) the unpriced 33.09M tokens add at
least $3.31; at the most expensive ($5.11/1M for `claude-opus-4.7`) they
add up to $169. Real number is almost certainly closer to the latter,
because `claude-haiku-4.5` is in the same family-of-pricing as opus, just
cheaper. Call it a $50–$120 blind spot — small in absolute terms, but a
useful reminder that the cost report has rate-table drift built into it.

## What it means

### The 79% number

`claude-opus-4.7` accounts for `$22,035.52 / $27,911.53 = 78.95%` of
the priced spend. It accounts for `369 / 746 = 49.5%` of priced events.
That gap — half the events, four-fifths of the dollars — is the single
most useful framing to come out of this snapshot, because it tells you
exactly where a token-budget intervention has leverage. If you halved
`claude-opus-4.7` usage (substituting onto `gpt-5.4` at $2.91/1M
input-equivalent), you would knock roughly $11,000 off the weekly
spend. If you halved `gpt-5.4` you would knock off $2,900. The first
intervention is 3.8× as financially valuable as the second, even though
the underlying event counts are essentially identical.

This is the cost-of-routing argument in concrete numbers. The router
treats the two models as substitutable for many requests — they get
roughly the same number of dispatches over the week — but the pricing
geometry doesn't agree. A round-trip on opus is 1.76× more expensive
per 1M tokens (`$5.11 / $2.91`), but more importantly the prompt
sizes that get sent to opus are also larger (we will see this in the
sibling post on prompt-size distribution: opus has a p95 of 22.7M
input tokens vs gpt-5.4's 9.6M). Multiply per-token rate by per-row
volume and you get the 4× separation in dollars from the 1× separation
in events.

The honest read: the routing layer is choosing opus for the heavier
jobs, which is probably correct on a quality basis, but nobody is
*pricing* that choice at the routing decision. The dollar consequence
is only visible in this report, after the fact, with a one-week lag.

### The cache-rebate footnote that isn't a footnote

The other number worth staring at: cache savings of `$48,302.58`
against a no-cache baseline of `$76,214.10`. That is a 63.4% rebate.
Almost two-thirds of the theoretical bill never materialises because
prompt-cache hits drop the per-token rate.

Two observations follow from that.

First, **the cache is doing more pricing work than the model picker**.
A 1.76× rate gap between models is pocket change next to a 2.74× rate
gap between cached and uncached input on the same model. The biggest
financial decision the system makes per request is not "which model"
but "did the prefix match the cache". Anything that perturbs cache
prefixes — system-prompt churn, tool-list churn, context-window
rotation, even retry jitter — has a direct dollar consequence that
will dwarf any model-routing optimisation.

Second, and this is the operationally scary one: **a cache-hit
regression is almost invisible from event counts alone**. If
tomorrow's prompt-cache hit ratio collapsed from its current implied
~63% level to, say, 20%, the event count for `claude-opus-4.7` would
not change. The dollar bill would roughly triple. The only signals
that would fire early are (a) the `cache-hit-ratio` subcommand drift,
which is being tracked in a separate post on the 0500 UTC trough, and
(b) this `cost` subcommand showing the no-cache-baseline-to-actual
ratio collapsing toward 1.0. Neither is wired into a default
dashboard. Both should be.

The breakdown helps locate where the cache is *actually* doing work.
For `claude-opus-4.7` the cached-input bill is `$4,891.22` against an
input bill of `$15,395.93` — so cached input is 24.1% of priced input
spend, but at a much lower per-token rate. Reverse-engineering: the
underlying cached-token *count* is much higher than 24% of input
tokens; it is whatever fraction makes the dollar arithmetic work given
the cache discount. For `gpt-5.4` the cached-input bill is `$474.93`
against `$5,282.89` of input — only 8.2%. The two providers'
caching behaviour is not symmetric, and the dollar split makes that
visible in a way the raw token counts in `cache-hit-ratio` do not.
Opus is leaning on its cache; gpt-5.4 is barely using it. That asymmetry
is itself a research question — is it the model's fault, the prompt
shape's fault, or the routing layer's fault for sending freshly-rotated
prefixes to one provider and stable prefixes to the other?

### Reasoning tokens are a $9 line item

The `reasoning` column is almost entirely zero. `gpt-5.4` shows
`$9.06` of reasoning spend across 371 events. `gpt-5.2` shows
`$0.0231`. `gpt-5-nano` shows `$0.002253`. Everything else is
flat zero. Two things are happening: claude-family models in this
window do not appear to be billing reasoning as a separate token
class (their reasoning is collapsed into output, or the rate table
has it priced at zero), and `gpt-5.4` is the only model where the
reasoning spend is a non-trivial line item — though "non-trivial" is
relative; $9 against $5,800 in total `gpt-5.4` spend is 0.16%.
Reasoning tokens are not the cost story for this week. They might
become one if the workload shifts toward problems where reasoning
chains lengthen.

### The unpriced tail is the audit risk

`unknown` and `claude-haiku-4.5` together represent 33.09M tokens
across 55 events that the rate table cannot price. The `unknown`
bucket is the more concerning one: it is 53 events, which is more
events than `claude-sonnet-4.6` (4), `gpt-5.2` (1), and `gpt-5-nano`
(1) combined. Roughly 7% of total event volume is on a model the
report cannot identify well enough to price. That's not a rounding
error. That's a class of activity that is structurally invisible to
this dashboard.

Two failure modes to consider. (1) The model identifier was logged
incorrectly — a transport-layer bug or a rate-card name mismatch —
which means the events are real, but the dollar attribution is off
by whatever the unknown model's rate is. (2) The model is genuinely
new (a fresh provider release, an experimental endpoint) and the
rate table simply hasn't caught up. In either case the operational
fix is the same: extend `~/.config/pew-insights/rates.json` until the
unpriced bucket is empty, and treat any non-zero unpriced count in
this report as a yellow alert. The current state of "5 models priced"
should be "7 models priced" before next week's snapshot.

### Composite read

The 79/63/8 numbers fit together into a single sentence: **one model
takes four-fifths of the dollars, the cache rebates two-thirds of the
theoretical bill, and 8% of events are running on models the rate
table cannot identify.** Each of those three numbers is a different
class of operational risk — concentration risk, cache-regression
risk, and observability risk — and a one-screen cost report that
fired on any of them moving by 5pp week-over-week would be more
operationally useful than every other dashboard combined.

## Caveats / what would falsify

The whole calculation is built on a static rate table. If the
provider rates have moved since the built-in defaults were last
updated, every dollar figure here is wrong by some multiplier. The
*ratios* (79%, 63%, model-vs-model splits) are more robust than
the absolute dollars, because rate updates tend to move all models
in the same direction at once, but a one-sided rate change (e.g.
opus gets cheaper, gpt-5.4 doesn't) would invert the concentration
story.

The "cache savings" line is computed as `(no-cache-baseline) - (actual)`,
which assumes the no-cache baseline is the same prompt sent without
the cache discount. It does not account for the alternative world in
which the absence of cache pressure would cause the operator to send
*fewer* big prompts. In other words: if you turned the cache off,
total spend probably wouldn't be $76k — the operator would adapt and
shrink the prompts. So treat $48,302.58 as an upper-bound on the
cache's value, not a precise figure.

Event count parity between opus and gpt-5.4 (369 vs 371) is striking
enough that it might be a routing artefact — e.g. an A/B router
splitting traffic 50/50 deliberately. If so, the "halve opus" lever
is not actually available without breaking whatever experiment the
router is running. Worth confirming by looking at the source-attribution
breakdown before treating the 79% concentration as actionable.

Finally, the report would be falsified by next week's snapshot
showing one of: (a) opus dropping below 60% of spend without a
corresponding cache-hit improvement (would imply a real workload
shift), (b) the cache-savings ratio dropping below 50% (would imply
a cache-prefix regression), or (c) the unpriced bucket growing
above 15% of events (would imply rate-table debt is compounding).

## Citations

- `pew-insights cost` snapshot, captured `2026-04-25T18:29:45.968Z`,
  window-since `2026-04-18T18:29:44.584Z`, source `~/.config/pew/queue.jsonl`.
- Tool: `pew-insights@0.5.4`, built from `~/Projects/Bojun-Vvibe/pew-insights`,
  invoked as `node dist/cli.js cost`.
- Cross-reference: sibling post `2026-04-26-cache-hit-by-hour-the-32-pp-source-spread-and-the-0500-utc-trough.md`
  for the cache-hit-ratio drift signal that this cost analysis depends on.
- Rate table: built-in defaults, 5 models priced. Extension target:
  `~/.config/pew-insights/rates.json` (currently absent).
