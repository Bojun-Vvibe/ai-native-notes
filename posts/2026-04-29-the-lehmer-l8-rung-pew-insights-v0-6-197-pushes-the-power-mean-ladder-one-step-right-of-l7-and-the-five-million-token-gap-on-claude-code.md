# The Lehmer L_8 rung: pew-insights v0.6.197 pushes the integer power-mean ladder one step right of L_7, and the +5,064,831-token l8L7Gap on claude-code that the new lens exposes

Published 2026-04-29.

## What shipped

`pew-insights` cut **v0.6.197** on 2026-04-29 with a single new
subcommand: `source-row-token-lehmer-8-mean`. It computes, per source,
the **Lehmer mean of order 8** of the per-row `total_tokens`
distribution sitting in `~/.config/pew/queue.jsonl`. The closed-form
definition is the one you would expect if you have been following the
ladder this week:

    L_8 = ( sum_{i=1..n} x_i^8 ) / ( sum_{i=1..n} x_i^7 )

That is, divide the sum of eighth powers by the sum of seventh powers.
Equivalently — and this is the framing the subcommand's `--help`
leans on — L_8 is the `x^7`-self-weighted arithmetic mean of `x`:
each row weights itself by its own seventh power before contributing
to the numerator, and by its own seventh power again in the
denominator, so the result is always a value between min and max of
the positive part of the sample, but pulled aggressively toward the
largest rows.

L_8 is the natural one-step-right extension of v0.6.196's L_7 lens
that the `posts/_meta/` corpus has been tracking since the L_3 rung
landed early in the week. By Lehmer monotonicity — a textbook
inequality on Lehmer means — `L_p <= L_q` for any non-negative
sample whenever `p <= q`, with equality iff the positive part of
the sample is constant. So the ladder of integer-order Lehmer means
shipped in `pew-insights` to date now reads, end to end:

    L_-3 <= L_-2 <= L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3 <= L_4 <= L_5 <= L_6 <= L_7 <= L_8

That is fourteen distinct location lenses, all computable from the
same per-source row stream, all guaranteed to satisfy a chain of
weak inequalities, and all carrying a different bias toward
small-tail vs. large-tail rows. The ladder is the underlying
mathematical object; each `pew-insights` release nails one more rung
to the wall.

## Live smoke against the queue

I ran the new subcommand against my live queue at the moment the
patch landed:

    cd ~/Projects/Bojun-Vvibe/pew-insights
    node dist/cli.js source-row-token-lehmer-8-mean ~/.config/pew/queue.jsonl

The header reports `as of: 2026-04-28T18:16:07.813Z`, `sources: 6`,
`rows: 1,850`, `dropped: 0` across all the standard gates
(bad `hour_start`, bad `total_tokens`, negative `total_tokens`,
source filter, min-rows, all-zero sources where `sum(x^7) = 0`,
min-lehmer-8-mean, and top cap). The full per-source table, with
the sixth source name redacted to `vscode-XXX` per local policy,
is:

    source        rows  mean         L_6          L_7          L_8          l8-l7        l8-l6          l8-mean
    ------------  ----  -----------  -----------  -----------  -----------  -----------  -------------  ------------
    claude-code   299   11512995.95  83284537.01  90021322.41  95086154.02  +5064831.62  +11801617.02   +83573158.08
    opencode      404   10427477.11  56418650.29  58068854.52  59343299.38  +1274444.86  +2924649.09    +48915822.27
    codex         64    12650385.31  52005666.14  53900819.55  55199228.84  +1298409.29  +3193562.70    +42548843.53
    openclaw      510   3847139.06   38459510.72  39962261.55  40978107.44  +1015845.89  +2518596.71    +37130968.38
    hermes        240   785371.66    4679917.34   5084797.18   5368276.84   +283479.66   +688359.50     +4582905.18
    vscode-XXX    333   5662.84      161923.95    164846.27    166889.44    +2043.17     +4965.49       +161226.60

Two free monotonicity checks fall out of this table immediately, both
required to hold by Lehmer's theorem: every `l8-l7` value is strictly
positive, and every `l8-l6` value is strictly larger than its
corresponding `l7-l6` value would be. Both checks pass on all six
sources. If either had failed, the patch would have either
introduced a numerical bug or violated the underlying inequality —
and both failure modes would be regressions worth shipping a hotfix
for.

