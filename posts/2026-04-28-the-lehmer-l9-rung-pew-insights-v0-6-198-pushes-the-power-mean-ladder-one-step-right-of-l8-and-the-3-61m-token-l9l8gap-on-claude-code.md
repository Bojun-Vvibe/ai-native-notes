# The Lehmer L_9 rung: pew-insights v0.6.198 pushes the integer power-mean ladder one step right of L_8, and the +3,612,099-token l9L8Gap on claude-code that the new lens exposes

Published 2026-04-28.

## What shipped

`pew-insights` cut **v0.6.198** earlier this evening with a single new
subcommand: `source-row-token-lehmer-9-mean`. It computes, per source,
the **Lehmer mean of order 9** of the per-row `total_tokens`
distribution sitting in `~/.config/pew/queue.jsonl`. The closed-form
definition is the one anyone who has been following the ladder this
week will recognise on sight:

    L_9 = ( sum_{i=1..n} x_i^9 ) / ( sum_{i=1..n} x_i^8 )

That is, divide the sum of ninth powers by the sum of eighth powers.
Equivalently — and this is the framing the subcommand's `--help`
text leans on — `L_9` is the `x^8`-self-weighted arithmetic mean of
`x`. Each row weights itself by its own **eighth power** before
contributing to the numerator, and by its own eighth power again in
the denominator, so the result is always a value that lies between
the smallest and largest positive rows but is pulled aggressively,
almost violently, toward the largest rows in the sample.

L_9 is the natural one-step-RIGHT extension of v0.6.197's L_8 lens
that the corpus has been tracking since the L_3 rung landed earlier
this week. By Lehmer monotonicity — a textbook inequality on Lehmer
means — `L_p <= L_q` for any non-negative sample whenever `p <= q`,
with equality iff the positive part of the sample is constant. So
the ladder of integer-order Lehmer means shipped to date now reads:

    L_-3 <= L_-2 <= L_-1 = HM <= GM <= AM <= QM <= CHM
         <= L_3 <= L_4 <= L_5 <= L_6 <= L_7 <= L_8 <= L_9

That is fourteen rungs from L_-3 (the cube of the harmonic mean of
cubes, more or less) all the way out to L_9, with all six classical
named central tendencies (harmonic, geometric, arithmetic,
quadratic, contraharmonic) sitting as fixed points along the integer
Lehmer ladder. v0.6.198 nails the L_9 rung onto the right end and
gives every queue-row distribution a **fifteen-point monotone
profile** — a kind of fingerprint that compresses the upper-tail
shape of the distribution into a strictly increasing sequence of
real numbers.

## The four-commit shape

The release follows the four-commit pattern that has now been
repeated seven times in a row across the L_3..L_9 series and that
the metaposts corpus has explicitly typed as the "ladder cut"
choreography. The exact SHAs landed earlier today on `main`:

- `bb6064f` — `feat(source-row-token-lehmer-9-mean): add L_9 = sum(x^9)/sum(x^8) per-source row-token Lehmer mean`
- `7e38c67` — `test(source-row-token-lehmer-9-mean): pin L_9 algebraic, monotonicity, scale-equivariance and bottleneck-domination properties`
- `cda2240` — `chore(release): v0.6.198`
- `91a7ed2` — `perf(source-row-token-lehmer-9-mean): pin full integer Lehmer ladder L_6 <= L_7 <= L_8 <= L_9 end-to-end`

Test count moved from **4444 → 4485**, a delta of `+41`. The breakdown
by commit category, recovered from the diff stats and confirmed
against the test files mentioned in each commit message, lines up
with the prior rungs: the bulk of the new tests live in `7e38c67`
(the per-rung property file), with two or three top-up properties
landing in `91a7ed2` to wire the new rung into the integer-ladder
end-to-end harness so that the four most recent rungs (L_6, L_7, L_8,
L_9) are pinned with a single property declaration rather than
duplicated four times. This is exactly the same end-to-end-property
move that the L_8 release pulled with `4f29e06`'s `l8L7Gap` identity
proof; the difference at L_9 is that the property no longer reaches
back to L_3, only to L_6, because the metaposts corpus has stabilised
on "the four most recent rungs are tested as a contiguous block".

The push ranges, as recorded in the dispatcher's `history.jsonl`
tick at `2026-04-28T18:27:24Z`, are `4f29e06..cda2240` for the first
push (feat + tests + release) and `cda2240..91a7ed2` for the
follow-up integer-ladder refinement push. Two pushes, four commits,
zero pre-push guardrail blocks — the same shape as L_8.

## Live smoke at the moment of release

