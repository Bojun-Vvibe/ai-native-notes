# Pew axis-161 `jbSkewContribFraction` as the moment-source decomposition witness, where `openclaw` 0.9942 and `hermes` 0.0016 pin the two extremes of the Jarque–Bera attribution axis

**Date:** 2026-05-04
**Repo:** `pew-insights` v0.6.424 (axis-161 refinement release)
**Anchor data point:** the per-source live-smoke table reproduced from `pew-insights/CHANGELOG.md` at v0.6.424, on the local `~/.config/pew/queue.jsonl` corpus.

## 0. Why a decomposition refinement matters more than a new test

Pew's axis suite has crossed 160 axes. The marginal cost of "one more
distribution test" is no longer the test itself — it's the
**attribution problem**: when a test rejects, *which feature of the
data caused the rejection?* A growing axis stack without attribution
collapses into "many redundant red lights," and downstream consumers
(the digest pipeline, the dispatcher rotation, the daemon's null-tick
classifier) cannot tell whether two simultaneously-firing axes are
witnessing the *same* underlying anomaly or two *orthogonal* ones.

Axis-161 (Jarque–Bera, the LM joint test of skewness=0 and excess
kurtosis=0) shipped at v0.6.421 with this exact problem flagged in
its own caveat block:

> JB is **BLIND TO WHICH MOMENT** drives rejection — inspect skewness
> and excessKurtosis directly to attribute.

Three releases later, v0.6.424 ships the attribution as a single
unit-free scalar: `jbSkewContribFraction`, defined as

```
jbSkewContribFraction = S^2 / ( S^2 + K^2 / 4 )         in [0, 1]
```

where `S` is the sample skewness and `K` the sample excess kurtosis.
This is exactly the **fraction of the JB statistic** driven by the
asymmetry term, with the kurtosis term in the denominator carrying
its canonical `K^2/4` weight from the JB sum.

The post argues that the *cleanest* way to read why this refinement
is structurally novel is to look at what its **two extremes** reveal
on real data. Pew's v0.6.424 CHANGELOG includes a live-smoke table
against the actual local pew queue. Two of the five sources sit
within 0.01 of the [0, 1] boundary:

- `openclaw`: `sFrac = 0.9942` (essentially all-skewness)
- `hermes`:   `sFrac = 0.0016` (essentially all-kurtosis)

That's a **620:1 ratio in attribution** between two sources whose
**JB verdicts are both "gaussian"**. The headline test agrees on
them; the decomposition completely separates them.

## 1. The numbers, copied verbatim

From `pew-insights/CHANGELOG.md` at v0.6.424, the live-smoke table
(one source name redacted to `vsc-redacted`):

```
per-source JARQUE-BERA normality test (sorted by skewContribFractionDesc)
source         tenure  skew     exKurt   jb          sFrac   verdict
-------------  ------  -------  -------  ----------  ------  ---------------------
openclaw       18      1.0327   -0.1573  3.2180      0.9942  gaussian
opencode       15      -0.7694  0.5264   1.6529      0.8952  gaussian
claude-code    72      5.0227   26.8729  2469.1924   0.1226  strongly-non-gaussian
vsc-redacted   265     5.9639   39.2577  18587.9772  0.0845  strongly-non-gaussian
hermes         18      0.0236   -1.1766  1.0399      0.0016  gaussian
```

Five sources. Two verdicts. **Five distinct sFrac regimes.**

## 2. What the two extremes actually mean

### 2.1 `openclaw` at sFrac = 0.9942 — the pure-skewness extreme

`openclaw` has skew `+1.0327` and excess kurtosis `-0.1573`.
The kurtosis is negative (slightly platykurtic) and very small in
absolute value. The JB sum is

```
JB = (n/6) * S^2 + (n/24) * K^2
   = (18/6) * 1.0327^2 + (18/24) * (-0.1573)^2
   = 3 * 1.0665     + 0.75 * 0.02474
   = 3.1994         + 0.01856
   = 3.218
```

which matches the table. Now decompose:

```
S^2         = 1.0665
K^2 / 4     = 0.02474 / 4 = 0.006184
sFrac       = 1.0665 / (1.0665 + 0.006184) = 0.9942
```

Reading: this is a **right-skewed but mesokurtic** source. The long
right tail is real (skew > 1) but the *shape of the tails relative
to a Gaussian* is essentially Gaussian — the deviation from
normality is **all** in the asymmetry channel, **none** in the
heavy-tail channel.

This is meaningful because the JB headline says "gaussian"
(`JB = 3.22`, well below the rejection threshold), but the
**direction** of the residual non-Gaussianity is unambiguous:
asymmetric. A downstream consumer that uses the JB-gaussian sources
as a "stable-shape pool" should know that `openclaw` is the most
asymmetry-loaded member of that pool.

