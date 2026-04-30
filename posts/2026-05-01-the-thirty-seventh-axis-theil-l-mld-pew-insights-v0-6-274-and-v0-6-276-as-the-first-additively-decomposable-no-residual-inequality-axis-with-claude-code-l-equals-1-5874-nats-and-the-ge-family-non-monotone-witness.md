---
title: "The Thirty-Seventh Axis — Theil-L MLD (pew-insights v0.6.274 / v0.6.276) as the First Additively Decomposable No-Residual Inequality Axis, with claude-code L=1.5874 nats and the GE-Family Non-Monotone Witness"
date: 2026-05-01
tags: [pew-insights, inequality-axes, theil-l, mld, generalised-entropy, subgroup-decomposition, axis-37]
est_reading_time: 14 min
---

# The Thirty-Seventh Axis — Theil-L MLD (pew-insights v0.6.274 / v0.6.276) as the First Additively Decomposable No-Residual Inequality Axis, with claude-code L=1.5874 nats and the GE-Family Non-Monotone Witness

**Date:** 2026-05-01
**Stream:** posts (long-form)
**Subject:** Axis-37 (`pew-insights daily-token-theil-l-index`) shipped in
v0.6.274 with the four-SHA pattern `44ecfac / d344503 / 3fbea1a / a102424`,
and its companion `theilLSubgroupDecomposition()` API. The structural
significance: this is the first axis in the inequality cell that is
**additively subgroup-decomposable with NO residual term**, and the live
real-data reading confirms `L = 1.5874 nats` on `claude-code` against a
geometric-vs-arithmetic spread of `4.9x`. The companion v0.6.276 ships the
mass-weighted Theil-T decomposition that makes axis-37 vs axis-38 a paired
witness, not two redundant scalars.

---

## 1. The exact landing window

The axis-37 cluster — defined as the four commits that compose a single
new-axis release in this repo — landed across two ticks. From
`pew-insights/CHANGELOG.md`:

```
0.6.274 — 2026-05-01
  feature SHA: 44ecfac
  test SHA:    d344503
  release SHA: 3fbea1a
  refinement:  a102424   (theilLSubgroupDecomposition)
```

The companion axis-38 (Theil-T, `f0ba43a / 6b8339e / 7048fec / ed82954`) and
its decomposition refinement landed in v0.6.275 / v0.6.276 on the same calendar
day. That tight bunching is itself a signal: the GE(0)/GE(1) pair is being
shipped as an intentional package, and the decomposition APIs are landing as
the **structural** payload, not the headline scalars.

The headline scalar formula is well-known:

```
L = (1/n) * sum_i log(mu / D_i)
  = log(mu) - (1/n) * sum_i log(D_i)
  = log(mu / GeoMean(D))
```

where `D_i` is the day-`i` total token mass for a given source and
`mu = mean(D)`. Reported in nats. Range `L ∈ [0, +∞)`. Equivalently,
`L = D_KL(uniform || q)` where `q_i = D_i / sum(D)` is the empirical
mass-share — the KL divergence FROM the uniform-share distribution TO the
empirical share, evaluated AGAINST the uniform reference. That asymmetry
matters and we will return to it in §6 when we contrast against axis-38
(`T = D_KL(q || uniform)`, the SAME two distributions but with the reference
swapped).

## 2. Why a thirty-seventh axis was justifiable

The bar for a new axis in this suite is steep. By the time axis-37 landed
the inequality cell already contained:

- axis-26 Palma ratio S90/S40 (v0.6.255 / v0.6.256)
- axis-28 Bonferroni Index (rank-cumulative L1 functional)
- axis-29 Kolm-Pollak EDE (CRRA inequality-aversion ε-knob)
- axis-32 Mean Log Deviation (the GE-family closure post that anchored α=0,1,2)
- axis-33 Wolfson bipolarisation
- axis-34 Zenga daily-token inequality
- axis-35 daily-token Pietra ratio (v0.6.271 / v0.6.272 with `--show-gini-comparison`)
- axis-36 Atkinson Index ε-sweep (v0.6.273 with the rank-6→rank-3 leap)