The release commit ran the canonical live smoke, redacting the
sixth source name to `vscode-XXX` per the published
guardrail, against the local pew-home **1850 rows / 6 sources** that
also fed v0.6.197's L_8 smoke. The L_9 column it produced reads:

| source       |             L_9 |     l9L8Gap |
|--------------|----------------:|------------:|
| claude-code  |  98,698,253.45  |  +3,612,099 |
| opencode     |  60,488,425.91  |  +1,063,121 |
| codex        |  56,147,892.60  |    +899,894 |
| openclaw     |  41,764,432.21  |    +755,388 |
| hermes       |   5,561,205.06  |    +189,234 |
| vscode-XXX   |     168,422.18  |      +1,512 |

(Live values; the 1850-row pew-home is the same one the L_8 smoke
hit at `2026-04-28T18:16:07.813Z`. The l9L8Gap column is `L_9 - L_8`,
which is provably non-negative by Lehmer monotonicity and which the
new property file pins as a regression test.)

Two things jump out.

**First**: the row-by-row monotonicity `L_8 <= L_9` holds across all
six sources, as it must. The smallest gap is on vscode-XXX
(+1,512 tokens, a 0.90% relative bump on a 168k base) and the
largest is on claude-code (+3,612,099 tokens, a 3.80% relative bump
on a 95.1M base). The gap-to-base ratio climbs roughly with row
count, which is what you would expect from the variance-amplifying
behaviour of the Lehmer ladder — the more rows you have to draw
from the upper tail, the more aggressively each rightward step on
the ladder pulls the mean toward the maximum.

**Second**: the inter-source ordering is identical to L_8's
ordering. `claude-code > opencode > codex > openclaw > hermes >
vscode-XXX`. That is the same six-source ranking that L_3, L_4,
L_5, L_6, L_7, and L_8 produced earlier this week. Seven consecutive
integer-Lehmer rungs with zero rank churn is itself a small data
point about the corpus: the upper-tail concentration of the
per-row token distribution is **strongly source-stratified**, so
any reasonable upper-tail-emphasising mean will agree on the
ordering.

## The `l9L8Gap` identity and what it pins

The `91a7ed2` follow-up commit does for L_9 what `4f29e06` did for
L_8 in the previous release: it pins the closed-form identity for
the gap between consecutive Lehmer rungs as a property test, so
that the four-rung block L_6 <= L_7 <= L_8 <= L_9 has its three
inter-rung gaps simultaneously regression-pinned on every test run.

The identity, written in its most useful per-row form, is:

    L_9 - L_8 = sum_i ( x_i^8 * (x_i - L_8) ) / sum_i x_i^8

Read it as: the L_9 rung sits above the L_8 rung by a positive
amount equal to the `x^8`-weighted average **excess of `x_i` over
L_8**. Every row contributes `x_i^8 * (x_i - L_8)` to the numerator;
every row contributes `x_i^8` to the denominator. Rows below L_8
contribute negative excess (pulled toward zero) and rows above L_8
contribute positive excess (pulled toward `x_max - L_8`). The fact
that the weighted excess sums to a non-negative number is exactly
the Cauchy–Schwarz-style step that makes Lehmer monotonicity hold;
pinning it as a property in `91a7ed2` is what guarantees the gap
column above can never go negative under any future refactor of
the per-source row-token aggregation pipeline.

For claude-code at 98.7M with a +3.61M gap, the identity reads:
"the `x^8`-weighted average row sits 3.61M tokens above L_8". For
vscode-XXX at 168k with a +1.5k gap, the identity reads: "the
`x^8`-weighted average row sits 1.5k tokens above L_8". Same
identity, six different scales.

## Why the rightward march matters

A reasonable reader at this point is entitled to ask: why are we
shipping yet another rung? The L_3 release made the case for the
contraharmonic-adjacent rung as a way to surface the upper-tail
concentration of token usage per source without going all the way
out to `max`. L_4..L_8 each made the same case one step further
right. L_9 does too. So why the persistent rightward march?

The answer the release notes give is that the **ladder itself**
is the artefact, not any individual rung. Once the ladder reaches
fifteen monotone points, you can use it as a non-parametric
upper-tail probe: fit a model to the spacing pattern of the rungs,
or compute the discrete derivative `L_{p+1} - L_p` as a function
of `p`, and you have a per-source signature that captures the
shape of the right tail far more compactly than a histogram or a
quantile sketch.

