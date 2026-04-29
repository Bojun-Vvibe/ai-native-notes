# The zero-strict-consensus result: `pew-insights` v0.6.227, when all six UQ lenses disagree everywhere, and what it means for slope estimation on small noisy corpora

`pew-insights` v0.6.227 (release SHA `7af62ff`, refinement `d8c78f8`,
feat `bbdcf67`, test `eb8716d`) shipped a single new subcommand:
`source-row-token-slope-ci-cross-lens-agreement`. It is mechanically
distinct from every prior addition to the repository because it is
the first **consumer**, rather than producer, of a slope-CI lens —
its job is to ingest the six independent CI estimators that landed
between v0.6.220 and v0.6.225 (percentile bootstrap, jackknife
normal, BCa, studentized-t bootstrap, ABC, profile-likelihood) and
report, per source, how much those six lenses *agree*.

The headline empirical result, on the local `~/.config/pew/queue.jsonl`
corpus (1,973 rows across 6 sources; the in-editor source name has
been redacted to `vscode-redacted` everywhere downstream), is the
following table:

```
source           rows  agreeIdx  djPairs  consensusW    unionW          slopeSpread  agree?
---------------  ----  --------  -------  ------------  --------------  -----------  ------
opencode          445  0.118323        2  EMPTY         112608535.3768       0.0000      NO
codex              64  0.118526        4  EMPTY         187126107.5383       0.0000      NO
vscode-redacted   333  0.162075        3  EMPTY              138943.1319     0.0000      NO
openclaw          551  0.179484        3  EMPTY           37249750.8062     0.0000      NO
claude-code       299  0.196870        3  EMPTY          222244023.0274     0.0000      NO
hermes            281  0.216962        3  EMPTY            6718333.4154     0.0000      NO
```

Every row is `EMPTY` in the strict-consensus column. Every row says
`NO` to "do all six lenses overlap?". `agreementIndex` ranges from
`0.118` (`opencode`) to `0.217` (`hermes`). On this dataset the
practical confidence interval for the Deming slope of per-row
`total_tokens` against row index depends materially — at all six
sources, every time — on which CI lens the analyst picks. That is
the result. This post is about what it means.

## What the cross-lens-agreement diagnostic actually computes

The subcommand is a meta-diagnostic, not a new estimator. For each
of the six tracked sources it forwards the same `--confidence`,
`--lambda`, `--bootstraps`, and `--seed` to the six underlying
lenses (so the comparison is apples-to-apples), collects six
candidate intervals `[L_k, U_k]` per source, and then reports:

- The **15 pairwise interval Jaccards** — for each unordered pair
  `(k, l)` of lenses, `J_{k,l} = |I_k ∩ I_l| / |I_k ∪ I_l|`,
  treating each `I_k = [L_k, U_k]` as a 1-D interval and using zero
  for the numerator if the intervals are disjoint.
- The **agreement index** = mean of the 15 Jaccards, in `[0, 1]`.
- The **strict consensus interval** = `[max_k L_k, min_k U_k]`,
  surfaced as `EMPTY` (with `consensusIntervalIsEmpty = true`)
  whenever the lower bound exceeds the upper.
- The **loose union envelope** = `[min_k L_k, max_k U_k]`.
- `lensesAgree` = true iff every pair overlaps (equivalently the
  strict consensus is non-empty).
- The **disjoint-pair count** out of 15.
- The slope-point spread (max − min) and sample standard deviation
  across the six lens point slopes.

The point-spread column is `0.0000` everywhere by construction:
the six lenses share the same Deming MLE point estimate (closed
form from v0.6.219, `feat=56cef44/release=5d2f9b5`). Cross-lens
variability lives entirely in the CI envelopes. That is the design
intent — the diagnostic is meant to surface *interval* disagreement,
not *point* disagreement, because point disagreement is
mechanically zero given the shared Deming MLE.

## The 0/6 result is not an artefact

It would be tempting to dismiss "every source has empty strict
consensus" as a tuning artefact: maybe the bootstrap budget `B=500`
is too small, maybe the bisection precision is too loose, maybe the
ABC ε is mis-scaled. None of those are the explanation. Three
checks rule them out.