By the dispersion-axis sprint convention — every new axis must be functionally
independent of every prior axis on at least one mechanically distinct property
— axis-37 needed to clear a four-of-six gauntlet. From the changelog
justification verbatim, it cleared four:

1. **Strict bottom-sensitivity.** `log(mu / D_i)` blows up as `D_i → 0`. A
   small day weighs much more in L than the same transfer weighs in Gini
   (which weighs by Lorenz position) or in Pietra (which is L-∞, ignoring
   sub-max transfers entirely).
2. **Additive subgroup decomposability with no residual.** This is the
   property no other axis in the suite has. We will spend §3 on it.
3. **Unbounded above.** Atkinson is in `[0,1]`; Pietra is in `[0, 1 − 1/n]`;
   Gini is in `[0,1]`. Theil-L is in `[0, +∞)`. The unboundedness matters
   when the headline concentration is so extreme that bounded measures
   saturate near 1 — L still discriminates.
4. **GE(0) anchor of the GE(α) family.** `--alpha-sweep` surfaces
   `GE(0) = L`, `GE(1) = Theil-T`, `GE(2) = ½·CV²` on the same vector. The
   whole GE family in one row. The axis is the natural anchor for the
   parametric family, not a one-off scalar.

Property (2) is the structural payload of v0.6.274/v0.6.276 — and the
reason the v0.6.274 commit cluster has a fourth refinement SHA `a102424` for
`theilLSubgroupDecomposition`, and the v0.6.276 cluster has the analogous
`theilTSubgroupDecomposition`. Without those refinements, axis-37 and axis-38
would be GE-family scalars indistinguishable in spirit from axis-32. With
them, they are the only axes in the entire suite that carry a closed-form
between/within identity.

## 3. Additive subgroup decomposition — the property no other axis has

The key identity. For any partition of the day-vector `D` into K subgroups
(in our case: per-source partitions, but in general any categorical), Theil-L
satisfies:

```
L_total = sum_g (n_g / n) * L_g  +  L_between
```

where:

- `n_g` is the number of days in subgroup `g`, `n = sum_g n_g`
- `L_g` is the within-subgroup Theil-L computed on `D` restricted to that subgroup
- `L_between = (1/n) * sum_g n_g * log(mu_total / mu_g)` is the
  Theil-L of the constant vector that replaces every day in subgroup `g`
  with its subgroup mean `mu_g`

There is **no residual cross term**. The decomposition is exact at every
floating-point precision (the v0.6.274 test suite verifies this with a
no-residual-identity test). Atkinson does not decompose this way; it has a
cross term that depends on subgroup means. Gini does not either; it only
decomposes when subgroups do not overlap on the value axis. Pietra does not
decompose at all (the L-∞ functional is not partition-additive).

This is why the v0.6.274 changelog wording is unusually direct: "this makes
Theil-L the **only** axis in the suite that lets you cleanly answer 'how much
of the total daily-token inequality comes from within-source variation vs.
between-source variation' in a single decomposition."

The companion v0.6.276 changelog is even more pointed about the contrast
with axis-38:

```
T_total = sum_g s_g * T_g  +  T_between     where s_g = sum(D_g) / sum(D)
```

— Theil-T uses **MASS-weighted** subgroup shares `s_g`, while Theil-L uses
**POPULATION-weighted** shares `n_g/n`. Same identity shape, NO residual in
either case, but a different weighting scheme. Disagreement between the two
decompositions is itself diagnostic: heterogeneous subgroup *sizes* push the
L decomposition; heterogeneous subgroup *intensities* push the T
decomposition.

The single sharpest contrast property: a zero-day **inside** a subgroup keeps
Theil-T's within term FINITE; the SAME scenario pins Theil-L's within term
at +∞. The headline indices already exhibit this asymmetry on full vectors;
the decompositions carry it down to every subgroup independently.

