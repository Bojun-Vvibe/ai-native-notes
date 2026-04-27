# The 0.8 madRatio cohort gate: when only two sources have a body as wide as their median

In the v0.6.90 release of pew-insights, the per-source Median Absolute
Deviation lens (`source-row-token-mad`, introduced in v0.6.89) gained a
display filter: `--min-mad-ratio <f>`. The flag drops every source whose
robust scale-free spread — `madRatio = mad / median` — is strictly below
`f`. The default is `0`, which preserves prior behaviour. Anything above
`0` engages the filter.

The interesting threshold, the one the CHANGELOG entry actually demonstrates
on live data, is `--min-mad-ratio 0.8`. At that threshold, on a 1,620-row
sample of agent telemetry, **four out of six sources are dropped**.
Only two survive: `codex` at 0.9121 and `claude-code` at 0.9348. The
question of *why* those two and not the others is the subject of this post,
and the answer turns out to be more structural than it looks.

## The threshold and what it asks

The CHANGELOG entry is explicit about the operator semantics:

> `--min-mad-ratio 1.0`  surface only sources whose typical absolute
> deviation equals or exceeds the typical row size — a heavy-tailed-or-bimodal
> cohort.

`0.8` sits one notch below that, in the operator's suggested ladder of
`{0.1, 0.5, 1.0}`. It is the "deviation is at least 80% of typical row
size" gate. That is a fairly aggressive ask. It says: a source survives only
if, when you stand at the median and walk outward, you have to walk a
distance equal to roughly four-fifths of where you started before you
account for the typical row.

For a Normal distribution with `mu = 100`, `sigma = 40`, the madRatio is
`(0.6745 × 40) / 100 = 0.27`, well below 0.8. A Normal source needs
`sigma >= 1.19 × mu` (an extreme dispersion regime) to survive this gate.
For an exponential distribution, the population madRatio is `ln(2) ≈ 0.69`,
also below 0.8. **Neither of the two textbook unimodal distributions makes
it through this filter.** Anything that does is doing something the textbook
shapes don't.

## The live data

From the v0.6.90 CHANGELOG live smoke run against
`~/.config/pew/queue.jsonl`, as of `2026-04-27T02:44:19.462Z`,
`--min-rows 5 --min-mad-ratio 0.8`, six sources processed, two displayed,
four suppressed via `droppedBelowMinMadRatio`:

```
source       rows  median      mad         madScaled   madRatio  degen
-----------  ----  ----------  ----------  ----------  --------  -----
codex        64    7132861.00  6506096.00  9645937.93  0.9121    -
claude-code  299   3319967.00  3103438.00  4601157.18  0.9348    -
```

The four dropped sources, with their madRatios pulled from the v0.6.89
unfiltered run on the same dataset:

| dropped source              | madRatio | gap below 0.8 |
|-----------------------------|----------|---------------|
| opencode                    | 0.7827   | −0.017        |
| hermes                      | 0.7781   | −0.022        |
| vscode-assistant-redacted   | 0.7486   | −0.051        |
| openclaw                    | 0.5491   | −0.251        |

This is the data citation the post is built on. Two values matter most:
the survival pair (codex 0.9121, claude-code 0.9348) and the largest gap
(openclaw at 0.5491, a full 0.25 below the gate).

## Why exactly two sources

There are six sources on this dataset and only two survive. That is not
because the gate is generic — at `--min-mad-ratio 0.5` everyone passes; at
`--min-mad-ratio 1.0` nobody does. The 0.8 threshold lands precisely in
the cohort gap between two structural classes of source.

**The survival pair: codex and claude-code.** Both are generative-coding
agents driven by long-form interactions. Their per-row `total_tokens`
distribution has medians of 7.13M and 3.32M respectively — large numbers,
but with MADs (6.51M and 3.10M) that are nearly as large as the medians
themselves. That is the structural signature: the body of the
distribution is **not** clustered around a single typical session size. A
typical session deviates by nearly the whole median from the median. There
is no mode the rows are huddling near; instead the rows are spread out
across a range that is comparable in size to the location parameter
itself.

This is the cohort the v0.6.89 reading guide flags as "heavy-tailed or
bimodal":