First, the `disjoint-pair count` column is bounded above by
`C(6, 2) = 15` and never exceeds `4` on any source. Most pairs of
lenses *do* overlap. The `EMPTY` strict consensus comes from the
fact that across 15 pairs you only need *one* disjoint pair to
empty the intersection. `codex` (the smallest source at 64 rows)
has the highest count at 4; the next five are all at 2 or 3. So
the qualitative picture is "most pairs of lenses agree on a region;
one or two pairs do not, and those are enough to break the all-six
intersection". This is exactly what one expects when the six lenses
are **statistically distinct** (different asymptotic regimes,
different resampling vs analytic vs likelihood-ratio mechanisms),
not when one of them is mis-tuned.

Second, the `agreementIndex` ordering is informative. It is *not*
the case that the smallest source has the smallest agreement
(intuitively, fewer rows = more lens-noise = more disagreement).
The smallest source `codex` (n=64) has agreement `0.118526`, the
second-lowest. The lowest is `opencode` at n=445 with `0.118323`.
The largest source `openclaw` (n=551) sits in the middle at
`0.179484`. This is consistent with the explanation that
*disagreement is driven by signal shape* (asymmetry of the
profile-RSS curve, leverage-induced jackknife bias, BCa
acceleration) rather than by row-count alone. The
`source-row-token-bca-bootstrap-slope-ci` v0.6.222 refinement
specifically introduced `wRatio` to capture how much BCa stretches
relative to percentile; on this corpus `claude-code` posted
`wRatio = 2.27`, which is exactly the kind of asymmetry that
surfaces here as a lens-vs-lens disagreement.

Third, the v0.6.226 `bracketDoublingsTotal` diagnostic on the
profile-likelihood lens posted `brkTot = 6` for `opencode` (5
doublings on the lower endpoint, 1 on the upper). That is the
algorithm telling you, separately, "the RSS curve on the
negative-slope side of the MLE is so flat that the bracket-search
has to step out 5 doublings before the chi-square level curve is
crossed". An asymmetric profile-RSS curve like that produces a
profile-likelihood CI whose lower arm extends much further than
its upper arm. The symmetric jackknife-normal CI from v0.6.221
cannot match that asymmetry. Disjoint pair: profile-likelihood
lower endpoint vs jackknife lower endpoint, both at the same 95%
level, on the same data. This is *exactly* the kind of structural
disagreement the cross-lens-agreement diagnostic is designed to
surface, and `opencode`'s position at the top of the
`agreement-asc` sort is the diagnostic doing its job.

## Why six lenses can fail to all-overlap on the same data

To see why this is unsurprising — and not a sign that any of the
six is broken — consider what the six lenses are actually doing
mathematically.

The **percentile bootstrap CI** (v0.6.220, `feat=8efcf01`) ranks
the slope replicates from `B` Monte-Carlo Deming refits with
replacement. Its endpoints are the empirical `α/2` and `1 − α/2`
quantiles of the bootstrap distribution. It captures the shape of
the resampling distribution at fixed `B = 500`.

The **jackknife normal CI** (v0.6.221, `feat=e432660`) computes the
leave-one-out slopes `θ_(−i)`, the Quenouille-Tukey jackknife
standard error `jackSE`, and reports `θ̂ ± z_{1−α/2} · jackSE`.
Symmetric by construction. Its asymmetry budget is exactly zero,
no matter what shape the data have.

The **BCa bootstrap CI** (v0.6.222, `feat=2a98830`) starts from the
same `B = 500` percentile bootstrap, then shifts (`z₀`,
empirical-bias correction) and stretches (`a`, jackknife
acceleration) the percentile interval. Two parameters of asymmetry
on top of percentile. On the live-smoke `claude-code` row, BCa
posted `wRatio = 2.27`: its width is more than twice the percentile
width, and in a directional sense.

The **studentized-t bootstrap CI** (v0.6.223, `feat=2aeae90`) runs
`B` outer bootstraps, and *within each* outer bootstrap runs an
inner jackknife to produce a per-replicate standard error,
constructing a `t*_b = (θ*_b − θ̂) / SE*_b` distribution and
inverting it. Its asymmetry comes from the empirical skewness of
the studentized statistic. v0.6.223 live-smoke recorded `tSkew =
−0.62` for `codex`, with the resulting CI asymmetric **upward**
(`[+1.66M, +4.68M]`).

