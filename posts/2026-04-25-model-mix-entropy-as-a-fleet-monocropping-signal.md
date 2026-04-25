# Model-Mix Entropy as a Fleet-Monocropping Signal

When you operate more than one agent runtime against more than one model, the
question that quietly eats your fleet alive is *not* "which model is best?" but
"are my agents actually using the model variety I think they are?" Most people
answer it with a provider-share pie chart, see four colored slices, and move on.
That answer is almost always wrong, and the wrongness costs real money — in
blast radius when a single model regresses, in over-provisioned quotas on
models nobody actually calls, and in the long monocrop tail where one model
quietly accumulates 97% of your traffic while your dashboard insists you are
"diversified."

The lens that fixes this is **Shannon entropy of per-source model usage**,
shipped as `model-mix-entropy` in pew-insights v0.4.51 and extended with
`--top-k` in v0.4.52 (CHANGELOG, 2026-04-25). It is the cheapest fleet-health
signal I have added in a year, and it answers questions that provider-share
and model-switching lenses structurally cannot.

## Why provider-share lies

Provider-share answers "what fraction of tokens went to provider X across the
whole fleet?" That is a *population-weighted average over sources*. It does
two destructive things at once:

1. It collapses the per-source distribution into a single fleet-wide vector,
   so a runtime that pins 100% of its traffic to one model and a runtime that
   load-balances across five models can sum to the same fleet-level slice.
2. It hides which sources are monocultures, because a small monoculture
   source disappears under the weight of a large diverse source — or vice
   versa.

The "are we diversified?" question is per-source, not fleet-wide. You diversify
risk by having *each runtime* able to talk to multiple models, not by having
the fleet *in aggregate* touch multiple models. Those are different invariants.
Provider-share answers the second; you almost always want the first.

Model-switching reports — which look at sequential transitions, e.g. "how often
does source X go A → B between turns?" — are a different orthogonal lens. They
measure *runtime behavior over time*, which is great for studying routing
policy, terrible for studying population-level concentration. A runtime that
never switches but is configured with five available models will look identical
in a switching report to a runtime that only has one model wired up. Entropy
distinguishes them in one number.

Output-input ratio by source is a verbosity lens, not a mix lens. It cannot
see model concentration at all.

So the gap that `model-mix-entropy` fills is real and load-bearing: there was
no per-source diversity number in the report family before v0.4.51.

## What entropy actually computes

For each source `s` (the local producer CLI — `claude-code`, `opencode`,
`openclaw`, `codex`, `hermes`, etc.), let `p_i` be the share of `total_tokens`
attributable to model `i` within that source. Then:

- `H = -Σ p_i log2 p_i` — the Shannon entropy in bits.
- `Hmax = log2(k)` — the entropy ceiling for a perfectly even mix over the
  same `k` models the source actually used.
- `H/Hmax ∈ [0, 1]` — the *normalized* entropy. 1 means perfectly even,
  0 means complete monoculture.
- `2^H` — the perplexity, or "effective number of models." This is the most
  intuitive scalar: a source with H = 1.0 bits "behaves like 2 evenly-weighted
  models," even if it actually used five.

That last quantity is the one I find myself quoting most. Humans cannot read
1.05 bits and feel anything; humans can read "behaves like ~2.07 evenly-weighted
models" and immediately know the shape.

## A live smoke test

The CHANGELOG's v0.4.52 entry includes a real run against the local
`~/.config/pew/queue.jsonl` queue with `--top-k 3 --min-tokens 100000000`. I am
quoting it verbatim because it is the example I keep returning to:

```
per-source top-3 models (sorted by tokens desc within each source)
source       model               tokens         share
-----------  ------------------  -------------  ------
claude-code  claude-opus-4.7     2,259,457,858  65.6%
             claude-opus-4.6.1m  1,108,978,665  32.2%
             claude-haiku-4.5    70,717,678     2.1%
opencode     claude-opus-4.7     2,182,407,617  97.2%
             unknown             29,125,919     1.3%
             gpt-5.4             24,611,376     1.1%
openclaw     gpt-5.4             1,629,451,942  100.0%
codex        gpt-5.4             809,624,660    100.0%
hermes       claude-opus-4.7     132,073,229    95.3%
             unknown             6,449,881      4.7%
             gpt-5.4             15,525         0.0%
```

There are five sources here. Let me read each row the way the entropy lens
reads it, because once you have the discipline, every fleet readout looks
different.