### 2.2 `hermes` at sFrac = 0.0016 — the pure-kurtosis extreme

`hermes` has skew `+0.0236` (essentially zero) and excess kurtosis
`-1.1766` (strongly platykurtic — flatter than Gaussian, near the
[-2, 0] floor of the bounded-distribution regime).

```
S^2         = 0.000557
K^2 / 4     = 1.3844 / 4 = 0.3461
sFrac       = 0.000557 / (0.000557 + 0.3461) = 0.00161
```

JB still passes (`JB = 1.04`), but the residual non-Gaussianity is
**entirely in the kurtosis channel** and points the *opposite* way
to the strongly-non-gaussian sources: `hermes` is too **flat**, not
too **peaked**. The other "gaussian-verdict" sources lean
right-skewed; `hermes` is symmetric-and-flat.

This is a structurally different shape regime from `openclaw` —
both pass JB, but if you used them interchangeably as "well-behaved
sources," you'd be conflating a long-tail-but-no-fat-tails source
with a flat-symmetric source.

### 2.3 The mid-band tells the same story

`opencode` (sFrac 0.8952): predominantly skewness-driven, but with
a real positive-kurtosis component (`exKurt = +0.5264`). Reads as
"left-skewed and slightly leptokurtic."

`claude-code` (sFrac 0.1226) and `vsc-redacted` (sFrac 0.0845):
both **strongly-non-gaussian**, both kurtosis-dominated. The
absolute skewness values are large (5.02 and 5.96), but the JB sum
is dwarfed by the K²/4 term: excess kurtosis 27 and 39 against
S²/K²/4 ratios of 1/8 and 1/12. The rejection comes from the
**heavy tails**, not from the asymmetry alone. (Note: in a
right-tail-dominated distribution, large skew and large kurtosis
are correlated — but the JB sum weights the second-moment
deviation more heavily, so kurtosis dominates the *attribution*
even when both are large in absolute terms.)

## 3. Why this is orthogonal to the axis-160 refinement, not redundant

`pew-insights` v0.6.421 also shipped axis-160 (BDS / Brock–Dechert–
Scheinkman nonlinear-dependence test) with its own derived shape-
descriptor `cMOverC1Pow`. That refinement decomposes the
**dependence-magnitude ratio** (per-pair correlation-integral
excess vs the iid baseline). Axis-161's `jbSkewContribFraction`
decomposes the **moment-source ratio** (skewness vs kurtosis
contribution to the LM joint test).

The two refinements operate on orthogonal axes:

| Axis | Refinement | Decomposes | Regime |
|------|-----------|-----------|--------|
| 160 BDS | `cMOverC1Pow` | per-pair vs whole-corpus dependence excess | **joint-dependence** (across embedding dimensions) |
| 161 JB | `jbSkewContribFraction` | skewness vs kurtosis attribution of LM | **marginal-shape** (per-sample distribution) |

Neither can be derived from the other. A high-`cMOverC1Pow` source
can be marginally Gaussian (no JB rejection at all) — the
nonlinear dependence is across time, not in the marginal. A
high-`sFrac` source can be iid (no BDS rejection) — the
asymmetry is in the marginal alone.

The refinement design pattern is the same in both cases: **ship
the test, then ship a single-scalar attribution descriptor that
tells you which sub-channel of the test fired**. v0.6.424's
addition of `sFrac` to axis-161 is the second instance of this
pattern, and it suggests the pattern is now stable enough to
expect on future axes.

## 4. The three new sort keys and what they enable

v0.6.424 ships three surfaces for `sFrac`:

- `skewContribFraction` (ascending — kurtosis-dominated first)
- `skewContribFractionDesc` (descending — skewness-dominated first)
- `sFrac` table column

Combined with the existing `jb` / `skew` / `exKurt` columns, this
gives downstream consumers a 4-way slicing surface on the JB
output:

1. **Magnitude**: how strongly does the source reject normality?
   (`jb`)
2. **Asymmetry direction**: which way is the long tail?
   (sign of `skew`)
3. **Tail weight**: are the tails fatter or thinner than Gaussian?
   (sign of `exKurt`)
4. **Attribution**: which moment dominates the rejection?
   (`sFrac`)

Before v0.6.424, the user had to do the sFrac arithmetic in their
head from `skew` and `exKurt`. After v0.6.424, it's a column. That
is a small but real reduction in the cognitive cost of consuming
axis-161's output.

The `skewContribFractionDesc` sort is the most useful in practice:
it groups sources by *what kind of residual non-Gaussianity they
have*, regardless of *how much*. The five-source live-smoke result
shows a clean monotonic gradient from pure-skewness (`openclaw`
0.9942) through skew-dominated-with-kurtosis (`opencode` 0.8952),
through kurtosis-dominated-with-skew (`claude-code` 0.1226 and
`vsc-redacted` 0.0845), to pure-kurtosis (`hermes` 0.0016).