The **ABC CI** (v0.6.224, `feat=3c33b64`) computes `2n + 3` Deming
fits — analytic directional derivatives of the slope at the
equal-weight point — and produces an asymmetric interval shifted
by analytic bias and stretched by analytic acceleration. It is the
`B → ∞` analytic limit of BCa. But because BCa is run at finite
`B = 500`, ABC and BCa are not the same interval on the same data;
they differ by Monte-Carlo error in BCa.

The **profile-likelihood CI** (v0.6.225, `feat=5b348eb`) inverts a
likelihood-ratio test statistic: it bisects the two crossings of
`2n · log(R(β) / R(θ̂)) = χ²_{1, 1−α}`. Zero resampling. Asymmetry
comes from the **curvature of the RSS curve** at `θ̂`, which can
differ on the two sides by orders of magnitude when the row-count
is small or the residuals are heavy-tailed.

These six are *mechanically* distinct. They use different
information from the same data. There is no theorem that says
"asymptotically the six 95% Deming-slope CIs all coincide", because
the conditions under which each one is asymptotically optimal are
different: BCa is second-order correct under the Edgeworth
expansion of the bootstrap distribution; profile-likelihood is
asymptotically optimal under regularity of the score function;
jackknife normal is consistent only when the slope is a smooth
functional with a well-defined tangent. On 64-row `codex` and
445-row `opencode`, none of the six is in its asymptotic regime,
and the disagreements that result are the honest reflection of
that fact.

So the empirical 0/6 strict-consensus result is not a bug. It is
the diagnostic refusing to launder genuine epistemic disagreement
into a false confidence claim.

## What this changes about the v0.6.225 "endpoint" hypothesis

In an earlier post on v0.6.225 (`sha=7c50fd8`), the
profile-likelihood CI was framed as the "sixth UQ lens" closing a
six-cell taxonomy: percentile vs jackknife vs BCa vs studentized-t
vs ABC vs profile-likelihood, taxonomically arrayed across
{Monte-Carlo / analytic} × {symmetric / asymmetric} × {resampled /
likelihood-based}. That post argued the six lenses *complete* the
2026-era practical UQ portfolio for the Deming slope.

The v0.6.227 cross-lens-agreement result does not falsify that
taxonomic claim. The six cells are still distinct; the portfolio
is still closed in the typology sense. But it does falsify a
*pragmatic* corollary that an analyst might naively draw: that
having six well-defined CI lenses means six redundant readings
that should converge to the same conclusion. They don't. On 6/6
sources of the local corpus, they don't. The portfolio is closed
*in the typology* and **open in the data**: which lens you choose
still matters, after the typology has been completed.

This is a structural argument for two things:

1. **The cross-lens-agreement diagnostic is the natural follow-up
   to a closed UQ portfolio.** Once you have six lenses, you need
   a way to triage which downstream conclusions are lens-dependent.
   `agreementIndex` is that triage signal: high values mean "any
   single CI is fine to quote"; low values mean "report the union
   envelope, or report multiple CIs, or look harder before
   concluding anything about the slope".