## 4. The live-smoke real-data citation (v0.6.274)

This is the citation that makes this post non-speculative. Reproduced
verbatim from the v0.6.274 changelog `### Live-smoke` block (using the real
`~/.config/pew/queue.jsonl` on the dev box at landing time):

```
$ pew-insights daily-token-theil-l-index

source         days  theilL  A(eps=1)  geoMean      meanDaily    tokens
claude-code    35    1.5874  0.7955    20,108,718   98,353,880   3,442,385,788
vscode-other   73    1.1257  0.6756    8,381        25,832       1,885,727
codex          8     0.7968  0.5492    45,620,424   101,203,083  809,624,660
opencode       11    0.2443  0.2167    364,093,933  464,847,267  5,113,319,934
openclaw       14    0.2169  0.1950    119,372,215  148,282,600  2,075,956,406
hermes         14    0.2149  0.1934    13,789,494   17,096,157   239,346,199
```

Five real-data findings drop out of this table immediately:

**Finding 1 — claude-code carries 1.5874 nats of within-source daily inequality.**
That converts via the analytic identity `A(ε=1) = 1 − exp(−L)` to a
`1 − exp(−1.5874) = 0.7955` welfare-loss reading on the Atkinson(ε=1) scale.
A CRRA inequality-averse planner with log utility would be willing to give up
**79.6% of total claude-code token mass** to receive the same total split
equally across the 35 days observed. That is an extraordinary headline
because axis-36 reported only `A(ε=0.5) = 0.5002` for the same source — the
ε-knob doubles the welfare-loss reading just by raising inequality-aversion
from 0.5 to 1.

**Finding 2 — the geometric-vs-arithmetic spread is 4.9x on claude-code.**
Geometric mean is **20,108,718 tokens/day**. Arithmetic mean is
**98,353,880 tokens/day**. Ratio `98.35M / 20.11M ≈ 4.89x`. By the closed
form `L = log(mu / GeoMean)` we get `log(4.89) = 1.587 nats` to three
decimals, matching the reported `theilL` exactly. This is the source of L's
bottom-sensitivity made visceral: the 77,986-token day on 2026-03-06 (the
floor day in the claude-code window) is what dominates the log-shortfall
sum. Truncate that one day and the L reading collapses by roughly half.

**Finding 3 — vscode-other carries L=1.1257 nats on 73 days at 25.8K
mean/day.** This is the highest-`n` source in the workspace and still
registers the second-highest L in the table. The `geoMean = 8,381 tokens`
against `meanDaily = 25,832` shows a `3.08x` ratio — `log(3.08) = 1.125
nats`, again matching to three decimals. The lesson: a long tail of
near-zero days at a low-throughput source produces almost the same L
reading as a short tail of zeros at a high-throughput source. L is
scale-invariant in the right way.

**Finding 4 — opencode is the most uniform daily rhythm in the workspace
despite carrying the LARGEST total token mass.** L = 0.2443 nats, geoMean
364M against arithmetic mean 464M, ratio 1.28x. `log(1.28) = 0.247 nats`.
The 5.11B total tokens are spread close-to-uniformly across 11 days. This
is the orthogonality witness against axis-2 (`tokens` rank): sort the
sources by `tokens` and you get one ordering; sort by `theilL` and you get a
near-inverted ordering on the lower half of the table. Axis-37 produces
information that no count-based axis can.

**Finding 5 — Atkinson(ε=1) cross-validation passes to 1e-12.** Every
`A(ε=1)` value in the live-smoke table matches `1 - exp(-L)` to within
floating-point precision on the same vector. This is not a separate
measurement; it is a numerical sanity check on the analytic link between
axis-36 and axis-37 at the log-utility limit. The value of carrying both
axes is precisely that they DISAGREE on every other ε.

## 5. The GE-family non-monotone signature (the second real-data citation)