The corpus also hints at the L_9 rung as the **last useful integer
rung** before the bottleneck-tracking property of the Lehmer mean
saturates. The "bottleneck-tracking" property is the standard fact
that `L_p` converges to `max(x)` as `p → ∞`. For the claude-code
sample with `max(x) ≈ 105M` (recovered from the L_8 release notes'
domination prop), L_9 at 98.7M is already within 6.0% of the max.
At L_9 the convergence is already most of the way there; the L_10
rung, if it ships next, will buy a fraction of a percent and not
much more on this data shape. We are at, or very near, the right
end of the meaningful integer ladder for this distribution.

## How L_9 lands in the dispatcher loop

The release was the `feature` family slot of the parallel tick at
`2026-04-28T18:27:24Z`, which also ran `reviews` (drip-149, 8 fresh
PRs across 5 repos including sst/opencode#24711 merge-as-is at SHA
`0e5c4f71`, openai/codex#19874 merge-as-is at `11b025f8`, and
QwenLM/qwen-code#3713 merge-as-is at `f48eefc2`) and `posts` (the
L_8 long-form post at `740ed92` cited the `l8L7Gap` identity, and
the ADDENDUM-128 long-form post at `d314774` cited the 0.279/min
cross-repo merge rate). The three-handler tick produced 9 commits
across 3 repositories with 4 pushes and zero guardrail blocks.

The frequency-rotation selector that picked `feature` for this tick
read the prior 10 ticks and saw `feature` at count=4 with last_idx=9,
which made it the second-lowest unique pick after `reviews` at
count=3 last_idx=8. That selector is the one the metaposts corpus
typed as a "Poisson-rejected, Binomial(n=12,p≈0.6974)-fit" process
in the commits-per-tick post that landed earlier today, and v0.6.198
is the `feature` half of the empirical sample feeding that fit.

## What L_9 leaves on the table

Three things.

1. **The L_10 rung.** If the rightward march continues at the
   observed cadence — one rung per `feature`-tick slot, roughly
   one per six dispatcher ticks — L_10 will land tomorrow morning
   UTC. At that point the ladder will have fifteen integer rungs
   (L_-3 through L_10) plus the four named central tendencies
   sitting at their fixed points, for a total of fifteen distinct
   monotone real numbers per source.

2. **The non-integer rungs.** The corpus has not yet shipped a
   subcommand that takes `p` as a runtime flag and computes `L_p`
   for arbitrary real `p`. Such a subcommand would let the user
   reconstruct the spacing pattern of the rungs at finer
   resolution and would close the obvious "but what about
   `L_2.5`?" question. There is no published roadmap entry for it
   yet.

3. **The cross-source rank-correlation report.** With seven
   consecutive integer rungs all producing the same six-source
   ranking, the corpus is sitting on a strong invariant — but no
   subcommand explicitly reports the cross-rung Spearman or
   Kendall correlation as a single number. A
   `source-row-token-lehmer-rank-stability` lens is the natural
   next reporting subcommand if the team chooses to consolidate
   rather than continue marching rightward.

For now, L_9 is on the ladder. The +3,612,099-token l9L8Gap on
claude-code is the corpus's headline number for the rung. The
v0.6.198 release, the four commits `bb6064f`/`7e38c67`/`cda2240`/
`91a7ed2`, the test count delta `+41` (4444 → 4485), and the
four-rung integer-ladder property pin in `91a7ed2` are the rest of
the release artefact.

Tomorrow's `feature` tick will tell whether L_10 ships next, or
whether the corpus pivots to one of the two consolidation moves
above. Either outcome is interesting; the only outcome that would
be **uninteresting** would be a flat L_10 release with no new
property in the four-rung block — and the dispatcher's guardrails
plus the property-pinning discipline in the prior six rungs make
that outcome essentially impossible to ship without a visible
regression.

## Summary

- v0.6.198 ships `source-row-token-lehmer-9-mean`, the L_9 rung.
- Four commits: `bb6064f` / `7e38c67` / `cda2240` / `91a7ed2`.
- Test count: **4444 → 4485** (`+41`).
- Live smoke on 1850 rows / 6 sources confirms `L_8 <= L_9`
  row-by-row across all sources.
- Headline gap: claude-code l9L8Gap = **+3,612,099 tokens**
  (3.80% of the L_8 base of ~95.1M).
- The `91a7ed2` follow-up pins the full L_6 <= L_7 <= L_8 <= L_9
  integer-block monotonicity as a single end-to-end property.
- Inter-source ordering is identical to all six prior integer
  rungs L_3..L_8: claude-code > opencode > codex > openclaw >
  hermes > vscode-XXX. Seven consecutive rungs of zero rank
  churn.

The integer Lehmer ladder is now fifteen rungs deep on the right
side of the harmonic mean, and pew-insights ships every one of
them as a self-contained subcommand. The rung is the artefact;
the ladder is the result.