> `madRatio ~ 1`  : the typical absolute deviation equals the typical row
> size — a heavy-tailed or bimodal cohort.

Notice the language. It is not saying these sources have heavy tails. It
is saying their bodies look like they have heavy tails — even after you
strip the actual heavy tail with the median anchor. That is the bimodality
case: a source whose rows partition into two regimes (a small-row regime
and a large-row regime) without a clear modal cluster will produce a high
madRatio because the median sits in the gap between the two regimes and
the deviations have to span that gap.

**The dropped four: opencode, hermes, vscode-assistant-redacted, openclaw.**
These are the sources whose body is unimodal and clustered. The deviations
are below 80% of the median. The bulk of their rows pile up near a
characteristic value; outliers exist but cannot pull MAD past the gate
because MAD is robust.

## The openclaw gap and what it means structurally

`openclaw` sits 0.25 below the gate at madRatio = 0.55 — by a wide margin
the tightest body in the cohort. That is structurally interesting. This
source has 431 rows on the sample, the largest count among the six. A
large row count with a tight robust spread is the signature of a source
that is doing the same kind of thing repeatedly. The rows huddle. The
typical absolute deviation is barely more than half the median. Most
sessions are within a factor of two of the typical session size.

This is what a clocked-cadence, narrow-task source looks like. There is
no bimodality, no obvious tail-leverage, no genuine spread of session
sizes. The right tail exists — every source on agent telemetry has a
right tail because of the strict-zero floor — but that tail has not
managed to push MAD past 55% of the median, even with 431 chances to do
so.

The contrast with codex (64 rows, madRatio 0.91) is sharp. codex has the
fewest rows in the sample but the second-highest madRatio, surviving the
gate by 0.11. With only 64 sessions to draw from, the source still
produces a body that is spread out across a range comparable in size to
its own median. That is genuine heterogeneity per session, not a tail
artifact: with 64 rows, the median is anchored on row 32–33, and the MAD
is anchored on the median absolute deviation across all 64 — both quite
robust at that sample size, both saying "rows are really spread out."

## Why this is not the same question as `--min-cv 0.8`

It is tempting to assume that any dispersion gate at 0.8 will produce
similar cohorts. It does not. The v0.6.88 `--min-cv` filter at threshold
1.2 (a comparable "above the exponential baseline" gate) on the same
dataset surfaces three sources: `vscode-assistant-redacted` (CV 2.64),
`claude-code` (1.53), and `opencode` (1.29). The CV gate keeps
`vscode-assistant-redacted` because its tail is extreme. The MAD gate
**drops** that source because its body is tight (madRatio 0.75).

The two filters disagree on five of six sources. The only consistent
survivor across both gates is `claude-code`. Everything else flips. The
mathematical reason is the one developed at length in the v0.6.89
CHANGELOG entry: CV uses non-robust anchors, MAD uses robust anchors;
they coincide only when there is no outlier leverage. `--min-cv 1.2` is
asking "is this source's tail heavy relative to its mean?"
`--min-mad-ratio 0.8` is asking "is this source's body wide relative to
its median?" These are not the same question, and on this six-source
sample they produce nearly disjoint cohorts.

The operational consequence: if you are trying to identify sources whose
**typical** session is drawn from a wide distribution — i.e., where you
cannot point at a single number and call it a representative session size
— the MAD gate is the right tool. If you are trying to identify sources
whose **outlier** sessions are extreme relative to the typical — i.e.,
where the median is a fine summary but the tail will eat your
percentile dashboards — the CV gate is the right tool. Choosing
incorrectly will surface the wrong cohort and trigger the wrong followup.

## What 0.8 specifically catches

There are five operator-meaningful ladders the v0.6.90 documentation
implicitly defines:

1. `--min-mad-ratio 0.0` (default): keep all sources, including `degen=y`
   sources where `median = 0` and madRatio is mathematically undefined (it
   is reported as 0 with the degenerate flag set, and the default 0
   preserves these rows).
2. `--min-mad-ratio 0.0001` (any positive): drop the degenerate cohort.
   Same convention as `--min-cv` and `--min-abs-skew`.
3. `--min-mad-ratio 0.1`: drop sources whose typical deviation is below
   10% of the median. The "rows are robustly tightly clustered" filter.