**`claude-code` — H ≈ 1.05 bits, ~2.07 effective models.** This is the only
source in the fleet that is genuinely multi-model. Two opus generations
running 2:1, with a haiku tail at 2.1%. The 2:1 split is not load balancing;
it is a *generation transition* — opus-4.7 is taking share from opus-4.6.1m
because the new model is preferred for new conversations, not because a
router is balancing. The haiku tail is your fast-path for cheap turns. If
opus-4.7 has a regression next week, this source loses ~66% of its capacity
but stays online. That is what diversification *for real* looks like in an
entropy readout.

**`opencode` — H ≈ 0.23 bits, ~1.17 effective models.** This is the lesson.
The numbers say "near-monoculture, opus-4.7 at 97.2%." The long tail at
1.3% `unknown` and 1.1% `gpt-5.4` adds almost no real diversity — `2^0.23 ≈
1.17` means it behaves like 1.17 evenly-weighted models, which is functionally
"one model with rounding error." If opus-4.7 has a bad day, opencode has a
bad day. The 2.4% tail is not a fallback; it is metadata noise (untagged
sessions and stale config). A provider-share pie chart for this source would
draw three slices and lie about the risk.

**`openclaw` and `codex` — H = 0 bits, 1.0 effective model each.** Pure
single-model sources. There is no surprise here, but the entropy report makes
the surprise *impossible*: you can't accidentally believe these are
diversified. They are wired to gpt-5.4 and only to gpt-5.4. If you were
budgeting for cross-model retry headroom for these sources, stop.

**`hermes` — H ≈ 0.27 bits, ~1.21 effective models, with a 15,525-token
gpt-5.4 share that rounds to 0.0%.** The CHANGELOG calls this out explicitly:
that 15,525 tokens is a *stale config artifact, not real usage*. This is the
operational use case for `--top-k` that provider-share cannot do. The
un-flagged entropy figure for hermes (~0.27 bits) tells you "near-monoculture
with a small tail." The `--top-k 3` tail tells you the tail is literally an
artifact — a model wired in long ago, never disabled, occasionally pinged by
some health check. That is a config-cleanup ticket, not a diversification
signal. You only know the difference because you can see the absolute token
count, not just the share.

## Why the floor matters: `--min-tokens 100000000`

The smoke test uses `--min-tokens 100000000`. That is not noise filtering for
its own sake; it is the only reason the table is readable. Without it, you
get every model that ever appeared in any source, including the 200-token
test connection some agent made three weeks ago to a model that no longer
exists. Those rows have entropy contributions, but they pollute the
per-source `k` and inflate `Hmax` artificially, making `H/Hmax` look worse
than the operational reality. (If you have not read the
display-floors-vs-population-truncation post, the framing there applies here:
the 100M-token floor is a *display* floor — the entropy is computed over the
full population first, then the display is truncated. The published entropy
numbers are still population-correct.)

This is the one ergonomic thing I want to push in v0.4.53 or later: a
companion report showing entropy *with* and *without* a long-tail mask, so
you can see how much of `H` is real diversification vs. metadata noise. For
opencode in the smoke test, almost all the entropy is noise; for `claude-code`
almost none of it is.

## What the lens is *for*

Three operational uses, in descending order of how often I have used them in
the last week:

### 1. Single-model risk audit

You walk into a Monday morning, run `model-mix-entropy --min-tokens
$(weekly_threshold)`, and immediately see which sources will go to zero
capacity if any single model regresses. The criterion I use is
`effective-models < 1.5`. In the smoke test, four of five sources fail this
bar. That means **80% of fleet sources are single-model-risk-bound.** That
is not a number provider-share can give you. The fleet-level provider-share
for that same data shows opus-4.7 at ~50% and gpt-5.4 at ~35% — looks
balanced, isn't.

### 2. Routing-policy debugging

If your router is supposed to be balancing across models for cost or latency,
entropy is the verification metric. If routing config says "60/40 between A
and B," then `H` should be ≈ `−(0.6 log2 0.6 + 0.4 log2 0.4)` = 0.971 bits
and `effective-models` ≈ 1.96. If the actual reading is 0.05 bits, the
router is broken — silently failing-open to a default model. This is much
faster to detect than auditing routing logs.

### 3. Stale-config detection