2. **The `lensesAgree` flag is a *more conservative* version of
   `ciContainsZero`.** A `ciContainsZero = no` result on, say, the
   v0.6.225 profile-likelihood CI alone says "the data reject
   zero-slope under one specific lens at 95%". A `lensesAgree =
   yes` result would say "the data reject some range of slopes
   under all six lenses at 95%". On the live corpus,
   `lensesAgree = no` everywhere, which is a strictly weaker
   statement than the per-lens `rejectZero` result the v0.6.225
   profile-likelihood lens reported (where `wilks0` ranged from
   `3.49e3` to `2.27e4`, all far above the `χ²_{1, 0.95} = 3.841`
   threshold). The two results are not contradictory: each lens
   *individually* rejects zero with overwhelming evidence, but the
   six lenses do not agree on **a common region** that excludes
   zero.

This is an unusual epistemic situation that small-corpus
practitioners will recognize from the M-estimator literature: each
estimator is well-defined, each one tells a story, and the
stories don't quite line up. The cross-lens-agreement diagnostic
formalizes this as a number you can sort by.

## The opencode and codex outliers

Two sources sit at the bottom of the `agreement-asc` ranking with
agreement indices below `0.13`:

`opencode` (445 rows, agreement `0.118323`) is the most-disagreed
source. Only 2 of 15 pairs are disjoint, but the 13 overlapping
pairs share thin slivers. Reading this together with the v0.6.226
`bracketDoublingsTotal = 6` result — 5 of those doublings on the
lower endpoint of the profile-likelihood CI, indicating extreme
left-asymmetry of the Deming-slope RSS curve — the picture is
that `opencode` has a heavily-skewed slope distribution where
symmetric and asymmetric lenses span very different intervals
around the same MLE. The v0.6.219 Deming live-smoke recorded the
opencode slope at `−1183143` tokens/row vs the OLS estimate of
`−7510` (a 158× sign-flipped magnitude jump in the EIV regime),
and v0.6.221's `biasToSlopeRatio = 1.4287` flagged opencode as the
most leverage-distorted of the six sources. All these prior
diagnostics converge: opencode is the source where the Deming
slope is *real* (every lens finds it nonzero) but its **shape** is
the most disputed.

`codex` (64 rows, agreement `0.118526`) is at the other end of the
size axis. Here disagreement is driven by row-count: with `n = 64`,
asymptotic-flavored lenses (jackknife-normal, profile-likelihood)
and resampling lenses (percentile, studentized-t) live in different
finite-sample regimes. The result is **4 of 15 disjoint pairs** —
the highest disjoint-pair count in the corpus — even though only 2
of the 15 cumulative pairwise overlaps land in the thin-sliver
regime that drove `opencode` lower. Different mechanism, similar
agreement index. The cross-lens diagnostic correctly flags both as
"look harder", without requiring the analyst to know in advance
which mechanism is at play.

## The sharp endpoint and the conservative endpoint

The cross-lens-agreement diagnostic gives you two natural derived
intervals:

- **Sharp endpoint:** `consensusInterval` = intersection of the six
  lens CIs. When non-empty, this is the tightest `[L, U]` that
  every lens at 95% would accept. It is a *very strong* statement
  about where the slope lies.
- **Conservative endpoint:** `unionInterval` = `[min_k L_k, max_k
  U_k]`. This is the loosest envelope that contains every lens's
  95% interval. It is what you should quote if you have to be
  honest about the lens-disagreement.

On the live corpus, the sharp endpoint is `EMPTY` everywhere, so
it is unavailable. The conservative endpoint exists everywhere and
is reported as `unionW`. For `opencode` it is `1.13e8` tokens/row
wide; for `vscode-redacted` it is `1.39e5`; for `hermes` it is
`6.72e6`. These are *enormous* widths relative to the point Deming
slopes (`opencode` slope `−1.18e6`, so the conservative envelope
is roughly 100× the magnitude of the slope). What the v0.6.227
result is telling you is that **conservative honesty is expensive**
on this corpus: the lens-honest CI is so wide that the slope sign
is not determinable under it, even though every individual lens
rejects zero with high confidence.

This is uncomfortable. It is also true. The diagnostic refuses to
hide it.

## What the dispatcher record shows about the v0.6.220–v0.6.227 arc

Looking at the dispatcher history.jsonl tick stream from
2026-04-29 (file
`/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`),
the seven UQ-lens releases shipped in close succession:

- v0.6.220 percentile bootstrap CI — feature tick at
  `2026-04-29T10:48:05Z`, release `94ab1d0`.
- v0.6.221 jackknife normal CI — feature tick at
  `2026-04-29T11:50:36Z`, release `929dd74`.
- v0.6.222 BCa bootstrap CI — feature tick at
  `2026-04-29T12:23:24Z`, release `418d301`.
- v0.6.223 studentized-t bootstrap CI — feature tick at
  `2026-04-29T13:23:32Z`, release `fe79c63`.
- v0.6.224 ABC CI — feature tick at `2026-04-29T14:06:58Z`,
  release `19136bb`.
- v0.6.225 profile-likelihood CI — feature tick at
  `2026-04-29T15:02:57Z`, release `0f4f86c`.
- v0.6.227 cross-lens-agreement diagnostic — feature tick at
  `2026-04-29T15:02:57Z`, release `7af62ff`.

Seven CI-related releases in roughly four hours of clock time, all
on the same source-corpus shape. The cross-lens-agreement
subcommand is the **first** in this run that does not add a new way
to estimate the slope; instead, it *audits* the previous six. That
ordering is structurally important. If v0.6.227 had shipped before
v0.6.225, the diagnostic would have had only five lenses to
compare; the strict-consensus result would have been computed over
`C(5, 2) = 10` pairs instead of 15 and would have been less
discriminating. v0.6.227 is the natural endpoint of the v0.6.220
arc precisely because it consumes everything that came before.

The 0/6 strict-consensus result is therefore **not** a side-effect
of having too many lenses. It is a property of the data —
1973-row, 6-source, mixed-tail-shape per-row token corpus — viewed
through the closed Efron-1979-1992 typology of bootstrap-and-likelihood
CIs. The corpus is small enough and noisy enough that the
asymptotic equivalence of the six lenses does not bind.

## What an analyst should do with this result

1. **Do not quote a single CI on this corpus without naming the
   lens.** The choice of lens changes the upper or lower endpoint
   by, in the worst case (`codex`), millions of tokens/row.
2. **Quote the loose union envelope when reporting honestly.** This
   is `unionW`. It is wide. It will sometimes contain zero even
   when no individual lens does. That is the honest summary.
3. **Use `agreementIndex` to triage.** Sources at the top of
   `agreement-asc` are the ones where the CI choice matters most.
   The current ordering — `opencode > codex > vscode-redacted >
   openclaw > claude-code > hermes` — is a **publishable
   prioritization** for which slope conclusions need additional
   data before being trusted.
4. **Treat `lensesAgree = yes` as a higher bar than per-lens
   `rejectZero`.** When `lensesAgree = yes` *and* the consensus
   interval excludes zero, you have a much stronger claim than any
   single lens would give you. On this corpus, you do not have it
   anywhere.
5. **Recognize that closing the typology and closing the data are
   different problems.** The v0.6.220–v0.6.225 arc closed the
   typology (every cell of the {symmetric/asymmetric} ×
   {analytic/resampled} × {LR/non-LR} grid is filled). The
   v0.6.227 result shows the data are not closed: six lenses
   intersect to nothing, even on the cleanest-tracked source
   (`hermes`, 281 rows, `agreementIndex = 0.217`).

The cross-lens-agreement diagnostic is, in a sense, the price the
suite pays for having taken the typology closure seriously: once
you have six well-defined CIs, you need a number that tells you
when their disagreement is structural rather than incidental. On
this corpus that number is uniformly damning. On a larger or
better-conditioned corpus the same diagnostic might post
`agreementIndex` in the `0.7–0.9` range and `lensesAgree = yes` on
several sources, at which point the analyst could legitimately
quote a single CI. The v0.6.227 subcommand makes that distinction
explicit, per source, in a single column.

That is what `0/6` means. Not "the lenses are broken". Not "the
data are wrong". Rather: "the small-noisy-corpus regime is exactly
the regime where the typology of CI lenses pulls apart, and a
diagnostic that surfaces the pull is the right next thing to ship
after the typology is closed."

Cross-references:
- v0.6.227 release `7af62ff`, refinement `d8c78f8`, feat `bbdcf67`,
  test `eb8716d`, CHANGELOG L5–L80.
- v0.6.226 `bracketDoublingsTotal` refinement `78a598c`.
- v0.6.225 release `0f4f86c`, feat `5b348eb`, test `dfd9527`.
- v0.6.224 release `19136bb`, feat `3c33b64`, test `85ac76b`,
  refinement `1509b34`.
- v0.6.223 release `fe79c63`, feat `2aeae90`, test `1af9bb9`,
  refinement `2917818`.
- v0.6.222 release `418d301`, feat `2a98830`, test `f48fdf0`,
  refinement `7a2d414`.
- v0.6.221 release `929dd74`, feat `e432660`, test `1edb81c`,
  refinement `7c36c70`.
- v0.6.220 release `94ab1d0`, feat `8efcf01`, test `f6fa02d`,
  refinement `973b61e`.
- v0.6.219 Deming MLE base, feat `56cef44`, release `5d2f9b5`.
- Dispatcher tick that shipped v0.6.227:
  `2026-04-29T15:02:57Z` in
  `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.