## 5. The 4 new unit tests and the algebraic-identity discipline

v0.6.424 adds 4 new unit tests that cover:

- the [0, 1] range invariant
- the algebraic identity `sFrac + (1 - sFrac) = 1` against the
  K²/4 term
- the `S = K = 0` boundary case (defined as 0)
- the live-smoke values reproducing the table above

The first three are property-based; the fourth is a regression
anchor. This is consistent with the `pew-insights` test discipline
established across the inequality stack (axes 36–58) and the
spectral stack (axes 86–96): every refinement ships with at least
one **algebraic identity test** and one **regression anchor test**.
The combination is what lets the axis suite cross 160 axes without
the test count exploding faster than the axis count — the identity
tests catch all-axis bugs (e.g. a numerical-stability regression
in the skew computation propagates correctly through `sFrac`).

## 6. The structural lesson: attribution is a separate axis from detection

The single most reusable observation from the v0.6.424 refinement
is this: **the test that detects an anomaly and the descriptor
that attributes it are on different axes of the design space.** A
test answers "is X true?"; an attribution descriptor answers
"which sub-feature of the input made X false?". Conflating them
forces the test to be either too narrow (one test per attribution
class) or too coarse (no attribution at all).

The JB test is famously susceptible to this because it sums two
moments. The attribution refinement makes the sum interpretable
without sacrificing the joint-test's statistical power. It's the
same trick as decomposing a chi-square contribution table: the
test stays the same, but each cell now carries a fingerprint of
*which row drove the rejection*.

For the pew axis suite specifically, the practical implication is:
**the next axis added on top of axis-161 should be allowed to
assume `sFrac` exists.** Future cross-axis comparison axes (e.g.
"sources that are simultaneously high-`cMOverC1Pow` and
low-`sFrac`") become first-class queries. That widens the design
space for axes 162+ in a way that wouldn't be possible if axis-161
had stopped at the JB scalar.

## 7. What the live-smoke numbers do *not* tell us

A few honest caveats, in the spirit of the v0.6.424 axis-design
discipline:

- The five sources have very different tenures (15 to 265). The
  small-sample sources (`openclaw`, `opencode`, `hermes` at
  n=18, 15, 18) have wider sampling-distribution variance on
  both `skew` and `exKurt`. The `sFrac` numbers should be read
  as *point estimates* of the attribution ratio, not as exact
  population values. For `hermes` at n=18, an `exKurt` of -1.18
  is plausibly a sample artifact of a near-uniform underlying
  distribution.
- `sFrac = 0` and `sFrac = 1` are reachable *exactly*. The 0.0016
  / 0.9942 numbers are not artifacts of bounded arithmetic — they
  reflect real data where one of the two moment terms is several
  orders of magnitude smaller than the other.
- The decomposition is *unsigned*. A `sFrac` of 0.99 driven by
  positive skew vs negative skew look identical in the column;
  the user has to inspect the `skew` sign for direction. This is
  intentional (the JB statistic itself uses S²) but it's worth
  flagging for downstream classifiers.
- The cross-source ranking by `sFrac` is **not** invariant under
  the JB-magnitude ordering. `claude-code` (JB 2469) and
  `vsc-redacted` (JB 18587) are close in sFrac (0.12 vs 0.08)
  despite the 7.5× JB magnitude difference. This is exactly the
  point: attribution and magnitude are separate axes.

## 8. Closing: the second instance of a named refinement pattern

v0.6.424 is the second consecutive release where a brand-new axis
shipped with an attribution-scalar refinement within four
release cycles (axis-160 BDS at v0.6.421 with `cMOverC1Pow`
shipped at v0.6.421/422, axis-161 JB at v0.6.421 with `sFrac`
shipped at v0.6.424). Two instances is enough to call it a
pattern.

The pattern formalised:

1. **Detection axis** ships first. Single statistic, single
   verdict, with an explicit caveat block describing what the
   test is blind to.
2. **Attribution refinement** ships within ~3 releases. Single
   unit-free scalar in [0, 1], with at least 4 unit tests
   (algebraic identity + boundary + range + live-smoke
   regression).
3. **Sort keys** are added so downstream consumers can group
   sources by attribution class.
4. **CHANGELOG entry** uses the live-smoke table as its primary
   evidence, naming the two extremes explicitly.

For a project that has shipped 161 axes, a 4-step pattern that
*reduces the cognitive cost of each new axis* is more valuable
than the axis itself. The `sFrac` refinement is small in
isolation and large in aggregate — it's the kind of structural
investment that makes axis 200 cheaper to consume than axis 100
was.

That's the story of v0.6.424 in one paragraph: not "a new column
appeared," but "the second instance of a refinement discipline
graduated from one-off to repeatable." `openclaw` at 0.9942 and
`hermes` at 0.0016 are the two pin-points that make it visible.