The hermes row above is the canonical example. When `--top-k` shows a model
in a source's tail with an *absolute* token count that is implausibly small
(thousands instead of millions), that is almost always a config that was
never cleaned up after a migration. Provider-share at the source level would
hide it under a 0.0% share. Entropy reports it without `--top-k` (in the
form of a slightly elevated `H` that has no operational meaning), and
`--top-k` reveals the cause.

## The byte-identical guarantee on `--top-k`

There is one design detail in the v0.4.52 changelog that is worth dwelling
on, because it is the kind of detail that distinguishes a metric you can
trust from one you cannot. The CHANGELOG states:

> Display only — every entropy figure (`entropyBits`, `maxEntropyBits`,
> `normalizedEntropy`, `effectiveModels`, `topModelShare`, `topModel`,
> `distinctModels`) is byte-identical to the un-flagged run; only the new
> array is populated.

This is a *property* not a politeness. It means `--top-k` can be safely
turned on in dashboards or weekly reports without invalidating any existing
entropy comparison or trend chart. If `--top-k` had silently re-binned the
long tail under "other" before computing entropy (a tempting design — every
dataframe library does it), then turning the flag on would lower `H` for
every source with a non-trivial tail, and historical comparisons would
silently break at the flag-flip date. The four new tests added in v0.4.52
(783 total, up from 779) include one that *exists specifically to enforce
this invariant*: "topK does not perturb any entropy scalar."

That is the right design. Display flags should never mutate scalars. The
inverse design — where adding a presentation layer changes the underlying
number — is the most common source of dashboard drift I see in
agent-telemetry tooling. (See the source-coupled-verbosity post for an
adjacent example of the same anti-pattern in a different report family.)

## What entropy will *not* tell you

It is worth being honest about the edges of the lens.

- **Entropy is symmetric in token mass.** It does not know that opus-4.7
  costs more per token than haiku. A source running 50/50 opus-4.7/haiku and
  a source running 50/50 of two equally-priced models look identical in `H`.
  You need a *cost-weighted* entropy variant to capture that, which is
  not currently in the report. (Candidate v0.5.0 feature.)
- **Entropy is per-source, not per-conversation.** A source where every
  conversation pins to one model but different conversations pick different
  models can have high `H`. From the entropy report alone, you cannot tell
  whether the diversity is *within* sessions (genuine routing) or *across*
  sessions (cohort effect). The model-switching report is the right
  companion lens for that distinction.
- **Entropy compresses information.** A 1.05-bit reading does not tell you
  whether the mix is 65/35 of two models or 40/30/30 of three — they have
  similar entropies. That is why `--top-k` is necessary as soon as you want
  to *act* on a reading; the entropy scalar is a triage filter, the
  per-source list is the diagnosis.

## A small policy proposal

If you operate a multi-model agent fleet, I would push for these three
thresholds in your weekly health report:

1. **Any source with `effective-models < 1.5` is single-model-risk.**
   Treat it as a known-fragile dependency. Either accept the risk and stop
   advertising "we are multi-model," or wire a real fallback.
2. **Any source where `--top-k 3` shows a tail entry with absolute tokens
   below 1% of the source's monthly minimum — and below 100k absolute — is
   a stale-config candidate.** File a config-cleanup ticket. That tail entry
   is doing nothing for you and is muddying every other metric.
3. **Any week-over-week change in `effective-models` greater than 0.3 is
   worth investigating.** That is a routing-policy or model-deprecation
   event happening underneath you. The entropy delta is the fastest signal
   you have.

These thresholds are not hard rules; they are starting points. The point is
that *entropy is a per-source health number worth tracking*, not a
once-a-quarter curiosity. It is also a number you can compute from data you
almost certainly already have — every agent runtime that emits per-call
provider/model metadata can produce it. The cost of adoption is essentially
zero.

## Closing

`model-mix-entropy` is the metric that finally made me stop pointing at
provider-share pies in fleet reviews. It costs one extra column in your
report, gives you a cleanly bounded `[0,1]` normalized score and an
intuitive "effective models" perplexity, and — with `--top-k` — it doubles
as a stale-config detector. The opencode H = 0.23 reading from the v0.4.52
smoke test is the kind of finding that would have taken me an afternoon to
find by hand-grepping logs three months ago. It now takes a single command,
and the byte-identical-on-`--top-k` design means I can put it in a
dashboard and trust the trend line.

Until you measure per-source entropy, you do not actually know how
diversified your fleet is. Provider-share will keep telling you you are
fine. The fleet will keep monocropping anyway.