## What L_8 reveals that L_7 did not

The interesting structural fact in this table is that the
**l8L7Gap** column ranges across more than three orders of
magnitude: from `+2,043.17` on `vscode-XXX` (which is a small-token
source where every row is tiny and the seventh-power weighting has
almost nothing to amplify) all the way up to `+5,064,831.62` on
`claude-code`. That number — five million tokens of additional pull
on the location estimate when you go from sixth-power weighting to
seventh-power weighting — is the headline.

To put it in perspective: `claude-code`'s plain arithmetic mean is
`11,512,995.95`. Its L_7 is `90,021,322.41` — already 7.82× the
arithmetic mean, because the source is heavy-tailed and L_7
aggressively weights large rows by their own sixth power. Going from
L_7 to L_8 adds another **5.62%** on top of that already-amplified
location, an absolute increment that is **~44% the size of the
arithmetic mean itself**. The marginal lens on `claude-code` between
L_7 and L_8 is, in absolute terms, comparable to a whole independent
mean-sized object. The largest rows in `claude-code`'s distribution
are doing meaningful work all the way up to the eighth power before
they stop materially shifting the location estimate.

By contrast, on `vscode-XXX` the same L_7→L_8 step adds `+2,043.17`
tokens against an L_7 of `164,846.27` — a 1.24% pull. The rows in
`vscode-XXX` are so uniformly small that even at the seventh power
they have nothing distinctive to contribute that the sixth power did
not already extract. The lens stops doing useful work very quickly
on that source, which is itself a structural fact: there is no fat
tail to amplify because there are no exceptionally large rows.

## The cross-source spread at the L_8 rung

Looking at the L_8 column directly — `95,086,154` for `claude-code`
down to `166,889` for `vscode-XXX` — the cross-source spread ratio
is **569.7×**. That is materially compressed compared to the
L_3→L_6 rungs the corpus was tracking earlier this week (where
the spread was approaching 17,000× at L_-1 levels in the negative
direction), and the compression is monotone with `p` in a specific
direction: as `p` increases, the largest row in each source
dominates more, and the comparison between sources collapses toward
a comparison of their largest rows. In the limit as `p → infinity`,
L_p approaches the maximum of the sample. So L_8 is approaching the
"max" lens but is not yet there — there is still meaningful
distinction between sources beyond just "what is your biggest row".

The l8-l6 spread tells a similar but distinct story. `claude-code`'s
l8L6Gap of `+11,801,617.02` is **2,375×** the corresponding gap on
`vscode-XXX` (`+4,965.49`). Two ladder steps to the right of L_6
amplify the absolute pull on heavy-tailed sources by an enormous
factor, while on light-tailed sources the same two steps barely
move the estimate at all. This is exactly the kind of asymmetric
behavior that motivates having a ladder rather than picking a
single "right" mean: each rung answers a slightly different
question about the shape of the per-row distribution, and the
gaps between rungs are themselves the answer to a meta-question
about how concentrated the source's tail is.

## The l8-mean column as a tail-weight signal

Possibly the most operationally useful column in the new output is
`l8-mean`, the absolute gap between L_8 and the plain arithmetic
mean. On `claude-code` this is `+83,573,158.08` — a number whose
order of magnitude (`8.36e7`) is itself a fingerprint of the
source's tail. The interpretation is direct: if you replaced
equal-weight averaging of `claude-code` rows with seventh-power
self-weighting, the location estimate moves by 83.6 million tokens.
On `vscode-XXX`, the same substitution moves the estimate by
`+161,226.60` — five orders of magnitude smaller. The l8-mean
column therefore acts as a **bounded tail-weight diagnostic**: it
is always non-negative (Lehmer monotonicity again), always zero iff
the positive part of the source is constant, and its magnitude
scales with how aggressively the source's tail can pull the
location around.

The same diagnostic at lower rungs (l3-mean, l4-mean, l5-mean) gives
a coarser version of this signal; the L_8 version is the most
amplified one shipped to date. If a source's l8-mean ever crosses
its arithmetic mean by some operationally-meaningful multiple
(say 5× or 10×), that source has a tail that materially distorts
average-based reasoning, and any downstream metric that assumes
"this source's typical row is around `mean`" should be inspected
for tail leakage.

## Cardinal-ordinal disagreement at the L_8 rung

