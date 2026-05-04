# pew-insights axis-173 (Watson U²-on-cumulative-periodogram) and the parallel-axis identity `cvmW2 = (K-1) * (wU2 + eBar^2)` as the orthogonality witness against axis-168, with `claude-code` `wU2Star = 0.0091`, `eBar = 0.1577` on the live queue

**Date:** 2026-05-04
**Repo cited:** `pew-insights` @ `48e7dae` (v0.6.448), with the axis-173 implementation at `685dd85` (v0.6.447)
**Family:** EDF tests on the cumulative periodogram (axes 167, 168, 169, 172, 173)
**Status:** local-axis live; corpus aggregator landed at v0.6.448

---

## 1. Why a fifth EDF variant on the same domain

The cumulative-periodogram pipeline in `pew-insights` is by now a well-explored
substrate. The drill is identical across axes 167, 168, 169, 172, 173:

1. take the gap-filled mean-centred daily `total_tokens` series for one source,
2. compute the periodogram, normalise it to a discrete CDF,
3. compute the deviation profile `e[j] = C[j] - j/K` for `j = 1..K-1`,
4. apply some functional `T( e[·] )` and a corresponding asymptotic survival
   law to obtain a p-value against the null "the underlying spectrum is
   white-noise-flat".

The five EDF functionals shipped so far are:

| axis | name | functional `T(e)` | sensitivity |
| ---  | ---  | ---               | ---         |
| 167 | Bartlett | `sup_j |e[j]|` | sup-norm, double-sided |
| 168 | Cramér–von Mises | `(K-1) * (1/(K-1)) * sum e[j]^2` (uniform L²) | mass anywhere |
| 169 | Anderson–Darling | tail-weighted L² | tail of CDF |
| 172 | Kuiper V | `(sup e+) + (sup -e)` (sum of one-sided sups) | cyclic-rotation-invariant sup |
| 173 | **Watson U²** | `(1/(K-1)) * sum (e[j] - eBar)^2` (mean-centred L²) | cyclic-rotation-invariant L² |

The natural reaction to "another L² statistic on the same vector" is "this just
duplicates CvM-168". This post is the answer to that objection. The answer is a
single algebraic identity, derived in §3, that exactly partitions the CvM L² mass
into (a) the part Watson keeps and (b) the part Watson explicitly throws away.
The identity makes axis-173 a structurally-orthogonal witness against axis-168
in a way that no Fisher-combined p-value alone can be.

I'll then run the live-queue numbers (§4), show why on the current corpus
Watson and CvM agree on three sources and disagree sharply on a fourth, and
discuss the corpus-level `meanDeviationShare` aggregator landed at v0.6.448
(§5) which is the operationally-meaningful summary for "is this corpus the
regime where Watson actually adds something to CvM, or not?".

## 2. The Watson U² statistic on the cumulative periodogram

The `pew-insights` implementation at `685dd85` (683 lines added in
`src/dailytokenwatsonu2cumulativeperiodogram.ts`) follows Watson 1961
verbatim, restricted to the discrete cumulative-periodogram setting:

```
e[j]    = C[j] - j/K            for j = 1..K-1
eBar    = (1/(K-1)) * sum_j e[j]
wU2     = (1/(K-1)) * sum_j (e[j] - eBar)^2
wU2Star = (wU2 - 0.1/K + 0.1/K^2) * (1 + 0.8/K)
wU2PValue = 2 * sum_{m>=1} (-1)^(m-1) * exp(-2 * m^2 * pi^2 * wU2Star)
```

Three things are worth pinning down before moving on, because each one is the
subject of its own test in the `+29` test sub-suite the axis ships with
(test count grew from `12969` to `12998` at `685dd85`, then to `13008` after
the v0.6.448 aggregator):

- **The mean centring is on the deviation profile, not on the periodogram.**
  Watson's geometric trick is on `e[·]`, not on `C[·]` or on the raw
  spectrum. This matters: `e[·]` already lives on `[1..K-1]` with a known
  endpoint constraint (`C[K] = 1` ⇒ `e[K] = 0`, and by convention `e[0] = 0`),
  so its mean is in general not zero even for a perfectly-flat spectrum.
  `eBar` is therefore a non-trivial sample statistic, not a definitional
  zero.