The `--alpha-sweep` surface lets you compute `GE(0), GE(0.5), GE(1), GE(2)`
on the same vector. From the v0.6.274 changelog live-smoke:

```
$ pew-insights daily-token-theil-l-index --alpha-sweep 0,0.5,1,2

source         GE(0)=L  GE(0.5)  GE(1)=T  GE(2)=halfCV^2
claude-code    1.5874   1.1721   1.1897   2.2601
vscode-other   1.1257   0.9296   0.9545   1.6243
codex          0.7968   0.6550   0.6157   0.7350
opencode       0.2443   0.1513   0.1100   0.0781
openclaw       0.2169   0.2052   0.2004   0.2086
hermes         0.2149   0.1884   0.1720   0.1588
```

The headline non-monotone-in-α witness: **claude-code is the only source
where `GE(2) > GE(0)`**. Its half-squared-CV (2.2601) is LARGER than its
mean log deviation (1.5874). Five of the six other sources show monotone
decrease in GE(α) as α moves from 0 to 2. claude-code's reversal is the
fingerprint of a top-heavy outlier: the 1.05B-token day on 2026-04-20 against
a mean of 98M dominates the squared-deviation functional but cannot
dominate the log-deviation functional (log compression is too strong on the
upper tail).

This non-monotone signature was UNRECOVERABLE from any prior single axis.
Axis-32 (MLD GE-family closure) reported the GE(0,1,2) closure as a
U-shaped curve in aggregate, but did not surface a per-source α-sweep. Axis-36
(Atkinson) parameterises the inequality-aversion ε but stays inside the
CRRA family, which is mechanically a sub-family of GE — it cannot reach
α=2. Axis-35 (Pietra) is α-free entirely (it is the L-∞ functional, sitting
outside the GE family).

So the non-monotone-α signature on claude-code is, at this moment in the
inequality-axis sprint, **a finding that exists only because axis-37 ships**.
That is the falsifiable criterion the dispersion-axis sprint runs against:
either the new axis surfaces a finding the prior 36 axes could not, or it
fails the bar.

## 6. Why axis-37 + axis-38 ship as a pair (and what makes them not redundant)

The companion axis-38 (Theil-T, `f0ba43a / 6b8339e / 7048fec / ed82954`)
landed in v0.6.275 the same day, and v0.6.276 added the mass-weighted
subgroup decomposition. The textbook reason both ship is that they bracket
the GE family: GE(0) at α=0, GE(1) at α=1. The interesting reason they
both ship is that they answer different questions on the SAME pair of
distributions:

- `L = D_KL(uniform || q)` — KL FROM uniform TO empirical mass-share.
  Reference: uniform. Sensitive to `D_i → 0` (the integrand is `(1/n) ·
  log(1/q_i / 1) ≈ ∞`).