Sorting sources by L_8 in descending order produces:
`claude-code` (95.1M), `opencode` (59.3M), `codex` (55.2M),
`openclaw` (41.0M), `hermes` (5.4M), `vscode-XXX` (167K).

Sorting the same six sources by **plain arithmetic mean** in
descending order produces a subtly different ordering:
`codex` (12.65M), `claude-code` (11.51M), `opencode` (10.43M),
`openclaw` (3.85M), `hermes` (785K), `vscode-XXX` (5.66K).

Notice the swap at the top: `codex` is the highest-arithmetic-mean
source (with `n=64` rows it is also the smallest cohort, which makes
its mean noisier), while `claude-code` is the highest-L_8 source.
This is a **cardinal vs. ordinal disagreement** and it is structural,
not a glitch. The arithmetic mean weights every row equally, so a
small cohort with consistently large rows can edge out a larger
cohort with a few enormous bursts. The L_8 lens weights each row by
its own seventh power, which means a single 50M-token row in
`claude-code` outweighs a hundred 10M-token rows in `codex` by an
overwhelming margin. The two lenses are answering different
questions, and for sources with heavy-tailed distributions the
answers will not align.

This is why the ladder matters as a complete object rather than
as a list of independently-interesting subcommands. You get to pick
your lens based on which question you are asking — "what does a
typical row look like", "what does the distribution look like
under squared loss", "what does the distribution look like under
extreme tail dominance" — and the ladder ensures the answers
satisfy a known set of inequalities that makes them comparable.

## What the ladder cannot tell you

L_8 is **scale-equivariant** — multiply every row by `c > 0` and
L_8 multiplies by `c` — but **NOT translation-equivariant**: add
a constant `k` to every row and L_8 does not increase by exactly
`k`. That non-translation-equivariance is shared with every Lehmer
mean other than L_1 (which is the arithmetic mean), and it is the
reason the Lehmer family is fundamentally different from the L-
estimator family (mean, trim25, midhinge, trimean, median) that
shipped to `pew-insights` over the prior weeks. L-estimators are
linear combinations of order statistics and are translation-
equivariant by construction; Lehmer means at orders other than 1
are not.

If a downstream consumer wants a translation-equivariant location
estimate — say, because they are computing differences between
locations across two snapshots and want the differences to be
invariant under a shared additive offset — they need to use the
arithmetic mean, the median, or another L-estimator, not a Lehmer
mean. The ladder is the right object for studying tail concentration
**at a fixed origin**; it is the wrong object for studying
locations across two distributions that differ by an additive
offset.

## Where the ladder goes next

The changelog block for v0.6.197 ends with the chain of
inequalities reproduced above and stops there. The natural next
rung is L_9, defined as `sum(x^9) / sum(x^8)`, which by
monotonicity must satisfy `L_8 <= L_9 <= max(x)`. On `claude-code`
the L_8 value is already `95.1M` against an unknown maximum (the
subcommand does not currently report the per-source max, but the
per-source max can be computed from the underlying queue.jsonl in
constant memory). The ceiling on how much further the ladder can
climb on `claude-code` is therefore exactly `max(claude-code) -
95,086,154`, and that ceiling shrinks monotonically toward zero
as the rung index grows. There is a natural stopping rule for the
ladder: once `L_p` for a given source is within some epsilon of
`max(source)`, additional rungs deliver no new information about
that source.

A note on source naming in the live-smoke output above: one source
in the queue is locally redacted to `vscode-XXX` per the standing
policy on source identifiers in published material. The redaction
is purely cosmetic and does not affect the per-source row counts,
total tokens, or any of the Lehmer-mean computations.

For now, the L_8 rung is the latest one nailed in. Six sources,
1,850 rows, zero dropped, every Lehmer monotonicity check passing,
and a `+5,064,831.62`-token l8L7Gap on `claude-code` that the L_7
lens — shipped just one patch version earlier — could not have
exposed.

## Citations

- `pew-insights` `CHANGELOG.md` entry for **v0.6.197 — 2026-04-29**, including the per-source live-smoke table reproduced above.
- Live smoke run at 2026-04-29 against `~/.config/pew/queue.jsonl` (`as of: 2026-04-28T18:16:07.813Z`, `rows: 1,850`, `dropped: 0`).
- Lehmer monotonicity: standard inequality on Lehmer means, see e.g. Bullen, *Handbook of Means and Their Inequalities*, ch. III.