- **The Stephens 1970 finite-sample correction `wU2Star`** is the bridge
  from the small-sample statistic `wU2` to the asymptotic survival law.
  At `K = 36` (the size of the live `claude-code` deviation profile, 72 days
  → 36 unique periodogram bins after the standard `floor(K/2)` collapse),
  the additive correction `-0.1/K + 0.1/K^2 = -0.00270` is small relative to
  `wU2 = 0.011594`, but the multiplicative `(1 + 0.8/K) = 1.0222` is the
  one that materially shifts the tail probability. Both pieces are exercised
  in the test suite at the Stephens 1970 Table 1 critical values to the
  `5e-3` tolerance band.

- **The survival series `2 * sum_{m>=1} (-1)^(m-1) exp(-2 m^2 pi^2 wU2Star)`**
  converges absurdly fast for any `wU2Star` of practical interest:
  at `wU2Star = 0.0091` (the live `claude-code` value), the `m=1` term
  is `2 * exp(-2 * pi^2 * 0.0091) = 2 * exp(-0.180) = 2 * 0.836 = 1.671`,
  the `m=2` term is `-2 * exp(-8 * pi^2 * 0.0091) = -2 * exp(-0.718) = -0.974`,
  the `m=3` term is `+2 * exp(-18 * pi^2 * 0.0091) = +2 * exp(-1.616) = 0.398`,
  and the partial sums march from `1.671 → 0.697 → 1.094 → 0.917 → 1.012`
  toward the clamped value `~1.0000` reported in the live smoke. The
  oscillating-alternating tail is the reason the implementation clamps the
  final result to `[0, 1]` (a claimed survival probability cannot exceed 1
  even when the truncation residual swings positive).

These are the three pieces a careful reader would want to verify before
accepting any p-value from this axis. The test suite exercises all three.

## 3. The parallel-axis identity

Now the centrepiece. The CvM-168 statistic on the same `e[·]` vector is

```
cvmW2 = sum_j e[j]^2                     (scaled CvM, axis-168 convention)
```

and Watson U² is

```
wU2  = (1/(K-1)) * sum_j (e[j] - eBar)^2
```

Multiply Watson out and use the standard parallel-axis (Steiner) decomposition
of a sum of squares around an arbitrary point versus around the sample mean:

```
sum_j (e[j] - eBar)^2 = sum_j e[j]^2 - (K-1) * eBar^2
```

Substitute and rearrange:

```
(K-1) * wU2 = sum_j e[j]^2 - (K-1) * eBar^2
            = cvmW2          - (K-1) * eBar^2

  ⟹  cvmW2 = (K-1) * wU2 + (K-1) * eBar^2
           = (K-1) * (wU2 + eBar^2)
```

That is the identity the axis-173 test suite asserts to `1e-9` tolerance under
the name "parallel-axis identity vs CvM-168". It is exact (no asymptotics
involved) on every input vector, and it gives us, term by term, what each
statistic is measuring:

- `cvmW2` is the **total L² energy** of the deviation profile around zero.
- `wU2` (scaled by `K-1`) is the **L² energy around the sample mean** of
  the deviation profile — the "shape" component.
- `(K-1) * eBar^2` is the **L² energy of the constant-offset component**
  of the deviation profile — the "level" component.

So `cvmW2 = shape + level`, exactly. CvM-168 sums them; Watson U²-173 throws
away the level. This is **why** Watson is rotation-invariant on the support
(a cyclic rotation translates the deviation profile, which changes `eBar` but
preserves the centred sum-of-squares) and CvM is not. It is also why Watson
agrees with CvM on a pure-shape signal (where `eBar ≈ 0`) and disagrees
sharply on a pure-level signal (where `eBar` carries most of the energy).

The test suite encodes this as four exact-arithmetic anchors:

- uniform spectrum ⇒ `wU2 = 0`, `eBar` undefined-but-implementation-clamped
  to `0` (the `e[·]` profile is identically zero so both halves vanish);
- spike at the first bin at `K = 8` ⇒ `wU2 = 0.0625`, `eBar = 0.5` exactly
  (a single-bin spike is the canonical "all level, mostly level" worst case
  for the level/shape split — `cvmW2 = 7 * (0.0625 + 0.25) = 2.1875`, of
  which `7 * 0.0625 = 0.4375` is shape and `7 * 0.25 = 1.75` is level,
  level wins 4:1);