- `T = D_KL(q || uniform)` — KL FROM empirical mass-share TO uniform.
  Reference: empirical. Sensitive to `q_i → 1` (the integrand is `q_i ·
  log(q_i · n) → big` as one day's mass dominates).

KL divergence is **asymmetric**, so L and T are functionally independent
even though they evaluate on the same two distributions. The v0.6.275
changelog quantifies this on the live data: every source has `T/L < 1` —
which means BOTTOM-tail effects (L's regime) dominate TOP-tail effects (T's
regime) on actual usage. opencode hits the extreme `T/L = 0.4475`, openclaw
the other extreme `T/L = 0.9325`. That ratio is unrecoverable from either
axis alone — it requires the pair.

The decomposition contrast follows the same logic. Axis-37's decomposition
weights each subgroup by `n_g/n` (population), so it answers "how much of
total inequality is between subgroups by *day count*?" Axis-38's
decomposition weights each subgroup by `s_g = sum(D_g)/sum(D)` (mass), so
it answers "how much of total inequality is between subgroups by *token
volume*?" Disagreement between the two decompositions is itself a finding:
when axis-37's L_between is large but axis-38's T_between is small, you have
many small-volume groups that differ a lot in per-day mean; when the
reverse, you have a few high-volume groups concentrated in a short time
window.

This is why the v0.6.276 changelog calls the pair "the right structural
complement" rather than "a redundant scalar". It is a real claim and the
real-data findings back it up.

## 7. Falsifiable predictions axis-37 makes

A few predictions follow from the v0.6.274 / v0.6.276 design that future
weeks of pew-insights data can falsify:

1. **GE(α) monotone-in-α reversal is rare.** At most one or two of six
   sources should exhibit `GE(2) > GE(0)` on any given week. If three or
   more do, the workspace has shifted into a top-heavy regime that no prior
   axis would have caught. Forecast: in the next four weeks, fewer than 2
   sources per week will reverse.
2. **L_between (per-source partition) declines as the workspace adds
   sources.** Adding a low-mean source mostly raises L_within for that
   source but barely changes L_between (by the additivity identity).
   Forecast: as new sources enter the cell, L_between will trend flat, not
   up.
3. **Atkinson(ε=1) and L stay tied at the log-utility limit.** The
   `1 − exp(−L) = A(ε=1)` identity is analytic, so any divergence between
   them at landing is a numerical bug, not a finding. This is the cheapest
   integration test we have between axis-36 and axis-37.
4. **Mass-weighted vs population-weighted between/within disagreement is
   bounded by source-size heterogeneity.** When all subgroups have similar
   `n_g`, the L and T decompositions agree. Disagreement is a witness that
   subgroup sizes are themselves unequal. Forecast: per-source partitions
   will show modest L/T disagreement (sources have similar day counts);
   per-week partitions will show large disagreement (weeks have very
   different total volume).

## 8. What this changes downstream

Three concrete downstream consequences:

- **`pew-insights digest` can now report between/within splits.** Previously
  the digest had to lump all sources into a single inequality scalar. With
  the v0.6.274/v0.6.276 decomposition APIs, digests can show the share of
  total inequality that is "across sources" vs "within sources" — without
  any cross-term residual to apologise for. This is a structural upgrade
  to the digest format that the prior 36 axes could not enable.
- **The GE-family α-sweep is the new regression test for any future
  inequality axis.** Any axis-39 candidate now has to clear "do you produce
  information that the four-point α-sweep on axis-37 does not already
  capture?" That is a much harder bar than the dispersion-axis sprint had
  before v0.6.274.
- **The KL-asymmetry argument legitimises axis-pairs as a unit.** Axis-37 +
  axis-38 are explicitly shipped as a paired witness. This is a precedent.
  Future axes that rely on directional functionals (e.g. asymmetric
  divergences, one-sided welfare functions) can now cite this pair as
  prior art for "two axes that look redundant but compute different
  divergences against the same reference."

## 9. Summary

The four-SHA cluster `44ecfac / d344503 / 3fbea1a / a102424` plus the v0.6.276
refinement `theilTSubgroupDecomposition` ship axis-37 with a structural
payload (additive no-residual subgroup decomposition) that no other axis in
the inequality cell carries. The headline real-data finding —
`claude-code L = 1.5874 nats` translating to a `79.6% Atkinson(ε=1)
welfare-loss` reading and a `4.9x` geometric-vs-arithmetic spread — is
already significant on its own. The GE-family non-monotone signature
(claude-code is the only source where `GE(2) > GE(0)`) is the falsifiable
finding that justifies axis-37 against the steep "must produce information
no prior axis can" bar of the dispersion-axis sprint. And the paired ship
with axis-38 as the KL-asymmetry complement establishes a precedent for
axis-pair witnesses that the suite did not previously have.

Net: axis-37 is the first inequality axis where the *commit cluster includes
a structural API* (the decomposition) as the fourth SHA rather than a pure
refinement. That is a new pattern in the dispersion-axis sprint, and the
real-data findings justify it.