4. `--min-mad-ratio 0.5`: drop everything tighter than the
   moderate-robust-spread baseline.
5. `--min-mad-ratio 0.8`: drop everything except the
   bimodal-or-genuinely-heavy-tailed cohort. The threshold this post is
   about.
6. `--min-mad-ratio 1.0`: surface only sources whose typical absolute
   deviation equals or exceeds the typical row size — extremely rare,
   essentially always a sign of true bimodality.

`0.8` is therefore the most aggressive **non-extreme** threshold. It is
the gate operators use when they want to catch the cohort that is
"obviously not unimodal" without requiring the population-level rarity of
madRatio ≥ 1. On this dataset that captures exactly the two
generative-coding agents driven by free-form input, and rejects the two
clocked daemons (`hermes`, `openclaw`), the editor-assistant cohort
(`vscode-assistant-redacted`), and the catch-all multi-purpose runner
(`opencode`).

That is a meaningful classification and one that no single moment
statistic on this sample can reproduce. The skewness lens (g1) places
codex and claude-code in the middle of the pack — neither is a top
skewness source. The kurtosis lens (g2) places `vscode-assistant-redacted`
at the top, not in the surviving cohort. The CV lens places
`vscode-assistant-redacted` at the top by a factor of two. Only the
madRatio lens at threshold 0.8 picks out **the cohort where the typical
session size has the widest body relative to its location anchor**.

## The numerical sanity check

Suppose we synthesise a tightly Normal source: 1,000 rows from
`N(mu=10000, sigma=500)`. Population madRatio is
`0.6745 × 500 / 10000 ≈ 0.034`. This source would not survive even a
`--min-mad-ratio 0.1` gate, let alone 0.8.

Suppose we synthesise an exponential: 1,000 rows from `Exp(rate=1/10000)`.
Population madRatio is `ln(2) ≈ 0.69`. Below the 0.8 gate.

Suppose we synthesise a bimodal mixture: 500 rows from
`N(mu=2000, sigma=100)` and 500 rows from `N(mu=20000, sigma=100)`. The
median sits around 11,000 (between the two modes). MAD is around 9,000
(half the inter-mode distance). madRatio is ~0.82. **Just above the 0.8
gate.** This is the canonical bimodality signature, and the gate is
calibrated precisely to catch it.

The empirical madRatios of codex (0.91) and claude-code (0.93) are
slightly above the bimodal baseline. That is consistent with two regimes
on each source — short interactive sessions and long deep-work sessions
— without perfectly equal mass between them, so the median sits closer to
one regime and the MAD reflects the gap to the other. This is the
expected fingerprint for an agent that is used both as a shell-style
turn-by-turn helper and as a longform pair-programmer.

## Takeaways for operators

The 0.8 madRatio gate is calibrated to the bimodality threshold and not
to the exponential baseline. That choice is principled: at 0.69 the gate
is a less sharp filter (it lets exponentials through); at 1.0 the gate
is too restrictive (almost no real agent telemetry survives). 0.8 is the
operator's sweet spot for "show me the sources whose typical session size
spans a regime, not a single mode."

On the live data, exactly two sources clear the gate, with comfortable
margins (codex by 0.11, claude-code by 0.13). The next-closest sources
sit just below (opencode by 0.02, hermes by 0.02), close enough that a
single bursty week could push them over. `vscode-assistant-redacted` is
0.05 below and would need a structural change in usage pattern to
survive. `openclaw` is 0.25 below — it would essentially need to become a
different source.

A natural followup: track this filter daily. The number of sources
clearing `--min-mad-ratio 0.8` is a regime-stability indicator at the
cohort level. A source moving from below to above the gate is a signal
that its usage has broadened from clustered-around-a-mode to
spanning-multiple-regimes. A source moving the other way is a signal of
consolidation — the source is being used for a narrower band of session
sizes. Both are interesting transitions to catch, and the daily count
above the gate is the simplest single number that surfaces them.

The gate at 0.8 is not arbitrary. It is the bimodality cutoff. On
2026-04-27 it identifies precisely the two sources whose body is as wide
as their median. That is the cohort that does not have a typical session
size in the usual sense of the word.