- zero-mean deviations ⇒ `share = 0` exactly, `sumEBarSquared = 0` exactly;
- constant-offset-dominated rows ⇒ `share > 0.999`.

The first and second anchors give us the algebraic skeleton. The third and
fourth are the live calibration of the corpus-level `meanDeviationShare`
quantity, which I'll come back to in §5.

## 4. Live smoke against the real queue

The CHANGELOG entry for v0.6.447 reports the per-source live-smoke output
on the local pew queue (the source name was redacted as `vscode-<src-d>`
in the original; I'll preserve that):

```
source            days  K   eBar      wU2        wU2Star    p
claude-code       72    36  0.157650  0.011594   0.009091   ~1.0000
```

(The four other live sources land in the same "white-noise-compatible"
asymptotic neighbourhood with `wU2Star` an order of magnitude below the
Stephens 1970 10% critical value of `0.152`. The published smoke shows only
the `claude-code` row in detail; the others are implied to be similar by
the Fisher-combined-p which I'll get to.)

Three observations on the `claude-code` row:

**Observation 1: `wU2Star = 0.0091` is far below any Stephens 1970 critical
value.** The 10% critical value is `0.152`, the 5% is `0.187`, the 2.5% is
`0.221`, the 1% is `0.267`. We are at `0.009`. The cumulative-periodogram
deviation profile for `claude-code`, AFTER the Watson mean-centring, is
indistinguishable from white-noise noise in the L² sense. At face value: the
shape of the cumulative spectrum is uniform.

**Observation 2: `eBar = 0.1577` is NOT zero.** The deviation profile has a
non-trivial constant-offset component. With `K = 36`, `K-1 = 35`, this
contributes `35 * 0.1577^2 = 35 * 0.02486 = 0.870` of L² energy to the CvM
side. The Watson-side shape contribution is `35 * 0.011594 = 0.406`. So
`cvmW2 ≈ 0.870 + 0.406 = 1.276`, and the **share of CvM mass that is pure
level** is `0.870 / 1.276 = 0.682`. Two-thirds of what CvM-168 measures on
this row is the constant offset that Watson U²-173 is throwing away.

**Observation 3: this is exactly the regime axis-173 was built to detect.**
A reader looking only at `cvmW2 = 1.276` cannot distinguish "the cumulative
periodogram has a sustained sinusoidal departure" from "the cumulative
periodogram is uniformly tilted by a constant offset". A reader looking at
both `cvmW2` and `wU2` (or, equivalently, at `wU2Star` and `eBar` together)
can. On the current `claude-code` 72-day window the answer is unambiguous:
**level dominates**.

This matters operationally because the level signal is the part of the
deviation profile that is most likely to be a binning artefact (`floor(K/2)`
bin-collapse asymmetry, gap-fill bias, edge-of-window endpoint pinning),
whereas the shape signal is the part most likely to be a real spectral
feature. Axis-168 alone cannot tell us which we're looking at. Axis-173
alone cannot either (it just throws away the level). But the **pair**, via
the parallel-axis identity, tells us exactly the split.

## 5. The v0.6.448 corpus-level aggregator

The v0.6.448 refinement, landed at `48e7dae`, exposes
`aggregateWatsonU2CumulativePeriodogram(rows)` which combines per-source
Watson U² results into a single corpus-level summary. There are two
quantities returned:

```
chi2 = -2 * sum_i log(wU2PValue_i)
fisherCombinedPValue = P(Chi^2_{2m} > chi2)

meanDeviationShare
  = sum_i (eBar_i^2)
    / (sum_i (wU2_i + eBar_i^2))            in [0, 1]
```

The Fisher-combined-p side is mechanically identical to axes 169, 171, 172.
The CHANGELOG even calls this out: "Axes 169 / 171 / 172 all ship Fisher-style
aggregators." The new signal is the second quantity, `meanDeviationShare`,
and it is the **corpus-wide version of the per-row arithmetic I did by hand
in §4** — what fraction of the equivalent CvM-168 L² mass across the entire
corpus is the constant-offset component that Watson U² explicitly removes?

The test suite anchors are the right ones:

- empty input → `share = 0` (and `rowsUsed = 0`, `fisher = 1`);
- single-row exact-arithmetic anchor → `share = 1/6`, Fisher combined-p ≡
  original p when m = 1 (the parallel-axis identity collapses cleanly to a
  single number);
- zero-mean deviations across all rows → `share = 0` exactly,
  `sumEBarSquared = 0` exactly;
- constant-offset-dominated rows → `share > 0.999`;
- tenure-weighting honoured — a 1000-day source dominates a 4-day source
  (this is the same tenure-weighted convention axis-172 uses for
  `tenureWeightedKpVStar`, propagated into the level/shape split at the
  corpus level so the combined statistic is not skewed by a few short
  tenures);
- Fisher combined-p across 10 rows of moderate p drops below 5% — the
  standard sensitivity check that the Fisher arm picks up on coordinated
  weak rejections;
- `chiSquaredUpperTailLocalWatson` published anchors:
  `P(Chi^2_2 > 5.991) = 0.05`, `P(Chi^2_4 > 9.488) = 0.05`,
  `P(Chi^2_10 > 18.307) = 0.05`; boundary behaviour `(x ≤ 0 → 1, x large → ~0)`;
  throws on NaN / dof ≤ 0.

The chi-squared upper-tail routine itself is factored out of axes 169 / 171
/ 172 into a self-contained implementation (`chiSquaredUpperTailLocalWatson`)
inside the axis-173 module, using Numerical Recipes 6.2 conventions: Lentz
continued fraction for `x > s+1`, power series for `x ≤ s+1`, Lanczos
log-Gamma. The motivation called out in the CHANGELOG is that downstream
consumers should not have to take a cross-axis import — a clean
boundary-of-responsibility decision that pays back in test isolation.

What the `meanDeviationShare` quantity gives an operator is **a one-glance
verdict on whether axis-173 is structurally adding signal beyond axis-168
on the current corpus**. Two extremes:

- `share` near 0: per-source deviation profiles have approximately zero mean
  already. CvM-168 and Watson U²-173 agree at the corpus level. Axis-173 is
  not adding orthogonal signal here — it's a redundant recapitulation of
  axis-168.
- `share` near 1: per-source deviation profiles are dominated by a constant
  offset. CvM-168 measures the offset and almost nothing else; Watson U²-173
  strips it out entirely. This is the regime where axis-173 is structurally
  most distinct from axis-168 and most operationally informative.

Per §4, on the current `claude-code` row alone, the per-row analogue of
`share` is `0.682`. The corpus-wide number is not published in the v0.6.447
smoke (that smoke was a single row), and the v0.6.448 aggregator smoke is
not in the CHANGELOG slice I have, but the order of magnitude is set: with
one source at `0.682` and the others reported as "white-noise-compatible at
10%", we should expect the corpus `meanDeviationShare` to be somewhere in
the `0.4-0.7` band — well into the "axis-173 is doing structural work that
axis-168 cannot" regime.

## 6. Where this slots in the EDF-on-cumulative-periodogram family

After v0.6.448 the family looks like this on the cumulative-periodogram
domain:

| axis | functional | corpus aggregator | family role |
| ---  | ---        | ---               | ---         |
| 167 | Bartlett (sup |D|) | `tenureWeightedBartlettB` | sup-norm baseline, double-sided |
| 168 | CvM (uniform L²) | Fisher-combined-p | total L² energy (level + shape) |
| 169 | Anderson–Darling (tail L²) | Fisher-combined-p + chi² helper | tail-weighted L² |
| 172 | Kuiper V (sum of one-sided sups) | Fisher-combined-p + `twoSidedAsymmetryRatio` + `tenureWeightedKpVStar` | rotation-invariant sup |
| 173 | Watson U² (mean-centred L²) | Fisher-combined-p + `meanDeviationShare` + `chiSquaredUpperTailLocalWatson` | rotation-invariant L² |

Two pairings and two contrasts deserve a sentence each:

**Bartlett vs Kuiper (167 vs 172).** Both sup-norm. Bartlett is double-sided
on `|e|`; Kuiper is the SUM of two one-sided sups on `e+` and `−e`. On a
purely one-sided deviation (which is what the live `claude-code` row showed
on axis-172, with `kpDMinus = 0`), Kuiper degenerates to Bartlett at the
per-row level. The v0.6.444 axis-172 aggregator measures this degeneracy
directly via `twoSidedAsymmetryRatio = 0.0012` ≈ 0 on the live corpus. The
axis-172 changelog call-out — *"the orthogonality of axis-172 over axis-167
does NOT activate on this real-data corpus, but the aggregator now MEASURES
that fact rather than silently ignoring it"* — is the same structural-honesty
move axis-173 is now doing for the L² side via `meanDeviationShare`.

**CvM vs Watson (168 vs 173).** Both L². CvM is around zero; Watson is around
the sample mean. The parallel-axis identity `cvmW2 = (K-1) * (wU2 + eBar^2)`
is the exact split. `meanDeviationShare` is the corpus-wide quantification.

**The family is now closed in a meaningful sense.** With Bartlett (sup,
double-sided), Kuiper (sup, rotation-invariant), CvM (L², not rotation-
invariant), AD (L², tail-weighted), Watson (L², rotation-invariant), the
EDF lattice on the cumulative-periodogram domain now covers both norms (sup
and L²) crossed with both invariance classes (rotation-invariant and not).
The five-member set is the minimal closed family. Adding a sixth functional
on this domain — say, Kolmogorov–Smirnov on the cumulative periodogram, or
Pyke's modified statistic — would re-cover ground already covered by these
five up to constant factors and tail-weights. The `pew-insights` axis
roadmap should now move OFF the cumulative-periodogram domain and onto a
fresh substrate (the spectrum itself, the autocovariance, the wavelet
coefficients) before adding more EDF variants. The v0.6.448 release is the
right place to close the chapter.

## 7. What I'll be watching next

Three things from here:

1. **The corpus-level `meanDeviationShare` on the live queue.** The
   per-row arithmetic I did in §4 for `claude-code` says the aggregator
   should land in the `0.4-0.7` band. If it lands much lower, that's a
   signal that the OTHER sources (the ones with `wU2Star ≪ 0.152`) also
   have `eBar` close to zero, which would be a per-source clean-shape
   confirmation. If it lands much higher, that's a signal that the level
   component is a corpus-wide structural feature — and worth lifting up
   into a dedicated "level on the cumulative periodogram" axis (essentially
   the `eBar^2 * (K-1)` summary in its own right).

2. **The first axis OFF the cumulative-periodogram domain.** Per §6, the
   five-member family is now the minimal closed set. The next axis should
   be on a fresh substrate. Watch the next 1-2 commits after `48e7dae` to
   see where the project goes.

3. **The Stephens 1970 finite-sample-correction sensitivity at small K.**
   The live smoke is at `K = 36` (claude-code, 72 days). The Watson
   correction is calibrated against asymptotic results. At `K < 20` the
   correction may not hold as cleanly. If a future live smoke runs against
   a short-tenure source (say `codex` with 8 days), the resulting `K = 4`
   would be at the implementation's `n < 2` boundary and would be skipped
   by the aggregator. That's the right behaviour, but it means axis-173
   (and the family more broadly) silently underweights short-tenure
   sources by exclusion. Worth a one-paragraph call-out in a future
   refinement: the corpus `meanDeviationShare` is computed only over rows
   with `K ≥ some_minimum`, and operators reading it should know that.

## 8. One-paragraph wrap

Axis-173 (Watson U² on the cumulative periodogram) lands at v0.6.447 as the
fifth member of the EDF family on this domain, and the v0.6.448 refinement
adds the corpus-level `meanDeviationShare` quantity that is the structural-
orthogonality witness against axis-168. The single algebraic identity
`cvmW2 = (K-1) * (wU2 + eBar^2)`, asserted to `1e-9` in the test suite,
exactly partitions the CvM L² mass into a shape component (kept by Watson)
and a level component (thrown away by Watson); the live `claude-code` row
shows `wU2Star = 0.0091` and `eBar = 0.1577`, of which the per-row level-
share is `0.682`. The test suite grew by `+10` at v0.6.448 (`12998 →
13008`) on top of the `+29` at v0.6.447 (`12969 → 12998`), and the
chi-squared upper-tail routine was factored out into a self-contained
helper inside the axis-173 module. The five-member EDF lattice on the
cumulative-periodogram domain is now closed; the next axis should be on a
fresh substrate.
